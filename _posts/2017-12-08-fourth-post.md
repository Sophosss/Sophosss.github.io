---
title: 你会判断亿级数据的存在与否吗?
author: Sophosss
layout: post
---
### Bloom Filter：

简单来说，它是一个很长的二进制向量和一系列随机映射函数，在一些分布式数据库中被广泛使用，比如我们熟悉的 HBase 等。它在这些数据库中扮演的角色就是判断一个值是否存在。这些分布式数据库之所以青睐它，就是因为它有很强大的性能，而且存储空间又小，空间效率和查询时间都远远超过一般的算法，缺点是牺牲了判断的准确率、删除的便利性。

### 数组和Hash：

和 map 不一样的是，数组中的值只有两种可能：0和1，hash 一般来说有多个 hash。每当插入一个值，就会把该值的几个 hash 后的映射值改为 1。那么，比如我要判断x是否存在，那么我就通过生成的 k 个 hash 函数来分别 hash 到数组的 k 个位置去，然后获取这个 k 个位置的值是否都为1，如果是，就认为 x 是极有可能存在的。反之，如果有一个位置的值为 0，那么 x 必然不存在。

![theme](https://Sophosss.github.io/assets/images/6.jpg)

### 实现与优化：

在使用和实现 Bloom Filter 时，不可避免要涉及到数据量，误判率，数组大小和 hash 函数的个数的等，在海量大数据的情况下，如果你的数组比较小，那么 hash 后取模落到数组中，就会大概率出现重叠，那么此时即使每个 hash 函数查出来都为 1 也不一定就表示某值存在。

所以，对于一个确定的场景，我们预估要存的数据量为 n，期望的误判率为 fpp，然后计算我们需要的数组的大小 m，以及 hash 函数的个数 k，并选择 hash 函数。

下面分别是根据预估数据量 n 以及误判率 fpp，bit 数组大小的 m 的计算方式，和根据预估数据量 n 以及数组长度 m，hash 函数的个数 k 的计算方式：

```java
/**
 * 计算 Bloom Filter的bit位数m
 *
 * @param n 预期数据量
 * @param p 误判率 (must be 0 < p < 1)
 */ 
@VisibleForTesting 
static long optimalNumOfBits(long n, double p) { 
  if (p == 0) { 
    p = Double.MIN_VALUE; 
  } 
  return (long) (-n * Math.log(p) / (Math.log(2) * Math.log(2))); 
} 
     
    
/**
 * 计算最佳k值，即在Bloom过滤器中插入的每个元素的哈希数
 *
 * @param n 预期数据量
 * @param m bloom filter中总的bit位数 (must be positive)
 */ 
@VisibleForTesting 
static int optimalNumOfHashFunctions(long n, long m) { 
  // (m / n) * log(2), but avoid truncation due to division! 
  return Math.max(1, (int) Math.round((double) m / n * Math.log(2))); 
```

哈希函数的选择对性能的影响是很大的，一个好的哈希函数要能近似等概率的将字符串映射到各个地方。Guava 里的 Bloom Filter 使用的是 Murmur 哈希算法，它是一种非加密型哈希函数，适用于一般的哈希检索操作。它与其它流行的哈希函数相比，对于规律性较强的 key，随机分布特征表现更良好，在 HBase，Redis，Kafka 中都有使用。

```java
long hash64 = Hashing.murmur3_128().hashObject(object, funnel).asLong();
```

### 应用场景：

常见的应用场景有如下几个：

- 黑名单

  最典型的一个应用就是黑名单功能，对用户名称或者 IP 或者 Email 进行过滤，每次检查时用 key 进行 hash 后，如果不在黑名单内的，肯定可以通行，如果在的则不允许通过，误判情况增加一个排除名单来进行排除。

- 爬虫重复URL检测

  爬取数据时，需要检测某个URL 是否已被爬取过，可用 bloom filter 过滤。

- 字典纠错

  在字典中检测某些单词是否拼写正确。

- 磁盘文件检测

  检测要访问的数据是否在磁盘或数据库中，比如在收集监控数据的时候， 有的系统的监控项量会很大，需要检查一个监控项的名字是否已经被记录到 DB 了， 如果没有的话就需要写入 DB。

- CDN缓存

  先查找本地有无 cache，如果没有则到其他兄弟 cache 服务器上去查找。为了避免无谓的查询，在每个 cache 服务器上保存其兄弟服务器的缓存关键字，以 Bloom Filter 方式存储。再去指定兄弟服务器查找之前，先检查 Bloom Filter 中是否有 URL，如果有，再去对应服务器查找。

- 垃圾邮件过滤

  如果用 hash 表，每存储一亿个 email 地址就需要 1.6GB 的内存（用哈希表实现的具体办法是将每一个 email 地址对应成一个八字节的信息指纹，然后将这些信息指纹存入哈希表，由于哈希表的存储效率一般只有 50%，因此一个 email 地址需要占用十六个字节。一亿个地址大约要 1.6GB，即十六亿字节的内存）。因此存储几十亿个邮件地址可能需要上百 GB 的内存。而Bloom Filter 只需要哈希表 1/8 到 1/4 的大小就能解决同样的问题。

