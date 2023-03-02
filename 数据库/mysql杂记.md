# mysql 数据库

```note
主要是记录一下mysql数据使用方面的一些知识点，以防自己忘记了
```



## 基本使用

``` mysql
//查看你的数据库
show databases;

//使用一个数据库
use [数据库名];
----
use Van;

//创建一个数据库
create database [数据库名];
----
create database Van;

//查看数据库下有哪些表，首先要use某个数据库
show tables;
----
use Van;
show tables;

//在某个数据库内通过sql文件导入数据，首先要use某个数据库
source [sql文件路径]	//注意：文件路径不要有中文！
----
use Van;
source E:\VanSama no note\database\bjpowernode.sql;

//查看某个表的全部数据
select * from [表名]
----
select * from dept;
+--------+------------+----------+
| DEPTNO | DNAME      | LOC      |
|     10 | ACCOUNTING | NEW YORK |
|     20 | RESEARCH   | DALLAS   |
|     30 | SALES      | CHICAGO  |
|     40 | OPERATIONS | BOSTON   |
+--------+------------+----------+

//查看某个表的结构
desc [表名]
----
desc dept;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| DEPTNO | int(2)      | NO   | PRI | NULL    |       |
| DNAME  | varchar(14) | YES  |     | NULL    |       |
| LOC    | varchar(13) | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+

//查看mysql数据库的版本号
select version();

//查看当前使用的是哪个数据库
select database();
```



## 数据库的基本单元：表

数据库当中最基本的单元是表：table

| 姓名 | 性别 | 年龄 |
| :--: | :--: | :--: |
| 张三 |  男  |  20  |
| 李四 |  女  |  21  |
| 王五 |  男  |  22  |

任何一张表都有行和列：

​	行（row）：称为数据/记录

​	列（column）：称为字段

姓名字段、性别字段、年龄字段

每一个字段都有：字段名、数据类型、约束等属性





## SQL语句的分类

SQL语句很多，最好进行分门别类，容易记忆

- **DQL**：
  - 数据查询语言（凡是带有select关键字的都是查询语言）
  - select……
- **DML**：
  - 数据操作语言（凡是对**表当中的数据**进行增删改的都是DML）
  - 增 insert…… 
  - 删 delete……
  - 改 update…… 
- DDL：
  - 数据定义语言（凡是带有create、drop、alter的都是DDL）
  - DDL主要操作的是**表的结构**，不是表的数据，主要是对字段、或者整个表进行结构的操作
  - create：新建
  - alter：修改
  - drop：删除
- TCL：
  - 事务控制语言
  - 事务提交：commit
  - 事务回滚：rollback
- DCL：
  - 数据控制语言
  - 授权：grant
  - 撤销权限：revoke



# -------- DQL --------

## 简单查询

### 1. 查询一个字段

```mysql
select 字段名 from 表名;
----
mysql> select dname from dept;
+------------+
| dname      |
+------------+
| ACCOUNTING |
| RESEARCH   |
| SALES      |
| OPERATIONS |
+------------+
```

注意：

​	select 和 from 都是关键字

​	字段名和表名都是标识符。

强调：

​	对于SQL语句来说，是通用的。

​	所有SQL语句“;”结尾。

​	SQL语句不区分大小写

### 2. 查询多个字段

```mysql
//使用逗号","隔开字段即可
select 字段名1, 字段名2 from 表名;
----
mysql> select deptno, dname from dept;
+--------+------------+
| deptno | dname      |
+--------+------------+
|     10 | ACCOUNTING |
|     20 | RESEARCH   |
|     30 | SALES      |
|     40 | OPERATIONS |
+--------+------------+
```

### 3. 查询所有字段

```mysql
//1.可以把每个字段都写上
select a,b,c,d,e,f... from 表名

//2.使用*号
select * from 表名
/*这种方法存在缺点：
//1. 效率低
//2. 可读性差
//实际开发中不建议，可以自己玩
//你可以在dos命令窗口快速查看全表数据可以采用这种方式*/
```

### 4. 给查询的列起别名

```mysql
/*使用as关键字起别名
//只是显示的时候字段名字变了，实际上原表字段名没变
//select语句永远不会进行更改操作*/
select 字段名 as 字段别名 from 表名;
//不省略as写法
select deptno, dname as deptname from dept;
//省略as写法
select deptno, dname deptname from dept;
//如果别名有空格，使用单引号
//注意：数据库中的字符串都是单引号括起来，双引号不标准
select deptno, dname 'dept name' from dept;
```

### 5. 列参与数学运算

```mysql
//字段可以参与数学运算
mysql> select ename, sal*12 from emp;
+--------+----------+
| ename  | sal*12   |
+--------+----------+
| SMITH  |  9600.00 |
| ALLEN  | 19200.00 |
| WARD   | 15000.00 |
| JONES  | 35700.00 |
| MARTIN | 15000.00 |
| BLAKE  | 34200.00 |
| CLARK  | 29400.00 |
| SCOTT  | 36000.00 |
| KING   | 60000.00 |
| TURNER | 18000.00 |
| ADAMS  | 13200.00 |
| JAMES  | 11400.00 |
| FORD   | 36000.00 |
| MILLER | 15600.00 |
+--------+----------+
//原表
mysql> select ename, sal from emp;
+--------+---------+
| ename  | sal     |
+--------+---------+
| SMITH  |  800.00 |
| ALLEN  | 1600.00 |
| WARD   | 1250.00 |
| JONES  | 2975.00 |
| MARTIN | 1250.00 |
| BLAKE  | 2850.00 |
| CLARK  | 2450.00 |
| SCOTT  | 3000.00 |
| KING   | 5000.00 |
| TURNER | 1500.00 |
| ADAMS  | 1100.00 |
| JAMES  |  950.00 |
| FORD   | 3000.00 |
| MILLER | 1300.00 |
+--------+---------+
```

## 条件查询

- 语法格式：

  - select 字段... from 表名 where **条件**;

- 都有哪些条件？

  ```mysql
  /*1. = ：等于
  //2. <> 或 != ：不等于
  //3. < ：小于
  //4. <= ：小于等于
  //5. > ：大于
  //6. >= ：大于等于*/
  
  select empno,ename,sal from emp where sal (条件符号) 数字;
  ---------------------------------------------------------------------
   
  /*7. between ... and ... 两个值之间，等同于 >= and <=*/
  select empno,ename,sal from emp where sal >= 2450 and sal <= 3000;
  
  select empno,ename,sal from emp where sal between 2450 and 3000;
  ---------------------------------------------------------------------
  
  /*8. is null :为 null (is not null 不为空)*/
  //在数据库中，null不能使用等号连接，需要使用is null，因为数据库中的null代表什么也没有，不是一个值
  select empno,ename,sal,comm from emp where comm is null;
  
  select empno,ename,sal,comm from emp where comm is not null;
  ---------------------------------------------------------------------
  
  /*9. and ：并且*/
  select empno,ename,job,sal from emp where job = 'manager' and sal > 2500;
  ---------------------------------------------------------------------
  
  /*10. or ：或者*/
  //查询工作岗位是Manager和Salesman的员工
  select empno,ename,job,sal from emp where job = 'manager' or job = 'salesman';
  /*注意and 和 or的优先级，and比or优先级高，根据情况可以加括号*/
  ---------------------------------------------------------------------
  
  
  /*11. in ：包含，相当于多个or（not in 不在这个范围中）*/
  //查询工作岗位是manager和salesman的员工
  //or写法
  select empno,ename,job from emp where job = 'manager' or job = 'salesman';
  //in写法
  select empno,ename,job from emp where job in('manager', 'salesman');
  
  //注意：in不是一个区间，in后面跟具体的值
  //查询薪资是800和5000的员工信息
  select ename,sal from emp where sal in(800, 5000);
  
  //not in 表示不在这几个值当中的数据
  select ename,sal from emp where sal not in(800, 5000, 3000);
  ---------------------------------------------------------------------
  
  /*12. not ：not 可以取非，主要用在is或in中*/
  // is null
  // is not null
  // in
  // not in
  
  /*13. like ：like称为模糊查询，支持%或下划线匹配*/
  //'%'匹配任意多个字符
  //找出名字含有o的
  select ename from emp where ename like '%o%';
  //找出名字以T结尾的
  select ename from emp where ename like '%T';
  //找出名字以K开始的
  select ename from emp where ename like 'K%';
  ---------------------------------------------------------------------
  
  //一个下划线'_'只匹配一个字符
  //找出第二个字母是A的
  select ename from emp where ename like '_A%';
  //找出第三个字符是R的
  select ename from emp where ename like '__R%';
  
  //当查找的内容存在'_'的时候，可以使用转义字符'\'
  select name from t_student where name like '%\_%';
  ---------------------------------------------------------------------
  
  ```

  

## 排序

- 语法格式：

  - select 字段名 from 表名 **order by** 字段名; 默认升序，也可以加'asc'说明
  - select 字段名 from 表名 order by 字段名 **desc**; 加上desc可以实现降序

- ```mysql
  /*多字段排序*/
  select ename,sal from emp order by sal asc, ename asc;
  //注意：sal在前，起主导，只有sal相等的时候，才会考虑启用ename排序。
  +--------+---------+
  | ename  | sal     |
  +--------+---------+
  | SMITH  |  800.00 |
  | JAMES  |  950.00 |
  | ADAMS  | 1100.00 |
  | MARTIN | 1250.00 |
  | WARD   | 1250.00 |
  | MILLER | 1300.00 |
  | TURNER | 1500.00 |
  | ALLEN  | 1600.00 |
  | CLARK  | 2450.00 |
  | BLAKE  | 2850.00 |
  | JONES  | 2975.00 |
  | FORD   | 3000.00 |
  | SCOTT  | 3000.00 |
  | KING   | 5000.00 |
  +--------+---------+
  ```



## 数据处理函数

数据处理函数又称为单行处理函数。

特点：一个输入对应一个输出，与之相对的是多行处理函数（特点：多个输入对应一个输出）

### 常见的单行处理函数

```mysql
/*lower：转换小写*/
select lower(ename) from emp;
---------------------------------------------------------------------

/*upper：转换大写*/
select upper(name) from t_student;
---------------------------------------------------------------------

/*substr：取子串*/
//起始下标从1开始，超，反人类了属于是
select substr(ename, 1, 1) as ename from emp;
+-------+
| ename |
+-------+
| S     |
| A     |
| W     |
| J     |
| M     |
| B     |
| C     |
| S     |
| K     |
| T     |
| A     |
| J     |
| F     |
| M     |
+-------+
---------------------------------------------------------------------

/*length：取长度*/
select length(ename) enamelength from emp;
---------------------------------------------------------------------

/*trim：去空格*/
select * from emp where name = trim('    King');
---------------------------------------------------------------------

/*str_to_date：将字符串转化成日期*/

/*date_format：格式化日期*/

/*format：设置千分位*/

/*round：四舍五入*/
round(1236.567, 1) //1236.6 保留1个小数
round(1236.567, 2) //1236.57 保留2个小数
round(1236.567, -1) //1240 保留到十位
---------------------------------------------------------------------

/*rand()：生成随机数*/
select rand() from emp;
+---------------------+
| rand()              |
+---------------------+
|  0.9737481083476432 |
|  0.4157529197779977 |
| 0.15752030458070365 |
|  0.5403427943031702 |
| 0.22915308850738508 |
|  0.5247370903610598 |
|  0.9362262170167884 |
| 0.10691984473440781 |
|  0.7259206033553188 |
| 0.30884351330701587 |
| 0.36645578720276784 |
|  0.9057484268264364 |
| 0.42937480325752386 |
| 0.42962876654195487 |
+---------------------+
---------------------------------------------------------------------

/*ifnull：可以将null转换成一个具体值*/
//ifnull是空处理函，专门处理空
//在所有数据库中，只要有null参与数学运算，最终结果都为null。
//ifnull用法：ifnull(数据，如果数据为null，那么被替换的值)
select ename, sal, comm from emp;
+--------+---------+---------+
| ename  | sal     | comm    |
+--------+---------+---------+
| SMITH  |  800.00 |    NULL |
| ALLEN  | 1600.00 |  300.00 |
| WARD   | 1250.00 |  500.00 |
| JONES  | 2975.00 |    NULL |
| MARTIN | 1250.00 | 1400.00 |
| BLAKE  | 2850.00 |    NULL |
| CLARK  | 2450.00 |    NULL |
| SCOTT  | 3000.00 |    NULL |
| KING   | 5000.00 |    NULL |
| TURNER | 1500.00 |    0.00 |
| ADAMS  | 1100.00 |    NULL |
| JAMES  |  950.00 |    NULL |
| FORD   | 3000.00 |    NULL |
| MILLER | 1300.00 |    NULL |
+--------+---------+---------+
select ename, sal + comm from emp;
+--------+------------+
| ename  | sal + comm |
+--------+------------+
| SMITH  |       NULL |
| ALLEN  |    1900.00 |
| WARD   |    1750.00 |
| JONES  |       NULL |
| MARTIN |    2650.00 |
| BLAKE  |       NULL |
| CLARK  |       NULL |
| SCOTT  |       NULL |
| KING   |       NULL |
| TURNER |    1500.00 |
| ADAMS  |       NULL |
| JAMES  |       NULL |
| FORD   |       NULL |
| MILLER |       NULL |
+--------+------------+
//ifnull使用
select ename, (sal + ifnull(comm, 0)) from emp;
+--------+-------------------------+
| ename  | (sal + ifnull(comm, 0)) |
+--------+-------------------------+
| SMITH  |                  800.00 |
| ALLEN  |                 1900.00 |
| WARD   |                 1750.00 |
| JONES  |                 2975.00 |
| MARTIN |                 2650.00 |
| BLAKE  |                 2850.00 |
| CLARK  |                 2450.00 |
| SCOTT  |                 3000.00 |
| KING   |                 5000.00 |
| TURNER |                 1500.00 |
| ADAMS  |                 1100.00 |
| JAMES  |                  950.00 |
| FORD   |                 3000.00 |
| MILLER |                 1300.00 |
+--------+-------------------------+
```



## 分组函数（多行处理函数）

特点：输入多行，最终输出一行

注意：分组函数在使用的时候必须先进行分组，然后才能使用，如果没有对数据分组，整张表默认为一组。

```mysql
/*count：计数*/
//计算员工数量
select count(ename) from emp;
+--------------+
| count(ename) |
+--------------+
|           14 |
+--------------+
---------------------------------------------------------------------

/*sum：求和*/
//计算员工总工资
select sum(sal) from emp;
+----------+
| sum(sal) |
+----------+
| 29025.00 |
+----------+
---------------------------------------------------------------------

/*avg：平均值*/
//计算员工平均工资
select avg(sal) from emp;
+-------------+
| avg(sal)    |
+-------------+
| 2073.214286 |
+-------------+
---------------------------------------------------------------------

/*max：最大值*/
//找出员工最多工资
select max(sal) from emp;
+----------+
| max(sal) |
+----------+
|  5000.00 |
+----------+
---------------------------------------------------------------------

/*min：最小值*/
//找出员工最低工资
select min(sal) from emp;
+----------+
| min(sal) |
+----------+
|   800.00 |
+----------+

/*分组函数在使用的时候需要注意什么*/
/*1. 分组函数自动忽略null，不需要提前对null进行处理*/
select comm from emp;
+---------+
| comm    |
+---------+
|    NULL |
|  300.00 |
|  500.00 |
|    NULL |
| 1400.00 |
|    NULL |
|    NULL |
|    NULL |
|    NULL |
|    0.00 |
|    NULL |
|    NULL |
|    NULL |
|    NULL |
+---------+
select sum(comm) from emp;
+-----------+
| sum(comm) |
+-----------+
|   2200.00 |
+-----------+
select count(comm) from emp;
+-------------+
| count(comm) |
+-------------+
|           4 |
+-------------+
---------------------------------------------------------------------

/*2. 分组函数中count(*)和count(具体字段)有什么区别*/
//count(*)：统计表当中的总行数。因为每一行记录不可能都为null
//count(具体字段)：表示统计该字段下所有不为null的元素的总数
---------------------------------------------------------------------

/*3. 分组函数不能够直接使用在where子句中*/
//找出比最低工资高的员工信息
//报错
select ename, sal from emp where sal > min(sal);
ERROR 1111 (HY000): Invalid use of group function
---------------------------------------------------------------------

/*4. 所有的分组函数可以组合起来一起用*/
select sum(sal), min(sal), max(sal), avg(sal), count(*) from emp;
+----------+----------+----------+-------------+----------+
| sum(sal) | min(sal) | max(sal) | avg(sal)    | count(*) |
+----------+----------+----------+-------------+----------+
| 29025.00 |   800.00 |  5000.00 | 2073.214286 |       14 |
+----------+----------+----------+-------------+----------+
---------------------------------------------------------------------
```



## 分组查询（重要）

- 什么是分组查询
  - 在实际应用中，可能有这样的需求，需要先进行分组，然后对每一组的数据进行操作。
  - 这个时候我们需要使用分组查询。
  - 例子：
    - 计算每个部门的工资和
    - 计算每个工作岗位的平均薪资
    - 找出每个工作岗位的最高薪资
- 写法：
  - select 字段 from 表名 **group by** ...

```mysql
/*将之前的关键字组合一起，看一下执行顺序是如何*/
select
	...
from
	...
where
	...
group by
	...
order by
	...
//以上关键字的顺序不能颠倒，需要记忆。
//执行顺序：
	1. from
	2. where
	3. group by
	4. select
	5. order by
	
/*为什么分组函数不能直接使用在where后面？*/
//因为分组函数在使用的时候必须先分了组才能用
//where在执行的时候，还没有执行group by分组，所以会报错

/*为什么select sum(sal) from emp;可以用呢*/
//因为select在group by之后才执行，就算没写group by，也默认将整个表作为一个组
---------------------------------------------------------------------

/*找出每个工作岗位的工资和*/
//思路：按照工作岗位分组，然后对工资求和
select job, sum(sal) from emp group by job;
+-----------+----------+
| job       | sum(sal) |
+-----------+----------+
| ANALYST   |  6000.00 |
| CLERK     |  4150.00 |
| MANAGER   |  8275.00 |
| PRESIDENT |  5000.00 |
| SALESMAN  |  5600.00 |
+-----------+----------+
//执行顺序
//先从emp表中查询数据
//根据job字段进行分组
//然后对每一组的数据进行sum(sal)
---------------------------------------------------------------------

/*select ename,job,sum(sal) from emp group by job; 有意义吗？*/
//没有
+-------+-----------+----------+
| ename | job       | sum(sal) |
+-------+-----------+----------+
| SCOTT | ANALYST   |  6000.00 |
| SMITH | CLERK     |  4150.00 |
| JONES | MANAGER   |  8275.00 |
| KING  | PRESIDENT |  5000.00 |
| ALLEN | SALESMAN  |  5600.00 |
+-------+-----------+----------+
//在mysql中可以执行，但是没有任何意义
/*重点结论：
//在一条select语句当中，如果有group by语句的话，
//select后面只能跟：参加分组的字段，以及分组函数
//其他一律不能跟（如这里的ename），否则就没有意义。*/
---------------------------------------------------------------------

/*找出每个部门的最高薪资*/
//思路：按照部门编号分组，求每一组的最大薪资
select deptno, max(sal) from emp group by deptno;
+--------+----------+
| deptno | max(sal) |
+--------+----------+
|     10 |  5000.00 |
|     20 |  3000.00 |
|     30 |  2850.00 |
+--------+----------+
---------------------------------------------------------------------

/*找出每个部门，不同工作岗位的最高薪资*/
//思路：先按部门分组，再按工作岗位分组，再每组找最高薪资
select deptno, job, max(sal) from emp group by deptno, job;
+--------+-----------+----------+
| deptno | job       | max(sal) |
+--------+-----------+----------+
|     10 | CLERK     |  1300.00 |
|     10 | MANAGER   |  2450.00 |
|     10 | PRESIDENT |  5000.00 |
|     20 | ANALYST   |  3000.00 |
|     20 | CLERK     |  1100.00 |
|     20 | MANAGER   |  2975.00 |
|     30 | CLERK     |   950.00 |
|     30 | MANAGER   |  2850.00 |
|     30 | SALESMAN  |  1600.00 |
+--------+-----------+----------+
---------------------------------------------------------------------

/*找出每个部门最高薪资，要求显示最高薪资大于3000的*/
//第一步：找出每个部门最高薪资，按照部门编号分组，求每一组最大值
select deptno, max(sal) from emp group by deptno;
+--------+----------+
| deptno | max(sal) |
+--------+----------+
|     10 |  5000.00 |
|     20 |  3000.00 |
|     30 |  2850.00 |
+--------+----------+
//第二步：要求显示最高薪资大于3000
select deptno, max(sal) from emp group by deptno having max(sal) > 3000;
+--------+----------+
| deptno | max(sal) |
+--------+----------+
|     10 |  5000.00 |
+--------+----------+
/*使用having可以对分完组之后的数据进一步过滤*/
/*having不能单独使用
//不能代替where
//必须和group by联合使用*/

/*但是上面的做法效率非常低，
//实际上可以先将大于3000的都用where找出来，再分组，因为分组在后where在前执行*/
select deptno, max(sal) from emp where sal > 3000 group by deptno;
+--------+----------+
| deptno | max(sal) |
+--------+----------+
|     10 |  5000.00 |
+--------+----------+

//策略：where和having，优先选择where，where实在无法完成，再选择having
/*找出每个部门平均薪资，要求显示平均薪资高于2500的*/
//这个时候就不能用where了，因为where无法使用avg分组函数来计算平均值，因为分组在where后执行
//所以这里只能使用having，这是一个特例
select deptno, avg(sal) from emp group by deptno having avg(sal) > 2500;
+--------+-------------+
| deptno | avg(sal)    |
+--------+-------------+
|     10 | 2916.666667 |
+--------+-------------+
---------------------------------------------------------------------
```



## 单表查询总结

```mysql
/*语法规范*/
select
	...
from
	...
where
	...
group by
	...
having
	...
order by
	...
	
/*执行顺序*/
1. from		//从某张表中查询数据
2. where	//经过where条件筛选有价值的数据
3. group by //对数据进行分组
4. having   //分组之后可以使用having继续筛选
5. select	//select查询出来
6. order by //最后排序输出

/*找出每个岗位的平均薪资，要求显示平均薪资大于1500的，除manager岗位之外，要求按照平均薪资降序排*/
select job, avg(sal) from emp where job != 'manager' group by job having avg(sal) > 1500 order by avg(sal) desc; 
+-----------+-------------+
| job       | avg(sal)    |
+-----------+-------------+
| PRESIDENT | 5000.000000 |
| ANALYST   | 3000.000000 |
+-----------+-------------+
---------------------------------------------------------------------
```



## distinct关键字

```mysql
/*把查询结果去除重复记录
//原表数据不会被修改，只是查询结果去重
*/
select distinct job from emp;
+-----------+
| job       |
+-----------+
| CLERK     |
| SALESMAN  |
| MANAGER   |
| ANALYST   |
| PRESIDENT |
+-----------+

//distinct只能出现在所有字段的最前方
//并且只有满足一行的所有字段都重复才会去重
select job, deptno from emp;
+-----------+--------+
| job       | deptno |
+-----------+--------+
| CLERK     |     20 |
| SALESMAN  |     30 |
| SALESMAN  |     30 |
| MANAGER   |     20 |
| SALESMAN  |     30 |
| MANAGER   |     30 |
| MANAGER   |     10 |
| ANALYST   |     20 |
| PRESIDENT |     10 |
| SALESMAN  |     30 |
| CLERK     |     20 |
| CLERK     |     30 |
| ANALYST   |     20 |
| CLERK     |     10 |
+-----------+--------+
select distinct job, deptno from emp;
+-----------+--------+
| job       | deptno |
+-----------+--------+
| CLERK     |     20 |
| SALESMAN  |     30 |
| MANAGER   |     20 |
| MANAGER   |     30 |
| MANAGER   |     10 |
| ANALYST   |     20 |
| PRESIDENT |     10 |
| CLERK     |     30 |
| CLERK     |     10 |
+-----------+--------+

//统计一下工作岗位的数量
select count(distinct job) from emp;
+---------------------+
| count(distinct job) |
+---------------------+
|                   5 |
+---------------------+
```



## 连接查询（重要）

多张表联合起来查询数据，被称为连接查询。比如说emp表和dept表联合起来查询数据，从emp表中取员工名字，从dept表中取部门名字。
根据表连接方式分类可以分为**外连接**和**内连接**和全连接（少用）
- 内连接：
  - 等值连接
  - 非等值连接
  - 自连接

- 外连接：
  - 左外连接（左连接）
  - 右外连接（右连接）

```mysql
/*当两张表进行连接查询时，没有任何条件的限制会发生什么现象？*/
//查询每个员工所在部门名称
select ename, deptno from emp;
+--------+--------+
| ename  | deptno |
+--------+--------+
| SMITH  |     20 |
| ALLEN  |     30 |
| WARD   |     30 |
| JONES  |     20 |
| MARTIN |     30 |
| BLAKE  |     30 |
| CLARK  |     10 |
| SCOTT  |     20 |
| KING   |     10 |
| TURNER |     30 |
| ADAMS  |     20 |
| JAMES  |     30 |
| FORD   |     20 |
| MILLER |     10 |
+--------+--------+
select * from dept;
+--------+------------+----------+
| DEPTNO | DNAME      | LOC      |
+--------+------------+----------+
|     10 | ACCOUNTING | NEW YORK |
|     20 | RESEARCH   | DALLAS   |
|     30 | SALES      | CHICAGO  |
|     40 | OPERATIONS | BOSTON   |
+--------+------------+----------+
select ename, dname from emp, dept;
+--------+------------+
| ename  | dname      |
+--------+------------+
| SMITH  | ACCOUNTING |
| SMITH  | RESEARCH   |
| SMITH  | SALES      |
| SMITH  | OPERATIONS |
| ALLEN  | ACCOUNTING |
| ALLEN  | RESEARCH   |
| ALLEN  | SALES      |
......
56 rows in set (0.01 sec)
//也就是14*4
//当两张表进行连接查询，没有任何条件限制的时候，最终查询结果条数是两张表条数的乘积，这种现象被称为：笛卡尔积现象。

/*查询的时候添加条件可避免笛卡尔积*/
select ename, dname from emp, dept where emp.deptno = dept.deptno;
+--------+------------+
| ename  | dname      |
+--------+------------+
| CLARK  | ACCOUNTING |
| KING   | ACCOUNTING |
| MILLER | ACCOUNTING |
| SMITH  | RESEARCH   |
| JONES  | RESEARCH   |
| SCOTT  | RESEARCH   |
| ADAMS  | RESEARCH   |
| FORD   | RESEARCH   |
| ALLEN  | SALES      |
| WARD   | SALES      |
| MARTIN | SALES      |
| BLAKE  | SALES      |
| TURNER | SALES      |
| JAMES  | SALES      |
+--------+------------+
//匹配次数没有减少，还是会搜索全表，效率比较低
//所以还可以给查询的数据也加上限定：
select emp.ename, dept.dname from emp, dept where emp.deptno = dept.deptno;
//结果也是一样的，但是效率高一些
//为了使用方便，可以给表起别名，书写效率高些
select e.ename, d.dname from emp e, dept d where e.deptno = d.deptno;
```



### 内连接

内连接特点：连接条件是能够完全匹配的，将匹配这个条件的数据查询出来。A和B连接，AB两张表没有主次关系。

```mysql
/*等值连接*/
/*连接条件是一个等量关系*/
//查询每个员工所在部门名称，显示员工名和部门名
//SQL92语法
//结构不清晰，表的连接条件，和后期进一步筛选的条件都放到了where后面
//已经被淘汰
select 
	emp.ename, dept.dname 
from
	emp, dept 
where
	emp.deptno = dept.deptno;

//SQL99语法
//表连接的条件是独立的，连接之后，如果还需要进一步筛选，可以继续添加where
select 
	emp.ename, dept.dname 
from 
	emp 
join 
	dept 
on 
	emp.deptno = dept.deptno;
	
/*SQL99语法
select
	字段
from
	表a
(inner) join
	表b
on
	a和b的连接条件
where
	筛选条件
*/
--------------------------------------------------------------------

/*非等值连接*/
/*连接条件不是一个等量关系*/
//找出每个员工的薪资等级，要求显示员工名、薪资、薪资等级
select * from emp;
+-------+--------+-----------+------+------------+---------+---------+--------+
| EMPNO | ENAME  | JOB       | MGR  | HIREDATE   | SAL     | COMM    | DEPTNO |
+-------+--------+-----------+------+------------+---------+---------+--------+
|  7369 | SMITH  | CLERK     | 7902 | 1980-12-17 |  800.00 |    NULL |     20 |
|  7499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 | 1600.00 |  300.00 |     30 |
|  7521 | WARD   | SALESMAN  | 7698 | 1981-02-22 | 1250.00 |  500.00 |     30 |
|  7566 | JONES  | MANAGER   | 7839 | 1981-04-02 | 2975.00 |    NULL |     20 |
|  7654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 | 1250.00 | 1400.00 |     30 |
|  7698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 | 2850.00 |    NULL |     30 |
|  7782 | CLARK  | MANAGER   | 7839 | 1981-06-09 | 2450.00 |    NULL |     10 |
|  7788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 | 3000.00 |    NULL |     20 |
|  7839 | KING   | PRESIDENT | NULL | 1981-11-17 | 5000.00 |    NULL |     10 |
|  7844 | TURNER | SALESMAN  | 7698 | 1981-09-08 | 1500.00 |    0.00 |     30 |
|  7876 | ADAMS  | CLERK     | 7788 | 1987-05-23 | 1100.00 |    NULL |     20 |
|  7900 | JAMES  | CLERK     | 7698 | 1981-12-03 |  950.00 |    NULL |     30 |
|  7902 | FORD   | ANALYST   | 7566 | 1981-12-03 | 3000.00 |    NULL |     20 |
|  7934 | MILLER | CLERK     | 7782 | 1982-01-23 | 1300.00 |    NULL |     10 |
+-------+--------+-----------+------+------------+---------+---------+--------+
select * from salgrade;
+-------+-------+-------+
| GRADE | LOSAL | HISAL |
+-------+-------+-------+
|     1 |   700 |  1200 |
|     2 |  1201 |  1400 |
|     3 |  1401 |  2000 |
|     4 |  2001 |  3000 |
|     5 |  3001 |  9999 |
+-------+-------+-------+
select
	e.ename, e.sal, s.grade
from
	emp e
inner(可以省略只写join) join
	salgrade s
on
	e.sal between s.losal and s.hisal;
+--------+---------+-------+
| ename  | sal     | grade |
+--------+---------+-------+
| SMITH  |  800.00 |     1 |
| ALLEN  | 1600.00 |     3 |
| WARD   | 1250.00 |     2 |
| JONES  | 2975.00 |     4 |
| MARTIN | 1250.00 |     2 |
| BLAKE  | 2850.00 |     4 |
| CLARK  | 2450.00 |     4 |
| SCOTT  | 3000.00 |     4 |
| KING   | 5000.00 |     5 |
| TURNER | 1500.00 |     3 |
| ADAMS  | 1100.00 |     1 |
| JAMES  |  950.00 |     1 |
| FORD   | 3000.00 |     4 |
| MILLER | 1300.00 |     2 |
+--------+---------+-------+
-----------------------------------------------------------------

/*自连接*/
//查询员工的上级领导，要求显示员工名和对应的领导名
select empno, ename, mgr from emp;
+-------+--------+------+
| empno | ename  | mgr  |
+-------+--------+------+
|  7369 | SMITH  | 7902 |
|  7499 | ALLEN  | 7698 |
|  7521 | WARD   | 7698 |
|  7566 | JONES  | 7839 |
|  7654 | MARTIN | 7698 |
|  7698 | BLAKE  | 7839 |
|  7782 | CLARK  | 7839 |
|  7788 | SCOTT  | 7566 |
|  7839 | KING   | NULL |
|  7844 | TURNER | 7698 |
|  7876 | ADAMS  | 7788 |
|  7900 | JAMES  | 7698 |
|  7902 | FORD   | 7566 |
|  7934 | MILLER | 7782 |
+-------+--------+------+
//一张表看成两张表来用，一张当成员工表（a），一张当成领导表（b）
select a.ename, b.ename from emp a join emp b on a.mgr = b.empno; 
+--------+-------+
| ename  | ename |
+--------+-------+
| SMITH  | FORD  |
| ALLEN  | BLAKE |
| WARD   | BLAKE |
| JONES  | KING  |
| MARTIN | BLAKE |
| BLAKE  | KING  |
| CLARK  | KING  |
| SCOTT  | JONES |
| TURNER | BLAKE |
| ADAMS  | SCOTT |
| JAMES  | BLAKE |
| FORD   | JONES |
| MILLER | CLARK |
+--------+-------+
```



### 外连接

```mysql
//先看看两个表
select * from emp;
+-------+--------+-----------+------+------------+---------+---------+--------+
| EMPNO | ENAME  | JOB       | MGR  | HIREDATE   | SAL     | COMM    | DEPTNO |
+-------+--------+-----------+------+------------+---------+---------+--------+
|  7369 | SMITH  | CLERK     | 7902 | 1980-12-17 |  800.00 |    NULL |     20 |
|  7499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 | 1600.00 |  300.00 |     30 |
|  7521 | WARD   | SALESMAN  | 7698 | 1981-02-22 | 1250.00 |  500.00 |     30 |
|  7566 | JONES  | MANAGER   | 7839 | 1981-04-02 | 2975.00 |    NULL |     20 |
|  7654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 | 1250.00 | 1400.00 |     30 |
|  7698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 | 2850.00 |    NULL |     30 |
|  7782 | CLARK  | MANAGER   | 7839 | 1981-06-09 | 2450.00 |    NULL |     10 |
|  7788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 | 3000.00 |    NULL |     20 |
|  7839 | KING   | PRESIDENT | NULL | 1981-11-17 | 5000.00 |    NULL |     10 |
|  7844 | TURNER | SALESMAN  | 7698 | 1981-09-08 | 1500.00 |    0.00 |     30 |
|  7876 | ADAMS  | CLERK     | 7788 | 1987-05-23 | 1100.00 |    NULL |     20 |
|  7900 | JAMES  | CLERK     | 7698 | 1981-12-03 |  950.00 |    NULL |     30 |
|  7902 | FORD   | ANALYST   | 7566 | 1981-12-03 | 3000.00 |    NULL |     20 |
|  7934 | MILLER | CLERK     | 7782 | 1982-01-23 | 1300.00 |    NULL |     10 |
+-------+--------+-----------+------+------------+---------+---------+--------+
select * from dept;
+--------+------------+----------+
| DEPTNO | DNAME      | LOC      |
+--------+------------+----------+
|     10 | ACCOUNTING | NEW YORK |
|     20 | RESEARCH   | DALLAS   |
|     30 | SALES      | CHICAGO  |
|     40 | OPERATIONS | BOSTON   |
+--------+------------+----------+

/*右外连接*/
select
	e.ename, d.dname
from
	emp e right join dept d
on
	e.deptno = d.deptno;
/*right代表什么？*/
//表示将join关键字右边的表看成主表，将全部数据都查询出来（就算没有关联）
//捎带着关联条件去查询左边的表
//这时候表是存在主次关系的，join右边的是主，左边的是次。
+--------+------------+
| ename  | dname      |
+--------+------------+
| SMITH  | RESEARCH   |
| ALLEN  | SALES      |
| WARD   | SALES      |
| JONES  | RESEARCH   |
| MARTIN | SALES      |
| BLAKE  | SALES      |
| CLARK  | ACCOUNTING |
| SCOTT  | RESEARCH   |
| KING   | ACCOUNTING |
| TURNER | SALES      |
| ADAMS  | RESEARCH   |
| JAMES  | SALES      |
| FORD   | RESEARCH   |
| MILLER | ACCOUNTING |
| NULL   | OPERATIONS |
+--------+------------+

//左外连接也是同样的道理
/*左外连接*/
//如果你还是想把dept当成主表，emp当成次表，可以反过来写
select
	e.ename, d.dname
from
	dept d left join emp e
on
	e.deptno = d.deptno;
//结果还是一样的

/*也就是说，左右连接是可以相互转换的，看你个人喜好和想法*/
/*
select
	字段
from
	表a
left/right (outer) join
	表b
on
	a和b的连接条件
where
	筛选条件
*/
---------------------------------------------------------------

/*外连接的查询结果条数一定是 >= 内连接的查询结果条数*/
//查询每个员工的上级领导，要求显示**所有的员工名字**和领导名
select a.ename as '员工名', b.ename as '领导名' from emp a left join emp b on a.mgr = b.empno;
+-----------+-----------+
| 员工名     | 领导名     |
+-----------+-----------+
| SMITH     | FORD      |
| ALLEN     | BLAKE     |
| WARD      | BLAKE     |
| JONES     | KING      |
| MARTIN    | BLAKE     |
| BLAKE     | KING      |
| CLARK     | KING      |
| SCOTT     | JONES     |
| KING      | NULL      |
| TURNER    | BLAKE     |
| ADAMS     | SCOTT     |
| JAMES     | BLAKE     |
| FORD      | JONES     |
| MILLER    | CLARK     |
+-----------+-----------+

//如果你这里把领导表作为主表，查询结果就不一样了
select a.ename as '员工名', b.ename as '领导名' from emp a right join emp b on a.mgr = b.empno;
+-----------+-----------+
| 员工名     | 领导名     |
+-----------+-----------+
| SMITH     | FORD      |
| ALLEN     | BLAKE     |
| WARD      | BLAKE     |
| JONES     | KING      |
| MARTIN    | BLAKE     |
| BLAKE     | KING      |
| CLARK     | KING      |
| SCOTT     | JONES     |
| TURNER    | BLAKE     |
| ADAMS     | SCOTT     |
| JAMES     | BLAKE     |
| FORD      | JONES     |
| MILLER    | CLARK     |
| NULL      | SMITH     |
| NULL      | ALLEN     |
| NULL      | WARD      |
| NULL      | MARTIN    |
| NULL      | TURNER    |
| NULL      | ADAMS     |
| NULL      | JAMES     |
| NULL      | MILLER    |
+-----------+-----------+
```



## 多表连接

```mysql
/*多表连接语法
select
	...
from 
	a
join
	b
on
	a和b的连接条件
join
	c
on
	a和c的连接条件
right join
	d
on
	a和d的连接条件
*/
//一条SQL中内连接和外连接可以混合使用

/*找出每个员工的部门名称以及工资等级，要求显示员工名、部门名、薪资、薪资等级*/
select 
	e.ename,d.dname,e.sal,s.grade
from 
	emp e
join
	dept d
on
	e.deptno = d.deptno
join
	salgrade s
on
	e.sal between s.losal and s.hisal;
	
+--------+------------+---------+-------+
| ename  | dname      | sal     | grade |
+--------+------------+---------+-------+
| SMITH  | RESEARCH   |  800.00 |     1 |
| ALLEN  | SALES      | 1600.00 |     3 |
| WARD   | SALES      | 1250.00 |     2 |
| JONES  | RESEARCH   | 2975.00 |     4 |
| MARTIN | SALES      | 1250.00 |     2 |
| BLAKE  | SALES      | 2850.00 |     4 |
| CLARK  | ACCOUNTING | 2450.00 |     4 |
| SCOTT  | RESEARCH   | 3000.00 |     4 |
| KING   | ACCOUNTING | 5000.00 |     5 |
| TURNER | SALES      | 1500.00 |     3 |
| ADAMS  | RESEARCH   | 1100.00 |     1 |
| JAMES  | SALES      |  950.00 |     1 |
| FORD   | RESEARCH   | 3000.00 |     4 |
| MILLER | ACCOUNTING | 1300.00 |     2 |
+--------+------------+---------+-------+
-----------------------------------------------------------------

/*找出每个员工的部门名称以及工资等级还有上级领导，要求显示员工名、领导名、部门名、薪资、薪资等级*/
select 
	e.ename,d.dname,e.sal,s.grade,l.ename
from 
	emp e
join
	dept d
on
	e.deptno = d.deptno
join
	salgrade s
on
	e.sal between s.losal and s.hisal
left join
	emp l
on
	e.mgr = l.empno;
	
+--------+------------+---------+-------+-------+
| ename  | dname      | sal     | grade | ename |
+--------+------------+---------+-------+-------+
| SMITH  | RESEARCH   |  800.00 |     1 | FORD  |
| ALLEN  | SALES      | 1600.00 |     3 | BLAKE |
| WARD   | SALES      | 1250.00 |     2 | BLAKE |
| JONES  | RESEARCH   | 2975.00 |     4 | KING  |
| MARTIN | SALES      | 1250.00 |     2 | BLAKE |
| BLAKE  | SALES      | 2850.00 |     4 | KING  |
| CLARK  | ACCOUNTING | 2450.00 |     4 | KING  |
| SCOTT  | RESEARCH   | 3000.00 |     4 | JONES |
| KING   | ACCOUNTING | 5000.00 |     5 | NULL  |
| TURNER | SALES      | 1500.00 |     3 | BLAKE |
| ADAMS  | RESEARCH   | 1100.00 |     1 | SCOTT |
| JAMES  | SALES      |  950.00 |     1 | BLAKE |
| FORD   | RESEARCH   | 3000.00 |     4 | JONES |
| MILLER | ACCOUNTING | 1300.00 |     2 | CLARK |
+--------+------------+---------+-------+-------+
```



## 子查询

```mysql
/*select语句中嵌套select语句，被嵌套的select语句称为子查询。*/

/*子查询都可以出现在哪里呢？*/
select
	..(select).
from
	..(select).
where
	..(select).
	
/*where子句中的子查询*/
/*利用子查询来找到where中的筛选条件*/
//找出比最低工资高的员工姓名和工资
//第一步：查询最低工资是多少
select min(sal) from emp;
+----------+
| min(sal) |
+----------+
|   800.00 |
+----------+
//第二步：找出工资>800的员工
select ename,sal from emp where sal > 800;
//第三步：合并
select ename,sal from emp where sal > (select min(sal) from emp);
+--------+---------+
| ename  | sal     |
+--------+---------+
| ALLEN  | 1600.00 |
| WARD   | 1250.00 |
| JONES  | 2975.00 |
| MARTIN | 1250.00 |
| BLAKE  | 2850.00 |
| CLARK  | 2450.00 |
| SCOTT  | 3000.00 |
| KING   | 5000.00 |
| TURNER | 1500.00 |
| ADAMS  | 1100.00 |
| JAMES  |  950.00 |
| FORD   | 3000.00 |
| MILLER | 1300.00 |
+--------+---------+
---------------------------------------------------------------

/*form子句中的子查询*/
/*注意：from中的子查询，可以讲子查询的查询结果当作一张临时表（技巧）

//找出每个岗位的平均工资的薪资等级*/
//第一步：先找出每个岗位的平均工资
//注意：一定要给avg(sal)起别名，不然会报错
select job,avg(sal) as avgsal from emp group by job;
+-----------+-------------+
| job       | avgsal      |
+-----------+-------------+
| ANALYST   | 3000.000000 |
| CLERK     | 1037.500000 |
| MANAGER   | 2758.333333 |
| PRESIDENT | 5000.000000 |
| SALESMAN  | 1400.000000 |
+-----------+-------------+
//第二步：和salgrade表连接，条件为判断工资等级
select 
	a.*, s.grade 
from 
	(select job,avg(sal) as avgsal from emp group by job) a 
join 
	salgrade s 
on 
	a.avgsal between s.losal and s.hisal;
+-----------+-------------+-------+
| job       | avgsal      | grade |
+-----------+-------------+-------+
| ANALYST   | 3000.000000 |     4 |
| CLERK     | 1037.500000 |     1 |
| MANAGER   | 2758.333333 |     4 |
| PRESIDENT | 5000.000000 |     5 |
| SALESMAN  | 1400.000000 |     2 |
+-----------+-------------+-------+
-------------------------------------------------------------------

/*select后的子查询（了解即可）*/
/*找出每个员工的部门名称，要求显示员工名，部门名*/
select
	e.ename, (select d.dname from dept d where e.deptno = d.deptno) as dname
from
	emp e;
+--------+------------+
| ename  | dname      |
+--------+------------+
| SMITH  | RESEARCH   |
| ALLEN  | SALES      |
| WARD   | SALES      |
| JONES  | RESEARCH   |
| MARTIN | SALES      |
| BLAKE  | SALES      |
| CLARK  | ACCOUNTING |
| SCOTT  | RESEARCH   |
| KING   | ACCOUNTING |
| TURNER | SALES      |
| ADAMS  | RESEARCH   |
| JAMES  | SALES      |
| FORD   | RESEARCH   |
| MILLER | ACCOUNTING |
+--------+------------+
```





## Union合并查询结果集

```mysql
/*查询工作岗位是manager和salesman的员工*/
select ename,job from emp where job = 'manager' or job ='salesman';
select ename,job from emp where job in('manager', 'salesman');
+--------+----------+
| ename  | job      |
+--------+----------+
| ALLEN  | SALESMAN |
| WARD   | SALESMAN |
| JONES  | MANAGER  |
| MARTIN | SALESMAN |
| BLAKE  | MANAGER  |
| CLARK  | MANAGER  |
| TURNER | SALESMAN |
+--------+----------+

/*使用Union*/
select ename,job from emp where job = 'manager'
union
select ename,job from emp where job = 'salesman';
+--------+----------+
| ename  | job      |
+--------+----------+
| JONES  | MANAGER  |
| BLAKE  | MANAGER  |
| CLARK  | MANAGER  |
| ALLEN  | SALESMAN |
| WARD   | SALESMAN |
| MARTIN | SALESMAN |
| TURNER | SALESMAN |
+--------+----------+

/*这个案例来说，结果是一样的
//union的效率要高一些，对于表连接来说，每连接一次新表都会做一次笛卡尔积再找满足条件的结果
//而union可以减少匹配的次数，在减少匹配次数的情况下完成两个结果集的拼凑
//
//a 连接 b 连接 c
//a 10条记录
//b 10条记录
//c 10条记录
//匹配次数是 10*10*10 = 1000
//
//a 连接 b 一个结果：10*10 -> 100次
//a 连接 c 一个结果：10*10 -> 100次
//使用union那么就是 100+100 = 200（把乘法变成了加法）
*/


/*union使用的注意事项*/
//错误的：union在进行结果集合并的时候，要求两个结果集的列数相同
select ename,job from emp where job = 'manager'
union
select ename from emp where job = 'salesman';
ERROR 1222 (21000): The used SELECT statements have a different number of columns

//MySQL中可以，但是oracle中语法严格，不可以
//要求：结果合并时列和列的数据类型也要一致
select ename,job from emp where job = 'manager'
union
select ename,sal from emp where job = 'salesman';
+--------+---------+
| ename  | job     |
+--------+---------+
| JONES  | MANAGER |
| BLAKE  | MANAGER |
| CLARK  | MANAGER |
| ALLEN  | 1600    |
| WARD   | 1250    |
| MARTIN | 1250    |
| TURNER | 1500    |
+--------+---------+
 
```





## limit取出查询结果集部分（重要）、分页

```mysql
/*limit是将查询结果集的一部分取出来，通常使用在分页查询当中
//百度默认：一页显示10条记录
//分页的作用是为了提高用户体验，一次显示全部鬼跟你看*/

/*limit的使用*/
//按照薪资降序，取出排名在前5名的员工

//完整用法：limit startIndex, length  即起始下标和长度
//起始下标从0开始
select ename,sal from emp order by sal desc limit 0,5;

//缺省用法：limit length  即取前length个
select ename,sal from emp order by sal desc limit 5;
+-------+---------+
| ename | sal     |
+-------+---------+
| KING  | 5000.00 |
| FORD  | 3000.00 |
| SCOTT | 3000.00 |
| JONES | 2975.00 |
| BLAKE | 2850.00 |
+-------+---------+
--------------------------------------------------------------------------

/*mysql中limit在order by之后执行*/
//取出工资排名在[3-5]名的员工
select ename,sal from emp order by sal desc limit 2,3;
+-------+---------+
| ename | sal     |
+-------+---------+
| SCOTT | 3000.00 |
| JONES | 2975.00 |
| BLAKE | 2850.00 |
+-------+---------+

//取出工资排名在[5-9]名的员工
select ename,sal from emp order by sal desc limit 4,5;
+--------+---------+
| ename  | sal     |
+--------+---------+
| BLAKE  | 2850.00 |
| CLARK  | 2450.00 |
| ALLEN  | 1600.00 |
| TURNER | 1500.00 |
| MILLER | 1300.00 |
+--------+---------+
--------------------------------------------------------------------------

/*分页*/
这个好像只是说了一下怎么在java里面写sql语句怎么分页：
String sql = "select ... limit " + startIndex + "," + pageSize;
limit (pageNo-1)*pageSize, pageSize
```



## DQL总结

```mysql
/*整个语句的结构*/
select
	...
where
	...
group by
	...
having
	...
order by
	...
limit
	...

/*执行顺序*/
1. from
2. where
3. group by
4. having
5. select
6. order by
7. limit

/*难点：多表连接查询（内连接外连接）*/
```



# -------- DDL --------

## 建表的语法格式

```mysql
create table 表名(
    字段名1 数据类型, 
    字段名2 数据类型, 
    字段名3 数据类型
);
/*表名：建议以t_或者tb_开始，可读性强。*/
```

## mysql中的数据类型

```mysql
/*常见的数据类型*/
/*给定的数字不是指多少个字节，是指多少个字符，也就是有多长，中文一个字长度也是1*/
varchar（最长255）
	—— 可变长度的字符串，比较智能，节省空间，会根据实际的数据长度分配空间。
char（最长255）
	—— 定长字符串，分配固定长度空间存储数据。效率比varchar高，但是使用不当会导致浪费
/*当数据定长或者大部分情况定长时，选择char，如性别；当数据不定长时，选varchar，如姓名*/

int（最长11）
	—— 数字中的整数型，等同于int
bigint
	—— 数字中的长整形，等同于long
float
	—— 单精度浮点型
double
	—— 双精度浮点型
date
	—— 短日期类型
datetime
	—— 长日期类型
clob（Character Large Object）
	—— 字符大对象，最多可以存储4G的字符串，比如存储一篇文章。超255都用这个
blob（Binary Large Object）
	—— 二进制大对象，专门用来存储图片、声音、视频等流媒体数据。往blob类型字段插入数据，需要使用IO流
```



## 建表和删除表

```mysql
/*建一个表*/
create table t_student(
	no int,
    name varchar(32),
    sex char(1),
    age int(3),
    email varchar(255)
);

/*删除一张表*/
drop table t_student; // 当表不存在的时候会报错！

drop table if exists t_student; // 如果这张表存在的话，删除

/*在建表的时候可以用default为某个字段添加默认值*/
create table t_student(
	no int,
    name varchar(32),
    sex char(1) default '男',
    age int(3) default 18
);
```



# -------- DML --------



## 插入数据 insert

```mysql
/*语法格式*/
insert into 表名(字段名1, 字段名2, 字段名3) values(值1, 值2, 值3);

insert into t_student(no, name, sex, age, email) values(1, 'vansama', '男', 20, 'van@qq.com');

// 字段和值对应就可以，打乱了也无所谓
insert into t_student(name, sex, age, email, no) values('bily', '男', 20, 'bily@qq.com', 2);

// 如果值只给了部分的字段，那么其他未赋值的字段就是NULL
insert into t_student(no, name) values(3, 'Mr.banana');
+------+-----------+------+------+-------------+
| no   | name      | sex  | age  | email       |
+------+-----------+------+------+-------------+
|    1 | vansama   | 男   |   20 | van@qq.com  |
|    2 | bily      | 男   |   20 | bily@qq.com |
|    3 | Mr.banana | NULL | NULL | NULL        |
+------+-----------+------+------+-------------+

/*insert语句中的"字段名"可以省略，但是值必须写完整*/
insert into t_student values(4, 'mugi', '男', 18, 'mugi@qq.com');
```



## insert 插入日期

```mysql
drop table if exists t_user;
// 第一种写法
create table t_user(
	id int,
    name varchar(32),
    birth date //生日可以使用date日期类型
);

// 第二种写法
create table t_user(
	id int,
    name varchar(32),
    birth char(10) //生日可以使用字符串类型
)

// 注意数据库中的命名规范：所有的标识符都是全部小写，用下划线衔接。

/*我们先使用上面的一种写法建表*/
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | YES  |     | NULL    |       |
| name  | varchar(32) | YES  |     | NULL    |       |
| birth | date        | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+

// 尝试插入数据
insert into t_user(id, name, birth) values(1, 'van', '04-05-2002');
// 类型不匹配，数据库中需要的是date类型，这里给的是varchar类型，虽然我这个版本的mysql如果写的是'2002-04-05'直接就可以了，那可能是因为帮你自动调用了str_to_date。
```



### mysql的日期格式

```mysql
%Y 年 // 注意Y一定要大写
%m 月
%d 日
%h 时
%i 分 // 因为minute的m被月的month占用了，所以用第二个字母i
%s 秒
```



### str_to_date

```mysql
/*str_to_date：将字符串varchar类型转换为date类型，通常使用在insert插入操作*/
// str_to_date('字符串日期', '日期格式')

insert into t_user(id, name, birth) values(1, 'van', str_to_date('04-05-2002', '%d-%m-%Y'));

/*如果提供的日期字符串为 %Y-%m-%d 这个格式，那么就不需要使用str_to_date函数也可以*/
insert into t_user(id, name, birth) values(1, 'van', '2002-04-05');
```



### date_format

```mysql
/*date_format：将date类型转换成具有一定格式的varchar类型*/

/*使用场景：查询select的时候使用某个特定的日期格式来展示*/
select id, name, date_format(birth, '%m/%d/%Y') as birth from t_user;
+------+------+-------------+
| id   | name | birth       |
+------+------+-------------+
|    1 | van  | 04/05/2002  |
|    1 | van  | 05/04/2002  |
+------+------+-------------+

/*其实mysql默认就会将日期格式化为："%Y-%m-%d"，自动将数据库中的date类型转换为varchar类型*/
select id, name, birth from t_user;
+------+------+------------+
| id   | name | birth      |
+------+------+------------+
|    1 | van  | 2002-04-05 |
|    1 | van  | 2002-05-04 |
+------+------+------------+
```



### date和datetime两个类型的区别

```mysql
/*date是短日期，只包括年月日信息*/
/*datetime是长日期，包括年月日时分秒信息*/

drop table if exists t_user;
create table t_user(
	id int,
    name varchar(32),
    birth date,
    create_time datetime
);

/*mysql短日期默认格式：%Y-%m-%d */
/*mysql长日期默认格式：%Y-%m-%d %h:%i:%s */

insert into t_user(id, name, birth, create_time) values(1, 'van', '2022-6-9', '2022-6-9 21:03:41');

select * from t_user;
+------+------+------------+---------------------+
| id   | name | birth      | create_time         |
+------+------+------------+---------------------+
|    1 | van  | 2022-06-09 | 2022-06-09 21:03:41 |
+------+------+------------+---------------------+

/*在mysql中怎么获取系统当前时间？ now()函数，带有时分秒信息*/
insert into t_user(id, name, birth, create_time) values(1, 'van', '2022-6-9', now());

select * from t_user;
+------+------+------------+---------------------+
| id   | name | birth      | create_time         |
+------+------+------------+---------------------+
|    1 | van  | 2022-06-09 | 2022-06-09 21:03:41 |
|    1 | van  | 2022-06-09 | 2022-06-09 21:10:54 |
+------+------+------------+---------------------+


```



## 修改数据 update

```mysql
/*update的语法格式*/
update 表名 set 字段名1=值1, 字段名2=值2, 字段名3=值3... where 条件;

// 注意：没有条件限制会导致整个表所有数据都修改！当然如果你的需求是全部表的该字段都改，那可以。

update t_user set name='bily', birth='1990-10-2' where id=2;

select * from t_user;
+------+------+------------+---------------------+
| id   | name | birth      | create_time         |
+------+------+------------+---------------------+
|    1 | van  | 2022-06-09 | 2022-06-09 21:03:41 |
|    2 | bily | 1990-10-02 | 2022-06-09 21:10:54 |
+------+------+------------+---------------------+
```



## 删除数据 delete

```mysql
/*delete的语法格式*/
delete from 表名 where 条件;

// 注意：没有条件，整张表的数据会全部删除！

delete from t_user where id = 2;

select * from t_user;
+------+------+------------+---------------------+
| id   | name | birth      | create_time         |
+------+------+------------+---------------------+
|    1 | van  | 2022-06-09 | 2022-06-09 21:03:41 |
+------+------+------------+---------------------+
```



## insert 语句插入多条记录

```mysql
/*在values 后加入多个括号即可插入多条记录，记录间用逗号隔开*/
insert into t_user(id, name, birth, create_time) values
(2, 'bily', '2000-1-1', now()),
(3, 'mugi', '2020-6-5', now()),
(4, 'Mr.banana', '1999-10-3', now());

+------+-----------+------------+---------------------+
| id   | name      | birth      | create_time         |
+------+-----------+------------+---------------------+
|    1 | van       | 2022-06-09 | 2022-06-09 21:03:41 |
|    2 | bily      | 2000-01-01 | 2022-06-09 22:27:45 |
|    3 | mugi      | 2020-06-05 | 2022-06-09 22:27:45 |
|    4 | Mr.banana | 1999-10-03 | 2022-06-09 22:27:45 |
+------+-----------+------------+---------------------+
```



## 快速复制表

```mysql
/*将查询的结果作为一张表创建出来*/
create table emp2 as select * from emp;
// 省略掉as关键字也是可以的

select * from emp2;
+-------+--------+-----------+------+------------+---------+---------+--------+
| EMPNO | ENAME  | JOB       | MGR  | HIREDATE   | SAL     | COMM    | DEPTNO |
+-------+--------+-----------+------+------------+---------+---------+--------+
|  7369 | SMITH  | CLERK     | 7902 | 1980-12-17 |  800.00 |    NULL |     20 |
|  7499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 | 1600.00 |  300.00 |     30 |
|  7521 | WARD   | SALESMAN  | 7698 | 1981-02-22 | 1250.00 |  500.00 |     30 |
|  7566 | JONES  | MANAGER   | 7839 | 1981-04-02 | 2975.00 |    NULL |     20 |
|  7654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 | 1250.00 | 1400.00 |     30 |
|  7698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 | 2850.00 |    NULL |     30 |
|  7782 | CLARK  | MANAGER   | 7839 | 1981-06-09 | 2450.00 |    NULL |     10 |
|  7788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 | 3000.00 |    NULL |     20 |
|  7839 | KING   | PRESIDENT | NULL | 1981-11-17 | 5000.00 |    NULL |     10 |
|  7844 | TURNER | SALESMAN  | 7698 | 1981-09-08 | 1500.00 |    0.00 |     30 |
|  7876 | ADAMS  | CLERK     | 7788 | 1987-05-23 | 1100.00 |    NULL |     20 |
|  7900 | JAMES  | CLERK     | 7698 | 1981-12-03 |  950.00 |    NULL |     30 |
|  7902 | FORD   | ANALYST   | 7566 | 1981-12-03 | 3000.00 |    NULL |     20 |
|  7934 | MILLER | CLERK     | 7782 | 1982-01-23 | 1300.00 |    NULL |     10 |
+-------+--------+-----------+------+------------+---------+---------+--------+
```



## 将查询结果 insert 插入一张表中（少用）

```mysql
/*和快速复制表差不多，就是改成了insert*/
insert into t_user select * from t_user;

select * from t_user;
+------+-----------+------------+---------------------+
| id   | name      | birth      | create_time         |
+------+-----------+------------+---------------------+
|    1 | van       | 2022-06-09 | 2022-06-09 21:03:41 |
|    2 | bily      | 2000-01-01 | 2022-06-09 22:27:45 |
|    3 | mugi      | 2020-06-05 | 2022-06-09 22:27:45 |
|    4 | Mr.banana | 1999-10-03 | 2022-06-09 22:27:45 |
|    1 | van       | 2022-06-09 | 2022-06-09 21:03:41 |
|    2 | bily      | 2000-01-01 | 2022-06-09 22:27:45 |
|    3 | mugi      | 2020-06-05 | 2022-06-09 22:27:45 |
|    4 | Mr.banana | 1999-10-03 | 2022-06-09 22:27:45 |
+------+-----------+------------+---------------------+
```



## 快速删除表中的数据

```mysql
/*一般的删除表中删除方式*/
delete from 表名;

// 这种删除数据的方式比较慢。
// 而且，表中的数据被删除了，但是数据在硬盘上的真实存储空间不会被释放。所以有后悔的机会，支持回滚rollback，可以撤回。


/*truncate语句删除数据 truncate:截断 */
truncate table 表名; // 属于DDL操作

// 这种删除效率比较高，表被一次截断，物理删除。
// 不支持回滚rollback

/*效率差距，假设有一亿条数据，delete要一个小时，truncate只需要一秒钟*/

/*区分：删除表操作是drop*/
drop table 表名; // 这不是删除表中的数据，是删除表，上面的两个是删除表中的数据，表、表中的字段还在。
```



## 对表结构的增删改（了解即可）

```mysql
// 对表结构的修改：就是添加一个字段、删除一个字段、修改一个字段

// 对表结构的修改需要使用：alter 属于DDL语句

// 在实际开发中，需求一旦确定，表一旦涉及好之后，很少对表结构修改，因为开发进行时，修改表结构成本很大。另外，修改表结构可以使用
```



# --------- 约束 ---------



## 什么是约束 constraint

在创建表的时候，我们可以给表中的字段加上一些约束，来保证这个表中数据的完整性、有效性。

约束的作用是为了保证表中数据**有效**。

## 约束包括哪些

- 非空约束：not null
- 唯一性约束：unique
- 主键约束：primary key 简称 PK
- 外键约束：foreign key 简称 FK
- 检查约束：check（mysql不支持，oracle支持）



## 非空约束 not null

```mysql
/*和default一样，只需要加在字段定义的后面就行*/
drop table if exists t_vip;
create table t_vip(
	id int,
    name varchar(255) not null
);
insert into t_vip values(1, 'van');

// 设定为 not null 之后，不插入值直接报错
insert into t_vip(id) values(2);
ERROR 1364 (HY000): Field 'name' doesn't have a default value


```



## 唯一性约束 unique

```mysql
/*所规定的字段值在一个表中不能重复*/
drop table if exists t_vip;
create table t_vip(
	id int unique,
    name varchar(255)
);

insert into t_vip values(1, 'van');
insert into t_vip values(2, 'bily');

// 设定为unique后，所插入的值必须为唯一的，不能和之前的重复。
insert into t_vip values(1, 'mr.banana');
ERROR 1062 (23000): Duplicate entry '1' for key 'id'

/*字段中的NULL不算重，NULL的意思是这里什么都没有，是空的*/
```



## 两个字段联合唯一

```mysql
/*如果name和email两个字段联合起来具有唯一性该怎么做？*/
/*还是使用unique：unique(字段1, 字段2)*/
drop table if exists t_vip;
create table t_vip(
	id int,
    name varchar(255),
    email varchar(255),
    unique(name, email)
);

// 两个字段联合起来不能重复，就能插进去 
insert into t_vip(id, name, email) values(1, 'van', 'van@qq.com');
insert into t_vip(id, name, email) values(2, 'van', 'van@163.com');

select * from t_vip;
+------+------+-------------+
| id   | name | email       |
+------+------+-------------+
|    1 | van  | van@qq.com  |
|    2 | van  | van@163.com |
+------+------+-------------+

// 两个字段联合起来是重复的，就插不进去
insert into t_vip(id, name, email) values(3, 'van', 'van@qq.com');
ERROR 1062 (23000): Duplicate entry 'van-van@qq.com' for key 'name'

/*约束如果在建表时直接添加在字段后面，是列级约束*/
/*约束如果在建表时直接另起一行写，是表级约束，如果是多个字段联合约束，就需要使用表级约束了*/
```



## unique 和 not null 联合使用

 ```mysql
 /*答案是可以的*/
 drop table if exists t_vip;
 create table t_vip(
 	id int,
     name varchar(255) not null unique
 );
 
 desc t_vip;
 +-------+--------------+------+-----+---------+-------+
 | Field | Type         | Null | Key | Default | Extra |
 +-------+--------------+------+-----+---------+-------+
 | id    | int(11)      | YES  |     | NULL    |       |
 | name  | varchar(255) | NO   | PRI | NULL    |       |
 +-------+--------------+------+-----+---------+-------+
 /*在mysql当中，如果一个字段同时被 not null 和 unique 约束的话，该字段自动变成主键字段（oracle中不一样）*/
 ```



## 主键约束（重要）

```mysql
/*主键的特征：not null + unique*/
drop table if exists t_vip;
create table t_vip(
	id int primary key,
    name varchar(255)
);

insert into t_vip values(1, 'van');
insert into t_vip values(2, 'bily');

insert into t_vip values(2, 'mr.banana');
ERROR 1062 (23000): Duplicate entry '2' for key 'PRIMARY'

/*当然也可以使用表级约束来添加主键*/
drop table if exists t_vip;
create table t_vip(
	id int,
    name varchar(255),
    primary key(id)
);

insert into t_vip values(1, 'van');
insert into t_vip values(2, 'bily');

insert into t_vip values(2, 'mr.banana');
ERROR 1062 (23000): Duplicate entry '2' for key 'PRIMARY'

/*复合主键：两个字段联合起来做主键，和上面所说的表级约束用法相同*/
/*在实际开发中不建议使用复合主键*/
/*一张表主键约束只能添加一个*/

/*主键值建议使用类型*/
int
bigint
char

// 不建议使用varchar来做主键，主键一般都是数字，而且是定长的。

/*主键还可以分为自然主键和业务主键*/
// 自然主键：主键值是一个自然数，和业务无关。
// 业务主键：主键值和业务紧密关联，比如拿银行卡账号做主键值

// 在实际开发中，使用自然主键多，因为主键只要做到不重复就行，不需要有意义。业务主键不好的地方在于和业务挂钩，一旦需要修改业务，可能会影响到主键值。

/*在mysql中，有一种机制，可以帮我们自动维护一个主键值：auto_increment*/
/*auto_increment：表示自增，从1开始以1自增*/
drop table if exists t_vip;
create table t_vip(
	id int primary key auto_increment,
    name varchar(255)
);
insert into t_vip(name) values('van');
insert into t_vip(name) values('van');
insert into t_vip(name) values('van');
insert into t_vip(name) values('van');

select * from t_vip;
+----+------+
| id | name |
+----+------+
|  1 | van  |
|  2 | van  |
|  3 | van  |
|  4 | van  |
+----+------+
```



## 外键约束（重要）

```mysql
/*业务背景：请设计数据库表，来描述班级和学生的信息*/

// 肯定是用两张表，一张班级，一张学生表

t_class 班级表
classno(pk)      classname
--------------------------------------
100              1班
101              2班

t_student 学生表
no(pk)     name     cno(FK引用t_class这张表的classno)
--------------------------------------
1          van      100
2          bily     101

// 当cno字段没有任何约束的时候，可能会导致数据无效，出现一个cno为102。为了保证cno字段中的值都是100和101，需要给cno字段添加外键约束。
// cno字段就是外键字段。cno字段中的每一个值都是外键值。

/*foreign key(外键字段) references 表名(表的字段，最好是主键)*/
drop table if exists t_student;
drop table if exists t_class;

create table t_class(
	classno int primary key,
    classname varchar(255)
);
create table t_student(
	no int primary key auto_increment,
    name varchar(255),
    cno int,
    foreign key(cno) references t_class(classno)
);

insert into t_class values(101, '1班');
insert into t_class values(102, '2班');

insert into t_student(name, cno) values('van', 101);
insert into t_student(name, cno) values('bily', 102);

// 这时cno如果输入了非外键字段中存在的外键值，那么就会报错
insert into t_student(name, cno) values('mugi', 103);
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`van`.`t_student`, CONSTRAINT `t_student_ibfk_1` FOREIGN KEY (`cno`) REFERENCES `t_class` (`classno`))

/*注意：t_class 是父表、t_student 是子表*/
// 各种顺序
// 删除表的顺序：先删子，后删父
// 创建表的顺序：先创建子，再创建父
// 删除数据的顺序：先删除子，再删除父
// 插入数据的顺序：先插入父，再插入子

/*子表中的外键引用父表中的某个字段，被引用的这个字段必须是主键吗？*/
// 不一定是主键，但至少要有unique约束

/*外键可以为NULL吗？*/
// k
```

