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
```




