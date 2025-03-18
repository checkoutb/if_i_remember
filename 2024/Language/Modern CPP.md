Modern CPP
2023年3月7日
16:47

Advanced C++

应该放偏向工程的代码。
高级的用法。
语言特性的叠加。

=======================

=======================

https://zhuanlan.zhihu.com/p/321370996

C++语言中std::array的神奇用法总结

std::array  是  C++ 11  增加的。

目的是  提供  和  原生数组类似的  功能  和  性能。也正因此，使得  std::array  有很多  和其他容器  不同的  特殊之处，  比如，元素直接存放在  实例内部，而不是在  堆上分配空间；  大小必须在  编译器确定；构造器，析构器，赋值操作符  都是编译器  隐式声明的

本文是  C++17  环境

自动推导数组大小
很多项目会有类似下面的  全局数组  作为  配置参数
uint32_t g_cfgPara[] = {1, 2, 5, 6, 7, 9, 3, 4};

当要使用  std::array  替换原生数组时，麻烦来了
array<uint32_t, 8> g_cfgPara = {1, 2, 5, 6, 7, 9, 3, 4};  // 注意模板参数“8”
需要维护这个  数组长度，用起来  没有  原生数组方便。

从C++17的  类模板参数推导  开始，不再需要  手工指定  类模板参数
array g_cfgPara = {1, 2, 5, 6, 7, 9, 3, 4};  // 数组大小与成员类型自动推导
但是，数组元素的类型是什么？  还是  std::uint32_t  ？
array<uint32_t> g_cfgPara = {1, 2, 5, 6, 7, 9, 3, 4};  // 编译错误

用函数返回std::array

解决思路  是  用  函数模板  来替代  类模板  ---  因为  C++允许  函数模板的  部分参数  自动推导  ---   我们可以联想到  std::make_tuple  这类辅助函数。  C++  在TSv2  试验版中  推出过  std::make_array，但是  因为  类模板参数推导  的问世，这个工具函数  后来被删除了。

但需求还在，在C++20中，增加了一个  辅助函数  std::to_array
这个函数的代码很简单，  我们可以  把它拿来  定义  在  自己的  C++ 17  代码中
template<typename R, typename P, size_t N, size_t... I>

constexpr array<R, N> to_array_impl(P (&a)[N], std::index_sequence<I...>) noexcept

{
    return { {a[I]...} };
}

template<typename T, size_t N>
constexpr auto to_array(T (&a)[N]) noexcept
{

    return to_array_impl<std::remove_cv_t<T>, T, N>(a, std::make_index_sequence<N>{});

}

template<typename R, typename P, size_t N, size_t... I>

constexpr array<R, N> to_array_impl(P (&&a)[N], std::index_sequence<I...>) noexcept

{
    return { {move(a[I])...} };
}

template<typename T, size_t N>
constexpr auto to_array(T (&&a)[N]) noexcept
{

    return to_array_impl<std::remove_cv_t<T>, T, N>(move(a), std::make_index_sequence<N>{});

}
上面的例子  和  C++20  的推荐  实现  有所差异。

尝试使用新方法  解决老问题
auto g_cfgPara = to_array<int>({1, 2, 5, 6, 7, 9, 3, 4});  // 类型不是uint32_t？

为什么元素类型不是  std::uint32_t？

因为  模板参数推导  对  std::initializer_list  的元素  拒绝  隐式转换，  如果  你把  to_array  的模板参数  从  int  改为  uint32_t  ，会得到如下  编译  错误

template argument deduction/substitution failed:
mismatched types 'unsigned int' and 'int'

还是  不能  强制  指定类型。

为  to_array_impl  增加一个  模板参数，  让输入数组的  元素  和  返回std::array  的元素  用不同的  类型参数表示，为了实现转换到  指定的类型，我们还需要添加  2个工具函数：

template<typename R, typename P, size_t N>
constexpr auto to_typed_array(P (&a)[N]) noexcept
{
    return to_array_impl<R, P, N>(a, std::make_index_sequence<N>{});
}

template<typename R, typename P, size_t N>
constexpr auto to_typed_array(P (&&a)[N]) noexcept
{
    return to_array_impl<R, P, N>(move(a), std::make_index_sequence<N>{});
}

新的函数  有  3个模板参数，  第一个是  要返回的  std::array  的  元素类型，  后2个  和  to_array  一样。  这样  我们就可以  通过  指定第一个参数  来实现  定制  std::array  元素类型：

auto g_cfgPara = to_typed_array<uint32_t>({1, 2, 5, 6, 7, 9, 3, 4});  // 自动把元素转换成uint32_t

这段代码  可以编译运行，但是有  类型转换的  编译告警。
只要你胆子够大，可以在  to_array_impl  中  放一个  static_cast。
但是  编译告警  提示了  我们一个  不能忽视的问题，万一输入的数值  溢出怎么办？
auto g_a = to_typed_array<uint8_t>({256, -1});  // 数字超出uint8_t范围
编译器还是可以  编译运行，  g_a  中  2个元素分别是  0  和  255。

编译期字面量数值合法性校验

一种是  在  to_array_impl  函数  中增加一个  if  判断之类的  语句。  这是可行的，但是  要注意的是，这些工具函数  是可以  在  运行期调用的，对于这种常见的  基础函数来说，性能至关重要。

理想的设计是：  只有在  编译期  生成的  数组  才进行校验，并且  编译报错。  运行期调用函数时  不要加入任何校验。
可惜，C++20之前，是没有办法  指定函数  只允许在  编译期执行。

熟悉C++的人都知道：  C++编译期  处理  大多可以用  模板的  trick  来完成  ---  因为模板参数  一定是  编译期常量。  因此我们可以用  模板参数  来完成  编译期处理  ---  只要把数组元素  全部作为  模板的  非类型参数  就可以了。  当然，这里有个问题：模板的  非类型参数的  类型  怎么确定？  正好  C++17  提供了  auto  模板参数  的功能：

template<typename T>
constexpr void CheckIntRanges() noexcept {}  // 用于终结递归

template<typename T, auto M, auto... N>
constexpr void CheckIntRanges() noexcept
{
    // 防止无符号与有符号比较
    static_assert(!((std::numeric_limits<T>::min() >= 0) && (M < 0)));

    // 范围校验
    static_assert((M >= std::numeric_limits<T>::min()) &&
                  (M <= std::numeric_limits<T>::max()));

    CheckIntRanges<T, N...>();
}

template<typename T, auto... N>
constexpr auto DeclareArray() noexcept
{
    CheckIntRanges<T, N...>();
    array<T, sizeof...(N)> a{{static_cast<T>(N)...}};
    return a;
};

注意，这个函数中，所有的校验  都通过  static_assert  完成，这就保证了  校验  一定只发生在  编译期，不会带来任何  运行时开销。

。。。
static_assert在编译期执行,而assert在运行期执行。
static_assert无论在debug模式还是release模式都会正常执行,而assert只能在debug模式执行,release模式下会被...
static_assert判断条件内只能写常量而不能写变量,assert则没有此限制
。。。

DeclareArray的使用方法如下：

constexpr auto a1 = DeclareArray<uint8_t, 1, 2, 3, 4, 255>();  // 声明一个std::array<uint8_t, 5>，元素分别为1, 2, 3, 4, 255

static_assert(a1.size() == 5);
static_assert(a1[3] == 4);
auto a2 = DeclareArray<uint8_t, 1, 2, 3, -1>();  // 编译错误，-1超出uint8_t范围
auto a3 = DeclareArray<uint16_t, 1, 2, 3, 65536>();  // 编译错误，65536超出uint16_t范围

这里有一个误区需要说明：有些人可能会把DeclareArray声明成这样：
template<typename T, T... N>  // 注意N的类型为T
constexpr auto DeclareArray() noexcept

这么做的话，会发现  对数值的  校验  总是通过  ---  因为  模板参数在进入  校验之前  就已经被转换为  T  类型了。  如果你的编译器不支持  C++17  的  auto  模板参数，那么可以通过  使用  std::uint64_t, std::int64_t  这些  最大的类型  来  间接达到目的。

另一点要说明的是，C++对于非类型模板参数的允许类型存在限制，DeclareArray的方法只能用于数组元素为基本类型的场景（至少在C++20以前如此）。但是这也足够了。如果数组的元素是自定义类型，就可以通过自定义的构造函数等方法来控制类型转换。

编译期生成数组

C++11  的  constexpr  可以在  编译期  完成很多计算工作。但是一般  constexpr  函数  只能返回  单个值，  一旦  你想用它  返回  一串对象的集合，  就会遇到麻烦：  stl  容器  都有动态  内存申请功能，不能作为  编译期常量  (至少C++20之前如此)；  而原生数组  作为返回值  会退化为  指针，导致  返回悬空的指针。  即使返回  数组的  引用  也不行，会产生  悬空的引用。

constexpr int* Func() noexcept
{
    int a[] = {1, 2, 3, 4};
    return a;  // 严重错误！返回局部对象的地址
}

直到std::array的出现，这个问题才得到较好解决。std::array既可以作为编译期常量，又可以作为函数返回值。于是，它成为了编译期返回集合数据的首选。

编译期冒泡排序
template<typename T, size_t N>
constexpr std::array<T, N> Sort(const std::array<T, N>& numbers) noexcept
{
    std::array<T, N> sorted(numbers);
    for (int i = 0; i < N; ++i) {
        for (int j = N - 1; j > i; --j) {
            if (sorted[j] < sorted[j - 1]) {
                T t = sorted[j];
                sorted[j] = sorted[j - 1];
                sorted[j - 1] = t;
            }
        }
    }
    return sorted;
}

int main()
{
    constexpr std::array<int, 4> before{4, 2, 3, 1};
    constexpr std::array<int, 4> after = Sort(before);
    static_assert(after[0] == 1);
    static_assert(after[1] == 2);
    static_assert(after[2] == 3);
    static_assert(after[3] == 4);
    return 0;
}

编写constexpr函数时，注意2点：

1. constexpr  函数中  不能调用  非  constexpr  函数。因此  在交换元素时  不能  std::swap，  排序也不能直接  std::sort

2. 传入的数组  是  constexpr  的，因此参数类型必须加上  const，  也不能对数据  进行原地排序，必须返回一个  新的数组。

虽然限制很多，但编译期算法的好处也是巨大的：如果运算中有数组越界等未定义行为，编译将会失败。相比起运行时的测试，编译期测试constexpr函数能有效的提前拦截问题。而且只要编译通过就意味着测试通过，比起额外跑白盒测试用例方便多了。

上面的一大串  static_assert  让人不舒服。这么写的原因是  std::array  的  operator==  函数  并非  constexpr (至少  C++20  之前如此)。  但是  我们也可以  自定义  一个模板函数  用来判断  2个数组  是否相等。

template<typename T, typename U, size_t M, size_t N>
constexpr bool EqualsImpl(const T& lhs, const U& rhs)
{
    static_assert(M == N);
    for (size_t i = 0; i < M; ++i) {
        if (lhs[i] != rhs[i]) {
            return false;
        }
    }
    return true;
}

template<typename T, typename U>
constexpr bool Equals(const T& lhs, const U& rhs)
{
    return EqualsImpl<T, U, size(lhs), size(rhs)>(lhs, rhs);
}

template<typename T, typename U, size_t N>
constexpr bool Equals(const T& lhs, const U (&rhs)[N])
{
    return EqualsImpl<T, const U (&)[N], size(lhs), N>(lhs, rhs);
}

int main()
{
    constexpr std::array<int, 4> before{4, 2, 3, 1};
    constexpr std::array<int, 4> after = Sort(before);
    static_assert(Equals(after, {1, 2, 3, 4}));  // 比较std::array和原生数组
    static_assert(!Equals(before, after));  // 比较两个std::array
    return 0;
}

我们定义的  Equals  比  std::array  的  比较运算符  更强大，甚至可以在  std::array  和  原生数组之间  比较。
。。不过constexpr  是  编译期执行，那就不可能  是  运行时  生成的数组，比如  从数据库读取的，  外部api  传过来的。
。。编译期  的  都是  预先配置的。  感觉意义不太大。

关于Equals  有  2点需要说明：

1. std::size  是  C++17  的，对各种  容器  和数组  都能返回其大小。当然，这里的  Equals  只会允许  编译期确定大小的  容器传入，否则  触发编译失败。

2. Equals  定义了  2个版本，  这是  因为  C++的限制：  C++  禁止  {...}  这种  std::initializer_list  字面量  被推导为  模板参数类型，因此  我们必须提供一个版本声明  参数类型  为数组，以便  {1,2,3,4}  这种表达式能作为参数  传进去。

编译器排序  是一个  启发性的尝试，我们可以用  类似的方法  生成  其他的编译器集合常量，比如指定长度的自然数序列：

template<typename T, size_t N>
constexpr auto NaturalNumbers() noexcept
{
    array<T, N> arr{0};  // 显式初始化不能省
    for (size_t i = 0; i < N; ++i) {
        arr[i] = i + 1;
    }
    return arr;
}

int main()
{
    constexpr auto arr = NaturalNumbers<uint32_t, 5>();
    static_assert(Equals(arr, {1, 2, 3, 4, 5}));
    return 0;
}

可以编译，运行，但并不推荐。  因为  NaturalNumbers  函数中，  先定义了一个  内容全  0  的局部数组，  然后再  挨个修改它的值，这样  没有  直接返回  指定值  的数组  效率高。  省略arr  的初始化  会导致  编译错误  --- constexpr  函数  中  不允许  定义  没有初始化的局部变量。

。。。太多了。。跳了，

=======================
https://zhuanlan.zhihu.com/p/413864991

traits

C++  的traits技术，是一种约定俗成  的技术方案，用来  为同一类数据  (包括自定义数据类型  和  内置数据类型)  提供统一的操作函数，例如  advance(),swap(),encode()/decode()

traits可以处理的问题：

我们拥有自定义类型  Foo,Bar，以及内置类型  int,double,string，  我们想要为这些  不同的类型  提供  统一的  编码函数  decode()，  应该如何实现？

解决方案一：函数重载
// 内置类型 int, double
void decode(const int data, char* buf);
void decode(const unsigned int data, char* buf);
void decode(const double data, char* buf);
// 自定义类型 Foo, Bar
void decode(const Foo& data, char* buf);
void decode(const Bar& data, char* buf);
可行，但不满足于此，因为每增加一种数据类型  就需要重新实现一个  函数。

方案二：模板函数  +  内置字段
尝试使用模板函数来实现，自定义数据类型中定义类型字段，然后在函数中进行判断
// 自定义类型
enum Type {
  TYPE_1,
  TYPE_2
};
class Foo {
  Type type = Type::TYPE_1;
};
class Bar {
public:
  Type type = Type::TYPE_2;
};
// 模板函数
template<typename T>
void decode(const T& data, char* buf) {
  if(T::type == Type::TYPE_1) {
    ...
  }
  else if(T::type == Type::TYPE_2) {
    ..
  }
  ...
}

这样，对于  同一种自定义类型，我们只需要写一遍  decode即可，但是  对于  内置类型  int,double，  是无法在其内部定义  type  的。

这是需要用到  traits技术

方案三，traits模板类
关键在于：  使用另外的  模板类  type_traits  来保存不同数据类型的  type，这样就可以  兼容  自定义数据类型  和  内置数据类型
// 定义数据 type 类
enum Type {
  TYPE_1,
  TYPE_2,
  TYPE_3
}
对于自定义类型，和方案二类似，我们在类中定义了  数据类型type，然后在  traits类中定义  同样的  type
// 自定义数据类型
class Foo {
public:
  Type type = TYPE_1;
};
class Bar {
public:
  Type type = TYPE_2;
};
template<typename T>
struct type_traits {
  Type type = T::type;
}

对于内置类型，使用模板类的特化  为自定义类型  生成  独有的  type_traits
// 内置数据类型
template<typename int>
struct type_traits {
  Type type = Type::TYPE_1;
}
template<typename double>
struct type_traits {
  Type type = Type::TYPE_3;
}

这样，就可以为  不同数据类型  生成  统一的模板函数
// 统一的编码函数
template<typename T>
void decode<const T& data, char* buf) {
  if(type_traits<T>::type == Type::TYPE_1) {
    ...
  }
  else if(type_traits<T>::type == Type::TYPE_2) {
    ...
  }
}

总结
1. traits  技术的关键在于  使用第三方模板类  traits，利用  模板特化的功能，实现对  自定义数据  和  内置数据的  统一
2. traits  技术常见于  标准库的  实现中，但对日常开发中  降低  代码冗余  也有很好的  借鉴意义
3. C++ 20  提供了  Concept  的特性，使用  Concept  可以使得  实现类似功能更加方便。

。
要增加metadata，直接修改类，在类中增加属性  来保存metadata。  但是对于，内置类型，第三方的类，  无法修改它们。
所以通过一种转换。  这种转换就是  模板，  因为  模板的特化  是  靠  类的，所以正好  对应了上面的  为类增加metadata。

对于可以自己修改的类，  由于这个是  泛型，所以  特化不成功就走这个：  template<typename I_am_owner> { return I_am_owner::type }       这里的Type  是从类上获取的。

对于无法控制的类，  进行特化：  template<typename SpecialClass> { return Type::type1; }      。这里就写死了  Type的类型。

=======================

工程化的C++太难了。

=======================
https://www.jianshu.com/p/20577b8d273e

C++  SFINAE  与  反射机制

Substitution  failure is not an error
模板匹配失败不是错误

利用这个机制可以检车一个结构体是否包含某个成员  等操作。
C++  本身没有提供  反射机制  (  也有利用  pb  实现反射)，  利用  sfinae  机制，可以实现类似  反射  的功能

基本语法
C++  使用  关键字  template，  typename，  可以实现  基本泛型函数  或者  类。

// 一个简单的函数模板例子
template<typename T>
T get_max(T a, T b)
{

    cout << "FUNCTION NAME : " << __FUNCTION__ << ", LINE : " <<__LINE__ << endl;

    cout << a << " " << b << endl;
    auto result = a > b ? a: b;
    return result;
}

// 提供特化版本，处理某些特定类型
template<>
float get_max(float a, float b)
{

    cout << "FUNCTION NAME : " << __FUNCTION__ << ", LINE : " <<__LINE__ << endl;

    cout << "float : " << a << " " << b << endl;
    auto result = a > b ? a: b;
    return result;
}

。。这也算  偏特化？  感觉不算吧。
// 提供偏特化版本，解决传递指针的问题
template<typename T>
T get_max(T* a, T* b)
{

    cout << "FUNCTION NAME : " << __FUNCTION__ << ", LINE : " <<__LINE__ << endl;

    cout << *a << " " << *b << endl;
    auto result = *a > *b ? *a: *b;
    return result;
}

// 默认值模板
template<typename T = std::string>
T get_max(string a, string b)  // 默认值这里的参数类型需要直接指定
{

    cout << "FUNCTION NAME : " << __FUNCTION__ << ", LINE : " <<__LINE__ << endl;

    cout << "STRING : " <<a << " " << b << endl;
    return a;
}

关键字  decltype  的用法
1. 尾置返回类型
通过  decltype  指定模板函数的  返回类型

// 尾置返回类型
template<typename T1, typename T2>
auto get_max(T1 a, T2 b) -> decltype(b < a ? a : b)
{

    cout << "FUNCTION NAME : " << __FUNCTION__ << ", LINE : " <<__LINE__ << endl;

    cout << a << " " << b << endl;
    return a > b ? a: b;
}

。。这不是lambda  的用法吗。。还能和  template  结合。。不，不是结合，lambda  是匿名的，   只能说  2个不同的东西  某一方面  相同。

1. 获得变量的类型
int a=8, b=3;
auto c = a + b;  //运行时需要实际执行a+b，哪怕编译时就能推导出类型
decltype(a+b) d; //编译期类型推导
// auto c;  // error // 不可以用auto c; 直接声明变量，必须同时初始化。

模板匹配  decay  机制

模板在进行匹配的时候，会进行一个退化，比如  const int  ，如果找不到  对应的类型，会退化为  int。  其实就是把  各种  修饰(比如  ref)  去掉，把  const int&  退化为  int。

// 模板类型退化 decay
template<typename T>
T get_max_decay(T a, T b)
{

    cout << "FUNCTION NAME : " << __FUNCTION__ << ", LINE : " <<__LINE__ << endl;

    return a > b ? a: b;
}

int main()
{
    // 模板类型退化 decay
    // 看着比较抽象，其实就是把各种引用啊什么的修饰去掉，把cosnt int&退化为int
    int const c = 42;
    int i = 1;
    get_max_decay(i, c); // OK: T 被推断为 int，c 中的 const 被 decay 掉
    get_max_decay(c, c); // OK: T 被推断为 int

    int& ir = i;
    get_max_decay(i, ir); // OK: T 被推断为 int， ir 中的引用被 decay 掉

    // ...
}
。。那这种退化以后，  如果  内部进行了修改  怎么办？

SFINAE机制
看着很抽象，实际非常简单，是一种常见的模板技巧。比如，利用模板的  SFINAE  判断一个  结构体中  是否包含  某个成员。

#include <iostream>
#include <type_traits>
using namespace std;

template<typename T>
struct check_has_member_id
{
    // 仅当T是一个类类型时，“U::*”才是存在的，从而这个泛型函数的实例化才是可行的
    // 否则，就将触发SFINAE
    template<typename U>
     static void check(decltype(&U::id)){}

    // // 仅当触发SFINAE时，编译器才会“被迫”选择这个版本
    template<typename U>
    static int check(...){}

    enum {value = std::is_void<decltype(check<T>(NULL))>::value};
};

struct TEST_STRUCT
{
    int rid;
};

struct TEST_STRUCT2
{
    int id;
};

int main()
{
    check_has_member_id<TEST_STRUCT> t1;
    cout << t1.value << endl;

    check_has_member_id<TEST_STRUCT2> t2;
    cout << t2.value << endl;

    check_has_member_id<int> t3;
    cout << t3.value << endl;
    return 0;
}
// g++ --std=c++11  xxx.c

核心代码是  在实例化  check_has_member_id  对象的时候，通过模板参数  T  的类型，决定了  结构体  中  对象  value  的值。  而  value  的值  是通过  check<T>  函数的返回值  是否是  void  决定的。

如果  T  中  含有  id  成员的话，那么就会匹配  第一个实例，返回  void；
如果  不包含id  的话，会匹配  默认的实例，返回  int。

。。。哪里简单了。。无法理解。感觉就是  根据  这个类  是否含有  某个  属性来  选择  模板，这个  感觉不可能的。完全没有理由的。。

利用这个机制还可以做出很多类似的判断，比如  判断  一个类  是否是  结构体
#include <iostream>
#include <type_traits>

// 2. 判断变量是否是一个struct 或者 类
// https://www.jianshu.com/p/d09373b83f86
template <typename T>
struct check
{
    template <typename U>
     static void check_class(int U::*) {}

    template <typename U>
    static int check_class(...) {}

    enum { value = std::is_void<decltype(check_class<T>(0))>::value };
};

class myclass {};

int main()
{
    check<myclass> t;
    std::cout << t.value << std::endl;

    check<int> t2;
    std::cout << t2.value << std::endl;
    return 0;
}

enable_if  语法
如果  T  是一个  int类型，那么返回值是  bool  类型。如果不是  int  的话，就匹配不到。
使用  enable_if  的好处是  控制函数  只接受  某些类型的(value == true)  的参数，否则编译报错。

// 判断class T 是否有某个成员函数
// enable if
// enable_if example: two ways of using enable_if
#include <iostream>
#include <type_traits>

// 1. the return type (bool) is only valid if T is an integral type:
template <class T>
typename std::enable_if<std::is_integral<T>::value,bool>::type
  is_odd (T i) {return bool(i%2);}

int main()
{
    short int i = 1;    // code does not compile if type of i is not integral
    std::cout << "i is odd: " << is_odd(i) << std::endl;

    // 编译错误
    // double j = 10.0;
    // std::cout << "i is odd: " << is_odd(j) << std::endl;

    return 0;
}

利用模板的这种机制，可以设计"通用"函数接口。比如，如果work_func<T>(T data)  函数传入的  类型T  中包含了  T::is_print  成员就打印  xx  否则打印  yy。

基于pb  的反射机制

int get_feature(const HeartBeatMessage& hb_msg, const std::string& name)
{
    const google::protobuf::Descriptor* des = hb_msg.GetDescriptor();
    const google::protobuf::FieldDescriptor* fdes = des->FindFieldByName(name);
    assert(fdes != nullptr);
    const google::protobuf::Reflection* ref = hb_msg.GetReflection();
    cout << ref->GetString(hb_msg, fdes) << endl;
    return 0;
}

int main()
{
    HeartBeatMessage msg;
    fstream input("./heartbeat.db",ios::in|ios::binary);
    if(!msg.ParseFromIstream(&input))
    {
        cerr << "read data from file error." << endl;
        return -1;
    }

    // msg 是pb的msg对象，从msg对象里面找到对应“hostname”字段
    get_feature(msg, "hostName");
}

----------------------

基础定义
template <typename T>
void f(typename T::foo) {

}

template <typename T>
void f( T) {

}

这个例子里, 如果 T 没有 foo, 当没有 sfinae 的时候, 就会直接编译错误, 因为展开的结果不合法, 不过因为现在有 sfinae 第一个展开发现不合法了不会立刻编译错误, 而是会尝试别的重载展开.

。。。？？？？  形参的  typename  是什么意思。

----------------------
https://blog.csdn.net/guangcheng0312q/article/details/103884392
SFINAE

C++  自省  是  SFINAE  的主要用法之一。

在运行时  检查对象的类型  或属性时，  C++  并不出色。默认情况下  提供的  最佳功能是  RTTI (run time type information)。  但  RTTI  并不总是可用，而且  它还提供给你的  不仅仅是  操作对象的  当前类型。  在某些情况下，例如，序列化，   动态语言或  具有反射功能的语言  确实方便，动态语言/反射  很容易就可以检查  对象  是否具有  属性  并查询该属性的值。

Boost.Hana  文档中使用  is_valid  提到的  C++ 14  解决方案
#include <boost/hana.hpp>
#include <iostream>
#include <string>

using namespace std;

namespace hana = boost::hana;
// 检查类型是否有一个serialize方法
auto hasSerialize = hana::is_valid([](auto &&x) -> decltype(x.serialize()) {});

// 序列化任意对象
template<typename T>
std::string serialize(T const &obj) {
    return hana::if_(hasSerialize(obj),
                     [](auto &x) { return x.serialize(); },
                     [](auto &x) { return to_string(x); }
    )(obj);
}

// 类型A只有to_string 方法
struct A {
};

std::string to_string(const A &) {
    return "I am A";
}

// 类型B有serialize方法
struct B {
    std::string serialize() const {
        return "I am B";
    }
};

// 类型C有个serialize数据成员与to_string方法
struct C {
    std::string serialize;
};

std::string to_string(const C &) {
    return "I am C";
}

int main() {
    A a;
    B b;
    C c;
    std::cout << serialize(a) << endl;
    std::cout << serialize(b) << endl;
    std::cout << serialize(c) << endl;
}

C++ 98  中的解决方案  依赖于  3个  关键概念：  重载解析，  SFINAE，  sizeof的静态行为。

当一个函数名称和某个函数模板名称匹配时，重载决议过程大致如下：
    根据名称找出所有适用的函数和函数模板对于适用的函数模板，要根据实际情况对模板形参进行替换；
    替换过程中如果发生错误，这个模板会被丢弃
    在上面两步生成的可行函数集合中，编译器会寻找一个最佳匹配，产生对该函数的调用
    如果没有找到最佳匹配，或者找到多个匹配程度相当的函数，则编译器需要报错。

在c++中，也有一些可以接受任何东西的陷洞函数(sink-hole functions)。首先，函数模板接受任何类型的参数(假设是T)，但是编译器的真正黑洞、魔鬼变量真空、被遗忘类型的遗忘都是可变参数函数。是的，就像可怕的C printf。

必须记住的一点是，函数模板不如可变参数函数通用。

注意：模板化函数实际上可以比普通函数更精确。但是，在平局的情况下，普通函数将具有优先级

函数调用  流程

![](../_resources/f6ee17ce3431462fad120d40f96fc6b0.png)

。。这个不需要  重载决议  吧？  形参也是  方法签名的一部分，  name lookup  能找到  void foo(string)  ？
。。好像真的只是  name  的lookup，  不算参数。。  重载决议  是非常常用的功能，  不是  极少情况才需要的。。

函数模板

![](../_resources/9d408e334d014a7389827c563140bd18.png)

SFINAE

![](../_resources/e0fd14552cf24a03a4c6f31418d82819.png)

替换就是  尝试用  提供的类型或值  替换模板参数的机制。在某些情况下，如果  替换  导致  无效代码，编译器  不应该  抛出大量错误，而应该  继续  尝试  其他  可用的  重载。SFINAE概念只是为“健全”的编译器保证这种“健全”的行为

。。。  额，这不是  很正常的吗。。  不匹配  肯定不能报错啊，  你报错了的话，怎么可能做到  重载决议，  决议  肯定是有些。。。  不匹配，  和  报错，  这2个  确实是有区别的。。。  看  解释了，  将  int::X*  解释为   匹配  还是  解释为  报错。   不过  这个  写法  真没见过：  int::X* = 0。感觉是  默认参数，  但是  int::X*  是什么？   指针？  还是  int  命名空间的，6  啊，这写的。。。

/*
 The compiler will try this overload since it's less generic than the variadic.

 T will be replace by int which gives us void f(const int& t, int::iterator* b = nullptr);

 int doesn't have an iterator sub-type, but the compiler doesn't throw a bunch of errors.

 It simply tries the next overload.
*/

template <typename T> void f(const T& t, typename T::iterator* it = nullptr) { }

// The sink-hole.
void f(...) { }

f(1); // Calls void f(...) { }

上述例子中：编译器尝试f重载,因为模板化函数比可变参数函数更精确(通用)。T将被int取代，这将使我们得到void f(const int& t, int::iterator* b = nullptr); int 没有迭代器子类型，但是编译器不会抛出一堆错误。它只是尝试下一个重载。

替换  就是  尝试用  提供的类型  或  值  替换  模板参数  的机制。在某些情况下，如果  替换  导致  无效代码，编译器  不应该  抛出大量错误，而  应该  继续  尝试  其他  可用的  重载。SFINAE概念  只是为“健全”的  编译器  保证  这种“健全”的  行为。

所有的表达式都不会导致  SFINAE，一个广泛的规则是说  功能/方法主体之外的  所有替代  都是  "安全的"。

函数体内的错误替换  将导致  可怕的  C++  模板错误

// The compiler will be really unhappy when it will later discover the call to hahahaICrash.

// 当以后发现对hahahaICrash的调用时，编译器将非常不满意。
template <typename T> void f(T t) { t.hahahaICrash(); }
。。编译器错误。  确实，选好  (特化的)方法以后，  就要开始  链接，不可能运行时链接的。

void f(...) { } // The sink-hole wasn't even considered.

int main() {
    f(1);
}

通过上述探讨，我们可以得到：

![](../_resources/d301018fdf5d47b7a76f2b933ca62728.png)

可以  has_type_x  不是编译时，因此我们需要一个  在编译时可以确定的  bool，  引出  sizeof  运算符。

sizeof运算符
允许我们在编译时  返回  类型  或表达式  的字节大小。
它可以精确地计算表达式，就像  编译表达式一样精确

#include <iostream>

typedef char type_test[42];

type_test &f() {}

int main() {

    // In the following lines f won't even be truly called but we can still access to the size of its return type.

    // Thanks to the "fake evaluation" of the sizeof operator.
    char arrayTest[sizeof(f())];
    std::cout << sizeof(f()) << std::endl; // Output 42.
}

但是，如果我们能处理一些  编译时整数，那么  我们能进行  编译时比较吗？
可以
typedef char yes; // Size: 1 byte.
typedef yes no[2]; // Size: 2 bytes.

// Two functions using our type with different size.
yes &f1() {}
no &f2() {}

int main() {
    std::cout << sizeof(f1()) << std::endl;
    std::cout << sizeof(f2()) << std::endl;
    std::cout << (sizeof(f1()) == sizeof(f2())) << std::endl; // Output 0.
}

![](../_resources/a32c62c7aa9d4f7088096c6449c260a1.png)

可以看到，  has_type_x  可以在编译时  计算出  对应的  value。

现在，我们有了所有工具来创建解决方案，以在编译时  检查类型中  方法的存在。

template <class T> struct has_serialize
{
typedef char yes[1];
typedef yes no[2];

template <class U, U u> struct really_has;

template <typename C> static yes& test(really_has<std::string (C::*)(), &C::serialize>*);

。。这个  *   应该是  指针的意思

template <typename C> static yes& test(really_has<std::string (C::*)() const, &C::serialize>*);

template <typename> static no& test(…);

//  如果想要一个  纯编译时常量，且  避免在  旧编译器上  出现一些错误，用这个
enum { value = sizeof(test<T>(0)) == sizeof(yes) };

//static const bool value = sizeof(test<T>(0) == sizeof(yes));
};
std::cout<< has_serialize<B>::value <<std::endl;
std::cout<< has_serialize<int>::value << std::endl;

它不能和  继承  一起使用。

C++  中继承  和  动态多态性  是一个  运行时  可用的概念，换句话说，就是编译器将不会  拥有  无法猜测的数据。  但是，编译时类型检查  效率更高，几乎  和  运行时  一样强大。

// Using the previous A struct and hasSerialize helper.
struct D : A
{
    std::string serialize() const
    {
        return "I am a D!";
    }
};

template <class T> bool testHasSerialize(const T& /*t*/) { return hasSerialize<T>::value; }

D d;
A& a = d; // Here we lost the type of d at compile time.
std::cout << testHasSerialize(d) << std::endl; // Output 1.
std::cout << testHasSerialize(a) << std::endl; // Output 0.

仿函数
struct E
{
    struct Functor
    {
        std::string operator()()
        {
            return "I am a E!";
        }
    };

    Functor serialize;
};

E e;
std::cout << e.serialize() << std::endl; // Succefully call the functor.
std::cout << testHasSerialize(e) << std::endl; // Output 0.

。。这里还是  C++ 98  的。  感觉没有什么意义，但是都抄了这么多了。。擦

现在，你可以认为  使用  hasSerialize  来创建一个  序列化函数非常容易：
template <class T> std::string serialize(const T& obj)
{
    if (hasSerialize<T>::value) {
        return obj.serialize(); // error: no member named 'serialize' in 'A'.
    } else {
        return to_string(obj);
    }
}

A a;
serialize(a);

编译器会报错！

模板展开后
std::string serialize(const A& obj)
{
    if (0) { // Dead branching, but the compiler will still consider it!
        return obj.serialize(); // error: no member named 'serialize' in 'A'.
    } else {
        return to_string(obj);
    }
}

分支0  永远不会执行，但是  编译器  还是  执行到了  这个分支下的代码。导致  编译器发现  obj  没有  serialize

这种情况下，obj  必须同时  具有  serialize  方法  和  to_string  重载。  解决方案  包括  将序列化功能分为2个不同的功能，一个  仅使用  obj.serialize()，另一个  根据  obj的类型  使用  to_string

根据类型拆分，可以通过  SFINAE。  到时候，可以将  hasSerialize  函数  重构成  序列化函数，并使其返回  std::string  而不是  编译时  boolean。  但是  我们不会这样做，将  hasSerialize  测试  与其  使用  序列化分开  是比较干净的。

。。感觉是  机翻的  视频。。

这个问题该如何解决
第一种方案，加上  constexpr。
C++ 17  引入  if constexpr  支持在编译期  执行，  可以将之应用于  泛型编程  中的条件判断。
if constexpr (hasSerialize<T>::value)

第二种方案，不用if语句，而是将  这个函数  分为  2个函数，每个函数  对应一个  分支。  使用  enable_if  来分。
template<bool B, class T = void> // Default template version.

struct enable_if {}; // This struct doesn't define "type" and the substitution will fail if you try to access it.

template<class T> // A specialisation used if the expression is true.

struct enable_if<true, T> { typedef T type; }; // This struct do have a "type" and won't fail on access.

。。这个  true  ，应该是  偏特化。

// Usage:
enable_if<true, int>::type t1; // Compiler happy. t's type is int.

enable_if<hasSerialize<B>::value, int>::type t2; // Compiler happy. t's type is int.

enable_if<false, int>::type t3; // Compiler unhappy. no type named 'type' in 'enable_if<false, int>';

enable_if<hasSerialize<A>::value, int>::type t4; // no type named 'type' in 'enable_if<false, int>';

。。？  enable_if  ，我以为是  关键字。结果  是一种  技术。  像  traits，  利用  模板匹配机制  来  区分不同的东西。
。。看了下面的，  我在想，  这里的  enable_if  是不是  C++  的标准实现。。

。。下面的写法不太懂。。感觉是错的，就是  typename  是什么作用。。

template <class T> typename enable_if<hasSerialize<T>::value, std::string>::type serialize(const T& obj)

{
    return obj.serialize();
}

template <class T> typename enable_if<!hasSerialize<T>::value, std::string>::type serialize(const T& obj)

{
    return to_string(obj);
}

A a;
B b;
C c;

// The following lines work like a charm!
std::cout << serialize(a) << std::endl;
std::cout << serialize(b) << std::endl;
std::cout << serialize(c) << std::endl;

值得注意的2个细节
1. 返回类型上  使用了  enable_if，以保持参数推导，否则，我们必须  明确指定  类型：  serialize<A>(a)

2. 即使  使用to_string  的版本  也必须  使用  enable_if，  否则  serialize(b)  将有  2个潜在的  可用重载  并引起歧义。

C++11  的方式

decltype, declval, auto, co

![](../_resources/db38a2ef6e6448a7a47672896723d758.png)

之前使用  sizeof  对  传给它的表达式  进行  伪计算，然后返回  表达式类型的  大小。  (。。。来区分不同的  类型)

C++11  增加了  decltype。

B b;

decltype(b.serialize()) test = "test"; // Evaluate b.serialize(), which is typed as std::string.

// Equivalent to std::string test = "test";

declval  可以为你  提供  对  无法轻松构造的  类型的  对象的  伪ref。  declval  对于  我们的  SFINAE  结构  确实  十分方便。

struct Default {
    int foo() const {return 1;}
};

struct NonDefault {
    NonDefault(const NonDefault&) {}
    int foo() const {return 1;}
};

int main()
{
    decltype(Default().foo()) n1 = 1; // int n1
//  decltype(NonDefault().foo()) n2 = n1; // error: no default constructor
    decltype(std::declval<NonDefault>().foo()) n2 = n1; // int n2
    std::cout << "n2 = " << n2 << '\n';
}

auto

![](../_resources/0fbbc92ae9be45799752d411f04a4aba.png)

bool f();
auto test = f(); // Famous usage, auto deduced that test is a boolean, hurray!

// t wasn't declare at that point, it will be after as a parameter!

template <typename T> decltype(t.serialize()) g(const T& t) {   } // Compilation error

// Less famous usage:
// auto delayed the return type specification!
// the return type is specified here and use t!

template <typename T> auto g(const T& t) -> decltype(t.serialize()) {   } // No compilation error.

constexpr

![](../_resources/a328ea02d62646208f2eff8745a68136.png)

C++ 11  还提供了一种  执行编译时  计算的  新方法
新关键字  constexpr  是  编译器的一个提示，  这意味着  这个表达式  是常量，可以在  编译时  直接求值。

constexpr int factorial(int n)
{
    return n <= 1? 1 : (n * factorial(n - 1));
}

int i = factorial(5); // Call to a constexpr function.
// Will be replace by a good compiler by:
// int i = 120;

constexpr  增加了  STL  中  std::true_type  和  std::false_type  的使用。  这些类型封装了  布尔  true  和  false。  类或结构  可以  从它们继承

struct testStruct : std::true_type { }; // Inherit from the true type.

constexpr bool testVar = testStruct(); // Generate a compile-time testStruct.
bool test = testStruct::value; // Equivalent to: test = true;

test = testVar; // true_type has a constexpr converter operator, equivalent to: test = true;

融合

template<class T>
struct hasSerialize {
    // We test if the type has serialize using decltype and declval.
    template<typename C>
    static constexpr decltype(std::declval<C>().serialize(), bool())
test(int /* unused */) {

        // We can return values, thanks to constexpr instead of playing with sizeof.

        return true;
    }

    template<typename C>
    static constexpr bool test(...) {
        return false;
    }

    // int is used to give the precedence!
    static constexpr bool value = test<T>(int());
};
std::cout<< has_serialize<B>::value << std::endl;
std::cout<< has_serialize<int>::value << std::endl;

C++  逗号运算符  可以创建  多个  表达式链。  在  decltype  中，  将评估所有  表达式，但  仅将  最后一个  表达式  视为该类型。  序列化不需要任何修改，减去了  STL  中  现在  提供  enable_if  函数的事实。

测试：
template<class T>
std::string serialize(const T &obj) {
    // 不加constexpr 报错：error: no member named 'serialize' in 'A'.
    // C++17的constexpr
    if constexpr (hasSerialize<T>::value)
        return obj.serialize();
    else
        return to_string(obj);
}

int main() {
    A a;
    B b;
    C c;

    // The following lines work like a charm!
    std::cout << serialize(a) << std::endl;
    std::cout << serialize(b) << std::endl;
    std::cout << serialize(c) << std::endl;
}

第二种解决方案
Boost.Hanna  介绍的另一个使用  std::true_type  和  std::false_type  的C++ 11  的解决方案：
template<typename T, typename=std::string>
struct hasSerialize : std::false_type {

};
template<typename T>

struct hasSerialize<T, decltype(std::declval<T>().serialize())> : std::true_type {

};

这种解决方案  更狡猾。
它依赖于不太知名的默认模板参数。

当我们使用hasSerialize <OurType> :: value时，默认参数会起作用，并且实际上我们在 primary template 和 specialisation方面都正在寻找hasSerialize <OurType，std :: string> :: value。同时，将处理decltype的替换和求值，并且如果OurType具有返回std :: string的序列化方法，则我们的specialisation会被替换为具有签名hasSerialize <OurType，std :: string>，否则替换将失败。因此，在良好情况下，specialisation优先。在这种情况下，将可以使用std :: void_t C ++ 17帮助程序。无论如何，这是您可以使用的要点！

。。。

第二种解决方案隐藏了很多复杂性，我们仍然有很多  C++ 11  特性没有被使用，比如  nullptr,lambda,r-values。  我们会在  C++14  中使用一些。

C++14  优势
auto, lambda

auto
1. 返回类型推断的结果
auto  可以用于  函数或  方法的返回类型
auto myFunction() // Automagically figures out that myFunction returns ints.
{
    return int();
}

lambda
1. 函数爱好者的功能
auto foo = [](auto& t) -> decltype(t.serialize()) { return t.serialize(); };

struct UnamedType
{
template <typename T>
auto operator()(T& t) const -> decltype(t.serialize())
{
return t.serialize();
}
}

C++11:
int main() {
    B b;

    auto l1 = [](B &b) { return b.serialize(); }; // Return type figured-out by the return statement.

    auto l3 = [](B &b) -> std::string { return b.serialize(); }; // Fixed return type.

    auto l2 = [](B &b) -> decltype(b.serialize()) { return b.serialize(); }; // Return type dependant to the B type.

    std::cout << l1(b) << std::endl; // Output: I am a B!
    std::cout << l2(b) << std::endl; // Output: I am a B!
    std::cout << l3(b) << std::endl; // Output: I am a B!
}

C++14  给  lambda  带来了变化：  lambda  接受  auto  参数，根据参数推导出  参数类型。
lambda  被实现为  一个具有  新创建的  未命名类型  (也称为闭包类型)  的对象。
如果  lambda  有一些自动参数，它的  operator()  将被  简单地模板化：

void fun(A a,B b,C c) {
    // ***** Simple lambda unamed type *****
    auto l4 = [](int a, int b) { return a + b; };
    std::cout << l4(4, 5) << std::endl; // Output 9.

    // Equivalent to:
    struct l4UnamedType {
        int operator()(int a, int b) const {
            return a + b;
        }
    };
    l4UnamedType l4Equivalent = l4UnamedType();
    std::cout << l4Equivalent(4, 5) << std::endl; // Output 9 too.
    // ***** auto parameters lambda unnamed type *****
    // b's type is automagically deduced!
    auto l5 = [](auto &t) -> decltype(t.serialize()) { return t.serialize(); };

    std::cout << l5(b) << std::endl; // Output: I am a B!

//    std::cout << l5(a) << std::endl; // Error: no member named 'serialize' in 'A'.

    l5UnamedType l5Equivalent = l5UnamedType();
    std::cout << l5Equivalent(b) << std::endl; // Output: I am a B!

//    std::cout << l5Equivalent(a) << std::endl; // Error: no member named 'serialize' in 'A'.

}

除了  lambda本身，我们对生成的  未命名类型  更加感兴趣：  它的  lambda操作符()  可以用作  SFINAE。

编写lambda  比  编写等价类型要  简单：
// Check if a type has a serialize method.

auto hasSerialize = hana::is_valid([](auto&& x) -> decltype(x.serialize()) { });

重建  is_valid
auto has_serialize = is_valid([](auto&& x) -> decltype(x.serialize()) {});
std::cout<< has_serialize(43);     // 0

现在，我们已经有了一种非常时尚的方式，可以使用lambda生成具有潜在SFINAE属性的未命名类型，我们需要弄清楚如何使用它们！如您所见，hana :: is_valid是一个将lambda作为参数并返回类型的函数。我们将is_valid返回的类型称为container。container将负责保留lambda的未命名类型以供以后使用。让我们从编写is_valid函数及其container开始：

。。。已经不想看了。感觉无法理解了。

template <typename UnnamedType> struct container
{
    // Remembers UnnamedType.
};

template <typename UnnamedType> constexpr auto is_valid(const UnnamedType& t)
{
    // We used auto for the return type: it will be deduced here.
    return container<UnnamedType>();
}

auto test = is_valid([](const auto& t) -> decltype(t.serialize()) {})

// Now 'test' remembers the type of the lambda and the signature of its operator()!

下一步是使用  operator()  扩展容器，例如，我们可以用一个  参数调用它，  此参数类型将针对  UnnamedType  进行测试。  为了对参数类型进行测试，我们可以再次  对一个  重新创建的  UnnamedType  对象  使用  SFINAE：

template <typename UnnamedType> struct container
{
// Let's put the test in private.
private:
    // We use std::declval to 'recreate' an object of 'UnnamedType'.
    // We use std::declval to also 'recreate' an object of type 'Param'.
    // We can use both of these recreated objects to test the validity!
    template <typename Param> constexpr auto testValidity(int /* unused */)

    -> decltype(std::declval<UnnamedType>()(std::declval<Param>()), std::true_type())

    {
        // If substitution didn't fail, we can return a true_type.
        return std::true_type();
    }

    template <typename Param> constexpr std::false_type testValidity(...)
    {
        // Our sink-hole returns a false_type.
        return std::false_type();
    }

public:

    // A public operator() that accept the argument we wish to test onto the UnnamedType.

    // Notice that the return type is automatic!
    template <typename Param> constexpr auto operator()(const Param& p)
    {
        // The argument is forwarded to one of the two overloads.
        // The SFINAE on the 'true_type' will come into play to dispatch.
        // Once again, we use the int for the precedence.
        return testValidity<Param>(int());
    }
};

template <typename UnnamedType> constexpr auto is_valid(const UnnamedType& t)
{
    // We used auto for the return type: it will be deduced here.
    return container<UnnamedType>();
}

// Check if a type has a serialize method.
auto hasSerialize = is_valid([](auto&& x) -> decltype(x.serialize()) { });

如果你在这点上有困惑，建议你花点时间重新阅读前面的所有示例。

C++ 17

if constexpr

template <class T>
std::string serialize(const T& obj)
{
if constexpr (has_serialize(obj))
{
return obj.serialize();
}
else
{
return std::to_string(obj);
}
}
serialize(42);
std::cout<< serialize(b)<<std::endl;

前面已经用过这个方法了，这里提及一下即可。

For the fun
我没有告诉你一些事情，是故意的。否则，我单向这篇文章要长2倍。
我强烈建议  Google  我所说的内容。

1. 如果你希望有一个  与  Boost  一起工作的  解决方案。  Boost.Hana static if_，  如果你需要通过  Hana  的  等效物  来改变  testValidity  方法的返回类型

template <typename Param> constexpr auto test_validity(int /* unused */)

-> decltype(std::declval<UnnamedType>()(std::declval<Param>()), boost::hana::true_c)

{
    // If substitution didn't fail, we can return a true_type.
    return boost::hana::true_c;
}

template <typename Param> constexpr decltype(boost::hana::false_c) test_validity(...)

{
    // Our sink-hole returns a false_type.
    return boost::hana::false_c;
}

静态  if  实现非常有趣，但至少与  我们在文本中解决的问题  一样  困难。
。。但是我没有看到代码中  有  if  啊。。。

1. 如果你注意到  我们一次  只检查一个参数，  我们不能做这样的事情

auto test = is_valid([](auto&& a, auto&& b) -> decltype(a.serialize(), b.serialize()) { });

A a;
B b;

std::cout << test(a, b) << std::endl;

实际上，我们可以使用一些参数包：

template <typename UnnamedType> struct container
{
// Let's put the test in private.
private:
    // We use std::declval to 'recreate' an object of 'UnnamedType'.
    // We use std::declval to also 'recreate' an object of type 'Param'.
    // We can use both of these recreated objects to test the validity!

    template <typename... Params> constexpr auto test_validity(int /* unused */)

    -> decltype(std::declval<UnnamedType>()(std::declval<Params>()...), std::true_type())

    {
        // If substitution didn't fail, we can return a true_type.
        return std::true_type();
    }

    template <typename... Params> constexpr std::false_type test_validity(...)
    {
        // Our sink-hole returns a false_type.
        return std::false_type();
    }

public:

    // A public operator() that accept the argument we wish to test onto the UnnamedType.

    // Notice that the return type is automatic!
    template <typename... Params> constexpr auto operator()(Params&& ...)
    {
        // The argument is forwarded to one of the two overloads.
        // The SFINAE on the 'true_type' will come into play to dispatch.
        return test_validity<Params...>(int());
    }
};

template <typename UnnamedType> constexpr auto is_valid(UnnamedType&& t)
{
    // We used auto for the return type: it will be deduced here.
    return container<UnnamedType>();
}

// Check if a type has a serialize method.
auto hasSerialize = is_valid([](auto &&x) -> decltype(x.serialize()) {});

// Notice how I simply swapped the return type on the right?
template<class T>
auto serialize(T &obj)

-> typename std::enable_if<decltype(hasSerialize(obj))::value, std::string>::type {

    return obj.serialize();
}

template<class T>
auto serialize(T &obj)

-> typename std::enable_if<!decltype(hasSerialize(obj))::value, std::string>::type {

    return to_string(obj);
}

int main() {
    A a;
    B b;
    C c;
    auto test = is_valid([](const auto &t) -> decltype(t.serialize()) {});

    std::cout << test(a,b) << std::endl;
    std::cout << test(b) << std::endl;
    std::cout << test(c) << std::endl;
    // The following lines work like a charm!
    std::cout << serialize(a) << std::endl;
    std::cout << serialize(b) << std::endl;
    std::cout << serialize(c) << std::endl;
}

=======================

=======================

enable_if

=======================

=======================
typename

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

=======================

已使用 Microsoft OneNote 2016 创建。