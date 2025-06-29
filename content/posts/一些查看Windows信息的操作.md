---
title: "一些查看Windows信息的操作"
date: 2023-07-17T14:17:47+08:00
draft: false
categories:
- 应急响应
tags:
- 应急响应
- 蓝队
---

## 查看账号信息

- 查看当前登录账户：queryuser
- 注销用户ID：logoff [ID]
- 查看用户：net user
- 查看用户登录情况：net user [username]
- 打开本地用户组：lusrmgr.msc
- 注册表查看账户，确认系统是否存在隐藏账户：regedit——>`HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\Names`

## 查看端口进程

- 查看当前连接：netstat -ano
- 查看当前成功建立的连接：netstat -ano | findstr "ESTABLISHED"
- 根据pid定位程序：tasklist | findstr "[pid]"
- 根据端口查看pid：netstat -ano | findstr "8080"
- 在“正在运行任务”中获取进程详细信息（进程开始时间、版本、大小等）：运行——msinfo32——软件环境——正在运行任务
- 利用wmic查看进程执行时的命令：
`Wmic process where name='[进程名称]' get name,Caption,executablepath,CommandLine ,processid,ParentProcessId  /value`
`Wmic process where processid='[进程ID]' get name,Caption,executablepath,CommandLine ,processid,ParentProcessId  /value`

## 查看启动项

- 查看系统启动项：运行——msconfig
- 注册表查看是否有异常启动项：
`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run`
`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Runonce`
`HKEY_CURRENT_USER\software\micorsoft\windows\currentversion\run`

## 查看计划任务

- 查看定时任务：终端运行schtasks，win7使用at
![qpesv3.png](https://files.catbox.moe/qpesv3.png)
- 查看任务清单：C:\Windows\System32\Tasks
![9eg1df.png](https://files.catbox.moe/9eg1df.png)
- 删除计划任务（管理员）：SchTasks /Delete /TN  [任务计划名称]

## 查看系统服务

- 查看服务：services.msc
- 删除服务：可手动删除，也可命令行删除。停止服务：sc stop [服务名称]；删除服务：sc delete [服务名称]

## 查看文件

- 查看最近打开的文件："%UserProfile%\Recent"
- 查看C:\Documents and Settings和C:\Users下是否存在可疑用户或文件

## 查看日志

```
Windows安全日志文件在以下路径：
Windows 7及之前的Windows版本：C:\Windows\System32\config\security.log
Windows 8/8.1：C:\Windows\System32\winevt\logs\Security.evtx
Windows 10：C:\Windows\System32\winevt\logs\Security.evtx

Windows Server的安全日志文件与Windows 10类似，位于以下路径：
Windows Server 2008及之前的Windows版本：C:\Windows\System32\config\security.log
Windows Server 2008 R2及更高版本：C:\Windows\System32\winevt\logs\Security.evtx

系统日志：%SystemRoot%\System32\Winevt\Logs\System.evtx

应用程序日志：%SystemRoot%\System32\Winevt\Logs\Application.evtx
```
Windows安全日志事件ID：

| 事件ID | 说明                                                         |
| :----: | :----------------------------------------------------------|
|  4624  | 表示登录成功事件，包括使用正常凭据和匿名登录      |
|  4625  | 表示登录失败事件，包括尝试使用无效凭据或密码    |
|  4634  | 表示注销成功事件，包括用户正常注销和系统强制注销      |
|  4647  | 表示用户启动的注销事件，例如用户通过任务管理器注销或关闭计算机    |
|  4648  | 试图使用显式凭据登录                                         |
| 4657 | 注册表值已修改 |
|  4672  | 使用超级用户（如管理员）进行登录                             |
| 4673 | 特权服务被召唤 |
| 4674 | 尝试对特权对象执行操作 |
| 4697 | 系统中安装了一项服务 |
| 4698 | 已创建计划任务 |
| 4699 | 计划任务已删除 |
| 4700 | 已启用计划任务 |
| 4701 | 计划任务已禁用 |
| 4702 | 计划任务已更新 |
| 4720  | 创建用户，包括新用户创建和现有用户重新启用                     |
| 4740 | 用户账户已被锁定 |
| 4741 | 已创建计算机账户 |
|  4774  | 表示帐户已登录映射事件，例如通过组策略或其他方式将某个帐户映射到登录凭据中 |
|  4775  | 表示无法映射的登录帐户事件，例如未找到与用户名和密码匹配的帐户   |
|  4776  | 表示计算机试图验证的帐户凭据事件，例如尝试使用无效的凭据进行登录   |
|  4777  | 表示域控制器无法验证帐户的凭据事件，例如无法与域控制器进行通信或凭据无效 |
|  4778  | 表示到窗口站重新连接会话事件，例如重新连接到远程桌面会话 |
|  4779  | 表示从窗口站，会话已断开连接事件，例如远程桌面会话意外断开连接 |
|  6005  | 表示计算机日志服务已启动，如果在事件查看器中发现某日的事件ID号为6005，就说明这天正常启动了windows系统。 |
|  6006  | 表示事件日志服务已停止，如果没有在事件查看器中发现某日的事件ID为6006的事件，就表示计算机在这天没关机或没有正常关机。 |

- 通过事件查看器查看日志：eventvwr.msc
- 日志分析工具[logparser](https://www.microsoft.com/en-us/download/details.aspx?id=24659)

**logparser使用：**
- 登录成功的所有事件：`LogParser.exe -i:EVT -o:DATAGRID "SELECT * FROM C:\Windows\System32\winevt\logs\Security.evtx where EventID=4624"`
- 登录失败的所有事件：`LogParser.exe -i:EVT -o:DATAGRID "SELECT * FROM C:\Windows\System32\winevt\logs\Security.evtx where EventID=4625"`
- 指定登录时间范围的事件：`LogParser.exe -i:EVT -o:DATAGRID  "SELECT *  FROM C:\Windows\System32\winevt\logs\Security.evtx where TimeGenerated>'2023-01-19 23:32:11' and TimeGenerated<'2023-06-20 23:34:00' and EventID=4624"`
- 将日志导出为CSV格式并进行分析：`LogParser.exe -i:EVT -o:CSV "SELECT * FROM C:\Windows\System32\winevt\logs\Security.evtx where EventID=4624" > C:\event.csv`
- 提取登录成功的用户名和IP：`LogParser.exe -i:EVT  –o:DATAGRID  "SELECT EXTRACT_TOKEN(Message,13,' ') as EventType,TimeGenerated as LoginTime,EXTRACT_TOKEN(Strings,5,'|') as Username,EXTRACT_TOKEN(Message,38,' ') as Loginip FROM C:\Windows\System32\winevt\logs\Security.evtx where EventID=4624"`
- 提取登录失败用户名进行聚合统计：`LogParser.exe  -i:EVT "SELECT  EXTRACT_TOKEN(Message,13,' ')  as EventType,EXTRACT_TOKEN(Message,19,' ') as user,count(EXTRACT_TOKEN(Message,19,' ')) as Times,EXTRACT_TOKEN(Message,39,' ') as Loginip FROM C:\Windows\System32\winevt\logs\Security.evtx where EventID=4625 GROUP BY Message"`

## webshell查杀工具

- [D盾](https://www.d99net.net/index.asp)
- [河马查杀](https://www.shellpub.com)

## 参考文章
https://bypass007.github.io/Emergency-Response-Notes/Summary/%E7%AC%AC1%E7%AF%87%EF%BC%9AWindow%E5%85%A5%E4%BE%B5%E6%8E%92%E6%9F%A5.html