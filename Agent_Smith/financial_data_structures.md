# Wells Fargo Credit Card #
## CSV structure ##
```csv
"01/29/2026","-22.11","*","","OPENAI *CHATGPT SUBSCR OPENAI.COM CA"
"01/29/2026","-10.48","*","","CHIPOTLE 2575 BELLEVUE WA"
"01/29/2026","-9.47","*","","MCDONALD'S F10651 BELLEVUE WA"
"01/28/2026","-9.47","*","","MCDONALD'S F10651 BELLEVUE WA"
```
## Ingestion process ##
Data ingestion begins by downloading a CSV file of recent Wells Fargo credit card transactions to my local machine. A script then parses this CSV and uploads the records into the Postgres database. Once stored, the data can be accessed by the Flask backend via standard database queries, exposed through API endpoints consumed by the React app, and ultimately made available through MCP tools, including interaction from clients like Claude Code.
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
### Script for taking csv data and uploading to postgres ###
```python
#!/usr/bin/env

"""
Import Wells Fargo credit card transactions from CSV to PostgreSQL
"""
import csv
import os
import sys
from pathlib import Path
from datetime import datetime
import psycopg2
from psycopg2.extras import execute_values

def find_csv_file():
    """Find CSV file in Downloads directory."""
    downloads = Path('~/Downloads').expanduser()
    csv_files = list(downloads.glob('CreditCard*.csv'))

    if not csv_files:
        print(f"No CSV file found in {downloads}")
        return None
    
    if len(csv_files) == 1:
        return csv_files[0]
    
    # Multiple files - let user choose
    print("Muliple CSV files found:")
    for i, file in enumerate(csv_files, 1):
        print(f"{i}. {file.name}")

    choice = int(input("Enter file number: ")) - 1
    return csv_files[choice]

def parse_amount(amount_str):
    """Parse amount, handling negative values and currency symbols."""
    cleaned = amount_str.replace('$', "").replace(',', '').strip()
    return float(cleaned)

def parse_date(date_str):
    """Parse date from common formats."""
    format = '%m/%d/%Y'
    try:
        return datetime.strptime(date_str, format).date()
    except ValueError:
        raise ValueError(f"Cannot parse date: {date_str}")

def import_transactions(csv_path, db_name='agent_smith'):
    """Import transactions from CSV to PostgreSQL database."""
    """Import transactions from CSV to PostgreSQL"""
    
    # Connect to PostgreSQL
    conn = psycopg2.connect(
        dbname=db_name,
        user=os.getenv('PGUSER', 'postgres'),
        password=os.getenv('PGPASSWORD', ''),
        host=os.getenv('PGHOST', 'localhost'),
        port=os.getenv('PGPORT', '5432')
    )
    
    try:
        with open(csv_path, 'r', encoding='utf-8-sig') as f:
            reader = csv.reader(f)

            # Wells Fargo CSV format (no headers):
            # Column 0: Date
            # Column 1: Amount
            # Column 2: Posted flag (*)
            # Column 3: Reference (often empty)
            # Column 4: Merchant/Description

            # Prepare data for batch insert
            transactions = []
            for row in reader:
                try:
                    if len(row) < 5:
                        print(f"Skipping row with insufficient columns: {row}")
                        continue

                    transaction = (
                        parse_date(row[0]),           # transaction_date
                        parse_amount(row[1]),         # amount
                        row[2] if row[2] else None,   # posted_flag
                        row[3] if row[3] else '',     # raw_reference
                        row[4],                       # raw_merchant
                        None,  # merchant_id (will be set by application logic)
                        None   # category_id (will be set by application logic)
                    )
                    transactions.append(transaction)
                except Exception as e:
                    print(f"Skipping row due to error: {e}")
                    print(f"Row data: {row}")
            
            # Batch insert
            with conn.cursor() as cur:
                execute_values(
                    cur,
                    """
                    INSERT INTO wells_fargo_credit_card_transactions 
                    (transaction_date, amount, posted_flag, raw_reference, 
                     raw_merchant, merchant_id, category_id)
                    VALUES %s
                    """,
                    transactions
                )
                conn.commit()
                print(f"\n✓ Successfully imported {len(transactions)} transactions")

        # Delete the CSV file after successful import
        os.remove(csv_path)
        print(f"✓ Deleted {csv_path}")

    except Exception as e:
        conn.rollback()
        print(f"Error: {e}")
        raise
    
    finally:
        conn.close()

if __name__ == '__main__':
    downloads_dir = sys.argv[1] if len(sys.argv) > 1 else '~/Downloads'
    
    csv_file = find_csv_file()
    if csv_file:
        print(f"Importing from: {csv_file}")
        import_transactions(csv_file)
    else:
        print("No CSV file found")
```
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
