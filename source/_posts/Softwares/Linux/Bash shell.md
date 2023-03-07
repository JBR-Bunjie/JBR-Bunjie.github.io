尝试进行

数据类型：

整数？、字符串



整数计算 - expr command (evaluate expressions)：

```bash
expr 1 + 2
```

- attention to the space
- use \\ in your command to make a reverse, for example \\\* and \\\(

```bash
n=`expr 1 \* 2`
echo ${n}
```

相似的运算符有：\(\(\)\), let, $\[\]

- (()):

```bash
for((i=0;i<num;i++))
do
	((total+=i))
done
# 该语句是对shell中算数及赋值运算的拓展，其中的所有表达式与c几乎一致，能够进行逻辑及四则运算
```

- let:

```bash
i=0
while((i<=5))
do
	echo $i
	let i++
done
# 1. let无需间隔每个字符
# 2. let
```

- $\[\]

```bash
echo $[3+5]
# 1. $[]会对表达式进行整数运算计算
# 2. $[]中的变量无需手动加上$，当然，你可以选择加上
# 3. $[]中的数字无需间隔
# 4. $[]不能单独使用，必须让计算结果有所指向
```





String









执行：

source, exec及shell运行的区别

sh方式

source方式

exec方式