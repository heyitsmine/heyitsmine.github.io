---
title: Generic Algorithm
date: 2019-09-06 19:10:35
categories:
- C++
tags:
- C++ Primer
- RE：从零开始的C++学习
- C++
---

# Lambda表达式

- 对于一个对象或表达式，如果可以对其使用调用运算符，则称它为可调用的。

- 除函数与函数指针外，还有两种可调用对象：重载了函数调用运算符的类以及**lambda表达式**（lambda expression）。lambda表达式的基本形式如下：
```c++
[capture list] (parameter list) -> return type  { function body }
```

- 与普通函数不同，lambda不能有默认参数。
- 如果lambda的函数体包含任何单一`return`语句之外的内容，且未指定返回类型，则返回`void`。
- 捕获列表只用于局部非`static`变量，lambda可以直接使用局部`static`变量和它所在函数之外声明的名字。
- 当定义一个lambda时，编译器生成一个与lambda对应的新的（未命名的）类类型。
- lambda捕获列表

<div align=center> {% asset_img 10-1.png %} </div>
- 如果函数返回一个lambda，则与函数不能返回一个局部变量的引用类似，此lambda也不能包含引用捕获。

# 参数绑定

- `functional`头文件中的`bind`函数可以接受一个可调用对象，生成一个新的可调用对象。调用`bind`的一般形式为：
```c++
auto newCallable = bind(callable, arg_list);
```

- `std::placeholder`命名空间中的`_n`是`bind`参数的占位符。对于`auto g = bind(f, a, b, _2, c, _1)`，调用`g(X, Y)`相当于调用`f(a, b, Y, c, X)`。

- 默认情况下，`bind`中的非占位符参数被拷贝到`bind`返回的可调用对象中。如果希望传递给`bind`一个对象而又不拷贝它，就必须使用标准库`ref`函数：
```c++
for_each(words.begin(), words.end(), bind(print, ref(os), _1, ' '));
```

# 再探迭代器

- 插入迭代器操作
<div align=center> {% asset_img 10-2.png %} </div>

- `istream_iterator`操作
<div align=center> {% asset_img 10-3.png %} </div>

- `ostream_iterator`操作
<div align=center> {% asset_img 10-4.png %} </div>

- 对于绑定到流的迭代器，一旦其关联的流遇到文件结束或IO错误，迭代器的值就与尾迭代器相等。
- 对于反向迭代器，可以使用`base()`成员函数返回其对应的普通迭代器，下图显示了普通迭代器与反向迭代器之间的关系。
<div align=center> {% asset_img 10-6.png %} </div>

- 当我们从一个普通迭代器初始化一个反向迭代器，或是给一个反向迭代器赋值时，结果迭代器与原迭代器指向的并不是相同的元素。
