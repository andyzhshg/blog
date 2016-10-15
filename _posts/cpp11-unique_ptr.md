title: C++11的智能指针(1) unique_ptr
date: 2016-10-09 20:00:00
tags: [技术, C++, C++0x, 智能指针]
categories: 技术

---

C++11新引入了几种智能指针：`unique_ptr`，`shared_ptr`和`weak_ptr`，而原来的`auto_ptr`被弃用。

我会写几篇文章分别来介绍这几种智能指针的用法，本篇主要介绍`unique_ptr`。


主要介绍`unique_ptr`的两个主要特性:

1. 保存对象的指针，当`unique_ptr`本身释放的时候，自动调用对象的析构函数。
2. 唯一拥有它指向的对象，无法通过拷贝构造或者等号进行赋值。

<!-- more -->

我们先定义一个简单的类作为示例：

```c++
//test.h

#include <iostream>

class Test {
public:
    //标准构造函数
    Test(int tag) : tag(tag) {
        std::cout << "Test::Test(int) " << tag << std::endl;
    }
    //标准析构函数
    ~Test() {
        std::cout << "Test::~Test() " << tag << std::endl;
    }
    //测试输出
    void test() {
        std::cout << "Test::test() " << tag << std::endl;
    }

private:
    int tag;
};
```

## 特性1 - 保存对象的指针，当`unique_ptr`本身释放的时候，自动调用对象的析构函数

这是一个智能指针的本分，让我们免去烦人又容易出错的`new/delete`操作。

### 示例1 - 最简单场景

```c++
//example1.cpp

#include <iostream>
#include <memory>
#include "test.h"

int main() {
    std::unique_ptr<Test> p(new Test(1));
    p->test();
    return 0;
}
```

编译并执行

```bash
g++ -o example1 -std=c++11 example1.cpp
./example1
```

输出结果

```
Test::Test(int) 1
Test::test() 1
Test::~Test() 1
```

基本不需要解释，我们看到我们并没有调用delete但是Test的析构函数还是被调用了。

### 示例2 - 有异常的场景

```c++
//example2.cpp

#include <iostream>
#include <memory>
#include <string>
#include "test.h"

int test(int tag) {
    std::unique_ptr<Test> p(new Test(tag));
    if (tag / 2) {
        throw "except, tag = " + std::to_string(tag); //抛出异常
    }
    return tag;
}

int main() {
    try {
        int ret = test(1);  //不会抛出异常
        std::cout << "test(1) return " << ret << std::endl;
        ret = test(2);      //会抛出异常
        //因为上边的语句会抛出异常，下边这句不会被执行
        std::cout << "test(2) return " << ret << std::endl; 
    } catch (std::string e) {
        //异常抛出是会执行
        std::cout << "exception caught: " << e << std::endl;
    }
    return 0;
}
```

编译并执行

```bash
g++ -o example2 -std=c++11 example2.cpp
./example2
```

输出结果

```
Test::Test(int) 1
Test::~Test() 1
test(1) return 1
Test::Test(int) 2
Test::~Test() 2
exception caught: except, tag = 2
```

第一次调用`test(1)`的时候，没有异常抛出，函数正常返回，我们看到函数返回前，`Test`的析构函数得到了调用。

第二次调用`test(2)`的时候，函数抛出了异常，要是普通指针的话，因为函数并没有正常结束，异常之后的语句就不再被调用，包括`delete`语句，就造成了内存泄漏。然而本例中我们看到即使异常抛出，`Test`的析构函数还是得到了调用，这就是智能指针的功劳。


## 特性2 - 唯一拥有它指向的对象，无法通过拷贝构造或者等号进行赋值。

这个特性就是`unique_ptr`独有的特性了。

理解这个特性，需要结合C++11新引入的move语义，move语义不在本文的讨论范围，以后有精力我可能会写一篇关于move语义的文章，现在你想了解move语义的话可以参考这几篇文章：

- [C++11新特性：右值引用与move语义](http://harttle.com/2015/10/11/cpp11-rvalue.html)

- [C++11 标准新特性: 右值引用与转移语义](http://www.ibm.com/developerworks/cn/aix/library/1307_lisl_c11/)

- [[译]详解C++右值引用](http://jxq.me/2012/06/06/%E8%AF%91%E8%AF%A6%E8%A7%A3c%E5%8F%B3%E5%80%BC%E5%BC%95%E7%94%A8/)

我们看一下下边的代码

```c++
//example3.cpp

#include <iostream>
#include <memory>
#include "test.h"

void passTest(std::unique_ptr<Test> t) {
    t->test();
}

std::unique_ptr<Test> getPtr(int tag) {
    std::unique_ptr<Test> p(new Test(tag));
    return p;
}

int main() {
    std::unique_ptr<Test> p = std::unique_ptr<Test>(new Test(1));   //(0)  
    p->test();

    // std::unique_ptr<Test> p1 = p;                //(1)编译失败
    // std::unique_ptr<Test> p1(p);                 //(2)编译失败
    std::unique_ptr<Test> p1 = std::move(p);        //(3)编译通过
    p1->test();                                     //(4)正确
    // p->test();                                   //(5)错误，未定义行为

    p = std::unique_ptr<Test>(new Test(2));
    // passTest(p);                                 //(6)编译失败
    passTest(std::move(p));                         //(7)编译通过
    passTest(std::unique_ptr<Test>(new Test(3)));   //(8)编译通过

    p = getPtr(4);                                  //(9)函数返回

    return 0;
}
```

编译并执行

```bash
g++ -o example3 -std=c++11 example3.cpp
./example3
```

输出结果

```
Test::Test(int) 1
Test::test() 1
Test::test() 1
Test::Test(int) 2
Test::test() 2
Test::~Test() 2
Test::Test(int) 3
Test::test() 3
Test::~Test() 3
Test::~Test() 1
```

这个例子主要是展示了`unique_ptr`的唯一性，也就是说`unique_ptr`唯一持有它指向的对象，无法通过赋值(1)或者拷贝构造(2)的方式进行初始化，它只能接受右值语义的参数来构造(0)(3)。

(4)(5)展示了`move`之后`p`已经失效。

(6)(7)(8)则展示了作为函数参数传递，同样要满足右值语义才可以。

(9)展示了作为函数返回值给`unique_ptr`赋值，这同样是满足右值语义的。

上边的这几个例子都说明了`unique_ptr`的唯一性，我们可以理解成任意时刻，只要你持有一个合法的`unique_ptr`，就可以保证你是唯一的一个持有人，不会出现另一个`unique_ptr`跟你相同的情况。




