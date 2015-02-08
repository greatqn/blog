---
layout: default
title: 用docker打造的开发环境
---

---

开发团队新组建，需要一套开发环境来支撑日常开发工作。以下带来的是我用单机，用docker打造的套装。用到的组件有seagull,mysql,redmine,jenkins,svn,lnmp,ftp。

单机配置：Ubuntu 14.04.1，8核，8G，2T。
/opt下分别建msyql,ftp,redmine,svn,jenkins目录，后面映射到docker的容器里。

step1:安装docker
sudo -s
apt-get update
curl -s https://get.docker.io/ubuntu/ | sudo sh

step2:安装海鸥，方便操作docker
git clone https://github.com/tobegit3hub/seagull.git
docker build -t tobegit3hub/seagull .

运行：
docker run -d -p 10086:10086 -v /var/run/docker.sock:/var/run/docker.sock tobegit3hub/seagull

访问：
http://host:10086 (host为主机的IP地址)

查看容器：
docker ps

启动容器：
docker start {cid}

进入容器：
docker exec -it {cid} /bin/bash

step3:安装mysql,redmine
https://github.com/sameersbn/docker-redmine

运行：
docker run --name=redminemysql -d \
  -e 'DB_NAME=redmine_production' -e 'DB_USER=redmine' -e 'DB_PASS=password' \
  -v /opt/redmine/mysql:/var/lib/mysql \
  sameersbn/mysql

docker run --name=redmine -it --rm -p 8081:80 --link redminemysql:mysql \
  -v /opt/redmine/data:/home/redmine/data \
  sameersbn/redmine

访问：
http://host:8081 
账号：admin 密码：admin

