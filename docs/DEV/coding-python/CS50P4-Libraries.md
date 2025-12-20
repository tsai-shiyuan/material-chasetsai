# CS50P: 4. Libraries

## libraries, modules

**libraries** are bits of code written by you or others we can use in our program

Python allows us to share functions or features with others as "**modules**"

## random

[python: random](https://docs.python.org/3/library/random.html)

`random` is a library that comes with Python that we could import into our own project

we need keyword `import`

### import

#### 作用

import the contents of the functions from some module

#### 使用random模块中的函数

```python
import random
coin = random.choice(["heads", "tails"])
print(coin)
```

`random` is the module we are importing. Inside that module, there is the `choice` function -- `random.choice(seq)`

参数：list

返回值：list中的随机一个元素 

### from

from: allows us to be very **specific** about what we’d like to import

import: bringing the entire contents of the functions of a module

```python
from random import choice
coin = choice(["heads", "tails"])
```

注：使用时直接用函数名 (因为只 import 了一部分)

### functions

#### random.randint(a, b)

return random int in [a,b] 区间

#### random.shuffle(x)

shuffle a list into a random order

注意：没有返回，而是直接作用于list

## statistics

[python: statistics](https://docs.python.org/3/library/statistics.html)

### mean function

```python
print(statistics.mean([100, 90]))
```

接收列表，返回平均值

## Command-Line Arguments

take input from the command-line命令行参数

### sys

a module that allows us to take arguments at the command line

[python: sys](https://docs.python.org/3/library/sys.html)

#### argv

a function within the sys module

```python
import sys
print("hello, my name is", sys.argv[1])
```

* sys.argv是一个list

* 命令行输入 `python3 sys.py chase`
* sys.argv[0] 是 `sys.py`
* `"chase tsai"` 视作一个命令（加了引号）

#### IndexError

如果使用命令 `python3 sys.py`，返回 `IndexError: list index out of range`

#### 改进

way 1

```python
try:
    print("hello, my name is", sys.argv[1])
except IndexError:
    print("Too few arguments")
```

way 2

```python
if len(sys.argv) < 2:
    print("Too few arguments")
elif len(sys.argv) > 2:
    print("Too many arguments")
else:
    print("hello, my name is", sys.argv[1])
```

#### sys.exit()

exit the program if an error was introduced by the user

```python
if len(sys.argv) < 2:
    sys.exit("Too few arguments")
```

常常用来**结束整个程序**

## slice

To take a slice of a list means to take a subset of it

```python
for arg in sys.argv[1:]:
    print("hello, my name is", arg)
```

* `:` 必须有 

* `sys.argv[1:-2]` 表示取左一到右二之间的子序列

## packages

One of the reasons Python is so popular is that there are numerous powerful **third-party libraries** that add functionality. We call these third-party libraries, implemented as a **folder**, "**packages**".

find other third-party packages at [PyPI](https://pypi.org/)

learn more on PyPI’s entry for [cowsay](https://pypi.org/project/cowsay/): a well-known package that allows a cow to talk to the user

### pip

install packages quickly onto your system

## APIs

APIs or “application program interfaces” allow you to connect to the code of others

### requests

[python: requests](https://docs.python-requests.org/).

`requests` is a package that allows our program to behave as a web browser would

```python
import requests
import sys

if len(sys.argv) != 2:
    sys.exit()
    
"""
looking for a song, with a limit of one result, that relates to the term called weezer
"""
response = requests.get("https://itunes.apple.com/search?entity=song&limit=1&term=" + sys.argv[1])	
print(response.json())
```

**第10行是把python连接到浏览器，就像我们在Safari中输入urls，按enter键**

Apple documentation about this API: what is returned is a JSON file. Running `python3 itunes.py weezer`, you will see the JSON file returned by Apple. 

![截屏2024-07-14 22.37.02](./image/%E6%88%AA%E5%B1%8F2024-07-14%2022.37.02.png)

However, the JSON response is converted by Python into a dictionary!!!

![截屏2024-07-14 22.40.15](./image/%E6%88%AA%E5%B1%8F2024-07-14%2022.40.15.png)

## JSON

### 库

python的一个库

功能：manipulate JSON data, nicely printing on the screen

 [python: JSON](https://docs.python.org/3/library/json.html)

### 应用样例

修改上例代码

```python
import requests
import sys
import json

if len(sys.argv) != 2:
    sys.exit()
response = requests.get("https://itunes.apple.com/search?entity=song&limit=1&term=" + sys.argv[1])
print(json.dumps(response.json(), indent=2))
```

![截屏2024-07-14 22.45.51](./image/%E6%88%AA%E5%B1%8F2024-07-14%2022.45.51.png)

进一步观察，删除 `&limit=1` 并插入代码：

```python
o = response.json()
for result in o["results"]:
    print(result["trackName"])
```

会得到所有的trackName：

![截屏2024-07-14 23.09.59](./image/%E6%88%AA%E5%B1%8F2024-07-14%2023.09.59.png)

## Making Our Own Libraries

### sayings.py

```python
def main():
    hello("world")
    goodbye("world")
    
def hello(name):
    print(f"hello, {name}")

def goodbye(name):
    print(f"goodbye, {name}")

main()
```

### say.py

```python
import sys
from sayings import hello
if len(sys.argv) == 2:
    hello(sys.argv[1])
```

sayings.py是我们自己写的module，用 `from sayings import hello` ，python会知道是在saying.py中找hello函数，然后执行**它以下的所有代码**

### 运行

命令行 `python3 say.py chase`：

```
hello, world
goodbye, world
hello, chase
```

### \__name__

只想输出 `hello, xxx` ，解决方法：

```python
if __name__ == "__main__":
    main()
```

`__name__` 是特殊变量，python将它自动设置成 `"__main__"` 当在command line运行该程序，本例中即 `python3 sayings.py` 。而我们在command line只运行了say.py程序，sayings.py程序只是import的，所以 `__name__` 没有自动初始化，不会被执行



