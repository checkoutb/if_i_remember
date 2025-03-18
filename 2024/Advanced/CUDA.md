CUDA
2023年4月8日
10:10

已使用 Microsoft OneNote 2016 创建。

CUDA vs OpenCL


《CUDA C编程权威指南》

Compute Unified Device Architecture


[[toc]]



# CUDA编程(个人博文)

https://zhuanlan.zhihu.com/p/548300959
这个的最下面

## 总览

在CPU上编程，无需关注底层CPU，因为高级编程语言和硬件分离。

在GPU上编程，你很难绕开底层，比如，不同的SM数量，就会影响你的程序运行结果。

### CPU，GPU 区别

GPU 和 CPU一样，有自己的内存，寄存器，缓存。

可以简单把GPU 理解为 超多核 的CPU

|GPU|CPU|
|--|--|
|多核|少数核|
|单个核计算能力弱|单个核计算能力强|
|适合计算密集型|适合控制密集型|
|线程轻量，切换开销小|线程重量，切换开销大|

应该把计算密集型任务放到GPU上处理，复杂逻辑控制分支语句放在CPU上。这种CPU+GPU的生产模式我们称为==异构并行计算架构==！

4核CPU一般同时运行不会超过32个线程，而GPU可以同时运行成千上万个线程，并且线程切换开销极小。

。。baidu了下，GPU里面的核的性能很低，而且不完整。
。。stream processor， stream multiprocessor。
。。H100，18432个核心。。 力大砖飞，不，人多力量大。 价格 4w美元。

把GPU称为设备端，把CPU称为主机端，他们两者通过PCle 总线相连。PCle的传输速度可达每秒几GB。

### CUDA源文件如何编译

nvcc 是CUDA的编译器，它会==自动分离=运行在GPU上的代码 和 运行在CPU上的代码。主机代码用C写的，那么就使用C语言编译器编译；设备端代码，也就是核函数，用CUDA C编写，那么使用nvcc编译。链接阶段，在内核程序调用GPU设备，添加运行时库。

CUDA C编程的文件后缀名为.cu和.cuh，分别是源代码文件和头文件。可以直接使用nvcc -o hello hello.cu 对源代码进行编译。


## 首次尝试

一个典型的 CUDA 程序流程 如下：
1. 把数据从 CPU内存 拷贝到 GPU内存
2. 调用 核函数 对存储在 GPU内存中的数据进行操作
3. 将数据从 GPU内存 传送回到 CPU内存

### 内存管理

用于执行GPU内存分配的是 `cudaMalloc` 函数，原型：
`cudaError_t cudaMalloc( void** dev_ptr, size_t size)`

|标准C函数|CUDA C函数|
|--|--|
|malloc|cudaMalloc|
|memcpy|cudaMemcpy|
|memset|cudaMemset|
|free|cudaFree|


值得注意的是 `cudaMemcpy` 函数，它负责 主机和设备 之间的数据传输，原型：
`cudaError_t cudaMemcpy( void** dst, const void* src, size_t count, cudaMemcpyKind kind)`

kind 参数设定 复制的方向：
- cudaMemcpyHostToHost
- cudaMemcpyHostToDevice
- cudaMemcpyDeviceToHost
- cudaMemcpyDeviceToDevice

注意，cudaMemcpy 是 同步执行的，即 CPU 会等待 copy 完成。 内存拷贝会占据大量时间。(后面会讲优化)

函数的返回值类型 cudaError_t 是一个枚举，成功时是 cudaSuccess, 失败时 cudaErrorMemoryAllocation。 
可以通过 `char* cudaGetErrorString(cudaError_t error)` 将错误转换为 可读的错误消息

GPU 的内存可以分为 **全局内存** ， **共享内存**， 前者类似CPU 的系统内存，后者类似 CPU 的 cache。 不过 GPU的 共享内存 是可以编程控制的， CPU的cache 不行。


### 线程管理

GPU 的线程数量非常多，成千上万， 单一层次的 线程管理不太理想，所以 我们需要 多层次化的结构来管理线程。

首先，OS中一个进程里有 多个线程。GPU进程 通常是通过 CPU 创建的，也就是说， 每个 GPU进程 都对应着 一个 CPU进程。 具有代表性的就是 核函数 kernel， 当调用 核函数kernel 时，其实就是创建了一个 GPU进程。 在GPU 上可以同时 运行多个 GPU进程 或者说 核函数。

在软件层面，逻辑层面上：
- 一个 GPU 进程 或者说 核函数 kernel 对应着 一个 线程网格 Grid
- 一个线程网格 Grid 可以被分为 多个线程块 block
- 一个线程块block 可以被分为 多个线程 thread

Grid,block,thread 组成了 软件层面上的 层次化管理。

### 核函数

#### 定义

在函数前加上 `__glob__` 修饰， 它是一个 函数类型限定符， 以数组加法为例：

```C++
__global__ void SumArraysCuda(float*a,float*b,float*res) // 定义了一个核函数，实现数组加法
{
  int i=threadIdx.x; // 后续介绍
  res[i]=a[i]+b[i];
} 
```

CUDA 一共有3种 限定符
- `__device__`，表示从 GPU上调用，在GPU上执行。也就是说，它可以被 `__global__`，或 `__device__` 修饰的函数 调用 ( 因为 `__global__` 修饰的函数 在 GPU上执行，此时就可以调用 `__device__`)。 这个 限定修饰符的 函数使用 有限制，比如在 G80/GT200 架构上 不能使用 递归，不能使用 函数指针 等。
- `__global__`，表示在 CPU上调用，在 GPU上执行，也就是所谓的 ==内核(kernel) 函数==，内核 只能被 主机调用
- `__host__`，表示 CPU上调用，CPU上执行， 这是默认的，是C语言的函数。


#### 调用

在调用核函数时 加上 `<<< >>>` ， 这是一个 配置运算符， 用来传递 核函数的 执行参数。

运算符有 4个参数：
- 第一个声明 网格的形状
- 第二个声明 块的形状
- 第三个声明 共享内存大小，默认0
- 第四个声明 执行流，默认0

```C++
int nElem = 32; // 每个数组的元素个数
int nByte = sizeof(float) * nElem; // 每个数组的所占内存

float *a_d,*b_d,*res_d;
// 在GPU上为数组分配内存
cudaMalloc((float**)&a_d,nByte));
cudaMalloc((float**)&b_d,nByte));
cudaMalloc((float**)&res_d,nByte)); // 存放结果

// 省略初始化数组

dim3 block(nElem);
dim3 grid(nElem/block.x);
// 调用核函数，该核函数包含(1,1,1)维的线程块，每一线程块包含(32,1,1)维的线程
SumArraysCuda<<<grid,block>>>(a_d,b_d,res_d); 
```

调用核函数时，需要指定该核函数的**线程组织形式**，通过配置运算符`<<< >>>`的两个参数，grid和block，都是dim3类型（CUDA内置的整数型向量），dim3包括三个整数，表示三个维度，可以通过通过它的x、 y、 z字段获得。当定义一个dim3类型的变量时， 所有未指定的元素都被初始化为1，比如dim3 item(10)，那么item.x=10，item.y和item.z都是1。

至此我们明白了`<<<grid,block>>>`中，grid表示线程**网格包含的线程块**以(grid.x，grid.y，grid.z)三维形式组织，block表示每一**线程块包含的线程**以 (block.x，block.y，block.z)三维形式组织。`<<<grid,block>>>`也可以接受整数类型，会把整数类型转换为y，z为1的dim3类型。

核函数的具体实现：
```C++
// 输入数组长度为32
// 通过SumArraysCuda<<<1,32>>>的方式调用
// 说明该核函数包含(1,1,1)维的线程块，每一线程块包含(32,1,1)维的线程
__global__ void SumArraysCuda(float*a,float*b,float*res) 
{
  int i=threadIdx.x; // threadIdx是线程块内的线程索引
  res[i]=a[i]+b[i];
} 
```

关键是**理解threadIdx起到什么作用**？答：使用线程索引建立数组索引。threadIdx（线程块内的线程索引）和blockIdx（线程块在线程网格内的索引）是核函数中需要预初始化的内置变量。 当执行一个核函数时， CUDA运行时为每个线程分配blockIdx和threadIdx。它们都是unit3整数向量类型，跟dim3类型类似，也包含3个无符号整数， 可以通过x、 y、 z三个字段来获得。

因此上面代码中总共有**32个线程同时并行**执行，每一个线程取得数组中的一对元素进行相加。 比起CPU的串行提速不少。

核函数还预初始化了两个内置变量blockDim（线程块的维度范围， 用每个线程块中的线程数来表示）和gridDim（线程格的维度范围， 用每个线程格中的线程数来表示）。它们都是dim3类型。

#### 完整代码

最后呢我们加上错误处理，给出完整的数组加法cuda代码：
```C++
// freshman.h头文件
#ifndef FRESHMAN_H
#define FRESHMAN_H
// 检查返回结果cudaError_t是否成功
#define CHECK(call)\   
{\
  const cudaError_t error=call;\
  if(error!=cudaSuccess)\
  {\
      printf("ERROR: %s:%d,",__FILE__,__LINE__);\
      printf("code:%d,reason:%s\n",error,cudaGetErrorString(error));\
      exit(1);\
  }\
}
#endif

// SumArrays.cu源代码文件
#include <cuda_runtime.h> // cuda相关头文件
#include <stdio.h>
#include "freshman.h"

void initialData(float* ip,int size)
{
  time_t t;
  srand((unsigned )time(&t));
  for(int i=0;i<size;i++)
  {
    ip[i]=(float)(rand()&0xffff)tf("\n");
  }
}

__global__ void sumArraysCuda(float*a,float*b,float*res)
{
  int i=threadIdx.x;
  res[i]=a[i]+b[i];
}
int main(int argc,char **argv)
{
  int dev = 0;
  cudaSetDevice(dev); // 当有多个GPU时，选定cuda设备，默认是0即第一个主GPU，多GPU时0,1,2以此类推

  int nElem=32;
  printf("Vector size:%d\n",nElem);
  int nByte=sizeof(float)*nElem;
  float *a_h=(float*)malloc(nByte);
  float *b_h=(float*)malloc(nByte);
  float *res_h=(float*)malloc(nByte);
  memset(res_h,0,nByte);

  float *a_d,*b_d,*res_d;
  CHECK(cudaMalloc((float**)&a_d,nByte));
  CHECK(cudaMalloc((float**)&b_d,nByte));
  CHECK(cudaMalloc((float**)&res_d,nByte));

  initialData(a_h,nElem); 
  initialData(b_h,nElem);

  CHECK(cudaMemcpy(a_d,a_h,nByte,cudaMemcpyHostToDevice)); // 从主机内存拷贝到设备内存
  CHECK(cudaMemcpy(b_d,b_h,nByte,cudaMemcpyHostToDevice)); 

  dim3 block(nElem);
  dim3 grid(nElem/block.x);
  sumArraysCuda<<<grid,block>>>(a_d,b_d,res_d);
  printf("Execution configuration<<<%d,%d>>>\n",block.x,grid.x);

  CHECK(cudaMemcpy(res_h,res_d,nByte,cudaMemcpyDeviceToHost)); // 将结果从设备内存拷贝到主机内存！
  cudaFree(a_d);
  cudaFree(b_d);
  cudaFree(res_d);

  free(a_h);
  free(b_h);
  free(res_h);

  return 0;
}
```

代码流程：把数据从CPU内存拷贝到GPU内存；调用核函数对存储在GPU内存中的数据进行操作；将数据从GPU内存传送回到CPU内存。

最后注意核函数是异步执行的，也就是GPU执行，CPU也在执行，可以通过cudaDeviceSynchronize();方法显式同步。


### 设备管理

学会如何查询GPU设备信息是很重要的， 因为在运行时你可以使用它来**帮助设置内核执行配置**。有两种方法查询和管理GPU设备：
- CUDA运行时API函数
- NVIDA系统管理界面( nvidia-smi) 命令行

#### CUDA运行时API

> cudaError_t cudaSetDevice(int dev)  
> 设置当前GPU设备  
> 当有多个GPU时，选定cuda设备，默认是0即第一个主GPU，多GPU时0,1,2以此类推。  

> cudaError_t cudaGetDeviceProperties(struct cudaDeviceProp* prop,int dev)  
> 返回设备信息  
> 以*prop形式返回设备dev的属性。struct cudaDevicePro结构体内定义了很多记录GPU设备信息的数据成员，比如multiProcessorCount表示设备上多处理器的数量。详见NVIDIA官网

> cudaError_t cudaGetDeviceCount( int* count )  
> 返回具体计算能力的设备数量  
> 以 *count 形式返回可用于执行的计算能力大于等于 1.0 的设备数量。

有多GPU设备时，我们可以根据multiProcessorCount设备的多处理器的数量，来选择最优的GPU。

```C++
int numDevices = 0 ;
cudaGetDeviceCount(&numDevices);
if (numDevices > 1) {
	int maxMultiprocessors = 0,
	maxDevice = 0 ;
	for (int device=0; device<numDevices; device++){
		cudaDeviceProp props;
		cudaGetDeviceProperties(&props, device);
		if (maxMultiprocessors < props.multiProcessorCount){
			maxMultiprocessors = props.multiProcessorCount;
			maxDevice = device;
		}
	}
	cudasetDevice(maxDevice);
}
```

## 组织并行线程

如何组织线程形式，关键是理解下面例子中Block索引和thread索引是如何对应到矩阵索引！

在软件层面上
- 一个GPU进程或者说核函数kernel对应着一个线程网格Grid；
- 一个线程网格Grid可以被组织成多个线程块Block；
- 一个线程块Block可以被组织成多个线程thread。

准确来说一个Grid由3维Block组成，一个Block由3维thread组成。

### 二维Grid 和 二维Block 与 二维矩阵的对应关系

以 `8*6` 的二维矩阵 为例，二维矩阵在内存中是以 一维线性的数组形式存放，也就是说`mat[ix][iy]` 对应 `array[iy*nx+ix]`


#### 线程如何与矩阵坐标对应？

1 通过 线程索引 和 块索引 映射到 矩阵坐标

```C++
ix=threadIdx.x+blockIdx.x*blockDim.x;
iy=threadIdx.y+blockIdx.y*blockDim.y;
```

2 再将 矩阵坐标 映射到 内存的 一维线性存储单元中  
`int idx=iy*nx+ix;`


##### Grid和Block的不同形状对核函数运行时间的影响。

我们设置两个较大的矩阵进行相加nx=1<<14; ny=1<<14; 尝试不同形状的线程组织形式
|核函数配置|运行时间|
|--|--|
|(512,512),(32,32)|0.0603|
|(512,1024),(32,16)|0.0380|
|(1024,1024),(16,16)|0.0455|
|(64,16384),(256,1)|0.0307|

这是 《CUDA C编程权威指南》书上的结果，在不同显卡上结果很有可能大不相同，总之，不同的线程组织配置对运行时间有很大影响。运行时间将受GPU当前可用内存、SM数量、wrapsize等影响。没有一个公式可以直接告诉我们最优的线程组织配置，先抛出==**一些经验**==：
- 一般Block中的 thread 数量要为 wrapsize 的整数倍
- 一般Grid中的 Block 数量要为GPU的 SM 数量的整数倍
- 一般在一定范围内Block的 数量越多 并行越多（但也太多会适得其反）

具体原因后面学习到GPU的底层时，自会明白。

此外Grid中Block的数量和Block中的thread数量都是有限制的：
- 计算能力为 1.x，Grid支持的只有 1D 和 2D，2.x才支持 3D 。
- block 则在 1.x 和 2.x 都支持3D , 在1.x / 2.x+的情况下, block的shape上限分别是`[512,512,64] / [1024,1024,64]` ，现代显卡的Grid的shape可以到`[2^31-1，65535，65535]`。


#### 如何对核函数进行计时

- 使用gettimeofday()
- nvprof

C语言的 clock() 不可以


使用gettimeofday
```C++
#include <sys/time.h>
double cpuSecond()
{
  struct timeval tp;
  gettimeofday(&tp,NULL);
  return((double)tp.tv_sec+(double)tp.tv_usec*1e-6);
}
```
gettimeofday是linux下的一个库函数，创建一个cpu计时器，从1970年1月1日0点以来到现在的秒数，需要头文件sys/time.h。

在使用时还需要注意，要用cudaDeviceSynchronize();来同步CPU和GPU，确保核函数已经执行完毕。

```C++
  double iStart,iElaps;
  iStart=cpuSecond();
  sumArraysGPU<<<grid,block>>>(a_d,b_d,res_d,nElem);
  cudaDeviceSynchronize();
  iElaps=cpuSecond()-iStart;
```

nvprof
自CUDA 5.0以来， NVIDIA提供了一个名为nvprof的命令行分析工具，功能十分强大。可以直接运行命令：nvprof 执行文件 ，会输出核函数运行时间以及拷贝内存的时间。
。。界面类似 jstat， 命令行命令。


## 硬件线程组织结构
要写好cuda程序，对GPU的硬件一定要深入了解。

GPU可以简单理解为多核的CPU，想要高效地管理多核，必须要层次化！下面先以Fermi架构为例，讲述GPU硬件的层次。**Fermi**架构是第一个完整的GPU计算架构， 能够为大多数高性能计算应用提供所需要的功能。

。。Fermi，2010年3月27日英伟达发布的一个显卡架构！

先回忆一下软件层的层次化：Grid、Block、thread。  
下面内容不仅将介绍硬件层的层次化，还会讲解硬件层与软件层的**对应关系**。  
注意本文不涉及GPU存储系统


### 第一层：设备

![1862d26680cc4321f2fc8ae587a1aefe.png](../_resources/1862d26680cc4321f2fc8ae587a1aefe.png)

上图中，紫色框框柱的就是一个 SM ( 流式多处理器)

通过上面的 设备结构图，不难发现 GPU 设备是一大堆 SM 组成的 阵列。比如，Fermi 有 16个SM。 此外还包括 DRAM 组成的 全局机载内存，Giga Thread 引擎 ( 一个全局调度器，用来分配 线程块 到 SM 上)

当调用一个内核函数，比如 执行配置为 `<<<10,32>>>`，由Giga Thread 引擎 负责把 10个线程块 分配到 SM上， Giga Thread 将根据 SM的资源占用情况 来进行分配，注意：
- 一个线程块 只能分配到 一个 SM上
- 多个线程块 可以分配到 同一个 SM上
- 提高并行性的一个 方法是，尽可能将线程块 均分到 SM上

最后一点，很好理解，想要提高并行性，就是尽可能压榨剥削GPU，有SM空闲不是我们希望看到的，把线程块均分到SM可以使得并行性最大化，所以得出一个结论，线程网格中的线程块数量应该要为显卡的SM数量的整数倍。


### 第二层：SM(流式多处理器)

SM结构图：

![722b05563a6c4b4b15b8755fc692323e.png](../_resources/722b05563a6c4b4b15b8755fc692323e.png)

SM是GPU的核心，重中之重。Fermi为例，一个SM主要包括：
- 32个CUDA核心
- 16个LD/ST单元（load/store）用来计算源地址和目的地址
- 4个SFU（Special function units）特殊计算单元，比如用来执行sin、cos等
- 两个线程束调度器（warp scheduler），每个线程调度器负责16个CUDA核心
- 一级缓存
- 共享内存

其中 CUDA核心 (又称 SP) 就是 GPU 真正的运算器，把数据放到上面，计算得到结果，包括一个浮点数计算单元和整数计算单元，在这里每个时钟周期执行一个整数或是浮点数指令，CUDA核心越多，自然算力更猛。

![32225028dde79326664058abaeb4b822.png](../_resources/32225028dde79326664058abaeb4b822.png)

Giga Thread会把线程块分配给SM，SM可以容纳多个线程块，直到内存等计算资源不够，Giga Thread引擎便不会再向该SM分配线程块。

SM如何管理调度线程块呢？首先我们知道线程块中的线程数量是不确定的，所以不能直接以线程块为处理单位。这时引入了wrap线程束概念，每32个线程为一组，被称为**线程束**。

把一个线程块划分为若干个线程束后，交给线程束调度器，顾名思义负责调度线程束，线程束有就绪态、运行态、阻塞态（跟cpu中的概念很像）。举个例子，遇到io，线程束进入阻塞态；cuda核心、内存等计算资源都空闲，转为就绪态。有多少个线程束调度器就能同时执行多少个线程束，Fermi架构有两个线程束调度器，Kepler架构有四个线程束调度器。

线程束调度器不断切换线程束，同一时刻只有两个线程束在运行。(。。因为Fermi只有2个线程束调度器)

![adb03dfc21a000c29025a084567edab8.png](../_resources/adb03dfc21a000c29025a084567edab8.png)

Fermi中的一个SM可以容纳48个线程束。也就说常驻1536个线程。


### 第三层：线程束

目前GPU的线程束中的线程数量wrapsize=32（32是一个重要的数字，后面会考）。线程束是并行处理的基本单元，线程束中的所有线程同时执行相同的指令！线程束的执行方式被称为==**单指令多线程**==，这与我们常说的==单指令多数据==的执行方式有什么区别呢？单指令多线程具有以下特征：
- 每个线程都有自己的指令地址计数器
- 每个线程都有自己的寄存器状态
- 每个线程可以有一个独立的执行路径（单指令多数据，那么全部执行，那么全部不执行，不支持if语句。而单指令多线程可以部分执行）

也就是说先把线程块分为若干个线程束，要是线程块中的线程数量不能整除wrapsize呢？比如一个线程块有80个线程，那么它会分为4个线程束，共计3*32=96。多出来的16个线程不活跃的，但记住即使这些线程未被使用， 它们仍然消耗SM的资源， 如寄存器。

在此我们得出一个结论：线程块中的线程数量应该为32的倍数！避免不必要的浪费，在写代码时如果需要用线程束的数量，最好以宏来代替32，虽然现在GPU的线程束都是32，但未来的架构是很有可能变化的。

### 线程束分化

设想一个非常简单的if语句if (cond) {……} else {……}。假设在一个线程束中有16个线程cond为true，但对于其他16个来说cond为false。这意味着一半的线程束需要执行if语句块中的指令，而另一半需要执行else语句块中的指令。 在同一线程束中的线程执行不同的指令，被称为**线程束分化**。

由于线程束是单指令多线程的执行，所以不支持16个线程执行if语句块，另外16个线程执行else语句块。那么当发生线程束分化时，线程束中16个线程执行if，冻结另外16个线程；然后，16个线程执行else，冻结另外16个线程。if语句变成了两步走，线程束分化会导致性能明显地下降。

这也解释了GPU不适合做逻辑复杂的控制密集型任务！像if、for、while语句:

```C++
__global__ void mathkernel1(float *c){
    int tid = blockIdx.x * blockDim.x +threadIdx.x;
    float a,b;
    a=b=0.0f;
    bool flag=tid % 2==0;
    if(flag){
        a=100.0f;
    }else{
        b=200.0f;
    }
    c[tid]=a+b;
}
```

上面的代码 就发生了 线程束分化，为什么不直接写if(tid %2==0)，CUDA编译器优化会将短的、有条件的代码段的断定指令取代了分支指令。所以要改写成 bool flag=tid %2==0;if(flag)。

我们不免会有逻辑控制的需求，那如何避免线程束分化呢？答案是：尝试调整分支粒度以适应线程束大小的倍数， 避免线程束分化。比如把上面条件代码改为(tid/warpSize)%2==0 。

。。。不知道。。


### 第四层：线程

线程是最小执行单位。

### 为什么需要 线程块Block

#### 通信
shared memory 是以 Block为单位分配的，一个Block内的线程共享 shared memory, 它伴随着 Block 的整个生命周期。 shared memory 是块内 thread 通信的 基本方式

#### 同步
CUDA 提供了 `__device__ void __syncthreads(void);` 函数来同步线程块中的所有线程。当`__syncthreads` 被调用时， 在同一个线程块中每个线程都必须等待直至该线程块中所有其他线程都已经达到这个同步点。 而Block之间，除结束核函数外是无法同步的。

#### 可拓展性
CUDA程序要保证可拓展性，即**在不同SM数量的GPU上都可以运行**。因此线程块分布在多个SM中。 网格中的线程块以并行或连续或任意的顺序被执行，这种独立性使得CUDA程序在任意数量的SM的GPU之间扩展。



## 存储系统结构

CPU 中 存储系统 分为： 内存，缓存，寄存器。 分层是为了 解决 CPU 和读写时间不匹配的问题。

GPU的 存储系统更复杂：
- 寄存器 register
- 本地内存 local memory
- 共享内存 shared memory
- 常量内存 constant memory
- 纹理内存 texture memory
- 全局内存 global memory
- 缓存 L1 cache, L2 cache, constant cache, texture cache

下面是 GPU 的存储系统结构图：

![cf06ea3104b4b195954154ae4076f8ce.png](../_resources/cf06ea3104b4b195954154ae4076f8ce.png)

通过上图，可以得到一些信息
- 寄存器和本地内存 是每个线程私有
- 共享内存为每一个线程块中的所有线程共享
- 全局内存，常量内存，纹理内存 被GPU 设备上的 所有线程共享。

### 1 寄存器
寄存器永远是 读写最快的存储结构，一般情况下 核函数没有特殊声明的话，变量都是放在寄存器中的。

每个线程私有的，生命周期和 线程一起，随着线程 释放而释放。并且注意寄存器是稀有资源。比如Fermi架构，每个thread上限为63个寄存器，Kepler则是255个。在编写代码时，让核函数使用更少的寄存器，这样每个SM能容纳更多Block，提高并行能力。

如果寄存器放不下，那会放到本地内存上，即所谓的register spilling（寄存溢出）。

### 2 本地内存

每个线程私有的  
当寄存器不够用时，就放到本地内存上，  
此外，还有几种情况，也会把变量放置在本地内存上：
- 编译期无法确定值的数组
- 占用较多内存的变量

### 3 共享内存

一个线程块中的所有线程共享。生命周期伴随整个Block，随着Block执行完成而释放。  
由`__shared__` 修饰符 修饰的变量放在共享内存中。相当于是CPU中的L1 cache，但是CPU的cache是不可编程的，而shared memory是可编程的。

共享内存常**用于块内线程通信**，同一个block中的thread通过共享内存来通信，在获取shared memory数据前必须先用`__syncthreads()`同步。


### 4 常量内存

设备中的所有线程可见  
每一个SM中都配备着专用的constant cache（per-SM）。声明时以`__connstant__`修饰。常量内存，顾名思义**只可读不可写**！


### 5 纹理内存
设备中的所有线程可见  
每一个SM中都配备着专用的texture cache（per-SM）。纹理内存是针对2D空间局部性的优化策略，所以thread要获取**2D数据就可以使用纹理内存来达到很高的性能**，D3D编程中有两种重要的基本存储空间，其中一个就是texture。

### 6 全局内存
全局内存的占有空间最大，对所有线程可见，全局就意味着伴随整个GPU进程生命周期。`__device__`修饰符表明是全局变量。比如cudaMalloc分配的就是全局内存！。


### 7 缓存

CPU 和 GPU的 cache 都不可编程，由 硬件自动实现缓存机制。 GPU的缓存 分为以下几种：
- L1 cache (per-SM)
- L2 cache (per-device)
- 只读的 constant cache (per-SM)
- 只读的 texture cache (per-SM)

每个 SM 都配置了 L1 cache、所有 SM 共享一个 L2 cache。 这2个都是用来 缓存 local 和 global memory 的。

在Fermi GPus 和 Kepler K40或者之后的GPU，CUDA允许我们配置读操作的数据是否使用L1和L2或者只使用L2。

在CPU方面，memory的load/store都可以被cache。但是在GPU上，只有load操作会被cache，store则不会。

每个SM都配备一个只读constant cache和texture cache来提升性能。


### 关键字 scope

各个存储结构的生命周期、作用域、限定符：

![df1c917646a5c822fb0c302b54e8b478.png](../_resources/df1c917646a5c822fb0c302b54e8b478.png)

`float var[100]`就是因为寄存器放不下，进而放在本地内存。
















