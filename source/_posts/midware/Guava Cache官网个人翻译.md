---
title: Guava Cache官网个人翻译
categories:
  - 中间件
tags:
  - guava
date: 2019-10-17 21:01:51
---

示例

```java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumSize(1000)
       .expireAfterWrite(10, TimeUnit.MINUTES)
       .removalListener(MY_LISTENER)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) throws AnyException {
               return createExpensiveGraph(key);
             }
           });
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 应用场景

缓存在很多场景里面非常有用。例如当你需要多次访问一个计算或者查询加载耗时成本高时，你就可以考虑使用缓存。

Guava的Cache（后文简写成Cache）类似于ConcurrentMap，但是并不完全一样。最根本的不同就是ConcurrentMap会一直持有所有缓存元素，直至它们被显示地移除。而Cache考虑到内存占用，通常支持配置缓存元素自动过期的策略。在一些场景，Guava的LoadingCache（后文简写成LoadingCache）会更有用，因为它能够自动加载缓存。

总的来说，Guava的缓存工具包可以适用于以下场景：

- 愿意使用空间来换取时间
- 多次访问同一key
- 需要缓存的数据不超过可以使用的内存

提示：

如果你不需要Cache的特性，ConcurrentMap内存使用效率更高。但是使用ConcurrentMap实现Cache的特性，会非常困难，而且几乎是不可能的。



# 加载

获取Cache的方式是通过CacheBuilder构造，它支持定制化构造Cache。

在正式获取Cache之前，你需要清楚缓存的value是怎么通过key得到的。以下有三种加载key-value到Cache中的方式。

## 从CacheLoader

LoadingCache是通过一个对应的CacheLoader构建的。创建一个CacheLoader最典型的就是只需要实现这个方法：V load(K key) throws Exception 。例如以下代码

```java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumSize(1000)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) throws AnyException {
               return createExpensiveGraph(key);
             }
           });
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

LoadingCache的经典使用就是get(key)获取缓存，机制是如果key对应的value已经存在于缓存中，就直接返回，否则使用之前定义的CacheLoader.load方法获取value，并返回。具体使用代码如下：

```java
try {
  return graphs.get(key);
} catch (ExecutionException e) {
  throw new OtherException(e.getCause());
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

需要tryCatch的原因就是CacheLoader.load()方法声明了抛出检查异常。如果不想使用try-catch可以使用getUnchecked(key)方法，实际上如果load抛出了检查异常，会包装成非检查异常后抛出。具体代码如下：

```java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .expireAfterAccess(10, TimeUnit.MINUTES)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) { // no checked exception
               return createExpensiveGraph(key);
             }
           });
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

获取缓存代码如下：

```java
return graphs.getUnchecked(key);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

批量查询能通过getAll(Iterable<? extends K>)方法实现，通过getAll方法能获取缓存里面所有的值。当批量查询比单独查询更有效率时，可以通过覆盖CacheLoader.loadAll方法来实现，相应的getAll(Iterable)的性能也会提高。

注意当你实现loadAll方法时，意味着会在同一时刻加载那些你并不急需的key-value。

## 从Callable

所有的Guava缓存都支持get(K, Callable<V>)方法，这个方法语义是：如果对应的key-value已经缓存，则直接返回；否则执行Callable方法获取value，然后缓存并返回。

```java
Cache<Key, Value> cache = CacheBuilder.newBuilder()
    .maximumSize(1000)
    .build(); // 没有CacheLoader
...
try {
  // 如果key对于value不在缓存中，则执行doThingsTheHardWay(key)方法
  cache.get(key, new Callable<Value>() {
    @Override
    public Value call() throws AnyException {
      return doThingsTheHardWay(key);
    }
  });
} catch (ExecutionException e) {
  throw new OtherException(e.getCause());
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 直接插入

Cache.put(key,value)方法可以直接插入缓存值，执行这个方法会覆盖掉缓存中之前已经存在key对应的value值。可以使用Cache.asMap()视图的公开的任何ConcurrentMap的方法对缓存进行修改，但是asMap视图的任何方法都不能保证缓存项被原子地加载到缓存中。进一步说，asMap视图的原子运算在Guava Cache的原子加载范畴之外，所以相比于Cache.asMap().putIfAbsent(K,V)，Cache.get(K, Callable<V>) 应该总是优先使用。



# 过期策略

不幸的是，我们总是没有足够的内存，去缓存所有想要的key-value。你必须决定什么时候不再持有某些key-value。Guava提供了三种基本的缓存过期策略：基于容量、基于时间和基于引用。

## 基于容量的过期策略

如果你的缓存不能超过一定容量，可以使用CacheBuilder.maximumSize(long)来设置。缓存会尝试回收最近没有被用或者没有被经常用的key-value。警告：可能会在缓存接近容量最大时，就执行回收操作。

另外，如果你缓存的value有不同权重（比如，占用内存的大小），可是使用CacheBuilder.weigher(Weigher)指定一个权重方法，并且通过CacheBuilder.maximumWeight(long)指定一个最大权重。警告：同样在接近最大权重的时候，可能就执行回收操作；而且权重方法是在创建或者覆盖时计算的，需要考虑该方法的复杂度。

```java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumWeight(100000)
       .weigher(new Weigher<Key, Graph>() {
          public int weigh(Key k, Graph g) {
            return g.vertices().size();
          }
        })
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) { // no checked exception
               return createExpensiveGraph(key);
             }
           });
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 基于时间的过期策略

CacheBuilder提供两种时间过期策略：

- expireAfterAccess(long, TimeUnit) 当key-value在最近访问（读或者写）后一段时间就过期。
- expireAfterWrite(long, TimeUnit) 当key-value在写入或者重新加载后一段时间就过期。

提示：对基于时间的过期策略进行测试，并不需要真正等时间过去很久，只需要使用CacheBuilder.ticker(Ticker) 方法就可以模拟测试了。

## 基于引用的过期策略

Guava支持使用弱引用设置key和value，使用软引用设置value。

- CacheBuilder.weakKeys() 用弱引用存储key，当key没有强和软引用的时候，key就可以被GC。
- CacheBuilder.weakValues() 用弱引用存储value，当value没有强和软引用的时候，value就可以被GC。
- CacheBuilder.softValues()用软引用存储value，当value没有强引用的时候，value在内存溢出之前，可以被GC。

Guava官方推荐使用更具有预测性基于容量的过期策略，而非是基于引用的过期策略。

## 主动失效

在任何时候，都可以主动失效key-value，而非要等到自动过期。

- 单个失效，Cache.invalidate(key)
- 批量失效，Cache.invalidateAll(keys)
- 全部失效，Cache.invalidateAll()

## 失效动作的监听器

可以为缓存项失效动作定义监听器，通过CacheBuilder.removalListener(RemovalListener)方法实现。注意：监听器里面任何异常都会被打印以及吞掉。

```java
CacheLoader<Key, DatabaseConnection> loader = new CacheLoader<Key, DatabaseConnection> () {
  public DatabaseConnection load(Key key) throws Exception {
    return openConnection(key);
  }
};
RemovalListener<Key, DatabaseConnection> removalListener = new RemovalListener<Key, DatabaseConnection>() {
  public void onRemoval(RemovalNotification<Key, DatabaseConnection> removal) {
    DatabaseConnection conn = removal.getValue();
    conn.close(); // tear down properly
  }
};

return CacheBuilder.newBuilder()
  .expireAfterWrite(2, TimeUnit.MINUTES)
  .removalListener(removalListener)
  .build(loader);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

警告：默认情况，监听器方法是在失效动作发生时同步调用的，如果你的监听器方法非常耗时，会影响一般的缓存功能。这种情况可以通过异步实现，具体为RemovalListeners.asynchronous(RemovalListener, Executor)

## 清理何时发生？

用CacheBuilder构建的Caches不会自动的执行清理和过期values，或者在一个值过期后立即执行，或者任何排序操作。代替地是，在写操作时或者当写操作很少时，在读操作执行少量的维护。

上述这样做的原因是：如果我们想要持续性的维护缓存，我们需要创建一个线程来实现；但是这个线程的操作可能与用户的操作发生竞争，抢锁。另外，一些环境可能限制线程的创建，会使得CacheBuilder无法使用。

相应的，我们把这个权利给到用户。如果你的缓存是高吞吐的，那么你不需要担心清理过期的缓存项。如果你的缓存写非常少，你不想清理操作阻塞缓存的读，那么你可能需要创建另外线程规律地来调度Cache.cleanUp()方法。

如果你想定时清理维护缓存，当写操作非常少的时候，你可以使用ScheduledExecutorService。

## 刷新

刷新和过期不同。刷新通过LoadingCache.refresh(K)实现，刷新动作可能是异步的。当key开始刷新，旧值依然可以访问。相反的，当value过期时，会强制等待直到新值被加载。

如果刷新的时候发生异常，旧值会被保存，并且异常会被打印和吞掉。

CacheLoader可以覆盖CacheLoader.reload方法，指定刷新时使用一些智能行为，可以在计算新值时使用旧值。

```java
// 部分key不需要刷新, 我们可以通过同步来刷新.
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumSize(1000)
       .refreshAfterWrite(1, TimeUnit.MINUTES)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) { // no checked exception
               return getGraphFromDatabase(key);
             }

             public ListenableFuture<Graph> reload(final Key key, Graph prevGraph) {
               if (neverNeedsRefresh(key)) {
                 return Futures.immediateFuture(prevGraph);
               } else {
                 // asynchronous!
                 ListenableFutureTask<Graph> task = ListenableFutureTask.create(new Callable<Graph>() {
                   public Graph call() {
                     return getGraphFromDatabase(key);
                   }
                 });
                 executor.execute(task);
                 return task;
               }
             }
           });
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

自动定时刷新能够通过CacheBuilder.refreshAfterWrite(long, TimeUnit)添加。跟expireAfterWrite相比，refreshAfterWrite会经过一定时间后刷新，但是刷新动作是在缓存项被访问时发生的。如果CacheLoader.reload是异步实现的，那么查询不会被刷新动作拖慢。你可以同时使用refreshAfterWrite和expireAfterWrite。

# 

# 特性

## 状态

调用CacheBuilder.recordStats()，可以得到缓存的状态集合。Cache.stats()方法返回一个CacheStats对象，提供状态信息如下：

- hitRate()，命中率
- averageLoadPenalty()，加载新值的平均耗时，单位纳秒
- evictionCount()，过期缓存的数量

还有更多其它的状态方法。建议在性能关键应用上，需要查看这些状态。

## asMap

您可以使用asmap视图将任何缓存作为concurrentmap查看，但是asmap视图如何与缓存交互需要一些解释。

- cache.asMap() 包含所有当前已经加载的缓存项，所以cache.asMap().keySet()包含所有当前已经加载的key。
- asMap().get(key)跟cache.getIfPresent(key)作用一样，并且不会引起值加载。
- 访问时间会被重置，当缓存读和写操作时。包括Cache.asMap().get(Object)和Cache.asMap().put(key,value)，但是不包括containsKey(Object)和Cache.asMap()操作。所以，cache.asMap().entrySet()的遍历也不会重置访问时间。

# 

# 中断

Loading方法（类似get方法）不会抛出中断异常。我们可以设计该方法支持中断，但是我们没有。因为支持中断只有少数用户获益，大多数用户反而会增加使用成本。详细原因，可以阅读英文原文。 
