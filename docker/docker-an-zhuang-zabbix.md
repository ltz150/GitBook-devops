# 快速部署一个Kubernetes集群

> 参考： 1. [https://www.zabbix.com/documentation/3.4/zh/manual/installation/containers?s\[\]=docker](https://www.zabbix.com/documentation/3.4/zh/manual/installation/containers?s[]=docker)

部署一套完整的zabbix，需要安装数据库、web服务器、zabbix-server和zabbix-agent，这几样服务，过程都比较复杂，一不留心就可能出错，所以今天我使用docker容器搭建了一套zabbix服务，不仅搭建快，而且不易出错。

## 1.  在docker hub上有以下版本

```text
这里使用5.7版本，在linux终端拉取mysql5.7镜像

docker pull mysql:5.7
查看下载的镜像

docker image ls

[root@k8s-master docker]# docker images
REPOSITORY                                                        TAG                 IMAGE ID            CREATED         
mysql                                                             5.7                 5d9483f9a7b2        24 hours ago    
zabbix/zabbix-java-gateway                                        latest              660ab57f1078        2 weeks ago     
zabbix/zabbix-web-nginx-mysql                                     latest              2cc114aa7b9c        2 weeks ago     
zabbix/zabbix-server-mysql                                        latest              2bb0dd73957d        2 weeks ago
```

## 2.  下载zabbix-server镜像

zabbix-server镜像分两种，支持MySQL数据库zabbix-server-mysql，支持支持PostgreSQL数据库zabbix/zabbix-server-pgsql。下面安装的是支持MySQL数据库的Server镜像。

```text
打开zabbix-server-mysql的docker hub，大家会发现，zabbix-server-mysql有下面这些版本:

Zabbix server 3.0 (tags: alpine-3.0-latest, ubuntu-3.0-latest, centos-3.0-latest)
Zabbix server 3.0.* (tags: alpine-3.0.*, ubuntu-3.0.*, centos-3.0.*)
Zabbix server 3.2 (tags: alpine-3.2-latest, ubuntu-3.2-latest, centos-3.2-latest)
Zabbix server 3.2.* (tags: alpine-3.2.*, ubuntu-3.2.*, centos-3.2.*)
Zabbix server 3.4 (tags: alpine-3.4-latest, ubuntu-3.4-latest, centos-3.4-latest, alpine-latest, ubuntu-latest, centos-latest, latest)
Zabbix server 3.4.* (tags: alpine-3.4.*, ubuntu-3.4.*, centos-3.4.*)
Zabbix server 4.0 (tags: alpine-trunk, ubuntu-trunk)
```

因为我的服务器是centos7版本，所以选择的是centos-latest版本。还有后面如果想自己重做镜像的话，选择centos版本，用着会更简单一点。在linux终端使用

> docker pull zabbix/zabbix-server-mysql:centos-latest

## 3. 下载Zabbix web镜像

这里使用的是基于Nginx web服务器及支持MySQL数据库的Zabbix web接口zabbix/zabbix-web-nginx-mysql。这里是用的是latest版本，在linux终端使用

> docker pull zabbix/zabbix-web-nginx-mysql:latest
>
> ## 4.  下载zabbix-java-gateway镜像
>
> Zabbix本身不支持直接监控Java，而是使用zabbix-java-gateway监控jvm/tomcat性能。这里我们使用latest版本，在linux终端使用
>
> docker pull zabbix/zabbix-java-gateway:latest

## 5. Docker 网络

启动zabbix等镜像之前，需要先创建一个新的 Docker 网络。需要将后面的zabbix-server、mysql、web等容器都加入到此网络中，方便互相访问。在终端使用下面命令创建。

> docker network create -d bridge zabbix\_net

创建后，可以查看是否创建成功。

> docker network ls

## 6. 运行mysql 镜像，创建mysql容器

```text
docker run -dit -p 3306:3306 --name zabbix-mysql --network zabbix_net --restart always -v /etc/localtime:/etc/localtime -e MYSQL_DATABASE="zabbix" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="zabbix123" -e MYSQL_ROOT_PASSWORD="root123" mysql:5.7
# 改进
docker run -dit  --name zabbix-mysql -p 53306:3306 --network zabbix_net --restart always -v /etc/localtime:/etc/localtime:ro -v /home/docker/mysql/config/my.cnf:/etc/my.cnf -v /home/docker/mysql/data:/var/lib/mysql -e MYSQL_DATABASE="zabbix" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="zabbix123" -e MYSQL_ROOT_PASSWORD="root123" mysql:5.7

注释：

MYSQL_DATABASE="zabbix"         在msql中创建的数据库的名

MYSQL_USER="zabbix"              创建msql的登录账户名

MYSQL_PASSWORD="zabbix123"      设置创建msql的登录账户的密码

MYSQL_ROOT_PASSWORD="root123"   设置msql数据库root 的密码


其中-p 是将容器中的3306端口映射到服务器的3306端口，

--network zabbix_net是将容器加入到zabbix_net网络中，

-v /etc/localtime:/etc/localtime是同步服务器和容器内部的时区，

--restart always设置自启动，

-e MYSQL_DATABASE="zabbix"，创建环境变量。

--name zabbix-mysql，给容器命名。
```

## 7. 创建zabbix-java-gateway容器。

```text
docker run -v /etc/localtime:/etc/localtime -dit --restart=always --name=zabbix-java-gateway --network zabbix_net zabbix/zabbix-java-gateway:latest
```

## 8. 创建zabbix-server-mysql容器

首先创建数据卷zabbix-server-vol，通过命令

```text
docker volume create zabbix-server-vol
```

启动zabbix-server-mysql容器。

```text
 docker run -dit -p 10051:10051 --mount source=zabbix-server-vol,target=/etc/zabbix -v /etc/localtime:/etc/localtime -v /usr/lib/zabbix/alertscripts:/usr/lib/zabbix/alertscripts --name=zabbix-server-mysql --restart=always --network zabbix_net -e DB_SERVER_HOST="zabbix-mysql" -e MYSQL_DATABASE="zabbix" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="zabbix123" -e MYSQL_ROOT_PASSWORD="root123" -e ZBX_JAVAGATEWAY="zabbix-java-gateway" zabbix/zabbix-server-mysql:centos-latest


此处的以下内容与 运行mysql 镜像，创建mysql容器设置的内容要一致

MYSQL_DATABASE="zabbix" 

MYSQL_USER="zabbix"

MYSQL_PASSWORD="zabbix123"

MYSQL_ROOT_PASSWORD="root123"
```

## 9. 创建zabbix-web-nginx-mysql容器

```text
docker run -dit -p 80:80 -v /etc/localtime:/etc/localtime --name zabbix-web-nginx-mysql --restart=always --network zabbix_net -e DB_SERVER_HOST="zabbix-mysql" -e MYSQL_DATABASE="zabbix" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="zabbix123" -e MYSQL_ROOT_PASSWORD="root123" -e ZBX_SERVER_HOST="zabbix-server-mysql" zabbix/zabbix-web-nginx-mysql:latest

iwahle现场：
docker run -dit -p 50080:8080 -v /etc/localtime:/etc/localtime --name zabbix-web-nginx-mysql --restart=always --network zabbix_net -e DB_SERVER_HOST="zabbix-mysql" -e MYSQL_DATABASE="zabbix" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="zabbix123" -e MYSQL_ROOT_PASSWORD="root123" -e ZBX_SERVER_HOST="zabbix-server-mysql" zabbix/zabbix-web-nginx-mysql:centos-5.0-latest


此处的以下内容与 运行mysql 镜像，创建mysql容器设置的内容要一致

MYSQL_DATABASE="zabbix" 

MYSQL_USER="zabbix"

MYSQL_PASSWORD="zabbix123"

MYSQL_ROOT_PASSWORD="root123"
```

zabbix所需容器已经全部启动

> docker ps

## 10. 打开zabbix

在浏览器中输入[http://IP/zabbix，打开zabbix首页，其中用户名密码分别是admin/zabbix。](http://IP/zabbix，打开zabbix首页，其中用户名密码分别是admin/zabbix。)

![zabbix](https://github.com/ltz150/GitBook-devops/tree/5d5675d31129c771ccbf7568dc2f6236744b1c71/docker/B218D5441F8944979414776C91096705)

## 11. 安装zabbix客户端agent

```text
docker run --name zabbix-agent \
-e ZBX_HOSTNAME="self" \
-e ZBX_SERVER_HOST="192.168.254.13" \
-e ZBX_METADATA="client" \
-p 10050:10050 \
-d zabbix/zabbix-agent:latest
```

## 12. 配置zabbix yum源

```text
yum install https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm

# cat zabbix.repo
[zabbix]
name=Zabbix Official Repository - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-frontend]
name=Zabbix Official Repository frontend - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/frontend
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-debuginfo]
name=Zabbix Official Repository debuginfo - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/debuginfo/
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591
gpgcheck=1

[zabbix-non-supported]
name=Zabbix Official Repository non-supported - $basearch
baseurl=http://repo.zabbix.com/non-supported/rhel/7/$basearch/
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
gpgcheck=1
```

## 13. docker中zabbix图形界面乱码

解决方案:

* 在zabbix-web容器中/usr/share/zabbix/assets/fonts/备份原始字体文件
* 字体存在于Windows的路径C:\Windows\Fonts
* 拷贝字体到docker 容器内部

```text
docker cp /home/hduser/simkai.ttf zabbix-web:/usr/share/zabbix/assets/fonts/
```

