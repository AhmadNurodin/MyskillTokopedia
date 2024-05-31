# E-COMMERCE SALES ANALYSIST

## Project Overview
This final project data analysis project aims to provide insight into the sales performance of an e-commerce company over past year. By analyzing various aspects of the sales data, we seek to identify trends, make data driven recommendations, and gain a deeper understanding of the company's performance

## Relational Database Schema

![schema table](https://github.com/AhmadNurodin/Myskill-Tokopedia-SQL/assets/161307267/c4ba7b0c-2e8e-4602-a36b-ead7708622a7)


### Data Sources
customer_detail [Download here](https://drive.google.com/file/d/1iRonHkbVSz9p1PtHS8lPFWSHhg7knyVq/view?usp=sharing)        
order_detail [Download here](https://drive.google.com/file/d/1asKIIjJLCeYVlL3E3GrFk5uRbVlRntTj/view?usp=sharing)        
payment_detail [Download here](https://drive.google.com/file/d/1NEVvGR72B22191DP5zsd_P6_JZQLwzJd/view?usp=sharing)        
sku_detail [Download here](https://drive.google.com/file/d/1UoJdsQ7kgpw8wi8fQFH78cmnEBxjbHkk/view?usp=sharing)     
        Merging Data Sets as fulldata [Download here](https://drive.google.com/file/d/1RCN023Ljm_fuucM9YDxGYO0JOnR9uBb0/view?usp=sharing)        
Merging Data Sets Code
```sql
SELECT
  od.id,
  od.customer_id,
  sd.sku_name,
  sd.base_price,
  sd.cogs,
  sd.category,
  cd.registered_date,
  od.order_date,
  od.sku_id,
  od.price,
  od.qty_ordered,
  od.before_discount,
  od.discount_amount,
  od.is_gross,
  od.is_valid,
  od.is_net,
  od.payment_id,
  pd.payment_method
  FROM 
  order_detail AS od
  LEFT JOIN 
  customer_detail AS cd ON od.customer_id = cd.id
  LEFT JOIN
  sku_detail as sd ON od.sku_id = sd.id
  LEFT JOIN
  payment_detail as pd on od.payment_id = pd.id;
```
## Tools 

- Excel - Data Cleaning
- Postgre SQL - Data Analysis

## Data Cleaning/Preparation

In the initial data preparation phase, we performed the following task:
1. Data loading and inspection.
2. Handling missing values.
3. Data cleaning and formatting.

## Exploratory Data Analysis

Exploratory Data Analysis involved exploring the sales data to answer key questions, such as:

- Find Top Transaction in 2021, by month
- Find Top Transaction in 2022 By Product Category
- By Category, compare transaction between 2021 and 2022, find which groups experienced increases and decreases
- Base on order, find top 5 payment methode mostly used
- Sort by transaction value on this group of product (Samsung, Apple, Sony, Huawei, Lenovo)

## Data Analysis

### Find Top Transaction in 2021, by month

```sql
SELECT
extract (month from order_date) as bulan,
sum(after_discount) as total_transaksi
FROM order_detail
WHERE extract (year from order_date) = 2021 and is_valid = 1
GROUP BY 1
ORDER BY 2 DESC
limit 1;
```

### Find Top Transaction in 2022 By Product Category

```sql
SELECT 
EXTRACT(
YEAR FROM od.order_date) as Year, 
sd.category, 
SUM(ROUND(od.after_discount)) as Total_transaction  
FROM 
order_detail as od
JOIN sku_detail as sd ON sd.Id = od.sku_id
WHERE 
EXTRACT(YEAR FROM od.order_date) = 2022
AND od.is_valid = 1
GROUP BY 
EXTRACT(
YEAR FROM od.order_date), 
sd.category
ORDER BY 
Total_transaction 
DESC
Limit 1;
```

### By Category, compare transaction between 2021 and 2022, find which groups experienced increases and decreases

```sql
SELECT
kategori,
total_transaksi_2021,
total_transaksi_2022,
CASE 
WHEN total_transaksi_2022 < total_transaksi_2021 THEN 'Penurunan'
ELSE 'Kenaikan'
END AS perubahan_transaksi
FROM (
SELECT
sd.category AS kategori,
SUM(CASE WHEN EXTRACT(year FROM od.order_date) = 2021 THEN od.after_discount ELSE 0 END) AS total_transaksi_2021,
SUM(CASE WHEN EXTRACT(year FROM od.order_date) = 2022 THEN od.after_discount ELSE 0 END) AS total_transaksi_2022
FROM 
order_detail AS od
LEFT JOIN 
sku_detail AS sd ON od.sku_id = sd.id
WHERE 
EXTRACT(year FROM od.order_date) IN (2021, 2022) AND od.is_valid = 1
GROUP BY 
sd.category
) AS subquery
ORDER BY 
    CASE 
        WHEN total_transaksi_2022 < total_transaksi_2021 THEN 'Penurunan'
        ELSE 'Kenaikan'
    END;

```

### Base on order, find top 5 payment methode mostly used

```sql
SELECT
    pd.payment_method as metode,
    COUNT(DISTINCT od.id) as total_unique_order
FROM 
    order_detail od
LEFT JOIN 
    payment_detail pd
ON 
    od.payment_id = pd.id
WHERE 
    EXTRACT(year FROM od.order_date) = 2022 
    AND od.is_valid = 1
GROUP BY 
    pd.payment_method
ORDER BY 
    total_unique_order DESC
LIMIT 5;
```

### Sort by transaction value on this group of product (Samsung, Apple, Sony, Huawei, Lenovo)

```sql
WITH Top_Produk as (
    SELECT
        CASE
            WHEN LOWER(sd.sku_name) LIKE '%samsung%' THEN 'samsung'
            WHEN LOWER(sd.sku_name) LIKE '%apple%' OR LOWER(sd.sku_name) LIKE '%iphone%' OR LOWER(sd.sku_name) LIKE '%macbook%' THEN 'apple'
            WHEN LOWER(sd.sku_name) LIKE '%sony%' THEN 'sony'
            WHEN LOWER(sd.sku_name) LIKE '%huawei%' THEN 'huawei'
            WHEN LOWER(sd.sku_name) LIKE '%lenovo%' THEN 'lenovo'
        END AS nama_produk,
        SUM(after_discount) as nilai_transaksi
    FROM 
        order_detail od
    LEFT JOIN 
        sku_detail sd
    ON 
        od.sku_id = sd.id
    WHERE 
        is_valid = 1
    GROUP BY 
        nama_produk
    ORDER BY 
        nilai_transaksi DESC
)
SELECT *
FROM Top_Produk
WHERE nama_produk IS NOT NULL;
```

## Results
The analysis results are sumarized as follows:
1. The company's sales peak in August with total transaction value 227862744.0, which is the largest in 2021.
2. Product Category Mobiles & Tablets is the top transaction in 2022.
3. The company's sales have been a lot of increasing over the past year, there are Appliances, Computing, Superstore, Mobile & Tablets, Health & Sport, Women Fashion, Entertainment, Home & Living, Men Fashion, Beauty & Grooming, School & Education, Soghaat, Kids & Baby categories increase and there was a decreases transaction value from 2021 to 2022 there are Others and Books.
4. The top 5 most popular payment methods used during 2022 are COD, Payaxis, CustomerCredit, Easypay, and Jazzwallet.
5. The following is the order of transaction values ​​based on product are samsung, apple, sony, huawei and lenovo.


## Recommendations
Based on the analysis, we recommend the following actions:
1. 
