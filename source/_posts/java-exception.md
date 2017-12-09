title: Java 学习总结二  异常
date: 2015-12-05 22:10:59
tags: [异常, exception]
---

### 异常的基本概念

为什么需要异常呢？ 在程序运行中，总是会出错的，错误的处理一直以来都是各个程序语言需要处理的问题，在C语言中，错误一般是通过返回值和全局错误标识来进行表示的，这是一种约定俗称的方法，这样的方法比较原始，在比较现代的语言,例如Python， PHP, C++, Java 都支持通过异常来处理错误
通过异常，我们可以在错误出现时，实行合适的操作来处理错误

#### 抛出异常

抛出异常通过关键字 throw 来实现

```java
	if(t == null)
		throw new NullPointerException()
```

在Java中，异常和其他对象一样，通过new 在堆上创建。标准异常类有两个构造器:一个是默认构造器，另外一个是以字符串作为参数，可以将相关信息放入异常对象中

当抛出异常时，函数会从方法或者作用域中退出(类似于 return的效果)

#### 异常捕获

通过try...catch...finally 来进行异常的捕获和处理

```java
package exception_sample;

import java.util.InputMismatchException;
import java.util.Scanner;
public class TestClass {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        try {
            System.out.println("input a integer");
            int one = input.nextInt();
        } catch (InputMismatchException e) {
            System.out.println("input doesn't matche, pliease input ainteger");
        }
    }
}
```

#### 异常匹配

抛出异常的时候，异常处理系统会按照代码书写的顺序找出最近的处理程序，找到匹配的处理程序之后，便会认为异常已经得到处理，不再继续查找，查找的时候不要求完全匹配，派生类的对象也可以匹配其积累的处理程序，因此在写catch的时候要将派生类写在前面，基类写在后面

```java
class Annoyance extends Exception{

}

class Sneeze extends Annoyance{

}

public class TestClass {
    public static void main(String[] args) {
        try{
            throw new Sneeze();
        } catch (Sneeze s) {
            System.out.println("Caught Sneeze");
        } catch (Annoyance a) {
            System.out.println("Caught Annoyance");
        }
        try{
            throw new Sneeze();
        } catch (Annoyance a) {
            System.out.println("Caught Annoyance");
        }
    }
}

```

#### 经验总结

1. 处理运行时异常时候，采用逻辑去合理规避同时辅助try-catch处理
2. 在多重catch块后面，可以加一个catche(Exception) 来处理可能会被遗漏的异常
3. 对于不确定的代码，也可以加上try-catch，处理潜在的异常
4. 尽量去处理异常， 不要只是简单调用printStackTrace()去打印
5. 具体如何处理异常，要根据不同的业务需求和异常类型去决定
6. 尽量添加finally语句块去释放占用的资源
