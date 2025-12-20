# C语言复习笔记

## 1 控制流

### if-else语句

```c
if (表达式) {
  	语句块1
}
else if (表达式) {
  	语句块2
}
else {
  	语句块3
}
```

条件表达式表示相等关系用 `==` 而非 `=`

默认 `else` 与最近的 `if` 相配

### 条件运算符

```c
condition? expression1: expression2;
```

当 `condition` 为真，执行 `expression1` ；否则执行 `expression2`。并将 `expression` 的值作为整个表达式的返回值

### switch语句

```c
switch (表达式) {
  	case 常量表达式1: 语句序列;
    case 常量表达式2: 语句序列;
  	default: 语句序列;
}
```

计算表达式的值，测试是否与常量整数相配，执行分支动作

`break` 使程序跳出switch语句

### continue

```c
for (x=1; x<10; x++) {
  	if (x==5)
      	continue;
  	printf("%d", x)
}
```

- 当 `x==5` 时，不执行 `printf` ，直接 `x++`

- `while` 语句 `continue` 直接到 `while ( 条件 )` ，可能死循环

## 2 函数

### 函数声明

```c
//<返回值类型> <函数名> (<参数类型><标识符>)
double atan2(double y, double x);
```

声明必须在调用之前完成，且不能放在其他函数体内

### 函数定义

```c
<返回值类型> <函数名> (<参数类型><标识符>) 
{
  	<函数体>
}
```

### 模块化编程

函数的形参与实参

- 形参：函数定义中的参数
- 实参：函数调用中的参数

参数的传递方式：**值传递**，将实参的值复制给形参，二者相互独立，不共享内存单元

### 空间域

`{}` 括住的就是变量的空间域（作用域）

变量重名时，空间域小的变量起作用

### 时间域

```c
<存储类型> <数据类型> <变量名>
```

存储类型决定变量的存储位置和存在状态

常用4种存储类型：

- auto
- register
- static
- extern

### 全局变量

全局变量是定义在 `main` 函数之外的变量，作用于整个程序

## 3 数组

### 二维数组

```c
<类型> <数组名> [<行数>][<列数>]
```

C语言采用行优先存储

`a[i][j]=str[i*n+j]` ，其中 `n` 为列数

### 字符串

字符数组以 `\0` 结尾形成字符串

只有初始化时可以给字符数组整体赋值，例如：

```c
char str[15]={"hello"};
char str[15]="hello";
char str[]="hello";
char str[15]={'h','e','l','l','o'}
```

`"hello"` 本身为指向字符串 `hello` 的指针，`"hello"[0]` 为 `'h'`

### gets, scanf, fgets

`gets(s)` 读到 `\n` 停止，并自动补 `\0` ，不会补 `\n`

`scanf("%s",s)` 以空格结束，`scanf("%[^\n]",str)` 以 `\n` 结束，不包括 `\n`

`fgets` 自动补 `\n`

### 常用的字符串处理函数

```c
#include <string.h>
puts(s);    //输出字符串, 自动换行
gets(s);    //读入字符串
fgets(s,n,stdin);    //限制读n个字符, 自动补\n
fputs(s,stdout);    //不会换行
strcat(s1,s2);    //将s2追加到s1后面, 并返回s1地址
strncat(s1,s2,n);    //将s2最多前n个字符追加到s1后面
strcpy(s1,s2);    //将s2复制给s1
strncpy(s1,s2,n);   //将s2的最多n个字符复制给s1
strcmp(s1,s2);    //字典序s1>s2, 返回正数
strlen(s);    //求字符串长度, 不包括\0
strstr(s1,s2);    //确认s2是否在s1中出现过, 是则返回第一次出现的位置, 否则返回NULL
strchr(str,val);	//在str中查找ASCII码为val的字符，找到则返回指向字符的指针
sprintf(字符数组名, "输出格式", 变量列表)
sscanf(字符数组名, "输入格式", 变量列表)
```

## 4 指针

### 指针与地址

指针是一种数据类型

指针变量是具有指针类型的变量，在数据类型前加 `*` ，就构成了指针类型。存储的内容永远是地址

```c
int x = 7;
int *p = &x;
```

- `x` 是 `int` 型，`&x` 返回的地址是 `int *` 型，可以直接赋给同类型的指针变量 `p`

- 变量的访问
  - 变量名直接访问
  - 通过指向变量的指针间接访问

```c
*p = 3;
```

`*p` 是指针 `p` 所指的值, 此语句使得变量 `x` 的值被更改为 `3`

### 指针与数组

数组名作为表达式可隐式转换为指向数组首元素的指针

`p[i]` $\iff$ `*(p+i)` 

注意数组名是常量

### const修饰

- 常量指针
  - 指向常量
  - `const char* p` 

- 指针常量
  - 指针本身不能改变指向
  - `char* const p;`
- 常量指针常量
  - `const char* const p`

### 字符串与char*型指针

```c
char s[20] = "point to me";
char *p = s;	//p指向字符串
s = "point to u";  //错误
p = "point to u";	 //正确，改变p的指向
```

### 指针与函数

- 希望函数返回多个值，可用指针作形参

### 优先级

`()`  `[]`  `.`  `->` 优先级最高

解引用 `*` = 自加/减 > 加减 ，相等时从右至左结合

`a = (*p)++` $\iff$​​ `a = *p; a++` 

`a = *p++` $\iff$ `a = *p; p++`

`a = *++p` 先对p自增，再取内容

### sprintf

将变量“输出”到字符串中（而非屏幕上）

例如：计算 `sin(x)` 并保留 `m` 位小数 

```c
int m; double x; char format[32];
scanf("%lf%d", &x, &m);
sprintf(format, "%%.%df\n", m);
printf(format, sin(x));
```

### sscanf

将字符串按指定格式读入

```c
int d, y ,h, m, s;
char mon[4], zone[6];
char buf[] = "17/Apr/2018:10:28:28 +0800";
sscanf(buf, "%d/%3c/%d:%d:%d:%d %s", &d, mon, &y, &h, &m, &s, zone);
mon[3] = '\0';
```

### 数组指针

```c
int a[10];
int (*p)[10] = &a;
```

`a` 等价于 `&a[0]` ，是 `int*` 型指针

`&a` 是 `int *[10]` 型指针，指向数组，加1移过整个数组，即10个 `int`

`p` 是 `a` 的地址，`p[0]` 是首元素的地址，`*p` 是数组 `a`

### 指针数组

重心在数组，元素类型为指针的数组

```c
char *PArr[20];
```

### 指针与二维数组

```c
int a[4][5];
```

`*a` 指向 `a[0][0]` ，是 `int*` 型

### 函数指针

定义: `类型 (* 指针变量名)(参数类型);`

```c
double (* funPtr)(int, int);
```

定义了一个函数指针 `funPtr`，指向的函数原型为 `double fun(int, int)`  

指向函数的指针指向函数的入口地址

给函数指针变量赋值时, 只需给出函数名

调用方式: `(* 函数指针变量名)(实参表)` `函数指针变量名 (实参表)`

### 返回指针值的函数

```
类型标识符 * 函数名 (形参表){...}
```

## 5 动态内存申请与释放

### malloc

`<stdlib.h>`

原型 `void *malloc(size_t);` 接收申请内存空间的字节数，返回指向这片空间的指针

### free

原型 `void free(void * _Memory);`

## 6 位运算

- `&` 按位与
- `|` 按位或
- `^` 按位异或
- `~` 按位取反
- `<<` 左移
- `>>` 右移

### 优先级

`!(逻辑非)` > `按位取反 ~` >`算术运算符` > `左移运算符 <<` `右移运算符 >>` > `关系运算符` > `按位与 &` `按位异或 ^` `按位或 |`> `逻辑与 &&` >`逻辑或 ||` > `赋值运算符`

### 计算

```c
unsigned int a, b;
unsigned int plus;
plus = ((a & b) << 1) + (a ^ b);	//a+b
```

## 7 结构体

### 定义

类型定义：

```c
struct Person {
  	char name[20];
  	int age;
  	double height;
};
```

变量定义：

```c
struct Person student1;
```

也可以一起定义：

```c
struct Person {
  	...
} student1;
```

### 指针

`struct Person *p`

引用成员：`(*p).age` 或 `p->age`

### 结构体快排

```c
struct Person students[50];
qsort(students, n, sizeof(struct Person), cmp);
```

### 成员的对齐

原理较复杂，总之尽量将占内存小的成员放在前

### 自定义类型名

`typedef 原类型名 新类型名;`

### 联合union

联合类型的变量在不同时刻维持定义的不同类型和不同长度的对象

```c
union 联合体名
{成员表};
```

联合体 `union` 和 `struct` 的定义和使用是相同的，但 `union` 中的各个数据之间共享同一个单元的，所占内存单元的大小是成员中长度最大的一个。

### 自引用结构

数据+指向自身结构的指针

### 枚举类型

## 8 输入与输出

### main函数的形参

```c
int main(int argc, char *argv[])
{...}
```

`argc` 是包含命令在内的参数个数

`argv` 是字符型指针数组, 指向每一个参数字符串，`argv[0]` 是命令本身

### 命令行之tail

显示文件的最后n行：`tail [-n] filename`

### 打开文件

```c
FILE *in, *out;
in = fopen("input.txt", "r");
out = fopen("output.txt", "w");
```

- `r` 只读
- `w` 只写。为写创建一个新文件，若指定的文件已存在，则其中原有内容被**删去**
- `a` 追加写。向文件尾增加数据。若指定的文件不存在，则创建一个新文件
- `r+` 读写。为读写打开一个文件。若指定的文件不存在，则返回NULL
- `w+` 读写。为读写打开一个新文件。若指定的文件已存在, 则其中原有内容被删去
- `a+` 读与追加写。为读写向文件尾增加数据打开一个文件，若指定的文件不存在，则创建一个新文件

### 读写文件

```c
c = fgetc(in);  //从文件中读入一个字符
fputc(c, out);  //输出一个字符到out文件
fgets(s, n, in);
fputs(s, out);
fscanf(in, "%d", &score);  //从in中读一个整数
fprintf(out, "%d", score);  //输出到out
```

### 关闭文件

```c
fclose(in);
fclose(out);
```

## End 补充

### memcpy

```c
memcpy(b, a, sizeof(a));
```

将数组 `a` 中的元素全部复制到数组 `b`

`sizeof(a)` 返回 `a` 占用的空间，字节数 $\times$ 元素数

