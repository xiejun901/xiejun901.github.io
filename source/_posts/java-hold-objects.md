title: Java 学习总结五 集合框架
date: 2015-12-10 14:34:52
tags: [generic, java, object]
---

当我们需要不知道具体多少个数量的对象时候，甚至是不知道对象的具体类型时，我们显然没法在代码里直接写的方式来表示这些对象

	Type refer1;
	Type refer2;
	Type refer3;
	.
	.
	.

这样是不可能的，因为不知道对象具体由多少个，所以我们需要编程语言提供一些机制来处理这个问题，Java有不少方式可以来解决这个问题，比如说数组，可以通过一个数组来持有多个对象，但是由于数组的长度是固定的，虽然我们可以申明足够大的容量来保证要用到的数量小于这个最大值，但我们很多时候希望由更灵活的方式来持有对象。
在Java中，除了数组，我们还可以用List，Set，Queue，Map来持有对象，这些对象类型又被称为**集合类**

#### 泛型和类型安全的容器

在Java支持泛型之前，Java中所有类都集成于Object类，因此可以靠持有Object类来持有其他类，但正因为这样，就意味着所有的类都可以往容器里放，一不小心，就会出现类型错误
在支持泛型之后，这个问题就得到了改善，可以通过泛型来保存一组容器，容器的类型会在编译时得到检测，这样就可以安全的持有对象

```java
import java.util.ArrayList;
public class TestClass {
    public static void main(String[] args) {
        ArrayList<Integer> arr = new ArrayList<Integer>();
        for(int i=0;i<5;i++) {
            arr.add(i);
        }
        for(int i = 0; i< arr.size(); i++ ) {
            System.out.println(arr.get(i));
        }
        for (Integer i : arr) {
            System.out.println(arr.get(i));
        }
    }
}
```

如果我们想要将一个Float对象放入arr中，那么在编译期就会报错，不再会是运行时错误，实现了类型安全

#### Collection 与 Map

1. Collection , 是一个接口，表示一个独立的序列，序列中的元素都服从一条或多条规则。由List， Set， Queue 三个子接口
2. Map ， 一组成对的键值对 对象，允许通过键值来查找值，也被称为关联数组。


#### List

List 由两种实现，ArrayList和LinkedList， 我们可以把他们分别看做数组和链表，其特性也是跟数组和链表类似的，ArrayList的随机访问性能较好(0(1))，但是在链表中插入和移除元素代价较大(O(n))，LinkList 的随机访问性能较差(O(n))，但提供更好的插入和删除操作(O(1))


#### 迭代器

##### Iterator
我们把容器的遍历操作抽象为迭代器的操作，这样我们就可以忽略掉不同容器的差异，使得代码的可复用能力更强，Java中的迭代器只能单向移动，只能用来:

1. 使用iterator()方法来要求容器返回一个iterator
2. 使用next() 获取下一个元素
3. 使用hasNext()来判断是否还有元素
4. 使用remove() 将迭代器返回的元素删除

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
public class TestClass {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i=0; i< 5; i++) {
            list.add(""+i);
        }
        System.out.println(list.toString());
        Iterator<String> iter;
        iter = list.iterator();
        while(iter.hasNext()) {
            iter.next();
            iter.remove();
        }
        System.out.println(list.toString());
    }
}
```

##### ListIterator

ListIterator具有更强大的功能，它只能应用于List类的访问，可以双向移动，可以通过一个listIterator()方法产生一个指向List开头的ListIterator，也可以通过list(n) 产生一个指向列表索引为n的ListIterator

#### Set

如其字面意思，Set表示集合，Set中不保存重复的元素，由 HashSet， TreeSet， LinkedHashSet 几种实现，通过名字我们就可以知道，这几种实现分别是基于hash表(二次探测)， 二叉平衡树， hash表（链表来处理冲突） 实现的


#### Java 中容器总结

Java中提供了大量组织多个对象的方式

1. 数组用来保存明确类型的对象，可以是多维的，可以保存基本类型，但是数组一旦生成，其容量不能发生改变
2. Collection用来保存单一的元素，Map用来保存相关的键值对。通过Java的泛型，可以指定放入容器的元素类型，来保证类型安全。容器不能持有基本类型，但是通过包装类型的自动包装机制可以很好的执行容器持有包装类型和基本类型之间的转换
3. 和数组类似，List具有相类似的功能，但是List可以自动扩容
4. List由两种实现，一种是ArrayList，另一种是LinkedList，ArrayList具有更好的随机访问性能，LinkedList在中间插入和删除元素时具有更好的表现
5. 各种Queue和栈的行为由LinkedList提供支持
6. 提供Map
7. 提供Set
8. 新程序中不应该使用过时的Vector, Hashtable 和 Stack

![](http://7q5fny.com1.z0.glb.clouddn.com/blogThinking%20in%20Java%204th%20Ed.jpg)