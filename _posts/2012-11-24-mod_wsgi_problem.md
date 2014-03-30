---
title: httpd-2.4上mod_wsgi编译出错
layout: post
---

### 问题

源码编译安装了httpd-2.4后，编译mod_wsgi-3.3：

    $ ./configure --with-apxs=PATH_TO_APXS --with-python=/usr/bin/python</code>
    $ make
        bq. mod_wsgi.c: In function 'wsgi_process_socket':
        mod_wsgi.c:10095:37: error: 'conn_rec' has no member named 'remote_addr'
        mod_wsgi.c:10103:27: error: 'conn_rec' has no member named 'remote_ip'
        mod_wsgi.c:10103:41: error: 'conn_rec' has no member named 'remote_addr'
        mod_wsgi.c: In function 'wsgi_hook_daemon_handler':
        mod_wsgi.c:12742:18: error: 'conn_rec' has no member named 'remote_ip'
        mod_wsgi.c: In function 'Auth_environ':
        mod_wsgi.c:13262:10: error: 'conn_rec' has no member named 'remote_ip'
        mod_wsgi.c:13263:18: error: 'conn_rec' has no member named 'remote_ip'
        mod_wsgi.c:13295:14: error: 'conn_rec' has no member named 'remote_addr'
        mod_wsgi.c: In function 'wsgi_hook_access_checker':
        mod_wsgi.c:14395:29: error: 'conn_rec' has no member named 'remote_ip'
        mod_wsgi.c: At top level:
        mod_wsgi.c:14697:5: warning: initialization from incompatible pointer type [enabled by default]
        mod_wsgi.c:14697:5: warning: (near initialization for 'wsgi_authz_provider.check_authorization') [enabled by default]
        apxs:Error: Command failed with rc=65536

* * *

### 解决方法

将mod_wsgi.c中所有remote_ip改成client_ip，remote_addr改成client_addr

google发现[这里博文](http://www.gzayong.info/ws/archives/223) 提到问题解决方法。

是httpd-2.4版本的新API做了修改： 详见[这里](http://httpd.apache.org/docs/2.4/developer/new_api_2_4.html)

* * *

### 启示

* 源码安装一个包可能会出现各种各样的问题，在./configure时候最好先./configure -h看一下需要注意的选项。尽量统一安装目录，不使用默认安装路径。
* 编译出问题，一定要去查看官方文档，发现问题。
