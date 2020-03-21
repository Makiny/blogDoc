## 前言
本文介绍在 CentOS7 上以 LAMP 方式部署 Nextcloud18 私有云盘
[scode type="share"]Nextcloud 是一个免费专业的私有云存储网盘开源项目，可以简单快速地在个人/公司电脑、服务器甚至是树莓派等设备上架设一套属于自己或团队专属的云同步网盘，从而实现跨平台跨设备文件同步、共享、版本控制、团队协作等功能。

Nextcloud 跨平台支持 Windows、Mac、Android、iOS、Linux 等平台，而且还提供了网页版以及 WebDAV 形式访问。

Nextcloud 还支持 API 和插件扩展，用户可以通过安装各种插件来增强网盘的功能，如 Markdown 编辑器、笔记、日历、任务列表、音乐播放器、文档编辑等。[/scode]
[scode type="green"]本文是 makiny 原创文章。如需转载请联系作者获得授权，并注明转载地址[/scode]

---
[scode type="lblue"]操作环境：
OS：CentOS Linux release 7.7
[/scode]

## 安装依赖项
安装部署期间需要的依赖项
```
sudo yum install -y epel-release yum-utils unzip curl wget \
bash-completion policycoreutils-python mlocate bzip2
sudo yum update -y
```

## 安装 apache
```
sudo yum install -y httpd
sudo systemctl enable httpd.service #设置开机启动
sudo systemctl start httpd.service  #启动 httpd 服务
```
放行`80/tcp`端口（这里使用 firewall 防火墙）
```
firewall-cmd --permanent --zone=public --add-port=80/tcp #开放 80/tcp 端口
firewall-cmd --reload                                    #防火墙重载
```  

firewall 常用命令
```
firewall-cmd --state                 #查看 firewall 状态
systemctl start firewalld            #启动 firewall
systemctl enable firewalld           #设置开机自启动
firewall-cmd --permanent --list-port #查看已放行端口
```

然后在浏览器输入服务器 IP，出现以下界面说明 apache 安装成功
![][1]

## 安装 php7.2
```
yum install -y centos-release-scl
yum install -y rh-php72 rh-php72-php rh-php72-php-gd rh-php72-php-mbstring \
rh-php72-php-intl rh-php72-php-pecl-apcu rh-php72-php-mysqlnd rh-php72-php-pecl-redis \
rh-php72-php-opcache rh-php72-php-imagick
```
创建符号链接
```
ln -s /opt/rh/httpd24/root/etc/httpd/conf.d/rh-php72-php.conf /etc/httpd/conf.d/
ln -s /opt/rh/httpd24/root/etc/httpd/conf.modules.d/15-rh-php72-php.conf /etc/httpd/conf.modules.d/
ln -s /opt/rh/httpd24/root/etc/httpd/modules/librh-php72-php7.so /etc/httpd/modules/
ln -s /opt/rh/rh-php72/root/bin/php /usr/bin/php
```

## 安装 MariaDB
```
yum install -y mariadb mariadb-server
systemctl enable mariadb.service
systemctl start mariadb.service
```
配置数据库，为 Nextcloud 创建数据库，并且创建数据库管理员
```
mysql_secure_installation  #设置数据库 root 密码，设置密码后提示选项，都选 ‘y’
mysql -uroot -p            #进入Mariadb，会提示输密码 输入上面设置的 root 密码
CREATE DATABASE nextcloud; #创建数据库 Mextcloud
                           #为新建的数据库创建一个非root管理员。数据库名、用户名、密码可根据情况自行设置
grant all privileges on `nextcloud`.* to 'nextcloudadmin'@'localhost' identified by 'passwd' with grant option; 
FLUSH PRIVILEGES;          #刷新权限
exit;                      #退出
```

## 安装 redis
```
sudo yum install -y redis
sudo systemctl enable redis.service
sudo systemctl start redis.service
```

## 安装 Nextcloud18
```
wget https://download.nextcloud.com/server/releases/nextcloud-18.0.2.zip #下载 Nextcloud18
unzip nextcloud-18.0.2.zip                     #解压
cp -R nextcloud/ /var/www/html/                #复制 Nextcloud18 到网站目录
mkdir /var/www/html/nextcloud/data          #建立数据文件夹，用于存储用户上传的文件
chown -R apache:apache /var/www/html/nextcloud #确保 apache 对整个 nextcloud 文件夹有读写权限            
```

## 配置 SELinux
```
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/nextcloud/data(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/nextcloud/config(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/nextcloud/apps(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/nextcloud/.htaccess'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/nextcloud/.user.ini'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/nextcloud/3rdparty/aws/aws-sdk-php/src/data/logs(/.*)?'
restorecon -R '/var/www/html/nextcloud/'
setsebool -P httpd_can_network_connect on
```

## 配置 Nextcloud18
在浏览器地址栏输入`x.x.x.x/nextcloud`进入 Nextcloud 配置界面，x.x.x.x 为服务器 IP。以下为配置界面
![][2]

设置 Nextcloud 用户名，密码
点击“存储与数据库” 展开设置项
数据目录默认不用修改（安装 Nextcloud18 时建立过`/var/www/html/nextcloud/data` 文件夹。）
点击 MySQL/MariaDB，按照之前填写的数据库信息，填写数据库设置。
最后点击最下面的“安装完成”按钮，完成 Nextcloud 配置。配置填写如下图
![][3]

等待 Nextcloud 配置完成，出现以下界面，部署完成。
![][4]

[scode type="yellow"]部署成功后，建议上传文件到 Nextcloud 之前首先解决 设置->概览 中的“所使用的数据库为 MySQL 但没有对4字节字符的支持”警告[/scode]
[scode type="green"]Nextcloud18 设置中安全与设置警告解决见以下博文
[Nextclou18 安全与设置警告解决][7]
[/scode]
[scode type="share"]参考：
[Nextcloud17 官方文档][5]
[Nextcloud18 官方文档][6]
[/scode]

  [1]: https://www.mtycho.com/usr/uploads/2020/03/1614978100.jpg
  [2]: https://www.mtycho.com/usr/uploads/2020/03/394713820.jpg
  [3]: https://www.mtycho.com/usr/uploads/2020/03/1040267736.jpg
  [4]: https://www.mtycho.com/usr/uploads/2020/03/1943791440.jpg
  [5]: https://docs.nextcloud.com/server/17/admin_manual/installation/php_72_installation.html
  [6]: https://docs.nextcloud.com/server/18/admin_manual/installation/example_centos.html
  [7]: https://www.mtycho.com/archives/11.html