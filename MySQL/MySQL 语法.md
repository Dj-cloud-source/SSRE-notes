
每次sql语句后面加上 `;`  号


创建用户并授权
```sql
create user 'user01'@ '网段' identified by '123456';

grant select,update,delete,insert on  *.* to 'user01'@'网段' with grant option; 

```
> 一般给开发创建的就是“增删改查”



## sql语句

sql语句
	1.DDL 数据定义
	2.DCL 数据控制
	3.DML 数据操作
	4.==DQL== 数据查询





## DDL

创建库
```sql
create database db01 ;
create database DB01 ;


drop database db01 ;

```


修改定义库
```sql
alter database da01 charset utf8;
show create database db01;
```


创建表
```sql

create table student(
 sid int,
 sname varchar(20),
 sage tinyint,
 sgender enum('m','f'),
 comtime datetime  
 );
 
```
```sql
create table student1 (
 sid int not null primary key auto_increment comment '学号', 
sname varchar (20) not null default 'm' comment '性别' ,
cometime datetime not null comment '入学时间 ')charset utf8 engine innodb;
```
```sql
#查看表
mysql> desc student1;

+----------+-------------+------+-----+---------+----------------+

| Field    | Type        | Null | Key | Default | Extra          |

+----------+-------------+------+-----+---------+----------------+

| sid      | int         | NO   | PRI | NULL    | auto_increment |

| sname    | varchar(20) | NO   |     | m       |                |

| cometime | datetime    | NO   |     | NULL    |                |

+----------+-------------+------+-----+---------+----------------+
```

删除表
```sql
drop table student ;
```


修改表
```sql
alter table student rename stu;

alter table stu add age int;

alter table stu add test varchar(20),add qq int ;


alter table stu drop qq ;

#修改属性
alter table stu modify sid varchar(20);

```


## DCL
dcl是针对权限进行控制

```sql
create user 'user01'@ '网段' identified by '123456';

#赋予权限
grant select,update,delete,insert on  *.* to 'user01'@'网段' with grant option; 

#收回权限
revoke select on *.* from user01@'192.168.175.%';

show grants for user01@'192.168.175.%'

```

## DML
* 插入数据
* 操作表中数据

插入数据
```sql
insert into stu 
(sid, sname, cometime, age, test, qq)
values
('linux01', '张三', NOW(), 20, '测试', 123456);
```

更新数据
```sql

update stu set sgender = 'f' where sid ='linux01' ;

```

```sql
mysql> select * from stu ;

+---------+--------+---------------------+------+--------+--------+---------+

| sid     | sname  | cometime            | age  | test   | qq     | sgender |

+---------+--------+---------------------+------+--------+--------+---------+

| linux01 | 张三   | 2026-07-16 09:07:59 |   20 | 测试   | 123456 | f       |

+---------+--------+---------------------+------+--------+--------+---------+

1 row in set (0.00 sec)
```

删除数据
```sql
delete from stu  where sid = 'linux01';
```

> 有时候重要数据不能直接删除。使用 update 代替 delete





## DQL

>[sql语句下载](https://download.s21i.faiusr.com/23126342/0/0/ABUIABAAGAAgzcXwhQYozuPv2AE?f=world.sql&v=1622942413)
>
>`mysql -uroot -p123456 < world.sql` 
>
>`mysql>use world ;`



### 查询语句==Select== 

```sql
mysql>select * from city limit 2 ;

+----+----------+-------------+----------+------------+

| ID | Name     | CountryCode | District | Population |

+----+----------+-------------+----------+------------+

|  1 | Kabul    | AFG         | Kabol    |    1780000 |

|  2 | Qandahar | AFG         | Qandahar |     237500 |

+----+----------+-------------+----------+------------+

2 rows in set (0.00 sec)

```

```sql
select countrycode,district from city;

select id ,name , population from city where countrycode ='CHN' and name = 'suzhou';


#模糊查询
#  `%`   模糊查询的规范语法是 like ➕  %

 select id ,name , population from city where countrycode ='CHN' and name like 'Nan%';

#倒序
select  name from city order by countrycode limit 10 ;

#范围查询
select * from  city where countrycode = 'CHN' and  population >1000000 order by population ;

```


[高级查询](MySQL/DQL查询练习)

联表查询   "==表尾接表头=="
B:
 ```sql
 select s_no from score where sc_degree between 60 and 80
 ```

```sql

 select s_name from student where s_no in (B) ;
```

```sql
 select s_name from student where s_no in (select s_no from score where sc_degree between 60 and 80) ;
```


升序 `desc` 
```sql
select * from student order by s_class desc;

#多条件顺序
select * from score order by c_no,sc_degree desc ;
```

统计
```sql
select count(*) from student ;
```

子查询(最大值max)
```sql
select * from score where sc_degree in (select  max(sc_degree ) from score) ;
```

切片（取前几行）
```sql
select * from student order by s_class desc limit 3 ;
```

以组为单位求平均
```sql
 select c_no ,avg(sc_degree) from score group by c_no;
```

范围查询
```sql
select s_no ,sc_degree from score where sc_degree between 70 and 90 ;
```

联表寻找相同条件
```sql
select s_name,sc_degree ,c_no from student ,score where student.s_no=score.s_no;
```

联表查询+以组为单位求平均
```sql
select c_no , avg(sc_degree ) from score where  s_no in(select s_no from student where s_class='95031') group by c_no ;
```
“至少”条件+以组为单位
```sql
#至少两名男生
select s_class from student where  s_sex='男' group by s_class  having count(s_no)>1 ;
```



子查询（别名联表查询、嵌套查询）

> 子查询好难啊

> 可以一步一步来，理清逻辑。
> 可以先desc table  查看表结构，理清表头

```sql
select * from score where c_no in (
select c_no from course where t_no = (
select t_no  from teacher where t_name=
'张旭 ' ) );
```

```sql
 select * from score where c_no in (select c_no from course where t_no in (select t_no from teacher where t_depart like  '计算%') );
```

```sql
 select  c_name from course where t_no in (select t_no from teacher where t_sex ='男' );
```

```sql
select *  from score where sc_degree =(select max(sc_degree  ) from score  ) ;
```


多条件并集
```sql
#选修计算机且男性的成绩表
 select * from score where   s_no in (select s_no from student where s_sex ='男' )   and   c_no in (select c_no from course where c_name like '计算机%' );
```

多字段排序
(先以前一个条件排序，再以后面条件排序)
```sql
select * from student order by s_class , s_birthday;
```


### 连接查询

#### 在开始连接查询前，我们先总结一下

前面大部分查询是在
- 从什么表筛哪些行？
- 用一个查询结果当另一个查询的条件吗？
- 按什么排序？
- ……
- 

子查询
- “先查出课程号，再拿这个课程号去 score 表查成绩”
- 一个查询嵌套在另一个查询里面
- 先查一批条件，再拿条件去查

连接查询
- “多张表之间有联系，我要把它们横向拼起来”
- 把多张表按关系拼成一张结果表

虽然有写过
```sql
from student,score
where student.s_no=score.s_no
```
但这是老写法，不够标准


```sql
mysql> select * from person;

+------+--------+--------+

| id   | name   | cardId |

+------+--------+--------+

|    1 | 张三   |      1 |

|    2 | 李四   |      3 |

|    3 | 王五   |      6 |

+------+--------+--------+

3 rows in set (0.00 sec)

mysql> select * from card ;

+------+-----------+

| id   | name      |

+------+-----------+

|    1 | 饭卡      |

|    2 | 建行卡    |

|    3 | 农行卡    |

|    4 | 工商卡    |

|    5 | 邮政卡    |

+------+-----------+

5 rows in set (0.01 sec)


mysql> select * from person inner join card  on person.cardId=card.id;

+------+--------+--------+------+-----------+

| id   | name   | cardId | id   | name      |

+------+--------+--------+------+-----------+

|    1 | 张三   |      1 |    1 | 饭卡      |

|    2 | 李四   |      3 |    3 | 农行卡    |

+------+--------+--------+------+-----------+
```

> 查询过程
> 从 person 表选取所有记录
> 对于每个person 记录，查找 card 表中 id 字段与 person.cardId 匹配的记录
> 将匹配的记录组合成一条结果返回

#### 语法
```sql
#内连接
select * from person inner join card  on person.cardId=card.id;
```

```sql
#左外连接会把左边的表的数据全部取出来  ，右边表如果没有就用NULL补上
mysql>  select * from person left join card on person.cardId=card.id;

+------+--------+--------+------+-----------+

| id   | name   | cardId | id   | name      |

+------+--------+--------+------+-----------+

|    1 | 张三   |      1 |    1 | 饭卡      |

|    2 | 李四   |      3 |    3 | 农行卡    |

|    3 | 王五   |      6 | NULL | NULL      |

+------+--------+--------+------+-----------+
3 rows in set (0.00 sec)

#右连接如法炮制
mysql>  select * from person right join card on person.cardId=card.id;

+------+--------+--------+------+-----------+

| id   | name   | cardId | id   | name      |

+------+--------+--------+------+-----------+

|    1 | 张三   |      1 |    1 | 饭卡      |

| NULL | NULL   |   NULL |    2 | 建行卡    |

|    2 | 李四   |      3 |    3 | 农行卡    |

| NULL | NULL   |   NULL |    4 | 工商卡    |

| NULL | NULL   |   NULL |    5 | 邮政卡    |

+------+--------+--------+------+-----------+


```

> 感觉实际开发还挺常用的，多表联查