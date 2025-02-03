# Python **Payment and Transaction Analysis**

**[Python] Payment and Transaction Analysis**
## I. Introduction
In this project, I will explore the status of payments and transactions within an e-wallet system to gain insights into its performance and user behavior. By analyzing transaction trends, payment statuses, and key metrics, this study aims to identify patterns, detect anomalies, and uncover opportunities for optimization. The findings will provide valuable insights to enhance the efficiency, security, and user experience of the e-wallet platform.

### Summary:
The purpose of this analysis is to identify products, teams, or categories that are performing well or poorly, as well as to analyze transactions with specific transaction types. Additionally, the focus will be on deeper analysis to detect unusual products or teams, contributions from refund transaction sources, and statistics on the number of transactions, total volume, and the number of senders and receivers.

### Dataset:
1. payment_report.csv (monthly payment volume of products)
2. product.csv (product information)
3. transactions.csv (transactions information)

Type of transaction:
- transType = 2 & merchant_id = 1205: Bank Transfer Transaction
- transType = 2 & merchant_id = 2260: Withdraw Money Transaction
- transType = 2 & merchant_id = 2270: Top Up Money Transaction
- transType = 2 & others merchant_id: Payment Transaction
- transType = 8, merchant_id = 2250: Transfer Money Transaction
- transType = 8 & others merchant_id: Split Bill Transaction
- Remained cases are invalid transactions

## II. Data processing
### 1. EDA:
Do EDA task:
- Df payment_enriched (Merge payment_report.csv with product.csv)
- Df transactions

Suggestions:
- Check each column: missing data? duplicates? incorrect data types?
    - Sample Answers:
        - Missing data: x rows in column A, y rows in column B -> Next step: No action/ Delete rows/…
        - Duplicates: PK? x rows? -> Next step: No action/ Delete rows/…
        - Incorrect data types: column A, column B -> Next step: No action/ Delete rows/…
        - Incorrect values: column A, column B -> Next step: No action/ Delete rows/…
- Summarize numerical data: any incorrect values?
```python
# View all rows of the payment_report DataFrame
payment_report.head(1000)

# View the first few of the product DataFrame
product.head(5)

# View the first few of the transactions DataFrame
transactions.head()

# Merge the payment_report and product DataFrames
payment_enriched = pd.merge(payment_report, product, on='product_id', how='left')
payment_enriched.head()

# Missing data checking:
payment_enriched.info()
transactions.info()

payment_enriched.duplicated().sum()
transactions.duplicated().sum()

payment_enriched.describe()
transactions.describe()

# Drop missing data in payment_enriched
payment_enriched.dropna(inplace=True)
```
### 2. Data Wrangling
- Check Top 3 `product_id` with the highest transaction volume.
```python
# Top 3 product_id:
top_3_products = payment_enriched.groupby('product_id')['volume'].sum().sort_values(ascending=False).head(3)
print(top_3_products)
```
- Check if there are any product_id violating the rule that each product_id belongs to only one team_own.
```python
# Check for any product_id that is associated with more than one team_own:
abnormal_products = payment_enriched.groupby('product_id')['team_own'].nunique()
abnormal_products = abnormal_products[abnormal_products > 1]
print(abnormal_products)
```
- Identify the team with the lowest performance since Q2 2023 and determine the category that contributes the least to the team's performance.
```python
# Since Q2.2023 data:
payment_enriched['report_month'] = pd.to_datetime(payment_enriched['report_month'])
q2_2023 = payment_enriched[payment_enriched['report_month'] >= '2023-04-01']
# The team has had the lowest performance:
lowest_team = q2_2023.groupby('team_own')['volume'].sum().idxmin()
# The category that contributes the least to that team
lowest_team_data = q2_2023[q2_2023['team_own'] == lowest_team]
lowest_category = lowest_team_data.groupby('category')['volume'].sum().idxmin()
print("Lowest Team:", lowest_team)
print("Lowest Category for the Lowest Team:", lowest_category)
```
- Analyze the contribution percentage of source_id in refund transactions and find the source_id with the highest contribution.
```python
# Filter refund transactions:
refunds = payment_enriched[payment_enriched['payment_group'] == 'refund']
print(refunds.head())
# Total refund volume:
source_contribution = refunds.groupby('source_id')['volume'].sum().sort_values(ascending=False)
print(source_contribution)
# Largest source_id:
largest_source = source_contribution.idxmax()
print("Largest Source ID:", largest_source)
```
- Calculate the number of transactions, total volume, the number of senders, and the number of receivers.
```python
# Find the number of transactions, volume, senders and receivers
def define_transaction_type(row):
    if row['transType'] == 2:
        if row['merchant_id'] == 1205:
            return 'Bank Transfer Transaction'
        elif row['merchant_id'] == 2260:
            return 'Withdraw Money Transaction'
        elif row['merchant_id'] == 2270:
            return 'Top Up Money Transaction'
        else:
            return 'Payment Transaction'
    elif row['transType'] == 8:
        if row['merchant_id'] == 2250:
            return 'Transfer Money Transaction'
        else:
            return 'Split Bill Transaction'
    else:
        return 'Invalid transactions'

transactions['transaction_type'] = transactions.apply(define_transaction_type, axis=1)
valid_transactions = transactions[transactions['transaction_type'] != 'Invalid transactions']

transaction_summary = valid_transactions.groupby('transaction_type').agg({
    'transaction_id': 'count',
    'volume': 'sum',
    'sender_id': 'nunique',
    'receiver_id': 'nunique'
}).rename(columns={'transaction_id': 'total_transactions'})
print(transaction_summary)
```
