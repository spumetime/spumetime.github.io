---
title: Github Pages+Jekyll做个人博客
date: 2025-06-09 01:49:37
tags: [猿工具]

---

官方文档：[https://docs.github.com/en/pages/quickstart](https://docs.github.com/en/pages/quickstart)

中文文档：[https://docs.github.com/zh/pages/quickstart](https://docs.github.com/en/pages/quickstart)

<h1 id="wrbQf">新建public仓库</h1>

在按照官方文档操作之前，需要注意一个点。名字只能是 name.github.io。name得是自己账户的name，如下两个仓库，只有第一个才能使用[spumetime.github.io](https://spumetime.github.io/)访问，后者不能使用zoe.github.io访问，而是spumetime.github.io/zoe.github.io。

![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749360496115-0ea7900a-4970-4eaa-9d3c-9fc0f664e195.png)

<h1 id="r1WM0">配置Github Pages</h1>

![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749361066195-ed017349-0b28-4a4f-a0d4-eb65427d31e3.png)

可以输入网址验证是否成功

![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749362165509-40a6c726-6fca-47a1-81f2-4c73690712f3.png)

<h1 id="k12ZT">配置Jekyll主题</h1>

参考文档：[https://docs.github.com/zh/pages/setting-up-a-github-pages-site-with-jekyll/adding-a-theme-to-your-github-pages-site-using-jekyll](https://docs.github.com/zh/pages/setting-up-a-github-pages-site-with-jekyll/adding-a-theme-to-your-github-pages-site-using-jekyll)

在仓库的根目录下创建_config.yml文件，里面添加  theme: jekyll-theme-minimal 

![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749362292690-16e9c377-81d8-4a19-ab09-00501ba39546.png)

再访问一次，就发现主题变了。

![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749362009060-333752d1-31ae-417c-8ee7-6bf527ba027a.png)

<h1 id="UsBpV">新增一个Blog</h1>

把git库克隆本地，按照官方文档加文件+文件夹即可：[https://jekyllrb.com/docs/structure/](https://jekyllrb.com/docs/structure/)。经过个人验证最小文件目录结构如下。注意_post文件夹是放博客的，博客名称有强制规则，YEAR-MONTH-DAY-title.MARKUP。

```shell
.
├── _config.yml
├── _posts
│   ├── 2007-10-29-why-every-programmer-should-play-nethack.md
│   └── 2009-04-26-barcamp-boston-4-roundup.md
└── index.html # can also be an 'index.md' with valid front matter

```

<h1 id="xGrcC">本地安装Jekyll</h1>

如果你不在本地部署个人博客的话，Jekyll可以不用了。

[https://jekyllrb.com/docs/installation/macos/](https://jekyllrb.com/docs/installation/macos/)

按照顺序安装即可

安装完成之后，去建一个jekyll项目验证下是否有用

```shell
-> % jekyll new test

输出：New jekyll site installed in /Users/linyong/myblog/test. 就是成功了

```

![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749367132097-56b51dae-728f-4ee7-88bc-2687a369e8cf.png)

cd到你刚创建的项目下，执行jekyll server命令，出现下面红框即为成功

![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749367392776-abc613c1-e7f1-4db2-9024-20c5b7f2f4ac.png)

输入 地址 [http://127.0.0.1:4000/](http://127.0.0.1:4000/)查看是否能访问

![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749367437298-134ecbc3-0245-4c6a-9a5e-edb57239e607.png)

<h1 id="wKRo7">换Github支持的Jekyll主题</h1>

换主题前，先理解一Gemfile 和 Gemfile.lock 文件。它们由 Bundler 用于跟踪构建 Jekyll 站点所需的 gem 及其版本。[https://jekylldo.cn/docs/themes/](https://jekylldo.cn/docs/themes/)

> 这里注意 jekyll server 和 bundle exec jekyll serve 两个的区别是前者基本本地 Jekyll 版本启动服务，后者基于目录下的 Gemfile 文件启动服务，所以我们要用后者。

这个是github支持的主题[https://pages.github.com/themes/](https://pages.github.com/themes/)

以换minimal-mistakes主题为例，参考文档 [https://github.com/mmistakes/minimal-mistakes?tab=readme-ov-file](https://github.com/mmistakes/minimal-mistakes?tab=readme-ov-file)，以及[https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#remote-theme-method](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#remote-theme-method)。采用远程主题方式。

![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749375088228-32d225c6-52f8-4864-96b4-0506d75887d7.png)

<h1 id="Nx0fc">换jekyllthemes.org主题</h1>

1. 地址：[http://jekyllthemes.org/](http://jekyllthemes.org/)选取一个主题下载后解压放入本地git仓库
2. 然后使用bundle一下，下载相关依赖![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749382047015-d3643714-1d9f-4118-ac51-decf5edfdb1a.png)
3. 使用jekyll server启动即可![](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/1749382082691-883e0abe-5046-4431-bbc9-de47b4c23ee9.png)
