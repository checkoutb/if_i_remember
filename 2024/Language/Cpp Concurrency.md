Cpp Concurrency
2023年3月18日
22:43

cppreference

C++  并发编程实战  第二版

=====================

https://en.cppreference.com/w/cpp/thread/packaged_task

reset

=====================

=====================

多进程并发
将应用程序分为多个独立的进程
独立的进程可以通过进程间常规的通信渠道传递讯息(信号、套接字、文件、管道等等)

使用多进程实现并发还有一个额外的优势———可以使用远程连接(可能需要联网)的方式，在不同的机器上运行独立的进程
。。不同机器上，  微服务？

多线程并发
进程中的所有线程都共享地址空间，并且所有线程访问到大部分数据———全局变量仍然是全局的，指针、对象的引用或数据可以在线程之间传递。

虽然，进程之间通常共享内存，但是这种共享通常是难以建立和管理的。因为，同一数据的内存地址在不同的进程中是不相同

多个单线程/进程间的通信(包含启动)要比单一进程中的多线程间的通信(包括启动)的开销大，若不考虑共享内存可能会带来的问题，多线程将会成为主流语言(包括C++)更青睐的并发途径。此外，C++标准并未对进程间通信提供任何原生支持

为什么使用并发？
主要原因有两个：关注点分离(SOC)和性能

编写软件时，分离关注点是个好主意。通过将相关的代码与无关的代码分离，可以使程序更容易理解和测试，从而减少出错的可能性

两种方式利用并发提高性能
第一，将一个单个任务分成几部分，且各自并行运行，从而降低总运行时间。这就是任务并行（task parallelism）。
各个部分之间可能存在着依赖
在过程方面——一个线程执行算法的一部分，而另一个线程执行算法的另一个部分
在数据方面——每个线程在不同的数据部分上执行相同的操作（第二种方式）。后一种方法被称为数据并行（data  parallelism）

第一种可并行执行的算法常被称为易并行(embarrassingly parallel)算法
易并行算法具有良好的可扩展特性——当可用硬件线程的数量增加时，算法的并行性也会随之增加。

第二种方法是使用可并行的方式，来解决更大的问题；与其同时处理一个文件，不如酌情处理2个、10个或20个。
这是数据并行的一种应用，但着重点不同

处理一个数据块仍然需要同样的时间，但在相同的时间内处理了更多的数据。当然，这种方法也有限制，并非在所有情况下都是有益的。不过，这种方法所带来的吞吐量提升，可以让某些新功能成为可能，例如：并行处理图片就能提高视频的分辨率。

什么时候不使用并发
上下文切换

为性能而使用并发就像所有其他优化策略一样:它拥有大幅度提高应用性能的潜力，但它也可能让代码更加复杂，更难以理解，并且更容易出错。

应用中只有具有显著增益潜力的性能关键部分，才值得进行并发化。

#include <iostream>
#include <thread> // 1
void hello() // 2
{
std::cout << "Hello Concurrent World\n";
}
int main()
{
std::thread t(hello); // 3
t.join(); // 4
}

因为每个线程都必须具有一个初始函
数(initial function)，新线程的执行从这里开始。对于应用程序来说，初始线程是main()，但是
对于其他线程，可以在 std::thread 对象的构造函数中指定

-----------------------
第2章  线程管理

启动一个线程，
等待这个线程结束，
或放在后台运行。

给已经启动的线程函数传递参数

将一个线程的所有权从当前 std::thread 对象移交给另一个。

确定线程数，以及识别特殊线程。

----------------

使用C++线程库启动线程，可以归结为构造 std::thread 对象
void do_some_work();
std::thread my_thread(do_some_work);

std::thread 可以用可调用类型构造，将带有函数调用符类型的实例传入 std::thread 类中，替换默认的构造函数。
class background_task
{
public:
void operator()() const
{
do_something();
do_something_else();
}
};
background_task f;
std::thread my_thread(f);

提供的函数对象会复制到新线程的存储空间当中
函数对象的执行和调用都在线程的内存空间中进行。
函数对象的副本应与原始函数对象保持一致，否则得到的结果会与我们的期望不同

注意
如果你传递了一个临时变量，而不是一个命名的变量；C++编译器会将其解析为函数声明，而不是类型对象的定义
std::thread my_thread(background_task());

这里相当与声明了一个名为my_thread的函数，返回一个 std::thread 对象的函数，而非启动了一个线程

使用在前面命名函数对象的方式，或使用多组括号①，或使用新统一的初始化语法②，可以避免这个问题。
std::thread my_thread((background_task())); // 1
std::thread my_thread{background_task()}; // 2

使用lambda表达式也能避免这个问题
std::thread my_thread([]{
do_something();
do_something_else();
});
。。这个lambda  没有  ()，  但是可以。

启动了线程，你需要确定是  等待线程结束(加入式)，  还是  让其自主运行(分离式)

如果不等待线程，就必须保证线程结束之前，可访问的数据的有效性

这种情况很可能发生在线程还没结束，函数已经退出的时候，这时线程函数还持有函数局部变量的指针或引用。下面的清单中就展示了这样的一种情况。

struct func
{
int& i;
func(int& i_) : i(i_) {}
void operator() ()
{
for (unsigned j=0 ; j<1000000 ; ++j)
{
do_something(i); // 1 潜在访问隐患：悬空引用
}
}
};
void oops()
{
int some_local_state=0;
func my_func(some_local_state);
std::thread my_thread(my_func);
my_thread.detach(); // 2 不等待线程结束
} // 3 新线程可能还在运行

处理这种情况的常规方法：使线程函数的功能齐全，将数据复制到线程中

此外，可以通过join()函数来确保线程在函数完成前结束

当你需要对等待中的线程有更灵活的控制时，比如，看一下某个线程是否结束，或者只等待一段时间(超过时间就判定为超时)。想要做到这些，你需要使用其他机制来完成，比如条件变量和期待(futures)，相关的讨论将会在第4章继续。

调用join()的行为，还清理了线程相关的存储部分，这样 std::thread 对象将不再与已经
完成的线程有任何关联。这意味着，只能对一个线程使用一次join();一旦已经使用过
join()， std::thread 对象就不能再次加入了，当对其使用joinable()时，将返回false。

如果想要分离一个线程，可以在线程启动后，直接使用detach()进行分离。
如果打算等待对应线程，则需要细心挑选调用join()的位置。当在线程运行之后产生异常，在join()调用之前抛出，就意味着这次调用会被跳过。

避免应用被抛出的异常所终止，就需要作出一个决定。通常，当倾向于在无异常的情况下使用join()时，需要在异常处理过程中调用join()，从而避免生命周期的问题

struct func; // 定义在清单2.1中
void f()
{
int some_local_state=0;
func my_func(some_local_state);
std::thread t(my_func);
try
{
do_something_in_current_thread();
}
catch(...)
{
t.join(); // 1
throw;
}
t.join(); // 2
}

一种方式是使用“资源获取即初始化方式”(RAII
在析构函数中使用join()

class thread_guard
{
std::thread& t;
public:
explicit thread_guard(std::thread& t_):
t(t_)
{}
~thread_guard()
{
if(t.joinable()) // 1
{
t.join(); // 2
}
}
thread_guard(thread_guard const&)=delete; // 3
thread_guard& operator=(thread_guard const&)=delete;
};
struct func; // 定义在清单2.1中
void f()
{
int some_local_state=0;
func my_func(some_local_state);
std::thread t(my_func);
thread_guard g(t);
do_something_in_current_thread();
} // 4

当线程执行到④处时，局部对象就要被逆序销毁了。因此，thread_guard对象g是第一个被销毁的，这时线程在析构函数中被  join  到原始线程中。即使do_something_in_current_thread抛出一个异常，这个销毁依旧会发生

。。换行要用  \n，  不能  endl，  多线程的情况下，  endl  和  它前面的  string  并不是原子的。

通常称分离线程为守护线程(daemon threads)，
UNIX中守护线程是指，没有任何显式的用户接口，并在后台运行的线程。
这种线程的特点就是长时间运行；线程的生命周期可能会从某一个应用起始到结束，可能会在后台监视文件系统，还有可能对缓存进行清理，亦或对数据结构进行优化。
另一方面，分离线程的另一方面只能确定线程什么时候结束，发后即忘(fireand forget)的任务就使用到线程的这种方式。

当  std::thread  对象使用t.joinable()返回的是true，就可以使用t.detach()

文字处理应用同时编辑多个文档
一种内部处理方式是，让每个文档处理窗口拥有自己的线程；每个线程运行同样的的代码，并隔离不同窗口处理的数据
void edit_document(std::string const& filename)
{
open_document_and_display_gui(filename);
while(!done_editing())
{
user_command cmd=get_user_input();
if(cmd.type==open_new_document)
{
std::string const new_name=get_filename_from_user();
std::thread t(edit_document,new_name); // 1
t.detach(); // 2
}
else
{
process_user_input(cmd);
}
}
}

如果用户选择打开一个新文档，需要启动一个新线程去打开新文档①，并分离线程②。

edit_document函数可以复用，通过传参的形式打开新的文件

不仅可以向 std::thread 构造函数①传递函数名，还可以传递函数所需的参数(实参)
也有其他方法完成这项功能，比如:使用一个带有数据成员的成员函数，代替一个需要传参的普通函数

2.2  向线程函数传递参数

需要注意的是，默认参数要拷贝到线程独立内存中，即使参数是引用的形式

void f(int i, std::string const& s);
std::thread t(f, 3, "hello");

函数f需要一个 std::string 对象作为第二个参数，但这里使用的是字符串的字面值，也就是 char const * 类型。之后，在线程的上下文中完成字面值向 std::string 对象的转化。

特别要注意，当指向动态变量的指针作为参数传递给线程的情况
void f(int i,std::string const& s);
void oops(int some_param)
{
char buffer[1024]; // 1
sprintf(buffer, "%i",some_param);
std::thread t(f,3,buffer); // 2
t.detach();
}

函数有很有可能会在字面值转化成 std::string 对象之前崩溃(oops)，从而导

致一些未定义的行为。并且想要依赖隐式转换将字面值转换为函数期待的 std::string 对象，但因 std::thread 的构造函数会复制提供的变量，就只复制了没有转换成期望类型的字符串字面值。

。。。？？？  感觉意思是，  char[]  转  string，出错的话，由于已经进入了  thread  类内部，所以会导致未定义行为。？

解决方案就是在传递到  std::thread  构造函数之前就将字面值转化为  std::string  对象
std::thread t(f,3,std::string(buffer)); // 使用std::string，避免悬垂指针

还可能遇到相反的情况：期望传递一个非常量引用(但这不会被编译)，但整个对象被复制了。你可能会尝试使用线程更新一个引用传递的数据结构

void update_data_for_widget(widget_id w,widget_data& data); // 1
void oops_again(widget_id w)
{
widget_data data;
std::thread t(update_data_for_widget,w,data); // 2
display_status();
t.join();
process_widget_data(data);
}

可以使用  std::ref  将参数转换成引用的形式
std::thread t(update_data_for_widget,w,std::ref(data));

。。想要引用，传实参，会被复制。  所以  要特别说明这个  是  引用。
。。不知道  thread  里面怎么处理的。  是不是  判断是否为  std::ref  ？
。。我试了下，  这里  强制要求  std::ref  的，  不写的话，编译失败。。

如果你熟悉 std::bind ，就应该不会对以上述传参的形式感到奇怪，因为 std::thread 构造函数和 std::bind 的操作都在标准库中定义好了，

可以传递一个成员函数指针作为线程函数，并提供一个合适的对象指针作为第一个参数

class X
{
public:
void do_lengthy_work();
};
X my_x;
std::thread t(&X::do_lengthy_work,&my_x); // 1

新线程将
my_x.do_lengthy_work()作为线程函数；
my_x的地址①作为指针对象提供给函数。
也可以为成员函数提供参数： std::thread 构造函数的第三个参数就是成员函数的第一个参数，以此类推。

提供的参数可以移动，但不能拷贝
std::unique_ptr  就是这样一种类型

同一时间内，只允许一个 std::unique_ptr 实现指向一个给定对象，并且当这个实现销毁时，指向的对象也将被删除。移动构造函数(move constructor)和移动赋值操作符(move assignment operator)允许一个对象在多个 std::unique_ptr 实现中传递

移动操作可以将对象转换成可接受的类型，例如:函数参数或函数返回的类型。当原对象是一个临时变量时，自动进行移动操作，但当原对象是一个命名变量，那么转移的时候就需要使用 std::move() 进行显示移动。下面的代码展示了 std::move 的用法，展示了 std::move 是如何转移一个动态对象到一个线程中去的：

void process_big_object(std::unique_ptr<big_object>);
std::unique_ptr<big_object> p(new big_object);
p->prepare_data(42);
std::thread t(process_big_object,std::move(p));

。。这个主要是因为  函数使用了  unique_ptr  ，又想用  thread的  参数自动转换  功能。。不然的话
。。也不对啊，  直接  unique_ptr  发进去就可以了啊，  unique_ptr  复制的时候  自动  保证唯一啊。
。。也不是，  直接  unique_ptr  的话，  外部  是无法  再看到这个  对象了。
。。这个不清楚。

2.3  转移线程所有权
假设要写一个在后台启动线程的函数，并想通过新线程返回的所有权去调用这个函数，而不是等待线程结束再去调用；
或完全与之相反的想法：创建一个线程，并在函数中转移所有权，都必须要等待线程结束。所以，新线程的所有权都需要转移。

创建了两个执行线程，并且在  std::thread  实例之间(t1,t2和t3)转移所有权
void some_function();
void some_other_function();
std::thread t1(some_function); // 1
std::thread t2=std::move(t1); // 2
t1=std::thread(some_other_function); // 3   所有者是一个临时对象——移动操作将会隐式的调用
std::thread t3; // 4
t3=std::move(t2); // 5
t1=std::move(t3); // 6 赋值操作将使程序崩溃

第六步，  t1已经有了一个关联的线程(执行some_other_function的线程)，所以这里系统直接调用 std::terminate() 终止程序继续运行。这样做（不抛出异常， std::terminate() 是noexcept函数)是为了保证与 std::thread 的析构函数的行为一致。2.1.1节中，需要在线程对象被析构前，显式的等待线程完成，或者分离它；进行赋值时也需要满足这些条件(说明：不能通过赋一个新值给 std::thread 对象的方式来"丢弃"一个线程)

。。。但  6  怎么崩溃。。
。。但是  析构器  中  又不是  调用  terminate，  难道  terminate中调用  析构器？？？
。。怎么知道  自己会被赋值？  就是  terminate  的调用的  入口是哪里？
。。6那里的  英语原文是：this assignment will terminate the program!
。。根据上下文，应该是  把  t1  的  thread  terminate掉。

。。试了下，  第六步  确实会导致  main  线程  崩溃：  abort() has been called。。

。。所以  terminate  是  终止了  main。  但是  怎么保证  和  析构函数一致？  abort  后，  thread_guard  的  析构函数  没有被调用，所以  也没有  join。

。。确实，  std::terminate,  不是  thread::terminate。

class scoped_thread
{
std::thread t;
public:
explicit scoped_thread(std::thread t_):                 // 1
t(std::move(t_))
{
if(!t.joinable())                                                   // 2
throw std::logic_error(“No thread”);
}
~scoped_thread()
{

t.join();                                                                    // 3

}
scoped_thread(scoped_thread const&)=delete;
scoped_thread& operator=(scoped_thread const&)=delete;
};
struct func; // 定义在清单2.1中
void f()
{
int some_local_state;
scoped_thread t(std::thread(func(some_local_state)));       // 4
do_something_in_current_thread();
}                                                             // 5

和  thread_guard  类似，不过新线程直接传递到scoped_thread中④，而非创建一个独立变量
这里把检查放在了构造函数中②，并且当线程不可加入时，抛出异常

C++17标准给出一个建议，就是添加一个joining_thread的类型，这个类型与 std::thread 类似；不同是的添加了析构函数，就类似于scoped_thread。

但委员会没有达成统一。
C++20中依然进行了探讨，名字为  std::jthread

P46，这里有  joining_thread  的代码。太长了。

量产线程，等待它们结束
void do_work(unsigned id);
void f()
{
std::vector<std::thread> threads;
for(unsigned i=0; i < 20; ++i)
{
threads.push_back(std::thread(do_work,i)); // 产生线程
}
std::for_each(threads.begin(),threads.end(),
std::mem_fn(&std::thread::join)); // 对每个线程调用join()
}

将 std::thread 放入 std::vector 是向线程自动化管理迈出的第一步：并非为这些线程创建独立的变量，并且直接加入，而是把它们当做一个组。创建一组线程(数量在运行时确定)，可使得这一步迈的更大，而非像  上面那样创建固定数量的线程。

。。英文版中  是  for(auto& entry : threads) entry.join();  。。。

。。本地测试的时候，发现  work(int, string)  是无法编译成功的，说  只需要一个参数，但是提供了3个，  work(int, int)  是可以的，  所以：  方法的参数  必须是  同一类型？？？

2.4  运行时决定线程数量

std::thread::hardware_concurrency()   返回  CPU  线程数。
。。本地  6核12线程，  这个方法返回  12  。
。。无法获取系统信息时，返回  0  。

Iterator block_start=first;
for(unsigned long i=0; i < (num_threads-1); ++i)
{
Iterator block_end=block_start;
std::advance(block_end,block_size); // 6
threads[i]=std::thread( // 7
accumulate_block<Iterator,T>(),
block_start,block_end,std::ref(results[i]));
block_start=block_end; // #8
}
书上这里一个  并行版的  accumulate的例子，  我只抄了一点点。

2.5  标识线程

线程标识类型为  std::thread::id
可以通过调用 std::thread 对象的成员函数 get_id() 来直接获取
当前线程中调用  std::this_thread::get_id()
。。是一个数字。

如果 std::thread 对象没有与任何执行线程相关联， get_id() 将返回 std::thread::type 默认构造值，这个值表示“无线程”

std::thread::id 类型对象提供相当丰富的对比操作
提供为不同的值进行排序，这意味着允许程序员将其当做为容器的键值
标准库也提供 std::hash<std::thread::id> 容器，所以 std::thread::id 也可以作为无序容器的键值。

std::thread::id 实例常用作检测线程是否需要进行一些操作，比如：当用线程来分割一项工作(如清单2.9)，主线程可能要做一些与其他线程不同的工作。这种情况下，启动其他线程前，它可以将自己的线程ID通过 std::this_thread::get_id() 得到，并进行存储。

。。但是，这种，明显  主线程  独立出来更好啊。

std::thread::id master_thread;
void some_core_part_of_algorithm()
{
if(std::this_thread::get_id()==master_thread)
{
do_master_thread_work();
}
do_common_work();
}

另外，当前线程的 std::thread::id 将存储到一个数据结构中。之后在这个结构体中对当前线程的ID与存储的线程ID做对比，来决定操作是被“允许”，还是“需要”

线程ID在容器中可作为键值
容器可以存储其掌控下每个线程的信息，或在多个线程中互传信息

C++标准的唯一要求就是要保证ID比较结果相等的线程，必须有相同的输出

-----------------------

第3章  线程间共享数据
共享数据带来的问题
使用互斥量保护数据
数据保护的替代方案

3.1  共享数据带来的问题
涉及到共享数据时，问题就可能是因为共享数据修改所导致

不变量(invariants)的概念对开发者们编写的程序会有一定的帮助
不变量通常会在一次更新中被破坏，特别是比较复杂的数据结构，或者一次更新就要改动很大的数据结构。

双链表，删除一个node的时候，需要把  前一个，后一个  node  都进行修改。都修改完后，不变量就又稳定了。
删除的步骤
a. 找到要删除的节点N
b. 更新前一个节点指向N的指针，让这个指针指向N的下一个节点
c. 更新后一个节点指向N的指针，让这个指正指向N的前一个节点
d. 删除节点N

删除过程中，如果有其他线程也在进行删除，可能会导致数据永久损坏。
这都是并行中常见错误：条件竞争(race  condition)

。。感觉，读取，不会有太大的问题。  一般的读取，本身就不太可能是  可重复读，可重复读，必然要  加锁。。

。。当然需要一定的步骤，  不能先将  前一个，后一个  节点  放到  临时变量，然后删除节点N，然后  重建  链接关系。  就是  前一个，后一个节点的  指针  必须是  指向有效，且  不会遗漏后续所有数据的  节点。

3.1.1  条件竞争
电影院，多个售票窗口，要买到票，就需要靠抢，是否能买到，取决于购买的相对顺序。

C++ 标准中也定义了数据竞争这个术语，一种特殊的条件竞争：并发的去修改一个独立
对象(参见5.1.2节)，数据竞争是(可怕的)未定义行为的起因。

条件竞争很难查找，也很难复现

当系统负载增加时，随着执行数量的增加，执行序列的问题复现的概率也在增加，这样的问题只可能会出现在负载比较大的情况下。

条件竞争通常是时间敏感的，所以程序以调试模式运行时，它们常会完全消失，因为调试模式会影响程序的执行时间(即使影响不多)

3.1.2  避免恶性条件竞争
最简单的办法就是对数据结构采用某种保护机制，确保只有进行修改的线程才能看到不变量被破坏时的中间状态。

另一个选择是对数据结构和不变量的设计进行修改，修改完的结构必须能完成一系列不可分割的变化，也就是保证每个不变量保持稳定的状态，这就是所谓的无锁编程。不过，这种方式很难得到正确的结果。如果到这个级别，无论是内存模型上的细微差异，还是线程访问数据的能力，都会让工作量变的很大。

另一种处理条件竞争的方式是，使用事务的方式去处理数据结构的更新(这里的"处理"就如同对数据库进行更新一样)。所需的一些数据和读取都存储在事务日志中，然后将之前的操作合为一步，再进行提交。当数据结构被另一个线程修改后，或处理已经重启的情况下，提交就会无法进行，这称作为“软件事务内存”(software transactional memory (STM))。

3.2  使用互斥量保护共享数据
当访问共享数据前，将数据锁住，在访问结束后，再将数据解锁。

互斥量一种数据保护通用机制，但它不是什么“银弹”；需要编排代码来保护数据的正确性(见3.2.2节)，并避免接口间的竞争条件(见3.2.3节)也非常重要。不过，互斥量自身也有问题，也会造成死锁(见3.2.4节)，或对数据保护的太多(或太少)(见3.2.8节)。

3.2.1 C++中使用互斥量
C++中通过实例化 std::mutex 创建互斥量实例，通过成员函数lock()对互斥量上锁，unlock()进行解锁。

实践中不推荐直接去调用成员函数，调用成员函数就意味着，必须在每个函数出口都要去调用unlock()，也包括异常的情况。

C++标准库为互斥量提供了一个RAII语法的模板类 std::lock_guard ，在构造时就能提供已锁的互斥量，并在析构的时候进行解锁。

std::list<int> some_list; // 1
std::mutex some_mutex; // 2
void add_to_list(int new_value)
{
std::lock_guard<std::mutex> guard(some_mutex); // 3
some_list.push_back(new_value);
}
bool list_contains(int value_to_find)
{
std::lock_guard<std::mutex> guard(some_mutex); // 4
return
std::find(some_list.begin(),some_list.end(),value_to_find) !=
some_list.end();
}

C++17中添加了一个新特性，称为模板类参数推导，这样类似 std::locak_guard 这样简单的模板类型的模板参数列表可以省略
③和④的代码可以简化成：
std::lock_guard guard(some_mutex);

3.2.4节中，会介绍C++17中的一种加强版数据保护机制—— std::scoped_lock ，所以在C++17的环境下，上面的这行代码也可以写成：
std::scoped_lock guard(some_mutex);

大多数情况下，互斥量通常会与需要保护的数据放在同一类中，而不是定义成全局变量

当其中一个成员函数返回的是保护数据的指针或引用时，会破坏数据。具有访问能力的指针或引用可以访问(并可能修改)被保护的数据，而不会被互斥锁限制。这就需要对接口有相当谨慎的设计，要确保互斥量能锁住数据的访问，并且不留后门。

3.2.2  用代码来保护共享数据
并不是仅仅在每一个成员函数中都加入一个 std::lock_guard 对象那么简单；一个指针或引用，也会让这种保护形同虚设

不过，检查指针或引用很容易，只要没有成员函数通过返回值或者输出参数的形式，向其调用者返回指向受保护数据的指针或引用，数据就是安全的。

。。这个翻译了个寂寞啊。
检查成员函数是否通过指针或引用的方式来调用也是很重要的(尤其是这个操作不在你的控制下时)。
it’s also important to check that they don’t pass these pointers or
references in to functions they call that aren’t under your control.
。。需要确保  成员函数  不会将  指针或ref  传递到  不受你控制的函数/lambda中。

template<typename Function>
void process_data(Function func)
{
std::lock_guard<std::mutex> l(m);
func(data); // 1 传递“保护”数据给用户函数
}

。。但是  func  的调用依然在  锁内部吧。。。ok，下面的代码  获得了  数据的指针

some_data* unprotected;   //  不受  mutex  保护
void malicious_function(some_data& protected_data)
{
unprotected=&protected_data;
}

。。大受震撼，  感觉  所有语言  都可以通过  这种方式  将  私有属性  公开出来。
。。但不可能有这种"漏洞"吧。

切勿将受保护数据的指针或引用传递到互斥锁作用域之外，无论是函数返回值，还是存储在外部可见内存，亦或是以参数的形式传递到用户提供的函数中去

3.2.3 定位接口间的条件竞争
。栈非空，则pop。。多线程的时候，就。。。

这是一个经典的条件竞争，使用互斥量对栈内部数据进行保护，但依旧不能阻止条件竞争的发生，这就是接口固有的问题。

当调用top()时，发现栈已经是空的了，那么就抛出异常。能直接解决这个问题，但这是一个笨拙的解决方案
。。应该不行吧，  只要  栈是否空  与  pop  不是原子的，  就不可能  解决这个问题。

。。。top  和  pop  是2个函数，。。只能自己加：  topAndPop() { lock_guard(mtx); t2=top(); pop(); return t2; }  。。  或者  直接让  pop  有返回值。

需要接口设计上有较大的改动，提议之一就是使用同一互斥量来保护top()和pop()

选项1：  传入一个引用
第一个选项是将变量的引用作为参数，传入pop()函数中获取想要的“弹出值”：
std::vector<int> result;
some_stack.pop(result);

缺点很明显：需要构造出一个栈中类型的实例，用于接收目标值。
临时构造一个实例，从时间和资源的角度上来看，都是不划算。
很多用户自定义类型可能都不支持赋值操作

选项2：无异常抛出的拷贝构造函数或移动构造函数
局限性太强。用户自定义的类型中，会有不抛出异常的拷贝构造函数或移动构造函数的类型， 那些有抛出异常的拷贝构造函数，但没有移动构造函数的类型往往更多

选项3：返回指向弹出值的指针
第三个选择是返回一个指向弹出元素的指针，而不是直接返回值。指针的优势是自由拷贝，并且不会产生异常

缺点就是返回一个指针需要对对象的内存分配进行管理，对于简单数据类型(比如：int)，内存管理的开销要远大于直接返回值。对于选择这个方案的接口，使用 std::shared_ptr 是个不错的选择

选项4：“选项1 +  选项2”或  “选项1 +  选项3”

线程安全的stack
std::shared_ptr<T> pop()
{
std::lock_guard<std::mutex> lock(m);
if(data.empty()) throw empty_stack(); // 在调用pop前，检查栈是否为空
std::shared_ptr<T> const res(std::make_shared<T>(data.top())); // 在修改堆栈前，分配出返回值
data.pop();
return res;
}

void pop(T& value)
{
std::lock_guard<std::mutex> lock(m);
if(data.empty()) throw empty_stack();
value=data.top();
data.pop();
}

3.2.4  死锁：问题描述及解决方案

避免死锁的一般建议，就是让两个互斥量总以相同的顺序上锁

事情没那么简单

。swap  交换  2个相同类  的不同实例。每个实例都需要  lock  自己的数据，防止  swap时  被其他线程  修改。  所以  swap  中  先锁第一个参数，然后锁  第二个参数。

。当swap  被调用  以  交换  2个相同的  instance  时，  死锁了。

。。这里的英文原文也没有看懂。

all it takes is for two  threads to try to exchange data between the same two instances with the parameters  swapped, and you have deadlock!

。。哪里来的  2 thread  。。。

很幸运，C++标准库有办法解决这个问题， std::lock ——可以一次性锁住多个(两个以上)的互斥量，并且没有副作用(死锁风险)。

friend void swap(X& lhs, X& rhs)
{
if(&lhs==&rhs)
return;
std::lock(lhs.m,rhs.m); // 1
std::lock_guard<std::mutex> lock_a(lhs.m,std::adopt_lock);  // 2
std::lock_guard<std::mutex> lock_b(rhs.m,std::adopt_lock);  // 3
swap(lhs.some_detail,rhs.some_detail);
}

调用 std::lock() ①锁住两个互斥量
。。锁  mutex。。。不，这里应该是  启用mutex的意思

提供 std::adopt_lock 参数除了表示 std::lock_guard 对象可获取锁之外，还将锁交由 std::lock_guard 对象管理，而不需要 std::lock_guard 对象再去构建新的锁

。。锁的  所有权的  转移。   就是  2个  mutex  已经启用了。

使用 std::lock 去锁lhs.m或rhs.m时，可能会抛出异常；这种情况下，异常会传播到 std::lock 之外。
当 std::lock 成功的获取一个互斥量上的锁，并且当其尝试从另一个互斥量上再获取锁时，就会有异常抛出，第一个锁也会随着异常的产生而自动释放

C++17对这种情况提供了支持， std::scoped_lock<> 一种新的RAII类型模板类型，

与 std::lock_guard<> 的功能等价，这个新类型能接受不定数量的互斥量类型作为模板参数，以及相应的互斥量(数量和类型)作为构造参数。互斥量支持构造即上锁，与 std::lock 的用法相同，其解锁阶段是在析构中进行。

void swap(X& lhs, X& rhs)
{
if(&lhs==&rhs)
return;
std::scoped_lock guard(lhs.m,rhs.m); // 1
swap(lhs.some_detail,rhs.some_detail);
}

这里使用了C++17的另一个特性：自动推导模板参数。1所在行，等价于：
std::scoped_lock<std::mutex,std::mutex> guard(lhs.m,rhs.m);

3.2.5  避免死锁的进阶指导
。2个  thread  各自调用对方的  join。  也会死锁。

为了避免死锁，这里的指导意见为：当机会来临时，不要拱手让人。
。don’t  wait for another thread if there’s a chance it’s waiting for you

。。生活亦是。但岁月无情，大道无情啊。

以下是几个建议：

避免嵌套锁
第一个建议往往是最简单的：一个线程已获得一个锁时，再别去获取第二个。
避免在持有锁时调用用户提供的代码
。。防止被偷家(你不能确定用户是否提供恶意代码)。

避免在持有锁时调用用户提供的代码
第二个建议是次简单的：你在持有锁的情况下，调用用户提供的代码；如果用户代码要获取一个锁，就会违反第一个指导意见，并造成死锁(有时，这是无法避免的)

使用固定顺序获取锁
当硬性条件要求你获取两个或两个以上的锁，并且不能使用 std::lock 单独操作来获取它
们；那么最好在每个线程上，用固定的顺序获取它们(锁)
关键是如何在线程之间，以一定的顺序获取锁
当一个线程删除一个节点，它必须获取三个节点上的互斥锁：将要删除的节点，两个邻接节点(因为他们也会被修改)。
定义遍历的顺序，一个线程必须先锁住A才能获取B的锁，在锁住B之后才能获取C的锁。这将消除死锁发生的可能性，不允许反向遍历的列表上

使用锁的层次结构
虽然，定义锁的顺序是一种特殊情况，但锁的层次的意义在于提供对运行时约定是否被坚持的检查。这个建议需要对你的应用进行分层，并且识别在给定层上所有可上锁的互斥量。

class hierarchical_mutex
{
std::mutex internal_mutex;
unsigned long const hierarchy_value;
unsigned long previous_hierarchy_value;
static thread_local  unsigned long this_thread_hierarchy_value;  // 1

void check_for_hierarchy_violation()
{
if(this_thread_hierarchy_value <= hierarchy_value) // 2
{
throw std::logic_error(“mutex hierarchy violated”);
}
}

void update_hierarchy_value()
{
previous_hierarchy_value=this_thread_hierarchy_value; // 3
this_thread_hierarchy_value=hierarchy_value;
}
public:
explicit hierarchical_mutex(unsigned long value):
hierarchy_value(value),
previous_hierarchy_value(0)
{}
void lock()
{
check_for_hierarchy_violation();
internal_mutex.lock(); // 4
update_hierarchy_value(); // 5
}
void unlock()
{
if(this_thread_hierarchy_value!=hierarchy_value)
throw std::logic_error(“mutex hierarchy violated”); // 9
this_thread_hierarchy_value=previous_hierarchy_value; // 6
internal_mutex.unlock();
}
bool try_lock()
{
check_for_hierarchy_violation();
if(!internal_mutex.try_lock()) // 7
return false;
update_hierarchy_value();
return true;
}
};

thread_local unsigned long  hierarchical_mutex::this_thread_hierarchy_value(ULONG_MAX);  // 8

这里重点是使用了thread_local的值来代表当前线程的层级值

锁的延伸扩展
死锁不仅仅会发生在锁之间；死锁也会发生在任何同步构造中(可能会产生一个等待循环)，因此这方面也需要有指导意见

代码已能规避死锁， std::lock() 和 std::lock_guard 可组成简单的锁，并覆盖大多数情况，但是有时需要更多的灵活性。在这种情况下，可以使用标准库提供的std::unique_lock 模板

3.2.6 std::unique_lock——灵活的锁

std::unqiue_lock 使用更为自由的不变量，这样 std::unique_lock 实例不会总与互斥量的数据类型相关，使用起来要比 std:lock_guard 更加灵活

。。是  unique_lock  不会  持有  mutex。   完全没有  data type  这种字眼啊。机翻都不如。

an std::unique_lock instance doesn’t always own the mutex that it’s  associated with.

class some_big_object;
void swap(some_big_object& lhs,some_big_object& rhs);
class X
{
private:
some_big_object some_detail;
std::mutex m;
public:
X(some_big_object const& sd):some_detail(sd){}
friend void swap(X& lhs, X& rhs)
{
if(&lhs==&rhs)
return;

// std::defer_lock  留下未上锁的互斥量
// 原文：std::defer_lock leaves mutexes unlocked
std::unique_lock<std::mutex> lock_a(lhs.m,std::defer_lock);
std::unique_lock<std::mutex> lock_b(rhs.m,std::defer_lock);

std::lock(lock_a,lock_b); // 2 互斥量在这里上锁

swap(lhs.some_detail,rhs.some_detail);
}
};
。。即不懂C++，也不懂英语，baidu都不会。

。。  随便看看吧。

除非你想将 std::unique_lock 的所有权进行转让，或是对其做一些其他的事情外，你最好使用C++17中提供的 std::scoped_lock

3.2.7  不同域中互斥量所有权的传递

std::unique_lock 是可移动，但不可赋值的类型

允许一个函数去锁住一个互斥量，并且将所有权移到调用者上，所以调用者可以在这个锁保护的范围内执行额外的动作。
。。提供  并发安全版  和  并发不安全版。  不是。  感觉没什么用。  就是暴露了底层的  mutex。  可能做一些  自定义的操作。

std::unique_lock<std::mutex> get_lock()
{
extern std::mutex some_mutex;
std::unique_lock<std::mutex> lk(some_mutex);
prepare_data();
return lk; // 1
}
void process_data()
{
std::unique_lock<std::mutex> lk(get_lock()); // 2
do_something();
}

。automatic variable：  自动变量;  局部变量

lk在函数中被声明为自动变量，它不需要调用 std::move() ，可以直接返回①(编译器负责调用移动构造函数)

std::unique_lock  实例在销毁前释放锁的能力，可以使用unlock()来做这件事

3.2.8  锁的粒度

std::unique_lock 在这种情况下工作正常，调用unlock()时，代码不需要再访问共享数据；而后当再次需要对共享数据进行访问时，就可以再调用lock()了。

void get_and_process_data()
{
std::unique_lock<std::mutex> my_lock(the_mutex);
some_class data_to_process=get_next_data_chunk();
my_lock.unlock(); // 1 不要让锁住的互斥量越过process()函数的调用
result_type result=process(data_to_process);
my_lock.lock(); // 2 为了写入数据，对互斥量再次上锁
write_result(data_to_process,result);
}

当你持有锁的时间没有达到整个操作时间，就会让自己处于条件竞争的状态

3.3  保护共享数据的其他方式

一个极端但常见的情况是，共享数据  只有在  初始化时  需要被保护  以防止  并发访问，初始化后  就不需要  被保护了  (可能是因为  后续都是  只读操作。)

3.3.1  保护共享数据的初始化过程
延迟初始化(Lazy initialization)在单线程代码很常见
std::shared_ptr<some_resource> resource_ptr;
void foo()
{
if(!resource_ptr)
{
resource_ptr.reset(new some_resource); // 1
}
resource_ptr->do_something();
}
转换为  多线程版本时，只有  1  处需要保护。

std::shared_ptr<some_resource> resource_ptr;
std::mutex resource_mutex;
void foo()
{
std::unique_lock<std::mutex> lk(resource_mutex); // 所有线程在此序列化
if(!resource_ptr)
{
resource_ptr.reset(new some_resource); // 只有初始化过程需要保护
}
lk.unlock();
resource_ptr->do_something();
}
上面的代码  会使得  线程  产生不必要的  序列化。  为了确定  source  已经被初始化，每个线程  都必须等待  互斥量。

void undefined_behaviour_with_double_checked_locking()
{
if(!resource_ptr) // 1
{
std::lock_guard<std::mutex> lk(resource_mutex);
if(!resource_ptr) // 2
{
resource_ptr.reset(new some_resource); // 3
}
}
resource_ptr->do_something(); // 4
}
声名狼藉  的  DCL

因为这里有  潜在的  条件竞争：  1  处的读取操作  没有  和  其他线程中的  3  处的  写入  进行  sync，因此  产生了  条件竞争。  这个  条件竞争  不仅包含  指针本身，还会  影响  其指向的对象：即  一个线程  知道  另一个线程  完成了  对  指针的  写入，但是  它可能  看不到  新建的  some_resource  实例。

这个一种典型的  条件竞争  ---  数据竞争。

C++标准库提供了 std::once_flag 和 std::call_once

std::shared_ptr<some_resource> resource_ptr;
std::once_flag resource_flag; // 1
void init_resource()
{
resource_ptr.reset(new some_resource);
}
void foo()
{
std::call_once(resource_flag,init_resource); // 可以完整的进行一次初始化
resource_ptr->do_something();
}

std::call_once()  可仅作为延迟初始化的类型成员

std::mutex 和 std::once_flag 的实例不能拷贝和移动，需要通过显式定义相应的成员函数，对这些类成员进行操作。

还有一种初始化过程中潜存着条件竞争：其中一个局部变量被声明为static类型，这种变量的在声明后就已经完成初始化；对于多线程调用的函数，这就意味着这里有条件竞争——抢着去定义这个变量

在C++11标准中，这些问题都被解决了：初始化及定义完全在一个线程中发生，并且没有其他线程可在初始化完成前对其进行处理，条件竞争终止于初始化阶段，这样比在之后再去处理好的多。在只需要一个全局实例情况下，这里提供一个 std::call_once 的替代方案

class my_class;
my_class& get_my_class_instance()
{
static my_class instance; // 线程安全的初始化过程
return instance;
}

3.3.2  保护不常更新的数据结构

C++17标准库提供了两种非常好的互斥量—— std::shared_mutex 和 std::shared_timed_mutex 。C++14只提供了 std::shared_timed_mutex ，并且在C++11中并未提供任何互斥量类型。

对于更新操作，可以使用 std::lock_guard<std::shared_mutex> 和 std::unique_lock<std::shared_mutex> 上锁，保证更新线程的独占访问

可以使用 std::shared_lock<std::shared_mutex> 获取访问权

3.3.3  嵌套锁
线程已经获得  std::mutex  时(已经锁上)，对其再次上锁，这个操作是错误的，会产生未定义行为。

C++标准库提供了 std::recursive_mutex 类。除了可以对同一线程的单个实例上获取多个锁，其他功能与 std::mutex 相同。互斥量锁住其他线程前，必须释放拥有的所有锁，所以当调用lock()三次后，也必须调用unlock()三次。

std::lock_guard<std::recursive_mutex>
std::unique_lock<std::recursive_mutex>

大多数情况下，当需要嵌套锁时，你需要审视你的代码设计。

第4章  同步并发操作

4.1  等待一个事件或其他条件
当一个线程等待另一个线程完成任务时，它会有很多选择。
第一，它可以持续的检查共享数据标志(用于做保护工作的互斥量)，直到另一线程完成工作时对这个标志进行重设

第二个选择是在等待线程在检查间隙，使用  std::this_thread::sleep_for()  进行周期性的间歇
。unlock -> sleep -> lock

第三个选择(也是优先的选择)是，使用C++标准库提供的工具去等待事件的发生。通过另一线程触发等待事件的机制是最基本的唤醒方式(例如：流水线上存在额外的任务时)，这种机制就称为“条件变量”。

4.1.1  等待条件达成

C++标准库对条件变量有两套实现： std::condition_variable 和 std::condition_variable_any 。这两个实现都包含在 <condition_variable> 头文件的声明中。两者都需要与一个互斥量一起才能工作(互斥量是为了同步)；前者仅限于与 std::mutex 一起工作，而后者可以和任何满足最低标准的互斥量一起工作，从而加上了_any的后缀

std::mutex mut;
std::queue<data_chunk> data_queue; // 1
std::condition_variable data_cond;
void data_preparation_thread()
{
while(more_data_to_prepare())
{
data_chunk const data=prepare_data();
std::lock_guard<std::mutex> lk(mut);
data_queue.push(data); // 2
data_cond.notify_one(); // 3
}
}
void data_processing_thread()
{
while(true)
{
std::unique_lock<std::mutex> lk(mut); // 4
data_cond.wait(lk,[]{return !data_queue.empty();}); // 5
data_chunk data=data_queue.front();
data_queue.pop();
lk.unlock(); // 6
process(data);
if(is_last_chunk(data))
break;
}
}

std::condition_variable  的成员函数wait()，传递一个锁和一个lambda函数表达式(作为等待的条件⑤)。

wait()会去检查这些条件(通过调用所提供的lambda函数)，当条件满足(lambda函数返回true)时返回。如果条件不满足(lambda函数返回false)，wait()函数将解锁互斥量，并且将这个线程(上段提到的处理数据的线程)置于阻塞或等待状态。当准备数据的线程调用notify_one()通知条件变量时，处理数据的线程从睡眠状态中苏醒，重新获取互斥锁，并且再次检查条件是否满足。在条件满足的情况下，从wait()返回并继续持有锁；当条件不满足时，线程将对互量解锁，并且重新开始等待。这就是为什么用  std::unique_lock  而不使用  std::lock_guard  ——等待中的线程必须在等待期间解锁互斥量，并在这之后对互斥量再次上锁，而  std::lock_guard  没有这么灵活。

4.1.2  使用条件变量构建线程安全队列
。构造一个  线程安全的  queue
。代码片段。  参考  std::queue

std::mutex mut;
std::queue<T> data_queue;
std::condition_variable data_cond;

void push(T new_value)
{
std::lock_guard<std::mutex> lk(mut);
data_queue.push(new_value);
data_cond.notify_one();
}

void wait_and_pop(T& value)
{
std::unique_lock<std::mutex> lk(mut);
data_cond.wait(lk,[this]{return !data_queue.empty();});
value=data_queue.front();
data_queue.pop();
}

bool try_pop(T& value)
{
std::lock_guard<std::mutex> lk(mut);
if(data_queue.empty())
return false;
value=data_queue.front();
data_queue.pop();
return true;
}

std::shared_ptr<T> wait_and_pop()
{
std::unique_lock<std::mutex> lk(mut);
data_cond.wait(lk,[this]{return !data_queue.empty();});
std::shared_ptr<T> res(std::make_shared<T>(data_queue.front()));
data_queue.pop();
return res;
}

std::shared_ptr<T> try_pop()
{
std::lock_guard<std::mutex> lk(mut);
if(data_queue.empty())
return std::shared_ptr<T>();
std::shared_ptr<T> res(std::make_shared<T>(data_queue.front()));
data_queue.pop();
return res;
}

4.2  使用期望值等待一次性事件

C++标准库模型将这种一次性事件称为期望值(future)。当线程需要等待特定的一次性事件时，某种程度上来说就需要知道这个事件在未来的期望结果。

C++标准库中，有两种期望值，使用两种类型模板实现，声明在  <future>  头文件中:  唯一期望值(unique futures)(  std::future<>  )和共享期望值(shared futures)( std::shared_future<> )。仿照了 std::unique_ptr 和 std::shared_ptr

std::future  的实例只能与一个指定事件相关联，而  std::shared_future  的实例就能关联多个事件。后者的实现中，所有实例会在同时变为就绪状态，并且他们可以访问与事件相关的任何数据。

与数据无关处，可以使用 std::future<void> 与 std::shared_future<void> 的特化模板

当多个线程需要访问一个独立期望值对象时，必须使用互斥量或类似同步机制对访问进行保护

并行技术规范将这两个模板类在 std::experimental 命名空间中进行了扩展： std::experimental::future<> 和 std::experimental::shared_future<>

你可以启动一个新线程来执行这个计算，但是这就意味着你必须关注如何传回计算的结果，因为std::thread并不提供直接接收返回值的机制。这里就需要std::async函数模板(也是在头文<future>中声明的)了。

std::async  会返回一个  std::future  对象。只需要调用这个对象的get()成员函数；并且会阻塞线程直到期望值状态为就绪为止

#include <future>
#include <iostream>
int find_the_answer_to_ltuae();
void do_other_stuff();
int main()
{
std::future<int>  the_answer=std::async(find_the_answer_to_ltuae);
do_other_stuff();
std::cout<<"The answer is "<<the_answer.get()<<std::endl;
}

与 std::thread 做的方式一样， std::async 允许你通过添加额外的调用参数，向函数传递额外的参数。

auto f1=std::async(&X::foo,&x,42,"hello"); // 调用p->foo(42,"hello")，p是指向x的指针
auto f2=std::async(&X::bar,x,"goodbye"); // 调用tmpx.bar("goodbye")， tmpx是x的拷贝副本

auto f3=std::async(Y(),3.141); // 调用tmpy(3.141)，tmpy通过Y的移动构造函数得到
auto f4=std::async(std::ref(y),2.718); // 调用y(2.718)
X baz(X&);
std::async(baz,std::ref(x)); // 调用baz(x)

auto f5=std::async(move_only()); //  调用tmp()，tmp是通过std::move(move_only())构造得到

可以在函数调用之前向  std::async  传递一个额外参数，这个参数的类型是  std::launch  ，
可以是  std::launch::defered  ，表明函数调用被延迟到wait()或get()函数调用时才执行，
std::launch::async  表明函数必须在其所在的独立线程上执行，
std::launch::deferred | std::launch::async  表明实现可以选择这两种方式的一种。
最后一个选项是默认的

auto f6=std::async(std::launch::async,Y(),1.2); //  在新线程上执行

auto f7=std::async(std::launch::deferred,baz,std::ref(x)); //  在wait()或get()调用时执行

auto f8=std::async(std::launch::deferred | std::launch::async,baz,std::ref(x)); //  实现选择执行方式

auto f9=std::async(baz,std::ref(x));
f7.wait(); //  调用延迟函数

这不是让 std::future 与任务实例相关联的唯一方式；
你也可以将任务包装入 std::packaged_task<> 实例中，
或通过编写代码的方式，使用 std::promise<> 类型模板显示设置值

4.2.2  任务与期望值关联

std::packaged_task<> 对一个函数或可调用对象，绑定一个期望值。当调用 std::packaged_task<> 对象时，它就会调用相关函数或可调用对象，将期望状态置为就

绪，返回值也会被存储。
这可以用在构建线程池的结构单元(可见第9章)，或用于其他任务的管理，

比如：在任务所在线程上运行其他任务，或将它们顺序的运行在一个特殊的后台线程上。当一个粒度较大的操作被分解为独立的子任务时，其中每个子任务都可以包含在一个 std::packaged_task<> 实例中，之后这个实例将传递到任务调度器或线程池中。对任务细节进行抽象，调度器仅处理  std::packaged_task<>  实例，而非处理单独的函数

std::packaged_task<> 的模板参数是一个函数签名，比如void()就是一个没有参数也没有返回值的函数，或int(std::string&, double*)就是有一个非const引用的 std::string 和一个指向double类型的指针，并且返回类型是int。

当构造出一个 std::packaged_task<> 实例时，就必须传入一个函数或可调用对象
类型可以隐式转换

函数签名的返回类型可以用来标识从get_future()返回的 std::future<> 的类型，而函数签名的参数列表，可用来指定packaged_task的函数调用操作符

因为 std::packaged_task 对象是一个可调用对象，所以可以封装在 std::function 对象中，从而作为线程函数传递到 std::thread 对象中，或作为可调用对象传递另一个函数中，或可以直接进行调用。当 std::packaged_task 作为一个函数被调用时，实参将由函数调用操作符传递到底层函数，并且返回值作为异步结果存储在 std::future ，可通过get_future()获取。因此你可以把用 std::packaged_task 打包任务，并在它被传到别处之前的适当时机取回期望值。当需要异步任务的返回值时，你可以等待期望的状态变为“就绪”

void gui_thread() //
{
while(!gui_shutdown_message_received()) // 2
{
get_and_process_gui_message(); // 3
std::packaged_task<void()> task;
{
std::lock_guard<std::mutex> lk(m);
if(tasks.empty()) // 4
continue;
task=std::move(tasks.front()); // 5
tasks.pop_front();
}
task(); // 6
}
}

template<typename Func>
std::future<void> post_task_for_gui_thread(Func f)
{
std::packaged_task<void()> task(f); // 7
std::future<void> res=task.get_future(); // 8
std::lock_guard<std::mutex> lk(m);
tasks.push_back(std::move(task)); // 9
return res; // 10
}

4.2.3  使用(std::)promises

通过少数线程(可能只有一个)处理网络连接，每个线程同时处理多个连接事件，对需要处理大量的网络连接的应用而言是普遍的做法。

一对 std::promise/std::future 会为这种方式提供一个可行的机制；期望值可以阻塞等待线程，同时，提供数据的线程可以使用组合中的承诺值来对相关值进行设置，并将期望值的状态置为“就绪”

通过一个给定的 std::promise 的get_future()成员函数来获取与之相关的 std::future 对象，跟 std::packaged_task 的用法类似。

当承诺值已经设置完毕(使用set_value()成员函数)，对应期望值的状态变为“就绪”，并且可用于检索已存储的值。当在设置值之前销毁 std::promise ，将会存储一个异常

4.2.4  将异常存与期望值中

函数作为 std::async 的一部分时，当调用抛出一个异常时，这个异常就会存储到期望值中，之后期望值的状态被置为“就绪”，之后调用get()会抛出这个已存储异常(注意：标准级别没有指定重新抛出的这个异常是原始的异常对象，还是一个拷贝；不同的编译器和库将会在这方面做出不同的选择)。

将函数打包入 std::packaged_task 任务包中后，到任务被调用时，同样的事情也会发生；打包函数抛出一个异常，这个异常将被存储在期望值中，准备在get()调用时再次抛出。

通过函数的显式调用， std::promise 也能提供同样的功能。当存入的是一个异常而非一个数值时，就需要调用set_exception()成员函数，而非set_value()

std::current_exception()
std::copy_exception(std::logic_error("foo "))

任何情况下，当期望值的状态还不是“就绪”时，调用 std::promise 或 std::packaged_task 的析构函数，将会存储一个与 std::future_errc::broken_promise 错误状态相关的 std::future_error 异常

不过， std::future 也有局限性。很多线程在等待的时候，只有一个线程能获取等待结果。当多个线程需要等待相同的事件的结果，就需要使用 std::shared_future 来替代 std::future 了

4.2.5  多个线程的等待

多线程在没有额外同步的情况下，访问一个独立的 std::future 对象时，就会有数据竞争和未定义的行为

std::future 模型独享同步结果的所有权，并且通过调用get()函数，一次性的获取数据，这就让并发访问变的毫无意义——只有一个线程可以获取结果值，因为在第一次调用get()后，就没有值可以再获取了

std::future 是只移动的，所以其所有权可以在不同的实例中互相传递，但是只有一个实例可以获得特定的同步结果，而 std::shared_future 实例是可拷贝的，所以多个对象可以引用同一关联期望值的结果。

每一个 std::shared_future 的独立对象上，成员函数调用返回的结果还是不同步的，所以为了在多个线程访问一个独立对象时避免数据竞争，必须使用锁来对访问进行保护。

优先使用的办法：为了替代只有一个拷贝对象的情况，可以让每个线程都拥有自己对应的拷贝对象。这样，当每个线程都通过自己拥有的 std::shared_future 对象获取结果，那么多个线程访问共享同步结果就是安全的

std::promise<int> p;
std::future<int> f(p.get_future());
assert(f.valid()); // 1 期望值 f 是合法的
std::shared_future<int> sf(std::move(f));
assert(!f.valid()); // 2 期望值 f 现在是不合法的
assert(sf.valid()); // 3 sf 现在是合法的

std::promise<std::string> p;
std::shared_future<std::string> sf(p.get_future()); // 1 隐式转移所有权

auto sf=p.get_future().share();

4.3  限定等待时间

两种指定超时方式：一种是“时延”，另一种是“绝对时间点”。
处理持续时间的变量以“_for”作为后缀，处理绝对时间的变量以"_until"作为后缀。

4.3.1  时钟

std::chrono::system_clock::now()  是将返回系统时钟的当前时间

时钟节拍被指定为1/x(x在不同硬件上有不同的值)秒，这是由时间周期所决定——一个时钟一秒有25个节拍，因此一个周期为 std::ratio<1, 25> ，当一个时钟的时钟节拍每2.5秒一次，周期就可以表示为 std::ratio<5, 2>

。。5秒2个

当时钟节拍均匀分布(无论是否与周期匹配)，并且不可调整，这种时钟就称为稳定时钟。当is_steady静态数据成员为true时，表明这个时钟就是稳定的

std::chrono::system_clock 是不稳定的，因为时钟是可调的，即是这种是完全自动适应本地账户的调节。这种调节可能造成的是，本次调用now()返回的时间要早于上次调用now()所返回的时间

C++标准库提供一个稳定时钟  std::chrono::steady_clock

4.3.2  时延

std::chrono::duration<>  函数模板能够对时延进行处理

第一个模板参数是一个类型表示(比如，int，long或double)，第二个模板参数是定制部分，表示每一个单元所用秒数。
例如，当几分钟的时间要存在short类型中时，可以写成 std::chrono::duration<short,
std::ratio<60, 1>> ，因为60秒是才是1分钟，所以第二个参数写成 std::ratio<60, 1> 。

当需要将毫秒级计数存在double类型中时，可以写成 td::chrono::duration<double, std::ratio<1,1000>> ，因为1秒等于1000毫秒。

标准库在 std::chrono 命名空间内，为延时变量提供一系列预定义类型：nanoseconds[纳秒] ,microseconds[微秒] , milliseconds[毫秒] , seconds[秒] , minutes[分]和hours[时]。

C++14中 std::chrono_literals 命名空间中，有许多预定义的后缀操作符用来表示时长。
using namespace std::chrono_literals;
auto one_day=24h;
auto half_an_hour=30min;
auto max_time_between_messages=30ms;

15ns  和 std::chrono::nanoseconds(15) 就是等价的

不过，当使用浮点字面量时，且未指明表示类型时，数值上会对浮点时长进行适当的缩放

当不要求截断值的情况下(时转换成秒是没问题，但是秒转换成时就不行)时延的转换是隐式的。显示转换可以由 std::chrono::duration_cast<> 来完成。

std::chrono::milliseconds ms(54802);

std::chrono::seconds s  =  std::chrono::duration_cast<std::chrono::seconds>(ms);

这里的结果就是截断的，而不是进行了舍入，所以s最后的值为54。

延迟支持四则运算

std::future<int> f=std::async(some_task);
if(f.wait_for(std::chrono::milliseconds(35))==std::future_status::ready)
do_something_with(f.get());

函数等待超时时，会返回 std::future_status::timeout ；
当期望值状态改变，函数会返回 std::future_status::ready ；
当与期望值相关的任务延迟了，函数会返回 std::future_status::deferred

4.3.3  时间点

时间点可以用 std::chrono::time_point<> 类型模板来表示，实例的第一个参数用来指定所要使用的时钟，第二个函数参数用来表示时间的计量单位

。。

4.3.4  具有超时功能的函数

std::this_thread::sleep_for()
std::this_thread::sleep_until()

std::mutex 和 std::recursive_mutex 都不支持超时锁，

但是 std::timed_mutex 和 std::recursive_timed_mutex 支持。这两种类型也有try_lock_for()和try_lock_until()成员函数

![std::this_thread  std::condition variable  std::condition_variable_any  std::timed mutex  std::recursive timed mutex  sleep_for(duration)  sleep_until(time_point)  wait_for(lock, duration)  wait_until(lock,  time _ point)  wait_for(lock, duration,  predicate)  wait_until(lock, duration,  predicate)  try_lock_for(duration)  try_lock_until(time_point)  unique_lock(lockable,  duration)  std::unique_lock<TimedLockable>  unique_lock(lockable,  time _ point)  try_lock_for(duration)  try_lock_until(time_point)  std::future<ValueType>  std::shared_future<ValueType>  wait_for(duration)  wait_until(time_point)  std::cv status::time out  std::cv status::no timet  bool  ik4k4i  bool  true '  N/A —  iBI owns_lock();  bool  std::future  std::future  std::future](../_resources/4be6654fe24341488f16e2ab6969e954.png)

4.4  使用同步操作简化代码

同步工具的使用在本章称为构建块

4.4.1  使用期望值的函数化编程

函数化编程(functional programming)引用于一种编程方式，这种方式中的函数结果只依赖于传入函数的参数，并不依赖外部状态。

C++11支持Lambda表达式(详见附录A，A.6节)，还加入了Boost和TR1中的 std::bind ，以及自动可以自行推断类型的自动变量(详见附录A，A.7节)。期望值作为拼图的最后一块，它使得函数化编程模式并发化(FP-style concurrency)在C++中成为可能

template<typename T>
std::list<T> sequential_quick_sort(std::list<T> input)
{
if(input.empty())
{
return input;
}
std::list<T> result;
result.splice(result.begin(),input,input.begin()); // 1
T const& pivot=*result.begin(); // 2

auto divide_point=std::partition(input.begin(),input.end(),  [&](T const& t){return t<pivot;}); // 3

std::list<T> lower_part;
lower_part.splice(lower_part.end(),input,input.begin(),  divide_point); // 4

auto new_lower(sequential_quick_sort(std::move(lower_part))); //5

auto new_higher(sequential_quick_sort(std::move(input))); // 6

result.splice(result.end(),new_higher); // 7

result.splice(result.begin(),new_lower); // 8

return result;
}

std::future<std::list<T>> new_lower(std::async(&parallel_quick_sort<T>,std::move(lower_part)));

。。替换  new_lower，  就是  并发版。

元素很多时，多次递归后，会产生很多  线程。  但是  是async，所以  运行库  会  自动  裁剪线程数量  (  使用  std::launch::deferred，而不是  std::launch::async)。  具体看  运行库的  实现文档。

下面本身并没有太多优势(事实上会造成大规模的超额任务)，但它可为转型成一个更
复杂的实现铺平道路，实现将会向队列添加任务，而后使用线程池的方式来运行它们

template<typename F,typename A>
std::future<std::result_of<F(A&&)>::type>
spawn_task(F&& f,A&& a)
{
typedef std::result_of<F(A&&)>::type result_type;
std::packaged_task<result_type(A&&)>  task(std::move(f)));
std::future<result_type> res(task.get_future());
std::thread t(std::move(task),std::move(a));
t.detach();
return res;
}

函数化编程可算作是并发编程的范型，并且也是通讯顺序进程(CSP，Communicating Sequential Processer[3])的范型
这里的线程理论上是完全分开的，也就是没有共享数据

但  有通讯通道允许信息在不同线程间进行传递。这种范型被Erlang语言所采纳，并且在MPI(Message Passing Interface，消息传递接口)上常用来做C和C++的高性能运算

4.4.2  使用消息传递的同步操作
CSP的概念十分简单：当没有共享数据时，每个线程可以进行独立思考，其行为纯粹基于所接收到的信息。每个线程就都有自己的状态机

真正通讯顺序处理没有共享数据，所有消息都是通过消息队列传递，但C++线程共享一块地址空间，所以达不到真正通讯顺序处理的要求。这就需要一些约定来支持：作为应用或是库作者，我们有责任确保在实现中，线程不存在共享数据。当然，为了线程间的通信，消息队列必须共享，具体的细节要包含在库中。

。通讯顺序处理  -> communicating sequential process, CSP

。这里有个  ATM的状态机的  例子，但是  messaging  这个命名空间  是谁的？
messaging::receiver incoming;
messaging::sender bank;
messaging::sender interface_hardware;

incoming.wait()
.handle<digit_pressed>( // 1
[&](digit_pressed const& msg)
{
unsigned const pin_length=4;
pin+=msg.digit;
if(pin.length()==pin_length)
{
bank.send(verify_pin(account,pin,incoming));
state=&atm::verifying_pin;
}
}
)
.handle<clear_last_pressed>( // 2
[&](clear_last_pressed const& msg)
{
if(!pin.empty())
{
pin.resize(pin.length()-1);
}
}
)
.handle<cancel_pressed>( // 3
[&](cancel_pressed const& msg)
{
state=&atm::done_processing;
});

。。设计模式  ->  责任链

4.4.3  并发技术扩展规范中的持续性并发

std::experimental::future
std::experimental::promise

std::experimental::future<int> find_the_answer;
auto fut=find_the_answer();
auto fut2=fut.then(find_the_question);
assert(!fut.valid());
assert(fut2.valid());

。看了  cppreference，  future  还是  C++11  的。里面没有  then  函数。  所以。。
。搜到了  ：  https://en.cppreference.com/w/cpp/experimental/future
。还是  experimental。

4.4.4  持续性连接

将整个任务分成了一系列任务，并且每个任务在完成的时候回连接到前一个任务上
。代码使用了  std::experimental::future

std::experimental::future<void> process_login(
std::string const& username, std::string const& password)
{
return backend.async_authenticate_user(username,  password).then(
[](std::experimental::future<user_id> id){       //  (auto id)
return backend.async_request_current_info(id.get());
}).then([](std::experimental::future<user_data>  info_to_display){
try{
update_display(info_to_display.get());
} catch(std::exception& e){
display_error(e);
}
});
}

auto fut = spawn_async(some_function).share();
auto fut2 = fut.then([](std::experimental::shared_future<some_data> data){
do_stuff(data);
});
auto fut3 = fut.then([](std::experimental::shared_future<some_data> data){
return do_other_stuff(data);
});

4.4.5  等待多个期望值

std::experimental::when_all

第四章剩下的都是  介绍  experimental  的东西。。省略了。

4.4.6  使用when_any等待第一个期望值
std::experimental::when_any

4.4.7  并发技术扩展规范中的锁存器和栅栏机制
。。Java：？

4.4.8 std::experimental::latch：基础的锁存器类型
std::experimental::latch  声明在  <experimental/latch>  头文件中

构造 std::experimental::latch 时，将计数器的值作为构造函数的唯一参数。
之后，当等待的事件发生，就会调用锁存器count_down成员函数；
当计数器为0时，锁存器状态变为就绪。
可以调用wait成员函数对锁存器进行阻塞，直到等待的锁存器处于就绪状态时释放；
如果需要对锁存器是否就绪的状态进行检查时，可调用is_ready成员函数。
想要减少计数器1并阻塞直至它抵达0，则可以调用count_down_and_wait成员函数

4.4.9  std::experimental::barrier：简单的栅栏机制

提供了两种栅栏机制， <experimental/barrier> 头文件中分别为： std::experimental::barrier 和 std::experimental::flex_barrier 。前者更简单，所以开销更低；后者更灵活，但是开销较大

所有线程都在处理下一个数据项前，都必须完成当前的任务。
std::experimental::barrier 正是针对这样的情况而设计

这里可以为同步组，指定线程的数量，并为这组线程构造栅栏

当每个线程完成其处理任务时，都会到达栅栏处，并且通过调用栅栏对象的arrive_and_wait成员函数，等待小组的其他成员线程。
当最后一个线程抵达时，所有线程将被释放，并且栅栏会被重置

可以通过显式调用栅栏对象的arrive_and_drop成员函数让线程退出组
这样线程就不用再受栅栏的约束，这样下一个周期到达的线程数就必须要比当前周期到达的线程数少一个了

4.4.10 std::experimental::flex_barrier—更灵活和友好版std::experimental::barrier

std::experimental::flex_barrier 与 std::experimental::barrier 有一点不同：
其有一个额外的构造函数，需要传递传入一个完整的函数和线程数量。
当所有线程都到达栅栏处，那么这个函数就由其中一个线程运行。
其不仅指定了一种串行代码块的运行方式，并且还提供了一种修改需要在下一个周期到达栅栏处线程个数的方式。
对于线程的计数可以修改成任何数字，无论这个数字比当前数字高或低；因为这个功能，开发者就能保证下一次到达栅栏处的线程数量时正确无误的。

第5章  C++内存模型和原子类型操作
一个十分重要特性：多线程(感知)内存模型

5.1  内存模型基础
内存模型：一方面是基本结构，这与内存布局的有关，另一方面就是并发

5.1.1  对象和内存位置
四个需要牢记的原则：
1. 每一个变量都是一个对象，包括作为其成员变量的对象。
2. 每个对象至少占有一个内存位置。
3. 基本类型都有确定的内存位置(无论类型大小如何，即使他们是相邻的，或是数组的一部
分)。
4. 相邻位域是相同内存中的一部分。

5.1.2  对象、内存位置和并发

所有东西都在内存中。当两个线程访问不同的内存位置时，不会存在任何问题，一切都工作顺利。当两个线程访问同一个内存位置，就要小心了。如果没有线程更新数据，那还好；只读数据不需要保护或同步。当有线程对内存位置上的数据进行修改，那就有可能会产生条件竞争

为了避免条件竞争，两个线程就需要一定的执行顺序。
第一种方式，如第3章所述，使用互斥量来确定访问的顺序
另一种是使用原子操作(详见5.2节中对于原子操作的定义)

未定义的行为是C++中最黑暗的角落

5.1.3  修改顺序

5.2 C++中的原子操作和原子类型

原子操作是个不可分割的操作。
系统的所有线程中，不可能观察到原子操作完成了一半；
要么就是做了，要么就是没做，只有这两种可能。

如果读取对象的加载操作是原子的，那么这个对象的所有修改操作也是原子的
。。？读取的时候加锁。写入的时候不加锁，能读取中间态。
。。原子，估计是说  不加锁  也是原子？

5.2.1  标准原子类型
头文件  <atomic>
这些类型的所有操作都是原子的

实际上，标准原子类型的实现就可能是这样模拟出来的：它们(几乎)都有一个 is_lock_free() 成员函数，这个函数可以让用户查询某原子类型的操作是直接用的原子指令( x.is_lock_free() 返回 true )，还是内部用了一个锁结构( x.is_lock_free() 返回 false )。

原子操作的关键就是使用一种同步操作方式，来替换使用互斥量的同步方式；如果操作内部使用互斥量实现，那么期望达到的性能提升就是不可能的事情

标准库提供了一组宏，在编译时对各种整型原子操作是否无锁进行判别

C++17中，所有原子类型有一个static constexpr成员变量，如果相应硬件上的原子类型X是无锁类型，那么X::is_always_lock_free将返回true

ATOMIC_BOOL_LOCK_FREE ,
ATOMIC_CHAR_LOCK_FREE ,
ATOMIC_CHAR16_T_LOCK_FREE ,
ATOMIC_CHAR32_T_LOCK_FREE ，
ATOMIC_WCHAR_T_LOCK_FREE，
ATOMIC_SHORT_LOCK_FREE ,
ATOMIC_INT_LOCK_FREE ,
ATOMIC_LONG_LOCK_FREE ,
ATOMIC_LLONG_LOCK_FREE
ATOMIC_POINTER_LOCK_FREE

如果原子类型不是无锁结构，那么值为0；如果原子类型是无锁结构，那么值为2；如果原子类型的无锁状态在运行时才能确定，那么值为1。

只有 std::atomic_flag 类型不提供is_lock_free()。该类型是一个简单的布尔标志，并且在这种类型上的操作都是无锁的

这些备选类型名可能参考相应的 std::atomic<> 特化类型，或是特化类型。同一程序中混合使用备选名与 std::atomic<> 特化类名，会使代码的移植性大打折扣

标准原子类型的备选名和与其相关的  std::atomic<>  特化类

|     |     |
| --- | --- |
| 原子类型 | 相关特化类 |
| atomic_bool | std::atomic<bool> |
| atomic_char | std::atomic<char> |
| atomic_schar | std::atomic<signed char> |
| atomic_uchar | std::atomic<unsigned char> |
| atomic_int | std::atomic<int> |
| atomic_uint | std::atomic<unsigned> |
| atomic_short | std::atomic<short> |
| atomic_ushort | std::atomic<unsigned short> |
| atomic_long | std::atomic<long> |
| atomic_ulong | std::atomic<unsigned long> |
| atomic_llong | std::atomic<long long> |
| atomic_ullong | std::atomic<unsigned long long> |
| atomic_char16_t | std::atomic<char16_t> |
| atomic_char32_t | std::atomic<char32_t> |
| atomic_wchar_t | std::atomic<wchar_t> |

C++标准库不仅提供基本原子类型，还定义了与原子类型对应的非原子类型，就如同标准库中的 std::size_t

标准原子类型定义(typedefs)和对应的内置类型定义(typedefs)

|     |     |
| --- | --- |
| 原子类型定义 | 标准库中相关类型定义 |
| atomic_int_least8_t | int_least8_t |
| atomic_uint_least8_t | uint_least8_t |
| atomic_int_least16_t | int_least16_t |
| atomic_uint_least16_t | uint_least16_t |
| atomic_int_least32_t | int_least32_t |
| atomic_uint_least32_t | uint_least32_t |
| atomic_int_least64_t | int_least64_t |
| atomic_uint_least64_t | uint_least64_t |
| atomic_int_fast8_t | int_fast8_t |
| atomic_uint_fast8_t | uint_fast8_t |
| atomic_int_fast16_t | int_fast16_t |
| atomic_uint_fast16_t | uint_fast16_t |
| atomic_int_fast32_t | int_fast32_t |
| atomic_uint_fast32_t | uint_fast32_t |
| atomic_int_fast64_t | int_fast64_t |
| atomic_uint_fast64_t | uint_fast64_t |
| atomic_intptr_t | intptr_t |
| atomic_uintptr_t | uintptr_t |
| atomic_size_t | size_t |
| atomic_ptrdiff_t | ptrdiff_t |
| atomic_intmax_t | intmax_t |
| atomic_uintmax_t | uintmax_t |

相关的原子类型就在原来的类型名前加上atomic_的前缀

通常，标准原子类型是不能进行拷贝和赋值的，它们没有拷贝构造函数和拷贝赋值操作符。但是，可以隐式转化成对应的内置类型，所以这些类型依旧支持赋值，可以使用load()和store()、exchange()、compare_exchange_weak()和compare_exchange_strong()。

它们都支持复合赋值符：+=, -=, *=, |= 等等。并且使用整型和指针的特化类型还支持++和--操作。当然，这些操作也有功能相同的成员函数所对应：fetch_add(), fetch_or()等等。

每种函数类型的操作都有一个内存排序参数，这个参数可以用来指定存储的顺序
5.3节中，会对存储顺序选项进行详述

5.2.2 std::atomic_flag的相关操作

std::atomic_flag  是最简单的原子类型，它表示了一个布尔标志
在两个状态间切换：设置和清除

std::atomic_flag  类型的对象必须被ATOMIC_FLAG_INIT。
初始化标志位是“清除”状态

是唯一需要以如此特殊的方式初始化的原子类型，但也是唯一保证无锁的类型。

。。。。也是啊，  bool，就使用了  一个  bit，   一个bit  就  2种状态，  不可能有中间态。  所以  必然无锁。

如果 std::atomic_flag 是静态存储的，那么就的保证其是静态初始化的，也就意味着没有初始化顺序问题

当标志对象已初始化，那么只能做三件事情：销毁，清除或设置(查询之前的值)。
这些操作对应的函数分别是：clear()成员函数和test_and_set()成员函数。
clear()和test_and_set()成员函数可以指定好内存顺序。
clear()是一个存储操作，所以不能有memory_order_acquire或memory_order_acq_rel语义，
但是test_and_set()是一个“读-改-写”操作，可以应用于任何内存顺序。
每一个原子操作，默认的内存序都是memory_order_seq_cst。

一个原子类型的所有操作都是原子的，因赋值和拷贝调用了两个对象，这就就破坏了操作的原子性。

有限的特性使得 std::atomic_flag 非常适合于作自旋互斥锁。初始化标志是“清除”，并且互斥量处于解锁状态。为了锁上互斥量，循环运行test_and_set()直到旧值为false，就意味着这个线程已经被设置为true了。解锁互斥量是一件很简单的事情，将标志清除即可。

class spinlock_mutex
{
std::atomic_flag flag;
public:
spinlock_mutex():
flag(ATOMIC_FLAG_INIT)
{}
void lock()
{
while(flag.test_and_set(std::memory_order_acquire));
}
void unlock()
{
flag.clear(std::memory_order_release);
}
};
。。这个不能用。。看ch5的代码

这样的互斥量是最基本的，但它已经足够  std::lock_guard<>  使用了

std::atomic_flag 局限性太强，没有非修改查询操作，甚至不能像普通的布尔标志那样使用。所以，实际中最好使用 std::atomic<bool>

5.2.3 std::atomic<bool>  的相关操作

不能拷贝构造和拷贝赋值，
但可以使用非原子的bool类型进行构造，所以可以被初始化为true或false，并且可以从非原子bool变量赋值给 std::atomic<bool>

std::atomic<bool> b(true);
b=false;

store()是一个存储操作，
而load()是一个加载操作。
exchange()是一个“读-改-写”操作

std::atomic<bool> b;
bool x=b.load(std::memory_order_acquire);
b.store(true);
x=b.exchange(false, std::memory_order_acq_rel);

compare_exchange_weak()
compare_exchange_strong()
。CAS

“比较/交换”操作是原子类型编程的基石

操作成功时返回true，失败时返回false

compare_exchange_weak()函数，当原始值与预期值一致时，存储也可能会不成功。；在这个例子中变量的值不会发生改变，并且compare_exchange_weak()的返回是false

因为compare_exchange_weak()可以伪失败，所以通常使用一个循环
bool expected=false;
extern atomic<bool> b; // 设置些什么
while(!b.compare_exchange_weak(expected,true) && !expected);

。。compare_exchange_weak/strong，  第一个参数是  ref，  第二个是value，  所以第一个参数  必须是  变量。   不能直接  false

。。不算CAS。

compare_exchange_weak/strong，  如果  this的值  和  excepted  不想等，那么  会把  this  的值  赋给  excepted (所以  是个  ref)  。。。  但是  配合上  while，  下次就必然进去了。  无法起到  乐观锁的  作用啊。

。所以就不能搭配  while，  而是  while (true) { exp = true; compare_exchange_() }
。。注意上面的   红色部分。  这样  excepted  必须是  false  的时候  才会  通过。

如果想要改变变量值，更新后的期望值将会变更有用；经历每次循环的时候，期望值都会重新加载，所以当没有其他线程同时修改期望时，循环中对compare_exchange_weak()或compare_exchange_strong()的调用都会在下一次(第二次)成功。

如果值很容易存储，那么使用compare_exchange_weak()能更好的避免一个双重循环的执行，即使compare_exchange_weak()可能会“伪失败”(因此compare_exchange_strong()包含一个循环)。

另一方面，如果值的存储本身是耗时的，那么当期望值不变时，使用compare_exchange_strong()可以避免对值的重复计算。对于 std::atomic<bool> 这些都不重要——毕竟只可能有两种值——但是对于其他的原子类型影响就比较大了。

。。所以  strong是  CAS？  不，  strong  还是会  修改  exp变量。

https://en.cppreference.com/w/cpp/atomic/atomic/is_lock_free

The C++ standard recommends (but does not require) that lock-free atomic operations are also address-free, that is, suitable for communication between processes using shared memory.

可以使用is_lock_free()成员函数，检查 std::atomic<bool> 上的操作是否无锁。
除了 std::atomic_flag 之外，所有原子类型都拥有的特征(is_lock_free)。
。。visual studio c++17/20 , atomic<bool>  是无锁的

。。换一种想法，这个  compare_exchange_  修改  expected  也有一点点的道理。
。。：这个值  增加  10，  ：  compare_exchange_(original, original+10)。
一次不行，就2次。

。。CAS  感觉  更多的是  想  作为  锁  / 乐观锁  。
。。就是看  original  的值  是否重要。

5.2.4 std::atomic<T*> :指针运算

std::atomic<T*> 也有

load(), store(), exchange(), compare_exchange_weak()和compare_exchage_strong()成员函数，

与 std::atomic<bool> 的语义相同，获取与返回的类型都是T*，而不是bool。

std::atomic<T*> 为指针运算提供新的操作。基本操作有fetch_add()和fetch_sub()提供，它们在存储地址上做原子加法和减法，为+=, -=, ++和--提供简易的封装

例如：如果x是 std::atomic<Foo*> 类型的数组的首地址，然后x+=3让其偏移到第四个元素的地址，并且返回一个普通的 Foo* 类型值，这个指针值是指向数组中第四个元素

fetch_add()和fetch_sub()的返回值略有不同(所以x.ftech_add(3)让x指向第四个元素，并且函数返回指向第一个元素的地址)
正像其他操作那样，返回值是一个普通的 T* 值，而非是 std::atomic<T*> 对象的引用

class Foo {};
void test_atomic_pointer()
{
Foo arr[5];
std::atomic<Foo*> p(arr);                                // 6
Foo* x = p.fetch_add(2);                // 返回老值
assert(x == arr);
assert(p.load() == &arr[2]);
x = (p -= 1);                // 返回新值
assert(x == &arr[1]);
assert(p.load() == &arr[1]);
cout << "ok" << endl;
}

函数也允许内存序语义作为给定函数的参数：
p.fetch_add(3,std::memory_order_release);

5.2.5  标准的原子整型的相关操作

如同普通的操作集合一样

(load(), store(), exchange(), compare_exchange_weak(), 和compare_exchange_strong())，

在 std::atomic<int> 和 std::atomic<unsigned long long> 也是有一套完整的操作可以供使用：

fetch_add(), fetch_sub(), fetch_and(), fetch_or(),fetch_xor()，还有复合赋值方式((+=, -=, &=, |=和^=)，以及++和--(++x, x++, --x和x--)。

对于普通的整型来说，这些复合赋值方式还不完全，但也接近完整了：只有除法、乘法和移位操作不在其中。因为，整型原子值通常用来作计数器，或者是掩码，所以以上操作的缺失显得不是那么重要

对于  std::atomic<T*>  类型紧密相关的两个函数就是fetch_add()和fetch_sub()
。。没太搞懂，  感觉  这个  T*  是为了数组而存在的。   不是  指针  的意思。

5.2.6 std::atomic<>  类模板

为了使用 std::atomic<UDT> (UDT是用户定义类型)，这个类型必须有拷贝赋值运算符。这就意味着这个类型不能有任何虚函数或虚基类，以及必须使用编译器创建的拷贝赋值操作。不仅仅是这些，自定义类型中所有的基类和非静态数据成员也都需要支持拷贝赋值操作。这(基本上)就允许编译器使用memcpy()或赋值操作的等价操作，因为实现中没有用户代码。

。。memcpy，，在  compare_exchange_weak  的  cppreference  中，看到过，  是通过  memcmp  来比较  excepted  和  this  是否相同。

通常情况下，编译器不会为 std::atomic<UDT> 类型生成无锁代码，所以所有操作使用一个内部锁

虽然使用 std::atomic<float> 或 std::atomic<double> (内置浮点类型满足使用memcpy和memcmp的标准)，但是在compare_exchange_strong函数中的表现可能会令人惊讶。当存储的值与当前值相等时，这个操作也可能失败，可能因为旧值是一个不同的表达。

如果你使用 std::atomic<> 特化一个用户自定义类型，且这个类型定义了比较操作，而这个比较操作与memcmp又有不同——操作可能会失败，因为两个相等的值拥有不同的表达方式。

。。。。。。

如果UDT类型的大小如同(或小于)一个int或 void* 类型时，大多数平台将会对 std::atomic<UDT> 使用原子指令。有些平台可能会对用户自定义类型(两倍于int或 void* 的大小)特化的 std::atmic<> 使用原子指令。这些平台通常支持所谓的“双字节比较和交换”(double-word-compare-and-swap，DWCAS)指令

以上的限制也意味着有些事情不能做，比如：创建一个 std::atomic<std::vector<int>> 类型。不能使用包含有计数器，标志指针和简单数组的类型，作为特化类型。

越是复杂的数据结构，就有越多的操作，而非只有赋值和比较。如果这种情况发生了，最好使用 std::mutex 保证数据能被保护

表5.3  每一个原子类型所能使用的操作

![a tomic  Operation  test and set  clear  is lock free  load  store  exchange  compare_exchange  weak , compare  exchange strong  fetch add,  fetch sub,  fetch or,  fetch and,  fetch xor,  atomic  flag  atomic  <bool >  <integral  ther -](../_resources/f4e1b31c70fb49e99ba450c7efc3e83e.png)

5.2.7  原子操作的释放函数
不同的原子类型中也有等价的非成员函数存在

大多数非成员函数的命名与对应成员函数有关，但是需要“ atomic_ ”作为前缀(比如， std::atomic_load() )

这些函数都会被不同的原子类型所重载，指定一个内存列标签时会分成两种：一种没有标签，另一种将“ _explicit ”作为后缀，并且需要一个额外的参数，或将内存序作为标签，亦或只有标签

例如， std::atomic_store(&atomic_var,new_value) 与  std::atomic_store_explicit(&atomic_var,new_value,std::memory_order_release

释放函数的设计是为了与C语言兼容，C语言中只能使用指针，没有引用

compare_exchange_weak()和compare_exchange_strong()成员函数的第一个参数(期望值)是一个引用，而 std::atomic_compare_exchange_weak() (第一个参数是指向对象的指针)第二个参数是一个指针

C++标准库也对在一个原子类型中的  std::shared_ptr<>  智能指针类型提供释放函数
std::shared_ptr<my_data> p;
void process_global_data()
{
std::shared_ptr<my_data> local=std::atomic_load(&p);
process_data(local);
}
void update_global_data()
{
std::shared_ptr<my_data> local(new my_data);
std::atomic_store(&p,local);
}

std::experimental::atomic_shared_ptr<T>

5.3  同步操作和强制排序

假设两个线程，一个向数据结构中填充数据，另一个读取数据结构中的数据。为了避免恶性条件竞争，第一个线程设置一个标志，用来表明数据已经准备就绪，并且第二个线程在这个标志设置前不能读取数据。下面的程序清单就是这样的情况

。。就是一个  互斥量  ，  读写锁啊。

5.3.1  同步发生  ( the synchronizes-with relationship )
“同步发生”只能在原子类型之间进行

“同步发生”的基本想法：原子写操作W对变量x进行标记，同步与对x进行原子读操作，读取的是W操作写入的内容；或是W之后，同一线程上的原子写操作对x写入的值；亦或是任意线程对x的一系列原子读-改-写操作(例如，fetch_add()或compare_exchange_weak())。这里，第一个线程所读取到的值为W操作写入(详见5.3.4节)。

5.3.2  先行发生  the happens-before relationship

。。这个得看其他更专业的文章。

“先行发生”关系是一个程序中，基本构建块的操作顺序：它指定了某个操作去影响另一个操作。
如果操作在同时发生，因为操作间无序执行，通常情况下它们就没有先行关系了。这就是另一种排序未被指定的情况。

5.3.3  原子操作的内存顺序

memory_order_relaxed,
memory_order_consume,
memory_order_acquire,
memory_order_release,
memory_order_acq_rel,
memory_order_seq_cst

内存序列选项对于所有原子类型默认都是memory_order_seq_cst

虽然有六个选项，但是它们仅代表三种内存模型：
排序一致序列(sequentially consistent)，
获取-释放序列(memory_order_consume, memory_order_acquire, memory_order_release和
memory_order_acq_rel)和
松散序列(memory_order_relaxed)。

不同的内存序列模型在不同的CPU架构下，功耗是不一样的

基于处理器架构的可视化精细操作的系统，比起其他系统，添加的同步指令可被排序一致序列使用(获取-释放序列和松散序列之前)，或被获取-释放序列调用(在松散序列之前)。

如果有多个处理器，这些额外添加的同步指令可能会消耗大量的时间，从而降低系统整体的性能。

另一方面，CPU使用的是x86或x86-64架构(例如，使用Intel或AMD处理器的台式电脑)，这种架构的CPU不需要任何对获取-释放序列添加额外的指令(没有保证原子性的必要了)，并且，即使是排序一致序列，对于加载操作也不需要任何特殊的处理，不过在进行存储时，需要额外的消耗。

不同种类的内存序列模型，允许专家利用其提升与更细粒度排序相关操作的性能。当默认使用排序一致序列(相较于其他序列，它是最简单的)时，对于在那些不大重要的情况下是有利的。

排序一致队列  SEQUENTIALLY CONSISTENT ORDERING
默认序列命名为排序一致，因为程序中的行为从任意角度去看，序列顺序都保持一致。
如果原子类型实例上的所有操作都是序列一致的，那么一个多线程程序的行为，就会以某种特殊的排序执行，如单线程那样。

意味着，所有操作都不能重排；如果代码在一个线程中，将一个操作放在另一个操作前面，那么这个顺序就必须让其他线程有所了解

对于使用排序一致的原子操作，都会在对值进行存储后进行加载

简单是要付出代价的。在一个多核机器上，会加强对性能的惩罚，因为整个序列中的操作都必须在多个处理器上保持一致，可能需要对处理器间的同步操作进行扩展(代价很昂贵！)。即便如此，一些处理器架构(比如通用x86和x86-64架构)就提供了相对廉价的序列一致，所以需要考虑使用序列一致对性能的影响，就需要去查阅你目标处理器的架构文档，进行更多的了解。

std::thread a(write_x);
std::thread b(write_y);
std::thread c(read_x_then_y);
std::thread d(read_y_then_x);
a.join();
b.join();
c.join();
d.join();
。。所有线程都  一致，  是否是指：  线程  会按照  a-b-c-d  的顺序  串行执行？

非排序一致内存模型
当踏出序列一致的世界，事情就开始变的复杂。可能最需要处理的问题就是：
再也不会有全局的序列了。这就意味着不同线程看到相同操作，不一定有着相同的顺序，还有对于不同线程的操作，都会一个接着另一个执行的想法不在可行。

在没有明确的顺序限制下，唯一的要求就是：所有线程都要统一对每一个独立变量的修改顺序

松散序列
原子类型上的操作以松散序列执行，没有任何同步关系。同一线程中对于同一变量的操作还是服从先发执行的关系，但是不同线程几乎不需要相对的顺序。

std::thread a(write_x_then_y);
std::thread b(read_y_then_x);
a.join();
b.join();

因为加载x的操作④可能读取到false，即使加载y的操作③读取到true，并且存储x的操作①先发与存储y的操作②。x和y是两个不同的变量，所以这里没有顺序去保证每个操作产生相关值的可见性。

非限制操作对于不同变量可以自由重排序，只要它们服从任意的先发执行关系即可(比如，在同一线程中)，它们不会引入同步相关的顺序

要想获取额外的同步，且不使用全局排序一致，可以使用获取-释放序列(acquire-release  ordering)。

获取-释放序列
这个序列是松散序列(relaxed ordering)的加强版；虽然操作依旧没有统一的顺序，但是在这个序列引入了同步。
这种序列模型中，
原子加载就是获取(acquire)操作(memory_order_acquire)，
原子存储就是释放(memory_order_release)操作，

原子读-改-写操作(例如fetch_add()或exchange())在这里，不是“获取”，就是“释放”，或者两者兼有的操作(memory_order_acq_rel)。

这里，同步在线程释放和获取间是成对的(pairwise)。释放操作与获取操作同步，这样就能读取已写入的值。这意味着不同线程看到的序列虽不同，但这些序列都是受限的

std::thread a(write_x);
std::thread b(write_y);
std::thread c(read_x_then_y);
std::thread d(read_y_then_x);
a.join();
b.join();
c.join();
d.join();

对于读取的结果，两个(读取)线程看到的是两个完全不同的世界。如前所述，这可能是因为这里没有对先行顺序进行强制规定导致的。

与同步传递相关的获取-释放序列

获取-释放序列和memory_order_consume的数据相关性

。。上面内容很多的。。都跳过了。

5.3.4  释放队列与同步

在这里可以做一个简单的概述。当存储操作被标记为
memory_order_release，memory_order_acq_rel或memory_order_seq_cst，加载被标记为
memory_order_consum，memory_order_acquire或memory_order_sqy_cst，并且操作链上
的每一加载操作都会读取之前操作写入的值，因此链上的操作构成了一个释放序列(release
sequence)，并且初始化存储同步(对应memory_order_acquire或memory_order_seq_cst)或
是前序依赖(对应memory_order_consume)的最终加载。操作链上的任何原子“读-改-写”操作
可以拥有任意个存储序列(甚至是memory_order_relaxed)。

。。。。。。。。

。这里有段代码，ch5中，但是会报错，不知道为什么  cnt变成  负数了。

-858993460表示一个没有初始化的值,即没有赋值的变量被输出了

当只有一个消费者线程时还好，fetch_sub()是一个带有memory_order_acquire的读取操作，
并且存储操作是带有memory_order_release语义，所以存储与加载同步，线程可以从缓存中
读取元素。当有两个读取线程时，第二个fetch_sub()操作将看到被第一个线程修改的值，且
没有值通过store写入其中。先不管释放序列的规则，第二个线程与第一个线程不存在先行关
系，并且对共享缓存中值的读取也不安全，除非第一个fetch_sub()是带有
memory_order_release语义的，这个语义为两个消费者线程建立了不必要的同步。无论是释
放序列的规则，还是带有memory_order_release语义的fetch_sub操作，第二个消费者看到的
是一个空的queue_data，无法从其获取任何数据，并且还会产生条件竞争。幸运的是，第一
个fetch_sub()对释放顺序做了一些事情，所以store()能同步与第二个fetch_sub()操作。两个消
费者线程间不需要同步关系。这个过程在图5.7中展示，其中虚线表示的就是释放顺序，实线
表示的是先行关系。

5.3.5  栅栏  fence

void write_x_then_y()
{
x.store(true,std::memory_order_relaxed); // 1
std::atomic_thread_fence(std::memory_order_release); // 2
y.store(true,std::memory_order_relaxed); // 3
}
。不是锁，是  内存屏障。

5.3.6  原子操作对非原子的操作排序

5.3.7  非原子操作排序

**第五章后半部分，很深**

第6章  基于锁的并发数据结构设计

6.1  为并发设计的意义何在？

设计并发数据结构意味着，多个线程可以并发的访问这个数据结构，线程可对这个数据结构做相同或不同的操作，并且每一个线程都能在自己域中看到该数据结构。多线程环境下，无

数据丢失和损毁，所有的数据需要维持原样，且无条件竞争。这样的数据结构，称之为“线程安全”的数据结构。

要为线程提供并发访问数据结构的机会。本质上，是使用互斥量提供互斥特性：在互斥量的保护下，同一时间内只有一个线程可以获取互斥锁。互斥量为了保护数据，显式的阻止了线程对数据结构的并发访问

这称为序列化(serialzation)：线程轮流访问被保护的数据。这是对数据进行串行的访问，而非并发。

减少保护区域，减少序列化操作，就能提升并发访问的能力。

6.1.1  数据结构并发设计的指导与建议(指南)
两方面：一是确保访问安全，二是真正并发访问

第三章，对如何保证数据结构是线程安全的做过简单的描述
确保无线程能够看到修改数据结构的“不变量”时的状态。
小心会引起条件竞争的接口，提供完整操作的函数，而非操作步骤。
注意数据结构的行为是否会产生异常，从而确保“不变量”的状态。
将死锁的概率降到最低。使用数据结构时，需要限制锁的范围，避免嵌套锁的存在。

还需要考虑数据结构对于使用者来说有什么限制；当线程通过特殊的函数对数据结构进行访问时，还有哪些函数能被其他的线程安全调用呢？

不能在构造函数完成前或析构函数完成后，对数据结构进行访问

当数据结构支持赋值操作swap()或拷贝构造时，作为数据结构的设计者，即使数据结构中有大量的函数被线程所操纵，也需要保证这些操作在并发环境下是安全的(或确保这些操作能够独立访问)，以保证并发访问时不会出现错误

设计数据结构时考虑以下问题：
在锁范围中进行操作，是否允许在锁外执行？
数据结构中不同的区域能是否被不同的互斥量所保护？
所有操作都需要同级互斥量保护吗？
能否对数据结构进行简单的修改，以增加并发访问的概率，且不影响操作语义？

这些问题都源于一个指导思想：如何让序列化访问最小化，让真实并发最大化？

。std::shared_mutex

6.2  基于锁的并发数据结构

基于锁的并发数据结构，需要确保访问线程持有锁的时间最短，对于只有一个互斥量的数据结构来说，这十分困难。需要保证数据不被锁之外的操作所访问，并且还要保证不会在结构上产生条件竞争(如第3章所述)。使用多个互斥量来保护数据结构中不同的区域时，问题会暴露的更加明显，当操作需要获取多个互斥锁时，就有可能产生死锁。所以在设计时，使用多个互斥量时需要格外小心。

6.2.1  线程安全栈——使用锁

struct empty_stack : std::exception
{
const char* what() const throw();
};
template<typename T>
class ts_stack
{
private:
std::stack<T> stk;
mutable std::mutex mtx;
public:
ts_stack() {}
ts_stack(const ts_stack& other)
{
std::lock_guard<std::mutex> lock(other.mtx);
stk = other.stk;
}
ts_stack& operator=(const ts_stack&) = delete;
void push(T val)
{
std::lock_guard<std::mutex> lock(mtx);
stk.push(std::move(val));
}
std::shared_ptr<T> pop()
{
std::lock_guard<std::mutex> lock(mtx);
if (stk.empty())
throw empty_stack();

std::shared_ptr<T> const res(std::make_shared<T>(std::move(stk.top())));
stk.pop();
return res;
}
void pop(T& val)
{
std::lock_guard<std::mutex> lk(mtx);
if (stk.empty())
throw empty_stack();

val = std::move(stk.top());
stk.pop();
}
bool empty() const
{
std::lock_guard<std::mutex> lk(mtx);
return stk.empty();
}
};

。对上面的代码分析了一通。

data.push()①的调用可能会抛出一个异常，不是拷贝/移动数据值，就是内存不足。
重载pop()函数是“异常-安全”的。
有两个地方可能会死锁：拷贝构造或移动构造(①，③)和拷贝赋值或移动赋值操作⑤；还有一个潜在死锁的地方在于用户定义的new操作符

6.2.2  线程安全队列——使用锁和条件变量

template<typename T>
class ts_queue
{
private:
mutable std::mutex mtx;
std::queue<T> que;
std::condition_variable cv;

public:
ts_queue() {}

void push(T val)
{
std::lock_guard<std::mutex> lk(mtx);
que.push(std::move(val));
cv.notify_one();
}

void wait_and_pop(T& val)
{
std::unique_lock<std::mutex> lk(mtx);
cv.wait(lk, [this]
//()
{ return !que.empty(); });
val = std::move(que.front());
que.pop();
}

std::shared_ptr<T> wait_and_pop()
{
std::unique_lock<std::mutex> lk(mtx);
cv.wait(lk, [this] { return !que.empty(); });
std::shared_ptr<T> res(std::make_shared<T>(std::move(que.front())));
que.pop();
return res;
}

bool try_pop(T& val)
{
std::lock_guard<std::mutex> lk(mtx);
if (que.empty())
return false;
val = std::move(que.front());
que.pop();
return true;
}

std::shared_ptr<T> try_pop()
{
std::lock_guard<std::mutex> lk(mtx);
if (que.empty())
return std::shared_ptr<T>();

std::shared_ptr<T> res(std::make_shared<T>(std::move(que.front())));
que.pop();
return res;
}

bool empty() const
{
std::lock_guard<std::mutex> lk(mtx);
return que.empty();
}
};

异常安全会有一些变化，不止一个线程等待对队列进行推送操作时，只会有一个线程因

data_cond.notify_one()而继续工作着。但是，如果这个工作线程在wait_and_pop()中抛出一个异常，例如：构造新的 std::shared_ptr<> 对象④时抛出异常，那么其他线程则会永世长眠。这种情况是不可接受的，

所以调用函数就需要改成data_cond.notify_all()，这个函数将唤醒所有的工作线程，不过当大多线程发现队列依旧是空时，又会耗费很多资源让线程重新进入睡眠状态。

第二种替代方案是，有异常抛出的时，让wait_and_pop()函数调用notify_one()，从而让个另一个线程可以去尝试索引存储的值。

第三种替代方案是，将 std::shared_ptr<> 的初始化过程移到push()中，并且存储 std::shared_ptr<> 实例，而不是直接使用数据的值。将 std::shared_ptr<> 拷贝到内部 std::queue<> 中就不会抛出异常了，这样wait_and_pop()又是安全的了。

。。无法理解  场景，  provider  失败，和  consumer  有什么关系。

第三种：
std::queue<std::shared_ptr<T> > data_queue;
std::shared_ptr<T> res=data_queue.front();

6.2.3  线程安全队列——使用细粒度锁和条件变量

。。list的  thread safe  版本，  很长。

。。cpp  标准  好像真没有提供  多线程版的  各种容器，   就是  类似  Java的  concurrency  包。
。。不过  我觉得  boost  应该提供了。

6.3  基于锁设计更加复杂的数据结构

先来看看，在设计查询表时所遇到的一些问题。

6.3.1  编写一个使用锁的线程安全查询表
查询表或字典是一种类型的值(键值)和另一种类型的值进行关联(映射)的方式

C++标准库中相关工具：  std::map<> , std::multimap<> , std::unordered_map<>  和  std::unordered_multimap<>

一个简单的域名系统(DNS)缓存，其特点是相较于  std::map<>  削减了很多的接口。和队列和栈一样，标准容器的接口不适合多线程进行并发访问，因为这些接口都存在固有的条件竞争，所以有些接口需要砍掉或重新修订。

并发访问时， std::map<> 接口最大的问题在于——迭代器

迭代器引用的元素被其他线程删除时，迭代器就会出问题。线程安全的查询表的第一次接口削减，需要绕过迭代器。

查询表的基本操作有：
添加一对“键值-数据”
修改指定键值所对应的数据
删除一组值
通过给定键值，获取对应数据

一种选择是在键值没有对应值的时候进行返回时，允许用户提供一个“默认”值：
mapped_type get_value(key_type const& key, mapped_type default_value);

也可以扩展成返回一个  std::pair<mapped_type, bool>  来代替mapped_type实例，其中bool代
表返回值是否是当前键对应的值。另一个选择是返回一个有指向数据的智能指针，当指针的
值是NULL时，这个键值就没有对应的数据。

为细粒度锁设计一个映射结构
三个常见关联容器的方式：
二叉树，比如：红黑树
有序数组
哈希表

二叉树的方式，不会对提高并发访问的能力；每一个查找或者修改操作都需要访问根节点，因此，根节点需要上锁。虽然，访问线程在向下移动时，这个锁可以进行释放，但相比横跨整个数据结构的单锁，并没有什么优势。

有序数组是最坏的选择，因为你无法提前言明数组中哪段是有序的，所以你需要用一个锁将整个数组锁起来。

那么就剩哈希表了。假设有固定数量的桶，每个桶都有一个键值(关键特性)，以及散列函数。这就意味着你可以安全的对每个桶上锁。当再次使用互斥量(支持多读者单作者)时，就能将并发访问的可能性增加N倍，这里N是桶的数量。

C++标准库提供  std::hash<>  模板

。。这段代码  3页。

6.3.2  编写一个使用锁的线程安全链表

迭代器的问题在于，STL类的迭代器需要持有容器内部属于的引用。当容器可被其他线程修改时，这个引用还是有效的；实际上就需要迭代器持有锁，对指定的结构中的部分进行上锁。在给定STL类迭代器的生命周期中，让其完全脱离容器的控制是很糟糕的做法

替代方案就是提供迭代函数，例如：将for_each作为容器本身的一部分。这就能让容器来对迭代的部分进行负责和锁定，不过这将违反第3章指导意见——对避免死锁建议。为了让for_each在任何情况下都有效，持有内部锁的时，必须调用用户提供的代码。不仅如此，需要传递一个对容器中元素的引用到用户代码中，就是让用户代码对容器中的元素进行操作。可以为了避免传递引用，而传出一个拷贝到用户代码中；不过当数据很大时，拷贝要付出的代价也很大。

。2页半  代码

第7章  无锁并发数据结构设计

上一章  第一组例子就是使用单个互斥量来保护整个数据结构，但之后的例子使用多个锁来保护数据结构不同的部分，并且允许对数据结构进行更高级别的并发访问

7.1  定义和意义

库函数将调用阻塞操作来对线程进行阻塞，在阻塞移除前线程无法继续自己的任务。通常，操作系统会完全挂起一个阻塞线程(并将其时间片交给其他线程)，直到其被其他线程“解阻塞”；“解阻塞”的方式很多，比如解锁一个互斥锁、通知条件变量达成，或让“期望”就绪

不使用阻塞库的数据结构和算法被称为“无阻塞结构”。不过，无阻塞的数据结构并非都是无锁的，那么就让我们见识一下各种各样的无阻塞数据结构吧！

7.1.1  非阻塞数据结构
自旋锁并不是无锁结构

无锁结构的具体定义
无阻碍
如果所有其他线程都暂停了，任何给定的线程都将在一定时间内完成其操作。
无锁
如果多个线程对一个数据结构进行操作，经过一定时间后，其中一个线程将完成其操作。
无等待
即使有其他线程也在对该数据结构进行操作，每个线程都将在一定的时间内完成其操作。

大多数情况下无阻碍算法不是特别有用——其他线程都暂停的情况太少见了

7.1.2  无锁数据结构

具有“比较/交换”操作的数据结构，通常在“比较/交换”操作实现中都有一个循环。使用“比较/交换”操作的原因：当有其他线程同时对指定的数据进行修改时，代码将尝试恢复数据。当其他线程被挂起时，“比较/交换”操作执行成功，这样的代码就是无锁的。当执行失败时，就需要一个自旋锁，且这个结构就是“无阻塞-有锁”的结构。

无锁算法中的循环会让一些线程处于“饥饿”状态。如有线程在“错误”时间执行，那么第一个线程将会不停的尝试所要完成的操作(其他程序继续执行)。“无锁-无等待”数据结构的出现，就为了避免这种问题。

7.1.3  无等待数据结构

无等待数据结构：首先是无锁数据结构，并且每个线程都能在有限的时间内完成操作，暂且不管其他线程是如何工作的。由于会和别的线程产生冲突，所以算法可以进行无数次尝试，因此并不是无等待的。本章的大多数例子都有一种特性——对compare_exchange_weak或compare_exchange_strong操作进行循环，并且循环次数没有上限。

正确实现一个无锁的结构十分困难。因为，要保证每一个线程都能在有限的时间里完成操作，就需要保证每一个操作可以被一次性执行完；当线程执行到某个操作时，应该不会让其他线程的操作失败。这就会让算法中所使用到的操作变的相当复杂。

7.1.4  无锁数据结构的利与弊

使用无锁结构的主要原因：将并发最大化

使用无锁数据结构的第二个原因就是鲁棒性。当一个线程在获取锁时被终止，那么数据结构将会被永久性的破坏。不过，当线程在无锁数据结构上执行操作，在执行到一半终止时，数据结构上的数据没有丢失(除了线程本身的数据)，其他线程依旧可以正常执行。

注意不变量的状态，或选择替代品来保持不变量的状态。

注意操作的顺序约束

必须使用原子操作对修改操作进行限制。不过，仅使用原子操作是不够的；需要确定被其他线程看到的修改，是遵循正确的顺序。

因为没有任何锁(有可能存在活锁)，死锁问题不会困扰无锁数据结构

活锁的产生是两个线程同时尝试修改数据结构，但每个线程所做的修改操作都会让另一个线程重启，所以两个线程就会陷入循环，多次的尝试完成自己的操作。

不过活锁的存在时间并不久，因为其依赖于线程调度。所以只是对性能有所消耗，而不是一个长期问题，但这个问题仍需要关注。根据定义，无等待的代码不会被活锁所困扰，因其操作执行步骤有上限。换个角度说，无等待的算法要比等待算法的复杂度高，且即使没有其他线程访问数据结构，也可能需要更多步骤来完成相应操作。

“无锁-无等待”代码的缺点：虽然提高了并发访问的能力，减少了单个线程的等待时间，但是其可能会将整体性能拉低。首先，原子操作的无锁代码要慢于无原子操作的代码，原子操作就相当于无锁数据结构中的锁。

硬件必须通过同一个原子变量对线程间的数据进行同步

7.2  无锁数据结构的例子
一些无锁实现的简单数据结构

无锁结构依赖于原子操作和内存序，以确保多线程以正确的顺序访问数据结构。所有原子操作默认使用的是memory_order_seq_cst内存序。不过，后面的例子中将会降低内存序的要求，使用memory_order_acquire，memory_order_release，甚至memory_order_relaxed。

虽然例子中没有直接使用锁，但需要注意的是对 std::atomic_flag 的使用。一些平台上无锁结构的实现(实际上在C++的标准库的实现中)，使用了内部锁(详见第5章)。另一些平台上，基于锁的简单数据结构可能会更加合适；当然，还有很多平台不明确实现细节；选择一种实现前，需要明确需求，并且配置各种选项以满足需求。

7.2.1  实现一个无锁的线程安全栈

使用  链表  完成一个  栈
添加一个节点相对来说很简单：
1. 创建一个新节点。
2. 当新节点的next指针指向当前的head节点。
3. 让head节点指向新节点。

当有两个线程同时添加节点的时候，第2步和第3步的时候会产生条件竞争：一个线程可能在修改head值时，另一个线程正在执行第2步，并且在第3步中对head进行更新。就会使之前那个线程的结果被丢弃，亦或是造成更加糟糕的后果。

还要注意一个事：当head更新并指向了新节点时，另一个线程就能读取到这个节点了。因此，head设置为指向新节点前，新节点完全准备就绪就变很重要；因为，在这之后就不能对节点进行修改了。

如何应对条件竞争呢？答案就是：第3步的时候使用原子“比较/交换”操作，来保证步骤2对head进行读取时，不会对head进行修改；有修改时可以循环“比较/交换”操作。

。。但是，这种，  要是  2个线程  都新建node  并指向  同一个  head，  然后  串行修改  head指针，   这不还是有问题吗。  2个新建node  的后继  是同一个节点。

。。compare_exchange_weak  会修改  后继

template<typename T>
class lock_free_stack
{
private:
struct node
{
T data;
node* next;

node(T const& _d) : data(_d) {}
};

std::atomic<node*> head;

public:

void push(T const& data)
{
node* const nd = new node(data);
nd->next = head.load();
while (!head.compare_exchange_weak(nd->next, nd));
// 不相等的话， nd->next 会被赋予 head 的值。
}
};

这里唯一一个能抛出异常的就构造新node时①，不过其会自行处理且链表中的内容没有被修改，所以是安全的。

删除数据的方法
1. 读取当前head指针的值。
2. 读取head->next。
3. 设置head到head->next。
4. 通过索引node，返回data数据。
5. 删除索引节点。

有两个线程要从栈中移除数据时，两个线程可能在步骤1中读取到同一个head(值相同)。其中一个线程处理到步骤5时，另一个线程还在处理步骤2，这个还在处理步骤2的线程将会解引用一个悬空指针。这是书写无锁代码所遇到的问题之一，所以现在只能跳过步骤5，让节点泄露。

两个线程读取到同一个head值时，它们将返回同一个节点。这就违反了栈结构的意图，所以需要避免这样的情况发生。可以像在push()函数中解决条件竞争那样来解决这个问题：使用“比较/交换”操作更新head。

void pop(T& result)
{
node* old = head.load();
while (!head.compare_exchange_weak(old, old->next));
result = old->next;
}

有两个节点泄露的问题

首先，这段代码在空链表时不工作：当head指针是空指针时，要访问next指针时，将引起未定义行为。很容易通过对nullptr的检查进行修复(在while循环中)，要不对空栈抛出一个异常，要不返回一个bool值来表明成功与否。

。。竟然不是空指针。  head  是  atomic，里面是  null。  所以后续  while中  判空。

第二个问题就是异常安全。

第3章中介绍栈结构时，了解了在返回值的时候会出现异常安全问题：有异常被抛出时，复制的值将丢失。这种情况下，传入引用是一种可以接受的解决方案；这样就能保证当有异常抛出时，栈上的数据不会丢失。不幸的是，不能这样做；只能在单一线程对值进行返回时，才进行拷贝以确保拷贝操作的安全性，这就意味着在拷贝结束后这个节点就会被删除。因此，通过引用获取返回值的方式没有任何优点：直接返回也是可以的。若想要安全的返回，必须使用第3章中的其他方法：返回指向数据值的(智能)指针。

template<typename T>
class lock_free_stack_2
{
private:
struct node
{
std::shared_ptr<T> data;
node* next;

node(T const& _d) : data(std::make_shared<T>(_d)) {}
};

public:
void push(T const& data)
{
node* const nd = new node(data);
nd->next = head.load();
while (!head.compare_exchange_weak(nd->next, nd));
}

std::shared_ptr<T> pop()
{
node* old = head.load();
while (old &&
!head.compare_exchange_weak(old, old.next));
return old ? old->data : std::shared_ptr<T>();
}
};

7.2.2  终止内存泄露：使用无锁数据结构管理内存

pop()时，为了避免条件竞争(当有线程删除一个节点的同时，其他线程还持有指向该节点的指针，并且要解引用)选择了带有内存泄露的节点。

基本问题在于，当要释放一个节点时，需要确认其他线程没有持有这个节点。只有一个线程调用pop()时，可以放心的释放。

添加节点到“可删除”列表中时，就能从结点中提取数据了。没有线程通过pop()访问节点时，可以安全的删除这些节点了。怎么知道没有线程调用pop()了呢？很简单——计数即可。

// 这个不完整，没有 node 的定义。 head 的定义。
template<typename T>
class lock_free_stack_3
{
private:
std::atomic<unsigned> th_in_pop;        // thread in pop
void try_reclaim(node* old);

public:
std::shared_ptr<T> pop()
{
++th_in_pop;                // ..没有看到 --的地方。

node* old = head.load();
while (old &&
!head.compare_exchange_weak(old, old->next));
std::shared_ptr<T> res;
if (old)
res.swap(old->data);

try_reclaim(old);
return res;
}
};

threads_in_pop①原子变量用来记录有多少线程试图弹出栈中的元素。调用pop()②函数时，计数器加一；调用try_reclaim()时，计数器减一；这个函数被节点调用时，说明节点已被删除④。因为暂时不需要将节点删除，可以通过swap()函数来删除节点上的数据③(而非只是拷贝指针)，当不再需要这些数据的时候，这些数据会自动删除，而不是持续存在着(因为还有对未删除节点的引用)。

。。try_reclaim  的代码  2页。

template<typename T>
class lock_free_stack_4
{
private:
std::atomic<node*> to_be_deleted;
static void delete_nodes(node* nodes)
{
while (nodes)
{
node* nxt = nodes->next;
delete nodes;
nodes = nxt;
}
}
void try_recliam(node* old_head)
{
if (threads_in_pop == 1)
{
node* nodes_to_delete = to_be_deleted.exchange(nullptr);
if (!--threads_in_pop)
{
delete_nodes(nodes_to_delete);
}
else if (nodes_to_delete)
{
chain_pending_nodes(nodes_to_delete);
}
delete old_head;
}
else
{
chain_pending_node(old_head);
--threads_in_pop;
}
}
void chain_pending_nodes(node* nodes)
{
node* lst = nodes;
while (node* const nxt = lst->next)
{
lst = nxt;
}
chain_pending_nodes(nodes, lst);
}
void chain_pending_nodes(node* first, node* last)
{
last->next = to_be_deleted;
while (!to_be_deleted.compare_exchange_weak(last->next, first));
}
void chain_pending_node(node* n)
{
chain_pending_nodes(n, n);
}
};

。。。。。
。很多说明

高负荷的情况，因为其他线程在初始化之后都能进入pop()，所以to_ne_deleted链表将会无界的增加，并且会再次泄露。

最简单的替换机制就是使用风险指针(hazard pointer)。

7.2.3  使用风险指针检测不可回收的节点

“风险指针”这个术语引用于Maged Michael的技术发现[1]。之所以这样叫是因为，删除一个节点可能会让其他引用其的线程处于危险之中。其他线程持有已删除节点的指针时，对其进行解引用操作时，会出现未定义行为。

基本观点就是，当有线程去访问要被(其他线程)删除的对象时，会先设置对这个对象设置风险指针，而后通知其他线程——使用这个指针是危险的行为。当这个对象不再被需要，那么就可以清除风险指针了。

当线程想要删除一个对象，就必须检查系统中其他线程是否持有风险指针。当没有风险指针时，就可以安全删除对象。否则，就必须等待风险指针消失。这样，线程就需要周期性的检查要删除的对象是否能安全删除。

C++中应该怎么做呢？

首先，需要能存储指向访问对象的指针，也就是风险指针。指针必须能让所有线程看到，需要线程能够对数据结构进行访问。正确且高效的分配这些线程的确是一个挑战，所以这个问题放在后面解决。

假设有一个get_hazard_pointer_for_current_thread()函数，可以返回风险指针的引用。当读取一个指针，并且想要解引用它的时候，就需要这个函数——这种情况下head数值源于下面的列表：

std::shared_ptr<T> pop()
{
std::atomic<void*>& hp = get_hazard_pointer_for_current_thread();
node* old = head.load();
node* tmp;
do
{
tmp = old;
hp.store(old);
old = head.load();
} while (old != tmp);
// ...
}

while循环能保证node不会在读取旧head指针①时，以及在设置风险指针的时被删除。
这种模式下，其他线程不知道有线程对这个节点进行了访问。
幸运的是，旧head节点要被删除时，head本身会发生变化，所以需要对head进行检查并持续循环，直到head指针中的值与风险指针中的值相同③。

使用风险指针，如同依赖对已删除对象的引用。使用默认的new和delete操作对风险指针进行操作时，会出现未定义行为，所以需要确定实现是否支持这样的操作，或使用自定义内存分配器来保证用法的正确性。

std::shared_ptr<T> pop()
{
std::atomic<void*>& hp = get_hazard_pointer_for_current_thread();
node* old = head.load();
node* tmp;
do
{
tmp = old;
hp.store(old);
old = head.load();
} while (old != tmp);

while (old && !head.compare_exchange_strong(old, old->next));

hp.store(nullptr);
std::shared_ptr<T> res;
if (old)
{
res.swap(old->data);
if (outstanding_hazard_pointers_for(old))
{
recliam_later(old);
}
else
{
delete old;
}
delete_nodes_with_no_hazards();
}
return res;
}

。。难懂。跳了。。

对风险指针(较好)的回收策略
。。。跳

风险指针另一个缺点：与IBM申请的专利所冲突

所以，是否有非专利的内存回收技术，且能被大多数人所使用呢？很幸运，的确有。引用计数就是这样的机制。

7.2.4  使用引用计数的节点

当能安全并精确的识别节点是否还被引用，以及没有线程访问这些节点的具体时间，即可判断是够能将对应节点进行删除。
风险指针是通过将使用中的节点存放到链表中，
而引用计数是通过对每个节点上访问的线程数量进行统计。

首先，想到的就是由 std::shared_ptr<> 来完成这个任务，其有内置引用计数的指针。不幸的是，虽然 std::shared_ptr<> 上的一些操作是原子的，不过其也不能保证是无锁的。智能指针上的原子操作和对其他原子类型的操作并没有什么不同，但是 std::shared_ptr<> 旨在用于有多个上下文的情况下，并且在无锁结构中使用原子操作，无异于对该类增加了很多性能开销。

如果平台支持  std::atomic_is_lock_free(&some_shared_ptr) 实现返回true，那么所有内存回收问题就都迎刃而解了

template<typename T>
class lock_free_stack_7
{
private:
struct node
{
std::shared_ptr<T> data;
std::shared_ptr<node> next;
node(T const& _d) : data(std::make_shared<T>(_d)) {}
};
std::shared_ptr<node> head;

public:
void push(T const& data)
{
std::shared_ptr<node> const nd = std::make_shared<node>(data);
nd->next = head.load();
while (!std::atomic_compare_exchange_weak(&head, &nd->next, nd));
}
std::shared_ptr<T> pop()
{
std::shared_ptr<node> old = std::atomic_load(&head);
while (old && !std::atomic_compare_exchange_weak(&head, &old, old->next));
if (old)
{
std::atomic_store(&old->next, std::shared_ptr<node>());
return old->data;
}
return std::shared_ptr<T>();
}
~lock_free_stack_7()
{
while (pop());
}
};

对 std::shared_ptr<> 使用无锁原子操作的实现不仅很少见，而且能想起为其使用一致性的原子操作也很难。

并发技术规范扩展提供了工具，可以来解决这个问题，扩展中在头文件 <experimental/atomic> 中提供了 std::experimental::atomic_shared_ptr<T> 。多数情况下这与 std::atomic<std::shared_ptr<T>> 等价(除非是 std::atomic<> 不使用 std::shared_ptr<T> )

。。就是上面的  std::atomic_xxx  直接作用到  head上。

std::experimental::atomic_shared_ptr<node> next;

std::experimental::atomic_shared_ptr<node> head;

while(!head.compare_exchange_weak(new_node->next,new_node));

一些情况下，使用  std::shared_ptr<>  实现的结构并非无锁，这就需要手动管理引用计数。

一种方式是对每个节点使用两个引用计数：内部计数和外部计数。两个值的总和就是对这个节点的引用数。外部计数记录有多少指针指向节点，即在指针每次进行读取的时候，外部计数加一。当线程结束对节点的访问时，内部计数减一。指针在读取时，外部计数加一；读取结束时，内部计数减一。

。。被访问，外部计数++，  结束访问，内部计数--

不需要“外部计数-指针”对时(该节点就不能被多线程所访问了)，外部计数减一和在被弃用的时候，内部计数将会增加。当内部计数等于0，就没有指针对该节点进行引用，就可以将该节点安全的删除。

template<typename T>
class lock_free_stack_9
{
private:
struct node;
struct counted_node_ptr
{
int external_count;
node* ptr;
};
struct node
{
std::shared_ptr<T> data;
std::atomic<int> internal_count;
counted_node_ptr next;

node(T const& _d) : data(std::make_shared<T>(_d)), internal_count(0) {}
};
std::atomic<counted_node_ptr> head;

public:
~lock_free_stack()
{
while (pop());
}

void push(T const& data)
{
counted_node_ptr nd;
nd.ptr = new node(data);
nd.external_count = 1;
nd.ptr->next = head.load();
while (!head.compare_exchange_weak(nd.ptr->next, nd));
}
};

counted_node_ptr体积够小，能够让 std::atomic<counted_node_ptr> 无锁。一些平台上支持双字比较和交换操作，可以直接对结构体进行操作。平台不支持这样的操作时，最好使

用 std::shared_ptr<> 变量；
当类型的体积过大，超出了平台支持指令，那么原子 std::atomic<> 将使用锁来保证其操作的原子性

。。pop代码一页。P279

目前，使用默认 std::memory_order_seq_cst 内存序来规定原子操作的执行顺序。大多数系统中，这种操作方式很耗时，且同步操作的开销要高于内存序。现在，可以考虑对数据结构的逻辑进行修改，对数据结构的部分放宽内存序要求；

7.2.5  无锁栈上的内存模型
修改内存序之前，需要检查一下操作之间的依赖关系。而后，再去确定适合这种需求关系的最小内存序。
需要在从线程的视角进行观察。其中最简单的视角就是，向栈中推入一个数据项，之后让其他线程从栈中弹出这个数据。
即使在简单的例子中，都需要三个重要的数据参与。
1. counted_node_ptr转移的数据head。
2. head引用的node。
3. 节点所指向的数据项。

做push()的线程，会先构造数据项和节点，再设置head。做pop()的线程，会先加载head的值，再做在循环中对head做“比较/交换”操作，并增加引用计数，再读取对应的node节点，获取next的指向值，现在可以看到一组需求关系。

next的值是普通的非原子对象，所以为了保证读取安全，必须确定存储(推送线程)和加载(弹出线程)的先行关系。

因为原子操作就是push()函数中的compare_exchange_weak()，需要释放操作来获取两个线程间的先行关系，compare_exchange_weak()必须是 std::memory_order_release 或更严格的内存序。当compare_exchange_weak()调用失败，什么都不会改变，并且可以持续循环下去，所以使用 std::memory_order_relaxed 就足够了。

void push(T const& data)
{
counted_node_ptr new_node;
new_node.ptr=new node(data);
new_node.external_count=1;
new_node.ptr->next=head.load(std::memory_order_relaxed)
while(!head.compare_exchange_weak(new_node.ptr->next,new_node,
std::memory_order_release,std::memory_order_relaxed));
}

pop()的实现呢？为了确定先行关系，必须在访问next值之前使用 std::memory_order_acquire 或更严格的内存序操作。因为，increase_head_count()中使用compare_exchange_strong()就获取next指针指向的旧值，所以要其获取成功就需要确定内存序。如同调用push()那样，当交换失败，循环会继续，所以在失败时使用松散的内存序

void increase_head_count(counted_node_ptr& old_counter)
{
counted_node_ptr new_counter;
do
{
new_counter=old_counter;
++new_counter.external_count;
}
while(!head.compare_exchange_strong(old_counter,new_counter,
std::memory_order_acquire,std::memory_order_relaxed));
old_counter.external_count=new_counter.external_count;
}

。。2页的代码。

7.2.6  实现一个无锁的线程安全队列

。。。。。

7.3  对于设计无锁数据结构的指导和建议

7.3.1 指导建议：使用 std::memory_order_seq_cst 的原型

std::memory_order_seq_cst 比起其他内存序要简单的多，因为所有操作都将其作为总序。本章的所有例子，都是从 std::memory_order_seq_cst 开始，只有当基本操作正常工作的时候，才放宽内存序的选择。

这种情况下，使用其他内存序就是优化(早期可以不用这样做)。通常，当了解整套代码对数据结构的操作后，才能决定是否要放宽内存序的选择。

7.3.2  指导建议：对无锁内存的回收策略
三种技术来保证内存可以被安全的回收：
等待无线程对数据结构进行访问时，删除所有等待删除的对象。
使用风险指针来标识正在被线程访问的对象。
对对象进行引用计数，当没有线程对对象进行引用时将其删除。

所有例子的想法都是使用一种方式去跟踪指定对象上的线程访问数量，当没有现成对对象进行引用时将对象删除

7.3.3  指导建议：小心ABA问题
基于“比较/交换”的算法中要格外小心“ABA问题”。

“ABA问题”在使用释放链表和循环使用节点的算法中很是普遍，而将节点返回给分配器，则不会引起这个问题。

7.3.4  指导建议：识别忙等待循环和帮助其他线程

第8章  并发代码设计

8.1  线程间划分工作的技术

需要决定使用多少个线程，并且这些线程应该去做什么。还需要决定是使用“全能”的线程去完成所有的任务，还是使用“专业”线程只去完成一件事情，或将两种方法混合。
使用并发的时候，需要作出诸多选择来驱动并发，选择会决定代码的性能和清晰度。因此选择至关重要，所以设计应用结构时，需要作出适当的决定。

8.1.1  线程处理前对数据进行划分
最简单的并行算法，就是并行化的 std::for_each ，其会对一个数据集中每个元素执行同一个操作。

使用过MPI(Message Passing Interface)[1]和OpenMP[2]的同学对这个结构一定很熟悉：一项任务被分割成多个，放入一个并行任务集中，执行线程独立的执行这些任务，结果在会有主线程中合并。

8.1.2  递归划分
。。forkjoin?

std::thread::hardware_concurrency()

。。3页的代码。

8.1.3  通过任务类型划分工作

对分工的排序，也就是从并发分离关注结果；每个线程都有不同的任务，这意味着真正意义上的线程独立。其他线程偶尔会向特定线程交付数据，或是通过触发事件的方式来进行处理；不过总体而言，每个线程只需要关注自己所要做的事情即可。其本身就是良好的设计，每一段代码只对自己的部分负责。

分离关注

当有多个任务需要持续运行一段时间，或需要及时进行处理的事件(比如，按键事件或传入网络数据)，且还有其他任务正在运行时，单线程应用采用的是单责任原则处理冲突。单线程的世界中，代码会执行任务A(部分)后，再去执行任务B(部分)，再检查按钮事件，再检查传入的网络包，然后在循环回去，执行任务A。这将会使得任务A复杂化，因为需要存储完成状态，以及定期从主循环中返回。

当使用独立线程执行任务时，操作系统会帮忙处理接口问题。执行任务A时，线程可以专注于执行任务，而不用为保存状态从主循环中返回。操作系统会自动保存状态，当需要的时候将线程切换到任务B或任务C。如果目标系统是带有多核或多处理器，任务A和任务B可很可能真正的并发执行。

处理按键时间或网络包的代码，就能及时执行了。所有事情都完成的很好，用户得到了及时的响应；当然，作为开发者只需要写具体操作的代码即可，不用再将控制分支和使用用户交互混在一起了。

事实真会如上面所说的那样简单？一切取决于细节。

多线程下有两个危险需要分离关注。

第一个是对错误担忧的分离，主要表现为线程间共享着很多的数据，或者不同的线程要相互等待，这两种情况都是因为线程间很密切的交互。这种情况发生时，就需要看一下为什么需要这么多交互。当所有交互都有同样的问题，就应该使用单线程来解决，并将引用同一源的线程提取出来。或者当有两个线程需要频繁的交流，在没有其他线程时，就可以将这两个线程合为一个线程。

当通过任务类型对线程间的任务进行划分时，不应该让线程处于完全隔离的状态。当多个输入数据集需要使用同样的操作序列，可以将序列中的操作分成多个阶段，来让每个线程执行。

划分任务序列

当任务会应用到相同操作序列，去处理独立的数据项时，就可以使用流水线(pipeline)系统进行并发。这好比一个物理管道：数据流从管道一端进入，进行一系列操作后，从管道另一端出去。

使用这种方式划分工作，可以为流水线中的每一阶段操作创建一个独立线程。当一个操作完成，数据元素会放在队列中，以供下一阶段的线程提取使用。这就允许第一个线程在完成对于第一个数据块的操作并要对第二个数据块进行操作时，第二个线程可以对第一个数据块执行管线中的第二个操作。

这就是线程间划分数据的一种替代方案(如8.1.1描述)，这种方式适合于操作开始前，且对输入数据处长度不清楚的情况。例如：数据来源可能是从网络，或者可能是通过扫描文件系统来确定要处理的文件。

8.2  影响并发代码性能的因素

8.2.1  多少个处理器？

std::thread::hardware_concurrency()

。。Java：Runtime.getRuntime().availableProcessors());

需要谨慎使用 std::thread::hardware_concurrency() ，因为代码不会考虑有其他运行在系统上的线程(除非已经将系统信息进行共享)。最坏的情况就是，多线程同时调用 std::thread::hardware_concurrency() 函数来对线程数量进行扩展，这样将导致庞大的超额认购。 std::async() 就能避免这个问题，因为标准库会对所有的调用进行适当的安排。同样，谨慎的使用线程池也可以避免这个问题。

随着处理器数量的增加，另一个问题就会来影响性能：多个处理器尝试访问同一个数据。

8.2.2  数据争用与乒乓缓存
当有线程对数据进行修改的时候，这个修改需要更新到其他核芯的缓存中去，就要耗费一定的时间。

根据线程的操作性质，以及使用到的内存序，这样的修改可能会让第二个处理器停下来，等待硬件内存更新缓存中的数据。即便是精确的时间取决于硬件的物理结构，不过根据CPU指令，这是一个特别特别慢的操作，相当于执行成百上千个独立指令

思考下面简短的代码段
std::atomic<unsigned long> counter(0);
void processing_loop()
{
while(counter.fetch_add(1,std::memory_order_relaxed)  <100000000)
{
do_something();
}
}

counter变量是全局的，所以任何线程都能调用processing_loop()去修改同一个变量。因此，当新增加的处理器时，counter变量必须要在缓存内做一份拷贝，再改变自己的值，或其他线程以发布的方式对缓存中的拷贝副本进行更新。即使用 std::memory_order_relaxed ，编译器不会为任何数据做同步操作，fetch_add是一个“读-改-写”操作，因此就要对最新的值进行检索。

一个处理器准备更新这个值，另一个处理器正在修改这个值，所以该处理器就需要等待第二个处理器更新完成，并且完成更新传递时才能执行更新。这种情况被称为高竞争(high  contention)。如果处理器很少需要互相等待，这种情况就是低竞争(low contention)。

循环中counter的数据将在每个缓存中传递若干次，这就叫做乒乓缓存(cache ping-pong)。这种情况会对应用的性能有着重大的影响。当一个处理器因为等待缓存转移而停止运行时，这个处理器就不能做任何事情，所以对于整个应用来说这是一个坏消息。

如果乒乓缓存是一个糟糕的现象，该怎么避免它呢？本章最后，答案会与提高并发潜能的指导意见相结合：减少两个线程对同一个内存位置的竞争。
虽然，要实现起来并不简单。即使给定内存位置被一个线程所访问，可能还是会有乒乓缓存存在，是因为另一种叫做伪共享(false sharing)的效应。

8.2.3  伪共享

处理器缓存通常不会用来处理单个存储点，会用来处理称为缓存行(cache lines)的内存块。内存块通常大小为32或64字节，实际大小需要由正在使用的处理器模型来决定。

当线程访问的一组数据是在同一数据行中，对于应用的性能来说就要好于对多个缓存行进行传播。不过，在同一缓存行存储的是无关数据时，且需要被不同线程访问，这就会造成性能问题。

假设你有一个int类型的数组，并且有一组线程可以访问数组中的元素，且对数组的访问很频繁(包括更新)。通常int类型的大小要小于一个缓存行，同一个缓存行中可以存储多个数据项。

因此，即使每个线程都能对数据中的成员进行访问，硬件缓存还是会产生乒乓缓存。
每当线程访问0号数据项，并对其值进行更新时，缓存行的所有权就需要转移给执行该线程的处理器，这仅是为了更新1号数据项的线程获取1号线程的所有权。
缓存行是共享的(即使没有数据存在)，因此使用伪共享来称呼这种方式。

这个问题的解决办法就是对数据进行构造，让同一线程访问的数据项存在临近的内存中(就像是放在同一缓存行中)，这样那些能被独立线程访问的数据将分布在相距很远的地方，并且可能是存储在不同的缓存行中。本章接下来的内容中可以看到，这种思路对代码和数据设计的影响。C++17并准在头文件 <new> 中定义了 std::hardware_destructive_interference_size 它指定了当前编译目标可能共享的连续字节的最大数目。如果确保数据间隔大于等于这个字节数，就不会有错误的共享存在了

8.2.4  如何让数据紧凑？

伪共享发生的原因：某个线程所要访问的数据过于接近另一线程的数据，另一个是与数据布局相关的陷阱会直接影响单线程的性能。问题在于数据过于接近：数据被单线程访问时，数据就已经在内存中展开，就像是分布在不同的缓存行上。另一方面，当内存中有能被单线程访问紧凑的数据时，就如同数据分布在同一缓存行上。因此，当数据已传播，将会有更多的缓存行从处理器的缓存上加载数据，这会增加访问内存的延迟，以及降低数据的性能(与紧凑的数据存储地址相比较)。

。。。感觉。。不太对。。可能  我还没有到那个层级。

现在，对于单线程代码来说就很关键了，何至于此呢？原因就是任务切换(task switching)。如果系统中的线程数量要比核芯多，每个核上都要运行多个线程。这就会增加缓存的压力，为了避免伪共享，努力让不同线程访问不同缓存行。因此，当处理器切换线程时，就要对不同内存行上的数据进行重新加载(当不同线程使用的数据跨越了多个缓存行时)，而非对缓存中的数据保持原样(当线程中的数据都在同一缓存行时)。C++17在头文件 <new> 中指定了一个常数 std::hardware_constructive_interference_size ，这是同一高速缓存行上的连续字节的最大数目(如果对齐)。当可以将所需的数据放在这个字节数内，就能提高缓存命中率。

。。感觉就是：  这个thread需要的数据，放在  一起，   把  其他线程需要的数据  从  这里  拿出去，并远离。
。。放一起，使得  cache line  命中率高。
。。拿出去&远离，  使得  不会出现  cache ping pong。

如果线程数量多于内核或处理器数量，操作系统可能也会选择将一个线程安排给这个核芯一段时间，之后再安排给另一个核芯一段时间。就需要将缓存行从一个内核上，转移到另一个内核上；这样的话，就需要转移很多缓存行，也就意味着要耗费很多时间。虽然，操作系统通常会避免这样的情况发生，不过当其发生的时候，对性能就会有很大的影响。

当有超级多的线程准备运行时(非等待状态)，任务切换问题就会频繁发生。这个问题我们之前也接触过：超额认购。

8.2.5  超额认购和频繁的任务切换
多线程系统中，通常线程的数量要多于处理的数量。

有时，线程数量大于处理器数量，对性能影响的主要来源是CPU的能力，而非I/O。

8.3  为多线程性能设计数据结构

当为多线程性能而设计数据结构的时候，需要考虑竞争(contention)，伪共享(false sharing)和数据距离(data proximity)，这三个因素对于性能都有着重大的影响，并且通常可以改善数据布局，或者将其他线程的数据元素进行修改。

8.3.1  为复杂操作划分数组元素

将两个很大的矩阵进行相乘
。左行  *  右列

假设两个矩阵都有上千行和上千列，使用多线程来优化矩阵乘法。通常，非稀疏矩阵可以用一个大数组来代表，也就是第二行的元素紧随着第一行的，以此类推。
为了完成矩阵乘法，这里就需要三个大数组。为了优化性能，需要仔细考虑数据访问的模式，特别是向第三个数组中写入的方式。

。2个矩阵相乘，产生一个  矩阵，  所以  2+1=3

线程间划分工作有很多种方式。假设矩阵的行或列数量大于处理器的数量，可以让每个线程计算出结果矩阵列上的元素，或是行上的元素，亦或计算一个子矩阵。

对于一个数组来说访问连续的元素是最好的方式，因为这将会减少缓存的刷新，并且降低伪共享的概率。如果要让每个线程处理几行，线程需要读取第一个矩阵中的每一个元素，并且读取第二个矩阵上的相关行上的数据，不过这里只需要对列的值进行写入。

给定的两个矩阵是以行连续的方式存储，这就意味着当访问第一个矩阵的第一行的前N个元素，而后是第二行的前N个元素，以此类推(N是列的数量)。其他线程会访问每行的的其他元素，访问相邻的列，所以从行上读取的N个元素也是连续的，这将最大程度的降低伪共享的几率。当然，如果空间已经被N个元素所占有，且N个元素也就是每个缓存行上具体的存储元素数量，就会让伪共享的情况消失，因为线程将会对独立缓存行上的数据进行操作。

。。。。。？
。。不过想到：  把  第二个矩阵  做一个  行列转置  。  这样  都是读取  行，都是  读取  连续的元素。  直接起飞。

因此，将矩阵分成小块或正方形的块，要比使用单线程来处理少量的列好的多。当然，可以根据源矩阵的大小和处理器的数量，在运行时对块的大小进行调整。和之前一样，性能是很重要的指标时，就需要对目标架构上的各项指标进行测量，并且查阅相关领域的文献——如果只是做矩阵乘法，我认为这并不是最好的选择。

如果不做矩阵乘法，该如何对上面提到的方案进行应用呢？同样的原理可以应用于任何情况，这种情况就是有很大的数据块需要在线程间进行划分；仔细观察所有数据访问的各个方面，以及确定性能问题产生的原因。各种领域中，出现问题的情况都很相似：改变划分方式就能够提高性能，而不需要对基本算法进行任何修改。

8.3.2  其他数据结构中的数据访问模式
同样的考虑适用于想要优化数据结构的数据访问模式，就像优化对数组的访问：
尝试调整数据在线程间的分布，就能让同一线程中的数据紧密联系在一起。
尝试减少线程上所需的数据量。
尝试让不同线程访问不同的存储位置，以避免伪共享。

一种测试伪共享问题的方法是：对大量的数据块填充数据，让不同线程并发的进行访问。比如，可以使用：
struct protected_data
{
std::mutex m;

char padding[65536]; // 如果你的编译器不支持std::hardware_destructive_interference_size，可以使用类似65536字节，这个数字肯定超过一个缓存行

my_data data_to_protect;
};

用来测试互斥量竞争

或
struct my_data
{
data_item1 d1;
data_item2 d2;
char padding[65536];
};
my_data some_array[256];
用来测试数组数据中的伪共享。如果这样能够提高性能，就能知道伪共享在这里的确存在。

8.4  设计并发代码的注意事项
线程间划分工作的方法，影响性能的因素
异常安全和可扩展性

随着系统中核数的增加，性能越来越高(无论是在减少执行时间，还是增加吞吐率)，这样的代码称为“可扩展”代码。理想状态下，性能随着核数的增加线性增长，也就是当系统有100个处理器时，其性能是系统只有1核时的100倍。

8.4.1  并行算法中的异常安全

。代码

异常要在哪抛出：基本上就是在调用函数的地方抛出异常，或在用户定义类型上执行某个操作时可能抛出异常。

添加异常安全

如果你仔细了解过新线程用来完成什么样的工作，要返回一个计算的结果的同时，允许代码产生异常。这可以与 std::packaged_task 和 std::future 相结合，来解决这个问题

std::packaged_task<T(Iterator,Iterator)>

问题就已经解决：工作线程上抛出的异常，可以在主线程上抛出
可以使用类似  std::nested_exception  来对所有抛出的异常进行捕捉

当生成第一个新线程和当所有线程都汇入主线程时抛出异常；这会让线程产生泄露。最简单的方法就是捕获所有抛出的线程，汇入的线程依旧是joinable()的，并且会再次抛出异常

try
{
for(unsigned long i=0;i<(num_threads-1);++i)
{
// ... as before
}
T last_result=accumulate_block()(block_start,last);
std::for_each(threads.begin(),threads.end(),
std::mem_fn(&std::thread::join));
}
catch(...)
{
for(unsigned long i=0;i<(num_thread-1);++i)
{
if(threads[i].joinable())
thread[i].join();
}
throw;
}

无论线程如何离开这段代码，所有线程都可以被汇入

try catch  不美观：

class join_threads
{
std::vector<std::thread>& threads;
public:
explicit join_threads(std::vector<std::thread>& threads_):  threads(threads_)
{}
~join_threads()
{
for(unsigned long i=0;i<threads.size();++i)
{
if(threads[i].joinable())
threads[i].join();
}
}
};

std::async()的异常安全
需要显式管理线程的时候，需要代码是异常安全的

下面的代码将展示使用 std::async() 完成异常安全的实现

。也是通过  async  返回的  future  的  get  来抛出异常。

8.4.2  可扩展性和Amdahl定律

一种简化的方式就是就是将程序划分成“串行”部分和“并行”部分。
串行部分：只能由单线程执行一些工作的地方。
并行部分：可以让所有可用的处理器一起工作的部分。当在多处理系统上运行应用时，
“并行”部分理论上会完成的相当快，因为其工作被划分为多份，放在不同的处理器上执行。
“串行”部分则不同，只能一个处理器执行所有工作。

这样的(简化)假设下，就可以随着处理数量的增加，估计一下性能的增益：当程序“串行”部分的时间用fs来表示，那么性能增益(P)就可以通过处理器数量(N)进行估计：

P = 1 / (fs + (1-fs)/N)

这就是Amdahl定律，讨论并发程序性能的时候都会引用到的公式。

8.4.3  使用多线程隐藏延迟
现在假设，线程在一个处理器上运行时不会偷懒，并且做的工作都很有用。当然，这只是假设；实际应用中，线程会经常因为等待某些事情而阻塞。

对修改(线程数量)前后性能的测量很重要；优化的线程数量高度依赖要完成工作的先天属性，以及等待时间所占的百分比。

应用可能不用额外的线程，而使用CPU的空闲时间。例如，如果一个线程因为I/O操作被阻塞，这个线程可能会使用异步I/O(如果可以用的话)，当I/O操作在后台执行完成后，线程就可以做其他有用的工作了。

。。没有明确的观点。。感觉就是，  使用多线程  更充分地使用CPU (就是  读取文件时，CPU不是等待，而是处理其他事件)

8.4.4  使用并发提高响应能力

很多流行的图形化用户接口框架都是事件驱动型(event driven)，对图形化接口进行操作是通过按下按键或移动鼠标进行，将产生一系列需要应用处理的事件或信息。

通过使用并发分离关注，可以将一个很长的任务交给一个全新的线程，并且留下一个专用的GUI线程来处理这些事件。线程可以通过简单的机制进行通讯，而不是将事件处理代码和任务代码混在一起。

通过这种方式对关注进行分离，用户线程将总能及时的对事件进行响应，及时完成任务需要花费很长时间。

8.5  在实践中设计并发代码

8.5.1  并行实现：  std::for_each

class join_threads
{
std::vector<std::thread>& threads;
public:
explicit join_threads(std::vector<std::thread>& threads_) :threads(threads_) {}

~join_threads()
{
for (size_t i = 0; i < threads.size(); ++i)
{
if (threads[i].joinable())
threads[i].join();
}
}
};

template<typename Iter, typename Func>
void parallel_for_each(Iter fst, Iter lst, Func f)
{
const size_t sz1 = std::distance(fst, lst);
if (!sz1)
return;

const size_t min_per_th = 25;
const size_t mx_th = (sz1 + min_per_th - 1) / min_per_th;
const size_t hd_th = std::thread::hardware_concurrency();
const size_t num_th = std::min(hd_th == 0 ? 2 : hd_th, mx_th);
const size_t block_size = sz1 / num_th;

std::vector<std::future<void>> futures(num_th - 1);
std::vector<std::thread> threads(num_th - 1);
join_threads joiner(threads);

Iter st = fst;
for (size_t i = 0; i < (num_th - 1); ++i)
{
Iter en = st;
std::advance(en, block_size);
std::packaged_task<void(void)> tsk([=]() {std::for_each(st, en, f); });
futures[i] = tsk.get_future();
threads[i] = std::thread(std::move(tsk));
st = en;
}

std::for_each(st, lst, f);
for (size_t i = 0; i < (num_th - 1); ++i)
{
futures[i].get();
}
}
futures[i].get()④只是提供检索工作线程异常的方法；如果不想把异常传递出去，就可以省略这一步

使用  std::async  会简化代码

template<typename Iter, typename Func>
void parallel_for_each_async(Iter fst, Iter lst, Func f)
{
const size_t sz1 = std::distance(fst, lst);
if (!sz1)
return;

const size_t min_per_th = 25;
if (sz1 < (2 * min_per_th))
{
std::for_each(fst, lst, f);
}
else
{
const Iter md = fst + sz1 / 2;

std::future<void> fst_half = std::async(&parallel_for_each_async<Iter, Func>, fst, md, f);

parallel_for_each_async(md, lst, f);
fst_half.get();
}
}

8.5.2  并行实现：  std::find

std::find 算法，因为不需要对数据元素做任何处理的算法。比如，当第一个元素就满足查找标准，就没有必要对其他元素进行搜索了。将会看到，算法属性对于性能具有很大的影响，并且对并行实现的设计有着直接的影响。

std::equal
std::any_of

当得到想要的答案就中断其他任务的执行

中断其他线程的一个办法就是使用原子变量作为一个标识，处理过每一个元素后就对这个标识进行检查。
缺点就是，加载原子变量是一个很慢的操作，会阻碍每个线程的运行。
。。atomic_flag  慢，还是  shared_lock(bool)  慢，  感觉后者慢？

如何返回值和传播异常
可以使用一个期望值数组，使用 std::packaged_task 来转移值和异常
或者使用 std::promise 对工作线程上的最终结果直接进行设置

这完全依赖于想怎么样处理工作线程上的异常。如果想停止第一个异常(即使还没有对所有元素进行处理)，就可以使用 std::promise 对异常和最终值进行设置。另外，如果想让其他工作线程继续查找，可以使用 std::packaged_task 来存储所有的异常，当线程没有找到匹配元素时，异常将再次抛出。

。。代码在ctmf

8.5.3  并行实现：  std::partial_sum

有一个序列[1，2，3，4，5]，在执行该算法后会成为：[1，3(1+2)，6(1+2+3)，10(1+2+3+4)，15(1+2+3+4+5)]。

。。
第一次累加i-1，第二次累加i-2，第三次累加i-3。。
。。没有必要啊。  原先就是  O(n)的。  改成  O(nlogn)

对于处理单元较少的情况，第一种方法会比较合适；对于大规模并行系统，第二种方法比较合适。

。栅栏类。

第9章  高级线程管理

9.1  线程池

9.1.1  最简单的线程池

作为最简单的线程池，其拥有固定数量的工作线程(通常工作线程数量与 std::thread::hardware_concurrency() 相同)。
工作需要完成时，可以调用函数将任务挂在任务队列中。
每个工作线程都会从任务队列上获取任务，然后执行这个任务，执行完成后再回来获取新的任务。
最简单的线程池中，线程就不需要等待其他线程完成对应任务了。如果需要等待，就需要对同步进行管理。

class thread_pool
{
std::atomic_bool done;
thread_safe_queue<std::function<void()>> work_queue;
std::vector<std::thread> threads;
join_threads joiner;

void work_thread()
{
while (!done)
{
std::function<void()> tsk;
if (work_queue.try_pop(tsk))
{
tsk();
}
else
{
std::this_thread::yield();
}
}
}

public:
thread_pool() : done(false), joiner(threads)
{
const size_t hw_th = std::thread::hardware_concurrency();

try
{
for (size_t i = 0; i < hw_th; ++i)
{
threads.push_back(std::thread(&thread_pool::worker_thread, this));
}
}
catch (...)
{
done = true;
throw;
}
}

~thread_pool()
{
done = true;
}

template<typename FuncType>
void submit(FuncType f)
{
work_queue.push(std::function<void()>(f));
}
};

9.1.2  等待提交到线程池中的任务

条件变量和期望值

std::packaged_task<> 实例是不可拷贝的，仅是可移动的，所以不能再使用 std::function<> 来实现任务队列，因为 std::function<> 需要存储可复制构造的函数对象。包装一个自定义函数，用来处理只可移动的类型，就是一个带有函数操作符的类型擦除类。只需要处理没有入参的函数和无返回的函数即可，所以这是一个简单的虚函数调用。

std::result_of<> ： std::result_of<FunctionType()>::type 是FunctionType类型的引用实例(如f)，并且没有参数。同样，函数中可以对result_type typedef②使用  std::result_of<> 。

9.1.3  等待依赖任务

9.1.4  避免队列中的任务竞争

随着处理器的增加，任务队列上就会有很多的竞争，这会让性能下降

使用无锁队列会让任务没有明显的等待，但乒乓缓存会消耗大量的时间。

为了避免乒乓缓存，每个线程建立独立的任务队列。这样，每个线程就会将新任务放在自己的任务队列上，并且当线程上的任务队列没有任务时，去全局的任务列表中取任务。下面列表中的实现，使用了一个thread_local变量，来保证每个线程都拥有自己的任务列表(如全局列表那样)。

这样就能有效的避免竞争，不过当任务分配不均时，造成的结果就是：某个线程本地队列中有很多任务的同时，其他线程无所事事。
这个问题是有解：本地工作队列和全局工作队列上没有任务时，可从别的线程队列中窃取任务。

9.1.5  窃取任务

9.2  中断线程
使用信号来让未结束线程停止运行。

9.2.1  启动和中断线程

最起码需要和 std::thread 相同的接口，还要多加一个interrupt()函数：
class interruptible_thread
{
public:
template<typename FunctionType>
interruptible_thread(FunctionType f);
void join();
void detach();
bool joinable() const;
void interrupt();
};

类内部可以使用  std::thread  来管理线程，并且使用一些自定义数据结构来处理中断

为了使断点能够正常使用，就需要使用一个没有参数的函数：interruption_point()。这意味着中断数据结构可以访问thread_local变量，并在线程运行时，对变量进行设置，因此当线程调用interruption_point()函数时，就会去检查当前运行线程的数据结构

thread_local标志是不能使用普通的 std::thread 管理线程的主要原因；需要使用一种方法分配出一个可访问的interruptible_thread实例，就像新启动一个线程一样。使用已提供函数来做这件事情前，需要将interruptible_thread实例传递给 std::thread 的构造函数，创建一个能够执行的线程

9.2.2  检查线程是否中断
。运行开始时，检查  中断标记。

最好是在线程等待或阻塞的时候中断线程，因为这时的线程不能运行，也就不能调用interruption_point()函数！线程等待时，什么方式才能去中断线程呢？

9.2.3  中断等待——条件变量

等待条件变量的通知。就需要一个新函数——interruptible_wait()——就可以运行各种需要等待的任务，并且可以知道如何中断等待。

最简单的方式是，当设置中断标志时需要提醒条件变量，并在等待后立即设置断点。为了让其工作，需要提醒所有等待对应条件变量的线程，就能确保相应的线程能够苏醒。伪苏醒是无论如何都要处理的，所以其他线程(非感兴趣线程)将会被当作伪苏醒处理——两者之间没什么区别。interrupt_flag结构需要存储一个指针指向一个条件变量，所以用set()函数对其进行提醒。

void interruptible_wait(std::condition_variable& cv,
std::unique_lock<std::mutex>& lk)
{
interruption_point();
this_thread_interrupt_flag.set_condition_variable(cv); // 1
cv.wait(lk); // 2
this_thread_interrupt_flag.clear_condition_variable(); // 3
interruption_point();
}

std::condition_variable::wait() 可以抛出异常，所以这里会直接退出，而没有通过条件变量删除相关的中断标志。这个问题很容易修复，就是在析构函数中添加删除操作。

第二个问题不大明显，这段代码存在条件竞争。虽然，线程可以通过调用interruption_point()被中断，不过在调用wait()后，条件变量和相关中断标志就没有什么系了，因为线程不是等待状态，所以不能通过条件变量的方式唤醒。就需要确保线程不会在最后一次中断检查和调用wait()间被唤醒。

一个选择就是放置超时等待

9.2.4  使用  std::condition_variable_any  中断等待

9.2.5  中断其他阻塞调用

通常情况下，可以使用  std::condition_variable  的超时选项

9.2.6  处理中断

try
{
do_something();
}
catch(thread_interrupted&)
{
handle_interruption();
}

当异常传入 std::thread 的析构函数时，将会调用 std::terminate() ，并且整个程序将会终止。为了避免这种情况，需要在每个将interruptible_thread变量作为参数传入的函数中放置catch(thread_interrupted)处理块，可以将catch块包装进interrupt_flag的初始化过程中

因为异常将会终止独立进程，这样就能保证未处理的中断是异常安全的。interruptible_thread构造函数中对线程的初始化，实现如下：

internal_thread=std::thread([f,&p]{
p.set_value(&this_thread_interrupt_flag);
try
{
f();
}
catch(thread_interrupted const&)
{}
});

9.2.7  退出时中断后台任务

第10章  并行算法
使用C++17并行算法

10.1  并行化标准库算法

C++17为标准库添加并行算法。都是对之前已存在的一些标准算法的重载，例如： std::find ， std::transform 和 std::reduce 。并行版本的签名除了和单线程版本的函数签名相同之外，在参数列表的中新添加了一个参数(第一个参数)——指定要使用的执行策略

std::vector<int> my_data;
std::sort(std::execution::par,my_data.begin(),my_data.end());

std::execution::par 的执行策略，表示允许使用多个线程调用此并行算法。这是一种权限，而不是一个申请——如果需要，这个算法依旧可以以串行的方式运行。通过指定执行策略，算法的需求复杂性已经发生变化，并且要比串行版的算法要求要宽松。

10.2  执行策略
<execution>  头文件

三个标注执行策略：
std::execution::sequenced_policy
std::execution::parallel_policy
std::execution::parallel_unsequenced_policy

三个相关的策略对象可以传递到算法中：
std::execution::seq
std::execution::par
std::execution::par_unseq

10.2.1  使用执行策略的影响
执行策略的影响：
算法复杂化  。  提升
抛出异常时的行为  。  terminate or re-throw
算法执行的位置、方式和时间

10.2.2 std::execution::sequenced_policy

顺序策略对算法使用的迭代器、相关值和可调用对象没什么要求：他们可以自由的使用同步机制，并且可以依赖于同一线程上的所有操作，不过他们不能依赖这些操作的顺序。

10.2.3 std::execution::parallel_policy

在给定线程上执行需要按照一定的顺序，不能交错执行，但十分具体的顺序是不指定的；并且在不同的调用间，指定的顺序可能会不同

这就对算法所使用的迭代器、相关值和可调用对象有了额外的要求：想要并行调用，他们间就不能有数据竞争，也不能依赖于线程上运行的其他操作，或者依赖的操作不能在同一线程

上。

10.2.4 std::execution::parallel_unsequenced_policy
并行不排序策略提供了最大程度的并行化算法

使用并行不排序策略时，算法使用的迭代器、相关值和可调用对象不能使用任何形式的同步，也不能调用任何需要同步的函数。
也就是，必须对相关的元素或基于该元素可以访问的数据进行操作，并且不能修改线程之间或元素之前共享的状态。

10.3 C++标准库中的并行算法
大多数被执行策略重载的算法都在  <algorithm>  和  <numeric>头文件  中
all_of，any_of，none_of，for_each，for_each_n，find，find_if，find_end，find_first_of，
adjacent_find，count，count_if，mismatch，equal，search，search_n，copy，copy_n，
copy_if，move，swap_ranges，transform，replace，replace_if，replace_copy，
replace_copy_if，fill，fill_n，generate，generate_n，remove，remove_if，remove_copy，
remove_copy_if，unique，unique_copy，reverse，reverse_copy，rotate，rotate_copy，
is_partitioned，partition，stable_partition，partition_copy，sort，stable_sort，
partial_sort，partial_sort_copy，is_sorted，is_sorted_until，nth_element，merge，
inplace_merge，includes，set_union，set_intersection，set_difference，
set_symmetric_difference，is_heap，is_heap_until，min_element，max_element，
minmax_element，lexicographical_compare，reduce，transform_reduce，

exclusive_scan，inclusive_scan，transform_exclusive_scan，transform_inclusive_scan和

adjacent_difference

第11章  多线程程序的测试和调试

11.1  与并发相关的错误类型
通常有两大类：
不必要的阻塞
条件竞争

11.1.1 不必要的阻塞
死锁

活锁
与死锁的情况类似。不同的地方在于线程不是阻塞等待，而是在循环中持续检查

I/O阻塞或外部输入

11.1.2 条件竞争
数据竞争
因为未同步访问一块共享内存，将会导致代码产生未定义行为
破坏不变量

主要表现为悬空指针(因为其他线程已经将要访问的数据删除了)，随机存储错误(因为局部更新，导致线程读取了不一样的数据)，以及双重释放(比如：当两个线程对同一个队列同时执行pop操作，想要删除同一个关联数据)，等等

生命周期问题
线程会访问不存在的数据，这可能是因为数据被删除或销毁了，或者转移到其他对象中去了。

11.2  定位并发错误的技术

11.2.1  代码审阅——发现潜在的错误
审阅多线程代码需要考虑的问题
并发访问时，哪些数据需要保护？
如何确定访问数据受到了保护？
是否会有多个线程同时访问这段代码？
这个线程获取了哪个互斥量？
其他线程可能获取哪些互斥量？
两个线程间的操作是否有依赖关系？如何满足这种关系？
这个线程加载的数据还是合法数据吗？数据是否被其他线程修改过？
当假设其他线程可以对数据进行修改，这将意味着什么？并且，怎么确保这样的事情不会发生？

11.2.2  通过测试定位并发相关的错误

11.2.3  可测试性设计
如果代码满足一下几点，就很容易进行测试：
每个函数和类的关系都很清楚。
函数短小精悍。
测试用例可以完全控制被测试代码周边的环境。
执行特定操作的代码应该集中测试，而非分布式测试。
需要在完成编写后，考虑如何进行测试。

11.2.4  多线程测试技术

蛮力测试(或称压力测试)

组合仿真测试

使用专用库对代码进行测试

11.2.5  构建多线程测试代码

11.2.6  测试多线程代码性能

附录

=====================

=====================

=====================

已使用 Microsoft OneNote 2016 创建。