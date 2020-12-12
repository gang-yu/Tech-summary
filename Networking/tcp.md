

![img](https://pic3.zhimg.com/80/v2-598eee1c80c322c65521c359cb0ba8a1_1440w.jpg?source=1940ef5c)

tcp整个过程是一个状态机。

![img](https://images2018.cnblogs.com/blog/1158196/201803/1158196-20180301191336765-262732211.png)

![img](https://images2018.cnblogs.com/blog/1158196/201803/1158196-20180301190534920-1824529844.png)



## 为什么tcp建立连接是三次握手？

```
A -----SYN-----> B
SYN_SEND         SYN_RCVD
A <----SYN+ACK------ B
ESTABLISHED
A -----ACK-----> B
                 ESTABLISHED
```
3次至少能证明对方能收能发。



## tcp断开连接的过程

```
A -----FIN-----> B
FIN_WAIT_1       CLOSE_WAIT
A <----ACK------ B
FIN_WAIT_2

(B can send more data here, this is half-close state)

A <----FIN------ B
TIME_WAIT        LAST_ACK
A -----ACK-----> B
|                CLOSED
2MSL Timer
|
CLOSED
```

https://networkengineering.stackexchange.com/questions/38805/why-is-the-last-ack-needed-in-tcp-four-way-termination

https://www.cnblogs.com/dj0325/p/8490293.html

https://www.zhihu.com/question/27564314/answer/162476313