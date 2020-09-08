# redis

## 安装和配置

### 安装

su - root

yum install  -y  gcc

useradd redis

su - redis

mkdir soft

cd soft

wget http://download.redis.io/releases/redis-5.0.8.tar.gz   # 上redis官网看好版本,再下载

tar xvf redis-5.0.8.tar.gz 

mv redis-5.0.8 ~/redis

cd ~/redis

make

### 配置

#### 系统

vi /usr/lib/sysctl.d/00-system.conf

```properties
net.core.somaxconn = 511
vm.overcommit_memory = 1
```

sysctl -p

执行

echo never > /sys/kernel/mm/transparent_hugepage/enabled

chmod +x /etc/rc.d/rc.local

vi /etc/rc.d/rc.local

```properties
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

#### redis.conf

##### 存内存模式

vi ~/redis/redis.conf

```properties
bind 127.0.0.1  # 注释bind，则监听0.0.0.0，但有点安全隐患，在keepalived环境必须。
# bind 127.0.0 本机ip  # 多个ip用空格分隔 127.0.0.1应该使用允许访问
daemonize yes   # 基于后台允许
databases 100   # 允许的最大数据库数量
#save 900 1     # 禁止持久化
#save 300 10
#save 60 10000
save ""
maxmemory 5368709120   # 允许使用的最大内存空间
# 当redis中内存数据达到maxmemory时,触发"清除策略"，volatile-lru  ->对"过期集合"中的数据采取LRU(近期最少使用)算法.如果对key使用"expire"指令指定了过期时间,那么此key将会被添加到"过期集合"中。将已经过期/LRU的数据优先移除.如果"过期集合"中全部移除仍不能满足内存需求,将OOM.
maxmemory-policy volatile-lru   
requirepass xxxxxx   # 秘钥
```



## redis命令

**进入控制台**
~/redis/src/redis-cli

**启动redis**
~/redis/src/redis-server ~/redis/redis.conf 

**停止redis**
~/redis/src/redis-cli shutdown

**查看redis运行信息**
~/redis/src/redis-cli info

**查看redis版本**
~/redis/src/redis-server -v

**远程操作**

~/redis/src/redis-cli -h ip -p port -a 秘钥 -n database

