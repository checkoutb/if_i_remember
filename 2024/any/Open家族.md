
2024-05-24 17:12


gpu/cpu 计算：opencl

mpi能当做mq？还是rpc？ 
。分布式进程间通信。
https://zhuanlan.zhihu.com/p/158584571
https://mpitutorial.com/tutorials/


openmp 是一个多线程CPU并发。 不是 GPU




[[toc]]

---
---


# open家族简介

https://zhuanlan.zhihu.com/p/675542877

## 0.Open-AD/自动驾驶

### OpenPilot 48.2k

Openpilot是comma.ai, Inc的开源半自动驾驶系统。当与comma硬件配合使用时，它取代了各种汽车中的先进驾驶辅助系统，从而对原始系统进行了改进。
截至 2023 年， openpilot支持 250 多种车型，拥有 6000 多名用户，行驶里程超过 9000 万英里（1.4 亿公里）。
Openpilot在 comma.ai 开发的 comma 2/3/X 硬件上运行。

OpenPilot 官网：https://www.comma.ai/openpilot

OpenPilot 代码：https://github.com/commaai/openpilot

OpenPilot Wiki：https://en.wikipedia.org/wiki/Openpilot

OpenPilot 博客：https://blog.comma.ai/openpilot-in-2021/

OpenPilot Calar：https://www.researchgate.net/figure/Overall-architecture-of-OpenPilot-integrated-with-CARLA-the-driver-reaction-simulator_fig5_359971634


## 1.Open-IC/集成电路

### OpenFPGA 752

OpenFPGA 框架是第一个开源 FPGA IP 生成器，其硅证明支持高度可定制的 FPGA 架构。
OpenFPGA 为定制 FPGA 提供完整的 EDA 支持，包括 Verilog 到比特流生成和自测试验证。

OpenFPGA 为芯片设计者和研究人员提供灵活的原型设计方法和不断发展的 EDA 工具，为 FPGA 技术和 EDA 技术的大众化打开了大门。

OpenFPGA 代码：https://github.com/lnis-uofu/OpenFPGA

OpenFPGA 文档：https://openfpga.readthedocs.io/en/master/

OpenFPGA 文档：https://openfpga.readthedocs.io/en/latest/overview/motivation/


### OpenISP 982

图像信号处理器（ISP）是进行数字图像处理的应用处理器，专门用于将RAW图像（从成像传感器获取）转换为RGB/YUV图像（以进一步处理或显示）。

OpenISP项目旨在提供 ISP 的概述，并从硬件角度激发整个 ISP 管道和一些调整功能。所提出的 ISP 管道由以下模块组成：坏点校正 (DPC)、黑电平补偿 (BLC)、镜头阴影校正 (LSC)、抗锯齿噪声滤波器 (ANF)、

自动白平衡增益控制 (AWB)、彩色滤波器阵列插值 (CFA)、伽玛校正 (GC)、色彩校正矩阵 (CCM)、色彩空间转换 (CSC)、亮度和色度噪声滤波器 (NF)、边缘增强 (EE)、伪色抑制 (FCS)、色调/饱和度/控制 (HSC) 和亮度/对比度控制 (BCC)。

OpenISP 代码：https://github.com/cruxopen/openISP

OpenISP 介绍：https://zhuanlan.zhihu.com/p/645873125

OpenISP 介绍：https://aijishu.com/a/1060000000330426


### OpenDSP 46

OpenDSP 是一款无头实时操作系统，专为 Raspberry Pi 等嵌入式设备上的音频和视频数字信号处理而设计。 
OpenDSP 守护程序服务作为一个综合框架，用于创建可与任何 MIDI 或 OSC 兼容设备无缝连接的 DSP 设备。

通过结合 OpenDSP操作系统和服务，您可以获得模拟众多昂贵的专有 DSP 机器的能力，甚至可以开发自己的定制 DSP 解决方案来满足您的特定要求。

OpenDSP 代码：https://github.com/midilab/opendsp

OpenDSP 主页：https://midilab.co/opendsp/


### OpenRISC 473

OpenRISC是一个项目，旨在根据既定的精简指令集计算机(RISC) 原理开发一系列基于开源硬件的中央处理单元(CPU) 。

它包括使用开源许可证的指令集架构(ISA) 。它是OpenCores社区最初的旗舰项目。

OpenRISC 主页：https://openrisc.io/

OpenRISC Wiki：https://en.wikipedia.org/wiki/OpenRISC

OpenRISC 代码：https://github.com/openrisc


### OpenPLC

OpenPLC 是一种基于易于使用的软件的开源可编程逻辑控制器。我们的重点是为自动化和研究提供低成本的工业解决方案。OpenPLC 已在许多研究论文中用作工业网络安全研究的框架，因为它是唯一提供完整源代码的控制器。

OpenPLC 代码：https://github.com/thiagoralves/OpenPLC_v3

OpenPLC 主页：https://autonomylogic.com/

OpenPLC 概述：https://autonomylogic.com/docs/openplc-overview/


### OpenCore 已放弃

OpenCores是一个通过电子设计自动化(EDA)开发数字 开源硬件的社区，其精神与自由软件运动类似。OpenCores希望消除冗余设计工作并大幅降低开发成本。

许多公司在芯片中采用 OpenCores IP，[1] [2]或作为 EDA 工具的附件。[3] [4] OpenCores 有时也被引用为电子硬件社区中开源的一个例子。

OpenCore 主页：https://opencores.org/

OpenCore Wiki：https://en.wikipedia.org/wiki/OpenCores

### OpenCPU 708

用于使用 ==R== 进行嵌入式科学计算和可重复研究的系统。OpenCPU 服务器公开了一个简单但功能强大的 HTTP API，用于 RPC 以及与 R 的数据交换。这为统计服务或构建 R Web 应用程序提供了可靠且可扩展的基础。

OpenCPU 服务器既可以作为交互式 R 会话中的单用户开发服务器运行，也可以作为基于 Apache2 的多用户 Linux 堆栈运行。整个系统完全开源并获得许可。

OpenCPU 代码：https://github.com/opencpu/opencpu

OpenCPU 文档：https://www.opencpu.org/api.html


### OpenGPU 已放弃

GPUOpen是一个中间件 软件套件，最初由AMD的 Radeon Technologies Group 开发，为计算机游戏提供高级视觉效果。它于 2016 年发布。

GPUOpen 是Nvidia GameWorks的替代品，也是其直接竞争对手。GPUOpen 与 GameWorks 类似，它包含多种不同的图形技术作为其主要组件，这些技术以前是相互独立且分离的。

OpenGPU Wiki：https://en.wikipedia.org/wiki/GPUOpen

### OpenROAD 1.4k

OpenROAD 是领先的半导体数字设计开源基础应用程序。OpenROAD 流程提供自主、无人参与环路 (NHIL) 流程，可从 RTL-GDSII 进行 24 小时周转，以实现快速设计探索和物理设计实施。

OpenROAD 代码：https://github.com/The-OpenROAD-Project/OpenROAD


### OpenCUDA 已放弃

。。这篇文章 是不是 Open 开头就拿进来？ github 上是中文的。我不觉得 OpenCUDA 这种 正式项目会 中文。 而且 7年前就不维护了。

基于CUDA的一套图像处理开源项目。

OpenCUDA 代码：https://github.com/LitLeo/OpenCUDA

OpenCUDA 介绍：https://blog.csdn.net/litdaguang/article/details/50143861

### OpenNPU

OpenNPU 主页：https://www.nvidia.com/en-us/networking/

### OpenDPU 已放弃

分布式处理单元

OpenDPU 代码：https://github.com/kasugare/openDPU

## 2.Open-HPC/高性能计算

### OpenCL 488

OpenCL(全称Open Computing Language，开放运算语言)是第一个面向异构系统通用目的并行编程的开放式、免费标准，也是一个统一的编程环境，便于软件开发人员为高性能计算服务器、桌面计算系统、手持设备编写高效轻便的代码，而且广泛适用于多核心处理器(CPU)、图形处理器(GPU)、Cell类型架构以及数字信号处理器(DSP)等其他并行处理器，在游戏、娱乐、科研、医疗等各种领域都有广阔的发展前景。

OpenCL 官网：https://opencl.org/

OpenCL Github：https://github.com/OpenCL

OpenCL SDK：https://github.com/KhronosGroup/OpenCL-SDK

OpenCL Guide：https://github.com/KhronosGroup/OpenCL-Guide

OpenCL Wiki：https://en.wikipedia.org/wiki/OpenCL

。。https://github.com/KhronosGroup/OpenCL-CLHPP
。。星有点少，我以为 万星起步的。。


。。NVIDA下搜 cuda， 
https://github.com/NVIDIA/cccl  838星
CUDA C++ Core Libraries (CCCL)

https://github.com/NVIDIA/cuda-samples  5.5k

https://github.com/NVIDIA/cutlass  4.7k
矩阵乘法

https://github.com/NVIDIA/libcudacxx  2.3k


### OpenACC 50

OpenACC 组织致力于通过扩展加速和并行计算技能来帮助研究和开发人员社区推进科学发展。有 3 个重点领域：参与计算生态系统开发，提供编程模型、资源和工具方面的培训和教育，以及开发 OpenACC 规范。

OpenACC 官网：https://www.openacc.org/

OpenACC Github：https://github.com/OpenACC

OpenACC Wiki：https://en.wikipedia.org/wiki/OpenACC


### OpenMPI 2k

OpenMPI 是一种高性能消息传递库，最初是作为融合的技术和资源从其他几个项目（FT- MPI, LA-MPI, LAM/MPI, 以及 PACX-MPI），它是MPI-2标准的一个开源实现，由一些科研机构和企业一起开发和维护。

OpenMPI 官网：https://www.open-mpi.org/

OpenMPI V5.0：https://www.open-mpi.org/software/ompi/v5.0/

OpenMPI Doc：https://www.open-mpi.org/doc/

OpenMPI 代码：https://github.com/open-mpi/ompi 
OpenMPI Wiki：https://en.wikipedia.org/wiki/Open_MPI

### OpenMP 220

。官网挺活跃的，活动很多。

。https://github.com/OpenMP/Examples

OpenMP（Open Multi-Processing）是一套支持跨平台共享内存方式的多线程并发的编程API，使用C,C++和Fortran语言，
可以在大多数的处理器体系和操作系统中运行，包括Solaris, AIX, HP-UX, GNU/Linux, Mac OS X, 和Microsoft Windows。
包括一套编译器指令、库和一些能够影响运行行为的环境变量。 
OpenMP采用可移植的、可扩展的模型，为程序员提供了一个简单而灵活的开发平台，从标准桌面电脑到超级计算机的并行应用程序接口。

OpenMP 官网：https://www.openmp.org/

OpenMP Github：https://github.com/OpenMP

OpenMP Example：https://github.com/OpenMP/Examples

OpenMP 代码：https://github.com/OpenMP/sources


### OpenBLAS 6k

OpenBLAS 是一个基于 GotoBLAS2 1.13 BSD 版本优化的 BLAS（基本线性代数子程序）库。

。。线性代数，那不得 矩阵运算

OpenBLAS 主页：https://www.openblas.net/

OpenBLAS Github：https://github.com/OpenMathLib/OpenBLAS

OpenBLAS Wiki：https://github.com/OpenMathLib/OpenBLAS/wiki

## 3.Open-Sensor/传感器

### OpenLiDAR 236

OpenLiDAR该项目的目标是支持各种传感器和控制器，以便人们可以使用现成的设备构建自己的 3D LiDAR 扫描仪。 
一般设置包括一个经济实惠的LiDAR 传感器，安装在一个结构上，可以精确且准确地平移（我们称之为安装座）。还有其他传感器可以帮助您的扫描仪获得更多或更好的数据，例如GPS和/或磁力计

OpenLiDAR 代码：https://github.com/patriciogonzalezvivo/OpenLiDAR

### OpenLiDAR 13

这Xaxxon OpenLIDAR 传感器是一款具有开放软件和硬件的旋转激光扫描仪，旨在与自主移动机器人和同步定位与建图 (SLAM) 应用程序一起使用。 即使在阳光照射的环境中，它也是避障、自主导航和测绘的理想选择。它与所有版本的ROS完全兼容。

OpenLidar 主页：http://www.xaxxon.com/xaxxon/openlidar

OpenLiDAR 代码：https://github.com/xaxxontech

OpenLiDAR 代码：https://github.com/xaxxontech/xaxxon_openlidar

OpenLiDAR 代码：https://github.com/xaxxontech/xaxxon_openlidar_pcb


### OpenRadar 50

免费开源天气雷达软件的社区平台。

OpenRadar 代码：https://github.com/openradar

OpenRadar 主页：https://openradarscience.org/


### OpenCamera 1.1k

打开相机项目 - 适用于 Android 的多功能相机应用程序。

OpenCamera 代码：https://github.com/almalence/OpenCamera

OpenCamera 主页：https://opencamera.org.uk/

## 4.Open-ASAM/自动驾驶

### OpenDrive 359

ASAM OpenDRIVE 格式提供了使用文件扩展名 xodr 的可扩展标记语言 (XML) 语法描述道路网络的通用基础。
存储在 ASAM OpenDRIVE 文件中的数据描述了道路、车道和物体的几何形状（例如道路上的路标）以及道路沿线的要素（例如信号）。

ASAM OpenDRIVE 文件中描述的道路网络可以是合成的，也可以基于真实数据。

OpenDrive 主页：https://www.asam.net/standards/detail/opendrive/

OpenDrive 文档：https://releases.asam.net/OpenDRIVE/1.6.0/ASAM_OpenDRIVE_BS_V1-6-0.html

OpenDrive 代码：https://github.com/pageldev/libOpenDRIVE

OpenDrive Wiki：https://en.wikipedia.org/wiki/OpenDRIVE_(specification)

### OpenScenario 248

ASAM OpenSCENARIO 定义了一种文件格式，用于描述驾驶和交通模拟器的动态内容。ASAM OpenSCENARIO 的主要用例是描述涉及车辆、行人和其他交通参与者等多个实体的复杂、同步的操作。
操纵的描述可以基于驾驶员动作（例如，执行变道）或基于轨迹（例如，从记录的驾驶操纵导出）。
其他内容，例如自我车辆的描述、驾驶员外观、行人、交通和环境条件也包含在标准中。

OpenScenario 主页：https://www.asam.net/standards/detail/openscenario/

OpenScenario 文档：https://www.asam.net/index.php?eID=dumpFile&t=f&f=4092&token=d3b6a55e911b22179e3c0895fe2caae8f5492467

OpenScenario 代码：https://github.com/pyoscx/scenariogeneration

OpenScenario Wiki：https://en.wikipedia.org/wiki/OpenDRIVE_(specification)

### OpenCRG 0

ASAM OpenCRG 定义了一种用于描述路面的文件格式。
它最初是为了存储路面扫描的高精度高程数据而开发的。
该数据的主要用途是轮胎、振动或驾驶模拟。
精确的海拔数据可以对车辆部件或整个车辆进行真实的耐久性模拟。
对于驾驶模拟器，它可以对路面进行逼真的 3D 渲染。
该文件格式还可用于其他类型的路面属性，例如摩擦系数或灰度值。

OpenCRG 主页：https://www.asam.net/standards/detail/opencrg/

OpenCRG 文档：https://www.asam.net/index.php?eID=dumpFile&t=f&f=3950&token=21a7ae456ec0eb0f9ec3aee5bae3e8c9ebaea140

OpenCRG 代码：https://github.com/hlrs-vis/opencrg

OpenCRG Wiki：https://en.wikipedia.org/wiki/OpenCRG

### OpenOSI

ASAM OSI（开放模拟接口）在自动驾驶功能和各种可用的驾驶模拟框架之间提供简单直接的兼容性。
它允许用户通过标准化接口将任何传感器连接到任何自动驾驶功能和任何驾驶模拟器工具。它简化了集成，从而显着增强了虚拟测试的可访问性和实用性。

OpenOSI 主页：https://www.asam.net/standards/detail/osi/

OpenOSI 文档：https://www.asam.net/project-detail/asam-osi-v400/

OpenOSI 代码：https://github.com/OpenSimulationInterface

OpenOSI 文档：https://www.asam.net/index.php?eID=dumpFile&t=f&f=3328&token=8722ea00c31868ea55a9efa037d6215d7e5d2343


### OpenLabel

ASAM OpenLABEL 定义了对象和场景的注释格式和标记方法。
ASAM OpenLABEL 提供了有关如何使用标签方法和定义的指南。
通过与不同客户的合作，每个组织对驾驶环境中的对象进行分类和描述的方式出现了明显的碎片化。
此类分类和描述是任何自动驾驶系统 (ADS) 感知堆栈的基本构建块，因为通过它们，ADS 才能对其周围环境的状态有基本而深刻的了解。

OpenLabel 主页：https://www.asam.net/standards/detail/openlabel/

OpenLabel 文档：https://www.asam.net/project-detail/asam-openlabel-v100/

### OpenODD

操作设计域定义 (ODD) 应在车辆的整个使用寿命期间有效，并且是其安全和操作概念的一部分。ODD 用于联网自动车辆的功能规范。它指定 CAV 必须能够管理哪些环境参数（静态和动态）。它们包括所有类型的交通参与者、天气条件、基础设施、位置、一天中的时间以及可能对驾驶情况产生影响的所有其他因素。

OpenODD 主页：https://www.asam.net/standards/detail/openodd/

OpenODD 主页：https://www.asam.net/project-detail/asam-openodd/

## 5.Open-CV/视觉图像

### OpenCV 76.1k

OpenCV（开源计算机视觉库）是一个主要针对实时计算机视觉的编程函数库。最初由Intel开发，后来得到Willow Garage的支持，然后是Itseez（后来被Intel收购）。

OpenCV库是跨平台的，并根据Apache License 2 作为免费开源软件获得许可。从 2011 年开始，OpenCV 具有用于实时操作的 GPU 加速功能。

OpenCV 官网：https://opencv.org/

OpenCV 代码：https://github.com/opencv/opencv

OpenCV Contrib 代码：https://github.com/opencv/opencv_contrib

OpenCV 教程：https://docs.opencv.org/4.x/d9/df8/tutorial_root.html

OpenCV Contrib 教程：https://docs.opencv.org/3.4/d3/d81/tutorial_contrib_root.html

OpenCV Wiki：https://en.wikipedia.org/wiki/OpenCV

### OpenVX

OpenVX 是一种开放、免版税的计算机视觉应用跨平台加速标准。 OpenVX 可实现性能和功耗优化的计算机视觉处理，这在嵌入式和实时用例中尤其重要，

例如面部、身体和手势跟踪、智能视频监控、高级驾驶员辅助系统 (ADAS)、对象和场景重建、增强现实、目视检查、机器人技术等。

OpenVX 官网：https://www.khronos.org/openvx/

OpenVX 示例：https://github.com/KhronosGroup/OpenVX-sample-impl

OpenVX Wiki：https://zh.wikipedia.org/zh-cn/OpenVX


### OpenMVG 5.5k

OpenMVG(Open Multiple View Geometry，多视图几何)提供由库、二进制文件和管道组成的图像框架的端到端 3D 重建。 这些库提供了对以下功能的轻松访问：

图像处理、特征描述和匹配、特征跟踪、相机模型、多视图几何、鲁棒估计、运动结构算法；

这些二进制文件解决了管道可能需要的单元任务：场景初始化、特征检测和匹配以及运动重建结构，将重建的场景导出到其他多视图立体框架以计算密集点云或纹理网格。

管道是通过链接各种二进制文件来创建的，以计算图像匹配关系，解决运动结构问题（重建、三角测量、定位）。

OpenMVG 采用 C++ 开发，可在 Android、iOS、Linux、macOS 和 Windows 上运行。

OpenMVG 代码：https://github.com/openMVG/openMVG

OpenMVG 文档：https://openmvg.readthedocs.io/en/latest/

OpenMVG Wiki：https://github.com/openMVG/openMVG/wiki

OpenMVG 介绍：https://docs.google.com/presentation/d/1eEd12oUeKRckuymHHfauclpc62QGpGRI67B02wcKA0I/edit#slide=id.gdc9849b1f9_0_614

### OpenMVS 3.1k

OpenMVS(Open Multiple View Stereovision，多视图立体)是一个面向计算机视觉科学家的库，特别针对多视图立体重建社区。OpenMVS旨在通过提供一整套算法来恢复要重建的场景的整个表面来填补这一空白。输入是一组相机姿势加上稀疏点云，输出是纹理网格。

该项目涵盖的主要主题是：密集点云重建以获得尽可能完整和准确的点云网格重建，用于估计最能解释输入点云的网格表面网格细化以恢复所有精细细节网格纹理，用于计算清晰且准确的纹理来为网格着色

OpenMVS 代码：https://github.com/cdcseacave/openMVS

OpenMVS 文档：https://openmvg.readthedocs.io/en/latest/openMVG/openMVG/

### OpenSFM 3.2k

OpenSfM 是一个用 Python 编写的 Motion 库的结构。该库充当从多个图像重建相机姿势和 3D 场景的处理管道。

它由 Structure from Motion 的基本模块（特征检测/匹配、最小解算器）组成，重点是构建强大且可扩展的重建管道。

它还集成了外部传感器（例如 GPS、加速计）测量，以实现地理对准和稳健性。提供了 JavaScript 查看器来预览模型和调试管道。

OpenSFM 代码：https://github.com/mapillary/OpenSfM

OpenSFM 主页：https://opensfm.org/

OpenSFM 文档：https://opensfm.readthedocs.io/en/latest/

### OpenDroneMap 4.6 py

OpenDroneMap是一个开源摄影测量工具包，用于将航空图像（通常来自无人机）处理为地图和 3D 模型。该软件在GitHub上免费托管和分发。

OpenDroneMap 主页：https://www.opendronemap.org/

OpenDroneMap 代码：https://github.com/OpenDroneMap

OpenDroneMap 代码：https://github.com/OpenDroneMap/ODM

OpenDroneMap Wiki：https://en.wikipedia.org/wiki/OpenDroneMap

### Open3D 10.6k

Open3D 是一个开源库，支持快速开发处理 3D 数据的软件。Open3D 前端公开了一组精心挑选的 C++ 和 Python 数据结构和算法。后端经过高度优化并设置为并行化。

Open3D 主页：https://www.open3d.org/

Open3D 代码：https://github.com/isl-org/Open3D

Open3D 文档：https://www.open3d.org/docs/release/introduction.html


### OpenFace 6.6k

OpenFace 是第一个能够进行面部标志检测、头部姿势估计、面部动作单元识别和眼睛注视估计的工具包，并提供用于运行和训练模型的可用源代码。

代表 OpenFace 核心的计算机视觉算法在上述所有任务中都展示了最先进的结果。此外，我们的工具具有实时性能，并且能够从简单的网络摄像头运行，无需任何专业硬件。

OpenFace 代码：https://github.com/TadasBaltrusaitis/OpenFace

### OpenPose

OpenPose 代码：https://github.com/CMU-Perceptual-Computing-Lab/openpose


### OpenALPR 10.9k

OpenALPR 是一个用 C++ 编写的开源_自动车牌识别_库，绑定了 C#、Java、Node.js、Go 和 Python。该库分析图像和视频流来识别车牌。输出是任何车牌字符的文本表示。

OpenALPR 代码：https://github.com/openalpr/openalpr

### OpenTLD 2.1k

TLD 是一种用于在无约束视频流中跟踪未知对象的算法。感兴趣的对象由单个帧中的边界框定义。TLD 同时跟踪对象、了解其外观并在视频中出现时检测到它。

OpenTLD 代码：https://github.com/zk00006/OpenTLD

### OpenPano 1.8k

OpenPano是一个用C++从头开始编写的全景拼接程序（没有任何视觉库）。它主要遵循论文Automatic Panoramic Image Stitching using Invariant Features中描述的例程，这也是AutoStitch使用的例程。

OpenPano 代码：https://github.com/ppwwyyxx/OpenPano

## 6.Open-CG/图形

### OpenGL

OpenGL（开放图形库[3]）是一种跨语言、跨平台的 应用程序编程接口（API），用于渲染2D和3D 矢量图形。该API通常用于与图形处理单元（GPU）交互，以实现硬件加速 渲染。

Silicon Graphics, Inc. (SGI)于1991年开始开发OpenGL，并于1992年6月30日发布；并应用广泛应用于计算机辅助设计（CAD）、虚拟现实、科学可视化、信息可视化、飞行模拟和视频游戏等领域。

OpenGL 官网：https://www.opengl.org/

OpenGL 教程：https://www.opengl.org/sdk/docs/tutorials/

OpenGL SDK：https://www.opengl.org/sdk/

OpenGL Wiki：https://en.wikipedia.org/wiki/OpenGL

### OpenVG

OpenVG（Open Vector Graphics）是针对诸如Flash和SVG的矢量图形算法库提供底层硬件加速界面的免授权费、跨平台应用程序接口API。

OpenVG 现仍处于发展阶段，其初始目标主要面向需要高质量矢量图形算法加速技术的便携手持设备，用以在小屏幕设备上实现动人心弦的用户界面和文本显示效果，并支持硬件加速以在极低的处理器功率级别下实现流畅的交互性能。

OpenVG 主页：https://www.khronos.org/openvg/

OpenVG 文档：https://github.com/KhronosGroup/OpenVG-Docs

OpenVG 百科：https://baike.baidu.com/item/OpenVG

OpenVG Wiki：https://en.wikipedia.org/wiki/OpenVG


### OpenSceneGraph 3.1k

OpenSceneGraph是一个开源3D 图形应用程序编程接口（库或框架），[2]供视觉模拟、计算机游戏、虚拟现实、科学可视化和建模等领域的应用程序开发人员使用。 该工具包使用OpenGL 用标准C++编写，可在各种操作系统上运行，包括Microsoft Windows、macOS、Linux、IRIX、Solaris和FreeBSD。

从3.0.0版本开始，OpenSceneGraph还支持移动平台的应用程序开发，即iOS和Android。

OpenSceneGraph 代码：https://github.com/openscenegraph/OpenSceneGraph

OpenSceneGraph 主页：https://www.openscenegraph.com/

OpenSceneGraph Wiki：https://en.wikipedia.org/wiki/OpenSceneGraph

### OpenSubdiv 2.8k

OpenSubdiv 是一组开源库，可在大规模并行 CPU 和 GPU 架构上实现高性能细分曲面 (subdiv) 评估。此代码路径针对以交互帧速率绘制具有静态拓扑的变形细分进行了优化。生成的极限曲面与 Pixar 的 Renderman 的数值精度相匹配。

OpenSubdiv 代码：https://github.com/PixarAnimationStudios/OpenSubdiv

### OpenEmu 15.9k

OpenEmu 是一个开源项目，其目的是将 macOS 游戏模拟带入一流公民领域。该项目利用现代 macOS 技术，例如 Cocoa、Metal、Core Animation 和其他第三方库。Sparkle 是第三方库的一个示例，它用于自动更新。

OpenEmu 使用模块化架构，允许游戏引擎插件，允许 OpenEmu 支持许多不同的模拟引擎和后端，同时保留熟悉的 macOS 原生前端。

OpenEmu 代码：https://github.com/OpenEmu/OpenEmu

### OpenRA 14.2k

支持早期 Westwood 经典的 Libre/免费实时策略游戏引擎。

OpenRA 代码：https://github.com/OpenRA/OpenRA

OpenRA 主页：https://www.openra.net/


## 7.Open-MMedia/多媒体

### OpenMG

OpenMG是索尼开发的数字版权管理(DRM) 系统，用于管理和保护个人计算机上的数字音乐数据。

它最初是为ATRAC3格式的音频文件设计的；兼容的软件，例如Sony SonicStage，通常能够将MP3和WAV文件转码为OpenMG/ATRAC3。OpenMG 加密文件使用的文件扩展名是和。

OpenMG Wiki：https://en.wikipedia.org/wiki/OpenMG

OpenMG 主页：https://en.wikipedia.org/wiki/OpenMG

### OpenMAX

OpenMAX 是一种免版税的跨平台 API，通过支持跨多个操作系统和芯片平台开发、集成和编程加速多媒体组件，提供全面的流媒体编解码器和应用程序可移植性。

OpenMAX API 将随处理器一起提供，以使库和编解码器实施者能够快速有效地利用新芯片的全部加速潜力 - 无论底层硬件架构如何。

OpenMAX Wiki：https://zh.wikipedia.org/wiki/OpenMAX

OpenMAX 介绍：https://www.cnblogs.com/wujianming-110117/p/14204684.html

OpenMAX 主页：https://www.khronos.org/openmax/

### OpenH264 5.4k

OpenH264是一个免费软件 库，用于实时编码和解码H.264/MPEG-4 AVC格式的视频流。[2]它是根据简化 BSD 许可证的条款发布的。

OpenH264 代码：https://github.com/cisco/openh264

OpenH264 主页：https://www.openh264.org/

OpenH264 Wiki：https://en.wikipedia.org/wiki/OpenH264


### OpenAL 2.1k

OpenAL Soft 是 LGPL 许可的跨平台 OpenAL 3D 音频 API 软件实现。它是从最初可从 http://openal.org 的 SVN 存储库（现已不复存在）获得的开源 Windows 版本中分叉出来的。

OpenAL 提供在虚拟 3D 环境中播放音频的功能。API 处理的功能包括距离衰减、多普勒频移和定向声音发射器。通过 EFX 扩展可以获得更高级的效果，包括空气吸收、遮挡和环境混响。它还有助于流式音频、多通道缓冲区和音频捕获。

OpenAL 代码：https://github.com/kcat/openal-soft


### OpenELEC 1.6k

。。已放弃 最后提交 7 years ago

OpenELEC 运行Kodi，这是一款屡获殊荣的免费开源 (GPL) 软件媒体播放器和数字媒体娱乐中心。基本系统的设计和构建是为了尽可能高效——仅消耗很小的磁盘和内存占用空间，并提供尖端的硬件支持来提供机顶盒体验。

OpenELEC 代码：https://github.com/OpenELEC/OpenELEC.tv

### OpenJK 2k

OpenJK 是 JACoders 小组的一项努力，旨在维护和改进 Jedi Academy (JA) 和 Jedi Outcast (JO) 游戏运行的游戏引擎，同时保持与现有游戏的完全向后兼容性。该项目不会尝试重新平衡或以其他方式修改核心游戏玩法。

OpenJK 代码：https://github.com/JACoders/OpenJK

## 8.Open-Game/游戏

### OpenRCT2 13k

RollerCoaster Tycoon 2 的开源重新实现，这是一款模拟游乐园管理的建设和管理模拟视频游戏。

OpenRCT2 代码：https://github.com/OpenRCT2/OpenRCT2

### OpenTTD

OpenTTD是一款商业模拟游戏，玩家尝试通过公路、铁路、水路和航空运输乘客、矿产和货物来赚钱。它是1995 年Chris Sawyer视频游戏Transport Tycoon Deluxe的开源[5] 重制版和扩展版。

OpenTTD 主页：https://www.openttd.org/

OpenTTD Wiki：https://en.wikipedia.org/wiki/OpenTTD


### OpenBBT 26.2k 金融终端

第一个免费且完全开源的金融终端。该终端拥有 600 多个命令，可以访问股票、期权、加密货币、外汇、宏观经济、固定收益、另类数据集等。

OpenBBT：https://github.com/OpenBB-finance/OpenBBTerminal


### OpenDiablo2 10.7k

OpenDiablo2是一个 ARPG 游戏引擎，与 2000 年代的游戏一脉相承，支持玩暗黑破坏神 2。 该引擎是用 Go 编写的，并且是跨平台的。

OpenDiablo2 代码：https://github.com/OpenDiablo2/OpenDiablo2

## 9.Open-ML/机器学习

### OpenNN 1.1k

OpenNN 是一个用 C++ 编写的用于高级分析的软件库。它实现了神经网络，这是最成功的机器学习方法。 OpenNN 的主要优点是其高性能。

该库在执行速度和内存分配方面表现出色。它不断优化和并行化，以最大限度地提高其效率。

OpenNN 的一些典型应用包括商业智能（客户细分、客户流失预防……）、医疗保健（早期诊断、微阵列分析……）和工程（性能优化、预测维护……）。

OpenNN 主页：https://www.opennn.net/

OpenNN 代码：https://github.com/Artelnics/opennn

OpenNN 文档：https://www.opennn.net/documentation/

OpenNN Wiki：https://en.wikipedia.org/wiki/OpenNN


### OpenML 645

OpenML 旨在通过创建一个开放、无摩擦的平台来访问和共享数据集、模型和实验，从而实现机器学习的民主化。随时随地。它使科学家能够轻松地在彼​​此的工作基础上进行构建，从过去的经验中学习，并自动化他们的工作流程。

OpenML 代码：https://github.com/openml

OpenML 代码：https://github.com/openml/OpenML


### OpenMLDB 1.6k

OpenMLDB 是一个开源机器学习数据库，为训练和推理提供计算一致特征的特征平台。

OpenMLDB 代码：https://github.com/4paradigm/OpenMLDB

### OpenNMT 2.4k

OpenNMT是一个用于神经机器翻译和神经序列学习的开源生态系统。

OpenNMT 主页：https://opennmt.net/

OpenNMT 代码：https://github.com/OpenNMT

OpenNMT 代码：https://github.com/OpenNMT/OpenNMT

### OpenNMYpy 6.6k

OpenNMT-py 是OpenNMT项目的PyTorch版本，OpenNMT 项目是一个开源 (MIT) 神经机器翻译（及其他！）框架。它旨在方便研究，以尝试翻译、语言建模、摘要和许多其他 NLP 任务中的新想法。一些公司已经证明代码可以投入生产。

OpenNMTpy 代码：https://github.com/OpenNMT/OpenNMT-py

### OpenNMTtf 1.4k

OpenNMT-tf 是一个使用 TensorFlow 2 的通用序列学习工具包。虽然神经机器翻译是主要目标任务，但它的设计目的是更广泛地支持： 序列到序列映射 序列标记 序列分类 语言建。

OpenNMTtf 代码：https://github.com/OpenNMT/OpenNMT-tf

## 10.Open-KG/知识图谱

### OpenKG 30

。中文

OpenKG：https://github.com/OpenKG-ORG

OpenKG：http://openkg.org/

OpenKG：http://www.openkg.cn/

### OpenConcepts 37

。。中文

OpenConcepts 是一个基于自动化知识抽取重构构建的中文概念图谱，以及由浙江大学知识引擎实验室贡献。本次开源了OpenConcepts中的440万概念核心实体，5万概念和1200万实体-概念三元组，并提供json,ttl, json-ld多种格式的原始转储下载。

OpenConcepts 代码：https://github.com/OpenKG-ORG/OpenConcepts

OpenConcepts 主页： http://openconcepts.zjukg.cn/

### OpenRichpedia 23

Richpedia：全面的多模态知识图谱。

OpenRichpedia 代码：https://github.com/OpenKG-ORG/OpenRichpedia


### OpenUE 34

OpenUE是一个简单可用的通用自然语言信息抽取工具，适用于python初学者或有经验的机器学习开发人员。

OpenUE 代码：https://github.com/OpenKG-ORG/OpenUE

### OpenSPG 441

OpenSPG是蚂蚁集团与OpenKG合作开发的知识图谱引擎，基于SPG（Semantic-enhanced Programmable Graph）框架，是蚂蚁集团多年在金融场景构建和应用不同领域知识图谱经验的总结。 。

OpenSPG 代码：https://github.com/OpenKG-ORG/openspg


### OpenEA 25

DeepKE是一个用于知识图谱构建的知识提取工具包，支持cnSchema、低资源、文档级和多模态场景的实体、关系和属性提取。我们为初学者提供文档、Google Colab 教程、在线演示、论文、幻灯片和海报。

OpenEA 代码：https://github.com/OpenKG-ORG/OpenEA


### OpenNE 1.7k

OpenNE 代码：https://github.com/thunlp/OpenNE


### OpenKE 3.7k

该存储库是 THU-OpenSKL 的一个项目，旨在通过表示学习来利用结构化知识和非结构化语言的力量。

OpenKE 代码：https://github.com/thunlp/OpenKE

### OpenNRE 4.3k

该存储库是 THU-OpenSKL 的一个项目，旨在通过表示学习来利用结构化知识和非结构化语言的力量。

OpenNRE 代码：https://github.com/thunlp/OpenNRE

## 11.Open-LLM/大模型

### OpenLLM 9k

OpenLLM 是一个开源平台，旨在促进大型语言模型 (LLM) 在实际应用中的部署和操作。借助 OpenLLM，您可以在任何开源 LLM 上运行推理，将其部署在云端或本地，并构建强大的 AI 应用程序。

OpenLLM：https://github.com/bentoml/OpenLLM

### OpenAgents 3.6k

OpenAgents，这是一个开放平台，用于在日常生活中使用和托管语言代理

OpenAgents代码：https://github.com/xlang-ai/OpenAgents

### OpenPrompt 4.2k py

提示学习是将预训练语言模型 (PLM) 应用于下游 NLP 任务的最新范例，它使用文本模板修改输入文本，并直接使用 PLM 来执行预训练任务。

该库提供了一个标准、灵活且可扩展的框架来部署即时学习管道。OpenPrompt 支持直接从Huggingface 转换器加载 PLM 。未来，我们还将支持其他图书馆实施的PLM。有关即时学习的更多资源，请查看我们的论文列表。

OpenPrompt 代码：https://github.com/thunlp/OpenPrompt

### OpenPromptStudio 5.8k vue

这是一个旨在把 AIGC 提示词（现在支持 Midjourney）可视化并提供编辑功能的工具，有以下特：

显示英文提示词的中文翻译；翻译输入的中文提示词到英文（因为 Midjourney 仅支持英文提示词） ；为提示词进行分类（普通、样式、质量、命令） ；轻松的排序、隐藏提示词 把提示词可视化结果导出为图片；常用提示词词典；通过 Notion 管理提示词词典。

OpenPromptStudio：https://github.com/Moonvy/OpenPromptStudio


## 12.Open-ICT/信息通信

### OpenWRT

OpenWrt 项目是一个针对嵌入式设备的 Linux 操作系统。OpenWrt 没有尝试创建单个静态固件，而是提供了一个具有包管理功能的完全可写文件系统。

这使您无需选择供应商提供的应用程序和配置，并允许您通过使用软件包来定制设备以适应任何应用程序。

对于开发人员来说，OpenWrt 是构建应用程序的框架，而无需围绕它构建完整的固件；对于用户来说，这意味着完全定制的能力，以前所未有的方式使用设备。

OpenWRT 代码：https://github.com/openwrt/openwrt

OpenWRT 主页：https://openwrt.org/

OpenWRT Wiki：https://zh.wikipedia.org/wiki/OpenWrt

### OpenBSD

OpenBSD是一个基于Berkeley Software Distribution (BSD) 的、注重安全的、免费的、开源的、类 Unix 操作系统。

Theo de Raadt于 1995 年通过分叉NetBSD 1.0 创建了 OpenBSD。OpenBSD项目强调可移植性、标准化、正确性、主动安全性和集成密码学。

OpenBSD 代码：https://en.wikipedia.org/wiki/OpenBSD

OpenBSD 官网：https://www.openbsd.org/

OpenBSD 文档：https://www.openbsd.org/74.html

OpenBSD Wiki：https://en.wikipedia.org/wiki/OpenBSD

### OpenFlow 300 ts

OpenFlow是一种通信协议，可通过网络访问网络交换机或路由器的转发平面。

OpenFlow 使网络控制器能够确定网络数据包在交换机网络中的路径。控制器与开关不同。与使用访问控制列表(ACL) 和路由协议相比，这种控制与转发的分离允许进行更复杂的流量管理。

此外，OpenFlow 允许使用单一开放协议远程管理来自不同供应商的交换机（通常每个供应商都有自己的专有接口和脚本语言）。该协议的发明者认为 OpenFlow 是软件定义网络(SDN)的推动者。

OpenFlow Wiki：https://en.wikipedia.org/wiki/OpenFlow

OpenFlow 代码：https://github.com/open-rpa/openflow

### OpenFire 2.8k java

。。MQTT 更简单，易用， 实时性要求不高 (比如，传感器，它上传数据后，不需要关心后面的事情)
。。XMPP 更复杂，全面， 有实时性要求，使用xml。
。。MQTT 是主流

Openfire是根据开源 Apache 许可证获得许可的实时协作 (RTC) 服务器。它使用唯一广泛采用的即时消息开放协议 XMPP（也称为 Jabber）。

Openfire 非常容易设置和管理，但提供坚如磐石的安全性和性能。 Openfire是一个根据开源 Apache 许可证授权的 XMPP 服务器。

OpenFire 代码：https://github.com/igniterealtime/Openfire

OpenFire Wiki：https://en.wikipedia.org/wiki/Openfire


### OpenSSH 2.8k

OpenSSH 是 SSH 协议（版本 2）的完整实现，用于安全远程登录、命令执行和文件传输。它包括客户端ssh和服务器sshd、文件传输实用程序scp以及sftp密钥生成 ( ssh-keygen)、运行时密钥存储 ( ssh-agent) 和许多支持程序的工具。

这是 OpenBSD 的OpenSSH到大多数类 Unix 操作系统的端口，包括 Linux、OS X 和 Cygwin。可移植的 OpenSSH 填充了其他地方不可用的 OpenBSD API，为更多操作系统添加了 sshd 沙箱，并包括对操作系统本机身份验证和审计的支持（例如使用 PAM）。

OpenSSH 主页：https://www.openssh.com/

OpenSSH 代码：https://github.com/openssh/openssh-portable

OpenSSH Wiki：https://en.wikipedia.org/wiki/OpenSSH

### OpenVPN 10.1k

OpenVPN是一种虚拟专用网络(VPN) 系统，它实现了在路由或桥接配置和远程访问设施中创建安全点对点或站点到站点连接的技术。它实现了客户端和服务器应用程序。

OpenVPN 允许对等方使用预先共享的密钥、证书或用户名/密码相互验证身份。当在多客户端服务器配置中使用时，它允许服务器使用签名和证书颁发机构为每个客户端发布身份验证证书。

OpenVPN 代码：https://github.com/OpenVPN/openvpn

OpenVPN 代码：https://github.com/OpenVPN/openvpn

OpenVPN 代码：https://github.com/OpenVPN/openvpn

OpenVPN Wiki：https://en.wikipedia.org/wiki/OpenVPN

### OpenLTE 211

OpenLTE是3GPP LTE规范的开源实现。在当前版本中，它包括一个内置简单演进分组核心的eNodeB，以及一些基于GNU Radio的用于扫描和记录LTE信号的工具。

OpenLTE 广泛用于各种项目，从部署定制的 4G 蜂窝网络、评估5G NR和 4G LTE 共存的动态频谱共享到现有电信网络的渗透测试。

OpenLTE 主页：https://openlte.sourceforge.net/

OpenLTE 代码：https://github.com/mgp25/OpenLTE

OpenLTE Wiki：https://en.wikipedia.org/wiki/OpenLTE

### OpenBTS 800

OpenBTS（开放基站收发站）是一个基于软件的GSM接入点，允许标准 GSM 兼容移动电话用作IP 语音(VoIP) 网络中的SIP端点。OpenBTS 是由Range Networks开发和维护的开源软件。

OpenBTS 的公开发布因其是行业标准 GSM协议栈下三层的第一个自由软件实现而引人注目。它是用C++编写的，并根据GNU Affero 通用公共许可证第 3 版的条款作为免费软件发布。

OpenBTS 代码：https://github.com/PentHertz/OpenBTS

OpenBTS Wiki：https://en.wikipedia.org/wiki/OpenBTS

### OpenMQTTGateway 3.5k

OpenMQTTGateway 旨在将各种技术和协议统一到单个固件中。这减少了对多个物理桥的需求，并简化了广泛使用的MQTT协议下的各种技术。

OpenMQTTGateway 代码：https://github.com/1technophile/OpenMQTTGateway

## 13.Open-Sim/仿真

### OpenMM 1.4k

OpenMM是一个用于在各种硬件架构上执行分子动力学模拟的库。它于 2010 年 1 月首次发布，[1]由斯坦福大学Vijay S. Pande实验室的 Peter Eastman 编写。

它因其在Folding@home项目的core22内核中的实现而闻名。Core22 也是由 Pande 实验室开发的，它使用 OpenMM通过CUDA和OpenCL在GPU上执行蛋白质动力学模拟。

OpenMM 代码：https://github.com/openmm/openmm

OpenMM 主页：https://openmm.org/

OpenMM 文档：http://docs.openmm.org/7.0.0/userguide/index.html

OpenMM Wiki：https://en.wikipedia.org/wiki/OpenMM

### OpenFOAM 1.4k

OpenFOAM 是 OpenFOAM 基金会发布的免费开源计算流体动力学 (CFD) 软件包。它在工程和科学的大多数领域拥有来自商业和学术组织的庞大用户群。OpenFOAM 具有广泛的功能，可以解决从涉及化学反应、湍流和传热的复杂流体流动到固体动力学和电磁学的任何问题。

OpenFOAM 代码：https://github.com/OpenFOAM/OpenFOAM-dev

### OpenSCAD 6.5k

OpenSCAD 是一款用于创建实体 3D CAD 对象的软件。它是免费软件，适用于 Linux/UNIX、MS Windows 和 macOS。

OpenSCAD 代码：https://github.com/openscad/openscad

### OpenFermion 1.5k

OpenFermion 是一个开源库，用于编译和分析量子算法以模拟费米子系统，包括量子化学。除其他功能外，该版本还具有用于获取和操作费米子和量子位哈密顿量表示的数据结构和工具

OpenFermion 代码：https://github.com/quantumlib/OpenFermion

### OpenKM

OpenKM 是一个文档管理系统 (DMS)，也称为企业内容管理 (ECM)、电子文档和记录管理系统 (EDRMS) 或内容管理系统 (CMS)。

OpenKM 可以存储、管理和跟踪电子文档以及使用文档扫描仪或其他方式捕获的纸质信息的图像。OpenKM 允许您的组织控制电子文档的生成、存储和管理。OpenKM 将所有必要的文档管理、协作和高级搜索功能集成到一个直观且易于使用的系统中。该应用程序包括用于定义用户角色、设置访问控制、设置用户配额、保护文档和设置自动化的管理工具。

OpenKM 主页：https://www.openkm.us/


## 14.Open-Infra/基础设施

### OpenShift 8.4k

OpenShift是红帽开发的一系列容器化软件产品。其旗舰产品是OpenShift 容器平台——一个混合云平台即服务，围绕Linux 容器构建，由Kubernetes在红帽企业 Linux的基础上编排和管理。

该系列的其他产品通过不同的环境提供该平台：OKD 作为社区驱动的上游（类似于Fedora是 Red Hat Enterprise Linux 的上游），多种部署方法可用，包括：自我管理、ROSA 下的云原生（AWS WS上的Red Hat OpenShift服务）、

AWS 、Azure和IBM Cloud上的ARO（Azure R ed Hat O penShift）和RHOIC（IBM Cloud上的 Red H at O penShift ），OpenShift Online 为软件即服务，而 OpenShift Dedicated 作为托管服务。

OpenShift Wiki：https://en.wikipedia.org/wiki/OpenShift

OpenShift 代码：https://github.com/openshift

### OpenGrok 4.3k

OpenGrok是一个源代码 交叉引用和搜索引擎。它帮助程序员搜索、交叉引用和导航源代码树，以帮助理解程序。 它可以读取程序 文件格式和版本控制历史记录，例如Monotone、Subversion、Mercurial、Git、ClearCase、Perforce、AccuRev、Razor和Bazaar

OpenGrok 代码：https://github.com/oracle/opengrok

OpenGrok 主页：https://oracle.github.io/opengrok/

OpenGrok Wiki：https://en.wikipedia.org/wiki/OpenGrok

### OpenSC 2.4k

OpenSC 提供了一组与智能卡配合使用的库和实用程序。它的主要重点是支持加密操作的卡，并促进它们在身份验证、邮件加密和数字签名等安全应用中的使用。

OpenSC 代码：https://github.com/OpenSC/OpenSC

OpenSC 主页：https://opensc.org/

OpenSC Wiki：https://github.com/OpenSC/OpenSC/wiki

### OpenDevOps 3.6k

OpenDevOps 代码：https://github.com/opendevops-cn/opendevops

### OpenWhisk 6.4k

OpenWhisk 是一个用于构建云应用程序的无服务器功能平台。OpenWhisk 提供了丰富的编程模型，用于从函数创建无服务器 API、将函数组合到无服务器工作流程以及使用规则和触发器将事件连接到函数。

OpenWhisk 代码：https://github.com/apache/openwhisk

### OpenMetaData 4.3 ts

OpenMetadata是一个统一的发现、可观察和治理平台，由中央元数据存储库、深入的沿袭和无缝团队协作提供支持。它是发展最快的开源项目之一，拥有充满活力的社区，并被各行业垂直领域的众多公司采用。

OpenMetadata 基于开放元数据标准和 API，支持各种数据服务的连接器，支持端到端元数据管理，让您可以自由地释放数据资产的价值。

OpenMetaData 代码：https://github.com/open-metadata/OpenMetadata

### OpenStack

OpenStack 是一个云操作系统，可控制整个数据中心内的大量计算、存储和网络资源，所有资源均通过具有通用身份验证机制的 API 进行管理和配置。还提供仪表板，使管理员能够进行控制，同时使用户能够通过 Web 界面配置资源。除了标准的基础设施即服务功能之外，其他

OpenStack 主页：https://www.openstack.org/software

### OpenRefine 10.5k

OpenRefine 是一个基于 Java 的强大工具，可让您加载数据、理解数据、清理数据、协调数据，并使用来自 Web 的数据对其进行扩充。一切都来自网络浏览器以及您自己计算机的舒适和隐私。

OpenRefine 代码：https://github.com/OpenRefine/OpenRefine

OpenRefine 主页：https://forum.openrefine.org/

### OpenSSL 24.4k

OpenSSL 是一个强大的、商业级的、功能齐全的开源工具包，适用于 TLS（以前称为 SSL）、DTLS 和 QUIC（目前仅限客户端）协议。协议实现基于全强度通用加密库，该库也可以独立使用。还包括经过验证符合 FIPS 标准的加密模块。OpenSSL 源自 Eric A. Young 和 Tim J. Hudson 开发的 SSLeay 库。

OpenSSL 代码：https://github.com/openssl/openssl

### OpenWPM

OpenWPM 是一个网络隐私测量框架，可以轻松收集数千到数百万个网站规模的隐私研究数据。OpenWPM 构建在 Firefox 之上，并由 Selenium 提供自动化功能。它包括几个用于数据收集的挂钩。查看下面的仪器部分了解更多详细信息。

OpenWPM 代码：https://github.com/openwpm/OpenWPM

## 15.Open-IT/信息技术

### OpenCraft 7.3k php

OpenCart 是一个为在线商家提供的免费开源电子商务平台。OpenCart 为建立成功的在线商店提供了专业且可靠的基础。

OpenCraft 代码：https://github.com/opencart/opencart

### OpenCharKit 9k py

OpenChatKit 提供了强大的开源基础，可以为各种应用程序创建专用和通用模型。该套件包括一个指令调整的语言模型、一个审核模型和一个可扩展的检索系统，用于包含来自自定义存储库的最新响应。

OpenChatKit 代码：https://github.com/togethercomputer/OpenChatKit

### OpenChat 5.1k js

OpenChat 是一个日常用户聊天机器人控制台，可简化大型语言模型的使用。随着人工智能的进步，这些模型的安装和使用已经变得势不可挡。OpenChat 旨在通过提供两步设置过程来创建全面的聊天机器人控制台来应对这一挑战。它充当管理多个定制聊天机器人的中心枢纽。

OpenChat 代码：https://github.com/openchatai/OpenChat

### OpenEMR 2.6k php

OpenEMR是一款免费、开源的电子健康记录和医疗实践管理应用程序。它具有完全集成的电子健康记录、实践管理、日程安排、电子计费、国际化、免费支持、充满活力的社区等等。它可以在 Windows、Linux、Mac OS X 和许多其他平台上运行。

OpenEMR：https://github.com/openemr/openemr

### OpenSID 997 php

OpenSID是由开放数字村庄协会（OpenDesa）法律研究所与村庄活动家社区共同开发的村庄信息系统，用于支持村政府行政管理的职能和任务，如总行政、人口管理、财务管理、发展管理、公共服务、公共信息服务等。

OpenSID 代码：https://github.com/OpenSID/OpenSID

OpenSID 主页：https://opensid.my.id/

## 16.Open-SW/软件工程

### OpenPDF 3.3k java

OpenPDF 是一个 Java 库，用于使用 LGPL 和 MPL 开源许可证创建和编辑 PDF 文件。OpenPDF 是 iText 的 LGPL/MPL 开源继承者，并且基于 iText 4 svn 标记的一些分支。

OpenPDF：https://github.com/LibrePDF/OpenPDF

### OpenDDS 1.3k cpp

OpenDDS 是对象管理组织规范“实时系统数据分发服务”(DDS) 以及其他一些相关规范的开源 C++ 实现。这些标准定义了一组用于开发基于发布-订阅和分布式缓存模型的分布式应用程序的接口和协议。

尽管OpenDDS本身是用C++开发的，但提供了Java绑定，以便Java应用程序可以使用OpenDDS。OpenDDS 还包括对 DDS 安全性和 XTypes 规范的支持。

OpenDDS 代码：https://github.com/OpenDDS/OpenDDS

### OpenDBC 1.7k py

车辆DBC文件开源库。

OpenDBC 代码：https://github.com/commaai/opendbc 




