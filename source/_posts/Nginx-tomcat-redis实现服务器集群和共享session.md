---
title: Nginx+tomcat+redis实现服务器集群和共享session
date: 2017-06-27 15:37:22
tags: 学习，Nginx，redis
---
## 有关Nginx
*  Nginx是一个代理服务器，主要功能是实现负载均衡和虚拟主机。
*  负载均衡就是使用Nginx作为代理服务器，用户请求首先是达到Nginx服务器，Nginx再根据实际情况将请求分流，分发到不同后端，这里就是后端实现了服务器集群，
*  下图就是负载均衡原理
![负载均衡原理](http://img.blog.csdn.net/20160612171735599?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
* 有关Nginx的详细介绍可以参考该博客[nginx详细介绍及配置](http://blog.csdn.net/hiyun9/article/details/51602428)
## 有关redis
* redis是一个数据库，一个键值型数据库
* redis将数据存储到内存，持久化到磁盘，所以读写速度很快
* redis可以做关系型数据库的缓存，存储一些数据量比较小的数据
## Nginx负载均衡中的session问题
* 负载均衡会将每次请求分发到不同的服务器
* session保存在服务器端，sessionid在本地，这就会导致在这个服务器生成的session下次请求另一个服务器时导致无法识别的问题。
* 这就需要服务器集群之间共享session
* 最简单的问题就是登陆验证的时候，验证码保存到session中，验证码的生成是一个服务器，最后登陆验证的时候是另一个服务器，这就会导致验证失败。
#### 几种解决session共享的方法
1. 配置Nginx中neinx.conf的server选项，如下加入ip_hash,这样是对于同一个ip只会分发到一个服务器，这样其实失去了负载均衡的意思，而且Nginx获取的必须是用户真实的ip地址，也就是必须是最前端的服务器，配置如下
```roboconf

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
upstream netitcast.com{
   server  127.0.0.1:18080 weight=1;
   server  127.0.0.1:28080 weight=2;
   ip_hash; #这里配置
}
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        #location / {
        #    root   html;
        #    index  index.html index.htm;
        #}
		location / {
			proxy_pass  http://netitcast.com;
			proxy_redirect  default;
		}

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```
2. 第二个就是使用redis存储session了
>  这里使用的是tomcat-redis-session-manager框架，该框架是基于tomcat服务器的，只需要在tomcat中加入tomcat-redis-session-1.0-SNAPSHOT.jar、jedis-2.7.2.jar、commons-pool2-2.0.jar三个包
>  三个包的下载地址[点击下载](http://download.csdn.net/detail/u010870518/9585716)网上很多包不能用，这里是可以用的，不过是对于tomcat7
* 修改Tomcat的conf/context.xml，加入redis的相关连接信息，配置如下
```xml
 <Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />  
  <Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"  
   host="localhost"  
   port="6379"  
   database="0"  
   maxInactiveInterval="60" />  
```
* 这样就可以在新建session的时候将session存储到redis然后读取的时候从redis获取，redis是服务器集群的共享，
下图是原理
![redis共享session的原理](http://img.blog.csdn.net/20160725171038125)
* 在默认的情况下Tomcat的Session管理，如果不进行设置的话是由Tomcat自带的tandardManager类进行控制的，我们可以根据这个类自定义一个Manager，主要重写的就是org.apache.catalina.session.ManagerBase里边的具体写的操作， 
这也是tomcat-redis-session-manager的基本原理，将tomcat的session存储位置指向了Redis
* RedisSessionManager继承了org.apache.catalina.session.ManagerBase并重写了add、findSession、createEmptySession、remove等方法，并将对session的增删改查操作指向了对Redis数据存储的操作

