---
layout: post
title: 同步Lotus Notes Calendar至Google Calendar
description: 记录如何同步Lotus Notes Calendar至Google Calendar
keyword: notes, calendar, gtd 
categories: programming
---

## 准备工作

1. 下载[http://sourceforge.net/projects/lngooglecalsync](http://sourceforge.net/projects/lngooglecalsync/)，解压。
2. 打开`HelpFile.html`，仔细阅读`Installation`部分。
3. 如果至`#13`步时出现如下错误：

	```
	google no application name
	Error: invalid_client

	Request Details
	cookie_policy_enforce=false
	scope=https://www.googleapis.com/auth/calendar
	response_type=code
	access_type=offline
	redirect_uri=http://localhost:35406/Callback
	display=page
	client_id=1060145331129.apps.googleusercontent.com
	```

可以参考[stackoverflow](http://stackoverflow.com/questions/18677244/error-invalid-client-no-application-name)来解决。

## 运行

1. Run `lngsync.sh`
2. 第一次可以用图形化的方式来配置各个选项，尤其是`Detect Lotus Settings`功能很赞
![](/images/201404/lngc.png)
3. 之后就完全可以在你的某台台式机定期地跑`lngsync.sh -silent`就自动同步了
