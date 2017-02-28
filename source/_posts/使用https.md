---
title: "使用https"
date: 2017-02-17 01:24:04
categories: 服务器
tags: 
        [nginx,https,ssl]
---

成文时间: 2017年2月17日 01:25:15

# 使用https
看到访问地址，好像缺点什么。htpps，就是它。

在阿里云上申请了一年的免费SSL证书，需要在服务器特定路径上传一个文件用以验证。
<!--more-->
这里引用了阿里云的安装介绍。

    ( 1 ) 在Nginx的安装目录下创建cert目录，并且将下载的全部文件拷贝到cert目录中。如果申请证书时是自己创建的CSR文件，请将对应的私钥文件放到cert目录下并且命名为cert.key；
    ( 2 ) 打开 Nginx 安装目录下 conf 目录中的 nginx.conf 文件，找到：
     server {
     listen 443;
     server_name localhost;
     ssl on;
     ssl_certificate cert.pem;
     ssl_certificate_key cert.key;
     ssl_session_timeout 5m;
     ssl_protocols SSLv2 SSLv3 TLSv1;
     ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
     ssl_prefer_server_ciphers on;
     location / {
     root html
     }
    }
    ( 3 ) 将其修改为 (以下属性中ssl开头的属性与证书配置有直接关系，其它属性请结合自己的实际情况复制或调整) :
    保存退出。
    ( 4 )重启 Nginx。
    ( 5 ) 通过 https 方式访问您的站点，测试站点证书的安装配置。

也可以不新开一个server， 直接在之前的server上

    server {
         listen 80;
         listen       443 ssl;
         server_name localhost;
         ssl_certificate cert.pem;
         ssl_certificate_key cert.key;
         ssl_session_timeout 5m;
         ssl_protocols SSLv2 SSLv3 TLSv1;
         ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
         ssl_prefer_server_ciphers on;
         location / {
         root html
         }
         #这样可以强制https
         if ($scheme = http ) { 
            return 301 https://$host$request_uri; 
        } 
    }

启动后，访问报错了。查询后发现是用了SSL却没有安装http_ssl_module模块，但http_ssl_module并不属于nginx的基本模块所以自己重新编译添加。

1.首先看下内核和系统的版本号。
[root@zabbix ~]# uname -a
Linux zabbix.nnkj.com 2.6.32-573.el6.x86_64 #1 SMP Thu Jul 23 15:44:03 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
[root@zabbix ~]# cat /etc/issue
CentOS release 6.7 (Final)
Kernel \r on an \m
 
2.看下编译安装nginx的时候，都编译安装的哪些模块。
[root@zabbix ~]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.8.0
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-17) (GCC) 
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx
 
3.进入之前下载并解压了的源码包目录；重新编译nginx
*注意- --prefix=PATH: 指定nginx的安装目录 ，这里要指定为之前的prefix*
[root@zabbix nginx-1.8.0]# cd /usr/local/src/nginx-1.8.0
[root@zabbix nginx-1.8.0]# ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
[root@zabbix nginx-1.8.0]# make
▲这一步千万不能 make install ；不然会把之前已经安装的nginx 覆盖掉
 

4.需要替换nginx二进制文件,先停止掉nginx进程；备份一下原来的启动脚本。
[root@zabbix nginx-1.8.0]# /etc/init.d/nginx stop
[root@zabbix nginx-1.8.0]# cp /etc/init.d/nginx /etc/init.d/nginx.bak
 
[root@zabbix nginx-1.8.0]# cp objs/nginx /usr/local/nginx/sbin/
cp: overwrite `/usr/local/nginx/sbin/nginx'? yes
 
5.查看nginx的模块，看下是否把需要的模块编译进去了

参考:http://www.itnpc.com/news/web/146771636486934.html

重启后，访问成功了，不过我这里提示有不受信任的资源，应该是引用了不少外部链接的原因吧。对了，[我的地址](http://www.gkwind.com)。

