---
title: 认识 Redis 
date: 2018-01-14 02:05:41
description: 关于 Redis 的一次入门总结
tags:
- Redis
categories:
- NoSQL
---

# 搭建

## 部署
**基于 `centos 6.5` 安装**

下载安装包


```
wget http://download.redis.io/redis-stable.tar.gz
```

解压安装包

```
tar -zxvf redis-stable.tar.gz
```

编译、安装

```
cd redis-stable
make
make install
```

**注：**  `make` 命令时可能会报错，如果提示 `gcc command` 不识别，自行安装 `gcc` ;

## 启动
加上 `&` 号使 `Redis` 以后台程序方式运行；加参数 `--protected-mode no` 可以使无保护模式启动；

```
redis-server ./path/to/conf-file &
```

或者在配置文件修改 `daemonize` 的值为 `yes`，也可以使 `Redis` 在后台启动；

## 检测

检测后台进程是否存在

```
ps -ef |grep redis
```

检测 6379 端口是否在监听

```
netstat -lntp | grep 6379
```

使用 `redis-cli` 客户端检测连接是否正常

```
redis-cli [-h localhost -p 6379]
```

如果外部无法访问，检查配置文件是否开启外部访问

```
# bind 127.0.0.1
```

# 持久化

`Redis` 有两种持久化的方式：快照（`RDB` 文件）和追加式文件（`AOF` 文件）:

- `RDB` 持久化方式是在一个特定的间隔保存某个时间点的一个数据快照。
- `AOF`（`Append only file`）持久化方式则会记录每一个服务器收到的写操作。数据恢复时，这些记录的操作会逐条执行从而重建出原来的数据。写操作命令记录的格式跟 `Redis` 协议一致，以追加的方式进行保存。


## RDB

`RDB` 就是`Snapshot`存储，是默认的持久化方式。按照一定的策略周期性的将数据保存到磁盘。对应产生的数据文件为 `dump.rdb`，通过配置文件中的`save`参数来定义快照的周期。`Redis`支持将当前数据的快照存成一个数据文件实现持久化。而一个持续写入的数据库如何生成快照呢。`Redis`借助了`fork`命令的`copy on write`机制。在生成快照时，将当前进程`fork`出一个子进程，然后在子进程中循环所有的数据，将数据写成为`RDB`文件。

`Client` 也可以使用 `save` 或者 `bgsave` 命令通知 `redis` 做一次快照持久化。`save` 操作是在主线程中保存快照的，由于 `redis` 是用一个主线程来处理所有 `client` 的请求，这种方式会阻塞所有 `client` 请求，所以不推荐使用。另一点需要注意的是，每次快照持久化都是将内存数据完整写入到磁盘一次，并不是增量的只同步脏数据。如果数据量大的话，而且写操作比较多，必然会引起大量的磁盘 `io` 操作，可能会严重影响性能。 

`Redis` 的 `RDB` 文件不会坏掉，因为其写操作是在一个新进程中进行的。当生成一个新的 `RDB` 文件时，`Redis` 生成的子进程会先将数据写到一个临时文件中，然后通过原子性 `rename` 系统调用将临时文件重命名为 `RDB` 文件。这样在任何时候出现故障，`Redis` 的 `RDB` 文件都总是可用的。并且 `Redis` 的 `RDB` 文件也是 `Redis` 主从同步内部实现中的一环。

> 主从同步：

>第一次 `Slave` 向 `Master` 同步的实现是：
`Slave` 向 `Master` 发出同步请求，`Master` 先 `dump` 出 `rdb` 文件，然后将 `rdb` 文件全量传输给 `slave`，然后 `Master` 把缓存的命令转发给 `Slave`，初次同步完成。

> 第二次 以及以后的同步实现是：
`Master` 将变量的快照直接实时依次发送给各个 `Slave`。但不管什么原因导致 `Slave` 和 `Master` 断开重连都会重复以上两个步骤 (完整 + 增量) 的过程。

> `Redis` 的主从复制是建立在内存快照的持久化基础上的，只要有 `Slave` 就一定会有内存快照发生。

### 工作原理

- `Redis` 调用 `fork()`，产生一个子进程。
- 父进程继续处理 `client` 请求，子进程把内存数据写到一个临时的 `RDB` 文件。由于 `os` 的写时复制机制（`copy on write`)父子进程会共享相同的物理页面，当父进程处理写请求时 `os` 会为父进程要修改的页面创建副本，而不是写共享的页面。所以子进程的地址空间内的数据是 `fork` 时刻整个数据库的一个快照。
- 当子进程将快照写入临时文件完毕后，用临时文件替换原来的快照文件，然后子进程退出

### 优缺点
**优点：**

- `RDB` 文件是一个很简洁的单文件，它保存了某个时间点的 `Redis` 数据，很适合用于做备份。你可以设定一个时间点对 `RDB` 文件进行归档，这样就能在需要的时候很轻易的把数据恢复到不同的版本。
- `RDB` 很适合用于灾备。单文件很方便就能传输到远程的服务器上。
- `RDB` 的性能很好，需要进行持久化时，主进程会 `fork` 一个子进程出来，然后把持久化的工作交给子进程，自己不会有相关的 `I/O` 操作。
- 比起 `AOF`，在数据量比较大的情况下，`RDB` 的启动速度更快。

**缺点：**

- `RDB` 容易造成数据的丢失。假设每 5 分钟保存一次快照，如果 `Redis` 因为某些原因不能正常工作，那么从上次产生快照到 `Redis` 出现问题这段时间的数据就会丢失了。
- `RDB` 使用 `fork()` 产生子进程进行数据的持久化，如果数据比较大的话可能就会花费点时间，造成 `Redis` 停止服务几毫秒。如果数据量很大且 `CPU` 性能不是很好的时候，停止服务的时间甚至会到 1 秒。

## AOF
快照并不是很可靠。如果服务器突然 `Crash` 了，那么最新的数据就会丢失。而 `AOF` 文件则提供了一种更为可靠的持久化方式。每当 `Redis` 接受到会修改数据集的命令时，就会把命令追加到 `AOF` 文件里，当你重启 `Redis` 时，`AOF` 里的命令会被重新执行一次，重建数据。

### 工作原理

- `redis` 调用 `fork` ，现在有父子两个进程
- 子进程根据内存中的数据库快照，往临时文件中写入重建数据库状态的命令
- 父进程继续处理 `client` 请求，除了把写命令写入到原来的 `aof` 文件中。同时把收到的写命令缓存起来。这样就能保证如果子进程重写失败的话并不会出问题
- 当子进程把快照内容写入已命令方式写到临时文件中后，子进程发信号通知父进程。然后父进程把缓存的写命令也写入到临时文件
- 现在父进程可以使用临时文件替换老的 `aof` 文件，并重命名，后面收到的写命令也开始往新的 `aof` 文件中追加

### 优缺点

**优点：**

- 比 `RDB` 可靠。你可以制定不同的 `fsync` 策略：不进行 `fsync`、每秒 `fsync` 一次和每次查询进行 `fsync`。默认是每秒 `fsync` 一次。这意味着你最多丢失一秒钟的数据。
- `AOF` 日志文件是一个纯追加的文件。就算服务器突然 `Crash`，也不会出现日志的定位或者损坏问题。甚至如果因为某些原因（例如磁盘满了）命令只写了一半到日志文件里，我们也可以用 `redis-check-aof` 这个工具很简单的进行修复。
- 当 `AOF` 文件太大时，`Redis`会自动在后台进行重写。重写很安全，因为重写是在一个新的文件上进行，同时 `Redis` 会继续往旧的文件追加数据。新文件上会写入能重建当前数据集的最小操作命令的集合。当新文件重写完，`Redis` 会把新旧文件进行切换，然后开始把数据写到新文件上。
- `AOF` 把操作命令以简单易懂的格式一条接一条的保存在文件里，很容易导出来用于恢复数据。例如我们不小心用 `FLUSHALL` 命令把所有数据刷掉了，只要文件没有被重写，我们可以把服务停掉，把最后那条命令删掉，然后重启服务，这样就能把被刷掉的数据恢复回来。


**缺点：**

- 在相同的数据集下，`AOF` 文件的大小一般会比 `RDB` 文件大。
- 在某些 `fsync` 策略下，`AOF` 的速度会比 `RDB` 慢。通常 `fsync` 设置为每秒一次就能获得比较高的性能，而在禁止 `fsync` 的情况下速度可以达到 `RDB` 的水平。
- 在过去曾经发现一些很罕见的 `BUG` 导致使用 `AOF` 重建的数据跟原数据不一致的问题。

### 日志重写

随着写操作的不断增加， `AOF` 文件会越来越大。例如你递增一个计数器 100 次，那么最终结果就是数据集里的计数器的值为最终的递增结果，但是 `AOF` 文件里却会把这 100 次操作完整的记录下来。而事实上要恢复这个记录，只需要 1 个命令就行了，也就是说 `AOF` 文件里那 100 条命令其实可以精简为 1 条。所以 `Redis` 支持这样一个功能：在不中断服务的情况下在后台重建 `AOF` 文件。

工作原理如下：

- `Redis` 调用 `fork()`，产生一个子进程。
- 子进程把新的 `AOF` 写到一个临时文件里。
- 主进程持续把新的变动写到内存里的 `buffer`，同时也会把这些新的变动写到旧的 `AOF` 里，这样即使重写失败也能保证数据的安全。
- 当子进程完成文件的重写后，主进程会获得一个信号，然后把内存里的 `buffer` 追加到子进程生成的那个新 `AOF` 里。

我们可以通过配置设置日志重写的条件：

```
#在日志重写时，不进行命令追加操作，而只是将其放在缓冲区里，避免与命令的追加造成DISK IO上的冲突。
#设置为yes表示rewrite期间对新写操作不fsync,暂时存在内存中,等rewrite完成后再写入，默认为no，建议yes

no-appendfsync-on-rewrite yes

# Redis会记住自从上一次重写后AOF文件的大小（如果自Redis启动后还没重写过，则记住启动时使用的AOF文件的大小）。
# 如果当前的文件大小比起记住的那个大小超过指定的百分比，则会触发重写。
# 同时需要设置一个文件大小最小值，只有大于这个值文件才会重写，以防文件很小，但是已经达到百分比的情况。
 
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

要禁用自动的日志重写功能，我们可以把百分比设置为0：

```
auto-aof-rewrite-percentage 0
```

### 数据损坏修复

如果因为某些原因（例如服务器崩溃）`AOF` 文件损坏了，导致 `Redis` 加载不了，可以通过以下方式进行修复：

1. 备份 `AOF` 文件。
2. 使用 `redis-check-aof` 命令修复原始的 `AOF` 文件： `redis-check-aof --fix`
3. 可以使用 `diff -u` 命令看下两个文件的差异。
4. 使用修复过的文件重启 `Redis` 服务。

### 从 RDB 切换到 AOF

这里只说 `Redis >= 2.2` 版本的方式：

1. 备份一个最新的 `dump.rdb` 的文件，并把备份文件放在一个安全的地方。
2. 运行以下两条命令：

 ```
 $ redis-cli config set appendonly yes
 $ redis-cli config set save ""
 ```
   第二条命令是用来禁用 `RDB` 的持久化方式，但是这不是必须的，因为你可以同时启用两种持久化方式。
3. 确保数据跟切换前一致。
4. 确保数据正确的写到 `AOF` 文件里。


## 总结

**从上面看出，`RDB` 和 `AOF` 操作都是顺序 `IO` 操作，性能都很高。而同时在通过 `RDB` 文件或者 `AOF` 日志进行数据库恢复的时候，也是顺序的读取数据加载到内存中。所以也不会造成磁盘的随机读。**

到底选择什么呢？下面是来自官方的建议：

通常，如果你要想提供很高的数据保障性，那么建议你同时使用两种持久化方式。如果你可以接受灾难带来的几分钟的数据丢失，那么你可以仅使用 `RDB`。很多用户仅使用了 `AOF`，但是我们建议，既然 `RDB` 可以时不时的给数据做个完整的快照，并且提供更快的重启，所以最好还是也使用 `RDB`。

在数据恢复方面：`RDB` 的启动时间会更短，原因有两个

- 一是 `RDB` 文件中每一条数据只有一条记录，不会像 `AOF` 日志那样可能有一条数据的多次操作记录。所以每条数据只需要写一次就行了。
- 另一个原因是 `RDB` 文件的存储格式和 `Redis` 数据在内存中的编码格式是一致的，不需要再进行数据编码工作，所以在 `CPU` 消耗上要远小于 `AOF` 日志的加载。 

**注意：**

上面说了 `RDB` 快照的持久化，需要注意：在进行快照的时候（`save`），`fork` 出来进行 `dump` 操作的子进程会占用与父进程一样的内存，真正的 `copy-on-write` ，对性能的影响和内存的耗用都是比较大的。比如机器 8G 内存， `Redis` 已经使用了 6G 内存，这时 `save` 的话会再生成 6G，变成 12G，大于系统的 8G。这时候会发生交换；要是虚拟内存不够则会崩溃，导致数据丢失。所以在用 `redis` 的时候一定对系统内存做好容量规划。

目前，通常的设计思路是利用 `Replication` 机制来弥补 `aof`、`snapshot` 性能上的不足，达到了数据可持久化。即 `Master` 上 `Snapshot` 和 `AOF` 都不做，来保证 `Master` 的读写性能，而 `Slave` 上则同时开启 `Snapshot` 和 `AOF` 来进行持久化，保证数据的安全性。

# 主从示例

现在开启三个 `redis` 服务器，分别对应三个不同的端口: 6379、6380、6381；

三台服务器，由 `端口 6379` 担当主服务器 `master`， `端口 6380` 担当从服务器 `slave1`， `端口 6381` 担当从服务器 `slave2`; 

**配置：**

`master` 的配置：

1. 关闭 `rdb` 备份；
2. `aof` 备份可要可不要；如果打开，则从服务器没有必要开启；如果关闭，把 `slave1` 的 `aof` 备份开启；

`slave1` 的配置：

```
# 端口与进程文件设置
port 6380
pidfile /var/run/redis_6380.pid
# 开启 rdb 备份
dbfilename dump6380.rdb
# 关闭 aof 备份
appendonly no
# 开启 slave
slaveof localhost 6379
# 保持该从服务器只读
slave-read-only yes
```

`slave2` 的配置：

```
# 端口与进程文件设置
port 6381
pidfile /var/run/redis_6381.pid
# 关闭 rdb 备份
#save 900 1
#save 300 10
#save 60 10000
# 关闭 aof 备份
appendonly no
# 开启 slave
slaveof localhost 6379
# 保持该从服务器只读
slave-read-only yes
```

## FAQ

1.如果误操作 `flushdb` 或 `flushall`，怎么办？ 

答： 通过 `aof` 恢复：立即执行 `shutdown nosave` ;将 `aof` 文件中的和 `flush` 操作有关的进行删除；然后，重启 `Redis` 服务器；

2.如何手动修改主从关系？

答：假设 6379 这个 `master` 宕机了，这时想设置 6380 这个 `slave` 主机为 `master` 主机， 做以下步骤：

1) 使 6380 不继续做 `master` 主机；（命令：`slaveof no one`）。修改 `readonly` 参数为 `no`；（命令：`config set slave-read-only no`）
2) 其它的 `slave` 从机再指定新的这台 6380 新 `master` 主机，命令该 `slave` 为新 `master` 主机的 `slave`；（命令：`slaveof [ip] [port]`）


# 附录

## 官方的配置文件例子

```
# Redis configuration file example.
#
# Note that in order to read the configuration file, Redis must be
# started with the file path as first argument:
#
# ./redis-server /path/to/redis.conf

# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth (sor forth: 等等):
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive (case insensitive: 大小写不敏感) so 1GB 1Gb 1gB are all the same.

################################## INCLUDES ###################################

# Include one or more other config files here.  This is useful if you
# have a standard template that goes to all Redis servers but also need
# to customize a few per-server settings.  Include files can include
# other files, so use this wisely.
#
# Notice option "include" won't be rewritten by command "CONFIG REWRITE"
# from admin or Redis Sentinel（哨兵）. Since Redis always uses the last processed（处理）
# line as value of a configuration directive, you'd better put includes
# at the beginning of this file to avoid overwriting config change at runtime.
#
# If instead you are interested in using includes to override configuration
# options, it is better to use include as the last line.
#
# include /path/to/local.conf
# include /path/to/other.conf

################################## NETWORK #####################################

# By default, if no "bind" configuration directive is specified, Redis listens
# for connections from all the network interfaces available on the server.
# It is possible to listen to just one or multiple selected interfaces using
# the "bind" configuration directive, followed by one or more IP addresses.
#
# Examples:
#
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
#
# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
# internet, binding to all the interfaces is dangerous and will expose the
# instance to everybody on the internet. So by default we uncomment（取消注释）
# the following bind directive, that will force Redis to listen only into
# the IPv4 lookback interface address (this means Redis will be able to
# accept connections only from clients running into the same computer it
# is running).
#
# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# bind 127.0.0.1

# Protected mode is a layer of security protection（security protection： 安全保护）, 
# in order to avoid that Redis instances left open on the internet 
# are accessed and exploited.
#
# When protected mode is on and if:
#
# 1) The server is not binding explicitly to （explicitly to: 明确的） a set 
#    of addresses using the "bind" directive.
# 2) No password is configured.
#
# The server only accepts connections from clients connecting from the
# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
# sockets.
#
# By default protected mode is enabled. You should disable it only if
# you are sure you want clients from other hosts to connect to Redis
# even if no authentication is configured, nor a specific set of interfaces
# are explicitly listed using the "bind" directive.
protected-mode no

# Accept connections on the specified port, default is 6379 (IANA #815344).
# If port 0 is specified Redis will not listen on a TCP socket.
port 6379

# TCP listen() backlog. （tcp 连接维护队列）
#
# In high requests-per-second environments you need an high backlog in order
# to avoid slow clients connections issues. Note that the Linux kernel
# will silently truncate it to the value of /proc/sys/net/core/somaxconn so
# make sure to raise both the value of somaxconn and tcp_max_syn_backlog
# in order to get the desired effect.
tcp-backlog 511

# Unix socket.
#
# Specify the path for the Unix socket that will be used to listen for
# incoming connections. There is no default, so Redis will not listen
# on a unix socket when not specified.
#
# unixsocket /tmp/redis.sock
# unixsocketperm 700

# Close the connection after a client is idle（闲置） for N seconds (0 to disable)
timeout 0

# TCP keepalive. （定时发送 Ack 检测客户端是否正常）
#
# If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence（缺席）
# of communication. This is useful for two reasons:
#
# 1) Detect dead peers （发现无法连接的结点）.
# 2) Take the connection alive from the point of view of network
#    equipment in the middle.
#
# On Linux, the specified value (in seconds) is the period used to send ACKs.
# Note that to close the connection the double of the time is needed.
# On other kernels the period depends on the kernel configuration.
#
# A reasonable value for this option is 300 seconds, which is the new
# Redis default starting with Redis 3.2.1.
tcp-keepalive 300

################################# GENERAL #####################################

# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize no

# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised no

# If a pid file is specified, Redis writes it where specified at startup
# and removes it at exit.
#
# When the server runs non daemonized, no pid file is created if none is
# specified in the configuration. When the server is daemonized, the pid file
# is used even if not specified, defaulting to "/var/run/redis.pid".
#
# Creating a pid file is best effort: if Redis is not able to create it
# nothing bad happens, the server will start and run normally.
pidfile /var/run/redis_6379.pid

# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice

# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile ""

# 是否开启 syslog 日志
# To enable logging to the system logger, just set 'syslog-enabled' to yes,
# and optionally update the other syslog parameters to suit your needs.
# syslog-enabled no

# 指定 syslog 的身份
# Specify the syslog identity.
# syslog-ident redis

# 指定  syslog 的级别
# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
# syslog-facility local0

# 设置数据库的数量
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16

################################ SNAPSHOTTING  ################################
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

save 900 1
save 300 10
save 60 10000

# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
stop-writes-on-bgsave-error yes

# Compress（压缩） string objects using LZF when dump .rdb databases?
# For default that's set to 'yes' as it's almost always a win.
# If you want to save（节省） some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible（可压缩的） values or keys.
rdbcompression yes

# rdb 文件较验
# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
# This makes the format more resistant to corruption but there is a performance
# hit to pay (around 10%) when saving and loading RDB files, so you can disable it
# for maximum performances.
#
# RDB files created with checksum disabled have a checksum of zero that will
# tell the loading code to skip the check.
rdbchecksum yes

# The filename where to dump the DB
dbfilename dump.rdb


# The working directory. （redis 的工作目录）
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir ./

################################# REPLICATION #################################

# Master-Slave replication. Use slaveof to make a Redis instance a copy of
# another Redis server. A few things to understand ASAP about Redis replication.
#
# 1) Redis replication is asynchronous（异步）, but you can configure a master to
#    stop accepting writes if it appears to be not connected with at least
#    a given number of slaves.
# 2) Redis slaves are able to perform a partial（局部的） resynchronization with the
#    master if the replication link is lost for a relatively small amount of
#    time. You may want to configure the replication backlog size (see the next
#    sections of this file) with a sensible value depending on your needs.
# 3) Replication is automatic and does not need user intervention. After a
#    network partition slaves automatically try to reconnect to masters
#    and resynchronize with them.
#
# slaveof <masterip> <masterport>

# If the master is password protected (using the "requirepass" configuration
# directive below) it is possible to tell the slave to authenticate before
# starting the replication synchronization process, otherwise the master will
# refuse the slave request.
#
# masterauth <master-password>

# When a slave loses its connection with the master, or when the replication
# is still in progress, the slave can act in two different ways:
#
# 1) if slave-serve-stale-data is set to 'yes' (the default) the slave will
#    still reply to client requests, possibly with out of date data, or the
#    data set may just be empty if this is the first synchronization.
#
# 2) if slave-serve-stale-data is set to 'no' the slave will reply with
#    an error "SYNC with master in progress" to all the kind of commands
#    but to INFO and SLAVEOF.
#
slave-serve-stale-data yes

# You can configure a slave instance to accept writes or not. Writing against
# a slave instance may be useful to store some ephemeral data (because data
# written on a slave will be easily deleted after resync with the master) but
# may also cause problems if clients are writing to it because of a
# misconfiguration.
#
# Since Redis 2.6 by default slaves are read-only.
#
#
# 只读的从redis并不适合直接暴露给不可信的客户端。为了尽量降低风险，可以使用
# rename-command 指令来将一些可能有破坏力的命令重命名，避免外部直接调用。比如：
#
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
#
#
# Note: read only slaves are not designed to be exposed to untrusted（不可信任的） clients
# on the internet. It's just a protection layer against misuse of the instance.
# Still a read only slave exports by default all the administrative commands
# such as CONFIG, DEBUG, and so forth. To a limited extent you can improve
# security of read only slaves using 'rename-command' to shadow（隐藏） all the
# administrative / dangerous commands.
slave-read-only yes

# Replication SYNC strategy(策略): disk or socket.
#
# -------------------------------------------------------
# WARNING: DISKLESS REPLICATION IS EXPERIMENTAL（试验性的） CURRENTLY
# -------------------------------------------------------
#
# New slaves and reconnecting slaves that are not able to continue the replication
# process just receiving differences, need to do what is called a "full
# synchronization". An RDB file is transmitted from the master to the slaves.
# The transmission can happen in two different ways:
#
# 1) Disk-backed: The Redis master creates a new process that writes the RDB
#                 file on disk. Later the file is transferred by the parent
#                 process to the slaves incrementally.
# 2) Diskless: The Redis master creates a new process that directly writes the
#              RDB file to slave sockets, without touching the disk at all.
#
# With disk-backed replication, while the RDB file is generated, more slaves
# can be queued and served with the RDB file as soon as the current child producing
# the RDB file finishes its work. With diskless replication instead once
# the transfer starts, new slaves arriving will be queued and a new transfer
# will start when the current one terminates.
#
# When diskless replication is used, the master waits a configurable amount of
# time (in seconds) before starting the transfer in the hope that multiple slaves
# will arrive and the transfer can be parallelized（平行）.
#
# With slow disks and fast (large bandwidth) networks, diskless replication
# works better.
repl-diskless-sync no

# When diskless replication is enabled, it is possible to configure the delay
# the server waits in order to spawn the child that transfers the RDB via socket
# to the slaves.
#
# This is important since once the transfer starts, it is not possible to serve
# new slaves arriving, that will be queued for the next RDB transfer, so the server
# waits a delay in order to let more slaves arrive.
#
# The delay is specified in seconds, and by default is 5 seconds. To disable
# it entirely just set it to 0 seconds and the transfer will start ASAP.
repl-diskless-sync-delay 5

# Slaves send PINGs to server in a predefined（预先确定的） interval（间隔）. 
# It's possible to change this interval with the repl_ping_slave_period option. 
# The default value is 10 seconds.
#
# repl-ping-slave-period 10

# The following option sets the replication timeout for:
#
# 1) Bulk transfer I/O （Bulk transfer I/O ： 大规模 IO 传输） during SYNC, from the point of view of slave.
# 2) Master timeout from the point of view of slaves (data, pings).
# 3) Slave timeout from the point of view of masters (REPLCONF ACK pings).
#
# It is important to make sure that this value is greater than the value
# specified for repl-ping-slave-period otherwise a timeout will be detected
# every time there is low traffic between the master and the slave.
#
# repl-timeout 60

# Disable TCP_NODELAY on the slave socket after SYNC?
#
# If you select "yes" Redis will use a smaller number of TCP packets and
# less bandwidth（带宽） to send data to slaves. But this can add a delay for
# the data to appear on the slave side, up to 40 milliseconds with
# Linux kernels using a default configuration.
#
# If you select "no" the delay for data to appear on the slave side will
# be reduced but more bandwidth will be used for replication.
#
# By default we optimize for low latency, but in very high traffic conditions
# or when the master and slaves are many hops away, turning this to "yes" may
# be a good idea.
repl-disable-tcp-nodelay no

# Set the replication backlog size. The backlog is a buffer that accumulates
# slave data when slaves are disconnected for some time, so that when a slave
# wants to reconnect again, often a full resync is not needed, but a partial
# resync is enough, just passing the portion（部分） of data the slave missed while
# disconnected.
#
# The bigger the replication backlog, the longer the time the slave can be
# disconnected and later be able to perform a partial resynchronization.
#
# The backlog is only allocated once there is at least a slave connected.
#
# repl-backlog-size 1mb

# After a master has no longer connected slaves for some time, the backlog
# will be freed（清除）. The following option configures the amount of seconds that
# need to elapse（消逝）, starting from the time the last slave disconnected, for
# the backlog buffer to be freed.
#
# A value of 0 means to never release the backlog.
#
# repl-backlog-ttl 3600

# The slave priority is an integer number published by Redis in the INFO output.
# It is used by Redis Sentinel in order to select a slave to promote into a
# master if the master is no longer working correctly.
#
# A slave with a low priority number is considered better for promotion, so
# for instance if there are three slaves with priority 10, 100, 25 Sentinel will
# pick the one with priority 10, that is the lowest.
#
# However a special priority of 0 marks the slave as not able to perform the
# role of master, so a slave with priority of 0 will never be selected by
# Redis Sentinel for promotion.
#
# By default the priority is 100.
slave-priority 100

# It is possible for a master to stop accepting writes if there are less than
# N slaves connected, having a lag less or equal than M seconds.
#
# The N slaves need to be in "online" state.
#
# The lag（延迟） in seconds, that must be <= the specified value, is calculated from
# the last ping received from the slave, that is usually sent every second.
#
# This option does not GUARANTEE that N replicas（复制品） will accept the write, but
# will limit the window of exposure（暴露） for lost writes in case not enough slaves
# are available, to the specified number of seconds.
#
# For example to require at least 3 slaves with a lag <= 10 seconds use:
#
# min-slaves-to-write 3
# min-slaves-max-lag 10
#
# Setting one or the other to 0 disables the feature.
#
# By default min-slaves-to-write is set to 0 (feature disabled) and
# min-slaves-max-lag is set to 10.

# A Redis master is able to list the address and port of the attached
# slaves in different ways. For example the "INFO replication" section
# offers this information, which is used, among other tools, by
# Redis Sentinel in order to discover slave instances.
# Another place where this info is available is in the output of the
# "ROLE" command of a masteer.
#
# The listed IP and address normally reported by a slave is obtained
# in the following way:
#
#   IP: The address is auto detected by checking the peer address
#   of the socket used by the slave to connect with the master.
#
#   Port: The port is communicated by the slave during the replication
#   handshake, and is normally the port that the slave is using to
#   list for connections.
#
# However when port forwarding or Network Address Translation (NAT) is
# used, the slave may be actually reachable via different IP and port
# pairs. The following two options can be used by a slave in order to
# report to its master a specific set of IP and port, so that both INFO
# and ROLE will report those values.
#
# There is no need to use both the options if you need to override just
# the port or the IP address.
#
# slave-announce-ip 5.5.5.5
# slave-announce-port 1234

################################## SECURITY ###################################

# Require clients to issue AUTH <PASSWORD> before processing any other
# commands.  This might be useful in environments in which you do not trust
# others with access to the host running redis-server.
#
# This should stay commented out for backward compatibility and because most
# people do not need auth (e.g. they run their own servers).
#
# Warning: since Redis is pretty fast an outside user can try up to
# 150k passwords per second against a good box. This means that you should
# use a very strong password otherwise it will be very easy to break.
#
# requirepass foobared

# Command renaming.
#
# It is possible to change the name of dangerous commands in a shared
# environment. For instance the CONFIG command may be renamed into something
# hard to guess so that it will still be available for internal-use tools
# but not available for general clients.
#
# Example:
#
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
#
# It is also possible to completely kill a command by renaming it into
# an empty string:
#
# rename-command CONFIG ""
#
# Please note that changing the name of commands that are logged into the
# AOF file or transmitted to slaves may cause problems.

################################### LIMITS ####################################

# Set the max number of connected clients at the same time. By default
# this limit is set to 10000 clients, however if the Redis server is not
# able to configure the process file limit to allow for the specified limit
# the max number of allowed clients is set to the current file limit
# minus（减去） 32 (as Redis reserves（保留） a few file descriptors for internal uses).
#
# Once the limit is reached Redis will close all the new connections sending
# an error 'max number of clients reached'.
#
# maxclients 10000

# Don't use more memory than the specified amount of bytes.
# When the memory limit is reached Redis will try to remove keys
# according to the eviction policy selected (see maxmemory-policy).
#
# If Redis can't remove keys according to the policy, or if the policy is
# set to 'noeviction', Redis will start to reply with errors to commands
# that would use more memory, like SET, LPUSH, and so on, and will continue
# to reply to read-only commands like GET.
#
# This option is usually useful when using Redis as an LRU cache, or to set
# a hard memory limit for an instance (using the 'noeviction' policy).
#
# WARNING: If you have slaves attached to an instance with maxmemory on,
# the size of the output buffers needed to feed the slaves are subtracted
# from the used memory count, so that network problems / resyncs will
# not trigger a loop where keys are evicted, and in turn the output
# buffer of slaves is full with DELs of keys evicted triggering the deletion
# of more keys, and so forth until the database is completely emptied（清空）.
#
# In short... if you have slaves attached it is suggested that you set a lower
# limit for maxmemory so that there is some free RAM on the system for slave
# output buffers (but this is not needed if the policy is 'noeviction（不移除）').
#
# maxmemory <bytes>

# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select among five behaviors:
#
# volatile-lru -> remove the key with an expire set using an LRU algorithm 使用LRU算法移除过期集合中的key
# allkeys-lru -> remove any key according to the LRU algorithm 使用LRU算法移除key
# volatile-random -> remove a random key with an expire set  在过期集合中移除随机的key
# allkeys-random -> remove a random key, any key 移除随机的key
# volatile-ttl -> remove the key with the nearest expire time (minor TTL) 移除那些TTL值最小的key，即那些最近才过期的key。
# noeviction -> don't expire at all, just return an error on write operations 不进行移除。针对写操作，只是返回错误信息。
#
# Note: with any of the above policies, Redis will return an error on write
#       operations, when there are no suitable keys for eviction.
#
#       At the date of writing these commands are: set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort
#
# The default is:
#
# maxmemory-policy noeviction

# LRU and minimal TTL algorithms are not precise（精确的） algorithms but approximated（估计的）
# algorithms (in order to save memory), so you can tune it for speed or
# accuracy. For default Redis will check five keys and pick the one that was
# used less recently, you can change the sample size using the following
# configuration directive.
#
# The default of 5 produces good enough results. 10 Approximates very closely
# true LRU but costs a bit more CPU. 3 is very fast but not very accurate.
#
# maxmemory-samples 5

############################## APPEND ONLY MODE ###############################

# By default Redis asynchronously（异步） dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage（a power outage：停电） may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability（耐用性）. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees（保证）.
#
# Please check http://redis.io/topics/persistence for more information.

appendonly no

# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof"

# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP（ASAP： 尽快）.
#
# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise（妥协）.
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with（live with: 容忍） the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary（on the contrary：正相反）, use "always" that's very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

# appendfsync always
appendfsync everysec
# appendfsync no

# 在日志重写时，不进行命令追加操作，而只是将其放在缓冲区里，避免与命令的追加造成 DISK IO 上的冲突。
# 设置为 yes 表示 rewrite 期间对新写操作不 fsync, 暂时存在内存中,等 rewrite 完成后再写入
#
# When the AOF fsync policy is set to always or everysec, and a background
# saving process (a background save or AOF log background rewriting) is
# performing a lot of I/O against the disk, in some Linux configurations
# Redis may block too long on the fsync() call. Note that there is no fix for
# this currently, as even performing fsync in a different thread will block
# our synchronous write(2) call.
#
# In order to mitigate（减轻） this problem it's possible to use the following option
# that will prevent（防止） fsync() from being called in the main process while a
# BGSAVE or BGREWRITEAOF is in progress.
#
# This means that while another child is saving, the durability of Redis is
# the same as "appendfsync none". In practical terms, this means that it is
# possible to lose up to 30 seconds of log in the worst scenario (with the
# default Linux settings).
#
# If you have latency（潜在） problems turn this to "yes". Otherwise leave it as
# "no" that is the safest pick from the point of view of durability.

no-appendfsync-on-rewrite no

# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly（含蓄地） calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# An AOF file may be found to be truncated（截断） at the end during the Redis
# startup process, when the AOF data gets loaded back into memory.
# This may happen when the system where Redis is running
# crashes（崩溃）, especially when an ext4 filesystem is mounted without the
# data=ordered option (however this can't happen when Redis itself
# crashes or aborts but the operating system still works correctly).
#
# Redis can either exit with an error when this happens, or load as much
# data as possible (the default now) and start if the AOF file is found
# to be truncated at the end. The following option controls this behavior.
#
# If aof-load-truncated is set to yes, a truncated AOF file is loaded and
# the Redis server starts emitting a log to inform the user of the event.
# Otherwise if the option is set to no, the server aborts with an error
# and refuses to start. When the option is set to no, the user requires
# to fix the AOF file using the "redis-check-aof" utility before to restart
# the server.
#
# Note that if the AOF file will be found to be corrupted in the middle
# the server will still exit with an error. This option only applies when
# Redis will try to read more data from the AOF file but not enough bytes
# will be found.
aof-load-truncated yes

################################ LUA SCRIPTING  ###############################

# Max execution time of a Lua script in milliseconds.
# lua脚本的最大运行时间是需要被严格限制的，要注意单位是毫秒
#
# If the maximum execution time is reached Redis will log that a script is
# still in execution after the maximum allowed time and will start to
# reply to queries with an error.
#
# When a long running script exceeds the maximum execution time only the
# SCRIPT KILL and SHUTDOWN NOSAVE commands are available. The first can be
# used to stop a script that did not yet called write commands. The second
# is the only way to shut down the server in the case a write command was
# already issued by the script but the user doesn't want to wait for the natural
# termination of the script.
#
# Set it to 0 or a negative value for unlimited execution without warnings.
lua-time-limit 5000

################################ REDIS CLUSTER  ###############################
#
# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# WARNING EXPERIMENTAL: Redis Cluster is considered to be stable code, however
# in order to mark it as "mature" we need to wait for a non trivial percentage
# of users to deploy it in production.
# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
#
# 如果配置yes则开启集群功能，此redis实例作为集群的一个节点，否则，它是一个普通的单一的redis实例。
# Normal Redis instances can't be part of a Redis Cluster; only nodes that are
# started as cluster nodes can. In order to start a Redis instance as a
# cluster node enable the cluster support uncommenting the following:
#
# cluster-enabled yes

# 虽然此配置的名字叫"集群配置文件"，但是此配置文件不能人工编辑，它是集群节点自动维护的文件，
# 主要用于记录集群中有哪些节点、他们的状态以及一些持久化参数等，方便在重启时恢复这些状态。
# 通常是在收到请求之后这个文件就会被更新。
# Every cluster node has a cluster configuration file. This file is not
# intended to be edited by hand. It is created and updated by Redis nodes.
# Every Redis Cluster node requires a different cluster configuration file.
# Make sure that instances running in the same system do not have
# overlapping cluster configuration file names.
#
# cluster-config-file nodes-6379.conf

# 这是集群中的节点能够失联的最大时间，超过这个时间，该节点就会被认为故障。
# 如果主节点超过这个时间还是不可达，则用它的从节点将启动故障迁移，升级成主节点。
# 注意，任何一个节点在这个时间之内如果还是没有连上大部分的主节点，则此节点将停止接收任何请求。一般设置为15秒即可。
# Cluster node timeout is the amount of milliseconds a node must be unreachable
# for it to be considered in failure state.
# Most other internal time limits are multiple of the node timeout.
#
# cluster-node-timeout 15000

# A slave of a failing master will avoid to start a failover if its data
# looks too old.
#
# There is no simple way for a slave to actually have a exact measure of
# its "data age", so the following two checks are performed:
#
# 1) If there are multiple slaves able to failover, they exchange messages
#    in order to try to give an advantage to the slave with the best
#    replication offset (more data from the master processed).
#    Slaves will try to get their rank by offset, and apply to the start
#    of the failover a delay proportional to their rank.
#
# 2) Every single slave computes the time of the last interaction with
#    its master. This can be the last ping or command received (if the master
#    is still in the "connected" state), or the time that elapsed since the
#    disconnection with the master (if the replication link is currently down).
#    If the last interaction is too old, the slave will not try to failover
#    at all.
#
# The point "2" can be tuned by user. Specifically a slave will not perform
# the failover if, since the last interaction with the master, the time
# elapsed is greater than:
#
#   (node-timeout * slave-validity-factor) + repl-ping-slave-period
#
# So for example if node-timeout is 30 seconds, and the slave-validity-factor
# is 10, and assuming a default repl-ping-slave-period of 10 seconds, the
# slave will not try to failover if it was not able to talk with the master
# for longer than 310 seconds.
#
# A large slave-validity-factor may allow slaves with too old data to failover
# a master, while a too small value may prevent the cluster from being able to
# elect a slave at all.
#
# For maximum availability, it is possible to set the slave-validity-factor
# to a value of 0, which means, that slaves will always try to failover the
# master regardless of the last time they interacted with the master.
# (However they'll always try to apply a delay proportional to their
# offset rank).
#
# Zero is the only value able to guarantee that when all the partitions heal
# the cluster will always be able to continue.
#
# cluster-slave-validity-factor 10

#  一个master可以拥有的最小slave数量。该项的作用是，当一个master没有任何slave的时候，
# 某些有富余slave的master节点，可以自动的分一个slave给它。
# Cluster slaves are able to migrate to orphaned（孤儿） masters, that are masters
# that are left without working slaves. This improves the cluster ability
# to resist to failures as otherwise an orphaned master can't be failed over
# in case of failure if it has no working slaves.
#
# Slaves migrate to orphaned masters only if there are still at least a
# given number of other working slaves for their old master. This number
# is the "migration barrier". A migration barrier of 1 means that a slave
# will migrate only if there is at least 1 other working slave for its master
# and so forth. It usually reflects the number of slaves you want for every
# master in your cluster.
#
# Default is 1 (slaves migrate only if their masters remain with at least
# one slave). To disable migration just set it to a very large value.
# A value of 0 can be set but is useful only for debugging and dangerous
# in production.
#
# cluster-migration-barrier 1

# 如果该项设置为yes（默认就是yes） 当一定比例的键空间没有被覆盖到
# （就是某一部分的哈希槽没了，有可能是暂时挂了）集群就停止处理任何查询炒作。
# 如果该项设置为no，那么就算请求中只有一部分的键可以被查到，一样可以查询（但是有可能会查不全）
# By default Redis Cluster nodes stop accepting queries if they detect there
# is at least an hash slot uncovered (no available node is serving it).
# This way if the cluster is partially down (for example a range of hash slots
# are no longer covered) all the cluster becomes, eventually, unavailable.
# It automatically returns available as soon as all the slots are covered again.
#
# However sometimes you want the subset of the cluster which is working,
# to continue to accept queries for the part of the key space that is still
# covered. In order to do so, just set the cluster-require-full-coverage
# option to no.
#
# cluster-require-full-coverage yes

# In order to setup your cluster make sure to read the documentation
# available at http://redis.io web site.

################################## SLOW LOG ###################################

# The Redis Slow Log is a system to log queries that exceeded a specified
# execution time. The execution time does not include the I/O operations
# like talking with the client, sending the reply and so forth,
# but just the time needed to actually execute the command (this is the only
# stage of command execution where the thread is blocked and can not serve
# other requests in the meantime).
#
# You can configure the slow log with two parameters: one tells Redis
# what is the execution time, in microseconds, to exceed in order for the
# command to get logged, and the other parameter is the length of the
# slow log. When a new command is logged the oldest one is removed from the
# queue of logged commands.

# The following time is expressed in microseconds, so 1000000 is equivalent
# to one second. Note that a negative number disables the slow log, while
# a value of zero forces the logging of every command.
slowlog-log-slower-than 10000

# There is no limit to this length. Just be aware that it will consume memory.
# You can reclaim memory used by the slow log with SLOWLOG RESET.
slowlog-max-len 128

################################ LATENCY MONITOR ##############################

# The Redis latency monitoring(latency monitoring: 延迟监控) subsystem samples different operations
# at runtime in order to collect data related to possible sources of
# latency of a Redis instance.
#
# Via the LATENCY command this information is available to the user that can
# print graphs and obtain reports.
#
# The system only logs operations that were performed in a time equal or
# greater than the amount of milliseconds specified via the
# latency-monitor-threshold configuration directive. When its value is set
# to zero, the latency monitor is turned off.
#
# By default latency monitoring is disabled since it is mostly not needed
# if you don't have latency issues, and collecting data has a performance
# impact, that while very small, can be measured under big load. Latency
# monitoring can easily be enabled at runtime using the command
# "CONFIG SET latency-monitor-threshold <milliseconds>" if needed.
latency-monitor-threshold 0

############################# EVENT NOTIFICATION ##############################

# Redis can notify Pub/Sub clients about events happening in the key space.
# This feature is documented at http://redis.io/topics/notifications
#
# For instance if keyspace events notification is enabled, and a client
# performs a DEL operation on key "foo" stored in the Database 0, two
# messages will be published via Pub/Sub:
#
# PUBLISH __keyspace@0__:foo del
# PUBLISH __keyevent@0__:del foo
#
# It is possible to select the events that Redis will notify among a set
# of classes. Every class is identified by a single character:
#
#  K     Keyspace events, published with __keyspace@<db>__ prefix.
#  E     Keyevent events, published with __keyevent@<db>__ prefix.
#  g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
#  $     String commands
#  l     List commands
#  s     Set commands
#  h     Hash commands
#  z     Sorted set commands
#  x     Expired events (events generated every time a key expires)
#  e     Evicted events (events generated when a key is evicted for maxmemory)
#  A     Alias for g$lshzxe, so that the "AKE" string means all the events.
#
#  The "notify-keyspace-events" takes as argument a string that is composed
#  of zero or multiple characters. The empty string means that notifications
#  are disabled.
#
#  Example: to enable list and generic events, from the point of view of the
#           event name, use:
#
#  notify-keyspace-events Elg
#
#  Example 2: to get the stream of the expired keys subscribing to channel
#             name __keyevent@0__:expired use:
#
#  notify-keyspace-events Ex
#
#  By default all notifications are disabled because most users don't need
#  this feature and the feature has some overhead. Note that if you don't
#  specify at least one of K or E, no events will be delivered.
notify-keyspace-events ""

############################### ADVANCED CONFIG ###############################

# Hashes are encoded using a memory efficient data structure when they have a
# small number of entries, and the biggest entry does not exceed a given
# threshold. These thresholds can be configured using the following directives.
hash-max-ziplist-entries 512
hash-max-ziplist-value 64

# Lists are also encoded in a special way to save a lot of space.
# The number of entries allowed per internal list node can be specified
# as a fixed maximum size or a maximum number of elements.
# For a fixed maximum size, use -5 through -1, meaning:
# -5: max size: 64 Kb  <-- not recommended for normal workloads
# -4: max size: 32 Kb  <-- not recommended
# -3: max size: 16 Kb  <-- probably not recommended
# -2: max size: 8 Kb   <-- good
# -1: max size: 4 Kb   <-- good
# Positive numbers mean store up to _exactly_ that number of elements
# per list node.
# The highest performing option is usually -2 (8 Kb size) or -1 (4 Kb size),
# but if your use case is unique, adjust the settings as necessary.
list-max-ziplist-size -2

# Lists may also be compressed.
# Compress depth is the number of quicklist ziplist nodes from *each* side of
# the list to *exclude* from compression.  The head and tail of the list
# are always uncompressed for fast push/pop operations.  Settings are:
# 0: disable all list compression
# 1: depth 1 means "don't start compressing until after 1 node into the list,
#    going from either the head or tail"
#    So: [head]->node->node->...->node->[tail]
#    [head], [tail] will always be uncompressed; inner nodes will compress.
# 2: [head]->[next]->node->node->...->node->[prev]->[tail]
#    2 here means: don't compress head or head->next or tail->prev or tail,
#    but compress all nodes between them.
# 3: [head]->[next]->[next]->node->node->...->node->[prev]->[prev]->[tail]
# etc.
list-compress-depth 0

# Sets have a special encoding in just one case: when a set is composed
# of just strings that happen to be integers in radix 10 in the range
# of 64 bit signed integers.
# The following configuration setting sets the limit in the size of the
# set in order to use this special memory saving encoding.
set-max-intset-entries 512

# Similarly to hashes and lists, sorted sets are also specially encoded in
# order to save a lot of space. This encoding is only used when the length and
# elements of a sorted set are below the following limits:
zset-max-ziplist-entries 128
zset-max-ziplist-value 64

# HyperLogLog sparse representation bytes limit. The limit includes the
# 16 bytes header. When an HyperLogLog using the sparse representation crosses
# this limit, it is converted into the dense representation.
#
# A value greater than 16000 is totally useless, since at that point the
# dense representation is more memory efficient.
#
# The suggested value is ~ 3000 in order to have the benefits of
# the space efficient encoding without slowing down too much PFADD,
# which is O(N) with the sparse encoding. The value can be raised to
# ~ 10000 when CPU is not a concern, but space is, and the data set is
# composed of many HyperLogLogs with cardinality in the 0 - 15000 range.
hll-sparse-max-bytes 3000

# Active rehashing uses 1 millisecond every 100 milliseconds of CPU time in
# order to help rehashing the main Redis hash table (the one mapping top-level
# keys to values). The hash table implementation Redis uses (see dict.c)
# performs a lazy rehashing: the more operation you run into a hash table
# that is rehashing, the more rehashing "steps" are performed, so if the
# server is idle the rehashing is never complete and some more memory is used
# by the hash table.
#
# The default is to use this millisecond 10 times every second in order to
# actively rehash the main dictionaries, freeing memory when possible.
#
# If unsure:
# use "activerehashing no" if you have hard latency requirements and it is
# not a good thing in your environment that Redis can reply from time to time
# to queries with 2 milliseconds delay.
#
# use "activerehashing yes" if you don't have such hard requirements but
# want to free memory asap when possible.
activerehashing yes

# The client output buffer limits can be used to force disconnection of clients
# that are not reading data from the server fast enough for some reason (a
# common reason is that a Pub/Sub client can't consume messages as fast as the
# publisher can produce them).
#
# The limit can be set differently for the three different classes of clients:
#
# normal -> normal clients including MONITOR clients
# slave  -> slave clients
# pubsub -> clients subscribed to at least one pubsub channel or pattern
#
# The syntax of every client-output-buffer-limit directive is the following:
#
# client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
#
# A client is immediately disconnected once the hard limit is reached, or if
# the soft limit is reached and remains reached for the specified number of
# seconds (continuously).
# So for instance if the hard limit is 32 megabytes and the soft limit is
# 16 megabytes / 10 seconds, the client will get disconnected immediately
# if the size of the output buffers reach 32 megabytes, but will also get
# disconnected if the client reaches 16 megabytes and continuously overcomes
# the limit for 10 seconds.
#
# By default normal clients are not limited because they don't receive data
# without asking (in a push way), but just after a request, so only
# asynchronous clients may create a scenario where data is requested faster
# than it can read.
#
# Instead there is a default limit for pubsub and slave clients, since
# subscribers and slaves receive data in a push fashion.
#
# Both the hard or the soft limit can be disabled by setting them to zero.
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

# Redis calls an internal function to perform many background tasks, like
# closing connections of clients in timeout, purging expired keys that are
# never requested, and so forth.
#
# Not all tasks are performed with the same frequency, but Redis checks for
# tasks to perform according to the specified "hz" value.
#
# By default "hz" is set to 10. Raising the value will use more CPU when
# Redis is idle, but at the same time will make Redis more responsive when
# there are many keys expiring at the same time, and timeouts may be
# handled with more precision.
#
# The range is between 1 and 500, however a value over 100 is usually not
# a good idea. Most users should use the default of 10 and raise this up to
# 100 only in environments where very low latency is required.
hz 10

# When a child rewrites the AOF file, if the following option is enabled
# the file will be fsync-ed every 32 MB of data generated. This is useful
# in order to commit the file to the disk more incrementally and avoid
# big latency spikes.
aof-rewrite-incremental-fsync yes
```

