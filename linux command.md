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

