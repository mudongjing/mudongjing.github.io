> 本文主要描述Redis大致的使用方式，为读者对Redis的使用有一定的全局观。更为细节的内容，可通过查询相关书籍或博客深入了解。

> 由于Redis很好地支持了大多数流行的编程语言，实际上大多在各自的程序中对Redis进行操作，因此，在了解Reids本身的操作后，另外需要花费少量时间了解对应语言对应的库的使用

Redis为C语言的开源项目，若是已经对Redis又深入的了解，可通过阅读[Redis源码](https://github.com/redis/redis "Redis源码")获得更深层次的认识，或根据自身需求做修改（自用可以，但实际工作不建议随意修改）。

### 1 基础知识
#### 1.1 登录
redis的命令不区分大小写。但之后的值匹配中需要注意大小写。
启动可直接在终端输入 `redis-server`。或者通过脚本运行，该脚本在redis安装目录下的 `utils`文件夹中，名为 `redis_init_script`，在文件中可修改对应的配置信息，并重命名复制目录/etc/init.d，通过该脚本可启动一个redis实例，通过下方命令可设置开机启动
```shell
sudo update-rc.d 文件名 defaults
```
当复制多个文件，并修改为不同的端口号后，便可同时运行多个redis实例。默认的端口号为6379。
启动后，首先登录redis:
```shell
[root@ ~]# redis-cli
127.0.0.1:6379> auth 密码
OK
```
终端输入**redis-cli**，开启redis，并输入自己的密码，否则无法操作。
登录远程redis服务器：
```shell
redis-cli -h IP -p 端口号
```
需要停止时：
```shell
redis-cli shutdown
```
#### 1.2 插入与查询
Redis系统中，不同于MySQL之类的关系型数据库，不仅数据保存在内存中，且数据记录以字典结构保存（MySQL中的存储结构多为B+树的形式，但根据引擎的不同有所区别）。
在MySQL中可以创建多个自定义命名的数据库，并在其中创建多个表，在表内存放一行行的数据。

- 相反的，Redis默认的数据库为16个，且无法命名，只能使用从0开始的序号表示，在Redis的配置文件中可以自定义数据库的数量，配置参数为 **database**。

- 在Redis中没有表的概念，只有键，通过查询相对应的键，可获取键值的内容。

为了记录信息，首先需选择对应的数据库，在数据库中创建新的键，再按照对应的方法插入数据。默认在登录的时候处于0数据库。

```shell
select 数据库序号
```
查询数据库中存在的键，使用命令 `keys`，后跟匹配模式，例如 **keys \*** 显示所有键。

- 由于键值的不同类型导致键的创建有所不同，下面给出不同类型的创建方式

| 类型     | 命令                                                         |
| -------- | ------------------------------------------------------------ |
| 字符串   | set 键名 键值                                                |
| 集合     | sadd 键名 键值                                               |
| 散列表   | hset 键名 域名 域值 <br>或<br> hmset 键名 域名 域值 域名 域值 ... |
| 有序集合 | zadd 键名 成员 分值                                          |
| 列表     | lpush 或 rpush （从左边或右边推入新值）                      |

- 上述命令不仅用于创建新键，也可以为已有键添加新值
- 为获取键中的数据，根据不同的类型同样有着不同的命令。

| 类型     | 命令                                                         |
| -------- | ------------------------------------------------------------ |
| 字符串   | get 键名                                                     |
| 集合     | smembers 键名                                                |
| 散列表   | hget 键名 域名 <br> 或<br> hgetall 键名（返回域名与对应的域值）<br> 或<br> hkeys 键名（返回所有的域键） |
| 有序集合 | zrange 键名 起始位置 结束位置 <br>或 <br> zrangebyscores 键名 分值下限 分值上限 <br> \[withscores\] |
| 列表     | lrange 键名 起始位置 结束位置<br> 或<br> lindex 键名 位置    |

- 为获取各个键中的元素长度，也需要不同的命令。
| 类型     | 命令   |
| -------- | ------ |
| 字符串   | strlen |
| 集合     | scard  |
| 散列表   | hlen   |
| 有序集合 | zcard  |
| 列表     | llen   |

#### 1.3 删除与判断
通过上述的介绍，已经明显看出由于类型的不同导致命令的区别。同样的，对元素的删除，也需要不同的命令，以及判断键值中是否存在给定的值。
删除操作：

| 类型     | 命令                                      | 判断命令              |
| -------- | ----------------------------------------- | --------------------- |
| 字符串   | del 键名                                  |                       |
| 集合     | srem 键名 给定值                          | sismeber 键名 给定值  |
| 散列表   | hdel 键名 域名                            | hexists 键名 给定域名 |
| 有序集合 | zrem 键名 成员名                          |                       |
| 列表     | lpop 或 rpop <br>（从左边或右边弹出元素） |                       |

删除数据库中的键统一用 `del`操作
> 但redis中只能一个个地删除，一旦需要删除一堆长相类似的键则需要结合Linux的命令
```shell
redis-cli -a 密码 -n 数据库序号 keys '匹配模式' | xargs redis-cli -a 密码 -n 数据库序号 del
```
最后，可用命令 `dbsize`查询当前数据库中键的数量。

#### 1.4 数据操作
##### 1.4.1 加减

|类型|加|减|
|--|--|
|字符串|incr 键名（加1）<br>或<br> incrby 键名 给定增量 <br> 或 <br> incrbyfloat 给定增量 <br> （对数值型的字符串操作）|decr 或 decrby|
|集合|||
|散列表|hincrby 或 hincrbyfloat||
|有序集合|zincrby ||
|列表|||

其中集合还包含交并集操作

| 操作     | 并集                                       | 交集        | 差集       |
| -------- | ------------------------------------------ | ----------- | ---------- |
| 简单命令 | sunion                                     | sinter      | sdiff      |
| 结果保存 | sunionstore 新集合键名<br> 键名1 键名2 ... | sinterstore | sdiffstore |

##### 1.4.2 期限
由于Redis的数据保存在内存中，企业大多用其存储热点信息。但考虑到内存容量和工作效率，需要对旧数据作清理，于是需要对数据设定一个存活时长，令数据在经历或到达一个时间后自发删除。

| 命令                 | 操作                 | 描述                                     |
| -------------------- | -------------------- | ---------------------------------------- |
| expire               | expire 键名 秒数     | 经历过对应的秒数后，该键删除             |
| expireat             | expireat 键名 时间戳 | 参数为UNIX时间戳，到达该时间后，删除该键 |
| pexpire<br>pexpireat | 与上面命令操作类似   | 时间参数以毫秒为单位                     |
| TTL                  | TTL 键名             | 返回该键的剩余时长                       |
| persist              | persist 键名         | 取消键的期限设置                         |

上述介绍了如何手动设置键的存活时长。但在实际中，例如微博或其它社交平台，总是会出现新的热点，而这些热点又很难确定何时消退热度，但为了保证内存不至于被这类所谓的热点塞满，需要系统自行删除一些数据。于是，就有了缓存策略。

- 在配置文件中，修改 **maxmemory**参数，限制Redis可用的最大内存。修改 **maxmemory-policy** 参数，告诉Redis一旦使用内存到达给定的限制，将按照何种规则删除数据。

规则如下：

| 规则            | 说明                                |
| --------------- | ----------------------------------- |
| volatile-lru    | 对设置了期限的键按照LRU算法删除一个 |
| allkeys-lru     | 对任意键按照LRU删除一个             |
| volatile-random | 对具有期限的键随机删除一个          |
| allkeys-random  | 随机删除一个键                      |
| volatile-ttl    | 删除过期时间最近的一个键            |
| noeviction      | 不删除，只返回错误                  |

其中，**LRU**(Least Recent Used)意为**最近最少使用**。
##### 1.4.3 排序

- 排序使用 `sort`进行操作。 `sort`支持列表、集合和有序集合的排序。且以对应的键值为基础排序，默认从小到大，且默认排序对象为数字。

**sort**的完整命令如下：
```shell
SORT key [BY pattern] [LIMIT start count] [GET pattern] [ASC|DESC] [ALPHA] [STORE dstkey] 
```
在介绍**\[BY pattern\]**之前，需要准备一个例子，
数据库中存在三种键，

| 类型   | 内容     | 组成                                                         |
| ------ | -------- | ------------------------------------------------------------ |
| 列表   | 用户序号 | uid：\[1，2，3，4\]                                          |
| 字符串 | 用户名   | user_name_1:admin<br> user_name_2:jack<br> user_name_3:pete<br> user_name_4:mary |
| 字符串 | 用户等级 | user_level_1:9999<br>user_level_2:10<br>user_level_3:25<br>user_level_4:70 |

这三种键实际是分类保存了用户的信息，而 **uid**常用来表示实际的用户。但为了获取该用户的其它信息，则需要与其它键建立关系，联系的方式，则是通过模式匹配键值

- BY
比如说， 以下代码让 uid 键按照 user_level_{uid} 的大小来排序：

```shell
redis 127.0.0.1:6379> SORT uid BY user_level_*
1) "2"      # jack , level = 10
2) "3"      # peter, level = 25
3) "4"      # mary, level = 70
4) "1"      # admin, level = 9999
```
user_level_* 是一个占位符， 它先取出 uid 中的值， 然后再用这个值来查找相应的键。

比如在对 uid 列表进行排序时， 程序就会先取出 uid 的值 1 、 2 、 3 、 4 ， 然后使用 user_level_1 、 user_level_2 、 user_level_3 和 user_level_4 的值作为排序 uid 的权重。

- LIMIT
限定返回结果的数量

- GET
使用 GET 选项， 可以根据排序的结果来取出相应的键值。

比如说， 以下代码先排序 uid ， 再取出键 user_name_{uid} 的值：

```shell
redis 127.0.0.1:6379> SORT uid GET user_name_*
1) "admin"
2) "jack"
3) "peter"
4) "mary"
```
**组合使用 BY 和 GET**
通过组合使用 BY 和 GET ， 可以让排序结果以更直观的方式显示出来。

比如说， 以下代码先按 user_level_{uid} 来排序 uid 列表， 再取出相应的 user_name_{uid} 的值：

```shell
redis 127.0.0.1:6379> SORT uid BY user_level_* GET user_name_*
1) "jack"       # level = 10
2) "peter"      # level = 25
3) "mary"       # level = 70
4) "admin"      # level = 9999
```

**获取多个外部键**
可以同时使用多个 GET 选项， 获取多个外部键的值。

以下代码就按 uid 分别获取 user_level_{uid} 和 user_name_{uid} ：

```shell
redis 127.0.0.1:6379> SORT uid GET user_level_* GET user_name_*
1) "9999"       # level
2) "admin"      # name
3) "10"
4) "jack"
5) "25"
6) "peter"
7) "70"
8) "mary"
```
GET 有一个额外的参数规则，可以用 # 获取被排序键的值。

以下代码就将 uid 的值、及其相应的 user_level_* 和 user_name_* 都返回为结果：

```shell
redis 127.0.0.1:6379> SORT uid GET # GET user_level_* GET user_name_*
1) "1"          # uid
2) "9999"       # level
3) "admin"      # name
4) "2"
5) "10"
6) "jack"
7) "3"
8) "25"
9) "peter"
10) "4"
11) "70"
12) "mary"
```
获取外部键，但不进行排序
通过将一个不存在的键作为参数传给 BY 选项， 可以让 SORT 跳过排序操作， 直接返回结果：

```shell
redis 127.0.0.1:6379> SORT uid BY not-exists-key
1) "4"
2) "3"
3) "2"
4) "1"
```
这种用法在单独使用时，没什么实际用处。

不过，通过将这种用法和 GET 选项配合， 就可以在不排序的情况下， 获取多个外部键， 相当于执行一个整合的获取操作（类似于 SQL 数据库的 join 关键字）。

以下代码演示了，如何在不引起排序的情况下，使用 SORT 、 BY 和 GET 获取多个外部键：

```shell
redis 127.0.0.1:6379> SORT uid BY not-exists-key GET # GET user_level_* GET user_name_*
1) "4"      # id
2) "70"     # level
3) "mary"   # name
4) "3"
5) "25"
6) "peter"
7) "2"
8) "10"
9) "jack"
10) "1"
11) "9999"
12) "admin"
```
**将哈希表作为 GET 或 BY 的参数**
除了可以将字符串键之外， 哈希表也可以作为 GET 或 BY 选项的参数来使用。

比如说，对于前面给出的用户信息表：

我们可以不将用户的名字和级别保存在 user_name_{uid} 和 user_level_{uid} 两个字符串键中， 而是用一个带有 name 域和 level 域的哈希表 user_info_{uid} 来保存用户的名字和级别信息：

```shell
redis 127.0.0.1:6379> HMSET user_info_1 name admin level 9999
OK

redis 127.0.0.1:6379> HMSET user_info_2 name jack level 10
OK

redis 127.0.0.1:6379> HMSET user_info_3 name peter level 25
OK

redis 127.0.0.1:6379> HMSET user_info_4 name mary level 70
OK
```
之后， BY 和 GET 选项都可以用 key->field 的格式来获取哈希表中的域的值， 其中 key 表示哈希表键， 而 field 则表示哈希表的域：

```shell
redis 127.0.0.1:6379> SORT uid BY user_info_*->level
1) "2"
2) "3"
3) "4"
4) "1"

redis 127.0.0.1:6379> SORT uid BY user_info_*->level GET user_info_*->name
1) "jack"
2) "peter"
3) "mary"
4) "admin"
```
- \[ASC|DESC\]
给定排序顺序

- ALPHA
当需要对字符串进行排序时， 需要显式地在 SORT 命令之后添加 ALPHA 修饰符：

- STORE dstkey
默认情况下， SORT 操作只是简单地返回排序结果，并不进行任何保存操作。

通过给 STORE 选项指定一个 key 参数，可以将排序结果保存到给定的键上。

如果被指定的 key 已存在，那么原有的值将被排序结果覆盖。

可以通过将 SORT 命令的执行结果保存，并用 EXPIRE key seconds 为结果设置生存时间，以此来产生一个 SORT 操作的结果缓存。

这样就可以避免对 SORT 操作的频繁调用：只有当结果集过期时，才需要再调用一次 SORT 操作。
### 2 事务操作
#### 2.1 multi和exec
与常用数据库类似，事务指一组命令需要被原子执行。但Redis相对于MySQL之类的事务实现简单，也不强大，最重要的是Redis没有回滚操作。

- multi命令，表示事务的开始，在之后输入一系列操作命令
- exec命令，代表事务的结束，之前的命令此时开始执行
- 若需要在写事务的中途退出，可输入命令 discard

一旦在执行中，某个命令发生错误，则事务中断，但之前已做出的改变无法恢复，需要操作者自行修复。
#### 2.2 管道pipeline
若需要在远程服务器上操作时，需要通过TCP协议建立联系，但是正常的通信总是需要信号不断往返，浪费时间，由此出现**pipeline**以实现单方面不间断发送命令。

- 首先，建议在程序语言中操作pipeline()命令，在Redis中操作非常繁琐
- 若是需要输入事务命令，`pipeline()`的参数不输入或输入 `True`，则服务器认为接下来为事务，在命令队列中，将后续的命令包裹在 `multi`和 `exec`之间
- 若只是普通的命令，命令之间没有联系时，则 `pipeline()`的参数中输入 `False`
#### 2.3 watch
由于在实际工作中，Redis需要时刻处理大量的操作，若是在事务操作中，其它客户端传入命令将部分数据已修改，则事务操作大概率会出现异常。于是，需要判断一些我们关心的数据是否在这期间被改动，使用命令 `watch`，

```shell
watch 键名
```
在事务操作中，使用该命令相当于监视数据，一旦数据被改动，则事务中断。在执行到 `exec`后，会自动取消前面的监视，或用命令 `unwatch`取消。
#### 2.4 锁
`watch`作为乐观锁，并没有对修改数据的行为作阻止。下面将介绍Redis中悲观锁的使用。
同其它地方的锁的构造相同，通过规定某个变量为锁的条件，只有进程对该变量完成某一操作后，才能进行自己的行为，否则则不断尝试（可设置尝试的时长期限）。

- 构建锁。
在Redis中利用命令 `setnx`，是 "SET if Not eXists"的简写，即仅当该键不存在时可以创建，并返回1，否则返回0。
通过创建于删除键完成锁的获取与释放。

- 同样的，可以使用 `incr`、`set`等命令创建不同的锁的条件。
- 利用上述的锁机制，既可以针对键创建锁，也可以为键中的各个数值创建对应的锁，也称之为**细粒度锁**，通过将锁细化，能保证有更多的进程可以同时对数据库进行操作。
- 为防止进程奔溃导致锁未被释放而导致死锁现象，可对锁设置期限。
##### 2.4.1 计数信号量

通过计数信号量，可限制进程数量，不同于锁的操作，当进程无法获得锁，则不断尝试，而信号量无法获取，则直接返回失败结果。
为安全考虑，获得信号量的进程同样需要设置期限。

- 不公平信号量
假设所有机器的时间是相同的，则根据每个进程加入时的UNIX时间戳排名从中选择前几名获得信号量。
具体操作为，使用有序集合，每个进程加入时获取一个随机序列值作为成员，携带的时间戳作为分值，根据信号量的个数结合进程的排名选择对应的进程。在挑选时，首先删除有序集合中那些达到期限的进程，再进行排名，未被选中的则被删除。
如果机器的时间是不同的，那么有些进程天然地比其它进程具有优势。

- 公平信号量
不同于前面利用系统的时间戳，为了避免自身系统的问题。公平信号量为自己设置了一个计数器，利用自增操作描述前后顺序，每个进程加入时，对计数器进行操作，并得到计数器当时的值，作为一个时间标志。其它操作类似。

### 3 安全机制
由于内存一旦断电则数据消失，为保证数据安全 ，首先需要将数据在硬盘中保存备份，称之为持久化。
#### 3.1 持久化

##### 3.1.1 快照持久化
这种方式是直接将整个存储复制到硬盘中，并有两种命令：

| 命令   | 说明                      |
| ------ | ------------------------- |
| bgsave | Redis创建一个进程负责复制 |
| save   | Redis将暂时停止响应命令   |

1. 配置文件中设置`save 60 10000`，代表从最后一次创建快照算起，若60秒内有10000次写入，则触发bgsave命令。
2. 若是多个Redis服务器之间通信时，一方向对方发送 `sync`命令，对方若是最近没有执行快照操作，则执行`bgsave`命令。

##### 3.1.2 AOF持久化（Append Only File）
由于每次数据的变化都需要利用写命令进行修改，因此，只需要从头到尾保存所有的写命令即可，通过从头到尾执行一遍就可以获得对应的结果。
首先使用**AOF**功能需要在配置文件中修改**appendonly**的参数为yes，默认为no。
此外，还有对应的AOF缓冲区同步文件策略，由**appendfsync**参数控制，

| 配置参数 | 说明                                       |
| -------- | ------------------------------------------ |
| always   | 每个命令都需要同步写入硬盘（安全但效率低） |
| everysec | 每秒执行一次同步，显式地将写命令同步到硬盘 |
| no       | 不主动同步，由系统控制（通常30秒同步一次） |

由于AOF文件包含大量的记录，导致文件大小不断膨胀，可通过删减其中的部分记录，减小所需的存储空间，称为**重写**，为此也可以在配置文件中设置何时执行，

| 参数                        | 说明                                                     |
| --------------------------- | -------------------------------------------------------- |
| auto-aof-rewrite-percentage | 指明当文献大小与上次重写前的大小百分比达到指定值，后重写 |
| auto-aof-rewrite-min-size   | 文件大小达到指定值，则重写                               |

除了这类自动重写的配置外，还可以通过发送命令 `bgrewriteaof`手动执行重写。

**重写的实现**
实际的重写操作不会对原本的文件做分析，而是根据当前的情况，将当前的状况重新写一遍，并删除了之前对键的各种操作，直接用最后的结果代替。

#### 3.2 内存优化
所谓的内存优化，实际对数据结构的优化。Redis中的数据均以列表、集合等数据结构形式保存，这类结构有时为了搜索效率的考虑需要附带额外的信息，导致实际的存储大小远比实际的数据量要大。
为此，当一个数据结构中的数据量较小时，采用额外的数据结构保存，丧失部分的效率，减小存储量。当数据量增大到影响效率后，再恢复到正常的结构。

基于上述的介绍，Redis中对于各个类型值的编码存在多种内部编码，根据用户的需求，可在配置文件中修改，不同编码的应用实时机。

| 数据类型 | 内部编码方式              | 编码结果   |
| -------- | ------------------------- | ---------- |
| 字符串   | REDIS_ENCODING_RAW        | raw        |
|          | REDIS_ENCODING_INT        | int        |
|          | REDIS_ENCODING_EMBSTR     | embstr     |
| 散列     | REDIS_ENCODING_HT         | hashtable  |
|          | REDIS_ENCODING_ZIPLIST    | ziplist    |
| 列表     | REDIS_ENCODING_LINKEDLIST | linkedlist |
|          | REDIS_ENCODING_ZIPLIST    | ziplist    |
| 集合     | REDIS_ENCODING_HT         | hashtable  |
|          | REDIS_ENCODING_INTSET     | intset     |
| 有序集合 | REDIS_ENCODING_SKIPLIST   | skiplist   |
|          | REDIS_ENCODING_ZIPLIST    | ziplist    |

- 字符串
当大小不超过39字节，内部采用REDIS_ENCODING_EMBSTR编码，当对该值做修改时，会将其转化为REDIS_ENCODING_RAW编码。

- 散列类型
可在配置文件中设定不同编码的使用时机。

```text
hash-max-ziplist-entries 参数值
hash-max-ziplist-value 参数值
```
当散列类型键的字段个数少于hash-max-ziplist-entries参数值且每个字段名和字段值的长度都小于 hash-max-ziplist-value 参数值（单位为字节）时，
Redis 就会使用 REDIS_ENCODING_ZIPLIST 来存储该键，否则就会使用 REDIS_ENCODING_HT。
转换过程是透明的，每当键值变更后Redis都会自动判断是否满足条件来完成转换。

- 列表类型
列表类型的内部编码方式可能是 REDIS_ENCODING_LINKEDLIST或 REDIS_ENCODING_ZIPLIST。
同样在配置文件中可以定义使用REDIS_ENCODING_ZIPLIST方式编码的时机：
```text
list-max-ziplist-entries 参数值
list-max-ziplist-value 参数值
```

- 集合类型
集合类型的内部编码方式可能是 REDIS_ENCODING_HT 或 REDIS_ENCODING_INTSET。
当集合中的所有元素都是整数且元素的个数小于配置文件中的set-max-intset-entries参数指定值（默认是512）时
Redis会使用REDIS_ENCODING_INTSET编码存储该集合，否则会使用REDIS_ENCODING_HT来存储。
```text
set-max-intset-entries 参数值
```

- 有序集合
有序集合类型的内部编码方式可能是 REDIS_ENCODING_SKIPLIST 或REDIS_ENCODING_ZIPLIST。
同样在配置文件中可以定义使用REDIS_ENCODING_ZIPLIST方式编码的时机：
```text
zset-max-ziplist-entries 参数值
zset-max-ziplist-value 参数值
```
----
>其中，Redis对于字符串，数字等常量值是作为共享值处理，即同样的字符串，它们的地址是相同的。但是，当对Redis的使用内存设置的限制，则不再共享，因为需要为每个值记录其使用率等信息，这涉及到Redis中内存的管理细节。
>关于Redis内存的其它细节，可深入了解Redis的内存机制。

----
### 4 分布式
#### 4.1 主从链
当业务规模较为庞大后，单台服务器同时完成读写操作较为困难，此时需要将负载分配到多个服务器上。为了保证数据的一致性，读操作可以由多个服务器分担，而写操作则需要一个服务器负责，其它服务器从该服务器获取数据副本。由此，产生主、从关系。

- 创建从属服务器。命令 `slaveof `，若服务器A试图同步服务器B的数据，则A为B的从属服务器。
```bash
服务器A> slaveof 服务器B的IP 端口
```
或者在配置文件中，对参数 `slaveof <masterip> <masterport>`进行配置，
若是主服务器被密码保护，则对参数 `masterauth <master-password>`进行配置。

此时A开始复制B的数据，若是需要停止复制，命令如下，
```bash
服务器A> slaveof no one
```
通过 `info replication`命令可查看是否与对应的主或从服务器连接，及其它信息。
##### 4.1.1 哨兵（Setinel)
在主从链中，一个主节点下面有若干个从节点。若主节点出现故障而下线，则最好在从节点中尽快选举出新的主节点，并且保证只有一个从节点转变为主节点。为实现自动化，让系统自动发现故障并修复，则出现**哨兵**。
在设置哨兵后，主从链的每个节点都可以由一个哨兵定期检查，一旦发现主节点故障，多个哨兵会一起协商，推举出新的主节点，并通知用户或客户端。

- 主从链配置
前面所述的主从链的配置相对简单，适合学习时操作。实际上，节点可能根据自身情况做不同的配置，则最好在各自的配置文件中操作。
在配置文件 **redis.conf**可以配置对应的IP、端口，以及日志和相关的文件目录。
另外，为方便起见，不同节点的文件，最好在文件名上有所标注，如标注对应的端口号之类的信息。
**主节点**
文件：redis-6379.conf
配置：

```text
bind 127.0.0.1 其它IP
port 6379
daemonize yes
logfile "6379.log"
dbfilename "dump-6379.rdb"
dir "给定目录"
```
启动主节点：

```shell
redis-server redis-6379.conf
```
**从节点**
文件：redis-6380.conf（端口号可自行决定）
配置：

```bash
port 6380
bind 127.0.0.1 其它IP
daemonize yes
logfile "6380.log"
dbfilename "dump-6380.rdb"
dir "目录"
slaveof 127.0.0.1或其它IP 6379
```
若是本机上不同端口对应的不同Redis实例，可用127.0.0.1，若是不同机器上的，则需要指明对应的IP地址
同样利用前述的方式启动
```bash
redis-server redis-6380.conf
```
- 部署Sentinel节点
Sentinel节点与普通的Redis节点类似，只是缺少持久化等额外的功能。Redis中存在相关的配置文件 **sentinel.conf**。
与上述节点的配置类似，
文件：redis-sentinel-26379.conf（文件名表明是哨兵节点，以及其端口号，可自定义端口）
配置：

```bash
port 26379
bind IP
daemonize yes
logfile "26379.log"
dir 目录
sentinel monitor mymaster IP 端口 个数
sentinel down-after-milliseconds mymaster 毫秒数
sentinel parallel-syncs mymaster 个数
sentinel failover-timeout mymaster 秒数
sentinel auth-pass <master-name> <password>
sentinel notification-script <master-name> <script-path>
sentinel client-reconfig-script <master-name> <script-path>
```
其中，**sentinel monitor mymaster**指，监控IP:端口所对应的Redis节点，若是需要判断主节点故障，则需要至少给定个数的哨兵都同意才可以。
下面的几个参数，根据其名字可知，**down-after-milliseconds mymaster**大致为超过给定时间未获得回复，则认为其下线；**parallel-syncs mymaster**指，当设置了新的主节点后，每次向该主节点发起复制操作的从节点的个数；**failover-timeout mymaster**为故障转移超时时限，通常是对主节点做故障切换时，若是超过该时限，则认为切换失败，其中，若是本次切换失败，下一次会将时限扩大为双倍。
**notification-script**，当发生某些警告级别的Sentinel事件，会触发对应目录上的脚本；
**client-reconfig-script**，在故障转移结束后，会触发对应路径的脚本，并向脚本发送故障转移结果的相关参数，当启用该功能时，配置如下：

```bash
<master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
```

**启动哨兵**

```bash
redis-sentinel redis-sentinel-26379.conf
```
或者

```bash
redis-server redis-sentinel-26379.conf --sentinel
```
#### 4.2 集群
前面的主从链主要通过对数据进行复制，保证数据的安全性。但是当数据本身非常巨大，导致单个服务器已无法妥善保存，则需要多个机器共同保存，由此产生集群。
集群的第一个问题是，数据以何种规则保存在哪个机器上，将机器组成的集群抽象化后，即为在一个大的存储空间上进行分区，问题转化为资源与分区的分配。
常用的方法有：节点取余分区，一致性哈希分区，虚拟槽分区（Redis采用此方法）。

- Redis分区方法：
Redis Cluster的槽（slot)数为0~16383，节点（即机器）均匀分配一定量的槽数。当键值通过哈希算法对应到槽的序号后，分配到对应的节点再到达对应的槽中。其中哈希计算公式为：
$ slot=CRC16 (key) \& 16383 $
该方法能解耦数据与节点的关系，方便扩展。

 **集群功能的限制**
>1）key批量操作支持有限。如mset、mget，目前只支持具有相同slot值的key执行批量操作。对于映射为不同slot值的key由于执行mget、mget等操作可能存在于多个节点上因此不被支持。
>2）key事务操作支持有限。同理只支持多key在同一节点上的事务操作，当多个key分布在不同的节点上时无法使用事务功能。
>3）key作为数据分区的最小粒度，因此不能将一个大的键值对象如hash、list等映射到不同的节点。
>4）不支持多数据库空间。单机下的Redis可以支持16个数据库，集群模式下只能使用一个数据库空间，即db0。
>5）复制结构只支持一层，从节点只能复制主节点，不支持嵌套树状复制结构。

##### 4.2.1 配置
首先创建并启动若干个Redis节点。
每个节点的配置文件中设置参数

```text
# 节点端口
port 6379#随意
# 开启集群模式
cluster-enabled yes
# 节点超时时间，单位毫秒
cluster-node-timeout 15000#可修改
# 集群内部配置文件
cluster-config-file "nodes-端口号.conf"
```
在第一次启动后，会根据 `cluster-config-file`的设置创建一个集群配置文件。
每个节点会生成一个40位的16进制的字符串作为节点ID，保存在各自的集群配置文件中。利用命令 `cluster nodes`可查看该集群节点的状态。
**当前，所有启动的节点并没有组成一个集群**

- 节点握手
从一个节点开始，向其它节点发送命令，让对方获悉自身的存在，当该节点与其它节点均建立联系后，所有节点会自发与其它节点均建立联系。节点之间使用Gossip协议通信。
握手命令如下：

```bash
cluster meet IP 端口
```

- 分配槽
命令 `cluster addslots`，使用如下：

```bash
redis-cli -h 127.0.0.1 -p 端口1 cluster addslots {0...5461}
redis-cli -h 127.0.0.1 -p 端口2 cluster addslots {5462...10922}
redis-cli -h 127.0.0.1 -p 端口3 cluster addslots {10923...16383}
```
为保证数据安全，同样可以设置主从链，命令 `cluster replicate`，使用如下：

```bash
IP:端口>cluster replicate 节点ID
```
##### 4.2.2 集群伸缩

- 扩展
当新节点加入后，由于槽数固定，则原本的节点需要将自身的部分槽转移给新节点。假设新节点为A，另外已有节点B，以A和B的通信为例，介绍数据如何进行转移。
操作如下，

```bash
#1. B向A发送命令，让A做好导入某个槽的数据的准备
cluster setslot {槽序号} importing {A-NodeId}
#2. A向B发送命令，让B准备导出某个槽的数据
cluster setslot {槽序号} migrating {B-NodeId}
#3. B循环执行命令，获得某个槽的若干个键
cluster getkeysinslot {槽序号} {键的个数}
#4. B执行命令，把上述获取的键通过管道发送给A
migrate {A-Ip} {A-Port} "" 0 {timeout} keys {keys...}
#重复执行 3和4，直到该槽内的键全部转移完
#5. 向集群所有节点发送命令，表示对应槽的位置改变
cluster setslot {槽序号} node {A-NodeId}
```

- 集群收缩
当节点退出时，将采用类似的命令将自身的槽转移到剩余节点中。


#### 5 缓存优化
正如之前已经提及的，实际中的使用常用Redis作为缓存部分，而存储层则使用MySQL等持久层的工具管理。由于存储层的工作效率远低于缓存，因此对于大规模的访问，更希望由缓存负责。如果实际运行后，出现大量的访问需要动用到存储层，不但意味着缓存的设计有问题，也会有极大的可能导致系统宕机。

- 穿透优化
当一个搜索在缓存中无法命中，随后到达存储层中，同样没有命中，如果这一现象非常多，则对存储层造成极大的压力。为解决这一问题，最简单的是，对于查询失败的信息，在缓存中保存一个结果，同时为防止这类信息过多，为其设置过期时限；另外，可设置布隆过滤器，详细的信息可自行搜索，主要思想为，在正式向缓存搜索前，先对搜索问题做过滤，将一些可能没有结果的问题拒绝查询。

- 无底洞优化
由于数据量的增大，导致集群的节点数量也需要增长，而节点数量的增长，也导致需要搜索的键保存在不同的节点上，当需要搜索多个键时，需要在不同节点上进行搜索。此时，节点数量的增加，导致增加的网络时延和执行时间的增加反而降低效率。
没有具体的解决方案，根据场景，可通过并行编程，hash_tag等方式优化。hash_tag将多个键强制保存到对应节点上。