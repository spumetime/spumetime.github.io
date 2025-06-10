---
title: 一次Mycat聚合排序问题定位
categories:
  - 中间件
tags:
  - mycat
date: 2019-11-20 14:20:56
---

## **背景：**

Mycat+Mysql实现分库分表；功能需要遍历表，数据量有点大，于是通过id排序分页查询。

## 问题：

在实现过程中发现有部分id查不出来，功能排期只能被迫延后。

## 定位第一阶段：

在确认了代码和SQL语句没有问题后，目光转至Mycat与Mysql排序逻辑上来。

## 定位第二阶段：

确认了Mycat与Mysql排序逻辑有问题，至少Mycat与Mysql排序逻辑表现不一致。

1.执行以下语句

```sql
select id from <表名> WHERE id='AAABaOrW/0roLRp3FoZOzIRplM1qVQtQ';
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

结果如下

![img](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/20250611012639910.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

2.执行以下语句

```sql
select id from <表名> WHERE id>='AAABaOrW/0roLRp3FoZOzIRplM1qVQtQ' ORDER BY id ASC limit 10;
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

结果如下，却没有AAABaOrW/0roLRp3FoZOzIRplM1qVQtQ这个id

![img](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/20250611012639923.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

## 定位第三阶段：

Mycat聚合排序原理

Mycat执行select * from <表名> where id>='AAABaOrW/0roLRp3FoZOzIRplM1qVQtQ' order by id asc limit 10时，会在全部Mysql分库中，执行上述语句，然后在Mycat中对所有Mysql分库的结果重新排序，拿出10条。

然后去找Mycat源码（1.6版本），找到了这个类RowDataPacketSorter，发现该类最后排序大小比较是采用BytesTools.compareTo方法，字节排序，那应该是大小写敏感的。于是想，mysql中排序是不是大小不敏感？

## 定位第四阶段：

看Mysql的字符排序

查看使用表的排序规则，发现是utf8mb4_general_ci，然后在去搜索一波这个排序规则发现

后缀  ci  :不区分大小写
 bin ：区分大小写

果不其然，我使用的mysql规则是大小不敏感的。

至此问题找到，于是分页不采用以UUID排序，而是使用了另一个全部大写的字段，问题也得以解决。

# 结论：

在使用Mycat+Mysql字符排序时要注意大小写是否敏感的问题。

之前网上搜索看到Mycat聚合排序有BUG，还以为被自己撞见了，原来发现非也、非也！
