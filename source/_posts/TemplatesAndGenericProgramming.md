---
title: Templates And Generic Programming
date: 2019-10-14 21:55:36
categories:
- C++
tags:
- C++ Primer
- RE：从零开始的C++学习
- C++
---

# 模板与泛型编程

## 定义模板

### 函数模板

- 可以定义一个通用的**函数模板**（function template），而不是为每个类型都定义一个新函数，来生成针对特定类型的新版本：
```c++
template <typename T>
int compare(const T &v1, const T &v2)
{
    if (v1 < v2) return -1;
    if (v2 < v1) return 1;
    return 0;
}
```

- 当调用一个函数模板时，编译器（通常）用函数实参来为我们推断模板实参，并使用推断出的模板参数**实例化**（instantiate）一个特定版本的函数。
```c++
// instantiates int compare(const int&, const int&)
cout << compare(1, 0) << endl;       // T is int
// instantiates int compare(const vector<int>&, const vector<int>&)
vector<int> vec1{1, 2, 3}, vec2{4, 5, 6};
cout << compare(vec1, vec2) << endl; // T is vector<int>
```

- 模板类型参数前必须使用关键字`class`或`typename`，并且在模板参数列表中，这两个关键字的含义相同，可以互换使用：
```c++
// error: must precede U with either typename or class
template <typename T, U> T calc(const T&, const U&);
// ok: no distinction between typename and class in a template parameter list
template <typename T, class U> calc (const T&, const U&);
```

- 除了定义类型参数，还可以在模板中定义**非类型参数**（nontype parameter）。一个非类型参数表示一个值而非一个类型。我们通过一个特定的类型名而非关键字`class`或`typename`来指定非类型参数。
- 当以一个模板被实例化时，非类型参数被一个用户提供的或编译器推断出的值所代替，这些值必须是常量表达式。对于以下函数模板，调用`compare("hi", "mom")`会实例化出`compare(const char (&p1)[3], const char (&p1)[4])`。
```c++
template<unsigned N, unsigned M>
int compare(const char (&p1)[N], const char (&p2)[M])
{
    return strcmp(p1, p2);
}
```

- 一个非类型参数可以是一个整型，或者是一个指向对象或函数类型的指针或（左值）引用。绑定到非类型整型参数的实参必须是一个常量表达式，绑定到指针或引用类型非类型参数的实参必须具有静态的生存期。
- 对于模板，为了生成一个实例化的版本，编译器需要掌握函数模板或类模板成员函数的定义。因此，模板的头文件通常既包括声明也包括定义。

### 类模板

- **类模板**（class template）是用来生成类的蓝图的，与函数模板不同的是，编译器不能为类模板推断模板函数类型，我们必须为类模板提供**显式模板实参**（explicit template）。
- 默认情况下，一个类模板的成员函数只有当程序用到他时才进行实例化。这个特性使得即使某种类型不能完全符合模板操作的要求，我们仍能用该类型实例化类。

- 当我们使用一个类模板类型时必须提供模板实参，但这规则有个例外。在类模板自己的作用域中，可以直接使用模板名而不提供实参，编译器将假定我们使用的类型与成员实例化所用的类型一致。
```c++
// BlobPtr throws an exception on attempts to access a nonexistent element
template <typename T> class BlobPtr
public:
    BlobPtr(): curr(0) { }
    BlobPtr(Blob<T> &a, size_t sz = 0):
            wptr(a.data), curr(sz) { }
    T& operator*() const
    { auto p = check(curr, "dereference past end");
      return (*p)[curr];  // (*p) is the vector to which this object points
    }
    // increment and decrement
    BlobPtr& operator++();        // prefix operators
    BlobPtr& operator--();
private:
    // check returns a shared_ptr to the vector if the check succeeds
    std::shared_ptr<std::vector<T>>
        check(std::size_t, const std::string&) const;
    // store a weak_ptr, which means the underlying vector might be destroyed
    std::weak_ptr<std::vector<T>> wptr;
    std::size_t curr;      // current position within the array
};
```

- 为了引用（类或函数）模板的一个特定实例，我们必须首先声明模板自身：
```c++
// forward declarations needed for friend declarations in Blob
template <typename> class BlobPtr;
template <typename> class Blob; // needed for parameters in operator==
template <typename T>
    bool operator==(const Blob<T>&, const Blob<T>&);
template <typename T> class Blob {
    // each instantiation of Blob grants access to the version of
    // BlobPtr and the equality operator instantiated with the same type
    friend class BlobPtr<T>;
    friend bool operator==<T>
           (const Blob<T>&, const Blob<T>&);
    // other members as in § 12.1.1 (p. 456)
};
```

- 为了让所有实例称为友元，友元声明中必须使用与类模板本身不同的模板参数。
```c++
// forward declaration necessary to befriend a specific instantiation of a template
template <typename T> class Pal;
class C {  //  C is an ordinary, nontemplate class
    friend class Pal<C>;  // Pal instantiated with class C is a friend to C
    // all instances of Pal2 are friends to C;
    // no forward declaration required when we befriend all instantiations
    template <typename T> friend class Pal2;
};
template <typename T> class C2 { // C2 is itself a class template
    // each instantiation of C2 has the same instance of Pal as a friend
    friend class Pal<T>;  // a template declaration for Pal must be in scope
    // all instances of Pal2 are friends of each instance of C2, prior declaration needed
    template <typename X> friend class Pal2;
    // Pal3 is a nontemplate class that is a friend of every instance of C2
    friend class Pal3;    // prior declaration for Pal3 not needed
};
```

- 在C++ 11标准中，可以将模板类型参数声明为友元：
```c++
template <typename Type> class Bar {
friend Type; // grants access to the type used to instantiate Bar
    //  ...
};
```

- C++ 11标准允许我们为模板类定义一个类型别名：
```c++
template<typename T> using twin = pair<T, T>;
twin<string> authors; // authors is a pair<string, string>
```

- 对于类模板的`static`成员，类模板的每个实例都有一个独有的`static`对象。因此，与定义模板的成员函数相似，`static`数据成员也需要定义为模板：
```c++
template <typename T> class Foo {
public:
   static std::size_t count() { return ctr; }
   // other interface members
private:
   static std::size_t ctr;
   // other implementation members
};

template <typename T>
size_t Foo<T>::ctr = 0; // define and initialize ctr
```

### 模板参数

- 默认情况下，C++语言假定通过作用域运算符访问的名字不是类型。因此，在使用一个模板类型参数的类型成员，就必须显式告诉编译器改名字是一个类型，我们通过使用关键字`typename`（而不是`class`）实现这一点：
```c++
template <typename T>
typename T::value_type top(const T& c)
{
    if (!c.empty())
        return c.back();
    else
        return typename T::value_type();
}
```

- 在C++11标准中，我们可以为函数模板和类模板提供**默认模板实参**（default template argument）：
```c++
// compare has a default template argument, less<T>
// and a default function argument, F()
template <typename T, typename F = less<T>>
int compare(const T &v1, const T &v2, F f = F())
{
    if (f(v1, v2)) return -1;
    if (f(v2, v1)) return 1;
    return 0;
}
```

### 成员模板

- 一个类（普通类或类模板）可以包含本身是模板的成员函数，这种成员被称为**成员模板**（member template），成员模板不能是虚函数。

- 在类模板外定义一个成员模板时，必须同时为类模板和成员模板提供模板参数列表。类模板的参数列表在前，后跟成员自己的模板参数列表：
```c++
template <typename T> class Blob {
    template <typename It> Blob(It b, It e);
    // ...
};

template <typename T>     // type parameter for the class
template <typename It>    // type parameter for the constructor
    Blob<T>::Blob(It b, It e):
              data(std::make_shared<std::vector<T>>(b, e)) {
}
```

### 控制实例化

- 在大系统中，在多个文件中实例化相同模板的额外开销可能非常严重，在C++ 11标准中，可以通过**显式实例化**（explicit instantiation）来避免这种开销。一个显式实例化有如下形式：
```c++
extern template declaration; // instantiation declaration
template declaration;        // instantiation definition
```

- 当编译器遇到`extern`模板声明时，它不会在本文件中生成实例化代码。将一个实例化声明为`extern`就表示承诺该程序在其他位置有该实例化的一个非`extern`声明（定义）。
- 一个类模板的实例化定义会实例化该模板的所有成员，包括内联的成员函数。

## 模板实参推断

- 从函数函数实参来确定模板实参的过程被称为**模板实参推断**（template argument deduction）。

### 类型转换与模板类型参数

- 只有有限的几种类型转换会自动地应用与实参，编译器通常不是对实参进行类型转换，而是生成一个新的模板实例。
- 与往常一样，顶层`const`无论在形参中还是实参中，都会被忽略。在其他类型转换中，能在调用中应用于函数模板的包括如下两项：
  - `const`转换
  - 指针或函数指针转换

### 函数模板显式实参

- 显式模板实参按由左至右的顺序与对应的模板参数匹配，只有尾部（最右）参数的显式模板实参才可以忽略，而且前提是它们可以从函数参数推断出来。
- 对于模板类型参数已经显式指定的实参，也会进行正常的类型转换：
```c++
long lng;
compare(lng, 1024);       // error: template parameters don't match
compare<long>(lng, 1024); // ok: instantiates compare(long, long)
compare<int>(lng, 1024);  // ok: instantiates compare(int, int)
```

### 尾置返回类型与类型转换

- 由于尾置返回出现在参数列表之后，它可以使用函数的参数：
```c++
// a trailing return lets us declare the return type after the parameter list is seen
template <typename It>
auto fcn(It beg, It end) -> decltype(*beg)
{
    // process the range
    return *beg;  // return a reference to an element from the range
}
```

- 为了获得元素类型，可以使用标准库的**类型转换**（type transformation）模板。可以使用`remove_reference`来获得元素类型：
```c++
template <typename It>
auto fcn2(It beg, It end) -> typename remove_reference<decltype(*beg)>::type
{
    // process the range
    return *beg;  // return a copy of an element from the range
}
```

### 函数指针和实参推断

- 当我们用一个函数模板初始化一个函数指针或为一个函数指针赋值时，编译器使用指针的类型来推断模板实参：
```c++
template <typename T> int compare(const T&, const T&);
// pf1 points to the instantiation int compare(const int&, const int&)
int (*pf1)(const int&, const int&) = compare;
```

### 模板实参推断和引用

- 从左值引用函数参数推断类型
```c++
template <typename T> void f1(T&);  // argument must be an lvalue
// calls to f1 use the referred-to type of the argument as the template parameter type
f1(i);   //  i is an int; template parameter T is int
f1(ci);  //  ci is a const int; template parameter T is const int
f1(5);   //  error: argument to a & parameter must be an lvalue

template <typename T> void f2(const T&); // can take an rvalue
// parameter in f2 is const &; const in the argument is irrelevant
// in each of these three calls, f2's function parameter is inferred as const int&
f2(i);  // i is an int; template parameter T is int
f2(ci); // ci is a const int, but template parameter T is int
f2(5);  // a const & parameter can be bound to an rvalue; T is int
```

- 当我们将一个左值传递给函数的右值引用参数，且此右值引用指向模板类型参数（如`T&&`）时，编译器推断模板类型参数为实参的左值引用类型。
- 通常我们不能（直接）定义一个引用的引用，但是通过类型别名或通过模板类型参数间接定义是可以的。如果我们间接创建一个引用的引用，则这些引用会形成折叠，即对于一个给定的类型X：
  - `X& &`、`X& &&`和`X&& &`都折叠成类型`X &`
  - `X&& &&`折叠成`X&&`

### 理解`std::move`

- `std::move`的定义：
```c++
template <typename T>
typename remove_reference<T>::type&& move(T&& t)
{
   return static_cast<typenameremove_reference<T>::type&&>(t);
}
```

- 虽然不能隐式地将一个左值转换为右值引用，但我们可以用`static_cast`显式的将一个左值转换为一个右值引用。

### 转发

- 通过将一个函数定义为一个指向模板类型参数的右值引用，我们可以保持其对应实参的所有类型信息。
- `forward`必须通过显式模板实参调用，`forward`返回该显式实参类型的右值引用。即，`forward<T>`的返回类型是`T&&`。
```c++
template <typename Type> intermediary(Type &&arg)
{
    finalFcn(std::forward<Type>(arg));
    // ...
}
```

## 重载与模板

- 当有多个重载模板对一个调用提供同样好的匹配时，应选择最特例化的版本。
- 对一个调用，如果一个非函数模板与一个函数模板提供同样好的匹配，则选择非模板版本。

## 可变参数模板

- 一个**可变参数模板**（variadic template）就是一个接受可变数目参数的模板函数或模板类。可变数目的参数被称为**参数包**（parameter packet）。存在两种参数包：**模板参数包**（template parameter packet），表示零个或多个模板参数；**函数参数包**（function parameter packet），表示零个或多个函数参数。
- 如果一个参数的类型是一个模板参数包，则此参数也是一个函数参数包：
```c++
// Args is a template parameter pack; rest is a function parameter pack
// Args represents zero or more template type parameters
// rest represents zero or more function parameters
template <typename T, typename... Args>
void foo(const T &t, const Args& ... rest);
```

- `sizeof ...`运算符返回参数包中元素个数：
```c++
template<typename ... Args> void g(Args ... args) {
    cout << sizeof...(Args) << endl;  // number of type parameters
    cout << sizeof...(args) << endl;  // number of function parameters
}
```

### 编写可变参数函数模板

- 可变参数函数通常是递归的：
```c++
// function to end the recursion and print the last element
// this function must be declared before the variadic version of print is defined
template<typename T>
ostream &print(ostream &os, const T &t)
{
    return os << t; // no separator after the last element in the pack
}
// this version of print will be called for all but the last element in the pack
template <typename T, typename... Args>
ostream &print(ostream &os, const T &t, const Args&... rest)
{
    os << t << ", ";           // print the first argument
    return print(os, rest...); // recursive call; print the other
arguments
}
```

- 非可变参数模板比可变参数模板更特例化。

### 包括展

- 对于一个参数包，除了可以获取其大小外，我们能对它做的唯一的事情就是**扩展**（expand）。当扩展一个包时，需要提供用于每个扩展元素的**模式**（pattern）。拓展一个包就是将它分解为构成的元素，对每个元素应用模式，获得扩展后的列表。通过在模式右边放一个省略号（...）来触发扩展操作。
```c++
template <typename T, typename... Args>
ostream &print(ostream &os, const T &t, const Args&... rest)// expand Args
{
    os << t << ", ";
    return print(os, rest...);                     // expand rest
}
```

## 模板特例化

- 一个特例化版本是模板的一个独立的定义，其中一个或多个模板参数被指定为特定的类型。
- 当我们特例化一个函数模板时，必须为原模板中每个模板参数都提供实参。为了指出我们正在实例化一个模板，应使用关键字`template`后跟空尖括号对`<>`。
```c++
// special version of compare to handle pointers to character arrays
template <>
int compare(const char* const &p1, const char* const &p2)
{
    return strcmp(p1, p2);
}
```

- 当我们定义一个特例化版本时，函数参数类型必须与一个先前声明的模板中对应的类型匹配。

- 特例化一个模板时，必须在原模板定义所在的命名空间中特例化它。

### 类模板部分特例化

标准库中的`remove_reference`类型是通过一系列的特例化版本实现其功能的：
```c++
// original, most general template
template <class T> struct remove_reference {
    typedef T type;
};
// partial specializations that will be used for lvalue and rvalue references
template <class T> struct remove_reference<T&>  // lvalue references
{ typedef T type; };
template <class T> struct remove_reference<T&&> // rvalue references
{ typedef T type; };
```

部分特例化的模板参数列表是原始模板的参数列表的一个子集或是一个特例化版本。