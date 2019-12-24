（1）基础数据结构：字符串String、字典Hash、列表List、集合Set、有序集合SortedSet。
（2）高级数据结构: HyperLogLog、Geo、Pub/Sub。
（3）BloomFilter，RedisSearch，Redis-ML
（4）Redis缓存穿透、缓存雪崩和缓存击穿
	 -Redis缓存穿透：缓存穿透，是指查询一个数据库一定不存在的数据。 
			解决方案：BloomFilter布隆过滤器，key获取value值为空时锁上
     -缓存雪崩：缓存雪崩，是指在某一个时间段，缓存集中过期失效      
			解决方案：失效时间错开
	 -缓存击穿：是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个屏障上凿开了一个洞。
			解决方案：直接设为永不过期
            补充方案：接口限流与熔断、降级
 （5）Redis 支持 Lua 脚本并保证其原子性，使用 Lua 脚本实现锁校验与释放，并使用 Redis 的 eval() 函数执行 Lua 脚本。