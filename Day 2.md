# 牛客网 SQ L实战 | Day2

[题目来源](https://www.nowcoder.com/ta/sql)

## 大纲

| 题号 |                  知识点                  |
| :--: | :--------------------------------------: |
|  7   |   COUNT() 函数、GROUP BY 、HAVING 子句   |
|  8   | DISTINCT（GROUP BY去重的用法）、ORDER BY |
|  9   |          INNER JOIN / 并列查询           |
|  10  |       LEFT JOIN / NOT IN / IS NULL       |
|  11  |              “不等于”的用法              |
|  12  |             MAX() / GROUP BY             |

## 题目

### 7. 查找薪水涨幅超过 15 次的员工号 emp_no 以及其对应的涨幅次数 t

#### 题目描述

查找薪水涨幅超过 15 次的员工号 emp_no 以及其对应的涨幅次数 t。

```mysql
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

#### 思路

1. 「涨幅超过15次」，用 `COUNT()` 函数计数，搭配 `GROUP BY` 使用，因为是按照对每个员工进行分组后，组内统计；
2. 根据输出描述，对 `COUNT()` 计算的结果，命名为 t；
3. 在 `GROUP BY` 子句中使用 `HAVING`，限制 t > 15 ，返回满足条件的 emp_no。
4. `WHERE` 语句在 `GROUPBY` 语句之前；SQL 会在分组之前计算 `WHERE` 语句。
5. `HAVING` 语句在 `GROUPBY` 语句之后；SQL 会在分组之后计算 `HAVING` 语句。

#### 代码

```mysql
SELECT emp_no, COUNT(emp_no) AS t
FROM salaries
GROUP BY emp_no
HAVING t > 15
```

### 8. 找出所有员工当前薪水 salary 情况

#### 题目描述

找出所有员工当前 (to_date=‘9999-01-01’) 具体的薪水 salary 情况，对于相同的薪水只显示一次,并按照逆序显示。

```mysql
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

#### 思路

1. 「对于相同的薪水只显示一次」，用 `DISTINCT` 去重；
2. 「按照逆序显示」，用 `ORDER BY` 排序。

#### 代码

```mysql
SELECT DISTINCT salary FROM salaries
WHERE to_date = '9999-01-01'
ORDER BY salary DESC
```

大数据量时，`DISTINCT` 的效率不高，可以用 `GROUP BY` 解决重复问题。

```mysql
SELECT salary FROM salaries
WHERE to_date = '9999-01-01'
GROUP by salary
ORDER BY salary DESC
```

### 9. 获取所有部门当前 manager 的当前薪水情况

#### 题目描述

获取所有部门当前 manager 的当前薪水情况，给出 dept_no, emp_no 以及 salary，当前表示 to_date='9999-01-01'。

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

1. `INNER JOIN` 连接两张表，或用并列查询；
2. 两个「当前」——当前 manager、当前薪水。

#### 代码

* 并列查询

```mysql
SELECT dm.dept_no, dm.emp_no, s.salary
FROM dept_manager AS dm, salaries AS s
WHERE dm.emp_no = s.emp_no AND dm.to_date = '9999-01-01' AND s.to_date = '9999-01-01'
```

* `INNER JOIN`

```mysql
SELECT dm.dept_no, dm.emp_no, s.salary
FROM dept_manager AS dm INNER JOIN salaries AS s
ON dm.emp_no = s.emp_no 
WHERE dm.to_date = '9999-01-01' AND s.to_date = '9999-01-01'
```

### 10. 获取所有非 manager 的员工 emp_no

#### 题目描述

获取所有非 manager 的员工 emp_no。

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

1. 思路一：employees 表 `LEFT JOIN` dept_emp 表，通过判断 dept_emp对应为空的记录，可以筛选出 「非 manager 的员工」。关于 **IS NULL** 和 = NULL：
   1）“ = NULL ” 是错误的语法！
   2） SQL中，使用 is NULL 或 is not NULL判断字段是否为空
2. 思路二：在表 employees 中排除 dept_emp 表中的记录，用 `NOT IN` 字段。

#### 代码

* 方法一：IS NULL

```mysql
SELECT e.emp_no FROM employees AS e
LEFT JOIN dept_manager AS dm
ON e.emp_no = dm.emp_no
WHERE dm.dept_no IS NULL;
```

* 方法二： NOT IN

```mysql
SELECT emp_no FROM employees
WHERE emp_no NOT IN (SELECT emp_no FROM dept_manager)
```

### 11. 获取所有员工当前的 manager

#### 题目描述

获取所有员工当前的 manager，如果当前的 manager 是自己的话结果不显示，当前表示to_date='9999-01-01'。
结果第一列给出当前员工的 emp_no,第二列给出其 manager 对应的 manager_no。

```mysql
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));

CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
```

#### 思路

1. `INNER JOIN` 通过关键字 dept_no 连接两张表；
2. 「如果当前的 manager 是自己的话结果不显示」，即 dept_manager 表的 emp_no 与 dept_emp 表的 emp_no 相同的记录，应去掉。所有数据库都支持 `<>` 表示“不等于”， 大部分数据库支持” != “；建议使用 `<> `。

#### 代码

```mysql
SELECT de.emp_no, dm.emp_no
FROM dept_emp AS de INNER JOIN dept_manager AS dm
ON de.dept_no = dm.dept_no
WHERE de.to_date='9999-01-01' AND dm.to_date='9999-01-01' AND de.emp_no <> dm.emp_no
```

### 12. 获取所有部门中当前员工薪水最高的相关信息

#### 题目描述

获取所有部门中当前员工薪水最高的相关信息，给出 dept_no, emp_no 以及其对应的salary。

```mysql
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));

CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
```

#### 思路

1. 「薪水最高」，即 `MAX(salary)`；
2. 根据输出描述，可见需按照 dept_no 分组，即 `GROUP BY dept_no`；
3. 两张表通过 `INNER JOIN` 连接。

#### 代码

```mysql
SELECT de.dept_no, de.emp_no, MAX(salary)
FROM dept_emp AS de INNER JOIN salaries AS s
ON de.emp_no = s.emp_no
WHERE de.to_date = '9999-01-01' AND s.to_date = '9999-01-01'
GROUP BY de.dept_no
```

