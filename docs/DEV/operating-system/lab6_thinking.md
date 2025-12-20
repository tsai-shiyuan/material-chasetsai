# Shell

> Fork from [Wokron's blog](https://wokron.github.io/posts/buaa-os-lab6/)

## 管道

在 Unix 中，管道由 `int pipe(int fd[2])` 函数创建，成功创建管道返回0，`fd[0]` 对应读端，`fd[1]` 对应写端。父进程调用 `pipe` 函数时，会打开两个新文件描述符，只读端和只写端，两个文件描述符都映射到同一片内存区域。在管道的读写过程中，要保证父子进程通过管道访问的内存相同。

### 创建管道

管道的相关操作位于 user/lib/pipe.c 中。创建管道的函数为 pipe。

pipe函数首先分配两个文件描述符 fd0 和 fd1，并为其分配空间，然后给 fd0 对应的虚拟地址分配一页物理内存，再将 fd1 对应的虚拟地址映射到这一页上。这个物理页是**共享页面**，具有权限位 PTE_LIBRARY，当父子进程试图写共享页面时，直接在该页面上进行写操作即可。**共享页面存放 Pipe 结构体**（我们可以发现之后常常使用 `(struct Pipe *) fd2data(fd)` ）。

```c
int pipe(int pfd[2]) {
	int r;
	void *va;
	struct Fd *fd0, *fd1;

	if ((r = fd_alloc(&fd0)) < 0 || 
        (r = syscall_mem_alloc(0, fd0, PTE_D | PTE_LIBRARY)) < 0) {
		goto err;
	}
	if ((r = fd_alloc(&fd1)) < 0 || 
        (r = syscall_mem_alloc(0, fd1, PTE_D | PTE_LIBRARY)) < 0) {
		goto err1;
	}
    
    va = fd2data(fd0);
	if ((r = syscall_mem_alloc(0, (void *)va, PTE_D | PTE_LIBRARY)) < 0) {
		goto err2;
	}
	if ((r = syscall_mem_map(0, (void *)va, 0, 
                             (void *)fd2data(fd1), PTE_D | PTE_LIBRARY)) < 0) {
		goto err3;
	}
```

接着设定文件描述符的相关属性，明确各自是只读还是只写，并且把两个 Fd 对象的设备类型设置为 devpipe，

```c
	fd0->fd_dev_id = devpipe.dev_id;
	fd0->fd_omode = O_RDONLY;

	fd1->fd_dev_id = devpipe.dev_id;
	fd1->fd_omode = O_WRONLY;
```

devpipe 结构体定义为:

```c
struct Dev devpipe = {
    .dev_id = 'p',
    .dev_name = "pipe",
    .dev_read = pipe_read,
    .dev_write = pipe_write,
    .dev_close = pipe_close,
    .dev_stat = pipe_stat,
};
```

最后保存文件描述符 Fd 的编号并返回。

```c
	pfd[0] = fd2num(fd0);
	pfd[1] = fd2num(fd1);
	return 0;

err3: syscall_mem_unmap(0, (void *)va);
err2: syscall_mem_unmap(0, fd1);
err1: syscall_mem_unmap(0, fd0);
err: return r;
}
```

### 关闭管道

关闭 fd 指向的管道即取消了 fd 到物理页的映射以及 fd 对应的 data 到物理页的映射。发现进行了两次 unmap 解除映射，但是两次系统调用之间可能因为进程切换而被打断！这会影响我们判断管道是否关闭的正确性。必须要先 unmap fd 再 unmap pipe 才能保证正确判断。（细节在指导书思考题里）

```c
// Close the pipe referred by 'fd'
static int pipe_close(struct Fd *fd) {
	syscall_mem_unmap(0, fd);
	syscall_mem_unmap(0, (void *)fd2data(fd));
	return 0;
}
```

检测某个管道是否被关闭，关键调用了 `_pipe_is_closed` 函数。

```c
int pipe_is_closed(int fdnum) {
	struct Fd *fd;
	struct Pipe *p;
	int r;
	if ((r = fd_lookup(fdnum, &fd)) < 0) {
		return r;
	}
	p = (struct Pipe *)fd2data(fd);
	return _pipe_is_closed(fd, p);
}
```

在创建管道的时候，我们分配了三页物理空间: 一页是读数据的文件描述符 rfd，一页是写数据的文件描述符 wfd， 还有被两个文件描述符共享的管道数据缓冲区 pipe。有下面的恒等式:
$$
\mathrm{pageref(rfd) + pageref(wfd) = pageref(pipe)}
$$
假设现在读者进程正在执行，而写者进程关闭了管道，那么此时有 $\mathrm{pageref(wfd)} = 0$ ，进而 $\mathrm{pageref(rfd) = pageref(pipe)}$ ，只要判断这个等式是否成立，就能知道另一端是否关闭。那我们的 `_pipe_is_closed` 函数可以这样写

```c
int _pipe_is_closed(struct Fd *fd, struct Pipe *p) {
    int fd_ref = pageref(fd);
    int pipe_ref = pageref(p);
    return fd_ref == pipe_ref;
}
```

这对么？在执行 pageref(fd) 和 pageref(p) 之间可能发生进程的切换，也可能导致判断 ref 等但是实际并没有关闭管道，为了保证同步性，我们要确认两次读取之间进程没有切换。我们用到 env 结构体中的 env_runs，它记录了一个进程调度运行的次数。修改为:

```c
static int _pipe_is_closed(struct Fd *fd, struct Pipe *p) {
	int fd_ref, pipe_ref, runs;
	while (1) {
		runs = env->env_runs;
		fd_ref = pageref(fd);
		pipe_ref = pageref(p);
		if (runs == env->env_runs) break;
	}
	return fd_ref == pipe_ref;
}
```

### 管道的读写

Pipe 结构体的内容为:

```c
struct Pipe {
	u_int p_rpos;		 // read position
	u_int p_wpos;		 // write position
	u_char p_buf[PIPE_SIZE];	// data buffer
};
```

只有读者可以更新 p_rpos，只有写者可以更新 p_wpos，有 PIPE_SIZE (32 Byte) 大小的缓冲区。读或写位置 `i` 实际上是对 `p_buf[i%PIPE_SIZE]` 位置进行操作，这样节省了队列的空间。

读者需要在队列中有数据时才能读，即 `p_rpos < p_wpos` ，否则应该切换到写者运行。写者必须在队列中有空位时才能写，即 `p_wpos - p_rpos < PIPE_SIZE` 时。具体的读写操作如下

pipe_read 从 fd 指向的管道读取最多 n 字节到 vbuf 中；

```c
static int pipe_read(struct Fd *fd, void *vbuf, u_int n, u_int offset) {
	int i;
	struct Pipe *p;
	char *rbuf;

	p = (struct Pipe *) fd2data(fd);
	i = 0;
	rbuf = (char *) vbuf;
	while (1) {
		if (i >= n) return i;
		while (p->p_rpos >= p->p_wpos) {
			if (i > 0 || _pipe_is_closed(fd, p)) return i;
			syscall_yield();
		}
		rbuf[i++] = p->p_buf[(p->p_rpos++) % PIPE_SIZE];
	}
	user_panic("pipe_read not implemented");
}
```

pipe_write 把 vbuf 中的 n 个字节写入 fd 指向的管道。

```c
static int pipe_write(struct Fd *fd, const void *vbuf, u_int n, u_int offset) {
	int i;
	struct Pipe *p;
	char *wbuf;

	p = (struct Pipe *) fd2data(fd);
	i = 0;
	wbuf = (char *) vbuf;
	while (1) {
		if (i >= n) return i;
		while (p->p_wpos - p->p_rpos >= PIPE_SIZE) {
			if (_pipe_is_closed(fd, p)) return i;
			syscall_yield();
		}
		p->p_buf[(p->p_wpos++) % PIPE_SIZE] = wbuf[i++];
	}
	user_panic("pipe_write not implemented");
	return n;
}
```

注意到读写函数都进行了 `_pipe_is_closed` 判断，主要是解决这种情况：假设管道写端已经全部关闭，读者读到缓冲区有效数据的末尾，此时有 p_rpos = p_wpos。按照上面的做法，我们这里应当切换到写者运行。但写者进程已经结束，进程切换就造成了死循环。

## 二进制文件的加载与执行

spawn 的作用是调用文件系统中的可执行文件并执行。会从文件系统打开对应的文件（二进制 ELF，MOS中的 *.b），创建子进程并将目标程序加载到子进程的地址空间中，设置子进程的寄存器 (栈指针 sp 和用户程序入口 EPC)，然后设置子进程为可执行态。

### 子进程栈的初始化

![截屏2025-06-12 22.31.17](./images_report/%E6%88%AA%E5%B1%8F2025-06-12%2022.31.17.png)

在 init_stack 函数中，我们要将 argc, argv 的数据写入要初始化的进程的栈中，以完成进程栈的初始化。主要思路是先将 argc, argv 写入 UTEMP 页 (就在 UCOW 下面，用途也和 UCOW 那页类似)，再将 UTEMP 映射到子进程的栈。

首先计算参数所需内存，如果超过一页直接返回异常。

```c
int init_stack(u_int child, char **argv, u_int *init_sp) {
	int argc, i, r, tot;
	char *strings;
	u_int *args;
    
	// Count the number of arguments (argc)
	// and the total amount of space needed for strings (tot)
	tot = 0;
	for (argc = 0; argv[argc]; argc++) {
		tot += strlen(argv[argc]) + 1;
    }
    if (ROUND(tot, 4) + 4 * (argc + 3) > PAGE_SIZE) {
		return -E_NO_MEM;
	}
```

接下来确定字符串和指针数组的地址，并为 UTEMP 申请了页面。

```c
	strings = (char *)(UTEMP + PAGE_SIZE) - tot;
	args = (u_int *)(UTEMP + PAGE_SIZE - ROUND(tot, 4) - 4 * (argc + 1));

	if ((r = syscall_mem_alloc(0, (void *)UTEMP, PTE_D)) < 0) {
		return r;
	}
```

接着复制了所有参数字符串。

```c
	char *ctemp, *argv_temp;
	u_int j;
	ctemp = strings;
	for (i = 0; i < argc; i++) {
		argv_temp = argv[i];
		for (j = 0; j < strlen(argv[i]); j++) {
			*ctemp = *argv_temp;
			ctemp++;
			argv_temp++;
		}
		*ctemp = 0;
		ctemp++;
	}
```

这一部分设置了指针数组的内容。

```c
	// Initialize args[0..argc-1] to be pointers to these strings
	// that will be valid addresses for the child environment
	// (for whom this page will be at USTACKTOP-PAGE_SIZE!).
	ctemp = (char *)(USTACKTOP - UTEMP - PAGE_SIZE + (u_int)strings);
	for (i = 0; i < argc; i++) {
		args[i] = (u_int)ctemp;
		ctemp += strlen(argv[i]) + 1;
	}
	// Set args[argc] to 0 to null-terminate the args array.
	ctemp--;
	args[argc] = (u_int)ctemp;
```

最后设置 `argc` 的值和 `argv` 数组指针的内容，将 `UTEMP` 页映射到用户栈真正应该处于的地址

```c
	// Push two more words onto the child's stack below 'args',
	// containing the argc and argv parameters to be passed
	// to the child's main() function.
	u_int *pargv_ptr;
	pargv_ptr = args - 1;
	*pargv_ptr = USTACKTOP - UTEMP - PAGE_SIZE + (u_int)args;
	pargv_ptr--;
	*pargv_ptr = argc;

	// Set *init_sp to the initial stack pointer for the child
	*init_sp = USTACKTOP - UTEMP - PAGE_SIZE + (u_int)pargv_ptr;
	if ((r = syscall_mem_map(0, (void *)UTEMP, child, 
                             (void *)(USTACKTOP - PAGE_SIZE), PTE_D)) < 0) {
		goto error;
	}
	if ((r = syscall_mem_unmap(0, (void *)UTEMP)) < 0) {
		goto error;
	}
	return 0;
error:
	syscall_mem_unmap(0, (void *)UTEMP);
	return r;
}
```

### spawn函数

spawn 函数位于 user/lib/spawn.c 中。首先根据文件路径 `prog` 打开文件，通过 `readn` 读取其文件头的信息。

```c
int spawn(char *prog, char **argv) {
	int fd;
	if ((fd = open(prog, O_RDONLY)) < 0) {
		return fd;
	}
	int r;
	u_char elfbuf[512];
	if ((r = readn(fd, elfbuf, sizeof(Elf32_Ehdr))) < 0 || (r != sizeof(Elf32_Ehdr))) {
		goto err;
	}
    const Elf32_Ehdr *ehdr = elf_from(elfbuf, sizeof(Elf32_Ehdr));
	if (!ehdr) {
		r = -E_NOT_EXEC;
		goto err;
	}
    u_long entrypoint = ehdr->e_entry;
```

接着使用系统调用创建一个新的进程。注意这里不需要像 `fork` 一样创建后判断 `child` 的值来区分父子进程。因为在 `spawn` 之后的内容中我们会替换子进程的代码和数据，不会再从此处继续执行。

```c
	u_int child;
	if ((child = syscall_exofork()) < 0) {
		r = child;
		goto err;
	}
```

随后在父进程中，调用 `init_stack` 完成子进程栈的初始化。

```c
	u_int sp;
	if ((r = init_stack(child, argv, &sp)) < 0) {
		goto err1;
	}
```

接下来我们遍历整个 ELF 头的程序段，将程序段的内容读到内存中。如果是需要加载的程序段，则首先根据程序段相对于文件的偏移得到其在内存中映射到的地址，接着就和 Lab3 的 `load_icode` 函数一样，调用 `elf_load_seg` 将程序段加载到适当的位置。（但还是要注意和 `load_icode` 的区别。此处我们设定的回调函数为 `spawn_mapper` 而非 `load_icode_mapper` ，`spawn_mapper` 处于用户态，所以使用了很多系统调用，同时传入的是子进程的进程 id 而非进程控制块）

```c
	size_t ph_off;
	ELF_FOREACH_PHDR_OFF (ph_off, ehdr) {
		// Read the program header in the file with offset 'ph_off' and length
		// 'ehdr->e_phentsize' into 'elfbuf'
		if ((r = seek(fd, ph_off)) < 0 || (r = readn(fd, elfbuf, ehdr->e_phentsize)) < 0) {
			goto err1;
		}
		Elf32_Phdr *ph = (Elf32_Phdr *)elfbuf;
		if (ph->p_type == PT_LOAD) {
			void *bin;
			// Read and map the ELF data in the file at 'ph->p_offset' into our memory
			if ((r = read_map(fd, ph->p_offset, &bin)) < 0) {
				goto err1;
			}
			// Load the segment 'ph' into the child's memory using 'elf_load_seg()'
			if ((r = elf_load_seg(ph, bin, spawn_mapper, &child)) < 0) {
				goto err1;
			}
		}
	}
	close(fd);
```

这样就将程序加载到了新创建的进程的适当位置了。之后设定栈帧，父子进程共享 `USTACKTOP` 地址之下的数据，这一部分和 `duppage` 的操作很相似，只不过这里不共享程序部分。

```c
	struct Trapframe tf = envs[ENVX(child)].env_tf;
	tf.cp0_epc = entrypoint;
	tf.regs[29] = sp;
	if ((r = syscall_set_trapframe(child, &tf)) != 0) {
		goto err2;
	}
	for (u_int pdeno = 0; pdeno <= PDX(USTACKTOP); pdeno++) {
		if (!(vpd[pdeno] & PTE_V)) {
			continue;
		}
		for (u_int pteno = 0; pteno <= PTX(~0); pteno++) {
			u_int pn = (pdeno << 10) + pteno;
			u_int perm = vpt[pn] & ((1 << PGSHIFT) - 1);
			if ((perm & PTE_V) && (perm & PTE_LIBRARY)) {
				void *va = (void *)(pn << PGSHIFT);

				if ((r = syscall_mem_map(0, va, child, va, perm)) < 0) {
					debugf("spawn: syscall_mem_map %x %x: %d\n", va, child, r);
					goto err2;
				}
			}
		}
	}
```

最后设定子进程为运行状态以将其加入进程调度队列，实现子进程的创建。

```c
	if ((r = syscall_set_env_status(child, ENV_RUNNABLE)) < 0) {
		debugf("spawn: syscall_set_env_status %x: %d\n", child, r);
		goto err2;
	}
	return child;
```

在最后则是异常处理部分，可以看出在 `spawn` 函数中因为分配的资源数量较多，异常处理部分也变复杂了。这里就可以看出使用 `goto` 进行异常处理的优点了。只需要按反向的顺序写出资源的释放函数，在异常时只需要跳转到对应位置就可以穿透标签，不断释放所有的资源。

```c
err2:
	syscall_env_destroy(child);
	return r;
err1:
	syscall_env_destroy(child);
err:
	close(fd);
	return r;
}
```

## Shell 进程的启动

### 共同祖先进程

这次我们还要回到 init/init.c 文件。我们的 MOS 的所有实验都结束之后，`mips_init` 函数应该是这样的

```c
void mips_init(u_int argc, char **argv, char **penv, u_int ram_low_size) {
	printk("init.c:\tmips_init() is called\n");

	// lab2:
	mips_detect_memory(ram_low_size);
	mips_vm_init();
	page_init();

	// lab3:
	env_init();

	// lab6:
	ENV_CREATE(user_icode);  // This must be the first env!

	// lab5:
	ENV_CREATE(fs_serv);  // This must be the second env!

	// lab3:
	schedule(0);
	halt();
}
```

其中我们使用 `ENV_CREATE` 创建了两个用户进程，user_icode 和 fs_serv，这两个进程的代码在编译时便写入了内核 ELF 文件中。第一个进程 `user_icode` 则是整个操作系统中除文件系统服务进程外所有进程的共同祖先进程，该进程便用于启动 Shell 进程。对应的文件位于 user/icode.c 中。

首先读取 motd 文件并输出其内容，motd 即 "message of today" 的缩写，这一步只是为了打印欢迎信息。

```c
int main() {
	int fd, n, r;
	char buf[512 + 1];
	debugf("icode: open /motd\n");
	if ((fd = open("/motd", O_RDONLY)) < 0) {
		user_panic("icode: open /motd: %d", fd);
	}
	debugf("icode: read /motd\n");
	while ((n = read(fd, buf, sizeof buf - 1)) > 0) {
		buf[n] = 0;
		debugf("%s\n", buf);
	}
	debugf("icode: close /motd\n");
	close(fd);
```

之后调用 `spawnl` 函数执行二进制文件 `init.b`。

```c
	debugf("icode: spawn /init\n");
	if ((r = spawnl("init.b", "init", "initarg1", "initarg2", NULL)) < 0) {
		user_panic("icode: spawn /init: %d", r);
	}
	debugf("icode: exiting\n");
	return 0;
}
```

### 进程初始化

执行的 `init.b` 文件的源代码在 user/init.c 中。其中前面一大部分都是测试 ELF 文件加载正确性的代码，就略过不讲了。重要的部分从这里开始。

```c
int main(int argc, char **argv) {
	int i, r, x, want;
	// omit...
    // stdin should be 0, because no file descriptors are open yet
	if ((r = opencons()) != 0) {
		user_panic("opencons: %d", r);
	}
```

首先调用了 `opencons` 打开一个终端文件，函数位于 user/lib/console.c 中（和 pipe 很像，都有自己的 read, write 等函数），申请了一个文件描述符，并设置 `fd->fd_dev_id = devcons.dev_id` 表示该文件属于终端文件，执行读写等操作时使用终端提供的函数。

```c
int opencons(void) {
	int r;
	struct Fd *fd;
	if ((r = fd_alloc(&fd)) < 0) {
		return r;
	}
	if ((r = syscall_mem_alloc(0, fd, PTE_D | PTE_LIBRARY)) < 0) {
		return r;
	}
	fd->fd_dev_id = devcons.dev_id;
	fd->fd_omode = O_RDWR;
	return fd2num(fd);
}
```

对于终端的读写操作，以写为例，对应的函数为 `cons_write`，本质上是通过系统调用实现读写。

```c
struct Dev devcons = {
    .dev_id = 'c',
    .dev_name = "cons",
    .dev_read = cons_read,
    .dev_write = cons_write,
    .dev_close = cons_close,
    .dev_stat = cons_stat,
};

int cons_write(struct Fd *fd, const void *buf, u_int n, u_int offset) {
	int r = syscall_print_cons(buf, n);
	if (r < 0) {
		return r;
	}
	return n;
}
```

让我们回到 user/init.c。因为调用 `opencons` 时系统内必定没有其他文件被打开，所以该函数申请得到的文件描述符必定为 0。

之后代码中又调用 `dup` 函数申请了一个新的文件描述符 1。该文件描述符是 0 的复制。也就是说，对这两个文件描述符进行读写等文件操作，都是对同一个文件（这里是终端文件）的操作。

```c
	if ((r = opencons()) != 0) {
		user_panic("opencons: %d", r);
	}
	// stdout
	if ((r = dup(0, 1)) < 0) {
		user_panic("dup: %d", r);
	}
```

`dup` 函数位于 user/lib/fd.c 中。在最开始做了一系列准备工作，首先是根据已有的文件描述符 `oldfdnum` 找到对应的文件描述结构体。

```c
int dup(int oldfdnum, int newfdnum) {
	int i, r;
	void *ova, *nva;
	u_int pte;
	struct Fd *oldfd, *newfd;
	if ((r = fd_lookup(oldfdnum, &oldfd)) < 0) {
		return r;
	}
```

之后需要注意我们调用了 `close` 函数关闭了要复制的文件描述符 `newfdnum` 原本对应的文件（如果有的话）。只有这样我们才能将 `oldfdnum` 对应的文件复制给 `newfdnum`。最后我们通过 `fd2data` 函数根据描述符得到文件内容所在的地址位置。

```c
	close(newfdnum);
	newfd = (struct Fd *)INDEX2FD(newfdnum);
	ova = fd2data(oldfd);
	nva = fd2data(newfd);
```

我们还要共享文件描述符和文件内容，首先映射文件中有效的页。

```c
	if (vpd[PDX(ova)]) {
		for (i = 0; i < PDMAP; i += PTMAP) {
			pte = vpt[VPN(ova + i)];
			if (pte & PTE_V) {				
				if ((r = syscall_mem_map(0, (void *)(ova + i), 0, (void *)(nva + i),
							 pte & (PTE_D | PTE_LIBRARY))) < 0) {
					goto err;
				}
			}
		}
	}
```

再共享文件描述符，通过系统调用 `syscall_mem_map` ，将 `oldfd` 所在的页映射到 `newfd` 所在的地址。需要注意这里的 `vpt[VPN(oldfd)] & (PTE_D | PTE_LIBRARY)` 表示为 `newfd` 设置与 `oldfd` 相同的可写可共享权限。全部都映射完成后，返回新的文件描述符。

```c
	if ((r = syscall_mem_map(0, oldfd, 0, newfd, vpt[VPN(oldfd)] & 
                             (PTE_D | PTE_LIBRARY))) < 0) {
		goto err;
	}
	return newfdnum;
```

`dup` 函数中使用到了 `goto` 进行异常处理，当出错后，会跳转到 `err` 标签的位置，解除已经建立的映射。

```c
err:
	panic_on(syscall_mem_unmap(0, newfd));
	for (i = 0; i < PDMAP; i += PTMAP) {
		panic_on(syscall_mem_unmap(0, (void *)(nva + i)));
	}
	return r;
}
```

让我们再回到 user/init.c。现在我们有了 0、1 两个文件描述符，分别表示了标准输入和标准输出（只是我们人为规定）。此后进程都会通过 `spawn` 或 `fork` 创建进程，这些新创建的进程也会继承两个标准输入输出，除非进程自己关闭。

最后是一个死循环，这个循环用于创建 Shell 进程。在调用 `spawnl` 创建 Shell 进程成功后，会调用 `wait` 等待 Shell 进程退出。因为处于循环中，所以当 Shell 进程退出后，又会创建一个新的 Shell 进程。

```c
	while (1) {
		debugf("init: starting sh\n");
		r = spawnl("sh.b", "sh", NULL);
		if (r < 0) {
			debugf("init: spawn sh: %d\n", r);
			return r;
		}
		wait(r);
	}
}
```

其中 `wait` 定义在 user/lib/wait.c 中。只是实现了一个简单的忙等，使得调用 `wait` 的函数被阻塞，直到 `envid` 对应的进程退出才继续执行。

```c
void wait(u_int envid) {
	const volatile struct Env *e;
	e = &envs[ENVX(envid)];
	while (e->env_id == envid && e->env_status != ENV_FREE) {
		syscall_yield();
	}
}
```

这样 Shell 进程就启动了。

## Shell 进程

Shell 进程不断地读取命令，然后 fork 出子进程来执行命令。

### main

```bash
main
 └─> readline → 获取命令
 └─> fork
     ├─ 子进程 → runcmd(buf) → 解释命令并执行（可能处理重定向、管道等）
     └─ 父进程 → wait 子进程
```

ARGBEGIN, ARGEND 宏和其中间的部分解析了命令中的选项，`interactive` 指示是否为交互式（也即标准输入是不是终端），`echocmds` 表示是否在执行前显示指令。中间 `argc == 1` 表示非交互式地，从脚本文件读取命令，关闭了 fd 0，然后再 open，就能将 fd 0 分配给打开的文件，进而将标准输入设置为打开的脚本。（感觉不会用到？）中间循环 readline $\rightarrow$ fork $\rightarrow$ 子进程 runcmd

```c
char buf[1024];

int main(int argc, char **argv) {
	int r;
	int interactive = iscons(0);	// 判断标准输入 fd 0 是否为终端设备
	int echocmds = 0;
	printf("\n:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::\n");
	printf("::                                                         ::\n");
	printf("::                     MOS Shell 2024                      ::\n");
	printf("::                                                         ::\n");
	printf(":::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::\n");
	ARGBEGIN {
	case 'i':
		interactive = 1;
		break;
	case 'x':
		echocmds = 1;
		break;
	default:
		usage();
	}
	ARGEND

	if (argc > 1) {
		usage();
	}
	if (argc == 1) {
		close(0);
		if ((r = open(argv[0], O_RDONLY)) < 0) {	
			user_panic("open %s: %d", argv[0], r);
		}
		user_assert(r == 0);
	}
	for (;;) {
		if (interactive) {
			printf("\n$ ");
		}
		readline(buf, sizeof buf);

		if (buf[0] == '#') {
			continue;
		}
		if (echocmds) {
			printf("# %s\n", buf);
		}
		if ((r = fork()) < 0) {
			user_panic("fork: %d", r);
		}
		if (r == 0) {
			runcmd(buf);
			exit();
		} else {
			wait(r);
		}
	}
	return 0;
}

void usage(void) {
	printf("usage: sh [-ix] [script-file]\n");
	exit();
}
```

### readline

`readline` 函数会逐字符读取。需要注意当标准输入未被重定向时（即从终端进行读取时），这一过程是和用户的输入同步的。当用户输入一个字符，`read` 才会读取一个字符，否则会被阻塞。

中间遇到 `\b` 的时候执行了 `i-=2` 或者 `i = -1` ，接着又 `buf[i]`，难道不会数组越界吗？不懂。

```c
void readline(char *buf, u_int n) {
	int r;
	for (int i = 0; i < n; i++) {
		if ((r = read(0, buf + i, 1)) != 1) {	// 从标准输入 fd0 逐字节读入
			if (r < 0) {
				debugf("read error: %d\n", r);
			}
			exit();
		}
		if (buf[i] == '\b' || buf[i] == 0x7f) {	// 遇到"退格"或"delete"
			if (i > 0) {
				i -= 2;
			} else {
				i = -1;
			}
			if (buf[i] != '\b') {
				printf("\b");
			}
		}
		if (buf[i] == '\r' || buf[i] == '\n') {
			buf[i] = 0;
			return;
		}
	}
	debugf("line too long\n");
	while ((r = read(0, buf, 1)) == 1 && buf[0] != '\r' && buf[0] != '\n') {
		;
	}
	buf[0] = 0;
}
```

阻塞的实现:

```c
int cons_read(struct Fd *fd, void *vbuf, u_int n, u_int offset) {
	int c;
	if (n == 0) {
		return 0;
	}
	while ((c = syscall_cgetc()) == 0) {
		syscall_yield();
	}
	if (c != '\r') {
		debugf("%c", c);
	} else {
		debugf("\n");
	}
	if (c < 0) {
		return c;
	}
	if (c == 0x04) { // ctl-d is eof
		return 0;
	}
	*(char *)vbuf = c;
	return 1;
}
```

接下来就该由子进程来 runcmd，我们先来看一些工具函数。

### _gettoken

这是底层的词法分析函数，将字符串 `s` 按照空格 WHITESPACE 和特殊符号 SYMBOLS 切分成单词或符号。`*p1` 指向当前 token 开始处，`*p2` 指向下一个符号开始处

```c
#define WHITESPACE " \t\r\n"
#define SYMBOLS "<|>&;()#"

int _gettoken(char *s, char **p1, char **p2) {
	*p1 = 0;
	*p2 = 0;
	if (s == 0) {
		return 0;
	}
	while (strchr(WHITESPACE, *s)) {	// 跳过前导 WHITESPACE
		*s++ = 0;
	}
	if (*s == 0) {
		return 0;	// 字符串结束
	}
	if (strchr(SYMBOLS, *s)) {
		int t = *s;
		*p1 = s;
		*s++ = 0;
		*p2 = s;
		return t;	// 返回字符本身
	}
	*p1 = s;
	while (*s && !strchr(WHITESPACE SYMBOLS, *s)) {
		s++;
	}
	*p2 = s;
	return 'w';		// 普通单词
}
```

### gettoken

第一次调用 gettoken 时会传入完整的字符串，初始化静态变量，例如: `gettoken(s, 0)` ；后续调用时传入 `s = 0`，就会返回上一次解析的 token，并更新为下一个 token，例如: `gettoken(0, &t)`。

`*p1` 返回当前 token 的起始位置，函数返回 token 的类型。

```c
int gettoken(char *s, char **p1) {
	static int c, nc;
	static char *np1, *np2;

	if (s) {
		nc = _gettoken(s, &np1, &np2);
		return 0;
	}
	c = nc;
	*p1 = np1;
	nc = _gettoken(np2, &np1, &np2);
	return c;
}
```

### runcmd

```bash
runcmd (char *s)
├─ gettoken (char *s, char **p1)
	└─ _gettoken (char *s, char **p1, char **p2)
├─ parsecmd (char **argv, int *rightpipe)
├─ spawn (argv[0], argv) - 左侧命令执行
├─ wait(child)
├─ wait(rightpipe) - 如果有管道
└─ exit()
```

可以发现，runcmd 并不自己处理管道，而是由 parsecmd 来处理，rightpipe 保存右边子进程的 id。

```c
void runcmd(char *s) {
	gettoken(s, 0);		// 最最初始的解析
    
	char *argv[MAXARGS];
	int rightpipe = 0;	// 用于处理管道 |
	int argc = parsecmd(argv, &rightpipe);	// 实际解析的参数个数
	if (argc == 0) {
		return;
	}
	argv[argc] = 0;

	int child = spawn(argv[0], argv);	// 执行指令
	if (child < 0) {
		char new_cmd[1024] = {0};
		strcpy(new_cmd, argv[0]);
		int len = strlen(argv[0]);
		new_cmd[len] = '.';
		new_cmd[len + 1] = 'b';
		new_cmd[len + 2] = '\0'; // 将ls扩展成ls.b
		child = spawn(new_cmd, argv);
	}
	close_all();

	if (child >= 0) {
		wait(child);
	} else {
		debugf("spawn %s: %d\n", argv[0], child);
	}
	if (rightpipe) {
		wait(rightpipe);
	}
	exit();
}
```

### parsecmd

循环提取 token 并解析，解析出来的当前命令的的参数列表存储到 argv。

```c
int parsecmd(char **argv, int *rightpipe) {
	int argc = 0;
	while (1) {
		char *t;
		int fd, r;
		int c = gettoken(0, &t);	// c 是类型, t 指向token的起始
		switch (c) {
		case 0:
		case '#':
			return argc;
		case 'w':
			if (argc >= MAXARGS) {
				debugf("too many arguments\n");
				exit();
			}
			argv[argc++] = t;
			break;
		case ';':
			int child = fork();
			if (child == 0) {
				return argc;
			} else if (child > 0) {
				wait(child);
				close(0);
				close(1);
				dup(opencons(), 1);		
				dup(1, 0);
				return parsecmd(argv, rightpipe);
			}
			break;
		case '<':
			if (gettoken(0, &t) != 'w') {
				debugf("syntax error: < not followed by word\n");
				exit();
			}
			// Open 't' for reading, dup it onto fd 0, and then close the original fd
			if ((fd = open(t, O_RDONLY)) < 0) {
				debugf("error opening\n");
				exit();
			}
			dup(fd, 0);
			close(fd);
			break;
			user_panic("< redirection not implemented");
			break;
		case '>':
			if (gettoken(0, &t) != 'w') {
				debugf("syntax error: > not followed by word\n");
				exit();
			}
			// Open 't' for writing, create it if not exist and trunc it if exist, dup
			// it onto fd 1, and then close the original fd
			if ((fd = open(t, O_WRONLY)) < 0) {
				debugf("error opening\n");
				exit();
			}
			dup(fd, 1);
			close(fd);
			break;
			user_panic("> redirection not implemented");
			break;
		case '|':;
			int p[2];
			pipe(p);
			*rightpipe = fork();	// 设置为右边子进程的 envid
			if (*rightpipe == 0) {	// 子进程 run 右边的命令
				dup(p[0], 0);		// 子进程的输入来自管道的读端
				close(p[0]);
				close(p[1]);
				return parsecmd(argv, rightpipe);
			} else if (*rightpipe > 0) {	// 本进程 run 左边的命令
				dup(p[1], 1);		// 输出到管道的写端
				close(p[1]);
				close(p[0]);
				return argc;
			}
			break;
			user_panic("| not implemented");
			break;
		}
	}
	return argc;
}
```

**（完）**