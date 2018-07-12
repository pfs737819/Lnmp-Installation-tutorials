# LNMP环境搭建
> ###  centos7 nginx1.12.0 mysql5.7 php7

---
### Nginx编译安装

* 1 . gcc 安装
 安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：
    ```
    yum install gcc-c++               
    ```

* 2 . PCRE pcre-devel 安装
PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：
    ```
    yum install -y pcre pcre-devel
    ```

* 3 . zlib 安装
zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。
    ```
    yum install -y zlib zlib-devel
    ```

* 3 . OpenSSL 安装
OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。
    ```
    yum install -y openssl openssl-devel
    ```
    
* 4 .  Nginx安装包下载 使用wget命令下载（推荐）
* 确保系统已经安装了wget，如果没有安装，执行 yum install wget 安装,我这里下载的目录是 '/usr/local/src/' ,首先进入此目录,再执行wget 进行下载
     
    ```
    cd /usr/local/src/

    wget -c https://nginx.org/download/nginx-1.12.0.tar.gz
    
    ```

* 5 . 解压nginx安装包,进入到该目录中
    ```
    tar -zxvf nginx-1.12.0.tar.gz

    cd nginx-1.12.0
    ```

* 6 . 预安装配置
 ```
 ./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_stub_status_module --with-pcre
 ```

* 7 . 编译安装
```
make && make install
```

* 8 . 启动nginx
```
/usr/local/nginx/sbin/nginx
```

### php编译安装


* 1 . 因为php安装需要编译，所以服务器应该保证gcc和g++环境的安装
```
yum install  gcc gcc-c++
```

* 2 . 安装其他依赖
```
 yum install libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel  curl  curl-devel  pcre  pcre-devel
```

* 3 .安装包下载
```
cd /usr/local/src

wget http://cn2.php.net/get/php-7.2.7.tar.gz/from/this/mirror
```

* 4 .解压并进入目录
```
tar -zxvf mirror

cd php-7.2.7
```

* 5 . 预安装配置
```
 ./configure --prefix=/usr/local/php --with-curl --with-freetype-dir --with-gd --with-gettext --with-iconv-dir --with-kerberos --with-libdir=lib64 --with-libxml-dir --with-mysqli --with-openssl --with-pcre-regex --enable-pdo --with-pdo-mysql --with-pdo-sqlite --with-pear --with-png-dir --with-jpeg-dir --with-xmlrpc --with-xsl --with-zlib --with-bz2 --with-mhash --enable-fpm --enable-bcmath --enable-libxml --enable-inline-optimization --enable-gd-native-ttf --enable-mbregex --enable-mbstring --enable-opcache --enable-pcntl --enable-shmop --enable-soap --with-mcrypt --with-sqlite3 --enable-calendar --enable-ftp --with-pcre-dir  --enable-exif  --with-zlib-dir --enable-sockets --enable-sysvmsg  --enable-sysvsem  --enable-sysvshm --enable-xml --enable-zip
```
实际上这里的配置项比上述还多，可以使用 ```./configure --help ``` 命令查看所有选项，这里注意在php7中--with-mysql原生支持已经不存在了，操作都变成mysqli或者pdo了；以上这些选项在正常的php开发中完全够用了，后期如果需要，可以选择手动开启相应的模块


* 6 . 编译安装
```
make && make install
```

* 7 . 文件配置
php的默认安装位置上面已经指定为/usr/local/php，接下来配置相应的文件
```
cp php.ini-development /usr/local/php/lib/php.ini

cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf

cp sapi/fpm/php-fpm /usr/local/bin

cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf

```
然后设置php.ini，使用： ``` vim /usr/local/php/lib/php.ini ``` 打开php配置文件找到cgi.fix_pathinfo配置项，这一项默认被注释并且值为1，根据官方文档的说明，这里为了当文件不存在时，阻止Nginx将请求发送到后端的PHP-FPM模块，从而避免恶意脚本注入的攻击，所以此项应该去掉注释并设置为0

创建web用户
```
groupadd www

useradd -g www www
```

默认user和group的设置为nobody，将其改为www
```
vim /usr/local/php/etc/php-fpm.d/www.conf
```

修改完成之后，保存并退出，然后执行以下命令启动php-fpm服务：``` /usr/local/php/sbin/php-fpm ``` 启动完毕之后，php-fpm服务默认使用9000端口，使用``` netstat -tln | grep 9000 ```
可以查看端口使用情况
![image][tmp]


然后执行 ``` vim /usr/local/nginx/conf/nginx.conf ``` 编辑nginx配置文件，具体路径根据实际的nginx.conf配置文件位置编辑，下面主要修改nginx
server {}配置块中的内容，修改location块，追加index.php让nginx服务器默认支持index.php为首页


然后配置.php请求被传送到后端的php-fpm模块，默认情况下php配置块是被注释的，此时去掉注释并修改为以下内容：

这里面很多都是默认的，root是配置php程序放置的根目录，
主要修改的就是fastcgi_param中的/scripts为$document_root
修改完上面的，回到nginx.conf第一行，默认是#user nobody;  
这里要去掉注释改为user www;或者user www www;表示nginx服务器的权限为www
修改完这些保存并退出，然后重启nginx：
```
/usr/local/nginx/sbin/nginx -s stop

/usr/local/nginx/sbin/nginx 
```

### Mysql YUM安装

* 1 . 下载mysql源安装包
```
wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm
```

* 2 . 安装mysql源
```
yum localinstall mysql57-community-release-el7-8.noarch.rpm
```
检查mysql源是否安装成功
```
yum repolist enabled | grep "mysql.*-community.*"
```

* 3 . 安装mysql
```
yum install mysql-community-server
```

* 4 . 安装mysql
```
systemctl start mysqld
```
查看MySQL的启动状态
```
systemctl status mysqld
```

* 5 . 开机启动
```
systemctl enable mysqld
```
重载所有修改过的配置文件
```
systemctl daemon-reload
```

* 6 . 修改root本地登录密码
mysql安装完成之后，在/var/log/mysqld.log文件中给root生成了一个默认密码。通过下面的方式找到root默认密码，然后登录mysql进行修改：
```
grep 'temporary password' /var/log/mysqld.log

mysql -uroot -p

mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'newpassword!'; 
```
注意：mysql5.7默认安装了密码安全检查插件（validate_password），默认密码检查策略要求密码必须包含：大小写字母、数字和特殊符号，并且长度不能少于8位。否则会提示ERROR 1819 (HY000): Your password does not satisfy the current policy requirements错误


* 7 . 添加远程登录用户
默认只允许root帐户在本地登录，如果要在其它机器上连接mysql，必须修改root允许远程连接，或者添加一个允许远程连接的帐户，为了安全起见，我添加一个新的帐户：
mysql> GRANT ALL PRIVILEGES ON *.* TO 'user'@'%' IDENTIFIED BY 'youpassword!' WITH GRANT OPTION;


* 8 . 配置默认编码为utf8
修改/etc/my.cnf配置文件，在[mysqld]下添加编码配置，如下所示：
[mysqld]
character_set_server=utf8
[client]
default-character-set = utf8

重新启动mysql服务使配置生效：
```
systemctl restart mysqld
```

## 问题反馈
在使用中有任何问题，欢迎反馈给我，可以用以下联系方式跟我交流

* 邮件(654498024@qq.com)
* QQ: 654498024

