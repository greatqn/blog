---
layout: default
title: 用docker打造的开发环境Ver2015
---

---

本文介绍的是一套内网开发环境，用于支撑50人以内的项目开发环境。单机用docker打造的一个套装。用到的组件有seagull,svn,mysql,redmine,jenkins,nexus,lnmp,ftp。 支持文档，源码，打包，测试，发布等环节。

单机配置：Ubuntu 14.04.1，8核，8G，2T。
/opt下分别建msyql,ftp,redmine,svn,jenkins目录，下文中会映射到docker的容器里。

###step1:安装docker
```
apt-get update
curl -s https://get.docker.io/ubuntu/ | sudo sh
```
###step2:安装海鸥，方便操作docker
```
git clone https://github.com/tobegit3hub/seagull.git
docker build -t tobegit3hub/seagull .
```
运行：

```
docker run -d -p 10086:10086 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  tobegit3hub/seagull
```

访问：

```
http://host:10086 (host为主机的IP地址)
```

查看容器：

```
docker ps -a
```
启动容器：

```
docker start {cid}
```

进入容器：

```
docker exec -it {cid} /bin/bash
```

###step3:安装mysql,redmine
```
https://github.com/sameersbn/docker-redmine
```

运行：

```
docker run --name=redminemysql -d \
  -e 'DB_NAME=redmine_production' -e 'DB_USER=redmine' -e 'DB_PASS=password' \
  -v /opt/redmine/mysql:/var/lib/mysql \
  sameersbn/mysql
```

```
docker run --name=redmine -it --rm -p 8081:80 \
  --link redminemysql:mysql \
  -v /opt/redmine/data:/home/redmine/data \
  sameersbn/redmine
```

访问：

```
http://host:8081 
账号：admin 密码：admin
```

###step3:安装svn
自己写的buildfile:

```
#
# svn server Dockerfile
#
 
FROM tutum/centos:6.5

MAINTAINER greatqn greatqn@163.com

RUN yum install subversion -y
RUN rm -rf /repos
RUN mkdir /repos 
RUN svnadmin create /repos
RUN echo "#!/bin/bash" >> /svn.sh &&\
    echo "svnserve -d -r /repos --listen-port 3391" >> /svn.sh &&\
    echo "/run.sh" >> /svn.sh  &&\
    chmod 777 /svn.sh

VOLUME /repos
EXPOSE 3391

CMD /svn.sh
```
编译：

```
docker build -t greatqn/svn .
```

运行：

```
docker run -d \
  --name svnserver \
  -p 3391:3391 \
  -v  /opt/svn/repos:/repos \
  greatqn/svn /svn.sh
```

访问：

```
svn://host:3391/
```

###step4:安装lnmp
```
https://github.com/tutumcloud/tutum-docker-lamp
docker push tutum/lamp
```

```
docker run --name="tutum_lamp" -d \
  -p 80:80 \
  tutum/lamp
```

```
docker run --name="wubin_lnmp" -d \
  -p 81:80 \
  wubin/lnmp_v1
```
内部环境：

```
nginx: master process /opt/nginx/sbin/nginx
php-fpm: master process (/opt/php/etc/php-fpm.conf)  
/opt/mysql/bin/mysqld --basedir=/opt/mysql --datadir=/opt/mysql/var

root            /opt/web/www;
set $host_dir   /opt/web/www;

/opt/nginx/conf/vhost/web.conf

/opt/php/etc/php.ini
```

###step5:安装jenkins
```
docker pull niaquinto/jenkins
```
运行：

```
docker run --name="jenkins" -d \
  -p 8099:8080 \
  -v /opt/jenkins:/var/lib/jenkins \
  niaquinto/jenkins
```
访问：

```
http://host:8099
```

###step6：安装nexus
```
docker run --name nexus -d \
  -p 8082:8081 \
  sonatype/nexus
```

访问：

```
http://host:8082
```

###step7:安装ftp
```
https://registry.hub.docker.com/u/penlook/ftp/
docker pull penlook/ftp
```

```
docker run --name="ftp" -d \
  -p 1021:21 -e USERNAME=username -e PASSWORD=password \
  -v /opt/ftp:/ftp \
  penlook/ftp 
```

只可以用主动模式连接。

```
ftp://host:1021
```

docker容器的实体目录

```
/var/lib/docker/aufs/mnt/{cid}
```

