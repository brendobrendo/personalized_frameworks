# Wells Fargo Credit Card Data Ingestion Process #

Data ingestion begins by downloading a CSV file of recent Wells Fargo credit card transactions to my local machine. A script then parses this CSV and uploads the records into the Postgres database. Once stored, the data can be accessed by the Flask backend via standard database queries, exposed through API endpoints consumed by the React app, and ultimately made available through MCP tools, including interaction from clients like Claude Code.

## CSV structure ##

Card activity data is downloaded as a CSV file from the Wells Fargo web platform. When exporting the data, I can select a start and end date, with a maximum historical range of 120 days.

Each row in the CSV contains five values. Wells Fargo does not publish formal documentation describing the schema, so some fields are inferred based on observed patterns.

- **Column 1** appears to be the transaction date, formatted as `MM/DD/YYYY`.
- **Column 2** is the transaction amount, where charges are represented as negative values and payments as positive values.
- **Column 3** consistently contains the value `"*"`.
- **Column 4** is consistently an empty string.
- **Column 5** is a free-form transaction description, typically including the merchant name and location.

An example of the exported CSV data is shown below:
```csv
"01/05/2026","-21.77","*","","MADRONA GROCERY OUT SEATTLE WA"
"01/04/2026","2378.01","*","","AUTOMATIC PAYMENT - THANK YOU"
"01/04/2026","-4.26","*","","STARBUCKS STORE 03702 SEATTLE WA"
"01/04/2026","-13.68","*","","BERT'S RED APPLE SEATTLE WA"
"01/03/2026","-15.48","*","","OXBOW SEATTLE WA"
```

## CSV Parser script ##
This script automates the ingestion of Wells Fargo credit card transaction data from a locally downloaded CSV file into a PostgreSQL database. Its primary purpose is to take a raw, undocumented CSV export from the Wells Fargo website, apply minimal but necessary parsing, and persist the data in the `wells_fargo_credit_card_transactions` table for downstream use by the application.

At a high level, the script:

1. Locates a Wells Fargo credit card CSV file in the user’s `Downloads` folder.
2. Parses dates and amounts into strongly typed Python values.
3. Transforms each CSV row into a database-ready transaction record.
4. Inserts all transactions into PostgreSQL in a single batch.
5. Deletes the CSV file after a successful import to prevent duplicate ingestion.

---

### Major parts of the script ###

### 1. CSV discovery (`find_csv_file`) ###

This function searches for files matching `CreditCard*.csv` in the user’s `~/Downloads` directory.

- If no matching file is found, the script exits cleanly.
- If multiple matching files are found, the user is prompted to choose which file to import.

This approach keeps the workflow simple and compatible with manual downloads, without hard-coding file paths.

---

### 2. Parsing helpers (`parse_date`, `parse_amount`) ###

These helper functions handle the quirks of Wells Fargo’s CSV format:

- **`parse_date`** converts `MM/DD/YYYY` strings into Python `date` objects.
- **`parse_amount`** removes currency symbols and commas, then converts values to floats, preserving Wells Fargo’s sign convention (charges are negative, payments are positive).

Centralizing this logic ensures consistent typing before data reaches the database.

---

### 3. Database import logic (`import_transactions`) ###

This function contains the core ingestion logic:

- Establishes a PostgreSQL connection using environment variables for credentials.
- Reads the CSV file row by row (the Wells Fargo export does not include headers).
- Maps each row into a structured transaction tuple aligned with the database schema.
- Accumulates valid rows in memory.
- Performs a single batch insert using `execute_values` for efficiency.
- Commits the transaction and reports how many rows were imported.

Malformed rows are skipped with a logged error, allowing the import to continue without failing entirely.

---

### 4. Cleanup and error handling ###

After a successful import:

- The CSV file is deleted to avoid accidental re-imports.
- The database connection is closed cleanly.

If an error occurs during import:

- The transaction is rolled back.
- The exception is raised so failures are visible and actionable.

---

### Most important part of the code ###

The most critical section of the script is the row-transformation logic inside `import_transactions`, where raw CSV data is converted into schema-aligned records:

```python
transaction = (
    parse_date(row[0]),           # transaction_date
    parse_amount(row[1]),         # amount
    row[2] if row[2] else None,   # posted_flag
    row[3] if row[3] else '',     # raw_reference
    row[4],                       # raw_merchant
    None,  # merchant_id (set later by application logic)
    None   # category_id (set later by application logic)
)
transactions.append(transaction)
```

This block is responsible for data integrity. Any error here propagates directly into the database and affects downstream reconciliation, normalization, and analytics. The subsequent batch insert via `execute_values` is equally important, as it ensures the import is fast, atomic, and scalable.

## Postgres table ##

The `wells_fargo_credit_card_transactions` table lives in the `public` schema and stores individual credit card transactions from Wells Fargo. Each row represents a single transaction and includes both required transactional fields and optional normalization metadata.

### Table: `wells_fargo_credit_card_transactions` ###

| Column Name        | Data Type                  | Nullable | Default             | Description |
|--------------------|----------------------------|----------|---------------------|-------------|
| id                 | uuid                       | No       | gen_random_uuid()   | Primary key for the transaction record |
| transaction_date   | date                       | No       | —                   | Date the transaction occurred |
| amount             | numeric(10,2)              | No       | —                   | Transaction amount; constrained to be non-zero |
| raw_merchant       | text                       | No       | —                   | Unnormalized merchant name from the source system |
| posted_flag        | char(1)                    | Yes      | —                   | Indicates whether the transaction has posted |
| raw_reference      | text                       | Yes      | —                   | Free-form reference or memo field from Wells Fargo |
| merchant_id        | uuid                       | Yes      | —                   | Foreign key to normalized merchant table |
| category_id        | uuid                       | Yes      | —                   | Foreign key to normalized category table |
| created_at         | timestamp with time zone   | No       | now()               | Record creation timestamp |
| updated_at         | timestamp with time zone   | No       | now()               | Record last update timestamp |

### Constraints & Indexes ###

| Type             | Name                    | Details |
|------------------|-------------------------|---------|
| Primary Key      | wells_fargo_credit_card_transactions_pkey | B-tree index on `id` |
| Check Constraint | —                       | Ensures `amount <> 0` |
| Foreign Key      | fk_receipt_credit_tx    | Referenced by `receipts`; `ON DELETE SET NULL` |

### Storage ###

| Property       | Value  |
|----------------|--------|
| Schema         | public |
| Access Method  | Heap   |

## Flask code ##

### Code snippet from Agent Smith Flask for reading cc data from postgres ###
```python
import os
from flask import jsonify, request, make_response
from app.utils import add_cors_headers
from app.logging_config import configure_logging
from pymongo import MongoClient
from bson.objectid import ObjectId
from . import credit_card_routes_bp
import uuid
from datetime import datetime, timedelta
from collections import defaultdict

client = MongoClient("mongodb://localhost:27017/")
db = client['FinancialData']
credit_card_collection = db['CreditCardTransactionsTest']
credit_card_statement_collection = db['CreditCardPaymentPeriodsTest']

from app import logger

# Create

# Read
def get_all_credit_card_transactions(limit=None):
    try:
        if limit is not None and isinstance(limit, int):
            transactions = list(credit_card_collection.find().sort({ "transaction_date": -1 }).limit(limit))
        else:
            transactions = list(credit_card_collection.find().sort({ "transaction_date": -1 }).limit(limit))
        for transaction in transactions:
            transaction['_id'] = str(transaction['_id'])
        return transactions
    except Exception as e:
        logger.error(f"Error fetching persons: {e}")
        raise

@credit_card_routes_bp.route('/credit_card_transactions', methods=['GET'])
def credit_card_transactions():
    try:
        sep_transactions = get_current_month_credit_card_transactions()
        sum_transactions_by_day(sep_transactions)
        limit = request.args.get('limit', default=None, type=int)
        transactions = get_all_credit_card_transactions(limit)
        return jsonify(transactions)
    except Exception as e:
        logger.error(f"Error in /credit_card_transactions endpoint: {e}")
        return jsonify({"error": str(e)}), 500

def get_transactions_by_date(transaction_date):
    try:
        # Query the collection to find transactions with the specified date
        transactions = list(credit_card_collection.find({"transaction_date": transaction_date}))

        # Convert ObjectId to string for JSON serialization
        for transaction in transactions:
            transaction['_id'] = str(transaction['_id'])

        return transactions
    except Exception as e:
        logger.error(f"Error fetching transactions by date: {e}")
        raise

@credit_card_routes_bp.route('/credit_card_transactions_by_date', methods=['GET'])
def credit_card_transactions_by_date():
    try:
        # Get the transaction_date from query parameters
        transaction_date = request.args.get('date')

        # Validate that the date parameter is provided
        if not transaction_date:
            return jsonify({"error": "The 'date' query parameter is required"}), 400
        
        # Fetch transactions for the specified date
        transactions = get_transactions_by_date(transaction_date)

        return jsonify(transactions)
    except Exception as e:
        logger.error(f"Error in /credit_card_transactions_by_date endpoint: {e}")
        return jsonify({"error": str(e)}), 500
    
def get_current_month_credit_card_transactions():
    try:
        # Get the current date and calculate the start and end of the current month
        now = datetime.now()
        start_of_month = datetime(now.year, now.month, 1)
        if now.month == 12:
            # If December, set end of month to January 1st of the next year
            end_of_month = datetime(now.year + 1, 1, 1)
        else:
            # Otherwise, set end of month to the first of the next month
            end_of_month = datetime(now.year, now.month +1, 1)

        # Query transactions that fall within the current month
        transactions = list(credit_card_collection.find({
            "transaction_date": {
                "$gte": start_of_month.strftime("%m/%d/%Y"),
                "$lte": end_of_month.strftime("%m/%d/%Y")
            }
        }))

        # Convert ObjectId to string for each transaction
        for transaction in transactions:
            transaction['_id'] = str(transaction['_id'])

        return transactions
    
    except Exception as e:
        print(f"Error fetching the current month's credit card transactions: {e}")
        return []
    
def sum_transactions_by_day(transactions):
    # Dictionary to hold the sum of transactions for each date
    daily_sums = defaultdict(float)

    for transaction in transactions:
        # Extract the date and amount from each transaction
        date_str = transaction['transaction_date']
        amount = transaction['transaction_amount']

        # Sum the transaction amounts per date
        daily_sums[date_str] += amount

    # Convert the daily_sums dictionary to a list of dictionaries
    summed_transactions = [
        {'date': date, 'daily_total': total}
        for date, total in daily_sums.items()
    ]

    # Sort the list by date
    summed_transactions.sort(
        key=lambda x: datetime.strptime(x['date'], '%m/%d/%Y')
    )

    print(summed_transactions)
    return summed_transactions
    
def get_last_credit_card_statement():
    try:
        last_statement = credit_card_statement_collection.find_one(
            sort=[("payment_period_end", -1)]
        )

        # Convert the MongoDB ObjectId to a string for the response
        if last_statement:
            last_statement['_id'] = str(last_statement['_id'])
        else:
            return jsonify({"error": "No credit card statements found"}), 404
        
        print(last_statement)
        return jsonify(last_statement), 200
    
    except Exception as e:
        logger.error(f"Error fetching the last credit card statement: {e}")
        return jsonify({"error": str(e)}), 500
    
def get_credit_card_balance_since_last_statement():
    try:

        # Fetch the most recent credit card statement
        last_statement = credit_card_statement_collection.find_one(
            sort=[("payment_period_end", -1)]
        )

        if not last_statement:
            return jsonify({"error": "No credit card statements found."}), 404
        
        # Extract necessary details from the last statement
        last_statement_end_date = datetime.strptime(last_statement['payment_period_end'], '%m/%d/%Y')
        print(last_statement_end_date)
        statement_balance = last_statement['statement_balance']
        print(statement_balance)

        # Fetch all transactions that occurred after the end of the last statement period
        transactions = list(
            credit_card_collection.find({
                "transaction_date": {
                    "$gt": last_statement_end_date.strftime('%m/%d/%Y')
                }
            }).sort("transaction_date", 1) # sort in ascending order by date
        )

        print('TRANSACTIONS: ', transactions)

        # Group transactions by day and calculate the running balance
        daily_balances = defaultdict(float)
        current_balance = statement_balance

        # Iterate over transactions and group them by day
        for transaction in transactions:
            # Convert string transaction_date to datetime object
            transaction_date = datetime.strptime(transaction['transaction_date'], '%m/%d/%Y').date()

            # Subtract transaction amount from current balance
            current_balance -= transaction['transaction_amount']

            # Store the balance for each day
            daily_balances[transaction_date] = current_balance

        # Prepare the output: a list of dates and their corresponding balances
        balance_history = [{"date": str(date), "balance": balance} for date, balance in sorted(daily_balances.items())]

        return balance_history
    
    except Exception as e:
        logger.error(f"Error fetching transactions from the last week: {e}")
        raise

@credit_card_routes_bp.route('/credit_card_balances_since_last_statement', methods=['GET'])
def credit_card_balances_since_last_statment():
    try:
        balances = get_credit_card_balance_since_last_statement()
        return jsonify(balances)
    except Exception as e:
        logger.error(f"Error in /credit_card_transactions_last_week endpoint: {e}")
        return jsonify({"error": str(e)}), 500
    
# Update
@credit_card_routes_bp.route('/credit_card_transactions/<transaction_id>', methods=['PUT', 'PATCH'])
def update_credit_card_transaction(transaction_id):
    try:
        updates = request.json
        if not updates:
            return jsonify({"error": "No update data provided"}), 400
        
        # Remove the '_id' field from updates if it exists
        if '_id' in updates:
            del updates['_id']
        
        # Convert transaction_id to ObjectId
        try:
            transaction_id = ObjectId(transaction_id)
        except Exception as e:
            return jsonify({"error": f"Invalid transaction ID: {e}"}), 400
        
        # Perform the update
        result = credit_card_collection.update_one(
            {"_id": transaction_id},
            {"$set": updates}
        )

        if result.matched_count == 0:
            return jsonify({"error": "No transaction found with the provided ID"})
        
        return jsonify({"message": "Transaction updated successfully"}), 200
    
    except Exception as e:
        logger.error(f"Error updating transaction: {e}")
        return jsonify({"error": str(e)}), 500
```
## React Typescript ##
```typescript
export interface ICCTransaction {
    _id: string;
    asterisk_field: string;
    blank_field: number;
    credit_card_balance: number | null;
    my_classification: string | null;
    my_description: string | null;
    transaction_amount: number;
    transaction_date: string;
    transaction_description: string;
    type_of_transaction: string;
    merchant: string;
    other_notes: string | null;
    receipt: IReceipt | null;
}
```
