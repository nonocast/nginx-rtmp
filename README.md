## prepare
```
# check system version
$ lsb_release -ds
Debian GNU/Linux 10 (buster)

$ sudo apt install -y build-essential git tree software-properties-common dirmngr apt-transport-https ufw
# optional dependencies
$ sudo apt install -y perl libperl-dev libgd3 libgd-dev libgeoip1 libgeoip-dev geoip-bin libxml2 libxml2-dev libxslt1.1 libxslt1-dev

# PCRE version 8.42
$ wget https://ftp.pcre.org/pub/pcre/pcre-8.42.tar.gz && tar xzvf pcre-8.42.tar.gz

# zlib version 1.2.11
$ wget https://www.zlib.net/zlib-1.2.11.tar.gz && tar xzvf zlib-1.2.11.tar.gz

# OpenSSL version 1.1.1a
$ wget https://www.openssl.org/source/openssl-1.1.1a.tar.gz && tar xzvf openssl-1.1.1a.tar.gz
```


## compile
```
$ mkdir nginx && cd $_
$ wget http://nginx.org/download/nginx-1.17.4.tar.gz
$ tar -zxvf nginx-1.17.4.tar.gz
$ ./configure --help
$ ./configure --with-pcre=../pcre-8.42 --with-zlib=../zlib-1.2.11 --with-openssl=../openssl-1.1.1a
$ make
$ sudo make install
```

说明:
- `configure --help`可以查看编译配置，打开关闭了哪些模块
- `--prefix=PATH` 设置程序安装路径, 默认/usr/local/nginx
- `--sbin-path=PATH` 设置可执行文件路径, 默认`prefix`/sbin/nginx
- `--config-path=PATH` 设置配置文件路径, 默认`prefix`/conf/nginx.conf

install以后安装/usr/local/nginx
```
$ pwd
/usr/local/nginx
$ ls
conf  html  logs  sbin
$ nginx tree -L 2 .
.
|-- conf
|   |-- fastcgi.conf
|   |-- fastcgi.conf.default
|   |-- fastcgi_params
|   |-- fastcgi_params.default
|   |-- koi-utf
|   |-- koi-win
|   |-- mime.types
|   |-- mime.types.default
|   |-- nginx.conf
|   |-- nginx.conf.default
|   |-- scgi_params
|   |-- scgi_params.default
|   |-- uwsgi_params
|   |-- uwsgi_params.default
|   `-- win-utf
|-- html
|   |-- 50x.html
|   `-- index.html
|-- logs
`-- sbin
    `-- nginx

4 directories, 18 files

$ /usr/local/nginx/sbin/nginx -v
nginx version: nginx/1.17.4

# link到系统路径下
$ sudo ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx 
$ sudo nginx -v
nginx version: nginx/1.17.4

# START
$ sudo nginx
```

然后浏览器 http://ip:port 就可以看到Welcome to nginx!

也可以用nc(netcat)来检查
```
~ nc localhost 80
GET /
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## NRM (nginx-rtmp-module)
> NGINX-based Media Streaming Server

Features
- RTMP/HLS/MPEG-DASH live streaming
- RTMP Video on demand FLV/MP4, playing from local filesystem or HTTP
- Stream relay support for distributed streaming: push & pull models
- Recording streams in multiple FLVs
- H264/AAC support
- Online transcoding with FFmpeg
- HTTP callbacks (publish/play/record/update etc)
- Running external programs on certain events (exec)
- HTTP control module for recording audio/video and dropping clients
- Advanced buffering techniques to keep memory allocations at a minimum level for faster streaming and low memory footprint
- Proved to work with Wirecast, FMS, Wowza, JWPlayer, FlowPlayer, StrobeMediaPlayback, ffmpeg, avconv, rtmpdump, flvstreamer -and many more
- Statistics in XML/XSL in machine- & human- readable form
- Linux/FreeBSD/MacOS/Windows

简单来说这个模块就是nginx+ffmpeg, 实现了rtmp, hls, mpeg-dash直播, 支持rtmp, hls点播, 支持录播。

```
$ git clone https://github.com/arut/nginx-rtmp-module.git

# 重新回炉编译
$ ./configure --with-pcre=../pcre-8.42 --with-zlib=../zlib-1.2.11 --with-openssl=../openssl-1.1.1a --add-module=../nginx-rtmp-module 
# 这时会在ngx_rmtp_eval.c:170:13处有一个warning, 所以需要加上忽略warning的设置
$ ./configure --with-pcre=../pcre-8.42 --with-zlib=../zlib-1.2.11 --with-openssl=../openssl-1.1.1a --add-module=../nginx-rtmp-module --with-debug --with-cc-opt="-Wimplicit-fallthrough=0"

# configure中会显示module信息
configuring additional modules
adding module in ../nginx-rtmp-module
 + ngx_rtmp_module was configured
 
$ make
$ sudo make install
$ sudo nginx -V
nginx version: nginx/1.17.4
built by gcc 8.3.0 (Debian 8.3.0-6) 
built with OpenSSL 1.1.1a  20 Nov 2018
TLS SNI support enabled
configure arguments: --with-pcre=../pcre-8.42 --with-zlib=../zlib-1.2.11 --with-openssl=../openssl-1.1.1a --add-module=../nginx-rtmp-module --with-debug --with-cc-opt=-Wimplicit-fallthrough=0
```
## VOD
nginx.conf
```
rtmp { 
  server { 
    listen 1935; 
    chunk_size: 4096; 
    application vod { 
      play /var/video; 
    }
  } 
}     
```

`sudo nginx -s reload`后vlc rtmp://192.168.3.132/vod/envoy.mp4 就可以播放视频了。

ffplay也ok,
```
# fs means fullscreen
ffplay -fs rtmp://192.168.3.132/vod/envoy.mp4
ffplay version 4.0 Copyright (c) 2003-2018 the FFmpeg developers
  built with Apple LLVM version 9.1.0 (clang-902.0.39.1)
  configuration: --prefix=/usr/local/Cellar/ffmpeg/4.0 --enable-shared --enable-pthreads --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags= --host-ldflags= --enable-gpl --enable-ffplay --enable-libmp3lame --enable-libx264 --enable-libxvid --enable-opencl --enable-videotoolbox --disable-lzma
  libavutil      56. 14.100 / 56. 14.100
  libavcodec     58. 18.100 / 58. 18.100
  libavformat    58. 12.100 / 58. 12.100
  libavdevice    58.  3.100 / 58.  3.100
  libavfilter     7. 16.100 /  7. 16.100
  libavresample   4.  0.  0 /  4.  0.  0
  libswscale      5.  1.100 /  5.  1.100
  libswresample   3.  1.100 /  3.  1.100
  libpostproc    55.  1.100 / 55.  1.100
Input #0, flv, from 'rtmp://192.168.3.132/vod/envoy.mp4':0B f=0/0   
  Metadata:
    displayWidth    : 1280
    displayHeight   : 720
  Duration: 00:02:20.44, start: 0.000000, bitrate: N/A
    Stream #0:0: Video: h264 (High), yuv420p(tv, bt709, progressive), 1280x720 [SAR 1:1 DAR 16:9], 24.42 fps, 23.98 tbr, 1k tbn, 47.95 tbc
    Stream #0:1: Audio: aac (LC), 48000 Hz, stereo, fltp
```


## LIVE

```
rtmp { 
  server { 
    listen 1935; 
    chunk_size: 4096; 
    application vod { 
      play /var/video; 
    }
    application live {
      live on;
    }
  } 
}     
```
通过ffmpeg推流
```
$ ffmpeg -i envoy.mp4 -vcodec libx264 -acodec aac -f flv rtmp://192.168.3.132:1935/live/envoy
```
-f flv是指rtmp
然后再次通过vlc/ffplay验证即可。

注: 这个1935可以省略, nginx会自动根据protocol判定。

## HLS
```
# HLS

# For HLS to work please create a directory in tmpfs (/tmp/hls here)
# for the fragments. The directory contents is served via HTTP (see
# http{} section in config)
#
# Incoming stream must be in H264/AAC. For iPhones use baseline H264
# profile (see ffmpeg example).
# This example creates RTMP stream from movie ready for HLS:
#
# ffmpeg -loglevel verbose -re -i movie.avi  -vcodec libx264
#    -vprofile baseline -acodec libmp3lame -ar 44100 -ac 1
#    -f flv rtmp://localhost:1935/hls/movie
#
# If you need to transcode live stream use 'exec' feature.
#
application hls {
    live on;
    hls on;
    hls_path /tmp/hls;
}

server {
  location /hls {
    # Serve HLS fragments
    types {
        application/vnd.apple.mpegurl m3u8;
        video/mp2t ts;
    }
    root /tmp;
    add_header Cache-Control no-cache;
  }
}
```
rtmp/application/hls对应rtmp, 而server/location/hls则对应http

重新推一下rtmp, 注意下推流的application name，这里对应hls
```
$ ffmpeg -i Teams.mp4 -vcodec libx264 -acodec aac -f flv rtmp://192.168.3.132/hls/teams
```
然后检查/tmp/hls
```
# ls /tmp/hls
teams-0.ts  teams-1.ts	teams-2.ts  teams-3.ts	teams-4.ts  teams-5.ts	teams-6.ts  teams-7.ts	teams.m3u8
```

电脑上检查,
```
ffplay http://192.168.3.132/hls/teams.m3u8
```
最后一步，掏出手机，打开safari访问`http://192.168.3.132/hls/teams.m3u8`，就可以看到直播的视频。
补充一句, hls的兼容性问题:
- iOS和Mac OSX的safari默认支持
- Chrome插件方式
- 通过[video.js](https://github.com/videojs/video.js)适配， 无敌

## STAT
在server中加入,
```
# This URL provides RTMP statistics in XML
location /stat {
  rtmp_stat all;

  # Use this stylesheet to view XML as web page
  # in browser
  rtmp_stat_stylesheet stat.xsl;
}

location /stat.xsl {
  # XML stylesheet to view RTMP stats.
  # Copy stat.xsl wherever you want
  # and put the full directory path here
  root /path/to/stat.xsl/;
}
```

## 防盗链和鉴权
还记得前面feature中的
> - HTTP callbacks (publish/play/record/update etc)
通过play的callback到nodejs api来验证token有效性即可。

[nginx-rtmp-module 权限控制 - iam_shuaidaile的博客 - CSDN博客](https://blog.csdn.net/iam_shuaidaile/article/details/50599943)
[nginx+rtmp | Hexo&Magic](https://magic-king.net/2019/05/17/nginx-rtmp/)
[nginx rtmp module添加鉴权机制 - 程序园](http://www.voidcn.com/article/p-kafkptvr-bpu.html)

参考阅读:
- [How to Build Nginx from source on Debian 9](https://www.howtoforge.com/how-to-build-nginx-from-source-on-debian-9/)

