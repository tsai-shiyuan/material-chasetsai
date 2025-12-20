# Markdown 语法

## 分级标题

```
# 一级标题 (注意有空格)
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题  <!--最多6级标题-->
```
## 目录
在任意位置插入 `[toc]` 显示全文目录结构
## 斜体/粗体/删除线/下划线/背景高亮
```
*斜体*    _斜体_
**粗体**    __粗体__
***加粗斜体***    ___加粗斜体___
~~删除线~~
<u>下划线</u>
==背景高亮==
```
## 无序列表/有序列表
### 无序列表
```
* 无序列表项 一
+ 无序列表项 二
- 无序列表项 三
```
* 无序列表项 一
+ 无序列表项 二
- 无序列表项 三
### 多级无序列表
```
* 今天`* + 空格键`
* 明天
    * 学习 `TAB(或4个空格) + * + 空格键`
    * 购物
        * 面包
        * 牛奶
* 后天
```
* 今天`* + 空格键`
* 明天
  * 学习 `TAB(或4个空格) + * + 空格键`
  * 购物
    * 面包
    * 牛奶
* 后天
### 有序列表/多级有序列表
```
1. 有序列表项 一 `数字 + . + 空格键`
2. 有序列表项 二
    1. 有序列表项 二(1) `TAB(或4个空格) + 数字 + . + 空格键`
    2. 有序列表项 二(2)
        1. 有序列表项 二(2).1
3. 有序列表项 三
```
1. 有序列表项 一 `数字 + . + 空格键`
2. 有序列表项 二
   1. 有序列表项 二(1) `TAB(或4个空格) + 数字 + . + 空格键`
   2. 有序列表项 二(2)
      1. 有序列表项 二(2).1
3. 有序列表项 三
## 公式
### 行内公式
用 `$` 符号扩起来，注意 `$` 和公式之间不要留空格
```
$a + b = c$
```
$a + b = c$
### 块级公式
两美元写在公式块的上一行和下一行
```
$$
\sum_{i=1}^{n} i = \frac{n(n+1)}{2}
$$
```
$$
\sum_{i=1}^{n} i = \frac{n(n+1)}{2}
$$

# 链接

`[]()` 来插入链接

 `!()[]` 来插入图片（可以是本地图片也可以是网络图片)

## 任务列表

- [ ] 任务一 未做任务 `- [ ] write sth.` （注意[ ]中间有空格）
- [x] 任务二 已做任务 `- [x] write sth.`

## 表格
第一行为表头，第二行分隔表头和主体部分(如果表格无法显示可以尝试把第二行的 `-` 变为 `---` )，可以指定所在列的对齐方式，第三行开始每一行为一个表格行。列与列之间用 `|` 隔开。(注：原生方式的表格每一行的两边也要有`|` )

**对齐方式** `:- 左对齐 - 中心对齐 -: 右对齐`

```
第一列|第二列|第三列
:-|-|-:
a11|a12|a13
a21|a22|a33
a31|a32|a33
```
## 代码块
### 行内代码块
用`左右包裹代码
### 多行代码块
用```上下包裹代码，在第一个三撇后加语言名称获得不同的高亮效果
## 对齐方式
```
<center>行中心对齐</center>
<p align="left">行左对齐</p>
<p align="right">行右对齐</p>
```
## 分割线
```
* * *
***
- - -
---
```
## 换行
不同markdown编辑器可能有不同的换行方式，最简单为直接敲回车

markdown文本内的连续两个或多个回车会被替换为一个回车
## 高级
### 设置字体/颜色
```
<font face="宋体" color=blue size=5>蓝色的字～</font>
```
#### 常用颜色
| 最常用                                                       | 其他                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| <font face="Times new roman" color=red size=3>red</font>     | <font face="Times new roman" color=greenyellow size=3>greenyellow</font> |
| <font face="Times new roman" color=orange size=3>orange</font> | <font face="Times new roman" color=lightgreen size=3>lightgreen</font> |
| <font face="Times new roman" color=yellow size=3>yellow</font> | <font face="Times new roman" color=lightblue size=3>lightblue</font> |
| <font face="Times new roman" color=green size=3>green</font> | <font face="Times new roman" color=pink size=3>pink</font>   |
| <font face="Times new roman" color=aqua size=3>aqua</font>   | <font face="Times new roman" color=gold size=3>gold</font>   |
| <font face="Times new roman" color=blue size=3>blue</font>   | <font face="Times new roman" color=silver size=3>silver</font> |
| <font face="Times new roman" color=purple size=3>purple</font> | <font face="Times new roman" color=brown size=3>brown</font> |

### 设置高亮颜色

```(空)
<table>   <tr>     <td bgcolor=Plum>like this</td>   </tr> </table>
```

<table>   <tr>     <td bgcolor=Plum>like this</td>   </tr> </table>

