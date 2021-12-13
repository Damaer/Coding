前面讲解到，如果出现网络延迟的情况下，多个请求阻塞，那么恶意攻击就可以全部请求领取接口成功，而针对这种做法，我们使用`setnx`来解决，确保只有一个请求可以进入接口请求。

![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/20210226230957.png)

```java
    public String receiveGitf(int activityId,int giftId,String uid){
        // isExist判断活动是否存在，内部包括redis和数据库请求，省略
        if(isActivityExist(activityId,giftId)){
            // 活动和礼品有效,判断是否领取过
            if(!userReceived(uid,activityId,giftId)){
                // 没有领取过，调用C系统
                try {
                    // setnx
                    if(redis.setnx("uid_activityId_giftId")){
                        boolean receivedResult = Http.getMethod(C_Client.class, "distributeGift");
                        if(receivedResult){
                            // 领取成功更新mysql
                            updateMysql(uid,activityId,giftId);
                        }else{
                            // 领取成功更新redis
                            deleteRedis(uid,activityId,giftId);
                            return "已经领过/领取失败";
                        }
                    }else{
                        return "已经领过/领取失败";
                    }
                }catch (Exception e){
                    // 记录日志
                    logHelper.log(e);
                    return "调用领券系统失败，请重试";
                }
            }
        }
        return "领取失败，活动不存在";
    }
```

下面，我们就专门讲解一下`setnx`，`setnx`可以用作分布式锁，但是**这个场景并不是分布式锁的一个较好的实践，因为每个用户的key都是不一样的，我们主要是防止同一个用户恶意领取**，`setnx`本身是一个原子操作，可以保证多个线程只有一个能拿到锁，能返回`true`,其他的都会返回`false`。

但是上面的做法，没有设置过期时间，在生产上一般是不可以这么使用。**不设置过期时间的key多了之后，redis服务器很容易内存打满，这时候不知道哪些是强制依赖的，只能扩容，从代码层面去清理，如果直接清理不常用的，也很难保证不出事。**（基本不允许这么干，除非是基础数据，跟着服务器启动，写入`redis`的，不会变更的，比如城市数据，国家数据等等，当然，这些也可以考虑在本地内存中实现）

如果在上面的代码中，加入超时时间，假设是一个月或者半年，流程变成这样：
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/20210228165201.png)

设置key的超时时间使用`expire`,但是这样还有缺陷么？

在`redis 2.6.12`之前，`setnx`和`expire`都不是原子操作，也就是很有可能在`setnx`成功之后，redis当季，expire设置失败，也就不会有超时时间了。虽然这个影响在当前业务不是很大，但是还是一个小缺陷。

`Redis2.6.12`以上版本，可以用`set`获取锁,set包含`setnx`和`expire`，实现了原子操作。也就是两步要么一起成功，要么一起失败。

除此之外，上面的流程可能还存在的一个问题，是请求`C`服务的时候出现超时，然后删除key，恰好这个时候`redis`有问题，删除失败了，这个`key`就永远存在了。表现在业务上，就是`A`用户点击了领取，领取失败了，但是后面再怎么点，都是已经领取的状态了。

**那这种现象怎么优化呢？**

这种情况，其实已经是很少见的情况，按照我们当前的业务场景也看，就是当前的用户，`redis`记录了它已经领取过了，但是由于接口的失败，成功之后还没将`mysql/其他数据库`更新,两个数据库不一致了。

我能想到的一个方法，就是再删除失败的时候，告警，并且将业务相关的数据记录下来，比如`key`，`uid`等等，针对这部分数据，做一次补发，或者手动删除key。

或者，启动一个定时任务或者`lua`脚本，去判定`redis`和数据库不一致的情况，但是切记不要全部查询，应该是隔一段时间，查询最后增加的部分，做一个校验以及相应的处理。枚举`key`是十分耗时的操作！！！

`setnx` 除了解决上面的问题，还可以应用在解决**缓存击穿**的问题上。

譬如现在有热点数据，不仅在`mysql`数据库存储了，还在`redis`中存了一份缓存，那么如果有一个时间点，缓存失效了，这时候，大量的请求打过来，同时到达，缓存拿不到数据，都去数据库取数据，假设数据库操作比较耗时，那么压力全都在数据库服务器上了。

这个时候所有的请求都去更新数据，明显是不合适的，应该是使用分布式锁，让一个线程去请求`mysql`一次即可。但是为了避免死锁的情况，如果超时，得及时额外释放锁，要不可能请求`mysql`都失败了，其他线程又拿不到锁，那么数据就会一直为`null`了。

可以使用以下的命令：
```shell
SETNX lock.foo <current Unix time + lock timeout + 1>
```

关于这个场景下的`setnx`先讲到这里，后面再讲讲分布式锁相关的知识。

