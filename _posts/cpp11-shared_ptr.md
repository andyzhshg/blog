title: C++11的智能指针(2) shared_ptr
date: 2016-10-19 23:17:00
tags: [技术, C++, C++0x, 智能指针]
categories: C++

---

C++11新引入了几种智能指针：`unique_ptr`，`shared_ptr`和`weak_ptr`，而原来的`auto_ptr`被弃用。

我会写几篇文章分别来介绍这几种智能指针的用法，本篇主要介绍`shared_ptr`。

`shared_ptr`可以说是我们最常规意义上理解的智能指针了，区别于`unique_ptr`，`share_ptr`有拷贝构造函数和赋值操作符，每当`shared_ptr`多出一个拷贝，所有拷贝的引用计数都会增加。

<!-- more -->

## `shared_ptr`的常规用法

```c++
// example1.cpp

#include <iostream>
#include <memory>

class Test {
public:
    Test(int tag) : _tag(tag) {
        std::cout << "Test::Test() " << _tag << std::endl;
    }

    ~Test() {
        std::cout << "Test::~Test() " << _tag << std::endl;
    }

    void test() {
        std::cout << "Test::test() " << _tag << std::endl;
    }
private:
    int _tag;
};

int main() {
    std::shared_ptr<Test> p1(new Test(1));
    p1->test();
    std::cout << "p1:" << p1.use_count() << std::endl;
    std::cout << "----------------" << std::endl;

    std::shared_ptr<Test> p2 = std::make_shared<Test>(2);
    p2->test();
    std::cout << "p2:" << p2.use_count() << std::endl;
    std::cout << "----------------" << std::endl;
    
    std::shared_ptr<Test> p3 = p1;
    p3->test();
    std::cout << "p1:" << p1.use_count() << std::endl;
    std::cout << "p3:" << p3.use_count() << std::endl;
    std::cout << "----------------" << std::endl;

    p1.reset();
    std::cout << "p1:" << p1.use_count() << std::endl;
    std::cout << "p3:" << p3.use_count() << std::endl;
    std::cout << "----------------" << std::endl;

    p2 = p3;
    p2->test();
    std::cout << "p2:" << p2.use_count() << std::endl;
    std::cout << "p3:" << p3.use_count() << std::endl;
    std::cout << "----------------" << std::endl;

    return 0;
}
```

编译

```
g++ -o example1 -std=c++11 example1.cpp
```

运行结果

```
Test::Test() 1			
Test::test() 1
p1:1						(1)
----------------
Test::Test() 2
Test::test() 2
p2:1						(2)
----------------
Test::test() 1
p1:2
p3:2						(3)
----------------
p1:0
p3:1						(4)
----------------
Test::~Test() 2
Test::test() 1
p2:2
p3:2						(5)
----------------
Test::~Test() 1		(6)
```

(1) 展示了用`shared_ptr`的构造函数来生成一个`shared_ptr`对象，并且我们看到他的引用计数现在是`1`。

(2) 展示了用`make_shared`来生成一个`shared_ptr`，可以看到，这里我们终于彻底告别了`new`，是不是感觉有点暗爽。

(3) 展示了`shared_ptr`的赋值操作，我们看到，赋值之后，`p1`和`p3`的引用计数都增加到了`2`。

(4) `reset`操作使得`p1`变为空的，所以它的引用计数为`0`，而`p3`的引用计数则减少到了1，此时的`p1`和`p3`已经完全不是一回事了。

(5) `p2 = p3`的操作使得`p2`原来指向的对象被释放，所以我们首先看到一条析构函数的输出。然后我们看到`p2` `p3`的的引用计数都变成了`2`。

(6) 程序退出的时候，很自然的，所有的智能指针都出了作用域，所以最后一条析构调用被输出。

## 类型转换

假设我们两个类
```c++
#include <iostream>
#include <memory>

class Base {
public:
    Base() {
        std::cout << "Base::Base()" << std::endl;
    }
    ~Base() {
        std::cout << "Base::~Base()" << std::endl;
    }
    virtual void test() {
        std::cout << "Base::test()" << std::endl;
    }
};

class Derived : public Base {
public:
    Derived() {
        std::cout << "Derived::Derived()" << std::endl;
    }
    ~Derived() {
        std::cout << "Derived::~Derived()" << std::endl;
    }
    
    virtual void test() {
        std::cout << "Derived::test()" << std::endl;
    }
};

int main() {
    std::shared_ptr<Base> pb = std::make_shared<Base>();
    std::shared_ptr<Derived> pd = std::make_shared<Derived>();
    std::cout << "pb.use_count() " << pb.use_count() << std::endl;
    std::cout << "pd.use_count() " << pd.use_count() << std::endl;
    pb = static_cast<Base>(pd);
    std::cout << "pb.use_count() " << pb.use_count() << std::endl;
    std::cout << "pd.use_count() " << pd.use_count() << std::endl;
    pb->test();
    return 0;
}
```
用static_cast, dynamic_cast, const_cast是无法用在不同的shared_ptr之上的。

## 自己实现一个`shared_ptr`

```c++
//shared_ptr.h
namespace up4dev {

    template<typename T>
    class shared_ptr {
        public:
            shared_ptr() : _p(nullptr), _c(nullptr){
            }

            ~shared_ptr() {
                reset();
            }

            shared_ptr(T* p) : _p(p), _c(new int(1)) {
            }

            shared_ptr(const shared_ptr& sp) {
                reset();
                _p = sp._p;
                _c = sp._c;
                *_c += 1;
            }

            shared_ptr& operator=(const shared_ptr& sp) {
                reset();
                _p = sp._p;
                _c = sp._c;
                *_c += 1;
                return *this;                
            }

            T* get() {
                return _p;
            }

            T* operator->() {
                return _p;
            }

            void reset() {
                if (_c) {
                    *_c -= 1;
                    if (*_c == 0) {
                        delete _p;
                        delete _c;
                    }
                    _p = nullptr;
                    _c = nullptr;
                }
            }

            int use_count() {
                return _c ? *_c : 0;
            }

        private:
            T* _p;
            int* _c;
    };

}
```

对前文的测试用例稍加修改，用新写的shared_ptr来替换标准库的实现

```c++
//example3.cpp

#include <iostream>
#include "shared_ptr.h"

class Test {
public:
    Test(int tag) : _tag(tag) {
        std::cout << "Test::Test() " << _tag << std::endl;
    }

    ~Test() {
        std::cout << "Test::~Test() " << _tag << std::endl;
    }

    void test() {
        std::cout << "Test::test() " << _tag << std::endl;
    }
private:
    int _tag;
};

int main() {
    up4dev::shared_ptr<Test> p1(new Test(1));
    p1->test();
    std::cout << "p1:" << p1.use_count() << std::endl;
    std::cout << "----------------" << std::endl;

    up4dev::shared_ptr<Test> p2 = up4dev::shared_ptr<Test>(new Test(2));
    p2->test();
    std::cout << "p2:" << p2.use_count() << std::endl;
    std::cout << "----------------" << std::endl;
    
    up4dev::shared_ptr<Test> p3 = p1;
    p3->test();
    std::cout << "p1:" << p1.use_count() << std::endl;
    std::cout << "p3:" << p3.use_count() << std::endl;
    std::cout << "----------------" << std::endl;

    p1.reset();
    std::cout << "p1:" << p1.use_count() << std::endl;
    std::cout << "p3:" << p3.use_count() << std::endl;
    std::cout << "----------------" << std::endl;

    p2 = p3;
    p2->test();
    std::cout << "p2:" << p2.use_count() << std::endl;
    std::cout << "p3:" << p3.use_count() << std::endl;
    std::cout << "----------------" << std::endl;

    return 0;
}
```

编译

```
g++ -o example3 -std=c++11 example3.cpp
```

运行结果

```
Test::Test() 1
Test::test() 1
p1:1
----------------
Test::Test() 2
Test::test() 2
p2:1
----------------
Test::test() 1
p1:2
p3:2
----------------
p1:0
p3:1
----------------
Test::~Test() 2
Test::test() 1
p2:2
p3:2
----------------
Test::~Test() 1
```






## C++11的智能指针系列文章

- [C++11的智能指针(1) unique_ptr](http://www.up4dev.com/2016/10/09/cpp11-unique_ptr/)

- [C++11的智能指针(2) shared_ptr](#)



