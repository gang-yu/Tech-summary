Consumer rebalance的时机



20s做一次

public class RebalanceService extends ServiceThread {

  /**

   \* 等待时间的间隔，毫秒，默认是20s

   */

  private static long waitInterval =

​    Long.parseLong(System.getProperty(

​      "rocketmq.client.rebalance.waitInterval", "20000"));



  @Override

  public void run() {

​    while (!this.isStopped()) {

​      // 等待20s，然后超时自动释放锁执行doRebalance

​      this.waitForRunning(waitInterval);

​      this.mqClientFactory.doRebalance();

​    }

  }

}



当一个consumer出现宕机后，默认最多20s，其它机器将重新消费已宕机的机器消费的queue，同样当有新的Consumer连接上后，20s内也会完成rebalance使得新的Consumer有机会消费queue里的msg。

等等，好像有问题：新上线一个Consumer要等20s才能负载均衡？这不是搞笑呢吗？肯定有猫腻。



新启动Consumer的话会立即唤醒沉睡的线程， 让他立马进行this.mqClientFactory.doRebalance();，源码如下

public class DefaultMQPushConsumerImpl implements MQConsumerInner {

  // 启动Consumer的入口函数

 public synchronized void start() throws MQClientException {     

​    // 看到了没！！！， 见名知意，立即rebalance负载均衡

​    this.mQClientFactory.rebalanceImmediately();

  }

}





https://zhuanlan.zhihu.com/p/159017211









消费者发送心跳到Broker，Broker端发现有新的消费者进来或者新增了topic订阅信息或者删除了topic订阅信息，Broker会通知所有消费者NOTIFY_CONSUMER_IDS_CHANGED，消费者收到请求后会立刻进行rebalance：MQClientInstance#rebalanceImmediately



https://cxis.me/2018/01/23/RocketMQ中Consumer的rebalance/