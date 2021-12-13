## 分布式唯一ID介绍

分布式系统全局唯一的 id 是所有系统都会遇到的场景，往往会被用在搜索，存储方面，用于作为唯一的标识或者排序，比如全局唯一的订单号，优惠券的券码等，如果出现两个相同的订单号，对于用户无疑将是一个巨大的bug。

在单体的系统中，生成唯一的 id 没有什么挑战，因为只有一台机器一个应用，直接使用单例加上一个原子操作自增即可。而在分布式系统中，不同的应用，不同的机房，不同的机器，要想生成的 ID 都是唯一的，确实需要下点功夫。

一句话总结：

> **分布式唯一ID是为了给数据进行唯一标识。**

### 分布式唯一ID的特征

分布式唯一ID的核心是唯一性，其他的都是附加属性，一般来说，一个优秀的全局唯一ID方案有以下的特点，仅供参考：

- 全局唯一：不可以重复，核心特点！
- 大致有序或者单调递增：自增的特性有利于搜索，排序，或者范围查询等
- 高性能：生成ID响应要快，延迟低
- 高可用：要是只能单机，挂了，全公司依赖全局唯一ID的服务，全部都不可用了，所以生成ID的服务必须高可用
- 方便使用：对接入者友好，能封装到开箱即用最好
- 信息安全：有些场景，如果连续，那么很容易被猜到，攻击也是有可能的，这得取舍。

## 分布式唯一ID的生成方案

### UUID直接生成

写过 Java 的朋友都知道，有时候我们写日志会用到一个类 UUID，会生成一个随机的ID，去作为当前用户请求记录的唯一识别码,只要用以下的代码：

```java
String uuid = UUID.randomUUID();
```

用法简单粗暴，UUID的全称其实是`Universally Unique IDentifier`,或者`GUID(Globally Unique IDentifier)`,它本质上是一个 128 位的二进制整数，通常我们会表示成为 32 个 16 进制数组成的字符串，几乎不会重复，2 的 128 次方，那是无比庞大的数字。

以下是百度百科说明：

> UUID由以下几部分的组合：
>
> （1）UUID的第一个部分与时间有关，如果你在生成一个UUID之后，过几秒又生成一个UUID，则第一个部分不同，其余相同。
>
> （2）时钟序列。
>
> （3）全局唯一的IEEE机器识别号，如果有网卡，从网卡MAC地址获得，没有网卡以其他方式获得。
>
> UUID的唯一缺陷在于生成的结果串会比较长。关于UUID这个标准使用最普遍的是微软的GUID(Globals Unique Identifiers)。在ColdFusion中可以用CreateUUID()函数很简单地生成UUID，其格式为：xxxxxxxx-xxxx- xxxx-xxxxxxxxxxxxxxxx(8-4-4-16)，其中每个 x 是 0-9 或 a-f 范围内的一个十六进制的数字。而标准的UUID格式为：xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx (8-4-4-4-12)，可以从cflib 下载CreateGUID() UDF进行转换。 [2] 
>
> （4）在 hibernate（Java orm框架）中， 采用 IP-JVM启动时间-当前时间右移32位-当前时间-内部计数（8-8-4-8-4）来组成UUID

要想重复，两台完全相同的虚拟机，开机时间一致，随机种子一致，同一时间生成uuid，才有极小的概率会重复，因此我们可认为，理论上会重复，实际不可能重复！！！

uuid优点：

- 性能好，效率高
- 不用网络请求，直接本地生成
- 不同的机器个干个的，不会重复

uuid 这么好，难不成是银弹？当然缺点也很突出：

- 没办法保证递增趋势，没法排序
- uuid太长了，存储占用空间大，特别落在数据库，对建立索引不友好
- 没有业务属性，这东西就是一串数字，没啥意义，或者说规律

当然也有人想要改进这家伙，比如不可读性改造，用`uuid to int64`，把它转成 long 类型：

```java
byte[] bytes = Guid.NewGuid().ToByteArray();
return BitConverter.ToInt64(bytes, 0);
```

又比如，改造无序性，比如 `NHibernate` 的 `Comb` 算法，把 uuid 的前 20 个字符保留下来，后面 12 个字符用 `guid` 生成的时间,时间是大致有序的，是一种小改进。

点评：**UUID不存在数据库当索引，作为一些日志，上下文的识别，还是挺香的，但是要是这玩意用来当订单号，真是令人崩溃**

### 数据库自增序列

#### 单机的数据库

数据库的主键本身就拥有一个自增的天然特性，只要设置ID为主键并且自增，我们就可以向数据库中插入一条记录，可以返回自增的ID，比如以下的建表语句：

```sql
CREATE DATABASE `test`;
use test;
CREATE TABLE id_table (
    id bigint(20) unsigned NOT NULL auto_increment, 
    value char(10) NOT NULL default '',
    PRIMARY KEY (id),
) ENGINE=MyISAM;
```

插入语句：

```sql
insert into id_table(value)  VALUES ('v1');
```

优点：

- 单机，简单，速度也很快
- 天然自增，原子性
- 数字id排序，搜索，分页都比较有利

缺点也很明显：

- 单机，挂了就要提桶跑路了
- 一台机器，高并发也不可能

#### 集群的数据库

既然单机高并发和高可用搞不定，那就加机器，搞集群模式的数据库，既然集群模式，如果有多个master，那肯定不能每台机器自己生成自己的id，这样会导致重复的id。

这个时候，每台机器设置**起始值**和**步长**，就尤为重要。比如三台机器V1，V2，V3：

```txt
统一步长：3
V1起始值：1
V2起始值：2
V3起始值：3
```

生成的ID：

```txt
V1：1, 4, 7, 10...
V2：2, 5, 8, 11...
V3：3, 6, 9, 12...
```

设置命令行可以使用：

```sql
set @@auto_increment_offset = 1;     // 起始值
set @@auto_increment_increment = 3;  // 步长
```

这样确实在master足够多的情况下，高性能保证了，就算有的机器宕机了，slave 也可以补充上来，基于主从复制就可以，可以大大降低对单台机器的压力。但是这样做还是有缺点：

- 主从复制延迟了，master宕机了，从节点切换成为主节点之后，可能会重复发号。
- 起始值和步长设置好之后，要是后面需要增加机器（水平拓展），要调整很麻烦，很多时候可能需要停机更新

#### 批量号段式数据库

上面的访问数据库太频繁了，并发量一上来，很多小概率问题都可能发生，那为什么我们不直接一次性拿出一段id呢？直接放在内存里，以供使用，用完了再申请一段就可以了。同样也可以保留集群模式的优点，每次从数据库取出一个范围的id，比如3台机器，发号：

```txt
每次取1000，每台步长3000
V1：1-1000,3001-4000,
V2：1001-2000,4001-5000
V3：2001-3000,5001-6000
```

当然，如果不搞多台机器，也是可以的，一次申请10000个号码，用乐观锁实现，加一个版本号，

```sql
CREATE TABLE id_table (
  id int(10) NOT NULL,
  max_id bigint(20) NOT NULL COMMENT '当前最大id',
  step int(20) NOT NULL COMMENT '号段的步长',
  version int(20) NOT NULL COMMENT '版本号',
  `create_time` datetime DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
  `update_time` datetime DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) 
```

只有用完的时候，才会重新去数据库申请，竞争的时候乐观锁保证只能一个请求成功，其他的直接等着别人取出来放在应用内存里面，再取就可以了，取的时候其实就是一个update操作：

```sql
update id_table set max_id = #{max_id+step}, version = version + 1 where version = # {version}
```

重点：

- 批量获取，减少数据库请求
- 乐观锁，保证数据准确
- 获取只能从数据库中获取，批量获取可以做成异步定时任务，发现少于某个阈值，自动补充



### Redis自增

redis有一个原子命令`incr`,原子自增，redis速度快，基于内存：

```shell
127.0.0.1:6379> set id 1
OK
127.0.0.1:6379> incr id      
(integer) 2
```

当然，redis 如果单机有问题，也可以上集群，同样可以用初始值 + 步长，可以用 `INCRBY` 命令，搞几台机器基本能抗住高并发。

优点：

- 基于内存，速度快
- 天然排序，自增，有利于排序搜索

缺点：

- 步长确定之后，增加机器也比较难调整
- 需要关注持久化，可用性等，增加系统复杂度

 redis持久化如果是RDB，一段时间打一个快照，那么可能会有数据没来得及被持久化到磁盘，就挂掉了，重启可能会出现重复的ID，同时要是主从延迟，主节点挂掉了，主从切换，也可能出现重复的ID。如果使用AOF，一条命令持久化一次，可能会拖慢速度，一秒钟持久化一次，那么就可能最多丢失一秒钟的数据，同时，数据恢复也会比较慢，这是一个取舍的过程。

### Zookeeper生成唯一ID

zookeeper其实是可以用来生成唯一ID的，但是大家不用，因为性能不高。znode有数据版本，可以生成32或者64位的序列号，这个序列号是唯一的，但是如果竞争比较大，还需要加分布式锁，不值得，效率低。



### 美团的Leaf

下面均来自美团的官方文档：https://tech.meituan.com/2019/03/07/open-source-project-leaf.html

> Leaf在设计之初就秉承着几点要求：
>
> 1. 全局唯一，绝对不会出现重复的ID，且ID整体趋势递增。
> 2. 高可用，服务完全基于分布式架构，即使MySQL宕机，也能容忍一段时间的数据库不可用。
> 3. 高并发低延时，在CentOS 4C8G的虚拟机上，远程调用QPS可达5W+，TP99在1ms内。
> 4. 接入简单，直接通过公司RPC服务或者HTTP调用即可接入。

文档里面讲得很清晰，一共有两个版本：

- V1：预分发的方式提供ID，也就是前面说的号段式分发，表设计也差不多，意思就是批量的拉取id

![image-20211012002835752](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/blog/image-20211012002835752.png)

这样做的缺点就是更新号段的时候，耗时比较高，还有就是如果这时候宕机或者主从复制，就不可用。

优化：

- 1.先做了一个双Buffer优化，就是异步更新，意思就是搞两个号段出来，一个号段比如被消耗10%的时候，就开始分配下一个号段，有种提前分配的意思，而且异步线程更新
- 2.上面的方案，号段可能固定，跨度可能太大或者太小，那就做成动态变化，根据流量来决定下一次的号段的大小，动态调整



- V2：Leaf-snowflake，Leaf提供了Java版本的实现，同时对Zookeeper生成机器号做了弱依赖处理，即使Zookeeper有问题，也不会影响服务。Leaf在第一次从Zookeeper拿取workerID后，会在本机文件系统上缓存一个workerID文件。即使ZooKeeper出现问题，同时恰好机器也在重启，也能保证服务的正常运行。这样做到了对第三方组件的弱依赖，一定程度上提高了SLA。

### snowflake(雪花算法）

snowflake 是 twitter 公司内部分布式项目采用的 ID 生成算法,开源后广受欢迎，它生成的ID是 `Long` 类型，8个字节，一共64位，从左到右：

- 1位：不使用，二进制中最高位是为1都是负数，但是要生成的唯一ID都是正整数，所以这个1位固定为0。
- 41位：记录时间戳(毫秒)，这个位数可以用 $(2^{41}-1) / (1000 * 60 * 60 * 24 * 365) = 69$年
- 10位：记录工作机器的ID，可以机器ID，也可以机房ID + 机器ID
- 12位：序列号，就是某个机房某台机器上这一毫秒内同时生成的 id 序号



那么每台机器按照上面的逻辑去生成ID，就会是趋势递增的，因为时间在递增，而且不需要搞个分布式的，简单很多。

可以看出 snowflake 是强依赖于时间的，因为时间理论上是不断往前的，所以这一部分的位数，也是趋势递增的。但是有一个问题，是时间回拨，也就是时间突然间倒退了，可能是故障，也可能是重启之后时间获取出问题了。那我们该如何解决时间回拨问题呢？

- 第一种方案：获取时间的时候判断，如果小于上一次的时间戳，那么就不要分配，继续循环获取时间，直到时间符合条件。
- 第二种方案：上面的方案只适合时钟回拨较小的，如果间隔过大，阻塞等待，肯定是不可取的，因此要么超过一定大小的回拨直接报错，拒绝服务，或者有一种方案是利用拓展位，回拨之后在拓展位上加1就可以了，这样ID依然可以保持唯一。



Java代码实现：

```java
public class SnowFlake {

    // 数据中心(机房) id
    private long datacenterId;
    // 机器ID
    private long workerId;
    // 同一时间的序列
    private long sequence;

    public SnowFlake(long workerId, long datacenterId) {
        this(workerId, datacenterId, 0);
    }

    public SnowFlake(long workerId, long datacenterId, long sequence) {
        // 合法判断
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
        }
        if (datacenterId > maxDatacenterId || datacenterId < 0) {
            throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
        }
        System.out.printf("worker starting. timestamp left shift %d, datacenter id bits %d, worker id bits %d, sequence bits %d, workerid %d",
                timestampLeftShift, datacenterIdBits, workerIdBits, sequenceBits, workerId);

        this.workerId = workerId;
        this.datacenterId = datacenterId;
        this.sequence = sequence;
    }

    // 开始时间戳
    private long twepoch = 1420041600000L;

    // 机房号，的ID所占的位数 5个bit 最大:11111(2进制)--> 31(10进制)
    private long datacenterIdBits = 5L;

    // 机器ID所占的位数 5个bit 最大:11111(2进制)--> 31(10进制)
    private long workerIdBits = 5L;

    // 5 bit最多只能有31个数字，就是说机器id最多只能是32以内
    private long maxWorkerId = -1L ^ (-1L << workerIdBits);

    // 5 bit最多只能有31个数字，机房id最多只能是32以内
    private long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);

    // 同一时间的序列所占的位数 12个bit 111111111111 = 4095  最多就是同一毫秒生成4096个
    private long sequenceBits = 12L;

    // workerId的偏移量
    private long workerIdShift = sequenceBits;

    // datacenterId的偏移量
    private long datacenterIdShift = sequenceBits + workerIdBits;

    // timestampLeft的偏移量
    private long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;

    // 序列号掩码 4095 (0b111111111111=0xfff=4095)
    // 用于序号的与运算，保证序号最大值在0-4095之间
    private long sequenceMask = -1L ^ (-1L << sequenceBits);

    // 最近一次时间戳
    private long lastTimestamp = -1L;


    // 获取机器ID
    public long getWorkerId() {
        return workerId;
    }


    // 获取机房ID
    public long getDatacenterId() {
        return datacenterId;
    }


    // 获取最新一次获取的时间戳
    public long getLastTimestamp() {
        return lastTimestamp;
    }


    // 获取下一个随机的ID
    public synchronized long nextId() {
        // 获取当前时间戳，单位毫秒
        long timestamp = timeGen();

        if (timestamp < lastTimestamp) {
            System.err.printf("clock is moving backwards.  Rejecting requests until %d.", lastTimestamp);
            throw new RuntimeException(String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds",
                    lastTimestamp - timestamp));
        }

        // 去重
        if (lastTimestamp == timestamp) {

            sequence = (sequence + 1) & sequenceMask;

            // sequence序列大于4095
            if (sequence == 0) {
                // 调用到下一个时间戳的方法
                timestamp = tilNextMillis(lastTimestamp);
            }
        } else {
            // 如果是当前时间的第一次获取，那么就置为0
            sequence = 0;
        }

        // 记录上一次的时间戳
        lastTimestamp = timestamp;

        // 偏移计算
        return ((timestamp - twepoch) << timestampLeftShift) |
                (datacenterId << datacenterIdShift) |
                (workerId << workerIdShift) |
                sequence;
    }

    private long tilNextMillis(long lastTimestamp) {
        // 获取最新时间戳
        long timestamp = timeGen();
        // 如果发现最新的时间戳小于或者等于序列号已经超4095的那个时间戳
        while (timestamp <= lastTimestamp) {
            // 不符合则继续
            timestamp = timeGen();
        }
        return timestamp;
    }

    private long timeGen() {
        return System.currentTimeMillis();
    }

    public static void main(String[] args) {
        SnowFlake worker = new SnowFlake(1, 1);
        long timer = System.currentTimeMillis();
        for (int i = 0; i < 100; i++) {
            worker.nextId();
        }
        System.out.println(System.currentTimeMillis());
        System.out.println(System.currentTimeMillis() - timer);
    }

}
  
```

### 百度 uid-generator

换汤不换药，百度开发的，基于`Snowflake`算法，不同的地方是可以自己定义每部分的位数,也做了不少优化和拓展：https://github.com/baidu/uid-generator/blob/master/README.zh_cn.md

> UidGenerator是Java实现的, 基于[Snowflake](https://github.com/twitter/snowflake)算法的唯一ID生成器。UidGenerator以组件形式工作在应用项目中, 支持自定义workerId位数和初始化策略, 从而适用于[docker](https://www.docker.com/)等虚拟化环境下实例自动重启、漂移等场景。 在实现上, UidGenerator通过借用未来时间来解决sequence天然存在的并发限制; 采用RingBuffer来缓存已生成的UID, 并行化UID的生产和消费, 同时对CacheLine补齐，避免了由RingBuffer带来的硬件级「伪共享」问题. 最终单机QPS可达600万。

## 秦怀の观点

不管哪一种uid生成器，保证唯一性是核心，在这个核心上才能去考虑其他的性能，或者高可用等问题，总体的方案分为两种：

- 中心化：第三方的一个中心，比如 Mysql，Redis，Zookeeper
  - 优点：趋势自增
  - 缺点：增加复杂度，一般得集群，提前约定步长之类
- 无中心化：直接本地机器上生成，snowflake，uuid
  - 优点：简单，高效，没有性能瓶颈
  - 缺点：数据比较长，自增属性较弱

没有哪一种是完美的，只有符合业务以及当前体量的方案，技术方案里面，没有最优解。

