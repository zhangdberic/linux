# tomcat

## tomcat9

### 准备工作

安装jdk1.8

su - xxx # 切换到指定用户下

### 下载最新的tomcat9

wget https://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.35/bin/apache-tomcat-9.0.35.tar.gz

tar xvf apache-tomcat-9.0.35.tar.gz

mv apache-tomcat-9.0.35/ tomcat

rm -rf apache-tomcat-9.0.35.tar.gz

### 修改server.xml配置文件

cp ~/tomcat/conf/server.xml ~/tomcat/conf/server.xml.default

vi ~/tomcat/conf/server.xml 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

  <Service name="Catalina">
     
    <Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="150" minSpareThreads="4"/>   

    <Connector executor="tomcatThreadPool"
               port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" address="127.0.0.1" enableLookups="false" URIEncoding="utf-8" useBodyEncodingForURI="true" maxKeepAliveRequests="1000" keepAliveTimeout="3600000"/>

    <Engine name="Catalina" defaultHost="localhost">
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="false">

        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />

      </Host>
    </Engine>
  </Service>
</Server>
```

注意：如果要开放tomcat端口，则address="127.0.0.1"修改为address="0.0.0.0"

官方配置：http://tomcat.apache.org/tomcat-9.0-doc/config/index.html

### 安全清理

rm -rf ~/tomcat/webapps/*

rm -rf ~/tomcat/conf/tomcat-users.*

rm -rf ~/tomcat/bin/*.bat

rm -rf ~/tomcat/work/Catalina/localhost/*

### 编辑catalina.sh

vi ~/tomcat/bin/catalina.sh

加入JAVA_OPTS

```properties
TOMCAT_HOME="$HOME/tomcat"
JAVA_OPTS="-Djava.security.egd=file:///dev/urandom -server -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Xmx3g -Xms3g -Xmn1g -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=256m -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -Xloggc:$TOMCAT_HOME/logs/jvm_gc.log -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=$TOMCAT_HOME/logs/head.dump -XX:ErrorFile=$TOMCAT_HOME/logs/java_error.log"
```

### 验证

~/tomcat/bin/catalina.sh run



