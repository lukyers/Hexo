---
title: 使用Squid搭建HTTPS代理服务器
date: 2016-06-23 16:03:36
categories: 
	- Blog
tags: 
	- HTTPS代理
	- 翻墙
	- VPN
---
# 使用Squid搭建HTTPS代理服务器
------

![git命令.png](http://ww2.sinaimg.cn/large/6b118931jw1f1txai3gspj20xb0xcter.jpg)

　　由于工作需要以及某些原因，经常需要上一些"其他"网站，所以上网利器是一个必不可少的工具，但苦于公司的IT不允许安装shadowsocks，第三方的VPN服务有时候会抽风，所以就在自己的AWS上搭建了代理，之前有用过TinyProxy的HTTP代理，由于没有加密传输所以还是会被墙，所以我们需要使用加密的HTTPS来穿透GFW，在网上找到了Squid来搭建的HTTPS代理，记录一下安装的过程。
本文转载自[使用Squid搭建HTTPS代理服务器](http://www.predatorray.me/%E5%9C%A8VPS%E4%B8%8A%E6%90%AD%E5%BB%BASquid%E4%BB%A3%E7%90%86%E6%9C%8D%E5%8A%A1%E5%99%A8/)
### **安装必要的软件**
------
安装`apache2-utils`用于HTTP认证文件的生成，
> apt-get install apache2-utils -y

安装`Squid`，

> apt-get install squid3 -y

安装`stunnel`，

> apt-get install stunnel4 -y

### **配置Squid**
------
生成HTTP认证文件，输入对应的密码。这个认证文件用于之后HTTP代理的认证登录，如果不需要登录认证，可以略过。
> htpasswd -c /etc/squid3/squid.passwd <登录用户名>

修改Squid默认配置，配置文件位于`/etc/squid3/squid.conf`。
#### **1. 修改监听地址与端口号**

找到`TAG: http_port`注释，把其下方的

> Squid normally listens to port 3128
http_port 3128

中`http_port`修改为`127.0.0.1:3128`，使得Squid只能被本地（127.0.0.1）访问。此处可以修改为监听其他端口号。

#### **2. 修改访问权限与HTTP认证（可选）**

若不需要添加HTTP认证，只需将`http_access deny all`修改为`http_access allow all`即可，无需下列的操作。

使用如下命令生成认证文件，

> htpasswd -c /etc/squid3/squid.passwd <登录用户名>

再次打开Squid配置文件`/etc/squid3/squid.conf`，找到`TAG: auth_param`注释，在其下方添加，

> auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid3/squid.passwd
auth_param basic children 5
auth_param basic realm Squid proxy-caching web server
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive off

找到`TAG: acl`，在其下方添加，

> acl ncsa_users proxy_auth REQUIRED

找到`TAG: http_access`，在其下方添加，使得只允许经过认证的用户访问，

> http_access deny !ncsa_users
http_access allow ncsa_users

#### **3. 重启Squid**

> service squid3 restart

### **配置stunnel**
------

接下来，我们需要在Squid上添加一层加密。

#### **1. 生成公钥和私钥**

生成私钥（`privatekey.pem`）：

> openssl genrsa -out privatekey.pem 2048

生成公钥（`publickey.pem`）：

> openssl req -new -x509 -key privatekey.pem -out publickey.pem -days 1095

（需要注意的是，`Common Name`需要与服务器的IP或者主机名一致）

合并：

> cat privatekey.pem publickey.pem >> /etc/stunnel/stunnel.pem

#### **2. 修改stunnel配置**

新建一个配置文件`/etc/stunnel/stunnel.conf`，输入如下内容

> client = no
[squid]
accept = 4128
connect = 127.0.0.1:3128
cert = /etc/stunnel/stunnel.pem

配置中指定了stunnel所暴露的HTTPS代理端口为4128，可以修改为其他的值。

修改`/etc/default/stunnel4`配置文件中`ENABLED`值为1。

> ENABLED=1

#### **3. 重启stunnel**

> service stunnel4 restart

至此，服务器端已配置完成了。

### **本地浏览器配置**
------
#### **添加证书到受信任的根证书颁发机构列表中**


以Windows下Chrome浏览器为例，将服务器上的公钥`publickey.pem`下载至本地，重命名至`publickey.crt`，在Chrome中依次点击 “设置” - “显示高级设置” - “HTTP/SSL” - “管理证书”，在“受信任的根证书颁发机构”选项卡中“导入”这个crt证书就完成了。

#### **代理客户端配置**

将本地的代理客户端指向`https://<你的服务器IP或主机名>:4128`，这里的IP或主机名和生成公钥时的`Common Name`一致，端口为stunnel的端口。如果有配置HTTP认证的话，需要在客户端中配置对应的用户名和密码。如果没有HTTP客户端的话，推荐使用Chrome的插件`Proxy SwitchyOmega`（使用教程可以参考Github上的Wiki）。

