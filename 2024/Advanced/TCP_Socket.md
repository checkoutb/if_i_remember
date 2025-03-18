TCP/Socket
2023年4月8日
9:45

code

[[toc]]



。。今天想学下c++ socket，结果baidu了下，没有合意的(权威，入门)。
。。bing了一下，第一个就是 geeksforgeeks。

--------------------

https://www.geeksforgeeks.org/socket-programming-cc/

Socket Programming in C/C++

# What is socket programming?
socket编程是 在网络上连接2个node来通信的方法。
socket在IP的某个端口上监听，当其他socket到达(reach out)时，构建连接。
当客户端到达服务器时，服务器组建一个 listener socket。

## State diagram for server and client model
![7a9a01b0973cbec4534d6398030bc4a8.png](../_resources/7a9a01b0973cbec4534d6398030bc4a8.png)

## Stages for Server
### Socket creation
``` C++
int sockfd = socket(domain, type, protocol)
```

- sockfd： socker descriptor, an integer (like a file-handle)
- domain： integer, 指定通信的domain。我们使用 POSIX标准中定义的 AF_LOCAL 在同一主机的不同进程间通信。对于不同主机的进程的通信，如果是IPv4，则使用 AF_INET，如果IPv6，使用AF_INET6。
- type： 通信类型
	- SOCK_STREAM： TCP (可靠的，面向连接的)
	- SOCK_DRAM： UDP (不可靠的，无连接的)
- protocol：协议值，对于IP协议(internet protocol)，是0。这个值等同于 包中的IP header中的 protocol 域。

### Setsockopt
``` C++
int setsockopt(int sockfd, int level, int optname,  const void *optval, socklen_t optlen);
```

这用于 操作 文件描述符(file descriptor) sockdf 的 option
这是 完全可选的，但 有助于 重用 地址和端口。防止诸如 address already in use 的错误。

### Bind
``` C++
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

在 socket 创建后，bind函数 绑定 socket 到 addr指定的 地址和端口号。
在例子中，我们绑定到localhost，因为我们使用 INADDR_ANY 来指定 IP地址。
。。这个例子是指下面的。

### Listen
``` C++
int listen(int sockfd, int backlog);
```

这个方法将 服务器套接字 设置为 被动模式 (passive mode)，等待客户端发起 连接。
backlog，定义了 sockfd 的 pending connection 的queue的最大长度。当queue满时，客户端发出 connection request，client会收到一个错误： ECONNREFUSED。

### Accept
``` C++
int new_socket= accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

这个方法从 listening socket (即 sockfd) 的 pending connection queue 中 提取一个 connection request，然后创建一个 新的 connected socket，然后返回指向 这个 socket的 新的 file descriptor。
此时，client 和 server 端 连接 establish，它们已经准备好 传输数据了。


## Stages for Client

### Socket connection
和 server 的socket creation 一样

### Connect
``` C++
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

connect() 系统调用，将 sockfd 指向的socket 连接到 addr指定的地址。
server的地址和端口，在 addr中指定。


## Implementation
我们通过在 server 和client 间 交换 hello message 来说明 client/server model

### Server.c
``` C++
// Server side C/C++ program to demonstrate Socket
// programming
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>
#define PORT 8080
int main(int argc, char const* argv[])
{
	int server_fd, new_socket, valread;
	struct sockaddr_in address;
	int opt = 1;
	int addrlen = sizeof(address);
	char buffer[1024] = { 0 };
	char* hello = "Hello from server";

	// Creating socket file descriptor
	if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
		perror("socket failed");
		exit(EXIT_FAILURE);
	}

	// Forcefully attaching socket to the port 8080
	if (setsockopt(server_fd, SOL_SOCKET,
				SO_REUSEADDR | SO_REUSEPORT, &opt,
				sizeof(opt))) {
		perror("setsockopt");
		exit(EXIT_FAILURE);
	}
	address.sin_family = AF_INET;
	address.sin_addr.s_addr = INADDR_ANY;
	address.sin_port = htons(PORT);

	// Forcefully attaching socket to the port 8080
	if (bind(server_fd, (struct sockaddr*)&address,
			sizeof(address))
		< 0) {
		perror("bind failed");
		exit(EXIT_FAILURE);
	}
	if (listen(server_fd, 3) < 0) {
		perror("listen");
		exit(EXIT_FAILURE);
	}
	if ((new_socket
		= accept(server_fd, (struct sockaddr*)&address,
				(socklen_t*)&addrlen))
		< 0) {
		perror("accept");
		exit(EXIT_FAILURE);
	}
	valread = read(new_socket, buffer, 1024);
	printf("%s\n", buffer);
	send(new_socket, hello, strlen(hello), 0);
	printf("Hello message sent\n");

	// closing the connected socket
	close(new_socket);
	// closing the listening socket
	shutdown(server_fd, SHUT_RDWR);
	return 0;
}
```


### Client.c
```C++
// Client side C/C++ program to demonstrate Socket
// programming
#include <arpa/inet.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>
#define PORT 8080

int main(int argc, char const* argv[])
{
	int status, valread, client_fd;
	struct sockaddr_in serv_addr;
	char* hello = "Hello from client";
	char buffer[1024] = { 0 };
	if ((client_fd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
		printf("\n Socket creation error \n");
		return -1;
	}

	serv_addr.sin_family = AF_INET;
	serv_addr.sin_port = htons(PORT);

	// Convert IPv4 and IPv6 addresses from text to binary
	// form
	if (inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr)
		<= 0) {
		printf(
			"\nInvalid address/ Address not supported \n");
		return -1;
	}

	if ((status
		= connect(client_fd, (struct sockaddr*)&serv_addr,
				sizeof(serv_addr)))
		< 0) {
		printf("\nConnection Failed \n");
		return -1;
	}
	send(client_fd, hello, strlen(hello), 0);
	printf("Hello message sent\n");
	valread = read(client_fd, buffer, 1024);
	printf("%s\n", buffer);

	// closing the connected socket
	close(client_fd);
	return 0;
}
```

### Compiling
```
gcc client.c -o client
gcc server.c -o server
```

### Output
```
Client:Hello message sent
Hello from server
Server:Hello from client
Hello message sent
```




--------------------
https://www.geeksforgeeks.org/socket-programming-in-cc-handling-multiple-clients-on-server-without-multi-threading/

# Socket Programming in C/C++: Handling multiple clients on server without multi threading

在基础的 server-client 模型中，server 一次只处理一个client，如果你想开发一个可伸缩的server模型，这会是一个大的 assumption (假定，承担)。

最简单的 处理多个client 的方法是 为每个新连接生成一个新线程。这种方式非常不推荐，因为：
- 线程难以code，debug，有时产生无法预测的结果
- 上下文切换的开销
- 对于大量客户端，不是 scalable
- 会发生死锁

## select()
处理多client 的更好的方法是 使用 select() 这个linux 命令
- select 命令可以监控 多个 file descriptor，一直等待，直到 某个 file descriptor 变成 active。
- 例如，如果某个监控的 socket上 有数据可读，那么 select 会提供那个信息。
- select 的行为 就像 interrupt handler，只要任意 file descriptor 发送任意数据，那么它就会被 激活。

### select 使用的数据结构：fd_set
fd_set 包含了 file descriptor的list，用来监控是否 activity。
fs_set 关联了 4个函数：

``` C++
fd_set readfds;

// Clear an fd_set
FD_ZERO(&readfds);  

// Add a descriptor to an fd_set
FD_SET(master_sock, &readfds);   

// Remove a descriptor from an fd_set
FD_CLR(master_sock, &readfds); 

//If something happened on the master socket , then its an incoming connection  
FD_ISSET(master_sock, &readfds); 
```

### activating select
请阅读 select 的 man page 来检查 select 命令的所有参数
``` C++
activity = select( max_fd + 1 , &readfds , NULL , NULL , NULL);
```

## Implementation
``` C++
//Example code: A simple server side code, which echos back the received message.
//Handle multiple socket connections with select and fd_set on Linux
#include <stdio.h>
#include <string.h> //strlen
#include <stdlib.h>
#include <errno.h>
#include <unistd.h> //close
#include <arpa/inet.h> //close
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <sys/time.h> //FD_SET, FD_ISSET, FD_ZERO macros
	
#define TRUE 1
#define FALSE 0
#define PORT 8888
	
int main(int argc , char *argv[])
{
	int opt = TRUE;
	int master_socket , addrlen , new_socket , client_socket[30] ,
		max_clients = 30 , activity, i , valread , sd;
	int max_sd;
	struct sockaddr_in address;
		
	char buffer[1025]; //data buffer of 1K
		
	//set of socket descriptors
	fd_set readfds;
		
	//a message
	char *message = "ECHO Daemon v1.0 \r\n";
	
	//initialise all client_socket[] to 0 so not checked
	for (i = 0; i < max_clients; i++)
	{
		client_socket[i] = 0;
	}
		
	//create a master socket
	if( (master_socket = socket(AF_INET , SOCK_STREAM , 0)) == 0)
	{
		perror("socket failed");
		exit(EXIT_FAILURE);
	}
	
	//set master socket to allow multiple connections ,
	//this is just a good habit, it will work without this
	if( setsockopt(master_socket, SOL_SOCKET, SO_REUSEADDR, (char *)&opt,
		sizeof(opt)) < 0 )
	{
		perror("setsockopt");
		exit(EXIT_FAILURE);
	}
	
	//type of socket created
	address.sin_family = AF_INET;
	address.sin_addr.s_addr = INADDR_ANY;
	address.sin_port = htons( PORT );
		
	//bind the socket to localhost port 8888
	if (bind(master_socket, (struct sockaddr *)&address, sizeof(address))<0)
	{
		perror("bind failed");
		exit(EXIT_FAILURE);
	}
	printf("Listener on port %d \n", PORT);
		
	//try to specify maximum of 3 pending connections for the master socket
	if (listen(master_socket, 3) < 0)
	{
		perror("listen");
		exit(EXIT_FAILURE);
	}
		
	//accept the incoming connection
	addrlen = sizeof(address);
	puts("Waiting for connections ...");
		
	while(TRUE)
	{
		//clear the socket set
		FD_ZERO(&readfds);
	
		//add master socket to set
		FD_SET(master_socket, &readfds);
		max_sd = master_socket;
			
		//add child sockets to set
		for ( i = 0 ; i < max_clients ; i++)
		{
			//socket descriptor
			sd = client_socket[i];
				
			//if valid socket descriptor then add to read list
			if(sd > 0)
				FD_SET( sd , &readfds);
				
			//highest file descriptor number, need it for the select function
			if(sd > max_sd)
				max_sd = sd;
		}
	
		//wait for an activity on one of the sockets , timeout is NULL ,
		//so wait indefinitely
		activity = select( max_sd + 1 , &readfds , NULL , NULL , NULL);
	
		if ((activity < 0) && (errno!=EINTR))
		{
			printf("select error");
		}
			
		//If something happened on the master socket ,
		//then its an incoming connection
		if (FD_ISSET(master_socket, &readfds))
		{
			if ((new_socket = accept(master_socket,
					(struct sockaddr *)&address, (socklen_t*)&addrlen))<0)
			{
				perror("accept");
				exit(EXIT_FAILURE);
			}
			
			//inform user of socket number - used in send and receive commands
			printf("New connection , socket fd is %d , ip is : %s , port : %d
				\n" , new_socket , inet_ntoa(address.sin_addr) , ntohs
				(address.sin_port));
		
			//send new connection greeting message
			if( send(new_socket, message, strlen(message), 0) != strlen(message) )
			{
				perror("send");
			}
				
			puts("Welcome message sent successfully");
				
			//add new socket to array of sockets
			for (i = 0; i < max_clients; i++)
			{
				//if position is empty
				if( client_socket[i] == 0 )
				{
					client_socket[i] = new_socket;
					printf("Adding to list of sockets as %d\n" , i);
						
					break;
				}
			}
		}
			
		//else its some IO operation on some other socket
		for (i = 0; i < max_clients; i++)
		{
			sd = client_socket[i];
				
			if (FD_ISSET( sd , &readfds))
			{
				//Check if it was for closing , and also read the
				//incoming message
				if ((valread = read( sd , buffer, 1024)) == 0)
				{
					//Somebody disconnected , get his details and print
					getpeername(sd , (struct sockaddr*)&address , \
						(socklen_t*)&addrlen);
					printf("Host disconnected , ip %s , port %d \n" ,
						inet_ntoa(address.sin_addr) , ntohs(address.sin_port));
						
					//Close the socket and mark as 0 in list for reuse
					close( sd );
					client_socket[i] = 0;
				}
					
				//Echo back the message that came in
				else
				{
					//set the string terminating NULL byte on the end
					//of the data read
					buffer[valread] = '\0';
					send(sd , buffer , strlen(buffer) , 0 );
				}
			}
		}
	}
	return 0;
}
```

编译并运行。
使用 telnet 命令模仿 client 的连接。


### Code Explanation:
- 我们创建 fs_set 类型的变量 readfds，它会监控 main server listening socket 以及 所有的 client 的 active file descriptor。
- 当新client 想要连接时，master_socket 会被激活 并为这个client 打开一个 新的 fd。我们会保存它的 fd 到 我们的 client_list，并且在 下次迭代时，我们会把 它 添加到 readfds 以监控 这个client 的 活动
- 类似的，如果一个 老的client 发送了一些数据，readfds 会被激活，我们会检查 现有client 的 list 来 知道 哪个client 被发送了数据。

## Alternatives
还有其他的函数 能像 select 一样 执行任务： pselect，pool，ppoll。





--------------------
https://www.geeksforgeeks.org/error-control-in-tcp/


--------------------
https://zhuanlan.zhihu.com/p/75672631



--------------------

--------------------

--------------------

--------------------

--------------------


# Socket and TCP
https://www.sohu.com/a/645498691_120070959

Socket是 应用层 与 TCP/IP协议簇 通信的 ==中间软件抽象层，它是一组接口==。

![aa3d58cd7b728161f60833a7cf3ef930.png](../_resources/aa3d58cd7b728161f60833a7cf3ef930.png)

主机 A 的应用程序必须通过 Socket 建立连接才能与主机B的应用程序通信，
而建立 Socket 连接需要底层 TCP/IP 协议来建立 TCP 连接。
而建立 TCP 连接需要底层 IP 协议来寻址网络中的主机。

Socket 连接是计算机网络中的一种通信机制，它允许两个程序在不同计算机上通过网络进行通信。
在使用套接字进行通信时，一个程序作为客户端，另一个程序作为服务器端，它们通过创建和使用套接字进行数据传输。
我们可以将套接字理解为==网络通信的接口==，它提供了一种==标准的通信方式==，使得不同的程序能够在网络上进行数据交换。


在客户端中，我们需要创建一个套接字并指定连接目标的 IP 地址和端口号，然后向服务器端发送连接请求。
在服务器端中，我们需要创建一个套接字并绑定到一个指定的端口号上，然后等待客户端的连接请求。

![d7cffe4e6a08cd595de1b6110720ee4d.png](../_resources/d7cffe4e6a08cd595de1b6110720ee4d.png)

在 Socket 连接中，常见的协议有 TCP 和 UDP 两种。







# Socket 凤凰架构

![c398e19a03049f3f332362f78da79b55.png](../_resources/c398e19a03049f3f332362f78da79b55.png)




几乎整个网络栈（应用层以下）都位于系统内核空间之中，之所以采用这种设计，主要是从数据安全隔离的角度出发来考虑的。
由==内核去处理==网络报文的收发，无疑会有==更高的执行开销==，譬如数据在内核态和用户态之间来回拷贝的额外成本，因此会==损失一些性能==，但是能够保证应用程序无法窃听到或者去伪造另一个应用程序的通信内容。
针对特别关注收发性能的应用场景，也有==直接在用户空间中实现全套协议栈的旁路方案==，譬如开源的Netmap 以及 Intel 的DPDK ，都能做到零拷贝收发网络数据包。

图中传输模型的箭头展示的是数据流动的方向，它体现了信息从程序中发出以后，到被另一个程序接收到之前，将经历如下几个阶段：
- Socket
  应用层的程序是通过 Socket 编程接口来和内核空间的网络协议栈通信的。Linux Socket 是从 BSD Socket 发展而来的，现在 Socket 已经不局限于某个操作系统的专属功能，成为各大主流操作系统共同支持的通用网络编程接口，是网络应用程序实际上的交互基础。应用程序通过读写收、发缓冲区（Receive/Send Buffer）来与 Socket 进行交互，在 UNIX 和 Linux 系统中，出于“一切皆是文件”的设计哲学，对 Socket 操作被实现为对文件系统（socketfs）的读写访问操作，通过文件描述符（File Descriptor）来进行。
- TCP/UDP
  传输层协议族里最重要的协议无疑是传输控制协议 （Transmission Control Protocol，TCP）和用户数据报协议 （User Datagram Protocol，UDP）两种，它们也是在 Linux 内核中被直接支持的协议。此外还有流控制传输协议 （Stream Control Transmission Protocol，SCTP）、数据报拥塞控制协议 （Datagram Congestion Control Protocol，DCCP）等等。
  不同的协议处理流程大致是一样的，只是封装的报文以及头、尾部信息会有所不同，这里以 TCP 协议为例，内核发现 Socket 的发送缓冲区中有新的数据被拷贝进来后，会把数据封装为 TCP Segment 报文，常见网络协议的报文基本上都是由报文头（Header）和报文体（Body，也叫荷载“Payload”）两部分组成。系统内核将缓冲区中用户要发送出去的数据作为报文体，然后把传输层中的必要控制信息，譬如代表哪个程序发、由哪个程序收的源、目标端口号，用于保证可靠通信（重发与控制顺序）的序列号、用于校验信息是否在传输中出现损失的校验和（Check Sum）等信息封装入报文头中。
- IP
  IP：网络层协议最主要就是网际协议 （Internet Protocol，IP），其他还有因特网组管理协议 （Internet Group Management Protocol，IGMP）、大量的路由协议（EGP、NHRP、OSPF、IGRP、……）等等。
  以 IP 协议为例，它会将来自上一层（本例中的 TCP 报文）的数据包作为报文体，再次加入自己的报文头，譬如指明数据应该发到哪里的路由地址、数据包的长度、协议的版本号，等等，封装成 IP 数据包后发往下一层。
- Device
  网络设备（Device）是网络访问层中面向系统一侧的接口，这里所说的设备与物理硬件设备并不是同一个概念，Device 只是一种向操作系统端开放的==接口==，其背后既可能代表着真实的物理硬件，也可能是某段具有特定功能的程序代码，譬如即使不存在物理网卡，也依然可以存在回环设备（Loopback Device）。
  许多网络抓包工具，如tcpdump 、Wirshark 便是在此处工作的，前面介绍微服务流量控制时曾提到过的网络流量整形，通常也是在这里完成的。
  Device 主要的作用是抽象出统一的界面，让程序代码去选择或影响收发包出入口，譬如决定数据应该从哪块网卡设备发送出去；还有就是准备好网卡驱动工作所需的数据，譬如来自上一层的 IP 数据包、下一跳 （Next Hop）的 MAC 地址（这个地址是通过ARP Request 得到的）等等。
- Driver
  网卡驱动程序（Driver）是网络访问层中面向硬件一侧的接口，网卡驱动程序会通过DMA 将主存中的待发送的数据包复制到驱动内部的缓冲区之中。数据被复制的同时，也会将上层提供的 IP 数据包、下一跳 MAC 地址这些信息，加上网卡的 MAC 地址、VLAN Tag 等信息一并封装成为==以太帧== （Ethernet Frame），并自动计算校验和。对于需要确认重发的信息，如果没有收到接收者的确认（ACK）响应，那==重发==的处理也是在这里自动完成的。



从 Linux Kernel 2.4 版开始，内核开放了一套通用的、可供代码干预数据在协议栈中流转的过滤器框架。这套名为 Netfilter 的框架是 Linux 防火墙和网络的主要维护者 Rusty Russell 提出并主导设计的，它围绕网络层（IP 协议）的周围，埋下了五个钩子 （Hooks），每当有数据包流到==网络层==，经过这些钩子时，就会自动触发由内核模块注册在这里的回调函数，程序代码就能够通过回调来干预 Linux 的网络通信。笔者先将这五个钩子的名字与含义列出：
- PREROUTING：
  来自设备的数据包进入协议栈后立即触发此钩子。PREROUTING 钩子在进入 IP 路由之前触发，这意味着只要接收到的数据包，无论是否真的发往本机，都会触发此钩子。
  一般用于目标网络地址转换（Destination NAT，DNAT）。
- INPUT：
  报文经过 IP 路由后，如果确定是发往本机的，将会触发此钩子，
  一般用于加工发往本地进程的数据包。
- FORWARD：
  报文经过 IP 路由后，如果确定不是发往本机的，将会触发此钩子，
  一般用于处理转发到其他机器的数据包。
- OUTPUT：
  从本机程序发出的数据包，在经过 IP 路由前，将会触发此钩子，
  一般用于加工本地进程的输出数据包。
- POSTROUTING：
  从本机网卡出去的数据包，无论是本机的程序所发出的，还是由本机转发给其他机器的，都会触发此钩子，
  一般用于源网络地址转换（Source NAT，SNAT）。

![449cf89c7237b7683bfd31fb9446d84a.png](../_resources/449cf89c7237b7683bfd31fb9446d84a.png)


Netfilter 允许在同一个钩子处注册多个回调函数，因此向钩子注册回调函数时必须提供明确的优先级，以便触发时能按照优先级从高到低进行激活。
虽然现在看来 Netfilter 只是一些简单的事件回调机制而已，然而这样一套简单的设计，却成为了整座 Linux 网络大厦的核心基石，Linux 系统提供的许多网络能力，如数据包过滤、封包处理（设置标志位、修改 TTL 等）、地址伪装、网络地址转换、透明代理、访问控制、基于协议类型的连接跟踪，带宽限速，等等，都是在 Netfilter 基础之上实现的。

以 Netfilter 为基础的应用有很多，其中使用最广泛的毫无疑问要数 Xtables 系列工具，譬如iptables 、ebtables、arptables、ip6tables 等等。

以下列出了部分 iptables 预置的行为：
- DROP：直接将数据包丢弃。
- REJECT：给客户端返回 Connection Refused 或 Destination Unreachable 报文。
- QUEUE：将数据包放入用户空间的队列，供用户空间的程序处理。
- RETURN：跳出当前链，该链里后续的规则不再执行。
- ACCEPT：同意数据包通过，继续执行后续的规则。
- JUMP：跳转到其他用户自定义的链继续执行。
- REDIRECT：在本机做端口映射。
- MASQUERADE：地址伪装，自动用修改源或目标的 IP 地址来做 NAT
- LOG：在/var/log/messages 文件中记录日志信息。

iptables 内置了五张不可扩展的规则表（其中 security 表并不常用，很多资料只计算了前四张表），如下所列：
- raw 表：用于去除数据包上的连接追踪机制（Connection Tracking）。
- mangle 表：用于修改数据包的报文头信息，如服务类型（Type Of Service，ToS）、生存周期（Time to Live，TTL）以及为数据包设置 Mark 标记，典型的应用是链路的服务质量管理（Quality Of Service，QoS）。
- nat 表：用于修改数据包的源或者目的地址等信息，典型的应用是网络地址转换（Network Address Translation）。
- filter 表：用于对数据包进行过滤，控制到达某条链上的数据包是继续放行、直接丢弃或拒绝（ACCEPT、DROP、REJECT），典型的应用是防火墙。
- security 表：用于在数据包上应用SELinux，这张表并不常用。

以上五张规则表是具有优先级的：raw→mangle→nat→filter→security，也即是上面列举它们的顺序
在 iptables 中新增规则时，需要按照规则的意图指定要存入到哪张表中，如果没有指定，默认将会存入 filter 表。
此外，每张表能够使用到的链也有所不同























