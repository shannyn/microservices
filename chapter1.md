# CentOS7.4中配置docker user namespace以及解决docker swarm集群监控问题

# CentOS7.4中配置docker user namespace

1，安装docker

yum install docker

2，以用户shannyn（uid：1000，gid：1000）为例配置docker user namespace，在/etc/docker/daemon.json中，输入下面内容：

{

```
"userns-remap": "shannyn"
```

}

3，需要把host的uid和gid跟docker容器内的uid、gid做一个对应：

echo "shannyn:1000:65536" &gt;&gt; /etc/subuid

echo "shannyn:1000:65536" &gt;&gt; /etc/subuid

意思是容器内id为0的用户，将会映射到host上id为1000的用户，后面65536个用户也会跟着做映射。

4，把shannyn用户加入到docker群组里：

sudo usermod -aG docker shannyn

4，CentOS7的kernel默认关闭了user namespace，需要重新打开

grubby --args="namespace.unpriv\_enable=1 user\_namespace.enable=1" --update-kernel="$\(grubby --default-kernel\)"

echo "user.max\_user\_namespaces=15076" &gt;&gt; /etc/sysctl.conf

reboot

5，启动docker

systemctl start docker

6，检查容器用户是否映射成功

## 解决CAdvisor在swarm中无法监控容器数据问题

docker swarm集群部署CAdvisor没有跟/var/run/docker.sock通信的权限，导致无法监控集群容器。解决方案：

1，在集群里的各台机器上对于需要访问/var/run/docker.sock的模块单独使用docker run加上--privileged参数启动。但这种方式实际生产环境操作起来比较麻烦，集群里的每台机器都要手动部署一堆模块，这些模块没有办法使用集群功能。

2, 使用socat转发/var/run/docker.sock，CAdvisor模块连接转发后的文件【推荐方案】

3, 记录一个巧合，不推荐使用。由于/var/run/docker.sock属于docker用户组，所以主机的docker gid刚好和容器内有root权限用户的gid映射上的时候，也可以访问到/var/run/docker.sock

