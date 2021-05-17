# Kafaka的前世今生

## 点到点的Data Pipeline

前端需要用到数据的部门，后端提供数据的部门，两个部门碰头，统一格式，然后实时的从数据的生产方到数据消费方，n个生产方，m个消费方，m * n个点到点的数据通道；期望建立一个中央式的数据通道，生产方只和中央交互，消费方只和中央交互。main-memory尝试过，但是解决不了linkedin的方差大问题
## 结构
8 
生产方append到日志的最末端，cosumer按照offset的方式顺序读取，存在磁盘里，顺序的磁盘读写几乎媲美内存的随机访问，持久化的磁盘上，可以抵抗波峰波谷，波峰来临时，consumer离prducer比较远，波峰结束后能赶上就行（流数据的持久化，可以重访性），main-memory只是短暂。
9
所有数据按照topic的方式来组织，topic也可分片，所有数据存在brokers，不同的topic和partition可以存在不同的brokers上，pro发布数据时，可以发布到topic上，然后持久化到partition上；consumer可以订阅多个topic，如果数据发布到consumer订阅的partition时，就会同步发布给订阅了他们的consumer；数据量增加时，我们就可以发布到新的partition，把这些partition建到新的机器上，有很好的扩展性。
10
Pub-Sub Messaging
Pub就是user data的tracking system，比如用户点击了推荐，都会通过kafka倒入hadoop等，日志报错之类的也会通过kafka同步到后端；
12
加入了数据备份
13
早期十tracking data和log data，不需要数据的完整性，丢了几个没关系；最好是一条数据都不丢（mission critical），如果实时运行的时候，一个服务器炸了，数据就永远丢了，
14
0.8对topic part进行备份，如果一个broker有3个partition，那就把3个partition发到3个broker上
15
如何做数据同步，一个新的数据发到集群上，如何保证replication leader发布到follower上
16
19
假定broker1是整个cluster的controller，如果bro1炸了，bro2和3就会通过zookeeper得到这个消息，然后二者会选举出新的controller，先更新ISR，先踢出1，然后告诉3，2成为新的leader，然后3就行会从2备份新的数据，3根据2补数据，补充完成之后再加入ISR
30
2012kafka可靠性得到了保障，kafka开始推广

0.9。0
加入了quota和security，多租户的kaffa，多个应用进行生产和消费，如果一个客户端发布了很多地DOS攻击，会使得kafka效率很低，每秒发上千请求，kafka就没有更多资源来处理别的请求
36
多租户的系统，会给每个用户分配网络的吞吐量，如果配额超出，就会对你进行延迟，定义每秒发布或者消费的量
37
认证、授权、加密，敏感的topic只有特定用户才能读写删除
38
两个数据中心，1中append log，然后通过异步的mirroring备份到另外的数据中心里