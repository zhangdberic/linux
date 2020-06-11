# gitlabs

## gitlab安装

**安装依赖库和打开http、ssh端口**

sudo yum install -y curl policycoreutils-python openssh-server openssh-clients cronie

**安装邮件服务器，并设置开机启动**

yum install -y postfix

systemctl start postfix 

systemctl enable postfix.service

**添加GitLab仓库到yum源,并用yum方式安装到服务器上**

使用国内的镜像

vim /etc/yum.repos.d/gitlab-ce.repo

```
[gitlab-ce]
name=gitlab-ce
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
```

yum makecache

yum install -y gitlab-ce

**修改gitlab配置**

vi /etc/gitlab/gitlab.rb

```
external_url 'http://39.105.202.78:3333'   # 修改访问的ip地址，本机对外提供的ip,如果有端口一定要加上。
nginx['listen_port'] = 3333   # 可以修改访问的端口号，注意去掉前面的#号
```

**重新配置**

gitlab-ctl reconfigure

**重新启动**

gitlab-ctl restart

gitlab-ctl tail

**注意：第一次初始化非常慢，30分钟左右，如果还没有初始化完，你发送请求则返回502错误。**

**注意：gitlab的puma需要8080端口，因此8080端要给gitlab留着，不能占用，否则无法启动gitlab。**

gitlab涉及的中间件非常多。

**浏览器，访问http://39.105.202.78:3333**

第一次访问，需要修改root用户的密码。

修改后自动登录了，你应该先sign-out，然后在重新登录一次测试一下root登录是否正常。

**禁用sign-up**

使用root用户登录，然后在主界面的左上角，有一个**扳手**的图标（Admin Area），点击"Settings"，Sign-up restrictions项目，去掉Sign-up enabled前面的勾选项，然后save保持。

**创建用户**

有一个**扳手**的图标（Admin Area），点击"Overview->Users"。

## gitlab命令

**监听日志**

gitlab-ctl tail

**查看状态**

gitlab-ctl status

```
run: alertmanager: (pid 31860) 263s; run: log: (pid 16172) 4116s
run: gitaly: (pid 31884) 262s; run: log: (pid 15148) 4259s
run: gitlab-exporter: (pid 31893) 262s; run: log: (pid 16000) 4133s
run: gitlab-workhorse: (pid 31912) 261s; run: log: (pid 15742) 4155s
run: grafana: (pid 31930) 261s; run: log: (pid 16461) 4062s
run: logrotate: (pid 31962) 261s; run: log: (pid 15868) 4144s
run: nginx: (pid 31968) 260s; run: log: (pid 15776) 4151s
run: node-exporter: (pid 31983) 260s; run: log: (pid 15920) 4139s
run: postgres-exporter: (pid 31989) 259s; run: log: (pid 16207) 4110s
run: postgresql: (pid 32000) 259s; run: log: (pid 15343) 4253s
run: prometheus: (pid 32091) 258s; run: log: (pid 16063) 4120s
run: puma: (pid 32112) 258s; run: log: (pid 15680) 4165s
run: redis: (pid 32118) 257s; run: log: (pid 15057) 4265s
run: redis-exporter: (pid 32123) 257s; run: log: (pid 16024) 4127s
run: sidekiq: (pid 32134) 253s; run: log: (pid 15705) 4159s
```

**重启gitlab**

gitlab-ctl restart

**启动gitlab**

gitlab-ctl start

启动会非常的慢，等吧，10分钟左右。

**停止gitlab**

gitlab-ctl stop

**重新配置(慎用)**

gitlab -reconfigure





# docker方式安装gitlabs

1、搜索可用的Gitlab images
   docker search gitlab
2、选区一个image 并下载Gitlab
   sudo docker pull gitlab/gitlab-ce:latest
3、查看镜像文件
   docker images
4、创建Gitlab挂载目录
   在创建/gitlab目录，并分别创建子目录config,logs,data
5、创建docker中的网络
   docker network create gitlab_net
6、使用image启动Gitlab容器, 启动前确认7001 没被使用 并且添加到iptables或者firewall里面(允许外部访问)
docker run --name='gitlabs' -d \
    --publish 1443:443 --publish 7001:80 --publish 7002:22 \
    --restart always \
    --volume /gitlab/config:/etc/gitlab \
    --volume /gitlab/logs:/var/log/gitlab \
    --volume /gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce

7、登录Gitlab 
   http://192.168.5.78:7001成功的话需要修改root账号的密码，随意设置即可。密码修改成功后，系统进入登录/注册页面

8.配置gitlab
  登入container： docker exec -it gitlab /bin/bash
  根据官方文档进行设置。编辑/etc/gitlab/gitlab.rb文件，这是Gitlab的全局配置文件
  官方文档：https://docs.gitlab.com/omnibus/settings/configuration.html

9.备份

只需备份/gitlab目录就可以了。



# Gitlabs数据备份与恢复

## 创建备份

```shell
gitlab-rake gitlab:backup:create
```

执行完备份命令后会在`/var/opt/gitlab/backups`目录下生成备份后的文件，如`1500809139_2017_07_23_gitlab_backup.tar`。1500809139是一个时间戳，从1970年1月1日0时到当前时间的秒数。这个压缩包包含Gitlab所有数据（例如：管理员、普通账户以及仓库等等）。

**备份gitlab.rb**

gitlab.rb文件存放在/etc/gitlab/gitlab.rb下，你在备份数据的后，也应该同步备份这个文件。

**备份gitlab-secrets.json**

gitlab-secrets.json存放在/etc/gitlab/gitlab-secrets.json下，你在备份数据的后，也应该同步备份这个文件。

## 备份恢复

本节说明如何在另一台主机上恢复数据。

将备份文件拷贝到`/var/opt/gitlab/backups`下（备份和恢复的GitLab版本尽量保持一致，后文描述了版本不匹配的处理方法）。

### 停止相关数据连接服务

```
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq 
```

### 从备份恢复

从指定时间戳的备份恢复（backups目录下有多个备份文件时）：

```
gitlab-rake gitlab:backup:restore BACKUP=1500809139
```

从默认备份恢复（backups目录下只有一个备份文件时）：

```
sudo gitlab-rake gitlab:backup:restore
```

### 修改默认备份目录【可选】

你也可以通过修改`/etc/gitlab/gitlab.rb`来修改默认存放备份文件的目录：

```
gitlab_rails['backup_path'] = '/home/backup'
```

`/home/backup`修改为你想存放备份的目录即可, 修改完成之后使用`gitlab-ctl reconfigure`命令重载配置文件即可。

