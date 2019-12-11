# redis持久化

## 目录
- [什么是持久化](#什么是持久化)
- [两种持久化方式](#两种持久化方式)
- [RDB持久化](#RDB持久化)
  - [RDB执行过程](#RDB执行过程)
  - [快照保存路径](#快照保存路径)
  - [RDB持久化执行策略](#RDB持久化执行策略)
    - [自动执行持久化](#自动执行持久化)
    - [手动执行持久化](#手动执行持久化)
  - [RDB优缺点](#RDB优缺点)
- [AOF持久化](#AOF持久化)
  - [开启AOF](#开启AOF)
  - [AOF执行过程](#AOF执行过程)
  - [AOF持久化执行策略](#AOF持久化执行策略)
  - [优化AOF备份文件](#优化AOF备份文件)
  - [AOF优缺点](#AOF优缺点)
- [参考](#参考)

### 什么是持久化
由于Redis的数据都存放在内存中，如果没有配置持久化，redis重启后数据就全丢失了，于是需要开启redis的持久化功能，将数据保存到磁盘上，当redis重启后，可以从磁盘中恢复数据。

### 两种持久化方式
```
Redis 提供两种持久化解决方案：RDB 持久化(默认)和 AOF 持久化。
RDB 持久化可以在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）。
AOF 持久化记录服务器执行的所有写操作(添加,修改,删除)命令，并在服务器启动时，通过重新执行这些命令来还原数据集。
```

### RDB持久化
采用 RDB 持久化方案时，Redis 会每隔一段时间对数据集进行快照备份，换句话说这种方案在服务器发生故障时可能造成数据的丢失。
#### RDB执行过程
当 Redis 需要保存 dump.rdb 文件时，服务器执行以下操作：
```
1. Redis 调用 fork() ，同时拥有父进程和子进程。
2. 子进程将数据集写入到一个临时 RDB 文件中。
3. 当子进程完成对新 RDB 文件的写入时，Redis 用新 RDB 文件替换原来的 RDB 文件，并删除旧的 RDB 文件。
```

#### 快照保存路径
```shell
vim redis.conf
```
![RDB](https://raw.githubusercontent.com/duiying/img/master/RDB.jpg)  

#### RDB持久化执行策略
##### 自动执行持久化
```shell
vim redis.conf
```
![RDB触发机制](https://raw.githubusercontent.com/duiying/img/master/RDB触发机制.jpg)  
上图中的含义是
```
900秒内如果超过1个key被修改, 则发起快照
300秒内如果超过10个key被修改, 则发起快照
60秒内如果超过10000个key被修改, 则发起快照
```

##### 手动执行持久化
可以通过调用 SAVE 或者 BGSAVE , 手动让 Redis 进行数据集保存操作。
```shell
[root@localhost src]# ll dump.rdb 
-rw-r--r-- 1 root root 108 Jun 23 16:14 dump.rdb
[root@localhost src]# redis-cli -h 127.0.0.1 -p 6397 save
OK
[root@localhost src]# ll dump.rdb 
-rw-r--r-- 1 root root 108 Jun 24 15:28 dump.rdb
[root@localhost src]# redis-cli -h 127.0.0.1 -p 6397 save
OK
[root@localhost src]# ll dump.rdb 
-rw-r--r-- 1 root root 108 Jun 24 15:29 dump.rdb
```
save 和 bgsave 都可以手动的执行 RDB 持久化处理。但是它们的工作模式完全不同。
```
1. 执行 SAVE 命令时，会阻塞 Redis 主进程，直到保存完成为止。在主进程阻塞期间，服务器不能处理客户端的任何请求。
2. BGSAVE 则 fork 出一个子进程，子进程负责执行保存处理，并在保存完成之后向主进程发送信号，通知主进程保存完成。
   所以 Redis 服务器在 BGSAVE 执行期间仍然可以继续处理客户端的请求。
```
注意:  
虽然通过 SAVE 命令可以执行 RDB 持久化处理，但是它的运行原理同自动持久化中的 save 指令是完全不同的，save 指令的工作原理同 BGSAVE 指令。


#### RDB优缺点
优点:
```
RDB 是一个非常紧凑（compact）的文件，它保存了 Redis 在某个时间点上的数据集。 
这种文件非常适合用于进行备份：比如说，你可以在最近的24小时内，每小时备份一次RDB文件，并且在每个月的每一天，也备份一个RDB文件。 
这样的话，即使遇上问题，也可以随时将数据集还原到不同的版本。

RDB 可以最大化 Redis 的性能：父进程在保存 RDB 文件时唯一要做的就是 fork 出一个子进程，
然后这个子进程就会处理接下来的所有保存工作，父进程无须执行任何磁盘 I/O 操作。

RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。
```
缺点:
```
如果你需要尽量避免在服务器故障时丢失数据，那么 RDB 不适合你。 
虽然 Redis 允许你设置不同的保存点（save point）来控制保存 RDB 文件的频率， 
但是， 因为 RDB 文件需要保存整个数据集的状态， 所以它并不是一个轻松的操作。 
因此你可能会至少 5 分钟才保存一次 RDB 文件。 在这种情况下， 一旦发生故障停机， 你就可能会丢失好几分钟的数据。
```

### AOF持久化
AOF: append only file  
通过 RDB 持久化方案的学习，我们知道它可能导致数据丢失，如果你的项目忍不了数据丢失的问题，那么可能就需要使用 AOF 持久化方案。

#### 开启AOF
默认情况下, Redis会禁用AOF, 我们需要到redis.conf配置文件中打开AOF, 然后重启redis服务    
![开启aof](https://raw.githubusercontent.com/duiying/img/master/开启aof.png)  

#### AOF执行过程
```
1. Redis 执行 fork() ，现在同时拥有父进程和子进程。
2. 子进程开始将新 AOF 文件的内容写入到临时文件。
3. 对于所有新执行的写入命令，父进程一边将它们累积到一个内存缓存中，一边将这些改动追加到现有 AOF 文件的末尾： 
   这样即使在重写的中途发生停机，现有的 AOF 文件也还是安全的。
4. 当子进程完成重写工作时，它给父进程发送一个信号，父进程在接收到信号之后，将内存缓存中的所有数据追加到新 AOF 文件的末尾。
5. 搞定！现在 Redis 原子地用新文件替换旧文件，之后所有命令都会直接追加到新 AOF 文件的末尾。
```

#### AOF持久化执行策略
```
AOF 持久化方案提供 3 种不同时间策略将数据同步到磁盘中，同步策略通过 appendfsync 指令完成：
1. everysec（默认）：表示每秒执行一次 fsync 同步策略，效率上同 RDB 持久化差不多。由于每秒同步一次，所以服务器故障时会丢失 1 秒内的数据。
2. always: 每个写命令都会调用 fsync 进行数据同步，最安全但影响性能。
3. no: 表示 Redis 从不执行 fsync，数据将完全由内核控制写入磁盘。对于 Linux 系统来说，每 30 秒写入一次。
```
推荐采用默认的 everysec 每秒同步策略，兼顾安全与效率。

#### 优化AOF备份文件
我们知道 AOF 的运行原理是不断的将写入的命令以 Redis 通信协议的数据格式追加到 .aof 文件末尾，这就会导致文件的体积不断增大。  

如果命令处理类似计数器的功能，比如执行 100 次 INCR（incr counter） 处理，AOF 文件会保存全部的 INCR 命令的执行记录，但实际上我们知道这些处理的结果同 set counter 100 并无二致。这就导致我们的 .aof 多存储了 99 条命令记录。

这时，我们就可以使用 Redis 提供的 BGREWRITEAOF 重写命令，将 AOF 文件进行重写优化。
```shell
127.0.0.1:6379> BGREWRITEAOF
Background append only file rewriting started
```

#### AOF优缺点
优点:
```
1. 提供比 RDB 持久化方案更安全的数据，由于默认采用每秒进行持久化处理，
   所有即使服务器重启或宕机，最多也就丢失 1 秒内的数据。
2. AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 
   因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。
```
缺点:
```
1. 相比于 RDB 持久化，AOF 文件会比 RDB 备份文件大得多。
2. AOF 持久化的速度可能比 RDB 持久化速度慢。
```


### 参考
- [http://redisdoc.com/topic/persistence.html](http://redisdoc.com/topic/persistence.html)
- [https://juejin.im/post/5b70dfcf518825610f1f5c16](https://juejin.im/post/5b70dfcf518825610f1f5c16)
- [https://learnku.com/articles/15422/redis-persistence-persistence-technology-pocket-book](https://learnku.com/articles/15422/redis-persistence-persistence-technology-pocket-book)