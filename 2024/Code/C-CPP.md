C-CPP
2021年8月25日
8:40

[[toc]]

---
---

INT_MIN = -INT_MIN

// 转换复数
//        sscanf(x.c_str(), "%d+%di", &a, &b);

//        char buff;
//        stringstream aa(a), bb(b), ans;
//        aa >> ra >> buff >> ia >> buff;
//  似乎会自动  识别  的。ra ia  是int。

partial_sum

//clock_t clock(void);
//CLOCKS_PRE_SEC

//https://en.cppreference.com/w/cpp/thread
// sem_wait
// condition_variable

//      if (g[i][j] == 0) {

//        unordered_set<int> s = { get(i + 1, j, g), get(i - 1, j, g), get(i, j + 1, g), get(i, j - 1, g) };

//        res = max(res, 1 + accumulate(begin(s), end(s), 0, [&](int a, int b) {return a + sizes[b]; }));

//      }

//inner_product(begin(s[i]), end(s[i]), begin(m[j]), 0, plus<int>(), equal_to<int>())

//int first[26] = {[0 ... 25] = INT_MAX}, last[26] = {}, res = 0;

//    int memo[27][27][301] = {[0 ... 26][0 ... 26][0 ... 300] = -1};

// not std::
//__builtin_ffs(x)          // sz1 - last 1's index
//__builtin_clz(x)          // prefix 0's count
//__builtin_ctz(x)          // suffix 0's count
//__builtin_popcount(x)     // 1's count
//__builtin_parity(x)       // 1's count % 2 == 1(or 0)

//这些函数都有相应的usigned long和usigned long版本，只需要在函数名后面加上l或ll就可以了，比如int __builtin_clzll。

// partial_sort 是  heap 来完成的。
    myvi v2 = {7,6,5,4,3,2,1};
    std::partial_sort(begin(v2), begin(v2) + 2, end(v2));
    showVectorInt(v2);
    // 是  第一个参数 到 第三个参数 之间的范围，有序后， 放到 第一个到 第二个参数 之间。 所以 后n个不可能。。除非 自定义方法，然后 降序。

    std::partial_sort(begin(v2), begin(v2) + 2, end(v2), [](const int a, const int b){

                        return a > b;
                      });

// vector<int> 的  == , 是比较元素的。

//        return all_of(begin(char_count), end(char_count),
//                      [&](int c) { return c % words.size() == 0; });

//res.erase(it) always returns the next valid iterator, if you erase the last element it will point to .end()

// while (it != res.end()) {
//        it = res.erase(it);
// }

//for( ; it != res.end();)
//{
//    it = res.erase(it);
//}

//for ( ; it != res.end(); ) {
//  if (condition) {
//    it = res.erase(it);
//  } else {
//    ++it;
//  }
//}

//        int maxSum[n][n];
//        memset(maxSum,0,sizeof maxSum);

//    auto fun = [&matrix](const pair<int, int>& p1, const pair<int, int>& p2){ return matrix[p1.first][p1.second] > matrix[p2.first][p2.second]; };

//    priority_queue<pair<int, int>, vector<pair<int, int>>, decltype(fun)> que1(fun);

//vvi.emplace_back((vector<int>){startTime[i], endTime[i], profit[i]});

//    fill(&dp[0][0][0],&dp[0][0][0]+(n+1)*27*27,INT_MAX) ;

    cout<<std::numeric_limits<int>::max()<<endl;

    int mod(1e9+7);
    cout<<mod<<endl;
//    int mod2{1e9 + 7};            // connot convert double to int...
//    cout<<mod2<<endl;

//        int dp[n+1][d+1];
//        fill_n(&dp[0][0],(n+1)*(d+1),1e9);

    int arr[2][2] = {{1,2},{3,4}};
    auto& [x,y] = arr[1];
    cout<<x<<", "<<y<<endl;

    using ti = tuple<int, int, int>;
    priority_queue<ti, vector<ti>, greater<>> pq;
    pq.emplace(1,5,1123);
    cout<<std::get<2>(pq.top())<<endl;

    auto [t1,t2,_] = pq.top();              // _ 是变量名字，不能重复，  []中变量个数要等于声明的tuple的元素个数。

                                    // 类型必须auto。 反正用int 不行。说 structured binding declaration cannot have type 'int'

string(1, s[0])
// 1个char

    try
    {
//        throw -2;         // int

//        throw "error";      // ..not string, it is    const char* chp,   must const.

//        throw 1.23;       // double
//        throw string("asdasd");     // string
        throw new string("zzz");        // string *
    }
    catch (int i)
    {
        cout<<"catch: i "<<i<<endl;
    }
    catch (double d)
    {
        cout<<"catch: d "<<d<<endl;
    }
    catch (const char* chp)
    {
        cout<<"catch: char* "<<chp<<endl;
    }
    catch (string *s)
    {
        cout<<"catch: &s "<<s<<", "<<*s<<endl;
        delete s;
    }
    catch (string s)

    {   // 我竟然尝试把  {}删除。。。  后来一想，java也不能删除的吧(还真没有试过不写{})。。。可能是 catch { 同行，导致代码压缩，有种不需要{}的错觉。。

        cout<<"catch: s "<<s<<endl;
    }
    catch (...)         // must be the last handler
    {
        cout<<"catch: "<<"......"<<endl;
    }

    // 默认是最大堆，下面是最小堆。

    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> q;

//    cout.setf(ios::fixed);        // == cout<<setiosflags(ios::fixed) == cout<<fixed;

    cout<<setiosflags(ios::fixed)<<setprecision(15)<<1.000000000000001<<endl;          // include iomanip

    cout.unsetf(ios::fixed);
    cout<<1.000000000000001<<endl;

//        istringstream in(preorder);
//        vector<string> v;
//        string t = "";
//        int cnt = 0;
//        while (getline(in, t, ',')) v.push_back(t);

//    nth_element(begin(nums), begin(nums) + k - 1, end(nums), [](const auto &a, const auto &b){

//        return a.size() > b.size() ? true : a.size() < b.size() ? false : a > b;

//    });

//  nth_element

You use while (start <= end) if you are returning the match from inside the loop.

You use while (start < end) if you want to exit out of the loop first, and then use the result of start or end to return the match.

。。但是感觉好像不是，我的感觉是  end的初始值是否  是  一个有效的下标。

===========================

class Solution {
public:

    vector<int> nums;

    Solution(vector<int>& nums) {
        // Do not allocate extra space for the nums array
        this->nums.swap(nums);
        // this->nums = nums; 其实就可以。上面这句话只是一个巧妙的表达
    }

================================

static_cast

================================

================================
函数后置const
表明该函数是只读函数，不会修改数据成员。

函数被const  标识后，不能修改成员数据
函数被const  标识后，只能调用  被const  标识的  函数

。。类函数

class Foo
{
public:
Foo()
{
data = 10;
}
void show() const
{
//data = 11; //无法修改
std::cout << data << std::endl;
}
private:
int data = 0;
};

这里的const表示，在方法内隐式的this指针是个const类型，实际上就是const Foo* this。所以我们无法在show方法内直接修改data的值，除非将int data = 0;这里的定义加一个关键字mutable。

class Foo
{
public:
Foo()
{
data = 10;
}
void show() const
{
//data = 11; //无法修改
std::cout << data << std::endl;
}
int& get()
{
return data;
}
//无法编译，因为const表示this指针为const，也就是无法修改成员
//因此你返回的引用也必须是const
//否则你返回成员引用，类外不就变成了可变吗。
/*int& get1() const
{
return data;
}*/
//const int& 是一个通用引用类型
//即可表示不可修改左值也可以表示右值
const int& get2() const
{
//return 11; //所以返回左值右值都可以
return data;
}
private:
int data = 0;
};

================================
函数后置  &

引用限定符  加载  成员方法之后，确保这个方法可以被什么类型的  对象调用。

class Foo
{
public:
Foo()
{
data = 10;
}
void show() &
{
std::cout << data << std::endl;
}
void show2() const &
{
std::cout << data << std::endl;
}
void show3() &&
{
std::cout << data << std::endl;
}
private:
int data = 0;
};
int main(int argc, char* argv[])
{
Foo f;
f.show(); //show方法可以被左值对象调用，没问题
f.show2(); //show2方法可以被const左值对象调用，虽然我们不是const，但是同样没问题
f.show3(); //报错 show3方法只允许右值对象调用

const Foo f2;
f2.show(); //报错 show方法只允许左值对象调用，但是我们现在是个const左值
f2.show2(); //show2方法可以被const左值对象调用，没问题
f2.show3(); //报错 show3方法只允许右值对象调用

//那么show3方法如何调用呢？还记得std::move()吗，他其实只是转义为右值，并没有真的去move什么东西。我们转为右值后调用试试
std::move(f).show3(); //没问题
return 0;
}

================================

================================

LT2558

    make_heap(begin(g), end(g));
    while (k--) {
        pop_heap(begin(g), end(g));
        g.back() = sqrt(g.back());
        push_heap(begin(g), end(g));
    }
    return accumulate(begin(g), end(g), 0LL);

================================

动态分配数组，元素是char
char  *ptr = new  char [N];

动态分配数组，元素是  char*
char  **chars = new  char * [vSize];

编译错误
char  **chars = new  ( char *) [vSize];
比较好的一个解释是   new 后面不能跟()，  c++标准中没有这样的语法。

int  *p = new  ( int )[N];    也是编译错误

================================

================================

================================

================================

================================

================================

================================

================================

================================

已使用 Microsoft OneNote 2016 创建。

```C++
// LT2679
        const int cols = size(nums[0]);
        vector<int> maxs(cols);
        for (vector<int>& row : nums) {
            sort(begin(row), end(row));
            for (int c = 0; c < cols; ++c)
                maxs[c] = max(maxs[c], row[c]);
        }
        return reduce(begin(maxs), end(maxs));

```


# `array<int, 3>` faster then `vector<int>`, why?

// LT1584's discuss's comment
// votrubac lzl124631x

Do you know why `array<int, 3>` is way performant than `vector<int>`?

**array** is just fixed size array and it is allocated on the stack by the compiler, 
==no sys call== is involved everything can be done in the user mode(no context switch etc), 

**vector** is dynamic and have some default initial size(here I am talking about "capacity", 
the size of the internal ds, which will be different depending on the implementation like 4, 8 ), 
it is allocated from the heap via malloc / new (which is implemented by ==mmap -> sys call==)




# cpp collection

## vector.at

std::out_of_range

```C++
    std::vector<int> data = {1, 2, 4, 5, 5, 6};
    // Set element 1 to 88
    data.at(1) = 88;
```

```C++
std::vector<int> vi(sz1);
for (int i = 0; i < sz1; ++i)
    cin >> vi.at(i);
```

## optional
- operator*, operator->
- has_value()
- value()
- value_or()
- reset(), 调用内部的值的析构函数来 销毁内部的值
- emplace()
- swap() 交换内部的值

### make_optional
```C++
int main()
{
    auto op1 = std::make_optional<std::vector<char>>({'a','b','c'});
    std::cout << "op1: ";
    for (char c : op1.value())
        std::cout << c << ',';

    auto op2 = std::make_optional<std::vector<int>>(5, 2);
    std::cout << "\nop2: ";
    for (int i : *op2)
        std::cout << i << ',';

    std::string str{"hello world"};
    auto op3 = std::make_optional<std::string>(std::move(str));
    std::cout << "\nop3: " << std::quoted(op3.value_or("empty value")) << '\n';
    std::cout << "str: " << std::quoted(str) << '\n';
}
```

## unique_ptr
- release(), 返回被管理对象的指针，并且释放所有权
- reset(x)，替换被管理的对象，删除原来的对象
- swap
- get，返回一个指针 指向 被管理的对象， 或者 nullptr
- get_deleter，用于析构被管理的对象，用不到的。

p.reset(p.release()) 不涉及 self-reset， p.reset(p.get())才涉及。

## make_unique
这个要提供构造器的形参

## make_unique_for_overwrite
这个直接 `new T`, 或 `new T[size]`

## shared_ptr
https://en.cppreference.com/w/cpp/memory/shared_ptr

- reset
- swap
- get
- *, ->
- []
- use_count
- unique

make_shared
make_shared_for_overwrite


## weak_ptr

- reset
- swap
- use_count, shared_ptr 的数量
- expired, check it was deleted?
- lock, create a shared_ptr



## promise/future/packaged_task

```C++
void f_promise(std::promise<int>& p) { p.set_value(123); }
void f_future(std::future<int>& f) { auto a = f.get(); }

std::promise<int> proms;
std::future fut = proms.get_future();

std::jthread j1 {f_promise, std::ref(proms)};
std::jthread j2 {f_future, std::ref(fut)};
```

```C++
double accu(std::vector<double>::iterator st, std::vector<double>::iterator en, double init) { }

// --------
std::packaged_task pt1 {accu};
std::packaged_task pt2 {accu};

std::future<double> f1 {pt1.get_future()};
std::future f2 {pt2.get_future()};

// ok
// std::jthread t1 {std::move(pt1), vd.begin(), vd.begin() + vd.size() / 2, 0};
// std::jthread t2 {std::move(pt2), vd.begin() + vd.size() / 2, vd.end(), 0};

// ok
std::thread t1 {std::move(pt1), vd.begin(), vd.begin() + vd.size() / 2, 0};
std::thread t2 {std::move(pt2), vd.begin() + vd.size() / 2, vd.end(), 0};
t1.join();      // 必须有，不然 terminate called without an active exception
t2.join();

return f1.get() + f2.get();
```

## async
```C++
double accu(std::vector<double>::iterator st, std::vector<double>::iterator en, double init) { }

double f_async(std::vector<double>& vd)
{
    if (vd.size() < 1)
    {
        return accu(vd.begin(), vd.end(), 0.0);
    }
    auto sz = vd.size();

    auto f0 = std::async(accu, vd.begin(), vd.begin() + sz / 4, 0.0);
    auto f1 = std::async(accu, vd.begin() + sz / 4, vd.begin() + sz / 2, 0.0);
    auto f2 = std::async(accu, vd.begin() + sz / 2, vd.begin() + sz * 3 / 4, 0.0);
    auto f3 = std::async(accu, vd.begin() + sz * 3 / 4, vd.end(), 0.0);

    return f0.get() + f1.get() + f2.get() + f3.get();
}
```


## stop_token

```C++
// stop other threads, when you got result
std::atomic<int> res_idx = -1;       // result's index

template<typename T> struct Range { T* first; T* last; };

void find(std::stop_token tok, const int* base, const Range<int> r, const int target)
{
    for (int* p = r.first; p != r.last && !tok.stop_requested(); ++p)
    {
        std::cout<<"to check "<<*p<<std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(20));
        if (target == *p)
        {
            res_idx = p - base;
            return;
        }
    }
}
void f_stop_other()
{
    std::vector<int> vi;
    for (int i = 0; i < 100; ++i)
    {
        vi.push_back(i);
    }
    int key = 65;

    // ---------
    int md = vi.size() / 2;         // 2 thread
    int* pvi = &vi[0];

    std::stop_source ss1{};
    std::jthread t1 {find, ss1.get_token(), pvi, Range{pvi, pvi + md}, key};

    std::stop_source ss2{};
    std::jthread t2 {find, ss2.get_token(), pvi, Range{pvi + md, pvi + vi.size()}, key};

    using namespace std::literals::chrono_literals;
    while (res_idx == -1)
        std::this_thread::sleep_for(10ms);

    ss1.request_stop();
    ss2.request_stop();

    std::cout<<"stop_thread.. "<<res_idx<<std::endl;
}
```


---

# C++线程安全的单例

BV12E421A7KT

## DCL

看了下评论，我能理解 DCL (double check lock) 理论上会出问题，但是我认为实际上不会出现这个问题。或者问题不是这个。

认为有问题的 DCL
```C++
#pragma once

#include <mutex>

class Singleton {
    static Singleton *instance;
    static std::mutex mutex;
    Singleton() = default;
public:
    Singleton(const Singleton &) = delete; // 。。对，const可以接收 普通的对象， 非const无法接收 const对象，所以这里 const形参 覆盖更广。

    Singleton &operator=(const Singleton&) = delete;

    static Singleton *getInstance() {
        if (instance == nullptr) {
            std::lock_guard<std::mutex> lock(mtx);
            if (instance == nullptr) {
                instance = new Singleton();
            }
            return instance;
        }
        return instance;
    }
};
Singleton *Singleton::instance = nullptr;
std::mutex Singleton::mutex;        // 。不太理解这2行的作用，是初始化？
```

评论是说，多线程情况下，可能 一个线程在 内部new后 写入到 instance，而另一个线程在外层读取 instance， 同时读写，发生 数据竞争。

。看到的时候，一些评论的评论 已经删了，不知道怎么反驳了，但是 删了，说明失败了。

。。但是
数据竞争，是会出现，但是不会影响逻辑啊，外层只是判断是否为 nullptr (0)。至于 64位指针 只被赋值了前32位，不会影响 判断结果。  
而且，现在64位系统，一次性就读写64位，不会出现 赋值一半吧。 32位系统，指针也是32位的，赋值 也是原子的吧。 。评论的评论 有提到这个，但是 又有人说，"架构比较弱，可能就不是原子的"。。反正我不明白。  

当然，可能出现错误，但是这种错误应该是不会发生的，毕竟写编译器的人 应该能确保 不会发生这种错误吧？  
这里的错误是指：线程A进行初始化，申请了空间，把地址赋给了 instance，此时还没有 执行构造器， 线程B 判断 instance 非空，然后 外面就开始使用 这个单例了， 此时这个单例 还没有 初始化。 会出问题。

看视频的时候，也提到了会出现这种乱序，但是我感觉不太可能啊。 而且 这个问题 通过 tmp 就应该解决了，不需要 设置 内存序吧。  
想到这个错误的时候，我没有想到解决方案，看视频的时候，用了 tmp，感觉就可以了， 但是又加 内存序。 感觉有点 画蛇添足了。   
但是现在想想，确实啊， 申请空间，赋值tmp，tmp赋给instance，执行构造器，确实有这个可能。   但 还是那句话，写编译器的人不会这么蠢吧。   
但是，好像和编译器也没什么关系， 乱序 可以发生在CPU上，所以 写编译器的人 只能保证 编译时 不发生乱序 (但是没有文章说 有这个保证)， 执行时 CPU还会乱序的，这个CPU乱序，编译器无法保证的。  所以 只能加 内存序。

。。所以，读写 的 数据竞争 并不会导致 逻辑错误，只有 乱序 会导致。
。。所以 只需要添加 内存序，不需要tmp ？
。。不去管了， 因为 有 静态局部变量，这种是 C++中 单例的 最好方法。


---

视频内容

DCL + tmp  
。。不过这个tmp和我想的不太一样，我想的是 只是 内层if 中: { Singleton* tmp = new Singleton(); instance = tmp }, 外面return 的是 instance。

```C++
static Singleton *getInstance() {
    Singleton *tmp = instance;
    if (tmp == nullptr) {
        std::lock_guard<std::mutex> lock(mutex);
        tmp = instance;
        if (tmp == nullptr) {
            tmp = new Singleton();
            instance = tmp;
        }
    }
    return tmp;
}
```

DCL + tmp + memory_order

instance 从 `static Singleton*` 类型变成了 `static std::atomic<Singleton*>` 类型。

```C++
static Singleton *getInstance() {
    Singleton *tmp = instance.load(std::memory_order_release);
    std::atomic_thread_fence(std::memory_order_acquire);
    if (tmp == nullptr) {
        std::lock_guard<std::mutex> lock(mutex);
        tmp = instance.load(std::memory_order_relaxed);
        if (tmp == nullptr) {
            tmp = new Singleton();
            std::atomic_thread_fence(std::memory_order_release);
            instance.store(tmp, std::memory_order_relaxed);
        }
    }
    return tmp;
}
```

。。我在想， lock_guard 应该 带了 内存屏障了，。。没用， 这个的屏障 无法影响到 构造。


更简洁的写法
```C++
static Singleton *getInstance() {
    Singleton *tmp = instance.load(std::memory_order_acquire);
    if (tmp == nullptr) {
        std::lock_guard<std::mutex> lock(mutex);
        tmp = instance.load(std::memory_order_relaxed);
        if (tmp == nullptr) {
            tmp = new Singleton();
            instance.store(tmp, std::memory_order_release);
        }
    }
    return tmp;
}
```


## call_once

```C++
#pragma once
#include <mutex>

class Singleton {
    static Singleton *instace;
    static std::once_flag initFlag;
    Singleton() = default;
    static void init() {
        instance = new Singleton();
    }
public:
    Singleton(const Singleton&) = delete;
    Singleton &operator=(const Singleton&) = delete;
    static Singleton *getInstace() {
        std::call_once(initFlag, &Singleton::init);
        return instance;
    }
};

Singleton *Singleton::instance = nullptr;
std::once_flag Singleton::initFlag; // 。也是 不清楚这2行的作用， 初始化？
```



## 静态局部变量(推荐)

```C++
#pragma once
class Singleton {
    Singleton() = default;
public:
    Singleton(const Singleton&) = delete;
    Singleton &operator=(const Singleton&) = delete;
    static Singleton &getInstance() {
        static Singleton single;
        return single;
    }
};
```

### 奇异递归模板(CRTP)

为了避免重复编写单例，可以使用 奇异递归模板

```C++
#pragma once

template<typename T>
class Singleton {
protected:
    Singleton() = default;
public:
    Singleton(const Singleton&) = delete;
    Singleton &operator=(const Singleton &) = delete;
    static T &getInstance() {
        static T single;
        return single;
    }
};
```

```C++
#pragma once
#include <iostream>
#include "singleton_template.h"

class Single: public Singleton<Single> {
    Single() = default;
    friend class Singleton<Single>;
public:
    void greet() {
        std::cout<<"single greet\n";
    }
};
```



---

# barrier

https://en.cppreference.com/w/cpp/thread/barrier

看api，看例子
没有手工重置的功能，自动重置的。



BV14GvCe7E6f

C++ 20

github my_cpp_code/any/concurrency/profession_cpp_ch27.cpp 中也有barrier的代码， 和这里的不同。

。看起来 barrier可以自动复位。不需要手工操作， 不知道能不能手工操作。


应用
1. 阶段同步，多个线程 都完成某个阶段后 才能进入下一个阶段
2. 批处理，多个线程 需要在 某些 关键点 汇聚数据或状态，然后再继续各自的任务。


代码
```C++
#include <iostream>
#include <thread>
#include <barrier>
#include <vector>

void worker(int id, std::barrier<>& sync_point) {
    std::cout<<"worker "<<id<<" is doing phase 1 work\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(100 * id));

    sync_point.arrive_and_wait();
    std::cout<<id<<" phase 2\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(100 * id));
}

int main() {
    const int num_threads = 5;
    std::barrier sync_point(num_threads);

    std::vector<std::thread> threads;
    for (int i = 0; i < num_threads; ++i) {
        threads.emplace_back(worker, i+1, std::ref(sync_point));
    }
    for (auto& t : threads) {
        t.join();
    }
    std::cout<<"all worker complete\n";

    return 0;
}
```

效果是： 5个worker完成 第一阶段， 然后一起开始 第二阶段。




# 异步

是一个合集

## future, async

BV1puaDeQEUN

C++11 标准库 引入了： std::async, std::future, std::promise


### 概念，future,async,promise,packaged_task

- future
  模板类， 用于表示一个异步操作的结果，提供了一种机制：允许你在将来的某个时间点获取异步操作的结果
  提供 get()，用于获得 异步操作的结果，如果结果未准备好，get会阻塞， ==get只能调用一次==，多次会抛出 std::future_error异常
  future和promise 共享状态，promise用于设置结果，future用于获取结果
  还提供了 wait，wait_for, wait_until 来阻塞当前线程，直到 结果可用 或 时间到
- async
  函数模板，用于启动一个异步任务，返回 std::future。
  内部 可以 是 启动新线程，也可以是 在调用 future的 get,wait 时 才真正运行
  启动策略：
  - std::launch::async，在新线程中执行
  - std::launch::deferred，在 get，wait 时，主线程 执行
  - 默认：std::launch::async | std::launch::deferred ，系统自动选择，视频说 由 编译器，库，当前系统资源 决定。 。。感觉不太可能 根据 当 前 系统资源，这个太难以界定了。 不过都用 async，肯定是为了 异步啊，你弄个 deferred 有什么意思？
- promise
  模板类，用于在不同线程之间传递异步操作的结果
  `set_value(ans)` 设置结果
  `promise.get_future()` 获取 future 对象
- packaged_task
  模板类，将 可调用对象 包装成异步任务，并关联一个 future
  `std::packaged_task([]{return 111;});`  包装任务
  `task.get_future()` 获得 future
  `task();` 执行任务
  packaged_task 是不可复制的对象


### code, async,future

```C++
#include <iostream>
#include <thread>

// #include <future>  // 有点奇怪，视频里没有future 头文件，但是baidu下， thread 不包含future啊

using namespace std;


void fun1() {
    cout<<"this is fun1"<<std::endl;
}

int fun2(int a, int b) {
    cout<<"fun2 "<<a<<", "<<b<<std::endl;
    return a + b;
}

int main() {
    thread t1(fun1);
    t1.join();

    thread t2(fun2, 11, 22);  // 无法获得返回值
    t2.join();


    // async(std::launch::async, fun2, 4, 5); // 通过 std::this_thread::get_id() 察看2种launch的区别
    std::future<int> fut = async(fun2, 33, 44);
    std::cout<<"fut: "<<fut.get()<<std::endl;


    return 0;
}
```

## promise

BV1HLaXexEjw

用于设置异步操作结果。

可以设置 值 或 异常

通常和 std::async, std::packaged_task, std::thread 结合使用。

一个promise 只能创建一个 future对象。 要多个future，需要调用 std::future::share() 获得 std::shared_future 对象。一旦share后，原future不再拥有 数据( 被move给 shared_future 了)。 ==shared_future 是 可以复制的==。 shared_future 可以直接把 future作为 构造器参数。
。https://en.cppreference.com/w/cpp/thread/shared_future

只有 promise的 set_value 或 set_exception 被调用， future才可以 获得 结果。


```C++
#include <iostream>
#include <thread>
#include <future>

using namespace std;

void async_task(std::promise<int> prom) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    prom.set_value(33);
}

int main() {
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();

    std::thread t(async_task, std::move(prom)); // move 或指针， 指针的话 async_task 形参也需要改

    std::cout<<"result: "<<fut.get()<<std::endl;

    t.join();  // 需要吗？ get 应该能卡住 t 了啊

    return 0;
}
```

## packaged_task

BV1jbY7eXEri

模板类，将一个可调用对象(lambda，函数，函数对象)包装起来，以便 异步地执行，并能获取其返回结果。

不可复制，只能move

创建 packaged_task，需要传递一个可调用对象
使用 get_future() 获得 future
通过 operator() 或在 新线程中调用 std::thread 来执行，  packaged_task 必须通过 std::move 传递

```C++
#include <iostream>
#include <future>

using namespace std;

int main() {
    std::packaged_task<int(int, int)> task([](int a, int b){
        return a + b;
    });

    future<int> fut = task.get_future();
    task(11, 22);   // 执行，非异步

    cout<<fut.get()<<endl;
    return 0;
}
```

---

```C++
#include <iostream>
#include <thread>
#include <future>
#include <functional>

int fun(int a, int b) {
    return a + b;
}

int main() {
    // or <decltype(fun)>
    std::packaged_task<int(int, int)> task(fun);

    std::future<int> fut = task.get_future();

    std::thread t(std::move(task), 22, 44);

    int value = result.get();

    std::cout<<"ans: "<<value<<std::endl;

    t.join(); // 需要？ get可以卡住t了啊？, 不过 算是一种 标准的 流程。

    return 0;
}

```

---









