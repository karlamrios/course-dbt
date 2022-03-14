# Analytics engineering with dbt

Template repository for the projects and environment of the course: Analytics engineering with dbt

> Please note that this sets some environment variables so if you create some new terminals please load them again.

## License

Apache 2.0

/*Q1: How many users are there? A: 130, none repeated in data set
No users entered two different email. 130 users seem unique
*/
select 
count(distinct(user_id)) as "dist_uid",
count(distinct(user_id)) as "count_uid",
count(distinct(lower(concat(first_name,last_name)))) as "unique_names",
count(distinct(email)) "unique_emails"
from users;

/*Q2: On average, how many orders do we receive per hour?
A: 8 orders per hour
*/
with hourly_orders as (
select
date_trunc('hour',created_at) as "date_hour_placed",
count(order_id) as "orders_placed"
from orders group by date_trunc('hour',created_at))
select round(avg(orders_placed),0) from hourly_orders;

/*Q3: On average, how long does an order take from being placed to being delivered?
A: On average cycle time is 3 days
*/
with delivery_cycletime as (
Select
(delivered_at - created_at) as "cycletime"
from orders 
where lower(status) = 'delivered')
select avg(cycletime) from delivery_cycletime

/*Q4: How many users have only made one purchase? Two purchases? Three+ purchases?
A: 25/124 users had 1 purchases, 28/124 users had 2 purchases, 34/124 users had 3 purchases
*/

with orders_per_user as (--For each user, count unique orders
select 
user_id,
count(distinct(order_id)) as "orders"
from orders group by user_id
order by count(distinct(order_id)) desc)
--count users with the same amount of orders
select 
orders_per_user.orders,
count(orders_per_user.user_id) as "users"
from orders_per_user
group by orders order by orders;
--Validating Query that there are 6 users witout orders
select count(*) from (
select u.user_id from users as u
left join orders as o on o.user_id=u.user_id
where o.user_id is NULL) as "users_no_orders";

/*Q5:On average, how many unique sessions do we have per hour?
A: 12 sessions per hour across all time stamps
*/

select round(avg (sessions_per_hour),0)
  from (
  with b as (select s_id.session_id, min(date_trunc('hour',events.created_at)) as first_detection
  from (select distinct (session_id) as "session_id" from events) as s_id 
    join events on s_id.session_id=events.session_id 
    --where s_id.session_id ='69afb40a-7647-458b-967b-3afbe9075dd0'
    where date_trunc('hour',events.created_at) <> '2021-02-11T14:00:00Z'
    group by s_id.session_id)
  select
  count(session_id) as sessions_per_hour ,first_detection
  from b 
  group by first_detection --this subquery shows 229 unique sessions for 2021-02-11T14:00:00Z, seems like an anomally date
) as a

/* Feedback and notes from a person who knows nothing about dbt or snowflake (which is often referrenced) - mostly analytics background
Difficulties with CLI / dbt
1. Even though it looks like you can export data, I couldn't make it WORK
2. How is a project saved?  Couldn't figure that out without having to push everything through GIT
3. I dont understand what the profile yml file is FOR
4. 
Difficulties with Course material
1.  It is not organized in a way that is easy to quickly find what you need since there is no index or glossary
2.  It would be easier to follow along the content referring videos and images, if they were actually labeled and I could easily find what I needed
3.  Set up steps in video without a checklist were frustrating.  Example: Video moved too fast and I only have laptop to work with.  Flipping between browswer pages
and maximazing the video so I could read what the person was typing was not frustrating. Sometimes video was too fast for me to understand what command or icon was 
being clicked.  It also was very time consuming to find the exact spot of the video I needed to re-watch.


*/
