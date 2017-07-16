---
layout: post
title: "iOS直播之搭建Linux直播RTMP服务器"
date: 2015-03-20 10:37:38 +0800
comments: true
categories: 
---
iOS直播技术有很多，针对iOS平台有苹果的HLS点播。还有其他流媒体协议例如RTMP协议、RTSP协议、MMS协议等。这里要讲的是iOS和Android通用的RTMP，利用RTMP我们可以传输H264视频流，iOS或Android客户端接收到视频流后可以用FFmpeg实现H264的解码最终实现视频的播放。

我们先来完成第一步，搭建一个RTMP服务器。然后我会在另一篇博客中介绍如何通过iOS客户端利用FFMpeg技术进行推流拉流完成直播。
<!--more-->
## 软硬件环境
MacbookPro +  Paralles Ubuntu12.04虚拟机

## 下载安装包
1.Nginx服务器 http://nginx.org/download/nginx-1.4.3.tar.gz。

2.Nginx RTMP模块 https://codeload.github.com/arut/nginx-rtmp-module/zip/master， 当然如果你熟练Git可以用Git去克隆一个。

3.OpenSSL 模块 http://www.openssl.org/source/openssl-1.0.0t.tar.gz

4.PCRE 模块 ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.37.zip

5.zlib模块 http://zlib.net/zlib-1.2.8.tar.gz

如果下载链接不可用可直接百度相应关键字进入官网下载即可。下载完这些安装包依次解压。（建议解压到同一个目录下，例如我在桌面建了一个目录:~/Desktop/software）

解压命令：

```bash
tar -xvf filename 	// 以.tar.gz结尾的
unzip filename	  	// 以.zip结尾的
```

## 编译
我们要把RTMP、OpenSSL、PCRE和zlib模块编译进nginx当中。

打开Linux的终端切到Nginx安装源目录`cd ~Desktop/softwares/nginx-1.4.3`

添加nginx-rtmp-module-master、pcre-8.37、openssl-1.0.0t和zlib-1.2.8这些模块`./configure --add-module=../nginx-rtmp-module-master --with-pcre=../pcre-8.37 --with-openssl=../openssl-1.0.0t --with-zlib=../zlib-1.2.8`

诸位在敲命令的时候需注意自己各个模块解压后目录名称，根据自己的目录名称对命令进行修改

如果出现错误 error: You need a C++ compiler for C++ support 表示你需要安装gcc和g++编译器，不怕这两条命令可以帮你搞定`sudo apt-get install gcc` `sudo apt-get install g++`

如果到这一步都没有问题，接下来我们进行编译(这个过程可能有点长)`sudo make`

然后执行安装命令`sudo make install`

将如下配置复制粘贴到Nginx配置文件中(/usr/local/nginx/conf/nginx.conf)
```bash
worker_processes  1;
error_log  logs/error.log debug;
events {
    worker_connections  1024;
}
rtmp {
    server {
        listen 1935;
        application myapp {
            live on;
            #record keyframes;
            #record_path /tmp;
            #record_max_size 128K;
            #record_interval 30s;
            #record_suffix .this.is.flv;
            #on_publish http://localhost:8080/publish;
            #on_play http://localhost:8080/play;
            #on_record_done http://localhost:8080/record_done;
        }
    }
}
http {
    server {
        listen      8080;
        location /stat {
            rtmp_stat all;
            rtmp_stat_stylesheet stat.xsl;
        }
        location /stat.xsl {
            root /path/to/nginx-rtmp-module/;
        }
        location /control {
            rtmp_control all;
        }
        #location /publish {
        #    return 201;
        #}
        #location /play {
        #    return 202;
        #}
        #location /record_done {
        #    return 203;
        #}
        #location /rtmp-publisher {
        #    root /usr/local/nginx/test;
        #}
        #location / {
        #    root /usr/local/nginx/test/www;
        #}
    }
}
```

最后我们启动Nginx服务器

```
cd /usr/local/nginx/sbin
sudo ./nginx -s stop
sudo ./nginx
```

