# jenkins文档

## 阿里镜像

https://developer.aliyun.com/mirror/

## 1.安装和配置

useradd jenkins
su - jenkins

参照"tomcat文档"，安装和配置tomcat。

下载最新稳定版的jenkins.war

wget https://mirrors.aliyun.com/jenkins/war-stable/latest/jenkins.war

把jenkins.war放在tomcat的webapps目录下

注意：server.xml配置的unpackWARs="true"，否则无法自动解压jenkins.war文件。

启动tomcat

### 隐藏的.jenkins文件

注意这个带点的jenkins目录[.jenkins]，这里存放jenkins的数据文件。

/home/jenkins/.jenkins

### 初始化

http://192.168.1.250:8080/jenkins

按照界面上的提示，查看这个文件，得到初始化密码

 cat /home/jenkins/.jenkins/secrets/initialAdminPassword

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

**jenkins重启一次**

http://192.168.1.250:8080/jenkins/restart



### 升级

日后如果有新版本了，可以通过如下步骤升级；

**server.xml配置的unpackWARs="true"，否则无法自动解压jenkins.war文件。**

停止tomcat

```
~/tomcat/bin/shutdown.sh
```

下载最新的**稳定版本**的jenkins.war

```
cd ~/soft
wget https://mirrors.aliyun.com/jenkins/war-stable/latest/jenkins.war
```

拷贝新的jenkins.war包文件到tomcat/webapps目录下

```
cp ~/soft/jenkins.war ~/tomcat/webapps/
```

启动tomcat

```
./tomcat/bin/startup.sh
```



## 2.设置

### 2.1 系统管理

#### 2.1.1 系统配置(Configure System)

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

​     **点击"Add"按钮**

SSH Servers 

name 起个名字，例如：POS-192.168.5.78

Hostname 主机地址

username 登录主机用户名

Remote Directory 远程目录，操作的起始目录，注意：是起始目录。例如：如果这项设置的是/software，那么如果执行命令：mkdir -p /xxx，不会在远程服务器的根据目录下创建xxx目录，而是创建了/software/xxx。如果你要创建在根目录下，这项应该设置为 /  ；

​     **点击高级按钮**

选择使用“Use password authentication, or use a different key"，使用密码认证；

Passphrase / Password 输入上面username对应的密码；

Port 指定ssh端口；

设置后点击“Test Configuration"，测试配置和ssh连接是否正确。

报错处理：

```
jenkins.plugins.publish_over.BapPublisherException: Failed to connect and initialize SSH connection. Message: [Failed to connect SFTP channel. Message [java.io.IOException: inputstream is closed]]
```

ssh连接服务器端的/etc/ssh/sshd_config配置有错误，修改如下，必须保证sftp-server文件存在：

```
# override default of no subsystems
Subsystem   sftp    /usr/libexec/sftp-server
```

报错处理：

```
jenkins.plugins.publish_over.BapPublisherException: Failed to connect and initialize SSH connection. Message: [Failed to connect session for config [POS_10.60.32.198_SPRINGCLOUD]. Message [Auth fail]]
```

没有在ssh的配置文件中加入允许这个用户访问配置。

```
allowUser xxxx
```



##### Docker Builder

Docker URL 设置 Docker server REST API URL，例如：tcp://192.168.1.250:2375/，设置后点击Test Connection按钮来测试，连接是否正确，连接成功提示：Connected to tcp://192.168.1.250:2375

#### 2.1.2 全局工具配置(Global Tool Configuration)

**Maven安装**

su - jenkins

cd ./soft

 wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz

tar xvf apache-maven-3.6.3-bin.tar.gz

mv apache-maven-3.6.3 ~/maven

cp ~/maven/conf/settings.xml ~/maven/conf/settings.xml.bak

mkdir -p home/jenkins/maven_repo   # 本地仓库目录

vi ~/maven/conf/settings.xml 

```xml
<?xml version="1.0" encoding="UTF-8"?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>/home/jenkins/maven_repo</localRepository>

  <pluginGroups>
  </pluginGroups>

  <proxies>
    <proxy>
      <id>my-proxy</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>10.60.32.xxx</host>
      <port>11001</port>
      <username>user</username>
      <password>name</password>
   </proxy>
  </proxies>


  <servers>
        <server>
                <id>releases</id>
                <username>zhangdb</username>
                <password>xxxxxxx</password>
        </server>
        <server>
                <id>snapshots</id>
                <username>zhangdb</username>
                <password>xxxxxxx</password>
        </server>
  </servers>

  <mirrors>
        <mirror>
                <id>releases</id>
                <mirrorOf>*</mirrorOf>
                <url>http://maven.dongyuit.cn:8081/nexus/content/groups/public/</url>
        </mirror>
  </mirrors>

  <profiles>

        <profile>
                <id>nexus</id>
                <repositories>
                        <repository>
                                <id>central</id>
                                <url>http://central</url>
                                <releases>
                                        <enabled>true</enabled>
                                </releases>
                                <snapshots>
                                        <enabled>true</enabled>
                                </snapshots>
                        </repository>
                </repositories>
                <pluginRepositories>
                        <pluginRepository>
                                <id>central</id>
                                <url>http://central</url>
                                <releases>
                                        <enabled>true</enabled>
                                </releases>
                                <snapshots>
                                        <enabled>true</enabled>
                                </snapshots>
                        </pluginRepository>
                </pluginRepositories>
        </profile>

  </profiles>


  <activeProfiles>
    <activeProfile>nexus</activeProfile>
  </activeProfiles>


</settings>

```



##### Maven配置(Maven Configuration)

选择settings file in filesystem

选择Global settings file in filesystem

两个settings的路径都设置为：

```
File path：/home/jenkins/maven/conf/settings.xml
```

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

```
Name maven3
MAVEN_HOME /home/jenkins/maven/
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

如果插件安装失败，应该使用http请求的方式来重新启动jenkins，例如：

http://192.168.1.2/jenkins/restart



## 3.构建

### 3.1.构建maven项目

构建maven项目依赖于上面的安装和配置文档(例如，依赖java_home和maven_home的设置)。

#### 3.1.1 General

描述，输入项目的描述信息；

#### 3.1.2 源码管理

##### Subversion

**Repository URL：**

注意：可以支持svn原生的svn协议；但需要在操作系统上，先安装svn命令，yum -y install svn

```
svn://svn.xxx.cn:9999/zframework/sgw
```

**Credentials：**

凭证，svn的用户名和密码，创建后的凭证可以在多个项目上复用；

**Local module directory：**

分为如下几种情况：

[.]点符合(默认)：把指定的svn url下内容检出到**根目录**，把指定的svn url，例如：checkout后在workspace目录下创建src目录、pom.xml文件。

如果是空白：把指定的svn url下内容检出到**svn url最后路径名,命名的目录下**，例如：checkout后在workspace的sgw目录下。

指定目录：把指定的svn url下内容检出到**指定目录下**，例如，指定为sss：checkout后在workspace的sss目录下。

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

#### 3.1.3 构建触发器(Build Triggers)

##### 定时构建

日常表

```
TZ=Asia/Shanghai
H 0 * * *
```

第一个TZ指定了时区，因为定时任何依赖于时间，而时间又依赖于时区；

H 0 * * *，这个例子：每天的0点触发构建，为什么第1个，month(分钟)是H而不是0，这是jenkins特殊的字符，其会根据负载情况，决定是0点0分运行，还是0点x分允许，而且jenkins提倡使用H。

#### 3.1.5 构建环境(Build Environment)

##### 先删除workspace内的文件

Delete workspace before build starts 构建前先删除这个项目，**生产环境建议选择**；

#### 3.1.6 构建()

##### maven构建

选择：调用顶层Maven目标

Maven版本 ：选择maven，这个是在“系统管理->全局工具配置-Maven”中安装时配置的别名。

目录：maven构建过程命令，例如：

clean package -Dmaven.test.skip=true -U

解释：

clean为先清理

package为编译打包

-Dmaven.test.skip=true跳过测试

-U为强制更新maven依赖包(-U 是强制检查的意思，-U并不是每次都会拉最新的，只有在时间戳落后于私库上的jar的时候才会下载最新)，其会下载依赖包所在的上级目录内的maven-metadata.xml文件，这个文件包括所有的版本，其会根据dependency的version属性来判断是否要重新获取依赖jar文件。

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

```
构建的文件位置：/home/jenkins/.jenkins/workspace/{jenkins项目名}/target
```

Remove prefix：传输过去后要去掉前置，例如：target/，传输过去后，只保留sgw-1.0.1.jar；

Remote directory：传输到远程的目录，注意：这个依赖于Publish over SSH配置的Remote Directory起始目录，例如:/target。

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

```
Source file: target/xxxx.jar  ,例如:target/dy-config-1.0.1.jar
Remove prefix: target
Remote directory: target
Exec command: ./bin/deploy.sh
```

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

目标服务创建用户，并初始化目录结构：

```bash
useradd xxx
su - xxx
mkdir target
mkdir bin
mkdir code_history
mkdir logs
cd bin
```

服务器端**tomcat环境deploy.sh**脚本如下：

```bash
#!/bin/bash
source $HOME/.bashrc

# 变量
nginx_home=$HOME/nginx
tomcat_home=$HOME/tomcat
package_filename=ROOT.war
package_filepath=$HOME/target/$package_filename
deploy_dir=$HOME/tgms
deploy_static_dir=$HOME/tgms_static

# 判断程序包是否存在
if [ ! -f $package_filepath ]; then
  echo $package_filepath 'does not exist.'
  exit 0
fi

# 停止
if [ -f $nginx_home/logs/nginx.pid ]; then
   $nginx_home/sbin/nginx -s stop
fi
$tomcat_home/bin/shutdown.sh
sleep 5s
pid=$(ps aux|grep '/home/tgms/jdk1.8/jre/bin/java'|grep -v "grep"|awk '{print $2}')
if [ -n "$pid" ]
then
    kill -9 $pid
    echo 'kill -9' $pid
fi
echo 'shutdown ok.'

# 备份
cp -r $deploy_dir $HOME/code_history/tgms`date +"%F-%T"`
echo 'backup ok.'

# 清除
rm -rf $deploy_dir
rm -rf $deploy_static_dir
echo 'clear ok.'

# 发布
mkdir $deploy_dir
cp -r $package_filepath $deploy_dir
cd $deploy_dir
$JAVA_HOME/bin/jar xf ROOT.war
rm -rf ROOT.war
mkdir $deploy_static_dir
cp -r $deploy_dir/* $deploy_static_dir
rm -rf $deploy_static_dir/WEB-INF
echo 'deploy ok.'

# 启动
$tomcat_home/bin/startup.sh
sleep 10s
$nginx_home/sbin/nginx
echo 'startup ok.'
```

spring boot的**jar环境deploy.sh**脚本如下：

**./bin/deploy.sh**

这是一个通用的发布脚本，你只需要修改第一个变量块就可以了：

app_name 应用名

app_version 版本

port 访问端口

profile 指定环境

springboot_props 覆盖的springboot属性，例如：

springboot_props='--spring.cloud.config.server.git.password=xxxx'

```bash
#!/bin/bash

app_name=dy-config
app_version=1.0.1
port=9000
profile=test
springboot_props=''

package_filename=$app_name-$app_version.jar
package_filepath=$HOME/target/$package_filename
deploy_filepath=$HOME/$package_filename

echo "=============================="
echo "deploy $package_filename"
echo "=============================="

# 判断程序包是否存在
if [ ! -f $package_filepath ]; then
  echo $package_filepath 'does not exist.'
  exit 0
fi

# 关闭系统(先使用kill -15 pid发送关闭通知,如不能正常关闭再使用kill -9 pid强制关闭)
pid=$(ps aux|grep 'java'|grep $app_name|grep -v "grep"|awk '{print $2}')
if [ -z "$pid" ]
then
    echo 'Application is already stopped'
else
    kill -15 $pid
    echo 'kill -15 '$pid
fi
sleep 5
pid=$(ps aux|grep 'java'|grep $app_name|grep -v "grep"|awk '{print $2}')
if [ -n "$pid" ]
then
    kill -9 $pid
    echo 'kill -9 '$pid
fi
echo 'shutdown ok.'

# 备份
if [ -f $deploy_filepath ]; then
  mv $deploy_filepath $HOME/code_history/$package_filename.`date +"%F-%T"`
  echo 'backup ok.'
fi

# 清理
rm -rf $deploy_filepath
echo 'clear ok.'

# 发布
cp $package_filepath $deploy_filepath
echo 'deploy ok.'

# 启动系统
nohup java -Dlogging.loghome=$HOME/logs -Djava.security.egd=file:///dev/urandom -server -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Xmx512m -Xms512m -Xmn200m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -jar $deploy_filepath --spring.profiles.active=$profile --server.port=$port $springboot_props > $HOME/logs/${app_name}_nohup.log 2>&1 &
echo 'startup ok.'


```

注意：上面脚本的kill部分，无法正确的识别一个唯一的应用进程，例如：两个用户分别启动了sgw和sgw-manager，那么在sgw-deploy.sh的脚本运行时候就会识别两个pid，在这种情况下，可以根据具体情况，重新修改：pid=$(ps aux|grep 'java'|grep $app_name|grep -v "grep"|awk '{print $2}')，例如：加入$HOME，修改后pid=$(ps aux|grep 'java'|grep $HOME/$app_name|grep -v "grep"|awk '{print $2}')。



### 2.docker打包(maven构建 -> docker仓库发布)

#### 程序部分

**dockerfile**

创建目录"/src/main/docker"，并设置为源码包(Use a Source Folder)，在其下创建Dockerfile文件，例如：

```dockerfile
FROM dockerdongyuit.cn:5000/java
RUN useradd -s /bin/bash 用户名
USER 用户名
ADD test-service-1.0.1.jar app.jar
ENTRYPOINT ["sh","-c","exec java $JAVA_OPTS -jar /app.jar $APP_ENV"]
```

FROM 指令指定了基础docker image镜像；

USER 指定了运行的用户; 

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

Docker registry URL：指定了docker 仓库的发布地址，例如：tcp://dyit.com:2375

Registry credentials：如果docker仓库访问需要用户名和密码，则需要先创建访问的凭证。

如果不能正常发布，则参见"2.1.1 系统配置 -> Docker Builder"，配置"Docker server REST API URL"。

2.11 构建，参见"3.1.6 构建"，参见"Execute shell script on remote host using ssh"，例如：

```
./bin/deploy.sh
```

docker服务器端deploy.sh脚本如下：

```bash
#!/bin/bash
$HOME/stop_container.sh dy-config2
/usr/bin/docker rmi $(docker images dockerdongyuit.cn:5000/dy-config -q)
docker pull dockerdongyuit.cn:5000/dy-config:1.0.1
docker run -itd --cap-add=SYS_PTRACE --user dy-config --name dy-config2 --net host -e JAVA_OPTS="-Xms1g -Xmx1g -Xmn300m -XX:+UseParNewGC -XX:+UseConcMarkSweepGC" -e APP_ENV="--spring.profiles.active=proc --spring.cloud.config.server.git.password=xxxxxxx --server.port=8081" dockerdongyuit.cn:5000/dy-config:1.0.1 
docker logs -f dy-config2
```

**注意**：因为docker是基于镜像堆叠，例如：dy-config:1.0.1镜像的FROM镜像是java:1.8，如果你的客户端不存在这个java:1.8镜像，则pull将下载整个dy-config:1.0.1(1.1G)非常耗时。你可以先首次pull下载java:1.8镜像，以后每次在pull下载dy-config:1.0.1，只下载在其上堆叠的dy-config-1.0.1.jar，非常快。