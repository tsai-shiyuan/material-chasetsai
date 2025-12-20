# Process

## 内核初始化

### 再度mips_init

Lab3 中 init/init.c 的 `mips_init` 函数的内容变化:

```c
	env_init();
	ENV_CREATE_PRIORITY(user_bare_loop, 1);
	ENV_CREATE_PRIORITY(user_bare_loop, 2);
```

### 进程管理的数据结构

进入 `env_init` 函数，首先初始化了两个链表:

```c
void env_init(void) {
	int i;
	LIST_INIT(&env_free_list);
	TAILQ_INIT(&env_sched_list);
```

env.h:

```c
LIST_HEAD(Env_list, Env);
TAILQ_HEAD(Env_sched_list, Env);
```

根据 Tail queue definitions:

```c
#define _TAILQ_HEAD(name, type, qual)	\
	struct name {	\
		qual type *tqh_first;	// first element	\
		qual type *qual *tqh_last;	// addr of last next element	\
	}
#define TAILQ_HEAD(name, type) _TAILQ_HEAD(name, struct type, )
```

展开得:

```c
struct Env_list {
    struct Env *lh_first;
};

struct Env_sched_list {
    struct Env *tqh_first;
    struct Env *tqh_last;
};
```

`Env` 结构体就是进程控制块 (Process Control Block, PCB)

```c
struct Env {
	struct Trapframe env_tf;	 // saved context (registers) before switching
	LIST_ENTRY(Env) env_link;
	u_int env_id;			 // unique environment identifier
	u_int env_asid;			 // 用于TLB,虚拟地址空间的标识
	u_int env_parent_id;	 // env_id of this env's parent
	u_int env_status;		 // 当前状态
	Pde *env_pgdir;			 // 当前进程拥有的页目录的虚拟地址
	TAILQ_ENTRY(Env) env_sched_link; // intrusive entry in 'env_sched_list'
	u_int env_pri;			 // schedule priority
	...
};
```

- `env_link` 和 `env_sched_link` 存储链表信息，展开后为

  ```c
  struct {
      struct Env *le_next;
      struct Env **le_prev;
  } env_link;
  
  struct {
  	struct Env *tqe_next;
      struct Env *tqe_prev;
  } env_sched_link;
  ```

- `env_tf` ，`Trapframe` 定义在 include/trap.h 中。在发生进程调度，或当陷入内核时，会将当时的进程上下文环境保存在 env_tf 变量中

  ```c
  struct Trapframe {
  	/* Saved main processor registers. */
  	unsigned long regs[32];
  
  	/* Saved special registers. */
  	unsigned long cp0_status;
  	unsigned long hi;
  	unsigned long lo;
  	unsigned long cp0_badvaddr;
  	unsigned long cp0_cause;
  	unsigned long cp0_epc;
  };
  ```

- `env_status` 存储进程的当前状态，包括空闲（没有被任何进程使用）、可运行（就绪/正运行）、阻塞

  ```c
  #define ENV_FREE 0
  #define ENV_RUNNABLE 1
  #define ENV_NOT_RUNNABLE 2
  ```

存放进程控制块的物理内存在系统启动后就已经分配好，就是 envs 数组。

```c
struct Env envs[NENV] __attribute__((aligned(PAGE_SIZE))); // All environments
```

我们用链表来管理进程控制块数组。`env_free_list` 存储了所有空闲的进程控制块，而 `env_sched_list` 列表用于组织进程调度。

```c
static struct Env_list env_free_list; // Free list
extern struct Env_sched_list env_sched_list; // runnable env list
```

`env_init` 后续会将所有的进程控制块都插入 `env_free_list` 中。并且标记所有块都为 `ENV_FREE`

```c
    for (i = NENV - 1; i >= 0; i--) {
        LIST_INSERT_HEAD(&env_free_list, envs + i, env_link);	// 插入空闲块链表
        envs[i].env_status = ENV_FREE;		// 控制块标记为Free
    }
```

### 模板页目录

在 `env_init` 的最后，我们还需要做一件事，就是创建一个 “模板页目录”，设置该页将 pages 和 envs (即所有页控制块和所有进程控制块的内存空间) 分别映射到 `UPAGES` 和 `UENVS` 的空间中 (即下图的pages和envs地带)。并且在后续进程创建新的页目录时，也要首先复制模板页目录中的内容。这样做的目的是使得用户程序也能够通过 `UPAGES` 和 `UENVS` 的用户地址空间获取 `Page` 和 `Env` 的信息。

```c
    struct Page *p;
    panic_on(page_alloc(&p));
    p->pp_ref++;	// 该页即为"模版页目录"

    // map_segment(Pde *pgdir, u_int asid, u_long pa, u_long va, u_int size, u_int perm);
    base_pgdir = (Pde *)page2kva(p);
    map_segment(base_pgdir, 0, PADDR(pages), UPAGES, 
                ROUND(npage * sizeof(struct Page), PAGE_SIZE), PTE_G);
    map_segment(base_pgdir, 0, PADDR(envs), UENVS,
                ROUND(NENV * sizeof(struct Env), PAGE_SIZE), PTE_G);
```

![截屏2025-04-19 13.21.06](./images_report/%E6%88%AA%E5%B1%8F2025-04-19%2013.21.06.png)

只设置了 `PTE_G` ，用户只有“只读”权限

### 段地址映射

在页目录 `pgdir` 中，将虚拟地址空间 `[va, va+size)` 映射到到物理地址空间 `[pa, pa+size)`，并赋予 `perm` 权限。因为是按页映射，要求 size 必须是页面大小的整数倍。

```c
static void map_segment(Pde *pgdir, u_int asid, u_long pa, u_long va, u_int size, u_int perm) {
```

具体实现方法是不断调用 `page_insert` 创建一个页大小的 `va` 到 `pa` 的映射，直到达到期望的 `size` 大小

```c
	for (int i = 0; i < size; i += PAGE_SIZE) {
		page_insert(pgdir, asid, pa2page(pa + i), va + i, perm);
	}
```

## 进程的创建概览

### 进程的标识

struct Env 进程控制块中的 `env_id` 域，是每个进程独一无二的标识符。env.c 中 `mkenvid` 函数的作用是生成一个新的进程 env_id，在进程创建的时候会调用该函数。

```c
u_int mkenvid(struct Env *e) {
	static u_int i = 0;
	return ((++i) << (1 + LOG2NENV)) | (e - envs);	// define LOG2NENV 10
}
```

PCB还有一个env_asid域，TLB存储进程的 ASID，用于区别不同的虚拟地址空间中的映射 (因为有进程的标识，切换页表的时候不用清空TLB)。ASID有唯一性，直到进程被销毁或 TLB 被清空时，其 ASID 才可以被分配给其他进程。MIPS中ASID占8 bit，使用位图法管理256个可用的ASID (还是很精巧的):

```c
static uint32_t asid_bitmap[NASID / 32] = {0};	// #define NASID 256

static int asid_alloc(u_int *asid) {
	for (u_int i = 0; i < NASID; ++i) {
		int index = i >> 5;		// 第几个32位整型数
		int inner = i & 31;
		if ((asid_bitmap[index] & (1 << inner)) == 0) {	// 有空位
			asid_bitmap[index] |= 1 << inner;
			*asid = i;
			return 0;
		}
	}
	return -E_NO_FREE_ENV;
}

static void asid_free(u_int i) {
	int index = i >> 5;
	int inner = i & 31;
	asid_bitmap[index] &= ~(1 << inner);
}
```

### 进程创建的流程

- 第一步: 申请一个空闲的 PCB，即从 env_free_list 中索取一个空闲PCB块
- 第二步: 初始化进程控制块
- 第三步: 为新进程初始化页目录，设置好进程的地址空间
- 第四步: 把PCB从空闲链表里摘出

## 创建进程

### 初始化进程控制块

通过 `env_alloc` 申请一个新的空闲进程控制块

```c
int env_alloc(struct Env **new, u_int parent_id) {
	int r;
	struct Env *e;

	if (LIST_EMPTY(&env_free_list)) {
		return -E_NO_FREE_ENV;
	}
	e = LIST_FIRST(&env_free_list);
```

接着调用 `env_setup_vm` 初始化进程控制块的用户地址空间，也就是为进程控制块创建对应的二级页表。函数的具体细节见后文。

```c
	if ((r = env_setup_vm(e)) != 0) {
		return r;
	}
```

然后我们设置进程的标识

```c
	e -> env_id = mkenvid(e);
	e -> env_parent_id = parent_id;
	if ((r = asid_alloc(&e->env_asid)) != 0) {	// 不为0,根据asid_alloc,发生异常
		return r;
	}
```

在最后，我们设置进程的 `status` 寄存器和 `sp` 寄存器的值。对于 `sp` ，在未执行的情况下，用户程序的 sp 寄存器应该处于栈顶 `USTACKTOP` 的位置。但为了给程序的 `main` 函数的参数 `argc` 和 `argv` 留出空间，需要减去 `sizeof(int) + sizeof(char **)` 的大小。

```c
	e->env_tf.cp0_status = STATUS_IM7 | STATUS_IE | STATUS_EXL | STATUS_UM;	// UM-user mode
	e->env_tf.regs[29] = USTACKTOP - sizeof(int) - sizeof(char **);	// sp
	LIST_REMOVE(e, env_link);	// #define LIST_REMOVE(elm, field)
	*new = e;
	return 0;
}
```

关注对 `cp0_status` 的赋值。IE 位表示中断是否开启，将 IE 及 IM7 设置为 1，表示中断使能，且 7 号中断 (时钟中断) 可以被响应。

![截屏2025-04-19 16.37.13](./images_report/%E6%88%AA%E5%B1%8F2025-04-19%2016.37.13.png)

那 `EXL` 和 `UM` 的设置是出于什么样的考虑呢?

在 MIPS 架构下，操作系统通过设置 Status 寄存器中的 `EXL` 和 `UM` 两个位来控制用户态和内核态之间的切换:

- **用户态:** 当且仅当 `EXL = 0` 且 `UM = 1` 时，处理器处于用户态

- **发生异常时:** `EXL` 被自动置为 1，表示处理器进入异常状态，处理器切换到内核态以执行异常处理程序

- **进程调度时的行为:** 当操作系统准备运行一个用户进程时，会执行这样的指令

  ```assembly
  RESTORE_ALL
  eret
  ```

  - `RESTORE_ALL` 是恢复现场的宏
  - `eret` 用于从异常中返回的指令。伴有副作用: 将 `EXL` 位置 `0` ，这意味着异常处理结束，处理器准备恢复到用户态

- 为什么必须在一开始的时候设置 `EXL` 位？如果不设置，`RESTORE_ALL` 的时候写入 `EXL = 0` ，处理器立即切换为用户态，但是此时还在恢复中，在访问内核资源。必须先让 `EXL = 1` 保护恢复过程，等一切恢复完成后，`eret` 再清除 EXL，安全进入用户态

### 初始化地址空间*

在 `env_alloc` 的时候，我们调用过 `env_setup_vm` 函数

```c
	if ((r = env_setup_vm(e)) != 0) {
		return r;
	}
```

首先我们申请一个物理页作为页目录，

```c
static int env_setup_vm(struct Env *e) {
	struct Page *p;
	try(page_alloc(&p));
	p -> pp_ref++;
	e -> env_pgdir = (Pde *)page2kva(p);
```

现在正在创建二级页表，我们可以将“模板页目录”中的内容复制到当前进程的页目录中

```c
	memcpy(e->env_pgdir + PDX(UTOP), base_pgdir + PDX(UTOP),
	       sizeof(Pde) * (PDX(UVPT) - PDX(UTOP)));
```

我们复制了 `UTOP` 到 `UVPT` 的虚拟地址空间对应的页表项到进程自己的页表中。这也是我们之前在 “模板页目录” 中映射的区域。

![截屏2025-04-19 13.21.06](./images_report/%E6%88%AA%E5%B1%8F2025-04-19%2013.21.06.png)

最后将 `UVPT` 虚拟地址映射到页目录本身的物理地址，并设置只读权限。这样的话，页目录中的项所对应的，就不只是二级页表，还包含有一个一级页表，也就是页目录自身。

```c
	e->env_pgdir[PDX(UVPT)] = PADDR(e->env_pgdir) | PTE_V;
	return 0;
}
```

>假设现在我们用比 `UVPT` 地址高一些的地址 `va` 进行访存，那么我们会取到那些信息呢？首先，这个地址会经过页目录，`PDX(va)` 的结果和 `UVPT` 相同，我们进入到索引对应的二级页表……不对，还是页目录自身！
>
>好的，我们在页目录中重新来一遍，这次通过 `PTX(va)` 计算索引，结果就不一定还是页目录项了。我们找到了一个物理页，取出了其中的数据。可是等等，这个物理页却不再是一般的物理页了，而是作为二级页表的物理页。
>
>另外假如我们恰好取得的 `PTX(va)` 值与 `PDX(va)` 相同，那么我们绕了两圈，最终还是处在页目录之中，我们取得的数据也是页目录中的内容。
>
>这样我们就可以明白自映射的作用了。它在用户内存空间中划分出一部分，使得用户可以通过访问这部分空间得到二级页表以及页目录中的数据。
>
>在 include/mmu.h 的内存分布图中，我们可以看出 `UVPT` 以上的 4kb（1024 个页表的大小）空间被标记为 `User VPT`。VPT 或为 virtual page table（虚拟页表）的意思。

### 加载二进制镜像

```c
static void load_icode(struct Env *e, const void *binary, size_t size);
static int load_icode_mapper(void *data, u_long va, size_t offset, u_int perm, const void *src, size_t len);
const Elf32_Ehdr *elf_from(const void *binary, size_t size);
int elf_load_seg(Elf32_Phdr *ph, const void *bin, elf_mapper_t map_page, void *data);
```

- `load_icode` 函数负责加载可执行文件 binary 到进程 e 的内存中
- `elf_from` 函数仅仅是检查是否符合ELF文件格式，并将 `binary` 以 `Elf32_Ehdr` 指针的形式返回
- `elf_load_seg` 负责将 ELF 文件的一个 segment 加载到内存。`elf_load_seg` 的最后两个参数接受一个自定义的回调函数 map_page，以及需要传递给回调函数的额外参数 data，回调函数负责单个页面的加载
- `load_icode_mapper` 是 map_page 回调函数的具体实现

`load_icode` 函数首先用 `elf_from` 解析 ELF header

``` c
static void load_icode(struct Env *e, const void *binary, size_t size) {
	const Elf32_Ehdr *ehdr = elf_from(binary, size);
	if (!ehdr) {
		panic("bad elf at %x", binary);
	}
```

然后调用 `elf_load_seg` 函数将参数指定的程序段 (program segment) 加载到进程的地址空间中

```c
	size_t ph_off;	// 宏内部会遍历更改
	ELF_FOREACH_PHDR_OFF (ph_off, ehdr) {
		Elf32_Phdr *ph = (Elf32_Phdr *)(binary + ph_off);
		if (ph->p_type == PT_LOAD) {	// 需要被加载到内存中
			panic_on(elf_load_seg(ph, binary + ph->p_offset, load_icode_mapper, e));
		}
	}
```

最后设定 trap frame 的 epc cp0 寄存器的值设置为 ELF 文件中设定的程序入口地址。当我们运行进程时，CPU 将自动从 PC 所指的位置开始执行二进制码。

```c
	e -> env_tf.cp0_epc = ehdr -> e_entry;	// ELF文件中设定的程序入口地址
```

再来看 `elf_load_seg` 函数，定义在 lib/elfloader.c 中，作用是根据程序头表中的信息将 `bin` 中的数据加载到指定位置。关注该函数的参数:

- `Elf32_Phdr *ph` 指向ELF文件段头，Program Header 包含段的虚拟地址、物理地址、大小、标志等信息
- `const void *bin` 指向ELF文件二进制代码和数据
- `elf_mapper_t map_page` 是一个回调函数，用于将数据映射到虚拟地址所在的页上
- `void *data` 回调函数的参数

```c
int elf_load_seg(Elf32_Phdr *ph, const void *bin, elf_mapper_t map_page, void *data) {
    u_long va = ph->p_vaddr;
	size_t bin_size = ph->p_filesz;
	size_t sgsize = ph->p_memsz;
	u_int perm = PTE_V;
	if (ph->p_flags & PF_W) {
		perm |= PTE_D;
	}
```

elf.h 中定义了 `elf_mapper_t`，此类型的函数接收数据要加载到的虚拟地址 `va`，数据加载的起始位置相对于页的偏移 `offset`，页的权限 `prem`，所要加载的数据 `src` 和要加载的数据大小 `len` 

```c
typedef int (*elf_mapper_t)(void *data, u_long va, size_t offset, u_int perm, const void *src, size_t len);
```

为函数原型 `int (*elf_mapper_t)(...);` 起了新名字 `elf_mapper_t` ，这是函数指针类型，以后就可以用 `elf_mapper_t f` 。

段内的内存布局可能较为复杂，elf_load_seg 只关心 ELF 段的结构，而把单页加载的细节交由回调函数处理。

![截屏2025-04-19 14.38.05](./images_report/%E6%88%AA%E5%B1%8F2025-04-19%2014.38.05.png)

在 `elf_load_seg` 中，我们首先需要处理要加载的虚拟地址不与页对齐的情况。我们将最开头不对齐的部分 “剪切” 下来，先映射到内存的页中。

```c
	int r;
	size_t i;
	u_long offset = va - ROUNDDOWN(va, PAGE_SIZE);
	if (offset != 0) {
		if ((r = map_page(data, va, offset, perm, bin,
				  MIN(bin_size, PAGE_SIZE - offset))) != 0) {
			return r;
		}
	}
```

接着我们处理数据中间完整的部分。我们通过循环不断将数据加载到页上。

```c
	/* Step 1: load all content of bin into memory. */
	for (i = offset ? MIN(bin_size, PAGE_SIZE - offset) : 0; i < bin_size; i += PAGE_SIZE) {
		if ((r = map_page(data, va + i, 0, perm, bin + i, MIN(bin_size - i, PAGE_SIZE))) !=
		    0) {
			return r;
		}
	}
```

最后我们处理段大小大于数据大小的情况。在这一部分，我们不断创建新的页，但是并不向其中加载任何内容。

```c
	/* Step 2: alloc pages to reach `sgsize` when `bin_size` < `sgsize`. */
	while (i < sgsize) {
		if ((r = map_page(data, va + i, 0, perm, NULL, MIN(sgsize - i, PAGE_SIZE))) != 0) {
			return r;
		}
		i += PAGE_SIZE;
	}
	return 0;
}
```

最后我们来看回调函数 `load_icode_mapper`

```c
panic_on(elf_load_seg(ph, binary + ph->p_offset, load_icode_mapper, e));
```

`load_icode_mapper` 定义在 kern/env.c 中。在函数的一开始，我们就将 `data` 还原为进程控制块

```c
static int load_icode_mapper(void *data, u_long va, size_t offset, u_int perm, const void *src, size_t len) {
	struct Env *env = (struct Env *)data;
	struct Page *p;
	int r;
```

我们想要将数据加载到内存，首先需要申请物理页。调用 `page_alloc` 函数申请空闲页

```c
	if ((r = page_alloc(&p)) != 0) {
		return r;
	}
```

接着，如果存在需要拷贝的数据，则将该数据复制到新申请的页所对应的内存空间中。我们使用 `page2kva` 获取页所对应的内核虚拟地址。另外注意这里需要考虑 `offset`。

```c
	if (src != NULL) {
		memcpy((void *)(page2kva(p) + offset), src, len);
	}
```

最后我们调用 `page_insert` 将虚拟地址映射到页上。为了区别不同进程的相同虚拟地址，我们需要附加 asid 信息，asid 保存在进程控制块中，这也是我们需要将进程控制块传入回调函数的原因。

```c
	return page_insert(env->env_pgdir, env->env_asid, p, va, perm);
```

### 创建进程 final

在 Lab 3 中我们“创建进程”是在操作系统初始化时直接创建，而不是通过 fork 系统调用来创建。在mips_init中，使用了宏:

```c
	ENV_CREATE_PRIORITY(user_bare_loop, 1);
	ENV_CREATE_PRIORITY(user_bare_loop, 2);
```

在 env.h 中定义:

```c
#define ENV_CREATE_PRIORITY(x, y)	\
	({	\
		extern u_char binary_##x##_start[];		\
		extern u_int binary_##x##_size;		\
		env_create(binary_##x##_start, (u_int)binary_##x##_size, y);	\
	})

#define ENV_CREATE(x)	\
	({		\
		extern u_char binary_##x##_start[];		\
		extern u_int binary_##x##_size;		\
		env_create(binary_##x##_start, (u_int)binary_##x##_size, 1);	\
	})
```

宏中定义了两个外部引用变量 `binary_##x##_start` 和 `binary_##x##_size`，`##` 是预处理器的连接操作符，它将 `x` 和 `_start` 拼接在一起，替换后即为:

```c
{
    extern u_char binary_user_bare_loop_start[];
    extern u_int binary_user_bare_loop_size;
    env_create(binary_user_bare_loop_start, (u_int)binary_user_bare_loop_size, 1)
}
```

可以看到，进程创建是由 `env_create` 函数进行的，

```c
struct Env *env_create(const void *binary, size_t size, int priority) {
```

`const void *binary` 是一个二进制的数据数组，该数组的大小为 `size_t size`，实际上此二进制数据即我们想要创建的进程的程序。“我们从哪里读入的程序？” 确实，我们根本没有进行磁盘操作。我们还没有实现文件系统，我们所 “加载” 的程序实际上是被一同编译到内核中的一段 ELF 格式的数据。这段数据中存在标签 `binary_user_bare_loop_start` 和 `binary_user_bare_loop_size`，所以我们才可以只通过引用外部变量的形式就 “加载” 了程序文件。

首先，`env_create` 调用 `env_alloc` 申请了一个新的空闲进程控制块，并对进程控制块的内容进行一些设置

```c
    struct Env *e;
    env_alloc(&e, 0);
	e -> env_pri = priority;
    e -> env_status = ENV_RUNNABLE;
```

接着使用 `load_icode` 函数为进程加载二进制程序，并且把进程控制块加到调度队列中

```c
    load_icode(e, binary, size);
    TAILQ_INSERT_HEAD(&env_sched_list, e, env_sched_link);
    return e;
}
```

到此为止，我们终于完成了进程创建的流程。在这一过程中，我们申请了新的进程控制块，初始化了该控制块的虚拟内存管理机制以及 trap frame 等其他信息。并将程序镜像加载到了该进程独占的虚拟内存空间中。

但是到目前为止，我们的进程还未运行起来，还不是动态的程序；仅是在内存空间中的一些有组织的数据而已。在接下来的小节中，我们会让进程运行起来。

## 异常处理

### CP0寄存器概览

| 寄存器助记符 | CP0 寄存器编号 | 描述                                            |
| ------------ | -------------- | ----------------------------------------------- |
| Status       | 12             | 状态寄存器，包括中断引脚使能，其他CPU模式等位域 |
| Cause        | 13             | 记录导致异常的原因                              |
| EPC          | 14             | 异常结束后程序恢复执行的位置                    |

Status:

![截屏2025-04-19 16.37.13](./images_report/%E6%88%AA%E5%B1%8F2025-04-19%2016.37.13.png)

Cause:

![截屏2025-04-19 16.38.29](./images_report/%E6%88%AA%E5%B1%8F2025-04-19%2016.38.29.png)

回顾一下计组P7，CPU处理异常需要:

1. 设置 EPC 指向从异常返回的地址
2. 设置 EXL 位，强制 CPU 进入内核态并禁止中断
3. 设置 Cause 寄存器，用于记录异常发生的原因
4. CPU 开始从异常入口位置取指，此后一切交给软件处理

操作系统的任务会从第4步开始

### 时钟中断

MOS使用的调度算法是时间片轮转算法，每个进程被分配一个时间片，即该进程允许运行的时间。如果在时间片结束时进程还在运行，则该进程将挂起，切换到另一个进程运行。如果该进程在时间片结束前阻塞或者结束，则立即切换到另一个进程运行。

MOS通过硬件定时器产生的时钟中断来判断一个进程是否用尽时间片。在MOS 中，时间片的长度是用时钟中断衡量的。比如设定某个进程的时间片的长度为 200 倍的 TIMER_INTERVAL (时钟中断间隔)，那么当MOS记录到该进程的执行中发生了 200 个时钟中断时，MOS 就知道该进程的时间片结束了。此时，当前运行的进程被挂起，MOS在调度队列中选取一个合适的进程运行。

4KC 中的 CP0 内置了一个可产生中断的 Timer，MOS就使用这个内置的 Timer 产生时钟中断。include/kclock.h 中的 RESET_KCLOCK 宏完成了对 CP0 中 Timer 的配置。

```assembly
#define TIMER_INTERVAL (500000)
.macro RESET_KCLOCK
	li 	t0, TIMER_INTERVAL
	mtc0 zero, CP0_COUNT
	mtc0 t0, CP0_COMPARE
.endm
```

具体来说 CP0 中存在两个用于控制此内置 Timer 的寄存器，即 Count 寄存器与 Compare 寄 存器。其中，Count 寄存器会按照某种仅与处理器流水线频率相关的频率不断自增，而 Compare 寄存器维持不变。当 Count 寄存器的值与 Compare 寄存器的值相等且非 0 时，时钟中断会被立即触发。

RESET_KCLOCK 宏将 Count 寄存器清零并将 Compare 寄存器配置为我们所期望的计时器周期数，这就对 Timer 完成了配置。在设定个时钟周期后，时钟中断将被触发。

MOS中，时钟中断的初始化发生在**调度执行每一个进程之前**。就是在 env_pop_tf 中调用了宏 RESET_KCLOCK，随后又在宏 RESTORE_ALL 中恢复了 Status 寄存器，开启了中断。

### 异常分发

当异常产生时，cpu 就会自动跳转到虚拟地址 `0x80000180` 处（特别的，当在用户态产生 TLB miss 异常时，会跳转到 `0x80000000`），从此处执行程序。这里的程序应该完成异常的处理，并使 cpu 返回正常程序。

对于 MOS 来说，此处实现了一个**异常分发**函数，根据异常的不同类型选择不同的异常处理函数。此函数在 kern/entry.S 中

```assembly
.section .text.tlb_miss_entry
tlb_miss_entry:
	j       exc_gen_entry

.section .text.exc_gen_entry
exc_gen_entry:
	SAVE_ALL
	mfc0    t0, CP0_STATUS
    and     t0, t0, ~(STATUS_UM | STATUS_EXL | STATUS_IE)
    mtc0    t0, CP0_STATUS
```

`tlb_miss_entry` 用于处理 TLB miss 异常，但实际上就是跳转到 `exc_gen_entry`。而在 `exc_gen_entry` 中，我们首先使用了一个宏 `SAVE_ALL`，该宏定义了一大段指令，用于将所有的寄存器值存储到栈帧中。这样我们便保存了异常发生时的上下文。需要注意的是，在 `SAVE_ALL` 中，我们将栈帧的初始位置设置为 `KSTACKTOP`（之前的 `sp` 位置保存在 `TF_REG29(sp)`）

```assembly
.macro SAVE_ALL
.set noat
.set noreorder
	mfc0    k0, CP0_STATUS
	andi    k0, STATUS_UM
	beqz    k0, 1f
	move    k0, sp
	/*
	* If STATUS_UM is not set, the exception was triggered in kernel mode.
	* $sp is already a kernel stack pointer, we don't need to set it again.
	*/
	li      sp, KSTACKTOP
1:
	subu    sp, sp, TF_SIZE
	sw      k0, TF_REG29(sp)
	...
	mfhi    k0
	sw      k0, TF_HI(sp)
	mflo    k0
	sw      k0, TF_LO(sp)
	sw      $0, TF_REG0(sp)
	sw      $1, TF_REG1(sp)
	...
.set at
.set reorder
.endm
```

注意，首先判断了sp指的位置

```bash
if (当前是用户态) {
    保存 sp（用户栈）
    切换 sp 到内核栈顶 KSTACKTOP
} else {
    不动，继续用当前 sp（已是内核栈）
}
```

接下来获取  `cause` 寄存器的值，取其 2-6 位，这部分对应异常码，用于区别不同的异常

```assembly
    mfc0 t0, CP0_CAUSE
    andi t0, 0x7c
    lw t0, exception_handlers(t0)
    jr t0
```

其中 `exception_handlers` 定义在 kern/traps.c 中。该数组是一个函数数组，其中每个元素都是异常码对应的异常处理函数。此数组称为异常向量组。

```c
extern void handle_int(void);
extern void handle_tlb(void);
extern void handle_sys(void);
extern void handle_mod(void);
extern void handle_reserved(void);

void (*exception_handlers[32])(void) = {
    [0 ... 31] = handle_reserved,
    [0] = handle_int,
    [2 ... 3] = handle_tlb,
#if !defined(LAB) || LAB >= 4
    [1] = handle_mod,
    [8] = handle_sys,	// 系统调用,用户通过此陷入内核
#endif
};
```

此语法是 GNU C 的扩展语法，`[first ... last] = value` 用于对数组上某个区间上元素赋同一个值。

最后，`tlb_miss_entry` 和 `exc_gen_entry` 还未被放在 `0x80000000` 和 `0x80000080` 处。我们需要在 kernel.lds 中添加内容，将这两个标签固定在特定的地址位置。

```c
	. = 0x80000000;
	.tlb_miss_entry : {
		*(.text.tlb_miss_entry)
	}

	. = 0x80000180;
	.exc_gen_entry : {
		*(.text.exc_gen_entry)
	}
```

### 中断处理

中断的处理流程:

1. 通过异常分发，判断出当前异常为中断异常，随后进入相应的中断处理程序。在 MOS 中即对应 `handle_int` 函数
2. 在中断处理程序中进一步判断 Cause 寄存器中是由几号中断位引发的中断，然后进入不同中断对应的中断服务函数
3. 中断处理完成，通过 `ret_from_exception` 函数恢复现场，继续执行

中断处理函数定义在 kern/genex.S 中:

我们定义了一个 `handle_\exception` 的函数，该函数调用 `\handler` 函数。返回后再调用 `ret_from_exception`

```assembly
.macro BUILD_HANDLER exception handler
NESTED(handle_\exception, TF_SIZE + 8, zero)
	move    a0, sp
	addiu   sp, sp, -8
	jal     \handler
	addiu   sp, sp, 8
	j       ret_from_exception
END(handle_\exception)
.endm

.text
FEXPORT(ret_from_exception)
	RESTORE_ALL
	eret
```

我们来看看 `ret_from_exception` 函数。首先该函数调用了一个宏 `RESTORE_ALL`，用于还原栈帧中通过调用 `SAVE_ALL` 保存的上下文

```assembly
FEXPORT(ret_from_exception)
	RESTORE_ALL
	eret
```

通过以下定义，我们创建了 `handle_tlb` 和 `handle_reserved` 函数。我们可以看到一个似曾相识的名字 `do_tlb_refill`。这个函数可以说是 Lab2 的核心。在 Lab2 的测试中我们只是模拟 tlb 的重填，而在 Lab3 中，我们终于将该函数实际应用了。

```assembly
BUILD_HANDLER tlb do_tlb_refill
BUILD_HANDLER reserved do_reserved
```

另外，直接定义了 `handle_int` 函数

```assembly
NESTED(handle_int, TF_SIZE, zero)
	mfc0    t0, CP0_CAUSE
	mfc0    t2, CP0_STATUS
	and     t0, t2
	andi    t1, t0, STATUS_IM7
	bnez    t1, timer_irq
timer_irq:
	li      a0, 0
	j       schedule
END(handle_int)
```

## 进程的调度

在 `handle_int` 中，如果检测到发生时钟中断，就会跳到 `timer_irq` ，进而到 `schedule` 函数处理中断

### 调度算法

`schedule` 函数定义在 `sched.c` 中，用的是**时间片轮转 (RR, Round Robin)** 算法。参数 `yield` 表示是否强制让出当前进程的CPU执行权。需要让出CPU执行权的情况:

- yield 为 1 时: 当前进程必须让出
- count 减为 0 时: 此时分给进程的时间片被用完，将执行权让给其他进程
- 无当前进程: 这必然是内核刚刚完成初始化，第一次产生时钟中断的情况，需要分配一个进程执行
- 进程状态不是可运行: 当前进程不能再继续执行，让给其他进程

其余情况继续执行当前进程即可。

```c
void schedule(int yield) {
	static int count = 0; // remaining time slices of current env
	struct Env *e = curenv;

	if (yield || count <= 0 || e == NULL || e -> env_status != ENV_RUNNABLE) {
		if (e != NULL) {
			TAILQ_REMOVE(&env_sched_list, e, env_sched_link);
			if (e -> env_status == ENV_RUNNABLE) {
				TAILQ_INSERT_TAIL(&env_sched_list, e, env_sched_link);
			}
		}
		panic_on(TAILQ_EMPTY(&env_sched_list));
		e = TAILQ_FIRST(&env_sched_list);
		count = e -> env_pri;	// 重置时间片
	}
	count--;
	env_run(e);
}
```

MOS 中设置时间片长度为为 N $\times$ TIMER_INTERVAL，其中 N 是 env 中的优先级。(就是用优先级规定时间片的长度)

### 进程的运行

`env_run` 函数负责进程的运行，如果 `curenv != NULL`，则需要切换进程，保存原来进程的上下文 

```c
void env_run(struct Env *e) {
	assert(e->env_status == ENV_RUNNABLE);
#ifdef MOS_PRE_ENV_RUN
	MOS_PRE_ENV_RUN_STMT
#endif
	if (curenv) {
		curenv->env_tf = *((struct Trapframe *)KSTACKTOP - 1);
	}
```

接着，我们将 `curenv` 的值变为 `e` 的值，实现当前进程的切换

```c
	curenv = e;
	curenv->env_runs++; // lab6
```

之后，我们将全局变量 `cur_pgdir` 设置为当前进程对应的页目录，实现页目录的切换

> 现在我们为 `cur_pgdir` 赋了值，就可以愉快地使用用户内存空间范围的虚拟地址了

```c
	cur_pgdir = curenv -> env_pgdir;	// cur_pgdir定义在pmap.c
```

最后，我们调用 `env_pop_tf` 函数，根据栈帧还原进程上下文，并运行程序。

```c
	env_pop_tf(&(curenv -> env_tf), curenv -> env_asid);
```

`env_pop_tf` 函数定义在 kern/env_asm.S 中。该函数将传入的 asid 值设置到 `EntryHi` 寄存器中，表示之后的虚拟内存访问都来自于 asid 所对应的进程。另外该函数将`sp` 寄存器地址设置为当前进程的 trap frame 地址，这样在最后调用 `ret_from_exception` 从异常处理中返回时，将使用当前进程的 trap frame 恢复上下文。程序也将从当前进程的 epc 中执行（epc 的值在 `load_icode` 中根据 elf 头设置为程序入口地址）。

```assembly
.text
LEAF(env_pop_tf)
.set reorder
.set at
	mtc0    a1, CP0_ENTRYHI
	move    sp, a0
	RESET_KCLOCK
	j       ret_from_exception
END(env_pop_tf)
```

最后，所有的寄存器都恢复成了当前进程所需要的状态，cpu 就像只知道当前进程这一个程序一样不断执行一条条指令，直到经过了一个时钟周期，又一个中断发生……

## Reference

- [Wokron｜BUAA-OS 实验笔记之 Lab3](https://wokron.github.io/posts/buaa-os-lab3/)
- [flyinglandlord｜lab-3 实验总结](https://flyinglandlord.github.io/2022/06/15/BUAA-OS-2022/lab3/doc-lab3/)
