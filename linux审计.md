# linux审计

参见文章：

https://blog.csdn.net/shaopeng1942/article/details/101703656

https://blog.csdn.net/u012759006/article/details/89670958

例如：

审计rm命令

auditctl -w /bin/rm -p x -k removefile

查看审计规则

auditctl -l

查看审计记录

ausearch -k removefile -i

例如：

审计目录

auditctl -w /root/test -p wxa -k root_test

查看审计规则

auditctl -l

查看审计记录

ausearch -k root_test -i