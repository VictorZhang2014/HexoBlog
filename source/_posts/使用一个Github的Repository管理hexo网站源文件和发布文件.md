---
title: 使用一个Github的Repository管理hexo网站源文件和发布文件
date: 2016-10-04 00:04:16
tags: deployment
---

看到这篇文件的读者，表明您已经创建过了自己的博客网站通过hexo, 现在要解决的问题就是，如果在多台电脑管理（创建，修改，发布）hexo网站

### 问题：如果在A电脑创建了hexo网站，并且已经发布到了github上，也在该电脑创建了多份博客，此时换成B电脑，如果保持A电脑的修改，并且新加文章，或者修改A电脑发布的文章

### 解答：
    思路：那本博主的站点为例，
         1.我创建了victorzhang2014.github.io的github的public repository
         2.创建一个名为"hexo"的分支
         3."master"主分支用作存储发布Hexo网站的分支，
           "hexo"分支用作存储hexo源文件的分支
         4.多台电脑管理情况如下
           4.1 各电脑都要安装nodejs, hexo，
           4.2 各电脑指定一个目录，用来存放hexo所有源文件，并且需要`hexo init`初始化到当前目录
           4.3 在该目录添加git库，remote地址指向github的分支"hexo"
           4.4 把
                .DS_Store
                Thumbs.db
                db.json
                *.log
                node_modules/
                这些目录或者文件添加到.gitignore
           4.5 然后提交到"hexo"分支
               `git push origin hexo`
               
               此时，其他电脑就可以下载试用了
