# device-mapper-multipath 存储挂载（多路径）

## 安装

```shell
yum -y install device-mapper
yum -y install device-mapper-multipath
```
## 获取wwid

```shell
multipath -v3 |grep sd.*36 # 获取所有的wwid(包括系统盘wwid)
```

例如：命令运行结果如下，包括系统盘wwid，这个系统盘的wwid下面文档要用。一般情况下系统盘多是服务器上的本机硬盘。

```
Dec 13 11:08:16 | sda: serial = 000911ef036234e3220b04465b980100
Dec 13 11:08:16 | sda: uid = 3660001985b46040b22e3346203ef1109 (callout) // 系统盘wwid
Dec 13 11:08:16 | sde: uid = 360060e801232c500504032c500000c0c (callout)
Dec 13 11:08:16 | sdc: uid = 360060e801232c500504032c500000c0a (callout)
Dec 13 11:08:16 | sdg: uid = 360060e801232c500504032c500000c0a (callout)
Dec 13 11:08:16 | sdi: uid = 360060e801232c500504032c500000c0c (callout)
Dec 13 11:08:16 | sdb: uid = 360060e801232c500504032c500000809 (callout)
Dec 13 11:08:16 | sdh: uid = 360060e801232c500504032c500000c0b (callout)
Dec 13 11:08:16 | sdf: uid = 360060e801232c500504032c500000809 (callout)
Dec 13 11:08:16 | sdd: uid = 360060e801232c500504032c500000c0b (callout)
```

uid就是wwid号，重复的wwid是配置了双HBA卡做了HA容错处理，其指向同一块存储块。



## 编写multipath.conf

vi /etc/multipath.conf，默认没有，这个文件需要手工创建，一般情况下只需要修改文件内容的wwid项即可(修改为上面系统盘的wwid)，其它的配置不需要改变。

注意：如果在下面启动mulitpart时，有可能其会自动在这个文件中生成最后multipaths配置项，则应该注释掉。

```shell
# multipath.conf written by anaconda

defaults {
	user_friendly_names yes
}
blacklist {
	devnode "^(ram|raw|loop|fd|md|dm-|sr|scd|st)[0-9]*"
	devnode "^hd[a-z]"
	devnode "^dcssblk[0-9]*"
	device {
		vendor "DGC"
		product "LUNZ"
	}
	device {
		vendor "IBM"
		product "S/390.*"
	}
	# don't count normal SATA devices as multipaths
	device {
		vendor  "ATA"
	}
	# don't count 3ware devices as multipaths
	device {
		vendor  "3ware"
	}
	device {
		vendor  "AMCC"
	}
	# nor highpoint devices
	device {
		vendor  "HPT"
	}
	wwid "3660001985b46040b22e3346203ef1109"   #将此处更改为本机的系统磁盘wwid
	device {
		vendor PIONEER
		product DVD-RW_DVR-XT11
	}
}
blacklist_exceptions {
}
#multipaths {
#	multipath {
#		uid 0
#		gid 0
#		wwid "3600b3423b87950cd28c7d4ac7d0000d1"
#		mode 0600
#	}
#	multipath {
#		uid 0
#		gid 0
#		wwid "3600b3424f91fbfed47abdaf8bd0000d1"
#		mode 0600
#	}
#}
```

## 编辑/etc/multipath/bindings

vi /etc/multipath/bindings，如果这个文件不存在，则手工创建，这个文件定义了磁盘名和wwid的绑定(映射)，例如，如下内容：

```shell
# Multipath bindings, Version : 1.0
# NOTE: this file is automatically maintained by the multipath program.
# You should not need to edit this file in normal circumstances.
#
# Format:
# alias wwid
#
mpatha 3660001985b46040b22e3346203ef1109
mpathb 360060e801232c500504032c500000c0a
mpathc 360060e801232c500504032c500000c0c
mpathd 360060e801232c500504032c500000c0b
mpathe 360060e801232c500504032c500000809
```

磁盘路径名可以自己定义，但建议使用mapthX的名称，例如：

mpatha 3660001985b46040b22e3346203ef1109   # 给系统盘wwid起名为mpatha

mpathb 360060e801232c500504032c500000c0a   # 给这个wwid顺序起名为mapthb

这里的wwid来源于上面章节 multipath -v3 |grep sd.*36 显示wwid，注意：因为是多路径我们要去掉重复的wwid。

## 编辑/etc/multipath/wwids

vi /etc/multipath/wwids，如果这个文件不存在，则手工创建，这个文件定义了有效的wwid。

例如，如下：

```shell
# Multipath wwids, Version : 1.0
# NOTE: This file is automatically maintained by multipath and multipathd.
# You should not need to edit this file in normal circumstances.
#
# Valid WWIDs:
/360060e801232c500504032c500000c0a/
/360060e801232c500504032c500000c0c/
/360060e801232c500504032c500000c0b/
/360060e801232c500504032c500000809/
```

根据上面/etc/multipath/bindings的配置来编辑这个文件，这个文件内包含了有效的wwid，和上面的bindings文件区别是去掉了"系统盘"的wwid，而且格式有变化，本文件的格式：/wwid/

## 启动multipath

service multipathd start

chkconfig multipathd on

## 查看multipath配置是否生效

fdisk -l |grep mpath

例如，命令结果：

```
Disk /dev/mapper/mpathc: 2199.0 GB, 2199023255552 bytes
Disk /dev/mapper/mpathb: 2199.0 GB, 2199023255552 bytes
Disk /dev/mapper/mpathe: 2638.8 GB, 2638827907072 bytes
Disk /dev/mapper/mpathd: 2199.0 GB, 2199023255552 bytes
```

## 查看WWN号

cat /sys/class/fc_host/host1/port_name

cat /sys/class/fc_host/host2/port_name

cat /sys/class/fc_host/host3/port_name

有几块HBA卡，就有几个WWN号，类似于网卡号。查看这个是方便和存储划分人员沟通。

## 增加磁盘

停止依赖于存储块的程序，例如：oracle数据库、web系统等。

备份重要的数据。

先运行如下命令，备份当前信息：

sudo multipath -ll

sudo multipath -v3 |grep sd.*36 # 获取所有的wwid(包括系统盘wwid)

sudo fdisk -l |grep '/dev/s'

sudo cat /etc/multipath/bindings

sudo cat /etc/multipath/wwids

sudo pvs

再执行如下命令，刷新磁盘（获取新挂载的磁盘）：

sudo ls -l /sys/class/scsi_host/host*

~~注意：有多少个host，每个都要执行，命令如下：~~

~~sudo sh -c 'sync && echo - - - > /sys/class/scsi_host/host0/scan'~~

~~sudo sh -c 'sync && echo - - - > /sys/class/scsi_host/host1/scan'~~

~~sudo sh -c 'sync && echo - - - > /sys/class/scsi_host/host2/scan'~~

~~上面执行后，就可以发现新的磁盘了。~~

使用sudo multipath -v3 |grep sd.*36命令，发现新的wwid号。

重新编辑：/etc/multipath/bindings，加入新的mpathX和wwid。

重新编辑：/etc/multipath/wwids，加入新的wwid。

验证：

sudo multipath -ll

























