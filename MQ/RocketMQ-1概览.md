

## **RocketMQ概览**

 https://help.aliyun.com/document_detail/155952.html?spm=a2c4g.11186623.6.549.7d0b7e80KC42oJ

![functionswithoutstomp](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6771879851/p68860.png)



## **RocketMq消息收发模型**

https://help.aliyun.com/document_detail/29532.html?spm=a2c4g.11186623.6.547.59782010PxlKcU

![messagingmodel](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6718273851/p68843.png)




## **适用场景**
https://help.aliyun.com/document_detail/112010.html?spm=a2c4g.11186623.6.560.5d0974a6OvX3gO#title-9bx-nr5-ifk

* 异步解耦

  ![img](https://img.alicdn.com/tfs/TB1t7EInsUrBKNjSZPxXXX00pXa-1134-910.png)

  ![img](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6219985751/p68913.png)

* 分布式事务的数据一致性
  * 普通消息
    ![inconsistentstatus](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6219985751/p68914.png)

  * 事务消息

![consistentstatus](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6219985751/p68919.png)


https://help.aliyun.com/document_detail/43348.html?spm=a2c4g.11186623.2.21.380a425cfoI8Cp#concept-2047067
事务消息/半事务消息/消息回查

![img](https://img.alicdn.com/tfs/TB1i2jrnRjTBKNjSZFwXXcG4XXa-1530-1140.png)

https://help.aliyun.com/document_detail/43348.html?spm=a2c4g.11186623.6.554.3e63425c0uJZQD

![事务消息](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6087385851/p69094.png)

* 消息顺序收发

  - 全局顺序：对于指定的一个 Topic，所有消息将按照严格的先入先出（FIFO）的顺序，进行顺序发布和顺序消费；

  - 分区顺序：对于指定的一个 Topic，所有消息根据 Sharding Key 进行区块分区，同一个分区内的消息将按照严格的 FIFO 的顺序，进行顺发布和顺序消费，可以保证一个消息被一个进程消费。

    在注册场景中，可使用用户 ID 作为 Sharding Key 来进行分区，同一个分区下的新建、更新或删除注册信息的消息必须按照 FIFO 的顺序发布和消费。

    ![img](https://img.alicdn.com/tfs/TB1PjVHumzqK1RjSZFjXXblCFXa-1125-800.gif)

* 削峰填谷

  ![img](https://img.alicdn.com/tfs/TB1DuTrnRjTBKNjSZFwXXcG4XXa-1135-911.png)

* 大规模机器的缓存同步

 ![img](https://img.alicdn.com/tfs/TB1ki7KXgMPMeJjy1XdXXasrXXa-1530-1140.png)





## RocketMQ——角色与术语详解
  http://jaskey.github.io/blog/2016/12/15/rocketmq-concept/

https://github.com/apache/rocketmq/blob/master/docs/cn/concept.md

https://github.com/apache/rocketmq/blob/master/docs/cn/features.md

> 角色：
> ​	producer
> ​	consumer（pull consumer/push consumer）
>
> 
> 术语：
> 	​	producer group
> 	​	consumer group
> 	​	topic
> 	​	tag
> 	​	message queue
> 	​	offset
> 	​	集群消费
> 	​	广播消费
> 	​	顺序消息
> 		​	普通顺序消息
> 		​	严格顺序消息
>

> 
> 上面这些术语**你都懂了么？**

深入理解源码需要理解的术语

> page cache页缓存
>
> ​			OS对文件的缓存。在读取CommitLog，会预读取多一点的消息到page cache；写入CommitLog，也是先写入page cache，然后刷盘到disk。
>
> process queue
>
> ​			consume拉取消息的本地缓存queue



### 代码样例

https://github.com/apache/rocketmq/blob/master/docs/cn/RocketMQ_Example.md

