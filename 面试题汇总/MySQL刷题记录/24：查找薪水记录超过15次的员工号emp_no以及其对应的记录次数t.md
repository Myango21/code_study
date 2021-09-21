题目：
有一个薪水表，salaries简况如下:
![8307279490CB9F89069769B3CDABC925](https://gitee.com/bruce_qiq/picture/raw/master/2021-9-21/1632230945628-8307279490CB9F89069769B3CDABC925.png)

请你查找薪水记录超过15次的员工号emp_no以及其对应的记录次数t，以上例子输出如下:
![85F393DA0762A39426D8D7C5958C8976](https://gitee.com/bruce_qiq/picture/raw/master/2021-9-21/1632230954530-85F393DA0762A39426D8D7C5958C8976.png)

数据表：
表结构
```mysql
drop table if exists  `salaries` ; 
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

数据：
insert语句
```mysql
INSERT INTO salaries VALUES(10001,60117,'1986-06-26','1987-06-26');
INSERT INTO salaries VALUES(10001,62102,'1987-06-26','1988-06-25');
INSERT INTO salaries VALUES(10001,66074,'1988-06-25','1989-06-25');
INSERT INTO salaries VALUES(10001,66596,'1989-06-25','1990-06-25');
INSERT INTO salaries VALUES(10001,66961,'1990-06-25','1991-06-25');
INSERT INTO salaries VALUES(10001,71046,'1991-06-25','1992-06-24');
INSERT INTO salaries VALUES(10001,74333,'1992-06-24','1993-06-24');
INSERT INTO salaries VALUES(10001,75286,'1993-06-24','1994-06-24');
INSERT INTO salaries VALUES(10001,75994,'1994-06-24','1995-06-24');
INSERT INTO salaries VALUES(10001,76884,'1995-06-24','1996-06-23');
INSERT INTO salaries VALUES(10001,80013,'1996-06-23','1997-06-23');
INSERT INTO salaries VALUES(10001,81025,'1997-06-23','1998-06-23');
INSERT INTO salaries VALUES(10001,81097,'1998-06-23','1999-06-23');
INSERT INTO salaries VALUES(10001,84917,'1999-06-23','2000-06-22');
INSERT INTO salaries VALUES(10001,85112,'2000-06-22','2001-06-22');
INSERT INTO salaries VALUES(10001,85097,'2001-06-22','2002-06-22');
INSERT INTO salaries VALUES(10001,88958,'2002-06-22','9999-01-01');
INSERT INTO salaries VALUES(10002,72527,'1996-08-03','1997-08-03');
```

解题思路：

1、用COUNT()函数和GROUP BY语句可以统计同一emp_no值的记录条数。

2、根据题意，输出的变动次数为t，故用AS语句将COUNT(emp_no)的值转换为t。

3、由于COUNT()函数不可用于WHERE语句中，故使用HAVING语句来限定t>15的条件。

参考答案：

```mysql
select emp_no, count(*) as t from salaries group by emp_no having t > 15;
```