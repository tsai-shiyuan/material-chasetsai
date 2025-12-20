# CS50P: 1. Conditionals

## 运算符

python支持 `90 <= score <= 100`

| C    | Python |
| ---- | ------ |
| \|\| | or     |
| &&   | and    |

## 布尔运算

`True` or `False`

## 选择语句

### if

```python
if x < y:
    print("x is less than y")
if x > y:
    print("x is greater than y")
if x == y:
    print("x is equal to y")
```

* `:`
* `....` 四个缩进，表示只有当 `if` 为真，该语句才能被执行。有缩进是一个代码块

代码的逻辑：三个 `if` 都会被执行

![截屏2024-07-10 08.42.30](./image/%E6%88%AA%E5%B1%8F2024-07-10%2008.42.30.png)

### elif

```python
if x < y:
    print("x is less than y")
elif x > y:
    print("x is greater than y")
elif x == y:
    print("x is equal to y")
```



![截屏2024-07-10 08.49.55](./image/%E6%88%AA%E5%B1%8F2024-07-10%2008.49.55.png)

### else

```python
if x < y:
    print("x is less than y")
elif x > y:
    print("x is greater than y")
else:
    print("x is equal to y")
```

![截屏2024-07-10 08.55.01](./image/%E6%88%AA%E5%B1%8F2024-07-10%2008.55.01.png)

### match

类似 `switch`

原：

```python
if name == "Harry" or name == "Hermione" or name == "Ron":		#字符串 ==
    print("Gryffindor")
elif name == "Draco":
    print("Slytherin")
else:
    print("Who?")
```

match：

```python
match name:
    case "Harry" | "Hermione" | "Ron": 
        print("Gryffindor")
    case "Draco":
        print("Slytherin")
    case _:
        print("Who?")
```

* `case _:` 表示else
* `|` 在case中表示或

## Pythonic Expressions

```python
def is_even(x):
    return True if x % 2 == 0 else False
```

```python
def is_even(x):
    return x % 2 == 0
```

