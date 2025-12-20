# 对象定位与求值规则

## 分配对象: 通过 Compound Literal

```c
(type-literal){initializer-list}
```

- 定义了一个匿名 (unnamed) 的对象 (在结构体赋值时用到过)
- 示例: 

```c
int* p = &(int){1};
```

![截屏2025-10-15 23.12.36](./images_notes/截屏2025-10-15%2023.12.36.png)

## 分配对象: 通过内存管理函数

![截屏2025-10-15 23.15.26](./images_notes/截屏2025-10-15%2023.15.26.png)

- 注意: 对齐要求 Fundamental Alignment 保证了 `(int *)malloc(...)` 类似的强转不出问题

>为什么 int* 转 char* 再转回有问题?

## 定位对象

> [!important] 一言以蔽之
**通过左值 (lvalue) 定位 (designate) 对象**
左值就是能定位对象的表达式

### 表达式

![截屏2025-10-15 23.23.15](./images_notes/截屏2025-10-15%2023.23.15.png)

#### Evaluation of Expression

![截屏2025-10-15 23.26.03](./images_notes/截屏2025-10-15%2023.26.03.png)

如果 lvalue 进行 evaluate ...

#### 基础表达式 (Primary Expression)

![截屏2025-10-15 23.27.37](./images_notes/截屏2025-10-15%2023.27.37.png)

## 基础表达式

> [!summary] 
> 基本表达式当 lvalue


### 标识符

```c
T O = initializer
```

`O` 为标识符, 表达式中的对象标识符都是合法的 lvalue (因为看到标识符名都能定位到对象)

lvalue 定位对象类型:

![截屏2025-10-15 23.32.44](./images_notes/截屏2025-10-15%2023.32.44.png)

#### lvalue 表达式的 Evaluate 规则

给定一个 lvalue 表达式, *如果*这个 lvalue 进行 evaluate:

1. Value Computation
	1. 如果 lvalue 定位的对象是**非数组**类型对象, 则 lvalue 表达式 evaluate 之后的 rvalue 就是该对象的<u>对象值</u>, rvalue 类型是该对象类型的<u>非限定类型</u>
	2. 如果 lvalue 定位的对象是**数组**类型对象, 则 evaluate 后的 rvalue 就是该对象<u>第一个元素的首地址</u>, rvalue 类型是<u>元素对象类型对应的指针类型</u>
	3. 该过程常被称为 Decay (非标准术语)
2. Side Effect: 无

> [!abstract] 
> lvalue 被 evaluate 时得到的 <rvalue, rvalue type> 就是对象七元组的 <value, value type> !

### 常量
C 语言包括 5 种常量:
- integer-constant
- floating-constant
- enumeration-constant: `enum DAYS {FRIDAY, SATURDAY, SUNDAY}`
- character-constant: `'a'`
- predefined-constant: `false` `true` `null_ptr`

#### 整数常量

整数常量是基础表达式, 如果被evaluate:
- 其 rvalue 是整数常量表征的值
- rvalue type 是按表查出来的类型, 例子: [[C语言系统级编程第3课.pdf#page=40&selection=0,0,2,2|C语言系统级编程第3课, page 40]]

#### 枚举常量

```c
enum Season {Spring, Summer, Autumn, Winter};
```

`Spring` `Summer` `Autumn` `Winter` 称为枚举常量, 值分别为 0、1、2、3; rvalue 类型为 `enum Season`

#### 预定义常量

有三个: `false` `true` `null_ptr`

- `false` 和 `true` :
	- 值分别为 0、1
	- rvalue type 为 `bool`
- `null_ptr` 是空指针常量
	- rvalue type 是在 `<stddef.h>` 中定义的 `nullptr_t` 类型

### 字符串

我们已经知道字符串能够用来分配一个对象, 同时, 字符串是一个 lvalue, 能够定位分配出来的那个对象

- 字符串分配的对象类型是字符数组类型
- 根据规则推导即可

1. 字符串意味着分配 (或重用) 一个字符数组对象
2. 字符串是 lvalue 表达式
3. 定位刚刚分配 (或重用) 的那个字符数组对象

### 括号表达式

`(exp)` 等价于 `exp` , 但是优先级提到最高

## lvalue 做 evaluate 和不做 evaluate 的情况

### 定位非数组对象的 lvalue

给定一个能定位<u>非数组对象</u>的 lvalue 表达式 exp, 如果这个表达式:
1. 跟 `sizeof` 结合, 如 `sizeof(exp)` `sizeof exp`
2. 跟 `typeof` 结合, 如 `typeof(exp)`
3. 跟 `&` 结合, 如 `&exp`
4. 跟一元运算符 `++/--` 和后缀运算符 `++/--` 结合, 如 ++exp/--exp/exp++/exp--
5. 如果 lvalue 定位的对象类型是结构体/联合体, 跟 `.` 结合, 如: `exp.`
6. 出现在赋值运算符的左侧, 如: `exp =`

*除了以上 6 种情况, 这个表达式都要做 evaluate* , 即:

![截屏2025-10-25 20.02.52](./images_notes/截屏2025-10-25%2020.02.52.png)

注意: 是对 `a` 不做 evaluate, 但是整体, 比如 `sizeof(a)` 可以做. (也好理解, 比如对 `a` 做 evaluate 的到了 <1, int>, 那 `sizeof(a)` 怎么取呢? )

![截屏2025-10-25 19.59.25](./images_notes/截屏2025-10-25%2019.59.25.png)
### 赋值表达式的赋值过程

```c
a = 2;
```

- 首先对 `2` 做 evaluate, 得到 `<2, int>` 
- 其次观察 `a` 和 rvalue type 匹配, 所以把 rvalue 放到 `a` 定位的对象中

`a = 2` 这个表达式的值是 `2` , 副作用是把 `a` 的值变为 `2`

### 定位数组对象的 lvalue

定位数组对象的 lvalue 表达式 exp, 如果这个表达式:
1. 跟 `sizeof` 结合, 如 `sizeof(exp)` `sizeof exp`
2. 跟 `typeof` 结合, 如 `typeof(exp)`
3. 跟 `&` 结合, 如 `&exp`
4. 如果 lvalue 定位的是一个 String Literal, 且用于初始化一个字符数组, 例如: `char str[] = "hello"` 里的 `"hello"`

*除了以上 4 种情况, 这个表达式都要做 evaluate*

```c
int b[1] = {1};
```

`b=NULL` 为什么不行? 因为 `b` 是不可修改左值