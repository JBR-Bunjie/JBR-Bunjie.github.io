# numpy

```import numpy as np```

## api内容：

#### > 数组的创建：

1. np.array()

   > numpy.array(object, dtype = None, copy = True, order = None, subok = False, ndmin = 0)
   >
   > ​	object: 数组或嵌套的数列
   >
   > ​	dtype:数组元素的数据类型，可选
   >
   > ​	copy:对象是否需要复制，可选
   >
   > ​	order:创建数组的样式，C为行方向，F为列方向，A为任意方向（默认）
   >
   > ​	subok:默认返回一个与基类类型一致的数组
   >
   > ​	ndmin:指定生成数组的最小维度

   ```python
   > a = np.array([ [1, 2, 3], [4, 5, 6], [7, 8, 9] ])
   > a
   array([[1, 2, 3],
          [4, 5, 6],
          [7, 8, 9]])
   ```

2. np.zeros()

   > numpy.zeros(shape, dtype = float, order = 'C')
   >
   > ​	shape:数组形状
   >
   > ​	order:'C' 用于 C 的行数组，或者 'F' 用于 FORTRAN 的列数组(C中的二维数组行优先，Fortran中的二维数组列优先)

   ```python
   > a = np.zeros([3, 3])
   > a
   array([[0., 0., 0.],
          [0., 0., 0.],
          [0., 0., 0.]]) #print(a[0][0])的结果是0.0 且 a[0][0] == 0结果为True!
   ```

3. np.ones() 

   > numpy.ones(shape, dtype = None, order = 'C')

   ```python
   > a = np.ones([3, 3])
   > a
   array([[1., 1., 1.],
          [1., 1., 1.],
          [1., 1., 1.]]) #print(a[0][0])的结果是1.0 且 a[0][0] == 1结果为True!
   ```

4. np.empty()

   > numpy.empty(shape, dtype = float, order = 'C')

   ```python
   > a = np.empty([2, 2])
   > a
   array([[1.18498951e-303, 5.97409933e-299],
          [7.54793231e+168, 4.94065646e-324]]) #注意 − 数组元素为随机值，因为它们未初始化。
   ```

5. np.full()

   ```python
   > np.full([3, 3], 6)
   array([[6, 6, 6],
          [6, 6, 6],
          [6, 6, 6]])
   ```

6. np.eye()

   ```python
   > np.eye(4)
   array([[1., 0., 0., 0.],
          [0., 1., 0., 0.],
          [0., 0., 1., 0.],
          [0., 0., 0., 1.]]) # 生成的矩阵称为“对角矩阵”
   ```

7. np.arange()

   > np.arange(start, stop, step, dtype)

8. np.linspace()

   > np.linspace(start, stop, num=50, endpoint=True, retstep=False, dtype=None)
   >
   > 用于创建等差数列

9. np.logspace()

   > np.logspace(start, stop, num=50, endpoint=True, base=10.0, dtype=None)
   >
   > 用于创建等比数列

10. np.diag()

    ```python
    > np.diag([1, 2, 3])
    array([[1, 0, 0],
           [0, 2, 0],
           [0, 0, 3]])
    ```

11. np.tri()

    ```python
    > np.tri(3)
    array([[1., 0., 0.],
           [1., 1., 0.],
           [1., 1., 1.]]) #np.tri(3)[0][0] == 1的结果为True!
    ```

12. np.vander()

    ```python
    > np.vander([3, 4, 5, 6])
    array([[ 27,   9,   3,   1],
           [ 64,  16,   4,   1],
           [125,  25,   5,   1],
           [216,  36,   6,   1]])
    ```

#### > 数组属性：

> 以array = np.vander([3, 4, 5, 6])生成的数组来做演示

1. array.shape

   ```python
   > array.shape
   (4, 4) # type(array.shape)返回：<class 'tuple'>——是“元组”
   ```

2. array.size

   ```python
   > array.size
   16
   ```

3. array.T

   ```python
   > array.T
   array([[ 27,  64, 125, 216],
          [  9,  16,  25,  36],
          [  3,   4,   5,   6],
          [  1,   1,   1,   1]]) #即：矩阵的倒置
   ```

4. array.real

   ```python
   > array.real
   array([[ 27,   9,   3,   1],
          [ 64,  16,   4,   1],
          [125,  25,   5,   1],
          [216,  36,   6,   1]]) #复数的实部
   ```

5. array.imag

   ```python
   > array.imag
   array([[0, 0, 0, 0],
          [0, 0, 0, 0],
          [0, 0, 0, 0],
          [0, 0, 0, 0]]) #复数的虚部
   ```

6. array.itemsize
7. array.
8. array.flags
9. array.dtype
10. array.ndim

#### > 数组的操作：

1. array.copy() 

   > 显式复制

   ```python
   > array = np.vander([4, 4])
   
   > cc = array.copy()
   > cc[0][0] = 1000
   > cc
   array([[1000,    9,    3,    1],
          [  64,   16,    4,    1],
          [ 125,   25,    5,    1],
          [ 216,   36,    6,    1]])
   > array
   array([[ 27,   9,   3,   1],
          [ 64,  16,   4,   1],
          [125,  25,   5,   1],
          [216,  36,   6,   1]])
   
   > cc = array
   > cc[0][0] = 1000
   > cc
   array([[1000,    9,    3,    1],
          [  64,   16,    4,    1],
          [ 125,   25,    5,    1],
          [ 216,   36,    6,    1]])
   > array
   array([[1000,    9,    3,    1],
          [  64,   16,    4,    1],
          [ 125,   25,    5,    1],
          [ 216,   36,    6,    1]])
   ```

2. array.reshape()

   ```python 
   > array = np.vander([3, 4, 5, 6])
   > cc = array.reshape(2, 8)
   > cc
   array([[ 27,   9,   3,   1,  64,  16,   4,   1],
          [125,  25,   5,   1, 216,  36,   6,   1]])
   > array
   array([[ 27,   9,   3,   1],
          [ 64,  16,   4,   1],
          [125,  25,   5,   1],
          [216,  36,   6,   1]])
   ```

3. array.resize()

   ```python
   # 接array.reshape()
   > cc = array.resize(2, 8)
   > cc # 没有返回值！cc变为空！
   > array
   array([[ 27,   9,   3,   1,  64,  16,   4,   1],
          [125,  25,   5,   1, 216,  36,   6,   1]])
   ```

   

4. array.flatten()

   ```python
   # 另接array.reshape()
   > cc = array.flatten()
   > cc
   array([ 27,   9,   3,   1,  64,  16,   4,   1, 125,  25,   5,   1, 216,
           36,   6,   1])
   > array
   array([[ 27,   9,   3,   1],
          [ 64,  16,   4,   1],
          [125,  25,   5,   1],
          [216,  36,   6,   1]])
   ```

5. array.max()

   > 取全数组的最大值

6. array.min()

   > 取全数组的最小值

#### > 数组的索引：

1. 切片

   ```python
   > array = np.vander([3, 4, 5, 6])
   > array[1:3, 1:3]
   array([[16,  4],
          [25,  5]])
   > array[1:3, 1:3][0][0]
   16
   ```

2. 键值索引

   ```python
   > array = np.vander([3, 4, 5, 6])
   > array[0, 2]
   3
   > array[0][2]
   3
   > array[[0, 2], [1, 3]]
   array([9, 1]) #规则：array[Ⅰ,Ⅱ]两处，Ⅰ处一定是行坐标，如果此处是一个列表，则同时锁定列表中数字所指定的所有行，Ⅱ处一定是列坐标且应用规则同理！
   ```

3. np.ix_ 索引（键值索引加强版）

   > 解释：np.ix_() 是将前后两个[ ]中的所有内容全部一一配对！

   ```python
   > array = np.vander([3, 4, 5, 6, 7, 8])
   > array
   array([[  243,    81,    27,     9,     3,     1],
          [ 1024,   256,    64,    16,     4,     1],
          [ 3125,   625,   125,    25,     5,     1],
          [ 7776,  1296,   216,    36,     6,     1],
          [16807,  2401,   343,    49,     7,     1],
          [32768,  4096,   512,    64,     8,     1]])
   > array[np.ix_([0, 1, 2], [1, 2, 3])]
   array([[ 81,  27,   9],
          [256,  64,  16],
          [625, 125,  25]])
   ```

4. np.nditer 索引

   ```python
   > array = np.np.vander([3, 4, 5])
   > for i in array:
       print(i)
   
   [9 3 1]
   [16  4  1]
   [25  5  1]
   > for i in np.nditer(array):
       print(i)
      
   9
   3
   1
   16
   4
   1
   25
   5
   1
   ```

#### > 数组的拼接：

1. vstack

   > v: vertical 

   ```python
   > array1 = np.ones([3, 3])
   > array2 = zeros([3, 3])
   > array = np.vstack([array1, array2])
   > array
   array([[1., 1., 1.],
          [1., 1., 1.],
          [1., 1., 1.],
          [0., 0., 0.],
          [0., 0., 0.],
          [0., 0., 0.]])
   > array.shape
   (6, 3)
   ```

2. hstack

   > h: horizontal

   ```python
   # 接vstack
   > array = np.hstack([array2, array1])
   > array
   array([[0., 0., 0., 1., 1., 1.],
          [0., 0., 0., 1., 1., 1.],
          [0., 0., 0., 1., 1., 1.]])
   > array.shape
   (3, 6)
   ```

3. stack

   ```python
   # 接vstack
   > array = np.stack([array1, array2])
   > array
   array([[[1., 1., 1.],
           [1., 1., 1.],
           [1., 1., 1.]],
          [[0., 0., 0.],
           [0., 0., 0.],
           [0., 0., 0.]]])
   > array.shape
   (2, 3, 3)
   ```

#### > 数组的拆分：

> 前提：
>
> /> array = np.vander([3, 4, 5, 6])
> /> array
>
> array([	[ 27,   9,   3,   1],
>      	  	[ 64,  16,   4,   1],
>        		[125,  25,   5,   1],
>        		[216,  36,   6,   1]])

1. vsplit

   ```python
   > np.vsplit(array, 2)
   [array([[27,  9,  3,  1],
          [64, 16,  4,  1]]), array([[125,  25,   5,   1],
          [216,  36,   6,   1]])]
   ```

2. hsplit

   ```python
   > np.hsplit(array, 2)
   [array([[ 27,   9],
          [ 64,  16],
          [125,  25],
          [216,  36]]),array([[3, 1],
          [4, 1],
          [5, 1],
          [6, 1]])]
   ```

3. split

   ```python
   > np.split(array, 2)
   [array([[27,  9,  3,  1],
          [64, 16,  4,  1]]), array([[125,  25,   5,   1],
          [216,  36,   6,   1]])]
   ```

#### > 从已有内容中创建numpy数组：

1. ### np.asarray()

   > numpy.asarray(a, dtype = None, order = None) 
   >
   > ​	a：任意形式的输入参数，可以是，列表, 列表的元组, 元组, 元组的元组, 元组的列表，多维数组

   ```python
   > a = [1, 2, 3]
   > b = np.array([1, 2, 3])
   > type(a)
   <class 'list'>
   > type(b)
   <class 'numpy.ndarray'>
   > a = np.asarray(a)
   > type(a)
   <class 'numpy.ndarray'>
   
   > c = (1, 2, 3)
   > type(c)
   <class 'tuple'>
   > c = np.asarray(c)
   > type(c)
   <class 'numpy.ndarray'>
   ```

2. np.frombuffer()

3. np.fromiter()

4. 