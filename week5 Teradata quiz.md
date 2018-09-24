## week5 Teradata Quiz

***第2個問題:How many distinct skus have the brand “Polo fas”, and are either size “XXL” or “black” in color?***

> 總共有sku, brand, size, color四個欄位


```
SELECT COUNT(DISTINCT sku) AS num_sku
FROM skuinfo
WHERE brand='Polo fas' AND SIZE='XXL' AND color=‘black';
```

>  有84個distinct skus 

***第 3 個問題:There was one store in the database which had only 11 days in one of its months (in other words, that store/month/year combination only contained 11 days of transaction data). In what city and state was this store located?***
************************************************************************
> 答案：
>
>- 6402 3 2005代表store是6402與2005年三月, 
>- 城市ATLANTA, 州GA,
>- 11天
************************************************************************




***第 4 個問題
Which sku number had the greatest increase in total sales revenue from November to December?
.***
************************************************************************
>答案：
>- sku=3949538, 
>- 從12月到11月增加了815080.21

************************************************************************

```
SELECT sku,
SUM(case WHEN extract(month from saledate)=12 THEN amt end)AS Dec_total_Rev,
SUM(case WHEN extract(month from saledate)=11 THEN amt end)AS Nov_total_Rev,
(Dec_total_Rev-Nov_total_Rev) AS Increase

FROM trnsact
GROUP BY sku
WHERE stype='p'
ORDER BY Increase DESC;

```



***第 5 個問題
What vendor has the greatest number of distinct skus in the transaction table that do not exist in the skstinfo table? (Remember that vendors are listed as distinct numbers in our data set).***

************************************************************************
>答案：
> - VENDOR=571232 
>- 有16024個sku number

************************************************************************

```
SELECT count(DISTINCT t.sku) as num_skus, si.vendor FROM trnsact t
LEFT JOIN skstinfo s
ON t.sku=s.sku AND t.store=s.store
JOIN skuinfo si ON t.sku=si.sku WHERE s.sku IS NULL
GROUP BY si.vendor
ORDER BY num_skus DESC;
```




***第 6 個問題
What is the brand of the sku with the greatest standard deviation in sprice? Only examine skus which have been part of over 100 transactions.***
************************************************************************
答案 brand:Hart Sch
************************************************************************


```
select top 3 stddev_samp(sprice) as std, sku	
from trnsact
GROUP by sku
having SUM(quantity) > 100
where stype = 'p' 
order by std desc;

SELECT brand
FROM skuinfo
WHERE sku=2762683;
```

#### **另外的版本(標準)

```
Database ua_dillards;
SELECT TOP 1 t.sku, STDDEV_POP(t.sprice) AS sprice_stdev, count(t.sprice) AS num_transactions, si.style, si.color, si.size, si.packsize, si.vendor, si.brand
FROM trnsact t JOIN skuinfo si
ON t.sku = si.sku
WHERE stype='P'
GROUP BY t.sku, si.style, si.color, si.size, si.packsize, si.vendor, si.brand HAVING num_transactions > 100
ORDER BY sprice_stdev DESC;
```


***第 7 個問題
What is the city and state of the store which had the greatest increase in average daily revenue (as defined in Teradata Week 5 Exercise Guide) from November to December?***
************************************************************************
city: METAIRIE, 
state:LA,
 Increase:41423, STORE:8402
************************************************************************



```

Database ua_dillards; 
select st.city, st.state,rev.store, Incre
from(select distinct store,
    sum(case when extract(month from saledate) =  12 then amt end) as Dec_rev,
    sum(case when extract(month from saledate) =  11 then amt end) as Nov_rev,
    count(distinct case when extract(month from saledate) =  12 then saledate end) as Dec_day,
    count(distinct case when extract(month from saledate) =  11 then saledate end) as Nov_day,
    Dec_rev/Dec_day as Dec_dailyrev,
    Nov_rev/Nov_day as Nov_dailyrev,
(Dec_dailyrev-Nov_dailyrev) as Incre

from trnsact 
group by store
where stype = 'p' 
having Dec_day > 20 and Nov_day > 20   	  
) as rev left join store_msa st on rev.store = st.store

group by st.city,st.state ,Incre, rev.store
order by Incre DESC;

```


***第 8 個問題
Compare the average daily revenue (as defined in Teradata Week 5 Exercise Guide) of the store with the highest msa_income and the store with the lowest median msa_income.
In what city and state were these two stores, and which store had a higher average daily revenue?***
************************************************************************
// highest msa_income, city:spanish Fort ,Avg_rev:17762; 
// Lowest msa_income, city:MCALLEN, Avg_Rev:57408
************************************************************************

```
SELECT (SUM(total_Rev_per_store)/SUM(num_Day)) AS avg_Rev, city, state, revenue.store, msa_income

FROM (SELECT TOP 3
    city, state,msa_income,
    store
    FROM store_msa
    ORDER BY msa_income DESC) AS highest_Income LEFT JOIN
(select distinct (extract(month from saledate) || extract(year from saledate) ||         store) as month_year, 
    store,
    SUM(amt) AS total_Rev_per_store,
    COUNT(distinct saledate) AS num_Day
    from trnsact  
    group by month_year,store
    WHERE stype='p'
    HAVING num_Day >20
) AS revenue ON highest_Income.store=revenue.store
GROUP BY city,state, revenue.store, msa_income;
```


***第 9 個問題
Divide the msa_income groups up so that 
msa_incomes between 1 and 20,000 are labeled 'low', 
msa_incomes between 20,001 and 30,000 are labeled 'med-low', 
msa_incomes between 30,001 and 40,000 are labeled 'med-high', and msa_incomes between 40,001 and 60,000 are labeled 'high'. 
Which of these groups has the highest average daily revenue (as defined in Teradata Week 5 Exercise Guide) per store?***
************************************************************************
> 答案
> - low:34637, 
> - med-high:22155,
> - med-low:19372, 
> - high:18104
************************************************************************

```
SELECT CASE WHEN msa_income >=1 AND msa_income <=20000 then 'low'
	WHEN msa_income >=20001 AND msa_income <=30000 then 'med-low'
	WHEN msa_income >=30001 AND msa_income <=40000 then 'med-high'
	WHEN msa_income >=40001 AND msa_income <=60000 then 'high' end as income_lev,
	(SUM(total_Rev_per_store)/SUM(num_Day)) AS avg_Rev

FROM store_msa
JOIN (SELECT distinct (extract(month from saledate) || extract(year from saledate)) as month_year,
	store,
	SUM(amt) AS total_Rev_per_store,

	COUNT(distinct saledate) AS num_Day
	from trnsact 
	group by month_year,store
	WHERE stype='p'
	HAVING num_Day >20
) AS revenue ON store_msa.store=revenue.store

GROUP BY income_lev

```


***第 10 個問題
Divide stores up so that 
stores with msa populations between 1 and 100,000 are labeled 'very small’,
stores with msa populations between 100,001 and 200,000 are labeled 'small', 
stores with msa populations between 200,001 and 500,000 are labeled 'med_small', 
stores with msa populations between 500,001 and 1,000,000 are labeled 'med_large', 
stores with msa populations between 1,000,001 and 5,000,000 are labeled “large”, and 
stores with msa_population greater than 5,000,000 are labeled “very large”. 
What is the average daily revenue (as defined in Teradata Week 5 Exercise Guide) for a store in a “very large” population msa?***

************************************************************************
> 答案
> - very large, avg_daily_Revenue:25619, 
> - 題目是25451.57, 兩者差距在於是否去除2005 aug的資料
************************************************************************

```
SELECT (CASE WHEN msa_pop >=1 AND msa_pop <=100000 then 'very small'
	WHEN msa_pop >=100001 AND msa_pop <=200000 then 'small'
	WHEN msa_pop >=200001 AND msa_pop <=500000 then 'med_small'
	WHEN msa_pop >=5000001 AND msa_pop <=1000000 then 'med_large'
	WHEN msa_pop >=1000001 AND msa_pop <=5000000 then 'large'
	WHEN msa_pop > 5000000 then 'very large' end) as population_lev,
	(SUM(total_Rev_per_store)/SUM(num_Day)) AS avg_Rev

FROM store_msa
JOIN (SELECT distinct (extract(month from saledate) || extract(year from saledate)) as month_year,
	store,
	SUM(amt) AS total_Rev_per_store,
	COUNT(distinct saledate) AS num_Day
	from trnsact 
	group by month_year, store
	WHERE stype='p'
	HAVING num_Day >20
) AS revenue ON store_msa.store=revenue.store

GROUP BY population_lev;
```





第 11 個問題
Which department in which store had the greatest percent increase in average daily sales revenue from November to December, 
and what city and state was that store located in? 
Only examine departments whose total sales were at least $1,000 in both November and December.
************************************************************************
> 我的回答：（是錯誤的答案）
deptdesc:clinique, 
city:CHARLOTTE
State:NC
跟題目沒有完全一致Clinique department, Odessa, TX，但我選最接近的
************************************************************************

```
select distinct t.store,s.dept
	,sum(case when extract(month from saledate) =  11 then t.amt end) as nrev
	,sum(case when extract(month from saledate) =  12 then t.amt end) as drev
	,count(distinct case when extract(month from saledate) =  11 then t.saledate end) as novday
	,count(distinct case when extract(month from saledate) =  12 then t.saledate end) as decday
	,nrev/novday as ndailyrev
	,drev/decday as ddailyrev
	,(ddailyrev-ndailyrev)/ndailyrev as perinc

from trnsact t JOIN skuinfo s ON t.sku=s.sku
group by t.store,s.dept
where stype = 'p' 
having  novday > 20 and decday > 20 AND nrev>1000 AND drev>1000
ORDER BY perinc DESC
```

************************************************************************
> 以下是正確版答案


```
SELECT s.store, s.city, s.state, d.deptdesc, sum(case when extract(month from saledate)=11 then amt
end) as November,
COUNT(DISTINCT (case WHEN EXTRACT(MONTH from saledate) ='11' then saledate END)) as Nov_numdays, sum(case when extract(month from saledate)=12 then amt end) as December,
COUNT(DISTINCT (case WHEN EXTRACT(MONTH from saledate) ='12' then saledate END)) as Dec_numdays, ((December/Dec_numdays)-(November/Nov_numdays))/(November/Nov_numdays)*100 AS bump
FROM trnsact t JOIN strinfo s
ON t.store=s.store JOIN skuinfo si
ON t.sku=si.sku JOIN deptinfo d
ON si.dept=d.dept
WHERE t.stype='P' and t.store||EXTRACT(YEAR from t.saledate)||EXTRACT(MONTH from t.saledate) IN (SELECT store||EXTRACT(YEAR from saledate)||EXTRACT(MONTH from saledate)

FROM trnsact
GROUP BY store, EXTRACT(YEAR from saledate), EXTRACT(MONTH from saledate)
HAVING COUNT(DISTINCT saledate)>= 20)
GROUP BY s.store, s.city, s.state, d.deptdesc
HAVING November > 1000 AND December > 1000
ORDER BY bump DESC;
```




***第 12 個問題
Which department within a particular store had the greatest decrease in average daily sales revenue from August to September, 
and in what city and state was that store located?***
************************************************************************ 
> 答案(這題有兩個版本)
> - city:Louisville, 
> - state:KY
> - department:800, 
> - Store:9103
************************************************************************


```
select distinct t.store
	,s.dept
	,sum(case when extract(month from saledate) =  8 then amt end) as augrev
	,sum(case when extract(month from saledate) =  9 then amt end) as septrev
	,count(distinct case when extract(month from saledate) =  8 then saledate end) as augday
	,count(distinct case when extract(month from saledate) =  9 then saledate end) as septday
	,augrev/augday as augdailyrev
	,septrev/septday as septdailyrev
	,(septdailyrev-augdailyrev) as decr

from trnsact t JOIN skuinfo s ON t.sku=s.sku
group by t.store, s.dept
where stype = 'p' and oreplace((extract(month from saledate) || extract(year from saledate)), ' ', '') not like '%82005%'                
having   augday > 20 and septday > 20
ORDER BY decr ASC   

SELECT city, state

FROM store_msa
WHERE store=9103;
```
#### 第二版本指令


```
SELECT s.city, s.state, d.deptdesc, t.store,
CASE when extract(year from t.saledate) = 2005 AND extract(month from t.saledate) = 8 then 'exclude'
END as exclude_flag,
SUM(case WHEN EXTRACT(MONTH from saledate) ='8' THEN amt END) as August,
SUM(case WHEN EXTRACT(MONTH from saledate) ='9' THEN amt END) as September,

COUNT(DISTINCT (case WHEN EXTRACT(MONTH from saledate) ='8' then saledate END)) as Aug_numdays, COUNT(DISTINCT (case WHEN EXTRACT(MONTH from saledate) ='9' then saledate END)) as Sept_numdays, 
(August/Aug_numdays)-(September/Sept_numdays) AS dip

FROM trnsact t JOIN strinfo s
ON t.store=s.store JOIN skuinfo si
ON t.sku=si.sku JOIN deptinfo d
ON si.dept=d.dept WHERE t.stype='P' AND exclude_flag IS NULL AND t.store||EXTRACT(YEAR from t.saledate)||EXTRACT(MONTH from t.saledate) IN (SELECT store||EXTRACT(YEAR from saledate)||EXTRACT(MONTH from saledate)

FROM trnsact
GROUP BY store, EXTRACT(YEAR from saledate), EXTRACT(MONTH from saledate)
HAVING COUNT(DISTINCT saledate)>= 20)
GROUP BY s.city, s.state, d.deptdesc, t.store, exclude_flag
ORDER BY dip DESC;
```


 
***第 13 個問題
Identify which department, in which city and state of what store, had the greatest DECREASE in the number of items sold from August to September.
How many fewer items did that department sell in September compared to August?***
************************************************************************
> 答案：
> - store:9103, 
> - dept:800, 
> - city:LOUISVILLE, 
> - Decrease_item:-1224
************************************************************************


```
SELECT s.state, s.city, d.deptdesc, s.store, decre_item

FROM (select distinct store,sku
	,sum(case when extract(month from saledate) =  8 then quantity end) as Aug_quant
	,sum(case when extract(month from saledate) =  9 then quantity end) as Sep_quant
	,count(distinct case when extract(month from saledate) =  8 then saledate end) as augday
	,count(distinct case when extract(month from saledate) =  9 then saledate end) as septday
	,Sep_quant - Aug_quant as decre_item
	from trnsact t
	group by store,sku
	where stype = 'p' 
	having   augday > 20 and septday > 20
) AS quantity

LEFT JOIN store_msa s ON quantity.store = s.store
LEFT JOIN skuinfo sk ON quantity.sku =sk.sku
LEFT JOIN deptinfo d ON sk.dept = d.dept

GROUP BY s.state, s.city, d.deptdesc, s.store, decre_item
ORDER BY decre_item asc

```

### 另一版本

```
SELECT s.city, s.state, d.deptdesc, t.store,
CASE when extract(year from t.saledate) = 2005 AND extract(month from t.saledate) = 8 then 'exclude' END as exclude_flag,

SUM(case WHEN EXTRACT(MONTH from saledate) = 8 then t.quantity END) as August,
SUM(case WHEN EXTRACT(MONTH from saledate) = 9 then t.quantity END) as September, August-September AS dip FROM trnsact t JOIN strinfo s
ON t.store=s.store JOIN skuinfo si
ON t.sku=si.sku JOIN deptinfo d
ON si.dept=d.dept
WHERE t.stype='P' AND exclude_flag IS NULL AND
t.store||EXTRACT(YEAR from t.saledate)||EXTRACT(MONTH from t.saledate) IN
(SELECT store||EXTRACT(YEAR from saledate)||EXTRACT(MONTH from saledate)
FROM trnsact
GROUP BY store, EXTRACT(YEAR from saledate), EXTRACT(MONTH from saledate)
HAVING COUNT(DISTINCT saledate)>= 20)
GROUP BY s.city, s.state, d.deptdesc, t.store, exclude_flag
  
ORDER BY dip DESC;

```



***第 14 個問題
For each store, determine the month with the minimum average daily revenue (as defined in Teradata Week 5 Exercise Guide) . 
For each of the twelve months of the year, count how many stores' minimum average daily revenue was in that month. 
During which month(s) did over 100 stores have their minimum average daily revenue?***
************************************************************************
> 答案：
> August only
************************************************************************

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
SUM(amt)/count(distinct saledate) as dailyrev, 
row_number() OVER(PARTITION BY store ORDER BY dailyrev DESC) AS "Rank_total_Revenue"
FROM trnsact 
group by store, mm
where stype = 'p' 
having count(distinct saledate) > 20
QUALIFY Rank_total_Revenue=1) AS modified_trnsact;
```


***第 15 個問題Write a query that determines the month in which each store had its maximum number of sku units returned.
During which month did the greatest number of stores have their maximum number of sku units returned?***

************************************************************************
> 答案：
> - 只有December超過100, 有295 stores(我的版本)
************************************************************************


```
select count(case when mm = 1 then mm end) as jancnt
    ,count(case when mm = 2 then mm end) as febcnt
    ,count(case when mm = 3 then mm end) as marcnt
    ,count(case when mm = 4 then mm end) as aprcnt
    ,count(case when mm = 5 then mm end) as maycnt
    ,count(case when mm = 6 then mm end) as juncnt
    ,count(case when mm = 7 then mm end) as julycnt
    ,count(case when mm = 8 then mm end) as augcnt
    ,count(case when mm = 9 then mm end) as septcnt
    ,count(case when mm = 10 then mm end) as otccnt
    ,count(case when mm = 11 then mm end) as novcnt
    ,count(case when mm = 12 then mm end) as deccnt		
from (select distinct store,
    extract(month from saledate) as mm,
    count(sku) as numsku,
    row_number() over (partition by store order by numsku desc) as row_num
    from trnsact 
    group by mm,store
    where stype = 'r' and oreplace((extract(month from saledate) || extract(year from saledate)), ' ', '') not like '%82005%' 						 
    having count(distinct saledate) > 20			  
    qualify row_num = 1	
)as skucnt;	
```
		

**********************************************************************
// 上面版本是我的，下面是Coursera提供的版本
**********************************************************************
			

```
SELECT CASE when max_month_table.month_num = 1 then 'January' when
max_month_table.month_num = 2 then 'February' when
max_month_table.month_num = 3 then 'March' when
max_month_table.month_num = 4 then 'April' when
max_month_table.month_num = 5 then 'May' when
max_month_table.month_num = 6 then 'June' when
max_month_table.month_num = 7 then 'July' when
max_month_table.month_num = 8 then 'August' when
max_month_table.month_num = 9 then 'September' when
max_month_table.month_num = 10 then 'October' when
max_month_table.month_num = 11 then 'November' when
max_month_table.month_num = 12 then 'December' END, COUNT(*)
FROM (SELECT DISTINCT extract(year from saledate) as year_num, extract(month from saledate) as month_num, CASE when extract(year from saledate) = 2004 AND extract(month from saledate) = 8 then 'exclude' END as exclude_flag, store, SUM(quantity) AS tot_returns, ROW_NUMBER () over (PARTITION BY store ORDER BY tot_returns DESC) AS month_rank
FROM trnsact
WHERE stype='R' AND exclude_flag IS NULL AND store||EXTRACT(YEAR from saledate)||EXTRACT(MONTH from saledate) IN (SELECT store||EXTRACT(YEAR from saledate)||EXTRACT(MONTH from saledate)
FROM trnsact
GROUP BY store, EXTRACT(YEAR from saledate), EXTRACT(MONTH from saledate)
HAVING COUNT(DISTINCT saledate)>= 20)
GROUP BY store, month_num, year_num
QUALIFY month_rank=1) as max_month_table
GROUP BY max_month_table.month_num
ORDER BY max_month_table.month_num

```
