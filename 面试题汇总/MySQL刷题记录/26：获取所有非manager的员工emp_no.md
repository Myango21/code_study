题目：
有一个员工表employees简况如下:
![013613CC3F594F2FB7444E6AD1DE4CDA](https://gitee.com/bruce_qiq/picture/raw/master/2021-9-29/1632848008360-013613CC3F594F2FB7444E6AD1DE4CDA.png)


有一个部门领导表dept_manager简况如下:
![EFBA0FC874C43A13F3732087E07217A6](https://gitee.com/bruce_qiq/picture/raw/master/2021-9-29/1632848022341-EFBA0FC874C43A13F3732087E07217A6.png)

请你找出所有非部门领导的员工emp_no，以上例子输出:
![1EB9F3AEE88291A8FF5D3E30063143CF](https://gitee.com/bruce_qiq/picture/raw/master/2021-9-29/1632848035412-1EB9F3AEE88291A8FF5D3E30063143CF.png)


数据表：
表结构
```mysql
drop table if exists  `employees` ; 
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
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

数据：
insert语句
```mysql
INSERT INTO dept_manager VALUES('d001',10002,'1996-08-03','9999-01-01');
INSERT INTO dept_manager VALUES('d002',10003,'1990-08-05','9999-01-01');
INSERT INTO employees VALUES(10001,'1953-09-02','Georgi','Facello','M','1986-06-26');
INSERT INTO employees VALUES(10002,'1964-06-02','Bezalel','Simmel','F','1985-11-21');
INSERT INTO employees VALUES(10003,'1959-12-03','Parto','Bamford','M','1986-08-28');
```

解题思路：

1、此题解题的思路，就是查找出部门领导中的员工编号，然后去员工表中查找，不在这部分员工编号中的数据即可。

参考答案：

```mysql
select emp_no from employees where emp_no not in 
(select emp_no from  dept_manager);
```
> 此题也有其他的解法，例如用left join 然后在使用is null。不是很推荐这种做法，这种查询的结构满足笛卡尔积，效率上更为低下一些。