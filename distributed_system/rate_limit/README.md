## 服务限流

### 服务限流场景

限流常见使用层面：

- 用户网络层：突发的流量场景如热点事件流量，恶意刷流，竞对爬虫等；

- 内部应用层：上游服务的异常调用，脚本异常请求，失败重试策略造成的流量突发；

### 实现方式

#### 计数器

比较常见且简单的方式即为计数器方式，使用某一公共存储变量进行计数，需要注意的是在计数时保证原子性。

- 使用场景：适用于做API限流或者根据IP做粒度控制等；

- 局限：由于计数一般为定速所以对于更细粒度时间控制能力较为有限；

#### 漏斗桶限流

![](http://img.zhengyua.cn/20210208203828.png)

漏斗桶核心在于它是匀速的，当桶满了，新流量过来就会被限流。

- 使用场景：与计数器限流相比限流过后的流量还有机会流入而不是直接舍弃，适合于频率控制操作；

- 局限：若短时有大量突发请求即使负载压力不大，但请求仍需要在队列处等待处理；

#### 令牌桶限流

![](http://img.zhengyua.cn/20210208214244.png)

令牌桶相对于漏斗桶控制时间粒度和应对突然流量的能力更加优秀。通过匀速往桶里放令牌，控制桶最大容量和放入令牌速率。请求若拿到了令牌则可以通过，反之则被限流。而令牌是可以积累的，说明能够应对流量不同大小的场景。

- 使用场景：应用较为广泛，如java中的guava就有实现；


## 参考

[分布式高并发服务限流实现方案](https://mp.weixin.qq.com/s?__biz=MzIyMzMxNjYwNw==&mid=2247483695&idx=1&sn=7d5528e8a6bc2296d4871e74a0270550&chksm=e8215e3fdf56d729b8e0d9fb077cbd37216173f6ffad442a6a3d383052d4f05892538ef64645&scene=178&cur_album_id=1511862059553095681#rd)