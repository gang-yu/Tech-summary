## 死信队列

（场景）在当正常业务处理时出现异常时，将消息拒绝则会进入到死信队列中，有助于统计异常数据并做后续的数据修复处理。

（产生）消费端，一直不回传消费的结果。rocketmq认为消息没收到，consumer下一次拉取，broker依然会发送该消息。

所以，任何异常都要返回ConsumeConcurrentlyStatus.RECONSUME_LATER，这样MQ会将消息放到重试队列。

这个重试TOPIC的名字是%RETRY%+consumergroup的名字

重试的消息在延迟的某个时间点（默认是10秒，业务可设置）后，再次投递到这个ConsumerGroup。而如果一直这样重复消费都持续失败到一定次数（默认16次），就会投递到DLQ死信队列重试队列在重试16次（默认次数）将消息放入死信队列

（解决）死信队列中的数据需要通过新订阅该topic进行消费。


## 坑
### instanceName

https://blog.csdn.net/lmx125254/article/details/90380874
rocketmq.clinet.name 默认是写死为 DEFAULT 的，所以我们所获取的 instanceName 在每次启动时虽然不一样，但是全局获取的 instanceName是一样的
>instanceName一定要设置


### jar不一致导致的重复消费
每次重启项目都会重复消费之前的消息，即使消费时直接返回ConsumeConcurrentlyStatus.CONSUME_SUCCESS也还是消费不成功，查看rocketmq-console时发现，difftotal时一直存在的，也就是说rocketmq认为是没有消费成功的

> rocketmq服务端和代码中的mqclient包必须保证一致

### 相同topic下的queue消费不平衡

http://cloudarchitectures.cn/rocketmq_consumer_unbalance/