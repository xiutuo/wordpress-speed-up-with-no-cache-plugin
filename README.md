# Howto wordpress-speed-up-with-no-cache-plugin

demo website: https://www.fitnesstotem.com/

# how to increase WordPress website speed without any WordPress cache plugin!

   The Ultimate software to Boost WordPress Speed & Performance : 
    **tengine(nginx) + redis2-nginx + srcache-nginx + redis**
   
   You know, There are a lot of good WordPress caching plugins available, 
   but we did not recommend using either of these: 
* WP Rocket (premium) 
* WP Super Cache (free) plugin 
* W3 Total Cache 
* Hyper Cache 
* Comet Cache 
* WP Fastest Cache 
* Hummingbird



## setup environments for web applications -WordPress, software list:

1. tengine (an open-source branch for Nginx from Taobao Alibaba group china)
  download [tengine-2.3.2](https://tengine.taobao.org/download/tengine-2.3.2.tar.gz)

      `wget -c https://tengine.taobao.org/download/tengine-2.3.2.tar.gz`

2. mysql
  download [mysql-8.0.19](https://cdn.mysql.com/archives/mysql-8.0/mysql-8.0.19.tar.gz)

     `wget -c https://cdn.mysql.com/archives/mysql-8.0/mysql-8.0.19.tar.gz`

3. PHP
      download [php-7.4.7](https://www.php.net/distributions/php-7.4.7.tar.xz)

      `wget -c https://www.php.net/distributions/php-7.4.7.tar.xz`

4. Wordpress
     download [wordpress last version](https://wordpress.org/latest.tar.gz)

5. other free nginx plugins 

     download [echo-nginx-module](https://github.com/openresty/echo-nginx-module)

     download [headers-more-nginx-module](https://github.com/openresty/headers-more-nginx-module)

     download [nginx-http-concat](https://github.com/alibaba/nginx-http-concat)

     download [luajit2](https://github.com/openresty/luajit2)

     download [ngx_devel_kit](https://github.com/vision5/ngx_devel_kit)

6. other Softwares

     download [openssl-1.1.1g](https://www.openssl.org/source/openssl-1.1.1g.tar.gz)

     download [redis2-nginx-module](https://github.com/openresty/redis2-nginx-module)

    download  [srcache-nginx-module](https://github.com/openresty/srcache-nginx-module)

## get all and put into /root/soft directory, Decompress!

> cd /root/soft/

> tar xvf zlib-1.2.11.tar.gz

> tar xvf openssl-1.1.1g.tar.gz

> unzip luajit2-2.1-agentzh.zip

> tar xvf tengine-2.3.2.tar.gz

> unzip echo-nginx-module-master.zi

> unzip headers-more-nginx-module-master.zip

> unzip nginx-http-concat-master.zip

> unzip ngx_devel_kit-master.zip

> unzip redis2-nginx-module-master.zip

> unzip set-misc-nginx-module-master.zip

> unzip srcache-nginx-module-master.zip

> tar xvf php-7.4.7.tar.xz

> tar xvf mysql-8.0.19.tar.gz


## before you compile from source,need to install rpms
> yum -y install ncurses-devel cmake gcc gcc-c++ bison rsync zlib zlib-devel pcre pcre-devel openssl openssl-devel xz perl perl-Data-Dumper

> yum -y install libxml2 gd libxml2-devel openssl-devel bzip2-devel libcurl curl libjpeg-turbo libjpeg-turbo-devel libpng libpng-devel freetype freetype-devel gd-devel libxslt-devel libxslt libzip-devel

## add user for mysql,php,nginx.

> useradd -M -s /sbin/nologin mysql

> useradd -M -s /sbin/nologin php-fpm

> useradd -M -s /sbin/nologin www

### install redis
> yum -y install redis

### install mysql8 from source ,surpport for systemd 

> cd /root/soft/mysql-8.0.19

> cmake . -DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_TCP_PORT=3306 \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DSYSTEMD_PID_DIR=/usr/local/mysql/run/ \
-DSYSTEMD_SERVICE_NAME=mysqld \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
-DENABLED_LOCAL_INFILE=ON \`
-DWITH_EXTRA_CHARSETS=complex \
-DENABLE_DEBUG_SYNC=OFF \
-DWITH_ZLIB=bundled \
-DINSTALL_LAYOUT=STANDALONE \
-DENABLED_PROFILING=ON \
-DMYSQL_MAINTAINER_MODE=OFF \
-DWITH_DEBUG=OFF \
-DENABLE_DOWNLOADS=OFF \
-DFORCE_INSOURCE_BUILD=ON \
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=/root/boost_1_70_0 \
-DWITH_INNODB_MEMCACHED=ON \
-DWITH_SYSTEMD=ON 

> make;
make install

initial the launch script,start with systemd
> scp /root/soft/mysql-8.0.19/scripts/mysqld.service /usr/lib/systemd/system/mysqld.service

> systemctl daemon-reload


configure mysql: /etc/my.cnf

>cat >/etc/my.cnf<<EOF \
[client] \
port            = 3306 \
socket          = /tmp/mysql.sock \
default-character-set=utf8mb4 \
[mysqld] \
port            = 3306 \
socket          = /tmp/mysql.sock \
group_concat_max_len=9000 \
character-set-server=utf8mb4 \
collation_server=utf8mb4_unicode_ci \
init_connect='SET NAMES utf8mb4' \
basedir=/usr/local/mysql/ \
#explicit_defaults_for_timestamp=0 \
event_scheduler=on \
connect_timeout=10 \
lower_case_table_names=1 \
max_connections=10000 \
max_connect_errors=3 \
open_files_limit = 400000 \
table_open_cache=600 \
skip-name-resolve \
skip-external-locking  \
max_allowed_packet = 900M \
back_log = 50 \
max_heap_table_size = 1G  \
tmp_table_size = 564M \
read_rnd_buffer_size = 1G \
max_length_for_sort_data=8096 \
max_sort_length=3000 \
sort_buffer_size = 8M \
join_buffer_size = 10M \
thread_cache_size = 300 \
ft_min_word_len = 4 \
thread_stack = 192K \
slow_query_log=on \
slow_query_log_file=/usr/local/mysql/data/mysqld_slow_query.log \
long_query_time = 2 \
log-bin=mysql-bin \
binlog_format=mixed \
binlog_cache_size = 1M \
binlog_expire_logs_seconds=5184000 \
log_bin_trust_function_creators=1 \
gtid_mode=ON  \
log-slave-updates=true \
enforce-gtid-consistency=true \
relay-log=el-test1-relay-bin \
relay_log_recovery=on \
server-id = 2 \
key_buffer_size = 384M \
bulk_insert_buffer_size = 128M \
myisam_sort_buffer_size = 128M \
myisam_max_sort_file_size = 2G \
myisam_repair_threads = 1 \
myisam_recover_options \
innodb_buffer_pool_size = 2G \
innodb_buffer_pool_instances= 16 \
transaction_isolation = REPEATABLE-READ \
innodb_data_file_path = ibdata1:500M:autoextend \
innodb_write_io_threads = 12 \
innodb_read_io_threads = 12 \
innodb_thread_concurrency = 40 \
innodb_thread_sleep_delay = 3000 \
innodb_read_ahead_threshold = 16 \
innodb_flush_log_at_trx_commit = 1 \
innodb_log_buffer_size = 8M \
innodb_log_file_size = 1G \
innodb_log_files_in_group = 2 \
innodb_max_dirty_pages_pct = 90 \
innodb_flush_method = O_DIRECT \
innodb_change_buffering=all \
innodb_adaptive_hash_index=on \
innodb_lock_wait_timeout = 120 \
innodb_spin_wait_delay=30 \
innodb_sync_spin_loops=100 \
innodb_purge_threads = 8 \
innodb_file_per_table = 1 \
[mysqldump] \
quick \
max_allowed_packet = 16M \
[mysql] \
no-auto-rehash \
[myisamchk] \
key_buffer_size = 512M \
sort_buffer_size = 512M \
read_buffer = 8M \
write_buffer = 8M \
[mysqlhotcopy] \
interactive-timeout \
[mysqld_safe] \
open-files-limit = 8192 \
EOF 

initial MySQL with none password

> /usr/local/mysql/bin/mysqld -umysql --initialize-insecure

start MySQL 
> service mysqld start

Init MySQL password

> /usr/local/mysql/bin/mysqladmin -uroot  password '111111'  \
/usr/local/mysql/bin/mysql -uroot -p'111111' <<EOF \
CREATE USER 'root'@'%' IDENTIFIED BY '111111';GRANT ALL ON *.* TO 'root'@'%'; \
flush privileges; \
EOF  \
echo "/usr/local/mysql/lib/" > /etc/ld.so.conf.d/libmysql.conf \
ldconfig


###install luajit2 from source

> cd /root/soft/luajit2-2.1-agentzh \
make install PREFIX=/usr/local/LuaJIT \
echo "/usr/local/LuaJIT/lib/" > /etc/ld.so.conf.d/libluajit.conf \
ldconfig 

###install tengine(nginx)
> cd /root/soft/tengine-2.3.2
./configure --prefix=/usr/local/nginx \
--pid-path=/var/run/nginx.pid \
--lock-path=/var/run/nginx.lock \
--with-pcre \
--with-openssl=../openssl-1.1.1g \
--add-module=../ngx_devel_kit-master \
--add-module=../set-misc-nginx-module-master \
--with-http_lua_module \
--with-luajit-inc=/usr/local/LuaJIT/include/luajit-2.1/ \
--with-luajit-lib=/usr/local/LuaJIT/lib \
--add-module=../redis2-nginx-module-master \
--add-module=../srcache-nginx-module-master \
--add-module=../nginx-http-concat-master \
--with-zlib=../zlib-1.2.11 \
--with-http_v2_module \
--add-module=../headers-more-nginx-module-master \
--add-module=../echo-nginx-module-master \
--user=www \
--group=www 

> make \
make install

start nginx with systemd 
> cat>nginx.service<<EOF \
[Unit] \
Description=The NGINX HTTP and reverse proxy server \
After=syslog.target network-online.target remote-fs.target nss-lookup.target \
Wants=network-online.target \
[Service] \
Type=forking \
PIDFile=/var/run/nginx.pid \
ExecStartPre=/usr/local/nginx/sbin/nginx -t \
ExecStart=/usr/local/nginx/sbin/nginx \
ExecReload=/usr/local/nginx/sbin/nginx -s reload \
ExecStop=/bin/kill -s QUIT $MAINPID \
PrivateTmp=true \
[Install] \
WantedBy=multi-user.target \
EOF 

### install php from source

> cd /root/soft/php-7.4.7

> ./configure  --prefix=/usr/local/php \
--with-config-file-path=/usr/local/php/etc \
--with-zlib-dir \
--with-webp \
--with-jpeg=/usr \
--enable-gd \
--with-freetype \
--enable-mbstring \
--with-expat \
--enable-xmlreader \
--enable-xmlwriter \
--enable-soap \
--enable-calendar \
--with-fpm-systemd \
--with-curl \
--with-pdo-sqlite \
--with-mysql-sock=/tmp/mysql.sock \
--enable-mysqlnd \
--with-pdo-mysql \
--with-mysqli \
--disable-rpath \
--enable-inline-optimization \
--with-zlib \
--enable-sockets \
--enable-sysvsem \
--enable-sysvshm \
--enable-pcntl \
--enable-mbregex \
--enable-exif \
--enable-bcmath \
--with-mhash \
--with-openssl \
--enable-ftp \
--with-kerberos \
--with-gettext \
--with-xmlrpc \
--with-xsl \
--enable-fpm \
--with-fpm-user=php-fpm \
--with-fpm-group=php-fpm \
--disable-fileinfo \
--disable-phpdbg \
--disable-dtrace

> make;make install;

start with systemd,init systemd script file
> scp /root/soft/php-7.4.7/sapi/fpm/php-fpm.service /usr/lib/systemd/system

config php  --- php.ini 

> scp /root/soft/php-7.4.7/php.ini-production /usr/local/php/etc/php.ini \
scp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf \
scp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf \
sed -i 's#;pid = run/php-fpm.pid#pid = /var/run/php-fpm.pid#g' /usr/local/php/etc/php-fpm.conf \
sed -i 's#;error_log = log/php-fpm.log#;error_log = /var/log/php-fpm.log#g' /usr/local/php/etc/php-fpm.conf \
sed -i 's#listen = 127.0.0.1:9000#listen = /var/run/php-fpm.socket#g' /usr/local/php/etc/php-fpm.d/www.conf \
sed -i 's#;listen.owner = php-fpm#listen.owner = www#g' /usr/local/php/etc/php-fpm.d/www.conf \
sed -i 's#;listen.group = php-fpm#listen.group = www#g' /usr/local/php/etc/php-fpm.d/www.conf \
sed -i 's#;listen.mode = 0660#listen.mode = 0660#g' /usr/local/php/etc/php-fpm.d/www.conf \

resolve the error  
ERROR: Unable to create the PID file (/usr/local/php/var/run/php-fpm.pid).: Read-only file system (30)

`sed -i 's#PIDFile=/usr/local/php/var/run/php-fpm.pid#PIDFile=/var/run/php-fpm.pid#g'  /usr/lib/systemd/system/php-fpm.service`

`systemctl daemon-reload `

## enable systemd for nginx,mysql,php
`systemctl daemon-reload `

`systemctl enable php-fpm`

`systemctl enable nginx`

`systemctl enable mysqld`

## config Nginx vhost for WordPress,it's very important!
> cat >/usr/local/nginx/conf/nginx.conf <<EOF
user  www; \
worker_processes  4; \
error_log  "pipe:rollback logs/error_log interval=1d baknum=7 maxsize=2G"; \
events { \
    worker_connections  4096; \
} \
http { \
    include       mime.types; \
    default_type  application/octet-stream; \
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" ' \
                      '$status $body_bytes_sent "$http_referer" ' \
                      '"$http_user_agent" "$http_x_forwarded_for"'; \
    access_log  logs/access.log  main; \
    access_log  "pipe:rollback logs/access_log interval=1d baknum=7 maxsize=2G"  main; \
    sendfile        on; \
    keepalive_timeout  65; \
    gzip on; \
    gzip_comp_level 3; \
    gzip_min_length 3k; \
    gzip_buffers 4 16k; \
    gzip_vary on; \
    gzip_proxied expired no-cache no-store private auth; \
    gzip_types text/plain text/css text/xml text/javascript application/x-javascript application/xml application/x-httpd-php image/jpeg image/gif image/png image/svg+xml application/rss+xml application/json; \
    gzip_disable "MSIE [1-6]\."; \
   include      /usr/local/nginx/conf/vhost/*.conf; \
} 

config www.fitnesstotem.com.conf

`ll vhost/ | grep www.fitnesstotem.com.conf`

`cat vhost/www.fitnesstotem.com.conf | grep -v '#'`

 download from [superhero gift ideas](https://www.fitnesstotem.com)

you can check the demo website: [superhero gift ideas](https://www.fitnesstotem.com)

