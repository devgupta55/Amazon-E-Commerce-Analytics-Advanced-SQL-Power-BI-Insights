# Amazon-E-Commerce-Analytics-Advanced-SQL-Power-BI-Insights
# 🚀 Amazon E-Commerce Analytics: SQL + Power BI

<img src = "https://github.com/user-attachments/assets/3c337b70-3755-411b-a130-bdcbb174dcc2" width = "600" alt = "AmazonImg">

## 📌 Project Overview

I have worked on analyzing a dataset of over 20,000 sales records from an Amazon-like e-commerce platform. This project involves extensive querying of customer behavior, product performance, and sales trends using PostgreSQL. Through this project, I have tackled various SQL problems, including revenue analysis, customer segmentation, and inventory management.

The project also focuses on data cleaning, handling null values, and solving real-world business problems using structured queries.

An ERD diagram is included to visually represent the database schema and relationships between tables.

This end-to-end data analysis project explores Amazon's e-commerce performance using:
- **SQL** for advanced data transformation (PostgreSQL)
- **Power BI** for interactive visualizations
- **CSV datasets** containing sales, products, and customer data

## 🛠️ Technical Stack
| Component | Technology |
|-----------|------------|
| **Database** | PostgreSQL 14 |
| **ETL** | CSV Import |
| **Analysis** | Advanced SQL (CTEs, Window Functions) |
| **Visualization** | Power BI |

### ✏️Entity Relationship Diagram (ERD)
<img src="https://github.com/user-attachments/assets/6d1be68f-6a24-4996-bb58-3e35411a7c08" width="600" alt="ZomatoERD">

## 🎯 Objective

The primary objective of this project is to showcase SQL proficiency through complex queries that address real-world e-commerce business challenges. The analysis covers various aspects of e-commerce operations, including:
- Customer behavior
- Sales trends
- Inventory management
- Payment and shipping analysis
- Forecasting and product performance

## 📋 Core Research Questions

### 📌 **Advanced Questions & Analysis**

#### **Q. Top Performing Sellers**  
*Find the top 5 sellers based on total sales value. Include both successful and failed orders, and display their percentage of successful orders.*
```sql
with sellersales as(
	select
    ss.seller_id,
		ss.seller_name,
		sum(price_per_unit*quantity) as TotalSalesValue,
		count(os.order_id) as TotalOrders,
		count(case when py.payment_status = 'Payment Successed' then 1 end) as SuccessfulOrders
	from orders os
		inner join order_items as osi
		on os.order_id = osi.order_id
		inner join sellers ss
		on ss.seller_id = os.seller_id
		inner join payments py
		on py.order_id = os.order_id
	group by 1,2
),
sellerrank as(
	select
		seller_id,
		seller_name,
		TotalSalesValue,
		SuccessfulOrders,
		round(100.0*SuccessfulOrders/nullif(TotalOrders,0),2) as SuccessRate,
		rank() over (order by TotalSalesValue desc) as SalesRank
	from sellersales
)
select
	seller_id,
	seller_name,
	TotalSalesValue,
	SuccessRate 
from sellerrank
where SalesRank<=5
order by TotalSalesValue desc
```
📺**Output**<br>
![image](https://github.com/user-attachments/assets/721fa5bf-73d3-4553-8ed8-983c46d7f749)

#### **Q. Product Profit Margin**  
*Calculate the profit margin for each product (difference between price and cost of goods sold). Rank products by their profit margin, showing highest to lowest.*
```sql
select
	product_id,
	price,
	cogs,
	round((price-cogs)::numeric,2) as Profit,
	round((100.0*(price-cogs)/price)::numeric,2) as ProfitMargin,
	dense_rank() over (order by round((100.0*(price-cogs)/price)::numeric,2) desc) as Rank
from products
```
📺**Output** (top 5 rows)<br>
![image](https://github.com/user-attachments/assets/f6721b61-4537-4eba-8fa0-ce1aede3736a)


#### **Q. Most Returned Products**  
*Query the top 10 products by the number of returns. Display the return rate as a percentage of total units sold for each product.*
```sql
with totalproductssold as(
	select
		ps.product_id,
		ps.product_name,
		count(*) as TotalProdsSold
	from orders os 
		inner join order_items osi
		on os.order_id = osi.order_id
		inner join products ps
		on ps.product_id = osi.product_id
	group by 1,2
),
returnedprods as (
	select
		ps.product_id,
		count(*) as ReturnedProds
	from orders os
		inner join order_items osi
		on os.order_id = osi.order_id
		inner join products ps
		on ps.product_id = osi.product_id
	where order_status = 'Returned'
	group by 1
)

select 
	tps.product_id,
	TotalProdsSold,
	coalesce(ReturnedProds,0) as ReturnedProds,
	round((100*coalesce(ReturnedProds,0)/nullif(TotalProdsSold,0)),2) as ReturnPercentage
from totalproductssold as tps
	left join returnedprods rps
	on tps.product_id = rps.product_id
order by ReturnedProds desc

limit 10
```
📺**Output**<br>
![image](https://github.com/user-attachments/assets/eee1faec-17d0-4c90-9748-c1c74483ddbc)

#### **Q. Revenue by Shipping Provider**  
*Calculate the total revenue handled by each shipping provider. Include the total number of orders handled and the average delivery time.*
```sql
select
	ss.seller_id,
	ss.seller_name,
	count(os.order_id) as TotalOrders,
	sum(price_per_unit * quantity) as TotalRevenue,
	round(avg (shipping_date - order_date),2) as AvgDeliveryTime
from orders os
	inner join order_items osi
	on os.order_id = osi.order_id
	inner join sellers ss
	on ss.seller_id = os.seller_id
	left join shipping sg
	on sg.order_id = os.order_id
group by 1,2
order by TotalRevenue Desc
```
📺**Output** (Top 5 Rows)<br>
![image](https://github.com/user-attachments/assets/4e02710c-f919-44b7-9e9c-35e4cb3c0fb6)


#### **Q. Declining Product Revenue**  
*Find the top 10 products with the highest decreasing revenue ratio compared to last year (2022) and current year (2023).*
```sql
with CTE as(
	select
		ps.product_id,
		ps.product_name,
		cg.category_name,
		sum(case when extract(year from os.order_date) = 2022 then price_per_unit*quantity end) as TotalRevenue_22,
		sum(case when extract(year from os.order_date) = 2023 then price_per_unit*quantity end) as TotalRevenue_23
	from orders os
		inner join order_items osi
		on osi.order_id = os.order_id
		inner join products ps
		on ps.product_id = osi.product_id
		inner join category cg
		on cg.category_id = ps.category_id
	group by 1,2,3
)
select
	product_id,
	product_name,
	category_name,
	TotalRevenue_22,
	coalesce(TotalRevenue_23,0) as TotalRevenue_23,
	round((100*(coalesce(TotalRevenue_23,0)-TotalRevenue_22)/nullif(TotalRevenue_22,0))::numeric,2) as DecreaseRatio
from CTE
order by 6 asc

limit 10
```
📺**Output**<br>
![image](https://github.com/user-attachments/assets/c4bf807a-940e-4d0c-a6bc-abb3ac45f8fa)


✔️### For more queries (Easy → Medium → Hard), [click here](./Advanced_Queries_Amazon.sql)

---

## <img src="https://github.com/microsoft/PowerBI-Icons/blob/main/SVG/Power-BI.svg" width="24" style="margin-bottom: -10 px"> Power BI Visuals
<img src="https://github.com/user-attachments/assets/f1d21b02-571c-422c-9fdb-f30ff647d533" width="450" alt="AmazonBI1"> <img src="https://github.com/user-attachments/assets/a487fd31-85b8-4f19-aad9-64b8eab81650" width="450" alt="AmazonBI2"> 
<img src="https://github.com/user-attachments/assets/08e95dd9-b52d-4edc-8037-a72a07e17595" width="450" alt="AmazonBI3">

[View the Amazon PowerBI Visuals](PowerBIAnalysis_Amazon.pbix)

## 🏁 Conclusion
This advanced SQL project successfully demonstrates my ability to solve real-world e-commerce problems using structured queries. From improving customer retention to optimizing inventory and logistics, the project provides valuable insights into operational challenges and solutions.
By completing this project, I have gained a deeper understanding of how SQL can be used to tackle complex data problems and drive business decision-making.

## 🔒 Data Confidentiality
*Due to confidentiality agreements, the original datasets cannot be shared publicly. Sample queries demonstrate the analytical approach using synthetic/scrambled data.*

