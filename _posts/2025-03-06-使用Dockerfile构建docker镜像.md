---
layout:     post     # 使用的布局（不需要改）
title:      使用Dockerfile构建docker镜像 
subtitle:   构建arm架构的xxl-job-admin
date:       2025-03-06 # 时间
author:     Tong # 作者
header-img: img/post-bg-2015.jpg #这篇文章标题背景图片
catalog: true # 是否归档
tags: #标签
    - docker
    - xxl-job-admin

typora-copy-images-to: ../assets/images
typora-root-url: ../../blog
---

最近要从零开始搭建整套微服务，发现xxl-job-admin没有Mac对应的镜像，所以需要自己手动构建一个。



## 1、拉取xxl-job代码

直接去github拉取就好了，没啥好说的



## 2、切换到自己想要的代码分支

也没啥好说的



## 3、执行doc/db目录下的tables_xxl_job.sql

效果如下：

<img src="/assets/images/image-20250306162921037.png" alt="image-20250306162921037" style="zoom:120%;" align=left />



## 4、修改配置

数据库配置

**注意：** **host.docker.internal** 这是宿主机的主机名，docker容器内访问宿主机可以通过主机名访问

```properties
### xxl-job, datasource
spring.datasource.url=jdbc:mysql://host.docker.internal:3306/xxl-job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=root123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```



Dockerfile配置

````dockerfile
FROM openjdk:8-slim
MAINTAINER tong

ENV PARAMS=""

ENV TZ=PRC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ADD target/xxl-job-admin-*.jar /app.jar

ENTRYPOINT ["sh","-c","java -jar $JAVA_OPTS /app.jar $PARAMS"]
````







## 5、构建镜像

进入/xxl-job/xxl-job-admin目录

**注意：**构建镜像的时候需要提前把**openjdk:8-slim**镜像拉下来，不然会因为**某种原因**报错。

```shell
docker pull openjdk:8-slim
```



执行构建命令

```shell
docker build --build-arg ARCH=arm64 --build-arg OS=linux -t tong/xxl-job-admin:2.3.0-arm64 -f Dockerfile .
```

构建结果

![image-20250306163922390](/assets/images/image-20250306163922390.png)

docker桌面版

![image-20250306164100894](/assets/images/image-20250306164100894.png)



## 6、运行容器

由于我在配置文件里改了数据库配置，这里就不指定了

```shell
docker run -it \
-p 8080:8080 \
--name cloud-xxl-job-admin  \
-d tong/xxl-job-admin:2.3.0-arm64
```

![image-20250306164327957](/assets/images/image-20250306164327957.png)

![image-20250306164354043](/assets/images/image-20250306164354043.png)





我们也可以通过指定数据库配置的方式运行

```shell
docker run -it \
-e PARAMS='--spring.datasource.url=jdbc:mysql://host.docker.internal:3306/xxl-job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai --spring.datasource.username=root --spring.datasource.password=root123456' \
-p 8080:8080 \
--name xxl-job-admin  \
-d tong/xxl-job-admin:2.3.0-arm64
```

效果是一样的，这就不演示了



自此，xxl-job-admin服务已搭建完成

