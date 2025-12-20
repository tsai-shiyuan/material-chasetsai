# 常量

## 函数调用

函数调用也是一个表达式 (后缀表达式), 函数调用这个表达式 evaluate 之后的 rvalue 类型就是函数类型中的**返回值类型**;

函数调用时, 所有实参也都需要按语法规定进行 evaluate;

参数传递的到底是什么?

### 函数调用时参数 Pass by Value

传递的是**实参 evaluate 后的 rvalue**, 当作 initializer 写入形参对象

- 形参中 `int[]` 等价于 `int *` , 注意这里的 `int[]` 不是不完全数组类型

### 二维数组的形参

```c
void foo(int* g, int row, int col);

int main() {
	int g[2][3] = {0};
	foo((int*) g, 2, 3);
	return 0;
}
```

- 这样就避免了传递数组的维数

![截屏2025-11-21 14.39.37](./images_notes/截屏2025-11-21%2014.39.37.png)

![截屏2025-11-21 14.40.46](./images_notes/截屏2025-11-21%2014.40.46.png)

## 结构体

结构体/联合体类型是派生类型, 也是**非数组类型** (意味着 V_T 就是 Obj_T)

![截屏2025-11-21 14.45.18](./images_notes/截屏2025-11-21%2014.45.18.png)

### 结构体作为函数参数

![截屏2025-11-21 14.48.51](./images_notes/截屏2025-11-21%2014.48.51.png)

- 还是 pass by value

### 结构体 lvalue

左值包含:

1. 对象标识符 (identifier)
2. 字符串字面量
3. `*exp` (指针取值)
4. `exp1[exp2]`
5. `(Type-name){Initializer-list}` (compound literal)
6. `exp.member` / `exp -> member`

`foo()` 不是左值, 但是可以 `foo().a` , 这是 C 中的特例

## 变长数组 (Variable Length Array)

```c
int m;
scanf("%d", &m);
int a[m];
```

```c
int m = 1;
int a[m];
```

对象 `a` 的类型为 `int[m]`

### 变长数组类型

变长数组类型 `T[M]` 

- 变长数组是数组类型, 元素类型为 `T` , 元素个数是表达式 `M`
- ( `M` 不是一个**整数常量 / 整数常量表达式**) 或者 (T 是一个变长数组)
- 数组大小在运行时才能确定, 首次执行到 `T[M]` 时大小确定, 之后不能修改

## 常量相关

### 5 种常量

![截屏2025-11-21 15.08.32](./images_notes/截屏2025-11-21%2015.08.32.png)

### 常量表达式

常量表达式能在编译时 evaluate, 像常量一样使用

- 常量表达式不能包含赋值、自增自减、函数调用、逗号运算符, 除非他们包含在不被 evaluate 的子表达中
- 常量表达式 evaluate 之后的 rvalue 的取值范围, 应该在该表达式 rvalue 类型的表征范围之内

### 整数常量表达式

1. 表达式 rvalue 类型为整数
2. 能在编译时进行 evaluate, 像常量一样使用

### 整数常量表达式 II

**named constant**:

1. `constexpr int n = 10;` 中的 `n` (注: `n` 为 `const int` 类型)

```c

constexpr float f = 10.0;
```

1. `enum Season {Spring, Summer}` 中的 `Spring` `Summer`
2. `true` `false` `nullptr`

compound literal constant: 被 constexpr 修饰的 compound literal

![截屏2025-11-21 15.22.34](./images_notes/截屏2025-11-21%2015.22.34.png)

## 通过 sizeof 理解 VLA

### sizeof(type-name)

- 如果 `type-name` 是 Non-VLA
	- `sizeof(type-name)` 在编译时得到 type-name 的大小
	- 返回值是整数常量
	- `type-name` 整体不做 evaluate
- 如果 `type-name` 是 VLA
	- `sizeof(type-name)` 不能在编译时得到 type-name 的大小
	- 返回值不是整数常量
	- `type-name` 子表达式需要 evaluate. 比如: `sizeof(int[++m])`

### sizeof(lvalue)

- 如果 lvalue 定位的对象类型是 Non-VLA
	- `sizeof(lvalue)` 在编译时得到 lvalue 定位对象的大小
	- 返回值是整数常量
	- lvalue 整体不做 evaluate
- 如果 lvalue 定位的对象类型是 VLA
	- `sizeof(lvalue)` 不能在编译时得到 lvalue 定位对象的大小
	- 返回值不是整数常量
	- lvalue 子表达式需要 evaluate

```c
int a[2][1+2];
sizeof(a[++i]);
```

### sizeof(non-lvalue)

如果表达式是一个 non-lvalue, 则:

- `sizeof(non-lvalue)` 返回这个 non-lvalue 做 evaluate 之后 rvalue 类型的大小
- 这个 non-lvalue 并不会真的做 evaluate

```c
int i = 1;
int a[2][5];
sizeof(a[++i]+1);
```

- 比如这里 `a[++i]+1` 不做 evaluate

待定: sizeof 有的时候是编译时算, VLA 运行时 (因为不是整数常量表达式)

### alignof(type-name)

`alignof(type-name)` 是整数常量表达式, `alignof(operand)` 返回的就是 operand 的对齐要求, 返回值是一个整数常量表达式, operand 不做 evaluate

### sizeof 用于 char

```c
sizeof(char);	// 1
char c = 'a';
sizeof(c);		// 1
sizeof(c++);	// 1
sizeof('a');	// 4
```

- `'a'` 是 int 类型

## 理解 size, padding, alignment

### size

自己写程序要写 `int32_t` , 不要直接 `int` `short`

### alignment

- 对象的首地址必须是对齐值的倍数, 对齐值必须是 $2^n$
- 通过 `alignof(T)` / `_Alignof(T)` 可以获得对象类型 T 的 alignment

### `_Alignas`

由 C 标准规定编译器必须支持的对齐要求叫作 fundamental alignment, `fundamental alignment <= _Alignof(max_align_t)`

- `max_align_t` 是一个类型, 拥有最大的基础对齐要求 (一般是 8 或 16)

为对象设置更大的对齐要求: `alignas(N) T O` 或 `_Alignas(N) T O`
- 比如: `_Alignas(64) int a`
- `_Alignof(a)` 返回 `<64, size_t>`

### 结构体的对齐要求

`T O` 中如果 `T` 是结构体类型, 成员对象分别为 $E_i$, 那么 $\text{\_Alignof(T)} = \max\{\_ \text{Alignof}(E_{i})\}$

比如:

```c
struct stru {
	char a;
	_Alignas(32) short b;
}s;
```

`_Alignof(struct stru)` 返回 `<32, size_t>`

### 修改结构体对象的对齐要求
![截屏2025-11-21 17.36.25](./images_notes/截屏2025-11-21%2017.36.25.png)
