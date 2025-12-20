# 对象求值和地址常量

## 可修改左值和不可修改左值

给定一个 lvalue 表达式 (就一定能定位一个对象), 这个 lvalue 定位一个对象的对象类型 `T` :
1. `T` 是数组类型
2. `T` 是 `const` 限定类型
3. `T` 是不完全对象类型
4. `T` 是结构体/联合体类型, 其成员对象类型有 `const` 限定类型

*则这个 lvalue 称为不可修改左值, 其他均为可修改左值*

> 这里可修改和不可修改只是从 lvalue 的视角看的, 而不是这片内存能否被修改

## 字符数组

初始化:

```c
char str[8] = {'h', 'e', 'l', 'l', 'o'};
```

- 后面会补上 3 个 `\0`

```c
char str[8] = "hello";
```

- 字符数组可以用 String Literal 初始化. 当看到 `"hello"` 就马上分配并定位到对象, 然后把这块对象搬过去
- 第 6 个 `\0` 是 `"hello"` 自己带的
- 此时 `"hello"` **不**做 evaluate, 用于初始化 (如果做的话会得到 V_T 为 `char *` , 又怎能赋值给 `char[]` 呢)

`U` 是定长的 `L` 是变长

## Compound Literal

1. Compound Literal 意味着**分配一个指定类型的对象**
2. Compound Literal 是一个有效的 lvalue 表达式
3. 定位刚刚分配的那个对象

(咳咳, 这就和字符串的语义一样嘛)

### Evaluate

对 `(int){1}` 这个 lvalue 进行 evaluate:

![截屏2025-11-17 21.29.42](./images_notes/截屏2025-11-17%2021.29.42.png)

- `(int){1}` 和 `int a = 1` 中的 `a` 是相同的作用

对 `(const int){1}` 这个 lvalue 进行 evaluate:

- 这个 lvalue 是不可修改左值 (因为定位的对象类型是 `const` ), 所以没有 `(const int){1}++`

### 分配数组

对 `(int[1]){1}` 这个 lvalue 表达式进行 evaluate

![截屏2025-11-17 21.43.40](./images_notes/截屏2025-11-17%2021.43.40.png)

- `&` 得到的是 `Obj_T*`

## `sizeof`, `&`

给定一个定位非数组对象的 lvalue 表达式:

`sizeof` :
1. 跟sizeof结合，该 lvalue **不做** evaluate
2. `sizeof(lvalue)` 的语义是获得 lvalue 定位对象的大小
3. 这个对象的大小由对象类型可知, *编译时可以推断出来*. 对象的size在生命周期内不会变化

`&`:

1. 跟 `&` 结合, lvalue 不做 evaluate
2. 如果 lvalue 定位对象的 Storage Duration 是 static, 其对象地址编译时可知; 否则该对象地址运行时才可获得

### 地址常量

地址常量的指的是:

1. 空指针
2. 指向一个 static storage duration 的对象的指针
3. 指向 Function Designator 的指针

地址常量是特定表达式 evaluate 的结果

![截屏2025-11-17 22.33.09](./images_notes/截屏2025-11-17%2022.33.09.png)

- `&z` 得到的 rvalue 是地址常量
- `"hello"` 是地址常量 (因为隐式地用数组开头的 `'h'`) (字符串分配对象都具有 static 的 storage duration)

## 定位指针类型对象的 lvalue

```c
int a = 1;
int* p = &a;
```

称对象标识符 `p` 能定位一个对象, 其对象类型为 `int*`

![截屏2025-11-18 16.13.38](./images_notes/截屏2025-11-18%2016.13.38.png)

- 开始 `p` 的 Value 是 Undefined

### 理解 `*exp`

`*exp` 是一个 **lvalue**, **定位**一个对象

> 注意之前所说的 `*` 就是取值是错误的, 也没有解引用的说法! 表达式的值只有在 evaluate 之后才有.

假设表达式 `exp` evaluate 之后的 rvalue 为 `<Value, Value_Type>` , 如果 `Value_Type` 是**对象指针**类型, 则可以用 `*exp` 的方式来定位到 `Value` 对应字节编号开头的一段内存 (也就是一个对象 M)

1. 该对象的 Address 为 Value
2. 该对象的 Object Type 为 Value_Type 对应的 *Referenced Type*
3. 对象的其他属性随之确定

![截屏2025-11-18 16.23.51](./images_notes/截屏2025-11-18%2016.23.51.png)

- `*p` 对象没有 name
- `*p` 和 `a` 定位到的对象是一样的

### `p+n` 的含义

标量类型 (3 种): 算术类型, 指针类型, nullptr -> 可以做加减法

![截屏2025-11-18 16.39.03](./images_notes/截屏2025-11-18%2016.39.03.png)

注意: 

`*(exp + n)` ⇔ `exp[n]`

`exp1[exp2]` ⇔ `exp2[exp1]` (要求其中一个表达式 evaluate 后的 rvalue 类型是有效的对象指针类型, 另一个是整数类型)

`int(*p)[2][3]，p+2`的rvalue相对于p的rvalue的offset是多少？

> 算 offset 这里要考哦

### 数组与指针

对象指针总是蕴含着对数组的访问

```c
int e[2];
```

![截屏2025-11-18 17.24.03](./images_notes/截屏2025-11-18%2017.24.03.png)

`e[0]` 是后缀表达式 (也是 lvalue 表达式), 有 2 个基础表达式: `e` 和 `0` . 定位数组中的第 0 个对象

不能 `e++`, 因为 `e` 是不可修改左值 (因为 Obj_T 是数组类型) . 但是:

```c
int* p = e; 
p++;
```

是可以的, 因为 `p` 的 Obj_T 是 `int*`

![截屏2025-11-18 17.26.50](./images_notes/截屏2025-11-18%2017.26.50.png)

- 对 `=` 有了新的理解, `=` 是把一个 Value 拷贝到另一片内存区域 (🌻 wow)
