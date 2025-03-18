C/C++-权威
2022年3月24日
10:49
.com/w/cpp/ 是 C++ ， C++基本在前半部分
.com/w/c/ 是 C， C基本在后半部分

[[toc]]


==================

https://en.cppreference.com/w/cpp/language/reference

https://en.cppreference.com/w/cpp/17

==================

https://zh.cppreference.com/w/cpp/utility

# util

工具库可以分为2类：
- 语言支持库
- 通用库

## 语言支持库

语言支持库提供的 函数 与语言特性 紧密相关，用以支持语言中一些 常见的惯用法。

类型支持  
基本类型 ( 如 std::size_t, std::nullptr_t), RTTI ( std::type_info), 类型特性 ( std::is_integral, std::rank)


常量求值语境  
C++20  
`<type_traits>`, is_constant_evaluated (函数), 检测调用是否在常量求值的语境内发生

实现属性  
C++20  
`<version>` 提供了关于C++标准库的实现依赖信息(如版本号 和发现日期)。它也定义了 特性测试宏。

程序工具  
终止(如 std::abort, std::atexit)，环境(如 std::system)，信号(如 std::raise)

动态内存管理  
智能指针( 如 std::shared_ptr)，分配器(如 std::allocator, std::pmr::memory_recource)，C风格内存管理(如 std::malloc)

错误处理  
异常(如 std::exception, std::terminate)，断言(如 assert)

源码信息捕获  
C++20  
`<source_location>`, source_location (类)，表示关于源码的信息，如文件名，行号，函数名的类

初始化器列表  
C++11  
`<initializer_list>`, initializer_list (类模板)， 在列表初始化中创建临时数组 然后引用它

三路比较  
C++20  
`<compare>`  
|函数|描述|
|--|--|
|three_way_comparable, three_way_comparable_with|和`<=>`运算符结果相同 |
|partial_ordering|三路比较的结果类型，支持所有6种运算，不可替换，允许不可比较的值 |
|weak_ordering|三路比较的结果类型，支持所有 6 种运算符且不可替换 |
|strong_ordering|三路比较的结果类型，支持所有 6 种运算符且可替换 |
|is_eq, is_neq, is_lt, is_lteq, is_gt, is_gteq|具名比较函数 |
|compare_three_way|实现 `x<=>y` 的函数对象 |
|compare_three_way_result|获得三路比较运算符 `<=>` 在给定类型上的结果 |
|common_comparison_category|给定的全部类型都能转换到的最强比较类别 |
|strong_order|进行三路比较并产生 std::strong_ordering 类型结果 |
|weak_order|进行三路比较并产生 std::strong_ordering 类型结果进行三路比较并产生 std::weak_ordering 类型结果 |
|partial_order|进行三路比较并产生 std::partial_ordering 类型结果 |
|compare_strong_order_fallback|进行三路比较并产生 std::partial_ordering 类型结果 |
|compare_weak_order_fallback|进行三路比较并产生 std::weak_ordering 类型的结果，即使 `operator<=>` 不可用 |
|compare_partial_order_fallback|进行三路比较并产生 std::partial_ordering 类型的结果，即使 `operator<=>` 不可用 |


**协程支持**  
C++20  
用于协程支持的类型，例如 std::coroutine_traits、std::coroutine_handle。

变参数函数  
支持接受任意数量参数的函数（例如通过 va_start、va_arg、va_end）。 


## 通用工具

交换  
`<utility>`  
|ff|Desc|
|--|--|
|swap|交换2个对象的值|
|exchange C++14|将实参替换为一个新值，并返回其先前值|
|ranges::swap C++20|交换2个对象的值|

类型运算  
`<utility>`  
|ff|Desc|
|--|--|
|forward 11|转发一个函数实参 |
|forward_like 23|转发函数参数，如同用类型为指定的类型模板实参的表达式的值类别与 const 性影响它 |
|move 11|获得右值引用 |
|move_if_noexcept 11|若移动构造函数不抛出则获得右值引用 |
|as_const 17|获得到其实参的 const 引用 |
|declval 11|获取到其实参的引用，用于不求值语境中 |
|to_underlying 23|转换枚举到其底层类型 |

整数比较函数  
`<utility>`  
|ff|Desc|
|--|--|
|cmp_equal, cmp_not_equal, cmp_less, cmp_greater, cmp_less_equal, cmp_greater_equal C++20|比较2个整数值，不会由于转换而改变数值(。。估计指 uint转 int)|
|in_range 20|检查整数值是否在 给定的整数范围内|

关系运算符  
> C++20废弃  

`<utility>`  
`std::rel_ops`  
|ff|Desc|
|--|--|
|operator!=, operator>, operator<=, operator>=|基于用户的 operator== 和 operator< 自动生成这些运算符|

对与元组  
`<utility>`  
|ff|Desc|
|--|--|
|pair|pair |
|piecewise_construct_t 11|用于为逐段构造选择正确函数重载的标签类型 |
|piecewise_construct 11|用于为逐段构造的函数消歧义的 piecewise_construct_t 类型的对象 |
|integer_sequence 14 `<tuple>`|实现编译时整数数列 |
|tuple 11|实现固定大小的容器，它保有类型可以相 异 的元素 |
|apply 17|以一个实参的元组来调用函数 |
|make_from_tuple 17|以一个实参元组构造对象 |

元组协议
`<tuple>, <utility>, <array>, <ranges>`  
|ff|Desc|
|--|--|
|tuple_size 11|获得元组式类型的元素数 |
|tuple_element|获得元组式类型的元素类型 |

Sum types and type erased wrappers  

|ff|header|Desc|
|--|--|--|
|optional 17|optional|可能或可能不保有一个对象的包装器 |
|expected 23|expected|含有期待或着不期待的值的包装 |
|variant 17|variant|类型安全的可辨识联合体 |
|any 17|any|可保有任何可复制构造 (CopyConstructible) 类型的实例的对象。 |
|in_place, in_place_type, in_place_index, in_place_t, in_place_type_t, in_place_index_t 17||原位构造标签 |

位集合  
`<bitset>`  
bitset: 实现常量长度的位数组 

函数对象  
部分函数应用（如 std::bind）及相关工具：用于绑定的工具，如 ==std::ref== 与 std::placeholders、多态函数包装器：std::function、预定义函数对象（如 std::plus、std::equal_to）、成员指针到函数转换器 ==std::mem_fn==。 

散列支持  
`<functional>`  
hash C++11: 散列函数对象

日期和时间  
时间跟踪（如 std::chrono::time_point、std::chrono::duration），C 风格日期和时间（如 std::time、std::clock）。 

基本字符串转换
`<charconv>`  
|ff|Desc|
|--|--|
|to_chars 17|转换整数或浮点值到字符序列 |
|from_chars 17|转换字符序列到整数或浮点值 |
|chars_format|指定 std::to_chars 和 std::from_chars 所用的格式 |

格式化库  
全部 C++20 开始  
类型安全的格式化库  
`<fromat>`  
|ff|Desc|
|--|--|
|format|在新 string 中存储参数的格式化表示 |
|format_to|通过输出迭代器写其参数的格式化表示 |
|format_to_n|通过输出迭代器写其参数的格式化表示，不超出指定的大小 |
|formatted_size|确定存储其参数的格式化表示所需的字符数 |
|vformat|std::format 的使用类型擦除的参数表示的非模板变体 |
|vformat_to|std::format_to 的使用类型擦除的参数表示的非模板变体 |
|formatter|定义给定类型的格式化规则的类模板 |
|format_error|格式化错误时抛出的异常类型 |






==================
https://en.cppreference.com/w/cpp/algorithm

https://zh.cppreference.com/w/cpp/algorithm

# algorithm

C++ 20 提供了大部分算法的 constrained(受限) 版本，在命名空间 std::ranges。

这些算法中，范围可以通过 iterator对 或 一个range参数 来指定，支持projection(推断) 和 指向元素的指针的调用。另外，大部分算法的返回类型被修改了，来返回算法执行期间的 所有潜在的 有用的 信息。

```C++
std::vector<int> v = {7, 1, 4, 0, -1};
std::ranges::sort(v); // constrained algorithm
```

。。直接一个 vector。而不是 begin，end

* * *

## 不修改序列的操作

`<algorithm>`

|     |     |
| --- | --- |
| 方法  | 描述  |
| all\_of; any\_of; none_of | 在range中，是否 predicate 是true，对于 all，any，none 的元素 |
| ranges::all\_of,any\_of,none_of | -   |
| for_each | 对范围内每个元素 应用 function |
| ranges::for_each | -   |
| for\_each\_n | 对序列中前 n 个元素 应用 function |
| ranges::for\_each\_n | -   |
| count; count_of | 返回满足指定标准的元素个数 |
| ranges::count; ranges::count_if | -   |
| mismatch | 2个range 不相同的 第一个位置 |
| ranges::mismatch | -   |
| find; find\_if; find\_if_not | 找到 第一个满足 指定标准的 元素 |
| ranges::find,find\_if,find\_if_not | -   |
| find_end, ranges:: | 在确定范围内的最后一个元素串 。。。这个方法接受2个范围。 |
| find\_first\_of, ranges:: | 搜索元素集合中的任意一个元素 |
| adjacent_find, ranges:: | 找第一对 2个相邻且相等(或都满足指定标准) 的元素 |
| search, ranges:: | 寻找相等的子串 |
| search_n, ranges:: | 寻找一个元素的连续n次出现 |
| ranges::starts_with, C++23 | 是否以另一个range为开始 |
| ranges::ends_with,C++23 | 是否以另一个range为结束 |

* * *

## 修改序列的操作

`<algorithm>`

|     |     |
| --- | --- |
| copy,copy_if, ranges:: | 复制元素到 新位置 |
| copy_n, ranges:: | 复制 n个元素到 新位置 |
| copy_backward, ranges:: | 反序复制 |
| move, ranges:: | 移动一系列元素到新位置。这个是接收3个参数的，不是右值的move |
| move_backward, ranges:: | 移动元素到新位置并反序 |
| fill, ranges:: | 给定的值，赋给每个元素 。。 这个估计要看值的 复制构造函数了。 |
| fill_n, ranges:: | 给定的值，赋予前 n 个元素 |
| transform, ranges:: | 应用function到元素，将结果存储到目标范围 |
| generate, ranges:: | 根据给定的生成器，填补first 到 last |
| remove,remove_if, | remove |
| remove\_copy, remove\_copy_if | 忽略某些(个)值，其他的全部复制 |
| replace, replace_if | 替换  |
| replace\_copy, replace\_copy_if | 复制，并且复制的过程中replace |
| swap | swap |
| swap_ranges | 3个参数，开始，结束，另一方的开始 |
| iter_swap | swap 迭代器指向的值 |
| reverse | reverse |
| reverse_copy | 复制，并复制的过程中reverse |
| rotate | fst,md,lst -> md,lst,fst |
| rotate_copy | 复制，复制的过程中rotate |
| shift\_left, shift\_right | 。   |
| random_shuffle (<C++17)<br>shuffle (C++11) | 随机重排范围内元素 |
| sample (C++17) | 从fst-lst中等概率地获得n个元素 |
| unique | 移除范围内的连续重复 |
| unique_copy | 。   |

* * *

## 划分操作

`<algorithm>`

|     |     |
| --- | --- |
| is_partitioned | 判断是否所有满足p的 都在 不满足p 的前面 |
| partition | 移动，使得满足p的在 不满足p的前面 |
| partition_copy | 。   |
| stable_partition | 满足原来相对顺序的 partition |
| partition_point | partition，并返回首个不满足p的元素的iter |

* * *

## 排序操作

`<algorithm>`

|     |     |
| --- | --- |
| is_sorted | 检查是否已升序 |
| is\_sorted\_until | 返回第一个不满足升序的元素的 iter |
| sort | 。   |
| partial_sort | 。。这个好难描述。排序后，保证前n个是 有序的 前n个 |
| partial\_sort\_copy | 。   |
| stable_sort | 排序，保持相等元素之间的 顺序 |
| nth_element | 排序后，保证第n个元素是 有序的第n个 |

* * *

## 二分搜索操作（在已排序范围上）

`<algorithm>`

|     |     |
| --- | --- |
| lower_bound | 返回第一个 不小于 (>=) 给定值的 元素的 iter |
| upper_bound | 第一个大于(>) |
| binary_search | 。   |
| equal_range | 等于给定值的 范围 |

* * *

## 其他已排序范围上的操作

`<algorithm>`

|     |     |
| --- | --- |
| merge | 合并2个有序的范围 |
| inplace_merge | 。   |

* * *

## 集合操作（在已排序范围上）

`<algorithm>`

|     |     |
| --- | --- |
| includes | 一个序列是 另一个序列的 子列，则返回 true |
| set_difference | 集合的差集 |
| set_intersection | 交集  |
| set\_symmetric\_difference | 对称差，2个集合中，非交集的部分。 |
| set_union | 并集  |

* * *

## 堆操作

`<algorithm>`

|     |     |
| --- | --- |
| is_heap | 判断给定范围是否是一个 max heap |
| is\_heap\_until | 查找 能成为最大堆的 最大子范围 |
| make_heap | 。   |
| push_heap | 。完成后，还是最大堆 |
| pop_heap | 。   |
| sort_heap | 排序后 不再是最大堆 |

* * *

## 最小/最大操作

`<algorithm>`

|     |     |
| --- | --- |
| max | 。   |
| max_element | 。   |
| min | 。   |
| min_element | 。   |
| minmax | 会悬垂ref。 |
| minmax_element | 。 这个的最大元素是最后的最大元素， max_element 是第一个最大元素 |
| clamp (C++17) | (v,lo,hi)，如果v&lt;lo,则返回lo，如果v&gt;hi，则返回hi，否则返回v。如果lo>hi，行为未定义。<br>就是返回 lo,hi 范围中 离 v 最近的 值。 |

* * *

## 比较操作

`<algorithm>`

|     |     |
| --- | --- |
| equal | 2个元素是否相等 |
| lexicographical_compare | 2个范围的 字典顺序。是否小于。 |
| lexicographical\_compare\_three_way (C++20) | 用三路比较 比较2个范围 |

* * *

## 排列操作

`<algorithm>`

|     |     |
| --- | --- |
| is_permutation | 判断一个序列是否 是另一个序列的 排列 |
| next_permutation | 字典顺序，下一个较大的 排列 |
| prev_permutation | 字典顺序，下一个较小 |

* * *

## 数值运算

`<numeric>`
这个表格没有range::。

|     |     |
| --- | --- |
| iota | 用从 起始值开始 连续递增的 值 填充一个范围 |
| accumulate | 范围内元素求和 |
| inner_product | 2个范围的 元素的 内积 |
| adjacent_difference | 范围内 各相邻元素之间的 差 |
| partial_sum | 以每个元素 为结尾的 prefix array 的和。 |
| reduce (C++17) | 类似 accumulate，但不依序执行 |
| exclusive_scan (C++17) | 类似std::partial_sum, 第i个和中排除第i个输入 |
| inclusive_scan (C++17) | 类似partial_sum，第i个和中包含第i个输入 |
| transform_reduce (C++17) | 应用一个函数对象，然后 reduce |
| transform\_exclusive\_scan(C++17) | 应用一个函数对象，然后 进行 exclusive_scan |
| transform\_inclusive\_scan(C++17) | 。   |

* * *

## 未初始化内存上的操作

`<memory>`

|     |     |
| --- | --- |
| uninitialized_copy | 将范围内的对象复制到 未初始化的内存区域 |

uninitialized\_copy\_n
uninitialized_fill
uninitialized\_fill\_n
uninitialized_move
uninitialized\_move\_n
uninitialized\_default\_construct
uninitialized\_default\_construct_n
uninitialized\_value\_construct
uninitialized\_value\_construct_n
destory
destory_n
destory_at
construct_at

==================

https://en.cppreference.com/w/cpp/iterator

# advance

==================

# std::ref

==================

# std::mem_fn

==================

10.3 C++标准库中的并行算法
大多数被执行策略重载的算法都在 <algorithm>和 <numeric>头文件 中
all\_of，any\_of，none\_of，for\_each，for\_each\_n，find，find\_if，find\_end，find\_first\_of，
adjacent\_find，count，count\_if，mismatch，equal，search，search\_n，copy，copy\_n，
copy\_if，move，swap\_ranges，transform，replace，replace\_if，replace\_copy，
replace\_copy\_if，fill，fill\_n，generate，generate\_n，remove，remove\_if，remove\_copy，
remove\_copy\_if，unique，unique\_copy，reverse，reverse\_copy，rotate，rotate_copy，
is\_partitioned，partition，stable\_partition，partition\_copy，sort，stable\_sort，
partial\_sort，partial\_sort\_copy，is\_sorted，is\_sorted\_until，nth_element，merge，
inplace\_merge，includes，set\_union，set\_intersection，set\_difference，
set\_symmetric\_difference，is\_heap，is\_heap\_until，min\_element，max_element，
minmax\_element，lexicographical\_compare，reduce，transform_reduce，</numeric></algorithm>

exclusive\_scan，inclusive\_scan，transform\_exclusive\_scan，transform\_inclusive\_scan和

adjacent_difference

==================
https://en.cppreference.com/w/cpp/language/object

==================

==================

==================
https://zh.cppreference.com/w/cpp/language/namespace
命名空间

==================

https://zh.cppreference.com/w/cpp/language/lookup

# 名字查找

是 当程序中 出现一个 名字时， 将其 与 引入它的声明 联系 起来的过程。

例如，为了编译 `std::cout<<std::endl;` 编译器进行了：

1.  名字 std 的 无限定 的名字查找，找到了 头文件 <iostream>中的 命名空间 std 的声明</iostream>
2.  名字 cout 的 有限定 的名字查找，找到了命名空间 std 中的 一个 变量声明
3.  名字 endl 的 有限定 名字查找，找到了 std 中的 一个函数模板声明
4.  名字 operator<< 的 实参依赖查找，找到了 std 中的 多个 函数模板声明 ， 以及名字 std::ostream::operator<< 的有限定名字查找，找到 std::ostream 类中的 多个成员函数声明

对于函数 和函数模板的名字，
名字查找 可以将 同一个 名字 和 多个声明 联系起来，
而且 可能从 实参依赖查找 中找到 额外的 声明。
还会进行 模板实参推导，并将 声明的 集合 交给 重载决议，由它选择 所要使用的 那个声明。
成员访问的规则 只会在 名字查找 和 重载解析 之后 才被 考虑，如果适用的话。

对于其他所有名字( 变量，命名空间，类 等) ，
名字查找 如果将 同一个 名字 和 多个声明 联系起来 就需要 它们都声明 同一个实体，否则 程序 只有在 只产生单个声明的情况下 才能编译。

对某个作用域中的名字 进行查找 将寻找到 该名字 的所有声明， 但是有一种例外，被称作 "struct hack" 或 "类型/非类型名字隐藏"： 同一个作用域中，某个名字的 一些出现 可以 代表 非 typedef 的 class/struct/union/enum 声明，而 其他出现 要么全都代表同一个变量，非静态数据成员 或 枚举项， 要么全都代表 可能重载的 函数 或函数模板名。 此情况下 无错误，但是 类型名 在查找中 被隐藏。(代码必须使用 详述类型说明符 来访问它)

查找的类型
如果名字 紧跟在 作用域解析符 ::， 或 可能跟在 :: 之后的消歧义关键字 template 的右侧，参见： 有限定的名字查找
否则 参见 ： 无限定的名字查找 (其中对于 函数名，还包括 ADL)

* * *

https://zh.cppreference.com/w/cpp/language/qualified_lookup

## 有限定的名字查找

出现在 作用域解析符:: 右侧 的名字 是 限定名。限定名可能代表的是：

- 类的成员 (包括 静态 和 非静态函数，类型 和 模板等)
- 命名空间的 成员 (包括 其他的 命名空间)
- 枚举项

如果 :: 左边为空， 那么查找过程 只会 考虑 ==全局命名空间作用域== 中作出 ( 或 通过 ==using 声明 引入到 全局== 命名空间中 ) 的声明。这使得 即使 ==被 局部声明 隐藏 的名字== 也能够 被访问：

```C++
#include <iostream>

int main()
{
     struct std {};

     std::cout << "失败\n";    // 错误：对 'std' 的无限定查找找到结构体
     ::std::cout << "成功\n"; // 正确：::std 找到命名空间 std
}
```

只有 完成了 对 :: 左边 的名字的查找 (除非 左边用到了 decltype 表达式 或 为空)， 才能对它 右边的 名字 进行 名字查找。 该左侧 查找 可以是 有限定 或 无限定的，取决于 这个名字 左边 是否有 另一个 :: ， 但 它只会 考虑 `命名空间，类类型，枚举 和 能特化为 类型的 模板`。 如果 左边的 名字 指定的不是 命名空间，类，枚举，待决类型， 那么 程序 非 良构：

```C++
struct A
{
     static int n;
};

int main()
{
     int A;
     A::n = 42; // 正确：对 ::  左边的  A 的无限定查找忽略变量
     A b;        // 错误：对 A 的无限定查找找到了变量 A
}

template<int>
struct B : A {};

namespace N
{
     template<int>
     void B();

     int f()
     {
         return B<0>::n; // 错误：N::B<0> 不是类型
     }
}
```

当限定名 是声明符时， 对同一声明符 中 随 该限定名之后， 而非 在它 之前 的名字的 无限查找， 是在成员的 所在 类 或 命名空间 的作用域 中进行的：

```C++
class X {};

constexpr int number = 100;

struct C
{
     class X {};
     static const int number = 50;
     static X arr[number];
};

X C::arr[number], brr[number];     // 错误：对 X 的查找找到  ::X 而不是 C::X
C::X C::arr[number], brr[number]; // OK ：arr 的大小是 50，brr 的大小是 100
```

如果 :: 后随字符 ~ 再跟着 一个标识符 ( 也就是 说 指定了 析构函数 或 伪析构函数)， 那么 该标识符 将在 与 :: 左边的 名字相同的 作用域 中查找。

```C++
struct C { typedef int I; };

typedef int I1, I2;

extern int *p, *q;

struct A { ~A(); };

typedef A AB;

int main()
{
     p->C::I::~I(); // ~ 之后的名字 I 在 :: 前面的 I 的同一个作用域中查找
                    //（也就是说，在 C 的作用域中查找，因此查找结果是 C::I）

     q->I1::~I2();   // 名字 I2 在 I1 的同一个作用域中查找，
                    // 也就是说从当前的作用域中查找，因此查找结果是 ::I2

     AB x;
     x.AB::~AB();    // ~ 之后的名字 AB 在 :: 前面的 AB 的同一个作用域中查找
                    // 也就是说从当前的作用域中查找，因此查找结果是 ::AB
}
```

### 枚举项

如果对左边的名字的查找结果是 枚举 (无论是 有作用域 还是 无作用域)， 那么 右边名字的 查找结果 必须 是 属于 枚举的 一个 枚举项， 否则程序 非良构。

### 类成员

如果 对左边的名字 的查找结果 是 某个类，结构体 或 联合体的 名字， 那么 :: 右边 的名字 在 该类 ，结构体，联合体 的作用域 中 进行查找 (因此，可能找到 该类 或 它的 基类的 成员声明) ，但有 以下例外情况：

析构函数 按如上所述 进行查找 (即 在 :: 左侧 的名字的 作用域 中查找)
用户定义转换 函数名 中的 转换类型标识 首先 在 该类类型 的 作用域中查找。 如果没有找到， 那么 就在 当前作用域中 查找 该名字
模板实参 中使用的名字，在当前 作用域 中查找 (而非 在模板名 的作用域 中查找)
using 声明中的名字，还考虑 在 当前 作用域中 声明的 变量，数据成员，函数 或 枚举项 所隐藏的 类 或枚举名。

如果 :: 右边 所 指名 的 和它左边 相同的类，那么 右边的名字 代表 该类的 构造函数。 这种限定名 只能用在 构造函数 的声明 以及 引入 继承 构造函数的 using 声明中。 在 所有忽略 函数名的 查找过程中 (即 在查找 :: 左边的名字，或查找 详述类型说明符 或 基类说明符 中的 名字时)， 将同样的 语法解释成 注入类名：

```C++
struct A { A(); };

struct B : A { B(); };

A::A() {} // A::A 指定的是构造函数，用于声明
B::B() {} // B::B 指定的是构造函数，用于声明

B::A ba;   // B::A（在 B 的作用域中查找）指定的是类型 A
A::A a;    // 错误：A::A 不是类型

struct A::A a2; // 正确：详述类型说明符中的查找是忽略函数的，
                 // 因此 A::A 在 A 的作用域中查找，于是指定的是类 A
                 // （也就是它的注入类名）
```

有限定名字 查找可以用来访问 被嵌套声明 或被派生类 隐藏了的类成员。对有限定的成员函数的调用绝不会是虚调用：

```C++
struct B { virtual void foo(); };

struct D : B { void foo() override; };

int main()
{
     D x;
     B& b = x;

     b.foo();     // 调用 D::foo（虚调用派发）
     b.B::foo(); // 调用 B::foo（静态调用派发）
}
```

### 命名空间的成员

如果 :: 左侧 的名字 代表了 命名空间，或 :: 左侧为空 (代表 全局命名空间)， 那么 :: 右边 的名字 就在 这个命名空间的 作用域中 进行查找， 但有以下特例：

在模板实参中 使用的名字 在当前作用域中查找

```C++
namespace N
{
     template<typename T>
     struct foo {};

     struct X {};
}

N::foo<X> x; // 错误：X 的查找结果是 ::X 而不是 N::X
。。这个X是说 foo<X> 中的 X
```

在命名空间 N 中进行 有限定查找时，首先要 考虑 处于 N 之中的所有声明，以及 处于 N 的 ==内联==命名空间成员 (并且 传递性地包括 它们的 内联命名空间) 之中 的所有声明。 如果这个集合 中没有找到 任何声明，那么 再考虑在 N 和N 的所有传递性 的内联命名空间成员中 发现的 所有 using 指令 指明 的 命名空间 中的 声明。 这条规则 是 递归实施的

```C++
int x;

namespace Y
{
     void f(float);
     void h(int);
}

namespace Z
{
     void h(double);
}

namespace A
{
     using namespace Y;
     void f(int);
     void g(int);
     int i;
}

namespace B
{
     using namespace Z;
     void f(char);
     int i;
}

namespace AB
{
     using namespace A;
     using namespace B;
     void g();
}

void h()
{
     AB::g();   // 在 AB 中查找，找到了 AB::g 并且选择了 AB::g(void)
               // （没有在 A 和 B 中查找）

     AB::f(1); // 首先在 AB 中查找，没有找到 f
               // 然后在 A 和 B 中查找
               // 找到了 A::f 和 B::f（但没有在 Y 中查找，因此不考虑 Y::f）
               // 重载决议选中 A::f(int)

     AB::x++;   // 首先在 AB 中查找，没有找到 x
               // 然后在 A 和 B 中查找。没有找到 x
               // 然后在 Y 和 Z 中查找。还是没有找到 x：这是一个错误

     AB::i++;   // 在 AB 中查找，没有找到 i
               // 然后在 A 和 B 中查找。找到了 A::i 和 B::i：这是一个错误

     AB::h(16.8); // 首先在 AB 中查找：没有找到 h
                  // 然后在 A 和 B 中查找。没有找到 h
                  // 然后在 Y 和 Z 中查找。
                  // 找到了 Y::h 和 Z::h。重载决议选中 Z::h(double)
}
```

同一个声明 可以被多次找到

```C++
namespace A { int a; }

namespace B { using namespace A; }

namespace D { using A::a; }

namespace BD
{
     using namespace B;
     using namespace D;
}

void g()
{
     BD::a++; // OK ： 通过 B 和 D 找到同一个 A::a
}
```

* * *

https://zh.cppreference.com/w/cpp/language/unqualified_lookup

## 无限定的名称查找

没有出现在 作用域解析符:: 右边的名字 是 无限定名，对它的名字查找按下文所述==检查各个作用域==，只要找到任何种类的至少一个声明就停止查找，即不再继续检查别的作用域。 (注意，某些语境中的 查找会忽略一些声明，如 对 用在 :: 左边的 名字 的查找 会忽略 函数，变量，枚举项的 声明，而对用作 基类 说明符 的名字的查找 会忽略 所有的 非类型的声明)

为了进行无限定的名字查找，来自 using 指令 所 指明的 命名空间 中的 所有声明，都被当做 如同 处于同时 直接 或间接包含 这条 using 指令 和 所 指明的 命名空间的 最内层 的 外层命名空间 之中。

对函数调用运算符 左侧 所使用的名字 (等价地 也包括 表达式中的运算符) 所进行的 无限定名字查找，在 实例依赖查找ADL 中说明

### 文件作用域

对于在 全局(顶层命名空间) 作用域中，且 在任何函数，类， 用户声明的 命名空间 ==之外== 使用的名字，检查 全局作用域中 本次 名字使用之前的 部分。

```C++
int n = 1;      // n 的声明
int x = n + 1; // OK：找到 ::n

int z = y - 1; // 错误：查找失败
int y = 2;      // y 的声明
```

### 命名空间作用域

对于 在 用户声明的 ==命名空间中==，且在 任何函数 或 类==之外== 所 使用的名字，首先在 该命名空间 中 本次名字使用之前 的部分， 然后 查找 外围 命名空间 在 声明 该命名空间 之前的 部分，依次类推，直到 抵达 全局命名空间。

```C++
int n = 1; // 声明

namespace N
{
     int m = 2;

     namespace Y
     {
         int x = n; // OK，找到 ::n
         int y = m; // OK，找到 ::N::m
         int z = k; // 错误：查找失败
     }

     int k = 3;
}
```

### 在命名空间外进行定义

当某个命名空间成员 的变量 在 该命名==空间外被定义==时，该 定义中用到的 名字 的查找 会以 与在命名空间之内 使用的名字相同的方式 进行

```C++
namespace X
{
     extern int x; // 声明，不是定义
     int n = 1;     // 找到第一个
};

int n = 2;         // 找到第二个
int X::x = n;      // 找到了 X::n，设 X::x 为 1
```

### 非成员函数的定义

在某个 用户声明的 或者 全局的 命名空间 的成员函数 定义中，对于在 该函数 的函数体 或者 作为 默认实参的一部分 而使用的名字，首先会 查找 包含 这次名字 使用的块 中 这次使用之前的 部分，然后查找 外围块 在该块 开头之前的 部分，依次类推， 直到 抵达 函数体 对应的块。  然后查找 声明该函数的 作用域 在 该函数 ==定义 (不是声明) 之前的部分==，再然后查找 外围作用域中 函数的 **定义之前**的 部分，以此类推。

```C++
namespace A
{
     namespace N
     {
         void f();
         int i = 3; // 找到第三个（如果第二个没找到）
     }

     int i = 4;      // 找到第四个（如果第三个没找到）
}

int i = 5;          // 找到第五个（如果第四个没找到）

void A::N::f()
{
     int i = 2;      // 找到第二个（如果第一个没找到）

     while (true)
     {
         int i = 1; // 找到第一个：查找结束
         std::cout << i;
     }
}

// int i;           // 找不到这个

namespace A
{
     namespace N
     {
         // int i;   // 找不到这个
     }
}
```

### 类的定义

对于在 类的定义 中使用的 名字( 包括 基类说明符 和 嵌套类定义)， 当出现于 成员函数体，成员函数的 默认实参，成员函数的 异常规定，默认成员初始化器 ( 其中该成员 可能属于 定义在 外围类体内的 嵌套类) ==以外== 的任何位置时，要在下列作用域中查找：

1.  类体之中 直到 这次 使用点之前的部分
2.  (各个) 基类 的 整个类体，找不到声明时， 递归 到 基类的 基类中
3.  当该类 是 嵌套类时， 它的 外围类 的类体中 该类的 定义 之前的 部分 以及 外围类的 (各个)基类
4.  当该类是 局部类 或 局部类的 嵌套类时，定义了 该类 的块作用域 中直到 该类的 定义点 之前的部分
5.  当该类是 某个 命名空间的 成员，某个命名空间 的 成员类的 嵌套类，或者 某个命名空间 的成员函数的 局部类时，分别在该 命名空间 作用域中 该类，该类所在的 外围类，或者该类所在的 外围函数 的 定义 之前的 部分进行查找；然后查找 外围 命名空间的这些部分，依次类推，直到 抵达 全局命名空间

对于==友元声明==，确定 它所 指代 的 先前声明的 实际的查找 按上述方式 继续，除了在 最内层的 外围命名空间后 停止。

```C++
namespace M
{
     // const int i = 1;                 // 找不到这个

     class B
     {
         // static const int i = 3;      // 找到了第三个（但无法通过访问检查）
     };
}

// const int i = 5;                     // 找到了第五个

namespace N
{
     // const int i = 4;                 // 找到了第四个

     class Y : public M::B
     {
         // static const int i = 2;      // 找到了第二个(c)

         class X
         {
             // static const int i = 1; // 找到了第一个
             int a[i]; // i 的使用
             // static const int i = 1; // 找不到这个
         };

         // static const int i = 2;      // 找不到这个
     };

     // const int i = 4;                 // 找不到这个
}

// const int i = 5;                     // 找不到这个
```

### 注入类名

对于在 类或类模板 或 其派生类或类模板 中所 使用的 这个类 或类模板的名字，无限定名字查找 将 会找到 当前 进行定义的类，如同 它的名字 是由 成员声明( 以公开成员访问) 的方式 所引入的一样。

### 成员函数的定义

对于在 成员函数体，成员函数的默认实参，成员函数的异常说明 或 默认成员初始化器 中 所使用的名字，进行查找的 各个作用域 和 类的定义中 相同，但是要考虑 ==这个类的整个作用域==，而==不只是== 使用了这个 名字的==声明之前==的 部分。对于嵌套类来说，它的 外围类的 整个类体 都要进行查找。

```C++
class B
{
     // int i;          // 找到第三个
};

namespace M
{
     // int i;          // 找到第五个

     namespace N
     {
         // int i;      // 找到第四个

         class X : public B
         {
             // int i; // 找到第二个
             void f();
             // int i; // 也找到第二个
         };

         // int i;      // 找到第四个
     }
}

// int i;              // 找到第六个

void M::N::X::f()
{
     // int i;          // 找到第一个
     i = 16;
。。这里的意思 就是 这句 i=16, 会找到 哪个 i

     // int i;          // 找不到这个
}

namespace M
{
     namespace N
     {
         // int i;      // 找不到这个
     }
}
```

无论哪种方式，当检查 派生类 的基类时， 需要遵守以下规则，它们有时被称为 虚继承中的 优先性：
C++11前

当子对象 A 是子对象 B 的基类子对象时，子对象 B 中找到的成员的名字将隐藏掉子对象 A 中相同的成员名。（但要注意这并不会隐藏继承晶格中 B 的基类以外的任何其他的 A 的非虚副本。这条规则只有在虚继承时有效。）由 using 声明所引入的名字会被当成是包含这个声明的类中的名字来处理。检查各个基类之后，它的结果集合必须包含来自同一个类型的子对象的静态成员的声明或者来自同一个子对象的非静态成员的声明。

C++11开始

构造一个查找集合，它由一些声明和在其中找到了这些声明的子对象所构成。using 声明被替换成它们所代表的成员，类型声明（包括注入类名）被替换成它们所代表的类型。如果类 C 在它的作用域中使用了这个名字，那么首先检查 C。如果 C 中的声明列表为空，那么对它的每个直接基类 Bi 各自构造查找集合（当 Bi 自身也有基类时，递归地应用这条规则）。构造完成后，将每个直接基类的查找集合根据以下规则合并为 C 的查找集合：

-        如果 Bi 中的声明集合为空，那么它会被丢弃
-        如果目前所构建的 C 的查找集合仍为空，那么它会被替换成 Bi 的查找集合
-        如果 Bi 的查找集合中的每个子对象都是已经加入到 C 的查找集合中至少一个子对象的基类子对象，那么丢弃 Bi 的查找集合
-        如果已经加入到 C 的查找集合中的每个子对象都是 Bi 的查找集合中至少一个子对象的基类子对象，那么 C 的查找集合会被丢弃并被替换成 Bi 的查找集合
-        否则，如果 Bi 和 C 中的声明集合不同，那么合并的结果有歧义：C 的新查找集合包含一个无效声明以及之前合并入 C 中的各子对象和由 Bi 所引入的子对象的并集。这个无效的查找集合如果在之后被丢弃了，那么它并不会导致错误。
-        否则，C 的新查找集合具有共同的声明集合和之前合并入 C 和由 Bi 所引入的子对象的并集

```C++
struct X { void f(); };

struct B1: virtual X { void f(); };

struct B2: virtual X {};

struct D : B1, B2
{
     void foo()
     {
         X::f(); // OK，调用了 X::f（有限定查找）
         f();     // OK，调用了 B1::f（无限定查找）
     }
};
```

// C++98 规则：B1::f 隐藏 X::f，因此即使 X::f 可以通过
// B2 从 D 访问，它也不能被 D 中的名字查找所找到。

// C++11 规则：在 D 中对 f 的查找集合并未找到任何东西，继续处理它的基类
//  在 B1 中对 f 的查找集合找到了 B1::f 并完成
// 合并时替换了空集，此时在 C 中 对 f 的查找集合包含 B1 中的 B1::f
//  在 B2 中对 f 的查找集合并未找到任何东西，继续处理它的基类
//    在 X 中对 f 的查找找到了 X::f
//  合并时替换了空集，此时在 B2 中对 f 的查找集合包含 X 中的 X::f
// 当向 C 中合并时发现在 B2 的查找集合中的每个子对象（X）都是
// 已经合并的各个子对象（B1）的基类，因此 B2 的集合被丢弃
// C 剩下来的就是在 B1 中所找到的 B1::f
// (如果使用 struct D : B2, B1，那么最后的合并将会*替换掉*
//  C 此时已经合并的 X 中的 X::f，因为已经加入到 C 中的每个子对象（就是 X）
//  都是新集合（B1）中的至少一个子对象的基类，
//  它们的最终结果是一样的：C 的查找集合只包含在 B1 中找到的 B1::f）

如果无限定的名字查找找到了 B 的静态成员，B 的嵌套类型或在 B 中声明的枚举项，那么即便在所检查的类的继承树中有多个 B 类型的非虚基类子对象，它也没有歧义：

```C++
struct V { int v; };

struct A
{
     int a;
     static int s;
     enum { e };
};

struct B : A, virtual V {};
struct C : A, virtual V {};
struct D : B, C {};

void f(D& pd)
{
     ++pd.v;        // OK：只有一个 v，因为只有一个虚基类子对象
     ++pd.s;        // OK：只有一个静态的 A::s，即便在 B 和 C 中都找到了它
     int i = pd.e; // OK：只有一个枚举项 A::e，即便在 B 和 C 中都找到了它
     ++pd.a;        // 错误，有歧义：B 中的 A::a 和 C 中的 A::a
}
```

### 友元函数的定义

#### (所有的都是从定义处向外看。不从声明处看。)

对于 在授予 友元关系 的类体之中 的 友元函数 的定义中所使用的名字，无限定的名字查找按照与成员函数相同的方式进行。对于定义于类体之外的友元函数中所使用的名字，无限定的名字查找按照与命名空间中的函数相同的方式进行。

```C++
int i = 3;                      // f1 找到的第三个，f2 找到的第二个

struct X
{
     static const int i = 2;     // f1 找到的第二个，f2 找不到这个

     friend void f1(int x)
     {
         // int i;               // 找到第一个
         i = x;                  // 找到并修改 X::i
     }

     friend int f2();

     // static const int i = 2; // f1 在类作用域中的任何地方找到第二个
};

void f2(int x)
{
     // int i;                   // 找到第一个
     i = x;                      // 找到并修改 ::i
}
```

### 友元函数的声明

对于在使来自其他类的成员函数为友元 的友元函数声明 的声明符中所使用的名字，如果该名字不是声明符中的标识符中的任何模板实参的一部分，那么无限定的查找首先检查成员函数所在类的整个作用域。如果在这个作用域中没有找到（或者如果这个名字是声明符中的标识符中的模板实参的一部分），那么继续以如同对授予友元关系的类的成员函数进行查找的方式继续查找。

```C++
template<class T>
struct S;

// 这个类的成员函数被作为友元
struct A
{
     typedef int AT;

     void f1(AT);
     void f2(float);

     template<class T>
     void f3();

     void f4(S<AT>);
};

// 这个类为 f1，f2 和 f3 授予友元关系
struct B
{
     typedef char AT;
     typedef float BT;

     friend void A::f1(AT);     // 对 AT 的查找找到的是 A::AT（在 A 中找到 AT）
     friend void A::f2(BT);     // 对 BT 的查找找到的是 B::BT（在 A 中找不到 AT）
     friend void A::f3<AT>();   // 对 AT 的查找找到的是 B::AT （不在 A 中进行查找，
                               //      因为 AT 在声明符中的标识符 A::f3<AT> 中）
};

// 这个类模板为 f4 授予友元关系
template<class AT>
struct C
{
     friend void A::f4(S<AT>); // 对 AT 的查找找到的是 A::AT
                               // （AT 不在声明符中的标识符 A::f4 中）
};
```

### 默认实参

对于在函数声明的默认实参中所使用的名字，或者在构造函数的成员初始化器的 表达式 部分所使用的名字，在检查它外围的块、类或命名空间作用域之前，首先会检查函数形参的名字：

```C++
class X
{
     int a, b, i, j;
public:
     const int& r;

     X(int i): r(a),       // 将 X::r 初始化为指代 X::a
               b(i),       // 将 X::b 初始化为形参 i 的值
               i(i),       // 将 X::i 初始化为形参 i 的值
               j(this->i) // 将 X::j 初始化为 X::i 的值
     {}
}

int a;
int f(int a, int b = a); // 错误：对 a 的查找找到了形参 a，而不是 ::a
                          // 但在默认实参中不允许使用形参
```

### 静态数据成员的定义

对于在静态数据成员的定义中所使用的名字，它的查找按照与对成员函数的定义中所使用的名字相同的方式进行。

```C++
struct X
{
     static  int x;
     static const int n = 1; // 找到第一个
};

int n = 2;                   // 找到第二个
int X::x = n;                // 找到了 X::n，将 X::x 设为 1 而不是 2
```

### 枚举项的声明

对于在枚举项的声明的初始化器部分中所使用的名字，在无限定的名字查找处理它外围的块、类或命名空间作用域之前，会首先检查同一个枚举中之前所声明的枚举项。

```C++
const int RED = 7;

enum class color
{
     RED,
     GREEN = RED + 2, // RED 找到了 color::RED，而不是 ::RED，所以 GREEN = 2
     BLUE = ::RED + 4 // 有限定查找找到 ::RED，BLUE = 11
};
```

### 函数 try 块的 catch 子句

对于在函数 try 块的 catch 子句中所使用的名字，它的查找按照如同对在函数体的最外层块的最开始处使用的名字一样进行（特别是，函数形参是可见的，但这个最外层块中声明的名字则不可见）。

```C++
int n = 3; // 找到第三个
int f(int n = 2)     // 找到第二个

try
{
     int n = -1;      // 找不到这个
}
catch(...)
{
     // int n = 1;    // 找到第一个
     assert(n == 2); // 对 n 的查找找到了函数形参 f
     throw;
}
```

### 重载的运算符

对于在表达式中所使用的运算符（比如在 a + b 中使用的 operator+），它的查找规则和对在如 operator+(a, b) 这样的显式函数调用表达式中所使用的运算符是有所不同的：当处理表达式时要分别进行两次查找：对非成员的运算符重载，也对成员运算符重载（对于同时允许两种形式的运算符）。然后按重载解析所述将这两个集合与内建的运算符重载以平等的方式合并到一起。而当使用显式函数调用语法时，会进行常规的无限定名字查找：

```C++
struct A {};
void operator+(A, A);   // 用户定义的非成员 operator+

struct B
{
     void operator+(B); // 用户定义的成员 operator+
     void f();
};

A a;

void B::f() // B 的成员函数定义
{
     operator+(a, a); // 错误：在成员函数中的常规名字查找
                      // 找到了 B 的作用域中的 operator+ 的声明
                      // 并于此停下，而不会达到全局作用域

     a + a; // OK：成员查找找到了  B::operator+，非成员查找
            // 找到了 ::operator+(A, A)，重载决议选中了 ::operator+(A, A)
}
```

### 模板的定义

对于在模板的定义中所使用的非待决名，当检查该模板的定义时将进行无限定的名字查找。在这个位置与声明之间的绑定并不会受到在实例化点可见的声明的影响。而对于在模板定义中所使用的待决名，它的查找会推迟到得知它的模板实参之时。此时，ADL 将同时在模板的定义语境和在模板的实例化语境中检查可见的具有外部连接的 (C++11 前)函数声明，而非 ADL 的查找只会检查在模板的定义语境中可见的具有外部连接的 (C++11 前)函数声明。（换句话说，在模板定义之后添加新的函数声明，除非通过 ADL 否则仍是不可见的。）如果在 ADL 查找所检查的命名空间中，在某个别的翻译单元中声明了一个具有外部连接的更好的匹配声明，或者如果当同样检查这些翻译单元时其查找会导致歧义，那么行为未定义。无论哪种情况，如果某个基类取决于某个模板形参，那么无限定名字查找不会检查它的作用域（在定义点和实例化点都不会）。

```C++
void f(char); // f 的第一个声明

template<class T>
void g(T t)
{
     f(1);     // 非待决名：名字查找找到了 ::f(char) 并在此时绑定
     f(T(1)); // 待决名：查找推迟
     f(t);     // 待决名：查找推迟
//   dd++;     // 非待决名：名字查找未找到声明
}

enum E { e };
void f(E);    // f 的第二个声明
void f(int); // f 的第三个声明
double dd;

void h()
{
     g(e);   // 实例化 g<E>，此处
            // 对 'f' 的第二次和第三次使用
            // 进行查找并找到了 ::f(char)（常规查找）和 ::f(E)（ADL）
            // 然后重载解析选择了 ::f(E)。
            // 这调用了 f(char)，然后两次调用 f(E)

     g(32); // 实例化 g<int>，此处
            // 对 'f' 的第二次和第三次使用
            // 进行了查找仅找到了 ::f(char)
            // 然后重载解析选择了 ::f(char)
            // 这三次调用了 f(char)
}

typedef double A;

template<class T>
class B
{
     typedef int A;
};

template<class T>
struct X : B<T>
{
     A a; // 对 A 的查找找到了 ::A (double)，而不是 B<T>::A
};
```

==================
https://zh.cppreference.com/w/cpp/language/overload_resolution

# 重载决议

为了编译函数调用，编译器 必须首先 进行 名字查找， 对函数 可能涉及 实参依赖查找， 而 对于 函数模板 可能 后随 模板实参推导。 如果这些步骤 产生了 多个 候选函数，那么 需要 进行 重载决议 选择 实际调用的 函数。

通常来说， 调用的函数 是 各形参 和 各实参 之间的 匹配 最紧密 的 候选函数
关于其他可以出现重载函数名的语境，见重载函数的地址。
如果函数无法被重载决议选择（例如它是有未被满足的约束的模板化实体），那么不能指名或再使用它。

细节

```text
重载决议前，将 名字查找 和 模板实参推导 所选择的 函数 组成 候选函数 的集合。

对于函数模板， 为找到能 在此时 使用的 模板实参值 (如果存在) 会进行 模板实参推导 和 显式模板实参检查：

如果两者 都成功， 那么 找到的 模板实参 会用来 合成 对应的 函数模板特化的 声明，那么这些特化会加入候选集，并且 在决胜规则 指定的 情况以外 都会被 视为 非模板函数

如果 模板实参推导失败 或 合成的函数模板特化 非良构，那么 这些函数 不会加入 候选集。

如果一个名字 指代 一个 或多个 函数模板，并且 同时指代 重载的 非模板函数，那么 这些函数 和 从模板生成的 特化 都是 候选。

C++20起，如果构造函数模板或转换函数模板拥有在推导后它恰好是值待决的条件性 explicit 说明符，且语境要求非 explicit 的候选而所生成的候选是 explicit 的，那么从候选集中移除它。

C++11起，候选函数列表中 始终不包含 被定义为 弃置 的 预置移动构造函数 和 移动赋值运算符。
在构造派生类对象时，候选函数 列表中不包含 继承的 复制 和 移动 构造函数。
```

## 隐式对象形参

如果有候选函数 是 除构造器 外 且 没有 显式对象形参(C++23起) 的 成员函数 (静态或非静态)，那么将它当做 如同它有一个 额外 形参( 隐式对象形参)， 代表调用函数 所用的 对象，并出现在其 首个实参之前。

类似地，调用成员函数 所用的 对象 会作为 隐含对象实参 前 附于 实参列表。
对于类X 的成员函数，隐含对象形参 的 类型 受 成员函数的 cv限定 和 引用限定 影响，如 成员函数 中所述。

就确定 隐式对象形参类型 而言，用户定义转换函数被认为 隐含对象实参 的成员。
就确定 隐式对象形参类型 而言，由 using 声明引入到 派生类 中的 成员函数 被认为 的 派生类的 成员。

对于 静态成员函数，它的隐式对象形参 被认为 匹配任何对象： 不检验它的类型，且不会为 它 尝试 转换序列。

对于 重载决议的 剩余部分，隐含对象实参 和 其他实参 不可辨别，但下列特殊规则适用于 隐式对象形参：
不能对 隐式对象形参 运用 用户定义转换
右值 能绑定到 非 const 的隐式对象形参 (除非是对引用限定的成员函数)， 且不影响 隐式转换的 等级。

```C++
struct B { void f(int); };
struct A { operator B&(); };

A a;
a.B::f(1); // 错误：不能对隐式对象形参运用用户定义转换
static_cast<B&>(a).f(1); // OK
```

## 候选函数

使用重载决议 的 每种语境 都有 它 独特的 准备 候选函数集合 和 实参列表：

- 调用具名函数
    
- 调用类对象
    
- 调用重载运算符
    
- 由构造函数初始化
    
- 通过转换进行复制初始化
    
- 通过转换进行非类初始化
    
- 通过转换进行引用初始化
    
- 列表初始化
    

## 可行函数

给定上述方式 构造的 候选函数集合后，重载决议的 下一个步骤 是 检验 各个实参和形参，并将 集合 缩减为 可行函数 的集合。
为了被 包含在 可行函数集合中，候选函数必须满足 下列条件：
如果有 M 个实参，那么刚好 具有 M 个形参 的 候选函数 可行
如果 有M 个实参 且 候选函数的形参 少于 M 个， 但是有一个 省略号形参， 可行
如果有 M 个实参， 且 形参 多于M 个，但是 M+1 个形参 和 后续形参 都具有 默认实参， 可行。 对于 剩余的 重载决议，形参列表 被 截断到M
C++20起，如果函数 拥有 关联的 约束，那么必须满足它。
对于每个实参，必须至少存在 一个 隐式转换序列 将它转换为 对应的形参。
如果任何形参具有引用类型，那么这一步负责引用绑定：如果右值实参对应非 const 左值引用形参，或左值实参对应右值引用形参，那么函数不可行。

## 最佳可行函数

对于每对可行函数 F1 和 F2，对从第 i 实参到第 i 形参的转换做排行，以确定哪一个更好（除了首个实参，静态成员函数的隐式对象实参在排行上没有影响）。
如果 F1 的所有实参的隐式转换不劣于 F2 的所有实参的隐式转换，且满足下列条件，那么确定 F1 是优于 F2 的函数：
至少存在一个 F1 的实参，它的隐式转换优于 F2 的该实参的对应的隐式转换
或若非如此，（只在通过转换进行非类初始化的语境中，）从 F1 的返回类型到要初始化的类型的标准转换序列优于从 F2 的返回类型到该类型的标准转换序列

C++11起，或若非如此，（仅在对函数类型的引用进行直接引用绑定所作的，通过转换函数进行初始化的语境中，）F1 的返回类型是与正在初始化的引用相同种类的引用（左值或右值），而 F2 的返回类型不是

或若非如此，F1 是非模板函数而 F2 是模板特化
或若非如此，F1 与 F2 都是模板特化，且按照模板特化的偏序规则，F1 更特殊
C++20起，或若非如此，F1 与 F2 是拥有相同形参类型列表的非模板函数，且按照约束的偏序规则，F1 比 F2 更受约束
C++11起，或若非如此，F1 是类 D 的构造函数，F2 是 D 的基类 B 的构造函数，且对应每个实参的 F1 和 F2 的形参均具有相同类型
20起，或若非如此，F2 是重写的候选而 F1 不是，
20起，或若非如此，F1 和 F2 都是重写候选，但 F2 是带逆序形参的合成重写候选而 F1 不是
17+，或若非如此，F1 是从用户定义推导指引所生成的而 F2 不是
17+，或若非如此，F1 是复制推导候选而 F2 不是
17+，或若非如此，F1 是从非模板构造函数生成而 F2 是从构造函数模板生成

## 隐式转换序列的排行

1.  准确匹配：不要求转换、左值到右值转换、限定性转换、函数指针转换、 (C++17 起)类类型到相同类的用户定义转换
2.  提升：整型提升、浮点提升
3.  转换：整型转换、浮点转换、浮点整型转换、指针转换、成员指针转换、布尔转换、派生类到它的基类的用户定义转换

## 列表初始化中的隐式转换序列

在列表初始化中，实参是 花括号初始化器列表，但它不是表达式，所以到就重载决议而言的形参类型的隐式转换序列以下列规则决定：

==================

==================
https://zh.cppreference.com/w/cpp/language/class

# class

类说明符拥有下列语法：

```text
类关键词 属性(可选)  类头名  final(可选) 基类子句(可选) { 成员说明 }    // 定义具名类
类关键词 属性(可选)  基类子句 { 成员说明 }                   // 定义无名类
```

|     |     |
| --- | --- |
| 类关键词 | class，struct 和 union 之一。<br>除了默认成员访问和默认基类访问之外，关键词 struct 和 class 是等同的。如果关键词是 union，那么声明引入一个联合体类型。 |
| 属性  | (C++11 起) 任意数量的属性，可以包含 alignas 指定符 |
| 类头名 | 所定义的类的名字，可以有限定 |
| final | (C++11 起) 出现时，该类无法被派生 |
| 基类子句 | 一个或多个基类以及各自所用的继承模型的列表（见派生类） |
| 成员说明 | 访问说明符、成员对象及成员函数的声明和定义的列表（见下文） |

## 前置声明

下列形式的声明

```text
类关键词 属性 标识符 ;
```

声明一个将稍后在此作用域定义的类类型。直到定义出现前，此类名具有不完整类型。这允许类之间互相引用：

```C++
class Vector; // 前置声明

class Matrix
{
     // ...
     friend Vector  operator*(const Matrix&, const Vector&);
};

class Vector
{
     // ...
     friend Vector operator*(const  Matrix&, const Vector&);
};
```

而且如果特定的源文件只用到该类的指针和引用，使用前置声明也可以减少 #include 依赖：

```C++
// 在 MyStruct.h  中
#include <iosfwd> // 含有 std::ostream 的前置声明

struct MyStruct
{
     int value;
     friend std::ostream& operator<<(std::ostream& os, const S& s);
     // 它的定义在 MyStruct.cpp 文件中提供，该文件使用 #include <ostream>
};
```

如果前置声明在局部作用域出现，那么它会隐藏它的外围作用域中可能出现的先前声明的相同名字的类、变量、函数，以及所有其他声明：

```C++
struct s { int a; };
struct s; // 不做任何事（s 已经在此作用域定义）

void g()
{
     struct s; // 新的局部结构体“s”的前置声明
               // 它隐藏全局的结构体 s  直至此块结尾

     s* p;      // 指向局部结构体 s 的指针

     struct s { char* p; }; //  局部结构体 s 的定义
}
```

注意，通过作为其他声明一部分的 详述类型说明符(。。。elaborated type specifier) 也可以引入新的类名，但只有在名字查找无法找到先前声明的有此名的类时才可以。

```C++
class U;

namespace ns
{
     class Y f(class T p); // 声明函数 ns::f  并声明 ns::T 与 ns::Y
     class U f();           //  U 指代 ::U

     // 可以使用到 T 和 Y 的指针及引用
     Y* p;
     T* q;
}
```

## 成员说明

成员说明，或类定义的体，是 花括号 环绕的 任何数量 下列各项 的序列：

1.  具有下列形式的成员声明
    属性(可选) 声明说明符序列(可选) 成员声明符列表(可选) ;

|     |     |
| --- | --- |
| 属性  | (C++11 起) 任意数量的属性 |
| 声明说明符序列 | 说明符的序列。它只能在构造函数，析构函数，以及用户定义转换函数中省略。 |
| 成员声明符列表 | 与 初始化声明符列表相同，但额外允许位域定义、纯说明符和虚说明符（override 或 final） (C++11 起)，并且不允许直接非列表初始化语法。 |

这种声明可以声明 静态 及 非静态 的 数据成员 与 成员函数、成员 typedef、成员枚举以及嵌套类。它也可以是友元声明。

```C++
class S
{
     int d1;             // 非静态数据成员
     int a[10] = {1, 2}; // 带初始化器的非静态数据成员（C++11）
     static const int d2 = 1; // 带初始化器的静态数据成员
     virtual void f1(int) = 0;  // 纯虚成员函数
     std::string d3, *d4, f2(int); // 两个数据成员和一个成员函数
     enum {NORTH, SOUTH, EAST, WEST};
     struct NestedS
     {
         std::string s;
     } d5, *d6;
     typedef NestedS value_type, *pointer_type;
};
```

2.  函数定义，同时声明并定义 成员函数 或者 友元函数 。成员函数 定义之后的 分号 是可选的。所有 定义于 类体之内 的函数 自动为 内联的，除非它们附着到 具名模块 (C++20 起)。

```C++
class M
{
     std::size_t C;
     std::vector<int> data;
public:
     M(std::size_t R, std::size_t C) : C(C), data(R*C) {} // 构造函数定义

     int operator()(size_t r, size_t c) const // 成员函数定义
     {
         return data[r * C + c];
     }

     int& operator()(size_t r, size_t c) // 另一个成员函数定义
     {
         return data[r * C + c];
     }
};
```

3.  访问说明符 public:、protected: 和 private:：

```C++
class S
{
public:
     S();           // 公开的构造函数
     S(const S&);   // 公开的复制构造函数
     virtual ~S(); // 公开的虚析构函数
private:
     int* ptr; // 私有的数据成员
};
```

4.  using 声明：

```C++
class Base
{
protected:
     int d;
};

class Derived : public Base
{
public:
     using Base::d; // 令 Base 的受保护成员 d 成为 Derived  的公开成员
     using Base::Base; // 继承基类的所有构造函数（C++11）
};
```

。。？这个 d 从 protected 变成 public。。。 ok ，没有问题， 本来想说， 访问权限改了，多态怎么办。。 结果这里 是 子类的访问范围 大于 父类的， 所以 多态没有问题。

。。那么 应该不能 将 父类的 protected属性 private 掉。

5.  static_assert 声明：

```C++
template<typename T>
struct Foo
{
     static_assert(std::is_floating_point<T>::value, "Foo<T>: T 必须是浮点数");
};
```

6.  成员模板声明：

```C++
struct S
{
     template<typename T>
     void f(T&& n);

     template<class CharT>
     struct NestedS
     {
         std::basic_string<CharT> s;
     };
};
```

7.  别名声明： C++11 起

```C++
template<typename T>
struct identity
{
     using type =  T;
};
```

8.  成员类模板 的 推导指引：  C++17 起

```C++
struct S
{
     template<class CharT>
     struct NestedS
     {
         std::basic_string<CharT> s;
     };

     template<class CharT>
     NestedS(std::basic_string<CharT>) -> NestedS<CharT>;
};
```

。。？what？

9.  using enum 声明：  C++ 20起

```C++
enum class color { red, orange, yellow };

struct highlight
{
     using enum  color;
};
```

### 局部类

类声明可以在函数体内出现，此时它定义 局部类。这种类的名字只存在于函数作用域中，且无法在函数外访问。
    局部类不能拥有静态数据成员
    局部类的成员函数无链接
    局部类的成员函数必须完全在类体内定义
    除闭包类型以外的 (C++14 起)局部类不能拥有成员模板
    局部类不能拥有友元模板
    局部类不能在类定义内定义友元函数
    函数（包括成员函数）内的局部类的外围函数能访问的名字也可以被该局部类访问
    局部类不能用作模板实参 (C++11 前)

```C++
#include <vector>
#include <algorithm>
#include <iostream>

int main()
{
     std::vector<int> v{1, 2, 3};

     struct Local
     {
         bool operator()(int n, int m)
         {
             return n > m;
         }
     };

     std::sort(v.begin(), v.end(), Local()); // C++11 起

     for (int n : v)
         std::cout << n << ' ';
}
```

==================

https://zh.cppreference.com/w/cpp/language/member_functions

# 非静态成员函数

非静态成员函数是 在 类的 成员说明中**不带 static 或 friend** 说明符声明的 函数。（这些关键词的效果见静态成员函数与友元声明）

```C++
class S
{
     int mf1(); // 非静态成员函数声明
     void mf2() volatile, mf3() &&; // 可以有 cv 限定符或引用限定符
         // 上面的声明与下面分开的两个声明等价：
         // void mf2() volatile;
         // void mf3() &&;

     int mf4() const { return data; } // 可以内联定义
     virtual void mf5() final; // 可以是虚函数，可以使用 final/override
     S() : data(12) {} // 构造函数也是成员函数
     int data;
};
int S::mf1() { return 7; } // 不内联定义就必须在命名空间定义
```

构造函数、析构函数和转换函数的声明语法是特殊的。本页描述的规则可能不适用于这些函数。细节可以参考它们对应的页面。

允许任何函数声明，外加 非静态 成员 函数 专用的语法元素：纯说明符，cv 限定符，引用限定符，final 与 override 说明符 (C++11 起)，以及成员初始化器列表。

可以通过以下方式调用类 X 的非静态成员函数：

1.  对 X 类型的对象使用类成员访问运算符调用
2.  对派生自 X 的类的对象调用
3.  从 X 的成员函数体内直接调用
4.  从派生自 X 的类的成员函数体内调用直接调用

在类型不是 X 或派生自 X 的对象上调用类 X 的非静态成员函数的行为未定义。

在 X 的非静态成员函数的体内，任何解析为 X 或 X 的某个基类的 非类型 非静态 成员的标识表达式 e（例如一个标识符）都会被变换为成员访问表达式 (*this).e（除非它已经是成员访问表达式的一部分）。模板定义语境中不会发生这种变换，因此有时需要明确地对某个名字前附 this->，以使其成为 待决的 名字。

```C++
struct S
{
     int n;
     void f();
};

void S::f()
{
     n = 1; // 变换为 (*this).n = 1;
}

int main()
{
     S s1, s2;
     s1.f(); // 更改 s1.n
}

在  X  的  非  静态  成员  函数  体内，任何解析到  X  或  X  的某个  基类的  静态  成员、枚举项或嵌套类型的  无  限定  标识都会被变换为对应的  有限定标识。

struct S
{
     static int n;
     void f();
};

void S::f()
{
     n = 1; // 变换为 S::n = 1;
}

int main()
{
     S s1, s2;
     s1.f(); // 更改 S::n
}
```

## 有 const 与 volatile 限定符的成员函数

非静态 成员 函数 可以带 cv 限定符序列（const、volatile 或 const 和 volatile 的组合）声明，这些 限定符 在函数声明中的 形参列表 之后出现。带有不同 cv 限定符（或无限定）的函数具有不同类型，从而可以相互重载。

在有 cv 限定符的函数体内，==*this 有同样的 cv 限定==，例如在有 const 限定符的成员函数中只能正常地调用其他有 const 限定符的成员函数。（如果应用了 const_cast，或通过不涉及 this 的访问路径，那么仍然可以调用没有 const 限定符的成员函数。）

```C++
#include <vector>

struct Array
{
     std::vector<int> data;
     Array(int sz) : data(sz) {}

     // const 成员函数
     int operator[](int idx) const
     {                      // this 具有类型 const Array*
         return data[idx]; // 变换为  (*this).data[idx];
     }

     // 非 const 成员函数
     int& operator[](int idx)
     {                      // this 具有类型 Array*
         return data[idx]; // 变换为 (*this).data[idx]
     }
};

int main()
{
     Array a(10);
     a[1] = 1;   // OK：a[1] 的类型是  int&
     const Array ca(10);
     ca[1] = 2;  // 错误：ca[1] 的类型是 int
}
```

## 有引用限定符的成员函数，C++11起

非 静态 成员 函数可以不带引用限定符，带有左值引用限定符（函数名后的 & 记号），或带有右值引用限定符（函数名后的 && 记号）声明。在重载决议中，按下列方式对待类 X 的非静态有 cv 限定符序列的成员函数：

不带引用限定符：隐式对象形参具有到 cv 限定的 X 的左值引用类型，并额外允许绑定到右值隐含对象实参
    左值引用限定符：隐式对象形参具有到 cv 限定的 X 的左值引用类型
    右值引用限定符：隐式对象形参具有到 cv 限定的 X 的右值引用类型

```C++
#include <iostream>

struct S
{
     void f()  &   {  std::cout << "左值\n"; }
     void f()  &&  { std::cout << "右值\n"; }
};

int main()
{
     S s;
     s.f();             // 打印“左值”
     std::move(s).f(); // 打印“右值”
     S().f();           // 打印“右值”
}
```

注意：与 cv 限定性不同，==引用限定性不改变 this 指针的性质==：即使在右值引用限定的函数中，*this 仍是左值表达式
。。
。。之前，重载的标准是， 方法名\+参数列表。 现在是 方法名\+参数列表\+ 调用者是否const + 调用者是否 左值/右值/默认
。。。。方法名+参数列表，应该是 非成员函数的。  不过 也没注意过 成员函数 是什么 重载标准。

## 虚函数和纯虚函数

非静态成员函数可以声明为 虚或 纯虚 函数。

## 显式对象形参   C++23起

非静态成员函数的声明可以通过在第一个形参前附关键词 this 来指定该形参为显式对象形参。

```C++
struct X
{
     void foo(this  X const&  self, int i); // 同 void foo(int i) const &;
//   void foo(int i) const &;              // 错误：已经声明

     void bar(this X self, int i); // 按值传递对象：复制 *this
};
```

。。这有什么用。 本来就有 this 的。  只是 指针，ref，值 的区别。

对于成员函数模板，显式对象形参的类型和值类别可以被推导，因此该语言特性也被称为“推导 this”：

```C++
struct X
{
     template<typename Self>
     void foo(this Self&&, int);
};

struct D : X {};

void ex(X& x, D& d)
{
     x.foo(1);        // Self = X&
     move(x).foo(2); // Self = X
     d.foo(3);        // Self = D&
}
```

这使得成员函数的带 const 限定和不带 const 限定版本只需要一次声明，编译器可根据推导结果选择合适的类外部的实现。

此外，显式对象形参会推导成派生类型，因此可以简化 CRTP：
。。

## CRTP: 奇特重现模板模式

```C++
// 一个 CRTP 特性
struct add_postfix_increment
{
     template<typename Self>
     auto operator++(this Self&& self, int)
     {
         auto tmp = self; // Self 会被推导成 some_type
         ++self;
         return tmp;
     }
};

struct some_type : add_postfix_increment
{
     some_type& operator++() { ... }
};
```

在有显式对象形参的函数体内==不能==使用 this 指针：所有成员访问必须通过第一个形参进行，就像在静态成员函数中一样：

```C++
struct C
{
     void bar();

     void foo(this C c)
     {
         auto x = this; // 错误：不能使用 this
         bar();          // 错误：没有隐式的 this->
         c.bar();        // OK
     }
};
```

指向有显式对象形参的成员函数的指针是通常的函数指针，而不是到成员的指针：

```C++
struct Y
{
     int f(int, int) const&;
     int g(this Y const&, int, int);
};

auto pf = &Y::f;
pf(y, 1, 2);               // 错误：不能调用指向成员函数的指针
(y.*pf)(1, 2);             // OK
std::invoke(pf, y, 1, 2); // OK

auto pg = &Y::g;
pg(y, 3, 4);               // OK
(y.*pg)(3, 4);             // 错误：pg 不是指向成员函数的指针
std::invoke(pg, y, 3, 4); // OK
```

有显式对象形参的成员函数不能是静态成员函数或虚函数，也不能带有 cv 或引用限定符。

## 特殊成员函数

一些成员函数是特殊的：在某些环境下，即使用户不定义编译器也会定义它们。它们是：
-    默认构造函数
-    复制构造函数
-    移动构造函数 (C++11 起)
-    复制赋值运算符
-    移动赋值运算符 (C++11 起)
-    析构函数 (C++20 前) 预期的析构函数 (C++20 起)

特殊成员函数以及 比较运算符 (C++20 起) 是 仅有的 能 被 预置的 函数，即使用 = default 替代函数体进行定义（细节见其相应页面）。

```C++
#include <iostream>
#include <string>
#include <utility>
#include <exception>

struct S
{
     int data;

     // 简单的转换构造函数（声明）
     S(int val);

     // 简单的显式构造函数（声明）
     explicit S(std::string str);

     // const 成员函数（定义）
     virtual int getData() const { return data; }

};

// 构造函数的定义
S::S(int  val) : data(val)
{
     std::cout << "调用构造函数1，data = " << data << '\n';
}

// 此构造函数拥有 catch 子句    。。。只看到try，没有catch 啊。在下面。。
S::S(std::string  str)  try : data(std::stoi(str))
{
     std::cout << "调用构造函数2，data = " << data << '\n';
}
catch(const std::exception&)
{
     std::cout << "构造函数2失败，字符串'" << str << "'\n";
     throw; // 构造函数的 catch 子句应该始终重新抛出异常
}
。。。这个写法 真的猛啊。

struct D : S
{
     int data2;
     // 带默认实参的构造函数
     D(int v1, int v2 = 11)  :  S(v1), data2(v2)  {}

     // 虚成员函数
     int getData() const  override  { return data * data2; }

     // 左值限定的赋值运算符
     D& operator=(D other) &
     {
         std::swap(other.data, data);
         std::swap(other.data2, data2);
         return *this;
     }
};

int main()
{
     D d1 = 1;
     S s2("2");

     try
    {
         S s3("不是数字");
    }
    catch(const std::exception&) {}

     std::cout << s2.getData() << '\n';

     D d2(3, 4);
     d2 = d1;    // OK：赋值给左值
//   D(5) = d1; // 错误：没有适合的 operator= 重载
}
```

==================

https://zh.cppreference.com/w/cpp/language/default_constructor

# 默认构造函数

默认构造函数是不需要实参就能调用的构造函数（以空的或为每个形参提供默认实参的参数列表定义）。拥有公开默认构造函数的类型是可默认构造 (DefaultConstructible) 的。

语法
类名 ();                     (1)
类名 :: 类名 () 函数体         (2)
类名 () = delete;         (3)         (C++11 起)
类名 () = default;         (4)         (C++11 起)
类名 :: 类名 () = default;         (5)         (C++11 起)

其中 类名 必须指名当前类（或类模板的当前实例化），或在命名空间作用域或友元声明中声明时，必须是有限定的类名。

解释

1.  类定义中的默认构造函数的声明。
2.  类定义之外的默认构造函数的定义（该类必须包含一条声明 (1)）。有关构造函数的 函数体 的细节，参见构造函数与成员初始化器列表。
3.  弃置的默认构造函数：如果它被重载决议选择，那么程序编译失败。
4.  预置的默认构造函数：即便其他构造函数存在，编译器也会定义隐式默认构造函数。
5.  类定义之外的预置的默认构造函数（该类必须包含一条声明 (1)）。这种构造函数被当做是用户提供的（user-provided）（见下文以及值初始化）。

默认构造函数会在默认初始化和值初始化中被调用。

## 隐式声明的默认构造函数

如果没有对类类型（struct、class 或 union）提供任何用户声明的构造函数，那么编译器将始终声明一个作为它的类的 inline public 成员的默认构造函数。

当存在用户声明的构造函数时，用户仍可以通过关键词 ==default== 强制编译器自动生成原本隐式声明的默认构造函数。 (C++11 起)

隐式声明（或在它的首个声明被预置）的默认构造函数具有动态异常说明 (C++17 前)异常说明 (C++17 起)中所描述的异常说明。

## 隐式定义的默认构造函数

如果隐式声明的默认构造函数未被定义为弃置的，那么当它被 ODR 式使用或者被常量求值所需要 (C++11 起)时，编译器会定义它（即生成函数体并编译），且它与拥有空函数体和空初始化器列表的用户定义的构造函数有严格相同的效果。

即它调用这个类的各基类和各非静态成员的默认构造函数。值初始化过程中由用户提供的空构造函数和隐式定义或预置的构造函数可能会以不同的方式处理。

。。。ODR （单一定义规则）。。。炸裂
。。anyway， 一定会声明 一个 构造器( 用户定义 或 默认 )， 但不一定有 定义。

C++11起

如果它满足对于 constexpr 构造函数 (C++23 前)constexpr 函数 (C++23 起)的要求，那么生成的构造函数是 constexpr 的。

当存在用户定义的构造函数时，用户仍可以通过关键词 default 强制编译器自动生成原本隐式声明的默认构造函数。

## 弃置的隐式声明的默认构造函数

如果满足下列任一条件，那么类 T 中隐式声明的 或 预置的 (C++11 起)默认构造函数不被定义 (C++11 前)  被定义为弃置的 (C++11 起)：

T 拥有无默认初始化器的 (C++11 起)引用类型的成员。
    T 拥有无默认成员初始化器的 (C++11 起)非 const 可默认构造的 const 成员。
    T 拥有默认构造函数被弃置，或对于此构造函数有歧义或不可访问的成员，且该成员无默认成员初始化器 (C++11 起)。
    T 拥有默认构造函数被弃置，或对于此构造函数有歧义或不可访问的直接或虚基类。
    T 拥有析构函数被弃置，或对于此构造函数不可访问的直接基类，虚基类或非静态数据成员。
    T 是所有变体成员都是 const 的联合体。
C++11起：
    T 是至少有一个变体成员有非平凡默认构造函数，而没有任何变体成员拥有默认成员初始化器的联合体。
    T 是拥有变体成员 M 的非联合类，该成员拥有非平凡默认构造函数，而包含 M 的匿名联合体没有任何变体成员拥有默认成员初始化器。

C++11起： 当不存在用户定义的构造函数，且隐式声明的默认构造函数非平凡时，用户仍可以通过关键词 delete 禁止编译器自动生成隐式定义的默认构造函数。

## 平凡默认构造函数

如果满足下列所有条件，那么类 T 的默认构造函数是平凡的（即没有任何动作）：
    构造函数并非用户提供的（即它被隐式定义或在它的首个声明中预置的）
    T 没有虚成员函数
    T 没有虚基类
    T 没有拥有默认初始化器的非静态数据成员。 (C++11 起)
    每个 T 的直接基类都拥有平凡默认构造函数
    每个类类型（或它的数组类型）的非静态成员都拥有平凡默认构造函数

平凡默认构造函数是没有任何动作的构造函数。所有与 C 语言兼容的数据类型（POD 类型）都是可以平凡默认构造的。

## 合格的默认构造函数

被用户声明或者同时被隐式声明且可定义的默认构造函数是合格的。 (C++11 前)
没有被弃置的默认构造函数是合格的。(C++11 起)(C++20 前)
满足下列所有条件的默认构造函数是合格的：
    它没有被弃置，且
    满足它的所有关联约束（如果存在），且
    没有比它更受约束的默认构造函数。
(C++20 起)

合格的默认构造函数的平凡性确定该类是否为隐式生存期类型，以及该类是否为平凡类型。

```C++
struct A
{
     int x;
     A(int x = 1): x(x) {} // 用户定义默认构造函数
};

struct B: A
{
     // 隐式定义 B::B()，调用 A::A()
};

struct C
{
     A a;
     // 隐式定义 C::C()，调用 A::A()
};

struct D: A
{
     D(int y): A(y) {}
     // 不会声明 D::D()，因为已经有其他构造函数
};

struct E: A
{
     E(int y): A(y) {}
     E() = default; // 显式预置，调用 A::A()
};

struct F
{
     int& ref; // 引用成员
     const int c; // const 成员
     // F::F() 被隐式定义为弃置的
};

// 用户声明的复制构造函数（由用户提供，被弃置或被预置）
// 防止隐式生成默认构造函数

struct G
{
     G(const G&) {}
     // G::G() 被隐式定义为弃置的
};

struct H
{
     H(const H&) = delete;
     // H::H() 被隐式定义为弃置的
};

struct I
{
     I(const I&) = default;
     // I::I() 被隐式定义为弃置的
};

int main()
{
     A a;
     B b;
     C c;
//   D d; // 编译错误
     E e;
//   F f; // 编译错误
//   G g; // 编译错误
//   H h; // 编译错误
//   I i; // 编译错误
}
```

参阅
    构造函数
    初始化
        聚合初始化
        常量初始化
        复制初始化
        默认初始化
        直接初始化
        列表初始化
        引用初始化
        值初始化
        零初始化
    new

。。。。。这个页面的最下面。

==================
https://zh.cppreference.com/w/cpp/language/constructor

# 构造函数与成员初始化器列表

构造函数是类的一种特殊的非静态成员函数，用于初始化该类类型的对象。

在类的构造函数定义中，成员初始化器列表指定各个直接基类、虚基类和非静态数据成员的初始化器。 （请勿与 std::initializer_list 混淆）

构造函数不能是协程。(C++20 起)

语法
构造函数用以下形式的成员函数声明符声明：

> 类名 ( 形参列表(可选) ) 异常说明(可选) 属性(可选)         (1)

其中 类名 必须指名当前类（或类模板的当前实例化），或当在命名空间作用域或在友元声明中声明时，它必须是有限定的类名。

构造函数声明的 声明说明符序列 中只允许说明符 friend、inline、constexpr (C++11 起)、consteval (C++20 起) 及 explicit（尤其是不允许返回类型）。注意 cv 及引用限定符也不受允许；const 与 volatile 语义对于构造过程中的对象没有效果，它们要到最终派生类的构造函数完成才会生效。

任何构造函数的函数定义的函数体可以在复合语句的开花括号之前包含成员初始化器列表，其语法是冒号字符 : 后随一个或多个 **成员初始化器** 的逗号分隔列表，每项均具有以下语法：

> 类或标识符 ( 表达式列表(可选) )         (1)
> 类或标识符 花括号初始化器列表         (2)         (C++11 起)
> 形参包 ...         (3)         (C++11 起)

1.  用直接初始化，或当 表达式列表 为空时用值初始化，初始化 类或标识符 所指名的基类或成员
    
2.  用列表初始化（列表为空时进行值初始化，而在初始化聚合体时进行聚合初始化），初始化 类或标识符 所指名的基类或成员
    
3.  用包展开初始化多个基类
    

|     |     |
| --- | --- |
| 类或标识符 | 任何指名非静态数据成员的标识符，或任何指名该类自身（对于委托构造函数）、直接基类或虚基类的类型名。 |
| 表达式列表 | 传递给基类或成员的参数的逗号分隔列表，可以为空 |
| 花括号初始化器列表 | 花括号包围的以逗号分隔的初始化器和嵌套的花括号初始化器列表的列表 |
| 形参包 | 变参模板形参包的名字 |

```C++
struct S
{
     int n;

     S(int);        // 构造函数声明

     S() : n(7) {} // 构造函数定义：
                   // ": n(7)" 是初始化器列表
                   // ": n(7) {}" 是函数体
};

S::S(int x) : n{x} {} // 构造函数定义：": n{x}" 是初始化器列表

int main()
{
     S s;       // 调用 S::S()
     S s2(10); // 调用 S::S(int)
}
```

解释

构造函数==没有名字==且无法被直接调用。它们在发生初始化时被调用，且它们按照初始化的规则进行选择。没有 explicit 说明符的构造函数是转换构造函数。有 constexpr 说明符的构造函数会让其类型成为字面类型 (LiteralType) 。可以不带任何实参调用的构造函数是默认构造函数。可以接收同类型的另一对象为实参的构造函数是复制构造函数和移动构造函数。

。。哈？我一直认为的是 构造器没有 返回值。  对啊， 它没有 return。
。。但是 没有名字 也好像 可以。
。。网上基本上都是说 没有返回值。

在开始执行组成构造函数体的复合语句之前，所有直接基类、虚基类和非静态数据成员的初始化均已结束。这些对象的非默认初始化只能在成员初始化器列表指定。对于不能默认初始化的基类或非静态数据成员，例如引用和 const 限定的类型的成员，必须指定成员初始化器。对没有成员初始化器的匿名联合体或变体成员不进行初始化。

类或标识符 指名虚基类的初始化器，在并非所构造对象的最终派生类的类的构造期间被忽略。

在 表达式列表 或 花括号初始化器列表 中出现的名字在构造函数的作用域中求值：

```C++
class X
{
     int a, b, i, j;
public:
     const int& r;
     X(int i)
       :  r(a)  // 初始化  X::r 以指代 X::a
       , b{i}  // 初始化 X::b 为形参 i 的值
       , i(i) // 初始化 X::i 为形参 i 的值
       , j(this->i) // 初始化 X::j 为 X::i 的值
     {}
};
```

成员初始化器所抛出的异常可以被函数 try 块处理。

### try + constructor

。。

```C++
struct S
{
     std::string m;

     S(const std::string& str, int idx) try : m(str, idx)
     {
         std::cout << "S(" << str << ", " << idx << ") 构造完成，m = " << m << '\n';
     }
     catch (const  std::exception& e)
     {

         std::cout << "S(" << str << ", " << idx << ") 失败：" << e.what() << '\n';

     } // 此处有隐含的 "throw;"
};
```

。。

成员函数（包括虚成员函数）可以从成员初始化器调用，但如果在该点还有直接基类尚未被初始化，那么行为未定义。

对于虚调用（如果在该点已初始化直接基类），适用与从构造函数与析构函数中进行虚函数调用相同的规则：虚成员函数表现如同 *this 的动态类型是正在构造的类的静态类型（动态派发不在继承层级下传），而对纯虚成员函数的虚调用（但非静态调用）是未定义行为。

C++11 如果非静态数据成员具有 默认成员初始化器且也在成员初始化器列表中出现，那么使用该成员初始化器而忽略默认成员初始化器：

```C++
struct S
{
     int n = 42;    // 默认成员初始化器
     S() : n(7) {} // 将设置 n 为 7，而非 42
};
```

引用成员不能绑定到成员初始化器列表中的临时量：

```C++
struct A
{
     A() : v(42) {} // 错误
     const int&  v;
};
```

注：这同样适用于默认成员初始化器。

## C++11 委托构造函数

如果类自身的名字在初始化器列表中作为 类或标识符 出现，那么该列表只能由这一个成员初始化器组成；这种构造函数被称为委托构造函数（delegating constructor），而构造函数列表的仅有成员所选择的构造函数是目标构造函数。

此时首先由重载决议选择目标构造函数并予以执行，然后控制返回到委托构造函数并执行其函数体。
委托构造函数不能递归。

```C++
class Foo
{
public:
     Foo(char x, int y) {}
     Foo(int y) : Foo('a', y)  {} // Foo(int) 委托到 Foo(char, int)
};
```

继承的构造函数
见 using 声明。

## 初始化顺序

列表中的成员初始化器的顺序是不相关的：初始化的实际顺序如下：

1.  如果构造函数是最终派生类的，那么按基类声明的深度优先、从左到右的遍历中的出现顺序（从左到右指的是基说明符列表中所呈现的），初始化各个虚基类
2.  然后，以在此类的基类说明符列表中出现的从左到右顺序，初始化各个直接基类
3.  然后，以类定义中的声明顺序，初始化各个非静态成员。
4.  最后，执行构造函数体
    （注意：如果初始化的顺序是由不同构造函数中的成员初始化器列表中的出现所控制，那么析构函数就无法确保销毁顺序是构造顺序的逆序了）

```C++
#include <fstream>
#include <string>
#include <mutex>

struct Base
{
     int n;
};

struct Class : public Base
{
     unsigned char x;
     unsigned char y;
     std::mutex m;
     std::lock_guard<std::mutex> lg;
     std::fstream f;
     std::string s;

     Class(int x) : Base{123}, // 初始化基类
         x (x),   // x（成员）以 x（形参）初始化
         y {0},   // y 初始化为 0
         f{"test.cc", std::ios::app}, // 在 m 和 lg 初始化之后发生
         s(__func__), //__func__ 可用，因为初始化器列表是构造函数的一部分
         lg (m), // lg 使用已经初始化的 m
         m{}      // m 在 lg 前初始化，即使它最后出现在此处
     {}           // 空复合语句

     Class(double a) : y(a + 1),
         x(y), // x 将在 y 前初始化，它的值不确定
         lg(m)
     {} // 基类初始化器未在列表中出现，它被默认初始化（这与使用 Base() 不同，那是值初始化）

     Class()
     try // 函数 try 块在包含初始化器列表的函数体之前开始
       : Class(0.0) // 委托构造函数
     {
         // ...
     }
     catch (...)
     {
         // 初始化中发生的异常
     }
};

int main()
{
     Class c;
     Class c1(1);
     Class c2(0.1);
}
```

参阅

复制消除
    转换构造函数
    复制赋值
    复制构造函数
    默认构造函数
    析构函数
    explicit
    初始化
        聚合初始化
        常量初始化
        复制初始化
        默认初始化
        直接初始化
        列表初始化
        引用初始化
        值初始化
        零初始化
    移动赋值
    移动构造函数
    new

* * *

https://zh.cppreference.com/w/cpp/language/copy_constructor

# 复制构造函数

类 T 的复制构造函数是首个形参是 T&、const T&、volatile T& 或 const volatile T&，而且要么没有其他形参，要么剩余形参均有默认值的非模板构造函数。

语法
类名 ( const 类名 & )         (1)
类名 ( const 类名 & ) = default;         (2)         (C++11 起)
类名 ( const 类名 & ) = delete;         (3)         (C++11 起)

其中 类名 必须指名当前类（或类模板的当前实例化），或在命名空间作用域或友元声明中声明时，必须是有限定的类名。

解释

1.  复制构造函数的典型声明。
2.  强制编译器生成复制构造函数。
3.  阻止隐式生成复制构造函数。

复制构造函数会在对象从同类型的另一对象（以直接初始化或复制初始化）初始化时调用（除非重载决议选择了更好的匹配或其调用被消除），情况包括：
    初始化：T a = b; 或 T a(b);，其中 b 的类型是 T；
    函数实参传递：f(a);，其中 a 的类型是 T 而 f 是 void f(T t)；
    函数返回：在像 T f() 这样的函数内部的 return a;，其中 a 的类型是 T 且它没有移动构造函数。

## 隐式声明的复制构造函数

如果没有向类类型（struct、class 或 union）提供任何用户定义的复制构造函数，那么编译器总是会声明一个复制构造函数作为这个类的非 explicit 的 inline public 成员。如果满足下列所有条件，那么这个隐式声明的复制构造函数拥有形式 T::T(const T&)：

T 的每个直接基类和虚基类 B 均拥有形参为 const B& 或 const volatile B& 的复制构造函数；
T 的每个类类型或类类型数组的非静态数据成员 M 均拥有形参为 const M& 或 const volatile M& 的复制构造函数。

否则，隐式声明的复制构造函数是 T::T(T&)。（注意因为这些规则，隐式声明的复制构造函数不能绑定到 volatile 左值实参）。

类可以拥有多个复制构造函数，如 T::T(const T&) 和 T::T(T&)。

(C++11 起) 当存在用户定义的复制构造函数时，用户仍可以使用关键词 default 强制编译器生成隐式声明的复制构造函数。

隐式声明（或在它的首个声明被预置）的复制构造函数具有动态异常说明 (C++17 前) noexcept 说明 (C++17 起)中所描述的异常说明。

## 弃置的隐式声明的复制构造函数

(C++11 前)如果满足下列任一条件，那么不定义类 T 的隐式声明的复制构造函数：
(C++11 起)如果满足下列任一条件，那么类 T 的隐式声明的复制构造函数被定义为弃置的：
T 拥有无法复制的非静态数据成员（拥有被弃置、不可访问或有歧义的复制构造函数）；
T 拥有无法复制的直接或虚基类（拥有被弃置、不可访问或有歧义的复制构造函数）；
T 拥有带被弃置或不可访问的析构函数的直接基类，虚基类或非静态数据成员；
C++11起：
T 是联合体式的类，且拥有带非平凡复制构造函数的变体成员；
T 拥有右值引用类型的数据成员；
T 拥有用户定义的移动构造函数或移动赋值运算符（此条件只会导致隐式声明的，而非预置的复制构造函数被弃置）。

## 平凡的复制构造函数

如果满足下列所有条件，那么类 T 的复制构造函数是平凡的：
    它不是用户提供的（即它是隐式定义或预置的）；
    T 没有虚成员函数；
    T 没有虚基类；
    为 T 的每个直接基类选择的复制构造函数都是平凡的；
    为 T 的每个类类型（或类类型数组）的非静态成员选择的复制构造函数都是平凡的；

非联合类的平凡复制构造函数的效果是复制实参的每个标量子对象（递归地包含子对象的子对象，以此类推），且不进行其他动作。不过不需要复制填充字节，甚至只要它们的值相同，每个复制的子对象的对象表示也不必相同。

可平凡复制 (TriviallyCopyable) 对象可以通过手动复制其对象表示来进行复制，例如用 std::memmove。所有与 C 语言兼容的数据类型（POD 类型）都可以平凡复制。

## 合格的复制构造函数

(C++11 前)
被用户声明或者同时被隐式声明且可定义的复制构造函数是合格的。

(C++11 起)
(C++20 前)
没有被弃置的复制构造函数是合格的。

(C++20 起)
满足下列所有条件的复制构造函数是合格的：
    它没有被弃置，且
    满足它的所有关联约束（如果存在），且
    没有比它更受约束且拥有相同的第一形参类型的复制构造函数。

合格的复制构造函数的平凡性确定该类是否为隐式生存期类型，以及该类是否为可平凡复制类型。

## 隐式定义的复制构造函数

如果隐式声明的复制构造函数没有被弃置，那么当它被 ODR 式使用或用于常量求值 (C++11 起)时，它会被编译器定义（即生成并编译函数体）。对于联合体类型，隐式定义的复制构造函数（如同以 std::memmove）复制它的对象表示。对于非联合类类型（class 与 struct），该构造函数用直接初始化，按照初始化顺序，对对象的各基类和非静态成员进行完整的逐成员复制。

(C++11 起)

如果它满足对于 constexpr 构造函数 (C++23 前)constexpr 函数 (C++23 前)的要求，那么生成的复制构造函数也是 constexpr 的。

当 T 拥有用户定义的析构函数或用户定义的复制赋值运算符时，隐式定义的复制构造函数的生成会被弃

注解
许多情况下，即使复制构造函数能够产生可观测副作用，它们也会被优化掉，见复制消除。
。https://zh.cppreference.com/w/cpp/language/copy_elision
示例

```C++
struct A
{
     int n;
     A(int n = 1) : n(n) {}
     A(const A& a) : n(a.n) {} // 用户定义的复制构造函数
};

struct B : A
{
     // 隐式默认构造函数 B::B()
     // 隐式复制构造函数 B::B(const B&)
};

struct C : B
{
     C() : B() {}
private:
     C(const C&); // 不可复制，C++98 风格
};

int main()
{
     A a1(7);
     A a2(a1); // 调用复制构造函数

     B b;
     B b2 = b;
     A a3 = b; // 转换到 A& 并调用复制构造函数

     volatile A va(10);
     // A a4 = va; // 编译错误

     C c;
     // C c2 = c; // 编译错误
}
```

* * *

https://zh.cppreference.com/w/cpp/language/move_constructor

# 移动构造函数

类 T 的移动构造函数是非模板构造函数，它的首个形参是 T&&、const T&&、volatile T&& 或 const volatile T&&，且没有其他形参，或剩余形参都有默认值。

语法
类名 ( 类名 && )         (1)         (C++11 起)
类名 ( 类名 && ) = default;         (2)         (C++11 起)
类名 ( 类名 && ) = delete;         (3)         (C++11 起)

其中 类名 必须指名当前类（或类模板的当前实例化），或在命名空间作用域或友元声明中声明时，必须是有限定的类名。

解释

1.  移动构造函数的典型声明。
2.  强制编译器生成移动构造函数。
3.  避免隐式生成移动构造函数。

当从同类型的右值（亡值或纯右值） (C++17 前)亡值 (C++17 起)初始化（直接初始化或复制初始化）对象时，会调用移动构造函数，情况包括：
    初始化：T a = std::move(b); 或 T a(std::move(b));，其中 b 的类型是 T ；
    函数实参传递：f(std::move(a));，其中 a 的类型是 T 且 f 是 Ret f(T t) ；
    函数返回：在像 T f() 这样的函数中的 return a;，其中 a 的类型是 T，且 T 有移动构造函数。

当初始化器是纯右值时，通常会优化掉 (C++17 前)始终不会进行 (C++17 起)对移动构造函数的调用，见复制消除。

典型的移动构造函数“窃取”实参曾保有的资源（例如指向动态分配对象的指针，文件描述符，TCP socket，输入输出流，运行的线程，等等），而非复制它们，并使它的实参遗留在某个合法但不确定的状态。例如，从 std::string 或从 std::vector 移动可以导致实参被置为空。但是不应该依赖此类行为。对于某些类型，例如 std::unique_ptr，移动后的状态是完全指定的。

## 隐式声明的移动构造函数

如果不对类类型（struct、class 或 union）提供任何用户定义的移动构造函数，且满足下列所有条件：
    没有用户声明的复制构造函数；
    没有用户声明的复制赋值运算符；
    没有用户声明的移动赋值运算符；
    没有用户声明的析构函数；

那么编译器将声明一个移动构造函数作为这个类的非 explicit 的 inline public 成员，签名为 T::T(T&&)。

一个类可以拥有多个移动构造函数，例如 T::T(const T&&) 和 T::T(T&&)。当存在用户定义的移动构造函数时，用户仍然可以通过关键词 default 强制编译器生成隐式声明的移动构造函数。

隐式声明（或在它的首个声明被预置）的移动构造函数具有动态异常说明 (C++17 前)noexcept 说明 (C++17 起)中所描述的异常说明。

## 弃置的隐式声明的移动构造函数

如果满足下列任一条件，那么类 T 的隐式声明或预置的移动构造函数被定义为弃置的：
    T 拥有无法移动（拥有被弃置、不可访问或有歧义的移动构造函数）的非静态数据成员；
    T 拥有无法移动（拥有被弃置、不可访问或有歧义的移动构造函数）的直接或虚基类；
    T 拥有带被弃置或不可访问的析构函数的直接或虚基类；
    T 是联合式的类，且拥有带非平凡移动构造函数的变体成员。

> 重载决议忽略被弃置的预置移动构造函数（否则它会阻止从右值复制初始化）。

## 平凡的移动构造函数

如果满足下列所有条件，类 T 移动构造函数是平凡的：
    它不是用户提供的（即它是隐式定义或预置的）；
    T 没有虚成员函数；
    T 没有虚基类；
    为 T 的每个直接基类选择的移动构造函数都是平凡的；
    为 T 的每个类类型（或类类型数组）的非静态成员选择的移动构造函数都是平凡的。

平凡的移动构造函数是与平凡的复制构造函数实施相同动作的构造函数，即它如同用 std::memmove 来进行对象表示的复制。所有与 C 兼容的数据类型（POD 类型）都可以平凡移动。

## 合格的移动构造函数

(C++20 前)
没有被弃置的移动构造函数是合格的。

(C++20 起)
满足下列所有条件的移动构造函数是合格的：
    它没有被弃置，且
    满足它的所有关联约束（如果存在），且
    没有比它更受约束且拥有相同的第一形参类型的移动构造函数。

合格的移动构造函数的平凡性确定该类是否为隐式生存期类型，以及该类是否为可平凡复制类型。

## 隐式定义的移动构造函数

如果隐式声明的移动构造函数没有被弃置也不平凡，那么当它被 ODR 式使用或用于常量求值时，它会被编译器定义（生成并编译函数体）。对于联合体类型，隐式定义的移动构造函数（如同以 std::memmove）复制它的对象表示。对于非联合类类型（class 与 struct），该构造函数用以亡值实参执行的直接初始化，按照初始化顺序，对对象的各基类和非静态成员进行完整的逐对象移动。如果它满足对于 constexpr 构造函数的要求，那么生成的移动构造函数也是 constexpr 的。

(C++11 起)

如果它满足对于 constexpr 构造函数 (C++23 前)constexpr 函数 (C++23 起)的要求，那么生成的移动构造函数也是 constexpr 的。

注解

为使强异常保证可行，用户定义的移动构造函数不应该抛出异常。例如，std::vector 在需要重新放置元素时，基于 std::move\_if\_noexcept 在移动和复制之间选择。

如果同时提供了复制和移动构造函数而没有其他可行的构造函数，那么重载决议在实参是相同类型的右值（如 std::move 的结果的亡值，或如无名临时量的纯右值）时选择移动构造函数，而在实参是左值（具名对象或返回左值引用的函数/运算符）时选择复制构造函数。如果只提供复制构造函数，那么重载决议对于所有实参类别都会选择它（只要它接收到 const 的引用，因为右值能绑定到 const 引用），这使得复制可以作为在移动不可用时的后备。

当接收右值引用为它的形参时，构造函数被称作“移动构造函数”。它没有义务移动任何内容，不要求类拥有要被移动的资源，而且在受允许（但可能没意义）的以 const 右值引用（const T&&）为形参的情况中，‘移动构造函数’可能无法移动资源。

```C++
#include <string>
#include <iostream>
#include <iomanip>
#include <utility>

struct A
{
     std::string s;
     int k;

     A() : s("测试"), k(-1) {}
     A(const A& o) : s(o.s), k(o.k) { std::cout << "移动失败！\n"; }
     A(A&& o) noexcept :
         s(std::move(o.s)),        // 类类型成员的显式移动
         k(std::exchange(o.k, 0)) // 非类类型成员的显式移动
     {}
};

A f(A a)
{
     return a;
}

struct B : A
{
     std::string s2;
     int n;
     // 隐式移动构造函数 B::(B&&)
     // 调用 A 的移动构造函数
     // 调用 s2 的移动构造函数
     // 并进行 n 的逐位复制
};

struct C : B
{
     ~C() {} // 析构函数阻止隐式移动构造函数 C::(C&&)
};

struct D : B
{
     D() {}
     ~D() {}            // 析构函数阻止隐式移动构造函数 D::(D&&)
     D(D&&) = default; // 强制生成移动构造函数
};

int main()
{
     std::cout << "尝试移动 A\n";
     A a1 = f(A()); // 按值返回时，从函数形参移动构造它的目标

     std::cout << "移动前，a1.s = " << std::quoted(a1.s)
         << " a1.k = " << a1.k << '\n';

     A a2 = std::move(a1); // 从亡值移动构造
     std::cout << "移动后，a1.s = " << std::quoted(a1.s)
         << " a1.k = " << a1.k << '\n';

     std::cout << "\n尝试移动 B\n";
     B b1;

     std::cout << "移动前，b1.s = " << std::quoted(b1.s) << "\n";

     B b2 = std::move(b1); // 调用隐式移动构造函数
     std::cout << "移动后，b1.s = " << std::quoted(b1.s) << "\n";

     std::cout << "\n尝试移动 C\n";
     C c1;
     C c2 = std::move(c1); // 调用复制构造函数

     std::cout << "\n尝试移动 D\n";
     D d1;
     D d2 = std::move(d1);
}
```

输出：

尝试移动 A
移动前，a1.s = "测试" a1.k = -1
移动后，a1.s = "" a1.k = 0

尝试移动 B
移动前，b1.s = "测试"
移动后，b1.s = ""

尝试移动 C
移动失败！

尝试移动 D

* * *

https://zh.cppreference.com/w/cpp/language/destructor

# 析构函数

析构函数是对象生存期终结时调用的特殊成员函数。析构函数的目的是释放对象可能在它的生存期间获得的资源。

(C++20 起)
析构函数不能是协程。

语法

~ 类名 ();                     (1)

virtual ~ 类名 ();         (2)

声明说明符序列(可选) ~ 类名 () = default;         (3)         (C++11 起)

声明说明符序列(可选) ~ 类名 () = delete;               (4)         (C++11 起)

属性(可选) 声明说明符序列(可选) 标识表达式 ( void(可选) )
异常说明(可选) requires子句(可选) 属性(可选) ;                  (5)

1.  预期的 (C++20 起)析构函数的典型声明
2.  基类通常需要虚析构函数
3.  强迫编译器生成析构函数
4.  禁用隐式析构函数
5.  预期的 (C++20 起)析构函数声明的正式语法

|     |     |
| --- | --- |
| 声明说明符序列 | friend、inline、virtual、constexpr、consteval (C++20 起) |
| 标识表达式 | 在类定义中时，是符号 ~ 后随 类名。在类模板中时，是符号 ~ 后随模板当前实例化的名字。在命名空间作用域，或在另一类的友元声明中时，是 嵌套名说明符 后随符号 ~ 再后随与 嵌套名说明符 所指名者相同的 类名。该名字在任何情况下都必须是类或模板的实际名字，而非 typedef。可以用不改变它的含义的括号环绕整个 标识表达式。 |
| 属性  | (C++11 起) 任意数量属性序列 |
| 异常说明 | 与在任何函数声明中一样的异常说明<br>(C++11 起)<br>    在有显式提供异常说明时，该异常说明被认为是隐式声明的析构函数（见下文）可能所用的异常说明之一。它大多数情况下是 noexcept(true)。因此抛出异常的析构函数必须显式声明为 noexcept(false)。 |
| requires子句 | (C++20 起) 声明预期析构函数的关联约束的 requires 子句，为使得预期析构函数被选为析构函数，它必须得到满足 |

解释
每当对象的生存期结束时都会调用析构函数，时间点包含：

程序终止，对于具有静态存储期的对象

(C++11 起)
    退出线程，对于具有线程局部存储期的对象

作用域结束，对于具有自动存储期的对象和生存期因绑定到引用而延长的临时量

delete 表达式，对于具有动态存储期的对象

完整表达式的结尾，对于无名临时量

栈回溯，对于具有自动存储期的对象，当未捕捉的异常脱离它所在的块时。

析构函数也可以直接调用，例如销毁用布置 new 或通过分配器成员函数（如 std::allocator::destroy()）构造的对象。注意，对普通对象（如局部变量）直接调用析构函数会导致在作用域结束处再次调用析构函数时引发未定义行为。

在泛型语境中，析构函数调用语法可以用于==非类类型==的对象；这被称为==伪==析构函数调用：见成员访问运算符。

(C++20 起)   预期的析构函数
一个类可以有一个或多个预期的析构函数，其中之一会被选为类的析构函数。

为确定哪个预期的析构函数是析构函数，重载决议会在类定义的末尾以空实参列表，在类中声明的潜在析构函数之间进行。如果重载决议失败，那么程序非良构。析构函数选择不会 odr 式使用选中的析构函数，而且选择的析构函数可以是弃置的。

所有预期的析构函数都是特殊成员函数。如果不对类 T 提供用户声明的预期的析构函数，那么编译器总是会声明一个（见后述），而该隐式声明的预期析构函数也是 T 的析构函数。

## 隐式声明的析构函数

如果不向类类型（struct、class 或 union）提供用户声明的析构函数，那么编译器总是会声明一个析构函数作为这个类的 inline public 成员。

与任何隐式声明的特殊成员函数相同，隐式声明的析构函数的异常说明是不抛出的，除非任何潜在构造的基类或成员的析构函数是潜在抛出的 (C++17 起)隐式定义会直接调用有不同异常说明的函数 (C++17 前)。实践上，隐式的析构函数是 noexcept 的，除非该类被析构函数被 noexcept(false) 的基类或成员所“毒害”。

## 弃置的隐式声明的析构函数

如果满足下列任一条件，那么类 T 的隐式声明或显式预置的析构函数不会被定义 (C++11 前)被定义为弃置的 (C++11 起)：
    T 拥有无法销毁（拥有被弃置或不可访问的析构函数）的非静态数据成员
    T 拥有无法销毁（拥有被弃置或不可访问的析构函数）的直接或虚基类

(C++11 起)
    T 是联合体并拥有带非平凡析构函数的变体成员。

隐式声明的析构函数是虚函数（因为基类有虚析构函数），且对解分配函数（operator delete()）的查找导致对有歧义、被弃置或不可访问函数的调用。

(C++20 起)
如果显式预置的 T 的预期析构函数不是 T 的析构函数，那么它被定义为弃置的。

## 平凡析构函数

如果满足下列全部条件，那么 T 的析构函数是平凡的：
    析构函数不是用户提供的（表示它是隐式声明，或在它的首个声明显式定义为预置的）
    析构函数非虚（即基类析构函数非虚）
    所有直接基类都拥有平凡析构函数
    所有类类型（或类的数组类型）的非静态数据成员都拥有平凡析构函数

平凡析构函数是不进行任何动作的析构函数。有平凡析构函数的对象不要求 delete 表达式，并可以通过简单地解分配它的存储进行释放。所有与 C 语言兼容的数据类型（POD 类型）都是可以平凡析构的。

## 隐式定义的析构函数

如果隐式声明的析构函数没有被弃置，那么当它被 ODR 式使用时，它会被编译器隐式定义（即生成并编译函数体）。这个隐式定义的析构函数拥有空的函数体。

(C++20 起)
如果生成的析构函数满足 constexpr 析构函数 (C++23 前)constexpr 函数 (C++23 起)的要求，那么它是 constexpr 的。

## 析构序列

对于用户定义或隐式定义的析构函数，在析构函数体执行后，编译器会以声明的逆序调用该类的所有非静态非变体数据成员的析构函数，然后以构造的逆序调用所有直接非虚基类的析构函数（继而调用它的成员与它的基类的析构函数，以此类推），最后，如果此对象类型是最终派生类，那么调用所有虚基类的析构函数。

即便在直接调用析构函数时（例如 obj.~Foo();），~Foo() 中的 return 语句也不会立即将控制返回到调用方：它首先调用它的所有成员及基类的析构函数。

## 虚析构函数

通过指向==基类的==指针删除对象会引发==未定义==行为，除非基类的析构函数是==虚==函数：

```C++
class Base
{
public:
     virtual  ~Base() {}
};

class Derived : public Base {};

Base* b = new Derived;
delete b; // 安全
```

> 一条常用方针是，基类的析构函数必须是公开且虚或受保护且非虚之一

## 纯虚析构函数

预期的 (C++20 起)析构函数可以声明为纯虚的，例如对于需要声明为抽象类，但没有其他可声明为纯虚的适合函数的基类。纯虚析构函数必须有定义，因为在销毁派生类时，所有基类析构函数都会被调用：

```C++
class AbstractBase
{
public:
     virtual ~AbstractBase() = 0;
};
AbstractBase::~AbstractBase() {}

class Derived : public AbstractBase {};

// AbstractBase obj; // 编译错误
Derived obj;          // OK
```

## 异常

与其他函数一样，析构函数可以通过抛出异常终止（这通常要求将它明确声明为 noexcept(false)） (C++11 起)，然而如果恰好在栈回溯期间调用析构函数，那么会改为调用 std::terminate。

虽然有时候可以用 std::uncaught_exception 来检测进展中的栈回溯，但允许任何析构函数通过抛异常终止通常被认为是坏的实践。尽管如此，一些库（如 SOCI 与 Galera 3 等）仍然使用了这项功能，它们利用了无名临时量的析构函数的能力，以在构造该临时量的完整表达式结束处抛异常。

库基础 TS v3 中的 std::experimental::scope_success 可以拥有潜在抛出的析构函数，它在正常退出作用域且退出函数抛异常时抛出异常。

。。在表达式 的结束处 抛出异常。 这个是用来干什么的。

```C++
#include <iostream>
struct A
{
     int i;
     A(int num) : i(num)
     {
         std::cout << "构造 a" << i << '\n';
     }

     ~A()
     {
         std::cout << "析构 a" << i << '\n';
     }
};
A a0(0);
int main()
{
     A a1(1);
     A* p;
     { // 嵌套的作用域
         A a2(2);
         p = new A(3);
     } // a2 离开作用域
     delete p; // 调用 a3 的析构函数
}
```

输出：
构造 a0
构造 a1
构造 a2
构造 a3
析构 a2
析构 a3
析构 a1
析构 a0

* * *

https://zh.cppreference.com/w/cpp/language/copy_assignment

# 复制赋值运算符

类 T 的复制赋值运算符是名为 operator= 的非模板非静态成员函数，它接受恰好一个 T、T&、const T&、volatile T& 或 const volatile T& 类型的形参。可复制赋值 (CopyAssignable) 类型必须有公开的复制赋值运算符。

语法
类名 & 类名 ::operator= ( 类名 )               (1)
类名 & 类名 ::operator= ( const 类名 & )         (2)
类名 & 类名 ::operator= ( const 类名 & ) = default;         (3)         (C++11 起)
类名 & 类名 ::operator= ( const 类名 & ) = delete;         (4)         (C++11 起)

解释

1.  复制赋值运算符在采用复制交换法时的典型声明。
2.  复制赋值运算符在不采用复制交换法时的典型声明。
3.  强制编译器生成复制赋值运算符。
4.  避免隐式复制赋值。

每当重载决议选择复制赋值运算符时，它都会被调用，例如对象出现在赋值表达式左侧时。

## 隐式声明的复制赋值运算符

如果没有对类类型（struct、class 或 union）提供任何用户定义的复制赋值运算符，那么编译器将始终声明一个，作为类的 inline public 成员。如果满足下列所有条件，那么这个隐式声明的复制赋值运算符拥有形式 T& T::operator=(const T&)：

T 的每个直接基类 B 均拥有形参是 B，const B& 或 const volatile B& 的复制赋值运算符；
    T 的每个类类型或类数组类型的非静态数据成员 M 均拥有形参是 M，const M& 或 const volatile M& 的复制赋值运算符。

否则，隐式声明的复制赋值运算符会被声明为 T& T::operator=(T&)。（注意因为这些规则，隐式声明的复制赋值运算符不能绑定到 volatile 左值实参。）

类可以拥有多个复制赋值运算符，如 T& T::operator=(T&) 和 T& T::operator=(T)。当存在用户定义的复制赋值运算符时，用户仍然可以通过关键词 default 强迫编译器生成隐式声明的复制赋值运算符。 (C++11 起)

隐式声明（或在它的首个声明被预置）的复制赋值运算符具有动态异常说明 (C++17 前)noexcept 说明 (C++17 起)中所描述的异常说明。

因为每个类总是会声明复制赋值运算符，所以基类的赋值运算符始终会被隐藏。当使用 using 声明从基类带入复制赋值运算符，且它的实参类型与派生类的隐式复制赋值运算符的实参类型相同时，该 using 声明也会被隐式声明隐藏。

## 弃置的隐式声明的复制赋值运算符

如果满足下列任一条件，那么类 T 的隐式声明的复制赋值运算符被定义为弃置的：
    T 拥有用户声明的移动构造函数；
    T 拥有用户声明的移动赋值运算符。

否则，它被定义为预置的。
如果满足下列任一条件，那么类 T 的预置的复制赋值运算符被定义为弃置的：
    T 拥有具有 const 限定的非类类型（或它的数组）的非静态数据成员；
    T 拥有引用类型的非静态数据成员；
    T 拥有无法复制赋值的非静态数据成员或直接基类（对复制赋值的重载决议失败，或选择弃置或不可访问的函数）；
    T 是联合体式的类，且拥有的某个变体成员对应的复制赋值运算符是非平凡的。

## 平凡的复制赋值运算符

如果满足下列所有条件，那么类 T 的复制赋值运算符是平凡的：
    它不是用户提供的（即它是隐式定义或预置的）；
    T 没有虚成员函数；
    T 没有虚基类；
    为 T 的每个直接基类选择的复制赋值运算符都是平凡的；
    为 T 的每个类类型（或类类型的数组）的非静态数据成员选择的复制赋值运算符都是平凡的。
平凡复制赋值运算符如同用 std::memmove 进行对象表示的复制。所有与 C 语言兼容的数据类型（POD 类型）都可以平凡复制。

## 合格的复制赋值运算符

(C++11 前)
被用户声明或者同时被隐式声明且可定义的复制赋值运算符是合格的。

(C++11 起)(C++20 前)
没有被弃置的复制赋值运算符是合格的。

(C++20 起)
满足下列所有条件的复制赋值运算符是合格的：
    它没有被弃置，且
    满足它的所有关联约束（如果存在），且
    没有比它更受约束且拥有相同的第一形参类型和相同的 cv 或引用限定符（如果存在）的复制赋值运算符。

合格复制赋值运算符的平凡性确定该类是否为可平凡复制类型。

## 隐式定义的复制赋值运算符

如果隐式声明的复制赋值运算符既没有被弃置也不平凡，那么当它被 ODR 式使用或用于常量求值 (C++14 起)时，它会被编译器定义（即生成并编译函数体）。对于联合体类型，隐式定义的复制赋值运算符（如同以 std::memmove）复制它的对象表示。对于非联合类类型（class 与 struct），编译器按照声明顺序对对象的各基类和非静态成员进行逐成员复制赋值，其中对标量进行内建赋值，而对类类型使用复制赋值运算符，

(C++14 起)(C++23 前)
如果满足下列所有条件，那么类 T 的隐式定义的复制赋值运算符是 constexpr 的：
    T 是字面类型，且
    复制每个直接基类子对象时选中的赋值运算符都是 constexpr 函数，且
    复制 T 的每个类（或它的数组）类型的数据成员时选中的赋值运算符都是 constexpr 函数。

(C++23 起)
类 T 的隐式定义的复制赋值运算符是 constexpr 的。

(C++11 起)
当 T 拥有用户定义的析构函数或用户定义的复制构造函数时，隐式定义的复制赋值运算符的生成被弃用。

注解

如果复制和移动赋值运算符都有提供，那么重载决议会在实参是右值（例如无名临时量的纯右值 或 std::move 的结果的亡值）时选择移动赋值，而在实参是左值（具名对象或返回左值引用的函数或运算符）时选择复制赋值。如果只提供了复制赋值，那么重载决议对于所有值类别都会选择它（只要它按值或按到 const 的引用接收它的实参），从而当移动赋值不可用时，复制赋值将会成为它的后备。

隐式定义的复制赋值运算符是否会多次对在继承网格中可通过多于一条路径访问的虚基类子对象赋值是未指明的（同样适用于移动赋值）。

有关用户定义的复制赋值运算符应当有哪些行为，见赋值运算符重载。

```C++
#include <iostream>
#include <memory>
#include <string>
#include <algorithm>

struct A
{
     int n;
     std::string s1;

     A() = default;
     A(A const&) = default;

     // 用户定义的复制赋值（复制交换法）
     A& operator=(A other)
     {
         std::cout << "A 的复制赋值\n";
         std::swap(n, other.n);
         std::swap(s1, other.s1);
         return *this;
     }
};

struct B : A
{
     std::string s2;
     // 隐式定义的复制赋值
};

struct C
{
     std::unique_ptr<int[]> data;
     std::size_t size;

     // 用户定义的复制赋值（非复制交换法）
     // 注意：复制交换法总是会重新分配资源
     C& operator=(const C& other)
     {
         if (this != &other) // 非自赋值
         {
             if (size != other.size) // 资源无法复用
             {
                 data.reset(new int[other.size]);
                 size = other.size;
             }
             std::copy(&other.data[0], &other.data[0] + size, &data[0]);
         }
         return *this;
     }
};

int main()
{
     A a1, a2;
     std::cout << "a1 = a2 调用 ";
     a1 = a2; // 用户定义的复制赋值

     B b1, b2;
     b2.s1 = "foo";
     b2.s2 = "bar";
     std::cout << "b1 = b2 调用 ";
     b1 = b2; // 隐式定义的复制赋值

     std::cout << "b1.s1 = " << b1.s1 << " b1.s2 = " << b1.s2 << '\n';
}
```

输出：
a1 = a2 调用 A 的复制赋值
b1 = b2 调用 A 的复制赋值
b1.s1 = foo b1.s2 = bar

* * *

https://zh.cppreference.com/w/cpp/language/move_assignment

# 移动赋值运算符

类 T 的移动赋值运算符是名为 operator=的非模板非静态成员函数，它接受恰好一个 T&&、const T&&、volatile T&& 或 const volatile T&& 类型的形参。

语法
类名 & 类名 ::operator= ( 类名 && )               (1)         (C++11 起)
类名 & 类名 ::operator= ( 类名 && ) = default;         (2)         (C++11 起)
类名 & 类名 ::operator= ( 类名 && ) = delete;         (3)         (C++11 起)

解释

1.  移动赋值运算符的典型声明。
2.  强制编译器生成移动赋值运算符。
3.  避免隐式移动赋值。

每当重载决议选择移动赋值运算符时，它都会被调用，例如当对象出现在赋值表达式左侧，而它的右侧是同类型或可隐式转换的类型的右值时。

典型的移动赋值运算符“窃取”实参曾保有的资源（例如指向动态分配对象的指针，文件描述符，TCP socket，输入输出流，运行的线程，等等），而非复制它们，并使得实参遗留在某个合法但不确定的状态。例如，从 std::string 或从 std::vector 移动赋值可能导致实参被置空。然而这并不保证会发生。移动赋值与普通赋值相比，它的定义较为宽松而非更严格；在完成时，普通赋值必须留下数据的两份副本，而移动赋值只要求留下一份。

## 隐式声明的移动赋值运算符

如果没有对类类型（struct、class 或 union）提供任何用户定义的移动赋值运算符，且满足下列所有条件：
-    没有用户声明的复制构造函数；
-    没有用户声明的移动构造函数；
-    没有用户声明的复制赋值运算符；
-    没有用户声明的析构函数，

那么编译器将声明一个，作为类的 inline public 成员，并拥有签名 T& T::operator=(T&&)。

类可以拥有多个移动赋值运算符，如 T& T::operator=(const T&&) 和 T& T::operator=(T&&)。当存在用户定义的移动赋值运算符时，用户仍然可以通过关键词 default 强迫编译器生成隐式声明的移动赋值运算符。

隐式声明（或在它的首个声明被预置）的移动赋值运算符具有动态异常说明 (C++17 前)noexcept 说明 (C++17 起)中所描述的异常说明。

因为每个类总是会声明赋值运算符（移动或复制），所以基类的赋值运算符始终被隐藏。当使用 using 声明从基类带入赋值运算符，且它的实参类型与派生类的隐式赋值运算符的实参类型相同时，该 using 声明也会被隐式声明隐藏。

## 弃置的隐式声明的移动赋值运算符

如果满足下列任一条件，那么类 T 的隐式声明或预置的移动赋值运算符被定义为弃置的：
    T 拥有 const 限定的非静态数据成员。
    T 拥有引用类型的非静态数据成员。
    T 拥有无法移动赋值（拥有被弃置、不可访问或有歧义的移动赋值运算符）的非静态数据成员或直接基类。
重载决议忽略被弃置的隐式声明的移动赋值运算符。

## 平凡的移动赋值运算符

如果满足下列所有条件，那么类 T 的移动赋值运算符是平凡的：
    它不是用户提供的（即它是隐式定义或预置的）；
    T 没有虚成员函数；
    T 没有虚基类；
    为 T 的每个直接基类选择的移动赋值运算符都是平凡的；
    为 T 的每个类类型（或类类型的数组）的非静态数据成员选择的移动赋值运算符都是平凡的；

平凡移动赋值运算符实施与平凡复制赋值运算符相同的动作，即如同以 std::memmove 进行对象表示的复制。所有与 C 兼容的数据类型（POD 类型）都可以平凡移动。

## 合格的移动赋值运算符

(C++20 前)
没有被弃置的移动赋值运算符是合格的。

(C++20 起)
满足下列所有条件的移动赋值运算符是合格的：
    它没有被弃置，且
    满足它的所有关联约束（如果存在），且
    没有比它更受约束且拥有相同的第一形参类型和相同的 cv 或引用限定符（如果存在）的移动赋值运算符。

合格移动赋值运算符的平凡性确定该类是否为该类是否为可平凡复制类型。

## 隐式定义的移动赋值运算符

如果隐式声明的移动赋值运算符既没有被弃置也不平凡，那么当它被 ODR 式使用或用于常量求值 (C++14 起)时，它会被编译器定义（即生成并编译函数体）。

对于联合体类型，隐式定义的移动赋值运算符（如用 std::memmove）复制它的对象表示。

对于非联合类类型（class 与 struct），移动赋值运算符按照声明顺序对对象的各直接基类和直接非静态成员进行完整的逐成员移动赋值，其中对标量用内建运算符，对数组用逐元素移动赋值，而对类类型用移动赋值运算符（非虚调用）。

(C++14 起)(C++23 前)
如果满足下列所有条件，那么类 T 的隐式定义的复制赋值运算符是 constexpr 的：
    T 是字面类型，且
    移动每个直接基类子对象时选中的赋值运算符都是 constexpr 函数，且
    移动 T 的每个类（或它的数组）类型的数据成员时选中的赋值运算符都是 constexpr 函数。

(C++23 起)
类 T 的隐式定义的复制赋值运算符是 constexpr 的。

与复制赋值一样，隐式定义的移动赋值运算符是否会多次对在继承网格中可通过多于一条路径访问的虚基类子对象赋值是未指明的：

```C++
struct V
{
     V& operator=(V&& other)
     {
         // 这可能会被调用一或两次
         // 如果调用两次，那么 'other' 是刚被移动的 V 子对象
         return *this;
     }
};
struct A : virtual V {}; // operator= 调用 V::operator=
struct B : virtual V {}; // operator= 调用 V::operator=
struct C : B, A {};       // operator= 调用 B::operator=，然后调用 A::operator=
                          // 但可能只调用一次 V::operator=

int main()
{
     C c1, c2;
     c2 = std::move(c1);
}
```

注解

如果复制和移动赋值运算符都有提供，那么重载决议会在实参是右值（例如无名临时量的纯右值 或 std::move 的结果的亡值）时选择移动赋值，而在实参是左值（具名对象或返回左值引用的函数或运算符）时选择复制赋值。如果只提供了复制赋值，那么重载决议对于所有值类别都会选择它（只要它按值或按到 const 的引用接收它的实参），从而当移动赋值不可用时，复制赋值将会成为它的后备。

隐式定义的移动赋值运算符是否会多次对在继承网格中可通过多于一条路径访问的虚基类子对象赋值是未指明的（同样适用于复制赋值）。
有关用户定义的移动赋值运算符应当有哪些行为，见赋值运算符重载。

```C++
#include <string>
#include <iostream>
#include <utility>

struct A
{
     std::string s;

     A() : s("测试") {}

     A(const A& o) : s(o.s) { std::cout << "移动失败！\n"; }

     A(A&& o) : s(std::move(o.s)) {}

     A& operator=(const A& other)
     {
          s = other.s;
          std::cout << "复制赋值\n";
          return *this;
     }

     A& operator=(A&& other)
     {
          s = std::move(other.s);
          std::cout << "移动赋值\n";
          return *this;
     }
};

A f(A a) { return a; }

struct B : A
{
     std::string s2;
     int n;
     // 隐式移动赋值运算符 B& B::operator=(B&&)
     // 调用 A 的移动赋值运算符
     // 调用 s2 的移动赋值运算符
     // 并进行 n 的逐位复制
};

struct C : B
{
     ~C() {} // 析构函数阻止隐式移动赋值
};

struct D : B
{
     D() {}
     ~D() {} // 析构函数本会阻止隐式移动赋值
     D& operator=(D&&) = default; // 无论如何都强制移动赋值
};

int main()
{
     A a1, a2;
     std::cout << "尝试从右值临时量移动赋值 A\n";
     a1 = f(A()); // 从右值临时量移动赋值
     std::cout << "尝试从亡值移动赋值 A\n";
     a2 = std::move(a1); // 从亡值移动赋值

     std::cout << "\n尝试移动赋值 B\n";
     B b1, b2;
     std::cout << "移动前，b1.s = \"" << b1.s << "\"\n";
     b2 = std::move(b1); // 调用隐式移动赋值
     std::cout << "移动后，b1.s = \"" << b1.s << "\"\n";

     std::cout << "\n尝试移动赋值 C\n";
     C c1, c2;
     c2 = std::move(c1); // 调用复制赋值运算符

     std::cout << "\n尝试移动赋值 D\n";
     D d1, d2;
     d2 = std::move(d1);
}
```

输出：
尝试从右值临时量移动赋值 A
移动赋值
尝试从亡值移动赋值 A
移动赋值

尝试移动赋值 B
移动前，b1.s = "测试"
移动赋值
移动后，b1.s = ""

尝试移动赋值 C
复制赋值

尝试移动赋值 D
移动赋值

==================

============================

# ==Thread==

https://zh.cppreference.com/w/cpp/thread
并发支持库

C++ 包含线程、原子操作、互斥、条件变量和 future 的内建支持。

## 线程

在标头 `<thread>` 定义

|     |     |
| --- | --- |
| thread(C++11) | 管理单独的线程(类) |
| jthread(C++20) | 有自动合并和取消支持的 std::thread(类) |

## 管理当前线程的函数

在命名空间 this_thread 定义

|     |     |
| --- | --- |
| yield(C++11) | 建议实现重新调度各执行线程(函数) |
| get_id(C++11) | 返回当前线程的线程 id(函数) |
| sleep_for(C++11) | 使当前线程的执行停止指定的时间段(函数) |
| sleep_until(C++11) | 使当前线程的执行停止直到指定的时间点(函数) |

## 线程取消

在标头 `<stop_token>` 定义

|     |     |
| --- | --- |
| stop_token(C++20) | 查询是否已经做出 std::jthread 取消请求的接口(类) |
| stop_source(C++20) | 表示请求停止一个或多个 std::jthread 的类(类) |
| stop_callback(C++20) | 在 std::jthread 取消上注册回调的接口(类模板) |

。。回调还需要注册？直接写在 执行体中不就可以了。

## 缓存大小访问

在标头 `<new>` 定义 (C++17)

|     |     |
| --- | --- |
| hardware\_destructive\_interference_size<br>hardware\_constructive\_interference_size | 避免假共享的最小偏移<br>促使真共享的最大偏移(常量) |

。。这2个确实是在一个格子里的。

## ==原子操作==

这些组建为细粒度的原子操作提供，允许无锁并发编程。涉及同一对象的每个原子操作，相对于任何其他原子操作是不可分的。原子对象不具有数据竞争。

(C++23 起)： `<stdatomic.h>` 以外的 C++ 标准库头文件不提供 _Atomic 宏或任何非宏的全局命名空间声明。

在标头 `<atomic>` 定义

### 原子类型

|     |     |
| --- | --- |
| atomic(C++11) | atomic 类模板及其针对布尔、整型和指针类型的特化(类模板) |
| atomic_ref(C++20) | 提供非原子对象上的原子操作(类模板) |

### 原子类型上的操作

。。有点多，但是不得不抄。。

|     |     |
| --- | --- |
| atomic\_is\_lock_free(C++11) | 检查对该原子类型的操作是否是无锁的(函数模板) |
| atomic_store(C++11)<br>atomic\_store\_explicit(C++11) | 原子地以非原子实参替换原子对象的值(函数模板)<br>。。。这个能用来解决 Double check lock 中 线程获得 另一根线程 写入一半的 对象 的 数据竞争 问题？感觉应该可以。 |
| atomic_load(C++11)<br>atomic\_load\_explicit(C++11) | 原子地获得存储于原子对象的值(函数模板) |
| atomic_exchange(C++11)<br>atomic\_exchange\_explicit(C++11) | 原子地以非原子实参的值替换原子对象的值，并返回该原子对象的旧值(函数模板) |
| atomic\_compare\_exchange_weak<br>atomic\_compare\_exchange\_weak\_explicit<br>atomic\_compare\_exchange_strong<br>atomic\_compare\_exchange\_strong\_explicit<br>(C++11)(C++11)(C++11)(C++11) | 原子地比较原子对象和非原子实参的值，若相等则进行 atomic\_exchange，若不相等则进行 atomic\_load(函数模板) |
| atomic\_fetch\_add(C++11)<br>atomic\_fetch\_add_explicit(C++11) | 将非原子值加到原子对象，并获得原子对象的先前值(函数模板)<br>。。。这里没有说是不是原子加，但是应该是的。 |
| atomic\_fetch\_sub(C++11)<br>atomic\_fetch\_sub_explicit(C++11) | 从原子对象减去非原子值，并获得原子对象的先前值(函数模板) |
| atomic\_fetch\_and(C++11)<br>atomic\_fetch\_and_explicit(C++11) | 将原子对象替换为与非原子实参逐位与的结果，并获得原子对象的先前值(函数模板) |
| atomic\_fetch\_or(C++11)<br>atomic\_fetch\_or_explicit(C++11) | 将原子对象替换为与非原子实参逐位或的结果，并获得原子对象的先前值(函数模板) |
| atomic\_fetch\_xor(C++11)<br>atomic\_fetch\_xor_explicit(C++11) | 将原子对象替换为与非原子实参逐位异或的结果，并获得原子对象的先前值(函数模板) |
| atomic_wait(C++20)<br>atomic\_wait\_explicit(C++20) | 阻塞线程直至被提醒且原子值更改(函数模板) |
| atomic\_notify\_one(C++20) | 提醒一个在 atomic_wait 中阻塞的线程(函数模板) |
| atomic\_notify\_all(C++20) | 提醒所有在 atomic_wait 中阻塞的线程(函数模板) |

### 标志类型及操作

|     |     |
| --- | --- |
| atomic_flag(C++11) | 免锁的布尔原子类型(类) |
| atomic\_flag\_test\_and\_set(C++11)<br>atomic\_flag\_test\_and\_set_explicit(C++11) | 原子地设置标志为 true 并返回其先前值(函数) |
| atomic\_flag\_clear(C++11)<br>atomic\_flag\_clear_explicit(C++11) | 原子地设置标志值为 false(函数) |
| atomic\_flag\_test(C++20)<br>atomic\_flag\_test_explicit(C++20) | 原子地返回标志的值(函数) |
| atomic\_flag\_wait(C++20)<br>atomic\_flag\_wait_explicit(C++20) | 阻塞线程，直至被提醒且标志更改(函数) |
| atomic\_flag\_notify_one(C++20) | 提醒一个在 atomic\_flag\_wait 中阻塞的线程(函数) |
| atomic\_flag\_notify_all(C++20) | 提醒所有在 atomic\_flag\_wait 中阻塞的线程(函数) |

### 初始化

|     |     |
| --- | --- |
| atomic_init(C++11)(C++20 中弃用) | 对默认构造的原子对象进行非原子初始化(函数模板) |
| ATOMIC\_VAR\_INIT(C++11)(C++20 中弃用) | 静态存储期的原子对象的常量初始化(宏函数) |
| ATOMIC\_FLAG\_INIT(C++11) | 将 std::atomic_flag 初始化为 false(宏常量) |

### 内存同步顺序

|     |     |
| --- | --- |
| memory_order(C++11) | 为给定的原子操作定义内存顺序约束(枚举) |
| kill_dependency(C++11) | 从 std::memory\_order\_consume 依赖树移除指定对象(函数模板) |
| atomic\_thread\_fence(C++11) | 通用的依赖内存顺序的栅栏同步原语(函数) |
| atomic\_signal\_fence(C++11)<br>在标头 &lt;stdatomic.h&gt; 定义 | 线程与执行于同一线程的信号处理函数间的栅栏(函数) |

。。。为原子操作 定义内存顺序约束。。自身是个枚举。不知道里面有什么。
。。这个在下面。

## C 兼容宏

|     |     |
| --- | --- |
| _Atomic(C++23) | 使得 _Atomic(T) 等同于 std::atomic <t>的兼容性宏(宏函数)</t> |

## ==互斥==

互斥算法避免多个线程同时访问共享资源。这会避免数据竞争，并提供线程间的同步支持。
在标头 `<mutex>` 定义

|     |     |
| --- | --- |
| mutex(C++11) | 提供基本互斥设施(类) |
| timed_mutex(C++11) | 提供互斥设施，实现有时限锁定(类) |
| recursive_mutex(C++11) | 提供能被同一线程递归锁定的互斥设施(类) |
| recursive\_timed\_mutex(C++11)<br>在标头 &lt;shared_mutex&gt; | 提供能被同一线程递归锁定的互斥设施，并实现有时限锁定(类) |
| 定义shared_mutex(C++17) | 提供共享互斥设施(类) |
| shared\_timed\_mutex(C++14) | 提供共享互斥设施并实现有时限锁定(类) |

## ==通用互斥管理==

在标头 `<mutex>` 定义

|     |     |
| --- | --- |
| lock_guard(C++11) | 实现严格基于作用域的互斥体所有权包装器(类模板) |
| scoped_lock(C++17) | 用于多个互斥体的免死锁 RAII 封装器(类模板) |
| unique_lock(C++11) | 实现可移动的互斥体所有权包装器(类模板) |
| shared_lock(C++14) | 实现可移动的共享互斥体所有权封装器(类模板) |
| defer\_lock\_t<br>try\_to\_lock_t<br>adopt\_lock\_t(C++11) | 用于指定锁定策略的标签类型(类) |
| defer_lock<br>try\_to\_lock<br>adopt_lock(C++11) | 用于指定锁定策略的标签常量(常量) |

## 通用锁定算法

|     |     |
| --- | --- |
| try_lock(C++11) | 试图通过重复调用 try_lock 获得互斥体的所有权(函数模板) |
| lock(C++11) | 锁定指定的互斥体，若任何一个不可用则阻塞(函数模板) |

## 单次调用

|     |     |
| --- | --- |
| once_flag(C++11) | 确保 call_once 只调用函数一次的帮助对象(类) |
| call_once(C++11) | 仅调用函数一次，即使从多个线程调用(函数模板) |

## ==条件变量==

条件变量是允许多个线程相互交流的同步原语。它允许一定量的线程等待（可以定时）另一线程的提醒，然后再继续。条件变量始终关联到一个互斥。

在标头 `<condition_variable>` 定义

|     |     |
| --- | --- |
| condition_variable(C++11) | 提供与 std::unique_lock 关联的条件变量(类) |
| condition\_variable\_any(C++11) | 提供与任何锁类型关联的条件变量(类) |
| notify\_all\_at\_thread\_exit(C++11) | 安排到在此线程完全结束时对 notify_all 的调用(函数) |
| cv_status(C++11) | 列出条件变量上定时等待的可能结果(枚举) |

## (C++20起) 信号量

信号量 (semaphore) 是一种轻量的同步原件，用于制约对共享资源的并发访问。在可以使用两者时，信号量能比条件变量更有效率。

在标头 `<semaphore>` 定义

|     |     |
| --- | --- |
| counting_semaphore(C++20) | 实现非负资源计数的信号量(类模板) |
| binary_semaphore(C++20) | 仅拥有二个状态的信号量(typedef) |

## C++20 锁存器与屏障

锁存器 (latch) 与屏障 (barrier) 是线程协调机制，允许任何数量的线程阻塞直至期待数量的线程到达该屏障。锁存器不能复用，屏障能重复使用。

在标头 `<latch>` 定义

|     |     |
| --- | --- |
| latch(C++20) | 单次使用的线程屏障(类) |

在标头 `<barrier>` 定义

|     |     |
| --- | --- |
| barrier(C++20) | 可复用的线程屏障(类模板) |

## ==Future==

标准库提供了一些工具来获取异步任务（即在单独的线程中启动的函数）的返回值，并捕捉其所抛出的异常。这些值在共享状态中传递，其中异步任务可以写入其返回值或存储异常，而且可以由持有该引用该共享态的 std::future 或 std::shared_future 实例的线程检验、等待或是操作这个状态。

在标头 `<future>` 定义

|     |     |
| --- | --- |
| promise(C++11) | 存储一个值以进行异步获取(类模板) |
| packaged_task(C++11) | 打包一个函数，存储其返回值以进行异步获取(类模板) |
| future(C++11) | 等待被异步设置的值(类模板) |
| shared_future(C++11) | 等待被异步设置的值（可能为其他 future 所引用）(类模板) |
| async(C++11) | 异步运行一个函数（有可能在新线程中执行），并返回保有其结果的 std::future(函数模板) |
| launch(C++11) | 指定 std::async 所用的运行策略(枚举) |
| future_status(C++11) | 指定在 std::future 和 std::shared_future 上的定时等待的结果(枚举) |

### Future 错误

|     |     |
| --- | --- |
| future_error(C++11) | 报告与 future 或 promise 有关的错误(类) |
| future_category(C++11) | 鉴别 future 错误类别(函数) |
| future_errc(C++11) | 鉴别 future 错误码(枚举) |


## Coroutine

https://en.cppreference.com/w/cpp/language/coroutines





* * *

https://zh.cppreference.com/w/c/thread
并发支持库
C 提供线程、原子操作、互斥、条件变量及线程特定存储的内建支持。

* * *

https://zh.cppreference.com/w/cpp/language/memory_model

# ==内存模型==

为 C++ 抽象机的目的定义了计算机内存存储的语义。
可为 C++ 程序所用的内存是一或多个字节的连续序列。内存中的每个字节拥有唯一的地址。

## 字节

字节（byte）是最小的可寻址内存单元。它被定义为相接的位序列，其大到足以保有
    任何 UTF-8 编码单元（256 个相异值）和 (C++14 起)
    基础执行字符集的任何成员。 (C++23 前)
    基础字面量字符集的任何成员的通常字面量编码。 (C++23 起)

与 C 相似，C++ 也支持 8 位或更大的字节。

char、unsigned char 和 signed char 类型把一个字节用于存储和值表示。字节中的位数可作为 CHAR_BIT 或 `std::numeric_limits<unsigned char>::digits` 访问。

## 内存位置

一个内存位置是
    一个标量类型（算术类型、指针类型、枚举类型或 std::nullptr_t）对象
    或非零长位域的最大相接序列

注意：语言的各种功能特性，例如引用和虚函数，可能涉及到程序不可访问，但为实现所管理的额外内存位置。

```C++
struct S {
     char a;      // 内存位置 #1
     int b : 5;   // 内存位置 #2
     int c : 11, // 内存位置 #2 （延续）
           : 0,
         d : 8;   // 内存位置 #3
     struct {
         int ee : 8; // 内存位置 #4
     } e;
} obj; // 对象 'obj' 由 4 个分离的内存位置组成
```

## 线程与数据竞争

执行线程是程序中的控制流，它始于 std::thread::thread、std::async 或以其他方式所进行的顶层函数调用。
任何线程都能潜在地访问程序中的任何对象（拥有自动或线程局部存储期的对象仍然可以被另一线程通过指针或引用访问）。
不同的执行线程始终可以同时访问（读和写）不同的内存位置，不需要干涉或同步的任何要求。
当某个表达式的求值写入某个内存位置，而另一求值读或修改同一内存位置时，称这些表达式冲突。拥有两个冲突的求值的程序就有数据竞争，除非
    两个求值都在同一线程上，或者在同一信号处理函数中执行，或
    两个冲突的求值都是原子操作（见 std::atomic ），或
    一个冲突的求值发生早于（happens-before）另一个（见 std::memory_order）
若出现数据竞争，则程序的行为未定义。

```C++
int cnt = 0;
auto f = [&]{cnt++;};
std::thread t1{f}, t2{f}, t3{f}; // 未定义行为

std::atomic<int> cnt{0};
auto  f = [&]{cnt++;};
std::thread t1{f}, t2{f}, t3{f}; // OK
```

## 内存顺序

当线程从某个内存位置读取值时，它可能看到初值，同一线程所写入的值，或另一线程所写入的值。有关线程所作的写入操作对其他线程变为可见的顺序上的细节，见 std::memory_order。

向前进展
免妨碍
当只有一个未在标准库函数中阻塞的线程执行某个免锁的原子函数时，保证该执行将会完成（所有标准库免锁操作均为免妨碍的）

免锁

当一或多个免锁原子函数同时运行时，保证其中至少一个将会完成（所有标准库免锁操作均为免锁的——确保其他线程不能不确定地活锁它们（例如以连续窃取缓存线的方式），是实现的工作）。

进展保证
合法的 C++ 程序中，每个线程最终要做下列之一：
    终止
    调用 I/O 库函数
    通过 volatile 泛左值进行访问
    进行原子操作或同步操作
这允许编译器移除所有无可观察行为的循环，而不必证明他们终将终止。因为实现可以假定没有线程能在不做任何这些可观察行为的情况下永远执行。

若线程执行了上述步骤之一（I/O、volatile、原子或同步操作），阻塞于标准库函数中，或调用由于某个未阻塞的并发线程而未能完成的原子免锁函数，则称它取得进展（make progress）。

* * *

https://zh.cppreference.com/w/c/language/memory_model

# C内存模型

* * *

https://zh.cppreference.com/w/cpp/atomic/memory_order

# memory order

在标头 `<atomic>` 定义

```C++
(C++11 起)(C++20 前)
typedef enum memory_order {
     memory_order_relaxed,
     memory_order_consume,
     memory_order_acquire,
     memory_order_release,
     memory_order_acq_rel,
     memory_order_seq_cst
} memory_order;

(C++20 起)
enum class memory_order : /*unspecified*/ {
     relaxed, consume, acquire, release, acq_rel, seq_cst
};
inline constexpr memory_order memory_order_relaxed = memory_order::relaxed;
inline constexpr memory_order memory_order_consume = memory_order::consume;
inline constexpr memory_order memory_order_acquire = memory_order::acquire;
inline constexpr memory_order memory_order_release = memory_order::release;
inline constexpr memory_order memory_order_acq_rel = memory_order::acq_rel;
inline constexpr memory_order memory_order_seq_cst = memory_order::seq_cst;
```

std::memory_order 指定内存访问，包括常规的非原子内存访问，如何围绕原子操作排序。在没有任何制约的多处理器系统上，多个线程同时读或写数个变量时，一个线程能观测到变量值更改的顺序不同于另一个线程写它们的顺序。其实，更改的顺序甚至能在多个读取线程间相异。一些类似的效果还能在单处理器系统上出现，因为内存模型允许编译器变换。

库中所有原子操作的默认行为提供序列一致顺序（见后述讨论）。该默认行为可能有损性能，不过可以给予库的原子操作额外的 std::memory_order 参数，以指定附加制约，在原子性外，编译器和处理器还必须强制该操作。

。。6，这都能控制。

常量
在标头 `<atomic>` 定义


---

https://zh.cppreference.com/w/cpp/atomic/memory_order

。。这里的 见下方xxx 的我没有复制。

|名称|解释|
|--|--|
|memory_order_relaxed 	|宽松操作：没有同步或定序约束，仅对此操作要求原子性（见下方宽松定序）。|
|memory_order_consume 	|有此内存定序的加载操作，在其影响的内存位置进行消费操作：当前线程中依赖于当前加载的值的读或写不能被重排到此加载之前。其他线程中对有数据依赖的变量进行的释放同一原子变量的写入，能为当前线程所见。在大多数平台上，这只影响到编译器优化（见下方释放-消费定序）。|
|memory_order_acquire 	|有此内存定序的加载操作，在其影响的内存位置进行获得操作：当前线程中读或写不能被重排到此加载之前。其他线程的所有释放同一原子变量的写入，能为当前线程所见（见下方释放-获得定序）。|
|memory_order_release 	|有此内存定序的存储操作进行释放操作：当前线程中的读或写不能被重排到此存储之后。当前线程的所有写入，可见于获得该同一原子变量的其他线程（见下方释放-获得定序），并且对该原子变量的带依赖写入变得对于其他消费同一原子对象的线程可见（见下方释放-消费定序）。|
|memory_order_acq_rel 	|带此内存定序的读修改写操作既是获得操作又是释放操作。当前线程的读或写内存不能被重排到此存储之前或之后。所有释放同一原子变量的线程的写入可见于修改之前，而且修改可见于其他获得同一原子变量的线程。|
|memory_order_seq_cst 	|有此内存定序的加载操作进行获得操作，存储操作进行释放操作，而读修改写操作进行获得操作和释放操作，再加上存在一个单独全序，其中所有线程以同一顺序观测到所有修改（见下方序列一致定序）。 |



## memory\_order\_relaxed

宽松操作：没有同步或顺序制约，仅对此操作要求原子性（见下方宽松顺序）。

## memory\_order\_consume

有此内存顺序的加载操作，在其影响的内存位置进行消费操作：**当前线程中依赖于当前加载的该值的读或写不能被重排到此加载前**。其他释放同一原子变量的线程的对数据依赖变量的写入，为当前线程所可见。在大多数平台上，这只影响到编译器优化（见下方释放消费顺序）。

。。感觉是：本线程 加载 & 消费 一个变量，本线程的消费 不能被 重排序到 加载前。并且 其他线程的 写入，对于本线程可见。

。。但是，感觉说了点什么，又什么都没说。 消费 肯定不能 重排序 到 加载前啊。 难道 还有(sb) 会把 读写 重排 到 加载前？ 想不出场景。   那就只剩一个 其他线程的写入 对本线程可见。  不可见的话就是 类似 SQL 的 MVCC 了。 应该不会这么复杂(指 MVCC)吧， 内存还搞一个 事务， 不如直接外面套层锁。

## memory\_order\_acquire

有此内存顺序的加载操作，在其影响的内存位置进行获得操作：**当前线程中读或写不能被重排到此加载前**。其他释放同一原子变量的线程的所有写入，能为当前线程所见（见下方释放获得顺序）。

。。这里 比上一步 更严格， 上一步是，依赖于加载的值的 读写 不能被重排序。 这里是 不依赖于 加载的值的 读写 也不能 重排序。

## memory\_order\_release

有此内存顺序的存储操作进行释放操作：**当前线程中的读或写不能被重排到此存储后**。当前线程的所有写入，可见于获得该同一原子变量的其他线程（见下方释放获得顺序），并且对该原子变量的带依赖写入变得对于其他消费同一原子对象的线程可见（见下方释放消费顺序）。

。。要保存值到 内存中， 之前的 读写 不能被 重排序。

## memory\_order\_acq_rel

带此内存顺序的读修改写操作既是获得操作又是释放操作。**当前线程的读或写内存不能被重排到此存储前或后**。所有释放同一原子变量的线程的写入可见于修改之前，而且修改可见于其他获得同一原子变量的线程。

。。

## memory\_order\_seq_cst

有此内存顺序的加载操作进行获得操作，存储操作进行释放操作，而读修改写操作进行获得操作和释放操作，再加上存在一个单独全序，其中所有线程以同一顺序观测到所有修改（见下方序列一致顺序）。

。。

。。。不知道。。 不理解。。

正式描述
线程间同步和内存顺序决定表达式的求值和副效应如何在不同的执行线程间排序。它们用下列

##术语定义：

### 先序于

在同一线程中，求值 A 可以先序于求值 B ，如求值顺序中所描述。

携带依赖
在同一线程中，若下列任一为真，则先序于求值 B 的求值 A 可能也会将依赖带入 B （即 B 依赖于 A ）

1.  A 的值被用作 B 的运算数，除了
    a) B 是对 std::kill_dependency 的调用
    b) A是内建 && 、 || 、 ?: 或 , 运算符的左运算数。
2.  A 写入标量对象 M，B 从 M 读取
3.  A 将依赖携带入另一求值 X ，而 X 将依赖携带入 B

修改顺序
对任何特定的原子变量的修改，以限定于此一原子变量的单独全序出现。
对所有原子操作保证下列四个要求：

1.  写写连贯：
    若修改某原子对象 M 的求值 A （写操作）先发生于修改 M 的求值 B ，则 A 在 M 的修改顺序中早于 B 出现。
2.  读读连贯：

若某原子对象 M 的值计算 A （读操作）先发生于对 M 的值计算 B ，且 A 的值来自对 M 的写操作 X ，则 B 的值要么是 X 所存储的值，要么是在 M 的修改顺序中后于 X 出现的 M 上的副效应 Y 所存储的值。

3.  读写连贯：
    若某原子对象 M 的值计算 A （读操作）先发生于 M 上的操作 B （写操作），则 A 的值来自 M 的修改顺序中早于 B 出现的副效应 X （写操作）。
4.  写读连贯：

若原子对象 M 上的副效应 X （写操作）先发生于 M 的值计算 B （读操作），则求值 B 应从 X 或从 M 的修改顺序中后随 X 的副效应 Y 取得其值。

### 释放序列

在原子对象 M 上执行一次释放操作 A 之后， M 的修改顺序的最长连续子序列由下列内容组成

1.  由执行 A 的同一线程所执行的写操作 (C++20 前)
2.  任何线程对 M 的原子的读-修改-写操作
    被称为以 A 为首的释放序列。

同步于

如果在线程A上的一个原子存储是释放操作，在线程B上的对相同变量的一个原子加载是获得操作，且线程B上的加载读取由线程A上的存储写入的值，则线程A上的存储同步于线程B上的加载。

此外，某些库调用也可能定义为同步于其它线程上的其它库调用。

依赖先序于
在线程间，若下列任一为真，则求值 A 依赖先序于求值 B

1.  A 在某原子对象 M 上进行释放操作，而不同的线程中， B 在同一原子对象 M 上进行消费操作，而 B 读取 A 所引领的释放序列的任何部分 (C++20 前)所写入的值。
    
2.  A 依赖先序于 X 且 X 携带依赖到 B 。
    

### 线程间先发生于

在线程间，若下列任一为真，则求值 A 线程间先发生于求值 B

1.  A 同步于 B
2.  A 依赖先序于 B
3.  A 同步于某求值 X ，而 X 先序于 B
4.  A 先序于某求值 X ，而 X 线程间先发生于 B
5.  A 线程间先发生于某求值 X ，而 X 线程间先发生于 B

先发生于
无关乎线程，若下列任一为真，则求值 A 先发生于求值 B ：

1.  A 先序于 B
2.  A 线程间先发生于 B
    要求实现确保先发生于关系是非循环的，若有必要则引入额外的同步（若引入消费操作，它才可能为必要

若一次求值修改一个内存位置，而其他求值读或修改同一内存位置，且至少一个求值不是原子操作，则程序的行为未定义（程序有数据竞争），除非这两个求值之间存在先发生于关系。

C++20起
简单先发生于
无关乎线程，若下列之一为真，则求值 A 简单先发生于求值 B ：

1.  A 先序于 B
2.  A 同步于 B
3.  A 简单先发生于 X ，而 X 简单先发生于 B
    注：不计消费操作，则简单先发生于与强先发生于关系是相同的

### 强先发生于

无关乎线程，若下列之一为真，则求值 A 强先发生于求值 B ：
C++20前

1.  A 先序于 B
2.  A 同步于 B
3.  A 强先发生于 X ，而 X 强先发生于 B

C++20后

1.  A 先序于 B
2.  A 同步于 B ，且 A 与 B 均为序列一致的原子操作
3.  A 先序于 X ， X 简单先发生于 Y ，而 Y 先序于 B
4.  A 强先发生于 X ，而 X 强先发生于 B
    注：非正式而言，若 A 强先发生于 B ，则在所有环境中 A 均显得在 B 之前得到求值。
    注：强先发生于排除消费操作。

### 可见副效应

若下列皆为真，则标量 M 上的副效应 A （写入）相对于 M 上的值计算（读取）可见：

1.  A 先发生于 B
2.  没有其他对 M 的副效应 X 满足 A 先发生于 X 且 X 先发生于 B

若副效应 A 相对于值计算 B 可见，则修改顺序中，满足 B 不先发生于它的对 M 的副效应的最长相接子集，被称为副效应的可见序列。（ B 所确定的 M 的值，将是这些副效应之一所存储的值）

注意：线程间同步可归结为避免数据竞争（通过建立先发生于关系），及定义在何种条件下哪些副效应成为可见。

## 消费操作

带 memory\_order\_consume 或更强标签的原子加载是**消费操作**。注意 std::atomic\_thread\_fence 会施加比消费操作更强的同步要求。

## 获得操作

带 memory\_order\_acquire 或更强标签的原子加载是**获得操作**。互斥体 (Mutex) 上的 lock() 操作亦为获得操作。注意 std::atomic\_thread\_fence 会施加比获得操作更强的同步要求。

## 释放操作

带 memory\_order\_release 或更强标签的原子存储是**释放操作**。互斥体 (Mutex) 上的 unlock() 操作亦为释放操作。注意 std::atomic\_thread\_fence 会施加比释放操作更强的同步要求。

解释

## 宽松顺序

带标签 memory\_order\_relaxed 的原子操作不是同步操作；它们不会为并发的内存访问行为添加顺序约束。它们只保证原子性和修改顺序的一致性。
例如，对于初始值为零的x 和 y ，
// 线程 1 ：
r1 = y.load(std::memory\_order\_relaxed); // A
x.store(r1, std::memory\_order\_relaxed); // B
// 线程 2 ：
r2 = x.load(std::memory\_order\_relaxed); // C
y.store(42, std::memory\_order\_relaxed); // D

结果可能会为 r1 == 42 && r2 == 42。因为即使线程 1 中 A 先序于 B 且线程 2 中 C 先序于 D ，却没有约定去避免 D 中的对 y 的修改会在 A 之前 ， B 中的对 x 的修改会在 C 之前 。 D 的副效应为它对 y 修改可能可见于 A 的加载操作， B 的副效应为它对 x 修改可能可见于 C 的加载操作。

C++14起
即使使用宽松内存模型，也不允许“无中生有”的值循环地依赖于其各自的计算，例如，对于初始值为零的 x 和 y ，
// 线程1：
r1 = x.load(std::memory\_order\_relaxed);
if (r1 == 42) y.store(r1, std::memory\_order\_relaxed);
// 线程2：
r2 = y.load(memory\_order\_relaxed);
if (r2 == 42) x.store(42, std::memory\_order\_relaxed);

不会出现 r1 == 42 && r2 == 42 。因为 y = 42 只会在 x = 42 之后才会发生，而这反过来依赖于 y = 42 。注意在 C++14 之前的规范中只是不推荐这种情况，但不禁止。

宽松内存顺序的典型的应用是计数器自增，例如 std::shared\_ptr 的引用计数器，因为这只要求原子性，但不要求顺序或同步（注意 std::shared\_ptr 计数器自减要求与析构函数进行获得释放同步）

## 释放获得顺序

若线程 A 中的一个原子存储带标签 memory\_order\_release ，而线程 B 中来自同一变量的原子加载带标签 memory\_order\_acquire ，则从线程 A 的视角先发生于原子存储的所有内存写入（非原子及宽松原子的），在线程 B 中成为可见副效应，即一旦原子加载完成，则保证线程 B 能观察到线程 A 写入内存的所有内容。

同步仅建立在释放和获得同一原子对象的线程之间。其他线程可能看到与被同步线程的一者或两者相异的内存访问顺序。

在强顺序系统（ x86 、 SPARC TSO 、 IBM 主框架）上，释放获得顺序对于多数操作是自动进行的。无需为此同步模式添加额外的 CPU 指令，只有某些编译器优化受影响（例如，编译器被禁止将非原子存储移到原子存储-释放后，或将非原子加载移到原子加载-获得前）。在弱顺序系统（ ARM 、 Itanium 、 Power PC ）上，必须使用特别的 CPU 加载或内存栅栏指令。

互斥锁（例如 std::mutex 或原子自旋锁）是释放获得同步的例子：线程 A 释放锁而线程 B 获得它时，发生于线程 A 环境的临界区（释放之前）中的所有事件，必须对于执行同一临界区的线程 B （获得之后）可见。

```C++
std::atomic<std::string*> ptr;
int data;

void producer()
{
     std::string* p   = new std::string("Hello");
     data = 42;
     ptr.store(p, std::memory_order_release);
}

void consumer()
{
     std::string* p2;
     while (!(p2 = ptr.load(std::memory_order_acquire)))
         ;
     assert(*p2 == "Hello"); // 绝无问题
     assert(data == 42); // 绝无问题
}

int main()
{
     std::thread t1(producer);
     std::thread t2(consumer);
     t1.join(); t2.join();
}
```

下例演示三个线程间传递性的释放获得顺序

```C++
std::vector<int> data;
std::atomic<int> flag = {0};

void thread_1()
{
     data.push_back(42);
     flag.store(1, std::memory_order_release);
}

void thread_2()
{
     int expected=1;

     while (!flag.compare_exchange_strong(expected, 2, std::memory_order_acq_rel)) {

         expected = 1;
     }
}

void thread_3()
{
     while (flag.load(std::memory_order_acquire) < 2)
         ;
     assert(data.at(0) == 42); // 决不出错
}

int main()
{
     std::thread a(thread_1);
     std::thread b(thread_2);
     std::thread c(thread_3);
     a.join(); b.join(); c.join();
}
```

## 释放消费顺序

若线程 A 中的原子存储带标签 memory\_order\_release 而线程 B 中来自同一对象的读取存储值的原子加载带标签 memory\_order\_consume ，则线程 A 视角中先发生于原子存储的所有内存写入（非原子和宽松原子的），会在线程 B 中该加载操作所携带依赖进入的操作中变成可见副效应，即一旦完成原子加载，则保证线程B中，使用从该加载获得的值的运算符和函数，能见到线程 A 写入内存的内容。

同步仅在释放和消费同一原子对象的线程间建立。其他线程能见到与被同步线程的一者或两者相异的内存访问顺序。

所有异于 DEC Alpha 的主流 CPU 上，依赖顺序是自动的，无需为此同步模式产生附加的 CPU 指令，只有某些编译器优化收益受影响（例如，编译器被禁止牵涉到依赖链的对象上的推测性加载）。

此顺序的典型使用情况，涉及对很少被写入的数据结构（安排表、配置、安全策略、防火墙规则等）的共时读取，和有指针中介发布的发布者-订阅者情形，即当生产者发布消费者能通过其访问信息的指针之时：无需令生产者写入内存的所有其他内容对消费者可见（这在弱顺序架构上可能是昂贵的操作）。这种场景的例子之一是 rcu 解引用。

细粒度依赖链控制可参阅 std::kill\_dependency 及 \[\[carries\_dependency\]\] 。

注意到 2015 年 2 月为止没有产品编译器跟踪依赖链：均将消费操作提升为获得操作。

释放消费顺序的规范正在修订中，而且暂时不鼓励使用 memory\_order\_consume 。(C++17 起)

此示例演示用于指针中介的发布的依赖定序同步：int data不由数据依赖关系关联到指向字符串的指针，从而其值在消费者中未定义。

```C++
std::atomic<std::string*> ptr;
int data;

void producer()
{
     std::string* p   = new std::string("Hello");
     data = 42;
     ptr.store(p, std::memory_order_release);
}

void consumer()
{
     std::string* p2;
     while (!(p2 = ptr.load(std::memory_order_consume)))
         ;
     assert(*p2 == "Hello"); // 绝无出错： *p2 从 ptr 携带依赖
     assert(data == 42); // 可能也可能不会出错： data 不从 ptr 携带依赖
}

int main()
{
     std::thread t1(producer);
     std::thread t2(consumer);
     t1.join(); t2.join();
}
```

## 序列一致顺序

带标签 memory\_order\_seq_cst 的原子操作不仅以与释放/获得顺序相同的方式排序内存（在一个线程中先发生于存储的任何结果都变成进行加载的线程中的可见副效应），还对所有带此标签的内存操作建立单独全序。

。。跳了。。

### 与 volatile 的关系

在执行线程中，不能将通过 volatile 泛左值的访问（读和写）重排到同线程内先序于或后序于它的可观测副效应（包含其他 volatile 访问）后，但不保证另一线程观察到此顺序，因为 volatile 访问不建立线程间同步。

另外， volatile 访问不是原子的（共时的读和写是数据竞争），且不排序内存（非 volatile 内存访问可以自由地重排到 volatile 访问前后）。

一个值得注意的例外是 Visual Studio ，其中默认设置下，每个 volatile 写拥有释放语义，而每个 volatile 读拥有获得语义（ MSDN ），故而可将 volatile 对象用于线程间同步。标准的 volatile 语义不可应用于多线程编程，尽管它们在应用到 sig\_atomic\_t 对象时，足以与例如运行于同一线程的 std::signal 处理函数交流。

* * *

https://zh.cppreference.com/w/cpp/thread/condition_variable

# std::condition_variable

在标头 `<condition_variable>` 定义
class condition_variable;  (C++11 起)

condition\_variable 类是同步原语，能用于阻塞一个线程，或同时阻塞多个线程，直至另一线程修改共享变量（条件）并通知 condition\_variable 。

有意修改变量的线程必须
    获得 std::mutex （常通过 std::lock_guard ）
    在保有锁时进行修改
    在 std::condition\_variable 上执行 notify\_one 或 notify_all （不需要为通知保有锁）

即使共享变量是原子的，也必须在互斥下修改它，以正确地发布修改到等待的线程。

任何有意在 std::condition_variable 上等待的线程必须
    在与用于保护共享变量者相同的互斥上获得 std::unique_lock[std::mutex](std::mutex)
    执行下列之一：
        检查条件，是否为已更新或提醒它的情况
        执行 wait 、 wait\_for 或 wait\_until ，等待操作自动释放互斥，并悬挂线程的执行。

condition_variable 被通知时，时限消失或虚假唤醒发生，线程被唤醒，且自动重获得互斥。之后线程应检查条件，若唤醒是虚假的，则继续等待。

或者
        使用 wait 、 wait\_for 及 wait\_until 的有谓词重载，它们包揽以上三个步骤

std::condition_variable 只可与 `std::unique_lock<std::mutex>` 一同使用；此限制在一些平台上允许最大效率。 std::condition\_variable\_any 提供可与任何基本可锁定 (BasicLockable) 对象，例如 std::shared_lock 一同使用的条件变量。

condition\_variable 容许 wait 、 wait\_for 、 wait\_until 、 notify\_one 及 notify_all 成员函数的同时调用。

类 std::condition_variable 是标准布局类型 (StandardLayoutType) 。它非可复制构造 (CopyConstructible) 、可移动构造 (MoveConstructible) 、可复制赋值 (CopyAssignable) 或可移动赋值 (MoveAssignable) 。

成员类型
成员类型         定义
native\_handle\_type         实现定义

成员函数

|     |     |
| --- | --- |
| (构造函数) | 构造对象(公开成员函数) |
| (析构函数) | 析构对象(公开成员函数) |
| operator=\[被删除\] | 不可复制赋值(公开成员函数) |

通知

|     |     |
| --- | --- |
| notify_one | 通知一个等待的线程(公开成员函数) |
| notify_all | 通知所有等待的线程(公开成员函数) |

等待

|     |     |
| --- | --- |
| wait | 阻塞当前线程，直到条件变量被唤醒(公开成员函数) |
| wait_for | 阻塞当前线程，直到条件变量被唤醒，或到指定时限时长后(公开成员函数) |
| wait_until | 阻塞当前线程，直到条件变量被唤醒，或直到抵达指定时间点(公开成员函数) |

原生句柄

|     |     |
| --- | --- |
| native_handle | 返回原生句柄(公开成员函数) |

与 std::mutex 组合使用 condition_variable ，以促进线程间交流。

```C++
std::mutex m;
std::condition_variable cv;
std::string data;
bool ready = false;
bool processed = false;

void worker_thread()
{
     // 等待直至 main() 发送数据
     std::unique_lock<std::mutex> lk(m);
     cv.wait(lk,  []{return ready;});

     // 等待后，我们占有锁。
     std::cout << "Worker thread is processing data\n";
     data += " after processing";

     // 发送数据回 main()
     processed = true;
     std::cout << "Worker thread signals data processing completed\n";

     // 通知前完成手动解锁，以避免等待线程才被唤醒就阻塞（细节见 notify_one ）
     lk.unlock();
     cv.notify_one();
}

int main()
{
     std::thread worker(worker_thread);

     data = "Example data";
     // 发送数据到 worker 线程
     {
         std::lock_guard<std::mutex> lk(m);

         ready = true;
         std::cout << "main() signals data ready for processing\n";
     }
     cv.notify_one();

     // 等候 worker
     {
         std::unique_lock<std::mutex> lk(m);
         cv.wait(lk, []{return processed;});
     }
     std::cout << "Back in main(), data = " << data << '\n';

     worker.join();
}
```

输出：
main() signals data ready for processing
Worker thread is processing data
Worker thread signals data processing completed
Back in main(), data = Example data after processing

* * *

https://zh.cppreference.com/w/cpp/header/thread

# 标准库标头 `<thread>`

此头文件是线程支持库的一部分。

## thread_local

mutex
lock_guard
unique_lock

once_flag
call_once

lock

* * *

# mutex

https://zh.cppreference.com/w/cpp/thread/mutex
std::mutex
在标头 `<mutex>` 定义 class mutex; (C++11 起)

mutex 类是能用于保护共享数据免受从多个线程同时访问的同步原语。

mutex 提供 ==排他性 非递归 所有权== 语义：
    调用方线程从它成功调用 lock 或 try_lock 开始，到它调用 unlock 为止占有 mutex 。

线程占有 mutex 时，所有其他线程若试图要求 mutex 的所有权，则将阻塞（对于 lock 的调用）或收到 false 返回值（对于 try_lock ）.

调用方线程在调用 lock 或 try_lock 前必须不占有 mutex 。
。。看成 不必占用了， 所以在想：这不是可以递归吗。。。
。。 必 须 不。
。。就是 非可重入锁。not ReentrantLock。

若 mutex 在仍为任何线程所占有时即被销毁，或在占有 mutex 时线程终止，则行为未定义。 mutex 类满足互斥体 (Mutex) 和标准布局类型 (StandardLayoutType) 的全部要求。

std::mutex 既==不可复制亦不可移动==。

成员类型

|     |     |
| --- | --- |
| 成员类型 | 定义  |
| native\_handle\_type(可选) | 实现定义 |

成员函数

|     |     |
| --- | --- |
| (构造函数) | 构造mutex(公开成员函数) |
| (析构函数) | 销毁mutex(公开成员函数) |
| operator=\[delete\] | 不可复制赋值(公开成员函数) |

。。这个也是机翻的啊。 这个表格里，原文是： 构造互斥，销毁互斥，\[被删除\] 。。。

## 锁定

|     |     |
| --- | --- |
| lock | 锁定互斥，若互斥不可用则阻塞(公开成员函数) |
| try_lock | 尝试锁定互斥，若互斥不可用则返回(公开成员函数) |
| unlock | 解锁互斥(公开成员函数) |

原生句柄

|     |     |
| --- | --- |
| native_handle | 返回底层实现定义的原生句柄(公开成员函数) |

注意

通常不直接使用 std::mutex ： std::unique\_lock 、 std::lock\_guard 或 std::scoped_lock (C++17 起)以更加异常安全的方式管理锁定。

示例
此示例展示 mutex 能如何用于在保护共享于二个线程间的 std::map 。

```C++
#include <iostream>
#include <map>
#include <string>
#include <chrono>
#include <thread>
#include <mutex>

std::map<std::string, std::string> g_pages;
std::mutex g_pages_mutex;

void save_page(const std::string &url)
{
     // 模拟长页面读取
     std::this_thread::sleep_for(std::chrono::seconds(2));
     std::string result = "fake content";

     std::lock_guard<std::mutex> guard(g_pages_mutex);
     g_pages[url] = result;
}

int main()
{
     std::thread t1(save_page, "[http://foo](http://foo/)");
     std::thread t2(save_page, "[http://bar](http://bar/)");
     t1.join();
     t2.join();

     // 现在访问g_pages是安全的，因为线程t1/t2生命周期已结束
     for (const auto &pair : g_pages) {
         std::cout << pair.first << " => " << pair.second << '\n';
     }
}
```

。。这个锁的功能 有点隐晦。 第一眼是，2个不同的key ，也需要锁？。 后来想了下，map的扩容。hash冲突，拉链的时候。
。。感觉不如，100个线程，每个线程for 100次 对int++。

输出：
[http://bar](http://bar/) =\> fake content
[http://foo](http://foo/) =\> fake content

* * *

https://zh.cppreference.com/w/cpp/named_req/Mutex

# C++ 具名要求：互斥体 (Mutex)

互斥体 (Mutex) 要求扩展 可锁定 (Lockable) 要求以包含线程间同步。
。。可锁定 就是 try_lock

要求
    可锁定 (Lockable)
    可默认构造 (DefaultConstructible)
    可析构 (Destructible)
    不可复制
    不可移动

对于互斥 (互斥体 (Mutex) ) 类型的对象 m ：
    表达式 m.lock() 有下列属性
        表现原子操作。
        阻塞调用方线程，直到能获得互斥的排他性所有权为止。
        先于同一互斥上的 m.unlock() 的操作同步于此锁操作（等价于释放获得 std::memory_order ）

若调用方线程已占有互斥，则行为未定义（除非 m 是 std::recursive\_mutex 或 std::recursive\_timed_mutex ）

错误时可能抛出 std::system_error 类型的异常，拥有下例错误码：
            若调用方线程无要求的权限，则为 std::errc::operation\_not\_permitted
            若实现检测出此线程将导致死锁，则为 std::errc::resource\_deadlock\_would_occur

表达式 m.try_lock() 拥有下列属性
        表现为原子操作。
        试图为调用方线程获得互斥的排他性所有权，而不阻塞。若未获得所有权，则立即返回。允许此函数虚假地失败并返回，即使互斥当前未为另一线程所占有。

若 try\_lock() 成功，则先于同一对象上 unlock() 的操作同步于此操作（等价于释放获得 std::memory\_order ）。 lock() 不与失败的 try_lock() 同步。

不抛异常。

表达式 m.unlock() 拥有下列属性
        表现为原子操作。
        释放调用方线程的互斥所有权，并同步于同一对象上的后继的成功锁操作。
        若调用方线程不占有互斥，则行为未定义。
        不抛异常。

单个互斥上的所有锁和解锁操作以能视为一个原子变量的修改顺序的单独全序发生：顺序对此单独互斥是特定的。

库类型
下列标准库类型满足互斥体 (Mutex) ：
-    std::mutex
-    std::recursive_mutex
-    std::timed_mutex
-    std::recursive\_timed\_mutex
-    std::shared_mutex

* * *

https://zh.cppreference.com/w/cpp/atomic/atomic

# std::atomic

在标头 `<atomic>` 定义

(1)         (C++11 起)
template&lt; class T &gt;
struct atomic;

(2)         (C++11 起)

```C++
template< class U >
struct atomic<U*>;
```

在标头 `<memory>` 定义

(3)         (C++20 起)

```C++
template<class U>
struct atomic<std::shared_ptr<U>>;
```

(4)         (C++20 起)

```C++
template<class U>
struct atomic<std::weak_ptr<U>>;
```

在标头 `<stdatomic.h>` 定义

(5)         (C++23 起)
#define _Atomic(T) /* 见下文 */

每个 std::atomic 模板的实例化和全特化定义一个原子类型。如果一个线程写入原子对象，同时另一线程从它读取，那么行为良好定义（数据竞争的细节见内存模型）。

另外，对原子对象的访问可以建立线程间同步，并按 std::memory_order 所对非原子内存访问定序。

std::atomic 既不可复制也不可移动。

(C++23 起)
兼容性宏 _Atomic 在 `<stdatomic.h>` 中提供，使得两者均良构时 _Atomic(T) 等同于 `std::atomic<T>` 。
包含 `<stdatomic.h>` 时未指定命名空间 std 中的任何声明是否可用。

## 特化

### 主模板

主 std::atomic 模板可用 任何 满足 可复制 构造 (CopyConstructible) 及 可复制 赋值 (CopyAssignable) 的 可 平凡 复制 (TriviallyCopyable) 类型 T 特化。如果下列任何值是 false，那么程序==非良构==：
-    `std::is_trivially_copyable<T>::value`
-    `std::is_copy_constructible<T>::value`
-    `std::is_move_constructible<T>::value`
-    `std::is_copy_assignable<T>::value`
-    `std::is_move_assignable<T>::value`
`std::atomic<bool>` 使用初等模板。它保证是标准布局类。

### 部分特化

标准库为下列类型提供 std::atomic 模板的特化，它们拥有初等模板所不拥有的额外属性：

2.  对所有指针类型的部分特化 `std::atomic<U*>`。这些特化拥有标准布局、平凡默认构造函数 (C++20 前)和平凡析构函数。除了为所有原子类型提供的操作，这些特化额外支持适合指针类型的原子算术运算，例如 fetch\_add、fetch\_sub。

。。这里确实没有1

(C++20 起)

3-4) 为 std::shared\_ptr 和 std::weak\_ptr 提供部分特化 `std::atomic<std::shared_ptr<U>>` 和 `std::atomic<std::weak_ptr<U>>`。

细节见 `std::atomic<std::shared_ptr> 和 std::atomic<std::weak_ptr>`。

### 对整数类型的特化

以下列整数类型之一实例化时，std::atomic 提供适合于整数类型的额外原子操作，例如 fetch\_add、fetch\_sub、fetch\_and、fetch\_or、fetch_xor：

字符类型 char、char8\_t (C++20 起)、char16\_t、char32\_t 和 wchar\_t；
        标准有符号整数类型：signed char、short、int、long 和 long long；

标准无符号整数类型：unsigned char、unsigned short、unsigned int、unsigned long 和 unsigned long long；

任何标头 <cstdint>中的 typedef 所需的额外整数类型。</cstdint>

另外，结果的 std::atomic&lt;整数&gt; 特化拥有标准布局、平凡默认构造函数 (C++20 前)和平凡析构函数。定义有符号整数算术为使用补码；无未定义结果。

C++20起

### 对浮点类型的特化

以无 cv 限定的浮点类型（float、double、long double 和无 cv 限定的扩展浮点类型 (C++23 起)）实例化时，std::atomic 提供适合于浮点类型的额外原子操作，例如 fetch\_add 和 fetch\_sub。

另外，结果的 std::atomic&lt;浮点&gt; 特化拥有标准布局和平凡析构函数。
无操作导致未定义行为，即使结果不能以浮点类型表示。有效的浮点环境可能与调用方线程的浮点环境不同。

## 类型别名

为 bool 和所有上面列出的整数类型提供如下类型别名：
。。太多了。随便抄几个。
(不是)所有 std::atomic&lt;整数&gt; 别名

|     |     |
| --- | --- |
| atomic_bool(C++11) | `std::atomic<bool>(typedef)` |
| atomic_uchar(C++11) | `std::atomic<unsigned char>(typedef)` |
| atomic_uint(C++11) | `std::atomic<unsigned int>(typedef)` |
| atomic_ullong(C++11) | `std::atomic<unsigned long long>(typedef)` |
| atomic\_char8\_t(C++20) | `std::atomic<char8_t>(typedef)` |
| atomic\_wchar\_t(C++11) | `std::atomic<wchar_t>(typedef)` |
| atomic\_uint64\_t(C++11)(可选) | `std::atomic<std::uint64_t>(typedef)` |
| atomic\_uint\_least64_t(C++11) | `std::atomic<std::uint_least64_t>(typedef)` |
| atomic\_uint\_fast64_t(C++11) | `std::atomic<std::uint_fast64_t>(typedef)` |
| atomic\_intptr\_t(C++11)(可选) | `std::atomic<std::intptr_t>(typedef)` |
| atomic\_ptrdiff\_t(C++11) | `std::atomic<std::ptrdiff_t>(typedef)` |
| atomic\_uintmax\_t(C++11) | `std::atomic<std::uintmax_t>(typedef)` |

特殊用途类型别名

|     |     |
| --- | --- |
| atomic\_signed\_lock_free(C++20) | 免锁且对于等待/提醒最高效的有符号整数原子类型(typedef) |
| atomic\_unsigned\_lock_free(C++20) | 免锁且对于等待/提醒最高效的无符号整数原子类型(typedef) |

注意：std::atomic\_intN\_t、std::atomic\_uintN\_t、std::atomic\_intptr\_t 和 atomic\_uintptr\_t 分别在当且仅当定义了 std::intN\_t、std::uintN\_t、std::intptr\_t 和 std::uintptr\_t 时才会有定义。

(C++20 起)
std::atomic\_signed\_lock\_free 与 std::atomic\_unsigned\_lock\_free 在自立实现中可选。

成员类型

|     |     |
| --- | --- |
| 成员类型 | 定义  |
| value_type | T (无论是否特化) |
| difference_type | value_type (仅对 atomic&lt;整数&gt; 和 atomic&lt;浮点&gt; (C++20 起) 特化)<br>std::ptrdiff_t (仅对 `std::atomic<U*>` 特化) |

difference\_type 不在主 std::atomic 模板中，或不在对 std::shared\_ptr 和 std::weak_ptr 的部分特化中定义。

成员函数

|     |     |
| --- | --- |
| (构造函数) | 构造原子对象(公开成员函数) |
| operator= | 存储值于原子对象(公开成员函数) |
| is\_lock\_free | 检查原子对象是否免锁(公开成员函数) |
| store | 原子地以非原子对象替换原子对象的值(公开成员函数) |
| load | 原子地获得原子对象的值(公开成员函数) |
| operator T | 从原子对象加载值(公开成员函数) |
| exchange | 原子地替换原子对象的值并获得它先前持有的值(公开成员函数) |
| compare\_exchange\_weakcompare\_exchange\_strong | 原子地比较原子对象与非原子参数的值，若相等则进行交换，若不相等则进行加载(公开成员函数) |
| wait(C++20) | 阻塞线程直至被提醒且原子值更改(公开成员函数) |
| notify_one(C++20) | 提醒至少一个在原子对象上的等待中阻塞的线程(公开成员函数) |
| notify_all(C++20) | 提醒所有在原子对象上的等待中阻塞的线程(公开成员函数) |

常量

|     |     |
| --- | --- |
| is\_always\_lock_free\[静态\] | (C++17)指示该类型是否始终免锁(公开静态成员常量) |

特化成员函数
fetch\_add, fetch\_sub, fetch\_and, fetch\_or, fetch_xor
operator++, operator++(int), operator--, operator--(int)
operator+=, operator-=, operator&=, operator|=, operator^=

注解

存在等价于 std::atomic 所有成员函数的非成员函数模板。这些非成员函数可以额外对非 std::atomic 特化的类型重载，但不能保证原子性。标准库中仅有的这种类型是 std::shared_ptr <u>。</u>

C 中 _Atomic 是关键词并用于提供原子类型。

推荐实现确保对于每个可能的类型 T，C 中 _Atomic(T) 的表示与 C++ 中 std::atomic <t>的相同。用于确保原子性与内存顺序的机制应该兼容。</t>

gcc 和 clang 上，此处描述的某些功能要求通过 -latomic 链接。

* * *

https://zh.cppreference.com/w/cpp/thread/thread

# 在标头 `<thread>` 定义

class thread;  (C++11 起)

类 thread 表示单个执行线程。线程允许多个函数同时执行。

线程在构造关联的线程对象时立即开始执行（等待任何OS调度延迟），从提供给作为构造函数参数的顶层函数开始。顶层函数的返回值将被忽略，而且若它以抛异常终止，则调用 std::terminate 。顶层函数可以通过 std::promise 或通过修改共享变量（可能需要同步，见 std::mutex 与 std::atomic ）将其返回值或异常传递给调用方。

std::thread 对象也可能处于不表示任何线程的状态（默认构造、被移动、 detach 或 join 后），并且执行线程可能与任何 thread 对象无关（ detach 后）。

没有两个 std::thread 对象会表示同一执行线程； std::thread 不是可复制构造 (CopyConstructible) 或可复制赋值 (CopyAssignable) 的，尽管它可移动构造 (MoveConstructible) 且可移动赋值 (MoveAssignable) 。

成员类型         定义

|     |     |
| --- | --- |
| native\_handle\_type (可选) | 实现定义 |

成员类

|     |     |
| --- | --- |
| id  | 表示线程的 id(公开成员类) |

成员函数

|     |     |
| --- | --- |
| (构造函数) | 构造新的 thread 对象(公开成员函数) |
| (析构函数) | 析构 thread 对象，必须合并或分离底层线程(公开成员函数) |
| operator= | 移动 thread 对象(公开成员函数) |

## 观察器

|     |     |
| --- | --- |
| joinable | 检查线程是否可合并，即潜在地运行于平行环境中(公开成员函数) |
| get_id | 返回线程的 id(公开成员函数) |
| native_handle | 返回底层实现定义的线程句柄(公开成员函数) |
| hardware_concurrency\[静态\] | 返回实现支持的并发线程数(公开静态成员函数) |

## 操作

|     |     |
| --- | --- |
| join | 等待线程完成其执行(公开成员函数) |
| detach | 容许线程从线程句柄独立开来执行(公开成员函数) |
| swap | 交换二个 thread 对象(公开成员函数) |

非成员函数

|     |     |
| --- | --- |
| std::swap(std::thread)(C++11) | 特化 std::swap 算法(函数) |

* * *

https://zh.cppreference.com/w/cpp/thread/jthread

# std::jthread

在标头 `<thread>` 定义

```C++
class jthread;      (C++20 起)
```

类 jthread 表示单个执行线程。它拥有通常同 std::thread 的行为，除了 jthread 在==析构时自动再结合，而且能在具体情况下取消/停止==。

线程在构造关联的线程对象时（在任何操作系统调度延迟后）立即开始执行，始于作为构造函数参数提供的顶层函数。忽略顶层函数的返回值，而若它因抛异常退出，则调用 std::terminate 。顶层函数可经由 std::promise 向调用方交流其返回值或异常，或通过修改共享变量（要求同步，见 std::mutex 与 std::atomic ）。

不同于 std::thread ， jthread 逻辑上保有一个内部的 std::stop_source 类型私有成员，它维持共享==停止状态==。 jthread 的构造函数接受一个 std::stop\_token 作为其首参数， jthread 将从其内部的 stop\_source 传递它。这允许==函数在其执行中检查是否已请求停止，而若已请求则返回==。

std::jthread 对象亦可在不表示任何线程的状态（在默认构造、被移动、 detach 或 join 后），而执行线程可以与任何 jthread 对象关联（ detach 后）。

没有二个 std::jthread 对象可表示同一执行线程； std::jthread 非可复制构造 (CopyConstructible) 或可复制赋值 (CopyAssignable) ，尽管它为可移动构造 (MoveConstructible) 及可移动赋值 (MoveAssignable) 。

成员类型         定义

|     |     |
| --- | --- |
| id  | std::thread::id |
| native\_handle\_type (可选) | std::thread::native\_handle\_type |

成员函数

|     |     |
| --- | --- |
| (构造函数) | 创建新的 jthread 对象(公开成员函数) |
| (析构函数) | 若 joinable() 为 true ，则调用 request_stop() 然后 join() ；不论如何都会销毁 jthread 对象。(公开成员函数) |
| operator= | 移动 jthread 对象(公开成员函数) |

观察器

|     |     |
| --- | --- |
| joinable | 检查线程是否可合并，即潜在地运行于平行环境中(公开成员函数) |
| get_id | 返回线程的 id(公开成员函数) |
| native_handle | 返回底层实现定义的线程句柄(公开成员函数) |
| hardware_concurrency\[静态\] | 返回实现支持的并发线程数(公开静态成员函数) |

操作

|     |     |
| --- | --- |
| join | 等待线程完成其执行(公开成员函数) |
| detach | 容许线程从线程句柄独立开来执行(公开成员函数) |
| swap | 交换二个 jthread 对象(公开成员函数) |

## 停止记号处理

|     |     |
| --- | --- |
| get\_stop\_source | 返回与线程的停止状态关联的 stop_source 对象(公开成员函数) |
| get\_stop\_token | 返回与线程的共享停止状态关联的 stop_token(公开成员函数) |
| request_stop | 请求执行经由线程的共享停止状态停止(公开成员函数) |

非成员函数

|     |     |
| --- | --- |
| swap(std::jthread)(C++20) | 特化 std::swap 算法(函数) |

============================

https://zh.cppreference.com/w/cpp/named_req/StandardLayoutType

# C++ 具名要求：标准布局类型 (StandardLayoutType)

指定一个类型为标准布局类型。标准布局类型适用于与其他语言编写的代码交流。

注意：标准中并没有定义具有这个名字的具名要求。这是核心语言所定义的一种类型类别。将它作为具名要求包含于此只是为了保持一致性。

要求
下列类型统称为标准布局类型：
    标量类型
    标准布局类类型
    上述类型的数组
    这些类型的有 cv 限定版本

标准布局
(C++11 前) 一个类只有在它是 POD 类的情况下是标准布局（standard-layout）的并拥有下述属性。

(C++11 起) 所有非静态数据成员均拥有相同访问控制，且满足其他特定条件的类被称作标准布局（standard-layout）类型（对它规定的列表见标准布局类）。

* * *

https://zh.cppreference.com/w/cpp/language/initialization

# 初始化

变量的初始化会在构造时提供变量的初始值。

初始值可以由声明符或 new 表达式的初始化器部分提供。在函数调用时也会发生：函数形参及函数返回值也会被初始化。

对于每个声明符，初始化器必须是下列之一：
( 表达式列表 )         (1)
= 表达式               (2)
{ 初始化器列表 }         (3)

1.  括号中的以逗号分隔的含有任意表达式和花括号初始化器列表的列表
2.  等号后面跟着一个表达式
3.  花括号初始化器列表：以逗号分隔且可以为空的含有表达式和其他花括号初始化器列表的列表

根据上下文，初始化器可以调用：
    值初始化，例如 std::string s{};
    直接初始化，例如 std::string s("hello");
    复制初始化，例如 std::string s = "hello";
    列表初始化，例如 std::string s{'a', 'b', 'c'};
    聚合初始化，例如 char a\[3\] = {'a', 'b'};
    引用初始化，例如 char& c = a\[0\];

如果不提供初始化器，那么就会应用默认初始化的规则。
初始化包括了对初始化器中的子表达式进行求值以及为函数实参和返回值创建临时对象。

## 非局部变量

所有具有静态存储期 的非局部变量的初始化 会作为 程序启动的 一部分 在 main 函数执行 之前 进行 (除非被延迟， 见下文)。 所有具有 线程 局部 存储期 的 非 局部变量的初始化会作为 线程启动 的一部分进行，按顺序 早于 线程函数的执行开始。对于这两种变量，初始化发生于两个截然不同的阶段：

## 静态初始化

有两种静态初始化的形式：

1.  如果可能，那么应用常量初始化。
2.  否则非局部静态及线程局域变量会被零初始化。
    实践中：
    常量初始化通常在编译期进行。预先被计算的对象表示会作为程序映像的一部分存储下来。如果编译器没有这样做，那么它仍然必须保证该初始化发生早于任何动态初始化。
    零初始化的变量将被置于程序映像的 .bss 段，它不占据磁盘空间，并在加载程序时由操作系统以零填充。

## 动态初始化

在所有静态初始化完成后，在下列情形中进行非局部变量的动态初始化：

1.  无序的动态初始化，仅适用于未被显式特化的（静态/线程局域）类模板的静态数据成员及变量模板 (C++14 起)。这些静态变量的初始化相对于所有其他动态初始化之间是顺序不确定的，除非程序在初始化某个变量之前开始了一个线程，此时初始化则是无顺序的 (C++17 起)。这些线程局域变量的初始化相对于所有其他动态初始化之间是无顺序的。
    
2.  (C++17 起) 部分有序的动态初始化，适用于并未被隐式或显式实例化的特化的所有 inline 变量。如果一个部分有序的 V 在每个翻译单元中比有序或部分有序的 W 更早定义，那么 V 的初始化按顺序早于（或若程序启动了线程，则为先发生于）W 的初始化。
    
3.  有序的动态初始化，适用于所有其他非局部变量：在单个翻译单元中，这些变量的初始化始终严格以其定义出现于源代码中的顺序定序。不同翻译单元中的静态变量的初始化之间是顺序不确定的。不同翻译单元中的线程局域变量的初始化之间是无顺序的。
    

当拥有静态或线程存储期的非局部变量的初始化通过异常退出时，调用 std::terminate。

### 提早动态初始化

在下列条件都满足的情况下，允许编译器将初始化动态初始化的变量作为静态初始化（实为编译期）的一部分：

1.  初始化的动态版本不改变命名空间作用域中任何先于其初始化的对象的值
2.  初始化的静态版本在被初始化变量中产生的值，与当所有不要求静态初始化的变量都被动态初始化时，由动态初始化所生成的值相同。

因为上述规则，如果某对象 o1 的初始化涉及到命名空间作用域对象 o2，而它**潜在地**要求动态初始化，但在同一翻译单元中在其之后定义，那么不指明所用的 o2 是完全初始化的 o2 的值（因为编译器把 o2 的初始化提升到编译时）还是 o2 仅被零初始化的值。

```C++
inline double fd() { return 1.0; }
extern double d1;
double d2 = d1;    // 未指明：
                   // 如果 d1 被动态初始化则动态初始化为 0.0，或
                   // 如果 d1 被静态初始化则动态初始化为 1.0，或
                   // 静态初始化为 0.0（因为当两个变量都被动态初始化时将为这个值）
double d1 = fd(); // 可能静态或动态初始化为 1.0
```

### 延迟动态初始化

动态初始化是发生早于（对于静态变量）主函数或（对于线程局域变量）其线程的首个函数的首条语句，还是延迟到发生晚于它们，是由实现定义的。

如果非内联变量 (C++17 起)的初始化延迟到发生晚于主/线程函数的首条语句，那么它发生早于与所初始化的变量定义于同一翻译单元中的任何拥有静态/线程存储期的变量的首次 ODR 使用。若给定翻译单元中没有 ODR 使用变量或函数，则定义于该翻译单元的非局部变量可能始终不被初始化（这模仿按需的动态库的行为）。然而，只要翻译单元中 ODR 使用了任何事物，就会初始化所有在初始化或销毁中拥有副作用的非局部变量，即使程序中没有用到它们。

如果内联变量的初始化被延迟，那么它早于这个特定变量的首次 ODR 使用发生。(C++17 起)

## 静态局部变量

有关局部（即块作用域）的静态和线程局部变量，见静态局部变量。
拥有 外部或内部链接 的变量的块作用域声明中 不允许 初始化器。这种声明必须带 extern 出现而且不能为定义。

## 类成员

非静态数据成员可以成员初始化器列表或以默认成员初始化器初始化。

注解

> 非局部变量的销毁顺序在 std::exit 中描述。

* * *

https://zh.cppreference.com/w/cpp/language/default_initialization

# 默认初始化

这是在不使用初始化器构造变量时执行的初始化。

语法

```C++
T 对象;         // 1
new T           // 2
new T ( ) (C++03 前)    // 2 。。是2
```

默认初始化在三种情况下进行：

1.  当不带初始化器而声明具有自动、静态或线程局部存储期的变量时；
2.  当以不带初始化器的 new 表达式创建具有动态存储期的对象时，或当以带有由一个空括号对组成的初始化器的 new 表达式创建对象时 (C++03 前)；
3.  当构造函数初始化器列表中未提及某个基类或非静态数据成员，且调用了该构造函数时。

默认初始化的效果是：

如果 T 是非 POD (C++11 前)类类型，那么考虑各构造函数并实施针对空实参列表的重载决议。调用所选的构造函数（即默认构造函数之一），以提供新对象的初始值；

如果 T 是数组类型，那么该数组的每个元素都被默认初始化；
    否则，不做任何事：具有自动存储期的对象（及其子对象）被初始化为不确定值。

只有具有自动存储期的（可有 cv 限定的）非 POD 类类型（或其数组）曾被认为在不使用初始化器时默认初始化。具有动态存储期的标量和 POD 类型曾被认为不被初始化（从 C++11 起，这种情况被重新分类为默认初始化的一种形式）。  (C++11 前)

在（引入值初始化的）C++03 前，表达式 new T()，以及带一个空括号对形式的指名基类或成员的成员初始化器，曾被分类为默认初始化，但对非类类型则指定为零初始化。  (C++03 前)

如果 T 是有 cv 限定的类型，那么默认初始化会采用其无 cv 限定版本。  (C++11 前)

## const 对象的默认初始化

如果程序调用有 const 限定类型的 T 对象的默认初始化，则 T 应当为 const 可默认构造类类型或其数组。

如果类类型 T 的默认初始化会调用用户提供（不是从基类继承） (C++11 起)的 T 构造函数，或满足以下条件，那么 T 为 const 可默认构造：
(C++11 前)
    T 的每个直接非静态数据成员 M 拥有类类型 X （或其数组）， X 为 const 可默认构造，且
    T 没有直接变体成员，且

(C++11 起)
    T 的每个直接非变体非静态数据成员 M 均拥有默认成员初始化器，或如果 M 拥有类类型 X （或其数组）， X 为 const 可默认构造，
    如果 T 至少拥有一个非静态数据成员的联合体，刚好有一个变体成员拥有默认成员初始化器，
    如果 T 不是联合体，那么对于每个至少拥有一个非静态数据成员的匿名联合体成员（若存在），刚好有一个非静态数据成员拥有默认成员初始化器，且

T 的每个潜在构造的基类均为 const 可默认构造。

从不确定字节读取
使用由默认初始化任何非类类型的变量所取得的不确定的值是未定义行为（特别是，它可能是一种陷阱表示），除了下列情况：

将 unsigned char 或 std::byte (C++17 起) 类型的不确定值赋值给另一拥有（可有 cv 限定的）unsigned char 或 std::byte (C++17 起) 类型的变量（变量的值变为不确定，但该行为不是未定义）；

用 unsigned char 或 std::byte (C++17 起) 类型的不确定值初始化另一拥有（可有 cv 限定的）unsigned char 或 std::byte (C++17 起)类型的变量；

从以下场合产生的 unsigned char 或 std::byte (C++17 起) 类型的不确定值
        条件表达式的第二或第三操作数，
        逗号运算符的右操作数，
        转型或转换到（可有 cv 限定的）unsigned char 或 std::byte (C++17 起)的操作数，
        弃值表达式。

```C++
int f(bool b)
{
     int x;                // OK：x 的值不确定
     int y = x;            //  未定义行为
     unsigned char c;      // OK：c 的值不确定
     unsigned char d = c; // OK：d  的值不确定
     int e = d;            // 未定义行为
     return b ? d : 0;     // 如果  b 为 true 则行为未定义
}
```

注解
具有自动和动态存储期的非类变量的默认初始化产生具有不确定值的对象（静态和线程局部对象进行的是零初始化）。

不能默认初始化引用和 const 标量对象。

```C++
struct T1 { int mem; };

struct T2
{
     int mem;
     T2() { } // "mem" 不在初始化器列表中
};

int n; // 静态非类，进行两阶段初始化：
        // 1) 零初始化将 n 初始化为零
        // 2) 默认初始化不做任何事，令 n 保留为零

int main()
{
     int n;             // 非类，值不确定
     std::string s;     // 类，调用默认构造函数，值是 ""（空字符串）
     std::string a[2]; // 数组，默认初始化其各元素，值是 {"", ""}
//   int& r;            // 错误：引用
//   const int n;       // 错误：const 的非类
//   const T1 t1;       // 错误：const 的带隐式默认构造函数的类
     T1 t1;             // 类，调用隐式默认构造函数
     const T2 t2;       // const 类，调用用户提供的默认构造函数
                       // t2.mem 被默认初始化（为不确定值）
}
```

* * *

https://zh.cppreference.com/w/cpp/language/zero_initialization

# 零初始化

将一个对象的初始值设为零。

语法
注意零初始化在语言中没有专用语法，因此下列语法不是零初始化语法。这些是可能会进行零初始化的其他初始化的例子。

```C++
(1)
static T 对象 ;

(2)
T () ;
T t = {} ;
T {} ; (C++11 起)

(3)
CharT 数组 [ n ] = " 短序列 ";
```

解释
在下列情形进行零初始化

1.  在所有其他初始化前，对每个具有静态或线程局部 (C++11 起)存储期的，不进行常量初始化的具名变量。
2.  作为非类类型变量，和被值初始化的无构造函数的类类型的成员的值初始化序列的一部分，包括未提供初始化器的聚合体元素的值初始化。
3.  以不够长的字符串字面量初始化任何字符类型数组时，零初始化数组的剩余部分。

零初始化的效果是：
    如果 T 是标量类型，那么对象的初始值是将整数字面量 0 显式转换到 T 的值。
    如果 T 是非联合体类类型，那么：
        初始化所有填充位为零位，
        零初始化所有非静态数据成员，
        零初始化所有非虚基类子对象，并且
        如果对象不是基类子对象，那么也零初始化所有虚基类子对象。

如果 T 是联合体类型，那么：
        初始化所有填充位为零位，
        零初始化对象的首个非静态具名数据成员。

如果 T 是数组类型，那么零初始化每个元素。
    如果 T 是引用类型，那么不做任何事。

注解

如 非局部初始化 中所述，在进行任何其他初始化前，零初始化所有未被常量初始化的静态和线程局部 (C++11 起)变量。如果非类类型非局部变量的定义没有初始化器，那么默认初始化不做任何事，不修改先前零初始化的结果。

零初始化的指针是其类型的空指针值，即使空指针的值并非整型的零。

```C++
struct A
{
     int a, b, c;
};

double f[3];    // 零初始化为三个 0.0

int* p;         // 零初始化为空指针值（即使该值可能不是整型 0）

std::string s; // 零初始化为不确定值
                // 再通过 std::string 的默认构造函数默认初始化为 ""

int main(int argc, char* argv[])
{
     delete p; // 可以安全删除空指针

     static int n = argc; // 零初始化为 0，然后复制初始化为 argc
     std::cout << "n = " << n << '\n';

     A a = A(); // 效果等同于 A a{}; 或 A a = {};
     std::cout << "a = {" << a.a << ' ' << a.b << ' ' << a.c << "}\n";
}

n = 1
a = {0 0 0}
```

* * *

https://zh.cppreference.com/w/cpp/language/storage_duration

# 存储类说明符

存储类说明符是一个名字的声明语法的声明说明符序列的一部分。它与名字的作用域一同控制名字的两个独立性质：它的“存储期”和它的“链接”。

|     |     |
| --- | --- |
| auto 或 (C++11 前)无说明符 | 自动存储期。 |
| register | 自动存储期，另提示编译器将此对象置于处理器的寄存器。(弃用) (C++17 前) |
| static | 静态或线程存储期和内部链接。 |
| extern | 静态或线程存储期和外部链接。 |
| thread_local | 线程存储期。(C++11 起) |
| mutable | 不影响存储期或链接。解释见 const/volatile。 |

声明中只能出现一个存储类说明符，但 thread_local 可以与 static 或 extern 结合 (C++11 起)。

## 解释

1.  auto 说明符只能搭配在块作用域或函数形参列表中声明的对象。它指示自动存储期，即这种声明的默认情况。此关键词的含义在 C++11 有变更。 (C++11 ==前==)
    
2.  register 说明符只能搭配在块作用域或函数形参列表中声明的对象。它指示自动存储期，即这种声明的默认情况。另外，此关键词的存在可以用来提示优化器将此变量的值存储到 CPU 寄存器。此关键词已经被==弃用==。(C++17 前)
    
3.  static 说明符只能搭配（函数形参列表外的）对象声明、（块作用域外的）函数声明及匿名联合体声明。当用于声明类成员时，它会声明一个静态成员。当用于==声明对象==时，它指定==静态存储期==（除非与 thread_local 协同出现）。在命名空间作用域内声明时，它指定==内部链接==。
    
4.  extern 声明符只能搭配变量声明和函数声明（除了类成员或函数形参）。它指定==外部链接==，而且技术上不影响存储期，但它不能用来定义自动存储期的对象，故所有 extern 对象都具有静态或线程存储期。另外，使用 extern 且没有初始化器的声明不是定义。
    
5.  thread\_local 关键词只能搭配在命名空间作用域声明的对象、在块作用域声明的对象及静态数据成员。它指示对象具有线程存储期。如果对块作用域变量只应用了 thread\_local 这一个存储类说明符，那么同时也意味着应用了 static。它能与 static 或 extern 结合，以分别指定内部或外部链接（但静态数据成员始终拥有外部链接）。(C++11 起)
    

> 。。外部链接意味着符号(函数或全局变量)可以在整个程序中访问，而内部链接意味着只能在一个翻译单元中访问。

## ==存储期==

程序中的所有对象都具有下列存储期之一：

自动（automatic）存储期。这类对象的存储在外围**代码块开始时分配，并在结束时解分配**。未声明为 static、extern 或 thread_local 的所有局部对象均拥有此存储期。

静态（static）存储期。这类对象的存储在==程序开始==时分配，并在**程序结束**时解分配。这类对象**只存在一个实例**。所有在命名空间（包含全局命名空间）作用域声明的对象，加上声明带有 **static 或 extern** 的对象均拥有此存储期。有关拥有此存储期的对象的初始化的细节，见非局部变量与静态局部变量。

线程（thread）存储期。这类对象的存储在**线程开始**时分配，并在线程结束时解分配。每个线程拥有它自身的对象实例。只有声明为 thread\_local 的对象拥有此存储期。thread\_local 能与 static 或 extern 一同出现，它们用于调整链接。关于具有此存储期的对象的初始化的细节，见非局部变量和静态局部变量。 (C++11 起)

动态（dynamic）存储期。这类对象的存储是通过使用==动态内存分配==函数来按请求进行分配和解分配的。关于具有此存储期的对象的初始化的细节，见 new 表达式。

子对象和引用成员的存储期与它们的完整对象一致。

## 链接

指代对象、引用、函数、类型、模板、命名空间或值的名字，可具有链接。如果某个名字具有链接，那么它所指代的实体与另一作用域中的声明所引入的相同名字指代相同的实体。如果有变量、函数或其他实体在数个作用域声明但没有足够的链接，那么就会生成该实体的多个实例。

以下各种链接可以被识别：

### 无链接

名字只能从它所在的作用域使用。 在块作用域声明的下列任何名字均无链接：
    未显式声明为 extern 的变量（不管有没有 static 修饰符）；
    局部类和它的成员函数；
    在块作用域声明的其他名字，例如 typedef、枚举及枚举项。

未指定为拥有外部、模块 (C++20 起)或内部链接的名字同样无链接，这与它的声明所处的作用域无关。

### 内部链接

名字可从**当前翻译单元**中的所有作用域使用。 在命名空间作用域声明的下列任何名字均具有内部链接；
    声明为 static 的变量、变量模板 (C++14 起)、函数或函数模板；

未声明为 extern 且先前未声明为具有外部链接的非 volatile 非模板 (C++14 起)非 inline (C++17 起)且未被导出 (C++20 起)的 const 限定的变量（包含 constexpr） (C++11 起)；

匿名联合体的数据成员。

另外，所有在无名命名空间或无名命名空间内的命名空间中声明的名字，即使显式声明为 extern，也均拥有内部链接。 (C++11 起)

### 外部链接

名字能从**其他翻译单元**中的作用域使用。具有外部链接的变量和函数也具有语言链接，这使得以不同编程语言编写的翻译单元可以互相链接。 在命名空间作用域声明的下列任何名字均具有外部链接，除非这些名字在无名命名空间内声明或它们在具名模块声明且未被导出 (C++20 起)：

以上未列出的变量与函数（即未声明为 static 的函数、命名空间作用域内未声明为 static 的非 const 变量，和所有声明为 extern 的变量）；

枚举；
    类以及其成员函数、静态数据成员（不论是否 const）、嵌套类及枚举，及首次以类体内的 friend 声明引入的函数的名字；
    所有未列于上的模板名（即未声明为 static 的函数模板）。

任何首次在块作用域声明的下列名称拥有外部链接：
    声明为 extern 的变量名；
    函数名。

### (C++20 起) 模块链接

名字只能从同一模块单元或同一具名模块中的其他翻译单元的作用域指代。
如果在命名空间作用域声明的名字附着到具名模块，未被导出且无内部链接，那么它拥有模块链接。

## 静态局部变量

在块作用域声明且带有 static 或 thread_local (C++11 起) 说明符的变量拥有静态或线程 (C++11 起)存储期，但在控制首次经过它的声明时才会被初始化（除非它被零初始化或常量初始化，这可以在首次进入块前进行）。在其后所有的调用中，声明都会被跳过。

如果初始化抛出异常，那么不认为变量被初始化，且控制下次经过该声明时将再次尝试初始化。

如果初始化递归地进入正在初始化的变量的块，那么行为未定义。

(C++11 起) 如果多个线程试图同时初始化同一静态局部变量，那么初始化严格发生一次（类似的行为也可对任意函数以 std::call_once 来达成）。
。。。那就是说，call_once 如果内部异常， 下次 还会继续 ？
注意：此功能特性的通常实现均使用双检查锁定模式的变体，这使得对已初始化的局部静态变量检查的运行时开销减少为单次非原子的布尔比较。

块作用域静态变量的析构函数在初始化已成功的情况下在程序退出时被调用。

在相同内联函数（可以是隐式内联）的所有定义中，函数局部的静态对象均指代在一个翻译单元中定义的同一对象，只要函数拥有外部链接。

注解
位于顶层命名空间作用域（C 中的文件作用域），且是 const 而非 extern 的名字在 C 中具有外部链接，但在 C++ 中具有内部链接。
C++11 起，auto 不再是存储类说明符；它被用于指示类型推导。

在 C 中，不能取 register 变量的地址，但在 C++ 中，声明为 register 的对象与声明不带任何存储类说明符的变量在语义上没有区别。 (C++17 前)

与 C 不同，在 C++ 中不能将变量声明为 register。 (C++17 起)

从不同作用域指代的且带内部或外部链接的 thread_local 变量的名字可能指代相同或不同的实例，这取决于代码在相同还是不同的线程执行。

extern 关键词也能用来指定语言链接和显式模板实例化声明，但它在这些情况中不是存储类说明符（但当声明直接在语言链接说明中所包含时将声明当做如同它含 extern 说明符）。

关键词 mutable 在 C++ 语言的文法中是存储类说明符，尽管它并不影响存储期或链接。

thread_local 以外的存储类说明符都不能显式特化及显式实例化中使用：

```C++
#include <iostream>
#include <mutex>
#include <string>
#include <thread>
 
thread_local unsigned int rage = 1; 
std::mutex cout_mutex;
 
void increase_rage(const std::string& thread_name)
{
    ++rage; // 在锁外修改 OK；这是线程局部变量
    std::lock_guard<std::mutex> lock(cout_mutex);
    std::cout << thread_name << " 的愤怒计数：" << rage << '\n';
}
 
int main()
{
    std::thread a(increase_rage, "a"), b(increase_rage, "b");
 
    {
        std::lock_guard<std::mutex> lock(cout_mutex);
        std::cout << "main 的愤怒计数：" << rage << '\n';
    }
 
    a.join();
    b.join();
}
```

可能的输出：
a 的愤怒计数：2
main 的愤怒计数：1
b 的愤怒计数：2

==================

https://zh.cppreference.com/w/cpp/language/namespace

# 命名空间

命名空间提供了一种在大项目中避免名字冲突的方法。
在命名空间块内声明的符号被放入一个具名的作用域中，避免这些符号被误认为其他作用域中的同名符号。
多个命名空间块的名字可以相同。这些块中的所有声明在该具名作用域声明。

语法
namespace 命名空间名 { 声明序列 }                (1)
命名空间 命名空间名 的具名命名空间定义。

inline namespace 命名空间名 { 声明序列 }         (2)         (C++11 起)
命名空间 命名空间名 的内联命名空间定义。命名空间名 内的声明将在它的外围命名空间可见。

namespace { 声明序列 }                          (3)
无名命名空间定义。它的成员具有从声明点到翻译单元结尾的潜在作用域，并具有内部连接。
。。。translation unit： 编译单元; 转换单元; 翻译单位; 翻译单元

命名空间名 :: 成员名                         (4)
命名空间名（还有类名）可以出现在作用域解析运算符的左侧，作为有限定的名字查找的一部分。

using namespace 命名空间名 ;                    (5)

using 指令：从 using 指令之后到指令出现的作用域结尾为止，以对任何名字的无限定名字查找的视点来说，来自 命名空间名 的任何名字均可见，如同它在同时含有该 using 指令和 命名空间名 两者的最接近外围命名空间作用域声明一样。

using 命名空间名 :: 成员名 ;                      (6)

using 声明：令来自命名空间 命名空间名 的符号 成员名 对无限定名字查找可见，如同将它在包含该 using 声明的相同的类作用域、块作用域或命名空间之中声明一样。

namespace 名字 = 有限定命名空间 ;                  (7)
命名空间别名定义（namespace-alias-definition）：令 名字 成为另一命名空间的同义词：见命名空间别名。

namespace 命名空间名 :: 成员名 { 声明序列 }    (8)         (C++17 起)

嵌套命名空间定义：namespace A::B::C { ... } 等价于 namespace A { namespace B { namespace C { ... } } }。

namespace 命名空间名::inline 成员名 { 声明序列 } (9)         (C++20 起)

嵌套内联命名空间定义：namespace A::B::inline C { ... } 等价于 namespace A::B { inline namespace C { ... } }。inline 可以在首个以外的每个命名空间名之前出现：namespace A::inline B::C {} 等价于 namespace A { inline namespace B { namespace C {} } }。

解释
命名空间
inline(可选) namespace 属性(可选) 标识符 { 命名空间体 }

|     |     |
| --- | --- |
| inline | (C++11 起) 存在时，此命名空间是内联命名空间（见下文）。如果原初命名空间定义不使用 inline，那么不能出现在扩展命名空间定义中 |
| 属性  | (C++17 起) 任意数量的属性的序列 |
| 标识符 | 下列之一： |
|     | 先前未使用过的标识符，此时这是原初命名空间定义 |
|     | 命名空间名，此时这是扩展命名空间定义 |
|     | 被 :: 分隔的外围命名空间说明符，以 标识符 结束，此时这是嵌套命名空间定义 (C++17 起) |
| 命名空间体 | 任何种类（包含类、函数定义和嵌套命名空间等）的声明的可能为空的序列 |

命名空间定义只允许在命名空间作用域，包括全局作用域中出现。

为了重新打开一个既存的命名空间（正式而言，作为扩展命名空间定义），对用于命名空间定义中的 标识符的查找必须解析为一个命名空间名（而非命名空间别名），该名字被声明为一个外围命名空间或外围命名空间中的内联命名空间的一个成员。

命名空间体 定义了一个命名空间作用域，这会影响名字查找。

在 命名空间体（包含嵌套命名空间定义）中出现的声明所引入的所有名字均成为命名空间 标识符 的成员，无论此命名空间定义是原初命名空间定义（引入 标识符 者）还是扩展命名空间定义（“重打开”已定义命名空间者）。

一个在命名空间体内声明的命名空间成员，可以在该命名空间体外部用显式限定进行定义或再声明：

```C++
namespace Q
{
    namespace V   // V 是 Q 的成员，且完全在 Q 内定义
    { // namespace Q::V { // C++17 起可以用来替代以上几行
        class C { void m(); }; // C 是 V 的成员且完全在 V 内定义，C::m 只是被声明
        void f(); // f 是 V 的成员，但只在此声明
    }

    void V::f() // 在 V 外对 V 的成员 f 的定义
                // f 的外围命名空间仍是全局命名空间、Q 与 Q::V
    {
        extern void h(); // 这声明了 ::Q::V::h
    }

    void V::C::m() // 在命名空间外（及类外）对 V::C::m 的定义
                   // 外围命名空间是全局命名空间、Q 与 Q::V
    {}
}
```

命名空间外的定义和再声明只能出现在
    声明点后，
    命名空间作用域中，且
    作用域的命名空间（包括全局命名空间）包围了原命名空间。

它们也应使用限定标识语法。         (C++14 起)

```C++
namespace Q
{
    namespace V    // V 的原初命名空间定义
    {
        void f();  // Q::V::f 的声明
    }

    void V::f() {} // OK
    void V::g() {} // 错误：g() 目前还不是 V 的成员

    namespace V    // V 的扩展命名空间定义
    {
        void g();  // Q::V::g 的声明
    }
}

namespace R           // 不是 Q 的外围命名空间
{
    void Q::V::g() {} // 错误：不能在 R 内定义 Q::V::g
}

void Q::V::g() {}     // OK：全局命名空间包围 Q
```

在非局部类 X 中由友元声明所引入的名字会成为 X 的最内层外围命名空间的成员，但它们对于通常的名字查找（无限定或有限定）不可见，除非在命名空间作用域中类定义前后提供与之匹配的声明。这种名字可以通过实参依赖查找找到，其中同时考虑命名空间和类。

在确定名字是否与先前声明的名字冲突时，这种友元声明只考虑最内层的外围命名空间。

```C++
void h(int);

namespace A
{
    class X
    {
        friend void f(X);       // A::f 是友元

class Y
        {
            friend void g();    // A::g 是友元
            friend void h(int); // A::h 是友元，与 ::h 不冲突
        };
    };
    // A::f、A::g 与 A::h 在命名空间作用域不可见
    // 虽然它们是命名空间 A 的成员

X x;
    void g()   // A::g 的定义
    {
        f(x);  // A::X::f 通过实参依赖查找找到
    }

void f(X) {}   // A::f 的定义
    void h(int) {} // A::h 的定义
    // A::f、A::g 与 A::h 现在在命名空间作用域可见
    // 而且它们也是 A::X 与 A::X::Y 的友元
}
```

## C++11, 内联命名空间

内联命名空间是在它的原初命名空间定义中使用了**可选的**关键词 inline 的命名空间。

在许多情况下（在下方列出），内联命名空间的成员都被当做如同它们是外围命名空间的成员一样。这项性质是传递性的：如果命名空间 N 包含内联命名空间 M，且它又继而包含内联命名空间 O，那么 O 的成员能按**如同它们是 M 或 N 的成员一样使用。**

在外围命名空间中，**隐含地**插入了一条指名**内联命名空间的 using 指令**（与无名命名空间的隐式 using 指令类似）

在实参依赖查找中，当一个命名空间被添加到关联命名空间集合时，它的内联命名空间也会一起被添加，且当一个内联命名空间被添加到关联命名空间列表时，它的外围命名空间也会一起被添加。

内联命名空间的每个成员，都能按照如同它是外围命名空间的成员一样，进行部分特化、显式实例化或显式特化。

。。这个不清楚，特化和命名空间的关系 是什么？ 是指 可以直接使用 外部的命名空间中的 类 进行特化，不需要显式 指定 命名空间？ 这个是 name lookup 的功能吧。

检验外围命名空间的有限定名字查找将包含来自它的各个内联命名空间的名称，即使同一名称已在外围命名空间存在。

```C++
// C++14 中，std::literals 和它的成员命名空间是内联的
{

using namespace std::string_literals; // 令来自 std::literals::string_literals 的 operator""s 可见

auto str = "abc"s;
}

{

using namespace std::literals; // 令 std::literals::string_literals::operator""s 与 std::literals::chrono_literals::operator""s 均可见

。。字面量操作符， 6 啊。

auto str = "abc"s;
    auto min = 60s;
}

{
    using std::operator""s; // 令 std::literals::string_literals::operator""s 与
                            // std::literals::chrono_literals::operator""s 均可见
    auto str = "abc"s;
    auto min = 60s;
}
```

注意：上述关于特化的规则允许建立库版本：库模板的不同实现可以在不同的内联命名空间定义，同时仍然允许用户通过主模板的显式特化来扩充父命名空间。

## 无名命名空间

无名命名空间定义是具有下列形式的命名空间定义。
inline(可选) namespace 属性(可选) { 命名空间体 }

|     |     |
| --- | --- |
| inline | (C++11 起) 存在时，此命名空间是内联命名空间 |
| 属性  | (C++17 起) 任意数量的属性的序列 |

此定义被当做一个拥有独有名字的命名空间定义，与当前作用域中指名此无名命名空间的一条 using 指令 （注：隐式添加的 using 指令使有限定查找和无限定查找可以发现该命名空间，但实参依赖查找不在此列）。

```C++
namespace
{
    int i; // 定义 ::(独有)::i
}

void f()
{
    i++;   // 自增 ::(独有)::i
}

namespace A
{
    namespace
    {
        int i;        // A::(独有)::i
        int j;        // A::(独有)::j
    }

    void g() { i++; } // A::(独有)::i++
}

using namespace A; // 从 A 引入所有名称到全局命名空间

void h()
{
    i++;    // 错误：::(独有)::i 与 ::A::(独有)::i 均在作用域中
    A::i++; // OK：自增 A::(独有)::i
    j++;    // OK：自增 A::(独有)::j
}
```

(C++11 前)
虽然无名命名空间中的名字可以声明为具有外部连接，但从其他翻译单元无法访问它们，因为它的命名空间名是独有的。

(C++11 起)
无名命名空间以及所有直接或间接在无名命名空间内声明的命名空间都具有内部连接，这表示在无名命名空间内声明的所有名字都具有内部连接。

## using 声明

引入在别处定义的名称到此 using 声明出现的声明性区域
using typename(可选) 嵌套名说明符 无限定标识 ;                 (C++17 前)
using 声明符列表 ;                 (C++17 起)
。。这里没有 namespace。

|     |     |
| --- | --- |
| typename | 当 using 声明向类模板引入基类的成员类型时，关键词 typename 可以在必要时用来解析待决名 |
| 嵌套名说明符 | 名字与作用域解析运算符 :: 的序列，以作用域解析运算符结束。单个 :: 代表全局命名空间。 |
| 无限定标识 | 标识表达式 |
| 声明符列表 | 一或多个形式为 typename(可选) 嵌套名说明符 无限定标识 的声明符的逗号分隔列表。声明符可以后随省略号以指示包展开，但这种形式只在派生类定义中有意义 |

using 声明可以用来将命名空间的成员引入到其他命名空间和块作用域中，或将基类的成员引入到派生类定义中，或将枚举项引入命名空间、块或类作用域中    (C++20 起)。

由 using 声明引入到命名空间作用域的名字，可以同任何其他名字一样使用，包含从其他作用域进行的有限定查找：

```C++
void f();

namespace A
{
    void g();
}

namespace X
{
    using ::f;        // 全局 f 现在作为 ::X::f 可见
    using A::g;       // A::g 现在作为 ::X::g 可见
    using A::g, A::g; //（C++17）OK：命名空间作用域允许双重声明
}

void h()
{
    X::f(); // 调用 ::f
    X::g(); // 调用 A::g
}
```

在用 using 声明从命名空间采取成员后，如果该命名空间被扩充并引入了同名的额外声明，那么这些额外声明不会通过该 using 声明变为可见（与 using 指令相反）。一个例外是 using 声明指名类模板时：后面引入的部分特化实际上是可见的，因为它的查找是通过主模板达成的。

```C++
namespace A
{
    void f(int);
}
using A::f; // ::f 现在是 A::f(int) 的同义词

namespace A       // 命名空间扩展
{
    void f(char); // 不更改 ::f 的含义
}

void foo()
{
    f('a'); // 调用 f(int)，即使 f(char) 存在。
}

void bar()
{
    using A::f; // 此 f 是 A::f(int) 与 A::f(char) 的同义词
    f('a');     // 调用 f(char)
}
```

using 声明不能指名模板标识或命名空间，或有作用域的枚举项 (C++20 前)。using 声明中的每个声明符引入一个且只有一个名字，例如枚举 的 using 声明不引入它的任何枚举项。

针对相同名字的常规声明的所有制约，隐藏和重载规则，均适用于 using 声明：

```C++
namespace A
{
    int x;
}

namespace B
{
    int i;
    struct g {};
    struct x {};

    void f(int);
    void f(double);
    void g(char); // OK：函数名 g 隐藏类 g
}

void func()
{
    int i;
    using B::i;   // 错误：i 声明了两次

    void f(char);
    using B::f;   // OK：f(char)、f(int)、f(double) 是重载
    f(3.5);       // 调用 B::f(double)

    using B::g;
    g('a');       // 调用 B::g(char)
    struct g g1;  // 声明 g1 拥有类型 struct B::g

    using B::x;
    using A::x;   // OK ：隐藏 B::x
    x = 99;       // 赋值给 A::x
    struct x x1;  // 声明 x1 拥有类型 struct B::x
}
```

如果 using 声明引入了一个函数，那么声明一个拥有相同名字和形参列表的函数是非良构的（除非是同一函数的声明）。如果 using 声明引入了一个函数模板，那么声明拥有相同名字、形参类型列表、返回类型及模板形参列表的函数模板是非良构的。两个 using 声明可以引入拥有相同名字和形参列表的函数，但试图调用该函数会导致程序非良构。

```C++
namespace B
{
    void f(int);
    void f(double);
}

namespace C
{
    void f(int);
    void f(double);
    void f(char);
}

void h()
{
    using B::f;  // 引入 B::f(int)、B::f(double)
    using C::f;  // 引入 C::f(int)、C::f(double) 及 C::f(char)
    f('h');      // 调用 C::f(char)
    f(1);        // 错误：B::f(int) 还是 C::f(int)？
    void f(int); // 错误：f(int) 与 C::f(int) 及 B::f(int) 冲突
}
```

如果某个实体被声明，但未在某内层命名空间中被定义，然后在外层命名空间中通过 using 声明予以声明，然后在外层命名空间中再出现拥有相同的非限定名的定义，那么该定义是外层命名空间的成员，且与 using 声明冲突：

```C++
namespace X
{
    namespace M
    {
        void g(); // 声明，但不定义 X::M::g()
    }
    using M::g;

    void g();   // 错误：试图声明与 X::M::g() 冲突的 X::g
}
```

更一般地，在任何命名空间作用域中出现并用无限定标识符引入名字的一个声明始终向它所在的命名空间中引入一个成员，而并非向任何其他命名空间引入。例外情况是对在内联命名空间定义的主模板进行的显式实例化和显式特化：因为它们不引入新名字，它们在外围命名空间中可以使用无限定标识。

using 指令
using 指令是拥有下列语法的块声明：
属性(可选) using namespace 嵌套名说明符(可选) 命名空间名 ;         (1)

|     |     |
| --- | --- |
| 属性  | (C++11 起) 应用到此 using 指令的任意数量的属性 |
| 嵌套名说明符 | 名字与作用域解析运算符 :: 的序列，以作用域解析运算符结束。单个 :: 代表全局命名空间。查找此序列中的名字时，查找只考虑命名空间声明 |
| 命名空间名 | 命名空间名。查找此名时，查找只考虑命名空间声明 |

using 指令只能在命名空间作用域和块作用域中出现。从某个 using 指令之后到该指令出现的作用域结尾为止，以任何名字的无限定名字查找的角度，来自 命名空间名 的每个名字均可见，如同它在同时包含该 using 指令和 命名空间名 两者的最接近外围命名空间声明一样。

using 指令不向它所出现的声明性区域添加任何名字（与 using 声明不同），因而并不妨碍再声明相同的名字。

using 指令对于无限定查找具有传递性：如果作用域包含指名 命名空间名 的 using 指令，而它自身包含对某 命名空间名2 的 using 指令，那么效果如同第二个命名空间中的 using 指令出现在第一个之中一样。这些传递性命名空间的出现顺序并不影响名字查找。

```C++
namespace A
{
    int i;
}

namespace B
{
    int i;
    int j;
    namespace C
    {
        namespace D
        {
            using namespace A; // 将所有来自 A 的名称注入全局命名空间

            int j;
            int k;
            int a = i;         // i 是 B::i，因为 B::i 隐藏 A::i
        }

        using namespace D; // 注入来自 D 的名称到 C
                           // 注入来自 A 的名称到全局命名空间

        int k = 89; // 声明与用 using 引入者等同的名称 OK
        int l = k;  // 歧义：C::k 还是 D::k
        int m = i;  // OK：B::i 隐藏 A::i
        int n = j;  // OK：D::j 隐藏 B::j
    }
}
```

在使用 using 指令指名某命名空间后，如果该命名空间被扩充并向它添加了额外的成员和/或 using 指令，那么这些额外成员和额外的命名空间通过该 using 指令可见（与 using 声明相反）

```C++
namespace D
{
    int d1;
    void f(char);
}
using namespace D; // 引入 D::d1、D::f、D::d2、D::f，
                   // E::e 及 E::f 到全局命名空间！

int d1;            // OK：声明时与 D::d1 不冲突

namespace E
{
    int e;
    void f(int);
}

namespace D            // 命名空间扩展
{
    int d2;
    using namespace E; // 传递性 using 指令
    void f(int);
}

void f()
{
    d1++;    // 错误：歧义：::d1 还是 D::d1？
    ::d1++;  // OK
    D::d1++; // OK
    d2++;    // OK，d2 是 D::d2

    e++;     // OK：因为传递性 using，所以 e 是 E::e

    f(1);    // 错误：歧义：D::f(int) 还是 E::f(int)？
    f('a');  // OK：f(char) 只能是 D::f(char)
}
```

注解

在任何命名空间作用域中的 using 指令 using namespace std;将命名空间 std 中的所有名字都引入到全局命名空间中（因为全局命名空间是同时包含 std 和任何用户声明命名空间的最近命名空间），这可能导致不合预期的名字冲突。通常认为，在头文件的文件作用域中采用它或其他的 using 指令是不良的实践（SF.7：不要在头文件的全局作用域中写 using namespace）。

示例
此例展示如何用命名空间创建已在 std 命名空间命名的类。

```C++
#include <vector>
 
namespace vec
{
    template<typename T>
    class vector
    {
        // ...
    };
}
 
int main()
{
    std::vector<int> v1; // 标准 vector。
    vec::vector<int> v2; // 用户定义 vector。
 
    v1 = v2; // 错误：v1 和 v2 是不同类型的对象。
 
    {
        using namespace std;
        vector<int> v3; // 同 std::vector
        v1 = v3; // OK
    }
 
    {
        using vec::vector;
        vector<int> v4; // 同 vec::vector
        v2 = v4; // OK
    }
 
    return 0;
}
```

* * *

命名空间别名

命名空间别名允许程序员定义命名空间的另一个名字。

它们常用作长的或嵌套过深的命名空间的简便使用方式。
语法
namespace 别名 = 命名空间名;         (1)
namespace 别名 = ::命名空间名;         (2)
namespace 别名 = 嵌套名::命名空间名;         (3)

==================

https://zh.cppreference.com/w/cpp/language/attributes

# 属性说明符序列(C++11 起)

==================
[https://zh.cppreference.com/w/cpp/language/rule\_of\_three](https://zh.cppreference.com/w/cpp/language/rule_of_three)

# ==三/五/零之法则==

## 三之法则

如果某个类需要==用户定义的**析构**函数、用户定义的**复制构造**函数或用户定义的**复制赋值**运算符==，那么它几乎肯定需要全部三者。

因为 C++ 在各种场合（按值传递/返回、操纵容器等）对对象进行复制和复制赋值时会在这些特殊成员函数可以访问的情况下调用它们，而且如果用户没有定义他们，那么编译器就会隐式定义。

如果类对某种资源进行管理，而资源句柄是非类类型的对象（裸指针、POSIX 文件描述符等），那么这些隐式定义的成员函数通常都不正确，该类的析构函数不会做任何事，而复制构造函数/复制赋值运算符会进行“浅复制”（复制句柄的值，而不复制底层资源）。

```C++
class rule_of_three
{
    char* cstring; // 以裸指针为动态分配内存的句柄
 
    rule_of_three(const char* s, std::size_t n) // 避免重复计数
    : cstring(new char[n]) // 分配
    {
        std::memcpy(cstring, s, n); // 填充
    }
public:
    rule_of_three(const char* s = "")
    : rule_of_three(s, std::strlen(s) + 1) {}
 
    ~rule_of_three() // I. 析构函数
    {
        delete[] cstring; // 解分配
    }
 
    rule_of_three(const rule_of_three& other) // II. 复制构造函数
    : rule_of_three(other.cstring) {}
 
    rule_of_three& operator=(const rule_of_three& other) // III. 复制赋值
    {
        if (this == &other)
            return *this;
 
        std::size_t n{std::strlen(other.cstring) + 1};
        char* new_cstring = new char[n];            // 分配
        std::memcpy(new_cstring, other.cstring, n); // 填充
        delete[] cstring;                           // 解分配
 
        cstring = new_cstring;
        return *this;
    }
 
    operator const char *() const // 访问器
    {
        return cstring;
    }
};
```

。。？ 这个delete\[\] cstring 是什么？ 没有 无参构造器，只有一个 private 的 有参构造器。
。。这种情况下，这个 复制赋值， 构造器是调用的 什么的？ 调用  复制构造器？

Person p;
Person p1 = p;    //  这句执行完之后会生成一个新的实例，所以调用的是拷贝构造函数
Person p2;
p2 = p;           // 这句不会生成新的实例，所以调用的是复制赋值运算符
f(p2);            // 这句执行完之后会生成一个新的实例f，所以调用的是拷贝构造函数

p2=F1 ();        // 这条语句拷贝构造函数和赋值运算符都调用了。函数f1以值的方式返回一个Person对象，在返回时会调用拷贝构造函数创建一个临时对象tmp作为返回值；返回后调用赋值运算符将临时对象tmp赋值给p2.

。。。也是。。如果 原本没有值，那么就走 复制构造，而不是 复制赋值。

通过可复制句柄来管理不可复制资源的类，可能必须将它的复制赋值和复制构造函数声明为私有的并且不提供它们的定义，或将它们定义为弃置的。这是三之法则的另一种应用：只删除其一却任由其他特殊成员被隐式定义很可能会导致错误。

## 五之法则

因为用户定义的==析构==函数、==复制构造==函数或==复制赋值==运算符的存在会阻止==移动构造==函数和==移动赋值==运算符的隐式定义，所以任何想要移动语义的类必须声明==全部五个==特殊成员函数：
。。会直接走 复制构造，赋值。

```C++
class rule_of_five
{
    char* cstring; // 以裸指针为动态分配内存的句柄
public:
    rule_of_five(const char* s = "") : cstring(nullptr)
    { 
        if (s)
        {
            std::size_t n = std::strlen(s) + 1;
            cstring = new char[n];      // 分配
            std::memcpy(cstring, s, n); // 填充
        } 
    }
 
    ~rule_of_five()
    {
        delete[] cstring; // 解分配
    }
 
    rule_of_five(const rule_of_five& other) // 复制构造函数
    : rule_of_five(other.cstring) {}
 
    rule_of_five(rule_of_five&& other) noexcept // 移动构造函数
    : cstring(std::exchange(other.cstring, nullptr)) {}
 
    rule_of_five& operator=(const rule_of_five& other) // 复制赋值
    {
        return *this = rule_of_five(other);
    }
 
    rule_of_five& operator=(rule_of_five&& other) noexcept // 移动赋值
    {
        std::swap(cstring, other.cstring);
        return *this;
    }
 
// 另一种方法是用以下函数替代两个赋值运算符：
//  rule_of_five& operator=(rule_of_five other) noexcept
//  {
//      std::swap(cstring, other.cstring);
//      return *this;
//  }
};
```

与三之法则不同的是，不提供移动构造函数和移动赋值运算符通常不是错误，但会导致失去优化机会。

## 零之法则

有自定义析构函数、复制/移动构造函数或复制/移动赋值运算符的类应该专门处理==所有权==（这遵循单一责任原则）。其他类都不应该拥有自定义的析构函数、复制/移动构造函数或复制/移动赋值运算符

这条法则也在 C++ 核心指南（C++ Core Guidelines）中出现—— C.20：一旦可以避免定义默认操作就应当施行

```C++
class rule_of_zero
{
    std::string cppstring;
public:
    rule_of_zero(const std::string& arg) : cppstring(arg) {}
};
```

当有意将某个基类用于多态用途时，可能需要将它的析构函数声明为公开的虚函数。由于这会阻拦隐式移动（并弃用隐式复制）的生成，因而必须将各特殊成员函数声明为预置的

```C++
class base_of_five_defaults
{
public:
    base_of_five_defaults(const base_of_five_defaults&) = default;
    base_of_five_defaults(base_of_five_defaults&&) = default;
    base_of_five_defaults& operator=(const base_of_five_defaults&) = default;
    base_of_five_defaults& operator=(base_of_five_defaults&&) = default;
    virtual ~base_of_five_defaults() = default;
};
```

然而这使得类有可能被切片，这是多态类经常把复制定义为弃置的原因（见 C++ 核心指南中的 C.67：多态类应该抑制复制操作），这带来了下列的五之法则的通用说法：

C.21：如果有任何默认操作被定义或 =delete，那么应当对它们全部进行定义或 =delete

==================

https://zh.cppreference.com/w/cpp/language/Zero-overhead_principle

# 零开销原则

零开销原则是一个 C++ 设计原则，所说的是：
    你不需要为你没有用到的（特性）付出。
    你所用的正与你所能合理手写的效率相同。

总而言之，这表示某特性引发的开销如果，不论在时间还是空间方面，会大于程序员自己实现该特性产生的开销，那么该特性不应该添加到 C++。

语言中仅有的两个不遵循零开销原则的特性是 运行时类型鉴别 与 异常，因此大多数编译器提供关闭它们的开关。

==================
https://zh.cppreference.com/w/cpp/language/pimpl

# PImpl

“指向实现的指针”或“pImpl”是一种 C++ 编程技巧，它将类的实现细节从对象表示中移除，放到一个分离的类中，并以一个不透明的指针进行访问：

```C++
// ---------------
// 接口（widget.h）
class widget
{
    // 公开成员
private:
    struct impl; // 实现类的前置声明
    // 一种实现示例：见下文中的其他设计选项和权衡
    std::experimental::propagate_const< // 转发 const 的指针包装器
        std::unique_ptr<                // 唯一所有权的不透明指针
            impl>> pImpl;               // 指向前置声明的实现类
};

// -----------------
// 实现（widget.cpp）
struct widget::impl
{
   // 实现细节
};
```

此技巧用于 ==构造拥有稳定 ABI 的 C++ 库接口，及减少编译时依赖==。

解释

因为类的私有数据成员参与其对象表示，影响大小和布局，也因为类的私有成员函数参与重载决议（这会在成员访问检查之前发生），因此对实现细节的任何更改都要求该类的所有用户重新编译。

pImpl 打破了这种编译依赖；实现的改动不会导致重编译。结果是，如果某个库在其 ABI 中使用 pImpl，那么这个库的新版本可以更改实现，并且与旧版本保持 ABI 兼容。

。。abstract binary interface

得失权衡
pImpl 技法的替代方案是：
    内联实现：私有成员和公开成员是同一类的成员
    纯虚类（OOP 工厂）：用户获得到某个轻量级或纯虚的基类的唯一指针，实现细节则处于覆盖其虚成员函数的派生类中。

编译防火墙

简单情况下，pImpl 和工厂方法都会打破实现和类接口的用户之间的编译时依赖。工厂方法创建对虚表的一次隐藏依赖，故而对虚函数进行重排序、添加或移除都会打破 ABI。pImpl 方法则没有隐藏的依赖，然而如果实现类是类模板特化，那么就会丧失编译防火墙的优势：接口的用户必须观测到整个模板定义，以实例化正确的特化。这种情况下的一种常见的设计方案是，以避免参数化的方式对实现进行重构，这是《C++ 核心指南》的 T.61 不要过分参数化成员 和 T.84 使用非模板核心实现以提供 ABI 稳定的接口 的另一种使用情况。

举例来说，以下的类模板在其私有成员或 push_back 函数体内并未使用类型 T：

```C++
template<class T>
class ptr_vector
{
    void **beg, **end, **cap;
public:
    void push_back(T* p)
    {
        if (end == cap)
            reallocate(end - beg + 1);
        *end++ = p;
    }
};
```

因此，能按原样把私有成员传递给实现，而且 push_back 可以转发到同样未在接口使用 T 的实现：

```C++
// ---------------------
// 头文件（ptr_vector.h）
class ptr_vector_base
{
    struct impl; // 不依赖 T
    std::unique_ptr<impl> pImpl;
protected:
    void push_back_fwd(void*);
    ... // 见特殊成员函数的实现部分
};
 
template<class T>
class ptr_vector : private ptr_vector_base
{
public:
    void push_back(T* p) { push_back_fwd(p); }
};
 
// -----------------------
// 源文件（ptr_vector.cpp）
struct ptr_vector_base::impl
{
    void **beg, **end, **cap;
    void push_back(void* p)
    {
        if (end == cap)
            reallocate(end - beg + 1);
        *end++ = p;
    }
    void reallocate(size_t sz) { ... }
};
 
void ptr_vector_base::push_back_fwd(void* p) { pImpl->push_back(p); }
ptr_vector_base::ptr_vector_base() : pImpl(std::make_unique<impl>()) {}
```

## 运行时开销

### 访问开销：

在 pImpl 中每次对私有成员函数的调用都会通过一个指针间接进行。私有成员对公开成员的每次访问也都会通过另一个指针间接进行。两个访问都跨翻译单元边界，从而只能被链接时优化优化掉。注意 OO 工厂对公开数据和实现细节的访问都要求跨翻译单元间接进行，而且由于虚派发存在，链接时优化器进行优化的机会更少。

### 空间开销：

pImpl 添加一个指针到公开组分，并且如果有任何私有成员需要访问公开成员，那么要么添加另一个指针到实现组分，要么每次调用要求它的私有成员时作为参数传递。如果支持有状态自定义分配器，那么必须一同存储分配器实例。

### 生存期管理开销：

pImpl（还有 OO 工厂）将实现对象放在堆上，这在构造与销毁时强加了显著的运行时开销。这可以部分地由自定义分配器所弥补，因为 pImpl（但非 OO 工厂）的大小在编译时是已知的。

另一方面，pImpl 类是对移动友好的；把大型的类重构为可以移动的 pImpl，可以提升对保有这些对象的容器进行操作的算法性能，即便可以移动的 pImpl 也具有额外的运行时开销：任何在被移动对象上容许使用并需要访问私有实现的公开成员函数必须进行空指针检查。

## 维护开销

pImpl 的使用要求专用的翻译单元（只有头文件的库无法使用 pImpl），引入一个额外类，一组转发函数，且当使用分配器时，会暴露公开接口所使用的分配器的细节。

因为虚成员是 pImpl 的接口组分的一部分，所以模拟 pImpl 意味着单独模拟接口组分。可以测试的 pImpl 典型情况下被设计为允许通过可用接口达成完整的测试覆盖。

实现
由于接口类型的对象控制实现类型对象的生存期，指向实现的指针通常是 std::unique_ptr。

因为 std::unique_ptr 要求被指向类型在任何实例化删除器的语境中均为完整类型，所以特殊成员函数必须由用户声明，并在实现文件（实现类完整处）中类外定义。

因为当 const 成员通过非 const 成员指针调用函数时会调用实现函数的非 const 重载，所以该指针必须包装于 std::experimental::propagate_const 或等价物中。

将所有私有数据成员和所有私有非虚成员函数放到实现类中。将所有公开、受保护和虚的成员留在接口类中（对替代方案的讨论见 GOTW #100）。

如果有任何私有成员需要访问公开或受保护的成员，那么可以将指向接口的引用或指针作为参数传递给私有函数。另外，也可以将回溯引用作为实现类的一部分维持。

如果有意使用非默认分配器以支持实现对象的分配，那么可以利用任何常用的具分配器模式，包括默认为 std::allocator 的分配器模板形参，以及类型为 std::pmr::memory_resource* 的构造函数实参。

示例

演示有传播 const 的 pImpl，带有作为参数传递的回溯引用，不具分配器，并启用不带运行时检查的移动

```C++
#include <iostream>
#include <memory>
#include <experimental/propagate_const>
 
// ---------------
// 接口（widget.h）
class widget
{
    class impl;
    std::experimental::propagate_const<std::unique_ptr<impl>> pImpl;
public:
    void draw() const; // 公开 API，将被转发给实现
    void draw();
    bool shown() const { return true; } // 公开 API，实现必须调用它
 
    widget(int);
    ~widget(); // 在实现文件中定义，其中 impl 是完整类型
    widget(widget&&); // 在实现文件中定义
                      // 注意：在被移动的对象上调用 draw() 是未定义行为
    widget(const widget&) = delete;
    widget& operator=(widget&&); // 在实现文件中定义
    widget& operator=(const widget&) = delete;
};
 
// -----------------
// 实现（widget.cpp）
class widget::impl
{
    int n; // 私有数据
public:
    void draw(const widget& w) const
    {
        if(w.shown()) // 对公开成员函数的此调用要求回溯引用
            std::cout << "正在绘制 const 组件 " << n << '\n';
    }
 
    void draw(const widget& w)
    {
        if(w.shown())
            std::cout << "正在绘制非 const 组件 " << n << '\n';
    }
 
    impl(int n) : n(n) {}
};
 
void widget::draw() const { pImpl->draw(*this); }
void widget::draw() { pImpl->draw(*this); }
widget::widget(int n) : pImpl{std::make_unique<impl>(n)} {}
widget::widget(widget&&) = default;
widget::~widget() = default;
widget& widget::operator=(widget&&) = default;
 
// ---------------
// 用户（main.cpp）
int main()
{
    widget w(7);
    const widget w2(8);
    w.draw();
    w2.draw();
}
```

输出：

正在绘制非 const 组件 7
正在绘制 const 组件 8

* * *

https://zh.cppreference.com/w/cpp/language/raii

# RAII

资源获取即初始化（Resource Acquisition Is Initialization）

它将必须在使用前请求的资源（==分配的堆内存、执行线程、打开的套接字、打开的文件、锁定的互斥体、磁盘空间、数据库连接等——任何存在受限供给中的事物==）的生命周期绑定与一个对象的生存期相绑定。

RAII 保证资源能够用于任何会访问该对象的函数（资源可用性是一种类不变式，这会消除冗余的运行时测试）。
它也保证对象在自己生存期结束时会以获取顺序的逆序释放它控制的所有资源。
类似地，如果资源获取失败（构造函数以异常退出），那么已经构造完成的对象和基类子对象所获取的所有资源就会以初始化顺序的逆序释放。

这有效地利用了语言特性（对象生存期、退出作用域、初始化顺序以及栈回溯）以消除内存泄漏并保证异常安全。根据 RAII 对象的生存期在退出作用域时结束这一基本状况，此技术也被称为作用域界定的资源管理（Scope-Bound Resource Management，SBRM）。

RAII 可以总结如下:
    将每个资源封装入一个类，其中：
        构造函数请求资源，并建立所有类不变式，或在它无法完成时抛出异常，
        析构函数释放资源并且决不会抛出异常；

在使用资源时始终通过 RAII 类的满足以下要求的实例：
        自身拥有自动存储期或临时生存期，或
        具有与自动或临时对象的生存期绑定的生存期

移动语义使得在对象间，跨作用域，以及在线程内外安全地移动所有权，而同时维护资源安全成为可能。   (C++11 起)

拥有 open()/close()、lock()/unlock()，或 init()/copyFrom()/destroy() 成员函数的类是典型的非 RAII 类的例子：

```C++
std::mutex m;

void bad()
{
    m.lock();                    // 请求互斥体
    f();                         // 如果 f() 抛出异常，那么互斥体永远不会被释放
    if(!everything_ok()) return; // 提早返回，互斥体永远不会被释放
    m.unlock();                  // 只有 bad() 抵达此语句，互斥体才会被释放
}

void good()
{
    std::lock_guard[std::mutex](std::mutex) lk(m); // RAII类：互斥体的请求即是初始化
    f();                               // 如果 f() 抛出异常，那么就会释放互斥体
    if(!everything_ok()) return;       // 提早返回也会释放互斥体
}                                      // 如果 good() 正常返回，那么就会释放互斥体
```

标准库

C++ 标准库遵循 RAII 管理其自身的资源：std::string、std::vector、std::jthread (C++20 起)，以及很多其他在构造函数中获取资源（错误时抛出异常），并在析构函数中将其释放（决不抛出）而不要求显式清理的类。

。。vector 也是 RAII？ 那出作用域 会调用析构？

(C++11 起) 另外，标准库提供几种 RAII 包装器以管理用户提供的资源：
    std::unique\_ptr 及 std::shared\_ptr 用于管理动态分配的内存，或以用户提供的删除器管理任何以普通指针表示的资源；
    std::lock\_guard、std::unique\_lock、std::shared_lock 用于管理互斥体。

注解
RAII 不适用于不会在使用前请求的资源：CPU 时间、核心，以及缓存容量、熵池容量、网络带宽、电力消费、栈内存等。

==================

https://zh.cppreference.com/w/cpp/language/move_constructor

# 隐式声明的移动构造函数

如果不对类类型（struct、class 或 union）提供任何用户定义的移动构造函数，且满足下列所有条件：
    没有用户声明的复制构造函数；
    没有用户声明的复制赋值运算符；
    没有用户声明的移动赋值运算符；
    没有用户声明的析构函数；
那么编译器将声明一个移动构造函数作为这个类的非 explicit 的 inline public 成员，签名为 T::T(T&&)。

==================

==================

==================

https://zh.cppreference.com/w/cpp/language/adl

# ==实参依赖查找,ADL==

argument-dependent lookup

又称 ADL 或 Koenig 查找， 是一组 对 函数调用表达式 (包括对 重载运算符 的隐式函数调用) 中 的 无限定的 函数名 进行查找的 规则。

在通常 无限定 名字查找 所考虑的 作用域 和 命名空间 之外， 还会在 它的 各个实参 的 命名空间 中 查找 这些函数。

实参依赖查找 使得 使用 定义于 不同命名空间 的 运算符 称为 可能。

```C++
#include <iostream>
int main()
{
    std::cout << "测试\n"; // 全局命名空间中没有 operator<<，但 ADL 检验 std 命名空间，
                           // 因为左实参在 std 命名空间中
                           // 并找到 std::operator<<(std::ostream&, const char*)
    operator<<(std::cout, "测试\n"); // 同上，用函数调用记法
 
    // 然而，
    std::cout << endl; // 错误：'endl' 未在此命名空间中声明。
                       // 这不是对 endl() 的函数调用，所以不适用 ADL
 
    endl(std::cout); // OK：这是函数调用：ADL 检验 std 命名空间，
                     // 因为 endl 的实参在 std 中，并找到了 std::endl
 
    (endl)(std::cout); // 错误：'endl' 未在此命名空间声明。
                       // 子表达式 (endl) 不是函数调用表达式
}
```

细节
首先，如果 通常的 无限定查找(。。就是上面的 名字查找) 所生成 的集合 含有 下列 任何内容，那么 不考虑 实参依赖查找：

1.  类成员的声明
2.  块作用域的 (非 using 声明的) 函数声明
3.  任何非函数 或 函数模板的声明 ( 例如 函数对象 或 另一变量， 它的 名字与 正在查找 的函数名 冲突)

否则，对于 每个函数 调用 表达式中的 实参， 检验它的 类型，以 确定 它 将向 查找 所添加的 命名空间 与 类的 关联集。

1.  对于基础类型的 实参，命名空间 与 类的 关联集 是 空集。
    
2.  对于类类型 (含 联合体) 的实参，集合由 以下组成：
    
    1.  该类自身
    2.  它所有的 直接 和 间接 基类
    3.  如果 该类 是 另一类的 成员，该外围类
    4.  添加到 集合 的 各个类 的 最内层 外围命名空间
3.  对于 类型 是 类模板 的 特化 的实参， 在上述 关于  类的 规则外， 还 检验 以下类型，并将 与它 关联的类 与 命名空间 添加到 集合中：
    
    1.  为 各类型模板 形参 提供 的 所有 模板实参的 类型 (跳过 非类型 模板形参 和 模板模板形参)
    2.  以任何模板模板实参 为其中成员的 命名空间
    3.  以 任何 模板模板实参 为 其中成员的 类 (如果 它们是 类成员模板)
4.  对于 任何枚举类型的 实参，向集合中 添加 该枚举类型 的声明 的 最内层 外围 命名空间
    
5.  对于 T 的指针 或 指向 T 的数组 的 指针 类型的 实参， 检查 类型T 并向 集合 添加它的类 与命名空间 的 关联集合。
    
6.  对于函数类型的 实参，检验 各 函数形参类型 与 函数返回值类型，并向集合中添加 它们的 类 与命名空间 的关联集合。
    
7.  对于 指向类 X 的成员函数 F 的 指针类型的 实参，检查 各函数形参类型，函数返回值类型 即 类 X， 并向集合中 添加 它们的 类 与命名空间 的关联集合。
    
8.  对于 指向类 X 的 数据成员 T 的 指针类型的 实参， 检查 该成员类型 和 类型 X，并向集合 添加 它们的类 与命名空间的关联集合
    
9.  如果实参 是 一组重载函数(或函数模板) 的名字 或取址 表达式，那么 检查 重载集合中的 各个函数，并向 集合 添加 它的 类 与命名空间的关联集合。
    
    1.  另外，如果以 模板标识(带模板实参的 模板名) 指明 重载集，那么 检查 它的所有 类型模板实参 与模板模板实参 (但不包括 非类型模板实参)，并向 集合添加它的类 与命名空间的 关联集合。

如果类与命名空间的关联集合中 的任何命名空间 是 内联命名空间(。。在本文前面有收录)，那么向 集合中 添加它的 外围命名空间。
如果类与命名空间的关联集合中的任何命名空间 直接含有 内联命名空间，那么向集合中添加该 内联命名空间。

在确定 命名空间与类 的关联集合后，为了进一步ADL处理，忽略 此集中 所有 在类中找到的 声明，但不包括 命名空间 作用域的 友元函数 及 函数模板。

根据下列特殊规则，将通过 常规名字查找 所找到的声明的集合，与通过 ADL 所 生成的 关联集合 的 所有元素 中找到的 声明集合 进行合并

1.  忽略 关联命名空间中的 using 指令
2.  在 关联类中 的 命名空间作用域 声明的 友元函数 (及函数模板) 通过 ADL 可见，即使它们通过 普通查找不可见。
3.  忽略 除 函数 与 函数模板外 的所有名字 (不会与 变量之间 发生冲突)

Notes
受 实参依赖查找 的影响，在 相同命名空间定义的 非成员函数 和 非成员运算符 被认为 是 该类 公开接口的一部分 (如果它们被 ADL 找到)。
ADL 是 泛型swap 能有效的原因:
using std::swap;
swap(obj1, obj2);

因为直接调用 std::swap(ob1, ob2) 的话，不会考虑 用户定义的 swap 函数，它可能在 obj1 和 obj2 类型定义 所在命名空间中； 但是 只调用 swap(o1,o2) 的话，会在 没有用户定义重载时 无法调用 任何函数， 特别是 std::iter_swap 与所有其他标准库算法 在处理 可交换 类型时 使用此手段。

名字查找规则 使得 在全局 或 用户 定义 命名空间 中 声明 对 来自std 命名空间 的类型 进行操作 的运算符 变得不切实际，例如，对于 std::vector 或 std::pair 的自定义 operator+ 或 operator>> (除非 vector/pair 的元素类型 是用户定义类型，这会将它的 命名空间 添加到 ADL 中)。 这种 运算符 不会 从 诸如 标准库算法 的模板实例化中 被查找到。进一步细节 见 待决名 https://zh.cppreference.com/w/cpp/language/dependent_name

ADL 能找到 完全在 类 或 类模板 内定义的 友元函数 (典型的例子是 重载的 运算符)，即使它始终没有在 命名空间层次进行声明

```C++
template<typename T>
struct number
{
    number(int);
    friend number gcd(number x, number y) { return 0; }; // 类模板内的定义
};
 
// 除非提供匹配声明，否则 gcd 是此命名空间的不可见成员（除非通过 ADL）
void g()
{
    number<double> a(3), b(4);
    a = gcd(a,b); // 找到 gcd ，因为 number<double> 是关联类，
                  // 使 gcd 在它的命名空间（全局命名空间）可见
//  b = gcd(3,4); // 错误：gcd 不可见
}
```

尽管 普通查找 在 找不到结果时 也能通过 ADL 解析函数调用，但是对 带 显式指定模板实参 的 函数模板 调用 还是 要求 存在 普通查找 所能找到的 模板声明 (否则，它将是 遇到未知名字后随小于号 的语法错误)

```C++
namespace N1
{
    struct S {};
 
    template<int X>
    void f(S);
}
 
namespace N2
{
    template<class T>
    void f(T t);
}
 
void g(N1::S s)
{
    f<3>(s);     // C++20 前是语法错误（无限定查找找不到 f）
    N1::f<3>(s); // OK，有限定查找找到模板 'f'
    N2::f<3>(s); // 错误： N2::f 不接收非类型模板形参
                 //       N1::f 不能被找到，因为 ADL 仅适用于无限定名
 
    using N2::f;
    f<3>(s); // OK：无限定查找现在找到 N2::f，
             //     然后因为此名无限定所以 ADL 表态并找到 N1::f
}
```

下列语境中进行ADL (即仅在关联的命名空间中查找)
对于 range for 循环，成员查找失败时，查找非成员函数 begin 和 end。 C++11起
从模板实例化点进行 待决名查找
结构化绑定声明 对 元组式 类型 查找非成员函数 get。 C++17起

==================

==================

==================

https://zh.cppreference.com/w/cpp/language/expressions

==================

==================

==================

==================

==================

==================

==================

==================

# C

https://en.cppreference.com/w/c/chrono

==================
https://en.cppreference.com/w/c/memory

==================
https://en.cppreference.com/w/c/program

==================
https://en.cppreference.com/w/c/header

==================

==================

https://en.cppreference.com/w/cpp/thread

==================

[https://en.cppreference.com/w/cpp/string/basic\_string\_view](https://en.cppreference.com/w/cpp/string/basic_string_view)

==================

https://zh.cppreference.com/w/cpp/meta

==================

==================

==================

==================

==================

==================
https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md

# ==C++ Core Guidelines==

October 6, 2022

。。这个有专门一个笔记的，但是没有完成。

==================

https://www.stroustrup.com/bs_faq2.html

# ==网站faq2==

==================

==================

==================

==================

==================

==================

==================

==================

==================

已使用 Microsoft OneNote 2016 创建。