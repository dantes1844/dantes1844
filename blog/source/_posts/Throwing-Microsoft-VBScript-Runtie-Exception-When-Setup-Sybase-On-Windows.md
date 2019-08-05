---
title: windows10 安装sybase报Microsoft VBScript 运行时错误 (0x800A0046) 没有权限的解决方案
date: 2019-07-30 17:19:44
tags: [Sybase,Windows10]
---
执行脚本文件`CreateSybaseMenus.vbs`一直报题中的错误。
文件的内容如下代码：
``` vbscript
Dim objShell
dim strAllUsersPrograms
dim objFolder
dim objShellLink

Dim OsType
Dim WshShell 
Dim sProgramFiles
Dim sWindows

Set WshShell = WScript.CreateObject("WScript.Shell") 

OsType = WshShell.RegRead("HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Environment\PROCESSOR_ARCHITECTURE")
'SQLAny12 Environment variable is not consistently available so I am making some assumptions.


If OsType = "x86" then
	'wscript.echo "Windows 32bit system detected"
	sProgramFiles = "C:\Program Files"
	sWindows = "%windir%\system32"
elseif OsType = "AMD64" then
	'wscript.echo "Windows 64bit system detected"
	sProgramFiles = "C:\Program Files (x86)"
	sWindows = "%windir%\SysWoW64"
end if

'WshShell.Run sWindows & "\msiexec /qb /i ""SQL Anywhere 12 Deployment.msi"

Set objShell = createobject("Wscript.Shell")
Set objFso = CreateObject("Scripting.FileSystemObject")

strAllUsersPrograms= objShell.SpecialFolders("AllUsersPrograms")

if objFSO.FolderExists( strAllUsersPrograms & "\SQL Anywhere 12")=false then
    '具体是以下这行报错'
	Set objFolder = objFSO.CreateFolder( strAllUsersPrograms & "\SQL Anywhere 12")
end if

if objFSO.FolderExists( strAllUsersPrograms & "\SQL Anywhere 12\Administration Tools")=false then
	Set objFolder = objFSO.CreateFolder( strAllUsersPrograms & "\SQL Anywhere 12\Administration Tools")
end if

if objFSO.FolderExists( strAllUsersPrograms & "\SQL Anywhere 12\SQL Anywhere")=false then
	Set objFolder = objFSO.CreateFolder( strAllUsersPrograms & "\SQL Anywhere 12\SQL Anywhere")
end if


If (objFso.FileExists(strAllUsersPrograms & "\SQL Anywhere 12\Administration Tools\Interactive SQL.lnk")) = false Then 
	Set objShellLink = objShell.CreateShortcut(strAllUsersPrograms & "\SQL Anywhere 12\Administration Tools\Interactive SQL.lnk")
	objShellLink.TargetPath = sProgramFiles & "\SQL Anywhere 12\Bin32\dbisql.exe"
	objShellLink.Description = "Interactive SQL"
	objShellLink.Save
	set objShellLink = nothing
end if

If (objFso.FileExists(strAllUsersPrograms & "\SQL Anywhere 12\Administration Tools\ODBC Data Source Administrator (32-bit).lnk")) = false Then 
	Set objShellLink = objShell.CreateShortcut(strAllUsersPrograms & "\SQL Anywhere 12\Administration Tools\ODBC Data Source Administrator (32-bit).lnk")
	objShellLink.TargetPath = sWindows & "\odbcad32.exe"
	objShellLink.Description = "ODBC Data Source Administrator (32-bit)"
	objShellLink.Save
	set objShellLink = nothing
end if

If (objFso.FileExists(strAllUsersPrograms & "\SQL Anywhere 12\Administration Tools\Sybase Central.lnk")) = false Then 
	Set objShellLink = objShell.CreateShortcut(strAllUsersPrograms & "\SQL Anywhere 12\Administration Tools\Sybase Central.lnk")
	objShellLink.TargetPath = sProgramFiles & "\SQL Anywhere 12\Bin32\scjview.exe"
	objShellLink.Description = "Sybase Central"
	objShellLink.Save
	Set objSystemFolder = nothing
end if

If (objFso.FileExists(strAllUsersPrograms & "\SQL Anywhere 12\SQL Anywhere\Network Server.lnk")) = false Then 
	Set objShellLink = objShell.CreateShortcut(strAllUsersPrograms & "\SQL Anywhere 12\SQL Anywhere\Network Server.lnk")
	objShellLink.TargetPath = sProgramFiles & "\SQL Anywhere 12\Bin32\dbsrv12.exe"
	objShellLink.Description = "Network Server"
	objShellLink.Save
	Set objSystemFolder = nothing
end if

If (objFso.FileExists(strAllUsersPrograms & "\SQL Anywhere 12\SQL Anywhere\Personal Server.lnk")) = false Then 
	Set objShellLink = objShell.CreateShortcut(strAllUsersPrograms & "\SQL Anywhere 12\SQL Anywhere\Personal Server.lnk")
	objShellLink.TargetPath = sProgramFiles & "\SQL Anywhere 12\Bin32\dbeng12.exe"
	objShellLink.Description = "Personal Server"
	objShellLink.Save
	Set objSystemFolder = nothing
end if

Set objFolder = nothing
Set objFSO = nothing
set objShell = nothing
                               
Call MsgBox("Installation Complete!",vbSystemModal, "SQL Anywhere 12")                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
```
具体是35行报错。可以看出是新建文件夹时的问题，找到其路径`C:\ProgramData\Microsoft\Windows\Start Menu\Programs `,给文件夹安全选项里添加相应的权限用户，再次执行即可。