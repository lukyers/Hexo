---
title: Vultr服务器使用锐速加速器
date: 2016-07-22 14:01:22
categories: 
	- Blog
tags: 
	- HTTPS代理
	- VPN加速
	- 锐速
	- Vultr VPS
---

## Vultr服务器安装锐速加速器##

#### 1. 更换内核####

Vultr自带的Ubuntu14.04.4的内核版本是3.13.0-91-generic，不在锐速支持[内核列表](http://dl.serverspeeder.com/ls.do?m=availables)里，所以我们需要手工更换内核。
首先，搜索可下载的内核

> sudo apt-cache search linux-image
<!-- more -->
我们会发现有好多内核版本可供下载，在刚才的内核列表里选择支持的内核版本即可，这里我选择的是3.13.0-46-generic，找到需要更换的内核就可以安装内核了。

> sudo apt-get install linux-image-3.13.0-46-generic
>
> sudo apt-get install linux-image-extra-3.13.0-46-generic
>

系统会自动安装选择的内核，这时候可以使用`uname -r`来查看当前的内核，但是此时查询到的仍是之前的91版本内核，我们需要把老版本卸载掉才可以生效。

> dpkg -l|grep linux-image

这个命令可以查看当前安装的内核，此时返回的是刚才安装的内核和之前服务器上安装的内核，然后就可以卸载以前安装的内核了。

> sudo apt-get purge linux-image-3.13.0-xx-generic linux-image-extra-3.13.0-xx-generic

这里的xx是上一步中看到的在服务器上安装的其他内核，注意如果当前服务器安装的不是最新的内核，卸载的同时会给服务器安装最新内核；为了能让服务器使用锐速支持的3.13.0-46-generic内核，我们还要再执行一次这个命令，把安装的最新内核卸载掉。

安装完毕后就可以重启服务器了`reboot`，之后可以使用`uname -r`来查看所需版本的内核是否安装成功。

然后需要更新一下引导文件`sudo update-grub ` 

#### 2. 安装锐速破解版

内核安装完毕后，锐速的安装就很简单了，只需要执行以下命令即可（破解版）：

> wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/serverspeeder/master/serverspeeder-all.sh && bash serverspeeder-all.sh

安装完毕后锐速会自动启动，此时可以查看以下外网速度，是不是变快了呢~~

如果想要重启锐速：`service serverSpeeder restart `

