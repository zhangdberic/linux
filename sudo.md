# 三权分立

## 1.创建admin代替root

### 1.1 创建admin

创建一个管理员用户，例如：admin

useradd admin

passwd admin

### 1.2 允许admin执行sudo

root用户执行，加入允许sudo的用户

```shell
visudo
```

查看

```
## Allow root to run any commands anywhere
```

在其下加入允许sudo的用户，例如：

```
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
admin   本机ip=(ALL)       ALL
```

验证

su - admin

```shell
cat /etc/sysconfig/iptables
```

cat: /etc/sysconfig/iptables: 权限不够

```linux
sudo cat /etc/sysconfig/iptables
```

[sudo] password for admin: 输入用户admin的密码。

可以查看iptables的内容了

## 2.创建auditor(审计员)

### 2.1 创建auditor

sudo useradd auditor

sudo passwd auditor

### 2.2 允许auditor执行sudo

root用户执行，加入允许sudo的用户

```
auditor    本机ip地址=(ALL)       /bin/cat,/bin/ls,/usr/bin/tail,/bin/more,/sbin/auditctl,/sbin/ausearch,/sbin/aureport
```

只允许执行定义的命令

验证

su - auditor

ausearch -i

sudo ausearch -i

## 2 不允许root登录

意义不大，不要做了

admin用户下执行：

sudo vi /etc/passwd

原来：

```
root:x:0:0:root:/root:/bin/bash
```

修改为：

```
root:x:0:0:root:/root:/sbin/nologin
```

修改后马上生效，不管理是su - root，还是ssh远程登录，都提示：

This account is currently not available.



注意：其只能保证远程终端无法登陆，如果已经使用其他用户到linux系统，并使用如下命令就可以正常切换到root下，su - root -s /bin/bash，理解为：默认su - root 使用/etc/passwd的配置，因为你修改为了nologin，所以无法正常登陆，但如果你指定了/bin/bash方式登陆，则就可以正常使用/bin/bash来登陆了。



## 3.审计

默认审计服务已经启动了。

### 3.1 创建审计规则

admin用户下执行：

sudo vi /etc/audit/audit.rules

加入

```
-w /etc/inittab -p wa
-w /etc/init.d/
-w /etc/init.d/auditd -p wa
-w /etc/cron.d/ -p wa
-w /etc/cron.daily/ -p wa
-w /etc/cron.hourly/ -p wa
-w /etc/cron.monthly/ -p wa
-w /etc/cron.weekly/ -p wa
-w /etc/crontab -p wa
-w /etc/group -p wa
-w /etc/passwd -p wa
-w /etc/sudoers -p wa
-w /etc/hosts -p wa
-w /etc/sysctl.conf -p wa
-w /etc/aliases -p wa
-w /etc/bashrc -p wa
-w /etc/profile -p wa
-w /etc/profile.d/
-w /var/log/lastlog
-w /var/log/yum.log
-w /etc/issue -p wa
-w /etc/issue.net -p wa
-w /usr/bin/ -p wa
-w /usr/sbin/ -p wa
-w /bin -p wa
-w /etc/ssh/sshd_config
```

参数说明：

-w 要添加观察器，可使用-w选项，后面跟着一个要观察的文件或目录。

-p 要限制文件或目录观察器为某些动作，可使用-p选项，后面跟着下面的选项中的一个或多个：r表示观察读动作，w表示观察写动作，x表示观察执行动作，a表示在末尾添加动作。

重启审计

sudo service auditd restart

查看审计规则

sudo auditctl -l



### 3.2 查看审计记录

auditor用户下执行：

例如：查看某天的审计日志

sudo ausearch -i |grep '2020年01月14日'

或者

sudo ausearch -i |grep '01/14/2020'

例如：查看某天、某个小时的审计日志

sudo ausearch -i |grep '2020年01月14日 09'

例如：查看某天的ssh登录

sudo ausearch -i |grep '2020年01月14日'|grep 'lport=22'

例如：查看某天远程ip的访问

sudo ausearch -i |grep '2020年01月14日'|grep 'addr=192.168.5.31'

例如：查看/bin/rm审计

 sudo ausearch -i -k removefile

注意：这里的removefile对于上面sudo auditctl -w /bin/rm -p x -k removefile。



4.如果为更则不允许admin和auditor远程登录

sudo vi /etc/ssh/sshd_config 

DenyUsers 加入 admin auditor

sudo service sshd restart