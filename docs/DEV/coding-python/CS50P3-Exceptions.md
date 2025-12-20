# CS50P: 3. Exceptions

## SyntaxError

语法错误

## ValueError

**情况一：user输入非期望数据类型**

```python
x = int(input("What's x? "))
print(f"x is {x}")
```

如果用户输入 `cat` ，会得到 `ValueError: invalid literal for int() with base 10: 'catcat'`

解决方案：

```python
try:
    x = int(input("What's x? "))
    print(f"x is {x}")
except ValueError:
    print("x is not an interger")
```

## try & except

python关键字

## NameError

doing sth with a variable that u shouldn't

```python
try:
    x = int(input("What's x? "))
except ValueError:
    print("x is not an interger")
print(f"x is {x}")
```

如果输入 `wolffy` ，返回 ` x is not an interger` 和 `NameError: name 'x' is not defined` ，因为在 `int` 接收字符串输入的时候就已经发生了ValueError，右边的值并未赋给左边。

**赋值即定义？**

## 增加else

```python
try:
    x = int(input("What's x? "))
except ValueError:
    print("x is not an interger")
else:
    print(f"x is {x}")
```

如果try成功了，没出现Error，就会执行 `print(x)`；如果try失败，执行except来处理ValueError，然后跳出程序(当前代码块，跳到下一个阶段)

### 改进

直到user输入合法才停止

```python
while True:
    try:
        x = int(input("What's x? "))
    except ValueError:
        print("x is not an interger")
    else:
        break
print(f"x is {x}") 
```

## pass

跳过某种情况，catch it, and ignore it

例如：

```python
except ValueError:
    pass
```

Pythonic: try things, hopefully they work, but if they don't, handle the exception

不同于C的使用if 

## raise

