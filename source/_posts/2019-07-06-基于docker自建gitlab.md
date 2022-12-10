---
title: 2019-07-06-基于docker自建gitlab
urlname: cb2b07a79cf3b8365bfd77d6d6306e10
date: '2022-12-09 20:02:33 +0800'
tags: []
categories: []
---

<hr />
<p>author: zyqzyq

date: 2019-07-06 10:10:32+00:00

layout: post

title: 基于 docker 自建 gitlab

categories: docker

key: 20190706

tags:</p>

<ul>
<li>gitlab</li>
</ul>
<hr />
<h1>gitlab_install</h1>
<p>记录gitlab安装过程</p>
<h1>安装docker</h1>
<p>官网地址：<a href="https://docs.docker.com/install/linux/docker-ce/centos/" target="_blank">安装docker-ce centos教程</a></p>
<h3>卸载旧版本</h3>
<pre><code>$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
</code></pre>
<h2>安装Docker CE</h2>
<h3>添加存储库</h3>
<pre><code>$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
</code></pre>
<pre><code>$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
</code></pre>
<p><em>备注</em>：如果失败看要修改镜像地址为https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo</p>
<pre><code>$ sudo yum-config-manager --enable docker-ce-edge
$ sudo yum-config-manager --enable docker-ce-test
</code></pre>
<h3>安装docker ce</h3>
<pre><code>$ sudo yum install docker-ce
$ sudo systemctl start docker
# 确认docker是否安装成功
$ sudo docker run hello-world
</code></pre>
<h1>安装gitlab centos docker版</h1>
<p>官网地址：<a href="https:_docs.gitlab.com_omnibus_docker_readme" target="_blank">gitlab docker安装</a></p>
<pre><code>sudo docker run --detach \
	--hostname gitlab.example.com \
	--publish 443:443 --publish 80:80 --publish 2222:22 \
	--name gitlab \
	--restart always \
	--volume /srv/gitlab/config:/etc/gitlab \
	--volume /srv/gitlab/logs:/var/log/gitlab \
	--volume /srv/gitlab/data:/var/opt/gitlab \
	gitlab/gitlab-ce:latest

</code></pre>

<p><em>备注</em>：默认镜像下载较慢，可更换</p>
<p>编辑 /etc/docker/deamon.json,也可自行选择其他加速地址。</p>
<h2>备份与恢复</h2>
<p>官网地址：<a href="https:_docs.gitlab.com_ce_raketasks_backup_restore" target="_blank">gitlab备份恢复</a></p>
<pre><code># 备份
sudo docker exec -t gitlab gitlab-rake gitlab:backup:create
# 恢复
sudo docker exec -it gitlab gitlab-rake gitlab:backup:restore
</code></pre>
<p><em>备注</em>：gitlab需与备份版本相同（当前版本latest 11.6.3）</p>
