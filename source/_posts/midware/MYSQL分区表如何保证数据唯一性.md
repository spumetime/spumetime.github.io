---
title: MYSQL分区表如何保证数据唯一性
categories:
  - 中间件
tags:
  - mysql
date: 2019-12-19 11:31:57
---

# **背景：**

由于业务数据量大，采用了分库分表（Mycat）；为了提高查询效率，使用了时间来分区；分区之后表的唯一索引必须带上分区字段。

假设有一张订单表（table_order）业务字段为order_no（订单号），分区字段为create_tm（创建时间）；唯一索引就是联合索引order_no+create_tm。

# 问题：

分布式系统中多个节点，对同一订单并发处理，发现table_order中没有该订单，然后都在该表中增加订单。因为订单号没有唯一索引，多个节点都新增数据成功，这就导致table_order中出现多条同样的订单数据。

![img](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/20250611012533362.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

# 解决方案一：

使用数据库唯一索引保证数据唯一性

建立一张中间表（假设为table_order_no），该表不分区，使用order_no作为唯一索引；同时保证每次在table_order中新增数据时，在table_order_no表中也新增数据。

![img](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/20250611012533367.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

# 解决方案二：

使用分布式锁防止并发处理

每次在处理订单时，都去尝试获取分布式锁，只有获取成功才能处理该订单。通过防止并发，保证一次只有一个节点处理订单，可以避免同时插入同一订单，从而保证订单唯一性。

![img](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/20250611012533572.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

# 方案二优化：

只在新增时，使用分布式锁，避免每次处理都去获取分布式锁，影响性能。

![img](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/20250611012533768.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑
