

#### NGINX性能优化

```
worker_processes 1;
进程个数的策略：worker 进程数可以设置为等于 CPU 的核数。高流量高并发场合也可以考虑将进程数提高至 CPU 核数 x 2。这个参数除了要和 CPU 核数匹配之外，也与硬盘存储的数据及系统的负载有关，设置为 CPU 核数是个好的起始配置，也是官方建议的。

events {
  use epoll;
}
events 指令是设定 Nginx 的工作模式及连接数上限。use指令用来指定 Nginx 的工作模式。Nginx 支持的工作模式有 select、 poll、 kqueue、 epoll 、 rtsig 和/ dev/poll。当然，也可以不指定事件处理模型，Nginx 会自动选择最佳的事件处理模型。

events {
  worker_connections 20480;
}
worker_connections 也是个事件模块指令，用于定义 Nginx 每个进程的最大连接数，默认是 1024。

worker_rlimit_nofile 65535;
配置 worker 进程的最大打开文件数

events {
  multi_accept on;
}
Nginx 进程只会在一个时刻接收一个新的连接，我们可以配置multi_accept 为 on，实现在一个时刻内可以接收多个新的连接，提高处理效率。该参数默认是 off，建议开启。

```

##### 理解sendfile

```
sendfile on;   
允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。

不用 sendfile的传统网络传输过程：
read(file,tmp_buf, len);
write(socket,tmp_buf, len);
硬盘 >> kernel buffer >> user buffer>> kernel socket buffer >>协议栈

1）一般来说一个网络应用是通过读硬盘数据，然后写数据到socket 来完成网络传输的。上面2行用代码解释了这一点，不过上面2行简单的代码掩盖了底层的很多操作。来看看底层是怎么执行上面2行代码的：
系统调用 read()产生一个上下文切换：从 user mode 切换到 kernel mode，然后 DMA 执行拷贝，把文件数据从硬盘读到一个 kernel buffer 里。
数据从 kernel buffer拷贝到 user buffer，然后系统调用 read() 返回，这时又产生一个上下文切换：从kernel mode 切换到 user mode。
系统调用write()产生一个上下文切换：从 user mode切换到 kernel mode，然后把步骤2读到 user buffer的数据拷贝到 kernel buffer（数据第2次拷贝到 kernel buffer），不过这次是个不同的 kernel buffer，这个 buffer和 socket相关联。
系统调用 write()返回，产生一个上下文切换：从 kernel mode 切换到 user mode（第4次切换了），然后 DMA 从 kernel buffer拷贝数据到协议栈（第4次拷贝了）。

在linux kernel2.0+ 版本中，系统调用 sendfile() 就是用来简化上面步骤提升性能的。sendfile() 不但能减少切换次数而且还能减少拷贝次数。
sendfile(socket,file, len);
硬盘 >> kernel buffer (快速拷贝到kernelsocket buffer) >>协议栈

系统调用sendfile()通过 DMA把硬盘数据拷贝到 kernel buffer，然后数据被 kernel直接拷贝到另外一个与 socket相关的 kernel buffer。这里没有 user mode和 kernel mode之间的切换，在 kernel中直接完成了从一个 buffer到另一个 buffer的拷贝。
DMA 把数据从 kernelbuffer 直接拷贝给协议栈，没有切换，也不需要数据从 user mode 拷贝到 kernel mode，因为数据就在 kernel 里。
```

##### 长连接（通过jemeter压测效果明显）

```
当使用nginx作为反向代理时，为了支持长连接，需要做到两点：
1.从client到nginx的连接是长连接
2.从nginx到server的连接是长连接

对于第一点
http {
    keepalive_timeout  120s 120s;
    keepalive_requests 10000;
}
1）keepalive_timeout
第一个参数：设置keep-alive客户端连接在服务器端保持开启的超时值（默认75s）；值为0会禁用keep-alive客户端连接；
第二个参数：可选、在响应的header域中设置一个值“Keep-Alive: timeout=time”；通常可以不用设置；
2）keepalive_requests：
keepalive_requests指令用于设置一个keep-alive连接上可以服务的请求的最大数量，当最大请求数量达到时，连接被关闭。默认是100。这个参数的真实含义，是指一个keep alive建立之后，nginx就会为这个连接设置一个计数器，记录这个keep alive的长连接上已经接收并处理的客户端请求的数量。如果达到这个参数设置的最大值时，则nginx会强行关闭这个长连接，逼迫客户端不得不重新建立新的长连接。

对于第二点
为了让nginx和后端server（nginx称为upstream）之间保持长连接，典型设置如下：（默认nginx访问后端都是用的短连接(HTTP1.0)，一个请求来了，Nginx 新开一个端口和后端建立连接，后端执行完毕后主动关闭该链接）
ttp {
    upstream  BACKEND {
        server   192.168.0.1：8080  weight=1 max_fails=2 fail_timeout=30s;
        server   192.168.0.2：8080  weight=1 max_fails=2 fail_timeout=30s;
        keepalive 300;        // 这个很重要！
    }
server {
        listen 8080 default_server;
        server_name "";
        location /  {
            proxy_pass http://BACKEND;
            proxy_set_header Host  $Host;
            proxy_set_header x-forwarded-for $remote_addr;
            proxy_set_header X-Real-IP $remote_addr;
            add_header Cache-Control no-store;
            add_header Pragma  no-cache;
            proxy_http_version 1.1;         // 这两个最好也设置
            proxy_set_header Connection "";
        }
    }
keepalive 含义是设置到upstream服务器的空闲keepalive连接的最大数量，当这个数量被突破时，最近使用最少的连接将被关闭，keepalive指令不会限制一个nginx worker进程到upstream服务器连接的总数量。


```







