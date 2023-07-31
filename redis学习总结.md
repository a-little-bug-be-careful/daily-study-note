# redis学习总结

## 一、基础数据类型及相关常用命令

1. string：基本的字符类串

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
   > 

2. set

   > ~~~sql
   > sadd key '1' '2'
   > spop key
   > scard key
   > smembers key
   > ~~~
   >
   > 

3. zset/sorter set

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



