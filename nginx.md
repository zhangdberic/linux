# nginx

## 1.nginx安装

### 1.1 nginx安装

useradd nginx

su - nginx

wget http://nginx.org/download/nginx-1.18.0.tar.gz

wget https://www.openssl.org/source/openssl-1.1.1g.tar.gz

tar xvf nginx-1.18.0.tar.gz

tar xvf openssl-1.1.1g.tar.gz

cd nginx-1.18.0

./configure --prefix=/home/nginx/nginx  --user=nginx --group=nginx --with-http_ssl_module --with-openssl=/home/nginx/openssl-1.1.1g/

make -j4

make install

### 1.2 80端口授权

su - root

chown root /home/nginx/nginx/sbin/nginx

chmod u+s /home/nginx/nginx/sbin/nginx

firewall-cmd --permanent --add-port=80/tcp

firewall-cmd --reload

su - nginx

~/nginx/sbin/nginx -t

## 2.nginx.conf

### 常用的nginx.conf配置

cp ~/nginx/conf/nginx.conf ~/nginx/conf/nginx.conf.bak

touch /home/nginx/nginx/conf/allowip.conf; 

touch /home/nginx/nginx/conf/denyip.conf; 

/bin/vi ~/nginx/conf/nginx.conf

```nginx
user nginx nginx;
worker_processes  2;
error_log  logs/error.log  crit;
worker_rlimit_nofile 60000;
events {
    use  epoll;
    worker_connections  60000;
}
http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format main '{"req_user_addr":"$remote_addr"'
    ',"req_user_real_addr":"$http_x_forwarded_for"'
    ',"req_time":"$time_local"'
    ',"req_scheme":"$scheme"'
    ',"req_protocol":"$server_protocol"'
    ',"req_method":"$request_method"'
    ',"req_url":"$request_uri"'
    ',"req_length":"$request_length"'
    ',"req_content_length":"$content_length"'
    ',"req_content_type":"$content_type"'
    ',"req_user_agent":"$http_user_agent"'
    ',"req_referer":"$http_referer"'
    ',"req_connection":"$http_connection"'
    ',"req_worker_pid":"$pid"'
    ',"req_tcp_id":"$connection"'
    ',"req_tcp_num":"$connection_requests"'
    ',"resp_status":"$status"'
    ',"resp_bytes":"$bytes_sent"'
    ',"resp_content_length":"$sent_http_content_length"'
    ',"resp_content_type":"$sent_http_content_type"'
    ',"resp_time":"$request_time"'
    ',"upstream_addr":"$upstream_addr"'
    ',"upstream_status":"$upstream_status"'
    ',"upstream_resp_time":"$upstream_response_time"'
    ',"upstream_resp_length":"$upstream_response_length"'
    '}';

    access_log  logs/access.log  main  buffer=8k;

    sendfile        on;
    server_tokens off;

    keepalive_timeout  5 5;
    client_header_timeout 5s;
    client_body_timeout 10s;
    send_timeout 10s;

    client_body_buffer_size 16K;
    client_max_body_size 16k;
    client_body_temp_path /home/nginx/nginx/client_body_temp 1 2;

   
    server {
        listen       80;
        server_name	location;
        charset utf-8;

        # 静态网页		
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        # 反向代理     
        location /services {
             proxy_pass http://services;
             proxy_redirect off;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_http_version 1.1;
             proxy_set_header Connection "";
             client_body_buffer_size 128k;
             client_max_body_size 5m;
             proxy_connect_timeout 60s;
             proxy_send_timeout 60s;
             proxy_read_timeout 60s;
        }
    } 
    
    upstream services {
        server 127.0.0.1:8080;
        keepalive 50;
    }

}

```

**验证配置**

~/nginx/sbin/nginx -t

**启动nginx**

~/nginx/sbin/nginx

### server

```nginx
	# 配置监听的端口
    listen       80;
```
```nginx
	# 服务名
    # nginx会取出请求头host与nginx.conf中每个server的server_name进行匹配(支持正则表达式匹配,例如：*.lnyg.net)，以此决定到底由哪一个server块来处理这个请求，如果没有匹配的上，默认的server为第一个server。
    server_name  localhost;
```
```nginx
	# 默认字符集
	charset utf-8;
```

#### location

##### 正则表达式匹配

```
~ 区分大小写匹配

~* 不区分大小写匹配

!~和!~*分别为区分大小写不匹配及不区分大小写不匹配

^ 以什么开头的匹配

$ 以什么结尾的匹配

转义字符。可以转. * ?等

* 代表任意字符
```

**文件和目录匹配**

```
-f和!-f用来判断是否存在文件

-d和!-d用来判断是否存在目录

-e和!-e用来判断是否存在文件或目录

-x和!-x用来判断文件是否可执行
```

##### 例子

举一个复杂的例子，来看看：

```nginx
location ~ ^/thumb/dfss/fss1/data/(.*)/(.*)/(.*)/([0-2])(.*)_(\d+)x(\d+)\.(jpg|png|bmp|gif)$ {
}
```

请求URI：http://192.168.5.78/thumb/dfss/fss1/data/2020/04/14/2vfXheLz2qA0W0mhsMNXe3632233996.jpg_300x300.jpg

根据location配置我们来剖析这个URI：

1.前缀必须是/thumb/dfss/fss1/data开通的URI；

2.前三个(.*)匹配，/thumb/dfss/fss1/data后紧跟着的三个xxxx/xx/xx，加括号的目的，是为可以再nginx.conf中通过变量序号引用括号内的内容，例如：$1，$2，来获取括号内的内容。对照上面请求URI，这里$1=2020，$2=04，$3=14 ;

3.([0-2])匹配第1个字符必须是0,1,2三个数字字符。对照上面请求URI，这里$4=2。紧跟着的(.*)为$5=vfXheLz2qA0W0mhsMNXe3632233996.jpg；

4.(\d+)x(\d+)，匹配300x300部分(\d+,必须为数字字符)，$6=300，$7=300。

5.如下，.\ 因为.是表单式匹配的特殊字符，如果要使用.字符需要在后面加\符号。(jpg|png|bmp|gif)，表示必须是这个4个字符串中的其中一个，最后$表示匹配结束。

```
.\(jpg|png|bmp|gif)$
```

##### nginx shell

###### 正则表达式变量

```nginx
location ~ ^/thumb/dfss/fss1/data/(.*)/(.*)/(.*)$ {
  set $year $1;
  set $month $2;
  set $date $3;  
}
```

正则表达式，括号部分声明了要获取的变量位置，你可以根据$序号的形式，来获取正则中括号的字符串，作为变量，例如：第1个括号内的内容通过$1变量引用，第2个括号内的内容通过$2变量引用，以此类推。

**括号**除了有定义序号变量的作用，还有就是提高优先级，例如，或者操作(jpg|png|bmp|gif)。

**注意：**

1.正则表达式不但可以在location中使用，还可以在location{}块中的任何位置使用，例如if判断语句、rewrite语句等。

2.序号变量$x，序号变量是上一个正则表达式的序列变量，这个必须牢记。如果再有第2个正则表达式，其会覆盖上面的正则表达式的序号变量。

例如：第1个正则表达式是location后的URI匹配表达式，其可以使用4个序列变量。第2个正则表达式是if ( $filename ~ ^(.*)\.del\..*$ )，其执行后会覆盖location后的URI匹配表达式的序列变量，如果你要保留第1个location后的URI匹配表达式，则应先使用临时的变量，保存起来，例如：set $fssx $1;

```nginx
    location ~ ^/thumb/dfss/(.*)/data/.*/.*/.*/2(.*)\.(.*)_\d+x\d+\.(jpg|png|bmp|gif)$ {
            set $fssx $1;
            set $filename $2;
            set $file_ext $3;
            set $thumb_img_ext $4;

            set $allow_fssx 'f';
            if ( $fssx = 'fss1' ) {
               set $allow_fssx 't';
            }
            if ( $allow_fssx != 't' ) {
               return 405 'fssx [$1] error';
            }
            if ( $filename ~ ^(.*)\.del\..*$ ) {
               return 405 'uri include error char, $1 ';
            }
			
    	  # 序号变量已结被第2个if判断的正则表达式覆盖了,下面的序号变量都是空格
          #return 200 'ok $1 file $2 ,ext $3 ,thumb ext $4 ';
          # 使用保存的临时变量
          #return 200 'ok $fssx file $filename ,ext $file_ext ,thumb ext $thumb_img_ext';
}
```



一定要在代码开始的位置，尽量靠前的位置，把序号变量赋值(set)给普通变量，否则在下面序号变量可能丢失。

###### set 赋值变量

```nginx
set $purview '1';
```

复杂一点的：

```nginx
set $thumb_image_path  /home/fss/fssdata/thumb/data/$1/$2/$3/$4$5_$6x$7.$8;
```

###### if 判断

注意：nginx shell不支持else判断。

```nginx
if ($purview = '2') {
    set $allow_access 't';
}
```

###### if判断相等或不等

```
=:等值比较;
~：与指定正则表达式模式匹配时返回“真”，判断匹配与否时区分字符大小写；
~*：与指定正则表达式模式匹配时返回“真”，判断匹配与否时不区分字符大小写；
!~：与指定正则表达式模式不匹配时返回“真”，判断匹配与否时区分字符大小写；
!~*：与指定正则表达式模式不匹配时返回“真”，判断匹配与否时不区分字符大小写；
```

###### if 判断文件是否存和不存在

```
-f, !-f：判断指定的路径是否为存在且为文件；
-d, !-d：判断指定的路径是否为存在且为目录；
-e, !-e：判断指定的路径是否存在，文件或目录均可；
-x, !-x：判断指定路径的文件是否存在且可执行；
```

判断文件存在

```nginx
set $thumb_image_path '/home/fss/fss1.jpg';
if ( -f $thumb_image_path ){
}
```

判断危机不存在

```nginx
set $thumb_image_path '/home/fss/fss1.jpg';
if ( !-f $thumb_image_path ){
}
```

###### if or 判断

```nginx
if ($remote_addr = "(59.197.168.224|59.197.168.225)") {
    set $allow_access 't'
}
```

```nginx
if ($http_host !~ "(app.lnyg.net|service.lnyg.net)") {
    return 403;
}
```

如下是更笨的方法：

下面的例子：判断$size变量必须是400x400、300x300、200x200、100x100其中之一。

```nginx
            set $verify_flag 'f';
            set $size $6x$7;
            if ( $size = '400x400' ) {
               set $verify_flag 't';
            }
            if ( $size = '300x300' ) {
               set $verify_flag 't';
            }
            if ( $size = '200x200' ) {
               set $verify_flag 't';
            }
            if ( $size = '100x100' ) {
               set $verify_flag 't';
            }
            if ( $verify_flag = 'f' ) {
               return 405 'size [$size] error.';
            }
```

##### root、alias

alias是一个目录别名的定义，root则是最上层目录的定义。

**例如：root方式**

```nginx
location /img/ {
    root /var/www/image;
}
```

请求URI，http://localhost/img/1.jpg，nginx查找目录，/var/www/image/img/1.jpg，解释为：请求匹配的URI路径为root的子目录。

请求：/img/xxx/yyy，查找路径：/var/www/image**/img/xxx/yyy**

**例如：alias**

```nginx
location /img/ {
    alias /var/www/image/;
}
```

请求URI，http://localhost/img/1.jpg，nginx查找目录，/var/www/image/1.jpg，解释为：请求匹配的URI路径为alias路径。

请求：/img/xxx/yyy，查找路径：/var/www/image**/xxx/yyy**

**alias特殊用处，URI完全匹配映射，实现请求URI到本地文件的映射**

例如：http://localhost/img/a/b/c/xxx.jpg

```
location /img/a/b/c/xxx.jpg {
    alias /var/www/image/xxx.jpg;
}
```

##### proxy_store(代理内容存储)

把proxy_pass请求返回的内容存储到磁盘上，例如：

把产生的缩率图，存储到磁盘上。proxy_store指定了存储内容的文件位置，proxy_store_access指定了存储文件的访问权限。

```nginx
proxy_pass http://127.0.0.1:$server_port/$internal_thumb_url;
# 下面为缩率图存储
proxy_store $thumb_image_path;
proxy_store_access user:rw group:r all:r;
```

##### allow和deny

allow允许访问的ip和ip段，deny禁止访问的ip和ip段，例如：只允许本机访问。

```nginx
            allow 127.0.0.0/8;
            deny all;
```

##### rewrite

如下：对这台主机/zsm的请求都会转发到192.168.5.78:3001端口上。

```nginx
        location /zsm {
           rewrite ^/zsm/(.*) http://192.168.5.78:3001/$1 permanent;
        }
```

## 3.nginx脚本

### 日志每天切换

vi /home/nginx/nginx/sbin/cut-log.sh

```bash
#!/bin/bash
## 零点执行该脚本
## Nginx 日志文件所在的目录
LOGS_PATH=/home/nginx/nginx/logs
## 获取昨天的 yyyy-MM-dd
YESTERDAY=$(date -d "yesterday" +%Y-%m-%d)
## 移动文件
mv ${LOGS_PATH}/access.log ${LOGS_PATH}/access_${YESTERDAY}.log
## 向 Nginx 主进程发送 USR1 信号。USR1 信号是重新打开日志文件
kill -USR1 $(cat /home/nginx/nginx/logs/nginx.pid)
```

crontab -e

```
0 1 * * * /home/nginx/nginx/sbin/cut-log.sh
```

## 4.安全加固

### nginx.conf

```nginx
	server {

       # ip访问规则,规则越靠前优先级别越高(只有allow ${ip};和deny all;两个成对出现才有意义)	    
       allow 10.60.9.0/24;
       deny all;

        # 限制域名访问,多个域名用空格分隔
        server_name  www.domain.com auth2.domain.com;
		# 请求头user_agent以下值拒绝
		if ($http_user_agent ~* "python|perl|ruby|curl|bash|echo|uname|base64|decode|md5sum|select|concat|httprequest|nmap|scan|spider" ) {
            return 403;
        }
		
		# 请求头host(指定范围)否则拒绝
		if ($http_host !~ "(www.domain.com|auth2.domain.com)") {
            return 403;
        }
		
		# 如下url请求拒绝
        location /(admin|phpadmin|status)  { deny all; }

        #拒绝所有以以下结尾的请求
        location ~* \.(php|asp|ASP|aspx|hp|ashx|zip|rar|gz|sql|tgz|dat|txt|mdb|properties|log|class)$ {
            deny all;
        }
	}

```



## 5.nginx正向代理

iptables可以做到基于ip地址的正向代理，但如果目标端是域名这个可以使用nginx的正向代理，其基于stream代理实现。

编辑加入:

```
 --with-stream
```

配置:

```nginx
stream {
    upstream backend {
       server www.dongyuit.cn:80;
       #server www.dongyuit.cn:81;
    }

    server {
        listen 12345;
        resolver 8.8.8.8:53 valid=30s;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass backend;
    }

}

```

这样当你访问nginx这台机器的12345端口，就是访问www.dongyuit.cn:80。

## 7.FAQ

允许nginx命令卡住不动

反向代理配置错误，检查nginx.conf的反向代理配置，重点查看proxy_pass和upstream的配置是否正确。



# 好文档

https://blog.csdn.net/ok449a6x1i6qq0g660fv/article/details/80276506

