# docker方式安装gitlabs

1、搜索可用的Gitlab images
   docker search gitlab
2、选区一个image 并下载Gitlab
   sudo docker pull gitlab/gitlab-ce:latest
3、查看镜像文件
   docker images
4、创建Gitlab挂载目录
   在/docker/gitlab下分别创建config,logs,data目录
5、创建docker中的网络
   docker network create gitlab_net
6、使用image启动Gitlab容器, 启动前确认7001 没被使用 并且添加到iptables或者firewall里面(允许外部访问)
docker run --name='gitlabs' -d \
    --publish 1443:443 --publish 7001:80 --publish 7002:22 \
    --restart always \
    --volume /gitlab/config:/etc/gitlab \
    --volume /gitlab/logs:/var/log/gitlab \
    --volume /gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce

7、登录Gitlab 
   http://192.168.5.78:7001成功的话需要修改root账号的密码，随意设置即可。密码修改成功后，系统进入登录/注册页面

8.配置gitlab
  登入container： docker exec -it gitlab /bin/bash
  根据官方文档进行设置。编辑/etc/gitlab/gitlab.rb文件，这是Gitlab的全局配置文件
  官方文档：https://docs.gitlab.com/omnibus/settings/configuration.html
 

