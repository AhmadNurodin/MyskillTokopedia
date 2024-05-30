# E-COMMERCE SALES ANALYSIST

### Project Overview
This final project data analysis project aims to provide insight into the sales performance of an e-commerce company over past year. By analyzing various aspects of the sales data, we seek to identify trends, make data driven recommendations, and gain a deeper understanding of the company's performance

### Relational Database Schema

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
  payment_detail as pd on od.payment_id = pd.id
```
### Tools 

- Excel - Data Cleaning
- Google Bigquery - Data Analysis

### Data Cleaning/Preparation

In the initial data preparation phase, we performed the following task:
1. Data loading and inspection.
2. Handling missing values.
3. Data cleaning and formatting.

### Find Top Transaction in 2021, by month

```sql
SELECT
extract (month from order_date) as bulan,
sum(after_discount) as total_transaksi
FROM `Final_project.order_detail`
WHERE extract (year from order_date) = 2021 and is_valid = 1
GROUP BY 1
ORDER BY 2 DESC
limit 1 
```


## Find Top Transaction in 2022 By Product Category

```sql
SELECT
kategori
total_transaksi_2021
total_transaksi_2022,
CASE WHEN total_transaksi_2022 = total_transaksi_2021 < 0 then 'Penurunan'
ELSE 'Kenaikan'
END AS perubahan_transaksi
FROM(
SELECT
sd.category AS kategori,
SUM(CASE WHEN EXTRACT(year from od.order_date)= 2021 then after_discount end) as total_transaksi_2021,
SUM(CASE WHEN EXTRACT(year from od.order_date)= 2022 then after discount end) as total_transaksi_2022,
from `Final_project.order_detail` AS od
LEFT JOIN `Final_project.sku_detail` AS sd
ON od.sku_id = sd.id
WHERE EXTRACT(year from od.order_date) IN (2021,2022) AND od.is_valid = 1
Group BY 1
)
ORDER BY 4
```


## By Category, compare transaction between 2021 and 2022, find which groups experienced increases and decreases

```sql
SELECT
kategori
total_transaksi_2021
total_transaksi_2022,
CASE WHEN total_transaksi_2022 = total_transaksi_2021 < 0 then 'Penurunan'
ELSE 'Kenaikan'
END AS perubahan_transaksi
FROM(
SELECT
sd.category AS kategori,
SUM(CASE WHEN EXTRACT(year from od.order_date)= 2021 then after_discount end) as total_transaksi_2021,
SUM(CASE WHEN EXTRACT(year from od.order_date)= 2022 then after discount end) as total_transaksi_2022,
from `Final_project.order_detail` AS od
LEFT JOIN `Final_project.sku_detail` AS sd
ON od.sku_id = sd.id
WHERE EXTRACT(year from od.order_date) IN (2021,2022) AND od.is_valid = 1
Group BY 1
)
ORDER BY 4
```


## Base on order, find top 5 payment methode mostly used

```sql
SELECT
pd.payment_method as metode
COUNT(DISTINCT od.id) as total_unique_order
FROM `Final_project.order_detail` od
LEFT JOIN `Final_project.payment_detail` pd
ON od.payment_id = pd.id
WHERE EXTRACT(year from od.order_date) = 2022 and od.is_valid = 1
GROUP BY 1
ORDER BY 2 DESC
Limit 5
```


## Sort by transaction value on this group of product (Samsung, Apple, Sony, Huawei, Lenovo)

```sql
WITH Top_Produk as (
SELECT
CASE
WHEN Lower(sd.sku_name) like `%samsung%` then `samsung`
WHEN lower(sd.sku_name) like `%apple%` or lower (sd.sku_name)
like `%iphone%` or lower (sd.sku_name) like `%macbook%` then `apple`
WHEN lower (sd.sku_name) like `%sony%` then `sony`
WHEN lower (sd.sku_name) like `%huawei%` then `huawei`
WHEN lower (sd.sku_nname) like `%lenovo%` then `lenovo`
END AS nama_produk
SUM(after_discount) as nilai_transaksi
FROM `Final_project.order_detail` od
LEFT JOIN `Final_project.sku_detail` sd
ON od.sku_id = sd.id
WHERE is_valid = 1
GROUP BY 1
ORDER BY 2 DESC
)
SELECT*
FROM Top_Produk
WHERE nama_produk is not null
```

