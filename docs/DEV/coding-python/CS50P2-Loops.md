# CS50P: 2. Loops

`control + C` 终止循环

## while循环

```python
#meow 3 times
i = 0
while i < 3:
    print("meow")
    i += 1		#python中没有i++
```

## for循环

```python
for i in [0, 1, 2]:
    print("meow")
```

i 初始为 0，依次取 1、2

**in 可以让 i 按序取遍 list 中的元素，其中元素可以是 int, dict, str, etc.**

```python
for _ in range(3):	
  	print("meow")
```

`range(x)` 返回 `[0, 1, ..., x-1]`

 `_` 是Pythonic的用法，因为 `i` 只用来计数，之后不会用到

## *

```python
print("meow\n" * 3, end = '')
```

## continue & break

```python
while True:    
    n = int(input("What's n? "))
    if n < 0:
        continue    #继续循环
    else:
        break
```

如果想让input符合预期，刻意用 `while True`

当然，还可以写为：

```python
if n > 0:
  	break
```

## list

a datatype

```python
students = ["Hermoine", "Harry", "Ron"]
for student in students:
    print(student) 
```

这样可以不考虑list的长度

student会初始化为 `Hermoine` ，然后取 `Harry` ……

## len

`len()` 返回列表的元素个数

```python
for i in range(len(students)):
    print(students[i])
```

使用列表中的元素 `students[i]`

## dict (dictionaries)

**associate** one value with another, keys(words) and values(definition)

### 定义变量

```python
students = {
    "Hermoine": "Gryffindor", 
    "Harry": "Gryffindor", 
    "Ron": "Gryffindor",
    "Draco": "Slytherin",
}
```

what do I want to associate Hermoine with? Gryffindor.

keys on the left, values on the right（美观）

### 引用

想输出 Hermoine 的 house？

**方法一**

```python
print(students["Hermoine"])
```

和 list 的数字下标不同，dict 使用 actual words as index，得到 value 

**方法二**

```python
for student in students:
    print(student)
```

这样会输出所有的 key

```python
for student in students:
    print(student, students[student])
```

`for student in students:` 会使 `student` 取 `"Hermione"` 到 `"Draco"`

如果 `student` 的名字是 key，那 `students[student]` 会去取 `student` 的 value

### 多个value

![截屏2024-07-11 08.39.10](./image/%E6%88%AA%E5%B1%8F2024-07-11%2008.39.10.png){width=50%}

a list of dictionaries, a list of 4 students, each of the students is a dictionary

```python
students = [
    {"name": "Hermoine", "house": "Gryffindor", "patronus": "Otter"},
    {"name": "Harry", "house": "Gryffindor", "patronus": "Stag"},
    {"name": "Ron", "house": "Gryffindor", "patronus": "Jack Russell terrier"},
    {"name": "Draco", "house": "Slytherin", "patronus": None}
]
```

### 引用多个value

```python
for student in students:
    print(student["name"], student["house"], student["patronus"])
```

student 是 dict 型变量

## None

python关键字，the absence of a value
