# redis

## Redis简介
Redis 是一个使用 C 语言编写的，开源的（BSD许可）高性能非关系型（NoSQL）的键值对数据库。

Redis 可以存储键和五种不同类型的值之间的映射。键的类型只能为字符串，值支持五种数据类型：字符串、列表、集合、散列表、有序集合。

与传统数据库不同的是 Redis 的数据是存在内存中的，所以读写速度非常快，因此 redis 被广泛应用于缓存方向，每秒可以处理超过 10万次读写操作，是已知性能最快的Key-Value DB。另外，Redis 也经常用来做分布式锁。除此之外，Redis 支持事务 、持久化、LUA脚本、LRU驱动事件、多种集群方案。

从2010年3月15日起，Redis的开发工作由VMware主持。从2013年5月开始，Redis的开发由Pivotal赞助。

## Redis的优缺点

### 优点
* 读写性能优异， Redis能读的速度是110000次/s，写的速度是81000次/s。
* 支持数据持久化，支持AOF和RDB两种持久化方式。
* 支持事务，Redis的所有操作都是原子性的，同时Redis还支持对几个操作合并后的原子性执行。
* 数据结构丰富，除了支持string类型的value外还支持hash、set、zset、list等数据结构。
* 支持主从复制，主机会自动将数据同步到从机，可以进行读写分离。

### 缺点
* 数据库容量受到物理内存的限制，不能用作海量数据的高性能读写，因此Redis适合的场景主要局限在较小数据量的高性能操作和运算上。
* Redis 不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的IP才能恢复。
* 主机宕机，宕机前有部分数据未能及时同步到从机，切换IP后还会引入数据不一致的问题，降低了系统的可用性。
* Redis 较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。为避免这一问题，运维人员在系统上线时必须确保有足够的空间，这对资源造成了很大的浪费。

## 数据类型

Redis主要有5种数据类型，包括String，List，Set，Zset，Hash

|数据类型|可以存储的值|操作|
|---|----------|---|
|STRING|字符串、整数或者浮点数|对整个字符串或者字符串的其中一部分执行操作对整数和浮点数执行自增或者自减操作|
|LIST|列表|从两端压入或者弹出元素对单个或者多个元素进行修剪，只保留一个范围内的元素|
|SET|无序集合|添加、获取、移除单个元素检查一个元素是否存在于集合中计算交集、并集、差集从集合里面随机获取元素|
|HASH|包含键值对的无序散列表|添加、获取、移除单个键值对获取所有键值对检查某个键是否存在|
|ZSET|有序集合|添加、获取、删除元素根据分值范围或者成员来获取元素计算一个键的排名|

## 使用场景
由于Redis优异的读写性能，持久化支持等优势，Redis的使用场景非常多，主要包括计数器，缓存，消息队列，分布式锁等

### 计数器
可以对 String 进行自增自减运算，从而实现计数器功能。

Redis 这种内存型数据库的读写性能非常高，很适合存储频繁读写的计数量。

### 缓存
可以使用 Redis 来统一存储多台应用服务器的会话信息。

当应用服务器不再存储用户的会话信息，也就不再具有状态，一个用户可以请求任意一个应用服务器，从而更容易实现高可用性以及可伸缩性。

### 全页缓存（FPC）
除基本的会话token之外，Redis还提供很简便的FPC平台。

以Magento为例，Magento提供一个插件来使用Redis作为全页缓存后端。此外，对WordPress的用户来说，Pantheon有一个非常好的插件 wp-redis，这个插件能帮助你以最快速度加载你曾浏览过的页面。

### 查找表
例如 DNS 记录就很适合使用 Redis 进行存储。

查找表和缓存类似，也是利用了 Redis 快速的查找特性。但是查找表的内容不能失效，而缓存的内容可以失效，因为缓存不作为可靠的数据来源。

### 消息队列(发布/订阅功能)
在分布式场景下，无法使用单机环境下的锁来对多个节点上的进程进行同步。

可以使用 Redis 自带的 SETNX 命令实现分布式锁，除此之外，还可以使用官方提供的 RedLock 分布式锁实现。

### 其它
Set 可以实现交集、并集等操作，从而实现共同好友等功能。

ZSet 可以实现有序性操作，从而实现排行榜等功能。

## 持久化
Redis 是内存型数据库，为了之后重用数据（比如重启机器、机器故障之后回复数据），或者是为了防止系统故障而将数据备份到一个远程位置，需要将内存中的数据持久化到硬盘上。

Redis 提供了RDB和AOF两种持久化方式。默认是只开启RDB，当Redis重启时，它会优先使用AOF文件来还原数据集。

### RDB 持久化(快照持久化)
RDB 持久化：将某个时间点的所有数据都存放到硬盘上。

可以将快照复制到其它服务器从而创建具有相同数据的服务器副本。如果系统发生故障，将会丢失最后一次创建快照之后的数据。如果数据量很大，保存快照的时间会很长。

快照持久化是Redis默认采用的持久化方式，在redis.conf配置文件中默认有此下配置：
```
#在900秒(15分钟)之后，如果至少有1个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
save 900 1              

#在300秒(5分钟)之后，如果至少有10个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
save 300 10            

#在60秒(1分钟)之后，如果至少有10000个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
save 60 10000        
```
根据配置，快照将被写入dbfilename选项指定的文件里面，并存储在dir选项指定的路径上面。如果在新的快照文件创建完毕之前，Redis、系统或者硬件这三者中的任意一个崩溃了，那么Redis将丢失最近一次创建快照写入的所有数据。

举个例子：假设Redis的上一个快照是2:35开始创建的，并且已经创建成功。下午3:06时，Redis又开始创建新的快照，并且在下午3:08快照创建完毕之前，有35个键进行了更新。如果在下午3:06到3:08期间，系统发生了崩溃，导致Redis无法完成新快照的创建工作，那么Redis将丢失下午2:35之后写入的所有数据。另一方面，如果系统恰好在新的快照文件创建完毕之后崩溃，那么Redis将丢失35个键的更新数据。

创建快照的办法有如下几种：

* BGSAVE命令 ：客户端向Redis发送 BGSAVE命令 来创建一个快照。对于支持BGSAVE命令的平台来说（基本上所有平台支持，除了Windows平台），Redis会调用fork来创建一个子进程，然后子进程负责将快照写入硬盘，而父进程则继续处理命令请求。

* SAVE命令 ：客户端还可以向Redis发送 SAVE命令 来创建一个快照，接到SAVE命令的Redis服务器在快照创建完毕之前不会再响应任何其他命令。SAVE命令不常用，我们通常只会在没有足够内存去执行BGSAVE命令的情况下，又或者即使等待持久化操作执行完毕也无所谓的情况下，才会使用这个命令。

* save选项 ：如果用户设置了save选项（一般会默认设置），比如 save 60 10000，那么从Redis最近一次创建快照之后开始算起，当“60秒之内有10000次写入”这个条件被满足时，Redis就会自动触发BGSAVE命令。

* SHUTDOWN命令 ：当Redis通过SHUTDOWN命令接收到关闭服务器的请求时，或者接收到标准TERM信号时，会执行一个SAVE命令，阻塞所有客户端，不再执行客户端发送的任何命令，并在SAVE命令执行完毕之后关闭服务器。

* 一个Redis服务器连接到另一个Redis服务器：当一个Redis服务器连接到另一个Redis服务器，并向对方发送SYNC命令来开始一次复制操作的时候，如果主服务器目前没有执行BGSAVE操作，或者主服务器并非刚刚执行完BGSAVE操作，那么主服务器就会执行BGSAVE命令

如果系统真的发生崩溃，用户将丢失最近一次生成快照之后更改的所有数据。因此，快照持久化只适用于即使丢失一部分数据也不会造成一些大问题的应用程序。不能接受这个缺点的话，可以考虑AOF持久化。

### AOF 持久化

AOF 持久化：将写命令添加到 AOF 文件（Append Only File）的末尾。

与快照持久化相比，AOF持久化 的实时性更好，因此已成为主流的持久化方案。默认情况下Redis没有开启AOF（append only file）方式的持久化，可以通过appendonly参数开启：
```
appendonly yes
```
开启AOF持久化后每执行一条会更改Redis中的数据的命令，Redis就会将该命令写入硬盘中的AOF文件。AOF文件的保存位置和RDB文件的位置相同，都是通过dir参数设置的，默认的文件名是appendonly.aof。

使用 AOF 持久化需要设置同步选项，从而确定写命令同步到磁盘文件上的时机。这是因为对文件进行写入并不会马上将内容同步到磁盘上，而是先存储到缓冲区，然后由操作系统决定什么时候同步到磁盘。在Redis的配置文件中存在三种同步方式

|选项|同步频率|
|---|-------|
|always|每个写命令都同步，这样会严重降低Redis的速度|
|everysec|每秒同步一次|
|no|让操作系统来决定何时同步|

appendfsync always 可以实现将数据丢失减到最少，不过这种方式需要对硬盘进行大量的写入而且每次只写入一个命令，十分影响Redis的速度。另外使用固态硬盘的用户谨慎使用appendfsync always选项，因为这会明显降低固态硬盘的使用寿命。

appendfsync everysec 为了兼顾数据和写入性能，用户可以考虑 appendfsync everysec选项 ，让Redis每秒同步一次AOF文件，Redis性能几乎没受到任何影响。而且这样即使出现系统崩溃，用户最多只会丢失一秒之内产生的数据。当硬盘忙于执行写入操作的时候，Redis还会优雅的放慢自己的速度以便适应硬盘的最大写入速度。

appendfsync no 选项一般不推荐，这种方案会使Redis丢失不定量的数据而且如果用户的硬盘处理写入操作的速度不够的话，那么当缓冲区被等待写入的数据填满时，Redis的写入操作将被阻塞，这会导致Redis的请求速度变慢。

随着服务器写请求的增多，AOF 文件会越来越大。Redis 提供了一种将 AOF 重写的特性，能够去除 AOF 文件中的冗余写命令。

虽然AOF持久化非常灵活地提供了多种不同的选项来满足不同应用程序对数据安全的不同要求，但AOF持久化也有缺陷——AOF文件的体积太大。

### 重写/压缩AOF

AOF虽然在某个角度可以将数据丢失降低到最小而且对性能影响也很小，但是极端的情况下，体积不断增大的AOF文件很可能会用完硬盘空间。另外，如果AOF体积过大，那么还原操作执行时间就可能会非常长。

为了解决AOF体积过大的问题，用户可以向Redis发送 BGREWRITEAOF命令 ，这个命令会通过移除AOF文件中的冗余命令来重写（rewrite）AOF文件来减小AOF文件的体积。BGREWRITEAOF命令和BGSAVE创建快照原理十分相似，所以AOF文件重写也需要用到子进程，这样会导致性能问题和内存占用问题，和快照持久化一样。更糟糕的是，如果不加以控制的话，AOF文件的体积可能会比快照文件大好几倍。

文件重写流程：


和快照持久化可以通过设置save选项来自动执行BGSAVE一样，AOF持久化设置以下参数

```
auto-aof-rewrite-percentage 和 auto-aof-rewrite-min-size
```

选项自动执行BGREWRITEAOF命令。举例：假设用户对Redis设置了如下配置选项并且启用了AOF持久化。那么当AOF文件体积大于64mb，并且AOF的体积比上一次重写之后的体积大了至少一倍（100%）的时候，Redis将执行BGREWRITEAOF命令。
```
auto-aof-rewrite-percentage 100  
auto-aof-rewrite-min-size 64mb
```

无论是AOF持久化还是快照持久化，将数据持久化到硬盘上都是非常有必要的，但除了进行持久化外，用户还必须对持久化得到的文件进行备份（最好是备份到不同的地方），这样才能尽量避免数据丢失事故发生。如果条件允许的话，最好能将快照文件和重新重写的AOF文件备份到不同的服务器上面。

随着负载量的上升，或者数据的完整性变得越来越重要时，用户可能需要使用到复制特性。

### Redis 4.0 对持久化机制的优化
Redis 4.0 开始支持 RDB 和 AOF 的混合持久化（默认关闭，可以通过配置项 aof-use-rdb-preamble 开启）。

如果把混合持久化打开，AOF 重写的时候就直接把 RDB 的内容写到 AOF 文件开头。这样做的好处是可以结合 RDB 和 AOF 的优点, 快速加载同时避免丢失过多的数据。当然缺点也是有的， AOF 里面的 RDB 部分就是压缩格式不再是 AOF 格式，可读性较差。

### 如何选择合适的持久化方式
* 一般来说， 如果想达到足以媲美PostgreSQL的数据安全性，你应该同时使用两种持久化功能。在这种情况下，当 Redis 重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。

* 如果你非常关心你的数据， 但仍然可以承受数分钟以内的数据丢失，那么你可以只使用RDB持久化。

* 有很多用户都只使用AOF持久化，但并不推荐这种方式，因为定时生成RDB快照（snapshot）非常便于进行数据库备份， 并且 RDB 恢复数据集的速度也要比AOF恢复的速度要快，除此之外，使用RDB还可以避免AOF程序的bug。

* 如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化方式。

## 过期键的删除策略
Redis中有个设置时间过期的功能，即对存储在 redis 数据库中的值可以设置一个过期时间。
作为一个缓存数据库，这是非常实用的。如我们一般项目中的 token 或者一些登录信息，
尤其是短信验证码都是有时间限制的，按照传统的数据库处理方式，一般都是自己判断过期，这样无疑会严重影响项目性能。

我们 set key 的时候，都可以给一个 expire time，就是过期时间，通过过期时间我们可以指定这个 key 可以存活的时间。

`注：对于散列表这种容器，只能为整个键设置过期时间（整个散列表），而不能为键里面的单个元素设置过期时间。`

如果一个键是过期的，那它到了过期时间之后是不是马上就从内存中被被删除呢？如果不是，那过期后到底什么时候被删除呢？

其实有三种不同的删除策略：
* 立即删除。在设置键的过期时间时，创建一个回调事件，当过期时间达到时，由时间处理器自动执行键的删除操作。
* 惰性删除。键过期了就过期了，不管。每次从dict字典中按key取值时，先检查此key是否已经过期，如果过期了就删除它，并返回nil，如果没过期，就返回键值。
* 定时删除。每隔一段时间，对expires字典进行检查，删除里面的过期键。

可以看到，第二种为被动删除，第一种和第三种为主动删除，且第一种实时性更高。下面对这三种删除策略进行具体分析。

### 立即删除
立即删除能保证内存中数据的最大新鲜度，因为它保证过期键值会在过期后马上被删除，其所占用的内存也会随之释放。但是立即删除对cpu是最不友好的。因为删除操作会占用cpu的时间，如果刚好碰上了cpu很忙的时候，比如正在做交集或排序等计算的时候，就会给cpu造成额外的压力。

而且目前redis事件处理器对时间事件的处理方式–无序链表，查找一个key的时间复杂度为O(n),所以并不适合用来处理大量的时间事件。

### 惰性删除
惰性删除是指，某个键值过期后，此键值不会马上被删除，而是等到下次被使用的时候，才会被检查到过期，此时才能得到删除。所以惰性删除的缺点很明显:浪费内存。dict字典和expires字典都要保存这个键值的信息。

举个例子，对于一些按时间点来更新的数据，比如log日志，过期后在很长的一段时间内可能都得不到访问，这样在这段时间内就要拜拜浪费这么多内存来存log。这对于性能非常依赖于内存大小的redis来说，是比较致命的。

### 定时删除

从上面分析来看，立即删除会短时间内占用大量cpu，惰性删除会在一段时间内浪费内存，所以定时删除是一个折中的办法。
定时删除是：每隔一段时间执行一次删除操作，并通过限制删除操作执行的时长和频率，来减少删除操作对cpu的影响。另一方面定时删除也有效的减少了因惰性删除带来的内存浪费。

### Redis使用的策略
redis使用的过期键值删除策略是：惰性删除加上定期删除，两者配合使用。

但是仅仅通过设置过期时间还是有问题的。我们想一下：如果定期删除漏掉了很多过期 key，然后你也没及时去查，也就没走惰性删除，此时会怎么样？如果大量过期key堆积在内存里，导致redis内存块耗尽了。怎么解决这个问题呢？ redis 数据淘汰策略。

## 数据淘汰策略
可以设置内存最大使用量，当内存使用量超出时，会施行数据淘汰策略。

如果Redis中数据非常多，将服务器中的内存都耗尽，这样就会出现内存溢出的情况，Redis开发组考虑到了这种问题，使用数据淘汰策略可以解决这个问题。

可以设置内存最大使用量，当内存使用量超出时，会施行数据淘汰策略。

Redis 具体有 6 种淘汰策略：

|策略|描述|应用场景|
|---|---|-------|
|volatile-lru|从已设置过期时间的数据集中挑选最近最少使用的数据淘汰|如果设置了过期时间，且分热数据与冷数据，推荐使用 volatile-lru 策略。|
|volatile-ttl|从已设置过期时间的数据集中挑选将要过期的数据淘汰|如果让 Redis 根据 TTL 来筛选需要删除的key，请使用 volatile-ttl 策略。|
|volatile-random|从已设置过期时间的数据集中任意选择数据淘汰|很少使用|
|allkeys-lru|从所有数据集中挑选最近最少使用的数据淘汰|使用 Redis 缓存数据时，为了提高缓存命中率，需要保证缓存数据都是热点数据。可以将内存最大使用量设置为热点数据占用的内存量，然后启用 allkeys-lru 淘汰策略，将最近最少使用的数据淘汰。值得一提的是，设置 expire 会消耗额外的内存，所以使用 allkeys-lru 策略，可以更高效地利用内存，因为这样就可以不再设置过期时间了。|
|allkeys-random|从所有数据集中任意选择数据进行淘汰|如果需要循环读写所有的key，或者各个key的访问频率差不多，可以使用 allkeys-random 策略|
|noeviction|不删除策略，达到最大内存限制时，如果需要更多内存，直接返回错误信息。大多数写命令都会导致占用更多的内存|很少使用|

作为内存数据库，出于对性能和内存消耗的考虑，Redis 的淘汰算法实际实现上并非针对所有 key，而是抽样一小部分并且从中选出被淘汰的 key。

Redis 4.0 引入了 volatile-lfu 和 allkeys-lfu 淘汰策略，LFU 策略通过统计访问频率，将访问频率最少的键值对淘汰。

您需要根据系统的特征，来选择合适的淘汰策略。 当然，在运行过程中也可以通过命令动态设置淘汰策略，并通过 INFO 命令监控缓存的 miss 和 hit，来进行调优。

### 淘汰策略的内部实现

* 客户端执行一个命令，导致 Redis 中的数据增加，占用更多内存
* Redis 检查内存使用量，如果超出 maxmemory 限制，根据策略清除部分 key
* 继续执行下一条命令，以此类推
* 在这个过程中，内存使用量会不断地达到 limit 值，然后超过，然后删除部分 key，使用量又下降到 limit 值之下。

如果某个命令导致大量内存占用(比如通过新key保存一个很大的set)，在一段时间内，可能内存的使用量会明显超过 maxmemory 限制。

## Redis与Memcached的区别
两者都是非关系型内存键值数据库，现在公司一般都是用 Redis 来实现缓存，而且 Redis 自身也越来越强大了！Redis 与 Memcached 主要有以下不同：

|对比参数|Redis|Memcached|
|------|-----|---------|
|类型|1. 支持内存 2. 非关系型数据库|1. 支持内存 2. 键值对形式 3. 缓存形式|
|数据存储类型|1. String 2. List 3. Set 4. Hash 5. Sort Set 【俗称ZSet】|1. 文本型 2. 二进制类型|
|查询【操作】类型|1. 批量操作 2. 事务支持 3. 每个类型不同的CRUD|1.常用的CRUD 2. 少量的其他命令|
|附加功能|1. 发布/订阅模式 2. 主从分区 3. 序列化支持 4. 脚本支持【Lua脚本】|1. 多线程服务支持|
|网络IO模型|1. 单线程的多路 IO 复用模型|1. 多线程，非阻塞IO模式|
|事件库|自封转简易事件库AeEvent	|贵族血统的LibEvent事件库|
|持久化支持|1. RDB 2. AOF|不支持|
|集群模式|原生支持 cluster 模式|没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据|
|内存管理机制|在 Redis 中，并不是所有数据都一直存储在内存中，可以将一些很久没用的 value 交换到磁盘|Memcached 的数据则会一直在内存中，Memcached 将内存分割成特定长度的块来存储数据，以完全解决内存碎片的问题。但是这种方式会使得内存的利用率不高，例如块的大小为 128 bytes，只存储 100 bytes 的数据，那么剩下的 28 bytes 就浪费掉了。|

## 事务
Redis 通过 MULTI、EXEC、WATCH 等命令来实现事务(transaction)功能。事务提供了一种将多个命令请求打包，然后一次性、按顺序地执行多个命令的机制，并且在事务执行期间，服务器不会中断事务而改去执行其他客户端的命令请求，它会将事务中的所有命令都执行完毕，然后才去处理其他客户端的命令请求。

事务中的多个命令被一次性发送给服务器，而不是一条一条发送，这种方式被称为流水线，可以减少客户端与服务器之间的网络通信次数从而提升性能。

在传统的关系式数据库中，常用 ACID 性质来检验事务功能的可靠性和安全性。在 Redis 中，事务总是具有原子性（Atomicity）、一致性（Consistency）和隔离性（Isolation），并且当 Redis 运行在某种特定的持久化模式下时，事务也具有持久性（Durability）。

## 事件
Redis 服务器是一个事件驱动程序。

## 文件事件
服务器通过套接字与客户端或者其它服务器进行通信，文件事件就是对套接字操作的抽象。

Redis 基于 Reactor 模式开发了自己的网络事件处理器，使用 I/O 多路复用程序来同时监听多个套接字，并将到达的事件传送给文件事件分派器，分派器会根据套接字产生的事件类型调用相应的事件处理器。

## 时间事件
服务器有一些操作需要在给定的时间点执行，时间事件是对这类定时操作的抽象。

时间事件又分为：

* 定时事件：是让一段程序在指定的时间之内执行一次
* 周期性事件：是让一段程序每隔指定时间就执行一次

目前Redis只使用周期性事件，而没有使用定时事件。 一个事件时间主要由三个属性组成：

* id：服务器为时间事件创建的全局唯一ID
* when：毫秒精度的UNIX时间戳，记录了时间事件的到达时间
* timeProc：时间事件处理器，一个函数

实现服务器将所有时间事件都放在一个无序链表中，每当时间事件执行器运行时，遍历整个链表，查找所有已到达的时间事件，并调用相应的事件处理器。（该链表为无序链表，不按when属性的大小排序）

## 事件的调度与执行
服务器需要不断监听文件事件的套接字才能得到待处理的文件事件，但是不能一直监听，否则时间事件无法在规定的时间内执行，因此监听时间应该根据距离现在最近的时间事件来决定。

## Sentinel
Sentinel（哨兵）可以监听集群中的服务器，并在主服务器进入下线状态时，自动从从服务器中选举出新的主服务器。

## 分片
分片是将数据划分为多个部分的方法，可以将数据存储到多台机器里面，这种方法在解决某些问题时可以获得线性级别的性能提升。

假设有 4 个 Redis 实例 R0，R1，R2，R3，还有很多表示用户的键 user:1，user:2，… ，有不同的方式来选择一个指定的键存储在哪个实例中。

* 最简单的方式是范围分片，例如用户 id 从 0~1000 的存储到实例 R0 中，用户 id 从 1001~2000 的存储到实例 R1 中，等等。但是这样需要维护一张映射范围表，维护操作代价很高。
* 还有一种方式是哈希分片，使用 CRC32 哈希函数将键转换为一个数字，再对实例数量求模就能知道应该存储的实例。

根据执行分片的位置，可以分为三种分片方式：
* 客户端分片：客户端使用一致性哈希等算法决定键应当分布到哪个节点。
* 代理分片：将客户端请求发送到代理上，由代理转发请求到正确的节点上。
* 服务器分片：Redis Cluster。

## 复制
通过使用 slaveof host port 命令来让一个服务器成为另一个服务器的从服务器。

一个从服务器只能有一个主服务器，并且不支持主主复制。

## 连接过程
* 主服务器创建快照文件，发送给从服务器，并在发送期间使用缓冲区记录执行的写命令。快照文件发送完毕之后，开始向从服务器发送存储在缓冲区中的写命令
* 从服务器丢弃所有旧数据，载入主服务器发来的快照文件，之后从服务器开始接受主服务器发来的写命令
* 主服务器每执行一次写命令，就向从服务器发送相同的写命令

## 主从链
随着负载不断上升，主服务器可能无法很快地更新所有从服务器，或者重新连接和重新同步从服务器将导致系统超载。为了解决这个问题，可以创建一个中间层来分担主服务器的复制工作。中间层的服务器是最上层服务器的从服务器，又是最下层服务器的主服务器。

## Redis中缓存雪崩、缓存穿透等问题的解决方案

### 缓存雪崩
缓存雪崩是指缓存同一时间大面积的失效，所以，后面的请求都会落到数据库上，造成数据库短时间内承受大量请求而崩掉。

解决方案

* 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
* 一般并发量不是特别多的时候，使用最多的解决方案是加锁排队。
* 给每一个缓存数据增加相应的缓存标记，记录缓存的是否失效，如果缓存标记失效，则更新数据缓存。

### 缓存穿透
缓存穿透是指缓存和数据库中都没有的数据，导致所有的请求都落到数据库上，造成数据库短时间内承受大量请求而崩掉。

解决方案

* 接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
* 从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击
* 采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉，从而避免了对底层存储系统的查询压力
附加

对于空间的利用到达了一种极致，那就是Bitmap和布隆过滤器(Bloom Filter)。
Bitmap： 典型的就是哈希表
缺点是，Bitmap对于每个元素只能记录1bit信息，如果还想完成额外的功能，恐怕只能靠牺牲更多的空间、时间来完成了。

布隆过滤器（推荐）

就是引入了k(k>1)k(k>1)个相互独立的哈希函数，保证在给定的空间、误判率下，完成元素判重的过程。
它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。
Bloom-Filter算法的核心思想就是利用多个不同的Hash函数来解决“冲突”。
Hash存在一个冲突（碰撞）的问题，用同一个Hash得到的两个URL的值有可能相同。为了减少冲突，我们可以多引入几个Hash，如果通过其中的一个Hash值我们得出某元素不在集合中，那么该元素肯定不在集合中。只有在所有的Hash函数告诉我们该元素在集合中时，才能确定该元素存在于集合中。这便是Bloom-Filter的基本思想。
Bloom-Filter一般用于在大数据量的集合中判定某元素是否存在。

### 缓存击穿
缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。和缓存雪崩不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。

解决方案

* 设置热点数据永远不过期。
* 加互斥锁，互斥锁

### 缓存预热
缓存预热就是系统上线后，将相关的缓存数据直接加载到缓存系统。这样就可以避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题！用户直接查询事先被预热的缓存数据！

解决方案

* 直接写个缓存刷新页面，上线时手工操作一下；

* 数据量不大，可以在项目启动的时候自动进行加载；

* 定时刷新缓存；

### 缓存降级
当访问量剧增、服务出现问题（如响应时间慢或不响应）或非核心服务影响到核心流程的性能时，仍然需要保证服务还是可用的，即使是有损服务。系统可以根据一些关键数据进行自动降级，也可以配置开关实现人工降级。

缓存降级的最终目的是保证核心服务可用，即使是有损的。而且有些服务是无法降级的（如加入购物车、结算）。

在进行降级之前要对系统进行梳理，看看系统是不是可以丢卒保帅；从而梳理出哪些必须誓死保护，哪些可降级；比如可以参考日志级别设置预案：

* 一般：比如有些服务偶尔因为网络抖动或者服务正在上线而超时，可以自动降级；

* 警告：有些服务在一段时间内成功率有波动（如在95~100%之间），可以自动降级或人工降级，并发送告警；

* 错误：比如可用率低于90%，或者数据库连接池被打爆了，或者访问量突然猛增到系统能承受的最大阀值，此时可以根据情况自动降级或者人工降级；

* 严重错误：比如因为特殊原因数据错误了，此时需要紧急人工降级。

服务降级的目的，是为了防止Redis服务故障，导致数据库跟着一起发生雪崩问题。因此，对于不重要的缓存数据，可以采取服务降级策略，例如一个比较常见的做法就是，Redis出现问题，不去数据库查询，而是直接返回默认值给用户。

### 热点数据和冷数据
热点数据，缓存才有价值

对于冷数据而言，大部分数据可能还没有再次访问到就已经被挤出内存，不仅占用内存，而且价值不大。频繁修改的数据，看情况考虑使用缓存

对于热点数据，比如我们的某IM产品，生日祝福模块，当天的寿星列表，缓存以后可能读取数十万次。再举个例子，某导航产品，我们将导航信息，缓存以后可能读取数百万次。

数据更新前至少读取两次，缓存才有意义。这个是最基本的策略，如果缓存还没有起作用就失效了，那就没有太大价值了。

那存不存在，修改频率很高，但是又不得不考虑缓存的场景呢？有！比如，这个读取接口对数据库的压力很大，但是又是热点数据，这个时候就需要考虑通过缓存手段，减少数据库的压力，比如我们的某助手产品的，点赞数，收藏数，分享数等是非常典型的热点数据，但是又不断变化，此时就需要将数据同步保存到Redis缓存，减少数据库压力。

### 缓存热点key
缓存中的一个Key(比如一个促销商品)，在某个时间点过期的时候，恰好在这个时间点对这个Key有大量的并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

解决方案

对缓存查询加锁，如果KEY不存在，就加锁，然后查DB入缓存，然后解锁；其他进程如果发现有锁就等待，然后等解锁后返回数据或者进入DB查询