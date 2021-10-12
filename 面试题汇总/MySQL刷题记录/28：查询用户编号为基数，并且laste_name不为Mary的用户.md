题目：
有一个员工表dept_emp简况如下:
![1](https://gitee.com/bruce_qiq/picture/raw/master/2021-10-12/1634050478641-1.png)

请你查找employees表所有emp_no为奇数，且last_name不为Mary的员工信息，并按照hire_date逆序排列，以上例子查询结果如下:
![2](https://gitee.com/bruce_qiq/picture/raw/master/2021-10-12/1634050492840-2.png)


数据表：
表结构
```mysql
drop table if exists  `employees` ; 
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

数据：
insert语句
```mysql
INSERT INTO employees VALUES(10001,'1953-09-02','Georgi','Facello','M','1986-06-26');
INSERT INTO employees VALUES(10002,'1964-06-02','Bezalel','Simmel','F','1985-11-21');
INSERT INTO employees VALUES(10003,'1959-12-03','Bezalel','Mary','M','1986-08-28');
INSERT INTO employees VALUES(10004,'1954-05-01','Chirstian','Koblick','M','1986-12-01');
INSERT INTO employees VALUES(10005,'1953-11-07','Mary','Sluis','F','1990-01-22');
```

解题思路：

1、此题非常简单，用户的编号为奇数。直接%2=1就可以了。可以考虑内置函数、位运算的解题方式。


参考答案：

```mysql
explain select * from employees where emp_no % 2 != 0 and last_name != 'Mary' order by hire_date desc;

select * from employees where emp_no & 1 and last_name != 'Mary' order by hire_date desc;

select * from employees where MOD(emp_no, 1) = 1 and last_name != 'Mary' order by hire_date desc;
```
> 此题通过explain走的是全盘扫描，可以考虑一下怎么不走全盘扫描。