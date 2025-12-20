# 结构体、限定类型

## 结构体对齐

![截屏2025-11-21 22.23.11](./images_notes/截屏2025-11-21%2022.23.11.png)

- offset 补到能被**对象**的 alignment 整除
- 结构体最终的 offset 也要补到能被**结构体类型**的 alignment 整除

### pragma

![截屏2025-11-21 22.34.20](./images_notes/截屏2025-11-21%2022.34.20.png)

- pragma 的用法: 不要用

## 限定类型在函数中的作用

### const

```c
void func(int const* p);
```

- 当数组对象作为参数时, 用 `const` 限定一下更好

### restrict

restrict 限定符只能用来修饰指针类型, 大概是要求两块内存区域不重叠

> 指针总是蕴含着数组的访问

## Storage-class specifiers

4 种存储周期: static, thread, automatic, allocated

- static: Its lifetime is the entire execution of the program and its stored value is initialized only once, prior to program startup
- automatic: An object whose identifier is declared with no linkage and without the storage-class specifier static has automatic storage duration

Storage-class specifiers: auto, constexpr, extern, register, static, thread_local

- 这些 specifiers 规定了标识符的不同属性, 比如 scope、name space、linkage

![截屏2025-11-22 09.50.56](./images_notes/截屏2025-11-22%2009.50.56.png)


### identifier 

标识符, which designates one or more entities

- 对象标识符
- 函数标识符

什么时候 identifier 能重名?

- 同一个标识符, 如果指代不同的实体时, 这些实体要么处于不同的 scope, 或者不同的 name space

### scope

scope: 一共有 4 种 scope

- function: 只有 label name (用于 goto) 拥有 function scope
- function prototype: 函数 declarations 里的变量拥有 function prototype scope (注意 definition 的变量不是)
- file: 全局变量
- block: `{}` 内或 definition 的函数参数

### name space 

包括 label name, tag, member of structures or unions, ordinary, 

### linkage

相同的标识符出现在同一个地方?

static 就自己用, 外部看不见

```c
int main() {
  int a;
}
```

这里 `a` 是 no linkage object i...

### register

编译器可以不理会

register 修饰的对象不能 &a