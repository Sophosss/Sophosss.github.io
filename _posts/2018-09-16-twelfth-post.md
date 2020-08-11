---
title: 三种方法用mapreduce实现join
author: Sophosss
layout: post
---
假设表A跟表B进行join操作，连接条件是A.id=B.id

### 方法一：

#### 过程简述

- map阶段：
  - 遍历A表的每行记录，提取出id字段作为key，这一整行记录，加上一个区分AB表的字段，作为value输出到reduce阶段去
  - 对B表也进行如此操作
- reduce阶段：
  - 这样reduce收到的数据以id作为key的列表，这个列表有表A的数据也有表B的数据，只需要遍历该列表所有的记录进行A表和B表关联即可

#### 优点

- 通用，不管是多大规模的表都可进行操作
- 多表关联同样适用

#### 缺点

- 由于reduce阶段是以id为key，不可避免会有倾斜的问题
- 只能进行等号连接，不能进行不等号连接

### 方法二：

方法二建立在有一张表是小表的情况下

#### 过程简述

- map阶段：
  - 将小表分发到所有的map，map直接加载小表进内存，以id做key初始化一个map
  - 遍历大表所有的记录，通过id(假设id=1)查询内存中小表id等于1的记录，进行关联操作
- reduce阶段：（并没有reduce阶段）

#### 优点

- 没有倾斜问题
- 连接条件可以是任意条件，可以有大于号，小于号，不等号等等

#### 缺点

- 建立在有一张表是小表的情况下

### 方法三

方法二是将小表数据放进内存初始化成一个map，方法三就是引用第三方数据库，将其中一张表的数据放进redis或hbase，在map阶段遍历另一张表进行关联操作