---
layout: post
title:  "前端的一些资料和工具"
date:   2015-05-18 14:06:05
categories: Front-end
excerpt: 记录一些好用的前端工具和框架。
---

* content
{:toc}

---

#Core C++ series: type deduction（类型推导/演绎）



##1. 类型推导介绍

```C++

/*Ex1*/

template <typename E, int N>

void f(E(&)[N]);

bool b[42];

f(b); // P=E(&)[N], A=bool [42] 比较P和A, 推导出E=bool, N=42，P中的引用修饰表明了推导的方法（数组不会退化转型成指针）

 

/*Ex2*/

template<typename T1, typename T2, typename T3>

void g(T1 (T2::*)(T3*));// 指向类T2成员函数的指针，其参数类型T3, 返回值类型T1

class S {

    public: void f(double*);

}

g(&S::f); // P=T1 (T2::*)(T3*)，A=(void)(S::*)(double*), 自顶向下比对各个构造，推导结果:T1=void, T2=S, T3=doule

```

上面两个例子是C++11以前就支持的类型推导，但C++11及以后的标准新增了许多需要类型推导的形式。

> C++98:  template type deduction(T&/T*, T)

> C++11:  template type deduction(T&&), auto object, decltype, lambda implicit return, lambda capture

> C++14:  auto return type, lambda auto parameter, decltype(auto), l