# 笔记｜递归下降法解析表达式

OOP Lecture 7 notes

## 文法

### 文法的概念

文法 (Grammer) 是语言的框架与规范

![截屏2024-10-24 19.18.47](./0img/%E6%88%AA%E5%B1%8F2024-10-24%2019.18.47.png)

面对一个给定的字符串时，应当如何准确判断它是否符合之前所定义的变量名规范？进一步，又该如何判定该字符串是否满足所描述的句子结构？需要进行词法和语法分析

### 词法分析 Lexer

把字符串按照文法符号中的最低层级非终结符来解析，得到 Token 序列。比如解析字符串，得到 `Var` `Num` `(` `)` `+` `*` 这样的序列

### 语法分析 Parser

词法分析后，我们要对 Token 序列进行语法分析

一个经典的文法示例:

![截屏2024-10-24 19.36.06](./0img/%E6%88%AA%E5%B1%8F2024-10-24%2019.36.06.png)

观察发现:

![截屏2024-10-24 19.36.11](./0img/%E6%88%AA%E5%B1%8F2024-10-24%2019.36.11.png)

用树可以更形象地描述表达式的结构:

![截屏2024-10-24 19.40.21](./0img/%E6%88%AA%E5%B1%8F2024-10-24%2019.40.21.png)

语法分析实质上就是根据<语法规则>，解析出<非终结符>，同时按照<非终结符>来创建<对象>，并按照文法所定义的<关系>构建一个<对象化语法树>

## 递归下降

![截屏2024-10-24 19.12.30](./0img/%E6%88%AA%E5%B1%8F2024-10-24%2019.12.30.png)

### 解析步骤

- 对**<**表达式**>**按照文法规则进行拆分
- 识别出构成表达式的多个顺序相连的**<**项**>**
- 对每个**<**项**>**按照文法规则进行拆分
- 识别出构成项的顺序相连的**<**因子**>**
- 构建成一棵自顶向下的语法树

Parser 类实现该流程

## 一个例子

<img src="./0img/%E6%88%AA%E5%B1%8F2024-10-31%2017.03.55.png" alt="截屏2024-10-31 17.03.55" style="zoom:50%;" />

![扫描全能王 2024-10-31_1](./0img/%E6%89%AB%E6%8F%8F%E5%85%A8%E8%83%BD%E7%8E%8B%202024-10-31_1.jpg)

### 实现 Lexer

建立 Token 类:

```java
public class Token {
    public enum Type {
        ADD, MUL, LPAREN, RPAREN, NUM, VAR
    }
    private final Type type;
    private final String content;
    public Token(Type type, String content) {
        this.type = type;
        this.content = content;
    }
    ...
}
```

Lexer 解析 Token:

```java
import java.util.ArrayList;
public class Lexer {
    private final ArrayList<Token> tokens = new ArrayList<>();
    private int index = 0;
    public Lexer(String input) {    // construct
        int pos = 0;
        while (pos < input.length()) {
            if (input.charAt(pos) == '(') {
                tokens.add(new Token(Token.Type.LPAREN, "("));
                pos++;
            } else if (input.charAt(pos) == ')') {
                tokens.add(new Token(Token.Type.RPAREN, ")"));
                pos++;
            } else if (input.charAt(pos) == '+') {
                tokens.add(new Token(Token.Type.ADD, "+"));
                pos++;
            } else if (input.charAt(pos) == '*') {
                tokens.add(new Token(Token.Type.MUL, "*"));
                pos++;
            } else if (input.charAt(pos) == 'x') {
                tokens.add(new Token(Token.Type.VAR, "x"));
                pos++;
            } else {
                char now = input.charAt(pos);
                StringBuilder sb = new StringBuilder();
                while (now >= '0' && now <= '9') {
                    sb.append(now);
                    pos++;
                    if (pos >= input.length()) {
                        break;
                    }
                    now = input.charAt(pos);
                }
                tokens.add(new Token(Token.Type.NUM, sb.toString()));
            }
        }
    }
    public Token getCurToken() {
        return tokens.get(index);
    }
    public void nextToken() {
        index++;
    }
    public boolean isEnd() {
        return index >= tokens.size();
    }
}
```

### 建立类

- `Num` `Var` 基本类

- `Expr` `Term` `SubExpr` `SubTerm` 类内含容器

`Term` 类示例:

```java
import java.util.ArrayList;

public class Term {
    private final ArrayList<Factor> factors = new ArrayList<>();
    public void addFactor(Factor factor) {
        factors.add(factor);
    }
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        for (Factor factor : factors) {
            if (factor instanceof SubExpr) {
                sb.append("(" + factor.toString() + ")");
            } else {
                sb.append(factor.toString());
            }
            sb.append("*");
        }
        return sb.substring(0, sb.length() - 1); // remove the last "*"
    }
    public void print() {
        System.out.println("Term " + this);
    }
}
```

![扫描全能王 2024-10-31_2](./0img/%E6%89%AB%E6%8F%8F%E5%85%A8%E8%83%BD%E7%8E%8B%202024-10-31_2.jpg)

### 实现 Parser

$\mathrm{Expr} \rightarrow \mathrm{Term} | \mathrm{Expr} + \mathrm{Term}$

如果我们这样解析:

```java
parseExpr() {
    parseExpr();
}
```

包含自我引用，可能导致无限递归，所以要改写规则:

$<\mathrm{Expr}> \rightarrow <\mathrm{Term}>\{+<\mathrm{Term}>\}$​

具体实现:

```java
public class Parser {
    private final Lexer lexer;
    public Parser(Lexer lexer) {
        this.lexer = lexer;
    }
    public Expr parseExpr() {	// parse返回一个表达式 (实例)
        Expr expr = new Expr();
        expr.addTerm(parseTerm());
        while (!lexer.isEnd() && lexer.getCurToken().getType() == Token.Type.ADD) {
            lexer.nextToken();
            expr.addTerm(parseTerm());
        }
        expr.print();
        return expr;
    }
    public Term parseTerm() {
        Term term = new Term();
        term.addFactor(parseFactor());
        while (!lexer.isEnd() && lexer.getCurToken().getType() == Token.Type.MUL) {
            lexer.nextToken();
            term.addFactor(parseFactor());
        }
        term.print();
        return term;
    }
    public Factor parseFactor() {
        Token token = lexer.getCurToken();
        if (token.getType() == Token.Type.NUM) {
            return parseNum();
        } else if (token.getType() == Token.Type.VAR) {
            return parseVar();
        } else {
            lexer.nextToken();
            Factor subExpr = parseSubExpr();
            lexer.nextToken();
            return subExpr;
        }
    }
    public SubExpr parseSubExpr() {
        SubExpr subExpression = new SubExpr();
        subExpression.addTerm(parseSubTerm());
        while (!lexer.isEnd() && lexer.getCurToken().getType() == Token.Type.ADD) {
            lexer.nextToken();
            subExpression.addTerm(parseSubTerm());
        }
        subExpression.print();
        return subExpression;
    }
    public SubTerm parseSubTerm() {
        SubTerm subTerm = new SubTerm();
        subTerm.addFactor(parseNum());
        while (!lexer.isEnd() && lexer.getCurToken().getType() == Token.Type.MUL) {
            lexer.nextToken();
            subTerm.addFactor(parseNum());
        }
        subTerm.print();
        return subTerm;
    }
    public Num parseNum() {
        Token token = lexer.getCurToken();
        lexer.nextToken();
        Num num = new Num(Integer.parseInt(token.getContent()));
        num.print();
        return num;
    }
    public Var parseVar() {
        Token token = lexer.getCurToken();
        lexer.nextToken();
        Var var = new Var(token.getContent());
        var.print();
        return var;
    }
}
```

parser 把 Token 字符串解析成有意义的类的实例

C 代码与 Java 的实现相近

---

Reference:

[博客园｜OO-表达式解析之递归下降法](https://www.cnblogs.com/dhy2000/p/15970225.html)
