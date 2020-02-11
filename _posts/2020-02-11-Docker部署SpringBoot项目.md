---
title: Docker部署SpringBoot项目
tags: Docker
sidebar:
  nav: docs-zh
---

### 将springboot项目打成jar包

### 准备Dockerfile文件

```
# 基础镜像使用java
FROM java:8
# VOLUME 指定了临时文件目录为/tmp。
# 其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp
VOLUME /tmp 
# 将jar包添加到容器中并更名为app.jar
ADD ms-platform-center.jar app.jar 
# 运行jar包
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]

```
jar包和Dockerfile文件放在同一目录下
[![docker](https://jialiangbujiaj1a.github.io/imgs/docker/Docker部署.png)]

### 制作镜像
docker build -t eureka .

[![docker](https://jialiangbujiaj1a.github.io/imgs/docker/dockerbuild.png)]

### 启动容器

docker run -d -p 8761:8761 eureka
-d参数是让容器后台运行 
-p 是做端口映射，此时将服务器中的8761端口映射到容器中的8761(项目中端口配置的是8761)端口


