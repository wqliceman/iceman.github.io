---
layout:     post                    	# 使用的布局（不需要改）
title:      nginx初探               # 标题 
subtitle:   一起学习Nginx 	#副标题
date:       2018-06-06              # 时间
author:     iceman                      # 作者
header-img: img/post-bg-nginx.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - nginx
    - 网络编程
---

## 什么是Nginx

Nginx（读 engine X）是一个使用C语言实现的跨平台的Web服务器，可运行在多操作系统平台上，并且可使用当前操作系统的一些高效API来提升自己的性能。同时，Nginx基于事件驱动的架构实现，能处理百万级的TCP连接，并且高度模块化，这样就使其形成了一个平台，基于此开发了许多第三方模块。

## 为什么选择Nginx

选择nginx的原因主要在于以下几个特点：

- 高性能：Nginx完全使用C语言进行开发，采用事件驱动模型，对操作性进行了特别优化，可无阻塞处理海量并发
- 高扩展：基于Nginx使用模块化的架构设计，其内部是由各个不同层级、同功能、不同类型的功能模块组合而成，对于模块的修改或升级，都可以专注于本模块，无需关注其他模块的耦合。同时其他开发者可以开发新的模块，Nginx框架允许其充分利用其提供的各种高效机制，使其完美融合到Nginx现有框架中。这就造就了庞大的第三方模块，满足日常需求，同时自己也可以按照自己业务逻辑进一步开发。我还看中了一个，目前Nginx还有一个良好的lua（后期对该脚步进行介绍）脚本语言开发环境，代表有：Openresty
- 低内存消耗：由于代码的质量高，同时没有使用像传统的多线程编程，这样就没有了进程和线程切换成本，资料查得一般情况下，10000个非活跃的HTTP Keep-Alive连接在Nginx中仅消耗2.5MB的内存
- 高可靠：Nginx内部使用内存池分配资源，避免了C常见的资源泄漏问题；同时模块化的设计使得各个功能完全解耦合；另一重大优势在于使用**one master/multi workers**进程池设计原则，保证工作进程的任何错误不影响整个系统的运行，同时**master**可以快速启动新的**worker**来提供稳定服务

## 安装

对于前期我们作为使用，并不进行模块开发，可以直接进行安装编译好的版本

### Windows安装

可直接到[官网](https://nginx.org/en/download.html)进行相应版本下载，解压即可

### Mac安装

- 方法1：可以用brew很方便地安装Nginx

```bash
sudo brew install nginx
```

- 方法2：通过源码编译进行安装

到官网下载`.tar.gz`包，解压之后进入目录，运行 

```bash
./configure
sudo make
sudo make install
```



## 运行命令

本文基于mac系统进行运行，各平台除路径不一定相同外，其他操作均一致

### 启动和停止服务

#### 使用默认配置启动

```bash
/usr/local/nginx/sbin/nginx #使用默认的配置文件启动Nginx
```


#### 启动指定配置选项
> **-c** 参数指定配置文件来启动Nginx
```bash
/usr/local/nginx/sbin/nginx -c my.conf
```

#### 指定工作目录启动
> **-p** 用来设置工作目录，参数后接路径
```bash
/usr/local/nginx/sbin/nginx -p ~/iceman/nginx
```

### 发送信号命令
> **-s** 可以快速停止或重启Nginx，参数后接信号，可以是：*stop，quit，reload，reopen*
```bash
/usr/local/nginx/sbin/nginx -s stop		#立即停止Nginx服务
/usr/local/nginx/sbin/nginx -s quit		#处理完成当前连接后即停止
/usr/local/nginx/sbin/nginx -s reload	#重启Nginx，重新加载配置文件
/usr/local/nginx/sbin/nginx -s reopen	#重新打开日志文件
```

### 测试配置信息是否粗五
> **-t** 测试配置文件是否正确
```bash
/usr/local/nginx/sbin/nginx -t			#检查默认配置文件
/usr/local/nginx/sbin/nginx -t my.conf	#检查my.conf配置文件
```

### 显示版本信息
> **-v** 显示Nginx的版本信息，-V显示完全版本信息
```bash
/usr/local/nginx/sbin/nginx -v
#nginx version: nginx/1.12.0
/usr/local/nginx/sbin/nginx -V
#此处除了显示版本信息，还会显示编译相关信息等
```

## 验证安装

如果已经启动了Nginx服务器，可以通过浏览器网页进行访问进行测试，也可以通过命令行工具curl进行测试。如此时默认配置开启了**localhost:80**服务，

### 浏览器访问
浏览器输入地址http://127.0.0.1:8914/
![nginx_chrome_test](http://ww1.sinaimg.cn/large/665db722gy1fs0qp5jeh7j20ap04raa3.jpg)

### curl访问

```bashe
iceman>curl http://127.0.0.1:8914/
<p>Hello, World!</p>
```

### 查看nginx进程信息

在类Unix系统下可使用如下命令

```bash
ps aux | grep nginx

➜  ngigx_project ps aux | grep nginx
nobody           50935   0.0  0.0  4288632   1100   ??  S     7:36AM   0:00.00 nginx: worker process
root             50934   0.0  0.0  4279956    520   ??  Ss    7:36AM   0:00.00 nginx: master process /usr/local/nginx/sbin/nginx
```

从ps的输出我么可以看到当前两个Nginx进程，其中进程号为50934的是master进程，50935的为worker进程



## 配置Nginx

### 配置文件介绍

Nginx的配置文件（**nginx.conf**）决定了Nginx的进程数量、日志信息、反向代理、请求处理等多方面的信息。启动的时候加载配置文件，通过配置信息调用不同的模块和运行参数

```bash
worker_processes  auto;
master_process on;
daemon	on;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    server {
        listen       8914;
        server_name  localhost;

        location / {
            root   html;
            index  index.html index.htm;
			default_type text/html;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

}
```



### 进程配置

以下三个进程配置只能在全局域中进行配置：

```bash
worker_processes number | auto;
```

用于设置工作进程的个数，通常设置为当期CPU核心数，这样能达到性能最佳，以让工作进程和CPU核心一一对应上，默认值为1，如果不清楚当前服务器的核心数量，可以设置为**auto**，Nginx会自动进行获取并设置

```bash
master_process on | off
```

是否启动Nginx的进程池机制，默认为**on**。如果设置为**off**，那么Nginx就不会创建master进程，只会用默认的一个工作进程进行处理，同时上面设置的**worker_processes**也会无效，并发处理能力就会大打折扣，此处可以在调试的时候进行设置为**off**，其他场景建议不要使用

```bash
daemon on| off
```

是否以守护进程的方式进行运行，默认为**on**，多数情况是运行在后台。当我们需要进行调试或需要获取调试信息的时候可禁用，方便再控制台进行日志输出



### events配置

配置影响Nginx服务器或与用户的网络连接。每个进程的最大连接数、选取哪种事件驱动模型处理连接请求、是否允许同时接受多个网路连接和开启多个网络连接序列化等 

```bash
events {
    accept_mutex on;   	#设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  	#设置一个进程是否同时接受多个网络连接，默认为off
    use epoll;      	#事件驱动模型，select|poll|kqueue|epoll|resig|...
    worker_connections  1024;    #最大连接数，默认为512
}
```



### http配置

可以嵌套多个server、配置代理、缓存、日志定义等，绝大多数功能和第三方模块的配置。

Nginx几乎90%的功能都是提供http服务，所以http块的配置也是异常复杂，几乎整个配置文件都是它的领地。一般模式如下：

```bash
http      #http块,所有HTTP相关功能
{
    ...   #http全局块
    upstream{   #upsteam块，配置上游服务器
        ....
    }
    server        #server块， 第一个虚拟主机
    { 
        ...       #server全局块
        location [PATTERN]   #location块
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     #http全局块
}
```

一般http块的内容较多，可以通过include引入子配置(将server、location等配置分离成单独文件)，这样可以降低配置复杂度

```bash
http      #http块,所有HTTP相关功能
{
    include common.conf #基本的HTTP配置文件，一般配置通用的参数
    include upstream.conf #配置上游服务器
    include servers/*.conf #所有服务器配置文件
}
```



### server配置

在http块中使用server指令定义一个虚拟主机，通过指令来确定域名、端口等参数

```bash
server {
	keepalive_timeout120; #设置keepalive的超时时间，默认75s
    listen 8080; # port 端口
    server_name localhost; #host名称
}
```

- 通过**keepalive_timeout**设置keepalive的超时时间，通常用于客户端复用HTTP的长链接，提高服务器的性能，如果希望发送完后就主动断开，那么就设置为0即可
- 通过**listen**来设置虚拟主机监听端口，默认80
- 通过**server_name**设置虚拟主机对外的主机名称



### location配置

location相当于虚拟主机的虚拟目录，Nginx再成功陪陪虚拟主机后，会继续查找虚拟目录(location块)以确定请求如何处理

```bash
server{
    listen 8080;
    server_name 127.0.0.1;
    location /image/ {  #匹配/image/*.*
        root img;	#设置根目录
        index 001.jpg #设置默认图片
    }
}
```

location 使用配置文件里的uri参数匹配HTTP请求行里的URI， 默认是前缀匹配，也支持正则表达式。 使用几个前缀来做进一步的匹配限定： 

- =： URI必须完全匹配 
- ~： 大小写敏感匹配
- ~*： 大小写不敏感匹配 
- ^~： 匹配前半部分即可
- @： 用于内部子请求，外部无法访问

### upstream配置

该配置主要用在反向代理时访问服务器集群和负载均衡策略，结合server节点下的location中的proxy_pass配置。其配置方式如下：

```bash
upstream server_end {
	# 默认使用轮询进行负载均衡策略
    server	192.168.0.10:8080;
    server	192.168.0.11:8080;
    server	192.168.0.12:80;
}

server{
    location / { 
     	proxy_pass http://server_end; 
	}
}
```

对于负载均衡除了默认的轮询还有其他几种方式进行分配：

- 轮询（默认）分配
- weight(加权轮询)分配：通过指定权重进行比率分配，当服务器性能参差不齐时进行几率轮询

```bash
upstream server_end {
	# 使用权重进行分配，访问192.168.0.12的几率是192.168.0.10的二倍
    server	192.168.0.10:8080 weight = 1;
    server	192.168.0.11:8080 weight = 1;
    server	192.168.0.12:80 weight = 2;
}
```

- ip_hash（通过请求访问ip的hash结果进行映射）分配：每一个客户端固定访问一个后端服务器，这样就可以确保session的问题

```bash
upstream server_end {
	# 由访问ip的hash值来决定访问的服务器
	ip_hash;
    server	192.168.0.10:8080;
    server	192.168.0.11:8080 ;
    server	192.168.0.12:80;
}
```

- backup 热备：如果有两台服务器，当一台服务器发生故障时， 才启动第二台服务器提供服务

  ```bash
  upstream server_end {
      server	192.168.0.10:8080;
      server	192.168.0.11:8080 backup; #热备
  }
  ```

关于nginx负载均衡配置的几个状态参数讲解

  - down，表示当前的server暂时不参与负载均衡。
  - backup，预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的压力最轻。
  - max_fails，允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误。
  - fail_timeout，在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用。

```bash
upstream server_end {
    server	192.168.0.10:8080 weight=2 max_fails=2 fail_timeout=2;
    server	192.168.0.11:8080 weight=1 max_fails=2 fail_timeout=2
}
```



欢迎关注交流共同进步
![奔跑阿甘](http://ww1.sinaimg.cn/large/665db722gy1frf76owwqjj2076076q3e.jpg)