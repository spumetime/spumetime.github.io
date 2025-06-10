---
title: Git常用命令
date: 2020-07-08 11:32:29
category: [猿工具]
tags: [git]
---

安装git之后，以下配置需要执行

# 账号配置

-- global代表是全局配置，如果使用这个参数，那么你本地所有git项目都是使用这两个

> user.name 用户名
>
> user.email 邮箱，可以选择不配置

配置全局参数

```bash
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

配置好了之后可以通过一下命令查询

```bash
$ git config --get user.name 
$ git config --get user.mail
```

如果使用以下不带global的命令，那么用户和邮箱只对当前仓库有效

```bash
$ git config user.name "John Doe"
$ git config user.email johndoe@example.com
```

# 账号授权

授权方式一般分为两种，https和ssh

## https

## ssh

> ssh原理可以参考本站ssh相关tag的文章

```
# 本地生成一个新的 SSH 密钥对
ssh-keygen -t ed25519 -C "your_email@example.com"
# -t ed25519	选用秘钥类型
# -C "your_email@example.com"	
# 上面语句执行完成之后，会在本地生成两个文件
# /Users/yourname/.ssh/id_ed25519 秘钥 严禁泄露，保存在本地
# /Users/yourname/.ssh/id_ed25519.pub 公钥，需上传至 GitHub/GitLab 等平台
# 生成之后可以执行以下命令查看
ls -al ~/.ssh
```

如果当前仓库是https协议，切换成ssh，执行以下命令

``` bash
git remote set-url origin git@github.com:someaccount/someproject.git
```

# 克隆代码

git clone

# 提交代码

git add 

git commit -m '提交信息'

git push

![image-20250610111325460](https://spumetime-blog.oss-cn-shenzhen.aliyuncs.com/img/image-20250610111325460.png)

# 其他命令

## 查看权限

第一步确定当前仓库链接远程代码是使用http协议还是ssh协议

```bash
git remote -v

-- 若显示有http则为http协议
```

查看当前本地仓库的账号是否有远程权限的方式

| 认证方式     | URL 格式                 | 验证命令                |
| ------------ | ------------------------ | ----------------------- |
| HTTPS + 令牌 | `https://github.com/...` | `git ls-remote`         |
| SSH          | `git@github.com:...`     | `ssh -T git@github.com` |
| GitHub CLI   | 自动管理，无需手动配置   | `gh auth status`        |
