CPP Code 注意点


[[toc]]


# ==抽象带来耦合==

# ==组合优于继承==




# 不要 using namespace std;

特别是全局中声明 using namespace std; ( 这个可以在 方法中声明，但也少用)
`使用 using std::string, std::cout, std::endl;`

---

# cout<<endl 会进行fflush，消耗时间

`cout<<std::endl`  会刷新buffer，消耗时间。
`cout<<'\n'` 即可

---

使用 range for，而不是普通的 for

---
使用stl中算法

---

# C风格的数组作为形参后，会退化为指针，丢失size信息

不要使用 C风格的数组，它会退化为指针，需要手动传递size。
使用 `std::array<int, n> arr{}`

---

# 不要使用 reinterpret_cast

从C++20 开始:
```C++
template<typename T>
viod print_bytes(const T& input)
{
	using bytearray = std::array<std::byte, sizeof(T)>;
	const auto& bytes = std::bit_cast<bytearray, T>(input);
}
```

---

# 使用 map2.at("a") 替换 map2["a"]

```C++
void fun(const std::unordered_map<std::string, int>& map2)
{
//	map2["a"];	// 报错，因为 [] 可能插入默认值。
	map2.at("a");	// at() 是 operator[] 的 const 版本
}
```

---
# 如果函数内部不修改形参，则声明const 形参

---

```C++
const char *string_lift() {
	return "string literals";
}
```

字符串保证在 整个程序生命周期中都存在，所以上面是可行的。 虽然看起来像是对 局部变量的 ref。

---
# 结构化绑定

``` C++
std::map<std::string, std::string> colors = {
	{"red", "#FF0000"},
	{"green", "#00FF00"},
	{"blue", "#0000FF"}
};

// bad
for (const auto& pair : colors) {
	std::cout<<pair.first<<", "<<pair.second<<'\n';
}

// good
for (const auto&[name, hex] : colors) {
	// ..
}

```

```C++
struct S {
	int a;
	std::string s;
};

S get_S();

void use_S() {
	// 根据结构中 声明的顺序
	const auto [name_for_a, name_for_s] = get_S();
}

```

---
# 不要使用 输出形参， 使用struct

```C++
// bad
void get_out_params(const int n, int& out1, int& out2) {
	// ...
}

// good
struct Values {
	int x, y;
};

Values get_out_struct(const int n) {
	return {n, n + 1};
}

// 还可以配合结构化绑定
void use_Values() {
	auto [x, y] = get_out_struct(2);
}

```

---
# 不要将 编译时可以完成的工作 放到 运行时去完成。
## constexpr
```C++
// bad
int sum_of_1_to_n(const int n) {
	return n * (n + 1) / 2;
}

// good, 参数在编译时已知，所以可以：
constexpr int sum_of_1_to_n_2(const int n) {
	return n * (n + 1) / 2;
}

void uses_sum() {
	const int limit = 1000;
	auto ans = sum_of_1_to_n(limit);
}

```

---

# 析构函数 必须 虚函数
## 派生类的 析构函数 标记为 override。
。。override 会检查这个方法是否 覆盖了父类的 某个方法。 如果父类的方法 不存在 或  不是虚函数，这里会报错。

```C++
～Derived() override {
	// ...
}
```

---

# 类成员初始化属性是 类中声明的顺序。
类成员的初始化顺序是 在==类中声明的顺序==，==不是==构造器中 初始化列表中的 顺序。
。。所以 编写 初始化列表的时候，注意 依赖关系。

---

# default 和 value initialization 之间的区别。
。。垃圾值 和 0值 的区别

## 默认构造器

```C++
void default_vs_value_init() {
	// 默认，垃圾值
	int x;
	int *x2 = new int;
	
	// 初始化，值是 零值
	int y{};
	int *y2 = new int{};
	int *y3 = new int();
	
	int z();		// 函数声明
}

struct S {
	int n, m;
	std::string s;
}

void default_vs_value_init_2() {
	// n和m是垃圾值，s是空字符串。
	// s是空字符串，是因为： 对于 默认 和 值初始化，如果你定义了 默认构造器，则它将被调用。
	S x;
	S *x2 = new S;
	
	// n,m 是0， s 是空字符串。
	S y{};
	S *y2 = new S{};
	S *y3 = new S();
}

```


---

在容器上 循环时，添加 或 删除 元素。需要特别注意。
可能会导致 迭代器失效。

vector 可以不使用迭代器，直接使用下标。更加地易读。

---

# 不要 返回 move 的局部变量。

```C++
std::vector<int> make_vi(const int n) {
	std::vector<int> vi{1,2,3,4};
	
	// 不必，因为编译器会进行 返回值优化。
	// 编译器它知道，它始终可以 move 一个 local variable。
//	return std::move(vi);
	return vi;
}
```


---

# move

```C++
// move 的标准实现

template<typename T>
constexpr std::remove_reference_t<T> &&
move(T&& value) noexcept {
	return static_cast<std::remove_reference_t<T> &&>(value);
}

// int的
cosntexpr int&&
move(int &value) noexcept {
	return static_cast<int &&>(value);
}
```


---

# eval 的顺序
```C++
void fun()
{
	string s = "but .........";
	s.replace(0, 4, "")
		.replace(s.find("even"), 4, "only")
		.replace(s.find("dont"), 6, "");
}
```

在 C++17 前， 允许编译器 以任意顺序 计算 子表达式。
所以 可能先计算 s.find("even")。然后执行 replace(0,4,"")。然后执行 replace( ,4,"only")，此时由于 s.find("even") 获得的下标是 replace(0,4,"") 之前的，所以错位。
C++17 可以保证，如果 z.a().b().c()，那么保证 a先执行，然后b，然后c

但是，即使在 C++20 中，形参值的 计算顺序 依然是 不能保证 左到右的

---

# 少用 heap allocation

下面的代码，使用 stack分配就可以了。
RAII防止泄漏。

```C++
struct Record {
	int id;
	string name;
}

void fun() {
	Record *a = new Record{0, "aa"};
	Record *b = new Record{0, "bb"};
	
	// ...
	
	delete a;
	delete b;
}
```


---

# 直接构造unique_ptr,shared_ptr,不使用 make_unique,make_shared。

。。不知道为什么。。不太理解。

直接构造 unique_ptr 或 shared_ptr， 不要使用 make_unique, make_shared。

---
# 任何使用 new delete 的地方 都考虑 RAII

---
# 任何使用 open close 的地方 使用 RAII

``` C++
void read_from_file(char *name) {
	
	// bad
	FILE *fp = fopen(name, "r");
	fclose(fp);
	
	// good
	std::ifstream input{name};
	
}
```

---
# shared_ptr 引用计数是原子的，其他的不是。所以对 shared_ptr 反引用的时候，不能保证 线程安全。

---

# const 指针，const 的是 const右侧的， 如果右侧没有，那么就是 左侧。

---



---

# 使用常量来代替注释

如果代码需要注释，那么考虑能否简化，重构以优化代码。

```C++
// bad
// status is 5, means message sent
if (status == 5)
	msg.markSent();

// good
MESSAGE_SENT = 5
if (status == MESSAGE_SENT)
	msg.markSent();


// bad
if (msg.user.id == current_user.id && msg.delivered_time() is none or (datatime.now() - message.delivered_time().seconds < 5*60) or current_user.type == User.admin)
	msg.update_text(text)

// good
// ..不过弹幕里说， 短路没了。。确实。。
FIVE_MINUTES = 5 * 60
user_is_auth = msg.user.id == current_user.id
is_recent = msg.deliver_time() is None or (datatime.now() - msg.deliver_time).seconds < FIVE_MINUTES;
user_is_admin = current_user.type == User.admin

if (user_is_auth and is_recent) or user_is_admin:
	msg.xxx
```


---

# 如果可能不返回值，使用 std::optional

```C++
//bad
// -1 means no value
int64_t fun();

// good
std::optional<int64_t> fun();

```


# 以良好的方式编写C++ class

## header中的防卫式声明
防止头文件的内容被多次包含。
```C++
complex.h:
# ifndef  __COMPLEX__
# define __COMPLEX__
class complex
{

}
# endif
```

## 把数据放在private声明下，提供接口访问数据
```C++
# ifndef  __COMPLEX__
# define __COMPLEX__
class complex
{
    public:
        double real() const {return re;}
        double imag() const {return im;}
    private:
        doubel re,im;
}
# endif
```

## 不会改变类属性（数据成员）的成员函数，全部加上const声明
而且，const对象才可以调用这些函数——const对象不能够调用非const成员函数。

。。？那不是变差了？ 。。不，变好了。 方法不带 const的话， 只有 非const对象 才能调用这些方法。  方法带上const后， 所有对象都可以 调用这些方法。

```C++
double real () `const` {return re;}
double imag() `const` {return im;}
```

## 使用构造函数初始值列表

在初始值列表中，才是初始化。在构造函数体内的，叫做赋值。

```C++
class complex
{
    public:
        complex(double r = 0, double i =0)
            : re(r), im(i)  { }
    private:
        doubel re,im;
}
```

## 如果可以，参数尽量使用reference to const

为complex 类添加一个+=操作符：

```C++
class complex
{
    public:
        complex& operator += (const complex &)
}
```
使用 ==引用 避免 类对象构造与析构的开销==，使用==const确保参数不会被改变==。内置类型的值传递与引用传递效率没有多大差别，甚至值传递效率会更高。

例如，传递char类型时，值传递只需传递一个字节；引用实际上是指针实现，需要四个字节（32位机）的传递开销。但是为了一致，不妨统一使用引用。


## 如果可以，函数返回值也尽量使用引用

以==引用方式返回函数局部变量==会引发程序==未定义==行为，离开函数作用域局部变量被销毁，引用该变量没有意义。但是我要说的是，如果可以，函数应该返回引用。

当然，要放回的变量要有一定限制：该变量的在进入函数前，已经被分配了内存。以此条件来考量，很容易决定是否要放回引用。而在函数被调用时才创建出来的对象，一定不能返回引用。

说回operator +=，其返回值就是引用，原因在于，执行a+=b时，a已经在内存上存在了。

而operator + ，其返回值不能是引用，因为a+b的值，在调用operator +的时候才产生。

下面是operator+= 与’operator +’ 的实现：

```C++
inline complex & complex :: operator += (const complex & r)
{
        this -> re+= r->re;
        this -> im+= r->im;
        return * this;
}
inline complex operator + (const complex & x , const complex & y)
{
        return complex ( real (x)+ real (y), //新创建的对象，不能返回引用
                         imag(x)+ imag(y));
}
```

在operator +=中返回引用还是必要的，这样可以使用连续的操作：
`c3 += c2 += c1;`


## 如果重载了操作符，就考虑是否需要多个重载
就我们的复数类来说，+可以有多种使用方式：
```C++
complex c1(2,1);
complex c2;
c2 = c1+ c2;
c2 = c1 + 5;
c2 = 7 + c1;
```

为了应付怎么多种加法，+需要有如下三种重载：
```C++
inline complex operator+ (const complex & x ,const complex & y)
{
    return complex (real(x)+real(y),
                    imag(x+imag(y););
}
inline complex operator + (const complex & x, double y)
{
    return complex (real(x)+y,imag(x));
}
inline complex operator + (double x，const complex &y)
{
    return complex (x+real(y),imag(y));
}
```

## 提供给外界使用的接口，放在类声明的最前面
类的用户用起来也舒服，一眼就能看见接口。


# class with pointer member

## 带指针数据成员的类，需要自己实现class三大件：拷贝构造，拷贝赋值，析构
C++的类可以分为==带指针数据成员==与==不带==指针数据成员两类,complex就属于不带指针成员的类。
而这里要说的字符串类String，一般的实现会带有一个char *指针。
==带指针==数据成员的类，需要自己实现class三大件：==拷贝构造函数、拷贝赋值函数、析构函数==。


```C++
class String
{
    public:
        String (const char * cstr = 0);
        String (const String & str);
        String & operator = (const String & str);
        ~String();
        char * get_c_str() const {return m_data};
    private:
        char * m_data;
}
```

带指针的类不能依赖编译器的默认实现——这涉及到==资源的释放、深拷贝与浅拷贝的问题==。在实现String类的过程中我们来阐述这些问题。

## 析构函数释放动态分配的内存资源
如果class里有指针，多半是需要进行==内存动态分配==（例如String），==析构==函数必须负责在对象==生命结束时释放掉动态申请来的内存==，否则就造成了==内存泄露==。

局部对象在离开函数作用域时，对象析构函数被自动调用，而使用new动态分配的对象，也需要显式的使用delete来删除对象。而==delete实际上会调用对象的析构函数==，我们必须==在析构函数中完成释放指针m_data所申请的内存==。下面是一个构造函数，体现了m_data的动态内存申请：

```C++
/*String的构造函数*/
inline
String ::String (const char *cstr = 0)
{
    if(cstr)
    {
        m_data = new char[strlen(cstr)+1];   // 这里，m_data申请了内存
        strcpy(m_data,cstr);
    }
    else
    {
        m_data= new char[1];
        *m_data = '\0';
    }
}
```

这个构造函数以C风格字符串为参数，当执行
`String *p = new String ("hello");`
m_data向系统申请了一块内存存放字符串hello：

析构函数必须负责把这段动态申请来的内存释放掉：
```C++
inline
String ::~String()
{
    delete[]m_data;
}
```

## 赋值构造函数与复制构造函数负责进行深拷贝

如果使用编译器为String默认生成的拷贝构造函数与赋值操作符会发生什么事情?
==默认的复制构造函数或赋值操作符==所做的事情是对==类的内存进行按位的拷贝==，也称为==浅拷贝==，它们只是把对象内存上的每一个bit复制到另一个对象上去，==在String中就只是复制了指针==，而不复制指针所指内容。

现在，有2个String对象
```C++
String a("Hello");
String b("World");
```
如果执行
`b = a;`

浅拷贝，会导致，a和b的 m_data 指针 指向同一个 内存块。
也会导致 b 之前指向的 内存块 没有 被指针指向，变成了 无法利用的内存， 发生了 ==内存泄漏==

如果 析构a 时， 使用了之前带 delete m_data 的 析构函数，存储 的 内存块 就被 释放了， b.m_data 就是一个 野指针。

---

## 拷贝赋值函数 要点

我们自己实现的构造函数 复制的是指针所指的内存内容，这称为深拷贝
```C++
/*拷贝赋值函数*/
inline String &String ::operator= (const String & str)
{
    if(this == &str)           //1
        return *this;
    delete[] m_data;        //2
    m_data = new char[strlen(str.m_data)+1];        //3
    strcpy(m_data,str.m_data);            //4
    return *this
}
```

这是拷贝赋值函数的经典实现，要点在于:
1. 处理自我赋值，如果不存在自我赋值问题，继续下列步骤：
2. 释放自身已经申请的内存
3. 申请一块大小与目标字符串一样大的内存
4. 进行字符串的拷贝

### 复制构造
复制构造函数也是一个深拷贝的过程
```C++
inline String ::String(const String & str )
{
    m_data = new char[ strlen (str) +1];
    strcpy(m_data,str.m_data);
}
```
一定要在 operator = 中检查是否self assignment 
假设这时候确实执行了对象的自我赋值，左右pointers指向同一个内存块，前面的步骤 ② delete掉该内存块。
当企图对rhs的内存进行访问是，结果是未定义的

。。大家都指向同一块内存， 结果一方删除了， 另一方就是 野指针了。 就。。


# static 与 类

## 不和对象直接相关的数据，声明为static

想象有一个银行账户的类，每个人都可以开银行账户。存在银行利率这个成员变量，它不应该属于对象，而应该属于银行这个类，由所有的用户来共享。

。。但是，利率 对于不同账户，应该是不同的。。 可能分类的。

static修饰成员变量时，该成员变量放在程序的==全局区==中，整个程序运行过程中只有该成员变量的一份副本。
而普通的成员变量存在每个对象的内存中，若把银行利率放在每个对象中，是浪费了内存。

## static成员函数没有this指针

static成员函数与普通函数一样，都是只有一份函数的副本，存储在进程的代码段上。
不一样的是，static成员函数没有this指针，所以它不能够调用普通的成员变量，只能调用static成员变量。
普通成员函数的调用需要通过对象来调用，编译器会把对象取地址，作为this指针的实参传递给成员函数：

```C++
obj.func() ---> Class :: fun(&obj);
```

而static成员函数即可以通过对象来调用，也可以通过类名称来调用。


## 在类的外部初始化static成员变量

static成员变量必须在类外部进行定义

。。第一次知道。. 应该是  只能外部初始化，  生命周期 只分为 声明，初始化，  没有定义， 定义 应该就是 声明啊。

```C++
class A
{
    private:
        static int a; //①
}
int A::a = 10;  //②
```

注意 ①是声明，②才是定义，定义为变量分配了内存。



## static与类的一些小应用

### 饿汉式 单例

```C++
class A
{
        public:
            static A& getInstance();
            setup(){...};
        private:
            A();
            A(const A & rhs);
            static A a;
}
```

这里把class A的构造函数都设置为私有，不允许用户代码创建对象。
要获取对象实例需要通过接口getInstance。
”饿汉式“缺点在于无论有没有代码需要a，a都被创建出来。

### 单例 懒汉式

```C++
class A
{
    public:
        static  A& getInstance();
        setup(){....};
    private:
        A();
        A(const A& rsh);
        ...
};
A& A::getInstance()
{
        static A a;
        return a;
}
```

“懒汉式”只有在真正需要a时，调用getInstance才创建出唯一实例。
这可以看成一个具有拖延症的单例模式，不到最后关头不干活。
很多设计都体现了这种拖延的思想，比如string的写时复制，真正需要的时候才分配内存给string对象管理的字符串。






