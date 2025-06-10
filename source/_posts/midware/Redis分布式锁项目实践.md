---
title: Redis分布式锁项目实践
categories:
  - 中间件
tags:
  - redis
date: 2020-01-09 13:00:16
---

#  实现方案：

1. setNx(set if not exist)，这个是原子操作；
2. 加锁了之后如果机器宕机那么这个锁就不会得到释放，所以要加入过期时间，而且加入过期时间需要和setNx同一个原子操作；在Redis2.8之前我们需要使用Lua脚本达到我们的目的，但是redis2.8之后redis支持nx和ex操作是同一原子操作。
3. 使用ShardedJedis实现。

# 加锁：

```java
    private static final String LOCK_SUCCESS = "OK";
    private static final String SET_IF_NOT_EXIST = "NX";
    private static final String SET_WITH_EXPIRE_TIME = "PX";
    private staiic final ShardedJedis sjedis;
    
    private boolean tryGetDistributedLock(String lockKey, String lockId, int expireTime) {
        String result = sjedis.set(lockKey, lockId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME,
            expireTime);
        return LOCK_SUCCESS.equals(result);
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

#   解锁：

```java
    private static final Long RELEASE_SUCCESS = 1L;    
    private boolean releaseDistributedLock(String lockKey, String lockId) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        Jedis j = sjedis.getShard(key);
        Object result = j.eval(lockKey, script, Collections.singletonList(lockKey),
            Collections.singletonList(lockId));
        return RELEASE_SUCCESS.equals(result);
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# **注意点：**

1.无续租功能，所以要注意锁过期时间的设定。
 2.多节点redis，可能会出现重复锁，所以unlock需要捕获检查异常处理。
