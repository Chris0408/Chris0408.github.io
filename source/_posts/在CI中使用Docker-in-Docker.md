---
title: 在CI中使用Docker-in-Docker
tags: 
- Docker
categories: 
- Docker
date: 2019-10-24
---
# 在CI中使用Docker-in-Docker
&emsp;&emsp;现在很多人喜欢使用`Docker-in-Dokcer`的方式去运行`CI`(e.g. 通过Jenkins),这样做看起来很便捷，但是同时也会带来许多问题。本文一起来探讨`Docker-in-Docker`可能带来的问题，以及避免这些问题的解决方案。
## Docker-in-Docker存在的主要问题
- **安全问题**  
&emsp;&emsp;在启动容器时，内部的Docker应用的安全配置文件可能会与外部Docker冲突或混淆，从而造成未知的安全问题。
- **存储驱动程序的问题**  
&emsp;&emsp;在使用`Docker-in-Docker`时，外部Docker在普通文件X系统（e.g. `EXT4`,`XFS`）之上运行，而内部的Docker在写时复制系统( `AUFS`, `Device Mapper`等上运行，具体取决于外部Docker。)但是有许多组合是不能生效的，比如在`AUFS`上运行`AUFS`是行不通的，而`Device mapper`则是没有命令空间隔离的，如果Docker的多个实例在同一台计算机上使用`Device Mapper`，则它们能够互相"看到"并影响彼此的镜像和容器支持设备。  
针对上述问题，Docker社区有人提出过一些解决办法，如将`/var/lib/docker`目录升级为卷就可以让`AUFS`运行在`AUFS`之上，手动为`Device Mapper`添加命名空间等。但这样做一是操作步骤十分复杂，二是可能为带来另外一些未知的异常。
- **镜像构建缓存问题**  
&emsp;&emsp;现在没有什么好的办法解决镜像构建缓存的问题。`Dokcer-in-Docker`中的内层Docker将无法视同宿主机上的镜像缓存，这意味着`CI`的每次构建和重建，都需要从头重新构建镜像，这将大大拖慢`CI`速度。
## 解决方案
&emsp;&emsp;其实大多数人运行`Docker-in-Docker`只是希望能够在CI系统本身位于容器中的情况下从CI系统运行Docker(e.g. 构建、运行、推送容器和镜像)  
&emsp;&emsp;其实最简单的办法是通过`-v`参数绑定挂载从而将宿主机Docker的Socket暴露给CI容器。在启动CI容器(Jenkins或其他)时可以使用类似如下所示的命令：
```bash
docker run -v /var/run/docker.sock:/var/run/docker.sock ...
```
&emsp;&emsp;现在，CI容器可以访问宿主机的Docker Socket,因此它可以创建容器以及进行一系列的Docker构建操作等。  
&emsp;&emsp;这样做看起来实现的`Docker-in-Docker`,使用体验也是如此，但其实并不是真正的`Docker-in-Docker`，当CI容器创建容器时,其实是在宿主机上进行了创建，这样就不会遇到上面描述的容器互相嵌套产生的一系列问题，并且可以复用构建缓存。


