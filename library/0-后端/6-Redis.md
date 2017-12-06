# <center>Redis</center>

## NoSql
非关系型数据库，适合高并发场景
![4种不同类型的NoSql比较](assets/0/20170819-9bac28ff.png)  

## Redis
**使用场景：**
- 缓存
- 任务队列
- 。。。  

**多种键值数据类型：**  
Redis使用键值对存储数据，它的value可以是多种数据结构。

## Jedis
Redis的Java客户端开发包
```Java
    //单例操作
    Jedis jedis = new Jedis("localhost", 6379);
    jedis.set("singleJedis", "hello jedis!");
    System.out.println(jedis.get("singleJedis"));
    jedis.close();  
```
```Java
    //使用连接池
    JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
    jedisPoolConfig.setMaxTotal(10);
    JedisPool pool = new JedisPool(jedisPoolConfig, "localhost", 6379);

    Jedis jedis = null;
    try{
        jedis = pool.getResource();
        jedis.set("pooledJedis", "hello jedis pool!");
        System.out.println(jedis.get("pooledJedis"));
    }catch(Exception e){
        e.printStackTrace();
    }finally {
        //还回pool中
        if(jedis != null){
            jedis.close();
        }
    }
    pool.close();
```

## Redis多数据库
一个Redis中最多可包含16个数据库，连接时可以通过指定连接哪个数据库  
默认连接到第0个数据库

## Redis事务操作
- multi 开启
- exec 提交
- discard 回滚

## Redis持久化
**RDB方式：**  
定时存储到硬盘：可能会丢失一定数据，效率高  
**AOF方式：**  
实时存储到硬盘：安全、效率低、文件较大  
