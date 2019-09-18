---
title: Classes
date: 2019-08-27 10:47:18
categories:
- C++
tags:
- C++ Primer
- RE：从零开始的C++学习
- C++
---

# 定义抽象数据类型

- 成员函数必须在类的内部声明，可以在类的内部或外部定义。定义在类内部的函数是隐式的`inline`函数。
- 调用成员函数时，`this`被初始化为调用成员函数的对象的地址。普通成员函数的`this`为指向非`const`对象的`const`指针，`const`成员函数的`this`为指向`const`对象的`const`指针。

- `const`对象以及指向`const`对象的引用或指针只能调用对象的`const`成员函数。
- 构造函数在创建类类型的对象时执行。构造函数不能被声明为`const`。当我们在创建一个类的`const`对象时，构造函数可以向对象写入值。
- 若一个类定义了构造函数，该类就不会再有默认构造函数，除非我们自己定义默认构造函数。

# 访问控制与封装

- `struct`关键字的默认访问权限为`public`，`class`关键字的默认访问权限为`private`。

## 友元

- 如果类想把一个函数作为它的友元，只需要增加一条以`friend`关键字开始的函数声明语句即可。
- 友元声明只能出现在类定义的内部，但在类内出现的具体位置不限。友元不是类的成员，也不受访问控制的约束。
- 友元的声明仅仅指定了访问的权限，而非一个真正意义上的函数声明。如果我们希望类的用户能够调用某个友元函数，那么我们必须在友元声明之外再对函数进行一次声明。

## 类的其他特性

## 类成员

- 由类定义的类型名与其他类成员一样存在访问权限控制，可以是`public`或`private`中的一种。
- 不同于普通成员，定义类型的成员必须先定义后使用。
- `mutable`数据成员永远不是`const`，即使它是`const`对象的成员。

## 返回`*this`的成员函数

- 一个`const`成员函数如果以引用的形式返回`*this`，那么它的返回类型是对`const`的引用。

## 类类型

- 可以只声明但不定义一个类，这样的声明称为**前向声明**（forward declaration），声明的类在定义之前称为**不完全类型**（incomplete type）。

# 类的作用域

## 名字查找与类的作用域

- 编译器处理完类内所有声明之后再处理成员函数的定义。
- 由于成员函数的函数体在整个类可见之后才被处理，因此它能使用类中定义的任何名字。但在类中的声明使用的名字必须在使用前确保可见。
- 在类中，如果一个成员使用了在外层作用域中定义的类型名，则类之后将不能重新定义该名字。
```c++
typedef double Money;
class Account {
public:
    Money balance() { return bal; }  // uses Money from the outer scope
    private:
    typedef double Money; // error: cannot redefine Money
    Money bal;
    // ...
};
```

# 构造函数再探

## 构造函数初始值列表

- 如果成员是`const`或引用的话，构造函数必须将其初始化。
- 类成员初始化的顺序由它们在类定义中出现的顺序决定，与构造函数初始化列表中的顺序无关。

## 委托构造函数

- 委托构造函数使用它所属类的其他构造函数执行它自己的初始化过程。
```c++
class Sales_data {
public:
    // nondelegating constructor initializes members from corresponding arguments
    Sales_data(std::string s, unsigned cnt, double price):
            bookNo(s), units_sold(cnt), revenue(cnt*price) {
}
    // remaining constructors all delegate to another constructor
    Sales_data(): Sales_data("", 0, 0) {}
    Sales_data(std::string s): Sales_data(s, 0,0) {}
    Sales_data(std::istream &is): Sales_data()
                                        { read(is, *this); }
    // other members as before
};
```
- 受委托的构造函数会先执行。


## 隐式类类型转换

- 可以将构造函数声明为`explicit`以避免隐式类型转换。

## 聚合类

## 字面值常量类

# 类的静态成员

