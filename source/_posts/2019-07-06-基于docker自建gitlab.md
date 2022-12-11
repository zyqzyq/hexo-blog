---
title: 基于docker自建gitlab
urlname: cb2b07a79cf3b8365bfd77d6d6306e10
date: '2019-07-06 10:10:32 +0800'
layout: post
categories:
  - docker
tags:
  - gitlab
---

# gitlab_install

记录 gitlab 安装过程

# 安装 docker

官网地址：[安装 docker-ce centos 教程](https://docs.docker.com/install/linux/docker-ce/centos/)

### 卸载旧版本

```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

## 安装 Docker CE

### 添加存储库

```
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

```
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

_备注_：如果失败看要修改镜像地址为[https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo](https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo)

```
$ sudo yum-config-manager --enable docker-ce-edge
$ sudo yum-config-manager --enable docker-ce-test
```

### 安装 docker ce

```
$ sudo yum install docker-ce
$ sudo systemctl start docker
# 确认docker是否安装成功
$ sudo docker run hello-world
```

# 安装 gitlab centos docker 版

官网地址：[gitlab docker 安装](https:_docs.gitlab.com_omnibus_docker_readme)

```
sudo docker run --detach \
	--hostname gitlab.example.com \
	--publish 443:443 --publish 80:80 --publish 2222:22 \
	--name gitlab \
	--restart always \
	--volume /srv/gitlab/config:/etc/gitlab \
	--volume /srv/gitlab/logs:/var/log/gitlab \
	--volume /srv/gitlab/data:/var/opt/gitlab \
	gitlab/gitlab-ce:latest
```

_备注_：默认镜像下载较慢，可更换
编辑 /etc/docker/deamon.json,也可自行选择其他加速地址。

## 备份与恢复

官网地址：[gitlab 备份恢复](https:_docs.gitlab.com_ce_raketasks_backup_restore)

```
# 备份
sudo docker exec -t gitlab gitlab-rake gitlab:backup:create
# 恢复
sudo docker exec -it gitlab gitlab-rake gitlab:backup:restore
```

_备注_：gitlab 需与备份版本相同（当前版本 latest 11.6.3）
