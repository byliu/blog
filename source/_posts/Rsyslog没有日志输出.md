---
title: Rsyslog没有日志输出
date: 2016-03-30 20:43:33
categories:
- Linux
tags:
- Linux
- Rsyslog
---

最近在给私有云存储集群扩容，添加新机器的时候，发现相关日志没有打印出来，而业务功能上是正常的，怀疑是日志打印这块出了问题

机器打印日志用的是rsyslog模块，检查对比了相关配置项后未发现问题，rsyslog进程也正常，未报错，比较奇怪的现象

思索无果下，就只能一步一步看日志来尝试解决了

<!--more-->

查看/var/log/messages，最后一条日志是

	Mar 30 00:19:12 localhost rsyslogd: [origin software="rsyslogd" swVersion="7.4.7" x-pid="903" x-info="http://www.rsyslog.com"] exiting on signal 15.
    
这里初现端倪，在凌晨的时候rsyslogd进程莫名退出了

原因呢，不清楚，Google it，找到了些资料，

参考

[http://askubuntu.com/questions/249490/rsyslogd-stopped-writing-to-var-log-syslog-two-days-ago](http://askubuntu.com/questions/249490/rsyslogd-stopped-writing-to-var-log-syslog-two-days-ago)

[https://bugzilla.redhat.com/show_bug.cgi?id=1116864](https://bugzilla.redhat.com/show_bug.cgi?id=1116864)

看来还是配置项的问题

rsyslog 3.21.1以上支持“配置验证模式”。在这种模式下，它解释和检查配置文件，但不启动。

也就是说我们可以通过执行

```bash
	$/path/to/rsyslogd -f/path/to/config-file -N1
```

来检查我们的配置文件是否有误

配置文件在/etc/rsyslog.d/下，根据资料我们执行

	[root@swift-node1 rsyslog.d]# rsyslogd -f/etc/rsyslog.d/listen.conf -N1
	rsyslogd: version 7.4.7, config validation run (level 1), master config /etc/rsyslog.d/listen.conf
	rsyslogd: invalid or yet-unknown config file command 'SystemLogSocketName' - have you forgotten to load a module? [try http://www.rsyslog.com/e/3003 ]
	rsyslogd: CONFIG ERROR: there are no active actions configured. Inputs will run, but no output whatsoever is created. [try http://www.rsyslog.com/e/2103 ]
	rsyslogd: run failed with error -2103 (see rsyslog.h or try http://www.rsyslog.com/e/2103 to learn what that number means)

嗯，报错了

看到这怀疑可能是/etc/rsyslog.d/listen.conf配置文件的问题了，

删除(记得备份)

	/etc/rsyslog.d/listen.conf

然后重启rsyslog服务，再重启相关业务，发现相关日志已经有了，问题解决

