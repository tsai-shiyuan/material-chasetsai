# P6-流水线CPU-2

很好的博客 d=====(￣▽￣*)b

- [P6 课下 & 课上总结 - 北航计算机组成原理 | Test Blog = FlyingLandlord's Blog](https://flyinglandlord.github.io/2021/12/08/BUAA-CO-2021/P6/P6%E8%AF%BE%E4%B8%8A&%E8%AF%BE%E4%B8%8B/)
- [「BUAA-CO」P6_流水线cpu（Plus） | Hyggge's Blog](https://hyggge.github.io/2021/11/26/co/co-p6-liu-shui-xian-cpu-plus/)
- [Computer-Organization-BUAA-2020/8-P6/P6课下解析.md at main · rfhits/Computer-Organization-BUAA-2020](https://github.com/rfhits/Computer-Organization-BUAA-2020/blob/main/8-P6/P6%E8%AF%BE%E4%B8%8B%E8%A7%A3%E6%9E%90.md)
- [[BUAA-CO-Lab] P6 流水线 CPU-lite2](https://roife.github.io/posts/buaa-co-lab-p6/)
- [P5 && P6-流水线CPU（verilog实现）（已更新）- BUAA-Wander - 博客园](https://www.cnblogs.com/BUAA-Wander/p/11990363.html)
- [【BUAA_CO_LAB】p5&p6碎碎念_buaa p5-CSDN博客](https://blog.csdn.net/cedr1c_wyc/article/details/121934083)

## 日志

### Day 1

很优雅的半字节代码:

```verilog
`define word                m_data_rdata[31:0]
`define half_word           m_data_rdata[(16*Addr[1] + 15) -: 16]
`define byte                m_data_rdata[(8*Addr[1:0] + 7) -: 8]
`define half_word_sign      m_data_rdata[16*Addr[1] + 15]
`define byte_sign           m_data_rdata[8*Addr[1:0] + 7]

module M_DE(
    input [31:0] Addr,
    input [2:0] DEOp,
    input [31:0] m_data_rdata,
    output reg [31:0] DMRD
    );
    always @(*) begin
        case(DEOp) 
            `DE_lw: DMRD = `word;
            `DE_lh: DMRD = {{16{`half_word_sign}}, `half_word};
            `DE_lhu: DMRD = {{16{1'b0}}, `half_word};
            `DE_lb: DMRD = {{24{`byte_sign}}, `byte};
            `DE_lbu: DMRD = {{24{1'b0}}, `byte};
        endcase
    end
endmodule
```

把官方 testbench 调换了顺序:

```verilog
always @(posedge clk) begin
    if (~reset) begin
        if (w_grf_we && (w_grf_addr != 0)) begin
            $display("%d@%h: $%d <= %h", $time, w_inst_addr, w_grf_addr, w_grf_wdata);
        end
    end
end
	
always @(posedge clk) begin
    if (reset) for (i = 0; i < 4096; i = i + 1) data[i] <= 0;
    else if (|m_data_byteen) begin
        data[fixed_addr >> 2] <= fixed_wdata;
        $display("%d@%h: *%h <= %h", $time, m_inst_addr, fixed_addr, fixed_wdata);
    end
end
```

以便按照指令执行顺序输出，还修改了 Flying 的

### Day 2

周二晚上一直在想乘除槽和乘除指令的阻塞问题，周三上午清醒的时候，正式动工接线

晚上正式 AC

找了一些自动化评测的工具:

- 学长的 Open Mars Here 小工具

### Day 3

To Do:

- [x]  走查整个 project，检查逻辑问题，完成设计文档
- [x]  学自动化运行

真的看出了一个 bug，先前把 `D_rs != 0` 漏掉了

```verilog
if (D_rs == E_des && E_mf && D_rs != 0) D_FwdA = 2'd2;
```

根据 warning 又找到一个潜在的 bug，线的位宽不一致

感觉 P6 好难构造数据，乘除槽很容易就除 0 了

### Day N

`madd` 将两个数有符号相乘，计算结果与之前的 HI, LO 寄存器中的值相加:

```verilog
// 错误写法1
{HI_temp, LO_temp} <= {HI, LO} + $signed(A) * $signed(B);
// 错误写法2
{HI_temp, LO_temp} <= {HI, LO} + $signed($signed(A) * $signed(B));
```

**错误1:** 位拼接 `{HI, LO}` 默认被当做无符号数，**无符号性**传递到 `$signed(A) * $signed(B)`，因此即使使用了 `$signed()` 还是会被当成无符号数进行乘法运算

**错误2:** 虽然使用了 `$signed()` 屏蔽了外界符号性的传入，但是也屏蔽了**位宽**信息的传入，所以 `$signed($signed(A) * $signed(B))` 的结果实际上是32位(因为 `$signed(A)` 和 `$signed(B)` 都是32位，又没有外界位宽信息的传入，因此结果被强制规定为32位)，即**高32位的数据被截去** 

所以需要把 64 位的信息传进乘法里:

```verilog
// 正确写法1
{tHI, tLO} = {HI, LO} + $signed($signed(64'b0) + $signed(A) * $signed(B));
// 正确写法2
{tHI, tLO} = {HI, LO} + $signed($signed({{32{A[31}}, A}) * $signed({{32{B[31]}}, B}))
// 正确写法3
{tHI, tLO} = $signed({HI, LO}) + $signed(A) * $signed(B)
```

这周的后面真的是过得很 awful 了，无所事事的，测试代码也感觉很难编，就到处翻翻找找的。还迷恋上了电子榨菜，熬夜刷手机……希望能改掉这个坏毛病，feeling ugly can make us more productive

## 遇到的 bug

### busy 信号的设置

```nasm
mult $3, $4
mflo $5
# end
```

现 `mult` 位于 D 级， `mflo` 位于 F 级

下一个时钟上跳沿来临时:  `mult` 进入 E 级，HILO 感受到的是**前一个瞬间**——E 级并非 `mult` ，所以 `busy` 保持 0，并未阻塞 D 级！

再下一个时钟上跳沿: HILO 感知刚刚是乘除法，根据条件 `busy` 置 1 ，后无指令了，出现高阻。总之达不到阻止 `mflo` 的效果

![屏幕截图 2024-11-13 171406.png](P6-%E6%B5%81%E6%B0%B4%E7%BA%BFCPU-2%2013c6c2fa7a338050b896ca09104624e3/%25E5%25B1%258F%25E5%25B9%2595%25E6%2588%25AA%25E5%259B%25BE_2024-11-13_171406.png)

!!!

`always @(posedge clk)` 用的是**前一个瞬间**的组合逻辑的值，兜兜转转一个学期，终于也理解了

## 课上总结

### T1 fmultu

`fmultu rs rt`

无符号乘法，计算占用3个周期

RTL:

```
prob = rs * rt
if prob[63:32] == 0
	hi = prob[63:32]
	lo = prob[31:0]
else 
	pos1 = highest_one_pos(prob)
	pos0 = lowest_zero_pos(prob)
	pos = |pos1 - pos0|
	temp = {prob[63:pos+1] || ~prob[pos] || prob[pos-1:0]}
	hi = temp[63:32]
	lo = temp[31:0]
```

`highest_one_pos(prob)` 函数返回 prob中 1 的最高位置的下标

思路: 改 CTRL 和 HILO 模块，全用了阻塞赋值

更好的方法:

```verilog
always @(*) begin
	// calculate temp final p_hi & p_lo
end

always @(posedge clk) begin
	case (MDOp)
		`MD_fmultu: begin
			busy <= 1;
			cnt <= 3;
			tHI <= p_hi;
			tLO <= p_lo;
		end
	endcase
end
```

bug:

- CTRL 里忘记加 MDOp = `MD_fmultu
- i < 63 ? I < 64!

### T2 bnumeq

`bnumeq rs, rt, offset`

GPR[rs] 和 GPR[rt] 的 1 的个数相同时，跳转 (和 b 类指令的跳转一致)

思路: 归类到 b 类，修改 CMP 就好

### T3 lwcm

名字可能记错了

`lwcm rt, offset(base)`

M 级增加寄存器 WD。按 lw 的方式从 memory 中读出 `mem_read_m` ，如果和 WD 内的值等，则写入 rt，否则 26 号。并且把这个新读到的值存入 WD。

思路: 加模块，改 SU / CTRL / M_BE…
