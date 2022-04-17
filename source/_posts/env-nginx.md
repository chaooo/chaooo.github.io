---
title: 「环境配置」Nginx常用配置详解
date: 2020-12-29 18:07:05
tags: [环境配置, Nginx]
categories: 环境配置
---


Nginx 的配置系统由一个主配置文件和其他一些辅助的配置文件构成。这些配置文件均是纯文本文件，全部位于Nginx安装目录下的`conf`目录下。

Nginx配置文件中每个指令必须有`;`结束。以`#`开头的行会被当做注释。

配置文件通常命名为`nginx.conf`并且默认放置在`/usr/local/nginx/conf`，`/etc/nginx`，或`/usr/local/etc/nginx`。
<!-- more -->

### 1. Nginx配置文件结构
* Nginx配置文件常用到的几个部分：main（全局设置），server（主机设置），upstream（上游服务器设置，主要为反向代理、负载均衡相关配置），location（URL匹配特定位置后的设置）
* 每部分包含若干个指令。
    + main模块：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。
    + events模块：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
    + http模块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
    + upstream模块：使nginx跨越单机的限制，完成网络数据的接收、处理和转发，该指令用于设置一系列的后端服务器，设置反向代理及后端服务器的负载均衡；
    + server模块：主要用于指定虚拟主机域名、IP和端口；
    + location模块：配置请求的路由，以及各种页面的处理情况（比如，根目录“/”,“/images”,等等）。
* 他们之间的关系式：
    + server继承main，location继承server；
    + upstream既不会继承指令也不会被继承。它有自己的特殊指令，不需要在其他地方的应用。

``` conf
...                           # main(全局)模块

events {                      # events模块
   ...
}
http {                        # http模块
    ...
    upstream {                # upstream模块
        ...
    }
    server {                  # server模块
        ...
        location [PATTERN] {  # location模块
            ...
        }
        location [PATTERN] {
            ...
        }
    }
    server {
      ...
    }
    ...
}
```


### 2. Nginx配置文件详解
#### 2.1 全局模块
``` conf
# ------------------------------ 全局模块 start ------------------------------
# user             www www;               # 配置用户或者组，默认为nobody nobody。

# worker_processes 1;                     # 允许生成的进程数，默认为1，建议设置为等于CPU总核心数。

# pid              /nginx/pid/nginx.pid;  # 指定nginx进程pid文件存放地址。

error_log          /usr/local/nginx/logs/error.log info; # 全局错误日志路径，级别，[ debug|info|notice|warn|error|crit|alert|emerg ]。
                                                         # 这个设置可以放入全局块，http块，server块

worker_rlimit_nofile 65535;   # 指定进程可以打开的最大描述符。
                              # 工作模式与连接数上限：这个指令是指当一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n 的值保持一致。
                              # 现在在linux 2.6内核下开启文件打开数为65535。因为nginx调度时分配请求到进程并不是那么的均衡，所以假如填写10240，总并发量达到3-4万时就有进程可能超过10240了，这时会返回502错误。
# ------------------------------ 全局模块 end ------------------------------
events { ... } # events块
http { ... }   # http块
```

#### 2.2 events模块
``` conf

# ------------------------------ events模块 start ------------------------------
events {
    use epoll;  # 参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; 
                # epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型，linux建议epoll，如果跑在FreeBSD上面，就用kqueue模型。
                # nginx针对不同的操作系统，有不同的事件模型：
                #    1.标准事件模型：Select、poll属于标准事件模型，如果当前系统不存在更有效的方法，nginx会选择select或poll
                #    2.高效事件模型：Kqueue，Epoll，/dev/poll，Eventport
                #        Kqueue：使用于FreeBSD 4.1+, OpenBSD 2.9+, NetBSD 2.0 和 MacOS X.使用双处理器的MacOS X系统使用kqueue可能会造成内核崩溃。
                #        Epoll：使用于Linux内核2.6版本及以后的系统。
                #        /dev/poll：使用于Solaris 7 11/99+，HP/UX 11.22+ (eventport)，IRIX 6.5.15+ 和 Tru64 UNIX 5.1A+。
                #        Eventport：使用于Solaris 10。 为了防止出现内核崩溃的问题， 有必要安装安全补丁。
    
    worker_connections 65535;   # 单个进程最大连接数（最大连接数=连接数*进程数）。
                                # 根据硬件调整，和前面工作进程配合起来用，尽量大，但是别把cpu跑到100%就行。每个进程允许的最多连接数，理论上每台nginx服务器的最大连接数为。

    accept_mutex on;            # 网路连接序列化，防止惊群现象发生，默认:on
    multi_accept on;            # 一个进程是否同时接受多个网络连接，默认:off
    keepalive_timeout 60;       # keepalive超时时间。

    client_header_buffer_size 4k;   # 客户端请求头部的缓冲区大小。
                                    # 分页大小可以用shell命令getconf PAGESIZE取得，如：[root@centos ~]# getconf PAGESIZE # 返回4096。
                                    # 但也有client_header_buffer_size超过4k的情况，但是client_header_buffer_size该值必须设置为“系统分页大小”的整倍数。

    open_file_cache max=65535 inactive=60s;   # 为打开文件指定缓存，默认是关闭的。max指定缓存数量，建议和打开文件数一致，inactive是指经过多长时间文件没被请求后删除缓存。

    open_file_cache_valid 80s;      # 检查open_file_cache中缓存项目的有效信息的时间间隔。
                                    # 默认值:60；使用字段:http, server, location

    open_file_cache_min_uses 1;     # 指定在open_file_cache指令无效的参数中一定的时间范围内可以使用的最小文件数,如果使用更大的值,文件描述符在cache中总是打开状态。
                                    # 如上例，如果有一个文件在inactive时间内一次没被使用，它将被移除。
                                    # 默认值:1；使用字段:http, server, location

    open_file_cache_errors on;      #　指定是否在搜索一个文件时记录cache错误[ on | off ]，默认值:off
}
# ------------------------------ events模块 end ------------------------------

http { ... }   # http块
```

#### 2.2 http模块
``` conf
... # 全局块
events { ... } # events块
# ------------------------------ http模块 start ------------------------------
http {      
    # 设定http服务器，利用它的反向代理功能提供负载均衡支持

    include       mime.types;                # 文件扩展名与文件类型映射表
    default_type  application/octet-stream;  # 默认文件类型，默认为text/plain
    #charset      utf-8;                     # 默认编码

    server_names_hash_bucket_size 128;  # 服务器名字的hash表大小
                                        # 保存服务器名字的hash表是由指令server_names_hash_max_size 和server_names_hash_bucket_size所控制的。参数hash bucket size总是等于hash表的大小，并且是一路处理器缓存大小的倍数。在减少了在内存中的存取次数后，使在处理器中加速查找hash表键值成为可能。如果hash bucket size等于一路处理器缓存的大小，那么在查找键的时候，最坏的情况下在内存中查找的次数为2。第一次是确定存储单元的地址，第二次是在存储单元中查找键 值。因此，如果Nginx给出需要增大hash max size 或 hash bucket size的提示，那么首要的是增大前一个参数的大小.

    client_header_buffer_size 32k;  # 客户端请求头部的缓冲区大小。   
                                    # 这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。分页大小可以用命令getconf PAGESIZE取得。

    large_client_header_buffers 4 64k;  # 客户请求头缓冲大小。
                                        # nginx默认会用client_header_buffer_size这个buffer来读取header值，如果header过大，它会使用large_client_header_buffers来读取。

    client_max_body_size 8m;  # 设定通过nginx上传文件的大小

    sendfile on;    # 允许sendfile方式（高效文件传输模式）传输文件，默认为off，可以在http块，server块，location块。
                    # 指定nginx是否调用sendfile函数（zero copy方式）来输出文件，对于普通应用，必须设为on。如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime。

    autoindex on;   # 开启目录列表访问，合适下载服务器，默认关闭。

    tcp_nopush on;  # 此选项允许或禁止使用socke的TCP_CORK的选项，此选项仅在使用sendfile的时候使用

    tcp_nodelay on; # 该参数最核心的功能，就是把小包组成成大包，提高带宽利用率也就是著名的nagle算法

    keepalive_timeout 120;   # 长连接超时时间，单位是秒

    # FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。下面参数看字面意思都能理解。
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;

    # gzip模块设置
    gzip on;                # 开启gzip压缩输出
    gzip_min_length 1k;     # 最小压缩文件大小
    gzip_buffers 4 16k;     # 压缩缓冲区
    gzip_http_version 1.0;  # 压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    gzip_comp_level 2;      # 压缩等级
    gzip_types text/plain application/x-javascript text/css application/xml; # 压缩类型，默认就已经包含textml，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    gzip_vary on;

    # limit_zone crawler $binary_remote_addr 10m; # 开启限制IP连接数的时候需要使用
    
    upstream { ... } # upstream模块
    server { ... }   # server模块
}
# ------------------------------ http模块 end ------------------------------
```

#### 2.3 upstream模块
``` conf
... # 全局块
events { ... } # events块
http {
    ...        # http块
# ------------------------------ upstream模块 start ------------------------------
    upstream www.test.com {    # upstream的负载均衡，weight是权重，可以根据机器配置定义权重。weigth参数表示权值，权值越高被分配到的几率越大。
        server 192.168.80.121:80 weight=3;
        server 192.168.80.122:80 weight=2;
        server 192.168.80.123:80 weight=3;
    }
    # nginx的upstream目前支持4种方式的分配: ①轮询（默认），②weight，③ip_hash，④第三方
        # 轮询（默认）：每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
        # weight：指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
            # 例如： upstream bakend {
            #           server 192.168.0.14 weight=10;
            #           server 192.168.0.15 weight=10;
            #       }
        # ip_hash：每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
            # 例如： upstream bakend {
            #           ip_hash;
            #           server 192.168.0.14:88;
            #           server 192.168.0.15:80;
            #        }
        # fair（第三方）：按后端服务器的响应时间来分配请求，响应时间短的优先分配。
            # 例如： upstream backend {
            #           server server1;
            #           server server2;
            #           fair;
            #        }
        # url_hash（第三方）：按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
            # 例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
            # upstream backend {
            #     server squid1:3128;
            #     server squid2:3128;
            #     hash $request_uri;
            #     hash_method crc32;
            # }
    # tips:
    # upstream bakend{  # 定义负载均衡设备的Ip及设备状态
    #    ip_hash;
    #    server 127.0.0.1:9090 down;
    #    server 127.0.0.1:8080 weight=2;
    #    server 127.0.0.1:6060 max_fails=10 fail_timeout=60s;
    #    server 127.0.0.1:7070 backup;
    # }
    # 在需要使用负载均衡的server中增加 proxy_pass http://bakend/;

    # 每个设备的状态设置为:
    #     1.down表示单前的server暂时不参与负载
    #     2.weight为weight越大，负载的权重就越大。
    #     3.max_fails：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream模块定义的错误
    #                  fail_timeout:max_fails次失败后，暂停的时间。
    #     4.backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

    # nginx支持同时设置多组的负载均衡，用来给不用的server来使用。
    # client_body_in_file_only设置为On 可以讲client post过来的数据记录到文件中用来做debug
    # client_body_temp_path设置记录文件的目录 可以设置最多3层目录
    # location对URL进行匹配.可以进行重定向或者进行新的代理 负载均衡

    server { ... }   # server模块
}
# ------------------------------ upstream模块 end ------------------------------
```

#### 2.3 server模块
``` conf
... # 全局块
events { ... } # events块
http {
    ...              # http块
    upstream { ... } # upstream模块

    # ------------------------------ server模块 start ------------------------------

    server {    # 虚拟主机的配置
        listen 80;      # 监听端口
        server_name www.test.com test.com;  # 监听地址，域名可以有多个，用空格隔开
        index index.html index.htm;       # 指定网站初始页
        root /data/www/test;                   # 指定网站根目录
        charset utf-8;
        rewrite ^(.*)$ https://www.test.com$1 permanent;  # URL重写

        access_log  /usr/local/nginx/logs/host.access.log; # 定义本虚拟主机的访问日志
    }
    server {
        location ...{ ... }  # location模块
    }

    # ------------------------------ server模块 end ------------------------------

    # include /etc/nginx/conf.d/*.conf;  # 导入其他server配置
```

+ URL重写（rewrite）配置及信息详解：
    * 语法：`rewrite <regex> <replacement> [flag];`
    * `regex`：perl兼容正则表达式语句进行规则匹配
    * `replacement`：将正则匹配的内容替换成replacement
    * `flag` rewrite支持的flag标记
        - `last`:本条规则匹配完成后，继续向下匹配新的location URI规则
        - `break`:本条规则匹配完成即终止，不再匹配后面的任何规则
        - `redirect`:返回302临时重定向，浏览器地址会显示跳转后的URL地址
        - `permanent`:返回301永久重定向，浏览器地址栏会显示跳转后的URL地址
    * 使用位置：server, location, if


#### 2.4 location模块
语法：`location  [ = | ~ | ~* | ^~ | @]  /uri/  { configuration }`。
匹配模式分为两种：普通字符串（literal string）和正则表达式（regular expression），其中 ~ 和 ~* 用于正则表达式， 其他前缀和无任何前缀都用于普通字符串。
+ 前缀含义：
    * `=`：=开头表示精确前缀匹配，只有完全匹配才能生效。
    * `~`：~开头表示区分大小写的正则匹配。
    * `~*`：~*开头表示不区分大小写的正则匹配。
    * `^~`：^~开头表示普通字符串匹配上以后不再进行正则匹配。

``` conf
... # 全局块
events { ... } # events块
http {
    ...              # http块
    upstream { ... } # upstream模块
    server {    # 虚拟主机的配置
        listen 80;      # 监听端口
        server_name www.test.com test.com;  # 监听地址

        # ------------------------------ location模块 start ------------------------------
        ### alias: 别名配置
        location /test/ {
            alias  /usr/local/;  # 在匹配到location配置的URL路径后，指向alias配置的路径
        }
        location ~* /img/(.+\.(gif|jpg|jpeg|png|bmp|swf)) {
            alias  /usr/local/images/$1;    # 请求中只要能匹配到正则，比如/img/test.png或者/resource/img/test.png，都会转换为请求/usr/local/images/test.png。
        }
        ### root: 根路径配置
        location /test/ {
            root  /usr/local/;  # 用于访问文件系统，在匹配到location配置的URL路径后，指向root配置的路径，并把请求路径附加到其后
        }
        ### proxy_pass: 反向代理配置
        location /test/ {
            proxy_pass  http://127.0.0.1:8080/;  # 用于代理请求，适用于前后端负载分离或多台机器、服务器负载分离的场景
        }
        ### JS和CSS缓存时间设置
        location ~ .*.(js|css)?$ {
            expires 1h;  # JS和CSS缓存时间设置
        }
        ### 反向代理的其他配置
        location / {             # 对 "/" 启用反向代理
            proxy_pass http://127.0.0.1:88;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;

            # 后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             
            # 以下是一些反向代理的配置，可选。
            proxy_set_header Host $host;
            client_max_body_size 10m;     # 允许客户端请求的最大单文件字节数
            client_body_buffer_size 128k; # 缓冲区代理缓冲用户端请求的最大字节数，
            proxy_intercept_errors on;    # 表示使nginx阻止HTTP应答代码为400或者更高的应答。
            proxy_connect_timeout 90;     # 后端服务器连接的超时时间_发起握手等候响应超时时间
            proxy_send_timeout 90;        # 后端服务器数据回传时间(代理发送超时)
            proxy_read_timeout 90;        # 连接成功后，后端服务器响应时间(代理接收超时)
            proxy_buffer_size 4k;         # 设置代理服务器（nginx）保存用户头信息的缓冲区大小
            proxy_buffers 4 32k;          # proxy_buffers缓冲区，网页平均在32k以下的设置，设置用于读取应答（来自被代理服务器）的缓冲区数目和大小，默认情况也为分页大小，根据操作系统的不同可能是4k或者8k
            proxy_busy_buffers_size 64k;  # 高负荷下缓冲大小（proxy_buffers*2）
            proxy_temp_file_write_size 64k; # 设置在写入proxy_temp_path时数据的大小，预防一个工作进程在传递文件时阻塞太长
                                            # 设定缓存文件夹大小大于这个值将从upstream服务器传
        }
        # ------------------------------ location模块 end ------------------------------
    }
```


### 3. Nginx日志配置
Nginx日志主要分为两种：`access_log(访问日志)`和`error_log(错误日志)`。
1. **访问日志**可以记录用户的IP地址、浏览器的信息，请求的处理时间等信息。
    + `access_log指令`的作用域：http，server，location，limit_except。
2. **错误日志**记录了访问出错的信息，可以帮助我们定位错误的原因。
    + `error_log指令`的作用域：main， http, mail, stream, server, location

#### 3.1 Nginx访问日志`access_log`
访问日志主要记录客户端的请求。客户端向Nginx服务器发起的每一次请求都记录在这里。
客户端IP，浏览器信息，referer，请求处理时间，请求URL等都可以在访问日志中得到。
可以通过log_format指令定义具体要记录哪些信息。

语法：
``` conf
# 关闭访问日志
access_log off; 
# 设置访问日志
# access_log  path  [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];
```

+ `path`：指定日志的存放位置。
+ `format`：指定日志的格式。默认使用预定义的combined。
+ `buffer`：用来指定日志写入时的缓存大小。默认是64k。
+ `gzip`：日志写入前先进行压缩。压缩率可以指定，从1到9数值越大压缩比越高，同时压缩的速度也越慢。默认是1。
+ `flush`：设置缓存的有效时间。如果超过flush指定的时间，缓存中的内容将被清空。
+ `if条件判断`：如果指定的条件计算为0或空字符串，那么该请求不会写入日志。

##### 3.1.1 使用log_format自定义日志格式
Nginx预定义了名为combined日志格式，如果没有明确指定日志格式默认使用该格式：
``` bash
log_format combined '$remote_addr - $remote_user [$time_local] ''"$request" $status $body_bytes_sent ''"$http_referer" "$http_user_agent"';
```

如果不想使用Nginx预定义的格式，可以通过log_format指令来自定义:
``` bash
log_format name [escape=default|json] string ...;
```

* name 格式名称。在access_log指令中引用。
* escape 设置变量中的字符编码方式是json还是default，默认是default。
* string 要定义的日志格式内容。该参数可以有多个。参数中可以使用Nginx变量。

实例：
``` conf
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';

access_log  /usr/local/nginx/logs/access.log  main;
```

* 日志格式设定常用Nginx变量：
    + `$remote_addr` / `$http_x_forwarded_for`：用以记录客户端的ip地址；
    + `$remote_user`：用来记录客户端用户名称；
    + `$time_local`： 用来记录访问时间与时区；
    + `$request`： 用来记录请求的url与http协议；
    + `$status`： 用来记录请求状态；成功是200，
    + `$body_bytes_sent`：记录发送给客户端文件主体内容大小；
    + `$http_referer`：用来记录从那个页面链接访问过来的；
    + `$http_user_agent`：记录客户浏览器的相关信息；



#### 3.2 Nginx错误日志`error_log`
错误日志在Nginx中是通过error_log指令实现的。该指令记录服务器和请求处理过程中的错误信息。

语法：`error_log  file  [level];`，默认是：`error_log  logs/error.log  error;`；
级别有：`[ debug|info|notice|warn|error|crit|alert|emerg ]`。

基本用法：
``` conf
error_log  /usr/local/nginx/logs/error.log  info;
```



#### 3.3 通过`open_log_file_cache`指令来设置日志文件描述符缓存
Nginx中通过`access_log`和`error_log`指令配置访问日志和错误日志，通过`log_format`我们可以自定义日志格式。

如果日志文件路径中使用了变量，我们可以通过`open_log_file_cache`指令来设置缓存，提升性能。它可以配置在http、server、location作用域中。

* 语法: `open_log_file_cache max=N [inactive=time] [min_uses=N] [valid=time];`
    + `max` 设置缓存中最多容纳的文件描述符数量，如果被占满，采用LRU算法将描述符关闭。
    + `inactive` 设置缓存存活时间，默认是10s。
    + `min_uses` 在inactive时间段内，日志文件最少使用几次，该日志文件描述符记入缓存，默认是1次。
    + `valid`：设置多久对日志文件名进行检查，看是否发生变化，默认是60s。
    + `off`：不使用缓存。默认为off。

基本用法: 
``` conf
open_log_file_cache max=1000 inactive=20s valid=1m min_uses=2;
```


### 4. Nginx配置文件实例：
``` conf
user  cmuser;
worker_processes  2;

events {
    worker_connections  8096;
    multi_accept        on;
    use                 epoll;
}

worker_rlimit_nofile 40000;

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /data/runtime/nginx/logs/access.log  main;
    error_log   /data/runtime/nginx/logs/error.log   error;

    sendfile           on;
    tcp_nopush         on;
    tcp_nodelay        on;

    keepalive_timeout  15;

    # ------------ www.test.com的访问配置 --------------------
    upstream www.test.com {
        ip_hash;
        server 10.104.60.165:8100;
        server 10.104.60.166:8100;
    }
    server {
        listen  80;
        server_name  www.test.com;
        charset utf-8;
        rewrite ^(.*)$ https://www.test.com$1 permanent;
    }
    server {                           # 配置SSL证书
        listen 443;
        server_name www.test.com;
        ssl on;                            
        ssl_certificate sslkey/_.test.com_bundle.crt;
        ssl_certificate_key sslkey/_.test.com.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
        ssl_prefer_server_ciphers on;
        # 上传文件的路径配置
        location /uploadFile/ {
            alias /mnt/newdatadrive/nfs_client/upload/;
        }
        location / {
            proxy_pass http://www.test.com;
            proxy_redirect  off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size 10m;
            client_body_buffer_size 256k;
            proxy_connect_timeout 90;
            proxy_send_timeout 90;
            proxy_read_timeout 90;
            proxy_buffer_size 8k;
            proxy_buffers 4 64k;
            proxy_busy_buffers_size 64k;
            proxy_temp_file_write_size 64k;
        }
    }

    # ------------------ 静态文件夹 -----------------
    server {
            listen  80;
            server_name  static.test.com;
            charset utf-8;
            location / {
                root /mnt/newdatadrive/static_files;
            }
    }
    server {
            listen 443;
            server_name static.test.com;
            ssl on;
            ssl_certificate sslkey/_.test.com_bundle.crt;
            ssl_certificate_key sslkey/_.test.com.key;
            ssl_session_timeout 5m;
            ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
            ssl_ciphers AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
            ssl_prefer_server_ciphers on;
    
            location / {
                root /mnt/newdatadrive/static_files;
            }
    }
    
    # ------------------ 引入外部其他配置 -----------------
    include /etc/nginx/conf.d/*.conf;
}
```



### 附. Nginx启动命令
运行可执行文件就可以启动`nginx`，可以使用`-c`参数指定配置文件，比如:
``` bash
nginx -c /usr/local/nginx/conf/nginx.conf
```

如果nginx已经启动，可以使用-s参数的可执行命令来控制：
``` bash
nginx -s [reload | stop | quit | reopen]

# stop — 直接关闭 nginx
# quit — 会在处理完当前正在的请求后退出，也叫优雅关闭
# reload — 重新加载配置文件，相当于重启
# reopen — 重新打开日志文件
```

比如，重载配置文件：

``` bash
nginx -s reload
```
