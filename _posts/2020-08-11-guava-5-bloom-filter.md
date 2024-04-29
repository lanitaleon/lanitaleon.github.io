---
layout: post
title: Guava(5) Bloom Filter
---

#### 1.简介

[布隆过滤器]([https://zh.wikipedia.org/wiki/%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8](https://zh.wikipedia.org/wiki/布隆过滤器))（Bloom Filter）用于检查一个元素是否存在一个集合中。

一般来说判断一个元素是否存在于集合中会将元素全部加载出来进行对比，如果元素数量过多则会导致内存溢出等问题。

原理简介：

将集合的所有元素都映射为一个位数组中的K个点，并置为1。

检查指定元素的对应点的值是否为1就代表该元素是否存在于集合中。

优点：时间少，空间少。

缺点：误算率，不可删除。

误算率是指，如果Bloom Filter的结果显示某个元素存在，其实代表该元素大概率存在。（不存在则一定不存在）

如果存入的元素数量少，则直接使用hash结构即可；

如果存入的元素数量多，则难免提升了误算率；

另外，不管是什么hash算法，都有可能会出现两个不同的元素得到相同的结果。

不可删除是因为删除一个元素前，首先要保证这个元素存在集合中，而这点不能被Bloom Filter保证。

不同的hash算法和调优策略导致Bloom Filter有很多变种，guava也有自己的实现。

```java
BloomFilter<Integer> filter = BloomFilter.create(
        Funnels.integerFunnel(),
        100000000,
        0.01);
for (int i = 0; i < 100000000; i++) {
    filter.put(i);
}
filter.mightContain(1)
```

#### 2.源码分析

guava的调优策略，实例创建，存取实现。

如何自实现Bloom Filter。

使用场景。

参考文档：

https://crossoverjie.top/2018/11/26/guava/guava-bloom-filter/

http://www.sigma.me/2011/09/13/hash-and-bloom-filter.html

https://www.cnblogs.com/CodeBear/p/10911177.html
