# P4-单周期CPU(Verilog)

先放参考文献hh:)

- [fysszlr](https://github.com/fysszlr/CO-buaa-2023/blob/main/README.md)

- [P4-单周期CPU（Verilog实现） - BUAA-Wander - 博客园](https://www.cnblogs.com/BUAA-Wander/p/11873946.html)
- [Computer-Organization-BUAA-2020/6-P4/P4课下解析.md at main · rfhits/Computer-Organization-BUAA-2020](https://github.com/rfhits/Computer-Organization-BUAA-2020/blob/main/6-P4/P4%E8%AF%BE%E4%B8%8B%E8%A7%A3%E6%9E%90.md)
- [BUAA-CO-2023-Aut/P4_v2 at main · Squirrel7ang/BUAA-CO-2023-Aut](https://github.com/Squirrel7ang/BUAA-CO-2023-Aut/tree/main/P4_v2)
- [「BUAA-CO」P4_单周期cpu（Verilog实现） | Hyggge's Blog](https://hyggge.github.io/2021/11/09/co/co-p4-dan-zhou-qi-cpu-verilog-shi-xian/#%E8%AE%BF%E5%AD%98%E6%8C%87%E4%BB%A4%E6%B5%8B%E8%AF%95)
- [P4 课下学习 — 单周期 CPU 设计 (2) - 北航计算机组成原理 | Test Blog = FlyingLandlord's Blog](https://flyinglandlord.github.io/2021/11/13/BUAA-CO-2021/P4/CPU%E8%AE%BE%E8%AE%A1%E6%96%87%E6%A1%A3/)
- [BUAA-CO/p4 at master · roife/BUAA-CO](https://github.com/roife/BUAA-CO/tree/master/p4)

## 课下学习

### 遇到的 bug

- 用宏一定要 `
- `ori` / `sw` 只执行了几行 (删了 bne 指令就对了，看来是无意间跳转了)
- 字对齐的输出: `$display` 要改成 `{MemAddr[31:2], 2'b00}`
- 数字在不声明位宽时默认 32 bit，不声明进制时默认**十进制**

### 设计细节

使用宏的好处: 我想修改某个控制信号的位宽时，只需要修改宏和 `input`  中的位宽。

## 课上总结

### T1 eam

`eam rd rs rt`

取 GPR[rs] 的低 16 位，作为有符号数对 17 取模，得到 16 位的 `temp`。若 GPR[rt] 的最高位为 1，则将 `temp` 零扩展，否则将 `temp` 1 扩展，结果写入 GPR[rd]。注意取模取最小的正数 (比如 `16'hffff % 17 = 16'h0010` )

思路: 在 ALU 模块内运算， `$signed(num) % 17` 求出来是距离 0 最近的数 (有负数) ，所以如果是负数，再加 17

### T2 cptl

`cptl rs rt offset`

如果 GPR[rs] 的低 18 位有连续 6 个 1，则将 PC+4 存入 GPR[rt]，否则将 PC+4 存入 31 号寄存器。并且无条件跳转至 PC+4+signExt(offset << 2  || 00)

思路: 和 b 类指令的跳转方式一样，所以归类为 `NPC_b` ，并且 `CMP_cptl` 时 CMP 模块始终给出 `branch` 为 1 的信号。然后处理寄存器写的部分，我是在 ALU 内计算是否有连续 6 个 1，结果给 ALUOut，顶层 mips 模块根据 ALUOut 再确定目的寄存器

### T3 olw

`olw rs rt offset`

有条件的 `lw` ，当取出的数的二进制从高位到低位是非递减的 (若某位出现了 1，则更低位只能 1) 时，才将这个数写入 GPR[rs]

思路: 先取数，再在 GRF 内判断是不是能写入

---

考前看了一个学长的回忆，注意到了 for 循环每次开始要清零的问题。考试一般是算数+跳转+访存。纠结的一类题: GRF 的写目的寄存器不确定，这种题要考虑在某个模块给出选择输出……

---

这次上机感觉挺不顺的，首先上机前在图书馆自习，但是，座位插座没电🪫，就只有在 Mac 上用 CotEditor 写代码。其次，考到最后下载设计文档的时候，我才发现交错了，交成代码了，助教问答环节感觉是挂了。提醒自己: **压缩文件认真命名。**考试开始的时候太慌张了，就没下设计文档，这么看来还是幸运的了，不然肯定更慌……再者，只能压缩为 `zip` 文件提交，考试的时候看压缩到 `rar` 就直接点了，WA 了一次……
