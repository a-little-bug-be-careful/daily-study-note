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

常用命令

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

  

