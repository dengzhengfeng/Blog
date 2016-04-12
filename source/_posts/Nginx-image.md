title: 使用nginx给图片加水印
date: 2016-04-12 12:25:35
tags: ["nginx"]
categories: ["nginx"]
---
有个需求要给图片加水印，（同时研究了一下nginx的配置）这里加水印的方法是使用Nginx-image-filter-watermak模块
<h2 id="theory">原理</h2>
>详见[github地址](https://github.com/intaro/nginx-image-filter-watermark)

因为整体简单，我就直接写配置了

<h2 id="setup">编译安装</h2>
>编译nginx（以Fedora为例）

下载nginx源码（当前稳定版）
`wget http://nginx.org/download/nginx-1.8.1.tar.gz`

解压
`tar zxvf nginx-1.8.1.tar.gz`

下载Nginx-image-filter-watermak
`git clone https://github.com/intaro/nginx-image-filter-watermark.git`

复制文件
`cp nginx-image-filter-watermark/ngx_http_image_filter_module.c nginx-1.8.1/src/http/modules/`

解决依赖
`sudo dnf install gd-devel pcre-devel libcurl-devel`
或
`sudo yum install gd-devel pcre-devel libcurl-devel`

编译nginx
`cd nginx-1.8.1`
`./configure --with-http_image_filter_module --prefix=<path> /usr/local/nginx`（prefix为安装地址）
`make`
`sudo make install`

<h2 id="conf">配置</h2>
>配置文件在nginx的安装目录中的conf下的nginx.conf

location /img/ {
image_filter watermark;#开启图片水印功能

image_filter_jpeg_quality   95;#图片质量
image_filter_buffer         2M;#缓冲大小，如水印文件大时要提高大小

image_filter_watermark_width_from 400; #当图片宽度大于400时才打水印
image_filter_watermark_height_from 400;#当图片高度大于400时才打水印

image_filter_watermark "PATH_TO_FILE";#水印文件路径（使用png格式）
image_filter_watermark_position center-center;#水印位置
｝

<h2 id="remark">备注</h2>
如果需要更复杂的功能，可以使用这个[模块](https://github.com/3078825/ngx_image_thumb)
nginx启动脚可以参考如下

``` +bash
#!/bin/bash
# chkconfig: - 85 15
# description: nginx is a World Wide Web server. It is used to serve
# Source Function Library
. /etc/init.d/functions

# Nginx Settings
NGINX_SBIN="/usr/local/nginx/sbin/nginx"
NGINX_CONF="/usr/local/nginx/conf/nginx.conf"
NGINX_PID="/usr/local/nginx/logs/nginx.pid"

RETVAL=0
prog="Nginx"

start() {
        echo -n $"Starting $prog: "
        mkdir -p /dev/shm/nginx_temp
        daemon $NGINX_SBIN -c $NGINX_CONF
        RETVAL=$?
        echo
        return $RETVAL
}

stop() {
        echo -n $"Stopping $prog: "
        killproc -p $NGINX_PID $NGINX_SBIN -TERM
        rm -rf /dev/shm/nginx_temp
        RETVAL=$?
        echo
        return $RETVAL
}

reload(){
        echo -n $"Reloading $prog: "
        killproc -p $NGINX_PID $NGINX_SBIN -HUP
        RETVAL=$?
        echo
        return $RETVAL
}

restart(){
        stop
        start
}

configtest(){
    $NGINX_SBIN -c $NGINX_CONF -t
    return 0
}

case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  reload)
        reload
        ;;
  restart)
        restart
        ;;
  configtest)
        configtest
        ;;
  *)
        echo $"Usage: $0 {start|stop|reload|restart|configtest}"
        RETVAL=1
esac

exit $RETVAL
```
