[docker 之基于Dockerfile构建 Nginx+ php7](http://blog.csdn.net/guyan0319/article/details/79633904)

**前言**

本文是将Nginx 和PHP7安装在不同容器，同–link实现交互通信，网上查有些资料将LNMP装在一个容器，我认为装在一起固然简单一些，但不够灵活（纯个人看法，有不同观点，欢迎回复）。 
系统环境：CentOS Linux release 7.4.1708 
一、构建nginx 
1、创建目录/var/docker/

```
mkdir -p /var/docker
```

从 [github](https://github.com/guyan0319/docker_lnmp) 拉取相关配置文件 ，将nginx文件夹复制到/var/docker/下。 
/var/docker/nginx 目录结构为

```
[root@idddfwetddffa nginx]# ll
-rw-r--r-- 1 root root  682 Mar 20 22:19 Dockerfile
-rw-r--r-- 1 root root 2572 Mar 20 16:30 nginx.conf
-rw-r--r-- 1 root root 8011 Mar 20 22:17 supervisord.conf
-rw-r--r-- 1 root root  351 Mar 20 22:19 www.conf
123456
```

查看Dockerfile （注意，Dockerfile名字第一个字母必须为大写）

```
FROM centos:7.4.1708
MAINTAINER 115946156@qq.com
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
#安装YUM源
RUN rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
RUN rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
#安装nginx
RUN yum -y install gcc gcc-c++ \
openssl openssl-devel \
zlib zlib-devel \
nginx supervisor
EXPOSE 80
RUN rm -rf /var/lib/yum/history/*.sqlite &&\
rm -rf /var/cache/yum
RUN rm -f /etc/supervisord.conf
COPY supervisord.conf /etc/
COPY www.conf /etc/nginx/conf.d/
COPY nginx.conf /etc/nginx/
#CMD ["/usr/sbin/nginx","-g","daemon off;"]
CMD ["/usr/bin/supervisord","-c","/etc/supervisord.conf"]
123456789101112131415161718192021
```

nginx.conf

```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;


     fastcgi_connect_timeout 300;
     fastcgi_send_timeout 300;
     fastcgi_read_timeout 300;
     fastcgi_buffer_size 64k;
     fastcgi_buffers 4 64k;


    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80;
        server_name  localhost;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2 default_server;
#        listen       [::]:443 ssl http2 default_server;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers HIGH:!aNULL:!MD5;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596
```

supervisord.conf （nginx进程管理软件）

```
; Sample supervisor config file.

[unix_http_server]
file=/var/run/supervisor/supervisor.sock   ; (the path to the socket file)
;chmod=0700                 ; sockef file mode (default 0700)
;chown=nobody:nogroup       ; socket file uid:gid owner
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

;[inet_http_server]         ; inet (TCP) server disabled by default
;port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

[supervisord]
logfile=/var/log/supervisor/supervisord.log  ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB       ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10          ; (num of main logfile rotation backups;default 10)
loglevel=info               ; (log level;default info; others: debug,warn,trace)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
nodaemon=true              ; (start in foreground if true;default false)
minfds=1024                 ; (min. avail startup file descriptors;default 1024)
minprocs=200                ; (min. avail process descriptors;default 200)
;umask=022                  ; (process file creation umask;default 022)
;user=chrism                 ; (default is current user, required if root)
;identifier=supervisor       ; (supervisord identifier, default is 'supervisor')
;directory=/tmp              ; (default is not to cd during start)
;nocleanup=true              ; (don't clean up tempfiles at start;default false)
;childlogdir=/tmp            ; ('AUTO' child log dir, default $TEMP)
;environment=KEY=value       ; (key value pairs to add to environment)
;strip_ansi=false            ; (strip ansi escape codes in logs; def. false)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor/supervisor.sock ; use a unix:// URL  for a unix socket
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
;username=chris              ; should be same as http_username if set
;password=123                ; should be same as http_password if set
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")
;history_file=~/.sc_history  ; use readline history if available

; The below sample program section shows all possible program subsection values,
; create one or more 'real' program: sections to be able to control them under
; supervisor.

;[program:theprogramname]
;command=/bin/cat              ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=999                  ; the relative start priority (default 999)
;autostart=true                ; start at supervisord start (default: true)
;autorestart=true              ; retstart at unexpected quit (default: true)
;startsecs=10                  ; number of secs prog must stay running (def. 1)
;startretries=3                ; max # of serial start failures (default 3)
;exitcodes=0,2                 ; 'expected' exit codes for process (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=true          ; redirect proc stderr to stdout (default false)
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A=1,B=2           ; process environment additions (def no adds)
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample eventlistener section shows all possible
; eventlistener subsection values, create one or more 'real'
; eventlistener: sections to be able to handle event notifications
; sent by supervisor.

;[eventlistener:theeventlistenername]
;command=/bin/eventlistener    ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;events=EVENT                  ; event notif. types to subscribe to (req'd)
;buffer_size=10                ; event buffer queue size (default 10)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=-1                   ; the relative start priority (default -1)
;autostart=true                ; start at supervisord start (default: true)
;autorestart=unexpected        ; restart at unexpected quit (default: unexpected)
;startsecs=10                  ; number of secs prog must stay running (def. 1)
;startretries=3                ; max # of serial start failures (default 3)
;exitcodes=0,2                 ; 'expected' exit codes for process (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=true          ; redirect proc stderr to stdout (default false)
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups        ; # of stderr logfile backups (default 10)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A=1,B=2           ; process environment additions
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample group section shows all possible group values,
; create one or more 'real' group: sections to create "heterogeneous"
; process groups.

;[group:thegroupname]
;programs=progname1,progname2  ; each refers to 'x' in [program:x] definitions
;priority=999                  ; the relative start priority (default 999)

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[program:nginx]
command=/usr/sbin/nginx -g 'daemon off;'


[include]
files = supervisord.d/*.ini
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134
```

主要修改 nodaemon=no改为nodaemon=true ，即关闭后台运行，因为docker容器内的supervisord服务必须运行在前台，如果运行在后台，容器会在服务启动完后退出。 
增加

```
[program:nginx]
command=/usr/sbin/nginx -g 'daemon off;'
```

而对于nginx来说： 
daemon off表示将后台运行关闭了，运行在前台，supervisord监控前台运行的程序。

www.conf

```
server {
    listen   80;
    server_name  www.gzqiang.cn;
    root /var/www;
    index index.htm index.html index.php;

    location ~ \.php$ {
        root /var/www;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    } 
}
123456789101112131415
```

将 www.gzqiang.cn 改成你自己的域名

2、 开始进行构建nginx：

在 /var/docker/nginx 下执行

```
docker build -t nginx .
```

二、构建 php7.2 
1、同上将php文件复制到 /var/docker下

目录为

```
-rw-r--r-- 1 root root 1229 Mar 20 22:33 Dockerfile
-rw-r--r-- 1 root root 8052 Mar 20 22:33 supervisord.conf
123
```

Dockerfile

```
FROM centos:7.4.1708
MAINTAINER 115946156@qq.com
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
#安装YUM源
RUN rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm 
RUN rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm 
#安装php-fpm
RUN yum -y install gcc gcc-c++ \
openssl openssl-devel \
zlib zlib-devel \
php72w-xml \
php72w-gd \
php72w-cli \
php72w-fpm \
php72w-pear \
php72w-mysqlnd \
php72w-devel \
php72w-mcrypt \
php72w-mbstring \
php72w-common \
php72w-process \
php72w-pdo \
php72w-opcache \
php72w-intl \
php72w-pecl-imagick \
php72w-pecl-redis \
php72w-pecl-memcached \
php72w-pecl-mongodb
EXPOSE 9000
RUN sed -i 's@daemonize = yes@daemonize = no@g' /etc/php-fpm.conf
RUN sed -i -e 's@listen = 127.0.0.1:9000@listen = 0.0.0.0:9000@g' /etc/php-fpm.d/www.conf
RUN sed -i -e 's@listen.allowed_clients = 127.0.0.1@;listen.allowed_clients = 127.0.0.1@g' /etc/php-fpm.d/www.conf
#安装supervisor
RUN yum -y install supervisor
RUN rm -rf /var/lib/yum/history/*.sqlite &&\
rm -rf /var/cache/yum
RUN rm -f /etc/supervisord.conf
ADD supervisord.conf /etc/
#CMD ["/usr/sbin/init"]
#CMD ["/usr/sbin/php-fpm"]
CMD ["/usr/bin/supervisord","-c","/etc/supervisord.conf"]
123456789101112131415161718192021222324252627282930313233343536373839404142
```

RUN sed -i ‘s@daemonize = yes@daemonize = no@g’ /etc/php-fpm.conf 
这里也是将daemon模式关闭了，php-fpm运行到前台。 
RUN sed -i -e ‘s@listen = 127.0.0.1:9000@listen = 0.0.0.0:9000@g’ /etc/php-fpm.d/www.conf 
fpm监听端口，即nginx中php处理的地址。 
RUN sed -i -e ‘s@listen.allowed_clients = 127.0.0.1@;listen.allowed_clients = 127.0.0.1@g’ /etc/php-fpm.d/www.conf 
允许访问FastCGI进程的IP，注释掉，默认为any

supervisord.conf

```
; Sample supervisor config file.

[unix_http_server]
file=/var/run/supervisor/supervisor.sock   ; (the path to the socket file)
;chmod=0700                 ; sockef file mode (default 0700)
;chown=nobody:nogroup       ; socket file uid:gid owner
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

;[inet_http_server]         ; inet (TCP) server disabled by default
;port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

[supervisord]
logfile=/var/log/supervisor/supervisord.log  ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB       ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10          ; (num of main logfile rotation backups;default 10)
loglevel=info               ; (log level;default info; others: debug,warn,trace)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
nodaemon=true              ; (start in foreground if true;default false)
minfds=1024                 ; (min. avail startup file descriptors;default 1024)
minprocs=200                ; (min. avail process descriptors;default 200)
;umask=022                  ; (process file creation umask;default 022)
;user=chrism                 ; (default is current user, required if root)
;identifier=supervisor       ; (supervisord identifier, default is 'supervisor')
;directory=/tmp              ; (default is not to cd during start)
;nocleanup=true              ; (don't clean up tempfiles at start;default false)
;childlogdir=/tmp            ; ('AUTO' child log dir, default $TEMP)
;environment=KEY=value       ; (key value pairs to add to environment)
;strip_ansi=false            ; (strip ansi escape codes in logs; def. false)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor/supervisor.sock ; use a unix:// URL  for a unix socket
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
;username=chris              ; should be same as http_username if set
;password=123                ; should be same as http_password if set
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")
;history_file=~/.sc_history  ; use readline history if available

; The below sample program section shows all possible program subsection values,
; create one or more 'real' program: sections to be able to control them under
; supervisor.

;[program:theprogramname]
;command=/bin/cat              ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=999                  ; the relative start priority (default 999)
;autostart=true                ; start at supervisord start (default: true)
;autorestart=true              ; retstart at unexpected quit (default: true)
;startsecs=10                  ; number of secs prog must stay running (def. 1)
;startretries=3                ; max # of serial start failures (default 3)
;exitcodes=0,2                 ; 'expected' exit codes for process (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=true          ; redirect proc stderr to stdout (default false)
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A=1,B=2           ; process environment additions (def no adds)
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample eventlistener section shows all possible
; eventlistener subsection values, create one or more 'real'
; eventlistener: sections to be able to handle event notifications
; sent by supervisor.

;[eventlistener:theeventlistenername]
;command=/bin/eventlistener    ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;events=EVENT                  ; event notif. types to subscribe to (req'd)
;buffer_size=10                ; event buffer queue size (default 10)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=-1                   ; the relative start priority (default -1)
;autostart=true                ; start at supervisord start (default: true)
;autorestart=unexpected        ; restart at unexpected quit (default: unexpected)
;startsecs=10                  ; number of secs prog must stay running (def. 1)
;startretries=3                ; max # of serial start failures (default 3)
;exitcodes=0,2                 ; 'expected' exit codes for process (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=true          ; redirect proc stderr to stdout (default false)
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups        ; # of stderr logfile backups (default 10)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A=1,B=2           ; process environment additions
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample group section shows all possible group values,
; create one or more 'real' group: sections to create "heterogeneous"
; process groups.

;[group:thegroupname]
;programs=progname1,progname2  ; each refers to 'x' in [program:x] definitions
;priority=999                  ; the relative start priority (default 999)

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[program:php-fpm]
;command=bash -c "sleep 1 && systemctl start php-fpm"
command=/usr/sbin/php-fpm


[include]
files = supervisord.d/*.ini
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135
```

nodaemon=true 
关闭后台运行。

[program:php-fpm] 
command=/usr/sbin/php-fpm 
增加启动php-fpm

2、构建php7.2

```
docker build -t php .
```

三、查看生成的镜像

![这里写图片描述](http://img-blog.csdn.net/2018032116470053?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2d1eWFuMDMxOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

四、利用构建的镜像启动php、nginx服务 
运行php

```
docker run -d --name=php   -v /var/www:/var/www php

```

记住加 -v /var/www:/var/www 即把宿主主机目录/var/www 映射到容器 /var/www ，如果不设置会出现 file not found的错误

```
docker run -d --name=nginx -p80:80 -v /var/www:/var/www --link=php:php nginx 
```

这里 –link=php:php 实现了nginx和php容器的互通

查看运行容器

![这里写图片描述](http://img-blog.csdn.net/20180321165344571?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2d1eWFuMDMxOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

五、测试

在/var/www目录下，新建index.php

```
<?php
phpinfo();

```

访问 <http://www.gzqiang.cn/index.php> 
![这里写图片描述](http://img-blog.csdn.net/20180321165706327?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2d1eWFuMDMxOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

进入php和nginx容器查看是如何通信的 
php 

![这里写图片描述](http://img-blog.csdn.net/20180321165958211?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2d1eWFuMDMxOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

nginx 

![这里写图片描述](http://img-blog.csdn.net/20180321170605567?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2d1eWFuMDMxOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

本文配置文件放在github： <https://github.com/guyan0319/docker_lnmp>
