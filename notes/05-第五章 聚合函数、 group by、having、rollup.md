# 第五章 聚合函数, group by, having, rollup

## 聚合函数 aggregate function
```sql
select
  max(invoice_total) as highest,
  min(invoice_total) as lowest,
  avg(invoice_total) as average,
  sum(invoice_total) as total,
  count(invoice_total) as number_of_invoices
-- 以上聚合函数只运行非空值,例如：
  count(payment_date) as number_of_payments
-- 由于payment_date有空值，因此得到的不是全部payment的数目，想要包含空值，要：
  count(*) as total_records
-- 括号内可以写列名，也可以写公式，函数：
  sum(invoice_total*1.1) as total,
from invoices
where invoice_date>'2019-07-01'
-- 以上函数会计算重复值，想要排除重复值，确定有几个客户：
  count(distinct client_id)
-- exercise
select
  'first half of 2019' as date_range,
  sum (invoice_total) as total_sales,
  sum(payment_total) as total_payments
  sum(invoice_total-payment_total) as what_we_expect
from invoices
where invoice_date
  between'2019-01-01' and '2019-06-30'
union
select
  'second half of 2019' as date_range,
  sum (invoice_total) as total_sales,
  sum(payment_total) as total_payments
  sum(invoice_total-payment_total) as what_we_expect
from invoices
where invoice_date
  between '2019-07-01' and '2019-12-31'
union
select
  'total' as date_range,
  sum (invoice_total) as total_sales,
  sum(payment_total) as total_payments
  sum(invoice_total-payment_total) as what_we_expect
from invoices
where invoice_date
  between'2019-01-01' and '2019-12-31'
```

## group by子句
```sql
select
  client_id
  sum(invoice_total) as total_sales
from invoices
group by client_id
order by total_sales desc
```
<img width="226" height="177" alt="image" src="https://github.com/user-attachments/assets/8eccb182-b2d9-46f3-8e05-a15b3219f694" />

```sql
select
  client_id
  sum(invoice_total) as total_sales
from invoices
where invoice_date>='2019-07-01'
group by client_id
order by total_sales desc
```
- 顺序很重要：
  select, from, where, group by, order by <br>
用多列进行分组：
```sql
select
  client_id
  sum(invoice_total) as total_sales
from invoices i
join clients using(client_id)
where invoice_date>='2019-07-01'
group by state, city
order by total_sales desc
-- exercise
select
p.date,
pm.payment_method,
sum(amount) as total_payments
from payments p
join payment_method pm
-- 因为在两个表中要链接的列名不一样，所以不能用using
 on p.payment_method=pm. payment_method_id
group by date, payment_method
order by date
```

## having子句

```sql
-- 想要销售额超过500的客户，不能：
select
  client_id
  sum(invoice_total) as total_sales
from invoices
where total_sales>500
group by client_id
-- 因为where执行的时候group by还没有被执行，没有total_sales列，因此要用having：
select
  client_id
  sum(invoice_total) as total_sales
from invoices
group by client_id
having total_sales>500
-- having在group by之后筛选数据

-- 真正的执行顺序：
from invoices
where ...
group by client_id
having ...
select ...

-- 想要选择销售额超过500，发票书超过5的
select
  client_id
  sum(invoice_total) as total_sales
  count(*) as number_of_invoices
from invoices
group by client_id
having total_sales>500 and、number_of_invoices>5
-- having中用到的列一定是select中选过的，而where不需要

-- exercise：
select 
c.customer_id,
c.first_name,
c.last_name,
sum(oi.quantity*oi.unit_price) as total_sales
from customers c
join orders o
using (customer_id)
join order_items oi
using (order_id)
where state='VA'
group by 
c.customer_id,
c.first_name,
c.last_name
having total_sales>100
```

## rollup运算符
```sql
select
  client_id
  sum(invoice_total) as total_sales
from invoices
group by client_id with roll up
-- rollup汇总整个结果集，用于聚合值的列
select
  state,
  city,
  sum(invoice_total) as total_sales
from invoices i
join clients c using(clients_id)
group by state,city with roll up
-- exercise
SELECT 
pm. name,
sum(amount) as total
FROM clients
join payments p using (client_id)
join payment_methods pm 
where p. payment_method=pm. payment_method_id
group by pm. name with rollup
```
