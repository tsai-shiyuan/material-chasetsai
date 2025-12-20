# 笔记｜设计模式

## 策略模式

[博客园｜策略模式](https://www.cnblogs.com/cosimo/p/18027303)

### 接口

问题: 根据不同国家的税率计算订单金额

思路: 创建接口 (概念上完成税额计算)

![截屏2024-10-17 16.54.29](./0img/%E6%88%AA%E5%B1%8F2024-10-17%2016.54.29.png)

---

问题背景: 设计一个销售各类书籍的电子商务网站的购物车系统。本网站对所有的高级会员提供每本20%的促销折扣；对中级会员提供每本10%的促销折扣；对初级会员没有折扣。

传统实现方式: `if...else...` ，但是当需求不断增多，我们就必须修改原码，违背开闭原则

### Strategy (抽象策略类)

为所支持的算法声明抽象方法，是所有策略类的父类。可以使抽象类或者具体类，也可以是接口。

```java
public interface MemberStrategy {
	// 计算价格的抽象方法
	// n: 商品个数
	public double calcuPrice (double price, int n);
}
```

### Concrete Strategy (具体策略类)

实现了抽象类中的方法

```java
// 普通会员——不打折
public class PrimaryMemberStrategy implements MemberStrategy {
	@Override
	public double calcuPrice(double price, int n) {
		return price * n;
	}
}

// 中级会员——打九折
public class IntermediateMemberStrategy implements MemberStrategy {
	@Override
	public double calcuPrice(double price, int n) {
		return price * n * 0.9;
	}
}

// 高级会员——打八折
public class AdvancedMemberStrategy implements MemberStrategy {
	@Override
	public double calcuPrice(double price, int n) {
		return price * n * 0.8;
	}
}
```

### Context (环境类)

负责和具体策略的交互，

```java
public class MemberContext {
	private MemberStrategy memberStrategy;
	
	public MemberContext (MemberStrategy memberStrategy) {
		this.memberStrategy = memberStrategy;
	}
	
	public double quotePrice (double price, int n) {
		return memberStrategy.calcuPrice(price, n);
	}
}
```

调用:

```java
public class Application {
	public static void main (String[] args) {
		MemberStrategy primaryMemberStrategy = new PrimaryMemberStrategy();
		MemberStrategy intermediateMemberStrategy = new IntermediateMemberStrategy();
		
		MemberContext primaryContext = new MemberContext(primaryMemberStrategy);
		MemberContext intermediateContext = new MemberContext(intermediateMemberStrategy);
		
		System.out.println("普通会员: " + primaryContext.quotePrice(300, 1));
        // 300
		System.out.println("中级会员: " + intermediateContext.quotePrice(300, 1));	
        // 270
	}
}
```

### 应用场景小结

系统中需要动态地在几种算法中选择一种

## 观察者模式

当一个对象的状态发生改变时（发生了所关注的事件），所有依赖于（或关注）它的对象都将得到通知

通知者&观察者: 让多个观察者对象同时监听主体对象，主体变化时，通知每一个观察者，使他们能自动更新

![截屏2024-10-17 17.01.58](./0img/%E6%88%AA%E5%B1%8F2024-10-17%2017.01.58.png)

![截屏2024-10-17 18.18.44](./0img/%E6%88%AA%E5%B1%8F2024-10-17%2018.18.44.png)

```java
abstract class Subject {
	private ArrayList<Observer> list = new ArrayList<Observer>();
	
	// 增加观察者
	public void attach(Observer observer) {
		list.add(observer);
	}
	// 减少观察者
	public void detach(Observer observer) {
		list.remove(observer);
	}
	// 通知观察者
	public void notify() {
		for (Observer item : list) {
			item.update();
		}
	}
	protected String subjectState;
	public String getSubjectState() {
		return this.subjectState;
	}
	public void setSubjectState(String value) {
		this.subjectState = value;
	}
}

// 抽象观察者接口
abstract class Observer {	// 也可以 interface
	public abstract void update();
}

// 具体通知者
class ConcreteSubject extends Subject {
	// 具体通知者方法
}

// 具体观察者
class ConcreteObserver extends Observer {
	private String name;
	private Subject sub;
	public ConcreteObserver(String name, Subject sub) {
		this.name = name;
		this.sub = sub;
	}
	public void update() {
		System.out.println("观察者" + this.name + "的新状态是" + this.sub.getSubjectState());
	}
}
```

## 单例模式

### 背景

背景: 菜单栏中的工具箱，我们希望要么不出现，要么出现同一个；在工具栏中点工具箱，也出现同一个？

可能的解答: 工具箱自己决定是否出现 $\Rightarrow$ 构造方法改为 private，外部程序不能 `new`，再写 `getInstance()` 的 public 方法，返回类的实例 $\Rightarrow$ 如果没有实例化过，就构造

```java
class Toolkit extends JFrame {
	private static Toolkit toolkit;		// 静态变量，工具箱
	private Toolkit(String title) {		// private 构造方法
		super(title)
	}
	public static Toolkit getInstance() {
		if (toolkit == null || !toolkit.isvisible()) {
			toolkit = new Toolkit ("工具箱");
			toolkit.setSize(150, 300);
			...
			toolkit.setVisible(true);
		}
		return toolkit;
	}
}
```

### 小结

```java
public class Singleton {
    private static Singleton singleton;		
    private Singleton(/*parameters*/) {
        /* ... */
    }
    public static Singleton getInstance() {	
        // 如果是第一次使用：
        if (singleton == null) {	// 懒汉式实例化
            singleton = new Singleton(/* paramters */);
        }
        return singleton;
    }
}    
```

line2: 也可以 `singleton = new Singleton();` 饿汉式实例化

## 简单工厂模式

背景: 设计了一个计算器，我们应该实例化哪些运算？需要用一个单独的类来进行创造实例的过程，即工厂

### 运算类

```java
// 加法类
public class Add extends Operation {
	public double getResult(double numA, double numB) {
		return numA + numB;
	}
}

// 减、乘、除...
public class Sub extends Operation {
	...
}
```

### 工厂类

```java
public class OperationFactory {
	public static Operation createOperate(String operate) {
		Operation oper = null;
		switch (operate) {
			case "+":
				oper = new Add();
				break;
			case "-":
				oper = new Sub();
				break;
			case "*":
				oper = new Mul();
				break;
			case "/":
				oper = new Div();
				break;
		}
		return oper;
	}
}
```

### 客户端

```java
Operation oper = OperationFactory.createOperate(strOperate);
double result = oper.getResult(numberA, numberB);
```

## 工厂模式

简单工厂模式是用一个厂生产多个产品，而工厂模式则是多个厂生产不同的产品

### 创建对象的接口

```java
public interface Factory {
    public Product create(int price);
}
```

### 子类决定实例化

```java
public class FactoryA implements Factory{
    @Override
    public Product create(int price) {
        return new ProductA(price);
    }
}
public class FactoryB implements Factory{
    @Override
    public Product create(int price) {
        return new ProductB(price);
    }
}
```

```java
public class MainClass {
    public static void main(String[] args) {
        Factory factoryA = new FactoryA();
        Factory factoryB = new FactoryB();
        factoryA.creat(100).use();
        factoryB.creat(200).use();
    }
}
```





## 抽象工厂模式

一个工厂可以生产多个类的产品

### 产品类

手机类产品：

```java
public abstract class Phone {
    private int price;
    public Phone(int price) {
        this.price = price;
    }
    public abstract void use();
}
public class iPhone extends Phone{
    public iPhone(int price) {
        super(price);
    }
    @Override
    public void use() {
        System.out.println("APPLE iPhone!!!");
    }
}
public class Honor extends Phone{
    public Honor(int price) {
        super(price);
    }
    @Override
    public void use() {
        System.out.println("HUAWEI Honor!!!");
    }
}
```

笔记本电脑类产品：

```java
public abstract class Laptop {
    private int size;
    public Laptop(int size) {
        this.size = size;
    }
    public abstract void use();
}
public class iMac extends Laptop{
    public iMac(int size) {
        super(size);
    }
    @Override
    public void use() {
        System.out.println("APPLE iMac!!");
    }
}
public class MateBook extends Laptop{
    public MateBook(int size) {
        super(size);
    }
    @Override
    public void use() {
        System.out.println("HUAWEI MateBook!!");
    }
}
```

### 工厂类

```java
public interface Factory {
    public Phone createPhone(int price);
    public Laptop createLaptop(int size);
}
public class BJFactory implements Factory {
    @Override
    public Phone createPhone(int price) {
        return new Honor(price);
    }
    @Override
    public Laptop createLaptop(int size) {
        return new MateBook(size);
    }
}
public class LAFactory implements Factory {
    @Override
    public Phone createPhone(int price) {
        return new iPhone(price);
    }
    @Override
    public Laptop createLaptop(int size) {
        return new iMac(size);
    }
}
```

### 客户端

```java
public class MainClass {
    public static void main(String[] args) {
        Factory Beijing = new BJFactory();
        Factory LosAngeles = new LAFactory();
        Beijing.creatPhone(8000).use();
        LosAngeles.creatPhone(11000).use();
        Beijing.creatLaptop(15).use();
        LosAngeles.creatLaptop(13).use();
    }
}
```

 