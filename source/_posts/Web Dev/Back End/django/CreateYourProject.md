### Step 0. figure out the environment of your computer 

- editor: vs code
- OS: windows 11
- Shell: powershell

### Step1. create the virtual environment

1. we create a new folder named "django_test"
2. we run this command

```python
python -m venv ./venv
```

so that we will see a folder in "django_test" named "venv" has been created, what happened?

- what is `python -m` ?



- what is venv?



we can have a pure new environment of python which help us a lot.

for example, we can use `pip freeze > requirement.txt`



3. active the virtual environment

after creating the venv folder, you can see there folders inside, and we enter the "Script" folder and exec activate.ps1 in powershell, then you should see the new command line like:

```powershell
(venv) PS C:\Project\django_test>
```

it means we have activated the virtual environment successfully.

Now, you can use where command and you will see that the python.exe of venv folder was placed at the first place.

```python
(venv) PS C:\Project\django_test> where.exe python
C:\Project\django_test\venv\Scripts\python.exe
C:\Users\m1518\AppData\Local\Programs\Python\Python310\python.exe
C:\Users\m1518\AppData\Local\Microsoft\WindowsApps\python.exe
```



特别的，当你在powershell下使用脚本时，可能会遇到如下问题：

```powershell
<YouProjectPath>\venv\Scripts> .\activate.ps1
.\activate.ps1 : 无法加载文件 <YouProjectPath>\venv\Scripts\Activate.ps1，
因为在此系统上禁止运行脚本。有关
详细信息，请参阅 https:/go.microsoft.com/fwlink/?LinkID=135170 
中的 about_Execution_Policies。
所在位置 行:1 字符: 1
+ .\activate.ps1
+ ~~~~~~~~~~~~~~
    + CategoryInfo          : SecurityError: (:) []，PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

这是因为在默认情况下，PowerShell对脚本的管理策略时是 Restricted。

```
Restricted 执行策略不允许任何脚本运行。AllSigned 和 RemoteSigned 执行策略可防止 Windows PowerShell 运行没有数字签名的脚本。本主题说明如何运行所选未签名脚本（即使在执行策略为 RemoteSigned 的情况下），还说明如何对 脚本进行签名以便您自己使用。有关 Windows PowerShell 执行策略的详细信息，请参阅 about_Execution_Policy。
```

想了解 计算机上的现用执行策略，在` PowerShell` 然后输入 `get-executionpolicy`

默认情况下返回的是 `Restricted`

以管理员身份打开`PowerShell` 输入 `**set-executionpolicy remotesigned**`

就可以正常在 `PowerShell` 中运行 `ps1` 文件了

4. 退出

退出虚拟环境很简单，只需要执行 `deactivate` 命令就行，这个命令也在虚拟环境的脚本目录下，因为激活时，将脚本目录设置到 PATH 中了，所以可以直接使用

当然也可以直接输入`powershell`或`bash`等来重启你的terminal

### Step 2. install Django

just use these commands below:

```powershell
pip install django

pip install django-fliter

# pip install markdown

cd ..
django-admin startproject <YourProjectName>
cd <YourProjectName>
```

You can see your project have been created well here.

You can start the server directly by using `python manage.py runserver`, you will see something.





### Links

- [windows - Equivalent of cmd's "where" in powershell - Super User](https://superuser.com/questions/675837/equivalent-of-cmds-where-in-powershell)
- https://www.youtube.com/watch?v=F5mRW0jo-U4&t=2866s
- [venv — Creation of virtual environments — Python 3.10.7 documentation](https://docs.python.org/3/library/venv.html)



























### Create a Superuser

```powershell
(venv) PS C:\Project\django_test\helloworld> python manage.py createsuperuser
Username (leave blank to use 'm1518'): 
Email address: jbr_bunjie@outlook.com
Password: 
Password (again): 
Superuser created successfully.
```



