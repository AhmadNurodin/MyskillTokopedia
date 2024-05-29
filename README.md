# E-COMMERCE SALES ANALYSIST

## Find Top Transaction in 2021, by month

SELECT  Year(order_date) as Year, Monthname(order_date) as Month, 
        sum(round(after_discount)) as Total_transaction
FROM order_detail
where Year(order_date) = 2021
and is_valid = 1
Group by Monthname(order_date)
order by Total_transaction desc;


## Find Top Transaction in 2022 By Product Category

SELECT Year(O.order_date) as Year, S.category, sum(round(O.after_discount)) as Total_transaction  
FROM order_detail as O
Join sku_detail as S on (S.sku_Id = O.Id_sku)
where Year(order_date) = 2022
and O.is_valid = 1
Group by S.category
order by Total_transaction desc;


## By Category, compare transaction between 2021 and 2022, find which groups experienced increases and decreases

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


## Base on order, find top 5 payment methode mostly used

SELECT  O.payment_Id, P.payment_method, count(DISTINCT O.Id_order) as Total_Order 
FROM order_detail as O
Join payment_detail as P on (P.payment_Id = O.payment_Id)
WHERE O.is_valid = 1
AND Year(order_date) = 2022
GROUP by P.payment_method
ORDER by Total_Order desc
Limit 5;


## sort by transaction value on this group of product (Samsung, Apple, Sony, Huawei, Lenovo)

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

## sort by transaction value on this group of product (Samsung, Apple, Sony, Huawei, Lenovo) with if condition

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
