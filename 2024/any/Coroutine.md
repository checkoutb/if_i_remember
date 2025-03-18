
2023-09-24 17:33

[[toc]]


# java 21

## virtual thread

https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html#GUID-DC4306FC-D6C1-4BCC-AECE-48C32C1A8DAA

虚拟线程是 轻量级线程，可以减少 编写，维护，调试 高吞吐量app 的工作量

> what is platform thread?

platform thread 被实现为 OS线程的 thin wrpper。
。。所以 java 的thread 和 OS 的thread  ==1对1== 的。

平台线程 在 它的底层的 OS 线程 上运行 java代码。
平台线程 在它的==整个生命周期中 都占用(capture) 它的OS线程==。
因此，可用的==平台线程数量 受限于 OS线程数量==。

通常 平台线程 有一个大的 thread stack 和 其它资源， 这些被==OS 管理==。
它们 可以运行 所有类型的 task，但是可能 是有限的资源。
。they are suitable for running all types of tasks but may be a limited resource.

> what is virtual thread?

和平台线程一样，虚拟线程 也是 java.lang.thread 的一个 实例。
但是 虚拟线程 没有被 绑定到 特定的OS线程。

虚拟线程 仍然是在 OS线程 上运行的。
但是，==当虚拟线程的代码执行一个 阻塞IO操作时，java runtime 挂起 虚拟线程 直到 它可以继续。 挂起的 虚拟线程的 OS thread 现在空闲了，可以执行其他的 虚拟线程。==
。。java runtime 管理的。

虚拟线程 的实现方式 和 虚拟内存类似。
为了模拟(simulate) 大量内存，OS 映射 虚拟地址空间 到 有限的 RAM上。
类似的，为了模拟 大量线程，java runtime 映射 大量的虚拟线程 到 少量的OS线程上。

和 平台线程 不同，虚拟线程 通常 有一个 浅(shallow) call stack，只执行 一个 http client call 或 一个 jdbc 查询。

虚拟线程 支持 thread-local 变量 和 可继承的 thread-local 变量， 但是 使用它们 要小心，因为 一个 JVM 可以支持 百万的 虚拟线程。

虚拟线程 适用于 大部分时间都block，等待IO操作完成 的 task。
不适用于 长时间运行的 CPU密集型 的操作。

。。文章 有 创建 虚拟线程 的 代码

```Java
Thread thread = Thread.ofVirtual().start(() -> System.out.println("Hello"));
thread.join();
```


```Java
Thread.Builder builder = Thread.ofVirtual().name("MyThread");
Runnable task = () -> {
    System.out.println("Running thread");
};
Thread t = builder.start(task);
System.out.println("Thread t name: " + t.getName());
t.join();
```


```Java
Thread.Builder builder = Thread.ofVirtual().name("worker-", 0);
Runnable task = () -> {
    System.out.println("Thread ID: " + Thread.currentThread().threadId());
};

// name "worker-0"
Thread t1 = builder.start(task);   
t1.join();
System.out.println(t1.getName() + " terminated");

// name "worker-1"
Thread t2 = builder.start(task);   
t2.join();  
System.out.println(t2.getName() + " terminated");
```


```Java
try (ExecutorService myExecutor = Executors.newVirtualThreadPerTaskExecutor()) {
    Future<?> future = myExecutor.submit(() -> System.out.println("Running thread"));
    future.get();
    System.out.println("Task completed");
    // ...
```


> Multithreaded Client Server Example

```Java
public class EchoServer {
    
    public static void main(String[] args) throws IOException {
         
        if (args.length != 1) {
            System.err.println("Usage: java EchoServer <port>");
            System.exit(1);
        }
         
        int portNumber = Integer.parseInt(args[0]);
        try (
            ServerSocket serverSocket =
                new ServerSocket(Integer.parseInt(args[0]));
        ) {                
            while (true) {
                Socket clientSocket = serverSocket.accept();
                // Accept incoming connections
                // Start a service thread
                Thread.ofVirtual().start(() -> {
                    try (
                        PrintWriter out =
                            new PrintWriter(clientSocket.getOutputStream(), true);
                        BufferedReader in = new BufferedReader(
                            new InputStreamReader(clientSocket.getInputStream()));
                    ) {
                        String inputLine;
                        while ((inputLine = in.readLine()) != null) {
                            System.out.println(inputLine);
                            out.println(inputLine);
                        }
                    
                    } catch (IOException e) { 
                        e.printStackTrace();
                    }
                });
            }
        } catch (IOException e) {
            System.out.println("Exception caught when trying to listen on port "
                + portNumber + " or listening for a connection");
            System.out.println(e.getMessage());
        }
    }
}
```


```Java
public class EchoClient {
    public static void main(String[] args) throws IOException {
        if (args.length != 2) {
            System.err.println(
                "Usage: java EchoClient <hostname> <port>");
            System.exit(1);
        }
        String hostName = args[0];
        int portNumber = Integer.parseInt(args[1]);
        try (
            Socket echoSocket = new Socket(hostName, portNumber);
            PrintWriter out =
                new PrintWriter(echoSocket.getOutputStream(), true);
            BufferedReader in =
                new BufferedReader(
                    new InputStreamReader(echoSocket.getInputStream()));
        ) {
            BufferedReader stdIn =
                new BufferedReader(
                    new InputStreamReader(System.in));
            String userInput;
            while ((userInput = stdIn.readLine()) != null) {
                out.println(userInput);
                System.out.println("echo: " + in.readLine());
                if (userInput.equals("bye")) break;
            }
        } catch (UnknownHostException e) {
            System.err.println("Don't know about host " + hostName);
            System.exit(1);
        } catch (IOException e) {
            System.err.println("Couldn't get I/O for the connection to " +
                hostName);
            System.exit(1);
        } 
    }
}
```


当 虚拟线程 被pin 到 carrier 上时，它无法被 解除unmount。
在下面场景中，虚拟线程 被pin 到 carrier上：
- 虚拟线程 执行 synchronized 块或方法 中的代码。
- 虚拟线程 执行 native方法 或 foreign 方法。

。synchronized，后面提到了，可以用下面代替。
```Java
lock.lock();
try {
	frequentIO();
} finally {
	lock.unlock();
}
```

pin 会降低吞吐量。



> virual thread guide

。。只是复制了标题。

- Write Simple, Synchronous Code Employing Blocking I/O APIs in the Thread-Per-Request Style
- Represent Every Concurrent Task as a Virtual Thread; Never Pool Virtual Threads
- Use Semaphores to Limit Concurrency
- Don't Cache Expensive Reusable Objects in Thread-Local Variables
- Avoid Lengthy and Frequent Pinning







# golang


https://golangdocs.com/goroutines-in-golang
。。这个不是原理。


。和 java差不多，也是 golang runtime 管理的

```Go
go funcName()
```

```Go
go func() {
	// ...
}
```




# C++20
https://en.cppreference.com/w/cpp/language/coroutines
写到 C/C++ 权威 中。这是 使用， 不是 java 那种的原理。

