### Week3 graded quiz using Teradata

#### //***[Teradata Viewpoint](http://uatdviewpoint.waltoncollege.uark.edu/)**//

***Q3. On what day was Dillard’s income based on total sum of purchases the greatest***
 

```
Database ua_dillards;
SELECT t.saledate, SUM(t.amt) AS total_purchases
FROM trnsact t
GROUP BY t.saledate
ORDER BY total_purchases DESC;
```

> 2004-12-18

***Q4.What is the deptdesc of the departments that have the top 3 greatest numbers of skus from the skuinfo table associated with them?***

```
Database ua_dillards; 
SELECT d.deptdesc, d.dept,COUNT(s.sku) AS NumSku
FROM deptinfo d INNER JOIN skuinfo s 
ON s.dept=d.dept
GROUP BY d.deptdesc, d.dept
ORDER BY NumSku DESC
```


> 前三名，是INVEST, POLOMEN, BRIOSO

***Q5.Which table contains the most distinct sku numbers?***


```
Database ua_dillards; 
SELECT COUNT(DISTINCT sku)
FROM skstinfo;

SELECT COUNT(DISTINCT sku)
FROM trnsact;

SELECT COUNT(DISTINCT sku)
FROM skuinfo;
```


***Q6. How many skus are in the skstinfo table, but NOT in the skuinfo table?*** 


```
Database ua_dillards; 
SELECT count(sku.sku)
FROM skstinfo sks JOIN  skuinfo sku
ON sks.sku=sku.sku
WHERE sku.sku IS NULL;
```

> 0

***Q7:What is the average amount of profit Dillard’s made per day?*** 

```
Database ua_dillards; 
SELECT SUM(t.amt - s.cost)/COUNT(DISTINCT t.saledate) AS AvgProfit
FROM trnsact t JOIN skstinfo s ON t.sku = s.sku AND t.store = s.store
WHERE t.stype = 'P' AND s.cost IS NOT NULL ;
```
> 1527903

***Q8.The store_msa table provides population statistics about the geographic location around a store.*** 

- **Hint**:Using one query to retrieve your answer, how many MSAs are there within the state of North Carolina (abbreviated “NC”), and within these MSAs, 
- what is the lowest population level (msa_pop) and highest income level (msa_income)? 


```
Database ua_dillards; 
SELECT COUNT(store) AS numStore, MIN(msa_pop), MAX(msa_income)
FROM store_msa
WHERE state = ‘NC';
```


> - Numstore:16, 
> - Minimal Population:339511, 
> - Maximum income:36151

***Q9.What department (with department description), brand, style, and color brought in the greatest total amount of sales?***


```
Database ua_dillards;
SELECT d.dept, d.deptdesc As Department, 
s.brand AS Brand, s.style As Style, s.color As Color, max (t.amt) as Amount

FROM skuinfo s JOIN deptinfo d ON s.dept = d.dept
JOIN trnsact  t ON s.sku = t.sku
WHERE t.stype='P'
group by d.dept, Department, Brand, Style, Color
order by Amount desc;
```


> DEPT 4407 // 這題我是錯的, this choice is wrong!!

***Ｑ10. How many stores have more than 180,000 distinct skus associated with them in the skstinfo table?*** 


```
Database ua_dillards;
SELECT COUNT(DISTINCT store) AS Num_Store
FROM skstinfo
WHERE  Num_Store>180000;
```


***Error Code - 3569 
Error Message - [Teradata Database] [TeraJDBC 15.10.00.09] [Error 3569] [SQLState 42000] Improper use of an aggregate function in a WHERE Clause.***

結果發現，要兩個table做join


```
Database ua_dillards;
SELECT a.store, COUNT(DISTINCT b.sku) AS Num_Store
FROM strinfo a JOIN skstinfo b ON a.store = b.store
GROUP BY a.store
HAVING  Num_Store>180000;
```
> 最後，有12個stores 擁有超過180000 sku 

***Q11:Look at the data from all the distinct skus in the “cop” department with a “federal” brand and a “rinse wash” color.***
 
- You'll see that these skus have the same values in some of the columns, meaning that they have some features in common.
 
- In which columns do these skus have dierent values from one another, meaning that their features dier in the categories represented by the columns? 


```
Database ua_dillards;
SELECT DISTINCT s.sku AS sku, s.style, s.size, s.packsize, s.vendor
FROM skuinfo s  JOIN deptinfo d ON s.dept = d.dept 
WHERE d.deptdesc='cop' AND s.brand='federal' AND s.color='rinse wash’;
```


> style, size

***Q12:How many skus are in the skuinfo table, but NOT in the skstinfo table?***
 

```
Database ua_dillards;
SELECT COUNT(a.sku) AS NumOfSkus
FROM skuinfo a LEFT JOIN skstinfo b ON a.sku = b.sku
WHERE b.sku IS NULL;
```


> 803966

***Q13:In what city and state is the store that had the greatest total sum of sales?*** 


```
Database ua_dillards;
SELECT SUM(t.amt) AS totalSales,s.state, s.city 
FROM strinfo s JOIN trnsact t ON s.store=t.store
GROUP BY s.state, s.city
WHERE t.stype='p'
ORDER BY totalSales DESC;
```


> - city:Houston, 
> - totalSales:53812530

#### 其他人的評論
> - Look again at the question: In what city and state is the store that had.. So you are after the store with the highest sales. 

> - Your query doesn't calculate total revenues by store so can't generate the correct answer. Have another look at your GROUP BY statement.


***Q14:Given Table A (rst table to be entered in the query) and Table B (second table to be entered in the query) the query result shown below is a result of what kind of join?*** 
>  Left JOIN

***Q15:How many states have more than 10 Dillards stores in them?***
 

```
Database ua_dillards;
SELECT state, COUNT(store) AS numOfStore
FROM strinfo
GROUP BY state
HAVING numOfStore >10;
```
> 15

***Q16:What is the suggested retail price of all the skus in the “reebok” department with the “skechers” brand and a “wht/saphire” color?*** 
> $29.00

