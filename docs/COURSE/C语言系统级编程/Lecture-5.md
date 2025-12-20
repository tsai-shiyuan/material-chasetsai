# 数组

## 数组

![截屏2025-11-19 14.34.33](./images_notes/截屏2025-11-19%2014.34.33.png)

![截屏2025-11-19 14.38.34](./images_notes/截屏2025-11-19%2014.38.34.png)

![截屏2025-11-19 14.38.40](./images_notes/截屏2025-11-19%2014.38.40.png)

## malloc

### malloc 语法

![截屏2025-11-19 15.21.36](./images_notes/截屏2025-11-19%2015.21.36.png)

- 这个对象**没有对象类型**

![截屏2025-11-19 15.22.49](./images_notes/截屏2025-11-19%2015.22.49.png)

- `max_align_t` (要考!) 
- fundamental alignment 是为了保证之后的转换满足别的类型的对齐要求

`malloc` 的返回值需要强制转换成有意义的对象类型

![截屏2025-11-19 15.29.11](./images_notes/截屏2025-11-19%2015.29.11.png)

- `malloc(4)` 是后缀表达式 (函数调用), `(int*)` 是 cast 表达式
- 分配的对象的 alignment 是 fundamental alignment, 但是从 `*p` 的视角看是 4 (因为是 `int`)
- 不要写 `4` , 而写 `sizeof(int)`

### `malloc(sizeof(Obj_T) * N)`

![截屏2025-11-19 16.00.50](./images_notes/截屏2025-11-19%2016.00.50.png)

malloc 申请 N 个 Obj_T 大小的内存可以形式化定义为:

```c
Obj_T* p = (Obj_T*) malloc(sizeof(Obj_T) * N);
```

- 可以视为分配了一个 `Obj_T[N]` 对象类型空间

`int* p = (int*) malloc(sizeof(int))` 其实等价于:

```c
int* p = (int*) malloc(sizeof(int) * 1);
```

- 蕴含着分配了一个 `int[1]` 类型的空间

### malloc 分配高维数组

![截屏2025-11-19 16.07.46](./images_notes/截屏2025-11-19%2016.07.46.png)

- 把第一块 `int[3]` 的首地址存到了 `ppArray[0]` 里
- 平时的 `int[2][3]` 占据 `4*2*3=24` 个字节, 这里还要多消耗 `4*2=8` 个字节

### 思考题

(1) 利用 `malloc` 分配 240 个字节, 让该 240 字节类型视为 `int[6][10]`

```c
int(*p)[10] = (int(*)[10])malloc(sizeof(int[10])*6);
```

(2) 

```c
void* p = malloc(32);
int (*s)[4] = (int(*)[4])p;
```

假设 `p` 的值是 `Add` , `s+1` 的值是多少?

- `s` 的 Obj_T 是 `int (*)[4]` , `sizeof(*s) = 4*4 = 16` , 所以是 `Add + 16`

### 指针?

`<address, pointer type>` 这个 `<rvalue, rvalue type>` 就是**指针**. 也就是说, pointer 是表达式 evaluate 后得到的

## const 限定符

`const` 限定数组类型时, 限定的是其元素类型

![截屏2025-11-19 16.41.30](./images_notes/截屏2025-11-19%2016.41.30.png)

![截屏2025-11-19 16.42.52](./images_notes/截屏2025-11-19%2016.42.52.png)

## 字符串不是常量!

```c
"hello"[0] = 'a';
```

`"hello"` 是不可修改的 lvalue, 对象七元组: `<char[6], char*, addr>`

`"hello"[0]` 是可修改 lvalue, 对象七元组: `<char, char, 'h'>` , 可以修改, 但是是 UB