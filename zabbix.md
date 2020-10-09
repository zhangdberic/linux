# zabbix5.0

## 安装

### 服务器端

#### 安装

##### 准备

**centos7以上**

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

set setenforce 0

##### 安装zabbix_server

**下载zabbix**

rpm -ivh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm

**编辑zabbix.repo指向国内源**

vi /etc/yum.repos.d/zabbix.repo

修改yum配置文件,指向阿里云镜像

```
[zabbix]
name=Zabbix Official Repository - $basearch
baseurl=https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-frontend]
name=Zabbix Official Repository frontend - $basearch
baseurl=https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/$basearch/frontend
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-debuginfo]
name=Zabbix Official Repository debuginfo - $basearch
baseurl=https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/$basearch/debuginfo/
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591
gpgcheck=1

[zabbix-non-supported]
name=Zabbix Official Repository non-supported - $basearch
baseurl=https://mirrors.aliyun.com/zabbix/non-supported/rhel/7/$basearch/
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
gpgcheck=1

```

**安装zabbix服务器端和mysql连接器**

yum -y install zabbix-server-mysql zabbix-agent

**安装centos软件集合库**

yum -y install centos-release-scl

**安装zabbix前端软件**

vi /etc/yum.repos.d/zabbix.repo

修改[zabbix-frontend]的enabled=1

yum -y install zabbix-web-mysql-scl zabbix-nginx-conf-scl

##### 安装mariadb

yum install  mariadb-server -y

**启动mariadb**

systemctl start mariadb && systemctl enable mariadb && systemctl status mariadb

**设置mariadb的root账号**

mysqladmin -u root password '12345678'

创建zabbix需要的库和表结构

注意'password'为zabbix用户的密码，这个生产环境必须修改。

```
# mysql -uroot -p
password
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
```

**导入zabbix表结果和数据**

zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix

**配置zabbix服务器连接数据库密码**

这个密码是上面mysql中zabbix用户的密码。

vi /etc/zabbix/zabbix_server.conf

```
DBPassword=password
```

##### 配置前置nginx

vi /etc/opt/rh/rh-nginx116/nginx/conf.d/zabbix.conf

去掉这两项的注释，并且根据实际情况编辑。

```nginx
        listen          80;
        server_name     location;
```



vi /etc/opt/rh/rh-nginx116/nginx/nginx.conf

编辑nginx.conf文件，去掉server部分，因为include的/etc/opt/rh/rh-nginx116/nginx/conf.d/zabbix.conf文件已经包含了zabbix要使用的server部分配置。

删除include /etc/opt/rh/rh-nginx116/nginx/conf.d/*.conf;配置后的所有配置，例如：

```nginx
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/opt/rh/rh-nginx116/log/nginx/error.log;
pid /var/opt/rh/rh-nginx116/run/nginx/nginx.pid;

# Load dynamic modules. See /opt/rh/rh-nginx116/root/usr/share/doc/README.dynamic.
include /opt/rh/rh-nginx116/root/usr/share/nginx/modules/*.conf;

events {
    worker_connections  1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/opt/rh/rh-nginx116/log/nginx/access.log  main;

    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;
    keepalive_timeout  65;
    types_hash_max_size 2048;

    include       /etc/opt/rh/rh-nginx116/nginx/mime.types;
    default_type  application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/opt/rh/rh-nginx116/nginx/conf.d/*.conf;

}

```

##### 配置前置php

配置zabbix.conf加入nginx

vi /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf

加入nginx

```
listen.acl_users = apache,nginx
```

修改为时区，修改为如下：

```
php_value[date.timezone] = Asia/Shanghai
```

##### 启动zabbix

```
systemctl restart zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm
systemctl enable zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm
```

##### 开放防火墙的80和10051端口（待验证）

```
firewall-cmd --permanent --add-port={80/tcp,10051/tcp}
firewall-cmd --reload
```

##### 安装配置

###### 初始化配置

直接访问zabbix服务器，如果页面还是nginx的官方首页则可以上面的nginx.confi配置不对，或者浏览器有缓存(ctrl+f5)需要刷出。

http://192.168.5.15/

第一个页面，下一步；

第二个页面，下一步；

第三个页面（Configure DB connection)，最后的Password为mysql的zabbix用户密码，例如默认为"password"，如果数据库不和zabbix在同机器，则要修改上面更多链接到mysql的信息；

第四个页面（Zabbix server details），最后的Name为zabbix命名，例如：zbxserver

最后一页，配置成功为：Congratulations! You have successfully installed Zabbix frontend.

###### 默认的登录用户密码和密码

Admin/zabbix，区别大小写。

###### 修改Admin密码

User settings菜单->点击Change password

###### 修改到中文

前提操作系统是中文环境，这个可以参加centos的安装文档。

User settings菜单->Language Chinese(zh_CN)；

修改字体，否则乱码：

cd /usr/share/zabbix/assets/fonts

mv graphfont.ttf graphfont.ttf.bak

windows操作系统，拷贝C:\Windows\Fonts微软雅黑字体(MSYH.TTF)到这个目录(/usr/share/zabbix/assets/fonts)，然后改名：

mv MSYH.TTF graphfont.ttf

改名后无需重启zabbix服务器，直接刷新页面就可以看到新字体了。

##### 验证安装是否成功

进入zabbix主页面->监测->主机->看Zabbix server的可用性项目ZBX是否为绿色。

**如果ZBX不为绿色**，则：

1.查看10050端口(agent)是否存在，netstat -ntlp

2.查看zabbix服务器端的agent是否正常启动，你可以使用systemctl restart zabbix-agent.service，看是否报错。

3.使用命令egrep -v '^$|#' /etc/zabbix/zabbix_agentd.conf，查看zabbix_agentd.conf文件是否存在，并且内容是否正确，参见：客户端zabbix_agent配置，服务器端agent和客户端agent的配置是一样的。

##### zabbix-get

**安装**

yum install zabbix-get.x86_64

**使用**

获取被监控主机上监控项目值

格式：

```
zabbix_get -s 被监控主机 -p zabbix_agent端口(默认10050) -k "监控项目"
```

具体的监控项目可以查看界面上模板的监控项

例如：

zabbix_get -s 192.168.5.54 -p 10050 -k "system.cpu.load[all,avg1]"   # 获取cpu负载

zabbix_get -s 192.168.5.54 -k system.cpu.num   # 获取cpu个数



### 客户端

#### 准备

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

set setenforce　0

#### centos7安装

下载zabbix

rpm -ivh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm

vi /etc/yum.repos.d/zabbix.repo

修改yum配置文件,指向阿里云镜像

```
[zabbix]
name=Zabbix Official Repository - $basearch
baseurl=https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-frontend]
name=Zabbix Official Repository frontend - $basearch
baseurl=https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/$basearch/frontend
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-debuginfo]
name=Zabbix Official Repository debuginfo - $basearch
baseurl=https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/$basearch/debuginfo/
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591
gpgcheck=1

[zabbix-non-supported]
name=Zabbix Official Repository non-supported - $basearch
baseurl=https://mirrors.aliyun.com/zabbix/non-supported/rhel/7/$basearch/
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
gpgcheck=1

```



yum -y install zabbix-agent zabbix-sender

systemctl enable zabbix-agent.service



vi /etc/zabbix/zabbix_agentd.conf 

Server=192.168.65.1 # zabbix server

#ServerActive=127.0.0.1 # 注释掉

Hostname=192.168.5.54  # 当前客户端主机名,这里为了方便记忆使用了本机ip地址，注意：服务器端创建主机的时候，主机名必须和这项相同。

egrep -v '^$|#' /etc/zabbix/zabbix_agentd.conf

```
PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=192.168.5.15
Hostname=192.168.5.54
Include=/etc/zabbix/zabbix_agentd.d/*.conf

```

systemctl restart zabbix-agent.service

**验证是否启动**

netstat -ntlp|grep 10050

```
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      9148/zabbix_agentd
```

**开放防火墙的10051端口**

```
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="zabbix服务器ip" port protocol="tcp" port="10050" accept"
firewall-cmd --reload
```

**测试zabbix是否可以访问客户端zabbix_agent**

**zabbix服务器端执行如下命令**，其会发生请求到客户端，来获取客户端cpu的个数

```
zabbix_get -s 客户端ip地址 -k system.cpu.num
```



#### centos6安装

**删除原来的zabbix_agentd版本**

yum remove zabbix-agent -y

yum remove zabbix-sender -y

rpm -e zabbix-release-3.4-1.el6.noarch

rpm -e zabbix-release-3.4-2.el7.noarch

rm -rf /etc/yum.repos.d/zabbix.repo

**安装新版本**

rpm -ivh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/6/x86_64/zabbix-release-5.0-1.el6.noarch.rpm

vi /etc/yum.repos.d/zabbix.repo

```
[zabbix]
name=Zabbix Official Repository - $basearch
baseurl=https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-frontend]
name=Zabbix Official Repository frontend - $basearch
baseurl=https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/6/$basearch/frontend
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-debuginfo]
name=Zabbix Official Repository debuginfo - $basearch
baseurl=https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/6/$basearch/debuginfo/
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591
gpgcheck=1

[zabbix-non-supported]
name=Zabbix Official Repository non-supported - $basearch
baseurl=https://mirrors.aliyun.com/zabbix/non-supported/rhel/6/$basearch/
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
gpgcheck=1

```



yum clean all

yum -y install zabbix-agent



vi /etc/zabbix/zabbix_agentd.conf 

Server=192.168.65.1 # zabbix server(起到的作用是限制能访问本agent的服务器ip地址，你可以指定为一个掩码端或0.0.0.0)

#ServerActive=127.0.0.1 # 注释掉

Hostname=192.168.5.54  # 当前客户端主机名,这里为了方便记忆使用了本机ip地址，注意：服务器端创建主机的时候，主机名必须和这项相同。

egrep -v '^$|#' /etc/zabbix/zabbix_agentd.conf

```
PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=192.168.5.15
Hostname=192.168.5.30
Include=/etc/zabbix/zabbix_agentd.d/*.conf

```

service zabbix-agent start

chkconfig zabbix-agent on

**验证是否启动**

netstat -ntlp|grep 10050

**开启防火墙**

vi /etc/sysconfig/iptables

```
-A INPUT -s zabbix服务器ip -m state --state NEW -m tcp -p tcp --dport 10050 -j ACCEPT
```

service iptables restart

**测试zabbix是否可以访问客户端zabbix_agent**

**zabbix服务器端执行如下命令**，其会发生请求到客户端，来获取客户端cpu的个数

```
zabbix_get -s 客户端ip地址 -k system.cpu.num
```

## ZABBIX界面

### 监测

#### 仪表盘

直观的显示整个zabbix监控主机的情况，其是一个portal页面，你可以自定义要展现的东西。

而且一般盘可以是多个，多个仪表盘基于tab显示，多个仪表盘：

**问题仪表盘**：重点显示所有主机的问题和动作。

**重点监控主机仪表盘**：你可以把一些重要的监控主机和监控项基于图表的形式，放在仪表盘上。

#### 问题

当前监控项的值满足了触发器的条件表达式（表达式计算结果为true），就变成了问题。

例如：CPU负载触发器，设置表达式：CPU负载>=60%，如果被监控的主机CPU当前负载为70%的时候，表达式计算结果就是true了，就变成了问题。

有些问题是可以自动恢复的，有些问题需要手工确认。例如：有段时间内CPU负载高，则CPU负载监控触发器被触发，页面出现警告，但过一段负载下来后又恢复了，这样问题是可以自动恢复的。但如果你修改了/etc/passwd文件(例如执行useradd abc)，则会触发/etc/passwd文件改变触发器，这种就没有自动恢复，需要你手工确认这个问题。

你可以通过问题页面来查看目前所有的问题信息：

严重性：为分类、信息、警告、一般严重、严重、灾难，问题程度越来越严重。

恢复时间：恢复问题的时间。

状态：问题、已解决。

信息：问题具体信息。

主机：发生问题的主机。

问题：问题。

持续时间：问题处理设置的持续时间。

确认：问题是否被确认(我知晓这个问题了)，自动恢复不会自动确认，确认必须手工完成（表明为知晓）。

动作：问题发生后，执行了那些操作，例如：发送警告邮件，重启服务器(远程命令)等。

标记：不明白。

**注意：一般情况下，发生问题和恢复是两条记录，恢复时间和状态为已解决的是恢复记录。当问题都是同一个问题，第一个是问题记录，第二个问题恢复记录。**

#### 主机

显示当前所有被监控的主机，ZBX图标显示被监控主机Zabbix_agent的是否在工作，也就是说是否可以和zabbix_agent通讯。

最新数据：显示所有最后一次从被监控主机上获取到的监控项数据。

问题：显示当前还没被确认的问题个数。

### 报表

#### 触发器 Top 100

通过这个功能可以看到那些触发器经常被触发，间接的发现那些问题所在，工作是不是不到位等。

#### 动作日志

这个功能比较重要，因为触发器被触发的时候，一般都要配置相应的处理**动作**，本功能就是看动作执行的日志和情况。

### 配置

#### 主机群组

对主机进行分组，例如：政务专网、政务外网，然后把不同的主机划分到不同的组下，方便管理。

#### 模板

把不同的监控项、触发器、图形等组合在一起形成一个模板，例如：windows操作系统监控模板、linux系统监控模板、nginx监控模板等。下面的主机部分会介绍到模板。

##### 监控项

配置->模板->选择一个模板（例如：Template OS Linux by Zabbix agent）->监控项

修改某个监控项的属性，举一个例子：点击某个一个监控项（例如：Checksum of /etc/passwd），修改刷新间隔(数据项采集时间间隔)，这个默认监控项默认是15m，我们修改为1m。

**测试验证**

到被监控的任何一台机器创建一个用户(例如：useradd abc)，其会间接的修改/etc/password文件，其会触发/etc/passwd has been changed，回到zabbix的主页监控仪表板查看情况。这个需要2分钟才可以体现出，因为你上面设置了Checksum of /etc/passwd的监控刷新1分钟，主页页面仪表盘问题项刷新也是1分钟，因此需要两分钟才会显示出问题。

##### 触发器

修改触发器的条件表达式，使其更符合你的实际情况下，例如：默认磁盘使用空间大于85%则触发，你可以修改为70%触发。

你也可以修改触发器的严重性，例如：从一般修改为严重。

##### 图形

有些默认的监控项没有提供图形，你可以自定义的来提供图形，例如：自定义的监控项基于图形显示。



#### 主机

前提被监控主机正确的安装和配置zabbix_agent（见客户端安装）。

配置->主机

##### 主机tab页

主机名称：必须与被监控主机配置文件/etc/zabbix/zabbix_agentd.conf内的Hostname相同。

客户端：IP地址：为客户端的ip地址，端口10050默认。

##### 模板tab页

点击Link new templates[选择]，选择Template，再选择Template OS Linux by Zabbix agent

点击，更新按钮。再点击，添加。

你可以为一台主机添加多个模板，例如：Template OS Linux by Zabbix agent 监控linux操作系统、Template App Nginx By Zabbix Agent 监控nginx。

##### 验证被监控主机是否添加成功

1.使用zabbix_get验证

在zabbix_server上执行，如下命令：-s后为zabbix客户端的ip地址。

zabbix_get -s 192.168.5.54 -p 10050 -k system.cpu.num

测试返回的客户端的cpu个数

2.zabbix监控页面上，监控->主机，查看ZBX图标是否为绿色，一般情况下60s内就有启作用，如果没有颜色或者为其它颜色，则需要查看/var/log/zabbix/zabbix_server.log的日志文件，看看是否有错误发生。

##### 图形监控个数

随着时间越来越长，监控获取到的数据越来越多，图片支持的监控项目也就越多。原因是想要图形显示则需要比较值，比较值则需要前后结果对比，则需要收集数据，而监控项收集数据是有时间间隔的，因此需要一段时间。



#### 动作

好文章：http://www.zsythink.net/archives/768

动作：理解为当触发器被触发的时候，执行某些操作。

下面举两个例子：

例子1：当被监控主机无法被监控到(宕机)，触发器({HOST.NAME} has been restarted (uptime < 10m))被触发，动作是发送邮件。触发条件如下，

```
{192.168.5.30:system.uptime.last()}<10m
```

例子2：当有人改变了/etc/passwd文件(例如：增加了一个用户)，触发器(/etc/passwd has been changed)被触发，动作是发送邮件。触发条件如下，

```
{192.168.5.30:vfs.file.cksum[/etc/passwd].diff()}>0
```

##### 动作tab

动作tab重点就是条件，也就是当满足条件的时候会执行动作，当然条件可以是一个也可以是多个组合，例如：一个条件，当某个触发器被触发的时候执行动作。多个条件，当某个主机的某个触发器被触发是执行动作。



##### 操作tab

当前动作条件满足的时候，执行的操作，操作可以是一个步骤也可以是多个步骤，多个步骤可以并行执行也可以串行执行，其是**由步骤值**决定的。

**默认操作步骤持续时间**：每执行一个步骤的时间，默认值为1h(1个小时)，如果你的动作就一个步骤这个值没什么意义，如果是多个步骤，那么第1个步骤执行后1个小时才会执行第2个步骤，以此类推。1m为一分钟，1s为一秒钟。

**暂停操作以制止问题**：这个目前没搞懂，默认选中，不用改。

**操作**

###### 发消息

**操作类型**：发消息

**步骤**：这个不好理解，要细品，步骤值有两个，值1和值2配合使用，可以确定步骤执行的先后和时间，我们通过下面的例子来分析，下面添加了3个步骤：

| 步骤  | 细节                                                         | 开始于   | 持续时间 | 动作     |
| :---- | :----------------------------------------------------------- | :------- | :------- | :------- |
| 1     | **发送消息给用户:** Admin (Zabbix Administrator) 通过 163email | 立即地   | 默认     | 编辑移除 |
| 1 - 2 | **发送消息给用户:** Admin (Zabbix Administrator) 通过 163email | 立即地   | 默认     | 编辑移除 |
| 1 - 3 | **发送消息给用户:** Admin (Zabbix Administrator) 通过 163email | 立即地   | 默认     | 编辑移除 |
| 2     | **发送消息给用户:** Admin (Zabbix Administrator) 通过 163email | 01:00:00 | 默认     | 编辑移除 |
| 2     | **发送消息给用户:** Admin (Zabbix Administrator) 通过 163email | 01:00:00 | 默认     | 编辑移除 |
| 3     | **发送消息给用户:** Admin (Zabbix Administrator) 通过 163email | 02:00:00 | 默认     | 编辑移除 |

第1行：**步骤1**(创建的时候1-1)，可以看到其最先执行，而且是马上执行。

第2行：**步骤1-2**(创建的时候1-2)，可以看到其和第1行一样，也是马上执行，两者是并行执行。因为**值1相同都是1**。

第3行：**步骤1-3**(创建的时候1-3)，可以看到其和第1行和第2行一样，也是马上执行，三者是并行执行。因为**值1相同都是1**。

第4行：**步骤2**(创建的时候2-2)，因为**值1是2**所以其在1个小时后执行，因为默认步骤持续时间为1个小时，要等步骤1执行完。

第5行：**步骤2**(创建的时候2-2)，和上面的第4行**值1是2**相同，和第4行并行执行。

第6行：**步骤3**(创建的时候3-3)，因为**值1是3**所以其在2个小时后执行，因为默认步骤持续时间为1个小时，其要等到步骤1和步骤2两个都执行完后才会执行。

总结：创建步骤的时候两个值，值1确定步骤执行先后，值2设置了在同一个步骤被执行的时候，是否并行执行。

**步骤持续时间：**你可以指定要本步骤需要执行的时间，如果是0则使用默认**操作步骤持续时间**。

主题：

```
{HOST.IP}主机，出现问题[{TRIGGER.NAME}]级别[{TRIGGER.SEVERITY}]
```

消息：

```
问题主机：{HOST.NAME}
主机IP ：{HOST.IP}
发生时间：{EVENT.DATE} {EVENT.TIME}
问题等级：{TRIGGER.SEVERITY}
问题信息：{TRIGGER.NAME}
问题详情：{ITEM.NAME}:{ITEM.VALUE}
问题状态: {TRIGGER.STATUS}:{ITEM.VALUE1}
事件ID： {EVENT.ID}
```



**恢复操作**

用于，一旦警告恢复了，需要执行的动作，例如：在主机宕机的时候发送了警告邮件(xxx主机无效)。当主机重启恢复后，zabbix可以正常监控到xxx主机，其会触发恢复动作，再发送邮件(xxx主机恢复)。

恢复操作消息主题：

```
{HOST.IP}主机，问题[{TRIGGER.NAME}]已恢复.
```

恢复操作消息内容：

```
恢复主机：{HOST.NAME}
主机IP ：{HOST.IP}
问题状态：{TRIGGER.STATUS}
恢复时间：{EVENT.DATE} {EVENT.TIME}
问题等级：{TRIGGER.SEVERITY}
问题信息：{TRIGGER.NAME}:{ITEM.VALUE}
问题 ID：{EVENT.ID} 
```



##### 常用的动作

###### 发生警告则发送邮件通知

**动作条件**

类型：触发器示警度，**操作者**(大于等于),**严重性**(警告)的触发动作。

**动作步骤**

操作类型：发消息

步骤：1-1

步骤持续时间：60s

Send to users：Admin

仅发送到：163mail

Custom messages：选中

消息的主体和内容，见上面的介绍。

### 管理

#### 报警媒介类型

##### 创建mail

管理->报警媒介类型-->创建媒介类型

###### 163mail

注意：先登录163邮箱开启smtp，并获取授权码

名称：163mail

类型：电子邮件(默认)

SMTP服务器：smtp.163.com

SMTP端口：465

SMTP HELO：163.com

SMTP电邮：邮件账号，例如：dongyuitheige@163.com

安全链接：选择SSL/TLS

SSL验证对端：选中

SSL验证主机：选中

认证：选择用户名和密码

用户名称：邮件账号，例如：dongyuitheige@163.com

密码：授权码，注意：不是邮箱密码，这授权码你可以通过开通邮箱的smtp功能来获取，例如：163邮箱开通smtp其会分配一个授权码。

点击->添加按钮

###### qq mail

注意：先登录qq邮箱开启smtp，并获取授权码

其它同上面的163mail

###### 测试

选择这个新建的163mail，选择测试，发送邮件，查看是否能正常收到，如果不能目标端也开启stmp，再测试一下。

###### 关联发送人

User settings -> 报警媒介 ->添加

选择上面新创建的mail媒介，例如：163mail

其它的不要动，输入收件人，点击添加。注意：还是点击"更新"，否则无效。

以上只完成了发送邮件的配置，但如果要完成，发现警告就发送邮件的操作，还需要添加动作，具体见"动作章节"。

**最后能把这个用户添加到邮件的白名单**

##### 企业微信

好文章

https://blog.csdn.net/chj_1224365967/article/details/108234354

###### 企业微信端操作

注册企业端微信、创建应用可以参见上面的URL。

其注册后，会生成如下有用的东西：

AgentId 、Secret 、企业ID，这三样必须保存好。

###### zabbix_server端安装weixin.py脚本

**安装python**

cd /etc/yum.repos.d/

curl http://mirrors.aliyun.com/repo/epel-7.repo > epel-7.repo

yum install python-pip -y

pip install requests

**编辑weixin.py脚本**

vi /usr/lib/zabbix/alertscripts/weixin.py

如下脚本，需要使用到注册企业微信获取到AgentId 、Secret 、企业ID。

```python
#!/usr/bin/env python
#coding:utf-8

import requests
import sys
import os
import json
import logging

logging.basicConfig(level = logging.DEBUG, format = '%(asctime)s, %(filename)s, %(levelname)s, %(message)s',
datefmt = '%a, %d %b %Y %H:%M:%S',
filename = os.path.join('/tmp','weixin.log'), # 指定临时weixin的日志路径
filemode = 'a')

corpid='xxx'  # 需要修改企业ID
appsecret="xxx"  # 企业的secret秘钥
agentid="xxx" # 修改agentid
token_url='https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=' + corpid + '&corpsecret=' + appsecret
req=requests.get(token_url)
accesstoken=req.json()['access_token']


msgsend_url='https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=' + accesstoken
touser=sys.argv[1]
subject=sys.argv[2]
message=sys.argv[2] + "\n\n" +sys.argv[3]


params={
"touser": touser,
"msgtype": "text",
"agentid": agentid,
"text": {
"content": message
},
"safe":0
}

req=requests.post(msgsend_url, data=json.dumps(params))
logging.info('sendto:' + touser + ';;subject:' + subject + ';;message:' + message)
```

设置执行权限

```bash
chmod +x /usr/lib/zabbix/alertscripts/weixin.py
chown zabbix. /usr/lib/zabbix/alertscripts/weixin.py 
```

发送信息测试

注意：ZhangDongBo为你企业微信用户名的汉子的字母全拼，例如：用户名：张三，则使用ZhangSan。

```
python /usr/lib/zabbix/alertscripts/weixin.py ZhangDongBo "测试" "人生苦短，我学Python！"
```

如果内网环境，则需要进行代理，脚本如下

vi /usr/lib/zabbix/alertscripts/weixin_proxy.sh

```bash
#!/bin/sh
export http_proxy="http://proxy_user:proxy_pass@10.60.32.xx:port"
export https_proxy="http://proxy_user:proxy_pass@10.60.32.xx:port"
/usr/bin/python /usr/lib/zabbix/alertscripts/weixin.py "$@"
```

chmod +x /usr/lib/zabbix/alertscripts/weixin_proxy.py
chown zabbix. /usr/lib/zabbix/alertscripts/weixin_proxy.py 

测试：

/usr/lib/zabbix/alertscripts/weixin_proxy.sh ZhangDongBo 黑哥 牛逼TEXT

###### zabbix微信警告

创建报警媒介

名称：微信

类型：脚本

脚本名称：weixin.py  或者 weixin_proxy.sh

脚本参数：{ALERT.SENDTO}

​                   {ALERT.SUBJECT}

​                   {ALERT.MESSAGE}

###### 关联发送人

User settings -> 报警媒介 ->添加

类型：微信

收件人：企业微信的用户名，这里必须是拼音字母全称，首字母大写，其它字母小写，例如：张三，那么收件人填写：ZhangSan，如果要发给所有的监控人，则需要两步：

1.企业微信端网页登录，应用->zabbix警告机器人->可见范围->加入相关人员。

2.收件人为@all

点击更新

测试：关闭一个主机的agent，看是否发生微信警告

## 监控中间件

### 监控nginx

/usr/share/doc/zabbix-agent-5.0.3/userparameter_mysql.conf



## FAQ

### zabbix server无10050监听端口

cat /var/log/zabbix/zabbix_server.log

cannot start preprocessing service: Cannot bind socket to "/var/run/zabbix/zabbix_server_preprocessing.sock": [13] Permission denied

解决方法关闭SELinux











