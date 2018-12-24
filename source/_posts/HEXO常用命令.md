title: HEXO常用命令
date: 2018-06-11 22:18:07
tags:
- HEXO



### HEXO常用的几个命令
---

***安装***
> npm install -g hexo-cli

***更新***
> npm update hexo -g

***初始化网站目录***

> hexo init "文件名"

> cd "文件名"

> npm install


***启动本地服务***
> hexo server

***生成静态页面***
> hexo generate

***部署网站到Github上***
> hexo deploy

***部署步骤,分三步***
>  
 1. hexo clean
 2. hexo generate
 3. hexo deploy

***新增文章***
> hexo n "文章名"

***更换主题***

 1. 从Github下载主题
 > git clone https://github.com/theme-next/hexo-theme-next themes/next

 2. 修改_config.yml中的theme配置
> theme : next
