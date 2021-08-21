---
title: Object-Oriented Programming
date: 2019-10-09 16:47:24
categories:
- C++
tags:
- C++ Primer
- RE：从零开始的C++学习
- C++
---

# 面向对象程序设计

## OOP：概述

- 对于某些函数，基类希望它的派生类各自定义适合自生的版本，此时基类就将这些函数声明成**虚函数**（virtual function）：
```c++
class Quote {
public:
    std::string isbn() const;
    virtual double net_price(std::size_t n) const;
};
```

- 派生类必须在其内部对所有重新定义的虚函数进行声明。C++11新标准允许派生类显式地注明它将使用哪个成员函数改写基类的虚函数，具体措施是在函数形参列表之后增加一个`override`关键字。

- `print_total`是使用引用类型调用`net_price`函数的，实际传入`print_total`的对象类型将决定执行`net_price`的哪个版本：
```c++
// calculate and print the price for the given number of copies, applying any discounts
double print_total(ostream &os,
                   const Quote &item, size_t n)
{
    // depending on the type of the object bound to the item parameter
    // calls either Quote::net_price or Bulk_quote::net_price
    double ret = item.net_price(n);
    os << "ISBN: " << item.isbn() // calls Quote::isbn
       << " # sold: " << n << " total due: " << ret << endl;
     return ret;
}

// basic has type Quote; bulk has type Bulk_quote
print_total(cout, basic, 20); //  calls Quote version of net_price
print_total(cout, bulk, 20);  //  calls Bulk_quote version of net_price
```

## 定义基类和派生类

### 定义基类

- 作为继承关系中根节点的类通常都会定义一个虚析构函数。
- 在C++语言中，基类必须将它的两种成员函数区分开：一种是基类希望其派生类进行覆盖的函数；另一种是基类希望派生类直接继承而不要改变的函数。对于前者，基类通常将其定义为**虚函数**（virtual）。当我们使用指针或引用调用虚函数时，该调用将被动态绑定。
- 任何构造函数之外的非静态函数都可以是虚函数。关键字`virtual`只能出现在类内部的声明语句之前而不能用于类外部的函数定义。
- 如果基类把一个函数声明为虚函数，则该函数在派生类中隐式地也是虚函数。

- 成员函数若没有被声明为虚函数，则其解析过程发生在编译时而非运行时。

- 对于某些成员，基类希望它的派生类有权限访问该成员，同时禁止其他用户访问。我们使用`protected`修饰这样的成员。

### 定义派生类

- 派生类使用**类派生列表**（class derivation list）明确指出它是从哪个（哪些）基类继承而来。其中每个基类前面可以有以下三种访问说明符中的一个：`public`、`protected`或者`private`。
- 如果一个派生是共有的，则基类的共有成员也是派生类接口的组成部分。并且我们可以将公有派生类型的对象绑定到基类的引用或指针上。
- 如果派生类没有覆盖其基类中的某个虚函数，则该虚函数的行为类似与其他的普通成员，派生类会直接继承其在基类中的版本。

- 派生类可以在它覆盖的函数前使用`virtual`关键字，但不是一定要这么做。
- 因为在派生类对象中含有与其基类对应的组成部分，所以我们能把派生类的对象当成基类对象来使用，而且我们也能将基类的引用或指针绑定到派生类对象的基类部分上。这种转换通常称为**派生类到基类**（derived-to-base）类型转换。
```c++
Quote item;        //  object of base type
Bulk_quote bulk;   //  object of derived type
Quote *p = &item;  //  p points to a Quote object
p = &bulk;         //  p points to the Quote part of bulk
Quote &r = bulk;   //  r bound to the Quote part of bulk
```

- 和其他创建了基类对象的代码一样，派生类也必须使用基类的构造函数来初始化它的基类部分。
```c++
Bulk_quote(const std::string& book, double p,
           std::size_t qty, double disc) :
           Quote(book, p), min_qty(qty), discount(disc) { }
    // as before
};
```

- 除非我们特别指出，否则派生类对象的基类部分会像数据成员一样执行默认初始化。
- 派生类可以访问基类中的`public`和`protected`成员：
```c++
// if the specified number of items are purchased, use the discounted price
double Bulk_quote::net_price(size_t cnt) const
{
    if (cnt >= min_qty)
        return cnt * (1 - discount) * price;
    else
        return cnt * price;
}
```

- 派生类的作用域嵌套在基类的作用域之内。
- 如果基类定义了一个`static`成员，则在整个继承体系中只存在该成员的唯一定义。
```c++
class Base {
public:
    static void statmem();
};
class Derived : public Base {
    void f(const Derived&);
};
```

- 假设某静态成员是可访问的，则我们既能通过基类使用它也能通过派生类使用它：
```c++
void Derived::f(const Derived &derived_obj)
{
    Base::statmem();    // ok: Base defines statmem
    Derived::statmem(); // ok: Derived inherits statmem
    // ok: derived objects can be used to access static from base
    derived_obj.statmem(); // accessed through a Derived object
    statmem();             // accessed through this object
}
```

- 派生类的声明中包含类名但不包含它的派生列表：
```c++
class Bulk_quote : public Quote; // error: derivation list can't appear here
class Bulk_quote;                // ok: right way to declare a derived class
```

- 一个类在被用作基类之前，必须以及定义而非仅仅声明：
```c++
class Quote;   // declared but not defined
// error: Quote must be defined
class Bulk_quote : public Quote { ... };
```

- C++11新标准提供了一种防止继承发生的方法，即在类名后跟一个关键字final：
```c++
class NoDerived final { /*  */ }; // NoDerived can't be a base class
class Base { /*  */ };
// Last is final; we cannot inherit from Last
class Last final : Base { /*  */ }; // Last can't be a base class
class Bad : NoDerived { /*  */ };   // error: NoDerived is final
class Bad2 : Last { /*  */ };       // error: Last is final
```

### 类型转换与继承

- 可以将基类的指针绑定到派生类对象上。
- 和内置指针一样，智能指针类也支持派生类向基类的类型转换，这意味着我们可以将一个派生类对象的指针存储在一个基类的智能指针内。
- 当我们使用存在继承关系的类型时，必须将一个变量或其他表达式的**静态类型**（static type）与该表达式表示对象的**动态类型**（dynamic type）区分开来。表达式的静态类型在编译时是已知的，它是变量声明时的类型或表达式生成的类型；动态类型则是变量或表达式表示的内存中的对象的类型。动态类型在运行时才可知。
- 如果表达式既不是引用也不是指针，则它的动态类型永远与静态类型一致。
- 因为一个基类的对象可能是派生类对象的一部分，也可能不是，所以不存在从基类向派生类的自动类型转换：
```c++
Quote base;
Bulk_quote* bulkP = &base;  // error: can't convert base to derived
Bulk_quote& bulkRef = base; // error: can't convert base to derived
```

- 派生类向基类的自动类型转换只对指针或引用类型有效，在派生类类型和基类类型之间不存在这样的转换。

## 虚函数

- 因为我们直到运行时才能知道到底调用了哪个版本的虚函数，所以所有的虚函数都必须有定义。
- 动态绑定只有当我们通过指针或引用调用虚函数时才会发生，当我们通过一个具有普通类型（非引用非指针）的表达式调用虚函数时，在编译时就会将调用的版本确定下来。
```c++
Quote base("0-201-82470-1", 50);
print_total(cout, base, 10);    // calls Quote::net_price
Bulk_quote derived("0-201-82470-1", 50, 5, .19);
print_total(cout, derived, 10); // calls Bulk_quote::net_price

base = derived;         // copies the Quote part of derived into base
base.net_price(20);     // calls Quote::net_price
```

- 当且仅当通过指针或引用调用虚函数时，才会在运行时解析该调用，也只有在这种情况下对象的动态类型才有可能与静态类型不同。
- 一旦某个函数被声明成虚函数，则在所有的派生类中他都是虚函数。
- 一个派生类的函数如果覆盖了某个继承而来的虚函数，则它的形参类型必须与被它覆盖的基类函数完全一致。
- 如果我们使用`override`标记了某个函数，但该函数没有覆盖已存在的虚函数，此时编译器将报错：
```c++
struct B {
    virtual void f1(int) const;
    virtual void f2();
    void f3();
};
struct D1 : B {
    void f1(int) const override; // ok: f1 matches f1 in the base
    void f2(int) override; // error: B has no f2(int) function
    void f3() override;    // error: f3 not virtual
    void f4() override;    // error: B doesn't have a function named f4
};
```

- 可以将某个函数指定为`final`，被定义为`final`的函数不能被它的派生类覆盖：
```c++
struct D2 : B {
    // inherits f2() and f3() from B and overrides f1(int)
    void f1(int) const final; // subsequent classes can't override f1(int)
};
struct D3 : D2 {
    void f2();          // ok: overrides f2 inherited from the indirect base, B
    void f1(int) const; // error: D2 declared f2 as final
};
```

- 如果某次函数调用使用了默认实参，则该实参值由本次调用的静态类型决定。也就是说，若通过基类的引用或者指针调用函数，则使用基类中定义的默认实参，即使实际运行的是派生类中的函数版本也是如此。
- 使用作用域运算符可以强制执行虚函数的某个特定版本：
```c++
//  calls the version from the base class regardless of the dynamic type of baseP
double undiscounted = baseP->Quote::net_price(42);
```

## 抽象基类

- 可以将函数定义为**纯虚**（pure virtual）函数，表面该函数是没有实际意义的。纯虚函数无需定义，通过在函数体的位置书写`=0`可以将一个虚函数说明为纯虚函数。其中，`=0`只能出现在类内部的虚函数声明语句处：
```c++
// class to hold the discount rate and quantity
// derived classes will implement pricing strategies using these data
class Disc_quote : public Quote {
public:
    Disc_quote() = default;
    Disc_quote(const std::string& book, double price,
              std::size_t qty, double disc):
                 Quote(book, price),
                 quantity(qty), discount(disc) { }
    double net_price(std::size_t) const = 0;
protected:
    std::size_t quantity = 0; //  purchase size for the discount to apply
    double discount = 0.0;    //  fractional discount to apply
};
```

- 可以为纯虚函数提供定义，不过函数体必须定义在类的外部，也就是说，不能在类的内部为一个`=0`的函数提供函数体。
- 含有（或者未经覆盖直接继承）纯虚函数的类是**抽象基类**（abstract base class）。不能直接创建一个抽象基类的对象。
```c++
// Disc_quote declares pure virtual functions, which Bulk_quote will override
Disc_quote discounted; // error: can't define a Disc_quote object
Bulk_quote bulk;       // ok: Bulk_quote has no pure virtual functions
```

## 访问控制与继承

- 一个类使用`protected`关键字来声明那些它希望与派生类分享但是不想被其他公共访问使用的成员。
- 派生类的成员或友元只能通过派生类对象来访问基类的受保护成员。派生类对于一个基类对象中的受保护成员没有任何访问特权。
```c++
class Base {
protected:
    int prot_mem;     // protected member
};
class Sneaky : public Base  {
    friend void clobber(Sneaky&);  // can access Sneaky::prot_mem
    friend void clobber(Base&);    // can't access Base::prot_mem
    int j;                          // j is private by default
};
// ok: clobber can access the private and protected members in Sneaky objects
void clobber(Sneaky &s) { s.j = s.prot_mem = 0; }
// error: clobber can't access the protected members in Base
void clobber(Base &b) { b.prot_mem = 0; }
```

- 派生访问说明符对于派生类的成员（及友元）能否访问其直接基类的成员没有影响，对基类成员的访问权限只与基类中的访问说明符有关。
- 派生访问说明符的目的是控制派生类用户（包括派生类的派生类在内）对于基类成员的访问权限：
```c++
class Base {
public:
    void pub_mem();   // public member
protected:
    int prot_mem;     // protected member
private:
    char priv_mem;    // private member
};
struct Pub_Derv : public Base {
    // ok: derived classes can access protected members
    int f() { return prot_mem; }
    // error: private members are inaccessible to derived classes
    char g() { return priv_mem; }
};
struct Priv_Derv : private Base {
    // private derivation doesn't affect access in the derived class
    int f1() const { return prot_mem; }
};

Pub_Derv d1;   //  members inherited from Base are public
Priv_Derv d2;  //  members inherited from Base are private
d1.pub_mem();  //  ok: pub_mem is public in the derived class
d2.pub_mem();  //  error: pub_mem is private in the derived class
```

- 派生类向基类的转换是否可访问由使用该转换的代码决定，同时派生类的派生访问说明符也会有影响。
- 对于代码中的某个给定节点来说，如果基类的共有成员是可访问的，则派生类向基类的类型转换也是可访问的；反之则不行。

- 就像友元关系不能传递一样，友元关系同样也不能继承。
```c++
class Base {
    // added friend declaration; other members as before
    friend class Pal; // Pal has no access to classes derived from Base
};
class Pal {
public:
    int f(Base b) { return b.prot_mem; } // ok: Pal is a friend of Base
    int f2(Sneaky s) { return s.j; } // error: Pal not friend of Sneaky
    // access to a base class is controlled by the base class, even inside a derived object
    int f3(Sneaky s) { return s.prot_mem; } // ok: Pal is a friend
};
```

- 通过在类的内部使用`using`声明语句，可以将该类的直接或间接基类中的任何可访问成员标记出来，改变这些成员的可访问性。
```c++
class Base {
public:
    std::size_t size() const { return n; }
protected:
    std::size_t n;
};
class Derived : private Base {    //  note: private inheritance
public:
    // maintain access levels for members related to the size of the object
    using Base::size;
protected:
    using Base::n;
};
```

- 默认情况下，使用`class`关键字定义的派生类是私有继承的；而使用`struct`关键字定义的派生类是共有继承的。

## 继承中的类作用域

- 如果一个名字在派生类的作用域内无法正确解析，则编译器将继续在外层的基类作用域中寻找该名字的定义。

## 构造函数与拷贝控制

### 虚析构函数

- 通常如果一个类需要析构函数，那么它同样需要拷贝和赋值操作。但一个基类总是需要虚析构函数，若该析构函数为了成为虚函数而令内容为空，则无法由此推断该基类还需要赋值运算符或拷贝构造函数。

### 合成拷贝控制与继承

- 在实际中，如果在基类中没有默认、拷贝或移动构造函数，则一般情况下派生类也不对定义相应的操作。

### 派生类的拷贝控制成员

- 当派生类定义了拷贝或移动操作时，该操作负责拷贝或移动包括基类部分成员在内的整个对象。
- 在默认情况下，基类默认构造函数初始化派生类对象的基类部分。如果我们想拷贝（或移动）基类部分，则必须在派生类的构造函数初始值列表中显式地使用基类的拷贝（或移动）构造函数。
- 与拷贝和移动构造函数一样，派生类的赋值运算符也必须显式地为其基类部分赋值：
```c++
// Base::operator=(const Base&) is not invoked automatically
D &D::operator=(const D &rhs)
{
    Base::operator=(rhs); // assigns the base part
    // assign the members in the derived class, as usual,
    // handling self-assignment and freeing existing resources as appropriate
    return *this;
}
```

- 在析构函数体执行完成后，对象的成员会被隐式销毁；类似的，对象的基类部分也是隐式销毁的。因此，与构造函数及赋值运算符不同，派生类析构函数只负责由派生类自己分配的资源：
```c++
class D: public Base {
public:
    // Base::~Base invoked automatically
    ~D() { /* do what it takes to clean up derived members   */ }
};
```

- 对象销毁的顺序正好与其创建的顺序相反：派生类析构函数首先执行，然后是基类的析构函数，以此类推，沿着集继承体系的反方向直至最后。
- 如果构造函数或者析构函数调用了某个虚函数，则我们应该执行与构造函数或析构函数所属类型相对应的虚函数版本。

### 继承的构造函数

- 通常，`using`声明语句只是令某个名字在当前作用域内可见。而当作用于构造函数时，`using`声明语句将令编译器产生代码。

- 在以下`Bulk_quote`类中，继承的构造函数与以下函数等价：
```c++
class Bulk_quote : public Disc_quote {
public:
    using Disc_quote::Disc_quote; // inherit Disc_quote's constructors
    double net_price(std::size_t) const;
};

Bulk_quote(const std::string& book, double price,
          std::size_t qty, double disc):
      Disc_quote(book, price, qty, disc) { }
```

- 和普通成员的`using`声明不一样，一个构造函数的`using`声明不会改变该构造函数的访问级别。不管`using`声明出现在哪，派生类从基类继承的构造函数与基类中的构造函数具有相同的访问级别。
- 如果派生类定义的构造函数与基类的构造函数具有相同的参数列表，则该构造函数将不会被继承。定义在派生类中的构造函数将替换继承而来的构造函数。
- 默认、拷贝和移动构造函数不会被继承。

## 容器与继承

- 当派生类对象被赋值给基类对象时，其中派生类部分将被“切掉”。
- 当我们希望在容器中存放具有继承关系的对象时，我们通常存放的是基类的指针（更好的选择是智能指针）。

- 当我们令一个类公有地继承另一个类时，派生类与基类的关系是“是一种（Is A）”。在设计良好的类体系中，公有派生类的对象应该可以用在任何需要基类对象的地方。