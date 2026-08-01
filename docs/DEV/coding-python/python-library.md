# Python Library

## Pandas

[User Guide](https://pandas.pydata.org/docs/user_guide/index.html)

[菜鸟教程｜Pandas 教程](https://www.runoob.com/pandas/pandas-tutorial.html)

Pandas 是 Python 语言的一个扩展程序库，用于数据分析。

### 应用

Pandas 可以从各种文件格式比如 CSV、JSON、SQL、Microsoft Excel 导入数据。

Pandas 可以对各种数据进行运算操作，比如归并、再成形、选择，还有数据清洗和数据加工特征。

### CSV

读取:

```python
df = pd.read_csv('nba.csv')
```

之后处理某列就用 `df['日期']`

输出:

```python
print(df.to_string())	# to_string() 用于返回 DataFrame 类型的数据，输出全部内容
print(df)	# 输出数据的前面 5 行和末尾 5 行，中间部分以 ... 代替
```

### 数据清洗

很多数据集存在数据缺失、数据格式错误、错误数据或重复数据的情况:

![截屏2024-08-30 11.35.43](./img_library/%E6%88%AA%E5%B1%8F2024-08-30%2011.35.43.png)

包含了四种空数据: n/a, NA, —, na (自己甄别)

判断空值:

```python
df = pd.read_csv('property-data.csv')
print (df['NUM_BEDROOMS'])
print (df['NUM_BEDROOMS'].isnull())
```

- `isnull()` 判断数据是否为空，"空"可以自己定义

```python
missing_values = ["n/a", "na", "--"]
df = pd.read_csv('property-data.csv', na_values = missing_values)
print (df['NUM_BEDROOMS'].isnull())
```

清洗空值:

```python
# 删除包含空字段的行/列
DataFrame.dropna(axis=0, how='any', thresh=None, subset=None, inplace=False)
```

- axis：默认为 0，表示逢空值剔除整行，如果设置参数 `axis＝1` 表示逢空值去掉整列。
- how：默认为 **'any'** 如果一行（或一列）里任何一个数据有出现 NA 就去掉整行，如果设置 **how='all'** 一行（或列）都是 NA 才去掉这整行。
- thresh：设置需要多少非空值的数据才可以保留下来的。
- subset：设置想要检查的列。如果是多个列，可以使用列名的 list 作为参数。
- inplace：如果设置 True，将计算得到的值直接覆盖之前的值并返回 None，修改的是源数据。

```python
new_df = df.dropna()
print(new_df.to_string())
```

替换单元格:

- `mean()`、`median()` 和 `mode()` 方法计算列的均值、中位数值和众数

```python
# 计算列的均值并替换空单元格
x = df["ST_NUM"].mean()
df["ST_NUM"].fillna(x, inplace = True)
```

删除重复数据:

```python
df.drop_duplicates(inplace = True)
```

清洗格式错误数据:

```python
df['日期'] = pd.to_datetime(df['日期'])
```

从日期中取出月份:

```python
df['月份'] = df['日期'].dt.month
```

## Matplotlib Pyplot

[菜鸟教程｜Matplotlib Pyplot](https://www.runoob.com/matplotlib/matplotlib-pyplot.html)

Pyplot 是 Matplotlib 的子库，提供了和 MATLAB 类似的绘图 API。

### 导入库

```python
import matplotlib.pyplot as plt
```

### 函数

- `plot()`：绘制线图和散点图
- `scatter()`：绘制散点图
- `bar()`：绘制垂直条形图和水平条形图
- `hist()`：绘制直方图
- `pie()`：绘制饼图
- `imshow()`：绘制图像
- `subplots()`：创建子图

### 绘图函数

```python
xpoints = np.array([0, 6])
ypoints = np.array([0, 100])

# 要绘制坐标 (0, 6) 到 (0, 100) 的线，我们就需要传递两个数组给 plot 函数
plt.plot(xpoints, ypoints)	
plt.show()
```

![截屏2024-08-30 10.25.39](./img_library/%E6%88%AA%E5%B1%8F2024-08-30%2010.25.39.png)

`plot()`：绘制点和线

```python
# 画单条线
plot([x], y, [fmt], *, data=None, **kwargs)
# 画多条线
plot([x], y, [fmt], [x2], y2, [fmt2], ..., **kwargs)
```

```python
>>> plot(x, y)        # 创建 y 中数据与 x 中对应值的二维线图，使用默认样式
>>> plot(x, y, 'bo')  # 创建 y 中数据与 x 中对应值的二维点图，使用蓝色实心圈绘制
>>> plot(y)           # x 的值为 0..N-1
>>> plot(y, 'r+')     # 使用红色 + 号
```

### 轴标签和标题

```python
plt.title("RUNOOB TEST TITLE")
plt.xlabel("x - label")
plt.ylabel("y - label")		# 设置 x 轴和 y 轴的标签
```

### 中文字体

把字体文件放在执行的代码文件中

```python
zhfont1 = matplotlib.font_manager.FontProperties(fname="SourceHanSansSC-Bold.otf")
plt.title("菜鸟教程 - 测试", fontproperties=zhfont1) 
```

## NumPy

[菜鸟教程｜NumPy 教程](https://www.runoob.com/numpy/numpy-tutorial.html)

NumPy(Numerical Python) 是 Python 语言的一个扩展程序库，支持大量的维度数组与矩阵运算，此外也针对数组运算提供大量的数学函数库。

### 导入库

```python
import numpy as np
```

### NumPy Ndarray 对象

N 维数组对象 ndarray，它是一系列同类型数据的集合，以 0 下标为开始进行集合中元素的索引

Eg1.

```python
a = np.array([1,2,3])  
print (a)
```

输出 `[1 2 3]`

Eg2.

```python
# 多于一个维度  
import numpy as np 
a = np.array([[1,  2],  [3,  4]])  
print (a)
```

输出:

```bash
[[1 2]
 [3 4]]
```

### NumPy 从数值范围创建数组

`numpy.arange`:

numpy 包中的使用 arange 函数创建数值范围并返回 ndarray 对象，函数格式如下：

```python
numpy.arange(start, stop, step, dtype)
```

### NumPy 数学函数

NumPy 提供了标准的三角函数：`sin()`、`cos()`、`tan()` ，计算数组中角度的三角函数值 (弧度制)

## Sympy

