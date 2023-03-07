[Windows 命令 | Microsoft Docs](https://docs.microsoft.com/zh-cn/windows-server/administration/windows-commands/windows-commands)

搜索文件中文本的模式

```powershell
| findstr
# https://docs.microsoft.com/zh-cn/windows-server/administration/windows-commands/findstr
```

搜索文件中的文本字符串，并显示包含指定字符串的文本行

```powershell
| find
# https://docs.microsoft.com/zh-cn/windows-server/administration/windows-commands/find
```



显示当前进程

```powershell
tasklist

```

杀死某个进程

```powershell
taskkill <exe程序名/PID>

```



历史命令记录

```powershell
# PowerShell获取当前会话中输入的命令
history
get-history
# PowerShell获取当前会话中输入的命令的全部信息
Get-History | Format-List -Property *
# CMD获取当前会话中输入的命令
doskey /h

# 查阅总的命令历史记录：
cat %USERPROFILE%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

# https://docs.microsoft.com/zh-cn/powershell/module/Microsoft.PowerShell.Core/about/about_history?view=powershell-7.2
```



### 生成随机数

```powershell
# cmd窗口直接输入：
set /a %random%%10+1
# bat文件：
set /a rnd=%random%%%10+1
echo %rnd%

# PowerShell
# 使用内置函数：Get-Random
Get-Random
# This command gets a random integer between 0 (zero) and Int32.MaxValue.
Get-Random -Maximum 100
# This command gets a random integer between 0 (zero) and 99.
Get-Random -Minimum 10.7 -Maximum 20.93
# This command gets a random floating-point number greater than or equal to 10.7 and less than 20.92.
Get-Random -InputObject 1, 2, 3, 5, 8, 13 -Count 3 
# This command gets three randomly selected numbers in random order from the array.
```





```powershel
arp -
```

