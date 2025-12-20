# CS50P: 9. Et Cetera

## set

python’s documentation of [set](https://docs.python.org/3/library/stdtypes.html#set)

- 类型

- 数学上的集合，没有重复元素

Q: 统计有多少个不同名house

### way1--list

```python
students = [
    {"name": "Hermione", "house": "Gryffindor"},
    {"name": "Harry", "house": "Gryffindor"},
    {"name": "Ron", "house": "Gryffindor"},
    {"name": "Draco", "house": "Slytherin"},
    {"name": "Padma", "house": "Ravenclaw"},
]
houses = []
for student in students:
    if student["house"] not in houses:
        houses.append(student["house"])
for house in sorted(houses):
    print(house)
```

line 8 等价于 `houses = list()`

### way2--set

set 自动删除重复元素

 ```python
 houses = set()
 for student in students:
     houses.add(student["house"])
 ```

## Global Variables 全局变量

### access

所有的函数都可以访问的变量，同C，对应 local variable

```python
balance = 0
def main():
    print("Balance:", balance)
    deposit(100)
    withdraw(50)
    print("Balance:", balance)
def deposit(n):
    balance += n
def withdraw(n):
    balance -= n
if __name__ == "__main__":
    main()
```

报错 `UnboundLocalError: cannot access local variable 'balance' where it is not associated with a value`

**global variable 可以访问，不能修改**

line 8 的 `balance` 变量会被认为是 local variable

### interact--global

```python
def deposit(n):
    global balance
    balance += n
def withdraw(n):
    global balance
    balance -= n
```

函数内加上 `global xxx` 指示 `xxx` 是全局变量

### object-oriented programming

```python
class Account:
    def __init__(self):
        self._balance = 0    # instance variable

    @property
    def balance(self):
        return self._balance
    
    def deposit(self, n):
        self._balance += n
    
    def withdraw(self, n):
        self._balance -= n

def main():
    account = Account()
    print("Balance:", account.balance)
    account.deposit(100)
    account.withdraw(50)
    print("Balance:", account.balance)

if __name__ == "__main__":
    main()
```

加 `_` 可以提醒自己其他地方不能 touch 这个 class 中的 function，比如 main 中不能有 `account.balance = 1000`

class 中的 `balance` 可以在其中的函数中修改，因为有 `self` 参数

## Constants

常量不可修改，通常大写并放在最前，但是python不知道是个常量，主要提醒自己不要修改大写变量

```python
MEOWS = 3
for _ in range(MEOWS):
    print("meow")
```

OOP:

```python
class Cat:
    MEOWS = 3   # class constant
    def meow(self):
        for _ in range(Cat.MEOWS):
            print("meow")
cat = Cat()     # instantiate a cat
cat.meow()      # let cat meow
```

## Type Hints

dynamic language，不用写变量的类型，python会自动识别

python’s documentation of [Type Hints](https://docs.python.org/3/library/typing.html)

### mypy

测试并保证变量使用合理

在终端运行 `mypy meow.py` ，会显示错误（程序员可以忽略错误），根据提示修改即可

 [mypy](https://mypy.readthedocs.io/)'s own documentation

### 变量类型

```python
def meow(n):
    for _ in range(n):
        print("meow")
number = input("Number: ")
meow(number)
```

报错：line 4 接收到的是字符

修改代码2.0：

```python
def meow(n: int):
```

`: int` 是 `type hint` ，表示 `n` 应该是 `int` 型变量。

修改代码3.0：

```python
def meow(n: int):
    for _ in range(n):
        print("meow")
number: int = int(input("Number: "))
meow(number)
```

### 函数返回值提示

思想：函数尽量要有返回值

```python
def meow(n: int) -> str:
    return "meow\n" * n
number: int = int(input("Number: "))
meows: str = meow(number)
print(meows, end="")
```

line 1: 是一个 `hint` ，提示我们函数默认返回类型为 `str` (可以是 `None` )

## Docstrings

python’s documentation of [docstrings](https://peps.python.org/pep-0257/)

左右三个引号指示本函数的作用

```python
def meow(n: int) -> str:
    """Meow n times."""
    return "meow\n" * n
```

给函数写说明文档

```python
def meow(n: int) -> str:
    """
    Meow n times.
    
    :param n: Number of times to meow
    :type n: int
    :raise TypeError: If n is not an int
    :return: A string of n meows, one per line
    :rtype: str
    """
    return "meow\n" * n
```

通过第三方库 [Sphinx ](https://www.sphinx-doc.org/en/master/index.html)可以生成 PDF 文档或网页

## Argparse

python’s documentation of [argparse](https://docs.python.org/3/library/argparse.html)

### sys

想用command-line输入，键入 `python3 meows.py -n 3`

```python
import sys
if len(sys.argv) == 1:
    print("meow")
elif len(sys.argv) == 3 and sys.argv[1] == "-n":
    n = int(sys.argv[2])
    for _ in range(n):
        print("meow")
else:
    print("usage: meow.py")
```

`-n` 常常表示后面的数字是次数

字母前用 `-` ，单词前用 `--`

使用很多 `if` 来分情况，参数位置严格

### argparse

a library that handles all the parsing of complicated strings of command-line arguments

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("-n")
args = parser.parse_args()

for _ in range(int(args.n)):
    print("meow")
```

> An object called `parser` is created from an `ArgumentParser` class. 
>
> That class’s `add_argument` method is used to tell `argparse` what arguments we should expect from the user when they run our program. 
>
> The parser’s `parse_args` method 相当于解析 `sys.argv` ，在终端中键入的所有参数都传给 `args` 
>
> Finally, running the parser’s `parse_args` method ensures that all of the arguments have been included properly by the user.
>
> `.` 引用一个 object 里的 property，`args.n` 是 `-n` 后输入的数字

### --h

`python3 meows.py -h` 或 `--help` 查看帮助信息

![截屏2024-08-04 21.14.04](./image/%E6%88%AA%E5%B1%8F2024-08-04%2021.14.04.png){width=70%}

`-n N` 表示需要在 `-n` 后输入数字作参数

更清楚的版本: 

```python
parser = argparse.ArgumentParser(description="Meow like a cat")
parser.add_argument("-n", help="number of times to meow")
```

![截屏2024-08-04 21.22.44](./image/%E6%88%AA%E5%B1%8F2024-08-04%2021.22.44.png){width=70%}

### default, type

```python
import argparse
parser = argparse.ArgumentParser(description="Meow like a cat")
parser.add_argument("-n", default=1, help="number of times to meow", type=int)
args = parser.parse_args()
for _ in range(args.n):
    print("meow")
```

line 3:

- `default=1` 默认 N 为 1
- `type=int` 会把参数按照数字处理（本来是str），line 5 就不用 `int()`

## Unpacking

Q：把一个list中的元素作为函数的参数？

```python
def total(galleons, sickles, knuts):
    return (galleons * 17 + sickles) * 29 + knuts
coins = [100, 50, 25]
print(total(coins[0], coins[1], coins[2]), "Knuts")
```

List就像packed with values，我们现在unpack list into multiple things

### *

在 **list** 变量前加 `*` 得到单个值

```python
print(total(*coins), "Knuts")
```

a `*` unpacks the sequence of the list of coins and passes in each of its individual elements to `total`

### 传参次序

```python
def total(galleons, sickles, knuts):
    return (galleons * 17 + sickles) * 29 + knuts
print(total(galleons=100, sickles=50, knuts=25), "knuts")
```

标明 `galleons=100` 后，参数可以不按顺序传递

### **

在 **dict** 变量前加 `*` 得到键值对

```python
coins = {"galleons": 100, "sickles": 50, "knuts": 25}
print(total(**coins), "knuts")
```

效果等价于 ⬆️

## args & kwargs

### args

`*args` 表示可以接收未知数目的参数，`args` are positional arguments

```python
def f(*args, **kwargs):
    print("Positional:", args)
f(100, 50, 25)
```

结果为 `Positional: (100, 50, 25)`

### kwargs

`kwargs` are named/keyword arguments

```python
def f(*args, **kwargs):
    print("Named:", kwargs)
f(galleons=100, sickles=50, knuts=25)
```

结果为 `Named: {'galleons': 100, 'sickles': 50, 'knuts': 25}` ，一个 dict

ps. 主要是 `*` 和 `**` ，`args` 和 `kwargs` 可以换名字

### print

```python
print(*objects, sep=' ', end='\n', file=sys.stdout, flush=False)
```

回顾print函数，`*objects` 表示可以接收任意数量的参数

## map

Q: 把一句话以大写输出

```python
def main():
    yell("This is CS50")
def yell(phrase):
    print(phrase.upper())
if __name__ == "__main__":
    main()
```

### 有返回值

```python
def main():
    yell(["This", "is", "CS50"])
def yell(words):
    uppercased = list()
    for word in words:
        uppercased.append(word.upper())
    print(*uppercased)
if __name__ == "__main__":
    main()
```

```python
def main():
    yell("This", "is", "CS50")
def yell(*words):
    uppercased = list()
    for word in words:
        uppercased.append(word.upper())
    print(*uppercased)
```

此时的yell很像print

### map

python’s documentation of [`map`](https://docs.python.org/3/library/functions.html#map)

apply some function to every element of some sequence like a list 对序列中的每个元素进行映射

```python
map(function, iterable, ...)
```

```python
def main():
    yell("This", "is", "CS50")
def yell(*words):
    uppercased = map(str.upper, words)
    print(*uppercased)  # unpack
```

str class中的upper function，注意没有 `()` ，map 会 call

## List Comprehensions

List comprehensions allow you to create a list on the fly in one elegant one-liner.

### list comprehensions

line 4 等价：

```python
uppercased = [word.upper() for word in words]
```

对每个参数，`.upper()` 都会作用在它们身上

### if condition

Q：筛选出住在Gryffindor的学生

```python
students = [
    {"name": "Hermione", "house": "Gryffindor"},
    {"name": "Harry", "house": "Gryffindor"},
    {"name": "Ron", "house": "Gryffindor"},
    {"name": "Draco", "house": "Slytherin"},
]
gryffindors = [
    student["name"] for student in students if student["house"]=="Gryffindor"
]
for gryffindor in sorted(gryffindors):
    print(gryffindor)
```

`gryffindors = []` 中的一整串是列表中的元素

## filter

python’s documentation of [`filter`](https://docs.python.org/3/library/functions.html#filter)

上述问题的另一种实现--filter

Using Python’s `filter` function allows us to return a subset of a sequence for which a certain condition is true

```python
def is_gryffindor(s):
    return s["house"] == "Gryffindor"
gryffindors = filter(is_gryffindor, students)
for gryffindor in sorted(gryffindors, key=lambda s: s["name"]):
    print(gryffindor["name"])
```

`filter( , )` 的第一个参数为函数（返回值是True or False），将会作用于序列的每个元素；第二个参数是序列

另一种传递函数的方法：

```python
gryffindors = filter(lambda s: s["house"] == "Gryffindor", students)
```

## Dictionary Comprehensions

Q: 已知学生名单，建立包含名字和房子的list

```python
students = ["Hermione", "Harry", "Ron"]
gryffindors = []
for student in students:
    gryffindors.append({"name": student, "house": "Gryffindor"})
print(gryffindors)
```

line 2 ~4 等价于：

```python
gryffindors = [{"name": student, "house": "Gryffindor"} for student in students]
```

进一步优化：

```python
students = ["Hermione", "Harry", "Ron"]
gryffindors = {student: "Gryffindor" for student in students}
print(gryffindors)
```

key是name，value都是Gryffindor

```bash
{'Hermione': 'Gryffindor', 'Harry': 'Gryffindor', 'Ron': 'Gryffindor'}
```

## enumerate

python’s documentation of [`enumerate`](https://docs.python.org/3/library/functions.html#enumerate)

Q：为输出编号

### way 1

```python
students = ["Hermione", "Harry", "Ron"]
for i in range(len(students)):
    print(i + 1, students[i])
```

```bash
1 Hermione
2 Harry
3 Ron
```

### way 2

```python
enumerate(iterable, start=0)
```

同时返回元素和index

```python
for i, student in enumerate(students):
    print(i + 1, student)
```

## Generators and Iterators

python’s documentation of [generators](https://docs.python.org/3/howto/functional.html#generators).

python’s documentation of [iterators](https://docs.python.org/3/howto/functional.html#iterators).

Q: 数羊

```python
def main():
    n = int(input("What's n? "))
    for i in range(n):
        print(sheep(i))
def sheep(n):
    return "🐑" * n
if __name__ == "__main__":
    main()
```

更好的做法是不在main中循环

```python
def main():
    n = int(input("What's n? "))
    for s in sheep(n):
        print(s)

def sheep(n):
    flock = []
    for i in range(n):
        flock.append("🐑" * i)
    return flock	# return a list
```

```bash

🐑
🐑🐑
```

### yield

如果输入 `100000` ，在建立 list 的时候内存占用过大，程序会崩溃，不会有任何输出

The `yield` generator can solve this problem by returning a small bit of the results at a time.

```python
def main():
    n = int(input("What's n? "))
    for s in sheep(n):
        print(s)
        
def sheep(n):
    for i in range(n):
        yield "🐑" * i 
```

`return` 会一次性返回；`yield` 在经历一次循环，就返回一个值。`return` 返回的是 `['', '🐑', '🐑🐑']` ；`yield` 分次返回  , `🐑` , `🐑🐑`

## Goodbye

>Ultimately then we hope with this course that you’ve not only learned python, that you’ve not only learned programming, but you’ve really learned how to solve problems, and ultimately how to teach yourself new languages.
>
>I mostly learned what I know now and even what I had to learn again for today, but just asking lots of questions. Be it a Google, or friends, who are more versed in this language than I. And so, having that instinct, having that vocabulary which to ask questions of others to search for answers to questions, you absolutely now have enough of a foundation in python and programming to stand on your own. So you can certainly and you’re welcome and encouraged to go on and take other courses in python and programming specifically. Find some projects that’s personally of interest. At least from my own experience, I tend to learn best and I hope you might too by actually applying these skills. Not to problems in the classroom, but really truly to problems in the real world.
>
>Our great hope is that you will use what you learned in this course to address real problems in the world, making our globe a better place.
>
>This was CS50!