---
title: OpenResty入门
categories: OpenResty
tags: OpenResty
date: 2019-08-13 08:27:52
---

OpenResty(又称：ngx_openresty) 是一个基于 nginx的可伸缩的 Web 平台，由中国人章亦春发起，提供了很多高质量的第三方模块。

OpenResty 是一个强大的 Web 应用服务器，Web 开发人员可以使用 Lua 脚本语言调动 Nginx 支持的各种 C 以及 Lua 模块,更主要的是在性能方面，OpenResty可以 快速构造出足以胜任 10K 以上并发连接响应的超高性能 Web 应用系统。

360，UPYUN，阿里云，新浪，腾讯网，去哪儿网，酷狗音乐等都是 OpenResty 的深度用户。

OpenResty 简单理解成 就相当于封装了nginx,并且集成了LUA脚本，开发人员只需要简单的其提供了模块就可以实现相关的逻辑，而不再像之前，还需要在nginx中自己编写lua的脚本，再进行调用了。

# 1 安装openresty

linux安装openresty:

1.添加仓库执行命令

```cpp
yum install yum-utils
 yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo
 ```

 2.执行安装

     yum install openresty

3.安装成功后 会在默认的目录如下：
 
     /usr/local/openresty

# 2 安装nginx

默认已经安装好了nginx,在目录：/usr/local/openresty/nginx 下。

修改/usr/local/openresty/nginx/conf/nginx.conf,将配置文件使用的根设置为root,目的就是将来要使用lua脚本的时候 ，直接可以加载在root下的lua脚本。

     cd /usr/local/openresty/nginx/conf
     vi nginx.conf  

 修改代码如下：

 ![2019-08-13-08-34-42.png](http://pv125k1gl.bkt.clouddn.com/2019-08-13-08-34-42.png)

 # 3 测试访问

 重启下centos虚拟机，然后访问测试Nginx

访问地址：http://192.168.211.132/

![2019-08-13-08-35-52.png](http://pv125k1gl.bkt.clouddn.com/2019-08-13-08-35-52.png)

     