ProtoBuf Thrift Avro
2021年11月18日
11:15

序列化工具还有   kyro  Protostuff flatbuffer

==========================
VS

GitHub
14.5k fork；  56.6k star
https://github.com/protocolbuffers/protobuf

3.9k fork；  9.4k star
https://github.com/apache/thrift

1.4k fork；  2.3k star
https://github.com/apache/avro

2.9k fork；  18.9k star
https://github.com/google/flatbuffers

290 fork；  1.9k star (这个应该不维护了，最近一次  commit 7个月前，其他的几天内有commit)
https://github.com/protostuff/protostuff

上述是  2022-10-20  的数据

https://blog.csdn.net/wangqiang9x/article/details/84750046

Google protobuf：
优点
二进制消息，性能好，效率高  (时间，空间  效率都  很不错)
proto文件  生成目标代码，简单易用
序列化  反序列化  直接  对应  程序中的  数据类，不需要在  解析后  再进行映射  (XML,JSON需要)
支持向前兼容(新加字段  采用默认值)  和  向后兼容(忽略  新加  字段)，简化升级
支持多种语言  (可以把  proto文件看做  IDL  文件)
Netty  等一些框架集成

。。IDL是Interface description language的缩写

缺点
官方只支持  C++，Java，Python
二进制可读性差
二进制  不具有  自描述特性
默认不具备动态特性
只涉及  序列化，反序列化，  不涉及  RPC  功能

Apache  Thrift
优点
支持非常多的语言
thrift  文件生成目标代码，简单易用
消息定义文件  支持  注释
数据结构  和  传输表现  的分离，  支持多种消息格式
包含  完整的  客户端  服务端  堆栈，可快速实现RPC
支持  同步  和  异步通信

缺点
和protobuf  一样不支持  动态特性

Apache Avro
优点
二进制消息，性能好，效率高
使用JSON  描述模式
模式和数据统一存储，消息自描述，不需要生成  stub  代码(支持生成  IDL)
RPC  调用  在  握手阶段  交换  模式定义
包含完整的  客户端，服务端  堆栈，可以快速实现RPC
支持同步  和异步通信
支持  动态消息
模式定义  允许定义数据的  排序(序列化时  会遵循这个顺序)
提供了  基于  Jetty  内核的服务，  基于Netty的服务

缺点
只支持Avro自己的序列化格式
语言绑定不是  Thrift  丰富

--------------------------------

https://blog.csdn.net/weixin_33727510/article/details/90323290

一条消息数据，protobuf  序列化后的  大小是
json  的  10分之一，
xml  的  20分之一，
二进制序列化的10分之一

google  的开源序列化框架，类似于xml，json  这样的数据表示语言

在google  中是一个  比较  核心的  基础库，作为  分布式  运算  涉及到的  大量的  不同业务消息  的传递，如何  高效简洁  的表示，操作  这些  业务消息  在google  这样的  大规模  应用中  是  至关重要的。

protobuf  在  效率，数据大小，易用性  之间取得了  很好的  平衡。

个人总结的适用  protobuf  的场合

1. 需要和  其他系统  做消息交换的，  对消息大小  很敏感的。  那么  protobuf  适合，  它语言无关，消息占用的空间  相对于  xml  和json  等  节省很多。

2. 小数据的场合。  如果你是  大数据，用它并不适合

3. 项目语言是  C++，Java，Python。  因为有  google  的源生类库。  其他语言需要  第三方  或  自己写，序列化  和反序列化  效率  不能保证

。。2是为什么？   是指  压缩了  格式，   但是  数据压缩不了？

Thrift  是  facebook 2007年开发，之后在  2008加入到  Apache。  是一个  跨语言的  轻量级RPC  消息  和  数据交换框架。

要从  性能上  区分Thrift  和  Protobuf  之间做  选择意义不大，因为  它们的性能  太接近。

我们应该从  项目支持，文档，易用性，特性方面  来进行  选择。

Thrift  和  Protobuf  的最大不同，在于  Thrift  提供了完整的  RPC  支持，包含了  Server，Client；  而Protobuf  只包含了  stub  的  生成器  和  格式定义。

数据类型

|     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- |
| protobuf | thrift | protobuf | thrift | protobuf | thrift | protobuf | thrift |
| double | double | float |     |     | byte |     | i16 |
| int32 | i32 | int64 | i64 | uint32 |     | uint64 |     |
| sint32 |     | sint64 |     | fixed32 |     | fixed64 |     |
| sfixed32 |     | sfixed64 |     | bool | bool | string | string |
| bytes | binary | message | struct | enum | enum | service | service |

综合对比

|     |     |     |
| --- | --- | --- |
|     | protobuf | thrift |
| 功能特性 | 主要是一种序列化机制 | 提供了全套RPC解决方案，包括序列化机制、传输层、并发处理框架等 |
| 支持语言 | C++/Java/Python | C++, Java, Python, Ruby, Perl, PHP, C#, Erlang, Haskell |
| 易用性 | 语法类似，使用方式等类似 |     |
| 生成代码的质量 | 可读性都还过得去，执行效率另测 |     |
| 升级时版本兼容性 | 均支持向后兼容和向前兼容 |     |
| 学习成本 | 功能单一，容易学习 | 功能丰富、学习成本高 |
| 文档&社区 | 官方文档较为丰富，google搜索protocol buffer有2000W+结果，google group被墙不能访问 | 官方文档较少，没有API文档，google搜索apache thrift仅40W结果，邮件列表不怎么活跃 |

性能对比
由于  thrift  功能比  protobuf  丰富，因此单从  序列化机制上  进行  性能比较，
按照  序列化后字节数，序列化时间，反序列化时间  三个指标进行，对  thrift  的二进制，压缩，protobuf  三种格式进行对比。

测试方法

取了  15000+  条样本数据，分别写了  3个  指标的  测试程序，在  自己的  电脑上执行，其中时间测试  循环  1000次，总的序列化  反序列化  次数  1500W+。

平均字节数

|     |     |
| --- | --- |
| thrift二进制 | 535 |
| thrift压缩 | 473 |
| protobuf | 477 |

序列化(1500W次)  时间(ms):

|     |     |
| --- | --- |
| thrift二进制 | 306034 |
| thrift压缩 | 304256 |
| protobuf | 177652 |

反序列化(1500W次)时间(ms):

|     |     |
| --- | --- |
| thrift二进制 | 287972 |
| thrift压缩 | 315991 |
| protobuf | 157192 |

thrift  的时间测试  可能不是很准，由于  thrift  产生  代码的  复杂性，  编写的  测试代码  为了  适应其接口，在  调用堆栈上  可能有一些额外开销。

==========================

https://github.com/protocolbuffers/protobuf

支持  C++，Java，Python，Objective-C，C#，Ruby，Go，PHP，Dart，Javascript

https://zhuanlan.zhihu.com/p/432875529
protobuf详解
。。这个可能过时了，因为  现在是  proto3。

是谷歌内部的  混合语言  数据标准。
通过将结构化的数据  进行序列化，用于  通讯协议，数据存储  等领域的  与  语言无关，平台无关，可扩展的序列化数据结构格式。

protobuf  通常包括下面三点
1. 一种二进制  数据交换格式。可以将  C++  中定义的  存储类的  内容  与  二进制序列串  相互转换，主要用于  数据传输  或  保存
2. 定义了  一种源文件，扩展名为  .proto ,使用这种  源文件，可以定义  存储类的  内容。

3. protobuf  有自己的  编译器  protoc，  可以将  .proto  编译成  .cc  文件，  使之  成为一个可以在  C++工程中直接使用的类。

。。？这个是序列化  数据，   还是序列化  代码？  主要是  第三点，  编译  成代码了。。
。。cc文件是Linux/Unix下为C++源文件的默认扩展名
。。都可以。看下面  一行。

。。不，感觉不是，看下来，应该是，定义一种  中间格式，  这种格式可以  得到  C++源码。所以是：各个API先定义  中间格式.proto文件，然后  各自把  proto  转成  自己使用的语言。

。。但是这篇只介绍了  proto  的定义，  没有说  具体某个语言  调用什么方法  来  将数据  转成  二进制。

序列化：将数据结构  或  对象转换为  二进制串  的过程。
反序列化：将序列化过程中产生的  二进制串  转换成  数据结构  或  对象的  过程。

定义proto文件
message介绍

protobuf  中定义  一个  消息类型  是通过  关键字  message  字段指定的，  这个关键字  类似于  C++ Java  中的  class  关键字。

使用  protobuf  编译器  将  proto  编译成  C++代码后，  每个  message  都会生成  一个  名字  与之对应的  C++类，该类  公开继承自  google::protobuf::Message

message消息定义
创建  tutorial.person.proto  文件，内容如下：

// FileName: tutorial.person.proto
// 通常文件名建议命名格式为 包名.消息名.proto

// 表示正在使用proto2命令
syntax = "proto2";

//包声明，tutorial 也可以声明为二级类型。
//例如a.b，表示a类别下b子类别
package tutorial;

//编译器将生成一个名为person的类
//类的字段信息包括姓名name,编号id,邮箱email，
//以及电话号码phones
message Person {

  required string name = 1;  // (位置1)
  required int32 id = 2;
   optional string email = 3;  // (位置2)

  enum PhoneType {  //电话类型枚举值
    MOBILE = 0;  //手机号
    HOME = 1;    //家庭联系电话
    WORK = 2;    //工作联系电话
  }

  //电话号码phone消息体
  //组成包括号码number、电话类型 type
   message PhoneNumber {
    required string number = 1;
    optional PhoneType type =
          2 [default = HOME]; // (位置3)
  }

   repeated PhoneNumber phones = 4; // (位置4)
}

// 通讯录消息体，包括一个Person类的people
message AddressBook {
  repeated Person people = 1;

}

字段解释
包声明

proto文件以  package  声明开头，这有助于  防止不同项目之间  命名冲突。在C++中，以  package  声明的文件内容生成的  类将放在  与包名  匹配的  namespace  中。

上面的  proto文件中所有的  声明  都属于  tutorial

字段规则
required：消息体中  必填字段，不设置  会导致  编解码  异常。  如位置1
optional：可选字段，可以通过  default  关键字设置  默认值。如位置2

repeated：可重复字段，  重复的值  的顺序  会被  保留。  如位置4  。其中  proto3  默认使用  packed  方式  存储，这样编码  节省内存。

标识号
在消息体的定义中，每个字段都  必须有一个  唯一的  标识号，标识号  是  [0, 2^29-1]  内的一个整数

数据定义

许多标准的  简单数据类型  都可以用作  message  字段类型，包括  bool，  int32，  float，  double  和  string。  还可以  使用  其他  message  类型  作为  字段类型。

在上面的示例中，  Person  包含  PhoneNumber message，  而  AddressBook  包含  Person message。  甚至可以  定义  嵌套  在  其他  message  中的  message。  例如上，  的  PhoneNumber  定义在  Person  中。

函数方法

用message  关键字  声明的消息体，允许你  检查，操作，读，写  整个消息，包括  解析  二进制字符串，以及序列化  二进制字符串。除此之外，也定义了  下列方法：

Person:缺省的构造函数。

~Person():缺省的析构函数。

Person(const Person& other):拷贝构造函数。

Person& operator=(const Person& other):
赋值 (Assignment ）操作符。

const UnknownFieldSet& unknown_fields() const:
返回当解析信息时遇到的未知字段的集合。

UnknownFieldSet* mutable_unknown_fields():
返回当前解析信息时遇到的未知字段的集合的一个mutale指针。

编译proto文件
可以执行以下  protoc  命令  对  .proto  文件进行编译，生成对应  的  c文件。
Linux  通过  help protoc  查看  命令详情。
protoc -I=$SRC_DIR
--cpp_out=$DST_DIR  xxx.proto

$SRC_DIR:proto 所在的源目录
--cpp_out 生成C++代码
$DST_DIR 生成代码的目标目录
xxx.proto:要针对哪个proto文件生成接口，例如 tutorial.person.proto

编译完成后，  将生成  2个文件  tutorial.pb.h  和  tutorial.pb.c，  其中  tutorial  表示  包名，pb  是  protobuf  的缩写。

此外，protocol buffer  编译器  为  .proto文件中  定义的  消息的  每个字段  生成一套存取器方法：
对于message Person  中的  required int32  id  = 2，  编译器将生成  下列存取器方法：
bool has_id() const:  用于判断  字段  id  是否存在，  如果字段被设置，返回  true
int32  id() const:  返回字段  id  的当前值，  如果没有被设置，返回缺省值

void set_id(int32 value)：  设置字段id  的值，调用此方法后，  has_id()  将返回  true，id()  将返回  value

void clear_id()：清除字段的值，调用后，has_id  返回false，  id  返回缺省值

对于其他字段，也会生成  存取方法，这里不一一列举

使用message
//获取person实例：

tutorial::AddressBook addressBook;
Person *person =
addressBook.add_people();

//写入一个message:

person->set_id(id);
getline(cin,*person->mutable_name());

//读取一个message:

person.id();
person.name();
person.email();

扩展一个  message
你  不能  修改  任何现有字段的  字段编号
你可以删除  optional  或  repeated  属性的  字段
你可以添加  新的  optional  或  repeated  字段，但必须使用  新的标记。

protobuf  的优点
性能
序列化后，数据大小可以缩小  3  倍
序列化速度快
传输速度快

使用
使用简单：  proto编译器  自动  序列化  反序列化
维护成本低：多平台  只需要  维护  一套  对象协议文件，  即  .proto  文件
可扩展性好：不必破坏  旧  的数据格式，就能对  数据结构进行  更新
加密型好：http  传输  抓包  只能抓到  字节数据

使用范围
跨平台，跨语言，可扩展性强

--------------------------------

https://www.jianshu.com/p/a24c88c0526a
深入 ProtoBuf - 简介

官方文档给出的定义和简介
protocol buffers  是一种语言无关，平台无关，可扩展的序列化结构数据的方法，它可用于  (数据)通信协议，数据存储  等。

protocol buffers  是一种灵活，高效，自动化  机制的  结构数据序列化方法  -  可类比  xml，但是  比  xml  更小(3-10倍)，更快(20-100倍)，更简单。

你可以定义数据的结构，然后使用  特殊生成的  源代码  轻松的在  各种  数据流中  使用各种语言进行编写  和读取结构数据。  你甚至可以更新数据结构，而不破坏  由  旧数据结构  编译的  已部署程序。

使用Protobuf
1. 创建  .proto文件，定义数据结构
// 例1: 在 xxx.proto 文件中定义 Example1 message
message Example1 {
    optional string stringVal = 1;
    optional bytes bytesVal = 2;
    message EmbeddedMessage {
        int32 int32Val = 1;
        string stringVal = 2;
    }
    optional EmbeddedMessage embeddedExample1 = 3;
    repeated int32 repeatedInt32Val = 4;
    repeated string repeatedStringVal = 5;
}

在上面的例子中，定义了  一个名为  Example1  的消息，语法很简单，message  关键字后面  跟上  消息名称：
message xxx {

}
之后我们在其中  定义了  message  具有的字段，形式为：
message xxx {
  // 字段规则：required -> 字段只能也必须出现 1 次
  // 字段规则：optional -> 字段可出现 0 次或1次
  // 字段规则：repeated -> 字段可出现任意多次（包括 0）
  // 类型：int32、int64、sint32、sint64、string、32-bit ....
  // 字段编号：0 ~ 536870911（除去 19000 到 19999 之间的数字）
  字段规则 类型 名称 = 字段编号;
}
在例子中，我们定义了
类型string，名为  stringVal  的  optional  可选字段，字段编号为1，可以出现0或1次
类型bytes，名为  bytesVal  的optional  可选字段，字段编号2，出现0或1次
类型EmbeddedMessage (自定义的内嵌message类型)，名为  embeddedExample1  的  optional  字段，字段编号3
类型int32，  名为  repeatedInt32Val  的  repeated  字段，编号4
类型string，名为  repeatedStringVal  的  repeated  字段，编号5

1. protoc编译  .proto  文件生成  读写接口
我们在  .proto  文件中定义了  数据结构，这些  数据结构是  面向  开发者  和  业务程序的，并不面向  存储  和传输。

当需要把  这些数据进行  存储  或传输时，  就需要  将这些  结构数据  进行序列化，反序列化  以及  读写，应该如何实现？  Protobuf  将会为我们提供  相应的接口代码。如何提供？  是通过  protoc  这个编译器

可以通过如下命令生成相应  的接口代码
// $SRC_DIR: .proto 所在的源目录
// --cpp_out: 生成 c++ 代码
// $DST_DIR: 生成代码的目标目录
// xxx.proto: 要针对哪个 proto 文件生成接口代码

protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/xxx.proto

1. 调用接口实现  序列化，反序列化，读写
针对第一步中  定义的  message，我们可以调用  第二步  生成的接口，测试代码如下：

//
// Created by yue on 18-7-21.
//
#include <iostream>
#include <fstream>
#include <string>
#include "single_length_delimited_all.pb.h"

int main() {
    Example1 example1;
    example1.set_stringval("hello,world");
    example1.set_bytesval("are you ok?");

    Example1_EmbeddedMessage *embeddedExample2 = new Example1_EmbeddedMessage();

    embeddedExample2->set_int32val(1);
    embeddedExample2->set_stringval("embeddedInfo");
    example1.set_allocated_embeddedexample1(embeddedExample2);

    example1.add_repeatedint32val(2);
    example1.add_repeatedint32val(3);
    example1.add_repeatedstringval("repeated1");
    example1.add_repeatedstringval("repeated2");

    std::string filename = "single_length_delimited_all_example1_val_result";

    std::fstream output(filename, std::ios::out | std::ios::trunc | std::ios::binary);

    if (!example1.SerializeToOstream(&output)) {
        std::cerr << "Failed to write example1." << std::endl;
        exit(-1);
    }

    return 0;
}

关于  ProtoBuf  的一些思考
个人认为  如果要将  ProtoBuf，XML，JSON  三者放到一起去比较，应该  分为  2个维度。  一个是  数据结构化，一个是  数据序列化。
这里的数据结构化  主要面向  开发  或  业务层面，数据序列化  面向  通信或存储层面。这2者  的区别  主要是  面向领域  和  场景不同
数据结构化侧重人类  可读性  甚至  会强调  语义表达能力，  而数据序列化  侧重  效率  和  压缩。

从这2个维度，我们可以做出下面的一些思考
XML  作为  扩展标记语言，JSON作为  源于  JS的数据结构，都具有  数据结构化的能力。

例如，XML可以衍生出  HTML (虽然  HTML早于  XML，但从概念上讲，HTML只是  预定义标签的  XML)。HTML 的作用是标记和表达万维网中资源的结构，以便浏览器更好的展示万维网资源，同时也要尽可能保证其人类可读以便开发人员进行编辑，这就是面向业务或开发层面的数据结构化。

再如  XML  还可以衍生  出  RDF/RDFS，进一步  表达语义网中  资源的  关系  和语义，同样，它强调  数据结构化的  能力  和  人类可读。

JSON也是，在很多场合，更多的是  体现了  数据结构化的能力。

Protobuf  也具有  数据结构化的能力，  就是上面  介绍的  message  定义。  我们能在  .proto  文件中，通过  message，import，内嵌message  等语法  来实现  数据结构化，但是容易看出，Protobuf  在数据化  结构化方面  和  xml json  相差较大，人类可读性较差。

但是在  数据序列化  方面，Protobuf  有明显优势，效率，速度，空间  全面占优。

----------------------------------
https://www.jianshu.com/p/73c9ed3a4877

深入 ProtoBuf - 编码

让我们看看  Protobuf  如何  尽其所能  压榨编码性能  和  效率

编码结构
TLV格式  是我们比较熟悉的  编码格式

所谓的  TLV  就是  tag - length - value。  tag  是该字段的唯一标识，length  代表  value  数据域  的长度，value  就是  数据本身

Protobuf  编码  类似，但是  有区别，  其编码结构可见下图：
![](../_resources/665e46ee89a44130bf35d0d06b2d790d.png)

接下来  解析上图的  编码结构
首先，每一个message  进行编码，其结构由  一个个字段组成，每个字段  可以划分为  Tag - [length] - Value。

这里的  [length]  是可选的，即  对部分数据结构  编码结构可能  变成  Tag-Value。没有length  如何确定  value  的边界？  答案是  Variant  编码，  后面会介绍。

Tag  由  field_number  和  wire_type 2部分组成
field_number：message  定义字段时  指定的  字段编码
wire_type：Protobuf  编码类型，根据这个类型  选择不同的  Value  编码方案

整个tag  采用  Varints  编码方案进行编码。

3bit  的  wire_type  最多可以表达  8种  编码类型，  目前  Protobuf  已经定义了  6种

表格分为  3列，  第一列  是  wire_type  编号，第二列是  面向  最终编码的  编码类型，第三列是  面向开发者的  message  字段的  类型。

注意  Start group, End group  已经  废弃。

注意，虽然  wire_type  代表  编码类型，但是  Varint  这个编码类型  中  针对  sint32, sint64  会有一些特别  编码  (ZigZag  编码)  处理。相当于  Variant  这个编码类型  里又  存在  2种不同编码。

我们可以理解  一个message  编码将有  一个个的  field  组成，每个  field  根据类型  将有  如下  2种格式：
Tag - Length - Value :  编码类型表中  Type = 2  即  Length-delimited  编码类型  将使用这种结构
Tag - Value  ：  编码类型表中  Varint，  64-bit，  32-bit  使用这种结构。

其中  Tag  由  字段编号  field_number  和  编码类型  wire_type  组成，  Tag整体采用  Varints  编码。

Varints  编码
总的来说，Variants  编码规则  主要为  以下三点：
在每个字节开头  的  bit  设置了  msb(most significant bit),  标识  是否需要  继续  读取下一个字节
存储数字  对应的  二进制  补码
补码的  低位  排在前面

为什么低位排在前面？  这里主要是为  编码实现  (移位操作)  做的一些小优化。  可以尝试  写一个  二进制  移位  进行  编码解码  的例子  来体会这一点。

先看一个最为简单的例子
int32 val =  1;  // 设置一个 int32 的字段的值 val = 1; 这时编码的结果如下
原码：0000 ... 0000 0001  // 1 的原码表示
补码：0000 ... 0000 0001  // 1 的补码表示
Varints 编码：0#000 0001（0x01）  // 1 的 Varints 编码，其中第一个字节的 msb = 0

从  补码的末端开始  每7位  一组，并  反转排序。  因为  0000 .. 0000 0001  除了  第一个取出的  7位组  (即  原补码的  后7位)，  剩下的均为  0。  所以  只需要  取  第一个  7位组，无需再取  下一个  7bit，  那么第一个  7bit  组的  msb = 0。最终得到  0000 0001

解码过程

对于收到的  0000 0001，  首先  每个字节的  第一个bit  是  msb位，  msb 1表示  需要  再读一个字节，  0表示  无需再读字节。

在上面的例子中，  第一个bit  是0，  所以无需再读取  字节。去掉  msb  后，获得  000 0001，就是  补码的逆序，由于这里  只有一个  字节，所以  不需要反转，直接  解释补码  000 0001，就得到  数字  1。

这里，编码  数字1  ，Variants  只使用了  1个字节。  正常情况下  int32  要使用  4个字节。

在看一个  需要2个字节的数字  666  的编码
int32 val = 666; // 设置一个 int32 的字段的值 val = 666; 这时编码的结果如下
原码：000 ... 101 0011010  // 666 的源码
补码：000 ... 101 0011010  // 666 的补码
Varints 编码：1#0011010  0#000 0101 （9a 05）  // 666 的 Varints 编码

Variants  的本质实际上是  每个字节  都牺牲一个  bit  位  (msb)，来表示  是否已经结束  (是否  还需要  读取下一个字节)，msb实际上起到了  Length  的作用。  正因为有了  msb，  所以我们可以  摆脱原来的  那种  无论数字  大小  都必须  分配4  个字节的  窘境。  通过  Variants  我们可以  让  小的数字  用更少的字节表示。  从而  提高  空间  利用。

但是，当前  Variants  编码  存在  明显的  缺陷。  就是  负数。
int32 val = -1
原码：1000 ... 0001  // 注意这里是 8 个字节
补码：1111 ... 1111  // 注意这里是 8 个字节
再次复习 Varints 编码：对补码取 7 bit 一组，低位放在前面。
上述补码 8 个字节共 64 bit，可分 9 组且这 9 组均为 1，这 9 组的 msb 均为 1（因为还有最后一组）
最后剩下一个 bit 的 1，用 0 补齐作为最后一组放在最后，最后得到 Varints 编码
Varints 编码：1#1111111 ... 0#000 0001 （FF FF FF FF FF FF FF FF FF 01）

因为  负数必须在最高位  置1，  这意味着  负数  都必须占用  所有字节，  所以它的  补码  总是  占满  8个字节。  加上msb，  最终是  10个字节。

Protobuf  为了兼容，所以  int32  也是  int64  的8字节。

ZigZag编码

为了解决  Variants  编码  对负数  编码效率低的问题。  Protobuf  提供了  sint32，  sint64，当你使用这  2  种类型时，  ProtoBuf  将使用  ZigZag  编码

ZigZag  编码：  有符号整数  映射到  无符号整数，然后再使用  Variants  编码

|     |     |
| --- | --- |
| 带符号数 | 无符号数 |
| 0   | 0   |
| -1  | 1   |
| 1   | 2   |
| -2  | 3   |
| INT_MAX | unsigned int max - 1 |
| INT_MIN | unsigned int max |

这里的映射通过  移位实现。

。。。没有具体例子
。不过看了下，  非负数  应该是  *2  ，  即  << 1

-1  0000001    1111110   1111111      1
-2  0000010    1111101   1111110      3
-3  0000011    1111100   1111101      5

对于负数，  只想到  取绝对值，  *2，  -1

负数  取绝对值，好像可以   最高位  设置0，其余全部取反，  然后  + 1  。   。。  就是  补码  再  "补码"。
。。。。

继续Variant类型

之前提到  wire_type  已经有  4种，  2种已经废弃。  只剩下  4种：  Varint，64-bit，Length-delimited, 32-bit。

之前已经强调过，  Varint  类型不止一种  编码策略。  除了  int32, int64  的  Varint编码，还有  sint32,sint64  的  ZigZag  编码。

int32, int64, uint32, uint64, bool, enum
当使用  上述  类型声明  字段类型时，字段值  将使用  之前介绍的  Varint  编码。
其中  bool  本质  是  0  和  1，  enum本质是  整数常量。

结合之前介绍的  编码结构  Tag - [length] - Value，  这里  Value  使用  Varint  编码，因此不需要  Length，  编码结构为  Tag - Value，  其中  Tag，  Value  都是  Varint  编码。

。。这个  Varint  到底带不带  s  的。。应该是带的。

bool  类型设置为  false时，  编码结果是空。  默认值机制，当读出来的  字段为空时，就设置默认值。
。。但是  怎么知道  下一个  byte  是  值  还是  空？  难道  Tag  也有  msb？

sint32, sint64
将使用  ZigZag  编码，  结构依然是  Tag - Value

64-bit  和  32-bit  类型
和  Varints  一样  编码结构为  Tag - Value。  一个存储8字节，  一个存储4字节。

为什么需要  64-bit，32-bit？  因为  之前分析  Varints  编码  在一定范围内  是高效的，但是  超过某一个数字  占用字节反而  更多，效率更低。

如果现在的场景是  存在  大量的  大数字，那么  Varints  就不太适用了，适用  64-bit，32-bit更合适。
具体来说，如果数值  比  2^56  大，则64bit  比  uint64高效，如果比  2^28  大，则32bit  比  uint32  高效。

fixed64, sfixed64, double
//  设置字段值 为 -2
example1.set_fixed64val(1)
example1.set_sfixed64val(-1)
example1.set_doubleval(1.2)

// 编码结果，总是 8 个字节
09 # 01 00 00 00 00 00 00 00
11 # FF FF FF FF FF FF FF FF （没有 ZigZag 编码）
19 # 33 33 33 33 33 33 F3 3F

fixed32, sfixed32, float
和  上面的  相同。

Length - delimited  类型
string, bytes, EmbeddedMessage, repeated
这个类型的  编码结构  就是  Tag - Length - Value

proto定义
syntax = "proto3";

// message 定义
message Example1 {
    string stringVal = 1;
    bytes bytesVal = 2;
    message EmbeddedMessage {
        int32 int32Val = 1;
        string stringVal = 2;
    }
    EmbeddedMessage embeddedExample1 = 3;
    repeated int32 repeatedInt32Val = 4;
    repeated string repeatedStringVal = 5;
}

设置值
Example1 example1;
example1.set_stringval("hello,world");
example1.set_bytesval("are you ok?");

Example1_EmbeddedMessage *embeddedExample2 = new Example1_EmbeddedMessage();

embeddedExample2->set_int32val(1);
embeddedExample2->set_stringval("embeddedInfo");
example1.set_allocated_embeddedexample1(embeddedExample2);

example1.add_repeatedint32val(2);
example1.add_repeatedint32val(3);
example1.add_repeatedstringval("repeated1");
example1.add_repeatedstringval("repeated2");

最终编码
0A 0B 68 65 6C 6C 6F 2C 77 6F 72 6C 64
12 0B 61 72 65 20 79 6F 75 20 6F 6B 3F
1A 10 08 01 12 0C 65 6D 62 65 64 64 65 64 49 6E 66 6F
22 02 02 03[ proto3 默认 packed = true]（编码结果打包处理，见下一小节的介绍）

2A 09 72 65 70 65 61 74 65 64 31 2A 09 72 65 70 65 61 74 65 64 32(repeated string 为啥不进行默认 packed ?)

补充  packed  编码
在proto2中，为我们提供了  可选的  设置  [packed = true]，  这一选项  在  proto3  中  是默认设置。
packed  目前只能用于  primitive(原始)  类型

packed  = true  主要是让  ProtoBuf  为我们把  repeated primitive  的编码  结果打包，从而  进一步压缩空间。

这里打包的含义：  原先的  repeated  字段的  编码结构是  Tag-Length-Value-Tag-Length-Value..  因为这些  Tag  都是相同的  (同一字段)，  所以可以将这些  Value  打包，变成  Tag-Length-Value-Vlaue-Value..

-------------------------------
https://www.jianshu.com/p/62f0238beec8

深入 ProtoBuf - 序列化源码解析

------------------------------
https://www.jianshu.com/p/ddc1aaca3691
深入 ProtoBuf - 反射原理解析

-------------------------

https://www.jianshu.com/p/b33ca81b19b5
最后部分是  Protobuf  官方文档的翻译  的链接

-----------------------------------
https://zhuanlan.zhihu.com/p/435944782
浅谈protobuf

protobuf  的核心内容包括：
定义消息：消息的结构体，以  message  标记
定义接口：接口路径和参数，以  service  标记

通过protobuf  提供的机制，服务端  之间  只需要关注  接口方法名(service)  和  参数  (message)  即可通信。而不需要关注  链路协议  和  字段解析，  降低服务器端的设计开发成本。

。。感觉就是  Mule  的  raml  ？
。。而且  (应该算是)  支持  RPC了？  不是，下面第五步。  不，算是支持了，不过Protobuf不是RPC。

使用案例

假设有一个服务端程序  SSR，  希望对外提供一个  注册接口，调用方法传入  手机号，姓名，邮箱，其中  邮箱选填。  那么如何通过  Protobuf  来实现  接口定义呢？

第一步：  定义服务名  SSR  和  方法名  register
service SSR {
    rpc register(RegisterRequest) returns (RegisterResponse);
};

第二步：  定义接口参数  和约束
message RegisterRequest {
    required string name = 1; // 姓名
    required string tel = 2; // 手机号
    optional string email = 3; // 邮箱：可选
};
message RegisterResponse {
    required int code = 1; // 错误码
    optional string err_msg = 2; // 错误信息
};

第三步：使用  protoc生成  语言相关的  协议代码
// for golang
protoc -I . -I /usr/local/include -I $(GOPATH)/src --go_out=. ssr.proto

// for cpp
protoc -I . -I /usr/local/include --cpp_out=. ssr.proto

第四步，实现  Protobuf  定义的接口
class SsrImpl : public proto::SSR {
    void register(::google::protobuf::RpcController* controller,
                    const proto::RegisterRequest* request,
                    proto::RegisterResponse* response,
                    ::google::protobuf::Closure* done);
}

第五步，通过  RPC  框架  注册  Service  和  实现业务逻辑
// 比如baidu-rpc框架为例
auto ssr = new baidu::rpc::Server();
ssr->AddService(new SsrImpl(), baidu::rpc::SERVER_DOESNT_OWN_SERVICE);

Protobuf JSON flatbuffer

Protobuf
优势
类型安全
易用性好
序列化  反序列化  性能好
兼容性好
不仅可以定义  结构体，还可以定义  RPC  服务接口
劣势
可读性差：没有schema  的情况下，  难以阅读  和  编辑
灵活性较差：无法动态修改  schema

JSON
优势
可读性好
易用性好
灵活性好：支持动态修改  schema
劣势
序列化  反序列化  性能差
编码问题导致解析失败等

flatbuffer
优势
灵活性：支持灵活的schema
性能好：  序列化，反序列化  几乎无消耗，高效的内存使用  和  读取速度
占用内存小

劣势
可读性差
易用性差：  使用成本高，接口复杂

==========================

Apache Thrift

==========================

==========================

==========================

==========================

已使用 Microsoft OneNote 2016 创建。