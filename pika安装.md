# 360 pika

## 安装

### **安装环境介绍**

操作系统：CentOS 7.6.1810

pika版本：v3.0.7

安装方式：编译安装

在github上，pika项目有release出对应的二进制，但是都是针对CentOS5、CentOS6，没有CentOS7。编译安装也很简单。

### **源码编译**

\# 1、安装依赖库（有一些包epel-release源上才有）

yum -y install epel-release && yum -y install snappy-devel glog-devel gflags-devel

\# 2、安装gcc、make，推荐使用gcc4.8以上的版本

yum install gcc-c++ make

\# 3、安装git，从github直接拉取代码库，也可以进入release页面下载对应版本的源码

yum -y install git

\# 4、clone代码库

git clone https://github.com/Qihoo360/pika.git

\# 5、查看获取版本标签，找到想要的版本

cd pika

git tag

\# 6、切换到tag的代码位置

git checkout v3.0.7

\# 7、使用make进行编译

make

\# 8、output目录就是编译出来的二进制文件所在的目录，直接拷贝到想要安装的目录就完成安装了。

cp -rp output /opt/pika

\# 9、启动pika

cd /opt/pika

./bin/pika -c ./conf/pika.conf

\# 10、测试

redis-benchmark -p 9221

## 常用脚本

**client.sh**

/home/pika/pika/redis-4.0.11/src/redis-cli -h 127.0.0.1 -p 9221

 **startup.sh**

/home/pika/pika/output/bin/pika -c /home/pika/pika/output/conf/pika.conf

 **stop.sh**

/home/pika/pika/redis-4.0.11/src/redis-cli -h 127.0.0.1 -p 9221 shutdown





