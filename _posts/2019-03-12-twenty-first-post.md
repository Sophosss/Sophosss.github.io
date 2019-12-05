---
title: Sql小优化
author: Sophosss
layout: post
---
今天发现一个 Sql 小优化。。记录一下吧

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

