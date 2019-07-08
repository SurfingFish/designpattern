今天整了一个虚拟机，里面安装了Redis想用来学习一下，结果在使用Redis Destop Manager连接虚拟机的Redis的时候被坑了。



排查步骤：

1、检查宿主机的防火墙(本机电脑防火墙)

2、检查虚拟机的防火墙

3、使用宿主机ping虚拟机的IP是否能ping通

4、使用虚拟机ping宿主机的IP是否能ping通

5、确认虚拟机中的redis服务是否已经启动了

6、在宿主机打开cmd 输入telnet 192.168.80.128(虚拟机IP)   6379(redis端口)   不报错且显示为黑屏，则表示宿主机可以连接到虚拟机的端口

7、是否在redis的配置文件中加了requirepass ，因为使用Redis Destop Manager 连接虚拟机的redis时需要密码

8、(额外步骤  将虚拟机的IP地址设置为静态IP 就不需要在每次连接的时候修改IP地址   <https://blog.csdn.net/yuioplv/article/details/79631745>)  

9、查看一下是否开启了代理上网，不要开代理！不要开代理！不要开代理！

