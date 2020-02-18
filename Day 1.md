# 牛客网 SQL 实战 | Day1

[题目来源](https://www.nowcoder.com/ta/sql)

## 大纲

| 题号 |                  知识点                   |
| :--: | :---------------------------------------: |
|  1   |                MAX() 函数                 |
|  2   |          LIMIT 和 OFFSET 的用法           |
|  3   |           INNER JOIN 连接两张表           |
|  4   |           INNER JOIN 连接两张表           |
|  5   |           LEFT JOIN 连接两张表            |
|  6   | INNER JOIN / 并列查询 / ORDER BY 逆序排列 |

## 题目

### 1. 查找最晚入职员工的所有信息

#### 题目描述

查找最晚入职员工的所有信息。

```mysql
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

#### 思路

用 `MAX()`函数找出“最晚”的记录。[MAX() 函数用法参考](https://wiki.jikexueyuan.com/project/mysql/useful-functions/max.html)

#### 代码

```mysql
mysql> SELECT * FROM employees
    -> WHERE hire_date = (SELECT MAX(hire_date) FROM employees);
```

### 2. 查找入职员工时间排名倒数第三的员工所有信息

#### 题目描述

查找入职员工时间排名倒数第三的员工所有信息。

```mysql
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

#### 思路

1. 用 `LIMIT` 和 `OFFSET` 找出「倒数第三」条记录。`Limit` 与 `offset` 一起使用的用法，举个例子：`LIMIT 3 OFFSET 1`， 这意味着，跳过第1条记录（即从第2条记录开始），返回接下来 3 条记录。即最终得到，原本的第 2、3 和 4 条记录。
2. 注意，用 `distinct` 去重，因为可能有多个重复的日期。

#### 代码

```mysql
mysql> SELECT * FROM employees
    -> WHERE hire_date =
    -> (SELECT hire_date FROM employees ORDER BY hire_date DESC LIMIT 1 OFFSET 2);
```

### 3. 查找当前薪水详情以及部门编号 dept_no

#### 题目描述

查找各个部门当前 (to_date='9999-01-01') 领导当前薪水详情以及其对应部门编号 dept_no。

```mysql
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));

CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

#### 思路

1. 两张表内连接（`inner join`），通过 `emp_no` 关联；
2. 两个“当前” —— 当前领导、当前薪水，用 `to_date = “9999-01-01” `限制；
3. 关于几种连接方式：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190609110535905.png)

4. 例子：

```mysql
CREATE TABLE `TableA` (
`id` int(11) NOT NULL,
`name` varchar(16) NOT NULL,
PRIMARY KEY (`id`));

CREATE TABLE `TableB` (
`id` int(11) NOT NULL,
`name` varchar(16) NOT NULL,
PRIMARY KEY (`id`));

INSERT INTO TableA (id, name) VALUES (1, 'Pirate');
INSERT INTO TableA (id, name) VALUES (2, 'Monkey');
INSERT INTO TableA (id, name) VALUES (3, 'Ninja');
INSERT INTO TableA (id, name) VALUES (4, 'Spaghetti');

INSERT INTO TableB VALUES(1, 'Rutabaga');
INSERT INTO TableB VALUES(2, 'Pirate');
INSERT INTO TableB VALUES(3, 'Darth Vade');
INSERT INTO TableB VALUES(4, 'Ninja');
```

* INNER OUTER JOIN

```mysql
SELECT * FROM TableA INNER JOIN TableB ON TableA.name = TableB.name;
```

* LEFT OUTER JOIN

```mysql
SELECT * FROM TableA LEFT OUTER JOIN TableB ON TableA.name = TableB.name;
```

* RIGHT OUTER JOIN

```mysql
SELECT * FROM TableA RIGHT OUTER JOIN TableB ON TableA.name = TableB.name;
```

* FULL OUTER JOIN

```mysql
mysql> SELECT * FROM TableA LEFT OUTER JOIN TableB ON TableA.name = TableB.name    
		-> UNION
    -> SELECT * FROM TableA RIGHT OUTER JOIN TableB
    -> ON TableA.name = TableB.name;
```

#### 代码

```mysql
mysql> SELECT s.*, dm.dept_no
    -> FROM salaries AS s INNER JOIN dept_manager as dm
    -> ON s.emp_no = dm.emp_no
    -> WHERE s.to_date='9999-01-01' AND dm.to_date='9999-01-01';
```

### 4. 查找所有已经分配部门的员工的 last_name 和 first_name

#### 题目描述

查找所有已经分配部门的员工的 last_name 和 first_name 以及 dept_no。

```mysql
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));

CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

#### 思路

为了筛选出「已经分配部门的员工 」，用 `INNER JOIN` 通过关键字 `emp_no` 连接两张表即可。类似题 3。

#### 代码

```mysql
mysql> SELECT e.last_name, e.first_name, de.dept_no
    -> FROM employees AS e INNER JOIN dept_emp AS de
    -> ON e.emp_no = de.emp_no;
```

### 5. 查找所有员工的 last_name 和 first_name 以及对应部门编号 dept_no

#### 题目描述

查找所有员工的 last_name 和 first_name 以及对应部门编号 dept_no，也包括展示没有分配具体部门的员工。

```mysql
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));

CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

#### 思路

1. 区别于题目 4， 「也包括展示没有分配具体部门的员工」，此时，用 `LEFT JOIN` 连接两张表。employees 表左连接 dept_emp 表，会返回 employees 表的全部数据，即便 dept_emp 表无对应数据。
2. `JOINS` 的用法可查看题目 3 的参考资料。

#### 代码

```mysql
mysql> SELECT e.last_name, e.first_name, de.dept_no
    -> FROM employees AS e LEFT JOIN dept_emp AS de
    -> ON e.emp_no = de.emp_no;
```

### 6. 查找所有员工入职时候的薪水情况

####  题目描述

查找所有员工入职时候的薪水情况，给出 emp_no 以及 salary， 并按照 emp_no 进行逆序。

```mysql
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));

CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

#### 思路

1. 用 emp_no 关键字连接两张表；
2. 「所有员工入职时候的薪水情况」，即通过 employees 表的 hire_date = salaries 表的 from_date 限制；
3. 「按照emp_no进行逆序」，即用 `ORDER BY`  对 emp_no 进行排序。

#### 代码

```mysql
mysql> SELECT e.emp_no, s.salary
    -> FROM employees AS e
    -> INNER JOIN salaries AS s
    -> ON e.emp_no = s.emp_no
    -> WHERE e.hire_date = s.hire_date
    -> ORDER BY e.emp_no DESC;
```

内连接是取左右两张表的交集形成一个新表，用 `FROM` 并列两张表后仍然还是两张表。如果还要对新表进行操作则要用内连接。从效率上看应该 `FROM` 并列查询比较快，因为不用形成新表。本题从效果上看两个方法没区别。

```mysql
SELECT e.emp_no, s.salary
FROM employees AS e, salaries AS s
WHERE e.emp_no = s.emp_no AND e.hire_date = s.from_date
ORDER BY e.emp_no DESC
```

