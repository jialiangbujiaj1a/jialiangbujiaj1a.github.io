# RabbitMQ笔记

## 1、基础概念

**ConnectionFactory、Connection、Channel：**ConnectionFactory、Connection、Channel都是RabbitMQ对外提供的API中最基本的对象。Connection是RabbitMQ的socket链接，它封装了socket协议相关部分逻辑。ConnectionFactory为Connection的制造工厂。 Channel是在 connection 内部建立的逻辑连接（每访问一次mq建立一个连接，开销巨大），如果应用程序支持多线程，通常每个thread创建单独的 channel 进行通讯，大部分的业务操作是在Channel这个接口中完成的，包括定义Queue、定义Exchange、绑定Queue与Exchange、发布消息等。

**Message acknowledgment：**消息回执

**Message durability：**消息持久化

**Broker:**接收和分发消息的应用，`RabbitMQ Server就是 Message Broker。`

**Virtual host:**把AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念

**Queue:**队列对象，用于存储消息

**Exchange：**message 到达 broker 的第一站，根据分发规则，`匹配查询表中的 routing key，`分发消息到queue 中去。常用的类型有：

- `direct (point-to-point)`把消息路由到那些binding key与routing key完全匹配的Queue中
- `topic (publish-subscribe)`把消息路由到那些binding key与routing key通配符匹配的Queue中，*用于匹配一个单词，#用于匹配多个单词（可以0个）
- `fanout (multicast)`把所有发送到该Exchange的消息路由到所有与它绑定的Queue中

**Routing key:**生产者在将消息发送给Exchange的时候，一般会指定一个routing key，来指定这个消息的路由规则，而这个routing key需要与Exchange Type及binding key联合使用才能最终生效。 RabbitMQ为routing key设定的长度限制为255 bytes。

**Binding:**exchange 和 queue 之间的虚拟连接，binding 中可以包含 routing key。Binding 信息被保存到 exchange 中的查询表中，用于 message 的分发依据

**Binding key:**在绑定（Binding）Exchange与Queue的同时，一般会指定一个binding key；消费者将消息发送给Exchange时，一般会指定一个routing key；当binding key与routing key相匹配时，消息将会被路由到对应的Queue中。

## 2、运转流程

- 生产者发送消息
  1. 生产者创建连接（Connection），开启一个信道（Channel），连接到RabbitMQ Broker；
  2. 声明队列并设置属性；如是否排它，是否持久化，是否自动删除；
  3. 将路由键（空字符串）与队列绑定起来；
  4. 发送消息至RabbitMQ Broker；
  5. 关闭信道；
  6. 关闭连接；
- 消费者接收消息
  1. 消费者创建连接（Connection），开启一个信道（Channel），连接到RabbitMQ Broker
  2. 向Broker 请求消费相应队列中的消息，设置相应的回调函数；
  3. 等待Broker回应闭关投递响应队列中的消息，消费者接收消息；
  4. 确认（ack，自动确认）接收到的消息；
  5. RabbitMQ从队列中删除相应已经被确认的消息；
  6. 关闭信道；
  7. 关闭连接；

## 3、TTL（Time-To-Live 过期时间）

RabbitMQ可以对消息和队列设置TTL。第一种方法是通过队列属性设置(channel.queueDeclare方法中加入x-message-ttl参数，单位为ms.)，队列中所有消息都有相同的过期时间。第二种方法是对消息进行单独设置（channel.basicPublish方法中加入expiration的属性参数，单位为ms.），每条消息TTL可以不同，每条消息是否过期是在该消息在队列头部时（消费时）才会判定。如果上述两种方法同时使用，则消息的过期时间以两者之间TTL较小的那个数值为准。消息在队列的生存时间一旦超过设置的TTL值，就称为dead message， 消费者将无法再收到该消息。

## 4、DLX(死信队列)

DLX, Dead-Letter-Exchange。利用DLX, 当消息在一个队列中变成死信（dead message）之后，它能被重新publish到另一个Exchange，这个Exchange就是DLX。消息变成死信一向有一下几种情况：

- 队列消息长度到达限制；

- 消费者拒接消费消息，basicNack/basicReject,并且不把消息重新放入原目标队列,requeue=false；

- 原队列存在消息过期设置，消息到达超时时间未被消费；

DLX也是一个正常的Exchange，和一般的Exchange没有区别，它能在任何的队列上被指定，实际上就是设置某个队列的属性，当这个队列中有死信时，RabbitMQ就会自动的将这个消息重新发布到设置的Exchange上去，进而被路由到另一个队列，可以监听这个队列中消息做相应的处理，这个特性可以弥补RabbitMQ 3.0以前支持的immediate参数（可以参考[RabbitMQ之mandatory和immediate](http://blog.csdn.net/u013256816/article/details/54914525)）的功能。

核心代码实现：通过在queueDeclare方法中加入“x-dead-letter-exchange”实现。你也可以为这个DLX指定routing key，如果没有特殊指定，则使用原队列的routing key。

## 5、延迟队列

TTL+DLX

## 6、消息可靠性

### 6.1 服务端确认机制-事务机制

通过一个 channel.txSelect()的方法把信道设置成事务模式，然后就可以发布消息给 RabbitMQ 了，如果 channel.txCommit();的方法调用成功，就说明事务提交成功，则消息一定到达了 RabbitMQ 中。如果在事务提交执行之前由于 RabbitMQ 异常崩溃或者其他原因抛出异常，这个时候我们便可以将其捕获，进而通过执行 channel.txRollback()方法来实现事务回滚。

**缺点**：在事务模式里面，只有收到了服务端的 Commit-OK 的指令，才能提交成功。所以可以解决生产者和服务端确认的问题。但是**事务模式有一个缺点，它是阻塞的**，

###  6.2 **服务端确认机制-Confirm(确认)模式**：

**1.普通确认模式**：在生产者这边通过调用 channel.confirmSelect()方法将信道设置为 Confirm 模式，然后发送消息。一旦消息被投递到所有匹配的队列之后，RabbitMQ 就会发送一个确认(Basic.Ack)给生产者，也就是调用 channel.waitForConfirms()返回 true，这样生产者就知道消息被服务端接收了！----》方式消息效率还不是太高

**2.批量确认模式**：批量确认，就是在开启 Confirm 模式后，先发送一批消息。只要channel.waitForConfirmsOrDie();方法没有抛出异常，就代表消息都被服务端接收了。

批量确认的方式比单条确认的方式效率要高，但是也有两个问题，第一个就是批量的数量的确定。对于不同的业务，到底发送多少条消息确认一次?数量太少，效率提升不上去。数量多的话，又会带来另一个问题，比如我们发 1000 条消息才确认一次，如果前面 999 条消息都被服务端接收了，如果第 1000 条消息被拒绝了，那么前面所有的消息都要重发。

**3.异步确认模式**：一边发送一边确认！异步确认模式需要添加一个 ConfirmListener，并且用一个 SortedSet 来维护没有被确认的消息。Confirm 模式是在 Channel 上开启的，Ø使用rabbitTemplate.setConfirmCallback设置回调函数。当消息发送到exchange后回调confirm方法。在方法中判断ack，如果为true，则发送成功，如果为false，则发送失败，需要处理。

### 6.3 消息从交换机路由到队列

rebbitmqTemplate设置mandatory为true

使用rabbitTemplate.setReturnCallback设置退回函数，当消息从exchange路由到queue失败后，如果设置了rabbitTemplate.setMandatory(true)参数，则会将消息退回给producer。并执行回调函数returnedMessage。

### 6.4 持久化

1）队列持久化 2）交换机持久化 3)消息持久化 4）集群

### 6.5 消费者ACK

ack指Acknowledge，确认。 表示消费端收到消息后的确认方式。

有三种确认方式：

•自动确认：acknowledge="none"

•手动确认：acknowledge="manual"

•根据异常情况确认：acknowledge="auto"，（这种方式使用麻烦，不作讲解）



其中自动确认是指，当消息一旦被Consumer接收到，则自动确认收到，并将相应 message 从 RabbitMQ 的消息缓存中移除。但是在实际业务处理中，很可能消息接收到，业务处理出现异常，那么该消息就会丢失。如果设置了手动确认方式，则需要在业务处理成功后，调用channel.basicAck()，手动签收，如果出现异常，则调用channel.basicNack()方法，让其自动重新发送消息。

## 7、消息幂等性

**乐观锁**

## 8、消息顺序性

方案一，拆分多个 queue，每个 queue 一个 consumer

方案二，或者就一个 queue 但是对应一个 consumer，然后这个 consumer 内部用内存队列做排队，然后分发给底层不同的 worker 来处理。