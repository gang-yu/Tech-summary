# **RocketMq架构**

 https://help.aliyun.com/document_detail/112008.html?spm=a2c4g.11186623.6.561.2bea2010MjKChJ

![architecture](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/4229885751/p68921.png)

图中所涉及到的概念如下所述：

- Name Server：是一个几乎无状态节点，可集群部署，在消息队列 RocketMQ 版中提供命名服务，更新和发现 Broker 服务。
- Broker：消息中转角色，负责存储消息，转发消息。分为 Master Broker 和 Slave Broker，一个 Master Broker 可以对应多个 Slave Broker，但是一个 Slave Broker 只能对应一个 Master Broker。Broker 启动后需要完成一次将自己注册至 Name Server 的操作；随后每隔 30s 定期向 Name Server 上报 Topic 路由信息。
- 生产者：与 Name Server 集群中的其中一个节点（随机）建立长链接（Keep-alive），定期从 Name Server 读取 Topic 路由信息，并向提供 Topic 服务的 Master Broker 建立长链接，且定时向 Master Broker 发送心跳。
- 消费者：与 Name Server 集群中的其中一个节点（随机）建立长连接，定期从 Name Server 拉取 Topic 路由信息，并向提供 Topic 服务的 Master Broker、Slave Broker 建立长连接，且定时向 Master Broker、Slave Broker 发送心跳。Consumer 既可以从 Master Broker 订阅消息，也可以从 Slave Broker 订阅消息，订阅规则由 Broker 配置决定。



https://rocketmq.apache.org/docs/rmq-arc/

![img](https://rocketmq.apache.org/assets/images/rmq-basic-arc.png)



Apache RocketMQ is a distributed messaging and streaming platform with low latency, high performance and reliability, trillion-level capacity and flexible scalability. It consists of four parts: name servers, brokers, producers and consumers. Each of them can be horizontally extended without a single Point of Failure.



### RocketMq中各个角色是如何分配职责？

Ans:

English https://rocketmq.apache.org/docs/rmq-arc/

中文 https://www.cnblogs.com/panxuejun/p/8797103.html

https://github.com/apache/rocketmq/blob/master/docs/cn/architecture.md

**NameServer Cluster**

Name Servers provide lightweight service discovery and routing. Each Name Server records full routing information, provides corresponding reading and writing service, and supports fast storage expansion.

NameServer is a fully functional server, which mainly includes two features:

- Broker Management, **NameServer** accepts the register from Broker cluster and provides heartbeat mechanism to check whether a broker is alive.
- Routing Management, each NameServer will hold whole routing info about the broker cluster and the **queue** info for clients query.

**Broker Cluster**

Brokers take care of message storage by providing lightweight TOPIC and QUEUE mechanisms. They support the Push and Pull model, contains fault tolerance mechanism (2 copies or 3 copies), and provides strong padding of peaks and capacity of accumulating hundreds of billion messages in their original time order. In addition, Brokers provide disaster recovery, rich metrics statistics, and alert mechanisms



Broker server is responsible for message store and delivery, message query, HA guarantee, and so on.

As shown in image below, Broker server has several important sub modules:

- Remoting Module, the entry of broker, handles the requests from clients.

- Client Manager, manages the clients (Producer/Consumer) and maintains topic subscription of consumer.

- Store Service, provides simple APIs to store or query message in physical disk.

- HA Service, provides data sync feature between master broker and slave broker.

- Index Service, builds index for messages by specified key and provides quick message query.

  ![img](https://rocketmq.apache.org/assets/images/rmq-basic-component.png)



### RocketMq中的各个角色是如何做交互的？

Ans:

https://www.cnblogs.com/panxuejun/p/8797103.html

https://github.com/apache/rocketmq/blob/master/docs/cn/architecture.md

Broker（不管是Master还是Slave）和每一台NameServer机器来建立TCP连接。

Broker在启动的时候调用BrokerController中start方法，获取远程nameServerAddressList(远程NameServer服务列表)，

Broker对nameServerAddressList进行for循环处理，注册自己配置的topic信息到NameServer集群的每一台机器中。

即每一台NameServer都有该Broker的topic的配置信息。



Producer和NameServer：

每一个Producer与NameServer集群中的一台机器建立TCP连接，从这台NameServer上拉取路由信息。

 

Producer和broker：

Producer和它要发送的topic相关的Master类型的Broker建立TCP连接，用于发送消息以及定时的心跳信息。

Broker中记录该Producer的信息，供查询使用。

 

Consumer和NameServer：

每一个Consumer会和NameServer集群中的一台机器建立TCP连接，会从这台NameServer上拉取路由信息，进行负载均衡。



Consumer和Broker：

Consumer可以与Master或者Slave的Broker建立TCP连接来进行消费消息，Consumer也会向它所消费的Broker发送心跳信息，供Broker记录。





### RocketMq的一些常见问题

ans:

* Name servers
  * Broker向Namesr发心跳时，会带上当前自己所负责的所有Topic信息，如果Topic个数太多（万级别），会导致一次心跳中，就Topic的数据就几十M，网络情况差的话，网络传输失败，心跳失败，导致Namesrv误认为Broker心跳失败。
  * 服务注册与发现组件namesrv可以独立部署，且namesrv与namesrv之间并无直接或间接的关联，双方不存在心跳检测，所以namesrv的之间不存在主备切换过程，如果其中一台namesrv宕机后，生产者消费者会直接从另一台namesrv中请求数据。但也正因为namesrv之间不存在心跳检测，主备切换，所以无法支持动态扩容，当机器宕机需要重新部署基于新的ip地址的namesrv需要更新生产者/消费者直连的namesrv地址。

* Broker servers

  * broker支持主从架构模式，但是并非是非常完善的主从架构，据说会在后续版本中优化。

    * 只有当master读压力高于某个点（master消息拉取出现堆积时），才会将读压力转给salver。
    * 无法做到主从切换，master宕机，salver只能提供消息消费，salver不会被选举为master来继续工作。如果master宕机，消息队列整个环境近乎瘫痪

  * 多master多salver场景

    如果用户体量稍微大一些，单master单slaver扛不住，可以采用多master/多slaver部署架构。

  * broker同步/异步刷盘

    顾名思义，同步刷盘指消息投放到broker之后，会在写入文件之后才返回成功，而异步刷盘则指消息投放broker成功后即可返回，同时启动另外的线程来存储消息。

    同步刷盘的有点非常明显，可以保证消息可靠性，但是性能上无疑要略逊一筹。
    异步刷盘反其道而行之，性能上肯定有了显著提高，但是消息可靠性却无所保证，因为在文件写入过程失败，无法通知生产者重试。

  * 不支持master/salver选举

    从RocketMQ的架构设计中可以看到起优雅之处，组件均设计为无状态，所以它强大到可以任何扩展某个一个点。

    比如当发现broker压力较大时，可独立扩展broker，只需要将broker地址注册到namesrv中即可。namesrv检测到来自broker的心跳检测后，会将broker信息保存在可用broker列表中。不管是消费者还是生产者，都会按照某个负载均衡算法去选择broker地址。当其中一台broker机器宕机后，namesrv不会顷刻间摘除心跳检测（多次无心跳检测才会摘除），而生产者/消费者亦会有轮询的方式，在数次请求无果后，会从可用列表中将该broker剔除掉，并将请求转发到另外的机器上。

    优点：
    	1、任何组件节点支持集群部署扩展
    	2、同步刷盘保证数据不丢失，异步主从架构备份（数据可能会缺失）

    弊端：
    	1、不支持master/salver选举，单master部署的情况下，master的宕机几乎是一件毁灭性的打击。
    	2、主从读写分离的缺陷

  * 心跳时间差

    消费者每隔30秒从nameserver获取所有topic的最新队列情况，这意味着某个broker如果宕机，客户端最多要30秒才能感知。连接建立后，从namesrv中获取当前消费Topic所涉及的Broker，直连Broker。
    Consumer跟Broker是长连接，会每隔30秒发心跳信息到Broker。Broker端每10秒检查一次当前存活的Consumer，若发现某个Consumer 2分钟内没有心跳，就断开与该Consumer的连接，并且向该消费组的其他实例发送通知，触发该消费者集群的负载均衡。

    生产者，也是一样。

* consumer group

  * topic数量不能太少

  消费者端的负载均衡，就是集群消费模式下，同一个ID的所有消费者实例平均消费该Topic的所有队列。

  但是Consumer 数量要小于等于队列数量，如果Consumer 超过队列数量，那么多余的Consumer 将不能消费消息



### consumer如何处理某个broker宕机？

Ans:

  假如某个Broker宕机，意味生产者最长需要30秒才能感知到。在这期间会向宕机的Broker发送消息。当一条消息发送到某个Broker失败后，会往该broker自动再重发2次，假如还是发送失败，则抛出发送失败异常。业务捕获异常，重新发送即可。客户端里会自动轮询另外一个Broker重新发送，这个对于用户是透明的。

  https://blog.csdn.net/sunjin9418/java/article/details/79949693



### RocketMq中的topic、topic分片、queue是什么关系？

Ans:

https://blog.csdn.net/qq_34930488/article/details/101282436

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xNDI1NzgwNC00ZWIxNDQ2MTcwYzFiYTE1LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwfGltYWdlVmlldzIvMi93Lzg0My9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

一个Topic可以分布在各个Broker上，我们可以把一个Topic分布在一个Broker上的子集定义为一个Topic分片。对应上图，TopicA有3个Topic分片，分布在Broker1,Broker2和Broker3上，TopicB有2个Topic分片，分布在Broker1和Broker2上，TopicC有2个Topic分片，分布在Broker2和Broker3上。

将Topic分片再切分为若干等分，其中的一份就是一个Queue。每个Topic分片等分的Queue的数量可以不同，由用户在创建Topic时指定。

**数据分片的主要目的是突破单点的资源（网络带宽，CPU，内存或文件存储）限制从而实现水平扩展。RocketMQ 在进行Topic分片以后，已经达到水平扩展的目的了，为什么还需要进一步切分为Queue呢？**

Consumer按照集群消费的方式消费消息，按照平均分配策略进行负载均衡得到的结果是：第一个 Consumer 消费3个Queue，第二个Consumer 消费2个Queue。如果增加Consumer，每个Consumer分配到的Queue会相应减少。Rocket MQ的负载均衡策略规定：Consumer数量应该小于等于Queue数量，如果Consumer超过Queue数量，那么多余的Consumer 将不能消费消息。

在一个Consumer Group内，Queue和Consumer之间的对应关系是一对多的关系：一个Queue最多只能分配给一个Consumer，一个Cosumer可以分配得到多个Queue。这样的分配规则，每个Queue只有一个消费者，可以避免消费过程中的多线程处理和资源锁定，有效提高各Consumer消费的并行度和处理效率。

  由此，我们可以给出Queue的定义：

  Queue是Topic在一个Broker上的分片等分为指定份数后的其中一份，是负载均衡过程中资源分配的基本单元。



### RocketMq如何做到高可用、高性能、数据完整性？

Ans:

1，高并发读写服务

Broker的高并发读写主要是依靠以下两点：

消息顺序写，所有Topic数据同时只会写一个文件，一个文件满1G，再写新文件，真正的顺序写盘，使得发消息TPS大幅提高。
消息随机读，RocketMQ尽可能让读命中系统pagecache，因为操作系统访问pagecache时，即使只访问1K的消息，系统也会提前预读出更多的数据，在下次读时就可能命中pagecache，减少IO操作。
2，负载均衡与动态伸缩
负载均衡：Broker上存Topic信息，Topic由多个队列组成，队列会平均分散在多个Broker上，而Producer的发送机制保证消息尽量平均分布到所有队列中，最终效果就是所有消息都平均落在每个Broker上。
动态伸缩能力（非顺序消息）：Broker的伸缩性体现在两个维度：Topic, Broker。

 

Topic维度：假如一个Topic的消息量特别大，但集群水位压力还是很低，就可以扩大该Topic的队列数，Topic的队列数跟发送、消费速度成正比。
Broker维度：如果集群水位很高了，需要扩容，直接加机器部署Broker就可以。Broker起来后向Namesrv注册，Producer、Consumer通过Namesrv发现新Broker，立即跟该Broker直连，收发消息。
3，高可用&高可靠
高可用：集群部署时一般都为主备，备机实时从主机同步消息，如果其中一个主机宕机，备机提供消费服务，但不提供写服务。
高可靠：所有发往broker的消息，有同步刷盘和异步刷盘机制；同步刷盘时，消息写入物理文件才会返回成功，异步刷盘时，只有机器宕机，才会产生消息丢失，broker挂掉可能会发生，但是机器宕机崩溃是很少发生的，除非突然断电
原文链接：https://blog.csdn.net/sunjin9418/java/article/details/79949693



#### Q: RocketMq负载均衡如何做？

Ans：

RocketMq都是分布式无状态设计，可以高度扩展。

* broker是以group为单位提供服务。

  一个group里面分master和slave,master和slave存储的数据一样，slave从master同步数据（同步双写或异步复制看配置）。

  客户端关心（注册或发送）一个个的topic路由信息。路由信息中会细化为message queue的路由信息。

  由于压力分摊到了不同的queue,不同的queue实际上分布在不同的Broker group，也就是说压力会分摊到不同的broker进程，这样消息的存储和转发均起到了负载均衡的作用。

  Broker一旦需要横向扩展，只需要增加broker group，然后把对应的topic建上，客户端的message queue集合即会变大，这样对于broker的负载则由更多的broker group来进行分担

  * 每个topic下面有很多message queue，但是message queue本身并不存储消息。真正的消息存储会写在CommitLog的文件，message queue只是存储CommitLog中对应的位置信息，方便通过message queue找到对应存储在CommitLog的消息。

    不同的topic，message queue都是写到相同的CommitLog 文件，也就是说CommitLog完全的顺序写。

    ![broker负载均衡](http://jaskey.github.io/images/rocketmq/broker-loadbalance.png)

* producer是轮询的策略，向所有message queue发送数据。

* consumer在集群模式下，每个message queue分配到不同的consumer，达到水平线性扩展

> RocketMQ——水平扩展及负载均衡详解 http://jaskey.github.io/blog/2016/12/19/rocketmq-rebalance/



### name servers如何动态获得？

Ans:

Name server之间没有联系。没有主从备份、master选举。通过动态url拼接，不用写死在配置文件中。

```
public static String getWSAddr() {
        String wsDomainName = System.getProperty("rocketmq.namesrv.domain", DEFAULT_NAMESRV_ADDR_LOOKUP);
        String wsDomainSubgroup = System.getProperty("rocketmq.namesrv.domain.subgroup", "nsaddr");
        String wsAddr = "http://" + wsDomainName + ":8080/rocketmq/" + wsDomainSubgroup;
        if (wsDomainName.indexOf(":") > 0) {
            wsAddr = "http://" + wsDomainName + "/rocketmq/" + wsDomainSubgroup;
        }
        return wsAddr;
    }
———————————————— 
https://blog.csdn.net/qq_14957991/java/article/details/89873539
```



## 重要reference

* 官方设计文档 https://github.com/apache/rocketmq/tree/master/docs/cn

  涵盖，概念、特性、架构、设计。

* 