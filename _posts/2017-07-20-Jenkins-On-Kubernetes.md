---
layout: post
title:  "Jenkins Pipeline遇上Kubernetes碰出的火花"
categories: Github
tags: Jenkins Kubernetes Pipeline
author: Simple Hacker
comments: true
---
## 背景介绍
当前开源组建支持了在Kubernetes上使用Jenkins搭建持续集成持续部署环境，整体架构如下图所示：
![jenkins-kubernetes-arch](https://user-images.githubusercontent.com/6065072/28499330-d35ffc0a-6fe5-11e7-97ac-02d9f20a5b32.png)
上图中，Jenkins Master运行在Kubernetes容器中，通过Kubernetes API Server跟Kubernetes整合，当有需要构建任务时，Jenkins通过API Server创建POD完成构建任务，该POD在构建结束以后被销毁。
Jenkins和Kubernetes的结合为我们提供了如何好处：
1. Jenkins可以根据构建任务动态创建Agent，解决了Agent容量管理的问题
2. 构建过程中需要的工具链可以由工程师包装在生成POD的Docker镜像中，解决了Jenkins构建工具链的维护问题
3. 整个Jenkins持续集成环境可以很方便的通过Kubernetes命令创建或回滚，提高了构建环境的健壮性

本文介绍了如何在Kubernetes上搭建Jenkins持续集成和持续部署环境。
## 前提条件
本文搭建步骤仅仅包含将Jenkins部署到Kubernetes上，默认已经有一个可用的Kubernetes环境。关于Kubernetes的部署，请参考Kubernetes部署文档。

## 搭建步骤
大体步骤如下：
1. 创建Jenkins Master镜像
2. 部署Jenkins到Kubernetes
3. 配置Jenkins云
### 构建Jenkins Master镜像
官方Jenkins镜像中不包含Jenkins插件，为了确保可重现性，保证基础设施可以从git源码重建。所以需要将Jenkins插件打到Docker镜像中。
```
FROM jenkins:2.7.4

# install jenkins plugins
......
RUN install-plugins.sh kubernetes-ci:1.3
RUN install-plugins.sh kubernetes:0.10
......

RUN chown -R jenkins:jenkins /var/jenkins_home
```
请注意：install-plugins.sh将从Jenkins更新中心下载插件，并安装到正确的目录下。


### 部署Jenkins到K8S
1. Jenkins部署
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: jenkins
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: master
    spec`
      containers:
      - name: master
        image: <jenkins-image-master>
        ports:
        - containerPort: 8080
        - containerPort: 50000
        readinessProbe:
          httpGet:
            path: /login
            port: 8080
          periodSeconds 10
          timeoutSeconds: 5
          successThreshold: 2
          failureThreshold: 5
        resources:
          limits:
            cpu: 4000m
            memory: 8000Mi
          requests:
            cpu: 4000m
            memory: 8000Mi
```
2. 创建Jenkins服务
```
---
  kind: Service
  apiVersion: v1
  metadata:
    name: jenkins-ui
  spec:
    type: NodePort
    selector:
      app: master
    ports:
      - protocol: TCP
        port: 8080
        targetPort: 8080
        name: ui
---
  kind: Service
  apiVersion: v1
  metadata:
    name: jenkins-discovery
  spec:
    selector:
      app: master
    ports:
      - protocol: TCP
        port: 50000
        targetPort: 50000
        name: slaves
```
3. 赋予服务账号权限
因为Jenkins将通过K8S服务账号default创建或者删除POD，同时该账号将在持续集成持续部署流水线上创建namespace用于集成测试，所以该服务账号需要拥有cluster范围内的admin权限。
```
# This cluster role binding allows default service account in specified(on commandline) namespace to have cluster wide admin permission
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: default-admin-binding
subjects:
- kind: ServiceAccount
  name: default
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```
### 配置Jenkins
通过配置->全局配置->添加云将Kubernetes和部署在Kubernetes之上的Jenkins集成起来。这样Jenkins有构建任务时，动态根据POD定义创建agent。
![add-kube](https://user-images.githubusercontent.com/6065072/28499817-27d0b95e-6ff1-11e7-87d6-ef97cda94ae6.png)
注意：上图中的Credentials是Kerbernetes Service Account类型的（安装了Kubernetes Plugin之后就有这种类型的credentials）。这里的服务账号是被赋予了cluster范围admin权限的default账号，就是部署环节提到的default账号。
## 参考文档
* [Kubernetes Plugin](https://github.com/jenkinsci/kubernetes-plugin)
* [使用Kubernetes-Jenkins实现CI/CD](https://www.kubernetes.org.cn/1791.html)
* [基于Kubernetes 部署 jenkins 并动态分配资源](http://www.cnblogs.com/hahp/p/5812455.html)
