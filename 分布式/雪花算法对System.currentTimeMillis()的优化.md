前面已经讲过了雪花算法，里面使用了`System.currentTimeMillis()`获取时间，有一种说法是认为`System.currentTimeMillis()`慢，是因为每次调用都会去跟系统打一次交道，在高并发情况下，大量并发的系统调用容易会影响性能（对它的调用甚至比`new`一个普通对象都要耗时，毕竟`new`产生的对象只是在`Java`内存中的堆中）。我们可以看到它调用的是`native` 方法：

```java
// 返回当前时间，以毫秒为单位。注意，虽然返回值的时间单位是毫秒，但值的粒度取决于底层操作系统，可能更大。例如，许多操作系统以数十毫秒为单位度量时间。
public static native long currentTimeMillis();
```

所以有人提议，用后台线程定时去更新时钟，并且是单例的，避免每次都与系统打交道，也避免了频繁的线程切换，这样或许可以提高效率。

## 这个优化成立么？

先上优化代码：

```java
package snowflake;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicLong;

public class SystemClock {

    private final int period;

    private final AtomicLong now;

    private static final SystemClock INSTANCE = new SystemClock(1);

    private SystemClock(int period) {
        this.period = period;
        now = new AtomicLong(System.currentTimeMillis());
        scheduleClockUpdating();
    }

    private void scheduleClockUpdating() {
        ScheduledExecutorService scheduleService = Executors.newSingleThreadScheduledExecutor((r) -> {
            Thread thread = new Thread(r);
            thread.setDaemon(true);
            return thread;
        });
        scheduleService.scheduleAtFixedRate(() -> {
            now.set(System.currentTimeMillis());
        }, 0, period, TimeUnit.MILLISECONDS);
    }

    private long get() {
        return now.get();
    }

    public static long now() {
        return INSTANCE.get();
    }

}
```

只需要用`SystemClock.now()`替换`System.currentTimeMillis()`即可。



雪花算法`SnowFlake`的代码也放在这里：

```java
package snowflake;

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

    // 开始时间戳（2021-10-16 22:03:32）
    private long twepoch = 1634393012000L;

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
        return SystemClock.now();
        // return System.currentTimeMillis();
    }

    public static void main(String[] args) {
        SnowFlake worker = new SnowFlake(1, 1);
        long timer = System.currentTimeMillis();
        for (int i = 0; i < 10000000; i++) {
            worker.nextId();
        }
        System.out.println(System.currentTimeMillis());
        System.out.println(System.currentTimeMillis() - timer);
    }
}
```



Windows：i5-4590 16G内存 4核 512固态

Mac: Mac pro 2020 512G固态 16G内存

Linux：deepin系统，虚拟机，160G磁盘，内存8G



单线程环境测试一下 `System.currentTimeMillis()`：

|  平台/数据量  | 10000 | 1000000 | 10000000 | 100000000 |
| --- | --- | --- | --- | --- |
|      mac      |   5   |   247   |   2444   |   24416   |
|    windows    |   3   |   249   |   2448   |   24426   |
| linux(deepin) |  135  |   598   |   4076   |   26388   |



单线程环境测试一下 `SystemClock.now()`：

|  平台/数据量  | 10000 | 1000000 | 10000000 | 100000000 |
| --- | --- | --- | --- | --- |
|      mac      |  52   |   299   |   2501   |   24674   |
|    windows    |  56   |  3942   |  38934   |  389983   |
| linux(deepin) |  336  |  1226   |   4454   |   27639   |



上面的单线程测试并没有体现出后台时钟线程处理的优势，反而在windows下，数据量大的时候，变得异常的慢,linux系统上，也并没有快，反而变慢了一点。



多线程测试代码：

```java
    public static void main(String[] args) throws InterruptedException {
        int threadNum = 16;
        CountDownLatch countDownLatch = new CountDownLatch(threadNum);
        int num = 100000000 / threadNum;
        long timer = System.currentTimeMillis();
        thread(num, countDownLatch);
        countDownLatch.await();
        System.out.println(System.currentTimeMillis() - timer);

    }

    public static void thread(int num, CountDownLatch countDownLatch) {
        List<Thread> threadList = new ArrayList<>();
        for (int i = 0; i < countDownLatch.getCount(); i++) {
            Thread cur = new Thread(new Runnable() {
                @Override
                public void run() {
                    SnowFlake worker = new SnowFlake(1, 1);
                    for (int i = 0; i < num; i++) {
                        worker.nextId();
                    }
                    countDownLatch.countDown();
                }
            });
            threadList.add(cur);
        }
        for (Thread t : threadList) {
            t.start();
        }
    }
```



下面我们用不同线程数来测试 100000000(一亿) 数据量 `System.currentTimeMillis()`：

| 平台/线程 |   2   |   4   |   8   |  16   |
| --- | --- | --- | --- | --- |
|    mac    | 14373 | 6132  | 3410  | 3247  |
|  windows  | 12408 | 6862  | 6791  | 7114  |
|   linux   | 20753 | 19055 | 18919 | 19602 |



用不同线程数来测试 100000000(一亿) 数据量 `SystemClock.now()`：

| 平台/线程 |   2    |   4    |   8    |   16   |
| --- | :----: | :----: | :----: | :----: |
|    mac    | 12319  |  6275  |  3691  |  3746  |
|  windows  | 194763 | 110442 | 153960 | 174974 |
|   linux   | 26516  | 25313  | 25497  | 25544  |

在多线程的情况下，我们可以看到mac上没有什么太大变化，随着线程数增加，速度还变快了，直到超过 8 的时候，但是windows上明显变慢了，测试的时候我都开始刷起了小视频，才跑出来结果。而且这个数据和处理器的核心也是相关的，当windows的线程数超过了 4 之后，就变慢了，原因是我的机器只有四核，超过了就会发生很多上下文切换的情况。

linux上由于虚拟机，核数增加的时候，并无太多作用，但是时间对比于直接调用 `System.currentTimeMillis()`其实是变慢的。



**但是还有个问题，到底不同方法调用，时间重复的概率哪一个大呢？**

```java
    static AtomicLong atomicLong = new AtomicLong(0);
    private long timeGen() {
        atomicLong.incrementAndGet();
        // return SystemClock.now();
        return System.currentTimeMillis();
    }
```



下面是1千万id，八个线程，测出来调用`timeGen()`的次数，也就是可以看出时间冲突的次数：

| 平台/方法 | SystemClock.now() | System.currentTimeMillis() |
| --- | --- | --- |
|    mac    |     23067209      |          12896314          |
|  windows  |     705460039     |          35164476          |
|   linux   |    1165552352     |          81422626          |

可以看出确实`SystemClock.now()`自己维护时间，获取的时间相同的可能性更大，会触发更多次数的重复调用，冲突次数变多，这个是不利因素！还有一个残酷的事实，那就是自己定义的后台时间刷新，获取的时间不是那么的准确。在linux中的这个差距就更大了，时间冲突次数太多了。



## 结果

实际测试下来，并没有发现`SystemClock.now()`能够优化很大的效率，反而会由于竞争，获取时间冲突的可能性更大。`JDK`开发人员真的不傻，他们应该也经过了很长时间的测试，比我们自己的测试靠谱得多，因此，个人观点，最终证明这个优化并不是那么的可靠。 

不要轻易相信某一个结论，如果有疑问，请一定做做实验，或者找足够权威的说法。

