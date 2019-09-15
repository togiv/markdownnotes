## 创建博客：首先下载完成hugo
## 创建本地
+ Hugo new site myblog(自己起名) 创建博客，申请站点
+ Cd myblog(目录名)
+ Git clone 网址 themes/主题名         主题网址
+ Hugo server -t 主题名 --buildDrafts   
+ 新建文章  hugo new post/名字
+ Github上建好仓库
+ Hugo --theme=主题名 --baseUrl=仓库地址(后半段)（也是远程博客地址）--buildDrafts  生成public文件
+ Cd public/  切换目录

## 接下来推到远端
*在public目录下*
+ Git init 
+ Git add *
+ Git commit -m “提交信息”
+ Git remote add origin 仓库地址.git
+ Git push -u origin master
