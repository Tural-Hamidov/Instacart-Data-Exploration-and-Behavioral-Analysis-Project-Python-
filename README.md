# 📊 Instacart Data Exploration and Behavioral Analysis Project (Python)

## 📌 Project Overview

This project focuses on the structured exploration, validation, cleaning, and behavioral analysis of the Instacart online grocery dataset using Python and Pandas.  
The objective was not limited to visualization. The workflow was intentionally designed as a disciplined analytical pipeline:

Data Inspection → Data Cleaning → Structural Validation → Behavioral Analysis → Customer-Level Insights → Product-Level Insights

All analytical conclusions were derived only after validating structural integrity and logical consistency of the dataset.

The dataset consists of five relational tables:

- orders
- order_products
- products
- aisles
- departments

These tables collectively represent customer purchase behavior at order-level and product-level granularity.

---

## 🔍 Initial Data Inspection and Structural Validation

Before conducting any analysis, the structure of each dataset was examined using:

- .info(show_counts=True)
- .describe(include="all")
- shape validation
- uniqueness checks on ID columns

This stage ensured:

- Correct data types
- Detection of missing values
- Identification of potential structural inconsistencies

Special attention was given to relational integrity between order_id, product_id, and user_id.

---

## 🧹 Duplicate Detection and Removal

### 📦 Orders Table

- 15 fully duplicated rows were detected in the orders table.
- These rows were removed to prevent:
  - Artificial inflation of total order counts
  - Distortion of customer-level frequency metrics
  - Incorrect reorder calculations

Post-cleaning validation confirmed:

- No remaining duplicate rows
- Full uniqueness of order_id

This ensured reliable relational joins with order_products.

---

### 🏷️ Products Table

- No fully duplicated rows detected.
- No duplicate product_id values found.

After converting product names to lowercase for comparison:

- 1,361 duplicated names were detected.
- 104 duplicates remained after excluding missing values.

These were not removed because identical product names may correspond to different product IDs. Removing them would compromise dataset integrity.

---

### 🛒 Order_Products Table

Validation confirmed:

- No fully duplicated rows.
- No duplicate (order_id, product_id) pairs.

This was critical to ensure correct basket size computation and reorder analysis.

---

## ⚙️ Missing Value Handling Strategy

### 🏷️ product_name

All missing product_name values were associated with:

- aisle_id = 100
- department_id = 21

Department 21 is labeled "missing".

Instead of dropping rows:

- Missing values were replaced with "Unknown".

Rationale:

- Preserve dataset completeness
- Prevent null-related errors during groupby and merge operations

---

### 📆 days_since_prior_order

Missing values were carefully analyzed.

Finding:

- Missing values occur only for first-time orders.

Decision:

- Replace missing values with 0
- Convert column to integer

Rationale:

The first order logically has no prior interval. Using mean or median imputation would introduce behavioral distortion.

---

### 🛍️ add_to_cart_order

Observed valid range: 1–64

Missing values appeared only in orders containing more than 64 items.

Decision:

- Replace missing values with 999 (marker value)
- Convert column to integer

Rationale:

- Preserve cart structure
- Avoid dropping rows
- Clearly flag abnormal cart positions

---

## 🔢 Data Type Adjustments

The following columns were explicitly converted to integer:

1. days_since_prior_order
2. add_to_cart_order

These columns initially became float due to NaN values.

Conversion ensured:

- Computational stability
- Logical numeric consistency
- Elimination of float artifacts in integer-based variables

---

## ✅ Logical Range Validation

Before behavioral analysis:

- order_hour_of_day was validated to be within [0, 23].
- order_dow was validated to be within [0, 6].

Both passed validation, confirming temporal integrity.

---

# 📈 Behavioral and Analytical Findings

## ⏰ Hourly Order Distribution

Analysis of order_hour_of_day revealed structured purchasing behavior:

- Minimal activity between 00:00–05:00
- Rapid increase after 07:00
- Peak activity between 10:00–16:00
- Gradual decline after 17:00

Interpretation:

Customer purchasing behavior is strongly concentrated during daytime hours. Orders are not randomly distributed across the day. This suggests routine-based shopping behavior aligned with daily schedules.

---

## 📅 Weekly Order Distribution

Analysis of order_dow showed:

- Highest order volume on Sunday and Monday
- Midweek decline (especially Wednesday & Thursday)
- Slight increase toward Friday and Saturday

Interpretation:

Customers exhibit weekly purchasing cycles, likely related to household replenishment patterns.  
Shopping activity is not evenly distributed throughout the week.

---

## 🔁 Time Until Next Order (Reorder Interval Analysis)

Histogram analysis of days_since_prior_order showed:

- High concentration within 7–10 days
- Significant spike at 30 days

Interpretation:

Two dominant purchasing cycles exist:

1. Weekly shopping pattern
2. Monthly shopping pattern

This confirms periodic purchasing behavior rather than random reordering.

---

## 🆚 Wednesday vs Saturday Comparison

Separate hourly histograms were generated for Wednesday and Saturday.

Findings:

- Similar peak structure (10:00–15:00)
- Saturday shows slightly higher peak intensity
- Saturday activity extends later into the evening

Interpretation:

Weekend shopping behavior is more flexible and less time-constrained compared to midweek patterns.

---

## 👥 Orders per Customer Distribution

Using groupby(user_id) and maximum order_number:

- Majority of customers placed between 1–20 orders.
- Small subset placed 50+ orders.
- Distribution is right-skewed.

Interpretation:

Customer engagement is uneven.  
A small segment contributes disproportionately to total order volume, resembling a Pareto-like distribution.

---

## 🛒 Basket Size Distribution

Computed by grouping order_products by order_id.

Descriptive statistics:

- Mean: 10.10 items
- Median: 8 items
- 50% between 5–14 items
- Maximum: 127 items
- Standard deviation: 7.54

Distribution is right-skewed.

Interpretation:

Typical orders contain a moderate number of items. Extremely large baskets are rare but present as outliers.

---

## 🥇 Top 20 Most Ordered Products

Frequency analysis showed:

- Banana (66,050 orders) ranks first.
- Organic fruits and fresh produce dominate.
- Dairy products also appear frequently.

Interpretation:

Demand is driven primarily by staple grocery items, particularly fresh and organic products.

---

## 🔄 Top 20 Most Reordered Products

Filtering for reordered == 1 revealed:

- Strong overlap with most ordered products.
- Banana again ranks first (55,763 reorders).

Interpretation:

Popular products are also repeatedly purchased, indicating habitual consumption behavior.

---

## 📊 Reorder Proportion per Product

Calculated using:

groupby(product_id)["reordered"].mean()

Some products showed reorder proportion = 1.0.

Interpretation:

- Indicates strong repeat behavior.
- However, may reflect small sample size for certain products.
- High reorder rate does not necessarily imply high overall demand.

---

## 👤 Reorder Proportion per Customer

Customer-level reorder proportion:

- Mean ≈ 0.49
- Median ≈ 0.50
- Range: 0–1

Histogram revealed polarization:

- Customers who rarely reorder
- Balanced shoppers
- Highly loyal customers

Interpretation:

Customer base is behaviorally heterogeneous.  
There is no single loyalty pattern across the platform.

---

## 🛍️ First Item Added to Cart

Filtering for add_to_cart_order == 1 showed:

Most common first-added items:

- Banana
- Milk
- Organic produce

Interpretation:

Customers tend to begin shopping sessions with essential or frequently consumed items.  
This suggests structured and goal-oriented shopping behavior rather than random browsing.

---

## 🧠 Overall Analytical Interpretation

The project demonstrates that:

- Purchasing behavior is temporally structured (daily and weekly cycles).
- Reordering behavior follows predictable periodic patterns.
- Basket sizes are moderately stable and right-skewed.
- Fresh produc
