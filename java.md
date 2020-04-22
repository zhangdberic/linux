# linux java相关安装

## jdk

### 系统级别有效

```shell
tar -zxvf jdk-8u192-linux-x64.tar.gz
rm -rf /usr/local/jdk
rm -rf /usr/local/jre
mv jdk1.8.0_181 /usr/local/jdk
rm -rf /usr/bin/java
rm -rf /usr/bin/javac
ln -s /usr/local/jdk/bin/java /usr/bin/java
ln -s /usr/local/jdk/bin/javac /usr/bin/javac
ln -s /usr/local/jdk/jre /usr/local/jre
```

设置环境变量

***\*vi /etc/profile.d/my_profile.sh 添加如下内容\****

```shell
# ======= java environment =======

JAVA_HOME=/usr/local/jdk
export JAVA_HOME
JRE_HOME=/usr/local/jre
export JRE_HOME

PATH=$JAVA_HOME/bin:$PATH
export PATH

CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export CLASSPATH

ulimit -c unlimited
```

运行，使环境变量生效

chmod 777 /etc/profile.d/my_profile.sh

source /etc/profile.d/my_profile.sh

退出shell环境重新登录

## maven

### 系统级别有效

```shell
 wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
 tar xvf apache-maven-3.6.3-bin.tar.gz
 mv apache-maven-3.6.3 /usr/local/maven
 ln -s /usr/local/maven/bin/mvn /usr/bin/mvn
```

如果你是私有仓库配置，还需要修改settings.xml，全局的位置：/usr/local/maven/conf/settings.xml

先备份一下：cp /usr/local/maven/conf/settings.xml /usr/local/maven/conf/settings.xml.bak

修改settings.xml，根据自己系统的情况吧。







