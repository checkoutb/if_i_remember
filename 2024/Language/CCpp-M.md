CCpp-M
2022年10月9日
14:39


[[toc]]




# 内存中存储区域如下：

## 堆，栈，代码，全局/静态(+常量)

堆区

由  程序员  手动  new  和  delete  的内存区域。由  低地址  向  高地址  申请；  内存空间大；存储地址不连续；一般是  链式的；速度慢。

栈区

由  编译器自动  分配  和释放，主要存储  函数的  参数值，函数内部的  变量的值，函数调用的  空间。  从高地址  向  低地址  申请；容量优先；速度较快，存储地址连续，会溢出。

代码区
又称  文本段，存放着  程序的  机器代码，可执行指令  就存储在这里，  这里的代码  是只读的。
全局区(静态区)

全局变量  和  静态变量  存储在这里。  初始化的全局变量  和  静态变量  在  一块区域  (.data)，  未初始化的  全局变量  和  未初始化的  静态变量  在  相邻的  另一块区域(.bbs)。程序结束后，由  系统释放。

常量区
常量字符串  放在这里，程序结束后，由系统进行释放。

。看其他的，全局  和  常量  是合并在一起的。

declval

============================

https://juejin.cn/post/6844903497075294216

# 右值

函数返回右值

```C++
#include <string>
#include <stdio.h>

int g_var = 8;
int& returnALvalue() {
   return g_var; //here we return a left value
}

int returnARvalue() {
   return g_var; //here we return a r-value
}

int main() {
   printf("%d", returnALvalue()++); // g_var += 1;
   printf("%d", returnARvalue());
}
```

输出
8
9

。。本来觉得毫无意义，毕竟  我只会  用  int& ans  作为方法参数，来  获得  结果。  不会使用  int&  作为返回值。  。  但是后来想到了  单例，工厂。  这样就有一点意义了，  当然  int& ans  也是可以的。

。。而且文章也说了，这种  函数返回左值  只是为了演示，  现实中不要这样写。
。。但是  感觉  单例  工厂，  会用到吧。

在右值引用(&&)  发明之前，右值就已经可以影响代码逻辑了。

比如：
const std::string& name = "rvalue";
没有问题
但
std::string& name = "rvalue"; // use a left reference for a rvalue
是编译不了的

error: non-const lvalue reference to type 'std::string' (aka 'basic_string<char, char_traits<char>, allocator<char> >') cannot bind to a value of unrelated type 'const char [7]'

说明编译器  强制  我们使用  常量ref  来  ref  右值。

来看一个更有趣的：
```C++
#include <stdio.h>
#include <string>

void print(const std::string& name) {
    printf("rvalue detected:%s\n", name.c_str());
}

void print(std::string& name) {
    printf("lvalue detected:%s\n", name.c_str());
}

int main() {
    std::string name = "lvalue";
    print(name); //compiler can detect the right function for lvalue
    print("rvalue"); // likewise for rvalue
}
```
输出
lvalue detected:lvalue
rvalue detected:rvalue

说明这个差异，足以让  编译器决定  重载函数。

但右值  不一定是  常量
```C++
#include <stdio.h>
#include <string>

void print(const std::string& name) {
  printf(“const value detected:%s\n”, name.c_str());
}

void print(std::string& name) {
  printf(“lvalue detected%s\n”, name.c_str());
}

void print(std::string&& name) {
  printf(“rvalue detected:%s\n”, name.c_str());
}

int main() {
  std::string name = “lvalue”;
  const std::string cname = “cvalue”;

  print(name);
  print(cname);
  print("rvalue");
}
```

输出：
lvalue detected:lvalue
const value detected:cvalue
rvalue detected:rvalue

说明  如果有专门的  右值重载  的函数时，  右值会使用  专门的函数，而不是  使用  接受常量引用的  函数。  所以  &&  可以更加细化  右值  和  常量ref。

## 右值可以避免不必要的深拷贝

&&  解决了什么问题
解决了：当参数是右值时，不必要的深拷贝。

&&  用来区分右值，这样  当  这个右值  是一个构造函数或  赋值函数的参数   且  对应的类  包含指针，并指向了一个  动态分配的资源(内存)时，  就可以在  函数内  避免  深拷贝。

```C++
#include <stdio.h>
#include <string>
#include <algorithm>

using namespace std;

class ResourceOwner {
public:
  ResourceOwner(const char res[]) {
    theResource = new string(res);
  }

  ResourceOwner(const ResourceOwner& other) {
    printf("copy %s\n", other.theResource->c_str());
    theResource = new string(other.theResource->c_str());
  }

  ResourceOwner& operator=(const ResourceOwner& other) {
    ResourceOwner tmp(other);
    swap(theResource, tmp.theResource);
    printf("assign %s\n", other.theResource->c_str());
  }

  ~ResourceOwner() {
    if (theResource) {
      printf("destructor %s\n", theResource->c_str());
      delete theResource;
    }
  }
private:
  string* theResource;
};

void testCopy() {
 // case 1
  printf("=====start testCopy()=====\n");
  ResourceOwner res1("res1");
  ResourceOwner res2 = res1;
  //copy res1
  printf("=====destructors for stack vars, ignore=====\n");
}

void testAssign() {
 // case 2
  printf("=====start testAssign()=====\n");
  ResourceOwner res1("res1");
  ResourceOwner res2("res2");
  res2 = res1;
 //copy res1, assign res1, destrctor res2
  printf("=====destructors for stack vars, ignore=====\n");
}

void testRValue() {
 // case 3
  printf("=====start testRValue()=====\n");
  ResourceOwner res2("res2");
  res2 = ResourceOwner("res1");
 //copy res1, assign res1, destructor res2, destructor res1
  printf("=====destructors for stack vars, ignore=====\n");
}

int main() {
  testCopy();
  testAssign();
  testRValue();
}
```

=====start testCopy()=====copy res1=====destructors for stack vars, ignore=====destructor res1destructor res1=====start testAssign()=====copy res1assign res1destructor res2=====destructors for stack vars, ignore=====destructor res1destructor res1=====start testRValue()=====copy res1assign res1destructor res2destructor res1=====destructors for stack vars, ignore=====destructor res1

## Move

如果参数是右值，就不拷贝，而是直接搬资源，我们把  赋值函数  用  右值引用重载下：
ResourceOwner& operator=(ResourceOwner&& other) {
  theResource = other.theResource;
  other.theResource = NULL;
}

std::move
当  我们知道一个参数是右值，但  编译器不知道时，  这个参数  是调不到  move  重载函数的。

一个常见情况是，在ResourceOwner  上再加一层类  ResourceHolder

```C++
#include <string>
#include <algorithm>

using namespace std;

class ResourceOwner {
public:
  ResourceOwner(const char res[]) {
    theResource = new string(res);
  }

  ResourceOwner(const ResourceOwner& other) {
    printf(“copy %s\n”, other.theResource->c_str());
    theResource = new string(other.theResource->c_str());
  }

++ResourceOwner(ResourceOwner&& other) {
++ printf(“move cons %s\n”, other.theResource->c_str());
++ theResource = other.theResource;
++ other.theResource = NULL;
++}

  ResourceOwner& operator=(const ResourceOwner& other) {
    ResourceOwner tmp(other);
    swap(theResource, tmp.theResource);
    printf(“assign %s\n”, other.theResource->c_str());
  }

++ResourceOwner& operator=(ResourceOwner&& other) {
++ printf(“move assign %s\n”, other.theResource->c_str());
++ theResource = other.theResource;
++ other.theResource = NULL;
++}

  ~ResourceOwner() {
    if (theResource) {
      printf(“destructor %s\n”, theResource->c_str());
      delete theResource;
    }
  }

private:
  string* theResource;
};

class ResourceHolder {
……
ResourceHolder& operator=(ResourceHolder&& other) {
  printf(“move assign %s\n”, other.theResource->c_str());
  resOwner = other.resOwner;
}
……
private:
  ResourceOwner resOwner;
}
```

在ResourceHolder  的move赋值函数中，  我们想调用的是  move  赋值函数，因为  右值的成员也是  右值，但是
resOwner = other.resOwner
其实调用了  普通赋值函数，还是做了深拷贝。

解决方案，我们使用  std::move  把这个变量  强制转为  右值

```C++
ResourceHolder& operator=(ResourceHolder&& other) {
  printf(“move assign %s\n”, other.theResource->c_str());
  resOwner = std::move(other.resOwner);
}
```

我们知道  强转除了让编译器闭嘴，其实是会生成对应的机器码的  (在不开O的情况下比较容易观察到)。  这个机器码会把  变量  在不同大小的  寄存器里面  移来移去  来  真正完成  强转操作。

所以  std::move  也和  强转做了类似的操作吗？

```C++
int main() {
  ResourceOwner res(“res1”);
  asm(“nop”); // remeber me
  ResourceOwner && rvalue = std::move(res);
  asm(“nop”); // remeber me
}
```

编译它，然后用下面的命令  获得  汇编
```C++
clang++ -g -c -std=c++11 -stdlib=libc++ -Weverything move.cc
gobjdump -d -D move.o

0000000000000000 <_main>:
 0: 55 push %rbp
 1: 48 89 e5 mov %rsp,%rbp
 4: 48 83 ec 20 sub $0x20,%rsp
 8: 48 8d 7d f0 lea -0x10(%rbp),%rdi
 c: 48 8d 35 41 03 00 00 lea 0x341(%rip),%rsi # 354
 <GCC_except_table5+0x18>
 13: e8 00 00 00 00 callq
 18 <_main+0x18> 18: 90 nop // remember me
 19: 48 8d 75 f0 lea -0x10(%rbp),%rsi
 1d: 48 89 75 f8 mov %rsi,-0x8(%rbp)
 21: 48 8b 75 f8 mov -0x8(%rbp),%rsi
 25: 48 89 75 e8 mov %rsi,-0x18(%rbp)
 29: 90 nop // remember me
 2a: 48 8d 7d f0 lea -0x10(%rbp),%rdi
 2e: e8 00 00 00 00 callq 33 <_main+0x33>
 33: 31 c0 xor %eax,%eax
 35: 48 83 c4 20 add $0x20,%rsp
 39: 5d pop %rbp
 3a: c3 retq
 3b: 0f 1f 44 00 00 nopl 0x0(%rax,%rax,1)
```

2个nop中间  确实生成了一些机器码，但是这些机器码  貌似  什么都没有做，只是简单的把  一个变量的  地址  赋给  另一个而已，  并且，如果我们把  O (-O1  就够了)  打开，所有的  nop  中间的  机器码就  消失了。

clang++ -g -c -O1 -std=c++11 -stdlib=libc++ -Weverything move.cc
gobjdump -d -D move.o

如果把关键行  改成
ResourceOwner & rvalue = res;

除了变量的  相对偏移有变化，其实生成的机器码  是一样的。

说明std::move  是一个语法糖，没有什么实际的操作。

============================

# Lock

https://blog.csdn.net/t567g123/article/details/123674458
这个没有什么用，需要关注的。

C++  11
mutex，独占互斥量，不能递归，
timed_mutex，具有超时机制的独占互斥量，不能递归
recursive_mutex，可以递归的  mutex
recursive_timed_mutex，可以递归的

C++  14
shared_timed_mutex，  具有超时机制的  可共享互斥量

C++  17
shared_mutex

读写锁(std::shared_mutex)
相比  互斥锁，允许  更高的并行性。
当读写锁以  读模式  lock时，  它是以共享模式lock  的，当以写模式  lock时，是独占模式。

读写锁有3种状态：
读模式加锁状态
写模式加锁状态
不加锁状态

如果线程通过  lock  或  try_lock  获取  独占锁(写锁)，则  其他线程无法获得锁(包括共享的)。尝试获得读锁的线程  也会被阻塞
仅当  没有线程获取  独占锁时，共享锁(写锁)  才可以被  多个  线程获取  (通过  lock_shared, try_lock_shared)。
在一个线程内，同一时刻  只能获取  一个锁  (共享  或  独占)

自旋锁
是一个  busy-waiting  锁。
如果线程想要获得  一个被占用的  自旋锁，它会一直占用CPU，直到获取这个锁位置。
发生阻塞时，互斥锁可以让CPU  处理  其他任务，而自旋锁  让CPU  一直不断循环  请求  获取这个锁。

条件锁
就是所谓的  条件变量。  condition_variable

某个线程  因为某个条件未满足时  可以使用  条件变量  使得  该程序处于  阻塞状态，一条条件满足，以  信号量的方式  唤醒  一个  因为这个条件  而被阻塞的  线程。

最常见的就是  线程池中，起初  没有任务时，任务队列为空，此时  线程池中的线程  因为  任务队列为空  这个条件  而处于  阻塞状态。一旦有任务进来，就会以  信号量的  方式  唤醒一个线程  来处理这个任务。

。。需要关注才能读其他的。

-----------------------------------------------------------
https://blog.csdn.net/g498912529/article/details/125897199

# 互斥锁Mutex

某一时刻，只有一个线程可以获取互斥锁，在释放互斥锁之前，其他线程  都不能获取该  互斥锁。  如果其他线程想要获取  这个互斥锁，那么这个线程之内以  阻塞的方式进行  等待。

互斥锁用于控制多个线程  对它们之间  共享资源  互斥访问的  一个  信号量。

std::mutex  来创建  互斥量，  lock()  来加锁，unlock()  来解锁。

## 递归锁 recursive_mutex
允许  同一个线程  在  未释放  其拥有的  锁时  反复  对该锁  进行  加锁。

递归锁，也称为  可重入锁，同一个线程  在不解锁的  情况下，可以多次  获取  锁定  同一个  递归锁，而不会产生死锁。
非递归锁，不可重入锁，不解锁的情况下，  同一个线程  多次  获取  同一个  非递归锁时，会产生死锁。

注意
windows  下  互斥量  和  临界区(关键段)  是递归锁

linux  下  互斥量  pthread_mutex_t  是非递归锁，但是可以在  创建互斥量时  设置  PTHREAD_MUTEX_RECURSIVE  属性  将它设置为  递归锁。

尽量不要使用  递归锁，即同一个线程中，尽量不要  重复  锁定同一个锁，读锁也不可以。  递归锁  用起来  固然简单，但是会  隐藏  某些代码问题。比如  调用函数  和  被调用函数  以为自己拿到了锁，  都在修改同一个对象，这就很容易出现问题。  因此  在能使用  非递归锁的情况下，尽量使用  非递归锁，因为  死锁  相对来说，更容易通过测试发现。

。。？无法理解，没有问题的啊，都锁了，而且是一个线程，就是线性执行了，都改又怎么啦。

## 条件锁 condition_variable

某个线程  在条件没有满足是  ，可以使用  条件变量  来使得  程序处于  阻塞状态。  一旦条件满足，就以  信号量的方式  唤醒  一个  因为该条件  而阻塞的  线程。

pthread_cond_wait  函数的语义相当于：  先解锁  互斥锁，然后  以阻塞方式等待  条件变量的信号，收到信号后，会对  互斥锁  加锁。

## 自旋锁 spinlock_t
使用  硬件提供的  swap  之类  或  test_and_set  指令实现。

## 读写锁
适用于  大量读，少量写
是非递归锁
注意，按照  POSIX  标准，在线程  申请读锁并没有释放前，本线程申请写锁  是成功的，但运行后逻辑结果  未知。

## 避免死锁 lock_guard

std::lock_guard  只有构造和析构函数，  简单来说，  调用构造函数时，自动调用  传入的对象的  lock  函数，  调用析构函数时，  自动调用  unlock()。(这就是所谓的  RAII)

使用  std::mutex  和  std::lock_guard  搭配使用，避免死锁的发生。

-----------------------------------------
https://www.jianshu.com/p/0abdfc4d4fd7

# 互斥锁 mutex

`<mutex>`

lock(),try_lock(),unlock()

try_lock  行为：
1. 没有任何线程占用锁，则使用lock()，返回true
2. 其他线程占用了锁，返回false
3. 本线程占用了锁，行为未定义

unlock  要和  lock  对应，没有lock  的情况下  调用unlock，行为未定义。

## lock_guard
建议使用lock_guard  代替  mutex.lock/unlock。
lock_guard  在构造器中  使用  mutex.lock，析构器中使用  mutex.unlock。
为了避免中途  异常退出  导致mutex  没有  unlock。
{
std::lock_guard lg(m_mutex);
//…
} //  出作用域后，自动  析构，自动释放锁。

## unique_lock

lock_guard  只在析构的时候释放锁，不提供  unlock  ，不够灵活。
unique_lock  提供  lock  和  unlock。

```C++
std::unique_lock<std::mutex> guard(_mu);
// do sth.
guard.unlock();      //  临时解锁
// do sth
guard.lock();       //  继续上锁
// do sth
//  退出时，自动析构，解锁。
```


lock_guard  和  unque_lock  构造器中的  部分参数：
defer_lock_t：初始化时，不进行  默认的  加锁操作，后续  需要上锁时  再lock
try_to_lock_t：初始化时，使用  try_lock
adopt_lock_t：初始化时，假定  当前线程  已经上锁成功，不再调用  lock()

## 条件锁
`<condition_variable>`

std::condition_variable (只和  std::mutex  一起)

std::condition_variable_any (符合  类似互斥元  的最低标准的  任何东西  一起工作)

成员方法：wait,wait_for,wait_until,notify_one,notify_all

条件锁，即条件变量，某个线程  因为  某个条件未满足时  可以使用  条件变量  使  该线程  处于阻塞状态，一旦条件满足，以"信号量"  的方式  唤醒  一个  因为该条件而被阻塞的  线程。  常和  互斥锁配置使用。

void wait(unique_lock<mutex>& lck, Predicate pred);

pred:   如果pred  返回false，方法只能block。  notification  只有在  pred变成true  的时候  才能  unblock。(这个对于  检查防止  spurious wake-up calls(虚假唤醒)  特别有用)。

## 自旋锁
获取锁失败，不会导致阻塞，而不是一直尝试获取锁。
适用于，  被持有时间短，线程不希望  在  重新调度上  花过多的时间的情况。

## 读写锁
读线程可以同时访问，但是  只允许一个线程  写入。

写锁：如果没有读者，也没有写者，则获取锁；否则等待，直到没有读者，没有写者。
读者：读者使用读锁，如果没有写者，则立刻获得读作，否则等待，直到没有写者。

## 递归锁
mutex  分为  递归锁  和  非递归锁。  可递归锁  也被称为  可重入锁。非递归锁也被称为  不可重入锁。
区别是，同一个线程  可以多次获取  同一个递归锁，不会死锁。   一个线程多次获取  同一个非递归锁，则会死锁。

============================

# concurrency lock
https://blog.csdn.net/weixin_44477424/article/details/130694304

## lock_guard

C++11 简单的 RAII。
不支持 手动 lock 和 unlock， 不支持 条件变量

## unique_lock
C++11 更灵活的锁
支持 手动 lock 和 unlock， 以及 和条件变量 一起使用。(是 lock_guard 的 进阶版)

还支持 延迟锁定，尝试锁定，可转移的锁所有权


## shared_lock
C++14
用于 共享互斥量，如 std::shared_mutex, std::shared_timed_mutex
允许多个线程 同时读取 共享数据。
支持 手动lock，unlock，尝试lock

## scoped_lock
C++17
同时锁定多个互斥量，以避免死锁。
不支持 手动lock，unlock， 不支持 条件变量

主要用于 为 同时锁定 多个互斥量 提供 简单，安全的 解决方案。


## recursive_mutex
场景：类的不同成员函数之间 相互调用。类中存在 互斥量 保护成员数据， 所以 fun1 需要获得锁，然后 操作 成员数据， fun2 也需要锁。 互相调用时，会导致 未定义行为，导致死锁。

里面包含了一个 计数器，当计数器为0时，释放互斥锁。




============================

# RAII

resource  acquisition is initialization
资源获取时  就是初始化。

C++标准  保证，已构造的对象  最终会被销毁，即它的析构函数最终会被调用。

RAII的做法，使用一个对象，在  构造时，获得资源，在这个对象的  生命周期  中，对资源的访问始终有效，  最后在  对象析构时  释放资源。

根据RAII  对资源的  所有权  可分为：  常性类型  和  变性类型，  代表着分别是  boost::shared_ptr<>，  std::auto_ptr<>。

从所管资源的  初始化位置上  可以分为  外部初始化类型  和  内部初始化类型。

常性类型是指  获取资源的地点是  构造函数，释放点  是析构函数。在这2点之间，任何对该  RAII  类型实例的  操作  都不应该  从它手里  拿走资源的  所有权。

变性类型是指  中途被设置为  接管另一个资源，或者  被置为  不拥有任何资源。

外部初始化是指  资源在外部被创建，然后传给  RAII实例的  构造函数，RAII  接管了  所有权。  boost::shared_ptr<>  ，std::auto_ptr<>  都是这个类型。

与之相对的就是  内部初始化类型。

常性内部初始化  的类型是  最纯粹的  RAII形式。

============================

============================
# Template

https://blog.csdn.net/weixin_62700590/article/details/125227121

模板分为  函数模板  和类模板
从模板联想到  泛型编程：编写与类型无关的  通用代码，是代码复用的一种手段。

函数重载，处理  2个  int，double，char  的  swap，  需要写  3个函数。
模板，只需要一个：
template<typename T>
void swap(T& left, T& right) {
T temp = left;
left = right;
right = temp;
}

告诉编译器一个模板，让编译器根据不同的类型  利用该模板生成代码。

## 函数模板
函数模板代表了一个函数家族，该函数模板与类型无关，在使用时被参数化，根据实参类型产生  函数的特定类型版本

typename  是用于  定义  模板参数的  关键字，也可以用  class。   但不能用  struct

函数模板是一个蓝图，它本身并不是函数，是  编译器  用于  产生  特定具体类型  函数的  模具。  所以  其实  模板  就是  将本来  应该我们做的  重复的事情  交给  编译器。

在编译阶段，如果代码使用了  模板函数，则  编译器  会根据  传入的  实参  的类型  来  推导  生成  对应类型的  函数，以供调用。

用不同类型的参数  使用  函数模板时，称为函数模板的  实例化。
模板参数实例化  分为：  隐式实例化  和  显式实例化。
隐式实例化：让编译器根据实参  推导  模板参数的  实际类型
显式实例化：在函数名  后面的  <>  中指定  模板参数的  实际类型

```C++
int a = 1;
double d = 2.0;

swap(a, b);       // error,没有  (int, double)  的函数
swap(a, (int) b);
swap((double) a, b);
swap<int>(a, b);
swap<double>(a, b);
```

要么自己强转，要么使用  显式实例化

### 模板参数的匹配原则
一个非模板函数  可以和  一个同名的  函数模板  同时存在，而且  该  函数模板  还可以  被实例化  为  这个  非函数模板。

swap(1, 2);   //  使用  非函数模板(如果有的话)
swap<int>(1, 2);   //  使用  编译器  通过函数模板  实例化出来的  函数

对于  非模板函数  和  同名模板函数，  调用时  优先  匹配  非模板函数。

模板函数不允许  自动类型转换，但是  普通函数  可以进行  自动类型转换。

声明定义分离
```C++
//  声明
template<typename T>
void swap(T& left, T& right);
```
```C++
//  实现
template<typename T>
void swap(T& left, T& right) {
    T temp = left;
    left = right;
    right = temp;
}
```

## 类模板
```C++
template<class T1, class T2, .. , class Tn>
class  类模板名
{
//  类内成员定义
}
```

这里就  习惯用上  class，  函数模板是  typename。

类模板的实例化
和函数模板实例化不同，  类模板实例化  需要在  类模板  名字  后面  跟<>，然后将实例化  的类型放在  <>  中。
类模板名字  不是真正的类，实例化的  结果才是真正的类。

```C++
template<class T1, class T2>
class Student
{
public:
    void Init(const string& name, const string& sex, const T1& age, const T2& height)
    {
        _age = age;
        _height = height;
        _name = name;
        _sex = sex;
    }
    void Print()
    {
        cout<<_name<<_sex<<_age<<_height<<endl;
    }
private:
    string _name;
    string _sex;
    T1 _age;
    T2 _height;
}

int main()
{
    Student<int, double> st1;
    Student<double, int> st2;
    st1.Init("aa", "male", 11.1, 111.1);
    st2.Init("bb", "female", 22.2, 222.2);
    st1.Print();
    st2.Print();
}
```

会有警告。

使用构造器
```C++
template<class T1, class T2>
class Student
{
public:
    Student(const string& name, const string& sex, const T1& age, const T2& height)
    {
        _age = age;
        _height = height;
        _name = name;
        _sex = sex;
    }
}

int main()
{
    Student<int, double> st1("aa", "male", 11.1, 111.1);
    Student<double, int> st2("bb", "female", 22.2, 222.2);
}
```

类模板  中函数放在  类外进行定义时  需要增加模板参数列表

```C++
template<class T1, class T2>
class Student
{
public:
    // …
    ~Student();         //  类内声明
}

template<class T1, class T2>
Student<T1, T2>::~Student()
{
//  实现
}
```

### 模板分离编译

分离编译：一个程序(项目)  由若干个  源文件共同实现，而每个源文件  单独编译生成目标文件，最后将所有  目标文件  链接起来  形成  单一的  可执行文件  的过程

模板的分离编译
以下场景中，模板的声明  和定义  分离开，在头文件中进行  声明，源文件中完成定义：
```C++
// a.h
template<class T>
T Add(const T& left, const T& right);
```

```C++
// a.cpp
template<class T>
T Add(const T& left, const T& right)
{
    return left + right;
}
```
```C++
// main.cpp
#include"a.h"
int main()
{
    Add(1, 2);
    Add(1.0, 2.0);
    return 0;
}
```

会出现链接错误

原因：程序运行  需要  预处理，编译汇编  和  链接。对于头文件的内容，a.cpp  模板都是空的，因为  编译器不知道  T  是什么  (模板是在  编译阶段处理，不是  预处理)

main.cpp  只有声明，  所以  call  是不知道  地址的。
然后链接的时候，因为  a.cpp  是没有生成  对应  的函数的  (因为之前  T  不知道)，  所以链接的时候  会发生链接错误

简单来说，就是  编译的时候  不知道T  是什么，所以没有生成对应的  汇编代码，导致  main.cpp  里面  想要调用的时候  找不到  对应的函数，从而发生  链接错误。

解决
1. 放到  .hpp  的文件中，是  .h  和  .cpp  的合体，寓意更好。  直接  .h  也可以
2. 对症下药，  因为  是  只有声明  没有  定义，  所以  直接在  a.cpp  中  显式实例化。

```C++
template
int Add<int>(int& left, int& right);
template
double Add<double>(double& left, double& right);
```

。。  .hpp

### 缺省值，返回值
缺省值  必须  从右往左  缺省  (类比函数)，  因为  实参  是从左往右传的。

```C++
template<class K, class V = char>
void func()
{
    cout<<sizeof(K)<<sizeof(V)<<endl;
}
```
。。  sizeof(  类名  )   也可以？

也可以使用模板作为返回值
```C++
template<class T = char>
T* func(int n)
{
    return new T[n];
}
```

总结
优点
模板复用了代码，节省了资源，更快的迭代开发，C++  的标准模板库  因此而产生
增强了代码的灵活性

缺点
模板导致  代码膨胀，编译时间变长
出现  模板  编译错误时，错误信息非常凌乱，难以定位问题

----------------------
https://www.zhihu.com/question/37990298

# C++ 为什么有时候必须额外写 template？

`this->x = result.template get<0>();`

get方法是模板，有时必须在前面加上  template，否则  认不出  get  是模板。

template  是用来消除歧义的，下面的代码
```C++
template<class T>
int f(T& x) {
    return x.template convert<3>(pi);
}
```

如果没有template，  则  `return x.convert<3>(pi);`  可能被理解为  `return ((x.convert) < 3) > (pi);`

使用  template  来说明  convert  不是一个  数据成员，而是一个模板函数。

下面是标准
使用template  的规则

当成员  模板  特化的  名字  出现在  一个  后缀表达式  中的  .  或  ->  之后，  或出现一个  限定标识中的  嵌套的  名字修饰符之后，并且  后缀表达式  或  限定标识  显式  依赖于  一个模板参数时，  成员模板  名字  必须  加  template  关键字  作为前缀，否则  该名字  就被  假定为  一个  非模板的名字。

如果后缀表达式  或  限定标识  不是  出现在  一个模板的  作用域时，成员模板的  名字  就不应该加上  template  关键字  作为前缀。

## 必须使用  template  的场合
在通过  .  ->  ::   限定的  依赖名  访问  成员模板  之前，  template  关键字  必不可少。
```C++
template<class T>
void f(T& x, T& y) {
    int n = x.template convert<int>();
    int m = y->template convert<int>();
}
template<class T> struct other;
template<class T>
struct dirived : other <T>::template base<int> {};
```

-----------------------
https://blog.csdn.net/qq_54169998/article/details/121001056

# C++ 类模板（template）详解

为什么需要类模板

类模板  和  函数模板  的定义  和  使用  类似，有时，有  2个  或  多个类，其功能是相同的，  仅仅是  数据类型不同，我们可以通过  如下语句  声明  一个  类模板：

```C++
template <typename T>
class A
{
public:
    A(T t){
        this->t = t;
    }

    T& getT(){
        return t;
    }

public:
    T t;
};
```

## 类模板定义
由  模板说明  和  类说明  构成
template <类型形式参数表>
类声明

例如
```C++
template  <typename Type>
class ClassName{
public:
    //ClassName 的成员函数
private:
    Type DataMember;
}
```

### 单个类模板的使用
```C++
#include <iostream>
using namespace std;

template <typename T>
class A
{
public:
    //函数的参数列表使用虚拟类型
    A(T t = 0){
        this->t = t;
    }
    //成员函数返回值使用虚拟类型
    T& getT(){
        return t;
    }

private:
    //成员变量使用虚拟类型
    T t;
};

void printA(A<int>& a) {
    cout << a.getT() << endl;
}

int main(void) {
    //1.模板类定义类对象，必须显示指定类型
    //2.模板种如果使用了构造函数，则遵守以前的类的构造函数的调用规则
    A<int>  a(666);
    cout << a.getT() << endl;

    //模板类做为函数参数
    printA(a);
    system("pause");
    return 0;
}
```

### 继承中类模板的使用
#### 父类是一般类，子类是模板类
和普通继承类似
```C++
class A {
public:
    A(int temp = 0) {
        this->temp = temp;
    }
    ~A(){}
private:
    int temp;
};

template <typename T>
class B :public A{
public:
    B(T t = 0) :A(666) {
        this->t = t;
    }
    ~B(){}
private:
    T t;
};
```

#### 子类是一般类，父类是模板类
继承时，必须在  子类里  实例化  父类的  类型参数

```C++
template <typename T>
class A {
public:
    A(T t = 0) {
        this->t = t;
    }
    ~A(){}
private:
    T t;
};

class B:public A<int> {
public:
    //也可以不显示指定，直接A(666)
    B(int temp = 0):A<int>(666) {
        this->temp = temp;
    }
    ~B() {}
private:
    int temp;
};
```

#### 父类和子类  都是  模板类
子类的  虚拟的  类型  可以传递到父类中。

--------------------------------------

[https://learn.microsoft.com/zh-cn/cpp/cpp/templates-cpp?view=msvc-170](https://learn.microsoft.com/zh-cn/cpp/cpp/templates-cpp?view=msvc-170)

# 模板 (C++)

模板是  C++  中  泛型编程的基础。
作为强类型语言，C++要求  所有变量  都具有  特定类型，  有程序员  显示声明  或  编译器推导。

但是，许多数据结构  和算法，  无论在哪种类型上  操作，看起来都是相同的。  使用模板可以定义  类  或函数的  操作，并让用户  指定这些操作  应  处理的具体类型

## 定义和使用模板
模板是基于用户为  模板参数  提供的  参数  在编译时  生成普通类型  或  函数的  构造
```C++
template <typename T>
T minimum(const T& lhs, const T& rhs)
{
    return lhs < rhs ? lhs : rhs;
}
```

类型参数  可以随意命名，最常  使用  单个  大写字母。
T  是模板参数，关键字  typename  表示  此参数  是类型的  占位符。

调用函数时，  编译器会将  每个  T  实例  替换为  由  用户指定  或  编译器  推导的  具体类型参数。  编译器  从模板生成  类  或函数的  过程  称为  模板实例化

用户可以声明  专用于  int  的模板实例。假设  get_a,get_b  是返回  int  的函数
```C++
int a = get_a();
int b = get_b();
int i = minimum<int>(a, b);
```

但是，由于这是一个  函数模板，编译器可以从  参数  a  和  b  中  推导出  类型，  因此可以像  普通函数  一样  调用它：
`int i = minimum(a, b);`

当编译器遇到最后一个语句时，  它会生成一个  新函数，在该函数中，T  在  模板中的  每个匹配项  都被替换为  int
```C++
int minimum(const int& lhs, const int& rhs)
{
    return lhs < rhs ? lhs : rhs;
}
```

## 类型参数

在上面的  minimum  模板中，请注意，在将  类型参数  T  用于  函数调用参数  (在这些参数中  会添加  const  和  引用  限定符)  之前，  不会以  任何方式  对其  进行限定。

类型参数的  数量没有  限制，以逗号分隔
`template <typename T, typename U, typename V> class Foo{};`

模板参数中，class  等于  typename
`template <class T, class U, class V> class Foo{};`


### 省略号运算符-任意数量

可以用省略号运算符  定义  采用任意数量(0或多个)  的  类型参数  的模板
```C++
template<typename... Arguments> class vtclass;

vtclass< > vtinstance1;
vtclass<int> vtinstance2;
vtclass<float, bool> vtinstance3;
```

任何内置类型  或  用户定义的  类型都可以用作  类型参数。

使用模板时  的主要限制  是  类型参数  必须支持  模板中应用到  类型上的  操作。

没有任何要求  规定  特定模板的类型参数  都属于  同一个  对象层次结构，  尽管可以  定义强制  实施  此类限制的模板。

可以将  面向对象的技巧  与  模板  相结合，例如，可以将  `Derived*`  存储在  `vector<Base*>`  中，  注意，自变量必须是  指针

## 非类型参数/值参数

例如，可以提供  常量整型值  来指定  数组的长度。
例如，在下面的示例中，  它类似于  标准库中的  std::array  类
```C++
template<typename T, size_t L>
class MyArray
{
    T arr[L];
public:
    MyArray() { ... }
};
```
。。。能默认值吧？

size_t  值在编译时  作为模板参数  传入，必须是  const  或  constexpr  表达式。

`MyArray<MyClass*, 10> arr;`

其他类型的值(包括指针和引用)  可以作为  非类型参数传入。例如，可以传入  指向  函数  或  函数对象的  指针，以自定义模板代码中的  某些操作。

非类型模板参数的类型推导
在VS 2017  及更高版本中，在  /std::c++17  模式或更高版本中，编译器会推导  使用  auto  声明的  非类型模板参数的类型：

```C++
template <auto x> constexpr auto constant = x;

auto v1 = constant<5>;      // v1 == 5, decltype(v1) is int
auto v2 = constant<true>;   // v2 == true, decltype(v2) is bool
auto v3 = constant<'a'>;    // v3 == 'a', decltype(v3) is char
```

## 模板作为模板参数
MyClass2  有  2个模板参数：  类型名称参数  T  和  模板参数  Arr

```C++
template<typename T, template<typename  U, int I> class Arr>
class MyClass2
{
    T t; //OK
    Arr<T, 10> a;
     U u; //Error. U not in scope
};
```

由于  Arr  参数本身没有正文，所以  不需要  参数名称。事实上，从  MyClass2  的正文中  引用  Arr  的类型名称  或  类型参数名称  是错误的。  因此，可以省略  Arr  的类型参数名称：

```C++
template<typename T, template<typename, int> class Arr>
class MyClass2
{
    T t; //OK
    Arr<T, 10> a;
};
```

## 默认模板自变量
类和函数模板可以具有  默认自变量。
如果模板具有  默认自变量，则可以在  使用时  不指定  该  自变量。
例如  std::vector  模板  有一个  用于分配器的  默认自变量
`template <class T, class Allocator = allocator<T>> class vector;`

多模板参数时，  第一个默认参数  后  所有的参数  必须  具有  默认参数
使用  参数  都是  默认值的  模板时，  请使用  空尖括号

```C++
template<typename A = int, typename B = double>
class Bar
{
    //...
};
...
int main()
{
    Bar<> bar; // use all default type arguments
}
```

## 模板特殊化
在某些情况下，模板不可能  或  不需要  为任何  类型  都定义完全相同的  代码。
例如，你可能  希望  定义   在类型参数是  指针，std::wstring  或  派生自特定基类  的  类型  时  才  执行的  代码路径。
在这种情况下，  可以为  该特定类型  定义  模板的  专用化。
当用户使用  该类型  对模板  进行实例化时，  编译器使用  该专用化  来生成  类。
而对于其他所有类型，编译器  会选择  更常规的  模板。
如果专用化中  的所有参数  都是专用的，则称为  "完整专用化"。  如果只有一些  参数  是专用的，  则称为  "部分专用化"。

```C++
template <typename K, typename V>
class MyMap{/*...*/};

// partial specialization for string keys
template<typename V>
class MyMap<string, V> {/*...*/};
...
MyMap<int, MyClass> classes; // uses original template
MyMap<string, MyClass> classes2; // uses the partial specialization
```

只要每个专用类型参数是唯一的，模板就可以具有任意数量的专用化。 只有类模板可能是部分专用。 模板的所有完整专用化和部分专用化都必须在与原始模板相同的命名空间中声明。

```C++
template <class T> struct PTS {
   enum {
      IsPointer = 0,
      IsPointerToDataMember = 0
   };
};

template <class T> struct PTS<T*> {
   enum {
      IsPointer = 1,
      IsPointerToDataMember = 0
   };
};

template <class T, class U> struct PTS<T U::*> {
   enum {
      IsPointer = 0,
      IsPointerToDataMember = 1
   };
};

struct S{};

int main() {
   printf_s("PTS<S>::IsPointer == %d \nPTS<S>::IsPointerToDataMember == %d\n",
           PTS<S>::IsPointer, PTS<S>:: IsPointerToDataMember);
   printf_s("PTS<S*>::IsPointer == %d \nPTS<S*>::IsPointerToDataMember == %d\n"
           , PTS<S*>::IsPointer, PTS<S*>:: IsPointerToDataMember);
   printf_s("PTS<int S::*>::IsPointer == %d \nPTS"
           "<int S::*>::IsPointerToDataMember == %d\n",
           PTS<int S::*>::IsPointer, PTS<int S::*>::
           IsPointerToDataMember);
}
```

============================

https://www.jianshu.com/p/d09373b83f86

# 模板特化、类型萃取

类型萃取  解决的问题：
在编译器判断类型T  是否为  class  类
判断某个  class T  是否有指定的成员变量  或  成员函数

要了解  类型萃取，要先学习  模板特化

有时为了需要，针对特定的类型，需要对模板进行特化，也就是所谓的特殊处理。比如有以下代码

```C++
template<typename C>
int test(C c) {
    cout<<"for all type"<<endl;
    return c.size();
}
```

我们写了一个模板函数test,如果我们在main函数中执行如下语句
```C++
int main() {
    string s = "sss";
    test<string>(s);//正确,输出“for all type”
    test<int>(1);    //报错，int型没有size()
    return 0;
}
```

显然test<int>(1); 会报错，因为int型并没有size()方法，此时我们再写一个也叫test的函数：
```C++
//为int提供的特化版本test函数
template<typename C>
int test(int i) {
    cout<<"for int type"<<endl;
    return 0;
}
```
。。`template<typename C>`  还在，只不过  方法中的  C  被  真实的类型代替了。
。。这种，有点怪，主要是  模板的声明还在，  但是  没有用到  C。

。。不过  移除  template  的声明，  应该不行，  移除后  就是  普通函数，  优先级  应该比  template  。。。  擦，  移除应该也可以的，  普通函数的  优先级  比  template  高。

。。但是  为什么不移除，而是  选择  一种  新的术语，模板特化  呢。

我们将此称为test函数的int型特化版本，
如果test()方法接收到int类型的参数会调用该特化版本

我们把这叫做模板特化，告诉编译器对于某种类型进行特殊输处理

判断类型 T 是否是一个class
我们想在编译期间判断一个类型T是否是一个类，套用 SFINAE 原则，我们需要两个返回不同值的模板函数
当传入  class  作为模板实参时  匹配到  其中一个  函数模板
而  传入其他  实参时  匹配到  另一个。
这样  我们就可以根据  返回值  将  class  和其他类型  分开。

```C++
template<typename T>
class IsClass {
  private:
    typedef char One;
    typedef struct { char a[2]; } Two;
    template<typename C> static One test(int C::*);
    template<typename C> static Two test(...);
  public:
    static bool value ;//= sizeof(IsClass<T>::test<T>(NULL)) == 1;
};
template<typename T>
bool IsClass<T>::value = sizeof(IsClass<T>::test<T>(NULL)) == 1;
```

int C::* 表示指向class C的成员的指针
sizeof ()操作符可以在编译期间返回指定的类型占用的空间大小

随便定义一个类
```C++
class Test {
};

int main() {
  bool is_class = IsClass<Test>::value;
  std::cout << is_class << value;  //Test是一个class，输出1
  bool is_class2 = IsClass<int>::value;
  std::cout << is_class2 << value;  //int不是一个类，输出0
}
```

如果T不是一个类，比如T是一个int型，在计算value时会调用返回Two的test函。因为返回One的参数是 int C::*,int型不具有指向其函数的指针，编译器匹配不上。

如果T是一个类，能够匹配上返回One的test函数，NULL能够被推导成int T::*。

有同学会问，T是一个类，也能匹配上返回One的test方法，怎么在二者之间做选择呢？《C++ templates》书中说道：

overload resolution prefers the conversion from zero to a null pointer constant over binding an argument to an ellipsis parameter (ellipsis parameters are the weakest kind of binding from an overload resolution perspective).

省略号参数是最弱的绑定类型，所以最后只有当其他都匹配不上才会去匹配参数为省略号的函数

判断class T 是否有某个成员函数
假设有Man和Woman两个类：
```C++
class Man {
public:
    void man_do(){}
    static void sayhi() {
        cout << "im a man" << endl;
    }
};

class Woman {
public:
    void woman_do(){}
    static void sayhi() {
        cout << "im a woman" << endl;
    }
};

int main()
{
    Say<Man>();
    Say<Woman>();
    return 0;
}
```

我们希望`Say<T>()`函数能够通过判断T类型有没有一个叫做"man_do（）"的方法，如果有，就执行man::sayhi(),没有就执行woman::sayhi()。

先定义Say函数，Say()函数会调用Check结构体的type类的sayhi()方法

```C++
template<class P>
void Say() {
    Check<P>::type::sayhi();
}
```

再来看Check结构体和IF结构体的内容：

```C++
template <class P>
struct Check {
    typedef typename IF<Trait<P>::has_mando, Man, Woman>::type type;
};

template <bool C, class T, class E>
struct IF { typedef T type; };              //非特化IF

template <class T, class E>
struct IF<false,  T, E> { typedef E type; }; //标明了false，特化版本的IF
```

关键就在于传给IF第一个参数的`Trait<P>::has_mando`

```C++
template <class P, class... Args>
struct Trait {
    static const bool has_mando = HasMando<P>::value;
};

template <class T>
struct HasMando {
    typedef char yes[1];
    typedef char no[2];
    template <class U>  static no&  test(...);
    template <class U> static yes& test(decltype(&U::man_do));
    static const bool value = sizeof(test<T>(0)) == sizeof(yes);
};
```



---

https://zhuanlan.zhihu.com/p/547313994

## stl && traits

stl 的萃取器 traits 用了很多与 泛型编程(模板)有关的技巧。

traits 是 模板元编程中常用的技术

 C++11提供了模板元基础库`#include<type_traits>`，里面定义了很多 类型萃取常用的 类模板函数，比如， is_convertible, is_trivially_destructible。

- type_traits 提供了在编译期进行计算，判断，转换，查询 等功能
- type_traits 提供了编译期的 true 和false。


### type_traits

先看type_traits提供的编译期的true和false。

其实就是定义了一个class用来区分正负，它具有一个布尔型的value成员。可以通过::value来获取。

```C++
 template<class T,T v>
 struct integral_constant
 {
   static constexpr T value = v; 
   // static和constexpr缺一不可，表示在编译器确定 
 };
 ​
 template<bool b>
 using bool_constant = integral_constant<bool,b>; 
 // using代替typedef，c++11特性
 ​
 typedef bool_constant<true> true_type;
 typedef bool_constant<false> false_type;
```

is_same用来判断两个type是否相同。

```C++
 template<typename T, typename U>
 struct is_same : public false_type {};
 // 主模板
 ​
 template<typename _Tp>
 struct is_same<_Tp, _Tp> : public true_type {};
 // 偏特化
```
利用偏特化的原理，如果两个type相同，那么匹配到下面true_type的子类，否则就是匹配到上面的false_type的子类。


利用is_same来生成想要的代码

```C++
 template <class T, class U, bool = is_same<T,U>::value > // 默认模板参数
 struct Foo
 {
     void print(){
         printf("they are different type");
     }
 };

// 。。什么时候触发下面的？感觉所有的类都可以 通过 is_same<T, U> 啊，这样就不会有SFNIAE 了，就不会走下面的了。
// 不过， int 吗？ 我记得之前看到 特化，为了 原生类 做了单独的特化，但是忘记是为了什么了。。 我感觉 int 是可以 匹配到 template<T> 的吧。

 template <class T,class U>
 struct Foo<T,U,true> // 偏特化版本
 {
     void print(){
         printf("they are same type");
     }
 };
```

### iterator_traits

stl 提供，用来 萃取指针 指向的元素的 type。

stl 定义了 5种指针
```C++
 // 五种迭代器类型
 struct input_iterator_tag {};
 struct output_iterator_tag {};
 struct forward_iterator_tag : public input_iterator_tag {};
 struct bidirectional_iterator_tag : public forward_iterator_tag {};
 struct random_access_iterator_tag : public bidirectional_iterator_tag {};
```

iterator 是一种设计模式，无论底层的数据结构是什么，对外统一提供 ++, >, ==, *, & 等 operator。

在萃取前，需要对 指针进行分类，分为 原生指针 (如 int*)，类类型( 如 `vector<int>::iterator`)。

stl 定义了 iterator_traits、iterator_traits_helper、iterator_traits_impl 3个类。

通过偏特化 对原生指针和 类类型进行分类：
- 如果是原生指针，会匹配到 `iterator_traits<T*>`，且 typedef 定义为 random。
- 如果是 类类型，则会 匹配到 主模板，并根据 has_iterator_cat 类模板 ( 根据value 可以判断类T 中是否具有 iterator_category 属性)， 如果为true，进而继承于 iterator_traits_impl，最后将 根据 is_convertible 类模板 ( 根据value 可以判断是否 类T 可以转变为 input 指针 和 output 指针) 决定是否进行 typedef 一系列操作。

其中有两个类模板，分别是has_iterator_cat和is_convertible，他们都继承于bool_constant，都包含一个bool型value成员。为什么需要这两个类模板呢？因为iterator_traits可以萃取的iterator是有前提的，只能对有iterator_category这个属性，并且该iterator_category可转换为input_iterator_tag和output_iterator_tag的iterator萃取。即要求iterator为我们定义的五个iterator类型。具体代码如下：

has_iterator_cat 使用了 SFNIAE

```C++
 template <class T>
 struct has_iterator_cat
 {
 private:
   struct two { char a; char b; };
   template <class U> static two test(...); // 模板函数
   template <class U> static char test(typename U::iterator_category* = 0);
   // 运用了SFINAE技巧，若有iterator_category属性，则会优先匹配到
 public:
   static const bool value = sizeof(test<T>(0)) == sizeof(char);
 };
 ​
 template <class Iterator, bool>
 struct iterator_traits_impl {};
 ​
 template <class Iterator>
 struct iterator_traits_impl<Iterator, true>
 {
   typedef typename Iterator::iterator_category iterator_category;
   typedef typename Iterator::value_type        value_type;
   typedef typename Iterator::pointer           pointer;
   typedef typename Iterator::reference         reference;
   typedef typename Iterator::difference_type   difference_type;
 };
 ​
 template <class Iterator, bool>
 struct iterator_traits_helper {};
 ​
 template <class Iterator>
 struct iterator_traits_helper<Iterator, true>
   : public iterator_traits_impl<Iterator,
   std::is_convertible<typename Iterator::iterator_category, input_iterator_tag>::value ||
   std::is_convertible<typename Iterator::iterator_category, output_iterator_tag>::value>
 {
 };
 ​
 // 萃取迭代器的特性
 template <class Iterator>
 struct iterator_traits 
   : public iterator_traits_helper<Iterator, has_iterator_cat<Iterator>::value> {};
 ​
 // 针对原生指针的偏特化版本
 template <class T>
 struct iterator_traits<T*>
 {
   typedef random_access_iterator_tag           iterator_category;
   typedef T                                    value_type;
   typedef T*                                   pointer;
   typedef T&                                   reference;
   typedef ptrdiff_t                            difference_type;
 };
 ​
 template <class T>
 struct iterator_traits<const T*>
 {
   typedef random_access_iterator_tag           iterator_category;
   typedef T                                    value_type;
   typedef const T*                             pointer;
   typedef const T&                             reference;
   typedef ptrdiff_t                            difference_type;
 };
```



# 萃取

BV12o4y1G7Nc

3种:
- 固定萃取，fixed traits， 给定一个类型，得到一个指定的类型
- 值萃取，value traits， 给定一个类型，得到一个值
- 类型萃取，type traits， 提取，转换 类型信息


```C++
template<class T1, class T2>
T1 func1(const T2 *start, const T2 *end) {

    T1 sum2{};
    // 累加
    return sum2
}

int main() {
    int a[] = {1,2,3};
    int b[] = {1000000000,1000000000,1000000000};
    char c[] = "abc";

    ::std::cout<< func1<int>(&a[0], &a[2]) <<std::endl;
    std::cout<<func1<int64_t>(&b[0], &b[2])<<std::endl;
    std::cout<<func1<int>(&c[0], &c[2])<<std::endl;

}
```

上面的代码 在调用 func1 时必须指定 输出的类型。


---

通过 固定萃取，可以不需要 指定输出类型

```C++
template<class T1>
struct sumType {};

template<>
struct sumType<char> {
    using type = int;
}

template<>
struct sumType<int> {
    using type = int64_t;
};

template<class T1>
auto func2(const T1 *st, const T1 *en) {
    
    using SumType = typename sumType<T1>::type;
    SumType sum2{};
    
    // 累加

    return sum2;
}

// 调用:
cout<<func2(&a[0], &a[2]);
```

---


打印变量的类型
```C++
template<class T1>
struct getType {
    using type = typename T1::value_type;
};

template<class T1, size_t size>
struct getType<array<T1, size>> {
    using type = T1;
};

template<class T1>
using TYPE = typename getType<T1>::type;

template<class T1>
void func1(const T1 &a) {
    cout << abi::__cxa_demangle(typeid(TYPE<T1>).name(), 0, 0, 0);
}

int main() {

    vector<int> a{};
    list<double> b{};
    array<char, 10> c{};

    func1(a);
    func1(b);
    func1(c);
}

```


---

退化

`std::decay<T>`

- 移除const
- 移除左右引用
- 数组转指针
- 函数名转函数指针

```C++

// 移除 const
template<class T>
struct clearConst {
    using type = T;
};
template<class T>
struct clearConst<const T> {
    using type = T;
};

// 移除引用
template<class T>
struct clearRef {
    using type = T;
};
template<class T>
struct clearRef<T&> {
    using type = T;
};
template<class T>
struct clearRef<T&&> {
    using type = T;
};


int main() {
    int a = 123;
    const int b = 456;
    int &c = a;  // lv, left value
    int &&d = 789; // rv
    const int &e = b; // const lv

    cout<<(bool)is_same<clearConst<decltype(b)>::type, int>::value<<std::endl;
    cout<<(bool)is_same<clearRef<decltype(c)>::type, int>::value<<std::endl;
    cout<<(bool)is_same<clearRef<decltype(d)>::type, int>::value<<std::endl;

    // 必须先ref，再const  。。really? why?
    cout<<(bool)is_same<clearConst<clearRef<decltype(e)>::type>::type, int>::value;
    
    return 0;
}
```






===========================



# 平凡，平凡可复制，标准布局


https://blog.csdn.net/blanklog/article/details/127038242


---

https://blog.csdn.net/vviccc/article/details/138089766


---
引入平凡类型，平凡可复制类型，标准布局类型 的主要原因是
1. 内存效率与性能，平凡类型允许位拷贝，提高内存使用效率和程序性能
2. 跨平台兼容性，标准布局类型确保 不同编译器，不同硬件平台上的一致性，增加了代码的可移植性
3. C语言兼容性，平凡类型 与C语言结构体的内存布局兼容，便于C C++互操作
4. 模板编程，这些概念提供了对模板元编程的支持
5. 多线程安全，平凡类型的操作线程安全，简化了并发编程中的同步和锁的需要 (。。？)
6. 简化代码和资源管理，减少了复杂资源管理的需求
7. 避免未定义行为，通过明确的类型特性，减少了因 对象操作导致的未定义行为

主要是为了兼容性





Trivial Type
构建的时候 什么事都不做。
拷贝的时候 只需要递归复制每个标量
随便找块内存，std::memmove 就可以得到对象。

定义：
1. 标量 scalar type
2. 平凡类型 trivial class
    1. 是平凡可复制的，trivially copyable
    2. 存在 平凡的 默认构造器
3. 1或2的数组
4. 1,2,3 的cv限定版


---

指那些在内存中的行为非常简单的类。它们的构造函数，析构器，拷贝构造器，赋值运算符 都没有自定义实现，完全由编译器提供默认行为。这意味着**这些类的对象可以像基本数据类型一样被创建和销毁，不需要特殊的资源管理代码**




Trivial Copy Type

---

是平凡类型的一个扩展，不仅包含 平凡类型，还包含 可以安全复制和移动的类型。例如，一个类可能有一个自定义的构造器，但如果它保证对象的内容可以通过简单的位拷贝来复制，那么它也可以被认为是平凡可复制的



Standard-layout Type

trivial type的内存模型(object representation)只有编译器知道，standard-layout type是透明的，所有人都知道。这样，不同编程语言之间可以用来交流
    
1. 子类和父类不能都有非静态成员，因为子类和基类内存分布不确定
2. 非静态成员必须有相同的访问权限，不同的访问权限段之间内存布局不确定(C++11)
3. 不能有虚函数，虚继承，引用等 跟编译器实现不确定的东西
4. 不能有 2个一样的基类 等 影响空基类优化的东西
5. 所有成员的 内存布局也得是 standard-layout type


---

指那些在内存布局上满足一定规则的类。这些规则包括 
所有非静态成员的访问权限必须相同，
类不能有虚函数或虚基类，且
所有基类也必须是标准布局类型。

标准布局类型的一个重要特性是 它们的内存布局 在不同的编译器上是一致的，这对于跨平台的二进制数据交换非常重要

一个类型是标准布局的，如果它满足下面的条件
1. 类型的所有非静态数据成员都是公共的
2. 类型不包含虚函数，虚基类 或非标准布局的基类
3. 类型的所有基类都是标准布局类型
4. 类型不包含动态内存分配，如没有指向其自身类型的指针成员
5. 类型的所有数据成员的访问权限都是相同的。

std::is_standard_layout 可以检查一个类型是否是标准布局类型。这在跨平台编程中非常重要，因为它保证了类型在不同平台上的ABI兼容性








POD Type
为了C++ 和 C 兼容而设的概念，同时包含 Trivial type 和 standard-layout type的含义。
已经被更细致的 trivial type，standard-layout type 概念所替代。


聚合体
Aggregate
一个纯粹和C++初始化有关的概念。 聚合体可以直接使用 花括号 对相应的成员进行聚合初始化
为了能进行 聚合初始化，有一些限制条件
1. 不能有用户定义继承的构造器，让编译器负责聚合初始化规则
2. 对象模型中成员拥有公共访问权限
3. 没有虚函数等需要特殊初始化处理的操作







============================

# POD
plain old data

针对POD对象，其二进制内容是可以随便复制的，在任何地方，只要其二进制内容在，就能还原出正确无误的POD对象。对于任何POD对象，都可以使用memset()函数或者其他类似的内存初始化函数。

传统的  C  风格  的  struct。

============================

# 外部模板

C++11  引入。

C++98/03  中，对于源码中出现的每一处模板实例化，  编译器都需要  进行实例化的工作，  在链接时，链接器还需要  移除重复的  实例化代码。

```C++
显示实例化语法：template class vector<MyClass>;
外部模板语法：extern template class vector<MyClass>;
```

在一个编译单元中使用了  外部模板声明，那么编译器在  编译该单元的时候，会跳过  和这个外部模板声明  匹配的  模板的  实例化。

```C++
//fun.h
template <typename T>
void fun(T t){
}

//use1.cpp
void test1(){
    fun<int>(1);
}

//use2.cpp
void test2(){
    fun<int>(1);
}
```

编译这2个  cpp  文件时，需要分别实例化  2个模板

```C++
//fun.h
template <typename T>
void fun(T t){
}

//use1.cpp
void test1(){
    fun<int>(1);
}

//use2.cpp
extern template void fun<int>(int);
void test2(){
    fun<int>(1);
}
```

============================

# 初始化列表

结构  或  数组  可以通过  给定一串  按照  结构中  成员定义  的次序  排列  的  参数  来创建。

初始化列表  可以  递归创建，因此，结构数组  或  包含其他结构的  结构  也可以  使用  初始化列表。

C++  存在  能让  对象初始化的  构造器特性。  但是  构造器  并不能取代  初始化列表的  所有功能。

C++  允许类  和  结构  使用初始化列表，但  它们==**必须满足  POD  的定义**==。   非  POD  的类  不能  使用  初始化列表，  std::vector，boost::array  也不行。

。。。vector  可以的吧。。。  看下面，  应该是  vector  的构造器  接受  initializer_list。  但是无法直接  使用初始化列表。

C++11  把  初始化列表  绑定为  一种  名为  `std::initializer_list`  的类型。  这将允许  构造器  及  其他函数  接受  初始化列表  作为其参数

============================

# new对象时是否增加()

在new  对象时，  加不加  ()  有什么区别：
```C++
    CBase *base = new CDerived();
    CBase *base = new CDeviced;
```

很多人说：  加括号  调用  没有参数的  构造器，  不加括号，调用  默认构造器  或  唯一的构造器。  这是有问题的。

## 对于自定义类型

如果  该类  没有  定义构造器  (  由  编译器  生成  默认构造器  )  也没有虚函数，那么  class c = new class;  将  不调用  生成的  默认构造器，  而  class c = new class();  将  调用  默认构造器。

。。不掉  生成的默认构造器，  你掉什么。。难道  是  随机值  和  nullptr  ？
。。不，下面有例子，A a3; 运行时异常。

如果该类  没有定义构造器  但有  虚函数，那么  加不加括号  都是  调用  默认构造器。

如果  定义了  默认构造器，那么  加不加括号  都是  调用  默认构造器。

## 对于内置类型

```C++
int* a = new int;  不会将申请到的  空间  初始化。
int* a = new int();  会初始化为  0

int *p1 = new int[10];       //  随机值
int *p2 = new int[10]();     //  全0
```

## 结论，new必须和()一起
结论：别使用不带括号的new。  。。。  new  必须和  ()  一起。

```C++
class A{
public:
    //A(){a=1;}
public:
    int a;
};
A *a1=new A;
A *a2=new A();

cout<<a1->a<<endl; //输出:-842150451
cout<<a2->a<<endl; //输出0
A a3;
cout<<a3.a<<endl; //运行异常，a没有初始化。
```

如果加上一个virtual，如virtual ~A(){},
则
    cout<<a1->a<<endl; //输出:-842150451
    cout<<a2->a<<endl; //输出 -842150451

。。。？感觉这文章不太对。  默认构造器  应该会初始化为  全0  的啊。  这里带上  虚函数后，应该  全部  输出0  才对啊。
。。好像没有问题，看下面的，  默认构造器不会初始化  int。。。  到底会不会？
。。而且，无论如何，a2 应该是 0 啊。

一个类满足下列其中任何一个条件:
1.包含了一个类的对象，这个对象有一个构造函数(包括编译器合成的默认构造函数)
2.如果继承自一些基类，其中某些基类有一个构造函数(包括编译器合成的默认构造函数)
3.有一个虚函数，或者继承到了虚函数
4.有虚基类

如果这个类没有默认的构造函数，编译器就会合成一个默认的构造函数，分别做以下事情
如果这个类有构造函数，编译器就会扩张所有构造函数，做以下事情
1.调用这个对象的构造函数
2.调用基类的构造函数
3.设置正确的虚函数表指针
4.设置指向虚基类对象的指针

C++在new时的初始化的规律可能为：
对于有构造函数的类，不论有没有括号，都用构造函数进行初始化；
如果没有构造函数，则不加括号的new只分配内存空间，不进行内存的初始化，而加了括号的new会在分配内存的同时初始化为0

============================

# 值初始化的效果

cppreference 中指出了值初始化的效果：
1) 若 T 是没有默认构造函数，或拥有用户提供的或被删除的默认构造函数的类类型，则默认初始化对象；

2) 若 T 是拥有默认构造函数的类类型，而默认构造函数既非用户提供亦未被删除（即它可以是拥有隐式定义的或默认化的默认构造函数的类），则零初始化对象，然后若其拥有非平凡的默认构造函数，则默认初始化它；（C++11 后）

标准指定在类拥有用户提供的或被删除的默认构造函数时不进行零初始化，即使重载决议不选择该默认构造函数。

默认初始化的效果是：若 T 是非 POD (C++11 前)类类型，则考虑各构造函数并实施针对空实参列表的重载决议。调用所选的构造函数（默认构造函数之一），以提供新对象的初始值；若 T 是数组类型，则每个数组元素都被默认初始化；否则，不做任何事：具有自动存储期的对象（及其子对象）被初始化为不确定值

在进行任何其它初始化前，对具有静态或线程局部存储期且不进行常量初始化的具名变量进行零初始化；

============================

============================

============================

============================

============================

============================

============================
unique_ptr

============================

============================

# 友元

https://blog.csdn.net/gx714433461/article/details/124039254

对于自定义类型  Date  类，为了打印  Date  类的信息，需要  我们自己在  Date  类中  写  打印函数：
```C++
void Print() const
{
    cout << _year << "-" << _month << "-" << _day << endl;
}
```

而内置类型  在  不写打印重载函数  的情况下  缺可以直接打印
```C++
#include<iostream>
using namespace std;

int main()
{
    int a = 0;
    cout << a << endl;
    return 0;
}
```

因为库中  对  内置类型  写了很多  运算符  重载函数，  包括  operator>>  和  operator<<  并能自动识别  类型  构成重构，所以  内置类型  直接能打印。  而  自定义类型  需要自己编写  打印函数。

对于自定义的  Date  类，只能自己写  operator<<  和  operator>>  重载函数了
```C++
#include<iostream>
using namespace std;

class Date
{
public:
    //构造函数
    Date(int year = 2022, int month = 4, int day = 8)
    {
        _year = year;
        _month = month;
        _day = day;
    }

    //void Date::Print() const
    //{
    //        cout << _year << "-" << _month << "-" << _day << endl;
    //}

    void operator<<(ostream& out)
    {
        out << _year << "-" << _month << "-" << _day << endl;
    }

    void operator>>(istream& in)
    {
        in >> _year;
        in >> _month;
        in >> _day;
    }

private:
    int _year;
    int _month;
    int _day;
};

int main()
{
    Date d1;
    cout << d1;

    return 0;
}
```

但是  `cout<<d1`  调用不了  <<  运算符
我们需要的是  `cout<<d1`，  但是  调用  operator<<  实际上是执行
`void operator<<(ostream& out) //void operator<<(Date* this, ostream& out)`

d1作为this  就成了  第一个参数，main  函数  里面  调用  `d1<<cout`，  编译通过
```C++
int main()
{
    Date d1;
    d1 << cout;//我们需要cout<<d1

    return 0;
}
```

d1 << cout  不符合我们的习惯，可读性不高，我们需要  cout  作为  第一个参数，这样就会造成  d1  和  cout  抢占  第一个参数的  问题。

假如把  operator>>  和  operator<<  重载不写在  类里面，而是写成  全局函数  写在  Date类  外面，我们可以指定参数顺序
```C++
#include<iostream>
using namespace std;

class Date
{
public:
//构造函数
    Date(int year = 2022, int month = 4, int day = 8)
    {
        _year = year;
        _month = month;
        _day = day;
    }

private:
    int _year;
    int _month;
    int _day;
    };

void operator<<(ostream& out,const Date& d)
{
    out << _year << "-" << _month << "-" << _day << endl;
}

void operator>>(istream& in, const Date& d)
{
    in >> _year;
    in >> _month;
    in >> _day;
}

int main()
{
    Date d1;
    cout << d1;

    return 0;
}
```

但是访问不了  私有成员变量  (_year, _month, _day)。

## 友元函数

是定义在类外部的  普通函数，不属于任何类，但  可以直接  访问  类的  私有成员，需要在  类的  内部  使用friend  关键字  进行声明，类就会把  友元函数  当做  类里的  成员。

```C++
#include<iostream>
using namespace std;

class Date
{
    //友元函数
    friend ostream& operator<<(ostream& out, const  Date& d);
    friend istream& operator>>(istream& in, Date& d);

public:
//构造函数
    Date(int year = 2022, int month = 4, int day = 8)
    {
        _year = year;
        _month = month;
        _day = day;
    }

private:
    int _year;
    int _month;
    int _day;
};

ostream& operator<<(ostream& out,const Date& d)
{
    out << d._year << "-" << d._month << "-" << d._day << endl;

    return _cout;
}

istream& operator>>(istream& in, Date& d)
{
    in >> d._year;
    in >> d._month;
    in >> d._day;

    return in;
}

int main()
{
    Date d1;
    cin >> d1;
    cout << d1 <<endl;

    return 0;
}
```

### 友元函数特性
友元函数  可以访问  类的  私有和保护成员，但不是类的  成员函数
友元函数  不能用const  修饰  (const  修饰  this  指针  指向的内容，  友元函数  作为全局函数  没有  this  指针)
友元函数  可以在  类定义的  任何地方声明，不受类访问限定符限制
一个函数  可以是  多个类的  友元函数
友元函数的调用  和  普通函数的  调用  和  原理  相同。

## 友元类
友元类  的  所有成员函数  都可以是  另一个类的  友元函数，  都可以访问  另一个类中的  非公有成员。
下面  Date  类  是  Time  类的  友元类，Date  类要访问  Time类，就要把  Date  类  定义为  Time类的  友元

```C++
class Date; // 前置声明
class Time
{
friend class Date; // 声明日期类为时间类的友元类，则在日期类中就直接访问Time类中的私有成员变量
public:
    Time(int hour = 0, int minute = 0, int second = 0)
        : _hour(hour)
        , _minute(minute)
        , _second(second)
    {}

private:
    int _hour;
    int _minute;
    int _second;
};

class Date
{
public:
    Date(int year = 1900, int month = 1, int day = 1)
        : _year(year)
        , _month(month)
        , _day(day)
    {}
    void SetTimeOfDate(int hour, int minute, int second)
    {
        // 直接访问时间类私有的成员变量
        _t._hour = hour;
        _t._minute = minute;
        _t._second = second;
    }

private:
    int _year;
    int _month;
    int _day;
    Time _t;
};

int main()
{
    return 0;
}
```

### 友元类特性
友元关系是单向的，不具有  交换性

Time类中，声明  Date  是  其友元类。  Date中可以直接访问  Time  的私有成员，  但是不能在  Time  中访问  Date  的  私有成员

友元关系  不能传递
A是B的友元，B是C的友元，  但  A  不是C  的友元

## 内部类
一个类定义在  另一个类的内部。

内部类  和  外部类关系
内部类  不属于  外部类，  更不能通过  外部类的  对象  调用  内部类。外部类  对  内部类  没有任何优越的  访问权限。

内部类是外部类的友元类，内部类  可以通过  外部类的  对象参数  来访问  外部类的  所有成员，但  外部类  不是内部类的  友元。

```C++
#include<iostream>
using namespace std;

class A {
private:
    static int k;
    int h;
public:

    class B  //B是A的内部类，B是A的友元类
    {
    public:
        void foo(const A& a)
        {
            cout << k << endl;//内部类可以访问外部类的非公有成员
            cout << a.h << endl;//普通成员只能通过对象访问，不能通过类名访问
        }

    private:
        int _b;
    };
};

int A::k = 1;

int main()
{
    A::B b; //定义一个内部类对象
    b.foo(A());

    return 0;
}
```

### 内部类特性
内部类  可以定义在  外部类的  public protected private  都是可以的。
注意内部类  可以直接  访问  外部类的  static  ，枚举成员，  不需要  外部类的  对象/类  名
sizeof(外部类) =  外部类，  和内部类没有任何关系

```C++
int main()
{
    A::B b;
    b.foo(A());

    //由于静态成员变量不在对象中存储，类的大小为非静态成员变量内存对齐的结果,A只有一个非静态成员变量
    cout << "sizeof(A) = " << sizeof(A) << endl;

    return 0;
}
```

builder，  iterator  作为友元(类)

============================
# 内部类

============================

============================

# 悬浮指针
声明了，但没有被  赋值的指针，指向了  内存中的  任意一个空间。
避免悬浮指针的  一个方式是  初始化为  NULL

## 野指针

不是  NULL  指针，是  指向  垃圾内存的  指针。  人们一般不会用错  NULL  指针，因为  if  语句很容易判断。  但是  野指针  很危险，  if  语句  对它不起作用。

野指针有  2个  成因
指针变量没有被初始化。
指针被  free  或  delete  后，没有置为NULL，  让人  误以为  p  是合法的指针。

还有一个需要注意的问题：  不要返回  指向  栈内存的  指针  或引用，因为  栈内存  在  函数  结束时  会被  释放。

============================

============================

# setjmp, longjmp

由於 setjmp 和 longjmp 不支援在 C++ 編譯器之間可移植的堆疊框架物件正確解構，而且因為可能會藉由防止區域變數優化來降低效能，因此不建議在 C++ 程式中使用它們。

< setjmp.h > 或 < setjmpex.h >

----------------------

C中，无法使用  goto  跳转到  另一个函数的  某个label处。  setjmp, longjmp  来完成这种跳转。这2个函数在  异常处理上  非常有用。

要完成下面的跳转：
```C
void f()
{
    //...
    Label:
    //...
}
void g()
{
    //...
    GOTO Label;
    //...
}
```

首先，我们要知道，实现这种类型的跳转，和操作系统中  任务切换的上下文切换有点类似，我们只需要回复  Label  标签处  函数上下文即可。
函数上下文包括：
函数栈帧，主要是栈帧指针BP和栈顶指针SP
程序指针PC，此处为指向Label语言的地址
其它寄存器，这是和体系相关的，在x86体系下需要保存有的AX/BX/CX等  callee-regs

```C
#include <setjmp.h>
int setjmp(jmp_buf env);
```

setjmp  函数的功能是  将函数在此处的  上下文保存在  jmp_buf  结构体中，以供  longjmp  从此结构体中恢复。
参数  env  就是  保存上下文的  jmp_buf  结构体变量

如果直接调用该函数，返回值为0；如果该函数从  longjmp  调用返回，返回值为  非零，由longjmp函数提供。提供函数的返回值，我们就知道  setjmp  函数调用  是第一次直接调用，还是由其它地方跳转过来的。

void longjmp(jmp_buf env, int val);
从  jmp_buf  结构体中  恢复由  setjmp  函数保存的上下文，该函数不返回，  而是从  setjmp  函数中返回。
参数env  是由  setjmp  函数保存过的  上下文。

参数  val  表示从  longjmp  函数传递给  setjmp  函数的返回值，如果  val  值为0，setjmp  将返回1，否则返回  val

longjmp  不直接返回，而是从  setjmp  函数中返回，longjmp  执行完后，程序就像  刚从  setjmp  函数返回一样。

C中没有  异常机制，但是可以使用  setjmp  和  longjmp  来模拟实现该功能：

```C
static jmp_buf env;

double divide(double to, double by)
{
    if(by == 0)
        longjmp(env, 1);
    return to / by;
}

void f()
{
    if (setjmp(env) == 0)
        divide(2, 0);
    else
        printf("Cannot / 0");
    printf("done");
}

更复杂一点，可以根据  longjmp  传递的返回值  来判断  各种不同的异常，来进行不同的处理：
switch(setjmp(env)):
    case 0:         //default
        //...
    case 1:         //exception 1
        //...
    case 2:         //exception 2
        //...
    //...
```

============================

`std::sort(begin(vi), end(vi), fun_a<b?(a, b))`

============================

============================

右值  + const  转成左值
左值  + move  转成右值

具名的右值  被当做左值。
。。由于被当做左值，所以无法发给右值，所以需要  forward  完美转发。？。  T&&  万能引用，T是模板参数。引用折叠

语义

# 完美转发

移动构造函数，形参是右值。。
延长生命周期。  可以将堆上的资源  的生命周期  扩展到  对象之外，  就是  原对象的内容给与新对象  原对象析构了，  但是原对象的内容没有被析构。
class A { A(A&& a) {this->bigData = a.bigData; a.bigData=null;} } ?

。。但是，为什么不直接用  原来的对象呢？  生成新对象的时候，原对象并没有被回收，那么说明  它们是  同一个  作用域的。并且  新对象的作用域  可能更小。

除非可以  return std::move(a);

------------------------------

https://juejin.cn/post/6861080373913370638

# 使用  &&  定义右值引用

右值引用  一定  不能被  左值初始化，只能用  右值初始化。  因为  右值ref  的目的是为了  延长  用来  初始化对象的  生命周期，  对于左值，其生命周期  和  其作用域有关，你没有必要  去延长。

既然是延长，就出现了下面的情况
int x = 20;  //  左值
int&& rx = x * 2;    //  x*2  的结果是一个  右值，  rx  延长其生命周期。
int y = rx + 2;         // 42
rx = 100; //  一旦你初始化一个  右值ref变量，该变量就变成  一个左值，可以被赋值。
。。。延长这个。。  直接  int rx = x * 2  不好吗。   可能是  大对象，就不需要重新  构造了。

## 右值ref的作用
1. 延长生命周期
C++11  之前

```C++
class A  {
public:
    A() : m_ptr(new int(0))  {
        cout << "construct" << endl;
    }
    A(const A& rhs) : m_ptr(new int(*rhs.m_ptr)) // 拷贝构造函数
    {
        cout << "copy construct" << endl;
    }
    ~A()  {
        cout << "destruct" << endl;
        delete m_ptr;
    }
private:
    int* m_ptr;
};

A Get(bool flag)  {
    A a, b;
    if (flag)  {
        return a;
    }
    return b;
}

int main()  {
    A a = Get(false);//调用拷贝构造函数
    return 0;
}
```

执行结果：
construct
construct
copy construct
destruct
destruct
destruct

上述例子中，  Get()  会返回临时变量，  然后通过  这个临时变量  拷贝构造  一个  新的  对象  a，  临时变量  在  拷贝构造之后  就销毁了。
```C++
class A  {
public:
    A() : m_ptr(new int(0))    {
        cout << "construct" << endl;
    }
     A(const A& rhs) : m_ptr(new int(*rhs.m_ptr)) // 拷贝构造函数
    {
        cout << "copy construct" << endl;
    }
     A(A&& rhs) : m_ptr(rhs.m_ptr) //移动构造函数
{
                  rhs.m_ptr = nullptr;
                  cout << "move construct" << endl;
}
    ~A()    {
        cout << "destruct" << endl;
        delete m_ptr;
    }
private:
    int* m_ptr;
};

A Get(bool flag)  {
    A a, b;
    if (flag)    {
        return a;
    }
    return b;
}

int main()  {

    A a = Get(false);//调用拷贝构造函数。。。。这里应该不是  拷贝构造了吧？  不过怎么  区分呢？完全一模一样啊。  感觉是  代码写错了，  这里应该是  A&& a = Get(false)；  才会  调用  移动构造吧？  。。。  不，  拷贝构造和  移动构造  是根据  形参  来  重载的，  不是根据  返回值，而且返回值也无法影响  重载。。  所以  就是  Get(false)  返回了一个  将亡值，  会  优先匹配到   移动构造？？

    return 0;
}
```

执行结果：
construct
construct
move construct
destruct
destruct
destruct

改进之后，不再调用  拷贝构造函数，而是  调用  移动构造函数。

移动构造函数  的  参数  是一个  右值ref，这里  没有  深拷贝，只有浅拷贝，这样就避免了  对临时对象的  深拷贝，和资源的  重复申请和释放，  提高了  性能，  延迟了  临时对象的  声明周期。

。。  感觉：  拷贝构造  是  深拷贝，   移动构造  是  浅拷贝，不算  浅拷贝，  算是  "掠夺"拷贝。。深拷贝  是  复制指针  +  实体，  浅拷贝是  复制指针，  移动构造是  直接把指针  偷过来了。。  导致  被移动的  对象  没有指针，  所以  被移动的对象  析构时，  没有指针，就无法  释放掉  实体。

。。等于就是  换个壳子，  因为  原先的壳子  出了  作用域  就析构了，并且  这个壳子的  析构函数  会把  壳子里的东西  也释放掉，  现在用了  移动构造，把  壳子里的东西  放到  新的  壳子里了，  原先的壳子  析构时，只是析构壳子，没有办法析构  壳子里的东西了(因为被搬走了)。

1. 转移不可拷贝的资源
如果资源是  不可拷贝的，那么  装载资源的  对象  也应该是  不可拷贝的。

如果资源对象不可拷贝，一般需要  定义  移动构造/移动赋值  函数，并  禁用  拷贝构造/拷贝赋值  函数。  例如，  std::unique_ptr  只能  移动。

```C++
template<typename T>
class unique_ptr {
 public:
  unique_ptr(const unique_ptr& rhs)  = delete;
  unique_ptr(unique_ptr&& rhs) noexcept;  // move only
 private:
  T* data_ = nullptr;
};

unique_ptr::unique_ptr(unique_ptr&& rhs) noexcept {
  auto &lhs = *this;
  lhs.data_ = rhs.data_;
  rhs.data_ = nullptr;
}
```

## 右值ref  的几点说明
右值ref  就是  对一个  右值进行  ref  的类型，  因为右值不具名，因此只能通过  ref  的方式  找到它。

。。。所以  之前的  A a = Get();  发生的  移动构造  所  使用的  实参  就是  Get()  返回的  对象，因为这个  返回的  对象  暂时  没有  名字。

通过右值ref  的声明，该右值  又"重获新生"，  其生命周期  和  右值ref类型变量  的  生命周期一样。
无论声明为  右值ref  还是  左值ref   都必须立即初始化。

## 右值ref解决如下问题
1. 移动语义  std::move
移动语义  是通过  右值ref  来匹配临时值的，那么  普通的左值  是否  也可以  借助  移动语义来  优化性能呢？
C++11  为了解决上述问题，提供了  std::move  方法来将  左值或右值  转换为  右值，从而  方便  应用  移动语义。
move  实际上不能移动任何东西，它唯一的  功能是将  左值  强转为  右值ref，  使我们可以  通过  右值  ref  使用该值。
```C++
 class string {
 public:
  string(const string& other);  // Copy constructor, exists pre C++11

  string(string&& other) {      // Move constructor, new in C++11
    length = other.length;
    capacity = other.capacity;
    data = other.data;

    other.data = nullptr; // 将临时对象的指针设置为空，other.data = nullptr;。若不将other.data设置为空，other.data 将被释放两次，导致程序异常。

  }

 private:
  size_t length;
  size_t capacity;
  const char* data;
};

int main(){
  string a(get_string());  // move constructor
  string b(a);             // copy constructor
  string c(std::move(b));  // move constructor
  return 0;
}
```

注意，如果  string  中没有  移动构造函数  string(string&& other)，  那么  string c(std::move(b))  仍然将  调用  拷贝构造函数，  因为  string(const string& other)  的参数  是  const reference。

C++11  中，编译器会在  类中自动添加  移动构造函数  和  移动赋值  操作符，  因此，声明一个类，编译器会  默认添加：构造，析构，拷贝构造，移动构造，移动赋值。

。。拷贝赋值  有吗。。拷贝赋值，好像没有意义，  就是  拷贝构造。。。

通过  std::move  将  左值  转为右值后，原来的  变量  放弃了  内存的  所有权，后续代码  不能  使用  该变量  来访问  对象，  如果使用，会出现  未定义行为。
```C++
std::string base_url = tag->GetBaseUrl();
if (!base_url.empty()) {

  UpdateQueryUrl(std::move(base_url) + "&q=" + word_); //若执行了，后续base_url不能使用，若使用会出现未定义行为。

}
LOG(INFO) << base_url;  // |base_url| may be moved-from
```

被移动的对象处于  合法  但未指定  状态：

1. 基本要求：能正确析构  (不会重复  释放  已经被移动了的  资源，例如  std::unique_ptr::~unique_ptr()  检查  指针是否需要  delete )

2. 一般要求：重新赋值后，和新的对象没有差别  (C++  标准库  基于这个假设)
3. 更高要求：恢复为默认值  (例如，std::unique_ptr  恢复为  nullptr)
```C++
auto p = std::make_unique<int>(1);
auto q = std::move(p);

assert(p == nullptr);  // OK: reset to default
p.reset(new int{2});   // or p = std::make_unique<int>(2);
assert(*p == 2);       // OK: reset to int*(2)
```

由于基本类型不包含资源，其  移动  和  拷贝相同：  被移动后，保持为  原有值：
int i = 1;
int j = std::move(i);

assert(i == j)

## 完美转发  std::forward
在传递过程中  保持  其值属性  的功能，即  如果是左值，传递之后依然是  左值，如果是右值，传递之后依然是  右值。

```C++
void func(int &arg) {
    std::cout << "func lvalue" << std::endl;
}
void func(int &&arg) {
    std::cout << "func rvalue" << std::endl;
}
template <typename T>
void wrapper(T &&args) { //无论args是什么引用类型，args都是左值
    func(args);
}
int main() {
    int a = 10;

    wrapper(a); //传递左值
    wrapper(20); //传递右值

    return 0;
}
```

以上函数输出：
func lvalue
func lvalue

在  wrapper  函数中  调用  func  函数时，对应的  ref类型  不能保持不变，即  ref属性  不能被传递。

```C++
void func(int &arg) {
  std::cout << "func lvalue" << std::endl;
}
void func(int &&arg) {
  std::cout << "func rvalue" << std::endl;
}
template <typename T>
void wrapper(T &&args) {
  func(std::forward<T>(args));
}
int main() {
  int a = 10;

  wrapper(a);
  wrapper(20);

  return 0;
}
```
结果输出如下：
  func lvalue
  func rvalue

## 拷贝省略  copy elision

尽管C++  引入  移动语义，  移动的过程仍然有优化的空间  ---  与其调用一次  没有意义的  移动构造函数，不如让  编译器  直接跳过这个过程  ---  于是就有了  拷贝省略。

很多人  会混淆  移动语义  和  拷贝省略  ：
移动语义  是  语言标准  提出的概念，  通过编写遵守  移动语义的  移动构造函数，右值  限定  成员函数，逻辑上  优化对象内资源的  转移流程。

拷贝省略  在C++17  前是  非标准的编译器优化，  跳过  移动/拷贝  构造函数，让  编译器直接  在  移动后的  对象内存上，构造  被移动的  对象。

std::unique_ptr<int> foo() {
  auto ret = std::make_unique<int>(1);
  //...
  return std::move(ret);  // -> return ret;
}

没有必要  使用std::move()  移动  非引用返回值。C++  会把  即将离开  作用域的  非ref类型的  返回值  当做  右值，对返回的  对象  进行  移动构造  (语言标准)；如果  编译器  允许  拷贝省略，还可以  省略这一步的构造，直接把  ret  存放到  返回值的  内存里  (编译器优化)

std::move  和  std::forward  的用处。
不移动右值ref参数
std::unique_ptr<int> bar(std::unique_ptr<int>&& val) {
  //...
  return val;    // not compile
                 // -> return std::move/forward(val);
}

上述代码的问题在于：没有对返回值使用  std::move() (编译器提示  std::unique_ptr(const std::unique_ptr&) = delete  错误)。

因为无论  左值ref  还是  右值ref  的变量(或参数)  在初始化后，都是  左值。  所以  返回右值ref  变量时，  需要使用  std::move() / std::forward()  显式地  移动转发  或  完美转发，  将  变量  "还原"  为右值(右值ref类型)

-----------------------------

https://juejin.cn/post/7137669991825932295

# 右值

C++11  在性能上做了很大的改进，最大程度减少了内存移动和复制，通过右值ref，forward，emplace  和一些无序容器，我们可以大幅度改进程序性能。

右值ref  仅仅是通过  改变资源的所有者  (剪切方式，而不是  拷贝方式)  来避免内存的拷贝，能大幅度提高性能。
forward  能根据  参数的  实际类型  转发  给  正确的函数
emplace  系列函数  通过  直接构造对象的方式  避免了  内存的拷贝  和移动。

左值是  表达式结束后  依然存在的  持久对象，  右值  是  表达式结束时  就不存在的  临时对象。

有地址  的变量  就是  左值，没有地址的  字面值，临时值  就是右值。

左值ref
ref  是  变量的别名，由于  右值没有地址，没法被修改，所以左值ref  无法指向右值。
int a = 5;
int &ref_a = a; // 左值引用指向左值，编译通过
int &ref_a = 5; // 左值引用指向了右值，会编译失败

需要注意，  const  左值  ref   是可以指向  右值的，  const左值ref  不会  修改指向的值，  所以可以指向右值
const int &ref_a = 5; // 编译通过

这也是为什么  要使用  const &  作为  函数参数的  原因之一，如  std::vector  的  push_back
void push_back (const value_type& val);
如果没有  const，  vi.push_back(5)  这样的代码  就无法编译。

右值ref
int &&ref_a_right = 5; // ok
int a = 5;
int &&ref_a_left = a; // 编译不过，右值引用不可以指向左值
ref_a_right = 6; // 右值引用的用途：可以修改右值

右值ref  就是对  一个右值  进行ref  的类型。  因为右值没有名字，所以  我们只能通过  ref  的方式找到它。

无论  声明  左值ref   还是  右值ref，  都必须立刻进行初始化。  因为  ref类型  本身  并不拥有  所绑定对象的  内存，只是  该对象的  一个别名

通过右值ref  的声明，该  右值  又  重获新生，  其生命周期  与  右值ref类型变量  的  生命周期  一样。

被声明出来的  左  右  值  都是  左值。  因为  被声明  出来的  左右值  ref  是有地址的，  也位于  等号  左边。

右值ref  可以  ref  左值和右值。
左值ref  只能指向  左值
const左值ref  可以指向右值。

1. 从性能上讲，左右值ref  没有区别，  传参  使用  左右值ref  都可以  避免拷贝

2. 右值ref  可以  直接ref到右值，  也可以通过  std::move  指向左值；左值ref  只能ref左值( const左值ref  可以ref右值)。

3. 作为函数形参时，右值ref  更加灵活。  虽然  const  左值ref   也可以  接收  左右值，但是它  无法修改，有一定局限性。

```C++
void f1(const int& n) {
    n += 1; // 编译失败，const左值引用不能修改指向变量
}
void f2(int && n) {
    n += 1; // ok
}
int main() {
    f1(5);
    f2(5);
}
```

1. 左值和右值  都是独立于  它们的类型的，右值ref类型  自身  可能是左值  也可能是右值。

2. auto&&  或  函数参数类型自动推导的T&&  是一个未定的ref类型，  被称为  universal references，  它可能是  左值ref  也可能是  右值ref类型，取决于  初始化的值类型。

3. 所有  右值ref  叠加到  右值ref  上仍然是一个  右值ref，其他ref折叠都为  左值ref。当  T&&  为模板参数时，输入左值，它会变成左值ref，而输入右值时，则变成具名的右值ref。

4. 编译器  将  已命名的右值ref  视为左值，  未命名的右值ref  视为右值。

emplace_back
就地构造，不是  构造后再次复制到容器中。因此效率更高。

```C++
vector<string> testVec;
testVec.push_back(string(16, 'a'));
```

底层实现：
1. string(16, 'a')  会创建  一个  string  类型的  临时对象，这涉及到  一次  string  构造。
2. vector  内会创建一个  新的  string  对象。
3. 在  push_back  结束时，  第一步的  临时对象会被析构。

emplace_back  可以直接在  vector  中构建一个  对象，而  不是  创建一个临时对象，再放进  vector，再销毁。  可以省略一次  构建  和  一次析构。

。。省略的是  临时对象，  所以如果不是临时对象的，那就无所谓了？

```C++
int main() {
    std::vector<std::string> v;
    int count = 10000000;
    v.reserve(count); //预分配十万大小，排除掉分配内存的时间
    {
        TIME_INTERVAL_SCOPE("push_back string:");
        for (int i = 0; i < count; i++) {
            std::string temp("ceshi");
            v.push_back(temp);// push_back(const string&)，参数是左值引用，内部做深拷贝
        }
    }
    v.clear();
    {
        TIME_INTERVAL_SCOPE("push_back move(string):");
        for (int i = 0; i < count; i++) {
            std::string temp("ceshi");

            v.push_back(std::move(temp));// push_back(string &&),  参数是右值引用,内部做move

        }
    }
    v.clear();
    {
        TIME_INTERVAL_SCOPE("push_back(string):");
        for (int i = 0; i < count; i++) {
            v.push_back(std::string("ceshi"));// push_back(string &&),  参数是右值引用
        }
    }
    v.clear();
    {
        TIME_INTERVAL_SCOPE("push_back(c string):");
        for (int i = 0; i < count; i++) {
            v.push_back("ceshi");// push_back(string &&),  参数是右值引用
        }
    }
    v.clear();
    {
        TIME_INTERVAL_SCOPE("emplace_back(c string):");
        for (int i = 0; i < count; i++) {
            v.emplace_back("ceshi");//  只有一次构造函数，不调用拷贝构造函数，速度最快
        }
    }
}
```

push_back string:408045 ms
push_back move(string):293146 ms
push_back(string):282487 ms
push_back(c string):272085 ms
emplace_back(c string):228662 ms

第一种耗时最长，因为  调用左值ref  的  push_back，会调用  string  的  拷贝构造。
第2,3,4  基本一样，参数  为右值，  调用   右值ref  的  push_back，会调用  string  的  移动构造函数。
5最少，因为  emplace_back  只调用  构造函数，没有  移动构造，也没有  拷贝构造。

---------------------------

============================

# ADL

argument dependent lookup
ADL，  Koenig查找
参数依赖查找

ADL  使得  使用  定义于  不同  命名空间  的运算符  可行。例如

```C++
#include <iostream>
int main()
{
    std::cout << "Test\n"; // 全局命名空间无 operator<< ，但 ADL 检验 std 命名空间，
                           // 因为左参数在 std 命名空间中
                           // 并找到 std::operator<<(std::ostream&, const char*)
    operator<<(std::cout, "Test\n"); // 同上，用函数调用记法

    // 然而，
    std::cout << endl; // 错误： 'endl' 不声明于此命名空间。
                       // 此非对 endl() 的函数调用，故不应用 ADL
。。？但是我写过cout<<endl;  啊。
。。。那是因为  我的代码中  使用了  using namespace std;

    endl(std::cout); // OK ：这是函数调用： ADL 检验 std 命名空间，
                     // 因为 endl 的参数在 std ，并找到 std::endl

    (endl)(std::cout); // 错误： 'endl' 不声明于此命名空间。
                       // 子表达式 (endl) 不是函数调用表达式
}
```

。。？？？

看  C/C++  权威。  里面有  cppreference  的说明。

============================

# 前向声明

类应该先声明，后使用。
如果要在某个类的声明之前  引用该类，则应该进行  前向引用声明

```C++
class B;

class A {
public:
    void f(B b);
};

class B {
public:
    void g(A a);
};
```

使用前向声明需要注意：

在提供一个  完整的  类声明  之前，不能声明  该类的  对象，  也不能在  内联成员函数中  使用  该类的  对象，  但是  可以声明  指向  该类的  指针。

在使用前向引用声明  前，  你只能  使用  被声明的符号，  不能涉及任何  类的细节。

----------------------------

类的前向声明  是利用了  编译器的特性，编译器  在编译的  过程中，  只需要知道  各个元素的  名称  和  相应的大小即可。  而在  C++  中  每个类的  大小是  固定的，这时  使用  前向声明的  类  就可以通过  编译器。

。。这里应该说错了，  是  类指针  或  类ref  的大小是固定的。

假设  B  类中已经  include  了  a.h，  那么  A类中  就不能再  包含  b.h  ，要在  A  类中  前向声明  类  B。
。。这个代码  和  描述反了。。

```C++
// A.h
#include "B.h"
class A {
    A(void);
    ~A(void);
    B b_;
};

// B.h
class A;
class B {
    B(void);
    ~B(void);
    void fun(A& a) {               //  只能是指针或引用

    }
    A* a_;              //  前向声明的类  不能实例化对象
};
```

。。但是，这个  B.h  怎么找到  A，  不不不，正好是之前提到的，编译器  只需要知道  名字  和  大小，  所以  找不找到  A  无所谓。

。。但是，B  类  中的  fun  函数  中  是无法调用  a  的吧，  因为  不知道  A  里面的方法  是什么。  那么  这个方法  就毫无意义啊。

。。A  应该是在  link  后  才知道内容？。  那么  B  里  就无法  调用  A的任何方法啊。
。。不太对。  这里是  .h  文件，  不是实现，  所以  fun  不应该有  {}  吧。

。。void  形参  是因为  兼容  早期的  K&C  标准。  K&C  中如果参数列表为空，则表明  可以传递任意的参数。  void  表示  不传任何参数。

。。C++标准  不需要  void  形参

---------------------------

C++  前向声明  是为了
避免头文件循环引用
避免引入头文件

头文件循环引用

```C++
#include "B.h"
class A {
private:
    B *b;
};

// ------

#include "A.h"
class B {

};
```

编译器在  编译到  B.cpp  时，  会先处理  A.cpp，然而  A.cpp  中引用了  B  会导致  循环引用。
解决方法，在  A.cpp  中  前向声明  B，  告诉编译器  B  是一个类。

```C++
#include "B.h"

class B;

class A {
private:
    B *b;
}
```

。。感觉这个  靠谱点。   之前的  那个  没有  #include "B.h"  ，怎么前向声明  B。
。。  也不对，  C++  编译的时候  只需要知道  是一个  类  就可以了，  在哪里  应该无所谓的。

。。  这里的话，  是  cpp文件，  需要  B  的  定义，所以  #include。  之前  是  .h  文件，只需要  知道个  名字，不需要知道定义。

。。但是  #include  后，  会循环依赖吧。

避免引入头文件
如果类A  用到了  某个内部的类，打包时  就不得不把  内部的头文件  也对外开放
库的开发者  肯定不希望  对外暴露  内部的细节，所以需要  利用  前向声明  跳过。

```C++
#include "B.h"

class A{
private:
    B *b;
}
```

可以改为  去除对B.h  的引用，  改为在  A.cpp  中  引用  B.h
```C++
class B;
class A{
private:
    B *b;
}
```

如果类A  在命名空间  test  中，则这样  声明
```C++
namespace test {
    class B;
}
```

----------------------------

编译器要确定没有拼写错误，并且传递给  函数的参数也是对的，  因此  编译器要求在  调用任何函数之前，必须  先看到  该函数的  声明。  简而言之，  任何  变量，函数  等，  都是要求  先声明  再使用。

。。。对，  有时候  定义和声明  是一起的，  但是  依然是  先声明(定义)  再使用。

函数声明  需要提供  返回类型，调用约定，方法名，参数，参数类型，
而定义  要求  有代码  实现。

通过使用 #include一些含有函数声明的头文件， 你可以将函数声明加到你的当前 .cpp文件或 .h文件中，然而，这会拖慢编译时间。尤其是如果在.h文件中#include 头文件时，这是因为，任何#include你写的.h文件的文件，结局是都将#include你所#include的头文件，编译器一下子需要编译无穷无尽的文件，即使需要调用的实际上只有一两个函数。

为了避免这种情况，你可以使用前向声明，也就是在文件顶部给出函数声明。对于大型项目，使用前向声明有可能使数小时的编译时间压缩为只需要几分钟。
。。但是  函数定义  不还是  得  #include  来导入？？？

```C++
// File Car.h
#include "Wheel.h"  // Include Wheel's definition so it can be used in Car.
#include <vector>
class Car
{
    std::vector<Wheel> wheels;
};
```

但是对于  头文件  Wheel.h，  必须提供  Car  的声明，因为  Wheel  包含了一个  指向  Car  的指针，但是不能在  文件  Wheel.h  中包含  Car.h  ，  如果包含，将产生  编译错误，因为头文件循环依赖。

解决这个问题的方法是  声明  Car，  不包含头文件。

```C++
// File Wheel.h
class Car;     // forward declaration
class Wheel
{
    Car* car;
};
```

如果类  Wheel  含有方法，  这些方法  需要  调用  Car  的方法，  那么  Wheel  的方法可以  定义在  Wheel.cpp  中，  Wheel.cpp  可以包含  Car.h，  不会导致  循环。

。。是头文件  包含头文件  才会尝试  依赖。  cpp  包含  头文件  不会。  cpp  包含  cpp  呢？

另外一个简单的例子，2个文件，  使用前向声明，不包含头文件，  可以编译通过。

```C++
// add.cpp:
int add(int x, int y)
{
    return x + y;
}

// -----------

// main.cpp:
#include <iostream>
int add(int x, int y); // 前向声明
int main()
{
    std::cout << "The sum of 3 and 4 is " << add(3, 4) << std::endl;
    return 0;
}
```
。。不过运行不了吧？  main  找不到  add  方法的实现啊。  除非  同一个  namespace？

============================

# volatile
程序每次  存储  或读取这个变量的时候，都会直接  从  内存中读取数据。
如果没有  volatile，  编译器可能会  优化  读取  和存储，可能  暂时  使用寄存器中的值。

不能用于  并发  的  原子性。

-------------------------

```C++
#include <stdio.h>
void main()
{
    int i = 10;
    int a = i;

    printf("i = %d", a);

    // 下面汇编语句的作用就是改变内存中 i 的值
    // 但是又不让编译器知道
    __asm {
        mov dword ptr [ebp-4], 20h
    }
    int b = i;
    printf("i = %d", b);
}
```

结果
i = 10
i = 10

```C++
#include <stdio.h>
void main()
{
     volatile int i = 10;
    int a = i;

    printf("i = %d", a);
     __asm {
        mov dword ptr [ebp-4], 20h
    }

    int b = i;
    printf("i = %d", b);
}
```

结果
i = 10
i = 32

volatile  使用场景
1. 中断服务程序中修改的  供其他程序检测的变量
2. 多任务环境下  各任务之间  共享的变量
3. 存储器映射的  硬件寄存器

4. 有些  变量  用  volatile  声明，  当2个线程  都需要  用到  某个变量  且  该变量的  值  会被改变时，  应该使用  volatile，  该关键字的  作用  是防止  优化器  将  变量  从  内存装入到  CPU寄存器。  如果变量被装入到  寄存器，  那么  2个线程  可能  一个使用  内存中的  变量，一个使用  寄存器中的  变量。

。。  ？  不过这个  不是原子性的，  可能  要求不是那么高的时候  可以用，  高并发  还是得加锁啊。

## volatile  指针
指向的对象是  volatile的
volatile char* vpch;

指针自己是  volatile的
char* volatile pchv

可以把一个  非  volatile int  赋给  volatile int，反之不行。
除了基本类型，  用户定义类型  也可以使用  volatile  类型修饰。

C++  中  一个有volatile  标识符  的类  只能访问  它接口的子集，  一个由类的  实现者  控制的子集。用户只能使用  const_cast  来获得  对  类型接口的  完全访问。  此外，  volatile  和  const  一样  会从  类传递到  它的成员。

============================
# assert

原型
void assert(int expression);

定义在  `<assert.h>`  中。

计算  expression  的值，如果为假(即0)，那么它  向  stderr  打印一条出错信息，然后通过调用  abort  来  终止程序。

在调试结束后，可以通过  #define NDEBUG  来  禁用  assert。

场景
1. 可以在  预计  正常情况下  程序不会到达的  地方  放置断言
2. 使用断言  测试  方法的  前置条件  和  后置条件
    1. 前置条件：代码执行前  必须具备的  特性
    2. 后置条件：代码执行后  必须具备的  特性
3. 使用断言  检测  类的不变状态，确保  任何情况下，某个变量的  状态或返回  必须满足。

注意事项
1. 每个assert  只检查一个条件，因为  同时检查多个条件  ，如果断言失败，无法直观判断  哪个条件失败。
2. 不能使用  改变环境  的语句。
3. assert  和后面的语句  应该  隔一行。
4. assert  不能  代替  条件过滤

----------------------------

assert  是一个  宏定义，  不是  函数。

c++中  include cassert  头文件，  cassert  最终引用  assert.h。

如果  assert  中的条件  返回错误，代码会  终止运行，并把  源文件，错误代码，行号，输出出来。

```C++
void func1() {
    int n = 2;
    assert(n==1);
}
```
Assertion failed: (n==1), function func1, file tempCodeRunnerFile.cc, line 6.

============================

# ==防御性编程==

防止  代码  被  错误地调用。

使用好的  编程风格  和  合理的设计
const  关键字
例如，在函数形参  前面添加  const，意味着  函数不会  修改这个参数，属于  输入参数。

volatile

在一些  并行的设备的  硬件寄存器  (如  状态寄存器)，  中断  服务  子程序  中  会访问到的  全局变量  以及  多线程应用中  被  几个任务  共享的  变量  前  使用  volatile  来防止  编译  优化。

static

函数体内的  static  变量的  作用域  是该  函数体，但是  不同于  auto  变量，该变量的  内存  只被分配  一次，因此  其值  在下次  调用时  仍然维持上次的值。

模块内的  static  全局变量  可以被  模块内的  所有函数  访问，但是  不能被  模块外的  其他函数  访问
模块内的  static  函数  只可能  被  这个模块内的  其他函数调用，这个函数的  使用范围  被限制在  声明它的  模块内。

位操作运算中，  尽可能使用  << >> & |，  少用  / % *

变量  和  函数的  命名要有意义，并且  尽可能做到  一个函数  只做一件事。

多采用  面向对象的  思想来  编写  代码

在编码前，考虑  大体的  设计方法。

不要仓促地编写代码
欲速则不达，  每敲  一个字，都要  清楚  你要输入的是什么。  在写每一行时  都三思而后行。  可能出现什么错误，是否考虑了所有的逻辑分支。

不要相信任何人
要用  怀疑的眼光  来  审视  所有的  输入  和  所有的结果，  直到你能证明这段代码  是正确的时候  为止。

编码的目标要清晰，而不是简洁
简单是一种美，不要让代码  过于复杂。即编写的代码  一定要  逻辑清晰，可读性强。

编译时打开所有警告开关

使用安全的数据结构
常见的一些安全隐患  大概是由  缓冲溢出  引起的。  缓冲溢出是由于  使用了  不正确的  使用固定大小的  数据结构  而造成的。
如下面的代码
```C++
char * unsafe_copy(const char * source)
{
   char *buffer = new char[10];
   strcpy(buffer,source);
   return buffer;
}
```
如果  source  中数据的长度超过  10字节，它就会造成问题。改成：
```C++
char * safe_copy(const char * source)
{
   char *buffer = new char[10];
   strncpy(buffer,source,10);  //用strncpy代替strcpy可以保护这个代码段
   return buffer;
}
```

检查所有的返回值

如果一个函数  返回一个值，这个值肯定有返回的理由。  检查这个返回值，如果  返回值  是一个  错误代码，你就  必须  识别  这个代码  并处理  所有的错误。  不要让  错误  悄无声息地  侵入  你的程序。  大多数  难以察觉  的错误  都是  因为程序员  没有  检查返回值  而出现的。

审慎地处理内存
对于  在执行期间  获得  的任何资源，必须彻底释放。

在声明位置初始化所有变量

编码时还需要注意一些细则
提供默认的行为：
switch  中  明确  default  子句。
如果  if  不带  else  子句，想一想  是否该处理  这个逻辑上的  默认情况。
数值的上下限，确保  运算不会溢出
强转  是否合理
声明变量，  可以  使  变量的  声明位置  和  使用位置  尽量接近，从而  防止  干扰代码的  其他部分。
加  合理的  异常处理，日志文件
正确设置常量。

    关心代码是否健壮
    确保每个设想都显示地体现在防御性代码中
    希望代码对无用信息的输入有正确的行为
    在编程的时候认真思考自己所编写的代码
    编写可以保护自己不受其他人的愚蠢伤害的代码。

============================

============================

# 内存对齐
元素是按照  定义顺序  一个个  放入到内存中的，但并不是紧密排列的。

从结构体  存储的  首地址开始，每个元素放到内存中时，  它都会认为  内存是  按照自己的大小  (  通常是4  或  8)  来划分的，因此  元素放置的位置  一定会在  自己宽度的  整数倍  上开始，这就是  所谓的  内存对齐。

理论上，  一个int 4字节，一个char 1字节，  但是  一个  包含  int  和char  的  struct  占8字节。

```C++
struct{
    int x;
    char y;
}Test;

int main()
{
    printf("%d\n",sizeof(Test)); // 输出8不是5
    return 0;
}
```

## 内存对齐的原因
平台原因(移植原因)
不是  所有的  硬件平台  都能访问  任意地址上  的  任意数据的。

性能原因

数据结构(尤其是栈)  应该尽可能  地  在  自然边界上对齐。因为  访问  未对齐的内存，处理器  需要2次  内存访问，而对齐的  内存访问  只需要一次。

如果没有内存对齐机制，数据可以任意存放，  现在  一个int  存放于  从地址1  开始  的  4个字节中，  处理器处理时，要  从  0地址读取  4个字节，剔除  地址0  的字节，然后  从地址4读取  4个字节，剔除  5,6,7的字节，然后  合并  ，放入  寄存器。

## 内存对齐规则
基本类型  的对齐值  就是  sizeof  值
数据成员对齐规则：

struct  或  union  的数据成员，第一个数据  存放于  offset  为0  的地方，以后  每个数据成员  的对齐  按照  #pragma pack  指定的值  和  这个数据成员自身长度  中，较小的那个  进行。

## 结构或联合的整体对齐规则

在数据成员  完成  对齐后，  结构/联合  自身  也需要对齐，  对齐  按照  #pragma pack  指定的数值  和  结构/联合  最大数据成员长度中，  比较小的  那个  进行。

```C++
struct
{
    int i;
    char c1;
    char c2;
}Test1;

struct{
    char c1;
    int i;
    char c2;
}Test2;

struct{
    char c1;
    char c2;
    int i;
}Test3;

int main()
{
    printf("%d\n",sizeof(Test1));  // 输出8
    printf("%d\n",sizeof(Test2));  // 输出12
    printf("%d\n",sizeof(Test3));  // 输出8
    return 0;
}
```

## 默认#pragma  pack(4)

```C++
struct S1
{
    int i:8;
    char j:4;
    int a:4;
    double b;
};

struct S2
{
    int i:8;
    char j:4;
    double b;
    int a:4;
};

struct S3
{
    int i;
    char j;
    double b;
    int a;
};

int main()
{
    sizeof(S1)=16
    sizeof(S2)=24
    sizeof(S3)=32
    return 0;
}
```


## 位域

在存储某些数据的时候，其实际需求的  数据长度  可能小于  1字节，只占  1bit  或  几  bit。为了  节省空间，c  中引入了  另一个结构，  称为  位预  或  位段

所谓  位域，就是  把  一个字节  中的  bit  按  实际的需求  分成  不同的  区域，表明  每个  区域的  位数，区域的域名，并允许  程序  按照  域名  进行操作。  如此  就可以把  不同的对象  用一个字节表示。

位域的长度  不能大于  1个字节。
。。跳

---------------------

结构体的总大小为结构体的有效对齐值的整数倍，结构体的有效对齐值的确定：
    当未明确指定时，以结构体或结构体所包含结构体成员中最长的成员长度为其有效值。
    当用#pragma pack(n)指定时，以n和结构体中最长的成员的长度中较小者为其值。
    当用__attribute__ ((packed))指定长度时，强制按照此值为结构体的有效对齐值。
    不管# pragma pack和__attribute__如何指定，结构体内部成员的自对齐仍然按照其自身的对齐值。

union  以  结构里  size  最大元素  作为  union  的size，  因为  在  任意时刻，union  只有一个  成员  真正存储于  该地址。

空类/静态成员
```C++
class A{
};

int main() {
    cout << sizeof(A) << endl;        // 1
}
```

有默认的构造，析构，所以  实际非空。
大小是1，  因为需要一个地址，  C++  不允许  2个不同对象  有相同的地址，  所以  C++  中  空类，空结构体  的大小都是1。

```C++
class A{
    A(){}
    ~A(){}
    void print() { printf("print()\n"); }
    void foo() { printf("print()\n"); }

    static void sprint() { printf("sprint()\n"); }
};

int main() {
    cout << sizeof(A) << endl;        // 1
}
```

类的  大小还是1，  成员函数，静态成员函数，静态成员变量  不占用  类的内存的，  因为这些东西  是类的，不是  每个对象分别存储的。
static  变量  就是  存储在  全局  静态区

需要注意，  子类继承  空类后，空基类的  1  个字节  并不会  加入到  子类中。
```C++
class Empty {};
struct D : public Empty {
    int a;
};
```

sizeof(D)  是  4

一个类包含  一个空类对象  数据成员，则  空类对象的  大小仍然是  1。
```C++
class Empty {};
class HaveAnInt {
    int x;
    Empty e;
}
```

大多数  编译器中，  sizeof(HaveAnInt)  的值  是  8。  这是  因为  Empty  类的  大小虽然是  1，  但是  为了  内存对齐，  编译器  会为  HaveAnInt  额外添加一些  字节。

默认情况下，类的  内存对齐  是以  class  中  最大的  那个  基本类型  为基准的，  如果  class  中  数据成员  包含  其他class，  则  递归  取  最大的  基本类型。

```C++
class BigData
{
    char array[33];
};

class Data
{
    BigData bd;
    int integer;
    double d;
};

cout << sizeof(BigData) << "   " << sizeof(Data) << endl;

class BigData
{
    char array[33];
};

class Data
{
    BigData bd;
    double d;
};

cout << sizeof(BigData) << "   " << sizeof(Data) << endl;
```

都输出  33 48

```C++
 class A {
 public:
     double len;
     char str[33];
 };

 class B {
 public:
     A a;
     int b;
 };

cout << sizeof(A) << "  " << sizeof(B) << endl;
```

输出  48 56
类  A  实际大小41字节，但是  8字节对齐，所以大小是  48

## 虚函数

C++的类中，如果有  虚函数，类内  就会有  一个  虚函数表  的  指针  _vptr，  指向自己的  虚函数表，_vptr  一般存放于  类的  最前面  (取决于  编译器)

```C++
class A {
public:
    A(){}
    virtual ~A(){}
    virtual void foo(){}
    virtual void print() {}
};
```

上面的  类  在  32位下  是  4  字节，  64位下  是  8字节。

## 继承
不同编译器  对  继承后  类的  大小  的计算方式  不同，  有的是  先继承后对齐，  有的是  先对其后继承。

```C++
class A
{
    int i;
    char c1;
}

class B:public A
{
    char c2;
}

class C:public B
{
    char c3;
}
```

sizeof(C)结果是多少呢，gcc和vs给出了不同的结果，分别是8、16。
    gcc中：C相当于把所有成员i、c1、c2、c3当作是在一个class内部，(先继承后对齐)

    vs中：对于A，对齐后其大小是8；对于B，c2加上对齐后的A的大小是9，对齐后就是12；对于C，c3加上对齐后的B大小是13，再对齐就是16 (先对齐后继承)

内存对齐的意义
效率
平台/移植

不能使用  memcmp(void*, void*)  来比较  2个  struct。
memcmp  是  逐字节比较，  struct  会进行内存对齐，内存对齐时  补的字节内容  是  垃圾值，所以  无法比较。

============================

memcmp  strcmp  都在  string.h  中

memcmp  是  按  字节对比
strcmp  是  按  char  对比。

============================

============================

# cv限定和引用限定

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

--------------------------
https://blog.csdn.net/WU9797/article/details/78167949

# cv限定
，就是  const  和/或  volatile。

const  有2种  用途：
1. 声明符号常量
2. 修饰常量的作用域  和  链接性
C++中，文件中  声明的  全局变量  默认  链接性是外部的，  但是  使用  const  定义的  全局变量  链接性  是内部的，如：

const int time = 12;     则常量  time  只在  声明它的文件  或代码段内可见，就像在定义时  添加了  static一样：  static const int time = 12;

假设一个常量  需要在  默认作用域  的外部  使用，则可以使用  extern  关键字  来  修改  默认的  内部链接性：  extern const int time = 12;    这样  在一个文件中声明的常量  time  就可以在  另一个  文件中使用，需要注意的是，在其他文件中使用time  时  必须使用  extern：    extern int time。   因为  time  在多个  文件中  共享，所以  只能有  一个文件  对其进行  初始化。

volatile  的作用就是改善  编译器的  优化能力
这个关键字表明，  即使程序代码  没有对  内存单元  进行修改，其值  也可能发生变化。

============================
https://blog.csdn.net/www_dong/article/details/116102066

# static

经过static  修饰的变量，存储在  内存的  全局静态区。  且被  static  修饰的  变量  只能在  本模块  的  所有函数  ref。

内存中  存储区域如下：
堆区

由  程序员  手动  new  和  delete  的内存区域。由  低地址  向  高地址  申请；  内存空间大；存储地址不连续；一般是  链式的；速度慢。

栈区

由  编译器自动  分配  和释放，主要存储  函数的  参数值，函数内部的  变量的值，函数调用的  空间。  从高地址  向  低地址  申请；容量优先；速度较快，存储地址连续，会溢出。

代码区
又称  文本段，存放着  程序的  机器代码，可执行指令  就存储在这里，  这里的代码  是只读的。
全局区(静态区)

全局变量  和  静态变量  存储在这里。  初始化的全局变量  和  静态变量  在  一块区域  (.data)，  未初始化的  全局变量  和  未初始化的  静态变量  在  相邻的  另一块区域(.bbs)。程序结束后，由  系统释放。

常量区
常量字符串  放在这里，程序结束后，由系统进行释放。

static  用法主要体现在  2个方面：  面向过程中的  static  和  面向对象的static
面向过程  的  主要包括  静态全局变量，静态局部变量  和  静态函数
面向对象  主要包括  静态成员变量，静态成员函数。

## 静态全局变量
全局变量前面添加  static  ，就变成了  静态全局变量。

```C++
#include <stdio.h>

static int a = 10;
```

特点：

1. 在全局数据中的  变量如果没有显式的  初始化  会自动  被程序初始化为0 (  这个特性  非静态全局变量也有)，  而在  函数体内  声明的变量  如果  不显式初始化  则会使用  一个  随机值

2. 静态全局变量  在  声明它的  整个文件中  都可见，  这个文件外不可见。
3. 静态全局变量  在  全局数据区  分配内存
4. 其他文件中  可以定义  相同的变量，不会冲突。

如果将static去掉，具有以下特点

1. 全局变量  默认  是具有  外部链接性  的，作用域  是整个工程，  在一个  文件内定义的  全局  变量  可以通过  包含  其所在  头文件  或  显式调用  extern  关键字  修饰  全局变量  的变量名  声明来引用。

2. 静态全局变量  是  显式调用static修饰的  全局变量，其作用域  只在  声明此变量的  文件中，其他  文件  即使使用  extern  也不可用。

## 静态局部变量
局部变量前面加  static。

```C++
#include <stdio.h>

void Func()
{
    static int a = 5;
    printf("a = %d\n", a);
    a++;
}
```

通常，在一个函数作用域内  定义  一个变量，  每次运行到  该函数时，系统会给  局部变量分配  内存。当函数  结束时，该变量的  内存会被  系统  回收到  栈内存中。

特点
1. 内存  存放在  程序的  全局数据区  中
2. 静态局部变量  在程序  首次  执行到  该  对象声明时，  会被  初始化。  其后  运行到  该对象的  声明时，  不会  再次初始化。
3. 如果  静态局部变量  没有  显式初始化，  自动赋  0  。
4. 局部  静态变量  不能被  其作用域之外  的其他模块调用，其调用范围  仅限于  声明该变量  的函数作用域当中。

## 静态函数
函数返回值  前面  加  static
```C++
#include <stdio.h>

static void Func()
{
    printf("This is a static function\n");
}
```

特点
1. 作用域  只在  声明它的  文件中，不能被其他  文件引用，其他文件可以定义  同样的全局函数
2. 其他  文件  要调用  本文件的  静态函数，需要  显式  extern  修饰  静态函数的声明。

面向对象中的  static

## 静态成员变量

```C++
#include <iostream>
#include <stdio.h>
using namespace std;

class Test
{
public:
    Test(int a, int b, int c) :
        m_a(a),
        m_b(b),
        m_c(c)
    {
        m = a + b + c;
    }

    void Show()
    {
        cout << "m = " << m << endl;
    }

private:
    int m_a, m_b, m_c;
    static int m;
};

int Test::m = 0; //初始化静态数据成员

int main()
{
    Test ClassA(1, 1, 1);
    ClassA.Show();  //输出: 3
    Test ClassB(3, 3, 3);
    ClassB.Show(); //输出: 9
    ClassA.Show(); //输出: 9

    system("pause");
    return 0;
}
```

特点
1. 静态数据成员  的服务对象  并非  单个  类实例化的对象，而是  所有  类实例化的对象  (  这点可以用于实现  单例模式)
2. 静态数据成员  必须  显式  初始化分配内存，在其包含类  没有任何实例化之前  已经有  内存分配。
3. 静态数据成员  和其他成员一样，  遵守  public,protected,pricate  的访问规则
4. 静态数据成员  内存  存储于  全局数据区，只  随着  进程的  消亡而消亡。

优势
1. 静态数据成员  不进入  程序  全局命名空间，不会与  其他全局命名空间  的同名同类型  变量冲突
2. 静态数据成员  可以实现  C++  的封装特性，由于它  遵守  类的  访问权限，所以  比  全局  变量  更加灵活。

## 静态成员函数

类成员函数  的返回类型  前面加  static
```C++
class Test
{
public:
    static void Show()
    {
    cout << "m = " << m << endl;
    }
```

特点
1. 静态成员函数  比  普通成员函数  多一种调用方式
2. 在没有  实例化的类对象  的情况下  可以调用  类的  静态成员函数
3. 静态成员函数  中没有  隐含的  this  指针，  所以  静态成员函数  不可以操作  类中的  非静态成员。   。。。  传进来就可以了。

## 总结
1. 静态数据成员  都是静态存储的，所以必须在  main函数  之前  显式  初始化

2. 可以在  头文件中  声明  静态全局变量，该头文件  被  多个  cpp  文件包含后，包含  该头文件的  cpp  文件  实际上  会  各自  拥有  独立的  同名变量

3. 不能将静态成员函数  定义为  虚函数
4. 静态成员函数  没有this  指针
5. static  缩短了  子类  对  父类静态成员访问的  时间，相对来说  节省了  内存空间

6. 如果不想在  子类中  操作父类的  静态成员，则可以在子类  中定义一个  同名的  static  成员。这样可以  覆盖  父类中的  静态成员，并且根据C++的  多态性变量命名规则，这样做是安全的

7. 静态成员  声明  在类中，  操作在其外部，所以  对其  取地址操作  就跟  取普通成员的操作  略有不同。  静态变量地址  是指向  其数据类型的指针，函数地址  则是一个  类型为  nonmember  的函数指针

----------------------
https://zhuanlan.zhihu.com/p/394975612

# static

三种完全不同的用途
1. 表示  一个静态储存期  的变量，且  具有  internal linkage，俗称  static  变量
2. 表示  一个  静态储存期  的变量，但是  只在  一个局部作用域中  可见，保证  仅初始化一次，俗称  static local  变量
3. 表示一个  class  的静态成员，  整个class  共享一份，不绑定到  任何  instance，俗称  static member  变量

## static  变量
在main函数开始之前  初始化，存活到  main函数  结束
`static int a; // static 变量`

## static local  变量
在一个  函数体  (严格来说是  block scope)  内部  ，用  static  变量  就是  static local变量
```C++
void func() {
    static int a; // static local 变量
}
```
这类变量  一般在第一次允许到这段代码时  进行初始化。(付出的代价就是  每次运行到这里  都需要检查  是否已经初始化)
并且这个初始化是多线程安全的(从C++11开始)，  可以用来做  线程安全的单例(常称为  Meyers  单例模式)

## static member  变量
这个是  本文重点。
先讨论  C++14  及之前的情况
声明一个  静态成员变量  很简单：
```C++
class A {
    static int a;
}
```

问题来了，当这段头文件  被  多个翻译单元  include  的时候，这个符号  a  放在  哪个文件里？
C++的方案：  再加一条语句  专门  用来生成  符号，顺便可以  初始化
```C++
// A.h (可以被多次包含)
class A {
    static int a; // 不生成符号
}

// A.cpp (只翻译一次)
#include "A.h"
int A::a; // 生成一个符号
```

那没有符号可以吗? 没有符号的东西没法放进内存, 所以也没法取地址, 没有办法 ODR-used.

但, 我们这个例子中, 如果只是为了读取 A::a 的值, 那完全可以不把 A::a 放进内存, 那就不需要这个符号了. 而某些情况下确实不需要放进内存.
```C++
// A.h
class A {
    static const int a = 0;
    // 不生成符号
    // 但是可以编译时读 a 的值
}
```

按C++  的设计，只有  const  的  static member  变量  支持  类内初始化，  如果  不在类外  提供一个  符号，那么对  A::a  取地址的时候，  链接器会  提示  没有符号  ( undefined reference to A::a)

inline  的  static member  变量
C++17  引入了  inline  的  static member  变量。
上面说  总不能每个  翻译单元  都有一个  符号吧，但  到  C++17  后  是可以的。
加了  inline  后，就可以  每个  翻译单元生成  一个符号，  然后  链接器典型实现  是选其中一个。

加了 inline 以后就不需要专门在 A.cpp 生成一个符号了!
```C++
// A.h (可以被多次包含)
class A {
    static inline int a;
    // 生成符号, 可以取地址
    // 并且支持类内初始化!
}
```

C++17 之后, constexpr 的 static member 变量默认 inline , 所以你可以写出这样的代码:

```C++
// A.h (可以被多次包含)
class A {
    static constexpr int a = 0;
    // 生成符号, 可以取地址
    // 并且支持类内初始化!
}
```

新版 GCC 和 Clang 在 C++11/14 也能用 inline 变量, 所以 C++11/14 的时候 constexpr 也能推出 inline . 但是 MSVC 用不了

============================

# inline

内联函数

inline必须==和函数定义放一起==，  放  函数声明前面  不起作用。

定义在  类声明  中的  成员函数  ==自动成为==  内联函数。
但编译器  是否  真正内联  要看  函数是如何定义的

内联函数  ==应该在头文件中定义==，这一点  不同于  其他函数。编译器  在  调用点  内联展开  函数的  代码时，  必须能够找到  inline  函数的  定义  才能将调用函数  替换为  函数代码，而对于  在头文件中  仅有  函数声明  是不够的。

当然内联函数定义也可以放在源文件中，但此时只有定义的那个源文件可以用它，而且必须为每个源文件拷贝一份定义(即每个源文件里的定义必须是完全相同的)，当然即使是放在头文件中，也是对每个定义做一份拷贝，只不过是编译器替你完成这种拷贝罢了。但相比于放在源文件中，放在头文件中既能够确保调用函数是定义是相同的，又能够保证在调用点能够找到函数定义从而完成内联(替换)。

慎用内联

============================

============================

https://mp.weixin.qq.com/s?__biz=MzkxNzQzNDM2Ng==&mid=2247493445&idx=2&sn=6554b92ec0bfc3210035ce03020e1a9e&source=41#wechat_redirect

http://www.broadview.com.cn/article/420006

https://blog.csdn.net/weixin_41055260/article/details/121413732
这篇文章传播挺广的，不知道第一篇是谁。  我按  weixin上的抄的

。。。md。。broadview  的内容是正确顺序的，但是没有分段，其他的2个，内容顺序错乱的。。看都看得差不多了。

# C++对象模型

关于对象
C语言是程序性的，语言本身并没有支持数据和函数之间的关联性。

C++  中可能采取  抽象数据类型  或者是  多层次的类  结构完成。

C++  的封装  并没有  增加多少  成本，每个  成员函数  虽然在  class  中声明，但  不会在  每个对象中出现：
> 每个  非内联  的成员函数  只会诞生  一个函数实例
> 每个  内联函数  会在  其每一个  使用者身上  产生一个  函数实例。

C++  在布局以及存储时间上  主要的额外负担是  由  virtual  引起的
虚函数机制  用来支持一个  有效率的  执行期绑定
虚基类  用来实现：多次出现在  继承关系中的基类，有一个  单一  且  被共享  的实例

还有一些  多重继承下的  额外负担，发生在  一个派生类  和  其第二或后继之基类的转换  之间。

## C++对象模式
每个类中  存放一个  指针vptr，  指向  虚函数表
表中每个都指向一个虚函数
非静态数据成员  放在  类对象中
静态数据成员  放在  类对象外
静态  和  非静态  成员函数  放在  类对象外
虚函数则不同

关键词所带来的差异
int ( *pq ) ();    //  声明
当语言无法区分  声明和表达式时，  我们需要一个超越  语言范围的  规则，而这个规则  会将上述  式子  判断为  一个  声明
struct  和  class  可以互换，他们只是  默认的  权限不同。

如果  你需要  拥有  C  声明的那种  struct  布局，  可以抽出来  单独  称为  struct  声明，并且  和  C++  部分组合  起来。

对象的差异
C++支持  3种程序范式：  程序模型，抽象数据类型模型，面向对象模型。
面向对象模型  在继承体系中，有时候  编译期  无法确定指针  或  ref  所指的类型。

C++  支持的多态类型：
经由一组隐式的  转化操作：  如  派生类指针  转化为  指向  父类的指针
经由虚函数机制
经由  dynamic_cast  和  typeid  运算符

### 一个class所占的大小包括：
其  非静态成员  所占的大小
由于  内存对齐  填补上的大小
加上支持  虚函数  产生的大小

指针的类型，只能代表  其让编译器  如何解释  其所指向的地址的  内容，和它本身类型无关，所以  转换其实是一种  编译器指令，不改变  所指向的地址，只影响怎么解释  它给出的地址。

。。如果是单继承，好像很好理解，就是  指针的地址，前  10  个byte  是  顶层类，  前  20个byte  是  次顶层类，  。。依次类推，   因为  子类  肯定包含  父类的信息，  所以将  父类的信息  放在  最前面，  这样的话，  转换指针类型，只是  转换了  一次读取  多少个  byte，以及这些byte  的  类  类型。

。。但是多继承，  2个  顶层类，内存上  肯定一个  在前，一个在后，  需要跳过一些东西的。就。。
。。但是不知道  内存究竟是怎么样的。

当一个  基类对象  被初始化为  一个子类对象时，派生类  就会  被切割  以  塞入  基类内存中，派生类不会留下任何东西，多态  也不会呈现。
。。这个学名叫什么？  见过，忘了。找不到。

默认构造函数的构造操作
### 下面4种情况，会合成有用的构造函数：
1. 类声明  (或继承)  一个虚函数
2. 类派生一个继承串链，其中有一个或更多的虚基类
3. 带有默认构造函数的成员函数对象，不过这个合成操作只有在构造函数真正需要被调用时才发生，但只是调用其成员的默认构造函数，其他则不会初始化

4. 如果一个派生类的父类带有默认构造函数，那么子类如果没有定义构造函数，则会合成默认构造函数，如果有的话但是没有调用父类的，则编译器会插入一些代码调用父类的默认构造函数

5. 带有一个虚函数的类
6. 带有一个虚基类的类

C++新手常见的2个误解
1. 任何class  如果没有定义默认构造函数，就会被合成出来一个
2. 编译器合成出来的  默认构造函数  会显式  设定类中的  每个数据成员的  默认值。

。。。？？？  对于1，  Leetcode的  LTXXX  不就是  没有默认的构造函数，但是可以new啊。

拷贝构造函数的构造操作
## 有3种情况会调用拷贝构造函数
1. 对一个对象  做显式的初始化操作
2. 当  对象被当做  参数  交给某个函数
3. 当函数  传回一个类对象时。

如果类没有声明  一个  拷贝函数，就会  有  隐式的声明  和  隐式的定义  出现，同  默认构造函数  一样  在使用时  才合成出来。
。。。？上面说  默认构造器  时  ，没有说  是  使用时  才合成吧？

。。话说，如果  只是定义，  不new，  那么  会生成  默认构造器？   应该会吧，  可能反射之类的，  就是  之前  Modern CPP  中  看到的  boost pb

什么情况下  一个类  不展现  "浅拷贝语义"

1. 编译器  会  合成一个  拷贝构造  函数，安插一些  代码用来设定虚基类指针和偏移的初值，对每个成员执行必要的深拷贝初始化操作，以及执行其他的内存相关工作

2. 编译器会  显式的  设定  新类的  虚函数表，而不是直接拷贝过来指向同一个。。。但是  虚函数表  指向的  函数  是不是(可能)相同的？
3. 这两个  编译器  都会  合成  拷贝构造函数  并且  安插进那个成员和基类的拷贝构造函数

。。。感觉就是  机翻的。。

1. 当类内含有一个成员类而后者的类声明中有一个拷贝构造函数（例如内含有string成员变量）
2. 当类继承自一个基类而基类中存在拷贝构造函数
3. 当类声明了一个或多个虚函数
4. 当类派生自一个继承串链，其中有一个或多个虚基类

程序转换语意学

在将  一个类  作为  另一个类  的初值  情况下，语言允许编译器有  大量的  自由发挥的  空间，用来  提升  效率，但是  缺点  是  不能安全的  规划  拷贝构造函数  的副作用，必须视其执行而定

。。。跳了，  挑一些看得懂的。

## 四种情况下你需要使用成员初始化列表
    当初始化一个引用成员变量
    当初始化一个const 成员变量
    当调用一个基类的构造函数，而它拥有一组参数
    当调用一个类成员变量的构造函数，而它拥有一组参数

```C++
class Word{
 String _name;
 int _cnt;
public:
 Word(){
  _name = 0;
  _cnt = 0;
 }
/*使用成员列表初始化可以解决
     Word() : _name(0)，_cnt(0){  }
*/
}
```

上式不会报错，但是会有效率问题，因为这样会先产生一个临时的string对象，然后将它初始化，之后以一个赋值运算符将临时对象指定给_name，再摧毁临时的对象

成员初始化列表中的初始化顺序是按照类中的成员变量声明的顺序，与成员初始化列表的排列顺序无关
```C++
class X{};
class Y : public virtual X {};
class Z : public virtual X {};
class A : public Y,public Z {};

sizeof(X) //1
sizeof(Y) //4
sizeof(Z) //4
sizeof(A) //8
```

X为1是因为编译器的处理，在其中插入了1个char，为了让其对象能在内存中有自己独立的地址
Y，Z是因为虚基类表的指针
A 中含有Y和Z所以是8

每一个类对象大小的影响因素：
    非静态成员变量的大小
    virtual特性
    内存对齐

如果类的内部有typedef,请把它放在类的起始处，因为防止先看到的是全局的和这个typedef相同的冲突，编译器会选择全局的，因为先看到全局的
。。会？  而且冲突  也会  编译失败吧。

     非静态成员变量的在内存中的顺序和其声明顺序是一致的
    但是不一定是连续的，因为中间可能有内存对齐的填补物
    virtual机制的指针所放的位置和编译器有关

静态变量都被放在一个全局区，与类的大小无关，正如对其取地址得到的是与类无关的数据类型，如果两个类有相同的静态成员变量，编译器会暗自为其名称编码，使两个名称都不同

。。那么，  这个地址  是代表  类A  还是  类B，  这个信息在哪里？  根据上面的  A  的  sizeof  是1，  看来，并不是保存在  类中的。

。。就是  怎么确定  这个地址  是  类A  还是  类B。

。。sizeof  是  1，  那就意味着  需要一张表  来  保存  哪个地址  是哪个类。  而不是放在类中，不不不，可能是  隐藏字段，不，int之类的  就是连着的，这个要自己试下，  打印地址，打印sizeof。

非静态成员变量则是直接放在对象内，经由对象的地址和在类中的偏移地址取得，但是在继承体系下，情况就会不一样，因为编译器无法确定此时的指针指的具体是父类对象还是子类对象

。。那总要确定的啊。

把数据放在同一个类中和继承起来的内存布局可能不同，因为每个类需要内存对齐

当加上多态之后，对空间上增加的额外负担包括：
    析构函数的调用顺序是反向的，从子类到父类
    导入一个虚函数表，表中的个数是声明的虚函数的个数加上一个或两个slots(用来支持运行类型识别)
    在每个对象中加入vptr，提供执行期的链接，使每一个类能找到相应的虚函数表
    加强构造函数，使它能够为vptr设定初值，让它指向对应的虚函数表，这可能意味着在派生类和每一个基类的构造函数中，重新设定vptr的值
    加强析构函数，使它能够消抹“指向类的相关虚函数表”的vptr,vptr很可能以及在子类析构函数中被设定为子类的虚表地址。

    以下是三种情况不同的继承下会有不同的布局
vptr  被放在  class  尾端
vptr  被放在class  前端
单一继承  并含  虚拟函数情况下的  数据分布  (vptr在父类的尾端)

### 多重继承
单一继承特点：  派生类  和  父类对象  都是从相同的地址开始，区别只是  派生类  比较大  能容纳它自己的  非静态成员变量。

多重继承下  比较复杂
一个派生对象，把它的地址指定给最左边的基类，和单一继承一样，因为起始地址是一样的，但是后面的需要更改，因为需要加上前面基类的大小，才能得到后面基类的地址

程序员如果关心程序效率，应该实际测试，不要光凭推论、常识判断或假设。
优化操作并不一定总是能够有效运行，我不止一次以优化方式来 编译一个已通过编译的正常程序，却以失败收场

vptr通常放在起始处或尾端，与编译器有关，C++标准允许放在类中的任何位置

取某个类成员变量的地址，通常取到得的是在类的首地址的偏移位置
例如 & Point3d::z; 将得到在类的偏移位置，最低限度是类的成员大小总和，而这个偏移量通常都被加上了1
。。Point3d  是  类，不是  对象

如果用一个真正绑定类对象（也就是使用 . 操作符访问成员变量）去取地址，得到的将会是内存中真正的地址

在多重继承下，若要将第二个基类的指针和一个与派生类绑定的成员结合起来，那么将会因为需要加入偏移量而变得相当复杂

Function 语意学
    C++支持三种类型的成员函数：static 、non-static 、virtual
        不能直接存取non-static数据
        不能被声明为const
        static函数限制：

成员函数的各种调用方式
非静态成员函数：C++会保证至少和一般的普通的函数有相同的效率，经过三个步骤的转换
    改写函数，安插一个额外的参数到该函数中，用来提供一个存取管道------即this指针
    对每一个非静态成员的存取操作改成使用this指针来调用
    将成员函数改写成一个外部函数，并且名称改为独一无二的
。。这个改写了干什么？  直接调用  不就完了，  为什么要改成  普通函数

虚函数成员函数：也会经过类似的转化
    vptr是编译器产生的指针，指向虚函数表，其名称也会被改为独一无二
    1 是该函数在虚函数表中的索引
    ptr 则是this指针
     例如：ptr->normalize()会被转化为( * ptr->vptr[1]  )( ptr )

静态成员函数：它没有this指针，因此会有以下限制：
    它不能直接存取类中的非成员变量
    它不能够被声明为const、volatile 和 virtual
    它不需要经过类的对象才能被调用----虽然很多事情况是这样调用的

详解虚成员函数
虚函数一般实现模型：每一个类都有一个虚函数表，内含该类中有作用的虚函数地址，然后每个对象有一个虚函数指针，指向虚表位置
多态含义：以一个基类的指针（或引用），寻址出一个子类对象

什么是积极多态？
当被指出的对象真正使用时，多态就变成积极的了

一个类派生自Point会发生什么事？三种可能的情况
    它可以继承基类所声明的虚函数的函数实例，该函数实例的地址会被拷贝进子类的虚表的相对应的slot之中
    它可以使用自己的虚函数实例----它自己的函数实例地址必须放在对应的slot之中
    它可以加入一个新的虚函数，这时候虚函数表的尺寸会增加一个slot，而新加入的函数实例地址也会被放进该slot之中

指向成员函数的指针
对于普通的成员函数，编译器会将其转化为一个函数指针，然后使用成员函数的地址去初始化
    例如  double (Point :: *pmf ) ( );
    转化为  double ( Point :: coord )( ) = &Point :: x;
    这样调用  (coord) (& origin)或 (coord)(ptr)

对一个虚函数取地址，在vc编译器下，要么得到vacll thunk地址（虚函数时候），要么得到的是函数地址（普通函数）

inline只是向编译器提出一个请求，是否真的优化取决于编译器自己的判定

对于形式参数，会采用：
    常量表达式替换
    常量替换
    引入临时变量来避免多次求值操作

对于局部变量，会采用：
    使用临时变量

构造、析构、拷贝语意学

应注意的一些问题：
        构造函数不要写为纯虚函数，因为当抽象类中有数据的时候，将无法初始化
        把所有函数设计成虚函数，再由编译器去除虚函数是错误的，不应该成为虚函数的函数不要设计成虚函数
        当你无法抉择一个函数是否需要为const时，尤其是抽象类，最好不设置成const

"无继承" 情况下的对象构造
对于没有初始化的全局变量，C语言中会将其放在一个未初始化全局区，而C++会将所有全局对象初始化

对于不需要构造函数、析构函数、赋值操作的类或结构体，C++会将其打上POD标签，赋值时按照c那样的位搬运

对于含有虚函数的类，编译器会在构造函数的开始放入一些初始化虚表和虚指针的操作

面对函数以值方式返回，编译器会将其优化为加入一个参数的引用方式，避免多次构造函数
。。？

## 继承体系下的对象构造

构造函数会含有大量的隐藏码，因为编译器会进行扩充：
    如果类被列于成员初始化列表中，那么如果有任何显式指定的参数，都应该传递过去。若没有列于list中，而类中有一个默认构造，亦应该调用
    此外，类中的每一个虚基类的偏移位置必须在执行期可被存取
    如果类对象是最底层的类，其构造函数可能被调用，某些用以支持这一行为的机制必须被放进来
    如果基类被列于成员初始化列表中，那么任何显式指定的参数应该传递过去
    如果基类没有被列于基类初始化列表中，而它有默认的构造函数，那么就调用
    如果基类是多重继承下的第二个或后继的基类，那么this指针必须有所调整
    记录在成员初始化列表中的成员数据初始化会被放进构造函数的本体，并以成员在类中声明的顺序为顺序
    如果有一个成员并没有出现在成员初始化列表中，但是它由一个默认构造函数，那么也会被调用
    在那之前，如果类对象由虚表指针，它必须被设定初值，指向适当的虚表
    在那之前，所有上一层的基类构造函数必须被调用，以基类声明的顺序（不是成员初始化列表出现的顺序）
    在那之前，所有虚基类构造函数必须被调用，从左到右，从最深到最浅

不要忘记在赋值函数中，检查自我赋值的情况

对象复制语意学
当一个类复制给另一个类时，能采用的有三种方式：
如果有需要，会自动生成一个浅拷贝，至于什么时候需要深拷贝（见第二章讲）
    什么都不做，会实施默认行为
    提供一个拷贝复制运算符
    显式地拒绝把一个类拷贝给另一个

虚基类会使其复制操作调用一次以上，因此我们应该避免在虚基类中声明数据成员

析构语意学
什么时候会合成析构函数？
    在类内含的成员函数有析构函数
    基类含有析构函数
。。不是默认生成？

## 析构的正确顺序：
    析构函数的本体首先被执行，vptr会在程序员的代码执行前被重设。
    如果类拥有成员类对象，而后者拥有析构函数，那么他们会以其声明顺序的相反顺序被调用
    如果类内涵一个vptr,现在被重新设定，指向适当的积累的虚表
    如果有任何直接的非虚基类拥有析构函数，它们会以其声明顺序的相反顺序被调用
    如果有任何虚基类拥有析构函数，而且目前讨论的这个类是最尾端的类，那么它们会以其原来的构造顺序的相反顺序被调用

执行期语意学
C++难以从程序源码看出表达式的复杂过程，因为你并不知道编译器会在其中添加多少代码

编译器对不同的对象会做不同的操作：

。。。从这里开始  看/抄  broadview  的文章。

对于全局对象：编译器会在添加__main函数和_exit函数(和C库中的不同)，并且在这两个函数中对所有全局对象进行静态初始化和析构

全局对象编译器操作
使用被静态初始化的对象，有一些缺点：
如果异常处理被支持，那么那些对象将不能被放置到try区段之内增加了程序的复杂度。因此，不建议用那些需要静态初始化的全局对象。

对于局部静态对象：

会增加临时的对象用来判断其是否被构造，用来保证在第一次进入含有该静态对象的起始处调用一次构造函数，并且在离开文件的时候利用临时对象判断是否已经被构造来决定是否析构

对于对象数组:（例如Point knots[10]）
如果对象没有定义构造函数和析构函数，那么编译器只需要分配需要存储10个连续空间即可
如果有构造函数，且如果有名字，则会分为是否含有虚基类调用不同的函数来构造，如果没有名字，例如没有knots，则会使用new来分配在堆中
当声明结束时会有类似构造的析构过程
我们无法在程序中取出一个构造函数的地址

new 和 delete 运算符

。。。不分段，真的是  句读之不知

对于普通类型变量：
例如int pi = new int(5)
调用函数库中的new运算符if(int pi = _new (sizeof(int) ))
再配置初值 *pi = 5

对于delete来说 delete pi;
则先进行保护 if( pi != 0)
再调用delete：_delete(pi)

对于成员对象：

对于Point3d* origin = new Point3d

实际调用operator new，其代码如下

```C++
extern void* operator new (size_t size){
    if(size == 0)
        size = 1;
    void * last_alloc;
    while(!(last_alloc = malloc(size))){
        if(_new_handler)
            ( *_new_handler)();
        else
            return 0;
    }
    return last_alloc;
}
```

语言要求每一次对new的调用都必须传回一个独一无二的指针，为了解决这个问题，传回一个指向默认为1Byte的内存区块，允许程序员自己定义_new_handler函数，并且循环调用

至于delete也相同
```C++
extern void operator delete (void *ptr){
 if(ptr)
  free( (char*)ptr)
}
```

对于对象数组，会在分配的内存上方放上cookies，来存储数组个数，方便delete调用来析构

程序员最好避免以一个基类指向一个子类所组成的数组——如果子类对象比其基类大的话
解决方式：
```C++
for(int ix = 0; ix < elem_count; ++ix){
 Point3d *p = &((Point3d*)ptr)[ix];
 delete p;
}
```

程序员必须迭代走过整个数组，把delete运算符实施与每一个元素身上。以此方式，调用操作将是virtual。因此，Point3d和Point的析构函数都会实施与每一个对象上

。。weixin  和  broadview  混合看。

### placement new
Placement Operator new的语意
有一个预先定义好的重载的new运算符，称为placement operator new。它需要第二个参数，类型为void*

形如 `Point2w *ptw = new (arena) Point2w`,其中arena指向内存中的一个区块，用以放置新产生出来的Point2w对象
```C++
void* operator new(size_t , void* p){
 return p;
}
```

。。我感觉  这不像是  C++。。都是什么功能。。
。。应该是  原地new，这样的话，减少  delete，减少一次  拷贝，内存分配。
。。真第一次见。

。。不过  arena  需要  也是  Point2w  吗？  还是  随便一个  对象都可以？  但是这样的话，就把  arena  原先的类型  变了，问题很大吧。

。。placement new操作的作用就是：创建对象(调用该类的构造函数)但是不分配内存，而是在已有的内存块上面创建对象。用于需要反复创建并删除的对象上，可以降低分配释放内存的性能消耗。

如果我们在已有对象的基础上调用placement new的话，原来的析构函数并不会被调用，而是直接删除原来的指针，但是不能使用delete 原来的指针
正确的方法应该是 :

```C++
//错误：
delete p2w;
p2w = new(arwna) Point2w;
//正确：
p2w->Point2w;
p2w = new(arena) Point2w;
```

临时性对象
临时对象在类的表达式并赋值，函数以值方式传参等都会产生临时对象——-而临时对象会构造和析构，所以会拖慢程序的效率，我们应该尽量避免

站在对象模型的顶端
三个著名的C++语言扩充性质：模板、异常（EH）、RTTI（runtime type identification）

## Template
模板实例化时间线
当编译器看到模板类的声明时，什么都不会做，不会进行实例化
模板类中明确类型的参数，通过模板类的某个实例化版本才能存取操作
即使是静态类型的变量，也需要与具体的实例版本关联，不同版本有不同的一份如果声明一个模板类的指针，那么不会进行实例化，但是如果是引用，那么会进行实例化
对于模板类中的成员函数，只有在函数使用的时候才会进行实例化

模板名称决议的方法——-即如果非成员函数在类中调用，那么会调用名称相同的哪个版本：
会根据该函数是否与模板有关来判断，如果是已知类型，那么会在定义的范围内直接查找，如果依赖模板的具体类型，那么会在实例化的范围查找

## 异常处理
C++异常处理由三个主要的语汇组件：
一个throw子语。它在程序某处发出一个exception，exception可以说内建类型也可以是自定义类型

一个或多个catch子句。每一个catch子句都是一个exceotion hander，它用来表示说，这个子句准备处理某种类型exception，并且在封闭的大括号区段中提供实际的处理程序

一个try区段。它被围绕一系列的叙述句，这些叙述句可能会引发catch子句起作用

当一个异常被抛出去，控制权会从函数调用中被释放出来，并寻找一个吻合的catch子句。如果都没有吻合者，那么默认的处理例程 terminate()会被调用，当控制权被放弃后，堆栈中的每一个函数调用也就被推离。这个程序被称为 unwingding the stack 。在每一个函数被推离堆栈之前，函数的局部类的析构会被调用

因此一个解决办法就是将类封装在一个类中，这样变成局部类，如果抛出异常也会被自动析构

当一个异常抛出，编译系统必须：
1.检查发生throw操作的函数
2.决定throw操作是否发生在try区段
3.若是，编译系统必须把异常类型拿来和每一个catch子句进行比较
4.如果比较后吻合，流程控制应该交到catch子句中

5.如果throw的发生并不在try区段中，或没有一个catch子句吻合，那么系统必须：摧毁所有活跃局部类 从堆栈中将目前的函数(unwind)释放掉 进行到程序堆栈的下一个函数中去，然后重复上述步骤2—5

## 执行期类型识别
当两个类有继承关系的时候，我们有转换需求时，可以进行向下转型，但是很多时候是不安全的

C++的RTTI (执行期类型识别)提供了一个安全的向下转型设备，但是只对多态（继承和动态绑定）的类型有效，其用来支持RTTI的策略就是，在C++的虚函数表的第一个slot处，放一个指针，指向保存该类的一些信息——即type_info类（在编译器文件中可以找到其定义）

dynamic_cast运算符可以在执行期决定真正的类型
对于指针来说：
如果转型成功，则会返回一个转换后的指针
如果是不安全的，会传回0
程序中通过 if 来判断是否成功，采取不同措施

对于引用来说：
如果引用真正转换到适当的子类，则可以继续
如果不能转换的话，会抛出 bad_cast 异常
通常使用 try 和 catch 来进行判断是否成功

Typeid运算符：

可以传入一个引用,typeid运算符会传回一个const reference,类型为type_info。其内部已经重载了 == 运算符，可以直接判断两个是否相等，回传一个bool值

例如 if( typeid( rt ) == typeid( fct ) )

RTTI虽然只适用于多态类，但是事实上type_info object也适用于内建类，以及非多态的使用者自定类型，只不过内建类型 得到的type_info类是静态取得，而不是执行期取得。

============================

============================

# 重写前置后置++操作

```C++
    Age&  operator++() //前置++
    {
        ++i;
        return *this;
    }

    const  Age operator++(int)  //后置++
    {
        Age tmp = *this;
         ++(*this);  //利用前置++
        return tmp;
    }
```

============================

https://zhuanlan.zhihu.com/p/359503982

# new与placement new

new operator就是new操作符，不能被重载，假如A是一个类，那么A * a=new A;实际上执行如下3个过程。
1. 调用operator new分配内存，operator new (sizeof(A))
2. 调用构造函数生成类对象，A::A()
3. 返回相应指针

事实上，分配内存这一操作就是由operator new(size_t)来完成的，如果类A重载了operator new，那么将调用A::operator new(size_t )，否则调用全局::operator new(size_t )，后者由C++默认提供。

operator new是函数，分为三种形式（前2种不调用构造函数，这点区别于new operator）

```C++
void* operator new (std::size_t size) throw (std::bad_alloc);

void* operator new (std::size_t size, const std::nothrow_t& nothrow_constant) throw();

void* operator new (std::size_t size, void* ptr) throw();
```

第一种分配size个字节的存储空间，并将对象类型进行内存对齐。如果成功，返回一个非空的指针指向首地址。失败抛出bad_alloc异常。

第二种在分配失败时不抛出异常，它返回一个NULL指针。

第三种是placement  new版本，它本质上是对operator new的重载，
定义于`#include <new>`中。它不分配内存，调用合适的构造函数在ptr所指的地方构造一个对象，之后返回实参指针ptr。

第一、第二个版本可以被用户重载，定义自己的版本，第三种placement new不可重载

```C++
A* a = new A; //调用第一种
A* a = new(std::nothrow) A; //调用第二种
new (p) A(); //调用第三种
```

new (p) A()调用placement new之后，还会在p上调用A::A()，这里的p可以是堆中动态分配的内存，也可以是栈中缓冲。

```C++
#include <iostream>
using namespace std;

class A
{
public:
    A()
    {
        cout << "A's constructor" << endl;
    }

    ~A()
    {
        cout << "A's destructor" << endl;
    }

    void show()
    {
        cout << "num:" << num << endl;
    }

private:
    int num;
};

int main()
{
    char mem[100];
    mem[0] = 'A';
    mem[1] = '\0';
    mem[2] = '\0';
    mem[3] = '\0';
    cout << (void*)mem << endl;
    A* p = new (mem)A;
    cout << p << endl;
    p->show();
    p->~A();
    getchar();
}
```

阅读以上程序，注意以下几点。

（1）用  placement  new操作，既可以在栈(stack)上生成对象，也可以在堆（heap）上生成对象。如本例就是在栈上生成一个对象。

（2）使用语句A* p=new (mem) A;定位生成对象时，指针p和数组名mem指向同一片存储区。所以，与其说定位放置new操作是申请空间，还不如说是利用已经请好的空间，真正的申请空间的工作是在此之前完成的。

（3）使用语句A *p=new (mem) A;定位生成对象时，会自动调用类A的构造函数，但是由于对象的空间不会自动释放（对象实际上是借用别人的空间），所以必须显示的调用类的析构函数，如本例中的p->~A()，但是内存并不会被释放，以便其他对象的构造。

（4）如果有这样一个场景，我们需要大量的申请一块类似的内存空间，然后又释放掉，比如在在一个server中对于客户端的请求，每个客户端的每一次上行数据我们都需要为此申请一块内存，当我们处理完请求给客户端下行回复时释放掉该内存

要知道每一次内存的申请，系统都要在内存中找到一块合适大小的连续的内存空间，这个过程是很慢的（相对而言)，极端情况下，如果当前系统中有大量的内存碎片，并且我们申请的空间很大，甚至有可能失败

所以placement new操作的作用就是：创建对象(调用该类的构造函数)但是不分配内存，而是在已有的内存块上面创建对象。用于需要反复创建并删除的对象上，可以降低分配释放内存的性能消耗。

下面是一个在堆上生成对象的例子。
B  和  A  一样，就是多了  GetNum SetNum

```C++
int main()
{
    char* mem = new char[10 * sizeof(B)];
    cout << (void*)mem << endl;
    B *p = new(mem)B;
    cout << p << endl;
    p->SetNum(10);
    cout << p->GetNum() << endl;
    p->~B();
    delete[]mem;
    getchar();
}
```

============================
# typename

https://blog.csdn.net/iotflh/article/details/114789270

typename  来源
从表面看，下面模板的参数  只支持  用户自定义类型，但  其实  对语言内置类型  或  指针调用  也支持
```C++
template <class T>
int compare(const T &v1, const T &v2)
{
    if (v1 < v2) return -1;
    if (v2 < v1) return 1;
    return 0;
}
//用指针调用:
int v1=1 ,v2=2;
int ret= cmpare(v1, v2)
```

class在类和模板中的表现的意义存在一些不一致。前者针对用户自定义类型，后者包含了语言内置类型和指针。

真正的原因
一些关键概念

## 限定名

限定了命名空间的名称
std::cout;    // std  就是一个  命名空间

## 依赖名和非依赖名

依赖名  是指  依赖于模板参数的名称，  而非依赖名则相反，指  不依赖于  模板参数的名称。

```C++
template<class T>
class Myclass
{
    int i;
    vector<int> vi;
    vector<int>::iterator vitr;  //非依赖名
    T t;
    vector<T> vt;
    vector<T>::iterator viter;  //依赖名
    //依赖于模板参数的名称就是依赖名
}
```

## 类作用域

在类外部访问类中的名字时，可以使用类用作域符，形如  class::name  的调用通常有3种：
静态数据成员，静态成员函数，嵌套类型

```C++
struct Myclass
{
    static int A;  //静态数据成员
    static int B(); //静态函数成员
    typedef int C; //嵌套类型
}
```

真实原因

```C++
template<class T>
void foo()
{
    T::iterator * iter;
}
```

T::iterator  这个定义  可以是  上述  3种类作用域  中的  任意一种类型

1. 如果  iterator  是嵌套类型  将正确执行，这段代码的意思是  定义一个  T::iterator  类型的  数据。

2. 当  iterator  是静态数据成员时，  以上代码将  被解释为  2个数  相乘，返回值抛弃。如果  iter  没有定义，将报错；但如果  iter  是  全局变量，将执行。

在模板实例化之前  完全没有办法区分，因此，有必要引入  新的关键字  typename  来区分这  2种情况。

typename  用法
C++  标准

对于用于模板定义  的依赖模板参数的名称，只有在  实例化的参数中  存在这个  类型名，或者  这个名称  前面  使用了  typename  关键字修饰，编译器  才会  将  这个  名字当做  是类型。  除了以上  2种情况，编译器不会  将它视为  类型。

即，当你想告知  编译器  iterator  是类型  而不是  变量，只需要  使用typename

```C++
template<class T>
void foo()
{
    typename T::iterator * iter;
}
```

此时，编译器可以  确信  T::iterator  是一个类型，而不是需要等到  实例化时  才能确定。

## 使用typename的规则

在以下情况  是==**禁用的**==：

模板定义之外，即typename  只能用于  模板中
非限定类型，比如前面介绍过的  int，  vector  之类
基类列表中，比如  template class C1: T::InnerType
构造函数的初始化列表中
如果类型是  依赖于  模板参数的  限定名，那么在它之前  必须加  typename (除非在类的初始化成员表中)
其他情况下，typename  是可选的，也就是说  对于一个  不是依赖名的限定名，该名称可选，比如  vector vi;

其他情况
对于不引起歧义的情况，仍然需要加  typename：

```C++
template<class T>
void foo(){
    typename T::iterator_type iterator_type;
    typedef typename _type_traits_<T>::iterator iterator_type;
}//将type_traits_<T>这个模板类中的iterator重命名为 iterator_type
```

--------------------

https://blog.csdn.net/lyn631579741/article/details/110730145

我们经常会这么用 typename，这是一项C++编程语言的泛型编程（或曰“模板编程”）的功能，typename关键字用于引入一个模板参数。

```C++
template <typename T>
const T& max(const T& x, const T& y)
{
  if (y < x) {
    return x;
  }
  return y;
}
```

在模板定义语法中关键字 class 与 typename 的作用完全一样

```C++
template<class T>
const T& max(const T& x, const T& y)
{
  if (y < x) {
    return x;
  }
  return y;
}
```

这里 class 关键字表明T是一个类型，后来为了避免 class 在这两个地方的使用可能给人带来混淆，所以引入了 typename 这个关键字，它的作用同 class 一样表明后面的符号为一个类型。

typename 的作用就是告诉 c++ 编译器，typename 后面的字符串为一个类型名称，而不是成员函数或者成员变量，这个时候如果前面没有 typename，编译器没有任何办法知道 T::LengthType 是一个类型还是一个成员名称(静态数据成员或者静态函数)，所以编译不能够通过。

什么情况下，class定义之后，编译不能通过呢？

```C++
template<typename T>
void fun(const T& proto){
        T::const_iterator it(proto.begin());
}
```

发生编译错误  是因为  编译器不知道  T::const_iterator  是个类型。直到确定了T是什么东西，编译器才会知道T::const_iterator是不是一个类型

C++标准：

对于用于模板定义的依赖于模板参数的名称，只有在实例化的参数中存在这个类型名，或者这个名称前使用了 typename 关键字来修饰，编译器才会将该名称当成是类型。除了以上这两种情况，绝不会被当成是类型

因此，如果你想直接告诉编译器 T::const_iterator 是类型而不是变量，只需用 typename修饰
typename  T::const_iterator it(proto.begin());

编译器就可以确定T::const_iterator是一个类型，而不再需要等到实例化时期才能确定

嵌套从属类型

事实上  类型  T::const_iterator  依赖于  模板参数  T， 模板  中  依赖于  模板参数  的  名称  称为从属名称（dependent  name）， 当一个从属名称嵌套在一个类里面时，称为嵌套从属名称（nested dependent name）。 其实T::const_iterator还是一个嵌套从属类型名称（nested dependent type name）。

嵌套从属名称是需要用typename声明的，其他的名称是不可以用typename声明的。比如下面是一个合法的声明
```C++
template<typename T>
void fun(const T& proto ,typename  T::const_iterator it);
```

============================

https://zhuanlan.zhihu.com/p/583428130

# ==thread_local==

thread_local  关键字用于  控制  name  的  storage duration  和  linkage。

从C++11开始，thread_local  可以和  static  和  extern  一起出现。

thread_local  只能用于  namespace scope  中的  对象声明，  block scope  中的对象声明  和  static  数据成员。

指明对象具有thread  存储周期
当  只有  thread_local  修饰  block  scope  变量时，  该变量  隐式  包含  static  语义。

thread_local  可以和  static  或  extern  一同使用，  分别表示  internal  或  external linkage (除了  static  数据成员，它总是  external linkage)

thread_local  存储周期
thread存储周期的对象  在线程开始时分配，  在线程结束时  释放。每个线程  具有  独自的对象。

non-local  变量

thread存储周期的  所有  non-local  变量  的初始化  是  thread launch  的一部分，  在  thread function  开始前。

static local  变量

在  block scope  中使用  static  或  thread_local  声明的  变量，  有  static  或  thread  存储周期，在  第一次调用时  初始化  (  除非它们的初始化是  零初始化  或  constant初始化，这种情况下，会在  block  第一次进入之前  初始化)

-------------------
https://www.cnblogs.com/zl1991/p/16454522.html

thread_local  是  C++11  新引入的  一种存储类型，它影响  变量的  存储周期。

C++  中有  4  种存储周期
automatic
static
dynamic
thread

只有  thread_local  修饰的  变量  具有  thread周期，  这些变量在  线程开始时  生成，线程结束时  销毁，  并且每个  线程  都拥有  一个独立的  变量实例。

thread_local  一般用于  需要保证  线程安全的  函数中。

需要注意：  如果  类的成员函数  内定义了  thread_local  变量，则  对于  同一个  线程  内的  该  类的  多个对象  都会  共享  同一个  实例变量，并且只会在  第一次执行  这个  成员函数时  初始化  这个  变量实例，  这一点  和  类的  静态成员变量  类似。

注意  static thread_local  和  thread_local  是等价的。

============================

https://blog.csdn.net/weixin_39640298/article/details/85056516

# const

## 修饰变量

修饰指针或ref
const  在  *  左侧：  指针指向的内容  为常量。
右侧：  指针本身为常量

ref本身就不能重新绑定(即，ref本身是常量)。  所以  const  只有  const int& b = a;  使得  指向的内容  变为常量。

## 修饰函数

**非成员函数  不允许  修饰符**。  所以对于  非成员函数，只有  修饰  参数  和  返回值：
int Add(int x, int y) const   //  编译器会报错，error C2270 "Add":  非成员函数不允许修饰符。
{ return x + y; }

const  修饰**函数参数**，则函数内部  不能改变参数，  函数外  不关心。
int func(const int x) { return ++x; }   //  编译报错

const  修饰**函数返回值**，则返回值不能进行修改。

## 修饰类对象
const  修饰类对象时  和  const修饰变量  没有  实质不同，只是在于  类对象的  "改变"  定义
类对象的  "改变"  定义：改变  任何成员变量的值，调用任何  非const  成员函数。
```C++
class myclass
{
public:
    void fun1();
    void fun2() const;
    int m_iVal;
};
const myclass temp;
temp.m_iVal = 10;     //  编译错误，不能修改成员变量值

temp.fun1();   //  编译错误，不能调用  非  const  成员函数
temp.fun2();   // ok
```

## 修饰类成员变量

被修饰的  成员变量  不能被  修改，所以  只能在  初始化列表  中被初始化。和  类中的  ref  变量  一样。
```C++
class myclass
{
public:
    myclass(int val) : m_iVal(val) { };
    const int m_iVal;
};
```

如果  是  静态  const  成员变量，  因为  属于  类，而不是  对象，  所以只能去  类外部  单独  定义  并初始化：
```C++
class myclass
{
    static const int m_iVal;      //  声明
};
const int myclass::m_iVal = 10;    //  定义/初始化
```

## 修饰类成员函数
表示  此函数  不能对  任何  成员变量  进行修改。一般  const  写在  函数的后面。  形如：  int func() const;

在成员函数调用的过程中，都有一个  this  指针被当做  参数  隐式  传给  成员函数。这个this指针，指向调用  这个函数的  对象，并且是  const  指针，不能修改其  指向  (被指向的内容  是可变的)。

传递给  const  成员函数的  this  指针，指向一个  const  对象。所以  在  const  成员函数内部，  this  指针  是  指向  const成员的  const  指针。

静态成员函数，没有this  指针，所以  静态成员函数不能  声明为  const。

2个成员函数，如果只是常量性不同，  可以被重载。  如果  2者  有着  实质等价的  实现，则令  非const  版  调用  const版。

### 修饰类成员函数与非成员函数 构成重构
重载的定义：  在同一个作用域中，同名函数的  形式参数  (参数个数，类型，或顺序)  不同时，  构成函数  重载。

乍看，成员函数  和  const成员函数  不应该  构成  重载，但下面的  类  是  合法的
```C++
class D
{
public:
    void funcA();
    void funcA() const;
    void funcB(int a);
    void funcB(const int a);
}
```
其中  funcA  的  2个函数  构成了  函数重载，而  funcB  则编译错误。

const  发生重载的  本质是：  由于隐含的  this  形参的存在，const版本的  成员函数  使得  作为形参的  this  指针  的类型  变为了  指向const对象的  指针，而  非  const版本的  成员函数  的  this  指针  是正常版本的  指针。

。。但是，int a,  和  const int a，  也是一个  正常  一个  const  啊。。  应该是  this  和  形参  还是有  区别的。   因为这个  const  是加载  函数后面的，肯定不一样。  所以不能当做  形参的  const  看。

。。不，下面提到了。  对于  非ref传参，  形参是否const  是等价的。

### const函数  和  非const函数的  调用规则

const对象  默认调用  const成员函数，非  const对象  默认调用  非const成员函数。

在同时存在  const函数  和  非const重载函数的  情况下，  如果  非  const  对象想调用  const  成员函数，则需要  显示的转化，例如  (const Student&) obj.getAge();

如果  const对象  想要调用  非const成员函数，也需要强转：  const_cast<Student&>(constObj).getAge();

当类中只有一种函数存在时
非  const  对象可以调用  const  或  非const  成员函数
const  对象  只能调用  const  成员函数，   调用非const  会  编译报错。

而  funcB  的  编译错误的原因在于：  对于非ref传参，形参是否  const  是等价的。


## mutable

C++  为了突破  const  的限制，提供了  关键字  mutable。  被  mutable  修饰的  变量，  将永远处于  可变的状态，  即使在  一个const  函数  或  const类对象  中  ：

下列代码是正确的。
```C++
class myclass
{
public:
    int func() const { return ++m_iVal; };

    mutable int m_iVal;
    mutable int m_iVal2;
};

const myclass temp;
temp.m_iVal2++;
```

mutable  只能修饰  非  静态数据成员。

============================
# mutable

lambda语法
[capture-list] (parameters) mutable -> return-type { statement }

lambda默认是常量函数。mutable可以取消  常量性

-----------------

https://blog.csdn.net/weixin_39640298/article/details/119391997

mutable

常规使用
const修饰的  成员函数，不能对  成员变量  进行修改，因为  隐式的this  是  const属性的。
实际开发中，我们可能需要对某些成员变量进行修改，就需要用到  mutable
一个mutable  的成员，永远不会是  const的，  即使它是  const对象的  成员。

用于lambda表达式

lambda  捕获列表  [=]  采用  传值方式  捕捉  外部变量时，  编译器将  表达式  翻译为  一个  未命名类  的  未命名对象。该类  含有一个  经过  const  修饰  的  operator()运算符。

```C++
int func()
{
    int a = 10;
    int b = 20;
    auto addfun = [=] (const int c) -> int { return a + c; };
    int c = addfun(b);
    cout<<c<<endl;
}
```

等价于：
```C++
class myclass
{
public:
    myclass(int a) : m_a(a) { };       //  形参  对应  捕获的变量

    //  该  operator()  的  返回类型，形参，函数体  都和  lambda  一致
    int operator()(const int c) const
    { return a + c; }
private:
    int m_a;        //  对应  通过  值捕获的  变量。
};
```

加入mutable  后，相当于把  operator()  的  const  修饰去掉，这样就可以在  函数内部  对  传值方式  捕获的  变量  进行修改。

和  const_cast  相比
区别：
1. const_type  中的  type  必须是  指针  或  ref
2. mutable  一般  只用于  成员变量  和  lambda  表达式

============================

# **字面量操作符**

operator""myDefinedSuffix

operator""s

operator""_C

============================

# variant C++17

```C++
#include <variant>
template <size_t n, typename... Args>
std::variant<Args...> _tuple_index(size_t i, const std::tuple<Args...>& tpl) {
    if (i == n)
        return std::get<n>(tpl);
    else if (n == sizeof...(Args) - 1)
        throw std::out_of_range("越界.");
    else
        return _tuple_index<(n < sizeof...(Args)-1 ? n+1 : 0)>(i, tpl);
}
template <typename... Args>
std::variant<Args...> tuple_index(size_t i, const std::tuple<Args...>& tpl) {
    return _tuple_index<0>(i, tpl);
}
```

```C++
#include <variant>
#include <string>
#include<cassert>
#include<iostream>
int main() {
    // 定义两个variant类型实例
    std::variant<int, float> v, w;
    v = 18;
    // index 可以判断实际对应的类型,，默认第一种类型。
    std::cout << "Corresponding type is " << w.index() << std::endl;
    int i = std::get<int>(v);
    std::cout << "value is " << i << std::endl;
    w = std::get<int>(v);
    w = std::get<0>(v); // 与上一行功能相同
    w = v; // 与上一行功能相同

    // 类型对不上会抛出异常
    try {
        std::get<float>(w);
    }
    catch (const std::bad_variant_access& ex) {
        std::cout << ex.what() << std::endl;
    }

    // 避免使用 try catch，可以使用get_if
    float* fres = std::get_if<float>(&v);
    if (fres) {
        std::cout << "float value is " << *fres << std::endl;
    }
    else {
        std::cout << "does not contain a value" << std::endl;
    }

    v = 1.23f;
    fres = std::get_if<float>(&v);
    if (fres) {
        std::cout << "float value is " << *fres << std::endl;
    }
    else {
        std::cout << "does not contain a value" << std::endl;
    }

    // index 可以判断实际对应的类型
    std::cout << "Corresponding type is " <<  w.index() << std::endl;

    // holds_alternative 判断当前是否使用此类型
    if (std::holds_alternative<float>(v)) {
        std::cout << "now using float type" << std::endl;
    }
    else {
        std::cout << "not using float type" << std::endl;
    }
    return 0;
}
```

Corresponding type is 0
value is 18
bad variant access
does not contain a value
float value is 1.23
Corresponding type is 0
now using float type


与union一样，如果某一variant保存类型T的一个值，那么T的对象被直接分配在variant的内部。
variant不能在动态内存分配方式中使用。
variant不可存放引用，数组或是void值。空varaint是错误的用法（应该用`std::variant<std::monostate>`取代）。

与union一样，vairant默认初始化为它模板类型列表中第一个值，除非该项无法默认构造（若默认构造函数无法编译，辅助类std::momostate可用来使variant成为默认可构造的）

`std::variant_alternative<2, decltype(var1)>::type val{ L"123" };`

============================

# C++20 span

https://blog.csdn.net/fpcc/article/details/122979853

C++  中  内存的管理和限制访问，  是一个问题。
字符串有  string_view。

对于  普通的连续内存。  C++20  有了  std::span

这个模板类的基本作用  就是  对连续内存的管理。
可以将  std::span  看做一种  索引，它能够  保证  传递的  连续内存的长度的  正确性。

想象下：  如果一个数组  退化成  指针后，传递给  函数，  长度的控制  一定是一个首要的问题。

C++中  span的定义

```C++
namespace std {
  template<class ElementType, size_t Extent = dynamic_extent>
  class span {
  public:
    // 常量与类型
     using element_type = ElementType;
    using value_type = remove_cv_t<ElementType>;
    using size_type = size_t;
    using difference_type = ptrdiff_t;
    using pointer = element_type*;
    using const_pointer = const element_type*;
    using reference = element_type&;
    using const_reference = const element_type&;
    using iterator = /* 由实现定义 */;
    using reverse_iterator = std::reverse_iterator<iterator>;
    static constexpr size_type extent = Extent;

    // 构造函数、复制与赋值
    constexpr span() noexcept;
    template<class It>

      constexpr explicit(extent != dynamic_extent) span(It first, size_type count);

    template<class It, class End>
      constexpr explicit(extent != dynamic_extent) span(It first, End last);
    template<size_t N>
      constexpr span(type_identity_t<element_type> (&arr)[N]) noexcept;
    template<class T, size_t N>
      constexpr span(array<T, N>& arr) noexcept;
    template<class T, size_t N>
      constexpr span(const array<T, N>& arr) noexcept;
    template<class R>
      constexpr explicit(extent != dynamic_extent) span(R&& r);
    constexpr span(const span& other) noexcept = default;
    template<class OtherElementType, size_t OtherExtent>
      constexpr explicit(/* 见描述 */)
        span(const span<OtherElementType, OtherExtent>& s) noexcept;

    ~span() noexcept = default;

    constexpr span& operator=(const span& other) noexcept = default;

    // 子视图
    template<size_t Count>
      constexpr span<element_type, Count> first() const;
    template<size_t Count>
      constexpr span<element_type, Count> last() const;
    template<size_t Offset, size_t Count = dynamic_extent>
      constexpr span<element_type, /* see description */> subspan() const;

    constexpr span<element_type, dynamic_extent> first(size_type count) const;
    constexpr span<element_type, dynamic_extent> last(size_type count) const;
    constexpr span<element_type, dynamic_extent> subspan(
      size_type offset, size_type count = dynamic_extent) const;

    // 观察器
    constexpr size_type size() const noexcept;
    constexpr size_type size_bytes() const noexcept;
    [[nodiscard]] constexpr bool empty() const noexcept;

    // 元素访问
    constexpr reference operator[](size_type idx) const;
    constexpr reference front() const;
    constexpr reference back() const;
    constexpr pointer data() const noexcept;

    // 迭代器支持
    constexpr iterator begin() const noexcept;
    constexpr iterator end() const noexcept;
    constexpr reverse_iterator rbegin() const noexcept;
    constexpr reverse_iterator rend() const noexcept;

  private:
    pointer data_;              // 仅用于阐释
    size_type size_;            // 仅用于阐释
  };

  template<class It, class EndOrSize>
    span(It, EndOrSize) -> span<remove_reference_t<iter_reference_t<It>>>;
  template<class T, size_t N>
    span(T (&)[N]) -> span<T, N>;
  template<class T, size_t N>
    span(array<T, N>&) -> span<T, N>;
  template<class T, size_t N>
    span(const array<T, N>&) -> span<const T, N>;
  template<class R>
    span(R&&) -> span<remove_reference_t<ranges::range_reference_t<R>>>;
}
```

```C++
#include <span>
#include <iostream>

int main()
{
    constexpr char str[] = "ABCDEF\n";

    const std::span sp{str};

    for (auto n{sp.size()}; n != 2; --n) {
        std::cout << sp.last(n).data();
    }
}
```

-------------------
https://zhuanlan.zhihu.com/p/530405436

https://en.cppreference.com/w/cpp/container/span

std::span  是指向  一组  连续的对象的  对象，  是一个视图  view，  不是一个拥有者  owner。

std::span  可以有  
## 2种范围
静态范围：  static extend  编译期就可以确定大小。
动态范围：  dynamic extend  由指向第一个对象的  指针  和连续对象的大小组成。

```C++
 #include <ranges>
 #include <vector>
 #include <iostream>
 #include <span>
 #include <format>
 ​
 void printSpan(std::span<int> container)
 {
     std::cout << std::format("container size: {} \n", container.size());
     for (auto ele : container)
     {
         std::cout << ele << " ";
     }
     std::cout << std::endl;
     std::cout << std::endl;
 }
 ​
 ​
 int main()
 {
     std::vector v1{1, 2, 3, 4, 5, 8};
     std::vector v2{9, 2, 4, 2, 6, 78};
 ​
     std::span<int> dynamicSpan(v1);
     std::span<int, 6> staticSpan(v2);
 ​
     printSpan(dynamicSpan);
     printSpan(staticSpan);
 }
```

## 构造
默认构造 `template< class T, std::size_t Extent = std::dynamic_extent > class span;`
使用迭代器(std::contiguous_iterator)
迭代器 + 大小
C-stye数组
```C++
 int main()
 {
     std::vector v1{1, 2, 3, 4, 5, 8};
     std::vector v2{9, 2, 4, 2, 6, 78};
 ​
     std::span<int> dynamicSpan(v1); // 默认构造
     std::span<int, 6> staticSpan(v2);
 ​
     std::span<int, 2> undefineSpan(v2); // 未定义行为 2 != 6
 ​
     std::span<int> iteratorSpan1(v1.begin(), v1.end());
     std::span<int> iteratorSpan2(v1.begin()+1, v1.end());
 ​
     std::span<int> iteratorsizeSpan3(v1.begin(), 6); // iterator + size
 ​
     int cArray[5]{ 1, 2, 3, 4, 5 };
     std::array<int, 5> cppArray{ 1, 2, 3, 4, 5 };
 ​
     std::span<int> cArrSpan(cArray);
     std::span<int> cppArrSpan{cppArrSpan};
     std::span<int,5> cppArrSpan1{cppArrSpan};
 }
```

std::span 在构造时不支持隐式类型转换
std::span  可以推断C-stye数组的大小

## 修改

```C++
 void printSpan(std::span<int> container)
 {
     std::cout << std::format("container size: {} \n", container.size());
     for (auto ele : container)
     {
         std::cout << ele << " ";
     }
     std::cout << std::endl;
     std::cout << std::endl;
 }
 ​
 int main()
 {
     std::cout << '\n';
     std::vector vec{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
     printSpan(vec);
     std::span span1(vec);
     std::span span2{span1.subspan(1, span1.size() - 2)};
     std::transform(span2.begin(), span2.end(),
                    span2.begin(),
                    [](int i) { return i * i; });
     printSpan(vec);
     printSpan(span1);
 }
```

============================

# type_traits

https://blog.csdn.net/wxj1992/article/details/122506368

type_traits是什么

type traits技术是用来获得类型的，而c++ 11引入的type_traits，就是围绕这一点来的，核心就是定义了一系列的类模板，使得程序员可以用来在编译期判断类型的属性、对给定类型进行一些操作获得另一种特定类型、判断类型和类型之间的关系等，这在模板编程里是很有用的，而它们的实现用到的核心技术就是编译期的常量和模板的最优匹配机制。

`<type_traits>`里面的内容可以分为

## 三大类：

（1）辅助基类: std::integral_constant以及两个常用特化true_type和false_type，用于创建编译器常量，也是类型萃取类模板的基类。

（2）类型萃取类模板: 用于以编译期常量的形式获得类型特征，比如某个类型是不是浮点数，某个类型是不是另一个类型的基类等等，大部分都是是与否的判断，但也有获取数组的秩这种比较特殊的。

（3）类型转换类模板: 用于通过执行特定操作从已有类型获得新类型，比如为一个类型添加const、去除volatile等。

type_traits通常用来做什么
利用type_traits里的各种模板类，最重要的就是我们可以在编译期获得特定的类型type，或者是特定的常量值value，用它们我们可以实现很多事情

（1）使用type进行变量的声明，比如std::conditional得到的类型：
//简单展示功能，实际使用种int和double通常都是模板参数相关的类型

```C++
typedef std::conditional<sizeof(int) >= sizeof(double), int, double>::type Type;

Type a;
a = 3;
```

（2）使用type进行模板匹配的选择，最典型的是enable_if来实现SFINAE，后面会详细聊下这个，以下是一个简单的例子：

```C++
// 1. the return type (bool) is only valid if T is an integral type:
template <class T>
typename std::enable_if<std::is_integral<T>::value,bool>::type
  is_odd (T i) {return bool(i%2);}

// 2. the second template argument is only valid if T is an integral type:
template < class T,
           class = typename std::enable_if<std::is_integral<T>::value>::type>
bool is_even (T i) {return !bool(i%2);}
```

这里的enable if确保只有整型才能够使用这两个函数模板，其他类型的`std::enable_if<std::is_integral::value>::type`是不存在的，所以对于这俩模板的实现会是Substitution Failure，但如果还有其他的可以Substitution success的模板也是没有问题的，如果没有就会报编译错误，这就是所谓的SFINAE：Substitution Failure Is Not An Error，也就是存在Substitution Failure不认为是错误，只要还有其他能够成功的。

（3）利用is_xxx所生成的bool类型的编译器常量值来做编译期的条件分支判断，如下：

```C++
void algorithm_signed  (int i)      { /*...*/ }
void algorithm_unsigned(unsigned u) { /*...*/ }

template <typename T>
void algorithm(T t)
{
    if constexpr(std::is_signed<T>::value)
        algorithm_signed(t);
    else
    if constexpr (std::is_unsigned<T>::value)
        algorithm_unsigned(t);
    else
        static_assert(std::is_signed<T>::value || std::is_unsigned<T>::value, "Must be signed or unsigned!");
}
```

这里的if constexpr是c++ 17后引入的新语法，可以在编译期做分支判断。

辅助基类
辅助类指的是std::integral_constant和以它为基础的两个实例化std::true_type 和std::false_type，
典型的定义如下：

```C++
template<class T, T v>
struct integral_constant {
    static constexpr T value = v;
    using value_type = T;
    using type = integral_constant; // using injected-class-name
    constexpr operator value_type() const noexcept { return value; }

    constexpr value_type operator()() const noexcept { return value; } // since c++14

};

using true_type = integral_constant<bool,true>;
using false_type = integral_constant<bool,false>;
```

。。就是做了一个封装啊，封装了  类  和  值。

std::integral_constant是一个类模板，用于为特定类型封装一个静态常量和对应类型，是整个c++ type traits的基础。

首先我们看模板参数，有两个参数，第一个是类型T，第二个是对应类型的值v，value是一个编译期就能确定的常量，值根据v确定，value_type就是类型T，而using type = integral_constant则是表示type指代实例化后的integral_constant类型。有两个成员函数，均用于返回wrapped value，都是constexpr，可以获得编译期的值：

（1）类型转换函数方式：constexpr operator value_type() const noexcept;
（2）仿函数方式：constexpr value_type operator()() const noexcept; (since C++14)

举一个简单的例子，如果有这样的定义：
```C++
    enum class my_e { e1, e2 };

    typedef std::integral_constant<my_e, my_e::e1> my_e_e1;
    typedef std::integral_constant<my_e, my_e::e2> my_e_e2;
```

如果我们要获取my_e_e1里的value，有以下几种方式
（1）my_e_e1::value，直接获取。

（2）my_e_e1() ，调用constexpr value_type operator()() const noexcept { return value; } // since c++14

（3）my_e_e1 v; 然后直接通过v 调用，实际上调用的是constexpr operator value_type() const noexcept { return value; }

类型相关判断信息获取
判断基础类型类别
此类模板类用于判断给定的类型是不是某种基础类型，一共有以下几种

https://zh.cppreference.com/w/cpp/meta

。感觉50,60个。反正各种  is_，  只要你想到的，基本都有。
。类型是否：void,空指针，整数，数组，枚举，联合，函数，指针，右值，左值，非静态成员对象的指针
。是否：基础类型，算术类型，标量，对象，左值ref

。是否：const，volatile，平凡，平凡复制，标准布局，POD，字面类型，强结构，抽象类，多态类，final类，无符号算术类型，有已知边界的数组类型，作用域枚举类型

。类型是否：  带有针对指定实参的构造器，有默认构造器，有复制构造器，从右值ref构造，虚析构器
。类型是否：相同，派生，转换，布局兼容
。等

以is_array为例，可能的一种实现如下：

```C++
template<class T>
struct is_array : std::false_type {};

template<class T>
struct is_array<T[]> : std::true_type {};

template<class T, std::size_t N>
struct is_array<T[N]> : std::true_type {};
```

定义了三个模板，名字都是is_array，第一个是最通用的，继承自false_type，使用这个模板产出的类里面有个bool类型的编译期常量value，值为false。第二个和第三个都继承自true_type，并且分别限定只能匹配T[]和T[N]的形式，根据模板匹配的规则，这两种形式会优先匹配到这两个模板上而不是第一个通用的模板上，从而得到值为true的编译器常量value。而其他形式的类型均会被第一个模板匹配得到false

```C++
template< class T >
struct is_arithmetic : std::integral_constant<bool,
        std::is_integral<T>::value ||  std::is_floating_point<T>::value> {};
```

这里用到了两外两个type traits，is_integral和is_floating_point，如果二者或为真，is_arithmetic就继承自value为true的integral_constant，反之value为false，从而得到了一个表明给定类型是否是数字的编译期常量。

```C++
template<class T> struct is_const          : std::false_type {};
template<class T> struct is_const<const T> : std::true_type {};
```

这也是典型的利用模板选择的规则，两个模板，如果用const xxx去调用，会match到第二个，否则会退而求其次match到第一个，从而得到true or false。

```C++
template<class T>
struct is_move_constructible :
      std::is_constructible<T, typename std::add_rvalue_reference<T>::type> {};
```

利用了另外两种traits，is_constructible是判断类型T是否能以特定类型的参数进行构造，这里这个类型是std::add_rvalue_reference<T>::type，也就是右值，add_rvalue_reference这个会使用引用折叠规则，因此用于调用is_move_constructible的T本身可以是左值也可以是右值。

```C++
template<class T>
struct rank : public std::integral_constant<std::size_t, 0> {};

template<class T>

struct rank<T[]> : public std::integral_constant<std::size_t, rank<T>::value + 1> {};

template<class T, std::size_t N>

struct rank<T[N]> : public std::integral_constant<std::size_t, rank<T>::value + 1> {};
```

这几个模板定义很有意思，对于T[]和T[N]这种形式的类型会优先匹配到下面两个模板，进而递归地调用自身并且每次加一，从而最终得到总的维度，例子：

```C++
cout<<std::rank<int>{};        // 0
rank<int[5]>{}       // 1
rank<int[5][5]>{}    // 2
rank<int[][5]5[]>{}    // 3

template<class T, class U>
struct is_same : std::false_type {};

template<class T>
struct is_same<T, T> : std::true_type {};

template< class T > struct remove_cv                   { typedef T type; };
template< class T > struct remove_cv<const T>          { typedef T type; };
template< class T > struct remove_cv<volatile T>       { typedef T type; };
template< class T > struct remove_cv<const volatile T> { typedef T type; };

template<class T> struct add_cv { typedef const volatile T type; };

template<bool B, class T = void>
struct enable_if {};

template<class T>
struct enable_if<true, T> { typedef T type; };
```

============================


# concept

```C++
#include <concepts>
#include <type_traits>
```

https://blog.csdn.net/guxch/article/details/110795047

C++20  模板类型约束(concept)

模板编程中，类型是一种输入参数，对某一具体模板类，多数情况下，并不是所有的类型都适合，类型必须符合某种要求，模板类型约束就是在编译期，用于限制模板参数。其实在C++11的type_traits中，就有很多类似于is_const、is_base_of、enable_if、conditional这样的类型判断，类型选择，类型操作等函数，它们在编译期施加到类型参数上，起到了类型限制等作用。

C++  20中为此引入两个关键字requires和concept，将类型限制上升到语言层面

## requires
对模板参数的类型约束，可直接写一个require表达式，它可以出现在两个位置（效果相同），如下例：
```C++
// int8_t doesnot satisfy the requirements
template<typename T>
int f(T)   requires (sizeof(T) > 1)  { return sizeof(T); }
//or
template<typename T> requires (sizeof(T) > 1)
int f(T)  { return sizeof(T); }

template<std::integral T>

int f(T)   requires (sizeof(T)>4)     // int64_t satisfies the requirements while int32_t doesnot
{
    return sizeof(T);
}
```

。。感觉  template  后面好，毕竟是  type  约束。  不是  method约束。
。而且清晰，  函数后面  还有  形参列表，  形参列表  后面是  requires，  有点多了。

requires后面可以是一个语句 (clause)或一个表达式(expression)。

clauses是一个可在编译期计算出的布尔表达式，上面给出的几个都是这样的例子，当然，也可以是若干逻辑判断的组合，如下例：

```C++
template<class T> requires std::is_pointer<T>::value && std::is_integral<T>::value

void f(T) {};
```

。。不需要  括号
。。is_pointer, is_intergral  就没有见过。
。。看起来是  C++11  的  type_traits  的东西。

requires expressions稍微复杂些，它的形式如下，其中参数是可选的，内容中的requirement-seq可以是一组requirement-seq，也可以再包含requirement-seq而形成嵌套的表达式。

`requires ( parameter-list(optional) ) { requirement-seq }`

目前的编译器仅支持将requirements expression定义为concept形式(关于concept，见下节说明)，有4种形式的requirements expression：

。这篇文章是2020-12发表的

## simple requirements
：只要语法正确即可，不会计算结果。

```C++
template<class T>
concept C= requires(T a){
    std::is_pointer<T>::value;      //actually not required T is a pointer
     a++;
};
```

。。这个还没有  应用到  函数上吧。  只是  声明定义了  一组  requires  ，赋给  concept。
。。那  a++是什么。。
。在  模板类型约束的  时候  对  类型的值a  做了操作。   指针后移。

## type requirements
：以typename开始，主要用于检测类型是否存在或是否有嵌套的子类型

```C++
template<class T>
class TypeA;
template<typename T> concept C =
requires {
    typename T::innertype; // innertype is required to exist within T
    typename TypeA<T>;     // the type name TypeA<T> is required to exist
};
```

。。第一个需要参数，所以  (T a)，  这里不需要，  就是  最开始说到的  参数可选。

## compound requirements
:它具有{ expression } noexcept(optional) ->return-type-requirement(optional)这样的形式，其中expression需要语法正确，如果对expression的返回值有要求，则它需要满足return-type-requirement。noexcept和return-type-requirement都是可选的。

```C++
template<typename T> concept C =
requires(T x) {
{*x} ;
// *x有意义

{x + 1} -> std::same_as<int>;
// x + 1有意义且std::same_as<decltype((x + 1)), int>，即x+1是int类型

{x * 1} -> std::convertible_to<T>;
// x * 1 有意义且std::convertible_to< decltype((x *1),T>
};
```

。*  是  解指针啊，  所以  x  是指针。   话说有  operator*，重载  解指针吗，  记得不行？

## nested requirements
：由若干条requires构成，每一条都需要满足。

```C++
template <class T>
concept Semiregular = std::destructible<T> && requires(T a, size_t n) {
    a();                //simple

    requires std::same_as<T*, decltype(&a)>;
// nested: "Same<...> evaluates to true"

    { a.~T() } noexcept;
// compound: "a.~T()" is a valid expression that doesn't throw

    { delete new T[n] };
// compound
};
```

## concept

concept的作用是将相关类型约束（constraints）“组合”起来，用一个名称表达，类似于命名数据类型：先定义，然后可多次使用，而requires则像匿名类型，仅用于一次性定义与使用。使用concept时，将以前的class/typename改为此concept名，表示对此类型进行约束。标准库增加了`<concepts>`，其中定义了若干concept，我们直接就可以应用，例如上面“requires”一节中的std::integral就是这样一个concept。

我们也可以自己定义concept：
```C++
template<typename T>
concept IntorFloat=std::integral<T> || std::floating_point<T>;

template <class T, class U>
concept Derived = std::is_base_of<U, T>::value;
template<Derived<Base> T>
void f(T);  // T is constrained by Derived<T, Base>

template< typename T>

concept IntPointer=std::is_pointer<T>::value && std::is_integral<std::remove_pointer_t<T>>::value ;
```

。。代替的是  class. typename  这2个关键字，  不是  T。

注意上面第二个例子，如果在定义concept时有多个参数，此时的concept可认为是一个concept模板（可以有缺省值），第一个参数是使用时对其限制的参数，其余在使用时以concept模板方式给出。

除了用于模板，concept也可以直接用于变量的类型限制

```C++
void IntSwap(std::integral auto &a, std::integral auto &b)
{
    std::integral auto c(0);
    c=a;
    a=b;
    b=c;
}
```

这样的写法还是改为模板函数好些，灵活也意味着各种“奇技”的出现。

-------------------

https://zhuanlan.zhihu.com/p/411883506

# concept

## 动机
C++的 Template 为编程带来的极大的方便，但是也有一些问题

```C++
template <typename T>
auto add(T first, T second)
{
    return first + second;
}

add(true, false);
```

编译器看到这条语句时，可以  实例化模板：

```C++
#ifdef INSIGHTS_USE_TEMPLATE
template<>
int add<bool>(bool first, bool second)
{
    return static_cast<int>(first) + static_cast<int>(second);
}
#endif
```

但是这种将  bool  升为  int  的方法，可能和我们的期待并不一致。

优点
将模板参数的要求（Requirements）变为接口的一部分
函数的重载和类模板的实例化可以基于 concept。
concept 可用于函数模板、类模板、类或类模板的泛型成员函数。
编译器会将模板参数的要求与给定的模板参数进行比较，从而得到更直观的错误消息。
您可以使用预定义的 concept，也可以定义自己的 concept。
auto 和 concept 的使用是统一的。你可以用一个 concept 来代替auto。
如果函数声明使用 concept，它将自动成为函数模板。因此，编写函数模板与编写函数一样简单

使用
## concept四种使用方式：
requires 语句
后缀 requires 语句
受约束的模板参数
简化的函数模板

例子：
```C++
#include <concepts>
#include <type_traits>
#include <iostream>

template <typename T>
requires std::integral<T>
T gcd(T a, T b)
{
    if (b == 0)
        return a;
    else
        return gcd(b, a % b);
}

template <typename T>
T gcd1(T a, T b) requires std::integral<T>
{
    if (b == 0)
        return a;
    else
        return gcd1(b, a % b);
}

template <std::integral T>
T gcd2(T a, T b)
{
    if (b == 0)
        return a;
    else
        return gcd2(b, a % b);
}

std::integral auto gcd3(std::integral auto a, std::integral auto b)
{
    if (b == 0)
        return a;
    else
        return gcd3(b, a % b);
}

int main()
{
    std::cout << "gcd(100, 10)= " << gcd(100, 10) << '\n';
    std::cout << "gcd1(100, 10)= " << gcd1(100, 10) << '\n';
    std::cout << "gcd2(100, 10)= " << gcd2(100, 10) << '\n';
    std::cout << "gcd3(100, 10)= " << gcd3(100, 10) << '\n';
}
```

头文件 `<concepts>`，这个头文件可以让我们使用 std::integral 等 concept。
。。那  type_traits  用来干什么？

requires 语句的更多用法

关键字 requires 可以指定对模板参数（gcd）或函数声明（gcd1）的约束。requires 后面必须跟编译时谓词，如命名 concept、命名 concept 的合取/析取或requires 表达式（Requires Expressions）或者编译期布尔表达式：

```C++
#include <iostream>

template <unsigned int i>
requires (i <= 20)
int sum(int j)
{
    return i + j;
}

int main()
{
    std::cout << "sum<20>(2000): " << sum<20>(2000) << '\n';
}
```

事实上，我们并不是很推荐这样的做法。一个更好地做法是使用命名 concept 或它们的组合，通过对 concept 进行命名可以实现它们的重用，具体方法我们将在后面进行介绍。

。。至今没有搞明白：  如果一个对象的  类型是运行时确定的，那么  template  会怎么处理，编译期无法确定类型。

。。但是，C++  是否存在  编译期无法确定类型的  对象呢？  好像没有。无论如何，你调用template的时候，是  要给一个  参数的，  那么这个参数  肯定有  类型的，  auto？  推导不出来的话，编译应该会失败吧。

。"使用 auto 类型推导的变量必须马上初始化"。

什么是 concept
concept 归根结底就是编译时谓词。而编译时谓词就是在编译时执行并返回布尔值的函数。

```C++
struct Test{};

int main()
{
    std::cout << '\n';
    std::cout << std::boolalpha;
    std::cout << "std::three_way_comparable<int>: "
              << std::three_way_comparable<int> << "\n";
    std::cout << "std::three_way_comparable<double>: ";

    if (std::three_way_comparable<double>)
        std::cout << "True";
    else
        std::cout << "False";
    std::cout << "\n\n";

    static_assert(std::three_way_comparable<std::string>);

    std::cout << "std::three_way_comparable<Test>: ";
    if constexpr (std::three_way_comparable<Test>)
        std::cout << "True";
    else
        std::cout << "False";
    std::cout << '\n';

    std::cout << "std::three_way_comparable<std::vector<int>>: ";
    if constexpr (std::three_way_comparable<std::vector<int>>)
        std::cout << "True";
    else
        std::cout << "False";
    std::cout << '\n';
}
```

std::three_way_comparable可以在编译时检查 T 是否支持六个比较运算符，程序运行结果如下：

```C++
std::three_way_comparable<int>: true
std::three_way_comparable<double>: True

std::three_way_comparable<Test>: False
std::three_way_comparable<std::vector<int>>: True
```


任意数量模板参数
concept同样支持任意数量模板参数，使用起来同样非常简单：

```C++
template<std::integral... Args>
bool all(Args... args) { return (... && args); }

template<std::integral... Args>
bool any(Args... args) { return (... || args); }

template<std::integral... Args>
bool none(Args... args) { return not(... || args); }

int main()
{
    std::cout << std::boolalpha << '\n';
    std::cout << "all(5, true, false): " << all(5, true, false) << '\n';
    std::cout << "any(5, true, false): " << any(5, true, false) << '\n';
    std::cout << "none(5, true, false): " << none(5, true, false) << '\n';
}
```

。。。？？？

。。我之前见过  任意数量的模板参数，但是记得是：  需要2个函数，  一个  不定长参数，一个  定长参数。   不定长参数中，每次  从参数列表中  获得一个  参数，  然后  递归  调用，  调用的时候  会根据参数的  个数，来决定  调用  定长参数  还是  不定长参数  (不太清楚这个是  需要  编码实现，还是  会自动  重载，  应该是  编码实现的。)。  。  定长参数  是  最后一步。

。
。这里  没有递归，  不清楚  (...)   是不是  递归？   如果是递归的话，  最后一步是什么？

重载示例
std::advance 是标准模板库的一种算法,它将给定的迭代器 iter 递增 n 个元素。
根据给定迭代器的功能，可以使用不同的高级策略。
例如，std::forward_list 支持只能单向前进的迭代器，
而 std::list 支持双向迭代器，
std::vector 支持随机访问迭代器。

因此，对于 std::forward_list 或 std ::list 提供的迭代器，对 std::advance(iter, n) 的调用必须递增n次。 而对于由 std::vector 提供的 std::random_access_iterator，这种时间复杂性不成立。

对于这几种明显不同的迭代器，C++也提供了对应的 concept

```C++
template <std::forward_iterator I>
void advance(I &iter, int n)
{
    std::cout << "forward_iterator" << '\n';
}

template <std::bidirectional_iterator I>
void advance(I &iter, int n)
{
    std::cout << "bidirectional_iterator" << '\n';
}

template <std::random_access_iterator I>
void advance(I &iter, int n)
{
    std::cout << "random_access_iterator" << '\n';
}

int main()
{
    std::cout << '\n';

    std::forward_list forwList{1, 2, 3};
    std::forward_list<int>::iterator itFor = forwList.begin();
    advance(itFor, 2);

    std::list li{1, 2, 3};
    std::list<int>::iterator itBi = li.begin();
    advance(itBi, 2);

    std::vector vec{1, 2, 3};
    std::vector<int>::iterator itRa = vec.begin();
    advance(itRa, 2);

    std::cout << '\n';
}
```

output：
forward_iterator
bidirectional_iterator
random_access_iterator

显式模板实例化
concept同样支持显式的模板实例化：

```C++
template <typename T>
struct Vector
{
    Vector()
    {
        std::cout << "Vector<T>" << '\n';
    }
};

template <std::regular Reg>
struct Vector<Reg>
{
    Vector()
    {
        std::cout << "Vector<std::regular>" << '\n';
    }
};
```

## 占位符
concept 可以用作返回类型、基于范围的for循环、变量类型。

```C++
std::integral auto getIntegral(int val)
{
    return val;
}

int main()
{
    std::cout << std::boolalpha << '\n';

    std::vector<int> vec{1, 2, 3, 4, 5};
    for (std::integral auto i : vec)
        std::cout << i << " ";

    std::integral auto b = true;
    std::cout << b << '\n';

     std::integral auto integ = getIntegral(10);
    std::cout << integ << '\n';

    auto integ1 = getIntegral(10);
    std::cout << integ1 << '\n';

    std::cout << '\n';
}
```

## 使用多于一个concept
同时使用的 concept 往往不止一个，这时候使用布尔运算符将不同的条件连接起来即可。

```C++
template<typename Iter, typename Val>
 requires std::input_iterator<Iter>
  && std::equality_comparable<Value_type<Iter>, Val>
Iter find(Iter b, Iter e, Val v)
```

find 要求迭代器 Iter 及其与 val 的比较满足以下两个条件：
迭代器必须是输入迭代器
迭代器的值类型必须与val相等。

当然，我们也可以等价表示为受约束的模板参数：
```C++
template<std::input_iterator Iter, typename Val>
 requires std::equality_comparable<Value_type<Iter>, Val>
Iter find(Iter b, Iter e, Val v)
```

## 缩写函数模板
在C++20中，既可以在函数声明中使用不受约束的占位符（auto），也可以使用约束的占位符（concept），并且此函数声明会**自动成为函数模板**。

```C++
template <typename T>
requires std::integral<T>
    T gcd(T a, T b)
{
    if (b == 0)
        return a;
    else
        return gcd(b, a % b);
}

template <typename T>
T gcd1(T a, T b) requires std::integral<T>
{
    if (b == 0)
        return a;
    else
        return gcd1(b, a % b);
}

template <std::integral T>
T gcd2(T a, T b)
{
    if (b == 0)
        return a;
    else
        return gcd2(b, a % b);
}

std::integral auto gcd3(std::integral auto a, std::integral auto b)
{
    if (b == 0)
        return a;
    else
        return gcd3(b, a % b);
}

auto gcd4(auto a, auto b)
{
    if (b == 0)
        return a;
    return gcd4(b, a % b);
}

int main()
{
    std::cout << '\n';
    std::cout << "gcd(100, 10)= " << gcd(100, 10) << '\n';
    std::cout << "gcd1(100, 10)= " << gcd1(100, 10) << '\n';
    std::cout << "gcd2(100, 10)= " << gcd2(100, 10) << '\n';
    std::cout << "gcd3(100, 10)= " << gcd3(100, 10) << '\n';
    std::cout << "gcd4(100, 10)= " << gcd4(100, 10) << '\n';
    std::cout << '\n';
}
```

函数模板 gcd3 具有作为类型参数的概念 std::integral，因此成为具有受限类型参数的函数模板。相反，gcd4 等同于对其类型参数没有限制的函数模板。在 gcd3 和 gcd4 中用于创建函数模板的语法被称为缩写函数模板语法。

## 模板匹配优先级
模板匹配的优先级：
完整类型
带约束的 concept
无约束的 auto

Predefined Concepts
three_way_comparable
该 concept 在 `<compare>` 中定义。

Concepts Library
最常见的 concept 都可以在 `<concepts>` 头文件中找到

Language-related concept
same_as
derived_from
convertible_to
common_reference_with
common_with
assignable_from
swappable

Arithmetic Concepts
integral
signed_integral
unsigned_integral
floating_point

定义：

```C++
template<class T>
concept integral = is_integral_v<T>;
template<class T>
concept signed_integral = integral<T> && is_signed_v<T>;
template<class T>
concept unsigned_integral = integral<T> && !signed_integral<T>;
template<class T>
concept floating_point = is_floating_point_v<T>;
```

Lifetime Concepts
destructible
constructible_from
default_constructible
move_constructible
copy_constructible

Comparison Concepts
equality_comparable
totally_ordered

Object Concepts
movable
copyable
semiregular
regular

Callable Concepts
invocable
regular_invocable：可调用、不修改函数参数、给定相同输入时给出相同输出
predicate：可调用、返回值为 bool 类型

Iterators Library
迭代器库有许多重要的 concept。它们在 <iterator> 头中定义。
input_iterator
output_iterator
forward_iterator
bidirectional_iterator
random_access_iterator
contiguous_iterator

|     |     |     |
| --- | --- | --- |
| Iterator Category | Properties | Containers |
| std::forward_iterator | ++It, It++<br>*It<br>It == It2, It != It2 | std::unordered_set<br>std::unordered_map<br>std::unordered_multiset<br>std::unordered_multimap<br>std::forward_list |
| std::bidirectional_iterator | --It, It-- | std::set<br>std::map<br>std::multiset<br>std::multimap<br>std::list |
| std::random_access_iterator | It[i]<br>It += n, It -= n<br>It + n , It - n<br>n + It<br>It - It2 It < It2<br>It <= It2 It > It2, It >= It2 | std::array<br>std::vector<br>std::deque<br>std::string |

随机访问迭代器是双向迭代器，双向迭代器是前向迭代器。连续迭代器是随机访问迭代器，要求容器的元素连续存储在内存中。

## Algorithm Concepts
permutable：可以就地重新排序元素
mergeable：可以将排序后的序列合并到输出序列中
sortable：将序列置换为有序序列是可能的

## Ranges Library
input_range：指定其迭代器类型满足 input_iterator 的范围（例如，可以至少从开始到结束迭代一次）
output_range：指定其迭代器类型满足 output_iterator 的范围
forward_range：指定其迭代器类型满足 forward_iterator 的范围（可以从开始到结束多次迭代）
bidirectional_range：指定其迭代器类型满足 bidirectional_iterator 的范围（可以向前和向后迭代多次）
random_access_range：指定其范围。用索引运算符 [] 取得任意元素的时间相同
contiguous_range：指定迭代器类型满足 contiguous_iterator 的范围（元素连续存储在内存中）

## Defined Concept

concept 的定义以 Template 关键字开头，并具有模板参数列表。第二行使用关键字 concept，后跟 concept 名称和约束表达式，形式如下：
```C++
template <template-parameter-list>
concept concept-name = constraint-expression;
```

其中，约束表达式可以是：
其他 concept 或编译时谓词的逻辑组合requires 表达式
Simple Requirments
Type Requirements
Compound Requirements
Nested Requirements
例如：
```C++
template <typename T>
concept Integral = std::is_integral<T>::value;

template <typename T>
concept SignedIntegral = Integral<T> && std::is_signed<T>::value;

template <typename T>
concept UnsignedIntegral = Integral<T> && !SignedIntegral<T>;
```

## requires语句
使用requires表达式，可以定义更强大的concept。requires表达式具有以下形式：
requires (parameter-list(optional)) {requirement-seq}

parameter-list：类似函数声明中的逗号分隔的参数列表
requirement-seq：requirements序列

### Simple Requirments
```C++
template<typename T>
concept Addable = requires (T a, T b) {
a + b;
};
```

### Type Requirements
```C++
template<typename T>
concept TypeRequirement = requires {
 typename T::value_type;
 typename Other<T>;
};
```

### Compound Requirements
`{expression} noexcept(optional) return-type-requirement(optional);`

Compound Requirements 在 Simple Requirments 的基础上，还可以有一个noexcept 说明符及对其返回类型的要求。

### Nested Requirements
requires constraint-expression;
Nested Requirements 用于指定类型参数的要求。
Nested Requirements 通过嵌套实现了 concept 的细分。

来自《C++ 20 Get the details》

============================

============================

============================

============================

============================

============================

============================

============================

# return value optimization, RVO

============================

# atomic.is_lock_free

https://en.cppreference.com/w/cpp/atomic/atomic/is_lock_free

The C++ standard recommends (but does not require) that lock-free atomic operations are also address-free, that is, suitable for communication between processes using shared memory.

============================

# string_view

============================

============================

# shared_ptr
可以多个  shared_ptr指向  资源，
当最后一个  shared_ptr  被析构时，资源被回收。

# unique_ptr
最多一个unique_ptr指向  资源，unique_ptr销毁时，资源被回收。
复制会报错
可以move

auto_ptr (不建议使用，使用unique_ptr更好)

============================

# 风险指针
hazard pointer

============================

# 内存序

============================

# Message  Passing Interface (MPI,  http://www.mpi-forum.org/)

# OpenMP (http://www.openmp.org/) frameworks

============================

============================

std::experimental::propagate_const

============================

============================

std::result_of<FunctionType()>::type

============================

============================

============================

============================

============================

============================

已使用 Microsoft OneNote 2016 创建。



---


# typedef, define

## typedef
typdef 为类型取一个新的名字

按照惯例，定义时会大写字母，以便提醒用户类型名称是一个象征性的缩写

```C++
typedef unsigned char BYTE;
BYTE  b1, b2;
```

使用 typedef 来为用户自定义的数据类型取一个新的名字。例如，您可以对结构体使用 typedef 来定义一个新的数据类型名字，然后使用这个新的数据类型来直接定义结构变量

```rust
#include <stdio.h>
#include <string.h>

typedef struct Books
{
   char  title[50];
   char  author[50];
   char  subject[100];
   int   book_id;
} Book;

int main( )
{
   Book book;
 
   strcpy( book.title, "C 教程");
   strcpy( book.author, "Runoob"); 
   strcpy( book.subject, "编程语言");
   book.book_id = 12345;
 
   printf( "书标题 : %s\n", book.title);
   printf( "书作者 : %s\n", book.author);
   printf( "书类目 : %s\n", book.subject);
   printf( "书 ID : %d\n", book.book_id);
 
   return 0;
}
```

---


### 别名
```C++
typedef unsigned char u8;
typedef unsigned int u16;

/* 可以这样把类型定义成自己想定义的英语单词 */
int main(void)
{
  u8 hh;
  return 0;
}
```

### 数组
```C++
typedef int Array[20];
 
/* 可以直接定义一个20个元素的数组，类型为Array */
int main(void)
{
  Array array;
}
```

### 隐藏指针
```C++
typedef char* pstr;
```

### 定义函数
```C++
typedef int GUI_GET_DATA_FUNC(void * p, const U8 ** ppData, unsigned NumBytes, U32 Off);
```

---

```C++
typedef int (*PTR_FUNC)（int v1, int v2);
```

此用法一般用在给函数定义别名的时候；上面的例子定义PTR_FUNC是一个函数指针，函数类型是带两个int类型的参数，返回一个int类型的数据；

分析此形式的用法时可以使用下面的方法：
> 先去掉typedef和PTR_FUNC，剩下的就是原变量的类型—— int(*)(int, int)


```C++
int (*ptrAddFunc)(int a, int b);
typedef int (*PTR_ADD_FUNC)(int a, int b);
```
> 第一个语句定义了一个名为ptrAddFunc的变量，这个变量的数据类型为指向拥有二个int类型的参数，返回值为int类型的函数。
> 
> 第二条语句只是将ptrAddFunc变量的数据类型取了一个别名——PTR_TEMP_FUNC。


```C
// 文件名：temp.c
#include <stdio.h>

typedef int (*PTR_FUNC)(int, int);

int Add(int a, int b) { return (a + b);}

int main(void) {
    int (*ptrAdd1)(int a, int b); // 定义一个名为ptrAdd1的变量
    PTR_FUNC ptrAdd2;       // 定义一个名为ptrAdd2的变量
    
    ptrAdd1 = Add;
    ptrAdd2 = Add;

    printf("%d\n", ptrAdd1(10, 20));
    printf("%d\n", ptrAdd2(1, 2));

    return 0;
}
```






---
有一个结构体的名字是 stu，要想定义一个结构体变量
```C++
struct stu stu1;
```

typedef  oldName  newName;

```C++
    typedef int INTEGER;
    INTEGER a, b;
    a = 1;
    b = 2;
```

typedef 还可以给数组、指针、结构体等类型定义别名
```C++
typedef char ARRAY20[20];
```
ARRAY20 是类型char [20]的别名

```C++
ARRAY20 a1, a2, s1, s2;
// ---等价于
char a1[20], a2[20], s1[20], s2[20];
```

```C++
    typedef struct stu{
        char name[20];
        int age;
        char sex;
    } STU;

STU body1,body2;
// -- 等价于
struct stu body1, body2;

```

为指针类型定义别名

```C++
typedef int (*PTR_TO_ARR)[4];

PTR_TO_ARR p1, p2;
```
PTR_TO_ARR 是类型int * [4]的别名，它是一个二维数组指针类型



为函数指针类型定义别名
```C++
typedef int (*PTR_TO_FUNC)(int, int);
PTR_TO_FUNC pfunc;
```



## define

```C++
#include <stdio.h>
 
#define TRUE  1
#define FALSE 0
 
int main( )
{
   printf( "TRUE 的值: %d\n", TRUE);
   printf( "FALSE 的值: %d\n", FALSE);
 
   return 0;
}
```

```C++
#define TITLE "*** Examples of Macros Without Parameters ***"
#define BUFFER_SIZE (4 * 512)
#define RANDOM (-1.0 + 2.0*(double)rand() / RAND_MAX)

// ...
static char myBuffer[BUFFER_SIZE];
setvbuf( fp, myBuffer, _IOFBF, BUFFER_SIZE );
for ( int i = 0; i < ARRAY_SIZE; ++i )
  data[i] = 10.0 * RANDOM;

// ...
static char myBuffer[(4 * 512)];
setvbuf( fp, myBuffer, 0, (4 * 512) );
for ( int i = 0; i < 100; ++i )
data[i] = 10.0 * (-1.0 + 2.0*(double)rand() / 2147483647);

```


### 带参数的宏

```C++
#define 宏名称( [形参列表] ) 替换文本
#define 宏名称( [形参列表 ,] ... ) 替换文本
```

两个函数 getchar（）和 putchar（），它们的宏定义在标准库头文件 stdio.h 中
```C++
    #define getchar() getc(stdin)
    #define putchar(x) putc(x, stdout)
```
当“调用”一个类函数宏时，预处理器会用调用时的实参取代替换文本中的形参。

C99 允许在调用宏的时候，宏的实参列表可以为空。在这种情况下，对应的替换文本中的形参不会被取代；也就是说，替换文本会删除该形参。然而，并非所有的编译器都支持这种“空实参”的做法。

```C++
#include <stdio.h>             // 包含putchar()的定义
#define DELIMITER ':'
#define SUB(a,b) (a-b)
putchar( DELIMITER );
putchar( str[i] );
int var = SUB( ,10);

//----如果 putchar（x）定义为 putc（x，stdout），预处理器会按如下方式展开最后三行代码： 
putc(':', stdout);
putc(str[i], stdout);
int var = (-10);
```

替换文本中所有出现的形参，应该==使用括号将其包围==。这样可以确保无论实参是否是表达式，都能正确地被计算
```C++
#define DISTANCE( x, y ) ((x)>=(y) ? (x)-(y) : (y)-(x))
d = DISTANCE( a, b+0.5 );
```


### 可选参数

C99 标准允许定义有省略号的宏，省略号必须放在参数列表的后面，以表示可选参数。你可以用可选参数来调用这类宏。

```C++
// 假设我们有一个已打开的日志文件，准备采用文件指针fp_log对其进行写入
#define printLog(...) fprintf( fp_log, __VA_ARGS__ )
// 使用宏printLog
printLog( "%s: intVar = %d\n", __func__, intVar );

// ---
fprintf( fp_log, "%s: intVar = %d\n", __func__, intVar );

```


### 字符串化运算符

一元运算符 # 常称为字符串化运算符（stringify operator 或 stringizing operator），因为它会把宏调用时的实参转换为字符串。# 的操作数必须是宏替换文本中的形参。当形参名称出现在替换文本中，并且具有前缀 # 字符时，预处理器会把与该形参对应的实参放到一对双引号中，形成一个字符串字面量。

```C++
#define printDBL( exp ) printf( #exp " = %f ", exp )
printDBL( 4 * atan(1.0));           // atan()在math.h中定义

// --- 展开形式如下
    printf( "4 * atan(1.0)" " = %f ", 4 * atan(1.0));
// --- 编译器会合并紧邻的字符串字面量, 所以上面等价于下面
    printf( "4 * atan(1.0) = %f ", 4 * atan(1.0));
```

### 记号粘贴运算符

使用 ## 运算符时，至少有一个操作数是宏的形参

```C++
#define TEXT_A "Hello, world!"
#define msg(x) puts( TEXT_ ## x )
msg(A);

// ---
    puts( TEXT_A );

// ---
    puts( "Hello, world!" );

```


### 宏内使用宏

在替换实参，以及执行完 # 和 ## 运算之后，预处理器会检查操作所得的替换文本，并展开其中包含的所有宏。但是，宏不可以递归地展开：如果预处理器在 A 宏的替换文本中又遇到了 A 宏的名称，或者从嵌套在 A 宏内的 B 宏内又遇到了 A 宏的名称，那么 A 宏的名称就会无法展开。


### 宏作用域和重新定义

```C++
#undef 宏名称
```
即使准备取消定义的宏是带有参数的，也不需要在 #undef 命令中指定参数列表

当某个宏首次遇到它的 #undef 命令时，它的作用域就会结束。如果没有关于该宏的 #undef 命令，那么它的作用域在该翻译单元结束时终止。



## typedef vs define


typedef 仅限于为类型定义符号名称，#define 不仅可以为类型定义别名，也能为数值定义别名，比如您可以定义 1 为 ONE。

typedef 是由编译器执行解释的，#define 语句是由预编译器进行处理的。

```C++
#define PTR_INT int *
PTR_INT p1, p2;
// ---
int *p1, p2;
```

```C++
typedef int * PTR_INT
PTR_INT p1, p2;
```
p1、p2 类型相同，它们都是指向 int 类型的指针。




# 初始化方式

https://learn.microsoft.com/zh-cn/cpp/cpp/initializers?view=msvc-170

## 零初始化

将变量设置为 将0隐式转换为该类型后的值。
- 程序启动时，对具有静态持续事件的所有 命名变量进行初始化。 这些变量 可以 稍后再次初始化
- 值初始化期间，对使用 空大括号 初始化 的 标量类型 和 POD 类型 进行初始化
- 对 只有部分成员初始化的 数组 进行初始化

```C++
struct my_struct{
    int i;
    char c;
};

int i0;              // zero-initialized to 0
int main() {
    static float f1;  // zero-initialized to 0.000000000
    double d{};     // zero-initialized to 0.00000000000000000
    int* ptr{};     // initialized to nullptr
    char s_array[3]{'a', 'b'};  // the third char is initialized to '\0'
    int int_array[5] = { 8, 9, 10 };  // the fourth and fifth ints are initialized to 0
    my_struct a_struct{};   // i = 0, c = '\0'
}
```

## 默认初始化

类，结构，联合 的默认初始化 是 具有默认构造函数的 初始化。
可以 不使用 初始化表达时 或 使用 new 关键字 调用 默认构造器

```C++
class MyClass {};

MyClass mc1;
MyClass& mc2 = new MyClass;
```

定义标量变量时 不使用 初始化表达式，则进行 默认初始化。 ==值是不确定的==
```C++
int i1;
float f;
char c;
```

定义数组时，不使用 初始化表达时，则进行 默认初始化。 数组元素 进行 默认初始化 并 ==具有不确定的值==
```C++
int arr[3];
```
如果 数据元素 没有 默认构造器，则编译错误

### 常量的默认初始化
常量变量 必须和 初始值 一起声明。
如果没有初始值， 标量类型 会编译器错误，  具有默认构造器的 类类型 会 警告

### 静态变量的默认初始化
如果静态变量的声明中没有初始值设定项，则初始化为 0 (隐式转为该类型)

## 值初始化
发生在以下情况
- 使用 空 大括号初始化 来初始化 已命名值
- 使用 空 圆括号或大括号 初始化 匿名临时对象
- 对象是 使用 new 和 空 圆括号或大括号 来初始化的

执行以下操作
- 对至少有一个公共构造器的类，将调用 默认构造器
- 对没有 声明构造器的 非联合类，该对象进行 0初始化，并调用 默认构造器
- 对于数组，每个元素 都进行 值初始化
- 其他所有情况下，变量进行 0初始化


## 复制初始化
是指使用一个不同的对象来初始化另一个对象。

在以下情况下，它会发生：
- 使用 等号 初始化变量
- 参数被传递给函数
- 从函数返回对象
- 引发 或 捕获异常
- 使用等号初始化 非静态数据成员
- 在 聚合初始化 期间 通过 复制初始化 来初始化类，结构和联合成员。 示例查看 聚合初始化

```C++
#include <iostream>
using namespace std;

class MyClass{
public:
    MyClass(int myInt) {}
    void set_int(int myInt) { m_int = myInt; }
    int get_int() const { return m_int; }
private:
    int m_int = 7; // copy initialization of m_int

};
class MyException : public exception{};
int main() {
    int i = 5;              // copy initialization of i
    MyClass mc1{ i };
    MyClass mc2 = mc1;      // copy initialization of mc2 from mc1
    MyClass mc1.set_int(i);    // copy initialization of parameter from i
    int i2 = mc2.get_int(); // copy initialization of i2 from return value of get_int()

    try{
        throw MyException();
    }
    catch (MyException ex){ // copy initialization of ex
        cout << ex.what();
    }
}
```

复制初始化 不能调用 显式构造器
```C++
vector<int> v = 10; // the constructor is explicit; compiler error C2440: can't convert from 'int' to 'std::vector<int,std::allocator<_Ty>>'
regex r = "a.*b"; // the constructor is explicit; same error
shared_ptr<int> sp = new int(1729); // the constructor is explicit; same error
```

如果类的复制构造器 被删除 或不可访问，复制初始化 会导致 编译器错误

## 直接初始化
使用 非空 大括号或圆括号的 初始化。
不同于复制初始化，直接初始化可以 调用 显式构造器

在以下情况，它会发生
- 使用  非空 大括号或圆括号 初始化变量
- 变量使用 new 和 非空大括号或圆括号 来初始化
- 使用 static_cast 初始化变量
- 在构造器中，使用 初始化列表 初始化基类 和 非静态成员
- lambda 中 捕获得 变量的副本

```C++
class BaseClass{
public:
    BaseClass(int n) :m_int(n){} // m_int is direct initialized
private:
    int m_int;
};

class DerivedClass : public BaseClass{
public:
    // BaseClass and m_char are direct initialized
    DerivedClass(int n, char c) : BaseClass(n), m_char(c) {}
private:
    char m_char;
};
int main(){
    BaseClass bc1(5);
    DerivedClass dc1{ 1, 'c' };
    BaseClass* bc2 = new BaseClass(7);
    BaseClass bc3 = static_cast<BaseClass>(dc1);

    int a = 1;
    function<int()> func = [a](){  return a + 1; }; // a is direct initialized
    int n = func();
}
```

## 列表初始化
使用大括号内的 初始值列表 初始化变量时，将发生 列表初始化。
大括号内的 初始值列表 可以在 下列情况中使用
- 初始化变量
- 类是使用 new 来初始化的
- 从函数返回对象
- 自变量传递给函数
- 直接初始化中的 自变量之一
- 在非静态数据成员 的初始值中
- 在构造器 初始化列表中

```C++
class MyClass {
public:
    MyClass(int myInt, char myChar) {}
private:
    int m_int[]{ 3 };
    char m_char;
};
class MyClassConsumer{
public:
    void set_class(MyClass c) {}
    MyClass get_class() { return MyClass{ 0, '\0' }; }
};
struct MyStruct{
    int my_int;
    char my_char;
    MyClass my_class;
};
int main() {
    MyClass mc1{ 1, 'a' };
    MyClass* mc2 = new MyClass{ 2, 'b' };
    MyClass mc3 = { 3, 'c' };

    MyClassConsumer mcc;
    mcc.set_class(MyClass{ 3, 'c' });
    mcc.set_class({ 4, 'd' });

    MyStruct ms1{ 1, 'a', { 2, 'b' } };
}
```

## 聚合初始化
是针对 数据 或 类类型 (通常为 结构 或联合) 的一种 列表初始化 方式
- 没有 私有或受保护的 成员
- 没有用户提供的 构造器，显式默认 或 删除 的构造器除外
- 没有基类
- 没有虚函数

聚合初始值 包括 等号 或 不含等号的 括号内的 初始化列表

```C++
#include <iostream>
using namespace std;

struct MyAggregate{
    int myInt;
    char myChar;
};

struct MyAggregate2{
    int myInt;
    char myChar = 'Z'; // member-initializer OK in C++14
};

int main() {
    MyAggregate agg1{ 1, 'c' };
    MyAggregate2 agg2{2};
    cout << "agg1: " << agg1.myChar << ": " << agg1.myInt << endl;
    cout << "agg2: " << agg2.myChar << ": " << agg2.myInt << endl;

    int myArr1[]{ 1, 2, 3, 4 };
    int myArr2[3] = { 5, 6, 7 };
    int myArr3[5] = { 8, 9, 10 };

    cout << "myArr1: ";
    for (int i : myArr1){
        cout << i << " ";
    }
    cout << endl;

    cout << "myArr3: ";
    for (auto const &i : myArr3) {
        cout << i << " ";
    }
    cout << endl;
}
```

### 初始化联合 和 结构

```C++
struct MyStruct {
    int myInt;
    char myChar;
};
union MyUnion {
    int my_int;
    char my_char;
    bool my_bool;
    MyStruct my_struct;
};

int main() {
    MyUnion mu1{ 'a' };  // my_int = 97, my_char = 'a', my_bool = true, {myInt = 97, myChar = '\0'}
    MyUnion mu2{ 1 };   // my_int = 1, my_char = 'x1', my_bool = true, {myInt = 1, myChar = '\0'}
    MyUnion mu3{};      // my_int = 0, my_char = '\0', my_bool = false, {myInt = 0, myChar = '\0'}
    MyUnion mu4 = mu3;  // my_int = 0, my_char = '\0', my_bool = false, {myInt = 0, myChar = '\0'}
    //MyUnion mu5{ 1, 'a', true };  // compiler error: C2078: too many initializers
    //MyUnion mu6 = 'a';            // compiler error: C2440: cannot convert from 'char' to 'MyUnion'
    //MyUnion mu7 = 1;              // compiler error: C2440: cannot convert from 'int' to 'MyUnion'

    MyStruct ms1{ 'a' };            // myInt = 97, myChar = '\0'
    MyStruct ms2{ 1 };              // myInt = 1, myChar = '\0'
    MyStruct ms3{};                 // myInt = 0, myChar = '\0'
    MyStruct ms4{1, 'a'};           // myInt = 1, myChar = 'a'
    MyStruct ms5 = { 2, 'b' };      // myInt = 2, myChar = 'b'
}
```

### 初始化包含聚合的聚合

```C++
struct MyStruct {
    int myInt;
    char myChar;
};
int main() {
    int intArr1[2][2]{{ 1, 2 }, { 3, 4 }};
    int intArr3[2][2] = {1, 2, 3, 4};
    MyStruct structArr[]{ { 1, 'a' }, { 2, 'b' }, {3, 'c'} };
}
```


## 引用初始化
引用类型的变量必须使用引用类型派生自的类型的对象进行初始化，或使用可转换为引用类型派生自的类型的类型的对象进行初始化

使用 临时 对象初始化引用的唯一方式是初始化 常量 临时对象。

---
---
---

## 指定初始化 C++20 designated initializers

```C++
struct Point {
    int x;
    int y;
};

Point p{1, 2}; // 在C++20之前的方式
Point p{.x = 1, .y = 2}; // C++20 指定初始化的方式
```

```C++
int arr[3] = {[1] = 2}; // 数组指定初始化
 
class MyClass {
public:
    MyClass(int a, int b) : a_(a), b_(b) {}
private:
    int a_;
    int b_;
};
 
MyClass obj{.a_ = 1, .b_ = 2}; // 类成员指定初始化
```

# decl

decltype
编译器 根据表达式值 获得其类型，典型用法： `using type = decltype(expr);`


std::declval

定义
```C++
template<class T>
typename std::add_rvalue_reference<T>::type declval() noexcept;
```

根据类型获得 值，`auto&& a = std::declval<type>();`
std::declval 没有定义，所以只能用于 模板推导中


---

# c++之美:代码整洁、安全又跑得快的30个要诀


第1章 避重就轻不可取 21
1.1 P.2：使用ISO C++标准编写代码 23
1.2 F.51：有选择时优先使用默认参数而非重载 33
1.3 C.45：不要定义仅初始化数据成员的默认构造函数，而应使用类成员初始化 43
1.4 C.131：避免平凡的get和set函数 51
1.5 ES.10：每条语句只声明一个名字 61
1.6 NR.2：不强求函数只用一条return语句 69

第2章 不要伤害自己 79
2.1 P.11：将凌乱的结构封装起来，而不是使其散布于代码中 81
2.2 I.23：尽量减少函数参数 91
2.3 I.26：使用C风格子集获取跨编译器的ABI 99
2.4 C.47：按成员声明顺序定义并初始化成员变量 107
2.5 CP.3：尽量减少可写数据的显式共享 117
2.6 T.120：只在真正需要时使用模板元编程 127

第3章 别再使用 139
3.1 I.11：切勿通过原生指针（T*）或引用（T&）转移所有权 141
3.2 I.3：避免使用单例 149
3.3 C.90：依靠构造函数和赋值运算符，而不是memset和memcpy 159
3.4 ES.50：不要用强制转换去除const限定符 169
3.5 E.28：避免基于全局状态（如errno）的错误处理 179
3.6 SF.7：不要在头文件的全局作用域写using namespace 189

第4章 正确使用新特性 199
4.1 F.21：优先选择结构体或元组返回多个“输出”值 201
4.2 Enum.3：优先选择类枚举而不是“普通”枚举 213
4.3 ES.5：保持作用域小 221
4.4 Con.5：使用constexpr表示编译时可以计算的值 233
4.5 T.1：使用模板提高代码的抽象层次 245
4.6 T.10：为所有模板参数指定概念 255

第5章 默认写出好代码 265
5.1 P.4：理想情况下，程序应具有静态类型安全性 267
5.2 P.10：优先选择不可变数据而不是可变数据 279
5.3 I.30：封装违反规则的部分 287
5.4 ES.22：确定初始值后再声明变量 295
5.5 Per.7：为促成优化而设计 305
5.6 E.6：使用RAII防止泄露 313
后记 325




# std::ref

std::ref 是 C++的一个 工具函数，用于将 对象封装为 引用包装器 (std::reference_wrapper)。
避免不必要的 对象拷贝。

常用于 函数式编程。 解决 函数参数 只能通过 值传递的 问题，使得值可以通过 引用传递。

## 基本用法

在函数式编程中，使用 std::bind, std::function 时，如果希望传递的参数能通过 ref传递 而不是 值传递，那么可以使用 std::ref

```C++
#include <functional>
#include <iostream>

void f(int& n1, int& n2, const int& n3) {
    std::cout << "In function: " << n1 << ' ' << n2 << ' ' << n3 << '\n';
    ++n1; // increments the copy of n1 stored in the function object
    ++n2; // increments the main()'s n2
    // ++n3; // compile error
}

int main() {
    int n1 = 1, n2 = 2, n3 = 3;
    std::function<void()> bound_f = std::bind(f, n1, std::ref(n2), std::cref(n3));
    n1 = 10;
    n2 = 11;
    n3 = 12;
    std::cout << "Before function: " << n1 << ' ' << n2 << ' ' << n3 << '\n';
    bound_f();
    std::cout << "After function: " << n1 << ' ' << n2 << ' ' << n3 << '\n';
}
```

std::bind 的第二个参数 n2 使用了 std::ref，所以 函数f中 对n2的修改 会影响外面的 n2。


## 原理

std::ref 通过 模板参数推导 生成一个 std::reference_wrapper对象， 它可以 隐式 转为 被ref 的 值的 引用类型。

---

在多线程编程中，线程函数的参数 需要 按ref传递时，必须使用 std::ref 进行包装。

---

https://en.cppreference.com/w/cpp/utility/functional/reference_wrapper

std::reference_wrapper 是 copy-constructible 和 copy-assignable的。 是对 T类型的 引用的 一个包装， 它可以隐式转为`T&`。

helper函数 std::ref, std::cref 用于 生成 std::reference_wrapper。

std::reference_wrapper 用于 通过引用 把 对象 传递给 std::bind, std::thread的构造器，std::make_pair，std::make_tuple。

也可以作为一种机制，来 在 标准容器(如 std::vector) 中 保存 引用。  标准容器 一般不能 hold 引用。 。。不过标准容器 可以 直接 保存指针。



# incomplete type

在 std::reference_wrapper 的 cppreference中有一句:
`T may be an incomplete type. (since C++20)`

---

baidu ai:

不完整类型 是 指在编程中，一种类型在编译阶段存在，但其内部结构并未被完全定义的情况。  

这种类型通常通过==前置声明==来表示，告诉编译器这个类型存在，但是现在 还不知道 它的具体结构。

限制
- 不能创建 不完整类型的 实例
- 不能获取大小 (sizeof)
- 不能访问成员

---

ms:

https://learn.microsoft.com/en-us/cpp/c-language/incomplete-types?view=msvc-170

描述了 标识符，但是 缺少 可以获得标识符大小的 信息

可能是:
- 还未指定成员的 结构类型
- 还未指定成员的 union类型
- 还未指定成员的 数组

void 类型 是 不完整类型， 且 无法 完整。

例子:
`struct student *ps;`  // incomplete
`struct student { int num; }`  // now complete

`char a[];`  // incomplete
`char a[22];`  // now complete



# std::shared_ptr

使用`std::make_shared<int>(22);` 来构建。
优点:
- 一次内存分配就搞定 对象 和 计数，这2个的 存储空间， 提高 内存分配效率，减少内存占用
- 异常安全， 即 构造对象时，如果抛出异常，不会留下 未删除的 计数 存储空间 。  。。使用 new 的话，如果 是 `new shared_ptr<int>(new int(22));` 里面的 new int成功，但是 new shared_ptr 失败，那么 里面 new 的int 不会被释放。

---


https://zhuanlan.zhihu.com/p/711161292


核心特性
- 共享所有权， shared_ptr 可以 复制，移动， 共享一份资源， 都参与 ref count的维护。
- 弱引用， std::weak_ptr, 不会增加 ref count， 用于解决循环ref 问题。
- 自定义删除器
- 原子性保证， ref count 的 增减是原子性的

---

重要接口

shared_ptr可以从 裸指针，shared_ptr，auto_ptr，weak_ptr 构造。
构造器的第二个参数是 删除器。对于 不是new分配 也不是 delete 释放的 资源 非常有用。
shared_ptr 可以像 普通指针一样使用， 除了 不能被显式地删除。

```C++
// p必须是有效的 指向T 的指针。 构造后 ref count 为1.
// 唯一可能抛出的异常是 std::bad_alloc ( 无法申请到 保存 ref count 所需要的空间)
template<class T>
explicit shared_ptr(T* p);
```

```C++
// 第二个是删除器，使用方法是: `d(p)`。
// 会抛出 std::bad_alloc
template<class T, class D>
shared_ptr(T* p, D d);
```

```C++
// r 中的资源 被新构造的 shared_ptr 共享， ref count +1,
// 不会抛出异常
shared_ptr(const shared_ptr& r);
```

```C++
// 从 weak_ptr 构造 shared_ptr，这使得 weak_ptr的使用具有线程安全性，因为 ref count需要+1。
// ==如果 weak_ptr为空( r.use_count() == 0)，则抛出 bad_weak_ptr 异常==
template<class T>
explicit shared_ptr(const weak_ptr<T>& r)
```

```C++
// 从 auto_ptr 获取资源的所有权 ( 复制指针 并 对auto_ptr调用 release() )。
// 构造后， ref count 为1， r变空。
// 会抛出 std::bad_alloc
template<typename T>
shared_ptr(auto_ptr<T>& r)
```

```C++
// 析构， ref count -1， 如果为0，则 删除 指针 ( 调用 operator delete， 或使用 删除器 d(p))。
// 析构函数不会抛出异常
~shared_ptr();
```

---

```C++
// 赋值操作， 共享 r 中的资源，并停止 对 原资源的共享
// 不会抛出异常
shared_ptr& operator =(const shared_ptr& r);
```

```C++
// 停止 对保存指针的 所有权的共享， ref cnt -1.
void reset();
```

```C++
// 返回 保存的指针 指向的对象的 引用。
// 如果 指针为空，未定义
// 不会抛出异常
T& operator*() const;
```

```C++
// 返回保存的指针， 这个操作符 与 operator* 一起，使得 shared_ptr 看起来 像 普通指针。
// 不会抛异常
T* operator->() cosnt;
```

```C++
// 返回保存的指针， 特别适合于 指针可能为空的情况 ( 也可以通过 隐式布尔类型转换 来判断 shared_ptr 是否含有 有效指针)
// 不会抛出异常
T* get() const;
```

```C++
// 本shared_ptr 是 资源的唯一拥有者 时 返回true
// 不会抛出异常
bool unique() const;
```

```C++
// 返回 指针的 ref count。   在调试时很有用，在关键点获得 ref count 的值。 小心使用，因为 代价昂贵
// 不会抛出异常
long use_count() const;
```

```C++
// 交换2个 shared_ptr
// 不会抛出异常
void swap(shared_ptr<T>& b);
```


```C++
// 要对 保存在 shared_ptr中的 指针指向 static_cast， 我们可以 取出指针，然后 强转， 但不能保存到 另一个 shared_ptr 中，因为 新的shared_ptr 会认为 它是第一个管理该资源的。
// 解决方案是 使用 static_pointer_cast， 可以保证 ref count 正确
// 不会抛出异常
template <typename T,typename U>
shared_ptr<T> static_pointer_cast(const shared_ptr<U>& r);
```

---

使用例子

```C++
#include <iostream>
#include <assert.h>

using namespace std;

int main() {
    shared_ptr<int> pInt1;
    assert(pInt1.use_count() == 0);

    {
        shared_ptr<int> pInt2(new int(66));
        assert(pInt1.use_count() == 1);

        pInt1 = pInt2;
        assert(pInt1.use_count() == 2);
        assert(pInt2.use_count() == 2);
    }

    assert(pInt1.use_count() == 1);

    return 0;
}
```

如果资源的创建 销毁 不是使用 new delete，那么需要指定删除器

```C++
#include <iostream>
using namespace std;
class CFileCloser {
public:
    void operator()(FILE* f) {
        if (f != NULL) {
            fclose(f);
            f = NULL;
        }
    }
};

int main() {
    shared_ptr<FILE> fp(fopen("C:\\1.txt", "r"), CFileCloser());
    return 0;
}
```


使用时，要放置 对一个指针 进行2次构造
```C++
{
    int *pint = new int(66);
    shared_ptr<int> sp1(pint);
    shared_ptr<int> sp2(pint);
    
    assert(sp1.use_count() == 1);
    assert(sp2.use_count() == 1);
} // 退出时 sp1, sp2 都会 对 pint 进行delete

// 在 原始指针 被传递给 shared_ptr后， 后续的操作都要对 智能指针进行
{
    shared_ptr<int> sp1(new int(55));
    assert(sp1.use_count() == 1);

    shared_ptr<int> sp2(sp1);
    assert(sp2.use_count() == 2);
}
```

在多线程中使用 shared_ptr时， 如果存在 拷贝 或赋值 操作，可能会由于 同时访问 计数 而导致 计数无效。 解决方法是: 向每个线程传递 公共的 weak_ptr， 需要使用 shared_ptr时，将 weak_ptr 转为 shared_ptr。


---

注意事项

1. 初始化与赋值， 可以通过 构造器 直接初始化，也可以通过 成员函数 reset 或 赋值运算符 改变所指向的对象
2. 循环引用， 使用 weak_ptr。
3. 性能开销， 更新ref count 需要性能开销。 对于 频繁创建销毁的小对象 或 单个owner的场景，使用 unique_ptr。
4. 异常安全， 具有异常安全保证，即 构造或赋值过程中 抛出异常，也不会导致 内存泄漏。




