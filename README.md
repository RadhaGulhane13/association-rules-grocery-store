# Association Rules for Grocery Store
## Objectives
The objectives of this project are:
- Practice the Association Analysis
- Understand “why” the particular topics, techniques, etc., are important from a practical perspective.
- Understand how to choose and use appropriate tools to solve the provided problems.

## The Dataset
The "AssociationsGroceriesDataV1.xlsx" dataset contains is a market basket dataset containing transactions.
The data file captures the data in "long format". Specifically, every row corresponds to the transaction id and the item. If the specific transaction id has multiple items, there will be multiple rows in the data.

## The Business Problem
Assume this dataset contains all of the transactions for one month for our store. We wish to find association rules that would improve our revenue as follows:
We would discount one of our products by 10% each month, with the hope that this would encourage customers to visit our store to purchse that product 5% more frequently, and also purchase other products (that are not discounted) more frequently.
Practically speaking, we would like to come up with two-item rules (one antecedent and one consequent: (A -> B)) and choose the one that best adds to our revenues (based on the rule support, confidence, etc.).

## Create Two-Itemsets

To get Two-Itemsets we can use FPGrowth and Apriori, both the algorithms.

Apriori generates candidate itemsets and checks them against the transaction database to determine their frequency. It employs the "Apriori principle" to prune infrequent itemsets, which helps to reduce the search space and improve efficiency. The algorithm generates a large number of candidate itemsets and then prunes them based on their frequency in the database. The main disadvantage of Apriori is that it can be computationally expensive, especially for large datasets.

FPGrowth is based on the idea of representing the database as a tree-like structure called a frequent pattern tree. It recursively builds this tree by compressing the transactions in the database into a set of frequent itemsets, and then using these itemsets to construct the tree. FPGrowth does not generate candidate itemsets like Apriori, which can make it more efficient for large datasets.

```
requent_itemsets_fp= fpgrowth(ohe_df, min_support=0.01, use_colnames=True)

# Extract the top 20 two-item sets with highest support
sorted_frequent_itemsets_fp = frequent_itemsets_fp.sort_values(by='support', ascending=False)
top_20_two_itemsets_fp = sorted_frequent_itemsets_fp[sorted_frequent_itemsets_fp['itemsets'].apply(lambda x: len(x) == 2)].head(20)

```

## Generate Rules

For the two-itemsets created above, create the related rules.

``` 
freq_two_item_sets = sorted_frequent_itemsets_fp[sorted_frequent_itemsets_fp['itemsets'].apply(lambda x: len(x) <= 2) & (frequent_itemsets_fp['support'] >= 0.02999)]

rules = association_rules(freq_two_item_sets, metric='confidence', min_threshold=0.01, support_only=False)

# Sort the rules by confidence in decreasing order
rules_sorted = rules.sort_values(by='confidence', ascending=False)
```

## Rule Evaluation
For the rules created above, find the single Item (that would be given the discount) that would cause the greatest increase in monthly store revenue.
```
for index, row in rules.iterrows():
    antecedent_price = item_df[item_df['ItemName'] == list(row['antecedents'])[0]]['UnitPrice'].values[0]
    consequent_price = item_df[item_df['ItemName'] == list(row['consequents'])[0]]['UnitPrice'].values[0]

    antecedent_revenue_before_change = num_transactions * row['antecedent support'] * antecedent_price
    consequent_revenue_before_change = num_transactions * row['consequent support'] * consequent_price

    antecedent_revenue_after_change = num_transactions * row['antecedent support'] * 1.05 * antecedent_price * 0.9
    consequent_revenue_after_change = num_transactions * row['consequent support'] * row['lift'] * consequent_price

    revenue_before = antecedent_revenue_before_change + consequent_revenue_before_change 
    revenue_after = antecedent_revenue_after_change + consequent_revenue_after_change 
 
    rev_gain = revenue_after - revenue_before

    if rev_gain > max_rev_gain:
        max_rev_gain = rev_gain
        max_rev_gain_row = row
        total_rev_after_decrease_val_of_antecedent = antecedent_revenue_after_change + consequent_revenue_before_change
        total_rev_after_increase_sales_of_consequent = antecedent_revenue_before_change + consequent_revenue_after_change
        dec_in_rev = ((total_rev_after_decrease_val_of_antecedent  - revenue_before)/ revenue_before)
        inc_in_rev = ((total_rev_after_increase_sales_of_consequent  - revenue_before)/ revenue_before)
        total_rev_inc = (rev_gain / revenue_before)    
```

## Conclusion
```
choosen_antecedent = list(max_rev_gain_row[0])[0]
choosen_consequent = list(max_rev_gain_row[1])[0]
```
Choosen antecedent is : *root vegetables* and choosen consequent is : *other vegetables*
Increase in total Monthly Revenue after decrease in price of root vegetables and increase in sales of root vegetables and other vegetables : 44209.00191
Increase in total Monthly Revenue after decrease in price of root vegetables and increase in sales of root vegetables and other vegetables : 85.97%
Question 5.A: Monthly revenue decrease after decrease in price of root vegetables and increase in sales of root vegetables and other vegetables : -1.63%
Question 5.B: Monthly revenue increase after increase in sales of other vegetables : 87.6%