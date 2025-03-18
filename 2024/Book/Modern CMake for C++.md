第二版 在线的不好找。。

中文翻译版  .这个是第一版的。
1-2
https://cloud.tencent.com/developer/article/2421611
3-5
https://cloud.tencent.com/developer/article/2421607
6-8
https://cloud.tencent.com/developer/article/2421608
9-11
https://cloud.tencent.com/developer/article/2421609
12
https://cloud.tencent.com/developer/article/2421610


代码。 第一版的。
https://github.com/PacktPublishing/Modern-CMake-for-Cpp



```text
// -B 是 BuildSystem
cmake -B <build tree> -S <source tree>
// --build 是构建，而且必须第一个参数
cmake --build <build tree>
```

cmake
ctest
cpack
cmake-gui
ccmake



[[toc]]

---


# ch1 简介


为了构建本书中的示例，始终使用建议的命令

```text
cmake -B <build tree> -S <source tree>
cmake --build <build tree>
```

build tree, source tree 替换为 合适的路径， build tree 是输出目录的路径， source tree 是源码所在的路径。


CMake功能：自动化构建，遍历我们的项目树并编译所有内容。
为了避免不必要的编译，CMake会检查源码 是否 自上次运行CMake 后被修改。
- 编译可执行文件和库
- 管理依赖项
- 测试
- 安装
- 打包
- 生成文档
- 再测试一下

CMake 跨平台

CMake 本身不能构建任何东西，  它是 构建过程的 协调者， 它知道 需要做什么，如何找到合适的 工具。
CMake 的工作分为 3个阶段
- 配置 config
- 生成 generate
- 构建 build

配置阶段：读取 项目的详细信息，为 生成 阶段准备 输出目录 或 构建树
1. CMake 创建一个 空的构建树， 收集 工作环境的信息，如 架构，可用的编译器，链接器，归档器。 并检查一个简单的测试程序(。估计是它内置的一个 helloworld) 是否可以编译。
2. 解析 并执行 CMakeLists.txt 。 这个文件 告诉 CMake： 项目结构，目标，依赖项(库和其他CMake包)

生成阶段，生成一个 构建系统。 构建系统是 为 其他构建工具 定制的 配置文件。

构建阶段：运行适当的 构建工具，使用 编译器，链接器，静态和动态分析工具，测试框架，报告工具 等 来生成目标。


## helloworld

hello.cpp 需要的 CMakeLists.txt 如下
```text
cmake_minimum_required(VERSION 3.20)
project(hello)
add_executable(Hello hello.cpp)
```

执行命令
`cmake -B buildtree.`
。。 ？ 这个 . 是什么意思？
.buildtree 应该是目录，  . 应该不需要。
。。看下面的， 估计是  buildtree/ ， / 打成 . 了

`cmake --build buildtree/`

`./buildtree/Hello`
执行程序

---


安装
docker，win，linux，macos

自己编译
```text
$ wget https://github.com/Kitware/CMake/releases/download/v3.20.0/cmake-3.20.0.tar.gz
$ tar xzf cmake-3.20.0.tar.gz
$ cd cmake-3.20.0
$ ./bootstrap
$ make
$ make install
```


CMake 一共有5个工具
- cmake，配置，生成，构建 项目
- ctest，测试，报告结果
- cpack，生成 安装程序 和源代码包
- cmake-gui，cmake的图形化界面
- ccmake，cmake的 控制台图形化界面

`cmake -S ./project -B ./build`


---

## 构建


当前目录中构建， 源码从上一级目录中获取 (-S 可选)
`cmake -S ..`

在 ./build 中构建，源码 从当前目录中获得
`cmake -B build`

---

## 生成器的选项

一般情况不需要，CMake 会自动选择， 除非你 安装了 多个 生成器

可以通过 环境变量`CMAKE_GENERATOR` 或直接命令行 来指定 生成器

`cmake -G <generator-name> <path-to-source>`

一些生成器 支持 更详细的 指定。 还有相应的环境变量： `CMAKE_GENERATOR_TOOLSET` `CMAKE_GENERATOR_PLATFORM` 。 也可以命令行
```text
cmake -G <generator-name> 
      -T <toolset-spec> -A <platform-name>
      <path-to-source>
```

==检查可用生成器==
`cmake --help`
。找 the following generators are available on this platform:


---

## 缓存选项

CMake 在配置阶段 查到的 各种信息，存储在 构建目录的 `CMakeCache.txt` 中

我们可以 预先填充缓存信息
`cmake -C <initial-cache-script> <path-to-source>`
我们要提供一个脚本，里面只包含 `set()` 命令列表， 用来 初始化 空构建树的 变量

预先填充缓存信息 也可以 命令行
`cmake -D <var>[:<type>]=<value> <path-to-source>`
`[]` 中是可选的， type可以是： BOOL, FILEPATH, PATH, STRING, INTERNAL。如果省略，那么是 已有变量的类型， 否则是 UNINITIALIZED

---

对于单配置生成器 (如 make，ninja) ，需要在配置阶段指定 `CMAKE_BUILD_TYPE`, 并为 每个类型的 配置 生成一个 单独的构建树 ： Debug，Release，MinSizeRel, RelWithDebInfo

`cmake -S . -B build -D CMAKE_BUILD_TYPE=Release`

多配置生成器 在构建阶段进行配置。

---

查看变量  。？ 需要核实
-D添加时，如果不指定 类型，将不可见
`cmake -L[A][H] <path-to-source>`

---

删除变量。
`cmake -U <globbing_expr> <path-to-source>`
支持通配符

---

## 调试 与 跟踪 选项

要获得 有关变量，命令， 宏， 其他设置的 一般信息

`cmake --system-information [file]`

file 是 保存 output 的文件

---

`message()` 来报告构建过程的 详细信息。 cmake 根据当前日志级别 (默认是 STATUS) 过滤这些 日志输出。
下面指定 感兴趣的日志级别
`cmake --log-level=<level>`
`<level>` 可以是 ERROR, WARNING, NOTICE, STATUS, VERBOSE, DEBUG, TRACE

可以在 `CMAKE_MESSAGE_LOG_LEVEL` 缓存变量中 永久指定

---

可以使用 message() 调用。 CMAKE_MESSAGE_CONTEXT 变量 可以向栈一样使用。
`cmake --log-context <path-to-source>`

---

大杀器： 跟踪模式， 将打印 每个命令 及 它的文件名 和 行号，参数。
启用：
`cmake --trace`

---

## 预设选项

开发者 提供 指定一些默认值的 `CMakePresets.json` 

列出所有可用的预设
`cmake --list-presets`

使用 预设
`cmake --preset=<preset>`


## 构建项目

生成 构建树后，就可以 运行构建工具

构建的语法
`cmake --build <dir> [<options>] [-- <build-tool-options>]`

大多数情况下，下面的就可以了
`cmake --build <dir>`

cmake 需要直到 生成的 构建树的 位置， 就是 `-B` 的路径

---

### 并行构建

下面2个命令参数都可以。

```text
cmake --build <dir> --parallel [<number-of-jobs>]
cmake --build <dir> -j [<number-of-jobs>]
```

还可以通过 `CMAKE_BUILD_PARALLEL_LEVEL` 环境变量指定。

---

### target 选项

每个项目都由 一个或多个 称为 target 的 部分组成。  
通常，我们要构建所有 target。 但是有时 只需要构建一部分

`cmake --build <dir> --target <target1> -t <target2> ...`

重复 -t 来指定多个。

通常 clean 这个 target 是不会构建的， 因为它会删除 所有构建的内容。 调用：
`cmake --build <dir> -t clean`

如果你想 先清理 然后 正常构建
`cmake --build <dir> --clean-first`

---

### 多配置生成器的 选项

`cmake --build <dir> --config <cfg>`

cmake 默认使用 Debug


### 调试选项

输出 详细日志

```text
cmake --build <dir> --verbose
cmake --build <dir> -v
```

`CMAKE_VERBOSE_MAKEFILE` 缓存变量 也可以 同样的效果。


## 安装项目

用户可以 安装到系统中。 这意味着 文件被复制到 正确的目录，安装库。 

`cmake --install <dir> [<options>]`


### 多配置生成器选项

`cmake --install <dir> --config <cfg>`

---

### 组件选项

开发者 可能 会选择 将 项目 拆分为 可以独立安装的组件。 类似于 application, docs, extra-tools 

要安装单个组件，使用
`cmake --install <dir> --component <comp>`


### 权限选择

类 unix 平台，可以使用下面的选项 指定 安装目录的 默认权限 ，格式是 `u=rwx,g=rx,o=rx`

`cmake --install <dir> --default-directory-permissions <permissions>`

---

### 安装目录选项

为项目中 指定的 安装路径 添加一个 自定义前缀。 

`cmake --install <dir> --prefix <prefix>`

window无效。

### 调试选项

查看 安装阶段的 详细输出， 下面2个都可以。

```text
cmake --install <dir> --verbose
cmake --install <dir> -v
```

也可以通过 设置 `VERBOSE` 环境变量 来 达到相同效果。

## 运行脚本


```text
cmake [{-D <var>=<value>}...] -P <cmake-script-file> 
      [-- <unparsed-options>...]
```

---

运行命令行工具

少数情况下， 我们需要 以平台无关的方式 运行单个指令， 可能是 复制文件 或计算校验和。

cmake 提供了一种模式，可以在 不同平台上 一致地 执行最常用的命令

`cmake -E <command> [<options>]`

可以使用 `cmake -E` 调出所有 可用的命令。 
。。 几十个呢

如果需要更复杂的行为，可以将其封装到脚本中，然后 -P 运行

---

## 获取帮助

`cmake --help[-<topic>]`

---

## CTest

CTest 是在更高层次的抽象中封装 CMake

更新、运行各种测试、将项目状态报告给外部仪表板以及运行编写在 CMake 语言中的脚本

CTest 标准化了使用 CMake 构建的解决方案的测试运行和报告。

直接在 生成的 构建树中 调用 `ctest`


## CPack

cmake-gui

ccmake
只能 类 unix
需要单独安装


## 浏览项目文件

cmake 使用很多 文件来管理其项目。 

我们试图了解 每个文件的作用。


### 源代码树

你的项目所在的目录 (也称为 项目根)， 包含 所有 c++源码 和 CMake项目文件

需要注意
- 必须在 顶部目录提供一个 `CMakeLists.txt`
- 应该使用 版本控制器 进行管理
- 此目录的 路径 由 用户通过 cmake -S 指定
- 避免 在 cmake代码中 硬编码 绝对路径

### 构建树

使用此目录 来存储构建过程中生成的 所有内容： 项目的工件，短暂配置，缓存，构建日志 等

需要注意
- 你的二进制文件 将在这里创建，如 可执行文件，库文件， 用于最终链接的 对象文件 和 归档文件
- 不要将 此目录 纳入 版本控制器
- cmake 建议 源外构建，或 生成目录 与 所有源文件 分离 的构建。这样， 避免 临时文件，系统特定文件 污染 源代码
- 如果提供了 源码路径 如 `cmake -S ../project ./`， 则使用 `-B` 作为最后一个参数 来指定目录
- 建议 你的项目 包含一个 安装阶段， 允许 你将 最终文件 放到 系统的 正确位置，以便可以 删除 构建的 所有临时文件


### 列表文件 CMakeLists.txt

包含 CMake 语言的文件 称为 列表文件

可以通过 `include()` , `find_package()` , 或 间接地通过 `add_subdirectory()` 来相互包含

cmake 不强制 命名，但通过 是 .cmake 扩展名

最重要的 是 CMakeLists.txt， 这是 配置阶段 执行的 第一个文件。  位于 源树的 顶部

当 cmake 遍历 源树 并包含 不同的 列表文件时， 下面的 变量将被设置 `CMAKE_CURRENT_LIST_DIR`, `CMAKE_CURRENT_LIST_FILE`, `CMAKE_PARENT_LIST_FILE`, `CMAKE_CURRENT_LIST_LINE`


CMakeLists.txt，只要包含2个命令
- `cmake_minimum_required(VERSION x.xx)`
- `project(<name> <OPTIONS>)`

---

随着 系统扩大，需要划分为 更小的单元。 
比如，你的项目结构：
```text
CMakeLists.txt
api/CMakeLists.txt
api/api.h
api/api.cpp
```

此时 CMakeLists.txt：
```text
cmake_minimum_required(VERSION 3.20)
project(app)
message("Top level CMakeLists.txt")
add_subdirectory(api)
```

`add_subdirectory(api)` 从 api目录中 寻找 CMakeLists.txt

---

### CMakeCache.txt

EXTERNAL 部分 是为了让 用户修改

INTERNAL 由cmake管理。不建议 用户修改。

注意点
- 你可以通过 cmake 手动管理这个文件
- 删除此文件，可以将项目重置为 其默认配置， 它将重新生成
- 缓存变量 可以从 列表文件中读写。


### 包的配置文件

外部包

- 配置文件，包含 如何使用库，头文件，辅助工具，  有时，它们暴露 CMake宏， 可以在你的项目中使用
- 使用 `find_package()` 来包含 包
- 描述 包的 cmake文件名是 `<package name>-config.cmake` 和 `<package name>Config.cmake`
- 使用包时，可以指定 需要的包的 版本。 会在 `<config>Version.cmake` 中检查这个版本
- 配置文件由 cmake 生态系统的 包供应商提供。
- cmake 提供了一个 包注册表，用于在 系统范围内 和 每个用户处 存储包


cmake_install.cmake
CTestTestfile.cmake
CPackConfig.cmake
这些文件 由 cmake 在 生成阶段 在构建树 中生成。  不建议 手动编辑。

CMakePresets.json
CMakeUserPersets.json
可以通过 gui选择预设， 或使用 命令行 `--list-presets`, `--preset=<preset>` 来选择 预设。

预设以相同的 json 保存在 上面的 2个文件中
一个是 项目作者 提供的 官方预设
一个是 用户自定义的 预设

```json
{
  "version": 1,
  "cmakeMinimumRequired": {
    "major": 3, "minor": 19, "patch": 3
  },
  "configurePresets": [ ],
  "vendor": {
    "vendor-one.com/ExampleIDE/1.0": {
      "buildQuickly": false
    }
  }
}
```

```json
{
  "name": "my-preset",
  "displayName": "Custom Preset",
  "description": "Custom build - Ninja",
  "generator": "Ninja",
  "binaryDir": "${sourceDir}/build/ninja",
  "cacheVariables": {
    "FIRST_CACHE_VARIABLE": {
      "type": "BOOL", "value": "OFF"
    },
    "SECOND_CACHE_VARIABLE": "Ninjas rock"
  },
  "environment": {
    "MY_ENVIRONMENT_VARIABLE": "Test",
    "PATH": "$env{HOME}/ninja/bin:$penv{PATH}"
  },
  "vendor": {
    "vendor-one.com/ExampleIDE/1.0": {
      "buildQuickly": true
    }
  }
},
```

```json
{
  "name": "my-preset-multi",
  "inherits": "my-preset",
  "displayName": "Custom Ninja Multi-Config",
  "description": "Custom build - Ninja Multi",
  "generator": "Ninja Multi-Config"
}
```

```text
预设被定义为具有以下字段的映射：

     name：这是一个必需的字符串，用于标识预设。它必须对机器友好，并且在两个文件中唯一。
     
     Hidden：这是一个可选的布尔值，用于隐藏预设，使其不在 GUI 和命令行列表中显示。这样的预设可以是另一个预设的父预设，并且不需要提供除其名称以外的任何内容。
     
     displayName：这是一个可选的字符串，有一个人类可读的名字。
     
     description：这是一个可选的字符串，用于描述预设。
     
     Inherits：这是一个可选的字符串或预设名称数组，用于从其中继承。在冲突的情况下，早期预设的值将被优先考虑，每个预设都可以覆盖任何继承的字段。此外，CMakeUserPresets.json可以继承项目预设，但反之则不行。
     
     Vendor：这是一个可选的供应商特定值的映射。它遵循与根级vendor字段相同的约定。
     
     Generator：这是一个必需或继承的字符串，用于指定预设要使用的生成器。
     
     architecture和toolset：这些是用于配置支持这些选项的生成器的可选字段（在生成项目构建系统部分提到）。每个字段可以是一个简单的字符串，或者一个带有value和strategy字段的哈希表，其中strategy是set或external。当strategy字段配置为set时，将设置字段值，如果生成器不支持此字段，则会产生错误。配置为external意味着字段值是为外部 IDE 设置的，CMake 应该忽略它。
     
     binaryDir：这是一个必需或继承的字符串，提供了构建树目录的路径（相对于源树是绝对路径或相对路径）。它支持宏扩展。
     
     cacheVariables：这是一个可选的缓存变量映射，其中键表示变量名。接受的值包括null、"TRUE"、"FALSE"、字符串值，或具有可选type字段和必需value字段的哈希。value可以是"TRUE"或"FALSE"的字符串值。除非明确指定为null，否则缓存变量会通过并集操作继承——在这种情况下，它将保持未设置。字符串值支持宏扩展。
     
     Environment: 这是一个可选的环境变量映射，其中键表示变量名。接受的值包括null或字符串值。除非明确指定为null，否则环境变量会通过并集操作继承——在这种情况下，它将保持未设置。字符串值支持宏扩展，变量可以以任何顺序相互引用，只要没有循环引用即可。
     

以下宏将被识别和评估：

     ${sourceDir}：这是源树的位置。
     
     ${sourceParentDir}：这是源树父目录的位置。
     
     sourceDirName:这是{sourceDir}的最后一个文件名组件。例如，对于/home/rafal/project，它就是project。 
     ${presetName}: 这是预设的名称字段的值。
     
     ${generator}：这是预设的生成器字段的值。
     
     dollar:这是一个字面意义上的美元符号（）。 
     $env{<variable-name>}：这是一个环境变量宏。如果预设中定义了该变量，它将返回预设中的变量值；否则，它将从父环境返回值。请注意，预设中的变量名是区分大小写的（与 Windows 环境不同）。
     
     penv<variable−name>：这个选项与
     $vendor{<macro-name>}：这使得供应商能够插入自己的宏。
```

---

git 中忽略

```text
build_debug/
build_release/
# Generated and user files
**/CMakeCache.txt
**/CMakeUserPresets.json
**/CTestTestfile.cmake
**/CPackConfig.cmake
**/cmake_install.cmake
**/install_manifest.txt
**/compile_commands.json
```


## 发现脚本和模块

为了配置项目构建，cmake提供了 平台无关的编程语言。 可以用来 编写 随项目提供 或完全独立的 脚本

`-P` 执行脚本  `cmake -P script.cmake`, 脚本内容如下

```text
# An example of a script
cmake_minimum_required(VERSION 3.20.0)
message("Hello world")
file(WRITE Hello.txt "I am writing to a file")
```


---

### 模块

cmake项目可以 使用 外部模块 来增强其功能。 

模块是 cmake语言编写。 包含 宏定义，变量，执行各种功能的 命令。

cmake 包含了 上百个 不同的 模块。  你可以 自己编写模块。

要使用模块 ，调用 `include(<module>)`

```text
cmake_minimum_required(VERSION 3.20.0)
project(ModuleExample)
include (TestBigEndian)
TEST_BIG_ENDIAN(IS_BIG_ENDIAN)
if(IS_BIG_ENDIAN)
 message("BIG_ENDIAN")
else()
 message("LITTLE_ENDIAN")
endif()
```

---

查找模块

find_package() 来查找模块。






# ch2 CMake语言

本章内容
- cmake 语言基础语法
- 使用变量
- 使用列表
- 理解cmake中的控制结构
- 有用的命令

---

## 基础语法

执行 从 源树根文件(CMakeLists.txt) 或 通过参数传递给 cmake (`cmake -P script.cmake`)   开始

### 注释

`#` 单行注释

`[]` 多行注释，可以嵌套.。 前面的 # 是可选的。  通过 不同的 = 数量 来匹配

```text
# single-line comments start with a hash sign "#"
# they can be placed on an empty line
message("Hi"); # or after a command like here.
#[=[ 
bracket comment
  #[[
    nested bracket comment
  #]]
#]=]
```

可以在 多行注释前面 添加一个 # 来取消注释
```text
##[=[ this is a single-line comment now      单行注释
no longer commented                           非注释
  #[[
    still, a nested comment                  注释
  #]]
#]=] this is a single-line comment now    单行注释
```


### 命令调用

需要 命令名字 括号   空格分割的参数  括号

命令不区分大小写，但有一个 约定，在命令名称中 使用 下划线相连的小写字母单词。

cmake中的 命令调用 不是表达式， 不能将 一个命令 作为 另一个命令的 参数。 因为 括号里的 内容 全是 命令的参数。

cmake 命令 调用结束时 不需要 ;

不能在 方括号 注释后面 放置命令

```text
command(argument1 "argument2" argument3) # comment
[[ multiline comment ]] 
```


命令分为3类
- 脚本命令， 始终可用， 用于改变 命令处理器的 状态，访问变量，并影响其他命令和环境
- 项目命令， 在项目中可用，用于 操作项目状态和 构建目标
- CTest命令， 在 CTest脚本中可用。用于管理测试


---

几乎每个命令都依赖于 语言的 其他元素： 变量，条件语句，命令行参数。

### 命令参数

CMake提供 3种参数类型
- 方括号参数
- 引号参数
- 未引用的参数

方括号参数不进行评估，因为 它们用于将 多行字符串 作为 单个参数 传递给命令， 而不做任何更改。
这些参数的 结构 和注释一样 `[=[`开头 `]=]`结尾。 等号数量(>=0)必须匹配。

```cmake
message([[multiline
  bracket
  argument
]])
message([==[
  because we used two equal-signs "=="
  following is still a single argument:
  { "petsArray" = [["mouse","cat"],["dog"]] }
]==])
```

---

引号参数

类似于 字符串。  有转义

不同：
引号参数 可以跨多行， 并且会把 插值变量 计算出来。

```text
message("1\. escape sequence: \" \n in a quoted argument")
message("2\. multi...
  line")
message("3\. and a variable reference: ${CMAKE_VERSION}")
```

---

未引用的参数

比较少见。

`;` cmake中是 分隔符， 拆分参数的。

```text
message(a\ single\ argument)
message(two arguments)
message(three;separated;arguments)
message(${CMAKE_VERSION})  # a variable reference
message(()()())            # matching parentheses 
```

```shell
$ cmake -P chapter02/01-arguments/unquoted.cmake
a single argument
twoarguments
threeseparatedarguments
3.16.3
()()() 
```

- a single argument 有 显式转义，被正确打印
- twoarguments，threeseparatedarguments， 被黏在一起， 因为 message() 本身不会添加任何空格  。。。所以 上面是 1个参数， 这里是 2个，3个参数？

。。不知道讲了什么。。


---

## ==在 cmake中处理变量==

相当复杂。 因为 变量有3种， 它们还存在于不同的 作用域中。 有着特定的 规则： 一个作用域 如何影响另一个。

大多数情况下， 对这些 规则的 误解  是 错误和头痛的 来源。

变量名是 区分大小写的 ，可以包含任何字符
所有的变量内部 都是以 字符串的形式存储，
基本操作时 set(), unset() , 还有其他可以影响变量的命令 如 string()， list()

---

设置变量，只需要 set 即可

```text
set(MyString1 "Text1")
set([[My String2]] "Text2")
set("My String 3" "Text3")
message(${MyString1})
message(${My\ String2})
message(${My\ String\ 3})
```

方括号，引号 的 变量名 允许 包含 空格。  但是 引用时，必须 使用 反斜杠 转义。
因此 建议 变量名 只包含 `字母  数字  -  _` 避免 `CMAKE_   _CMAKE_   _` 开头


取消变量
`unset(MyString1)`

---

变量引用

`message(${MyString1})` ， 遍历 作用域 堆栈， 将 `${MyString1}` 替换为一个值， 如果没有找到，则为 一个空字符串 , 不会产生错误。

遇到 `${MyOuter{MyInner}}` 时 会先搜索 MyInner 的值， 展开后， 继续 展开 MyOuterXXX 的值


- `${}`，用于 引用 普通 或缓存变量
- `$ENV{}`， 用于引用 环境变量
- `$CACHE{}`， 用于引用 缓存变量


注意：
在 -- 后通过 命令行 向 脚本 传递参数， 值将存储在 `CMAKE_ARGV<n>` 变量中， 传递的 参数数量 将在 `CMAKE_ARGC` 中。


---

使用环境变量


cmake 会复制 启动cmake 时 环境的变量，并 使它们在一个 单独的 全局作用域中可用。  

要引用这些变量 使用 `$ENV{name}`

cmake还允许你 设置set() 和 取消设置unset() 这些变量， 但更改只会对 运行中的 cmake 过程中的 本地副本 修改，不会影响 实际 系统环境。 
这些更改也不会对 后续的 构建，测试 运行，可见。

修改或创建 环境变量
`set(ENV{var} value)`

`set(ENV{CXX} "clang++")`

清除环境变量
`unset(ENV{var})`

---

一些环境变量 会影响 CMake 的行为， 比如 CXX 指定 用于 编译C++文件的 编译器。
完整列表： https://cmake.org/cmake/help/latest/manual/cmake-env-variables.7.html


---

如果你将 ENV 变量作为命令的参数， 它们的值 将在 构建系统的 生成过程中 进行插值。 这意味着 ==它们将被编织到 构建树中，更改构建阶段的环境 将没有任何效果==

```text
cmake_minimum_required(VERSION 3.20.0)
project(Environment)
message("generated with " $ENV{myenv})
add_custom_target(EchoEnv ALL COMMAND echo "myenv in build
  is" $ENV{myenv})
```

```shell
#!/bin/bash
export myenv=first
echo myenv is now $myenv
cmake -B build .
cd build
export myenv=second
echo myenv is now $myenv
cmake --build .
```

```shell
$ ./build.sh | grep -v "\-\-"
myenv is now first
generated with first
myenv is now second
Scanning dependencies of target EchoEnv
myenv in build is first
Built target EchoEnv
```


---

### 使用缓存变量

`$CACHE{name}`

`set(<variable> <value> CACHE <type> <docstring> [FORCE])`

CACHE， FORCE 是关键字。

将 CACHE 指定为 set() 参数 意味着 我们打算 改变 在配置阶段提供的内容，并强制 提供变量 type， docstring
type可以： BOOL, FILEPATH, PATH, STRING, INTERAL
docstring 只是一个标签。

`set(FOO "BAR" CACHE STRING "interesting value")`
如果变量在 缓存中存在 ，上述调用将没有 永久效果。
如果 不在缓存中存在，或 指定了 FORCE，则 被 持久化
`set(FOO "BAR" CACHE STRING "interesting value" FORCE)`

设置缓存变量后， 同名的正常变量 会被移除。

---

### 变量作用域

可能是 整个cmake 语言概念中 最难的部分。 可能是因为 我们习惯了 支持 命名空间 和 作用域操作符 的高级语言。

作用域作为一个 一般概念 是为了 将不同层次的 抽象分离，以便 当 调用一个 用户定义的函数时，函数中设置的 变量是 局部的。 这些 局部变量 即使与全局变量 名称相同，也不会 影响全部变量。
如果需要，函数应该 对 全局变量也有 读写权限。

CMake 有2个作用域
- 执行 `function()`
- 从 `add_subdirectory()` 开始，在嵌套目录中 执行 CMakeLists.txt， 

创建 嵌套作用域时， cmake 将当前作用域的 所有变量的副本 填充它。 ==后续的命令(set,unset)会影响这些副本==。  
但 ==一旦 嵌套作用域执行完，所有的副本都被删除，并恢复 原始的 父作用域==。


`${}`， 会 从当前作用域 搜索， 如果找不到，那么 搜索 缓存变量。

要修改 父作用域，需要 `PARENT_SCOPE`, 局限是 不能访问 超过一级的变量。  
需要注意， parent_scope 不改变 当前作用域中的 变量

```javascript
set(MyVariable "New Value" PARENT_SCOPE)
unset(MyVariable PARENT_SCOPE) 
```

```js
function(Inner)
  message("  > Inner: ${V}")
  set(V 3)
  message("  < Inner: ${V}")
endfunction()
function(Outer)
  message(" > Outer: ${V}")
  set(V 2)
  Inner()
  message(" < Outer: ${V}")
endfunction()
set(V 1)
message("> Global: ${V}")
Outer()
message("< Global: ${V}")
```

```js
> Global: 1
 > Outer: 1
  > Inner: 2
  < Inner: 3
 < Outer: 2
< Global: 1
```

---

## 使用列表

要存储 ;  需要进行 转义。

要创建列表，使用 set().  
`set(myList a list of five elements)`

由于列表的存储方式，下面的命令 效果完全一样
`set(myList "a;list;of;five;elements")`
`set(myList a list "of;five;elements")`

cmake 会自动解包列表。
`message("the list is:" ${myList}) `
message 收到 6个字符， "the list is:", "a", "list","of","five","elements" 。  并且 输出的时候， message 不会添加空格


---

CMake 提供了一个list()命令，该命令提供了许多子命令来读取、搜索、修改和排序列表

```js
list(LENGTH <list> <out-var>)
list(GET <list> <element index> [<index> ...] <out-var>)
list(JOIN <list> <glue> <out-var>)
list(SUBLIST <list> <begin> <length> <out-var>)
list(FIND <list> <value> <out-var>)
list(APPEND <list> [<element>...])
list(FILTER <list> {INCLUDE | EXCLUDE} REGEX <regex>)
list(INSERT <list> <index> [<element>...])
list(POP_BACK <list> [<out-var>...])
list(POP_FRONT <list> [<out-var>...])
list(PREPEND <list> [<element>...])
list(REMOVE_ITEM <list> <value>...)
list(REMOVE_AT <list> <index>...)
list(REMOVE_DUPLICATES <list>)
list(TRANSFORM <list> <ACTION> [...])
list(REVERSE <list>)
list(SORT <list> [...])
```

实际，大多数时候，并不需要使用 列表。

---

## cmake 控制结构

- 条件块 if
- 循环 while, foreach
- 命令定义 marco, function

---

条件块

`if, elseif, else, endif`

```js
if(<condition>)
  <commands>
elseif(<condition>) # optional block, can be repeated
  <commands>
else()              # optional block
  <commands>
endif()
```

---

逻辑运算符

支持， NOT，AND，OR

() 修改 运算符优先级

。。不过没说，默认是什么优先级。

---

### ==字符串，变量的评估==

==由于历史原因， cmake 会尝试 将 未引用的 参数 评估为 变量引用。 即 在条件中使用 普通变量名 等同于 ${xx}==

```js
set(VAR1 FALSE)
set(VAR2 "VAR1")
if(${VAR2})
```
会推导出 VAR1， 然后 推导为 FALSE

只有下面的 才是 布尔真 (不区分大小写)
- ON
- Y
- YES
- TRUE
- 非0数字

---

还有一个 ==陷阱： 一个未引用的参数的条件评估==， 考虑下面的代码
```js
set(FOO BAR)
if(FOO)
```
这不是 FALSE。 

cmake 在 ==未引用的变量引用方面 作为例外==。 FOO 不会被评估为 BAR 来产生 `if("BAR")` 语句 (这个是 false)。 
相反， 只有当 FOO 是下列常量(不区分大小写)时， `if(FOO)` 才是false
- off,no,false,n,ignore,notfound
- 以 -notfound 结尾的字符串
- 空字符串
- 零

所以，简单地 询问 未定义的变量 将被 评估为 false
`if (FOO)`


而事先定义一个变量 会 为 true
```js
set(FOO "FOO")
if (FOO)
```

。。。？
。所以是， 如果 无法 通过 无限推导 来获得 值， 那么 只要能 推导一次，就是 true。 
。。 如果可以 推导到值，那么就是 值
。。
。。推导 是 string 才推导， 所以 必须终止于 非 string 啊。  真的奇怪。


明确地检查 变量是否定义
```js
if(DEFINED <name>)
if(DEFINED CACHE{<name>})
if(DEFINED ENV{<name>})
```

---

### 比较值

支持的操作： `EQUAL`,`LESS`,`LESS_EQUAL`,`GREATER`,`GREATER_EQUAL`

`if (1 LESS 2)`

注意：
根据文档，如果操作数之一 不是数字，那么直接是false。   ==但是== 实际，以数字开头的 字符串 能正确工作
`if (20 EQUALS "20 GB")`

在 上面列举的 操作 前面加 `VERSION_` 前缀，按照 `major[.minor[.patch[.tweak]]]` 格式==比较软件版本==
`if (1.3.4 VERSION_LESS_EQUAL 1.4)`
缺少的部分，被视为0

对于==字典序==，需要加上 `STR`前缀 , 无_
`if ("A" STREQUAL "${B}")`

---

经常需要 比 简单比较 更高级的机制。

cmake 支持 `MATCHES`
`<变量名|字符串> MATCHES <正则表达式>`
任何匹配，都被捕获在 `CMAKE_MATCH_<n>` 中

---

简单检查

之前已经有 简单检查 ： `DEFINED`

还有：
- 如果值在列表中：`<变量名|字符串> IN_LIST <变量名>`
- 如果一个命令可以被调用 `COMMAND <命令名>`
- 如果存在 cmake 策略 `POLICY <策略 ID>`
- 如果一个 CTest 测试 是 `add_test()` 添加的 `TEST <测试名称>`
- 如果定义了一个 构建目标：`TARGET <目标名称>`

---

检查文件系统

cmake提供了许多处理文件的方法。  很少需要直接操作它们，通常 更愿意使用 高层次的方法。
附录中提供了 与文件相关的 命令。
但大多数时候，只需要以下命令
- `EXISTS <path-to-file-or-directory>`， 文件或目录 是否存在
- `<file1> IS_NEWER_THAN <file2>`， 哪个文件更新
- `IS_DIRECTORY path-to-directory`， 是否是目录
- `IS_SYMLINK file-name`， 是否是 符号链接
- `IS_ABSOLUTE path`， 是否是 绝对路径

---

### 循环

`while()`  `foreach()`  来反复执行 相同的命令集

这2个命令都支持 循环控制机制
- break()
- continue()

---

while 需要 endwhile 匹配

```js
while(<condition>)
  <commands>
endwhile()
```

---

foreach 有几个变体

需要有 endforeach


从 [0, max] (包含起止)
```js
foreach(<loop_var> RANGE <max>)
  <commands>
endforeach()
```

min,max,step
```js
foreach(<loop_var> RANGE <min> <max> [<step>])
```

处理列表
遍历的是 lists + items 组成的 列表。
```js
foreach(<loop_variable> IN [LISTS <lists>] [ITEMS <items>])
```

```js
set(MY_LIST 1 2 3)
foreach(VAR IN LISTS MY_LIST ITEMS e f)
  message(${VAR})
endforeach()
```
输出
```js
1
2
3
e
f
```

简短版本
`foreach(VAR 1 2 3 e f)`


从3.17开始，可以 `ZIP_LISTS`
简单地遍历多个列表，而不是 列表中的值
`foreach(<loop_var>... IN ZIP_LISTS <lists>)`

```js
set(L1 "one;two;three;four")
set(L2 "1;2;3;4;5")
foreach(num IN ZIP_LISTS L1 L2)
    message("num_0=${num_0}, num_1=${num_1}")
endforeach()
```
为每个列表创建一个 `num_<N>` 变量。

可以传递多个 loop_var  (数量 和 列表 数量一致, 如果不一致，不会为 多余的列表 定义 变量)
```js
foreach(word num IN ZIP_LISTS L1 L2)
    message("word=${word}, num=${num}")
```

---

## 命令定义

2种方法，
- macro
- function

这2个的区别 类似 C的预处理器宏 和 C++函数 的区别

macro() 更像一个 查找和 替换命令，而不是 实际的 子程序调用。  宏不会在 调用栈上 创建一个 单独的条目，这意味着 在 宏中 调用 return() 比 函数中 return() 高一个级别。

function() 为 局部变量创建一个 单独的 作用域。  这可能导致 混淆的结果。

这2种方法都接受 可以在 命令块 内部命名和 引用的参数。 cmake允许你 使用 以下 引用访问 在命名调用中 传递的参数
- `${ARGC}` 参数数量
- `${ARGV}` 所有参数的列表
- `${ARG0}`,`${ARG1}`,`${ARG2}` ... 匿名参数， 在最后一个预期参数之后  。就是 实参 比 形参 多


---

### 宏

定义宏
```js
macro(<name> [<argument>…])
  <commands>
endmacro()
```

调用，  调用不区分大小写
```js
macro(MyMacro myVar)
  set(myVar "new value")
  message("argument: ${myVar}")
endmacro()
set(myVar "first value")
message("myVar is now: ${myVar}")
MyMacro("called value")
message("myVar is now: ${myVar}")
```

```shell
$ cmake -P chapter02/08-definitions/macro.cmake
myVar is now: first value
argument: called value     // 没有收到 set 的影响，因为 传递给宏的参数 不是作为真正的变量处理，而是作为 常量的查找和替换
myVar is now: new value  // 宏中的 set 影响了 外部
```

==建议尽可能多地使用函数，因为这可能会节省你很多头疼的问题。==

---

### 函数

```js
function(<name> [<argument>…])
  <commands>
endfunction()
```

如果实参比形参多，那么 多余的参数 会被 解释为 匿名参数 并存放到 ARG0 ARG1 .. 中

函数中的 set 只影响它自己， 除非 PARENT_SCOPE

通过 return() 推出调用作用域

cmake为每个函数设置了下列变量
- CMAKE_CURRENT_FUNCTION
- CMAKE_CURRENT_FUNCTION_LIST_DIR
- CMAKE_CURRENT_FUNCTION_LIST_FILE
- CMAKE_CURRENT_FUNCTION_LIST_LINE


```js
function(MyFunction FirstArg)
  message("Function: ${CMAKE_CURRENT_FUNCTION}")
  message("File: ${CMAKE_CURRENT_FUNCTION_LIST_FILE}")
  message("FirstArg: ${FirstArg}")
  set(FirstArg "new value")
  message("FirstArg again: ${FirstArg}")
  message("ARGV0: ${ARGV0} ARGV1: ${ARGV1} ARGC: ${ARGC}")
endfunction()
set(FirstArg "first value")
MyFunction("Value1" "Value2")
message("FirstArg in global scope: ${FirstArg}")
```

输出
```js
Function: MyFunction
File: /root/examples/chapter02/08-definitions/function.cmake
FirstArg: Value1
FirstArg again: new value
ARGV0: Value1 ARGV1: Value2 ARGC: 2
FirstArg in global scope: first value
```

---

### 有用命令

#### message()

可以通过 MODE参数 提供自己的样式, 在出错的情况下，可以停止代码的执行 

`message(<MODE> "text")`

可用的模式
- FATAL_ERROR， 停止处理 和 生成
- SEND_ERROR， 继续处理，但 不生成
- WARNING， 继续处理
- AUTHOR_WARNING， cmake警告，继续
- DEPRECATION， 如果启用了 CMAKE_ERROR_DEPRECATED 或 CMAKE_WARN_DEPRECATED ，会执行 相应工作
- NOTICE，默认， 在 stderr 上打印消息
- STATUS， 建议用于 用户信息
- VERBOSE，用于 不必要太详细的 更多信息
- DEBUG， debug信息， 用于 项目出问题时
- TRACE， trace信息， 用于 开发期间

`cmake --log-context` 消息将带有 上下文， 该上下文 存储在 CMAKE_MESSAGE_CONTEXT 中。 。。 需要手动改
```js
function(foo)
  list(APPEND CMAKE_MESSAGE_CONTEXT "foo")
  message("foo message")
endfunction()
list(APPEND CMAKE_MESSAGE_CONTEXT "top")
message("Before `foo`")
foo()
message("After `foo`")
```

```shell
$ cmake -P message_context.cmake --log-context
[top] Before `foo`
[top.foo] foo message
[top] After `foo`
```

message() 另一个技巧是 向 `CMAKE_MESSAGE_INDENT` 中添加 缩进。
`list(APPEND CMAKE_MESSAGE_INDENT "  ")`

```text
Before `foo`
  foo message
After `foo`
```

---

#### include()

我们可以将 cmake 分割到 单独的文件中。
通过 include 从父列表中 引用它们

`include(<file|module> [OPTIONAL] [RESULT_VARIABLE <var>])`

如果提供一个 .cmake结尾的 文件名， 那么 将 尝试 打开并执行。  注意，这不会 创建 嵌套的 独立作用域。

如果文件不存在，会 抛出错误， 除非使用了 `OPTIONAL`。

是否成功，可以通过 RESULT_VARIABLE 关键字 来获得， 成功时，它将填充 成功包含的 文件的 完整路径，或 在失败时，不包含。


当以 脚本模式运行时，任何相对路径都将 从 当前工作目录解析。

要在脚本中搜索，需要提供一个 绝对路径
`include("${CMAKE_CURRENT_LIST_DIR}/<filename>.cmake") `

==如果没有提供路径==，但提供了 模块的名称 ( 不带.cmake) ， cmake将尝试 查找 模块 并将其包含。 cmake将在 CMAKE_MODULE_PATH 中搜索 `<模块>.cmake` 文件， 然后在其模块目录中搜索。


#### include_guard()

当我们包含 有副作用的文件时， 我们可能希望 只包含一次。

`include_guard([DIRECTORY|GLOBAL])` 放在 被包含文件的 顶部， 当cmake首次遇到它时， 会处理。 再次遇到，不会处理。

DIRECTORY关键字将在当前目录及其子目录内应用保护，
GLOBAL关键字将对整个构建过程应用保护。


#### file()

文件操作命令 的最有用变体

```js
file(READ <filename> <out-var> [...])
file({WRITE | APPEND} <filename> <content>...)
file(DOWNLOAD <url> [<file>] [...])
```

file()命令将让您以系统无关的方式读取、写入和传输文件，以及与文件系统、文件锁、路径和存档进行交互。


#### execute_process

运行其他进程 并收集它们的输出。

`execute_process(COMMAND <cmd1> [<arguments>]… [OPTIONS])`

使用 OS 的API 来创建 子进程， 因此 &&,||,> 这样的 shell操作符 无效。

可以 多次提供 `COMMAND <cmd> [<args>]` 来 链接命令，将 一个的输出 传递给 另一个。

可以使用 `TIMEOUT <秒>` 来终止进程。

可以 `WORKING_DIRECTORY <目录>` 设置目录

所有任务的 退出代码可以通过 `RESULTS_VARIABLE <变量>` 收集到 列表中。 如果只对 最后结果有兴趣，那么使用 单数 `RESULT_VARIABLE <变量>`

为了收集输出，cmake提供了 2个参数  OUTPUT_VARIABLE, ERROR_VARIABLE












# ch3 设置第一个CMake项目

https://cloud.tencent.com/developer/article/2421607

充分利用 cmake 将改善 开发体验 和 生成代码的质量， 因为我们可以自动化很多 单调的任务，如 在 构建后运行测试，检查代码覆盖率，格式化代码， 使用 linter 等 工具 检查 源码

为了充分发挥cmake，需要知道，如何正确配置 整个项目， 如何划分项目 和 设置源代码树

本章涵盖：
- 基本指令和命令
- 如何划分你的项目
- 思考项目结构
- 作用域环境
- 配置工具链
- 禁用源代码内构建


## 基本指令和命令

### cmake_minimum_required 指定最小cmake版本

会检查 是否有正确的 cmake版本。

隐式地，还会调用另一个命令 `cmake_policy(VERSION)` 这将 告诉 cmake 对于这个项目 应该使用哪些策略。

cmake 发展了20多年， 有很多策略，所以需要 版本 来 确定 应用哪些策略。


### project，定义语言和元数据

从技术上讲，不需要 project。

任何包含 CMakeLists.txt 的目录 都会以 project模式被解析。  cmake 隐式 在文件顶部添加这个目录。

2种形式

```js
project(<PROJECT-NAME> [<language-name>...])

project(<PROJECT-NAME>
        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [DESCRIPTION <project-description-string>]
        [HOMEPAGE_URL <url-string>]
        [LANGUAGES <language-name>...])
```

project-name 必选， 其他 可选

---

使用这个 命令会 隐式设置 以下变量
- PROJECT_NAME
- CMAKE_PROJECT_NAME, 仅在最顶层的 CMakeLists.txt 中
- PROJECT_SOURCE_DIR, `<PROJECT-NAME>_SOURCE_DIR`
- PROJECT_BINARY_DIR, `<PROJECT-NAME>_BINARY_DIR`

---

支持的语言：
- C
- CXX
- CUDA
- OBJC
- OBJCXX
- Fortran
- ISPC
- ASM
- CSharp
- Java

默认支持 c 和 c++。


project 会检测 和测试 可用编译器。

---

指定 VERSION 会使得下列变量可用
- PROJECT_VERSION, `<Project-name>_VERSION`
- CMAKE_PROJECT_VERSION, 仅在 顶级 CMakeLists.txt 中
- PROJECT_VERSION_MAJOR ， MINOR， PATH， TWEAK
- `<PROJECT-NAME>_VERSION_MAJOR`， MINOR， PATH， TWEAK


## 划分你的项目

场景： 一个 小型汽车租赁公司的软件， 有很多源文件，包括：管理客户，车辆，停车位，长期合同，维护记录，员工记录 等。
我们 创建多个目录，将 相关文件移入其中
CMakeLists.txt 看起来：
```js
cmake_minimum_required(VERSION 3.20.0)
project(Rental CXX)
add_executable(Rental
               main.cpp
               cars/car.cpp  
               # more files in other directories 
)
```

依然有 源文件列表。

我们可以将 源文件列表 放到另一个文件中，并使用 include 和 cars_sources 变量 ，就像
```js
cmake_minimum_required(VERSION 3.20.0)
project(Rental CXX)
include(cars/cars.cmake)
add_executable(Rental
               main.cpp
               ${cars_sources}
               # ${more variables}
)
```

```js
set(cars_sources
    cars/car.cpp
#   cars/car_maintenance.cpp
)
```

缺点：
- 嵌套目录中的变量 将污染 顶层作用域 (反之亦然)
- 所有目录共享相同的配置
- 存在共享编译触发器
- 所有路径都是相对于顶层而言的

另一种方法是  add_subdirectory()， 它引入了 变量作用域

### 作用域子目录

常见的做法是 按照文件系统的 自然结构来组织项目， 嵌套目录表示 APP 的离散元素： 业务逻辑，gui，api。 单独的目录包含 测试，外部依赖，脚本，文档。

```js
add_subdirectory(source_dir [binary_dir]
  [EXCLUDE_FROM_ALL])
```

这将为我们的构建添加一个 ==源==目录，  可选的 binary_dir 是 生成文件的目录， ==EXCLUDE_FROM_ALL== 将 禁用 子目录中定义的 目标的默认构建。  。。感觉就是可以禁止 project(xxx)。

这个命令 会在 `source_dir` 路径 中寻找 CMakeLists.txt， 然后在 该目录 作用域中 解析 该文件， 意味着 前面的缺点将不复存在
- 变量更改 被限制在 嵌套作用域内
- 你可以自由地以任何喜欢的方式 配置 嵌套的部分
- 更改嵌套的 CMakeLists.txt 文件不需要构建无关的目标
- 路径仅限于目录本地， 如果需要，可以添加到 父级 include路径中


例子：
工程结构
```text
chapter03/03-add_subdirectory# tree -A
.
├── CMakeLists.txt
├── cars
│   ├── CMakeLists.txt
│   ├── car.cpp
│   └── car.h
└── main.cpp
```

cmake配置
```js
cmake_minimum_required(VERSION 3.20.0)
project(Rental CXX)
add_executable(Rental main.cpp)
add_subdirectory(cars)
target_link_libraries(Rental PRIVATE cars)
```

最后一行 将 cars的 结果 连接到 Rental 可执行文件。


cars的 cmake
```js
add_library(cars OBJECT
    car.cpp
#   car_maintenance.cpp
)
target_include_directories(cars PUBLIC .)
```
add_library 产生一个 全局可见的 目标 cars。 并使用 target_inlclude_directories 将其 添加到 公共include 目录。  这允许 main.cpp 不提供相对路径 即可包含 `car.h`
`#include "car.h"`

由于 OBJECT， 所以 add_library 的 并不是 库，而是 对象文件。


### 嵌套项目

什么时候 子文件夹中的 CMakeLists.txt 应该有自己的 project() 呢？

- 当你在一个 CICD管道中 构建多个 C++项目 ( 也许是在构建框架或一系列库)
- 从历史代码中 移植构建系统， 你需要 逐步分解成 独立的单元。

### 外部项目

技术上可以从一个项目 到达另一个项目， cmake 在一定程度上支持这一点。 甚至还有一个 `load_cache()` 命令， 允许你从 另一个项目的缓存中加载值。
但不推荐，因为会导致 循环依赖 和项目耦合


### 思考项目结构

项目结构应该
- 易于导航和扩展
- 自包含的， 项目的文件 都在项目中，而不是外部
- 抽象层次应该通过可执行文件和二进制文件来表达。

建议遵循下面的模板，因为简单 且 方便扩展
- project/
  - cmake/
    - include/
    - module/
    - script/
  - src/
    - app1/
    - app2/
    - lib1/
    - lib2/
  - doc/
  - extern/
  - test/


cmake 包含了 宏和函数，find_modules 和一次性脚本
src 二进制文件 和库的 源代码
doc 用于构建文档
extern 外部项目的配置
test 自动化测试的代码

CMakeLists.txt 文件应该存在于 以下目录中： 
- 根目录
- src
- doc
- extern
- test

主列表文件(。。应该是指 顶级目录的 CMakeLists.txt ) 不应该声明任何自身的 构建步骤， 而是应该使用 add_subdirectory() 来执行 嵌套目录中的 所有 列表文件。

一些开发者 会将 可执行文件 和 库分开， 创建 2个 目录 src 和lib。  cmake将这2种 同等对待，所以 这里的拆分 并不重要。

src中分为 多个目录， 对于 大型项目非常有用。  
如果只是构建一个 可执行文件 或库，那么直接 将源文件存储到 src中。

你的目标文件树 可能类似：
- app1/
  - include/
    - class_a==.h==
    - class_b==.h==
  - lib3/
  - test/
  - CMakeLists.txt
  - main.cpp
  - class_a==.cpp==
  - class_b==.cpp==


库的样子
- lib3/
  - include/
    - lib3/      // 可选，只有 其他项目会使用这个库时 才需要。
      - public.h
    - private.h
  - test/
  - CMakeLists.txt
  - lib3.cpp


CMakeLists.txt 的功能
- /
  - CMake policies
  - project settings
  - global variables
  - global includes
  - global dependencies
  - src/
    - app1/
      - lib3/    
        - add lib3 static library
      - add app1 executable
    - app2/    
      - add app2 executable
    - lib1/    
      - add lib1 static lib
    - lib2/    
      - add lib2 dynamic lib
  - doc/    
    - add doc target
  - extern/    
    - download external project,   clone git repo
  - test/   
    - config CTest,   add test target

1. 从项目的根开始，即 从源树的一个 CMakeLists 开始，这个文件 将设置 最小cmake版本 和 相应的策略，设置 项目名称，支持的语言，全局变量， 并包含来自 cmake 目录的文件，以便它们的内容 全局可用
2. 进入 src 作用域，通过调用 ==add_subdirectory(src bin)== 来完成， 将编译后的文件 放在 bin中， 而不是 src中
3. cmake 读取 src/CMakeLists.txt 并发现 这个文件的 唯一目的是 添加 4个嵌套子目录：app1,app2,lib1,lib2
4. cmake 进入 app1 的变量作用域， 了解到 嵌套库 lib3， 它有 自己的 CMakeLists， 
5. 进入 lib3 作用域， lib3 库添加了一个 与名称相同的 静态库目标，
6. 返回 app1 作用域， app1 添加了一个 依赖于 lib3 的可执行文件，
7. 返回 src作用域， 继续剩余的 app2,lib1,lib2
8. 返回顶层作用域，并执行 剩余的 3个命令 add_subdirectory(doc), add_subdirectory(extern), add_subdirectory(test)
9. 所有目标都被收集，并检查正确性。  cmake现在拥有生成构建系统的 所有必要信息

上面是按照我们编写的命令的顺序发生的。 (。 就是 add_subdirectory 的顺序) 。 有时，顺序并不重要， 下章 就解决这个问题。

开发者通常不愿意创建目录，特别是在 项目的根目录中， 如果我们提供一个良好的结构，人们倾向于遵循它。
。。所以 一开始的时候 就把 空文件夹 建好。




## 环境作用域

cmake提供了多种查询环境的方法，使用  `CMAKE_` 变量， ENV 变量， 特殊命令。  

对于性能关键的 应用程序， 了解目标平台的 ==所有特性 (如 指令集，cpu核心数) 将很有用。 然后将这些 信息 传递给 编译后的 二进制文件==， 以便它们可以完美地 被调整。

看看 cmake 提供了哪些信息：

### OS

`CMAKE_SYSTEM_NAME`

```js
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  message(STATUS "Doing things the usual way")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
  message(STATUS "Thinking differently")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  message(STATUS "I'm supported here too.")
elseif(CMAKE_SYSTEM_NAME STREQUAL "AIX")
  message(STATUS "I buy mainframes.")
else()
  message(STATUS "This is ${CMAKE_SYSTEM_NAME} speaking.")
endif()
```

`CMAKE_SYSTEM_VERSION` 包含：OS版本

建议 使你的方案 尽可能无平台无关，  使用 cmake的 内置的 跨平台功能， 特别是 操作文件系统时， 应该使用 file() 命令


### ==交叉编译，什么是宿主和目标系统==

在一台机器上编译代码，在另一台机器上运行，被称为 交叉编译。

交叉编译 不在本书范围内，但了解它如何影响 cmake 的某些部分 是重要的

允许 交叉编译的 必要步骤之一是 将 `CMAKE_SYSTEM_NAME`, `CMAKE_SYSTEM_VERSION` 设置为 目标OS， cmake中称为 目标系统。
执行构建的 OS 称为 宿主系统

。。交叉编译，是交叉OS，  可以交叉到 硬件吗？  估计不行，毕竟 app 是和 OS通信的， os才和硬件通信。。。不是，有 x86 给 arm编译， 或者反过来 。 话说，嵌入式 有 OS的概念吗？ 好像没有吧。。。有的 embedded os   。 可以交叉到硬件的， 看 下面的 cmake 的文档 。 `set(CMAKE_SYSTEM_PROCESSOR arm)`

无论如何， 宿主系统的信息 总可以通过 带有 HOST 关键字的 变量获得 ：CMAKE_HOST_SYSTEM, CMAKE_HOST_SYSTEM_NAME, CMAKE_HOST_SYSTEM_PROCESSOR, CMAKE_HOST_SYSTEM_VERSION.

所有 HOST 都是 宿主系统， 不带HOST的都是 目标系统 (通常是 宿主系统，除非我们进行交叉编译)。

cmake的 交叉编译 文档： https://cmake.org/cmake/help/latest/manual/cmake-toolchains.7.html


### 缩写变量

cmake 预定义了 一些变量，提供宿主 和目标系统的信息。
如果使用特定的系统，相应的变量被设置为 非假 ( 1 或true)： ANDROID,APPLE,CYGWIN,UNIX,ISO,WIN32,WINCE,WINDOWS_PHONE,  ,,,   CMAKE_HOST_APPLE, CMAKE_HOST_SOLARIS, CMAKE_HOST_UNIX, CMAKE_HOST_WIN32

WIN32, CMAKE_HOST_WIN32， 对于 32位，64位的 windows和 MSYS 保持为 真。
UNIX 对于 linux，macos，cygwin 为真


### 宿主系统信息

cmake可以提供很多信息，但是为了节省时间，它不查询 环境中的 罕见信息， 例如 处理器是否支持MMX， 总物理内存是多少。

这些信息 需要 显式请求 `cmake_host_system_information(RESULT <VARIABLE> QUERY <KEY>…)`

![c3409dc1a1ac5dd1de980595331b22d9.png](../_resources/c3409dc1a1ac5dd1de980595331b22d9.png)

---

如果需要，甚至可以 查询 处理器 特定信息

![f55c0160bd6f1503d211ecea56e724ae.png](../_resources/f55c0160bd6f1503d211ecea56e724ae.png)

---

#### 32位 or 64位？

CMAKE_SIZEOF_VOID_P变量可获得此信息，对于 64 位该值为8（因为指针是 8 字节宽）和对于 32 位该值为4（4 字节）：

```js
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  message(STATUS "Target is 64 bits")
endif()
```

#### 系统的字节序是什么？

大端：高字节 存 低内存地址

cmake提供了 `CMAKE_<lang>_BYTE_ORDER`, lang可选：C,CXX,OBJC,CUDA,  值时： BIG_ENDIAN, LITTEL_ENDIAN



## 配置工具链

工具链包括： 工作环境，生成器，cmake执行文件本身， 编译器

cmake 可以在 构建前 检查 编译器 是否包含了 所有必要的功能

### 设置c++标准

`CMAKE_CXX_STANDARD` 设置为 98, 11, 14, 17, 20, 23

如果需要，你可以按照每个目标单独覆盖它
`set_property(TARGET <target> PROPERTY CXX_STANDARD <standard>)`

---

上面设置的 c++版本， 如果不是 编译器 支持的。  cmake 不会报错。

我们可以明确 要求 c++版本 必须符合：
`set(CMAKE_CXX_STANDARD_REQUIRED ON)`

设置后， 如果 编译器不支持，那么 会报错， 并停止构建
`Target "Standard" requires the language dialect "CXX23" (with compiler extensions), but CMake does not know the compile flags to use to enable it.`


### 供应商特定扩展

你喜欢 -std=gnu++14  而不是 -std=c++14

`CMAKE_CXX_EXTENSIONS`  默认可以扩展。 除非 `set(CMAKE_CXX_EXTENSIONS OFF)`

建议 OFF，这样 代码和 供应商无关。


### 跨过程优化 IPO

通常，编译器 在 每个 翻译单元 上优化， 这意味着 .cpp 被 预处理，编译，然后优化。

然后这些 文件 被 链接器 用来 构建 二进制文件。 现代编译器 可以在 链接后 进行优化 ( 称为 链接时优化)。

如果你的编译器 支持 跨过程优化，那么使用它。  cmake提供了 CMAKE_INTERPROCEDURAL_OPTIMIZATION
在设置前，我们要确保 它是支持的

。。 IPO： interprocedural optimization 

```js
include(CheckIPOSupported) 
check_ipo_supported(RESULT ipo_supported)
if(ipo_supported)
  set(CMAKE_INTERPROCEDURAL_OPTIMIZATION True)
endif()
```

---

### 检查支持的编译器功能

cmake 在配置阶段 询问编译器 支持的特性，并保存到 `CMAKE_CXX_COMPILE_FEATURES`

我们可以编写一个 具体的检查，询问某个特性是否可用
```js
list(FIND CMAKE_CXX_COMPILE_FEATURES 
  cxx_variable_templates result)
if(result EQUAL -1)
  message(FATAL_ERROR "I really need variable templates.")
endif()
```

为每个特性编写 测试， 非常艰巨。

cmake的作者 建议 只检查某些 高级元特性： cxx_std_98, cxx_std_11,cxx_std_14,cxx_std_17,cxx_std_20,cxx_std_23 。 每个元特性 都表明 编译器支持 特定的 c++ 标准

cmake 所有的 特性： https://cmake.org/cmake/help/latest/prop_gbl/CMAKE_CXX_KNOWN_FEATURES.html


---

### 编译 测试特性的文件

通过创建一个测试文件，填入你想检查的特性 的代码。  。。场景是 编译器bug 导致某些头文件不存在，所以 编译始终失败。

cmake提供了 `try_compile()`, `try_run()` 来让你 验证 

try_run 更好，更完整， 但是 交叉编译时 不可用。


测试用代码
```C++
#include <iostream>
int main()
{
  std::cout << "Quick check if things work." << std::endl;
}
```

CMakeLists.txt
```C++
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
try_run(run_result compile_result
${CMAKE_BINARY_DIR}/test_output 
        ${CMAKE_SOURCE_DIR}/main.cpp
        RUN_OUTPUT_VARIABLE output)
message("run_result: ${run_result}")
message("compile_result: ${compile_result}")
message("output:\n" ${output})
```


```js
try_run(<runResultVar> <compileResultVar>
        <bindir> <srcfile> [CMAKE_FLAGS <flags>...]
        [COMPILE_DEFINITIONS <defs>...]
        [LINK_OPTIONS <options>...]
        [LINK_LIBRARIES <libs>...]
        [COMPILE_OUTPUT_VARIABLE <var>]
        [RUN_OUTPUT_VARIABLE <var>]
        [OUTPUT_VARIABLE <var>]
        [WORKING_DIRECTORY <var>]
        [ARGS <args>...])
```

## 禁用源内构建

新的，非官方的， 支持性不好 ， 3.20 会失败。
如果官方支持，那么 这个最好。
```js
# add this options before PROJECT keyword
set(CMAKE_DISABLE_SOURCE_CHANGES ON)
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
```

老的，完全支持
```js
cmake_minimum_required(VERSION 3.20.0)
project(NoInSource CXX)
if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
  message(FATAL_ERROR "In-source builds are not allowed")
endif()
message("Build successful!")
```




# ch4 与目标工作

软件开发中，我们故意划定界线，将 组件分为 一个或 多个 翻译单元(.cpp文件) 。 原因有： 增加代码可读性，管理耦合，加快构建速度， 提取可重用的组件

## 目标概念

如果你用过 gnu make，你已经看到了 目标的 概念。 本质上，它 是一个构建系统， 是 将一组文件 编译成 另一组文件的 食谱。

cmake 允许你节省时间 并跳过 食谱的 中间步骤， 它在更高级别上抽象， 它理解如何直接从 源文件构建可执行文件。  ==所以==你不需要编写任何 显式的 食谱 来编译。 所需的==只是== 一个 add_executable() 命令， 带有 可执行==目标的名字== 和 要作为其元素的 ==文件列表==
`add_executable(app1 a.cpp b.cpp c.cpp)`

cmake 有3个命令 来创建目标
- add_executable() ， 构建可执行文件
- add_library()， 构建库
- add_custom_target()， 指定自己的命令，可以：计算其他二进制文件的校验和，运行代码cleaner，将编译报告发送到 数据处理管道

---

```js
add_custom_target(Name [ALL] [command1 [args1...]]
                  [COMMAND command2 [args2...] ...]
                  [DEPENDS depend depend depend ... ]
                  [BYPRODUCTS [files...]]
                  [WORKING_DIRECTORY dir]
                  [COMMENT comment]
                  [JOB_POOL job_pool]
                  [VERBATIM] [USES_TERMINAL]
                  [COMMAND_EXPAND_LISTS]
                  [SOURCES src1 [src2...]])
```

---

custom target 的一个好的用例是 构建时 删除特定文件。
```js
add_custom_target(clean_stale_coverage_files 
          COMMAND find . -name "*.gcda" -type f -delete)
```

---


### 依赖图

成熟的 应用 由许多组件 ( 指内部库，不是外部依赖) 组成， 它们之间存在 依赖关系。


应用的目标的依赖关系：
checksum(custom) -> guiapp(executable) -> drawing(library) + calculations(library)
               |--> terminalapp(executable) -> calculations(library)

```js
cmake_minimum_required(VERSION 3.19.2)
project(BankApp CXX)
add_executable(terminal_app terminal_app.cpp)
add_executable(gui_app gui_app.cpp)
target_link_libraries(terminal_app calculations)
target_link_libraries(gui_app calculations drawing)
add_library(calculations calculations.cpp)
add_library(drawing drawing.cpp)
add_custom_target(checksum ALL
    COMMAND sh -c "cksum terminal_app>terminal.ck"
    COMMAND sh -c "cksum gui_app>gui.ck"
    BYPRODUCTS terminal.ck gui.ck
    COMMENT "Checking the sums..."
)
```

上面使用 target_link_libraries() 来将 库和 可执行文件 链接起来。

注意到：在 target_link_libraries 之后 我们才 add_library。

cmake 启动后，会 先收集 target 和 它们的信息 (名称，源文件，依赖关系 等)
解析完 所有文件后，尝试构建一个 依赖关系图。 是一个 有向无环图

有一个小问题，前面的 无法保证 checksum 在 可执行文件 之后 构建。cmake 不知道 checksum 依赖于 可执行文件。
为了解决这个问题，我们可以吧 ==add_dependencies() 放在文件末尾。==
`add_dependencies(checksum terminal_app gui_app)`
。。这个依赖 是 target 之间的依赖，不是 外部的依赖。。


### 可视化依赖关系

cmake的一个模块，以 `dot/graphviz` 格式 生成依赖图，且支持 内部依赖 和 外部依赖
`cmake --graphviz=test.dot .`
将生成一个文本文件，可以将其导入到 Graphviz中。
也有在线的 https://dreampuf.github.io/GraphvizOnline/

重要说明： 自定义目标 默认不可见。需要创建一个 特殊的配置文件 CMakeGraphVizOptions.cmake，一个方便的命令是 `set(GRAPHVIZ_CUSTOM_TARGETS TRUE)`, 将其添加到 特殊配置文件。
。。但没有说 这个文件 放哪里。。 是每个文件夹都需要？


### 目标属性

cmake 为 target 定义了大量属性， 这些属性取决于 目标类型 ( 可执行文件，库，自定义)。

也可以添加自己的属性。

```js
get_target_property(<var> <target> <property-name>)
set_target_properties(<target1> <target2> ...
                      PROPERTIES <prop1-name> <value1>
                      <prop2-name> <value2> ...)
```

属性的概念 不限于 target，还支持其他的: GLOBAL,DIRECTORY,SOURCE,INSTALL,TEST,CACHE

get_properties,set_properties 也可以操作属性
`set_property(TARGET <target> PROPERTY <name> <value>)`


通常，使用高级命令更好。


### 传递使用要求是什么

每个target 都有特定的要求：链接一些库，包含一个目录 等。 这些要求 被称为 属性或依赖项

cmake 将 target的某些属性/要求 会附加到 使用 这些target 的 target的属性上。

`target_compile_definitions(<source> <INTERFACE|PUBLIC|PRIVATE> [items1...])`

这个目录将 将 填充 source 的 COMPILE_DEFINITIONS 属性。功能 是 传递给 编译器的 -Dname=definition 标志。 用于 配置 c++ 预处理器定义、  我们需要 指定3个值之一 (interface,public,private) ，来控制 属性应该传递给 哪些target.

- PRIVATE, 用于设置 源目标属性
- INTERFACE， 设置 目标目标属性
- PUBLIC，设置 源目标 和 目标目标属性

源目标 在实现(.cpp)中不使用属性，在头文件中使用，并且要被传递给 消费者目标，那么使用 interface。

内部实现：如果是 private，public，在目标属性中存储提供的值。  如果是 interface，public，它将 带有 `INTERFACE_` 前缀的属性 中存储值

3.20 中，有12个属性 通过 适当的命令( 如 target_link_options ) 或直接通过 set_target_properties() 进行管理
- AUTOUIC_OPTIONS
- COMPILE_DEFINITIONS
- COMPILE_FEATURES
- COMPILE_OPTIONS
- INCLUDE_DIRECTORIES
- LINK_DEPENDS
- LINK_DIRECTORIES
- LINK_LIBRARIES
- LINK_OPTIONS
- POSITION_INDEPENDENT_CODE
- PRECOMPILE_HEADERS
- SOURCES

文档： `https://cmake.org/cmake/help/latest/prop_tgt/<PROPERTY>.html`


---

```js
target_link_libraries(<target>
                     <PRIVATE|PUBLIC|INTERFACE> <item>...
                    [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)
```

- PRIVATE，将源值附加到目的地 私有属性
- INTERFACE，将原值附加到 目的地 接口属性
- PUBLIC，附加到 目的地的 2个属性


#### 处理冲突的传播属性

当一个目标依赖于 多个其他目标时，可能会出现 传播属性 冲突。

比如，一个 目标将 POSITION_IDEPENDENT_CODE 设置为 true， 另一个设置为 false。 cmake 会认为是冲突，并打印
`CMake Error: The INTERFACE_POSITION_INDEPENDENT_CODE property of "source_target2" does not agree with the value of POSITION_INDEPENDENT_CODE already determined for "destination_target".`


cmake 默认不会传播 自定义属性。 我们必须明确地将 自定义属性 添加到 兼容属性的列表中。
每个目标都有4个这样的列表
- COMPATIBLE_INTERFACE_BOOL
- COMPATIBLE_INTERFACE_STRING
- COMPATIBLE_INTERFACE_NUMBER_MAX
- COMPATIBLE_INTERFACE_NUMBER_MIN

将你的属性 添加到其中一个，会触发 传播和兼容性检查。

```js
cmake_minimum_required(VERSION 3.20.0)
project(PropagatedProperties CXX)
add_library(source1 empty.cpp)
set_property(TARGET source1 PROPERTY INTERFACE_LIB_VERSION
   4)
set_property(TARGET source1 APPEND PROPERTY
 COMPATIBLE_INTERFACE_STRING LIB_VERSION
)
add_library(source2 empty.cpp)
set_property(TARGET source2 PROPERTY INTERFACE_LIB_VERSION
   4)
add_library(destination empty.cpp)
target_link_libraries(destination source1 source2)
```


### 导入的目标


IMPORTED， find_package()


### 别名目标

```js
add_executable(<name> ALIAS <target>)
add_library(<name> ALIAS <target>)
```

别名目标的属性是 只读的，不能安装 或 导出


### 接口库

add_library(INTERFACE)

```js
add_library(Eigen INTERFACE 
  src/eigen.h src/vector.h src/matrix.h
)
target_include_directories(Eigen INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
  $<INSTALL_INTERFACE:include/Eigen>
)
```

上面的代码，创建了一个 包含 3个头文件的 Eigen 接口库。
然后 设置 include目录为， 导出时 为 `${CMAKE_CURRENT_SOURCE_DIR}/src`. 安装时为 `include/Eigen`

要使用这样的库，只需要 链接它
`target_link_libraries(executable Eigen)`
这里实际上 并不发生 链接，但 cmake 会理解这个命令 为一个请求， 将所有的 INTERFACE 属性 创博到 executable 目标。

---

第二个用例，使用了相同的机制，但是 目的不同，它创建了一个 逻辑目标，可以作为 传播属性的占位符。 我们随后可以使用 这个目标作为 其他目标依赖

```js
add_library(warning_props INTERFACE)
target_compile_options(warning_props INTERFACE 
  -Wall -Wextra -Wpedantic
) 
target_link_libraries(executable warning_props)
```
add_library(INTERFACE)命令创建了一个逻辑warning_props目标，用于在第二个命令中设置编译选项在executable目标上


---

### 指定构建的目标

`cmake --build <build tree>`, 目标 默认是 ALL。
`--target <name>` 选择目标

另一个隐式定义的 构建目标 是 clean。


## 编写自定义命令

使用自定义目标有一个缺点： 一旦你把它们添加到 ALL 目标中，或者 让它们为其他目标提供依赖，它们将会在每次构建时 都被构建 ( 当然，依然可以通过 if块中启用 来限制这种情况)。

自定义目标有 2个签名， 一个是 add_custom_target 的扩展版
```js
add_custom_command(OUTPUT output1 [output2 ...]
                   COMMAND command1 [ARGS] [args1...]
                   [COMMAND command2 [ARGS] [args2...] ...]
                   [MAIN_DEPENDENCY depend]
                   [DEPENDS [depends...]]
                   [BYPRODUCTS [files...]]
                   [IMPLICIT_DEPENDS <lang1> depend1
                                    [<lang2> depend2] ...]
                   [WORKING_DIRECTORY dir]
                   [COMMENT comment]
                   [DEPFILE depfile]
                   [JOB_POOL job_pool]
                   [VERBATIM] [APPEND] [USES_TERMINAL]
                   [COMMAND_EXPAND_LISTS])
```

自定义命令 不会创建一个逻辑目标，但与 自定义目标一样，它也必须 添加到 依赖图中。  
有2种方式： 使用其输出 作为可执行文件(或库)的源， 或者明确将其 添加到 自定义目标的 一个 depends 列表中。

### 将自定义命令作为生成器使用

不是每个项目都需要从其他文件生成 c++代码。 比如 .proto 文件的编译。

在 .proto 中定义模型：
```js
message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;
}
```

自定义命令
```js
add_custom_command(OUTPUT person.pb.h person.pb.cc
        COMMAND protoc ARGS person.proto
        DEPENDS person.proto
)
```

为了允许 可执行文件进行序列化， 只需将 输出文件 添加到 源文件中。
`add_executable(serializer serializer.cpp person.pb.cc)`

假设我们正确处理了头文件的包含和 protobuf 库的链接，当我们对.proto文件进行更改时，一切都会自动编译和更新


### 使用自定义命令作为目标钩子 第二版签名

add_custom_command() 第二版 引入了 在 构建目标之前 或之后 执行命令的 机制

```js
add_custom_command(TARGET <target>
                   PRE_BUILD | PRE_LINK | POST_BUILD
                   COMMAND command1 [ARGS] [args1...]
                   [COMMAND command2 [ARGS] [args2...] ...]
                   [BYPRODUCTS [files...]]
                   [WORKING_DIRECTORY dir]
                   [COMMENT comment]
                   [VERBATIM] [USES_TERMINAL]
                   [COMMAND_EXPAND_LISTS])
```

使用这个 版本， 可以 复制之前的 checksum ：

```js
cmake_minimum_required(VERSION 3.20.0)
project(Command CXX)
add_executable(main main.cpp)
add_custom_command(TARGET main POST_BUILD
                   COMMAND cksum 
                   ARGS "$<TARGET_FILE:main>" > "main.ck")
```

cmake 将执行 cksum命令， 并提供参数。 
第一个参数 不是变量， 是一个 生成表达式，推导 目标二进制文件 的完整路径。


## 理解生成器表达式

cmake 分为3个阶段： config，generate，build。 

通常在 config阶段 就获得了 所有必须的数据。 
但是，偶尔，会遇到 鸡和蛋的问题。 

我们可以为这些信息 创建一个 占位符， 将它的 推导 延迟至 下一阶段，即生成阶段

这就是 生成器表达式 所做的。

提示：生成器表达式 在 生成阶段 推导， 这意味着 你很难 将它们的输出 捕获到 一个 变量中 并打印到 控制台。 要调试它们，你可以：
- 将其写入文件  `file(GENERATE OUTPUT filename CONTENT "$<...>")`
- or 从命令行显式 添加一个 自定义目标 并构建它  `add_custom_target(gendbg COMMAND CMAKECOMMAND−Eecho`  。。这条命令是不对的。( 像是ocr的)


### 一般语法

最简单的例子
```js
target_compile_definitions(foo PUBLIC
  BAR=$<TARGET_FILE:foo>)
```

前面的命令向编译器参数添加了一个`-D`定义标志（现在忽略PUBLIC）来设置BAR预处理器定义为foo 目标的可执行文件路径。

生成器表达式的 结构
![bb492130a8a3c61d0ebd248ef1d24d96.png](../_resources/bb492130a8a3c61d0ebd248ef1d24d96.png)

- 以 `$<` 开头
- expression 名字
- 如果需要参数，那么添加 冒号，并提供 arg1,arg2 等，用 逗号分隔。
- `>` 结尾

当使用 高级功能时， 生成器表达式 可能会让人 困惑。

### 嵌套

不难理解。
`$<UPPER_CASE:$<PLATFORM_ID>>`

`$<UPPER_CASE:${my_variable}>`

### 条件表达式

`$<IF:condition,true_string,false_string>`

可以在 条件未满足时跳过值
`$<IF:condition,true_string,>`
`$<condition:true_string>`  等价于上面

---

```js
$<$<AND:$<COMPILE_LANGUAGE:CXX>,$<CXX_COMPILER_ID:AppleClan
  g,Clang>>:COMPILING_CXX_WITH_CLANG>
```

### 评估类型

生成表达式 被推导为2种类型之一： string 或 bool。 布尔用 1，0 表示。


逻辑运算符
- `$<NOT:arg>`
- `$<AND:arg1,arg2,arg3...>`  都1，则1
- `$<OR:arg1,arg2,arg3...>` 有1，就1
- `$<BOOL:string_arg>`  将string 转为 bool

string 转为bool时， 下列情况会转为 false
- 空字符串
- 0, false, off, n, no, ignore, notfound， 不区分大小写
- 以 `-NOTFOUND` 结尾 ， 区分大小写


---

字符串比较

满足则为1， 否则0
- `$<STREQUAL:arg1,arg2>`  区分大小写的字符串比较
- `$<EQUAL:arg1,arg2>`  转为数字 再比较
- `$<IN_LIST:arg,list>` 检查 arg 是否在 list 中。
- `$<VERSIONEQUAL:v1,v2>, $<VERSIONGREATER:v1,v2>`


---

变量查询

满足1，否则0

简单查询
- `$<TARGET_EXISTS:arg>`, arg目标存在吗？

扫描传递的参数 以查找特定值
- `$<CONFIG:args>`   当前配置(Debug,Release 等)
- `$<PLATFORM_ID:args>`  平台id
- `$<LANG_COMPILER_ID:args>`  cmake中 lang的id，  lang是： C，CXX，OBJC，CUDA 等
- `$<LANG_COMPILER_VERSION:args>`， cmake中 lang 的 编译器版本
- `$<COMPILE_FEATURES:features>`， 如果 features 被 编译器支持，返回 true
- `$<COMPILE_LANG_AND_ID:lang,compiler_id1,compiler_id2...>`， lang 和 编译器id 是否存在于 compiler_ids 列表中

---

如果我们用AppleClang或Clang编译CXX编译器，将设置-DCXX_CLANG定义。
对于来自 Intel 的CXX编译器，将设置-DCXX_INTEL定义标志。
最后，对于C和Clang编译器，我们将得到一个-DC_CLANG定义。

```js
target_compile_definitions(myapp PRIVATE 
 $<$<COMPILE_LANG_AND_ID:CXX,AppleClang,Clang>:CXX_CLAN
  G>
 $<$<COMPILE_LANG_AND_ID:CXX,Intel>:CXX_INTEL>
 $<$<COMPILE_LANG_AND_ID:C,Clang>:C_CLANG>
)
```

---

`$<LINK_LANG_AND_ID:lang,compiler_id1,compiler_id2...>` 与COMPILE_LANG_AND_ID类似，但检查链接步骤使用的语言。
使用此表达式指定特定语言和链接器组合的目标的链接库、链接选项、链接目录和链接依赖项。
     
`$<LINK_LANGUAGE:args>` 是args中链接步骤使用的语言。


---

评估为字符串


变量查询
这些表达式在生成阶段将评估为一个特定的值
- `$<CONFIG>`   配置（Debug和Release）名称。
- `$<PLATFORM_ID>`  当前系统的 CMake 平台 ID（Linux、Windows或Darwin）。我们在上一章的环境范围部分讨论了平台。
- `$<LANG_COMPILER_ID>`  这是用于LANG编译器的 CMake 编译器 ID，其中LANG是C、CXX、CUDA、OBJC、OBJCXX、Fortran或ISPC中的一个。
- `$<LANG_COMPILER_VERSION>`  这是用于LANG编译器的 CMake 编译器版本，其中LANG是C、CXX、CUDA、OBJC、OBJCXX、Fortran或ISPC中的一个。
- `$<LINK_LANGUAGE>`  在评估链接选项时，目标的语言。



目标依赖查询

使用以下查询，您可以评估可执行文件或库目标属性
自 CMake 3.19 以来，对于在另一个目标上下文中查询大多数目标表达式，不再创建这些目标之间的自动依赖关系

- `$<TARGET_NAME_IF_EXISTS:target>` – 如果存在，则是target的目标名称；否则为空字符串。
- `$<TARGET_FILE:target>` – target二进制文件的完整路径。
- `$<TARGET_FILE_NAME:target>` – target文件名。
- `$<TARGET_LINKER_FILE:target>` – 链接到target目标时使用的文件。通常，它是target表示的库（在具有.lib导入库的平台上的.a、.lib、.so）。

---

TARGET_LINKER_FILE 提供了与常规TARGET_FILE表达式相同的系列表达式
- `$<TARGET_LINKER_FILE_NAME:target>, <TARGET_LINKER_FILE_PREFIX:target>`
- `$<TARGET_SONAME_FILE:target>` – 具有 so 名字的文件的完整路径（.so.3）。
- `$<TARGET_SONAME_FILE_NAME:target>` – 具有 so名字 的文件名称。
- `$<TARGET_SONAME_FILE_DIR:target>` – 具有 so名字 的文件的目录。
- `$<TARGET_PDB_FILE:target>` – 链接器生成的程序数据库文件（.pdb）的完整路径，用于target

---

。。很多都直接复制了。 不管了。

- `<TARGETPDBFILEBASENAME:target>`
- `$<TARGET_BUNDLE_DIR:target>` – 目标（target）的全路径到捆绑包（Apple 特定的包）目录（my.app、my.framework或my.bundle）。
- `$<TARGET_BUNDLE_CONTENT_DIR:target>` – target的全路径的捆绑内容目录。在 macOS 上，它是my.app/Contents，my.framework或my.bundle/Contents。其他的my.app，my.framework或my.bundle。
- `$<TARGET_PROPERTY:target,prop>` – target的prop值。
- `$<TARGET_PROPERTY:prop>` – 正在评估的表达式的target的prop值。
- `$<INSTALL_PREFIX>` – 当目标用install(EXPORT)导出时或在INSTALL_NAME_DIR中评估时，安装前缀为；否则，为空。
     

---

转义

很少情况下，需要向 具有特殊意义的生成器表达式传递一个字符

- `$<ANGLE-R>` – 字面上的>符号（比较包含>的字符串）
- `$<COMMA>` – 字面上的,符号（比较包含,的字符串）
- `$<SEMICOLON>` – 字面上的；符号（防止在带有；的参数上进行列表展开）
     

---

字符串转换

在生成器阶段 处理字符串是可能的，使用：
- `$<JOIN:list,d>` – 使用d分隔符将分号分隔的list连接起来。
- `$<REMOVE_DUPLICATES:list>` – 不排序地删除list中的重复项。
- `$<FILTER:list,INCLUDE|EXCLUDE,regex>` – 使用regex正则表达式从列表中包含/排除项。
- `$<LOWERCASE:string>，$<GENEX_EVAL:expr>` – 以当前目标的嵌套表达式的上下文评估expr字符串。当嵌套表达式的评估返回另一个表达式时（它们不是递归评估的），这很有用。
- `$<TARGET_GENEX_EVAL:target,expr>` – 以与GENEX_EVAL转换类似的方式评估expr，但在target的上下文中。
     

---

输出相关表达式

以下表达式如果满足特定条件将返回其第一个参数，否则返回空字符串：
- `$<LINK_ONLY:deps>` – 在target_link_libraries()中隐式设置以存储PRIVATE deps链接依赖项，这些依赖项不会作为使用要求传播。
- `$<INSTALL_INTERFACE:content>` – 如果用于install(EXPORT)，则返回content。
- `$<BUILD_INTERFACE:content>` – 如果与export()命令一起使用或在与同一构建系统中的另一个目标一起使用时，返回content。
     

以下输出表达式将对其参数执行字符串转换：
- `$<MAKE_C_IDENTIFIER:input>` – 转换为遵循与string(MAKE_C_IDENTIFIER)相同行为的 C 标识符。
- `$<SHELL_PATH:input>` – 将绝对路径（或路径列表）转换为与目标操作系统匹配的壳路径样式。在 Windows 壳中，反斜杠被转换为斜杠，在 MSYS 壳中，驱动器字母被转换为 POSIX 路径。
     


游离变量查询表达式：
- `$<TARGET_OBJECTS:target>` – 从target对象库返回一个对象文件列表

---

### 例子

第一章中，讨论了 指定我们要构建的配置的 构建类型 : Debug，Release 等


#### 基于构建类型，希望采取不同的行为

最简单的方法是 `$<CONFIG>`

```js
target_compile_options(tgt $<$<CONFIG:DEBUG>:-ginline-
  points>)
```
上面检查 config 是否是 DEBUG， 如果是，嵌套表达式 被评估为1， 外面的 简写 if 就是 true， 那么 `-ginline-points` 这个 调试选项 就加入到 选项中。

---

#### 特定于系统的单行命令

生成器表达式 还可以将 冗长的 if 压缩成一行
```js
if (${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
     target_compile_definitions(myProject PRIVATE LINUX=1)
endif()
```

```js
target_compile_definitions(myProject PRIVATE
  $<$<CMAKE_SYSTEM_NAME:LINUX>:LINUX=1>)
```

---


#### 与编译器特定标志相关的接口库

接口库可以用来提供与编译器匹配的标志：

```js
add_library(enable_rtti INTERFACE)
target_compile_options(enable_rtti INTERFACE
  $<$<OR:$<COMPILER_ID:GNU>,$<COMPILER_ID:Clang>>:-rtti>
)
```
即使在这个简单的例子中，我们也可以看到当我们嵌套太多生成器表达式时会发生什么。
不幸的是，有时这是实现所需效果的唯一方法。这里发生了什么：
1. 我们检查COMPILER_ID是否为GNU；如果是这样，我们将OR评估为1。
2. 如果不是，我们检查COMPILER_ID是否为Clang，并将OR评估为1。否则，将OR评估为0。
3. 如果OR被评估为1，则将`-rtti`添加到enable_rtti编译选项。否则，什么都不做。
     
接下来，我们可以用 `enable_rtti` 接口库链接我们的库和可执行文件。
如果编译器支持，CMake 将添加`-rtti`标志。


---

#### 嵌套生成器表达式

有时，在尝试在生成器表达式中嵌套元素时，不清楚会发生什么。我们可以通过生成测试输出到调试文件来调试这些表达式

```js
set(myvar "small text")
set(myvar2 "small > text")
file(GENERATE OUTPUT nesting CONTENT
  "1 $<PLATFORM_ID>
  2 $<UPPER_CASE:$<PLATFORM_ID>>
  3 $<UPPER_CASE:hello world>
  4 $<UPPER_CASE:${myvar}>
  5 $<UPPER_CASE:${myvar2}>
")
```

输出如下

```text
# cat nesting
1 Linux
  2 LINUX
  3 HELLO WORLD
  4 SMALL TEXT
  5 SMALL  text>
```

---

#### 条件表达式与 BOOL 运算符评估之间的区别

生成器表达式在评估布尔类型到字符串时可能会有些令人困惑。
理解它们与普通的条件表达式有何不同是很重要的，尤其是从一个显式的IF关键字开始：

```js
file(GENERATE OUTPUT boolean CONTENT
  "1 $<0:TRUE>
  2 $<0:TRUE,FALSE> (won't work)
  3 $<1:TRUE,FALSE>
  4 $<IF:0,TRUE,FALSE>
  5 $<IF:0,TRUE,>
")
```
这将产生一个文件，像这样：
```js
# cat boolean
1
  2  (won't work)
  3 TRUE,FALSE
  4 FALSE
  5
```

让我们检查每行的输出：
- 这是一个布尔展开，其中BOOL是0；因此，没有写入TRUE字符串。
- 这是一个典型的错误——作者本意是想根据BOOL值的TRUE或FALSE打印，但由于它也是一个布尔的false展开，两个参数被视为一个，因此没有打印。
- 这是一个反转值的相同错误——它是一个布尔true展开，在单行中写入两个参数。
- 这是一个从IF开始的正确条件表达式——它打印FALSE，因为第一个参数是0。
- 这是条件表达式的错误用法——当我们需要为布尔false不写值时，我们应该使用第一种形式。
     

生成器表达式以其复杂的语法而闻名。
本例中提到的区别即使是经验丰富的构建者也会感到困惑。
如果有疑问，将这样的表达式复制到另一个文件中，通过增加缩进和空格来拆分它，以便更好地理解。






# ch5 使用CMake编译C++源码

编译是如何工作的，它的内部阶段是什么，以及它们如何影响二进制输出。

可以使用哪些命令来调整编译，如何从编译器那里要求特定的功能，以及如何向编译器提供必须处理的输入文件。

我们将提供包含头文件的路径，并研究如何插入 CMake 和环境预处理器定义。

优化器以及不同标志如何影响性能

优化的代价——调试被破坏的代码有多困难。

如何通过使用预编译头和单元编译来减少编译时间，为发现错误做准备，调试构建，以及在最终二进制文件中存储调试信息。


## ==编译的基础==

将用高级编程语言编写的指令翻译成低级机器代码的过程

创建并运行一个 cpp程序需要几个步骤
1. 设计你的app并自己编码
2. 将单个 .cpp 文件 ( 翻译单元) 编译成目标
3. 将 目标文件 链接成 单个可执行文件，并添加所有其他 依赖项: 动态，静态库
4. 要运行程序，OS将使用 一个 称为 加载器 的工具 将 app的机器代码 和 需要的 动态库 映射到 虚拟内存。 加载器 然后读取 文件头 以检查 程序从哪里开始，并将 控制权 交给代码
5. 启动C++运行时，执行特殊的 _start 函数 来收集 命令行参数 和 环境变量。 启动线程，初始化静态符号，注册 清理回调函数。 然后 由它调用 程序员编写的 main() 函数


### 编译器如何工作

编译时 将高级语言翻译为低级语言 的过程，具体来说，是通过 产生特定处理器可以直接执行的机器代码，以二进制==对象文件==格式。 该格式特定于平台，linux是ELF(可执行和可链接格式)，win是PE/COFF，mac是 Mach。

对象文件 是单个源文件的直接翻译， 每个对象文件需要 单独编译，之后 链接器 将它们合并成一个可执行文件 或库。  因此，当你修改了代码，只需重新编译 受影响的文件，就可以节约时间。

编译器 创建对象文件的步骤
- 预处理
  `#include, #define -D, #if ..` 类似查找和替换工具。 很重要，因为 将代码分成部分 并在 多个翻译单元之间 共享声明 是代码可重用的基础
- 语言分析
  逐字符扫描文件，进行==词法分析==，将字符 分组成 有意义的==标记==: 关键字，操作符，变量名等。
  标记被分组为标记链，检查它们的顺序和存在 是否遵循C++的规则， 这个过程是 ==语法分析== ( 通常，编译器错误大多是这个阶段的)
  最后，进行 ==语义分析==，编译器尝试检测文件中的语句是否真的有意义，例如 它们必须满足类型正确性检查( 不能将 int赋给 string变量)
- 汇编
  将标记翻译为 基于平台可用指令集的 CPU特定指令。
  一些编译器实际上是 创建一个 汇编输出文件，然后传递给 专门的 汇编器，来产生CPU可执行的机器代码。
- 优化
  在整个编译过程中，在每个阶段逐步进行。
  在生成第一个汇编版本后 有一个明确的阶段，负责最小化寄存器的使用 和删除未使用的代码。
  一个有趣且重要的优化是 内联，编译器 复制 函数主体 替换 其调用。 这个过程加快了 执行速度 并减少了 内存使用， 但是 对调试有重大缺点 (执行的代码不再在原始行上)。
- 代码生成
  根据目标平台指定的格式 将优化后的 机器码写入 ==对象文件==， 这个对象文件 不能直接执行，它必须传递给下一个 工具: ==链接器==，它将 适当移动 对象文件的各个部分并解决 对外部符号的引用


每个阶段都有重要意义，让我们来看 如何使用cmake管理 各个阶段


### ==初始配置==

cmake提供多个命令 来影响各个阶段

- target_compile_features()：要求具有特定特性的编译器编译此目标。
- target_sources()：向已定义的目标添加源文件。
- target_include_directories()：设置预处理器包含==路径==。
- target_compile_definitions()：设置预处理器定义。
- target_compile_options()：命令行上的编译器特定选项。
- target_precompile_headers()：优化外部头的编译。

所有上述命令都接受
```js
target_...(<target name> <INTERFACE|PUBLIC|PRIVATE>
  <value>)
```
这意味着它们支持属性传播，既可以用于 可执行文件 也可以用于 库。  
所有这些命令都支持 生成器表达式


---

要求编译器具有特定的特性

指定构建目标 所需的 所有特性
```js
target_compile_features(<target> <PRIVATE|PUBLIC|INTERFACE>
                        <feature> [...])
```

https://cmake.org/cmake/help/latest/command/target_compile_features.html

https://cmake.org/cmake/help/latest/prop_gbl/CMAKE_CXX_KNOWN_FEATURES.html#prop_gbl:CMAKE_CXX_KNOWN_FEATURES

。feature 一般就是 大版本， cxx_std_11, 14, 这些。 还有一部分 底层的


---

### 管理目标源代码

我们已经知道 通过 `add_executable()` `add_library()`  来告知 cmake 哪些源文件组成一个目标( 可执行文件或 库)

随着时间推移， 我们可能会得到 一些非常长的  `add_...()` 命令。  
一种可能的方案是 使用 GLOB 模式的 file()， 它 可以收集 子目录中所有的 文件 并存储到一个 变量中。

```js
file(GLOB helloworld_SRC "*.h" "*.cpp")
add_executable(helloworld ${helloworld_SRC})
```

但这个 方案 ==不推荐==
因为: cmake 根据列表文件的变化 生成构建系统，因此如果没有进行任何更改，构建可能在 ==没有警告==的情况下失败。  而且，不在目标声明中列出所有 源代码 可能导致 IDE 的代码审查失败。

如果不建议 在目标声明中使用变量，我们如何才能在 例如 处理 平台特定的 实现文件(如 gui_linux.cpp，gui_windows.cpp) 是 条件性地添加源文件呢?

我们可以使用 `target_sources()` 将文件追加到 先前创建的目标

```js
add_executable(main main.cpp)
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  target_sources(main PRIVATE gui_linux.cpp)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  target_sources(main PRIVATE gui_windows.cpp)
endif()
```

可以平台区分， 但是长文件还是没有办法，只能手动添加。

---


## 预处理器配置

为包含文件提供路径
使用预处理器定义
使用cmake配置包含的头文件

---

为包含文件提供路径

预处理器最基本的功能是 `#include`
有2种形式 `#include <path>`  `#include "path"`

通常，尖括号 将检查 标准include目录; 引号 先在当前目录中搜索，然后检查 尖括号 的目录

cmake提供了一个命令，用于搜索包含文件所需的路径
```js
target_include_directories(<target> [SYSTEM] [AFTER|BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [item1...]
  [<INTERFACE|PUBLIC|PRIVATE> [item2...] ...])
```

可以添加自定义路径。 cmake 会提供给 编译器 ( 通常是 `-I`)

BEFORE,AFTER 确定路径 应该添加到目标 INCLUDE_DIRECTORIES 属性之前还是之后 (。。?)。  
这里的目录 在 默认目录之前还是之后(通常 是之后) 检查 由编译器决定

SYSTEM 告知编译器，提供的目录是作为 标准系统目录 (与 尖括号一起使用)。 一般通过 `-isystem` 作为编译器参数

---

预处理器定义

`#define, #if, #elif, #endif`

现有如下代码
```C++
#include <iostream>
int main() {
#if defined(ABC)
    std::cout << "ABC is defined!" << std::endl;
#endif
#if (DEF < 2*4-3)
    std::cout << "DEF is greater than 5!" << std::endl;
#endif
}
```

由于没有 定义 ABC，DEF，所以上面的 方法体 是 空的。

在cpp文件顶部增加
```C++
#define ABC
#define DEF 8
```
后可以看到 2行输出。

如果我们需要根据外部因素(如os，cpu架构 等) 来配置， 那么需要 `target_compile_definitions()`
```js
set(VAR 8)
add_executable(defined definitions.cpp)
target_compile_definitions(defined PRIVATE ABC
  "DEF=${VAR}")
```
上面的配置 和 上上面的 一致。

这些定义 通过 `-D` 传给 编译器。

一些程序员 依然手动添加 -D， cmake会识别出来，并删除 -D ， `target_compile_definitions(hello PRIVATE -DFOO)`
还会忽略空字符串: `target_compile_definitions(hello PRIVATE -D FOO)`， `-D` 是一个独立参数，移除后变成 空字符串， 然后被忽略。

---

单元测试 私有类字段时的常见陷阱

一些人 会建议在 单元测试中 使用 -D 与 #define 结合 。  最简单的 应用是 在 单元测试时 忽略访问修饰符

```C++
class X {
#ifndef UNIT_TEST
 private: 
#endif
  int x_;
}
```

虽然很方便， 但非常不整洁。   
单元测试应该只测试 公共接口，将底层视为黑盒。
建议只有万不得已的时候才使用。

---

使用git 提交跟踪编译版本

。。看了下，应该是 在程序中 打印 git 的 commitid

```js
add_executable(print_commit print_commit.cpp)
execute_process(COMMAND git log -1 --pretty=format:%h
                OUTPUT_VARIABLE SHA)
target_compile_definitions(print_commit PRIVATE
  "SHA=${SHA}")
```

```C++
#include <iostream>
// special macros to convert definitions into c-strings:
#define str(s) #s
#define xstr(s) str(s)
int main()
{
#if defined(SHA)     // ? 不是 #ifdef ?  https://learn.microsoft.com/zh-cn/cpp/preprocessor/hash-ifdef-and-hash-ifndef-directives-c-cpp?view=msvc-170   。。 有相同效果。  但是 () 需要吗?
    std::cout << "GIT commit: " << xstr(SHA) << std::endl;
#endif
}
```

上述代码需要用户在他们的 PATH 中安装并可访问 git


---

### ==替代==target_compile_definitions,配置头文件

如果有多个变量，那么通过 上面的 target_compile_definitions 传递 有些繁琐

我们可以提供一个 带有引用各种变量的占位符的 头文件， 然后 让 cmake 填充。

`configure_file(<input> <output>)`

新文件:configure.h.in
```text
#cmakedefine FOO_ENABLE
#cmakedefine FOO_STRING1 "@FOO_STRING@"
#cmakedefine FOO_STRING2 "${FOO_STRING}"
#cmakedefine FOO_UNDEFINED "@FOO_UNDEFINED@"
```

```js
add_executable(configure configure.cpp)
set(FOO_ENABLE ON)
set(FOO_STRING1 "abc")
set(FOO_STRING2 "def")
configure_file(configure.h.in configured/configure.h)
target_include_directories(configure PRIVATE 
                           ${CMAKE_CURRENT_BINARY_DIR})
```

构建时，cmake会生成一个文件 configure.h :
```C++
#define FOO_ENABLE
#define FOO_STRING1 "abc"
#define FOO_STRING2 "def"
/* #undef FOO_UNDEFINED "@FOO_UNDEFINED@" */
```

如果你需要为#if块提供显式的#define 1或#define 0，请使用#cmakedefine01。

cpp中使用
```C++
#include <iostream>
#include "configured/configure.h"
// special macros to convert definitions into c-strings:
#define str(s) #s
#define xstr(s) str(s)
using namespace std;
int main()
{
#ifdef FOO_ENABLE
  cout << "FOO_ENABLE: ON" << endl;
#endif
  cout << "FOO_ENABLE1: " << xstr(FOO_ENABLE1) << endl;
  cout << "FOO_ENABLE2: " << xstr(FOO_ENABLE2) << endl;
  cout << "FOO_UNDEFINED: " << xstr(FOO_UNDEFINED) << endl;
}
```

# ==QQQ==

。。? 上面的 define 是什么意思?  #s 有什么用?  不是直接 s ?  为什么要 xstr 转 str  转 s ?
。看下面的输出，似乎是 转为字面意思，而不是 值。





输出
```text
FOO_ENABLE: ON
FOO_ENABLE1: FOO_ENABLE1
FOO_ENABLE2: FOO_ENABLE2
FOO_UNDEFINED: FOO_UNDEFINED
```


configure_file()命令还具有许多格式化和文件权限选项，这里不介绍








## 配置优化器

。。 -O1 这种?


优化器会分析 上阶段的 结构，并使用 一些 不整洁的技巧 ( 但可能提高性能)。

优化器 会对 源码进行大量转换，以至于 无法辨认。

优化器会
- 决定哪些函数可以被删除 或压缩
- 移动代码， 多次重复代码
- 判断代码是否是 死代码，是则移除。
- 重复利用内存，所以 多个变量 在不同时间 使用 同一个内存块。
- 将控制结构 换成完全不同的结构

优化器一般 不启用任何优化。

`target_compile_options()` 精确指定我们需要哪些优化

```js
target_compile_options(<target> [BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```

。。还是不知道 before 是什么， 这个要看 文档了。

target_compile_options 还可以提供 -D 的参数。 建议 不要用。 -D 用专门的 target_compile_definition

---

通用级别

大多编译器提供了 4个基本级别的优化，从 0到3。 
- -O0，没有优化，通常是 编译器的 默认级别
- -O1，启用了 适量优化，不会使 编译速度变得太慢
- -O2，被认为是 完全优化。 生成 高度优化的代码，但编译时间最慢
- -O3，完全优化，类似于 -O2， 但是 子程序内联，循环向量化 方面 更激进

还有一些变体
- -Os，优化 生成文件的大小 (而不是 执行速度)
- -Ofast， 超级激进，不严格符合C++标准。 最明显的区别是 -ffast-math, -ffinite-math， 如果你的程序是 关于精确计算的， 不要使用这个优化。


cmake做了一层封装
- CMAKE_CXX_FLAGS_DEBUG，等同于 -g
- CMAKE_CXX_FLAGS_RELEASE，等同于 -O3 -DNDEBUG


`CMAKE_<LANG>_FLAGS_<CONFIG>` 变量是全局的， 适用于所有目标。
建议使用 target_compile_options 将 配置 应用到 具体的 target ，而 不依赖于 全局变量。


==先通过 -Ox 来选择优化级别，然后通过  -f， -fno 微调==

---

函数内联

内联后，不必经历 方法调用的 帧 的创建和销毁， 不需要 寻找 下一条要执行 的 指令地址。

内联会消耗 内存。

调试时 增加难度: 行号不准， 内联函数内的断点 永不触发。
所以在 调试时，需要关闭内联。 

和内联有关的参数
- -O0
- -finline-functions-called-once：仅 GCC 支持
- -finline-functions：Clang 和 GCC
- -finline-hint-functions：仅 Clang 支持
- -finline-functions-called-once：仅 GCC 支持

使用`-fno-inline-...`显式禁用内联。
无论如何，对于详细信息，请参阅您编译器的特定版本的文档。


---

循环展开

将循环 转换为 相同效果的语句， 加速，因为减少了 或消除了 控制循环的指令。

```C++
void func() {
  for(int i = 0; i < 3; i++)
    cout << "hello\n";
}
```
展开后
```C++
void func() {
    cout << "hello\n";
    cout << "hello\n";
    cout << "hello\n";
} 
```

循环展开 ，只有 在编译器知道 迭代次数时 才可以。 
循环展开，可能对 现代CPU 产生 负面影响，因为 代码大小的增加 会 降低 缓存命中率

有关的参数
- -floop-unroll,  GCC
- -funroll-loops,  Clang

---

循环向量化

单指令多数据(SIMD) 是 为 实现并行化而开发的机制。 它可以 同时对 多块信息执行相同操作。

```C++
int a[128];
int b[128];
// initialize b
for (i = 0; i<128; i++)
  a[i] = b[i] + 5;
```
通常 上面的代码会循环 128次。

编译器可能将上面的代码转换为
```C++
for (i = 0; i<32; i+=4) {
  a[ i ] = b[ i ] + 5;
  a[i+1] = b[i+1] + 5;
  a[i+2] = b[i+2] + 5;
  a[i+3] = b[i+3] + 5;
}
```

。。duff device?。。不。下面

gcc 在 -O3 时 启用 循环向量化。 clang默认开启。

- -ftree-vectorize -ftree-slp-vectorize   在 GCC 中启用
- -fno-vectorize -fno-slp-vectorize   在 Clang 中禁用

==向量化性能的提升来自 利用 CPU制造商 提供的特殊指令， 而不是 简单地将 循环 展开。==


## 管理编译过程

我们需要考虑 编译的 其他方面:
- 完成所需的时间
- 如何 容易地发现 和 修复 在构建中发生的错误


---

### 减少编译时间

- 头文件预编译
- 单元构建

---

头文件预编译

.h 在实际编译前 由 预处理器 包含到 翻译单元中。 这意味着， 每当 cpp 文件发生变化， 它们都要重新编译。

3.16开始，cmake提供一个命令 来启用 头文件预编译， 使得编译器可以单独处理 头文件和 实现文件。

```js
target_precompile_headers(<target>
  <INTERFACE|PUBLIC|PRIVATE> [header1...]
  [<INTERFACE|PUBLIC|PRIVATE> [header2...] ...])
```

添加的头文件列表 存储在 `PRECOMPILE_HEADERS` 目标属性中。
可以使用 传播属性 public或interface 将 头文件和 任何依赖的目标 共享。

但 对于 install 导出的目标， 不应该这样做。  其他项目不应该被迫使用 我们的预编译头文件。

重要提示: 如果你需要内部预编译头文件但仍然希望安装导出目标，那么第四章，《使用目标》中描述的`$<BUILD_INTERFACE:...>`生成器表达式将防止头文件出现在使用要求中。然而，它们仍然会被添加到使用export()命令从构建树导出的目标中。


cmake 会将所有头文件的名字 放入 `cmake_pch.h|xx` 的文件中， 然后 根据不同编译器，预编译为 .pch, .gch, .pchi

。。第二段是 create，但是还是不理解 怎么制作，怎么使用 预编译头文件，原理是什么? 。。 https://gcc.gnu.org/onlinedocs/gcc/Precompiled-Headers.html


例子:
```js
add_executable(precompiled hello.cpp)
target_precompile_headers(precompiled PRIVATE <iostream>)
```

```C++
int main() {  // 没有 头文件!
  std::cout << "hello world" << std::endl;
} 
```
请注意，在我们的main.cpp文件中，我们不需要包含cmake_pch.h或其他任何头文件——CMake 会使用特定的命令行选项强制包含它们


上面使用了 官方的 iostream 头文件， 也可以使用自己的:
- `header.h`， 相对于当前 源目录， 并且 包含到 cpp的时候 使用绝对路径
- `[["header.h"]]`， 根据编译器的实现来解释，通常可以在 INCLUDE_DIRECTORIES 变量中找到， 使用 `target_include_directories()` 来配置。


从一个target中 复用 预编译的头文件 到另一个target
```js
target_precompile_headers(<target> REUSE_FROM
  <other_target>)
```

这设置了使用头文件的目标的PRECOMPILE_HEADERS_REUSE_FROM属性，并在这些目标之间创建了一个依赖关系
使用这种方法，消费目标无法再指定自己的预编译头文件。
另外，所有编译选项、编译标志和编译定义必须在目标之间匹配。
注意要求，特别是如果你有任何使用双括号格式的头文件（`[["header.h"]]`）。两个目标都需要适当地设置它们的包含路径，以确保编译器能够找到这些头文件。

---

unity构建

3.16开始， 编译时间优化功能: 统一构建 或称为 巨构建。

将多个实现源文件 与 `#include` 指令结合


避免在 cmake创建统一构建文件时 在不同翻译单元 重新编译头文件
```C++
#include "source_a.cpp"
#include "source_b.cpp"
```
当这 2个 cpp文件 都包含 header.h 文件时， 多亏了 包含守卫 (#ifndef #define XXX_H), 它 只会被解析一次。  不如 预编译头 优雅，但是 可行。

这种构建的 第二个好处是， 优化器 现在可以 更大规模地使用， 并优化 所有捆绑源 之间的 跨过程调用。

这些好处 是有代价的， 我们减少了 对象文件的数量和处理步骤，但增加了处理更大文件所需的内存。 此外，我们减少了 并行化工作量。 编译器 并不擅长 多线程编译。 当我们把 所有文件放一起，会使它变得困难，因为 cmake 现在会在 我们创建的 多个统一构建 之间 安排 并行构建

在重新编译时，统一构建 不受欢迎，因为它们会编译 比所需更多的文件。
当代码 旨在 尽可能快 地整体编译所有文件时， 统一构建 效果最佳，可以提升 20%-50%

有2个选项，启用 统一构建
- 将 CMAKE_UNITY_BUILD 变量设置为 true， 它会为 定义在后面的 每个target 初始化 UNITY_BUILD 属性
- 手动将 target的 UNITY_BUILD 设置为 true

第二种方式:
```js
set_target_properties(<target1> <target2> ... 
                      PROPERTIES UNITY_BUILD true)
```

默认情况下，cmake 将创建 包含 8个源文件的 构建，这是由 UNITY_BUILD_BATCH_SIZE 属性指定的 ( 这个值 是 在创建target时 从 CMAKE_UNITY_BUILD_BATCH_SIZE 复制)。

3.18开始，你可以明确地定义文件如何与命名组一起打包。
将目标的UNITY_BUILD_MODE属性更改为GROUP（默认值始终为BATCH）。然后，你需要通过将他们的UNITY_GROUP属性设置为你选择的名称来为源文件分配组：
```C++
set_property(SOURCE <src1> <src2>... 
             PROPERTY UNITY_GROUP "GroupA") 
```
然后，CMake 将忽略UNITY_BUILD_BATCH_SIZE，并将组中的所有文件添加到单个巨构构建中。


CMake 的文档建议不要默认启用公共项目的统一构建。
建议您的应用程序的最终用户能够通过提供DCMAKE_UNITY_BUILD命令行参数来决定他们是否需要巨构构建。
更重要的是，如果由于您的代码编写方式而引起问题，您应该明确将目标属性设置为false。
然而，这并不妨碍您为内部使用的代码启用此功能，例如在公司内部或为您私人项目使用。


---


### 3.28开始，支持的C++20 module

通过 module，我们可以创建一个 带有模块声明的单文件，而不是创建一个单独的头部和实现文件。

```C++
export module hello_world;
import <iostream>; 
export void hello() {
    std::cout << "Hello world!\n";
}
```

使用时，导入即可
```C++
import hello_world;
int main() {
    hello();
}
```

这样，不再依赖预处理器。

。写书的时候 是3.20，还没有支持。


### 查找错误

`target_compile_options`

---

配置错误和警告

一个好的建议是 为 所有构建 启用 `-Werror` ，这样 所有警告都被视为 错误。 
但很难做到: 警告之所以是警告，它只是让人知道有这么一件事情，你决定 处理 or 忽略;   当 编译器升级时，可能带来 很多警告。

如果你觉得 有必要 吹毛求疵，可以使用 `-Wpedantic` ， 它启用了所有 严格遵循 ISO C 和 C++ 所要求的 警告。

更宽容 和 脚踏实地 的程序员 会对 `-Wall` 感到满意， 可选地 加上 `-Wextra`。

---

调试构建过程

在第一章中 已经知道了 如何打印 更详细的 cmake输出。

但如何分析 在每个阶段 实际发生的情况呢?

---

调试单个阶段

向编译器传递 `-save-temps` (gcc，clang都支持)， 它将 强制将 每个阶段的输出 存储到文件。

```js
add_executable(debug hello.cpp)
target_compile_options(debug PRIVATE -save-temps=obj)
```

上面的配置，会产生2个额外的文件: 
- `<build-tree>/CMakeFiles/<target>.dir/<source>.ii` 保存预处理阶段的输出
- `<build-tree>/CMakeFiles/<target>.dir/<source>.s`  保存语言分析阶段的输出(之后进入 汇编阶段)

根据问题的性质，我们通常可以发现实际的问题所在

---

解决头文件包含的调试问题

错误地包含文件 是一个 难以调试的问题。

如果要知道 使用了哪些 头文件， 使用 `-H`
```js
add_executable(debug hello.cpp)
target_compile_options(debug PRIVATE -H)
```

打印类似:
```text
[ 25%] Building CXX object 
  CMakeFiles/inclusion.dir/hello.cpp.o
. /usr/include/c++/9/iostream
.. /usr/include/x86_64-linux-gnu/c++/9/bits/c++config.h
... /usr/include/x86_64-linux-gnu/c++/9/bits/os_defines.h
.... /usr/include/features.h
-- removed for brevity --
.. /usr/include/c++/9/ostream
```

在这个输出的末尾，你也许还会找到对代码可能的改进建议：
```text
Multiple include guards may be useful for:
/usr/include/c++/9/clocale
/usr/include/c++/9/cstdio
/usr/include/c++/9/cstdlib
```


---

提供调试器信息

机器码是 机器读取的， 所以不包含任何的 可读性 (变量名，函数签名等)

我们可以要求 编译器将 源代码存储在 二进制文件中。 并且 将 源码 和编译后的代码 的映射 一起存储。  这样，通过 调试器 就可以 知道 在执行哪行源码。

- CMAKE_CXX_FLAGS_DEBUG 包含了-g。
- CMAKE_CXX_FLAGS_RELEASE 包含了-DNDEBUG。
     
-g 是添加调试信息。 这些信息可以被 gdb 使用。
-DNDEBUG ， 非调试构建， 一些 面向调试的 宏不会展开， 如 assert。


如果你在实践断言性编程的同时还需要在发布构建中使用assert()，你会怎么做？
要么更改 CMake 提供的默认设置（从CMAKE_CXX_FLAGS_RELEASE中移除NDEBUG）
要么通过在包含头文件前取消定义宏来实现硬编码覆盖：
```C++
#undef NDEBUG
#include <assert.h>
```





# ch6 使用CMake链接

`target_link_libraries` 是唯一一个 实际配置 链接 的属性 的命令

我们会
- 了解链接器如何工作
- 对象文件的 内部结构，如何重定位和 引用解析
- 可执行文件 与其他文件的区别， 系统如何构建 进程映像
- 介绍各种库: 静态库，共享库( 。就是动态库)，共享模块。
- odr， one definition rule, 单定义规则 ， 处理重复的符号
- 为什么有时 链接器找不到外部符号
- 使用 链接器 进行测试


## ==链接器基础知识==

elf格式的 对象文件

![90cc50afa076175e24091054ad51e692.png](../_resources/90cc50afa076175e24091054ad51e692.png)


编译器为每个翻译单元(.cpp文件) 准备一个 对象文件。 这些文件将用于构建我们程序的内存映像。对象文件包含以下元素：
- 一个 ELF头
- 按类型分组的信息段
- 一个段头表，包含关于 名称，类型，标志，内存中地址，文件中偏移量 等信息。 它用于理解这个文件中有哪些段以及它们的位置，就像目录一样。

编译器在处理源码时，会将收集到的信息进行分组:
- .text，机器代码，要执行的所有指令
- .data，所有初始化的全局和静态对象(变量)的值
- .bss，未初始化的全局和静态变量，将在程序启动时初始化为0
- .rodata，所有常量
- .strtab，常量字符串表
- .shstretab，包含所有段名称的字符串表

不能直接加载对象文件到内存，因为每个对象文件都有自己的集合。 因为，这样会浪费大量内存，并且 跨翻译单元的调用需要 跨文件 .text，需要大量CPU时钟(数万)

我们需要把 对象文件的 相同段 合并起来， 这被称为 Relocatable。
除了将 相应段放一起，还需要 更新文件内的 内部关联，即 变量的地址，函数的地址，符号表索引，字符串表索引。

链接器 还需要处理 extern。 编译器收集 未解决的 外部符号引用，在合并到 可执行文件后 找到 并填充 它们所在的地址。

最终的==可执行文件== 和 ==对象文件== 非常相似， 主要区别在于 程序头

![e4092f0982e95250fc308ab1df63dfd4.png](../_resources/e4092f0982e95250fc308ab1df63dfd4.png)

程序头 位于 elf头之后，系统加载器 读取此头 以创建 进程映像。
程序头 包含 一些通用信息 和内存布局的描述。 布局中 每个条目 代表一个 称为 段 的内存片段。 条目指定 要读取 哪些段，以什么顺序，及 虚拟内存中的哪些地址，它们的 标志是什么 (读，写或执行) 等。

对象文件 也可以被 打包到 库中，这是一种 中间产品。

---

## add_library,构建不同类型的库

在源码编译后，我们希望 避免在 同一个平台上 再次编译， 也希望 与外部项目共享。

最简单的是 直接提供所有的目标文件，但有几个缺点， 分发多个文件 并分别添加到 构建系统中 更加困难。

我们可以简单地 将 所有目标文件 合并到 一个 单一的目标中 并共享它。

我们可以使用 `add_library()` ( 与 target_link_libraries 匹配) 创建这些库

惯例， 所有库都共享一个 公共前缀 lib。
静态库, 在类unix上 后缀名 .a,   win后缀 .lib
共享库(。即 动态库)，类unix上 .so,  win .dll


---

### 静态库

构建静态库
`add_library(<name> [<source>...])`

如果`BUILD_SHARED_LIBS`变量没有设置为ON，上述代码将生成一个静态库。
如果我们想无论如何都构建一个静态库，我们可以提供一个显式的关键字：
`add_library(<name> STATIC [<source>...])`

静态库本质上 是一组存储在归档中的原始目标文件。 在unix上，`ar` ==创建== 归档文件。  
静态库是 最古老，最基本的 提供编译代码的方式。  

如果你想 避免 将你的依赖项 和 可执行文件分离 ( 。依赖 和 可执行文件 绑定在一起) ， 那么使用它们， 代价是 可执行文件的大小 和 占用内存 会增加。


---

### 共享库

SHARED 关键字
`add_library(<name> SHARED [<source>...])`

或 BUILD_SHARED_LIBS变量设置为ON: 
`add_library(<name> [<source>...])`

共享库 使用 链接器==创建==， 并将执行 链接器的 2个阶段。 这意味着我们会收到 带有 正确头，段，段头表的 文件。

共享库 可以在 多个不同 app 间共享。

OS 在第一个使用 共享库的 app 中 加载 库的实例 到内存， 后续的 app 都会收到 相同的 地址。
只有 ==.data, .bss 段 是每个进程 一份。==

---

### 共享模块

使用 MODULE 

`add_library(<name> MODULE [<source>...])`

作为插件 在==运行时加载==的 共享库， 而不是在 ==编译时== 与 可执行文件 链接的东西。

共享模块不会 随着 app启动 而自动加载。只有在app 通过 系统调用 ( win: LoadLibrary, linux: dlopen()/dlsym()) 明确请求后，才加载

不要尝试 将 共享模块 与 可执行文件 链接，  不能在所有平台上 保证有效。

---

### -fPIC 位置无关代码

所有共享库 和 模块的 源码 都应该使用 位置无关代码 标志 编译。

cmake 检查目标的 POSITION_INDENPENDENT_CODE 属性，并添加适当的 编译器标志: gcc/clang: -fPIC

在库的编译过程中，无法预知 库将放到 虚拟内存的哪个位置， 或 以什么顺序加载 ，这意味着 符号的地址 是未知的， 相对于 机器代码的位置也是未知的

为了解决这个问题，我们需要一个 中间层， PIC 为我们添加一个 新的节 到 输出中, .text 节在链接时是已知的。 因此所有 符号引用 可以在那时 指向占位符 GOT。
指向内存中符号的 实际值 将在 首次访问引用符号时填充。 那时，加载器将设置 GOT中特定条目的值

共享库 和模块 自动将 POSITION_INDENPENDENT_CODE 设置为 ON。
要记住，如果你的共享库被连接到 另一个目标，比如 静态库/对象库，那么 这个目标上 ==也需要设置==这个属性。
```js
set_target_properties(dependency_target 
                      PROPERTIES POSITION_INDEPENDENT_CODE
                      ON)
```
不这样做 会在cmake 上遇到麻烦， 因为默认情况下，此属性会以 传播属性冲突 被报告。



## ==使用ODR解决问题==

名称冲突导致定义不明确和不一致。


遵守ODR，意味着， 在单个翻译单元(.cpp) 的作用域内， 你需要且仅需要 定义一次， 即使 你多次声明 相同的名称 ( 变量，函数，类类型，枚举，概念，模板)。

反面 例子

shared.h : 
```C++
int i;
```

one.cpp:
```C++
#include <iostream>
#include "shared.h"
int main() {
  std::cout << i << std::endl;
}
```

tow.cpp:
```C++
#include "shared.h"
```

```js
cmake_minimum_required(VERSION 3.20.0)
project(ODR CXX)
set(CMAKE_CXX_STANDARD 20)
add_executable(odr one.cpp two.cpp)
```

上面的配置 很简单， 
创建一个 shared.h 头文件，
然后在 2个 独立的 翻译单元中使用。
然后 链接 2个 文件 形成 一个 可执行文件。
会收到下面的错误
```text
[100%] Linking CXX executable odr
/usr/bin/ld: CMakeFiles/odr.dir/two.cpp.o:(.bss+0x0): multiple definition of 'i'
; CMakeFiles/odr.dir/one.cpp.o:(.bss+0x0): first defined here
collect2: error: ld returned 1 exit status
```

你不能定义2次。  有一个例外: 类型，模板，外部内联函数 可以在 多个翻译单元中 重复定义，只要它们完全相同 ( 即 标记顺序相同)。 比如:
shared.h 改成
```C++
struct shared {
  static inline int i = 1;
};
```
其他不变， 是可以 编译 并链接 成功的。


或者 我们可以将 变量标记为 翻译单元局部。
`static int i;`
其他不变，可以成功。

---

### 动态链接的重复符号，按链接顺序先到先得

odr 对 静态库 和 对象文件的 作用 是一样的， 但是 对于 共享库 不同。

链接器 将运行 在此处 重复符号。

下面的例子中，创建 2个共享库 A，B， 然后 main 链接它们

```C++
// a.cpp
#include <iostream>
void a() {
  std::cout << "A" << std::endl;
}
void duplicated() {
  std::cout << "duplicated A" << std::endl;
}
```

```C++
// b.cpp // 和 a.cpp 非常类似
#include <iostream>
void b() {
  std::cout << "B" << std::endl;
}
void duplicated() {
  std::cout << "duplicated B" << std::endl;
}
```

```C++
// main.cpp
extern void a();
extern void b();
extern void duplicated();
int main() {
  a();
  b();
  duplicated();
}
```

这种情况下，==链接顺序很重要==

```js
cmake_minimum_required(VERSION 3.20.0)
project(Dynamic CXX)
add_library(a SHARED a.cpp)
add_library(b SHARED b.cpp)
add_executable(main_1 main.cpp)
target_link_libraries(main_1 a b)
add_executable(main_2 main.cpp)
target_link_libraries(main_2 b a)
```

```shell
root@ce492a7cd64b:/root/examples/chapter06/05-dynamic# b/main_1
A
B
duplicated A   // ...
root@ce492a7cd64b:/root/examples/chapter06/05-dynamic# b/main_2
A
B
duplicated B     // 。。
```

有一些例外， 如果我们定义本地可见符号，它们将优先于 动态库中的
比如，在main.cpp 中添加:
```C++
#include <iostream>
void duplicated() {
  std::cout << "duplicated MAIN" << std::endl;
}
```


#### 使用命名空间，不要依赖链接器

命名空间是为了避免上面的问题， 并以一种可管理的方式 处理 ODR。

所以 建议 使用 和 库同名的 命名空间 包裹你的 库代码。 这样可以 摆脱所有的 重复符号的问题。

我们可能会遇到这种情况: 共享库A 需要B，B需要C， 形成一条长链。 重要的是要记住， 将库链接到 另一个库 并不会 有任何的 命名空间继承。  这个链中的 每个符号 都保存在 最初编译它们的 命名空间中。


## 链接顺序和未知符号

考虑一个依赖链， 可执行文件 依赖 outer 库， outer 库依赖 nested 库 (包含必要的 int b;)

但是收到错误: `outer.cpp:(.text+0x1f): undefined reference to 'b'`

通常，这意味着 我们忘记向链接器 添加一个必要的库。 但是，库实际上已经正确添加了 `target_link_libraries()` 中
```js
cmake_minimum_required(VERSION 3.20.0)
project(Order CXX)
add_library(outer outer.cpp)
add_library(nested nested.cpp)
add_executable(main main.cpp)
target_link_libraries(main nested outer)
```
。。这个 我知道，只会往后找，直到找到，不会往前的。


解决未定义符号的方式 是这样的: 链接器 从左到右 处理二进制文件， 当链接器 遍历二进制文件时，它将执行以下操作
1. 收集此二进制文件导出的 所有未定义符号 并 存储
2. 尝试使用 此二进制文件中 定义的符号 解决 至今收集到的未定义符号
3. 对下一个二进制文件 重复此过程

只需要调整顺序 就可以
`target_link_libraries(main outer nested)`

还有一个不太优雅的选项是 重复库 ( 这对 循环引用很有用)
`target_link_libraries(main nested outer nested)`

最后，我们可以尝试使用 链接器 特定的标志，如 `--start-group` `--end-group`

。。 这2个选项之间的 静态库 可以任意顺序，不会出现 找不到引用的问题。
```js
target_link_libraries(
    x
    -Wl,--start-group
    libX3.a
    libX2.a
    libX1.a
    -Wl,--end-group
)
```
。。。



## main

链接器强制执行odr，并确保 在链接过程中 为所有外部符号 提供它们的定义。

C++的编译性质实际上并不支持可以仅用于测试目的而临时注入到二进制文件中的可插拔单元。这似乎需要一个相当复杂的解决方案。

幸运的是，我们可以使用链接器以优雅的方式帮助我们处理这个问题。考虑将您程序的main()中的所有逻辑提取到一个外部函数start_program()中
```C++
// main.cpp
extern int start_program(int, const char**);
int main(int argc, const char** argv) {
  return start_program(argc, argv);
}
```

```C++
// program.cpp
#include <iostream>
int start_program(int argc, const char** argv) {
  if (argc <= 1) {
    std::cout << "Not enough arguments" << std::endl;
    return 1;
  }
  return 0;
}
```

```js
cmake_minimum_required(VERSION 3.20.0)
project(Testing CXX)
add_library(program program.cpp)
add_executable(main main.cpp)
target_link_libraries(main program)
```

main目标只是提供了所需的main()函数。program目标包含了所有的逻辑。
现在我们可以通过创建另一个包含其自己的main()和测试逻辑的可执行文件来测试它。

在现实场景中，像main()方法这样的框架可以用来替换程序的入口点并运行所有定义的测试。
我们将在第八章深入研究实际的测试主题，测试框架。
现在，让我们关注通用原则，并在另一个main()函数中编写我们自己的测试：
```C++
#include <iostream>
extern int start_program(int, const char**);
using namespace std;
int main() {
  auto exit_code = start_program(0, nullptr);
  if (exit_code == 0)
    cout << "Non-zero exit code expected" << endl;
  const char* arguments[2] = {"hello", "world"};
  exit_code = start_program(2, arguments);
  if (exit_code != 0)
    cout << "Zero exit code expected" << endl;
}
```

前面的代码将两次调用start_program，带参数和不带参数，并检查返回的退出码是否正确。
这个单元测试在代码整洁和优雅的测试实践方面还有很多不足，但至少它是一个开始。
重要的是，我们现在定义了两次main()：
- 用于生产环境的main.cpp
- 用于测试目的的test.cpp
     
我们现在将在CMakeLists.txt的底部添加第二个可执行文件：
```js
add_executable(test test.cpp)
target_link_libraries(test program)
```

这创建了另一个目标，它与生产中的完全相同的二进制代码链接，但它允许我们以任何喜欢的方式调用所有导出的函数。
得益于这一点，我们可以自动运行所有代码路径，并检查它们是否如预期般工作。太好了！




# ch7 使用CMake管理依赖项


## find_package,找到已安装的包

系统中已经安装了依赖项，使用 find_package

以安装protobuf为例
```sh
apt update
apt install protobuf-compiler libprotobuf-dev
```

每个系统都有它自己的 安装和管理包的方式， 找到一个包所在的路径可能会很棘手。

如果包提供了一个合适的配置文件，允许cmake确定支持该包所需的变量，那么 find_package 可以找到包。
许多项目都可以，它们在安装过程中提供了 这个文件给cmake。  
如果某个流行的库没有提供此文件，那么可能 已经内置到 cmake中了 ( 这些被称为 find-modules)
如果不是，那么还有一些选择:
- 为某个包 提供我们自己的 find-modules
- 编写一个配置文件，请 包维护者 将 该包和文件 一起分发


cmake提供了 150多个查找模块，可以查找各种各样的 库， 列表: https://cmake.org/cmake/help/latest/manual/cmake-modules.7.html#find%20modules


查找模块 和配置文件 都可以在 cmake中使用 find_package()， 
==先找 模块，找不到就找 配置文件==。搜索将从 CMAKE_MODULE_PATH 变量中的路径开始 (默认情况下，是空的)。
最后，找 cmake的内置模块

---

cmake 中预设了很多 适合宿主系统的 路径，搜索 匹配的文件名
- `<CamelCasePackageName>Config.cmake`
- `<kebab-case-package-name>-config.cmake`

---


```js
// message.proto
syntax = "proto3";
message Message {
    int32 id = 1;
}
```
```C++
#include "message.pb.h"
#include <fstream>
using namespace std;
int main()
{
  Message m;
  m.set_id(123);
  m.PrintDebugString();
  fstream fo("./hello.data", ios::binary | ios::out);
  m.SerializeToOstream(&fo);
  fo.close();
  return 0;
}
```
我已经包含了一个message.pb.h头文件。这个文件还不存在；它需要在message.proto编译期间由 Protobuf 编译器protoc创建。

```C++
cmake_minimum_required(VERSION 3.20.0)
project(FindPackageProtobufVariables CXX)
find_package(Protobuf REQUIRED)
protobuf_generate_cpp(GENERATED_SRC GENERATED_HEADER
  message.proto)
add_executable(main main.cpp 
  ${GENERATED_SRC} ${GENERATED_HEADER})
target_link_libraries(main PRIVATE ${Protobuf_LIBRARIES})
target_include_directories(main PRIVATE 
  ${Protobuf_INCLUDE_DIRS}
  ${CMAKE_CURRENT_BINARY_DIR})
```

- 前两行我们已经知道了；它们创建了一个项目和声明了它的语言。
- find_package(Protobuf REQUIRED) 要求 CMake 运行捆绑的FindProtobuf.cmake查找模块，并为我们设置 Protobuf 库。那个查找模块将扫描常用路径（因为我们提供了REQUIRED关键字）并在找不到库时终止。它还将指定有用的变量和函数（如下面的行所示）。
- protobuf_generate_cpp 是 Protobuf 查找模块中定义的自定义函数。在其内部，它调用add_custom_command()，该命令使用适当的参数调用protoc编译器。我们通过提供两个变量来使用这个函数，这些变量将被填充生成的源文件（GENERATED_SRC）和头文件（GENERATED_HEADER）的路径，以及要编译的文件列表（message.proto）。
- 如我们所知，add_executable 将使用main.cpp和前面命令中配置的 Protobuf 文件创建我们的可执行文件。
- target_link_libraries 将由find_package()找到的（静态或共享）库添加到我们的main目标链接命令中。
- target_include_directories() 将必要的INCLUDE_DIRS（由包提供）添加到包含路径中，以及CMAKE_CURRENT_BINARY_DIR。后者是必需的，以便编译器可以找到生成的message.pb.h头文件。

换句话说，它实现了以下功能：
- 查找库和编译器的所在位置
- 提供辅助函数，教会 CMake 如何调用.proto文件的定制编译器
- 添加包含包含和链接所需路径的变量
     

find_package 找到包后，下面的变量会被设置
- `<PKG_NAME>_FOUND`
- `<PKG_NAME>_INCLUDE_DIRS` 或 `<PKG_NAME>_INCLUDES`
- `<PKG_NAME>_LIBRARIES` 或 `<PKG_NAME>_LIBRARIES` 或 `<PKG_NAME>_LIBS`
- `<PKG_NAME>_DEFINITIONS`
- 由查找模块或配置文件指定的 IMPORTED 目标


如果一个包支持所谓的“现代 CMake”（以目标为中心），它将提供这些IMPORTED目标而不是这些变量，这使得代码更简洁、更简单。
建议优先使用目标而不是变量。

Protobuf 是一个很好的例子，因为它提供了变量和IMPORTED目标（自从 CMake 3.10 以来）：protobuf::libprotobuf，protobuf::libprotobuf-lite，protobuf::libprotoc和protobuf::protoc。这允许我们编写更加简洁的代码
```js
cmake_minimum_required(VERSION 3.20.0)
project(FindPackageProtobufTargets CXX)
find_package(Protobuf REQUIRED)
protobuf_generate_cpp(GENERATED_SRC GENERATED_HEADER
  message.proto)
add_executable(main main.cpp
  ${GENERATED_SRC} ${GENERATED_HEADER})
target_link_libraries(main PRIVATE protobuf::libprotobuf)
target_include_directories(main PRIVATE
                                ${CMAKE_CURRENT_BINARY_DIR})
```

protobuf::libprotobuf导入的目标隐式地指定了包含目录，并且多亏了传递依赖（或者我叫它们传播属性），它们与我们的main目标共享。链接器和编译器标志也是同样的过程。

如果你需要确切知道特定 find-module 提供了什么，最好是访问其在线文档。Protobuf 的一个可以在以下位置找到：cmake.org/cmake/help/latest/module/FindProtobuf.html


为了保持简单，本节中的示例如果用户系统中没有找到 protobuf 库（或其编译器）将简单地失败。
但一个真正健壮的解决方案应该通过检查Protobuf_FOUND变量并相应地行事，要么打印给用户的清晰诊断消息（这样他们可以安装它）要么自动执行安装。

---

find_package 完整的签名很长，下面是 基本的签名

`find_package(<Name> [version] [EXACT] [QUIET] [REQUIRED])`

- `[version]`，它允许我们选择性地请求一个特定的版本。使用major.minor.patch.tweak格式（如1.22）或提供一个范围——1.22...1.40.1（使用三个点作为分隔符）。
- EXACT 关键字意味着我们想要一个确切的版本（这里不支持版本范围）。
- QUIET 关键字可以静默所有关于找到/未找到包的消息。
- REQUIRED 关键字如果找不到包将停止执行，并打印一个诊断消息（即使启用了QUIET也是如此）。

有关命令的更多信息可以在文档页面找到：cmake.org/cmake/help/latest/command/find_package.html。



## 使用 FindPkgConfig 发现遗留包

pkg-config正逐渐被其他更现代的解决方案所取代。
这里出现了一个==问题——你应该投入时间支持它吗？答案==一如既往——视情况而定：
- 如果一个库真的很受欢迎，它可能已经有了自己的 CMake find-module；在这种情况下，你可能不需要它。
- 如果没有 find-module（或者它不适用于您的库）并且库只提供 PkgConfig 的 .pc文件，只需使用现成的即可。
     
许多（如果不是大多数）库已经采用了 CMake，并在当前版本中提供了包配置文件。如果您不发布您的解决方案并且您控制环境，请使用find_package()，不要担心遗留版本。

在任何情况下，==首先使用find_package()==，如前一部分所述，如果`<PKG_NAME>_FOUND`为假，则==退回到 PkgConfig==。这样，我们覆盖了一种场景，即环境升级后我们只需使用主方法而无需更改代码。

```js
// foobar.pc
prefix=/usr/local
exec_prefix=${prefix}
includedir=${prefix}/include
libdir=${exec_prefix}/lib
Name: foobar
Description: A foobar library
Version: 1.0.0
Cflags: -I${includedir}/foobar
Libs: -L${libdir} -lfoobar
```

这个格式相当直接，轻量级，甚至支持基本变量扩展
尽管 PkgConfig 极其易于使用，但其功能却相当有限：
- 检查系统中是否存在库，并且是否提供了与之一起的.pc文件
- 检查是否有一个库的足够新版本可用
- 通过运行 pkg-config --libs libfoo 获取库的链接器标志
- 获取库的包含目录（此字段技术上可以包含其他编译器标志）——pkg-config --cflags libfoo
     
FindPkgConfig 它遵循大多数常规查找模块的规则，但不是提供PKG_CONFIG_INCLUDE_DIRS或PKG_CONFIG_LIBS变量，而是设置一个变量，直接指向二进制文件的路径——PKG_CONFIG_EXECUTABLE。
不出所料，PKG_CONFIG_FOUND变量也被设置了——我们将使用它来确认系统中是否有这个工具，然后使用模块中定义的pkg_check_modules()帮助命令扫描一个pkg_check_modules()包。



例子
一个提供 .pc 文件的相对受欢迎的库的一个例子是一个 PostgreSQL 数据库的客户端——libpqxx

debian 安装: `apt-get install libpqxx-dev`

```C++
// main.cpp
#include <pqxx/pqxx>
int main()
{
  // We're not actually connecting, but
  // just proving that pqxx is available.
  pqxx::nullconnection connection;
}
```

```js
cmake_minimum_required(VERSION 3.20.0)
project(FindPkgConfig CXX)
find_package(PkgConfig REQUIRED)
pkg_check_modules(PQXX REQUIRED IMPORTED_TARGET libpqxx)
message("PQXX_FOUND: ${PQXX_FOUND}")
add_executable(main main.cpp)
target_link_libraries(main PRIVATE PkgConfig::PQXX)
```

- 我们要求 CMake 使用find_package()命令==查找 PkgConfig== 可执行文件。如果因为REQUIRED关键字而没有pkg-config，它将会失败。
- 在FindPkgConfig查找模块中定义的==pkg_check_modules()自定义宏==被调用，以创建一个名为PQXX的新==IMPORTED目标==。查找模块将搜索一个名为libpxx的依赖项，同样，因为REQUIRED关键字，如果库不可用，它将会失败。注意==IMPORTED_TARGET关键字==——没有它，就不会自动创建目标，我们必须手动定义由宏创建的变量。
- 我们通过打印PQXX_FOUND来确认一切是否正确，并显示诊断信息。如果我们之前没有指定REQUIRED，我们在这里可以检查这个变量是否被设置（也许是为了允许其他备选机制介入）。
- 我们创建了main可执行文件。
- 我们链接了==由pkg_check_modules()创建的PkgConfig::PQXX IMPORTED目标==。注意PkgConfig::是一个常量前缀，PQXX来自传递给该命令的第一个参数。


这是一种相当方便的方法，可以引入尚不支持 CMake 的依赖项。这个查找模块还有其他一些方法和选项；如果你对了解更多感兴趣，我建议你参考官方文档：cmake.org/cmake/help/latest/module/FindPkgConfig.html。



## 编写自己的 find-modules

上一节中熟悉了libpqxx，那么现在就让我们为它编写一个好的查找模块吧。
首先在项目中 源代码树的cmake/module目录 下创建一个新文件 FindPQXX.cmake，并开始编写。
我们需要确保当调用find_package()时，CMake 能够发现这个查找模块，因此我们将这个路径添加到CMakeLists.txt中的==CMAKE_MODULE_PATH==变量里，用list(APPEND)

```js
// 根目录 CMakeLists.txt
cmake_minimum_required(VERSION 3.20.0)
project(FindPackageCustom CXX)
list(APPEND CMAKE_MODULE_PATH
  "${CMAKE_SOURCE_DIR}/cmake/module/")
find_package(PQXX REQUIRED)
add_executable(main main.cpp)
target_link_libraries(main PRIVATE PQXX::PQXX)
```

---

从技术上讲，如果FindPQXX.cmake文件为空，将不会有任何事情发生：即使用户调用find_package()时使用了REQUIRED，CMake 也不会抱怨一些特定的变量没有被设置（包括PQXX_FOUND），这是==查找模块的作者需要==尊重 CMake 文档中概述的约定：
- 当调用`find_package(<PKG_NAME> REQUIRED)`时，CMake 将提供一个`<PKG_NAME>_FIND_REQUIRED`变量，设置为1。如果找不到库，查找模块应该调用message(FATAL_ERROR)。
- 当调用`find_package(<PKG_NAME> QUIET)`时，CMake 将提供一个`<PKG_NAME>_FIND_QUIETLY`变量，设置为1。查找模块应避免打印诊断信息（除了前面提到的一次）。
- 当调用列表文件时，CMake 将提供一个`<PKG_NAME>_FIND_VERSION`变量，设置为所需版本。查找模块应该找到适当的版本，或者发出FATAL_ERROR。

---


当然，为了与其他查找模块保持一致性，最好遵循前面的规则。让我们讨论创建一个优雅的PQXX查找模块所需的步骤：
- 如果已知库和头文件的路径（要么由用户提供，要么来自之前运行的缓存），使用这些路径并创建一个IMPORTED目标。在此结束。
- 否则，请找到嵌套依赖项——PostgreSQL 的库和头文件。
- 在已知的路径中搜索 PostgreSQL 客户端库的二进制版本。
- 在已知的路径中搜索 PostgreSQL 客户端包含头文件。
- 检查是否找到了库和包含头文件；如果是，创建一个IMPORTED目标。


创建IMPORTED目标发生了两次——如果用户从命令行提供了库的路径，或者如果它们是自动找到的。我们将从编写一个函数来处理我们搜索过程的结果开始，并保持我们的代码 DRY。
要创建一个IMPORTED目标，我们只需要一个带有IMPORTED关键字的库（以便在CMakeLists.txt中的target_link_libraries()命令中使用它）。该库必须提供一个类型——我们将其标记为UNKNOWN，以表示我们不希望检测找到的库是静态的还是动态的；我们只想为链接器提供一个参数。
接下来，我们将IMPORTED_LOCATION和INTERFACE_INCLUDE_DIRECTORIES``IMPORTED目标的必需属性设置为函数被调用时传递的参数。我们还可以指定其他属性（如COMPILE_DEFINITIONS）；它们对于PQXX来说只是不必要的。
在那之后，我们将路径存储在缓存变量中，这样我们就无需再次执行搜索。值得一提的是，PQXX_FOUND在缓存中被显式设置，因此它在全局变量作用域中可见（所以它可以被用户的CMakeLists.txt访问）。
最后，我们将缓存变量标记为高级，这意味着除非启用“高级”选项，否则它们不会在 CMake GUI 中显示。对于这些变量，这是一种常见的做法，我们也应该遵循约定：

```js
// FindPQXX.cmake
function(add_imported_library library headers)
  add_library(PQXX::PQXX UNKNOWN IMPORTED)
  set_target_properties(PQXX::PQXX PROPERTIES
    IMPORTED_LOCATION ${library}
    INTERFACE_INCLUDE_DIRECTORIES ${headers}
  )
  set(PQXX_FOUND 1 CACHE INTERNAL "PQXX found" FORCE)
  set(PQXX_LIBRARIES ${library}
      CACHE STRING "Path to pqxx library" FORCE)
  set(PQXX_INCLUDES ${headers}
      CACHE STRING "Path to pqxx headers" FORCE)
  mark_as_advanced(FORCE PQXX_LIBRARIES)
  mark_as_advanced(FORCE PQXX_INCLUDES)
endfunction()
```

接下来，我们覆盖第一种情况——一个用户如果将他们的PQXX安装在非标准位置，可以通过命令行（使用-D参数）提供必要的路径。如果是这种情况，我们只需调用我们刚刚定义的函数并使用return()放弃搜索。我们相信用户最清楚，能提供库及其依赖项（PostgreSQL）的正确路径给我们。
如果配置阶段在过去已经执行过，这个条件也将为真，因为PQXX_LIBRARIES和PQXX_INCLUDES变量是被缓存的。
```js
if (PQXX_LIBRARIES AND PQXX_INCLUDES)
  add_imported_library(${PQXX_LIBRARIES} ${PQXX_INCLUDES})
  return()
endif()
```

---

是时候找到一些嵌套依赖项了。
为了使用PQXX，宿主机器还需要 PostgreSQL。
在我们的查找模块中使用另一个查找模块是完全合法的，但我们应该将REQUIRED和QUIET标志传递给它（以便嵌套搜索与外层搜索行为一致）。
这不是复杂的逻辑，但我们应该尽量避免不必要的代码。

CMake 有一个内置的帮助宏，正是为此而设计——find_dependency()。
有趣的是，文档中指出它不适合用于 find-modules，因为它如果在找不到依赖项时调用return()命令。
因为这是一个宏（而不是一个函数），return()将退出调用者的作用域，即FindPQXX.cmake文件，停止外层 find-module 的执行。
可能有些情况下这是不希望的，但在这个情况下，这正是我们想要做的——阻止 CMake 深入寻找PQXX的组件，因为我们已经知道 PostgreSQL 不可用：

```js
# deliberately used in mind-module against the documentation
include(CMakeFindDependencyMacro)
find_dependency(PostgreSQL)
```

---

为了找到PQXX库，我们将设置一个_PQXX_DIR帮助变量（转换为 CMake 风格的路径）并使用find_library()命令扫描我们在PATHS关键字之后提供的路径列表。
该命令将检查是否有与NAMES关键字之后提供的名称匹配的库二进制文件。
如果找到了匹配的文件，其路径将被存储在PQXX_LIBRARY_PATH变量中。
否则，该变量将被设置为`<VAR>-NOTFOUND`，在这种情况下是PQXX_HEADER_PATH-NOTFOUND。

NO_DEFAULT_PATH关键字禁用了默认行为，这将扫描 CMake 为该主机环境提供的默认路径列表：

```js
file(TO_CMAKE_PATH "$ENV{PQXX_DIR}" _PQXX_DIR)
find_library(PQXX_LIBRARY_PATH NAMES libpqxx pqxx
  PATHS
    ${_PQXX_DIR}/lib/${CMAKE_LIBRARY_ARCHITECTURE}
    # (...) many other paths - removed for brevity 
    /usr/lib
  NO_DEFAULT_PATH
)
```

---

接下来，我们将使用find_path()命令搜索所有已知的头文件，这个命令的工作方式与find_library()非常相似。
主要区别在于find_library()知道库的系统特定扩展，并将这些扩展作为需要自动添加，而对于find_path()，我们需要提供确切的名称。

在这里也不要混淆pqxx/pqxx。这是一个实际的头文件，但库作者故意省略了扩展名，以符合 C++风格#include指令（而不是遵循 C 风格.h扩展名）：`#include <pqxx/pqxx>`

```js
find_path(PQXX_HEADER_PATH NAMES pqxx/pqxx
  PATHS
    ${_PQXX_DIR}/include
    # (...) many other paths - removed for brevity
    /usr/include
  NO_DEFAULT_PATH
)
```

---

现在是检查PQXX_LIBRARY_PATH和PQXX_HEADER_PATH变量是否包含任何-NOTFOUND值的时候。
同样，我们可以手动进行这项工作，然后根据约定打印诊断信息或终止构建执行，或者我们可以使用 CMake 提供的 FindPackageHandleStandardArgs 列表文件中的 find_package_handle_standard_args()帮助函数。
这是一个帮助命令，如果路径变量被填充，则将`<PKG_NAME>_FOUND`变量设置为1，并提供关于成功和失败的正确诊断信息（它将尊重QUIET关键字）。
如果传递了REQUIRED关键字给 find-module，而其中一个提供的路径变量为空，它还将以FATAL_ERROR终止执行。

如果找到了库，我们将调用函数定义IMPORTED目标并将路径存储在缓存中：

```js
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(
  PQXX DEFAULT_MSG PQXX_LIBRARY_PATH PQXX_HEADER_PATH
)
if (PQXX_FOUND)
  add_imported_library(
    "${PQXX_LIBRARY_PATH};${POSTGRES_LIBRARIES}"
    "${PQXX_HEADER_PATH};${POSTGRES_INCLUDE_DIRECTORIES}"
  )
endif()
```

就这些。这个 find-module 将找到PQXX并创建相应的PQXX::PQXX目标。你可以在整个文件中找到这个模块，文件位于书籍示例仓库中：chapter07/04-find-package-custom/cmake/module/FindPQXX.cmake。

如果一个库很受欢迎，并且很可能会在系统中已经安装，这种方法非常有效。然而，并非所有的库随时都能获得。我们能否让这个步骤变得简单，让我们的用户使用 CMake 获取和构建这些依赖项？




## 与git仓库协作

许多项目依赖于 Git 作为版本控制系统。
假设我们的项目和外部库都在使用它，有没有某种 Git 魔法能让我们把这些仓库链接在一起？
我们能否构建库的特定（或最新）版本，作为构建我们项目的一个步骤？如果是，怎么做？


---

### 通过 Git 子模块提供外部库

一个可能的解决方案是使用 Git 内置的机制，称为Git 子模块。
子模块允许项目仓库使用其他 Git 仓库，而实际上不将引用的文件添加到项目仓库中。
它们的工作方式与软链接类似——它们指向外部仓库中的特定分支或提交（但你需要显式地更新它们）。

要向你的仓库中==添加==一个子模块（并克隆其仓库），执行以下命令：
`git submodule add <repository-url>`

如果你拉取了一个==已经包含子模块==的仓库，你需要==初始化==它们：
`git submodule update --init -- <local-path-to-submodule>`

一个小缺点是，当用户克隆带有根项目的仓库时，子模块不会自动拉取。需要一个显式的init/pull命令

---

首先，让我们看看我们如何在代码中使用一个新创建的子模块。

为了这个例子，我决定写一个小程序，从 YAML 文件中读取一个名字，并在欢迎消息中打印出来。
YAML 是一种很好的简单格式，用于存储可读的配置，但机器解析起来相当复杂。
我找到了一个由 Jesse Beder（及当时 92 名其他贡献者）解决这个问题的整洁小型项目，称为 yaml-cpp(github.com/jbeder/yaml-cpp)。
这个例子相当直接。它是一个问候程序，打印出欢迎<名字>的消息。name的默认值将是Guest，但我们可以在 YAML 配置文件中指定一个不同的名字。以下是代码：

```C++
// root/main.cpp
#include <string>
#include <iostream>
#include "yaml-cpp/yaml.h"
using namespace std;
int main() {
  string name = "Guest";
  YAML::Node config = YAML::LoadFile("config.yaml");
  if (config["name"])
    name = config["name"].as<string>();
  cout << "Welcome " << name << endl;
  return 0;
}
```

```yaml
# root/config.yaml
name: Rafal
```

---

main用到了"yaml-cpp/yaml.h"头文件。为了使其可用，我们需要克隆yaml-cpp项目并构建它。
让我们创建一个extern目录来存储所有第三方依赖项（如第三章、设置你的第一个 CMake 项目部分中所述）并添加一个 Git 子模块，引用库的仓库：

```shell
$ mkdir extern
$ cd extern
$ git submodule add https://github.com/jbeder/yaml-cpp.git
Cloning into 'chapter07/01-git-submodule-manual/extern/yaml-cpp'...
remote: Enumerating objects: 8134, done.
remote: Total 8134 (delta 0), reused 0 (delta 0), pack-reused 8134
Receiving objects: 100% (8134/8134), 3.86 MiB | 3.24 MiB/s, done.
Resolving deltas: 100% (5307/5307), done.
```

---

Git 已经克隆了仓库；现在我们可以将其作为项目的依赖项，并让 CMake 负责构建：

```js
// root/CMakeLists.txt
cmake_minimum_required(VERSION 3.20.0)
project(GitSubmoduleManual CXX)
add_executable(welcome main.cpp)
configure_file(config.yaml config.yaml COPYONLY)
add_subdirectory(extern/yaml-cpp)
target_link_libraries(welcome PRIVATE yaml-cpp)
```

1. 设置项目并添加我们的welcome可执行文件。
2. 接下来，调用configure_file，但实际上不配置任何内容。通过提供COPYONLY关键字，我们只是将我们的config.yaml复制到构建树中，这样可执行文件在运行时能够找到它。
3. 添加 yaml-cpp 仓库的子目录。CMake 会将其视为项目的一部分，并递归执行任何嵌套的CMakeLists.txt文件。
4. 将库提供的yaml-cpp目标与welcome目标链接。

yaml-cpp 的作者遵循在第三章《设置你的第一个 CMake 项目》中概述的实践，并将公共头文件存储在单独的目录中——<项目名称>/include/<项目名称>。
这允许库的客户（如main.cpp）通过包含"yaml-cpp/yaml.h"库名称的路径来访问这些文件。这种命名实践非常适合发现——我们立即知道是哪个库提供了这个头文件。

这并不是一个非常复杂的过程，但它并不理想——用户在克隆仓库后必须手动初始化我们添加的子模块。
更糟糕的是，它没有考虑到用户可能已经在他们的系统上安装了这个库



### 自动初始化 Git 子模块

如果一个库提供了一个包配置文件，我们只需让find_package()在安装的库中搜索它。
正如承诺的那样，CMake 首先检查是否有合适的 find 模块，如果没有，它将寻找配置文件。

我们已经知道，如果`<LIB_NAME>_FOUND`变量被设置为1，则库被找到，我们可以直接使用它。
我们也可以在库未找到时采取行动，并提供方便的解决方法来默默改善用户的体验：退回到获取子模块并从源代码构建库。

让我们将上一个示例中的代码进行扩展：

```js
cmake_minimum_required(VERSION 3.20.0)
project(GitSubmoduleAuto CXX)
add_executable(welcome main.cpp)
configure_file(config.yaml config.yaml COPYONLY)
find_package(yaml-cpp QUIET)
if (NOT yaml-cpp_FOUND)
  message("yaml-cpp not found, initializing git submodule")
  execute_process(
    COMMAND git submodule update --init -- extern/yaml-cpp
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  )
  add_subdirectory(extern/yaml-cpp)
endif()
target_link_libraries(welcome PRIVATE yaml-cpp)
```

- 我们将尝试悄悄地查找 yaml-cpp 并使用它。
- 如果它不存在，我们将打印一个简短的诊断信息，并使用execute_process()命令来初始化子模块。这实际上是从引用仓库中克隆文件。
- 最后，我们将add_subdirectory()用于从源代码构建依赖项。

简短而精炼。这也适用于未使用 CMake 构建的库——我们可以遵循 git submodule 的示例，再次调用 execute_process() 以同样的方式启动任何外部构建工具。


---


### 为不使用 Git 的项目克隆依赖项

如果您使用另一个 VCS 或者提供源代码的存档，您可能会在依赖 Git submodules 将外部依赖项引入您的仓库时遇到困难。很有可能是构建您代码的环境安装了 Git 并能执行 git clone 命令。


```js
cmake_minimum_required(VERSION 3.20.0)
project(GitClone CXX)
add_executable(welcome main.cpp)
configure_file(config.yaml config.yaml COPYONLY)
find_package(yaml-cpp QUIET)
if (NOT yaml-cpp_FOUND)
  message("yaml-cpp not found, cloning git repository")
  find_package(Git)
  if (NOT Git_FOUND)
    message(FATAL_ERROR "Git not found, can't initialize!")
  endif ()
  execute_process(
    COMMAND ${GIT_EXECUTABLE} clone
    https://github.com/jbeder/yaml-cpp.git
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/extern
  )  
  add_subdirectory(extern/yaml-cpp)
endif()
target_link_libraries(welcome PRIVATE yaml-cpp)
```

- 首先，我们通过 FindGit 查找模块检查 Git 是否可用。
- 如果不可以使用，我们就束手无策了。我们将发出 FATAL_ERROR，并希望用户知道接下来该做什么。
- 否则，我们将使用 FindGit 查找模块设置的 GIT_EXECUTABLE 变量调用 execute_process() 并克隆我们感兴趣的仓库。

。。? ? 就是 使用git下载， 但是不是说 不使用git吗。
。。可能是 我们的工程不使用git， 所以没有 办法 git submodule。 只能 git clone。





## 使用 ExternalProject，FetchContent模块

在线 CMake 参考书籍将建议使用 ExternalProject 和 FetchContent 模块来处理更复杂项目中依赖项的管理。这实际上是个好建议，但它通常在没有适当上下文的情况下给出。


### 外部项目

CMake 3.0.0 引入了一个 名为 ExternalProject 的模块。 目的 是为了添加 对在线仓库中 可用的 外部项目的支持。
该模块逐渐扩展以满足不同的需求，最终变得相当复杂的命令——ExternalProject_Add()
它接受超过 85 个不同的选项。不足为奇，因为它提供了一组令人印象深刻的特性：
- 为外部项目管理目录结构
- 从 URL 下载源代码（如有需要，从归档中提取）
- 支持 Git、Subversion、Mercurial 和 CVS 仓库
- 如有需要，获取更新
- 使用 CMake、Make 配置和构建项目，或使用用户指定的工具
- 执行安装和运行测试
- 记录到文件
- 从终端请求用户输入
- 依赖于其他目标
- 向构建过程中添加自定义命令/步骤

ExternalProject 模块在构建阶段填充依赖项。对于通过 ExternalProject_Add() 添加的每个外部项目，CMake 将执行以下步骤：
- mkdir – 为外部项目创建子目录
- download – 从仓库或 URL 获取项目文件
- update – 在支持差量更新的下载方法中重新运行时更新文件
- patch – 可选执行一个补丁命令，用于修改下载文件以满足项目需求
- configure – 为 CMake 项目执行配置阶段，或为非 CMake 依赖手动指定命令
- build – 为 CMake 项目执行构建阶段，对于其他依赖项，执行 make 命令
- install – 安装 CMake 项目，对于其他依赖项，执行 make install 命令
- test – 如果定义了任何 TEST_... 选项，则执行依赖项的测试

步骤按照前面的确切顺序进行，除了 test 步骤，该步骤可以通过 `TEST_BEFORE_INSTALL <bool>` 或 `TEST_AFTER_INSTALL <bool>` 选项在 install 步骤之前或之后可选地启用。


---

下载步骤选项

我们主要关注控制 download 步骤或 CMake 如何获取依赖项的选项。
首先，我们可能选择不使用 CMake 内置的此方法，而是提供一个自定义命令（在此处支持生成器表达式）：
`DOWNLOAD_COMMAND <cmd>...`

这样做后，我们告诉 CMake 忽略此步骤的所有其他选项，只需执行一个特定于系统的命令。空字符串也被接受，用于禁用此步骤。

---

从 URL 下载依赖项

我们可以提供一系列 URL，按顺序扫描直到下载成功。CMake 将识别下载文件是否为归档文件，并默认进行解压
`URL <url1> [<url2>...]`

其他选项允许我们进一步自定义此方法的行为：
- `URL_HASH <algo>=<hashValue>` – 检查通过 `<algo>` 生成的下载文件的校验和是否与提供的 `<hashValue>` 匹配。建议确保下载的完整性。支持的算法包括 MD5、SHA1、SHA224、SHA256、SHA384、SHA512、SHA3_224、SHA3_256、SHA3_384 和 SHA3_512，这些算法由 string(`<HASH>`) 命令定义。对于 MD5，我们可以使用简写选项 URL_MD5 `<md5>`。
- `DOWNLOAD_NO_EXTRACT <bool>` – 显式禁用下载后的提取。我们可以通过访问 `<DOWNLOADED_FILE>` 变量，在后续步骤中使用下载文件的文件名。
- `DOWNLOAD_NO_PROGRESS <bool>` – 不记录下载进度。
- `TIMEOUT <seconds>` 和 `INACTIVITY_TIMEOUT <seconds>` – 在固定总时间或无活动期后终止下载的超时时间。
- `HTTP_USERNAME <username>`和 `HTTP_PASSWORD <password>` – 提供 HTTP 认证值的选项。确保在项目中避免硬编码任何凭据。
- `HTTP_HEADER <header1> [<header2>…]` – 发送额外的 HTTP 头。用这个来访问 AWS 中的内容或传递一些自定义令牌。
- `TLS_VERIFY <bool>` – 验证 SSL 证书。如果没有设置，CMake 将从CMAKE_TLS_VERIFY变量中读取这个设置，默认为false。跳过 TLS 验证是一种不安全、糟糕的做法，应该避免，尤其是在生产环境中。
- `TLS_CAINFO <file>` – 如果你的公司发行自签名 SSL 证书，这个选项很有用。这个选项提供了一个权威文件的路径；如果没有指定，CMake 将从CMAKE_TLS_CAINFO变量中读取这个设置。


---

从 Git 下载依赖项

要从 Git 下载依赖项，你需要确保主机安装了 Git 1.6.5 或更高版本

```js
GIT_REPOSITORY <url>
GIT_TAG <tag>
```

`<url>和<tag>`都应该符合git命令能理解的格式。
此外，建议使用特定的 git 哈希，以确保生成的二进制文件可以追溯到特定的提交，并且不会执行不必要的git fetch。如果你坚持使用分支，
使用如origin/main的远程名称。这保证了本地克隆的正确同步。

其他选项如下：
- `GIT_REMOTE_NAME <name>` – 远程名称，默认为origin。
- `GIT_SUBMODULES <module>...` – 指定应该更新的子模块。从 3.16 起，这个值默认为无（之前，所有子模块都被更新）。
- `GIT_SUBMODULES_RECURSE 1` – 启用子模块的递归更新。
- `GIT_SHALLOW 1` – 执行浅克隆（不下载历史提交）。这个选项推荐用于性能。
- `TLS_VERIFY <bool>` – 这个选项在从 URL 下载依赖项部分解释过。它也适用于 Git，并且为了安全起见应该启用。


---

从 Subversion 下载依赖项

要从 Subversion 下载，我们应该指定以下选项：
```js
SVN_REPOSITORY <url>
SVN_REVISION -r<rev>
```

- `SVN_USERNAME <user>` 和 `SVN_PASSWORD <password>` – 用于检出和更新的凭据。像往常一样，避免在项目中硬编码它们。
- `SVN_TRUST_CERT <bool>` – 跳过对 Subversion 服务器证书的验证。只有在你信任网络路径到服务器及其完整性时才使用这个选项。默认是禁用的。

---

从 Mercurial 下载依赖项

```js
HG_REPOSITORY <url>
HG_TAG <tag>
```

---

从 CVS 下载依赖项

```js
CVS_REPOSITORY <cvsroot>
CVS_MODULE <module>
CVS_TAG <tag>
```

---

更新步骤选项

默认情况下，update步骤如果支持更新，将会重新下载外部项目的文件。我们可以用两种方式覆盖这个行为：
- 提供一个自定义命令，在更新期间执行 `UPDATE_COMMAND <cmd>`。
- 完全禁用update步骤（允许在断开网络的情况下构建）– `UPDATE_DISCONNECTED <bool>`。请注意，第一次构建期间的download步骤仍然会发生。

---

修补步骤选项

Patch是一个可选步骤，在源代码获取后执行。要启用它，我们需要指定我们要执行的确切命令：
`PATCH_COMMAND <cmd>...`

---

前面提到的选项列表只包含最常用的条目。确保参考官方文档以获取更多详细信息和描述其他步骤的选项：
cmake.org/cmake/help/latest/module/ExternalProject.html


### 在实际中使用 ExternalProject

依赖项在构建阶段被填充非常重要，它有两个效果——项目的命名空间完全分离，任何外部项目定义的目标在主项目中不可见。
后者尤其痛苦，因为我们在使用find_package()命令后不能以同样的方式使用target_link_libraries()。这是因为两个配置阶段的分离。
主项目必须完成配置阶段并开始构建阶段，然后依赖项才能下载并配置。
这是一个问题，但我们将学习如何处理第二个。
现在，让我们看看ExternalProject_Add()如何与我们在 previous examples 中使用的 yaml-cpp 库工作：

```js
cmake_minimum_required(VERSION 3.20.0)
project(ExternalProjectGit CXX)
add_executable(welcome main.cpp)
configure_file(config.yaml config.yaml COPYONLY)
include(ExternalProject)
ExternalProject_Add(external-yaml-cpp
  GIT_REPOSITORY    https://github.com/jbeder/yaml-cpp.git
  GIT_TAG           yaml-cpp-0.6.3
)
target_link_libraries(welcome PRIVATE yaml-cpp)
```

构建该项目采取以下步骤：
- 我们包含了ExternalProject模块以访问其功能。
- 我们调用了FindExternalProject_Add()命令，该命令将构建阶段任务为下载必要文件，并在我们的系统中配置、构建和安装依赖项。


我们需要小心这里，并理解这个例子之所以能工作，是因为 yaml-cpp 库在其CMakeLists.txt中定义了一个安装阶段。
这个阶段将库文件复制到系统中的标准位置。
target_link_libraries()命令中的yaml-cpp参数被 CMake 解释为直接传递给链接器的参数——-lyaml-cpp。
这个行为与之前的例子不同，在那里我们明确定义了yaml-cpp目标。
如果库不提供安装阶段（或者二进制版本的名字不匹配），链接器将抛出错误。


在此之际，我们应该更深入地探讨每个阶段的配置，并解释如何使用不同的下载方法。
我们将在FetchContent部分讨论这些问题，

但首先，让我们回到讨论ExternalProject导致的依赖项晚获取问题。
我们不能在外部项目被获取的时候使用它们的目标，因为编译阶段已经结束了。
CMake 将通过将其标记为特殊的`UTILITY`类型来显式保护使用FindExternalProject_Add()创建的目标。
当你错误地尝试在主项目中使用这样一个目标（也许是为了链接它）时，CMake 将抛出一个错误：
`Target "external-yaml-cpp-build" of type UTILITY may not be linked into another target.`

为了绕过这个限制，
技术上我们可以创建另一个目标，一个IMPORTED库，然后使用它（就像我们在这个章节前面用FindPQXX.cmake做的那样）。
但这实在太麻烦了。
更糟糕的是，CMake 实际上理解外部 CMake 项目创建的目标（因为它在构建它们）。
在主项目中重复这些声明不会是一个非常 DRY 的做法。

另一个可能的解决方案是将整个依赖项的获取和构建提取到一个独立的子项目中，并在配置阶段构建该子项目。
要实现这一点，我们需要用execute_process()启动 CMake 的另一个实例。
通过一些技巧和add_subdirectory()命令，我们随后可以将这个子项目的列表文件和二进制文件合并到主项目中。
这种方法（有时被称为超级构建）过时且不必要的复杂。
在这里我不详细说明，因为对初学者来说没有太大用处。
如果你好奇，可以阅读 Craig Scott 这篇很好的文章：crascit.com/2015/07/25/cmake-gtest/。

总之，==当项目间存在命名空间冲突时，ExternalProject可以帮我们摆脱困境，但在其他所有情况下，FetchContent都远远优于它==。让我们来找出为什么。


### FetchContent

现在，建议使用FetchContent模块来导入外部项目。
这个模块自 CMake 3.11 版本以来一直可用，但我们建议至少使用 3.14 版本才能有效地与之工作。

本质上，它是一个高级别的ExternalProject包装器，提供类似的功能和更多功能。
关键区别在于执行阶段——与ExternalProject不同，FetchContent在配置阶段填充依赖项，将外部项目声明的所有目标带到主项目的范围内。
这样，我们可以像定义自己的目标一样精确地使用它们。

使用FetchContent模块需要三个步骤：
- 将模块包含在你的项目中，使用include(FetchModule)。
- 使用FetchContent_Declare()命令配置依赖项。
- 使用FetchContent_MakeAvailable()命令填充依赖项——下载、构建、安装，并将其列表文件添加到主项目中并解析。

你可能会问自己为什么Declare和MakeAvailable命令被分开。这是为了在层次化项目中启用配置覆盖。这是一个场景——一个父项目依赖于A和B外部库。A库也依赖于B，但A库的作者仍在使用与父项目不同的旧版本
而且，对MakeAvailable的依赖既不能配置也不能填充依赖，因为要覆盖A库中的版本，父项目将被迫无论在A库中最终是否需要，都要填充依赖。

由于有了单独的配置步骤，我们能够为父项目指定一个版本，并在所有子项目和依赖项中使用它：
```js
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  # release-1.11.0
  GIT_TAG        e2239ee6043f73722e7aa812a459f54a28552929 
)
```
任何后续调用FetchContent_Declare()，以googletest作为第一个参数，都将被忽略，以允许层次结构最高的项目决定如何处理这个依赖。

FetchContent_Declare()命令的签名与ExternalProject_Add()完全相同：
`FetchContent_Declare(<depName> <contentOptions>...)`

这些参数会被 CMake 存储，直到调用FetchContent_MakeAvailable()并且需要填充时才会传递。
然后，内部会将这些参数传递给ExternalProject_Add()命令。
然而，并非所有的选项都是允许的。
我们可以指定download、update或patch步骤的任何选项，但不能是configure、build、install或test步骤。


当配置就绪后，我们会像这样填充依赖项：
`FetchContent_MakeAvailable(<depName>)`

这将下载文件并读取目标到项目中，但在这次调用中实际发生了什么？
FetchContent_MakeAvailable()是在 CMake 3.14 中添加的，以将最常用的场景封装在一个命令中。在图 7.2中，你可以看到这个过程的详细信息：
- 调用FetchContent_GetProperties()，从全局变量将FetchContent_Declare()设置的配置从全局变量传递到局部变量。
- 检查（不区分大小写）是否已经为具有此名称的依赖项进行了填充，以避免重复下载。如果是，就在这里停止。
- 调用==FetchContent_Populate()。它会配置包装的ExternalProject模块==，通过传递我们设置的（但跳过禁用的）选项并下载依赖项。它还会设置一些变量，以防止后续调用重新下载，并将必要的路径传递给下一个命令。
- 最后，add_subdirectory()带着源和构建树作为参数调用，告诉父项目列表文件在哪里以及构建工件应放在哪里。

显然，我们可能遇到两个无关项目声明具有相同名称的目标的情况。
这是一个只能通过回退到ExternalProject或其他方法来解决的问题。

---

```js
cmake_minimum_required(VERSION 3.20.0)
project(ExternalProjectGit CXX)
add_executable(welcome main.cpp)
configure_file(config.yaml config.yaml COPYONLY)
include(FetchContent)
FetchContent_Declare(external-yaml-cpp
  GIT_REPOSITORY    https://github.com/jbeder/yaml-cpp.git
  GIT_TAG           yaml-cpp-0.6.3
)
FetchContent_MakeAvailable(external-yaml-cpp)
target_link_libraries(welcome PRIVATE yaml-cpp)
```
ExternalProject_Add直接被FetchContent_Declare替换，我们还添加了另一个命令——FetchContent_MakeAvailable。
代码的变化微乎其微，但实际的区别却很大！
我们可以明确地访问由 yaml-cpp 库创建的目标。

为了证明这一点，我们将使用CMakePrintHelpers帮助模块，并向之前的文件添加这些行：
```js
include(CMakePrintHelpers)
cmake_print_properties(TARGETS yaml-cpp 
  PROPERTIES TYPE SOURCE_DIR)
```

现在，配置阶段将打印以下输出：
```js
Properties for TARGET yaml-cpp:
   yaml-cpp.TYPE = "STATIC_LIBRARY"
   yaml-cpp.SOURCE_DIR = "/tmp/b/_deps/external-yaml-cpp-src"
```

目标存在；它是一个静态库，其源代码目录位于构建树内部。
使用相同的助手在ExternalProject示例中调试目标简单地返回：
`No such TARGET "yaml-cpp" !`

在配置阶段目标没有被识别。这就是为什么FetchContent要好得多，并且应该尽可能地在任何地方使用






# ch8 测试框架

完成前面的章节后，你已经变成了一个能够使用 CMake 构建各种项目的自给自足的构建工程师。
成为 CMake 专家的最后一个步骤是学习如何引入和自动化各种质量检查，并为协作工作和发布做好准备。
在大型公司内部开发的高质量项目往往共享同样的理念：自动化耗竭心灵能量的重复性任务，以便重要决策得以实施。

有经验的专家知道测试必须自动化。

在本章中，我们将学习测试的重要性以及如何使用与 CMake 捆绑的 CTest 工具来协调测试执行。CTest 能够查询可用的测试、过滤执行、洗牌、重复和限制时间。我们将探讨如何使用这些特性、控制 CTest 的输出以及处理测试失败。
接下来，我们将调整我们项目的结构以支持测试，并创建我们自己的测试运行器。在讨论基本原理之后，我们将继续添加流行的测试框架：Catch2 和 GoogleTest 及其模拟库。最后，我们将介绍使用 LCOV 进行详细测试覆盖率报告。




## 使用 CTest 在 CMake 中标准化测试

CMake 通过引入一个独立的 ctest 命令行工具来解决这个问题(。。标准化)。
它由项目作者通过列表文件进行配置，并为执行测试提供了一个统一的方式：对于使用 CMake 构建的每个项目，都有一个相同的、标准化的接口。
如果你遵循这个约定，你将享受其他好处：将项目添加到（CI/CD）流水线将更容易，在诸如 Visual Studio 或 CLion 等（IDE）中突出显示它们——所有这些事情都将得到简化，更加方便。
更重要的是，你将用非常少的投入获得一个更强大的测试运行工具。

如何在一个已经配置的项目上使用 CTest 执行测试？我们需要选择以下三种操作模式之一：
- 测试
- 构建与测试
- 仪表板客户端

最后一种模式允许您将测试结果发送到一个名为 CDash 的单独工具（也来自 Kitware）
CDash 不在本书的范围内，如果你有兴趣:
- cmake.org/cmake/help/latest/manual/ctest.1.html#dashboard-client
- www.cdash.org/


---

测试模式

`ctest [<options>]`

在这种模式下，应在构建树中执行 CTest，在用cmake构建项目之后。
在开发周期中，这有点繁琐，因为您需要执行多个命令并来回更改工作目录。
为了简化这个过程，CTest 增加了一个第二种模式：build-and-test模式。

---

构建和测试模式

以`--build-and-test` 开始执行ctest

```js
ctest --build-and-test <path-to-source> <path-to-build>
      --build-generator <generator> [<options>...]
      [--build-options <opts>...] 
      [--test-command <command> [<args>...]]
```

这是一个简单的包装器，它围绕常规测试模式接受一些构建配置选项，并允许我们添加第一个模式下的命令。
换句话说，所有可以传递给`ctest <options>`的选项，在传递给`ctest --build-and-test`时也会生效。
这里唯一的要求是在`--test-command`参数之后传递完整的命令。
与您可能认为的相反，除非在`--test-command`后面提供ctest关键字，否则构建和测试模式==实际上不会运行==任何测试，如下所示：
`ctest --build-and-test project/source-tree /tmp/build-tree --build-generator "Unix Makefiles" --test-command ctest`

在这个命令中，我们需要指定源和构建路径，并选择一个构建生成器。
这三个都是必需的，并且遵循cmake命令的规则


您可以传递额外的参数给这个模式。它们分为三组，分别控制配置、构建过程或测试。

以下是控制配置阶段的参数：
- `--build-options`—任何额外的cmake配置（不是构建工具）选项应紧接在--test-command之前，这是最后一个参数。
- `--build-two-config`—为 CMake 运行两次配置阶段。
- `--build-nocmake`—跳过配置阶段。
- `--build-generator-platform, --build-generator-toolset`—提供生成器特定的平台和工具集。
- `--build-makeprogram`—在使用 Make 或 Ninja 生成器时指定make可执行文件。

以下是控制构建阶段的参数：
- `--build-target`—构建指定的目标（而不是all目标）。
- `--build-noclean`—在不首先构建clean目标的情况下进行构建。
- `--build-project`—提供构建项目的名称。

这是用于控制测试阶段的参数：
- `--test-timeout`—限制测试的执行时间（以秒为单位）。

剩下的就是在`--test-command ctest`参数之后配置常规测试模式。


---

测试模式

假设我们已经构建了我们的项目，并且我们在构建树中执行ctest（或者我们使用build-and-test包装器），我们最终可以执行我们的测试。

在没有任何参数的情况下，一个简单的ctest命令通常足以在大多数场景中获得满意的结果。
如果所有测试都通过，ctest将返回一个0的退出码。您可以在 CI/CD 管道中使用此命令，以防止有错误的提交合并到您仓库的生产分支。

CTest 允许你影响测试选择、它们的顺序、产生的输出、时间限制、重复等等

---

查询测试

我们可能需要做的第一件事就是理解哪些测试实际上是为本项目编写的。
CTest 提供了一个`-N`选项，它禁用执行，只打印列表

```shell
# ctest -N
Test project /tmp/b
  Test #1: SumAddsTwoInts
  Test #2: MultiplyMultipliesTwoInts
Total Tests: 2
```

你可能想用下一节中描述的筛选器与`-N`一起使用，以检查当应用筛选器时会执行哪些测试。

如果你需要一个可以被自动化工具消费的 JSON 格式，请用`--show-only=json-v1`执行ctest。

CTest 还提供了一个用LABELS关键字来分组测试的机制。
要列出所有可用的标签（而不实际执行任何测试），请使用`--print-labels`。
这个选项在测试用手动定义时很有帮助，例如在你的列表文件中使用`add_test(<name> <test-command>)`命令，因为你可以通过测试属性指定个别标签，像这样：
`set_tests_properties(<name> PROPERTIES LABELS "<label>")`


---

过滤测试

根据提供的`<r>` 正则表达式（regex）过滤测试，如下所示：
- `-R <r>`, `--tests-regex <r>`—只运行名称匹配`<r>`的测试
- `-E <r>`, `--exclude-regex <r>`—跳过名称匹配`<r>`的测试
- `-L <r>`, `--label-regex <r>`—只运行标签匹配`<r>`的测试
- `-LE <r>`, `--label-exclude <正则表达式>`—跳过标签匹配`<r>`的测试

使用`--tests-information`选项（或更短的形式，`-I`）可以实现高级场景。
用这个筛选器提供一个逗号分隔的范围内的值：`<开始>, <结束>, <步长>`。
任意字段都可以为空，再有一个逗号之后，你可以附加个别`<测试 ID>`值来运行它们。以下是几个例子：

- `-I 3,,` 将跳过 1 和 2 个测试（执行从第三个测试开始）
- `-I ,2,` 只运行第一和第二个测试
- `-I 2,,3` 将从第二行开始运行每个第三测试
- `-I ,0,,3,9,7` 将只运行第三、第九和第七个测试

默认情况下，与`-R`一起使用的`-I`选项将缩小执行范围（仅运行同时满足两个要求的测试）。
如果您需要两个要求的并集来执行（任一要求即可），请添加`-U`选项。

如前所述，您可以使用-N选项来检查过滤结果。


---

洗牌测试

编写单元测试可能很棘手。
遇到的一个更令人惊讶的问题就是测试耦合，这是一种情况，其中一个测试通过不完全设置或清除 SUT 的状态来影响另一个测试。

发现此类问题的一种方法是单独运行每个测试。
通常，当我们直接从测试框架中执行测试运行器而不使用 CTest 时，并非如此。
要运行单个测试，您需要向测试可执行文件传递框架特定的参数。
这允许您检测在测试套件中通过但在单独执行时失败的测试。

另一方面，CTest 有效地消除了所有基于内存的测试交叉污染，通过隐式执行子 CTest 实例中的每个测试用例。
您甚至可以更进一步，添加`--force-new-ctest-process`选项以强制使用单独的进程。

不幸的是，仅凭这一点还不足以应对测试使用的外部、争用资源，如 GPU、数据库或文件。
我们可以采取的额外预防措施之一是简单地随机化测试执行顺序。
这种干扰通常足以最终检测到这种虚假通过的测试。
CTest 支持这种策略，通过`--schedule-random`选项。


---

处理失败

Fail early, fail often, but always fail forward

除非你在运行测试时附带了调试器，否则很难了解到你在哪里出了错，因为 CTest 会保持简洁，只列出失败的测试，而不实际打印它们的输出。

测试案例或 SUT 打印到stdout的信息可能对确定具体出了什么问题非常有价值。
为了看到这些信息，我们可以使用`--output-on-failure`运行ctest。
另外，设置`CTEST_OUTPUT_ON_FAILURE`环境变量也会有相同的效果。

根据解决方案的大小，在任何一个测试失败后停止执行可能是有意义的。
这可以通过向ctest提供`--stop-on-failure`参数来实现。


CTest 存储了失败测试的名称。
为了在漫长的测试套件中节省时间，我们可以关注这些失败的测试，并在解决问题前跳过运行通过的测试。
这个特性可以通过使用`--rerun-failed`选项来实现（忽略其他任何过滤器）
记得在解决问题后运行所有测试，以确保在此期间没有引入回归。


当 CTest 没有检测到任何测试时，这可能意味着两件事：要么是测试不存在，要么是项目有问题。
默认情况下，ctest会打印一条警告信息并返回一个0退出码，以避免混淆。
大多数用户会有足够的上下文来理解他们遇到了哪种情况以及接下来应该做什么。
然而，在某些环境中，ctest总是会执行，作为自动化流水线的一部分。
那么，我们可能需要明确表示，测试的缺失应该被解释为错误（并返回非零退出码）。
我们可以通过提供`--no-tests=error`参数来配置这种行为。
要实现相反的行为（不警告），请使用`--no-tests=ignore`选项。


---

重复执行测试

测试会因为环境原因而失败：由于错误地模拟了时间、事件循环问题、异步执行处理不当、并发性、散列冲突，以及其他在每次运行时都不会发生的非常复杂的情况。
这些不可靠的测试被称为“flaky tests”。


主要有三个担忧，如下所述：
- 如果你在你的代码库中收集了足够的不稳定测试，它们将成为代码变更顺利交付的一个严重障碍。尤其是当你急于回家（比如周五下午）或交付一个严重影响客户问题的紧急修复时，这种情况尤其令人沮丧。
- 你无法真正确信你的不稳定测试之所以失败是因为测试环境的不足。可能正好相反：它们失败是因为它们复现了一个在生产环境中已经发生的罕见场景。只是还没有足够明显地发出警报… 而已。
- 不是测试本身具有不稳定性——是你的代码有问题！环境有时确实会出问题——作为程序员，我们以确定性的方式处理这些问题。如果 SUT 以这种方式运行，这是一个严重错误的迹象——例如，代码可能正在读取未初始化的内存。


没有一种完美的方式来解决所有上述情况——可能的原因太多。然而，我们可以通过使用`–repeat <mode>:<#>`选项来重复运行测试，从而增加我们识别不稳定测试的机会。以下是三种可供选择的模式：
- `until-fail`—运行测试`<#>`次；所有运行都必须通过。
- `until-pass`—运行测试`<#>`次；至少要通过一次。当处理已知具有不稳定性的测试时，这个方法很有用，但这些测试太难且重要，无法进行调试或禁用。
- `after-timeout`—运行测试`<#>`次，但只有在测试超时的情况下才重试。在繁忙的测试环境中使用它。



---

控制输出

每次都将所有信息打印到屏幕上会立即变得非常繁忙。
Ctest 减少了噪音，并将它执行的测试的输出收集到日志文件中，在常规运行中只提供最有用的信息。
当事情变坏，测试失败时，如果你启用了`--output-on-failure`（如前面所述），你可以期待一个摘要，可能还有一些日志。

为了获取更详细的输出，可以添加`-V`选项（或者如果你想在自动化管道中明确表示，可以使用`--verbose`）。
如果这还不够，你可能想要`-VV`或`--extra-verbose`。
对于非常深入的调试，可以使用`--debug`（但要做好准备，因为会有很多文本细节）。

CTest 还提供了通过`-Q`启用的“禅模式”，或`--quiet`。那时将不会打印任何输出
输出仍然会存储在测试文件中（默认在`./Testing/Temporary`中）
自动化管道可以通过检查退出代码是否非零值，并在不向开发者输出可能混淆的详细信息的情况下，收集日志文件进行进一步处理。

要将在特定路径存储日志，请使用`-O <文件>`、`--output-log <文件>`选项。
如果您苦于输出过长，有两个限制选项可以将它们限制为每个测试给定的字节数：`--test-output-size-passed <大小>`和`--test-output-size-failed <大小>`。

---

杂项

还有一些其他的有用选项，可以满足你日常测试需求，如下所述：
- `-C <配置>, --build-config <配置>`（仅限多配置生成器）—使用此选项指定要测试的配置。Debug配置通常包含调试符号，使事情更容易理解，但Release也应该测试，因为强烈的优化选项可能会潜在地影响 SUT 的行为。
- `-j <作业数>, --parallel <作业数>`—这设置了并行执行的测试数量。在开发过程中，它非常有用，可以加快长测试的执行。请注意，在一个繁忙的环境中（在共享的测试运行器上），它可能会因调度而产生不利影响。这可以通过下一个选项稍微缓解。
- `--test-load <级别>`—使用此选项以一种方式安排并行测试，使 CPU 负载不超过`<级别>`值（尽最大努力）。
- `--timeout <秒>`—使用此选项指定单个测试的默认时间限制。





## 为 CTest 创建最基本的单元测试

技术上讲，编写单元测试可以在没有任何框架的情况下进行。
我们只需要做的是创建一个我们想要测试的类的实例，执行其一种方法，并检查返回的新状态或值是否符合我们的期望。
然后，我们报告结果并删除被测试对象。让我们试一试。

工程结构
```js
- CMakeLists.txt
- src
  |- CMakeLists.txt
  |- calc.cpp
  |- calc.h
  |- main.cpp
- test
  |- CMakeLists.txt
  |- calc_test.cpp
  |- unit_tests.cpp
```

```C++
// main.cpp
#include <iostream>
#include "calc.h"
using namespace std;
int main() {
  Calc c;
  cout << "2 + 2 = " << c.Sum(2, 2) << endl;
  cout << "3 * 3 = " << c.Multiply(3, 3) << endl;
}
```

```C++
// calc.h
#pragma once
class Calc {
 public:
   int Sum(int a, int b);
   int Multiply(int a, int b);
};
```

```C++
// calc.cpp
#include "calc.h"
int Calc::Sum(int a, int b) {
  return a + b;
}
int Calc::Multiply(int a, int b) {
  return a * a; // a mistake!
}
```

```C++
// calc_test.cpp
#include "calc.h"
#include <cstdlib>
void SumAddsTwoIntegers() {
  Calc sut;
  if (4 != sut.Sum(2, 2))
    std::exit(1);
}
void MultiplyMultipliesTwoIntegers() {
  Calc sut;
  if(3 != sut.Multiply(1, 3))
    std::exit(1);
}
```

如果从调用方法返回的值与期望不符，每个函数都将调用`std::exit(1)`。
我们本可以使用`assert()、abort()或terminate()`，但那样的话，在ctest的输出中，我们将得到一个更难读的Subprocess aborted消息，而不是更易读的Failed消息。


```C++
// 测试运行器  unit_tests.cpp
#include <string>
void SumAddsTwoIntegers();
void MultiplyMultipliesTwoIntegers();
int main(int argc, char *argv[]) {
  if (argc < 2 || argv[1] == std::string("1"))
    SumAddsTwoIntegers();
  if (argc < 2 || argv[1] == std::string("2"))
    MultiplyMultipliesTwoIntegers();
}
```


下面是这里发生的事情的分解：
- 我们声明了两个外部函数，它们将从另一个翻译单元链接过来。
- 如果没有提供任何参数，执行两个测试（argv[]中的零元素总是程序名）。
- 如果第一个参数是测试的标识符，执行它。
- 如果有任何测试失败，它内部调用exit()并返回1退出码。
- 如果没有执行任何测试或所有测试都通过，它隐式地返回0退出码。

要运行第一个测试，我们将==执行==`./unit_tests 1`；要运行第二个，我们将执行`./unit_tests 2`

---

让我们看看我们如何使用它与 CTest结合

```js
// root/CMakeLists.txt
cmake_minimum_required(VERSION 3.20.0)
project(NoFrameworkTests CXX)
enable_testing()
add_subdirectory(src bin)
add_subdirectory(test)
```

enable_testing() 告诉 CTest 我们想在当前目录及其子目录中启用测试。
接下来，我们在每个子目录中包含两个嵌套的列表文件：src和test。
bin表示我们希望src子目录的二进制输出放在`<build_tree>/bin`中。
否则，二进制文件将出现在`<build_tree>/src`中，这可能会引起混淆。毕竟，构建工件不再是源文件。

src目录的列表文件非常直接，包含一个简单的main目标定义，如下所示：
```js
// root/src/CMakeLists.txt
add_executable(main main.cpp calc.cpp)
```

test目录编写一个列表文件
```js
add_executable(unit_tests
               unit_tests.cpp
               calc_test.cpp
               ../src/calc.cpp)
target_include_directories(unit_tests PRIVATE ../src)
add_test(NAME SumAddsTwoInts COMMAND unit_tests 1)
add_test(NAME MultiplyMultipliesTwoInts COMMAND unit_tests 2)
```

我们现在定义了第二个unit_tests目标，它也使用src/calc.cpp实现文件和相应的头文件。
最后，我们明确添加了两个测试：SumAddsTwoInts和MultiplyMultipliesTwoInts。
每个都将其 ID 作为add_test()命令的参数。
CTest将简单地取COMMAND关键字之后提供的一切，并在子壳中执行它，收集输出和退出代码。
不要对add_test()过于依赖——在单元测试框架部分，我们将发现处理测试用例的更好方法，所以我们在这里不详细描述它。


输出
```text
# ctest
Test project /tmp/b
    Start 1: SumAddsTwoInts
1/2 Test #1: SumAddsTwoInts ...................   Passed    0.00 sec
    Start 2: MultiplyMultipliesTwoInts
2/2 Test #2: MultiplyMultipliesTwoInts ........***Failed    0.00 sec
50% tests passed, 1 tests failed out of 2
Total Test time (real) =   0.00 sec
The following tests FAILED:
          2 - MultiplyMultipliesTwoInts (Failed)
Errors while running CTest
Output from these tests are in: /tmp/b/Testing/Temporary/LastTest.log
Use "--rerun-failed --output-on-failure" to re-run the failed cases verbosely.
```


---

### 为测试搭建项目结构

在我们采用单元测试框架之前，我们需要重新思考项目的结构。

我们如何避免重复编译，并在测试和生产之间重用工件？

最优雅的方法是将整个解决方案构建为一个库，并与单元测试链接。
你可能会问：“我们怎么运行它呢？”我们需要一个引导可执行文件，它将链接库并运行其代码。

首先，将您当前的main()函数重命名为run()、start_program()或类似名称。
然后，创建另一个实现文件（bootstrap.cpp），其中包含一个新的main()函数，仅此而已。

这将成为我们的适配器（或者说是包装器）：它的唯一作用是提供一个入口点并调用run()转发命令行参数（如果有）。
剩下的就是将所有内容链接在一起，这样我们就有了一个可测试的项目。

通过重命名main()，我们现在可以链接被测试的系统（SUT）和测试，并且还能测试它的主要功能。
否则，我们就违反了main()函数。正如第六章“为测试分离 main()”部分所承诺的，我们将详细解释这个主题。

测试框架可能提供自己的main()函数实现，所以我们不需要编写。通常，它会检测我们链接的所有测试，并根据所需配置执行它们。

这种方法产生的工件可以分为以下目标：
- 带有生产代码的sut库
- bootstrap带有main()包装器，调用sut中的run()
- 带有main()包装器，运行所有sut测试的单元测试

![67738b296cc18ac8a277b74d641369c3.png](../_resources/67738b296cc18ac8a277b74d641369c3.png)

我们最终会得到六个实现文件，它们将生成各自的（.o）目标文件，如下所示：
- calc.cpp—要进行单元测试的Calc类。这被称为被测试单元（UUT），因为 UUT 是 SUT 的一个特化。
- run.cpp—原始入口点重命名为run()，现在可以进行测试。
- bootstrap.cpp—新的main()入口点调用run()。
- calc_test.cpp—测试Calc类。
- run_test.cpp—新增run()的测试可以放在这里。
- unit_tests.o—单元测试的入口点，扩展为调用run()的测试。


main.cpp 改名为 run.cpp， 且 里面的 main方法改名为 run方法， 添加return 0;
```C++
#include <iostream>
#include "calc.h"
using namespace std;
int run() {
  Calc c;
  cout << "2 + 2 = " << c.Sum(2, 2) << endl;
  cout << "3 * 3 = " << c.Multiply(3, 3) << endl;
  return 0;
}
```

bootstrap.cpp 中 main方法
```C++
int run(); // declaration
int main() {
  run();
}
```

src/CMakeLists.txt
```C++
add_library(sut STATIC calc.cpp run.cpp)
target_include_directories(sut PUBLIC .)
add_executable(bootstrap bootstrap.cpp)
target_link_libraries(bootstrap PRIVATE sut)
```

首先，我们创建了一个sut库，并将.标记为PUBLIC 包含目录，以便将其传播到所有将链接sut的目标（即bootstrap和unit_tests）。
请注意，包含目录是相对于列表文件的，因此我们可以使用点（.）来引用当前的`<source_tree>/src`目录。

是时候更新我们的unit_tests目标了。在这里，我们将移除对`../src/calc.cpp`文件的直接引用，改为sut的链接引用作为unit_tests目标。
我们还将为run_test.cpp文件中的主函数添加一个新测试。
为了简洁起见，我们将跳过讨论那个部分，但如果您感兴趣，可以查看在线示例。同时，这是整个test列表文件：
```js
// test/CMakeLists.txt
add_executable(unit_tests
               unit_tests.cpp
               calc_test.cpp
               run_test.cpp)
target_link_libraries(unit_tests PRIVATE sut)
```
注册新的测试
```js
add_test(NAME SumAddsTwoInts COMMAND unit_tests 1)
add_test(NAME MultiplyMultipliesTwoInts COMMAND unit_tests 2)
add_test(NAME RunOutputsCorrectEquations COMMAND unit_tests 3)
```

## 单元测试框架

根据所选框架的规则在实现文件中编写测试，并将这些测试与框架提供的测试运行器链接起来。
测试运行器是您的入口点，将启动所选测试的执行。
与我们在本章早些时候看到的基本的unit_tests.cpp文件不同，许多它们将自动检测所有测试


本章我决定介绍两个单元测试框架。我选择它们的原因如下：
- Catch2 是一个相对容易学习、得到良好支持和文档的项目。它提供了简单的测试用例，但同时也提供了用于行为驱动开发（BDD）的优雅宏。它缺少一些功能，但在需要时可以与外部工具配合使用。您可以在这里访问其主页：github.com/catchorg/Catch2。
- GTest 也非常方便，但功能更加强大。它的关键特性是一组丰富的断言、用户定义的断言、死亡测试、致命和非致命失败、值和类型参数化测试、XML 测试报告生成以及模拟。最后一个是通过从同一存储库中可用的 GMock 模块提供的： github.com/google/googletest。

您应该选择哪个框架取决于您的学习方法和项目大小。
如果您喜欢缓慢、逐步的过程，并且不需要所有的花哨功能，那么选择 Catch2。
那些喜欢从深层次开始并需要大量火力支持的开发人员将受益于选择 GTest。



### Catch2

首先，让我们简要地看看我们可以为我们的Calc类编写单元测试的实现

```C++
// /test/calc_test.cpp
#include <catch2/catch_test_macros.hpp>
#include "calc.h"
TEST_CASE("SumAddsTwoInts", "[calc]") {
  Calc sut;
  CHECK(4 == sut.Sum(2, 2));
}
TEST_CASE("MultiplyMultipliesTwoInts", "[calc]") {
  Calc sut;
  CHECK(12 == sut.Multiply(3, 4));
}
```

CHECK()宏不仅验证期望是否满足——它们还会收集所有失败的断言，并在单个输出中呈现它们，这样你就可以进行一次修复，避免重复编译。

如果允许的话，Catch2 会自动将你的测试注册到 CTest。在上一节中描述的配置项目后，添加框架非常简单。我们需要使用FetchContent()将其引入项目。

有两个主要版本可供选择：v2和v3。版本 2 作为一个单头库（只需`#include <catch2/catch.hpp>`）提供给 C++11，最终将被版本 3 所取代。
这个版本由多个头文件组成，被编译为静态库，并要求 C++14。
当然，如果你能使用现代 C++（是的，C++11 不再被认为是“现代”的），那么推荐使用更新的版本。
在与 Catch2 合作时，你应该选择一个 Git 标签并在你的列表文件中固定它。
换句话说，不能保证升级不会破坏你的代码（升级很可能不会破坏代码，但如果你不需要，不要使用devel分支）。
要获取 Catch2，我们需要提供一个仓库的 URL
```js
// /test/CMakeLists.txt
include(FetchContent)
FetchContent_Declare(
  Catch2
  GIT_REPOSITORY https://github.com/catchorg/Catch2.git
  GIT_TAG        v3.0.0
)
FetchContent_MakeAvailable(Catch2)
```

然后，我们需要定义我们的unit_tests目标，并将其与sut以及一个框架提供的入口点和Catch2::Catch2WithMain库链接。
由于 Catch2 提供了自己的main()函数，我们不再使用unit_tests.cpp文件（这个文件可以删除）

```js
// /test/CMakeLists.txt 续
add_executable(unit_tests 
               calc_test.cpp 
               run_test.cpp)
target_link_libraries(unit_tests PRIVATE 
                      sut Catch2::Catch2WithMain)
```

最后，我们使用由 Catch2 提供的模块中定义的catch_discover_tests()命令，该命令将检测unit_tests中的所有测试用例并将它们注册到 CTest

```js
// /test/CMakeLists.txt 续
list(APPEND CMAKE_MODULE_PATH ${catch2_SOURCE_DIR}/extras)
include(Catch)
catch_discover_tests(unit_tests)
```

输出
```js
# ./test/unit_tests
unit_tests is a Catch v3.0.0 host application.
Run with -? for options
--------------------------------------------------------------
MultiplyMultipliesTwoInts
--------------------------------------------------------------
examples/chapter08/03-catch2/test/calc_test.cpp:9
..............................................................
examples/chapter08/03-catch2/test/calc_test.cpp:11: FAILED:
  CHECK( 12 == sut.Multiply(3, 4) )
with expansion:
  12 == 9
==============================================================
test cases: 3 | 2 passed | 1 failed
assertions: 3 | 2 passed | 1 failed
```

直接执行运行器（编译的unit_test可执行文件）稍微快一点，但通常，你希望使用`ctest --output-on-failure`命令，而不是直接执行测试运行器，以获得前面提到的所有 CTest 好处。
注意 Catch2 能够方便地将sut.Multiply(3, 4)表达式扩展为9，为我们提供更多上下文。

这个框架包含了一些有趣的小技巧：事件监听器、数据生成器和微基准测试，但它并不提供模拟功能。
如果你不知道什么是模拟，继续阅读——我们马上就会涉及到这一点。
然而，如果你发现自己需要模拟，你总是可以在这里列出的一些模拟框架旁边添加 Catch2：
- FakeIt (github.com/eranpeer/FakeIt)
- Hippomocks (github.com/dascandy/hippomocks)
- Trompeloeil (github.com/rollbear/trompeloeil)


### GTest

使用 GTest 有几个重要的优点：它已经存在很长时间，并且在 C++社区中高度认可（因此，多个 IDE 支持它）。
背后最大的搜索引擎公司的维护和广泛使用，所以它很可能在不久的将来变得过时或被遗弃。
它可以测试 C++11 及以上版本，所以如果你被困在一个稍微老一点的环境中，你很幸运。

GTest 仓库包括两个项目：GTest（主测试框架）和 GMock（一个添加模拟功能的库）。
这意味着你可以用一个FetchContent()调用来下载它们。

---

使用 GTest

要使用 GTest，我们的项目需要遵循为测试结构化项目部分的方向。这就是我们在这个框架中编写单元测试的方法：

```C++
// /test/calc_test.cpp
#include <gtest/gtest.h>
#include "calc.h"
class CalcTestSuite : public ::testing::Test {
 protected:
  Calc sut_;
};
TEST_F(CalcTestSuite, SumAddsTwoInts) {
  EXPECT_EQ(4, sut_.Sum(2, 2));
}
TEST_F(CalcTestSuite, MultiplyMultipliesTwoInts) {
  EXPECT_EQ(12, sut_.Multiply(3, 4));
}
```

由于这个例子也将用于 GMock，我决定将测试放在一个CalcTestSuite类中。
测试套件是相关测试的组，因此它们可以重用相同的字段、方法和设置（初始化）以及清理步骤。
要==创建一个测试套件，我们需要==声明一个新的类，从::testing::Test继承，并将重用元素（字段、方法）放在其protected部分。

测试套件中的每个测试用例都是用`TEST_F()`预处理器宏声明的，该宏将测试套件和测试用例提供的名称字符串化（还有一个简单的TEST()宏，定义不相关的测试）。
因为我们已经在类中定义了`Calc sut_`，每个测试用例可以像CalcTestSuite的一个方法一样访问它。
实际上，每个测试用例在其自己的类中隐式继承自CalcTestSuite运行（这就是我们需要protected关键字的原因）。请注意，重用字段==不是==为了在连续测试之间共享任何数据——它们的目的是保持代码DRY。

GTest 没有提供像 Catch2 那样的自然断言语法。
相反，我们需要使用一个显式的比较，比如EXPECT_EQ()。
按照惯例，我们将期望值作为第一个参数，实际值作为第二个参数。
还有许多其他断言、助手和宏值得学习。

与 Catch2 不同，GTest 倾向于采用“现场开发”的理念（起源于 GTest 所依赖的 Abseil 项目）。
它指出：“如果你从源代码构建我们的依赖项并遵循我们的 API，你不会遇到任何问题。”

如果你习惯于遵循这个规则（并且从源代码构建没有问题），将你的 Git 标签设置为master分支。否则，从 GTest 仓库中选择一个版本。
我们还可以选择首先在宿主机器上搜索已安装的副本，因为 CMake 提供了一个捆绑的FindGTest模块来查找本地安装。
自 v3.20 起，CMake 将使用上游的GTestConfig.cmake配置文件（如果存在），而不是依赖于可能过时的查找模块。


```js
// /test/CMakeLists.txt
include(FetchContent)
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG master
)
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)
```
我们遵循与 Catch2 相同的方法——执行FetchContent()并从源代码构建框架。
唯一的区别是在 GTest 作者建议的set(gtest...)命令，以防止在 Windows 上覆盖父项目的编译器和链接器设置。

最后，我们可以声明我们的测试运行器可执行文件，链接gtest_main，并借助内置的 CMake GoogleTest模块自动发现我们的测试用例，如下所示：
```js
// /test/CMakeLists.txt 续
add_executable(unit_tests
               calc_test.cpp
               run_test.cpp)
target_link_libraries(unit_tests PRIVATE sut gtest_main)
include(GoogleTest)
gtest_discover_tests(unit_tests)
```

测试运行器的输出比 Catch2 更详细，但我们可以传递--gtest_brief=1来限制它仅显示失败
```js
# ./test/unit_tests --gtest_brief=1
~/examples/chapter08/04-gtest/test/calc_test.cpp:15: Failure
Expected equality of these values:
  12
  sut_.Multiply(3, 4)
    Which is: 9
[  FAILED  ] CalcTestSuite.MultiplyMultipliesTwoInts (0 ms)
[==========] 3 tests from 2 test suites ran. (0 ms total)
[  PASSED  ] 2 tests.
```


---

### GMock

为了测试目的而解耦单元，开发人员使用测试替身或类的特殊版本，这些类被测试类使用。一些例子包括伪造品、存根和模拟。
以下是这些的一些大致定义：
- 伪造品 是某些更复杂类的有限实现。一个例子可能是在实际数据库客户端之内的内存映射。
- 存根 为方法调用提供特定的、预先录制的回答，限于测试中使用的回答。它还可以记录调用了哪些方法以及发生了多少次。
- 模拟 是存根的一个更扩展版本。它还将验证测试期间方法是否如预期地被调用。

---

例子

为Calc类添加一个功能，它将提供一个随机数添加到提供的参数。它将通过一个AddRandomNumber()方法表示，该方法返回这个和作为一个int。

```C++
// /src/rng.h
#pragma once
class RandomNumberGenerator {
 public:
  virtual int Get() = 0;
  virtual ~RandomNumberGenerator() = default;
};
```
注意virtual关键字——除非我们希望涉及更复杂的基于模板的模拟，否则所有要模拟的方法都必须有它，除非我们希望涉及更复杂的基于模板的模拟。
我们还需要记得添加一个虚拟析构函数

```C++
// /src/calc.h
#pragma once
#include "rng.h"
class Calc {
  RandomNumberGenerator* rng_;
 public:
   Calc(RandomNumberGenerator* rng);
   int Sum(int a, int b);
   int Multiply(int a, int b);
   int AddRandomNumber(int a);
};
```

```C++
// /src/calc.cpp
#include "calc.h"
Calc::Calc(RandomNumberGenerator* rng) {
  rng_ = rng;
}
int Calc::Sum(int a, int b) {
  return a + b;
}
int Calc::Multiply(int a, int b) {
  return a * b; // now corrected
}
int Calc::AddRandomNumber(int a) {
  return a + rng_->Get();
}
```

```C++
// /src/rng_mt19937.cpp
#include <random>
#include "rng_mt19937.h"
int RandomNumberGeneratorMt19937::Get() {
  std::random_device rd;
  std::mt19937 gen(rd());
  std::uniform_int_distribution<> distrib(1, 6);
  return distrib(gen);
}
```

```C++
// /stc/rng_mt19937.h
#include "rng.h"
class RandomNumberGeneratorMt19937
      : public RandomNumberGenerator {
 public:
  int Get() override;
};
```

```C++
// /src/run.cpp
#include <iostream>
#include "calc.h"
#include "rng_mt19937.h"
using namespace std;
int run() {
  auto rng = new RandomNumberGeneratorMt19937();
  Calc c(rng);
  cout << "Random dice throw + 1 = " 
       << c.AddRandomNumber(1) << endl;
  delete rng;
  return 0; 
}
```


```C++
// /test/mock/rng_mock.h
#pragma once
#include "gmock/gmock.h"
class RandomNumberGeneratorMock : public
 RandomNumberGenerator {
 public:
  MOCK_METHOD(int, Get, (), (override));
};
```


```C++
// /test/calc_test.cpp
#include <gtest/gtest.h>
#include "calc.h"
#include "mocks/rng_mock.h"
using namespace ::testing;
class CalcTestSuite : public Test {
 protected:
  RandomNumberGeneratorMock rng_mock_;
  Calc sut_{&rng_mock_};
};
TEST_F(CalcTestSuite, AddRandomNumberAddsThree) {
  EXPECT_CALL(rng_mock_,
Get()).Times(1).WillOnce(Return(3));
  EXPECT_EQ(4, sut_.AddRandomNumber(1));
}
```

我们在测试套件中添加了新的头文件并为rng_mock_创建了一个新字段。
接下来，将模拟的地址传递给sut_的构造函数。
我们可以这样做，因为字段是按声明顺序初始化的（rng_mock_先于sut_）。

在我们的测试用例中，
我们对rng_mock_的Get()方法调用 GMock 的EXPECT_CALL宏。
这告诉框架，如果在执行过程中没有调用Get()方法，则测试失败。
Times链式调用明确指出，为了测试通过，必须发生多少次调用。
WillOnce确定在方法调用后，模拟框架做什么（它返回3）。



```js
// /test/CMakeLists.txt
include(FetchContent)
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG release-1.11.0
)
# For Windows: Prevent overriding the parent project's
  compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)
add_executable(unit_tests
               calc_test.cpp
               run_test.cpp)
target_link_libraries(unit_tests PRIVATE sut gtest_main
gmock)
include(GoogleTest)
gtest_discover_tests(unit_tests)
```





## 生成测试覆盖率报告

生成类似报告有多种方法，它们在平台和编译器之间有所不同，但它们通常遵循相同的程序：准备要测量的 SUT，获取基线，测量和报告。

执行这项工作的最简单工具名叫gcov，它是gcov的一个覆盖率工具，用于测量覆盖率。
如果你在使用 Clang，不用担心——Clang 支持生成这种格式的指标。你可以从由Linux 测试项目维护的官方仓库获取 `LCOV`(github.com/linux-test-project/lcov)，或者简单地使用包管理器。
正如其名，它是一个面向 Linux 的工具。
虽然可以在 macOS 上运行它，但不支持 Windows 平台。
最终用户通常不关心测试覆盖率，所以通常可以手动在自建的构建环境中安装 LCOV，而不是将其绑定到项目中。

为了测量覆盖率，我们需要做以下工作：
- 以 `Debug`配置编译 ，使用编译器标志启用代码覆盖。这将生成覆盖注释（.gcno）文件。
- 将测试可执行文件与gcov库链接。
- 在不运行任何测试的情况下收集基线覆盖率指标。
- 运行测试。这将创建覆盖数据（.gcda）文件。
- 将指标收集到聚合信息文件中。
- 生成一个（.html）报告。

首先解释为什么代码必须以Debug配置编译。最重要的原因是，通常Debug配置使用-O0标志禁用了任何优化。
为了防止任何内联和其他类型的隐式代码简化。否则，将很难追踪哪一行机器指令来自哪一行源代码。

两个主要的编译器—GCC 和 Clang—提供相同的`--coverage`标志，以启用覆盖率，生成 GCC 兼容的gcov格式的数据。

```js
// /src/CMakeLists.txt
add_library(sut STATIC calc.cpp run.cpp rng_mt19937.cpp)
target_include_directories(sut PUBLIC .)
if (CMAKE_BUILD_TYPE STREQUAL Debug)
  target_compile_options(sut PRIVATE --coverage)
  target_link_options(sut PUBLIC --coverage)
  add_custom_command(TARGET sut PRE_BUILD COMMAND
                     find ${CMAKE_BINARY_DIR} -type f
                     -name '*.gcda' -exec rm {} +)
endif()
add_executable(bootstrap bootstrap.cpp)
target_link_libraries(bootstrap PRIVATE sut)
```

让我们逐步分解，如下所述：
- 确保我们正在使用if(STREQUAL)命令以Debug配置运行。记住，除非你使用`-DCMAKE_BUILD_TYPE=Debug`选项运行cmake，否则你无法获得任何覆盖率。
- 为sut库的所有object files的PRIVATE编译选项添加`--coverage`。
- 为PUBLIC链接器选项添加`--coverage`： both GCC 和 Clang 将此解释为请求与所有依赖于sut的目标链接gcov（或兼容）库（由于传播属性）。
- add_custom_command()命令被引入以清除任何陈旧的.gcda文件。讨论添加此命令的原因在避免 SEGFAULT 陷阱部分中有详细说明。


要获取报告，我们需要自己使用 LCOV 生成它们。

为此目的，最好定义一个名为coverage的新目标。为了保持整洁，我们将在另一个文件中定义一个单独的函数AddCoverage，用于在test列表文件中使用，如下所示：

```js
function(AddCoverage target)
  find_program(LCOV_PATH lcov REQUIRED)
  find_program(GENHTML_PATH genhtml REQUIRED)
  add_custom_target(coverage
    COMMENT "Running coverage for ${target}..."
    COMMAND ${LCOV_PATH} -d . --zerocounters
    COMMAND $<TARGET_FILE:${target}>
    COMMAND ${LCOV_PATH} -d . --capture -o coverage.info
    COMMAND ${LCOV_PATH} -r coverage.info '/usr/include/*' 
                         -o filtered.info
    COMMAND ${GENHTML_PATH} -o coverage filtered.info 
      --legend
    COMMAND rm -rf coverage.info filtered.info
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
  )
endfunction()
```

我们首先检测lcov和genhtml（来自 LCOV 包的两个命令行工具）的路径。REQUIRED关键字指示 CMake 在找不到它们时抛出错误。接下来，我们按照以下步骤添加一个自定义的coverage目标：
- 清除之前运行的任何计数器。
- 运行target可执行文件（使用生成器表达式获取其路径）。`$<TARGET_FILE:target>`是一个特殊的生成器表达式，在此情况下它会隐式地添加对target的依赖，使其在执行所有命令之前构建。我们将target作为此函数的参数提供。
- 从当前目录（-d .）收集解决方案的度量，并输出到文件（-o coverage.info）中。
- 删除（-r）不需要的覆盖数据（'/usr/include/*'）并输出到另一个文件（-o filtered.info）。
- 在coverage目录中生成 HTML 报告，并添加一个--legend颜色。
- 删除临时.info文件。
- 指定WORKING_DIRECTORY关键字可以将二叉树作为所有命令的工作目录。


这些是 GCC 和 Clang 通用的步骤，但重要的是要知道gcov工具的版本必须与编译器的版本匹配。
换句话说，不能用 GCC 的gcov工具来编译 Clang 代码。
要使lcov指向 Clang 的gcov工具，我们可以使用`--gcov-tool`参数。
这里唯一的问题是它必须是一个单一的可执行文件。
为了解决这个问题，我们可以提供一个简单的包装脚本（别忘了用chmod +x将其标记为可执行文件）

```shell
// gcov-llvm-wrapper.sh
#!/bin/bash
exec llvm-cov gcov "$@"
```

我们之前函数中所有对`${LCOV_PATH}`的调用应接受以下额外标志：
`--gcov-tool ${CMAKE_SOURCE_DIR}/cmake/gcov-llvm-wrapper.sh`

确保此函数可用于包含在test列表文件中
我们可以通过在主列表文件中扩展包含搜索路径来实现：
```js
// /root/CMakeLists.txt
cmake_minimum_required(VERSION 3.20.0)
project(Coverage CXX)
enable_testing()
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
add_subdirectory(src bin)
add_subdirectory(test)
```
这行小代码允许我们将cmake目录中的所有.cmake文件包括在我们的项目中。
现在我们可以在test列表文件中使用Coverage.cmake

```js
// /test/CMakeLists.txt 片段
# ... skipped unit_tests target declaration for brevity
include(Coverage)
AddCoverage(unit_tests)
include(GoogleTest)
gtest_discover_tests(unit_tests)
```

为了构建这个目标，请使用以下命令（注意第一个命令以DCMAKE_BUILD_TYPE=Debug构建类型选择结束）：
```js
# cmake -B <binary_tree> -S <source_tree> 
  -DCMAKE_BUILD_TYPE=Debug
# cmake --build <binary_tree> -t coverage
```

完成所有提到的步骤后，你将看到一个简短的摘要，就像这样：
```js
Writing directory view page.
Overall coverage rate:
  lines......: 95.2% (20 of 21 lines)
  functions..: 75.0% (6 of 8 functions)
[100%] Built target coverage
```

接下来，在你的浏览器中打开coverage/index.html文件，享受这些报告吧！不过有一个小问题……

`避免 SEGFAULT 陷阱`

当我们开始在如此解决方案中编辑源代码时，我们可能会陷入困境。这是因为覆盖信息被分成两部分，如下所示：
- gcno文件，或GNU 覆盖笔记，在 SUT 编译期间生成
- gcda文件，或GNU 覆盖数据，在测试运行期间生成和更新

“更新”功能是段错误的一个潜在来源。
在我们最初运行测试后，我们留下了许多gcda文件，在任何时候都没有被移除。
如果我们对源代码做一些更改并重新编译对象文件，将创建新的gcno文件。
然而，没有擦除步骤——旧的gcda文件仍然跟随过时的源代码。
当我们执行unit_tests二进制文件（它在gtest_discover_tests宏中发生）时，覆盖信息文件将不匹配，我们将收到一个SEGFAULT（段错误）错误。

为了避免这个问题，我们应该删除任何过时的gcda文件。
由于我们的sut实例是一个静态库，我们可以将add_custom_command(TARGET)命令挂钩到构建事件上。
在重建开始之前，将执行清理。





# ch9 程序分析工具



## 强制格式化

`clang-format`

`clang-format -i --style=LLVM filename1.cpp filename2.cpp`
`-i`选项告诉 ClangFormat 就地编辑文件。
`--style`选择应使用哪种支持的格式化样式：LLVM、Google、Chromium、Mozilla、WebKit或自定义，从file提供（在进一步阅读部分有详细信息的链接）。

当然，我们不想每次修改后都手动执行这个命令；CMake 应该在构建过程中处理这个问题。
我们已经知道如何在系统中找到clang-format（我们之前需要手动安装它）。我们还没有讨论的是将外部工具应用于所有源文件的过程。
为此，我们将创建一个方便的函数，可以从cmake目录中包含：

```js
// Format.cmake
function(Format target directory)
  find_program(CLANG-FORMAT_PATH clang-format REQUIRED)
  set(EXPRESSION h hpp hh c cc cxx cpp)
  list(TRANSFORM EXPRESSION PREPEND "${directory}/*.")
  file(GLOB_RECURSE SOURCE_FILES FOLLOW_SYMLINKS
       LIST_DIRECTORIES false ${EXPRESSION}
  )
  add_custom_command(TARGET ${target} PRE_BUILD COMMAND
    ${CLANG-FORMAT_PATH} -i --style=file ${SOURCE_FILES}
  )
endfunction()
```

Format函数接受两个参数：target和directory。
它将格式化来自directory的所有源文件，在构建target之前。

这个函数有以下几个步骤：
- 查找系统中安装的clang-format二进制文件。REQUIRED关键字将在找不到二进制文件时停止配置并显示错误。
- 创建一个要格式化的文件扩展名列表（用作通配符表达式）。
- 在每个表达式前加上directory的路径。
- 递归搜索源文件和头文件（使用之前创建的列表），跳过目录，并将它们的路径放入SOURCE_FILES变量中。
- 将格式化命令作为target的`PRE_BUILD`步骤。


这个命令对于小到中等大小的代码库来说效果很好。
对于大量文件，我们需要将绝对文件路径转换为相对路径，并使用directory作为工作目录执行格式化（list(TRANSFORM)命令在这里很有用）。
这可能是因为传递给 shell 的命令长度有限制（通常约为 13,000 个字符），而太多的长路径根本放不下。

项目结构
```js
- CMakeLists.txt
- .clang-format
- cmake
  |- Format.cmake
- src
  |- CMakeLists.txt
  |- header.h
  |- main.cpp
```

我们需要设置项目并将cmake目录添加到模块路径中，这样我们稍后才能包含它

```js
// /root/CMakeLists.txt
cmake_minimum_required(VERSION 3.20.0)
project(Formatting CXX)
enable_testing()
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
add_subdirectory(src bin)
```

```js
// src/CMakeLists.txt
add_executable(main main.cpp)
include(Format)
Format(main .)
```
创建了一个名为main的可执行目标，包含了Format.cmake模块，并在当前目录（src）中调用了Format()函数。


格式化器的配置文件（可在命令行中使用--style=file参数启用）
```yaml
// .clang-format
BasedOnStyle: Google
ColumnLimit: 140
UseTab: Never
AllowShortLoopsOnASingleLine: false
AllowShortFunctionsOnASingleLine: false
AllowShortIfStatementsOnASingleLine: false
```
限制列数为 140 个字符，移除制表符，并允许短循环、函数和if语句。


。。我还是喜欢， eclipse那种，选中多行，c+s+f


## 使用静态检查器

严格应用静态检查器显著提高了代码的质量

CMake 支持为以下工具启用检查器：
- include-what-you-use (include-what-you-use.org)
- Clang-Tidy (clang.llvm.org/extra/clang-tidy)
- 链接你所使用的（内置的 CMake 检查器）
- cpplint (github.com/cpplint/cpplint)
- Cppchecker (cppcheck.sourceforge.io)

我们只需要做的是为适当的目标属性设置一个分号分隔的列表，该列表包含检查器可执行文件的路径，后跟任何应传递给检查器的命令行选项：
- `<LANG>_CLANG_TIDY`
- `<LANG>_CPPCHECK`
- `<LANG>_CPPLINT`
- `<LANG>_INCLUDE_WHAT_YOU_USE`
- `LINK_WHAT_YOU_USE`

`<LANG>`应该用所使用的语言替换，所以用C表示 C 源文件，用CXX表示 C++。
如果你不需要针对每个目标控制检查器，可以通过设置一个前缀为CMAKE_的适当的全局变量，为项目中的所有目标指定一个默认值，例如以下：
`set(CMAKE_CXX_CLANG_TIDY /usr/bin/clang-tidy-3.9;-checks=*)`
在此声明之后定义的任何目标，其`CXX_CLANG_TIDY`属性将以此方式设置


更细粒度地控制检查器如何测试目标有一定的价值。我们可以编写一个简单的函数来解决这个问题：

```js
// /root/cmake/ClangTidy.cmake
function(AddClangTidy target)
  find_program(CLANG-TIDY_PATH clang-tidy REQUIRED)
  set_target_properties(${target}
    PROPERTIES CXX_CLANG_TIDY
    "${CLANG-TIDY_PATH};-checks=*;--warnings-as-errors=*"
  )
endfunction()
```

AddClangTidy函数有两个简单步骤：
- 查找 Clang-Tidy 二进制文件并将其路径存储在CLANG-TIDY_PATH中。REQUIRED关键字将在找不到二进制文件时停止配置并显示错误。
- 在target上启用 Clang-Tidy，提供二进制文件的路径和自定义选项以启用所有检查，并将警告视为错误。

要使用这个函数，我们只需要包含模块并针对所选目标调用它：
```js
// src/CMakeLists.txt
add_library(sut STATIC calc.cpp run.cpp)
target_include_directories(sut PUBLIC .)
add_executable(bootstrap bootstrap.cpp)
target_link_libraries(bootstrap PRIVATE sut)
include(ClangTidy)
AddClangTidy(sut)
```

clang-tidy还提供了一个有趣的--fix选项，它可以自动尽可能地修复你的代码。这绝对是节省时间的好方法，并且在增加检查数量时可以随时使用。
与格式化一样，确保在将静态分析工具生成的任何更改引入遗留代码库时避免合并冲突。

clang-tidy，基于clang
cpplint，用于检查遵循 Google C++风格指南的 C/C++文件的风格问题，用python编写
cppcheck，旨在能够分析具有非标准语法（在嵌入式项目中很常见）的您的 C/C++代码。这个工具非常值得推荐，它能让您在使用时无忧无虑，避免由于误报而产生的不必要噪音
include-what-you-use  主要目标是去除不必要的#include
Link what you use  内置的 CMake 功能，使用 ld 和 ldd 的选项来输出如果可执行文件链接了比实际需要更多的库



## 使用Valgrind进行动态分析

Valgrind (www.valgrind.org) 是一个允许构建动态分析工具的框架——即在程序运行时执行的分析。它提供了一个广泛的工具套件，允许进行各种调查和检查。其中一些工具如下：
- Memcheck – 检测==内存==管理问题
- Cachegrind – 分析 ==CPU 缓存==，并定位缓存缺失和其他缓存问题
- Callgrind – Cachegrind 的扩展，带有关于==调用图==的额外信息
- Massif – 一种堆分析器，可以显示程序随时间使用==堆==的情况
- Helgrind – 线程调试器，有助于解决==数据竞争==问题
- DRD – Helgrind 的更轻量级、有限版本

当人们提到 Valgrind 时，他们经常会指的是 Valgrind 的 Memcheck。
让我们找出如何使用它与 CMake 一起工作——这将为您需要它们时采用其他工具铺平道路。



### memcheck

可能出现各种错误：读取未分配的内存、读取已经释放的内存、尝试多次释放内存以及写入错误的地址

调用memcheck
`valgrind [valgrind-options] tested-binary [binary-options]`

Memcheck 是 Valgrind 的默认工具，但您也可以明确选择它
`valgrind --tool=memcheck tested-binary`


运行 Memcheck 代价昂贵；手册（参见进一步阅读中的链接）说，用它 instrumented 的程序可以慢 10-15 倍。
为了避免每次运行测试时都要等待 Valgrind，我们将创建一个可以在需要测试代码时从命令行调用的独立目标。
理想情况下，开发者会在将他们的更改合并到仓库的默认分支之前运行它。
这可以通过早期 Git 钩子或添加为 CI 管道中的一个步骤来实现。
在生成阶段完成后，我们将使用以下命令来构建自定义目标

`cmake --build <build-tree> -t valgrind`


```js
// cmake/Valgrind.cmake
function(AddValgrind target)
  find_program(VALGRIND_PATH valgrind REQUIRED)
  add_custom_target(valgrind
    COMMAND ${VALGRIND_PATH} --leak-check=yes 
            $<TARGET_FILE:${target}>
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
  )
endfunction()
```
- CMake 会在默认的系统路径中搜索valgrind可执行文件，并将其存储在VALGRIND_PATH变量中。如果找不到二进制文件，REQUIRED关键字会导致配置出现错误而停止。
- 创建了一个自定义目标valgrind；它将在target二进制文件上执行 Memcheck 工具。我们还添加了一个选项，始终检查内存泄漏。

谈到 Valgrind 选项时，我们可以提供命令行参数，也可以如下进行：
- `~/.valgrindrc`文件（在你的家目录中）
- `$VALGRIND_OPTS`环境变量
- `./.valgrindrc`文件（在工作目录中）
这些按顺序进行检查

要使用AddValgrind函数，我们应该向其提供一个 unit_tests 目标
```js
// test/CMakeLists.txt 片段
# ...
add_executable(unit_tests calc_test.cpp run_test.cpp)
# ...
include(Valgrind)
AddValgrind(unit_tests)
```

请记住，使用Debug配置生成构建树可以让 Valgrind 访问调试信息，这使得它的输出更加清晰
`# cmake --build <build-tree> -t valgrind`


### memcheck-cover

Valgrind 是一个非常实用的工具，但在处理更复杂的程序时可能会变得有些冗长。必须有一种方法以更易管理的形式收集这些信息。

商业 IDE，如 CLion，原生支持解析 Valgrind 的输出，以便可以通过 GUI 轻松导航，而不必滚动控制台窗口以找到正确的消息。
如果你的编辑器没有这个选项，你仍然可以通过使用第三方报告生成器获得更清晰的错误视图。
由 David Garcin 编写的 Memcheck-cover 提供了一个更愉快的体验，以生成的 HTML 文件的形式


这个小巧的项目在 GitHub 上可用（github.com/Farigh/memcheck-cover）；它需要 Valgrind 和gawk（GNU AWK 工具）。
要使用它，我们将在一个单独的 CMake 模块中准备一个设置函数。它将由两部分组成：
- 获取和配置工具
- 添加一个自定义目标，执行 Valgrind 并生成报告


```js
// /cmake/Memcheck.cmake
function(AddMemcheck target)
  include(FetchContent)
  FetchContent_Declare(
   memcheck-cover
   GIT_REPOSITORY https://github.com/Farigh/memcheck-cover.git
   GIT_TAG        release-1.2
  )
  FetchContent_MakeAvailable(memcheck-cover)
  set(MEMCHECK_PATH ${memcheck-cover_SOURCE_DIR}/bin)
```

在第一部分中，我们遵循与常规依赖项相同的实践：
包含FetchContent模块，
并在FetchContent_Declare中指定项目的存储库和所需的 Git 标签。
接下来，我们启动获取过程，并使用由FetchContent_Populate设置的（由FetchContent_MakeAvailable隐式调用）memcheck-cover_SOURCE_DIR变量配置二进制文件的路径。


函数的第二部分是创建生成报告的目标。
我们将其命名为memcheck（这样如果出于某种原因想要保留这两个选项，它就不会与之前的valgrind目标重叠）：
```js
// /cmake/CMakeLists.txt 续
  add_custom_target(memcheck
    COMMAND ${MEMCHECK_PATH}/memcheck_runner.sh -o 
      "${CMAKE_BINARY_DIR}/valgrind/report" 
      -- $<TARGET_FILE:${target}>
    COMMAND ${MEMCHECK_PATH}/generate_html_report.sh 
      -i "${CMAKE_BINARY_DIR}/valgrind" 
      -o "${CMAKE_BINARY_DIR}/valgrind"
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
  )
endfunction()
```
这种情况发生在两个命令中：
- 首先，我们将运行memcheck_runner.sh包装脚本，该脚本将执行 Valgrind 的 Memcheck 并收集通过`-o`参数提供的文件输出的输出。
- 然后，我们将解析输出并使用generate_html_report.sh创建报告。这个脚本需要通过-i和-o参数提供的输入和输出目录。


我们还需要在我们的列表文件中添加的最后一样东西，当然是调用这个函数的调用。它的模式和AddValgrind一样：
```js
// test/CMakeLists.txt
include(Memcheck)
AddMemcheck(unit_tests)
```

在用Debug配置生成构建系统后，我们可以用以下命令来构建目标：
`cmake --build <build-tree> -t memcheck`



# ch10 生成文档


## 向您的项目添加 Doxygen

Doxygen 可以生成以下格式的文档：
- 超文本标记语言（HTML）
- 富文本格式（RTF）
- 便携式文档格式（PDF）
- Lamport 的 TeX（LaTeX）
- PostScript（PS）
- Unix 手册（手册页）
- 微软编译的 HTML 帮助文件（CHM）


如果你用 Doxygen 指定的格式为代码添加注释，提供额外信息，它将被解析以丰富输出文件。
更重要的是，将分析代码结构以生成有益的图表和图表。
后者是可选的，因为它需要一个外部的 Graphviz 工具（graphviz.org/）。

开发者首先应该回答以下问题：
*项目的用户只是获得文档，还是他们自己生成文档（也许是在从源代码构建时）？*
第一个选项意味着文档与二进制文件一起提供，可供在线获取，或者（不那么优雅地）与源代码一起提交到仓库中。

答案很重要，因为如果我们希望用户在构建过程中生成文档，他们需要在他们的系统中拥有这些依赖项。
由于 Doxygen 可以通过大多数包管理器（以及 Graphviz）获得，所需的就是一个简单的命令，比如针对 Debian 的这样一个命令
`apt-get install doxygen graphviz`

当 Doxygen 和 Graphviz 安装在系统中时，我们可以将生成功能添加到我们的项目中。
与在线资料所建议的不同，这并不像我们想象的那么困难或复杂。
我们不需要创建外部配置文件，提供doxygen可执行文件的路径，或者添加自定义目标。
自从 CMake 3.9 以来，我们可以使用FindDoxygen模块中的doxygen_add_docs()函数来设置文档目标。

```js
doxygen_add_docs(targetName [sourceFilesOrDirs...]
  [ALL] [USE_STAMP_FILE] [WORKING_DIRECTORY dir]
  [COMMENT comment])
```

第一个参数指定了目标名称，我们需要使用cmake的-t参数（在生成构建树之后）显式构建它
`cmake --build <build-tree> -t targetName`

或者，我们总是可以通过添加 ALL 参数（通常不必要）来构建它。
其他选项相当直观，除了可能 USE_STAMP_FILE。
这允许 CMake 在源文件没有更改的情况下跳过文档的重新生成（但要求 sourceFilesOrDirs 只包含文件）。

我们将遵循前几章的做法，创建一个带有辅助函数的工具模块（以便在其他项目中重复使用）

```js
// /cmake/Doxygen.cmake
function(Doxygen input output)
  find_package(Doxygen)
  if (NOT DOXYGEN_FOUND)
    add_custom_target(doxygen COMMAND false 
      COMMENT "Doxygen not found")
    return()
  endif()
  set(DOXYGEN_GENERATE_HTML YES)
  set(DOXYGEN_HTML_OUTPUT
    ${PROJECT_BINARY_DIR}/${output})
  doxygen_add_docs(doxygen
      ${PROJECT_SOURCE_DIR}/${input}
      COMMENT "Generate HTML documentation"
  )
endfunction()
```

该函数接受两个参数——input 和 output 目录，并将创建一个自定义 doxygen 目标。这里发生了什么：
- 首先，我们将使用 CMake 内置的 Doxygen 查找模块来确定系统中是否可用 Doxygen。
- 如果不可用，我们将创建一个虚拟 doxygen 目标，该目标将通知用户并运行一个 false 命令，该命令（在 Unix-like 系统上）返回 1，导致构建失败。我们在此时终止函数并用 return()。
- 如果系统中可用 Doxygen，我们将配置它以在提供的 output 目录中生成 HTML 输出。Doxygen 非常可配置（更多信息请参阅官方文档）。要设置任何选项，只需按照示例通过调用 set() 并将其名称前缀为 DOXYGEN_。
- 设置实际的 doxygen 目标：所有 DOXYGEN_ 变量都将转发到 Doxygen 的配置文件中，并且将从源树中的提供的 input 目录生成文档。

如果你 documentation 要由用户生成，步骤 2 可能应该涉及安装必要的依赖项。

要使用这个函数，我们可以在我们项目的 main listfile 中添加它
```js
// /root/CMakeLists.txt
cmake_minimum_required(VERSION 3.20.0)
project(Doxygen CXX)
enable_testing()
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
add_subdirectory(src bin)
include(Doxygen)
Doxygen(src docs)
```


你可以在成员函数文档中看到的额外描述是通过在头文件中添加适当注释来实现的：
```C++
   /**
    Multiply... Who would have thought?
    @param a the first factor
    @param b the second factor
    @result The product
   */
   int Multiply(int a, int b);
```

如果安装了 Graphviz，Doxygen 将检测到它并生成依赖关系图



## 使用现代外观生成文档

许多开发者会抱怨 Doxygen 提供的设计过时，这让他们犹豫是否向客户展示生成的文档。别担心——有一个简单的解决方案可以解决这个问题。

一个名为jothepro的开发者创建了一个名为doxygen-awesome-css的主题，它提供了一个现代、可自定义的设计。它甚至还有夜间模式
该主题不需要任何额外的依赖项，可以很容易地从其 GitHub 页面`github.com/jothepro/doxygen-awesome-css`获取。


让我们看看如何通过添加一个新的宏来扩展我们的Doxygen.cmake文件以使用它
```js
// /cmake/Doxygen.cmake
macro(UseDoxygenAwesomeCss)
  include(FetchContent)
  FetchContent_Declare(doxygen-awesome-css
    GIT_REPOSITORY
      https://github.com/jothepro/doxygen-awesome-css.git
    GIT_TAG
      v1.6.0
  )
  FetchContent_MakeAvailable(doxygen-awesome-css)
  set(DOXYGEN_GENERATE_TREEVIEW     YES)
  set(DOXYGEN_HAVE_DOT              YES)
  set(DOXYGEN_DOT_IMAGE_FORMAT      svg)
  set(DOXYGEN_DOT_TRANSPARENT       YES)
  set(DOXYGEN_HTML_EXTRA_STYLESHEET
    ${doxygen-awesome-css_SOURCE_DIR}/doxygen-awesome.css)
endmacro()
```

我们已经在书的 previous chapters 中了解到了所有这些命令，但为了完全清晰，让我们重申一下发生了什么，如下所示：
- doxygen-awesome-css通过FetchContent模块从 Git 中提取，并作为项目的一部分提供。
- 为 Doxygen 配置了额外的选项，如主题的README文件中所建议。
- DOXYGEN_HTML_EXTRA_STYLESHEET配置了主题的.css文件的路径。它将被复制到输出目录。


正如您所想象的，最好在Doxygen函数中调用这个宏，在doxygen_add_docs()之前
```js
// /cmake/Doxygen.cmake
function(Doxygen input output)
  ...
  UseDoxygenAwesomeCss()
  doxygen_add_docs (...)
endfunction()
macro(UseDoxygenAwesomeCss)
  ...
endmacro()
```
提醒，宏中的所有变量都在调用函数的作用域中设置。


---

### 其他文档生成工具

还有数十种其他工具未在此书中涉及，因为我们专注于由 CMake 支持的项目。然而，其中一些可能更适合您的用例。如果您想冒险，可以访问我在这里列出的两个我觉得有趣的项目的网站：

- Adobe 的 Hyde：github.com/adobe/hyde
  针对 Clang 编译器，Hyde 生成 Markdown 文件，这些文件可以被如 Jekyll(jekyllrb.com/)等工具消费，Jekyll 是一个由 GitHub 支持的静态页面生成器。

- Standardese：github.com/standardese/standardese
  该工具使用libclang编译您的代码，并提供 HTML、Markdown、LaTex 和 man 页面的输出。它大胆地目标是成为下一个 Doxygen。






# ch11 安装和打包

安装使我们的项目能够在系统范围内被发现和访问

设置可重用的 CMake 包，以便它们可以被其他项目通过调用find_package()发现。

## 无需安装导出

我们如何使项目A的目标对消费项目B可用？
通常，我们会使用find_package()命令，但这意味着我们需要创建一个包并在系统上安装它。
这种方法很有用，但需要一些工作


我们可以通过包含A的主列表文件来节省一些时间：它已经包含了所有的目标定义。
不幸的是，它也可能包含很多其他内容：全局配置、需求、具有副作用的 CMake 命令、附加依赖项，以及我们可能不想在B中出现的目标（如单元测试）。
所以，我们不要这样做。
更好的方法是提供B，并通过include()命令包含
```js
cmake_minimum_required(VERSION 3.20.0)
project(B)
include(/path/to/project-A/ProjectATargets.cmake)
```
执行此操作将为A的所有目标提供正确的属性集定义（如add_library()和add_executable()等命令）。

当然，我们不会手动写这样的文件——这不会是一个非常 DRY 的方法。CMake 可以用export()命令为我们生成这些文件，该命令具有以下签名：
```js
export(TARGETS [target1 [target2 [...]]] 
  [NAMESPACE <namespace>] [APPEND] FILE <path>
  [EXPORT_LINK_INTERFACE_LIBRARIES])
```

我们必须提供所有我们想要导出的目标，在TARGET关键字之后，并提供目标文件名在FILE之后。其他参数是可选的：
- NAMESPACE建议作为一个提示，说明目标已经从其他项目中导入。
- APPEND告诉 CMake 在写入文件之前不要擦除文件的内容。
- EXPORT_LINK_INTERFACE_LIBRARIES将导出目标链接依赖（包括导入和配置特定的变体）。

---

让我们用我们示例中的 Calc 库来看看这个功能，它提供了两个简单的方法：
```C++
// /src/include/calc/calc.h
#pragma once
int Sum(int a, int b);
int Multiply(int a, int b);
```

我们这样声明它的目标
```js
// /src/CMakeLists.txt
add_library(calc STATIC calc.cpp)
target_include_directories(calc INTERFACE include)
```

然后，我们要求 CMake 使用export(TARGETS)命令生成导出文件
```js
// /CMakeLists.txt 片段
cmake_minimum_required(VERSION 3.20.0)
project(ExportCalcCXX)
add_subdirectory(src bin)
set(EXPORT_DIR "${CMAKE_CURRENT_BINARY_DIR}/cmake")
export(TARGETS calc
  FILE "${EXPORT_DIR}/CalcTargets.cmake"
  NAMESPACE Calc::
)
...
```

在前面的代码中，我们可以看到EXPORT_DIR变量已被设置为构建树中的cmake子目录（按照.cmake文件的约定）。
然后，我们导出目标声明文件CalcTargets.cmake，其中有一个名为calc的单一目标，对于将包含此文件的工程项目，它将作为Calc::calc可见。

请注意，这个导出文件还不是包。
更重要的是，这个文件中的所有路径都是绝对的，且硬编码到构建树中。
换句话说，它们是不可移动的（我们将在理解可移动目标的问题部分讨论这个问题）

---

export()命令还有一个更短的版本：
`export(EXPORT <export> [NAMESPACE <namespace>] [FILE <path>])`

它需要一个`<export>`名称，而不是一个导出的目标列表。这样的`<export>`实例是由install(TARGETS)定义的目标的命名列表（我们将在安装逻辑目标部分介绍这个命令）。

```js
// /root/CMakeLists.txt 续
...
install(TARGETS calc EXPORT CalcTargets)
export(EXPORT CalcTargets
  FILE "${EXPORT_DIR}/CalcTargets2.cmake"
  NAMESPACE Calc::
)
```
前面的代码与之前的代码完全一样，但现在，export() 和 install() 命令之间的单个目标列表被共享。

生成导出文件的两个方法会产生相同的结果。它们将包含一些模板代码和几行定义目标的内容。将 /tmp/b 设置为构建树路径时，它们看起来像这样：
```js
// /tmp/b/cmake/CalcTargets.cmake 片段
# Create imported target Calc::calc
add_library(Calc::calc STATIC IMPORTED)
set_target_properties(Calc::calc PROPERTIES
  INTERFACE_INCLUDE_DIRECTORIES
  "/root/examples/chapter11/01-export/src/include"
)
# Import target "Calc::calc" for configuration ""
set_property(TARGET Calc::calc APPEND PROPERTY
  IMPORTED_CONFIGURATIONS NOCONFIG
)
set_target_properties(Calc::calc PROPERTIES
  IMPORTED_LINK_INTERFACE_LANGUAGES_NOCONFIG "CXX"
  IMPORTED_LOCATION_NOCONFIG "/tmp/b/libcalc.a"
)
```
通常，我们不会编辑这个文件，甚至不会打开它，但我想要强调这个生成文件中的硬编码路径。以其当前形式，这个包是不可移动的。



## 在系统上安装项目

在第1章 CMake 初学者中，我们提到 CMake 提供了一个命令行模式，可以在系统上安装构建好的项目：

`cmake --install <dir> [<options>]`

`<dir>` 是生成构建树的目标路径（必需）。
我们的 `<options>` 如下：
- `--config <cfg>`：这对于多配置生成器，选择构建配置。
- `--component <comp>`：这限制了安装到给定组件。
- `--default-directory-permissions <permissions>`：这设置了安装目录的默认权限（在 `<u=rwx,g=rx,o=rx>` 格式中）。
- `--prefix <prefix>`：这指定了非默认的安装路径（存储在 CMAKE_INSTALL_PREFIX 变量中）。对于类 Unix 系统，默认为 /usr/local，对于 Windows，默认为 c:/Program Files/${PROJECT_NAME}。
- `-v, --verbose`：这会使输出详细（这也可以通过设置 VERBOSE 环境变量来实现）。

安装可以由许多步骤组成，但它们的本质是将生成的工件和必要的依赖项复制到系统上的某个目录中。使用 CMake 进行安装不仅为所有 CMake 项目引入了一个方便的标准，而且还做了以下事情：
- 为根据它们的类型提供特定于平台的安装路径（遵循GNU 编码标准）
- 通过生成目标导出文件，增强安装过程，允许项目目标直接被其他项目重用
- 通过配置文件创建可发现的包，这些文件封装了目标导出文件以及作者定义的特定于包的 CMake 宏和函数

这些功能非常强大，因为它们节省了很多时间，并简化了以这种方式准备的项目使用。执行基本安装的第一步是将构建好的工件复制到目标目录。

这让我们来到了 install() 命令及其各种模式：
- install(TARGETS)：这会安装输出工件，如库和可执行文件。
- install(FILES|PROGRAMS)：这会安装单个文件并设置它们的权限。
- install(DIRECTORY): 这会安装整个目录。
- install(SCRIPT|CODE)：在安装期间运行 CMake 脚本或代码段。
- install(EXPORT)：这生成并安装一个目标导出文件。

将这些命令添加到您的列表文件中将生成一个`cmake_install.cmake`文件在您的构建树中。
虽然可以手动调用此脚本使用`cmake -P`，但不建议这样做。
这个文件是用来在执行`cmake --install`时由 CMake 内部使用的。

即将推出的 CMake 版本还将支持安装运行时工件和依赖集合，因此请务必查阅最新文档以了解更多信息。

每个install()模式都有一组广泛的选项。其中一些是共享的，并且工作方式相同：
- DESTINATION：这指定了安装路径。相对路径将前缀CMAKE_INSTALL_PREFIX，而绝对路径则直接使用（并且cpack不支持）。
- PERMISSIONS：这设置了支持它们的平台上的文件权限。可用的值有OWNER_READ、OWNER_WRITE、OWNER_EXECUTE、GROUP_READ、GROUP_WRITE、GROUP_EXECUTE、WORLD_READ、WORLD_WRITE、WORLD_EXECUTE、SETUID 和 SETGID。在安装期间创建的目录的默认权限可以通过指定CMAKE_INSTALL_DEFAULT_DIRECTORY_PERMISSIONS变量来设置。
- CONFIGURATIONS：这指定了一个配置列表（Debug、Release）。此命令中跟随此关键字的所有选项仅当当前构建配置在此列表中时才会被应用。
- OPTIONAL：这禁用了在安装的文件不存在时引发错误。

在组件特定安装中还使用了两个共享选项：COMPONENT和EXCLUDE_FROM_ALL。我们将在定义组件部分详细讨论这些内容。

---

### 安装逻辑目标

让我们看看第一个安装模式：install(TARGETS)。

由add_library()和add_executable()定义的目标可以很容易地使用`install(TARGETS)`命令安装。
这意味着将构建系统产生的工件复制到适当的目标目录并将它们的文件权限设置为合适。此模式的通用签名如下：
```js
install(TARGETS <target>... [EXPORT <export-name>]
        [<output-artifact-configuration> ...]
        [INCLUDES DESTINATION [<dir> ...]]
        )
```
在初始模式指定符 – 即TARGETS – 之后，我们必须提供一个我们想要安装的目标列表。
在这里，我们可以选择性地将它们分配给EXPORT选项，该选项可用于export(EXPORT)和install(EXPORT)以生成目标导出文件。
然后，我们必须配置输出工件（按类型分组）的安装。
可选地，我们可以提供一系列目录，这些目录将添加到每个目标在其INTERFACE_INCLUDE_DIRECTORIES属性中的目标导出文件中。

`[<output-artifact-configuration>...]` 提供了一个配置块列表。单个块的完整语法如下：
```js
<TYPE> [DESTINATION <dir>] [PERMISSIONS permissions...]
       [CONFIGURATIONS [Debug|Release|...]]
       [COMPONENT <component>]
       [NAMELINK_COMPONENT <component>]
       [OPTIONAL] [EXCLUDE_FROM_ALL]
       [NAMELINK_ONLY|NAMELINK_SKIP]
```

每个输出工件块都必须以`<TYPE>`开头（这是唯一必需的元素）。CMake 识别它们中的几个：
- ARCHIVE：静态库（.a）和基于 Windows 系统的 DLL 导入库（.lib）。
- LIBRARY：共享库（.so），但不包括 DLL。
- RUNTIME：可执行文件和 DLL。
- OBJECTS：来自OBJECT库的对象文件。
- FRAMEWORK：设置了FRAMEWORK属性的静态和共享库（这使它们不属于ARCHIVE和LIBRARY）。这是 macOS 特定的。
- BUNDLE：标记有MACOSX_BUNDLE的可执行文件（也不是RUNTIME的一部分）。
- PUBLIC_HEADER、PRIVATE_HEADER、RESOURCE：在目标属性中指定相同名称的文件（在苹果平台上，它们应该设置在FRAMEWORK或BUNDLE目标上）。

CMake 文档声称，如果你只配置了一种工件类型（例如，LIBRARY），只有这种类型将被安装。
对于 CMake 3.20.0 版本，这并==不正确：所有工件==都将以默认选项配置的方式安装。
这可以通过为所有不需要的工件类型指定`<TYPE> EXCLUDE_FROM_ALL`来解决。

单个install(TARGETS)命令可以有多个工件配置块。但是，请注意，每次调用您可能只能指定每种类型的一个。
也就是说，如果您想要为Debug和Release配置指定不同位置的ARCHIVE工件，那么您必须分别进行两次install(TARGETS ... ARCHIVE)调用。

你也可以省略类型名称，为所有工件指定选项：
```js
install(TARGETS executable, static_lib1
  DESTINATION /tmp
)
```


### 为不同平台确定正确的目的地

目标路径的公式如下所示：
`${CMAKE_INSTALL_PREFIX} + ${DESTINATION}`

如果未提供DESTINATION，CMake 将使用每个类型的内置默认值：

|artifact type|built-in guess|install directory variable|
|--|--|--|
|RUNTIME|bin|CMAKE_INSTALL_BINDIR|
|LIBRARY,ARCHIVE|lib|CMAKE_INSTALL_LIBDIR|
|PRIVATE_HEADER,PUBLIC_HEADER|include|CMAKE_INSTALL_INCLUDEDIR|



### 处理公共头文件

install(TARGETS)文档建议我们在库目标的PUBLIC_HEADER属性中（用分号分隔）指定公共头文件

```js
// /src/CMakeLists.txt
add_library(calc STATIC calc.cpp)
target_include_directories(calc INTERFACE include)
set_target_properties(calc PROPERTIES
  PUBLIC_HEADER src/include/calc/calc.h
)
```
如果我们使用 Unix 的默认“猜测”方式，文件最终会出现在/usr/local/include

我们想要在公共头文件的目的地calc前加上CMAKE_INSTALL_INCLUDEDIR
```js
// root/CMakeLists.txt
cmake_minimum_required(VERSION 3.20.0)
project(InstallTargets CXX) 
add_subdirectory(src bin)
include(GNUInstallDirs)
install(TARGETS calc
  ARCHIVE
  PUBLIC_HEADER
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/calc
)
```

在从src包含列表文件，定义了我们的calc目标之后，我们必须配置静态库及其公共头文件的安装。
我们已经包含了 GNUInstallDirs 模块，并明确指定了DESTINATION为PUBLIC_HEADERS。
以安装模式运行cmake将按预期工作

```js
# cmake -S <source-tree> -B <build-tree>
# cmake --build <build-tree>
# cmake --install <build-tree>
-- Install configuration: ""
-- Installing: /usr/local/lib/libcalc.a
-- Installing: /usr/local/include/calc/calc.h
```

这种方式对于这个基本案例来说很好，但有一个轻微的缺点：以这种方式指定的文件不保留它们的目录结构。它们都将被安装在同一个目的地，即使它们嵌套在不同的基本目录中。

计划在新版本中（CMake 3.23.0）使用FILE_SET关键字更好地管理头文件：

```js
target_sources(<target>
  [<PUBLIC|PRIVATE|INTERFACE>
   [FILE_SET <name> TYPE <type> [BASE_DIR <dir>] FILES]
   <files>...
  ]...
)
```

### 低级安装

现代 CMake 正在逐步放弃直接操作文件的概念。
理想情况下，我们总是将它们添加到一个逻辑目标中，并使用这个更高层次的抽象来表示所有底层资产：源文件、头文件、资源、配置等等。
主要优点是代码的简洁性：通常，我们添加一个文件到目标时不需要更改多于一行代码。

不幸的是，将每个已安装的文件添加到目标上并不总是可能的或方便的。
对于这种情况，有三种选择可用：`install(FILES)、install(PROGRAMS) 和 install(DIRECTORY)`。


---

使用 install(FILES|PROGRAMS) 安装文件集

FILES 和 PROGRAMS 模式非常相似。它们可以用来安装公共头文件、文档、shell 脚本、配置文件，以及所有种类的资产，包括图像、音频文件和将在运行时使用的数据集。
```js
install(<FILES|PROGRAMS> files...
        TYPE <type> | DESTINATION <dir>
        [PERMISSIONS permissions...]
        [CONFIGURATIONS [Debug|Release|...]]
        [COMPONENT <component>]
        [RENAME <name>] [OPTIONAL] [EXCLUDE_FROM_ALL])
```

FILES 和 PROGRAMS 之间的主要区别是新复制文件的默认文件权限设置。
install(PROGRAMS) 也会为所有用户设置 EXECUTE 权限，而 install(FILES) 不会（两者都会设置 OWNER_WRITE、OWNER_READ、GROUP_READ 和 WORLD_READ）。
你可以通过提供可选的 PERMISSIONS 关键字来改变这种行为，然后选择领先的关键字作为安装内容的指示器：FILES 或 PROGRAMS。
我们已经讨论了 PERMISSIONS、CONFIGURATIONS 和 OPTIONAL 如何工作。COMPONENT 和 EXCLUDE_FROM_ALL 在 定义组件 部分中稍后讨论。


下一个必需的关键字是 TYPE 或 DESTINATION。
我们可以显式提供 DESTINATION 路径，或者要求 CMake 为特定 TYPE 文件查找它。
与 install(TARGETS) 不同，TYPE 并不声称选择性地将要安装的文件子集安装到指定位置。
然而，计算安装路径遵循相同的模式（+ 符号表示平台特定的路径分隔符）
`${CMAKE_INSTALL_PREFIX} + ${DESTINATION}`

每个TYPE 都有内置猜测

![e1571fe818cab72995a427a49e12f2c5.png](../_resources/e1571fe818cab72995a427a49e12f2c5.png)

表中一些内置猜测的前缀是安装目录变量：
- $LOCALSTATE 是 CMAKE_INSTALL_LOCALSTATEDIR 或默认为 var
- $DATAROOT 是 CMAKE_INSTALL_DATAROOTDIR 或默认为 share



# ==。。g。。==


https://cloud.tencent.com/developer/article/2421609


## 创建可重用的包

我们需要完成几步，以便 CMake 可以将我们的项目视为一个连贯的包：
- 使我们的目标可移动。
- 将目标导出文件安装到标准位置。
- 为包创建配置文件和版本文件。




## 定义组件





## 使用 CPack 打包


对于终端用户来说，一种更加便捷的软件分发方式是使用包含编译工件和其他运行时所需静态文件的二进制包。CMake 通过名为cpack的命令行工具支持生成多种此类包。

以下表格列出了可用的包生成器

![9ada1869d89239da22ab0d65ee19e61d.png](../_resources/9ada1869d89239da22ab0d65ee19e61d.png)


要使用 CPack，我们需要正确配置项目的安装，并使用必要的install()命令构建项目。
在我们构建树中生成的cmake_install.cmake将用于cpack根据配置文件（CPackConfig.cmake）准备二进制包

```js
cmake_minimum_required(VERSION 3.20.0)
project(CPackPackage VERSION 1.2.3 LANGUAGES CXX)
include(GNUInstallDirs)
add_subdirectory(src bin)
install(...)
install(...)
install(...)
set(CPACK_PACKAGE_VENDOR "Rafal Swidzinski")
set(CPACK_PACKAGE_CONTACT "email@example.com")
set(CPACK_PACKAGE_DESCRIPTION "Simple Calculator")
include(CPack)
```

代码相当直观，所以我们不会过多地解释（请参考模块文档，可以在进一步阅读部分找到）。这里值得注意的一点是，CPack模块将从project()命令中推断出一些值：
- CPACK_PACKAGE_NAME
- CPACK_PACKAGE_VERSION
- CPACK_PACKAGE_FILE_NAME

最后一个值将用于生成输出包。其结构如下
`$CPACK_PACKAGE_NAME-$CPACK_PACKAGE_VERSION-$CPACK_SYSTEM_NAME`

在我们构建项目之后，我们可以在构建树中运行cpack二进制文件来生成实际的包
`cpack [<options>]`

从技术上讲，CPack 能够读取放置在当前工作目录中的所有配置文件选项，但你也可以选择从命令行覆盖这些设置：
- `-G <generators>`:这是一个由分号分隔的包生成器列表。默认值可以在CPackConfig.cmake中的CPACK_GENERATOR变量中指定。
- `-C <configs>`:这是一个由分号分隔的构建配置（调试、发布）列表，用于生成包（对于多配置构建系统生成器，这是必需的）。
- `-D <var>=<value>`: 这个选项会覆盖CPackConfig.cmake文件中设置的`<var>`变量，以`<value>`为准。
- `--config <config-file>`: 这是你应该使用的配置文件，而不是默认的CPackConfig.cmake。
- `--verbose, -V`: 提供详细输出。
- `-P <packageName>`: 覆盖包名称。
- `-R <packageVersion>`: 覆盖包版本。
- `--vendor <vendorName>`: 覆盖包供应商。
- `-B <packageDirectory>`: 为cpack指定输出目录（默认情况下，这将是目前的工作目录）。


让我们尝试为我们的12-cpack输出生成包。我们将使用 ZIP、7Z 和 Debian 包生成器：
`cpack -G "ZIP;7Z;DEB" -B packages`


以下应该生成以下包：
- CPackPackage-1.2.3-Linux.7z
- CPackPackage-1.2.3-Linux.deb
- CPackPackage-1.2.3-Linux.zip





# ch12 创建你的专业项目


https://cloud.tencent.com/developer/article/2421610



# 附录


string()命令

list()命令

file()命令

math()命令





