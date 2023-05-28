# Coffee_Factory_Ecommerce_data_analysis_sql

Date source are from Udemy course.
 

# Objectives
Maven Fuzzy Factory is a growing eCommerce company that specializes in selling handmade crafts and accessories. As the company expands, it is becoming increasingly important to analyze and interpret data to make informed decisions.
As eCommerce Database Analyst for Maven Fuzzy Factory, an retailer which has just launched their first product. We need to analyze and optimize marketing channels, measure and test website conversion performance, and use data to understand the impact of new product launches
 
- Access and explore the Maven Fuzzy Factory database
- Become the data expert for the company, and the go-to person for mission critical analyses
- Analyze and optimize the business’ marketing channels, website, and product portfolio

 
# Tables
| Table Name | Rows | Primary Key |Description |
| --- | --- |--- |--- |
| `Website_sessions` | 40086 |website_session_id|when a user browse the website, it records the time and source (http_refer) and devices types`|
| `Website_pageviews` | 1129174 |website_pageview_id | helps figure out where customers are landing on your website, what happen after they get there|
| `Orders` | 32349 |order_id | store the orders info that customers place at what time from which channels |
| `Orders_items` | 31171 |order_item_id | a customer could place multiple items  |
| `Order_item_refund` | 1731  |order_item_refund_id | store the refund date and amount for specific orders and order items  |
| `Products` |  4096  |product_id | store the product name and id |


 
We will be working with six related tables, which contain eCommerce data about:
- **Website Activity**
- **Products**

We'll use MySQL to understand how customers access and interact with the site, analyze landing page performance and conversion, and explore product-level sales

 
# Data model_Schema
![coffee factory data schema sql](https://github.com/Shuangyi-im/Coffee_Factory_Ecommerce_data_analysis_sql/assets/78413872/d12fbd80-4b72-4254-a709-f76eb299ad09)

# SQL coding and analysis

**Analytics Traffic Sources**

When businesses run paid marketing campaigns, they often obsess over performance and
measure *everything*; how much they spend, how well traffic converts to sales, etc.



Paid traffic is commonly tagged with **tracking (UTM) parameters**, which are appended to
URLs and allow us to tie website activity back to specific traffic sources and campaigns
 we will use the trm parameters stored in the database to identify paid website sessions**
 
## 1 Finding Top Traffic Source

**❓ Question 1:** we will use the trm parameters stored in the database to identify paid website sessions link to order data to understand **how much revenue our paid campaigns are driving**


```sql

/*
 we will use the trm parameters stored in the database to identify paid website sessions
link to order data to understand how much revenue our paid campaigns are driving*/
USE mavenfuzzyfactory;

select utm_content,
count(distinct ws.website_session_id) as sessions,
count(distinct o.order_id) as orders
from website_sessions as ws
left join orders as o
on o.website_session_id = ws.website_session_id
where ws.website_session_id between 1000 and 2000 -- arbitrary 
group by ws.utm_content,o.order_id 
order by sessions desc; 
```
 
 
 
**❓ Question 2:** Manager wants to know where the bulk of website sessions are coming from, 
breakdown by *utm_source,utm_campaign,referring domain (http_referer)*
 
 
```sql
select utm_source,utm_campaign,http_referer,count(distinct website_session_id) as sessions
from website_sessions
where created_at < '2012-04-12'
group by utm_source,utm_campaign,http_referer
order by sessions desc;
```
 
  
**Output**
![q2 answers](https://github.com/Shuangyi-im/Coffee_Factory_Ecommerce_data_analysis_sql/assets/78413872/07b1da3a-c271-487e-83a3-240bd7a04c1b)
 
**Conclusion**: gsearch nonbrand is our major traffic source, and we need to understand if those sessions are driving sales. Next ,drill deeper into gsearch nonbrand campaign traffic to explore potential optimization opportunities
 
 
 
**❓Question 3:**
 
Could you please **calculate the conversion rate (CVR) from session to order**? Based on what we're paying for clicks, we’ll need a CVR of **at least 4%** to make the numbers work.
 
```sql
select 
count(distinct ws.website_session_id) as sessions,
count(distinct o.order_id) as orders,
count(distinct o.order_id) / count(distinct ws.website_session_id) as session_to_order_conv_rt
from website_sessions as ws
left join orders as o
on o.website_session_id = ws.website_session_id
where ws.created_at < '2012-04-14' 
and utm_campaign = 'nonbrand'
and utm_source = 'gsearch';
```
 
**Output**
 
![calculate the conversion rate (CVR) output](https://github.com/Shuangyi-im/Coffee_Factory_Ecommerce_data_analysis_sql/assets/78413872/a47f8810-92f0-47f5-81a7-bd91faf4ef7a)
 
its below the 4% threshold，need to conduct further analysis and discuss with the team about the reasons/actions.
 
  
   
## 2.Bid optimisation and trend analysis 
* *Business concepts: Analyzing for bid optimization is about understanding the value of various
segments of paid traffic, so that you can optimize your marketing budge* *

**COMMON USE CASES:**

- Using conversion rate and revenue per click analyses to figure out how much you should
spend per click to acquire customers
- Understanding how your **website and products perform** for various subsegments of traffic (i.e. mobile vs desktop) to optimize within **channels**
- Analyzing the **impact** that bid changes have on your **ranking in the auctions**, and the volume of customers driven to your site

![bid](https://github.com/Shuangyi-im/Coffee_Factory_Ecommerce_data_analysis_sql/assets/78413872/c737541c-e209-4499-b3c4-80259a7d93e7)

```sql
select 
year(created_at),
week(created_at),
MIN(date(created_at)) as week_start, -- very important to show the starting date for that week you select
count(distinct website_session_id) as sessions
from website_sessions
group by 1,2;
```


Output 

![output bid analysis](https://github.com/Shuangyi-im/Coffee_Factory_Ecommerce_data_analysis_sql/assets/78413872/3411f90b-a1a0-498d-bd27-18c2523282fe)

### PRO TIP: “PIVOTING” DATA WITH COUNT & CASE
> Excel’s ability to pivot to columns can be replicated in SQL using COUNT and CASE

Here we’re breaking down the count of order_id by primary_product_id (rows) and items_purchased (columns) to see how many orders were placed for each primary product and how many of those orders included multiple items


```sql
select primary_product_id,
		count(distinct case when items_purchased = 1 then order_id else null end) as single_item_orders,
		count(distinct case when items_purchased = 2 then order_id else null end) as two_item_orders
from orders 
where order_id between 31000 and 32000 -- arbitrary
group by 1;
```
![pivot](https://github.com/Shuangyi-im/Coffee_Factory_Ecommerce_data_analysis_sql/assets/78413872/c54780b2-e00d-4868-900c-952ab139c945)


