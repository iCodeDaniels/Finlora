**Finlora: Data Cleaning & EDA Summary**

Stage 1 cleaned two datasets a 7,200-row accounts table and a 126,000 row transactions table resolving nulls in `merchant_name`, `device_id`, and `is_new_device`, and converting `timestamp` to a proper datetime type. Both tables came out fully clean, with zero nulls and zero duplicates.

Stage 2 began by merging the two tables on `account_id` using a left join, which preserved all 126,000 transaction rows while attaching account-level details like KYC tier and account type. Overlapping column names from both tables (`account_type`, `kyc_tier`, `currency`, `home_country`) initially created duplicate `_x`/`_y` columns, which were resolved by dropping the redundant copies and renaming the rest.

The target variable, `is_fraud`, turned out to be heavily imbalanced only 2.66% of transactions are fraud, meaning accuracy alone won't be a meaningful metric for any model built later; precision, recall, and AUC will matter more.

Several features showed strong, genuine fraud signals. `Amount` was around 20 times higher for fraud transactions at the median. `Amount_to_avg_ratio` how unusual a transaction is relative to that account's normal spending was about 6 times higher for fraud at the median, and the gap widened dramatically at higher percentiles, making it especially good at catching extreme, obvious fraud cases. `Transaction_velocity_1h` showed the strongest overall correlation with fraud at 0.577, suggesting rapid repeat transactions are a meaningful, consistent fraud indicator. By contrast, `account_age_days` showed almost no difference between fraud and non-fraud groups and was ruled out, as were negative transaction amounts, which actually showed a slightly lower fraud rate than the overall baseline.

Category-level patterns also emerged. Business accounts had nearly double the fraud rate of Individual accounts (3.94% versus 1.98%). Lower-verification KYC tiers carried more risk than higher tiers — Tier1_Basic sat at 3.25% fraud versus 2.20% for Tier3_Enhanced. Among merchant categories, Wire Transfer, Payroll Transfer, and Crypto Exchange had the highest fraud rates, all well above the 2.66% baseline, while everyday spending categories like restaurants and groceries sat well below it. Channel showed a smaller effect, with API/Integration slightly elevated and USSD the lowest.

Finally, a correlation check across all numeric features found no pair strongly correlated with each other, meaning there's no dangerous redundancy to worry about heading into modeling.

Overall, the EDA stage is complete. Fraud in this dataset is rare but shows consistent, explainable patterns tied to transaction size, spending behavior relative to the account's norm, transaction velocity, account type, KYC tier, and merchant category all of which give a solid foundation to build features from in Stage 3.
