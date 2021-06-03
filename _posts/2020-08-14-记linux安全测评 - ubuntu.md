---
layout:     post
title:      记linux安全测评
subtitle:   linux安全测评配置记录
date:       2020-08-14
author:    Sunsj
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Linux
---

由于系统需要进行安全测评,需要满足二级安全等级.需配合进行以下模块相关的检测.

- 身份鉴别
- 访问控制
- 安全审计
- 入侵防范
- 恶意代码防范
- 可信验证
- 数据备份恢复
- 剩余信息保护
- 终端杀毒


以下为服务器端此次对应安全整改的操作记录.


**1. 登陆策略设置 - [入侵防范]**

```
# 编辑ssh协议配置
vi /etc/ssh/sshd_config

...

# 最长活动时间 单位s
ClientAliveInterval 600 
# 最大活动数量 
ClientAliveCountMax 2 
# 最多尝试次数 
MaxAuthTries 4
...

# 编辑验证登陆配置
vi /etc/pam.d/login

...
# deny 尝试次数限制3	普通用户解锁数据600s	root解锁时间300s
auth required pam_tally2.so deny=3 unlock_time=600 even_deny_root root_un lock_time=300
...

# 编辑终端停留超时时间
vi /etc/profile

...
export TMOUT=300s
...

```


**2. 密码策略设置 - [入侵防范]**

```
#安装cracklib密码校验工具
apt-get update && apt-get install libpam-cracklib

# 编辑密码校验配置
vim /etc/pam.d/common-password

...

# 最短长度为8 与最近密码不同字符数为3个 同时包含三种类型的字符 
password requisite pam_cracklib.so retry=3 m inlen=8 difok=3 minclass=3 
password [success=1 default=ignore] pam_unix.so obscure use_a uthtok try_first_pass sha512 remember=5 minlen=9
...

# 编辑密码默认配置
vi /etc/login.defs

...

# 密码有效天数	
PASS_MAX_DAYS	180
PASS_MIN_DAYS	0
PASS_WARN_AGE	7
...

```

**3. 审计日志相关配置 - [安全审计]**

```
# 安装审计工具
apt-get update && apt-get install auditd

# 配置审计规则
vi /etc/audit/audit.rules

...
# 文件审计	操作系统用户账户/密码信息
-w /etc/passwd -p rwxa # 目录审计 系统日志信息
-w /var/log # 重要配置
-w /etc/nginx # 项目相关
-w /home/deploy/webroot
...

# 重启
/etc/init.d/auditd restart 

# 查看日志是否生效
sudo ausearch -f 1 /etc/passwd

```

**4. 审计日志备份 - [数据备份恢复]**

采用脚本和crontab进行定时备份,并定时发送至指定备份机多地备份.

**5. 安装杀毒软件 - [安全审计]**


服务器安装厂商提供的杀毒软件,并定时更新病毒库.

**6. 统计所有中间件版本号 - [安全审计]**

根据对应版本号查询对应版本是否有未修复漏洞,若有需及时修复漏洞或者升级版本.

**7. 按照三权分立原则进行权限分离 - [安全审计]**

系统授予不同用户为完成各自承担的任务所需的最小权限，对不同管理用户之间实现权限分离，并设置独立的安全审计员角色，对各类用户的操作行为进行审计监督。

**7. 终端工具版本上报杀毒 - [终端杀毒]**
