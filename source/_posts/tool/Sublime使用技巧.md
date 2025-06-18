---
title: Sublime使用技巧
categories:
  - 猿工具
tags:
  - sublime
date: 2025-06-11 01:35:36
banner: sublime text
---

# 控制台用sublime打开文件

sublime安装的时候，自带subl命令行工具。路径在sublime安装包下的bin目录，/Applications/Sublime\ Text.app/Contents/SharedSupport/bin/。

![image-20250617151313190](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/image-20250617151313190.png)

将subl拷贝到自己命令环境里

> 查看MAC的命令环境目录有哪些
>
> echo $PATH

我使用/usr/local/bin目录（如果目录不存在，则需要手动创建一次）,将subl工具拷贝到/usr/local/bin目录下，命令如下

> sudo ln -s /Applications/Sublime\ Text.app/Contents/SharedSupport/bin/subl /usr/local/bin/subl

# 安装插件

## 安装插件入口

![image-20250617152118591](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/image-20250617152118591.png)

点击Install Package命令

![image-20250617152342157](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/image-20250617152342157.png)

## XML插件

插件名：Indent XML

![image-20250617152301982](/Users/admin/Library/Application Support/typora-user-images/image-20250617152301982.png)

格式化按钮路径

![image-20250617152626278](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/image-20250617152626278.png)

## JSON插件

安装pretty json插件，方式和XML的一样

![image-20250617152809237](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/image-20250617152809237.png)

配置快捷键

![image-20250617153355897](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/image-20250617153355897.png)

输入以下配置，就可以快速格式化JSON了

> {"keys": ["ctrl+command+j" ], "command": "pretty_json" }

![image-20250617153446881](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/image-20250617153446881.png)

