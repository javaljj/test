https://blog.csdn.net/Coy993055697/article/details/77145584
https://blog.csdn.net/zhaoweixs/article/details/8693076
https://www.csdn.net/gather_20/MtTaMg3sMTE2Mi1ibG9n.html
https://www.cnblogs.com/agileai/p/7000064.html
               
               
               数据库优化总结
1优化全表扫描
	以下方式会触发全表扫描
对返回的行无任何限定条件，即没有where 子句，就是一个全表扫描



1.不要使用in操作符，这样数据库会进行全表扫描， 
推荐方案：在业务密集的SQL当中尽量不采用IN操作符

2.not in 使用not in也不会走索引 
推荐方案：用not exists或者（外联结+判断为空）来代替

3<> 操作符（不等于） 使用<>同样不会使用索引，因此对它的处理只会产生全表扫描 
推荐方案：用其它相同功能的操作运算代替，如 
a<>0 改为 a>0 or a<0
a<>’’ 改为 a>’’

4.IS NULL 或IS NOT NULL操作（判断字段是否为空）

判断字段是否为空一般是不会应用索引的，因为B树索引是不索引空值的。 
推荐方案：用其它相同功能的操作运算代替，如 
a is not null 改为 a>0 或a>’’等。

5.> 及 < 操作符（大于或小于操作符）
大于或小于操作符一般情况下是不用调整的，因为它有索引就会采用索引查找，但有的情况下可以对它进行优化，
如一个表有100万记录，一个数值型字段 A，30万记录的A=0，30万记录的A=1，39万记录的A=2，1万记录的A=3。那么执行A>2与A>=3的效果就有很大的区别了，
因为A>2时ORACLE会先找出为2的记录索引再进行比较，而A>=3时ORACLE则直接找到=3的记录索引。

6.LIKE操作符

LIKE操作符可以应用通配符查询，里面的通配符组合可能达到几乎是任意的查询，但是如果用得不好则会产生性能上的问题，
如LIKE ‘%5400%’ 这种查询不会引用索引，而LIKE ‘5400%’则会引用范围索引。 可以采用substr(column,1,4)=’5400’

7.UNION操作符
UNION在进行表链接后会筛选掉重复的记录，所以在表链接后会对所产生的结果集进行排序运算，删除重复的记录再返回结果。
 实际大部分应用中是不会产生重复的记录，最常见的是过程表与历史表UNION。如： 
select * from gc_dfys

union

select * from ls_jg_dfys

这个SQL在运行时先取出两个表的结果，再用排序空间进行排序删除重复的记录，最后返回结果集，如果表数据量大的话可能会导致用磁盘进行排序。

推荐方案：采用UNION ALL操作符替代UNION，因为UNION ALL操作只是简单的将两个结果合并后就返回。

8.SQL书写的影响（针对oracle而言）

同一功能同一性能不同写法SQL的影响

如一个SQL在A程序员写的为

Select * from zl_yhjbqk

B程序员写的为

Select * from dlyx.zl_yhjbqk（带表所有者的前缀）

C程序员写的为

Select * from DLYX.ZLYHJBQK（大写表名）

D程序员写的为

Select * from DLYX.ZLYHJBQK（中间多了空格）

以上四个SQL在ORACLE分析整理之后产生的结果及执行的时间是一样的，但是从ORACLE共享内存SGA的原理，可以得出ORACLE对每个 SQL 都会对其进行一次分析，并且占用共享内存，如果将SQL的字符串及格式写得完全相同则ORACLE只会分析一次，共享内存也只会留下一次的分析结果，这不仅可以减少分析SQL的时间，而且可以减少共享内存重复的信息，ORACLE也可以准确统计SQL的执行频率。

9.WHERE后面的条件顺序影响

WHERE子句后面的条件顺序对大数据量表的查询会产生直接的影响，如

Select * from zl_yhjbqk where dy_dj = '1K以下' and xh_bz=1

Select * from zl_yhjbqk where xh_bz=1 and dy_dj = '1K以下'

以上两个SQL中dy_dj及xh_bz两个字段都没进行索引，所以执行的时候都是全表扫描，第一条SQL的dy_dj = '1KV以下'条件在记录集内比率为99%，而xh_bz=1的比率只为0.5%，在进行第一条SQL的时候99%条记录都进行dy_dj及xh_bz的比较，而在进行第二条SQL的时候0.5%条记录都进行dy_dj及xh_bz的比较，以此可以得出第二条SQL的CPU占用率明显比第一条低。

10.查询表顺序的影响

在FROM后面的表中的列表顺序会对SQL执行性能影响，在没有索引及ORACLE没有对表进行统计分析的情况下ORACLE会按表出现的顺序进行链接，由此因为表的顺序不对会产生十分耗服务器资源的数据交叉。（注：如果对表进行了统计分析，ORACLE会自动先进小表的链接，再进行大表的链接）
--------------------- 
作者：zhaoweixs 
来源：CSDN 
原文：https://blog.csdn.net/zhaoweixs/article/details/8693076 
版权声明：本文为博主原创文章，转载请附上博文链接！
						   
