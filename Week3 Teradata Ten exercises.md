#### 2018-09-10(一)

### Week3 Teradata exercise

- Exercise 1: (a) Use COUNT and DISTINCT to determine how many distinct skus there are in pairs of the skuinfo, skstinfo, and trnsact tables. Which skus are common to pairs of tables, or unique to specific tables? 


>Database ua_dillards;
>
> SELECT sku, retail,cost, COUNT(sku) 
>
> FROM skstinfo GROUP BY sku,retail, cost 


 SELECT sku, COUNT(sku) FROM skuinfo GROUP BY sku 
在skuinfo都只有一個單獨的sku 
> SELECT sku, COUNT(sku) FROM skstinfo GROUP BY sku 在skstnfo

一個sku 有很多pairs > SELECT sku, COUNT(sku) FROM trnsact GROUP BY sku 在trnsact table內， 一個sku 有很多pairs

- Exercise 1:(b) Use COUNT to determine how many instances there are of each sku associated with each store in the skstinfo table and the trnsact table? 

> Database ua_dillards; 
>
> SELECT sku, store, COUNT(sku), COUNT(store) FROM **skstinfo** 
>
> GROUP BY sku, store; 

> Database ua_dillards; 
>
> SELECT sku, store, 
> COUNT(sku),COUNT(store) FROM **trnsact** 
> GROUP BY sku,store;

### Week3 Teradata exercise

- Exercise 1: (a) Use COUNT and DISTINCT to determine how many distinct skus there are in pairs of the skuinfo, skstinfo, and trnsact tables. Which skus are common to pairs of tables, or unique to specific tables? 


>Database ua_dillards;
>
> SELECT sku, retail,cost, COUNT(sku) 
>
> FROM skstinfo GROUP BY sku,retail, cost 


 SELECT sku, COUNT(sku) FROM skuinfo GROUP BY sku 
在skuinfo都只有一個單獨的sku 
> SELECT sku, COUNT(sku) FROM skstinfo GROUP BY sku 在skstnfo

一個sku 有很多pairs > SELECT sku, COUNT(sku) FROM trnsact GROUP BY sku 在trnsact table內， 一個sku 有很多pairs

- Exercise 1:(b) Use COUNT to determine how many instances there are of each sku associated with each store in the skstinfo table and the trnsact table? 

> Database ua_dillards; 
>
> SELECT sku, store, COUNT(sku), COUNT(store) FROM **skstinfo** 
>
> GROUP BY sku, store; 

> Database ua_dillards; 
>
> SELECT sku, store, 
> COUNT(sku),COUNT(store) FROM **trnsact** 
> GROUP BY sku,store;

- Exercise 2: (a) Use COUNT and DISTINCT to determine how many distinct stores there are in the strinfo, store_msa, skstinfo, and trnsact tables. 

#### Answer 2(a) 

> SELECT COUNT(DISTINCT store) AS Num_strinfo_store FROM strinfo; 

453 

> SELECT COUNT(DISTINCT store) AS Num_store_msa_store FROM store_msa; 

333 
 
 
>  SELECT COUNT(DISTINCT store) AS Num_skstinfo_store FROM skstinfo; 
 
 357 
 
> SELECT COUNT(DISTINCT store) AS Num_trnsact_store FROM trnsact; 
 
 332 

- ANSWER 2(b) 既然要找，四個tables內，共同的store，那就用INNER JOIN，做多次的交集 接著，要比較四個tables，找出specific 的store數量(目前我不曉得怎麼”一次“比四個) 

> Database ua_dillards; 

> SELECT COUNT(DISTINCT a.store) FROM strinfo a LEFT JOIN skstinfo b ON a.store = b.store WHERE b.store IS NULL; 


> SELECT COUNT(DISTINCT a.store) FROM skstinfo a LEFT JOIN store_msa b ON a.store = b.store WHERE b.store IS NULL; 


> SELECT COUNT(DISTINCT a.store) FROM store_msa a LEFT JOIN trnsact b ON a.store = b.store WHERE b.store IS NULL; 

- Exercise 3: It turns out there are many skus in the trnsact table that are not in the skstinfo table. As a consequence, we will not be able to complete many desirable analyses of Dillard’s profit, as opposed to revenue, because we do not have the cost information for all the skus in the transact table **(recall that profit = revenue - cost)**. 

- Examine some of the rows in the trnsact table that are not in the skstinfo table; can you find any common features that could explain why the cost information is missing? 


##### ANSWER 3

> Database ua_dillards; 
>
> SELECT COUNT(t.sku) FROM trnsact t LEFT JOIN skstinfo s ON t.sku = s.sku WHERE s.sku IS NULL 

25951074 

> Database ua_dillards; 
> 
> SELECT COUNT(DISTINCT t.sku) FROM trnsact t LEFT JOIN skstinfo s ON t.sku = s.sku WHERE s.sku IS NULL 

171986

### Week3 Teradata exercise

#### Exercise 4: 
- Although we can’t complete all the analyses we’d like to on Dillard’s profit, we can look at general trends. What is Dillard’s average profit per day? 

> SELECT SUM(t.amt - s.cost)/COUNT(DISTINCT t.saledate) > AS AvgProfit
>
> FROM trnsact t JOIN skstinfo s ON t.sku = s.sku AND > t.store = s.store
>
> WHERE t.stype = 'P' AND s.cost IS NOT NULL AND t.register=640

- 這一題，我覺得關鍵要現把算式列出來，並且逐步拆解，首先，要算每天的平均收益，從收益出發，意味著要用“營收”扣掉“支出”，這個動作是要加總每一筆營收與支出的計算，接著再將此值除上“總營業天數”，這樣才能求出“每天平均收益”。

- 另外，這裡還需要標明t.stype是purchased，標記是被購買的型態，還有排除s.cost為空值的情況。

- Answer4:10779.20

#### Exercise 5: 
> - On what day was the total value (in $) of returned goods the greatest? On what day was the total number of individual returned items the greatest? 

##### 5.(a)在哪一天，退貨的總金額是最高的？

- If you are interested in looking at the total value of goods purchased or returned, use the “amt” field 

> Database ua_dillards; 
> SELECT t.saledate, SUM(s.cost) AS TotalCost, SUM(t.AMT) AS > TotalReturnGoodValue
>
> FROM trnsact t INNER JOIN skstinfo s ON  t.sku=s.sku AND t.store=s.store
> 
> WHERE t.stype='R'
>
> GROUP BY t.saledate
>
> ORDER BY TotalReturnGoodValue DESC;

- 上述發生錯誤，
Error Code - 3504 
Error Message - [Teradata Database] [TeraJDBC 15.10.00.09] [Error 3504] [SQLState HY000] Selected non-aggregate values must be part of the associated group.

- 我在想，應該是要加上GROUP BY t.saledate

- 修正後，SaleDate:2004-12-27,  Total value of goods is $1212071 

##### 5(b).在哪一天，退貨的項目數是最高的？

- If you are interested in looking at the total number of goods purchased or returned, use the “quantity” field. 

> Database ua_dillards;

> SELECT t.saledate, SUM(s.cost) AS TotalCost, SUM(t.quantity) AS 
TotalReturnGoodItem
>
>FROM trnsact t INNER JOIN skstinfo s ON  t.sku=s.sku AND t.store=s.store
>
> WHERE t.stype=‘R’
>
> GROUP BY t.saledate
>
> ORDER BY TotalReturnGoodItem DESC;

- Salsedate:2005-07-30, total number of goods is 36984


### Week3 Teradata exercise



#### Exercise 6: What is the maximum price paid for an item in our database? What is the minimum price paid for an item in our database? 

在我們資料庫中，物品單價最高的數字是多少？還有物品單價最低是多少？
目前的難題是，我到底要選 original price, sale price, retail price來判斷呢。我先選sprice, 他的描述寫sale price of the item stock,

> Database ua_dillards;
>
> SELECT t.sku, t.sprice
> FROM trnsact t INNER JOIN skstinfo s ON  t.sku=s.sku AND t.store=s.store
> 
> WHERE t.stype='P'
> GROUP BY t.sku
>
> ORDER BY t.sprice DESC;

Error Code - 3504 
Error Message - [Teradata Database] [TeraJDBC 15.10.00.09] [Error 3504] [SQLState HY000] Selected non-aggregate values must be part of the associated group.

> 去除GROUP BY t.sku後，本以為是正確數據，但是把ORDER BY 的DESC 換成ASC後，結果變得很奇怪。



不過，我查了其他人的做法是：
> Database ua_dillards;
> SELECT MIN(sprice) AS MaxPrice
> FROM trnsact;

不管是，max, min 的值，都得到相同的值

#### Exercise 7: How many departments have more than 100 brands associated with them, and what are their descriptions? 

我的想法是，先列出dept 與 該dept的brand數目

> Database ua_dillards;
> 
> SELECT  d.dept, COUNT(s.brand) AS NumOfBrand
> 
> FROM deptinfo d INNER JOIN skuinfo s ON  d.dept =s.dept
>
> GROUP BY d.dept
>
> HAVING NumOfBrand >100
>
> ORDER BY NumOfBrand DESC;


- 結果有60 rows，最小的brand數目是269
但是，核對其他人的指令結果後，我發現自己忽略一個問題，那就是沒有把重複品牌排除，

- 修正點：COUNT(DISTINCT s.brand)

#### Exercise 8: Write a query that retrieves the department descriptions of each of the skus in the skstinfo table. 

看到該題目，第一點是先看有涉及幾個tables, 並找出這些table的join關係，例如left join, inner join, outer join，結果看了表後，才發現不是題目提到2個table這麼簡單。

接著，在題目後段的提示
- ***You will need to join 3 tables in this query. 
Start by writing a query that connects the skstinfo and skuinfo tables. Once you are sure that query works, connect the deptinfo table.*** 

- ***You may want to explore what happens when you incorporate aggregate functions into your query. When you do so, make sure all of your column names are referenced by the correct table aliases.***

- ***If you have written your query correctly, you will find that the department description for sku #5020024 is “LESLIE”.***

> SELECT COUNT(DISTINCT a.sku)
>
> FROM skstinfo a LEFT JOIN skuinfo b ON a.sku=b.sku ;

- *答案是：760212

第二次調整後，
> Database ua_dillards;
> 
> SELECT a.sku, d.deptdesc
> FROM skstinfo a JOIN skuinfo b ON a.sku=b.sku 
> JOIN deptinfo d ON d.dept = b.dept
> 
> WHERE a.sku = 5020024;

- 輸出結果有 115 rows, 這應該把重複sku排除


### Week3 Teradata exercise

#### Exercise 9: What department (with department description), brand, style, and color had the greatest total value of returned items? 

我認為是要join三個table, skuinfo, transact, deptinfo

> Database ua_dillards;
> 
> SELECT d.dept, d.deptdesc, sku.brand, sku.style, sku.color, SUM(t.AMT) AS TotalReturnGoodValue
>
> FROM skuinfo sku JOIN deptinfo d ON sku.dept = d.dept JOIN trnsact  t ON sku.sku = t.sku
>
> WHERE t.stype=‘R’
>
> GROUP BY d.dept, d.deptdesc
>
> ORDER BY TotalReturnGoodValue DESC;


Error Code - 3504 
Error Message - [Teradata Database] [TeraJDBC 15.10.00.09] [Error 3504] [SQLState HY000] Selected non-aggregate values must be part of the associated group.

寫到著，我覺得我就困住了，接著看看提示 

***You will need to join 3 tables in this query. Start by writing a query that connects 2 tables. Once you are sure that query works, connect the 3rd table.***

***Make sure you include both of these fields in the SELECT and GROUP BY clauses. 
Make sure any non-aggregate column in your SELECT list is also listed in your GROUP BY clause.*** 

If you have written your query correctly, you will find that the department with the 5th highest total value of returned items is #2200 with a department description of “CELEBRT”, a brand of “LANCOME”, 
a style of “1042”, a color of “00-NONE”, and a total value of returned items of $177,142.50. 

> 後來發現，錯誤在於我的group by後面項目不夠，要在增加sku.brand, sku.style, sku.color

#### Exercise 10: In what state and zip code is the store that had the greatest total revenue during the time period monitored in our dataset? 

首先，先看會涉及哪些tables, trnsact與store_msa，

> SELECT s.state, s.zip, SUM(t.AMT) AS TotalReturnGoodValue
> 
> FROM trnsact t JOIN store_msa s ON t.store=s.store
>
> WHERE t.stype=‘p’
>
> GROUP BY s.state, s.zip
> ORDER BY TotalReturnGoodValue DESC












