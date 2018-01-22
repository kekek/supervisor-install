
#### file list
.
├── README.md
├── supervisor-event-listener.ini 	// 运行时配置
├── supervisord-listener		// 实际运行程序
└── supervisord-listener.ini     	// supervisord listen 配置 

# supervisor-event-listener
Supervisor事件通知, 支持邮件, Slack, WebHook

## 简介
Supervisor是*nix环境下的进程管理工具, 可以把前台进程转换为守护进程, 当进程异常退出时自动重启.  
supervisor-event-listener监听进程异常退出事件, 并发送通知.
  

## Supervisor配置
```ini
[eventlistener:supervisor-event-listener]
; 默认读取配置文件/etc/supervisor-event-listener.ini
command=/path/to/supervisor-event-listener
; 指定配置文件路径
;command=/path/to/supervisor-event-listener -c /path/to/supervisor-event-listener.ini
events=PROCESS_STATE
......
```

## 配置文件, 默认读取`/etc/supervisor-event-listener.ini`

```ini 
[default]

debug = true

# 通知类型 mail,slack,webhook,shell 多种，逗号分隔
notify_type = mail

# 邮件服务器配置
mail.server.user = test@163.com
mail.server.password = 123456
mail.server.host = smtp.163.com
mail.server.port = 25

# 邮件收件人配置, 多个收件人, 逗号分隔
mail.user = hello@163.com

# Slack配置
slack.webhook_url = https://hooks.slack.com/services/xxxx/xxx/xxxx
slack.channel = exception

# WebHook通知URL配置 
webhook_url = http://my.webhook.com

# shell 脚本，会传递 eventname  playload ,  例/bin/echo eventname ip ProcessName GroupName FromState Expected Pid. 见 restart.sh
shell.command = "/bin/echo"

# 监听事件，逗号分隔
events = PROCESS_STATE_EXITED,PROCESS_STATE_RUNNING

```

## 通知内容
邮件、Slack
```shell
Host: ip(hostname)
Process: process-name
PID: 6152
EXITED FROM state: RUNNING
```
WebHook, Post请求, 字段含义查看Supervisor文档
```json
{
  "Header": {
    "Ver": "3.0",
    "Server": "supervisor",
    "Serial": 11,
    "Pool": "supervisor-listener",
    "PoolSerial": 11,
    "EventName": "PROCESS_STATE_EXITED",
    "Len": 84
  },
  "Payload": {
    "Ip": "ip(hostname)",
    "ProcessName": "process-name",
    "GroupName": "group-name",
    "FromState": "RUNNING",
    "Expected": 0,
    "Pid": 6371
  }
}
```


## shell 例子
```
#!/bin/bash
case $1 in
'PROCESS_STATE_RUNNING')
	/bin/echo "$3 run" >> /root/event.log
	if [ $3 == 'micro_teacher_server' ];then
		supervisorctl restart micro_teacher_api
	fi
;;

'PROCESS_STATE_STOPPED')
# supervisorctl stop 
	/bin/echo "$3 stoped" >> /root/event.log
;;

'PROCESS_STATE_EXITED')
	/bin/echo "$3 exited" >> /root/event.log

;;

'PROCESS_STATE_FATAL')
	/bin/echo "$3 fatal" >> /root/event.log
;;

esac
```
