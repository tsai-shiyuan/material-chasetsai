# Bootstrap

>引用框内是GPT的回答~
>

## 启动概述

> 操作系统的 boot 过程是一个复杂繁琐的过程，bios 从上电后的启动地址开始执行，初始化硬件，读取磁盘的主引导记录，跳转到 bootloader，加载内核程序，跳转到操作系统入口……

一般都会将硬件初始化的相关工作作为“bootloader”放在非易失存储器中 (ROM或FLASH)，而将操作系统内核放在磁盘中。bootloader在硬件初始化完成后，将内核镜像从存储器读到RAM中。将CPU的控制权转交给内核。

实验编写的操作系统代码在 Linux 环境中通过 Makefile 来组织，通过交叉编译 (编译和运行分别在不同的体系架构上) 产生可执行文件，最后使用 QEMU 模拟器运行该可执行文件，实现 MOS 操作系统的运行。QEMU 模拟器支持直接加载 ELF 格式的内核，也即，在 MOS 操作系统的运行第一行代码前，我们就已经拥有一个正常的程序运行环境。

Lab1的启动流程就被简化为加载内核到内存，之后跳转到内核的入口。

## 内核的入口

怎么确定我们MOS操作系统内核代码在内存中存放的位置呢？内核入口的设置在 `kernel.lds` 中，这是linker script，指导链接器将多个 .o 文件链接成目标可执行文件。kernel.lds的开头:

```c
OUTPUT_ARCH(mips)
ENTRY(_start)
```

`OUTPUT_ARCH(mips)` 设置了最终生成文件采用的架构，`ENTRY(_start)` 设置程序入口为 `_start`，对应`init/start.S` 中的:

```assembly
.text
EXPORT(_start)
.set at
.set reorder
```

`EXPORT` 是一个宏，该宏将符号设置为全局符号，这样才对链接器可见。

```assembly
#define EXPORT(symbol) \
	.globl symbol; \
	symbol:
```

回到 `kernel.lds` ，其定义了各个段的位置。`.` 表示当前地址。

```c
SECTIONS {
    /* Step 1: Set the loading address of the text section to the location counter ".". */
	. = 0x80020000;			
	.text : { *(.text) }	/* Step 2: Define the text section. */
	.data : { *(.data) }	/* Step 3: Define the data section. */
    
	bss_start = .;
	.bss : { *(.bss) }		/* Step 4: Define the bss section. */
	bss_end = .;
	. = 0x80400000;
	end = . ;
}
```

`.text` 会从0x80020000开始，`.bss : { *(.bss) }` 表示将**所有**输入文件的.bss节都放到输出的.bss节的位置。

> 这些设置的依据是什么呢？实际上只是人为的规定。在裸机上，我们事先规定好了不同区域的内存用于何种功能 **(OS的内存位置有讲究，详见“MIPS内存布局”)**

内存布局图可在 `include/mmu.h` 中找到:

<img src="./images_report/%E6%88%AA%E5%B1%8F2025-03-16%2015.00.50.png" alt="截屏2025-03-16 15.00.50" style="zoom:33%;" />

## 编译内核

准备好了内核代码和链接脚本等文件后，我们就可以将我们的内核编译成可执行文件了。

我们通过阅读Makefile来看内核是怎么生成的，顶层的Makefile如下:

```makefile
include include.mk

target_dir              := target
mos_elf                 := $(target_dir)/mos
user_disk               := $(target_dir)/fs.img
empty_disk              := $(target_dir)/empty.img
qemu_pts                := $(shell [ -f .qemu_log ] && grep -Eo '/dev/pts/[0-9]+' .qemu_log)
link_script             := kernel.lds

modules                 := lib init kern	# 需要生成的子模块
targets                 := $(mos_elf)
# 编译出内核所依赖的所有目标文件
objects                 := $(addsuffix /*.o, $(modules)) $(addsuffix /*.x, $(user_modules))

$(modules): tools
        $(MAKE) --directory=$@
# 调用链接器 $(LD) 链接所有目标文件
$(mos_elf): $(modules) $(target_dir)
        $(LD) $(LDFLAGS) -o $(mos_elf) -N -T $(link_script) $(objects)
```

>`:=` 是赋值符号
>
>modules包含 `lib` `init` `kern` ，他们的Makefile的生成规则为 (以lib为例):
>
>```makefile
>lib: tools
>	$(Make) --directory=$@
>```
>
>- \$(MAKE) 是一个特殊变量表示调用 `make` 本身，即等价于 `make`
>
>- --directory=\$@ 等价于 -C \$@， 表示进入规则当前的目录，即 `cd lib && make`

LD等变量来自 include.mk (主要设置每次实验需要构建的目标文件)。`lib` 目录存放常用库函数，`kern` 目录中存放内核的主体代码，`init` 目录则负责初始化内核。

## 内核初始化

kernel.lds中设置了内核的入口，即 `_start` 函数，那我们来看看初始化时进行了哪些工作。`init/start.S` 主要内容如下:

```assembly
/* disable interrupts */
mtc0    zero, CP0_STATUS
la sp, KSTACKTOP
j mips_init
```

可以发现，我们初始化了 `sp` 寄存器的地址，最后，跳到 `mips_init` 函数，其在 `init/init.c` 中定义:

```c
void mips_init(u_int argc, char **argv, char **penv, u_int ram_low_size) {
	printk("init.c:\tmips_init() is called\n");		// 到这里, lab1的小内核就完成啦
}
```

## 实现printk()

在 `kern/printk.c` 下有 `printk()` 的定义:

```c
void printk(const char *fmt, ...) {		// ... 表示其余参数
	va_list ap;
	va_start(ap, fmt);	// 处理变长参数
	vprintfmt(outputk, NULL, fmt, ap);		// 格式化输出
	va_end(ap);
}
```

> **NOTE｜C中的可变长参数**
>
> `stdarg.h` 头文件为处理变长参数表定义了一组宏:
>
> - `va_list` 变长参数表的变量类型
> - `va_start(va_list ap, lastarg)` 用于初始化变长参数表
> - `va_arg(va_list ap, 类型)` 取变长参数表下一个参数
> - `va_end(va_list ap)` 结束使用变长参数表
>
> 看个例子:
>
> ```c
> va_list ap;		// 先声明va_list类型的变量ap
> va_start(ap, lastarg);	// 然后用va_start初始化,lastarg为该函数最后一个命名的形式参数
> int num;
> num = va_arg(ap, int);	// 按照参数类型获取一个形参
> va_end(ap);		// 结束使用
> ```
>

关键在第4行的 `vprintfmt` 函数。第一个参数 `outputk` 是一个回调函数，其定义同在 `kern/printk.c`

```c
void outputk(void *data, const char *buf, size_t len) {
	for (int i = 0; i < len; i++) {
		printcharc(buf[i]);
	}
}
```

`printcharc()` 的作用肯定是输出一个字符，更深入一层，在 `kern/machine.c` 中给出了他的定义:

```c
void printcharc(char ch) {
	*((volatile uint8_t *)(KSEG1 + MALTA_SERIAL_DATA)) = ch;
}
```

输出字符的本质是对内存写一个字节。

我们继续分析 `vprintfmt` 函数，他定义在 `lib/print.c`。首先定义一些变量:

```c
void vprintfmt(fmt_callback_t out, void *data, const char *fmt, va_list ap) {
	char c;
	const char *s;
	long num;

	int width;
	int long_flag; // output is long (rather than int)
	int neg_flag;  // output is negative
	int ladjust;   // output is left-aligned
	char padc;     // padding char,填充多余位置所用字符
```

然后解析输出含有 `%` 的 `fmt` 字符串。核心思路是遍历 `fmt` ，找到 `%` 并按照格式进行参数替换。

格式串为 `%[flags][width][length]<specifier>` ，参数的含义如下

- flags: "-"左对齐，"0"填充0
- width: 打印数字的最小宽度
- length: 修改数据类型的长度
- specifier: 输出变量的类型

## QEMU运行内核

QEMU可以模拟各种CPU (如 x86, RISC-V, ARM)，提供虚拟的**CPU, 内存, I/O** 等，使得内核像在真实计算机上运行一样。那么，QEMU怎么把我们的MOS可执行文件 (也就是MIPS结构下的指令) 放到这些虚拟的硬件之中的呢？怎么读懂我们的MOS可执行文件的？

答案就是，我们的MOS内核可执行文件是**ELF**格式的，QEMU可以运行ELF格式的内核！

ELF (Executable and Linkable Format) 是一种用于可执行文件、目标文件和库的文件格式。.o文件属于可重定位 (relocatable) 文件。

> `file` 可以获得文件的类型，例如:
>
> ```bash
> $ file lib/print.o
> lib/print.o: ELF 32-bit LSB relocatable, MIPS, MIPS32 version 1 (SYSV), with debug_info, not stripped
> $ file lib/print.c
> lib/print.c: C source, ASCII text
> ```

ELF文件的结构为:

![截屏2025-03-18 19.58.37](./images_report/%E6%88%AA%E5%B1%8F2025-03-18%2019.58.37.png)

>段segment: 告诉操作系统如何将文件的内容映射到进程的地址空间，比如 .text 代码段
>
>节section: 编译链接时按节定位

QEMU解析ELF可执行文件，加载到虚拟内存，并将**程序入口点 (entry point)**设置为CPU的起始执行地址。然后顺利模拟运行。