---
title: 模板友元
date: 2019-10-19 15:42:47
categories:
- C++
tags:
- C++
---

# 模板友元

函数模板与类模板的声明都可以在非局部类或类模板中使用`friend`说明符修饰（但只有函数模板可以定义在授予它友元权限的类或类模板中）。在这种情况下，所有特例化的模板都会成为一个友元，无论他是被隐式实例化、部分特例化或显式特例化。

```c++
class A {
    template<typename T>
    friend class B; // every B<T> is a friend of A
 
    template<typename T>
    friend void f(T) {} // every f<T> is a friend of A
};
```

友元声明不能用于部分特例化，但可以用于完全（显式）特例化：

```c++
template<class T> class A {}; // primary
template<class T> class A<T*> {}; // partial
template<> class A<int> {}; // full
class X {
    template<class T> friend class A<T*>; // error!
    friend class A<int>; // OK
};
```

当友元声明用于一个函数模板的完全特例化时，不能使用`inline`关键字和默认参数：

```c++
template<class T> void f(int);
template<> void f<int>(int);
 
class X {
    friend void f<int>(int x = 1); // error: default args not allowed
};
```

# 模板友元运算符

模板友元的一个常见用途是声明一个非成员运算符重载其默认操作；例如，对用户定义的`Foo<T>`声明`operator<<(std::ostream&, const Foo<T>&)`。

该运算符可以定义在类内部，这样的效果是为每一个类型`T`生成一个独立的非模板的`operator<<`，并使这个非模板的`operator<<`成为`Foo<T>`的友元。

```c++
#include <iostream>
 
template<typename T>
class Foo {
 public:
    Foo(const T& val) : data(val) {}
 private:
    T data;
 
    // generates a non-template operator<< for this T
    friend std::ostream& operator<<(std::ostream& os, const Foo& obj)
    {
        return os << obj.data;
    }
};
 
int main()
{
    Foo<double> obj(1.23);
    std::cout << obj << '\n';
}
```

或者函数模板必须在类之前进行模板声明，在这种情况下`Foo<T>`之内的友元声明可使用类型`T`进行完全特例化的`operator<<`：

```c++
#include <iostream>
 
template<typename T>
class Foo; // forward declare to make function declaration possible
 
template<typename T> // declaration
std::ostream& operator<<(std::ostream&, const Foo<T>&);
 
template<typename T>
class Foo {
 public:
    Foo(const T& val) : data(val) {}
 private:
    T data;
 
    // refers to a full specialization for this particular T 
    friend std::ostream& operator<< <> (std::ostream&, const Foo&);
    // note: this relies on template argument deduction in declarations
    // can also specify the template argument with operator<< <T>"
};
 
// definition
template<typename T>
std::ostream& operator<<(std::ostream& os, const Foo<T>& obj)
{
    return os << obj.data;
}
 
int main()
{
    Foo<double> obj(1.23);
    std::cout << obj << '\n';
}
```





内容来自：https://en.cppreference.com/w/cpp/language/friend

