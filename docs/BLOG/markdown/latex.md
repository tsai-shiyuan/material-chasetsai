# LaTeX

LaTeX is a form of "program code", one which specializes in document typesetting

- 在线编辑器 [overleaf](https://www.overleaf.com/project)
- 语法学习 [Learn LaTeX in 30 minutes](https://www.overleaf.com/learn/latex/Learn_LaTeX_in_30_minutes)
- 进阶 [free add-on packages](https://www.ctan.org/pkg)

## 1 文档结构

### 基本要素

```tex
\documentclass[a4paper, 12pt]{article}

\begin{document}
  A sentence of text.
\end{document}
```

`\documentclass[]{}` 的 `{}` 指定文章类型，有:

- article
- report 长文，例如博士论文
- proc 会议论文集
- book
- beamer

`\begin{document}` 前是前导命令，`\end{document}` 之后的文本会被忽视

### 标题

`\maketitle` 命令可以给文档创建标题

```tex
\begin{document}
  \title{My First Document}
  \author{My Name}
  \date{\today}
  \maketitle

  A sentence of text.
\end{document}
```

```tex
\documentclass{article}
\begin{document}
First document. This is a simple example, with no 
extra parameters or packages included.
\end{document}
```

### Preamble 序

`\begin{document}` 前的是 preamble

```tex
\documentclass[12pt, letterpaper]{article}
\usepackage{graphicx}
```

- `12pt` 设置字体大小 (默认 `10pt`)
- `letterpaper` 设置纸张大小

### Title, author, date

在 preamble 中添加

```tex
\title{My first LaTeX document}
\author{Hubert Farnsworth\thanks{Funded by the Overleaf team.}}
\date{August 2022}
```

在正文中使用 `\maketitle` 命令

```tex
\begin{document}
\maketitle
We have now added a title, author and date to our first \LaTeX{} document!
\end{document}
```

## 2 Comments

行首 `%`

## 3 Text

### Bold, italics, underlining

text formatting commands:

- **Bold**: `\textbf{...}`
- *Italics*: `\textit{...}`
- <u>Underline</u>: `\underline{...}`

\emph{*argument*} 在普通文本中 `argument` 会呈现*斜体*，在斜体中会呈现为普通文本

### Images

```tex
\documentclass{article}
\usepackage{graphicx} % LaTeX package to import graphics
\graphicspath{{images/}} % configuring the graphicx package, 是 graphicx 包中的一个命令
\begin{document}
The universe is immense and it seems to be homogeneous, 
on a large scale, everywhere we look.

% The \includegraphics command is provided (implemented) by the graphicx package
\includegraphics{universe} % 没用扩展名
 
There's a picture of a galaxy above.
\end{document}
```

图片和表格强制固定在当前位置？`\usepackage{float}` + 参数 `[H]`

### Captions, labels, references

图片的标记

```tex
\begin{figure}[h]
    \centering
    \includegraphics[width=0.75\textwidth]{mesh}
    \caption{A nice plot.}
    \label{fig:mesh1}
\end{figure}

As you can see in figure \ref{fig:mesh1}, the function grows near the origin. This example is on page \pageref{fig:mesh1}.
```

- `\label` 命令指定标签，为图像生成一个数字，利于下文的图片引用
- `\ref{fig：mesh1}` 会被引用图对应的数字替换

### List

以 `\begin{environment-name}` 开头，以 `\end{environment-name}` 结尾

Unsorted:

```tex
\begin{itemize}
  \item The individual entries are indicated with a black dot, a so-called bullet.
  \item The text in the entries may be of any length.
\end{itemize}
```

- 每条前有 `\item`

Ordered:

与无序列表语法相同，把 `itemize` 改成 `enumerate`

```tex
\begin{enumerate}
  \item balabala
\end{enumerate}
```

## 4 Math

### *inline* math

```tex
In physics, the mass-energy equivalence is stated 
by the equation $E=mc^2$, discovered in 1905 by Albert Einstein.
```

以下分隔符均可:

- `\( \)`
- `$ $`
- `\begin{math} ... \end{math}`

### *display* math

```tex
The mass-energy equivalence is described by the famous equation
\[ E=mc^2 \] discovered in 1905 by Albert Einstein. 

In natural units ($c = 1$), the formula expresses the identity
\begin{equation}
E=m
\end{equation}
```

- `\[ \]` 不带编号
- `\begin{equation}...\end{equation}` 带编号

### amsmath package

```tex
\usepackage{amsmath}% For the equation* environment
```

## 5 Basic Document Structure

### Abstract

```tex
\begin{document}
\begin{abstract}
This is a simple paragraph at the beginning of the 
document. A brief introduction about the main subject.
\end{abstract}
\end{document}
```

### Paragraphs & New lines

和 `markdown` 相似，1个enter并不会造成换行

2个enter会得到新段落

`\\` 或 `\newline` 会开启新行而不开启新段

### Chapters & sections

```tex
\documentclass{book}
\begin{document}

\chapter{First Chapter}

\section{Introduction}

This is the first section.

Lorem  ipsum  dolor  sit  amet,  consectetuer  adipiscing  
elit. Etiam  lobortisfacilisis sem.  Nullam nec mi et 
neque pharetra sollicitudin.  Praesent imperdietmi nec ante. 
Donec ullamcorper, felis non sodales...

\section{Second Section}

Lorem ipsum dolor sit amet, consectetuer adipiscing elit.  
Etiam lobortis facilisissem.  Nullam nec mi et neque pharetra 
sollicitudin.  Praesent imperdiet mi necante...

\subsection{First Subsection}
Praesent imperdietmi nec ante. Donec ullamcorper, felis non sodales...

\section*{Unnumbered Section}
Lorem ipsum dolor sit amet, consectetuer adipiscing elit.  
Etiam lobortis facilisissem...
\end{document}
```

![LL30Fig16-plain](./images/LL30Fig16-plain.svg)

- `\part{part}` (只在 `report` 和 `book` 类中使用)
- `\chapter{First Chapter}`
- `\section{Introduction}`
- `\subsection{...}`
- `\subsubsection{...}`
- `\paragraph{paragraph}`
- `\subparagraph{subparagraph}`

ps. `\section*{...}` 命令加 `*` 来禁用编号

## 6 Catalogue

用 `\tableofcontents`

```tex
\begin{document}
  
\maketitle
  
\tableofcontents

\section{Introduction}
...
```

## 7 参考文献

创建 `reference.bib` 文档

在序中添加

```tex
\usepackage[style=numeric,backend=biber,sorting=none]{biblatex}
\addbibresource{reference.bib}
```

在 Google Scholar 中找好文献后以 `BibTex` 方式引用，把文献信息复制到 `reference.bib` 中

再在原文 `\cite{article}`

另外，发现了 [scribbr](https://www.scribbr.com)，Citing $\rightarrow$ Citation Generator，输入参考文献的名字，即可自动转换格式

## 8 附录

`\hspace*{\parindent}` 手动缩进

`\noindent` 取消
