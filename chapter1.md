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

另外记录一个很有意思的问题。部署CAdvisor监控时，如果在CentOS7上开启了user namespace，需要启动的时候添加 --privileged=true及--userns=host两个参数。但很可惜这两个参数不兼容swarm集群，导致在线上docker swarm集群中CAdvisor无法监控集群容器。

但奇葩的是，在部署过程中，线下集群里成功监控到了数据，导致我一度认为是官方文档有误，喜滋滋的跑去部署线上，果然，没数据。

翻了至少一百个网页和文档并且多方尝试之后得出结论docker swarm集群部署CAdvisor确实没有跟/var/run/docker.sock通信的权限，才会导致无法监控集群容器，官方文档诚不欺我。因此线上集群取不到数据是正常现象，线下能够取到数据反而不正常。（此处耗费20根头发）

那么就只好对比下线上和线下到底有什么差别了。

一通操作后得出结论：主机环境没有差别，docker版本号一致，镜像一致，系统相关配置一致。（此处想不通挠掉10根头发）

线上和线下主机上/var/run/docker.sock的权限都是srw-rw--- root docker，既然线上容器内没有权限访问，那对比下两处容器内的权限吧。哎哟，果然有新发现，线下容器docker.sock所属用户及群组是nobody bin，线上容器内所属用户群组则是nobody nobody。

再回忆下上面配置的docker用户映射，是把uid/gid为1000的用户映射到了容器内0的用户/群组。检查下容器内的群组编号，两个环境容器内群组bin编号都为1。好吧，问题终于解决了，显然是把host上gid为1001的群组映射到了容器内gid为1的bin群组，好巧不巧，线下gid为1001的群组恰好是docker群组，才导致容器内有连接权限。把线上的docker群组gid改为1001之后，果然线上也取到了数据。（此处太兴奋，甩掉了20根头发）

由于本司用户量级并不是很大，研发人员也不多，因此在一开始选型时就排除了相对较复杂且难维护的k8s集群，折中之下只好采用这种不太优雅的办法解决问题。期待docker官方能够早日提供更适当的解决方案。

另外，还需要给/var/docker/lib/1000.1000/image/overlay2/layerdb/mounts/路径增加所有用户的rx权限，才能取到docker容器的监控数据。

