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
        Merging Data Sets [Download here](https://drive.google.com/file/d/1RCN023Ljm_fuucM9YDxGYO0JOnR9uBb0/view?usp=sharing)        
Code
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
1. Data loading and inspaction.
2. handling missing values.
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
SELECT Year(O.order_date) as Year, S.category, sum(round(O.after_discount)) as Total_transaction  
FROM order_detail as O
Join sku_detail as S on (S.sku_Id = O.Id_sku)
where Year(order_date) = 2022
and O.is_valid = 1
Group by S.category
order by Total_transaction desc
```


## By Category, compare transaction between 2021 and 2022, find which groups experienced increases and decreases

```sql
WITH CT2021 as (
    SELECT Year(O.order_date) as Year, S.category, sum(round(O.after_discount)) as Total_transaction2021  
    FROM order_detail as O
    Join sku_detail as S on (S.sku_Id = O.Id_sku)
    where Year(order_date) = 2021
    and O.is_valid = 1
    Group by S.category
    ),
    CT2022 as (
    SELECT Year(O.order_date) as Year, S.category, sum(round(O.after_discount)) as Total_transaction2022  
    FROM order_detail as O
    Join sku_detail as S on (S.sku_Id = O.Id_sku)
    where Year(order_date) = 2022
    and O.is_valid = 1
    Group by S.category
    )
SELECT  CT2021.Category, Total_transaction2021, Total_transaction2022,
        ((Total_transaction2022-Total_transaction2021)/Total_transaction2021)*100 as GrowPercentage,
        CASE 
            When Total_transaction2022 > Total_transaction2021 Then 'Increase'
            When Total_transaction2022 < Total_transaction2021 Then 'Decrease'
        END as Expl
FROM CT2021
Join CT2022 on (CT2022.category = CT2021.category)
Order by GrowPercentage desc;
```


## Base on order, find top 5 payment methode mostly used

```sql
SELECT  O.payment_Id, P.payment_method, count(DISTINCT O.Id_order) as Total_Order 
FROM order_detail as O
Join payment_detail as P on (P.payment_Id = O.payment_Id)
WHERE O.is_valid = 1
AND Year(order_date) = 2022
GROUP by P.payment_method
ORDER by Total_Order desc
Limit 5;
```


## sort by transaction value on this group of product (Samsung, Apple, Sony, Huawei, Lenovo)

```sql
WITH Gadget_Order as (
    SELECT  DISTINCT O.Id_order as ID_Order, 
            S.sku_name as Product_name, 'Samsung' as Product_brand, 
            O.qty_ordered as Total_order, O.after_discount as Total_Transaction   
    FROM order_detail as O
    Join sku_detail as S on (S.sku_Id = O.Id_sku)
    WHERE O.is_valid = 1 and S.sku_name like '%samsung%'
        UNION
    SELECT  DISTINCT O.Id_order as ID_Order, 
            S.sku_name as Product_name, 'Apple' as Product_brand, 
            O.qty_ordered as Total_order, O.after_discount as Total_Transaction   
    FROM order_detail as O
    Join sku_detail as S on (S.sku_Id = O.Id_sku)
    WHERE O.is_valid = 1 and S.sku_name like '%apple%' or S.sku_name like '%iphone%' or S.sku_name like '%imac%'
        UNION
    SELECT  DISTINCT O.Id_order as ID_Order, 
            S.sku_name as Product_name, 'Sony' as Product_brand, 
            O.qty_ordered as Total_order, O.after_discount as Total_Transaction   
    FROM order_detail as O
    Join sku_detail as S on (S.sku_Id = O.Id_sku)
    WHERE O.is_valid = 1 and S.sku_name like '%sony%'
        UNION
    SELECT  DISTINCT O.Id_order as ID_Order, 
            S.sku_name as Product_name, 'Huawei' as Product_brand, 
            O.qty_ordered as Total_order, O.after_discount as Total_Transaction   
    FROM order_detail as O
    Join sku_detail as S on (S.sku_Id = O.Id_sku)
    WHERE O.is_valid = 1 and S.sku_name like '%huawei%'
        UNION
    SELECT  DISTINCT O.Id_order as ID_Order, 
            S.sku_name as Product_name, 'Lenovo' as Product_brand, 
            O.qty_ordered as Total_order, O.after_discount as Total_Transaction   
    FROM order_detail as O
    Join sku_detail as S on (S.sku_Id = O.Id_sku)
    WHERE O.is_valid = 1 and S.sku_name like '%lenovo%')
SELECT  Product_brand,
        sum(round(Total_order)) as Total_order, 
        sum(round(Total_Transaction)) as Total_Transaction 
FROM Gadget_Order
GROUP by Product_brand
ORDER by Total_Transaction Desc;
```
## sort by transaction value on this group of product (Samsung, Apple, Sony, Huawei, Lenovo) with if condition

``` sql
WITH Gadget_Order as (
    SELECT  DISTINCT O.Id_order, S.sku_name,
        CASE
            When S.sku_name like '%samsung%' Then 'Samsung'
            When S.sku_name like '%sony%' Then 'Sony'
            When S.sku_name like '%huawei%' Then 'Huawei'
            When S.sku_name like '%lenovo%' Then 'Lenovo'
            when Lower(S.sku_name) like '%apple%' or lower(S.sku_name) like '%iphone%' Then 'Apple'
        END as Product_brand,
        O.qty_ordered as Total_order,
        O.after_discount as Total_Transaction
    FROM order_detail as O
    Join sku_detail as S on (S.sku_Id = O.Id_sku)
    WHERE O.is_valid = 1)
SELECT  Product_brand,
        sum(round(Total_order)) as Total_order, 
        sum(round(Total_Transaction)) as Total_Transaction
FROM Gadget_Order
WHERE Product_brand is not null
GROUP by Product_brand
ORDER by Total_Transaction Desc;
```
