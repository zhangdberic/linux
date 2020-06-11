# linux command

## 查看

### 过滤掉注释

```bash
egrep -v '^$|#' /etc/zabbix/zabbix_agentd.conf
```

## 网络

### 查看所有的监听端口

```bash
netstat -tuln
```

### 查看tcp连接

```bash
netstat -ant
```

### 查看udp连接

```bash
netstat -nua 
```

### ssh远程连接

ssh -p 22 root@10.60.32.197

### 防火墙(firewalld)

#### 基本命令

查看服务状态

systemctl status firewalld.service  安装centos后默认就开启

查看防火墙状态

systemctl status firewalld

启动防火墙

systemctl start firewalld

关闭防火墙

systemctl stop firewalld

重新启动防火墙

systemctl restart firewalld

查看当前规则

firewall-cmd --list-all

开启启动

systemctl enable firewalld

#### 允许外界访问某个端口

firewall-cmd --permanent --add-port=80/tcp

firewall-cmd --reload

####  移除这个规则(不允许访问某个端口)

firewall-cmd --permanent --remove-port=80/tcp

firewall-cmd --reload

####  允许某个网段访问某个端口

firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="172.17.0.0/16" port protocol="tcp" port="8070" accept"

firewall-cmd --reload

####  保存规则

目前已知情况是，命令执行后马上保持，保持到/etc/firewalld/zones/public.xml。

vi /etc/firewalld/zones/public.xml

### curl

#### 计算base64

```
echo -n "用户名：密码" | base64
```

#### 发送认证请求

```
curl -v -H "Authorization: Basic 认证信息" -X GET https://docker.yun.ccb.com/v2/<name>/tags/list
```



## 文件

### 遍历文件操作

```sh
#! /bin/sh -
for file in `ls`
do
  jar xf $file
done
```

## 服务

查看所有的服务

```shell
systemctl list-unit-files
```

启动服务

```
systemctl start xxxx
```

停止服务

```
systemctl stop xxxx
```

重启服务

```
systemctl restart xxxx
```

开机启动

```
systemctl enable xxxx
```

禁止开机启动

```
systemctl disable xxx
```



## 用户

修改用户uid和gid

例如：

 #test用户uid改为100,gid改为101

```
usermod -u 100 test && groupmod -g 101 test
```

## 进程

### kill

当你调用kill命令时，不只是使用kill -9 pid，其实还是有很多的数字可以代替9的而且更可以更优雅的关闭进程。

下面列出了所有的数字和对于的意义：

```
1) SIGHUP     2) SIGINT     3) SIGQUIT     4) SIGILL     5) SIGTRAP
6) SIGABRT     7) SIGBUS     8) SIGFPE     9) SIGKILL    10) SIGUSR1
11) SIGSEGV    12) SIGUSR2    13) SIGPIPE    14) SIGALRM    15) SIGTERM
16) SIGSTKFLT    17) SIGCHLD    18) SIGCONT    19) SIGSTOP    20) SIGTSTP
21) SIGTTIN    22) SIGTTOU    23) SIGURG    24) SIGXCPU    25) SIGXFSZ
26) SIGVTALRM    27) SIGPROF    28) SIGWINCH    29) SIGIO    30) SIGPWR
31) SIGSYS    34) SIGRTMIN    35) SIGRTMIN+1    36) SIGRTMIN+2    37) SIGRTMIN+3
38) SIGRTMIN+4    39) SIGRTMIN+5    40) SIGRTMIN+6    41) SIGRTMIN+7    42) SIGRTMIN+8
43) SIGRTMIN+9    44) SIGRTMIN+10    45) SIGRTMIN+11    46) SIGRTMIN+12    47) SIGRTMIN+13
48) SIGRTMIN+14    49) SIGRTMIN+15    50) SIGRTMAX-14    51) SIGRTMAX-13    52) SIGRTMAX-12
53) SIGRTMAX-11    54) SIGRTMAX-10    55) SIGRTMAX-9    56) SIGRTMAX-8    57) SIGRTMAX-7
58) SIGRTMAX-6    59) SIGRTMAX-5    60) SIGRTMAX-4    61) SIGRTMAX-3    62) SIGRTMAX-2
63) SIGRTMAX-1    64) SIGRTMAX
```

下面重点介绍几个重要的：

#### kill -2 pid

kill -2 pid，模拟ctrl+c，发送SIGINT信号，大部分程序可以相应SIGINT信号，例如：springboot程序，但其有个问题，如果你的spring boot程序是基于nohup & 后台运行的，则不会响应SIGINT信号也就说不会响应ctrl+c。这个需要kill -15 pid来代替。

#### kill - 15 pid

kill -15 pid，发送SIGTERM信号，程序可以选择响应这个信号情况，也可以不响应。spring boot可以响应这个信号，其在spring boot application注册的时候会注册一个钩子线程，钩子响应SIGTERM信号，实现优雅的退出。



## CURL

### POST提交

格式：

```shell
curl 请求url -X POST -d 'POST内容'
```

例如：

```
curl localhost:3000/api/basic -X POST -d 'hello=world'
```

### 加入请求头

加入--header 'name:value'

例如：

```
--header "User-Agent:Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36"
```

### 显示请求头和响应头信息

加入-v参数

例如：

```
curl localhost:3000/api/basic -X POST -d 'hello=world' -v
```

