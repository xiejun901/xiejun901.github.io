title: singleton 单例模式
date: 2015-10-12 21:07:59
tags: [singleton, c++, design pattern, 设计模式, 单例模式]
categories: design pattern
---

单例模式是一种常见的面向对象设计模式，这里大概总结一下c++中单例模式的实现

在C++中，单例模式通常采用模板实现，一个最基本的实现如下

```c++
template<typename T>
class singleton {
public:
    static T& getInstance() {
        static T _instance;
        return _instance;
    }
private:
    singleton();
    singleton(const singleton&);
    singleton &operator=(const singleton&);
    ~singleton();
};
```

使用方法为 singleton<Widget>::instance.fun(); 函数内的静态局部变量在第一次调用时初始化，但是这个方法对于c++11以前的标准是非线程安全的，标准并未规定多线程中对于静态局部变量的处理
那很自然的办法就是加锁

```c++
template<typename T>
class singleton {
public:
    static T& getInstance() {
        std::lock_guard<std::mutex> lock(_m);
        static T _instance;
        return _instance;
    }
private:
    static std::mutex _m;
    singleton();
    singleton(const singleton&);
    singleton &operator=(const singleton&);
    ~singleton();
};
template<typename T>
std::mutex singleton<T>::_m;

```

但是按照这种方式，每次调用getInstance都要枷锁会导致效率底下，因此人们又采用了一个两次加锁(DCL)的方法

```c++
template<typename T>
class singleton {
public:
    static T& getInstance() {
        if (nullptr == _instance) {
            std::lock_guard<std::mutex> lock(_m);
            if (nullptr == _instance) {
                _instance = new T();
            }
        }
        return *_instance;
    }
private:
    static std::mutex _m;
    static T *_instance;
    singleton();
    singleton(const singleton&);
    singleton &operator=(const singleton&);
    ~singleton();
};
template<typename T>
std::mutex singleton<T>::_m;
template<typename T>
T *singleton<T>::_instance=nullptr;

```

但是由于乱序执行的影响，即由于编译器的优化等，会导致程序实际执行的顺序发生变化，DCL也不能完全保证线程安全，(1. 陈硕：LINUX多线程服务端编程 2. http://www.cs.wustl.edu/~schmidt/PDF/DC-Locking.pdf)，这个时候就需要加入
memeory barrier来保证程序的顺序执行


如下给出一个在linux系统上可用的通过pthread_once来实现的线程安全的单例模式,通过pthread库来保证初始化只被执行一次

```c++
template<typename T>
class singleton {
public:
    static T *instance() {
        pthread_once(&_once, &singleton::init)
        return _instance;
    }
private:
    static pthread_once_t _once;
    static void init(){
        _instance = new T();
    }
    
    static T *_instance;
    singleton() = delete;
    ~singleton() = delete;
    singleton(const singleton &) = delete;
};
template<typename T>
T * singleton<T>::_instance=nullptr;
template<typename T>
pthread_once_t singleton<T>::_once=PTHREAD_ONCE_INIT;
```

不过对于C++11 ,标准中明确规定了静态局部变量在多线程下的处理方式，因此可以直接依靠编译器来保证只被初始化一次，因此就又回到了最初的版本~


```c++
template<typename T>
class singleton {
public:
    static T& getInstance() {
        static T _instance;
        return _instance;
    }
private:
    singleton();
    singleton(const singleton&);
    singleton &operator=(const singleton&);
    ~singleton();
};
```

提供一些参考资料

[C++ and the Perils of Double-Checked Locking](http://www.aristeia.com/Papers/DDJ_Jul_Aug_2004_revised.pdf) 翻译为中文的：[C++和双重检查锁定模式(DCLP)的风险](http://blog.jobbole.com/86392/)

[Double-Checked Locking is Fixed In C++11](http://preshing.com/20130930/double-checked-locking-is-fixed-in-cpp11/)  中文：[C++11 修复了双重检查锁定问题](http://blog.jobbole.com/52164/)

[ C++中线程安全并且高效的singleton](http://www.klayge.org/2015/04/27/c%E4%B8%AD%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E5%B9%B6%E4%B8%94%E9%AB%98%E6%95%88%E7%9A%84singleton/)

[知乎的一些讨论](http://www.zhihu.com/question/35522476)