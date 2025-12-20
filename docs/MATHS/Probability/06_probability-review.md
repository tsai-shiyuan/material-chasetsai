# BUAA｜概统复习

一些资料

- [MIT Lecture Notes](https://ocw.mit.edu/courses/6-041-probabilistic-systems-analysis-and-applied-probability-fall-2010/pages/lecture-notes/)
- [浙大第四版《概率论与数理统计》课后习题答案](https://zhuanlan.zhihu.com/p/358299124)

## 01 随机试验的概率

### 随机试验

随机试验 ($E$):

- 可在相同的条件下重复进行
- 能事先明确试验的所有可能结果
- 试验之前，不能确定哪一结果会出现

样本空间 ($S$): 随机试验 $E$ 所有可能的结果组成的集合

样本点: $E$ 的每个结果

随机事件 ($A$): 样本空间 $S$ 的子集

### 条件概率与乘法公式

条件概率: 在事件 $A$ 发生的条件下事件 $B$ 发生的概率

$$
P(B|A) = \frac{P(AB)}{P(A)}
$$

变形得乘法公式:

$$
P(AB) = P(B|A) P(A)
$$

### 全概率公式与贝叶斯公式

![alt text](<images_review/截屏2024-11-30 10.05.29.png>)

![alt text](<images_review/截屏2024-11-30 10.03.53.png>)

贝叶斯公式:

$$
P(B_i|A) = \frac{P(AB_i)}{P(A)} = \frac{P(B_i)P(A|B_i)}{\sum\limits_{j=1}^{n}P(B_j)P(A|B_j)}
$$

把 $B$ 视作原因，$A$ 视作结果，已知原因 $\rightarrow$ 结果。我们现在假设结果发生，想知道哪个因素导致的 $A$ 发生

### 独立性

当事件 $A$ 的发生不影响事件 $B$ 发生的概率时，则事件 $A$ 和事件 $B$ 独立，此时有 $P(B|A) = P(B)$， 进而 $P(AB) = P(A)P(B)$

三事件相互独立需同时满足:

![alt text](<images_review/截屏2024-11-30 10.42.01.png>)

![](<images_review/截屏2024-11-30 10.41.09.png>)

## 02 随机变量的分布

### 随机变量

设随机试验 $E$ 的样本空间为 $S=\{e_1, e_2, ...\}$，对每一个 $e \in S$，都有唯一的一个实数 $X(e)$ 与之对应，则称 $X = X(e)$ 为随机变量，简记为 $X$

随机事件的概率问题就转化为随机变量取值的概率问题

分布函数: $F(x) = P(X \leqslant x), -\infty < x < +\infty$

- 范围: $0 \leqslant F(x) \leqslant 1$ 且 $\lim\limits_{x\rightarrow+\infty} F(x) = 1, \lim\limits_{x\rightarrow-\infty} F(x) = 0$
- 单调不减: $x_1 < x_2 \Rightarrow F(x_1) \leqslant F(x_2)$
- 右连续: $F(x+0) = F(x)$

具备上述三个性质的函数 $F(x)$ 都可以是某一随机变量的分布函数

分布律: $p(x)$

---

==离散型随机变量的分布==

### 两点分布/01分布

$$
\begin{cases}
 & P\{X = 1\} = p,\\
 & P\{X = 0\} = 1-p 
\end{cases}
$$

### 二项分布

$n$ 次试验，事件 $A$ 出现 $k$ 次的概率

<img src="images_review/截屏2024-12-01 14.56.27.png" alt="alt text" style="zoom: 33%;" />

称 $X$ 服从参数为 $n,p$ 的二项分布，$X \sim B(n,p)$

泊松定理: 当 $n$ 较大，$p$ 较小，$\lambda = np$ 适中，有近似公式:

$$
C_{n}^{k} p^k (1-p)^{n-k} \approx e^{-\lambda}\frac{\lambda ^k}{k!}
$$

### Poisson分布

$P(X=k) = e^{-\lambda}\frac{\lambda ^k}{k!}$，则 $X \sim \Pi(\lambda)$ (或记为 $X \sim P(\lambda)$)

---

==连续型随机变量的分布==

### 均匀分布

$$
 f(x) = \begin{cases}
\frac{1}{a-b}, & \text{} a < x < b \\
 0, & \text{else} 
\end{cases}
$$

$X \sim U(a, b)$

### 指数分布

$$
f(x) = \begin{cases}
\lambda e^{-\lambda x}, & \text{} x > 0\\
 0, & \text{else} 
\end{cases}
$$

$X \sim E(\lambda)$

### 高斯分布/正态分布

$$
f(x) = \frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(x-\mu)^2}{2\sigma^2}}, -\infty < x < +\infty
$$

$X \sim N(\mu, \sigma^2)$

标准正态分布 $N(0,1)$: $\varphi (x) = \frac{1}{\sqrt{2\pi}}e^{-\frac{x^2}{2}}, -\infty < x < +\infty$, 分布函数记为 $\phi(x)$

一般正态分布 $N(\mu, \sigma^2)$ 的分布函数 $F(x) = \phi(\frac{x-\mu}{\sigma})$

### 随机变量的函数

已知 $X$ 的概率密度 $f_{X}(x)$，求 $Y=g(X)$ 的概率密度?

求分布函数 $F_{Y}(y) \Rightarrow$ 求导得概率密度 $f_{Y}(y)$

## 03 二维随机变量

### 分布函数

分布函数: $F(x,y) = P(X \leqslant x, Y \leqslant y)$

二维离散型有联合分布律: $P(X = x_i, Y = y_i) = p_{ij}$

二维连续型有联合密度函数: $f(x,y)$, 满足 $F(x,y) = \int_{-\infty}^{x}\int_{-\infty}^{y}f(u,v) \mathrm{d}u\mathrm{d}v$

### 常见连续型分布

均匀分布:

$$
f(x,y)=\begin{cases}
 \frac{1}{A},& \text{} (x,y) \in G \\
 0,& \text{else}
\end{cases}
$$

$A$ 为区域 $G$ 的面积，$(X,Y) \sim U(G)$

二维正态分布:

$$
 f(x,y) = \frac{1}{2 \pi \sigma_1 \sigma_2 \sqrt{1-\rho^2}}e^{-\frac{1}{2(1-\rho^2)}[\frac{(x-\mu_1)^2}{\sigma_1^2}+\frac{(y-\mu_2)^2}{\sigma_2^2}-2\rho \frac{(x-\mu_1)(y-\mu_2)}{\sigma_1 \sigma_2}]}, |\rho| <1
$$

$(X,Y) \sim N(\mu_1, \sigma_1^2, \mu_2, \sigma_2^2, \rho)$

性质:

1. 两个边缘分布为正态分布，$X \sim N(\mu_1, \sigma_1^2)$
2. $X,Y$ 的线性组合仍服从正态分布

### 边缘分布函数

$F_X(x) = P(X \leqslant x) = F(x, +\infty)$ 

$F_Y(y) = P(Y \leqslant y) = F(+\infty, y)$

离散型有边缘分布律: 各行或列相加

连续型有边缘密度函数: $f_X(x) = \int_{-\infty}^{+\infty}f(x,v)\mathrm{d}v$

### 条件分布律 (离散)

在 $X = x_i$ 的条件下，$Y$ 的条件分布律: $P(Y=y_j|X=x_i)=\frac{P(X=x_i,Y=y_j)}{P(X=x_i)} = \frac{p_{ij}}{p_{i.}}$

### 条件分布函数 (连续)

$F_{X|Y}(x|y) = P(X \leqslant x | Y = y) = \frac{P(X \leqslant x, Y=y)}{P(Y=y)} = \frac{0}{0}$

(因为体积为 0)

推导得: $P(X \leqslant x | y < Y \leqslant y + \varepsilon)=\int_{-\infty}^{x}\frac{f(x,y)}{f_Y(y)}\mathrm{d}x$

$\frac{f(x,y)}{f_Y(y)}$ 为在 $Y=y$ 条件下 $X$ 的**条件概率密度**，记为 $f_{X|Y}(x|y)=\frac{f(x,y)}{f_Y(y)}$

### 独立性

将事件的独立性推广到随机变量

`定义` 若 $P(X \leqslant x, Y \leqslant y) = P(X \leqslant x) P(Y \leqslant y)$ 对任意 $x,y$ 都成立，则称随机变量 $X,Y$ 独立

由定义得出: $(X,Y)$ 独立 $\iff$ $F(x,y) =F_X(x)F_Y(y) \iff P(a < X \leqslant b,c < Y \leqslant d)=P(a < X \leqslant b)P(c < Y \leqslant d)$

离散型 $(X,Y)$ 相互独立 $\iff P(X=x_i,Y=y_j)=P(X=x_i)P(Y=y_j)$ (由第二个等价式推得)

连续型 $(X,Y)$ 相互独立 $\iff f(x,y) = f_X(x)f_Y(y)$

## 04 随机变量的数字特征

### 数学期望

离散: $E(X) = \sum\limits_{k=1}^{+\infty}x_k p_k$

连续: $E(X) = \int_{-\infty}^{+\infty}xf(x)\mathrm{d}x$

常见分布的期望:

- $X \sim B(n,p) \Rightarrow E(X) = np$

- $X \sim \Pi(\lambda) \Rightarrow E(X) = \lambda$ (泊松分布)

- $X \sim E(\lambda) \Rightarrow E(X) = \frac{1}{\lambda}$ (指数分布)

- $X \sim N(\mu,\sigma^2) \Rightarrow E(X) = \mu$

若 $Y=g(X)$，则 $E(Y) = \int_{-\infty}^{+\infty}g(x)f(x)\mathrm{d}x$

性质:

- $E(X+Y) = E(X) + E(Y)$
- 当 $X,Y$ 相互独立时，$E(XY) = E(X) E(Y)$🌟

### 方差

随机变量 $X$ 的方差: $D(X) = E((X-E(X))^2)$

方差公式: $D(X) = E(X^2) - E^2(X)$

泊松分布的方差: $X \sim P(\lambda), D(X) = \lambda$

<img src="images_review/截屏2024-12-05 15.57.26.png" alt="alt text" style="zoom: 33%;" />

正态分布的方差: $X \sim N(\mu, \sigma^2), D(X)=\sigma^2$

性质:

- $D(aX) = a^2D(X)$
- $D(X \pm Y) = D(X) + D(Y) \pm 2E((X-E(X))(Y-E(Y)))=D(X) + D(Y) \pm 2(E(XY)-E(X)E(Y))$

### 协方差

>期望和方差反映随机变量自身的数字特征，协方差和相关系数描述随机变量之间的相关关系

$Cov(X,Y) = E((X-E(X))(Y-E(Y))) = E(XY) - E(X)E(Y)$

### 相关系数

$\rho _{XY} = E(\frac{X-E(X)}{\sqrt{D(X)}}.\frac{Y-E(Y)}{\sqrt{D(Y)}}) = \frac{Cov(X,Y)}{\sqrt{D(X)}\sqrt{D(Y)}}$

若 $\rho_{XY} = 0$，称 $X,Y$ 不相关

$\rho_{XY} = 0 \iff X, Y$ 不相关 $\iff Cov(X,Y) = 0 \iff E(XY) = E(X)E(Y)$ 

### 独立性与相关性

$E(XY) = E(X) E(Y) \Rightarrow Cov(X,Y)=0 \Rightarrow \rho_{XY} = 0 \Rightarrow X, Y$ 不相关

独立 $\Rightarrow$ 不相关

若 $X , Y$ 服从二维正态分布，则 $X,Y$ 相互独立 $\iff$ $X,Y$ 不相关

## 05 大数定律和中心极限定理

[notes](https://tsai-shiyuan.github.io/material-chasetsai/M%26P/Probability/02_大数定律%2B中心极限定理)

一定条件下，$Y_n\sim B(n,p)$ 可近似为 $Y_n \sim N(np, np(1-p))$

## 06 样本及抽样分布

### 总体与样本

从总体 $X$ 中，随机抽取 $n$ 个个体，得到样本 $(X_1, X_2, \cdots, X_n)$，每个样本的分布与 $X$ 同，记 $(x_1, x_2, \cdots, x_n)$ 为样本值

### 统计量

设 $(X_1, X_2, \cdots, X_n)$ 是取自总体的一个样本，$g(r_1, r_2, \cdots,r_n)$ 为一不含参数的连续实值函数，则称随机变量 $g(X_1, X_2, \cdots, X_n)$ 为统计量

常用统计量:

- 样本均值: $\bar{X} = \frac{1}{n}\sum\limits_{i=1}^{n}X_i$
- 样本方差: $S^2 =\frac{1}{n-1}\sum\limits_{i=1}^{n}(X_i-\bar{X})^2$
- $k$ 阶原点矩: $A_k = \frac{1}{n}\sum\limits_{i=1}^{n}X_i^k$
- $k$ 阶中心矩: $B_k =\frac{1}{n}\sum\limits_{i=1}^{n}(X_i-\bar{X})^k$
- 二阶中心矩: $S_n^2=\frac{1}{n}\sum\limits_{i=1}^{n}(X_i-\bar{X})^2$

可以证明: $E(S) = \sigma^2$

!!! abstract "Note"
    若总体服从正态分布 $N(\mu, \sigma^2)$，则样本均值 $\bar{X}$ 服从正态分布 $N(\mu, \frac{\sigma^2}{n})$

### $\chi^2(n)$ 分布

设 $(X_1, X_2, \cdots, X_n)$ 相互独立，且 $X_i \sim N(0,1)$，则称统计量 $\chi^2 = X_1^2 + X_2^2 + \cdots + X_n^2$ 服从自由度为 $n$ 的 $\chi^2(n)$ 分布，即 $\chi^2 \sim \chi^2(n)$

1. $E(\chi^2) = n, D(\chi^2) = 2n$
2. 若 $X_1 \sim \chi^2(n_1),X_2 \sim \chi^2(n_2)$，$X_1,X_2$ 相互独立，则 $X_1 + X_2 \sim \chi^2(n_1 + n_2)$
3. $\chi^2(n)$ 的 $\alpha$ 分位数有表可查

### $T$ 分布

设 $X \sim N(0,1),Y \sim \chi^2(n)$，$X,Y$ 相互独立，则 $T = \frac{X}{\sqrt{Y/n}}$ 服从自由度为 $n$ 的 $T$ 分布

1. $f_n(t)$ 是偶函数
2. $n \rightarrow \infty$，$f_n(t)$ 曲线趋于标准正态分布密度曲线

### $F$ 分布

设 $X \sim \chi^2(n),Y \sim \chi^2(m)$，$X,Y$ 相互独立，则 $F = \frac{X/n}{Y/m}$ 服从第一自由度为 $n$，第二自由度为 $m$ 的 $F$ 分布，即 $F \sim F(n,m)$

1. 若 $F \sim F(n,m)$，则 $\frac{1}{F} \sim F(m,n)$
2. $F_{1-\alpha}(n,m) = \frac{1}{F_{\alpha}(m,n)}$

### 抽样分布

设 $X_1, X_2, \cdots, X_n$ 是来自总体 $N(\mu,\sigma^2)$ 的样本，则:

1. $\frac{(n-1)S^2}{\sigma^2} \sim \chi^2(n-1)$ 🌟
2. $\bar{X}$ 与 $S^2$ 相独立
3. $\frac{\bar{X}-\mu}{S/\sqrt{n}}\sim t(n-1)$ (由1推)

ps. $\frac{\bar{X}-\mu}{\sigma / \sqrt{n}} \sim N(0,1)$

设 $X_1, X_2, \cdots, X_{n_1}$ 和 $Y_1, Y_2, \cdots, Y_{n_2}$ 分别是来自正态总体 $N(\mu_1,\sigma_1^2)$ 和 $N(\mu_2, \sigma_2^2)$ 的样本，且两个样本相互独立，则:

![alt text](<images_review/截屏2024-12-10 21.39.28.png>)

## 07 参数估计

>$X$ 的分布函数已知，但参数未知 $\Rightarrow$ 通过样本来估计

### 矩估计法 (点估计)

$\frac{1}{n}\sum\limits_{i=1}^{n}X_i =  \hat{E}(X) \rightarrow E(X)$

$\frac{1}{n}\sum\limits_{i=1}^{n}X_i^2 =  \hat{E}(X^2) \rightarrow E(X^2)$

>用的是原点矩。我们已知左边的量，以此估计右边。

例题: 设总体 $X \sim E(\lambda)$，$X_1, X_2, \cdots, X_n$ 为总体的样本，求 $\lambda$ 的矩法估计量

解: $E(X)=\frac{1}{\lambda} \rightarrow \hat{E}(X) = \frac{1}{\hat{\lambda}} = \bar{X} \rightarrow \hat{\lambda} = \frac{1}{\bar{X}}$

![alt text](<images_review/截屏2024-12-19 19.13.10.png>)

### 极大似然法 (点估计)

>箱子1: 99白，1红；箱子2: 99红，1白。取一次，得白球，问: 从哪箱取的？

$P(X = x) = f(x,\theta)$

$P(X_1 = x_1, X_2 = x_2, \cdots X_n = x_n) = f(x_1, \theta)f(x_2, \theta)\cdots f(x_n,\theta) = L(x_1,x_2,\cdots, \theta) = L(\theta)$

$L(\theta)$ 称为似然函数，取 $\theta$ 使得 $L(\theta)$ 取到极大值

例题: 总体 $X$ 服从 0-1 分布，且 $P(X=1) =p$，估计 $p$ 的值？

解:

<img src="images_review/截屏2024-12-11 13.01.37.png" alt="alt text" style="zoom:33%;" />

现求 $p$ 使得 $L(p)$ 取到最大值 



极大似然估计值不变性原理: $\hat{\theta}$ 是 $\theta$ 的极大似然估计值，则 $\mu(\hat{\theta})$ 是 $\mu(\theta)$ 的极大似然估计值

### 点估计的评价标准

无偏性: 设$(X_1, X_2, \cdots, X_n)$ 是总体 $X$ 的样本，$\hat{\theta}=\hat{\theta}(X_1,X_2, \cdots, X_n)$ 是 $\theta$ 的估计量，若 $E(\hat{\theta}) = \theta$，则 $\hat{\theta}$ 是无偏估计量

例如: 

- 样本均值 $\bar{X}$ 是总体期望 $E(X)$ 的无偏估计量
- 样本二阶原点矩 $A_2 = \frac{1}{n}\sum\limits_{i=1}^nX_i^2$ 是总体二阶原点矩 $\mu_2 = E(X^2)$ 的无偏估计量
- 样本方差 $S^2 =\frac{1}{n-1}\sum\limits_{i=1}^{n}(X_i - \bar{X})^2$ 是 $D(X)$ 的无偏估计量

有效性: 设 $\hat{\theta_1} = \theta_1(X_1,X_2,\cdots,X_n)$ 和 $\hat{\theta_2} = \theta_2(X_1,X_2,\cdots,X_n)$ 都是总体参数 $\theta$ 的无偏估计量，且 $D(\hat{\theta_1}) < D(\hat{\theta_2})$，则称 $\hat{\theta_1}$ 比 $\hat{\theta_2}$ 更有效

一致性/相合性: 

![alt text](<images_review/截屏2024-12-11 14.32.36.png>)

先算 $P(|\hat{\theta}-\theta|<\varepsilon)$，再取极限

### 区间估计

>见笔记置信区间，关键在于构建统计量

正态总体 $X \sim N(\mu, \sigma^2)$

$\sigma^2$ 已知，$\mu$ 的置信区间: $\frac{\bar{X}-\mu}{\sigma/\sqrt{n}}\sim N(0,1)$

$\sigma^2$ 未知，$\mu$ 的置信区间: $T = \frac{\bar{X}-\mu}{S/\sqrt{n}} \sim t(n-1)$

$\mu$ 已知，$\sigma^2$ 的置信区间: $Q = \sum\limits_{i=1}^{n}(\frac{X_i-\mu}{\sigma})^2 \sim \chi^2(n)$

$\mu$ 未知，$\sigma^2$ 的置信区间: $K = \frac{(n-1)S^2}{\sigma^2}\sim \chi^2(n-1)$

### 单侧置信区间

由样本 $X_1, X_2, \cdots, X_n$ 确定的统计量 $\underline{\theta}$ ，对于任何 $\theta \in \Theta$ 满足 $P\{\theta > \underline{\theta}\} \geqslant 1 - \alpha$，则称随机区间 $(\underline{\theta}, +\infty)$ 是 $\theta$ 的置信水平为 $1-\alpha$ 的单侧置信区间，$\underline{\theta}$ 称为单侧置信下限

## 08 假设检验

思想: 先对总体的参数提出假设值，再用样本的数据去验证这个假设

注意: 左边检验是 $H_1: \theta < \theta_0$

见笔记本📒🧡

### 显著性水平

在给定 $\alpha$ 的前提下，接受还是拒绝原假设完全取决于 `样本值`，因此所作检验可能导致以下两类错误的产生:

![alt text](<images_review/截屏2024-12-12 20.17.37.png>)

犯第一类错误的概率 $P(拒绝 H_0|H_0为真) = \alpha$

备择假设往往顺着题面，比如: 原来的均值是 $\mu_0$，现在有了新工艺，取到的样本均值为 $\bar{X}$，新工艺水平是否提高了？我们的原假设是 $H_0: \mu = \mu_0$，备择假设 $H_1: \mu > \mu_0$

通常把有经验的结论作为原假设，尽量让错误为第一类错误。把可能性小的作为备择假设

比如: 

- 不得低于，字面上说明 $\mu \geqslant \mu_0$ 的概率大些，所以 $\mu < \mu_0$ 是小概率事件，所以记 $H_1: \mu < \mu_0$
- 是否显著大于 $\mu_0$，显著感觉可能性会小一点，所以 $H_1: \mu > \mu_0$

### 常用的统计量

$Z$ 检验 / $U$ 检验: $Z = \frac{\bar{X}-\mu_0}{\frac{\sigma}{\sqrt{n}}} \sim N(0,1)$

$T$ 检验: $T = \frac{\bar{X}-\mu_0}{S/\sqrt{n}} \sim t(n-1)$

$T = \frac{(\bar{X}-\bar{Y})-\delta}{s_w\sqrt{\frac{1}{n_1}+\frac{1}{n_2}}} \sim t(n_1 + n_2 -2)$，$s_w^2 = \frac{(n_1-1)S_1^2 + (n_2 -1)S_2^2}{n_1 + n_2 -2}$

$\chi^2 = \frac{(n-1)S^2}{\sigma_0^2}\sim\chi^2(n-1)$

$\chi^2 = \frac{1}{\sigma_0^2}\sum\limits_{i=1}^{n}(x_i-\mu)^2 \sim \chi^2(n)$

$F$ 检验: $\frac{S_1^2/S_2^2}{\sigma_1^2/\sigma_2^2}\sim F(n_1-1,n_2-1)$

### 单侧置信区间

单侧置信区间的思想是: 我们抛开想验证的 $\mu_0, \sigma_0^2$ 不谈，假设已知的 $\bar{X},S^2$ 是符合假设 $H_0$ 的，求出一个 $\mu, \sigma^2$ 的区间 (就是置信区间)，再看 $\mu_0,\sigma_0^2$ 在不在这个区间

双边检验的置信区间形式为: $\underline{\theta}(x_1, x_2, \cdots, x_n) < \theta_0 < \overline{\theta}(x_1, x_2, \cdots, x_n)$

左边检验问题 $H_0: \theta \geqslant \theta_0, H_1: \theta < \theta_0$ 的单侧置信区间是 $(-\infty, \overline{\theta}(x_1, x_2, \cdots, x_n))$，当 $\theta_0 \in (-\infty, \overline{\theta}(x_1, x_2, \cdots, x_n))$ 时接受 $H_0$

## 09 方差分析

>没弄懂原理🫠

### 单因素试验

单因素方差分析表:

$S_E$ 是各个因素样本观察值和样本均值的差异，叫做 `误差平方和`，$S_A$ 是样本均值与数据总平均的差异

![alt text](<images_review/截屏2024-12-13 16.05.27.png>)

计算 $S_A,S_E$:

![alt text](<images_review/截屏2024-12-13 16.08.15.png>)

拒绝域是 $F \geqslant F_\alpha(s-1, n-s)$

未知参数的估计:

$\sigma^2: \hat{\sigma}^2 = \frac{S_E}{n-s}$

$\delta_j: \hat{\delta}_j = \bar{X}_{\cdot j}-\bar{X}$

### 双因素试验

<img src="images_review/截屏2024-12-13 16.35.39.png" alt="alt text" style="zoom:33%;" />


![alt text](<images_review/截屏2024-12-13 16.31.39.png>)

### 双因素无重复试验

不存在交互作用

![alt text](<images_review/截屏2024-12-13 16.35.58.png>)
方差分析表:

<img src="images_review/截屏2024-12-13 16.36.03.png" alt="alt text" style="zoom:33%;" />

### 一元线性回归分析

设随机变量 $Y$ 与 $x$ 之间存在着某种相关关系，如果 $Y$ 的数学期望存在，那么其值随 $x$ 的取值而定，是 $x$ 的函数，记为 $\mu(x)$，我们就将讨论 $Y$ 与 $x$ 的相关关系的问题转换为讨论 $E(Y)=\mu(x)$ 与 $x$ 的函数关系了。那么，现在就想用样本来估计 $Y$ 关于 $x$ 的回归函数 $\mu(x)$

![alt text](<images_review/截屏2024-12-13 16.59.02.png>)

一元线性回归要解决的问题:

1. 估计 $a,b$
2. 估计 $\sigma^2$
3. 线性假设的显著性检验
4. $b$ 的置信区间
5. 回归函数 $\mu(x) = a + bx$ 的点估计和置信区间

假设对于 $x$ 的每一个值有 $Y \sim N(a + bx, \sigma^2)$，相当于假设 $Y = a + bx + \varepsilon$，$\varepsilon \sim N(0, \sigma^2)$。再用极大似然法求得 $\hat{a},\hat{b}$

<img src="images_review/截屏2024-12-13 17.02.01.png" alt="alt text" style="zoom:33%;" />

得到 `经验回归方程`:

$$
\hat{y} = \hat{a} + \hat{b}x
$$

现估计 $\sigma^2$: $E\{[Y-(a+bx)]^2\}=E(\varepsilon^2)=D(\varepsilon)+E(\varepsilon)^2 = \sigma^2$，需要利用样本来估计 $\sigma^2$。引入残差 $y_i - \hat{y}_i$，残差平方和 $Q_e = \sum\limits_{i=1}^{n}(y_i-\hat{y}_i)^2$，且有 $\frac{Q_e}{\sigma^2} \sim \chi^2(n-2)$。$\sigma^2$ 的无偏估计量: 

$$
\hat{\sigma^2}=\frac{Q_e}{n-2} = \frac{1}{n-1}(S_{YY}-\hat{b}S_{xY})
$$

若线性假设符合实际，则 $b$ 不应该为 $0$，因此作假设 $H_0: b=0, H_1: b \neq 0$

![alt text](<images_review/截屏2024-12-13 19.07.21.png>)

当回归效果显著时，我们要对 $b$ 作区间估计

![alt text](<images_review/截屏2024-12-13 19.11.52.png>)

## 10 彩蛋 🍭

关于关联与因果:

数据与关联规则挖掘的应用: 给定一系列购物记录，捕捉其中商品共同出现的规律，从而预测其他商品的购买

eg. 啤酒尿布 🍺 (尿布和啤酒经常被一同购买)

“关联”只探讨相关性，现在我们讨论因果性

<img src="images_review/截屏2024-12-20 11.35.09.png" alt="alt text" style="zoom:33%;" />

辛普森悖论: 探究两种变量 (比如录取率与性别) 是否具有相关性时的整体趋势，与按另一个变量 (比如专业) 分组后每组趋势不同甚至相反

原因: 分组变量起到了混杂因素 (confounder) 影响，在本例中，有可能女生很多都选的是录取率低的专业

冰淇凌销量与汽车抛锚率呈正相关 $\Rightarrow$ 冰淇凌损害发动机？ (真实的原因是夏天温度高🍦)

## 11 Questions

![alt text](<images_review/截屏2024-12-27 16.54.32.png>)

![alt text](<images_review/截屏2024-12-27 16.54.47.png>)