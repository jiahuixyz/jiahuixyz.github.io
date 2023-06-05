---
title: RedisTemplate大量使用的callback机制（转）
date: 2023-06-05 16:01:13
tags:
---

假如要让你封装jedis以便让外界调用你大概率会像下面方法一样实现。

    @Service
    public class JedisSpringDemo {
        @Resource(name = "shardedJedisPool")
        private ShardedJedisPool shardedJedisPool;
        
        public String set(String key, String value){
            ShardedJedis shardedJedis = null;
            try{
                // 从连接池中获取jedis分片对象
                shardedJedis = shardedJedisPool.getResource();
                // 设置值到redis中
                return shardedJedis.set(key, value);
            }catch (Exception e){
                System.out.println(e.getMessage());
            }finally {
                if(null != shardedJedis){
                    shardedJedis.close();
                }
            }
            return null;
        }

        public String get(String key){
            ShardedJedis shardedJedis = null;
            try{
                // 从连接池中获取jedis分片对象
                shardedJedis = shardedJedisPool.getResource();
                // 从redis中获取key对应value
                return shardedJedis.get(key);
            }catch (Exception e){
                System.out.println(e.getMessage());
            }finally {
                if(null != shardedJedis){
                    shardedJedis.close();
                }
            }
            return null;
        }
    }

上面的这段代码违反了DRY原则，两个方法get()和set()大部分代码是相同的（try，catch，finally这部分代码），唯一不同的就是我们return的那一行具体的调用方法，如果像这种方法很多的话(jedis提供了几十种类似的方法)，我们的代码重复率是很高的，代码重复率一旦高起来，相应的维护成本也会提高，下面我们就来引进一种改进方法--回调机制。

首先，我们创建一个接口类，该接口定义Jedis的操作，代码如下：

    public interface RedisOperations {
        <T> T execute(ConnectionCallback<T> action);
       String set(final String key, final String value);
       String get(final String key);
    }

其次，定义连接Redis服务器的回调接口，代码如下：

    public interface ConnectionCallback<T> {
        T doInRedis(ShardedJedis shardedJedis);
    }

最后定义具体的操作服务类，代码如下：

    @Service("redisTemplate")
    public class RedisTemplate implements RedisOperations{
        @Resource(name = "shardedJedisPool")
        private ShardedJedisPool shardedJedisPool;
        
        @Override
        public <T> T execute(ConnectionCallback<T> action) {
            ShardedJedis shardedJedis = null;
            try{
                // 从连接池中获取jedis分片对象
                shardedJedis = shardedJedisPool.getResource();
                
                return action.doInRedis(shardedJedis);
                
            }catch (Exception e){
                System.out.println(e.getMessage());
            }finally {
                if(null != shardedJedis){
                    shardedJedis.close();
                }
            }
            return null;
        }
        
       /**
         * attention:真正封装的方法，非常的简洁干脆
         */
        public String set(final String key, final String value){
            return execute(new ConnectionCallback<String>() {
                @Override
                public String doInRedis(
                        ShardedJedis shardedJedis) {
                    return shardedJedis.set(key, value);
                }
            });
        }
        
        public String get(final String key){
            return execute(new ConnectionCallback<String>(){
                @Override
                public String doInRedis(ShardedJedis shardedJedis) {
                    return shardedJedis.get(key);
                }
            });
        }
    }

通过上面的代码，我们可以清晰的看到，将try，catch，finally部分的公共代码都封装到了execute()函数中，在具体方法内实现回调方法的具体业务逻辑即可，代码的复用率更高了。\
如果大家对spring jdbc或者是spring data redis的源码研究过，就应该知道JdbcTemplate和RedisTemplate这两个类，这两个框架中用到了大量的callback机制。\
一言以蔽之--如果你调用我，那么我就回调你传给我的对象的方法。

\
原文链接：<https://www.jianshu.com/p/26ba689b5549>
