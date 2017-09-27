---
title: 使用Hexo搭建个人博客网站
date: 2016-10-03 23:50:11
tags: victor, hexo
---

使用Hexo搭建个人博客网站，类似这种文章在网上能找到很多，在这里我就不再一一赘述了，以下是个人搭建博客站点的经验所得，以分享。

Let's get started!

需要安装git和nodejs

安装nvm，Node版本管理器
```
brew install nvm
```

安装nodejs
```
nvm install 4.5.0
```

### 注意：如果以上遇到nvm和nodejs的命令安装不成功的话，请使用各自官网的MAC OS X的Installer

#### 1.新建一个github的repository，名称必须后缀为.github.io
   例如：  victorzhang2014.github.io 名称必须和自己的github账户名相同

#### 2.安装hexo  全局安装，加-g参数
  ```
   sudo npm install hexo --save   # 或者 sudo npm install -g hexo
   sudo npm install hexo-deployer-git -save
   sudo npm install hexo-server --save
  ```

#### 3.新建一个hexo文件夹
  ```
  hexo init victorzhang
  ```
hexo会在victorzhang目录下新建hexo所需要的文件

   依次执行以下命令
   ```
   npm install hexo-renderer-ejs --save
   npm install hexo-renderer-stylus --save
   npm install hexo-renderer-marked --save
   ```

   主题的切换，hexo默认的主题的landscape, 也可以不需要下载jacman主题风格
   主题网站：  https://hexo.io/themes/
   找一个喜欢的，先clone到themes目录下，然后在_config.yml文件改下theme: 你指定的主题名称
   ```
   git clone https://github.com/duanhjlt/jacman.git themes/jacman
   ```
   在_config.yml的配置文件中修改配置如下
   ```
   theme: jacman
   ```

#### 4.启动服务，可以在本地预览了
   ```
   hexo server
   ```

#### 5.新建文件
   先停止本地运行的server
    ```
    hexo new “hello world“
    ```
  会生成一个文件，然后编辑文件

#### 6.生成静态文件
   ```
   hexo  generate
   ```
 
#### 7.配置_config.yml文件
   拖到最后，例如下面：
   注意：冒号后面一定要有一个空格，否则不生效
   ```
   deploy:
     type:git
     repository:git@github.com:VictorZhang2014/victorzhang.github.io.git
     branch:master
   ```

一定要先配置好ssh访问github
如果不想使用SSH配置，当然也可以配置https访问，
例如：
```
deploy:
  type: git
  repo: https://github.com/VictorZhang2014/victorzhang2014.github.io.git
  branch: master
```

#### 8.安装hexo-deployer-git
   第一次使用需要安装
  ```
  npm install hexo-deployer-git —save
  ```

#### 9.发布
 ```
 hexo deploy
 ```
 它会自动把本地文件上传到github上

#### 10.可以成功预览了
  https://victorzhang2014.github.io

#### 11.如果需要绑定自己的申请的域名和hexo站点文件，需要建立一个CNAME文件
  ```
   echo  > public/CNAME
  ```
  CNAME内容是，例如：(以下是我个人的网站域名，读者请换成自己申请的域名)
  ```
  www.googleplus.party
  googleplus.party
  blog.googleplus.party
  ```

常用命令
```
hexo new   文章名称 #新建文章
hexo new page  页面名称 #新建页面
hexo generate  #生成静态页面至public目录
hexo server   #开启本地预览访问端口
hexo deploy  #发布到github上
hexo help  #查看帮助
hexo version #查看hexo版本号
```
