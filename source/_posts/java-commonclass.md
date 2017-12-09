title: Java 学习总结四 Java中的常用类
date: 2015-12-8 10:17:51
tags: [java]
---

#### Java 中的包装类

为了使Java中的基本数据类型，int, float, doubtl, boolean, char等表现得跟类一样，为每个基本类型提供了一个包装类(Wrapper type)

|基本类型|包装类型|
|----|----|
|byte|Byte|
|short|Short|
|int|Integer|
|long|Long|
|float|Float|
|double|Double|
|char|Character|
|boolean|Boolean|

包装类主要提供两大类方法

1. 将本类型和其他基本类型进行转换
2. 将字符串和本类型及包装类的相互转换


```java
public class HelloWorld {
    public static void main(String[] args) {
        
		// 定义int类型变量，值为86
		int score1 = 86; 
        
		// 创建Integer包装类对象，表示变量score1的值
		Integer score2=new Integer(score1);
        
		// 将Integer包装类转换为double类型
		double score3=score2.doubleValue();
        
		// 将Integer包装类转换为float类型
		float score4=score2.floatValue();
        
		// 将Integer包装类转换为int类型
		int score5 =score2.intValue();

		System.out.println("Integer包装类：" + score2);
		System.out.println("double类型：" + score3);
		System.out.println("float类型：" + score4);
		System.out.println("int类型：" + score5);
	}
}
```

##### 包装类和基本类型的相互转换

可以通过两种方式完成包装类和基本类型的换换

手动转换

	WrapperType wp = new WrapperType(someBuildInType)  
	BuildInType  t = someWrapperType.BuildInTypeValue()

自动转换

	WrapperType wp = someBuildInType;
	BuildInType t = someWrapperType
	
```java
public class HelloWorld {
    public static void main(String[] args) {
        
        // 定义double类型变量
		double a = 91.5;
        
         // 手动装箱
		Double b =  new Double(a);      
        
        // 自动装箱
		Double c =    a;   

        System.out.println("装箱后的结果为：" + b + "和" + c);
        
        // 定义一个Double包装类对象，值为8
		Double d = new Double(87.0);
        
        // 手动拆箱
		double e =     d.doubleValue()          ;
        
        // 自动拆箱
		double f =     d           ;
        
         System.out.println("拆箱后的结果为：" + e + "和" + f);
	}
}
```

##### Java中基本类型和字符串之间的转换

可以通过如下方法来进行基本类型到字符串之间的转换

1. 使用包装类toString() 方法
2. 使用String类的valueOf()方法
3. 使用一个空字符串加上基本类型

字符串转换为基本类型由以下方法

1. 调用包装类的parseType() 静态方法
2. 调用包装类的valueOf() 方法 转换为包装类，再由包装类自动转换为基本类型

```java
public class HelloWorld {
    public static void main(String[] args) {
        
		double m = 78.5;
		//将基本类型转换为字符串
		String str1 =  String.valueOf(m);
        String str2 =  Double.toString(m);
        String str3 = ""+m;
        
		System.out.println("m 转换为String型后与整数20的求和结果为： "+(str1+20));
		System.out.println("m 转换为String型后与整数20的求和结果为： "+(str2+20));
		System.out.println("m 转换为String型后与整数20的求和结果为： "+(str3+20));
		
		String str = "180.20";
	    // 将字符串转换为基本类型
		Double a1 =    Double.parseDouble(str);
        Double a2 = Double.valueOf(str);
	
		System.out.println("str 转换为double型后与整数20的求和结果为： "+(a2+20));
		System.out.println("str 转换为double型后与整数20的求和结果为： "+(a2+20));
	}
}
```

#### java中日期的表示

Date类位于  java.util包中，主要用于获取当时的时间

```java
import java.util.Date;
/**
 * Created by xiejun on 2015/12/12.
 */
public class TestClass {
    public static void main(String[] args) {
        Date date = new Date();
        System.out.println(date);
    }
}
```

SimpleDateFromat 类位于  java.text 包中，主要用于时间的格式化，其提供了format()方法用于时间的格式化，parse()方法用于从字符串转换为时间

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class TestClass {

    public static void main(String[] args) throws ParseException {

        // 使用format()方法将日期转换为指定格式的文本
        SimpleDateFormat sdf1 = new SimpleDateFormat("yyyy年MM月dd日 HH时mm分ss秒");
        SimpleDateFormat sdf2 = new SimpleDateFormat("yyyy/MM/dd HH:mm");
        SimpleDateFormat sdf3 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        // 创建Date对象，表示当前时间
        Date now = new Date();

        // 调用format()方法，将日期转换为字符串并输出
        System.out.println( sdf1.format(now)                         );
        System.out.println(sdf2.format(now));
        System.out.println(sdf3.format(now));

        // 使用parse()方法将文本转换为日期
        String d = "2014-6-1 21:05:36";
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        // 调用parse()方法，将字符串转换为日期
        Date date = sdf.parse(d);

        System.out.println(date);
    }
}
```

Calander 类

```java
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

public class HelloWorld {
    
    public static void main(String[] args) {
		// 创建Canlendar对象
		Calendar c = Calendar.getInstance();
        
		// 将Calendar对象转换为Date对象
		Date date = c.getTime();
        
		// 创建SimpleDateFormat对象，指定目标格式
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        
		// 将日期转换为指定格式的字符串
		String now = sdf.format(date);
		System.out.println("当前时间：" + now);
	}
}
```

#### Math 类

Math类位于java.lang 包中，包含了一些基本的数学运算方法

常用方法

 | 返回值| 方法名|说明|
 |----|-----|
 |long|round()|返回四舍五入后的整数|
 |double|floor()|向下取整|
 |double|cell()|向上取整|
 |double|random()|[0,1）之间的随机数|
 
更多方法参考  [oracle Math类参考文档](https://docs.oracle.com/javase/7/docs/api/java/lang/Math.html) 
 


