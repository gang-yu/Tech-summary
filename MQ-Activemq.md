**spring中如何使用activemq**



基本安装，直接执行官方文档的命令即可，不说了。



## 0 java中使用activemq

[Java消息队列--ActiveMq 初体验](https://www.cnblogs.com/jaycekon/p/6225058.html)



## 1 spring中使用 activemq

[Java消息队列-Spring整合ActiveMq](https://www.cnblogs.com/jaycekon/p/ActiveMq.html)



## 2 spring boot中使用activemq

[spring boot整合JMS(ActiveMQ实现)](https://blog.csdn.net/liuchuanhong1/article/details/54603546)



## 3 activemq中的事务？如何做消息确认
A:
https://www.cnblogs.com/SzeCheng/p/4792084.html

## 4 activemq消息消费失败如何处理？
A:
https://blog.csdn.net/whiteforever/article/details/49929497



## 学习路线

看看java如何直接调用activemq，在看看spring如何直接调用activemq，那么理解spring boot的调用就简单很多了。spring boot的auto config，配置了很多默认选项、以及template/listerner的实例化。



## 真的看明白了么？

看完上面3篇，你大致理解了么？好！那么，请问回答几个基础问题。

* 什么是JmsTemplate？与activemq什么关系？

* JmsTemplate有什么问题？

  JmsTemplate默认每次都会关闭connection。connenction是一次性的。

* 什么是MessageListener？

  可以通过JmsTemplate手动获取message，但是这样很可能不能及时获知message的到来。

  这是Listener就派上用场了。

* 为什么会用到CachingConnectionFactory或者PoolingConnectionFactory？





### reference

activeMQ - spring support官网 http://activemq.apache.org/spring-support.html
