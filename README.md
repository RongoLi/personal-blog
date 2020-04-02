## 个人博客源码
### 说明
该博客使用Hexo + hexo-theme-matery(主题)搭建完成，你可以基于该部分源码快速搭建属于你自己的个人博客。
在开始搭建你的个人博客前有些必要的工具以及配置信息你需要了解和注意。该Project分为两部分:第一部分是就是整个项目工程的[源码仓库](https://github.com/RongoLi/personal-blog)，第二部分是通过Github创建的[个人博客站点](https://github.com/RongoLi/rongoli.github.io)。第一部分就是一个普通的代码仓库，这里就不多说了，如果需要通过Github的免费站点搭建个人博客的话你可能需要了解第二部分，详细请看下面的`基于Github搭建个人博客`

### 基本工具
#### Git的基本使用
本项目的版本管理是依赖Git,所以在开始前你得对Git有一定的了解以及基本的使用。这里你可以查看个人总结的[Git使用摘要](http://rongo.icu/2019/07/09/git-shi-yong-zhai-yao/)进行简要了解。

#### Hexo命令
由于个人博客的发布需要使用到Hexo的一些指令，所以你需要对[Hexo Command](https://hexo.io/zh-cn/docs/commands)有一些了解，如果不需要做其他操作的话，你只需要一条指令即可：
```sh
$ Hexo d -g
```

### 目录结构
该工程的目录结构主要分如下几个部分：
```
根目录
|------ _config.yml 主工程的配置信息
|
|------ source 
|            |--— _post Markdown编写的博客文章，会被编译成H5文件    
|
|------ themes 主题
|            |
|            |--- hexo-thme-matery 当前所选用的主题
|                                 |
|                                 |--- _config.yml 当前主题配置信息
| 
|------ public 最终生成的可部署文件（执行 hexo d 生成）                               
|
|
|------ .deploy 通过配置根目录下的 _config.yml deploy后自动生成，用于自动部署到Github博客站点
|
```
### 依赖模块加载
进入当前工程根目录
```
$ yarn
or 
$ npm install
```

### 配置
Project的配置信息分两部分：
#### 1.工程配置请参考[Hexo 配置](https://hexo.io/zh-cn/docs/configuration.html)
#### 2.主题配置请参考[hexo-thme-matery](https://github.com/blinkfox/hexo-theme-matery)

### 部署
这里部署方案有两个：部署到个人的云服务器或者部署到Github的免费站点。

#### 云服务器部署
首先你得有个云服务器，具体流程请自行百度并购买。由于都是静态文件，我这边直接通过nginx进行部署。
Nginx部署流程：
##### 1. Nginx安装
```sh
$sudo apt-get install nginx
```
##### 2. Nginx配置
我这里的配置比较简单，资源文件放在当前用户下的`personal-blog`文件夹下面，所整个Nginx配置如下：
```sh
# 1.root权限进入到/etc/nginx/nginx.conf，新增
include /etc/nginx/sites-enabled/blog;

# 2. 进入到/etc/nginx/sites-availabl创建blog的service文件，主要设置root文件夹映射
root /home/ubuntu/personal-blog;

# 3. 进入到/etc/nginx/sites-enabled创建可用服务的软连接,
$ ln  -s /etc/nginx/sites-available/blog blog

#创建成功后的软连信息如下：
lrwxrwxrwx 1 root root 31 Mar  5 15:48 blog -> /etc/nginx/sites-available/blog

# 4. 启动nginx服务
$ sudo /etc/init.d/nginx start
```
#### 3. 资源文件上传
```sh 
scp -r ./public/*  yourServerName@129.204.226.xxx:~/personal-blog  # 注意personal-blog是否已经创建
```
#### 基于Github搭建个人博客
首先到你的个人Github上创建xxxx.github.io仓库（xxx你的Github用户名称，如 rongoli.github.io）
然后修改根目录下_config.yml的配置信息：
```
deploy:
  type: git
  repo: git@github.com:RongoLi/rongoli.github.io.git #修改成你自己的仓库信息，注意我这里是使用的SSH
  branch: master
```
修改好配置之后，就可以直接执行如下命令就可以发布了
```
$ hexo d -g
```

### 友情提示
该仓库我会一直同步博客Markdown源文件，所以你可以参考这部分源文件编写属于你自己的博客文章，在进行发布的的时候请删除掉我在_posts文件夹下的所有文件或者在文章头部加上转载信息，多谢配合！
```
本文章转载自[Rongo的个人博客](http://rongo.icu/)
```