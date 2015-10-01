---
layout: post
title:  "Core C++: type deduction"
date:   2015-10-01 23:06:05
categories: C++
excerpt: c++ type decduction。

---
* content
{:toc}
---


#Core C++ series: type deduction（类型推导/演绎）

##1. 类型推导介绍
```C++
/*Ex1*/
template <typename E, int N>
void f(E(&)[N]); //形参类型ParamType = E(&)[N]
bool b[42];
f(b); // 实参类型ExprType = bool [42] 比较ParamType和ExprType, 推导出E=bool, N=42，ParamType中的引用修饰表明了推导的方法（后文详述）

/*Ex2*/
template<typename T1, typename T2, typename T3>
void g(T1 (T2::*)(T3*));// ParamType = T1 (T2::*)(T3*) 指向类T2成员函数的指针，其参数类型T3, 返回值类型T1
class S {
    public: void f(double*);
}
g(&S::f); // ExprType =(void)(S::*)(double*), 自顶向下比对各个构造，推导结果:T1=void, T2=S, T3=doule
```

上面两个例子是C++11以前就支持的类型推导，但C++11及以后的标准新增了许多需要类型推导的形式。

> C++98:  template type deduction(T&/T*, T)
> C++11:  template type deduction(T&&), auto object, decltype, lambda implicit return, lambda capture
> C++14:  auto return type, lambda auto parameter, decltype(auto), lambda init capture

##2. 类型推导的方法
###2.1 template type deduction
```C++
// 一般化的情况
template<typename T>
void f(ParamType param);
f(expr); // expr的类型是ExprType
```
type deduction 要做的就是对比ParamType 和ExprType，推导出模板参数T的类型，进而得出ParamType。但值得注意的地方在于，`原类型ExprType并不是直接拿来使用的，而是需要调整的，并且调整方式与ParamType的形式有关`。

```C++
// 这里的const属性，换成volatile，相应的结论同样成立
int val;
int& expr1 = val;
const vector<string> &  expr2;
const char* const expr3;
const char* const & expr4;
const char expr5[10];
f(string()); // expr6: string()是右值
```

通常ParamType有如下三种形式：
* ParamType is a reference or pointer, but not a universal refrence
* ParamType is a universal reference 
* ParamType is neither reference nor pointer
 
无论哪种形式的ParamType，ExprType的引用属性都要除去, 例如ExprType1的`int&`要调整成`int`。

####2.1.1 ParamType是引用或指针（非universal reference）
```C++
template<typename T>
void f(T& param);
f(expr);

// 调整原类型：
ExprType1 => int
ExprType2 => const vector<string>
ExprType3 => const char* const
ExprType4 => const char* const
ExprType5 => const char [10]
ExprType6 // failed to compile for non-const reference bind to a rvalue
 
// 模板参数的类型T = ExprType
// 如果是形如`void f(const T& param)`, 只是在推导T是去掉ExprType的top-level const修饰，因为ParamType中已经含有const修饰了，比如以上的`T3 = const char*`
```

这种情况下，会保留ExprType的const/volatile属性，也不会发生数组/函数到指针的转型。但指针的情况又稍微特殊点，比如原类型ExprType是`const char* const`，调整后会变成`const char*`，top-level的const修饰还是会被舍弃，请看下面的例子：

```C++
/*所指向对象的const属性保留，但top-level的const属性会去除*/
template<typename T>
void f(T* param);
f(expr);

// 调整原类型：
ExprType1 => // failed to compile for no matching function
ExprType2 => // failed to compile for no matching function
ExprType3 => const char*  // T = const char
ExprType4 => const char*  // T = const char
ExprType5 => const char*  // T = const char
ExprType6 // failed to deduction

// 模板参数的类型：T = (Exprtype 去掉指针符号)
```

####2.1.2 ParamType是universal reference
关于ExprType的调整方式与T&的调整一样，只不过需要额外考虑expr的左/右值属性：`如果expr是左值，则ParamType推导结果是左值引用，如果expr是右值，则ParamType推导结果是右值引用`。为了达到这个效果，需要考虑在推导T的时候额外加上引用属性，然后利用reference collapse完成推导。更多的细节可以参考[universal reference][1]/forward reference。

```C++
template <typename T>
void f(T&& param); // const T&&的形式不是universal reference
f(expr);

// 调整原类型：
ExprType1 => int
ExprType2 => const vector<string>
ExprType3 => const char* const
ExprType4 => const char* const
ExprType5 => const char [10]
ExprType6 => std::basic_string<char> 

// 模板参数的类型：T = ExprType & || T = ExprType ：
T1 => int & // expr是左值，则需额外加引用属性，即T = ExprType &
T2 => const vector<string>& // 同上
T3 => const char* const &   // 同上
T4 => const char* const &   // 同上
T5 => const char(&)[10]     // 同上
T6 => std::basic_string<char> // expr 是右值，无需额外加引用
```

####2.1.3  ParamType是传值的形式
这种形式的推导不仅会去除原类型ExprType的const/volatile修饰，在类型推导之前还会发生数组/函数到指针的转型。

```C++
template<typename T>
void f(T param);
f(expr); 

// 调整原类型：const属性去除，会发生数组/函数到指针的转型
ExprType1 => int
ExprType2 => vector<string>
ExprType3 => const char*
ExprType4 => const char*
ExprType5 => const char*
ExprType6 => std::basic_string<char>

// 模板参数的类型：T = ExprType
```

**小结**：如果形如T&/T&&，则尽可能保留表达式类型属性（const/volatile，数组维度等），而形如T的时候会尽可能去掉附加属性(const/volatile, 数组/函数的转型)，universal reference会考虑表达式的左/右值属性，来额外加引用修饰。建议参考[C++ Type Deduction and Why You Care CppCon2014][4]. auto type deduction与template type deduction的方法类似，也有三种形式：`auto, auto&, auto&&`，但两者还是存在少许的差别，比如template和auto遇上initializer_list时：

```C++
template<typename T>
void f(T param);
f({1, 2, 3, 4}); // error，初看起来推导出T=initializer_list<int>，但C++标准并不容许这么做。
 
auto x = {1, 2, 3, 4}; // auto 可以推导出x的类型为initializer_list<int>
auto x {1, 2, 3, 4}; // error
auto x {1}; // ok，x的类型是int

// 事实上，用initilizer_list初始化一个auto变量时，最好用拷贝构造的形式，它的类型推导结果都是initializer_list<T>，这也是Clang编译器建议使用的形式。
```

###2.2 decltype的类型推导

decltype顾名思义就是推导变量声明时的类型，比如`int& x = y;` `decltype(x)`就该为`int&`，保留了原类型的属性，来看看C++11标准的规定：
> 1) If the argument is an unparenthesized id-expression or an
> unparenthesized class member access, then decltype yields the type of
> the entity named by this expression. If there is no such entity, or if
> the argument names a set of overloaded functions, the program is
> ill-formed. 
2) If the argument is any other expression of type T, and
>     - a) if the value category of expression is xvalue, then decltype yields T&&;
>     - b) if the value category of expression is lvalue, then decltype yields T&;
>     - c) if the value category of expression is prvalue, then decltype yields T.

标准可以解释为优先看表达式是不是某个id（比如`x, classa.member_var`）, 如果是，则直接取声明时的类型，如果不是，则需要根据[lvalue, xvalue, pvalue][2]属性来调整类型。

```C++
//** use stantard item 1
int y = 1;
const int& x = y;
decltype(x) xType; // xType的类型是const int&

//** use stantard item 2-a
decltype(std::move(x)) xType; // xType的类型是const int&&
 
//** use stantard item 2-b
const vector<int> v(1);
decltype(v[0]) xType = 1; // operator[] 返回的是左值常量引用, xType是const int&
decltype((y)) yType; // "(y)"的类型是左值引用, yType的类型是int&, 注意decltype(y)的结果是int
 
//** use standard item 2-c
decltype(0) xType; // 0是pvalue, xType的类型是int
```

###2.3 函数返回值类型推导

```C++
auto lookupValue() // C++11需要加trialing return type
{
    static std::vector<int> values = {1, 2, 3, 4, 5};
    int idx = 3;
    return values[idx]; // lvalue reference
}
```
auto的类型推导与template by value形式的推导方式一致，原类型的reference/const属性要丢弃，此处函数返回值类型是int。当然，返回形式可以声明为`auto&, auto&&`，推导方式分别与template的推导方式一致。`T&&和auto &&`都能根据原对象的左/右值属性来调整推导结果是左/右值引用，但是原对象的T1 (T2::*)(T3*)引用属性却是丢失的，表现如下：

```C++
// auto用作函数返回值类型的缺点
template<class Fun, class... Args>
auto fun_wrapper(Fun fun, Args&&... args)
{
    return fun(std::forward<Args>(args)...);
}
```

如果`fun(...)`返回的`T&`，根据`auto`的推导规则，`fun_wrapper(...)`的返回值类型是T，不能达到完美转发的目的。如果声明成`auto& fun_wrapper(...)`，若`fun(...)`的返回值类型是T，`fun_wrapper(...)`的返回类型却是`T&`，依旧不能完美转发。
为了克服这个缺点，C++14 引入`decltype(auto)`，主要应用`decltype`的规则做推导，保证`fun_wrapper(...)`的返回类型与`fun(...)`一致。注意，C++标准只定义了`decltype(auto)`的规则，`decltype(auto&)`与`decltype(auto&&)`均是非法。

```C++
// decltype(auto)
template<class Fun, class... Args>
decltype(auto) fun_perfect_wrapper(Fun fun, Args&&... args)
{
    return fun(std::forward<Args>(args)...);
}

// 一个非法使用的例子
decltype(auto) lookupValue()
{
    static std::vector<int> values = {1, 2, 3, 4, 5};
    int idx = 1;
    auto x = values[idx]; // auto 推导的结果是int
    return (x);// "(x)"的类型是int&
}

// decltype(auto)推导的结果是int&
lookupValue() = 12; // 编译可以过，但会遇到运行期错误，对函数的局部变量x写操作
```

###2.4 lambda 表达式中的类型推导
lambda中变量捕获的类型推导更接近函数传参的方式，而不是template/auto的类型推导方式。需要注意的一点是变量捕获时会保留原类型的const/volatile修饰。

```C++
// C++11，用传参数的方式来考虑
const int cx = 0;
auto lambda_byval = [cx]() {}; // by value, 保留原类型的cont/volatile修饰
auto lambda_byref = [cx&]() {}; // by reference

// C++14
auto lambda_init = [cy = cx]() {}; // by value 初始化捕获，cy的类型是int，没有保留原来类型的const修饰
auto lambda_init_ref = [&ref = x](){};// by reference 初始化捕获，x不能是const，否则编译不过，另外捕获中不能有const& 写法。
auto lambda = [](auto x, auto y){return x+y; };//generic lambda，参数使用auto的类型推导
```

##3. 模板参数类型推导上下文
之前的内容都是在讨论单一(ParamType, ExprType)的推导，但实际中经常会遇到多对(ParamTypeN,  ExprTypeN)的推导，此外还会遇到类型无法参与推导的情况，这些都涉及到**推导上下文**。
###3.1 多对推导
-  每对(ParamType, ExprType)都分别进行推导，如果推导出的类型有冲突，则该推导失败
-  只有部分模板参数进行推导，其他模板参数使用已经指定/推导的类型，如果没有可使用的类型，则模板推导是失败的。

```C++
/*Ex1 结论矛盾*/
template <typename T> void f(T t1, T t2);
f(1, 1.2); // t1 推导出T = int，t2 推导出T = double，两者矛盾，推导失败

/*Ex2 结论矛盾*/
template <typename T>
T const& max(T const& a, T const& b);
// 如果max("Apple", "Pear"), 就会发生错误. “Apple”的原始类型是char[6]，而Pear的原始类型是char[5], 形参中有引用修饰，不能发生数组到指针转型(decay)，(A1,P1)(A2,P2)两者类型推导的结果是冲突的，该推导失败

/*Ex3 使用已推导的结果*/
template <int N>
class X{
    public:
        typedef int I;
        void f (int) {}
};

template<int N>
void f_wrap(void (X<N>::*p)(typename X<N>::I )) {} // X<N>::I 限定符左边部分的不参与推导，

int main(){
f_wrap(&X<33>::f); // ok，X<N>::*p 推导出N=33，然后带入到X<N>::I中
}
```

### 3.2 推导的条件

本小节给出部分无法推导的上下文，更多的规则，参考[Template argument deduction: Non-deduced contexts][3]

```C++
/*Ex1:*/
// 模板参数处于限定符::左边，无法参与类型推导，是个无法推导的上下文
template <typename T> struct identity { typedef T type; };

template <typename T>
void bad(std::vector<T> x, T value = 1);

template <typename T>
void good(std::vector<T> x, typename identity<T>::type value = 1);

std::vector<std::complex<double>> x;

bad(x, 1.2); // error, ExrpType1和ExrpType2都参加类型推导，而ExrpType1推导的T=std::complex<double>，
             // ExrpType2推导的T=double，两者冲突，类型推导失败。
good(x, 1.2); // ok，ExrpType2处于non-deduced contexts，只ExrpType1有类型推导。

/*Ex2:*/
// 子表达式引用了模板参数
template<std::size_t N> void f(std::array<int, 2*N> a);
std::array<int, 10> a;
f(a); // error, std::array<int, 2*N>中的2*N属于std::array的非类型模板参数，其子表达式引用了f的模板参数N，违反了C++标准。直觉上看，我们可以推导出2*N=10，进一步推导出N=5，但很遗憾标准规定2*N属于不可推导的上下文。

/*Ex3*/
//如果形参类型不是引用/指针，会退化成指针，第一维度的信息丢失，无法参与推导的
template<int i> void f1(int a[10][i]);
template<int i> void f2(int a[i][20]);
template<int i> void f3(int (&a)[i][20]); // &修饰，会保持实参类型的数组类型不变（避免转成指针）

void g() {
  int v[10][20];
  f1(v);            // ok: i deduced to be 20
  f1<20>(v);        // ok
  f2(v);            // error: cannot deduce template-argument i
  f2<10>(v);        // ok
  f3(v);            // ok: i deduced to be 10
}

/*Ex4*/
template<typename T> void f(T=5, T=7);
void g(){
f(1); //ok: call f<int>(1, 7)
f(); // error, 默认参数不参与类型推导
f<int>(); // ok
}
```

这里也给出能够形成推导的上下文，但又比较特殊的例子：

```C++
/*Ex1 转型运算符模板*/
// user-defined conversion template based on return type
template<typename T>
using sid=T;

class S{
public:
    template<typename T, int N>
    operator sid<T(&)[N]> (); // // 不能单纯写成operator T(&)[N]()，编译不过
};

void f(int(&)[20]) {}
void g(S s) { cout << typeid(f(s)).name(); }
 
/*Ex2 子类到父类的转型*/
template <class T> struct B { };
template <class T> struct D : public B<T> {};
template <class T> void f(B<T>&){}

void f() {
    D<int> d;
    f(d); // ParamType是B<T>, ExprType是D<int>，推导出T=int，D<int>到B<int>是成立的
}
```

[1]: https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers
[2]: http://en.cppreference.com/w/cpp/language/value_category
[3]: http://en.cppreference.com/w/cpp/language/template_argument_deduction
[4]: http://www.aristeia.com/TalkNotes/C++TypeDeductionandWhyYouCareCppCon2014.pdf