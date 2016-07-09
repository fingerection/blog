---
layout: post
title: 内网服务器端口被外网访问的几种方法
---
## 需求
在局域网内部的主机端口需要被外网访问。直接访问内网IP肯定不行，那就想办法把内网IP和端口号映射到外网的IP和端口号。

![](68747470733a2f2f6e67726f6b2e636f6d2f7374617469632f696d672f6f766572766965772e706e67.png "图片来源：ngrok github主页")
图片来源：ngrok官方github

简单来说，有几种方法可以映射：
1. 路由器端口转发 （需要路由器管理权限）
2. ssh 反向隧道 （需要公网服务器）
3. ngrok 服务器转发 （需要公网服务器或注册ngrok会员）

## 一句话介绍如何选择方案
如果你有路由器管理权限就在路由器上配，最省事简单。
如果你有自己的公网服务器，不想装其他软件就用，用ssh反向隧道，如果想加点管理功能，以后用着方便，用ngrok自建服务器。
如果你没有自己服务器，可以翻墙，就用ngrok注册用户服务。
如果你没有自己服务器，不能翻墙，试试在国内搜索ngrok的服务，有一些开放的服务器。
## 路由端口转发
### 原理
路由器具有公网的地址，访问路由器公网地址的端口，路由器把端口转发到内网服务器的某个端口。
### 配置
随路由器种类不同而不同。
### 优缺点
优点：内网服务器无需配置
缺点：需要路由器的管理权限，并且知道路由器的公网IP地址
## ssh 反向隧道
### 原理
ssh tunnel有两种，转发本地端口到远端，转发远端端口到本地。这里用的是后者，前者常用于翻墙。
### 配置
在内网机器`~/.ssh/config`文件中配置ssh连接方式。

     Host cli-ssh-tunnel
        HostName      myserver.com
        User          ubuntu
        Port          22
        IdentityFile  ~/.ssh/tunnel.pem
        RemoteForward 2222 localhost:22
        ServerAliveInterval 30
        ServerAliveCountMax 3

RemoteForward 第一个参数是远端端口号，第二个参数是映射到本地的端口号。运行命令：

    ssh cli-ssh-tunnel

用autos做自动ssh重连(autossh介绍配置：[链接](https://www.everythingcli.org/ssh-tunnelling-for-fun-and-profit-autossh/))

    autossh -M 0 -f -T -N cli-ssh-tunnel

在外网服务器`/etc/ssh/sshd_config`配置GatewayPorts选项
    GatewayPorts yes
这样才能绑定到外网的公网IP，否则会一直绑定到127.0.0.1的地址。

### 优缺点
优点：无需路由权限，无需安装第三方软件，方便快捷的临时解决方案
缺点：管理功能少，配置多个比较复杂。

## ngrok 服务器转发
### 原理
和ssh服务器转发原理类似，只是增加了方便的API命令行接口和一系列管理功能。
### 配置
#### ngrok官方服务器
如果是翻墙用户可以直接注册ngrok账户就获得了ngrok提供的服务器。
在客户端下载ngrok官方的客户端就可以完成端口映射。（客户端需要翻墙能力）
#### 自建ngrok服务器
如果是墙内用户，可以自己搭建私有ngrok服务器。编译ngrok的源码生成服务器端和客户端的可执行文件。
来源：[搭建 ngrok 服务实现内网穿透](https://imququ.com/post/self-hosted-ngrokd.html) (有修改）

    sudo apt-get install build-essential golang mercurial git

    git clone https://github.com/inconshreveable/ngrok.git ngrok
生成证书，TLS加密通信用的。这里的域名一定要写对，如果是二级域名就写二级域名
    NGROK_DOMAIN="imququ.com"
    openssl genrsa -out base.key 2048
    openssl req -new -x509 -nodes -key base.key -days 10000 -subj "/CN=$NGROK_DOMAIN" -out base.pem
    openssl genrsa -out server.key 2048
    openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csropenssl x509 -req -in server.csr -CA base.pem -CAkey base.key -CAcreateserial -days 10000 -out server.crt

需要用自己的证书替换ngrok的证书

    cp base.pem ngrok/assets/client/tls/ngrokroot.crt

编译

    sudo make release-server release-client

如果一切正常，ngrok/bin 目录下应该有 ngrok、ngrokd 两个可执行文件。

    sudo ./bin/ngrokd -tlsKey=server.key -tlsCrt=server.crt -domain="imququ.com" -httpAddr=":8081" -httpsAddr=":8082"

到这一步，ngrok 服务已经跑起来了，可以通过屏幕上显示的日志查看更多信息。httpAddr、httpsAddr 分别是 ngrok 用来转发 http、https 服务的端口，可以随意指定。ngrokd 还会开一个 4443 端口用来跟客户端通讯（可通过 -tunnelAddr=":xxx" 指定），如果你配置了 iptables 规则，需要放行这三个端口上的 TCP 协议。

#### 自建ngrok客户端
如果要把 linux 上的服务映射出去，客户端就是前面生成的 ngrok 文件。但我用的是 Mac，需要指定环境变量再编一次：
    sudo GOOS=darwin GOARCH=amd64 make release-server release-client
这样在 ngrok/bin 目录下会多出来一个 `darwin\_amd64` 目录，这里的 ngrok 文件就可以拷到 Mac 系统用了。
写一个简单的配置文件，随意命名如 ngrok.cfg：
    server_addr: imququ.com:4443
    trust_host_root_certs: false
指定子域、要转发的协议和端口，以及配置文件，运行客户端：
    ./ngrok -proto=http -config=ngrok.cfg 80
不出意外可以看到这样的界面，这说明已经成功连上远端服务了
#### ngrok 自建的域名问题
1. http方式转发时，外网地址用多级域名进行区分(比如ngrok.example.com会分配your service.ngrok.example.com的地址，需要在DNS提供商处修改，支持三级域名泛匹配 `*.ngrok.example.com` -\> IP
### 优缺点
优点：使用方便，有管理功能，安全性高（tls加密）
缺点：需要额外安装第三方软件


