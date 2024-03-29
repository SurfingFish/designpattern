Redis文件配置详解  

Redis是一个高性能的key-value数据库。

  Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
  Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
  Redis支持数据的备份，即master-slave模式的数据备份。

　为了更好的使用redis,我们需要详细的了解redis配置文件及相关参数作用。

```
`include ``/path/to/local``.conf`
```

 额外载入配置文件，如果有需要的话，可以开启此配置

 

```
`bind 127.0.0.1``bind 192.168.1.100`
```

绑定redis服务器网卡IP，默认为127.0.0.1,即本地回环地址。这样的话，访问redis服务只能通过本机的客户端连接，而无法通过远程连接。如果bind选项为空的话，那会接受所有来自于可用网络接口的连接。如上配置，绑定一个127.0.0.1的本机地址和192.168.1.100的外网地址。

 

```
protected-mode yes
```

保护模式，默认是开启状态，只允许本地客户端连接， 可以设置密码或添加bind来连接

 

```
port 6379
```

监听端口号，默认为6379，如果设为0，redis将不在socket 上监听任何客户端连接

 

```
tcp-backlog 511
```

TCP监听的最大容纳数量，在高并发的环境下，你需要把这个值调高以避免客户端连接缓慢的问题。Linux 内核会把这个值缩小成 /proc/sys/net/core/somaxconn对应的值，要提升并发量需要修改这两个值才能达到目的

 

```
unixsocket /tmp/redis.sock
unixsocketperm 700
```

指定redis监听的unix socket路径，默认不启用，unixsocketper指定文件的权限

 

```
timeout 0
```

指定在一个 client 空闲多少秒之后关闭连接（0表示永不关闭）

 

```
tcp-keepalive 300
```

单位是秒，表示将周期性的使用SO_KEEPALIVE检测客户端是否还处于健康状态，避免服务器一直阻塞，官方给出的建议值是300s，如果设置为0，则不会周期性的检测

 

```
daemonize yes
```

默认情况下 redis 不是作为守护进程运行的，如果你想让它在后台运行，你就把它改成 yes。当redis作为守护进程运行的时候，它会写一个 pid 到 /var/run/redis.pid 文件里面

 

```
supervised no
```

可以通过upstart和systemd管理Redis守护进程
选项：
   supervised no - 没有监督互动
   supervised upstart - 通过将Redis置于SIGSTOP模式来启动信号
   supervised systemd - signal systemd将READY = 1写入$ NOTIFY_SOCKET
   supervised auto - 检测upstart或systemd方法基于 UPSTART_JOB或NOTIFY_SOCKET环境变量

 

```
pidfile /var/redis/run/redis_6379.pid
```

配置PID文件路径，当redis作为守护进程运行的时候，它会把 pid 默认写到 /var/redis/run/redis_6379.pid 文件里面

 

```
loglevel notice
```

定义日志级别。
  可以是下面的这些值：
  debug（记录大量日志信息，适用于开发、测试阶段）
  verbose（较多日志信息）
  notice（适量日志信息，使用于生产环境）
  warning（仅有部分重要、关键信息才会被记录）

 

```
logfile /var/redis/log/redis_6379.log
```

日志文件的位置，当指定为空字符串时，为标准输出，如果redis已守护进程模式运行，那么日志将会输出到/dev/null

 

```
syslog-enabled no
```

要想把日志记录到系统日志，就把它改成 yes，也可以可选择性的更新其他的syslog 参数以达到你的要求

 

```
syslog-ident redis
```

设置系统日志的ID

 

```
syslog-facility local0
```

指定系统日志设置，必须是 USER 或者是 LOCAL0-LOCAL7 之间的值

 

```
databases 16
```

设置数据库的数目。默认的数据库是DB 0 ，可以在每个连接上使用select  <dbid> 命令选择一个不同的数据库，dbid是一个介于0到databases - 1 之间的数值。

 

```
save 900 1
save 300 10
save 60 10000
```

存 DB 到磁盘：
    格式：save <间隔时间（秒）> <写入次数>
    根据给定的时间间隔和写入次数将数据保存到磁盘
    下面的例子的意思是：
    900 秒内如果至少有 1 个 key 的值变化，则保存
    300 秒内如果至少有 10 个 key 的值变化，则保存
    60 秒内如果至少有 10000 个 key 的值变化，则保存
 　　
    注意：你可以注释掉所有的 save 行来停用保存功能。
    也可以直接一个空字符串来实现停用：
    save ""

 

```
stop-writes-on-bgsave-error yes
```

  如果用户开启了RDB快照功能，那么在redis持久化数据到磁盘时如果出现失败，默认情况下，redis会停止接受所有的写请求。
  这样做的好处在于可以让用户很明确的知道内存中的数据和磁盘上的数据已经存在不一致了。
  如果redis不顾这种不一致，一意孤行的继续接收写请求，就可能会引起一些灾难性的后果。
  如果下一次RDB持久化成功，redis会自动恢复接受写请求。
  如果不在乎这种数据不一致或者有其他的手段发现和控制这种不一致的话，可以关闭这个功能，
  以便在快照写入失败时，也能确保redis继续接受新的写请求。

 

```
rdbcompression yes
```

  对于存储到磁盘中的快照，可以设置是否进行压缩存储。
  如果是的话，redis会采用LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，
  可以设置为关闭此功能，但是存储在磁盘上的快照会比较大。

 

```
rdbchecksum yes
```

  在存储快照后，我们还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，
  如果希望获取到最大的性能提升，可以关闭此功能。

 

```
dbfilename dump.rdb
```

设置快照的文件名

 

```
dir /var/redis/6379
```

  设置快照文件的存放路径，这个配置项一定是个目录，而不能是文件名

 

```
slaveof <masterip> <masterport>
```

  主从复制，使用 slaveof 来让一个 redis 实例成为另一个reids 实例的副本，默认关闭
  注意这个只需要在 slave 上配置

 

```
masterauth <master-password>
```

 如果 master 需要密码认证，就在这里设置，默认不设置

 

```
slave-serve-stale-data yes
```

  当一个 slave 与 master 失去联系，或者复制正在进行的时候，
  slave 可能会有两种表现：
  1) 如果为 yes ，slave 仍然会应答客户端请求，但返回的数据可能是过时，
     或者数据可能是空的在第一次同步的时候 
  2) 如果为 no ，在你执行除了 info he salveof 之外的其他命令时，
     slave 都将返回一个 "SYNC with master in progress" 的错误

 

```
slave-read-only yes
```

  你可以配置一个 slave 实体是否接受写入操作。
  通过写入操作来存储一些短暂的数据对于一个 slave 实例来说可能是有用的，
  因为相对从 master 重新同步数而言，据数据写入到 slave 会更容易被删除。
  但是如果客户端因为一个错误的配置写入，也可能会导致一些问题。
  从 redis 2.6 版起，默认 slaves 都是只读的。

 

```
repl-diskless-sync no
```

  主从数据复制是否使用无硬盘复制功能。
  新的从站和重连后不能继续备份的从站，需要做所谓的“完全备份”，即将一个RDB文件从主站传送到从站。
  这个传送有以下两种方式：
  1）硬盘备份：redis主站创建一个新的进程，用于把RDB文件写到硬盘上。过一会儿，其父进程递增地将文件传送给从站。
  2）无硬盘备份：redis主站创建一个新的进程，子进程直接把RDB文件写到从站的套接字，不需要用到硬盘。
  在硬盘备份的情况下，主站的子进程生成RDB文件。一旦生成，多个从站可以立即排成队列使用主站的RDB文件。
  在无硬盘备份的情况下，一次RDB传送开始，新的从站到达后，需要等待现在的传送结束，才能开启新的传送。
  如果使用无硬盘备份，主站会在开始传送之间等待一段时间（可配置，以秒为单位），希望等待多个子站到达后并行传送。
  在硬盘低速而网络高速（高带宽）情况下，无硬盘备份更好。

 

```
repl-diskless-sync-delay 5
```

  当启用无硬盘备份，服务器等待一段时间后才会通过套接字向从站传送RDB文件，这个等待时间是可配置的。
  这一点很重要，因为一旦传送开始，就不可能再为一个新到达的从站服务。从站则要排队等待下一次RDB传送。因此服务器等待一段
  时间以期更多的从站到达。
  延迟时间以秒为单位，默认为5秒。要关掉这一功能，只需将它设置为0秒，传送会立即启动。

 

```
repl-ping-slave-period 10
```

  从redis会周期性的向主redis发出PING包，你可以通过repl_ping_slave_period指令来控制其周期，默认是10秒。

 

```
repl-timeout 60
```

  接下来的选项为以下内容设置备份的超时时间：
  1）从从站的角度，同步期间的批量传输的I/O
  2）从站角度认为的主站超时（数据，ping）
  3）主站角度认为的从站超时（REPLCONF ACK pings)
  确认这些值比定义的repl-ping-slave-period要大，否则每次主站和从站之间通信低速时都会被检测为超时。

 

```
repl-disable-tcp-nodelay no
```

  同步之后是否禁用从站上的TCP_NODELAY
  如果你选择yes，redis会使用较少量的TCP包和带宽向从站发送数据。但这会导致在从站增加一点数据的延时。
  Linux内核默认配置情况下最多40毫秒的延时。
  如果选择no，从站的数据延时不会那么多，但备份需要的带宽相对较多。
  默认情况下我们将潜在因素优化，但在高负载情况下或者在主从站都跳的情况下，把它切换为yes是个好主意。

 

```
repl-backlog-size 1mb
```

  设置备份的工作储备大小。工作储备是一个缓冲区，当从站断开一段时间的情况时，它替从站接收存储数据，
  因此当从站重连时，通常不需要完全备份，只需要一个部分同步就可以，即把从站断开时错过的一部分数据接收。
  工作储备越大，从站可以断开并稍后执行部分同步的断开时间就越长。
  只要有一个从站连接，就会立刻分配一个工作储备。

 

```
repl-backlog-ttl 3600
```

  主站有一段时间没有与从站连接，对应的工作储备就会自动释放。
  这个选项用于配置释放前等待的秒数，秒数从断开的那一刻开始计算，值为0表示不释放。

 

```
slave-priority 100
```

  从站优先级是可以从redis的INFO命令输出中查到的一个整数。当主站不能正常工作时
  redis sentinel使用它来选择一个从站并将它提升为主站。
  低优先级的从站被认为更适合于提升，因此如果有三个从站优先级分别是10， 
  100，25，sentinel会选择优先级为10的从站，因为它的优先级最低。
  然而优先级值为0的从站不能执行主站的角色，因此优先级为0的从站永远不会被redis sentinel提升。
  默认优先级是100

 

```
  min-slaves-to-write 3
  min-slaves-max-lag 10
```

  主站可以停止接受写请求，当与它连接的从站少于N个，滞后少于M秒，N个从站必须是在线状态。
  延迟的秒数必须<=所定义的值，延迟秒数是从最后一次收到的来自从站的ping开始计算。ping通常是每秒一次。
  这一选项并不保证N个备份都会接受写请求，但是会限制在指定秒数内由于从站数量不够导致的写操作丢失的情况。
  如果想要至少3个从站且延迟少于10秒，如上配置即可

 

```
slave-announce-ip 5.5.5.5
slave-announce-port 1234
```

 Redis master能够以不同的方式列出所连接slave的地址和端口。 
 例如，“INFO replication”部分提供此信息，除了其他工具之外，Redis Sentinel还使用该信息来发现slave实例。
 此信息可用的另一个地方在masterser的“ROLE”命令的输出中。
 通常由slave报告的列出的IP和地址,通过以下方式获得：
 IP：通过检查slave与master连接使用的套接字的对等体地址自动检测地址。
 端口：端口在复制握手期间由slavet通信，并且通常是slave正在使用列出连接的端口。
 然而，当使用端口转发或网络地址转换（NAT）时，slave实际上可以通过(不同的IP和端口对)来到达。 slave可以使用以下两个选项，以便向master报告一组特定的IP和端口，
 以便INFO和ROLE将报告这些值。
 如果你需要仅覆盖端口或IP地址，则没必要使用这两个选项。

 

```
requirepass foobared
```

  设置redis连接密码

 

```
rename-command CONFIG ""
```

  将命令重命名，为了安全考虑，可以将某些重要的、危险的命令重命名。
  当你把某个命令重命名成空字符串的时候就等于取消了这个命令。

 

```
maxclients 10000
```

  设置客户端最大并发连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件
  描述符数-32（redis server自身会使用一些），如果设置 maxclients为0 
  表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息

 

```
maxmemory <bytes>
```

  指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key
  当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，
  会把Key存放内存，Value会存放在swap区，格式：maxmemory <bytes>

 

```
maxmemory-policy noeviction
```

  当内存使用达到最大值时，redis使用的清楚策略。有以下几种可以选择：
  1）volatile-lru   利用LRU算法移除设置过过期时间的key (LRU:最近使用 Least Recently Used ) 
  2）allkeys-lru   利用LRU算法移除任何key 
  3）volatile-random 移除设置过过期时间的随机key 
  4）allkeys-random  移除随机ke
  5）volatile-ttl   移除即将过期的key(minor TTL) 
  6）noeviction  noeviction   不移除任何key，只是返回一个写错误 ，默认选项

 

```
maxmemory-samples 5
```

  LRU 和 minimal TTL 算法都不是精准的算法，但是相对精确的算法(为了节省内存)
  随意你可以选择样本大小进行检，redis默认选择3个样本进行检测，你可以通过maxmemory-samples进行设置样本数

 

```
appendonly no
```

  默认redis使用的是rdb方式持久化，这种方式在许多应用中已经足够用了。但是redis如果中途宕机，
  会导致可能有几分钟的数据丢失，根据save来策略进行持久化，Append Only File是另一种持久化方式，
  可以提供更好的持久化特性。Redis会把每次写入的数据在接收后都写入appendonly.aof文件，
  每次启动时Redis都会先把这个文件的数据读入内存里，先忽略RDB文件。

 

```
appendfilename "appendonly.aof"
```

  aof文件名

 

```
appendfsync always
appendfsync everysec
appendfsync no
```

  aof持久化策略的配置
  no表示不执行fsync，由操作系统保证数据同步到磁盘，速度最快。
  always表示每次写入都执行fsync，以保证数据同步到磁盘。
  everysec表示每秒执行一次fsync，可能会导致丢失这1s数据

 

```
no-appendfsync-on-rewrite no
```

   在aof重写或者写入rdb文件的时候，会执行大量IO，此时对于everysec和always的aof模式来说，
   执行fsync会造成阻塞过长时间，no-appendfsync-on-rewrite字段设置为默认设置为no。
   如果对延迟要求很高的应用，这个字段可以设置为yes，否则还是设置为no，这样对持久化特性来说这是更安全的选择。
   设置为yes表示rewrite期间对新写操作不fsync,暂时存在内存中,等rewrite完成后再写入，默认为no，建议yes。
   Linux的默认fsync策略是30秒。可能丢失30秒数据。

 

```
auto-aof-rewrite-percentage 100
```

  aof自动重写配置，当目前aof文件大小超过上一次重写的aof文件大小的百分之多少进行重写，
  即当aof文件增长到一定大小的时候，Redis能够调用bgrewriteaof对日志文件进行重写。
  当前AOF文件大小是上次日志重写得到AOF文件大小的二倍（设置为100）时，自动启动新的日志重写过程。

 

```
auto-aof-rewrite-min-size 64mb
```

  设置允许重写的最小aof文件大小，避免了达到约定百分比但尺寸仍然很小的情况还要重写

 

```
aof-load-truncated yes
```

  aof文件可能在尾部是不完整的，当redis启动的时候，aof文件的数据被载入内存。
  重启可能发生在redis所在的主机操作系统宕机后，尤其在ext4文件系统没有加上data=ordered选项，出现这种现象
  redis宕机或者异常终止不会造成尾部不完整现象，可以选择让redis退出，或者导入尽可能多的数据。
  如果选择的是yes，当截断的aof文件被导入的时候，会自动发布一个log给客户端然后load。
  如果是no，用户必须手动redis-check-aof修复AOF文件才可以。

 

```
lua-time-limit 5000
```

  如果达到最大时间限制（毫秒），redis会记个log，然后返回error。当一个脚本超过了最大时限。
  只有SCRIPT KILL和SHUTDOWN NOSAVE可以用。第一个可以杀没有调write命令的东西。
  要是已经调用了write，只能用第二个命令杀

 

```
cluster-enabled yes
```

  集群开关，默认是不开启集群模式

 

```
cluster-config-file nodes-6379.conf
```

 集群配置文件的名称，每个节点都有一个集群相关的配置文件，持久化保存集群的信息。
 这个文件并不需要手动配置，这个配置文件有Redis生成并更新，每个Redis集群节点需要一个单独的配置文件
 请确保与实例运行的系统中配置文件名称不冲突

 

```
cluster-node-timeout 15000
```

 节点互连超时的阀值，集群节点超时毫秒数

 

```
cluster-slave-validity-factor 10
```

  在进行故障转移的时候，全部slave都会请求申请为master，但是有些slave可能与master断开连接一段时间了，
  导致数据过于陈旧，这样的slave不应该被提升为master。该参数就是用来判断slave节点与master断线的时间是否过长。
  判断方法是：
     比较slave断开连接的时间和(node-timeout * slave-validity-factor) + repl-ping-slave-period
     如果节点超时时间为三十秒, 并且slave-validity-factor为10,
     假设默认的repl-ping-slave-period是10秒，即如果超过310秒slave将不会尝试进行故障转移

 

```
cluster-migration-barrier 1
```

  master的slave数量大于该值，slave才能迁移到其他孤立master上，如这个参数若被设为2，
  那么只有当一个主节点拥有2 个可工作的从节点时，它的一个从节点会尝试迁移。

 

```
cluster-require-full-coverage yes
```

  默认情况下，集群全部的slot有节点负责，集群状态才为ok，才能提供服务。
  设置为no，可以在slot没有全部分配的时候提供服务。
  不建议打开该配置，这样会造成分区的时候，小分区的master一直在接受写请求，而造成很长时间数据不一致

 

```
slowlog-log-slower-than 10000
```

 slog log是用来记录redis运行中执行比较慢的命令耗时。
 当命令的执行超过了指定时间，就记录在slow log中，slog log保存在内存中，所以没有IO操作。
 执行时间比slowlog-log-slower-than大的请求记录到slowlog里面，单位是微秒，所以1000000就是1秒。
 注意，负数时间会禁用慢查询日志，而0则会强制记录所有命令。

 

```
slowlog-max-len 128
```

 慢查询日志长度。当一个新的命令被写进日志的时候，最老的那个记录会被删掉，这个长度没有限制。
 只要有足够的内存就行，你可以通过 SLOWLOG RESET 来释放内存

 

```
latency-monitor-threshold 0
```

  延迟监控功能是用来监控redis中执行比较缓慢的一些操作，用LATENCY打印redis实例在跑命令时的耗时图表。
  只记录大于等于下边设置的值的操作，0的话，就是关闭监视。
  默认延迟监控功能是关闭的，如果你需要打开，也可以通过CONFIG SET命令动态设置。

 

```
notify-keyspace-events ""
```

键空间通知使得客户端可以通过订阅频道或模式，来接收那些以某种方式改动了 Redis 数据集的事件。因为开启键空间通知功能需要消耗一些 CPU ，所以在默认配置下，该功能处于关闭状态。
 notify-keyspace-events 的参数可以是以下字符的任意组合，它指定了服务器该发送哪些类型的通知：
  K 键空间通知，所有通知以 __keyspace@__ 为前缀
  E 键事件通知，所有通知以 __keyevent@__ 为前缀
  g DEL 、 EXPIRE 、 RENAME 等类型无关的通用命令的通知
  $ 字符串命令的通知
  l 列表命令的通知
  s 集合命令的通知
  h 哈希命令的通知
  z 有序集合命令的通知
  x 过期事件：每当有过期键被删除时发送
  e 驱逐(evict)事件：每当有键因为 maxmemory 政策而被删除时发送
  A 参数 g$lshzxe 的别名
 输入的参数中至少要有一个 K 或者 E，否则的话，不管其余的参数是什么，都不会有任何 通知被分发。

 

```
hash-max-ziplist-entries 512
```

 hash类型的数据结构在编码上可以使用ziplist和hashtable。
 ziplist的特点就是文件存储(以及内存存储)所需的空间较小,在内容较小时,性能和hashtable几乎一样。
 因此redis对hash类型默认采取ziplist。如果hash中条目的条目个数或者value长度达到阀值,将会被重构为hashtable。
 这个参数指的是ziplist中允许存储的最大条目个数，，默认为512，建议为128

 

```
hash-max-ziplist-value 64
```

 ziplist中允许条目value值最大字节数，默认为64，建议为1024

 

```
list-max-ziplist-size -2
```

当取正值的时候，表示按照数据项个数来限定每个quicklist节点上的ziplist长度。比如，当这个参数配置成5的时候，表示每个quicklist节点的ziplist最多包含5个数据项。
当取负值的时候，表示按照占用字节数来限定每个quicklist节点上的ziplist长度。这时，它只能取-1到-5这五个值，每个值含义如下：
    -5: 每个quicklist节点上的ziplist大小不能超过64 Kb。（注：1kb => 1024 bytes）
    -4: 每个quicklist节点上的ziplist大小不能超过32 Kb。
    -3: 每个quicklist节点上的ziplist大小不能超过16 Kb。
    -2: 每个quicklist节点上的ziplist大小不能超过8 Kb。（-2是Redis给出的默认值）
    -1: 每个quicklist节点上的ziplist大小不能超过4 Kb。

 

```
list-compress-depth 0
```

这个参数表示一个quicklist两端不被压缩的节点个数。
注：这里的节点个数是指quicklist双向链表的节点个数，而不是指ziplist里面的数据项个数。
实际上，一个quicklist节点上的ziplist，如果被压缩，就是整体被压缩的。
参数list-compress-depth的取值含义如下：
    0: 是个特殊值，表示都不压缩。这是Redis的默认值。
    1: 表示quicklist两端各有1个节点不压缩，中间的节点压缩。
    2: 表示quicklist两端各有2个节点不压缩，中间的节点压缩。
    3: 表示quicklist两端各有3个节点不压缩，中间的节点压缩。
    依此类推…
由于0是个特殊值，很容易看出quicklist的头节点和尾节点总是不被压缩的，以便于在表的两端进行快速存取。

 

```
set-max-intset-entries 512
```

 数据量小于等于set-max-intset-entries用intset，大于set-max-intset-entries用set

 

```
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
```

 数据量小于等于zset-max-ziplist-entries用ziplist，大于zset-max-ziplist-entries用zset

 

```
hll-sparse-max-bytes 3000
```

  value大小小于等于hll-sparse-max-bytes使用稀疏数据结构（sparse）
  大于hll-sparse-max-bytes使用稠密的数据结构（dense），一个比16000大的value是几乎没用的，
  建议的value大概为3000。如果对CPU要求不高，对空间要求较高的，建议设置到10000左右

 

```
activerehashing yes
```

  Redis将在每100毫秒时使用1毫秒的CPU时间来对redis的hash表进行重新hash，可以降低内存的使用。
  当你的使用场景中，有非常严格的实时性需要，不能够接受Redis时不时的对请求有2毫秒的延迟的话，把这项配置为no。
  如果没有这么严格的实时性要求，可以设置为yes，以便能够尽可能快的释放内存

 

```
client-output-buffer-limit normal 0 0 0
```

 对客户端输出缓冲进行限制可以强迫那些不从服务器读取数据的客户端断开连接，用来强制关闭传输缓慢的客户端。
 对于normal client，第一个0表示取消hard limit，第二个0和第三个0表示取消soft limit，normal client默认取消限制，因为如果没有寻问，他们是不会接收数据的

 

```
client-output-buffer-limit slave 256mb 64mb 60
```

 对于slave client和MONITER client，如果client-output-buffer一旦超过256mb，又或者超过64mb持续60秒，那么服务器就会立即断开客户端连接。

 

```
client-output-buffer-limit pubsub 32mb 8mb 60
```

 对于pubsub client，如果client-output-buffer一旦超过32mb，又或者超过8mb持续60秒，那么服务器就会立即断开客户端连接。



```
hz 10
```

  redis执行任务的频率为1s除以hz

 

```
aof-rewrite-incremental-fsync yes
```

  在aof重写的时候，如果打开了aof-rewrite-incremental-fsync开关，系统会每32MB执行一次fsync。
  这对于把文件写入磁盘是有帮助的，可以避免过大的延迟峰值