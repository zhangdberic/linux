# jenkins文档

## 1.安装和配置

docker，见docker文档。

## 2.设置

### 2.1 系统管理

#### 2.1.1 系统配置

**SSH remote hosts**

需要 SSH plugin 插件支撑；

用于：“构建->构建->Execute shell script on remote host using ssh"，在远程服务器执行命令。

新增

hostname 要访问的服务器ip地址(ssh ip);

port ssh端口号；

Credentials 凭据，需要在凭据管理上增加访问服务器的ssh用户名和密码；

pty，这个不选；

serverAliveInterval 每隔多少时间(毫秒),发送一次心跳，防止ssh长时间不用自动关闭；

timeout 连接超时时间（毫秒）；

配置后，点击Check connection，测试ssh连接配置是否正确。

**Publish over SSH**

用于："构建->构建后操作->Send build artifacts over SSH"，传输jenkins构建后的jar或者war文件到远程服务器，并在远程服务器执行一个命令。

需要Publish Over SSH插件支撑

**证书访问**

Passphrase 如果你准备基于证书访问，这个为证书的密码；

Path to key 证书key路径；

key 粘贴key，和上面的Path to key区别是，Path to key证书存放在文件中，这个直接拷贝到输入框；

**密码访问**

SSH Servers 

name 起个名字，例如：POS-192.168.5.78

Hostname 主机地址

username 登录主机用户名

Remote Directory 远程目录，操作的起始目录，注意：是起始目录。例如：如果这项设置的是/software，那么如果执行命令：mkdir -p /xxx，不会在远程服务器的根据目录下创建xxx目录，而是创建了/software/xxx。如果你要创建在根目录下，这项应该设置为 /  ；

点击高级按钮

选择使用“Use password authentication, or use a different key"，使用密码认证；

Passphrase / Password 输入上面username对应的密码；

Port 指定ssh端口；

设置后点击“Test Configuration"，测试配置和ssh连接是否正确。



#### 2.1.2 全局配置

**Maven配置**

因为在jenkins启动(docker模式)的时候已经指定了挂载maven映射，因此这里两个settings的路径都设置为：

```
文件系统中的settings文件

文件路径：/maven/conf/settings.xml
```

这里要注意：settings.xml的<localRepository>/maven_repo</localRepository>设置为/maven_repo，因为在jenkines的docker启动的时候设置了maven本地仓库映射，-v /data/maven_repo:/maven_repo；

**JDK**

点击JDK安装

因为在jenkins启动(docker模式)的时候已经指定了挂载jdk映射，因此这里的JAVA_HOME设置为：

```
JDK 别名    JDK1.8
JAVA_HOME  /jdk
```

**Maven**

点击Maven安装

因为在jenkins启动(docker模式)的时候已经指定了挂载maven映射，因此这里的MAVEN_HOME设置为：

```
Name maven3
MAVEN_HOME /maven
```



#### 2.1.6 插件管理

http://192.168.5.78:10000/

系统管理->插件管理->可选插件

```
Oracle Java SE Development Kit Installer Plugin

JUnit Plugin

Timestamper

Build Timeout

Subversion Plug-in

GitHub plugin

GitLab Plugin

Maven Integration plugin

Publish Over FTP

Publish Over SSH

SSH plugin

Role-based Authorization Strategy

Pipeline

docker-build-step

Credentials Plugin

Email Extension Plugin
```



## 3.构建

#### 3.1.构建maven项目

构建maven项目依赖于上面的安装和配置文档(例如，依赖java_home和maven_home的设置)。

##### 3.1.1 General

描述，输入项目的描述信息；

##### 3.1.2 源码管理

###### 3.1.2.3 Subversion

**Repository URL：**

注意：可以支持svn原生的svn协议；

```
svn://svn.xxx.cn:9999/zframework/sgw
```

**Credentials：**

凭证，svn的用户名和密码，创建后的凭证可以在多个项目上复用；

**Local module directory：**

理解为项目目录，这里不用特殊设置，默认的即可，其会在/var/jenkins_home/workspace目录下自动创建一个"项目名称-build"的目录，例如：sgw项目，其会自动创建/var/jenkins_home/workspace/sgw-build目录；

**Repository depth：**

仓库的深度；

使用默认的infinity；使用递归穷尽；

**Ignore externals** ：

默认勾选；svn checkout, svn update commands to disable externals definition processing.

**cancel process on externals fail;**

 默认勾选；svn:externals failed. Will work when "Ignore externals" box is not checked. Default choice is to cancelled the process when checkout/update svn:externals failed.

**Additional Credentials ；**

不用管这个，这个和上面的externals有关。

**Checkout-out strategy：**

checkout项目的策略，这里可以有多种：

Use 'svn update' as much as possible（默认）. 尽可能使用“ svn更新”；这个时间短，只获取更新的代码；**开发环境使用这个；**

Always check out a fresh copy. 始终签出新副本；这个比较耗时，但比较准确，从新获取所有的代码；**生产环境应该使用这个；**

Do not touch working copy, it is updated by other script. 不创建工作副本，使用更新脚本；

Emulate clean checkout by first deleting unversioned/ignored files, then 'svn update' 首先删除未经版本控制/忽略的文件，然后进行“ svn更新”，以模拟干净签出；

Use 'svn update' as much as possible, with 'svn revert' before update 尽可能使用“ svn更新”，并在更新前先使用“ svn恢复”；

```
1、Use‘svn update’ as much as possible
（1）第一次发布把工作空间清空，然后checkout一份到工作空间
（2）以后更新的时候只要svn里面的文件没有更新就用工作空间的，更新了再会把工作空间的更新
（3）有个局限就是工作空间的文件内容修改了跟svn不一样了，也不会更新了，不过一般不会修改工作空间的文件内容
（4）svn删除了文件，工作目录也会删除
2、Alwayscheck out a fresh copy
（1）第一次发布把工作空间清空，然后checkout一份到工作空间
（2）以后的每一次更新都清空工作空间然后checkout一份下来。也就是说svn里有一个文件更新，也会把整个目录checkout一次到工作空间
3、Emulateclean checkout by first deleting unversioned/ignored files，then ‘svn update’
（1）第一次发布把工作空间清空，然后checkout一份到工作空间
（2）以后更新的时候会判断工作目录下的文件是否在svn里存在，不存在则删除，存在且SVN有新版本则更新，没有新版本则不更新
（3）如果工作空间目录被修改了，则不管有没有新版本都会checkout下svn中的最新版本
（4）svn删除了文件，工作目录也会删除
4、Use‘svn update’ as much as possible，with ‘svn revert’ before update
（1）第一次发布把工作空间清空，然后checkout一份到工作空间
（2）以后更新的时候不会判断工作目录下的文件是否在svn里存在
（3）如果工作空间目录被修改了，则不管有没有新版本都会checkout下svn中的最新版本
（4）svn删除了文件，工作目录也会删除
```

**Quiet check-out 默认勾选；**

源码库浏览器，默认（自动）；

##### 3.1.5 构建环境

Delete workspace before build starts 构建前先删除这个项目，**生产环境建议选择**；

##### 3.1.6 构建

###### 3.1.6.5 maven构建

选择：调用顶层Maven目标

Maven版本 ：选择maven，这个是在“系统管理->全局工具配置-Maven”中安装时配置的别名。

目录：maven构建过程命令，例如：clean install -Dmaven.test.skip=true

###### 3.1.7 构建后操作

选择：Send Build artifacts over SSH

SSH Server：选择一个已经配置的Publish Over SSH，参见”2.1.2 系统配置 -> Publish over SSH"

Source files：传输本地到远程服务器的文件，起始目录为build目录，例如：target/sgw-1.0.1.jar；

Remove prefix：传输过去后要去掉前置，例如：target/，传输过去后，只保留sgw-1.0.1.jar；

Remote directory：传输到远程的目录，注意：这个依赖于Publish over SSH配置的Remote Directory起始目录。

Exec command：传输文件到远程服务器后，在远程服务器执行的目录，例如：deploy.sh



### 5.构建和发布

一般项目都会有四个环境：

1.开发环境，程序员本机；

2.测试环境，测试人员使用环境；

3.预发布环境，客户测试或者是生产环境模拟；

4.生产环境；

我们开发的程序或者项目如何部署到这4个环境上?

目前我的设计是这样的：

开发环境：可以忽略不提；

测试环境：独立部署一套jenkins，因为通常测试环境和生产环境会进行隔离，而且更分开部署jenkins更安全；

预发布环境和生产环境：独立部署一套jenkins，直接从互联网的云服务器上获取源码，在本地编译，然后发布，速度更快，而且和测试环境的jenkins区分开，更可靠。生产环境的发布，必须



# jenkins 时区设置

https://www.cnblogs.com/jwentest/p/7270692.html