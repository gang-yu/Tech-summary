**spring中如何使用activemq**



基本安装，直接执行官方文档的命令即可，不说了。



## 0 java中使用activemq

[Java消息队列--ActiveMq 初体验](https://www.cnblogs.com/jaycekon/p/6225058.html)



## 1 spring中使用 activemq

[Java消息队列-Spring整合ActiveMq](https://www.cnblogs.com/jaycekon/p/ActiveMq.html)



## 2 spring boot中使用activemq

[spring boot整合JMS(ActiveMQ实现)](https://blog.csdn.net/liuchuanhong1/article/details/54603546)



> 学习步骤

看看java如何直接调用activemq，在看看spring如何直接调用activemq，那么理解spring boot的调用就简单很多了。spring boot的auto config，配置了很多默认选项、以及template/listerner的实例化。



> 检验

**真的看明白了么？**

看完上面3篇，你大致理解了么？好！那么，请问回答几个基础使用问题。

* 什么是JmsTemplate？与activemq什么关系？

* JmsTemplate有什么问题？

  hint：
  JmsTemplate默认每次都会关闭connection。connenction是一次性的。

* 什么是MessageListener？

  hint:
  可以通过JmsTemplate的receiver手动获取message，但是这样很可能不能及时获知message的到来。

  这时Listener就派上用场了。

* 为什么会用到CachingConnectionFactory或者PoolingConnectionFactory？
Ans:
https://medium.com/@bdarfler/efficient-lightweight-jms-with-spring-and-activemq-51ff6a135946 



## 进阶

## activemq使用api过程涉及到哪些主体？不包含自己定义的。
Ans:
https://blog.csdn.net/fangxinde/article/details/88305666
实体有：
broker（activemq服务端的代理）
messageProducer 
messageConusmer
关联:
connection
session


## activemq事务是什么？有什么用？如何做消息确认
A:
消息事务是在生产者producer到broker或broker到consumer过程中同一个session中发生的，保证几条消息在发送过程中的原子性。
    在支持事务的session中，producer发送message时在message中带有transactionID。broker收到message后判断是否有transactionID，如果有就把message保存在transaction store中，等待commit或者rollback消息。https://blog.csdn.net/songhaifengshuaige/java/article/details/54176849

## activemq消息消费失败如何处理？
A:
https://blog.csdn.net/whiteforever/article/details/49929497


## activem的死信队列Dead LetterQueue有什么用？
hint:
http://activemq.apache.org/message-redelivery-and-dlq-handling.html

## activemq接受消息方法中，JmsTemplate的receive与MessageListerner的onMessage有什么区别？
hint:
从阻塞、异步的角度来思考。
A:

* receive是blocking的。
  它一直在等待下一个消息。只要读取到消息，才会返回。同时，它只要处理当前的消息才会去读取下一条。
  onMessage是异步的。
  只有在有消息到来的时候才会被调用。

* receive没有session的pooling
https://medium.com/@bdarfler/efficient-lightweight-jms-with-spring-and-activemq-51ff6a135946 
  onMessage或者说DefaultMessageListenerContainer能pool这些session。当然connection不pooling。



### reference

activeMQ - spring support http://activemq.apache.org/spring-support.html
activeMQ - FAQ>JMS http://activemq.apache.org/jms
how do I use JMS efficiently http://activemq.apache.org/how-do-i-use-jms-efficiently.html
