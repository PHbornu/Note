# 安装Nginx
## 下载依赖环境
```shell
# 安装GCC,G++
$ apt instal g++
$ apt install gcc
# 下载并解压依赖软件包
$ wget https://buildpack.oss-cn-shanghai.aliyuncs.com/static/r6d/nginx/nginx-compile-lib/pcre-8.44.tar.gz

$ wget https://buildpack.oss-cn-shanghai.aliyuncs.com/static/r6d/nginx/nginx-compile-lib/zlib-1.2.11.tar.gz

$ wget https://buildpack.oss-cn-shanghai.aliyuncs.com/static/r6d/nginx/nginx-compile-lib/openssl-1.1.1l.tar.gz

```

## 下载Nginx解压,开始编译
```shell
# 注意环境的目录地址,根据自己的情况自用选择
$ ./configure \
--prefix=/usr/local/nginx \
--with-http_ssl_module \
--with-http_dav_module \
--with-openssl=../openssl-1.1.1l \
--with-pcre=../pcre-8.44 \
--with-zlib=../zlib-1.2.11
```

## 编译和安装
```shell
$ make && make install
```




# Nginx目录索引文件下载

## 说明
```
官方文档
http://nginx.org/en/docs/http/ngx_http_autoindex_module.html
```
```
Syntax:    autoindex on | off;
Default:    
autoindex off;
Context:    http, server, location


# autoindex on   ； 表示开启目录索引


Syntax:    autoindex_localtime on | off;
Default:    
autoindex_localtime off;
Context:    http, server, location


# autoindex_localtime on; 显示文件为服务器的时间


Syntax:    autoindex_exact_size on | off;
Default:    
autoindex_exact_size on;
Context:    http, server, location

# autoindex_exact_size on; 显示确切bytes单位
# autoindex_exact_size off; 显示文件大概单位，是KB、MB、GB


若目录有中文，nginx.conf中添加utf8编码
charset utf-8,gbk;
```

## 配置文件
```conf
server{

    listen 8888;
    server_name localhost;
    charset utf-8,gbk;
    location / {
        # root E:\download;
        root /yuchaoit/;
        autoindex on;
        autoindex_localtime on;
        autoindex_exact_size off;
    }
}
```


# 反向代理
服务端反向代理解决跨域问题

```conf
    server {
        listen       80;    #nginx监听端口
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
		
        location / {
            root   html;
            index  index.html index.htm;
        }
		location /Air{
			proxy_pass http://localhost:3000;   #nodejs服务端监听端口
			add_header Access-Control-Allow-Origin *;   #跨域头部，星号运行全部
            #这里`Air`是`nodesjs`路由api
		}
    }
```

# 搭建WebDav
[库文件下载地址](https://github.com/arut/nginx-dav-ext-module)

``` shell

# 检查nginx是否包含http_dav_module 模块
# 如果输出包含 with-http_dav_module，则模块已启用。否则，你需要重新编译 Nginx 并添加此模
nginx -V 2>&1 | grep -o with-http_dav_module
块。

# 添加库文件
--with-http_dav_module --add-dynamic-module=../nginx-dav-ext-module


# webdav配置

```config
        location /webdav {
            # 存储路径
            root /home/;

            # 用户名 密码
            auth_basic "Restricted";
            auth_basic_user_file /etc/nginx/htpasswd;
            
            dav_methods PUT DELETE MKCOL COPY MOVE;         # DAV支持的请求方法
            dav_ext_methods PROPFIND OPTIONS LOCK UNLOCK;   #DAV扩展支持的请求方法
            dav_access user:rw group:rw all:rw;             #设置创建的文件及目录的访问权限
            create_full_put_path on;                        #启用创建目录支持
            dav_ext_lock zone=davlock;                      # DAV扩展锁绑定的内存区域
        }
```

```
## 错误信息
1、./configure: error: the HTTP XSLT module requires the libxml2/libxslt
libraries. You can either do not enable the module or install the libraries.
```
apt-get install libxslt-dev
```


# 搭建推流直播

|协议|URL 格式|延迟|优点|缺点|
|---|---|---|---|---|
|RTMP   |rtmp://ip/live/test   |1-3 秒   |极低延迟 |浏览器不支持，需播放器 |				
|HTTP-FLV	|http://ip/live/test.flv	|2-5 秒	|低延迟，网页可播	|需 flv.js 支持|
|HLS	|http://ip/hls/test.m3u8	|10-30 秒	|兼容性最好	|延迟高


## 方式一：VLC 播放
打开 VLC -> 媒体 -> 打开网络串流。
输入：rtmp://你的IP/live/test
回车，画面出来了！

## 方式二：浏览器播放 HLS（Safari/Edge）
直接在地址栏输入：http://你的IP/hls/test.m3u8
Safari 和 Edge 原生支持 HLS，直接就能看。Chrome 需要插件。

## 方式三：网页播放 HTTP-FLV
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Nginx-RTMP 直播</title>
    <script src="https://cdn.bootcss.com/flv.js/1.5.0/flv.min.js"></script>
</head>
<body>
    <h1>我的直播间</h1>
    <video id="videoElement" controls width="800" height="450"></video>
    <script>
        if (flvjs.isSupported()) {
            var videoElement = document.getElementById('videoElement');
            var flvPlayer = flvjs.createPlayer({
                type: 'flv',
                url: 'http://你的IP/live/test.flv' // 注意这里是 flv
            });
            flvPlayer.attachMediaElement(videoElement);
            flvPlayer.load();
            flvPlayer.play();
        }
    </script>
</body>
</html>
```

## 配置文件
```conf
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}

rtmp {
    server {
        listen 1935; # RTMP 默认端口
        chunk_size 4096;

        application live {
            live on;             # 开启直播模式
            record off;          # 关闭录制（节省硬盘）
            allow publish 127.0.0.1; # 允许推流的客户端 IP，生产环境建议改成内网 IP 或密码验证
            
            # 开启 HLS 切片
            hls on;
            #hls_path /tmp/hls;   # Ubuntu 路径
            hls_path E:/nginx/nginx-rtmp-win32-1.2.1/temp/hls; # Windows 路径，注意反斜杠
            hls_fragment 3s;     # 每个切片 3 秒
            hls_playlist_length 10s; # 播放列表长度 10 秒
        }
    }
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       80; # HTTP 默认端口，用于播放 HLS
        server_name  localhost;

        # HLS 播放配置
        location /hls {
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            #root /tmp; # Ubuntu
            root E:/nginx/nginx-rtmp-win32-1.2.1/temp; # Windows
            add_header Cache-Control no-cache;
			
        }
		location /live {
			#网页播放 HTTP-FLV
			flv_live on;
			chunked_transfer_encoding on;
			add_header 'Access-Control-Allow-Origin' '*';
		}
    }
}
```
