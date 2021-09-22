题目：
有一个薪水表，salaries简况如下:
![0B07E4326822CEB36DFC0A8856794250](https://gitee.com/bruce_qiq/picture/raw/master/2021-9-22/1632321864847-0B07E4326822CEB36DFC0A8856794250.png)

请你找出所有员工具体的薪水salary情况，对于相同的薪水只显示一次,并按照逆序显示，以上例子输出如下:

![D272D88115F2A8870C9D588A098CDD57](https://gitee.com/bruce_qiq/picture/raw/master/2021-9-22/1632321847755-D272D88115F2A8870C9D588A098CDD57.png)


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
INSERT INTO salaries VALUES(10001,72527,'2002-06-22','9999-01-01');
INSERT INTO salaries VALUES(10002,72527,'2001-08-02','9999-01-01');
INSERT INTO salaries VALUES(10003,43311,'2001-12-01','9999-01-01');
```

解题思路：

1、此题考查的知识点就是一个去重问题，一般我们是使用distinct或者使用group by来实现。

2、在数据量大的情况看下，distinct效率比group by低，因此推荐养成一个习惯使用group by来实现。

参考答案：

```mysql
select salary from salaries  group by salary order by salary desc;
```