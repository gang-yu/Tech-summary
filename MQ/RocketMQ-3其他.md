


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



## consumer stops pull 

拉取消息较大，抑或处理消息时读取的数据库数据等较大，在jvm堆分配较小的情况下导致了OOM

https://github.com/apache/rocketmq/issues/1918#issuecomment-650994400