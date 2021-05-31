# Apache Kafaka

## 点到点的Data Pipeline

要了解Kafka，我们需要了解Kafka诞生的背景，Linkedin是一个数据驱动的公司（or 产品）。在Kafka诞生之前的开发流程是这样的，举例来说，前端需要用到数据的部门，和后端提供数据的部门，两个部门碰头开会，统一数据格式格式，然后拉通点到点的数据流，数据实时的从数据的生产方到数据消费方，但是问题来了，假如有 $n$ 个生产方，$m$ 个消费方，那么就有了 $$m * n$$ 个点到点的数据通道，对资源的要求非常高；Linkedin期望建立一个中央式的数据通道，生产方只和中央交互，消费方也只和中央交互。
## 概念

###  生产与消费

**这里需要再听一下**  
Kafka中的消息（Messages）类似于一个队列，生产方append到日志的最末端，consumer按照自己的offset顺序读取。日志都是持久化在磁盘里的，存在磁盘的原因是，第一，顺序的磁盘读写可以几乎媲美内存的随机访问，第二，持久化到磁盘上，可以抵抗波峰波谷，波峰来临时，consumer离prducer比较远，波峰结束后能赶上就行（流数据的持久化，可以重访性），main-memory只是短暂。
9  
在Kafka中，数据按照topic的方式来组织，topic也可以分片（Partitions），所有数据存在Brokers，不同的topic和partition可以存在不同的brokers上。Producer发布数据时，发布到topic上，然后持久化到属于这个Tobic的partition上；consumer可以订阅多个topic，如果数据发布到consumer订阅的topic时，就会同步发布给订阅了他们的consumer；这里还有一个好处是，数据量增加时，数据就可以发布到新的partition，把这些partition建到新的机器上，有很好的扩展性。
10

## Pub-Sub Messaging

Pub就是user data的tracking system，比如用户点击了推荐，都会通过kafka倒入hadoop等，日志报错之类的也会通过kafka同步到后端；  

## Replicas and Layout

Kafka0.8版本加入了数据备份，早期的Kafka是没有备份机制的，在当时的业务场景下，tracking data和log data不需要数据的完整性，即使丢失了一些数据没关系；但是对于mission critical的任务来说（比如银行流水），是一条数据都不丢的，在当时的情况下，如果业务服务在实时运行的时候，一个服务器炸了，数据就永远丢了，Kafka 0.8对topic part进行备份，如果一个broker有3个partition，那就把3个partition发到3个broker上。具体的过程是，假设broker1是整个cluster的controller，如果broker1炸了，broker2和broker3就会通过zookeeper得到这个消息，然后二者会选举出新的controller，先更新ISR，先踢出broker1，然后告诉broker3，broker2成为新的leader，然后broker3就行会从broker2备份新的数据，broker3根据broker2补数据，补充完成之后再加入ISR。这样的话，从2012年开始，kafka的可靠性得到了保障，开始大面积推广。

## quota和security

Kafka0.9版本，加入了quota和security机制。Kafka是一个多租户的系统，即有多个应用进行生产和消费，假设一个客户端发布了很多的DOS攻击，会使得kafka效率很低，如果每秒发上千请求，kafka就没有更多资源来处理别的请求。基于上述背景，0.9.0版本把kafka打造为了多租户的系统，Kafka会给每个用户分配网络的吞吐量，定义每秒发布或者消费的量，如果配额超出，就会对你进行延迟，
37
安全机制指的是认证、授权、加密，敏感的topic只有特定用户才能读写或删除
38
两个数据中心，1中append log，然后通过异步的mirroring备份到另外的数据中心里

45

Exactly-once 有且仅有一次的计算保障，输入的数据在failure发生时也有且仅有计算一次