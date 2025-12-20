# P2-MIPS汇编

## 问题

怎么用 `li $v0, 12` 读字符串并存入字符串数组？

## 课下

闲言碎语

课下花了很长很长的时间 debug，全排列 (周二晚上) & 地图 (周三上午) & 高精度乘法 (周三下午+晚上)，我觉得我出错的原因在于，没有正确地使用宏。我本来以为用宏封装，能让代码更清楚，正确率更高，然而事实相反—— 高精度乘法没用宏写，只花了十几分钟，并且成功了一部分数据 (其他就不是 MIPS 使用的问题了，是 C 代码的问题)，所以使用宏的时候一定要清醒细心！

### 递归

在函数调用的时候 push，调用完立刻 pop

```c
vis[i] = 1; // 先标记本数不能被之后的dfs填
stack[step] = i;    
dfs(step + 1);
vis[i] = 0; 
```

```assembly
setVisOne($t0)			
setStack($t0, $a0)
			
push($ra)
push($a0)
push($t0)
addi $a0, $a0, 1	# change parameter
jal dfs	# here we go
pop($t0)
pop($a0)
pop($ra)

setVisZero($t0)
```

注意 `syscall`  的时候可能会改 `$a0` ，也要压进栈

### 常用的宏

- 二维数组取地址

```assembly
.macro INDEX(%ans, %i, %j, %col)
	multu %i, %col
	mflo %ans
	add %ans, %ans, %j
	sll %ans, %ans, 2
.end_macro
```

- 读入和输出整数

```assembly
.macro RI(%des)
	li $v0, 5
	syscall
	move %des, $v0
.end_macro

.macro PI(%src)
	move $a0, %src
	li $v0, 1
	syscall
.end_macro
```

- 进栈出栈

```assembly
.macro push(%src)
	addi $sp, $sp, -4
	sw %src, 0($sp)
.end_macro

.macro pop(%des)
	lw %des, 0($sp)
	addi $sp, $sp, 4
.end_macro
```

- 输出空白和换行

```assembly
.data
	space: .asciiz " "
	enter: .asciiz "\n"

.macro P_SPACE
	la $a0, space
	li $v0, 4
	syscall
.end_macro

.macro P_ENTER
	la $a0, enter
	li $v0, 4
	syscall
.end_macro
```

### 宏的使用

宏不应该改变除 `%des`  以外的寄存器的值

eg. 输出换行的代码中 `la $a0, space`  修改了 `$a0` ，在有函数调用的时候，会出现矛盾。稳妥写法:

```assembly
.macro enter
	push($a0)
	la $a0, enter
	li $v0, 4
	syscall
	pop($a0)
.end_macro
```

宏中出现过的寄存器就不能设为 `des`  ，get 一定不行，set 要判断，所以最好避免

eg1. 如果调用 `getVis($t3, $t0)` ， `$t3` 最后弹出原来的值

```assembly
.macro getVis(%des, %i)
	push($t3)
	sll $t3, %i, 2
	lw %des, vis($t3)
	pop($t3)
.end_macro
```

eg2. 若 `%des = $t3` ， `sll $t3, %pos, 2` 改变了本来想填的 `$t3` 的值，错误

```assembly
.macro setStack(%des, %pos)
	push($t3)
	sll $t3, %pos, 2
	sw %des, stack($t3)
	pop($t3)
.end_macro
```

### 字符和字符串

- 字符串读入—8

```assembly
 li $v0, 8
 la $a0, buffer      # 字符串存储位置
 li $a1, 100    # 最多读取 100 个字符
 syscall
```

`li $v0, 8` 与 `fgets()` 行为相同

- `li $a0, 1`，不会有内容写入到内存中
- `li $a0, 2`，将会一次读取字母，一次读取换行符
- `li $a0, 3`，那么将一次性读取一个字母加换行符。因为只关心字母，下一读取时写入的内存地址只需+1，覆盖掉换行符
- 字符读入—12

```assembly
 li $v0, 12
 syscall
 move $t0, $v0  # 将读取的字符保存到 $t0
```

### 保存寄存器的值

> 除 `$0,$gp,$sp` 外，对于认为不会发生改变的寄存器，如果我们在函数内中需要对其进行修改，则应在**函数开始**处进行一次入栈，**函数结尾**处进行一次出栈，遵循先入后出的规则。同时，`$ra` 一定是第一个入栈的，`$fp` 入栈前一定仅有 `$ra` 入栈。
> 

> 对于认为会发生改变的寄存器，如果在调用函数后仍然需要使用该寄存器内的值，则无论函数内部是否对其进行修改，都应该在**调用函数前**进行一次入栈，**调用函数后**进行一次出栈，遵循先入后出的规则。
> 

### 其他常见错误

- 定义了 `.data` 字段就必须有 `.text`

### MARS 查看运行指令数

Tools → Instruction Counter/Statistics → Connect to MIPS → 不用关闭窗口，直接运行即可

## 课上总结

### T1 mooncake

分月饼 (想起离散课 lzj 的 ppt 了) ， `n` 个人，月饼至少 `l` 个，至多 `r` 个，求带多少月饼实现每个人分得尽量多，且剩得尽量少，输出剩余月饼数。

思路：比较 `l/n` 和 `r/n` ，如果一样，剩余 `l%n` ；否则，剩得最少是 `0`

### T2 prime

判断 [2, 2000] 内的某个整数是不是质数。

给了 C 代码：先判断所有数是质数还是合数，再输出要求的那个。 `i`  为质数，则 `2i` `3i` `4i` …为合数

### T3 graph

有向图，找给定点能到达的出度为 0 的点的个数 (不含自身？)

挺有意思的题，也给了代码。上机的时候 `jal dfs` 完了后写了句 `jr $ra` ，自然是有问题。 `jal dfs` 相当于函数的调用， `jr $ra` 相当于 return 语句，注意什么时候写。

### 助教问答

怎么导出机器码； `$a0` `$t0` 是什么类型寄存器；

`nop` 是什么？没答上来。就是空语句

### Thoughts

这周写 C 代码的时候，自己是边写边思考 (以往是先打草稿) ，是 Paul 提到的那种感觉！

写第一题不在状态，没想好怎么写，就当跳跳虎，跳过了。感觉做不出来就大胆跳吧！先做能让自己静下来的题。看到前排的同学在自己研究 T3 的代码，沉着而冷静~