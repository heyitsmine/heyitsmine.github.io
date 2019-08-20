---
title: Functions
date: 2019-08-19 21:50:40
categories:
- C++
tags:
- C++ Primer
- RE：从零开始的C++学习
- C++
---

# 函数基础

- 实参（argument）是函数形参（parameter）的初始值。
- 虽然实参与形参存在对应关系，但C++对实参的求值顺序没有保证。
- 形参名是可选的，某些情况下函数的某些形参是未被用到的，此类形参通常不命名以表示在函数体内不会使用它。函数调用必须为每个形参提供实参，即使这个形参不会被用到。
- 函数的返回值不能为数组或函数类型，但可以是指向数组或函数的指针。

## 局部对象

- 只存在于块执行期间的对象为**自动对象**（automatic object）。
- 对于局部变量对应的自动对象，有两种情况：如果变量定义本身含有初始值，就用这个初始值进行初始化；否则执行默认初始化。
- **局部静态对象**（local static object）在程序的执行路径第一次经过对象定义语句时初始化，并在程序结束时被销毁，在此期间对象所在函数执行结束不会对它造成影响。
- 如果局部静态对象没有显式的初始值，他将执行值初始化，这意味着内置类型的局部静态对象被初始化为0。

## 函数声明

- 由于函数声明没有函数体，可以在声明中省去形参名。
- 返回值类型、函数名和形参类型描述了函数的接口，并说明了调用函数需要的信息。函数声明也被称为**函数原型**（function prototype）。

## 分离式编译
- 为了允许编写程序时按照逻辑关系将其划分开，C++支持*分离式编译*。分类式编译允许我们将程序划分到几个文件中，每个文件可以独立编译。

- 为了生成可执行文件，必须告诉编译器如何找到所有的代码。假设`fact`函数的定义位于`fact.c`文件中，它的声明位于`Chapter6.h`头文件中。另外在`factMain.cc`文件中创建`main`函数，`main`函数将调用`fact`函数。对于上述几个文件，编译过程如下：
```shell
$ g++ factMain.cc fact.cc -o main
```

- 如果只修改了其中一个源文件，可以只重新编译修改的文件。大多数编译器都提供了分离式编译单个文件的机制，这个过程会产生`.obj`（Windows）或`.o`（Unix）文件，后缀名的含义是该文件包含*对象代码*（object code）。接下来编译器将对象文件*链接*在一起形成可执行文件。
```shell
$ g++ -c factMain.cc                # generates factMain.o
$ g++ -c fact.cc                    # generates fact.o
$ g++ factMain.o fact.o -o main     # generates main
```

# 参数传递

- 如果形参是引用类型，它将绑定到对应的实参上；否则，将实参的值拷贝后赋给形参。

## 传值参数

## 传引用参数

- 如果无需改变引用形参的值，最好将其声明为常量引用。

## `const`形参和实参

- 在C++中，我们可以定义名字相同的函数，但这些函数的形参列表必须有足够的区别。
```c++
void fcn(const int i) { /* fcn can read but not write to i */ }
void fcn(int i) { /* ... */ } //error: redefines fcn(int)
```

- 可用非`const`对象来初始化底层`const`对象，但反过来不行。一个普通的引用必须用相同类型的对象初始化。

## 数组形参

- 因为数组不能被拷贝，所以我们无法以值传递的方式使用数组参数。因为数组会被转换为指针，所以当我们为函数传递一个数组时，实际上传递的是指向数组首元素的指针。
- 以下声明是等价的：都声明了具有一个`const int*`类型形参的函数。
```c++
// despite apperances, these three declaration of print are equivalent
void print(const int*);
void print(const int[]);    // shows the intent that the function takes an array
void print(const int[10]);  // dimension for decumentation purposes (at best)
```

- 使用标记指定数组长度
```c++
void print(const char *cp)
{
    if (cp)                // if cp is not a null pointer
        while (*cp)        // so long as the character it points to is not a null character
            cout << *cp++; // print the character and advance the pointer
}
```

- 使用标准库规范
```c++
void print(const int *beg, const int *end)
{
    // print every element starting at beg up to but not including end
    while (beg != end)
        cout << *beg++ << endl; // print the current element
                                // and advance the pointer
}
```

- 显式传递一个表示数组大小的形参
```c++
// const int ia[] is equivalent to const int* ia
// size is passed explicitly and used to control access to elements of ia
void print(const int ia[], size_t size)
{
    for (size_t i = 0; i != size; ++i) {
        cout << ia[i] << endl;
    }
}
```

- 传递多维数组
```c++
// matrix points to the first element in an array whose elements are arrays of ten ints
void print(int (*matrix)[10], int rowSize) { /* . . . */ }
```
也可以使用数组的语法，此时编译器会忽略第一个维度：
```c++
// equivalent definition
void print(int matrix[][10], int rowSize) { /* . . . */ }
```

## `main`:处理命令行选项

## 含有可变形参的函数

- 如果函数形参的数量未知但它们的类型相同，可以使用`initializer_list`类型的形参。
```c++
void error_msg(initializer_list<string> il)
{
    for (auto beg = il.begin(); beg != il.end(); ++beg)
        cout << *beg << " " ;
    cout << endl;
}
```

# 返回类型和`return`语句

## 无返回值函数

- 返回值类型为`void`的函数可以返回另一个返回值为`void`的函数。

## 有返回值函数

- 函数调用是否为左值取决于函数的返回类型：调用一个返回引用的函数得到左值，其他返回类型得到右值。
- 列表初始化返回值：
```c++
vector<string> process()
{
    // . . .
    // expected and actual are strings
    if (expected.empty())
        return {};  // return an empty vector
    else if (expected == actual)
        return {"functionX", "okay"}; // return list-initialized vector
    else
        return {"functionX", expected, actual};
}
```
如果函数返回的是内置类型，则花括号包围的列表最多包含一个值，而且该值所占的空间不能大于目标类型的空间。

- 如果`main`函数的结尾没有`return`语句，编译器将隐式的插入一条返回`0`的`return`语句。

## 返回指向数组的指针

- 定义一个返回数组的指针最直接的方法是使用类型别名：
```c++
typedef int arrT[10];  // arrT is a synonym for the type array of ten ints
using arrtT = int[10]; // equivalent declaration of arrT
arrT* func(int i);     // func returns a pointer to an array of five ints
```
- 与以下声明相同，如果要定义一个返回数组指针的函数，则数组的维度必须跟在函数名之后。
```c++
int arr[10];          // arr is an array of ten ints
int *p1[10];          // p1 is an array of ten pointers
int (*p2)[10] = &arr; // p2 points to an array of ten ints
```
因此，返回数组指针的函数形式如下所示：

```c++
Type (*function(parameter_list))[dimension]
```

- 在C++11新标准中，可以使用**尾置返回类型**（trailing return type）简化函数声明。
```c++
// fcn takes an int argument and returns a pointer to an array of ten ints
auto func(int i) -> int(*)[10];
```

- `decltype`不会自动将数组转换为对应的指针类型：
```c++
int odd[] = {1,3,5,7,9};
int even[] = {0,2,4,6,8};
// returns a pointer to an array of five int elements
decltype(odd) *arrPtr(int i)
{
    return (i % 2) ? &odd : &even; // returns a pointer to the array
}
```

# 函数重载

- 对于重载函数来说，它们必须在形参数量或形参类型上有所不同。
- 一个拥有顶层`const`的形参无法和另一个没有顶层`const`的形参区分开。
```c++
Record lookup(Phone);
Record lookup(const Phone);   // redeclares Record lookup(Phone)
Record lookup(Phone*);
Record lookup(Phone* const);  // redeclares Record lookup(Phone*)
```

- 如果形参是某种类型的指针或引用，则通过区分其指向的是`const`对象还是非`const`对象可以实现函数重载，此时的`const`是底层的。

# 特殊用途语言特性

## 默认参数