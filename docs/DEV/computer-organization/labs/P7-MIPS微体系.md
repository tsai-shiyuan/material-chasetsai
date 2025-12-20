# P7-MIPS微体系

这些博客有助于理解:

- [P7 课下 & 课上总结 - 北航计算机组成原理 | Test Blog = FlyingLandlord's Blog](https://flyinglandlord.github.io/2021/12/15/BUAA-CO-2021/P7/P7%E8%AF%BE%E4%B8%8A&%E8%AF%BE%E4%B8%8B/)
- [[BUAA-CO-Lab] P7 MIPS 微体系](https://roife.github.io/posts/buaa-co-lab-p7/)
- [计算机组成-支持中断的CPU](https://thysrael.github.io/posts/4c542040/) Thysrael还讲了测试方案
- [CO-P7 支持异常的流水线 CPU](https://chlience.com/BUAA-CO-P7/#%E4%BB%80%E4%B9%88%E6%98%AF%E5%BC%82%E5%B8%B8)

## 课下

### Day 1

aaa不知道要干什么…

### Day 2

要做的事:

- 更改流水线各级使之可以产生异常
- 添加 CP0 处理异常
- 添加 Bridge 与外设交互

异常 Exception & 中断 Interrupt 发生后的行为:

- 跳到固定位置
- 要把当前指令地址写到 EPC (延迟槽写当前地址-4)

检测异常:

- 各级给出本级的异常信号
- 流水到 M 级的 CP0
- CP0 决定是否处理这个异常，是的话就跳转

新增指令:

- `eret`
    - 跳回 EPC 中的地址
    - 其后的指令不能执行
    - 在 D 级判断，置位 pcF 为 EPC，npc 为 EPC + 4 (当D级是 `eret` 时，马上就会把F级读出的指令改变为EPC位置的指令，下一个周期用npc直接读出下一条)

清除异常后的指令: 指令流经流水线时，可能留下影响的操作只有修改 `PC` ，写入流水线寄存器，写入乘除模块，写入 `DM` 和写入 `GRF`

- 由于后续需要跳转到异常处理程序， `PC` 无需处理
- 流水线寄存器储存的是指令中间信息，直接清空

最后异常处理程序将会使用命令 `eret` 跳转到 `EPC` 对应地址，即从异常处理程序返回

由于阻塞的存在，受害指令可能有两种情况：

1. 一条平平无奇的普通指令
2. 由于阻塞产生的空泡指令

### Day 3

发生异常的指令在一条转移指令的延迟槽内，EPC 要指向转移指令

### Week 2 Tue

评测机初尝试🥳

### Week 2 Wed

我的评测机为我检测出了我的一个bug，先前的 HILO 逻辑:

```verilog
if (reset) begin
	// ...
end 
else if (cnt == 0) begin
	// ...
end 
else if (cnt > 1) begin
	// ...
end
else begin
	Busy <= 0;
	cnt <= cnt - 1;
	HI <= tHI;
	LO <= tLO;
end
```

最后的 `else` 必然是 `cnt==1` 的情况，此时进行 cnt 的自减，HI / LO 的写入是合理的。然而 P7 当 `Req` 有效时，我们说不能开始 md / mt 指令:

```verilog
if (reset) begin
	// ...
end 
else if (cnt == 0 && !Req) begin
	// calculate tHI, tLO
	// modify cnt
end 
else if (cnt > 1) begin
	cnt <= cnt - 1;
end
else begin
	Busy <= 0;
	cnt <= cnt - 1;
	HI <= tHI;
	LO <= tLO;
end
```

当 `cnt==0` 且M级 `Req==1` 时，本来应该不进行任何操作 (既不计算临时值，也不修改 cnt)，但是上述代码会进入 `else` 并执行自减 `cnt` ，这样 `cnt` 将变为 `5'b11111` ，以后的乘除指令将无法正常启动！甚至非乘除指令也会触发！

修改为:

```verilog
else if (cnt == 1) begin
	Busy <= 0;
	// ...
end
```

搭好了评测机，好耶🥳🥰

### Week 2 Thu

走查代码，一些疑惑:

PC: EXCAdEL why except EretD

中断:

- 全局中断使能位 SR(IE) 必须置 1，否则不会服务任何中断
- SR(EXL) 如果为 1 则禁止中断 (因为正在处理异常/中断)
- SR(IM) 为 1 时该中断才能发生

当M级的CP0决定响应异常/中断时，此时Req为1，下一个上跳沿:

- PC:
    - D / E / M / W 级均为 0x4180 (清除4条指令)
    - F 级为 0x4180 (进入处理程序)
- Instr:
    - D / E / M / W 级均为 0
    - F 级为从 0x4180 取出的

当D级正为 `eret` :

- F 级 PC 应该立刻变成 EPC (组合逻辑才能反应)
- D 级的 npc 设为 EPC+4

检测出错即可清除这条指令

我感觉我梦到过讨论区说 `assign d_instr = d_err_ri ? 32'b0 : d_tmp_instr;` 清零与否

CPU地址空间被划分为若干区域，每个区域对应一个存储器 / 设备。CPU 不能为每一个设备提供一套地址。增加系统桥。CPU侧一组接口，设备侧N组接口。由Bridge完成地址、数据转换，控制信号产生

中断发生器: 会随机的产生外部中断信号，产生的中断信号在 CPU 响应前会持续置高

中断发生器都是在测试的 tb 上实现的:

- 通过外部端口接受外部中断信号 (计时器中实现)
- 通过访问地址 `0x7F20` 的 `store` 类指令，改变对应的微系统输出信号 ( `m_int_addr`，`m_int_byteen` )

`m_int_addr[31:0]` : 中断发生器待写入地址。当该信号命中中断发生器响应地址，且字节使能信号有效时，视为响应外部中断

`m_int_byteen` : 中断发生器字节使能信号，当该信号任意一位置位时视为有效

## 问题

```verilog
assign EXCAdEL = !EretD && (pc < `PC_START || pc > `PC_END || pc[1:0] != 0);	
// why !EretD ????
```

`eret_m` 的时候发生了中断？先处理谁？

Fly 和 zlr 在 CP0 写了 `testResponse` ，有什么作用？

## Bug 记录

### 重大Bug: instr

有溢出的 add 指令即将进入 E 级时卡住，信号不断变化。

```nasm
lui $t0, 0x7fff
lui $t1, 0x7fff
add $t2, $t0, $t1
```

原因: add 即将进入 E 级时， `instr_e` 是 `temp_instr_e` ，正常转发，之后检测出溢出，此时 `instr_e` 置0，传入 FU，不再进行转发， `rt` 只读到了0，瞬间算出 `exc_code_e` 没有异常，由会把 `instr_e` 变为 `temp_instr_e` ，陷入循环

在 F / D / E / M 级我都用到了 `temp_instr` ，再由此进行组合逻辑运算，但是 SU 和 FU 用的是 `instr` ，应该把 `temp_instr` 传入 SU / FU 判断。

debug: 拖了很多信号进波形图，发现 `new_rd2_e` 反复跳动，由此发现症结所在

### DE 的 AdEL 判断错误

load 类指令是可以读 Count 寄存器的值的，直接复制 BE 模块就出错了

### 指令是否处在延迟槽

```verilog
assign delay_slot_f = (npc_op_d != `NPC_pc4);
```

`eret` 之后的指令的BD会被置1，如果有两个连续异常的指令，处理完第一个指令的异常后，由 `eret` 返回，第二个指令被当成延迟槽，会一直循环执行……修改为:

```verilog
assign delay_slot_f = (npc_op_d != `NPC_pc4 && npc_op_d != `NPC_eret);
```

测出错误的代码:

```nasm
lui $5, 0xfff8
lw $4, 0x7f04($5)
lb $5, 0x7f04($0)
```

很意外地发现了错误，一念之间，很幸运地编两个连续溢出样例！

何时不是延迟槽？普通的 pc4， `eret` 之后，其余都是了。

## 课上测试

四道强测+一条新指令

`withdraw`

题意：在CP0中新增4个寄存器: `LastAddr` , `LastData` , `IMBoundaryHi` , `IMBoundaryLo` 存放 `sw` 指令修改的那片内存之前的信息。 `withdraw` 指令到来时，就根据 Last 中的信息，“撤销” `sw` 的影响，恢复内存。

细节：

- reset 为1时，四个寄存器置0
- `IMBoundaryHi` , `IMBoundaryLo` 支持 `mfc0` `mtc0`
- `LastAddr` , `LastData` 支持 `mfc0` , 会忽略 `mtc0` 指令对其的写入
- 自reset以来未出现 `sw` 就进行 `withdraw` ，忽略该指令
- 如果 `LastAddr` 不在 Boundary 范围内，发生 `AdES` 异常

在我的架构内，F / D / E 级就会完成所有指令的异常判定，所以能流水到 M 级的 `sw` 就一定是正确的。在 M 级只需要由 CP0 判断是否有新增的指令的异常

当 `sw` 指令到达 M 级时，BE 模块会写内存，同时，我们要通知 CP0，让它保存 `sw` 地址的原信息，存入 Last 寄存器。原地址和 `sw` 要写的地址一样，原数据需要从内存中读出来——需要更改 DE。DE 遇到 `sw` 指令时，要从内存读出字，并交给 CP0。CP0还要增设 `start` 寄存器，置1表示出现了 `sw` 指令。

此时不会发生任何异常。

当 `withdraw` 指令到达 M 级时，CP0要判断是否重写内存，给出 `OK` 信号，判断是否出现异常 `AdES` :

- 先前没有 `sw` , 不重写，无异常
- `LastAddr` 超范围，发生异常，此时 Req 会变为 1，BE 不会重写
- 除此之外，重写

BE 接收 OK 信号，如果是 `withdraw` 指令并且 OK，则写内存，否则不写。

BE 写的数据要在 `LastData` 和 GPR[rt] 中选。

And after that…AC…Congratulations!

最后一次的助教提答，问了宏观PC，测试思路，eret的行为，得到了巧克力🍫，Awesome CO! (◡ᴗ◡✿)