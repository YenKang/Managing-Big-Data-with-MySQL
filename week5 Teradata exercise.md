## week5 Teradata exercise

#### [Teradata Viewpoint](http://uatdviewpoint.waltoncollege.uark.edu/)



***Exercise 1. How many distinct dates are there in the saledate column of the transaction table for each month/year combination in the database?***


```
Database ua_dillards;
SELECT COUNT(DISTINCT EXTRACT(MONTH from saledate)) AS month_num
FROM trnsact;

SELECT COUNT(DISTINCT EXTRACT(YEAR from saledate)) AS year_num
FROM trnsact;
```


> 月份有12個月, 年有2年

##### 第二次答案

```
Database ua_dillards;
select distinct (extract(YEAR from saledate) || (extract(MONTH from saledate))) AS year_month  
from trnsact
ORDER BY year_month ;
```

***Exercise 2. Use a CASE statement within an aggregate function to determine which sku had the greatest total sales during the combined summer months of June, July, and August.***


```
SELECT MAX(CASE WHEN EXTRACT(MONTH from saledate)=12 THEN saledate END) 
AS last_day_in_Dec
FROM trnsact;
```

> 2004-12-31


```
SELECT MAX(CASE WHEN EXTRACT(MONTH from saledate)=12 THEN saledate END) 
AS last_day_in_Dec
FROM trnsact;
```

>  2005-05-05



```
select distinct sku,
sum(case when extract(month from saledate) = 6 then amt end) as june_sale,
sum(case when extract(month from saledate) = 7 then amt end) as july_sale,
sum(case when extract(month from saledate) = 8 then amt end) as aug_sale,
(june_sale + july_sale + aug_sale) as total_sale

from trnsact  
group by sku
order by total_sale desc;

```

***Exercise 3. How many distinct dates are there in the saledate column of the transaction table for each month/year/store combination in the database? Sort your results by the number of days per combination in ascending order.*** 

> 根據每一個 月、年、店的組合，在saledate column中，查看有多少相異的日子


```
Database ua_dillards;
select distinct(extract(MONTH from saledate)|| extract(YEAR from saledate)|| store) AS month_year_store, 
COUNT(month_year_store) AS Num_Dates

FROM trnsact
GROUP BY month_year_store
ORDER BY Num_Dates ASC,month_year_store DESC;
```


> - (month_year_store, Num_month_year_store)
> - (2004-08 8304, 1)
> - (2005-08 8304, 1)
> - (2004-09 4402, 178)
> - (2005-07 7604, 1079)

> 目前我覺得數據怪怪的，原因是一個月份中，日子的上界應該是31，不會出現到178 或是1079的數字

***Exercise 4. What is the average daily revenue for each store/month/year combination in the database? Calculate this by dividing the total revenue for a group by the number of sales days available in the transaction table for that group.***


```
Database ua_dillards;
select distinct (extract(month from saledate) || extract(year from saledate) || store) as mys, 
(SUM(amt)/COUNT(DISTINCT saledate)) AS dailyRevenue

from trnsact 
group by mys
WHERE stype='p'
HAVING COUNT(DISTINCT saledate) >20
order by dailyRevenue ASC;
```
> 目前我還不曉得要怎麼把August, 2005去除

***Exercise 5. What is the average daily revenue brought in by Dillard’s stores in areas of high, medium, or low levels of high school education?*** 

> 一個區域，擁有高、中、低層級的教育水平，要用什麼指標來區分呢？文本提到用高中
畢業率當作區分的點

> 第一步，先把exercise4的答案先複製過來


```
Database ua_dillards;
select distinct (extract(month from saledate) || extract(year from saledate) || store) as mys, 
(SUM(amt)/COUNT(DISTINCT saledate)) AS dailyRevenue

from trnsact 
group by mys
WHERE stype='p' AND store=204
HAVING COUNT(DISTINCT saledate) >20
order by dailyRevenue ASC;
```


> 接著，看要如何把教育水平的指標納入這query中


```
SELECT CASE when msa_high >= 50 and msa_high <= 60 then 'low' 
            when msa_high >  60 and msa_high <= 70 then 'medium'
	    when msa_high >  70 then 'high'
	    end as edu_level, 
	    (SUM(total_Rev_per_store)/SUM(num_Day)) AS avg_Rev
FROM store_msa 
JOIN(select distinct (extract(month from saledate) || extract(year from saledate) || store) as month_year, store,
    SUM(amt) AS total_Rev_per_store,
    COUNT(distinct saledate) AS num_Day
    from trnsact 
    group by month_year,store
    WHERE stype='p'
    HAVING num_Day >20
) AS revenue ON store_msa.store=revenue.store

GROUP BY edu_level
```

***Exercise 6. Compare the average daily revenues of the stores with the highest median msa_income and the lowest median msa_income. 
In what city and state were these stores, and which store had a higher average daily revenue?*** 



```
SELECT (SUM(total_Rev_per_store)/SUM(num_Day)) AS avg_Rev, city, state

FROM (SELECT TOP 3 city, state,msa_income, store
FROM store_msa
ORDER BY msa_income DESC) AS highest_Income
LEFT JOIN(
    select distinct (extract(month from saledate) || extract(year from saledate) || store) as  month_year, store,
    SUM(amt) AS total_Rev_per_store,
    COUNT(distinct saledate) AS num_Day
    
    from trnsact  
    group by month_year, store
    WHERE stype='p'
    HAVING num_Day >20
) AS revenue ON highest_Income.store=revenue.store

GROUP BY city,state;
```


***Exercise 7: What is the brand of the sku with the greatest standard deviation in sprice? Only examine skus that have been part of over 100 transactions.*** 

> 察覺商品的波動性，波動性太高，會影響庫存的判斷，進而影響運作效率，接著，進入指令的部分，相對MySQL的STDDEV, TERADATA是用“STDDEV_SAMP()”
>
>看到題目，先找出所有的column, brand, sku, sprice, transaction的數量(quantity), 然後找出這五的欄位對應的table, 在進行分類


```
Database ua_dillards;
SELECT s.brand, cleaned_trnsact.std_price
FROM(SELECT TOP 3 stddev_samp(sprice) AS std_price,sku
	FROM trnsact
	GROUP BY sku
	HAVING SUM(quantity) >100
	WHERE stype='p'
	ORDER BY std_price DESC) AS cleaned_trnsact 
LEFT JOIN skuinfo s ON s.sku=cleaned_trnsact.sku
```

***Exercise 8:Examine all the transactions for the sku with the greatest standard deviation in sprice, but only consider skus that are part of more than 100 transactions.*** 

>// 這題我有做調整，挑選欄位部分是sku 與 他的波動價格，並且限制在前十名


```
Database ua_dillards;
SELECT cleaned_trnsact.sku AS sku, cleaned_trnsact.std_price
FROM(SELECT TOP 10 stddev_samp(sprice) AS std_price,sku
	FROM trnsact
	GROUP BY sku
	HAVING SUM(quantity) >100
	WHERE stype='p'
	ORDER BY std_price DESC) AS cleaned_trnsact 
LEFT JOIN skuinfo s ON s.sku=cleaned_trnsact.sku
```


> // (sku, std_price):
>第一名：(2762683, 175.8), 
>第二名：(5453849, 169.4), 
>第三名：(5623849, 164.4)

***Exercise 9: What was the average daily revenue Dillard’s brought in during each month of the year?***

> // 承接exercise4, 我先把年分為2004與2005來看，得到兩個結果，

```
Database ua_dillards;
select distinct (extract(YEAR from saledate) || extract(MONTH from saledate)) as year_month, (SUM(amt)/COUNT(DISTINCT saledate)) AS dailyRevenue
from trnsact 
group by year_month
where stype = 'p' and oreplace(year_month, ' ', '') not like '%82005%'
HAVING COUNT(DISTINCT saledate) >20
ORDER by dailyRevenue ASC;
```

- 2004 9, 5596588.02
- 2004 8, 5616841.37
- 2004 10, 6106357.90
- 2004 11, 6296913.50
- 2004 12, 11333356.01 // 12月，營收最高
- 
- 2005 1, 5836833.31
- 2005 6, 6524845.42
- 2005 5, 6666962.59
- 2005 3, 6736315.39
- 2005 4, 6949616.95
- 2005 7, 7271088.69
- 2005 2, 7363752.69
- 2005 8, 7450440.39

***Exercise 10: Which department, in which city and state of what store, had the greatest % increase in average daily sales revenue from November to December?***


```
SELECT store.city,
store.state, 
sku.dept, 
cleaned_trnsact.Gain

FROM(
    SELECT DISTINCT store, sku,
    SUM(CASE WHEN EXTRACT(MONTH from saledate)=12 THEN  amt end) as Dec_Revenue,
    SUM(CASE WHEN EXTRACT(MONTH from saledate)=11 THEN  amt end) as Nov_Revenue,
    COUNT(CASE WHEN EXTRACT(MONTH from saledate)=11 THEN  saledate end) as Nov_num_day,
    COUNT(CASE WHEN EXTRACT(MONTH from saledate)=12 THEN  saledate end) as Dec_num_day,
    (Dec_Revenue/Dec_num_day) AS Dec_Daily_Rev,
    (Nov_Revenue/Nov_num_day) AS Nov_Daily_Rev,
    ((Dec_Daily_Rev/Nov_Daily_Rev-1)*100) as Gain
    FROM trnsact
    group by store,sku
    WHERE stype='p'
    HAVING Nov_num_day>20 AND Dec_num_day>20
)AS cleaned_trnsact

LEFT JOIN skuinfo sku ON cleaned_trnsact.sku=sku.sku 
LEFT JOIN store_msa store ON cleaned_trnsact.store=store.store
LEFT JOIN deptinfo dep ON sku.dept=dep.dept
GROUP BY store.city,store.state, sku.dept, cleaned_trnsact.Gain
ORDER BY GAIN DESC;
```


***Exercise 11: What is the city and state of the store that had the greatest decrease in average daily revenue from August to September?***


```
select st.city, st.state,decr
from(
	select distinct store
	,sku
	,sum(case when extract(month from saledate) =  8 then amt end) as augrev
	,sum(case when extract(month from saledate) =  9 then amt end) as septrev
	,count(distinct case when extract(month from saledate) =  8 then saledate end) as augday
	,count(distinct case when extract(month from saledate) =  9 then saledate end) as septday
	,augrev/augday as augdailyrev
	,septrev/septday as septdailyrev
	,(septdailyrev-augdailyrev) as decr
	from trnsact 
	group by store,sku
	where stype = 'p' 
	having augday > 20 and septday > 20   	  
) as rev left join store_msa st on rev.store = st.store
group by st.city,st.state ,decr
order by decr asc;
```

> //(city, state,decrease)=(KNOXVILLE, TN, -444.28)

***Exercise 12: Determine the month of maximum total revenue for each store. Count the number of stores whose month of maximum total revenue was in each of the twelve months. 
Then determine the month of maximum average daily revenue. Count the number of stores whose month of maximum average daily revenue was in each of the twelve months. How do they compare?*** 

> 第一階段，先試用ROW＿NUMBER() 對daily Revenue的排序結果


```
Database ua_dillards;
select distinct (store || extract(MONTH from saledate)) as store_month,(sum(amt)/count(distinct saledate)) as dailyrev,
ROW_NUMBER() OVER (ORDER BY dailyrev DESC) AS Rank_avgDailyRev

from trnsact 
group by store_month
where stype = 'p' 
having count(distinct saledate) > 20
ORDER BY Rank_avgDailyRev ASC;
```


> - 結果是，有三個欄位，第一欄位是“store 代碼+月份”; 第二欄位是Daily Revenue數字，由最大排到最小，第三欄是“Rank_avgDailyRev”，根據第二欄的數目做排列，最大的數值，就排名rank 1，數值最小。
> - 但是，目前的問題就是，rank會從第1名排到第3893名，這是有問題的，因爲文檔說 “Make sure you partition by store in your ROW_NUMBER”
修改後的指令是，


```
Database ua_dillards;
select distinct (store || extract(MONTH from saledate)) as store_month,
(sum(amt)/count(distinct saledate)) as dailyrev,
RANK() OVER (PARTITION BY store ORDER BY dailyrev DESC) AS "Rank_avgDailyRev"

FROM trnsact 
group by store, store_month
where stype = 'p' 
having count(distinct saledate) > 20
QUALIFY Rank_avgDailyRev<=12;
```


> //結果是，同一個store分為12個月份，而針對這十二個月份的平均營收對此作出排名，從第一到第十二名

> 接著，把QUALIFY Rank_avgDailyRev=1
結果顯示很多都是12月為第一名


```
SELECT count(case when mm = 1 then mm end) as count_jan,
count(case when mm = 2 then mm end) as count_feb,
count(case when mm = 3 then mm end) as count_mar,
count(case when mm = 4 then mm end) as count_apr,
count(case when mm = 5 then mm end) as count_may,
count(case when mm = 6 then mm end) as count_jun,
count(case when mm = 7 then mm end) as count_july,
count(case when mm = 8 then mm end) as count_aug,
count(case when mm = 9 then mm end) as count_sep,
count(case when mm = 10 then mm end) as count_otc,
count(case when mm = 11 then mm end) as count_nov,
count(case when mm = 12 then mm end) as count_dec

FROM(select distinct store,
extract(month from saledate) as mm,
SUM(amt) AS total_Revenue,
RANK() OVER(PARTITION BY store ORDER BY total_Revenue ASC) AS "Rank_total_Revenue"
FROM trnsact 
group by store, mm
where stype = 'p' 
having count(distinct saledate) > 20
QUALIFY Rank_total_Revenue=1) AS modified_trnsact;
```




















