---
title: Sql小优化
author: Sophosss
layout: post
---
今天总结下两个 Sql 小优化。。记录一下吧

在业务逻辑中，很有可能会遇到这样的情况：我要更新一条记录的值。但是我不确定这条记录存不存在。那如果存在我就更新，如果不存在我就插入。

通常情况下，就是写出三条 Sql ，第一条 Sql 查询这条记录，然后用程序判断，如果存在，则更新，如果不存在，则插入。

那么有没有一次性解决的呢？

有的。

```sql
BEGIN
    #定义该记录是否存在
    declare num int;
    
    #into num就是把count(*)查出的值，赋给到num中
    select count(*) into num from db where TO_DAYS(now())=TO_DAYS(day);
    
    #判断
    if(num=0)
        Then
        insert into db(name,id,day)values(1,1,now());

    else
        update db set name=name+1;

  end if;
END
```



第二个优化是，当limit offset rows中的 offset 很大时，会出现效率问题，比如下面这条语句：

```sql
select * from test where val=4 limit 300000,5;
```

它的查询过程分为：

- 查询到索引叶子节点数据。
- 根据叶子节点上的主键值去聚簇索引上查询需要的全部字段值。

也就是说它查询了300005次索引节点，查询300005次聚簇索引的数据，最后再将结果过滤掉前300000条，取出最后5条。MySQL耗费了大量随机I/O在查询聚簇索引的数据上，而有300000次随机I/O查询到的数据是不会出现在结果集当中的。

将语句优化为如下，就可以将数据页的访问次数缩小到只有5次。

```sql
select * from test a inner join (select id from test where val=4 limit 300000,5) b on a.id=b.id;
```

语句速度对比如下：

```sql
mysql> select * from test where val=4 limit 300000,5;
+---------+-----+--------+
| id      | val | source |
+---------+-----+--------+
| 3327622 |   4 |      4 |
| 3327632 |   4 |      4 |
| 3327642 |   4 |      4 |
| 3327652 |   4 |      4 |
| 3327662 |   4 |      4 |
+---------+-----+--------+
5 rows in set (26.19 sec)



mysql> select * from test a inner join (select id from test where val=4 limit 300000,5) b on a.id=b.id;
+---------+-----+--------+---------+
| id      | val | source | id      |
+---------+-----+--------+---------+
| 3327622 |   4 |      4 | 3327622 |
| 3327632 |   4 |      4 | 3327632 |
| 3327642 |   4 |      4 | 3327642 |
| 3327652 |   4 |      4 | 3327652 |
| 3327662 |   4 |      4 | 3327662 |
+---------+-----+--------+---------+
5 rows in set (0.09 sec)
```

