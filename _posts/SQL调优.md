结构优化

1. 创建索引。
2. 对于Mysql 的 InnoDB 引擎，使用一个业务无关的自增主键。（InnoDB的数据文件本身要按主键聚集，所以InnoDB要求表必须有主键（MyISAM可以没有），如果没有显式指定，则MySQL系统会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在这种列，则MySQL自动为InnoDB表生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整形。）
3. 最左前缀原理：

查询优化

1. 避免在索引上使用计算。
2. 尽量用 where 替代 having，where 中条件的顺序，是过滤最多的条件放在最前面。
3. 使用预编译的语句，这样既能防止 SQL 注入，还能提高效率。
4. 对于select 语句，尽量避免使用 select * ，而是用什么 select 什么。某些时候可以避免 InnoDB 中的回表查询。
5. 对表使用别名。
6. 使用 union all 替代  union。
7. 尽量将多条SQL 简化为一条。
8. 使用 explain 分析 select SQL 语句。
9. 只在必要的时候使用 begin transaction。

慢查询排错

1. 先执行，确认是SQL执行慢
2. 对每个条件单独执行 select，查看是那个select 最慢。
3. 





参考：

[](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

