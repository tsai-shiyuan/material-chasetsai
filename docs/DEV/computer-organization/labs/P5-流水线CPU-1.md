# P5-流水线CPU-1

写得很好的博客:

- [P5 课下学习 — 流水线 CPU 设计 (1) - 北航计算机组成原理 | Test Blog = FlyingLandlord's Blog](https://flyinglandlord.github.io/2021/12/01/BUAA-CO-2021/P5/P5%E8%AF%BE%E4%B8%8B/)
- [Computer-Organization-BUAA-2020/7-P5/P5课下学习.md at main · rfhits/Computer-Organization-BUAA-2020](https://github.com/rfhits/Computer-Organization-BUAA-2020/blob/main/7-P5/P5%E8%AF%BE%E4%B8%8B%E5%AD%A6%E4%B9%A0.md)
- [「BUAA-CO」P5_流水线cpu（Basic） | Hyggge's Blog](https://hyggge.github.io/2021/11/15/co/co-p5-liu-shui-xian-cpu-basic/)

## Prepare

### 第一天

重定向: 

- 执行 (Execute) 阶段的**源**寄存器 = 存储 (Memory) 阶段 / 写回 (WriteBack) 阶段的**目的**寄存器
- 利用同一个周期内的不同部件的数据进行重定向
- 接收 Memory / WriteBack 阶段的 RegWrite → 如果为 0，说明不会写入 GRF，不需要重定向
- 此时 Execute 的源:
    - GRF
    - 来自 Memory / WriteBack
- `$0` 不需要重定向，始终保持 GRF 读出的 `0`
- Memory 的优先级高于 WriteBack (更新)

```verilog
if ((rsE != 0) && (rsE == WriteRegM) && RegWriteM)
	ForwardAE = 2'b10;
else if ((rsE != 0) && (rsE == WriteRegW) && RegWriteW) 
	ForwardAE = 2'b01;
else
	ForwardAE = 2'b00;
```

阻塞:

- `lw` 直到 DM 后才能读出数据，无法重定向
- 阻塞: 将操作挂起直到数据有效
- 检查 Execute 阶段的指令，如果是 `lw` && 目的寄存器 `rtE` 匹配译码阶段的源操作数 (rsD / rtD) ，则阻塞
- 译码 & 取指的寄存器增加使能输入 `En` ，执行阶段寄存器增加同步复位 / 清除 (CLR)，当 `lw` 阻塞出现:
    - `StallD` 和 `StallF` 有效，使译码和取指保持原值
    - `FlushE` 有效，清除 ALU 的内容

```verilog
lwstall = ((rsD == rtE) || (rtD == rtE)) && MemtoRegE;
StallF = StallD = FlushE = lwstall;
```

控制冲突:

- 比较器在译码阶段结束时确定下一个 PC

今日小结:

画了图，原理大致明白，明天想仿照 hyggge 把图的接线再画清楚些。（P5很难!）

### 第二天

更新了图，大体写出了各个模块

### 第三天

计组教程阅读

将会发生冒险的指令阻塞在 D 流水级上

看懂了教程……看懂了 roife 的代码，参考 flyinglandlord 的设计文档初步写成所有功能模块 (重写了)

### 第四天

加 nop:

```bash
 65@00003010: $ 8 <= 00640000
115@00003024: $ 9 <= 000a0000
165@00003038: $10 <= ffff0000
215@xxxxxxxx: $11 <= 00000000
```

不加 nop:

```bash
55@0000300c: $ 8 <= 00640000
65@00003010: $ 9 <= 000a0000
85@00003018: $10 <= ffff0000
95@xxxxxxxx: $11 <= 00000000
```

完成了顶层模块的接线！成功导入 roife 的指令显示模块！接下来细节检查+测试+debug+学长博客

提交的时候记得删掉 roife 的 DASM 部分

### 第五天

AK 弱测。还需要做的事:

- 把 WriteReg 沿流水线传，如果目的寄存不定，CTRL 无法译码。WriteReg 需要送入 FU 里。

### 第N天

T: D级的跳转指令在判定为跳时，清空延迟槽 (抛弃此时F级的指令)

A: (细节) 阻塞信号 StallD 生效时不能 Flush，要把 `if (reset || FlushD)`  的 `FlushD` 放到 `if (StallD)` 之后

一切 if-else 都要考虑优先级

当**数据和目的寄存器**都准备好的时候，我们才说状态确定，即在此刻才能计算 `Tnew`

当 StallD 生效(指令被堵在 F)，D 级的 beq 正在等后面计算出用于比较的数，此时 FlushD 为 1 是不稳定的，不能起作用

```nasm
lw $6, 4($0)
sw $6, 12($0)
```

我的架构里，遇到这样的组合会阻塞，而 roife 的不会，但我觉得我的挺好的，因为减少从 W 到 M 的转发。

想做的事:

- 完善 datamaker
- 自动化运行，搭个评测机？！

## Bug 记录

### W 阶段 GRF

GRF: 连成了 `pc_d` ，实际应连 `pc_w` (想想何时写)

### FU

```verilog
if (E_rs == M_des && E_rs != 0) begin
	E_FwdA = 2'd2;
end
else if (E_rt == M_des && E_rt != 0) begin
	E_FwdB = 2'd2;
end
else if (E_rs == W_des && E_rs != 0) begin
	E_FwdA = 2'd1;
end
else if (E_rt == W_des && E_rt != 0) begin
	E_FwdB = 2'd1;
end
else begin
	E_FwdA = 2'd0;
	E_FwdB = 2'd0;
end
```

A 和 B 的转发不能同时判断！

但是样例 9 为什么 E_rs 是 2

```verilog
if (E_rs == M_des && E_rs != 0) begin
	E_FwdA = 2'd2;
end
else if (E_rs == W_des && E_rs != 0) begin
	E_FwdA = 2'd1;
end
else begin
	E_FwdA = 2'd0;
end

if (E_rt == M_des && E_rt != 0) begin
	E_FwdB = 2'd2;
end
else if (E_rt == W_des && E_rt != 0) begin
	E_FwdB = 2'd1;
end
else begin
	E_FwdB = 2'd0;
end
```

分开是对的。

### 非常奇怪的仿真 clk 问题！

仿真时间太久了，重合到前面来了？

![屏幕截图 2024-11-07 121936.png](P5-%E6%B5%81%E6%B0%B4%E7%BA%BFCPU-1%201336c2fa7a33805a8c83e03450038dbe/%25E5%25B1%258F%25E5%25B9%2595%25E6%2588%25AA%25E5%259B%25BE_2024-11-07_121936.png)

模块实例化的最后一个端口没有 `,`

### store 类指令的 Tuse

原来:

```verilog
wire [2:0] Tuse_rt = (D_branch) ? 3'd0 :
					 (D_cal_r) ? 3'd1 :
					 3'd3;
```

因为 `sw` 在 E 级就要确定 GprRt

```verilog
wire [2:0] Tuse_rt = (D_branch) ? 3'd0 :
					 (D_cal_r | D_store) ? 3'd1 :
					 3'd3;
```

多亏了 Gavin 同学的测试数据。这么大的 bug 为什么之前没测出来呢 (￢_￢)

## 课上总结

### T1 xe

`xe rd, rs, rt`

temp[31:0] 的偶数位是 GPR[rs] 的偶数位与 GPR[rt] 异或，奇数位为 GPR[rs] 的对应位。

解法:

- ALU 增加功能
- CTRL 里 `xe` 归类为 `cal_r`

### T2 jtl

`jtl rs, rt, label`

RTL 语言:

```bash
I1:
	temp1 = PC + 4 + sign(offset || 0^2)
	temp2 = GPR[rs] + GPR[rt]
	condition = temp1 < temp2
	GPR[31] <- PC + 8
I2:
	if condition:
		PC <- temp1
	else:
		PC <- temp2
		NullilyCurrentInstruction()
	endif
```

像 `jr` ，解法:

- NPC: GPR[rt] 传入 NPC，在 NPC 内完成 PC 的计算，并给出 jump 信号，为 1 表示需要清空延迟槽
- F / D 级寄存器: 增加 FlushD 信号，为 1 时则刷新 (信号全给 0 )，注意 StallD 生效时刷新无效
- CTRL: 新增专属 `jtl` 的分类 `t_jtl`
- SU: 和 `jr` 相似，不过要增加对 `rt` 的判断，如果 `rt` 和后面的 `WriteReg` 等且非 0 且满足 Tuse < Tnew，需要阻塞

出错的地方:

- NPC 里 assign 转成 case 的时候写成
  
    ```verilog
    always @(*) begin
    	case (op)
    		`NPC_b && branch: begin
    			...
    		end
    	endcase
    end
    ```
    
    应该是:
    
    ```verilog
    always @(*) begin
    	case (op)
    		`NPC_b: begin
    			if (branch) ...
    			else ...
    		end
    	endcase
    end
    ```
    
- FU `t_jtl` 没写全 (新架构不存在此问题)

### T3 lwoc

`lwoc base, rt, offset`

F / D / E / M 级的行为均和 `lw` 相似，从 Memory[GPR[base] + offset] 中取出 text，将 text 与 0x80000000 比较，text 小则将 text 写入 text[3:0] 号寄存器，否则写入 rt 寄存器

在我的架构里，WriteReg 会随流水线传下去，D 级译码出 WriteReg = 0。解法:

- CTRL: 增加分类 `t_lwoc` ，增加信号 `Wolffy` 表示是 `lwoc` 指令
- M / W 级流水线寄存器: 把原来输出的 `write_reg_w` 换成 `temp_reg_w`
- mips W 级: 根据 `Wolffy` 信号+和 `mem_data_w` 比较结果确定写的目的寄存器 `write_reg_w`
- SU: 当 `lwoc` 指令出现在 E / M 级时，由于不知道写的目的，如果满足 Tuse < Tnew 都需要阻塞，不需要额外判断 rt / rs 是否与 `des` 相等 (也无法判断，因为我置 0 了)，如果直接这样阻塞，会 TLE，需要修改:
    - `lwoc` 写的目的只能是 rt 或 0 ~ 15 号，其他情况可以不用阻塞
      
        ```verilog
        wire stall_rs_e = (Tuse_rs < E_Tnew) && 
        									((E_lwoc && D_rs == E_Instr[20:16]) ||
        									 (E_lwoc && D_rs[4] == 0) ||
        									 (D_rs == E_des && D_rs != 0));
        ```
        
    - 这样仍然会 TLE 一个点，当 `D_rs` 为 0 时，也不用阻塞，最终的阻塞逻辑:
      
        ```verilog
        wire stall_rs_e = (Tuse_rs < E_Tnew) && 
        									((E_lwoc && D_rs == E_Instr[20:16] && D_rs != 0) ||
        									 (E_lwoc && D_rs[4] == 0 && D_rs != 0) ||
        									 (D_rs == E_des && D_rs != 0));
        ```
        
    - AC!!!!! ヽ(￣ω￣(￣ω￣〃)ゝ

### 助教问答

什么是延迟槽？课下测试思路？课上测试代码？怎么在课上 debug 的？

考试的时候依然是瞪眼法直接 de 的，de 出来一个就交上去……感觉真得好好准备课上测试程序，学学除了面向评测机，怎么用实例来 debug

---

很开心的 P5，考前比较焦虑我的 `sw` 和 `lui` 没有全力转发会不会超时，第一题提交直接 AC 给了我很大的信心，证明课下没有错，测试很充分，也不会出现这两个指令的超时。第二题错了几次，好在检查每个修改过的地方后查出了 bug，此时才过一个小时不到。斗志昂扬地步入第三题，我一开始就分了类，写着写着觉得和 `lw` 太像了，纠结要不要并入 `lw` ，最后还是按新分类写下去，最后特判阻塞逻辑的时候才发现这样是对的，相信自己的感觉~一直 TLE，最后几分钟改了 0 特判，通过！

考试前十分钟网站崩了，趁此机会又画了流水线草图。

条件访存比较难，特别是它的阻塞逻辑

课下第一次自己编了测试代码生成程序，Nothing is impossible!

相信自己，继续设计我的 CPU ~