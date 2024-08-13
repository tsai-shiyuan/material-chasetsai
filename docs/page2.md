# Page 2

## Another heading

#### 函数参数

Default: 

```python

```

#### 字符串格式化

```py linenums="1" title="hello" hl_lines="2"
print(*objects, sep=' ', end='\n', file=sys.stdout, flush=False)
print(f"hello, {name}")
```

字符串前加上 `f` 


interactive mode

注：单引号和双引号在python中有同样的作用

## functions

CLI：command line interface

GUI：graphical user interface

### print()

`print("hello, " + name)`  连接 `+`

`print("hello,",name)` 多个argument(参数) 输出用 `,` 隔开，会自动添加空格

`print("hello, world")` 会自动换行（更好的理解：光标移动到下一行）

`print('hello, "friend"')` 引号保持一致可以输出，输出 `hello, "friend"`

`print("hello, \"friend\"")` 转义也可以输出引号

#### 函数参数

Default: 

```python
print(*objects, sep=' ', end='\n', file=sys.stdout, flush=False)
```

#### 字符串格式化

```python
print(f"hello, {name}")
```

字符串前加上 `f` 

## variables

**store the returned values and do something**

### str

`name = name.strip()`

返回移除字符串头尾指定的字符生成的新字符串（默认为空格或换行符）

`name = name.capitalize()`

Capitalize the very first letter

`name = name.title()`

Capitalize the first letter of each word

`name = name.strip().title()`

和分开写的效果一样

`first, last = name.split()`

按序返回，并能同时赋给左边的变量

**只有函数的返回值是`str` 时，才能用 `.func()` 的方式调用**

### int

#### 范围

在python中，int类型可以无限大

#### 加法

```python
x = input("What's x? ")
y = input("What's y? ")
z = x + y
print(z)
```

`+` 会把 `x` 字符串和 `y` 字符串连在一起，那何时做整数加法？修改：

```python
z = int(x) + int(y)
```

由此观之，`int` 还是一个函数，把输入的字符串 `x` 变为可以运算的整数

```python
x = int(input("What's x? "))
y = int(input("What's y? "))
print(x + y)
```

**内层函数的返回值(return value)会作为外层函数的参数(argument)**

所以还可以这样写（但这样会增加阅读难度和犯错概率）：

```python
print(int(input("What's x? ")) + int(input("What's y? ")))
```

### float

#### 四舍五入近似

```python
round(number[, ndigits])
```

保留成最近的整数，ndigits表示近似到几位

`[]` 括起来表示括号里面的内容是可以选择写或不写的

#### 格式化输出：小数位数

```python
print(f"{z:.2f}")
```

#### 格式化输出：每三位一个逗号

想输出 `1,000,000` ？

 ```python
print(f"{z:,}")
 ```

## 自定义函数

### def

```python
def hello():		#()为空表示不接收参数
    print("hello")		#只要有四个缩进就是定义的函数的内容
```

一个sayhello程序

```python
def hello(to="world"):	#给参数一个默认值
    print("hello,",to)

hello()		#没有参数argument，就会代入默认的world
name = input("What's your name? ").strip().title()
hello(name)
```

### 规范

先define，再call

```python
def main():
    name = input("What's your name? ").strip().title()
    hello(name)

def hello(to="world"):
    print("hello,",to)

main()
```

### scope作用范围

变量在定义它的函数中起作用（和C差不多）

### return返回值

```python
def main():
    x= int(input("What's x? "))
    print("x squared is",square(x))

def square(n):
    return n * n

main()
```

 #### ps. 乘方运算

例如：计算 `n^2`

`pow(n, 2)` 或 `n**2`

## comments

note for myself

* intention
* Posedocode: outline the program in advance(to-do list)

单行注释：

```python
# ...comment
```

多行注释：

```python
"""
...comment
"""
```

