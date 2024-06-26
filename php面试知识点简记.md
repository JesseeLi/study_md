####  一、面向对象三大特性

- 封装  封装成抽象的类。使用者只要知道如何使用就行，具体实现不需要知道

- 继承  可以继承父类的一些功能
- 多态  不同类收到相同的消息，返回不同的结果

魔术方法

- __set
- __get
- __call
- __callstatic
- __construct
- __destruct
- __invoke       引入这个之后，实例化对象之后可以当做函数执行调用

#### 二、抽象类和接口
- abstract   抽象类，只要类内存在abstract，就是抽象类，类前就得加abstract， 抽象方法不能实现具体功能和接口一样

- interface  接口 通过implements继承，必须实现接口内所有方法，还能定义常量 

####  三、mysql
1. 存储引擎
      MyiSAM  查询快，写入慢，不支持事务
      InnoDB  支持事务

2. 事务特性
      原子性  一个事务内要么成功要么失败
      一致性  事务操作前后，满足数据完整性约束，数据库保持一致性状态
      隔离性  多个事务执行时，不会相互干扰
      持久性  提交后，永久的保存在硬盘内

3. InnoDB事务隔离级别
- 读未提交
	- 一个事务可以读取另一个未提交事务的数据
	- `脏读`：事务A 修改完数据，还未提交事务，这个时候 事务B 读取了 事务A 修改后的数据的值，正好这个时候 事务A 回滚了。
- 读提交
  - 一个事务只能读取到其他事务已经提交的数据
  - 事务A 修改数据，还未提交，事务B 只能等待 事务A 提交之后才能读取。如果 事务A 提交，那么 事务B 读取的是 事务A 提交后的值，如果 事务A 回滚，则 事务B 读取的是 事务A 操作之前的值
  - `不可重复读`：事务A 内多次读取数据时，两次读取的结果不一致，因为在两次读取之间，另一个已提交的事务正好修改了这个数据

- 可重复读
  - 确保同一事务中多次读取的结果是一致的，不会看到其他事务在这个事务执行期间做出的修改。
  - `幻读`：就是想查询有没有这条记录，没有就添加。  事务A 开始时， 多次读取这个记录，发现都是没有查到（可重复读级别，每次查询数据都是一样的），但是另一个 事务B 这个时候已经添加完了这条数据。这个时候提交 事务A 就失败了

- 可串行化
  - 处理一个事务的时候，别的事务都得等待这个事务完成。读的时候加 共享锁，写的时候加 排它锁


4. 优化MySQL
      sql语句优化，减少请求的数据量
      添加索引、索引的优化
      数据库表结构的优化
      配置优化
      硬件优化

5. 索引
      普通索引	create index `index_name` on table_name(key1)
      唯一索引	create unique index `index_name` on table_name(key1)
      全文索引	create fulltext index `index_name` on table_name(key1)
      主键索引
      联合索引	create index `index_name` on table_name(key1,key2)

6. 索引失效

      `对索引使用左或者左右模糊匹配` like %xx 或者 like %xx%, 都会造成索引失效

      `对索引使用函数`（mysql 8.0 添加了函数索引，可以实现【和普通索引一样，只不过索引的字段是 使用函数后的字段 alter table t_user add key idx_name_length ((length(name)))】）

      `对索引进行表达式计算`

      `对索引进行隐式类型转换`

      `联合索引非最左匹配`。（创建了一个 (a, b, c) 联合索引，where a = 1; where a=1 and b=2 and c=3；where a=1 and b=2；这些都能生效。非 a 开头就都失败了。 where a = 1 and c = 3  索引截断， ）

      `where 子句中的 or, 两边都得是 索引字段`

7. in 和 exists 的区别

- 1.select * from A where id in(select id from B) 2.select a.* from A a where exists(select 1 from B b where a.id=b.id)

- in 语句是把 外表 和 内表 做 join 链接，而 exists 语句是对 外表 做 nest loop 循环，每次循环再对 内表 进行查询
- 外层查询表小于子查询表（where），则用 `exists`，外层查询表大于子查询表（where），则用 `in` ，如果外层和子查询表差不多，则那个都行。
- `in` 查询的子条件返回结果必须只有一个字段。`exists` 的条件就像一个 `bool` 条件，当能返回结果集则为 `true`，不能返回结果集则为 `false`。
- `not in` 内外表都进行全表扫描，没有用到索引。而 `not exists` 的子查询依然能用到表上的索引。


#### 四、Redis

   数据类型
   string  字符串
   hash
   set  集合
   zset  有序集合
   list  链表

   bitmap
   hyperLogLog
   Geo  地理位置
   pub/sub  订阅

   redis事务
   multi  exec 

   redis持久化
   aof日志   没执行一条写入操作，就把命令追加到日志内
   rdb   将某一刻的数据以二进制写入到磁盘内

   分布式锁
   set   set user 1 nx px 1000
   setnx expire
   lua脚本

   缓存雪崩  某一时间内，一大批数据同时过期    解决：随机过期时间、设置不过期
   缓存穿透  既不在缓存内，又不在数据库内。    解决：非法请求限制。  设置空值或默认值。  使用布隆过滤器
   缓存击穿  热点数据过期，查询都集中到数据库      解决：不设置过期时间，异步后台更新数据。  对请求加锁

   缓存更新策略及数据一致性问题
   先更新数据库 再删除缓存   （存在缓存写入比数据库写入快，导致数据不一致问题，大部分可以用这种）

#### 五、cron定时任务

   分 时 日期 月份 星期 [年份]

   每天凌晨3点执行 0 0 3 * * ?

   

   #### 六、其他
   冒泡排序   双重循环。主要进行相邻的两个进行比较，然后调换数组位置
   快速排序   选择一个中间值，进行比较，小的在左，大的在右。然后递归左右数组，最后合并左中右


   七层网络协议  应表会传 物联网  即 应用层 表示层 会话层 传输层 网络层 数据链路层 物理层
   五层网络协议  应用层  传输层  网络层  数据链路层  物理层

   TCP和UDP协议都是在 传输层
   Http、WebStock、RPC、FTP、SMTP等 都是在 应用层

   RPC协议 远程过程调用  主要用于C/S架构  目前一般用在内部网络里，可以实现在本机调用另外一台机器的方法
   http协议  超文本传输协议  主要用于B/S架构
   WebStock  同一时间里，双方都可以主动向对方发送数据。全双工。  服务器可以主动向客户端发送消息                                    
             轮训和长轮训  轮训，js使用定时器每秒向后端请求。   长轮训，将 HTTP 请求将超时设置的很大，比如 30 秒，在这 30 秒内只要服务器收到了扫码请求，就立马返回给客户端网页。如果超时，那就立马发起下一次请求。


   TCP 是面向连接的、可靠的、基于字节流的传输层通信协议。
   面向连接 一定是「一对一」才能连接。不是udp一对多
   可靠的   网络再差也能到达接收端
   字节流   用户消息通过 TCP 协议传输时，消息可能会被操作系统「分组」成多个的 TCP 报文，如果接收方的程序如果不知道「消息的边界」，是无法读出一个有效的用户消息的。
           并且 TCP 报文是「有序的」，当「前一个」TCP 报文没有收到的时候，即使它先收到了后面的 TCP 报文，那么也不能扔给应用层去处理，同时对「重复」的 TCP 报文会自动丢弃。

##### TCP三次握手四次挥手
- 三次握手
  1. 客户端和服务端都处于关闭状态。 服务端主动监听某个端口，处于listen状态
  2. 第一次 客户端随机初始化序号（client_isn）,放在 TCP 首部的【序号】字段内，同时把 SYN 标志位置为 1 ，表示 SYN 报文。 把 SYN 报文发送给服务端，不包含应用层数据，之后客户端处于 【SYN-SENT】 状态。
  3. 第二次 服务端收到 SYN 报文后，服务端随机初始化序号（server_isn），放在 TCP 首部的【序号】字段内，其次把 TCP 首部的【确认应答号】填入（client_isn + 1）,接着把 SYN 和 ACK 标志位置为 1 。最后发送给客户端，不包含应用层数据，之后客户端处于 【SYN-RCVD】 状态
  4. 第三次 客户端收到后，还要向服务端发送 应答报文，首先把 应答报文 TCP 首部 ACK 标志位置为 1 ，其次 【确认应答号】填入（server-isn + 1）,发送给服务端。这次可以携带要发送的数据，之后客户端处于 【ESTABLISHED】 状态。
  5. 服务端收到应答报文后，也处于 【ESTABLISHED】 状态。
- 四次挥手
  1. 客户端打算关闭连接，此时会发送一个 TCP 首部 `FIN` 标志位被置为 `1` 的报文，也即 `FIN` 报文，之后客户端进入 `FIN_WAIT_1` 状态。
  1. 服务端收到该报文后，就向客户端发送 `ACK` 应答报文，接着服务端进入 `CLOSE_WAIT` 状态。
  1. 客户端收到服务端的 `ACK` 应答报文后，之后进入 `FIN_WAIT_2` 状态。
  1. 等待服务端处理完数据后，也向客户端发送 `FIN` 报文，之后服务端进入 `LAST_ACK` 状态。
  1. 客户端收到服务端的 `FIN` 报文后，回一个 `ACK` 应答报文，之后进入 `TIME_WAIT` 状态。
  1. 服务端收到了 `ACK` 应答报文后，就进入了 `CLOSE` 状态，至此服务端已经完成连接的关闭。
  1. 客户端在经过 `2MSL` 一段时间后，自动进入 `CLOSE` 状态，至此客户端也完成连接的关闭。




#### 七、中间件

   RabbitMQ

   一、五种工作模式
   1. 简单模式              生产者 -> 队列 -> 消费者(一个消费者)
   2. work工作模式          生产者 -> 队列 -> 多个消费者来消费（多个消费者，获得每个消息不一样）
   3. 订阅模式              生产者 -> 交换机 -> 消费者（获得的消息是一模一样的）                                (不传队列名，由mq自动生成)无差别的把消息发送给消费者
   4. 路由模式              生产者 -> 交换机 -> 根据发送来的路由键发送到对应的队列内                            根据路由键进行筛选，（精确匹配）匹配到对应的队列内
   5. 主题模式              生产者 -> 交换机（topic） ->                                                   根据路由键进行筛选，（模糊匹配，有通配符 * ，#）匹配到对应的队列内

   二、消息的持久化和ack应答 及高级特性
   1. 持久化包括  交换机、队列、消息 的持久化，一般是 durable 参数设置。  消息持久化的前提是队列持久化

   2. ACK 两种确认方式 
           自动确认        消费者收到消息后自动ack ，然后再处理业务，加入业务出现问题，消息也会被确认
           手动确认        消费者收到消息后，消息状态为 unack。 有业务逻辑指定 ack 位置，如果没有手动 ack ,则消息不会减少

   3. 生产者确保消息发送百分百 事务，confirm 机制
      事务
      $channel->tx_select()           开启事务
      $channel->tx_commit()           提交事务
      $channel->tx_rollback()         回滚事务
      confirm机制
      $channel->confirm_select()              开启确认模式
      $channel->wait_for_pending_acks()       添加监听
      $channel->set_ack_handler()             推送成功事件
      $channel->set_nack_handler()            推送失败事件

   三、将 交换机、队列、消息 都持久化后。消息就不会丢失吗
   否定的，还会丢失

   1. 从消费端 consumer 。 如果 ack 为 自动应答时，消息没来及处理就 crash（崩溃）掉了，数据就丢失了。      解决方案：手动 ack
   2. 从生产端 producer 。 如果放入队列的时候 crash 了，消息就丢失了。     解决方案：引入事务机制 或者 confirm机制 来确保消息已经发送到mq了
   3. 消息正确存入 mq 后，持久化还需要一段时间保存到磁盘内，如果此时 mq crash 了，消息也会丢失。           解决方案：镜像队列 ？

   四、死信队列
   DLX (Dead-Letter_Exchange) , 也可以称为 死信交换机 ，当一个队列内的消息变为死信之后，会被重新发送到另一个交换机，这个交换机就是 DLX ,而 绑定DLX 的队列就是 死信队列

   消费被拒绝
   消息过期 ！！！
   队列到达最大长度

   使用时只要定义队列的时候设置 x-dead-letter-exchange 指定交换机就行

   五、延时队列
   当消息发送后，不想让消费者立即拿到消息，而是等待特定时间后才能拿到消息。     延时功能可以通过 设置过期时间 + 死信队列 来实现
   设置 x-dead-letter-exchange（死信交换机）、x-dead-letter-routing-key（路由key）、x-message-ttl（消息的存活时间）。当队列内的消息存活时间过期后，转存入设置好的 死信交换机分配的死信队列内

   六、集群
   RabbitMq 会始终记录一下四种内部元数据
   1. 队列元数据 - 队列名称 和他们的属性（是否持久化、是否自动删除）
   2. 交换机元数据 - 交换机类型、名称、属性
   3. 绑定元数据 - 一张简单的表格展示如何将消息路由到队列
   4. vhost元数据 - 为 vhost 内的 队列、交换机、和绑定 提供命名空间和安全属性

   3个模式  主备模式、镜像模式、异地多活

   负载均衡需要插件。 HAProxy

   集群搭建        
   1. 先在服务器上配置host，例如 192.168.1.1 node1、192.168.1.2 node2、192.168.1.3 node3    
   2. 拷贝 node1 节点上的 erlang.cookie 保证三个节点内容相同
   3. 删除 Rabbitmq 下的 mnesia, 使用 rabbitmq-server -detached 启动节点
           rabbitmqctl stop_app
           rabbitmqctl reset
           rabbitmqctl join_cluster rabbit@'hostname -s'
           rabbitmqctl start_app
           rabbitmqctl cluster_stauts

   

   ElasticSearch

   基于 Lucene 构建的分布式搜索引擎，提供 RESTful API ，实时的、文档（Document）导向的，每个文档都是 JSON 格式
   Lucene底层结构                索引Index -> 段Segment   文档Document -> 域Field -> 分词
   FST 有限状态机，表示有限个状态以及在这些状态之间转移和动作等行为的数学模型
   倒排索引        单词词典和倒排文件 组成

   中文分词插件 ik分词器 采用正向匹配算法  ik_smart 最粗粒度拆分   ik_max_word 最细粒度拆分
   ik 热更新词库

   

#### 前端

   一、SPA 单一页面应用

   优点：1. 前后端分离   2. 避免不必要的跳转和渲染，提高了用户体验
   缺点：1. 初次加载耗时长 2. 浏览器的前进后退不能使用，要自己建立堆栈进行页面切换 3. seo难度大


   二、computed 和 watch

   computed  计算属性，其结果具有缓存属性，只有它依赖的值发生变化，才会在下次获得computed的值的时候重新计算computed。一般在进行数值计算并且依赖他的结果时使用。
   watch  更多是“监视作用”，一般在异步执行或者开销较大时使用。开销较大的操作指的是当数据发生变化时执行的复杂的业务逻辑。

   三、 生命周期

   1. beforeCreate
   2. created                          在这里发送axios请求
   3. beforeMounted
   4. mounted                          在这里操作DOM
   5. beforeUpdate
   6. update
   7. activated
   8. deactivated  
   9. beforeDestroy                    组件销毁前调用。
   10. destroyed                       组件销毁后调用。

   四、 组件间通信
   1. props   $emit                 子组件props接受数据  $emit 子传父，父组件methods接收
   2. ref
   3. vuex
   4. eventBus 发布订阅实现