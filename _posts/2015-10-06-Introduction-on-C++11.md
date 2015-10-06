---
layout: post
title:  "C++11: new feature on function"
date:   2015-10-06 12:13:05
categories: C++, C++11
excerpt: C++11 new feature on function

---

* content
{:toc}

---

##function declaration
`attr(optional) decl-specifier-seq(optional) declarator virt-specifier-seq(optional) function-body`
**attr**: `[[attr]] [[attr1, attr2, attr3(args)]] [[namespace::attr(args)]] alignas_specifier`
**decl-specifier-seq**:  return type
**declarator**:`noptr-declarator ( parameter-list ) cv(optional) ref(optional) except(optional) attr(optional) -> trailing`
**function-body**: `ctor-initializer(optional) compound-statement` or `function-try-block` or `= delete ;` or `= default ;`
Note: `attr` can be placed at begin of function declaration/defination, or just after funcation id which in `noptr-declarator`, or after `cvr specifier`.

###1.const & volatile & reference
>A non-static member function can be declared with a const, volatile, or const volatile qualifier;
>A non-static member function can be declared with either an lvalue ref-qualifier (the character & after the function name) or rvalue ref-qualifier (the character && after the function name);   
 
Overload resolution will choose right version based on the property of `this` e.g.: `const, non-const, glvalue, rvalue`, sometimes introduce `non-const` to `const` conversion.
 
``` C++
// const/volatile qualified
#include <vector>
struct Array {
   ...
    // const member function
    int operator[](int idx) const {
        return data[idx]; // transformed to (*this).data[idx];
    }
    // non-const member function
    int& operator[](int idx) {
        return data[idx]; // transformed to (*this).data[idx]
    }
};

Array a(10);
a[1]; // overload resolution select non-const version. If not const version, conver to const, then choose const version
const Array ca(10);
ca[1]; // overload resolution select const version. If not const version, compile error
    
// ref-qualified
#include <iostream>
struct S {
    void f() & { std::cout << "lvalue\n"; }
    void f() &&{ std::cout << "rvalue\n"; }
};
 
int main(){
    S s;
    s.f();            // prints "lvalue"
    std::move(s).f(); // prints "rvalue"
    S().f();          // prints "rvalue"
}
```
###2.exception
>either dynamic exception specification(deprecated) or noexcept specification(C++11)

**noexcept specifier syntax**: `noexcept,  noexcept(expression)`
**exception operator syntax**: `noexcept( expression )`,  performs a compile-time check that returns true if an expression is declared to not throw any exceptions.

```C++
// whether foo is declared noexcept depends on if the expression
// T() will throw any exceptions
template <class T>
	void foo() noexcept(noexcept(T())) {} // inner most noexcept is a operator
 
void bar() noexcept(true) {}
void baz() noexcept { throw 42; }  // noexcept is the same as noexcept(true)
 
int main() 
{
    foo<int>();  // noexcept(noexcept(int())) => noexcept(true), so this is fine
 
    bar();  // fine
    baz();  // compiles, but at runtime this calls std::terminate
}
```
>Note:
>1. The noexcept-specification is not a part of the function type;
>2. It cannot appear in a typedef or type alias declaration(using ...);
>3. `noexcept` is an improved version of throw(), which is deprecated in C++11. Unlike throw(), noexcept will not call std::unexpected and may or may not unwind the stack, which potentially allows the compiler to implement noexcept without the runtime overhead of throw().    
[More details][1].

###3.attribute
`attr` introduces implementation-defined attributes for types, objects, code, etc.  The GNU and IBM language's extensions syntax are ` __attribute__((...))`, Microsoft 's extension is  `__declspec()`.
>In declarations, attributes may appear both before and directly after the name of the entity that is declared, in which case they are combined.

```C++
// C++ standard
[[ noreturn ]] void f() {
throw "error"; // OK
}
[[ noreturn ]] void q(int i) {
// behavior is undefined if called with an argument <= 0
if (i > 0) throw "positive";
}
// void q(int) [[noreturn]]; // error/warning: 'noreturn' attribute cannot be applied to types (warning in gcc, error in Clang). It is a little different from cppreference's description about function declarator.

// GNU style 
extern void exit(int) __attribute__((noreturn));

// Clang: this applies the GNU unused attribute to a and f, and also applies the GNU noreturn attribute to f.
[[gnu::unused]] int a, f [[gnu::noreturn]] (); 
```
###4.trailing return type
Trailing return type is useful if the return type depends on argument names, such as `template <class T, class U> auto add(T t, U u) -> decltype(t + u); `or is complicated, such as in `auto fpif(int)->int(*)(int)`. From C++14, the trailing return type can be ignored.
###function specifier: explicit & override & final
`virtual` and `inline` have been used widely.  C++11 introduce `explicit`.The `explicit` specifier shall be used only in the declaration of a constructor or conversion function within its class definition. Specifies that this user-defined conversion function is only considered for direct initialization (including explicit conversions)[More details][2].

```C++
explicit class_name ( params )	 (1)	
explicit operator type ( ) (since C++11) (2)	
```
`override` and `final` are virtual function specifier. The syntax is **`declarator virt-specifier-seq(optional) pure-specifier(optional)`**, e.g. 

```C++
class A
{
  virtual auto fun[[noreturn]](int x) const volatile && noexcept -> decltype(x) final{}
};
```
`override` means this function must be a virtual function derived from base class, which has a compile-time check. `final` means this function cannot be overridden in its derived class. Moreover `final` can be used on class .

```C++
// override
struct A{
    virtual void foo();
    void bar();
};
 
struct B : A{
    void foo() const override; // Error: B::foo does not override A::foo
                               // (signature mismatch)
    void foo() override; // OK: B::foo overrides A::foo
    void bar() override; // Error: A::bar is not virtual
};

// final
struct A
{
    virtual void foo() final; // A::foo is final
    void bar() final; // Error: non-virtual function cannot be final
};
 
struct B final : A // struct B is final
{
    void foo(); // Error: foo cannot be overridden as it's final in A
};
 
struct C : B // Error: B is final
{ };
```

###5.function body: '=delete' & '= default'
`=delete` and `=default` are introduced since C++11, present function body. `=delete` explicitly delete function definition. The deleted definition of a function must be the first declaration in a translation unit: a previously-declared function cannot be redeclared as deleted.`=default` explicitly use defaulted function definition, only allowed for [special member functions][3].

```C++
struct sometype
{
    void* operator new(std::size_t) = delete;
    void* operator new[](std::size_t) = delete;
};
sometype* p = new sometype; // error, attempts to call deleted sometype::operator new
```

[1]: http://en.cppreference.com/w/cpp/language/noexcept_spec
[2]: http://en.cppreference.com/w/cpp/language/explicit
[3]: http://en.cppreference.com/w/cpp/language/member_functions#Special_member_functions
[4]: http://en.cppreference.com/w/cpp/language/attributes