几乎所有的项目部署，都会用到Nginx。

主要用到Nginx的功能有如下：

- 正向、反向代理
- 过滤IP
- 负载均衡

这三个功能是大部分公司的线上项目部署用到的，掌握Nginx并不难，主要是学会配置`nginx.conf` 文件，这个文件是用到了一些Nginx的语法，会用简单，但是用好却很难。

下面来简单介绍一下Nginx。

## 1、Nginx介绍

Nginx下载地址：http://nginx.org

Nginx 是开源、高性能、高可靠的 Web 和反向代理服务器，而且支持热部署，几乎可以做到 7 * 24 小时不间断运行，即使运行几个月也不需要重新启动，还能在不间断服务的情况下对软件版本进行热更新。



如果你是前端，相信你用过Node作为服务器。

如果你是后端，相信你用过Tomcat作为服务器。

Nginx也作为一个轻量级的服务器，其功能与Node、Tomcat是不冲突的。

Nginx的优势就是其性能，其占用内存少、并发能力强、能支持高达 5w 个并发连接数。

所以Nginx一般被用来当做服务器的第一道门，其应用场景有：

- 正向、反向代理
- 过滤IP，白名单
- 负载均衡
- 静态资源服务

## 2、Nginx的一些概念

在学习Nginx之前，先要了解一下下面几个概念，做简单了解即可。

### 2.1、CORS

CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing），也就是我们常说的**跨域**。

在浏览器上当前访问的网站向另一个网站发送请求获取数据的过程就是**跨域请求**。因为浏览器有安全策略，不然随便访问其他网站资源很可能会造成安全隐患，而CORS就可以打破这个限制。

它允许浏览器向跨源服务器发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制。

CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。

整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。

浏览器将CORS请求分成两类：`简单请求（simple request）`和`非简单请求（not-so-simple request`）。

> 简单的说，浏览器在发送跨域请求的时候，会先判断下是简单请求还是非简单请求，如果是简单请求，就先执行服务端程序，然后浏览器才会判断是否跨域。

以下是一些跨域的例子：

```bash
# 不同源的例子
http://example.com/app1   # 协议不同
https://example.com/app2

http://example.com        # host 不同
http://www.example.com
http://myapp.example.com

http://example.com        # 端口不同
http://example.com:8080
```



### 2.2、两种请求

#### 2.2.1、简单请求

1. 请求方法是 `HEAD`、`GET`、`POST` 三种之一；
2. HTTP 头信息不超过这几个字段：`Accept`、`Accept-Language`、`Content-Language`、`Last-Event-ID` `Content-Type` 
3. 而且`Content-Type` 只限于三个值 `application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`；

如果**同时满足上面三个条件**的，就属于**简单请求**。

凡是**不同时满足上面三个条件**的，就属于**非简单请求**。

对于简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息之中，增加一个`Origin`字段。

:airplane:

下面是一个例子，浏览器发现这次跨源AJAX请求是简单请求，就自动在头信息之中，添加一个`Origin`字段。

> GET /cors HTTP/1.1
> Origin: http://rain.baimuxym.cn 
> Host: api.alice.com
> Accept-Language: en-US
> Connection: keep-alive
> User-Agent: Mozilla/5.0…

上面的头信息中，`Origin`字段用来说明，本次请求来自哪个源（协议 + 域名 + 端口）。服务器根据这个值，决定是否同意这次请求，Host是目前的域名。

如果`Origin`指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含`Access-Control-Allow-Origin`字段（详见下文），就知道出错了，从而抛出一个错误，被`XMLHttpRequest`的`onerror`回调函数捕获。注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。

如果`Origin`指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。

> Access-Control-Allow-Origin: http://rain.baimuxym.cn
> Access-Control-Allow-Credentials: true
> Access-Control-Expose-Headers: FooBar
> Content-Type: text/html; charset=utf-8

#### 2.2.2、非简单请求

常见的**非简单请求**有：

- put，delete 方法的ajax请求

- 发送json格式的ajax请求，`Content-Type` 值为 `application/json`

- 带自定义头的ajax请求

非简单请求的CORS请求，会在正式通信之前，**增加一次HTTP查询请求，称为"预检"请求（preflight）**，浏览器发送一次 HTTP 预检 `OPTIONS`请求，先询问服务器，当前网页所在的**域名是否在服务器的许可名单之中**，以及可以使用哪些 HTTP 请求方法和头信息字段。

下面是这个"预检"请求的HTTP头信息。

> OPTIONS /cors HTTP/1.1
> Origin: http://rain.baimuxym.cn 
> Access-Control-Request-Method: PUT
> Access-Control-Request-Headers: X-Custom-Header
> Host: api.alice.com
> Accept-Language: en-US
> Connection: keep-alive
> User-Agent: Mozilla/5.0…

"预检"请求用的请求方法是`OPTIONS`，表示这个请求是用来询问的。头信息里面，关键字段是`Origin`，表示请求来自哪个源。

服务器收到"预检"请求以后，检查了`Origin`、`Access-Control-Request-Method`和`Access-Control-Request-Headers`字段以后，确认允许跨源请求，就可以做出回应。

只有得到肯定答复，浏览器才会发出正式的 `XHR` 请求，否则报错。

以上简单介绍了一下跨域的原理，其实跨域的解决方法在前端和后端都可以配置，方法也很多，这里就不展开讲了。



### 2.3、正向代理和反向代理

**正向代理**是一个位于客户端和原始服务器之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定原始服务器，然后代理向原始服务器转交请求并将获得的内容返回给客户端。代理服务器和客户端处于同一个局域网内。

比如说 你要访问 pornhub，你的浏览器是无法直接的，于是我就通过代理服务器（fanqiang）让它帮我转发，这种方式是客户端指定请求URL的。

> "喂？是代理服务器吗，帮我访问一下 www.pornhub.com"，正向代理的工作原理就像一个跳板。

**反向代理**实际运行方式是代理服务器接受网络上的连接请求。它将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给网络上请求连接的客户端 。

代理服务器和原始服务器处于同一个局域网内。反向代理隐藏了真实的服务器，为服务器收发请求，使真实服务器对客户端不可见。一般在处理跨域请求的时候比较常用。

比如说我要访问 https://rain.baimuxym.cn/images.jpg，对我来说不知道图片是不是同一个服务器返回回来的，甚至这个图片根本不在这台服务器，图片可以是服务器偷偷从其他 服务器，比如从 https://images.baimuxym.cn 拿回来的，但是用户并不知情。 

![ ](https://cdn.jsdelivr.net/gh/DogerRain/image@main/img/image-20201028150646664.png)

 

**代理的好处：**

- 保护了真实的web服务器，保证了web服务器的资源安全

只用于代理内部网络对Internet外部网络的连接请求，不支持外部网络对内部网络的连接请求，因为内部网络对外部网络是不可见的。所有的静态网页或者CGI程序，都保存在内部的Web服务器上。因此对反向代理服务器的攻击并不会使得网页信息遭到破坏，这样就增强了Web服务器的安全性。

- 节约了有限的IP地址资源

共享一个在internet中注册的IP地址（内网局域网），这些服务器分配私有地址，采用虚拟主机的方式对外提供服务。

- 减少WEB服务器压力，提高响应速度

负载的实现方式之一，不同的资源可以放在不同的服务器。（下面提到的负载均衡、动静分离也是通过反向代理实现的）

### 2.4、负载均衡

随着业务的发展，单个机器性能有限，无法承受巨大的请求压力，这个时候集群的概念产生了，将请求分发到各个服务器上（每台服务器的代码都是一样的），将负载分发到不同的服务器，这就是**负载均衡**，核心是「分摊压力」。

如何分发压力？Nginx也可以配置规则，比如说权重、轮询、hash 等等。

![](https://cdn.jsdelivr.net/gh/DogerRain/image@main/img-20210401/image-20210429101548741.png)



### 2.5、动静分离

动静分离的初衷是为了**加快网站的访问速度**。像大图片、视频、CSS 这种静态资源，如果都放在同一个服务器，在请求的时候就会造成带宽压力，如果把这些静态资源分散到不同的服务器，配合CDN，就可以减少服务器的压力。

由于 Nginx 的高并发和静态资源缓存等特性，经常将静态资源部署在 Nginx 上。

如果请求的是静态资源，直接到静态资源目录获取资源，如果是动态资源的请求，则利用**反向代理**的原理，把请求转发给对应后台应用去处理，从而实现动静分离。

![](https://cdn.jsdelivr.net/gh/DogerRain/image@main/img-20210401/image-20210429101604508.png)

## 3、安装Nginx

Nginx下载地址：https://nginx.org/en/download.html

我这里演示的Linux Centos7的安装，如果你是Windows，那就更简单了，直接解压即可。

**1、gcc 安装**

Nginx 是 C语言 开发，安装 Nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：

```bash
yum install gcc-c++
```

我这里显示已经是最新了：

```shell
[root@VM-8-8-centos ~]# yum install gcc-c++
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
Package gcc-c++-4.8.5-39.el7.x86_64 already installed and latest version
Nothing to do
```

**2、 PCRE pcre-devel 安装**
PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。

nginx也需要此库。命令：

```shell
yum install -y pcre pcre-devel
```

**3、zlib 安装**
zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。

```shell
yum install -y zlib zlib-devel
```

**4、OpenSSL 安装**
OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。

```shell
yum install -y openssl openssl-devel
```

**5、Nginx解压**

```shell
tar -zxvf nginx-1.15.2.tar.gz -C /var/www/web
```

**6、编译Nginx**

进入Nginx安装目录，输入 `./configure`，表示使用Nginx的默认配置，开启ssl

```shell
[root@VM-8-8-centos software]# cd /var/www/web/nginx-1.15.2/
[root@VM-8-8-centos nginx-1.15.2]# ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```

> 注意！`--prefix=/usr/local/nginx` 的意思是Nginx安装目录，如果不写默认的目录也是这个，如果你自定义，在下一步需要到你自定义的目录执行`make`和`make install`

然后输入：

```shell
make
```

编译成功再输入：

```shell
make install
```

Nginx的默认安装目录是 `/usr/local/nginx`

**7、启动Nginx**

Nginx的默认配置，是指向 `/usr/local/nginx/sbin/`这个目录的，配置文件是这里 `/usr/local/nginx/conf`

Nginx常用命令：（需要在安装目录执行，我的是 `/usr/local/nginx/sbin/` ）

```
./nginx  # 启动
./nginx -s stop #停止
./nginx -s reload #重启
```



启动，成功找到Nginx的进程。

```shell
[root@VM-8-8-centos nginx-1.15.2]# cd /usr/local/nginx/sbin/
[root@VM-8-8-centos sbin]# ./nginx
[root@VM-8-8-centos sbin]# ps -aux|grep nginx
root     11505  0.0  0.0  20556   616 ?        Ss   18:00   0:00 nginx: master process ./nginx
nobody   11506  0.0  0.0  23092  1384 ?        S    18:00   0:00 nginx: worker process
root     11545  0.0  0.0 112712   960 pts/4    R+   18:00   0:00 grep --color=auto nginx
```



然后浏览器输入自己的IP，出现**Welcome to Nginx** 就表示成功了。

 ![](https://cdn.jsdelivr.net/gh/DogerRain/image@main/img-20210401/image-20210116125047136.png)



为了方便使用，我们把Nginx添加到全局变量：

```bash
[root@VM-8-8-centos sbin]# ln -s /usr/local/nginx/sbin/nginx /usr/local/bin/
```

添加完毕，我们在任何一个目录就可以使用Nginx命令了。

这里要记住的是你的Nginx配置文件`nginx.conf` 目录是：`/usr/local/nginx/conf/nginx.conf`

## 4、Nginx常见命令

这里是把Nginx添加到全局变量才能这样操作，否则需要进入到 `/usr/local/nginx/sbin/` 目录，windows直接进入安装目录即可。

```
帮助命令：nginx -h
启动Nginx服务器 ： nginx
查看进程： ps aux | grep nginx
配置文件路径：/usr/local/nginx/conf/nginx.conf
检查配置文件： nginx -t
指定启动配置文件： nginx -c /usr/local/nginx/conf/nginx.conf
暴力停止服务： nginx -s stop
优雅停止服务： nginx -s quit
重新加载配置文件： nginx -s reload
```



## 5、Nginx的配置

### 5.1、Nginx的构成

打开`/usr/local/nginx/conf/nginx.conf` 文件，windows的配置文件在安装目录的 `conf/nginx.conf`

```
main        # 全局配置，也称为Main块，对全局生效
├── events  # 配置影响 Nginx 服务器或与用户的网络连接
├── http    # 配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置
│   ├── upstream # 配置后端服务器具体地址，负载均衡配置不可或缺的部分
│   ├── server   # 配置虚拟主机的相关参数，一个 http 块中可以有多个 server 块
│   ├── server
│   │   ├── location  # server 块可以包含多个 location 块，location 指令用于匹配 uri 
│   │   ├── location
│   │   └── ...
│   └── ...
└── ...
```

这个就像是Java的类一样，有公共的，也有私有的，server就像是一个 `if`  语法块，location又是一个子`if`语法块，只有匹配到了才会进入最后的URL。

一个 Nginx 配置文件的结构就像 `nginx.conf` 显示的那样，配置文件的语法规则：

1. 配置文件由指令与指令块构成；
2. 每条指令以 `;` 分号结尾，指令与参数间以空格符号分隔；
3. 指令块以 `{}` 大括号将多条指令组织在一起；
4. `include` 语句允许组合多个配置文件以提升可维护性；
5. 使用 `#` 符号添加注释，提高可读性；
6. 使用 `$` 符号使用变量；
7. 部分指令的参数支持正则表达式；

### 5.2、Nginx的典型配置

```bash
#-----------全局块 START-----------
user  nobody;                        # 运行用户，默认即是nginx，可以不进行设置
worker_processes  1;                # Nginx 进程数，一般设置为和 CPU 核数一样
error_log  /var/log/nginx/error.log warn;   # Nginx 的错误日志存放目录，日志级别是warn
pid        /var/run/nginx.pid;      # Nginx 服务启动时的 pid 存放位置

#-----------全局块 END-----------

#-----------events块 START-----------
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    use epoll;   #使用epoll的I/O模型，建议使用默认
    # 事件驱动模型有 select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections 1024;   # 每个进程允许最大并发数
}
#-----------events块 END-----------

http {   # 配置使用最频繁的部分，代理、缓存、日志定义等绝大多数功能和第三方模块的配置都在这里设置
    # 设置日志自定义格式
    log_format  myFormat  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  myFormat;   # Nginx访问日志存放位置，并采用上面定义好的格式

    sendfile            on;   # 开启高效传输模式
    tcp_nopush          on;   # 减少网络报文段的数量
    tcp_nodelay         on;
    keepalive_timeout   65;   # 保持连接的时间，也叫超时时间，单位秒
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;      # 文件扩展名与类型映射表
    default_type        application/octet-stream;   # 默认文件类型

    include /etc/nginx/conf.d/*.conf;   # 使用include引入其他子配置项，不存在会报错
    
    server {
        keepalive_requests 120; #单连接请求上限次数。
    	listen       80;       # 配置监听的端口
    	server_name  localhost 127.0.0.1 baimuxym.cn;    # 配置监听的域名或者地址，可多个，用空格隔开
    	
    	location / {
    		root   /usr/share/nginx/html;  # 网站根目录
    		index  index.html index.htm;   # 默认首页文件
    		deny 172.18.5.54   # 禁止访问的ip地址，可以为all
    		allow 172.18.5.53;# 允许访问的ip地址，可以为all
    	}
    	#新增内容，可以在自己的server单独配置错误日志
    	error_log    logs/error_localhost.log    error;
        
    	error_page 500 502 503 504 /50x.html;  # 默认50x对应的访问页面
    	error_page 400 404 error.html;   # 同上
    }
}
```



**特别说明一下：**

#### 1、error_log的默认值

```bash
error_log logs/error.log error;
```

error_log的语法格式及参数语法说明如下：

```
 error_log  <FILE>  <LEVEL>;
 关键字     日志文件  错误日志级别
```

关键字：其中关键字error_log不能改变

日志文件：可以指定任意存放日志的目录

错误日志级别：常见的错误日志级别有`[debug | info | notice | warn | error | crit | alert | emerg]`，级别越高记录的信息越少。

> 生产场景一般是 warn | error | crit 这三个级别之一
>
> 注意：不要配置info等级较低的级别，会带来大量的磁盘I/O消耗。

error_log参数允许放置的标签段位置：

```
main, http, server, location
```

测试了一下，日志文件不存在，Nginx不会创建，所以要提前手动创建。

#### 2、events 块

events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。 上述例子就表示每个 work process 支持的最大连接数为 1024，这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置。

### 5.3 、Nginx的location

server 块可以包含多个 location 块，location 指令用于匹配 uri，语法：

```bash
location [ = | ~ | ~* | ^~ ]  /uri/  {
	...
}
```

- `=` 开头表示精确匹配，如果匹配成功，不再进行后续的查找；
- `^`  以xxx开头的匹配
- `$`  以xxx结尾的匹配
- `*`  代表任意字符
- `^~` 匹配**开头以 xxx 的路径**，理解为匹配 url 路径即可。nginx不对url做编码，因此请求为`/static/20%/HaC`，可以被规则`^~ /static/ /HaC`匹配到（注意是空格）
- `~` 开头表示**区分大小写**的正则匹配          
- `~*` 开头表示**不区分大小写**的正则匹配        
- `!` 表示不包含xxx就匹配
- `/` 通用匹配，任何请求都会匹配到。



`~*` 和 `~`优先级都比较低，如有多个location的正则能匹配的话，则使用正则表达式最长的那个；

如果 uri 包含正则表达式，则必须要有 `~` 或 `~*` 标志。

借用https://www.cnblogs.com/jpfss/p/10232980.html博客一文的例子来说明一下：

```php
server {
    listen 80;
    server_name  localhost;
    location = / {
       #规则A
    }
    location = /login {
       #规则B
    }
    location ^~ /static/ {
       #规则C
    }
    location ~ \.(gif|jpg|png|js|css)$ {
       #规则D，注意：是根据括号内的大小写进行匹配。括号内全是小写，只匹配小写
    }
    location ~* \.png$ {
       #规则E
    }
    location !~ \.xhtml$ {
       #规则F
    }
    location !~* \.xhtml$ {
       #规则G
    }
    location / {
       #规则H
    }
}
```

**那么产生的效果如下：**

访问根目录/， 比如http://localhost/ 将匹配规则A

访问 http://localhost/login 将匹配规则B，http://localhost/register 则匹配规则H

访问 http://localhost/static/a.html 将匹配规则C

访问 http://localhost/a.gif, http://localhost/b.jpg 将匹配规则D和规则E，但是规则D顺序优先，规则E不起作用， 而 http://localhost/static/c.png 则优先匹配到 规则C

访问 http://localhost/a.PNG 则匹配规则E， 而不会匹配规则D，因为规则E不区分大小写。

访问 http://localhost/a.xhtml 不会匹配规则F和规则G，

http://localhost/a.XHTML不会匹配规则G，（因为!）。规则F，规则G属于排除法，符合匹配规则也不会匹配到，所以想想看实际应用中哪里会用到。

访问 http://localhost/category/id/1111 则最终匹配到规则H，因为以上规则都不匹配，这个时候nginx转发请求给后端应用服务器，比如FastCGI（php），tomcat（jsp），nginx作为方向代理服务器存在。



所以实际使用中，个人觉得至少有三个匹配规则定义，如下：

```bash
#直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理，官网如是说。
#这里是直接转发给后端应用服务器了，也可以是一个静态首页
# 第一个必选规则
location = / {
	#反向代理
    proxy_pass http://127.0.0.1:8080/index
}
 
# 第二个必选规则是处理静态文件请求，这是nginx作为http服务器的强项
# 有两种配置模式，目录匹配或后缀匹配,任选其一或搭配使用
location ^~ /static/ {                              //以xx开头
    root /webroot/static/;
}
location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {     //以xx结尾
    root /webroot/res/;
}
 
#第三个规则就是通用规则，用来转发动态请求到后端应用服务器
#非静态文件请求就默认是动态请求，自己根据实际把握
location / {
    proxy_pass http://tomcat:8080/
}
```



### 5.4、Nginx的全局变量

Nginx提供了一些全局变量，我们可以在任意的位置使用这些变量。

| 全局变量名         | 功能                                                         |
| :----------------- | :----------------------------------------------------------- |
| `$host`            | 请求信息中的 `Host`，如果请求中没有 `Host`行，则等于设置的服务器名，不包含端口 |
| `$request_method`  | 客户端请求类型，如 `GET`、`POST`                             |
| `$remote_addr`     | 客户端的 `IP` 地址                                           |
| `$args`            | 请求中的参数                                                 |
| `$arg_PARAMETER`   | `GET` 请求中变量名 PARAMETER 参数的值，例如：`$http_user_agent`(Uaer-Agent 值), `$http_referer`... |
| `$content_length`  | 请求头中的 `Content-length` 字段                             |
| `$http_user_agent` | 客户端agent信息                                              |
| `$http_cookie`     | 客户端cookie信息                                             |
| `$remote_addr`     | 客户端的IP地址                                               |
| `$remote_port`     | 客户端的端口                                                 |
| `$http_user_agent` | 客户端agent信息                                              |
| `$server_protocol` | 请求使用的协议，如 `HTTP/1.0`、`HTTP/1.1`                    |
| `$server_addr`     | 服务器地址                                                   |
| `$server_name`     | 服务器名称                                                   |
| `$server_port`     | 服务器的端口号                                               |
| `$scheme`          | HTTP类型（如http，https）                                    |



## 6、负载均衡与限流

### 6.1、负载均衡

Nginx 默认提供的负载均衡策略：

- 1、轮询（默认）round_robin

  > 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。

- 2、IP 哈希 ip_hash

  > 每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 共享的问题。
  >
  > 当然，实际场景下，一般不考虑使用 ip_hash 解决 session 共享。

- 3、最少连接 least_conn

  > 下一个请求将被分派到活动连接数量最少的服务器

- 4、权重 weight

  >weight的值越大分配到的访问概率越高，主要用于后端每台服务器性能不均衡的情况下，达到合理的资源利用率。

还可以通过插件支持其他策略。

例如：使用weight负载均衡策略分发请求到不同的服务

```bash
	upstream mysite {
        server 127.0.0.1:8090 weight=1;
        server 127.0.0.1:8091 weight=1;
    }
    server {
        listen       80;
        server_name  hellocoder.com ;

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
		
		location / {
		 proxy_pass http://mysite;
        }
    }
```

### 6.2、限流

`ngx_http_limit_req_module`  模块提供了漏桶算法(leaky bucket)，可以限制单个IP的请求处理频率。

如：

**正常限流：**

```bash
http {
	limit_req_zone 192.168.1.1 zone=myLimit:10m rate=5r/s;
}

server {
	location / {
		limit_req zone=myLimit;
		rewrite / http://www.hac.cn permanent;
	}
}
```

参数解释：

```
key: 定义需要限流的对象。
zone: 定义共享内存区来存储访问信息。
rate: 用于设置最大访问速率。
```

表示基于客户端192.168.1.1进行限流，定义了一个大小为10M，名称为myLimit的内存区，用于存储IP地址访问信息。

`rate` 设置IP访问频率，`rate=5r/s`表示每秒只能处理每个IP地址的5个请求。Nginx限流是按照毫秒级为单位的，也就是说1秒处理5个请求会变成每200ms只处理一个请求。如果200ms内已经处理完1个请求，但是还是有有新的请求到达，这时候Nginx就会拒绝处理该请求。



## 7、Nginx实践

### 7.1、正向代理

我上传了一个静态网站在某个服务器目录，假如我的服务器是`81.71.16.134`，

那要如何通过我的域名 [https://learnjava.baimuxym.cn](https://learnjava.baimuxym.cn) 访问它呢？

步骤如下：

1、首先需要把域名解析到我的IP

2、申请SSL证书，配置服务器

3、正向代理，监听443端口（因为https的端口是443），转发；监听 80 端口，`http://` 强制 跳到 `https://`。

```bash
  server {
       listen       80; #监听端口
       server_name  learnjava.baimuxym.cn; #请求域名
       return      301 https://$host$request_uri; #重定向至https访问。
   }
   
    server {
    listen 443 ssl; # 这里改的是
    server_name learnjava.baimuxym.cn; # 改为绑定证书的域名

    ssl on;
    ssl_certificate /usr/local/nginx/learnjava.baimuxym.cn/1_learnjava.baimuxym.cn_bundle.crt; # 改为自己申请得到的 crt 文件的名称或者 pem文件的名称
    ssl_certificate_key /usr/local/nginx/learnjava.baimuxym.cn/2_learnjava.baimuxym.cn.key; # 改为自己申请得到的 key 文件的名称
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;

    location / { # 匹配所有
    root  /var/www/web/LearnJavaToFindAJob/docs; # 你的静态网站项目路径，刚刚上传的目录
    index index.html;	 # 首页名称
    	}
    }
```



参考：

- https://mp.weixin.qq.com/s/pjhZi5cmpewwHZNn-aQJiA
- https://blog.csdn.net/yexudengzhidao/article/details/100104134
- https://www.cnblogs.com/jpfss/p/10232980.html

