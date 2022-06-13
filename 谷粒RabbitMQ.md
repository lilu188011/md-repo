## RabbitMQ

### 一、MQ简介

```
大多应用中，可通过消息服务中间件来提升系统异步通信、扩展解耦能力。
消息的发送者和接收者不需要同时与消息队列交互。消息会保存在队列中，知道接收者取回它。
```

下面是架构图：

![img](https://raw.githubusercontent.com/lilu188011/img-repo/master/7ny8Sxglws.png)

- Producer：消息生产者，负责生产和发送消息到Broker；
- Broker：消息处理中心，负责消息存储、确认、重试等；
- Consumer：消息消费中心，负责从Broker中获取消息并处理。

**消息队列-特性**

- 异步性：将耗时的同步任务通过发送消息的方式进行异步处理，减少等待时间。
- 松耦合：不同系统、服务之间可以通过消息队列进行通信，不用关心彼此的实现细节，数据格式一致。
- 分布式：为了防止消息堵塞，可以对消费者集群进行横向扩展，避免单点故障，同样队列本身也可以。
- 可靠性：将接收到的消息落盘，就算服务器重启或者发生故障，恢复之后也能重新加载。

**RabbitMQ的核心概念**

RabbitMQ是一个由erlang开发的AMQP(Advanved Message Queue Protocol)的开源实现。

**Message**

- 消息，消息是不具名的，它由消息头和消息体组成
- 消息头，包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等

**Publisher**

- 消息的生产者，也是一个向交换器发布消息的客户端应用程序

**Exchange**

- 交换器，将生产者消息路由给服务器中的队列
- 类型有direct(默认)，fanout, topic, 和headers，具有不同转发策略

**Queue**

- 消息队列，保存消息直到发送给消费者

**Binding**

- 绑定，用于消息队列和交换器之间的关联

**Connection**

- 网络连接，比如一个TCP连接

**Channel**

- 信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内的虚拟连接，AMQP命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成的。因为对于操作系统来说建立和销毁TCP都是非常昂贵的开销，所以引入了信道的概念，以复用一条TCP连接。

**Consumer**

- 消息的消费者，表示一个从消息队列中取得消息的客户端应用程序

**Virtual Host**

- 虚拟主机，表示一批交换器、消息队列和相关对象。
- vhost 是 AMQP 概念的基础，必须在连接时指定
- RabbitMQ 默认的 vhost 是 /

**Broker**

- 消息队列服务器实体

![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/J4fCJgjnrc.png)

### 二、Docker 安装RabbitMQ

```
docker run -d --name rabbitmq -p 5671:5671 -p 5672:5672 \
-p 4369:4369  -p 25672:25672 -p 15671:15671 -p 15672:15672 \
rabbitmq:management
```

**说明**

```
4369,25672 (Erlang发现&集群端口）
5672,5671（AMQP端口）
15672（web管理后台端口）
61613，61614（STOMP协议端口）
1883，8883（MQTT协议端口）
```

设置开机自启动：

```
docker update rabbitmq --restart=always
```

访问管理页面：

http://192.168.10.10:15672/#/

![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/e7oOyJjWdR.png)

### 三、RabbitMQ消息发送测试

RabbitMQ提供了四种Exchange模式：fanout,direct,topic,header 。 header模式在实际使用中较少，这里只讨论前三种模式.

#### fanout 模式

fanout 模式就是广播模式~，消息来了，会发给所有的队列~

![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/FJRA7rQdpo.png)

测试广播模式：
先在交换机创建 fanout模式的交换机，命名为 `my.fanout.exchange`，然后再到队列创建多个队列，再到交换机绑定队列，fanout可以不设置路由key，因为这个是广播模式的，最后发消息测试。

![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/0m1Hl4YcMY.png)

队列接收消息：

![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/g3WxNQVNak.png)

可以看到绑定的队列已经收到消息了。

#### Direct 模式

Direct 模式就是指定队列模式， 消息来了，只发给指定的 Queue, 其他Queue 都收不到。

![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/YvMx2yexTE.png)

#### Topic 模式

主题模式，注意这里的主题模式，和 ActivityMQ 里的不一样。 ActivityMQ 里的主题，更像是广播模式。
那么这里的主题模式是什么意思呢？ 如图所示消息来源有： 美国新闻，美国天气，欧洲新闻，欧洲天气。
如果你想看 美国主题： 那么就会收到 美国新闻，美国天气。
如果你想看 新闻主题： 那么就会收到 美国新闻，欧洲新闻。
如果你想看 天气主题： 那么就会收到 美国天气，欧洲天气。
如果你想看 欧洲主题： 那么就会收到 欧洲新闻，欧洲天气。

这样就可以灵活搭配~

![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/OXR8Xm1IDf.png)

### 三、springboot整合RabbitMQ

#### 1) 引入依赖

```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

**说明：**

```
使用RabbitMQ
1、引入amqp场景;RabbitAutoConfiguration就会自动生效
2、给容器中自动配置了
      RabbitTemplate、AmqpAdmin、CachingConnectionFactory、RabbitMessagingTemplate
            @ConfigurationProperties(prefix="spring.rabbitmq")
            public class RabbitProperties
3、给配置文件中配置 spring.rabbitmq 信息
4、@EnableRabbit (开启RMQ注解)
```

#### 2) 启动类开启RMQ注解

```java
 @EnableRabbit   // 开启消息队列
// 添加注册发现功能
@EnableDiscoveryClient
@MapperScan("com.atguigu.gulimall.order.dao")
@SpringBootApplication
public class GulimallOrderApplication {

  public static void main(String[] args) {
    SpringApplication.run(GulimallOrderApplication.class, args);
  }

}
```

#### 3) 配置文件增加RMQ属性

```properties
 # RabbitMQ配置
spring.rabbitmq.host=192.168.10.10
spring.rabbitmq.port=5672
# 虚拟主机配置
spring.rabbitmq.virtual-host=/
# 开启发送端消息抵达Broker确认
spring.rabbitmq.publisher-confirms=true
# 开启发送端消息抵达Queue确认
spring.rabbitmq.publisher-returns=true
# 只要消息抵达Queue，就会异步发送优先回调returnfirm
spring.rabbitmq.template.mandatory=true
# 手动ack消息，不使用默认的消费端确认
spring.rabbitmq.listener.simple.acknowledge-mode=manual

```

#### 4) 使用RMQ

```java
 package com.atguigu.gulimall.order;

import com.atguigu.gulimall.order.entity.OrderReturnReasonEntity;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.AmqpAdmin;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Exchange;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;

import java.util.Date;
import java.util.HashMap;
import java.util.UUID;

@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class GulimallOrderApplicationTests {

  @Autowired
  private AmqpAdmin amqpAdmin;   // 创建交换机名，创建队列，创建绑定

  @Autowired
  private RabbitTemplate rabbitTemplate;  // 发送消息相关

  /**
   * 1、如何创建Exchange、Queue、Binding
   *      1）、使用AmqpAdmin进行创建
   * 2、如何收发消息
   */
  @Test
  public void createExchange() {

    Exchange directExchange = new DirectExchange("hello-java-exchange",true,false);
    amqpAdmin.declareExchange(directExchange);
    log.info("Exchange[{}]创建成功：","hello-java-exchange");
  }

  @Test
  public void testCreateQueue() {
    Queue queue = new Queue("hello-java-queue",true,false,false);
    amqpAdmin.declareQueue(queue);
    log.info("Queue[{}]创建成功：","hello-java-queue");
  }

  @Test
  public void createBinding() {

    Binding binding = new Binding("hello-java-queue",
        Binding.DestinationType.QUEUE,
        "hello-java-exchange",
        "hello.java",
        null);
    amqpAdmin.declareBinding(binding);
    log.info("Binding[{}]创建成功：","hello-java-binding");

  }

  @Test
  public void create() {
    HashMap<String, Object> arguments = new HashMap<>();
    arguments.put("x-dead-letter-exchange", "order-event-exchange");
    arguments.put("x-dead-letter-routing-key", "order.release.order");
    arguments.put("x-message-ttl", 60000); // 消息过期时间 1分钟
    Queue queue = new Queue("order.delay.queue", true, false, false, arguments);
    amqpAdmin.declareQueue(queue);
    log.info("Queue[{}]创建成功：","order.delay.queue");
  }

    @Test
  public void sendMessageTest() {
    OrderReturnReasonEntity reasonEntity = new OrderReturnReasonEntity();
    reasonEntity.setId(1L);
    reasonEntity.setCreateTime(new Date());
    reasonEntity.setName("reason");
    reasonEntity.setStatus(1);
    reasonEntity.setSort(2);
    String msg = "Hello World";
    //1、发送消息,如果发送的消息是个对象，会使用序列化机制，将对象写出去，对象必须实现Serializable接口

    //2、发送的对象类型的消息，可以是一个json
    rabbitTemplate.convertAndSend("hello-java-exchange","hello2.java",
        reasonEntity,new CorrelationData(UUID.randomUUID().toString()));
    log.info("消息发送完成:{}",reasonEntity);
  }
}
```

#### 5)RMQ队列监听

~~~java
`com/atguigu/gulimall/order/service/impl/OrderItemServiceImpl.java`
```
 /**
 * 监听队列
 * 参数可以写类型
 * 1、Message message:原生消息详细信息。头+体
 * queues：声明需要监听的所有队列
 * channel：当前传输数据的通道
 *
 * Queue:可以很多人来监听。只要收到消息，队列删除消息，而且只能有一个收到消息（分布式场景）
 * 场景：
 *     1）、订单服务启动多个：同一个消息，只能有一个客户端收到
 */
@RabbitListener(queues = {"hello-java-queue"})
public void revieveMessage(Message message,
    OrderReturnReasonEntity content) {
    //拿到主体内容
    byte[] body = message.getBody();
    //拿到的消息头属性信息
    MessageProperties messageProperties = message.getMessageProperties();
    System.out.println("接受到的消息...内容" + message + "===内容：" + content);

}
```
~~~

### 四、RabbitMQ消息队列-可靠投递

#### 1) 发送端确认

保证消息不丢失，可靠抵达，可以使用事务消息，性能下降250倍，为此引入确认机制。

- publisher confirmCallback 确认模式
- publisher returnCallback 未投递到 queue 退回模式
- consumer ack机制

![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/iLEi4HN1Aw.png)

修改 `application.properties` 文件

```properties
# 开启发送端消息抵达Broker确认
spring.rabbitmq.publisher-confirms=true
# 开启发送端消息抵达Queue确认
spring.rabbitmq.publisher-returns=true
```

创建配置文件：`gulimall-order/xxx/order/config/MyRabbitConfig.java`

数据转为Json,并且确认机制：

```java
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;

/**
 * @author: kaiyi
 * @create: 2020-09-11 14:21
 */

@Configuration
public class MyRabbitConfig {

  @Autowired
  RabbitTemplate rabbitTemplate;

  @Primary
  @Bean
  public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
    RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
    this.rabbitTemplate = rabbitTemplate;
    rabbitTemplate.setMessageConverter(messageConverter());
    initRabbitTemplate();
    return rabbitTemplate;
  }

  /**
   * 使用JSON序列化机制，进行消息转换
   *
   * @return
   */
  @Bean
  public MessageConverter messageConverter() {
    return new Jackson2JsonMessageConverter();
  }

  /**
   * 定制RabbitTemplate
   * 1、服务收到消息就会回调
   * 1、spring.rabbitmq.publisher-confirms: true
   * 2、设置确认回调
   * 2、消息正确抵达队列就会进行回调
   * 1、spring.rabbitmq.publisher-returns: true
   * spring.rabbitmq.template.mandatory: true
   * 2、设置确认回调ReturnCallback
   * <p>
   * 3、消费端确认(保证每个消息都被正确消费，此时才可以broker删除这个消息)
   */
  @PostConstruct  //MyRabbitConfig对象创建完成以后，执行这个方法
  public void initRabbitTemplate() {

    /**
     * 1、只要消息抵达Broker就ack=true
     * correlationData：当前消息的唯一关联数据(这个是消息的唯一id)
     * ack：消息是否成功收到
     * cause：失败的原因
     */
    //设置确认回调
    rabbitTemplate.setConfirmCallback((correlationData,ack,cause) -> {
      System.out.println("confirm...correlationData["+correlationData+"]==>ack:["+ack+"]==>cause:["+cause+"]");
    });

     /**
         * 只要消息没有投递给指定的队列，就触发这个失败回调
         * message：投递失败的消息详细信息
         * replyCode：回复的状态码
         * replyText：回复的文本内容
         * exchange：当时这个消息发给哪个交换机
         * routingKey：当时这个消息用哪个路邮键
         */
        rabbitTemplate.setReturnCallback((message,replyCode,replyText,exchange,routingKey) -> {
            System.out.println("Fail Message["+message+"]==>replyCode["+replyCode+"]" +
                    "==>replyText["+replyText+"]==>exchange["+exchange+"]==>routingKey["+routingKey+"]");
        });
  }
}
```

#### 2) 消费端确认

可靠抵达-Ack消息确认机制说明：

**1、消费者获取到消息，成功处理，可以回复Ack给Broker**

- basic.ack 用于肯定确认；broker将移除此消息
- basic.nack 用于否定确认；可以指定broker是否丢弃此消息，可以批量
- basic.reject 用于否定确认；同上，但不能批量

**2、默认，消息被消费者收到，就会从broker的queue中移除**
**3、queue无消费者，消息依然会被存储，直到消费者消费**
**4、消费者收到消息，默认会自动ack。但是如果无法确定此消息是否被处理完成，或者成功处理。我们可以开启手动ack模式。**

- 消息处理成功，ack(),接受下一个消息，此消息broker就会移除。
- 消息处理失败，nack()/reject(), 重新发送给其他人处理，或者容错处理后ack。
- 消息一直没有调用ack/nack方法，broker认为此消息正在被处理，不会投递给别人，此时客户端断开，消息不会被broker移除，会投递给别人。

修改 application.properties 配置文件：

```properties
# 开启发送端消息抵达Broker确认
spring.rabbitmq.publisher-confirms=true
# 开启发送端消息抵达Queue确认
spring.rabbitmq.publisher-returns=true
# 只要消息抵达Queue，就会异步发送优先回调returnfirm
spring.rabbitmq.template.mandatory=true
# 手动ack消息，不使用默认的消费端确认
spring.rabbitmq.listener.simple.acknowledge-mode=manual
```

测试：

```java
 //@RabbitListener(queues = {"hello-java-queue"})
    @RabbitHandler
    public void revieveMessage(Message message,
        OrderReturnReasonEntity content, Channel channel) throws IOException {

        System.out.println("接收到消息..."+content);

        //拿到主体内容
        byte[] body = message.getBody();

        //拿到的消息头属性信息
        MessageProperties messageProperties = message.getMessageProperties();
        System.out.println("接受到的消息...内容" + message + "===内容：" + content);

        // Thread.sleep(3000);
        // Channel内按顺序自增
        long deliveryTag = message.getMessageProperties().getDeliveryTag();
        System.out.println("deliveryTag===>" + deliveryTag);

        try{
            // 签收货物,非批量模式
            channel.basicAck(deliveryTag, false);
        }catch (Exception e){
            // 网络中断（突然）
        }

    }
```

### 五、RabbitMQ-死信(延時)队列

#### 1)RabbitMQ 延时队列

RabbitMQ延时队列实现定时任务。

**场景：**
比如未付款订单，超过一定时间后，系统自动取消订单并释放占有的库存。

**常用解决方案：**
spring的schedule定时任务轮训数据库

**缺点：**
消耗系统内存，增加了数据库的压力、存在较大的时间误差

**解决：**
Rabbit的消息 TTL 和私信Exchange结合。

**消息的TTL**

消息的TTL(Time To Live)就是消息的存活时间，单位是毫秒。。
RabbitMQ 可以对**队列**和**消息**分别设置TTL。

- 对队列设置就是队列没有消费者连着的保留时间，`也可以对每一个单独的消息做单独的设置。超过了这个时间，我们认为这个消息就是死了，称之为死信`。
- 如果队列设置了，消息也设置了，那么`会取小`的。所以一个消息如果被路由到不同的队列中，这个消息死亡的时间有可能不一样（不同的队列设置）。这里单讲单个消息的TTL，因为它才是实现延迟任务的关键。可以通过`设置消息的expiration 字段或者 x-message-ttl 属性来设置时间`，两者是一样的效果。

注意：延时消息放入到队列中，没有被任何消费者监听，如果监听就拿到了，也就被消费了，队列里边的消息只要一过设置的过期时间，就成了死信队列，服务器就会丢弃。

那么，如何设置这个TTL值呢？有两种方式，第一种是在创建队列的时候设置队列的“x-message-ttl”属性，如下：

```java
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-message-ttl", 6000);
channel.queueDeclare(queueName, durable, exclusive, autoDelete, args);
```

这样所有被投递到该队列的消息都最多不会存活超过6s。

另一种方式便是针对每条消息设置TTL，代码如下：

```java
AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
builder.expiration("6000");
AMQP.BasicProperties properties = builder.build();
channel.basicPublish(exchangeName, routingKey, mandatory, properties, "msg body".getBytes());

```

这样这条消息的过期时间也被设置成了6s。

但这两种方式是有区别的，如果设置了队列的TTL属性，那么一旦消息过期，就会被队列丢弃，而第二种方式，消息即使过期，也不一定会被马上丢弃，因为消息是否过期是在即将投递到消费者之前判定的，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间。

另外，还需要注意的一点是，如果不设置TTL，表示消息永远不会过期，如果将TTL设置为0，则表示除非此时可以直接投递该消息到消费者，否则该消息将会被丢弃。

**死信**

死信：Dead Letter Exchange(DLX)

一个消息在满足如下条件，会进**死信路由**，记住这里是路由而不是队列，一个路由可以对应很多队列。

- 一个消息被Consumer拒收了，并且reject方法的参数里 requeue 是false。也就是说不会被放在队列里，被其他消费者使用。(basic.reject/basic.nack) requeue=false
- 上面的消息的TTL到了，消息过期了。
- 队列的长度限制满了。排在前面的消息会被丢弃或者扔到死信路由。

Dead Letter Exchange 其实就是一种普通的 exchange，和创建其他exchange一样。只是在某一个设置Dead Letter Exchange 的队列中有信息过期了，会自动触发消息的转发，发送到 Dead Letter Exhange中去。

我们既可以控制消息在一段时间后变成死信，又可以控制变成死信的消息被路由到某一个指定的交换机，结合二者，其实就可以实现一个延时队列。

![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/aXIENTeuyN.png)

#### 2)实战

**延时关单**

场景：用户下单，过了30分钟没有支付，系统会默认关闭该订单，以前可以用定时任务做，现在使用延时队列。

![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/A80ZWU7pxI.png)

**规范设计**

- 设计建议规范（基于事件模型的交换机设计）：
  1、交换机命名：业务+exchange;交换机为Topic
  2、路由键：事件.需要感知的业务（可以不写）
  3、队列命名：事件+想要监听服务名+queue
  4、绑定关系：事件.感知的业务(#)

- 整体业务设计：

![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/zpAUnDy6xL.png)

- 按照上边的规范设计，对关单业务进行升级设计：

  ![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/cjxhYkqf3V.png)

上图说明：交换机 `order-event-exchange` 绑定了一个延时队列`order.delay.queue`，路由key是 `order.create.order`, 当创建了一个订单时，会发消息到该延时队列，等到TTL过期，变为死信，会自动触发消息的转发，发送到 Dead Letter Exhange（order-event-exchange） 中去,注意死信路由是 `order.release.order`，然后exchange根据路由key `order.release.order`转发消息到 `order.release.order.queue`队列，客户端监听该队列获取消息。

根据上图的业务设计分析，需要创建两个队列，一个交换机，和两个绑定。

```java
import com.atguigu.gulimall.order.entity.OrderEntity;
import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.Exchange;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.io.IOException;
import java.util.HashMap;

/**
 * @author: kaiyi
 * @create: 2020-09-16 13:53
 */
@Configuration
public class MyMQConfig {

  /* 容器中的Queue、Exchange、Binding 会自动创建（在RabbitMQ）不存在的情况下 */

  /**
   * 客户端监听队列（测试）
   * @param orderEntity
   * @param channel
   * @param message
   * @throws IOException
   */
  @RabbitListener(queues = "order.release.order.queue")
  public void listener(OrderEntity orderEntity, Channel channel, Message message) throws IOException {

    System.out.println("收到过期的订单信息：准备关闭订单" + orderEntity.getOrderSn());
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);

  }

  /**
   * 死信队列
   *
   * @return
   */
  @Bean
  public Queue orderDelayQueue(){

     /*
            Queue(String name,  队列名字
            boolean durable,  是否持久化
            boolean exclusive,  是否排他
            boolean autoDelete, 是否自动删除
            Map<String, Object> arguments) 属性
         */
    HashMap<String, Object> arguments = new HashMap<>();
    arguments.put("x-dead-letter-exchange", "order-event-exchange");
    arguments.put("x-dead-letter-routing-key", "order.release.order");
    arguments.put("x-message-ttl", 60000); // 消息过期时间 1分钟

    Queue queue = new Queue("order.delay.queue", true, false, false, arguments);
    return queue;
  }

  /**
   * 普通队列
   *
   * @return
   */
  @Bean
  public Queue orderReleaseQueue(){

    Queue queue = new Queue("order.release.order.queue", true, false, false);
    return queue;
  }

  /**
   * TopicExchange
   *
   * @return
   */
  @Bean
  public Exchange orderEventExchange(){
    /*
     *   String name,
     *   boolean durable,
     *   boolean autoDelete,
     *   Map<String, Object> arguments
     * */

    return new TopicExchange("order-event-exchange", true, false);
  }

  @Bean
  public Binding orderCreateBinding() {
    /*
     * String destination, 目的地（队列名或者交换机名字）
     * DestinationType destinationType, 目的地类型（Queue、Exhcange）
     * String exchange,
     * String routingKey,
     * Map<String, Object> arguments
     * */
    return new Binding("order.delay.queue",
        Binding.DestinationType.QUEUE,
        "order-event-exchange",
        "order.create.order",  // 路由key一般为事件名
        null);
  }

  @Bean
  public Binding orderReleaseBinding() {

    return new Binding("order.release.order.queue",
        Binding.DestinationType.QUEUE,
        "order-event-exchange",
        "order.release.order",
        null);
  }

}
```

然后在控制器创建测试消息：

```java
@Controller
public class HelloController {

  @Autowired
  private RabbitTemplate rabbitTemplate;

  @ResponseBody
  @GetMapping(value = "/test/createOrder")
  public String createOrderTest() {

    //订单下单成功
    OrderEntity orderEntity = new OrderEntity();
    orderEntity.setOrderSn(UUID.randomUUID().toString());
    orderEntity.setModifyTime(new Date());

    //给MQ发送消息
    rabbitTemplate.convertAndSend("order-event-exchange","order.create.order",orderEntity);

    return "ok";
  }
}
```

发送消息，然后去RMQ管理界面可以看到创建的消息已经成功了。

交换机：

![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/0JNNSCHgmC.png)

交换机绑定的队列（路由key）:

![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/KpOKbBHl3D.png)

队列：

![file](https://raw.githubusercontent.com/lilu188011/img-repo/master/f63bnMtqq0.png)

控制器输出的监控信息：

```text
收到过期的订单信息：准备关闭订单321c3329-d57a-4613-a4ff-331066d4105a
收到过期的订单信息：准备关闭订单44fcf65f-1e7a-40c6-8336-a6c60362920b

```

