# Finlora — Data Cleaning (Stage 1)

Data cleaning stage for the Finlora fraud risk scoring project. This covers
loading, validating, and cleaning the two raw source datasets ahead of EDA
and feature engineering.

## Datasets

| File | Rows | Columns | Description |
| `data/raw/finlora_accounts.csv` | 7,200 | 8 | One row per account — holder, type, KYC tier, spend baseline |
| `data/raw/finlora_transactions.csv` | 126,000 | 24 | One row per transaction — amount, channel, device, fraud label |

### Accounts schema

`account_id`, `account_holder_name`, `account_type` (Individual / Business),
`home_country`, `currency`, `kyc_tier`, `account_created_date`,
`personal_spend_baseline_usd`

### Transactions schema

`transaction_id`, `account_id`, `account_type`, `kyc_tier`, `timestamp`,
`day_of_week`, `hour_of_day`, `description`, `merchant_name`,
`merchant_category`, `channel`, `amount`, `currency`, `amount_to_avg_ratio`,
`avg_transaction_amount_30d`, `transaction_velocity_1h`,
`transaction_country`, `home_country`, `is_cross_border`, `device_id`,
`is_new_device`, `account_age_days`, `status`, `is_fraud` (target)

## Cleaning Steps

1. **Load the data**
   ```python
   acct = pd.read_csv("data/raw/finlora_accounts.csv")
   txn = pd.read_csv("data/raw/finlora_transactions.csv")
   ```

2. **Validate the accounts table**
   - `acct.info()` — confirmed all 8 columns fully populated (7,200 non-null)
   - `acct.isnull().sum()` — zero nulls across every column
   - `acct.duplicated().sum()` — zero duplicate rows
   - Spot-checked `account_holder_name`, `personal_spend_baseline_usd`, and
     `account_type` with `.unique()` to catch unexpected categories or typos

3. **Validate the transactions table**
   - `txn.info()` — 126,000 rows across 24 columns
   - `txn.isnull().sum()` — identified nulls in three columns:
     - `merchant_name`: 28,610 missing
     - `device_id`: 14,334 missing
     - `is_new_device`: 14,334 missing
   - `txn.duplicated().sum()` — zero duplicate rows

4. **Handle missing values**
   ```python
   txn['merchant_name'] = txn['merchant_name'].fillna('Unknown')
   txn['device_id'] = txn['device_id'].fillna('Unknown_id')
   ```
   `is_new_device` nulls were cleaned and imputed alongside `device_id`
   (rows missing a device ID had no device history to flag as new/not new).

5. **Fix data types**
   ```python
   txn['timestamp'] = pd.to_datetime(txn['timestamp'], format='%Y-%m-%d %H:%M:%S')
   ```
   Converted `timestamp` from string to `datetime64` for downstream time-based
   feature engineering.

6. **Final validation**
   - Re-ran `txn.isnull().sum()` — zero nulls across all 24 columns
   - Re-ran `txn.duplicated().sum()` — zero duplicates confirmed

## Result

Both `acct` and `txn` are fully clean: no missing values, no duplicate rows,
correct data types on the timestamp field. Ready to move into Stage 2 (EDA).

## Tools & Environment

- **Language:** Python 3.12
- **Core libraries:** pandas, numpy, matplotlib, seaborn
- **Environment:** VS Code + Jupyter Notebooks, `.venv` virtual environment
- **Notebook:** `notebook/01_data_cleaning.ipynb`
