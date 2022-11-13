---

author: zyqzyq
date: 2022-11-13 10:10:32+00:00
layout: post
title: k8s安装ingress-nginx
categories: k8s
key: 20221113
tags:
- k8s
---

# 安装ingress nginx

官网地址：[安装ingress-nginx教程](https://github.com/kubernetes/ingress-nginx/blob/nginx-0.30.0/docs/deploy/index.md)

 k8s版本:1.18

ingress 版本: 0.30

因 ingress1.0以上版本至少需要k8s 1.19以上版本

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml
```

有需要可以自己下载部署文件修改以后手动部署

### 提供服务（自建服务器）

官网地址：[裸机提供服务方法](https://github.com/kubernetes/ingress-nginx/blob/nginx-0.30.0/docs/deploy/baremetal.md)

##### 基于nodeport

注意namespace与ingress-nginx保持一致

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/provider/baremetal/service-nodeport.yaml
```

nodePort的部署思路就是通过在每个节点上开辟nodePort的端口，将流量引入进来，而后通过iptables首先转发到ingress-controller容器中，而后由nginx根据ingress的规则进行判断，将其转发到对应的应用web容器中

##### 基于 hostNetwork

可以修改之前部署的pod文件添加以下配置

```
template:
  spec:
    hostNetwork: true
```

hostNetwork模式不再需要创建一个nodePort的svc，而是直接在每个节点都创建一个ingress-controller的容器，而且将该容器的网络模式设为hostNetwork。也就是说每个节点物理机的80和443端口将会被ingress-controller中的nginx容器占用。当流量通过80/443端口进入时，将直接进入到nginx中。而后nginx根据ingress规则再将流量转发到对应的web应用容器中。

#### 两种部署方式的比较

相比较起来，nodePort部署模式中需要部署的ingress-controller容器较少。一个集群可以部署几个就可以了。而hostNetwork模式需要在每个节点部署一个ingress-controller容器，因此总起来消耗资源较多。另外一个比较直观的区别，nodePort模式主要占用的是svc的nodePort端口。而hostNetwork则需要占用物理机的80和443端口。

从网络流转来说，通过nodePort访问时，该node节点不一定部署了ingress-controller容器。因此还需要iptables将其转发到部署有ingress-controller的节点上去，多了一层流转。

另外，通过nodePort访问时，nginx接收到的http请求中的source ip将会被转换为接受该请求的node节点的ip，而非真正的client端ip。

而使用hostNetwork的方式，ingress-controller将会使用的是物理机的DNS域名解析(即物理机的/etc/resolv.conf)。而无法使用内部的比如coredns的域名解析。

因此具体使用哪种部署方式，需要根据实际情况和需求进行选择。

### 编写ingress

```
apiVersion: networking.k8s.io/v1
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
```

# 测试

可以先手动修改hosts

```
192.168.1.101 www.test.com
```

```
kubectl apply -f test-ingress.yaml
```

最后打开浏览器访问 www.test.com即可完成测试
