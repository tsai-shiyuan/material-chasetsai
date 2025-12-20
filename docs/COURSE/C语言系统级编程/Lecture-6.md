# 表达式副作用

## volatile

### `int volatile` 的含义

```c
int volatile a = 10;
```

- 代码没有改 `a` , 但是发现 `a` 变成 `20` 了? 是谁改的呢?

### MMIO

Memory-Mapped Input / Output (MMIO) : 给 I/O 设备流出内存空间

```c
#define MYNUM (*(int volatile*)0x12340000)
```

- `MYNUM` 是左值, 对象类型是 `int volatile` , 就意味着这段内存会被我们未知的方式修改 (可能是传感器去改的)

### volatile 对象的 evaluate

如果要 evaluate, 都必须去访问对应的内存

> 也好理解, 因为在寄存器的值和内存可能不一样了

## Evaluation of Expression

1. Value computation 求值, 返回 `<V, V_T>`
2. Initiation of side effect 副作用

![截屏2025-11-20 16.51.46](./images_notes/截屏2025-11-20%2016.51.46.png)

1. `a=b` 和 `b` 的求值是否有先后顺序?
	1. 有, An assignment expression has the value of the left operand **after** the assignment
2. `a` 的值什么时候改的?
	1. The side effect of updating the stored value of the left operand is **sequenced after the value computations** of the left and right operands.
	2. 在这里当 `b` 求完值就可以改了

### Sequenced Before 

![截屏2025-11-20 17.00.02](./images_notes/截屏2025-11-20%2017.00.02.png)

## 函数

函数七元组:

- value 是函数的地址
- value type 是函数类型的指针类型
- 函数没有对齐要求

Function Designator

1. Function identifier (函数标识符)
2. 如果一个表达式 exp, 其 rvalue 类型是函数指针类型, 则 `*exp` 是一个合法的 Function Designator

