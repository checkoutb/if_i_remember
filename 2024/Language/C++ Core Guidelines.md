C++ Core Guidelines
2023年3月28日
15:14

===============

===============

https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md
October 6, 2022

文档的目的是帮助人们更有效地使用modern C++。

文档着重于相对高层的问题，比如，接口，资源管理，内存管理，并发。这些规则  影响着  应用的架构  和  library的设计。遵守这些规则  可以使得  代码  静态类型安全，没有资源泄漏，发现更多的编程逻辑错误。可以让你的代码更快。

In: Introduction
。。跳

In.0: Don't panic!
。。不要慌。。
花时间理解  rule对你的代码的  影响。

本份准则  着重于  C++的  核心。  大型组织，大型工程  需要  进一步的rule，可能更加严格的限制，更多的  库支持。例如，  高实时系统  的开发者  通常  不能使用  free store (dynamic memory)，并且  对  库的选择  有更大的限制。

一些rule  旨在增加  各种形式的  安全性，一些旨在  降低  发生事故的几率。

这些  rule  是为了让  代码更简单，更安全，而不损失性能。

某些  rule  在  某些场景下  是有害的，  我们的目标是：do the most good for most programmers。

P: Philosophy
。。哲学
哲学的规则很难使用机器来检查。
没有哲学的基础，rule  就缺乏  成立的理由

P.1: Express ideas directly in code
。直接在代码中展现idea
。。应该就是  代码  能直接描述  需求。

编译器  不会读取  注释，也不读取  设计文档，  许多开发人员也是。
代码中  表达的语义  能够被  编译器和其他工具  检查。

Example
class Date {
public:
    Month month() const;  // do
    int month();          // don't
    // ...
};

第一个month  函数的  声明  是  explicit(明确的):  它返回  Month，并且  不会  修改  Date对象的状态。
第二个版本  使得reader  需要猜测(。。函数的功能，(副)作用)，增加了  导致  未捕获的bug  的  可能性。

Example, bad
下面这个loop  是  std::find  的  受限形式
void f(vector<string>& v)
{
    string val;
    cin >> val;
    // ...
    int index = -1;                    // bad, plus should use gsl::index
    for (int i = 0; i < v.size(); ++i) {
        if (v[i] == val) {
            index = i;
            break;
        }
    }
    // ...
}
。。查了下，  gsl  好少，  应该是  GNU Scitentific Library  ？   科学计算库。。
。。不是，  是  GSL: Guidelines support library  。  就是  这篇文章  的  配套。
。。微软实现的。。
https://github.com/microsoft/GSL
gsl::index

An alias to std::ptrdiff_t. It serves as the index type for all container indexes/subscripts/sizes.

。。

Example, good
void f(vector<string>& v)
{
    string val;
    cin >> val;
    // ...
    auto p = find(begin(v), end(v), val);  // better
    // ...
}

设计良好的  库  表达出来的意图(  做出什么，而不是  在做什么  )  比  直接使用编程语言  表达出来的意图  更好。

C++开发人员  应该知道  标准库的  基本功能，并且  在  合适的时候  使用。
所有开发人员  都应该知道  工程所使用的  基础库  的  功能，并  合理地使用。
使用本  guidelines  的人  都应该  知道  guidelines support library，并  合理使用。

Example
change_speed(double s);   // bad: what does s signify?
// ...
change_speed(2.3);
。。声明  和  调用。

更好的方法是  明确  double  的含义  (  绝对速度  还是  增量速度)
change_speed(Speed s);    // better: the meaning of s is specified
// ...
change_speed(2.3);        // error: no unit
change_speed(23_m / 10s);  // meters per second

。。这nm要扩写多少。。

我们可以接受  一个  原生  double，  当做  delta，但是  容易出错。  如果  我们同时  想要  绝对速度  和  增量速度，  我们应该  定义  Delta  类型。

Enforcement  强制
very hard in general (总的来说很难)

一直使用  const (。检查  成员函数  是否修改了  this对象，检查  函数  是否修改了  通过  指针传递  或  ref传递  过来的  实参)

转换的时候  使用  强制转换(  。不要依赖于  隐式转换)
检测  模仿标准库的  代码  (  。就是说，使用  标准库  来完成  类似功能)

P.2: Write in ISO Standard C++
本份指南是编写  ISO  标准C++  的  指南。(。所以。)

一些  必须要扩展  的  场景：  比如  要访问系统资源。  这种情况下，  本地化必要扩展的使用，使用非核心代码指南  控制它们的使用。  如果可以的话，使用接口封装这些扩展，来使得它们可以在  不支持这些扩展的  系统上  关闭编译。

。。不知道说了什么。但是又说了点什么

扩展  所表达  的语义  并不严格。甚至  不同编译器  实现的扩展  的行为，边界条件  都稍有不同。  当使用这种扩展时，  可移植性会  受到影响。

使用  valid ISO C++  不能保证  移植性  (更别说  正确性)。  避免  依赖  未定义行为  (如，evel  的  未定义的  顺序)，注意  实现的构造的  含义  (如  sizeof(int))

限制  标准C++  或  库  功能的使用  是  必要的，比如，要避免  动态内存分配，这个功能在  飞机控制软件标准  中是  必须的。这种情况下，通过  为特殊环境  特化的  代码指南  来控制  扩展的  使用或不使用。

Enforcement
使用最新的  C++  编译器  (目前是  C++20  或  C++17)，并且  使用  不接收扩展  的  选项。
。。C++  中  extension  是什么？
。。感觉是一些  C++  的  编译器的  非标准的  扩展？

P.3: Express intent
除非  一些代码的意图  已经  被表达  (比如，在  名称  或  注释中)，  否则，不可能  确定  代码  是否  如同预期般  运行。

Example
gsl::index i = 0;
while (i < v.size()) {
    // ... do something with v[i] ...
}
上面没有表达出：遍历  v  个每个元素。  下标的实现  被暴露了  (  可能导致  下标被误用)，i  在  循环的scope  外。

Better:
for (const auto& x : v) { /* do something with the value of x */ }

for (auto& x : v) { /* modify x */ }

使用  具名算法，可以更好
for_each(v, [](int x) { /* do something with the value of x */ });
for_each(par, v, [](int x) { /* do something with the value of x */ });

最后一个变种，展示了  我们并不关心  v  中元素的顺序。

开发人员应该熟悉：
    The guidelines support library
    The ISO C++ Standard Library
    Whatever foundation libraries are used for the current project(s)
。。第1,2 Library  是  本文的  一个章节。

备选方案：说应该做些什么，而不是仅仅应该如何做。

一些语言结构  表达的意图  比其他的结构好。

Example
如果2个int  代表  2维平面的  一个点：
draw_line(int, int, int, int);  // obscure(晦涩)
draw_line(Point, Point);        // clearer

Enforcement
找到这些  可以有更好的替代的  模式
简单  for  循环  vs range-for
f(T*, int)  接口  vs f(span<T>)接口
范围  过大的  循环变量
裸  new  和  delete   。。。。这个我知道，RAII包装下，C++保证析构，从而保证delete
函数的  内建类型的  参数  太多。  。。  应该是指  上的  2个int  代表  2维的点。

P.4: Ideally, a program should be statically type safe

理想中，程序应该是  完全  静态(编译时)  类型安全的。不幸的是，这是不可能的：
问题领域：
unions
casts
array decay(衰退，腐朽)
range errors
narrowing conversions  缩小  转换

这些领域  是  严重问题(崩溃，安全)的  来源。我们尝试提供  替代技术

Enforcement
我们可以  禁止，限制，检测  不同的问题类别，  根据  不同程序的  要求  和  可行性：
unions ->  使用  variant ( in C++17)
casts ->  最少化使用；借助模板
array decay ->  使用  span (  从  GSL)
range errors ->  使用  span
narrowing conversions ->  最小化它们的使用，使用  narrow  或  narrow_cast (  从GSL)

P.5: Prefer compile-time checking to run-time checking
原因
代码清晰和性能。  你不需要为  编译时错误  编写  error handler。

Example
// Int is an alias used for integers
int bits = 0;         // don't: avoidable code
for (Int i = 1; i; i <<= 1)
    ++bits;
if (bits < 32)
    cerr << "Int too small\n";

这个例子没有实现  它所想完成的功能  (因为  溢出是  未定义的)  ，应该使用  static_assert  来代替：
// Int is an alias used for integers
static_assert(sizeof(Int) >= 4);    // do: compile-time check

更好的是  使用  类型系统，使用int32_t  代替  Int。

Example
void read(int* p, int n);   // read max n integers into *p

int a[100];
read(a, 1000);    // bad, off the end

更好的：
void read(span<int> r); // read into the range of integers r

int a[100];
read(a);        // better: let the compiler figure out the number of elements

替代方案：不要将  可以在  编译期完成的事情  延后到  run time

Enforcement
    Look for pointer arguments.
    Look for run-time checks for range violations.
。寻找指针  (并修改(  为span))
。寻找运行时的  range check

P.6: What cannot be checked at compile time should be checkable at run time
不能在编译期  检查的  东西  应该能在运行时  检查。

将难以检测  的错误  保留在  程序中，就是  自取灭亡。

理想情况下，我们捕获所有的  错误，在编译  或运行  时。
在编译期  捕获所有  error  是不可能的，且  通常无法  在  run time  捕获所有  剩余的  error。

但是，我们应该  努力(endeavor)  编写  原则上  可以check  的程序，只要有足够的资源  (分析程序，run time check,  机器资源，时间)

Example, bad
// separately compiled, possibly dynamically loaded
extern void f(int* p);

void g(int n)
{
    // bad: the number of elements is not passed to f()
    f(new int[n]);
}

这里，一个关键信息  (元素的数量)  被完全  掩盖了，  以至于  静态分析  可能变得  不可行。  而当  f()  是  ABI  的  一部分时，动态检查可能非常困难。

Example, bad
// separately compiled, possibly dynamically loaded
extern void f2(int* p, int n);

void g2(int n)
{

    f2(new int[n], m);  // bad: a  wrong number of elements can be passed to  f2()

}

将数组元素个数  作为参数  比  只传递指针然后依赖某种约定来获得数组长度  更好。
。。sizeof(arr)/sizeof(int)  。。  这个难道不通用吗？

但是如上所示，  一个简单的错字  会引起  验证的错误。

而且，这里也  隐式表达了：  f2()  支持  delete  参数。

。。ABI：  application binary interface

Example, bad
标准库资源管理指针，  当它指向对象时，无法传递  size。
// separately compiled, possibly dynamically loaded
// NB: this assumes the calling code is ABI-compatible, using a
// compatible C++ compiler and the same stdlib implementation
extern void f3(unique_ptr<int[]>, int n);

void g3(int n)
{
    f3(make_unique<int[]>(n), m);    // bad: pass ownership and size separately
}

Example
我们必须  传递  指针  和  元素个数

extern void f4(vector<int>&);   // separately compiled, possibly dynamically loaded

extern void f4(span<int>);      // separately compiled, possibly dynamically loaded

                                // NB: this assumes the calling code is ABI-compatible, using a

                                // compatible C++ compiler and the same stdlib implementation

void g3(int n)
{
    vector<int> v(n);
    f4(v);                     // pass a reference, retain ownership
    f4(span<int>{v});          // pass a view, retain ownership
}

这种设计，将  元素个数  作为  对象的一部分，  这样，错误不太可能发生，并且  动态(run time)  检查  总是可行的。

Example
我们如何转移  所有权  以及  用于校验的所有信息？
vector<int> f5(int n)    // OK: move
{
    vector<int> v(n);
    // ... initialize v ...
    return v;
}

unique_ptr<int[]> f6(int n)    // bad: loses n
{
    auto p = make_unique<int[]>(n);
    // ... initialize *p ...
    return p;
}

owner<int*> f7(int n)    // bad: loses n and we might forget to delete
{
    owner<int*> p = new int[n];
    // ... initialize *p ...
    return p;
}

Enforcement

Flag (pointer, count)-style interfaces (this will flag a lot of examples that can't be fixed for compatibility reasons)

。。感觉没什么用，我还是喜欢  sizeof() / sizeof()  。  不知道这个  会有什么坑。

P.7: Catch run-time errors early
Reason
避免"神秘的"崩溃，避免error  导致  (可能无法察觉的)  错误的结果。

Example
void increment1(int* p, int n)    // bad: error-prone
{
    for (int i = 0; i < n; ++i) ++p[i];
}

void use1(int m)
{
    const int n = 10;
    int a[n] = {};
    // ...
    increment1(a, m);   // maybe typo, maybe m <= n is supposed
                        // but assume that m == 20
    // ...
}

改进：
void increment2(span<int> p)
{
    for (int& x : p) ++x;
}

void use2(int m)
{
    const int n = 10;
    int a[n] = {};
    // ...
    increment2({a, m});    // maybe typo, maybe m <= n is supposed
    // ...
}
现在，  m<=n  可以在  调用时  检查，而不是  后续遍历时检查。

进一步简化
void use3(int m)
{
    const int n = 10;
    int a[n] = {};
    // ...
    increment2(a);   // the number of elements of a need not be repeated
    // ...
}
。。？这个是因为  span  有一个  接受数组的构造器，  所以进行了  隐式转换，转成span  ？

Example, bad
不要重复检查  同一个值，不要使用  string  传递  有结构的data：
Date read_date(istream& is);    // read date from istream

Date extract_date(const string& s);    // extract date from string

void user1(const string& date)    // manipulate date
{
    auto d = extract_date(date);
    // ...
}

void user2()
{
    Date d = read_date(cin);
    // ...
    user1(d.to_string());
    // ...
}
数据被  校验了  2次  (  通过  Date  构造器  )，并  通过  字符串(无结构的数据)  传递

Example
多余的检查可能是昂贵的。

过早的检查  是  低效的，因为  你可能  根本不会用  这个值，  或者  只使用了一部分，(检查一部分  比检查  全部  简单很多)。

不要增加  会改变你的接口的  asymptotic behavior  (渐进行为。。不知道是什么，就当时间复杂度吧  ）  的  有效性检查  (  比如，在一个  平均复杂度  O(1)  的接口中  进行  O(n)  的  check )

class Jet {    // Physics says: e * e < x * x + y * y + z * z
    float x;
    float y;
    float z;
    float e;
public:
    Jet(float x, float y, float z, float e)
        :x(x), y(y), z(z), e(e)
    {
        // Should I check here that the values are physically meaningful?
    }

    float m() const
    {
        // Should I handle the degenerate case here?
        return sqrt(x * x + y * y + z * z - e * e);
    }
};
Jet(射流)  的物理定律  (e*e<x*x+y*y+z*z)  不是  不变量，  因为可能存在  测量误差

Enforcement
    Look at pointers and arrays: Do range-checking early and not repeatedly
对于指针和数组：不要  过早  和  重复  range-check
    Look at conversions: Eliminate or mark narrowing conversions
对于转换：消除  或  标记  缩小的转换
    Look for unchecked values coming from input
注意  来自input  的  未检查值

    Look for structured data (objects of classes with invariants) being converted into strings

注意  结构化的数据  变成  string

P.8: Don't leak any resources
Reason
即使是  一个缓慢增长的资源需求，  随着时间推移，这些资源  也会被耗尽。
这对  长期运行的  程序  是非常重要的，  也是  开发人员  重要的责任

Example, bad
void f(char* name)
{
    FILE* input = fopen(name, "r");
    // ...

    if (something) return;   // bad: if something == true, a file handle is leaked

    // ...
    fclose(input);
}

Prefer RAII:
void f(char* name)
{
    ifstream input {name};
    // ...
    if (something) return;   // OK: no leak
    // ...
}

leak  一般指  没有被清理的东西。  更重要的含义是：  无法再被清理的  东西。  比如，  在heap上  申请一个  对象，然后  丢失了  这个  对象的  所有  指针。

强制执行  lifetime safety profile (本文的另一个章节)  消除了  leak。  当  和  RAII  提供的  资源安全  结合时，就  消除了  GC(  。java的gc)  的需要  (  通常不产生  垃圾  )。  将此  与  type and bound profile  结合，你获得了  完全  type安全，完全  resource安全。

Enforcement

关注  指针：将  指针分为  non-owner(默认)  和  owner。  如果可以，使用  标准库资源处理  替换  owner (如上面的例子)，  或者  使用  GSL  中的  owner  来标记  owner指针

关注  裸的  new  和  delete

关注  返回  原始指针  的  资源申请函数  (如  fopen, malloc, strdup )

P.9: Don't waste time or space
Reason
This is C++.

你为完成目标  (如，开发速度，资源安全，简化测试)  而花费的  时间，空间  不是浪费。

"Another benefit of striving for efficiency is that the process forces you to understand the problem in more depth." - Alex Stepanov

追求效率的另一个好处是，这个过程  迫使你  更深入理解  问题。

Example, bad

struct X {
    char ch;
    int i;
    string s;
    char ch2;

    X& operator=(const X& a);
    X(const X&);
};

X waste(const char* p)
{
    if (!p) throw Nullptr_error{};
    int n = strlen(p);
    auto buf = new char[n];
    if (!buf) throw Allocation_error{};
    for (int i = 0; i < n; ++i) buf[i] = p[i];
    // ... manipulate buffer ...
    X x;
    x.ch = 'a';
    x.s = string(n);    // give x.s space for *p

    for (gsl::index i = 0; i < x.s.size(); ++i) x.s[i] = buf[i];  // copy buf into x.s

    delete[] buf;
    return x;
}

void driver()
{
    X x = waste("Typical argument");
    // ...
}

X  的布局( layout)  至少浪费了  6  个  byte (很可能更多)。

copy operation  的定义  禁用了  移动语义，因此  操作会慢很多  (  注意，这里不能保证  返回值优化  (return value optimization, RVO))

buf  变量的  new  和  delete  是冗余的；  如果  我们  真的  需要  一个  本次字符串，  应该使用  local的  string。

还有几个  性能bug  和  无端的复杂性

Example, bad
void lower(zstring s)
{
    for (int i = 0; i < strlen(s); ++i) s[i] = tolower(s[i]);
}

条件：  i < strlen(s)。  这个表达式  在每次  迭代时  都  eval，  这意味着  strlen  在每次循环时  都必须  遍历  string  来  获得  长度。

当  string  内容被  修改时，  可以  假定：  tolower  不会  影响  string  的长度，  所以  可以  在  loop  外  cache  长度，  来减少  每次迭代的  消耗。

Note
waste  的  单个  例子  很少有重大意义。  在有重大意义的地方，  很容易  被专家消除。
但，在代码库中  随机分布的  water，很容易  产生  重大影响，  此时  专家也很难处理这种情况。

这个规则的  目的是  消除  和  C++的使用  有关的  waste，  在  waste  发生前。  然后  我们可以  关注  算法和需求的  浪费，不过  已经超出了  本指南的范围。

Enforcement
许多更具体的规则  致力于  整体目标的  简单性  和  消除无理由的浪费。

如果  用户定义的  非默认的  后缀  operator++, operator--  的返回值  没有被使用，那么  使用  前缀  operator++,--。  (  注意，用户定义的非默认的  旨在降低noise。  如果实践中  noise  任然太大，请review  这个  enforecement)

。。不知道说的什么。。

P.10: Prefer immutable data to mutable data
Reason

简单就能证明  常量  比  变量好。  不可变的对象  不会被  无意中修改。  有时，不可变  允许  更好的性能。  常量  不会有  data race (数据竞争)

P.11: Encapsulate messy constructs, rather than spreading through the code
Reason

杂乱的代码  更能  隐藏bug  ，  更难理解。  一个好的接口  使用起来  更容易，更安全。  杂乱，low-level  的代码  带来  更多的  杂乱，low-level的代码。

Example
int sz = 100;
int* p = (int*) malloc(sizeof(int) * sz);
int count = 0;
// ...
for (;;) {
    // ... read an int into x, exit loop if end of file is reached ...
    // ... check that x is valid ...
    if (count == sz)
        p = (int*) realloc(p, sizeof(int) * sz * 2);
    p[count++] = x;
    // ...
}
这是  低级，冗长，容易出错的。
我们  "忘记"  测试  memory exhaustion。

我们可以使用  vector
vector<int> v;
v.reserve(100);
// ...
for (int x; cin >> x; ) {
    // ... check that x is valid ...
    v.push_back(x);
}

标准库  和GSL  就是  这个规则的  最好体现。

例如，不使用  数组，union，cast，棘手的生存期问题，gsl::owner  等，  我们使用  vector, span, lock_guard, future，它们是  专业人士  设计实现的  库。

同样，我们  可以  也应该  设计实现  更专业的库，而不是  让用户  (通常是我们自己)  面对  low-level  代码。

Enforcement
寻找  混乱的代码，  比如  复杂的指针操作，  抽象的实现  之外的  强制转换。

P.12: Use supporting tools as appropriate
Reason
很多事情，机器做的更好。
重复的task下，机器不会疲倦。
我们通常有  比  重复的常规任务  更好(重要)的事情要做

Example

Run a static analyzer to verify that your code follows the guidelines you want it to follow.

Static analysis tools
Concurrency tools
Testing tools

注意不要依赖  过于精细  或  过于专业  的工具链。这些  可能会  使得  原本  可移植的代码  变得  不可移植。

P.13: Use support libraries as appropriate
Reason
使用一个  设计良好，文档优秀，有人能响应(  。论坛之类的)  的  库  节约时间和精力；
库的  质量和文档  比  你能做到的  更好。
库的  cost (  时间，精力，钱)  能  分散到  许多  user中。
一个  广泛使用的  库  更  可能  保持  更新，  移植到新的系统。
一个广泛使用的  库的  知识  可以  节约  在其他  (  也使用这个库的)  工程上的  时间

Example
std::sort(begin(v), end(v), std::greater<>());
除非你  精通  排序算法  且  有  很多时间，不然  stl  的代码  正确性更大，速度更快。
使用  stl  不需要理由。  不使用stl  需要理由。

By default use
    The ISO C++ Standard Library
    The Guidelines Support Library

如果没有  设计良好，文档良好，支持良好  的库，  可能  你应该  设计  并实现它，然后  使用它。

I: Interfaces
接口  是  程序的  2部分  之间的  约束。
精确地说明  服务的  提供者，消费者  的预期  非常重要。
拥有  良好的  (  易于理解，鼓励高效使用，不易出错，支持测试  等)  接口  可能是  代码组织  的最重要的  一个方面。

I.1: Make interfaces explicit  (明确的)
正确性。  接口中  没有  明确说明的  前提假设  很容易被忽略，也很难测试。

Example, bad
通过  global (namespace scope)  变量  来控制  函数的行为  是隐式的，并且  令人疑惑。
int round(double d)
{
    return (round_up) ? ceil(d) : d;    // don't: "invisible" dependency
}

调用者  不会观察到：  2次  round(7.2)  可能有不同的  结果。

Exception

===============

===============

===============

===============

已使用 Microsoft OneNote 2016 创建。