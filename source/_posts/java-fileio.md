title: java 学习总结六 文件操作
date: 2015-12-11 20:51:24
tags: [file, java, io]
---

#### File 类

```java
import java.io.File;
import java.io.FilenameFilter;
import java.util.regex.Pattern;

public class TestClass {
    public static void main(String[] args) {
        File path = new File(".");
        String[] list;
        list  = path.list(new DirFilter());
        for(String str : list) {
            System.out.println(str);
        }
    }
}

class DirFilter implements FilenameFilter{
    private Pattern pattern;
    public DirFilter(){
        pattern = Pattern.compile(".*\\.java");
    }
    public boolean accept(File dir, String name) {
        return pattern.matcher(name).matches();
    }
}
```
这段代码匹展示出了列出当前目录下以.java结尾的文件功能   其中用到了File类和FilenameFilter接口


#### 目录的检查及创建

```java
import java.io.*;
public class MakeDirectories {
    public static void main(String[] args) {
        if(args.length < 1) {
            System.exit(1);
        } else if (args[0].equals("-r")) {
            File old = new File(args[1]), rname = new File(args[2]);
            old.renameTo(rname);
            return;
        }
        int count = 0;
        boolean del = false;
        if(args[0].equals("-d")) {
            count ++;
            del = true;
        }
        count -- ;
        while (++count < args.length) {
            File f = new File(args[count]);
            if(f.exists()) {
                System.out.println("正在删除文件" + f);
                f.delete();
            }
            else {
                if(!del) {
                    f.mkdirs();
                    System.out.println("创建文件" + f);
                }
            }
        }
    }
}

```
以上代码展示了文件的创建，重命名，删除操作

#### 输入和输出

编程语言的I/O库中通常使用流这个概念， 它代表任何由能力产出数据的数据源对象或者由能力接受数据的借手段对象。流 屏蔽了实际的I/0设备中处理数据的细节。
在Java中，任何从InputStream或Reader派生而来的类都含有名为()的基本方法，用于读取单个字节或者字节数组。任何从OutputStream或Writer派生而来的类都含有write()方法，用于写单个字节或者字节数组。

##### InputStream类型


|类 | 功能 | 说明 |
|--|---|----|
|ByteArrayInputStream|允许将内存的缓冲区当作InputStream使用|构造参数为缓冲区。字节将从中取出。将其于FilterInputStream对象相连以提供游泳的借口|
|StringBufferInputStream|将String转换为InputStream|构造参数为字符串。将其于FilterInputStream对象相连以提供游泳的借口|
|FileInputStream|用于从文件中读取信息|使用文件名，文件，或FileDescriptor进行构造。将其于FilterInputStream对象相连以提供游泳的借口|
|PipedInputStream|产生用于写入相关PipedOutputStream的数据。实现管道化概念|构造参数为PipedOutputStream。将其于FilterInputStream对象相连以提供游泳的借口|
|SequenceInputSteam|将两个或者多个InputStream对象转换为单一InputStream|构造参数为两个InputStream对象或者一个容纳InputStream对象的容器Enumeration。。将其于FilterInputStream对象相连以提供游泳的借口|
|FilterInputStream|抽象类，作为装饰器借口。其中装饰器为其他的InputStream类提供有用的功能||

##### OutputStream类型

|类 | 功能 | 说明 |
|--|---|----|
|ByteArrayOutputStream|内存中创建缓冲区，所有送往流的数据都要放置在此缓冲区|构造参数为缓冲区初始化尺寸(可选)。字节将从中取出。将其于FilterOutputStream对象相连以提供游泳的借口|
|FileOutputStream|用于将信息写入文件|使用文件名，文件，或FileDescriptor进行构造。将其于FilterOutputStream对象相连以提供游泳的借口|
|PipedOutputStream|任何写入其中的信息都会自动作为相关PipedInputStream的输出。实现管道化概念|构造参数为PipedInputStream，指定用于多线程的数据的目的地。将其于FilterOutputStream对象相连以提供游泳的借口|
|SequenceInputSteam|将两个或者多个InputStream对象转换为单一InputStream|构造参数为两个InputStream对象或者一个容纳InputStream对象的容器Enumeration。。将其于FilterInputStream对象相连以提供游泳的借口|
|FilterOutputStream|抽象类，作为装饰器借口。其中装饰器为其他的OutputStream类提供有用的功能||

##### FiterInputStream

|类|功能|构造器参数|如何使用|
|--|---|---|---|
|DataInputStream|与DataOutputStream搭配使用，因此我们可以按照可移植方式从流读取基本数据类型|InputStream|包含读取基本类型数据的全部接口|
|BufferedInputStram|使用它可以防止每次读取时都得进行实际操作。代表使用缓冲区|InputStream，可以指定缓冲区大小(可选的)|本质上不提供接口，只不过是向进程中添加缓冲区所必须的。与借口对象搭配|
|LineNumberInputStream|跟踪输入流中的行号，可以调用getLineNumber()和setLineNumber(int)|InputStream|仅增加了行号，因此可能要与借口对象配合使用|
|PushbackInputStream|具有 能"弹出一个字节的缓冲区"。因此可以将读到的最后一个字符回退|InputStream|通常为编译器的扫描器|


##### FiterOutputStream

|类|功能|构造器参数|如何使用|
|--|---|---|---|
|DataOutputStream|与DataInputputStream搭配使用，因此我们可以按照可移植方式从流读取基本数据类型|OutputStream|包含读取基本类型数据的全部接口|
|PrintStram|用于产生格式化输出。其中DataOutputStream处理数据的存储，PrintStream处理显示|OutputStream，可以指定缓冲区大小(可选的)|本质上不提供接口，只不过是向进程中添加缓冲区所必须的。与借口对象搭配|
|BufferedOutputStream|使用它可以避免每次发送数据时都要进行实际操作，可以调用flush()清空缓冲区|OutputStream|仅增加了行号，因此可能要与借口对象配合使用|


```java
import java.io.*;

/**
 * Created by xiejun on 2015/12/17.
 */
public class TestClass {
    public static void main(String[] args) throws Exception{
        //ByteArrayInputStream 测试
        testByteArrayInputStream();
        //FileInputStream 测试
        testFileInputStream();
        //StringBufferInputStream()
        testStringBufferInputStream();

    }

    /**
     * ByteArrayInputStream 类测试
     */
    public static void testByteArrayInputStream() {
        byte[] bytes = new byte[10];
        for(int i=0; i< bytes.length; i++) {
            bytes[i] = (byte)i;
        }
        ByteArrayInputStream byteStream = new ByteArrayInputStream(bytes);
        DataInputStream dis = new DataInputStream(byteStream);
        try {
            while (true) {
                byte b = dis.readByte();
                System.out.println(b);
            }
        } catch (EOFException e1) {
            System.out.println("ending");
        } catch (IOException e2) {
            System.out.println("IO error");
        }
    }

    /**
     * FileInputStream 测试
     */
    public static void testFileInputStream() {
        //构造参数为字符串的文件名
        try {
            FileInputStream fs = new FileInputStream(new File("data.txt"));
            DataInputStream dis = new DataInputStream(fs);
            while(true) {
                byte value = dis.readByte();
                System.out.println(value);
            }
        } catch (EOFException e) {
            System.out.println("read finish");
        }
        catch (IOException e) {
            e.printStackTrace();
        }
    }
    /**
     *StringBufferInputStream 测试
     */
    public static void testStringBufferInputStream() {
        try{
            StringBufferInputStream sbis = new StringBufferInputStream("123 456");
            DataInputStream dis = new DataInputStream(sbis);
            while(true) {
                byte value = dis.readByte();
                System.out.println(value);
            }
        } catch (EOFException e) {
            System.out.println("read finish");
        }
        catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

#### Reader 和 Writer

之前提到的I/O流只支持9位字节流，并不能很好的处理16位的Unicode字符。Reader和Writer是为了在所有的I/o操作中都支持Unicode

```java
import java.io.*;

/**
 * Created by xiejun on 2015/12/17.
 */
public class IoClass {
    public static void main(String[] args) throws Exception{
        testReadter();

    }

    /**
     * Reader 测试
     */
    public static void testReadter() {
        System.out.println("测试Reader");
        try {
            FileReader fr = new FileReader("data.txt");
            System.out.println(fr.getEncoding());
            char[] value = new char[100];
            fr.read(value);
            System.out.println(value);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

````

#### RandomAccessFile

RandomAccessFile 适用于由大小移植的记录组成的文件，所以我们可以使用()将记录从一处转移到另一处，然后读取或者修改记录， RandomAccessFile 不属于 InputStream 和 OutputStream继承体系。

```java
public static void testRandomAccessFile() {
    System.out.println("测试RandomAccessFile");
    try {
        RandomAccessFile raf = new RandomAccessFile("data.txt", "rw");
        byte[] value =  new byte[10];
        raf.seek(3);
        raf.read(value);
        for(byte b : value) {
            System.out.println(b);
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

#### 对象的序列化与反序列化

将对象从Object转换为byte序列称为对象的序列化，反之则是对象的反序列化
对象必须实现Serializable借口才能进行序列化，否则将出现异常

一个简单的例子

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

/**
 * Created by xiejun on 2015/12/18.
 */
public class TestClass {
    public static void main(String[] args) {
        try {
            ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("data.txt"));
            String s = "xxxx";
            oos.writeObject(s);
            ObjectInputStream ois = new ObjectInputStream(new FileInputStream("data.txt"));
            String s1;
            s1 = (String)ois.readObject();
            System.out.println(s1);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### Java 中的XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bookstore>
	<book id = "1">
		<name>冰与火之歌</name>
		<author>乔治马丁</author>
	</book>
	<book id = "2">
		<name>安徒生童话</name>
		<year>2004</year>
	</book>
</bookstore>
```

XML的作用是用于数据共享，用于不同应用程序之间，不同操作系统之间，不同平台间的数据共享。

##### 通过 DOM 来解析XML文件

在 Java 中，解析XML文件有以下集中方式

1. DOM
2. SAX
3. DOM4J
4. JDOM

其中 DOM 和 SAX 解析是 Java 官方提供的， DOM 和 JDOM 解析是第三方库提供的，以下为一小部分的例子

```java

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;

import org.w3c.dom.Document;
import org.w3c.dom.NamedNodeMap;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;

public class TestClass {
    public static void main(String[] args) {
        DocumentBuilderFactory xmlParser = DocumentBuilderFactory.newInstance();
        try {
            DocumentBuilder docBuilder =  xmlParser.newDocumentBuilder();
            Document doc = docBuilder.parse("bookstore.xml");
            NodeList bookList = doc.getElementsByTagName("book");
            System.out.println(bookList.getLength());
            for (int i=0; i<bookList.getLength(); i++) {
                Node book = bookList.item(i);
                NamedNodeMap attrs = book.getAttributes();
                for(int j=0; j<attrs.getLength();j++) {
                    System.out.println(attrs.item(j).getNodeName() + ":" + attrs.item(j).getNodeValue());
                }
                NodeList childNodes = book.getChildNodes();
                System.out.println(childNodes.getLength());
                for (int j=0; j < childNodes.getLength();j++) {
                    Node childNode = childNodes.item(j);
                    if (childNode.getNodeType() == Node.ELEMENT_NODE) {
                        System.out.println(childNode.getNodeName() );
                        System.out.println(childNode.getFirstChild().getNodeValue());
                    }
                }
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}


```

##### 通过 SAX 来解析XML文件

```java
import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;

public class SAXTest {
    public static void main(String[] args) {
        SAXParserFactory factory = SAXParserFactory.newInstance();
        try {
            SAXParser parser = factory.newSAXParser();
            SAXParserHandler hd = new SAXParserHandler();
            parser.parse("bookstore.xml",hd);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}


import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.helpers.DefaultHandler;


public class SAXParserHandler extends DefaultHandler{

    public int bookIndex = 0;
    @Override
    public void startDocument() throws SAXException {
        super.startDocument();
        System.out.println("开始解析");
    }

    @Override
    public void endDocument() throws SAXException {
        super.endDocument();
        System.out.println("结束解析");
    }

    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        super.startElement(uri, localName, qName, attributes);
        if (qName.equals("book")) {
            bookIndex++;
            //创建一个book对象
            System.out.println("======================开始遍历某一本书的内容=================");
            int num = attributes.getLength();
            for(int i = 0; i < num; i++){
                System.out.print("book元素的第" + (i + 1) +  "个属性名是："
                        + attributes.getQName(i));
                System.out.println("---属性值是：" + attributes.getValue(i));
            }
        }
        else if (!qName.equals("name") && !qName.equals("bookstore")) {
            System.out.print("节点名是：" + qName + "---");
        }
    }

    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        super.endElement(uri, localName, qName);
        if (qName == "book") {
            System.out.println("======================结束遍历某一本书的内容=================");
        }
    }
}


```