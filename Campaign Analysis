create view sessions_orders as
select ws.*,o.order_id,o.primary_product_id,o.items_purchased,o.price_usd,o.cogs_usd
from website_sessions ws
left join orders o on ws.website_session_id=o.website_session_id
--created a view for querying convenience

--TRAFFIC ANALYSIS--
--year month wise traffic--
SELECT 
YEAR(created_at) AS year,
MONTH(created_at) AS month,count(*) as total_traffic,count(distinct order_id)as total_orders,
count(distinct order_id)*1.0/count(*)*100 as converstion_rate
FROM sessions_orders
GROUP BY YEAR(created_at), MONTH(created_at)
ORDER BY year,month
--found a traffic spike in nov dec every year
--traffic & conversion rate increasing over time, which is a good sign 

--utm content wise check if it's affecting the traffic spike in Nov & Dec--(focus on = month 10,11,12)
SELECT utm_content,
YEAR(created_at) AS year,
MONTH(created_at) AS month,count(*) as total_traffic,count(distinct order_id)as total_orders,
count(distinct order_id)*1.0/count(*)*100 as converstion_rate
FROM sessions_orders
where utm_content ='g_ad_1'
GROUP BY utm_content,YEAR(created_at), MONTH(created_at)
having utm_content is not null and MONTH(created_at) in (9,10,11,12)
order by utm_content,year,month
--found g_ad_1 aquireing high traffic in every year nov and dec

--Product analysis--
--Product QTY sold
select product_name,count(order_item_id) total_orders
from sessions_orders so
left join order_items oi on oi.order_id=so.order_id
left join products p on p.product_id=oi.product_id
group by product_name
having product_name is not null 
order by product_name 
--The Original Mr. Fuzzy is the top-selling product.

--CHANNEL PERFORMANCE--
--Channel wise traffic--
SELECT utm_source,utm_campaign,utm_content,count(*) as total_traffic,count(distinct order_id)as total_orders,
count(distinct order_id)*1.0/count(*)*100 as converstion_rate
FROM sessions_orders
GROUP BY utm_source,utm_campaign,utm_content
having utm_campaign is not null
order by converstion_rate desc
--Brand campaign has lower traffic than nonbrand
--Social_ad_1 has the lowest conversion

--utm_content wise funnel analysis--
with cte as(
SELECT
    wp.website_session_id,so.utm_content,
    MAX(CASE WHEN pageview_url LIKE '%lander%' OR pageview_url LIKE '%home%' THEN 1 ELSE null END) AS Landing,
    MAX(CASE WHEN pageview_url LIKE '%products%' THEN 1 ELSE null END) AS Product_Listing,
    MAX(CASE WHEN pageview_url LIKE '%the%' THEN 1 ELSE null END) AS Product_Detail,
    MAX(CASE WHEN pageview_url LIKE '%cart%' THEN 1 ELSE null END) AS Cart,
    MAX(CASE WHEN pageview_url LIKE '%shipping%' THEN 1 ELSE null END) AS Shipping,
    MAX(CASE WHEN pageview_url LIKE '%billing%' THEN 1 ELSE null END) AS Billing,
    MAX(CASE WHEN pageview_url LIKE '%thank-you%' THEN 1 ELSE null END) AS Ordered
FROM website_pageviews wp
join sessions_orders so on wp.website_session_id=so.website_session_id
GROUP BY wp.website_session_id,so.utm_content
)
SELECT utm_content,
1.0 * SUM(Product_Listing) / sum(Landing)*100 AS Landing_to_Product_Listing_click_rate,
1.0 * SUM(Product_Detail) / SUM(Product_Listing)*100 AS Product_Detail_click_rate,
1.0 * SUM(Cart) / SUM(Product_Detail)*100 AS Cart_click_rate,
1.0 * SUM(Shipping) / SUM(Cart)*100 AS Shipping_click_rate,
1.0 * SUM(Billing) / SUM(Shipping)*100 AS Billing_click_rate,
1.0 * SUM(Ordered) / SUM(Billing)*100 AS Ordered_click_rate
from cte
group by utm_content
having utm_content is not null
--found traffic is clicking the ads, but after seeing the landing page, they drop off social_ad_1 and social_ad_2
--huge traffic dropping off at the billing page 

--AB testing to find the root cause behind landing page drops on social ads
SELECT 
    pageview_url,
    COUNT(DISTINCT so.website_session_id) AS sessions,
    COUNT(DISTINCT order_id) AS orders,
    100.0 * COUNT(DISTINCT order_id) / COUNT(DISTINCT so.website_session_id) AS billing_to_order_rate
FROM website_pageviews wp
LEFT JOIN sessions_orders so ON wp.website_session_id = so.website_session_id
WHERE pageview_url LIKE '%lander%' OR pageview_url LIKE '%home%' and utm_content in ('social_ad_1','social_ad_2')
GROUP BY pageview_url
--found very low conversion in lander 1 and lander 3

--AB test to find the root cause behind billing page drops
SELECT 
    pageview_url,
    COUNT(DISTINCT so.website_session_id) AS sessions,
    COUNT(DISTINCT order_id) AS orders,
    100.0 * COUNT(DISTINCT order_id) / COUNT(DISTINCT so.website_session_id) AS billing_to_order_rate
FROM website_pageviews wp
LEFT JOIN sessions_orders so ON wp.website_session_id = so.website_session_id
WHERE pageview_url LIKE '%billing%'
GROUP BY pageview_url
--found that billing has a lower conversion rate than billing 2

--USER BEHAVIOR--
--traffic by device
select device_type,count(*) as total_traffic,count(distinct order_id)as total_orders,
count(distinct order_id)*1.0/count(*)*100 as converstion_rate
from sessions_orders
group by device_type
--mobile converstion rate drop

--FUNNEL ANALYSIS device wise--
with cte as(
SELECT
    wp.website_session_id,so.device_type,
    MAX(CASE WHEN pageview_url LIKE '%lander%' OR pageview_url LIKE '%home%' THEN 1 ELSE null END) AS Landing,
    MAX(CASE WHEN pageview_url LIKE '%products%' THEN 1 ELSE null END) AS Product_Listing,
    MAX(CASE WHEN pageview_url LIKE '%the%' THEN 1 ELSE null END) AS Product_Detail,
    MAX(CASE WHEN pageview_url LIKE '%cart%' THEN 1 ELSE null END) AS Cart,
    MAX(CASE WHEN pageview_url LIKE '%shipping%' THEN 1 ELSE null END) AS Shipping,
    MAX(CASE WHEN pageview_url LIKE '%billing%' THEN 1 ELSE null END) AS Billing,
    MAX(CASE WHEN pageview_url LIKE '%thank-you%' THEN 1 ELSE null END) AS Ordered
FROM website_pageviews wp
join sessions_orders so on wp.website_session_id=so.website_session_id
GROUP BY wp.website_session_id,device_type
)
SELECT device_type,
1.0 * SUM(Product_Listing) / sum(Landing)*100 AS Landing_to_Product_Listing_click_rate,
1.0 * SUM(Product_Detail) / SUM(Product_Listing)*100 AS Product_Detail_click_rate,
1.0 * SUM(Cart) / SUM(Product_Detail)*100 AS Cart_click_rate,
1.0 * SUM(Shipping) / SUM(Cart)*100 AS Shipping_click_rate,
1.0 * SUM(Billing) / SUM(Shipping)*100 AS Billing_click_rate,
1.0 * SUM(Ordered) / SUM(Billing)*100 AS Ordered_click_rate
from cte
group by device_type
--16% less mobile traffic is going to the product_detail page than desktop
--around 10 % mobile traffic drop in shipping and billing clicks than desktop

--repete vs new
select sum(is_repeat_session)as repete_session,count(*)-sum(is_repeat_session) as new_session
from sessions_orders
-- repete sessions bahut kam hai

--converstion
select is_repeat_session,round(count(distinct order_id)*1.0/count(*)*100,2) as converstion_rate
from sessions_orders
group by is_repeat_session
--high convesion in repete sessions

--Weekday performance--
SELECT datename(WEEKDAY,created_at) AS weekdays,count(*) as total_traffic,count(distinct order_id)as total_orders,
count(distinct order_id)*1.0/count(*)*100 as converstion_rate
FROM sessions_orders
GROUP BY datename(WEEKDAY,created_at)
order by total_traffic desc
--low orders on weekends

--REVENUE ANALYSIS--
select year(created_at)as year ,month(created_at)as month,
round(sum(price_usd),2)as total_sale,round(sum(price_usd)-sum(cogs_usd),2)as total_profit,
(sum(price_usd)-sum(cogs_usd))/sum(price_usd)*100 profit_pct
from orders
group by year(created_at),month(created_at)
order by year ,month

--RPO/RPS
SELECT 
SUM(price_usd) / COUNT(DISTINCT order_id) AS revenue_per_order,
SUM(price_usd) / COUNT(DISTINCT website_session_id) AS revenue_per_session
FROM sessions_orders
