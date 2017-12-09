title: Java 学习总结一 多态
date: 2015-12-01 21:36:07
tags: [多态, 抽象类, 接口]
---

## 多态

### 多态

多态(polymorphism)是面向对象的核心思想之一，多态就是说在调用类的方法的时候，由运行时的类型来决定调用的是哪个方法，也可称作后期绑定，动态绑定。
在java中，除了 static 和 final 修饰的方法都是动态绑定的(等同于C++中使用virtual修饰)，可以很方便的实现多态.
通过多态，我们可以将做什么和怎么做进行解耦，使得代码的扩展能力更强
如以下的代码，我们可以看到TestClass中的draw函数接受了一个Shape类型的引用，在实际运行时，只要是Shape的派生类都能通过参数传递，然后调用时会根据s 的类型来决定是调用Circle的draw方法还是Square的draw方法
如果我们新增了一个类Rectangle继承于Shape,那么draw函数是不需要做更改就能处理Rectangle这个类的

```java
public class Shape {
    public void draw(){
        System.out.println("drawing an shape");
    }
}

public class Circle extends Shape {

    @Override
    public void draw() {
        System.out.println("drawing a circle");
    }
}

public class Square extends Shape {

    @Override
    public void draw() {
        System.out.println("drawing an square");
    }
}

public class TestClass {
    public static void main(String[] args) {
        Shape s1 = new Square();
        Shape s2 = new Circle();
        draw(s1);
        draw(s2);
    }

    public static void draw(Shape s) {
        s.draw();
    }
}

```

### 不要覆盖私有方法

由于对于继承，如果基类中的方法定义为private， 那么会被自动认为是final的，并且对导出类是不可见的，也就是说如果在导出类中写了同样的方法，那么将是一个全新的方法，并不是对基类的覆盖

```java
public class PrivateOverride {
    private void fun() {
        System.out.println("private fun() in PrivateOveride");
    }
    public static void main(String[] args) {
        PrivateOverride po = new Derived();
        po.fun();
    }
}

class Derived extends PrivateOverride{
    public void fun() {
        System.out.println("public fun() in Derived");
    }
}
```
对于以上代码，会输出

    private fun() in PrivateOveride

基类中的方法并没有被覆盖，调用时前期绑定，直接调用了PrivateOverride类中的fun方法

### 不要覆盖域和静态方法

与上面提到的一样，对于域和静态方法，也是不具有多态性的，要想拿到父类的域或者方法可以通过 super 来调用

### 向下转型与运行时类型识别

在进行向上转型时候，由于基类不会具有大于导出类的接口，因此向上转型是安全的，如果我们要进行向下转型，那么如果类的运行时类型是Circle，我们想要将它转型成一个Square，显然是不合理的，在Java中，我们所有的转型都会进行类型检查，因此当我们进行向下转型的时候，如果类型是不符的，将会抛出一个ClassCastException
在运行期间对类型进行检查的行为称作“运行时类型识别”（RTTI， Run Time Type Identification ）

```java
public class TestClass {
    public static void main(String[] args) {
        Shape s1 = new Square();
        Square s2 = (Square) s1;
        Circle s3 = (Circle) s1;
    }

}

```
在运行以上代码的时候我们可以发现 Circle s3 = (Circle) s1; 这一句会抛出异常

    Exception in thread "main" java.lang.ClassCastException: shape.Square cannot be cast to shape.Circle
	at TestClass.main(TestClass.java:12)
    

## 接口

接口是用来将实现和接口进行分离的方法，在Jva中，从语言层面对接口进行了支持

### 抽象类

抽象类是在普通类和接口之间的一种中庸之道， 在很多的继承体系中，基类中的方法往往是不需要被实现的，只是为了为为其导出的类提供一个统一的接口，此时我们可以将方法申明为抽象方法，通过以下方式来进行申明

```java
abstract void f();
```

包含了抽象方法的类就被成为抽象类。如果一个类包含了一个或多个抽象方法，那么此类必须被限定为抽象类，对于抽象类，我们无法从该类实例化一个对象

```java
public abstract class Instrument {
    public abstract void play();
}

public class Wind extends Instrument{
    public void play() {
        System.out.println("Wind play");
    }
}

public class Percussion extends Instrument{
    public void play() {
        System.out.println("Percussion play");
    }
}

```

当我们试图实例化一个抽象类的时候，编译器会报错
```java

public class TestAbstractClass {
    public static void main(String[] args) {
       Instrument ins = new Instrument();
       }
    }
}

```
以上代码是无法通过编译的

### 接口

在 java 中， 通过 interface 关键字来支持接口。通过接口，进行了更为高级的抽象。接口产生了一个完全抽象的类，在类中确定了方法名，参数列表和返回类型，但是没有任何实体，接口用来建立类与类之间的协议
通过以下代码来创建一个接口
```java
interface  Instrument {
    void play();
    void what();
    void adjust();
}
```
我们可以看到，方法的申明和接口的申明并未用public修饰，因为对于接口而言，不加public表示接口和方法默认是public的，此时在试验一个接口的时候，就需要将方法定义为public的，否则的话会导致访问权限被降低，这在java中是不允许的
```java
class Wind implements Instrument {
    private void play(){}
    private void what(){}
    private void adjust(){}
}
```

    Error:(7, 18) java: interface_sample.Wind中的play()无法实现interface_sample.Instrument中的play()
    正在尝试分配更低的访问权限; 以前为public
    
我们可以看到以上代码是无法通过编译的，需要将方法申明为public

我们可以发现，在使用的时候，接口和抽象类是类似的，都可以利用他们的向上转型来实现多态，可以通过接口或者抽象类或者普通的基类来将一组不同类型的对象组织在容器里，使用的时候并没有什么区别
从语言层面上来看，我们可以对接口和抽象类做以下对比

| 参数	| 抽象类	| 接口 |
| ----    | ------  | ---- |
|默认的方法实现	| 它可以有默认的方法实现	| 接口完全是抽象的。它根本不存在方法的实现|
|实现	|子类使用extends关键字来继承抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明的方法的实现。	|子类使用关键字implements来实现接口。它需要提供接口中所有声明的方法的实现|
|构造器|	抽象类可以有构造器|	接口不能有构造器|
|与正常Java类的区别	|除了你不能实例化抽象类之外，它和普通Java类没有任何区别	|接口是完全不同的类型|
|访问修饰符	|抽象方法可以有public、protected和default这些修饰符	|接口方法默认修饰符是public。你不可以使用其它修饰符。|
|main方法	|抽象方法可以有main方法并且我们可以运行它	|接口没有main方法，因此我们不能运行它。|
|多继承	|抽象方法可以继承一个类和实现多个接口	|接口只可以继承一个或多个其它接口|
|速度	|它比接口速度要快	|接口是稍微有点慢的，因为它需要时间去寻找在类中实现的方法。|
|添加新方法|	如果你往抽象类中添加新的方法，你可以给它提供默认的实现。因此你不需要改变你现在的代码。	|如果你往接口中添加方法，那么你必须改变实现该接口的类。|

不过从语法上来看，这些区别除了多继承外，其实差别并不算大，接口和抽象类的主要区别还是在设计上面
对于抽象类，我们强调的是它是由一组类的公共部分抽象而来，是满足is a 这种原则的, 比如对于汽车，马车，飞机  它们都是交通工具，因此我们可以抽象出一个交通工具的类出来，汽车 is a 交通工具, 马车 is a 交通工具
而对于接口呢，我们强调的是遵循的一个公共接口，比如说对于汽车, 轮船，飞机，他们都能用来装人，那么我们可以抽象出一个装人的接口出来(初略理解)
举个简单的例子，以前看algorithm4的时候还不懂java，所以基本只看算法了，对于代码都是略过的，里面提到过一点接口的内容，如以下代码，sort接收一组Comparable[]:

```java
public class Quick{
    public static void sort(Comparable[] a)
    {
    }
}

public class Test{
    public static void main(Sting[] agrs) {
        Quick quick = new Quick();
        SomeClass[] objs = new SomeClass[100]; 
        quick.sort(objs);
    }
}
```
在这个代码中，Comparable就是一种接口，这种接口的实现都实现了less这个方法，因此可以用来比较大小，在调用的时候就可以将一组对象传入sort这个方法中，在这儿并不强调SomeClass is a Comparable

### Java 中的多重继承

从语言上来讲，Java 并不支持 public class Derive extends Base1, base2 {} ,但是却可以实现多个接口的，因此可以靠接口来完成多重继承这个概念，如以下代码

```java

public interface CanFight {
    void fight();
}

public interface CanFly {
    void fly();
}

public interface CanSwim {
    void swim();
}

public class AcationCharater {
    public void fight(){}
}

public class Hero extends AcationCharater
implements CanFight, CanSwim, CanFly{
    public void fly() {}
    public void swim() {}
}
``` 

值得注意的是，Hero从AcationCharater继承而来的fight方法也成为了接口的实现，编译器并未因为写 Hero 类时并未给出fight方法而报错

就thinging in java 推荐来看，如果我们要创建不带任何方法定义和成员变量的基类，我们应该选用接口，或者说是在我们知道某事物会成为一个基类，第一选择应该是让它成为一个接口

### 通过继承来扩展接口 

通过继承，可以很容易的在接口中添加新的方法，因为接口支持多继承，因此可以通过接口来组合多个接口

    [修饰符] interface 接口名 [extends 父接口1, 父接口2 ... ] {
        
    }
    
    
### 适配接口

同一个接口，可以允许有多个不同的实现，其中一个体现就是一个接收接口类型的方法，该接口的实现和向该方法传递的对象取决于方法的使用者。

一个常见常见的用法就是策略模式，我们编写一个执行某些操作的方法，该方法接收一个我们指定的接口。使用者只需要实现这个接口，那么就可以使用这个方法，通过这种原则，可以使得方法更加灵活，通用和更强的可复用性

举个简单的例子，还是用排序算法来举例,只要使用者的类实现了Comparable，那么久可以将sort作用于它

```java
public class Quick{
    public static void sort(Comparable[] a)
    {
        //will call a.less()
    }
}

public interface Comparable {
    bool less();
}
```

### 接口中的域

值得注意的是，接口中的域都是 static 和 final的，可以被非常量表达式初始化
