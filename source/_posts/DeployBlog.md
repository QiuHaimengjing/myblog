---
title: 将我的blog部署在云服务器ECS上
date: 2023-04-15 15:33:22
tags: [Hexo]
categories: [技术]
thumbnail: "https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_1.jpg"
excerpt: "本篇文章用于记录我是如何将blog从github pages部署到云服务器ECS的"
---

# 序言

这次的网站部署工作还挺不容易的，是一次非常新奇的尝试，从此刻执笔写下这篇文章开始，我已经意识到这将是一个漫长的过程，也说明本篇的内容很长，对于我或是读者来说都是一段漫长的征途了。

![网站主页](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_2.png "网站主页")

早在一年前我就搭建好了我的 blog，并且购买并配置了域名，那时候我的 blog 一直都是在 GitHub Pages 上的，但是访问速度实在太慢，于是想到把 blog 部署到国内的服务器上，本篇文章记录的正是这个部署的过程。

# 服务器

那首先我得有一台国内的服务器，因此我购买了一台云服务器 ECS。

> 服务器的操作系统是**CentOS 7.9 64 位**

之后配置服务器的过程，因为不同的操作系统命令会有区别，还请读者根据自己的操作系统查询命令。

# 安装 MATE 桌面环境

**这个环节是非必要的，读者可跳过，这只是我熟悉服务器的一个小过程。**

1.执行以下命令，更新系统的软件包

```shell
yum -y upgrade
```

2.依次执行以下命令，安装 MATE 桌面环境  
**之后会出现一些提示，都让他通过就行**

```shell
yum groups install "X Window System"
yum groups install "MATE Desktop"
```

3.设置默认使用图形化桌面环境启动实例

```shell
systemctl set-default graphical.target
```

4.执行以下命令重启 ECS 实例，也可以在控制台手动重启

```shell
reboot
```

![控制台重启ECS](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_7.png "控制台重启ECS")

5.之后通过 ECS 管理控制台的 VNC 连接实例就可以进入到图形界面
![VNC连接ECS](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_8.png "VNC连接ECS")
![图形界面](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_9.png "图形界面")

更多的详细过程可参考阿里云的文档：  
[如何在 Linux 系统的 ECS 实例中安装图形界面](https://help.aliyun.com/document_detail/41227.html)

# 安装 Git

> Git 是分布式版本控制系统，有了它我们能很容易地进行主机与服务器的同步

1.首先查看服务器上是否有安装 Git

```shell
git --version
```

2.执行以下命令安装 Git

```shell
yum install git
```

之后碰到提示直接输入 y 通过。安装完成会出现**Complete!**

3.执行以下命令创建一个 Git 用户

```shell
useradd git
```

4.设置 Git 账户的密码

```shell
passwd git
```

![设置Git账户和密码](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_10.png "设置Git账户和密码")

# 配置 ssh

> 要想实现主机和服务器的 Git 连接，我们需要给到服务器我们主机的密钥

首先主机上要安装 Git，主机 Git 的安装过程在此略过......  
之前我一直都用着 Git，所以已经配置过 ssh，但为了温故而知新，咱们从头再来配置一遍。

## 生成 ssh

1.我们在**Desktop 右键选择 Git Bash Here**，然后输入命令，引号内为你的 Git 用户名

```shell
git config --global user.name '用户名'
```

2.输入邮箱

```shell
git config --global user.email '邮箱'
```

然后可以输入以下命令确认下账户

```shell
git config --list
```

3.输入以下命令生成 ssh，遇到暂停输入的情况就按下回车

```shell
ssh-keygen -t rsa -C "邮箱"
```

![配置ssh](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_11.png "配置ssh")
`之后可以在C:\Users\用户名\.ssh目录下看到ssh`

## 更新 Github 的 SSHkey

**因为这里我们的 ssh 变了，所以 Github 上的 ssh 也应该重新设置下**

登录 Github，点击个人头像中的 Settings，找到 SSH and GPG keys，删除原来的 SSH Keys，建立新的 SSH Keys

**在 ssh 目录下找到 id_rsa.pub**，拷贝其中内容到 GithubSSHkey 中
![Github的SSH keys](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_12.png "Github的SSH keys")

OK，之后我们在使用 Git 管理你的 Github 仓库时就不会出问题了。

## 将公钥给服务器

将公钥给到服务器，在 ssh 目录下右键选择 Git Bash Here，输入以下命令

```shell
ssh-copy-id -i id_rsa.pub git@服务器IP地址
```

> 注：服务器 IP 地址为公网 IP 地址

![将公钥给服务器](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_13.png "将公钥给服务器")

好的，到这里我们的 ssh 全部完成，让我们回到服务器上。

# Nginx

> Nginx 是一个高性能的 HTTP 和反向代理服务器，我选择使用 Nginx 来作为 web 服务器。

## 安装 Nginx

1.执行以下命令安装 Nginx，版本我选择了 1.20.2

```shell
wget http://nginx.org/download/nginx-1.20.2.tar.gz
```

2.安装依赖

```shell
yum -y install gcc pcre-devel zlib-devel openssl openssl-devel
```

3.上一步完成后，解压依赖

```shell
tar -zxvf nginx-1.20.2.tar.gz
```

4.解压后进行配置，依次输入以下命令

```shell
cd nginx-1.20.2
./configure
```

![配置Nginx](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_14.png "配置Nginx")
之后再依次输入以下命令

```shell
make
make install
```

到此 Nginx 就安装好了。

## 运行 Nginx 及欢迎页面问题

进入到 nginx 文件夹下的 sbin 目录启动 nginx，依次执行以下命令

```shell
cd /usr/local/nginx/sbin
./nginx
```

![运行Nginx](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_15.png "运行Nginx")

之后在浏览器输入服务器的公网 IP 地址，就会出现 Nginx 的欢迎页面

![Nginx欢迎页面](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_16.png "Nginx欢迎页面")

也有可能出现 CentOS 的欢迎页面

![CentOS欢迎页面](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_17.png "CentOS欢迎页面")

**关于出现 Nginx 或 CentOS 页面的问题，我当时在这卡了一段时间，因为我格式化过服务器，第一次安装 Nginx 时出现的是 Nginx 的欢迎页面，第二次安装 Nginx 就出现了 CentOS 的欢迎页面，我就以为 Nginx 没安装成功，但是查看 Nginx 的确实在运行，于是找了下问题所在：**

> 检查了阿里云的安全组策略，Nginx 的安装步骤，都没有发现问题，安装 Nginx 时阅读 nginx.conf 配置文件会发现欢迎页 index.html 文件路径。找到上面路径下的 html 文件，通过阅读发现这就是 CentOS 欢迎页面显示的内容，这证明安装 Nginx 的欢迎页已经不是 Nginx 欢迎页面了，所以我们的 Nginx 安装是完全正确的，只是显示页面改变了。

## Nginx 页面无法访问和服务器防火墙问题

在面对防火墙之前，我们找到服务器的网络安全组，看一看有没有开放**80 端口**，没有的话要添加一个。  
![开放80端口](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_18.png "开放80端口")

然后我们回到服务器。

1.查看防火墙状态

```shell
firewall-cmd --state
```

如果没有运行，执行以下命令运行起来

```shell
systemctl start firewalld.service
```

再次查看防火墙的状态会显示 `running`

2.依次执行以下命令，手动开放 80 端口

```shell
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
firewall-cmd --permanent --add-port=80/tcp
```

出现`success`  
至此问题解决，可以正常访问 Nginx 欢迎页面。

# 创建 blog 仓库和部署

到这一步可以说是万事俱备，只欠东风了，现在我们需要进行：

1. 新建仓库用来存放网站的内容
2. 提交后把内容自动同步到站点目录

## 创建仓库

1.依次执行以下命令进入 git 目录，新建一个仓库

```shell
cd /home/git
git init --bare blog.git
```

2.进入 hooks 文件夹

```shell
cd blog.git/
cd hooks/
```

![创建仓库](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_19.png "创建仓库")

## hooks

**Git 钩子：Git 钩子是每次在 Git 存储库中发生特定事件时自动运行的脚本。它们允许您自定义 Git 的内部行为，并在开发生命周期的关键点触发可自定义的操作。**

- Git 钩子(hooks)是在 Git 仓库中特定事件(certain points)触发后被调用的脚本。
- 通过钩子可以自定义 Git 内部的相关（如 git push）行为，关键点（如 git push）触发自定义的行为。
- Git 含有两种类型的钩子：客户端的和服务器端。
- 客户端钩子由诸如提交和合并这样的操作所调用，而服务器端钩子作用于接收部署提交的代码。实现服务器和本地的 git 互通。

Git 钩子存在于每个 Git 仓库的 .git/hooks 目录中。
![Git 钩子](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_20.png "Git 钩子")

1.编写 post-receive 脚本

```shell
vi post-receive
```

输入 i 进入 INSERT 模式，内容如下

```
#!/bin/bash
#nginx下html文件夹目录
DIR=/usr/local/naginx/html
git --work-tree=${DIR} clean -fd
#直接强制检出
git --work-tree=${DIR} checkout --force
```

写好后 ESC 退出 INSERT 模式，:wq 保存退出

2.授予运行权限

```shell
chmod +x post-receive
```

3.授予 git 用户

```shell
chown -R git post-receive
```

4.给到一个读写的最高权限

```shell
chmod 777 post-receive
```

![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_21.png)

5.回到 git 目录下，给仓库同样的操作

```shell
cd /home/git
chmod 777 blog.git/
chown -R git blog.git/
```

![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_22.png)

6.被同步的目录也需要授予最高权限和 git 用户

```shell
cd /usr/local/nginx
chmod 777 html/
chown -R git html/
```

![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_23.png)

## hexo 配置

打开 hexo 的主配置文件，添加 deploy 仓库

```yml
type: git
repo: git@服务器IP地址:/home/git/blog.git
branch: master
```

![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_24.png)

## 同步到服务器

在本地 blog 文件夹下 Git Bash Here，执行以下命令

```shell
hexo clean
hexo deploy
```

完成后，输入服务器的 IP 地址就可以访问到网站了。

可以检查下服务器上是否有我们的博客文件

```shell
cd /usr/local/nginx/html
ls
```

检查无误就大功告成了。

# 域名解析

这一步是让我的域名绑定服务器，绑定之后就能以域名访问网站了。这一步很简单，只需要添加域名解析就行。

**这里添加两个记录：www 和@，记录值都是服务器的 IP 地址**
![域名解析](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_25.png "域名解析")

# SSL 证书

> SSL 证书就是遵守 SSL 安全套接层协议的服务器数字证书。而 SSL 安全协议最初是由美国网景 Netscape Communication 公司设计开发的,全称为:安全套接层协议 (Secure Sockets Layer) , 它指定了在应用程序协议(如 HTTP、Telnet、FTP)和 TCP/IP 之间提供数据安全性 分层的机制,它是在传输通信协议(TCP/IP)上实现的一种安全协议,采用公开密钥技术,它为 TCP/IP 连接提供数据加密、服务器认证、消息完整性以及可选的客户机认证。由于此协议很好地解决了互联网明文传输的不安全问题,很快得到了业界的支持,并已经成为国际标准。SSL 证书由浏览器中“受信任的根证书颁发机构”在验证服务器身份后颁发,实现网站身份验证和加密传输功能。

装载 SSL 证书产品后自动激活浏览器中显示“锁”型安全标志，地址栏以“https”开头。

## 获取 SSL 证书

获取的方式挺多，可以购买，也可以免费获取，我的证书是领的阿里云免费给的 20 张证书。
![免费的SSL证书](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_26.png "免费的SSL证书")

## 下载 SSL 证书

点击证书栏右侧“下载”，找到服务器类型 Nginx 下载。
![下载SSL证书](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_27.png "下载SSL证书")

## 传输到服务器

下载后的证书是一个压缩包，解压后会有两个文件：**.key 和.pem**

这里可以解压后传输到服务器，也可以把压缩包直接传输到服务器，但是需要在服务器上解压，所以服务器需要安装 ZIP 解压软件。这里我选择先解压再传输到服务器。

安装 unzip

```shell
yum install unzip
```

1.在 Nginx 根目录下 conf 文件夹下创建存放证书的目录 cert

```shell
cd /usr/local/nginx/conf
mkdir cert
```

这里我选择先解压再传输到服务器。

2.在 ECS 控制台发送.key 和.pem 文件，目标路径为

```shell
/usr/local/nginx/conf/cert/
```

![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_29.png)

上传成功后，进入 cert 文件夹可以看到存在这两个文件了
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_30.png)

## 修改 server

1..返回 conf 文件夹编辑 Nginx 配置文件 nginx.conf，修改与证书相关的配置，目的是打开 443 端口。

```shell
vim nginx.conf
```

![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_28.png)

2.找到 HTTPS server，**将其内容解注释并修改**。

以下步骤含错误示范，还请读者不要着急模仿，可以先去下面看一眼“**重启失败解决方案**”的内容，方便之后一步到位。

当然也可以跟着我的步骤来，之后修改错误。

原来的 HTTPS server 内容

```conf
   # HTTPS server
   #
   # server {
   #     listen       443 ssl;
   #     server_name  localhost;


   #     ssl_certificate      cert.pem;
   #     ssl_certificate_key  cert.key;

   #     ssl_session_cache    shared:SSL:1m;
   #     ssl_session_timeout  5m;


   #     ssl_ciphers HIGH:!aNULL:!MD5;
   #     ssl_prefer_server_ciphers  on;

   #     location / {
   #         root   html;
   #         index  index.html index.htm;
   #     }
   #}
```

修改后的 HTTPS server

```conf
    HTTPS server

    server {
        #HTTPS的默认访问端口为443
        #如果未在此处配置HTTPS的默认访问端口，可能会造成Nginx无法启动
        #如果使用Nginx 1.15.0及以上版本，请使用listen 443 ssl 代替listen 443和ssl on
        listen       443 ssl;
        #填写证书绑定的域名
        server_name  www.invictusqiu.com;
        root html;
        index index.html index.htm;

        #填写证书文件名称
        ssl_certificate      cert/9575407_www.invictusqiu.com.pem;
        ssl_certificate_key  cert/9575407_www.invictusqiu.com.key;

        ssl_session_timeout  5m;

        #表示使用的加密套件的类型
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        #表示使用的TLS协议的类型，需要自行评估是否配置TLSv1.1协议
        ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;

        ssl_prefer_server_ciphers  on;

        location / {
        #Web网站程序存放目录
            root   html;
            index  index.html index.htm;
        }
    }
```

3.修改 80 端口 server 的内容

原来 80 端口的 server

```conf
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        #后面内容可忽略
    }
```

修改后的 80 端口 server

```conf
    server {
        listen       80;
        #填写证书绑定的域名
        server_name  www.invictusqiu.com;
        #将所有HTTP请求通过rewrite指令重定向到HTTPS。
        rewrite ^(.*)$ https://$host$1;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        #后面内容可忽略
    }
```

服务器界面展示：
![443端口原来的server](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_31.png "443端口原来的server")
![443端口现在的server](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_32.png "443端口现在的server")
![80端口原来的server](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_33.png "80端口原来的server")
![80端口现在的server](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_34.png "80端口现在的server")

**修改完成后保存退出**

## 重启 Nginx 服务

来到 nginx 的 sbin 目录执行重启命令

```shell
cd /usr/local/nginx/sbin
./nginx -s reload
```

发现报错：
![重启错误](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_35.png "重启错误")

## 重启失败解决方案

好的，不论是跳转来这一步的朋友，还是跟着我步骤的朋友，**现在看一看 nginx.conf 文件 443 端口那里的 HTTPS server 是不是注释掉的。**

也就是如下

```conf
   # HTTPS server

    server {
        listen       443 ssl;
        server_name  localhost;
```

**还没修改 server 的朋友请注意不要将 HTTPS server 解注释。**

**已经跟着我走的朋友请回去将其注释掉**。

之后再次重启 Nginx 服务就成功了。

## 重启失败解决方案第二版

为什么有第二版解决方案呢，这版解决方案是针对：  
**由于我们未装 SSL 模块，启动时，会提示 nginx:[emerg]unknown directive ssl 错误**

因为我是先遇到第一版错误，不知道错哪了，查资料时干脆把这一版的解决方案先做了，到最后发现我并未存在此版错误，但我认为还是有必要提一下的。

**如果未做该版解决方案的朋友你跟着我前面的步骤很顺利的重启了，那么你可以跳过此小节到“放行 443 端口”。**

好的，让我们看看这一错误该怎么解决。

先执行`cd ~`

1.检查你是否安装了 ssl 模块

```shell
cd /usr/local/nginx
./sbin/nginx -V
```

如下图所示显示已经安装 ssl 模块则证明你不存在此版错误。
![存在ssl模块](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_36.png "存在ssl模块")

2.如果没有 ssl 模块，我们先来到 Nginx 的解压目录，跟着我的步骤走的朋友路径如下，其他的朋友可能你的解压目录在/usr/local/nginx-1.20.2

```shell
cd ~
cd /root/nginx-1.20.2
```

![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_37.png)

3.添加 ssl 模块

```shell
./configure --with-http_ssl_module
```

4.执行 make 命令，编译安装包

```shell
make
```

5.查看 objs 文件夹下有一个 nginx 文件，这个就是新版程序，然后备份下之前的 nginx

```shell
cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak
```

6.把编译好的 nginx 文件替换之前的

```shell
cp objs/nginx /usr/local/nginx/sbin/nginx
```

如果无法替换，显示  
`cannot create regular file '/usr/local/nginx/sbin/nginx': Text file busy`

执行以下命令查看 nginx 进程

```shell
ps -ef | grep nginx
```

发现正在运行

查看进程号，执行以下命令关闭 nginx 进程

```shell
kill -QUIT 4809
```

再次查看 nginx 进程，可以看到已经关闭，之后再次执行上面的替换命令，就能成功替换了。

![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_38.png)

7.最后查看下是否安装成功了

```shell
cd /usr/local/nginx
./sbin/nginx -V
```

显示有 ssl 模块，那么就证明我们安装成功了，之后就能正常重启 nginx 了。

## 放行 443 端口

1.执行以下命令查看 443 端口是否在运行

```shell
netstat -nplt lgrep 443
```

可以看到正在运行
![443端口正常运行](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_39.png "443端口正常运行")

2.添加防火墙端口

查看防火墙运行状态

```shell
firewall-cmd --state
```

显示`running`，则进入下一步。

查看开放的端口

```shell
firewall-cmd --list-ports
```

发现没有 443 端口。

添加 443 端口

```shell
firewall-cmd --zone=public --add-port=443/tcp --permanent
```

显示`success`

3.重新加载防火墙

```shell
firewall-cmd --reload
```

显示`success`

再次查看开放的端口

```shell
firewall-cmd --list-ports
```

可以看到存在 443 端口。

![添加防火墙端口](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_40.png "添加防火墙端口")

4.检查本地 443 端口加载的 HTTPS 服务以及证书是否正常，域名名称为你自己的域名

```shell
echo | openssl s_client -connect 127.0.0.1:443 -servername invictusqiu.com 2>/dev/null
```

结果出现`SSL-Session`表示 HTTPS 服务正常运行。

![SSL-Session](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_41.png "SSL-Session")

5.进入 ECS 安全组策略
![进入ECS安全组策略](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_42.png "进入ECS安全组策略")

放行 TCP 协议 443 端口的入方向请求
![放行443端口](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_43.png "放行443端口")

6.执行 curl 命令，查询服务器响应 header 信息

```shell
curl -l https://invictusqiu.com
```

结果显示 HTTPS 请求可以正常响应。
![响应header信息](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_44.png "响应header信息")

**至此所有工作都完成了！**

# 结尾

此番部署工作可谓是困难重重，在此途中卡了好多次，甚至一个重启 nginx 的问题都解决了好久，关于我是怎么知道 HTTPS server 需要注释掉的呢？在网上翻来覆去搞了半天，看到这个视频：
![](https://invictusqiu.oss-cn-beijing.aliyuncs.com/blog/DeployBlog/DeployBlog_45.png)

哈哈哈，找个错误 CPU 都干烧了，结果是这种错误。  
这种 Error 可谓是读计算机专业家常便饭的事了。

最后分享一件事作为本篇文章的结尾：  
&emsp;&emsp;我一直很钦佩纯粹的人，特别是以具体的目光投射到他身上时也依然纯粹的人，但这样的人只是我心目中的理想化，我也只是追寻着“这类不存在的人”的脚步前进罢了。几天前跟朋友聊天，聊到了跟几年前的自己做对比。假如需要你去做一件短时间内几乎不可能完成的事，以至于需要“燃尽”自己，你还会去做吗？我不可否认，在塑造世界观的那个年纪，现实、电影、小说、电子游戏等都或多或少影响到过我的世界观。  
&emsp;&emsp;在几年前如果你问我这个问题，我的回答甚至偏向于“会”，但是现在我想还是算了罢，我想这就是现在的自己和几年前自己一个最大的对比。  
&emsp;&emsp;我并不是否认“会”这个选项，只是它**现在**不是最优解了。所以我为什么钦佩纯粹的人，是因为他们从不会因为“时态”而去改变自身的选择，这也是我为什么仅仅只是追随他们而不是成为他们的原因吧。

> 本章一句：  
> 以我残躯化烈火。—— Cyberpunk 2077 隐藏结局
