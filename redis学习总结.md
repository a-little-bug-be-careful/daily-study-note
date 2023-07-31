# redis学习总结

## 一、基础数据类型及相关常用命令

1. string：基本的字符类串，不仅可以存储字符串，还可以存储数字，最大存储512M

   > ~~~sql
   > set testStr '1'
   > get testStr
   > exists testStr
   > getrange key start end  --获取指定key从start(开始为0)到end（包括end）的子串
   > getset testStr '12' --设置key的新值并返回旧值
   > mset t1 't1' t2 t2 --批量设置多个key
   > mget t1 t2
   > setnx t1 'hahaha' --如果不存在对应的key才设置值，否则返回false
   > setnx t3 't3'
   > ~~~
   >
   > 应用场景：
   >
   > - 缓存json字符串对象
   >
   > - 简单计数，incr和decr命令
   >
   > - 实现简单的分布式锁
   >
   >   > 思路1：
   >   >
   >   > 加锁利用setnx命令，因为redis命令执行时是原子操作，索引不会存在线程安全性问题，同时该命令意思是**如果不存在该key值，则设置该值，如果存在，则返回false，啥都不做**
   >   >
   >   > 解锁使用del命令删除对应的key值
   >   >
   >   > ~~~sql
   >   > --加锁
   >   > setnx redisLock 'thread-1'
   >   > --解锁
   >   > del redisLock
   >   > --解锁  为了防止锁被别的线程误删除，可以先判断当前锁是不是当前线程加的，如果是，再执行删除操作，否则不删除，由于使用了两个操作，一是判断锁，而是删除锁，需要保证两个操作在同一个事务内，保证其原子性，可以考虑使用lua脚本
   >   > if redis.call('GET','redisLock') == 'thread-1'
   >   > then
   >   > 	redis.call('DEL', 'redisLock')
   >   > 	return 1;
   >   > else
   >   > 	return 0;
   >   > end
   >   > ~~~
   >   >
   >   > 思路2：
   >   >
   >   > 思路1可以保证锁资源的互斥性，同一时刻只能有一个线程获取到锁，其他线程需要等待，但是如果持有锁的线程挂了，一直不释放锁，就会导致所有的线程都阻塞在那，无法运行。针对这种情况，可以设置一个过期时间，让锁到期释放即可。
   >   >
   >   > ~~~sql
   >   > --加锁
   >   > set redisLock 'thread-1' EX 3 NX --当redisLock不存在时，设置val值，同时设置过期时间3秒
   >   > --解锁
   >   > del redisLock
   >   > --解锁  为了防止锁被别的线程误删除，可以先判断当前锁是不是当前线程加的，如果是，再执行删除操作，否则不删除，主要是为了解决这个场景，比如线程A获取锁并设置了过期时间，然而在线程A没运行完锁就被释放了，线程B此时获取到了锁，开始执行线程B的逻辑，但是线程A此时突然执行完成了，就要执行释放锁的逻辑，但是此时的锁实际上已经是线程B设置的锁了，不再是线程A自己当初那把锁（已经过期了），如果不加比较操作，线程A就会释放这把锁，显然这不是我们想看到的。
   >   > --另外，由于使用了两个操作，一是判断锁，而是删除锁，需要保证两个操作在同一个事务内，保证其原子性，可以考虑使用lua脚本
   >   > if redis.call('GET','redisLock') == 'thread-1'
   >   > then
   >   > 	redis.call('DEL', 'redisLock')
   >   > 	return 1;
   >   > else
   >   > 	return 0;
   >   > end
   >   > ~~~
   >   >
   >   > 思路3：
   >   >
   >   > 思路3可以解决线程挂掉导致锁不能自动释放的问题，但是伴随过期时间的设置，也会有新的问题，比如设置的过期时间太短，线程任务还未执行完，锁就被释放了，这时候共享资源就有可能被其他线程访问修改造成线程不一致问题，或者设置的过期时间太长，线程任务执行完了，锁还未被释放，共享资源还是不能被其他线程访问。这种情况下可以考虑采用续费机制
   >   >
   >   > ~~~sql
   >   > 
   >   > ~~~
   >   >
   >   > 

2. set：不允许重复数字的集合，可以用来去重

   > ~~~sql
   > sadd key '1' '2'
   > spop key
   > scard key
   > smembers key
   > ~~~
   >
   > 

3. zset/sorter set：有序且不重复的数字集合，

   >~~~sql
   >zadd key score1 member1 [socre2 member2]
   >zcard key
   >~~~
   >
   >

4. hash

   > ~~~sql
   > hset h1 'key1' 'val1'
   > hget h1 'key1'
   > hmset h3 'key1' 'val1' 'key2' 'val2'
   > hmget h3 'key1' 'key2'
   > hdel h1 'key1' 'key2'
   > del h1
   > 
   > ~~~
   >
   > 

5. list

   > ~~~sql
   > lpush key val1 [val2] -- 列表头部加入元素
   > rpush key val1 [val2] -- 列表尾部加入元素
   > ~~~
   >
   > 



## 二、redis应用场景

1. 缓存
2. 分布式锁



## 三、redis和memcached对比

1. 

## 四、redis发布订阅

subscribe和publish命令实现消息的发布和订阅

## 五、redis事务

- multi开启事务，将多个命令放入一个事务，exec执行所有的命令。redis的命令是原子性的，但是事务并不是原子性的，也就是说事务中的一组命令，并不是要么全部成功，要么全部回滚，其中如果有命令出错，之前已执行的命令依然会生效，后续的命令依然可以被执行。

- multi开启事务之后，后续的命令都放在队列中缓存，不会立刻执行，当执行exec之后才会执行
- 在事务执行过程中，其他客户端提交的命令不会插入到当前事务待执行命令中。

## 六、redis之lua脚本

1. 客户端中lua常用命令

- EVAL：EVAL script numkeys key [key...] arg [arg...]

  script：一段lua脚本

  numkeys：代表key的数量

  key[key...]：代表具体的key值，在lua脚本中通过KEYS[1]、KEYS[2]...获取

  arg[arg...]：代表附加参数信息，在lua脚本中通过ARGV[1]、ARGV[2]...获取

  eg.  

  ~~~lua
   EVAL "redis.call('SET', KEYS[1], ARGV[1]);redis.call('EXPIRE', KEYS[1],ARGV[2]);return 1;" 1 luatest 'lua' 60
  --redis.call代表通过redis调用redis命令将第一个key（luatest）的值设置为'lua'，过期时间设置为60秒
  ~~~

- EVALSHA和SCRIPT LOAD

  > * SCRIPT LOAD命令格式：SCRIPT LOAD script
  >
  >   该命令并不会立即执行，而是加入到**redis服务器**脚本缓存中，计算出输入脚本的SHA1校验和，如果给定的脚本已存在，则不进行任何操作
  >
  > * EVALSHA命令格式：EVALSHA sha1 numkeys key [key...] arg [arg...]
  >
  >   通过load命令将脚本加入到redis缓存之后，可以在任何客户端通过EVALSHA命令执行该脚本，并传入指定的参数信息
  >
  >   ~~~sql
  >   script load "redis.call('SETNX',KEYS[1], ARGV[1]);return redis.call('GET',KEYS[1]);" --定义一个lua脚本
  >   evalsha 7ea0203ff452258e5d3a6e06173085f86e39265b 1 'luahahaha' 'nihaoa' --执行该脚本将得到luahahaha这个key对应的值nihaoa
  >   ~~~
  >
  >   

- SCRIPT EXISTS sha [sha1...]

  > 判断脚本是否存在
  >
  > ~~~sql
  > script exists 7ea0203ff452258e5d3a6e06173085f86e39265b --如果脚本存在输出true，否则输出false
  > ~~~
  >
  > 

- SCRIPT FLUSH

  > 刷新redis服务器的脚本缓存，清理掉所有的脚本缓存
  >
  > ~~~sql
  > script flush --清除redis服务器中所有lua脚本
  > script exists 7ea0203ff452258e5d3a6e06173085f86e39265b --输出false
  > ~~~
  >
  > 

- SCRIPT KILL

  > 终止lua脚本的执行，如果该脚本还未写入数据，则可以正常终止（比如一些死循环），否则不能够通过该命令终止命令执行，因为不符合lua脚本原子性的特征，lua脚本可以保证一组redis命令的原子性。

2. redis直接执行lua脚本文件

   利用lua脚本实现CAS

   ~~~sql
   --lua脚本
   local key = KEYS[1];
   local val = redis.call('GET', key);
   if val == ARGV[1]
   then
   	redis.call('SET', key, ARGV[2]);
   	return 1;
   else
   	return 0;
   end
   --直接通过redis-cli命令执行
   redis-cli -a 'pwd' --eval 'lua脚本的目录' key [key1...] , argv [argv1...]
   注意key和argv之间的 , 前后都要有空格
   ~~~

3. lua脚本的有点

   - 减少网络开销

     不需要每条redis命令都去建立连接请求，可以将一组命令作为一个脚本一起发送执行

   - 原子操作

     保证一组redis命令可以原子执行，完成一些复杂操作

   - 复用

     客户端发送的脚本可以永久存储在redis服务端lua脚本缓存中，其他客户端可以直接使用，不需要重复创建



