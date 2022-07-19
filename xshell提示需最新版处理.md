## 													xshell提示需最新版处理

```powershell
@echo off
%1 mshta vbscript:CreateObject("Shell.Application").ShellExecute("cmd.exe","/c%~s0::","","runas",1)(window.close)
title Xshell启动器
set atime=%date:~0,4%-%date:~5,2%-%date:~8,2%
 
#设置系统时间
date 2018-12-31
 
#启动xshell设置路径
start ""  "C:\Program Files (x86)\NetSarang\Xshell 7\Xshell.exe"
 
echo 开始同步时间..
ping 0.0.0.0 -n 10> null
echo 同步时间中，完成后自动关闭窗口...
 
date %atime%
exit
 

```



