---
layout:     post     # 使用的布局（不需要改）
title:      Sharding Sphere学习  
subtitle:   分库分表中间件Sharding JDBC学习
date:       2024-04-05
author:     Tong 
header-img: img/post-bg-2015.jpg 
catalog: true # 是否归档
tags: #标签
    - 分库分表
    - Sharding Sphere
    - 技术框架
typora-copy-images-to: ../assets/images
typora-root-url: ../../blog
---

## 如何打造高性能数据库集群服务？

面对大规模数据存储和处理需求时所遇到的一系列挑战，包括性能瓶颈、数据查询效率低、数据管理困难等。我们主要有两种处理方式：

- 读写分离
- 数据库分片



### 1、读写分离架构

读写分离的基本原理是将数据库的读操作和写操作分散到不同节点上，以减轻单个节点的访问压力。



### 2、数据分片架构

读写分离减轻了单台服务器的压力，但没有分散存储的压力，为了满足大数据量的存储需求，我们要对数据进行分片处理，数据库分片的有效手段是分库分表。数据分片的拆分方式可以分为水平拆分和垂直拆分。

- 水平拆分
  - *水平分表：* 单表切分为多表后，新的表即使在同一个数据库服务器中，也可以带来可观的性能提升，如果性能能够满足业务要求，可以不拆分到多台数据库服务器，毕竟业务分库也会引入复杂性，例如事务等。
  - *水平分库：* 如果单表拆分为多表后，单台数据库服务器依旧无法满足性能要求，那就需要将多个表分散在不同的数据库服务器中，两者方式可以结合起来使用。
- 垂直拆分

![image-20240406000306498](/assets/images/image-20240406000306498.png)



读写分离和数据分片架构应用程序与数据库集群之间的拓扑关系图：

![image-20240406001706373](/assets/images/image-20240406001706373.png)

上图中有n个应用程序和三个一主一从的数据库集群之间的数据交互关系。



### 3、常用数据分片策略

#### 1、取模分片

例如根据id取模分片

> 如果id是递增的，则数据均匀分布到各个分片上。但扩容时，原本是模3的现在要新增一个分片，那就需要对原先的所有数据进行模4再重新分配分片。

**优点**：**数据分布较为均匀**

**缺点**：**扩容需要大量数据迁移**



#### 2、按范围分片

例如按照时间月份进行分片

> 每个月份新增一个分片，旧分片的数据不需要迁移。但有六一八、双十一这种订单量大的时候，个别月份的分片数据量会比其他分片数据更多。也就会造成数据倾斜。

**优点：** **扩容不需要进行数据迁移**

**缺点：** **数据存放不均匀，容易产生数据倾斜**



#### 3、根据业务场景，灵活制定分片策略

待续。。。



#### 4、如何设计零迁移数据扩容分片方案？

![image-20240508232314117](/assets/images/image-20240508232314117.png)









