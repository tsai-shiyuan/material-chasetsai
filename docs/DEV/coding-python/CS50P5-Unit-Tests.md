# CS50P: 5. Unit Tests

## assert

[python: assert](https://docs.python.org/3/reference/simple_stmts.html#assert)

calculator.py :

```python
def main():
    x = int(input("What's x? "))
    print("x squared is", square(x))
def square(n):
    return n + n	#刻意为之
if __name__ == "__main__":
    main()
```

test_calculator.py :

```python
from calculator import square
def main():
    test_square()
def test_square():
    assert square(2) == 4
    assert square(3) == 9
if __name__ == "__main__":
    main()
```

在终端运行，得到 `AssertionError` ，同样可以用 `try & except` 语句

```python
def test_square():
    try:
        assert square(2) == 4
    except AssertionError:
        print("2 squared is not 4")
    try:
        assert square(3) == 9
    except AssertionError:
        print("3 squared is not 9")
    try:
        assert square(-2) == 4
    except AssertionError:
        print("-2 squared is not 4")
    try:
        assert square(-3) == 9
    except AssertionError:
        print("-3 squared is not 9")
    try:
        assert square(0) == 0
    except AssertionError:
        print("0 squared is not 0")
```

## pytest

[pytest’s documentation](https://docs.pytest.org/en/7.1.x/getting-started.html)

可以测试函数的第三方库，相当于帮我们做了 `try & except`

### 示例

test_calculator.py :

```python
from calculator import square
def test_square():
    assert square(2) == 4
    assert square(3) == 9
    assert square(-2) == 4
    assert square(-3) == 9
    assert square(0) == 0
```

注意到没有 main 函数了

运行 `pytest test_calculator.py`

![截屏2024-07-16 17.10.54](./image/%E6%88%AA%E5%B1%8F2024-07-16%2017.10.54.png)

### categories test

上例中遇到错误即停止，但我们往往希望一次获得更多的错误信息，以便debug，需要分割测试数据

代码：

```python
def test_positive():
    assert square(2) == 4
    assert square(3) == 9

def test_negative():
    assert square(-2) == 4
    assert square(-3) == 9
    
def test_zero():
    assert square(0) == 0
```

运行结果：（F == Fail；. == Pass）

![截屏2024-07-16 17.49.34](./image/%E6%88%AA%E5%B1%8F2024-07-16%2017.49.34.png)

某一函数测试出错，之后的函数照旧进行测试。

### 特例——用户输入字符串

应用 pytest 库中的 raises 函数

```python
import pytest
```

```python
def test_str():
    with pytest.raises(TypeError):
        square("cat")
```

括号内是可能发生的错误类型，什么时候期望错误被提出？当输入字符时会

## tests文件夹

当需要对多个函数进行测试时，往往将测试文件整理为一个文件夹

Command window :

```bash
mkdir test
code test/__init__.py
```

空的 `__init__.py` 指示出 `test` 文件夹不单单是file或module，是一个package，里面有可以进行测试的文件

测试时：

```bash
pytest test
```

