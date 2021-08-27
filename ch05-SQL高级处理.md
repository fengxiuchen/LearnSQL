# [ch05-集合运算](https://github.com/datawhalechina/wonderful-sql/blob/main/ch05:%20SQL%E9%AB%98%E7%BA%A7%E5%A4%84%E7%90%86.md)
## 一、窗口函数
### （一）窗口函数概念及其使用
#### 1、窗口函数概念
窗口函数也称为OLAP函数。OLAP 是 OnLine AnalyticalProcessing 的简称，意思是对数据库数据进行实时分析处理。  
为了便于理解，称之为 窗口函数。常规的SELECT语句都是对整张表进行查询，而窗口函数可以让我们有选择的去某一部分数据进行汇总、计算和排序。  
窗口函数的通用形式：
```
<窗口函数> OVER ([PARTITION BY <列名>]
                     ORDER BY <排序用列名>)  
```
#### 2、窗口函数使用
```
SELECT product_name
       ,product_type
       ,sale_price
       ,RANK() OVER (PARTITION BY product_type
                         ORDER BY sale_price) AS ranking
  FROM product  
```

## 二、能够作为窗口函数使用的函数
大致来说，窗口函数可以分为两类。  
一是 将SUM、MAX、MIN等聚合函数用在窗口函数中  
二是 RANK、DENSE_RANK等排序用的专用窗口函数  
### （一）、专用窗口函数
- RANK函数
计算排序时，如果存在相同位次的记录，则会跳过之后的位次。  
例）有 3 条记录排在第 1 位时：1 位、1 位、1 位、4 位……  

- DENSE_RANK函数
同样是计算排序，即使存在相同位次的记录，也不会跳过之后的位次。  
例）有 3 条记录排在第 1 位时：1 位、1 位、1 位、2 位……  

- ROW_NUMBER函数
赋予唯一的连续位次。  
例）有 3 条记录排在第 1 位时：1 位、2 位、3 位、4 位  

运行以下代码：
```
SELECT  product_name
       ,product_type
       ,sale_price
       ,RANK() OVER (ORDER BY sale_price) AS ranking
       ,DENSE_RANK() OVER (ORDER BY sale_price) AS dense_ranking
       ,ROW_NUMBER() OVER (ORDER BY sale_price) AS row_num
  FROM product  
```
### （二）、聚合函数在窗口函数上的使用
聚合函数在开窗函数中的使用方法和之前的专用窗口函数一样，只是出来的结果是一个累计的聚合函数值。  
运行以下代码：
```
SELECT  product_id
       ,product_name
       ,sale_price
       ,SUM(sale_price) OVER (ORDER BY product_id) AS current_sum
       ,AVG(sale_price) OVER (ORDER BY product_id) AS current_avg  
  FROM product;  
```
## 三、计算移动平均值
在上面提到，聚合函数在窗口函数使用时，计算的是累积到当前行的所有的数据的聚合。 实际上，还可以指定更加详细的汇总范围。该汇总范围成为框架(frame)。  
语法：
```
<窗口函数> OVER (ORDER BY <排序用列名>
                 ROWS n PRECEDING )  
                 
<窗口函数> OVER (ORDER BY <排序用列名>
                 ROWS BETWEEN n PRECEDING AND n FOLLOWING)
```
PRECEDING（“之前”）， 将框架指定为 “截止到之前 n 行”，加上自身行  
FOLLOWING（“之后”）， 将框架指定为 “截止到之后 n 行”，加上自身行  
BETWEEN 1 PRECEDING AND 1 FOLLOWING，将框架指定为 “之前1行” + “之后1行” + “自身”  
执行以下代码：
```
SELECT  product_id
       ,product_name
       ,sale_price
       ,AVG(sale_price) OVER (ORDER BY product_id
                               ROWS 2 PRECEDING) AS moving_avg
       ,AVG(sale_price) OVER (ORDER BY product_id
                               ROWS BETWEEN 1 PRECEDING 
                                        AND 1 FOLLOWING) AS moving_avg  
  FROM product  
```
## 四、GROUPING运算符
### （一）、ROLLUP - 计算合计及小计
常规的GROUP BY 只能得到每个分类的小计，有时候还需要计算分类的合计，可以用 ROLLUP关键字。
```
SELECT  product_type
       ,regist_date
       ,SUM(sale_price) AS sum_price
  FROM product
 GROUP BY product_type, regist_date WITH ROLLUP 
```









