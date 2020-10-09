# rsyslog日志集中收集

## 服务器端

**编辑配置文件**

使用tcp协议收集日志

vi /etc/rsyslog.conf

```
# Provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 514
```

**重启rsyslog服务**

```
service rsyslog restart
```

**开启防火墙**

允许那些ip地址发送日志信息到rsyslog服务器

vi /etc/sysconfig/iptables

```
# rsyslog client
-A INPUT -s 10.60.32.171 -m state --state NEW -m tcp -p tcp --dport 514 -j ACCEPT
-A INPUT -s 10.60.32.165 -m state --state NEW -m tcp -p tcp --dport 514 -j ACCEPT
```

service iptables restart

## 客户端

**服务端防火墙加入客户端ip地址**

见"服务器端"，开启防火墙



**配置输入命令记录到rsyslog**

因为rsyslog默认会收集/var/log/messages文件内容，这里把在shell控制台上输入的命令重定向到/var/log/messages，这样可以实现命令的rsyslog集中收集了。

vi /etc/profile

vi /etc/bashrc 

```
export HISTFILESIZE=1000000000
export HISTSIZE=1000000
export PROMPT_COMMAND="history -a"
export HISTTIMEFORMAT="%Y-%m-%d_%H:%M:%S "

export PROMPT_COMMAND='{ msg=$(history 1 | { read x y; echo $y; }); logger "[euid=$(whoami)]":$(who am i):[`pwd`]"$msg";}'
```

source /etc/bashrc

**编辑配置文件**

vi /etc/rsyslog.conf

加入:

```
*.* @@10.60.32.83
```

重启rsyslog服务

service rsyslog restart

或

systemctl rsyslog restart

**测试验证**

rsyslog服务器端

tail -f /var/log/messages

