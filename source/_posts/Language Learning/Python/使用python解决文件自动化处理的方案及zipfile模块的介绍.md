## 使用Python解决文件自动化改名和命名

### 原因：获取压缩文件的实际名称：

压缩文件名称使用了随机字符，而原文件夹作为压缩文件的根目录来保存，为了避免打开压缩文件才能知道打开的到底是哪个文件，需要将压缩文件内的文件夹名复制出来为外部压缩文件改名

### 采用方案：

#### 解压缩模块：zipfile

official site [zipfile — Work with ZIP archives — Python 3.9.6 documentation](https://docs.python.org/3/library/zipfile.html)

1. 提取所有压缩文件的实际名称

#### 文件处理模块：os

1. 确定实际工作目录
2. 找到所有需要改名的文件并存储
3. 在压缩文件中提取完实际名称后对源文件进行改名

#### 文字提取模块（正则表达式）：re

1. 对路径使用正则表达式来处理文件

程序实例：

```python
import os
import zipfile
import re

os.getcwd()			# 等待结果
path = r""			# 输入要处理文件的路径
os.chdir(path)		# 进入目标目录

A = []
for i in os.walk(path):
    A.append(i)
    
B = A[0][2] # B按顺序存储了需要修改的所有文件，注意A[0][2]的真正含义！

# 注意，B可能包含已经修改了的文件，这时有两个方案
	# 1. （通过操作）规避这些文件，单独处理（费时费力，但是可以在一个文件夹下完成所有操作，也符合我个人直觉，毕竟是存在某些文件没有
	# 2. 将这些文件全部复制到新文件夹下保持文件纯净
# 采用方案2

C = [] # 存储真实名称

for i in B:
    with zipfile.ZipFile(i, 'r') as z:
        for i in z.namelist():
            C.append(i)
        z.close() # 可能会有几百行？_(:з」∠)_，总之，拿下了就行

get_name_rule = re.compile(r" ", re.S) # 正则表达式的筛选规则
D = []
for i in C:
	if re.findall(get_name_rule, i) != []:
        D.append(re.findall(get_name_rule, i)[0])

'''
E = []
rule_extended = re.compile(r" ", re.S)
for i in D:
    if re.findall(rule_extended, i) != []:
    	E.append(re.findall(rule_extended, i)[0])
    else:
    	E.append(i)
补充提取，将所有违规字符如'/'，'*'等全部剔除
'''

count = 0
for i in B:
    temp = E[count] + ".zip" 	# 注意后缀！
    os.rename(i, temp)
    count = count + 1

    
    
    
    
以上全部！
```





## 有关`zipfile`的常用方法：

1. 一看就懂：

   ```python
   import zipfile
   f = zipfile.ZipFile('filename.zip', 'w' ,zipfile.ZIP_DEFLATED)
   f.write('file1.txt')
   f.write('file2.doc')
   f.write('file3.rar')
   f.close()
   f = zipfile.ZipFile('filename')
   f.extractall()
   f.close()
   ```

   1.1 `zipfile.`**ZipFile**(*file*, *mode='r'*, *compression=ZIP_STORED*, *allowZip64=True*, *compresslevel=None*, ***, *strict_timestamps=True*)

   Open a ZIP file, where *file* can be a path to a file (a string), a file-like object or a [path-like object](https://docs.python.org/3/glossary.html#term-path-like-object).

   > fileName是没有什么疑问的了。
   >
   > mode和一般的文件操作一样,'r'表示打开一个存在的只读ZIP文件；'w'表示清空并打开一个只写的ZIP文件，或创建一个只写的ZIP文件；'a'表示打开一个ZIP文件，并添加内容。

> > The *mode* parameter should be `'r'` to read an existing file, 
> >
> > `'w'` to truncate and write a new file, 
> >
> > `'x'` to exclusively create and write a new file. 
> >
> > > If *mode* is `'x'` and *file* refers to an existing file, a [`FileExistsError`](https://docs.python.org/3/library/exceptions.html#FileExistsError) will be raised. 
> >
> > `'a'` to append to an existing file, 
> >
> > > If *mode* is `'a'` and *file* refers to an existing ZIP file, then additional files are added to it. 
> > >
> > > If *file* does not refer to a ZIP file, then a new ZIP archive is appended to the file. This is meant for adding a ZIP archive to another file (such as `python.exe`).
> > >
> > > ```python
> > > #比如：我有一个new.txt文件
> > > a = zipfile.ZipFile('new.txt', 'a') # 此时new.txt文件会被归档，new.txt会被改写为zip文件
> > > a.write('new.txt')
> > > a.close()
> > > # 并在外侧改写new.txt的后缀为.zip，发现存在是一个正式的.zip文件且存在一个new.txt
> > > ```
> > >
> > > If *mode* is `'a'` and the file does not exist at all, it is created. 
> >
> > If *mode* is `'r'` or `'a'`, the file should be seekable.

> compression表示压缩格式，可选的压缩格式只有2个：ZIP_STORE;ZIP_DEFLATED。ZIP_STORE是默认的，表示不压缩；ZIP_DEFLATED表示压缩。
>
> allowZip64为True时，表示支持64位的压缩，一般而言，在所压缩的文件大于2G时，会用到这个选项；默认情况下，该值为False，因为Unix系统不支持。

1.2 `zipfile`.**close**()

> 你写入的任何文件在关闭之前不会真正写入磁盘。

1.3 `zipfile`.**write**(filename[, arcname[, compress_type]])

> acrname是压缩文件中该文件的名字，默认情况下和filename一样 
>
> compress_type的存在是因为zip文件允许被压缩的文件可以有不同的压缩类型。

1.4 `zipfile`.**extractall**([path[, member[, password]]])

> path解压缩目录
>
> member需要解压缩的文件名儿列表
>
> password当zip文件有密码时需要该选项
>
> > Extract all members from the archive to the current working directory. 
> >
> > *path* specifies a different directory to extract to. *members* is optional and must be a subset of the list returned by [`namelist()`](https://docs.python.org/3/library/zipfile.html#zipfile.ZipFile.namelist). *pwd* is the password used for encrypted files.

1. `ZipFile.`**namelist**()

3. 