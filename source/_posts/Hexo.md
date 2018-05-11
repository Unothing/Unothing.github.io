---
title: Hexo
date: 2000-01-01 00:00:01
categories: 搭博客
tags: hexo
toc: true
---
### 0
建议先看[Git](/2000/01/01/Git/)

### 1.安装NodeJs

https://nodejs.org/en/

### 2.安装hexo
```
mkdir Myblog
cd Myblog

npm install -g hexo
npm install hexo-deployer-git --save
npm install hexo-generator-feed
hexo init
```

目录结构：
```
- node_modules  依赖包
- public    生成页面
- scaffolds 模板
- source    文章
- themes    主题
    - theme_name
        - _config.yml  主题配置文件
- _config.yml   博客配置文件
- db.json   source解析所得到的
- package.json  项目所需模块项目的配置信息
```

### 3.配置github

新建名为`yourname.github.io`的项目

### 4.配置blog

**注：`_config.yml`中所有`:`后都要有个空格！**

修改`Myblog/_config.yml`
主要部分
```
# Site
title: 
subtitle:
description:
author: 
language: zh-CN

#Writing
post_asset_folder: true     # 添加文章时自动生成同名文件夹用来放图片

# URL
url: https://yourname.github.io     # github.io分配的域名
permalink: posts/:title/			# 优化网站链接

# Extensions
theme: theme_name
plugin:
- hexo-generator-feed
feed:
    type: atom
    path: atom.xml
    limit: 20

# Deployment
deploy:
  type: git
  repo: git@github.com:yourname/yourname.github.io.git
  branch: master
```

修改`Myblog/theme/theme_name/_config.yml`
```
menu:
  首页: /#blog
  关于: /about
  归档: /archive
  RSS: /atom.xml
```

### 5.添加文章、页面 
```
hexo new page pagename      # 在source目录下生成pagename目录及pagename/index.md
hexo new post article      # 在source\_post目录下生成article.md及article文件夹（放置图片）
    #  hexo new article  ==   hexo new post article  
```
由于前面配了`post_asset_folder: true`在写文章时，只需把图片放到同名文件夹中，引用时无需具体路径直接写图片名即可

### 6.部署
```
hexo clean      #缩写：hexo cl
hexo generate   #缩写：hexo g    

hexo server     #缩写：hexo s     # 部署在本地        
hexo deploy     #缩写：hexo d     # 部署到github
```

### 7.备份
由于`hexo d`只会把生成的静态页面上传到github，为方便修改blog，把`MyBlog`下的所有源码传到`yourname.github.io.git`的`backup`分支
```
cd MyBlog
git init
git remote add origin git@github.com:username/yourname.github.io.git
git add .
git commit -m "#"
git push origin backup
```
如果对应的主题没有成功上传可能原因是，主题是通过`git clone`下载的，此时主题目录下会有`.git`，相当于一个项目下包含了另一个项目，导致的出错。解决办法：
```
删除对应主题目录下的.git
git rm -rf --cached theme_name/
```

### 参考
https://hexo.io/docs/