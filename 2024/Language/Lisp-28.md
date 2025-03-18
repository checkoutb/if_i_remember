Lisp-28
2021年8月27日
11:00

已使用 Microsoft OneNote 2016 创建。

[[toc]]

---
---

# emacs lisp
2023-05-13 09:03

https://www.emacswiki.org/emacs/EmacsLisp

https://www.emacswiki.org/emacs/CommonLisp

https://www.emacswiki.org/emacs/EmacsLispLimitations

https://www.gnu.org/savannah-checkouts/gnu/emacs/tour/index.html

https://www.gnu.org/software/emacs/manual/html_node/emacs/index.html

https://www.gnu.org/software/emacs/manual/html_node/elisp/index.html


# common lisp

https://acl.readthedocs.io/en/latest/



---
---

# elisp

2024-05-13 20:15

https://www.gnu.org/software/emacs/manual/html_node/eintr/





---

# emacs lisp

2024.06.04 17:40

使用了 我的 github中的 html

`file:///mnt/239G/my_github/elisp-manual-cn/html/00_content.htm`




## ch1 简介

主要受Maclisp的启发，也有一点受Common Lisp的启发

为了减少GNU Emacs的内存需求，Common Lisp的许多特性被省略或简化。
有时，简化是如此剧烈，以至于Common Lisp用户可能会感到非常困惑。

通过 cl-lib 库可以获得一定数量的Common Lisp仿真。

Emacs Lisp完全不受Scheme的影响；但是GNU项目有一个方案的实现，叫做==Guile==。我们在所有要求可扩展性的新GNU软件中使用它。   
。。见过，但是 并没有流行。 不知道有什么作用。


。。应该加一个 下一页 的快捷键。。 done


### 1.3.2 nil 和 t

- 名称为 nil 的符号
- 逻辑假值
- 空列表

用作变量时， nil 始终具有 nil值

对于lisp 解释器而言，() 和 nil 是相同的。

本手册中，() 表达空列表， nil 表达 逻辑假。

任何非0值都是真值，但是 t 是 表达真值的 首选。

符号 t 始终具有值 t 。 

函数 booleanp object ， 如果 object 是 t 或 nil，则返回 非nil

---

Lisp 表达式称为 形式 ( form )   
两种 形式 ( form ) 的精确等价用 ≡ 表示。  
(make-sparse-keymap) ≡ (list 'keymap)  


### 1.3.4 运行(打印文本)

如果您通过在示例的右括号后键入 C-j 在 Lisp 交互缓冲区（例如缓冲区 *scratch*）中执行示例代码，则打印的文本将插入到缓冲区中。  
如果您通过其他方式执行示例（例如通过评估函数 eval-region ），则打印的文本将显示在回显区域中。 

。。。懂了。。
emacs 打开 scratch  复制 `(progn (prin1 'foo) (princ "\n") (prin1 'bar))`

。。这个似乎只能 单行运行，因为，在文件中，增加 ;; 注释后，Cx Ce报错了。

1. 然后 按 `C-j`  ，你会发现 在 光标位置 会有输出， 这个输出 就是 这个 scratch 的输出。

2. 也可以 `C-x, C-e` , 会在 下面的跳出一个输出框

在文件中 可以使用 第二种方法，  第一种无效。

如果kill掉 scratch，  使用 `M-x scratch-buffer` 重新打开
https://www.gnu.org/software/emacs/manual/html_node/emacs/Lisp-Interaction.html






## ch2 lisp数据类型

每个对象==至少属于一种类型==。  
类型可以重叠，对象可以属于两种或多种类型。  
我们可以询问对象==是否属于==特定类型，但==不能询问对象的类型==。   

内置了一些基本的对象类型。  
每个对象都属于==一种且仅一种原始类型==。  
integer、float、cons、symbol、string、vector、hash-table、subr、byte-code function 和 record，还有与编辑有关的几种特殊类型，如 缓冲区。  

每个原始类型都有一个相应的 Lisp 函数，用于检查对象是否是该类型的成员。

Lisp 变量可以具有任何类型的值，并且它会记住您存储在其中的任何值，类型和所有内容  
。。python 一样， 不需要声明类型， 可以指向任何类型

在 Lisp 中，表达式主要是 Lisp 对象，其次是作为对象读取语法的文本。



### 2.4.1 整数类型 fixnum, bignum

fixnum, 取决于机器， 至少 -2^29 - (2^29-1)

bignum 是 任意精度， 使得 fixnum 溢出的操作 会赶回 bignum

所有数字都可以用 eql 或 = 进行比较
fixnum 还可以使用 eq

要测试一个整数是 fixnum 还是 bignum，您可以将其与 most-negative-fixnum 和 most-positive-fixnum 进行比较，或者您可以在任何对象上使用便利谓词 fixnump 和 bignump。 



### 浮点数 double

浮点数表示方式
- 小数点，及 后面至少有一个数字  。。所以 `1.` 是整数
- 指数
- 上面2个的混合


### 字符, elisp中是整数

ascii 码

由于字符实际上是整数，因此字符的打印表示是十进制数。  
这也是字符的一种可能的读取语法   
您应该始终使用 Emacs Lisp 为字符提供的特殊读取语法格式。这些语法格式以问号开头。   

字母数字字符的通常读取语法是问号后跟字符； 因此， ?A 表示字符 A， ?B 表示字符 B， ?a 表示字符 a。 

。。`?Q` 输出 81

如果标点符号在 Lisp 中具有特殊的句法含义，则必须用 `\` 将其引用。  
例如，`?\(` 是左括号字符的书写方式。  
同样，如果字符是 `\`，则必须使用第二个 `\` 来引用它：`?\\`。 

```text
?\a ⇒ 7                 ; control-g, C-g
?\b ⇒ 8                 ; backspace, BS, C-h
?\t ⇒ 9                 ; tab, TAB, C-i
?\n ⇒ 10                ; newline, C-j
?\v ⇒ 11                ; vertical tab, C-k
?\f ⇒ 12                ; formfeed character, C-l
?\r ⇒ 13                ; carriage return, RET, C-m
?\e ⇒ 27                ; escape character, ESC, C-[
?\s ⇒ 32                ; space character, SPC
?\\ ⇒ 92                ; backslash character, \
?\d ⇒ 127               ; delete character, DEL
```

在没有特殊转义含义的任何字符之前允许使用反斜杠，并且无害； 因此，’?\+’ 等价于 ’?+’。


#### unicode

`?\NAME` 表示 NAME的 unicode字符

`?\N{U+E0}`, `?\u00e0` and `?\U000000E0` 都是 à 

`?\x41`  16进制表示， A

`?\101`  8进制 A ， 最多 777
。。`?\65` 输出是 十进制 53



#### 控制字符

`?\^I` ⇒ 9         。试了下 确实是9，但是 9 是 水平制表符啊。 控制字符 和普通字符 怎么区别？ 试了下， `?\^a` 是 1, `?\^z` 是 26  , 大小写相同。
`?\C-I` ⇒ 9

?\^? ⇒ 127     
?\C-? ⇒ 127


---

### 符号

符号是一个有名字的对象。

在普通的 Lisp 使用中，使用一个 obarray（请参阅创建和内部符号），一个符号的名称是唯一的——没有两个符号具有相同的名称。 


符号可以用作==变量、函数名或保存属性列表==。  
或者它可能仅用于与所有其他 Lisp 对象不同，以便可以可靠地识别它在数据结构中的存在。


名称以冒号 (`:`) 开头的符号称为关键字符号。  
这些符号自动充当常量

符号 的名字 不是数字 就可以。  或者 数字前面 加 \ ， 也可以作为  符号的名称。


符号 名称中的 \ 只是 简单地 引用 \ 后面的字符，如 \t 在 字符串中是 制表符， 但是 在 名字 中只是 t

common lisp 中，小写字母 总是 被 转换为 大写。 
elisp中， 大小写 敏感。


### 序列

一组有序元素的 Lisp 对象

列表 和 数组

列表是最常用的序列。列表可以包含任何类型的元素，并且可以通过添加或删除元素轻松更改其长度。

数组是固定长度的序列。它们进一步细分为字符串、向量、字符表和布尔向量。
- 向量可以包含任何类型的元素，而
- 字符串元素必须是字符，而
- 布尔向量元素必须是 t 或 nil。
- 字符表类似于向量，只是它们由任何有效的字符代码索引。


通常不可能两次读取相同的序列，因为序列总是在读取时重新创建。  
如果您将一个序列的读取语法阅读两次，您将得到两个内容相同的序列。有一个例外：空列表 () 总是代表同一个对象，nil。 


### 2.4.6 缺点单元格和列表类型 ==cons==

一个 cons 单元是一个由两个槽组成的对象，称为 ==CAR 槽和 CDR== 槽。每个插槽可以容纳任何 Lisp 对象

==列表是一系列 cons 单元==，它们链接在一起，以便每个 cons 单元的 CDR 槽保存下一个 cons 单元或空列表。

列表的读取语法和打印表示是相同的，并且由左括号、任意数量的元素和右括号组成。
```text
(A 2 "A")            ; A list of three elements.
()                   ; A list of no elements (the empty list).
nil                  ; A list of no elements (the empty list).
("A ()")             ; A list of one element: the string "A ()".
(A ())               ; A list of two elements: A and the empty list.
(A nil)              ; Equivalent to the previous.
((A B C))            ; A list of one element
                        ;   (which is a list of three elements).
```
。。scratch 中不行。 (1 2 3) 会报错  无法prin1

。。`(cdr '(1,2,3,4))`  `'` 是什么意思？


没有元素的列表是空列表； 它与符号 nil 相同。换句话说，nil 既是符号又是列表。


---

`(a.b)` 代表一个 cons单元，CAR 是对象 a， CDR 是对象 b。

点对符号 比 列表语法 更通用，因为CDR 不必是 列表。  
但是 在 列表语法可以工作的情况下，它更麻烦。 在点对符号中， 列表`(1 2 3)` 需要写为 `(1.(2.(3.nil)))`  
对于以 nil 结尾的列表，你可以使用 任意一种表示法， 但列表表示法 通常更清晰 方便。  
打印列表时， 仅当 cons 的 CDR 不是列表时 才使用 点对符号。


---

关联列表 或 alist 是一个 特殊结构的列表，其元素是 cons 单元格

在每个元素中，CAR 被认为是一个键，而 CDR 被认为是一个关联的值

关联列表通常用作堆栈，因为在列表的前面添加或删除关联很容易。 

```lisp
(setq alist-of-colors
        '((rose . red) (lily . white) (buttercup . yellow)))
```

### 数组类型

数组由任意数量的槽组成，用于保存或引用其他 Lisp 对象，排列在连续的内存块中

4种类型的数组
- 字符串，字符数组
- 向量，任意对象数组
- 布尔向量，只包含 t 或 nil
- 字符表， 由任何有效字符代码 索引的 稀疏数组，可以持有任意对象

第一个元素的 索引为0

数组在创建后，长度就固定

所有 emacs lisp 数组都是一维的。 二维数组 需要通过 嵌套 来实现。



### 字符串类型

"string"


```lisp
"is asd
int asdf,
but asdsd\
asf"
```
输出：
```text
"is asd
int asdf,
but asdsdasf"
```

。。py 的 """xxx"""


字符串中 非 ASCII 有2种文本表示方法
- 多字节
- 单字节

简单来说， 单字节存储 原始字节， 多字节存储人类可读的文本。


字符串中的文本属性

字符串还可以保存它所包含的字符的属性。这使得在字符串和缓冲区之间复制文本的程序无需特别努力即可复制文本的属性

`#("characters" property-data...)`

其中 property-data 由零个或多个元素组成，以三个为一组，`beg end plist`
元素 beg 和 end 是整数，它们共同指定字符串中的索引范围； plist 是该范围的属性列表  
`#("foo bar" 0 3 (face bold) 3 4 nil 4 7 (face italic))`

表示文本内容为 ’foo bar’ 的字符串，其中前三个字符具有值为粗体的面属性，后三个字符具有值为斜体的面属性。  
第四个字符没有文本属性，所以它的属性列表是 nil。实际上没有必要以 nil 作为属性列表来提及范围，因为任何范围内未提及的任何字符都将默认没有属性。


### 2.4.9 向量类型 ( 类似list)

任何类型元素的一维数组

。就是list，访问需要从头开始计算。

`[1 "two" (three)]`


### 2.4.10 char table type

任何类型的元素的 一维数组， 通过 字符进行索引

字符表可以有一个 要继承的父级，一个默认值 和 少量额外的 插槽 来用于 特殊用途。  
还可以为 整个字符集 指定单个值

字符表的打印就向一个 向量，只是在开头有一个 额外的 `#^`

字符表用途
- 案例表 case table  4.10 
- 字符类别表 character category table
- 显示表格 display table  40.22.2
- 语法表 syntax table  36



### bool vector

元素必须 t 或 nil

`(make-bool-vector 3 t)`

`(equal #&3"\377" #&3"\007")`
t

没说怎么修改。怎么获得 某个bit。难道不能修改的？


### hash table


`(make-hash-table)`
;; #s(hash-table size 65 test eql rehash-size 1.5 rehash-threshold 0.8125 data ())



### 2.4.13 功能(function)类型 (lambda)

函数也是 lisp对象

在运行时 构建 或 获取函数对象， 然后使用 funcall 和 apply 调用它


### 宏类型 marco

lisp宏是扩展lisp语言的 用户定义结构。它被表示为一个 与函数 非常相似的 对象， 但具有不同的参数传递语义。  

lisp宏 通常使用内置的 defmacro 宏定义。   
emacs 中任何 macro 开头的 都是 宏


### 原始函数类型 primirive function

是可以从 lisp 调用的 使用C 写的 函数

也称为 子函数 或 内置函数 (subr 源自 subroutine)

对于调用者， 函数是否是 原始函数 无关紧要

但是如果你 尝试 使用 lisp 编写的函数 重新定义原语，这很重要。 因为可以 直接从 C 调用 原始函数， 所以 重新定义原语后， 从lisp 调用的是 新的含义的原语， 从C调用的 可能依然是 内置的 定义。  
不提倡 重新定义 原始函数

原始函数 没有读取语法， 以散列表示法 打印子例程的名称

```lisp
(symbol-function 'car)
;; #<subr car>

(subrp (symbol-function 'car))
;; t
```

(symbol-function 'car)          ; Access the function cell
                                ;   of the symbol.
     ⇒ #<subr car>
(subrp (symbol-function 'car))  ; Is this a primitive function?
     ⇒ t                       ; Yes.


### 字节码函数类型 byte code function

通过 字节编译 lisp代码产生的。

在内部。字节码函数对象 很像一个 向量。 

但是当它出现在函数调用中时，求值器 会特别处理这种数据类型。

打印和读取 类似于 向量，在 开头的 `[` 之前有一个 #



### record 类型

很像一个向量。但是 第一个元素 用于 保存 由 type-of 返回的类型。

允许 程序员 创建 具有 未内置于 emacs 中的 新类型的 对象


### 2.4.18 类型描述符

保存有关类型信息的记录。
record 中的 slot 1 必须是一个 命名类型的符号， type-of 依靠这个 来返回 记录对象的类型。


### 自动加载类型

是一个列表，第一个元素是 符号自动加载。  
它存储为符号的函数定义，用作实际定义的占位符。  
autoload 对象表示真正的定义 位于lisp代码文件中，必要时 要加载该文件。

自动加载对象通常使用函数 autoload 创建，该函数将对象存储在符号的函数单元格中。


### 终结器类型

帮助lisp 在 不再需要对象 之后 进行 清理。

垃圾回收。



### 2.5 编辑类型 editing type

### 2.5.1 缓冲区类型

保存可编辑文本的对象

每个缓冲区 都有一个 称为 点 的指定位置， 大多数编辑命令作用于 点 附近的 当前缓冲区中的内容

许多标准的 emacs函数 操作或测试 当前缓冲区中的字符。

其他几个数据结构和 每个缓冲区想关联
- 本地语法表
- 本地键盘映射
- 缓冲区局部变量绑定列表
- 叠加
- 缓冲区中文本的文本属性


缓冲区可能是 间接的，这意味着它共享 另一个缓冲区的 文本，但 呈现方式不同。

缓冲区没有读取语法

```lisp
(current-buffer)
;; #<buffer 01_.el>
```


### 标记(marker)类型

标记 表示 特定缓冲区 中的位置。  
依次，标记有2个组成部分， 一个用于缓冲区，一个用于位置。

缓冲区文本中的 更改 会根据需要 自动重新定位位置值，以确保 标记始终指向 缓冲区中 相同的两个字符之间。

```lisp
(point-marker)
;; #<marker at 2406 in 01_.el>
```

### 2.5.3 窗口类型

窗口 描述了 emacs 用来显示 缓冲区 的屏幕部分。  
每个活动窗口 都有一个 关联的缓冲区。  
一个缓冲区 可能 不出现在窗口 或 出现在 一个窗口 或 出现在多个窗口 中。  
窗口在屏幕上被分组为 框架，每个窗口只属于一帧。

尽管可能存在多个窗口，但在任何时候 都会将一个 窗口 指定为 选定窗口。 这是 emacs 准备好执行命令时 显示光标的窗口。  
选定的窗口 通常 显示当前缓冲区 ，但不一定 如此。

windows 没有读取语法。

```lisp
(selected-window)
;; #<window 3 on 01_.el>
```


### 帧类型

框架是包含一个 或多个 Emacs 窗口的 屏幕区域，我们使用 术语 帧 来指代 Emacs 用来指代 屏幕区域的 lisp 对象。

帧没有读取语法

```lisp
(selected-frame)
;; #<frame 01_.el - GNU Emacs in 192.xxx>
```


### 终端类型

终端是 能够显示一个 或多个 Emacs帧的设备

没有读取语法

```lisp
(get-device-terminal nil)
;; #<terminal 1 on wayland-1>
```


### 窗口配置类型

在框架中 存储有关窗口位置，大小和 内容的信息。 因此，你可以稍后 重新创建相同的 窗口排列

没有读取语法。   

打印语法看起来想 `#<window-configuration>`


### 2.5.7 帧配置类型

框架配置中 存储有关所有框架中 窗口的位置，大小，内容的信息。  
它不是原始类型， 实际上是一个 列表， car 为帧配置， cdr 为 alist。 每个alist 元素描述一个 帧，该帧 显示为该元素的 car。


### 流程(process)类型

emacs lisp 中 process 是一个 lisp 对象，指定由 emacs进程 创建的 子进程。  
shell，gdb，ftp 和编译器 等程序 在 emacs的子进程中 运行，扩展 emacs的功能。  
emacs 子进程 从 emacs 获得 文本输入 并将 文本 输出 返回给 emacs 以供 进一步操作。 emcas 也可以向 子进程 发送信号

没有读取语法。

```lisp
(process-list)
;; #<process server>
```


### 线程类型

emacs中的 一个线程 代表 emacs lisp 执行的 一个 单独线程。 它运行 自己的 lisp 程序， 拥有自己的 当前 缓冲区，并且可以把 子进程 锁定到 它， 即 只有该线程 可以接收 其输出的 子进程。

没有读取语法

```lisp
(all-threads)
;; (#<thread 0x123123>)
```


### 2.5.10 互斥体类型

用于线程同步

没有读取语法

```lisp
(make-mutex "my-mutex")
;; #<mutex my-mutex>

(make-mutex)
;; #<mutex 0x123123>
```



### 条件变量

支持 比 互斥锁 更复杂的 线程同步

没有读取语法

```lisp
(make-condition-variable (make-mutex))
;; #<condvar 0x123123>
```


### 流类型

可以用作字符源 或接收器的 对象，即 可以为输入 提供字符，也可以 接受它们 作为输出。

许多不同的类型 都可以这样用： 标记，缓冲区，字符串，函数。

大多数情况下， 输入流(字符源) 从 键盘，缓冲区 或 文件 中获得字符，  输出流(字符接收器) 将 字符发送到 缓冲区 (如 help 缓冲区) 或 回显区域

对象 nil除了它的其他含义外，还可以用作流。  
它代表 变量标准输入 或标准输出的值。 此外，作为流的对象 t 指定使用 minibuffer 的输入 或回声区域中的输出。

流没有特殊的 打印表示 或读取语法，并且可以打印为 任何原始类型


### 2.5.13 键盘映射类型

键盘映射将用户键入的 键 映射到命令   
控制 如何执行用户的命令输入。   
键映射 实际上是一个 列表，car 是符号键映射。

### 覆盖(overlay)类型

指定应用于缓冲区的 一部分的属性。每个覆盖适用于 缓冲区的 指定范围，并包含一个 属性列表。   

覆盖属性 用于 临时以不同的显示样式 呈现缓冲区的 一部分。

没有读取语法

### 字体(Font)类型

指定如何在 图形终端上显示文本。实际上存在3种不同的字体类型：字体对象，字体规范，字体实体， 每种都有 略微不同的属性

没有读取语法

打印类似 `#<font-obect>` `#<font-spec>` `#<font-entity>`



### 2.6 循环对象的读语法

要表示 Lisp 对象复合体中的共享或循环结构，您可以使用阅读器构造 `#n=` 和 `#n#`。 


```lisp
'(#1=(a) b #1#)
;; ((a) b (a))  前面没有 ' 了。
;; 代码中不带 ' 就是 invalid function
```

在这里，第一个 和 第三个 是相同的lisp对象


```lisp
(proq1 nil
       (setq x '(#1=(a) b #1#)))
(eq (nth 0 x) (nth 2 x))
```
。本地不行。 void variable x 。估计是 运行方式的问题。感觉 我这里的 是一行一行运行的， 没有 上下文关系的 (可能 我这 只执行 最后一行)。
。。这个不知道怎么弄。 估计是 放到文件里，然后 执行一个 文件？  但 怎么弄？

`(setq x '((a) b (a)))`
这种，第一个，第三个 是不同的对象



### 2.7 类型谓词 (列举了几十个)

emacs lisp 解释器 本身不会 在调用函数时 对 传递给函数的 实参 进行 类型检查。  
它不能这样做，因为 lisp 中 形参没有声明 数据类型。  
因此，需要由单个函数来测试每个 实参是否 可用

所有内置函数 都会在 适当的时候 检查 实参类型，并在 参数类型 错误时 发出 错误信号。

`(+ 2 'a)`
```lisp
Debugger entered--Lisp error: (wrong-type-argument number-or-marker-p a)
  +(2 a)
  elisp--eval-last-sexp(nil)
  #<subr eval-last-sexp>(nil)
  apply(#<subr eval-last-sexp> nil)
  eval-last-sexp(nil)
  funcall-interactively(eval-last-sexp nil)
  command-execute(eval-last-sexp)
```

使用谓词 listp 来检查 列表， symbolp来检查符号的 例子


```lisp
(defun add-on (x)
  (cond ((symbolp x)
         (setq list (cons x list)))
        ((listp x)
         (setq list (append x list)))
        (t
         (error "invalid argument %s in add-on" x))))
```
。。调不来


预定义类型谓词
`file:///mnt/239G/my_github/elisp-manual-cn/html/02.7_%E7%B1%BB%E5%9E%8B%E8%B0%93%E8%AF%8D.html`

atom,arrayp,bignump,bool-vector-p,booleanp,bufferp,byte-code-function-p,case-table-p,char-or-string-p,
char-table-p,commandp,condition-variable-p,consp,custom-variable-p,fixnump,floatp,fontp,frame-configuration-p,
frame-live-p,framep,functionp,hash-table-p,integer-or-marker-p,integerp,keymapp,keywordp,listp,
markerp,mutexp,nlistp,number-or-marker-p,numberp,overlayp,processp,recordp,sequencep,string-or-null-p,
stringp,subrp,symbolp,syntax-table-p,threadp,vectorp,wholenump,window-configuration-p,window-live-p,windowp


检查对象类型的 最通用的方法是 `type-of`。


```lisp
(type-of 1)
;; integer

(type-of 'nil)
;; symbol

(type-of '())
;; symbol


(type-of '(x))
;; cons

(type-of (record 'foo))
;; foo
```


### 2.8 等式谓词

测试2个对象 是否相等。

`(eq 1 2)`

返回 t 或 nil

```lisp
(eq 1 1)
;; t

(eq 'fff 'fff)
;; t

(eq ?A ?A)
;; t

(eq 3.0 3.0)
;; nil . 书上是 nil or t，都可能

(eq (make-string 3 ?A) (make-string 3 ?A))
;; nil, 书上也是 nil。 这个 挺反常的

;(eq (make-string 3 "a") (make-string 3 "a"))
;; 语法错误。。 a, 'a' 也试过，不行 

(eq "asd" "asd")
;; nil, 书上是 nil or t

(eq "A" "A")
;; nil

(eq '(1(2(3))) '(1(2(3))))
;; nil

(setq foo '(1(2(3))))
(eq foo foo)
;; void variable foo

(eq [1 2 3] [1 2 3])
;; nil
```


make-symbol 返回一个 uninterned 符号， 与在 lisp 表达式 中写入名称时 使用的符号不同。具有 相同名称的 不同符号 不是 eq。
```lisp
(eq (make-symbol "foo") 'foo)
;; nil
```

emacs lisp 字节编译器 可能将 相同的文字对象 (例如 文字字符串) 折叠成 同一对象的 引用，  其效果是 字节编译的 代码 将此类 对象比较 为 eq， 而 同一个代码的 解释版本 则不会。  
。。编译为字节码 或 直接解释

如果 2个对象 eq， 那么 总是 equal 的 。 反之 并不正确

```lisp
(equal "A" "A")
;; t
```

字符串比较 区分大小写，但是 不考虑 文本属性

出于技术原因，当且仅当单字节字符串和多字节字符串包含相同的字符代码序列并且所有这些代码都在 0 到 127 (ASCII) 范围内时，它们才相等。 

equal 函数递归地比较对象的内容

相等是递归定义的 
因此，比较循环列表可能会导致导致错误的深度递归


equal-including-properties 和 equal 相同，除了 要求 字符串的 文本属性 也要相同

``` lisp
(equal-including-properties "A" (propertize "A" 'A t))
;; nil

(equal "A" (propertize "A" 'A t))
;; t
```



### 2.9 可变性

一些 lisp对象 永远不应该改变。  
例如 list 表达式 aaa 产生一个字符串， 但你 不应该 改变它的内容。

如果 可变对象 是 被评估的 表达式的一部分， 则它不再是可变的

如果试图 修改 不应改变的对象， 则 结果行为 是不确定的。 lisp 解释器 可能发出 错误信号，可能崩溃，或其他 不可预测行为

当类似的 常量作为 程序的一部分出现时，lisp 解释器可能会通过 重用 现有常量 或组件 来解释 时间或空间。
`(eq "ab" "ab")` 如果 解释器 只创建一个 实例，那么 返回t。





## ch 3 数字


































































































































































































































































