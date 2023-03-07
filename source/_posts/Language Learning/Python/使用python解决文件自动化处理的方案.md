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



