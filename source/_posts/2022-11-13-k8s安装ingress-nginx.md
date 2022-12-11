---
title: k8s安装ingress-nginx
urlname: 545db93f4db4d7ab858dc6064788f79f
date: '2022-11-13 10:10:32 +0800'
tags:
  - k8s
  - node
categories:
  - k8s
---

# 安装 ingress nginx

官网地址：[安装 ingress-nginx 教程](https:_github.com_kubernetes_ingress-nginx_blob_nginx-0.30.0_docs_deploy_index)
k8s 版本:1.18
ingress 版本: 0.30
因 ingress1.0 以上版本至少需要 k8s 1.19 以上版本

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml
```

有需要可以自己下载部署文件修改以后手动部署

### 提供服务（自建服务器）

官网地址：[裸机提供服务方法](https:_github.com_kubernetes_ingress-nginx_blob_nginx-0.30.0_docs_deploy_baremetal)

##### 基于 nodeport

注意 namespace 与 ingress-nginx 保持一致

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/provider/baremetal/service-nodeport.yaml
```

nodePort 的部署思路就是通过在每个节点上开辟 nodePort 的端口，将流量引入进来，而后通过 iptables 首先转发到 ingress-controller 容器中，而后由 nginx 根据 ingress 的规则进行判断，将其转发到对应的应用 web 容器中

##### 基于 hostNetwork

可以修改之前部署的 pod 文件添加以下配置

```
template:
  spec:
    hostNetwork: true
```

hostNetwork 模式不再需要创建一个 nodePort 的 svc，而是直接在每个节点都创建一个 ingress-controller 的容器，而且将该容器的网络模式设为 hostNetwork。也就是说每个节点物理机的 80 和 443 端口将会被 ingress-controller 中的 nginx 容器占用。当流量通过 80/443 端口进入时，将直接进入到 nginx 中。而后 nginx 根据 ingress 规则再将流量转发到对应的 web 应用容器中。

#### 两种部署方式的比较

相比较起来，nodePort 部署模式中需要部署的 ingress-controller 容器较少。一个集群可以部署几个就可以了。而 hostNetwork 模式需要在每个节点部署一个 ingress-controller 容器，因此总起来消耗资源较多。另外一个比较直观的区别，nodePort 模式主要占用的是 svc 的 nodePort 端口。而 hostNetwork 则需要占用物理机的 80 和 443 端口。
从网络流转来说，通过 nodePort 访问时，该 node 节点不一定部署了 ingress-controller 容器。因此还需要 iptables 将其转发到部署有 ingress-controller 的节点上去，多了一层流转。
另外，通过 nodePort 访问时，nginx 接收到的 http 请求中的 source ip 将会被转换为接受该请求的 node 节点的 ip，而非真正的 client 端 ip。
而使用 hostNetwork 的方式，ingress-controller 将会使用的是物理机的 DNS 域名解析(即物理机的/etc/resolv.conf)。而无法使用内部的比如 coredns 的域名解析。
因此具体使用哪种部署方式，需要根据实际情况和需求进行选择。

### 编写 ingress

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

可以先手动修改 hosts

```
192.168.1.101 www.test.com
```

```
kubectl apply -f test-ingress.yaml
```

最后打开浏览器访问 www.test.com即可完成测试
