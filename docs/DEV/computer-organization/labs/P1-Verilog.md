# P1-Verilog

## 课下知识

### 位扩展

```verilog
{{16{imm[15]}}, imm} // 将 [15:0] 的 imm 扩展到 32 位
```

- 注意括号

### Gray 码

相邻数字只有一位不同

### 字符

`"c"`  表示字符

当把字符串赋值给 reg 时，reg 会存 ASCII，高位会补 0

`bfr = ""`  得到空串 `0000..00` 

### 同步复位 & 异步复位

异步

```verilog
always @(posedge clk, posedge clear) begin end
```

- `clear`  一定要写 `posedge/negedge`

GPT 帮忙 debug，AI 真的很强大

同步

```verilog
always @(posege clk) begin
		if (clear) // ...
end
```

### 端口类型

output 端口是

- reg 型，在 always 块内赋值
- wire 型，在 assign 中由组合逻辑判断

### Testbench

在 `initial`  块中补充：

- 时间间隔 `#`
- 信号的变化

例如: `#20; in = "e"`

时间的设置：

- 一般设为 `#10`  `#20`  (波形图默认走到 1000 ns)

注意tb中wire信号不能赋值

```verilog
// Initialize Inputs
clk = 0;
reset = 1;
coin = 0;

// Wait 100 ns for global reset to finish
#10;
reset = 0;

// Add stimulus here
coin = 1;
```

### disable

Verilog 中没有 `break` ! System Verilog 中才有！

用 `disable` 来禁用语句块 (带上标签)

```verilog
begin: continue
	for (i = 0; i < 4; i = i + 1) begin
		a = a + 1;
		if (i == 2) disable continue;    // 跳出整个循环，即break
		b = b + 1;
	end
end

for (i = 0; i < 4; i = i + 1) begin: continue
	a = a + 1;
	if (i == 2) disable continue;    // 中止一次循环，进入下一次，即continue
	b = b + 1;
end
```

## 课下题目

### BlockChecker

![CO_P1_BlockChecker.drawio.png](P1-Verilog%201136c2fa7a33805d9d92cb3477e48434/CO_P1_BlockChecker.drawio.png)

输出：

- 当 endSum > begSum 时，fatal 置 1，一定不合法
- 其余通过比较 endSum 和 begSum 来确定输出

Sum 的更新：

- 处于状态 4 或 7 并读到了 n/d，就要更新 Sum
- 处于状态 5 或 8
    - 如果读到 alpha ，说明不是 `begin` 或 `end` ，Sum - 1
    - 如果读到空，注意更新 fatal：endSum > begSum → fatal =1; (else) fatal = fatal (注意不能是 fatal = 0)

(附)波形图：

![屏幕截图 2024-10-03 090118.png](P1-Verilog%201136c2fa7a33805d9d92cb3477e48434/%25E5%25B1%258F%25E5%25B9%2595%25E6%2588%25AA%25E5%259B%25BE_2024-10-03_090118.png)

Testbench 的优雅写法:

```verilog
always #10 clk <= ~clk;
reg [0:1023] S = "a begin EndC end end Begin";

initial begin
		// Initialize Inputs
	clk = 0;
	reset = 0;
	
	while(!S[0:7]) S = S << 8;    // 很关键
	for( ; S[0:7]; S = S << 8) begin
		in = S[0:7];
		#20;
	end
end
```

`a begin EndC end end Begin` 存在 `S` 的 [x:1023] 位

优雅的解法:

```verilog
module BlockChecker(
    input clk,
    input reset,
    input [7:0] in,
    output result
    );

    reg [31:0] cnt;
    reg valid;
    reg [255:0] bfr; // 缓冲区

    assign result = ((cnt == 0) && valid);

    initial begin
        cnt <= 0;
        valid <= 1'b1;
        bfr = "";     // 相当于256位0
    end

    always @(posedge clk, posedge reset) begin
        if (reset) begin
            cnt <= 0;
            valid <= 1'b1;
            bfr = ""; // 每次复位一定要清除缓冲区
        end 
        else begin
            bfr = (bfr << 8) | in | 8'h20;      // 8'h20是32，全部转换为小写，妙啊！
            if (valid) begin
                if (bfr[47:0] == " begin") cnt <= cnt + 1;    // 遇begin加1
                else if (bfr[55:8] == " begin" && bfr[7:0] != " ") cnt <= cnt - 1;
                else if (bfr[31:0] == " end") cnt <= cnt - 1;
                else if (bfr[39:8] == " end" && bfr[7:0] != " ") cnt <= cnt + 1;
                else if (bfr[39:8] == " end" && bfr[7:0] == " " && cnt[31]) valid <= 1'b0;      // cnt[31]是符号位，等于1是负数
                // 其余情况 valid 不变，合理
            end
        end
    end
endmodule
```

### 收获

学会了设计 Testbench 仿真查看波形图。以前不懂的知识，在经历一段时间后慢慢地就熟悉了。前四题不会用 Testbench，平均需要四次才能过，附加题学会调试后，先在本地验证再提交很容易就过了。

## 课上总结

### T1 different

32 位宽的 a, b，拿 a 的第 0 位与 b 的第 31 位比较，a 的第 1 位与 b 的第 32 位比较……算一共有多少位不同。用 for 循环解答。

```verilog
integer i;
always @(*) begin
	for (i = 0; i <= 31; i = i + 1) begin
		if (a[i] != b[31-i]) cnt = cnt + 1;
	end
end
```

### T2 alternate

串行输入 01 序列，如果 0 和 1 的个数满足奇偶交替，check 置 1，其他情况 check 置 0。比如 0001111011…，0 和 1 的个数为 3 4 1 2 … 此时会输出 1。如果序列中只有 0 或 1，不能称为交替，输出 0。

| 端口名称 | 端口类型 | 位宽 |
| --- | --- | --- |
| number | I | 1 |
| rst_n (异步复位信号) | I | 1 |
| check | O | 1 |

| 状态编码 | 含义 |
| --- | --- |
| 0 | 初态 |
| 1 | 0 (奇数个 0，从初态来) |
| 2 | 00 (偶数个 0) |
| 3 | 1 (奇数个 1) |
| 4 | 11 (偶数个 1) |
| 5 | 0 |
| 6 | 00 |
| 7 | 1 |
| 8 | 11 |
| 9 | 非法态 |
- 在 5/6 状态下读 1 时，如果此时的 check 是 0，今后也无法再补救，进入非法态
- 在 5/6 状态下读 0 时，check 取反

这道题卡了挺久的，吸取教训：

- 当确定不合法，就丢入不合法状态，不要设置 `fatal` (此题这样写太麻烦)
- 复位的时候注意考虑所有 `reg` ( 考试的时候 check 忘记置 0 了)

讨论区:

- 初态来的 01 记录可以省略的，因为此时 check 刚刚置 0 (nonono，如果 01，会被识别为锁定非法，还是要有初态)

### T3 键值对检验

输入是形如 `xxxx{xxxx:xxxx,}xxxx{}` 的键值对，有一系列要求：

- `{key1: value1, key2: value2, }`
- `xxxx` 不会是 `{` `:` `,` `}` (也就是说，我们可以根据 `{` `:` `,` `}` 来分隔状态)
- `key`  只含大小写&数字，且不以数字开头； `value`  只含数字。 `key` `value` 内若有空格均视作不合法，除此之外可以任意添加空格
- 若 `{}` 内至少有一对合法键值对，则最后一对后有 `,`

 `each` 记录当前 `{}` 内有效键值对的数量， `total` 记录自复位以来最大的 `each`

![屏幕截图 2024-10-14 221411.png](P1-Verilog%201136c2fa7a33805d9d92cb3477e48434/%25E5%25B1%258F%25E5%25B9%2595%25E6%2588%25AA%25E5%259B%25BE_2024-10-14_221411.png)

- 当读到 `}`  就要设置 `each` 为 0，根据当前的 `each`  更新 `total`
- 在 7/8 状态下读到 `,` ， `each = each + 1`
- 每个状态需要考虑输入为 digit  alnum  `,`  `:`   ``  `{` `}`
- 不合法状态下读到 `,` `}` 会回到合法态

### 助教问答

阻塞 & 非阻塞；问了第2题的思路，嘴太笨了，讲不出来，助教说他去年跟我一样直接根据逻辑写的，也没画状态图；Testbench 怎么写
