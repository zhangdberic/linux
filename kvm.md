## linux kvm安装和配置

## 一、安装KVM

1、进入系统后，检查cpu参数是否支持虚拟化：

```
[root@localhost ~]# grep -Ei 'vmx|svm' /proc/cpuinfo
```

如果有出现vmx或者svm关键字就代表支持虚拟化，vmx代表Intel的CPU,svm代表AMD的CPU。

2、安装KVM

```
[root@localhost ~]# yum install -y  virt-*  libvirt  bridge-utils qemu-img
```

3、查看kvm模块支持确认载入kvm模块验证方法

```
[root@localhost ~]# lsmod | grep kvm
```

4、启动libvirtd服务

```
[root@localhost ~]# systemctl start libvirtd
```

5、验证kvm是否成功安装

```
virsh list
```

## 二、配置网卡

安装完KVM之后，需要配置一下网卡，增加一个桥接网卡(br0)：

1.编辑物理网卡信息(eth0)

vi /etc/sysconfig/network-scripts/ifcfg-eth0

注意：只需要在最后加入BRIDGE=br0

```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp3s0
UUID=57ca8932-ba1d-4fd7-9f60-65dd49db8479
DEVICE=enp3s0
ONBOOT=yes
BRIDGE=br0
```

2.编辑桥接网卡(br0)

vi /etc/sysconfig/network-scripts/ifcfg-br0

注意：本处才是设置ip地址的位置。

```
TYPE=Bridge
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.5.38
NETMASK=255.255.255.0
GATEWAY=192.168.5.1
NAME=br0
DEVICE=br0
```

3.重启网卡测试

```
systemctl restart network
```

## 三、virsh命令

```
virsh console xxx # 进入指定的虚拟机，进入的时候还需要按一下回车
virsh start xxx # 启动虚拟机
virsh shutdown xxx # 关闭虚拟机
virsh destroy xxx # 强制停止虚拟机
virsh undefine xxx # 彻底销毁虚拟机，会删除虚拟机配置文件，但不会删除虚拟磁盘
virsh autostart xxx # 设置宿主机开机时该虚拟机也开机
virsh autostart --disable xxx # 解除开机启动
virsh suspend xxx # 挂起虚拟机
virsh resume xxx # 恢复挂起的虚拟机
```

### install(安装)

```
virt-install --name=centos7 --memory=2048,maxmemory=10240 --vcpus=1,maxvcpus=6 --os-type=linux --os-variant=rhel7 --location=/vm/iso/CentOS-7-x86_64-Minimal-2003.iso --disk path=/vm/data/cetnos701.img,size=40 --bridge=br0 --graphics=none --console=pty,target_type=serial --extra-args="console=tty0 console=ttyS0"
```

--name 虚拟机名称

--memory=内存,maxmemory=最大使用内存

--vcpus=cpu个数,maxvcpus=最大允许cpu个数

--os-type=linux --os-variant=rhel7，操作系统类型

--location=/vm/iso/CentOS-7-x86_64-Minimal-2003.iso，使用的iso文件位置

--disk path=/vm/data/cetnos701.img,size=40 生成的虚拟机文件路径,size=文件大小(G)

--bridge=br0 使用的桥接网卡名

--graphics=none 非图像模式



### console(克隆)

```
virt-clone --connect qemu:///system --original 原虚拟机名 --name 新虚拟机名 --file /vm/data/新虚拟机名称01.img
```

例如：克隆centos7到middleware

```
virt-clone --connect qemu:///system --original centos7 --name middleware --file /vm/data/middleware01.img
```



### 修改虚拟机配置(edit、define)

virsh edit 虚拟机

例如：

```
virsh edit middleware
```

修改cpu，current='4'当前使用的cpu个数，<vcpu>最大允许cpu个数</vcpu>

```
<vcpu placement='static' current='4'>6</vcpu>
```

修改memory，<memory unit='KiB'>允许最大内存</memory>，  <currentMemory unit='KiB'>当前使用内存大小</currentMemory>

```
  <memory unit='KiB'>10485760</memory>
  <currentMemory unit='KiB'>8388608</currentMemory>
```

编辑后使用:x保存



编辑后生效

virsh define /etc/libvirt/qemu/虚拟机名称.xml

例如：

```
virsh define /etc/libvirt/qemu/middleware.xml
```



### 查看当前虚拟机配置(dominfo)

virsh dominfo 虚拟机名称

例如;

```
virsh dominfo middleware
```

显示结果：

```
名称：       middleware
UUID:           3064598f-b8f3-4463-8017-b01e11085c4f
OS 类型：    hvm
状态：       running
CPU：          1
CPU 时间：   37.4s
最大内存： 10485760 KiB
使用的内存： 2097152 KiB
持久：       是
自动启动： 禁用
管理的保存： 否
安全性模式： selinux
安全性 DOI： 0
安全性标签： system_u:system_r:svirt_t:s0:c317,c649 (enforcing)

```

