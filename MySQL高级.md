# MySQL高级

## 1.MySQL的架构介绍

### 1.1、MySQL简介

MySQL内核

MySQL工程师

MySQL服务器的优化

各种参数常量设定

查询语句优化

主从复制

软硬件升级

容灾备份

SQL编程

###1.2、MySQLLinux版的安装

检测Linux下有没有按照MySQL

```xml
rpm -qa|grep -i mysql
```



### 1.3、MySQL配置文件

```xaml
my-default.cnf
show variables like 'character%'
show variables like '%char%'

```

可以在配置文件中手动设置客户端，服务器编码

二进制日志log-bin   主从复制

错误日志log-error：默认是关闭的，记录严重的警告和错误信息，每次启动和关闭的详细信息等

查询日志log：默认关闭，记录查询的SQL语句，如果开启会降低MySQL的整体性能

数据文件：	frm 存放表结构

​			myd:数据

​			myi：索引

如何配置

### 1.4、MySQL逻辑架构介绍

![1525930579594](C:\Users\ADMINI~1\AppData\Local\Temp\1525930579594.png)

连接层

服务层，集群，复制，存储过程，解析，查询优化，缓存

引擎层

存储层

### 1.5、MySQL存储引擎

![1525931435134](C:\Users\ADMINI~1\AppData\Local\Temp\1525931435134.png)





## 2.索引优化分析

### 2.1、性能下降SQL慢、执行时间长、等待时间长

查询语句写的烂

索引失效：单值；复合

关联查询太多join（设计缺陷，极端需求）

服务器调优及各个参数设置

### 2.2、常见通用的join查询

####SQL执行顺序

手写

机读

![1525932630405](C:\Users\ADMINI~1\AppData\Local\Temp\1525932630405.png)

![1525932704284](C:\Users\ADMINI~1\AppData\Local\Temp\1525932704284.png)

#### 七种join

![timg](C:\Users\Administrator\Desktop\timg.jpg)

join等于inner join 

join是inner join 的简写

left join 是要左表的全部，加上共有

right join 是要右表的全部加上共有

full join 完全链接

![1525933242513](C:\Users\ADMINI~1\AppData\Local\Temp\1525933242513.png)

```sql
select <select_list> from tableA a 
left join tableB b 
on a.key=b.key
where b.key is null
```

![1525933314448](C:\Users\ADMINI~1\AppData\Local\Temp\1525933314448.png)

```sql
select <select_list> from tableA a 
inner join tableB b 
on a.key=b.key
```

```sql
select <select_list> from tableA a 
right join tableB b 
on a.key=b.key
where a.key is null
```

```sql
select <select_list> from tableA a 
full join tableB b 
on a.key=b.key
where a.key is null
```

```sql
--a 独有 + b 独有 + ab 共有
select * from tableA a left join tableB b on a.key = b.key
union --合并加去重
select * from tableA a right join tableB b on a.key=b.key

--a 独有 + b 独有
select * from tableA a left join tableB b on a.key = b.key where b.key is null
union --合并加去重
select * from tableA a right join tableB b on a.key=b.key where a.key is null
```

```sql
union --合并加去重
union all --合并不去重
```



### 2.3、索引简介

#### 是什么

索引（index）是帮助MySQL高效获取数据的数据结构，索引是数据结构

排好序的快速查找数据结构

索引会影响where后面的查找条件和order by后面的排序条件

在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构一某种方式引用数据

二叉查找树

每个节点都包含一个键值和一个指向数据物理地址的地址

一般来说，索引本身也很大,不可以全部存在内存中，因此索引以索引文件的形式存在磁盘上

频繁修改的数据不适合建索引

#### 优势

提高数据检索效率，降低数据库io成本

降低数据库排序成本，降低了cpu消耗

#### 劣势

#### 索引分类

单值索引，即一个索引只包含单个列，一个表可以有多个单列索引

唯一索引，索引列的值必须唯一，但允许有空值

复合索引，一个索引包含多个列

#### 那些情况需要建立索引

主键自动建立唯一索引

频繁作为查询条件的字段

查询中与其他表关联的字段

组合索引优于单值索引

查询中排序的字段，排序字段通过索引区访问，大大提高培训速度

查询中统计或者分组的字段（分组必排序）

#### 那些情况不要创建索引

表记录太少

频繁更新的字段

where条件里用不到的字段

数据重复，分布平均的字段，没有多大效果

### 2.4、性能分析

### 2.5、索引优化



## 3.查询截取分析

### 3.1、查询优化

### 3.2、慢查询日志

### 3.3、批量数据脚本

### 3.4、show Profile

### 3.5、全局查询日志



## 4.MySQL锁机制

### 4.1、行锁

### 4.2、表锁

### 4.3、页锁



## 5.主从复制

###5.1、