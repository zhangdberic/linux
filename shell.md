# linux shell 

## /bin/bash

### shell 文件头

```bash
#!/bin/bash
```

下面的空行很有必要。

### 变量

#### 系统变量

$HOME 当前用户所在的家目录，例如：

```bash
nginx_home=$HOME/nginx
```

#### 自定义变量

```bash
#!/bin/bash

# 判断程序包是否存在
warfile="$HOME/ROOT.war"
echo $warfile 'does not exist.'
```

#### 变量使用

```bash
app_name="dy-config"
echo $app_name
echo ${app_name}_nohup.log
```

注意：如果变量在字符串中使用，最好使用${variable}。

### 显示输出

```bash
echo 'xxx'
```

### 判断文件是否存在

```bash
#!/bin/bash

if [ -f /home/tgms/ROOT.war ]; then
  echo 'exist.'
else
  echo 'does not exist.'
fi
```

### 判断文件不存在

```bash
#!/bin/bash

if [ ! -f /home/tgms/ROOT.war ]; then
  echo 'does not exist.'
fi
```

```bash
#!/bin/bash

# 判断程序包是否存在
warfile="$HOME/ROOT.war"
if [ ! -f $warfile ]; then
  echo $warfile 'does not exist.'
  exit 0
fi
```

### 判断字符串为空

```bash
if [ -z "$pid" ]
then
  echo 'empty'
fi
```

### 判断字符串不为空

```bash
if [ -n "$pid" ]
then
  echo 'not empty'
fi
```

### 退出

```bash
exit 0
```

### 休眠

```bash
sleep 5s
```

### 获取某个进程的pid

```shell
ps aux|grep 'java -jar dy-config-1.0.1.jar'|grep -v "grep"|awk '{print $2}'
```

第1个grep后的字符应该是进程唯一标识，第二个grep -v "grep"很重要，其起到去掉pts进程的作用，你可以单独执行以下：ps aux|grep 'java -jar dy-config-1.0.1.jar'，看看返回几行。

### kill某个进程

```bash
pid=$(ps aux|grep 'java -jar dy-config-1.0.1.jar'|grep -v "grep"|awk '{print $2}')
if [ -n "$pid" ]
then
    kill -9 $pid
    echo 'kill -9' $pid
fi
echo 'shutdown ok.'
```

注意：你如果把上面的shell定义为一个sh文件，这个sh文件名一定不要和grep 'xxxx'内的字符匹配上，否则会出现多个pid，因为如果你执行这个sh，这个sh的进程名就是sh文件名。例如：grep 'dy-config'，而你的sh文件名为dy-config-shutdown.sh，那么会出现上面的问题。对于java进程可以使用如下策略，用java字样+程序标识组合为唯一标识，这个策略更好，例如：

```bash
pid=$(ps aux|grep 'java'|grep 'dy-config'|grep -v "grep"|awk '{print $2}')
if [ -n "$pid" ]
then
    kill -9 $pid
    echo 'kill -9' $pid
fi
echo 'shutdown ok.'
```



### 判断nginx进程是否启动

```bash
#!/bin/bash

nginx_home=$HOME/nginx
if [ -f $nginx_home/logs/nginx.pid ]; then
	$nginx_home/sbin/nginx -s stop
fi
```

### 防篡改

#### 生成样板文件

每次代码修改都要重新生成样板文件，例如：你可以在jenkins发布程序后，重新生成样板。

```bash
find /home/ygjzxfw/tomcat8/webapps ! -name '*.log*' -type f -print0 | xargs -0 md5sum|sort -k 2 > /root/monitor/ygjzxfw_sample.md5
```

#### 验证文件是否篡改

每隔10分钟允许一次，重新生成为项目的每个文件生成md5和样板文件比较，如果不一样则说被篡改了。

```bash
#!/bin/sh

find /home/ygjzxfw/tomcat8/webapps ! -name '*.log*' -type f -print0 | xargs -0 md5sum|sort -k 2 > /root/monitor/ygjzxfw.md5

diff /root/monitor/ygjzxfw_sample.md5 /root/monitor/ygjzxfw.md5
if [[ $? = 0 ]];then
    echo "ygjzxfw not modified."
else
    diff /root/monitor/ygjzxfw_sample.md5 /root/monitor/ygjzxfw.md5 > /root/monitor/ygjzxfw_md5_diff_result.txt
    cat /root/monitor/ygjzxfw_md5_diff_result.txt | mail -s 在线查询被篡改 909933699@qq.com
fi
```

