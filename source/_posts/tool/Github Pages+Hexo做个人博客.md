---
title: Github Pages+Hexo做个人博客
date: 2025-06-09 01:49:37
categories: [猿工具]
tags: [个人博客]
leftbar: ['related']
---

Hexo官网教学：[https://hexo.io/zh-cn/docs/](https://hexo.io/zh-cn/docs/)。下面是我自己的实操，请配合使用。

# 本地安装Hexo

前置要求：

+ git ：我都是使用idea时，直接下载好
+ node.js：MAC直接下载dmg文件即可。官网下载地址：[https://nodejs.org/zh-cn](https://nodejs.org/zh-cn)

接下来安装hexo：**<font style="color:#DF2A3F;">Mac使用brew install hexo命令</font>**。

![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749399679624-c151ca99-e37a-4882-9ea8-7218cf72b409.png)

验证一下 hexo -v，成功

![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749399750487-7b97b305-5601-41d0-961e-f4d1c517aaaf.png)

# 本地Hexo建站

先使用hexo init myhexo命令，其中myhexo是文件夹。执行完成后cd进入 ll命令 可以看到以下目录![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749400080520-19a75ec2-0bce-4297-95c5-d261c652207f.png)



然后执行npm install，会补充一些文件![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749400122776-bc1a4ae4-b5a9-412a-8648-9fda239d526c.png)



# 本地启动Hexo

Hexo 3.0 把服务器独立成了个别模块。 您必须先安装 hexo-server 才能使用。命令如下

> npm install hexo-server --save

安装hexo-server之后，执行hexo server命令启动，访问地址为http://localhost:4000，和jekyll一样。![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749400517546-cadd1215-c538-496a-b4a5-037da222f627.png)

访问[http://localhost:4000](http://localhost:4000)，可以看到下面博客首页。不得不说就是比jekyll简单。

![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749400558016-b2e56547-0ff5-4d37-b39e-b07d6eef9411.png)

# 部署到Github

本节参考官网：[https://hexo.io/zh-cn/docs/github-pages](https://hexo.io/zh-cn/docs/github-pages)

建立名为 username.github.io的储存库，为什么要这样命名可以参考我的另一篇文章《Github Pages+Jekyll做个人博客》。

提交代码到github，我的.gitignore配置如下。

```plain
/public/
.idea
node_modules
```

然后node --version，得到版本号。我的是v23.11.0

在储存库中前往 Settings > Pages > Source 。 将 source 更改为 GitHub Actions，然后保存。

在储存库中建立 .github/workflows/pages.yml，切记要在这里建

![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749402973502-c9b07fa9-2de9-4d84-9ba0-d95ba175e422.png)

并填入以下内容 (将 20 替换为上个步骤中记下的版本)：

```plain
name: Pages

on:
  push:
    branches:
      - main # default branch 请一定要注意和你的主干分支对得上

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 20
        uses: actions/setup-node@v4
        with:
          # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
          # Ref: https://github.com/actions/setup-node#supported-version-syntax
          node-version: "20"
      - name: Cache NPM dependencies
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

然后等待github部署完成，就可以输入name.github.io访问了

![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749404573682-243248d9-a571-4a40-a292-ab3d0750dea3.png)

# 修改主题

找到hexo的主题官网：https://hexo.io/themes/，我看中下面这个Claudia主题点击标题就可以看到如何安装了。

![image-20250609215355907](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/image-20250609215355907.png)

![image-20250609215417144](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/image-20250609215417144.png)

按提示运行即可

![image-20250609215645475](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/image-20250609215645475.png)

# Github Pages支持外链图片

我们技术文章一般都包含图片，我之前是用语雀写的，语雀图片链接是防盗链，github无法访问。尝试了好多其他方法都不行，最后就按下面方法搭建的图床服务解决的。这个需要花点钱。



