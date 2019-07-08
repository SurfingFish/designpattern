**1：搜索某个文件里面是否包含字符串，使用grep "search content" filename1， 例如**

```shell
$  grep appendonly no ./redis.conf
```



**2、通过sed命令直接修改文件内容**，例如：

```shell
$ sed -i 's/appendonly no/appendonly yes/g' ./redis.conf
```

在进行修改之前，要确定文件内被修改的只有一个匹配项，不然就会把其他的地方也修改了



**3、根据文件名查找文件**

```shell
$ find /usr/local/src/redis/redis-3.2.0 -name 'appendonly.aof'
```



**4、查看网络状态**

```
netstat -n | awk '/^tcp/{++S[$NF]}END{for(a in S) print a S[a]}'

ESTABLISHED10

TIME_WAIT9

上面是用来查看服务器中有多少TCP连接处于ESTABLISHED 和TIME_WAIT的

如果CLOSE_WAIT 有几千的话，就表示程序有问题了 CLOSE_WAIT和文件句柄一一对应 CLOSE_WAIT如果一直被保持，就表示对应数据的通道一直被占用着，一旦达到了上限，则新的请求无法被处理，会造成软件服务器的崩溃
```

 

