---
title: Git
date: 2000-01-01 00:00:00
categories: 搭博客
tags: git
toc: true
---

### 1.初始化
```
git config --global user.name "Your Name"
git config --global user.email "email@example.com"

mkdir MyBlog
cd MyBlog
git init
```

### 2.关联github
本地git关联github帐号
```
ssh-keygen -t rsa -C "youremail@example.com"

在用户主目录中.ssh目录里生成id_rsa和id_rsa.pub两个文件
把id_rsa.pub粘贴到github的SSH KEYS里
```

关联仓库的两种情况：
```
git remote add origin git@github.com:username/MyBlog.git
	# 除了git协议还支持https://github.com/username/MyBlog.git，但是用https，在push时会要求填帐号密码


git clone git@github.com:username/MyBlog.git
	#从远程仓库克隆时，实际上Git会自动把本地的master分支和远程的master分支对应起来，并且远程仓库的默认名称是origin。
```

### 3.添加与推送文件
```
vim readme.txt
vim index.php
git add readme.txt
git add index.php
    # $ git add readme.txt
    # warning: LF will be replaced by CRLF in readme.txt.
    # The file will have its original line endings in your working directory.
    # 原因：windows下vim自动给最后加了一个换行，可改用notepad++编辑文件

#可以用 git add -A 或 git add . 提交所有变化

git commit -m "commit 2 files together"
    # git commit -m "#"     无描述

rm readme.txt
git rm readme.txt
git commit -m "remove readme.txt"

git push -u origin master       # 第一次加上-u和远程master分支关联
git push origin master

```

### 4.分支
```
git branch dev      # 添加dev分支
git checkout dev        # 切换分支      
        # git checkout -b dev       新建并切换到dev分支
git branch      # 查看分支

git checkout master
git merge dev       # git merge命令用于合并指定分支到当前分支

git branch -d dev       #删除dev

git push origin dev

# 默认git clone 只有master 不会有分支，该命令把远程的dev分支拉到本地
git checkout -b dev origin/dev
```

### 5
```
# 拉取最新变更
git pull git@github.com:yourname/MyBlog.git
```

### 参考
[廖雪峰git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

https://git-scm.com/

https://github.com/github/gitignore


