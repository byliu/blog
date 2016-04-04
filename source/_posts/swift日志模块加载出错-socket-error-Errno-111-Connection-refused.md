---
title: 'swift日志模块加载出错 socket.error: [Errno 111] Connection refused'
date: 2016-04-04 15:53:24
categories:
- OpenStack
- Swift
tags:
- Linux
- Rsyslog 
---

最近在龟速整理笔记中

发现这条笔记感觉还挺有用的，贴出来分享下

## 报错

配置swift完成后，启动swift相关服务时报错

	socket.error: [Errno 111] Connection refused

<!--more-->

详细错误提示

	root@ubuntu:/etc/swift# swift-init account restart -n    
	Signal account-server  pid: 22679  signal: 15
	No account-server running
	Starting account-server...(/etc/swift/account-server/1.conf)
	Traceback (most recent call last):
	  File "/usr/local/bin/swift-account-server", line 23, in <module>
	    sys.exit(run_wsgi(conf_file, 'account-server', **options))
	  File "/usr/local/lib/python2.7/dist-packages/swift/common/wsgi.py", line 455, in run_wsgi
	    _initrp(conf_path, app_section, *args, **kwargs)
	  File "/usr/local/lib/python2.7/dist-packages/swift/common/wsgi.py", line 565, in _initrp
	    log_route='wsgi')
	  File "/usr/local/lib/python2.7/dist-packages/swift/common/utils.py", line 1481, in get_logger
	    raise e
	socket.error: [Errno 111] Connection refused

参考资料

[http://www.rsyslog.com/doc/v8-stable/configuration/modules/imuxsock.html](http://www.rsyslog.com/doc/v8-stable/configuration/modules/imuxsock.html)

## 原因

*	rsyslog 没有对/dev/log监听导致

## 解决办法

	vim /etc/rsyslog.conf

添加

	$AddUnixListenSocket /dev/log

然后再重启相关服务，正常

问题解决




