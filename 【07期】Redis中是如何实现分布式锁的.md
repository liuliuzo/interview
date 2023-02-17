#介绍
分布式锁常见的三种实现方式：
	- 数据库乐观锁；
	- 基于Redis的分布式锁；
	- 基于ZooKeeper的分布式锁。

#要点
Redis要实现分布式锁，以下条件应该得到满足
	- 互斥性: 在任意时刻，只有一个客户端能持有锁。
	- 不能死锁: 客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
	- 容错性: 只要大部分的Redis节点正常运行，客户端就可以加锁和解锁。

#实现
可以直接通过 `set key value px milliseconds nx ` 命令实现加锁， 通过Lua脚本实现解锁。

```
//获取锁（unique_value可以是UUID等）
SET resource_name unique_value NX PX  30000

//释放锁（lua脚本中，一定要比较value，防止误解锁）
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

##代码解释
- set 命令要用 `set key value px milliseconds nx`，替代 setnx + expire 需要分两次执行命令的方式，保证了原子性，
- value 要具有唯一性，可以使用`UUID.randomUUID().toString()` 方法生成，用来标识这把锁是属于哪个请求加的，在解锁的时候就可以有依据；
- 释放锁时要验证 value 值，防止误解锁；
- 通过 Lua 脚本来避免 Check And Set 模型的并发问题，因为在释放锁的时候因为涉及到多个Redis操作 （利用了eval命令执行Lua脚本的原子性）；

##加锁代码分析
首先，set()加入了NX参数，可以保证如果已有key存在，则函数不会调用成功，也就是只有一个客户端能持有锁，满足互斥性。其次，由于我们对锁设置了过期时间，即使锁的持有者后续发生崩溃而没有解锁，锁也会因为到了过期时间而自动解锁（即key被删除），不会发生死锁。最后，因为我们将value赋值为requestId，用来标识这把锁是属于哪个请求加的，那么在客户端在解锁的时候就可以进行校验是否是同一个客户端。

##解锁代码分析
将Lua代码传到jedis.eval()方法里，并使参数KEYS[1]赋值为lockKey，ARGV[1]赋值为requestId。在执行的时候，首先会获取锁对应的value值，检查是否与requestId相等，如果相等则解锁（删除key）。

##存在的风险
如果存储锁对应key的那个节点挂了的话，就可能存在丢失锁的风险，导致出现多个客户端持有锁的情况，这样就不能实现资源的独享了。
- 客户端A从master获取到锁
- 在master将锁同步到slave之前，master宕掉了（Redis的主从同步通常是异步的）。
主从切换，slave节点被晋级为master节点
- 客户端B取得了同一个资源被客户端A已经获取到的另外一个锁。导致存在同一时刻存不止一个线程获取到锁的情况。

##redlock算法出现 `TODO`

##Redisson实现 `TODO`