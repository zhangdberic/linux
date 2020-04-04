# 1.安装Hyper-V

## 1.2 windows10安装Hyper-V

安装Hyper-V管理器
控制面板->程序和功能->启动和关闭wndows功能->选中Hyper-V->确定->重新启动系统

## 1.2 windows7 安装Hyper-V

下载地址：

https://www.microsoft.com/en-us/download/details.aspx?displaylang=en&id=7887

安装：

先安装这个sp1，安装后重启，控制面板->程序和功能->远程[服务器管理](https://www.baidu.com/s?wd=服务器管理&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)工具->Hyper-V选中->安装后不用重启，额可以直接使用。

# 2.Hyper-V 操作

## 2.1 Hyper-V创建虚拟交换机

启动Hyper-V管理器

创建一个虚拟交换机：Hyper-V管理器->虚拟交换机管理器->外部->创建虚拟交换机(S)->确定

修改为固定ip地址(非dhcp)：修改这个新创建“虚拟交换机”->internet协议版本4(TCP/IPv4)->属性->(ip地址192.168.0.253、掩码255.255.255.0、默认网关192.168.0.1、DNS 114.114.114.114)->确定

注意：这里设置的ip地址，是根据路由器来设置的，例如：tplink的路由器，登录管理界面，查看dhcp分配ip的范围。

# 2.2 安装linux虚拟机

下载centos7,到官网下载centos7的mini x86_64版本iso文件

创建linux虚拟机
Hyper-V管理器->新建->虚拟机(M)->...(按照向导操作就可以了)->确定
禁止启用安全：选择对应的虚拟机->安全->去掉“启用安全启动(E)”->确定
设置固定MAC地址：选择对应的虚拟机->网络适配器->高级功能->MAC地址(静态)->确定。记住这个MAC地址，下面要用。

安装linux
Hyper-V管理器->选择"创建的虚拟机"->右键->启动->...正常安装linux系统就可以了.

设置linux系统网络
vi /etc/sysconfig/network-scripts/ifcfg-eth0 
TYPE=Ethernet
BOOTPROTO=static
IPADDR=192.168.0.250
NETMASK=255.255.255.0
DEVICE=eth0
ONBOOT=yes
HWADDR=00:15:5D:00:08:02
GATEWAY=192.168.0.1

设置dns

vi /etc/resolv.conf

nameserver 8.8.8.8

nameserver 114.114.114.114

重新启动网络
service network restart

## 2.3 测试宿主机和虚拟机的网络连接

两台机器相互ping
如果linux虚拟机无法ping通宿主机(win10)，有可能win10禁ping了(默认是禁止的)。
开通ping(ICMP)：控制面板->Windows Defender防火墙->高级设置->入站规则->"文件和打印机共享(回显请求 - ICMPv4-ln)，注意是两个,两个是同样的字样，选中两个后，点击启用规则。

## 2.4 增加硬盘

**Hyper为虚拟机增加硬盘**

选择要增加虚拟机点击”设置“，选择”SCSI控制器"，选择“硬盘控制器"，点击”添加“按键，新建，按照向导操作。

添加的硬盘，命名为**虚拟机名称X.vhdx**。

**linux虚拟机挂载硬盘**

无效重新启动，在线挂载新硬盘。

fdisk -l 可以查看到刚挂载的硬盘，例如：下面的/dev/sdb

```
磁盘 /dev/sdb：32.2 GB, 32212254720 字节，62914560 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 4096 字节
I/O 大小(最小/最佳)：4096 字节 / 4096 字节
```

**创建物理卷**

pvcreate /dev/sdb

**把新创建的物理卷加入卷组**

vgextend centos /dev/sdb

**查看卷组的大小是否有变化**

vgdisplay

**扩展逻辑卷**

lvextend -L 46G /dev/centos/root

```
  Size of logical volume centos/root changed from 16.80 GiB (4301 extents) to 46.00 GiB (11776 extents).
  Logical volume centos/root successfully resized.
```

**重置逻辑卷**

xfs_growfs /dev/vg_bak/lv_bak

**查看文件系统**

df -h

```
文件系统                 容量  已用  可用 已用% 挂载点
devtmpfs                 1.9G     0  1.9G    0% /dev
tmpfs                    1.9G     0  1.9G    0% /dev/shm
tmpfs                    1.9G  8.6M  1.9G    1% /run
tmpfs                    1.9G     0  1.9G    0% /sys/fs/cgroup
/dev/mapper/centos-root   46G   16G   31G   33% /
/dev/sda2               1014M  141M  874M   14% /boot
/dev/sda1                200M   12M  189M    6% /boot/efi
tmpfs                    379M     0  379M    0% /run/user/0

```















