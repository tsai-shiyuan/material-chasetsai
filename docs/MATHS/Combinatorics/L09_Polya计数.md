# Pólya计数

> 给可以自由转动的正多边形的顶点着色，计数着色方案

## 置换群

### 置换群

**群:** 

给定集合 $G$ 和 $G$ 上的二元运算 $\cdot$ ，代数结构 $(G,\cdot)$ 为**群**要满足:

- 封闭性: 对任意 $a,b \in G$ ，都有 $a\cdot b \in G$
- 结合律: 对任意 $a,b,c\in G$ ，都有 $a \cdot (b \cdot c) = (a\cdot b) \cdot c$
- 存在单位元: 存在 $e\in G$ ，对于任意 $a\in G$ ，满足 $e \cdot a = a \cdot e = a$
- 存在逆元: 对任意 $a\in G$ ，存在 $a^{-1} \in G$ ，满足 $a\cdot a^{-1} = a^{-1}\cdot a = e$

$|G|:$ 阶，即有限群 $G$ 的元素个数

循环群与生成元: 

- 在群 $(G, \cdot)$ 中，若存在 $a\in G$ ，$G$ 中的任意元素 $b$ 均能表示成 $a$ 的幂，则称 $G$ 为循环群
- $a$ 为该群的生成元

**置换:**

设 $X$ 是有限集，就设为 $\{1,2,\cdots,n\}$ ，$X$ 的**置换** $i_1,i_2, \cdots, i_n$ 是 $X$ 到自身的一对一**函数** $f: X \rightarrow X$ ，其中 $f(1)=i_1, f(2)=i_2, \cdots, f(n)=i_n$

$2 \times n$ 阵列表示置换:
$$
\begin{pmatrix}
1 & 2 & \cdots & n \\
i_1 & i_2 & \cdots & i_n \\
\end{pmatrix}
$$

- **置换是函数**，可以使用合成运算 $g \circ f$
- 记 $n!$ 个置换构成的集合为 $S_n$
- 恒等置换: $\iota =\begin{pmatrix}
  1 & 2 & \cdots & n \\
  1 & 2 & \cdots & n \\
  \end{pmatrix}$

**置换群:**

![截屏2025-06-03 22.28.28](./images_notes/%E6%88%AA%E5%B1%8F2025-06-03%2022.28.28.png)

例子: $\rho_n$ 表示置换:

$$
\rho_n = 
\begin{pmatrix}
1 & 2 & \cdots & n-1 & n \\
2 & 3 & \cdots & n & 1 \\
\end{pmatrix}
$$

从而

$$
\rho_n^k = 
\begin{pmatrix}
1 & 2 & \cdots & \cdots & n \\
k+1 & k+2 & \cdots & \cdots & k \\
\end{pmatrix}
$$

> 可以想象为 $1$ 转到了原来 $k+1$ 的位置上

![截屏2025-06-03 22.45.10](./images_notes/%E6%88%AA%E5%B1%8F2025-06-03%2022.45.10.png)

得到 $C_n = \{\rho_n^0=\iota, \rho_n, \rho_n^2,\cdots,\rho_n^{n-1}\}$ 是一个置换群

**对称:**

`定义` 设 $\Omega$ 是一个几何图形，$\Omega$ 到它自身的一个几何运动 (motion) 或全等 (congruence) 称为 $\Omega$ 的一个对称（想象成一个框在空间中动）

几何运动包括

- 旋转
- 反射 (即沿轴线翻转，中途离开平面): 
  - 顶点数为偶: 对角点连线 + 对边中点
  - 顶点数为奇: 角点与对边连线

**角点对称群:** 关注点的位置变化

- 例如五边形的角点对称群:

<img src="./images_notes/%E6%88%AA%E5%B1%8F2025-06-04%2010.01.28.png" alt="截屏2025-06-04 10.01.28" style="zoom: 33%;" />

<img src="./images_notes/%E6%88%AA%E5%B1%8F2025-06-04%2010.01.39.png" alt="截屏2025-06-04 10.01.39" style="zoom:33%;" />

> 原来的第一排到了原来的第二排的位置
>
> 可以想象为从固定位置开始读现在的点的编号

### 着色

**着色:**

`定义` 集合 $X$ 的一种着色是给 $X$ 的每个元素指定一种颜色

假设有集合 $X$ 的置换群 $G$ ，设

- $\mathbf{c}$ 是 $X$ 的一种着色，$1, 2, \cdots, n$ 的颜色分别记为 $c(1),c(2),\cdots,c(n)$ 
- $f$ 是 $G$ 中的一个置换，$f = \begin{pmatrix}
  1 & 2 & \cdots & n \\
  i_1 & i_2 & \cdots & i_n \\
  \end{pmatrix}$

定义 $f \ast \mathbf{c}$ 为使得 $i_k$ 具有颜色 $c(k)$ 的**着色**，即 $(f \ast \mathbf{c})(i_k) = c(k)$ ，即把 $k$ 的颜色转移给了 $i_k$

**着色集:**

着色集 $C$ 是 $X$ 的着色的集合，并且要满足对 $G$ 中的任意元 $f$ 和 $C$ 中任意元 $\mathbf{c}$ ，$f \ast \mathbf{c}$ 仍属于 $C$

**一个相等关系:**
$$
(g\circ f)\ast \mathbf{c} = g \ast (f \ast \mathbf{c})
$$
理解: 等式左边是把 $k$ 的颜色转移到 $(g \circ f)(k)$ 的着色，右边是把 $k$ 的颜色先转移给 $f(k)$ 再转移给 $g(f(k))$

**着色等价:** 

`定义` 设 $\mathbf{c}_1$ 与 $\mathbf{c}_2$ 是 $C$ 中的两种着色，如果存在 $G$ 中的置换，使得
$$
f\ast \mathbf{c}_1 = \mathbf{c}_2
$$
则称 $\mathbf{c}_1$ 等价于 $\mathbf{c}_2$ ，记为 $\mathbf{c}_1 \sim \mathbf{c}_2$

> 即存在 $G$ 中的一个置换，把一个着色送到另一个着色
>
> 我们接下来就要计算非等价的着色数

## Burnside定理

### 稳定核与不变着色集

适当选取 $f$ 与 $\mathbf{c}$ ，可使得 $f \ast \mathbf{c} = \mathbf{c}$

- 使着色 $\mathbf{c}$ 保持不变的 $G$ 中置换的集合: $G(\mathbf{c}) = \{f: f\in G, f\ast \mathbf{c} = \mathbf{c}\}$
- 在 $f$ 作用下使着色 $\mathbf{c}$ 保持不变的 $C$ 中着色的集合: $C(f) = \{\mathbf{c} : \mathbf{c} \in C, f \ast \mathbf{c} = \mathbf{c}\}$

**稳定核:**

`定义` $G(\mathbf{c})$ 称为 $\mathbf{c}$ 的稳定核

`定理` 

$\mathbf{c}$ 的稳定核 $G(\mathbf{c})$ 是置换群，而且对 $G$ 中的任意置换 $f$ 与 $g$ ，$g \ast \mathbf{c} = f \ast \mathbf{c}$ 当且仅当 $f^{-1} \circ g \in G(\mathbf{c})$

证明: 

- 如果 $f$ 和 $g$ 都使 $\mathbf{c}$ 保持不变，则先 $f$ 后 $g$ 也将使 $\mathbf{c}$ 保持不变，即 $(g \circ f) \ast \mathbf{c} = \mathbf{c}$ ，所以有封闭性；单位元 $\iota$ ；逆元 $f^{-1}$ $\Rightarrow$ 置换群
- $f \ast \mathbf{c} = g \ast \mathbf{c} \Rightarrow (f^{-1} \circ g) \ast \mathbf{c} = f^{-1} \ast (g \ast \mathbf{c}) = f^{-1} \ast (f \ast \mathbf{c}) = (f^{-1} \circ f) \ast \mathbf{c} = \mathbf{c}$ 
- $(f^{-1} \circ g) \ast \mathbf{c} = \mathbf{c} \Rightarrow f^{-1}\ast(g \ast \mathbf{c}) = \mathbf{c} \Rightarrow f \ast f^{-1}\ast(g \ast \mathbf{c}) = f \ast \mathbf{c} \Rightarrow g \ast \mathbf{c} = f \ast \mathbf{c}$

`推论` 设 $\mathbf{c}$ 为 $C$ 中的一个着色，那么与 $\mathbf{c}$ 等价的着色数 $|\{f \ast \mathbf{c}: f\in G\}|$ 等于
$$
\frac{|G|}{|G(\mathbf{c})|}
$$
证明: 

设 $f$ 是 $G$ 中的置换，那么满足 $g \ast \mathbf{c} = f \ast \mathbf{c}$ 的置换 $g$ 实际上就是 $\{f \circ h : h \in G(\mathbf{c})\}$ 中的那些置换，由消去律，$f \circ h = f \circ h'$ 可推出 $h = h'$ ，所以 $\{f \circ h : h \in G(\mathbf{c})\}$ 中的置换个数等于 $|G(\mathbf{c})|$ 

对每个 $f$ ，恰好存在 $|G(\mathbf{c})|$ 个置换，这些置换作用在 $\mathbf{c}$ 上与 $f$ 有同样的效果，所以，与 $\mathbf{c}$ 等价的着色数等于 $\frac{|G|}{|G(\mathbf{c})|}$ 

>"等价的着色"肯定互不相同，而且能通过置换变到 $\mathbf{c}$

### Burnside定理

`定理` 设 $G$ 是 $X$ 的置换群，而 $C$ 是 $X$ 中的满足后面条件的着色集合: 对于 $G$ 中所有的 $f$ 和 $C$ 中所有的 $\mathbf{c}$ 都有 $f \ast \mathbf{c}$ 仍在 $C$ 中，则 $C$ 中非等价着色数 $N(G,C)$ 为

$$
N(G,C) = \frac{1}{|G|}\sum\limits_{f \in G}|C(f)|
$$

> 就是在置换中保持不变的着色的平均数

`证明`

我们计数满足 $f \ast \mathbf{c} = \mathbf{c}$ 的对偶 $(f,\mathbf{c})$ 的个数

- 方法1: 遍历 $G$ 中的 $f$ ，得到 $\sum\limits_{f \in G}|C(f)|$
- 方法2: 遍历 $C$ 中的 $\mathbf{c}$ ，得到 $\sum\limits_{\mathbf{c} \in C} |G(\mathbf{c})| = \sum\limits_{\mathbf{c} \in C}\frac{|G|}{与 \mathbf{c} 等价的着色数}$ ，等式右边按照等价类来算，每个等价类的求和是 $1$，所以总体等于 $N(G,C) \times |G|$
- 两个方法计数的结果相同

例题:

用红色、蓝色与绿色对正五边形的顶点进行着色，有多少种非等价方法？

- 恒等置换 $\iota$ 使 $3^5 = 243$ 种着色保持不变
- 其他旋转只有当全着一种颜色才能保持着色不变，各 $3$ 种
- 每个反射使 $3\times 3 \times 3 = 27$ 种着色保持不变

所以非等价着色数为 $N(D_5,C) = \frac{1}{10}(243 + 3*4 + 5*27) = 39$

> 最复杂是 $|C(f)|$ 的计算，下一小节我们将引入循环结构，简化计算过程

## Pólya计数

### 有向图与循环
有向图的构建: 设 $f$ 是 $X = \{1,2,\cdots,n\}$ 的一个置换，$D_f = (X, A_f)$ 是顶点集为 $X$ 且弧集为 $A_f = \{(i,f(i)) : i \in X\}$ 的有向图

我们可以发现，这个图是由许多有向圈构成的

例子: $f = \begin{pmatrix}
  1 & 2 & 3 & 4 & 5 & 6 & 7 & 8 \\
  6 & 8 & 5 & 4 & 1 & 3 & 2 & 7 \\
  \end{pmatrix}$

$D_f$ 划分为这样的有向圈: $1 \rightarrow 6 \rightarrow 3 \rightarrow 5 \rightarrow 1, 2 \rightarrow 8 \rightarrow 7 \rightarrow 2, 4 \rightarrow 4$

我们保持其他位置不变，记 

$$
[1 \space 6 \space 3 \space 5] = \begin{pmatrix}
  1 & 2 & 3 & 4 & 5 & 6 & 7 & 8 \\
  6 & 2 & 5 & 4 & 1 & 3 & 7 & 8 \\
  \end{pmatrix}
$$

像这样的，一个置换中某些元素以循环的方式置换且余下的元素保持不变，则称这样的置换为 `循环置换` / `循环` 。循环中的元素为 k，则称循环为 `k循环` 。例如 `[1 6 3 5]` 是一个4循环

$$
f = \begin{pmatrix}
  1 & 2 & 3 & 4 & 5 & 6 & 7 & 8 \\
  6 & 8 & 5 & 4 & 1 & 3 & 2 & 7 \\
  \end{pmatrix} = [1 \space 6 \space 3 \space 5] \circ [2 \space 8 \space 7] \circ [4]
$$

我们称此式为 $f$ 的 `循环因子分解`

记置换 $f$ 的循环分解中的循环个数为 $\# (f)$

### 着色

我们可以发现，颜色只会在每个循环内部进行转移，所以只要保证循环的颜色一样就能满足 $f * \mathbf{c} = \mathbf{c}$

`定理` 设 $f$ 是集合 $X$ 的置换，假如我们用 $k$ 种颜色对 $X$ 的元素进行着色，设 $C$ 是 $X$ 的所有着色的集合，则 $f$ 中保持不变的着色数为
$$
|C(f)| = k^{\#(f)}
$$

### 置换的生成函数

设 $f$ 是一个置换，$f$ 的循环因子分解有 $e_1$ 个 1 循环，$e_2$ 个 2 循环，...，$e_n$ 个 n 循环，则有:

$$
1e_1 + 2e_2 + \cdots + ne_n = n
$$

称 $n$ 元组 $(e_1,e_2,\cdots, e_n)$ 是置换 $f$ 的类型，记为

$$
\text{type}(f) = (e_1, e_2, \cdots, e_n)
$$

循环数 $\# (f) = e_1 + e_2 + \cdots + e_n$

>是否可以仅通过类型来区分置换？

引进 $n$ 个未定元 $z_1, z_2,\cdots, z_n$ ，$z_k$ 对应 $k$ 循环

- 定义 $f$ 的单项式为 $\text{mon}(f) = z_1^{e_1} z_2^{e_2} \cdots z_n^{e_n}$
- $G$ 是置换群，对 $G$ 中每个置换 $f$ 的单项式求和，得到**置换的生成函数**

$$
\sum\limits_{f\in G}\text{mon}(f) = \sum\limits_{f\in G}z_1^{e_1} z_2^{e_2} \cdots z_n^{e_n}
$$

- 定义 $G$ 的 `循环指数` 为 $P_G(z_1, z_2, \cdots, z_n) = \frac{1}{|G|}\sum\limits_{f\in G}z_1^{e_1} z_2^{e_2} \cdots z_n^{e_n}$
  - 例如，二面体群 $D_4$ 的循环指数:
    ![alt text](<images_notes/截屏2025-06-18 10.11.59.png>)

`定理` 设 $X$ 是有 $n$ 个元素的集合，假设我们用 $k$ 种颜色对 $X$ 的元素进行着色，$C$ 是着色集合，$G$ 是 $X$ 的置换群，则非等价的着色数是用 $z_i = k$ 代入 $G$ 的循环指数中得到的值，即 

$$
N(G,C) = P_G(k,k,\cdots, k)
$$

>问题: 各颜色只能使用特定次数的非等价着色数？

### Pólya定理

假设只有红蓝两种颜色，$C_{p,q}$ 表示有 $p$ 个元素着红色，$q = n-p$ 个元素着蓝色的着色集合

进一步，这类着色 (非等价着色) 的个数为下式中 $r^pb^q$ 的系数

$$
\frac{1}{|G|}\sum\limits_{f\in G}(r + b)^{e_1}(r^2+b^2)^{e_2}\cdots (r^n+b^n)^{e_n}
$$ 

实际上就是对 $f$ 的单项式做代换

$$
z_1 = r + b, z_2 = r^2 + b^2, \cdots, z_n = r^n + b^n
$$

`Pólya定理` 设 $G$ 是 $X$ 的置换群，$\{u_1, u_2, \cdots, u_k\}$ 是 $k$ 种颜色的集合，$C$ 是 $X$ 的任意着色集。这时针对个颜色的数目的 $C$ 的非等价着色数的生成函数是

$$
P_G(u_1 + \cdots + u_k, u_1^2 + \cdots + u_k^2,\cdots, u_1^n + \cdots + u_k^n)
$$

其中 $u_1^{p_1} u_2^{p_2}\cdots u_k^{p_k}$ 的系数等于 $C$ 中把 $X$ 的 $p_1$ 个元素着色成颜色 $u_1$ ，$p_2$ 个元素着色成颜色 $u_2$，……，$p_k$ 个元素着色成颜色 $u_k$ 的**非等价着色数**

>可以计算同构的图？！

