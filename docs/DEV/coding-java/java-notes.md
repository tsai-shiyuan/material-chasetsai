# Java学习笔记

## Chapter 0  前言

### 为什么学Java？

OOpre开课啦！课程用的语言是Java。久仰Java大名，借此机会，一睹真容。

### 怎么学Java？

Google出了 [二哥的Java进阶之路](https://javabetter.cn/overview/jdk-install-config.html#_01、jvm、jre、jdk-有什么关系) ，我觉得Java环境的配置讲得特别好，知识点目测很全，是下面的教材很好的补充。

csdiy指路 [MIT 6.092: Introduction To Programming In Java](https://ocw.mit.edu/courses/6-092-introduction-to-programming-in-java-january-iap-2010/pages/lecture-notes/) ，我看了课程补充资料，也是课程教材 [Think Java: How to Think Like a Computer Scientist](https://greenteapress.com/thinkjava6/thinkjava.pdf) ，深深地被吸引了（很喜欢，maybe bcs很像Malan讲课的目录?），所以系统学习Java主要靠这本教材。

学习到有些章节的时候，提到将来可能遇到的问题或句法，you might want to re-read it from time to time：

- Appendix [C](https://greenteapress.com/thinkjava6/html/thinkjava6018.html#debugappendix), we’ve collected some of our favorite debugging advice.
- [all of Java's packages](http://docs.oracle.com/javase/8/docs/api/)

### Java开发工具配置

#### JVM, JRE, JDK的关系

转载自[二哥的Java进阶之路](https://javabetter.cn/overview/jdk-install-config.html#_01、jvm、jre、jdk-有什么关系)：

- JDK（Java Development Kit）是用于开发 Java 应用程序的软件环境。里面包含运行时环境（JRE）和其他 Java 开发所需的工具，比如说解释器（java）、编译器（javac）、文档生成器（javadoc）等等。

- JRE（Java Runtime Environment）是用于运行 Java 应用程序的软件环境。也就是说，如果只想运行 Java 程序而不需要开发 Java 程序的话，只需要安装 JRE 就可以了。

- JVM (Java Virtual Machine) ，也就是 Java 虚拟机，由一套字节码指令集、一组寄存器、一个栈、一个垃圾回收堆和一个存储方法域等组成，屏蔽了不同操作系统（macOS、Windows、Linux）的差异性，使得 Java 能够“一次编译，到处运行”。

#### 安装 IntelliJ IDEA

IntelliJ IDEA 简称 IDEA，是业界公认为最好的 Java 集成开发工具，尤其是在代码自动提示、代码重构、代码版本管理、单元测试、代码分析等方面有着亮眼的发挥。

## Chapter 1  程序之道

### 程序

- **输入：**从键盘、文件、传感器或其他设备获取数据
- **输出：**在屏幕上显示数据，或将数据发送到文件或其他设备
- **数学：**执行加法和除法等基本数学运算
- **决定：**检查某些条件并执行适当的代码
- **重复：**重复执行某些动作，通常会有一些变化

### 编程语言

语言：

低级语言/机器语言
- 只能在一种计算机上运行

高级语言
- 运行前需要翻译为低级语言
- 可移植性，可以在不同类型的计算机上运行

将高级语言翻译成低级语言：

解释器
- 读取高级程序并执行它
- 每次处理一点程序，交替读取行和执行计算

![截屏2024-08-20 19.19.19](./Java_image/%E6%88%AA%E5%B1%8F2024-08-20%2019.19.19.png)

编译器
- 在程序开始运行之前会读取整个程序并将其完全翻译
- 高级程序称为**源代码**，而翻译后的程序称为**目标代码**或**可执行文件**

Java：

- 既是编译型语言又是解释型语言
- Java 编译器不会将程序直接翻译成机器语言，而是生成**字节码**，是可移植的
- 运行字节码的解释器称为“Java 虚拟机”（JVM）

![截屏2024-08-20 19.25.17](./Java_image/%E6%88%AA%E5%B1%8F2024-08-20%2019.25.17.png)

### Hello, world!

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, world!");		//print line, add '\n' by default
    }
}
```

Java 程序由 *class* and *method* 组成

- methods are made up of *statements*. A **statement** is a line of code that performs a basic operation

This program defines one method named `main`: 

```java
public static void main(String[] args)
```

A **class** is a collection of methods. This program defines a class named `HelloWorld` 

class's name:

- it is conventional to start with a capital letter
- The name of the class has to match the name of the file it is in, so this class has to be in a file named `HelloWorld.java`

`print()` 不会自动加换行

`""` 引用的为字符串

感觉Java和Python的类很相似

> 有关换行：
>
> `println()` 的换行是把光标移到下一行，如果之后有输出，会从光标所在的地方开始

### 转义符号 (Escape Sequences)

The backslash ( `\` ) allows you to “escape” the string’s literal interpretation.

### 代码规范 (Formatting Code)

Google publishes its Java coding standards for use in open-source projects: http://google.github.io/styleguide/javaguide.html

### Debugging

> start with a program that does *something* and make small modifications, debugging them as you go, until the program does what you want

## Chapter 2  Variables and operators

### Declaring variables

A variable is a named location that stores a **value**.

When you declare a variable, you create a named storage location.

```java
String message;
```

定义了名为 `message` 的变量，类型是 `String`

变量的命名：

- 体现变量的存储内容
- capitalize the first letter of each word except the first, like `String firstName;`

### Assignment (赋值)

```java
message = "Hello!";  // give message the value "Hello!"
```

### Printing variables

```java
String firstLine = "Hello, again!";
System.out.println(firstLine);
```

任何类型的变量都可通过 `println(<变量名>)` 来打印

注意：在许多电脑上，`print` 的输出结果是存起来的，直到遇到 `println` 才输出

### Arithmetic Operators

Java performs “integer division” when the operands are integers

### Floating-point Numbers

默认浮点类型为 `double`

当算式里有浮点数，Java就会执行 “floating-point division”

```java
double y = 1 / 3;
```

- 先算整数除法，得 `0` ，再转换成浮点数 `0.0`

### Operators for strings

`+` 连接两个字符串

`+` 可以连接数字和字符串

```java
System.out.println(1 + 2 + "Hello");
// the output is 3Hello, 先算 1+2=3
System.out.println("Hello" + 1 + 2);
//the output is Hello12
```

### Types of errors

- compile-time errors:
  - violate the syntax rules of the Java language

- run-time errors:
  - 也叫做 `exceptions`

- logic errors:
  - 得不到期望的输出

## Chapter 3  Input and output

### The System class

`System` is a **class** that provides methods related to the “system” or environment where programs run. It also provides `System.out`, which is a special value that provides methods for displaying output, including `println`.

```java
System.out.println(System.out);
```

返回:

```java
java.io.PrintStream@685d72cd
```

- `System.out` 是一个 ` PrintStream`，定义在叫作 `java.io` 的 **package** 里

- A **package** is a collection of related classes; `java.io` contains classes for “I/O” (input and output)

- `@` 后的数字是 `System.out` 的**地址**(十六进制)

![截屏2024-08-22 10.24.37](./Java_image/%E6%88%AA%E5%B1%8F2024-08-22%2010.24.37.png)

`System` is defined in a file called System.java, and `PrintStream` is defined in PrintStream.java. These files are part of the Java **library**, which is an extensive collection of classes you can use in your programs.

### The Scanner class

The `System` class also provides the special value `System.in`, which is an `InputStream` that provides methods for reading input from the keyboard. 但是这些methods不是很好利用

Java提供了其他classes来处理input--Scanner class是其中之一：

- `Scanner` 由 `java.util` package 提供，使用前需要 `import java.util.Scanner`
- **import statement** 告诉编译器，当我们使用 `Scanner` 的时候指的是 `java.util` 内的

create a `Scanner`:

```java
Scanner in = new Scanner(System.in);
```

- declares a `Scanner` variable named `in` and creates a new `Scanner` that takes input from `System.in`

`Scanner` provides a method called `nextLine` that reads a line of input from the keyboard and returns a `String`:

```java
import java.util.Scanner;
public class Echo {
    public static void main(String[] args) {
        String line;
        Scanner in = new Scanner(System.in);

        System.out.print("Type something: ");
        line = in.nextLine();
        System.out.println("You said: " + line);
    }
}
```

- `nextInt` 可以读入整数

System class:

- 属于 `java.lang` package, 会自动import
- `java.lang` “provides classes that are fundamental to the design of the Java programming language.” The `String` class is also part of the `java.lang` package

### Program structure

![截屏2024-08-22 11.05.29](./Java_image/%E6%88%AA%E5%B1%8F2024-08-22%2011.05.29.png)

> a package is a collection of classes, which define methods. Methods contain statements, some of which contain expressions. Expressions are made up of **tokens**, which are the basic elements of a program, including numbers, variable names, operators, keywords, and punctuation like parentheses, braces and semicolons.

### Literals and constants

最好是把数字存进变量，便于修改和阅读

`final` 表示常量，之后不能修改它的值。常量通常全大写，用 `_` 连接

```java
final double CM_PER_INCH = 2.54;
```

### Formatting output

`System.out` 提供另一个 method -- `printf`，来控制输出的格式

```java
System.out.printf("Four thirds = %.3f", 4.0 / 3.0);
```

返回：

```java
Four thirds = 1.333		// 没有换行
```

![截屏2024-08-22 13.29.02](./Java_image/%E6%88%AA%E5%B1%8F2024-08-22%2013.29.02.png)

注意保留1位小数要 `%.1f`

### Type cast (类型转换)

同C

```java
double pi = 3.14159;
int x = (int)pi;		//舍去小数部分
```

type cast 的优先级高于算术运算

### The Scanner bug

```java
System.out.print("What is your age? ");
age = in.nextInt();
System.out.print("What is your name? ");
name = in.nextLine();
System.out.printf("Hello %s, age %d\n", name, age);
```

- 输入完 age 为 `19` 后换行，会立即输出 `What is your name? Hello , age 19`

原理：

- Scanner gets a “stream of characters”

![截屏2024-08-22 14.23.54](./Java_image/%E6%88%AA%E5%B1%8F2024-08-22%2014.23.54.png)

- Line 2 `nextInt` 读取数字，直到非数字的时候停止，返回 `19` 

![截屏2024-08-22 14.24.36](./Java_image/%E6%88%AA%E5%B1%8F2024-08-22%2014.24.36.png)

- 然后 `nextLine` 读一行，直到换行符 (吸收回车)，但是当前的符号已经是换行，所以直接返回空 `""` 

解决方案：

```java
System.out.print("What is your age? ");
age = in.nextInt();
in.nextLine();  // read the newline
System.out.print("What is your name? ");
name = in.nextLine();
System.out.printf("Hello %s, age %d\n", name, age);
```

## Chapter 4  Void methods

### Math methods

`Math` is in the `java.lang` package, 不用自己import

```java
double root = Math.sqrt(1.70);
double angle = 1.5;
double height = Math.sin(angle);		//三角函数的参数用弧度制
double degrees = 90;
double angle = degrees / 180.0 * Math.PI;		//constant double PI
double x = Math.exp(Math.log(10.0));		//log的底数为e
```

- `PI` 是variable，所以没有 `()`
- `e` 用 `Math.E`

Java提供角度和弧度的转换method

```java
double radians = Math.toRadians(180.0);
double degrees = Math.toDegrees(Math.PI);
```

近似method--round，近似到最近的整数，long型

```java
long x = Math.round(Math.PI * 20.0);
```

### Adding new methods

```java
public class NewLine {

    public static void newLine() {
        System.out.println();
    }

    public static void main(String[] args) {
        System.out.println("First line.");
        newLine();
        System.out.println("Second line.");
    }
}
```

- The name of the class is `NewLine` (通常以大写字母开头), `NewLine` contains two methods, `newLine` and `main`
- `public` 意味着 `main` 和 `newLine` 可以被其他class引用
- `main` 有一个parameter，叫作 `args` ，类型是 `String[]` (an array of strings)
- Line 9 因为 `newLine` 和 `main` 在同一个class内，所以不需要写class name

### Flow of execution

从 `main` 开始执行

Tip: 我们在读代码的时候，按照程序的执行顺序读

### Parameters and arguments

When you use a method, you provide the arguments (实参). When you write a method, you name the parameters (形参).

传递的参数的类型要一致

Java有时会自动进行类型转换，比如 `Math.sqrt(25)` ，`25` 会转换为 `25.0`

### Reading documentation

术语比较难懂，我们可以关注例子，在本地运行一下

### Writing documentation

在代码中写documentation后，可以使用 **Javadoc** 工具生成HTML网页

规范：以 `/**` 开始，以 `*/` 结束，它们之间的就会被视为documentation

```java
/**
 * Example program that demonstrates print vs println.
 */
public class Goodbye {

    /**
     * Prints a greeting.
     */
    public static void main(String[] args) {
        System.out.print("Goodbye, ");  // note the space
        System.out.println("cruel world");
    }
}
```

## Chapter 5  Conditionals and logic

### Relational operators

```java
x == y          // x is equal to y
x != y          // x is not equal to y
x > y           // x is greater than y
x < y           // x is less than y
x >= y          // x is greater than or equal to y
x <= y          // x is less than or equal to y
```

表达式返回 `true` 或 `false`，属于boolean类型

`==` 和 `!=` 可以作用于String，但并不表示相等关系 (见Chapter 9)

判断String是否相等使用 `<String1>.equals(<String2>)`

### Logical operators

`&&` , `||` , `!` $\iff$ *and* , *or* , *not*

**short circuit** evaluation：在明确表达式结果后不再进行其他部分的计算

### Conditional statements

`if` statement:

```java
if (x > 0) {
  	System.out.println("x is positive");
}
```

`if-else` statement:

```java
if (x % 2 == 0) {
  	System.out.println("x is even");
} else {
  	System.out.println("x is odd");
}
```

只有一行可以省略 `{}` ，但不推荐:

```java
if (x % 2 == 0)
	System.out.println("x is even");
else
	System.out.println("x is odd");
```

`else if`:

```java
if (x > 0) {
    System.out.println("x is positive");
} else if (x < 0) {
    System.out.println("x is negative");
} else {
    System.out.println("x is zero");
}
```

### Flag variables

我们可以用boolean variable存储true/false，例如 `boolean flag = true;`

### Return statement

```java
public static void printLog(double x) {
    if (x <= 0.0) {
        System.err.println("Error: x must be positive");
        return;
    }
    double result = Math.log(x);
    System.out.println("The log of x is " + result);
}
```

`return` 退出当前进程

`System.err` 是用于error message和warning的OutputStream

### Validating input

在使用 `in.nextInt` 等进行读入时，我们可以先判断这个位置是否有 `int` 型，防止后续传递参数出现错误

```java
public static void scanDouble(Scanner in) {
    System.out.print("Enter a double: ");
    if (!in.hasNextDouble()) {
        String word  = in.next();
        System.err.println(word + " is not a number");
        return;
    }
    double x = in.nextDouble();
    printLog(x);
}
```

### Recursive methods (递归)

例子：输出n行空白

```java
public static void nLines(int n) {
    if (n > 0) {
        System.out.println();
        nLines(n-1);
    }
}
```

递归一定要写基本情况！

递归的执行：

1. 判断基本条件
2. 输出
3. 递归

（其中23可换序）

### Binary numbers

例子：十进制转二进制

```java
public static void displayBinary(int value) {
    if (value > 0) {
        displayBinary(value / 2);
        System.out.print(value % 2);
    }
}
```

递归很方便实现顺序或倒序，只需要修改递归语句和输出语句的顺序

## Chapter 6  Value methods

### Return values

和C差不多，注意有条件判断的话要把所有条件及返回值都考虑到

### Writing methods

**incremental development** (循序渐进地写代码)：

- Start with a working program and make **small**, incremental changes. At any point, if there is an error, you will know where to look.
- Use variables to **hold intermediate values** so you can check them, either with print statements or by using a debugger.
- Once the program is working, you can **consolidate** multiple statements into compound expressions (but only if it does not make the program more difficult to read). (IDEA的黄色波浪线提示可以压缩)

```java
public static double distance(double x1, double y1, double x2, double y2) {
    double dx = x2 - x1;
    double dy = y2 - y1;
    return Math.sqrt(dx * dx + dy * dy);
}
public static double calculateArea(double radius) {
    return Math.PI * radius * radius;
}
public static double circleArea(double xc, double yc, double xp, double yp) {
    return calculateArea(distance(xc, yc, xp, yp));
}
```

### Overloading

上例的calculateArea method和circleArea method做的是同样的事，即算面积，只不过接收的参数不同。我们通常会给这两个method取同样的名字，这种现象叫overloading

### Javadoc tags

为了把documentation归类，可以用 `@<tag>` 来提示，例如 `@param` `@return`

```java
/**
 * Tests whether x is a single digit integer.
 *
 * @param x the integer to test
 * @return true if x has one digit, false otherwise
 */
public static boolean isSingleDigit(int x) {
```

![截屏2024-08-24 15.25.37](./Java_image/%E6%88%AA%E5%B1%8F2024-08-24%2015.25.37.png)

## Chapter 7  Loops

### The while statement

```java
while (<条件>) {
    //body
}
```

### Encapsulation and generalization

The steps are:

1. Write a few lines of code in `main` or another method, and test them.
2. When they are working, wrap them in a **new method**, and test again.
3. If it’s appropriate, replace literal values with variables and **parameters**.(具体的数字换成概括的n)

eg.

```java
int i = 1;
while (i <= 6) {
    System.out.printf("%4d", i * 2);
    i = i + 1;
}
System.out.println();
```

- 输出 `   2   4   6   8  10  12`

- 我们先封装成一个新method (encapsulate) 

```java
public static void printRow() {
    int i = 1;
    while (i <= 6) {
        System.out.printf("%4d", i * 2);
        i = i + 1;
    }
    System.out.println();
}
```

- 现在把 `2` 换成 `n` (general, less specific)

### The for statement

```java
for(i = 1; i <= rows; i = i + 1) {
    //body
}
```

Java同样支持 `i++` , `i += 2`

### The do-while loop

run the body of the loop at least once

### Break and continue

同C

## Chapter 8  Arrays

### Creating arrays

为了创建数组，需要先声明一个有 array type 的变量，array type 比其他类型多了 `[]`

```java
int[] counts;	//counts is an “integer array”
double[] values;	//values is a “double array”
int[] a = {1, 2, 3, 4};
```

创建数组用 `new` 运算符

```java
counts = new int[10];
values = new double[size];	
```

### Accessing elements

![截屏2024-08-25 09.14.12](./Java_image/%E6%88%AA%E5%B1%8F2024-08-25%2009.14.12.png)

- `int` 数组中的元素会初始化为 `0`

- `counts` 指向数组 (存的是地址)，可以改变它的指向
- `[]` 访问数组元素

```java
System.out.println("The zero element is " + counts[0]);
```

### Displaying arrays

逐个输出：

```java
public static void printArray(int[] a) {
    System.out.print("{" + a[0]);
    for (int i = 1; i < a.length; i++) {
        System.out.print(", " + a[i]);
    }
    System.out.println("}");
}
```

转换成字符输出：

Java提供 java.util.Arrays class ，有数组 method，其中 `toString` 返回数组的字符串形式

```java
import java.util.Arrays;
//...
System.out.println(Arrays.toString(counts));
```

输出 `[1, 2, 3, 4]`

### Copying arrays

数组变量的赋值语句只会改变指向，并不会拷贝数组本身

```java
double[] a = new double[3];
double[] b = a;
```

![截屏2024-08-25 09.30.38](./Java_image/%E6%88%AA%E5%B1%8F2024-08-25%2009.30.38.png)

如果想拷贝数组得到新的？可以逐个拷贝或使用 java.util.Arrays 中的 copyOf method

```java
double[] b = Arrays.copyOf(a, 3);	//3是想拷贝的元素个数
```

### Array length

`a.length` 得到数组 `a` 的元素个数

### Random numbers

生成随机数：

```java
import java.util.Random;
public class GuessStarter {
    public static void main(String[] args) {
        // pick a random number
        Random random = new Random();
        int number = random.nextInt(100) + 1;
        System.out.println(number);
    }
}
```

`random.nextInt(n)` 生成位于 `0` 到 `n-1` 之间的随机数

生成无法预测的序列：

```java
public static int[] randomArray(int size) {
    Random random = new Random();
    int[] a = new int[size];
    for (int i = 0; i < a.length; i++) {
        a[i] = random.nextInt(100);
    }
    return a;
}
```

### The enhanced for loop

```java
for (int value : values) {
    System.out.println(value);
}
```

“for each `value` in `values` ”

通常数组变量名用复数

## Chapter 9  Strings and things

> an **object** is a collection of data that provides a set of methods.
>
> For example, `Scanner` is an object that provides methods for parsing input. `System.out` and `System.in` are also objects.
>
> Strings are objects, too. They contain characters and provide methods for manipulating character data.
>
> Not everything in Java is an object: `int`, `double`, and `boolean` are so-called **primitive** types.

### Characters

Strings提供 `charAt` method，接收index，返回位置上的字符

```java
String fruit = "banana";
char letter = fruit.charAt(0);
```

`char` 用单引号，例如：`'a'` ，同C

### Strings are immutable (不可改变)

字符串是不可改变的，作用在字符串的方法会返回新字符串

```java
String name = "Alan Turing";
String upperName = name.toUpperCase();
```

```java
String text = "Computer Science is fun!";
text = text.replace("Computer Science", "CS");	//CS is fun!
```

### String traversal

```java
for (int i = 0; i < fruit.length(); i++) {
    System.out.println(fruit.charAt(i));
}
```

Strings提供 `length` method，返回字符个数。因为是method，所以使用时要加 `()`

The enhanced for loop不适用于String，但可以把String转换为 `Char[]` 数组

```java
for (char letter : fruit.toCharArray()) {
    System.out.println(letter);
}
```

逆转字符串：

```java
public static String reverse(String s) {
    String r = "";
    for (int i = s.length() - 1; i >= 0; i--) {
        r = r + s.charAt(i);	//追加
    }
    return r;
}
```

### Substrings

`substring` method (不同参数有不同效果):

返回从index开始到结尾的子串

- `fruit.substring(0)` returns `"banana"`

- `fruit.substring(2)` returns `"nana"`

返回两个index之间的子串（不包括后一个index）
- `fruit.substring(0, 3)` returns `"ban"`

### The indexOf method

在字符串中找一个字符第一次出现的位置

```java
String fruit = "banana";
int index = fruit.indexOf('a');
```

找某次出现的位置

```java
int index = fruit.indexOf('a', 2);	//返回3
```

不存在返回 `-1` 

可以找特定字符串，例如 `fruit.indexOf("nan")` returns `2`.

### String comparison

```java
String name1 = "Alan Turing";
String name2 = "Ada Lovelace";
if (name1 == name2) {                 // wrong!
    System.out.println("The names are the same.");
}
```

运行结果大多是对的。但是！the `==` operator checks whether the two variables refer to the same object. If you give it two different strings that contain the same letters, it yields `false`.

正确的做法是用 `equals` method，返回 true or false:

```java
if (name1.equals(name2)) {
    System.out.println("The names are the same.");
}
```

`compareTo` method看字典序:

```java
int diff = name1.compareTo(name2);
if (diff == 0) {
    System.out.println("The names are the same.");
} else if (diff < 0) {
    System.out.println("name1 comes before name2.");
} else if (diff > 0) {
    System.out.println("name1 comes after name2.");
}
```

### String formatting

24-hour to 12-hour format:

```java
public static String timeString(int hour, int minute) {
    String ampm;
    if (hour < 12) {
        ampm = "AM";
        if (hour == 0) {
            hour = 12;  //midnight
        }
    } else {
        ampm = "PM";
        hour = hour - 12;
    }
    return String.format("%02d:%02d %s", hour, minute, ampm);
}
```

`System.out.printf` displays the result on the screen; `String.format` creates a new string, but does not display anything.

### Wrapper classes

Primitive values (like `int`s, `double`s, and `char`s) do not provide methods. 即不能使用 `<int variable>.<method>` 

But for each primitive type, there is a corresponding class in the Java library, called a **wrapper class**. 包括 `Character` `Integer` `Boolean` `Long` `Double` ，它们在 java.lang package 内，无需import

共同点：

- 都定义了常量 `MAX_VALUE` 和 `MIN_VALUE`

- 都能把字符串转换成相应类型，例如字符串转int：

  ```java
  String str = "12345";
  int num = Integer.parseInt(str);
  ```

- 都能转换成字符串

  ```java
  int num = 12345;
  String str = Integer.toString(num);
  ```

### Command-line arguments

回到 `public static void main(String[] args)`

```java
public class Max {
    public static void main(String[] args) {
        System.out.println(Arrays.toString(args));
    }
}
```

input: `java Max 10 -3 55 0 14`

output: `[10, -3, 55, 0, 14]`

## Chapter 10  Objects

> an object is a collection of data that provides a set of methods. For example, a `String` is a collection of characters that provides methods like `charAt` and `substring`.

### Point objects

`java.awt` package 提供了 `Point` class，可以表示笛卡尔坐标系中的坐标

import:

```java
import java.awt.Point;
```

create a new point:

```java
Point blank;
blank = new Point(3, 4);
```

line 1 表明 `blank` 有 `Point` 类型

`new` 的返回指向 new object 的 reference (指针?)

 ![截屏2024-08-26 09.30.27](./Java_image/%E6%88%AA%E5%B1%8F2024-08-26%2009.30.27.png)

### Attributes

Variables that belong to an object are usually called **attributes/fields/variables**

取 object 中的 variable 用 `.` (很像结构体)

```java
int x = blank.x;
```

- “go to the object `blank` refers to, and get the value of the attribute `x`.”

### Objects as parameters

```java
public static void printPoint(Point p) {
    System.out.println("(" + p.x + ", " + p.y + ")");
}
```

如果引用 `System.out.println(blank);` ，会返回 `java.awt.Point[x=3,y=4]` ，因为 `Point` object 提供了 `toString` method，当我们 call `println()` 时，会自动 call `toString` 

### Objects as return types

`java.awt` package 还提供了 `Rectangle` class，`Rectangle` objects 和 `Point` objects 很像，但有4个 attributes: x, y, width, height

import:

```java
import java.awt.Rectangle;
```

create a `Rectangle` object:

```java
Rectangle box = new Rectangle(0, 0, 100, 200);
```

同样可以 `System.out.println(box);` , return `java.awt.Rectangle[x=0,y=0,width=100,height=200]`

我们可以写返回值是 object 的 method:

```java
public static Point findCenter(Rectangle box) {
    int x = box.x + box.width / 2;
    int y = box.y + box.height / 2;
    return new Point(x, y);		//create a new Point object and returns a reference to it
}
```

Java提供移动矩形的method:

```java
box.translate(50, 100);
```

Java提供扩大矩形的method:

```java
box.grow(50, 100);	//左右各宽50，上下各高100
```

### Aliasing (同义词)

```java
Rectangle box1 = new Rectangle(0, 0, 100, 200);
Rectangle box2 = box1;
```

![截屏2024-08-26 19.17.43](./Java_image/%E6%88%AA%E5%B1%8F2024-08-26%2019.17.43.png)

`box1` 和 `box2` 的指向相同，所以显然，修改 `box1.x` 也会使 `box2.x` 改变

### The Null keyword

the keyword `null` is a special value that means “no object”

### Garbage collection

```java
Point blank = new Point(3, 4);
blank = null;
```

相当于我们丢掉了指针与内存的箭头，内存再也无法被我们访问。If there are no references to an object, there is no way to access its attributes or invoke a method on it.

![截屏2024-08-26 19.26.23](./Java_image/%E6%88%AA%E5%B1%8F2024-08-26%2019.26.23.png)

不同于C，Java有回收类似无法被访问的空间的机制，不用我们free

### Class diagrams

**Unified Modeling Language** (UML) defines a standard way to summarize the design of a class.

![截屏2024-08-26 19.30.06](./Java_image/%E6%88%AA%E5%B1%8F2024-08-26%2019.30.06.png)

一个 **class diagram** 被分成 attributes 和 methods

## Chapter 11  Classes

Whenever you define a new class, you also create a new type with the same name.(定义了新类型)

### The Time class

Attributes are also called **instance variables**，先在class的开头声明instance variables

```java
public class Time {
    private int hour;
    private int minute;
    private double second;
}
```

- `Time` class is `public` : it can be used in other classes

- instance variables are `private` : they can only be accessed from inside the `Time` class，实现 **information hiding**

### Constructors

ps. `⌘ + N` / `alt + insert`可以给到应该定义的方法的提示

下一步定义constructor，是用于初始化instance variables的特殊的method，特殊在于：

- The name of the constructor is the same as the name of the class
- Constructors have no return type (and no return value)
- The keyword `static` is omitted

```java
public Time() {		//无参数
    this.hour = 0;
    this.minute = 0;
    this.second = 0.0;
}
```

- `this` is a keyword that refers to the object we are creating

- `this` 和variable的使用规则相同，但是不用声明变量，也不能赋值操作

To create a Time object, we must use the `new` operator:

```java
Time time = new Time();
```

- When you invoke `new`, Java creates the object and calls your constructor to initialize the instance variables
- `new` returns a reference to the new object

### More constructors

constructors can be overloaded，根据参数判断用哪个constructor

```java
public Time(int hour, int minute, double second) {
    this.hour = hour;
    this.minute = minute;
    this.second = second;
}
```

创建Time object: `Time time = new Time(11, 59, 59.9)`

### Getters and setters

a class that uses objects defined in another class is called a **client**:

```java
public class TimeClient {
    public static void main(String[] args) {
        Time time = new Time(11, 59, 59.9);
        System.out.println(time.hour);      // compile error
    }
}
```

`hour has private access in Time` 的解决方法:

1 `public`

2 provide methods to access the instance variables, eg:

```java
public int getHour() {
    return this.hour;
}
```

- 这种method被称为accessors/**getters**

想让 `TimeClient` 修改变量:

```java
public void setHour(int hour) {
    this.hour = hour;
}
```

- 这种method被称为mutators/**setters**

### Displaying objects

直接 `System.out.println(time)` 返回十六进制的地址 `Time@6acbcfc0`

要想输出内容:

```java
public static void printTime(Time t) {
    System.out.print(t.hour);
    System.out.print(":");
    System.out.print(t.minute);
    System.out.print(":");
    System.out.println(t.second);
}
```

more concisely:

```java
System.out.printf("%02d:%02d:%04.1f", t.hour, t.minute, t.second);
```

`%04.1f` : total width 4, one digit after the decimal point, leading zeros if necessary

自己写的:

```java
public static void printTime() {
    System.out.println(this.hour);
}
```

报错: 'Time. this' cannot be referenced from a static context

### The toString method

Every object type has a method called `toString` that returns a string representation of the object. When you display an object using `print` or `println`, Java invokes the object’s `toString` method.

```java
public String toString() {
    return String.format("%02d:%02d:%04.1f\n", this.hour, this.minute, this.second);
}
```

注意没有 `static`，因为这不是 `static method`，而是 `instance method`。因为 when you invoke it, you invoke it on an instance of the class ( `Time` in this case) (在类的实例上调用)

现在 `System.out.println(time);` 会输出 `11:59:59.9`

### The equals method

`==` vs `equals` :

- 当比较基本类型 (String不在其中) 时，是在比较变量保存的数据是否相等

- The `==` operator checks whether objects are identical; that is, whether they are the same object.
- The `equals` method checks whether they are **equivalent**; that is, whether they have the same value.

默认的 `equals` method 和 `==` 效果一样，我们需要自己定义 `equals` 来覆盖:

```java
public boolean equals(Time that) {
    return this.hour == that.hour
            && this.minute == that.minute
            && this.second == that.second;
}
```

`equals` is an instance method, so it uses `this` to refer to the current object and it doesn’t have the keyword `static`

例子: `time1.equals(time3)`

### Adding times

Q：如何把两段时间加起来？(更好的add需要进位)

- We could write a static method that takes the two `Time` objects as parameters.
- We could write an instance method that gets invoked on one object and takes the other as a parameter.

static method:

```java
public static Time add(Time t1, Time t2) {
    Time sum = new Time();
    sum.hour = t1.hour + t2.hour;
    sum.minute = t1.minute + t2.minute;
    sum.second = t1.second + t2.second;
    return sum;
}
```

instance method:

```java
public Time add(Time t2) {
    Time sum = new Time();
    sum.hour = this.hour + t2.hour;
    sum.minute = this.minute + t2.minute;
    sum.second = this.second + t2.second;
    return sum;
}
```

instance method相较于static method

- 没有 `static`
- 用 `this` 替换 `t1`

### Pure methods and modifiers

我们可以改变自身的时间:

```java
public void increment(double seconds) {
    this.second += seconds;
    while (this.second >= 60.0) {
        this.second -= 60.0;
        this.minute += 1;
    }
    while (this.minute >= 60) {
        this.minute -= 60;
        this.hour += 1;
    }
}
```

methods like `add` are called **pure** bcs:

- They don’t modify the parameters.
- They don’t have any other “side effects”, like printing.
- The return value only depends on the parameters, not on any other state.

`increment` 打破第一条，被叫作 **modifiers**，常常是 void methods，有时返回指向更改过的变量的指针

==本章完整代码见附录[➡️](https://tsai-shiyuan.github.io/material-chasetsai/Dev/编程/Java/Java笔记/?h=instance+method#a-codes-of-chapter11-classes)==

---

## Chapter 12  Arrays of objects

接下来的几章会完成一个 card 游戏: *Crazy Eights*

英文名词:

A standard 52-card French-suited deck comprises 13 **ranks** in each of the four **suits**: clubs (♣), diamonds (♦), hearts (♥) and spades (♠)

Each suit includes three court cards (face cards): King,  Queen and Jack

Each suit also includes ten numeral cards or pip cards, from one (Ace) to ten

### Card objects

a mapping for suits:

![截屏2024-09-02 12.54.40](./Java_image/%E6%88%AA%E5%B1%8F2024-09-02%2012.54.40.png)



for face cards:

![截屏2024-09-02 12.55.40](./Java_image/%E6%88%AA%E5%B1%8F2024-09-02%2012.55.40.png)

用数字给每张卡编号，包括 rank (号码) 和suit (花纹)

```java
public class Card {
    //we can access them from inside this class, but not from other classes
    private int rank;	
    private int suit;

    public Card(int rank, int suit) {
        this.rank = rank;
        this.suit = suit;
    }
}
```

### Card toString

我们要把 Card objects 的信息展示为易读的，需要将数字映射到单词

先把数字suit映射到单词:

```java
String[] suits = new String[4];
suits[0] = "Clubs";
suits[1] = "Diamonds";
suits[2] = "Hearts";
suits[3] = "Spades";
```

也可以写作:

```java
String[] suits = {"Clubs", "Diamonds", "Hearts", "Spades"};
```

![截屏2024-09-02 15.56.29](./Java_image/%E6%88%AA%E5%B1%8F2024-09-02%2015.56.29.png)

Each element of the array is a reference to a `String`

再将rank映射到单词，最终:

```java
public String toString() {
    String[] suits = {"Clubs", "Diamonds", "Hearts", "Spades"};
    String[] ranks = {null, "Ace", "2", "3", "4", "5", "6", "7", "8", "9", "10", "Jack", "Queen", "King"};
    String s = ranks[this.rank] + "of" + suits[this.suit];
    return s;
}
```

### Class variables

和 instance variables 很像，class variables 会在 methods 前定义，但它们有关键字 `static`

They are created when the program begins (or when the class is used for the first time) and survive until the program ends. Class variables are *shared* across all instances of the class.

```java
public static final String[] RANKS = {
        null, "Ace", "2", "3", "4", "5", "6", "7",
        "8", "9", "10", "Jack", "Queen", "King"};
public static final String[] SUITS = {
        "Clubs", "Diamonds", "Hearts", "Spades"};

// instance variables and constructors go here

public String toString() {
    return RANKS[this.rank] + " of " + SUITS[this.suit];
}
```

Class variables are often used to store constant values that are needed in several places. In that case, they should also be defined as `final`. 

`static` means the variable is shared, and `final` means the variable is constant.

`static final` 变量名通常全部大写

> 一般的instance variable的引用需要 `this.<name>` ，而有static关键字的class variable被共享，可以直接用(就像method内的local variables)

为什么定义 RANKS 和 SUITS？这样它们就不会在引用 `toString` 时被定义与回收

为什么public？因为已经 `final` ，不会有隐患

### The compareTo method

要使卡片可以比较，我们规定先看suits (Clubs最小)，再看ranks。this大时返回1

```java
public int compareTo(Card that) {
    if (this.suit > that.suit) {
        return 1;
    }
    if (this.suit < that.suit) {
        return -1;
    }
    if (this.rank > that.rank) {
        return 1;
    }
    if (this.rank < that.rank) {
        return -1;
    }
    return 0;
}
```

### Cards are immutable

instance variables are `private`，所以我们用 getter 来让使其他 class 获取变量的值

```java
public int getRank() {
    return this.rank;
}
public int getSuit() {
    return this.suit;
}
```

本游戏中不需要也不能更改卡片的属性，所以:

- 不设置任何 modifier
- 加 `final` : `private final int rank;`

### Arrays of cards

The following statement creates an array of 52 cards:

```java
Card[] cards = new Card[52];
```

![截屏2024-09-02 16.32.53](./Java_image/%E6%88%AA%E5%B1%8F2024-09-02%2016.32.53.png)

数组里的每个元素都是"指针"，refer to the object，一开始都是 `null`，现在让object定居:

```java
int index = 0;
for (int suit = 0; suit <= 3; suit++) {
    for (int rank = 1; rank <=13; rank++) {
        cards[index] = new Card(rank, suit);
        index++;
    }
}
```

creating cards:

![截屏2024-09-02 16.37.24](./Java_image/%E6%88%AA%E5%B1%8F2024-09-02%2016.37.24.png)

display contents:

```java
public static void printDeck(Card[] cards) {
    for (int i = 0; i < cards.length; i++) {
        System.out.println(cards[i]);
    }
}
```

`cards` 是 `Card[]` 类型，所以 `cards` 内的元素是 `Card` 类型，所以 `println` 会引用 `card` class 内的 `toString` method

### Sequential search

接收一个数组和一个 `Card` object，返回对象在数组中的位置

```java
public static int search (Card[] cards, Card target) {
    for (int i = 0; i < cards.length; i++) {
        if (cards[i].equals(target)) {
            return i;
        }
    }
    return -1;
}
```

### Binary search

```java
public static int binarySearch (Card[] cards, Card target) {
    int low = 0;
    int high = cards.length - 1;
    while (low <= high) {
        int mid = (low + high) / 2;
        int comp = cards[mid].compareTo(target);
        if (comp == 0) {
            return mid;
        } else if (comp > 0) {
            high = mid - 1;
        } else {
            low = mid + 1;
        }
    }
    return -1;
}
```

### Recursive version

```java
public static int binarySearch (Card[] cards, Card target, int low, int high) {
    if (low > high) {
        return -1;
    }

    int mid = (low + high) / 2;
    int comp = cards[mid].compareTo(target);
    if (comp == 0) {
        return mid;
    } else if (comp > 0) {
        return binarySearch(cards, target, low, mid - 1);
    } else {
        return binarySearch(cards, target, mid + 1, high);
    }
}
```

## Chapter 13  Objects of arrays

### The Deck class

```java
private Card[] cards;
public Deck(int n) {
    // initializes the instance variable with an array of n cards
    // doen't create any objects
    cards = new Card[n];	
}
```

![截屏2024-09-03 07.39.56](./Java_image/%E6%88%AA%E5%B1%8F2024-09-03%2007.39.56.png)

add a second constructor that makes a standard 52-card deck and populates it with `Card` objects:

```java
public Deck() {
    this.cards = new Card[52];
    int index = 0;
    for (int suit = 0; suit < 4; suit++) {
        for (int rank = 1; rank <= 13; rank++) {
            cards[index] = new Card(rank, suit);
            index++;
        }
    }
}	
```

```java
public void print() {
    for (int i = 0; i < this.cards.length; i++) {
        System.out.println(this.cards[i]);
    }
}
```

### Shuffling decks

shuffle: put the cards in a random order

human ways: dividing the deck in two halves and then choosing alternately from each one, after about seven iterations the order of the deck is pretty well randomized

computer: traverse the deck one card at a time, and at each iteration choose two cards and swap them

```java
for each index i {
    // choose a random number between i and length - 1
    // swap the ith card and the randomly-chosen card
}
```

我们需要helper methods:

- 生成random
- 换位

### Selection sort

## More...

Chapter 14  Objects of objects 

Chapter 15  Development tools

Chapter 16  Java 2D graphics

Chapter 17  Debugging

## Appendix

### A. codes of Chapter11 (classes)

```java
public class Time {
    // instance variables
    private int hour;
    private int minute;
    private double second;

    // constructors
    public Time() {
        this.hour = 0;
        this.minute = 0;
        this.second = 0.0;
    }
    public Time(int hour, int minute, double second) {
        this.hour = hour;
        this.minute = minute;
        this.second = second;
    }

    // getters
    public int getHour() {
        return this.hour;
    }
    public int getMinute() {
        return this.minute;
    }
    public double getSecond() {
        return this.second;
    }

    // setters
    public void setHour(int hour) {
        this.hour = hour;
    }
    public void setMinute(int minute) {
        this.minute = minute;
    }
    public void setSecond(double second) {
        this.second = second;
    }

    // self print
    public static void printTime(Time t) {
        System.out.printf("%02d:%02d:%04.1f",t.hour, t.minute, t.second);
    }

    // toString
    public String toString() {
        return String.format("%02d:%02d:%04.1f\n", this.hour, this.minute, this.second);
    }

    // equals
    public boolean equals(Time that) {
        return this.hour == that.hour
                && this.minute == that.minute
                && this.second == that.second;
    }

    // add
    public static Time add(Time t1, Time t2) {
        Time sum = new Time();
        sum.hour = t1.hour + t2.hour;
        sum.minute = t1.minute + t2.minute;
        sum.second = t1.second + t2.second;
        return sum;
    }
    public Time add(Time t2) {
        Time sum = new Time();
        sum.hour = this.hour + t2.hour;
        sum.minute = this.minute + t2.minute;
        sum.second = this.second + t2.second;

        if (sum.second >= 60.0) {
            sum.second -= 60.0;
            sum.minute += 1;
        }
        if (sum.minute >= 60) {
            sum.minute -= 60;
            sum.hour +=1;
        }
        return sum;
    }

    // increment
    public void increment(double seconds) {
        this.second += seconds;
        while (this.second >= 60.0) {
            this.second -= 60.0;
            this.minute += 1;
        }
        while (this.minute >= 60) {
            this.minute -= 60;
            this.hour += 1;
        }
    }
}
```

