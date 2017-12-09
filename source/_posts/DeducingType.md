title: C++ 类型推导
date: 2015-10-12 22:07:06
tags: [C++, Effective Modern C++, C++类型推导, template, auto, decltype]
categories: Effective Modern C++
---

本篇大概记录一下看Effective Modern C++这本书时对于类型推导这一部分内容。

## 如何查看类型推导

要了解类型推导首先应该是知道如何查看类型推导，下面简要介绍几种查看类型推导的方法

### 编译期查看

#### 通过IDE提供的信息查看

以下是通过vs来查看auto的类型推断截图
![](http://7q5fny.com1.z0.glb.clouddn.com/blogtype_deduced.jpg)

#### 通过错误提示信息查看

这方法也是罪了,如下代码
```c++
template<typename T>
class D;
int main()
{
    int x;
    const auto &y = x;
    D<decltype(x)> xType;
    D<decltype(y)> yType;
}
```
编译提示信息：

    1>source2.cpp(7): error C2079: 'xType' uses undefined class 'D<int>'
    1>source2.cpp(8): error C2079: 'yType' uses undefined class 'D<const int &>'
    
通过错误信息可以很清楚的看到 x 和 y 的类型

### 运行期查看

先介绍一个类 type_info，这个类是用来保存类的类型信息的，有一个成员name是来表示类型的名字的，可以通过typeid和type_info来在运行期获取类型的信息，
不过由于type_info处理类型信息时是值传递的，所以显示信息会不太准，一些引用特性和常量特性会被消除，具体哪些会被消除可以根据后边会介绍的模板类型来知道。我们可以通过boost::typeindex来获得更准确类型信息

## 模板的类型推导

后边通过如下一段伪代码来举例, 我们需要得到ParamType 和 T

```c++
template<typename T>
void f(ParamType param);

f(expr);
```

ParamType 通过expr的类型推断而来，T的类型由ParamType 和 expr的类型共同得出
具体可以分三种情况来看：
1. ParamType 是指针或者引用但是不是universal reference(好吧，读到后面终于搞懂这个是个啥东西了。。对于type&& 是分两种情况的，一种是只能帮定位右值引用，另一种就随便了，因此被作者称为universal reference)
2. ParamType 是universal reference
3. ParamType 既不是指针也不是引用

### ParamType 是指针或者引用但是不是universal reference

对于这种情况，类型推导的原则是
1. 如果expr是引用类型，忽视掉引用部分
2. 然后通过expr的类型去匹配ParamType 来 得出T

例如
```c++
template<typename T>
void f(T& param);

int x=27;
const int cx=x;
const int& rx=x;

f(x);   //T的类型是int,        param 的类型是 int&
f(cx);  //T的类型是const int,  param 的类型是 const int&
f(rx);  //T的类型是const int,  param 的类型是 const int&
```

改变一下ParamType 再看！

```c++
template<typename T>
void f(const T& param);

int x=27;
const int cx=x;
const int& rx=x;

f(x);   //T的类型是int,        param 的类型是 const int&
f(cx);  //T的类型是int,        param 的类型是 const int&
f(rx);  //T的类型是int,        param 的类型是 const int&
```

指针类型原则是一样的，就不再细说了

### ParamType 是 universal reference

对于这种情况，类型推导的原则是
1. 如果expr是左值，那么ParamType和T都被推导为左值引用类型
2. 如果expr是右值，那么按照之前的规则，根据expr匹配出ParamType，然后再推导出T

需要注意的是，情况1有两个特殊的店，第一，它是唯一一种T被推导为引用类型的，第二，尽管ParamType被定义为&& 但是推导的类型为&, 下面举例说明

```c++
template<typename T>
void f(T&& param);

int x=27;
const int cx=x;
const int& rx=x;

f(x);   //T的类型是int&,              param 的类型是 int&
f(cx);  //T的类型是const int&,        param 的类型是 const int&
f(rx);  //T的类型是const int&,        param 的类型是 const int&
f(27)   //T的类型是int                param 的类型是 int&&
```

### ParamType既不是引用也不是指针

对于这种情况，类型推导的原则如下
1. 如果expr是引用，那么忽略掉引用
2. 如果是const, 那么忽略掉const。对于volatile也同样处理

看下下面的例子

```c++
template<typename T>
void f(T param);

int x=27;
const int cx=x;
const int& rx=x;

f(x);   //T的类型是int,    param 的类型是 int
f(cx);  //T的类型是int,    param 的类型是 int
f(rx);  //T的类型是int,    param 的类型是 int
```
这里我们关注一下指向常量的常量指针的问题，
例如我们有如下定义
```c++
const char* p2k;       //(1)
char* const kp;        //(2)
const char* const kp2k //(3)
```
这三种情况是不一样的，
情况1是指向常量的指针，表示的是指针指向的内容不可变，即我们无法执行*p2k = another这种操作
情况2是常量指针，表示的是该指针变量是一个常量，即我们无法执行++kp这种操作
情况3是以上两种情况结合， 指针不能变，指针指向的内容也不能变

所以对于以下代码
```c++
template<typename T>
void f(T param);

const char* const ptr = "hello";

f(ptr);   //T的类型是const char*,    param 的类型是 const char*
```
为什么呢，因为我们模板参数是按值传递的，所以ptr的常量特性被忽略掉了，然后得出了ParamType为const char\* ， 从而推出T是 const char\* ,
这也很好理解，因为ptr指向的内容是不能改变的，那么即便通过函数传递进去了，内存的那部分内容还是应该是不能改变的，只是通过值传递的param是可以改变的了

### 数组和函数的类型推导
数组和函数在作为参数传递时，如果是按值传递，那么会自动退化为指针，如果是按引用传递，则不会退化为指针，根据这个原则，数组和函数的类型推导就很明白了
举一个利用类型推导获取数组size的例子
```c++
template<typename T, std::size_t N>
constexpr std::size_t arrySize(T (&)[N]) noexcept
{
    return N;
}
```

## auto的类型推导 

auto的类型推导在大部分情况下都和模板类型推导一模一样，除了当类型为initial_list的时候
先介绍一下通常情况下的，与模板类型推导一样的那一部分
```c++
auto x = 27;           //x类型为int
const auto cx = x;     //cx类型为const int
const auto &rx = x;    //rx的类型为const int&

auto&& uref1 = x;    //uref1的类型是 int&
auto&& uref2 = cx;   //uref2的类型为 const int&
auto&& uref3 = rx;   //uref3的类型为 const int&
auto&& uref4 = 27;   //uref4的类型为 int&&

const char name[]="xiejun";

auto arr1 = name; //arr1的类型为 const char*;
auto arr2 = name; //arr2的类型为 const char (&)[7]

void Func(int, double);
auto func1 = Func; //func1的类型为 void (*)(int, double)
auto &func2 = Func; //func2的类型为 void (&)(int, double)

```
接下来看一下对于initial_list 的类型推导

```c++
auto x1 = {27};           //x类型为initial_list<int>
auto x2 = {27, 3.0};  //报错

template<typename T>
void f(T param);

f({11, 23, 9}); //报错无法进行类型推导

template<typename T>
void f(std::initial_list<T> param);

f({11, 23, 9}); //ParamType 为initial_list<int> T为int

```

对于c++14, auto 可以用来定义函数返回值和lambda 表达式的参数，但是对于这两种情况，auto 将按照模板的类型推导来进行，对于initial_list将无法推导

```c++
auto createInitList()
{
    return {1, 2, 3, 4, 5}; //出错
}

auto resetV = [&v](const auto &newValue){v=newValue;}
resetV({1, 2, 3}); //出错
```

## decltype的使用
decltype用来获取一个表达式的类型，举以下几个例子
```c++
int i=0;  //decltype(i) 是int
const int& cri=i;  //decltype(i) 是const int&
bool f();  /decltype(f) 是bool
```
在C++11 中，decltype的主要作用是用来定义一个模板函数的返回值类型依赖于其输入类型，例如我们通常情况下vector<T> 的operator[]返回值为T&，
但是对于vector<bool>，operator[]返回值为bool，不再是引用，我们想要我们写的函数的返回值与标准库中的operator[]表现一样，
这时候便可以采用decltype来完成

```c++
template<typename T>
class D;

template<typename Container, typename Index>
auto authAndAccess(Container &c, Index i)
->decltype(c[i])
{
    return c[i];
}
int main()
{
    vector<int> vec{ 1, 2, 3, 4 };
    D<decltype(authAndAccess(vec, 2))> d;
    
    return 0;
}
```

编译信息：
1>error C2079: 'd' uses undefined class 'D<int &>'
通过以上代码，我们通过错误信息可以得出函数的返回值是引用类型

不过对于C++14，从语法层面上，可以不加decltype的
```c++
template<typename T>
class D;

template<typename Container, typename Index>
auto authAndAccess(Container &c, Index i)
{
    return c[i];
}
int main()
{
    vector<int> vec{ 1, 2, 3, 4 };
    D<decltype(authAndAccess(vec, 2))> d;
    
    return 0;
}
```
这种方式，如果采用c++11编译时会报出错误信息的

    ‘authAndAccess’ function uses ‘auto’ type specifier without trailing return type

对于c++14是能编译通过的(auto语法部分而不是整个示例代码)，但是正如我们前面提到的，返回值的引用类型或者常量类型是会被忽略的，因此对于以上示例代码，auto 推导出来的类型会是int而不是int&， 提示信息如下

    error: aggregate ‘D<int> d1’ has incomplete type and cannot be defined D<decltype(authAndAccess(vec1, 2))> d1;
    
所以说对于 C++14 我们可以采用如下方式来写代码
```c++
template<typename T>
class D;

template<typename Container, typename Index>
decltype(auto) authAndAccess(Container &c, Index i)
{
    return c[i];
}
int main()
{
    vector<int> vec{ 1, 2, 3, 4 };
    D<decltype(authAndAccess(vec, 2))> d;
    
    return 0;
}
```
auto 表明这个地方是要使用类型推断，decltype表明这个地方使用的是decltype的规则来进行类型推断，这样的话，推断出来的类型就是 int&了

    error: aggregate ‘D<int&> d’ has incomplete type and cannot be defined D<decltype(authAndAccess(vec, 2))> d;


不过这个写法还是有一定问题的，需要继续被完善。我们可以知道，以上代码参数传递是时无法传递右值的，因此我们最终的写法是
```c++
//C++14
template<typename Container, typename Index>
decltype(auto) authAndAccess(Container &&c, Index i)
{
    return std::forward<Container>(c)[i];
}
//C++11
template<typename Container, typename Index>
auto authAndAccess(Container &&c, Index i)
->decltype(std::forward<Container>(c)[i])
{
    return std::forward<Container>(c)[i];
}
```

关于decltype还有一个很好玩的地方，对于 int x=0; decltype(x) 是int , 但是decltype((x)) 是int& 这个要注意一下
也就是说会有如下情况
```c++
decltype(auto) f1()
{
    int x=0;
    ...
    return x; //decltype(x) 是int, 函数f1的返回值是int类型
}

decltype(auto) f1()
{
    int x=0;
    ...
    return (x); //decltype(x) 是int&, 函数f1的返回值是int&类型
}
```


    
