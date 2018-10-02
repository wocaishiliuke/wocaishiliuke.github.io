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

## 3.主题

#### 3.1 选择主题

> [选则一款主题](http://jekyllthemes.org/)。这里选用了NexT主题，该主题衍生自[Hexo NexT](http://theme-next.iissnan.com/)。参考[Jekyll NexT的安装说明](http://theme-next.simpleyyt.com/getting-started.html)，其中包括了：

- 1.bundler安装
- 2.clone主题到本地
- 3.使用bundle安装依赖
- 4.运行Jekyll
- 5.本地访问http://localhost:4000，查看效果

> 在bundle安装依赖时会无反应，参考[ali的ruby镜像说明](https://ruby.taobao.org/)，因为使用bundle，使用第二种bundle config的方式（可能要重启），再执行bundle install

#### 3.2 主题设置

> 参考[NexT官网](http://theme-next.simpleyyt.com/theme-settings.html)的主题样式设置，也可以集成第三方服务，完成本地测试

#### 3.3 主题集成

> 将本地测试后的主题，集成到username.github.io项目，主要包括：

- 将NexT项目除README等都拷贝到自己的项目
- 在_posts文件夹中存放编写的blog源文件，注意格式要求（日期和标题）
- _site用于存放Jekyll转换生成的页面，要在.gitignore中忽略掉
- 提交代码即可

## 4.集成第三方服务

#### 4.1 百度统计

- 1.在百度统计添加该博客的网站列表
- 2.在代码获取中，得到hm.js?后的参数
- 3.在主配置文件_config.yml中设置baidu_analytics即可（注意yml格式）

```
<script>
var _hmt = _hmt || [];
(function() {
  var hm = document.createElement("script");
  hm.src = "https://hm.baidu.com/hm.js?xxxxxxxxxxxxxxxxxxxx";
  var s = document.getElementsByTagName("script")[0]; 
  s.parentNode.insertBefore(hm, s);
})();
</script>
```

#### 4.2 TODO

# III.目录介绍

> 该博客工程的目录结构：

|文件/目录 | 描述 |
|:------------|:-------|
|_config.yml |保存配置数据。很多配置选项都会直接从命令行中进行设置，但是如果你把那些配置写在这儿，你就不用非要去记住那些命令了|
|_drafts|drafts 是未发布的文章。这些文件的格式中都没有 title.MARKUP 数据。学习如何使用 drafts|
|_includes| 你可以加载这些包含部分到你的布局或者文章中以方便重用。可以用这个标签 {!% include xxx.ext %!} 来把文件 _includes/xxx.ext 包含进来（该标签没有!，是为了不报错）|
|_layouts   layouts |是包裹在文章外部的模板。布局可以在 YAML 头信息中根据不同文章进行选择。 这将在下一个部分进行介绍。标签 {{ content }} 可以将content插入页面中|
|_posts|这里放的就是你的文章了。文件格式很重要，必须要符合: YEAR-MONTH-DAY-title.MARKUP。 The permalinks 可以在文章中自己定制，但是数据和标记语言都是根据文件名来确定的|
|_data|放一些其他配置文件,必须是.yml或者.yaml格式的,比如有一个文件叫members.yml,如果想引用这个文件里的内容就通过site.data.membres来引用|
|_site|一旦 Jekyll 完成转换，就会将生成的页面放在这里（默认）。最好将这个目录放进你的 .gitignore 文件中|

# IV.域名绑定

## 1.域名购买

> 阿里云购买

## 2.域名解析

> 在阿里云控制台设置。因为是域名间跳转，所以添加CNAME类型的域名解析

|记录类型|主机记录|解析线路|记录值|TL|备注|
|:------|:-----|:------|:----|:--|:--|
|CNAME|blog（.wocaishiliuke.cn）|默认|wocaishiliuke.github.io|10 min| |
|CNAME|@（.wocaishiliuke.cn）|默认|wocaishiliuke.github.io（要.?）|10 min|可选，@是为了wocaishiliuke.cn也能访问|

> 域名解析变更需要时间来设置各级DNS服务器

## 3.CNAME

> - 该文件告诉Github Pages服务器要想指定的域名，该域名不能包含前缀信息
> - 该文件只能配置一个域名，要映射多个域名，需要创建多个CNAME文件

#### 方式1：页面中设置

> - settings-->GitHub Pages的Custom domain中设置跳转的域名
> - 该方式会自动创建CANME文件

#### 方式2：手动创建

> 创建CNAME文件（文件名必须大写，没有后缀。只能配置一个）

```
blog.wocaishiliuke.cn
```

## 5.参考

- [Jekyll](https://www.jekyll.com.cn/docs/github-pages/)
- [Jekyll主题](http://jekyllthemes.org/)
- [NexT源码](https://github.com/simpleyyt/jekyll-theme-next)
- [NexT-Jekyll官网](http://theme-next.simpleyyt.com/getting-started.html)