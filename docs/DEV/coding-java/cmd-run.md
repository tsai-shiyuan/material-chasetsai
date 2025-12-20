# 命令行编译运行 Java 项目

参考 [命令行编译运行Java项目](https://shaojunying.github.io/6644e3ad18df41f990932bcf62294e82.html#https://www.notion.so/e611cc8d5f9b487eb74f8c20f97581a3)

## 自己的发现

在未编译运行程序时，Project文件目录

![截屏2024-08-25 12.23.10](./Java_image/%E6%88%AA%E5%B1%8F2024-08-25%2012.23.10.png){width=70%}

当点击三角形运行，会发现多了out文件夹

![截屏2024-08-25 12.23.54](./Java_image/%E6%88%AA%E5%B1%8F2024-08-25%2012.23.54.png)

![截屏2024-08-25 12.26.38](./Java_image/%E6%88%AA%E5%B1%8F2024-08-25%2012.26.38.png){width=70%}

## 命令行运行程序

### 将src下的Java文件编译到out文件夹下

创建out目录

```bash
mkdir out
```

执行javac命令

```bash
#-d 制定了class文件的保存路径，这里保存在了out目录下
#src/**/*.java 表示编译src下的所有Java文件
javac -d out src/**/*.java	
```

编译后的项目目录为

```bash
Max
-- out
------ Main.class
------ Max.class
-- src
------ Main.java
------ Max.java
```

### 运行out下的class文件

由于默认情况下java命令只会将当前目录添加为classpath，所以有两种方式可以执行class文件

**Way 1. 在out目录下运行**

```bash
cd out
java Max
```

**Way 2. 在项目根目录下运行**

需要将out目录添加到classpath下

```bash
java -cp out Main #这里的-cp命令将out目录添加到了classpath中
```

但有点奇怪，输出的是Main的内容