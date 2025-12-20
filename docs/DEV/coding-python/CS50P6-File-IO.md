# CS50P: 6. File I/O

## File I/O

### list相关函数

[python: list](https://docs.python.org/3/library/stdtypes.html#lists)

`append() `

- 向列表末尾添加元素

```python
list.append(element)
```

`sorted(*, key=None, reverse=False)`

* [python: sorted](https://docs.python.org/3/library/functions.html#sorted)

- sorts the list in place, using only `<` comparisons between items

* key: eg. `key=str.lower()` 说明用小写来排序，但是列表还是不变
* 稳定排序

### 程序

```python
names = []
for _ in range(3):
    names.append(input("What's your name? "))
for name in sorted(names):
    print(f"hello, {name}")
```

程序一旦结束，所有信息都会消失，File I/O可以存储信息，便于后续使用

## open

### open files

[python: open](https://docs.python.org/3/library/functions.html#open)

```python
file = open("name.txt", "w") 
```

`open` 以写入的方式打开name.txt文件。如果不存在，就创建一个文件；如果存在，会清空

返回“文件指针”

### 写入文件

```python
name = input("What's your name? ")
file = open("name.txt", "w")
file.write(name)
file.close()
```

每次运行程序都从头重写name.txt文件

### 追加内容

`"w"` 替换成 `"a"` (append)

## with

自动关闭文件

```python
with open("name.txt", "a") as file:
    file.write(f"{name}\n")
```

file得到open的返回值

## 读文件

### 全部读

```python
with open("name.txt", "r") as file:
    lines = file.readlines()
for line in lines:
    print("hello,", line.rstrip())	#去掉文件换行符
```

read所有行，存入lines，是一个list

`"r"` 可省，是默认值

### 逐行读

```python
with open("name.txt", "r") as file:
    for line in file:
        print("hello,", line.rstrip())
```

不需要先读所有行，`for line in file` 可以一行一行地读

### 排序

```python
names = []
with open("name.txt", "r") as file:
    for line in file:
        names.append(line.rstrip())
for name in sorted(names):
    print(f"hello, {name}")
```

## CSV

CSV stands for "comma separated values"

### students.csv文件

```
Hermione,Gryffindor
Harry,Gryffindor
Ron,Gryffindor
Draco,Slytherin
```

一列名字，一列房屋，用 `,` 隔开

### 读取信息

```python
with open("students.csv") as file:
    for line in file:
        row = line.rstrip().split(',')
        print(f"{row[0]} is in {row[1]}")
```

`split()` 返回list

设置两个变量接收信息，而非一个list：

```python
name, house = line.rstrip().split(',')
print(f"{name} is in {house}")
```

### 排序

#### 存成元素为 dict 的 list

```python
students = []

with open("students.csv") as file:
    for line in file:
        name, house = line.rstrip().split(',')
        student = {}    #student是一个dict
        student["name"] = name
        student["house"] = house
        students.append(student) 

for student in students:
    print(f"{student['name']} is in {student['house']}")
```

注意：

1. line 12 中 `'name'` 用单引号括起来，因为已经使用过双引号

2. line 6~8 可以换成 `student = {"name": name, "house": house}`

#### sort keys

```python
def get_name(student):      #student is a dict
    return student["name"]
for student in sorted(students, key=get_name):
    print(f"{student['name']} is in {student['house']}")
```

line 3 `key=get_name` ：sorted函数会用list中的每个字典自动帮我们call get_name(student)，可以理解为key指示如何排序

另一种写法：

```python
for student in sorted(students, key=lambda student: student["name"]):
```

**`lambda` function**: 未名函数，"Hey Python, here is a function that has no name: Given a `student` (参数), access their `name` and return that to the `key` "

### CSV Library

CSV是Python的一个库，[python: CSV](https://docs.python.org/3/library/csv.html)

#### csv.reader

假设csv文件某行有不止一个 `,` ，可以用 `""` 括起来，再调用库

```bash
Harry,"Number Four, Privet Drive"
Ron,The Burrow
Draco,Malfoy Manor
```

```python
import csv
students = []
with open("students.csv") as file:
    reader = csv.reader(file)
    for row in reader:
        students.append({"name": row[0], "home": row[1]})
```

A `reader` works in a `for` loop, where each iteration the `reader` gives us another row from our CSV file. This row itself is a list, where each value in the list corresponds to an element in that row

因为 row 是一个 list，所以 line 5~6 可写为：

```python
for name, home in reader:
    students.append({"name": name, "home": home})
```

#### csv.DictReader

通常csv文件头部是描述 (header information)，比如：

```bash
name,home
```

```python
with open("students.csv") as file:
    reader = csv.DictReader(file)
    for row in reader:	#row是字典
        students.append({"name": row["name"], "home": row["home"]})
```

DictReader 从头到尾，把每一行看作一个 dict 并返回

the compiler will directly access the `row` dictionary, getting the `name` and `home` of each student

`row["name"]` 需要 `.csv` 文件中有 `name` 一栏，并且不要求 `name` 和 `home` 有序

#### csv.writer

```python
import csv
name = input("What's your name? ")
home = input("Where's your home? ")
with open("students.csv", "a") as file:
    writer = csv.writer(file)
    writer.writerow([name, home])
```

#### csv.DictWriter

```python
with open("students.csv", "a") as file:
    writer = csv.DictWriter(file, fieldnames=["name", "home"])
    writer.writerow({"home": home, "name": name})
```

fieldnames 参数决定了 csv 文件里 name 在前，home 在后，即 line 3 的输入同样会得到先名字再房子

## PIL

a python library

[Pillow’s documentation of: PIL](https://pillow.readthedocs.io/)

```python
import sys
from PIL import Image

images = []

for arg in sys.argv[1:]:
    image = Image.open(arg)
    images.append(image)

images[0].save(
    "costumes.gif", save_all=True, append_images=[images[1]], duration=200, loop=0
)
```

The last lines of code saves the first image and also appends a second image to it as well, creating an animated gif

`loop=0` 循环无限次

![costumes](./image/costumes.gif){width=25%}