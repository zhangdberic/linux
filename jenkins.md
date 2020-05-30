# jenkins文档

## 1.安装和配置

useradd jenkins
su - jenkins

参照"tomcat文档"，安装和配置tomcat。

下载最新稳定版的jenkins.war

wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war

把jenkins.war放在tomcat的webapps目录下

注意：server.xml配置的unpackWARs="true"，否则无法自动解压jenkins.war文件。

启动tomcat

### 隐藏的.jenkins文件

注意这个带点的jenkins目录[.jenkins]，这里存放jenkins的数据文件。

/home/jenkins/.jenkins

### 初始化

http://192.168.1.250:8080/jenkins

按照界面上的提示，查看这个文件，得到初始化密码

不选择插件

创建用户

**安装插件**

Manage Jenkins -> Manage Plugins -> Advanced -> Update Site 

修改 URL（国内的镜像源）：

```
http://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```

点击 Check now 按钮，下载update-center.json文件。

**修改插件源索引文件地址**

**注意：每点击一次check now按钮，都会下载新的插件源索引文件到本地并覆盖default.json文件。**

**修改插件源索引文件内插件URL地址**

即使你的索引文件是国内镜像源，但其内插件的URL地址还是国外地址，那也无法正常安装插件。下面的语句，修改到国内的镜像源(还必须是http)。

```bash
cd /home/jenkins/.jenkins/updates/
cp default.json default.json.bak
sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json
sed -i 's/http:\/\/updates.jenkins-ci.org\/download/http:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json
```

注意：你每次更新完插件索引文件(点击 check now按钮)都要执行上面的操作。

### jenkins命令

重启

http://192.168.1.250:8080/jenkins/restart



## 2.设置

### 2.1 系统管理

#### 2.1.1 系统配置

##### SSH remote hosts

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

##### Publish over SSH

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

##### Docker Builder

Docker URL 设置 Docker server REST API URL，例如：tcp://192.168.1.250:2375/，设置后点击Test Connection按钮来测试，连接是否正确。

#### 2.1.2 全局工具配置(Global Tool Configuration)

##### Maven配置

因为在jenkins启动(docker模式)的时候已经把宿主的maven目录挂载到了jenkins docker的/maven目录，因此这里两个settings的路径都设置为：

```
文件系统中的settings文件

文件路径：/maven/conf/settings.xml
```

注意：settings.xml的<localRepository>/maven_repo</localRepository>本地仓库设置为/maven_repo，因为在jenkins的docker启动的时候已经把宿主的maven_repo目录挂载到了jenkins docker的/maven_repo目录；

如果你**代理访问**，你还是需要修改/maven/conf/settings.xml，设置proxy：

```xml
  <proxies>
    <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>user</username>
      <password>password</password>
      <host>proxy-ip</host>
      <port>proxy-port</port>
      <nonProxyHosts>localhost|127.0.0.1|10.60.*.*|192.168.*.*|dockerdongyuit.cn</nonProxyHosts>
    </proxy>
  </proxies>
```



##### JDK

点击JDK安装

因为在jenkins启动(docker模式)的时候已经把宿主的jdk目录挂载到了jenkins docker的/jdk目录，因此这里的JAVA_HOME设置为：

```
JDK 别名    JDK1.8
JAVA_HOME  /jdk
```

##### Maven

点击Maven安装

因为在jenkins启动(docker模式)的时候已经把宿主的maven目录挂载到了jenkins docker的/maven目录，因此这里的MAVEN_HOME设置为：

```
Name maven3
MAVEN_HOME /maven
```

#### 2.1.6 插件管理

**代理**

如果你的jenkins处于内网环境，需要通过代理获取插件，则需要设置代理。

注意：目前测试jenkins的代理只支持http，不支持https。因此你在设置插件源的时候，必须是http的地址。

再有，如果你docker启动的jenkins，则应使用--network host网络模式启动，否则默认的网络模式，代理也无法被访问。

Manage Jenkins -> Manage Plugins - > Advanced

no Proxy Host设置为：

```
localhost|127.0.0.1|10.60.*.*|192.168.*.*|dockerdongyuit.cn
```

Avanced..按钮，测试代理设置和插件源是否可以正常访问。

Test Url设置：

```
http://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```

点击validate proxy按钮测试。成功返回success。**注意：这里插件源地址必须http不能是https。**



**修改插件源索引文件地址**

默认的插件源更新地址是国外的官网，我们需要修改为国内的镜像地址，而且还必须是http源地址，例如：

```
http://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```

设置后点击submit，如果是第一次设置后，可以点击后面的check now按钮。

**注意：每点击一次check now按钮，都会下载新的插件源索引文件到本地并覆盖default.json文件。**

**修改插件源索引文件内插件URL地址**

即使你的索引文件是国内镜像源，但其内插件的URL地址还是国外地址，那也无法正常安装插件。下面的语句，修改到国内的镜像源(还必须是http)。

```bash
cd /home/docker/jenkins/jenkins_home/updates
cp default.json default.json.bak
sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json
sed -i 's/http:\/\/updates.jenkins-ci.org\/download/http:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json
```

注意：你每次更新完插件索引文件(点击 check now按钮)都执行上面的操作。



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

Email Extension Plugin
```



## 3.构建

### 3.1.构建maven项目

构建maven项目依赖于上面的安装和配置文档(例如，依赖java_home和maven_home的设置)。

#### 3.1.1 General

描述，输入项目的描述信息；

#### 3.1.2 源码管理

##### Subversion

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

#### 3.1.3 构建触发器

##### 定时构建

日常表

```
TZ=Asia/Shanghai
H 0 * * *
```

第一个TZ指定了时区，因为定时任何依赖于时间，而时间又依赖于时区；

H 0 * * *，这个例子：每天的0点触发构建，为什么第1个，month(分钟)是H而不是0，这是jenkins特殊的字符，其会根据负载情况，决定是0点0分运行，还是0点x分允许，而且jenkins提倡使用H。

#### 3.1.5 构建环境

##### 先删除workspace内的文件

Delete workspace before build starts 构建前先删除这个项目，**生产环境建议选择**；

#### 3.1.6 构建

##### maven构建

选择：调用顶层Maven目标

Maven版本 ：选择maven，这个是在“系统管理->全局工具配置-Maven”中安装时配置的别名。

目录：maven构建过程命令，例如：

clean install -Dmaven.test.skip=true

clean package -Dmaven.test.skip=true

##### Execute shell script on remote host using ssh

参见，“2.1.1 系统配置 SSH remote hosts"的配置。

SSH site 项目可选项，来源于上面SSH remote hosts的配置。

Command，输入要在远程服务器上执行的命令。

##### execute shell

选择：execute shell

在构建后执行shell脚本(注意是jenkins的本地脚本)，例如：对构建后的jar或者war包进行操作。

##### Execute Docker command

选择：Execute Docker command

在构建后执行对应的docker命令，这个需要先安装插件"docker-build-step"。

**Create/build image命令**

创建和构建docker image镜像

Build context foler，指定了Dockerfile文件所在的目录，例如：$WORKSPACE/target

Tag of the resulting docker image，指定创建docker image的tag，例如：dyit.com:5000/dy/test-service:1.0.2

**Push image命令**

发布docker image镜像到docker仓库

Name of image to push(repository/image) ，指定了要发布到docker的docker image镜像，例如：dyit.com:5000/dy/test-service，注意不包括tag部分。

Tag：指定了发布镜像的tag，例如：1.0.2

Docker registry URL：指定了docker 仓库的发布地址，例如：http://dyit.com:2375

Registry credentials：如果docker仓库访问需要用户名和密码，则需要先创建访问的凭证。

如果不能正常发布，则参见"2.1.1 系统配置 -> Docker Builder"，配置"Docker server REST API URL"。

#### 3.1.7 构建后操作

##### Send Build artifacts over SSH

选择：Send Build artifacts over SSH

SSH Server：选择一个已经配置的Publish Over SSH，参见”2.1.2 系统配置 -> Publish over SSH"

Source files：传输本地到远程服务器的文件，起始目录为build目录，例如：target/sgw-1.0.1.jar；

Remove prefix：传输过去后要去掉前置，例如：target/，传输过去后，只保留sgw-1.0.1.jar；

Remote directory：传输到远程的目录，注意：这个依赖于Publish over SSH配置的Remote Directory起始目录。

Exec command：传输文件到远程服务器后，在远程服务器执行的目录，例如：./bin/deploy.sh



## 5.构建和发布

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



## 构建流程

### 1.jar或war打包(maven构建->ssh发布)

1.1 创建任务;

1.2 构建一个自由风的软件项目;

1.3 Genreal，输入描述，简介明了；

1.4 源码管理，参见“3.1.2 源码管理"根据代码来源配置git或subversion；

1.5 构建触发器，参见"3.1.3 构建触发器"，例如要构建一个定时build的任务，则可以配置"定时构建"；

1.6 构建环境，参见"3.1.5 构建环境"，例如构建前要删除workspace内的文件(获取一个干净的环境)；

1.7 构建，参见"3.1.6 构建"，参见"调用顶层maven目标"；

1.8 构建后操作，参见"3.1.7 构建后操作"，参见“Send Build artifacts over SSH"；

例如：发布到tgms用户的$HOME目录下，并调用$HOME/bin/deploy.sh执行发布程序；

Send Build artifacts over SSH配置如下：

```
Name:POS-192.168.5.254-TGMS
Source files:target/ROOT.war
Remove prefix:target/
Remote directory:/
Exec command:./bin/deploy.sh
```

例子的"POS-192.168.5.254-TGMS"，依赖于"系统配置 -> Publish over SSH"，配置如下：

```
Name:POS-192.168.5.254-TGMS
Remote Directory:/home/tgms  # 注意本处配置已经限定了以后shell操作的起始目录
```

服务器端deploy.sh脚本如下：

```bash
#!/bin/bash

# 判断程序包是否存在
warfile="$HOME/ROOT.war"
if [ ! -f $warfile ]; then
  echo $warfile 'does not exist.'
  exit 0
fi

# 停止
nginx_home=$HOME/nginx
tomcat_home=$HOME/tomcat
if [ -f $nginx_home/logs/nginx.pid ]; then
   $nginx_home/sbin/nginx -s stop
fi
$tomcat_home/bin/shutdown.sh
sleep 5s
PID=$(ps -ef | grep '/home/tgms/jdk1.8/jre/bin/java' | grep -v grep | awk '{ print $2 }')
if [ -z "$PID" ]
then
    echo 'Application is already stopped'
else
    echo 'kill' $PID
fi
echo 'shutdown ok.'

# 清除
rm -rf $HOME/tgms
rm -rf $HOME/tgms_static
echo clear ok.

# 备份
cp $HOME/ROOT.war $HOME/code_history/ROOT.war.`date +"%F-%T"`
echo bak ROOT.war ok.

# 发布
mkdir $HOME/tgms
cp -r $HOME/ROOT.war $HOME/tgms
cd $HOME/tgms
$HOME/jdk1.8/bin/jar xf ROOT.war
rm -rf ROOT.war
mkdir $HOME/tgms_static
cp -r $HOME/tgms/* $HOME/tgms_static
rm -rf $HOME/tgms_static/WEB-INF
echo deploy ok.

# 启动
$tomcat_home/bin/startup.sh
sleep 10s
$nginx_home/sbin/nginx
echo startup ok.

```

### 2.docker打包(maven构建 -> docker仓库发布)

#### 程序部分

**dockerfile**

创建目录"/src/main/docker"，并设置为源码包(Use a Source Folder)，在其下创建Dockerfile文件，例如：

```dockerfile
FROM base/java:1.8
ADD test-service-1.0.1.jar app.jar
ENTRYPOINT ["sh","-c","exec java $JAVA_OPTS -jar /app.jar $APP_ENV"]
```

FROM 指令指定了基础docker image镜像；

ADD 添加target目录下test-service-1.0.1.jar到docker镜像的/app.jar。

ENTRYPOINT 指定了docker启动时只需的命令行。

#### jenkins部分

2.1 创建任务;

2.2 构建一个自由风的软件项目;

2.3 Genreal，输入描述，简介明了；

2.4 源码管理，参见“3.1.2 源码管理"根据代码来源配置git或subversion；

2.5 构建触发器，参见"3.1.3 构建触发器"，例如要构建一个定时build的任务，则可以配置"定时构建"；

2.6 构建环境，参见"3.1.5 构建环境"，例如构建前要删除workspace内的文件(获取一个干净的环境)；

2.7 构建，参见"3.1.6 构建"，参见"调用顶层maven目标"；

2.8 构建，参见"3.1.6 构建"，参见"execute shell"，把/src/main/docker/Dockerfile文件拷贝到工程的target目录下，例如：

```bash
cp $WORKSPACE/src/main/docker/Dockerfile $WORKSPACE/target
```

2.9 构建，参见"3.1.6 构建"，参见"Execute Docker command -> Create/build image"，构建docker image，例如：

Build context foler，指定了Dockerfile文件所在的目录，例如：$WORKSPACE/target

Tag of the resulting docker image，指定创建docker image的tag，例如：dyit.com:5000/dy/test-service:1.0.2

2.10 构建，参见"3.1.6 构建"，参见"Execute Docker command -> Push image"，发布新构建的docker image到私有仓库，例如：

Name of image to push(repository/image) ，指定了要发布到docker的docker image镜像，例如：dyit.com:5000/dy/test-service，注意不包括tag部分。

Tag：指定了发布镜像的tag，例如：1.0.2

Docker registry URL：指定了docker 仓库的发布地址，例如：http://dyit.com:2375

Registry credentials：如果docker仓库访问需要用户名和密码，则需要先创建访问的凭证。

如果不能正常发布，则参见"2.1.1 系统配置 -> Docker Builder"，配置"Docker server REST API URL"。

2.11 构建，参见"3.1.6 构建"，参见"Execute shell script on remote host using ssh"，例如：

```
./bin/deploy.sh
```

docker服务器端deploy.sh脚本如下：

```bash
#!/bin/bash
# 从docker仓库中拉去新的镜像

# 停止并删除docker镜像

# 启动新的docker镜像


```

