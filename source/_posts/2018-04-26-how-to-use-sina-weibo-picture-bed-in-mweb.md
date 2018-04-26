
---
layout: post
title: "MWeb如何使用新浪微博图床"
description: 
category: 技术
tags: [MWeb]
---

- MWeb如何使用新浪微博图床
	- 第0节 背景
	   - 虽然我比较建议使用腾讯云、七牛云之类的，不过新浪微博图床也是蛮不错的，应该有需要之人，我就比较喜欢使用，因为它的https链接都是「免费」的。
<!-- more -->

	- 第 1 节 本地上传服务器
		- 1A命令行
			- 下载源码：git clone git@github.com <mailto:git@github.com>:scue/sinaPicHostingApi.git
			- 启动程序：
				- 安装好yarn和pm2
				- yarn install && yarn start
			- 修改配置：vim config.json修改一下80端口为8015
				- 端口：80→8015
				- 用户：填写自己的新浪微博用户名
				- 密码：填写密码
				- 修改完成后生效方式: yarn restart
		- 1B 验证效果
			- 可以使用curl先验证
				- 命令行：curl -F 'file=@/Users/scue/Downloads/webwxgeticon.jpeg' http://127.0.0.1:8015/upload
				- 返回值：{"status":"success","url":"https://ws1.sinaimg.cn/large/6e22ca27gy1fqqfmochlwj20hs0hsjtv"}
	- 第 2 节 MWeb配置
		- 2A 小节 配置图床
			- ![](https://ws1.sinaimg.cn/large/6e22ca27gy1fqqggr6zc7j20g20hfwgp)

		- 2B 小节 配置Mweb博客
			- ![](https://ws1.sinaimg.cn/large/6e22ca27gy1fqqggzk12zj209e0f9wgh)


这样子的话，只要你粘贴了图片，图片就会自动往微博图床上传了，并自动填写好URL，的确很方便的。

谢谢您的阅读~




