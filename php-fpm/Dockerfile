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
