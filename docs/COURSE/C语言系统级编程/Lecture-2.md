# 对象属性、对象分配

> 可修改左值是从我们程序员的视角看的, 但内存是否会修改是另一回事, 有可能是未定义行为

## 对象属性: 物理属性

![截屏2025-10-15 20.41.04](./images_notes/截屏2025-10-15%2020.41.04.png)

- 对象类型 (Object Type): T
- 对象名称 (Object Name): O
	- 有名称的对象被称为具名对象, 反之则是匿名对象 (Unnamed Object)
- 大小 (size): 
	- 连续字节的个数, 通过 sizeof(T) 获得
	- 假设常用类型大小为: char 1, short 2, int 4, float 4, double 8, pointer type 4
- 地址 (Address) 和对齐要求 (Alignment):
	- 对象地址由系统确定
	- 地址要满足对齐要求, 即 alignof(T)
- 对象值和对象表示: 映射

## 分配对象: 对象定义

> [!summary] 分配对象的方法: 
> 1. 对象定义
> 2. String Literal (字符串)
> 3. Compound Literal
> 4. 内存管理函数 (malloc)

```c
Storage-class specifiers T O = initializer;
```

![截屏2025-10-15 20.43.52](./images_notes/截屏2025-10-15%2020.43.52.png)

- 分配了一个 `T` 类型的对象, `T` 要看成**整体**
- 声明一个对象标识符 `O` , 让 `O` 与这个对象关联起来
- 用 `initializer` 初始化这个对象的**对象表示**. 如果没有 initializer, 则不构成对象定义
- Storage-class specifiers 指定对象存储周期

### `T`、`T[N]`、`T*`

![截屏2025-10-15 20.47.52](./images_notes/截屏2025-10-15%2020.47.52.png)

比如:

```c
int b[10] = {0};
int *c = NULL;
```

### 一次定义多个对象

![截屏2025-10-15 20.51.26](./images_notes/截屏2025-10-15%2020.51.26.png)

- `T` 需要是整体

一次定义多个 const 类型对象:

![截屏2025-10-15 20.56.15](./images_notes/截屏2025-10-15%2020.56.15.png)

### 标量类型对象的 initializer

标量类型: 

![截屏2025-10-15 21.00.55](./images_notes/截屏2025-10-15%2021.00.55.png)


initializer:

![截屏2025-10-15 21.02.09](./images_notes/截屏2025-10-15%2021.02.09.png)

- `{}` 将对象的值设置成该类型的*缺省值* (0, 空指针)
- `{exp}` 表达式求值之后的右值用来初始化对象表示
	- 很神奇, 居然可以这样

```c
int a = {1+2};
printf("%d\n", a);
```

<u>完整的 initializer 就是对象的对象值 (Object Value)</u>

### 数组类型对象的 initializer

![截屏2025-10-15 21.05.09](./images_notes/截屏2025-10-15%2021.05.09.png)

- 当数组是普通数组, 可以: `{[constant-expression-i]=initializer-i, ...}` 设定第 `i` 个元素的值 
	- 试了一下会报错呀?
- 当 `N` 不存在:
	![截屏2025-10-15 21.06.31](./images_notes/截屏2025-10-15%2021.06.31.png)

- 当数组是变长数组:

	![截屏2025-10-15 21.06.59](./images_notes/截屏2025-10-15%2021.06.59.png)

### 若没有 initializer

`int a` 这个问题和 scope 的 linkage 有关:

1. 在某些情况下, `int a` 是一个对象定义, 且值是确定的
2. 在某些情况下, `int a` 是一个对象定义, 但值不确定
3. 在某些情况下, `int a` 不是一个对象定义, 只是一个标识符声明

此时的 Object Value ?

```c
int main() {
	int a;
	return 0;
}
```

- 这里对象 `a` 的对象表示被称为 indeterminate representation
- 对象 `a` 的对象值也可称为 indeterminate 不确定的
- (其实从硬件角度上是确定的, 就是 0 或 1, 但是我们程序员不知道)

## 对象属性: 逻辑属性

![截屏2025-10-15 21.17.23](./images_notes/截屏2025-10-15%2021.17.23.png)

**表示值**和**表示值类型**:

![截屏2025-10-15 21.19.40](./images_notes/截屏2025-10-15%2021.19.40.png)

<Value, Value Type> 获取规则:

![截屏2025-10-15 21.20.57](./images_notes/截屏2025-10-15%2021.20.57.png)

### 对象七元组模型

![截屏2025-10-15 21.24.39](./images_notes/截屏2025-10-15%2021.24.39.png)

## 分配对象: 通过 String Literal

String Literal: 编码前缀 + 双引号引导的字符序列

- 5 种编码前缀: 无编码、u8、u、U、L. 例如: "hello" u8"hello" u"hello" U"hello" L"hello"
- 所有 String Literal 具有静态存储周期 (有效期覆盖整个程序运行时间, 可初始化为缺省值)

> C语言定义了 4 种对象存储周期
> - static, automatic, allocated, thread
> - automatic 相当于栈, allocated 相当于堆
> - 存储周期决定了对象的有效期

### "hello" 分配对象

![截屏2025-10-15 21.40.06](./images_notes/截屏2025-10-15%2021.40.06.png)

### 其他前缀

![截屏2025-10-15 21.45.20](./images_notes/截屏2025-10-15%2021.45.20.png)

- 有中文一定要前缀, 否则存在跨平台问题

### String Literal 和 String

String 的 `""` 内不能有 `\0`




