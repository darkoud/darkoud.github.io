---
title: mysql索引
date: '2020-09-12'
type: 技术
tags: [mysql, mysql索引]
categories: mysql
note: mysql索引类型和索引命中
---
# B-Tree B+Tree 和索引命中
## B-Tree
* 叶节点具有相同的深度,叶节点的指针为空
* 所有索引元素不重复
* 节点中的数据索引从左到右递增排列

![B-Tree](/images/mysql_index/B-Tree.png)


## B+Tree(B-Tree变种)
* 非叶子节点不存储data,只存储索引(冗余),可以放更多的索引
* 叶子节点包含所有索引字段
* 叶子节点用指针连接,提高区间访问的性能

![B+Tree](/images/mysql_index/B+Tree.png)


## MyISAM和InnoDB索引的区别
* MyISAM索引文件和数据文件是分离的(非聚集), 叶子节点储存数据的地址

![MyISAM](/images/mysql_index/MyISAM.png)

* InnoDB索引实现(聚集)
    - 表数据文件本身就是按B+Tree组织的一个索引结构文件
    - 聚集索引-叶节点包含了完整的数据记录

![InnoDB](/images/mysql_index/InnoDB.png)


## hash索引
很多时候hash索引效率高于B+tree,但是hash索引不支持范围查找

## 聚集索引/聚术索引
mysql的innodb主键索引,如果没有主键索引就是唯一索引,该索引的子节点包含了所有的data数据
非聚集索引,即**二级索引**

![clusteredIndex](/images/mysql_index/clusteredIndex.png)

* InnoDB聚合索引: 索引字段在一起存储到key,按照索引排序排列
* innodb联合索引示例(索引最左前缀原理)


## sql执行计划(explan + sql)

![explan](/images/mysql_index/explan.png)

1. **id**
>ID可以如果相同认为是同一组,从上往下执行,在所有组中id越大,优先级越高,越先执行

2. **select_type**查询类型
> 1. SIMPLE       简单查询,不包括子查询或UNION
> 2. PRIMARY      查询中包含任何复杂的子部分,最外层查询被标记为
> 3. SUBQUERY     在select或where里包含了子查询
> 4. DERIVED      在from列表中包含了子查询被标记为DERIVED(衍生),mysql会递归执行这些子查询,把结果放在临时表
> 5. UNION          若在第二个select出现在union后,会标记为UNION.若union包含在from子句的查询中,外层会标记为DERIVED
> 6. UNION RESULT  从UNION中获取select

3. **table**
> 表示explain的一行正在访问哪个表,这一行数据显示的表. type是null会直接走索引,不会走表,效率最好

4. **type**    
> 从最好到最差的顺序system > const > 	eq_ref > ref > range > index > ALL  一般来说最少达到range,最好能达到ref
> 1. const, system: mysql能对查询的某部分进行优化并将其转化成一个常量(可以看show warnings 的结果).用于primary key 或 unique key 的所有列与常数比较时, 所以表最多有一个匹配行, 读取1次, 速度比较快.system是const的特例, 表里只有一条元组匹配时为system
> 2. eq_ref: primary key 或 unique key 索引的所有部分被连接使用,最多只会返回一条符合条件的记录.这可能是在const 之外最好的联接类型了,简单的 select 查询不会出现这种 type.
> 3. ref: 相比 eq_ref,不使用唯一索引,而是使用普通索引或者唯一性索引的部分前缀,索引要和某个值相比较,可能会找到多个符合条件的行.
> 4. range:范围扫描通常出现在 in(), between ,> ,<, >= 等操作中. 使用一个索引来检索给定范围的行. 
> 5. index: 扫描全索引就能拿到结果,一般是扫描某个二级索引,这种扫描不会从索引树根节点开始快速查找,而是直接对二级索引的叶子节点遍历和扫描,速度还是比较慢的,这种查询一般为使用覆盖索引,二级索引一般比较小,所以这种通常比ALL快一些
> 6. ALL: 即全表扫描,扫描你的聚簇索引的所有叶子节点.通常情况下这需要增加索引来进行优化了. 

5. **possible_keys**    
> 显示可能应用在这张表中的索引,一个或多个,查询涉及到的字段若存在索引,则也列出来,但不一定被查询实际用到

6. **key**
> **实际使用的索引,如果查找集在二级索引里面,一般使用二级索引,因为二级索引空间小,不用回表**

7. **ken_len**
> 索引字段的最大可能长度,并非实际长度,key_len长度越短越好
> 1. ken_len 越短说明索引本身短
> 2. 越长表示走的多,但肯能查询的列更少

8. **ref**
> 显示了在key列记录的索引中,表查找值所用到的列或常量,常见的有:const(常量),字段名

9. **rows**   
> 根据表统计信息和索引选用的情况,大致估算出找到所需记录的读取行数

10. **Extra**其他的信息
> 1. Using index: 使用覆盖索引  整个查询结果只通过辅助索引就能拿到结果,不需要去回表. 
> 2. Using where: 使用 where 语句来处理结果, 查询的列未被索引覆盖
> 3. Using index condition: 查询的列不完全被索引覆盖, where条件中是一个前导列的范围
> 4. Using temporary: mysql需要创建一张临时表来处理查询. 出现这种情况一般是要进行优化的, 首先是想到用索引来优化
> 5. Using filesort: 将用外部排序而不是索引排序,数据较小时从内存排序,否则需要在磁盘完成排序. 这种情况下一般也是要考虑使用索引来优化的
> 6. Select tables optimized away: 使用某些聚合函数(比如 max、min)来访问存在索引的某个字段是

## 查看sql自动优化情况
> sql执行后面加上 show warnings; 
 
![sqlOptimize](/images/mysql_index/sqlOptimize.png)


## 索引失效的情况
> 1. (复合索引)全值匹配我最爱
> 2. 最佳左前缀法则(带头大哥不能死,中间兄弟不能断)
> 3. 不在索引列上做任何操作(计算,函数,(自动or手动)类型转换),会导致索引失效而转向全表扫描
> 4. 存储引擎不能使用索引中范围条件右边的列
> 5. 尽量使用覆盖索引(只访问索引的查询(索引列和查询列一致)),减少select*
> 6. mysql在使用不等于(! =或者<>)的时候无法使用索引会导致全表扫描
> 7. is null,is not nul 也无法使用索引
> 8. like以通配符开头(“%abc.…)mysql索引失效会变成全表扫描的操作
> 9. 字符串不加单引号索引失效,会出现函数操作,导致索引失效
> 10. 少用or,用它来连接时会索引失效

**全值匹配我最爱(聚合索引), 最左前缀要遵守;**  
**带头大哥不能死, 中间兄弟不能断;**  
**索引列上少计算, 范围之后全失效;**  
**Like百分写最右, 覆盖索引不写星;**  
**不等空值还有or, 索引失效要少用;**  
**VAR引号不可丢, SQL高级也不难!**  
解决like‘%字符串%’时索引不被使用,使用覆盖索引,即建的索引和查询的字段个数顺序最好完全一致

* sql优化小表驱动大表,非要大表驱动小表用exists
* mysql支撑Index和FileSort两种方式排序排序,Index效率高,FileSort效率低
* mysql慢查询是否开启  > show variables like 'slow_query%';  
* 开启慢查询> set global slow_query_log='ON'; 
* 设置慢查询时间> set global long_query_time=1;
* 查看设置后的参数(重新建连或新开回话查看) > show variables like 'long_query_time';