# MySQL高级

## 课程简介

| 序号 | Day01              | Day02       | Day03          | Day04         |
| ---- | ------------------ | ----------- | -------------- | ------------- |
| 1    | Linux系统安装MySQL | 体系结构    | 应用优化       | MySQL常用工具 |
| 2    | 索引               | 存储引擎    | 查询缓存优化   | MySQL日志     |
| 3    | 视图               | 优化SQL步骤 | 内存管理及优化 | MySQL主从复制 |
| 4    | 存储过程和函数     | 索引使用    | MySQL锁问题    | 综合案例      |
| 5    | 触发器             | SQL优化     | 常用SQL技巧    |               |

## 优化SQL

### 执行计划

#### select_type

| select_type  | 含义 |
| ------------ | ---- |
| SIMPLE       |      |
| PRIMARY      |      |
| SUBQUERY     |      |
| DERIVED      |      |
| UNION        |      |
| UNION RESULT |      |

从上到下，效率依次变低

#### type

| type   | 含义 |
| ------ | ---- |
| NULL   |      |
| system |      |
| const  |      |
| eq_ref |      |
| ref    |      |
| range  |      |
| index  |      |
| all    |      |

从上到下，从好到坏

一般来说，我们需要保证至少达到range级别，最好达到ref

#### key

possible_keys：显示可能应用在这张表的索引，一个或多个。

key：实际使用的索引，如果为null，则没有使用索引。

key_len：表示索引中使用的字节数，该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下，长度越短越好。

#### extra

| extra           | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| using filesort  | 说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取，称为“文件排序”，效率低。 |
| using temporary | 使用了临时表保存中间结果，mysql在堆查询结果排序时使用临时表。常见于order by和group by。效率低。 |
| using index     | 表示相应的select操作使用了覆盖索引，避免访问表的数据行，效率不错。 |

### show profile

> -- 查看是否支持profile
> select @@have_profiling;
>
> -- 默认profiling是关闭的，可以通过set语句在session级别开启profiling
> select @@profiling;
> -- 开启profiling开关
> set profiling=1;
>
> show profiles;
>
> -- 141是通过show profiles得当的query_id
> show profile for query 141;

## 索引的使用

全值匹配



最左前缀法则



索引失效的情况：

1）范围查询右边的列，不会使用索引

2）在索引列上进行运算操作，索引会失效

3）字符串类型查询不加单引号，索引会失效



注意：

尽量使用覆盖索引（只访问索引的查询（索引列完全包含查询列）），避免select *

1）using index：使用覆盖索引的时候就会出现

2）using where：在查找使用索引的情况下，需要回表去查询所需的数据

3）using index condition：查找使用了索引，但是需要回表查询数据

4）using index；using where：查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据



用or分割开的条件，如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到。



以%开头的like模糊查询，索引失效。（如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效）。

解决方案：通过覆盖索引来解决。



如果mysql评估使用索引比全表更慢，则不使用索引。



is null, is not null 有时索引失效。

这个跟数据量有关，如果表的is null多，is null就会走索引，is not null就不会走索引。

如果表的is not null多，is not null就会走索引，is null就不会走索引。



in走索引，not in索引失效。



单列索引和复合索引：

尽量使用复合索引，而少使用单列索引。

多个单列索引：数据库会选择一个最优的索引来使用，并不会使用全部索引。

## SQL优化

## Mysql高级

### 应用优化

1、使用连接池

2、减少多mysql的访问

3、负载均衡，可以通过mysql的主从复制来完成

### 缓存

>-- 查看是否支持查询缓存
>show variables like 'have_query_cache';
>
>-- 查看是否开启了查询缓存
>show variables like 'query_cache_type';
>
>-- 查看查询缓存的占用大小。单位：字节
>show variables like 'query_cache_size';

### 内存优化

### 锁

