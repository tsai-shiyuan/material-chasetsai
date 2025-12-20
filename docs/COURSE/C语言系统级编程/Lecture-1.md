# 内存、对象、类型

## 标准化C语言

```c
int a = 0;
const int b = 0;
const volatile int c = 0;	// 程序员不能改, 系统能以未知的方式修改
int d[10] = {0};
const int e[10] = {0};	// 不能 e[0]=1
const volatile int f[10] = {0};
```

`a` `b` `c` `d` `e` `f` 叫作什么? 变量 or 常量 or 可变常量, 这样分类过于复杂. 

$\Rightarrow$ 不应该称 `a` 为变量, 标准文本也并没有 <u>variable</u> 这个术语

### 区分常量和常量表达式

constant vs. constant expression:

- 10 是 _整数常量_
- +10 不是整数常量, 而是整数常量表达式
- (int)(10.0) 不是整数常量, 而是整数常量表达式
- (int)(+10.0) 不是整数常量, 也不是整数常量表达式

### C语言标准背后的设计思想

1. 术语规则的**一般化**
2. 概念组织的**系统化**

$\Rightarrow$ 在这样的思想下, 把 `a` `b` `c` `d` `e` `f` 全部称为**对象标识符**, 既不是变量也不是常量, 其处理逻辑规则是一致的 (在 C 语言中**没有区别**)

### C语言概念体系

以对象为核心构建:

![截屏2025-09-20 21.10.36](./images_notes/%E6%88%AA%E5%B1%8F2025-09-20%2021.10.36.png)

## 内存

1 byte 由 CHAR_BIT bit 组成, 内存由一系列 byte 按线型顺序排列

- CHAR_BIT $\geqslant$ 8

内存地址: 字节在内存中的编号

## 对象

![截屏2025-09-20 21.16.52](./images_notes/%E6%88%AA%E5%B1%8F2025-09-20%2021.16.52.png)

对象: 

- region of data storage, the **contents** of which can represent **values**
- objects are composed of contiguous sequences of **one or more bytes**

### 对象类型

一个对象一般有一个对象类型 (Object Type), 形式化定义成 **T**

对象的类型可以分为**完全对象类型**和**不完全对象类型**. 一个对象, 如果其类型是完全对象类型, 则大小程序员可知 (个人理解: 能通过 sizeof 得到), 反之无法获知其大小.

### 对象表示和对象值

![截屏2025-09-20 21.22.42](./images_notes/%E6%88%AA%E5%B1%8F2025-09-20%2021.22.42.png)

对象各个 bit 组成的二进制串就是这个对象的**对象表示 (Object Representation)**, 再加上对象类型, 就能得到**对象值 (Object Value)**. (ps. 对象表示和对象值可以看作一个东西, 是可以通过对象类型进行双向映射的)

组成对象表示的 bits 分为 Value Bits 和 Padding Bits

### size

若一个对象包含字节个数 n, 则称这个对象的 size 为 n ( `sizeof(obj) = n` ), 该对象包含的 bit 数为 n\*CHAR_BIT

### 对象地址和对齐要求

![截屏2025-09-20 21.30.08](./images_notes/%E6%88%AA%E5%B1%8F2025-09-20%2021.30.08.png)

sizeof(T): 如果有一个 T 类型对象, 这个对象的大小

- 注意不要先入为主地认为 int 一定是 4 bytes, 唯一的标准是 sizeof 的输出

alignof(T): 一个 T 类型的对象的缺省对齐要求 (即程序员可以修改)

- 缺省对齐要求: 编译器默认的对齐方式

## 对象类型分类

![截屏2025-09-20 21.42.18](./images_notes/%E6%88%AA%E5%B1%8F2025-09-20%2021.42.18.png)

### 算术类型

![截屏2025-09-20 19.12.54](./images_notes/%E6%88%AA%E5%B1%8F2025-09-20%2019.12.54.png)

- 算术类型都是完全对象类型
- 除了枚举类型, 其他所有的算术类型都是基础类型
- char, signed char, unsigned char 是三种不同的类型, 编译器会指定 char 类型与 signed char/unsigned char 其中之一行为一致
- 标准有符号整数类型的对象
  - bits 的组成: 1 个 sign bit + 若干个 value bits + 若干个 padding bits
  - sign bit + value bits 的个数是 N, N 称为对象类型 T 的宽度
  - signed char 的宽度一定等于 CHAR_BIT
- 枚举类型
  - size 由实际实现类型决定
  - 定义时指定实现类型: `enum Season : int {Spring, Summer, Autumn, Winter};`
  - 如果不指定实现类型, 实现类型由编译器自行设置

### 派生类型

程序员构造的一些新类型

<img src="./images_notes/%E6%88%AA%E5%B1%8F2025-09-20%2022.52.13.png" alt="截屏2025-09-20 22.52.13" style="zoom:50%;" />

#### 数组类型

定义: `T[N]`

- `T[N]` 被视作从 T 类型派生的类型, **Array of T**
- T: 是**元素类型**, *必须是完全对象类型*
	- 比如: 因为 void 是不完全对象类型, 所以不能派生 `void[10]`
- N: 元素类型的个数
	- 没有 N, `T[]` 为 <u>不完全对象类型</u>
	- N 为**整数常量 / 整数常量表达式**, 则 `T[N]` 为普通数组类型
	- 否则, `T[N]` 为变长数组类型
	- 此时 , `T[N]` 是完全对象类型

大小: 

- sizeof(`T[N]`) = N $\times$ sizeof(T)
- `T[]` 为不完全对象类型, 没有 size

对齐要求:

- alignof(`T[N]`) = alignof(T) 数组类型的对齐要求是元素类型的对齐要求
- `T[]` 为不完全对象, 没有 alignment

**typedef:**

```c
typedef T Alias		// Alias 在语法上是一个整体
```

把 `int[5]` 设置成别名 AINT:

```c
typedef int AINT[5];
typedef int[5] AINT;	// wrong
```

- 因为 `T[N]` 在语法上不是一个整体, 但 AINT 在语法上是一个整体, 所以第二行的定义是错的

`typeof` 将数组类型形式化为一个 T:

```c
typedef typeof(int[5]) AINT;
```

继续派生, `int[5]` 当作 T, N 是 10, 则派生出 `int[10][5]` , Array of `int[5]` ; 现在 `int[10][5]` 等价于 `AINT[10]` $\Rightarrow$ 数组都是“一维”的

- `T[N]` *从左到右*把第一个碰到的 `[N]` 删掉, 剩下的就是元素类型

#### 指针类型

指针类型的形式化定义: T*

- T: Referenced Type, **对象类型或函数类型**. (包括完全对象类型和不完全对象类型, 还包括函数类型)
- Referenced Type (T) 和 Pointer Type (T*) 一一对应
- T* 被视作一个从 T 类型派生的类型, **Pointer to T**
- 任何指针类型都是**完全对象类型**

![截屏2025-09-21 09.44.19](./images_notes/%E6%88%AA%E5%B1%8F2025-09-21%2009.44.19.png)

既然 T* 都是完全对象类型, 那么它就可以用来派生数组类型, 规则如下:

| 类型   | 派生数组类型    | 派生指针类型  |
| ---- | --------- | ------- |
| T    | T[N]      | T*      |
| T[M] | `T[N][M]` | T(*)[M] |
| T*   | T*[N]     | T**     |

**复杂类型识别:** 识别的过程 () 优先级最高, 且有可能嵌套, 先识别最内部的 ()

### 限定类型

>类型 T 可以构造新的限定类型 (Qualified Type)

#### 四种限定符 (Qualifier)

(1) `_Atomic`

- 类型 T 的原子限定类型为 `_Atomic T` / `T _Atomic` 
- 类型对象的大小和缺省要求是 `sizeof(_Atomic T)` 和 `alignof(_Atomic T)`

(2) const

(3) volatile

(4) restrict

给定类型 T , 给定三个限定符之一的 Q , 其**限定类型**为 QT 或 TQ

- T 和 QT 或 TQ 是两种不同的 Type, 但大小、表示值、对齐方式一样
- <u>当 T 是一个整体时</u>, Q 在 T 的左边右边都一样, 比如 Q typeof(T) $\equiv$ typeof(T) Q
- 对 `T[N]` 、T* , Q 在左边右边是有区别的:

![截屏2025-09-21 10.35.34](./images_notes/%E6%88%AA%E5%B1%8F2025-09-21%2010.35.34.png)

基于限定类型派生:

![截屏2025-09-23 21.23.58](./images_notes/%E6%88%AA%E5%B1%8F2025-09-23%2021.23.58.png)

- 都是在整体后面加上 `[N]` 或 `*`

#### 限定符限定数组

`Q typeof(T[N])` 等价于 `Q typeof(T)[N]` (限定符 Q 先去结合元素类型 T 形成限定类型, 再构造 N 个这样的类型形成数组)

#### 多个不同限定符的语义

一个类型可以同时使用多个限定符, 比如: const volatile int

那限定符的顺序不同会怎样呢? 答案是与语义相同, 比如: const volatile int 和 volatile const int 等价.

### typedef

typedef 为 类型 T 设置别名:

![截屏2025-09-21 10.27.12](./images_notes/%E6%88%AA%E5%B1%8F2025-09-21%2010.27.12.png)

- 感觉 Alias 是紧跟在 * 之后, `[N]` 之前

例如:

```c
// int*[10] 是数组类型
typedef int* APINT[10];

// int(*)[10] 是指针类型
typedef int(*PAINT)[10];
```

typedef 为 QT、TQ、T\*Q 设置别名:

![截屏2025-10-15 19.01.08](./images_notes/截屏2025-10-15%2019.01.08.png)

- Alias 紧跟在整体的后面

#### 语法整体性

![截屏2025-10-15 19.30.52](./images_notes/截屏2025-10-15%2019.30.52.png)
