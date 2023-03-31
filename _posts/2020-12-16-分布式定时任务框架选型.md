#### 一、背景

近期规划提出要将线上项目部署为多实例，多实例场景下必然要处理定时任务重复执行、负载均衡的问题。

本文档为解决**分布式场景下定时任务重复执行**的问题提供选型参考。其中大部分描述来自互联网资料整合，相关链接在文档底部。

主要需求如下：

1.避免重复执行

2.动态CRUD，如动态启停、业务中允许更新执行时间。

3.失效转移、负载均衡等分布式作业管理

4.定时任务管理，如查看当前任务列表、手动启停等。

5.日志回溯、异常报警等。

#### 二、概览

经过检索找到如下几种用户群较多的候选框架。

quartz是最经典最著名的任务调度框架，其他候选至少初期都是基于它开发，进而改进了quartz的问题。

| 候选                                            | 缺点                                                                                        | 复杂度                                     | 背景         |
| --------------------------------------------- | ----------------------------------------------------------------------------------------- | --------------------------------------- | ---------- |
| quartz                                        | 1.通过独占锁来保证只有一个节点执行，没有负载均衡。<br>2.需要持久化任务信息到业务数据表，据说有10张以上，有侵入性。<br>3.调度和执行并存于同一个项目，相互影响性能。 |                                         |            |
| elastic-job（elasticjob-lite，elasticjob-cloud） |                                                                                           | 最重，依赖zookeeper，mesos（仅elasticjob-cloud） | 当当网2015开源  |
| XXL-job                                       | 基于数据库的集群方案，意味着会有性能瓶颈，但是目前的需求还远远达不到需要加机器解决瓶颈的程度。                                           | 最轻，主打开箱即用                               | 大众点评2015开源 |
| Staturn                                       | 文档少                                                                                       | zookeeper                               | 唯品会2016开源  |

#### 三、Elastic-Job VS XXL-Job

由于以上，接下来只需要详细对比一下Elastic-Job和XXL-Job。

| 维度   | ElasticJob                                                                                                    | XXL-Job                       |
| ---- | ------------------------------------------------------------------------------------------------------------- | ----------------------------- |
| 依赖   | ZooKeeper，mesos（仅elasticjob-cloud）                                                                            |                               |
| 易用性  | 较复杂                                                                                                           | 一般                            |
| 场景   | 注重数据设计，适合数据量大、服务器多的场景。                                                                                        | 数据量较小、服务器较少的快速开发场景，主打轻量级。     |
| 注册中心 | Zookeeper                                                                                                     |                               |
| 其他   | 当当网2015年开源；于2020年5月28日成为 Apache ShardingSphere 的子项目；github star 6.7k（应该是因为2020年5月才回归所以数据相对较低，在此之前有两年没更新）      | 大众点评2015开源；github star 16.7k； |
| 独有   | 资源分配，应用分发，进程级调度，瞬时任务，进程隔离。（Mesos自研框架实现）                                                                       |                               |
| 都有   | 1.作业治理（失效转移，错过重新执行，自诊断修复，负载均衡）<br/>2.弹性扩容，数据分片。<br/>3.可视化管理，任务统计，日志，报警。<br/>4.动态CRUD<br/>5.与Spring、Dubbo配合良好。 |                               |

#### 四、参考资料

Quartz：

https://www.w3cschool.cn/quartz_doc/

ElasticJob官方：

https://shardingsphere.apache.org/elasticjob/current/cn/overview/

https://github.com/apache/shardingsphere-elasticjob

XXL-Job官方：

https://www.xuxueli.com/xxl-job/

https://github.com/xuxueli/xxl-job

Saturn官方：

https://github.com/vipshop/Saturn

互联网资源（存在时效性，提到elastic-job不再更新的为过期文档）：

https://blog.csdn.net/en_joker/article/details/104407313

https://yq.aliyun.com/articles/748660

https://www.cnblogs.com/ssslinppp/p/12485273.html

https://blog.csdn.net/luomao2012/article/details/107035609

https://my.oschina.net/u/2274056/blog/3096594

https://zhuanlan.zhihu.com/p/160260842

https://zhuanlan.zhihu.com/p/138967032