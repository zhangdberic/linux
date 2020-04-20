# nginx

## 1.nginx安装

### 1.1 nginx docker方式安装

nginx docker方式安装和配置，见“docker.md"文档。

### 1.2 nginx 编译安装



## 2.nginx.conf

**下面的配置说明层次结构同nginx.conf配置文件内容层次：**

### server

```nginx
	# 配置监听的端口
    listen       80;
```
```nginx
    # 服务器名,这里如果是localhost,则监听的地址为0.0.0.0，如果配置指定的ip，则监听是特定的ip地址。
    server_name  localhost;
```
```nginx
	# 默认字符集
	charset utf-8;
```



#### location

location 4种模式：

```nginx
    # location 第1种，URI前置匹配,如下：/abc、/abc?p1、/abc/xx/、/abcde 会被匹配到.
    location /abc {
    }
```
```nginx
    # location 第2种，URI精确匹配，如下：/abc、/abc?p1 会被匹配到.
    location = /abc {
    }
```
```nginx
    # location 第3种，正则表达式区分大小写，如下：/abc、/abc?p1=x 会被匹配到，/ABC不会。
    location ~ ^/abc$ {
    }
```
```nginx
    # location 第4种，正则表达式不区分大小写，如下:/abc、/abc?p1=x、/ABC 会被匹配到。
    location ~* ^/abc$ {
    }
```
##### 正则表达式匹配

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

###### 使用location正则表达式变量

```
location ~ ^/thumb/dfss/fss1/data/(.*)/(.*)/(.*)${
  set $day $1$2$3;
}
```

你可以根据$序号的形势，来获取location中匹配的字符，作为变量，例如：第1个括号内的内容通过$1变量引用，第2个括号内的内容通过$2变量引用，以此类推。

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

###### if 判断文件是否存和不存在

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

nginx.shell默认是不支持 or 运算符的，可以使用如下方法：

下面的例子：判断$size变量必须是400x400、300x300、200x200、100x100其中之一。

```nginx
            set $verify_flag 'f';
            set $size $6x$7;
            if ($size = '400x400'){
               set $verify_flag 't';
            }
            if ($size = '300x300'){
               set $verify_flag 't';
            }
            if ($size = '200x200'){
               set $verify_flag 't';
            }
            if ($size = '100x100'){
               set $verify_flag 't';
            }
            if ($verify_flag = 'f'){
               return 405;
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



# 好文档

https://blog.csdn.net/ok449a6x1i6qq0g660fv/article/details/80276506

