Google Guava包含了Google的Java项目许多依赖的库，如：集合 [collections] 、缓存 [caching] 、原生类型支持 [primitives support] 、并发库 [concurrency libraries] 、通用注解 [common annotations] 、字符串处理 [string processing] 、I/O 等等。本文只介绍其中的缓存部分。

Guava Cache 是一种本地缓存实现，支持多种缓存过期策略。性能好，简单易用。缓存在很多场景下都是很有用的。如，通过key获取一个value的花费的时间很多，而且获取的次数不止一次的时候，就应该考虑使用缓存。Guava Cache与ConcurrentMap很相似，但也不完全一样。**最基本的区别是ConcurrentMap会一致保存所有添加的元素，直到显示地移除。而Guava Cache为了限制内存占用，通常都设定为自动回收元素。在某些场景下，尽管LoadingCache不回收元素，它也会自动加载缓存。**

Guava Cache 适用于以下应用场景：

​	系统的访问速度首要考虑，而内存空间为次要考虑。

​	某些key对于的value会被查询多次。

​	缓存中存放的数据总量不会超出内存的全部大小。



