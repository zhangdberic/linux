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

### 退出

```bash
exit 0
```

### 休眠

```bash
sleep 5s
```

### kill某个进程

```bash
#!/bin/bash

PID=$(ps -ef | grep '/home/tgms/jdk1.8/jre/bin/java' | grep -v grep | awk '{ print $2 }')
if [ -z "$PID" ]
then
    echo 'Application is already stopped'
else
    echo 'kill' $PID
fi

```

grep 后为这个进程信息的唯一标识，也就是查找进程的条件，这个必须唯一，你可以通过先执行：ps -ef | grep 'xxx'，看返回的行数，必须是1行。

### 判断nginx进程是否启动

```bash
#!/bin/bash

nginx_home=$HOME/nginx
if [ -f $nginx_home/logs/nginx.pid ]; then
	$nginx_home/sbin/nginx -s stop
fi
```

