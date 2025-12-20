# P3-单周期CPU(Logisim)

> ps. 设计文档里有一些实现细节
> 

## 课下总结

### immediate 不合理时的处理

ori

```assembly
ori $a0, $0, 1
ori $a1, $a0, -3
```

- `ori` 是无符号扩展，如果出现负数，Mars 会转成很多指令，最终得到 `$a1 = -3`

### 评测原理

> 非常重要，当觉得没有逻辑错误的时候往往是忽略了评测细节
>
> 转载讨论区帖子: http://cscore.buaa.edu.cn/#/discussion_area/1416/1660/posts

比对: 将输出文件进行逐行比较，当某一行的文本不相同时，返回WA和错误提示

输出文件的生成:

- 当写寄存器信号 RegWrite=1 时，会在输出文件中写入一行字符串
    - 格式: `[Instr] @[pc] : $[RegAddr] <= [RegData]`
- 当写内存信号 MemWrite=1 时，会在输出文件中写入一行字符串
    - 格式: `[Instr] @[pc] : *[MemAddr] <= [MemData]`

所以，需要确保我设置的端口和题目的端口的实际含义相同！

### 测试 by others

https://foreveryolo.top/posts/6588/index.html

https://triplecamera.github.io/co-discussions/1070

## 课上总结

### T1 cwp

`cwp rs rt imm` 

imm 后八位补 0，与 `PC` 比较。若大于 `PC` ，则将原 imm 存入 `$rt` ，否则存入 `$rs` 

在原电路上修改:

ALU: imm 无符号扩展送入 ALU 的 `SrcB` ，PC 给 `SrcA` ，内部完成计算并给出 `ItoR`  信号 (1→ `$rt` 0→ `$rs` )

CTRL: 增加 PCCmp 信号，表示用 PC 比

GRF 的 WA: PCCmp 和 ItoR 信号共同决定

### T2 bgc

`bgc rs rt offset`

GPR[rs] 每两位转为 Gray Code (即 00→00, 01→ 01, 10→11, 11→10)，计算其中 1 的数目，与 `rt` 的值比较 (注意不是 GRF[rt])，1 的数目大于 `rt`  则跳转

思路: 在 ALU 内算，结果输送到 IFU 中，当是这个指令且满足条件则跳

### T3 lwso

`lwso base rt offset`

对 GPR[rt] 进行移位操作 (规则比较长但是很好实现) 得到 5 位的 reg_left 和 reg_right，取到的 Mem[addr] 是奇数，则将 Mem[addr] 给 reg_left，否则给 reg_right

### 助教问答

检查测试程序，考试的时候没写 (我觉得过了就没必要写吧，写了也不会存)。了解了导入 class 的操作 (需要课程组的 MARS 才行) :

![屏幕截图 2024-10-28 211705.png](P3-%E5%8D%95%E5%91%A8%E6%9C%9FCPU(Logisim)%201286c2fa7a33801fa96cf49a2ff57cb5/%25E5%25B1%258F%25E5%25B9%2595%25E6%2588%25AA%25E5%259B%25BE_2024-10-28_211705.png)