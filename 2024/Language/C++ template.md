C++ template

[[toc]]



================
https://zh.cppreference.com/w/cpp/language/templates
# 模板

模板是一个 C++ 实体，可以分为：
- 一族类 ( 类模板)，可以是 嵌套类
- 一族函数 ( 函数模板)，可以是 成员函数
- 一族类型的别名 ( 别名模板) C++11起
- 一族变量 ( 变量模板) C++14起
- concept C++20起

模板以一个或多个模板形参参数化，形参有三种：类型模板形参、非类型模板形参和模板模板形参。 

当提供了模板实参，或当函数模板或类 (C++17 起)模板的模板实参被推导出时，它们替换各模板形参，以获得模板的一个特化（specialization），即一个特定类型或一个特定函数左值。
特化也可以显式提供：对类、变量 (C++14 起)和函数模板都允许全特化，只允许对类模板和变量模板 (C++14 起)部分特化。 

在要求完整对象类型的语境中引用某个类模板特化时，或在要求函数定义存在的语境中引用某个函数模板特化时，除非模板已经被显式特化或显式实例化，否则模板即被实例化（instantiate）（它的代码被实际编译）。类模板的实例化不会实例化其任何成员函数，除非它们也被使用。在连接时，不同翻译单元生成的相同实例被合并。

模板的定义必须在隐式实例化点可见，这就是模板库通常都在头文件中提供所有模板定义的原因（例如大多数 boost 库只有头文件）

## 语法
```
template <形参列表 > requires子句 (可选) 声明      (1) 	
export template <形参列表 > 声明                (2)(C++11 前)
template <形参列表 > concept 概念名 = 约束表达式 ;    (3)(C++20 起)
```

形参列表：非空的模板形参的逗号分隔列表，其中每项是非类型形参、类型形参、模板形参或任何这些的形参包之一。

requires子句：(C++20 起) 指定了各模板实参上的约束的 requires子句。

声明：类（包括 struct 和 union），成员类或成员枚举类型，函数或成员函数，命名空间作用域的静态数据成员，变量或类作用域的静态数据成员， (C++14 起)或别名模板 (C++11 起)的声明。它也可以定义模板特化。

概念名，约束表达式：见约束与概念

## 模板标识
`模板名 < 形参列表 >` 		

### 模板名
- 指名模板的标识符（这种情况下称之为 "简单模板标识"），或重载运算符模板或用户定义字面量模板的名字。
- 指名类模板特化的 简单模板标识 指名一个类。
- 指名别名模版特化的 模板标识 指名一个类型。
- 指名函数模板特化的 模板标识 指名一个函数。

模板标识 只有在符合下列条件时才合法：
- 实参数量不多于形参，或有一个形参是模板形参包，
- 每个没有默认模板实参的不可推导的非包形参都有一个实参，
- 每个模板实参都与对应的模板形参相匹配，
- 替换每个模板实参到其后续模板形参（如果存在）中均成功，而且 
- 如果 模板标识 非待决，那么它关联得约束需要按下述要求得以满足。 (C++20 起)

无效的 简单模板标识 是编译时错误，除非它指名的是函数模板特化（此时适用 ==SFINAE==）。 

。。6， S::type 是类型，所以是声明了一个变量 n ，然后赋值0。
``` C++
template<class T, T::type n = 0>
class X;
 
struct S
{
  using type = int;
};
 
using T1 = X<S, int, int>; // 错误：实参过多
using T2 = X<>;            // 错误：第一个模板形参没有默认实参
using T3 = X<1>;           // 错误：值 1 不匹配类型形参
using T4 = X<int>;         // 错误：第二个模板形参替换失败
using T5 = X<S>;           // OK
```


C++ 20 起：
如果在 简单模板标识 的模板名指名受约束的非函数模板或受约束的模板模板形参，但不是作为未知特化的成员的成员模板，而且 简单模板标识 中的所有模板实参均非待决，那么必须满足受约束模板的各项关联约束： 
``` C++
template<typename T>
concept C1 = sizeof(T) != sizeof(int);
 
template<C1 T> struct S1 {};
template<C1 T> using Ptr = T*;
 
S1<int>* p;                      // 错误：不满足约束
Ptr<int> p;                      // 错误：不满足约束
 
template<typename T>
struct S2 { Ptr<int> x; };       // 错误，不要求诊断
 
template<typename T>
struct S3 { Ptr<T> x; };         // OK：不要求满足
 
S3<int> x;                       // 错误：不满足约束
 
template<template<C1 T> class X>
struct S4
{
    X<int> x;                    // 错误，不要求诊断
};
 
template<typename T>
concept C2 = sizeof(T) == 1;
 
template<C2 T>
struct S {};
 
template struct S<char[2]>;      // 错误：不满足约束
template<> struct S<char[2]> {}; // 错误：不满足约束
```

两个 `模板标识` 在满足下列条件时相同：
-    它们的 模板名 指代同一模板，且
-    它们对应的类型模板实参是同一类型，且
-    它们对应的非类型模板实参在转换到模板形参的类型后模板实参等价，且
-    它们对应的模板模板实参指代同一模板。 
相同的两个 模板标识 指代同一个变量、 (C++14 起)类或函数。 


## 模板化实体
模板化实体（某些资料称之为 "temploid"）是在模板定义内定义（或对于 lambda 表达式 为创建） (C++11 起)的实体。下列所有实体都是模板化实体：
- 类/函数/变量 (C++14 起)模板 
- concept (C++20 起)
- 模板化实体的成员（例如类模板的非模板成员函数）
- 作为模板化实体的枚举的枚举项
- 任何模板化实体中定义或创建的实体：局部类，局部变量，友元函数，等等
- 模板化实体的声明中出现的 lambda 表达式的闭包类型 

在以下模板中：

```
template<typename T>
struct A
{
    void f() {}
};
```
函数 A::f 不是函数模板，但它仍然会被当做是模板化的。 

模板化函数 是函数模板或模板化的函数。
模板化类 是类模板或模板化的类。
模板化变量 是变量模板或模板化的变量。	(C++14 起)






================
https://zh.cppreference.com/w/cpp/language/template_parameters

# 模板形参
每个模板都会由一个或多个模板形参参数化，它们在模板声明语法中的 形参列表 中指定：
template < 形参列表 > 声明

形参列表 中的每个形参可以是：
-    模板非类型形参；
-    模板类型形参；
-    模板模板形参。

## 模板非类型形参
1. `类型 名字(可选)` 	(1) 	
2. `类型 名字(可选) = 默认值` 	(2) 	
3. `类型 ... 名字(可选)` 	(3) 	(C++11 起)
4. `占位符 名字` 	(4) 	(C++17 起)

1) 可以有名字的模板`非类型形参`。
2) 可以有名字和默认值的模板非类型形参。
3) 可以有名字的模板非类型形参包。
4) 带占位符类型的模板非类型形参。占位符 可以是包含占位符 auto 的任何类型（例如单纯的 auto、auto ** 或 auto &），被推导类类型的占位符 (C++20 起)，或者 decltype(auto)。

模板非类型形参必须拥有结构化类型，它是下列类型之一（可以有 cv 限定，忽略限定符）： 
-     （到对象或函数的）左值引用类型；
-    整数类型；
-    （指向对象或函数的）指针类型；
-    （指向成员对象或成员函数的）成员指针类型；
-    枚举类型； 

C++11起：
-    std::nullptr_t

C++20起：     
-    浮点类型；
-    拥有下列属性的字面类类型： 
	- 所有基类与非静态数据成员是公开且非 mutable 的，且
	- 所有基类与非静态数据成员的类型都是结构化类型或它的（可能多维的）数组。 

数组与函数类型可以写在模板声明中，但它们会被自动替换为适合的**对象指针和函数指针**。 
在类模板体内使用的模板非类型形参的名字是不可修改的纯右值，除非它的类型是左值引用类型`或类类型 (C++20 起)`。 


C++17起
如果模板`非类型形参`的`类型`包含占位符类型 auto，被推导类型的占位符 (C++20 起)，或 decltype(auto)，那么它可以被推导。推导会如同在虚设的声明 T x = 模板实参; 中推导变量 x 的类型一样进行，其中 T 是模板形参的声明类型。如果被推导的类型不能用于模板非类型形参，那么程序非良构。 

``` C++
template<auto n>
struct B { /* ... */ };
 
B<5> b1;   // OK：模板非类型形参的类型是 int
B<'a'> b2; // OK：模板非类型形参的类型是 char
B<2.5> b3; // 错误（C++20 前）：模板非类型形参的类型不能是 double
```

对于类型中使用了占位符类型的模板非类型形参包，每个模板实参的类型会独立进行推导，而且不需要互相匹配： 

```
template<auto...>
struct C {};
 
C<'C', 0, 2L, nullptr> x; // OK
```


C++20起
指名类类型 T 的某个模板非类型形参的标识符代表一个 const T 类型的静态存储期对象，该对象被称为 `模板形参对象` ，它的值是对应模板实参转换到模板形参的类型之后的值。程序中所有具有相同类型和值的这种模板形参都代表同一模板形参对象。模板形参对象应当拥有常量析构。 

```
struct A
{
    friend bool operator==(const A&, const A&) = default;
};
 
template<A a>
void f()
{
    &a;                       // OK
    const A& ra = a, &rb = a; // 都绑定到同一个模板形参对象
    assert(&ra == &rb);       // 通过
}
```


## 模板类型形参
```
类型形参关键词 名字(可选) 	(1) 	
类型形参关键词 名字(可选) = 默认值 	(2) 	
类型形参关键词 ... 名字(可选) 	(3) 	(C++11 起)
类型约束 名字(可选) 	(4) 	(C++20 起)
类型约束 名字(可选) = 默认值 	(5) 	(C++20 起)
类型约束 ... 名字(可选) 	 (6) 	(C++20 起)
```

类型形参关键词 	- 	typename 或 class 之一。这两个关键词在模板类型形参声明中没有区别
类型约束 	- 	概念的名字或概念名后随模板实参列表（在角括号中）。概念名均对于两者都可以有限定 

1) 没有默认类型的模板类型形参。 
```
template<class T>
class My_vector { /* ... */ };
```

2) 有默认类型的模板类型形参。 
```
template<class T = void>
struct My_op_functor { /* ... */ };
```

3) 模板类型形参包。 
```
template<typename... Ts>
class My_tuple { /* ... */ };
```

4) 没有默认实参的受约束模板类型形参。 
```
template <My_concept T>
class My_constrained_vector { /* ... */ };
```

5) 有默认实参的受约束模板类型形参。 
```
template <My_concept T = void>
class My_constrained_op_functor { /* ... */ };
```

6) 受约束的模板类型形参包。 
```
template <My_concept... Ts>
class My_constrained_tuple { /* ... */ };
```

形参的名字是可选的： 
```
// 对上面所示模板的声明：
template<class>
class My_vector;
template<class = void>
struct My_op_functor;
template<typename...>
class My_tuple;
```

在模板声明体内，类型形参的名字是 typedef 名字，它是当模板被实例化时所提供的类型的别名。 

C++20起
对于每个受约束形参 P，它的 类型约束 Q 指定了概念 C，那么根据以下规则引入一个约束表达式 E：
-    如果 Q 是 C（没有实参列表）， 
     -   如果 P 不是形参包，那么 E 是 C<P>
     -   否则，P 是形参包，那么 E 是折叠表达式 (C<P> && ...) 
-    如果 Q 是 C<A1,A2...,AN>，那么 E 分别是 C<P,A1,A2,...AN> 或 (C<P,A1,A2,...AN> && ...)。 

``` C++
template<typename T>
concept C1 = true;
template<typename... Ts>
concept C2 = true; // 变参概念
template<typename T, typename U>
concept C3 = true;
 
template<C1 T>         struct s1; // 约束表达式是 C1<T>
template<C1... T>      struct s2; // 约束表达式是 (C1<T> && ...)
template<C2... T>      struct s3; // 约束表达式是 (C2<T> && ...)
template<C3<int> T>    struct s4; // 约束表达式是 C3<T, int>
template<C3<int>... T> struct s5; // 约束表达式是 (C3<T, int> && ...)
```


## 模板模板形参

``` C++
template < 形参列表 > typename(C++17)|class 名字(可选) 	(1) 	
template < 形参列表 > typename(C++17)|class 名字(可选) = default 	(2) 	
template < 形参列表 > typename(C++17)|class ... 名字(可选) 	(3) 	(C++11 起)
```

类型形参关键词 	- 	class 或 typename 之一 (C++17 起)
	1) 可以有名字的模板模板形参。
	2) 有默认模板且可以有名字的模板模板形参。
	3) 可以有名字的模板模板形参包。


在模板声明体内，此形参的名字是一个模板名（且需要实参以实例化）。 
```
template<typename T>
class my_array {};
 
// 两个模板类型形参和一个模板模板形参：
template<typename K, typename V, template<typename> typename C = my_array>
class Map
{
    C<K> key;
    C<V> value;
};
```


## 模板形参的名字决议
模板形参的名字不能在它的作用域（包括内嵌作用域）内重声明。模板形参的名字不能与模板的名字相同。 

```
template<class T, int N>
class Y
{
    int T;      // 错误：重声明模板形参
    void f()
    {
        char T; // 错误：重声明模板形参
    }
};
 
template<class X>
class X; // 错误：重声明模板形参
```

在某个类模板定义外的出现的类模板成员定义中，类模板成员名会隐藏任何外围类模板的模板形参名，但如果该成员是类或函数模板就不会隐藏该成员的模板形参。 
。。。？

```
template<class T>
struct A
{
    struct B {};
    typedef void C;
    void f();
 
    template<class U>
    void g(U);
};
 
template<class B>
void A<B>::f()
{
    B b; // A 的 B ，不是模板形参
}
 
template<class B>
template<class C>
void A<B>::g(C)
{
    B b; // A 的 B ，不是模板形参
    C c; // 模板形参 C ，不是 A 的 C
}
```

在包含某个类模板的定义的命名空间外出现的该类模板的成员定义中，模板形参名隐藏此命名空间的成员名。 

```
namespace N
{
    class C {};
 
    template<class T>
    class B
    {
        void f(T);
    };
}
 
template<class C>
void N::B<C>::f(C)
{
    C b; // C 是模板形参，不是 N::C
}
```

在类模板定义中，或在模板定义外的这种成员的定义中出现，对于每个非待决基类，如果基类名或基类成员名与模板形参名相同，那么该基类名或成员名隐藏模板形参名。 

```
struct A
{
    struct B {};
    int C;
    int Y;
};
 
template<class B, class C>
struct X : A
{
    B b; // A 的 B
    C b; // 错误： A 的 C 不是类型名
};
```

## 模板实参

为使模板被实例化，它的每个模板形参（类型、非类型或模板）都必须被一个对应的模板实参替换。对于类模板，实参可以被显式提供，或从初始化器推导， (C++17 起)或为默认。对于函数模板，实参可以被显式提供，或从语境推导，或为默认。

如果实参可以同时被解释为类型标识和表达式，那么它始终会被解释为类型标识，即使它对应的是模板非类型形参： 

```
template<class T>
void f(); // #1
 
template<int I>
void f(); // #2
 
void g()
{
    f<int()>(); // "int()" 既是类型又是表达式，
                // 因为它被解释成类型，所以调用 #1
}
```

### 模板非类型实参
在实例化拥有模板非类型形参的模板时应用下列限制：
C++17前：
-     对于整型和算术类型，实例化时所提供的模板实参必须是模板形参类型的经转换常量表达式（因此适用某些隐式转换）。
-    对于对象指针，模板实参必须指定某个具有静态存储期和（内部或外部）连接的完整对象的地址，或者是求值为适当的空指针或 std::nullptr_t (C++11 起)值的常量表达式。
-    对于函数指针，合法的实参是指向具有连接的函数的指针（或求值为空指针值的常量表达式）。
-    对于左值引用形参，实例化时所提供的实参不能是临时量、无名左值或无连接的具名左值（换言之，实参必须具有连接）。
-    对于成员指针，实参必须是表示成 &Class::Member 的成员指针，或求值为空指针值或 std::nullptr_t (C++11 起)值的常量表达式。 

特别是，这意味着字符串字面量、数组元素的地址和非静态成员的地址，不能被用作模板实参，来实例化它对应的模板非类型形参是对象指针的模板形参的模板。 

C++17起：
模板非类型形参可以使用的模板实参，可以是该模板形参类型的任何经转换常量表达式。 

```
template<const int* pci>
struct X {};
 
int ai[10];
X<ai> xi; // OK：数组到指针转换和 cv 限定转换
 
struct Y {};
 
template<const Y& b>
struct Z {};
 
Y y;
Z<y> z;   // OK：没有转换
 
template<int (&pa)[5]>
struct W {};
 
int b[5];
W<b> w;   // OK：没有转换
 
void f(char);
void f(int);
 
template<void (*pf)(int)>
struct A {};
 
A<&f> a;  // OK：重载决议选择 f(int)
```

仅有的例外是引用或指针类型的模板非类型形参以及类类型的模板非类型形参和它的子对象之中的引用或指针类型的非静态数据成员 (C++20 起)，它们不能指代下列对象或者是下列对象的地址：
-    临时对象（包括在引用初始化期间创建的对象）；
-    字符串字面量；
-    typeid 的结果；
-    预定义变量 __func__；
-    或以上之一的 (C++20 起)子对象（包括非静态类成员、基类子对象或数组元素）。 

```
template<class T, const char* p>
class X {};
 
X<int, "Studebaker"> x1; // 错误：将字符串字面量用作模板实参
 
template<int* p>
class X {};
 
int a[10];
 
struct S
{
    int m;
    static int s;
} s;
 
X<&a[2]> x3; // 错误（C++20 前）：数组元素的地址
X<&s.m> x4;  // 错误（C++20 前）：非静态成员的地址
X<&s.s> x5;  // OK：静态成员的地址
X<&S::s> x6; // OK：静态成员的地址
 
template<const int& CRI>
struct B {};
 
B<1> b2;     // 错误：模板实参要求临时量
int c = 1;
B<c> b1;     // OK
```


### 模板类型实参
模板类型形参的模板实参必须是类型标识，它可以指名不完整类型： 

```
template<typename T>
class X {}; // 类模板
 
struct A;            // 不完整类型
typedef struct {} B; // 无名类型的类型别名
 
int main()
{
    X<A> x1;  // OK：'A' 指名类型
    X<A*> x2; // OK：'A*' 指名类型
    X<B> x3;  // OK：'B' 指名类型
}
```

### 模板模板实参
模板模板形参的模板实参是必须是一个 标识表达式，它指名一个类模板或模板别名。

当实参是类模板时，进行形参匹配时只考虑它的主模板。即使存在部分特化，它们也只会在基于此模板模板形参的特化恰好要被实例化时才会被考虑。 

```
template<typename T> // 主模板
class A { int x; };
 
template<typename T> // 部分特化
class A<T*> { long x; };
 
// 带有模板模板形参 V 的类模板
template<template<typename> class V>
class C
{
    V<int> y;  // 使用主模板
    V<int*> z; // 使用部分特化
};
 
C<A> c; // c.y.x 的类型是 int，c.z.x 的类型是 long
```

。。跳

### 默认模板实参

默认模板实参在形参列表中在 = 号之后指定。可以为任何种类的模板形参（类型、非类型或模板）指定默认实参，但不能对形参包指定 (C++11 起)。

如果为主类模板、主变量模板 (C++14 起)或别名模版的模板形参指定默认实参，那么它后面的所有模板形参都必须有默认实参，但最后一个可以是模板形参包 (C++11 起)。在函数模板中，对跟在默认实参之后的形参没有限制，而只有在类型形参具有默认实参，或可以从函数实参推导时，才能跟在形参包之后 (C++11 起)。

以下情况不允许默认形参： 
-    在类模板的成员的类外定义中（必须在类体内的声明中提供它们）。注意非模板类的成员模板可以在它的类外定义中使用默认形参（见 GCC 漏洞 53856）
-    在友元类模板声明中 
-    在任何函数模板声明或定义中 (C++11 前)
-    在友元函数模板的声明上，仅当声明是定义，且此翻译单元不出现此函数的其他声明时，才允许默认模板实参。(C++11 起)

各个声明中所出现的默认模板实参，以类似默认函数实参的方式合并： 
```
template<typename T1, typename T2 = int> class A;
template<typename T1 = int, typename T2> class A;
// 如上与如下相同：
template<typename T1 = int, typename T2 = int> class A;
```

但在同一作用域中不能两次为同一形参指定默认实参： 
```
template<typename T = int> class X;
template<typename T = int> class X {}; // 错误
```

模板模板形参的模板形参列表可拥有它自己的默认实参，它只会在模板模板实参自身处于作用域中时有效： 
```
// 类模板，带有默认实参的模板类型形参
template<typename T = float>
struct B {};
 
// 模板模板形参 T 有形参列表，
// 它由一个带默认实参的模板类型形参组成
template<template<typename = float> typename T>
struct A
{
    void f();
    void g();
};
 
// 类体外的成员函数模板定义
 
template<template<typename TT> class T>
void A<T>::f()
{
    T<> t; // 错误：TT 在作用域中没有默认实参
}
 
template<template<typename TT = char> class T>
void A<T>::g()
{
    T<> t; // OK：t 是 T<char>
}
```

默认模板形参中所用的名字的成员访问，在声明中，而非在使用点检查： 
```
class B {};
 
template<typename T>
class C
{
protected:
    typedef T TT;
};
 
template<typename U, typename V = typename U::TT>
class D: public U {};
 
D<C<B>>* d; // 错误：C::TT 是受保护的
```

C++14起
默认模板实参在需要该默认实参的值时被隐式实例化，除非模板用于指名函数： 
```
template<typename T, typename U = int>
struct S {};
 
S<bool>* p; // 默认模板实参 U 在此点实例化
            // p 的类型是 S<bool, int>*
```

### 模板实参等价性

模板实参等价性用来确定二个模板标识是否相同。
如果两个值拥有相同的类型，且满足以下条件之一，那么它们模板实参等价：
-    它们拥有整数或枚举类型且它们的值相同
-    或它们拥有指针类型且它们拥有同一指针值
-    或它们拥有成员指针类型且它们指代同一类成员或都是空成员指针值
-    或它们拥有左值引用类型且它们指代同一对象或函数 
C++11 起
-    或它们拥有 std::nullptr_t 类型 
C++20 起
-    或它们拥有浮点类型且它们的值相同
-    或它们拥有数组类型（此情况下数组必须是某类/联合体的成员对象）且它们对应的元素模板实参等价
-    或它们拥有联合体类型且它们均无活跃成员，或它们拥有相同的活跃成员且它们的活跃成员模板实参等价
-    或如果它们拥有非联合类类型且它们对应的直接子对象和引用成员模板实参等价 


### 示例

```
#include <iostream>
 
// 简单的模板非类型形参
template<int N>
struct S { int a[N]; };
 
template<const char*>
struct S2 {};
 
// 复杂的非类型形参的例子
template
<
    char c,             // 整型类型
    int (&ra)[5],       // 到（数组类型）对象的左值引用
    int (*pf)(int),     // 函数指针
    int (S<10>::*a)[10] // 指向（int[10] 类型的）成员对象的指针
>
struct Complicated
{
    // 调用编译时所选择的函数
    // 并在编译时将它的结果存储在数组中
    void foo(char base)
    {
        ra[4] = pf(c - base);
    }
};
 
//  S2<"fail"> s2;        // 错误：不能用字符串字面量
    char okay[] = "okay"; // 有连接的静态对象
//  S2< &okay[0] > s3;    // 错误：数组元素无连接
    S2<okay> s4;          // 能用
 
int a[5];
int f(int n) { return n; }
 
// C++20：模板非类型形参可以具有字面量类类型
template<std::array arr>
constexpr
auto sum() { return std::accumulate(arr.cbegin(), arr.cend(), 0); }
 
// C++20：可以在调用处推导出类模板实参
static_assert(sum<std::array<double, 8>{3, 1, 4, 1, 5, 9, 2, 6}>() == 31.0);
// C++20：模板非类型形参的实参推导和类模板实参推导
static_assert(sum<std::array{2, 7, 1, 8, 2, 8}>() == 28);
 
int main()
{
    S<10> s; // s.a 是含有 10 个 int 的数组
    s.a[9] = 4;
 
    Complicated<'2', a, f, &S<10>::a> c;
    c.foo('0');
 
    std::cout << s.a[9] << a[4] << '\n';
}
```









================
https://zh.cppreference.com/w/cpp/language/class_template

# 类模板

类模板定义一族类。 

## 语法
```
template < 形参列表 > 类声明 	(1) 	
export template < 形参列表 > 类声明 	(2) 	(C++11 前)
```

类声明 	- 	类声明。所声明的类名成为模板名。 
形参列表 	- 	非空的模板形参的逗号分隔列表，每项是非类型形参、类型形参、模板形参或任何这些形参的形参包之一。 

C++11前
export 是一个可选的修饰符，它用来表示模板被导出（用于声明类模板时，它也声明其所有成员被导出）。对被导出模板进行实例化的文件不需要包含其定义：有声明即已充分。export 的实现稀少，且在细节上相互不一致。 


## 类模板实例化
类模板自身并不是类型、对象或任何其他实体。不会从只包含模板定义的源文件生成任何代码。模板只有实例化才会有代码出现：必须提供模板实参，使得编译器能生成实际的类（或从函数模板生成函数）。 

### 显式实例化

```
template 类关键词 模板名 < 实参列表 > ; 	(1) 	
extern template 类关键词 模板名 < 实参列表 > ; 	(2) 	(C++11 起)
```

类关键词 	- 	class，struct 或 union
1) 显式实例化定义
2) 显式实例化声明

显式实例化定义强制实例化其所指代的 class、struct 或 union。它可以出现在程序中模板定义后的任何位置。而对于给定的实参列表，它在整个程序中只能出现一次，不要求诊断。 

 C++11起： 显式实例化声明（extern 模板）跳过隐式的实例化步骤：本来会导致隐式实例化的代码会以别处提供的显式实例化定义（如果这种实例化不存在，则导致连接错误）取代。这个机制可被用于减少编译时间：通过在除了一个文件以外的所有源文件中明确声明模板实例化，而在剩下的那个文件中明确定义它。

类、函数、变量 (C++14 起)和成员模板的特化可以从其模板显式实例化。成员函数、成员类和类模板的静态数据成员可以从其成员定义显式实例化。

显式实例化只能在该模板的外围命名空间出现，除非使用有限定标识： 

```
namespace N
{
    template<class T>
    class Y // 模板定义
    {
        void mf() {}
    };
}
 
// template class Y<int>; // 错误：类模板 Y 在全局命名空间不可见
using N::Y;
// template class Y<int>; // 错误：在模板的命名空间外进行显式实例化
template class N::Y<char*>;       // OK：显式实例化
template void N::Y<double>::mf(); // OK：显式实例化
```

如果同一组模板实参的**显式特化**在显式实例化之前出现，那么显式实例化没有效果。

当显式实例化函数模板、`变量模板 (C++14 起)`、类模板的成员函数或静态数据成员，或成员函数模板时，只需要它的声明可见。类模板、类模板的成员类或成员类模板在显式实例化之前必须出现完整定义，除非之前已经出现了拥有相同模板实参的显式特化。

如果以显式实例化定义来对函数模板、`变量模板 (C++14 起)`、成员函数模板或类模板的成员函数或静态数据成员进行显式实例化，那么同一翻译单元中必须存在它的模板定义。

当显式实例化指名某个类模板特化时，对于它的每个在当前翻译单元中尚未显式特化的非继承且非模板成员，它同时作为其同种显式实例化（声明或定义）。如果此显式实例化是定义，那么仅对于在此点已被定义的成员，它也是显式实例化定义。

显式实例化定义忽略成员访问说明符：形参类型和返回类型可以是 private。 


### 隐式实例化
当代码在要求完整定义的类型的语境中涉指某个模板时，或当类型的完整性对代码有影响，而这个特定类型尚未被显式实例化时，发生隐式实例化。例如当构造此类型的对象之时，但不包括构造指向此类型的指针之时。

这也适用于类模板的成员：除非成员在程序中被使用，否则并不会实例化它，并且不要求其有定义。 

```
template<class T>
struct Z // 模板定义
{
    void f() {}
    void g(); // 永远不会定义
};
 
template struct Z<double>; // 显式实例化 Z<double>
Z<int> a;                  // 隐式实例化 Z<int>
Z<char>* p;                // 此处不实例化任何内容
 
p->f(); // 隐式实例化 Z<char> 且 Z<char>::f() 在此出现。
        // 始终不需要且不实例化 Z<char>::g()：不必对其进行定义
```
。。template struct/class/union 显式实例化， 不写就是 隐式实例化

如果类模板已经声明但尚未定义，那么它的实例化会在实例化点产生不完整的类类型： 

```
template<class T>
class X;    // 声明，非定义
 
X<char> ch; // 错误：不完整类型 X<char>
```

C++17 起：局部类和它的成员中所使用的任何模板都会作为该局部类或枚举所处于的实体的实例化的一部分进行实例化。



================
https://zh.cppreference.com/w/cpp/language/function_template

# 函数模板
函数模板定义一族函数。 

语法
```
template < 形参列表 > 函数声明 	(1) 	
template < 形参列表 > requires 约束 函数声明 	(2) 	(C++20 起)
带占位符函数声明 	(3) 	(C++20 起)
export template < 形参列表 > 函数声明 	(4) 	(C++11 中移除)
```

形参列表 	
- 	非空的模板形参的逗号分隔列表，每项是非类型形参、类型形参、模板形参或这些的形参包之一。与任何模板一样，形参可以受约束。 (C++20 起)

函数声明 	
- 	函数声明。所声明的函数名成为模板名。

约束 	
- 	约束表达式，它限制此函数模板接受的模板形参。

带占位符函数声明 	
- 	函数声明，其中至少一个形参的类型使用了占位符 auto 或 概念 auto：每个占位符都会对应模板形参列表中的一个虚设形参。（见下文“简写函数模板”） 

。。概念auto： concept auto

C++20起：简写函数模板
当函数声明或函数模板声明的**形参**列表中**出现占位符类型**（auto 或 概念 auto）时，该声明会声明一个函数模板，并且为每个占位符向模板形参列表追加一个虚设的模板形参： 
``` C++
void f1(auto); // 与 template<class T> void f(T) 相同
void f2(C1 auto); // 如果 C1 是概念，那么与 template<C1 T> void f2(T) 相同
void f3(C2 auto...); // 如果 C2 是概念，那么与 template<C2... Ts> void f3(Ts...) 相同
void f4(const C3 auto*, C4 auto&); // 与 template<C3 T, C4 U> void f4(const T*, U&); 相同
 
template<class T, C U>
void g(T x, U y, C auto z); // 与 template<class T, C U, C W> void g(T x, U y, W z); 相同
```
简写函数模板可以和所有函数模板一样进行特化。 
```
template<>
void f4<int>(const int*, const double&); // f4<int, const double> 的特化
```

## 函数模板实例化
函数模板自身并不是类型、函数或任何其他实体。不会从只包含模板定义的源文件生成任何代码。模板只有实例化才会有代码出现：必须确定各模板实参，使得编译器能生成实际的函数（或从类模板生成类）。 

### 显式实例化
```
template 返回类型 名字 < 实参列表 > ( 形参列表 ) ; 	(1) 	
template 返回类型 名字 ( 形参列表 ) ; 	(2) 	
extern template 返回类型 名字 < 实参列表 > ( 形参列表 ) ; 	(3) 	(C++11 起)
extern template 返回类型 名字 ( 形参列表 ) ; 	(4) 	(C++11 起)
```
1) 显式实例化定义（显式指定所有无默认值模板形参时不会推导模板实参）
2) 显式实例化定义，对所有形参进行模板实参推导
3) 显式实例化声明（显式指定所有无默认值模板形参时不会推导模板实参）
4) 显式实例化声明，对所有形参进行模板实参推导

显式实例化定义强制实例化它所指代的函数或成员函数。它可以出现在程序中模板定义后的任何位置，而对于给定的实参列表，它在整个程序中只能出现一次，不要求诊断。 

C++11： 显式实例化声明（extern 模板）阻止隐式实例化：本来会导致隐式实例化的代码必须改为使用已在程序的别处所提供的显式实例化。 

在函数模板特化或成员函数模板特化的显式实例化中，尾部的各模板实参在能从函数参数推导时不需要指定： 
```
template<typename T>
void f(T s)
{
    std::cout << s << '\n';
}
 
template void f<double>(double); // 实例化 f<double>(double)
template void f<>(char);         // 实例化 f<char>(char)，推导出模板实参
template void f(int);            // 实例化 f<int>(int)，推导出模板实参
```

函数模板或类模板成员函数的显式实例化不能使用 inline 或 constexpr。如果显式实例化的声明指名了某个隐式声明的特殊成员函数，那么程序非良构。 

构造函数的显式实例化不能使用模板形参列表（语法 (1)），也始终不需要使用，因为能推导它们（语法 (2)）。 

C++20： 预期的析构函数的显式实例化必须指名该类的被选择的析构函数。

显式实例化声明不会抑制 inline 函数，auto 声明，引用，以及类模板特化的隐式实例化。（从而当作为显式实例化声明目标的 inline 函数被 ODR 式使用时，它会为内联而隐式实例化，但此翻译单元中不会生成它的非内联副本）

带有默认实参的函数模板的显式实例化定义不会使用或试图初始化该实参： 
```
char* p = 0;
 
template<class T>
T g(T x = &p) { return x; }
 
template int g<int>(int); // OK，即使 &p 不是 int。
```

### 隐式实例化
当代码在要求存在函数定义的语境中指涉某个函数，或定义存在与否会影响程序语义 (C++11 起)，而这个特定函数尚未被显式实例化时，发生隐式实例化。如果模板实参列表能从语境推导，那么不必提供它。 
```
template<typename T>
void f(T s)
{
    std::cout << s << '\n';
}
 
int main()
{
    f<double>(1); // 实例化并调用 f<double>(double)
    f<>('a');     // 实例化并调用 f<char>(char)
    f(7);         // 实例化并调用 f<int>(int)
    void (*pf)(std::string) = f; // 实例化 f<string>(string)
    pf("∇");                     // 调用 f<string>(string)
}
```

C++11： 如果有表达式需要某函数进行常量求值，那么函数定义存在与否会影响程序语义，即使不要求常量求值表达式，或常量表达式求值不使用该定义。 

```
template<typename T>
constexpr int f() { return T::value; }
 
template<bool B, typename T>
void g(decltype(B ? f<T>() : 0));
template<bool B, typename T>
void g(...);
 
template<bool B, typename T>
void h(decltype(int{B ? f<T>() : 0}));
template<bool B, typename T>
void h(...);
 
void x()
{
    g<false, int>(0); // OK： B ? f<T>() : 0 不会潜在常量求值
    h<false, int>(0); // 错误：即使 B 求值为 false 且
                      // 从 int 到 int 的列表初始化不可能是窄化仍实例化 f<int>
}
```

注意：完全省略 <> 允许重载决议同时检验模板与非模板重载。 


### 模板实参推导
实例化一个函数模板需要知道它的所有模板实参，但不需要指定每个模板实参。编译器会尽可能从函数实参推导缺失的模板实参。这会在尝试进行函数调用以及取函数模板的地址时发生。 
```
template<typename To, typename From>
To convert(From f);
 
void g(double d) 
{
    int i = convert<int>(d);    // 调用 convert<int,double>(double)
    char c = convert<char>(d);  // 调用 convert<char,double>(double)
    int(*ptr)(float) = convert; // 实例化 convert<int, float>(float)
}
```

模板运算符依赖此机制，因为除了将它重写为函数调用表达式之外，不存在为运算符指定模板实参的语法。 
```
int main() 
{
    std::cout << "Hello, world" << std::endl;
    // operator<< 经由 ADL 查找为 std::operator<<，
    // 然后推导出 operator<<<char, std::char_traits<char>>
    // 同时推导 std::endl 为 &std::endl<char, std::char_traits<char>>
}
```

模板实参推导在函数模板的名字查找（可能涉及实参依赖查找）之后，重载决议之前进行。
细节见模板实参推导。 

### 显式模板实参
函数模板的模板实参可从以下途径获得：
-    模板实参推导
-    默认模板实参
-    显式指定，可以在下列语境中进行： 
	-    在函数调用表达式中
	-    当取函数地址时
	-    当初始化到函数的引用时
	-    当构成成员函数指针时
	-    在显式特化中
	-    在显式实例化中
	-    在友元声明中 

不存在为**重载的运算符**、**转换函数**和**构造函数**显式指定模板实参的方法，因为它们**不会通过函数名调用。** 

所指定的各模板实参必须与各模板形参在种类上相匹配（即类型对类型，非类型对非类型，模板对模板）。实参的数量不能多于形参（除非形参中有形参包，这种情况下每个非包形参必须对应一个实参）。 

所指定的非类型实参必须要么与其对应的非类型模板形参的类型相匹配，要么可以转换到这些类型。 

不参与模板实参推导的函数实参（例如对应的模板实参已经被显式指定）会参与到它对应的函数形参的类型的隐式转换（如在通常重载决议中一样）。

当有额外的实参时，模板实参推导可以扩充显式指定的模板形参包： 

```
template<class... Types>
void f(Types... values);
 
void g()
{
    f<int*, float*>(0, 0, 0); // Types = {int*, float*, int}
}
```

### 模板实参替换
当已经指定、推导出或从默认模板实参获得了所有的模板实参之后，函数形参列表中对模板形参的每次使用都会被替换成对应的模板实参。

函数模板在替换失败时（即以推导或提供的模板实参替换模板形参失败）会从重载集中移除。这样就有许多方式通过模板元编程来操作重载集：细节见 **SFINAE**。

替换之后，所有数组和函数类型的函数形参都被调整为指针，且所有函数形参都会移除顶层 cv 限定（如在**常规函数声明**中一样）。
移除顶层 cv 限定并不会影响形参在函数内展现的类型： 

```
template<class T>
void f(T t);
 
template<class X>
void g(const X x);
 
template<class Z>
void h(Z z, Z* zp);
 
// 两个不同的函数的类型相同，但 t 在这些函数中有不同的 cv 限定
f<int>(1);       // 函数类型是 void(int)，t 是 int
f<const int>(1); // 函数类型是 void(int)，t 是 const int
 
// 两个不同的函数的类型和 x 相同
// （指向这两个函数的指针不相等，且函数局部的静态变量可以拥有不同地址）
g<int>(1);       // 函数类型是 void(int)，x 是 const int
g<const int>(1); // 函数类型是 void(int)，x 是 const int
 
// 只移除顶层 cv 限定：
h<const int>(1, NULL); // 函数类型是 void(int, const int*) 
                       // z 是 const int，zp 是 int*
```

### 函数模板重载
函数模板与非模板函数可以重载。

非模板函数与具有相同类型的模板特化始终不同。即使具有相同类型，不同函数模板的特化也始终互不相同。两个具有相同返回类型和相同形参列表的函数模板是不同的，而且可以用显式模板实参列表进行区分。

当使用了类型或非类型模板形参的表达式在函数形参列表或返回类型中出现时，该表达式会为了重载而保留为函数模板签名的一部分： 
```
template<int I, int J>
A<I+J> f(A<I>, A<J>); // 重载 #1
 
template<int K, int L>
A<K+L> f(A<K>, A<L>); // 同 #1
 
template<int I, int J>
A<I-J> f(A<I>, A<J>); // 重载 #2
```

对于两个涉及模板形参的表达式，如果两个包含这些表达式的函数定义根据 单一定义规则相同，那么称它们等价，就是说，除了模板形参的命名可以不同之外，这两个表达式含有相同的记号序列，其中的各个名字通过名字查找都解析到相同的实体。两个 lambda 表达式始终不等价。 (C++20 起)
```
template<int I, int J>
void f(A<I+J>);  // 模板重载 #1
 
template<int K, int L>
void f(A<K+L>);  // 等价于 #1
```

在确定两个待决表达式是否等价时，只考虑其中所涉及的各待决名，而不考虑名字查找的结果。如果相同模板的多个声明在名字查找的结果上有所不同，那么使用它们中的第一个： 
```
template<class T>
decltype(g(T())) h(); // decltype(g(T())) 是待决类型
 
int g(int);
 
template<class T>
decltype(g(T())) h()
{                  // h() 的再声明使用较早的查找，
    return g(T()); // 尽管此处的查找找到了 g(int)
}
 
int i = h<int>(); // 模板实参替换失败；g(int) 不在 h() 的首个声明处的作用域中
```

当满足下列条件时，认为两个函数模板等价：
-    它们在同一作用域声明
-    它们具有相同的名字
-    它们拥有等价的模板形参列表，意思是列表长度相同，且每对对应的形参均满足下列条件： 
	-     两个形参的种类相同（都是类型、都是非类型或都是模板），
	-     它们都是形参包或都不是，
	-     如果是非类型，那么它们的类型等价，
	-     如果是模板，那么它们的模板形参等价， 
	-     如果有一个声明带概念名，那么另一个也有等价的概念名。(C++20 起)
-    它们的返回类型和形参列表中所有涉及模板实参的表达式均等价 
-    模板形参列表之后的 requires 子句（如果存在）中的各个表达式均等价 (C++20 起)
-    函数声明符之后的 requires 子句（如果存在）中的各个表达式均等价 (C++20 起)

对于两个涉及模板形参的潜在求值 (C++20 起)表达式，如果它们不等价但它们对于任何给定的模板实参集的求值都产生相同的值，那么称它们功能等价。

如果两个函数模板本来可以等价，但它们的返回类型和形参列表中一或多个涉及模板形参的表达式功能等价，那么称它们功能等价。

另外，如果为两个函数模板指定的约束不同，但它们接受且被相同的模板实参列表的集合所满足，那么它们功能等价但不等价。(C++20 起)

如果程序含有功能等价但不等价的函数模板声明，那么程序非良构；不要求诊断。 

```
// 等价
template<int I>
void f(A<I>, A<I+10>); // 重载 #1
template<int I>
void f(A<I>, A<I+10>); // 重载 #1 的再声明
 
// 不等价
template<int I>
void f(A<I>, A<I+10>); // 重载 #1
template<int I>
void f(A<I>, A<I+11>); // 重载 #2
 
// 功能等价但不等价
// 程序非良构，不要求诊断
template<int I>
void f(A<I>, A<I+10>);      // 重载 #1
template<int I>
void f(A<I>, A<I+1+2+3+4>); // 功能等价
```

当同一个函数模板特化与多于一个重载的函数模板相匹配时（这通常由模板实参推导所导致），执行重载函数模板的**偏序处理**以选择最佳匹配。

具体而言，在以下情形中发生偏序处理： 
1) 对函数模板特化的调用的重载决议： 
```
template<class X>
void f(X a);
template<class X>
void f(X* a);
 
int* p;
f(p);
```
2) 取函数模板特化的地址时： 
```
template<class X>
void f(X a);
template<class X>
void f(X* a);
 
void (*p)(int*) = &f;
```
3) 选择作为函数模板特化的布置 operator delete 以匹配布置 operator new 时： 
4) 当友元函数声明、显式实例化或显式特化指代函数模板特化时： 
```
template<class X>
void f(X a);  // 第一个模板 f
template<class X>
void f(X* a); // 第二个模板 f
 
template<> void f<>(int *a) {} // 显式特化
 
// 模板实参推导出现两个候选：
// f<int*>(int*) 与 f<int>(int*)
// 偏序选择 f<int>(int*)，因为它更特殊
```

非正式而言，“A 比 B 更特殊”意味着“A 比 B 接受更少的类型”。

正式而言，为确定任意两个函数模板中哪个更特殊，偏序处理首先对两个模板之一进行以下变换：
-    对于每个类型、非类型及模板形参，包括形参包，生成一个唯一的虚构类型、值或模板，并将其替换到模板的函数类型中
-    如果要比较的两个函数模板中只有一个是成员函数，且该函数模板是某个类 A 的非静态成员，那么向它的形参列表的开头插入一个新的形参。给定 cv 作为该函数模板的 cv 限定符和 `ref 作为该函数模板的引用限定符 (C++11 起)`，该形参的类型是 cv A&，`除非 ref 是 &&，或 ref 不存在且要比较的另一个模板的首个形参的类型是右值引用类型，此时新形参的类型是 cv A&& (C++11 起)`。这有助于对运算符的定序，它们是同时作为成员和非成员函数查找的： 

```
struct A {};
 
template<class T>
struct B
{
    template<class R>
    int operator*(R&); // #1
};
 
template<class T, class R>
int operator*(T&, R&); // #2
 
int main()
{
    A a;
    B<A> b;
    b * a; // 模板实参推导对于 int B<A>::operator*(R&) 给出 R=A 
           //           对于 int operator*(T&, R&)，T=B<A>，R=A
 
    // 为进行偏序处理，将成员 template B<A>::operator*
    // 变换成 template<class R> int operator*(B<A>&, R&);
 
    //     int operator*(   T&, R&)  T=B<A>, R=A
    // 与  int operator*(B<A>&, R&)  R=A 间的偏序
    // 选择 int operator*(B<A>&, A&) 为更特殊者
}
```

。。跳

### 函数重载 vs 函数特化
注意，只有非模板和主模板重载参与重载决议。特化并不是重载，因此不受考虑。只有在重载决议选择最佳匹配的主函数模板后，才检验它的特化以查看最佳匹配者。 
```
template<class T>
void f(T);    // #1：所有类型的重载
template<>
void f(int*); // #2：#1 的特化，针对指向 int 的指针
template<class T>
void f(T*);   // #3：所有指针类型的重载
 
f(new int(1)); // 调用 #3，虽然 #1 的特化是完美匹配
```
在对翻译单元的头文件进行排序时，记住此规则很重要


### 函数模板特化








================
https://zh.cppreference.com/w/cpp/language/type_alias

# 类型别名，别名模板 (C++11 起)

类型别名是指代【先前定义的类型】的名字（与 typedef 类似）。
别名模版是指代一族类型的名字。 

语法
别名声明是具有下列语法的声明：
```
using 标识符 属性(可选) = 类型标识 ; 	(1) 	

template < 模板形参列表 >
using 标识符 属性(可选) = 类型标识 ;   (2)
``` 	

属性(C++11) 	- 	可选的任意数量属性的序列
标识符 	- 	此声明引入的名字，它成为一个类型名 (1) 或一个模板名 (2)
模板形参列表 	- 	模板形参列表，同模板声明
类型标识 	- 	抽象声明符或其他任何合法的 类型标识（可以引入新类型，如 类型标识 中所注明）。类型标识 不能直接或间接涉指 标识符。注意，标识符的声明点处于跟在 类型标识 之后的分号处。 


。。跳





================
https://zh.cppreference.com/w/cpp/language/variable_template

# 变量模板(C++14 起)

变量模板定义一族变量或静态数据成员。 

语法
```
template < 形参列表 > 变量声明
``` 

变量声明 	- 	变量的声明。声明的变量名成为模板名。
形参列表 	- 	非空的模板形参的逗号分隔列表，每项是非类型形参、类型形参、模板形参，或任何上述的形参包之一。 


从变量模板实例化的变量被称为 被实例化变量，从静态数据成员模板实例化的变量被称为 被实例化静态数据成员。

变量模板可以通过处于命名空间作用域中的模板声明引入，其中 变量声明 声明一个变量。 

```
template<class T>
constexpr T pi = T(3.1415926535897932385L); // 变量模板
 
template<class T>
T circular_area(T r) // 函数模板
{
    return pi<T> * r * r; // pi<T> 是变量模板实例化
}
```

在类作用域中使用时，变量模板声明一个静态数据成员模板。 
```
using namespace std::literals;
struct matrix_constants
{
    template<class T>
    using pauli = hermitian_matrix<T, 2>; // 别名模版
 
    template<class T> // 静态数据成员模板
    static constexpr pauli<T> sigmaX = {{0, 1}, { 1, 0 }}; 
 
    template<class T>
    static constexpr pauli<T> sigmaY = {{0, -1i}, {1i, 0}};
 
    template<class T>
    static constexpr pauli<T> sigmaZ = {{1, 0}, {0, -1}};
};
```

与其他静态成员一样，静态数据成员模板的需要一个定义。这种定义可以在类定义外提供。处于命名空间作用域的静态数据成员的模板声明也可以是类模板的非模板数据成员的定义： 

```
struct limits
{
    template<typename T>
    static const T min; // 静态数据成员模板的声明
};
 
template<typename T>
const T limits::min = {}; // 静态数据成员模板的定义
 
template<class T>
class X
{
    static T s; // 类模板的非模板静态数据成员的声明
};
 
template<class T>
T X<T>::s = 0; // 类模板的非模板静态数据成员的定义
```

除非变量模板被显式特化或显式实例化，否则在变量模板的特化在要求变量定义存在的语境中被引用，或定义存在与否影响程序语义时，即表达式（可能不使用定义）对常量求值需要该变量时，隐式实例化它。

如果有表达式需要某变量进行常量求值，那么变量定义存在与否会影响程序语义，即使不要求常量求值表达式，或常量表达式求值不使用该定义。 


注解
在 C++14 引入变量模板前，参数化变量通常实现为类模板的静态数据成员，或返回所需值的 constexpr 函数模板。

变量模板不能用作模板模板实参。 





================


================
https://zh.cppreference.com/w/cpp/language/parameter_pack

# 形参包 (C++11 起)

模板形参包是接受零个或更多个模板实参（非类型、类型或模板）的模板形参。函数形参包是接受零个或更多个函数实参的函数形参。

至少有一个形参包的模板被称作变参模板。 


语法
模板形参包（在别名模版、类模板、变量模板 (C++14 起)及函数模板形参列表中出现）
```
类型 ... 包名(可选) 	(1) 	
typename|class ... 包名(可选) 	(2) 	
类型约束 ... 包名(可选) 	(3) 	(C++20 起)
template < 形参列表 > class ... 包名(可选) 	(4) 	(C++17 前)
template < 形参列表 > typename|class ... 包名(可选) 	(4) 	(C++17 起)
```

函数参数包（声明符的一种形式，在变参函数模板的函数形参列表中出现）
```
包名 ... 包形参名(可选) 	(5)
```

形参包展开（在变参模板体中出现）
```
模式 ... 	(6)
```

1) 可以有名字的非类型模板形参包
2) 可以有名字的类型模板形参包
3) 可以有名字的受约束的类型模板形参包 (C++20 起)
4) 可以有名字的模板模板形参包
5) 可以有名字的函数形参包
6) 形参包展开：展开成零个或更多个 模式 的逗号分隔列表。模式必须包含至少一个形参包。


解释
变参类模板可以用**任意数量**的模板实参实例化： 
```
template<class... Types>
struct Tuple {};
 
Tuple<> t0;           // Types 不包含实参
Tuple<int> t1;        // Types 包含一个实参：int
Tuple<int, float> t2; // Types 包含两个实参：int 与 float
Tuple<0> error;       // 错误：0 不是类型
```

变参函数模板可以用任意数量的函数实参调用（模板实参通过模板实参推导推导）： 
```
template<class... Types>
void f(Types... args);

f();       // OK：args 不包含实参
f(1);      // OK：args 包含一个实参：int
f(2, 1.0); // OK：args 包含两个实参：int 与 double
```

在主类模板中，模板形参包必须是模板形参列表的最后一个形参。在函数模板中，模板参数包可以在列表中更早出现，只要其后的所有形参都可以从函数实参推导或拥有默认实参即可： 
```
template<typename U, typename... Ts>    // OK：能推导出 U
struct valid;
// template<typename... Ts, typename U> // 错误：Ts... 不在结尾
// struct Invalid;
 
template<typename... Ts, typename U, typename=void>
void valid(U, Ts...);    // OK：能推导出 U
// void valid(Ts..., U); // 不能使用：Ts... 在此位置是不推导语境
 
valid(1.0, 1, 2, 3);     // OK：推导出 U 是 double，Ts 是 {int, int, int}
```

如果变参模板的每个合法的特化都要求空模板形参包，那么程序非良构，不要求诊断。

## 包展开
后随省略号且其中至少有一个形参包的名字至少出现了一次的模式会被展开成零个或更多个逗号分隔的模式实例，其中形参包的名字按顺序被替换成包中的各个元素： 
```
template<class... Us>
void f(Us... pargs) {}
 
template<class... Ts>
void g(Ts... args)
{
    f(&args...); // “&args...” 是包展开
                 // “&args” 是它的模式
}
 
g(1, 0.2, "a"); // Ts... args 会展开成 int E1, double E2, const char* E3
                // &args... 会展开成 &E1, &E2, &E3
                // Us... 会展开成 int* E1, double* E2, const char** E3
```

如果两个形参包在同一模式中出现，那么它们同时展开而且长度必须相同： 
```
template<typename...>
struct Tuple {};
 
template<typename T1, typename T2>
struct Pair {};
 
template<class... Args1>
struct zip
{
    template<class... Args2>
    struct with
    {
        typedef Tuple<Pair<Args1, Args2>...> type;
        // Pair<Args1, Args2>... 是包展开
        // Pair<Args1, Args2> 是模式
    };
};
 
typedef zip<short, int>::with<unsigned short, unsigned>::type T1;
// Pair<Args1, Args2>... 会展开成
// Pair<short, unsigned short>, Pair<int, unsigned int> 
// T1 是 Tuple<Pair<short, unsigned short>, Pair<int, unsigned>>
 
typedef zip<short>::with<unsigned short, unsigned>::type T2;
// 错误：包展开中的形参包包含不同长度
```

如果包展开内嵌于另一个包展开中，那么它所展开的是在最内层包展开出现的形参包，并且在外围（而非最内层）的包展开中必须提及其它形参包： 
```
template<class... Args>
void g(Args... args)
{
    f(const_cast<const Args*>(&args)...); 
    // const_cast<const Args*>(&args) 是模式，它同时展开两个包（Args 与 args）
 
    f(h(args...) + args...); // 嵌套包展开：
    // 内层包展开是 “args...”，它首先展开
    // 外层包展开是 h(E1, E2, E3) + args 它其次被展开
    // （成为 h(E1, E2, E3) + E1, h(E1, E2, E3) + E2, h(E1, E2, E3) + E3）
}
```

## 展开场所
展开所产生的逗号分隔列表按发生展开的各个场所可以是不同种类的列表：函数形参列表，成员初始化器列表，属性列表，等等。以下列出了所有允许的语境。 

函数实参列表
。。里面内容略
有括号初始化器
花括号包围的初始化器
模板实参列表
函数形参列表
模板形参列表
基类说明符与成员初始化器列表
Lambda 捕获
sizeof... 运算符
动态异常说明
对齐说明符
属性列表
折叠表达式
using 声明


下面的例子定义了类似 std::printf 的函数，并以一个值替换格式字符串中字符 % 的每次出现。

首个重载在仅传递格式字符串且无形参展开时调用。

第二个重载中分别包含针对实参头的一个模板形参和一个形参包，这样就可以在递归调用中只传递形参的尾部，直到它变为空。

Targs 是模板形参包而 Fargs 是函数形参包。 

```
#include <iostream>
 
void tprintf(const char* format) // 基础函数
{
    std::cout << format;
}
 
template<typename T, typename... Targs>
void tprintf(const char* format, T value, Targs... Fargs) // 递归变参函数
{
    for (; *format != '\0'; format++)
    {
        if ( *format == '%' )
        {
            std::cout << value;
            tprintf(format + 1, Fargs...); // 递归调用
            return;
        }
        std::cout << *format;
    }
}
 
int main()
{
    tprintf("% world% %\n", "Hello", '!', 123);
    return 0;
}
```

================


================


================
https://zh.cppreference.com/w/cpp/language/sfinae

# SFINAE

“替换失败不是错误” (Substitution Failure Is Not An Error) 

在函数模板的重载决议中会应用此规则：**当模板形参在替换成显式指定的类型或推导出的类型失败时，从重载集中丢弃这个特化，而非导致编译失败**。

此特性被用于==模板元编程==。 

解释
对函数模板形参进行两次替换（由模板实参所替代）：
-    显式指定的模板实参在模板实参推导前替换
-    推导出的实参和从默认项获得的实参在模板实参推导后替换 

替换发生于
-    函数类型中使用的所有类型（包括返回类型和所有形参的类型）
-    各个模板形参声明中使用的所有类型 
-    函数类型中使用的所有表达式   C++11
-    各个模板形参声明中使用的所有表达式  C++11
-    explicit 说明符中使用的所有表达式  C++20

以上类型或表达式在以用来替换的实参写出时谬构（并带有必要的诊断）的场合是替换失败。

只有在函数类型或其模板形参类型或其 explicit 说明符 (C++20 起)的立即语境中的类型与表达式中的失败是 SFINAE 错误。如果对替换后的类型/表达式的求值导致副作用，例如实例化某模板特化、生成某隐式定义的成员函数等，那么这些副作用中的错误被当做硬错误。lambda 表达式不被当作是立即语境的一部分。 (C++20 起) 

替换以词法序进行，并在遇到失败时终止。 

如果多个声明具有不同的词法顺序（例如，某个函数模板声明为具有尾随返回类型，它在某个形参之后替换，然后被重声明为具有常规返回类型，它则在该形参之前替换），而这会导致模板实例化以不同顺序出现或完全不出现，那么程序谬构；不要求诊断。

```
template<typename A>
struct B { using type = typename A::type; };
 
template<
    class T,
    class U = typename T::type,    // 如果 T 没有成员 type 那么就是 SFINAE 失败
    class V = typename B<T>::type> // 如果 T 没有成员 type 那么就是硬错误
                                   // （经由 CWG 1227 保证不出现，
                                   // 因为到 U 的默认模板实参中的替换会首先失败）
void foo (int);
 
template<class T>
typename T::type h(typename B<T>::type);
 
template<class T>
auto h(typename B<T>::type) -> typename T::type; // 重声明
 
template<class T>
void h(...) {}
 
using R = decltype(h<int>(0));     // 谬构，不要求诊断
```

## 类型 SFINAE

下列类型的错误是 SFINAE 错误：
-    试图实例化含有多个不同长度的包的包展开 	(C++11 起)
-    试图创建 void 的数组，引用的数组，函数的数组，负大小的数组，非整型大小的数组，或者零大小的数组： 

```
template<int I>
void div(char(*)[I % 2 == 0] = 0)
{
    // 当 I 是偶数时选择这个重载
}
 
template<int I>
void div(char(*)[I % 2 == 1] = 0)
{
    // 当 I 是奇数时选择这个重载
}
```

- 试图在作用域解析运算符 :: 左侧使用类和枚举以外的类型： 
```
template<class T>
int f(typename T::B*);
 
template<class T>
int f(T);
 
int i = f<int>(0); // 使用第二个重载
```

- 试图使用类型的成员，其中 
        - 类型不包含指定成员
        - 在要求类型的地方，指定成员不是类型
        - 在要求模板的地方，指定成员不是模板
        - 在要求非类型的地方，指定成员不是非类型 


- 试图创建指向引用的指针
- 试图创建到 void 的引用
- 试图创建指向 T 的成员的指针，其中 T 不是类类型： 

- 试图将非法类型给予非类型模板形参： 
- 试图在以下语境中进行非法转换： 
	- 模板实参表达式
	- 函数声明中使用的表达式： 
- 试图创建形参类型为 void 的函数类型
- 试图创建返回数组类型或函数类型的函数类型 


## 表达式 SFINAE
C++11 前，只有在类型中使用的常量表达式（例如数组边界）才要求被当做 SFINAE（而非硬错误）。 

C++11起，下列表达式错误是 SFINAE 错误：
-    模板形参类型中使用的谬构表达式
-    函数类型中使用的谬构表达式 

```
struct X {};
struct Y { Y(X){} }; // X 可以转换到 Y
 
template<class T>
auto f(T t1, T t2) -> decltype(t1 + t2); // 重载 #1
 
X f(Y, Y);                               // 重载 #2
 
X x1, x2;
X x3 = f(x1, x2);  // 推导在 #1 上失败（表达式 x1+x2 为谬构）
                   // 重载集中只有 #2，调用它
```

## 部分特化中的 SFINAE
在确定一个类或变量 (C++14 起)模板的特化是由部分特化还是主模板生成的时候也会出现推导与替换。在这种确定期间，编译器不会把替换失败当作硬错误，反而像涉及函数模板的重载决议中一样忽略对应的部分特化声明。 


## 库支持
(C++11 起) 标准库组件 std::enable_if 允许创建替换失败，以基于某个在编译时求值的条件来启用或禁用特定的重载。
另外，许多类型特性在适合的编译器扩展不可用时必须以 SFINAE 实现。

(C++17 起) 标准库组件 std::void_t 是另一个简化部分特化 SFINAE 的应用的工具元函数。


## 替代方案
只要可用，标签派发、if constexpr (C++17 起)以及概念 (C++20 起)通常都比直接使用 SFINAE 更好。
。。Where applicable, tag dispatch, if constexpr (since C++17), and concepts (since C++20) are usually preferred over use of SFINAE. 
如果只想要条件性的编译时错误，static_assert 通常比 SFINAE 更适合。 (C++11 起)



================


================


================


================


================


================

https://zh.cppreference.com/w/cpp/language/constraints

# 约束与概念 (C++20 起)
。。Constraints and concepts

类模板，函数模板，以及非模板函数（通常是类模板的成员），可以关联到约束（constraint），它指定了对模板实参的一些要求，这些要求可以被用于选择最恰当的函数重载和模板特化。

这种要求的具名集合被称为概念（concept）。每个概念都是谓词，在编译时求值，并在自己被用作约束时成为模板接口的一部分： 

```
// 概念 "Hashable" 的声明可以被符合以下条件的任意类型 T 满足：
// 对于 T 类型的值 a，表达式 std::hash<T>{}(a) 可以编译并且它的结果可以转换到 std::size_t
template<typename T>
concept Hashable = requires(T a)
{
    { std::hash<T>{}(a) } -> std::convertible_to<std::size_t>;
};
 
struct meow {};
 
template<Hashable T>
void f(T); // 受约束的 C++20 函数模板
 
// 应用相同约束的另一种方式：
// template<typename T>
//     requires Hashable<T>
// void f(T); 
// 
// template<typename T>
// void f(T) requires Hashable<T>; 
 
int main()
{
    using std::operator""s;
 
    f("abc"s); // OK，std::string 满足 Hashable
    f(meow{}); // 错误：meow 不满足 Hashable
}
```

在编译时（模板实例化过程的早期）就检测是否违背约束，这样错误信息就更容易理解： 
```
std::list<int> l = {3,-1,10};
std::sort(l.begin(), l.end()); 
// 没有概念时典型编译器的诊断：
// 二元表达式的操作数非法 ('std::_List_iterator<int>' 和 'std::_List_iterator<int>')
//                           std::__lg(__last - __first) * 2);
//                                     ~~~~~~ ^ ~~~~~~~
// …… 50 行输出……
//
// 有概念时典型编译器的诊断：
// 错误：无法以 std::_List_iterator<int> 调用 std::sort
// 注意：未满足概念 RandomAccessIterator<std::_List_iterator<int>>
```

概念的目的是塑造==语义分类==（Number、Range、RegularFunction）而非语法上的限制（HasPlus、Array）。按照 ISO C++ 核心方针 T.20 所说，“与语法限制相反，真正的概念的一个决定性的特征是有能力指定有意义的语义。” 

## 概念
概念是要求的具名集合。概念的定义必须在命名空间作用域中出现。
概念定义拥有以下形式：
```
template < 模板形参列表 >
concept 概念名 属性(可选) = 约束表达式;
```
属性 	- 	任意数量的属性序列 

```
// 概念
template<class T, class U>
concept Derived = std::is_base_of<U, T>::value;
```

概念不能递归地提及自身，而且不能受约束： 
```
template<typename T>
concept V = V<T*>; // 错误：递归的概念
 
template<class T>
concept C1 = true;
template<C1 T>
concept Error1 = true; // 错误：C1 T 试图约束概念定义
template<class T> requires C1<T>
concept Error2 = true; // 错误：requires 子句试图约束概念
```

概念不能被显式实例化、显式特化或部分特化（不能更改约束的原初定义的含义）。

概念可以在标识表达式中命名。该标识表达式的值在满足约束表达式时是 true，否则是 false。

概念在作为以下内容的一部分时也可以在类型约束中被命名： 
-     类型模板形参声明
-    占位类型说明符
-    复合要求 

**概念在 类型约束 中接受的实参要比它的形参列表要求的==要少一个==，因为按语境推导出的类型会==隐式地==用作第一个实参**： 

```
template<class T, class U>
concept Derived = std::is_base_of<U, T>::value;
 
template<Derived<Base> T>
void f(T); // T 被 Derived<T, Base> 约束
```


## 约束
约束是逻辑操作和操作数的序列，它指定了对模板实参的要求。它们可以在 requires 表达式（见下文）中出现，也可以直接作为概念的主体。
有三种类型的约束：
1) 合取（conjunction）
2) 析取（disjunction）
3) 不可分割约束（atomic constraint）

对包含遵循以下顺序的操作数的逻辑与表达式进行规范化，确定与一个声明关联的约束：
-    每个声明中受约束的类型模板形参或带占位类型声明的非类型模板形参所引入的约束表达式，按出现顺序；
-    模板形参列表之后的 requires 子句中的约束表达式；
-    简写函数模板声明中每个拥有受约束占位类型的形参所引入的约束表达式;
-    尾部的 requires 子句中的约束表达式。 

这个顺序决定了在检查是否满足时各个约束时的实例化顺序。

受约束的声明只能以相同的语法形式重声明。不要求诊断： 

```
template<Incrementable T>
void f(T) requires Decrementable<T>;
 
template<Incrementable T>
void f(T) requires Decrementable<T>; // OK：重声明
 
template<typename T>	
    requires Incrementable<T> && Decrementable<T>
void f(T); // 非良构，不要求诊断
 
// 下列两个声明拥有不同的约束：
// 第一个声明拥有 Incrementable<T> && Decrementable<T>
// 第二个声明拥有 Decrementable<T> && Incrementable<T>
// 尽管它们在逻辑上等价
 
template<Incrementable T> 
void g(T) requires Decrementable<T>;
 
template<Decrementable T> 
void g(T) requires Incrementable<T>; // 非良构，不要求诊断
```


### 合取

两个约束的合取是通过在约束表达式中使用 && 运算符来构成的：

```
template<class T>
concept Integral = std::is_integral<T>::value;
template<class T>
concept SignedIntegral = Integral<T> && std::is_signed<T>::value;
template<class T>
concept UnsignedIntegral = Integral<T> && !SignedIntegral<T>;
```

两个约束的合取只有在两个约束都被满足时才会得到满足。合取从左到右短路求值（如果不满足左侧的约束，那么就不会尝试对右侧的约束进行模板实参替换：这样就会防止出现立即语境外的替换所导致的失败）。 

```
template<typename T>
constexpr bool get_value() { return T::value; }
 
template<typename T>
    requires (sizeof(T) > 1 && get_value<T>())
void f(T);   // #1
 
void f(int); // #2
 
void g() 
{
    f('A'); // OK，调用 #2。当检查 #1 的约束时，
            // 不满足 'sizeof(char) > 1'，故不检查 get_value<T>()
}
```

### 析取
两个约束的析取，是通过在约束表达式中使用 || 运算符来构成的：

如果其中一个约束得到满足，那么两个约束的析取的到满足。析取从左到右短路求值（如果满足左侧约束，那么就不会尝试对右侧约束进行模板实参替换）。

```
template<class T = void>
    requires EqualityComparable<T> || Same<T, void>
struct equal_to;
```

### 不可分割约束

不可分割约束由一个表达式 E，和一个从 E 内出现的模板形参到（对受约束实体的各模板形参的有所涉及的）模板实参的映射组成。这种映射被称作形参映射。

不可分割约束在约束规范化过程中形成。E 始终不会是逻辑与（AND）或者逻辑或（OR）表达式（它们分别构成析取和合取）。

对不可分割约束是否满足的检查会通过替换形参映射和各个模板实参到表达式 E 中来进行。如果替换产生了无效的类型或表达式，那么约束就没有被满足。否则，在任何左值到右值转换后，E 应当是 bool 类型的纯右值常量表达式，当且仅当它求值为 true 时该约束得以满足。

E 在替换后的类型必须严格为 bool。不能有任何转换：

```
template<typename T>
struct S
{
    constexpr operator bool() const { return true; }
};
 
template<typename T>
    requires (S<T>{})
void f(T);   // #1
 
void f(int); // #2
 
void g()
{
    f(0); // 错误：检查 #1 时 S<int>{} 不具有 bool 类型，
          // 尽管 #2 能更好地匹配
}
```

如果两个不可分割约束由在源码层面上相同的表达式组成，且它们的形参映射等价，那么认为它们等同。 

```
template<class T>
constexpr bool is_meowable = true;
 
template<class T>
constexpr bool is_cat = true;
 
template<class T>
concept Meowable = is_meowable<T>;
 
template<class T>
concept BadMeowableCat = is_meowable<T> && is_cat<T>;
 
template<class T>
concept GoodMeowableCat = Meowable<T> && is_cat<T>;
 
template<Meowable T>
void f1(T); // #1
 
template<BadMeowableCat T>
void f1(T); // #2
 
template<Meowable T>
void f2(T); // #3
 
template<GoodMeowableCat T>
void f2(T); // #4
 
void g()
{
    f1(0); // 错误，有歧义：
           // BadMeowableCat 和 Meowable 中的 is_meowable<T>
           // 构成了有区别的不可分割约束且它们并不等同（因此它们不互相包含）
 
    f2(0); // OK，调用 #4，它比 #3 更受约束
           // GoodMeowableCat 从 Meowable 获得其 is_meowable<T>
}
```

#### 约束规范化
约束规范化是将一个约束表达式变换为一个不可分割约束的合取与析取的序列的过程。表达式的范式定义如下：
-    表达式 (E) 的范式就是 E 的范式；
-    表达式 E1 && E2 的范式是 E1 和 E2 范式的合取；
-    表达式 E1 || E2 的范式是 E1 和 E2 范式的析取；
-    表达式 C<A1, A2, ... , AN>（其中 C 指名某个概念）的范式，是以 A1, A2, ... , AN 对 C 的每个不可分割约束的形参映射中的 C 的对应模板形参进行替换之后，C 的约束表达式的范式。如果在这种形参映射中的替换产生了无效的类型或表达式，那么程序非良构，不要求诊断。 

```
template<typename T>
concept A = T::value || true;
 
template<typename U> 
concept B = A<U*>; // OK：规范化为以下各项的析取
                   // - T::value（映射为 T -> U*）和
                   // - true（映射为空）。
                   // 映射中没有无效类型，尽管 T::value 对所有指针类型均非良构
 
template<typename V> 
concept C = B<V&>; // 规范化为以下的析取
                   // - T::value（映射为 T-> V&*）和
                   // - true（映射为空）。
                   // 映射中构成了无效类型 V&* => 非良构，不要求诊断
```

-    任何其他表达式 E 的范式是一条不可分割约束，它的表达式是 E 而它的形参映射是恒等映射。这包括所有折叠表达式，甚至包括以 && 或 || 运算符进行的折叠。 

用户定义的 && 或 || 重载在约束规范化上无效。 


## requires 子句
关键词 requires 用来引入 requires 子句，它指定对各模板实参，或对函数声明的约束。 
```
template<typename T>
void f(T&&) requires Eq<T>; // 可以作为函数声明符的末尾元素出现
 
template<typename T> requires Addable<T> // 或者在模板形参列表的右边
T add(T a, T b) { return a + b; }
```

这种情况下，关键词 requires 必须后随某个常量表达式（因此可以写成 requires true），但这是为了使用一个具名概念（如上例），具名概念的一条合取/析取，或者一个 requires 表达式。
表达式必须具有下列形式之一：
-    初等表达式，例如 Swappable<T>、std::is_integral<T>::value、(std::is_object_v<Args> && ...) 或任何带括号的表达式
-    以运算符 && 联结的初等表达式的序列
-    以运算符 || 联结的前述表达式的序列 

```
template<class T>
constexpr bool is_meowable = true;
 
template<class T>
constexpr bool is_purrable() { return true; }
 
template<class T>
void f(T) requires is_meowable<T>; // OK
 
template<class T>
void g(T) requires is_purrable<T>(); // 错误：is_purrable<T>() 不是初等表达式
 
template<class T>
void h(T) requires (is_purrable<T>()); // OK
```


## 约束的偏序
在任何进一步的分析之前都会对各个约束进行规范化，对每个具名概念的主体和每个 requires 表达式进行替换，直到剩下不可分割约束的合取与析取的序列为止。

如果根据约束 P 和约束 Q 中的各不可分割约束的同一性可以证明 P 蕴含 Q，那么称 P 归入（subsume） Q。（并进行类型和表达式的等价性分析：N > 0 并不归入 N >= 0）。 

具体来说，首先转换 P 为析取范式并转换 Q 为合取范式。当且仅当以下情况下 P 归入 Q：
-    P 的析取范式中的每个析取子句都能归入 Q 的合取范式中的每个合取子句，其中
-    当且仅当析取子句中存在不可分割约束 U 而合取子句中存在不可分割约束 V，使得 U 归入 V 时，析取子句能归入合取子句；
-    当且仅当使用上文所述的规则判定为等同时，称不可分割约束 A 能归入不可分割约束 B。 

归入关系定义了约束的偏序，用于确定：
-    重载决议中非模板函数的最佳可行候选
-    重载集中的非模板函数的地址
-    模板模板实参的最佳匹配
-    类模板特化的偏序
-    函数模板的偏序 

如果声明 D1 和 D2 均受约束，且 D1 关联的约束能归入 D2 关联的约束，（或 D2 没有约束），那么称 D1 与 D2 相比至少一样受约束。如果 D1 至少与 D2 一样受约束，而 D2 并非至少与 D1 一样受约束，那么 D1 比 D2 更受约束。 

```
template<typename T>
concept Decrementable = requires(T t) { --t; };
template<typename T>
concept RevIterator = Decrementable<T> && requires(T t) { *t; };
 
// RevIterator 能归入 Decrementable，但反之不行
 
template<Decrementable T>
void f(T); // #1
 
template<RevIterator T>
void f(T); // #2，比 #1 更受约束
 
f(0);       // int 只满足 Decrementable，选择 #1
f((int*)0); // int* 满足两个约束，选择 #2，因为它更受约束
 
template<class T>
void g(T); // #3（无约束）
 
template<Decrementable T>
void g(T); // #4
 
g(true); // bool 不满足 Decrementable，选择 #3
g(0);    // int 满足 Decrementable，选择 #4，因为它更受约束
 
template<typename T>
concept RevIterator2 = requires(T t) { --t; *t; };
 
template<Decrementable T>
void h(T); // #5
 
template<RevIterator2 T>
void h(T); // #6
 
h((int*)0); // 有歧义
```


================
https://zh.cppreference.com/w/cpp/language/requires

# Requires 表达式 (C++20 起)

产生一个描述约束的 bool 类型的纯右值表达式。 


```
requires { 要求序列 } 		
requires ( 参数列表(可选) ) { 要求序列 }
``` 		
参数列表 	- 	一个用逗号分隔的参数列表，就像在函数声明中一样，但不允许使用默认参数，而且不能以省略号结尾（除了一个表示参数包的省略号）。这些参数没有存储周期、链接性质或生命周期，只用于协助指定要求。这些参数的作用域直到要求序列的结束符 }。

要求序列 	- 	要求序列，每一个要求都是如下形式的一种:
-    简单要求
-    类型要求
-    复合要求
-    嵌套要求 


要求可以引用作用域内的模板参数、参数列表中引入的局部参数，以及其他任何在上下文中可见的声明。

将模板参数替换到模板化实体的声明中使用的 requires 表达式中，可能会导致在其要求中形成无效的类型或表达式，或者违反这些要求的语义。在这种情况下，requires 表达式的值为 false，不会导致非良构。替换和语义约束检查按照词法顺序进行，当遇到决定 requires 表达式结果的条件时就停止。如果替换（若存在）和语义约束检查成功，则 requires 表达式的结果为 true。

如果在一个 requires 表达式中，任何模板参数替换后都不可能满足要求，则程序非良构，不要求诊断： 
```
template<class T>
concept C = requires
{
    new int[-(int)sizeof(T)]; // 对任何 T 都无效: 非良构，不要求诊断
};
```
如果一个 requires 表达式在其要求中包含无效的类型或表达式，并且它没有出现在模板化实体的声明中，则程序非良构。 

## 简单要求
一个简单的要求是一个任何不以关键字 requires 开始的表达式语句。它断言该表达式是有效的。表达式是一个不求值的操作数；只检查语言的正确性。 
```
template<typename T>
concept Addable = requires (T a, T b)
{
    a + b; // "需要表达式 a+b 被编译为有效的表达式"
};
 
template<class T, class U = T>
concept Swappable = requires(T&& t, U&& u)
{
    swap(std::forward<T>(t), std::forward<U>(u));
    swap(std::forward<U>(u), std::forward<T>(t));
};
```
一个以关键字 requires 开始的要求总是被解释为一个嵌套要求。因此一个简单的要求不能以非父级的 requires 表达式开始。 

## 类型要求
类型要求是关键字 typename 后面接一个可选限定的类型名称。该要求是指命名的类型是有效的：可以用来验证某个命名的嵌套类型是否存在，或者某个类模板特化是否命名了某个类型，或者某个别名模板特化是否命名了某个类型。一个命名类模板特化的类型要求并不要求该类型是完整的。 

```
template<typename T>
using Ref = T&;
 
template<typename T>
concept C = requires
{
    typename T::inner; // 需要嵌套成员名
    typename S<T>;     // 需要类模板特化
    typename Ref<T>;   // 需要别名模板替换
};
 
template<class T, class U>
using CommonType = std::common_type_t<T, U>;
 
template<class T, class U>
concept Common = requires (T&& t, U&& u)
{
    typename CommonType<T, U>; // CommonType<T, U> 是一个合法的类型名
    { CommonType<T, U>{std::forward<T>(t)} }; 
    { CommonType<T, U>{std::forward<U>(u)} }; 
};
```

## 复合要求
一个复合要求具有如下形式
```
{ 表达式 } noexcept(可选) 返回类型要求(可选) ;
``` 		

返回类型要求 	- 	-> 类型约束

并断言命名表达式的属性。替换和语义约束检查按以下顺序进行：
1) 模板参数 (若存在) 被替换到表达式;
2) 如果使用 noexcept，表达式 一定不能潜在抛出;
3) 如果 返回类型要求 存在，则：
	a) 模板参数被替换到 返回类型要求};
	b) decltype((expression)) 必须满足类型约束的约束。否则，被包含的 requires 表达式是 false。

```
template<typename T>
concept C2 = requires(T x)
{
    // 表达式 *x 必须合法
    // 并且 类型 T::inner 必须存在
    // 并且 *x 的结果必须可以转换为 T::inner
    {*x} -> std::convertible_to<typename T::inner>;
 
    // 表达式 x + 1 必须合法
    // 并且 std::same_as<decltype((x + 1)), int> 必须满足
    // 即, (x + 1) 必须为 int 类型的纯右值
    {x + 1} -> std::same_as<int>;
 
    // 表达式 x * 1 必须合法
    // 并且 它的结果必须可以转换为 T
    {x * 1} -> std::convertible_to<T>;
};
```

## 嵌套要求
一个嵌套要求具有如下形式
```
requires 约束表达式 ;
``` 		

它可用于根据本地参数指定其他约束。约束表达式必须由被替换的模板参数（若存在）满足。将模板参数替换到嵌套要求中会导致替换到约束表达式中，仅限于确定是否满足约束表达式所需的程度。 

```
template<class T>
concept Semiregular = DefaultConstructible<T> &&
    CopyConstructible<T> && Destructible<T> && CopyAssignable<T> &&
requires(T a, size_t n)
{  
    requires Same<T*, decltype(&a)>; // 嵌套："Same<...> 被求值为真"
    { a.~T() } noexcept; // 复合："a.~T()" 是一个不会抛出的合法表达式
    requires Same<T*, decltype(new T)>; // 嵌套："Same<...> 被求值为真"
    requires Same<T*, decltype(new T[n])>; // 嵌套
    { delete new T }; // 复合
    { delete new T[n] }; // 复合
};
```

注解

关键字 requires 也用来引入 requires 子句。 
```
template<typename T>
concept Addable = requires (T x) { x + x; }; // requires 表达式
 
template<typename T> requires Addable<T> // requires 子句，不是 requires 表达式
T add(T a, T b) { return a + b; }
 
template<typename T>
    requires requires (T x) { x + x; } // 临时约束，注意关键字用了两次
T add(T a, T b) { return a + b; }
```











================


================


================

================