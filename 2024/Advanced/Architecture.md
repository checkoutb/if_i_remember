Architecture
2021年9月1日
9:13

Netty多路复用

=========================================

Netty多路复用

服务端编程经常需要高性能的IO模型，常见的IO模型有4种：

|     |     |
| --- | --- |
| 同步阻塞IO(Blocking IO) | 即传统IO模型 |
| 同步非阻塞IO(Non-blocking IO) | 默认创建的socket都是阻塞的，非阻塞的要求socket被设置为NONBLOCK，这里的NIO并非Java的NIO(New IO)库 |
| IO多路复用(IO Multiplexing) | 经典的Reactor设计模式，有时也被称为  异步阻塞IO，Java中的Selector和Linux中的epoll  都是这种模型 |
| 异步IO(Asynchronous  IO) | 经典的Proactor设计模式，也称为  异步非阻塞IO |

同步，异步  描述的是  用户线程与  内核的交互方式：  同步是指用户线程发起IO请求后需要等待  或者  轮询内核IO操作完成后才能继续执行，异步是指用户线程发起IO请求后  继续执行，当内核IO操作完成后  通知用户线程，或者调用用户线程注册的回调函数。

阻塞，非阻塞  描述的是  用户线程调用内核IO操作的  方式：  阻塞是指IO操作需要彻底完成后才返回到用户空间，而非阻塞是指IO操作被调用后立即返回给用户一个状态值，无需等到IO操作彻底完成。

。。感觉是  同步异步  是描述  用户线程  和  系统线程的  关系，  阻塞，非阻塞  是描述  系统线程怎么返回的。

同步阻塞IO
用户线程  在内核进行IO操作时  被阻塞。

![计算机生成了可选文字: 用户纟踅彳呈 read讠青求 彳呈阝且塞 read完成 内才亥 等彳寺喽攵据到达 喽攵据扌考贝](../_resources/3bc759a01d7e4d58a76e6bced9b4e8ae.png)

{
read(socket, buffer);
process(buffer);
}

用户在等待时，不能做其他事情，导致CPU利用率不够。

同步非阻塞IO
在同步阻塞IO的基础上，将socket设置为NONBLOCK。这样，用户线程可以在发起IO请求后立刻返回。

![计算机生成了可选文字: 用户纟踅彳呈 read讠青求1 立即返回 礻呈车询 read讠青求n read完成 内才亥 等彳寺喽攵据到达 喽攵据扌考贝](../_resources/cffd9df8a95f43c1b3198bfac4a20fbb.png)

{
while(read(socket, buffer) != SUCCESS)
;
process(buffer);
}

用户线程需要不断地发起IO请求(调用read方法)，直到数据到达后，才真正读到数据。
每次发起IO请求都立即返回，但是为了等待数据，仍然需要不断轮询，重复请求，消耗大量CPU，一般很少直接用这种模型。

IO多路复用
建立在内核提供的多路分离函数select基础之上的，使用select函数可以避免同步非阻塞IO模型中  轮询等待  的问题。

![计算机生成了可选文字: 用户纟呈 监礻见socket 纟彳呈阝且塞 se工ec乜 socket可讠卖 了ead讠青求 read完成 内才亥 等彳寺喽攵扌居到达 喽攵据扌考贝](../_resources/0689d54b7ef74ee38c5b5f8859be4afd.png)

用户先将需要进行IO操作的socket添加到select中，然后阻塞等待select系统调用返回。当数据到达时，socket被激活，select函数返回，用户线程正式发起read请求，读取数据并继续执行。

从流程上来看，使用select函数进行IO请求和同步阻塞模型没有太大区别，甚至还多了添加监视socket，以及调用select函数的  额外操作，效率更差。

但是使用select以后最大的优势是用户可以在一个线程内同时处理多个socket的IO请求。用户可以注册多个socket，然后不断地调用select读取被激活的socket，即可达到在  同一个线程内同时处理多个IO请求的目的。而在同步阻塞模型中，必须通过多线程的方式才能达到这个目的。

伪代码描述：
 {
select(socket);
while(1) {
sockets = select();
for(socket in sockets) {
if(can_read(socket)) {
read(socket, buffer);
process(buffer);
}
}
}
}

while循环前将socket添加到select监视中，然后再while内一直调用select获取被激活的socket，一旦socket可读，便调用read函数将socket中数据读取出来。

使用select函数的优点并不仅限于此。  虽然上述方式允许单线程内处理多个IO请求，但是每个IO请求的过程还是阻塞的(在select函数上阻塞)，平均时间甚至比同步阻塞IO模型还要长。如果用户线程只注册自己感兴趣的socket或IO请求，然后去做自己的事情，等待数据到来时再进下处理，则可以提高CPU利用率

IO多路复用模型使用了Reactor设计模式实现了这一机制

![计算机生成了可选文字: Reactor ande_events() register_event() removeevent SynchronousEvent Demutiplexer elect() 1 Handle EventHandler andle—event() ethandle( Concrete EventHandlerA andle—event() et—handle Concrete EventHandlerB andle—event() et—handle](../_resources/df23b56852324056b49950389f0c7778.png)

如图所示，EventHandler抽象类表示IO事件处理器，它拥有IO文件句柄Handle(通过get_handle获得)，以及对Handle的操作handle_event(读/写等)。

继承于EventHandler的子类可以对事件处理器的行为进行定制。

Reactor类用于管理EventHandler(注册，删除等)，并使用handle_events  实现事件循环，不断调用同步事件多路分离器(一般是内核)的多路分离函数select，只要某个文件句柄被激活(可读/写等)，select就返回，handle_events  就会调用与文件句柄关联的事件处理器的handle_event进行相关操作。

![计算机生成了可选文字: 用户纟呈 Reactor 氵主册事亻午处里 纟彳呈车询 口用户纟彳呈 了ead讠青求 read完成 se工ec乜 socket可讠卖 内才亥 等彳寺喽攵扌居到达 喽攵据扌考贝](../_resources/9345b575502e4640b3f2e66b29c99276.png)

如上所示，通过Reactor的方式，可以将用户线程轮询IO操作状态的工作统一交给handle_events事件循环进行处理。
用户注册事件处理器之后可以继续执行其他的工作(异步)，而Reactor线程负责调用内核的select函数检查socket状态。
当有socket被激活时，通知相应的用户线程(或执行用户线程的回调函数)，执行handle_event进行数据读取，处理的工作。

由于select函数是阻塞的，因此多路IO复用模型也被称为异步阻塞IO模型。这里所说的阻塞是指select函数执行时线程被阻塞，而不是指socket。一般在使用IO多路复用模型时，socket都被设置为NONBLOCK的，不过这并不会产生影响，因为用户发起IO请求时，数据已经到达了，用户线程一定不会被阻塞。

用户线程使用IO多路复用模型的伪代码描述为：
void UserEventHandler::handle_event() {
if(can_read(socket)) {
read(socket, buffer);
process(buffer);
}
}
{
Reactor.register(new UserEventHandler(socket));
}

用户需要重写EventHandler的handle_event函数来进行读取数据，处理数据的工作，用户线程只需要将自己的EventHandler注册到Reactor即可。Reactor中handle_events事件循环的伪代码：

Reactor::handle_events() {
while(1) {
sockets = select();
for(socket in sockets) {
get_event_handler(socket).handle_event();
}
}
}
事件循环不断地调用select获取  被激活的socket，然后根据socket获取对应的EventHandler，执行handler_event函数。

IO多路复用是最常用的IO模型，但是其异步程序还不够彻底，因为它使用了会阻塞线程的select系统调用。所以它只能被称为  异步阻塞IO，不是真正的异步IO

异步IO
真正的异步IO操作  需要操作系统更强的支持。
在IO多路复用模型中，事件循环将文件句柄的状态事件通知给用户线程，由用户线程自行读取数据，处理数据。
而在异步IO模型中，当用户线程收到通知时，数据已经被内核读取完毕，并放在了用户线程指定的缓冲区内，内核在IO完成后通知用户线程直接使用即可。

异步IO模型使用Proactor设计模式  实现这一机制

![计算机生成了可选文字: Client <<SpecifY>> V Asynchronous Operation execute() <<exeute>> <<SpecifY>> V Proactor ae_events <<dispatchnotification>> <<SpecifY>> ton Handler andle_event( <<delegate» Asynchronous 《OperationProcessor egister() Hand CompletionHandIerA andle_event() CompletionHandIerg andle_event()](../_resources/16cfb6fe8b0744d683d9a903c2411ba3.jpg)

Proactor模式和Reactor模式  在结构上比较相似，不过在用户(Client)使用方式上差别较大。
Reactor模式中，用户线程通过向Reactor对象注册感兴趣的事件监听，然后事件触发时调用事件处理函数。

在Proactor模式中，用户线程将  Asynchronous Operation(读/写等)，Proactor以及操作完成时的CompletionHandler注册到AsynchronousOperationProcessor。AOP使用Facade模式提供了一组异步操作API(读/写等)供用户使用，当用户线程调用异步API后，便继续执行自己的任务。AOP会开启独立的内核线程执行异步操作，实现真正的异步。异步IO操作完成时，AOP将用户线程与AsynchronousOperation一起注册的Proactor  和CompletionHandler  取出，然后将CompletionHandler  与IO操作的结果数据  一起转发给  Proactor，Proactor负责回调每一个异步操作的书剑完成处理函数handle_event。虽然Proactor模式中的每一个异步操作  都可以绑定一个Proactor对象，但是一般在操作系统中，Proactor被实现为单例模式，以便于集中化分发操作完成事件。

![计算机生成了可选文字: 用户纟呈 Proactor 异步ead讠青求 内才亥 等彳寺喽攵扌居到达 喽攵扌居扌考贝 通。用户纟芄彳呈 rea 喽攵据分发 完成](../_resources/5edb7ab38bbf4301b05a0917ed8413b8.png)

异步IO模型中，用户线程直接使用内核提供的异步IO  api发起read请求，且发起后立刻返回，继续执行用户线程代码。  不过此时用户线程已经将调用的AsynchronousOperation  和  CompletionHandler注册到内核，然后操作系统开启独立的内核线程去处理IO操作。当read请求的数据到达时，由内核负责读取socket中的数据，并写入用户指定的缓冲区中。最后内核将read的数据和  用户线程注册的CompetionHandler分发给内部Proactor，Proactor将IO完成的信息通知给用户线程(一般通过调用用户线程注册的完成事件处理函数)，完成异步IO。

用户线程使用异步IO模型的伪代码：

void UserCompletionHandler::handle_event(buffer) {
process(buffer);
}
{
aio_read(socket, new UserCompletionHandler);
}

用户需要重写CompletionHandler的handle_event函数进行处理数据的工作，参数buffer表示Proactor已经准备好的数据，用户线程直接调用内核提供的异步IO api，并将重写的CompletionHandler注册即可。

异步IO并不常见，  IO多路复用+多线程任务  的架构基本满足了  高性能并发。

=========================================

=========================================

=========================================

=========================================

=========================================

=========================================

=========================================

=========================================

=========================================

已使用 Microsoft OneNote 2016 创建。