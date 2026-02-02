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

## Legacy Flask routes and method logic ##

This explains the legacy and discarded Flask routes and supporting logic for reading, aggregating, and updating credit card transaction data stored in MongoDB. It acts as the API layer between the application frontend and the underlying credit card transaction collections, exposing endpoints for retrieving transactions, querying by date, computing balances, and updating individual records.

I am keepin a description of the legacy functionaliy and workflow so that I can build it into my new routes and method logic that use the Postgres database.

At a high level, this module:

1. Connects to MongoDB collections that store credit card transactions and statement periods.
2. Exposes read-focused API endpoints for transactions and balances.
3. Performs lightweight aggregation and date-based calculations.
4. Supports updating individual credit card transactions via REST endpoints.

---

### Major parts of the module ###

### 1. Database and collection setup ###

At import time, the module establishes a connection to MongoDB and initializes references to the relevant collections:

- `CreditCardTransactionsTest` — stores individual credit card transactions.
- `CreditCardPaymentPeriodsTest` — stores credit card statement periods and balances.

These collections serve as the backing store for all route logic in this file.

---

### 2. Core data access helpers ###

#### `get_all_credit_card_transactions(limit=None)` ####

Fetches credit card transactions from MongoDB, sorted by `transaction_date` in descending order.

- Supports an optional `limit` query parameter.
- Converts MongoDB `ObjectId` values to strings for JSON serialization.
- Acts as the primary read path for the `/credit_card_transactions` endpoint.

---

#### `get_transactions_by_date(transaction_date)` ####

Retrieves all transactions that match a specific transaction date.

- Used by the date-filtered API endpoint.
- Ensures returned documents are JSON-safe by converting `_id` fields.

---

### 3. API routes ###

#### `GET /credit_card_transactions` ####

Returns a list of credit card transactions.

- Accepts an optional `limit` query parameter.
- Internally calls helper functions to retrieve and serialize transactions.
- Currently invokes month-based aggregation helpers (useful for analytics or dashboards).

---

#### `GET /credit_card_transactions_by_date` ####

Returns transactions for a specific date.

- Requires a `date` query parameter (`MM/DD/YYYY`).
- Returns a `400` error if the parameter is missing.
- Useful for drill-down views and day-level analysis.

---

#### `GET /credit_card_balances_since_last_statement` ####

Returns a daily running balance starting from the most recent credit card statement.

- Uses the last statement’s ending balance as a baseline.
- Applies all subsequent transactions in chronological order.
- Returns a list of `{ date, balance }` records suitable for charts or trend analysis.

---

### 4. Date-based aggregation helpers ###

#### `get_current_month_credit_card_transactions()` ####

Fetches transactions that fall within the current calendar month.

- Calculates the start and end of the month dynamically.
- Filters transactions using string-based date comparisons.
- Primarily used as input for aggregation logic.

---

#### `sum_transactions_by_day(transactions)` ####

Aggregates a list of transactions by day.

- Groups transactions by `transaction_date`.
- Computes daily spending totals.
- Sorts results chronologically.
- Returns a list of `{ date, daily_total }` objects.

This function is useful for generating daily spend charts or summaries.

---

### 5. Statement and balance logic ###

#### `get_last_credit_card_statement()` ####

Fetches the most recent credit card statement from MongoDB.

- Sorts by `payment_period_end` descending.
- Converts the statement `_id` to a string for API responses.
- Acts as a building block for balance calculations.

---

#### `get_credit_card_balance_since_last_statement()` ####

Calculates a running balance starting from the last statement balance.

- Retrieves all transactions after the statement end date.
- Applies each transaction sequentially.
- Produces a day-by-day balance history.
- Returns structured data suitable for time-series visualization.

---

### 6. Update route ###

#### `PUT / PATCH /credit_card_transactions/<transaction_id>` ####

Updates an individual credit card transaction.

- Accepts partial updates via JSON payload.
- Strips `_id` from the update payload to prevent accidental mutation.
- Validates the provided `transaction_id`.
- Returns appropriate error responses if the transaction is not found or input is invalid.

---

### Most important part of the code ###

The most critical logic in this module is the **balance calculation since the last statement**, found in `get_credit_card_balance_since_last_statement()`:

```python
current_balance = statement_balance

for transaction in transactions:
    transaction_date = datetime.strptime(
        transaction['transaction_date'], '%m/%d/%Y'
    ).date()

    current_balance -= transaction['transaction_amount']
    daily_balances[transaction_date] = current_balance
```

This block defines how your system derives a user’s *current credit card balance* from historical data. Any error here directly impacts financial accuracy, reporting, and trust in downstream dashboards. It is the conceptual bridge between static statement data and real-time transaction activity.

This Python file defines the Flask route and supporting logic for **reading Wells Fargo credit card transaction data from Postgres and exposing it via an HTTP API**. It serves as the boundary between the `wells_fargo_credit_card_transactions` database table and any frontend, service, or MCP client consuming the `/credit_card_transactions` endpoint.

At a high level, this file is responsible for:

- Querying credit card transaction records from Postgres
- Converting database rows into JSON-serializable objects
- Returning structured transaction data over HTTP
- Handling errors and ensuring database sessions are properly closed

---

## Major parts of the script

### 1. Imports and dependencies

#### Dependencies: ####

- **Flask** handles request parsing and JSON responses.
- **SQLAlchemy `text` queries** are used instead of ORM models for explicit control.
- **`get_db_session()`** abstracts Postgres connection management.
- **Flask Blueprint** cleanly namespaces the credit card routes.
- **Centralized logging** captures runtime and query errors.

---

### 2. Row serialization helper (`row_to_dict`)

```python
def row_to_dict(row):
    """Convert a SQLAlchemy row to a dictionary with serializable values."""
```

This helper function converts a raw SQLAlchemy result row into a JSON-safe dictionary:

- UUIDs are cast to strings
- Dates and timestamps are converted to ISO-8601 strings
- Numeric values are explicitly cast to floats
- Nullable foreign keys are handled defensively

This function defines the **API contract** for credit card transactions and cleanly separates database representation from response format.

---

### 3. Route definition (`GET /credit_card_transactions`)

```python
@credit_card_routes_bp.route('/credit_card_transactions', methods=['GET'])
def get_all_transactions():
```

This route:

- Accepts an optional `limit` query parameter
- Queries the `wells_fargo_credit_card_transactions` table
- Orders results by `transaction_date` (most recent first)
- Serializes each row into a dictionary
- Returns the result as JSON

Raw SQL is used deliberately to keep query behavior predictable and transparent.

---

### 4. Query construction and execution

```python
query = """
    SELECT * FROM wells_fargo_credit_card_transactions
    ORDER BY transaction_date DESC
"""
if limit is not None:
    query += " LIMIT :limit"
    result = db.execute(text(query), {"limit": limit})
```

This logic:

- Adds a `LIMIT` clause only when requested
- Uses bound parameters to prevent SQL injection
- Keeps SQL readable and easy to debug

---

### 5. Error handling and cleanup

```python
except Exception as e:
    logger.error(f"Error in /credit_card_transactions endpoint: {e}")
    return jsonify({"error": str(e)}), 500
finally:
    db.close()
```

This ensures:

- Errors are logged with useful context
- Clients receive meaningful HTTP 500 responses on failure
- Database connections are always released

This pattern prevents connection leaks and silent failures.

---

## Most important part of the code

The most critical section of this file is the **row serialization and schema alignment layer**:

```python
transactions = [row_to_dict(row) for row in result]
```

This line is where:

- Database schema meets the API contract
- Column names must match exactly
- Data types are normalized for downstream consumers

Any mismatch here immediately breaks the endpoint, making this the most fragile and important seam in the system.

---

## Important note

The route currently returns a placeholder response:

```python
return jsonify({'value': 'hello world'})
```

Once development testing is complete, this should be replaced with:

```python
return jsonify(transactions)
```

to return the actual transaction data.

---

## React Typescript mismatch between old and new interface structures ##

The legacy interface was pulling data downstream from a mongodb and the new interface is pulling data from postgres- hence the difference in the id key names- '_id' vs 'id'. Other differences: 

I also think I can take out the 'asterisk_field' and 'blank_field' keys from each because they are not used in the React code. I also think I can take out the 'credit_card_balance' key from both because that is not being used and I think I will find a different way of implementing the balance calculation by having a separate table for credit card statements.


### New Interface Structure ###
```typescript
export interface ICreditCardTransaction {
    id: string; // 
    credit_card_balance: number | null; // Calculation method not implemented yet
    my_classification: string | null; // Can this map to category_id?
    my_description: string | null; // Can this map to raw_merchant
    transaction_amount: number; // Can this map to amount?
    transaction_date: string;
    transaction_description: string; // Can this map to raw_merchant?
    type_of_transaction: string; // This will be either CASH_INFLOW or CASH_OUTFLOW based on if amount is negative or positive
    merchant: string; // Can this be raw_merchant for now?
    other_notes: string | null;
    receipt: IReceipt | null; // Potentially change this to receipt id. Then can call new api call to get receipt info one at a time in another interface
}
```
### Legacy Interface Structure ###
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

# Wells Fargo Checking Account Data Ingestion Process #

# Reconciliation explanation #

Reconciliation SQL refers to a query (or small set of queries) that verifies your statement data is internally consistent by comparing what Wells Fargo says your balances are with what your transaction data mathematically implies they should be—typically:
starting_balance + sum(posted transactions in the statement window) = ending_balance. In practice, this SQL groups transactions by statement_id (or by transaction_date between start_date and end_date), sums the posted amounts, and then calculates a delta against the stored ending_balance; any non-zero difference flags missing transactions, duplicate imports, incorrect sign conventions, or timing issues (pending vs posted). This is exactly the kind of guardrail you want in Agent Smith to catch ingestion bugs early and keep downstream analytics trustworthy.

On the second point: no—there is not already a trigger that auto-assigns statement_id, because in the earlier steps we only created the wells_fargo_credit_card_statements table (and optionally added a statement_id column to transactions). A trigger would be an additional, explicit database object that fires on INSERT/UPDATE of transactions and assigns the correct statement based on transaction_date (and possibly posted_flag). If you want, the next clean step is to define that trigger so the database—not your Flask code—owns statement assignment logic.
