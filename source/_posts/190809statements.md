---
title: Statements
date: 2019-08-09 23:10:11
categories:
- C++
tags:
- C++ Primer
- RE：从零开始的C++学习
- C++
---

# `switch`语句

- c++语言规定，不允许跨过变量的初始化语句直接跳转到该变量作用域内的另一个位置。
```c++
case true: // this switch statement is illegal because these initializations might be bypassed
    string file_name; // error: control bypasses an implicitly initialized variable
    int ival = 0;     // error: control bypasses an explicitly initialized variable
    int jval;         // ok: because jval is not initialized
    break;
case false:
    // ok: jval is in scope but is uninitialized
    jval = next_num(); // ok: assign a value to jval
    if (file_name.empty()) // file_name is in scope but wasn't initialized
        // ...
```

# 跳转语句

- 标签标识符独立于变量名和其他标识符。
- 与`switch`语句相同，`goto`无法跨过变量的初始化语句直接跳转到该变量作用域内的另一个位置。
- 跳转到一个变量创建前的位置会销毁该变量并重新创建。