# 第5章 Pandas入门

>   Getting Started with pandas 

pandas、NumPy、SciPy、matplotlib经常一起使用

-   NumPy适合处理同质性数组
-   Pandas适合处理表格型、异质性数据

## 5.1 数据结构介绍

>   Introduction to pandas Data Structures 

入门Pandas，需要熟悉两个最常用的工具数据结构：Series和 DataFrame，这是非常重要的基础。

### 5.1.1 Series

Series是一种一维数组对象，它由索引和值构成：值+index索引。


```python
# 默认索引 0，1，2
obj = pd.Series([4, 7, -5, 3])
print(obj)
# 0    4
# 1    7
# 2   -5
# 3    3
# dtype: int64

# 单独得到值
print(obj.values)
# array([ 4,  7, -5,  3], dtype=int64)

# 单独得到索引
print(obj.index)  
# RangeIndex(start=0, stop=4, step=1)

# like range(4)
# 跟range的原理一样，[0,4),左闭右开
```

索引序列可以自己创建的，可以自己使用标签标识每个数据点，选择数据时也可以按照自定义的标签进行索引。


```python
# 自定义索引
obj2 = pd.Series([4, 7, -5, 3], index=['d', 'b', 'a', 'c'])
print(obj2)
# d    4
# b    7
# a   -5
# c    3
# dtype: int64

print(obj2.index)
# Index(['d', 'b', 'a', 'c'], dtype='object')
```


```python
# 索引元素
print(obj2['a'])
# -5

# 用索引改值，此处相当于重新建立了一个新的ndarray，而非重排索引
obj2['d'] = 6
print(obj2[['c', 'a', 'd']])
# c    3
# a   -5
# d    6
# dtype: int64
```

使用布尔值数组进行过滤元素：


```python
# 这个也算是布尔值索引
print(obj2[obj2 > 0])
# d    6
# b    7
# c    3
# dtype: int64
```

应用数学函数


```python
# 数值运算
print(obj2 * 2)
# d    12
# b    14
# a   -10
# c     6
# dtype: int64

# exp：Exponential以e为底的指数曲线
print(np.exp(obj2))
# d     403.428793
# b    1096.633158
# a       0.006738
# c      20.085537
# dtype: float64
```


```python
# 长度固定且有序的字典，可以判断
print('b' in obj2)
# True

print('e' in obj2)
# False
```

如果现在已经有一个字典，可以使用该字典生成一个Series.而且，当你把字典传递给Series构造函数时，Series索引序列将是排序好的字典键。

如果你想自己决定索引顺序，可以把字典键按照你想要的顺序传递给构造函数：


```python
# 可以用字典生成series，默认的index就是字典的key
sdata = {'Ohio': 35000, 'Texas': 71000, 'Oregon': 16000, 'Utah': 5000}
obj3 = pd.Series(sdata)
print(obj3)
# Ohio      35000
# Texas     71000
# Oregon    16000
# Utah       5000
# dtype: int64
```


```python
# 字典生成series，自定义index
states = ['California', 'Ohio', 'Oregon', 'Texas']
obj4 = pd.Series(sdata, index=states)
print(obj4)
# California        NaN
# Ohio          35000.0
# Oregon        16000.0
# Texas         71000.0
# dtype: float64
```

因为`index`在原来的字典`key`值，就对应取值，`index`的`California`不在`key`值里，就取`NaN`。自定义索引内`index`没有`Utah`，所以这个值会被丢弃

先看看如何判断有没有数据缺失，也就是`NaN`。


```python
# NaN返回True
print(pd.isnull(obj4))
# California     True
# Ohio          False
# Oregon        False
# Texas         False
# dtype: bool

# 不是NaN返回True
print(pd.notnull(obj4))
# California    False
# Ohio           True
# Oregon         True
# Texas          True
# dtype: bool

# 另一种写法
print(obj4.isnull())
# California     True
# Ohio          False
# Oregon        False
# Texas         False
# dtype: bool
```

自动补齐：


```python
# 看一眼相加是是什么效果
# 看一下obj3，四个数，注意index
print(obj3)
# Ohio      35000
# Texas     71000
# Oregon    16000
# Utah       5000
# dtype: int64

# 看一眼obj4，注意index
# 跟obj3有三个是重复的
print(obj4)
# California        NaN
# Ohio          35000.0
# Oregon        16000.0
# Texas         71000.0
# dtype: float64

## 相加之后所有的index都被保留
## 共同的三个index对应value数值相加
## 非共同index变成NaN
print(obj3 + obj4)
# California         NaN
# Ohio           70000.0
# Oregon         32000.0
# Texas         142000.0
# Utah               NaN
# dtype: float64
```

Series对象自身及其索引都有`name`属性，也就是说可以对Series序列包括其索引进行命名：


```python
obj4.name = 'population'
obj4.index.name = 'state'
print(obj4)
# state是变成名字了嘛

# state
# California        NaN
# Ohio          35000.0
# Oregon        16000.0
# Texas         71000.0
# Name: population, dtype: float64
```

Series的索引还可以通过按位置赋值的方式进行改变：


```python
print(obj)
# 0    4
# 1    7
# 2   -5
# 3    3
# dtype: int64

obj.index = ['Bob', 'Steve', 'Jeff', 'Ryan']
print(obj)
# Bob      4
# Steve    7
# Jeff    -5
# Ryan     3
# dtype: int64
```

### 5.1.2 DataFrame

-   DataFrame表示矩阵的数据表，它是二维的。DataFrame既有行索引又有列索引，它可以被视为一个共享相同索引的Series的字典。

-   从DataFrame中选取的列是数据的视图而不是拷贝，因此，对Series的修改会直接映射到DataFrame中

-   如果需要复制，应当显式的使用Series的copy方法。

最常用的构建DataFrame的方法：利用包含等长度列表或Numpy数组的字典来形成DataFrame。如下：


```python
# 字典生成DataFrame
data = {'state': ['Ohio', 'Ohio', 'Ohio', 'Nevada', 'Nevada', 'Nevada'],
        'year': [2000, 2001, 2002, 2001, 2002, 2003],
        'pop': [1.5, 1.7, 3.6, 2.4, 2.9, 3.2]}
frame = pd.DataFrame(data)
print(frame)
#     state  year  pop
# 0    Ohio  2000  1.5
# 1    Ohio  2001  1.7
# 2    Ohio  2002  3.6
# 3  Nevada  2001  2.4
# 4  Nevada  2002  2.9
# 5  Nevada  2003  3.2
```

`.head(val)`方法，其中`val`为展示前多少行，默认值为5。


```python
print(frame.head()) # 只看前五行
#     state  year  pop
# 0    Ohio  2000  1.5
# 1    Ohio  2001  1.7
# 2    Ohio  2002  3.6
# 3  Nevada  2001  2.4
# 4  Nevada  2002  2.9
# 5  Nevada  2003  3.2
```

默认情况下，DataFrame的列是按照所传字典的顺序排列的，但是你也可以自己指定列的顺序：


```python
# 改变列的顺序
print(pd.DataFrame(data, columns=['year', 'state', 'pop']))
#    year   state  pop
# 0  2000    Ohio  1.5
# 1  2001    Ohio  1.7
# 2  2002    Ohio  3.6
# 3  2001  Nevada  2.4
# 4  2002  Nevada  2.9
# 5  2003  Nevada  3.2
```

如果你传入了不包含在字典中的列，结果将会出现缺失值：


```python
# 如果列不在DataFrame里，会显示缺失值
frame2 = pd.DataFrame(data, columns=['year', 'state', 'pop', 'debt'],
                      index=['one', 'two', 'three', 'four',
                             'five', 'six'])
print(frame2)
#        year   state  pop debt
# one    2000    Ohio  1.5  NaN
# two    2001    Ohio  1.7  NaN
# three  2002    Ohio  3.6  NaN
# four   2001  Nevada  2.4  NaN
# five   2002  Nevada  2.9  NaN
# six    2003  Nevada  3.2  NaN

# 查看列名称
print(frame2.columns)
# Index(['year', 'state', 'pop', 'debt'], dtype='object')

# 查看某列
print(frame2['state'])
# one        Ohio
# two        Ohio
# three      Ohio
# four     Nevada
# five     Nevada
# six      Nevada
# Name: state, dtype: object
```

如果想查看DataFrame中的某一列，按照正常方式检索即可，所得结果是一个Series：

```python
# 查看某列 可用frame2.year与frame2['year']两种方式
print(frame2.year)
# one      2000
# two      2001
# three    2002
# four     2001
# five     2002
# six      2003
# Name: year, dtype: int64
```

如果想查看DataFrame中的某一行，可以通过位置或特殊属性`loc`进行选取。例如，我想查看Frame的第3行：

```python
# 查看行
print(frame2.loc['three'])
# year     2002
# state    Ohio
# pop       3.6
# debt      NaN
# Name: three, dtype: object
```

DataFrame中，列的引用也是可以修改的。比如现在，country这一列是空的，我们可以对其进行赋值。且赋值为标量值或值数组均可，只需保持长度一致即可。

甚至可以将Series赋值给一个列，如果被赋值的列不存在，将会生成一个新的列。

```python
# 给一列赋值，给一个数，默认所有的值都得这个
frame2['debt'] = 16.5
print(frame2)
#        year   state  pop  debt
# one    2000    Ohio  1.5  16.5
# two    2001    Ohio  1.7  16.5
# three  2002    Ohio  3.6  16.5
# four   2001  Nevada  2.4  16.5
# five   2002  Nevada  2.9  16.5
# six    2003  Nevada  3.2  16.5

# 给一个array，按顺序赋值
# 6.表示一位小数
frame2['debt'] = np.arange(6.)
print(frame2)
#        year   state  pop  debt
# one    2000    Ohio  1.5   0.0
# two    2001    Ohio  1.7   1.0
# three  2002    Ohio  3.6   2.0
# four   2001  Nevada  2.4   3.0
# five   2002  Nevada  2.9   4.0
# six    2003  Nevada  3.2   5.0
```


```python
# 通过series给列赋值，先定义一个
# 然后部分的index相同
val = pd.Series([-1.2, -1.5, -1.7], index=['two', 'four', 'five'])
print(val)
# two    -1.2
# four   -1.5
# five   -1.7
# dtype: float64

# index相同的部分会赋值
frame2['debt'] = val
print(frame2)
#        year   state  pop  debt
# one    2000    Ohio  1.5   NaN
# two    2001    Ohio  1.7  -1.2
# three  2002    Ohio  3.6   NaN
# four   2001  Nevada  2.4  -1.5
# five   2002  Nevada  2.9  -1.7
# six    2003  Nevada  3.2   NaN

frame2['eastern'] = frame2.state == 'Ohio'
print(frame2)
#        year   state  pop  debt  eastern
# one    2000    Ohio  1.5   0.0     True
# two    2001    Ohio  1.7   1.0     True
# three  2002    Ohio  3.6   2.0     True
# four   2001  Nevada  2.4   3.0    False
# five   2002  Nevada  2.9   4.0    False
# six    2003  Nevada  3.2   5.0    False

# 新加一个列，eastern，应该是判断是否属于东部
# 给的值是布尔值True或False
# 判断的依据，是该行的state是不是Ohio
```


```python
# 删除一列，用del
del frame2['eastern']
print(frame2)
#        year   state  pop  debt
# one    2000    Ohio  1.5   0.0
# two    2001    Ohio  1.7   1.0
# three  2002    Ohio  3.6   2.0
# four   2001  Nevada  2.4   3.0
# five   2002  Nevada  2.9   4.0
# six    2003  Nevada  3.2   5.0

print(frame2.columns)
# Index(['year', 'state', 'pop', 'debt'], dtype='object')
```

嵌套字典创建DataFrame


```python
# 创建DataFrame
# 用嵌套字典赋值
# 一级键作为列
# 二级键作为索引
pop = {'Nevada': {2001: 2.4, 2002: 2.9},
       'Ohio': { 2001: 1.7, 2002: 3.6, 2000: 1.5}}

# 将字典作为参数传递DataFrame函数
# DataFrame对键进行联合、排序
frame3 = pd.DataFrame(pop)
print(frame3)
#       Nevada  Ohio
# 2001     2.4   1.7
# 2002     2.9   3.6
# 2000     NaN   1.5
```

DataFrame转置


```python
# 可以使用NumPy语法对DataFrame进行转置操作
print(frame3.T)
#         2001  2002  2000
# Nevada   2.4   2.9   NaN
# Ohio     1.7   3.6   1.5
```

利用pop嵌套字典创建的DataFrame，也可自定义索引。

```python
# 一样可以自定义索引
# 2000被舍弃
# 2003被创建，没有值得地方默认NaN
# 自己指定索引，就不会被排序了
print(pd.DataFrame(pop, index=[2002, 2001, 2003]))
#       Nevada  Ohio
# 2002     2.9   3.6
# 2001     2.4   1.7
# 2003     NaN   NaN

# 用frame的series创建DataFrame
# 一级关键字还是列
# 二级关键字，某人是原来的index，2000，2001，2002
pdata = {'Ohio': frame3['Ohio'][:-1],
         'Nevada': frame3['Nevada'][:2]}
# Ohio，Nevada排序，LMNOPQ，所以是Nevada，Ohio
# Nevada里面，对应frame3的[0,2)我要是发明语言，切片就这么写
# Ohio对应的[0,-1)，倒数第一个为止，也就是倒数第一个不要

print(frame3) ## 看一眼frame3
#       Nevada  Ohio
# 2001     2.4   1.7
# 2002     2.9   3.6
# 2000     NaN   1.5

print(pd.DataFrame(pdata))
#       Ohio  Nevada
# 2001   1.7     2.4
# 2002   3.6     2.9

# index和columns有name属性
frame3.index.name = 'year'
frame3.columns.name = 'state'
print(frame3)
# state  Nevada  Ohio
# year               
# 2001      2.4   1.7
# 2002      2.9   3.6
# 2000      NaN   1.5

# 与Series类似，values值以二维nadarry返回
print(frame3.values)
# array([[nan, 1.5],
#        [2.4, 1.7],
#        [2.9, 3.6]])

print(frame2.values) # 一样的
# array([[2000, 'Ohio', 1.5, nan, True],
#        [2001, 'Ohio', 1.7, -1.2, True],
#        [2002, 'Ohio', 3.6, nan, True],
#        [2001, 'Nevada', 2.4, -1.5, False],
#        [2002, 'Nevada', 2.9, -1.7, False],
#        [2003, 'Nevada', 3.2, nan, False]], dtype=object)
```

**表5-1 DataFrame可以接受的参数类型**

| 类型                       | 注释                                                    |
| -------------------------- | ------------------------------------------------------- |
| 2D ndarray                 | 二维数据矩阵                                            |
| 数组、列表和元组组成的字典 | 每个序列成为DataFrame的一列                             |
| NumPy结构化数组            | 与数组构成的字典一致                                    |
| Series构成的字典           | 每个值成为一列                                          |
| 字典构成的字典             | 每一个内部字典成为一列                                  |
| 字典或Series构成的列表     | 列表中的元素形成DataFrame的一行，键或Series的索引形成列 |
| 列表或元组构成的列表       | 与2D ndarray的情况一致                                  |
| 其他DataFrame              | 如果不显示传递索引，则利用原索引                        |
| NumPy MaskedArray          |                                                         |

### 5.1.3 索引对象

>   Index Objects 

pandas中的索引对象是用于存储轴标签和其它元数据的（例如轴名称或标签）。索引对象是不可变的，用户无法对其进行修改。

与Python集合不同，Pandas索引对象可以包含重复标签，根据重复标签进行筛选时，会选出所有重复标签对应的数据。

数组或标签序列可以转换出索引类型

**<u>特点：不可变，可重复</u>**


```python
import pandas as pd # 默认的导入模式
import numpy as np

obj = pd.Series(range(3), index=['a', 'b', 'c'])
print(obj)
# a    0
# b    1
# c    2
# dtype: int64

index1 = obj.index
print(index1)
# Index(['a', 'b', 'c'], dtype='object')

# 切片功能跟列表很像
print(index1[1:])
# Index(['b', 'c'], dtype='object')

# # 不可修改
# index1[1] = 'd'
# # 修改会报错
```

不可变性使得在多种数据结构中分享索引更安全


```python
# 生成一个索引对象
labels = pd.Index(np.arange(3))
print(labels)
# Int64Index([0, 1, 2], dtype='int64')

# 生成一个序列，就用我们上面定义的索引
obj2 = pd.Series([1.5, -2.5, 0], index=labels)
print(obj2)
# 0    1.5
# 1   -2.5
# 2    0.0
# dtype: float64

# 我滴妈，还可以判断
print(obj2.index is labels)
# True
```

还可以各种判断


```python
# 定义一个表格
pop = {'Nevada': {2001: 2.4, 2002: 2.9},
       'Ohio': { 2001: 1.7, 2002: 3.6, 2000: 1.5}}
frame3 = pd.DataFrame(pop)
print(frame3)

#       Nevada  Ohio
# 2001     2.4   1.7
# 2002     2.9   3.6
# 2000     NaN   1.5

print(frame3.columns) # 列名也是Index类型对象
# Index(['Nevada', 'Ohio'], dtype='object')

print('Ohio' in frame3.columns) # 判断Ohio是否在这个Index对象里
# True

print(2003 in frame3.index) # 判断
# False
```


可重复

```python
dup_labels = pd.Index(['foo', 'foo', 'bar', 'bar'])
print(dup_labels)
# Index(['foo', 'foo', 'bar', 'bar'], dtype='object')
```

**表5-2：index对象的方法和属性**

类似列表、集合：

| 方法         | 描述                                       |
| ------------ | ------------------------------------------ |
| append       | 将额外的索引加入现有的原索引，形成新的索引 |
| difference   | 差集                                       |
| intersection | 交集                                       |
| union        | 并集                                       |
| isin         | 判断                                       |
| delete       | 删除                                       |
| drop         | 删除指定索引值，并产生新的索引             |
| insert       | 插入                                       |
| is_monotonic | 如果索引序列递增返回True                   |
| is_unique    | 如果索引序列唯一返回True                   |
| unique       | 计算索引的唯一值序列                       |

## 5.2 主要功能

>   Essential Functionality 

本节将指引你了解与Series或DataFrame中数据交互的基础机制

基础，最重要的特性

### 5.2.1 重置索引

>   Reindexing 

reindex是Pandas对象的重要方法，该方法可以创建一个符合新索引的新对象。

当调用该方法时，数据会按照新的索引重新排序，若某个索引值之前不存在，则会引入缺失值。


```python
obj = pd.Series([4.5, 7.2, -5.3, 3.6], index=['d', 'b', 'a', 'c'])
# 这个索引dbac
print(obj)
# d    4.5
# b    7.2
# a   -5.3
# c    3.6
# dtype: float6

obj2 = obj.reindex(['a', 'b', 'c', 'd', 'e'])
# 取obj的值
# reindex重置索引为abcde
# 改变顺序，原来有值取原来的值，没有的默认NaN
print(obj2)
# a   -5.3
# b    7.2
# c    3.6
# d    4.5
# e    NaN
# dtype: float64
```

有的时候需要自动填充一些值，要用method。

**比如method = ffill，将值向前填充：**


```python
obj3 = pd.Series(['blue', 'purple', 'yellow'], index=[0, 2, 4])
print(obj3)
# 0      blue
# 2    purple
# 4    yellow
# dtype: object  

print(obj3.reindex(range(7), method='ffill'))
# 0      blue
# 1      blue
# 2    purple
# 3    purple
# 4    yellow
# 5    yellow
# 6    yellow
# dtype: object
```

重建索引：改变行索引，使用`columns`关键字改变列索引。使用`loc`同时改变行索引和列索引


```python
frame = pd.DataFrame(np.arange(9).reshape((3, 3)),
                     index=['a', 'c', 'd'],
                     columns=['Ohio', 'Texas', 'California'])
print(frame)
#    Ohio  Texas  California
# a     0      1           2
# c     3      4           5
# d     6      7           8

frame2 = frame.reindex(['a', 'b', 'c', 'd'])
print(frame2)
#    Ohio  Texas  California
# a   0.0    1.0         2.0
# b   NaN    NaN         NaN
# c   3.0    4.0         5.0
# d   6.0    7.0         8.0
```

**用`columns`改变列索引，用`loc`同时改变行列索引：**


```python
states = ['Texas', 'Utah', 'California']
# 换顺序删除Ohio，加入Utah
print(frame.reindex(columns=states))
#    Texas  Utah  California
# a      1   NaN           2
# c      4   NaN           5
# d      7   NaN           8

print(frame.loc[['a', 'b', 'c','d'], states])
# 此处会报错，loc方法不支持传入不存在的索引
```

此处利用给reindex传入多参数，同时重建行列索引

```python
print(frame.reindex(index=['a','b','c','d'],columns=states))
#    Texas  Utah  California
# a      1   NaN           2
# b      NaN NaN           NaN
# c      4   NaN           5
# d      7   NaN           8

print(frame.reindex(index=['a','b','c','d'],columns=states).ffill())
#    Texas  Utah  California
# a    1.0   NaN         2.0
# b    1.0   NaN         2.0
# c    4.0   NaN         5.0
# d    7.0   NaN         8.0

# 此处不能使用method参数，应把'method=ffill'修改为.ffill()方法
```

**表5-3：reindex方的参数**

| 参数       | 描述                                         |
| ---------- | -------------------------------------------- |
| index      | 索引或其他序列型数据结构                     |
| method     | ffill向填充，bfill向后填充                   |
| fill_value | 缺失值的默认值                               |
| limit      | 限制，填充时，限制数量？                     |
| tolerance  | 宽容，填充时，容错？                         |
| level      | 匹配MultiIndex级别的简单索引，否则选择自己   |
| copy       | copy =True，复制底层数据，=False，不复制数据 |

### 5.2.2 轴向上删除条目

>   Dropping Entries from an Axis 

本小节将介绍`drop`方法，该方法会返回一个经删除修改后的新对象，原对象没有变。

默认情况下，调用drop是根据行标签去删除相应的值，若要在列上删除值，需要传递参数 `axis=1`或`axis='columns'`


```python
obj5 = pd.Series(np.arange(5.), index=['a', 'b', 'c', 'd', 'e'])
print(obj5)
# a    0.0
# b    1.0
# c    2.0
# d    3.0
# e    4.0
# dtype: float64

# 生成一个新的对象
new_obj = obj5.drop('c')
print(new_obj)
# a    0.0
# b    1.0
# d    3.0
# e    4.0
# dtype: float64

## 原来的obj5并没有改变
obj5.drop(['d', 'c'])
print(obj5)
# a    0.0
# b    1.0
# c    2.0
# d    3.0
# e    4.0
# dtype: float64

data5 = pd.DataFrame(np.arange(16).reshape((4, 4)),
                    index=['Ohio', 'Colorado', 'Utah', 'New York'],
                    columns=['one', 'two', 'three', 'four'])
print(data5)
#           one  two  three  four
# Ohio        0    1      2     3
# Colorado    4    5      6     7
# Utah        8    9     10    11
# New York   12   13     14    15

print(data5.drop(['Colorado', 'Ohio']))  # 默认就是删掉索引对应的行或列
#           one  two  three  four
# Utah        8    9     10    11
# New York   12   13     14    15

print(data5.drop('two', axis=1))  # 可以指定是哪个轴，axis=1
#           one  three  four
# Ohio        0      2     3
# Colorado    4      6     7
# Utah        8     10    11
# New York   12     14    15

print(data5.drop(['two', 'four'], axis='columns'))  # 用axis=columns也可以
#           one  three
# Ohio        0      2
# Colorado    4      6
# Utah        8     10
# New York   12     14

```

想要删除替换原来的表格，直接用`inplace=True`，drop的值会被清除，原来的表格会被改变。


```python
obj.drop('c', inplace=True)
print(obj)
# d    4.5
# b    7.2
# a   -5.3
# dtype: float64
```

### 5.2.3 索引、选择、过滤

>   Indexing, Selection, and Filtering 

又要干啥，直接看代码


```python
obj5 = pd.Series(np.arange(4.), index=['a', 'b', 'c', 'd'])
print(obj5)
# a    0.0
# b    1.0
# c    2.0
# d    3.0
# dtype: float64

print(obj5['b'])
# 1.0

print(obj5[1])
# 1.0

print(obj5[2:4])
# c    2.0
# d    3.0
# dtype: float64

print(obj5[['b', 'a', 'd']])
# b    1.0
# a    0.0
# d    3.0
# dtype: float64

print(obj5[[1, 3]])
# b    1.0
# d    3.0
# dtype: float64

print(obj5[obj5 < 2])
# a    0.0
# b    1.0
```

**如果用自定义的索引切片，其区间为左右都是闭区间**

**若用默认的索引（数字）切片，其区间为左闭右开**


```python
print(obj5['a':'c'])  # 自定义索引切片
# a    0.0
# b    1.0
# c    2.0
# dtype: float64

print(obj5[0:2])  # 默认索引切片
# a    0.0
# b    1.0
# dtype: float64

obj5['b':'c'] = 5  # 赋值
print(obj5)
# a    0.0
# b    5.0
# c    5.0
# d    3.0
# dtype: float64
```

使用单个值或者序列


```python
data6 = pd.DataFrame(np.arange(16).reshape((4, 4)),
                    index=['Ohio', 'Colorado', 'Utah', 'New York'],
                    columns=['one', 'two', 'three', 'four'])
print(data6)
#           one  two  three  four
# Ohio        0    1      2     3
# Colorado    4    5      6     7
# Utah        8    9     10    11
# New York   12   13     14    15

print(data6['two'])
# Ohio         1
# Colorado     5
# Utah         9
# New York    13
# Name: two, dtype: int32

print(data6[['three', 'one']])
#           three  one
# Ohio          2    0
# Colorado      6    4
# Utah         10    8
# New York     14   12

print(data6[:2])
#           one  two  three  four
# Ohio        0    1      2     3
# Colorado    4    5      6     7

print(data6[data6['three'] > 5]) ## 布尔值索引
#           one  two  three  four
# Colorado    4    5      6     7
# Utah        8    9     10    11
# New York   12   13     14    15

print(data6 < 5) # 布尔值判断
#             one    two  three   four
# Ohio       True   True   True   True
# Colorado   True  False  False  False
# Utah      False  False  False  False
# New York  False  False  False  False

data6[data6 < 5] = 0 # 布尔值索引赋值
print(data6)
#           one  two  three  four
# Ohio        0    0      0     0
# Colorado    0    5      6     7
# Utah        8    9     10    11
# New York   12   13     14    15
```

#### 5.2.3.1 使用loc和iloc选择数据

>   Selection with loc and iloc 

使用索引`loc`(轴标签）和`iloc`(整数标签）以Numpy风格的语法，从DataFrame中选出数组的行和列的子集。


```python
data6 = pd.DataFrame(np.arange(16).reshape((4, 4)),
                    index=['Ohio', 'Colorado', 'Utah', 'New York'],
                    columns=['one', 'two', 'three', 'four'])
print(data6)
#           one  two  three  four
# Ohio        0    1      2     3
# Colorado    4    5      6     7
# Utah        8    9     10    11
# New York   12   13     14    15

print(data6.loc['Colorado', ['two', 'three']])
# two      5
# three    6
# Name: Colorado, dtype: int32
      
# 用loc
# 第一个元素，一级索引，确定行，Colorado
# 第二个[]，二级索引，确定列
```

`iloc`是integer location


```python
print(data6.iloc[2, [3, 0, 1]])
# four    11
# one      8
# two      9
# Name: Utah, dtype: int32

# 用iloc，2，Utah
# 3 0 1 列

print(data6.iloc[2]) # 第二行，默认一行都取出来
# one       8
# two       9
# three    10
# four     11
# Name: Utah, dtype: int32

print(data6.iloc[[1, 2], [3, 0, 1]]) # 各种试一下
#           four  one  two
# Colorado     7    4    5
# Utah        11    8    9

print(data6.loc[:'Utah', 'two'])
# Ohio        1
# Colorado    5
# Utah        9

# Name: two, dtype: int32
# 行，[Ohio，Utah]
# 列选two

print(data6.iloc[:, :3][data6.three > 5])
#           one  two  three
# Colorado    4    5      6
# Utah        8    9     10
# New York   12   13     14
```

**表 5-4：DataFrame索引选项**

| 类型                     | 描述                             |
| ------------------------ | -------------------------------- |
| def[val]                 | 好比data6['two']，取行货列       |
| df.loc[val]              | 根据标签名单行单列               |
| df.loc[: val]            | 根据标签，类似于切片模式多行多列 |
| df.loc[val1, val2]       | val1行，val2列                   |
| df.ilo                   | 同理，只不过根据整数索引取       |
| df.at[label_i,label_j]   | 行列，取一个值                   |
| df.iat[i, j]             | 跟上面一样，用整数索引           |
| reindex                  | 通过标签选择、改变行或列         |
| get_value，set_value方法 | 根据行列标签设置单个值           |

### 5.2.4 整数索引

>   Integer Indexes 

- `loc`，根据元素的内容确定

- `iloc`，根据索引确定

- 直接用`series[]`这种，如果内容不是数字，他就跟`iloc`一样，如果内容是数字，他就跟`loc`一样


```python
ser2 = pd.Series(np.arange(3.), index=['a', 'b', 'c'])
# 非整数索引就没事儿
print(ser2)
# a    0.0
# b    1.0
# c    2.0
# dtype: float64
```

这里使用方框，情况相当于`iloc，index`


```python
print(ser2[-1]) # 默认整数索引
# 2.0
```

但你的index内容就是3，2，1

比如下面这种情况


```python
ser = pd.Series(np.arange(3.),index = [3,2,1])
# index是整数
print(ser)
# 3    0.0
# 2    1.0
# 1    2.0
# dtype: float64
```

这里使用方框，情况相当于`loc`


```python
print(ser[1]) ## 这种引用，直接默认是指的内容，location
# 2.0
```

无法启用`index`


```python
print(ser[-1]) ## 报错，不会启用index
```


为了避免出现这种歧义，通常使用`loc`和`iloc`声明你指的是内容，还是索引


```python
print(ser)
print(ser[:1]) # index,iloc

print(ser.loc[:1]) # location 左闭右闭区间

print(ser.iloc[:2]) # index location 左闭右开区间

print(ser.iloc[-1]) # index location
```

### 5.2.5 四则运算和数据调整

>   Arithmetic and Data Alignment 

两个不同索引对象之间进行相加，在没有交叠的标签位置上，内部数据对齐会产生缺失值NaN，若将两个行列完全不同的对象相加，结果将全为空。


```python
# 一个轴不同
s1 = pd.Series([7.3, -2.5, 3.4, 1.5], index=['a', 'c', 'd', 'e'])
s2 = pd.Series([-2.1, 3.6, -1.5, 4, 3.1],
               index=['a', 'c', 'e', 'f', 'g'])

print(s1+s2)
# a    5.2
# c    1.1
# d    NaN
# e    0.0
# f    NaN
# g    NaN
# dtype: float64
```

直接相加就是索引内容相同的相加，就是索引相同的相加，不同的返回NaN。

两张表相加，纵轴横轴都有所不同


```python
df1 = pd.DataFrame(np.arange(9.).reshape((3, 3)), columns=list('bcd'),
                   index=['Ohio', 'Texas', 'Colorado'])
df2 = pd.DataFrame(np.arange(12.).reshape((4, 3)), columns=list('bde'),
                   index=['Utah', 'Ohio', 'Texas', 'Oregon'])
print(df1 + df2)
#             b   c     d   e
# Colorado  NaN NaN   NaN NaN
# Ohio      3.0 NaN   6.0 NaN
# Oregon    NaN NaN   NaN NaN
# Texas     9.0 NaN  12.0 NaN
# Utah      NaN NaN   NaN NaN

# b，d，Texas，Ohio对应的有数，其他都是NaN
```

横轴完全不同，都是NaN


```python
df1 = pd.DataFrame({'A': [1, 2]})
df2 = pd.DataFrame({'B': [3, 4]})
print(df1 - df2)
#     A   B
# 0 NaN NaN
# 1 NaN NaN
```

#### 5.2.5.1 使用填充值的算法方法

>Arithmetic methods with fill values 


用`fill_value=方法`


```python
df1 = pd.DataFrame(np.arange(12.).reshape((3, 4)),
                   columns=list('abcd'))
df1.loc[0, 'b'] = np.nan
print(df1)
#      a    b     c     d
# 0  0.0  NaN   2.0   3.0
# 1  4.0  5.0   6.0   7.0
# 2  8.0  9.0  10.0  11.0

df2 = pd.DataFrame(np.arange(20.).reshape((4, 5)),
                   columns=list('abcde'))
df2.loc[1, 'b'] = np.nan
df2.loc[1, 'e'] = np.nan
print(df2)
#       a     b     c     d     e
# 0   0.0   1.0   2.0   3.0   4.0
# 1   5.0   NaN   7.0   8.0   NaN
# 2  10.0  11.0  12.0  13.0  14.0
# 3  15.0  16.0  17.0  18.0  19.0

# 设置一个NaN，缺失值
print(df1 + df2)
#       a     b     c     d   e
# 0   0.0   NaN   4.0   6.0 NaN
# 1   9.0   NaN  13.0  15.0 NaN
# 2  18.0  20.0  22.0  24.0 NaN
# 3   NaN   NaN   NaN   NaN NaN

# 两个相加，有了更多的NaN

print(df1.add(df2, fill_value=0))
#       a     b     c     d     e
# 0   0.0   1.0   4.0   6.0   4.0
# 1   9.0   5.0  13.0  15.0   NaN
# 2  18.0  20.0  22.0  24.0  14.0
# 3  15.0  16.0  17.0  18.0  19.0

# 对df1的NaN填充值，NaN=0
# df1+df2时，df1的值为0，其相加的结果就为df2的值
# 若对应元素df2=NaN，则0+NaN也为NaN

print(df2.add(df1, fill_value=0))
#       a     b     c     d     e
# 0   0.0   1.0   4.0   6.0   4.0
# 1   9.0   5.0  13.0  15.0   NaN
# 2  18.0  20.0  22.0  24.0  14.0
# 3  15.0  16.0  17.0  18.0  19.0
```

我自己在试一个例子，`fill_value`方法填充NaN数据，不过两个`df`中都为NaN的数据，该方法不会填充。


```python
df3 = pd.DataFrame(np.arange(12.).reshape((3, 4)),
                   columns=list('abcd'))
df3.loc[1, 'b'] = np.nan

df2 = pd.DataFrame(np.arange(20.).reshape((4, 5)),
                   columns=list('abcde'))
df2.loc[1, 'b'] = np.nan  ## df2，df3（1，b）都是NaN

print(df3.add(df2, fill_value=0))
# a     b     c     d     e
# 0   0.0   2.0   4.0   6.0   4.0
# 1   9.0   NaN  13.0  15.0   9.0
# 2  18.0  20.0  22.0  24.0  14.0
# 3  15.0  16.0  17.0  18.0  19.0
      
# 这种情况下补充，NaN还是NaN
```

算法方法是对每个元素的算术方法


```python
print(1 / df1) # 所有的值取倒数
#        a         b         c         d
# 0    inf       NaN  0.500000  0.333333
# 1  0.250  0.200000  0.166667  0.142857
# 2  0.125  0.111111  0.100000  0.090909

print(df1.rdiv(1))
#        a         b         c         d
# 0    inf       NaN  0.500000  0.333333
# 1  0.250  0.200000  0.166667  0.142857
# 2  0.125  0.111111  0.100000  0.090909

# 等价于1 div df1，也就是1除以df1
# 看样子r应该是翻转的意思，就是换个位置
```

重建index时也可以用`fill_value=0`这里就默认了=0


```python
print(df1.reindex(columns=df2.columns, fill_value=0))
#      a    b     c     d  e
# 0  0.0  NaN   2.0   3.0  0
# 1  4.0  5.0   6.0   7.0  0
# 2  8.0  9.0  10.0  11.0  0
```

**表 5-5：四则运算**

| 方法                | 描述 |
| ------------------- | ---- |
| add,radd            | 加   |
| sub,rsub            | 减   |
| div,rdiv            | 除   |
| floorrdiv,rfloordiv | 整除 |
| mul,rmul            | 乘   |
| pow,rpow            | 次方 |

#### 5.2.5.2 DataFrame和Series间的操作

> Operations between DataFrame and Series

DataFrame和Series之间的操作

说的这么复杂，就是一个表加减一行的情况


```python
arr = np.arange(12.).reshape((3, 4))
print(arr)
# [[ 0.  1.  2.  3.]
#  [ 4.  5.  6.  7.]
#  [ 8.  9. 10. 11.]]

print(arr[0])
# [0. 1. 2. 3.]

print(arr - arr[0])
# [[0. 0. 0. 0.]
#  [4. 4. 4. 4.]
#  [8. 8. 8. 8.]]

## 每一行都减去arr[0]
```


再来个例子


```python
frame6 = pd.DataFrame(np.arange(12.).reshape((4, 3)),
                     columns=list('bde'),
                     index=['Utah', 'Ohio', 'Texas', 'Oregon'])

print(frame6)
#           b     d     e
# Utah    0.0   1.0   2.0
# Ohio    3.0   4.0   5.0
# Texas   6.0   7.0   8.0
# Oregon  9.0  10.0  11.0

series6 = frame6.iloc[0]
print(series6)
# b    0.0
# d    1.0
# e    2.0
# Name: Utah, dtype: float64

print(frame6 - series6)
#           b    d    e
# Utah    0.0  0.0  0.0
# Ohio    3.0  3.0  3.0
# Texas   6.0  6.0  6.0
# Oregon  9.0  9.0  9.0

series7 = pd.Series(range(3), index=['b', 'e', 'f'])
print(series7)
# b    0
# e    1
# f    2
# dtype: int64

print(frame6 + series7)
#           b   d     e   f
# Utah    0.0 NaN   3.0 NaN
# Ohio    3.0 NaN   6.0 NaN
# Texas   6.0 NaN   9.0 NaN
# Oregon  9.0 NaN  12.0 NaN

# bde+bef=be
# 每行都加上series7
# 其他都是NaN
```

如果想对一列进行操作怎么办，需要使用算数方法`sub`等。


```python
print(frame6) # 这个是之前的表格
#           b     d     e
# Utah    0.0   1.0   2.0
# Ohio    3.0   4.0   5.0
# Texas   6.0   7.0   8.0
# Oregon  9.0  10.0  11.0

series8 = frame6['d']
print(series8)
# Utah       1.0
# Ohio       4.0
# Texas      7.0
# Oregon    10.0
# Name: d, dtype: float64

# 取出其中的一列

print(frame6.sub(series8, axis='index'))
#           b    d    e
# Utah   -1.0  0.0  1.0
# Ohio   -1.0  0.0  1.0
# Texas  -1.0  0.0  1.0
# Oregon -1.0  0.0  1.0

# 要用sub这个函数
# index就是人名那个轴
# frame6的每列减去d的值
```

### 5.2.6 函数应用和映射

>   Function Application and Mapping 

Numpy的通用函数对Pandas对象同样有效，例如`np.abs()`等（是对数组进行逐元素操作）

另一常见操作是将函数应用到一行或一列的一维数组上，DataFrame的`apply`方法可以实现这个功能：


```python
frame7 = pd.DataFrame(np.random.randn(4, 3), columns=list('bde'),
                     index=['Utah', 'Ohio', 'Texas', 'Oregon'])
print(frame7)
#                b         d         e
# Utah   -1.749765  0.342680  1.153036
# Ohio   -0.252436  0.981321  0.514219
# Texas   0.221180 -1.070043 -0.189496
# Oregon  0.255001 -0.458027  0.435163

print(np.abs(frame7)) # abs是绝对值
#                b         d         e
# Utah    1.749765  0.342680  1.153036
# Ohio    0.252436  0.981321  0.514219
# Texas   0.221180  1.070043  0.189496
# Oregon  0.255001  0.458027  0.435163
```

如果想实现一列中的最大值-最小值，也就是要对每一列执行同一个函数操作，用`apply`：


```python
f = lambda x: x.max() - x.min()
print(frame7.apply(f))
# b    2.004767
# d    2.051364
# e    1.342532
# dtype: float64

# 比如b这列，是0.255001-(-1.749765)=2.004767

print(frame7.apply(f, axis='columns'))
# Utah      2.902801
# Ohio      1.233757
# Texas     1.291223
# Oregon    0.893190
# dtype: float64

# 沿着columns的方向
# 每行来一次
```

返回的值可以带有多个值的Series，比如想留下最大值和最小值


```python
def f(x):
    return pd.Series([x.min(), x.max()], index=['min', 'max'])
print(frame7.apply(f))
#             b         d         e
# min -1.749765 -1.070043 -0.189496
# max  0.255001  0.981321  1.153036
```

frame7里面的是浮点数，这个浮点数保留两位，变成字符串，用`applymap`：


```python
format = lambda x: '%.2f' % x
# 这个就是保留两位小数
print(frame7.applymap(format))
#             b      d      e
# Utah     2.09  -1.25  -0.98
# Ohio    -0.56  -0.28  -1.68
# Texas   -0.23   0.61  -0.06
# Oregon   1.21  -1.06   0.84

# 这里是applymap，对每一列都进行操作
# 区别于map，是对某一列，也就是某Series进行操作

# map方法
print(frame7['e'].map(format))
# Utah      -0.98
# Ohio      -1.68
# Texas     -0.06
# Oregon     0.84
# Name: e, dtype: object
```

### 5.2.7 排序和排名

>   Sorting and Ranking 

-   按索引排序，用`sort_index`

-   按值排序，用`sort_values`

根据某些准则对数据集进行排序是一个重要的内建操作。使用`sort_index`方法，可以根据行或列索引进行字典型排序，并返回一个新的排序好的对象。


```python
obj8 = pd.Series(range(4), index=['d', 'a', 'b', 'c'])
print(obj8)
# d    0
# a    1
# b    2
# c    3
# dtype: int64.

print(obj8.sort_index())
# a    1
# b    2
# c    3
# d    0
# dtype: int64

# 默认给axis=0排序
```

```python
frame = pd.DataFrame(np.arange(8).reshape((2, 4)),
                     index=['three', 'one'],
                     columns=['d', 'a', 'b', 'c'])
print(frame.sort_index())
#        d  a  b  c
# one    4  5  6  7
# three  0  1  2  3

# 给axis=1排序需要指定
print(frame.sort_index(axis=1))
#        a  b  c  d
# three  1  2  3  0
# one    5  6  7  4

# 默认升序，想要降序ascending=False
print(frame.sort_index(axis=1, ascending=False))
#        d  c  b  a
# three  0  3  2  1
# one    4  7  6  5
```

-   除了可以根据索引进行排序之外，还可以根据序列的值进行排序，方法：`sort_values`。

-   无论是默认情况下的升序排列，还是自己设置成降序排列，所有缺失值都会被排在Series的尾部。

-   其实`sort_values`还有一个可选参数by，可以用于选择某列或多列作为排序键。

**`sort_values()`按值排序：**


```python
obj9 = pd.Series([4, 7, -3, 2])
print(obj9)
# 0    4
# 1    7
# 2   -3
# 3    2
# dtype: int64

## 默认index是0123
## 对应值

print(obj9.sort_values())
# 2   -3
# 3    2
# 0    4
# 1    7
# dtype: int64

# 按值从小到大排序
```

缺失值会被排序值Series的尾部


```python
obj10 = pd.Series([4, np.nan, 7, np.nan, -3, 2])
print(obj10.sort_values())
# 4   -3.0
# 5    2.0
# 0    4.0
# 2    7.0
# 1    NaN
# 3    NaN
# dtype: float64
```

给DataFrame，就是表格排序时，`sort_values(by=排序的根据)`：


```python
frame10 = pd.DataFrame({'b': [4, 7, -3, 2], 'a': [0, 1, 0, 1]})
print(frame10)
#    b  a
# 0  4  0
# 1  7  1
# 2 -3  0
# 3  2  1

print(frame10.sort_values(by='b'))  # 根据b这列排序
#    b  a
# 2 -3  0
# 3  2  1
# 0  4  0
# 1  7  1

print(frame10.sort_values(by=['a', 'b']))  # 排序优先次序，a然后b
#    b  a
# 2 -3  0
# 0  4  0
# 3  2  1
# 1  7  1
```

Series和DataFrame中实现排名的方法是`rank`，默认方法是`'average'`在组中平均分配排名；还有`‘first’`方法，按照数值在数组中出现顺序排名：

```python
obj10 = pd.Series([7, -5, 7, 4, 2, 0, 4])
print(obj10)
# 0    7
# 1   -5
# 2    7
# 3    4
# 4    2
# 5    0
# 6    4
# dtype: int64

print(obj10.rank())
# 0    6.5
# 1    1.0
# 2    6.5
# 3    4.5
# 4    3.0
# 5    2.0
# 6    4.5
# dtype: float64

# 按值从小到大
# 排名，1：-5排在第一位
# 5：0第二位
# 4：2第三位
# 6：4，3：4相等，并列第四，也就是第四和第五，所以都是4.5位
# 以此类推
```

平级关系打破方法：

```python
print(obj10.rank(method='first'))
# 0    6.0
# 1    1.0
# 2    7.0
# 3    4.0
# 4    3.0
# 5    2.0
# 6    5.0
# dtype: float64

# 3和6对应的值都是4
# method=first就是先看到的优先
# 也就是3：4排第4位
# 6：4排在第5位

# Assign tie values the maximum rank in the group
print(obj10.rank(ascending=False, method='max'))
# 0    2.0
# 1    7.0
# 2    2.0
# 3    4.0
# 4    5.0
# 5    6.0
# 6    4.0
# dtype: float64

# 首先是反序，由大到小
# [7, -5, 7, 4, 2, 0, 4]
# 0：7和2：7，最大，在第一位和第二位
# method=max就是取最大的数
# 也就是两个的排名都是2
```


```python
frame11 = pd.DataFrame({'b': [4.3, 7, -3, 2], 'a': [0, 1, 0, 1],
                      'c': [-2, 5, 8, -2.5]})
print(frame11)
#      b  a    c
# 0  4.3  0 -2.0
# 1  7.0  1  5.0
# 2 -3.0  0  8.0
# 3  2.0  1 -2.5

print(frame11.rank(axis='columns'))
#      b    a    c
# 0  3.0  2.0  1.0
# 1  3.0  1.0  2.0
# 2  1.0  2.0  3.0
# 3  3.0  2.0  1.0

# 明显是abc对应的三个数排名
```

**表 5-6：排名优先级**

| 方法    | 描述                       |
| ------- | -------------------------- |
| average | 默认是这个，那就都是4.5    |
| min     | 都是第四                   |
| max     | 都是第五                   |
| first   | 先来的第四，后来的第五     |
| dense   | 类似于min，不过组间排名加1 |

加入两个并列第四

### 5.2.8 含有重复标签的轴索引

>   Axis Indexes with Duplicate Labels 

我们遇到以及常用的对象索引都是唯一的，但这并不是强制性的。下面我们来看数组具有重复索引的情况：


```python
## 如果索引有重复的，比如，两个a，两个b
import pandas as pd
import numpy as np
obj = pd.Series(range(5), index=['a', 'a', 'b', 'b', 'c'])
print(obj)
# a    0
# a    1
# b    2
# b    3
# c    4
# dtype: int64

# 判断索引元素是否是唯一的
print(obj.index.is_unique)
# False

# 使用索引，什么效果
print(obj['a'])
# a    0
# a    1
# dtype: int64

print(obj['c'])
4
```


```python
# 如果是4*4的看一哈
df = pd.DataFrame(np.random.randn(4, 3), index=['a', 'a', 'b', 'b'])
print(df)
#           0         1         2
# a -1.409549  0.585280 -0.206055
# a -1.069996 -1.156485 -0.324945
# b  1.064709 -1.449141 -0.213063
# b  1.066568 -0.761694  0.646288

print(df.loc['b'])
#           0         1         2
# b  1.064709 -1.449141 -0.213063
# b  1.066568 -0.761694  0.646288
```

当标签重复时，检索该标签会出现所有符合条件的值（会返回一个序列），而单个条目返回的是标量值。

DataFrame中同理。

## 5.3 描述性统计的概述与计算

>Summarizing and Computing Descriptive Statistics 

Pandas对象装配了一个常用数学、统计学方法的集合。与Numpy相比，它内建了处理缺失值的功能。以一个小型DataFrame为例：

主要是进行统计运算时，对缺失值的处理


```python
# 建立一个有缺失值的表
df = pd.DataFrame([[1.4, np.nan], [7.1, -4.5],
                   [np.nan, np.nan], [0.75, -1.3]],
                  index=['a', 'b', 'c', 'd'],
                  columns=['one', 'two'])
print(df)
#     one  two
# a  1.40  NaN
# b  7.10 -4.5
# c   NaN  NaN
# d  0.75 -1.3

# 感觉NaN被默认为0
print(df.sum())  # 默认计算axis=0方向，竖着加
# one    9.25
# two   -5.80
# dtype: float64

print(df.sum(axis='columns')) # 计算axis=1方向，横着加，c行印证了NaN默认为零
# a    1.40
# b    2.60
# c    0.00
# d   -0.55
# dtype: float64
```

通过上述加和我们发现，缺失值被自动排除了，没有对操作造成影响。有些时候我们需要考虑到缺失值，可以使用skipna实现不排除缺失值：

```python
# 如果想保留缺失值，则加入参数skipna=False
print(df.mean(axis='columns', skipna=False))
# a      NaN
# b    1.300
# c      NaN
# d   -0.275
# dtype: float64
```

**表5-7：归约方法可选参数**

| 方法   | 描述                                               |
| ------ | -------------------------------------------------- |
| axis   | 归约轴                                             |
| skipna | 排除缺失值，默认为True                             |
| level  | 如果轴是多层索引MultiIndex，该参数可以缩减分组层级 |

还有许多方法，例如 `idmin\inmax`统计的是最值的索引，`cumsum`可得到数组累积和，...

```python
## 最大值的索引值，one这一列，最大值7.10，对应索引为b
print(df.idxmax())
# one    b
# two    d
# dtype: object


# 积累型计算
# 1.4
# 1.4+7.1=8.5
# NaN
# 1.4+7.1+0.75=9.25
print(df.cumsum())
#     one  two
# a  1.40  NaN
# b  8.50 -4.5
# c   NaN  NaN
# d  9.25 -5.8
```

需要特别介绍的是describe方法，它可以计算各列的汇总统计集合，而且对于数值型数据和非数值型数据汇总统计结果不同：

```python
# 一次性产生多个汇总统计
print(df.describe())
#             one       two
# count  3.000000  2.000000
# mean   3.083333 -2.900000
# std    3.493685  2.262742
# min    0.750000 -4.500000
# 25%    1.075000 -3.700000
# 50%    1.400000 -2.900000
# 75%    4.250000 -2.100000
# max    7.100000 -1.300000


# 对于非数值型数据，产生另一种汇总统计
obj = pd.Series(['a', 'a', 'b', 'c'] * 4)
print(obj)
# 0     a
# 1     a
# 2     b
# 3     c
# 4     a
# 5     a
# 6     b
# 7     c
# 8     a
# 9     a
# 10    b
# 11    c
# 12    a
# 13    a
# 14    b
# 15    c
# dtype: object


print(obj.describe())
# count     16
# unique     3
# top        a
# freq       8   # 表明重复最多元素的重复次数
# dtype: object

# 共16个
# 元素abc三个
# top最多的是a
# frequency，重复的次数是8
```



**表5-8：描述性统计和汇总统计**

| 方法           | 描述                                         |
| -------------- | -------------------------------------------- |
| count          | 非NA值得个数                                 |
| describe       | 计算Series或DataFrame各列的汇总统计集合      |
| min, max       | 计算最小值、最大值                           |
| argmin，argmax | 分别计算最小值、最大值所在的索引位置（整数） |
| idxmin，idxmax | 分别计算最小值或最大值所在的索引标签         |
| quantile       | 计算样本的从0到1间的分位数                   |
| sum            | 加和                                         |
| mean           | 均值                                         |
| median         | 中位数                                       |
| mad            | 平均值和平均绝对偏差                         |
| prod           | 所有值得积                                   |
| var            | 值的样本方差                                 |
| std            | 值的样本标准差                               |
| skew           | 样本偏度值                                   |
| kurt           | 样本鞥度值                                   |
| cumsum         | 累计值                                       |
| cummin，cummax | 累计最小值或最大值                           |
| cumprod        | 值的累计积                                   |
| diff           | 计算第一个算术差值                           |

### 5.3.1  相关性和协方差

>   Correlation and Covariance

相关性 corr()  ,协方差 cov()

库：pandas-datareader

数据：Yahoo！Finance上获取包含股价交易量的DataFrame

可以通过conda或pip安装


```python
# conda install pandas-datareader
!pip install pandas-datareader
```


亲测好用


```python
# 使用pandas_datareader下载一些数据（需要科学上网，才能下载数据）
import pandas_datareader.data as web
all_data = {ticker: web.get_data_yahoo(ticker)
            for ticker in ['AAPL', 'IBM', 'MSFT', 'GOOG']}
# 下载股票代号ticker为这几个的数据
```

或者用作者已经准备好的数据


```python
# 书上准备好的数据集
# price = pd.read_pickle('examples/yahoo_price.pkl')
# volume = pd.read_pickle('examples/yahoo_volume.pkl')
```

查看一下这个数据

```python
print(all_data)
# {'AAPL':                   High         Low  ...      Volume   Adj Close
# Date                                ...                        
# 2015-05-11  127.559998  125.629997  ...  42035800.0  116.732750
# 2015-05-12  126.879997  124.820000  ...  48160000.0  116.316917
# 2015-05-13  127.190002  125.870003  ...  34694200.0  116.446297
# 2015-05-14  128.949997  127.160004  ...  45203500.0  119.163139
# 2015-05-15  129.490005  128.210007  ...  38208000.0  118.996811
# ...                ...         ...  ...         ...         ...
# 2020-05-01  299.000000  285.850006  ...  60154200.0  289.070007
# 2020-05-04  293.690002  286.320007  ...  33392000.0  293.160004
# 2020-05-05  301.000000  294.459991  ...  36937800.0  297.559998
# 2020-05-06  303.239990  298.869995  ...  35583400.0  300.630005
# 2020-05-07  305.170013  301.970001  ...  28690300.0  303.739990

# [1258 rows x 6 columns], 
# 'IBM':                   High         Low  ...     Volume   Adj  Close
# Date                                ...                       
# 2015-05-11  172.990005  170.860001  ...  2661000.0  137.684235
# 2015-05-12  171.490005  168.839996  ...  2962400.0  137.225616
# 2015-05-13  172.740005  170.750000  ...  2457500.0  138.617584
# 2015-05-14  174.399994  173.220001  ...  2439100.0  140.041748
# 2015-05-15  174.410004  172.600006  ...  2916600.0  139.406113
# ...                ...         ...  ...        ...         ...
# 2020-05-01  123.470001  121.389999  ...  4925500.0  120.257210
# 2020-05-04  121.970001  119.389999  ...  4017900.0  120.069717
# 2020-05-05  124.320000  122.470001  ...  3899400.0  120.957809
# 2020-05-06  124.050003  122.410004  ...  3864600.0  121.540001
# 2020-05-07  123.260002  120.849998  ...  4412600.0  121.230003

# [1258 rows x 6 columns], 
# 'MSFT':                   High         Low  ...      Volume   Adj Close
# Date                                ...                        
# 2015-05-11   47.910000   47.369999  ...  24609400.0   42.712799
# 2015-05-12   47.680000   46.419998  ...  29928300.0   42.694778
# 2015-05-13   48.320000   47.570000  ...  34184600.0   42.947243
# 2015-05-14   48.820000   48.029999  ...  32980900.0   43.930080
# 2015-05-15   48.910000   48.049999  ...  28642700.0   43.551365
# ...                ...         ...  ...         ...         ...
# 2020-05-01  178.639999  174.009995  ...  39370500.0  174.570007
# 2020-05-04  179.000000  173.800003  ...  30372900.0  178.839996
# 2020-05-05  183.649994  179.899994  ...  36839200.0  180.759995
# 2020-05-06  184.199997  181.630005  ...  32139300.0  182.539993
# 2020-05-07  184.550003  182.580002  ...  28255500.0  183.600006

# [1258 rows x 6 columns], 
# 'GOOG':                    High          Low  ...   Volume    Adj Close
# Date                                  ...                      
# 2015-05-11   541.979980   535.400024  ...   905300   535.700012
# 2015-05-12   533.208984   525.260010  ...  1634200   529.039978
# 2015-05-13   534.322021   528.655029  ...  1252300   529.619995
# 2015-05-14   539.000000   532.409973  ...  1403900   538.400024
# 2015-05-15   539.273987   530.380005  ...  1971300   533.849976
# ...                 ...          ...  ...      ...          ...
# 2020-05-01  1352.069946  1311.000000  ...  2072500  1320.609985
# 2020-05-04  1327.660034  1299.000000  ...  1504000  1326.800049
# 2020-05-05  1373.939941  1337.459961  ...  1651500  1351.109985
# 2020-05-06  1371.119995  1347.290039  ...  1215400  1347.300049
# 2020-05-07  1377.599976  1355.270020  ...  1397600  1372.560059

# [1258 rows x 6 columns]}
```

High当日最高价

Low当日最低价

Open当日开盘价

Close当日收盘价

Volume当日成交量

Adj Close 前复权收盘价，为了更准确反应股票的价值，针对分红对影响进行的调整

[知乎：股价的前复权和后复权是什么鬼](https://zhuanlan.zhihu.com/p/49187942)

这个数据本身是一个字典


```python
print(type(all_data))
#<class 'dict'>
```


字典的key值为股票代号，value值是一个DataFrame


```python
print(type(all_data['AAPL']))
# <class 'pandas.core.frame.DataFrame'>
```

```python
# 取出DataFrame里面的一个列Series
print(all_data['AAPL']['Adj Close'])
# Date
# 2009-12-31     26.193771
# 2010-01-04     26.601469
# 2010-01-05     26.647457
# 2010-01-06     26.223597
# 2010-01-07     26.175119
# 2010-01-08     26.349140
# 2010-01-11     26.116703
# 2010-01-12     25.819624
# 2010-01-13     26.183823
# 2010-01-14     26.032179
#                  ...    
# 2019-11-05    256.360352
# 2019-11-06    256.470001
# 2019-11-07    259.429993
# 2019-11-08    260.140015
# 2019-11-11    262.200012
# 2019-11-12    261.959991
# 2019-11-13    264.470001
# 2019-11-14    262.640015
# 2019-11-15    265.760010
# 2019-11-18    265.233398
# Name: Adj Close, Length: 2488, dtype: float64
```

取出四只股票代码和价格，通过字典生成一个DataFrame，其中字典的`key`值来自与`all_data`里面的`key`值，`value`值是`all_data`里面DataFrame的一列，这里用的是`Adj Close`。


```python
# 取出价格生成一个DataFrame
price = pd.DataFrame({ticker: data['Adj Close'] 
                      for ticker, data in all_data.items()})
print(price)
#                   AAPL         IBM        MSFT         GOOG
# Date                                                       
# 2015-05-11  116.732750  137.684235   42.712799   535.700012
# 2015-05-12  116.316917  137.225616   42.694778   529.039978
# 2015-05-13  116.446297  138.617584   42.947243   529.619995
# 2015-05-14  119.163139  140.041748   43.930080   538.400024
# 2015-05-15  118.996811  139.406113   43.551365   533.849976
# ...                ...         ...         ...          ...
# 2020-05-01  289.070007  120.257210  174.570007  1320.609985
# 2020-05-04  293.160004  120.069717  178.839996  1326.800049
# 2020-05-05  297.559998  120.957809  180.759995  1351.109985
# 2020-05-06  300.630005  121.540001  182.539993  1347.300049
# 2020-05-07  303.739990  121.230003  183.600006  1372.560059

# [1258 rows x 4 columns]
```

取出四只股票当日成交量


```python
volume = pd.DataFrame({ticker: data['Volume'] for ticker,
                       data in all_data.items()})
print(volume)
#                   AAPL        IBM        MSFT     GOOG
# Date                                                  
# 2015-05-11  42035800.0  2661000.0  24609400.0   905300
# 2015-05-12  48160000.0  2962400.0  29928300.0  1634200
# 2015-05-13  34694200.0  2457500.0  34184600.0  1252300
# 2015-05-14  45203500.0  2439100.0  32980900.0  1403900
# 2015-05-15  38208000.0  2916600.0  28642700.0  1971300
# ...                ...        ...         ...      ...
# 2020-05-01  60154200.0  4925500.0  39370500.0  2072500
# 2020-05-04  33392000.0  4017900.0  30372900.0  1504000
# 2020-05-05  36937800.0  3899400.0  36839200.0  1651500
# 2020-05-06  35583400.0  3864600.0  32139300.0  1215400
# 2020-05-07  28690300.0  4412600.0  28255500.0  1397600

# [1258 rows x 4 columns]
```

计算股价变动百分比


```python
returns = price.pct_change()
print(returns.tail())
#                 AAPL       IBM      MSFT      GOOG
# Date                                              
# 2020-05-01 -0.016099 -0.029388 -0.025891 -0.020798
# 2020-05-04  0.014149 -0.001559  0.024460  0.004687
# 2020-05-05  0.015009  0.007396  0.010736  0.018322
# 2020-05-06  0.010317  0.004813  0.009847 -0.002820
# 2020-05-07  0.010345 -0.002551  0.005807  0.018749
```

算了两列的相关性


```python
print(returns['MSFT'].corr(returns['IBM']))
# 0.4906562644244645
```

算两列的协方差


```python
print(returns['MSFT'].cov(returns['IBM']))
# 8.738668052033956e-05
```

也可以写成下面这种形式


```python
print(returns.MSFT.corr(returns.IBM))
# 0.4906562644244645
```

返回四只股票价格变动之间的相关性矩阵


```python
print(returns.corr())
#           AAPL       IBM      MSFT      GOOG
# AAPL  1.000000  0.532317  0.710846  0.642081
# IBM   0.532317  1.000000  0.600171  0.529213
# MSFT  0.710846  0.600171  1.000000  0.750424
# GOOG  0.642081  0.529213  0.750424  1.000000
```

返回四只股票的协方差矩阵


```python
print(returns.cov())
#           AAPL       IBM      MSFT      GOOG
# AAPL  0.000328  0.000151  0.000221  0.000199
# IBM   0.000151  0.000245  0.000162  0.000142
# MSFT  0.000221  0.000162  0.000296  0.000221
# GOOG  0.000199  0.000142  0.000221  0.000293
```

如果想要每只股票与IBM股价的相关性


```python
print(returns.corrwith(returns.IBM))
# AAPL    0.385371
# IBM     1.000000
# MSFT    0.490656
# GOOG    0.406842
# dtype: float64
```

成交量与价格变化百分比的相关性


```python
print(returns.corrwith(volume))
# AAPL   -0.065362
# IBM    -0.157569
# MSFT   -0.090185
# GOOG   -0.019751
# dtype: float64
```

### 5.3.2 唯一值、计数和成员属性

>Unique Values, Value Counts, and Membership

`unique`函数给出序列中的唯一值

`value_counts`计算Series包含的值的个数

`isin`执行向量化的成员属性检查


```python
import pandas as pd

obj = pd.Series(['c', 'a', 'd', 'a', 'a', 'b', 'b', 'c', 'c'])
print(obj)
# 0    c
# 1    a
# 2    d
# 3    a
# 4    a
# 5    b
# 6    b
# 7    c
# 8    c
# dtype: object

# 去掉Series里面的重复值，每个元素留一个
uniques = obj.unique()
print(uniques)
# array(['c', 'a', 'd', 'b'], dtype=object)

# 计算包含值的个数，默认按数量由多到少排序
print(obj.value_counts())
# a    3
# c    3
# b    2
# d    1
# dtype: int64

# 这个是没排序还是怎么着
print(pd.value_counts(obj.values, sort=True))
# a    3
# c    3
# b    2
# d    1
# dtype: int64
```

`isin`执行向量化检查


```python
# 判断，判断值是不是在这个位置
mask = obj.isin(['b', 'c'])
print(mask) ## 返回布尔值
# 0     True
# 1    False
# 2    False
# 3    False
# 4    False
# 5     True
# 6     True
# 7     True
# 8     True
# dtype: bool

print(obj[mask]) # 使用布尔值索引
# 0    c
# 5    b
# 6    b
# 7    c
# 8    c
# dtype: object
```

`Index.get_indexer`方法


```python
to_match = pd.Series(['c', 'a', 'b', 'b', 'c', 'a','d'])
print(to_match)
# 0    c
# 1    a
# 2    b
# 3    b
# 4    c
# 5    a
# 6    d
# dtype: object

unique_vals = pd.Series(['c', 'b', 'a'])
print(unique_vals)
# 0    c
# 1    b
# 2    a
# dtype: object

# Index.get_indexer方法，可以提供一个索引数组
# 将可能非唯一值的数组转换为另一个唯一值数组？？？
print(pd.Index(unique_vals).get_indexer(to_match))
# array([ 0,  2,  1,  1,  0,  2, -1], dtype=int64)
```

举例说明下`Index.get_indexer`方法


```python
l = list(unique_vals)
for x in to_match:
    if x in l:
        print(l.index(x))
    else:
        print(-1)
        
# 0
# 2
# 1
# 1
# 0
# 2
# -1
```

**0：**`to_match[0]`为`c` 在`unique_vals索引`为0 

**2：**`to_match[1]`为`a` 在`unique_vals索引`为2 

**1：**`to_match[2]`为`b` 在`unique_vals索引`为1 

**1：**`to_match[3]`为`b` 在`unique_vals索引`为1 

**0：**`to_match[4]`为`c` 在`unique_vals索引`为0 

**2：**`to_match[5]`为`a` 在`unique_vals`索引为2 

要是元素不存在，比如`unique_vals = pd.Series(['e'])`，索引就为-1

**表 5-9：唯一值、计数和集合成员属性方法**

| 方法        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| isin        | 值在不在，返回布尔值                                         |
| match       | 计算数组每个值的索引，形成一个唯一值数组                     |
| unique      | 计算Series值中唯一值数组，按照观察顺序返回                   |
| valu_counts | 返回一个Series，索引是唯一值序列，值是计数个数，按照个数降序排序 |

```python
# 计算多个列的直方图
data = pd.DataFrame({'Qu1': [1, 3, 4, 3, 4],
                     'Qu2': [2, 3, 1, 2, 3],
                     'Qu3': [1, 5, 2, 4, 4]})
print(data)
#    Qu1  Qu2  Qu3
# 0    1    2    1
# 1    3    3    5
# 2    4    1    2
# 3    3    2    4
# 4    4    3    4


# 行标签，12345是各种不同的值
# 对应的值则是在某列出现的次数
# 如，1在Qu1里面出现了了一次
result = data.apply(pd.value_counts).fillna(0)
print(result)
#    Qu1  Qu2  Qu3
# 1  1.0  1.0  1.0
# 2  0.0  2.0  1.0
# 3  2.0  2.0  0.0
# 4  2.0  0.0  2.0
# 5  0.0  0.0  1.0
```
