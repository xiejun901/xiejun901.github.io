title: Java学习总结三 字符串
date: 2015-12-6 23:44:06
tags: [string, java]
---

#### 不可变String

String对象是不可改变的，String类中看起来能改变字符串内容的方法实际上都是创建了一个新的String对象，原来的对象并未发生改变,如下代码

```java
public class TestClass {
    public static void main(String[] args) {
        String s1 = "howdy";
        System.out.println(s1);
        String s2 = s1.toUpperCase();
        System.out.println(s2);
        System.out.println(s1);
    }
}
```
输出为:

	howdy
	HOWDY
	howdy
    
#### StringBuilder

考虑到如下代码
```java
public class TestClass {
    public static void main(String[] args) {
        String s1 = "yeyeyeye";
        String s2 = "1" + s1 + "2" + "3";
    }
}

```
我们知道，String对象是不可变的，那么从代码上来看s2的创建过程中,"1"+s1首先会生成一个临时对象，然后再加上“2”会在生成一个临时对象，再加上“3”会生成最后的对象把引用赋给上s2，这过程中产生的临时对下你个是会对效率产生影响的
不过对于这种简单的语句，我们编译器是会自动采取优化的，会自动引入StringBuilder类来构造s2，这样就会省略掉这些中间对象
对以上代码字节码进行分析  javap -c TestClass

    Compiled from "TestClass.java"
    public class TestClass {
    public TestClass();
        Code:
        0: aload_0
        1: invokespecial #1                  // Method java/lang/Object."<init>":()V
        4: return
    
    public static void main(java.lang.String[]);
        Code:
        0: ldc           #2                  // String yeyeyeye
        2: astore_1
        3: new           #3                  // class java/lang/StringBuilder
        6: dup
        7: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
        10: ldc           #5                  // String 1
        12: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        15: aload_1
        16: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        19: ldc           #7                  // String 2
        21: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        24: ldc           #8                  // String 3
        26: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        29: invokevirtual #9                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        32: astore_2
        33: getstatic     #10                 // Field java/lang/System.out:Ljava/io/PrintStream;
        36: aload_2
        37: invokevirtual #11                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        40: return
    }

    
我们可以看到，生成的字节码中自动引入了StringBuilder类，使用它的append方法来添加新的内容，防止多次构造临时对象，再看看下面一段代码和其生成的字节码

```java
public class TestClass {
    public static void main(String[] args) {
    }

    public String implicit(String[] s)
    {
        String result = "";
        for(int i=0; i< s.length; i++) {
            result += s[i];
        }
        return result;
    }

    public String explicit(String[] s)
    {
        StringBuilder result= new StringBuilder();
        for(int i=0; i< s.length; i++) {
            result.append(s[i]);
        }
        return result.toString();
    }
}
```

    Compiled from "TestClass.java"
    public class TestClass {
    public TestClass();
        Code:
        0: aload_0
        1: invokespecial #1                  // Method java/lang/Object."<init>":()V
        4: return
    
    public static void main(java.lang.String[]);
        Code:
        0: return
    
    public java.lang.String implicit(java.lang.String[]);
        Code:
        0: ldc           #2                  // String
        2: astore_2
        3: iconst_0
        4: istore_3
        5: iload_3
        6: aload_1
        7: arraylength
        8: if_icmpge     38
        11: new           #3                  // class java/lang/StringBuilder
        14: dup
        15: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
        18: aload_2
        19: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        22: aload_1
        23: iload_3
        24: aaload
        25: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        28: invokevirtual #6                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        31: astore_2
        32: iinc          3, 1
        35: goto          5
        38: aload_2
        39: areturn
    
    public java.lang.String explicit(java.lang.String[]);
        Code:
        0: new           #3                  // class java/lang/StringBuilder
        3: dup
        4: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
        7: astore_2
        8: iconst_0
        9: istore_3
        10: iload_3
        11: aload_1
        12: arraylength
        13: if_icmpge     30
        16: aload_2
        17: aload_1
        18: iload_3
        19: aaload
        20: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        23: pop
        24: iinc          3, 1
        27: goto          10
        30: aload_2
        31: invokevirtual #6                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        34: areturn
    }

我们可以看到在implicit方法中，每次循环都会将StringBuilder变成String，在explicit方法中，最后再执行的StringBuilder到String的转换，显然显然，explicit方法效率会高很多，因此在我们使用String的时候需要注意这个问题，如果遇到循环，我们可以自己手动使用StringBuilder来优化，防止多次，无用的构造String临时对象
StringBuilder 是Java SE5 引入的，在这之前用的是StringBuffer, 后者是线程安全的，同时开销也会大一些

#### String 类提供的方法

##### 构造器

构造一个 String 对象

```java
String s1 = new String();
System.out.println(s1);
byte[] bytes = { 97, 98, 99, 100, 101};
String s2 = new String(bytes);
System.out.println(s2);
char[] chars = {'a', 'b', 'c', 'd', 'e'};
String s3 = new String(chars);
System.out.println(s3);
int[] codePoints = {97, 98, 99, 100, 101};
String s4 = new String(chars);
System.out.println(s4);
StringBuilder sb = new StringBuilder();
for (int i=1;i<6;i++) {
    sb.append(i);
}
String s5 = new String(sb);
System.out.println(s5);
StringBuffer sbuffer = new StringBuffer();
for (int i=6;i<11;i++) {
    sbuffer.append(i);
}
String s6 = new String(sbuffer);
System.out.println(s6);
```

##### length()

获取 String 对象的长度
```java
public int length();
```

```java
String s1 = "xyzwaaa";
System.out.println(s1.length()); // 将会输出 7
```

##### charAt()

获取指定位置的char
```java
public char charAt(int index);
```

```java
String s = "ABCDEFGHIJKLM";
char ch = s.charAt(4);  //char 为 'E'
```

##### getChars(), getBytes()

复制chars或者bytes到一个数组

```java
public void getChars(int srcBegin, int srcEnd, char[] dst, int destBegin);
public void getBytes(int srcBegin, int srcEnd, byte[] dst, int destBegin); 
```

```java
String s = "ABCDEFGHIJKLM";
char[] dest1 =new char[10];
s.getChars(0,5,dest1,0);  //dest1: {A, B, C, D, E, , , , ,}
byte[] dest2 = new byte[10];
s.getBytes(5,10,dest2,5); //dest2: {0, 0, 0, 0, 0, 70, 71, 72, 73, 74}
```

##### toCharArray()

转换为一个char[]的数组

```java
public char[] toCharArray()
```

```java
String s= "ABCDEF";
char[] chars = s.toCharArray(); // CHARS: {A, B, C, D, E, F}
```

##### equals(), equalsIgnoreCase()

检测是否相等(内容而不是引用)

```java
boolean equals(Object anObject); //比较和另外一个对象是否相 当且仅当参数不为null且为一个序列相等的String对象的时候返回true
boolean equalsIgnoreCase(String anotherString); //比较和另外一个string是否相等，忽略大小写
```

```java
char[] chars = {'A', 'b', 'C', 'd'};
String s1 = new String(chars);
String s2 = new String(chars);
String s3 = "ABCD";
System.out.println(s1 == s2); //false
System.out.println(s1.equals(s2));  // true
System.out.println(s1.equals(chars)); //false
System.out.println(s1.equals(s3)); //false
System.out.println(s1.equalsIgnoreCase(s3)); //true
```

##### compareTo(), compareToIgnoreCase()

与另外String进行比较，返回-1,0,1

```java
public int compareTo(String anotherString)
public int compareToIgnoreCase(String anotherString)
```

```java
String s1 = "abcde";
String s2 = "abcde";
String s3 = "bbcde";
System.out.println(s1.compareTo(s2)); //  0
System.out.println(s1.compareTo(s3)); // -1
System.out.println(s3.compareTo(s1)); // 1
```

#### 更多方法， 参考[String](http://docs.oracle.com/javase/7/docs/api/java/lang/String.html)

[oracle String 文档](http://docs.oracle.com/javase/7/docs/api/java/lang/String.html)

#### 格式化输出

##### printf()

Java也支持同C类似的printf

```java
System.out.printf("%d,%d,%c", 10,20,'A');
```

##### format()

fromat 和 printf是等价的，作用于PringStream或PrintWritter对象

```java
System.out.format("%d,%d,%c", 10,20,'A');
```

##### Formatter类

新的格式化功能由java.util.Formatter类处理。

```java
public static void main(String[] args) {
    Formatter f = new Formatter(System.out);
    String name = "xiejun";
    int age = 25;
    f.format("name:%s, age: %d",name, age);
}
```

##### 格式化说明符

其语法如下:

    %[argument_index$][flags][width][.precision]conversion
    
```java
public static void main(String[] args) {
    Formatter f = new Formatter(System.out);
    String name = "xiejun";
    int age = 25;
    double height = 1.75;
    f.format("name:%-15s, age: %5d, age: %7f\n",name, age, height);
    f.format("name:%15s, age: %5d, age: %7f\n",name, age, height);
    f.format("name:%2.2s, age: %5d, age: %1.4f\n",name, age, height);
}
```

    name:xiejun         , age:    25, age: 1.750000
    name:         xiejun, age:    25, age: 1.750000
    name:     xi, age:    25, age: 1.7500

width 应用于各种类型表示数据最小尺寸， precision用于控制最大尺寸，应用于String时，表示打印出的最大字符量，对浮点数时，表示小数部分要显示出来的位数，对整数使用则会触发异常

|      |   |
| ----  | --- |
| %d  |  十进制整形数字 |
| %c  |  Unicode字符|
|%b  | boolean值|
|%s |String|
|%f | 浮点，十进制|
|%e  |浮点，科学计数|
|%x|整数，十六进制|
|%h|hash code，十六进制|
|%%|字符%|

##### String.format()

类似于C中的sprintf，String.format() 接受与Formatter.fromat()相同的参数，但是返回一个String对象

#### 正则表达式

 正则表达式是一种很强大的文本处理工具，通过正则表示我们可以很方便的通过编程来处理字符串的模式匹配问题
 
 ##### 基础
 
 通过String.matches来使用正则表达式
 
 ```java
 public boolean matches(String regex)
 ```
 
 ```java
    public static void main(String[] args) {
    System.out.println("-12345".matches("-?\\d+")); //true
    System.out.println("5678".matches("-?\\d+")); //true
    System.out.println("+91011".matches("-?\\d+")); //false
    System.out.println("+911".matches("(-|\\+)?\\d+")); //true
}
 ```
 
    ? ：代表出现一次或多次
    + ：代表出现一次或者多次
    | ：代表并联
    \\ : 表示开始一个转义字符
    
通过String.split()方法也能使用正则表达式

```java
public String[] split(String regex, int limit)
```

```java
String s1 = "abcd,efgh;mnky!dsdy";
String[] ss = s1.split(",|;|!"); // : {abcd, efgh, mnky, dsdy}
```

##### 创建正则表达式

  更多正则表达式相关内容后面再补充吧