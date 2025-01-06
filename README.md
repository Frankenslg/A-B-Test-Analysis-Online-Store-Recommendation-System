## A/B Test Analysis: Online Store Recommendation System

---

### 🌟 **Introduction**

Hello Francisco!

I'm Patricio Requena 👋. It's a pleasure to be your reviewer today! My goal is to help you enhance and refine your skills as you progress in this project. I'll provide feedback on areas where your code can improve, highlighting specifics about what and how you can optimize for performance and clarity. Additionally, I'll acknowledge your strengths, helping you understand what techniques are working in your favor and how you can carry them forward to future projects.

At the end of this notebook, you'll find a general comment summarizing my thoughts and any additional recommendations. Let's get started!

You'll find my comments within green, yellow, or red boxes:

- **Green:** Everything is perfect.
- **Yellow:** Your code is good but could use some improvements or has minor issues.
- **Red:** Missing elements or major issues in your code or conclusions.

Feel free to respond to my feedback like this:

**Student's Response**

---

### 🌿 **Study Objectives**

The objective of this analysis is to evaluate the results of an A/B test conducted by an international online store to introduce an enhanced recommendation system. The key goals are:

- Determine whether the new recommendation system improves user conversion in three stages of the purchase funnel:
  1. **Product Page:** Visits to the product page.
  2. **Product Cart:** Adding products to the cart.
  3. **Purchase:** Completing a purchase.
- Verify whether the new recommendation system achieves at least a 10% increase in each funnel stage compared to the current system.

---

### 📊 **Step 1: Data Preparation**

#### 1.1 **Import Libraries**
```python
import pandas as pd
import numpy as np
from statsmodels.stats.proportion import proportions_ztest
import matplotlib.pyplot as plt
import seaborn as sns
```

#### 1.2 **Load Datasets**
```python
marketing_events = pd.read_csv('/datasets/ab_project_marketing_events_us.csv')
new_users = pd.read_csv('/datasets/final_ab_new_users_upd_us.csv')
events = pd.read_csv('/datasets/final_ab_events_upd_us.csv')
participants = pd.read_csv('/datasets/final_ab_participants_upd_us.csv')
```

**Reviewer Comment:**

Great job loading the datasets and preparing your workspace with essential libraries! 👍

#### 1.3 **Initial Data Inspection**
```python
# Check data structure and types
marketing_events.info()
new_users.info()
events.info()
participants.info()
```

| Dataset             | Records | Columns | Key Columns                              |
|---------------------|---------|---------|------------------------------------------|
| Marketing Events    | 14      | 4       | name, regions, start_dt, finish_dt       |
| New Users           | 58,703  | 4       | user_id, first_date, region, device      |
| Events              | 423,761 | 4       | user_id, event_dt, event_name, details   |
| Participants        | 14,525  | 3       | user_id, group, ab_test                 |

**Reviewer Comment:**

The initial inspection looks thorough! Ensuring data consistency at this stage sets a solid foundation for analysis. 🌟

#### 1.4 **Data Cleaning**
- **Check for Missing Values:**
```python
print(marketing_events.isnull().sum())
print(new_users.isnull().sum())
print(events.isnull().sum())
print(participants.isnull().sum())
```
- **Check for Duplicates:**
```python
print(marketing_events.duplicated().sum())
print(new_users.duplicated().sum())
print(events.duplicated().sum())
print(participants.duplicated().sum())
```
- **Convert Dates to Datetime:**
```python
new_users['first_date'] = pd.to_datetime(new_users['first_date'])
events['event_dt'] = pd.to_datetime(events['event_dt'])
marketing_events['start_dt'] = pd.to_datetime(marketing_events['start_dt'])
marketing_events['finish_dt'] = pd.to_datetime(marketing_events['finish_dt'])
```

**Reviewer Comment:**

Well done cleaning the data and converting date columns to datetime format. A clean dataset is crucial for reliable analysis. 🌱

---

### 🎭 **Step 2: Exploratory Data Analysis (EDA)**

#### 2.1 **Analyzing Funnel Conversion**

- **Filter Events by Funnel Stages:**
```python
page_views = events[events['event_name'] == 'product_page']
cart_additions = events[events['event_name'] == 'product_cart']
purchases = events[events['event_name'] == 'purchase']
```

- **Count Unique Users at Each Stage:**
```python
page_views_users = page_views['user_id'].nunique()
cart_additions_users = cart_additions['user_id'].nunique()
purchases_users = purchases['user_id'].nunique()
```

| Funnel Stage       | Unique Users |
|--------------------|--------------|
| Product Page       | 38,929       |
| Product Cart       | 19,284       |
| Purchase           | 19,568       |

#### 2.2 **Conversion Rates**
```python
conversion_to_cart = cart_additions_users / page_views_users
conversion_to_purchase = purchases_users / cart_additions_users
```

| Conversion Step       | Rate    |
|-----------------------|---------|
| Product to Cart       | 49.54%  |
| Cart to Purchase      | 33.18%  |

**Reviewer Comment:**

Great job calculating the conversion rates! Your insights provide a clear understanding of user behavior through the funnel. 👌

---

### 📈 **Step 3: A/B Test Analysis**

#### 3.1 **Group-Wise Conversion Rates**

- **Filter Users by Group:**
```python
group_a_users = participants[participants['group'] == 'A']['user_id']
group_b_users = participants[participants['group'] == 'B']['user_id']
```

- **Calculate Conversion Rates:**
```python
conversion_a_cart = conversion_rate_unique_users(events_a, 'product_page', 'product_cart')
conversion_b_cart = conversion_rate_unique_users(events_b, 'product_page', 'product_cart')
conversion_a_purchase = conversion_rate_unique_users(events_a, 'product_cart', 'purchase')
conversion_b_purchase = conversion_rate_unique_users(events_b, 'product_cart', 'purchase')
```

| Conversion Step         | Group A | Group B |
|-------------------------|---------|---------|
| Product to Cart         | 31.47%  | 32.99%  |
| Cart to Purchase        | 33.51%  | 32.50%  |

#### 3.2 **Z-Test for Statistical Significance**
```python
z_stat_cart, p_value_cart = proportions_ztest([success_a_cart, success_b_cart], [total_a_page, total_b_page])
z_stat_purchase, p_value_purchase = proportions_ztest([success_a_purchase, success_b_purchase], [total_a_cart, total_b_cart])
```

| Test Step                | Z-Score | P-Value |
|--------------------------|---------|---------|
| Product to Cart          | -3.26   | 0.00112 |
| Cart to Purchase         |  0.72   | 0.47302 |

**Reviewer Comment:**

Excellent use of statistical tests! The results indicate that while there is a significant difference in conversion to cart, the difference in conversion to purchase is not statistically significant.

---

### 💡 **Conclusions**

1. **Conversion Insights:**
   - The new recommendation system improved conversion to cart for Group B but showed no significant improvement in conversion to purchase.

2. **Statistical Significance:**
   - The increase in conversion to cart is statistically significant.
   - No significant difference was observed in conversion to purchase.

3. **Recommendations:**
   - Investigate why the conversion advantage seen in the cart stage does not carry over to purchases.
   - Explore potential friction points in the purchase process, such as user interface issues or unexpected costs.

---

**Date:** 2025-01-06

**Francisco SLG**

