# [ch060-秋招秘籍](https://github.com/fengxiuchen/LearnSQL/blob/main/ch05-SQL%E9%AB%98%E7%BA%A7%E5%A4%84%E7%90%86.md)
## 一、秘籍A
```
USE partical;
```
### 练习一：各部门工资最高的员工
##### 建表和插入数据
```
DROP TABLE IF EXISTS Employees;
CREATE TABLE Employees
(
Id INT,
Name VARCHAR(20),
Salary INT,
DepartmentID INT,
PRIMARY KEY (Id)
)
```

```
DROP TABLE IF EXISTS Departments;
CREATE TABLE Departments
(
Id INT,
Name VARCHAR(20),
PRIMARY KEY (Id)
)
```

```
INSERT INTO Employees
values(1,"Joe",70000,1),
(2,"Henry",80000,2),
(3,"Sam",60000,2),
(4,"Max",90000,1);
```

```
INSERT INTO Departments
VALUES(1,"IT"),
(2,"Sales");
```


#### 解答

##### 解法一
```
SELECT d.Name AS Department,e.Name AS Employee,Max(Salary)
FROM employees as e,departments as d
WHERE e.DepartmentId = d.Id
GROUP BY Department;
```

##### 解法二
```
SELECT d.Name AS Department,e.Name AS Employee,Max(Salary)
FROM employees as e
INNER JOIN departments as d
ON e.DepartmentId = d.Id
GROUP BY Department;
```


### 练习二：换座位

#### 建表和插入数据
```
DROP TABLE IF EXISTS seats;
CREATE TABLE seats
(
id INT,
student VARCHAR(20),
PRIMARY KEY (id)
);
```

```
INSERT INTO seats
VALUES(1,"Abbot"),
(2,"Doris"),
(3,"Emerson"),
(4,"Green"),
(5,"Jeames");
```


#### 解答

##### 解法：针对Id

###### IF套IF
```
SELECT IF(Id=b.COUNTS && mod(Id,2)<>0,Id,
IF(mod(Id,2)=0,Id-1,Id+1)
) AS Id,student
FROM seats,(SELECT COUNT(*) AS COUNTS FROM seats) AS b
ORDER BY Id;
```

###### CASE条件控制
```
SELECT 
CASE
WHEN MOD(id,2)<>0 AND id=b.COUNTS THEN id
WHEN MOD(id,2)=0 THEN id-1
ELSE id+1
END AS ID,student
FROM seats,(SELECT COUNT(*) AS COUNTS FROM seats) AS b
ORDER BY ID;
```
针对ID进行处理的话，主要难点在于边界的处理，因此优先判断是否为边界值并且边界值是否为奇数，若都符合条件，  
则返回原ID，接下来就方便处理，直接进行奇偶数的判断，并修改ID,因为是修改ID，最后重新排序一下；




### 练习三：分数排名

#### 建表和插入数据
```
DROP TABLE if exists score;
CREATE TABLE score_rank
(
class INT,
score_avg INT,
PRIMARY KEY(class)
);
```

```
INSERT INTO score_rank
VALUES(1,93),(2,93),(3,93),(4,91);
```

#### 解法一：ORDER BY，升序
```
SELECT * FROM score_rank ORDER BY score_avg ASC;
```

#### 解法二：ORDER BY，降序
```
SELECT * FROM score_rank ORDER BY score_avg DESC;
```

### 解法三：DENSE_RANK()，升序
```
SELECT score_avg,DENSE_RANK() OVER (ORDER BY score_avg ASC) FROM score_rank;
```

#### 解法四：DENSE_RANK()，降序
```
SELECT score_avg,DENSE_RANK() OVER (ORDER BY score_avg DESC) FROM score_rank;
```

#### 解法五：RANK(),升序
```
SELECT score_avg,RANK() OVER (ORDER BY score_avg ASC) FROM score_rank;
```

#### 解法六：RANK(),降序
```
SELECT score_avg,RANK() OVER (ORDER BY score_avg DESC) FROM score_rank;
```

#### 解法七：ROW_NUMBER(),升序
```
SELECT score_avg,ROW_NUMBER() OVER (ORDER BY score_avg ASC) FROM score_rank;
```

### 解法八：ROW_NUMBER(),降序
```
SELECT score_avg,ROW_NUMBER() OVER (ORDER BY score_avg DESC) FROM score_rank;
```


### 练习四：连续出现的数字

#### 建表和插入数据
```
DROP TABLE IF EXISTS cnums;
CREATE TABLE cnums
(id INT,
Num INT,
PRIMARY KEY(id))
```

```
INSERT INTO cnums
VALUES(1,1),
(2,1),
(3,1),
(4,2),
(5,1),
(6,2),
(7,2);
```


#### 解法
```
SELECT Num AS ConsecutiveNums,COUNT(Consecutive) AS counts
FROM cnums,(SELECT Id,RANK() OVER (ORDER BY Num) AS Consecutive FROM cnums) AS c
WHERE cnums.Id=c.Id
GROUP BY c.Consecutive
HAVING counts >= 3;
```


### 练习五：树节点

#### 建表和插入数据
```
DROP TABLE IF EXISTS tree;
CREATE TABLE tree
(id INT,
p_id INT,
PRIMARY KEY(id));
```

```
INSERT INTO tree
VALUES(1,NULL),
(2,1),
(3,1),
(4,2),
(5,2);
```


#### 解法
```
SELECT id,
CASE
WHEN p_id is NULL THEN "Root"
WHEN id IN (SELECT tree.p_id FROM tree WHERE p_id IS NOT NULL) THEN "Inner"
ELSE "Leaf"
END AS Type
FROM tree;
```


### 练习六：至少有五名直接下属的经理

#### 建表和插入数据
```
DROP TABLE IF EXISTS Employee;
CREATE TABLE Employee
(ID INT,
Name VARCHAR(20),
Department VARCHAR(1),
ManagerID INT,
PRIMARY KEY(ID));
```

```
INSERT INTO Employee
VALUES(101,"John","A",null),
(102,"Dan","A",101),
(103,"James","A",101),
(104,"Amy","A",101),
(105,"Anne","A",101),
(106,"Ron","B",101);
```


#### 解法
```
SELECT Name
FROM Employee AS e,(SELECT ManagerID,count(ManagerID)
FROM Employee
WHERE ManagerID IS NOT NULL
GROUP BY ManagerID
HAVING count(ManagerID)>=5) AS m
WHERE e.Id = m.ManagerID;
```


### 练习七：回答率最高的问题

#### 建表和插入数据
```
DROP TABLE IF EXISTS survey_log;
CREATE TABLE survey_log
(
uid INT,
action VARCHAR(20),
question_id INT,
answer_id INT,
q_num INT,
timestamp INT
)
```

```
INSERT INTO survey_log
VALUES(5,"show",285,null,1,123),
(5,"answer",285,124124,1,124),
(5,"show",369,null,2,125),
(5,"skip",369,null,2,126);
```


#### 解法
```
SELECT question_id AS survey_log
FROM survey_log
GROUP BY question_id
ORDER BY COUNT(answer_id)/count(IF(action="show",1,0)) DESC
LIMIT 1;
```


### 练习八：各部门工资前三高的员工
```
DROP TABLE IF EXISTS employees;
CREATE TABLE Employees
(
ID INT,
Name VARCHAR(20),
Salary INT,
DepartmentId INT,
PRIMARY KEY(ID)
);
```

```
INSERT INTO Employees
VALUES(1,"Joe",70000,1),
(2,"Henry",80000,2),
(3,"Sam",60000,2),
(4,"Max",90000,1),
(5,"Jnaet",69000,1),
(6,"Rnady",85000,1);
```


### 解法
```
SELECT 
d.name AS Department,
e.name AS Employee,
e.salary
FROM employees as e,departments as d
WHERE e.departmentId = d.Id
AND
3 > (SELECT COUNT(DISTINCT e2.salary)
FROM Employees as e2
WHERE e.salary < e2.salary
AND e.DepartmentID = e2.DepartmentID)
ORDER BY d.name,salary DESC;
```


### 练习题九：平面上最近的距离

#### 建表和插入数据
```
DROP TABLE IF EXISTS point_2d;
CREATE TABLE point_2d
(
x INT,
Y INT
);
```

```
INSERT INTO point_2d
VALUES(-1,-1),
(0,0),
(-1,-2);
```

#### 解法
```
SELECT 
ROUND(SQRT(MIN(POW(P1.X - P2.X,2)+POW(p1.y - p2.y,2)))) AS shortest
FROM point_2d p1
JOIN point_2d p2
ON p1.x != p2.x OR p1.y != p2.y;
```


### 练习题十：行程和用户

#### 建表和插入数据
```
DROP TABLE if EXISTS Trips;
CREATE TABLE Trips
(Id INT,
Client_Id INT,
Driver_Id INT,
City_Id INT,
Status VARCHAR(30),
Request_at DATE,
PRIMARY KEY (Id));
```

```
INSERT INTO Trips VALUES (1, 1, 10, 1, 'completed', '2013-10-1');
INSERT INTO Trips VALUES (2, 2, 11, 1, 'cancelled_by_driver', '2013-10-1');
INSERT INTO Trips VALUES (3, 3, 12, 6, 'completed', '2013-10-1');
INSERT INTO Trips VALUES (4, 4, 13, 6, 'cancelled_by_client', '2013-10-1');
INSERT INTO Trips VALUES (5, 1, 10, 1, 'completed', '2013-10-2');
INSERT INTO Trips VALUES (6, 2, 11, 6, 'completed', '2013-10-2');
INSERT INTO Trips VALUES (7, 3, 12, 6, 'completed', '2013-10-2');
INSERT INTO Trips VALUES (8, 2, 12, 12, 'completed', '2013-10-3');
INSERT INTO Trips VALUES (9, 3, 10, 12, 'completed', '2013-10-3');
INSERT INTO Trips VALUES (10, 4, 13, 12, 'cancelled_by_driver', '2013-10-3');
```

```
DROP TABLE if EXISTS Users ;
CREATE TABLE Users 
(Users_Id  INT,
 Banned    VARCHAR(30),
 Role      VARCHAR(30),
PRIMARY KEY (Users_Id));
```

```
INSERT INTO Users VALUES (1,    'No',  'client');
INSERT INTO Users VALUES (2,    'Yes', 'client');
INSERT INTO Users VALUES (3,    'No',  'client');
INSERT INTO Users VALUES (4,    'No',  'client');
INSERT INTO Users VALUES (10,   'No',  'driver');
INSERT INTO Users VALUES (11,   'No',  'driver');
INSERT INTO Users VALUES (12,   'No',  'driver');
INSERT INTO Users VALUES (13,   'No',  'driver');
```


#### 解法
```
SELECT
    request_at as 'Day', round(avg(Status!='completed'), 2) as 'Cancellation Rate'
FROM 
    trips t JOIN users u1 ON (t.client_id = u1.users_id AND u1.banned = 'No')
    JOIN users u2 ON (t.driver_id = u2.users_id AND u2.banned = 'No')
WHERE	
    request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY 
    request_at
```

## 二、秘籍B

## 三、秘籍C

没时间做呀~~~

## 写一写本次组队学习的收获和感受
虽然忙着工作，没时间仔细去学，但是真的感受到开源的精神和力量，也认识了一些高手和很负责的队长，下次还有其他喜欢的课程一定会继续参加
