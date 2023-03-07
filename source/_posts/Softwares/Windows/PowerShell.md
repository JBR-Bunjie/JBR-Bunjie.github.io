介绍——什么是PowerShell？

[什么是 PowerShell？ - PowerShell | Microsoft Docs](https://docs.microsoft.com/zh-cn/powershell/scripting/overview?view=powershell-7.1)

 

1. PowerShell和CMD的关系：

2. 1. 在Windows下，简单的说，PowerShell可以近似认为是cmd的超集，换句话说，PowerShell几乎兼容所有cmd命令。cmd能做的事情，PowerShell都能做，但是PowerShell还能额外做许多cmd不能做的活。

（来源：PowerShell 命令称为 cmdlet（读作 command-let）。 除了 cmdlet 外，使用 PowerShell 还可以在系统上运行任何可用命令。）

1. 但是PowerShell对于某些CMD中某些命令有更加严格的限制，比如你用npm全局安装的包，默认不能在Powershell上面直接运行
2. ***！！！！存在部分CMD命令无法直接在PowerShell上运行！！！！***

 

1. Powershell特点：

2. 1. PowerShell      是新式命令 shell，其中包括其他常用 shell 的最佳功能。 与大多数仅接受并返回文本的 shell 不同，PowerShell 接受并返回 .NET 对象
   2. 是一种脚本语言shell，可以用于自动执行系统管理。如和office套件联动
   3. PowerShell跨平台