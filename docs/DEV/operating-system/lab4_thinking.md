# Syscall & Fork

> Fork from [wokron's blog](https://wokron.github.io/posts/buaa-os-lab4/)
>
> 引用框是 Gemini 或 GPT 的回答。俺在学习的时候发现了 Google NotebookLM，非常推荐！可以把资料发给她，就会拥有OS实验的专属指导老师 🥰

## 系统调用

用户进程如果想执行特定的操作，如操作硬件、动态分配内存、与其他进程进行通信等，会引发特定的异常以陷入内核态，再在内核调用对应的函数。我们写程序是调用API，API内部再调用syscall。

在内核处理进程发起的系统调用时，我们并没有切换地址空间 (页目录地址)，也不需要将进程上下文 (Trapframe) 保存到进程控制块中，只是切换到内核态下，执行了一些内核代码。也就是说，处理系统调用时的内核仍然是代表当前进程的，这也是系统调用、TLB 缺失等同步异常与时钟中断等异步异常的本质区别。

### 从一个用户程序引入

在 Lab3 中使用 `ENV_CREATE` 完成了一些程序的加载，我们现在来看一下被加载程序的源代码？代码在 user/bare 路径下，查看其中的 put_a.c 程序

```c
void _start() {
	for (unsigned i = 0;; ++i) {
		if ((i & ((1 << 16) - 1)) == 0) {
			*(volatile char *)0xb80003f8 = 'a';
			*(volatile char *)0xb80003f8 = ' ';
		}
	}
}
```

发现这些所谓的程序并没有 `main` 函数，而是 `_start`。实际上和内核一样，我们同样在用于用户程序编译的链接器脚本中将程序入口设定为 `_start`。该脚本为 user/user.lds，其中同样有

```c
ENTRY(_start)
```

但是如果你查看一下我们将要查看的 user/tltest.c 程序，

```c
#include <lib.h>
int main() {
	debugf("Smashing some kernel codes...\n"
	       "If your implementation is correct, you may see unknown exception here:\n");
	*(int *)KERNBASE = 0;
	debugf("My mission completed!\n");
	return 0;
}
```

就会发现其又是以 `main` 函数为入口了。这是为什么呢？这是因为我们在 user/lib/entry.S 中定义了统一的 `_start` 函数。

```c
.text
EXPORT(_start)
	lw      a0, 0(sp)
	lw      a1, 4(sp)
	jal     libmain
```

值得注意跳转前的两条指令。这两条指令加载了 `argc` 和 `argv`。你可能还记得 Lab3 中 `env_alloc` 函数的这条语句，它将栈底的 8 个字节留给 `argc` 和 `argv`。

```c
	e->env_tf.regs[29] = USTACKTOP - sizeof(int) - sizeof(char **);
```

在 `_start` 函数的最后，跳转到 `libmain`。这个函数定义在 user/lib/libos.c 中: 

```c
void libmain(int argc, char **argv) {
	env = &envs[ENVX(syscall_getenvid())];
```

在这个函数中，我们通过一个函数 `syscall_getenvid` 获取了当前进程的 envid。此函数是一个系统调用。之后使用宏 `ENVX` 根据 envid 获取到该 envid 对应的进程控制块相对于进程控制块数组的索引值，并取得当前进程的进程控制块。

其中，`env` 和 `envs` 的定义，都在 user/include/lib.h 中。Lab3 中我们将页控制块数组和进程控制块数组映射到用户虚拟地址空间的某一位置，这里 `envs` 就是映射到的用户虚拟地址，同理还有 `pages`。他们和内核空间中的 `envs` 和 `pages` 是有区别的。虽然它们确实表示同一物理地址的相同数据，但是用户程序无法访问内核地址空间中的 `envs` 和 `pages`，如果访问会产生异常。

另外 `env` 用于表示本用户程序的进程控制块。我们可以通过访问 `env` 获取当前进程的信息。

```c
#define envs ((const volatile struct Env *)UENVS)
#define pages ((const volatile struct Page *)UPAGES)
extern volatile struct Env *env;
```

之后，`libmain` 函数便调用了 `main` 函数

```c
	main(argc, argv);
```

最后，当 main 函数返回后，我们调用 `exit` 函数结束进程

```c
	exit();
}
```

`exit` 函数同样定义在 user/lib/libos.c 中。

```c
void exit(void) {
	// After fs is ready (lab5), all our open files should be closed before dying.
#if !defined(LAB) || LAB >= 5
	close_all();
#endif
	syscall_env_destroy(0);
	user_panic("unreachable code");
}
```

`exit` 函数只调用了一个 `syscall_env_destroy`，该函数也是一个系统调用，用于销毁进程，释放进程资源。这里传入了一个参数 asid = 0，表示销毁的是当前进程。在之后还会多次出现 asid = 0 表示当前进程（调用函数的进程）的用法。

值得注意，在 `syscall_env_destroy` 之后还有一条语句 `user_panic`。此函数类似于 `panic`，不过用于表示用户程序出现了难以恢复的错误。在调用该函数打印错误信息后，就会结束该进程。

```c
// user/include/lib.h
void _user_panic(const char *, int, const char *, ...) __attribute__((noreturn));
#define user_panic(...) _user_panic(__FILE__, __LINE__, __VA_ARGS__)

// user/lib/debug.c
void _user_panic(const char *file, int line, const char *fmt, ...) {
	debugf("panic at %s:%d: ", file, line);
	va_list ap;
	va_start(ap, fmt);
	vdebugf(fmt, ap);
	va_end(ap);
	debugf("\n");
	exit();
}
```

在 `exit` 函数中出现的 `user_panic` 很明显的指出 `syscall_env_destroy` 是一个不会返回的函数。

这样，用户程序的入口的故事我们就讲完了。现在就可以理解 Lab3 中使用的用户程序位于 bare 路径下的原因了。因为他们没有被 `libmain` 包裹，是赤裸裸地暴露在外运行的。

之后我们再看一下 user/tltest.c 程序吧

```c
int main() {
	debugf("Smashing some kernel codes...\n"
	       "If your implementation is correct, you may see unknown exception here:\n");
	*(int *)KERNBASE = 0;
	debugf("My mission completed!\n");
	return 0;
}
```

这里用户程序试图向内核空间写入数据，于是就会产生异常，运行的结果如下。`do_reserved` 函数用于处理除了定义过异常处理函数的其他异常。5 号异常表示地址错误。

```c
Smashing some kernel codes...
If your implementation is correct, you may see unknown exception here:
...
panic at traps.c:24 (do_reserved): Unknown ExcCode 5
```

这里出现了一个新函数 `debugf`，该函数在用户程序的地位相当于内核中的 `printk` 。`debugf` 和相关函数的调用关系为: `debugf -> vdebugf -> vprintfmt -> debug_output -> debug_flush -> syscall_print_cons`。最终，为了向屏幕输出字符，我们的用户程序使用了系统调用 `syscall_print_cons`。

```c
static void debug_flush(struct debug_ctx *ctx) {
	if (ctx->pos == 0) {
		return;
	}
	int r;
	if ((r = syscall_print_cons(ctx->buf, ctx->pos)) != 0) {
		user_panic("syscall_print_cons: %d", r);
	}
	ctx->pos = 0;
}

static void debug_output(void *data, const char *s, size_t l) {
	struct debug_ctx *ctx = (struct debug_ctx *)data;
	while (ctx->pos + l > BUF_LEN) {
		size_t n = BUF_LEN - ctx->pos;
		memcpy(ctx->buf + ctx->pos, s, n);
		s += n;
		l -= n;
		ctx->pos = BUF_LEN;
		debug_flush(ctx);
	}
	memcpy(ctx->buf + ctx->pos, s, l);
	ctx->pos += l;
}

static void vdebugf(const char *fmt, va_list ap) {
	struct debug_ctx ctx;
	ctx.pos = 0;
	vprintfmt(debug_output, &ctx, fmt, ap);
	debug_flush(&ctx);
}

void debugf(const char *fmt, ...) {
	va_list ap;
	va_start(ap, fmt);
	vdebugf(fmt, ap);
	va_end(ap);
}
```

### 系统调用机制

![截屏2025-05-03 09.25.14](./images_report/%E6%88%AA%E5%B1%8F2025-05-03%2009.25.14.png)

MOS提供给用户程序的库中，所有的系统调用均为 `syscall_*` 的形式，他们都定义在 user/lib/syscall_lib.c 中，例如:

```c
void syscall_putchar(int ch) {
	msyscall(SYS_putchar, ch);
}

int syscall_print_cons(const void *str, u_int num) {
	return msyscall(SYS_print_cons, str, num);
}
```

这些函数（所有的系统调用函数）都调用了 `msyscall` 函数，并为其传入了不同数量、类型的参数，也即 `msyscall` 是一个拥有变长参数的函数。可以在 user/include/lib.h 中找到该函数的声明:

```c
extern int msyscall(int, ...);
```

第一个参数传入与调用名相似的宏 (比如 `SYS_putchar` )，称其为系统调用号，是内核区分不同系统调用的唯一依据。`msyscall` 最多有6个参数，这些参数如何从用户态传入内核态呢？使用**栈帧**。

> 理解一下栈帧:
>
> 1. 在 MIPS 的调用规范中，进入函数体时会通过对栈指针做减法 (压栈) 的方式为该函数自身的局部变量、返回地址、调用函数的参数分配存储空间。
>
> 2. 而在函数调用结束之后会对栈指针做加法 (弹栈) 来释放这部分空间。
>
> 3. 这部分空间称为栈帧 (stack frame)。 
>
> 栈帧的使用方法:
>
> - 调用方: 在自身栈帧的底部预留被调用函数的参数存储空间
> - 被调用方: 从调用方的栈帧中读取参数

寄存器 \$a0-\$a3 存放函数调用的前四个参数，其余的参数存放在栈中，但是前四个参数还需要在栈上预留空间，就像它们需要从栈上传递一样。根据这个规范，传递给内核的系统调用号及其他参数已经被合理安置。

`msyscall` 实现在 user/lib/syscall_wrap.S 中:

```assembly
LEAF(msyscall)
	syscall
	jr ra
END(msyscall)
```

真正关键的是 `syscall` 指令。这条指令用于让程序自行产生一个异常，该异常被称为系统调用异常。系统调用异常的处理入口为 `handle_sys` 函数，本质上是 `do_syscall` 函数在工作

```assembly
# kern/genex.S
.macro BUILD_HANDLER exception handler
NESTED(handle_\exception, TF_SIZE + 8, zero)
	move    a0, sp
	addiu   sp, sp, -8
	jal     \handler
	addiu   sp, sp, 8
	j       ret_from_exception
END(handle_\exception)
.endm

BUILD_HANDLER mod do_tlb_mod
BUILD_HANDLER sys do_syscall
```

回忆一下 Lab3 中从进入异常到进入特定异常的处理函数中间的过程。在这中间我们使用 `SAVE_ALL` 宏保存了发生异常时的现场。对于系统调用异常来说，需要着重强调，我们保存了调用 `msyscall` 函数时的参数信息。

`do_syscall` 函数位于 kern/syscall_all.c 中，该函数用于实现系统调用的分发和运行。

首先，我们取出用户程序 trap frame 中的 `a0` 寄存器的值。该值即调用 `msyscall` 函数时的第一个参数，是系统调用的类型。之后我们判断 `sysno` (系统调用号) 是否处于范围内，如果不是则 “返回”。需要注意这里返回值的设定方法，我们为 trap frame 中的 `v0` 寄存器赋值。

```c
void do_syscall(struct Trapframe *tf) {
	int (*func)(u_int, u_int, u_int, u_int, u_int);
	int sysno = tf->regs[4];
	if (sysno < 0 || sysno >= MAX_SYSNO) {
		tf->regs[2] = -E_NO_SYS;
		return;
	}
```

之后，我们将 `epc` 寄存器的地址加 4。`epc` 寄存器存储了发生异常的指令地址，当完成异常处理，调用 `ret_from_exception` 从异常中返回时，也是回到 `epc` 指向的指令再次执行。这里将 `epc` 寄存器的地址加 4，意味着从异常返回后并不重新执行 `syscall` 指令，而是其下一条指令。

```c
	tf -> cp0_epc += 4;
```

然后，我们根据 `sysno` 取得对应的系统调用函数。

```c
	func = syscall_table[sysno];
```

其中 `syscall_table` 中存储了所有的系统调用函数

```c
void *syscall_table[MAX_SYSNO] = {
    [SYS_putchar] = sys_putchar,
    [SYS_print_cons] = sys_print_cons,
    [SYS_getenvid] = sys_getenvid,
    [SYS_yield] = sys_yield,
    ...
};
```

最后，我们从用户程序的 trap frame 中取出调用 `msyscall` 时传入的参数，调用对应的系统调用函数，并将其返回值存储在 `v0` 寄存器中。

```c
	u_int arg1 = tf->regs[5];
	u_int arg2 = tf->regs[6];
	u_int arg3 = tf->regs[7];

	u_int arg4, arg5;
	arg4 = *(u_int*)(tf->regs[29] + 16);
	arg5 = *(u_int*)(tf->regs[29] + 20);

	tf->regs[2] = func(arg1, arg2, arg3, arg4, arg5);
```

实际上我们不难看出，在 `do_syscall` 函数中我们修改了用户程序 trap frame 的值，其目的就是模拟函数调用的效果。这样在恢复现场，用户程序继续运行时，就会感觉和平常的函数调用没有不同之处。异常的处理过程，就仿佛变成了内核暴露给用户程序的接口了。

### 一些系统调用函数实现

kern/syscall_all.c 中定义了一系列系统调用:

先看一个辅助函数 `envid2env` : 函数的作用是根据进程标识符 `envid` 获取对应的进程。

```c
int envid2env(u_int envid, struct Env **penv, int checkperm) {
	struct Env *e;
	if (envid == 0) {
		*penv = curenv;
		return 0;
	}
	e = envs + ENVX(envid);
```

这里出现了 envid = 0 时直接返回当前进程的实现。另外，因为 envid = 0 有特殊含义，所以生成进程标识符的函数 `mkenvid` 永远不可能生成为 0 的 envid。对于 envid 不为 0 的情况，我们通过 `ENVX` 宏获取 envid 对应的进程控制块相对于进程控制块数组的索引。`mkenvid` 的算法中，envid 的低十位就是进程控制块的索引，因此我们才可以通过 envid 获取对应的进程控制块。

在 `envid2env` 的后半部分，我们进行了一系列进程控制块的有效性检查。注意这里的 `e->env_id != envid`，按说我们是通过 `envid` 获取的进程控制块，怎么可能进程控制块的 `env_id` 不相同呢？这实际上考虑了这样一种情况，某一进程完成运行，资源被回收，这时其对应的进程控制块会插入回 `env_free_list` 中。当我们需要再次创建内存时，就可能重新取得该进程控制块，并为其赋予不同的 envid。这时，已销毁进程的 envid 和新创建进程的 envid 都能通过 `ENVX` 宏取得相同的值，对应了同一个进程控制块。可是已销毁进程的 envid 却不应当再次出现，`e->env_id != envid` 就处理了 `envid` 属于已销毁进程的情况。

```c
	if (e->env_status == ENV_FREE || e->env_id != envid) {
		return -E_BAD_ENV;
	}
```

对于设置了 `checkperm` 的情况，我们还需要额外检查传入的 `envid` 是否与当前进程具有直接亲缘关系。由此也可以得知，对于一些操作，只有父进程对子进程具有权限。

```c
	if (checkperm) {
		if (curenv -> env_id != e -> env_id && curenv -> env_id != e -> env_parent_id) {
			return -E_BAD_ENV;
		}
	}
```

最后返回取得的进程控制块

```c
	*penv = e;
	return 0;
}
```

**再看一些系统调用函数:**

- `sys_mem_alloc` :

通过这个系统调用，用户程序可以给该程序所允许的虚拟内存空间显式地分配实际的物理内存。对于程序员，是我们编写的程序在内存中申请了一片空间；对于操作系统内核，是一个进程请求将其运行空间中的某段地址与实际物理内存进行映射。

首先判断虚拟地址是否合法，

```c
int sys_mem_alloc(u_int envid, u_int va, u_int perm) {
	struct Env *env;
	struct Page *pp;
    
	if (is_illegal_va(va)) {
		return -E_INVAL;
	}
```

其中的 `is_illegal_va()` 函数的定义为:

```c
static inline int is_illegal_va(u_long va) {
	return va < UTEMP || va >= UTOP;
}
```

然后使用了 `envid2env` 来根据进程标识符 `envid` 获取对应的进程。

```c
	try(envid2env(envid, &env, 1));
```

随后分配一个物理页，并建立虚拟地址 `va` 到物理页的映射

```c
	try(page_alloc(&pp));
	return page_insert(env->env_pgdir, env->env_asid, pp, va, perm);
}
```

- `sys_mem_map` :

将源进程地址空间中的相应内存映射到目标进程的相应地址空间的相应虚拟内存中去，此时两者共享一页物理内存

```c
int sys_mem_map(u_int srcid, u_int srcva, u_int dstid, u_int dstva, u_int perm) {
	struct Env *srcenv;
	struct Env *dstenv;
	struct Page *pp;

	if (is_illegal_va(srcva) || is_illegal_va(dstva)) {
		return -E_INVAL;
	}	
	try(envid2env(srcid, &srcenv, 1));
	try(envid2env(dstid, &dstenv, 1));

	// Find the physical page mapped at 'srcva' in the address space of 'srcid'
	pp = page_lookup(srcenv -> env_pgdir, srcva, NULL);
	if (pp == NULL) {
		return -E_INVAL;
	}

	// Map the physical page at 'dstva' in the address space of 'dstid'
	return page_insert(dstenv->env_pgdir, dstenv->env_asid, pp, dstva, perm);
}
```

- `sys_mem_unmap` :

解除某个进程地址空间虚拟内存和物理内存之间的映射关系

```c
int sys_mem_unmap(u_int envid, u_int va) {
	struct Env *e;
	if (is_illegal_va(va)) {
		return -E_INVAL;
	}
	try(envid2env(envid, &e, 1));

	// Unmap the physical page at 'va' in the address space of 'envid'
	page_remove(e->env_pgdir, e->env_asid, va);
	return 0;
}
```

- `sys_yield` : 

实现用户进程对 CPU 的放弃，从而调度其他的进程

```c
void __attribute__((noreturn)) sys_yield(void) {
	schedule(1);
}
```

- `sys_env_destroy` :

Lab3 出现过 `env_destroy`，现在我们将其封装成系统调用，作为 main 函数之后的资源回收函数。我们使用的系统调用为 `sys_env_destroy`，封装了一下 `env_destroy`。

```c
int sys_env_destroy(u_int envid) {
	struct Env *e;
	try(envid2env(envid, &e, 1));
	printk("[%08x] destroying %08x\n", curenv->env_id, e->env_id);
	env_destroy(e);
	return 0;
}
```

那么 `env_destroy` 呢？该函数主要作用是调用了 `env_free` 函数。另外对于是当前函数被销毁的情况，需要进行进程调度。

```c
// kern/env.c
void env_destroy(struct Env *e) {
	env_free(e);
	if (curenv == e) {
		curenv = NULL;
		printk("i am killed ... \n");
		schedule(1);
	}
}
```

对于 `env_free` 函数。首先遍历所有页表项，使用 `page_remove` 删除虚拟地址到物理页的映射；另外使用 `page_decref` 释放页表和页目录本身。其中还使用 `asid_free` 释放了 asid。对页表项进行修改后，使用了 `tlb_invalidate` 将对应的项无效化。

```c
void env_free(struct Env *e) {
	Pte *pt;
	u_int pdeno, pteno, pa;
	printk("[%08x] free env %08x\n", curenv ? curenv->env_id : 0, e->env_id);
    
	// Flush all mapped pages in the user portion of the address space
	for (pdeno = 0; pdeno < PDX(UTOP); pdeno++) {
		if (!(e->env_pgdir[pdeno] & PTE_V)) {	// only look at mapped page tables
			continue;
		}
		pa = PTE_ADDR(e->env_pgdir[pdeno]);
		pt = (Pte *)KADDR(pa);
		// Unmap all PTEs in this page table
		for (pteno = 0; pteno <= PTX(~0); pteno++) {
			if (pt[pteno] & PTE_V) {
				page_remove(e->env_pgdir, e->env_asid,
					    (pdeno << PDSHIFT) | (pteno << PGSHIFT));
			}
		}
		e->env_pgdir[pdeno] = 0;
		page_decref(pa2page(pa));
		tlb_invalidate(e->env_asid, UVPT + (pdeno << PGSHIFT));
	}
	page_decref(pa2page(PADDR(e->env_pgdir)));
	asid_free(e->env_asid);
	tlb_invalidate(e->env_asid, UVPT + (PDX(UVPT) << PGSHIFT));
```

最后修改进程控制块的状态为 `ENV_FREE`，将该控制块从调度队列中删除，重新放回空闲列表中。

```c
	e->env_status = ENV_FREE;
	LIST_INSERT_HEAD((&env_free_list), (e), env_link);
	TAILQ_REMOVE(&env_sched_list, (e), env_sched_link);
}
```

## fork 的实现

![截屏2025-05-04 10.14.28](./images_report/%E6%88%AA%E5%B1%8F2025-05-04%2010.14.28.png)

### 理解 fork

进程可以创建进程，一个进程在调用 `fork()` 函数后，将从此分叉成为两个进程 (父进程和子进程) 运行。子进程开始运行时的大部分上下文状态与父进程相同 (包括通用寄存器和程序计数器 PC 等)。父进程的返回值为子进程的进程标识符 envid，而子进程的返回值为 0。我们也知道，envid = 0 可以表示当前进程，这样的话也可以理解成不管在父进程还是子进程，返回值都为指示子进程的 envid。

### 用户态异常处理的实现

我们看一下 `fork` 函数的代码吧，它位于 user/lib/fork.c 中。`fork` 本身不是系统调用，但该函数使用了许多系统调用来完成子进程的创建。

```c
int fork(void) {
	u_int child;
	u_int i;

    // env定义在 user/include/lib.h
    // extern const volatile struct Env *env;
	if (env -> env_user_tlb_mod_entry != (u_int)cow_entry) {
		try(syscall_set_tlb_mod_entry(0, cow_entry));
	}
```

首先我们遇到了一个系统调用 `syscall_set_tlb_mod_entry`，对应的内核函数为 `sys_set_tlb_mod_entry`，只是设置了进程控制块的 `env_user_tlb_mod_entry` 参数。

```c
int sys_set_tlb_mod_entry(u_int envid, u_int func) {
	struct Env *env;
	try(envid2env(envid, &env, 1));
	env -> env_user_tlb_mod_entry = func;
	return 0;
}
```

执行这个系统调用的目的是设置 TLB Mod 异常的处理函数。TLB Mod 异常即页写入（Modify）异常，会在程序试图写入不可写入（对应页表项无 `PTE_D`）的物理页面时产生。

>内核在捕获到一个常规的缺页中断 (TLB 缺失异常) 时会陷入异常，跳转到异常处理函数 `handle_tlb` 中，这一汇编函数的实现在 kern/genex.S 中，通过调用 `do_tlb_refill` 函数，在页表中进行查找，将物理地址填入 TLB 并返回到用户程序中的异常地址，再次执行访存指令。
>
>当用户程序写入一个在 TLB 中被标记为不可写入 (无 PTE_D) 的页面时，MIPS 会陷入页写入异常 (TLB Mod)，我们在异常向量组中为其注册了一个处理函数 handle_mod，这一函数会跳转到 kern/tlbex.c 中的 `do_tlb_mod` 函数中，这个函数正是处理页写入异常的内核函数。
>
>页写入异常的处理，是不能直接使用正常情况下的用户栈的 (因为用户栈的页面也可能发生页写入异常)，所以用户进程就需要一个单独的栈来执行处理程序， 我们把这个栈称作**异常处理栈**，它的栈顶对应的是内存布局中的 **UXSTACKTOP**
>
>```bash
> o      UVPT     -----> +----------------------------+------------0x7fc0 0000    |
> o                      |           pages            |     PDMAP                 |
> o      UPAGES   -----> +----------------------------+------------0x7f80 0000    |
> o                      |           envs             |     PDMAP                 |
> o  UTOP,UENVS   -----> +----------------------------+------------0x7f40 0000    |
> o  UXSTACKTOP -/       |     user exception stack   |     PTMAP                 |
> o                      +----------------------------+------------0x7f3f f000    |
> o                      |                            |     PTMAP                 |
> o     USTACKTOP -----> +----------------------------+------------0x7f3f e000    |
> o                      |     normal user stack      |     PTMAP                 |
> o                      +----------------------------+------------0x7f3f d000    |
>```

我们看一下内核中 TLB Mod 异常的处理函数 `do_tlb_mod` :

在该函数中，我们首先将 `sp` 寄存器设置到 `UXSTACKTOP` 的位置。`UXSTACKTOP` 和 `KSTACKTOP` 类似，都是在处理异常时使用的调用栈，因为我们要将异常处理交由用户态执行，因此需要在用户的地址空间中分配。另外这里添加判断语句是考虑到在异常处理的过程中再次出现异常的情况，我们不希望之前的异常处理过程的信息丢失，而希望在处理了后一个异常后再次继续处理前一个异常，于是对于 `sp` 已经在异常栈中的情况，就不再从异常栈顶开始。

```c
void do_tlb_mod(struct Trapframe *tf) {
	struct Trapframe tmp_tf = *tf;	// 保留原先 trap frame 的内容

	if (tf->regs[29] < USTACKTOP || tf->regs[29] >= UXSTACKTOP) {
		tf->regs[29] = UXSTACKTOP;
	}
```

随后我们在用户异常栈底分配一块空间，用于存储 trap frame。这里和在 `KSTACKTOP` 底使用 `SAVE_ALL` 类似。

```c
	tf->regs[29] -= sizeof(struct Trapframe);
	*(struct Trapframe *)tf->regs[29] = tmp_tf;
	// tf的sp限定在了 USTACKTOP 和 UXSTACKTOP 之间
	// 把原先的 trap frame 搬到了tf的sp指向的位置
	// 此时，用户异常处理栈上存放了一个未经修改的原始的异常现场
	// 将被作为参数传递给用户态异常处理函数
```

最后我们从当前进程的进程控制块的 `env_user_tlb_mod_entry` 取出当前进程的 TLB Mod 异常处理函数，“调用” 该处理函数。当然，因为处于异常处理过程中，所以这里我们采用和一般系统调用类似的处理方法。首先我们设定 `a0` 寄存器的值为 trap frame 所在的地址，接着 `sp` 寄存器自减，留出第一个参数的空间。最后设定 `epc` 寄存器的值为用户的 TLB Mod 函数的地址，使得恢复现场后跳转到该函数的位置继续执行。

```c
	Pte *pte;
	page_lookup(cur_pgdir, tf->cp0_badvaddr, &pte);
	if (curenv->env_user_tlb_mod_entry) {
		tf->regs[4] = tf->regs[29];		// 修改a0,maybe将作为cow_entry的参数
		tf->regs[29] -= sizeof(tf->regs[4]);	
		tf->cp0_epc = curenv->env_user_tlb_mod_entry;
	} else {
		panic("TLB Mod but no user handler registered");
	}
}
```

应当注意，作为用户态异常处理函数的参数的 `TrapFrame *tf` (tmp_tf) 和当前进程的 `tf` 是不同的。前者依旧是产生 TLB Mod 异常时的现场；而后者则经过了修改，以便在从异常处理返回时跳转到用户态异常处理函数而非产生异常的位置。在用户态异常处理函数完成异常处理后，我们会通过前者返回产生异常的位置。

> `void do_tlb_mod(struct Trapframe *tf)` 的 `struct Trapframe *tf` 是当TLB Mod异常发生时，CPU从用户态切换到内核态后，在内核栈上保存的那个原始的Trapframe，包含了异常发生瞬间用户进程的所有寄存器状态
>
> 很神奇，syscall完后会跳入用户设置的异常，内核仅仅设置跳过去的地址
>
> 页写入异常处理流程图:
>
> ![截屏2025-05-08 10.15.33](./images_report/%E6%88%AA%E5%B1%8F2025-05-08%2010.15.33.png)

### 写时复制技术 (COW, Copy on Write)

所以说，我们为什么需要设置 TLB Mod 的异常处理函数呢？在这里我们的目的是实现进程的写时复制。我们知道，`fork` 会根据复制调用进程来创建一个新进程。可如果每创建一个新的进程就要在内存中复制一份相同的数据，开销就太大了。所以我们可以在创建子进程时只是让子进程映射到和父进程相同的物理页。这样如果父进程和子进程只是读取其中的内容，就可以共享同一片物理空间。那么如果有进程想要修改该空间内的数据要怎么办？这时我们才需要将这块物理空间复制一份，让想要修改的进程只修改属于自己的数据。

在 MOS 中，对于可以共享的页，我们会去掉其写入 (`PTE_D`) 权限，为其赋予写时复制标记 (`PTE_COW`，为了区分只读与写时复制)，这样当共享该页面的某一个进程尝试修改该页的数据时，因为不具有 `PTE_D` 权限，就会产生 TLB Mod 异常。转到异常处理函数。

根据 `fork` 中的语句可知，我们的异常处理函数为 `cow_entry`，同样位于 user/lib/fork.c 中。

```c
static void __attribute__((noreturn)) cow_entry(struct Trapframe *tf) {
	u_int va = tf->cp0_badvaddr;
	u_int perm;
	perm = vpt[VPN(va)] & 0xfff;
	if (!(perm & PTE_COW)) {
		user_panic("perm doesn't have PTE_COW");
	}
```

首先，我们取得发生异常时尝试写入的虚拟地址位置，使用 `VPN` 宏获取该虚拟地址对应的页表项索引。之后使用 `vpt` 获取页表项的内容，通过位操作取出该页表项的权限。发生 TLB Mod 的必然是被共享的页面，因此如果不具有 `PTE_COW` 则产生一个 `user_panic`。

值得注意的是 `vpt`

```c
#define vpt ((const volatile Pte *)UVPT)
#define vpd ((const volatile Pde *)(UVPT + (PDX(UVPT) << PGSHIFT)))
```

从表现上来看，`vpt` 是一个 `Pte` 类型的数组，其中按虚拟地址的顺序存储了所有的页表项。从本质上看，`vpt` 是用户空间中的地址，并且**正是页表自映射时设置的基地址**。这样我们就不难理解为什么可以通过这取得所有页表项了。类似的还有 `vpd`，是存储页目录项的数组。

如果忘了页表自映射，可以回头看看 Lab2 和 Lab3。在 MOS 中实现页表自映射的语句位于 kern/env.c 的 `env_setup_vm` 中。

```c
	/* Map its own page table at 'UVPT' with readonly permission.
	 * As a result, user programs can read its page table through 'UVPT' */
	e->env_pgdir[PDX(UVPT)] = PADDR(e->env_pgdir) | PTE_V;
```

让我们回到 `cow_entry`，接下来重新设置页的权限，去掉 `PTE_COW`，加上 `PTE_D`。

```c
	perm = (perm & ~PTE_COW) | PTE_D;
```

接着再申请一页新的物理页，该物理页对应的虚拟地址为 `UCOW`。因为处于用户程序中，所以不能使用 `page_alloc`，而需要使用系统调用。需要注意，因为 asid = 0，所以新物理页是属于当前调用进程的。

```c
	syscall_mem_alloc(0, (void *)UCOW, perm);
```

> 还有一点值得思考，这里我们申请的物理页不在发生异常的 `va` 的位置，而是特定的 `UCOW`。之后又通过一系列映射操作将该物理页映射到 `va`。为什么要这样做呢？很简单，如果我们一开始就映射到 `va`，那么如果你还记得 `page_insert` 内容的话，就会知道这样会使原本的映射丢失，我们就不能访问原本 `va` 映射到的物理页的内容了。这样的话我们又要如何复制呢？所以要先映射到一个完全无关的地址 `UCOW`。该地址之上 `PAGE_SIZE` 大小的空间专门用于写时复制时申请新的物理页。

回到 `cow_entry`，接下来我们将 `va` 所在物理页的内容复制到新申请的页面中。注意 `va` 可能只是随意的一个虚拟地址，不一定页对齐，因此还需要使用 `ROUNDDOWN` 将其对齐。

```c
	memcpy((void *)UCOW, (void *)ROUNDDOWN(va, PAGE_SIZE), PAGE_SIZE);
```

现在我们只需要取消 `va` 到原物理页的映射，将 `va` 映射到新申请的物理页即可。因此首先，使用系统调用 `syscall_mem_map` 将当前进程的 `UCOW` 所在的物理页，作为当前进程的 `va` 地址所映射的物理页，并设定其权限即可。

```c
	syscall_mem_map(0, (void *)UCOW, 0, (void *)va, perm);
```

`UCOW` 处还存在到新申请物理页的映射，我们需要取消该映射。

```c
	syscall_mem_unmap(0, (void *)UCOW);
```

最后，我们需要在异常处理完成后**恢复现场**，可现在位于用户态，去哪里恢复现场呢？我们需要使用系统调用 `syscall_set_trapframe`。也因此 `cow_entry` 成为了一个不返回的函数 （`__attribute__((noreturn))`）。

```c
	int r = syscall_set_trapframe(0, tf);
	user_panic("syscall_set_trapframe returned %d", r);
}
```

`sys_set_trapframe` 将 trap frame 修改为传入的参数 `struct Trapframe *tf` 对应的 trap frame。这样当**从该系统调用返回**时，将返回设置的栈帧中 `epc` 的位置。对于 `cow_entry` 来说，意味着恢复到产生 TLB Mod 时的现场。

```c
int sys_set_trapframe(u_int envid, struct Trapframe *tf) {
	if (is_illegal_va_range((u_long)tf, sizeof *tf)) {
		return -E_INVAL;
	}
	struct Env *env;
	try(envid2env(envid, &env, 1));
	if (env == curenv) {
		*((struct Trapframe *)KSTACKTOP - 1) = *tf;
		return tf->regs[2];
	} else {
		env->env_tf = *tf;
		return 0;
	}
}
```

!!! note "Mod异常处理流程"
    当写入不能写入的页面时 (即没有PTE_D标识位的页面)，会触发TLB Mod异常，将陷入内核处理异常，流程为:
    ![截屏2025-05-07 18.29.50](./images_report/%E6%88%AA%E5%B1%8F2025-05-07%2018.29.50.png)

### 一次调用、两次返回

我们继续看 `fork` 函数。接下来的步骤是 `fork` 的关键。我们使用 `syscall_exofork` 系统调用，作用是复制父进程的信息，创建一个子进程。该系统调用实现了一次调用、两次返回，也是 `fork` 拥有此能力的原因。

```c
int fork(void) {
	u_int child;
	u_int i;
	if (env->env_user_tlb_mod_entry != (u_int)cow_entry) {
		try(syscall_set_tlb_mod_entry(0, cow_entry));
	}

	child = syscall_exofork();
```

在此调用之后的一条语句，我们就通过不同返回值实现了父子进程的不同流程，对于子进程来说，我们直接返回 0。这里我们还设置了 `env` 的值为当前进程（因为复制后，`env` 原本还指向父进程）。为了取得当前进程的 envid，我们又使用了系统调用 `syscall_getenvid`。再次强调，`env` 和 `envs` 是位于用户空间的。

```c
	if (child == 0) {
		env = envs + ENVX(syscall_getenvid());
		return 0;
	}
```

> `env` 定义在 libos.c 里？`const volatile struct Env *env;` `env` 的本意是指向自身的进程控制块

接下来我们分析一下 `syscall_exofork`。首先，为了创建新的进程，我们需要申请一个进程控制块。需要注意，这里 `env_alloc` 的 parent 参数为当前进程的 envid。

```c
int sys_exofork(void) {
	struct Env *e;
	try(env_alloc(&e, curenv->env_id));
```

接着我们复制父进程调用系统操作时的现场。

```c
	e->env_tf = *((struct Trapframe *)KSTACKTOP - 1);
```

需要注意这里不能使用 `curenv->env_tf`。因为 `curenv->env_tf` 存储的是进程调度，切换为其他进程之前的 trap frame。而父进程调用 `syscall_exofork` 时保存的现场，并不等同于`curenv->env_tf` 中的信息。

>`KSTACKTOP` 的含义:
>
>- 从用户态陷入内核态时，CPU会将当前用户态的寄存器现场自动保存到**当前进程的内核栈顶**。
>- KSTACKTOP是内核栈的最高地址，`(struct Trapframe *)KSTACKTOP - 1` 是指向刚刚压入内核栈的 `Trapframe` 结构体的指针，这个 `Trapframe` 表示期望从系统调用返回到用户态时的状态。
>
>`curenv->env_tf` 的含义:
>
>- `curenv->env_tf` 保存的是 `curenv` 这个进程**上一次因为进程调度 (而不是当前这次系统调用) 而被换下CPU时**的现场
>
>- 只有一处进行了 `env_tf` 的赋值。就是在 `env_run` 函数中
>
>  ```c
>  if (curenv) {
>      curenv->env_tf = *((struct Trapframe *)KSTACKTOP - 1);
>  }
>  ```

最后 `syscall_exofork` 将子进程的返回值设置为 0，同时设置进程的状态和优先级，返回新创建进程的 envid。

```c
	e->env_tf.regs[2] = 0;
	e->env_status = ENV_NOT_RUNNABLE;
	e->env_pri = curenv->env_pri;
	return e->env_id;
}
```

到这里你应该能理解何为 “一次调用、两次返回” 了。一次调用指的是只有父进程调用了 `syscall_exofork`，两次返回分别是父进程调用 `syscall_exofork` 得到的返回值和被创建的子进程中设定了 `v0` 寄存器的值为 0 作为返回值。这样当子进程开始运行时，就会拥有一个和父进程不同的返回值。

### 子进程运行前的设置

我们已经创建了子进程并对进程控制块进行了设置，但是子进程现在还没有加入调度队列，同时子进程建立好了自己的页目录并拷贝了模板页目录内地址空间的映射 ( syscall_exofork $\rightarrow$ env_alloc $\rightarrow$ env_setup_vm )，但其他页表还没有“复制”，页表项还没有设置 `PTE_COW` 位。这些我们都要进行处理。

设置 `PTE_COW` 位需要通过 `duppage` 函数。如果当前页表项具有 `PTE_D` 权限（且不是共享页面 `PTE_LIBRARY`），则需要重新设置页表项的权限。`duppage` 会对每一个页表项进行操作，因此我们需要在 `fork` 中遍历所有的页表项。相比于在内核态中，在用户态中遍历页表项更为方便。

这里我们只遍历 `USTACKTOP` 之下的地址空间，因为其上的空间总是会被共享。在调用 `duppage` 之前，我们判断页目录项和页表项是否有效。如果不判断则会在 `duppage` 函数中发生异常（最终是由 `page_lookup` 产生的）。

```c
int fork(void) {
	u_int child;
	u_int i;
	if (env->env_user_tlb_mod_entry != (u_int)cow_entry) {
		try(syscall_set_tlb_mod_entry(0, cow_entry));
	}
	child = syscall_exofork();
	if (child == 0) {
		env = envs + ENVX(syscall_getenvid());
		return 0;
	}	

    // Map all mapped pages below 'USTACKTOP' into the child's address space
	for (i = 0; i < VPN(USTACKTOP); i++) {
		if ((vpd[i >> 10] & PTE_V) && (vpt[i] & PTE_V)) {	// PDE有效且PTE有效
			duppage(child, i);
		}
	}
```

需要注意取页目录项的方法，`vpd` 是页目录项数组，`i` 相当于地址的高 20 位，我们需要取得地址的高 10 位作为页目录的索引，因此有 `vpd[i >> 10]`

接下来让我们看一下 `duppage` 函数。首先取得页表项对应的虚拟地址和权限。

```c
static void duppage(u_int envid, u_int vpn) {
	int r;
	u_int addr;
	u_int perm;
    
	addr = vpn << PGSHIFT;
	perm = vpt[vpn] & 0xfff;
```

接着，对所有有效的页，我们都需要通过系统调用 `syscall_mem_map` 实现父进程与子进程页面的共享。特别的，对于可写的，且不是共享的页，我们还需要更新页表项的权限。在用户态，我们不能直接修改页表项，因此需要通过系统调用来实现修改。我们同样使用的是 `syscall_mem_map`。虽然之前我们使用此系统调用来进行页的共享和复制，但由于该系统调用具有新的映射会覆盖旧的映射的特点，因此可以对本来就有的关系采取重新映射，只改变权限位的设置，就可以实现权限位的修改。

```c
	int flag = 0;
	if ((perm & PTE_D) && !(perm & PTE_LIBRARY) && !(perm & PTE_COW)) {
		perm = (perm | PTE_COW) & ~PTE_D;
		flag = 1;
	}
	// int sys_mem_map(u_int srcid, u_int srcva, u_int dstid, u_int dstva, u_int perm)
	try(syscall_mem_map(0, addr, envid, addr, perm));
	if (flag) {
		try(syscall_mem_map(0, addr, 0, addr, perm));	// 修改自己的权限
	}
}
```

这里需要注意，父进程将页映射到子进程应该先于对自己权限的修改。如果先修改自己的权限位，则该页表就不再可写，这样的话就会发生 TLB Mod 异常，而不能实现父进程将页映射到子进程。

之后在 `fork` 中，我们同样设置子进程的 TLB Mod 异常处理函数为 `cow_entry`。

```c
	try(syscall_set_tlb_mod_entry(child, cow_entry));
```

最后，我们调用 `syscall_set_env_status` 将子进程状态设定为 `RUNNABLE` 并将其加入调度队列中。返回子进程的 envid。作为父进程 `fork` 的返回值。

```c
	try(syscall_set_env_status(child, ENV_RUNNABLE));
	return child;
}
```

`syscall_set_env_status` 较为简单，只是根据设定的状态将进程加入或移除调度队列而已。

```c
int sys_set_env_status(u_int envid, u_int status) {
	struct Env *env;
	if (status != ENV_RUNNABLE && status != ENV_NOT_RUNNABLE) {
		return -E_INVAL;
	}
	try(envid2env(envid, &env, 1));
    // Update 'env_sched_list' if the 'env_status' of 'env' is being changed
	if (status == ENV_NOT_RUNNABLE && env -> env_status != ENV_NOT_RUNNABLE) {
		TAILQ_REMOVE(&env_sched_list, env, env_sched_link);
	}
	if (status == ENV_RUNNABLE && env -> env_status != ENV_RUNNABLE) {
		TAILQ_INSERT_TAIL(&env_sched_list, env, env_sched_link);
	}
	env->env_status = status;
	if (env == curenv) {
		schedule(1);
	}
	return 0;
}
```

这样，我们就通过 `fork` 完成了子进程的创建。

## 进程间通信

我们还需要实现进程间通信，也就是把一个地址空间中的东西传给另一个地址空间。所有的进程都共享同一个内核空间 (主要为 kseg0)。因此，我们就可以借助于内核空间来实现数据交换。发送方进程可以将数据存放在进程控制块中，接收方进程在进程控制块中找到对应的数据。

Lab4用到了进程控制块的这些域:

```c
struct Env {
    // ipc
	u_int env_ipc_value;   // the value sent to us
	u_int env_ipc_from;    // envid of the sender
	u_int env_ipc_recving; // whether this env is blocked receiving
	u_int env_ipc_dstva;   // va at which the received page should be mapped
	u_int env_ipc_perm;    // perm in which the received page should be mapped

	// fault handling
	u_int env_user_tlb_mod_entry; // userspace TLB Mod handler
}
```

还需要实现两个系统调用 `syscall_ipc_try_send` 和 `syscall_ipc_recv`。

### 信息接收

调用 `syscall_ipc_recv` 后会阻塞当前进程，直到收到信息。

该调用的参数为要接收信息的虚拟地址。首先我们要检查虚拟地址是否处于用户空间。另外当 `dstva` 为 0 时表示不需要传输整个页面。

```c
int sys_ipc_recv(u_int dstva) {
	if (dstva != 0 && is_illegal_va(dstva)) {
		return -E_INVAL;
	}
```

接着我们设置进程控制块的字段，`env_ipc_recving` 表示进程是否正在接收信息；`env_ipc_dstva` 存储要接收信息的地址。

```c
	curenv -> env_ipc_recving = 1;
	curenv -> env_ipc_dstva = dstva;
```

接着我们要阻塞当前进程，将该进程从调度队列中移出

```c
	curenv -> env_status = ENV_NOT_RUNNABLE;
	TAILQ_REMOVE(&env_sched_list, curenv, env_sched_link);
```

最后我们将返回值设置为 0，调用 `schedule` 函数进行进程切换。

```c
	((struct Trapframe *)KSTACKTOP - 1)->regs[2] = 0;
	schedule(1);
}
```

`schedule` 函数不是没有返回的吗？那这里设置的返回值保存在哪里？另外如果没有返回，该系统调用又要如何返回到用户程序中？关键在于 `env_run` 函数。在进程切换之前，会将 trap frame 存储到 `env->env_tf` 中。当重新轮到该进程运行的时候，就会从 `env_tf` 存储的位置恢复现场。这里也重新强调了 `sys_exofork` 中为什么不能使用 `env_tf`。

```c
void env_run(struct Env *e) {
	assert(e->env_status == ENV_RUNNABLE);
	if (curenv) {
		curenv->env_tf = *((struct Trapframe *)KSTACKTOP - 1);	// 其中的regs[2]被设置为0了
	}
	curenv = e;
	curenv->env_runs++; // lab6
	cur_pgdir = curenv -> env_pgdir;
	env_pop_tf(&(curenv -> env_tf), curenv -> env_asid);
}
```

### 信息发送

`sys_ipc_try_send` 函数的功能是传递一个 value 和一个位于 srcva 的页面。在 `sys_ipc_try_send` 中我们同样判断地址是否正确，并通过 envid 获取进程控制块 (等待接收信息的进程)。需要注意这里 `envid2env` 的 `checkperm` 参数为 0，而此前所有的参数值均为 1，这是因为进程通信不一定只在父子进程之间。

```c
int sys_ipc_try_send(u_int envid, u_int value, u_int srcva, u_int perm) {
	struct Env *e;
	struct Page *p;
	if (srcva != 0 && is_illegal_va(srcva)) {
		return -E_INVAL;
	}
	try(envid2env(envid, &e, 0));
```

然后我们检查 `env_ipc_recving`，这一字段在信息接收时设置。

```c
	if (e -> env_ipc_recving == 0) {
		return -E_IPC_NOT_RECV;
	}
```

接下来我们传输一些信息，并将 `env_ipc_recving` 重新置 0，表示接收进程已经接收到信息。

```c
	e->env_ipc_value = value;
	e->env_ipc_from = curenv->env_id;
	e->env_ipc_perm = PTE_V | perm;
	e->env_ipc_recving = 0;
```

既然接收到了信息，那么我们就要取消接收进程的阻塞状态。

```c
	e -> env_status = ENV_RUNNABLE;
	TAILQ_INSERT_TAIL(&env_sched_list, e, env_sched_link);
```

最后，我们还需要将当前进程的一个页面共享到接收进程。只有这样，接收进程才能通过该页面获得发送进程发送的一些信息。

```c
	if (srcva != 0) {
		p = page_lookup(curenv -> env_pgdir, srcva, NULL);
		if (p == NULL) {
			return -E_INVAL;
		}
		try(page_insert(e->env_pgdir, e->env_asid, p, e->env_ipc_dstva, perm));
	}
	return 0;
}
```

### 进程通信总览

![截屏2025-05-08 10.05.23](./images_report/%E6%88%AA%E5%B1%8F2025-05-08%2010.05.23.png)