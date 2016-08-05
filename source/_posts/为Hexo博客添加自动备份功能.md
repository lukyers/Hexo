---
title: 为Hexo博客添加自动备份功能
date: 2016-08-02 11:09:06
tags:
	- Hexo博客自动备份
categories:
	- Blog
---

## 为Hexo博客添加自动备份功能

Hexo博客作为一个简单方便的个人博客框架很适合开发者使用，但是如果换了电脑想更新博客文章就需要费一些事了。之前自己的解决方案是手动把文件添加的github上，但是每次都需要hexo deploy后手动输入git push，有时候忘记了就会有问题，最近看到网上有用js写的脚本来实现每次调用deploy后自动进行上传备份，经测试后发现这个功能确实有效。

<!-- more -->

-------------------------

#### 1 安装shelljs

实现自动备份功能需要安装一个NodeJs的一个`shelljs`模块，该模块可以使调用系统命令更加方便。

在Hexo根目录下键入以下命令来安装`shelljs`

```javascript
npm install --save shelljs
```

#### 2 编写自动备份脚本

在Hexo根目录下新建`script`目录，然后新建后缀名为`.js`的文件，名字可以随意起，比如`backup.js`

脚本内容如下：

```javascript
require('shelljs/global');

try {
	hexo.on('deployAfter', function() {//当deploy完成后执行备份
		run();
	});
} catch (e) {
	console.log("产生了一个错误<(￣3￣)> !，错误详情为：" + e.toString());
}

function run() {
	if (!which('git')) {
		echo('Sorry, this script requires git');
		exit(1);
	} else {
		echo("======================Auto Backup Begin===========================");
		cd('D:/hexo');    //此处修改为Hexo根目录路径
		if (exec('git add --all').code !== 0) {
			echo('Error: Git add failed');
			exit(1);

		}
		if (exec('git commit -am "Form auto backup script\'s commit"').code !== 0) {
			echo('Error: Git commit failed');
			exit(1);

		}
      	//如果Git远程仓库名称不是origin的话,修改为自己的远程仓库名字即可
		if (exec('git push origin master').code !== 0) {
			echo('Error: Git push failed');
			exit(1);

		}
		echo("==================Auto Backup Complete============================")
	}
}
```

保存脚本并退出，然后就可以新建文章了，最后执行`hexo deploy`命令后，脚本就会在deploy完成后自动调用push指令了: )。



![微信截图_20160805181936.png](https://ooo.0o0.ooo/2016/08/05/57a46a45394b7.png)

__这里的*号不能少，缺少的话后面的通配符不起作用。__

