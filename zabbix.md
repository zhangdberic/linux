# zabbix5.0安装和配置

## 服务器端

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

set setenforce　0

### 安装

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

安装zabbix服务器端和mysql连接器

yum -y install zabbix-server-mysql zabbix-agent

安装centos发布集合

yum -y install centos-release-scl

安装zabbix前端

vi /etc/yum.repos.d/zabbix.repo

修改[zabbix-frontend]的enabled=1

yum -y install zabbix-web-mysql-scl zabbix-nginx-conf-scl



安装mariadb

yum install  mariadb-server -y

启动mariadb

systemctl start mariadb && systemctl enable mariadb && systemctl status mariadb

设置mariadb的root账号

mysqladmin -u root password '12345678'

创建zabbix需要的库和表结构

**注意'password'为zabbix用户的密码，这个生产环境必须修改。**

```
# mysql -uroot -p
password
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
```

导入zabbix数据

zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix

配置zabbix服务器连接数据库密码

这个密码是上面mysql中zabbix用户的密码。

vi /etc/zabbix/zabbix_server.conf

```
DBPassword=password
```

配置PHP的前端nginx

vi /etc/opt/rh/rh-nginx116/nginx/conf.d/zabbix.conf

```nginx
        listen          80;
        server_name     location;
```

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

启动zabbix

```
 systemctl restart zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm
systemctl enable zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm

```

开放防火墙的80和10050端口



编辑nginx.conf文件，去掉server部分

 vi /etc/opt/rh/rh-nginx116/nginx/nginx.conf

编辑nginx.conf文件，去掉server部分，因为include的/etc/opt/rh/rh-nginx116/nginx/conf.d/zabbix.conf文件已经包含了zabbix要使用的server部分配置。

去掉include /etc/opt/rh/rh-nginx116/nginx/conf.d/*.conf;配置后的所有配置，例如：

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

配置后，重启

systemctl restart zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm



初始化配置

直接访问zabbix服务器，如果页面还是nginx的首页则可以上面的nginx.confi配置不对，或者浏览器有缓存(ctrl+f5)需要刷出。

http://192.168.5.15/

第三个页面（Configure DB connection)，最后的Password为mysql的zabbix用户密码，例如默认为"password"，如果数据库不和zabbix在同机器，则要修改上面更多链接到mysql的信息。

第四个页面（Zabbix server details），最后的Name为zabbix命名，例如：zbxserver

最后一页，配置成功为：Congratulations! You have successfully installed Zabbix frontend.



默认的登录用户密码和密码

Admin/zabbix，区别大小写。

### 配置

#### 修改Admin密码

User settings菜单->点击Change password

#### 修改到中文

User settings菜单->Language Chinese(zh_CN)

修改字体,否则乱码

cd /usr/share/zabbix/assets/fonts

mv graphfont.ttf graphfont.ttf.bak

windows：C:\Windows\Fonts，上传微软雅黑字体，到这个目录，然后改名

mv MSYH.TTF graphfont.ttf

改名后无需重启zabbix服务器，直接刷新页面就可以看到新字体了。

#### 验证安装是否成功

进入zabbix主页面->监测->主机->看Zabbix server的可用性项目ZBX是否为绿色。



### 管理

#### 安装zabbix-get命令

yum install zabbix-get.x86_64

使用

zabbix_get -s 192.168.5.54 -p 10050 -k "system.cpu.load[all,avg1]"

zabbix_get -s 192.168.5.54 -k system.cpu.num

客户测试客户端，是否畅通，相同的健康项是否正常等。



#### 创建主机

配置->主机

##### 主机tab页

主机名称：必须了客户端配置文件/etc/zabbix/zabbix_agentd.conf内的Hostname相同。

客户端：

​	IP地址：为客户端的ip地址，端口10050默认。

##### 模板tab页

点击Link new templates[选择]，选择Template，再选择Template OS Linux by Zabbix agent。

点击，更新按钮。



点击，添加。

##### 测试

在zabbix_server上执行，如下命令：-s后为zabbix客户端的ip地址。

zabbix_get -s 192.168.5.54 -p 10050 -k system.cpu.num

测试返回的客户端的cpu个数



##### 图形监控个数

随着时间越来越长，监控获取到的数据越来越多，图片支持的也就越多。原因是想要图形显示则需要比较值，比较值则需要前后结果对比，则需要收集数据，而监控项收集数据是有时间间隔的，因此需要一段时间。

#### 

#### 模板

##### 监控项

配置->模板->选择一个模板（例如：Template OS Linux by Zabbix agent）->监控项

###### 修改

点击某个一个监控项（例如：Checksum of /etc/passwd）

例如，我们修改刷新间隔(数据项采集时间间隔)，这个默认监控项默认是15m，我们修改为1m。

**测试验证**

到监控的任何一台机器创建一个用户useradd abc，创建用户间接的修改了/etc/password，回到zabbix的主页监控仪表板查看情况。这个需要2分钟才可以体现出，因为你上面设置了Checksum of /etc/passwd的监控刷新1分钟，主页页面仪表盘问题项刷新也是1分钟，因此需要两分钟。

#### 触发器

#### 问题触发

如果你触发一个问题，但没有确认，那么如果在这段时间，重复这个问题，zabbix将不处理(丢弃)。你只有确认了这个问题，那么才可以再触发相同的问题，当然是相对于同一台被监控的主机。

## 



### 动作

http://www.zsythink.net/archives/768

动作：理解为当触发器被触发的时候，执行某些操作。

下面举两个例子：

例子1：当被监控主机无法监控(宕机)，触发器({HOST.NAME} has been restarted (uptime < 10m))被触发，动作是发送邮件。触发条件如下，

```
{192.168.5.30:system.uptime.last()}<10m
```

例子2：当有人改变了/etc/passwd文件(例如：增加了一个用户)，触发器(/etc/passwd has been changed)被触发，动作是发送邮件。触发条件如下，

```
{192.168.5.30:vfs.file.cksum[/etc/passwd].diff()}>0
```

#### 动作tab

动作tab重点就是条件，也就是当满足条件的时候会执行动作，当然条件可以是一个也可以是多个组合，例如：一个条件，当某个触发器被触发的时候执行动作。多个条件，当某个主机的某个触发器被触发是执行动作。



#### 操作tab

当前动作条件满足的时候，执行的操作，操作可以是一个步骤也可以是多个步骤，多个步骤可以并行执行也可以串行执行，其是**由步骤值**决定的。

默认操作步骤持续时间：每执行一个步骤的时间，默认值为1h(1个小时)，如果你的动作就一个步骤没什么意义，如果是多个步骤，那么第1个步骤执行后1个小时，才会执行第2个步骤。

暂停操作以制止问题：这个目前没搞懂，默认选中，不用改。

**操作**

操作类型：发消息，下面都是发消息的介绍

步骤：这个不好理解，要细品，步骤值有两个，值1和值2配合使用，可以确定步骤执行的先后和时间，我们通过下面的例子来分析，下面添加了6个步骤：

| 步骤  | 细节                                                         | 开始于   | 持续时间 | 动作     |
| :---- | :----------------------------------------------------------- | :------- | :------- | :------- |
| 1     | **发送消息给用户:** Admin (Zabbix Administrator) 通过 所有介质 | 立即地   | 默认     | 编辑移除 |
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

**恢复操作**

用于，一旦警告恢复了，需要执行的动作，例如：在主机宕机的时候发送了警告邮件(xxx主机无效)。当主机重启恢复后，zabbix可以正常监控到xxx主机，其会触发恢复动作，再发送邮件(xxx主机恢复)。



#### 常用的动作



##### 条件

添加条件

触发条件，可以有很多，下面举一些常用的例子：

类型：触发器示警度，常用的是**操作者**(大于等于),**严重性**(警告)的触发动作。比较比较常用，多用于发生警告则发送邮件。



故障模板

```
故障{TRIGGER.STATUS},服务器:{HOSTNAME1}发生: {TRIGGER.NAME}故障!

告警主机:{HOSTNAME1}
告警时间:{EVENT.DATE} {EVENT.TIME}
告警等级:{TRIGGER.SEVERITY}
告警信息: {TRIGGER.NAME}
告警项目:{TRIGGER.KEY1}
问题详情:{ITEM.NAME}:{ITEM.VALUE}
当前状态:{TRIGGER.STATUS}:{ITEM.VALUE1}
事件ID:{EVENT.ID}
```

恢复模板

```
恢复{TRIGGER.STATUS}, 服务器:{HOSTNAME1}: {TRIGGER.NAME}已恢复!

告警主机:{HOSTNAME1}
告警时间:{EVENT.DATE} {EVENT.TIME}
告警等级:{TRIGGER.SEVERITY}
告警信息: {TRIGGER.NAME}
告警项目:{TRIGGER.KEY1}
问题详情:{ITEM.NAME}:{ITEM.VALUE}
当前状态:{TRIGGER.STATUS}:{ITEM.VALUE1}
事件ID:{EVENT.ID}
```

### 发送邮件

#### 创建mail

管理->报警媒介类型-->创建媒介类型

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

#### 测试

选择这个新建的163mail，选择测试，发送邮件，查看是否能正常收到，如果不能目标端也开启stmp，再测试一下。

#### 关联发送人

User settings -> 报警媒介 ->添加

选择上面新创建的mail媒介，例如：163mail

其它的不要动，输入收件人，点击添加。注意：还是点击"更新"，否则无效。

以上只完成了发送邮件的配置，但如果要完成，发现警告就发送邮件的操作，还需要添加动作，具体见"动作章节"。

**最后能把这个用户添加到邮件的白名单**





## 问题

### Zabbix agent is not available

ZBX图标变红

一般情况下是监控的客户端主机无法联系了，具体：监控主机宕机、断网、zabbix_agent被关闭等。

问题解决：如果能正常连接上这个监控主机的zabbix_agent就可以指定恢复了。

### 主机 has been restarted

由[Zabbix agent is not available]错误恢复到正常情况下，则提示Zabbix agent is not available。





## 客户端

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

set setenforce　0

### centos7安装

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

验证是否启动

netstat -ntlp|grep 10050

```
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      9148/zabbix_agentd
```

### centos6安装

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

Server=192.168.65.1 # zabbix server

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



验证是否启动

netstat -ntlp|grep 10050

## FAQ

### zabbix server无10050监听端口

cat /var/log/zabbix/zabbix_server.log

cannot start preprocessing service: Cannot bind socket to "/var/run/zabbix/zabbix_server_preprocessing.sock": [13] Permission denied

解决方法关闭SELinux











