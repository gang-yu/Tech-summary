## 快速入门

* https://blog.csdn.net/fly_fly_fly_pig/article/details/81812750 dubbo工程的创建

* http://dubbo.apache.org/en-us/docs/user/quick-start.html



https://zhuanlan.zhihu.com/p/266607403

## dubbo的工作原理

服务启动的时候，provider和consumer根据配置信息，连接到注册中心register，分别向注册中心注册和订阅服务
 

register根据服务订阅关系，返回provider信息到consumer，同时consumer会把provider信息缓存到本地。如果信息有变更，consumer会收到来自register的推送
 

3、consumer生成代理对象，同时根据负载均衡策略，选择一台provider，同时定时向monitor记录接口的调用次数和时间信息

拿到代理对象之后，consumer通过代理对象发起接口调用
 

5、provider收到请求后对数据进行反序列化，然后通过代理调用具体的接口实现



## 如何设计一个rpc框架

1. 首先需要一个服务注册中心，这样consumer和provider才能去注册和订阅服务
2. 需要负载均衡的机制来决定consumer如何调用客户端，这其中还当然要包含容错和重试的机制
3. 需要通信协议和工具框架，比如通过http或者rmi的协议通信，然后再根据协议选择使用什么框架和工具来进行通信，当然，数据的传输序列化要考虑
4. 除了基本的要素之外，像一些监控、配置管理页面、日志是额外的优化考虑因素。

![img](https://pic4.zhimg.com/80/v2-31d20842a882bac4d6e5df2abd13992b_1440w.jpg)