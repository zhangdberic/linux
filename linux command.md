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

### 同一块网多IP地址

**加入另一个ip**

例如：我们要为eth0网卡添加另一个ip地址

cd /etc/sysconfig/network-scripts

cp ifcfg-eth0 ifcfg-eth0:1

vi ifcfg-eth0:1

修改DEVICE=eth0:1

加入ip地址：

IPADDR=10.60.9.160
NETMASK=255.255.255.192

**验证**

service network restart

ip a 查看是否生效

**添加路由**

例如：为eth0网卡添加路由文件

vi /etc/sysconfig/network-scripts/route-eth0

格式：ip/netmask via getway dev 逻辑网卡名

例如：

```
10.56.6.0/24 via 10.60.9.129 dev eth0 
10.60.32.0/24 via 10.60.9.129 dev eth0 
10.60.33.0/24 via 10.60.9.129 dev eth0
```



### 防火墙(firewalld)

好文章：http://www.bubuko.com/infodetail-2754116.html

或者搜索：centos7 firewall-cmd 理解多区域配置中的 firewalld 防火墙，firewall要写明白比较复杂，我这里复制了网上的一片文章，写的比较好，可以查看firewall.md文档。

#### 基本命令

查看服务状态

systemctl status firewalld.service  安装centos后默认就开启

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

#### 临时修改和永久保留

临时修改，下次重新加载时被覆盖：

firewall-cmd <some modification>

永久保留，下次重新加载后会永久保存：

firewall-cmd --permanent <some modification>
firewall-cmd --reload

#### 获取定义区域

firewall-cmd --get-zones

```
block dmz drop external home internal public trusted work
```

上面返回的这些都是系统默认的区域，正常情况下系统初始化后，就public区域是激活(active)状态。

#### 返回已经被激活(使用)区域

firewall-cmd --get-active-zones

例如:这里返回了两个区域，正常系统默认的就public一个区域，这里返回的区域：

internal为内部区域，为**源区域**定义ip的访问范围。

public为公共区域，接口区域，其它区域的处理不了的都由公共区域来处理，公共区域还解决不了的默认拒绝(reject)。

```
internal
  sources: 192.168.5.0/24 10.60.32.198
public
  interfaces: eth0
```



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

#### 端口转发

firewall-cmd --add-masquerade --permanent
firewall-cmd --add-forward-port=port=9999:proto=tcp:toaddr=192.168.5.36:toport=9999 --permanent
firewall-cmd --reload
firewall-cmd --list-all

或者

vi /etc/firewalld/zones/public.xml

```xml
  <masquerade/>
  <!-- forward svn -->
  <forward-port to-addr="192.168.5.36" to-port="9999" protocol="tcp" port="9999"/>
```

 

#### 查看某个用户网络流量

yum install -y nethogs

运行：nethogs

只查看某个网卡：nethogs ethX



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

禁止开机启动

```
systemctl disable xxx
```



## 用户

### 修改用户uid和gid

例如：

 #test用户uid改为100,gid改为101

```
usermod -u 100 test && groupmod -g 101 test
```

### 清零失败次数

sgw-manager为用户名

```
pam_tally2 -u sgw-manager --reset
```

### 修改用户所在的用户组

app为修改后的用户组，web1为要修改的用户，这里理解修改web1的隶属用户组到app，前提这个app的用户组必须存在。

```
 usermod -g app web1
```

这里修改了用户所在用户组，如果再新建文件则文件隶属于修改后的用户组(新)。但以前的文件还是修改前的用户组，因此要使用下面的命令来修改：

```
chgrp -R app /home/web1
```

/home/web1默认的访问权限drwx------ 只能自身可以访问，如果要组内访问，这要使用下面语句修改为drwxr-x---

```
chmod 750 /home/web1
```



## 进程

### kill

当你调用kill命令时，不只是使用kill -9 pid，其实还是有很多的数字可以代替9的而且更可以更优雅的关闭进程。

下面列出了所有的数字和对于的意义：

```
1) SIGHUP     2) SIGINT     3) SIGQUIT     4) SIGILL     5) SIGTRAP
6) SIGABRT     7) SIGBUS     8) SIGFPE     9) SIGKILL    10) SIGUSR1
11) SIGSEGV    12) SIGUSR2    13) SIGPIPE    14) SIGALRM    15) SIGTERM
16) SIGSTKFLT    17) SIGCHLD    18) SIGCONT    19) SIGSTOP    20) SIGTSTP
21) SIGTTIN    22) SIGTTOU    23) SIGURG    24) SIGXCPU    25) SIGXFSZ
26) SIGVTALRM    27) SIGPROF    28) SIGWINCH    29) SIGIO    30) SIGPWR
31) SIGSYS    34) SIGRTMIN    35) SIGRTMIN+1    36) SIGRTMIN+2    37) SIGRTMIN+3
38) SIGRTMIN+4    39) SIGRTMIN+5    40) SIGRTMIN+6    41) SIGRTMIN+7    42) SIGRTMIN+8
43) SIGRTMIN+9    44) SIGRTMIN+10    45) SIGRTMIN+11    46) SIGRTMIN+12    47) SIGRTMIN+13
48) SIGRTMIN+14    49) SIGRTMIN+15    50) SIGRTMAX-14    51) SIGRTMAX-13    52) SIGRTMAX-12
53) SIGRTMAX-11    54) SIGRTMAX-10    55) SIGRTMAX-9    56) SIGRTMAX-8    57) SIGRTMAX-7
58) SIGRTMAX-6    59) SIGRTMAX-5    60) SIGRTMAX-4    61) SIGRTMAX-3    62) SIGRTMAX-2
63) SIGRTMAX-1    64) SIGRTMAX
```

下面重点介绍几个重要的：

#### kill -2 pid

kill -2 pid，模拟ctrl+c，发送SIGINT信号，大部分程序可以相应SIGINT信号，例如：springboot程序，但其有个问题，如果你的spring boot程序是基于nohup & 后台运行的，则不会响应SIGINT信号也就说不会响应ctrl+c。这个需要kill -15 pid来代替。

#### kill - 15 pid

kill -15 pid，发送SIGTERM信号，程序可以选择响应这个信号情况，也可以不响应。spring boot可以响应这个信号，其在spring boot application注册的时候会注册一个钩子线程，钩子响应SIGTERM信号，实现优雅的退出。



## CURL

### POST提交

格式：

```shell
curl 请求url -X POST -d 'POST内容'
```

例如：

```
curl localhost:3000/api/basic -X POST -d 'hello=world'
```

### 加入请求头

加入--header 'name:value'

例如：

```
--header "User-Agent:Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36"
```

### 上传文件请求

加入-F参数，上传文件 -F "file=@/software/8888.jpg"，这里的/software/8888.jpg为文件路径。如果带上请求参数， -F "purview=2"

```
curl -F "file=@/software/8888.jpg" -F "purview=2"  -v 'http://10.60.33.18/services?appkey=test&service=dfss.upload&version=1.0&serialnum=4f7bb19b83b94ab2b48fde0d8da624c4&timestamp=1595395828&sign=0CA57636354F65AFC764F592FD0F1A7E41960FB7&format=json&protocol=openapi&resource=0'
```



### 显示请求头和响应头信息

加入-I参数

例如：

```
curl -I 'http://10.60.33.18:7070/services?appkey=test&service=dfss.get_metadata&version=1.0&serialnum=0a368d56b52f432191475a384442ba7d&timestamp=1595383656&sign=C9312BA00561A22FB8560FB51698282907E703DC&format=json&protocol=openapi&resource=0&id=/dfss/fss1/data/2020/07/22/2O6YRx2R0D4ECTBY4wiwM3632233996.txt'
```



### 显示请求和响应详细信息

加入-v参数

例如：

```
curl localhost:3000/api/basic -X POST -d 'hello=world' -v
```

### 计算base64

```
echo -n "用户名:密码" | base64
```

### 发送认证请求

```
curl -v -H "Authorization: Basic 认证信息" -X GET https://docker.yun.ccb.com/v2/<name>/tags/list
```

### basic认证

```
curl -u dy-config:12345678 -X POST http://192.168.5.76:29000/actuator/bus-refresh/sgw
```

### HTTPS访问

```
curl -k --tlsv1 https://www.baidu.com
```



## grep

查找**符合条件**表达式(字符串)

```
grep 'pattern' file.txt
```

查找**不符合条件(NOT)**表达式(字符串) 加入 -v 

```
grep -v 'pattern' file.txt
```

高亮显示

你可以在 ~/.bashrc 内加上这行：『alias grep='grep --color=auto'』再以『 source ~/.bashrc 』来立即生效即可喔！ 这样每次运行 grep 他都会自动帮你加上颜色显示啦

**OR操作**

符号：

```
\|
```

来表示或者，例如：

```
grep 'appkey=tgms\|appkey=wccy\|appkey=lnbcbzp\|appkey=qybs\|jtbscjx'
```

**OR和NOT混合使用**

```
grep -v 'appkey=tgms\|appkey=wccy\|appkey=lnbcbzp\|appkey=qybs\|jtbscjx'
```

## awk

### awk输出第几列

awk '{print $x}'

例如，输出第4列

```
awk '{print $4}'
```

### 指定分隔符

awk默认的分隔符为空格，你可以手工指定分隔符，例如，使用:符号作为分隔符

awk -F ':'

例如：

```
awk -F ':' '{print $4}'
```

### 输出固定字符串

在print内输出固定字符串，使用双引号括固定字符串，例如：

```
awk -F ':' '{print "insert into SERVER(ip,port) values('\''10.60.33.21'\''," $2 ");"}'
```

如果要输出单引号、双引号则要加入转移字符，例如上面的单因为转移字符。

## find

### 查看目录(递归)下的所有文件名和目录名

例如：

```
find /home/ygjzxfw/tomcat8/webapps
```

### 只显示文件名或目录名

```
find /home/ygjzxfw/tomcat8/webapps -type f
find /home/ygjzxfw/tomcat8/webapps -type d
```

### 过滤文件名

例如：

```
find . -name '*.jpg'
```

### 过滤文件名或(or)操作

```
find /home/ygjzxfw/tomcat8/webapps -name '*.jpg' -o -name '*.png'  -type f
```

### 过滤文件名否定(not)操作

```
find /home/ygjzxfw/tomcat8/webapps ! -name '*.log*' -type f
```

### 查找后直接操作文件

例如：查找后直接计算文件的md5值

```
find /home/ygjzxfw/tomcat8/webapps ! -name '*.log*' -type f -print0 | xargs -0 md5sum
```

## 发邮件

1.注册一个163.com的邮件。

2.开启POP3/SMTP服务，默认是关闭的，开启后会给你的授权码，这个要留好，下面要用。

3.centos6

yum -y install mail

vi /etc/mail.rc

加入：

```
set from=dongyuitheige@163.com smtp=smtp.163.com
set smtp-auth-user=dongyuitheige@163.com smtp-auth-password=授权吗 smtp-auth=login
```

4.发送邮件

邮件内容 | mail -s 标题 目的地邮箱

例如：

```
cat /root/monitor/ygjzxfw_md5_diff_result.txt | mail -s 在线查询被篡改 909933699@qq.com
```



## 磁盘

### 系统盘扩容

在没有lvm情况下，为系统盘扩容。

```
yum install cloud-utils-growpart
yum install xfsprogs
```

lsblk 命令，查看当前磁盘容量和已经分配给分区的容量；

df -Th 命令，查看当前系统挂载情况、容量、文件系统类型；

growpart /dev/vda 1 命令，对/dev/vda的第一个分配进行扩容；

如果报错：unexpected output in sfdisk --version [sfdisk，来自 util-linux 2.23.2]，则修改系统字符集：LANG=en_US.UTF-8，然后重新执行这个命令，如果还报错，则locale查看，保证LC_ALL=en_US.UTF-8，如果不是则也要设置这个变量，再重试。

成功提示：CHANGED: partition=1 start=2048 old: size=83883999 end=83886047 new: size=209713119 end=209715167

df -Th 命令，查看当前系统挂载情况、容量、文件系统类型；

根据文件系统扩容：

扩展文件系统。

根据文件系统类型选择以下扩展方式，如何查看文件系统类型请参见步骤。

- ext*文件系统（例如ext3和ext4）：运行

  ```
  resize2fs <PartitionName>
  ```

  命令。

  示例命令表示为扩容系统盘的/dev/vda1分区的文件系统。

  ```
  [root@ecshost ~]# resize2fs /dev/vda1
  resize2fs 1.42.9 (28-Dec-2013)
  Filesystem at /dev/vda1 is mounted on /; on-line resizing required
  old_desc_blocks = 3, new_desc_blocks = 7
  The filesystem on /dev/vda1 is now 26214139 blocks long.
  ```

- xfs文件系统：运行

  ```
  xfs_growfs <mountpoint>
  ```

  命令。

  示例命令表示为扩容系统盘的/dev/vda1分区的文件系统。其中根目录（/）为/dev/vda1的挂载点。

  ```
  [root@ecshost ~]# xfs_growfs /
  meta-data=/dev/vda1              isize=512    agcount=13, agsize=1310656 blks
           =                       sectsz=512   attr=2, projid32bit=1
           =                       crc=1        finobt=1, sparse=1, rmapbt=0
           =                       reflink=1
  data     =                       bsize=4096   blocks=15728379, imaxpct=25
           =                       sunit=0      swidth=0 blks
  naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
  log      =internal log           bsize=4096   blocks=2560, version=2
           =                       sectsz=512   sunit=0 blks, lazy-count=1
  realtime =none                   extsz=4096   blocks=0, rtextents=0
  data blocks changed from 15728379 to 20971259
  ```

最好能reboot重启系统，df -Th观察；

## SSH

### 远程登录

```
ssh -p{ssh_port} {username}@{ip}
```

例如：

ssh -p10911 sgw-manager@10.60.33.18

## Apache AB

apache ab工具用于压力测试

### 安装

```
 yum -y install httpd-tools
```

### 查看请求和响应内容

通常用于在压力测试前，检查URL是否有效，是否可以正常返回期望的内容。

ab -v2 'url'

例如：

```
ab -v2 'http://10.60.33.18:5500/get_metadata?appkey=test&service=dfss.get_metadata&version=1.0&serialnum=718d2d900b2b48919224ac398d1b7292&timestamp=1595212963&sign=53BFF757350227A28A29790757285D84BF331BD5&format=json&protocol=openapi&resource=0&id=/dfss/fss1/data/2020/07/20/2E9XmlbXDFQc2vOlPrfvO3632233996.txt'
```

### 压力测试

ab -n请求数量 -c并发数 -k 'url'

参数-k：为保持(keepalive)socket长链接，这个参数对压力测试很重要，没有这个参数则会产生大量的短连接和closed状态连接，但需要http服务器也开启keepalive，spring boot tomcat默认已经已经开启了(keepalive-timeout=60,maxkeepalivedrequest=100)，你可以通过ss -s命令查看。

例如：

```
ab -n100000 -c100 -k 'http://10.60.33.18:5500/get_metadata?appkey=test&service=dfss.get_metadata&version=1.0&serialnum=718d2d900b2b48919224ac398d1b7292&timestamp=1595212963&sign=53BFF757350227A28A29790757285D84BF331BD5&format=json&protocol=openapi&resource=0&id=/dfss/fss1/data/2020/07/20/2E9XmlbXDFQc2vOlPrfvO3632233996.txt'
```

**压力测试结果**

```
Document Length:        70 bytes   # 单个请求返回字节数

Concurrency Level:      100 # 并发数
Time taken for tests:   5.555 seconds
Complete requests:      100000 # 完成请求个数
Failed requests:        0    # 失败请求数
Write errors:           0
Keep-Alive requests:    99048  # keepalive请求数(ab -k参数起作用的数量，复用的长链接请求数量)
Total transferred:      53472392 bytes
HTML transferred:       7000000 bytes
Requests per second:    18002.92 [#/sec] (mean)  # 每秒处理的请求数
Time per request:       5.555 [ms] (mean) # 并发数量内的请求数，例如：100个并发请求数消耗时间
Time per request:       0.056 [ms] (mean, across all concurrent requests)  # 每个请求消耗的时间
Transfer rate:          9400.97 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.1      0       3
Processing:     1    6   1.6      5      22
Waiting:        1    6   1.6      5      22
Total:          1    6   1.6      5      22

Percentage of the requests served within a certain time (ms)
  50%      5  # 50%的请求在5ms内完成
  66%      6
  75%      6
  80%      6
  90%      7
  95%      7
  98%     11
  99%     14  # 99%的请求在14ms内完成
 100%     22 (longest request)

```

```shell

```

### 错误处理

apr_socket_recv: Connection reset by peer (104)错误

```
 vim /etc/sysctl.conf 
 net.ipv4.tcp_syncookies = 0
 sysctl -p
```

如果还不行，你可以为ab加入-r参数，这个参数将忽略tcp连接错误。

tomcat对tcp连接的处理能力有限，可以前置nginx，ab请求发送nginx来进行压力测试，测试结果有一定的损耗，但不会太多。

### 上传文件

ab对上传文件的处理比较垃圾，需要前期做什么很多工作。

**1.生成满足rfc1867规范的文件**

创建一个spring boot web项目，创建FirstServlet类，其负责把上传的内容输出到文件中，输出的文件自动满足rfc1867格式。你可以使用postman或者curl来上传一个文件。

```java
@SpringBootApplication
@ServletComponentScan
public class FssApplication {

	public static void main(String[] args) {
		SpringApplication.run(FssApplication.class, args);
	}
	
}

```



```java
@WebServlet(name = "firstServlet", urlPatterns = "/test")
public class FirstServlet extends HttpServlet {

	private static final long serialVersionUID = 1L;

	@Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		System.out.println(req.getContentType());
		File outFile = new File("c:/8888.jpg");
		FileOutputStream output =null;
		try {
			output = new FileOutputStream(outFile);
			int size = copy(req.getInputStream(),output);
			System.out.println(size);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally {
			try {
				output.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	
	
	public static int copy(InputStream in, OutputStream out) throws IOException {
		Assert.notNull(in, "No InputStream specified");
		Assert.notNull(out, "No OutputStream specified");
			int byteCount = 0;
			byte[] buffer = new byte[1024];
			int bytesRead = -1;
			while ((bytesRead = in.read(buffer)) != -1) {
				out.write(buffer, 0, bytesRead);
				byteCount += bytesRead;
			}
			out.flush();
			return byteCount;
	}

}
```

**ab上传文件**

-T 'multipart/form-data; boundary=--------------------------717331577278210071020116'，必须有，其来至于上面的eclipse的servlet执行输出。

-p 为要上传的文件，这个文件就是上面servlet输出的文件，文件头和尾已经加入了rfc1867格式。

```
ab -n20000 -c100 -k -T 'multipart/form-data; boundary=--------------------------717331577278210071020116' -p /software/8888.jpg 'http://10.60.33.18/services?appkey=test&service=dfss.upload&version=1.0&serialnum=15ff51d7cbd44dd9a6800f50d293aca8&timestamp=1595403591&sign=407FE2751B12E361D2EFA4838CC5CB0F04CE387B&format=json&protocol=openapi&resource=0'
```

## yum

需要手工输入yes后才安装

yum install xxx

不需要手工确认直接安全

yum -y install xxx

显示已经安装的软件

yum list installed

## 打包压缩

### pigz多核压缩

神器，支持多核压缩和解压，基本可以压满CPU，特别适合多CPU、大内存、高IO存储。

gzip和biz2不支持多核pigz完美支持多核。

yum -y install pigz

**压缩**

tar --use-compress-program=pigz -cvpf package.tgz ./package

**解压**

tar --use-compress-program=pigz -xvpf package.tgz -C ./package

### split分隔文件

把一个大文件分隔，特别适合于大文件，窄带宽传输成功率。

split -b 分隔单个文件大小(k,m) 被分隔的大文件 -a 序号位数(例如:-a 3，则分隔后fileaaa,filebbb) 分隔后单个文件名前缀

例如：

split -b 1024m 50g.tgz -a 3 50g

解释：-b 1024m 分隔为大小为1024m的单个文件，50.tgz为这个大文件名，-a 3分隔后文件后置序号为3位(例如：fileaaa,filebbb)，50g分隔后单个文件前缀

**恢复**

使用cat命令把多个文件，合成到一个文件。

cat 50gaa* > 50g.tgz



