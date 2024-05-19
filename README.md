## Credit Card Spending Habits in India

# SQL Portfolio Project

- **About the database:**
The database has been taken from https://www.kaggle.com/datasets/thedevastator/analyzing-credit-card-spending-habits-in-india
import the dataset in SQL server with table name: credit_card_transcations. In this database, it is shown that the spending habits of the credit cards in India's 986 unique cities.
There are 4 credit card types: **Silver, Signature, Gold, and Platinum**
Expense_Type of the credit cards are **Entertainment, Food, Bills, Fuel, Travel, Grocery**
It's a database with **26,052 rows**.

**1.** **write a query to print top 5 cities with highest spends and their percentage contribution of total credit card spends**
```
with cte as(
select city, SUM(amount) spend
from credit_card_transcations$ 
group by city), cte2 as(
select SUM(spend)  total_spend
from cte
)
, cte3 as(select *, (cte.spend/cte2.total_spend)*100 p_c from cte join cte2 on 1=1)
select top 5 city from cte3
order by spend desc, p_c desc;
```
**2.** **write a query to print highest spend month and amount spent in that month for each card type**
```
with cte as(
select card_type, DATEPART(month,transaction_date) month, sum(amount) spend
from credit_card_transcations$
group by card_type, DATEPART(month,transaction_date) ),
cte2 as(

select *, RANK()over(partition by card_type order by spend desc) as rnk
from cte)
select * from cte2
where rnk=1;
```
**3.** **write a query to print the transaction details(all columns from the table) for each card type when**
**it reaches a cumulative of 1000000 total spends(We should have 4 rows in the o/p one for each card type)**
```
with cte as(
select * ,sum(amount)over(partition by card_type order by transaction_id,transaction_date) spend
from credit_card_transcations$)
, cte2 as(select *, rank() over(partition by card_type order by spend) as rn from cte where spend >= 1000000)
select *   
from cte2 where  rn=1
```
**4.** **write a query to find city which had lowest percentage spend for gold card type**
```
select * from credit_card_transcations$;

select top 1 city,card_type,(SUM(amount)/(select SUM(amount) from credit_card_transcations$))*100 as p_c from credit_card_transcations$
where card_type = 'Gold'
group by city,card_type
order by p_c

with cte as (
select top 1 city,card_type,sum(amount) as amount
,sum(case when card_type='Gold' then amount end) as gold_amount
from credit_card_transcations$
group by city,card_type)
select 
city,sum(gold_amount)*1.0/sum(amount) as gold_ratio
from cte
group by city
having count(gold_amount) > 0 and sum(gold_amount)>0
order by gold_ratio;
```
**5.** **write a query to print 3 columns:  city, highest_expense_type , lowest_expense_type (example format : Delhi , bills, Fuel)**
```
with cte as(
select city, exp_type, SUM(amount) spend  from credit_card_transcations$
group by city, exp_type)
, cte2 as(
select *, RANK()over (partition by city order by spend asc)rn_asc, RANK()over (partition by city order by spend desc) rn_desc
from cte)
select city, max(case when rn_desc=1 then exp_type end) highest_expense_type , min(case when rn_asc=1 then exp_type end) lowest_expense_type from cte2
group by city;
```
**6.** **write a query to find percentage contribution of spends by females for each expense type**
```
with cte as(
select exp_type, sum(amount) total_spend from credit_card_transcations$
group by exp_type)
, cte2 as(select exp_type, sum(amount) spend from credit_card_transcations$
where gender = 'F'
group by exp_type)
select cte.exp_type, (cte2.spend/cte.total_spend)*100 as p_c from cte2 join cte on cte.exp_type=cte2.exp_type
;

```
**7.** **which card and expense type combination saw highest month over month growth in Jan-2014**
```
select * from credit_card_transcations$;
with cte as(
select card_type,exp_type,DATEPART(year, transaction_date)year,datepart(month,transaction_date)month,sum(amount)  spend
from credit_card_transcations$
group by card_type,exp_type,DATEPART(year, transaction_date),datepart(month,transaction_date))
, cte2 as(
select * , lag(spend,1,0)over(partition by card_type,exp_type order by year,month) as prev_sales 
from cte
), cte3 as(
select *, rank()over (order by (spend-prev_sales)/(prev_sales) desc)rnk from cte2
where year = 2014 and month = 1)
select * from cte3
where rnk = 1;
```
**8.** **during weekends which city has highest total spend to total no of transcations ratio**
```
select top 1 city,sum(amount)/count(*) ratio
from credit_card_transcations$
where DATENAME(WEEKDAY,transaction_date) in ('Saturday','Sunday')
group by city
order by ratio desc
```
**9.** **which city took least number of days to reach its 500th transaction after the first transaction in that city**
```
with cte as(
select *,ROW_NUMBER()over(partition by city order by transaction_date asc) row_num from credit_card_transcations$),
cte2 as(select city , DATEDIFF(DAY,MIN(transaction_date),max(transaction_date)) as datediff1 from cte
where row_num = 1 or row_num = 500
group by city
having count(*)=2
)
select top 1 * from cte2
order by datediff1; 
```
