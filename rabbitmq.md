# rabbitmq

## 安装和配置

### 安装erlang

root用户下安装erlang

**依赖**

yum -y install make gcc gcc-c++ kernel-devel m4 ncurses-devel openssl-devel perl unixODBC unixODBC-devel zip unzip xmlto

**查看erlang和rabbitmq版本是否匹配**

https://www.rabbitmq.com/which-erlang.html

otp_src_22.3

rabbitmq 3.7.26

**下载**

cd /software

wget http://erlang.org/download/otp_src_22.3.tar.gz

tar xvf otp_src_22.3.tar.gz

cd otp_src_22.3

**配置安装**

./configure --prefix=/usr/local/erlang --with-ssl -enable-threads -enable-smmp-support -enable-kernel-poll --enable-hipe --without-javac

出现如果警告可以忽略：

```
./configure: line 4640: wx-config: command not found
configure: WARNING:
                wxWidgets must be installed on your system.

		Please check that wx-config is in path, the directory
		where wxWidgets libraries are installed (returned by
		'wx-config --libs' or 'wx-config --static --libs' command)
		is in LD_LIBRARY_PATH or equivalent variable and
		wxWidgets version is 2.8.4 or above.

*********************************************************************
**********************  APPLICATIONS DISABLED  **********************
*********************************************************************

jinterface     : Java compiler disabled by user

*********************************************************************
*********************************************************************
**********************  APPLICATIONS INFORMATION  *******************
*********************************************************************

wx             : No OpenGL headers found, wx will NOT be usable
No GLU headers (glu.h) found, wx will NOT be usable
wxWidgets not found, wx will NOT be usable

*********************************************************************
*********************************************************************
**********************  DOCUMENTATION INFORMATION  ******************
*********************************************************************

documentation  : 
                 fop is missing.
                 Using fakefop to generate placeholder PDF files.

*********************************************************************
```

make -j2

make install

安装后目录：

```
/usr/local/erlang
```

**配置profile**
vi /etc/profile

尾部加入如下内容：
ERLANG_HOME=/usr/local/erlang
PATH=$PATH:$ERLANG_HOME/bin

使其生效：source /etc/profile

**检验erl**
erl

### 安装Rabbitmq

useradd rabbitmq

su - rabbitmq

tar xvf rabbitmq-server-generic-unix-3.7.26.tar.xz

mv rabbitmq_server-3.7.26 rabbitmq

cd rabbitmq

### 设置rabbitmq持久化消息的位置

su - root
mkdir /home/rabbitmq/rabbitmq_data
mount 逻辑卷(快盘) /home/rabbitmq/rabbitmq_data
chown rabbitmq:rabbitmq /home/rabbitmq/rabbitmq_data

vi /etc/fstab
添加内容：
逻辑卷路径 /home/rabbitmq/rabbitmq_data     ext4    defaults,noatime,nodiratime        0 0

设置rabbitmq的环境变量
su - rabbitmq
mkdir -p /home/rabbitmq/rabbitmq_data/mnesia
mkdir -p /home/rabbitmq/rabbitmq_data/logs

vi ~/rabbitmq/etc/rabbitmq/rabbitmq-env.conf 
内容如下：
MNESIA_BASE=/home/rabbitmq/rabbitmq_data/mnesia
LOG_BASE=/home/rabbitmq/rabbitmq_data/logs

更改日志和mnesia数据库的位置，mnesia存放消息队列信息持久化的位置，可以放在一个稳定的快盘上。

### rabbitmq初次配置

先启动rabbitmq
su - rabbitmq
cd rabbitmq
cd sbin/
./rabbitmq-server &

添加web界面管理插件
./rabbitmq-plugins enable rabbitmq_management



修改默认的guest密码(安全考虑)

./rabbitmqctl change_password guest password



添加管理员用户
./rabbitmqctl add_user admin password
admin为用户名，password为密码

设置管理员用户角色(administrator)
./rabbitmqctl set_user_tags admin administrator
admin为用户名，administrator为管理员角色

设置管理员用户可以访问的虚拟机(/)和配置、写、读队列名，这里'.*'为所有队列。

```
./rabbitmqctl  set_permissions -p / admin  '.*' '.*' '.*'
```



添加一个普通用户

./rabbitmqctl add_user zhangdb password

zhangdb为用户名，password为密码

设置普通用户角色(other)
./rabbitmqctl set_user_tags zhangdb other
admin为用户名，other角色(不允许登录管理界面,只能发送和接收消息)

如下普通用户可以访问的虚拟机(/)和配置、写、读队列名，这里'.*'为所有队列。

```
./rabbitmqctl  set_permissions -p / zhangdb  '.*' '.*' '.*'
```



添加stomp协议插件
./rabbitmq-plugins enable rabbitmq_stomp
启动后，会监听61613端口

### 开启防火墙

```
5672 amqp端口
15672 web管理端口
61613 stomp端口
25672 集群通信端口
4369 epmd端口
```



## rabbitmq命令

**启动**
su - rabbitmq
cd rabbitmq_server-3.7.26/
cd sbin/
./rabbitmq-server &

**停止**
./rabbitmqctl stop

**查看状态**
./rabbitmqctl -q status

**管理界面URL**
http://IP:15672/



## 配置文件

```
默认请求下，rabbitmq没有创建rabbitmq.config配置文件，如果需要配置则需要手工创建这个文件，这个文件的位置：rabbitmq的home目录的/etc/rabbitmq/rabbitmq.config，例如：/home/rabbitmq/rabbitmq_server-3.7.26/etc/rabbitmq/rabbitmq.config
配置格式如下：
[
  {rabbit, [{tcp_listeners, [5672]},{hipe_compile, true},{vm_memory_high_watermark, 0.4}]}
].
或
[
  {rabbit,[{tcp_listeners, [5672]},{rabbitmq_stomp,[{tcp_listen_options,[{backlog,4096},{nodelay,true},{exit_on_close,true},{sndbuf,196608},{recbuf,196608},{keepalive,true},{send_timeout,10000}]}]},{vm_memory_high_watermark, 0.7}]}
].
```

## 压力测试

基于openresty+stomp客户端，发送2000000个请求，500并发请求下，发送内容大小为900个字节，每秒可以处理22415个请求，但还是受到服务网关lua+redis代码和nginx绑定6个cpu限制，如果就是简单openresty+stomp代码，效率能更好。
Rabbitmq不同的队列配置，性能差距比较大，例如：就是简单的基于内存，不持久化，不镜像，那么每秒35000处理个请求，而基于内存的HA镜像队列每秒5000多个请求，基于内存和持久化10000多个请求。
例如：2000000请求500并发，基于内存，不持久化，不镜像，压力测试结果22177/s，客户端send配置的timeout=2000不报错（说明延时低）。

## rabbitmq admin

### 状态说明

#### ready

ready状态的消息是，队列中堆积待处理的消息。

#### unacked

unacked状态消息是，接收器已经接收到了消息，但没有确认ack（设置为了手工确认模式）的消息，这样的消息及时你的程序处理了，但没有确认还是不会从队列中删除，消息的状态变为了unacked。再有就是你手工拒绝(basicReject)重新返回到队列中的消息。但如果这是你把消费端关闭，则状态还好切换回ready了，等待下一次重新消费处理。

# rabbitmq理论

rabbitmq实现了AMQP协议，而AMQP最好的地方，就是在于生产者和消费者消息路由进一步解耦。传统的消息队列，是生产者直接发送请求到队列(一对一)，或者直接发送请求到主题(一对多)。如下图：

```
生产者----->队列<-----消费者
或者
生产者----->   <-----消费者1
           主题
              <-----消费者2

```

AMQP协议要求生产者只能发送请求到exchange(交换机)，交换机根据路由规则转发消息到队列，消费者只能消费某个队列。如下图：

```
生产者----->交换机(exchange)--路由规则(binding)-->队列(queue)<-----消费者
```

**因为中间加入一层路由规则，把交换机和队列解耦。**

## 路由规则bindings

bindings是一个概念(绑定)，其负责绑定exchange(交换机)到queue(队列)，实现消息路由。创建一个binding(绑定规则)，需要起一个名字，这个名字就是routing key。因为在exchange(交换机)和queue(队列)之间加入了一层，这样就会实现灵活的路由规则。

to queue

创建exchange消息到queue的binding(路由)，给这个binding起名，例如：toQueue1，这里名称不叫name，叫routing key。

to exchange

创建exchange消息到另一个exchange的binding(路由)，给这个binding起名，例如：toExchange1，这里名称不叫name，叫routing key。

发送消息时指定exchange和routing key，而不是直接指定队列名，这样你可以通过修改routing key规则(无效修改routing key名称)来实现：

1.转发到不同的队列。

2.队列模式转换到主题模式，例如：原routing key起名为rk1，规则绑定到queue1，而现在再创建一个同名rk1，规则绑定到queue2。

这也就是为什么，生产者在发送消息的时候，需要指定exchange和routing key参数。

## AMQP四种exchange

Direct：如果发送消息头的routing key与binding的routing key**直接匹配**的话，消息将会路由到该队列上。

Topic：如果发送消息头的routing key与binding的routing key**通配符匹配**的话，消费会转发到匹配成功的所有队列上。

~~Default~~：一个特殊的Exchange。它会将消息路由到名字和消息routing key相同的队列上，你无需创建binding。和传统的队列模式相同。

~~Fanout~~：不管生产者发送消息的routing key是什么，消息都会路由到所有绑定的队列上。和传统的主题默模式相同。





