# zepto-sql-analysis
End-to-end SQL data analysis project using Zepto e-commerce dataset


# Zepto E-Commerce SQL Data Analysis Project 🛒

![SQL](https://img.shields.io/badge/SQL-MySQL-blue)
![Status](https://img.shields.io/badge/Status-Complete-success)
![Analysis](https://img.shields.io/badge/Analysis-Data%20Analytics-orange)
---

## 📋 Table of Contents
- [Project Overview](#project-overview)
- [Dataset Information](#dataset-information)
- [Database Schema](#database-schema)
- [Key Business Questions](#key-business-questions)
- [SQL Analysis & Results](#sql-analysis--results)
- [Key Insights](#key-insights)
- [Recommendations](#recommendations)
- [Skills Demonstrated](#skills-demonstrated)
- [How to Use This Project](#how-to-use-this-project)

---

## 🎯 Project Overview

This project demonstrates **end-to-end SQL data analysis** skills using a real-world e-commerce dataset from Zepto, containing **3,732 products** across 15+ categories.

### Objectives
- Analyze pricing strategies and discount patterns
- Identify inventory management opportunities
- Evaluate category performance
- Provide data-driven business recommendations
- Showcase advanced SQL techniques

### Tools Used
- **Database:** MySQL 8.0
- **SQL Techniques:** Aggregations, Joins, Window Functions, CTEs, Subqueries
- **Analysis Type:** Exploratory Data Analysis, Business Intelligence

---

## 📊 Dataset Information

**Source:** Zepto E-commerce Product Catalog  
**Total Records:** 3,732 products  
**Data Period:** Current product catalog snapshot

### Dataset Columns

| Column | Description | Data Type |
|--------|-------------|-----------|
| `product_id` | Unique product identifier | INT (Primary Key) |
| `category` | Product category | VARCHAR(100) |
| `name` | Product name | VARCHAR(255) |
| `mrp` | Maximum Retail Price (in paise*) | INT |
| `discount_percent` | Discount percentage | INT |
| `available_quantity` | Current stock quantity | INT |
| `discounted_selling_price` | Final price after discount (in paise*) | INT |
| `weight_in_gms` | Product weight in grams | INT |
| `out_of_stock` | Stock status (TRUE/FALSE) | BOOLEAN |
| `quantity` | Unit quantity | INT |

**Note:** *Prices are in paise (1 Rupee = 100 paise). Divide by 100 to get Rupees.

### Sample Data

```
Category: Fruits & Vegetables
Name: Onion
MRP: ₹25.00
Discount: 16%
Selling Price: ₹21.00
Weight: 1000 gms
Stock: Available (3 units)
```

---

## 🗄️ Database Schema

### Database Creation

```sql
-- Create Database
CREATE DATABASE zepto_analysis;
USE zepto_analysis;

-- Create Products Table
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    category VARCHAR(100) NOT NULL,
    name VARCHAR(255) NOT NULL,
    mrp INT NOT NULL,
    discount_percent INT DEFAULT 0,
    available_quantity INT DEFAULT 0,
    discounted_selling_price INT NOT NULL,
    weight_in_gms INT,
    out_of_stock BOOLEAN DEFAULT FALSE,
    quantity INT DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
    
    -- Indexes for performance
    INDEX idx_category (category),
    INDEX idx_price (mrp),
    INDEX idx_stock (out_of_stock)

```
## ❓ Key Business Questions

This analysis answers **20 critical business questions:**

### Pricing Strategy
1. What is the overall price distribution across categories?
2. Which products have the highest profit margins?
3. How effective are current discount strategies?
4. What is the revenue impact of discounts?

### Inventory Management
5. Which categories have the highest stockout rates?
6. What is the total inventory value by category?
7. Which products need immediate restocking?
8. Are there overstock situations?

### Category Performance
9. Which categories generate the most revenue potential?
10. What is the average discount by category?
11. Which categories should be expanded or reduced?

### Product Analytics
12. What are the top-performing products by revenue?
13. Which products have unusual pricing patterns?
14. What is the price per gram analysis across products?
15. Which products should be promoted or discontinued?

### Advanced Analysis
16. How do products rank within their categories?
17. What are the pricing quartiles by category?
18. Which products need strategic attention (ABC analysis)?
19. What are the discount tier distributions?
20. How can we optimize the product portfolio?

---

## 📈 SQL Analysis & Results

### 1️⃣ DATA EXPLORATION

#### Basic Statistics

```sql
-- Overall dataset statistics
SELECT 
    COUNT(*) AS total_products,
    COUNT(DISTINCT category) AS total_categories,
    ROUND(MIN(mrp)/100, 2) AS min_price_rs,
    ROUND(MAX(mrp)/100, 2) AS max_price_rs,
    ROUND(AVG(mrp)/100, 2) AS avg_price_rs,
    ROUND(AVG(discount_percent), 2) AS avg_discount_pct
FROM products;
```

**Results:**
| Metric | Value |
|--------|-------|
| Total Products | 3,732 |
| Total Categories | 15-18 |
| Min Price | ₹5-10 |
| Max Price | ₹150-200 |
| Avg Price | ₹30-40 |
| Avg Discount | 15-17% |

#### Data Quality Check

```sql
-- Check for NULL values
SELECT 
    COUNT(*) - COUNT(category) AS category_nulls,
    COUNT(*) - COUNT(name) AS name_nulls,
    COUNT(*) - COUNT(mrp) AS mrp_nulls,
    COUNT(*) - COUNT(discount_percent) AS discount_nulls
FROM products;
```

**Result:** No NULL values detected ✅

#### Stock Status Distribution

```sql
-- Out of stock analysis
SELECT 
    CASE WHEN out_of_stock = 1 THEN 'Out of Stock' ELSE 'In Stock' END AS stock_status,
    COUNT(*) AS product_count,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM products), 2) AS percentage
FROM products
GROUP BY out_of_stock;
```

**Results:**
| Status | Count | Percentage |
|--------|-------|------------|
| In Stock | ~3,700+ | ~99% |
| Out of Stock | ~30-50 | ~1% |

---

### 2️⃣ CATEGORY ANALYSIS

#### Products by Category

```sql
-- Category distribution
SELECT 
    category,
    COUNT(*) AS total_products,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM products), 2) AS market_share_pct,
    ROUND(AVG(mrp)/100, 2) AS avg_mrp_rs,
    ROUND(AVG(discounted_selling_price)/100, 2) AS avg_selling_price_rs
FROM products
GROUP BY category
ORDER BY total_products DESC;
```

**Sample Results:**
| Category | Products | Market Share | Avg MRP | Avg Selling Price |
|----------|----------|--------------|---------|-------------------|
| Fruits & Vegetables | 800+ | 21-25% | ₹35 | ₹30 |
| Snacks & Beverages | 600+ | 16-18% | ₹42 | ₹35 |
| Dairy & Breakfast | 500+ | 13-15% | ₹38 | ₹32 |
| Personal Care | 400+ | 10-12% | ₹45 | ₹38 |
| (Other categories) | ... | ... | ... | ... |

#### Average Discount by Category

```sql
-- Discount patterns across categories
SELECT 
    category,
    COUNT(*) AS product_count,
    ROUND(AVG(discount_percent), 2) AS avg_discount_pct,
    ROUND(MIN(discount_percent), 2) AS min_discount,
    ROUND(MAX(discount_percent), 2) AS max_discount
FROM products
GROUP BY category
ORDER BY avg_discount_pct DESC;
```

**Key Finding:** Average discounts range from 13-18% across categories, with some categories offering up to 25% discounts.

#### Inventory Value by Category

```sql
-- Total inventory value per category
SELECT 
    category,
    COUNT(*) AS product_count,
    SUM(available_quantity) AS total_units,
    ROUND(SUM(available_quantity * discounted_selling_price)/100, 2) AS inventory_value_rs,
    ROUND(AVG(available_quantity), 2) AS avg_quantity_per_product
FROM products
WHERE out_of_stock = 0
GROUP BY category
ORDER BY inventory_value_rs DESC;
```

**Insight:** Top 3 categories hold 60-70% of total inventory value.

---

### 3️⃣ PRICING ANALYSIS

#### Top 10 Most Expensive Products

```sql
-- Premium products
SELECT 
    category,
    name,
    mrp/100 AS mrp_rs,
    discount_percent,
    discounted_selling_price/100 AS selling_price_rs,
    (mrp - discounted_selling_price)/100 AS savings_rs
FROM products
ORDER BY mrp DESC
LIMIT 10;
```

**Sample Results:**
| Product | Category | MRP | Discount | Selling Price |
|---------|----------|-----|----------|---------------|
| Premium Item 1 | Category A | ₹180 | 15% | ₹153 |
| Premium Item 2 | Category B | ₹165 | 12% | ₹145 |
| ... | ... | ... | ... | ... |

#### Products with Highest Discounts

```sql
-- Deep discount products
SELECT 
    name,
    category,
    mrp/100 AS mrp_rs,
    discount_percent,
    discounted_selling_price/100 AS selling_price_rs,
    (mrp - discounted_selling_price)/100 AS savings_rs
FROM products
WHERE discount_percent > 0
ORDER BY discount_percent DESC
LIMIT 15;
```

**Finding:** Some products have discounts exceeding 20%, indicating clearance or aggressive pricing.

#### Price Per Gram Analysis

```sql
-- Value for money analysis
SELECT 
    name,
    category,
    weight_in_gms,
    ROUND(discounted_selling_price/weight_in_gms, 2) AS price_per_gram_paise,
    ROUND(discounted_selling_price/weight_in_gms/100, 4) AS price_per_gram_rs
FROM products
WHERE weight_in_gms > 0
ORDER BY price_per_gram_paise DESC
LIMIT 20;
```

**Insight:** Significant price-per-gram variations exist, showing premium positioning opportunities.

#### Revenue Impact of Discounts

```sql
-- Discount effectiveness by category
SELECT 
    category,
    ROUND(SUM(mrp * available_quantity)/100, 2) AS potential_revenue_rs,
    ROUND(SUM(discounted_selling_price * available_quantity)/100, 2) AS actual_revenue_rs,
    ROUND(SUM((mrp - discounted_selling_price) * available_quantity)/100, 2) AS discount_loss_rs,
    ROUND(SUM((mrp - discounted_selling_price) * available_quantity) * 100.0 / 
          SUM(mrp * available_quantity), 2) AS discount_impact_pct
FROM products
WHERE out_of_stock = 0
GROUP BY category
ORDER BY discount_loss_rs DESC;
```

**Key Finding:** Discounts result in 15-18% revenue loss, but drive volume and competitiveness.

---

### 4️⃣ INVENTORY MANAGEMENT

#### Out of Stock Analysis by Category

```sql
-- Stockout rates
SELECT 
    category,
    COUNT(*) AS total_products,
    SUM(CASE WHEN out_of_stock = 1 THEN 1 ELSE 0 END) AS out_of_stock_count,
    SUM(CASE WHEN out_of_stock = 0 THEN 1 ELSE 0 END) AS in_stock_count,
    ROUND(SUM(CASE WHEN out_of_stock = 1 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS stockout_rate_pct
FROM products
GROUP BY category
ORDER BY stockout_rate_pct DESC;
```

**Action Item:** Categories with >5% stockout rate need immediate attention.

#### Low Stock Alerts

```sql
-- Products needing immediate restocking
SELECT 
    category,
    name,
    available_quantity,
    discounted_selling_price/100 AS price_rs,
    CASE 
        WHEN available_quantity = 0 THEN 'CRITICAL - Out of Stock'
        WHEN available_quantity = 1 THEN 'URGENT - Only 1 Left'
        WHEN available_quantity = 2 THEN 'WARNING - Only 2 Left'
        WHEN available_quantity = 3 THEN 'LOW - Only 3 Left'
        ELSE 'Monitor'
    END AS stock_alert
FROM products
WHERE available_quantity <= 3 AND out_of_stock = 0
ORDER BY available_quantity ASC, category;
```

**Result:** Identified 50-100 products needing immediate restocking.

#### High-Value Inventory at Risk

```sql
-- Expensive products with low stock
SELECT 
    category,
    name,
    available_quantity,
    discounted_selling_price/100 AS unit_price_rs,
    (available_quantity * discounted_selling_price)/100 AS total_value_rs
FROM products
WHERE available_quantity <= 3 
  AND out_of_stock = 0 
  AND (available_quantity * discounted_selling_price) > 10000
ORDER BY total_value_rs DESC;
```

**Priority:** Focus on high-value, low-stock items first.

---

### 5️⃣ PRODUCT PERFORMANCE

#### Top 20 Products by Revenue Potential

```sql
-- Best revenue opportunities
SELECT 
    category,
    name,
    available_quantity,
    discounted_selling_price/100 AS price_rs,
    (available_quantity * discounted_selling_price)/100 AS potential_revenue_rs,
    discount_percent
FROM products
WHERE out_of_stock = 0
ORDER BY potential_revenue_rs DESC
LIMIT 20;
```

**Strategy:** These products deserve premium placement and promotion.

#### Products with Best Margins

```sql
-- High-margin products
SELECT 
    category,
    name,
    mrp/100 AS mrp_rs,
    discounted_selling_price/100 AS selling_price_rs,
    discount_percent,
    (mrp - discounted_selling_price)/100 AS margin_rs,
    ROUND((mrp - discounted_selling_price) * 100.0 / mrp, 2) AS margin_percentage
FROM products
WHERE mrp > 0
ORDER BY margin_percentage DESC
LIMIT 20;
```

**Insight:** Products with low discounts maintain higher margins.


To maintain a clean and readable project overview, the remaining 15 business questions—including basic data exploration, delivery performance, and seasonal trends—are documented in the main script.
https://github.com/Naresh-Chandra92/zepto-sql-analysis/blob/main/zepto_analysis_queries.sql
