# Data & AI Engineer — Take-home Project
## Late Deliveries & What Customers Say

## 1. The problem in your own words

The marketplace wants to understand how often customer orders are delivered late and whether late deliveries are associated with poorer customer feedback. The goal of this project is to measure late-delivery performance, compare customer review scores for late and on-time orders, and analyze what customers actually complain about in their written reviews.

The analysis uses the Olist Brazilian E-Commerce Public Dataset to build a reliable delivered-order table, quantify late deliveries, analyze review scores, and use an LLM to classify 200 customer comments into five themes. The results are then used to identify practical actions that the Operations team can take to reduce delivery-related problems and improve customer experience. 

## 2. Assumptions

- Only orders with `order_status = delivered` are included in the final order-level analysis.
- An order is considered late when the delivered customer date is later than the estimated delivery date. Orders delivered on or before the estimated date have `days_late = 0`.
- Days late is measured in whole days.
- Order value is calculated as the sum of the `price` values of all items in the order. Freight charges are not included.
- When an order contains multiple items, all item rows are aggregated to the order level before joining with the orders table.
- When an order has multiple review records, the mean review score is used as the order-level review score.
- Orders without a written review comment are excluded from the 200-review AI analysis, but they remain in the delivery analysis.
- The AI analysis uses a sample of 200 reviews and its theme percentages are treated as sample-level findings, not as estimates for all customer reviews.
- For the optional seller analysis, only sellers with at least 50 orders are included.

## 3. Data source and how to run

### Data source

The data comes from the **Brazilian E-Commerce Public Dataset by Olist** on Kaggle:

https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

The project uses these four files:

- `olist_orders_dataset.csv`
- `olist_order_items_dataset.csv`
- `olist_order_reviews_dataset.csv`
- `olist_sellers_dataset.csv` — used only for the optional Data Platform extra.

The raw CSV files are kept locally in the `Dataset/` folder and are not included in the GitHub repository.

### Tools and packages

- Python
- Jupyter Notebook
- pandas
- Power BI Desktop

The Python analysis was developed in `Caspad_Data_AI_Project.ipynb`.

### How to run

1. Download the Olist dataset from the Kaggle source above.
2. Place the required CSV files inside the `Dataset/` folder.
3. Open `Caspad_Data_AI_Project.ipynb` in Jupyter Notebook.
4. Run the notebook cells from top to bottom.
5. The notebook creates the cleaned delivered-order dataset:
   - `olist_delivered_orders.csv`
6. The notebook also creates the Part C AI results and validation files:
   - `part_c_ai_results.csv`
   - `part_c_ai_prompt.txt`
   - `part_c_manual_validation_30.csv`
   - `part_c_ai_raw_outputs.txt`
7. For the optional Data Platform analysis, the notebook creates:
   - `seller_late_share.csv`
8. Open `olist_delivered_orders.csv` in Power BI Desktop to reproduce the Part B dashboard.

The Part C LLM classification was performed using ChatGPT in batches with the same fixed prompt for all 200 reviews. The raw outputs and prompt are saved for reproducibility.


## 4. Part A: cleaning rules, with number of rows affected by each, and checks

### Cleaning rules

- The orders dataset contained **99,441 orders**.
- Only orders with `order_status = delivered` were retained for the final delivered-order analysis, resulting in **96,478 delivered orders**. The remaining **2,963 non-delivered orders** were excluded.
- Order items were aggregated to the order level before joining with the orders table. This produced one `item_count` and one `order_value` per order and avoided multiplying orders when an order contained multiple items.
- `order_value` was calculated as the sum of item `price` values for each order. Freight charges were not included.
- Review records were aggregated to the order level using the mean `review_score` before joining with the order data. This prevented multiple review records from duplicating order rows.
- Dates were converted to datetime format before calculating delivery lateness.
- `days_late` was calculated as the difference between the delivered customer date and the estimated delivery date. Negative values were replaced with 0, so orders delivered on or before the estimated date are treated as on-time.
- The final delivered-order table contains **96,478 rows**, with one row per delivered order.
- **646 delivered orders** had no review score. These were not filled with an artificial value; they remain missing and are excluded from review-score calculations.
- No delivered orders were missing `item_count` or `order_value`.

### Data-quality checks

The following checks were performed on the final delivered-order table:

1. **Order ID uniqueness:** passed. Each delivered order appears exactly once.
2. **Delivered date after purchase date:** passed. There were **0** delivered orders where the delivered date was earlier than the purchase date.
3. **Order value validation:** passed. The calculated `order_value` matched the raw sum of item `price` values for all delivered orders, with **0 mismatches**.

## 5. Part B: key numbers and charts

The Power BI analysis was built from the final delivered-order table containing 96,478 delivered orders.

### Late delivery share

- Late orders: **6,534**
- Late delivery share: **6.77%**

**Explanation:** 6.77% of delivered orders were delivered after the estimated delivery date.

### Late delivery share by month

A line chart was created using the order purchase month to show how the late-delivery share changed over time.

**Explanation:** The monthly view shows how late-delivery performance varied across the available order months.

### Average review score: late vs on-time

- On-time orders: **4.29**
- Late orders: **2.27**

**Explanation:** Orders delivered late had an average review score 2.02 points lower than orders delivered on time.

### Average review score for orders more than 7 days late

- Orders more than 7 days late: **1.70**

**Explanation:** Customer ratings were even lower for orders delivered more than seven days after the estimated date.

### Overall finding

Late deliveries were associated with substantially lower customer review scores in this dataset. This is an observed association and does not by itself establish that delivery delay caused the lower scores.

## 6. Part C: model or tool used, prompt, how checked, how often right

### Model/tool used

ChatGPT was used to classify **200 customer reviews containing written comments**.

The reviews were classified into five fixed themes:

1. `late_or_not_received`
2. `wrong_or_missing_item`
3. `product_quality`
4. `good_experience`
5. `other`

The same prompt was used for all 200 reviews. The reviews were submitted in multiple batches to the chat application.

### Prompt

The prompt instructed the model to:

- assign exactly one of the five predefined themes;
- select the most important theme when a review mentioned multiple issues;
- identify delivery delays or non-receipt as `late_or_not_received`;
- identify wrong, missing, or incomplete orders as `wrong_or_missing_item`;
- identify defective, damaged, broken, or poor-quality products as `product_quality`;
- identify clearly positive experiences as `good_experience`;
- use `other` when the review did not fit the other categories;
- understand Portuguese reviews and provide the summary in English;
- provide a short one-sentence English summary for each review.

The full prompt is saved in `part_c_ai_prompt.txt`.

### Validation

A manual validation sample of **30 reviews** was checked by reading the original review comments and assigning a theme independently.

The AI classification was compared with the manual classification:

- Correct classifications: **30 / 30**
- Validation accuracy: **100%**

The manual validation results are saved in `part_c_manual_validation_30.csv`.

### Theme mix

The 200-review sample contained:

| Theme | Late | On-Time |
|---|---:|---:|
| Good experience | 1 (4.8%) | 113 (63.1%) |
| Late / not received | 18 (85.7%) | 10 (5.6%) |
| Other | 0 (0.0%) | 11 (6.1%) |
| Product quality | 0 (0.0%) | 27 (15.1%) |
| Wrong / missing item | 2 (9.5%) | 18 (10.1%) |
| **Total** | **21 (100%)** | **179 (100%)** |

### Finding

Within this 200-review sample, **85.7% of reviews from late orders were classified as delivery-related complaints**, compared with **5.6% for on-time orders.** On-time orders were predominantly classified as good experiences (**63.1%**).

The late-order group contains only 21 reviews, so these percentages should be treated as sample-level findings rather than estimates for all customer reviews.

### Saved AI outputs

The following files were saved:

- `part_c_ai_results.csv` — structured classifications and summaries.
- `part_c_ai_prompt.txt` — exact prompt used.
- `part_c_manual_validation_30.csv` — manual validation results.
- `part_c_ai_raw_outputs.txt` — raw ChatGPT outputs.

## 7. Limitations and what I would do next

- The AI theme analysis uses a sample of **200 reviews**, so the theme percentages should not be treated as representative of all customer reviews.
- Only reviews containing written comments were considered for the AI analysis. Reviews without written comments were not classified.
- The manual AI validation covered **30 reviews**, so the measured 100% validation accuracy applies only to this checked sample.
- The analysis uses the estimated delivery date provided in the source data. It does not investigate why an order became late or identify the specific operational cause.
- The analysis shows an association between late delivery and lower review scores, but it does not prove that late delivery directly caused lower customer satisfaction.
- The `days_late` calculation uses whole days, so partial days are not represented.
- The dataset is historical Olist data from 2016–2018, so the findings may not reflect current marketplace operations.

### What I would do next

- Classify a larger sample of customer reviews and expand the manual validation set.
- Analyze late deliveries by seller, location, product category, and logistics characteristics to identify operational patterns.
- Investigate the relationship between delivery delay length and customer satisfaction in more detail.
- Build an automated and repeatable AI classification pipeline instead of manually submitting review batches.
- Monitor these delivery and customer-experience metrics regularly so the Operations team can identify deteriorating performance earlier.

## 8. How I used AI tools

AI tools were used selectively as supporting tools during the project. I was responsible for the implementation, data analysis, validation, Power BI work, and final decisions.

ChatGPT was used for occasional code review, troubleshooting, and checking implementation approaches during development.

Claude was used as an additional tool for reviewing selected code and comparing implementation approaches.

For Part C, ChatGPT was used to classify the 200 selected customer reviews using the fixed prompt documented in part_c_ai_prompt.txt.

The AI classifications were manually checked against 30 original reviews to evaluate the results, with 30 out of 30 classifications matching the manual validation.

AI suggestions were reviewed and tested before being incorporated into the project. Final calculations, data-quality checks, analytical findings, and business recommendations were verified against the project data.

The Part C prompt and raw AI outputs are saved in the project files for transparency and reproducibility.

## 9. Optional extra

### Seller-level late-delivery analysis

As the Data Platform optional extra, I added the Olist sellers dataset and calculated late-delivery share at seller level.

To avoid counting multiple items from the same order as multiple orders, unique `order_id` and `seller_id` pairs were created before calculating seller order counts.

Only sellers with **at least 50 orders** were included.

- Eligible sellers: **429**
- Minimum seller order count: **50**
- Seller details matched successfully: **100%**
- Late-delivery share was calculated as the number of late delivered orders divided by the number of delivered orders for each seller.
- The highest observed late-delivery share among eligible sellers was **30.1%**.

This analysis shows that late-delivery performance varies substantially across sellers and can help the Operations team prioritize sellers with high observed late-delivery shares.

The resulting seller-level dataset is saved as `seller_late_share.csv`.

