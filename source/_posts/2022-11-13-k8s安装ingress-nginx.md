---
title: 2022-11-13-k8s安装ingress-nginx
urlname: 545db93f4db4d7ab858dc6064788f79f
date: '2022-12-09 20:02:34 +0800'
tags: []
categories: []
---

<hr />
<p>author: zyqzyq

date: 2022-11-13 10:10:32+00:00

layout: post

title: k8s 安装 ingress-nginx

categories: k8s

key: 20221113

tags:</p>

<ul>
<li>k8s</li>
</ul>
<hr />
<h1>安装ingress nginx</h1>
<p>官网地址：<a href="https:_github.com_kubernetes_ingress-nginx_blob_nginx-0.30.0_docs_deploy_index" target="_blank">安装ingress-nginx教程</a></p>
<p>k8s版本:1.18</p>
<p>ingress 版本: 0.30</p>
<p>因 ingress1.0以上版本至少需要k8s 1.19以上版本</p>
<pre><code>kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml
</code></pre>
<p>有需要可以自己下载部署文件修改以后手动部署</p>
<h3>提供服务（自建服务器）</h3>
<p>官网地址：<a href="https:_github.com_kubernetes_ingress-nginx_blob_nginx-0.30.0_docs_deploy_baremetal" target="_blank">裸机提供服务方法</a></p>
<h5>基于nodeport</h5>
<p>注意namespace与ingress-nginx保持一致</p>
<pre><code>kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/provider/baremetal/service-nodeport.yaml
</code></pre>
<p>nodePort的部署思路就是通过在每个节点上开辟nodePort的端口，将流量引入进来，而后通过iptables首先转发到ingress-controller容器中，而后由nginx根据ingress的规则进行判断，将其转发到对应的应用web容器中</p>
<h5>基于 hostNetwork</h5>
<p>可以修改之前部署的pod文件添加以下配置</p>
<pre><code>template:
  spec:
    hostNetwork: true
</code></pre>
<p>hostNetwork模式不再需要创建一个nodePort的svc，而是直接在每个节点都创建一个ingress-controller的容器，而且将该容器的网络模式设为hostNetwork。也就是说每个节点物理机的80和443端口将会被ingress-controller中的nginx容器占用。当流量通过80/443端口进入时，将直接进入到nginx中。而后nginx根据ingress规则再将流量转发到对应的web应用容器中。</p>
<h4>两种部署方式的比较</h4>
<p>相比较起来，nodePort部署模式中需要部署的ingress-controller容器较少。一个集群可以部署几个就可以了。而hostNetwork模式需要在每个节点部署一个ingress-controller容器，因此总起来消耗资源较多。另外一个比较直观的区别，nodePort模式主要占用的是svc的nodePort端口。而hostNetwork则需要占用物理机的80和443端口。</p>
<p>从网络流转来说，通过nodePort访问时，该node节点不一定部署了ingress-controller容器。因此还需要iptables将其转发到部署有ingress-controller的节点上去，多了一层流转。</p>
<p>另外，通过nodePort访问时，nginx接收到的http请求中的source ip将会被转换为接受该请求的node节点的ip，而非真正的client端ip。</p>
<p>而使用hostNetwork的方式，ingress-controller将会使用的是物理机的DNS域名解析(即物理机的/etc/resolv.conf)。而无法使用内部的比如coredns的域名解析。</p>
<p>因此具体使用哪种部署方式，需要根据实际情况和需求进行选择。</p>
<h3>编写ingress</h3>
<pre><code>apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
spec:
  rules:
  - host: www.test.com
    http:
      paths:
      - path: /
        backend:
          service:
            name: test-backend
            port:
              number: 8080
</code></pre>
<h1>测试</h1>
<p>可以先手动修改hosts</p>
<pre><code>192.168.1.101 www.test.com
</code></pre>
<pre><code>kubectl apply -f test-ingress.yaml
</code></pre>
<p>最后打开浏览器访问 www.test.com即可完成测试</p>
