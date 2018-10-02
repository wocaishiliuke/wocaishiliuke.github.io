---
title: Github Pages + Jekyll博客搭建
date: 2018-05-01 00:00:00
categories:
    - Blog
tags:
    - Github Pages
    - Jekyll
    - Blog
---

本文介绍使用Github Pages和Jekyll搭建博客。该方式方便简单，不需要个人服务器、管理域名等，和github浑然一体，方便管理和更新。

<!-- more -->

##### 目录
+ I.简介
+ II.创建
+ III.目录介绍

---

> 系统：Ubuntu16.04LTS

# I.简介

> - [Github Pages](https://pages.github.com/)
分为[两种基本类型](https://www.jekyll.com.cn/docs/github-pages/)：用户/组织的站点、项目的站点
> - 用户和组织的站点：被放置在一个专用仓库中，在该仓库中只存在Github Pages的相关文件。这个仓库应该根据用户/组织的名称来命名：username.github.io。仓库master分支的文件将会被用来生成Github Pages站点
> - 项目的站点：文件存放在项目本身仓库的gh-pages分支中。该分支下的文件将会被Jekyll处理，生成的站点会被部署到用户站点的子目录上，例如username.github.io/project （除非指定了自定义的域名）

> [Jekyll](https://www.jekyll.com.cn/)是一个简单的博客形态的静态站点生产机器。可以将 Markdown、Textile和Liquid转化成一个完整的可发布的静态网站。也可以运行在GitHub Page上，即使用GitHub的服务搭建博客或者网站等

> 这里使用用户/组织的站点形式，即在Github上创建username.github.io项目即可

# II.创建

## 1.创建github项目

> 参考[官网](https://pages.github.com/)的 User or organization site

```
# clone到本地  
git clone .../username.github.io.git

# 生成index页面
echo "Hello GitHub Pages!" > index.html

# 提交至master
git add index.html                          
git commit -m "first commit"  
git push origin master
```

> 访问https://username.github.io即可

## 2.安装Jekyll

- 1.配置ruby环境

> 参考[Ubuntu环境搭建](http://wocaishiliuke.github.io/linux/2018/06/30/Ubuntu01/)，完成Ruby和RubyGems的安装

- 2.安装Jekyll

```
gem install jekyll
```

