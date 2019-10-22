---
title: K8S使用Spring Boot Actuator健康检查
tags: 
- Java
- K8S
categories: 
- K8S
date: 2019-8-8
---
# K8S使用Spring Boot Actuator健康检查   
公司之前的集群一直用的是阿里云容器服务Swarm版。但是之前阿里云发公告称19年12月31日之后，将停止维护swarm集群，并下线控制台，开始主推K8S集群。我们也开始逐步将服务迁移至K8S集群，在K8S中使用`Spring Boot Actuator`健康检查时出现了一些小问题，在此记录一下。
## K8S的健康检查机制
K8S提供两类探针，可以对POD的健康状态进行检查
- K8S使用Spring Boot Actuator健康检查  
用于判断容器是否存活，即Pod是否为running状态，如果LivenessProbe探针探测到容器不健康，则kubelet将kill掉容器，并根据容器的重启策略决定是否重启，如果一个容器不包含LivenessProbe探针，则Kubelet认为容器的LivenessProbe探针的返回值永远成功。
- ReadinessProbe探针：  
用于判断容器是否启动完成，即容器的Ready是否为True，可以接收请求，如果ReadinessProbe探测失败，则容器的Ready将为False，控制器将此Pod的Endpoint从对应的service的Endpoint列表中移除，从此不再将任何请求调度此Pod上，直到下次探测成功。
- 每类探针都支持三种探测方法：  
exec：通过执行命令来检查服务是否正常，针对复杂检测或无HTTP接口的服务，命令返回值为0则表示容器健康。
httpGet：通过发送http请求检查服务是否正常，返回200-399状态码则表明容器健康。
tcpSocket：通过容器的IP和Port执行TCP检查，如果能够建立TCP连接，则表明容器健康。  

**一组探针示例如下:**
```bash
livenessProbe:               #存活检查，异常将重新调度
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 120    #等待程序启动时间
  timeoutSeconds: 6           #检测超时时间
  periodSeconds: 20           #检测频率
readinessProbe:               #就绪检测，未就绪不接入路由
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 100    #等待程序启动时间
  timeoutSeconds: 3           #检测超时时间
  periodSeconds: 10           #检测频率
```
## Spring Boot Actuator健康检查
 &ensp; &ensp; &ensp;Actuator 是 Spring Boot 提供的对应用系统的检查和监控的集成功能，可以查看应用配置的详细信息，例如自动化配置信息、创建的 Spring beans 以及一些环境属性等。使用Spring Boot Actuator还需特别注意安全性问题，屏蔽一些敏感信息的显示。具体怎么个屏蔽法就不说了，一般开发同事应该都会。  
&ensp; &ensp; &ensp;Actuator的health端点是主要用来检查应用的运行状态，也是使用频率最高的一个监控点。我们K8S中各个服务健康检查使用的就是Actuator提供的health进行的。  
&ensp; &ensp; &ensp;但是这里有一个问题需要注意下，特别是在微服务架构中。Actuator和health检查除了会检查应用自身的运行状态之外，还会默认检查，`数据库连接`、`Redis连接`、`RabbitMQ连接`等与服务相关的状态。只要有一个不正常，则健康检查判定为失败。所以在实际使用的过程中，特别是微服务架构中需要注意以下问题：  
&ensp; &ensp; &ensp;比如A服务数据库连接出现问题时，A服务健康检查就会失败，导致k8s尝试重启此服务，此时需要调用A服务的B服务调用A服务异常，导致B服务健康检查也失败，尝试重启。以此类推，形成了连锁反应，造成了服务大面积挂掉。所以，虽然Actuator提供的健康检查很方便，但是需要结合实际情况来看，是否需要开发同事自己实现一个健康检查接口，将单个服务挂掉的影响范围降到最小。
