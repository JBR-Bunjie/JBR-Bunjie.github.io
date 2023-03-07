如何在 Windows 中启用详细登录消息？



麻烦up置顶一下，一键添加的cmd命令：
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v VerboseStatus /t REG_DWORD /d 1



有用！不过要提权顺便科普下：
右键单击 System 项，选择“权限”，然后选择“高级”->“添加”->"选择主体"->输入"ev"然后“确定”->勾选“完全控制”然后一路确定即可





组策略里启用:非常详细的信息。就行了

组策略→管理模板→所有设置→输入“显示”
会自动跳转

使用组策略也可以
依次找到“计算机配置-管理模板-系统”，在右侧找到“显示非常详细的状态信息”然后，在弹出的对话窗口中选择“已启用”选项，点击“应用→确定”就OK了





HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System