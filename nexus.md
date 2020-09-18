# 安装和nexus(私服maven)

## 安装

useradd nexus

su - nexus

tar zxvf nexus-3.25.0-03-unix.tar.gz

解压缩后会在当前目录下生成两个目录：

nexus为执行文件目录，sonatype-work为生成文件目录(数据目录)。不建议修改这个目录位置，如果可以你可以拷贝这个sonatype-work目录和文件到其它目录，但需要建立软连接到当前目录(/home/nexus)，例如：ln -s 源目录 /home/nexus/sonatype-work

mv nexus-3.25.0-03 nexus

vi ./nexus/bin/nexus

查找run_as_root修改为false

启动

./nexus/bin/nexus start

查看状态

./nexus/bin/nexus status

修改端口

默认是8081

vi ~/nexus/etc/nexus-default.properties 

application-port=xxxx







第一次启动很慢，等待一下

netstat -ntlp 查看是否有8081端口



开防火墙允许访问8081端口



初始化



http://192.168.5.36:8081/

登录

Sign in

用户名：admin

密码，根据提示框查看/home/nexus/sonatype-work/nexus3/admin.password文件，内admin初始密码。

修改admin密码

是否允许匿名访问，为了安全，应该禁止



主页面头(header)上的，有使用和设置两个图标注意。



删除nuget仓库

nuget仓库是微软net的仓库，可以删除。



修改maven-central仓库

Repository->Repositories->maven-central点击

修改

Proxy->Remote storage，修改为国内镜像https://maven.aliyun.com/repository/central，默认地址：https://repo1.maven.org/maven2/

这个应该根据你所在的环境，如果环境可以还是建议使用官网的https://repo1.maven.org/maven2/，你可以测试一下本地访问官网的速度。

点击save按钮保存



仓库的三种类型，你可以查看type项

proxy：是代理，可以设置多个，国内的：华为、阿里，国外的：maven2等等，指的是如果你当前私服没有可用jar，需要去哪下载。
hosted：本地的，指代当前私服。存放你上传的第三方jar、已下载的jar等。
group：管理本地和代理（以上两个）
配置顺序：先配置proxy和hosted，最后配置group管理他们。



修改maven-release

允许重复发布

Repository-repositores->maven-release

Hosted：Deployment ploicy：Allow redeploy（允许重复发布）

点击Save按钮

修改maven-snapshots

允许重复发布

Repository-repositores->maven-release

Hosted：Deployment ploicy：Allow redeploy（允许重复发布）

点击Save按钮



创建第三方仓库

Repository-repositores->Create repository(按钮)->选择maven2(hosted)

Name：3rdparty

Hosted：Deployment ploicy：Allow redeploy（允许重复发布）

点击Create repository按钮





修改maven-public

Repository-repositores->maven-public

Group->Member repositories

Available，把3rdpart加入到右侧的Members

调整右侧Members的顺序，调整后：

maven-release

3rdparty

maven-central

maven-snashots

点击Save按钮





下载maven仓库索引

System->Tasks

Task Name：download index  创建任务的名称

Notification emial：909933699@qq.com  邮件通知

Send Notification  on：Success or Failure 成功和失败都发送

Repository：maven-central  操作的仓库

task frequency：Manual  任务周期，Manual(手工,一次性)，其它为周期性。

回到任务主菜单

选择download index任务，点击run按钮。



nexus日志目录

/home/nexus/sonatype-work/nexus3/log



上传第三方包到3rdparty仓库

Browse-Upload - > 3rdparty

选择文件，根据扩展名，自动识别。

验证：

Browse->3rdparty，查看到刚上传的文件。







nexus的eclipse客户端

settings.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>


<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">


	<localRepository>I:\maven_repo</localRepository>


	<pluginGroups>

	</pluginGroups>


	<proxies>

	</proxies>


	<servers>
		<server>
			<id>releases</id>
			<username>zhangdb</username>
			<password>密码</password>
		</server>
		<server>
			<id>snapshots</id>
			<username>zhangdb</username>
			<password>密码</password>
		</server>
	</servers>


	<mirrors>
		<mirror>
			<id>releases</id>
			<mirrorOf>*</mirrorOf>
			<url>http://192.168.5.36:8081/repository/maven-public/</url>
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

maven包更新失败

由于网络的各种原因，可以报更新失败，失败处理方法：

例如：

根据eclipse的Problems提示：

点击这个详细的Problem信息，例如：

上部分为出差项目的信息：

On element: dy-oauth2，这个错误属于dy-oauth2项目

下部分为具体问题描述：

```
The container 'Maven Dependencies' references non existing library 'I:\maven_repo\commons-collections\commons-collections\3.2\commons-collections-3.2.jar'
```

查看这个本地目录：I:\maven_repo\commons-collections\commons-collections\3.2，是否存在commons-collections-3.2.jar这个文件，如果不存在则说明上传maven更新失败了。

处理办法：

删除  I:\maven_repo\commons-collections\commons-collections\3.2 文件夹；

关闭dy-oauth2项目（eclispe 项目右键  Close Project)；

重新打开dy-oauth2项目（双击这个项目)；







