---
layout: post
title: "关于kafka和logflume的小节"
date: 2015-06-02 16:00:00 + 0800
comments: true
categories: scala 读书笔记
---
# Kafka & logflume

标签： kafka logflume MQ hadoop

---
##Kafka
1. 分布式的发布-订阅消息系统。
2. Apache顶级项目。
3. 由Linkedin开发并在公司内部大规模使用（实时tracking，data bridge）。

Kafka设计目标：

* Persistent messaging with O(1) disk structures that provide constant time performance even with many TB of stored messages.
* High-throughput: even with very modest hardware Kafka can support hundreds of thousands of messages per second. --10w
* Explicit support for partitioning messages over Kafka servers and distributing consumption over a cluster of consumer machines while maintaining per-partition ordering semantics.
* Easily scale out.
* Support for parallel data load into Hadoop.

###Kafka组成
![Kafka WorkFlow](https://kafka.apache.org/images/producer_consumer.png)

* Broker
 * topic
     * partition
* Producer
  * Sync Producer
  * Async Producer  
* Comsumer
  * Comsumer Group
* Zookeeper (optional, external) 

![Comsumer Group](https://kafka.apache.org/images/consumer-groups.png)


###Kafka的负载均衡
![Topics and Logs](https://kafka.apache.org/images/log_anatomy.png)


###消息队列工作模式选型

|Pub-Sub|Pull|Push|
|:-:|:---:|:---:|
|负责主体|下游端|上游段|
|实时性|取决于pull周期|较好|
|上游状态|无|要保存push信息|
|状态保存|分布式|集中式|
|安全性|安全|不保证|
|速度适应|根据consumer的消费能力来控制速度|由broker控制，很难适应消费速率不同的消费者|


对于Kafka而言，pull模式更合适。pull模式可简化broker的设计，Consumer可自主控制消费消息的速率，同时Consumer可以自己控制消费方式——即可批量消费也可逐条消费，同时还能选择不同的提交方式从而实现不同的传输语义。
　　
- - - 
###扩展阅读
关于Kafka的消息交付保障策略的介绍
> [Kafka Delivery Guarantee](https://kafka.apache.org/documentation.html#semantics)

push和pull的两篇介绍，通俗易懂
> [Thinking about Pubsh and Pull and Twitter in the Enterprise](http://confusedofcalcutta.com/2007/12/27/thinking-about-push-and-pull-and-twitter-in-the-enterprise/)
> [微博feed系统的推模式与拉模式和时间分区拉模式架构讨论](http://www.cnblogs.com/sunli/archive/2010/08/24/twitter_feeds_push_pull.html)

Kafka为何能面对TB级别并发读写 *微微一笑，绝对不抽!*
> [通过零拷贝实现有效数据传输](http://www.ibm.com/developerworks/cn/java/j-zerocopy/index.html)
> [Linux 中的零拷贝技术 part1](http://www.ibm.com/developerworks/cn/linux/l-cn-zerocopy1/index.html)
> [Linux 中的零拷贝技术 part2](http://www.ibm.com/developerworks/cn/linux/l-cn-zerocopy2/index.html)

- - - 

##logflume
###友盟数据平台收集日志的正确姿势
####logflume的前世今生与未来
####很久……很久以前…………  *久到已经不好找现成的配图了*
* 前端有N台nginx服务接log，把接收到的json log转给logtorrent再写成local text file
* 每小时rotate一次，并在本地gz压缩。Then scp到dp0，最终再写一个空的done文件
* 等N个done文件都齐全之后，degz->UALogParser->UALog(on HDFS)

存在问题：

* 单点compress与decompress，性能压力大
* 网络流量过于集中，每小时都会有单点性能问题
* 任务流水线长，优化空间大（前期准备工作1小时以上）
* 日志rotate切分不准，前后小时间会有log乱入！

####The Ultimate savior--logflume
![日志收集](http://7tsz8v.com1.z0.glb.clouddn.com/LogAggregate.png)
#一揽子解决以上所有问题，完美~
##（*此处有掌声*）

####使用kafka之后的其他bonus
* 并发度过高，缓解压力
* 上下游解耦，异步通信
* 冗余，容错处理
* 顺序保证，灵活性
* 可扩展性，峰值处理能力

###logflume---Mapreduce与kafka的纽带
####看logflume如何对接两者以及在概念上的对应关系：

|Kafka|Mapreduce|
|:---:|:---:|
|Topic|Log category|
|Partition|Map(split)|
|Producer|Map / Reducer|
|Consumer Group|Job's map|
|Consumer|Map|
|Zookeeper(其中offset部分)|HDFS|

**On-disk format of a message**

|message length | 4 bytes (value: 1+4+n) |
|-|-|
|"magic" value  | 1 byte|
|crc            | 4 bytes|
|payload        | n bytes|

####kafka的两种Java API
1. high level api
2. low！！！

####logflume如何解决log乱入问题
1. kafka的message是无时间概念的（不标准，有很弱的按时间切分能力）
2. logflume双offset设计
3. 可控的延迟机制

###logflume的今天和明天
1. 赋予logflume强大的流处理能力
2. 增加流量控制功能
    详见相关邮件和github的[issues 670](http://github.umeng.co/dp/iceberg/issues/670)
3. 增加reduce来控制输出文件个数

- - - 
###几次线上机器出现硬件问题的紧急修复分享
1. bla…… 
2. bla……
3. bla……

- - -
Q & A:

1. offset写入到zk中，每隔一段时间写一次， 这样很容易出现消费的offset没有及时写会zk，就导致一个消息多次消费，如何解决?

    So effectively Kafka guarantees ***at-least-once*** delivery by default and allows the user to implement at most once delivery by disabling retries on the producer and ***committing its offset prior to processing a batch of messages***. ***Exactly-once*** delivery requires co-operation with the destination storage system but Kafka provides the offset which makes implementing this straight-forward.
    
2. low level api 一般都怎么实现保存offset问题,因为这个是程序来控制offset更新,看过一些文章说把消费信息和offset放到hdfs中.
3. 当kafka长时间没有进行consume操作,再次consume时，会稍微有些时间延迟, 这样问题 咱们有遇到么?
4. kafka控制台上一直报[2015-06-02 17:08:40,898] INFO Closing socket connection to /10.15.190.61. (kafka.network.Processor)类似这样的错误,貌似是一个已知的问题，问问怎么不让console显示这样的问题


