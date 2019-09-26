---
title: Copy Control
date: 2019-09-23 15:42:57
categories:
- C++
tags:
- C++ Primer
- RE：从零开始的C++学习
- C++
---

# 拷贝控制

当我们定义一个类时，我们显式地或隐式地指定在此类型的对象拷贝、移动、赋值和销毁时做什么。一个类通过定义五种特殊的成员函数来控制这些操作，包括：**拷贝构造函数**（copy constructor）、**拷贝赋值运算符**（copy assignment operator）、**移动构造函数**（move constructor）、**移动赋值运算符**（move assignment operator）和**析构函数**（destructor）。

## 拷贝、赋值与销毁

### 拷贝构造函数

- 如果一个构造函数的第一个参数是对自身类型的引用，并且其他参数都有默认值，则此构造函数是拷贝构造函数：
```c++
class Foo {
public:
    Foo();            // default constructor
    Foo(const Foo&);  // copy constructor
    // ...
};
```

- 与合成默认构造函数不同，即使我们定义了其他构造函数，编译器也会为我们合成一个拷贝构造函数。
- 合成拷贝构造函数依次将每个非`static`成员拷贝到正在创建的对象中。每个成员的类型决定了它如何拷贝：对类类型的成员，使用其拷贝构造函数来拷贝；内置类型的成员则直接拷贝。
- 虽然我们不能直接拷贝一个数组，但合成拷贝构造函数会逐元素地拷贝一个数组类型的成员。如果数组元素是类类型，则使用元素的拷贝构造函数来进行拷贝。
- 拷贝初始化不仅在使用`=`定义变量时发生，在下列情况下也会发生：
  - 将一个对象作为实参传递给一个非引用类型的形参
  - 从一个返回类型为非引用类型的函数返回一个对象
  - 用花括号列表初始化一个数组中的元素或一个聚合类中的成员

### 拷贝赋值运算符

- 如果一个运算符是一个成员函数，其左侧运算对象就绑定到隐式的`this`参数。对于二元运算符，其右侧对象作为显式参数传递。

### 析构函数

- 由于析构函数没有参数，因此它不能被重载。对一个给定的类，只会有一个析构函数。
- 在析构函数中，首先执行函数体，然后销毁成员。成员按初始化的逆序销毁。
- 销毁类类型的成员需要执行成员自己的析构函数。内置类型没有析构函数，所以销毁内置类型成员什么都不需要做。

### 三/五法则

- 如果一个类需要析构函数，那么它几乎肯定也需要拷贝构造函数和拷贝赋值运算符。
- 如果一个类需要拷贝构造函数，那么它几乎一定需要拷贝赋值运算符，反之亦然。

### 使用`=default`

- 当在类内使用`=default`修饰成员的声明时，合成的函数将隐式地声明为内联的。

### 阻值拷贝

- `=delete`必须出现在函数第一次声明的时候，并且可以对任何函数指定`=delete`。

- 如果一个类成员的析构函数是删除的，则该成员无法被销毁；而如果一个成员无法被销毁，则该对象整体也无法被销毁。
- 如果一个类有数据成员不能默认构造、拷贝、复制或销毁，则类的合成拷贝控制成员将被定义为删除的。

## 拷贝控制和资源管理

### 行为像值的类

- 编写赋值运算符时，需要记住两点：
  - 赋值运算符必须在一个对象赋值给他本身时正确工作。
  - 大多数赋值运算组合了析构函数和构造函数的工作。

### 行为像指针的类

- 使用动态内存进行引用计数。

## 交换操作

- 如果存在类型特定的`swap`版本，其匹配优先程度会优于`std`中定义的版本。

## 动态内存管理

- 移动构造函数通常是将资源从给定对象“移动”而不是拷贝到正在创建的对象。
- 调用`move`的返回结果会令`construct`使用`string`的移动构造函数。
- 对一个对象调用`move`之后，我们无法知道该对象中保存的值，但是可以保证该对象的构造函数可以正确地执行。

## 对象移动

### 右值引用

- **右值引用**（rvalue reference）是必须绑定到右值的引用，通过`&&`而不是`&`来获得右值引用。
- 左值有持久的状态，而右值要么是字面常量，要么是在表达式求值过程中创建的临时变量。
- 变量是左值，因此我们不能将一个右值引用直接绑定到一个变量上，即使这个变量是右值引用类型也不行。

- `move`函数返回给定对象的右值引用。

- 应该直接调用`std::move`而不是`move`。

### 移动构造函数和移动赋值运算符

- 一旦资源完成移动，源对象必须不再指向被移动的资源——这些资源的所有权已经归属新创建的对象。

- 不会抛出异常的移动构造函数和移动赋值运算符必须标记为`noexcept`。
- 移动操作必须保证移动源对象仍是有效的；一般来说对象有效是指可以安全地为其赋予新值或者可以安全地使用而不依赖其当前值。例如，在移动一个`string`或者其他容器对象后，该移后源对象仍然是有效的，我们可以对它执行`empty`、`size`等操作，但结果无法得到保证。

- 如果一个类定义了自己的拷贝构造函数、拷贝赋值运算符或者析构函数，编译器就不会为它合成移动构造函数和移动赋值运算符了。如果一个类没有移动操作，通过正常的函数匹配，类会使用对应的拷贝操作来代替移动操作。
- 只有当一个类没有定义任任何自己版本的拷贝控制成员，且它的所有数据成员都可以移动构造或移动赋值时，编译器才会为它合成移动构造函数或移动赋值运算符。
- “何时将合成的移动操作定义为删除的函数”遵循与“定义删除的合成拷贝操作“类似的原则。
  - 与拷贝构造函数不同的，移动构造函数被定义为删除函数的条件是：有类成员定义了自己的拷贝构造函数且未定义移动构造函数，或是有类成员未定义自己的拷贝构造函数且编译器不能为其合成移动构造函数。移动赋值运算符的情况类似。
  - 其他条件与拷贝操作类似。
- 定义了一个移动构造函数或者移动赋值运算符的类必须也定义自己的拷贝操作，否则这些成员默认地被定义为删除的。
- 如果一个类既有移动构造函数，也有拷贝构造函数，编译器将使用普通的函数匹配规则来确定使用哪个构造函数。
- 如果一个类没有移动构造函数，函数匹配保证该类型的对象会被拷贝，即使调用了`move`也是如此：
```c++
class Foo {
public:
    Foo() = default;
    Foo(const Foo&); // copy constructor
    // other members, but Foo does not define a move constructor
};
Foo x;
Foo y(x);            // copy constructor; x is an lvalue
Foo z(std::move(x)); // copy constructor, because there is no move constructor
```

- 对**移动迭代器**（move iterator）进行解引用得到的是右值引用。
```c++
void StrVec::reallocate()
{
    // allocate space for twice as many elements as the current size
    auto newcapacity = size() ? 2 * size() : 1;
    auto first = alloc.allocate(newcapacity);
    // move the elements
    auto last = uninitialized_copy(make_move_iterator(begin()), 
                                   make_move_iterator(end()), first);
    free(); // free the old space
    elements = first; // update the pointers
    first_free = last;
    cap = elements + newcapacity;
}
```

### 右值引用与成员函数

- 区分移动和拷贝的重载函数通常有一个版本接受一个`const T&`，而另一个版本接受一个`T&&`。

- 通常，无论对象是左值还是右值，我们都可以调用它的成员函数：
```c++
string s1 = "a value", s2 = "another";
auto n = (s1 + s2).find('a');
```

- 指定`this`的左值/右值属性的方式与定义`const`成员函数相同，即在参数列表后放置一个**引用限定符**（reference qualifier）：
```c++
class Foo {
public:
    Foo &operator=(const Foo&) &; // may assign only to modifiable lvalues
    // other members of Foo
};
Foo &Foo::operator=(const Foo &rhs) &
{
    // do whatever is needed to assign rhs to this object
    return *this;
}
```

- 引用限定符可以是`&`或者`&&`，分别指出`this`指向一个左值或右值。
- 一个函数可以同时用`const`和引用限定，引用限定符必须跟随在`const`限定符之后：
```c++
class Foo {
public:
    Foo someMem() & const;    // error: const qualifier must come first
    Foo anotherMem() const &; // ok: const qualifier comes first
};
```

- 可以根据引用限定符对函数进行重载：
```c++
class Foo {
public:
    Foo sorted() &&; // may run on modifiable rvalues
    Foo sorted() const &; // may run on any kind of Foo
    // other members of Foo
private:
    vector<int> data;
};
// this object is an rvalue, so we can sort in place
Foo Foo::sorted() &&
{
    sort(data.begin(), data.end());
    return *this;
}
    // this object is either const or it is an lvalue; either way we can't sort in place
Foo Foo::sorted() const & {
    Foo ret(*this); // make a copy
    sort(ret.data.begin(), ret.data.end()); // sort the copy
    return ret; // return the copy
}
```

- 如果我们定义两个或两个以上具有相同名字和相同参数列表的成员函数，就必须对所有的函数都加上引用限定符，或者所有的都不加：
```c++
class Foo {
public:
    Foo sorted() &&;
    Foo sorted() const; // error: must have reference qualifier
    // Comp is type alias for the function type (see § 6.7 (p. 249))
    // that can be used to compare int values
    using Comp = bool(const int&, const int&);
    Foo sorted(Comp*); // ok: different parameter list
    Foo sorted(Comp*) const; // ok: neither version is reference qualified
};
```

