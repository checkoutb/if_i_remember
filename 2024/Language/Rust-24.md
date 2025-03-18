Rust-24

Rust-24
2021年8月27日
11:08

https://doc.rust-lang.org/book/

已使用 Microsoft OneNote 2016 创建。

2023-05-03 13:11

宏：
- https://doc.rust-lang.org/reference/macros-by-example.html
- https://veykril.github.io/tlborm/

std:
- https://doc.rust-lang.org/std/index.html      这个约等于cppreference的作用


lint:
- https://github.com/rust-lang/rust-clippy


vscode: rust-analyzer


https://doc.rust-lang.org/stable/reference/


- fn do1(c: String) {}：表示实参会将所有权传递给 c
- fn do2(c: &String) {}：表示实参的不可变引用（指针）传递给 c，实参需带 & 声明
- fn do3(c: &mut String) {}：表示实参可变引用（指针）传递给 c，实参需带 let mut 声明，且传入需带 &mut
- fn do4(mut c: String) {}：表示实参会将所有权传递给 c，且在函数体内 c 是可读可写的，实参无需 mut 声明
- fn do5(mut c: &mut String) {}：表示实参可变引用指向的值传递给 c，且 c 在函数体内部是可读可写的，实参需带 let mut 声明，且传入需带 &mut





---
---
---
---

[[toc]]



* * *

# cargo profile

https://doc.rust-lang.org/cargo/reference/profiles.html

。。这个是 cargo 的文档，还有许多其他的命令。

`help, version`

`bench, build, check, clean, doc, fetch, fix, run, rustc, rustdoc, test, report`

`add generate-lockfile locate-project metadata pkgid remove tree update vendor verify-project`

`init install new search uninstall`

`login owner package publish yank`

# install rust

中文版
https://kaisery.github.io/trpl-zh-cn/

https://doc.rust-lang.org/book/ch01-01-installation.html

We’ll download Rust through rustup, a command line tool for managing Rust versions and associated tools

`curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh`

。。会让你选择 默认模式，自定义模式，不安装。 我用的默认

```
/home/qwer/.rustup
/home/qwer/.cargo
/home/qwer/.cargo/bin
/home/qwer/.profile       # 下面是增加PATH
/home/qwer/.bash_profile
/home/qwer/.bashrc
```

。。安装了，stable-x86_64-unknown-linux-gnu installed - rustc 1.69.0 (84c898d65 2023-04-16)
。。看了下硬盘，少了一个g，看了下 .rustup, .cargo， 真有 1.2g，安装的时候下载速度很快啊。

`rustc --version`

## update uninstall

```
rustup update
rustup self uninstall
```

# hello world (rustc)

```Rust
fn main() {
    println!("Hello, world!");
}
```

```
[qwer@192 hello_world]$ rustc my_rust.rs 
[qwer@192 hello_world]$ ls
my_rust  my_rust.rs
[qwer@192 hello_world]$ ./my_rust
hello world!
```

The ==main== function is special: it is always the first code that runs in every executable Rust program.

`println!` calls a Rust macro. If it had called a function instead, it would be entered as println (without the !).

For now, you just need to know that using a ! means that you’re calling a macro instead of a normal function and that macros don’t always follow the same rules as functions.

# hello cargo (new)

Cargo is Rust’s build system and package manager

```
cargo --version
```

**create** a new `project` using Cargo

```
cargo new hello_cargo
cd hello_cargo
```

设置版本控制

```
cargo new --vcs=git
```

cargo 默认生成的 Cargo.toml

```
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

In Rust, packages of code are referred to as crates

## build and run cargo project (build, run, check)

```
[qwer@192 my_cargo]$ ls
Cargo.toml  src

[qwer@192 my_cargo]$ cargo build
   Compiling my_cargo v0.1.0 (/run/media/qwer/239G/rust/self/my_first_rust_projrct/my_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.56s

[qwer@192 my_cargo]$ ls
Cargo.lock  Cargo.toml  src  target

[qwer@192 my_cargo]$ ./target/debug/my_cargo
Hello, world!
```

`Cargo.lock`. This file keeps track of the exact versions of dependencies in your project.

编译/构建 \+ 运行

```
[qwer@192 my_cargo]$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/my_cargo`
Hello, world!
```

cargo check. This command quickly checks your code to make sure it compiles but doesn’t produce an executable:

```
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

cargo check 比 build 快很多， 所以你可以在 编写代码时，经常 check 来确保 代码可编译。

会在 target/release 中生成 可执行文件，而不是 target/debug。
这个模式下，代码运行速度更快，但是编译耗时长。

```
cargo build --release
```

# project: guess number

```rust
use std::io;

fn main() {
    println!("guess number");
    println!("plz input ur number");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess).expect("failed to read line");

    println!("you guessed: {guess}")
}
```

## In Rust, variables are immutable by default

```
let apples = 5; // immutable
let mut bananas = 5; // mutable
```

如果没有导入 std::io ， 可以直接 `std::io::stdin`

## '&' means reference (immutable)

For now, all you need to know is that, like variables, references are immutable by default.
stdin() 返回一个 Result 值，它是一个 枚举。

println

```
let x = 5;
let y = 10;
println!("x = {x} and y + 2 = {}", y + 2);
```

## generate random number

### https://crates.io/

Cargo.toml

```
[dependencies]
rand = "0.8.5"
```

The specifier 0.8.5 is actually shorthand for ^0.8.5, which means any version that is at least 0.8.5 but below 0.9.0.

如果没有修改代码，cargo是不会重新编译的，修改了代码，会重新编译，但是toml中依赖没有改的话，不会重新下载依赖

Cargo.lock，里面包含了所有依赖的具体版本，这样，任何时候，你都可以重新build，不会由于 依赖的版本的变化 而build 失败。

更新依赖时，使用下面的语句，可以无视 Cargo.lock，把toml中的依赖更新到 最新版本，然后写入到 Cargo.lock 中。
并且 update的时候，还是 和 build时一样，不会 超过 0.9.0

```
cargo update
```

### cargo doc --open (666)

可以把所有依赖的文档组合成一个文档，浏览器打开。
不过看了下体积有点大。就一个 猜数字，target 30mb+， doc 40mb+

### (形似)修改变量类型

```rust
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

。。有点猛，这个直接改 guess 的类型了。
。。好像不是修改，隐藏了前面的guess。
。。不过也 修改也差不多， 不知道 后续能 访问到 string 类型的 guess 吗？
Rust allows us to shadow the previous value of guess with a new one

for now, know that this feature is often used when you want to convert a value from one type to another type.

。。不太清楚，这里的 parse，是根据 u32 推导的，还是什么呢？

```
plz input ur number
6575675765675675576
thread 'main' panicked at 'plz input NUMBER: ParseIntError { kind: PosOverflow }', src/main.rs:22:43
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

### loop

```rust
use std::io;
use rand::Rng;
use std::cmp::Ordering;

fn main() {
    println!("guess number");
    let randnum = rand::thread_rng().gen_range(1..=100);
    println!("rand num is {randnum}");
    loop {
        println!("plz input ur number");
        let mut guess = String::new();
        io::stdin().read_line(&mut guess).expect("failed to read line");
        // let guess: u64 = guess.trim().parse().expect("plz input NUMBER");
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
        println!("you guessed: {guess}");
        match guess.cmp(&randnum) {
            Ordering::Less => println!("too small"),
            Ordering::Greater => println!("too big"),
            Ordering::Equal => {
                println!("you win");
                break;
            }
        }
    }
}
```

# variables and mutability

## 变量默认不可变。为了安全 和 更容易并发。

## const

const 变量名需要全大写，不然是 warning

```
const CST: u32 = 123;
```

# Data Types

https://doc.rust-lang.org/book/ch03-02-data-types.html

2个数据类型子集：scalar，compound (标量，复合物)

rust 是静态类型语言，即，必须在编译时知道 所有变量的类型。
编译器可以 根据 value and how we use it 推导出 我们需要的类型。

如果有多种可能时，我们需要增加 type annotation :

```
let guess: u32 = "42".parse().expect("not a number");
```

如果没有 : u32 这个 type annotation， rust 会报错。

## Scalar Types

代表一个 single value。
有4种： integers, floating-point number, booleans, characters。

### Integer Types

| Length | Signed | Unsigned |
| --- | --- | --- |
| 8-bit | i8  | u8  |
| 16-bit | i16 | u16 |
| 32-bit | i32 | u32 |
| 64-bit | i64 | u64 |
| 128-bit | i128 | u128 |
| arch | isize | usize |

最后一个 arch 是根据架构不同而不同，如果你是 32位，那么就是32bit。

| Number literals | Example |
| --- | --- |
| Decimal | 98_222 |
| Hex | 0xff |
| Octal | 0o77 |
| Binary | 0b1111_0000 |
| Byte (u8 only) | b'A' |

Rust 默认 i32

isize 和 usize的主要场景是： indexing some sort of collection。

integer overflow
debug模式时，integer overflow 会抛出异常。
release模式时，溢出时 执行 tow's complement wrapping。 u8类型时，256变成0,257变成1
。。感觉是，直接截断。和cpp的类似。

。。给了几个方法 用来 控制 overflow，但是一大段，不太理解，先把方法抄下来。
wrapping_* (wrapping_add)
checked_*
overflowing_*
saturating_*

### Floating-Point Types

浮点数有2个 原始类型， f32, f64
默认 f64， 因为在 modern CPU上，它和 f32 的速度一样，但是精度更高。

```
fn main() {
    let x = 2.0; // f64
    let y: f32 = 3.0; // f32
}
```

### Numeric Operations

支持 \+ \- \* / %

```
// division
    let quotient = 56.7 / 32.2;
    let truncated = -5 / 3; // Results in -1
```

### bool

2个值， true， false
占用 1 byte ( 8 bit)

```
fn main() {
    let t = true;
    let f: bool = false; // with explicit type annotation
}
```

### char

```
fn main() {
    let c = 'z';
    let z: char = 'ℤ'; // with explicit type annotation
    let heart_eyed_cat = '😻';
}
```

4个byte，可以表达 Unicode。

## Compound Types (复合类型)(2个 tuple, array)

compound type 将多个 value 组成一个 type。
有2个主要的 compound type： tuple ， array

### tuple

将==不同类型==的值组合成一个类型。

固定长度：一旦声明，不能再伸缩
。。不过rust本来就可以 为变量 赋予一个新的type，所以 近似伸缩了啊。

```
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

```
    let tup = (500, 6.4, 1);
    let (x, y, z) = tup;
```

```
    let x: (i32, f64, u8) = (500, 6.4, 1);
    let five_hundred = x.0;
    let six_point_four = x.1;
    let one = x.2;
```

没有任何值 的 tuple 被称为 unit， 这个值 和它的类型 都是 written()，且 代表了 空的值 或 空的return type。表达式 如果没有任何return的值，那么就 return unit。

### Array

和 tuple的不同是： array 中的每个元素 都是 相同类型。
array 是固定长度的。

```
fn main() {
    let a = [1, 2, 3, 4, 5];
}
```

标准库提供了 vector，和array类似，但是size可伸缩。

```
let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];

// 声明了类型，size
let a: [i32; 5] = [1, 2, 3, 4, 5];

// value : repeat count
// 等价于 let a = [3, 3, 3, 3, 3];
let a = [3; 5];
```

```
    let a = [1, 2, 3, 4, 5];
    let first = a[0];
    let second = a[1];
```

数组下标越界时会抛出运行时异常

# Functions

## snake case

rust 使用 snake case 作为 function 和 variable 的命名约束： 全小写，word之间用 下划线。

rust 中方法的定义 不需要 c++的前后关系。即 前面的方法可以调用后面的，而不必 提前声明后面的方法。

## parameter

实参被称为 argument

```rust
fn main() {
    another_function(5);
}

fn another_function(x: i32) {
    println!("The value of x is: {x}");
}
```

函数的形参的类型是必须的

```
fn main() {
    print_labeled_measurement(5, 'h');
}

fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {value}{unit_label}");
}
```

## statement, expression

方法的body 是由 一系列的 statement组成，并且 (可选地) 以 一个 expression 作为结尾。

**statement：执行一些动作，不返回 值 的 指令
expression：推导出一个 值**

let y = 6; 是一个 statement，没有返回值，所以不能： let x = (let y = 6);

### ==expression不带;;;;;;;;;;;;;;;;;;;;;;;;==

带上就是 statement

expression

```
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {y}");
}
```

## 带返回的 function

```
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {x}");
}
```

# Comments 只有一种：// 单行注释

```
// So we’re doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what’s going on.
```

# Control Flow

## if - else if - else

if 的条件 必须是 bool，不能是整数

。。不知道 条件加个 括号 行不行。。 就是 if (number < 5) 。。应该可以吧， () 当做 显式的优先级

```
    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
```

```
    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
```

### let 和 if (三目运算符)

变量的类型通过 第一个 {} 中的值推导，所以 2个 {} 里的值的类型 必须一样。

```
    let condition = true;
    let number = if condition { 5 } else { 6 };
```

## 循环：loop, while, for

### loop

endless loop

```
fn main() {
    loop {
        println!("again!");
    }
}
```

#### 从loop中返回值

。。break 带返回值。 continue 可以带值？ rust 有return吗？

```
    let mut counter = 0;
    let result = loop {     // 20
        counter += 1;
        if counter == 10 {
            break counter * 2;
        }
    };
```

#### loop label， 用于消除嵌套loop的歧义

break 和 continue 应用于包含 它们的 最内层的loop
你可以通过 loop label 来将 break 和 continue 应用到 指定的外层 loop

**Loop labels must begin with a single quote.**
。。一个单引号

```
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }
        count += 1;
    }
    println!("End count = {count}");
}
```

### while

```
    let mut number = 3;
    while number != 0 {
        println!("{number}!");
        number -= 1;
    }
```

### for (只能 for-each)

```
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {element}");
    }
```

下标需要使用 while

```
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;
    while index < 5 {
        println!("the value is: {}", a[index]);
        index += 1;
    }
```

。。(1..4) 是一个数组？

```
    for number in (1..4).rev() {
        println!("{number}!");
    }
```

# Ownership

最独特的功能，使得 rust可以 保证 内存安全 ，而不需要 gc。

Ownership 是 规则的集合，影响了 rust 的 内存管理。

内存的管理 有3种

- gc
- 程序员申请和释放内存
- rust 设置了一系列的规则，compiler 会检查这些规则。

编译时检查，所以不会对 运行 有性能影响。

。。那这些规则岂不是 所有语言都可以。 只要 静态编译。 不一定，可能有些规则 和 语言的特性 冲突。只能砍掉 某些特性。 也是可以的，自我限制下。

很多语言并不需要 思考 堆区 和 栈区。
但 rust 不行， 值 保存在 堆区 还是 栈区， 会影响 语言的 行为。

==11111111111111111111==
==11111111111111111111==
==11111111111111111111==
**内存中，栈区是 FILO， 无法 从 stack 的 中间 / 底部 进行 add，remove。 stack 中的所有数据 必须有一个 固定的，已知的 size。**

**编译时 size不可知 或 size会改变的 的 数据 必须到 heap 中。**

**堆的限制很少，当你 存在数据到 heap时， 你 申请 一块内存，memory allocator 会找到一块足够大的 空闲 内存，标记为 使用中，返回 一个 指针。 (指向heap的) 指针 是 已知的，固定的 size，你可以存储 到 stack 中，当你需要 真正的 数据时， 你必须 反引用指针。**

**往stack中 push 比 在heap中 allocate 更快，因为 stack 不需要搜索 可以用于保存数据的 位置，stack 的push位置 总是在 top。**

**访问 heap 的数据 比 访问 stack的慢，因为 你需要通过 指针才能找到 heap中的数据**

内存跳跃次数越少，那么处理器越快。
在一个table中获得所有order 是最高效的。
从 table A 中获得 一个order， 然后 从 table B 中获得一个order，然后从 A 中获得，然后从 B 中获得，这种 处理慢很多。
在 数据较为 集中的情况下，处理器工作得更好。

当你调用一个方法时，值被传递到方法中，方法的local 变量被压入栈。当方法结束时，这些值会出栈。
。。。但是我还是不知道，处理语句的时候，难道真的要清空栈，才能获得实参？

**追踪 使用heap 中数据的代码，最小化 heap 中重复对象，清理heap中不使用的数据**

## Ownership rules

记住以下rule

- 每个value 都有一个 owner
- 任意一个时间点，只能有一个 owner
- 当 owner 离开 scope 时，value会被 drop

。。最后一个。。感觉就是 unique_ptr 。

### variable scope

作为 ownership 的第一个例子，我们来看看 变量的 scope，scope 是 代码中的一段范围，在这个范围内，item 是有效的。

变量 从 声明的地方开始 有效，直到 当前 scope 的结束。
。。。？ RAII？？？？

```
    {       // s is not valid here, it’s not yet declared
        let s = "hello";   // s is valid from this point forward
        // do stuff with s
    }       // this scope is now over, and s is no longer valid
```

#### String Type

为了说明 ownership 的规则，我们需要一个 复杂的 数据类型。
之前遇到的类型 都是 一个已知的size，能被存储到栈中，并在scope结束时pop出来，能方便地copy生成一个新的，代码在不同的scope中使用相同的value时，这些实例是独立的。

当我们想要观察 存储到 heap 的数据，解释 rust 怎么知道 什么时候可以清理数据，String 是一个非常好的例子。

Rust 的String 类型，管理着 从heap allocate的数据，可以存储 编译时 未知的 文本。

你可以使用 from 方法 从 string字面量生成 String。

```
let s = String::from("hello");
```

。。不需要头文件。

双冒号 操作符 允许 我们将 这个 from 方法 声明为 String类型 下的方法，而不是使用一些其他的名字(比如 string_from )。

```
    let mut s = String::from("hello");

    s.push_str(", world!"); // push_str() appends a literal to a String

    println!("{}", s); // This will print `hello, world!`
```

这里 s 是 mutable的， 字面量不行。

### memory and allocation

字面量： 编译时知道内容。直接hardcode 到 最终的可执行文件中。这就是 为什么 字面量 快 和 高效。 但是 字面量是 不可变的。

String：为了支持 可变的 文本，我们需要 在 heap 上 allocate 内存，这个是 在编译时 未知的，来存放内容。

- 必须在 运行时 通过 内存allocator 来申请内存
- 当我们处理完 String后，需要通过 一种方式来将 内存 返还给 allocator。

第一部分，由我们 通过 String::from 来完成，这个方法的实现 会申请所需的内存。这种在所有 编程语言都有。

第二部分就不同了，对于有gc的语言，gc追踪和清理内存。对于没有gc的语言，开发人员负责 识别出 不再使用的 内存，并调用代码 free 它，Doing this correctly has historically been a difficult programming problem. If we forget, we’ll waste memory. If we do it too early, we’ll have an invalid variable. If we do it twice, that’s a bug too. We need to pair exactly one allocate with exactly one free.

Rust 使用了不同的方法：==the memory is automatically returned once the variable that owns it goes out of scope==

当变量 出scope时，释放内存，是非常自然的。
当变量出 scope时， Rust 调用 特殊的方法， 这个方法 称为 ==drop==，来归还内存。
Rust 在 最经的 `}` 处 自动调用 drop

Note: In C++, this pattern of deallocating resources at the end of an item’s lifetime is sometimes called Resource Acquisition Is Initialization (RAII). The drop function in Rust will be familiar to you if you’ve used RAII patterns.

#### 变量和 move

不同的变量可以有相同的数据

```
    let x = 5;
    let y = x;
```

我们可能会这样猜测： 绑定 value 5 到 x，然后 copy x 并绑定到 y。
确实是这样的，因为 整数 是简单类型，有着固定的 size，这2个5 都被 push 到 stack中。

```
    let s1 = String::from("hello");
    let s2 = s1;
```

这个和之前的不同。

String 由 3部分 组成： 一个指向内存的 指针，长度，容量。这些数据被保存在 stack，字符串的内存保存在heap中。

![afbefaa892c9f74057c09611db82f3ae.png](resources/179980edb697483d8a32f262d3fafbab.png)

当我们把 s1 赋给 s2 时， String 的数据被复制了，即 我们复制了 stack上的 指针，长度，容量 这些信息。 ==我们没有复制heap的数据==

![5f34ee8ba209b43956b13f65872ebfb9.png](resources/b568aac0fa9841869f1bf69371532048.png)

之前 我们说过，出 scope时， rust 自动调用 drop 方法来 清理 heap。
但是上图可以看到 2个 指针 指向了同一个地值。 就有一个问题： 当 s1 和 s2 出 scope 时，它们 都会 尝试 释放 同一块内存， 这会导致 double free 问题。

==为了确保安全，在 `let s2 = s1;` 后， rust 认为 s1 不再有效==， 因此，在 s1 出 scope时， 不会 free 任何东西。

```
    let s1 = String::from("hello");
    let s2 = s1;
    println!("{}, world!", s1);
```

==这段代码 会编译失败， 因为 rust 不允许你使用 无效的变量。==

Rust 的 赋予， 就像是 浅拷贝。 但是 Rust 使得 第一个变量 无效了， 所以 可以认为是 move。
在这个例子中，我们说： ==s1 被 move 到 s2 中==。

这样就解决了我们的问题，只有 s2 有效，所以 s2 出scope 时，会 free 内存。

这里也暗示了一个设计：==Rust 不会自动 创建 深拷贝==。 所以 所有自动的 copy 对于 性能的损耗 非常小。

#### 变量和 clone

如果我们想要深拷贝，我们可以使用 一个 普通方法： ==clone==。

```
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
```

当你看到 clone时，有一些 (可能是昂贵的) 代码被执行了。

### Stack-Only Data: Copy

下面的代码是 有效的。但是 我们没有调用 clone

```
    let x = 5;
    let y = x;

    println!("x = {}, y = {}", x, y);
```

因为， 在 编译时 就知道 确切 size 的类型 (比如 整数) 是 全部被存储在 stack 上的， 所以 拷贝 值 是非常快的。 我们没有理由 在 创建 y 以后 让 x 变得 无效。
在这里，深拷贝 和 浅拷贝 是一样的。 所以 调用 clone 完成的 动作 和 浅拷贝 一样，所以 不必调用 clone()。

。。确实，let y = x.clone(); 真的可以。。

rust 有一个 特殊的 annotation， 称为 Copy trait， 我们可以 把它放置到 可以存储在 stack 中的 type 上。
==如果 一个类型 实现了 Copy trait，它的变量 就不会 move，只会 复制， 使得它们 在 赋予 另一个变量后 依然 有效==

Rust 不允许 我们 annotation Copy 到 那些 实现了 Drop trait，或 它的任意一部分 实现了Drop trait 的 类上。 会是一个 编译时错误。

什么Type 实现了 Copy trait？ 你可以查看 type 的 文档 来确定。
通常来说，包含 简单标量值的 type 可以 实现 Copy，不需要allocation 的type 可以实现 Copy
下面是一些实现了 Copy的类型：

- 所有 整数类型
- bool
- 所有浮点数类型
- char
- 只包含 可以实现 Copy 的类型的 tuple，例如 (i32, u32)。 (i32, String) 不可以Copy

## Ownership and Functions

传递 `值` 到 方法的 机制 与 将value 赋予变量 类型。
传递 `变量`到 方法，会 move 或 copy，就像 赋予 一样。

```
fn main() {
    let s = String::from("hello");  // s comes into scope
    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here
    let x = 5;                      // x comes into scope
    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

如果我们像在 takes_ownership 之后 使用 s， Rust 会抛出 编译期异常。

## Return Values and Scopes

```
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1
    let s2 = String::from("hello");     // s2 comes into scope
    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("yours"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```

如果我们在 函数调用后，还想继续使用 变量，那么：

```
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}
```

。。 6， ==多值返回==

* * *

# References and Borrowing

reference 类似于指针，它保存一个地址。 但是 数据的 owner 是其他 变量。
不同于指针的是， ref 保证 在它的生命周期内 指向 某个类型的 有效的 value。

```
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

![b848b56e012c59e70f38ebd258cf11e3.png](resources/aa81801cea744c1fb5e9aacd7db32b7f.png)

对于 & 的 反引用 是 *

我们称 这种 创建ref 的 动作 是 borrowing，因为不是 owner，只是借，用完以后会还的。

如果 尝试修改 借来的 数据，会发生什么？

```
fn main() {
    let s = String::from("hello");
    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

会报错：cannot borrow `*some_string` as mutable, as it is behind a `&` reference

就像 变量 默认是 不可变的， ref 也是。

## Mutable ref

```
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

将 s 修改为 mut， 并且 将 ref 也改成 mut

mut ref 有一个限制： ==如果 对一个value 存在一个 mut ref，那么 不能再有 任何 其他的 ref 指向这个 value。==

```
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s;  // error[E0499]: cannot borrow `s` as mutable more than once at a time

    println!("{}, {}", r1, r2);
```

编译时 就可以检查到 多个 mut ref 。
防止了 ==数据竞争==。 数据竞争 类似于 竞态条件 ，当下面的 行为 发生时， 会发生 数据竞争

- 在某个时间点，有至少2个指针 可以访问到 同一个数据
- 至少有一个 指针 用来写入 数据
- 没有机制 来同步数据

我们可以通过 {} 来创建新scope，允许 多个 mut ref，但是 它们不是 同时发生的。

```
    let mut s = String::from("hello");

    {
        let r1 = &mut s;
    } // r1 goes out of scope here, so we can make a new reference with no problems.

    let r2 = &mut s;
```

==如果有 immut ref， 且 申请完 mut ref 后使用 immut ref， 会编译错误==

```
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
```

```
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    let r3 = &mut s; // BIG PROBLEM

    println!("{}, {}, and {}", r1, r2, r3);
```

==ref的 scope 是 从 引入ref的地方开始，知道 最后一次使用ref 的地方==
所以 有 immut ref， 申请 mut ref，后续不使用 immut ref 就不会 有编译问题。

```
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!("{} and {}", r1, r2);
    // variables r1 and r2 will not be used after this point

    let r3 = &mut s; // no problem
    println!("{}", r3);
```

## Dangling References 悬垂ref

在有指针的 编程语言中， 很容易就可以创建一个 dangling pointer， 指针指向了 已经被free的内存地址。

在Rust 中，编译器保证 ref 不会变成 悬垂 ref： 当你有一个 ref 指向一些数据，编译器保证： 数据不会在 ref离开scope 之前 离开 scope。

```
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("hello");

    &s
}
```

```
error[E0106]: missing lifetime specifier
 --> src/main.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected named lifetime parameter
```

因为 s 是local variable，出了方法 就drop了，此时 &s 就是一个 悬垂ref，是非常危险的。
要返回 local variable， 就直接返回 String， 而不是 ref

```
fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```

## The Rules of References

- 任何时候，你要么 持有一个mut ref, 要么持有多个 immut ref。
- ref 永远都是有效的。

# The Slice Type

Slice 可以 让你 ref 集合中的一段连续的元素，而不需要 ref 整个集合。
Slice 是 ref 的一种，所以它也没有 ownership。

场景： 编写一个方法，接受String，然后 返回 第一个单词。

```
fn first_word(s: &String) -> ?
```

应该返回什么？

我们可以返回一个 下标值，表示 单词的结束位置 ( 即空格的下标)：

```
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }
    s.len()
}
```

我们要遍历 String 元素，判断是否是 空格，所以 将 String 转为 byte 数组

```
let bytes = s.as_bytes();
```

然后创建一个 iterator 遍历 数组

```
for (i, &item) in bytes.iter().enumerate() {
```

iter 方法 返回 集合中的 每个元素，
enumerate 方法 封装 iter 的 结果，将 每个元素 作为 tuple 的一部分。 tuple 的第一个元素是 下标，第二个元素是 元素的ref。
由于 enumerate 返回的是 tuple，所以 我们可以使用 pattern 来 解构tuple。所以 i 是 下标，&item 是 byte。 由于 enumerate 返回的 第二个元素 是 ref ，所以 也使用 ref。

在 loop 内，通过使用 byte 字面量语法 获得 空格的byte值。

还有一个问题，我们返回了 usize，它是 依赖于 &String 的：

```
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // word will get the value 5

    s.clear(); // this empties the String, making it equal to ""

    // word still has the value 5 here, but there's no more string that
    // we could meaningfully use the value 5 with. word is now totally invalid!
}
```

当我们想要 第二个单词时，就需要返回 startIndex, endIndex 。。

```
fn second_word(s: &String) -> (usize, usize) {
```

## String Slice (&str)

string slice 是一个 ref 指向 String 的一部分：

```
    let s = String::from("hello world");
    let hello = &s[0..5];
    let world = &s[6..11];
```

。这个是 `[ , )` 区间

![e1213646c36bd20f3cffc77dfe02b9ee.png](resources/54e15ecf2b7c465bb675b559eb8b506c.png)

使用 Rust 的 `..` 这个 range语法。
如果从下标0开始， 那么可以 省略0 ， 下面是等价的：

```
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

同样，如果你 包含了 String的 最后一个byte，你也可以不写：

```
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

包含全部，也可以：

```
let slice = &s[0..len];
let slice = &s[..];
```

strnig slice 必须以 有效的 UTF-8 字符作为边界的划分。

==string slice 的 type 是 &str==

重写 first_word :

```
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}
```

```
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
```

```
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // error!

    println!("the first word is: {}", word);
}
```

println 的时候，需要 word， 所以 immut ref 还是生效的。
而 clear 需要修改，所以 需要 mut ref， 但此时 immut ref 还存在，所以报错了。

### String Literals as Slices

```
let s = "Hello, world!";
```

```
s 的type 是 &str
```

The type of s here is &str: it’s a slice pointing to that specific point of the binary. This is also why string literals are immutable; &str is an immutable reference.

### String Slices as Parameters

```
fn first_word(s: &str) -> &str {
```

只需要 实现上面的方法，就可以如下使用。

```
fn main() {
    let my_string = String::from("hello world");

    // `first_word` works on slices of `String`s, whether partial or whole
    let word = first_word(&my_string[0..6]);
    let word = first_word(&my_string[..]);
    // `first_word` also works on references to `String`s, which are equivalent
    // to whole slices of `String`s
    let word = first_word(&my_string);

    let my_string_literal = "hello world";

    // `first_word` works on slices of string literals, whether partial or whole
    let word = first_word(&my_string_literal[0..6]);
    let word = first_word(&my_string_literal[..]);

    // Because string literals *are* string slices already,
    // this works too, without the slice syntax!
    let word = first_word(my_string_literal);
}
```

## Other Slices

下面的 slice 的type 是 `&[i32]`

```
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

。。 还有一个问题： 当生成一个 slice后 且 在出scope前还会被使用， 是不是 原来的 mut ref/value 就变成 immut 了？ 因为 slice 不可变，所以导致 它指向的 mut ref/value 也变成 不可变了。
。。没有明确说明，但是应该是这样的。

# struct (定义和初始化)

struct 和 tuple 类似， 都 保存了 多个值。 但是struct 有名字，更灵活，不用像 tuple 那样 只能靠 下标 来访问。

## define

以 关键字 struct 开头，然后是 名字， 然后是 {}， 里面可以定义 名字 和 type，称之为 field。

。。最后没有 ;;;;;;

```
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

## create instance

创建实例， 通过 struct 的名字 ， 花括号，及 k:v 对， k:v对的 顺序无所谓：
要获得 值，可以通过 `.` 来完成 。 如果 struct 实例 是 mut 的，那么可以 修改值。

rust 无法控制 单个 field 的 mut， 只能整个 struct instance 的 mut

```
fn main() {
    let mut user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };
    user1.email = String::from("anotheremail@example.com");
}
```

```
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username: username,
        email: email,
        sign_in_count: 1,
    }
}
```

### Using the Field Init Shorthand

上面的 k:v 中，field 的名字 和 形参名字 是相同的，所以可以 使用 field init shorthand 语法：

```
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,
        email,
        sign_in_count: 1,
    }
}
```

### create instance from other instance (update)

创建 一个新 instance，包含了 另一个实例的 大部分内容，但修改了一部分。 通过 struct update syntax 来完成。

原始的方法

```
fn main() {
    // --snip--

    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"),
        sign_in_count: user1.sign_in_count,
    };
}
```

struct update syntax ， 和上面相同， 所以 上面也是 user1 不能再使用了
`..user1` 这个必须在最后。

```
fn main() {
    // --snip--

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```

注意， struct update syntax 使用 = 就像 赋予，因为它 ==move 了数据==。
所以， ==在创建 user2 后， user1 就不能再 使用了， 因为 user1 的 String 类型的 username field 被 move 到 user2 中。==
==如果 我们只使用了 user1 的 active，sign\_in\_count， ( 即 user2 的 email 和 username 是 新的 String 值)， 那么 user1 依然可以 使用。==

。。。6

## tuple struct

rust 也支持 看起来 和 tuple 类似的 struct， 称为 tuple struct。
tuple struct 有 struct 的名字，但是 field没有名字，只有类型。
当你需要给 tuple 一个名字 来和其他tuple 区分， 又懒得写冗长的field名字 的时候，很有用。

定义 tuple struct，以 struct 关键字开始，然后是 struct 的名字，然后是 tuple 的类型。

```
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

注意，black 和 origin 值 是不同的类型，因为 它们是 不同 tuple struct 的实例。
你定义的每个 tuple struct 的 type都是不同的，即使 它们的 fields 是相同类型。

可以像 tuple 一样 解构 struct tuple。
可以和 tuple 一样，使用 `.` \+ 下标 来访问 value。

```rust
struct Point(usize, usize, usize);
// ...
point = Point(0,0,0);
let Point(mut i, mut j, dir) = point
```

。。。下面2个是从 tuple 复制的。

```
    let tup = (500, 6.4, 1);
    let (x, y, z) = tup;
```

```
    let x: (i32, f64, u8) = (500, 6.4, 1);
    let five_hundred = x.0;
    let six_point_four = x.1;
    let one = x.2;
```

## unit-like struct (no field)

定义不带 任何 field 的 struct，被称为 unit-like structs， 因为它们的行为 类似于 `()`
unit type 我们在 tuple type 中提到过。

当你需要 在某些type 上实现 trait，但不要存储 任何数据时，这个有用。

```
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```

struct + 名字 + 分号

* * *

## Ownership of Struxt Data

在 User struct中，我们使用 有拥有权的 String，而不是 &str 这种 string slice type。
这是 故意的，因为 我们希望 每个 struct 的 instance 都拥有 它的所有数据的 own，只要 struct 实例有效，那么里面的 data 也是有效的。

也可以在 struct 中 存储 ref 指向 其他变量 own 的 数据， 但是这样做，需要 使用 lifetime。
lifetime 是 Rust 的 feature。 lifetime 确保 只要struct 实例有效，那么 struct中 ref 的数据 也 有效。

不明确lifetime，编译失败。 chapter10 会讨论。 目前，只能使用 String，不能使用 &str

## example

2个变量

```
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}
```

tuple

```
fn main() {
    let rect1 = (30, 50);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

struct

```
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

### 打印struct

继承 Debug模式，里面有打印的format 方法。

```
。直接 print 报的错是 help: the trait `std::fmt::Display` is not implemented for `Rectangle`
。带上 {:?} 后： error[E0277]: `Rectangle` doesn't implement `Debug`
```

```
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {:?}", rect1);  // 尝试 {:#?}
}
```

```
{:?}
rect1 is Rectangle { width: 30, height: 50 }

{:#?}
rect1 is Rectangle {
    width: 30,
    height: 50,
}
```

```
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
```

## Method Syntax (定义在struct中)

method 和 function 类似： 需要 fn + 名字，有参数 和 返回值， 包含 一些代码来run。
但 method 被定义在 struct (或 enum，或 trait object ) 中， 且第一个参数 始终是 self。
。。self， python:???

### define

```
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

要定义 方法到 Rectangle 的 context 中，我们 ==为 Rectangle 创建一个 impl 块。 这个 impl 块中的 所有东西 都会关联到 Rectangle 类型。==

==method 可以 own self ( 这样就可变)， 也可以不own (不可变)。 上面的代码是 self 不可变的。==

==如果要修改，那么就使用 &mut self==。

`&self` 是 `self: &Self` 的简写
在 impl 块中， Self 类型是 impl 所在 类型的 别名

method 的名字可以和 field 同名。 调用的时候 带括号就是method，不带就是 field。
一般，同名method 是作为 getter

* * *

C,C++ 中有 -> ， rust 没有。
rust是 自动 ref 和 de-ref。调用 method 是少数几个 有这个 功能的 地方。

when you call a method with object.something(), Rust automatically adds in &, &mut, or * so object matches the signature of the method. In other words, the following are the same:

```
p1.distance(&p2);
(&p1).distance(&p2);
```

Given the receiver and name of a method, Rust can figure out definitively whether the method is ==reading (&self), mutating (&mut self), or consuming (self).==

增加参数

```
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

### Associated Functions

定义在 impl 块中的 所有 方法都称为 associated function， 因为 它们关联了 impl 后的 type name。

你可以定义 associated function，不包含 self。 比如 String 类型的 String::from 函数。
一般用来作为 "构造器"。

```
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
```

Self 关键字 是 return type。

要调用这个 associated function，我们使用 struct name + ::

```
let sq = Rectangle::square(3);
```

这个function 被 struct namespace。

### multiple impl blocks

每个 struct 都可以有 多个 impl 块。

```
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

# Enum

枚举

IP v4，v6

```
enum IpAddrKind {
    V4,
    V6,
}
```

```
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
```

```
fn route(ip_kind: IpAddrKind) {}

    route(IpAddrKind::V4);
    route(IpAddrKind::V6);
```

```
    enum IpAddrKind {
        V4,
        V6,
    }

    struct IpAddr {
        kind: IpAddrKind,
        address: String,
    }

    let home = IpAddr {
        kind: IpAddrKind::V4,
        address: String::from("127.0.0.1"),
    };

    let loopback = IpAddr {
        kind: IpAddrKind::V6,
        address: String::from("::1"),
    };
```

* * *

使用 enum 代替 上面的 struct IpAddr，可以更加 简明。

```
    enum IpAddr {
        V4(String),
        V6(String),
    }
    let home = IpAddr::V4(String::from("127.0.0.1"));
    let loopback = IpAddr::V6(String::from("::1"));
```

直接把 数据放到 enum 的值上，这样就不需要 额外的 struct 了。

`IpAddr::V4()` 是一个方法，接收 String，返回 IpAddr 类型。

enum 比 struct 的另一个优势是：每个 枚举值 有不同的类型 和 数据。所以可以：

```
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from("::1"));
```

。。6

Rust 有标准库 保存IP地址：

```
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

你可以将任何值放入到 枚举变量中，string，数值，struct，甚至另一个 enum。

让我们来看 另一个 enum 的例子： 枚举值中有 各种类型。

```
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

这个enum 有 4个 不同type 的 variant：

- Quit，没有关联的数据
- Move，有 具名的field，就像 struct
- Write，包含一个 String
- ChangeColor，包含 3个 i32的值

我们可以使用 4个struct 来代替上面的enum：

```
struct QuitMessage; // unit struct
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // tuple struct
struct ChangeColorMessage(i32, i32, i32); // tuple struct
```

但是，如果使用4个 struct，那么 它们各自就有自己的type， 我们无法定义 一个方法来接收 这4种 message struct，并处理。

enum 和 struct 还有一个相似的地方： 我们可以为 struct 使用 impl 定义method，也可以为 enum定义method。

```
    impl Message {
        fn call(&self) {
            // method body would be defined here
        }
    }

    let m = Message::Write(String::from("hello"));
    m.call();
```

## Option Enum，比Null值更好

Option 是 标准库 定义 的一个 enum。
Option 用于处理：一个变量可能有值，也可能为null 的情况，比如，你要从 list 中取出一个元素，如果list 为空，那么元素就是null。
。。不过Rust 的 null 叫什么？ Rust doesn’t have the null feature that many other languages have

null 的含义就是 目前这个值是无效的。
Rust 用 `Option<T>` 来承担这个职责

```
enum Option<T> {
    None,
    Some(T),
}
```

。。md，模板

Option 非常有用，它是 默认导入的。它的枚举值 也是默认导入的，你可以直接使用 Some 和 None，不需要 Option:: 前缀。

`<T>` 是 泛型。

```
    let some_number = Some(5);  // type is Option<i32>
    let some_char = Some('e');  // type is Option<char>

    let absent_number: Option<i32> = None;
```

### Option's Doc URL

https://doc.rust-lang.org/std/option/enum.Option.html

# Match

Rust 有一个很强的 控制结构，称为 match，可以 比较 值 和 一系列的pattern，然后基于 匹配的 pattern 来执行对应的代码。
pattern 可以由 字面量，变量名，通配符，等组成。

。。case 自带break 的 switch，但 匹配的能力( case的可选范围) 更强大。

```
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

。。注意 最后一个 逗号

match 后面是 (可以推导出任意类型的) 表达式
if 后面的表达式 只能推导出 bool

每个 arm 是一个表达式

```
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

## Patterns That Bind to Values (pattern从value抽取part)

```
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```

## Matching with `Option<T>`

。。抽取出 i。然后 进行操作。

```
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
```

Combining match and enums is useful in many situations. You’ll see this pattern a lot in Rust code: match against an enum, bind a variable to the data inside, and then execute code based on it. It’s a bit tricky at first, but once you get used to it, you’ll wish you had it in all languages. It’s consistently a user favorite.

## Matches Are Exhaustive

arm 的pattern 必须覆盖全部的可能

## other 和 _

other 用来接收所有剩余的选项。 如果 other 后面还有 分支，后面的分支不会运行， 编译器会有 warn。

_ 用于 不需要 绑定值到 具名的other 时。 就是 你用不到这个 value。

```
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn move_player(num_spaces: u8) {}
```

```
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
```

返回一个 unit value (即 empty tuple type)

```
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => (),
    }
```

# if let

结合了 if 和 let，来处理 只匹配一个pattern 并忽略其余 的值。

考虑如下代码，
如果是 Some，绑定值到 max，并且 打印。
如果不是Some，那么就不做任何事。

```
let config_max = Some(3u8);
match config_max {
    Some(max) => println!("The maximum is configured to be {}", max),
    _ => (),
}
```

简写。

```
let config_max = Some(3u8);
if let Some(max) = config_max {
    println!("The maximum is configured to be {}", max);
}
```

下面2个等价

```rust
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}
```

```rust
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```

# 通过 package, crate, module 管理工程

- package, cargo的feature，用来 build，test，share crates
- crate，由模块组成的的 tree，提供 library 或 可执行文件
- module and use，用来 控制 path 的 阻止，scope，privacy
- path，命名一个 item ( 如 struct，function，module ) 的方式。

## Packages and Crates

module system 的第一部分就是 packages 和 crates

crate 是编译器一次编译的 最小单位。
如果 你用 rustc 来构建 一个文件，那么 编译器 就认为这个文件 是一个 crate。

crate 可以包含 module，module 可能定义在 和 crate一起编译的 其他文件中。

crate 有 2 种格式，binary crate, library crate。

binary crate 是那些代码， 你编译后可以获得 可执行文件。 每个binary crate 都必须有一个 main 函数，它定义了 执行时 会发生什么。

library crate 没有main方法，所以不能编译为 可执行文件。它定义了一些方法，可以在不同的工程中使用。

crate root，是一个 source 文件，编译器 从这个文件 开始，构建 crate的 root module。

package 是 一个或多个 crate 组成的 bundle(包)， 提供了一组方法。package 包含 一个 Cargo.toml 文件，这个文件描述了 如何build 这些 crate。

https://doc.rust-lang.org/book/ch07-01-packages-and-crates.html
。。跳了。

# Defining Modules to Control Scope and Privacy

。。只复制了代码

`use` keyword that brings a path into scope;
`pub` keyword to make items public.
`as` keyword

```
mod garden;
```

```
use crate::garden::vegetables::Asparagus;

pub mod garden;

fn main() {
    let plant = Asparagus {};
    println!("I'm growing {:?}!", plant);
}
```

```
pub mod vegetables;
```

```
#[derive(Debug)]
pub struct Asparagus {}
```

```
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

Create a new library named restaurant by running `cargo new restaurant --lib`

# Paths for Referring to an Item in the Module Tree

creta, use the crate keyword to start an absolute path
self
super，relative paths that begin in the parent module

```
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

pub mod

```
mod front_of_house {
    pub mod hosting {
        fn add_to_waitlist() {}
    }
}
```

pub mod, pub fn

```
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

We mentioned a package can contain both a src/main.rs binary crate root as well as a src/lib.rs library crate root, and both crates will have the package name by default.

```
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::deliver_order();
    }

    fn cook_order() {}
}
```

## Making Structs and Enums Public

```
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // Order a breakfast in the summer with Rye toast
    let mut meal = back_of_house::Breakfast::summer("Rye");
    // Change our mind about what bread we'd like
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast please", meal.toast);

    // The next line won't compile if we uncomment it; we're not allowed
    // to see or modify the seasonal fruit that comes with the meal
    // meal.seasonal_fruit = String::from("blueberries");
}
```

# Bringing Paths into Scope with the use Keyword

```
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

## Creating Idiomatic use Paths

```
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting::add_to_waitlist;

pub fn eat_at_restaurant() {
    add_to_waitlist();
}
```

```
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

## Providing New Names with the `as` Keyword

```
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
```

## Re-exporting Names with pub use

```
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

```
// --snip--
use std::cmp::Ordering;
use std::io;
// --snip--


// --snip--
use std::{cmp::Ordering, io};
// --snip--
```

```
use std::io;
use std::io::Write;

use std::io::{self, Write};
```

```
use std::collections::*;
```

## Separating Modules into Different Files

* * *

# Common Collections

一些有用的数据结构。

array ， tuple 是内置的。 保存在栈上(?没有明说，但是作为了对比?)

不同于 array，tuple， collection 保存在 heap上， 意味着它们 不需要在 编译时 确定 size。

我们讨论3个常用的 collection

- vector，`Vec<T>`
- string， String
- hash map，

# Storing Lists of Values with Vectors

`Vec<T>`

内存中 所有value 紧挨着。

只能保存 相同类型的 value。

## create

### create empty vector

```
let v: Vec<i32> = Vec::new();
```

### vector with init

```
let v = vec![1, 2, 3];
```

## update

```
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```

## read

via indexing or using the get method

```Rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("The third element is {third}");

let third: Option<&i32> = v.get(2);
match third {
    Some(third) => println!("The third element is {third}"),
    None => println!("There is no third element."),
}
```

`& 和 []` 配合使用，获得 ref
get 方法返回 `Option<&T>`

不能同时有 mut ref 和 immut ref，所以下面的代码无法通过编译：

```rust
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];  // 在这里，v 是immut

v.push(6);  // 这里，v 是 mut

println!("The first element is: {first}");   // immut 的最后使用。
```

是由于，Vec 是可变长度的，所以当 push 的时候，发现容量用完，那么会进行 resize，会申请新空间，并将老空间返还给OS， 所以 first 这个 ref 会指向已经被释放的 内存。

## iterate vector

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{i}");
}
```

因为是 &mut v， 即ref，所以需要 * de-ref。

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

## Use Enum to Store multiple types

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

## drop

出scope就 free

# String

语言核心中只有一个 字符串类型，即 str 。

String类型，是 Rust 的 标准库提供的，而不是 语言提供的。

String 是 byte的 集合。

## create

```rust
let mut s = String::new();
```

任何 实现了 Display trait 的 类 都有 to_string 方法。

```rust
let data = "initial contents";

let s = data.to_string();

// the method also works on a literal directly:
let s = "initial contents".to_string();
```

下面的 from 和上面的 to_string 等价

```rust
let s = String::from("initial contents");
```

## update

append

```rust
    let mut s = String::from("foo");
    s.push_str("bar");
```

```rust
    let mut s1 = String::from("foo");
    let s2 = "bar";
    s1.push_str(s2);
    println!("s2 is {s2}");
```

push_str 方法 获得了 s2 的 ownership，我们无法执行 最后一行的 println。

push 方法 接收一个char，然后添加到 String。

```rust
    let mut s = String::from("lo");
    s.push('l');
```

## \+ operator and format! 宏

连接 2个已存在的 string 的一种方法是， +号

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
```

s1 变成 invalid，s2 传入 ref 的原因是， `+ operator` 调用的是 add 方法，add 是标准库中 的 泛型方法，对于 String 会是：

```rust
fn add(self, s: &str) -> String {
```

s2 有 ref，意味着 我们 将 第二个string的 ref 添加到 第一个string。

注意，&s2 的类型是 &String， 方法签名中是 &str。 这是因为 编译器 可以 自动强转 &String 为 &str。当调用 add 方法时，Rust 使用 deref coercion， 会将 &s2 转为 `&s2[..]` 。

add 的形参是 self，不是 &self，所以 s1 会被 move 到 add 中，add方法之后 s1就 无效了。

所以是 获得了 s1 的ownership， 然后append 了 s2 的copy 。效率高。

连接多个string

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
```

上面的方法 非常笨拙。我们可以使用 format! 宏。

format! 宏，不会获得 任何ownership，所以 所有String 还是有效的。

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{s1}-{s2}-{s3}");
```

## index string

其他语言都可以，通过 有效的 下标 来访问 string 中的 char。

Rust中不行。

### internal representation

String 是 `Vec<u8>` 的 封装。

```rust
let hello = String::from("Hola");
```

上面的 len 是4， 因为 保存 Hola 的 vector 是 4 byte 长。用UTF-8 编码时，每个字母 一个 byte。

但是，下面的 len 是 24，因为 用 UTF-8 编码时，每个字符 占 2个byte。

```rust
let hello = String::from("Здравствуйте");
```

下面是 非法的，因为 一个byte只是 3 的一部分。

```rust
let hello = "Здравствуйте";
let answer = &hello[0];
```

所以 rust 没有 string的 下标 以获得 char 的功能。

## slicing strings

可以通过 range 来获得一个 slice。下面的 s 会是 4byte长，包含 2个 字符。

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

`&hello[0..1]` ，如果你将 一个字符的 byte 切开了，那么会有 runtime panic。

## iterate string (chars, bytes)

处理 string 的 最好方法是，明确说明 你想要 char 还是 byte。

如果想要 Unicode 值，使用 chars 方法。

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

会输出

```text
З
д
```

如果你要 byte，那么调用 bytes 方法

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

输出

```text
208
151
208
180
```

### contains, replace

contains 方法用于 string 中 搜索

replace 用于替换。

# hash map

`HashMap<K, V>`

## create

create empty hash map

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

和 vector 一样，HashMap 在 heap 上保存数据。

## access

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name).copied().unwrap_or(0);
```

get 方法返回 `Option<&V>`

代码中 通过 调用 copied 来处理 Option，调用 copied 后 获得 `Option<i32>`， 不是 get的返回值 `Oprion<&i32>` 。

如果 scores中不存在这个 key，那么 unwrap_or 会返回 形参 0 。

## iterate k-v pair by for (。无序的)

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{key}: {value}");
}
```

## hash map and ownership

对于 实现了 Copy trait 的 type ( 如 i32 ), 值会被 复制到 hash map 中。

对于 owned 的 值 ( 如 String) ， 值会被 move 到 hash map 中，然后 hash map 成为 这些值的 owner。

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name and field_value are invalid at this point, try using them and
// see what compiler error you get!
```

如果 insert 值的ref 到 hash map中， 值不会被 move 到 hash map 中， 这个 值 的有效 存活期必须比 hash map的 长。

## update

### over-write a value (by insert)

insert 已存在的key，就可以覆盖之前的 value

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);  // 25
```

### add if absent (by entry, or_insert)

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

### update value based on the old value

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
```

将 mut ref 保存到 count 变量中，然后 使用 * 来 de-ref。

## hashing function

HashMap 默认使用 称为 SipHash的 hash方法，可以 防护 对hash table 的 DoS ( 拒绝服务，Denial of Service) 攻击。

这个并不是 最快的 hash方法。

你可以通过 指定 hasher 来切换 hash方法， hasher 是 实现了 BuildHasher trait 的 类型。

你可以在 crates.io 中找到 其他人分享的 hasher。 不必自己写。

# error handling

rust 将 error 分为 2大类： recoverable, unrecoverable error。

## unrevocerable errors with panic

2种触发 panic 的方式：

- 调用会导致 panic 的代码 ( 如 数组越界访问)
- 显式调用 panic! 宏

默认下，panic 会导致 打印失败信息，unwind，清理stack，quit。

通过 环境变量，你可以让 rust 展示 panic发生时的 调用栈。

### unwinding the stack or aborting in response to a panic

默认，当panic 发生，程序就开始 unwinding，这意味着， Rust 将 stack 往回走，清理 遇到的每个 方法的 数据。

这个 walk back 和 clean up 需要 做很多事情。 因为， Rust 可以让你 直接 立刻 aborting，这会 停止程序，不做clean up。

程序用到的 内存 会被 OS 清理。如果 需要让 工程 生成的 二进制程序 尽可能小，你可以选择 从 unwinding 变成 aborting，通过 在 Cargo.toml 的 `[profile]` 中增加 `panic = 'abort'` 。

例如，如果你希望在 release mode 下，使用 abort：

```text
[profile.release]
panic = 'abort'
```

```rust
fn main() {
    panic!("crash and burn");
}
```

### using panic! backtrace

```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

设置 RUST_BACKTRACE 环境变量，来获得 error的具体的 backtrace

```shell
$ RUST_BACKTRACE=1 cargo run
```

要获得 backtrace， 必须使用 debug 模式。 使用 cargo build, cargo run， 不带 --release 时 自动开启debug模式。

## recoverable error with Result

大部分error 并没有严重到 需要 程序整体暂停。

例如，你尝试 打开一个文件，但是 由于不存在，所以失败了，你可能希望 创建一个文件，而不是中断程序。

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`File::open` 的返回值类型是 `Result<T, E>`

T 会是 成功后的值， std::fs::File 类型，是一个 file handle

E 会是 std::io::Error。

如果 File::ope 成功，那么 greeting\_file\_result 会是 Ok的 实例，并且有 一个 file handle。

如果失败，会是 Err 的实例，保存了 error 的信息

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");
}
```

和 Option 枚举一样， Result 枚举 也是 预定义到 scope 的， 所以不需要 Result:: 。

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}
```

### match different error

我们要区分具体的error：如果是 不存在，那么就新建， 如果是 没有权限，那么就应该将 error 抛出。

所以我们增加了一个 match

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {  // 不是 other 或者 _ 吗？
                panic!("Problem opening the file: {:?}", other_error);
            }
        },
    };
}
```

File::open 在失败时，放入到 Err 中的是 io::Error 类型，这是 标准库提供的 struct，它有一个 kind 方法，我们可以得到 io::ErrorKind 值。枚举 io::ErrorKind 也是 标准库的，有 多个枚举值 来表示 io操作的 不同的 错误类型。

### alternative to use math with `Result<T, E>`

上面的代码中match很多。

match 虽然很有用，但是有点原始。

13章会介绍 closure。更简约。

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

### shortcut for panic ( unwrap, expect)

match 有点冗长。

Result 有许多 helper 方法。

unwrap ， 如果 Result 是 Ok，那么返回值，如果是Err，那么 触发一个 panic! 。

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap();
}
```

expect 方法 和 unwrap 类似，增加了 让我们设置 panic! 的错误信息

实际的代码，大多数情况下，选择 expect。

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
}
```

### propagate error

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),   // 。。。。。。return
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```

### shortcut for propagate error ( ? operator)

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```

在 Result 值 后的 ? 的行为 和 match 类似，

如果 Result 是 Ok，那么 取出 其中的 value，

如果是 Err，那么 会从 整个 方法中 return。error 就会扩散出去。

match 和 ? 的不同是：

error values that have the `?` operator called on them go through the `from` function, defined in the `From` trait in the standard library, which is used to convert values from one type into another. When the `?` operator calls the `from` function, the error type received is converted into the error type defined in the return type of the current function. This is useful when a function returns one error type to represent all the ways a function might fail, even if parts might fail for many different reasons.

例如，在 一个方法中 抛出 自定义错误类型： OurError。 如果我们 为 OurError 定义了 `impl From<io::Error>`，来 从 OurError 构造一个 io::Error， 那么 ? 操作 收到 OurError 时，会使用 from 转换 错误类型。

* * *

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();

    File::open("hello.txt")?.read_to_string(&mut username)?;

    Ok(username)
}
```

```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

#### where the ? operator can be used

? operator 只能用在 满足后续条件的 function： 它们的 return type 必须和 ?operator 返回的 value 匹配。

下面的报错：

```console
error[E0277]: the `?` operator can only be used in a function that returns `Result` or `Option` (or another type that implements `FromResidual`)
```

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")?;
}
```

错误信息显示，只允许 ? operator 返回 Result, Option，或其他 实现了 FromResidual 的类。

2种处理方式，一种是 修改方法声明的返回值， 另一种是 使用 match。

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}
```

The `main` function may return any types that implement the `std::process::Termination` trait, which contains a function `report` that returns an `ExitCode`

## panic or not panic

调用 panic! 还是 return Result ？

Therefore, returning `Result` is a good default choice when you’re defining a function that might fail.

#### Guidelines for Error Handling

https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling

如果 会导致的你代码进入 bad state ，那么使用 panic。

bad state 是指 assumption, guarantee, contract, or invariant (假设，保证，约束，不变量) 被打破。

。。太多了。跳了， error handling 这个大章 也跳了好多。

# generic types, traits, lifetime

## generic data types

使用泛型定义 function, struct, enum, method

编译时确定真实类型。

### define function

```rust
fn largest_i32(list: &[i32]) -> &i32 {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn largest_char(list: &[char]) -> &char {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);
    let char_list = vec!['y', 'm', 'a', 'q'];
    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
}
```

```rust
fn largest<T>(list: &[T]) -> &T {
```

```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {  // error[E0369]: binary operation `>` cannot be applied to type `&T`
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

```text
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0369]: binary operation `>` cannot be applied to type `&T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- &T
  |            |
  |            &T
  |
help: consider restricting type parameter `T`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {
  |             ++++++++++++++++++++++

For more information about this error, try `rustc --explain E0369`.
error: could not compile `chapter10` due to previous error

```

`std::cmp::PartialOrd` 是 trait

### define struct

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

### define enum

```rust
enum Option<T> {
    Some(T),
    None,
}
```

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### define method

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

下面这个就是 f32 类型 独有的方法。

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

## trait: define shared behavior

trait 定义了 某个类型所具有的 以及 能和其他 type 共享的 功能。

我们 使用 trait 以 抽象的方式 定义 共享的功能

我们使用 trait bound 来指定 泛型类型可以是 有特定行为的任 任何类。

Note： trait 和 其它语言中的 interface 有点像。

### define trait

type的行为 包含了 我们可以在这个type 上调用的 method。

如果 我们可以在 一些type 上 调用相同的行为，那么 这些不同的type 有着相同的行为。

trait definition 是一种 对 method 签名 进行 分组的方法，将 为了实现某些目标 而必须有的 method 定义到一起。

例如，我们有2个struct，保存不同类型 不同数量的 文本：

- NewsArticle struct，保存了在 特定地点报道的一条新闻，
- Tweet struct，最多280个char，包含了 用来表明 这是一条 new tweet, retweet 还是 reply to another tweet 的 元信息

我们想要做一个 媒体聚合库 ( media aggregator library)，创建 aggregator 来展示 保存在 NewsArticle 或 Tweet 中的 数据的 summary。 为了实现这个目标，我们需要 对每个 type 都进行 summary， 我们通过 调用 struct 实例上的 summarize 方法来 获得 summary。 定义一个 public Summary trait 来表达这个 行为。

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

定义为 pub， 这样 依赖这个 crate 的 crate 也可以使用这个 trait

### impl trait on a type

`impl Summary for NewsArticle {`

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {   // <<<<<<<<<<<
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

an example of how a binary crate could use our `aggregator` library crate:

```rust
use aggregator::{Summary, Tweet};

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}
```

其他依赖 aggregator 的 crate 也可以 引入 Summary 到 scope 来 在它们自己的类上 实现 Summary。

有一个限制是： 我们可以在 一个type 上 实现 trait 只有当 at least one of the trait or the type is local to our crate.

比如，我们可以在 我们的 aggregator crate 中 在 自定义type ( 如 Tweet) 上 实现 标准库的Display trait，因为 Tweet is local to aggregator crate

我们也可以在 我们的 aggregator crate 中为 `Vec<T>` 实现 Summary，因为 Summary is local to aggregator crate。

我们不能在 external type 上 实现 external trait。比如，我们不能在 aggregator crate 中 为 `Vec<T>` 实现 Display trait， 因为 Display 和 `Vec<T>` 都是定义在 标准库中的， 不是 local to aggregator的。

这个限制 是 被称为 coherence (连贯性，相干性) 的性质 的一部分。 更具体地说，是 orphan (孤儿) rule，因为 parent type 不存在。

这个 规则 确保，别人的代码不会 破坏你的代码，反之也不会。没有这个规则的的话，2个 crate 可以 为 相同的type 实现 相同的 trait，rust 不知道该用哪个。

### default impl

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}

// ---------
impl Summary for NewsArticle {}

// ---------
let article = NewsArticle {
    headline: String::from("Penguins win the Stanley Cup Championship!"),
    location: String::from("Pittsburgh, PA, USA"),
    author: String::from("Iceburgh"),
    content: String::from(
        "The Pittsburgh Penguins once again are the best \
        hockey team in the NHL.",
    ),
};

println!("New article available! {}", article.summarize());
```

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

一但 自己实现了 trait的方法，就没有办法 再调用 默认实现了

### trait as parameter

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

### trait bound syntax

impl Trait 语法 是直接了当的，对于 较长的形式 有一个 语法糖，称为 trait bound ：

。。这个 impl Trait 是指 上面的 trait as parameter ？ 是的 ，后续说到了。

。。下面没有办法缩短吧。 就是 多了 impl 4个char 而以啊。。。 后面有。。

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
// -----
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

### 指明 多个 trait bound by +

type 必须实现 Summary 和 Display trait

```rust
pub fn notify(item: &(impl Summary + Display)) {
//---
pub fn notify<T: Summary + Display>(item: &T) {
```

### 更清晰的 trait bound with where 子句

太多的 trait bound 会 使得 方法名字 和 参数列表 间隔太远，难以阅读。

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {

// -----------

fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
```

### return type that impl trait

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```

只能在 返回 一种 type 时， 使用上面的 语法， 如果 返回 2种type ( if xx return Tweet else return NewArticle)，编译失败。

### use trait bound to conditionally impl method

通过 trait bound 和 使用泛型的 impl 块，我们可以 有条件地实现 method。

例如，`Pair<T>` 实现了 new 方法 返回 实例。

但是在 下一个 impl 块中， `Pair<T>` 实现了 cmp_display，如果 T 类型实现了 PartialOrd trait ( 用于比较) 和 Display ( 用于打印)

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

#### 可以为实现了某个trait 的所有type 增加一个 method

为 某个 trait 的所有 type 实现 方法，称为 blanket implementations，在 rust的 标准库中 使用的非常多。

例如，标准库 实现了 为所有实现了 Display trait 的 type 生成了 ToString 方法。类似于下面。

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

我们可以 在 所有实现 Display trait 的 type 上 调用 ToString trait 定义的 to_string 方法。

```rust
let s = 3.to_string();
```

* * *

# 开始中文doc

英文为辅

# validate ref with lifetime

https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html

lifetime 是另一种 泛型。 但不是用于 确保 type 有我们想要的行为。

lifetime 确保 ref 始终有效。

rust中每个ref 都有 lifetime，即 ref 有效的 scope。

大多数 lifetime 是隐含的，并可以推导的，正如大多数的时候 type 也可以推导。
但，如果有 多种可能性时， 必须 显示指定。

## lifetime 防止悬垂ref

lifetime 的主要目标是 避免 悬垂ref， 后者会导致程序 引用了 非预期 引用 的数据

下面的程序，有一个外部scope 和一个内部scope

编译失败， r ref了 x， x 已经free了。

```rust
fn main() {
    let r;  // 可以不初始化，只要在初始化之前，不使用这个变量。

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
```

## borrow checker

借用检查器 它 比较作用域 来确保 所有的borrow 都是有效的。

r 的 lifetime 是 'a, x的lifetime是 'b。 编译时，rust 比较这 2个 lifetime，发现 r 拥有lifetime 'a。 但是它 ref 了一个 lifetime 'b 的对象。 由于 'b 比 'a 小： 被ref 的对象 比 它的引用者 的存在时间 更短。所以 拒绝编译。

```rust
fn main() {
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}                         // ---------+
```

下面是正确的， 没有悬垂ref， 的 例子

```rust
fn main() {
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+
```

这里 x 的 lifetime 'b ，比 'a 要大，这就说明 r 可以 ref x， 因为Rust 知道 r有效的时候，x 肯定有效。

## function 中泛型生命周期

编写一个，返回 2个字符串slice 中 较长者的 函数。这个函数 获取 2个 字符串slice 并返回一个 字符串slice。

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```

注意，这个函数获取 作为ref 的 字符串slice，而不是 字符串。 因为我们不希望 函数获得参数的 所有权。

下面的实现 并不能编译

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

返回值 需要一个 泛型生命周期，但是 rust 并不知道 要返回 的 ref 是x 还是 y。

## 生命周期注解语法

生命周期注解 不改变 生命周期。

只是描述 多个 ref之间的 生命周期的关系。

和 方法指定 泛型后，可以 接收任意type 一样， 在指定 泛型生命周期形参后，方法可以接收任意 ref。

。。应该是指 T 可以被任何type 替换， 而 泛型生命周期 L 也可以被 任意 生命周期替换。

生命周期注解的语法： 以`'`开头，名称通常是 非常短的 全小写。

大多数人 使用 `'a` 作为 第一个生命周期注解。

生命周期参数注解 位于 ref 的 & 后。并有 空格 和 ref-type 分开。

例如，没有生命周期参数的i32 ref，一个有叫做 'a 的 生命周期的 i32 的ref。一个 生命周期也是 'a 的 i32 的 可变ref

```rust
&i32        // 引用
&'a i32     // 带有显式生命周期的引用
&'a mut i32 // 带有显式生命周期的可变引用
```

单个的 生命周期注解 没有什么意义， 因为 生命周期 注解 告诉 Rust 多个 ref 的 泛型生命周期参数 如何相互联系。

假如，函数有一个 生命周期'a 的 i32 的ref 参数 first， 还有一个 也是 'a 的 i32 的ref 参数 second， 这2个生命周期注解 意味着 first 和 second 的 生命周期 一样。

。。这段，英文doc 中没有啊。

## function 签名中的生命周期注解

为了在函数签名中使用生命周期注解，需要在函数名和参数列表间的尖括号中声明泛型生命周期（lifetime）参数，就像泛型类型（type）参数一样。

下面的方法签名表明：接受2个 至少 'a 生命周期的参数， 并返回一个 至少 'a 声明周期的对象。

`'a` 实际上就是 2个参数中 ==生命周期短的== 那个生命周期。

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

。。这个中文文档里面的复制，怎么会带上 main，页面上是不带的。但是点击复制按钮后，就是带的。 并且 英文文档，复制按钮后，是不带main的

传递 不同生命周期的 ref ， 下面是正确的。

如果 println! 和 它后面一行 交换，并且 result在外部声明，那么就 因为 生命周期 编译失败。

```rust
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

## 深入理解生命周期

如果 函数 始终返回 第一个参数，那么 就不需要指定 第二个参数的 生命周期：

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

当从 函数 返回一个 ref时， 返回值的 生命周期 需要和 一个 参数的 生命周期 参数匹配。

如果返回的 ref 没有指向 任何一个 参数，那么唯一的 可能就是，它指向了 函数内部 创建的值，它会是一个 悬垂ref。

下面会编译失败， 因为 返回值的 生命周期 和 参数 没有任何关系。

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```

## 结构体中的生命周期注解

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

## 生命周期省略

每个ref 都有一个 生命周期， 并且 我们需要为 使用了 ref 的 函数 或 结构体 指定 生命周期。

rust 会推导 生命周期，所以 有时可以省略。

函数 或 方法的参数的 生命周期 被称为 输入生命周期(input lifetime)，而返回值的 生命周期 被称为 输出生命周期 (output lifetime)

编译器采用 3条规则 来判断 什么时候 ref 不需要 明确的 生命周期，第一条适用于 输入生命周期，后2条适用于 输出生命周期。

如果 编译器在 检查完 3条规则后，依然没有计算出 生命周期，那么 会 编译失败。

这些规则适用于 fn 定义 和 impl 块

1.  编译器为每个 ref 参数 都分配一个独立的 生命周期参数。
2.  如果只有 一个输入生命周期参数， 那么会将 它的 生命周期 赋给 所有 输出参数
3.  如果 方法有多个 输入生命周期参数，且 其中一个参数 是 &self 或 &mut self， 说明是 对象的 method，那么 所有 输出生命周期 参数 被赋予 self 的生命周期。 这条规则 使得 方法更容易读写，因为 只需要 更少的符号。

## method definition 中的生命周期注解

实现method时，结构体字段的 生命周期 必须总是 在 impl 关键字之后声明，并在 结构体名称 之后 使用，因为这些 生命周期 是 struct类型的 一部分。

impl块中的 方法签名里，ref 可能与 struct 字段中的 ref 相关联，也可能独立。

生命周期省略 也经常让我们 无需在 method签名中 使用 生命周期注解。

下面是一个 method，只有一个 self参数，并且 返回 i32，没有任何 ref

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {  // 只有这个是页面显示的，其他3个都是copy的时候送的
    fn level(&self) -> i32 {
        3
    }
}

impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

impl 之后 和 类型名称 之后的 生命周期参数 是必要的，不过因为 第一条生命周期规则，我们并不需要 标注 &self 的 生命周期

下面是一个 适用于 第三条 生命周期省略规则的 例子。
下面有 2个 input lifetime， 所以 rust 应用第一条规则，为 &self，announcement 分别赋予 独立的 生命周期, 然后，因为 其中一个参数是 &self，返回值 类型 被赋予了 &self 的生命周期，这样 所有的 生命周期都被计算出来了。

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}

impl<'a> ImportantExcerpt<'a> {  // -----------这个是页面的，其他3个是送的。
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

## 静态生命周期

特殊的 生命周期 `'static` 。 它可以在 整个程序期间 存活。

所有的 字符串字面值 都有 'static 生命周期， 可以显式标注：

```rust
#![allow(unused)]
fn main() {
    let s: &'static str = "I have a static lifetime.";
}
```

在将 ref 指定为 'static 之前，三思， 是否真的整个程序期间 有效？ 是否希望活得那么久？

大多数情况下， 编译器的错误信息 建议你 使用 'static 生命周期 都是因为 尝试 创建一个 悬垂ref 或 可用的 生命周期不匹配。 此时 应该修改 悬垂ref 或生命周期不匹配的问题 ，不是 创建 'static

## 结合 泛型参数，trait bounds, 生命周期

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest_with_an_announcement(
        string1.as_str(),
        string2,
        "Today is someone's birthday!",
    );
    println!("The longest string is {}", result);
}

use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(   // <<<
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

# 编写自动化测试

https://kaisery.github.io/trpl-zh-cn/ch11-00-testing.html

## 如何编写测试

测试函数体 通常只需下列3种操作

1.  设置任何所需的 数据或状态
2.  运行需要测试的代码
3.  断言其结果是我们所期望的

rust 提供了 专门用于 编写测试的功能： test属性，宏，should_panic属性

### 测试函数剖析

最简单的例子： 一个带有 test 属性注解的函数， 属性(attribute) 是 rust 代码片段的元数据。

为了将 一个方法变为 test方法，需要在 fn 上行加上 `#[test]` 。

执行 `cargo test` 时， rust 会构建一个 测试执行程序 用来 调用 被标注的 函数，并报告 每个测试 是 通过还是失败。

每次使用 cargo 创建 library project 时，它会自动 生成一个 测试模块 和一个测试函数。

```rust
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

adder 库中 src/lib.rs 的内容应该如下：

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
}
```

执行 `cargo test`

### 性能测试

目前不是稳定版：

https://doc.rust-lang.org/unstable-book/library-features/test.html

### 使用 assert! 宏来检查结果

为 assert! 宏提供一个 值类型为bool的参数。如果 true，什么都不做。如果false，assert! 调用 panic! 。

被测试代码：

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

测试用代码：

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

#[cfg(test)]   // <<< 页面只有这个，上面是 复制按钮 多的。。
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(larger.can_hold(&smaller));
    }
}
```

tests 是一个普通的模块。是一个内部模块，需要测试外部模块中的代码，需要将其引入到 内部模块的 作用域中。

这里选择使用 glob 全局导入。

### 使用 assert\_eq! 和 assert\_ne! 宏来测试 相等。

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```

assert\_eq! 和 assert\_ne! 宏在底层分别使用了 == 和 != 。断言失败时，调用 debug 格式打印参数。

所以 被比较的值 必需 实现 PartialEq 和 Debug trait。 所有的 基本类型 和 大部分标准库 类型 都实现了 这些 trait。

对于自定义的 struct 和 enum， 需要实现 PartialEq 才能断言 值是否相等。需要实现 Debug 才能在断言时打印它们的值。

由于这 2个 trait 都是 derivable trait，所以直接在 struct 或 enum 上添加 `#[derive(PartialEq, Debug)]`

。。 似乎没有说要实现？ 只需要 添加 继承声明就可以了。

### 自定义失败信息

```rust
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(
            result.contains("Carol"),
            "Greeting did not contain name, value was `{}`",
            result
        );
    }
}
```

### 使用 should_panic 检查panic

被测试的代码 panic 认为成功，没有panic 则认为失败。

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }
        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

panic 可能并不是我们所希望的。

增加一个 expected 参数，测试工具 会保证 错误信息中包含 expected 中的文本。

```rust
pub struct Guess {
    value: i32,
}

// --snip--

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(
                "Guess value must be greater than or equal to 1, got {}.",
                value
            );
        } else if value > 100 {
            panic!(
                "Guess value must be less than or equal to 100, got {}.",
                value
            );
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

### 在测试中使用 `Result<T, E>`

到目前为止，编写的测试案例，在失败时 都会panic。

我们也可以使用 `Result<T, E>` 来编写测试。

下面失败时 返回 Err 而非 panic。

it\_works 的返回值类型为 Result&lt;(), String&gt;， 在函数体中，不是之前的 调用 assert\_eq! 宏，而是在 测试通过时 返回 Ok(())， 测试失败时 返回 带有 String 的Err。

这样编写测试 来返回 `Result<T, E>` 就可以在 函数体中 使用问号运算符。

不要对 使用 `Result<T, E>` 的测试 使用 `#[should_panic]` 注解。为了断言 一个操作 返回 Err 成员，不要 对 `Result<T, E>` 使用 问号表达式， 而是使用 assert!(value.is_err()) 。

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

## 控制测试如何允许

cargo test 在测试模式下 编译代码并运行。

cargo test 产生的 二进制文件的 默认行为 是 并发运行所有的测试，并截获 测试运行 过程中 产生的 输出，阻止他们被显示出来。

可以将 一部分命令行命令 传递给 cargo test， 同时 将另一部分传递给 生成的 测试二进制文件。

为了分隔 这 2种参数，需要 先列出 cargo test 的参数， 然后是 分隔符 `--` ，然后是 传递给 二进制文件的 参数。

`cargo test --help`

`cargo test -- --help`

### 并行或串行运行测试 --test-threads=1

rust 默认多线程并发执行 test。 所以你要确保 test 不能相互依赖，不能共享状态。

控制线程数量 为1 改为串行：

```shell
$ cargo test -- --test-threads=1
```

### 显示函数输出 --show-output

```shell
$ cargo test -- --show-output
```

### 通过指定名字来运行部分测试

现有如下的 test代码

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }

    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }

    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}
```

#### 运行单个测试

```rust
$ cargo test one_hundred
```

#### 运行多个测试

可以指定 部分测试的名称，任何名称 匹配这个名称的 test 会被执行， 例如 2个test 包含 add，所以 可以 下面的命令来执行 这2个测试

```rust
$ cargo test add
```

### 忽略某些测试 #\[ignore\]

通过 `#[ignore]` , 让 `cargo test` 忽略掉

```rust
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]
fn expensive_test() {
    // 需要运行一个小时的代码
}
```

#### 只运行被忽略的测试

```ruby
$ cargo test -- --ignored
```

#### 全部运行

`cargo test -- --include-ignored`

## 测试的组织结构

rust社区 倾向于 根据测试的2个主要分类 来考虑问题：单元测试 unit test 和 集成测试 integration test。

单元测试倾向于更小更集中，在隔离的环境中一次测试一个模块，或者是测试 私有接口。

集成测试 对于你的库来说 是完全外部的。 它们和其他外部代码一样，通过相同的方式 使用你的代码，只测试 公有接口，每个测试可能会 测试多个模块。

### 单元测试

https://kaisery.github.io/trpl-zh-cn/ch11-03-test-organization.html

。。跳

`#[cfg(test)]` 只在 `cargo test` 时 才编译和运行代码。 `cargo build` 的时候不会编译这些代码，节省时间。

集成测试是另外一个文件夹，所以不需要 上面的注解。

# 一个IO项目，构建命令行程序

https://kaisery.github.io/trpl-zh-cn/ch12-01-accepting-command-line-arguments.html

。。跳

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    dbg!(args);
}
```

```rust
use std::env;
use std::fs;

fn main() {
    // --snip--
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let file_path = &args[2];

    println!("Searching for {}", query);
    println!("In file {}", file_path);

    let contents = fs::read_to_string(file_path)
        .expect("Should have been able to read the file");

    println!("With text:\n{contents}");
}
```

# Rust的函数式语言功能：迭代器和闭包

本章会涉及

1.  闭包
2.  迭代器
3.  闭包和迭代器的使用，改进上一章的工程
4.  闭包和迭代器的性能(非常快)

## 闭包：可以捕获环境的匿名函数

rust的 闭包 是可以保存到一个变量 或作为参数传递 给其他函数的 匿名函数。

可以在 一个地方创建 闭包，然后在不同的 上下文中 执行闭包计算。

不同于函数，闭包允许捕获被定义时 所在作用域中的值。

### 闭包会捕获其环境

首先了解如何通过闭包捕获定义它的环境中的值以便之后使用。

考虑以下场景： 有时 T恤公司会 送 T恤给 邮件列表中的成员作为促销。邮件列表中的成员可以 选择将他们的喜爱的颜色添加到 个人信息中。如果 被选中的成员 设置了 喜爱的颜色，他们将获得那个颜色的T恤，如果他没有设置 喜爱的颜色，他们会获赠公司现存最多的 颜色的款式。

一种实现方式：建立 ShirtColor 枚举 (里面有 Red 和Blue)。 使用 Inventory struct 来代表公司的库存， 它有一个类型为 `Vec<ShirtColor>` 的 shirts 字段表示 库存中 T恤的颜色。 Inventory上定义 giveaway 方法获取 免费T恤得主 所喜爱的颜色， 并返回其获得的T恤的颜色

```rust
#[derive(Debug, PartialEq, Copy, Clone)]
enum ShirtColor {
    Red,
    Blue,
}

struct Inventory {
    shirts: Vec<ShirtColor>,
}

impl Inventory {
    fn giveaway(&self, user_preference: Option<ShirtColor>) -> ShirtColor {
        user_preference.unwrap_or_else(|| self.most_stocked())
    }

    fn most_stocked(&self) -> ShirtColor {
        let mut num_red = 0;
        let mut num_blue = 0;

        for color in &self.shirts {
            match color {
                ShirtColor::Red => num_red += 1,
                ShirtColor::Blue => num_blue += 1,
            }
        }
        if num_red > num_blue {
            ShirtColor::Red
        } else {
            ShirtColor::Blue
        }
    }
}

fn main() {
    let store = Inventory {
        shirts: vec![ShirtColor::Blue, ShirtColor::Red, ShirtColor::Blue],
    };

    let user_pref1 = Some(ShirtColor::Red);
    let giveaway1 = store.giveaway(user_pref1);
    println!(
        "The user with preference {:?} gets {:?}",
        user_pref1, giveaway1
    );

    let user_pref2 = None;
    let giveaway2 = store.giveaway(user_pref2);
    println!(
        "The user with preference {:?} gets {:?}",
        user_pref2, giveaway2
    );
}
```

main 中定义的 store 还剩2件 蓝T恤 和 1件红T恤。

我们用一个期望获得 红T 和一个没有期望的 用户来调用 giveaway 方法

实现方法多种多样，本次为了使用 闭包，所以，giveaway 方法获取了 `Option<ShirtColor>` 类型 作为用户的 期望颜色，并在 user\_preference 上调用 unwrap\_or_else 方法。 `Option<T>` 的 unwrap\_or\_else 方法由 标准库定义， 它获取一个 无参，返回值类型为T (这个T等同于 Option的Some的值的类型 ) 的 闭包作为参数。 如果 Option 是 Some成员，则返回 Some 中的值，如果是None，则 调用 闭包 并返回 闭包的值。

闭包表达式 `|| self.most_stocked()` 用作 unwrap\_or\_else 的参数。这是一个 无参 的闭包 ( 如果有参数，会出现在 || 之间)。 闭包体 调用了 self.most\_stocked()。 我们定义了闭包， unwrap\_or_else 会在之后需要 其结果时 执行 闭包。

我们传递了一个 会在当前 Inventory 实例上 调用 self.most_stocked() 的闭包。标准库 并不需要知道 我们定义的 Inventory 或 ShirtColor，或我们的逻辑。

### 闭包类型推断 和注解

闭包不像 fn 那样 必须在 参数和 返回值上 注明类型。

fn 需要类型注解，是因为 它们是 暴露给 用户的 显式接口的一部分。严格定义这些接口 对保证 所有人 都对 函数使用 和返回值的类型 理解一致 是很重要的。

闭包 不用于这样 暴露在外的接口： 它们存储在 变量中，并被 使用，不用命名它们 或暴露给 库的用户调用。

闭包通常很短，并 只关联于 小范围的上下文 而非任意场景。在这些 有限制的上下文中，编译器可以 可靠地 推导出 参数 和返回值的类型，类似于它是如何能够推断大部分变量的类型一样（同时也有编译器需要闭包类型注解的罕见情况）。

如果我们希望增加明确性和清晰度也可以添加类型标注，坏处是使代码变得更啰嗦

```rust
    let expensive_closure = |num: u32| -> u32 {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    };
```

。。单行的时候，以把 {} 省略？

下面是等价的fn 和闭包。

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

调用闭包，是 add\_one\_v3, add\_one\_v4 能编译的 必要条件，因为 类型将从其用法中推断出来。

这类似于 `let v = Vec::new();` 需要类型注解 或 某种类型的值 插入到Vec 才能推断其类型。

编译器会为闭包定义中的每个参数和返回值推断一个具体类型

闭包的类型一旦推导出来，就不能再变了。 (即 不能 用int调用后用 string 调用)

### 捕获引用或者移动所有权

闭包可以通过三种方式捕获其环境，它们直接对应到函数获取参数的三种方式：

- 不可变借用，
- 可变借用
- 获取所有权。

闭包会根据函数体中如何使用被捕获的值决定用哪种方式捕获。

一个捕获名为 list 的 vector 的不可变引用的闭包，因为只需不可变引用就能打印其值：

```rust
fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    let only_borrows = || println!("From closure: {:?}", list);

    println!("Before calling closure: {:?}", list);
    only_borrows();
    println!("After calling closure: {:?}", list);
}
```

变量可以绑定一个闭包定义，并且之后可以使用变量名和括号来调用闭包，就像变量名是函数名一样

修改闭包体让它向 list vector 增加一个元素。闭包现在捕获一个可变引用：

```rust
fn main() {
    let mut list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    let mut borrows_mutably = || list.push(7);

    borrows_mutably();
    println!("After calling closure: {:?}", list);
}
```

闭包的定义和调用之间不再有 println!，当 borrows_mutably 定义时，它捕获了 list 的可变引用。闭包在被调用后就不再被使用，这时可变借用结束。因为`当可变借用存在时不允许有其它的借用`，所以在闭包定义和调用之间不能有不可变引用来进行打印。

闭包体不严格需要所有权，如果希望强制闭包获取它用到的环境中值的所有权，可以在参数列表前使用 move 关键字。

在将闭包传递到一个新的线程时这个技巧很有用，它可以移动数据所有权给新线程。

move 关键字的闭包来生成新的线程, 在一个新的线程而非主线程中打印 vector

```rust
use std::thread;

fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    thread::spawn(move || println!("From thread: {:?}", list))
        .join()
        .unwrap();
}
```

如果主线程维护了 list 的所有权但却在新线程之前结束并且丢弃了 list，则在线程中的不可变引用将失效。因此，编译器要求 list 被移动到在新线程中运行的闭包中，这样引用就是有效的。

### 将被捕获的值移出闭包和 Fn trait

闭包体可以做以下任何事：

- 将一个捕获的值移出闭包，
- 修改捕获的值，
- 既不移动也不修改值，
- 或者一开始就不从环境中捕获值。

闭包捕获和处理 环境中的值的方式 影响闭包实现的 trait。 Trait 是函数和结构体指定它们能用的闭包的类型的方式。取决于闭包体如何处理值，闭包自动、渐进地实现一个、两个或三个 Fn trait。

- FnOnce 适用于能被调用一次的闭包，所有闭包都至少实现了这个 trait，因为所有闭包都能被调用。一个会将捕获的值移出闭包体的闭包只实现 FnOnce trait，这是因为它只能被调用一次。
- FnMut 适用于不会将捕获的值移出闭包体的闭包，但它可能会修改被捕获的值。这类闭包可以被调用多次。
- Fn 适用于既不将被捕获的值移出闭包体也不修改被捕获的值的闭包，当然也包括不从环境中捕获值的闭包。这类闭包可以被调用多次而不改变它们的环境，这在会多次并发调用闭包的场景中十分重要。

。。 fn Fn
。。好像没有办法直接声明，必须得是根据内容，行为 来确定的。

```rust
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

泛型 F 的 trait bound 是 FnOnce() -> T，这意味着 F 必须能够被调用一次，没有参数并返回一个 T

现在让我们来看定义在 slice 上的标准库方法 sort\_by\_key，看看它与 unwrap\_or\_else 的区别以及为什么 sort\_by\_key 使用 FnMut 而不是 FnOnce trait bound。这个闭包以一个 slice 中当前被考虑的元素的引用作为参数，返回一个可以用来排序的 K 类型的值。当你想按照 slice 中元素的某个属性来进行排序时这个函数很有用。在示例 13-7 中有一个 Rectangle 实例的列表，我们使用 sort\_by\_key 按 Rectangle 的 width 属性对它们从低到高排序：

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    list.sort_by_key(|r| r.width);
    println!("{:#?}", list);
}
```

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut num_sort_operations = 0;
    list.sort_by_key(|r| {
        num_sort_operations += 1;
        r.width
    });
    println!("{:#?}, sorted in {num_sort_operations} operations", list);
}
```

## 使用迭代器处理元素序列

迭代器（iterator）负责遍历序列中的每一项和决定序列何时结束的逻辑。

在 Rust 中，迭代器是 惰性的（lazy），这意味着在调用方法使用迭代器之前它都不会有效果

由于没有被调用方法，这段代码本身没有任何用处：

```rust
    let v1 = vec![1, 2, 3];
    let v1_iter = v1.iter();
```

使用 for 循环来遍历一个数组并在每一个项上执行了一些代码。在底层它隐式地创建并接着消费了一个迭代器

```rust
fn main() {
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    for val in v1_iter {
        println!("Got: {}", val);
    }
}
```

### Iterator trait和next方法

迭代器都实现了 一个叫做 Iterator 的定义在标准库中的 trait，类似：

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // 此处省略了方法的默认实现
}
```

还未讲到的新语法：type Item 和 Self::Item，他们定义了 trait 的 关联类型（associated type）

现在只需知道这段代码表明实现 Iterator trait 要求同时定义一个 Item 类型，这个 Item 类型被用作 next 方法的返回值类型。换句话说，Item 类型将是迭代器返回元素的类型。

next 是 Iterator 实现者被要求定义的唯一方法。next 一次返回迭代器中的一个项，封装在 Some 中，当迭代器结束时，它返回 None。

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn iterator_demonstration() {
        let v1 = vec![1, 2, 3];

        let mut v1_iter = v1.iter();

        assert_eq!(v1_iter.next(), Some(&1));
        assert_eq!(v1_iter.next(), Some(&2));
        assert_eq!(v1_iter.next(), Some(&3));
        assert_eq!(v1_iter.next(), None);
    }
}
```

v1_iter 需要是可变的：在迭代器上调用 next 方法改变了迭代器中用来记录序列位置的状态。

for 循环时无需使 v1\_iter 可变因为 for 循环会获取 v1\_iter 的所有权并在后台使 v1_iter 可变。

。。？for 获得所有权，那么 for 后，这个 v1_iter 岂不是无效了？

iter 方法生成一个不可变引用的迭代器。如果我们需要一个获取 v1 所有权并返回拥有所有权的迭代器，则可以调用 into\_iter 而不是 iter。类似的，如果我们希望迭代可变引用，则可以调用 iter\_mut 而不是 iter。

### 消费迭代器的方法

Iterator trait 的标准库 API 文档中找到所有这些方法。
一些方法在其定义中调用了 next 方法，这也就是为什么在实现 Iterator trait 时要求实现 next 方法的原因。

这些调用 next 方法的方法被称为 消费适配器（consuming adaptors），因为调用他们会消耗迭代器。一个消费适配器的例子是 sum 方法。这个方法获取迭代器的所有权并反复调用 next 来遍历迭代器，因而会消费迭代器。当其遍历每一个项时，它将每一个项加总到一个总和并在迭代完成时返回总和。

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn iterator_sum() {
        let v1 = vec![1, 2, 3];

        let v1_iter = v1.iter();

        let total: i32 = v1_iter.sum();

        assert_eq!(total, 6);
    }
}
```

调用 sum 之后不再允许使用 v1_iter 因为调用 sum 时它会获取迭代器的所有权。

### 产生其他迭代器的方法

Iterator trait 中定义了另一类方法，被称为 迭代器适配器（iterator adaptors），他们允许我们将当前迭代器变为不同类型的迭代器。可以链式调用多个迭代器适配器。不过因为所有的迭代器都是惰性的，必须调用一个消费适配器方法以便获取迭代器适配器调用的结果。

一个调用迭代器适配器方法 map 的例子，该 map 方法使用闭包来调用每个元素以生成新的迭代器。这里的闭包创建了一个新的迭代器，对其中 vector 中的每个元素都被加 1。

```rust
fn main() {
    let v1: Vec<i32> = vec![1, 2, 3];

    v1.iter().map(|x| x + 1);
}
```

```rust
fn main() {
    let v1: Vec<i32> = vec![1, 2, 3];

    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

    assert_eq!(v2, vec![2, 3, 4]);
}
```

### 使用捕获其环境的闭包

很多迭代器适配器接受闭包作为参数，而通常指定为迭代器适配器参数的闭包会是捕获其环境的闭包。

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn filters_by_size() {
        let shoes = vec![
            Shoe {
                size: 10,
                style: String::from("sneaker"),
            },
            Shoe {
                size: 13,
                style: String::from("sandal"),
            },
            Shoe {
                size: 10,
                style: String::from("boot"),
            },
        ];

        let in_my_size = shoes_in_size(shoes, 10);

        assert_eq!(
            in_my_size,
            vec![
                Shoe {
                    size: 10,
                    style: String::from("sneaker")
                },
                Shoe {
                    size: 10,
                    style: String::from("boot")
                },
            ]
        );
    }
}
```

shoes\_in\_my_size 函数获取一个鞋子 vector 的所有权和一个鞋子大小作为参数。它返回一个只包含指定大小鞋子的 vector。

shoes\_in\_my\_size 函数体中调用了 into\_iter 来创建一个获取 vector 所有权的迭代器。接着调用 filter 将这个迭代器适配成一个只含有那些闭包返回 true 的元素的新迭代器。

闭包从环境中捕获了 shoe_size 变量并使用其值与每一只鞋的大小作比较，只保留指定大小的鞋子。最终，调用 collect 将迭代器适配器返回的值收集进一个 vector 并返回。

这个测试展示当调用 shoes\_in\_my_size 时，我们只会得到与指定值相同大小的鞋子。

## 改进 I/O 项目

### 使用迭代器并去掉 clone

```rust
impl Config {
    pub fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}```

#### 直接使用返回的迭代器

```rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    // --snip--
}
```

```rust
fn main() {
    let config = Config::build(env::args()).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    // --snip--
}
```

```rust
impl Config {
    pub fn build(
        mut args: impl Iterator<Item = String>,
    ) -> Result<Config, &'static str> {
        // --snip--
```

https://kaisery.github.io/trpl-zh-cn/ch13-03-improving-our-io-project.html

这里有上面的代码，复制出来很长很长。 页面只显示了上面的那些。

### 使用 Iterator trait 代替索引

```rust
impl Config {
    pub fn build(
        mut args: impl Iterator<Item = String>,
    ) -> Result<Config, &'static str> {
        args.next();

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a query string"),
        };

        let file_path = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a file path"),
        };

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}
```

### 使用迭代器适配器来使代码更简明

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

可以通过使用迭代器适配器方法来编写更简明的代码。这也避免了一个可变的中间 results vector 的使用。

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents
        .lines()
        .filter(|line| line.contains(query))
        .collect()
}
```

。。类似 java stream。 不，只是链式调用， stream 的话，可以直接改为 parallel，并且可以满足条件后直接退出的(就是 需要收集3个元素，那么只需要计算出前3个就可以了，不会 整个方法都执行完，然后调用这一个方法。 虽然我觉得不太可能。)

## *性能对比：循环 VS 迭代器*

我们运行了一个性能测试，通过将阿瑟·柯南·道尔的“福尔摩斯探案集”的全部内容加载进 String 并寻找其中的单词 “the”。如下是 for 循环版本和迭代器版本的 search 函数的性能测试结果：

```
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

迭代器，作为一个高级的抽象，被编译成了与手写的底层代码大体一致性能代码。迭代器是 Rust 的 **零成本抽象**（zero-cost abstractions）之一，

它意味着抽象并不会引入运行时开销，

它与本贾尼·斯特劳斯特卢普（C++ 的设计和实现者）在 “Foundations of C++”（2012）中所定义的 零开销（zero-overhead）如出一辙：

```
In general, C++ implementations obey the zero-overhead principle: What you don't use, you don't pay for. And further: What you do use, you couldn't hand code any better.

    Bjarne Stroustrup "Foundations of C++"

从整体来说，C++ 的实现遵循了零开销原则：你不需要的，无需为他们买单。更有甚者的是：你需要的时候，也不可能找到其他更好的代码了。

    本贾尼·斯特劳斯特卢普 "Foundations of C++"
```

一些取自于音频解码器的代码。解码算法使用线性预测数学运算（linear prediction mathematical operation）来根据之前样本的线性函数预测将来的值。这些代码使用迭代器链来对作用域中的三个变量进行了某种数学计算：一个叫 buffer 的数据 slice、一个有 12 个元素的数组 coefficients、和一个代表位移位数的 qlp_shift。例子中声明了这些变量但并没有提供任何值；虽然这些代码在其上下文之外没有什么意义，不过仍是一个简明的现实中的例子，来展示 Rust 如何将高级概念转换为底层代码：

```rust
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

为了计算 prediction 的值，这些代码遍历了 coefficients 中的 12 个值，使用 zip 方法将系数与 buffer 的前 12 个值组合在一起。接着将每一对值相乘，再将所有结果相加，然后将总和右移 qlp_shift 位。

。。想起了那个 xxx device。 靠 switch 炸性能。

像音频解码器这样的程序通常最看重计算的性能。这里，我们创建了一个迭代器，使用了两个适配器，接着消费了其值。Rust 代码将会被编译为什么样的汇编代码呢？好吧，在编写本书的这个时候，它被编译成与手写的相同的汇编代码。遍历 coefficients 的值完全用不到循环：Rust 知道这里会迭代 12 次，所以它“展开”（unroll）了循环。展开是一种移除循环控制代码的开销并替换为每个迭代中的重复代码的优化。

所有的系数都被储存在了寄存器中，这意味着访问他们非常快。这里也没有运行时数组访问边界检查。所有这些 Rust 能够提供的优化使得结果代码极为高效。现在知道这些了，

> 请放心大胆的使用迭代器和闭包吧！他们使得代码看起来更高级，但并不为此引入运行时性能损失。

# 进一步认识 Cargo 和 Crates.io

## 采用发布配置自定义构建

在 Rust 中 **发布配置**（*release profiles*）是预定义的、可定制的带有不同选项的配置，他们允许程序员更灵活地控制代码编译的多种选项。每一个配置都彼此相互独立。

Cargo 有两个主要的配置：运行 `cargo build` 时采用的 `dev` 配置和运行 `cargo build --release` 的 `release` 配置。`dev` 配置被定义为开发时的好的默认配置，`release` 配置则有着良好的发布构建的默认配置。

```shell
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
$ cargo build --release
    Finished release [optimized] target(s) in 0.0s
```

*Cargo.toml* 文件中没有显式增加任何 `[profile.*]` 部分的时候，Cargo 会对每一个配置都采用默认设置。通过增加任何希望定制的配置对应的 `[profile.*]` 部分，我们可以选择覆盖任意默认设置的子集。

如下是 `dev` 和 `release` 配置的 `opt-level` 设置的默认值：

文件名：Cargo.toml

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` 设置控制 Rust 会对代码进行何种程度的优化。这个配置的值从 0 到 3。越高的优化级别需要更多的时间编译

## 将 crate 发布到 Crates.io

https://kaisery.github.io/trpl-zh-cn/ch14-02-publishing-to-crates-io.html

## cargo 工作空间

**工作空间** 是一系列共享同样的 *Cargo.lock* 和输出目录的包。

通过共享一个 *target* 目录，工作空间可以避免其他 crate 多余的重复构建。

## 使用 cargo install 安装二进制文件

第十二章提到的叫做 `ripgrep` 的用于搜索文件的 `grep` 的 Rust 实现。为了安装 `ripgrep` 运行如下：

```shell
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v13.0.0
  Downloaded 1 crate (243.3 KB) in 0.88s
  Installing ripgrep v13.0.0
--snip--
   Compiling ripgrep v13.0.0
    Finished release [optimized + debuginfo] target(s) in 3m 10s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v13.0.0` (executable `rg`)
```

最后一行输出展示了安装的二进制文件的位置和名称，在这里 `ripgrep` 被命名为 `rg`。只要你像上面提到的那样将安装目录加入 `$PATH`，就可以运行 `rg --help` 并开始使用一个更快更 Rust 的工具来搜索文件了！

## cargo 自定义扩展命令

如果 `$PATH` 中有类似 `cargo-something` 的二进制文件，就可以通过 `cargo something` 来像 Cargo 子命令一样运行它。像这样的自定义命令也可以运行 `cargo --list` 来展示出来。

# 智能指针

rust中最常用的指针是 引用。

ref 以 &符号为标志，并借用了它们所指向的值。 除了 引用数据，没有其他特殊功能，也没有额外开销。

**智能指针**（*smart pointers*）是一类数据结构，他们的表现类似指针，但是也拥有额外的元数据和功能。

之前已经出现了一些 智能指针，如 String， `Vec<T>` 。 这些类型都属于 智能指针，是因为它们拥有一些数据，并允许你修改它们。它们也拥有元数据和额外的功能或保证。例如 `String` 存储了其容量作为元数据，并拥有额外的能力确保其数据总是有效的 UTF-8 编码。

智能指针通常使用结构体实现。智能指针不同于结构体的地方在于其实现了 `Deref` 和 `Drop` trait。`Deref` trait 允许智能指针结构体实例表现的像引用一样，这样就可以编写既用于引用、又用于智能指针的代码。`Drop` trait 允许我们自定义当智能指针离开作用域时运行的代码。

将会讲到的是来自标准库中最常用的一些：

- `Box<T>`，用于在堆上分配值
- `Rc<T>`，一个引用计数类型，其数据可以有多个所有者
- `Ref<T>` 和 `RefMut<T>`，通过 `RefCell<T>` 访问。（ `RefCell<T>` 是一个在运行时而不是在编译时执行借用规则的类型）。

我们会涉及 **内部可变性**（*interior mutability*）模式，这是不可变类型暴露出改变其内部值的 API。我们也会讨论 **引用循环**（*reference cycles*）会如何泄漏内存，以及如何避免。

## 使用 `Box<T>` 指向堆上的数据

多用于如下场景：

- 当有一个在编译时未知大小的类型，而又想要在需要确切大小的上下文中使用这个类型值的时候
- 当有大量数据并希望在确保数据不被拷贝的情况下转移所有权的时候
- 当希望拥有一个值并只关心它的类型是否实现了特定 trait 而不是其具体类型的时候

### 使用 `Box<T>` 在堆上存储数据

使用 box 在堆上储存一个 `i32`

我们可以像数据是储存在栈上的那样访问 box 中的数据

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```

### Box允许创建递归类型

**递归类型**（*recursive type*）的值可以拥有另一个同类型的值作为其的一部分。这会产生一个问题因为 Rust 需要在编译时知道类型占用多少空间。递归类型的值嵌套理论上可以无限的进行下去，所以 Rust 不知道递归类型需要多少空间。因为 box 有一个已知的大小，所以通过在循环类型定义中插入 box，就可以创建递归类型了。

作为一个递归类型的例子，让我们探索一下 *cons list*。这是一个函数式编程语言中常见的数据类型，来展示这个（递归类型）概念

*cons list* 是一个来源于 Lisp 编程语言及其方言的数据结构，它由嵌套的列表组成。它的名字来源于 Lisp 中的 `cons` 函数（“construct function" 的缩写），它利用两个参数来构造一个新的列表。通过对一个包含值的列表和另一个值调用 `cons`，可以构建由递归列表组成的 cons list。

例如这里有一个包含列表 1，2，3 的 cons list 的伪代码表示，其每一个列表在一个括号中：

```lisp
(1, (2, (3, Nil)))
```

cons list 的每一项都包含两个元素：当前项的值和下一项。其最后一项值包含一个叫做 `Nil` 的值且没有下一项。cons list 通过递归调用 `cons` 函数产生。代表递归的终止条件（base case）的规范名称是 `Nil`，它宣布列表的终止。注意这不同于第六章中的 “null” 或 “nil” 的概念，他们代表无效或缺失的值。

cons list 并不是一个 Rust 中常见的类型。大部分在 Rust 中需要列表的时候，`Vec<T>` 是一个更好的选择。

一个 cons list 的枚举定义。注意这还不能编译因为这个类型没有已知的大小，之后我们会展示：

```rust
enum List {
    Cons(i32, List),
    Nil,
}
```

```rust
use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Cons(2, Cons(3, Nil)));
}
```

编译会得到如下错误

```shell
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
error[E0072]: recursive type `List` has infinite size
 --> src/main.rs:1:1
  |
1 | enum List {
  | ^^^^^^^^^ recursive type has infinite size
2 |     Cons(i32, List),
  |               ---- recursive without indirection
  |
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to make `List` representable
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +

For more information about this error, try `rustc --explain E0072`.
error: could not compile `cons-list` due to previous error

```

### 计算非递归类型的大小

以如下枚举为例

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

当 Rust 需要知道要为 `Message` 值分配多少空间时，它可以检查每一个成员并发现 `Message::Quit` 并不需要任何空间，`Message::Move` 需要足够储存两个 `i32` 值的空间，依此类推。因为 enum 实际上只会使用其中的一个成员，所以 `Message` 值所需的空间等于储存其最大成员的空间大小。

#### 使用Box 给递归类型一个已知的大小

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

`Cons` 成员将会需要一个 `i32` 的大小加上储存 box 指针数据的空间

box 只提供了间接存储和堆分配；他们并没有任何其他特殊的功能，比如我们将会见到的其他智能指针。它们也没有这些特殊功能带来的性能损失，所以他们可以用于像 cons list 这样间接存储是唯一所需功能的场景。

`Box<T>` 类型是一个智能指针，因为它实现了 `Deref` trait，它允许 `Box<T>` 值被当作引用对待。当 `Box<T>` 值离开作用域时，由于 `Box<T>` 类型 `Drop` trait 的实现，box 所指向的堆数据也会被清除。

## 通过 Deref trait 将 智能指针当作常规ref处理

实现 Deref trait 允许我们 重载 解引用运算符`*` (dereference operator)， 不要和 乘法，通配符 混淆。

实现 Deref 的智能指针可以 当作 常规ref 来对待。

### 追踪指针的值

常规ref 是一个指针类型，一种理解指针的方式是将其看成指向储存在其他某处值的箭头。

创建了一个 `i32` 值的引用，接着使用解引用运算符来跟踪所引用的值：

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

#### 像引用一样使用 `Box<T>`

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

#### 自定义智能指针

创建一个类似 标准库 `Box<T>` 的 智能指针

从根本上说，`Box<T>` 被定义为包含一个元素的元组结构体

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

。。tuple struct。。都忘了。。。

#### 通过实现 Deref 将类型像ref一样处理

```rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0   // --返回的是 &
    }
}
// -- 页面只有上面的，下面是 copy 带的。

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

`type Target = T;` 语法定义了用于此 trait 的关联类型。关联类型是一个稍有不同的定义泛型参数的方式，现在还无需过多的担心它；第十九章会详细介绍。

`deref` 方法体中写入了 `&self.0`，这样 `deref` 返回了我希望通过 `*` 运算符访问的值的引用。 `.0` 用来访问元组结构体的第一个元素。

输入 `*y` 时，Rust 事实上在底层运行了如下代码：

```rust
*(y.deref())
```

Rust 将 `*` 运算符替换为先调用 `deref` 方法再进行普通解引用的操作，如此我们便不用担心是否还需手动调用 `deref` 方法了。Rust 的这个特性可以让我们写出行为一致的代码，无论是面对的是常规引用还是实现了 `Deref` 的类型。

`deref` 方法返回值的引用，以及 `*(y.deref())` 括号外边的普通解引用仍为必须的原因在于所有权。如果 `deref` 方法`直接返回值而不是值的引用，其值（的所有权）将被移出` `self`。在这里以及大部分使用解引用运算符的情况下我们并不希望获取 `MyBox<T>` 内部值的所有权。

注意，每次当我们在代码中使用 `*` 时， `*` 运算符都被替换成了先调用 `deref` 方法再接着使用 `*` 解引用的操作，且只会发生一次，不会对 `*` 操作符无限递归替换，解引用出上面 `i32` 类型的值就停止了

。。拿如果 T 是自定义的类型呢？并且是 没有实现 Deref 的类型。

### 函数和方法的 隐式 Deref 强转

**Deref 强制转换**（*deref coercions*）将实现了 `Deref` trait 的类型的引用转换为另一种类型的引用。例如，Deref 强制转换可以将 `&String` 转换为 `&str`，因为 `String` 实现了 `Deref` trait 因此可以返回 `&str`。Deref 强制转换是 Rust 在函数或方法传参上的一种便利操作，并且只能作用于实现了 `Deref` trait 的类型。当这种特定类型的引用作为实参传递给和形参类型不同的函数或方法时将自动进行。这时会有一系列的 `deref` 方法被调用，把我们提供的类型转换成了参数所需的类型。

```rust
fn hello(name: &str) {
    println!("Hello, {name}!");
}
```

Deref 强制转换使得用 `MyBox<String>` 类型值的引用调用 `hello` 成为可能

```rust
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```

如果 Rust 没有实现 Deref 强制转换，为了使用 `&MyBox<String>` 类型的值调用 `hello`，则不得不编写

```rust
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&(*m)[..]);
}
```

`(*m)` 将 `MyBox<String>` 解引用为 `String`。接着 `&` 和 `[..]` 获取了整个 `String` 的字符串 slice 来匹配 `hello` 的签名。

### Deref 强转如何与可变性交互

Deref 重载了 immut ref 的 * 运算符，

DerefMut 用于重载 mut ref 的 * 运算符。

Rust 在发现类型和 trait 实现满足三种情况时会进行 Deref 强制转换：

- 当 `T: Deref<Target=U>` 时从 `&T` 到 `&U`。
- 当 `T: DerefMut<Target=U>` 时从 `&mut T` 到 `&mut U`。
- 当 `T: Deref<Target=U>` 时从 `&mut T` 到 `&U`。

第三个情况有些微妙：Rust 也会将可变引用强转为不可变引用。但是反之是 **不可能** 的：不可变引用永远也不能强转为可变引用

## 使用 Drop trait 清理代码

对于智能指针模式来说第二个重要的 trait 是 `Drop`，其允许我们在值要离开作用域时执行一些代码。可以为任何类型提供 `Drop` trait 的实现，同时所指定的代码被用于释放类似于文件或网络连接的资源。

`Drop` trait 要求实现一个叫做 `drop` 的方法，它获取一个 `self` 的可变引用

当其实例离开作用域时，打印出 `Dropping CustomSmartPointer!`

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("my stuff"),
    };
    let d = CustomSmartPointer {
        data: String::from("other stuff"),
    };
    println!("CustomSmartPointers created.");
}
```

`Drop` trait 包含在 prelude 中，所以无需导入它

### 通过 std::mem::drop 提早丢弃值

Rust 并不允许我们主动调用 `Drop` trait 的 `drop` 方法；当我们希望在作用域结束之前就强制释放变量的话，我们应该使用的是由标准库提供的 `std::mem::drop`。

```rust
fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    println!("CustomSmartPointer created.");
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}
```

## `Rc<T>` 引用计数 智能指针

大部分情况下所有权是非常明确的：可以准确地知道哪个变量拥有某个值。

然而，有些情况单个值可能会有多个所有者。

为了启用==**多所有权**==需要显式地使用 Rust 类型 `Rc<T>`，其为 **引用计数**（*reference counting*）的缩写。引用计数意味着记录一个值引用的数量来知晓这个值是否仍在被使用。如果某个值有零个引用，就代表没有任何有效引用并可以被清理。

`Rc<T>` 用于当我们希望在堆上分配一些内存供程序的多个部分读取，而且无法在编译时确定程序的哪一部分会最后结束使用它的时候。

如果确实知道哪部分是最后一个结束使用的话，就可以令其成为数据的所有者，正常的所有权规则就可以在编译时生效。

注意 `Rc<T>` 只能用于==单线程==场景；第十六章并发会涉及到如何在多线程程序中进行引用计数。

### 使用 `Rc<T>` 共享数据

我们希望创建如下的列表，2个共享了 第3个列表 所有权的列表。

![3cf5feebde5812a906ebc05a0f800219.png](resources/048bc9ecd2654d499839861c230ac334.png)

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
    let b = Cons(3, Box::new(a));
    let c = Cons(4, Box::new(a));
}
```

编译会得到如下错误:

```shell
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
error[E0382]: use of moved value: `a`
  --> src/main.rs:11:30
   |
9  |     let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
   |         - move occurs because `a` has type `List`, which does not implement the `Copy` trait
10 |     let b = Cons(3, Box::new(a));
   |                              - value moved here
11 |     let c = Cons(4, Box::new(a));
   |                              ^ value used here after move

For more information about this error, try `rustc --explain E0382`.
error: could not compile `cons-list` due to previous error
```

`Cons` 成员拥有其储存的数据，所以当创建 `b` 列表时，`a` 被移动进了 `b` 这样 `b` 就拥有了 `a`。接着当再次尝试使用 `a` 创建 `c` 时，这不被允许，因为 `a` 的所有权已经被移动。

可以改变 `Cons` 的定义来存放一个引用，不过接着必须指定生命周期参数。通过指定生命周期参数，表明列表中的每一个元素都至少与列表本身存在的一样久。

修改 `List` 的定义为使用 `Rc<T>` 代替 `Box<T>`，如列表 15-18 所示。现在每一个 `Cons` 变量都包含一个值和一个指向 `List` 的 `Rc<T>`。当创建 `b` 时，不同于获取 `a` 的所有权，这里会克隆 `a` 所包含的 `Rc<List>`，这会将引用计数从 1 增加到 2 并允许 `a` 和 `b` 共享 `Rc<List>` 中数据的所有权。创建 `c` 时也会克隆 `a`，这会将引用计数从 2 增加为 3。每次调用 `Rc::clone`，`Rc<List>` 中数据的引用计数都会增加，直到有零个引用之前其数据都不会被清理。

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```

需要使用 `use` 语句将 `Rc<T>` 引入作用域，因为它不在 prelude 中。在 `main` 中创建了存放 5 和 10 的列表并将其存放在 `a` 的新的 `Rc<List>` 中。接着当创建 `b` 和 `c` 时，调用 `Rc::clone` 函数并传递 `a` 中 `Rc<List>` 的引用作为参数。

也可以调用 `a.clone()` 而不是 `Rc::clone(&a)`，不过在这里 Rust 的习惯是使用 `Rc::clone`。`Rc::clone` 的实现并不像大部分类型的 `clone` 实现那样对所有数据进行深拷贝。`Rc::clone` 只会增加引用计数，这并不会花费多少时间。深拷贝可能会花费很长时间。通过使用 `Rc::clone` 进行引用计数，可以明显的区别深拷贝类的克隆和增加引用计数类的克隆。当查找代码中的性能问题时，只需考虑深拷贝类的克隆而无需考虑 `Rc::clone` 调用。

### clone `Rc<T>` 会增加 ref count

```rust
fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("count after creating a = {}", Rc::strong_count(&a));
    let b = Cons(3, Rc::clone(&a));
    println!("count after creating b = {}", Rc::strong_count(&a));
    {
        let c = Cons(4, Rc::clone(&a));
        println!("count after creating c = {}", Rc::strong_count(&a));
    }
    println!("count after c goes out of scope = {}", Rc::strong_count(&a));
}
```

## `RefCell<T>` 和内部可变性模式

**内部可变性**（*Interior mutability*）是 Rust 中的一个设计模式，它允许你即使在有不可变引用时也可以改变数据，这通常是借用规则所不允许的。为了改变数据，该模式在数据结构中使用 `unsafe` 代码来模糊 Rust 通常的可变性和借用规则。不安全代码表明我们在手动检查这些规则而不是让编译器替我们检查。第十九章会更详细地介绍不安全代码。

当可以确保代码在运行时会遵守借用规则，即使编译器不能保证的情况，可以选择使用那些运用内部可变性模式的类型。所涉及的 `unsafe` 代码将被封装进安全的 API 中，而外部类型仍然是不可变的。

### 通过 `RefCell<T>` 在运行时检查 借用规则

不同于 `Rc<T>`，`RefCell<T>` 代表其数据的唯一的所有权。那么是什么让 `RefCell<T>` 不同于像 `Box<T>` 这样的类型呢？回忆一下第四章所学的借用规则：

1.  在任意给定时刻，只能拥有一个可变引用或任意数量的不可变引用 **之一**（而不是两者）。
2.  引用必须总是有效的。

对于引用和 `Box<T>`，借用规则的不可变性作用于==编译时==。对于 `RefCell<T>`，这些不可变性作用于 ==**运行时**==。对于引用，如果违反这些规则，会得到一个编译错误。而对于 `RefCell<T>`，如果违反这些规则程序会 panic 并退出。

相反在运行时检查借用规则的好处则是允许出现特定内存安全的场景，而它们在编译时检查中是不允许的。静态分析，正如 Rust 编译器，是天生保守的。但代码的一些属性不可能通过分析代码发现：其中最著名的就是 [停机问题（Halting Problem）](https://zh.wikipedia.org/wiki/%E5%81%9C%E6%9C%BA%E9%97%AE%E9%A2%98)，这超出了本书的范畴，不过如果你感兴趣的话这是一个值得研究的有趣主题。

。。。

停机问题是逻辑学的焦点，也是[第三次数学危机](https://baike.baidu.com/item/%E7%AC%AC%E4%B8%89%E6%AC%A1%E6%95%B0%E5%AD%A6%E5%8D%B1%E6%9C%BA/3891463?fromModule=lemma_inlink)的解决方案。其本质问题是: 给定一个[图灵机](https://baike.baidu.com/item/%E5%9B%BE%E7%81%B5%E6%9C%BA/2112989?fromModule=lemma_inlink) T，和一个任意语言集合 S, 是否 T 会最终停机于每一个s∈S。其意义相同于可确定语言。显然任意有限 S 是可判定性的，可列的(countable) S 也是可停机的。

**停机问题**（英语：halting problem）是[逻辑数学](https://baike.baidu.com/item/%E9%80%BB%E8%BE%91%E6%95%B0%E5%AD%A6?fromModule=lemma_inlink)中[可计算性理论](https://baike.baidu.com/item/%E5%8F%AF%E8%AE%A1%E7%AE%97%E6%80%A7%E7%90%86%E8%AE%BA?fromModule=lemma_inlink)的一个问题。通俗地说，停机问题就是判断任意一个[程序](https://baike.baidu.com/item/%E7%A8%8B%E5%BA%8F/13831935?fromModule=lemma_inlink)是否能在有限的时间之内结束运行的问题。该问题等价于如下的判定问题：是否存在一个程序P，对于任意输入的程序w，能够判断w会在有限时间内结束或者[死循环](https://baike.baidu.com/item/%E6%AD%BB%E5%BE%AA%E7%8E%AF/4141007?fromModule=lemma_inlink)。

通俗的说，停机问题就是判断任意一个程序是否会在有限的时间之内结束运行的问题。如果这个问题可以在有限的时间之内解决，则有一个程序判断其本身是否会停机并做出相反的行为，这时候显然不管停机问题的结果是什么都不会符合要求。所以这是一个不可解的问题。

停机问题本质是一高阶逻辑的不自恰性和不完备性。类似的命题有[理发师悖论](https://baike.baidu.com/item/%E7%90%86%E5%8F%91%E5%B8%88%E6%82%96%E8%AE%BA?fromModule=lemma_inlink)、[全能悖论](https://baike.baidu.com/item/%E5%85%A8%E8%83%BD%E6%82%96%E8%AE%BA?fromModule=lemma_inlink)等

。。。

类似于 `Rc<T>`，`RefCell<T>` 只能用于==单线程==场景。如果尝试在多线程上下文中使用`RefCell<T>`，会得到一个编译错误。

如下为选择 `Box<T>`，`Rc<T>` 或 `RefCell<T>` 的理由：

- `Rc<T>` 允许相同数据有多个所有者；`Box<T>` 和 `RefCell<T>` 有单一所有者。
- `Box<T>` 允许在编译时执行不可变或可变借用检查；`Rc<T>`仅允许在编译时执行不可变借用检查；`RefCell<T>` 允许在运行时执行不可变或可变借用检查。
- 因为 `RefCell<T>` 允许在运行时执行可变借用检查，所以我们可以在即便 `RefCell<T>` 自身是不可变的情况下修改其内部的值。

在不可变值内部改变值就是 ==**内部可变性**== 模式。让我们看看何时内部可变性是有用的，并讨论这是如何成为可能的。

### 内部可变性：不可变值的可变借用

借用规则的 一个推论是： 当有一个不可变值时，不能可变地 借用它， 下面的代码不能编译：

```rust
fn main() {
    let x = 5;
    let y = &mut x;
}
```

特定情况下，令一个值在其方法内部能够修改自身，而在其他代码中仍视为不可变，是很有用的。值方法外部的代码就不能修改其值了。`RefCell<T>` 是一个获得内部可变性的方法。`RefCell<T>` 并没有完全绕开借用规则，编译器中的借用检查器允许内部可变性并相应地在运行时检查借用规则。如果违反了这些规则，会出现 panic 而不是编译错误。

### 内部可变性的用例：mock 对象

https://kaisery.github.io/trpl-zh-cn/ch15-05-interior-mutability.html

有时在测试中程序员会用某个类型替换另一个类型，以便观察特定的行为并断言它是被正确实现的。这个占位符类型被称为 **测试替身**（*test double*）。可以把它想象成电影制作中的 “替身演员”（"stunt double"），这是一个龙套进入并替代演员进行一些特定的高难度操作。测试替身在运行测试时替代某个类型。**mock 对象** 是特定类型的测试替身，它们记录测试过程中发生了什么以便可以断言操作是正确的。

如下是一个我们想要测试的场景：我们在编写一个记录某个值与最大值的差距的库，并根据当前值与最大值的差距来发送消息。例如，这个库可以用于记录用户所允许的 API 调用数量限额。

该库只提供记录与最大值的差距，以及何种情况发送什么消息的功能。使用此库的程序则期望提供实际发送消息的机制：程序可以选择记录一条消息、发送 email、发送短信等等。库本身无需知道这些细节；只需实现其提供的 `Messenger` trait 即可。示例 15-20 展示了库代码：

```rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &'a T, max: usize) -> LimitTracker<'a, T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
            self.messenger
                .send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger
                .send("Warning: You've used up over 75% of your quota!");
        }
    }
}
```

这些代码中一个重要部分是拥有一个方法 `send` 的 `Messenger` trait，其获取一个 `self` 的不可变引用和文本信息。这个 trait 是 mock 对象所需要实现的接口库，这样 mock 就能像一个真正的对象那样使用了。另一个重要的部分是我们需要测试 `LimitTracker` 的 `set_value` 方法的行为。可以改变传递的 `value` 参数的值，不过 `set_value` 并没有返回任何可供断言的值。也就是说，如果使用某个实现了 `Messenger` trait 的值和特定的 `max` 创建 `LimitTracker`，当传递不同 `value` 值时，消息发送者应被告知发送合适的消息。

我们所需的 mock 对象是，调用 `send` 并不实际发送 email 或消息，而是只记录信息被通知要发送了。可以新建一个 mock 对象实例，用其创建 `LimitTracker`，调用 `LimitTracker` 的 `set_value` 方法，然后检查 mock 对象是否有我们期望的消息。示例 15-21 展示了一个如此尝试的 mock 对象实现，不过借用检查器并不允许：

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: RefCell::new(vec![]),
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        // --snip--

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
```

### `RefCell<T>` 在运行时记录借用

当创建不可变和可变引用时，我们分别使用 `&` 和 `&mut` 语法。对于 `RefCell<T>` 来说，则是 `borrow` 和 `borrow_mut` 方法，这属于 `RefCell<T>` 安全 API 的一部分。==`borrow`== 方法返回 `Ref<T>` 类型的智能指针，==`borrow_mut`== 方法返回 `RefMut<T>` 类型的智能指针。这两个类型都实现了 `Deref`，所以可以当作常规引用对待。

`RefCell<T>` 记录当前有多少个活动的 `Ref<T>` 和 `RefMut<T>` 智能指针。每次调用 `borrow`，`RefCell<T>` 将活动的不可变借用计数加一。当 `Ref<T>` 值离开作用域时，不可变借用计数减一。就像编译时借用规则一样，`RefCell<T>` 在任何时候==只允许有多个不可变借用或一个可变借用==。

如果我们尝试违反这些规则，相比引用时的编译时错误，`RefCell<T>` 的实现会在运行时出现 panic。

### 结合 `Rc<T>` 和 `RefCell<T>` 来拥有多个可变数据 所有者

`RefCell<T>` 的一个常见用法是与 `Rc<T>` 结合。回忆一下 `Rc<T>` 允许对相同数据有多个所有者，不过只能提供数据的不可变访问。如果有一个储存了 `RefCell<T>` 的 `Rc<T>` 的话，就可以得到有多个所有者 **并且** 可以修改的值了！

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

## 引用循环 和 内存泄漏

Rust 的内存安全性保证使其难以意外地制造永远也不会被清理的内存（被称为 **内存泄漏**（*memory leak*）），但并不是不可能。与在编译时拒绝数据竞争不同，Rust 并不保证完全地避免内存泄漏，这意味着内存泄漏在 Rust 被认为是内存安全的。这一点可以通过 `Rc<T>` 和 `RefCell<T>` 看出：创建引用循环的可能性是存在的。这会造成内存泄漏，因为每一项的引用计数永远也到不了 0，其值也永远不会被丢弃。

### 制造 ref 循环

```rust
use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {}
```

```rust
fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a initial rc count = {}", Rc::strong_count(&a));
    println!("a next item = {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("a rc count after b creation = {}", Rc::strong_count(&a));
    println!("b initial rc count = {}", Rc::strong_count(&b));
    println!("b next item = {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("b rc count after changing a = {}", Rc::strong_count(&b));
    println!("a rc count after changing a = {}", Rc::strong_count(&a));

    // Uncomment the next line to see that we have a cycle;
    // it will overflow the stack
    // println!("a next item = {:?}", a.tail());
}
```

如果你有包含 `Rc<T>` 的 `RefCell<T>` 值或类似的嵌套结合了内部可变性和引用计数的类型，请务必小心确保你没有形成一个引用循环；你无法指望 Rust 帮你捕获它们。创建引用循环是一个程序上的逻辑 bug，你应该使用自动化测试、代码评审和其他软件开发最佳实践来使其最小化。

另一个解决方案是重新组织数据结构，使得一部分引用拥有所有权而另一部分没有。换句话说，循环将由一些拥有所有权的关系和一些无所有权的关系组成，只有所有权关系才能影响值是否可以被丢弃。

### 避免循环ref ： 将 `Rc<T>` 变为 `Weak<T>`

。。。Java.WeakReference: ?

到目前为止，我们已经展示了调用 `Rc::clone` 会增加 `Rc<T>` 实例的 `strong_count`，和只在其 `strong_count` 为 0 时才会被清理的 `Rc<T>` 实例。你也可以通过调用 `Rc::downgrade` 并传递 `Rc<T>` 实例的引用来创建其值的 **弱引用**（*weak reference*）。强引用代表如何共享 `Rc<T>` 实例的所有权。弱引用并不属于所有权关系，当 `Rc<T>` 实例被清理时其计数没有影响。他们不会造成引用循环，因为任何弱引用的循环会在其相关的强引用计数为 0 时被打断。

。。会空指针吗？

因为 `Weak<T>` 引用的值可能已经被丢弃了，为了使用 `Weak<T>` 所指向的值，我们必须确保其值仍然有效。为此可以调用 `Weak<T>` 实例的 `upgrade` 方法，这会返回 `Option<Rc<T>>`。如果 `Rc<T>` 值还未被丢弃，则结果是 `Some`；如果 `Rc<T>` 已被丢弃，则结果是 `None`。

### 创建树形数据结构：带有子结点的 Node

```rust
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        children: RefCell::new(vec![]),
    });

    let branch = Rc::new(Node {
        value: 5,
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });
}
```

#### 增加从子到父的ref

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
}
```

### 可视化 strong\_count 和 weak\_count 的改变

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );

    {
        let branch = Rc::new(Node {
            value: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });

        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

        println!(
            "branch strong = {}, weak = {}",
            Rc::strong_count(&branch),
            Rc::weak_count(&branch),
        );

        println!(
            "leaf strong = {}, weak = {}",
            Rc::strong_count(&leaf),
            Rc::weak_count(&leaf),
        );
    }

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );
}
```

# 无畏并发

https://kaisery.github.io/trpl-zh-cn/ch16-00-concurrency.html

**并发编程**（*Concurrent programming*），代表程序的不同部分相互独立的执行，而 **并行编程**（*parallel programming*）代表程序不同部分于同时执行

团队发现所有权和类型系统是一系列解决内存安全 **和** 并发问题的强有力的工具！通过利用所有权和类型检查，在 Rust 中很多并发错误都是 **编译时** 错误，而非运行时错误。因此，相比花费大量时间尝试重现运行时并发 bug 出现的特定情况，Rust 会拒绝编译不正确的代码并提供解释问题的错误信息。

出于简洁的考虑，我们将很多问题归类为 **并发**，而不是更准确的区分 **并发和（或）并行**。如果这是一本专注于并发和/或并行的书，我们肯定会更加精确的。对于本章，当我们谈到 **并发** 时，请自行脑内替换为 **并发和（或）并行**。

## 使用线程 同时运行代码

- 竞态条件（Race conditions），多个线程以不一致的顺序访问数据或资源
- 死锁（Deadlocks），两个线程相互等待对方，这会阻止两者继续运行
- 只会发生在特定情况且难以稳定重现和修复的 bug

编程语言有一些不同的方法来实现线程，而且很多操作系统提供了创建新线程的 API。Rust 标准库使用 *1:1* 线程实现，这代表程序的每一个语言级线程使用一个系统线程。有一些 crate 实现了其他有着不同于 1:1 模型取舍的线程模型。

### 使用 spawn 创建新线程

为了创建一个新线程，需要调用 `thread::spawn` 函数并传递一个闭包（第十三章学习了闭包），并在其中包含希望在新线程运行的代码。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

注意当 Rust 程序的主线程结束时，新线程也会结束，而不管其是否执行完毕

### 使用 join 等待所有线程结束

`thread::spawn` 的返回值类型是 `JoinHandle`。

`JoinHandle` 是一个拥有所有权的值，当对其调用 `join` 方法时，它会等待其线程结束。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });
    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
    handle.join().unwrap();
}
```

通过调用 handle 的 `join` 会阻塞当前线程直到 handle 所代表的线程结束。

### 线程与 move 闭包

`move` 关键字经常用于传递给 `thread::spawn` 的闭包，因为闭包会获取从环境中取得的值的所有权，因此会将这些值的所有权从一个线程传送到另一个线程。

在第十三章中，我们讲到可以在参数列表前使用 `move` 关键字强制闭包获取其使用的环境值的所有权。这个技巧在创建新线程将值的所有权从一个线程移动到另一个线程时最为实用。

Rust 会 **推断** 如何捕获 `v`，因为 `println!` 只需要 `v` 的引用，闭包尝试借用 `v`。然而这有一个问题：Rust 不知道这个新建线程会执行多久，所以无法知晓 `v` 的引用是否一直有效。

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];
    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });
    handle.join().unwrap();
}
```

一个 `v` 的引用很有可能不再有效的场景：

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];
    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });
    drop(v); // oh no!
    handle.join().unwrap();
}
```

通过在闭包之前增加 `move` 关键字，我们强制闭包获取其使用的值的所有权，而不是任由 Rust 推断它应该借用值。

所有权移动给闭包后，drop(v) 也就不能调用了。

```rust
use std::thread;
fn main() {
    let v = vec![1, 2, 3];
    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });
    handle.join().unwrap();
}
```

## 使用消息传递 在线程间传送数据

一个日益流行的确保安全并发的方式是 **消息传递**（*message passing*），这里线程或 actor 通过发送包含数据的消息来相互沟通。这个思想来源于 [Go 编程语言文档中](https://golang.org/doc/effective_go.html#concurrency) 的口号：“不要通过共享内存来通讯；而是通过通讯来共享内存。”（“Do not communicate by sharing memory; instead, share memory by communicating.”）

为了实现消息传递并发，Rust 标准库提供了一个 **信道**（*channel*）实现。信道是一个通用编程概念，表示数据从一个线程发送到另一个线程。

。。土拨鼠：？真就后发优势是吧。。

。。不过，这种 channel， C++应该怎么实现呢？我感觉像一个MQ一样，好复杂。

。。或者 go rust 的 channel 的底层实现是怎么样的？ epoll？事件监听？

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
}
```

使用 `mpsc::channel` 函数创建一个新的信道；`mpsc` 是 **多个生产者，单个消费者**（*multiple producer, single consumer*）的缩写。

`mpsc::channel` 函数返回一个元组：第一个元素是发送端 \-\- 发送者，而第二个元素是接收端 \-\- 接收者。由于历史原因，`tx` 和 `rx` 通常作为 **发送者**（*transmitter*）和 **接收者**（*receiver*）的缩写，所以这就是我们将用来绑定这两端变量的名字。这里使用了一个 `let` 语句和模式来解构了此元组

`send` 方法返回一个 `Result<T, E>` 类型，所以如果接收端已经被丢弃了，将没有发送值的目标，所以发送操作会返回错误。

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });
}
```

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

信道的接收者有两个有用的方法：`recv` 和 `try_recv`。

这里，我们使用了 `recv`，它是 *receive* 的缩写。这个方法会阻塞主线程执行直到从信道中接收一个值。

一旦发送了一个值，`recv` 会在一个 `Result<T, E>` 中返回它。当信道发送端关闭，`recv` 会返回一个错误表明不会再有新的值到来了。

`try_recv` 不会阻塞，相反它立刻返回一个 `Result<T, E>`：`Ok` 值包含可用的信息，而 `Err` 值代表此时没有任何消息。

### channel 与 所有权转移

我们将尝试在新建线程中的信道中发送完 `val` 值 **之后** 再使用它。尝试编译示例 16-9 中的代码并看看为何这是不允许的：

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();
    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        println!("val is {}", val);  // <<<<<
    });
    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

这里尝试在通过 `tx.send` 发送 `val` 到信道中之后将其打印出来。允许这么做是一个坏主意：一旦将值发送到另一个线程后，那个线程可能会在我们再次使用它之前就将其修改或者丢弃。其他线程对值可能的修改会由于不一致或不存在的数据而导致错误或意外的结果。

### 发送多个值并观察接收者的等待

新建线程现在会发送多个消息并在每个消息之间暂停一秒钟。

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];
        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });
    for received in rx {
        println!("Got: {}", received);
    }
}
```

。。？ 这个什么时候结束呢？ 就是 rx 始终有可能来数据的啊，怎么结束 main ？ 难道不结束的？

将看到如下输出，每一行都会暂停一秒：

```shell
Got: hi
Got: from
Got: the
Got: thread
```

### 通过clone 发送者 来创建 多个生产者

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    // --snip--

    let (tx, rx) = mpsc::channel();

    let tx1 = tx.clone();
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    thread::spawn(move || {
        let vals = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }

    // --snip--
}
```

## 共享并发状态

虽然消息传递是一个很好的处理并发的方式，但并不是唯一一个。另一种方式是让多个线程拥有相同的共享数据。再一次思考一下 Go 编程语言文档中口号的这一部分：“不要通过共享内存来通讯”（“do not communicate by sharing memory.”）

在某种程度上，任何编程语言中的信道都类似于==单所有权==，因为一旦将一个值传送到信道中，将无法再使用这个值。

共享内存类似于==多所有权==：多个线程可以同时访问相同的内存位置。

### mutex 一次只允许一个线程访问数据

mutex 是 mutual exclusion 的缩写。

互斥器以难以使用著称，因为你不得不记住：

1.  在使用数据之前尝试获取锁。
2.  处理完被互斥器所保护的数据之后，必须解锁数据，这样其他线程才能够获取锁。

### `Mutex<T>` 的API

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

像很多类型一样，我们使用关联函数 `new` 来创建一个 `Mutex<T>`。使用 `lock` 方法获取锁，以访问互斥器中的数据。这个调用会阻塞当前线程，直到我们拥有锁为止。

如果另一个线程拥有锁，并且那个线程 panic 了，则 `lock` 调用会失败。在这种情况下，没人能够再获取锁，所以这里选择 `unwrap` 并在遇到这种情况时使线程 panic。

一旦获取了锁，就可以将返回值（在这里是`num`）视为一个其内部数据的可变引用了。类型系统确保了我们在使用 `m` 中的值之前获取锁。`m` 的类型是 `Mutex<i32>` 而不是 `i32`，所以 **必须** 获取锁才能使用这个 `i32` 值。我们是不会忘记这么做的，因为反之类型系统不允许访问内部的 `i32` 值。

`Mutex<T>` 是一个智能指针。更准确的说，`lock` 调用 **返回** 一个叫做 `MutexGuard` 的智能指针。这个智能指针实现了 `Deref` 来指向其内部数据；其也提供了一个 `Drop` 实现当 `MutexGuard` 离开作用域时自动释放锁

### 线程间共享 `Mutex<T>`

启动十个线程，并在各个线程中对同一个计数器值加一，这样计数器将从 0 变为 10

会出现编译错误，而我们将通过这些错误来学习如何使用 `Mutex<T>`，以及 Rust 又是如何帮助我们正确使用的。

```rust
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

```rust
$ cargo run
   Compiling shared-state v0.1.0 (file:///projects/shared-state)
error[E0382]: use of moved value: `counter`
  --> src/main.rs:9:36
   |
5  |     let counter = Mutex::new(0);
   |         ------- move occurs because `counter` has type `Mutex<i32>`, which does not implement the `Copy` trait
...
9  |         let handle = thread::spawn(move || {
   |                                    ^^^^^^^ value moved into closure here, in previous iteration of loop
10 |             let mut num = counter.lock().unwrap();
   |                           ------- use occurs due to use in closure

For more information about this error, try `rustc --explain E0382`.
error: could not compile `shared-state` due to previous error
```

错误信息表明 `counter` 值在上一次循环中被移动了。所以 Rust 告诉我们不能将 `counter` 锁的所有权移动到多个线程中。让我们通过一个第十五章讨论过的多所有权手段来修复这个编译错误。

### 多线程 和多所有权

在第十五章中，通过使用智能指针 `Rc<T>` 来创建引用计数的值，以便拥有多所有者。让我们在这也这么做看看会发生什么。

```rust
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

不幸的是，`Rc<T>` 并不能安全的在线程间共享。当 `Rc<T>` 管理引用计数时，它必须在每一个 `clone` 调用时增加计数，并在每一个克隆被丢弃时减少计数。`Rc<T>` 并没有使用任何并发原语，来确保改变计数的操作不会被其他线程打断。

### 原子引用计数 `Arc<T>`

**原子引用计数**（*atomically reference counted*）类型。

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

### `RefCell<T>/Rc<T>` 与 `Mutex<T>/Arc<T>` 的相似性

## 使用 Sync 和 Send trait 的 可扩展并发

两个并发概念是内嵌于语言中的：`std::marker` 中的 `Sync` 和 `Send` trait。

### 通过Send 允许在 线程间转移所有权

`Send` trait 表明实现了 `Send` 的类型值的所有权可以在线程间传送。几乎所有的 Rust 类型都是`Send` 的，不过有一些例外，包括 `Rc<T>`：这是不能 `Send` 的，因为如果克隆了 `Rc<T>` 的值并尝试将克隆的所有权转移到另一个线程，这两个线程都可能同时更新引用计数。为此，`Rc<T>` 被实现为用于单线程场景，这时不需要为拥有线程安全的引用计数而付出性能代价。

任何完全由 `Send` 的类型组成的类型也会自动被标记为 `Send`。几乎所有基本类型都是 `Send` 的，除了第十九章将会讨论的裸指针（raw pointer）。

### Sync 允许多线程访问

`Sync` trait 表明一个实现了 `Sync` 的类型可以安全的在多个线程中拥有其值的引用。换一种方式来说，对于任意类型 `T`，如果 `&T`（`T` 的不可变引用）是 `Send` 的话 `T` 就是 `Sync` 的，这意味着其引用就可以安全的发送到另一个线程。类似于 `Send` 的情况，基本类型是 `Sync` 的，完全由 `Sync` 的类型组成的类型也是 `Sync` 的。

### 手动实现 Send 和 Sync 是不安全的

通常并不需要手动实现 `Send` 和 `Sync` trait，因为由 `Send` 和 `Sync` 的类型组成的类型，自动就是 `Send` 和 `Sync` 的。因为他们是标记 trait，甚至都不需要实现任何方法。他们只是用来加强并发相关的不可变性的。

手动实现这些标记 trait 涉及到编写不安全的 Rust 代码，第十九章将会讲述具体的方法；当前重要的是，在创建新的由不是 `Send` 和 `Sync` 的部分构成的并发类型时需要多加小心，以确保维持其安全保证。[“The Rustonomicon”](https://doc.rust-lang.org/nomicon/index.html) 中有更多关于这些保证以及如何维持他们的信息。

## 总结

Rust 本身很少有处理并发的部分内容，有很多的并发方案都由 crate 实现。他们比标准库要发展的更快；请在网上搜索当前最新的用于多线程场景的 crate。

# Rust OOP

关于 OOP 是什么有很多相互矛盾的定义；在一些定义下，Rust 是面向对象的；在其他定义下，Rust 不是

## 面向对象语言的特征

### 对象包含数据和行为

由 Erich Gamma、Richard Helm、Ralph Johnson 和 John Vlissides（Addison-Wesley Professional, 1994）编写的书 *Design Patterns: Elements of Reusable Object-Oriented Software* 被俗称为 *The Gang of Four*

> Object-oriented programs are made up of objects. An *object* packages both data and the procedures that operate on that data. The procedures are typically called *methods* or *operations*.
> 
> 面向对象的程序是由对象组成的。一个 **对象** 包含数据和操作这些数据的过程。这些过程通常被称为 **方法** 或 **操作**。

在这个定义下，Rust 是面向对象的：结构体和枚举包含数据而 `impl` 块提供了在结构体和枚举之上的方法。虽然带有方法的结构体和枚举并不被 **称为** 对象，但是他们提供了与对象相同的功能，参考 *The Gang of Four* 中对象的定义。

### 封装隐藏了实现细节

另一个通常与面向对象编程相关的方面是 **封装**（*encapsulation*）的思想：对象的实现细节不能被使用对象的代码获取到。所以唯一与对象交互的方式是通过对象提供的公有 API；使用对象的代码无法深入到对象内部并直接改变数据或者行为。封装使得改变和重构对象的内部时无需改变使用对象的代码。

可以使用 `pub` 关键字来决定模块、类型、函数和方法是公有的，而默认情况下其他一切都是私有的。

```rust
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}
```

注意，结构体自身被标记为 `pub`，这样其他代码就可以使用这个结构体，但是在结构体内部的字段仍然是私有的。这是非常重要的，因为我们希望保证变量被增加到列表或者被从列表删除时，也会同时更新平均值。可以通过在结构体上实现 `add`、`remove` 和 `average` 方法来做到这一点

```rust
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}

impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            }
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
```

如果封装是一个语言被认为是面向对象语言所必要的方面的话，那么 Rust 满足这个要求。在代码中不同的部分使用 `pub` 与否可以封装其实现细节。

### 继承，作为类型系统与代码共享

**继承**（*Inheritance*）是一个很多编程语言都提供的机制，一个对象可以定义为继承另一个对象定义中的元素，这使其可以获得父对象的数据和行为，而无需重新定义。

如果一个语言必须有继承才能被称为面向对象语言的话，那么 ==Rust 就不是面向对象的==。因为没有宏则无法定义一个结构体继承父结构体的成员和方法。

选择继承有两个主要的原因。

- 第一个是为了重用代码：一旦为一个类型实现了特定行为，继承可以对一个不同的类型重用这个实现。
    - Rust 代码中可以使用默认 trait 方法实现来进行有限的共享
- 第二个使用继承的原因与类型系统有关：表现为子类型可以用于父类型被使用的地方。这也被称为 **多态**（*polymorphism*）
    - 近来继承作为一种语言设计的解决方案在很多语言中失宠了，因为其时常带有共享多于所需的代码的风险。子类不应总是共享其父类的所有特征，但是继承却始终如此。如此会使程序设计更为不灵活，并引入无意义的子类方法调用，或由于方法实际并不适用于子类而造成错误的可能性。

Rust 选择了一个不同的途径，使用 trait 对象而不是继承。让我们看一下 Rust 中的 trait 对象是如何==实现多态==的。

## 顾及不同类型值的 trait 对象

https://kaisery.github.io/trpl-zh-cn/ch17-02-trait-objects.html

场景： 图形界面GUI，所有的组件( Button`、`Image`和`SelectBox) 都需要 draw，所以可以定义一个 名为 Component 的类，在这个类上 有一个 draw 方法。其他的类，Button`、`Image`和`SelectBox 从 Component 派生， 继承 draw 方法。它们可以覆盖 draw方法，但是 框架会把所有这些类型都当作 Component的实例 并在 Component 的实例上 调用 draw。

Rust 并没有继承，我们得另寻出路。

### 定义通用行为的 trait

为了实现上面的场景， 我们定义一个 Draw trait，里面有 draw 方法。

然后定义一个 存放 trait 对象 的 vector。trait 对象指向一个实现了我们指定 trait 的类型的实例，以及一个用于在运行时查找该类型的 trait 方法的表。

我们通过指定某种指针来创建 trait 对象，例如 `&` 引用或 `Box<T>` 智能指针，还有 `dyn` keyword，以及指定相关的 trait（第十九章 [“动态大小类型和 `Sized` trait”](https://kaisery.github.io/trpl-zh-cn/ch19-04-advanced-types.html#%E5%8A%A8%E6%80%81%E5%A4%A7%E5%B0%8F%E7%B1%BB%E5%9E%8B%E5%92%8C-sized-trait) 部分会介绍 trait 对象必须使用指针的原因）。我们可以使用 trait 对象代替泛型或具体类型。任何使用 trait 对象的位置，Rust 的类型系统会在编译时确保任何在此上下文中使用的值会实现其 trait 对象的 trait。如此便无需在编译时就知晓所有可能的类型。

之前提到过，Rust 刻意不将结构体与枚举称为 “对象”，以便与其他语言中的对象相区别。在结构体或枚举中，结构体字段中的数据和 `impl` 块中的行为是分开的，不同于其他语言中将数据和行为组合进一个称为对象的概念中。trait 对象将数据和行为两者相结合，从这种意义上说 **则** 其更类似其他语言中的对象。不过 trait 对象不同于传统的对象，因为不能向 trait 对象增加数据。trait 对象并不像其他语言中的对象那么通用：其（trait 对象）具体的作用是允许对通用行为进行抽象。

定义一个带有 `draw` 方法的 trait `Draw`

```rust
pub trait Draw {
    fn draw(&self);
}
```

定义了一个存放了名叫 `components` 的 vector 的结构体 `Screen`。这个 vector 的类型是 `Box<dyn Draw>`，此为一个 trait 对象：它是 `Box` 中任何实现了 `Draw` trait 的类型的替身。

```rust
pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}
```

在 `Screen` 结构体上，我们将定义一个 `run` 方法，该方法会对其 `components` 上的每一个组件调用 `draw` 方法

```rust
impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

这与定义使用了带有 trait bound 的泛型类型参数的结构体不同。泛型类型参数一次只能替代一个具体类型，而 trait 对象则允许在运行时替代多种具体类型。例如，可以定义 `Screen` 结构体来使用泛型和 trait bound

```rust
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

impl<T> Screen<T>
where
    T: Draw,
{
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

这限制了 `Screen` 实例必须拥有一个全是 `Button` 类型或者全是 `TextField` 类型的组件列表。如果只需要同质（相同类型）集合，则倾向于使用泛型和 trait bound，因为其定义会在编译时采用具体类型进行单态化。

另一方面，通过使用 trait 对象的方法，一个 `Screen` 实例可以存放一个既能包含 `Box<Button>`，也能包含 `Box<TextField>` 的 `Vec<T>`。让我们看看它是如何工作的，接着会讲到其运行时性能影响。

### 实现 trait

。。g。

## 面向对象设计模式的实现

https://kaisery.github.io/trpl-zh-cn/ch17-03-oo-design-patterns.html

**状态模式**（*state pattern*）是一个面向对象设计模式。该模式的关键在于定义一系列值的内含状态。这些状态体现为一系列的 **状态对象**，同时值的行为随着其内部状态而改变。我们将编写一个博客发布结构体的例子，它拥有一个包含其状态的字段，这是一个有着 "draft"、"review" 或 "published" 的状态对象

# 模式与模式匹配

**模式**（*Patterns*）是 Rust 中特殊的语法，它用来匹配类型中的结构，无论类型是简单还是复杂。结合使用模式和 `match` 表达式以及其他结构可以提供更多对程序控制流的支配权。模式由如下一些内容组合而成：

- 字面值
- 解构的数组、枚举、结构体或者元组
- 变量
- 通配符
- 占位符

## 所有可能会用到模式的位置

### match 分支

一个模式常用的位置是 `match` 表达式的分支。在形式上 `match` 表达式由 `match` 关键字、用于匹配的值和一个或多个分支构成，这些分支包含一个模式和在值匹配分支的模式时运行的表达式：

```rust
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

`match` 表达式必须是 **穷尽**（*exhaustive*）的，意为 `match` 表达式所有可能的值都必须被考虑到。

一个特定的模式 `_` 可以匹配所有情况，不过它从不绑定任何变量。

### if let 条件表达式

主要用于编写等同于只关心一个情况的 `match` 语句简写的。`if let` 可以对应一个可选的带有代码的 `else` 在 `if let` 中的模式不匹配时运行。

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {color}, as the background");
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

如果用户指定了中意的颜色，将使用其作为背景颜色。如果没有指定中意的颜色且今天是星期二，背景颜色将是绿色。如果用户指定了他们的年龄字符串并能够成功将其解析为数字的话，我们将根据这个数字使用紫色或者橙色。最后，如果没有一个条件符合，背景颜色将是蓝色。

这个条件结构允许我们支持复杂的需求。使用这里硬编码的值，例子会打印出 `Using purple as the background color`。

注意 `if let` 也可以像 `match` 分支那样引入覆盖变量：`if let Ok(age) = age` 引入了一个新的覆盖变量 `age`，它包含 `Ok` 成员中的值。这意味着 `if age > 30` 条件需要位于这个代码块内部；不能将两个条件组合为 `if let Ok(age) = age && age > 30`，因为我们希望与 30 进行比较的被覆盖的 `age` 直到大括号开始的新作用域才是有效的。

`if let` 表达式的缺点在于其穷尽性没有为编译器所检查，而 `match` 表达式则检查了。如果去掉最后的 `else` 块而遗漏处理一些情况，编译器也不会警告这类可能的逻辑错误。

### while let 条件循环

```rust
fn main() {
    let mut stack = Vec::new();

    stack.push(1);
    stack.push(2);
    stack.push(3);

    while let Some(top) = stack.pop() {
        println!("{}", top);
    }
}
```

使用 `while let` 循环只要 `stack.pop()` 返回 `Some` 就打印出其值

### for 循环

```rust
fn main() {
    let v = vec!['a', 'b', 'c'];

    for (index, value) in v.iter().enumerate() {
        println!("{} is at index {}", value, index);
    }
}
```

### let

```rust
let x = 5;
```

不过你可能没有发觉，每一次像这样使用 `let` 语句就是在使用模式！`let` 语句更为正式的样子如下：

```rust
let PATTERN = EXPRESSION;
```

```rust
fn main() {
    let (x, y, z) = (1, 2, 3);
}
```

如果模式中元素的数量不匹配元组中元素的数量，则整个类型不匹配，并会得到一个编译时错误

> 为了修复这个错误，可以使用 `_` 或 `..` 来忽略元组中一个或多个值

### 函数参数

```rust
fn foo(x: i32) {
    // code goes here
}
```

`x` 部分就是一个模式！类似于之前对 `let` 所做的，可以在函数参数中匹配元组。

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

> 因为如第十三章所讲闭包类似于函数，也可以在闭包参数列表中使用模式。

## Refutability ( 可反驳性)： 模式是否会匹配失效

模式有两种形式：refutable（可反驳的）和 irrefutable（不可反驳的）。

能匹配任何传递的可能值的模式被称为是 **不可反驳的**（*irrefutable*）。一个例子就是 `let x = 5;` 语句中的 `x`，因为 `x` 可以匹配任何值所以不可能会失败。

对某些可能的值进行匹配会失败的模式被称为是 **可反驳的**（*refutable*）。一个这样的例子便是 `if let Some(x) = a_value` 表达式中的 `Some(x)`；如果变量 `a_value` 中的值是 `None` 而不是 `Some`，那么 `Some(x)` 模式不能匹配。

函数参数、 `let` 语句和 `for` 循环只能接受不可反驳的模式，因为通过不匹配的值程序无法进行有意义的工作。

`if let` 和 `while let` 表达式被限制为只能接受可反驳的模式，因为根据定义他们意在处理可能的失败：条件表达式的功能就是根据成功或失败执行不同的操作。

## 所有 模式语法

### 匹配字面值

```rust
fn main() {
    let x = 1;

    match x {
        1 => println!("one"),
        2 => println!("two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
}
```

### 匹配命名变量

命名变量是匹配任何值的不可反驳模式，这在之前已经使用过数次。然而当其用于 `match` 表达式时情况会有些复杂。因为 `match` 会开始一个新作用域，`match` 表达式中作为模式的一部分声明的变量会覆盖 `match` 结构之外的同名变量，与所有变量一样。

在示例 18-11 中，声明了一个值为 `Some(5)` 的变量 `x` 和一个值为 `10` 的变量 `y`。接着在值 `x` 上创建了一个 `match` 表达式。观察匹配分支中的模式和结尾的 `println!`，并在运行此代码或进一步阅读之前推断这段代码会打印什么。

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {y}"),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {y}", x);
}
```

让我们看看当 `match` 语句运行的时候发生了什么。第一个匹配分支的模式并不匹配 `x` 中定义的值，所以代码继续执行。

第二个匹配分支中的模式引入了一个新变量 `y`，它会匹配任何 `Some` 中的值。因为我们在 `match` 表达式的新作用域中，这是一个新变量，而不是开头声明为值 10 的那个 `y`。这个新的 `y` 绑定会匹配任何 `Some` 中的值，在这里是 `x` 中的值。因此这个 `y` 绑定了 `x` 中 `Some` 内部的值。这个值是 5，所以这个分支的表达式将会执行并打印出 `Matched, y = 5`。

如果 `x` 的值是 `None` 而不是 `Some(5)`，头两个分支的模式不会匹配，所以会匹配下划线。这个分支的模式中没有引入变量 `x`，所以此时表达式中的 `x` 会是外部没有被覆盖的 `x`。在这个假想的例子中，`match` 将会打印 `Default case, x = None`。

一旦 `match` 表达式执行完毕，其作用域也就结束了，同理内部 `y` 的作用域也结束了。最后的 `println!` 会打印 `at the end: x = Some(5), y = 10`。

为了创建能够比较外部 `x` 和 `y` 的值，而不引入覆盖变量的 `match` 表达式，我们需要相应地使用带有条件的匹配守卫（match guard）。我们稍后将在 [“匹配守卫提供的额外条件”](https://kaisery.github.io/trpl-zh-cn/ch18-03-pattern-syntax.html#%E5%8C%B9%E9%85%8D%E5%AE%88%E5%8D%AB%E6%8F%90%E4%BE%9B%E7%9A%84%E9%A2%9D%E5%A4%96%E6%9D%A1%E4%BB%B6) 这一小节讨论匹配守卫。

### 多个模式

在 `match` 表达式中，可以使用 `|` 语法匹配多个模式，它代表 **或**（*or*）运算符模式。例如，如下代码将 `x` 的值与匹配分支相比较，第一个分支有 **或** 选项，意味着如果 `x` 的值匹配此分支的任一个值，它就会运行：

```rust
fn main() {
    let x = 1;

    match x {
        1 | 2 => println!("one or two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
}
```

### 通过 ..= 匹配值的范围

`..=` 语法允许你匹配一个闭区间范围内的值。在如下代码中，当模式匹配任何在给定范围内的值时，该分支会执行：

```rust
fn main() {
    let x = 5;

    match x {
        1..=5 => println!("one through five"),
        _ => println!("something else"),
    }
}
```

如果 `x` 是 1、2、3、4 或 5，第一个分支就会匹配。这个语法在匹配多个值时相比使用 `|` 运算符来表达相同的意思更为方便；如果使用 `|` 则不得不指定 `1 | 2 | 3 | 4 | 5`。相反指定范围就简短的多，特别是在希望匹配比如从 1 到 1000 的数字的时候！

编译器会在编译时检查范围不为空，而 `char` 和数字值是 Rust 仅有的可以判断范围是否为空的类型，所以范围只允许用于数字或 `char` 值。

```rust
fn main() {
    let x = 'c';

    match x {
        'a'..='j' => println!("early ASCII letter"),
        'k'..='z' => println!("late ASCII letter"),
        _ => println!("something else"),
    }
}
```

### 解构并分解值

可以使用模式来解构结构体、枚举和元组，以便使用这些值的不同部分。让我们来分别看一看。

#### 解构结构体

带有两个字段 `x` 和 `y` 的结构体 `Point`，可以通过带有模式的 `let` 语句将其分解：

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;  // <<<<<<<
    assert_eq!(0, a);
    assert_eq!(7, b);
}
```

不过通常希望变量名与字段名一致以便于理解变量来自于哪些字段。因为变量名匹配字段名是常见的，同时因为 `let Point { x: x, y: y } = p;` 包含了很多重复，所以对于匹配结构体字段的模式存在简写：只需列出结构体字段的名称，则模式创建的变量会有相同的名称。

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

一个 `match` 语句将 `Point` 值分成了三种情况：直接位于 `x` 轴上（此时 `y = 0` 为真）、位于 `y` 轴上（`x = 0`）或不在任何轴上的点。

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {x}"),
        Point { x: 0, y } => println!("On the y axis at {y}"),
        Point { x, y } => {
            println!("On neither axis: ({x}, {y})");
        }
    }
}
```

#### 解构枚举

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.");
        }
        Message::Move { x, y } => {
            println!("Move in the x direction {x} and in the y direction {y}");
        }
        Message::Write(text) => {
            println!("Text message: {text}");
        }
        Message::ChangeColor(r, g, b) => {
            println!("Change the color to red {r}, green {g}, and blue {b}",)
        }
    }
}
```

#### 解构嵌套的结构体和枚举

```rust
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!("Change color to red {r}, green {g}, and blue {b}");
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!("Change color to hue {h}, saturation {s}, value {v}")
        }
        _ => (),
    }
}
```

#### 解构结构体和元组

```rust
fn main() {
    struct Point {
        x: i32,
        y: i32,
    }

    let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
}
```

### 忽略模式中的值

#### 使用 `_` 忽略整个值

虽然这作为 `match` 表达式最后的分支特别有用，也可以将其用于任意模式，包括函数参数中

```rust
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}

fn main() {
    foo(3, 4);
}
```

这段代码会完全忽略作为第一个参数传递的值 `3`，并会打印出 `This code only uses the y parameter: 4`。

大部分情况当你不再需要特定函数参数时，最好修改签名不再包含无用的参数。在一些情况下忽略函数参数会变得特别有用，比如实现 trait 时，当你需要特定类型签名但是函数实现并不需要某个参数时。这样可以避免一个存在未使用的函数参数的编译警告，就跟使用命名参数一样。

#### 使用嵌套的 `_` 忽略部分值

可以在一个模式内部使用`_` 忽略部分值，例如，当只需要测试部分值但在期望运行的代码中没有用到其他部分时。示例 18-18 展示了负责管理设置值的代码。业务需求是用户不允许覆盖现有的自定义设置，但是可以取消设置，也可以在当前未设置时为其提供设置。

```rust
fn main() {
    let mut setting_value = Some(5);
    let new_setting_value = Some(10);

    match (setting_value, new_setting_value) {
        (Some(_), Some(_)) => {
            println!("Can't overwrite an existing customized value");
        }
        _ => {
            setting_value = new_setting_value;
        }
    }
    println!("setting is {:?}", setting_value);
}
```

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, _, third, _, fifth) => {
            println!("Some numbers: {first}, {third}, {fifth}")
        }
    }
}
```

#### 在名字前面以一个`_`开头来忽略 未使用的变量

如果你创建了一个变量却不在任何地方使用它，Rust 通常会给你一个警告，因为未使用的变量可能会是个 bug。但是有时创建一个还未使用的变量是有用的，比如你正在设计原型或刚刚开始一个项目。这时你希望告诉 Rust 不要警告未使用的变量，为此可以用下划线作为变量名的开头。示例 18-20 中创建了两个未使用变量，不过当编译代码时只会得到其中一个的警告：

```rust
fn main() {
    let _x = 5;
    let y = 10;
}
```

只使用 `_` 和使用以下划线开头的名称有些微妙的不同：比如 `_x` 仍会将值绑定到变量，而 `_` 则完全不会绑定。

示例 18-21 会产生一个错误

```rust
fn main() {
    let s = Some(String::from("Hello!"));
    if let Some(_s) = s {
        println!("found a string");
    }
    println!("{:?}", s);
}
```

我们会得到一个错误，因为 `s` 的值仍然会移动进 `_s`，并阻止我们再次使用 `s`。然而只使用下划线本身，并不会绑定值。

#### 用 `..` 忽略剩余值

`..` 模式会忽略模式中剩余的任何没有显式匹配的值部分。

```rust
fn main() {
    struct Point {
        x: i32,
        y: i32,
        z: i32,
    }
    let origin = Point { x: 0, y: 0, z: 0 };
    match origin {
        Point { x, .. } => println!("x is {}", x),
    }
}
```

`..` 会扩展为所需要的值的数量。

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {first}, {last}");
        }
    }
}
```

### 匹配守卫 提供的额外条件

**匹配守卫**（*match guard*）是一个指定于 `match` 分支模式之后的额外 `if` 条件，它也必须被满足才能选择此分支。匹配守卫用于表达比单独的模式所能允许的更为复杂的情况。

这个条件可以使用模式中创建的变量。示例 18-26 展示了一个 `match`，其中第一个分支有模式 `Some(x)` 还有匹配守卫 `if x % 2 == 0` (当 `x` 是偶数的时候为真)：

```rust
fn main() {
    let num = Some(4);

    match num {
        Some(x) if x % 2 == 0 => println!("The number {} is even", x),
        Some(x) => println!("The number {} is odd", x),
        None => (),
    }
}
```

无法在模式中表达类似 `if x % 2 == 0` 的条件，所以通过匹配守卫提供了表达类似逻辑的能力。这种替代表达方式的缺点是，编译器不会尝试为包含匹配守卫的模式检查穷尽性。

在示例 18-11 中，我们提到可以使用匹配守卫来解决模式中变量覆盖的问题，那里 `match` 表达式的模式中新建了一个变量而不是使用 `match` 之外的同名变量。新变量意味着不能够测试外部变量的值。示例 18-27 展示了如何使用匹配守卫修复这个问题。

。。就是要和一个变量进行比较，如果直接 Some(y)。那么这个y 是新的变量，不是 需要进行比较的变量。

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {n}"),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {y}", x);
}
```

可以在匹配守卫中使用 **或** 运算符 `|` 来指定多个模式，同时匹配守卫的条件会作用于所有的模式。示例 18-28 展示了结合匹配守卫与使用了 `|` 的模式的优先级。这个例子中重要的部分是匹配守卫 `if y` 作用于 `4`、`5` **和** `6`，即使这看起来好像 `if y` 只作用于 `6`：

```rust
fn main() {
    let x = 4;
    let y = false;

    match x {
        4 | 5 | 6 if y => println!("yes"),
        _ => println!("no"),
    }
}
```

。。不知道能不能 (4 | 5 | 6) if y 。。 要是可以的话，就清晰很多

### @绑定

*at* 运算符（`@`）允许我们在创建一个存放值的变量的同时测试其值是否匹配模式。示例 18-29 展示了一个例子，这里我们希望测试 `Message::Hello` 的 `id` 字段是否位于 `3..=7` 范围内，同时也希望能将其值绑定到 `id_variable` 变量中以便此分支相关联的代码可以使用它。可以将 `id_variable` 命名为 `id`，与字段同名，不过出于示例的目的这里选择了不同的名称。

```rust
fn main() {
    enum Message {
        Hello { id: i32 },
    }

    let msg = Message::Hello { id: 5 };

    match msg {
        Message::Hello {
            id: id_variable @ 3..=7,
        } => println!("Found an id in range: {}", id_variable),
        Message::Hello { id: 10..=12 } => {
            println!("Found an id in another range")
        }
        Message::Hello { id } => println!("Found some other id: {}", id),
    }
}
```

通过在 `3..=7` 之前指定 `id_variable @`，我们捕获了任何匹配此范围的值并同时测试其值匹配这个范围模式。

使用 `@` 可以在一个模式中同时测试和保存变量值。

# 高级特征

涉及如下内容：

- 不安全 Rust：用于当需要舍弃 Rust 的某些保证并负责手动维持这些保证
- 高级 trait：与 trait 相关的关联类型，默认类型参数，完全限定语法（fully qualified syntax），超（父）trait（supertraits）和 newtype 模式
- 高级类型：关于 newtype 模式的更多内容，类型别名，never 类型和动态大小类型
- 高级函数和闭包：函数指针和返回闭包
- 宏：定义在编译时定义更多代码的方式

## 不安全 Rust

https://kaisery.github.io/trpl-zh-cn/ch19-01-unsafe-rust.html

通过 `unsafe` 关键字来切换到不安全 Rust

有五类可以在不安全 Rust 中进行而不能用于安全 Rust 的操作，它们称之为 “不安全的超能力。（*unsafe superpowers*）” 这些超能力是：

- 解引用裸指针
- 调用不安全的函数或方法
- 访问或修改可变静态变量
- 实现不安全 trait
- 访问 `union` 的字段

`unsafe` 并不会关闭借用检查器或禁用任何其他 Rust 安全检查：如果在不安全代码中使用引用，它仍会被检查。`unsafe` 关键字只是提供了那五个不会被编译器检查内存安全的功能。你仍然能在不安全块中获得某种程度的安全。

### 解引用裸指针

回到第四章的 \[“悬垂引用”\]\[dangling-references\] 部分，那里提到了编译器会确保引用总是有效的。不安全 Rust 有两个被称为 **裸指针**（*raw pointers*）的类似于引用的新类型。和引用一样，裸指针是不可变或可变的，分别写作 ==`*const T` 和 `*mut T`==。这里的星号==不是解引用运算符==；它是类型名称的一部分。在裸指针的上下文中，**不可变** 意味着指针解引用之后不能直接赋值。

裸指针与引用和智能指针的区别在于

- 允许忽略借用规则，可以同时拥有不可变和可变的指针，或多个指向相同位置的可变指针
- 不保证指向有效的内存
- 允许为空
- 不能实现任何自动清理功能

通过去掉 Rust 强加的保证，你可以放弃安全保证以换取性能或使用另一个语言或硬件接口的能力，此时 Rust 的保证并不适用。

```rust
fn main() {
    let mut num = 5;

    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
}
```

注意这里没有引入 `unsafe` 关键字。可以在安全代码中 **创建** 裸指针，只是不能在不安全块之外 **解引用** 裸指针，稍后便会看到。

使用 `as` 将不可变和可变引用强转为对应的裸指针类型。因为直接从保证安全的引用来创建他们，可以知道这些特定的裸指针是有效，但是不能对任何裸指针做出如此假设。

创建一个不能确定其有效性的裸指针，示例 19-2 展示了如何创建一个**指向任意内存地址**的裸指针。尝试使用任意内存是未定义行为：此地址可能有数据也可能没有，编译器可能会优化掉这个内存访问，或者程序可能会出现段错误（segmentation fault）。通常没有好的理由编写这样的代码，不过却是可行的：

```rust
fn main() {
    let address = 0x012345usize;
    let r = address as *const i32;
}
```

可以在安全代码中创建裸指针，不过不能 **解引用** 裸指针和读取其指向的数据。现在我们要做的就是对裸指针使用解引用运算符 `*`，这需要一个 `unsafe` 块

```rust
fn main() {
    let mut num = 5;
    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
    unsafe {
        println!("r1 is: {}", *r1);
        println!("r2 is: {}", *r2);
    }
}
```

上面创建了同时指向相同内存位置 `num` 的裸指针 `*const i32` 和 `*mut i32`。相反如果尝试同时创建 `num` 的不可变和可变引用，将无法通过编译，因为 Rust 的所有权规则不允许在拥有任何不可变引用的同时再创建一个可变引用。通过裸指针，就能够同时创建同一地址的可变指针和不可变指针，若通过可变指针修改数据，则可能潜在造成数据竞争。请多加小心！

为何还要使用裸指针呢？

- 一个主要的应用场景便是调用 C 代码接口，这在下一部分 [“调用不安全函数或方法”](https://kaisery.github.io/trpl-zh-cn/ch19-01-unsafe-rust.html#calling-an-unsafe-function-or-method) 中会讲到。
- 另一个场景是构建借用检查器无法理解的安全抽象。

### 调用不安全函数或方法

第二类可以在进行不安全块的操作是调用不安全函数。不安全函数和方法与常规函数方法十分类似，除了其开头有一个额外的 `unsafe`。在此上下文中，关键字`unsafe`表示该函数具有调用时需要满足的要求，而 Rust 不会保证满足这些要求。通过在 `unsafe` 块中调用不安全函数，表明我们已经阅读过此函数的文档并对其是否满足函数自身的契约负责。

```rust
fn main() {
    unsafe fn dangerous() {}

    unsafe {
        dangerous();
    }
}
```

不安全函数体也是有效的 `unsafe` 块，所以在不安全函数中进行另一个不安全操作时无需新增额外的 `unsafe` 块。

### 创建不安全代码的安全抽象

仅仅因为函数包含不安全代码并不意味着整个函数都需要标记为不安全的。事实上，将不安全代码封装进安全函数是一个常见的抽象。作为一个例子，了解一下标准库中的函数 `split_at_mut`，它需要一些不安全代码，让我们探索如何可以实现它。这个安全函数定义于可变 slice 之上：它获取一个 slice 并从给定的索引参数开始将其分为两个 slice。`split_at_mut` 的用法如示例 19-4 所示：

```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5, 6];

    let r = &mut v[..];

    let (a, b) = r.split_at_mut(3);

    assert_eq!(a, &mut [1, 2, 3]);
    assert_eq!(b, &mut [4, 5, 6]);
}
```

出于简单考虑，我们将 `split_at_mut` 实现为函数而不是方法，并只处理 `i32` 值而非泛型 `T` 的 slice。

```rust
fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();
    assert!(mid <= len);
    (&mut values[..mid], &mut values[mid..])
}
```

编译报错：

```rust
$ cargo run
   Compiling unsafe-example v0.1.0 (file:///projects/unsafe-example)
error[E0499]: cannot borrow `*values` as mutable more than once at a time
 --> src/main.rs:6:31
  |
1 | fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
  |                         - let's call the lifetime of this reference `'1`
...
6 |     (&mut values[..mid], &mut values[mid..])
  |     --------------------------^^^^^^--------
  |     |     |                   |
  |     |     |                   second mutable borrow occurs here
  |     |     first mutable borrow occurs here
  |     returning this value requires that `*values` is borrowed for `'1`

For more information about this error, try `rustc --explain E0499`.
error: could not compile `unsafe-example` due to previous error
```

Rust 的借用检查器不能理解我们要借用这个 slice 的两个不同部分：它只知道我们借用了同一个 slice 两次。本质上借用 slice 的不同部分是可以的，因为结果两个 slice 不会重叠，不过 Rust 还没有智能到能够理解这些。当我们知道某些事是可以的而 Rust 不知道的时候，就是触及不安全代码的时候了

使用 `unsafe` 块，裸指针和一些不安全函数调用来实现 `split_at_mut`：

```rust
use std::slice;
fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();
    let ptr = values.as_mut_ptr();
    assert!(mid <= len);
    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
```

slice 是一个指向一些数据的指针，并带有该 slice 的长度。可以使用 len 方法获取 slice 的长度，使用 as_mut_ptr 方法访问 slice 的裸指针。在这个例子中，因为有一个 i32 值的可变 slice，as_mut_ptr 返回一个 *mut i32 类型的裸指针，储存在 ptr 变量中。

我们保持索引 mid 位于 slice 中的断言。接着是不安全代码：slice::from_raw_parts_mut 函数获取一个裸指针和一个长度来创建一个 slice。这里使用此函数从 ptr 中创建了一个有 mid 个项的 slice。之后在 ptr 上调用 add 方法并使用 mid 作为参数来获取一个从 mid 开始的裸指针，使用这个裸指针并以 mid 之后项的数量为长度创建一个 slice。


slice::from_raw_parts_mut 函数是不安全的因为它获取一个裸指针，并必须确信这个指针是有效的。裸指针上的 add 方法也是不安全的，因为其必须确信此地址偏移量也是有效的指针。因此必须将 slice::from_raw_parts_mut 和 add 放入 unsafe 块中以便能调用它们。通过观察代码，和增加 mid 必然小于等于 len 的断言，我们可以说 unsafe 块中所有的裸指针将是有效的 slice 中数据的指针。这是一个可以接受的 unsafe 的恰当用法。

无需将 split_at_mut 函数的结果标记为 unsafe，并可以在安全 Rust 中调用此函数。我们创建了一个不安全代码的安全抽象，其代码以一种安全的方式使用了 unsafe 代码，因为其只从这个函数访问的数据中创建了有效的指针。


下面的代码中 slice::from_raw_parts_mut 在使用 slice 时很有可能会崩溃。这段代码获取任意内存地址并创建了一个长为一万的 slice：
```rust
fn main() {
    use std::slice;

    let address = 0x01234usize;
    let r = address as *mut i32;

    let values: &[i32] = unsafe { slice::from_raw_parts_mut(r, 10000) };
}
```

我们并不拥有这个任意地址的内存，也不能保证这段代码创建的 slice 包含有效的 i32 值。试图使用臆测为有效的 values 会导致==未定义的行为==。


### 使用 extern 函数调用外部代码

Rust 有一个关键字，extern，有助于创建和使用 外部函数接口（Foreign Function Interface，FFI）。外部函数接口是一个编程语言用以定义函数的方式，其允许不同（外部）编程语言调用这些函数。

展示了如何集成 C 标准库中的 abs 函数。extern 块中声明的函数在 Rust 代码中总是不安全的。因为其他语言不会强制执行 Rust 的规则且 Rust 无法检查它们，所以确保其安全是程序员的责任：

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

在 extern "C" 块中，列出了我们希望能够调用的另一个语言中的外部函数的签名和名称。"C" 部分定义了外部函数所使用的 应用二进制接口（application binary interface，ABI） —— ABI 定义了如何在汇编语言层面调用此函数。"C" ABI 是最常见的，并遵循 C 编程语言的 ABI。



#### 从其他语言调用 Rust 函数

使用 extern 来创建一个允许其他语言调用 Rust 函数的接口。不同于创建整个 extern 块，就在 fn 关键字之前增加 extern 关键字并为相关函数指定所用到的 ABI。还需增加 #[no_mangle] 注解来告诉 Rust 编译器不要 mangle 此函数的名称。

在如下的例子中，一旦其编译为动态库并从 C 语言中链接，call_from_c 函数就能够在 C 代码中访问：

```rust
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```
extern 的使用无需 unsafe。


### 访问或修改可变静态变量

目前为止全书都尽量避免讨论 全局变量（global variables），Rust 确实支持他们，不过这对于 Rust 的所有权规则来说是有问题的。如果有两个线程访问相同的可变全局变量，则可能会造成数据竞争。

全局变量在 Rust 中被称为 静态（static）变量。

```rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
```

静态（static）变量类似于第三章 [“变量和常量的区别”][differences-between-variables-and-constants] 部分讨论的常量。通常静态变量的名称采用 SCREAMING_SNAKE_CASE 写法。静态变量只能储存拥有 'static 生命周期的引用，这意味着 Rust 编译器可以自己计算出其生命周期而无需显式标注。访问不可变静态变量是安全的。

常量与不可变静态变量的一个微妙的区别是静态变量中的值有一个固定的内存地址。使用这个值总是会访问相同的地址。另一方面，常量则允许在任何被用到的时候复制其数据。另一个区别在于静态变量可以是可变的。访问和修改可变静态变量都是 不安全 的。

```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

就像常规变量一样，我们使用 mut 关键来指定可变性。任何读写 COUNTER 的代码都必须位于 unsafe 块中。

拥有可以全局访问的可变数据，难以保证不存在数据竞争，这就是为何 Rust 认为可变静态变量是不安全的。任何可能的情况，请优先使用第十六章讨论的并发技术和线程安全智能指针，这样编译器就能检测不同线程间的数据访问是否是安全的。


### 实现不安全trait

unsafe 的另一个操作用例是实现不安全 trait。当 trait 中至少有一个方法中包含编译器无法验证的不变式（invariant）时 trait 是不安全的。可以在 trait 之前增加 unsafe 关键字将 trait 声明为 unsafe，同时 trait 的实现也必须标记为 unsafe

```rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}

fn main() {}
```

通过 unsafe impl，我们承诺将保证编译器所不能验证的不变量。

作为一个例子，回忆第十六章 [“使用 Sync 和 Send trait 的可扩展并发”][extensible-concurrency-with-the-sync-and-send-traits] 部分中的 Sync 和 Send 标记 trait，编译器会自动为完全由 Send 和 Sync 类型组成的类型自动实现他们。如果实现了一个包含一些不是 Send 或 Sync 的类型，比如裸指针，并希望将此类型标记为 Send 或 Sync，则必须使用 unsafe。Rust 不能验证我们的类型保证可以安全的跨线程发送或在多线程间访问，所以需要我们自己进行检查并通过 unsafe 表明。


### 访问联合体中的字段

仅适用于 unsafe 的最后一个操作是访问 联合体 中的字段，union 和 struct 类似，但是在一个实例中同时只能使用一个声明的字段。联合体主要用于和 C 代码中的联合体交互。访问联合体的字段是不安全的，因为 Rust 无法保证当前存储在联合体实例中数据的类型。可以查看 [参考 Rust 文档][reference] 了解有关联合体的更多信息。


### 如何使用不安全代码

使用 unsafe 来进行这五个操作（超能力）之一是没有问题的，甚至是不需要深思熟虑的，不过使得 unsafe 代码正确也实属不易，因为编译器不能帮助保证内存安全。当有理由使用 unsafe 代码时，是可以这么做的，通过使用显式的 unsafe 标注可以更容易地在错误发生时追踪问题的源头。



## 高级trait

### 关联类型在 trait 定义中指定占位符类型

关联类型（associated types）是一个将类型占位符与 trait 相关联的方式，这样 trait 的方法签名中就可以使用这些占位符类型。trait 的实现者会针对特定的实现在这个占位符类型指定相应的具体类型。如此可以定义一个使用多种类型的 trait，直到实现此 trait 时都无需知道这些类型具体是什么。

本章所描述的大部分内容都非常少见。关联类型则比较适中；它们比本书其他的内容要少见，不过比本章中的很多内容要更常见。

。。想跳过了。 感觉是 模板 or 泛型

一个带有关联类型的 trait 的例子是标准库提供的 Iterator trait。它有一个叫做 Item 的关联类型来替代遍历的值的类型。Iterator trait 的定义如示例 19-12 所示：
```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

Item 是一个占位符类型，同时 next 方法定义表明它返回 `Option<Self::Item>` 类型的值。这个 trait 的实现者会指定 Item 的具体类型，然而不管实现者指定何种类型，next 方法都会返回一个包含了此具体类型值的 Option。

关联类型看起来像一个类似泛型的概念，因为它允许定义一个函数而不指定其可以处理的类型。让我们通过在一个 Counter 结构体上实现 Iterator trait 的例子来检视其中的区别。这个实现中指定了 Item 的类型为 u32：

```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
        if self.count < 5 {
            self.count += 1;
            Some(self.count)
        } else {
            None
        }
    }
}
```

这个语法类似于泛型。那么为什么 Iterator trait 不像示例 19-13 那样定义呢？
```rust
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```

区别在于当如示例 19-13 那样使用泛型时，则不得不在每一个实现中标注类型。这是因为我们也可以实现为 Iterator<String> for Counter，或任何其他类型，这样就可以有多个 Counter 的 Iterator 的实现。换句话说，当 trait 有泛型参数时，可以多次实现这个 trait，每次需改变泛型参数的具体类型。接着当使用 Counter 的 next 方法时，必须提供类型注解来表明希望使用 Iterator 的哪一个实现。

通过关联类型，则无需标注类型，因为不能多次实现这个 trait。对于示例 19-12 使用关联类型的定义，我们只能选择一次 Item 会是什么类型，因为只能有一个 impl Iterator for Counter。当调用 Counter 的 next 时不必每次指定我们需要 u32 值的迭代器。

关联类型也会成为 trait 契约的一部分：trait 的实现必须提供一个类型来替代关联类型占位符。关联类型通常有一个描述类型用途的名字，并且在 API 文档中为关联类型编写文档是一个最佳实践。


### 默认泛型类型参数和运算符重载

当使用泛型类型参数时，可以为泛型指定一个默认的具体类型。
如果默认类型就足够的话，这消除了为具体类型实现 trait 的需要。
为泛型类型指定默认类型的语法是在声明泛型类型时使用 `<PlaceholderType=ConcreteType>`。

Rust 并不允许创建自定义运算符或重载任意运算符，
不过 std::ops 中所列出的运算符和相应的 trait 可以通过实现运算符相关 trait 来重载。

下面展示了如何在 Point 结构体上实现 Add trait 来重载 + 运算符，这样就可以将两个 Point 实例相加了：

```rust
use std::ops::Add;

#[derive(Debug, Copy, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(
        Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
        Point { x: 3, y: 3 }
    );
}
```

这里默认泛型类型位于 Add trait 中。这里是其定义：
```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

这是一个带有一个方法和一个关联类型的 trait。

尖括号中的 Rhs=Self：这个语法叫做 默认类型参数（default type parameters）。Rhs 是一个泛型类型参数（“right hand side” 的缩写），它用于定义 add 方法中的 rhs 参数。

如果实现 Add trait 时不指定 Rhs 的具体类型，Rhs 的类型将是默认的 Self 类型，也就是在其上实现 Add 的类型。


这里有两个存放不同单元值的结构体，Millimeters 和 Meters。（这种将现有类型简单封装进另一个结构体的方式被称为 newtype 模式（newtype pattern，之后的 “为了类型安全和抽象而使用 newtype 模式” 部分会详细介绍。）我们希望能够将毫米值与米值相加，并让 Add 的实现正确处理转换。可以为 Millimeters 实现 Add 并以 Meters 作为 Rhs

```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```
为了使 Millimeters 和 Meters 能够相加，我们指定 impl Add<Meters> 来设定 Rhs 类型参数的值而不是使用默认的 Self。

默认参数类型主要用于如下两个方面：

    扩展类型而不破坏现有代码。
    在大部分用户都不需要的特定情况进行自定义。



### 完全限定语法与消歧义：调用相同名称的方法

Rust 既不能避免一个 trait 与另一个 trait 拥有相同名称的方法，也不能阻止为同一类型同时实现这两个 trait。甚至直接在类型上实现开始已经有的同名方法也是可能的！

当调用这些同名方法时，需要告诉 Rust 我们希望使用哪一个

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}
```

当调用 Human 实例的 fly 时，编译器==默认调用直接实现在类型上的方法==
```rust
fn main() {
    let person = Human;
    person.fly();
}
```

```rust
fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
```

在方法名前指定 trait 名向 Rust 澄清了我们希望调用哪个 fly 实现。

也可以选择写成 Human::fly(&person)，这等同于 person.fly()


不是方法的关联函数没有 self 参数。当存在多个类型或者 trait 定义了相同函数名的非方法函数时，Rust 就不总是能计算出我们期望的是哪一个类型，除非使用 完全限定语法（fully qualified syntax）。

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {  // <<<< Dog 的
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {  // <<<< 也是 Dog 的
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", Dog::baby_name());
}
```

会输出 `A baby dog is called a Spot`


这并不是我们需要的。我们希望调用的是 Dog 上 Animal trait 实现那部分的 baby_name 函数，这样能够打印出 A baby dog is called a puppy。

示例 19-18 中用到的技术在这并不管用；如果将 main 改为示例 19-20 中的代码，则会得到一个编译错误：
```rust
fn main() {
    println!("A baby dog is called a {}", Animal::baby_name());
}
```
因为 Animal::baby_name 没有 self 参数，同时这可能会有其它类型实现了 Animal trait，Rust 无法计算出所需的是哪一个 Animal::baby_name 实现。我们会得到这个编译错误：
```shell
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
error[E0790]: cannot call associated function on trait without specifying the corresponding `impl` type
  --> src/main.rs:20:43
   |
2  |     fn baby_name() -> String;
   |     ------------------------- `Animal::baby_name` defined here
...
20 |     println!("A baby dog is called a {}", Animal::baby_name());
   |                                           ^^^^^^^^^^^^^^^^^ cannot call associated function of trait
   |
help: use the fully-qualified path to the only available implementation
   |
20 |     println!("A baby dog is called a {}", <::Dog as Animal>::baby_name());
   |                                           +++++++++       +

For more information about this error, try `rustc --explain E0790`.
error: could not compile `traits-example` due to previous error
```

为了消歧义并告诉 Rust 我们希望使用的是 Dog 的 Animal 实现而不是其它类型的 Animal 实现，需要使用 完全限定语法，这是调用函数时最为明确的方式。
```rust
fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```
使用完全限定语法来指定我们希望调用的是 Dog 上 Animal trait 实现中的 baby_name 函数

我们在尖括号中向 Rust 提供了类型注解，并通过在此函数调用中将 Dog 类型当作 Animal 对待，来指定希望调用的是 Dog 上 Animal trait 实现中的 baby_name 函数。


通常，完全限定语法定义为：
```rust
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

对于不是方法的关联函数，其没有一个 receiver，故只会有其他参数的列表。可以选择在任何函数或方法调用处使用完全限定语法。然而，允许省略任何 Rust 能够从程序中的其他信息中计算出的部分。只有当存在多个同名实现而 Rust 需要帮助以便知道我们希望调用哪个实现时，才需要使用这个较为冗长的语法。


### 父 trait 用于在另一个trait中使用某trait 的功能

有时我们可能会需要编写一个依赖另一个 trait 的 trait 定义：对于一个实现了第一个 trait 的类型，你希望要求这个类型也实现了第二个 trait。如此就可使 trait 定义使用第二个 trait 的关联项。这个所需的 trait 是我们实现的 trait 的 父（超）trait（supertrait）。


例如我们希望创建一个带有 outline_print 方法的 trait OutlinePrint，它会将给定的值格式化为带有星号框。也就是说，给定一个实现了标准库 Display trait 的并返回 (x, y) 的 Point，当调用以 1 作为 x 和 3 作为 y 的 Point 实例的 outline_print 会显示如下：

```text
**********
*        *
* (1, 3) *
*        *
**********
```

在 outline_print 的实现中，因为希望能够使用 Display trait 的功能，则需要说明 OutlinePrint 只能用于同时也实现了 Display 并提供了 OutlinePrint 需要的功能的类型。可以通过在 trait 定义中指定 OutlinePrint: Display 来做到这一点。这类似于为 trait 增加 trait bound。

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}

struct Point {
    x: i32,
    y: i32,
}

impl OutlinePrint for Point {}

fn main() {
    let p = Point { x: 1, y: 3 };
    p.outline_print();
}
```

```rust
use std::fmt;

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```


### newtype 模式用以在外部类型上实现外部trait

在第十章的 “为类型实现 trait” 部分，我们提到了孤儿规则（orphan rule），它说明只要 trait 或类型对于当前 crate 是本地的话就可以在此类型上实现该 trait。一个绕开这个限制的方法是使用 newtype 模式（newtype pattern），它涉及到在一个元组结构体（第五章 “用没有命名字段的元组结构体来创建不同的类型” 部分介绍了元组结构体）中创建一个新类型。这个元组结构体带有一个字段作为希望实现 trait 的类型的简单封装。接着这个封装类型对于 crate 是本地的，这样就可以在这个封装上实现 trait。Newtype 是一个源自 （U.C.0079，逃） Haskell 编程语言的概念。使用这个模式没有运行时性能惩罚，这个封装类型在编译时就被省略了。

如果想要在 `Vec<T>` 上实现 Display，而孤儿规则阻止我们直接这么做，因为 Display trait 和 `Vec<T>` 都定义于我们的 crate 之外。可以创建一个包含 `Vec<T>` 实例的 Wrapper 结构体，接着可以如列表 19-23 那样在 Wrapper 上实现 Display 并使用 `Vec<T>` 的值：

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

Display 的实现使用 self.0 来访问其内部的 `Vec<T>`，因为 Wrapper 是元组结构体而 `Vec<T>` 是结构体总位于索引 0 的项。接着就可以使用 Wrapper 中 Display 的功能了。

此方法的缺点是，因为 Wrapper 是一个新类型，它没有定义于其值之上的方法；==必须直接在 Wrapper 上实现 `Vec<T>` 的所有方法==，这样就可以代理到self.0 上 —— 这就允许我们完全像 `Vec<T>` 那样对待 Wrapper。如果希望新类型拥有其内部类型的每一个方法，为封装类型实现 Deref trait（第十五章 “通过 Deref trait 将智能指针当作常规引用处理” 部分讨论过）并返回其内部类型是一种解决方案。如果不希望封装类型拥有所有内部类型的方法 —— 比如为了限制封装类型的行为 —— 则必须只自行实现所需的方法。

。。必须实现所有？ 只需要实现 用得到的就可以了吧。 Wrapper 是一个新类型，没有继承Vec， 而且继承还更好，有默认的实现。

。。这种就是 组合啊。



## 高级类型

### 为了类型安全 和抽象 而使用 newtype 模式

newtype 模式也可以用于一些其他我们还未讨论的功能，包括静态的确保某值不被混淆，和用来表示一个值的单位。实际上示例 19-15 中已经有一个这样的例子：Millimeters 和 Meters 结构体都在 newtype 中封装了 u32 值。如果编写了一个有 Millimeters 类型参数的函数，不小心使用 Meters 或普通的 u32 值来调用该函数的程序是不能编译的。

newtype 模式也可以用于抽象掉一些类型的实现细节：例如，封装类型可以暴露出与直接使用其内部私有类型时所不同的公有 API。

newtype 也可以隐藏其内部的泛型类型。例如，可以提供一个封装了 `HashMap<i32, String>` 的 People 类型，用来储存人名以及相应的 ID。使用 People 的代码只需与提供的公有 API 交互即可，比如向 People 集合增加名字字符串的方法，这样这些代码就无需知道在内部我们将一个 i32 ID 赋予了这个名字了。newtype 模式是一种实现第十七章 “封装隐藏了实现细节” 部分所讨论的隐藏实现细节的封装的轻量级方法。


### 类型别名用来创建类型同义词

```rust
fn main() {
    type Kilometers = i32;

    let x: i32 = 5;
    let y: Kilometers = 5;

    println!("x + y = {}", x + y);
}
```

类型别名的主要用途是减少重复。
```rust
    let f: Box<dyn Fn() + Send + 'static> = Box::new(|| println!("hi"));

    fn takes_long_type(f: Box<dyn Fn() + Send + 'static>) {
        // --snip--
    }

    fn returns_long_type() -> Box<dyn Fn() + Send + 'static> {
        // --snip--
    }

//--------------

    type Thunk = Box<dyn Fn() + Send + 'static>;

    let f: Thunk = Box::new(|| println!("hi"));

    fn takes_long_type(f: Thunk) {
        // --snip--
    }

    fn returns_long_type() -> Thunk {
        // --snip--
    }
```

类型别名也经常与 `Result<T, E>` 结合使用来减少重复。
标准库中的 std::io::Error 结构体代表了所有可能的 I/O 错误。std::io 中大部分函数会返回 `Result<T, E>`，其中 E 是 std::io::Error

```rust
use std::fmt;
use std::io::Error;

pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize, Error>;
    fn flush(&mut self) -> Result<(), Error>;

    fn write_all(&mut self, buf: &[u8]) -> Result<(), Error>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<(), Error>;
}
//-----------------
type Result<T> = std::result::Result<T, std::io::Error>;

pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize>;
    fn flush(&mut self) -> Result<()>;

    fn write_all(&mut self, buf: &[u8]) -> Result<()>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<()>;
}
```
类型别名在两个方面有帮助：易于编写 并 在整个 std::io 中提供了一致的接口。因为这是一个别名，它只是另一个 `Result<T, E>`，这意味着可以在其上使用 `Result<T, E>` 的任何方法，以及像 ? 这样的特殊语法。


### 从不返回的 never type

Rust 有一个叫做 ! 的特殊类型。在类型理论术语中，它被称为 empty type，因为它没有值。我们更倾向于称之为 never type。这个名字描述了它的作用：在函数从不返回的时候充当返回值。

```rust
fn bar() -> ! {
    // --snip--
}
```
从不返回的函数被称为 发散函数（diverging functions）。不能创建 ! 类型的值，所以 bar 也不可能返回值。

下面的代码可行，match 的分支必须返回相同的类型，那么 continue 的返回值是什么？

continue 的值是 !。也就是说，当 Rust 要计算 guess 的类型时，它查看这两个分支。前者是 u32 值，而后者是 ! 值。因为 ! 并没有一个值，Rust 决定 guess 的类型是 u32。

```rust
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
```

never type 的另一个用途是 panic!。还记得 `Option<T>` 上的 unwrap 函数吗？它产生一个值或 panic。这里是它的定义：

```rust
impl<T> Option<T> {
    pub fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!("called `Option::unwrap()` on a `None` value"),
        }
    }
}
```

Rust 知道 val 是 T 类型，panic! 是 ! 类型，所以整个 match 表达式的结果是 T 类型。这能工作是因为 panic! 并不产生一个值；它会终止程序。对于 None 的情况，unwrap 并不返回一个值，所以这些代码是有效的。



```rust
fn main() {
    print!("forever ");

    loop {
        print!("and ever ");
    }
}
```
循环永远也不结束，所以此表达式的值是 !。但是如果引入 break 这就不为真了，因为循环在执行到 break 后就会终止。


### 动态大小类型 和 Sized trait

Rust 需要知道有关类型的某些细节，例如为特定类型的值需要分配多少空间。这便是起初留下的一个类型系统中令人迷惑的角落：即 动态大小类型（dynamically sized types）。这有时被称为 “DST” 或 “unsized types”，这些类型允许我们处理只有在运行时才知道大小的类型。


让我们深入研究一个贯穿本书都在使用的动态大小类型的细节：str。没错，不是 &str，而是 str 本身。str 是一个 DST；直到运行时我们都不知道字符串有多长。因为直到运行时都不能知道其大小，也就意味着不能创建 str 类型的变量，也不能获取 str 类型的参数。考虑一下这些代码，他们不能工作：
```rust
fn main() {
    let s1: str = "Hello there!";
    let s2: str = "How's it going?";
}
```
你已经知道了这种问题的答案：s1 和 s2 的类型是 &str 而不是 str。

slice 数据结构仅仅储存了开始位置和 slice 的长度。所以虽然 &T 是一个储存了 T 所在的内存位置的单个值，&str 则是 两个 值：str 的地址和其长度。这样，&str 就有了一个在编译时可以知道的大小：它是 usize 长度的两倍。也就是说，我们总是知道 &str 的大小，而无论其引用的字符串是多长。这里是 Rust 中动态大小类型的常规用法：他们有一些额外的元信息来储存动态信息的大小。这引出了动态大小类型的黄金规则：必须将动态大小类型的值置于某种指针之后。

可以将 str 与所有类型的指针结合：比如 `Box<str>` 或 `Rc<str>`

事实上，之前我们已经见过了，不过是另一个动态大小类型：trait。每一个 trait 都是一个可以通过 trait 名称来引用的动态大小类型。在第十七章 “为使用不同类型的值而设计的 trait 对象” 部分，我们提到了为了将 trait 用于 trait 对象，必须将他们放入指针之后，比如 &dyn Trait 或 `Box<dyn Trait>`（`Rc<dyn Trait>` 也可以）。

为了处理 DST，Rust 提供了 Sized trait 来决定一个类型的大小是否在编译时可知。这个 trait 自动为编译器在编译时就知道大小的类型实现。另外，Rust 隐式的**为每一个泛型函数增加了 Sized bound**。也就是说，对于如下泛型函数定义：

```rust
fn generic<T>(t: T) {
    // --snip--
}
```
实际上被当作如下处理：

```rust
fn generic<T: Sized>(t: T) {
    // --snip--
}
```
泛型函数默认只能用于在编译时已知大小的类型。然而可以使用如下特殊语法来放宽这个限制：
```rust
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
```

?Sized 上的 trait bound 意味着 “T 可能是也可能不是 Sized” 同时这个注解会覆盖泛型类型必须在编译时拥有固定大小的默认规则。这种意义的 ?Trait 语法只能用于 Sized ，而不能用于任何其他 trait。

我们将 t 参数的类型从 T 变为了 &T：因为其类型可能不是 Sized 的，所以需要将其置于某种指针之后。在这个例子中选择了引用。


## 高级函数和闭包

### 函数指针

我们讨论过了如何向函数传递闭包；也可以向函数传递常规函数！

这个技术在我们希望传递已经定义的函数而不是重新定义闭包作为参数时很有用。函数满足类型 fn（小写的 f），不要与闭包 trait 的 Fn 相混淆。

fn 被称为 函数指针（function pointer）。通过函数指针允许我们使用函数作为另一个函数的参数。

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!("The answer is: {}", answer);
}
```
这会打印出 The answer is: 12

fn 是一个类型而不是一个 trait，所以直接指定 fn 作为参数而不是声明一个带有 Fn 作为 trait bound 的泛型参数。


函数指针实现了所有三个闭包 trait（Fn、FnMut 和 FnOnce），所以总是可以在调用期望闭包的函数时传递函数指针作为参数。倾向于编写使用泛型和闭包 trait 的函数，这样它就能接受函数或闭包作为参数。

一个只期望接受 fn 而不接受闭包的情况的例子是与不存在闭包的外部代码交互时：C 语言的函数可以接受函数作为参数，但 C 语言没有闭包。


一个既可以使用内联定义的闭包又可以使用命名函数的例子:使用 map 函数将一个数字 vector 转换为一个字符串 vector，就可以使用闭包
```rust
fn main() {
    let list_of_numbers = vec![1, 2, 3];
    let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(|i| i.to_string()).collect();
}
```

可以将函数作为 map 的参数来代替闭包

```rust
fn main() {
    let list_of_numbers = vec![1, 2, 3];
    let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(ToString::to_string).collect();
}
```

。。jdk8 stream, lambda, function

注意这里必须使用 “高级 trait” 部分讲到的完全限定语法，因为存在多个叫做 to_string 的函数；这里使用了定义于 ToString trait 的 to_string 函数，标准库为所有实现了 Display 的类型实现了这个 trait。

第六章 “枚举值” 部分中定义的每一个枚举成员也变成了一个构造函数。我们可以使用这些构造函数作为实现了闭包 trait 的函数指针，这意味着可以指定构造函数作为接受闭包的方法的参数

```rust
fn main() {
    enum Status {
        Value(u32),
        Stop,
    }

    let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect();
}
```

创建了 Status::Value 实例，它通过 map 用范围的每一个 u32 值调用 Status::Value 的初始化函数


### 返回闭包

闭包表现为 trait，这意味着不能直接返回闭包。对于大部分需要返回 trait 的情况，可以使用实现了期望返回的 trait 的具体类型来替代函数的返回值。但是这不能用于闭包，因为他们没有一个可返回的具体类型；例如不允许使用函数指针 fn 作为返回值类型。

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```


## 宏

宏（Macro）指的是 Rust 中一系列的功能：使用 macro_rules! 的 声明（Declarative）宏，和三种 过程（Procedural）宏：
- 自定义 #[derive] 宏在结构体和枚举上指定通过 derive 属性添加的代码
- 类属性（Attribute-like）宏定义可用于任意项的自定义属性
- 类函数宏看起来像函数不过作用于作为参数传递的 token


### 宏和函数的区别

从根本上来说，宏是一种为写其他代码而写代码的方式，即所谓的 元编程（metaprogramming）。

宏以 展开 的方式来生成比你所手写出的更多的代码。

元编程对于减少大量编写和维护的代码是非常有用的，它也扮演了函数扮演的角色。但宏有一些函数所没有的附加能力。

一个函数签名必须声明函数参数个数和类型。相比之下，**宏能够接收不同数量的参数**：用一个参数调用 println!("hello") 或用两个参数调用 println!("hello {}", name) 。而且，宏可以在编译器翻译代码前展开，例如，**宏可以在一个给定类型上实现 trait**。而函数则不行，因为函数是在运行时被调用，同时 trait 需要在编译时实现。

实现宏不如实现函数的一面是宏定义要比函数定义更复杂，因为你正在编写生成 Rust 代码的 Rust 代码。由于这样的间接性，宏定义通常要比函数定义更难阅读、理解以及维护。

宏和函数的最后一个重要的区别是：在一个文件里调用宏 之前 必须定义它，或将其引入作用域，而函数则可以在任何地方定义和调用。


### 使用 macro_rules! 的声明宏 用于 通用元编程

Rust 最常用的宏形式是 声明宏（declarative macros）。它们有时也被称为 “macros by example”、“macro_rules! 宏” 或者就是 “macros”。

其核心概念是，声明宏允许我们编写一些类似 Rust match 表达式的代码。

match 表达式是控制结构，其接收一个表达式，与表达式的结果进行模式匹配，然后根据模式匹配执行相关代码。

宏也将一个值和包含相关代码的模式进行比较；此种情况下，该值是传递给宏的 Rust 源代码字面值，模式用于和前面提到的源代码字面值进行比较，每个模式的相关代码会替换传递给宏的代码。所有这一切都发生于编译时。

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

下面是一个 简化后的 vec! 的定义
```rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```
。。。。。。

#[macro_export] 注解表明只要导入了定义这个宏的 crate，该宏就应该是可用的。如果没有该注解，这个宏不能被引入作用域。

接着使用 macro_rules! 和宏名称开始宏定义，且所定义的宏并 不带 感叹号。名字后跟大括号表示宏定义体，在该例中宏名称是 vec 。

vec! 宏的结构和 match 表达式的结构类似。此处有一个分支模式 ( $( $x:expr ),* ) ，后跟 => 以及和模式相关的代码块。如果模式匹配，该相关代码块将被执行。这里这个宏只有一个模式，那就只有一个有效匹配方向，其他任何模式方向（译者注：不匹配这个模式）都会导致错误。更复杂的宏会有多个分支模式。

宏定义中有效模式语法和在第十八章提及的模式语法是不同的，因为宏模式所匹配的是 Rust 代码结构而不是值。

首先，一对括号包含了整个模式。我们使用美元符号（$）在宏系统中声明一个变量来包含匹配该模式的 Rust 代码。美元符号明确表明这是一个宏变量而不是普通 Rust 变量。之后是一对括号，其捕获了符合括号内模式的值用以在替代代码中使用。$() 内则是 $x:expr ，其匹配 Rust 的任意表达式，并将该表达式命名为 $x。

`$()` 之后的逗号说明一个可有可无的逗号分隔符可以出现在 `$()` 所匹配的代码之后。紧随逗号之后的 * 说明该模式匹配零个或更多个 * 之前的任何模式。

当以 vec![1, 2, 3]; 调用宏时，$x 模式与三个表达式 1、2 和 3 进行了三次匹配。

现在让我们来看看与此分支模式相关联的代码块中的模式：匹配到模式中的`$()`的每一部分，都会在（=>右侧）`$()*` 里生成 `temp_vec.push($x)`，生成零次还是多次取决于模式匹配到多少次。`$x` 由每个与之相匹配的表达式所替换。当以 vec![1, 2, 3]; 调用该宏时，替换该宏调用所生成的代码会是下面这样：

```rust
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

我们已经定义了一个宏，其可以接收任意数量和类型的参数，同时可以生成能够创建包含指定元素的 vector 的代码。

。。好像比 C的 宏高级一点点。。忘记C的宏 能不能不定长参数了。。。
。。不过 C 可以直接 代码替换 `#define AAA vector<int>vi; for(int i=1;i<10;++i) {vi.push_back(i);}`  吧？
。。C++可以模板，模板可以不定长参数，而且可以每次 从参数列表中获得尾巴。 获得尾巴还是头？ 应该是头， 因为 变长参数 必须是 最后一个。


### 用于从属性生成代码的过程宏

第二种形式的宏被称为 过程宏（procedural macros），因为它们更像函数（一种过程类型）。过程宏接收 Rust 代码作为输入，在这些代码上进行操作，然后产生另一些代码作为输出，而非像声明式宏那样匹配对应模式然后以另一部分代码替换当前代码。有三种类型的过程宏（自定义派生（derive），类属性和类函数），不过它们的工作方式都类似。

创建过程宏时，其定义必须驻留在它们自己的具有特殊 crate 类型的 crate 中。这么做出于复杂的技术原因，将来我们希望能够消除这些限制。

some_attribute 是一个使用特定宏变体的占位符。

```rust
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

定义过程宏的函数接收一个 TokenStream 作为输入并生成 TokenStream 作为输出。TokenStream 是定义于proc_macro crate 里代表一系列 token 的类型，Rust 默认携带了proc_macro crate。这就是宏的核心：宏所处理的源代码组成了输入 TokenStream，宏生成的代码是输出 TokenStream。函数上还有一个属性；这个属性指明了我们创建的过程宏的类型。在同一 crate 中可以有多种的过程宏。


### 如何编写自定义 derive 宏

https://kaisery.github.io/trpl-zh-cn/ch19-06-macros.html

让我们创建一个 hello_macro crate，其包含名为 HelloMacro 的 trait 和关联函数 hello_macro。不同于让用户为其每一个类型实现 HelloMacro trait，我们将会提供一个过程式宏以便用户可以使用 `#[derive(HelloMacro)]` 注解他们的类型来得到 hello_macro 函数的默认实现。该默认实现会打印 Hello, Macro! My name is TypeName!，其中 TypeName 为定义了 trait 的类型名。

能够使得如下的代码可以执行：
```rust
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro();
}
```


第一步是像下面这样新建一个库 crate：

```rust
$ cargo new hello_macro --lib
```

src/lib.rs 中定义 HelloMacro trait 以及其关联函数：
```rust
pub trait HelloMacro {
    fn hello_macro();
}
```

现在有了一个包含函数的 trait。此时，crate 用户可以实现该 trait 以达到其期望的功能，像这样：
```rust
use hello_macro::HelloMacro;

struct Pancakes;

impl HelloMacro for Pancakes {
    fn hello_macro() {
        println!("Hello, Macro! My name is Pancakes!");
    }
}

fn main() {
    Pancakes::hello_macro();
}
```

然而，他们需要为每一个他们想使用 hello_macro 的类型编写实现的代码块。我们希望为其节约这些工作。

我们也无法为 hello_macro 函数提供一个能够打印实现了该 trait 的类型的名字的默认实现：Rust 没有反射的能力，因此其无法在运行时获取类型名。我们需要一个在编译时生成代码的宏。

下一步是定义过程式宏。在编写本部分时，过程式宏必须在其自己的 crate 内。该限制最终可能被取消。构造 crate 和其中宏的惯例如下：对于一个 foo 的包来说，一个自定义的派生过程宏的包被称为 foo_derive 。在 hello_macro 项目中新建名为 hello_macro_derive 的包。

```rust
$ cargo new hello_macro_derive --lib
```

hello_macro_derive/Cargo.toml
```toml
[lib]
proc-macro = true

[dependencies]
syn = "1.0"
quote = "1.0"
```

hello_macro_derive/src/lib.rs
```rust
use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // Construct a representation of Rust code as a syntax tree
    // that we can manipulate
    let ast = syn::parse(input).unwrap();

    // Build the trait implementation
    impl_hello_macro(&ast)
}
```

将代码分成了hello_macro_derive 和 impl_macro_derive 两个函数，前者负责解析 TokenStream，后者负责转换语法树：这使得编写过程宏更方便。

syn crate 将字符串中的 Rust 代码解析成为一个可以操作的数据结构。quote 则将 syn 解析的数据结构转换回 Rust 代码。这些 crate 让解析任何我们所要处理的 Rust 代码变得更简单：为 Rust 编写整个的解析器并不是一件简单的工作。

该函数首先将来自 TokenStream 的 input 转换为一个我们可以解释和操作的数据结构。这正是 syn 派上用场的地方。syn 中的 parse 函数获取一个 TokenStream 并返回一个表示解析出 Rust 代码的 DeriveInput 结构体。

```rust
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

hello_macro_derive/src/lib.rs
```rust
use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // Construct a representation of Rust code as a syntax tree
    // that we can manipulate
    let ast = syn::parse(input).unwrap();

    // Build the trait implementation
    impl_hello_macro(&ast)
}

fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}!", stringify!(#name));
            }
        }
    };
    gen.into()
}
```


### 类属性宏

类属性宏与自定义派生宏相似，不同的是 derive 属性生成代码，它们（类属性宏）能让你创建新的属性。它们也更为灵活；derive 只能用于结构体和枚举；属性还可以用于其它的项，比如函数。作为一个使用类属性宏的例子，可以创建一个名为 route 的属性用于注解 web 应用程序框架（web application framework）的函数：

```rust
#[route(GET, "/")]
fn index() {
```

`#[route]` 属性将由框架本身定义为一个过程宏。其宏定义的函数签名看起来像这样：

```rust
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

这里有两个 TokenStream 类型的参数；第一个用于属性内容本身，也就是 GET, "/" 部分。第二个是属性所标记的项：在本例中，是 fn index() {} 和剩下的函数体。

除此之外，类属性宏与自定义派生宏工作方式一致：创建 proc-macro crate 类型的 crate 并实现希望生成代码的函数！


### 类(似)函数宏

类函数（Function-like）宏的定义看起来像函数调用的宏。类似于 macro_rules!，它们比函数更灵活；例如，可以接受未知数量的参数。然而 macro_rules! 宏只能使用之前 “使用 macro_rules! 的声明宏用于通用元编程” 介绍的类匹配的语法定义。类函数宏获取 TokenStream 参数，其定义使用 Rust 代码操纵 TokenStream，就像另两种过程宏一样。一个类函数宏例子是可以像这样被调用的 sql! 宏：

```rust
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

这个宏会解析其中的 SQL 语句并检查其是否是句法正确的，这是比 macro_rules! 可以做到的更为复杂的处理。sql! 宏应该被定义为如此：

```rust
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

这类似于自定义派生宏的签名：获取括号中的 token，并返回希望生成的代码。



# 最后的项目：构建多线程 web server

https://kaisery.github.io/trpl-zh-cn/ch20-00-final-project-a-web-server.html






















# 附录

## 关键字

### 使用中的关键字

```text
as - 强制类型转换，消除特定包含项的 trait 的歧义，或者对 use 语句中的项重命名
async - 返回一个 Future 而不是阻塞当前线程
await - 暂停执行直到 Future 的结果就绪
break - 立刻退出循环
const - 定义常量或不变裸指针（constant raw pointer）
continue - 继续进入下一次循环迭代
crate - 在模块路径中，代指 crate root
dyn - 动态分发 trait 对象
else - 作为 if 和 if let 控制流结构的 fallback
enum - 定义一个枚举
extern - 链接一个外部函数或变量
false - 布尔字面值 false
fn - 定义一个函数或 函数指针类型 (function pointer type)
for - 遍历一个迭代器或实现一个 trait 或者指定一个更高级的生命周期
if - 基于条件表达式的结果分支
impl - 实现自有或 trait 功能
in - for 循环语法的一部分
let - 绑定一个变量
loop - 无条件循环
match - 模式匹配
mod - 定义一个模块
move - 使闭包获取其所捕获项的所有权
mut - 表示引用、裸指针或模式绑定的可变性
pub - 表示结构体字段、impl 块或模块的公有可见性
ref - 通过引用绑定
return - 从函数中返回
Self - 定义或实现 trait 的类型的类型别名
self - 表示方法本身或当前模块
static - 表示全局变量或在整个程序执行期间保持其生命周期
struct - 定义一个结构体
super - 表示当前模块的父模块
trait - 定义一个 trait
true - 布尔字面值 true
type - 定义一个类型别名或关联类型
union - 定义一个 union 并且是 union 声明中唯一用到的关键字
unsafe - 表示不安全的代码、函数、trait 或实现
use - 引入外部空间的符号
where - 表示一个约束类型的从句
while - 基于一个表达式的结果判断是否进行循环
```

### 保留将来使用的关键字
```text
    abstract
    become
    box
    do
    final
    macro
    override
    priv
    try
    typeof
    unsized
    virtual
    yield
```

### 原始标识符
原始标识符（Raw identifiers）允许你使用通常不能使用的关键字，其带有 r# 前缀。

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```


## 运算符与符号

### 运算符

|运算符	|示例	|解释	|是否可重载|
|--|--|--|--|
!	|ident!(...), ident!{...}, ident![...]	|宏展开	|
!	|!expr	|按位非或逻辑非	|Not	|
!=	|expr != expr	|不等比较	|PartialEq	|
%	|expr % expr	|算术取余	|Rem	|
%=	|var %= expr	|算术取余与赋值	|RemAssign	|
&	|&expr, &mut expr	|借用	|
&	|&type, &mut type, &'a type, &'a mut type	|借用指针类型	|
|&	|expr & expr	|按位与	|BitAnd	|
|&=	|var &= expr	|按位与及赋值	|BitAndAssign	|
|&&	|expr && expr	|短路（Short-circuiting）逻辑与	|
|*	|expr * expr	|算术乘法	|Mul	|
|*=	|var *= expr	|算术乘法与赋值	|MulAssign	|
|*	|*expr	|解引用	|Deref	|
|*	|*const type, *mut type	|裸指针	|
|+	|trait + trait, 'a + trait	|复合类型限制	|
|+	|expr + expr	|算术加法	|Add	|
|+=	|var += expr	|算术加法与赋值	|AddAssign	|
|,	|expr, expr	|参数以及元素分隔符	|
|-	|- expr	|算术取负	|Neg	|
|-	|expr - expr	|算术减法	|Sub	|
|-=	|var -= expr	|算术减法与赋值	|SubAssign	|
|->	|fn(...) -> type, \|...\| -> type	|函数与闭包，返回类型	|
|.	|expr.ident	|成员访问	|
|..	|.., expr.., ..expr, expr..expr	|右开区间范围	|PartialOrd	|
|..=	|..=expr, expr..=expr	|右闭区间范围模式	|PartialOrd	|
|..	|..expr	|结构体更新语法	|
|..	|variant(x, ..), struct_type { x, .. }	|“与剩余部分” 的模式绑定	|
|...	|expr...expr	|（Deprecated，请使用 ..=）在模式中：闭区间范围模式	|
|/	|expr / expr	|算术除法	|Div	|
|/=	|var /= expr	|算术除法与赋值	|DivAssign	|
|:	|pat: type, ident: type	|约束	|
|:	|ident: expr	|结构体字段初始化	|
|:	|'a: loop {...}	|循环标志	|
|;	|expr;	|语句和语句结束符	|
|;	|[...; len]	|固定大小数组语法的部分	|
|<<	|expr << expr	|左移	|Shl	|
|<<=	|var <<= expr	|左移与赋值	|ShlAssign	|
|<	|expr < expr	|小于比较	|PartialOrd	|
|<=	|expr <= expr	|小于等于比较	|PartialOrd	|
|=	|var = expr, ident = type	|赋值/等值	|
|==	|expr == expr	|等于比较	|PartialEq	|
|=>	|pat => expr	|匹配准备语法的部分	|
|>	|expr > expr	|大于比较	|PartialOrd	|
|>=	|expr >= expr	|大于等于比较	|PartialOrd	|
|>>	|expr >> expr	|右移	|Shr	|
|>>=	|var >>= expr	|右移与赋值	|ShrAssign	|
|@	|ident @ pat	|模式绑定	|
|^	|expr ^ expr	|按位异或	|BitXor	|
|^=	|var ^= expr	|按位异或与赋值	|BitXorAssign	|
|\|	|pat \| pat	|模式选择	|
|\|	|expr \| expr	|按位或	|BitOr	|
|\|=	|var \|= expr	|按位或与赋值	|BitOrAssign	|
|\|\|	|expr \|\| expr	|短路（Short-circuiting）逻辑或	|
|?	|expr?	|错误传播	|



### 非运算符符号

#### 独立语法

展示了以其自身出现以及出现在合法其他各个地方的符号。

|符号	|解释|
|--|--|
|'ident	|命名生命周期或循环标签|
|...u8, ...i32, ...f64, ...usize 等	|指定类型的数值常量|
|"..."	|字符串常量|
|r"...", r#"..."#, r##"..."##, etc.	|原始字符串字面值，未处理的转义字符|
|b"..."	|字节字符串字面值; 构造一个字节数组类型而非字符串|
|br"...", br#"..."#, br##"..."## 等	|原始字节字符串字面值，原始和字节字符串字面值的结合|
|'...'	|字符字面值|
|b'...'	|ASCII 码字节字面值|
|\|...\| expr	|闭包|
|!	|离散函数的总是为空的类型|
|_	|“忽略” 模式绑定；也用于增强整型字面值的可读性|



#### 路径相关语法

出现在从模块结构到项的路径上下文中的符号

|符号	|解释|
|--|--|
|ident::ident	|命名空间路径|
|::path	|与 crate 根相对的路径（如一个显式绝对路径）|
|self::path	|与当前模块相对的路径（如一个显式相对路径）|
|super::path	|与父模块相对的路径|
|type::ident, `<type as trait>::ident`	|关联常量、函数以及类型|
|`<type>::...`	|不可以被直接命名的关联项类型（如 `<&T>::...，<[T]>::...`，等）|
|trait::method(...)	|通过命名定义的 trait 来消除方法调用的二义性|
|type::method(...)	|通过命名定义的类型来消除方法调用的二义性|
|`<type as trait>::method(...)`	|通过命名 trait 和类型来消除方法调用的二义性|


#### 泛型

出现在泛型类型参数上下文中的符号。
|符号	|解释|
|--|--|
|path<...>	|为一个类型中的泛型指定具体参数（如 `Vec<u8>`）|
|path::<...>, method::<...>	|为一个泛型、函数或表达式中的方法指定具体参数，通常指 turbofish（如 `"42".parse::<i32>()`）|
|fn ident<...> ...	|泛型函数定义|
|struct ident<...> ...	|泛型结构体定义|
|enum ident<...> ...	|泛型枚举定义|
|impl<...> ...	|定义泛型实现|
|for<...> type	|高级生命周期限制|
|`type<ident=type>`	|泛型，其一个或多个相关类型必须被指定为特定类型（如 `Iterator<Item=T>`）|


#### trait bound 约束

出现在使用 trait bounds 约束泛型参数上下文中的符号。

|符号	|解释|
|--|--|
|T: U	|泛型参数 T 约束于实现了 U 的类型|
|T: 'a	|泛型 T 的生命周期必须长于 'a（意味着该类型不能传递包含生命周期短于 'a 的任何引用）|
|T: 'static	|泛型 T 不包含除 'static 之外的借用引用|
|'b: 'a	|泛型 'b 生命周期必须长于泛型 'a|
|T: ?Sized	|使用一个不定大小的泛型类型|
|'a + trait, trait + trait	|复合类型限制|


#### 宏与属性

在调用或定义宏以及在其上指定属性时的上下文中出现的符号。

|符号	|解释|
|--|--|
|#[meta]	|外部属性|
|#![meta]	|内部属性|
|$ident	|宏替换|
|$ident:kind	|宏捕获|
|$(…)…	|宏重复|
|ident!(...), ident!{...}, ident![...]	|宏调用|


#### 注释

```text
//	行注释
//!	内部行文档注释
///	外部行文档注释
/*...*/	块注释
/*!...*/	内部块文档注释
/**...*/	外部块文档注释
```

#### 元组

出现在使用元组时上下文中的符号。

|符号	|解释|
|--|--|
|()	|空元组（亦称单元），即是字面值也是类型|
|(expr)	|括号表达式|
|(expr,)	|单一元素元组表达式|
|(type,)	|单一元素元组类型|
|(expr, ...)	|元组表达式|
|(type, ...)	|元组类型|
|expr(expr, ...)	|函数调用表达式；也用于初始化元组结构体 struct 以及元组枚举 enum 变体|
|expr.0, expr.1, etc.	|元组索引|


#### 大括号

```text
{...}	块表达式
Type {...}	struct 字面值
```

#### 方括号

|符号	|解释|
|--|--|
|[...]	|数组|
|[expr; len]	|复制了 len个 expr的数组|
|[type; len]	|包含 len个 type 类型的数组|
|expr[expr]	|集合索引。重载（Index, IndexMut）|
|expr[..], expr[a..], expr[..b], expr[a..b]	|集合索引，使用 Range，RangeFrom，RangeTo 或 RangeFull 作为索引来代替集合 slice|



## 可派生的trait

https://kaisery.github.io/trpl-zh-cn/appendix-03-derivable-traits.html

derive 属性会在使用 derive 语法标记的类型上生成对应 trait 的默认实现的代码。

更多信息查询： https://doc.rust-lang.org/std/index.html


### 用于程序员输出的 Debug

Debug trait 用于开启格式化字符串中的调试格式，其通过在 {} 占位符中增加 :? 表明。

在使用 assert_eq! 宏时，Debug trait 是必须的。


### 等值比较的 PartialEq 和 Eq

PartialEq trait 可以比较一个类型的实例以检查是否相等，并开启了 == 和 != 运算符的功能。

当使用 assert_eq! 宏时，需要比较一个类型的两个实例是否相等，则 PartialEq trait 是必须的。

Eq trait 没有方法。其作用是表明每一个被标记类型的值等于其自身。Eq trait 只能应用于那些实现了 PartialEq 的类型，但并非所有实现了 PartialEq 的类型都可以实现 Eq。浮点类型就是一个例子：浮点数的实现表明两个非数字（NaN，not-a-number）值是互不相等的。

### 次序比较的 PartialOrd 和 Ord

PartialOrd trait 可以基于排序的目的而比较一个类型的实例。实现了 PartialOrd 的类型可以使用 <、 >、<= 和 >= 操作符。但只能在同时实现了 PartialEq 的类型上使用 PartialOrd。


### 复制值的 Clone 和 Copy

Clone trait 可以明确地创建一个值的深拷贝（deep copy），复制过程可能包含任意代码的执行以及堆上数据的复制

Copy trait 允许你通过==只拷贝存储在栈上的位==来复制值而不需要额外的代码。


### 固定大小的值到值映射的 Hash

Hash trait 可以实例化一个任意大小的类型，并且能够用哈希（hash）函数将该实例映射到一个固定大小的值上。派生 Hash 实现了 hash 方法。

hash 方法的派生实现结合了在类型的每部分调用 hash 的结果，这意味着所有的字段或值也必须实现了 Hash，这样才能够派生 Hash。

例如，在 HashMap<K, V> 上存储数据，存放 key 的时候，Hash 是必须的。


### 默认值的 Default

Default trait 使你创建一个类型的默认值。派生 Default 实现了 default 函数。default 函数的派生实现调用了类型每部分的 default 函数，这意味着类型中所有的字段或值也必须实现了 Default，这样才能够派生 Default 。

例如，当你在 Option<T> 实例上使用 unwrap_or_default 方法时，Default trait 是必须的。如果 Option<T> 是 None的话，unwrap_or_default 方法将返回存储在 Option<T> 中 T 类型的 Default::default 的结果。



## 实用开发工具

### 通过 rustfmt 自动格式化

安装 rustfmt
`$ rustup component add rustfmt`

格式化整个 Cargo 项目
`$ cargo fmt`


### 通过 rustfix 修复代码

如果你编写过 Rust 代码，那么你可能见过那些有很明显修复方式的编译器警告

可以通过 cargo fix 命令使用 rustfix 工具来自动采用该建议：
`$ cargo fix`

cargo fix 命令可以用于在不同 Rust 版本间迁移代码。

### 通过 clippy 提供更多 lint 功能

https://github.com/rust-lang/rust-clippy

安装 clippy：
`$ rustup component add clippy`

对任何 Cargo 项目运行 clippy 的 lint：
`$ cargo clippy`


如果程序使用了如 pi 这样数学常数的近似值
```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

在此项目上运行 cargo clippy 会导致这个错误：

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

### 使用 rust-analyzer 的 IDE 集成

为了帮助 IDE 集成，Rust 社区建议使用 rust-analyzer。这个工具是一组以编译器为中心的实用程序，它实现了 Language Server Protocol（一个 IDE 与编程语言之间的通信规范）。rust-analyzer 可以用于不同的客户端，比如 Visual Studio Code 的 Rust analyzer 插件。

访问 rust-analyzer 项目的 主页 来了解如何安装安装它，然后为你的 IDE 安装 language server 支持。如此你的 IDE 便会获得如自动补全、跳转到定义和 inline error 之类的功能。

> For VS Code, install rust-analyzer extension from the marketplace.


2023-05-17 11:04

---
---


2023-10-17 13:51

# course
https://github.com/sunface/rust-course


## cargo

### command
Rust 项目分为2种， bin 和 lib， 前者是可运行，后者是依赖库
早期，下面的命令必须添加 --bin 后缀。 现在默认 bin 类型。
`cargo new hi_world_project`


在工程的顶级目录中，即 /../../hi_world_project 中。
`cargo run`

cargo run 是编译 + 运行。所以等于：
`cargo build`
`./target/debug/hi_world_project`

默认是debug模式，编译时，不进行优化。
`cargo run --release`
`cargo build --release`
`./target/release/hi_world_project`

项目大了以后，cargo run 和 build 会变慢，有没有更快的方法验证代码的正确性？
下面是快速检查 代码是否能够编译通过
`cargo check`

rust的编译速度 比 go 慢。

### Cargo.toml  Cargo.lock
Cargo.toml 和 Cargo.lock 是 cargo 的==核心==文件，它的所有活动均基于此二者。
- Cargo.toml 
  是 cargo 特有的项目数据描述文件。它存储了项目的==所有元配置信息==，如果 Rust 开发者希望 Rust 项目能够按照期望的方式进行构建、测试和运行，那么，必须按照合理的方式构建 Cargo.toml。

- Cargo.lock 
  文件是 cargo 工具根据同一项目的 toml 文件生成的项目依赖详细清单，因此我们==一般不用==修改它，只需要对着 Cargo.toml 文件撸就行了。

如果是 bin(可运行) 项目，那么上传 Cargo.lock 到git， 如果是 lib 项目，添加到 .gitignore 中， 不要上传


Cargo.toml 例子
```text
[package]
name = "world_hello"
version = "0.1.0"
edition = "2021"
```
。。1.69.0 的 edition 还是 2021 。。。

name是项目名称， 
version当前版本，新项目默认 0.1.0
edition 是rust大版本。

#### 定义依赖

在 Cargo.toml 中，主要通过各种依赖段落来描述该项目的各种依赖项：
- 基于 Rust 官方仓库 crates.io，通过版本说明来描述
- 基于项目源代码的 git 仓库地址，通过 URL 来描述
- 基于本地项目的绝对路径或者相对路径，通过类 Unix 模式的路径来描述

3种具体写法：
```text
[dependencies]
rand = "0.3"
hammer = { version = "0.5.0"}
color = { git = "https://github.com/bjz/color-rs" }
geometry = { path = "crates/geometry" }
```


#### 依赖的镜像

>> 增加镜像

修改/新增 下列文件
`$HOME/.cargo/config.toml`

添加
```text
[registries]
ustc = { index = "https://mirrors.ustc.edu.cn/crates.io-index/" }
```

这里是增加了一个镜像，所以在 引入依赖时，需要指定镜像
在`Cargo.toml` 中：
```text
[dependencies]
time = {  registry = "ustc" }
```

重新配置后，初次构建可能要较长时间，因为要下载 ustc 注册服务的 index文件。

注意：
1. cargo 1.68开始，支持稀疏索引，不再需要下载完成后的 crates.io-index 仓库。
```text
[source.ustc]
registry = "sparse+https://mirrors.ustc.edu.cn/crates.io-index/"
```
。。这个应该是 home的 config.toml 中的吧。

2. cargo search 无法使用镜像


> 字节跳动的镜像

。。根据覆盖镜像 的描述，下面是覆盖镜像不是新增镜像。
```text
[source.crates-io]
replace-with = 'rsproxy'

[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"

# 稀疏索引，要求 cargo >= 1.68
[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index/"

[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"

[net]
git-fetch-with-cli = true
```

##### 覆盖镜像
更推荐这种。
不需要修改 Cargo.toml

在 `$HOME/.cargo/config.toml` 添加以下内容：
```text
[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
```



## ch02 基本概念

- 所有权、借用、生命周期
- 宏编程
- 模式匹配

字符串 ""
单个字符 ''

### 变量绑定与解构

rust 即支持声明可变变量，又支持声明不可变变量。(函数式语言只支持 不可变变量)

变量的 不可变可以在 运行期 避免一些多余的 runtime check。

rust中称为 变量绑定， 而不是 赋值
`let a = "hi";`

绑定更加精确，涉及 Rust的最核心的原则：所有权：
==任何内存对象都是有主人的，而且一般情况下完全属于它的主人，绑定就是把这个对象绑定给一个变量，让这个变量成为它的主人==



变量没有被使用，编译器会 warning
下划线开头忽略未使用的变量， 编译器不会 warning

#### 变量解构
let 表达式不仅仅用于变量的绑定，还能进行复杂变量的解构：从一个相对复杂的变量中，匹配出该变量的一部分内容：

`let (a, mut b): (bool, bool) = (true, false);`

。。mut 是变量的，  不是 类型的。。
。。不
fn clear(text: &mut String) -> () {
    *text = String::from("");
}
fn clear22(mut text: String) -> () {
    text = String::from("");
}
。。上面这2个都可以。 所以 mut 可以放2个地方。

rust 1.59后，可以在 赋值语句的 左式 中使用 元组，切片，结构体模式。

```rust
struct MyStruct {
    e: i32
}
fn main() {
    let (a,b,c,d,e);
    (a,b) = (1,2);
    [c, .., d, _] = [1,2,3,4,5];
    MyStruct { e, .. } = MyStruct { e: 5 };

    assert_eq!([1,2,1,4,5], [a,b,c,d,e])
}
```

+= 的赋值语句还不支持解构式赋值。


#### 常量 const代替let
常量的 类型必须标注。
`const MAX_POINTS: u32 = 100_000;`


#### 变量遮蔽(shadowing)

rust允许声明相同的变量名，后面声明的变量会 遮蔽掉 前面声明的。

变量遮蔽的用处在于，如果你在某个作用域内无需再使用之前的变量（在被遮蔽后，无法再访问到之前的同名变量），就可以重复的使用变量名字，而不用绞尽脑汁去想更多的名字。



### 基本类型


- 数值类型: 
  有符号整数 (i8, i16, i32, i64, isize)、 无符号整数 (u8, u16, u32, u64, usize) 、浮点数 (f32, f64)、以及有理数、复数
- 字符串：
  字符串==字面量==和字符串==切片== ==&str==
- 布尔类型： 
  true和false
- 字符类型: 
  表示单个 Unicode 字符，存储为 4 个字节
- 单元类型: 
  即 () ，其唯一的值也是 ()

rust 无法推导出下面的 类型
`let guess = "42".parse().expect("Not a number!");`

需要改成
`let guess: i32 = ...` 
或
`"42".parse::<i32>()`




#### 整数类型

默认 i32

|长度	|有符号类型|	无符号类型|
|--|--|--|
|8 位	|i8	|u8|
|16 位	|i16|	u16|
|32 位	|i32|	u32|
|64 位	|i64|	u64|
|128 位	|i128|	u128|
|视架构而定|	isize|	usize|

isize，usize， 32位CPU，就是32位， 64位CPU就是64位。


书写形式

|数字字面量	|示例|
|--|--|
|十进制	|98_222|
|十六进制	|0xff|
|八进制	|0o77|
|二进制	|0b1111_0000|
|字节 (仅限于 u8)	|b'A'|


#### 整型溢出
将 256赋予 u8时， 发生整型溢出。

debug模式编译时，发现溢出 就编译失败。

release 编译，不检查溢出， 按照补码循环溢出，
简而言之，大于该类型最大值的数值会被补码转换成该类型能够支持的对应数字的最小值。比如在 u8 的情况下，256 变成 0，257 变成 1，依此类推。

要显式处理可能的溢出，可以使用标准库针对原始数字类型提供的这些方法：
- 使用 wrapping_* 方法在所有模式下都按照==补码循环溢出==规则处理，例如 wrapping_add
- 如果使用 checked_* 方法时发生溢出，则返回 ==None== 值
- 使用 overflowing_* 方法返回==该值==和一个指示==是否存在溢出的布尔值==
- 使用 saturating_* 方法使值达到==最小值或最大值==

```rust
fn main() {
    let a : u8 = 255;
    let b = a.wrapping_add(20);  // <<<<<<
    println!("{}", b);  // 19
}
```


#### 浮点

f32, f64。
默认 f64

##### 浮点数陷阱
- 浮点数是你想要的数的==近似==表达
- 某些特性是反直觉的
  f32 ， f64 上的比较运算实现的是 std::cmp::PartialEq 特征(类似其他语言的接口)，
  但是并没有实现 std::cmp::Eq 特征，但是后者在其它数值类型上都有定义
  无法作为 HashMap 的 key

遵守
- 避免在浮点数上测试相等性
- 当结果在数学上可能存在未定义时，需要格外的小心




#### NaN
```rust
let x = (-42.0_f32).sqrt();
if x.is_nan() {
    println!("未定义的数学行为")
}
```

`-42.0_f32` ？？？  确实可以。 32._f32 不行。  32.0_f32 可以
21f32 可以。
22i32 可以

`let twenty_two = 22i32;`

`let one_million: i64 = 1_000_000;`

```rust
  let forty_twos = [
    42.0,
    42f32,
    42.0_f32,
  ];
```



#### 位运算
```
&
|
^
!
<<
>>
```



#### range

```rust
for i in 1..=5 {

}

for i in 'a'..='z' {
    
}
```

序列只允许用于数字或字符类型，
原因是：它们可以连续，同时编译器在编译期可以检查该序列是否为空，字符和数字值是 Rust 中仅有的可以用于判断是否为空的类型

。。字符类型，，char 还是 u8 ？


#### char

由于 Unicode 都是 4 个字节编码，因此字符类型也是占用 4 个字节：
```rust
fn main() {
    let x = '中';
    println!("字符'中'占用了{}字节的内存大小",std::mem::size_of_val(&x));
}
```

。。没说，类型是什么。。



#### 单元类型()
单元类型就是 () ，对，你没看错，就是 () ，唯一的值也是 () 

main 函数就返回这个单元类型 ()，你不能说 main 函数无返回值，
因为没有返回值的函数在 Rust 中是有单独的定义的：发散函数( diverge function )，顾名思义，无法收敛的函数。

例如常见的 println!() 的返回值也是单元类型 ()。

再比如，你可以用 () 作为 map 的值，表示我们不关注具体的值，只关注 key。 
这种用法和 Go 语言的 struct{} 类似，可以作为一个值用来占位，但是完全不占用任何内存。



#### 语句，表达式

基于表达式是函数式语言的重要特征，表达式总要返回值。

由于 let 是语句，因此不能将 let 语句赋值给其它值，如下形式是==错误==的：
`let b = (let a = 8);`



```rust
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {}", y);
}
```

注意 x + 1 不能以分号结尾，否则就会从表达式变成语句， ==表达式不能包含分号==。
这一点非常重要，一旦你在表达式后加上分号，它就会变成一条语句，再也不会返回一个值，请牢记！


```rust
    let y = if x % 2 == 1 {
        "odd"
    } else {
        "even"
    };        // 。。。 注意这个 ;;;;;;

    let z = if x % 2 == 1 { "odd" } else { "even" };
```


#### 函数

```rust
fn add(i: i32, j: i32) -> i32 {
   i + j
 }
```

![313db8d174ce666b1f762d026eddc904.png](resources/72605ff831f741899a9f437a50e4621a.png)



函数名和 变量都使用 蛇形命名法。
。。就是 全小写，单词之间 下划线 分隔。

函数的位置可以随便放。

每个函数参数都需要标注类型。
。。写go，坑惨了。。


无返回
```rust
fn report<T: Debug>(item: T) {      // 隐式返回 ()

}
// 上下的返回是等价的。
fn add(x: u32, y: u32) -> () {      // 显式返回 ()

}

```


#### 返回!

当用 ! 作函数返回类型的时候，表示该函数永不返回( diverge function )，特别的，这种语法往往用做会==导致程序崩溃的函数==：

```rust
fn dead_end() -> ! {
  panic!("你已经到了穷途末路，崩溃吧！");
}

// loop 永不返回。
fn forever() -> ! {
  loop {
    //...
  };
}
```


### ==所有权和借用==

https://course.rs/basic/ownership/ownership.html


释放内存，有3种
- 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，典型代表：Java、Go
- 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
- 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查


Rust 使用了 第三种， 并且 检查 只发生在 编译期。


糟糕的代码
```rust
int* foo() {
    int a;          // 变量a的作用域开始
    a = 100;
    char *c = "xyz";   // 变量c的作用域开始
    return &a;
}                   // 变量a和c的作用域结束
```
局部变量 a 存在栈中，在离开作用域后，a 所申请的栈上内存都会被系统回收，从而造成了 悬空指针(Dangling Pointer) 的问题。
这是一个非常典型的内存安全问题，虽然编译可以通过，但是运行的时候会出现错误, 很多编程语言都存在。

再来看变量 c，c 的值是常量字符串，存储于常量区，可能这个函数我们只调用了一次，也可能我们不再会使用这个字符串，但 "xyz" 只有当整个==程序结束后==系统才能回收这片内存。



栈，堆
性能：
写入方面： 入栈快， 因为无需 分配新的空间，
读取方面： 得益于CPU高速缓存，使得CPU可以减少对内存的访问，高速缓存和内存的访问速度的差异在 10倍以上。 栈数据可以直接存储在CPU高速缓存中，堆数据只能存储在内存中。 访问堆上的数据 需要先访问栈 获得指针。


#### ==所有权原则==

- Rust 中每一个值都被一个变量所拥有，该变量被称为值的所有者
- 一个值同时只能被一个变量所拥有，或者说一个值只能拥有一个所有者
- 当所有者(变量)离开作用域范围时，这个值将被丢弃(drop)

```rust
    let s1 = String::from("hi");
    let s2 = s1;         // 所有权发生了转移
    // println!("{}", s1);      // 编译失败。value used here after move
```

这个例子中，x 只是引用了存储在二进制中的字符串 "hello, world"，并没有持有所有权。
因此 let y = x 中，仅仅是对该引用进行了拷贝
```rust
    let x: &str = "hello, world";
    let y = x;
    println!("{},{}",x,y);      // ok
```



Rust ==永远不会自动创建数据的 “深拷贝==”。
因此，任何自动的复制都不是深拷贝，可以被认为对运行时性能影响较小。

深拷贝，只能 手动 clone()。
```rust
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
```


如果一个类型拥有 ==Copy 特征==，一个旧的变量在被赋值给其他变量后仍然可用。
任何基本类型的组合可以 Copy ，不需要分配内存或某种形式资源的类型是可以 Copy 的

一些 Copy 的类型：
- 所有整数类型，比如 u32
- 布尔类型，bool，它的值是 true 和 false
- 所有浮点数类型，比如 f64
- 字符类型，char
- 元组，当且仅当其包含的类型也都是 Copy 的时候。比如，(i32, i32) 是 Copy 的，但 (i32, String) 就不是
- 不可变引用 &T ，例如转移所有权中的最后一个例子，但是注意: 可变引用 &mut T 是不可以 Copy的



将值传递给函数，一样会发生 移动 或者 复制，就跟 let 语句一样，下面的代码展示了所有权、作用域的规则：
```rust
fn main() {
    let s = String::from("hello");  // s 进入作用域

    takes_ownership(s);             // s 的值移动到函数里 ...
                                    // ... 所以到这里不再有效

    let x = 5;                      // x 进入作用域

    makes_copy(x);                  // x 应该移动函数里，
                                    // 但 i32 是 Copy 的，所以在后面可继续使用 x

} // 这里, x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
  // 所以不会有特殊操作

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
} // 这里，some_integer 移出作用域。不会有特殊操作
```


函数返回值的所有权转移
```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership 将返回值
                                        // 移给 s1

    let s2 = String::from("hello");     // s2 进入作用域

    let s3 = takes_and_gives_back(s2);  // s2 被移动到
                                        // takes_and_gives_back 中,
                                        // 它也将返回值移给 s3
} // 这里, s3 移出作用域并被丢弃。s2 也移出作用域，但已被移走，
  // 所以什么也不会发生。s1 移出作用域并被丢弃

fn gives_ownership() -> String {             // gives_ownership 将返回值移动给
                                             // 调用它的函数

    let some_string = String::from("hello"); // some_string 进入作用域.

    some_string                              // 返回 some_string 并移出给调用的函数
}

// takes_and_gives_back 将传入字符串并返回该值
fn takes_and_gives_back(a_string: String) -> String { // a_string 进入作用域

    a_string  // 返回 a_string 并移出给调用的函数
}
```



##### 引用和借用
如果仅仅支持通过转移所有权的方式获取一个值，那会让程序变得复杂。 
Rust 能否像其它编程语言一样，使用某个变量的指针或者引用呢？答案是可以。

Rust 通过 ==借用==(Borrowing) 这个概念来达成上述的目的，获取变量的引用，称之为借用(borrowing)。


> 引用与解引用

常规引用是一个指针类型，指向了对象存储的内存地址。在下面代码中，我们创建一个 i32 值的引用 y，然后使用解引用运算符来解出 y 所使用的值:

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

用 s1 的引用作为参数传递给 calculate_length 函数，而不是把 s1 的所有权转移给该函数：
```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

==& 符号即是引用，它们允许你使用值，但是不获取所有权==

borrow来的值，不能修改。
。。所以 & 和 mut 不能一起用。 或者说 & 和 mut 一起的时候， & 起效，所以 不可修改 。。？
。。当我没说。。

可变引用
```rust
fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

> 同一作用域，特定数据只能有一个可变引用：

下面会报错。
```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;

println!("{}, {}", r1, r2);
```

这种限制的好处就是使 Rust 在==编译期就避免数据竞争==，数据竞争可由以下行为造成：
- 两个或更多的指针同时访问同一数据
- 至少有一个指针被用来写入数据
- 没有同步数据访问的机制


> 可变引用与不可变引用不能同时存在


引用的作用域 s 从创建开始，一直持续到它最后一次使用的地方，这个跟变量的作用域有所不同，
变量的作用域从创建持续到某一个花括号 }

对于这种编译器优化行为，Rust 专门起了一个名字 —— Non-Lexical Lifetimes(**NLL**)，专门用于找到某个引用在作用域(})结束前就不再被使用的代码位置。


#### 悬垂引用(Dangling References)

悬垂引用也叫做悬垂指针，意思为指针指向某个值后，这个值被释放掉了，而指针仍然存在，其指向的内存可能不存在任何值或已被其它变量重新使用。
在 Rust 中编译器可以确保引用永远也不会变成悬垂状态：当你获取数据的引用后，编译器可以确保数据不会在引用结束前被释放，要想释放数据，必须先停止其引用的使用。

```rust
fn main() {
    let reference_to_nothing = dangle();
}

// help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
// help: consider using the `'static` lifetime
fn dangle() -> &String {
    let s = String::from("hello");

    &s
}

// ok, String 的 所有权被转移给外面的调用者。
fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```


#### 借用规则总结
借用规则如下：
- 同一时刻，你只能拥有要么一个可变引用, 要么任意多个不可变引用
- 引用必须总是有效的




### 复合类型
最典型的就是结构体 struct 和枚举 enum


#### 字符串

`let my_name = "Pascal";`
my_name 是 &str 类型， 不是 String。


#### 切片

对于字符串而言，切片就是对 String 类型中某一部分的引用，它看起来像这样：
```rust
let s = String::from("hello world");

let hello = &s[0..5];       // 右半开区间
let world = &s[6..11];

let slice = &s[0..2];
let slice = &s[..2];        // 等价于上面

let len = s.len();
let slice = &s[4..len];
let slice = &s[4..];        // 等价于上面

let slice = &s[0..len];
let slice = &s[..];     // 等价于上面
```


在对字符串使用切片语法时需要格外小心，
切片的索引必须落在字符之间的边界位置，也就是 UTF-8 字符的边界，
例如中文在 UTF-8 中占用三个字节，下面的代码就会==崩溃==：
```rust
let s = "中国人";
let a = &s[0..2];
println!("{}",a);
```




借用的规则：当我们已经有了可变借用时，就无法再拥有不可变的借用。
因为 clear 需要清空改变 String，因此它需要一个可变借用（利用 VSCode 可以看到该方法的声明是 pub fn clear(&mut self) ，参数是对自身的可变借用 ）；
而之后的 println! 又使用了不可变借用，也就是在 s.clear() 处==可变借用与不可变借用试图同时生效==，因此编译无法通过。
```rust
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear();    // error!

    println!("the first word is: {}", word);
}
fn first_word(s: &String) -> &str {
    &s[..1]
}
```



```rust
let a = [1, 2, 3, 4, 5];
let slice = &a[1..3];
assert_eq!(slice, &[2, 3]);
```
该数组切片的类型是 &[i32]



Rust 中的字符是 Unicode 类型，因此每个字符占据 4 个字节内存空间，
但是在字符串中不一样，字符串是 UTF-8 编码，也就是字符串中的字符所占的字节数是变化的(1 - 4)


Rust 在语言级别，只有一种字符串类型： str，它通常是以引用类型出现 &str，也就是上文提到的字符串切片。
虽然语言级别只有上述的 str 类型，但是在==标准库==里，还有多种不同用途的字符串类型，其中使用最广的即是 String 类型。


当 Rust 用户提到字符串时，往往指的就是 String 类型和 &str 字符串切片类型，这两个类型都是 UTF-8 编码。


Rust 的标准库还提供了其他类型的字符串，例如 OsString， OsStr， CsString 和 CsStr 等，
注意到这些名字都以 String 或者 Str 结尾了吗？它们分别对应的是具有所有权和被借用的变量。



#### String 与 &str 的转换

&str -> String
```rust
String::from("hello,world")
"hello,world".to_string()
```

String -> &str
```rust
fn main() {
    let s = String::from("hello,world!");
    say_hello(&s);
    say_hello(&s[..]);
    say_hello(s.as_str());
}

fn say_hello(s: &str) {
    println!("{}",s);
}
```



rust 没有 `str[0]`


#### 操作字符串

##### push()
- push() 方法追加字符 char
- push_str() 方法追加字符串字面量

字符串变量必须由 mut 关键字修饰。
在原有的字符串上追加，并不会返回新的字符串

```rust
    let mut s = String::from("Hello ");

    s.push_str("rust");
    println!("追加字符串 push_str() -> {}", s);

    s.push('!');
    println!("追加字符 push() -> {}", s);
```




##### insert()
- insert() 方法插入单个字符 char
- insert_str() 方法插入字符串字面量

- 第一个参数是字符（串）插入位置的索引，
- 第二个参数是要插入的字符（串），
索引从 0 开始计数，如果越界则会发生错误

修改原来的字符串
字符串变量必须由 mut 关键字修饰。

```rust
    let mut s = String::from("Hello rust!");
    s.insert(5, ',');
    println!("插入字符 insert() -> {}", s);
    s.insert_str(6, " I like");
    println!("插入字符串 insert_str() -> {}", s);
```





##### replace()

适用于 String 和 &str 类型

- 第一个参数是要被替换的字符串，
- 第二个参数是新的字符串。
该方法会替换所有匹配到的字符串

该方法是返回一个新的字符串，而不是操作原来的字符串。

```rust
    let string_replace = String::from("I like rust. Learning rust is my favorite!");
    let new_string_replace = string_replace.replace("rust", "RUST");
    dbg!(new_string_replace);
```



##### replacen()
适用于 String 和 &str 类型。

replacen() 方法接收三个参数，
前两个参数与 replace() 方法一样，
第三个参数则表示替换的个数。

该方法是返回一个新的字符串，而不是操作原来的字符串

```rust
    let string_replace = "I like rust. Learning rust is my favorite!";
    let new_string_replacen = string_replace.replacen("rust", "RUST", 1);
    dbg!(new_string_replacen);
```



##### replace_range()
适用于 String 类型

第一个参数是要替换字符串的范围（Range），
第二个参数是新的字符串。

该方法是直接操作原来的字符串，不会返回新的字符串。
该方法需要使用 mut 关键字修饰。

```rust
    let mut string_replace_range = String::from("I like rust!");
    string_replace_range.replace_range(7..8, "R");
    dbg!(string_replace_range);
```


##### pop()
仅适用于 String 类型。

删除并返回字符串的最后一个字符

该方法是直接操作原来的字符串
返回值是一个 Option 类型，如果字符串为空，则返回 None

```rust
    let mut string_pop = String::from("rust pop 中文!");
    let p1 = string_pop.pop();
    let p2 = string_pop.pop();
    dbg!(p1);
    dbg!(p2);
    dbg!(string_pop);
```

##### remove()
仅适用于 String 类型。

删除并返回字符串中指定位置的字符

该方法是直接操作原来的字符串

返回值是删除位置的字符串，只接收一个参数，表示该字符起始索引位置
。。一个字符 多个 byte，所以 说 起始索引位置

remove() 方法是按照字节来处理字符串的，如果参数所给的位置不是合法的字符边界，则会发生错误。

```rust
    let mut string_remove = String::from("测试remove方法");
    println!(
        "string_remove 占 {} 个字节",
        std::mem::size_of_val(string_remove.as_str())
    );
    // 删除第一个汉字
    string_remove.remove(0);
    // 下面代码会发生错误
    // string_remove.remove(1);
    // 直接删除第二个汉字
    // string_remove.remove(3);
```



##### truncate()
仅适用于 String 类型。
删除字符串中从指定位置开始到结尾的全部字符
该方法是直接操作原来的字符串
无返回值
按照字节来处理字符串的，如果参数所给的位置不是合法的字符边界，则会发生错误。
```rust
    let mut string_truncate = String::from("测试truncate");
    string_truncate.truncate(3);
```


##### chear()
仅适用于 String 类型。
该方法是直接操作原来的字符串
```rust
    let mut string_clear = String::from("string clear");
    string_clear.clear();
```


##### +, +=

使用 + 或者 += 连接字符串，要求右边的参数必须为字符串的切片引用（Slice）类型。
其实当调用 + 的操作符时，相当于调用了 std::string 标准库中的 add() 方法，这里 add() 方法的第二个参数是一个引用的类型。
因此我们在使用 +， 必须传递切片引用类型。
不能直接传递 String 类型。
+ 是返回一个新的字符串，所以变量声明可以不需要 mut 关键字修饰。

```rust
    let string_append = String::from("hello ");
    let string_rust = String::from("rust");
    // &string_rust会自动解引用为&str
    let result = string_append + &string_rust;
    let mut result = result + "!"; // `result + "!"` 中的 `result` 是不可变的
    result += "!!!";
```

##### add()
`fn add(self, s: &str) -> String`

```rust
    let s1 = String::from("hello,");
    let s2 = String::from("world!");
    // 在下句中，s1的所有权被转移走了，因此后面不能再使用s1
    let s3 = s1 + &s2;
    assert_eq!(s3,"hello,world!");
    // 下面的语句如果去掉注释，就会报错
    // println!("{}",s1);
```

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

// String = String + &str + &str + &str + &str
let s = s1 + "-" + &s2 + "-" + &s3;
```


##### format!() 连接字符串

```rust
    let s1 = "hello";
    let s2 = String::from("rust");
    let s = format!("{} {}!", s1, s2);
    println!("{}", s);
```




#### 字符串转义
```rust
    // \u 可以输出一个 unicode 字符
    let unicode_codepoint = "\u{211D}";

    // 通过 \ + 字符的十六进制表示，转义输出一个字符
    let byte_escape = "I'm writing \x52\x75\x73\x74!";
```

不转义
```rust
    println!("{}", "hello \\x52\\x75\\x73\\x74");

    // 如果字符串包含双引号，可以在开头和结尾加 #
    let quotes = r#"And then I said: "There is no escape!""#;

    // 如果还是有歧义，可以继续增加，没有限制
    let longer_delimiter = r###"A string with "# in it. And even "##!"###;
```


#### 操作 UTF-8 字符串

字符
如果你想要以 Unicode 字符的方式遍历字符串，最好的办法是使用 chars 方法，例如：
```rust
for c in "中国人".chars() {
    println!("{}", c);
}
```


字节
```rust
for b in "中国人".bytes() {
    println!("{}", b);
}
```


想要准确的从 UTF-8 字符串中获取子串是较为复杂的事情，例如想要从 holla中国人नमस्ते 这种变长的字符串中取出某一个子串，使用标准库你是做不到的。 你需要在 crates.io 上搜索 utf8 来寻找想要的功能。






变量在离开作用域后，就自动释放其占用的内存

```rust
{
    let s = String::from("hello"); // 从此处起，s 是有效的

    // 使用 s
}                                  // 此作用域已结束，
                                   // s 不再有效，内存被释放
```

与其它系统编程语言的 free 函数相同，Rust 也提供了一个释放内存的函数： drop，
但是不同的是，其它语言要手动调用 free 来释放每一个变量占用的内存，
而 Rust 则在变量离开作用域时，自动调用 drop 函数: 上面代码中，Rust 在结尾的 } 处自动调用 drop。

。。因为 rust 知道 owner， 所以由 owner 释放。  C++之类的不知道。





### 元组

https://course.rs/basic/compound-type/tuple.html

元组长度固定，元素顺序固定

创建元组：
```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```


```rust
// multi return value
fn main() {
    let s1 = String::from("hello");
    let (s2, len) = calculate_length(s1);
    println!("The length of '{}' is {}.", s2, len);
}
fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() 返回字符串的长度
    (s, length)
}
```

在其他语言中，可以用结构体来声明一个三维空间中的点，
例如 Point(10, 20, 30)，虽然使用 Rust 元组也可以做到：(10, 20, 30)，但是这样写有个非常重大的缺陷：
不具备任何清晰的含义



### 结构体

struct 和 tuple 类似的地方：都是由多种类型组合而成
不同： struct 可以为 每个字段 起一个 有含义的名称。
因此 struct 更灵活，更强大，你不需要依赖 字段顺序 来访问 和解析它们。

语法
一个结构体由几部分组成：
- 通过关键字 struct 定义
- 一个清晰明确的结构体 名称
- 几个有名字的结构体 字段

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}               // 没有;;;
```

```rust
let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};          // 有;;;

user1.email = String::from("anotheremail@example.com");
```

- 初始化实例时，每个字段都需要进行初始化
- 初始化时的字段顺序不需要和结构体定义时的顺序一致


必须要将结构体实例声明为可变的，才能修改其中的字段，
Rust 不支持将某个结构体某个字段标记为可变。


```rust
// 简化create
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,          // ..
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

当函数参数和结构体字段同名时，可以直接使用缩略的方式进行初始化，跟 TypeScript 中一模一样。


#### struct更新
。。不过应该算 复制。 真是更新。。主要是 user1 不能再使用了。 属性的==所有权 被转移==给 user2 了。。
。。user1 中具有 Copy trait的 不会转移，只是 克隆。 而且 Copy 属性 可以访问。
。。 所以如果 struct User 全是 Copy 的， 那么 user1 user2 是不是都可以使用？

```rust
  let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"),
        sign_in_count: user1.sign_in_count,
    };
```

```rust
// 简化
  let user2 = User {
        email: String::from("another@example.com"),
        ..user1     // 必须在尾部
    };
```

结构体更新语法跟赋值语句 = 非常相像，因此在上面代码中，user1 的==部分==字段所有权被转移到 user2 中：username 字段发生了所有权转移，作为结果，user1 无法再被使用。

明明有三个字段进行了自动赋值，为何只有 username 发生了所有权转移？
仔细回想一下所有权那一节的内容，我们提到了 ==Copy== 特征：实现了 Copy 特征的类型无需所有权转移，可以直接在赋值时进行 数据拷贝，
其中 bool 和 u64 类型就实现了 Copy 特征，因此 active 和 sign_in_count 字段在赋值给 user2 时，仅仅发生了==拷贝==，而不是所有权转移。

值得注意的是：username 所有权被转移给了 user2，导致了 user1 无法再被使用，但是并不代表 user1 内部的其它字段不能被继续使用

```rust
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
let user2 = User {
    active: user1.active,
    username: user1.username,
    email: String::from("another@example.com"),
    sign_in_count: user1.sign_in_count,
};
println!("{}", user1.active);
// 下面这行会报错
println!("{:?}", user1);
```



```rust
 struct File {
   name: String,
   data: Vec<u8>,
 }
```
上面的结构体 在 内存中的 排列如下

![ce113aa1f017056228f06982249bd0e6.png](resources/a438ae77a13e4ab5818da234e8af600d.png)

看出 File 结构体两个字段 name 和 data 分别拥有底层两个 [u8] 数组的所有权(String 类型的底层也是 [u8] 数组)，
通过 ptr 指针指向底层数组的内存地址，这里你可以把 ptr 指针理解为 Rust 中的引用类型。

侧面印证了：把结构体中具有所有权的字段转移出去后，将无法再访问该字段，但是可以正常访问其它的字段。


#### tuple struct

结构体必须要有名称，
结构体的==字段可以没有名称==，这种结构体长得很像元组，因此被称为元组结构体

```rust
    struct Color(i32, i32, i32);
    struct Point(i32, i32, i32);

    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
```

元组结构体在你希望有一个整体名称，但是又不关心里面字段的名称时将非常有用。
例如上面的 Point 元组结构体，众所周知 3D 点是 (x, y, z) 形式的坐标点，因此我们无需再为内部的字段逐一命名为：x, y, z。


#### 单元结构体(Unit-like Struct)

如果你定义一个类型，但是不关心该类型的内容, 只关心它的行为时，就可以使用 单元结构体

```rust
struct AlwaysEqual;

let subject = AlwaysEqual;

// 我们不关心 AlwaysEqual 的字段数据，只关心它的行为，因此将它声明为单元结构体，然后再为它实现某个特征
impl SomeTrait for AlwaysEqual {

}
```

#### 结构体数据的所有权

你也可以让 User 结构体从其它对象借用数据，不过这么做，就需要引入生命周期(lifetimes)这个新概念（也是一个复杂的概念），
简而言之，生命周期能确保结构体的作用范围要比它所借用的数据的作用范围要小。

```rust
struct User {
    username: &str,
    email: &str,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    let user1 = User {
        email: "someone@example.com",
        username: "someusername123",
        active: true,
        sign_in_count: 1,
    };
}
```
```text
error[E0106]: missing lifetime specifier
 --> src/main.rs:2:15
  |
2 |     username: &str,
  |               ^ expected named lifetime parameter // 需要一个生命周期
  |
help: consider introducing a named lifetime parameter // 考虑像下面的代码这样引入一个生命周期
  |
1 ~ struct User<'a> {
2 ~     username: &'a str,
  |
```

这些在 生命周期 中讲。


使用 `#[derive(Debug)]` 来打印结构体的信息

结构体不带 `#[derive(Debug)]`时，
pringln!("{}", xx), 会报错:
```text
error[E0277]: `Rectangle` doesn't implement `std::fmt::Display`
```

println!("rect1 is {:?}", rect1); 也报错
```text
error[E0277]: `Rectangle` doesn't implement `Debug`
```
会提示你：
```text
= help: the trait `Debug` is not implemented for `Rectangle`
= note: add `#[derive(Debug)]` to `Rectangle` or manually `impl Debug for Rectangle`
```

Rust默认 不会为我们实现 Debug， 我们有2种方法实现：
- 手动实现
- 使用 derive 派生实现

后者简单，但是有限制。具体看 附录D

加上 `#[derive(Debug)]` 后， 
{:?} 可以了。 
{:#?} 的输出表现更好。

还有一个简单的输出 debug信息的 方法， 就是 `dbg!` 宏。
它会拿走表达式的所有权，然后打印出相应的文件名、行号等 debug 信息，
当然还有我们需要的表达式的求值结果。
除此之外，它最终还会把表达式值的==所有权返回==！

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
```


### 枚举

```rust
enum PokerSuit {
  Clubs,
  Spades,
  Diamonds,
  Hearts,
}
```
枚举类型是一个类型，它会包含所有可能的枚举成员, 而枚举值是该类型中的具体某个成员的实例。

```rust
let heart = PokerSuit::Hearts;
let diamond = PokerSuit::Diamonds;
```

```rust
enum PokerSuit {
    Clubs,
    Spades,
    Diamonds,
    Hearts,
}

struct PokerCard {
    suit: PokerSuit,
    value: u8
}

fn main() {
   let c1 = PokerCard {
       suit: PokerSuit::Clubs,
       value: 1,
   };
   let c2 = PokerCard {
       suit: PokerSuit::Diamonds,
       value: 12,
   };
}
```


```rust
enum PokerCard {
    Clubs(u8),
    Spades(u8),
    Diamonds(u8),
    Hearts(u8),
}

fn main() {
   let c1 = PokerCard::Spades(5);
   let c2 = PokerCard::Diamonds(13);
}
```

```rust
enum PokerCard {
    Clubs(u8),
    Spades(u8),
    Diamonds(char),
    Hearts(char),
}

fn main() {
   let c1 = PokerCard::Spades(5);
   let c2 = PokerCard::Diamonds('A');
}
```

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```


任何类型的数据都可以放入枚举成员中: 例如字符串、数值、结构体甚至另一个枚举。
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let m1 = Message::Quit;
    let m2 = Message::Move{x:1,y:1};
    let m3 = Message::ChangeColor(255,255,0);
}
```
Quit 没有任何关联数据
Move 包含一个匿名结构体
Write 包含一个 String 字符串
ChangeColor 包含三个 i32


> 同一化类型

例如我们有一个 WEB 服务，需要接受用户的长连接，假设连接有两种：TcpStream 和 TlsStream，
但是我们希望对这两个连接的处理流程相同，也就是用同一个函数来处理这两个连接，代码如下：
```rust
fn new (stream: TcpStream) {
  let mut s = stream;
  if tls {
    s = negotiate_tls(stream)
  }

  // websocket是一个WebSocket<TcpStream>
  // 或者
  // WebSocket<native_tls::TlsStream<TcpStream>>类型
  websocket = WebSocket::from_raw_socket(stream, ......)
}
```
此时，枚举类型就能帮上大忙：
```rust
enum Websocket {
  Tcp(Websocket<TcpStream>),
  Tls(Websocket<native_tls::TlsStream<TcpStream>>),
}
```
。。没有理解。



#### Option

doc: https://doc.rust-lang.org/std/option/enum.Option.html



```rust
enum Option<T> {
    Some(T),
    None,
}
```

Option 是标准库
无需使用 Option:: 前缀就可直接使用 Some 和 None

```rust
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
```

总的来说，为了使用 `Option<T>` 值，需要编写处理每个成员的代码。
你想要一些代码只当拥有 Some(T) 值时运行，允许这些代码使用其中的 T。
也希望一些代码在值为 None 时运行，这些代码并没有一个可用的 T 值。
match 表达式就是这么一个处理枚举的控制流结构：它会根据枚举的成员运行不同的代码，这些代码可以使用匹配到的值中的数据。

这里先简单看一下 match 的大致模样，在模式匹配中，我们会详细讲解：
```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```


### 数组

在 Rust 中，最常用的数组有两种，
第一种是速度很快但是长度固定的 array，
第二种是可动态增长的但是有性能损耗的 Vector，

在本书中，我们称 array 为数组，Vector 为动态数组。

数组的三要素：
- 长度固定
- 元素必须有相同的类型
- 依次线性排列


```rust
fn main() {
    let a = [1, 2, 3, 4, 5];
}
```

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

某个值重复出现 N 次的数组：
```rust
let a = [3; 5];
```
3是值，5是长度


访问
```rust
fn main() {
    let a = [9, 8, 7, 6, 5];

    let first = a[0]; // 获取a数组第一个元素
    let second = a[1]; // 获取第二个元素
}
```
数组的索引下标是从 0 开始的


数组越界会 panic

#### 数组元素为非基础类型
`let array = [String::from("rust is good!"); 8];` 会报错：

```text
the trait `std::marker::Copy` is not implemented for `String`
```

正确的写法
```rust
let array: [String; 8] = std::array::from_fn(|_i| String::from("rust is good!"));

println!("{:#?}", array);
```
。。。？？？


#### 数组切片
```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
let slice: &[i32] = &a[1..3];
assert_eq!(slice, &[2, 3]);
```

slice 的类型是`&[i32]`，与之对比，数组的类型是`[i32;5]`

切片特点：
- 切片的长度可以与数组不同，并不是固定的，而是取决于你使用时指定的起始和结束位置
- 创建切片的==代价非常小==，因为切片只是针对底层数组的一个引用
- 切片类型[T]拥有不固定的大小，而切片引用类型&[T]则具有固定的大小，因为 Rust 很多时候都需要固定大小数据类型，因此&[T]更有用,&str字符串切片也同理


-----------------

#### 综合例子

```rust
fn main() {
  // 编译器自动推导出one的类型
  let one             = [1, 2, 3];
  // 显式类型标注
  let two: [u8; 3]    = [1, 2, 3];
  let blank1          = [0; 3];
  let blank2: [u8; 3] = [0; 3];

  // arrays是一个二维数组，其中每一个元素都是一个数组，元素类型是[u8; 3]
  let arrays: [[u8; 3]; 4]  = [one, two, blank1, blank2];

  // 借用arrays的元素用作循环中
  for a in &arrays {
    print!("{:?}: ", a);
    // 将a变成一个迭代器，用于循环
    // 你也可以直接用for n in a {}来进行循环
    for n in a.iter() {
      print!("\t{} + 10 = {}", n, n+10);
    }

    let mut sum = 0;
    // 0..a.len,是一个 Rust 的语法糖，其实就等于一个数组，元素是从0,1,2一直增加到到a.len-1
    for i in 0..a.len() {
      sum += a[i];
    }
    println!("\t({:?} = {})", a, sum);
  }
}
```



注意
- 数组类型容易跟数组切片混淆，[T;n]描述了一个数组的类型，而[T]描述了切片的类型， 因为切片是运行期的数据结构，它的长度无法在编译期得知，因此不能用[T;n]的形式去描述
- [u8; 3]和[u8; 4]是不同的类型，数组的长度也是类型的一部分
- 在实际开发中，使用最多的是数组切片[T]，我们往往通过引用的方式去使用&[T]，因为后者有固定的类型大小



### 流程控制

https://course.rs/basic/flow-control.html


```rust
if condition == true {
    // A...
} else {
    // B...
}
```

```rust
let condition = true;
let number = if condition {
    5
} else {
    6
};
```

```rust
    if n % 4 == 0 {
        println!("number is divisible by 4");
    } else if n % 3 == 0 {
        println!("number is divisible by 3");
    } else if n % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
```


---

```rust
fn main() {
    for i in 1..=5 {
        println!("{}", i);
    }
}
```

#### for item in &mut set2

|使用方法	|等价使用方式	|所有权|
|--|--|--|
|for item in collection	|for item in IntoIterator::into_iter(collection)	|转移所有权|
|for item in &collection	|for item in collection.iter()	|不可变借用|
|for item in &mut collection	|for item in collection.iter_mut()	|可变借用|

for 元素 in 集合 {

使用 for 时我们往往使用集合的==引用形式==，除非你不想在后面的代码中继续使用该集合（比如我们这里使用了 container 的引用）。
如果不使用引用的话，所有权会被==转移（move==）到 for 语句块中，后面就无法再使用这个集合了)：

对于实现了 ==copy 特征==的数组(例如 [i32; 10] )而言， for item in arr 并==不会==把 arr 的所有权转移，而是直接对其进行了==拷贝==，因此循环之后仍然可以使用 arr 。

```rust
for item in &container {
  // ...
}
```

```rust
for item in &mut collection {
  // ...
}
```

```rust
fn main() {
    let a = [4, 3, 2, 1];
    // `.iter()` 方法把 `a` 数组变成一个迭代器
    for (i, v) in a.iter().enumerate() {
        println!("第{}个元素是{}", i + 1, v);
    }
}
```

```rust
for _ in 0..10 {
  // ...
}
```

```rust
// 第一种
let collection = [1, 2, 3, 4, 5];
for i in 0..collection.len() {
  let item = collection[i];
  // ...
}

// 第二种
// 。。这个就是 for element in a.iter() { 的缩写。
for item in collection {

}
```
第一种和第二种的对比：
- 性能，
  第一种，使用了 索引访问，会进行边界检查，==降低性能==。
  第二种不会触发这种检查，因为编译器在==编译时==就完成分析，并证明这种访问是合法的。
- 安全
  第一种，对集合的访问是 非连续的，存在：2次访问之间，collection发生了变化，导致==脏数据==产生。
  第二种，是连续访问，不存在这个风险(由于所有权的限制，访问过程中，数据并不会发生变化)




```rust
    let mut n = 0;

    while n <= 5  {
        println!("{}!", n);

        n = n + 1;
    }
```

```rust
    let mut n = 0;

    loop {
        if n > 5 {
            break
        }
        println!("{}", n);
        n+=1;
    }
```

注意
- break 可以单独使用，也可以带一个返回值，有些类似 return
- loop 是一个表达式，因此可以返回一个值



### match

- 对 Option 的处理
- 代替 else if


```rust
enum Direction {
    East,
    West,
    North,
    South,
}

fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        Direction::North | Direction::South => {
            println!("South or North");
        },
        _ => println!("West"),
    };
}
```

- match 的匹配必须要穷举出所有可能，因此这里用 _ 来代表未列出的所有可能性
- match 的每一个分支都必须是一个==表达式==，且所有分支的表达式最终返回值的类型必须相同
- X | Y，类似逻辑运算符 或，代表该分支可以匹配 X 也可以匹配 Y，只要满足一个即可

```rust
match target {
    模式1 => 表达式1,
    模式2 => {
        语句1;
        语句2;
        表达式2
    },
    _ => 表达式3
}
```

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny =>  {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

#### let match

```rust
enum IpAddr {
   Ipv4,
   Ipv6
}

fn main() {
    let ip1 = IpAddr::Ipv6;
    let ip_str = match ip1 {            // 赋值
        IpAddr::Ipv4 => "127.0.0.1",
        _ => "::1",
    };

    println!("{}", ip_str);
}
```


#### 模式绑定

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState), // 25美分硬币
}

// 获取到 25 美分硬币上刻印的州的名称：
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

在匹配 Coin::Quarter(state) 模式时，我们把它内部存储的值绑定到了 state 变量上，因此 state 变量就是对应的 UsState 枚举类型。


```rust
enum Action {
    Say(String),
    MoveTo(i32, i32),
    ChangeColorRGB(u16, u16, u16),
}

fn main() {
    let actions = [
        Action::Say("Hello Rust".to_string()),
        Action::MoveTo(1,2),
        Action::ChangeColorRGB(255,255,0),
    ];
    for action in actions {
        match action {
            Action::Say(s) => {
                println!("{}", s);
            },
            Action::MoveTo(x, y) => {
                println!("point from (0, 0) move to ({}, {})", x, y);
            },
            Action::ChangeColorRGB(r, g, _) => {
                println!("change color into '(r:{}, g:{}, b:0)', 'b' has been ignored",
                    r, g,
                );
            }
        }
    }
}
```


_通配符

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```

除了_通配符，用一个变量来承载其他情况也是可以的。

```rust
#[derive(Debug)]
enum Direction {
    East,
    West,
    North,
    South,
}

fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        other => println!("other direction: {:?}", other),
    };
}
```


#### if let

有时会遇到只有==一个模式的值==需要被处理，其它值直接忽略的场景，如果用 match 来处理就要写成下面这样：

```rust
    let v = Some(3u8);
    match v {
        Some(3) => println!("three"),
        _ => (),
    }
```

```rust
let v = Some(3u8);
if let Some(3) = v {
    println!("three");
}
```

当你只要匹配一个条件，且忽略其他条件时就用 if let ，否则都用 match。



#### matches! 宏

宏：matches!，它可以将一个表达式跟模式进行匹配，然后返回匹配的结果 true or false。

假设有以下的枚举 和 枚举数组
```rust
enum MyEnum {
    Foo,
    Bar
}

fn main() {
    let v = vec![MyEnum::Foo,MyEnum::Bar,MyEnum::Foo];
}
```

现在如果想对 v 进行过滤，只保留类型是 MyEnum::Foo 的元素，你可能想这么写：
`v.iter().filter(|x| x == MyEnum::Foo);`
会报错，因为你无法将 x 直接跟一个枚举成员进行比较
可以使用 match 来完成，但是会导致代码更为啰嗦，是否有更简洁的方式？
答案是使用 matches!：
`v.iter().filter(|x| matches!(x, MyEnum::Foo));`


更多例子
```rust
let foo = 'f';
assert!(matches!(foo, 'A'..='Z' | 'a'..='z'));

let bar = Some(4);
assert!(matches!(bar, Some(x) if x > 2));
```


------------
变量遮蔽
```rust
fn main() {
   let age = Some(30);
   println!("在匹配前，age是{:?}",age);
   if let Some(age) = age {
       println!("匹配出来的age是{}",age);
   }

   println!("在匹配后，age是{:?}",age);
}
```
```text
在匹配前，age是Some(30)
匹配出来的age是30
在匹配后，age是Some(30)
```




### 解构Option

```rust
enum Option<T> {
    Some(T),
    None,
}
```

一个变量要么有值：Some(T), 要么为空：None。


从 Some 中取出其内部的 T 值
以及处理没有值的情况

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

-----------

#### 模式适用场景

模式一般由以下内容组合而成：
- 字面值
- 解构的数组、枚举、结构体或者元组
- 变量
- 通配符
- 占位符


```rust
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    _ => EXPRESSION,
}
```


```rust
if let PATTERN = SOME_VALUE {

}
```

```rust
// Vec是动态数组
let mut stack = Vec::new();

// 向数组尾部插入元素
stack.push(1);
stack.push(2);
stack.push(3);

// stack.pop从数组尾部弹出元素
while let Some(top) = stack.pop() {
    println!("{}", top);
}
```


```rust
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```

`let (x, y, z) = (1, 2, 3);`


函数参数也是模式：
```rust
fn foo(x: i32) {
    // 代码
}
```

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```
&(3, 5) 会匹配模式 &(x, y)，因此 x 得到了 3，y 得到了 5。



下面的代码编译器会报错：
`let Some(x) = some_option_value;`
因为右边的值可能不为 Some，而是 None，这种时候就不能进行匹配

对于 if let，就可以这样使用：
```rust
if let Some(x) = some_option_value {
    println!("{}", x);
}
```



#### 全模式列表

本节的目标就是把这些模式语法都罗列出来，方便大家检索查阅

##### 匹配字面值
```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

##### 匹配命名变量
```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {:?}", y),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {:?}", x, y);
}
```

##### 单分支多模式
```rust
let x = 1;

match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

##### match { 1..=5 => }
```rust
let x = 5;

match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
}
```

```rust
let x = 'c';

match x {
    'a'..='j' => println!("early ASCII letter"),
    'k'..='z' => println!("late ASCII letter"),
    _ => println!("something else"),
}
```
------------

##### 解构结构体
```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
}

    let p = Point { x: 0, y: 7 };
    let Point { x, y } = p;     // 同名不需要 :x
```

```rust
fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
}
```
--------
#####  解构枚举
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.")
        }
        Message::Move { x, y } => {
            println!(
                "Move in the x direction {} and in the y direction {}",
                x,
                y
            );
        }
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
    }
}
```

##### 解构嵌套的结构体和枚举
```rust
enum Color {
   Rgb(i32, i32, i32),
   Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!(
                "Change the color to hue {}, saturation {}, and value {}",
                h,
                s,
                v
            )
        }
        _ => ()
    }
}
```

##### 解构结构体和元组
```rust
struct Point {
     x: i32,
     y: i32,
 }

let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });
```

##### 解构数组
```rust
let arr: [u16; 2] = [114, 514];
let [x, y] = arr;

assert_eq!(x, 114);
assert_eq!(y, 514);
```

##### _忽略整个值
```rust
fn foo(_: i32, y: i32) {            // _
    println!("This code only uses the y parameter: {}", y);
}

fn main() {
    foo(3, 4);
}
```

##### 使用嵌套的 _ 忽略部分值
```rust
let mut setting_value = Some(5);
let new_setting_value = Some(10);

match (setting_value, new_setting_value) {
    (Some(_), Some(_)) => {
        println!("Can't overwrite an existing customized value");
    }
    _ => {
        setting_value = new_setting_value;
    }
}

println!("setting is {:?}", setting_value);
```
这段代码会打印出 Can't overwrite an existing customized value 接着是 setting is Some(5)。

```rust
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, _, third, _, fifth) => {
        println!("Some numbers: {}, {}, {}", first, third, fifth)
    },
}
```

##### 使用下划线开头忽略未使用的变量
```rust
fn main() {
    let _x = 5;
    let y = 10;
}
```

只使用 _ 和使用以下划线开头的名称有些微妙的不同：
比如 _x 仍会将值绑定到变量，而 _ 则完全不会绑定。

```rust
let s = Some(String::from("Hello!"));

if let Some(_s) = s {
    println!("found a string");
}

println!("{:?}", s);    // 错误，字符串的所有权已经给 _s 了。

// --------------

let s = Some(String::from("Hello!"));

if let Some(_) = s {
    println!("found a string");
}

println!("{:?}", s);        // ok, _ 完全不会绑定，所以 所有权没有移走。
```

##### 用 .. 忽略剩余所有值
```rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}
```

省略中间
```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {}, {}", first, last);
        },
    }
}
```


##### 匹配守卫提供的额外条件
匹配守卫（match guard）是一个位于 match 分支模式之后的额外 if 条件，它能为分支模式提供更进一步的匹配条件。
```rust
let num = Some(4);

match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
```

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {}", n),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {}", x, y);
}
```

匹配守卫 if y 作用于 4、5 和 6 ，
在满足 x 属于 4 | 5 | 6 后才会判断 y 是否为 true：
```rust
let x = 4;
let y = false;

match x {
    4 | 5 | 6 if y => println!("yes"),
    _ => println!("no"),
}
```


##### @绑定
@（读作 at）运算符允许为一个字段绑定另外一个变量。
下面例子中，我们希望测试 Message::Hello 的 id 字段是否位于 3..=7 范围内，同时也希望能将其值绑定到 id_variable 变量中以便此分支中相关的代码可以使用它。
我们可以将 id_variable 命名为 id，与字段同名，不过出于示例的目的这里选择了不同的名称。
```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3..=7 } => {
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
}
```


##### @前绑定后解构(Rust 1.56 新增)
```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    // 绑定新变量 `p`，同时对 `Point` 进行解构
    let p @ Point {x: px, y: py } = Point {x: 10, y: 23};
    println!("x: {}, y: {}", px, py);
    println!("{:?}", p);


    let point = Point {x: 10, y: 5};
    if let p @ Point {x: 10, y} = point {
        println!("x is 10 and y is {} in {:?}", y, p);
    } else {
        println!("x was not 10 :(");
    }
}
```



##### @新特性(Rust 1.53 新增)
```rust
fn main() {
    match 1 {
        num @ 1 | 2 => {
            println!("{}", num);
        }
        _ => {}
    }
}
```



### 方法method

https://course.rs/basic/method.html

Rust 使用 impl 来定义方法

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    // new是Circle的关联函数，因为它的第一个参数不是self，且new并不是关键字
    // 这种方法往往用于初始化当前结构体的实例
    fn new(x: f64, y: f64, radius: f64) -> Circle {
        Circle {
            x: x,
            y: y,
            radius: radius,
        }
    }

    // Circle的方法，&self表示借用当前的Circle结构体
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}
```

定义了一个 Rectangle 结构体，并且在其上定义了一个 area 方法，用于计算该矩形的面积。
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

#### self、&self 和 &mut self

在 area 的签名中，我们使用 `&self` 替代 `rectangle: &Rectangle`，`&self` 其实是 `self: &Self` 的简写（注意大小写）。
在一个 impl 块内，`Self` 指代被实现方法的结构体==类型==，`self` 指代此类型的==实例==，
换句话说，self 指代的是 Rectangle 结构体实例，这样的写法会让我们的代码简洁很多，
而且非常便于理解：我们为哪个结构体实现方法，那么 self 就是指代哪个结构体的实例。


self 依然有==所有权==的概念：
- self 表示 Rectangle 的所有权转移到该方法中，这种形式==用的较少==
- &self 表示该方法对 Rectangle 的不可变借用
- &mut self 表示可变借用

通过使用 self 作为第一个参数来使方法获取实例的所有权是很少见的，
这种使用方式往往用于把当前的对象==转成==另外一个对象时使用，
转换完后，就不再关注之前的对象，且可以防止对之前对象的误调用。


使用方法代替函数有以下好处：
- 不用在函数签名中重复书写 self 对应的类型
- 代码的组织性和内聚性更强，对于代码维护和阅读来说，好处巨大



方法名跟结构体字段名相同
在 Rust 中，允许方法名跟结构体的字段名相同：
```rust
impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    if rect1.width() {
        println!("The rectangle has a nonzero width; it is {}", rect1.width);
    }
}
```
使用 rect1.width() 时，Rust 知道我们调用的是它的方法，
使用 rect1.width，则是访问它的字段。


一般来说，方法跟字段同名，往往适用于实现 getter 访问器
```rust
pub struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    pub fn new(width: u32, height: u32) -> Self {
        Rectangle { width, height }
    }
    pub fn width(&self) -> u32 {
        return self.width;
    }
}

fn main() {
    let rect1 = Rectangle::new(30, 50);

    println!("{}", rect1.width());
}
```
用这种方式，我们可以把 Rectangle 的==字段设置为私有==属性，
只需把它的 new 和 width 方法设置为公开可见，那么用户就可以创建一个矩形，
同时通过访问器 rect1.width() 方法来获取矩形的宽度，因为 width 字段是私有的，
当用户访问 rect1.width 字段时，就会报错。
注意在此例中，Self 指代的就是被实现方法的结构体 Rectangle。

。。怎么设置 为私有。 和之前的对比，发现是 struct 前面多了一个 pub 。
。。难道 不写pub，默认全部 pub。  写了pub，就是属性默认 私有？
。。有人回： "默认所有字段都是私有的，只对当前文件公开；"


Rust 有一个叫 ==自动引用和解引用==的功能。
方法调用是 Rust 中少数几个拥有这种行为的地方。

当使用 object.something() 调用方法时，Rust 会==自动==为 object 添加 ==&、&mut 或 *== 以便使 object ==与方法签名匹配==。
也就是说，这些代码是等价的：
```text
p1.distance(&p2);
(&p1).distance(&p2);
```

这种自动引用的行为之所以有效，是因为方法有一个明确的接收者———— self 的类型。
在给出接收者和方法名的前提下，Rust 可以明确地计算出方法是仅仅读取（&self），做出修改（&mut self）或者是获取所有权（self）。
事实上，Rust 对方法接收者的隐式借用让所有权在实践中更友好。


带有多个参数的方法
```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    let rect2 = Rectangle { width: 10, height: 40 };
    let rect3 = Rectangle { width: 60, height: 45 };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

#### 关联函数
如何为一个结构体定义一个构造器方法？也就是接受几个参数，然后构造并返回该结构体的实例。
很简单，参数中不包含 self 即可。

这种定义在 impl 中且没有 self 的函数被称之为关联函数： 
因为它没有 self，不能用 f.read() 的形式调用，因此它是一个==函数而不是方法==，
它又在 impl 中，与结构体紧密关联，因此称为关联函数。

我们已经多次使用过关联函数，例如 String::from，用于创建一个动态字符串。

Rust 中有一个约定俗成的规则，使用 ==new 来作为构造器的名称==，出于设计上的考虑，Rust 特地没有用 new 作为关键字。

因为是函数，所以不能用 . 的方式来调用，我们需要用 :: 来调用，例如 let sq = Rectangle::new(3, 3);

-----------
多个 impl 定义

Rust 允许我们为一个结构体定义多个 impl 块，
目的是提供更多的灵活性和代码组织性，
例如当方法多了后，可以把相关的方法组织在同一个 impl 块中，那么就可以形成多个 impl 块，各自完成一块儿目标：

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

--------------
为枚举实现方法

枚举类型之所以强大，不仅仅在于它好用、可以同一化类型，还在于，我们可以像结构体一样，为枚举实现方法：

```rust
#![allow(unused)]
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        // 在这里定义方法体
    }
}

fn main() {
    let m = Message::Write(String::from("hello"));
    m.call();
}
```



### 泛型和特征，generics and trait


```rust
fn add<T>(a:T, b:T) -> T {
// error
// cannot add `T` to `T` // 无法将 `T` 类型跟 `T` 类型进行相加
// 不是所有类型都实现 + 操作
    a + b
}

fn main() {
    println!("add i8: {}", add(2i8, 3i8));
    println!("add i32: {}", add(20, 30));
    println!("add f64: {}", add(1.23, 1.23));
}
```


`fn largest<T>(list: &[T]) -> T {`

```rust
fn add<T: std::ops::Add<Output = T>>(a:T, b:T) -> T {
    a + b
}
```

`fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> T {`
这样就可以使用 > 进行比较。


---------------

结构体中使用泛型

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

```rust
struct Point<T,U> {
    x: T,
    y: U,
}
fn main() {
    let p = Point{x: 1, y :1.1};
}
```


------------
枚举中使用泛型

```rust
enum Option<T> {
    Some(T),
    None,
}
```

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```


#### 方法中使用泛型

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    // ------ 上面 T U 下面 V W，是可以的。
    // TU 是结构体Point 的泛型参数
    // VW 是方法 mixup 的泛型参数。
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```


##### 为具体的泛型类型实现方法
```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

这段代码意味着 `Point<f32>` 类型会有一个方法 distance_from_origin，
而其他 T 不是 f32 类型的 `Point<T>` 实例则没有定义此方法。

这样我们就能针对特定的泛型类型实现某个特定的方法，对于其它泛型类型则没有定义该方法。


----------------------
const 泛型（Rust 1.51 版本引入的重要特性）

在之前的泛型中，可以抽象为一句话：==针对类型实现的泛型==，所有的泛型都是为了抽象不同的类型，那有没有==针对值的泛型==？

`[i32; 2]` 和 `[i32; 3]` 是不同的数组类型

```rust
fn display_array(arr: [i32; 3]) {
    println!("{:?}", arr);
}
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(arr);

    let arr: [i32; 2] = [1,2];
    display_array(arr);     // error[E0308]: mismatched types
}
```

让我们修改代码，让 display_array 能打印任意长度的 i32 数组：
```rust
fn display_array(arr: &[i32]) {
    println!("{:?}", arr);
}
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(&arr);

    let arr: [i32; 2] = [1,2];
    display_array(&arr);
}
```

将 i32 改成所有类型的数组：
```rust
fn display_array<T: std::fmt::Debug>(arr: &[T]) {
    println!("{:?}", arr);
}
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(&arr);

    let arr: [i32; 2] = [1,2];
    display_array(&arr);
}
```
也不难，唯一要注意的是需要对 T 加一个限制 `std::fmt::Debug`，该限制表明 T 可以用在 println!("{:?}", arr) 中，因为 {:?} 形式的格式化输出需要 arr 实现该特征。


通过引用，我们可以很轻松的解决处理任何类型数组的问题，但是如果在某些场景下引用不适宜用或者干脆不能用呢

现在咱们有了 const 泛型，也就是针对值的泛型，正好可以用于处理数组长度的问题：
```rust
fn display_array<T: std::fmt::Debug, const    N    : usize>(arr: [T; N]) {
    println!("{:?}", arr);
}
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(arr);

    let arr: [i32; 2] = [1, 2];
    display_array(arr);
}
```

定义了一个类型为 [T; N] 的数组，其中 T 是一个基于类型的泛型参数
重点在于 N 这个泛型参数，它是一个基于值的泛型参数！因为它用来替代的是数组的长度。

N 就是 const 泛型，定义的语法是 `const N: usize`，表示 const 泛型 N ，它基于的值类型是 usize。



const 泛型表达式

假设我们某段代码需要在内存很小的平台上工作，因此需要限制函数参数占用的内存大小，此时就可以使用 const 泛型表达式来实现：

```rust
// 目前只能在nightly版本下使用
#![allow(incomplete_features)]
#![feature(generic_const_exprs)]

fn something<T>(val: T)
where
    Assert<{ core::mem::size_of::<T>() < 768 }>: IsTrue,
    //       ^-----------------------------^ 这里是一个 const 表达式，换成其它的 const 表达式也可以
{
    //
}

fn main() {
    something([0u8; 0]); // ok
    something([0u8; 512]); // ok
    something([0u8; 1024]); // 编译错误，数组长度是1024字节，超过了768字节的参数长度限制
}

// ---

pub enum Assert<const CHECK: bool> {
    //
}

pub trait IsTrue {
    //
}

impl IsTrue for Assert<true> {
    //
}
```

##### const fn
@todo


泛型的性能
在 Rust 中泛型是零成本的抽象，意味着你在使用泛型时，完全不用担心性能上的问题。

Rust 是在编译期为泛型对应的多个类型，生成各自的代码，因此损失了编译速度和增大了最终生成文件的大小。

Rust 通过在编译时进行泛型代码的 单态化(monomorphization)来保证效率。
单态化是一个通过填充编译时使用的具体类型，将通用代码转换为特定代码的过程。


```rust
let integer = Some(5);
let float = Some(5.0);
```
编译器生成的单态化版本的代码看起来像这样：
```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```



#### traits

https://course.rs/basic/trait/trait.html

`#[derive(Debug)]`，它在我们定义的类型(struct)上自动派生 Debug 特征，接着可以使用 println!("{:?}", x) 打印这个类型

特征定义了一组可以被共享的行为，只要实现了特征，你就能使用这组行为。


##### 定义特征

我们现在有文章 Post 和微博 Weibo 两种内容载体，而我们想对相应的内容进行总结，也就是无论是文章内容，还是微博内容，都可以在某个时间点进行总结，那么总结这个行为就是共享的，因此可以用特征来定义：

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```
我们只定义特征方法的签名，而不进行实现，此时方法签名结尾是 ;，而不是一个 {}。


##### 为类型实现特征

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

// ---------

pub struct Post {
    pub title: String, // 标题
    pub author: String, // 作者
    pub content: String, // 内容
}

impl Summary for Post {
    fn summarize(&self) -> String {
        format!("文章{}, 作者是{}", self.title, self.author)
    }
}

// ---------

pub struct Weibo {
    pub username: String,
    pub content: String
}

impl Summary for Weibo {
    fn summarize(&self) -> String {
        format!("{}发表了微博{}", self.username, self.content)
    }
}
```

```rust
fn main() {
    let post = Post{title: "Rust语言简介".to_string(),author: "Sunface".to_string(), content: "Rust棒极了!".to_string()};
    let weibo = Weibo{username: "sunface".to_string(),content: "好像微博没Tweet好用".to_string()};

    println!("{}",post.summarize());
    println!("{}",weibo.summarize());
}
```


-------------
##### 特征定义与实现的位置(孤儿规则)

`如果你想要为类型 A 实现特征 T，那么 A 或者 T 至少有一个是在当前作用域中定义的！`

我们可以为上面的 ==Post 类型实现标准库中的 Display== 特征，这是因为 Post 类型定义在当前的作用域中。
同时，我们也可以在当前包中==为 String 类型实现 Summary 特征==，因为 Summary 定义在当前作用域中。

但是你==无法==在当前作用域中，为 String 类型实现 Display 特征，因为它们俩都定义在标准库中



##### 默认实现

```rust
pub trait Summary {     // trait 中实现方法。 可以直接使用，可以被重载
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```
为了使用 Summary，只需要实现 summarize_author 方法即可：
```rust
impl Summary for Weibo {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
println!("1 new weibo: {}", weibo.summarize());
```



##### 使用特征作为函数参数
真正可以让特征大放光彩的地方。

先定义一个函数，使用 实现了Summary特征 的 item 参数。
```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```
。。这个是语法糖，下面是底层。

###### 特征约束(trait bound)

虽然 impl Trait 这种语法非常好理解，但是实际上它只是一个语法糖：
```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```
真正的完整书写形式如上所述，形如 `T: Summary` 被称为特征约束。


impl Trait 这种语法糖就足够使用，但是复杂场景，特征约束有更大的灵活性和语法表现能力。
```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {}

pub fn notify<T: Summary>(item1: &T, item2: &T) {}
```

###### 多重约束

```rust

// 。。括号 包含了 impl
pub fn notify(item: &(impl Summary + Display)) {}

pub fn notify<T: Summary + Display>(item: &T) {}
```


###### where约束

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {}

fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```

###### 使用特征约束有条件地实现方法或特征

特征约束，可以让我们在指定类型 + 指定特征的条件下去实现方法

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

cmp_display 方法，并不是所有的 `Pair<T>` 结构体对象都可以拥有，
只有 T 同时实现了 Display + PartialOrd 的 `Pair<T>` 才可以拥有此方法。 
该函数可读性会更好，因为泛型参数、参数、返回值都在一起，可以快速的阅读

###### impl <T: already> new for T
也可以`有条件地实现特征`, 例如，标准库为任何实现了 Display 特征的类型实现了 ToString 特征：
```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

我们可以对任何实现了 Display 特征的类型调用由 ToString 定义的 to_string 方法。
例如，可以将整型转换为对应的 String 值，因为整型实现了 Display：
`let s = 3.to_string();`



###### 函数返回中的 impl Trait

```rust
fn returns_summarizable() -> impl Summary {
    Weibo {
        username: String::from("sunface"),
        content: String::from(
            "m1 max太厉害了，电脑再也不会卡",
        )
    }
}
```

例如，闭包和迭代器就是很复杂，只有编译器才知道那玩意的真实类型，如果让你写出来它们的具体类型，估计内心有一万只草泥马奔腾，
好在你可以用 impl Iterator 来告诉调用者，返回了一个迭代器，因为所有迭代器都会实现 Iterator 特征。

一个很大的限制：只能有一个具体的类型
。。就是 多个return 的值 必须是 相同类型。 不能是 实现了 trait 的不同类型。

如果想要实现返回不同的类型，需要使用下一章节中的特征对象。


-----------
修复之前的 largest

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}
```

##### 通过 derive 派生特征

在本书中，形如 `#[derive(Debug)]` 的代码已经出现了很多次，这种是一种特征派生语法，被 derive 标记的对象会自动实现对应的默认特征代码，继承相应的功能。

例如 Debug 特征，它有一套自动实现的默认代码，当你给一个结构体标记后，就可以使用 `println!("{:?}", s)` 的形式打印该结构体的对象。

详细的 derive 列表参见附录-派生特征。
https://course.rs/appendix/derive.html



##### 调用方法需要引入特征

在一些场景中，使用 as 关键字做类型转换会有比较大的限制，
因为你想要在类型转换上拥有`完全的控制`，
例如处理转换错误，那么你将需要 `TryInto`

```rust
use std::convert::TryInto;

fn main() {
  let a: i32 = 10;
  let b: u16 = 100;

  let b_ = b.try_into()
            .unwrap();

  if a < b_ {
    println!("Ten is less than one hundred.");
  }
}
```
上面代码中引入了 `std::convert::TryInto` 特征，但是却没有使用它，可能有些同学会为此困惑，
主要原因在于==如果你要使用一个特征的方法，那么你需要将该特征引入当前的作用域中==，我们在上面用到了 ==try_into 方法，因此需要引入对应的特征==。

但是 Rust 又提供了一个非常便利的办法，即把最常用的标准库中的特征通过 std::prelude 模块提前引入到当前作用域中，其中包括了 std::convert::TryInto，
你可以尝试删除第一行的代码 use ...，看看是否会报错。



##### 例子

###### 为自定义类型实现 + 操作

```rust
use std::ops::Add;

// 为Point结构体派生Debug特征，用于格式化输出
#[derive(Debug)]
struct Point<T: Add<T, Output = T>> { //限制类型T必须实现了Add特征，否则无法进行+操作。
    x: T,
    y: T,
}

impl<T: Add<T, Output = T>> Add for Point<T> {
    type Output = Point<T>;

    fn add(self, p: Point<T>) -> Point<T> {
        Point{
            x: self.x + p.x,
            y: self.y + p.y,
        }
    }
}

fn add<T: Add<T, Output=T>>(a:T, b:T) -> T {
    a + b
}

fn main() {
    let p1 = Point{x: 1.1f32, y: 1.1f32};
    let p2 = Point{x: 2.1f32, y: 2.1f32};
    println!("{:?}", add(p1, p2));

    let p3 = Point{x: 1i32, y: 1i32};
    let p4 = Point{x: 2i32, y: 2i32};
    println!("{:?}", add(p3, p4));
}
```


###### 自定义类型的打印输出

往往只要使用 `#[derive(Debug)]` 对我们的自定义类型进行标注，即可实现打印输出的功能

```rust
#[derive(Debug)]
struct Point{
    x: i32,
    y: i32
}
fn main() {
    let p = Point{x:3,y:3};
    println!("{:?}",p);
}
```

需要对我们的自定义类型进行自定义的格式化输出
为自定义类型实现 std::fmt::Display 特征

```rust
#![allow(dead_code)]

use std::fmt;
use std::fmt::{Display};

#[derive(Debug,PartialEq)]
enum FileState {
  Open,
  Closed,
}

#[derive(Debug)]
struct File {
  name: String,
  data: Vec<u8>,
  state: FileState,
}

impl Display for FileState {
   fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
     match *self {
         FileState::Open => write!(f, "OPEN"),
         FileState::Closed => write!(f, "CLOSED"),
     }
   }
}

impl Display for File {
   fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
      write!(f, "<{} ({})>",
             self.name, self.state)
   }
}

impl File {
  fn new(name: &str) -> File {
    File {
        name: String::from(name),
        data: Vec::new(),
        state: FileState::Closed,
    }
  }
}

fn main() {
  let f6 = File::new("f6.txt");
  //...
  println!("{:?}", f6);
  println!("{}", f6);
}
```

#### 特征对象

在上一节中有一段代码无法通过编译:

```rust
// 就是2个不同的类型，都 imple Summary，但是编译会失败。
// impl Trait 的返回值类型并不支持多种不同的类型返回
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        Post {
           // ...
        }
    } else {
        Weibo {
            // ...
        }
    }
}
```

一种是通过枚举
但，只能用于完全由自己编写的代码中。
```rust
#[derive(Debug)]
enum UiObject {
    Button,
    SelectBox,
}

fn main() {
    let objects = [
        UiObject::Button,
        UiObject::SelectBox
    ];

    for o in objects {
        draw(o)
    }
}

fn draw(o: UiObject) {
    println!("{:?}",o);
}
```

在拥有继承的语言中，可以定义一个名为 Component 的类，该类上有一个 draw 方法。
其他的类比如 Button、Image 和 SelectBox 会从 Component 派生并因此继承 draw 方法。
它们各自都可以覆盖 draw 方法来定义自己的行为，但是框架会把所有这些类型当作是 Component 的实例，并在其上调用 draw。
不过 Rust 并没有继承，我们得另寻出路。


##### 特征对象定义

在介绍特征对象之前，先来为之前的 UI 组件定义一个特征：
```rust
pub trait Draw {
    fn draw(&self);
}
```
只要组件实现了 Draw 特征，就可以调用 draw 方法来进行渲染。
假设有一个 Button 和 SelectBox 组件实现了 Draw 特征

```rust
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // 绘制按钮的代码
    }
}

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // 绘制SelectBox的代码
    }
}
```

需要一个动态数组来存储这些 UI 对象：
```rust
pub struct Screen {
    pub components: Vec<xxx>,
}
```
xxx 应该是什么类型？
因为 Button 和 SelectBox 都实现了 Draw 特征，那我们是不是可以把 Draw 特征的对象作为类型，填入到数组中呢？答案是肯定的。

特征对象指向实现了 Draw 特征的类型的实例，也就是指向了 Button 或者 SelectBox 的实例，这种映射关系是存储在一张表中，可以在==运行时==通过特征对象找到具体调用的类型方法。

。。对象是指 有指定 type 的 某个对象。  特征对象 是指 实现了 指定 trait 的 一些对象

可以通过 `& 引用`或者 `Box<T>` 智能指针的方式来==创建特征对象==。

`Box<T>` 在后面章节会详细讲解，大家现在把它当成一个引用即可，只不过它包裹的值会被强制分配在堆上。

```rust
trait Draw {
    fn draw(&self) -> String;
}

impl Draw for u8 {
    fn draw(&self) -> String {
        format!("u8: {}", *self)
    }
}

impl Draw for f64 {
    fn draw(&self) -> String {
        format!("f64: {}", *self)
    }
}

// 若 T 实现了 Draw 特征， 则调用该函数时传入的 Box<T> 可以被隐式转换成函数参数签名中的 Box<dyn Draw>
fn draw1(x: Box<dyn Draw>) {
    // 由于实现了 Deref 特征，Box 智能指针会自动解引用为它所包裹的值，然后调用该值对应的类型上定义的 `draw` 方法
    x.draw();
}

fn draw2(x: &dyn Draw) {
    x.draw();
}

fn main() {
    let x = 1.1f64;
    // do_something(&x);
    let y = 8u8;

    // x 和 y 的类型 T 都实现了 `Draw` 特征，因为 Box<T> 可以在函数调用时隐式地被转换为特征对象 Box<dyn Draw> 
    // 基于 x 的值创建一个 Box<f64> 类型的智能指针，指针指向的数据被放置在了堆上
    draw1(Box::new(x));
    // 基于 y 的值创建一个 Box<u8> 类型的智能指针
    draw1(Box::new(y));
    draw2(&x);
    draw2(&y);
}
```

- draw1 函数的参数是 `Box<dyn Draw>` 形式的特征对象，该特征对象是通过 Box::new(x) 的方式创建的
- draw2 函数的参数是 &dyn Draw 形式的特征对象，该特征对象是通过 &x 的方式创建的
- dyn 关键字只用在特征对象的类型声明上，在创建时无需使用 dyn

---------

可以使用特征对象来代表泛型或具体的类型。

-----------

继续来完善之前的 UI 组件代码，首先来实现 Screen：
```rust
pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}
```
其中存储了一个动态数组，里面元素的类型是 Draw 特征对象：`Box<dyn Draw>`，任何实现了 Draw 特征的类型，都可以存放其中。

再来为 Screen 定义 run 方法，用于将列表中的 UI 组件渲染在屏幕上：
```rust
impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```
至此，我们就完成了之前的目标：
在列表中存储多种不同类型的实例，然后将它们使用同一个方法逐一渲染在屏幕上！

-------------

如果通过泛型实现
```rust
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

impl<T> Screen<T>
    where T: Draw {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```
这种写法限制了 Screen 实例的 `Vec<T>` 中的每个元素必须是 Button 类型或者全是 SelectBox 类型。
如果只需要同质（相同类型）集合，更倾向于采用泛型+特征约束这种写法，因其实现更清晰，且性能更好(特征对象，需要在运行时从 vtable 动态查找需要调用的方法)。


运行
```rust
fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No")
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```

在动态类型语言中，有一个很重要的概念：鸭子类型(duck typing)，简单来说，就是只关心值长啥样，而不关心它实际是什么。
当一个东西走起来像鸭子，叫起来像鸭子，那么它就是一只鸭子，就算它实际上是一个奥特曼，也不重要，我们就当它是鸭子。

在上例中，Screen 在 run 的时候，我们并不需要知道各个组件的具体类型是什么。
它也不检查组件到底是 Button 还是 SelectBox 的实例，只要它实现了 Draw 特征，就能通过 Box::new 包装成 `Box<dyn Draw>` 特征对象，然后被渲染在屏幕上。

使用特征对象和 Rust 类型系统来进行类似鸭子类型操作的优势是，无需在运行时检查一个值是否实现了特定方法或者担心在调用时因为值没有实现方法而产生错误。
如果值没有实现特征对象所需的特征， 那么 Rust 根本就不会编译这些代码：
。。会编译失败。


注意 dyn 不能单独作为特征对象的定义，例如下面的代码编译器会==报错==，原因是特征对象可以是任意实现了某个特征的类型，编译器在编译期不知道该类型的大小，不同的类型大小是不同的。

而 &dyn 和 `Box<dyn>` 在==编译期都是已知大小==，所以可以用作特征对象的定义。
```rust
fn draw2(x: dyn Draw) {
    x.draw();
}
```
```text
10 | fn draw2(x: dyn Draw) {
   |          ^ doesn't have a size known at compile-time
   |
   = help: the trait `Sized` is not implemented for `(dyn Draw + 'static)`
help: function arguments must have a statically known size, borrowed types always have a known size
```


##### 特征对象的动态分发

==泛型==是在==编译期完成==处理的：编译器会为每一个泛型参数对应的具体类型生成一份代码，这种方式是==静态分发==(static dispatch)，因为是在编译期完成的，对于运行期性能完全没有任何影响。

与静态分发相对应的是==动态分发==(dynamic dispatch)，在这种情况下，直到运行时，才能确定需要调用什么方法。之前代码中的关键字 ==dyn== 正是在强调这一“==动态==”的特点

当使用特征对象时，Rust 必须使用动态分发。
编译器无法知晓所有可能用于特征对象代码的类型，所以它也不知道应该调用哪个类型的哪个方法实现。
为此，Rust 在运行时使用特征对象中的指针来知晓需要调用哪个方法。
动态分发也阻止编译器有选择的内联方法代码，这会相应的禁用一些优化。

下面这张图很好的解释了静态分发 `Box<T>` 和动态分发 `Box<dyn Trait>` 的区别：

![a2f0396a8a17bf0c1240761b4adfaa7b.png](resources/6b218f8925304299a40585818f1279d4.png)

结合上文的内容和这张图可以了解：
- 特征对象大小不固定
- 几乎总是使用特征对象的引用方式，如 `&dyn Draw`、`Box<dyn Draw>`

特征对象没有固定大小，但是它的 ref类型的大小是固定的，由 2个指针组成(ptr, vptr)，因此占用 2个指针大小。
- ptr，指向 实现了特征 Draw 的具体类型的 实例。比如 Button实例。
- vptr，指向一个 虚表 vtable，vtable 中保存了 类型 Button 或类型 SelectBox 的实例 对于 可以调用的实现了特征 Draw 的方法。
  当调用方法时，直接从vtable中找到方法并调用。
  btn 是哪个特征对象的实例，它的 vtable 中就包含了该==特征的==方法。


##### Self 与 self

一个指代当前的实例对象，一个指代特征或者方法类型的别名

```rust
trait Draw {
    fn draw(&self) -> Self;
}

#[derive(Clone)]
struct Button;
impl Draw for Button {
    fn draw(&self) -> Self {
        return self.clone()
    }
}

fn main() {
    let button = Button;
    let newb = button.draw();
}
```
self指代的就是当前的实例对象，也就是 button.draw() 中的 button 实例，Self 则指代的是 Button 类型。


当理解了 self 与 Self 的区别后，我们再来看看何为对象安全。

##### 特征对象的限制

不是所有特征都能拥有特征对象，只有`对象安全的特征`才行。
当一个特征的`所有方法`都有如下属性时，它的`对象才是安全`的：
- 方法的返回类型不能是 Self
- 方法没有任何泛型参数

对象安全对于特征对象是必须的，因为一旦有了特征对象，就不再需要知道实现该特征的具体类型是什么了。
如果==特征==方法返回了具体的 Self 类型，但是特征对象==忘记了其真正的类型==，那这个 ==Self== 就非常尴尬，因为==没人知道它是谁==了。
但是对于==泛型==类型参数来说，当使用特征时其会==放入具体的类型==参数：此==具体类型变成了实现该特征的类型的一部分==。
而当使用==特征对象==时其==具体类型被抹去==了，故而无从得知放入泛型参数类型到底是什么。

标准库中的 Clone 特征就==不符合==对象安全的要求：
```rust
pub trait Clone {
    fn clone(&self) -> Self;
}
```


```rust
pub struct Screen {
    pub components: Vec<Box<dyn Clone>>,
}
```
会得到下面的错误
```text
error[E0038]: the trait `std::clone::Clone` cannot be made into an object
 --> src/lib.rs:2:5
  |
2 |     pub components: Vec<Box<dyn Clone>>,
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the trait `std::clone::Clone`
  cannot be made into an object
  |
  = note: the trait cannot require that `Self : Sized`
```



#### 深入了解特征

特征之于 Rust 更甚于接口之于其他语言


##### 关联类型

关联类型和关联函数并没有任何交集

关联类型是在 `特征定义`的语句块中，`申明`一个`自定义类型`，这样就可以在特征的方法签名中使用该类型：

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```
以上是标准库中的迭代器特征 Iterator，它有一个 Item 关联类型，用于替代遍历的值的类型。

next 方法也返回了一个 Item 类型，不过使用 Option 枚举进行了包裹，假如迭代器中的值是 i32 类型，那么调用 next 方法就将获取一个 `Option<i32>` 的值。

Self 用来指代当前调用者的具体类型，那么 Self::Item 就用来指代该类型实现中定义的 Item 类型

```rust
impl Iterator for Counter {
    type Item = u32;            // ... 6

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
    }
}

fn main() {
    let c = Counter{..}
    c.next()
}
```


为何不用泛型，例如如下代码：
```rust
pub trait Iterator<Item> {
    fn next(&mut self) -> Option<Item>;
}
```

很简单，为了代码的可读性，当你使用了泛型后，你需要在所有地方都写 `Iterator<Item>`，而使用了关联类型，你只需要写 Iterator，当类型定义复杂时，这种写法可以极大的增加可读性：

```rust
pub trait CacheableItem: Clone + Default + fmt::Debug + Decodable + Encodable {
  type Address: AsRef<[u8]> + Clone + fmt::Debug + Eq + Hash;
  fn is_null(&self) -> bool;
}
```

Address 的写法自然远比 `AsRef<[u8]> + Clone + fmt::Debug + Eq + Hash` 要简单的多，而且含义清晰。

如果使用泛型，你将得到以下的代码：
```rust
trait Container<A,B> {
    fn contains(&self,a: A,b: B) -> bool;
}

fn difference<A,B,C>(container: &C) -> i32
  where
    C : Container<A,B> {...}
```

可以看到，由于使用了泛型，导致函数头部也必须增加泛型的声明，而使用关联类型，将得到可读性好得多的代码：
```rust
trait Container{
    type A;
    type B;
    fn contains(&self, a: &Self::A, b: &Self::B) -> bool;
}

fn difference<C: Container>(container: &C) {}
```

##### 默认泛型类型参数

当使用泛型类型参数时，可以为其指定一个`默认的具体类型`，例如标准库中的 `std::ops::Add` 特征：

```rust
trait Add<RHS=Self> {
    type Output;

    fn add(self, rhs: RHS) -> Self::Output;
}
```
这里它给 RHS 一个默认值，也就是当用户不指定 RHS 时，默认使用两个同样类型的值进行相加，然后返回一个关联类型 Output。


```rust
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {        // 没有指定 <xxx>
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
               Point { x: 3, y: 3 });
}
```
为 Point 结构体提供 + 的能力，这就是运算符重载，
不过 Rust 并不支持创建自定义运算符，你也无法为所有运算符进行重载，
目前来说，只有定义在 std::ops 中的运算符才能进行重载。

跟 + 对应的特征是 std::ops::Add，我们在之前也看过它的定义 `trait Add<RHS=Self>`，但是上面的例子中并没有为 Point 实现 `Add<RHS>` 特征，而是实现了 Add 特征（没有默认泛型类型参数），这意味着我们使用了 RHS 的默认类型，也就是 Self。
换句话说，我们这里定义的是两个相同的 Point 类型相加，因此无需指定 RHS。

下面的例子，我们来创建两个不同类型的相加：
```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {          // 指定了 <xxx>
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

默认类型参数主要用于两个方面：
- 减少实现的样板代码
- 扩展类型但是无需大幅修改现有的代码


##### 调用同名的方法

不同特征拥有同名的方法是很正常的事情，你没有任何办法阻止这一点；
甚至除了特征上的同名方法外，在你的类型上，也有同名方法：

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}
```

----------

优先调用类型上的方法

当调用 Human 实例的 fly 时，编译器默认调用该类型中定义的方法：
```rust
fn main() {
    let person = Human;
    person.fly();
}
```
打印 `*waving arms furiously*`, 说明直接调用了类型上定义的方法。

-----------

调用特征上的方法

为了能够调用两个特征的方法，需要使用显式调用的语法：
```rust
fn main() {
    let person = Human;
    Pilot::fly(&person); // 调用Pilot特征上的方法
    Wizard::fly(&person); // 调用Wizard特征上的方法
    person.fly(); // 调用Human类型自身的方法
}
```

-------------

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", Dog::baby_name());
}
```

完全限定语法

```rust
fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```
在尖括号中，通过 as 关键字，我们向 Rust 编译器提供了类型注解，也就是 Animal 就是 Dog，而不是其他动物，
因此最终会调用 impl Animal for Dog 中的方法，获取到其它动物对狗宝宝的称呼：puppy。


完全限定语法定义为：
`<Type as Trait>::function(receiver_if_method, next_arg, ...);`


##### 特征定义中的特征约束

有时，我们会需要让某个特征 A 能使用另一个特征 B 的功能(另一种形式的特征约束)，
这种情况下，不仅仅要为类型实现特征 A，还要为类型实现特征 B 才行，这就是 `supertrait`

例如有一个特征 OutlinePrint，它有一个方法，能够对当前的实现类型进行格式化输出：
```rust
use std::fmt::Display;

trait OutlinePrint: Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

```rust
use std::fmt;

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

##### 在外部类型上实现外部特征(newtype)

在特征章节中，有提到孤儿规则，简单来说，就是特征或者类型必需至少有一个是本地的，才能在此类型上定义特征。

这里提供一个办法来==绕过==孤儿规则，那就是使用`newtype 模式`，
简而言之：就是为一个元组结构体创建新类型。该元组结构体封装有一个字段，该字段就是希望实现特征的具体类型。

该封装类型是本地的，因此我们可以为此类型实现外部的特征。

newtype 不仅仅能实现以上的功能，而且它在运行时没有任何性能损耗，因为在编译期，该类型会被自动忽略。

一个例子，我们有一个动态数组类型： `Vec<T>`，它定义在标准库中，还有一个特征 Display，它也定义在标准库中，如果没有 newtype，我们是无法为 `Vec<T>` 实现 Display 的：

```rust
error[E0117]: only traits defined in the current crate can be implemented for arbitrary types
--> src/main.rs:5:1
|
5 | impl<T> std::fmt::Display for Vec<T> {
| ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^------
| |                             |
| |                             Vec is not defined in the current crate
| impl doesn't use only types from inside the current crate
|
= note: define and implement a trait or new type instead
```
注意报错的最后一句，提示了 trait 或 new type， 所以选择 new type

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```
`struct Wrapper(Vec<String>)` 就是一个元组结构体，它定义了一个新类型 Wrapper

既然 new type 有这么多好处，它有没有不好的地方呢？答案是肯定的。注意到我们怎么访问里面的数组吗？`self.0.join(", ")`，是的，很啰嗦，因为需要先从 Wrapper 中取出数组: self.0，然后才能执行 join 方法。

类似的，任何数组上的方法，你都无法直接调用，需要先用 self.0 取出数组，然后再进行调用。

Rust 提供了一个特征叫 Deref，实现该特征后，可以自动做一层类似类型转换的操作，可以将 Wrapper 变成 `Vec<String>` 来使用。这样就会像直接使用数组那样去使用 Wrapper，而无需为每一个操作都添加上 self.0。

同时，如果不想 Wrapper 暴露底层数组的所有方法，我们还可以为 Wrapper 去重载这些方法，实现隐藏的目的。



-----------------

### 集合类型

https://course.rs/basic/collections/intro.html


#### `Vec<T>`

`Vec<T>`

动态数组只能存储相同类型的元素

如果你想存储不同类型的元素，可以使用之前讲过的==枚举类型==或者==特征对象==。

> 创建

```rust
let v: Vec<i32> = Vec::new();

let mut v = Vec::new();
v.push(1);

let v = Vec::with_capacity(capacity);
```

```rust
let v = vec![1, 2, 3];

let v = vec![-1; 100];
```



读取指定位置的元素有两种方式可选：
- 通过下标索引访问。
- 使用 get 方法。返回 Option<&T>

越界的时候，索引会异常，get是 None。

能确保索引不越界，就用索引。
如果是外部输入，不可控，那么 get

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("第三个元素是 {}", third);

match v.get(2) {
    Some(third) => println!("第三个元素是 {third}"),
    None => println!("去你的第三个元素，根本没有！"),
}
```


```rust
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];      // 不可变借用

v.push(6);      // 可变借用

println!("The first element is: {first}");      // 使用 不可变借用。 error
```

数组的大小是可变的，当旧数组的大小不够用时，Rust 会重新分配一块更大的内存空间，然后把旧数组拷贝过来。
这种情况下，之前的引用显然会指向一块无效的内存



##### iter

```rust
let mut v = vec![1, 2, 3];
for i in &mut v {
    *i += 10
}
```

##### 存储不同类型的元素

> 枚举

```rust
#[derive(Debug)]
enum IpAddr {
    V4(String),
    V6(String)
}
fn main() {
    let v = vec![
        IpAddr::V4("127.0.0.1".to_string()),
        IpAddr::V6("::1".to_string())
    ];

    for ip in v {
        show_addr(ip)
    }
}

fn show_addr(ip: IpAddr) {
    println!("{:?}",ip);
}
```

> 特征对象

```rust
trait IpAddr {
    fn display(&self);
}

struct V4(String);
impl IpAddr for V4 {
    fn display(&self) {
        println!("ipv4: {:?}",self.0)
    }
}
struct V6(String);
impl IpAddr for V6 {
    fn display(&self) {
        println!("ipv6: {:?}",self.0)
    }
}

fn main() {
    let v: Vec<Box<dyn IpAddr>> = vec![
        Box::new(V4("127.0.0.1".to_string())),
        Box::new(V6("::1".to_string())),
    ];

    for ip in v {
        ip.display();
    }
}
```


排序

稳定的排序 sort 和 sort_by，、
非稳定排序 sort_unstable 和 sort_unstable_by。

总体而言，非稳定 排序的算法的速度会优于 稳定 排序算法，
同时，稳定 排序还会额外分配原数组一半的空间。

```rust
fn main() {
    let mut vec = vec![1, 5, 10, 2, 15];    
    vec.sort_unstable();    
    assert_eq!(vec, vec![1, 2, 5, 10, 15]);
}
```

在浮点数当中，存在一个 NAN 的值，这个值无法与其他的浮点数进行对比，
因此，浮点数类型并没有实现全数值可比较 Ord 的特性，而是实现了部分可比较的特性 PartialOrd。
。。浮点数，直接sort() 会编译异常。要用下面的。

如果我们确定在我们的浮点数数组当中，不包含 NAN 值，那么我们可以使用 partial_cmp 来作为大小判断的依据。
```rust
fn main() {
    let mut vec = vec![1.0, 5.6, 10.3, 2.0, 15f32];    
    vec.sort_unstable_by(|a, b| a.partial_cmp(b).unwrap());    
    assert_eq!(vec, vec![1.0, 2.0, 5.6, 10.3, 15f32]);
}
```


```rust
struct Person {
    name: String,
    age: u32,
}

people.sort_unstable_by(|a, b| b.age.cmp(&a.age));  // 按年龄倒序
```


实现 Ord 需要我们实现 Ord、Eq、PartialEq、PartialOrd 这些属性。
你可以 derive 这些属性

```rust
#[derive(Debug, Ord, Eq, PartialEq, PartialOrd)]
struct Person {
    name: String,
    age: u32,
}

impl Person {
    fn new(name: String, age: u32) -> Person {
        Person { name, age }
    }
}

fn main() {
    let mut people = vec![
        Person::new("Zoe".to_string(), 25),
        Person::new("Al".to_string(), 60),
        Person::new("Al".to_string(), 30),
        Person::new("John".to_string(), 1),
        Person::new("John".to_string(), 25),
    ];

    people.sort_unstable();

    println!("{:?}", people);
}
```

需要 derive Ord 相关特性，需要确保你的结构体中所有的属性均实现了 Ord 相关特性，否则会发生编译错误。
derive 的默认实现会依据==属性的顺序依次进行比较==，如上述例子中，当 Person 的 name 值相同，则会使用 age 进行比较。



#### HashMap

`HashMap<&str,i32>`
```rust
use std::collections::HashMap;

// 创建一个HashMap，用于存储宝石种类和对应的数量
let mut my_gems = HashMap::new();

// 将宝石类型和对应的数量写入表中
my_gems.insert("红宝石", 1);
```

所有的集合类型都是动态的，意味着它们没有固定的内存大小，因此它们底层的数据都存储在内存堆上，然后通过一个存储在栈中的引用类型来访问。
同时，跟其它集合类型一致，HashMap 也是内聚性的，即所有的 K 必须拥有同样的类型，V 也是如此。

如果预先知道要存储的 KV 对个数，可以使用 `HashMap::with_capacity(capacity)`


```rust
    let teams_list = vec![
        ("中国队".to_string(), 100),
        ("美国队".to_string(), 10),
        ("日本队".to_string(), 50),
    ];

    let mut teams_map = HashMap::new();
    for team in &teams_list {
        teams_map.insert(&team.0, team.1);
    }
```
Rust 为我们提供了一个非常精妙的解决办法：先将 Vec 转为迭代器，接着通过 collect 方法，将迭代器中的元素收集后，转成 HashMap：
```rust
    let teams_list = vec![
        ("中国队".to_string(), 100),
        ("美国队".to_string(), 10),
        ("日本队".to_string(), 50),
    ];

    let teams_map: HashMap<_,_> = teams_list.into_iter().collect();
```
collect 方法在内部实际上支持生成多种类型的目标集合，因此我们需要通过类型标注 HashMap<_,_> 来告诉编译器



> 所有权

- 若类型实现 Copy 特征，该类型会被复制进 HashMap，因此无所谓所有权
- 若没实现 Copy 特征，所有权将被转移给 HashMap 中


如果你使用引用类型放入 HashMap 中，请确保该引用的生命周期至少跟 HashMap 活得一样久：



> 查询

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score: Option<&i32> = scores.get(&team_name);
```

- get 方法返回一个 Option<&i32> 类型：当查询不到时，会返回一个 None，查询到时返回 Some(&i32)
- &i32 是对 HashMap 中值的借用，如果不使用借用，可能会发生所有权的转移


`let score: i32 = scores.get(&team_name).copied().unwrap_or(0);`


```rust
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```


##### 更新HashMap

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert("Blue", 10);

    // 覆盖已有的值
    let old = scores.insert("Blue", 20);
    assert_eq!(old, Some(10));

    // 查询新插入的值
    let new = scores.get("Blue");
    assert_eq!(new, Some(&20));

    // 查询Yellow对应的值，若不存在则插入新值
    let v = scores.entry("Yellow").or_insert(5);
    assert_eq!(*v, 5); // 不存在，插入5

    // 查询Yellow对应的值，若不存在则插入新值
    let v = scores.entry("Yellow").or_insert(50);
    assert_eq!(*v, 5); // 已经存在，因此50没有插入
}
```

##### upsert

有就更新，没有就插入
如：在文本中统计词语出现的次数
```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();
// 根据空格来切分字符串(英文单词都是通过空格切分)
for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
```

- or_insert 返回了 ==&mut v== 引用，因此可以通过该可变引用直接修改 map 中对应的值
- 使用 count 引用时，需要先进行==解引用 *count==，否则会出现类型不匹配



##### hash函数的替换

f32 和 f64 浮点数，没有实现 std::cmp::Eq 特征，因此不可以用作 HashMap 的 Key。

如何保证不同 Key 通过哈希后的两个值不会相同？如果相同，那意味着我们使用不同的 Key，却查到了同一个结果，这种明显是错误的行为。 此时，就涉及到==安全性跟性能的取舍==了。

若要追求安全，尽可能减少冲突，同时防止拒绝服务（Denial of Service, DoS）攻击，就要使用==密码学安全的哈希函数==，HashMap 就是使用了这样的哈希函数。反之若要追求性能，就需要使用没有那么安全的算法。


因此若性能测试显示当前标准库默认的哈希函数==不能满足你的性能需求==，就需要去 crates.io 上寻找其它的哈希函数实现，使用方法很简单：

```rust
use std::hash::BuildHasherDefault;
use std::collections::HashMap;
// 引入第三方的哈希函数
use twox_hash::XxHash64;

// 指定HashMap使用第三方的哈希函数XxHash64
let mut hash: HashMap<_, _, BuildHasherDefault<XxHash64>> = Default::default();
hash.insert(42, "the answer");
assert_eq!(hash.get(&42), Some(&"the answer"));
```

目前，HashMap 使用的哈希函数是 SipHash，它的性能不是很高，但是安全性很高。
SipHash 在中等大小的 Key 上，性能相当不错，但是对于小型的 Key （例如整数）或者大型 Key （例如字符串）来说，性能还是不够好。
若你需要极致性能，例如实现算法，可以考虑这个库：ahash。https://github.com/tkaitchuck/ahash




### 生命周期

生命周期，简而言之就是引用的有效作用域。

当多个生命周期存在，且编译器无法推导出某个引用的生命周期时，就需要我们手动标明生命周期

#### 悬垂指针与生命周期

生命周期的主要作用是避免悬垂引用，它会导致程序引用了本不该引用的数据

```rust
{
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
```
注意
- let r; 的声明方式貌似存在使用 null 的风险，实际上，当我们不初始化它就使用时，编译器会给予报错
- r 引用了内部花括号中的 x 变量，但是 x 会在内部花括号 } 处被释放，因此回到外部花括号后，r 会引用一个无效的 x



-------------

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
```text
error[E0106]: missing lifetime specifier
 --> src/main.rs:9:33
  |
9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter // 参数需要一个生命周期
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is
  borrowed from `x` or `y`
  = 帮助： 该函数的返回值是一个引用类型，但是函数签名无法说明，该引用是借用自 `x` 还是 `y`
help: consider introducing a named lifetime parameter // 考虑引入一个生命周期
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ^^^^    ^^^^^^^     ^^^^^^^     ^^^
```

#### 生命周期标注

只是标注，并不修改生命周期

```text
&i32        // 一个引用
&'a i32     // 具有显式生命周期的引用
&'a mut i32 // 具有显式生命周期的可变引用
```

此处生命周期标注仅仅说明，这两个参数 first 和 second 至少活得和'a 一样久，至于到底活多久或者哪个活得更久，抱歉我们都无法得知：
```rust
fn useless<'a>(first: &'a i32, second: &'a i32) {}
```

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

- 和泛型一样，使用生命周期参数，需要先声明 <'a>
- x、y 和返回值至少活得和 'a 一样久(因为返回值要么是 x，要么是 y)

虽然两个参数的生命周期都是标注了 'a，但是实际上这两个参数的真实生命周期可能是不一样的(生命周期 'a 不代表生命周期等于 'a，而是大于等于 'a)。

在通过函数签名指定生命周期参数时，我们并没有改变传入引用或者返回引用的真实生命周期，而是告诉编译器当不满足此约束条件时，就拒绝编译通过。


-----------

```rust
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
```

在上例中，string1 的作用域直到 main 函数的结束，而 string2 的作用域到内部花括号的结束 }，
那么根据之前的理论，'a 是两者中作用域较小的那个，也就是 'a 的生命周期等于 string2 的生命周期，
同理，由于函数返回的生命周期也是 'a，可以得出函数返回的生命周期也等于 string2 的生命周期。


-----------

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```
y 完全没有被使用，因此 y 的生命周期与 x 和返回值的生命周期没有任何关系，意味着我们也不必再为 y 标注生命周期，只需要标注 x 参数和返回值即可。


函数的返回值如果是一个引用类型，那么它的生命周期只会来源于：
- 函数参数的生命周期
- 函数体中某个新建引用的生命周期

若是后者情况，就是典型的悬垂引用场景


生命周期语法用来将函数的==多个引用 参数和返回值==的作用域关联到一起，一旦关联到一起后，Rust 就拥有充分的信息来确保我们的操作是内存安全的。


#### 结构体中的生命周期

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

下面的不行，因为结构体比string活的更久
```rust
#[derive(Debug)]
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let i;
    {
        let novel = String::from("Call me Ishmael. Some years ago...");
        let first_sentence = novel.split('.').next().expect("Could not find a '.'");
        i = ImportantExcerpt {
            part: first_sentence,
        };
    }
    println!("{:?}",i);
}
```


-------------
生命周期消除

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```
对于 first_word 函数，它的返回值是一个引用类型，那么该引用只有两种情况：
- 从参数获取
- 从函数体内部新创建的变量获取

如果是后者，就会出现悬垂引用，最终被编译器拒绝，因此只剩一种情况：返回值的引用是获取自参数，这就意味着参数和返回值的生命周期是一样的。

参数的生命周期被称为 输入生命周期，返回值的生命周期被称为 输出生命周期

-------
#### 三条消除规则

编译器使用三条消除规则来确定哪些场景不需要显式地去标注生命周期

其中第一条规则应用在输入生命周期上，第二、三条应用在输出生命周期上。
若编译器发现三条规则都不适用时，就会报错，提示你需要手动标注生命周期。

1. 每一个引用参数都会获得独自的生命周期
2. 若只有一个输入生命周期(函数参数中只有一个引用类型)，那么该生命周期会被赋给所有的输出生命周期
3. 若存在多个输入生命周期，且其中一个是 &self 或 &mut self，则 &self 的生命周期被赋给所有的输出生命周期

对于第三条规则，若一个方法，它的返回值的生命周期就是跟参数 &self 的不一样怎么办？总不能强迫我返回的值总是和 &self 活得一样久吧？
答案很简单：手动标注生命周期


-----------

为具有生命周期的结构体实现方法时，我们使用的语法跟泛型参数语法很相似

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

- impl 中必须使用结构体的完整名称，包括 <'a>，因为生命周期标注也是结构体类型的一部分！
- 方法签名中，往往不需要标注生命周期，得益于生命周期消除的第一和第三规则


-----------

```rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part<'b>(&'a self, announcement: &'b str) -> &'b str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```
编译器会报错，因为它不知道 'a 'b 的关系。
&self 的生命周期是 'a, 所以 返回值 self.part 的生命周期也是 'a， 但是声明中使用了 'b, 所以需要知道 'a 'b 的关系。

容易推理出来：由于 &'a self 是被引用的一方，因此引用它的 &'b str 必须要活得比它短，否则会出现悬垂引用。
因此说明生命周期 'b 必须要比 'a 小，只要满足了这一点，编译器就不会再报错：
```rust
impl<'a: 'b, 'b> ImportantExcerpt<'a> {
    fn announce_and_return_part(&'a self, announcement: &'b str) -> &'b str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```

- 'a: 'b，是生命周期约束语法，跟泛型约束非常相似，用于说明 'a 必须比 'b 活得久
- 可以把 'a 和 'b 都在同一个地方声明（如上），或者分开声明但通过 where 'a: 'b 约束生命周期关系，如下：

```rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part<'b>(&'a self, announcement: &'b str) -> &'b str
    where
        'a: 'b,
    {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```


#### 静态生命周期

非常特殊的生命周期，那就是 'static，拥有该生命周期的引用可以和整个程序活得一样久。

字符串字面量，提到过它是被硬编码进 Rust 的二进制文件中，因此这些字符串变量全部具有 'static 的生命周期：
`let s: &'static str = "我没啥优点，就是活得久，嘿嘿";`

- 生命周期 'static 意味着能和程序活得一样久，例如字符串字面量和特征对象
- 实在遇到解决不了的生命周期标注问题，可以尝试 T: 'static，有时候它会给你==奇迹==

事实上，关于 'static, 有两种用法: &'static 和 T: 'static，详细内容请参见此处。https://course.rs/advance/lifetime/static.html



--------------

一个复杂例子: 泛型、特征约束

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```


### 返回值和错误处理

`Result<T, E>` 用于可恢复错误，panic! 用于不可恢复错误。



-----------------

在 Rust 中触发 panic 有两种方式：被动触发和主动调用

> 被动触发

```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

> 主动调用

```rust
fn main() {
    panic!("crash and burn");
}
```


-----------
backtrace 栈展开

```text
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```
根据上面的提示：
`RUST_BACKTRACE=1 cargo run 或 $env:RUST_BACKTRACE=1 ; cargo run`

要获取到栈回溯信息，你还需要开启 debug 标志，该标志在使用 cargo run 或者 cargo build 时自动开启（这两个操作默认是 Debug 运行方式）


------------
panic 时 2种终止方式

两种方式来处理终止流程：栈展开和直接终止。

默认的方式就是 栈展开，这意味着 Rust 会回溯栈上数据和函数调用，因此也意味着更多的善后工作，好处是可以给出充分的报错信息和栈调用信息，便于事后的问题复盘。

直接终止，顾名思义，不清理数据就直接退出程序，善后工作交与操作系统来负责。

当你关心最终编译出的二进制可执行文件大小时，那么可以尝试去使用直接终止的方式，
例如下面的配置修改 Cargo.toml 文件，实现在 release 模式下遇到 panic 直接终止：
```text
[profile.release]
panic = 'abort'
```

--------------
线程panic后，程序是否会终止？
长话短说，如果是 main 线程，则程序会终止，如果是其它子线程，该线程会终止，但是不会影响 main 线程。
因此，尽量不要在 main 线程中做太多任务，将这些任务交由子线程去做，就算子线程 panic 也不会导致整个程序的结束。



-----------
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

对于 Result 返回我们有很多处理方法，最简单粗暴的就是 unwrap 和 expect，这两个函数非常类似，我们以 unwrap 举例：
```rust
use std::net::IpAddr;
let home: IpAddr = "127.0.0.1".parse().unwrap();
```
如果解析成功，则把 Ok(IpAddr) 中的值赋给 home，如果失败，则不处理 Err(E)，而是直接 panic。
因此 unwrap 简而言之：成功则返回值，失败则 panic，总之不进行任何错误处理。



-----------------
示例、原型、测试

这几个场景下，需要快速地搭建代码，错误处理会拖慢编码的速度，也不是特别有必要，因此通过 unwrap、expect 等方法来处理是最快的。

同时，当我们回头准备做错误处理时，可以全局搜索这些方法，不遗漏地进行替换。

```rust
use std::net::IpAddr;
let home: IpAddr = "127.0.0.1".parse().unwrap();
```
例如上面的例子，"127.0.0.1" 就是 ip 地址，因此我们知道 parse 方法==一定会成功==，那么就可以直接用 unwrap 方法进行处理。

当然，如果该字符串是来自于用户输入，那在实际项目中，就必须用错误处理的方式，而不是 unwrap，否则你的程序一天要崩溃几十万次吧！



有害状态大概分为几类：
- 非预期的错误
- 后续代码的运行会受到显著影响
- 内存安全的问题

。。上面3类都应该直接 panic。 而不是进行修复



#### panic原理
当调用 panic! 宏时，它会
- 格式化 panic 信息，然后使用该信息作为参数，调用 std::panic::panic_any() 函数
- panic_any 会检查应用是否使用了 panic hook，如果使用了，该 hook 函数就会被调用（hook 是一个钩子函数，是外部代码设置的，用于在 panic 触发时，执行外部代码所需的功能）
- 当 hook 函数返回后，当前的线程就开始进行栈展开：从 panic_any 开始，如果寄存器或者栈因为某些原因信息错乱了，那很可能该展开会发生异常，最终线程会直接停止，展开也无法继续进行
- 展开的过程是一帧一帧的去回溯整个栈，每个帧的数据都会随之被丢弃，但是在展开过程中，你可能会遇到被用户标记为 catching 的帧（通过 std::panic::catch_unwind() 函数标记），此时用户提供的 catch 函数会被调用，展开也随之停止：当然，如果 catch 选择在内部调用 std::panic::resume_unwind() 函数，则展开还会继续。

一旦线程展开被终止或者完成，最终的输出结果是取决于哪个线程 panic：
- 对于 main 线程，操作系统提供的终止功能 core::intrinsics::abort() 会被调用，最终结束当前的 panic 进程；
- 如果是其它子线程，那么子线程就会简单的终止，同时信息会在稍后通过 std::thread::join() 进行收集。


#### 可恢复的错误 Result

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```


> 对返回的错误进行处理

直接 panic 还是过于粗暴，因为实际上 IO 的错误有很多种，我们需要对部分错误进行特殊处理，而不是所有错误都直接崩溃：

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```


-----------
在不需要处理错误的场景，例如写原型、示例时，我们不想使用 match 去匹配 `Result<T, E>` 以获取其中的 T 值，因为 match 的穷尽匹配特性，你总要去处理下 Err 分支。那么有没有办法简化这个过程？有，答案就是 unwrap 和 expect。

它们的作用就是，如果返回成功，就将 Ok(T) 中的值取出来，如果失败，就直接 panic

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

expect 跟 unwrap 很像，也是遇到错误直接 panic, 但是会带上自定义的错误提示信息，相当于重载了错误打印的函数：
```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```
expect 相比 unwrap 能提供更精确的错误信息，在有些场景也会更加实用。


#### 传播错误

实际应用中，大概率会把错误层层上传然后交给调用链的上游函数进行处理，错误传播将极为常见。

例如以下函数从文件中读取用户名，然后将结果进行返回：
```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    // 打开文件，f是`Result<文件句柄,io::Error>`
    let f = File::open("hello.txt");

    let mut f = match f {
        // 打开文件成功，将file句柄赋值给f
        Ok(file) => file,
        // 打开文件失败，将错误返回(向上传播)
        Err(e) => return Err(e),
    };
    // 创建动态字符串s
    let mut s = String::new();
    // 从f文件句柄读取数据并写入s中
    match f.read_to_string(&mut s) {
        // 读取成功，返回Ok封装的字符串
        Ok(_) => Ok(s),
        // 将错误向上传播
        Err(e) => Err(e),
    }
}
```


#### 错误传播(Result)的符号: ?

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```
其实 ? 就是一个宏，它的作用跟上面的 match 几乎一模一样：
```rust
let mut f = match f {
    // 打开文件成功，将file句柄赋值给f
    Ok(file) => file,
    // 打开文件失败，将错误返回(向上传播)
    Err(e) => return Err(e),
};
```
如果结果是 Ok(T)，则把 T 赋值给 f，如果结果是 Err(E)，则返回该错误，所以 ? 特别适合用来传播错误。

虽然 ? 和 match 功能一致，但是事实上 ? 会更胜一筹。何解？

想象一下，一个设计良好的系统中，肯定有自定义的错误特征，错误之间很可能会存在上下级关系，例如标准库中的 std::io::Error 和 std::error::Error，前者是 IO 相关的错误结构体，后者是一个最最通用的标准错误特征，同时前者实现了后者，因此 std::io::Error 可以转换为 std:error::Error。

明白了以上的错误转换，? 的更胜一筹就很好理解了，它可以自动进行类型提升（转换）：
```rust
fn open_file() -> Result<File, Box<dyn std::error::Error>> {
    let mut f = File::open("hello.txt")?;
    Ok(f)
}
```

根本原因是在于标准库中定义的 ==From== 特征，该特征有一个方法 from，用于把一个类型转成另外一个类型，? 可以自动调用该方法，然后进行隐式类型转换。因此只要函数返回的错误 ReturnError 实现了 `From<OtherError>` 特征，那么 ? 就会自动把 OtherError 转换为 ReturnError。

这种转换非常好用，意味着你可以用一个大而全的 ReturnError 来覆盖所有错误类型，只需要为各种子错误类型实现这种转换即可。


---------------------

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```
? 还能实现链式调用


------------
```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    // read_to_string是定义在std::io中的方法，因此需要在上面进行引用
    fs::read_to_string("hello.txt")
}
```


#### ? 用于 Option 的返回

? 不仅仅可以用于 Result 的传播，还能用于 Option 的传播，再来回忆下 Option 的定义：
```rust
pub enum Option<T> {
    Some(T),
    None
}
```

Result 通过 ? 返回错误，那么 Option 就通过 ? 返回 None：

```rust
fn first(arr: &[i32]) -> Option<&i32> {
   let v = arr.get(0)?;
   Some(v)
}
```
arr.get 返回一个 Option<&i32> 类型，因为 ? 的使用，
如果 get 的结果是 None，则直接返回 None，
如果是 Some(&i32)，则把里面的值赋给 v。


```rust
fn first(arr: &[i32]) -> Option<&i32> {
   arr.get(0)
}
```

更简单的版本：
```rust
fn first(arr: &[i32]) -> Option<&i32> {
   arr.get(0)
}
```


```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```


##### 新手用 ? 常会犯的错误

```rust
fn first(arr: &[i32]) -> Option<&i32> {
   arr.get(0)?
}
```
无法通过编译，切记：? 操作符需要一个变量来承载正确的值
如果数组中存在 0 号元素，那么函数第二行使用 ? 后的返回类型为 `&i32` 而不是 Some(&i32)

因此 ? 只能用于以下形式：
- let v = xxx()?;
- xxx()?.yyy()?;

。都有 ;;;;


--------------
带返回值的 main 函数

一种main：`fn main() { .. }`

另外一种形式的 main 函数：
```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let f = File::open("hello.txt")?;

    Ok(())
}
```

至于 main 函数可以有多种返回值，那是因为实现了 std::process::Termination 特征，目前为止该特征还没进入稳定版 Rust 中，也许未来你可以为自己的类型实现该特征！



-----------
> try!

在 ? 横空出世之前( Rust 1.13 )，Rust 开发者还可以使用 try! 来处理错误

```rust
macro_rules! try {
    ($e:expr) => (match $e {
        Ok(val) => val,
        Err(err) => return Err(::std::convert::From::from(err)),
    });
}
```

```rust
//  `?`
let x = function_with_error()?; // 若返回 Err, 则立刻返回；若返回 Ok(255)，则将 x 的值设置为 255

// `try!()`
let x = try!(function_with_error());
```
可以看出 ? 的优势非常明显，何况 ? 还能做链式调用。

我们要尽量避免使用 try!。


### 包和模块

Rust 也提供了相应概念用于代码的组织管理：
- 项目(Packages)：一个 Cargo 提供的 feature，可以用来构建、测试和分享包
- 包(Crate)：一个由多个模块组成的树形结构，可以作为三方库进行分发，也可以生成可执行文件进行运行
- 模块(Module)：可以一个文件多个模块，也可以一个文件一个模块，模块可以被认为是真实项目中的代码组织单元

--------------
。。 package ： 项目
。。 crate ： 包

Rust 为我们提供了强大的包管理工具：
- 项目(Package)：可以用来构建、测试和分享包
- 工作空间(WorkSpace)：对于大型项目，可以进一步将多个包联合在一起，组织成工作空间
- 包(Crate)：一个由多个模块组成的树形结构，可以作为三方库进行分发，也可以生成可执行文件进行运行
- 模块(Module)：可以一个文件多个模块，也可以一个文件一个模块，模块可以被认为是真实项目中的代码组织单元


-----------
包 Crate

对于 Rust 而言，包是一个独立的可编译单元，它编译后会生成一个可执行文件或者一个库。

同一个包中不能有同名的类型，但是在不同包中就可以。例如，虽然 rand 包中，有一个 Rng 特征，可是我们依然可以在自己的项目中定义一个 Rng，前者通过 rand::Rng 访问，后者通过 Rng 访问


----------------
项目 package

由于 Package 就是一个项目，因此它包含有独立的 Cargo.toml 文件，以及因为功能性被组织在一起的一个或多个包。
一个 Package 只能包含一个库(library)类型的包，但是可以包含多个二进制可执行类型的包。


--------------
二进制 Package

创建一个二进制 Package：
```sh
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```
Cargo 为我们创建了一个名称是 my-project 的 Package，同时在其中创建了 Cargo.toml 文件，
可以看一下该文件，里面并没有提到 src/main.rs 作为程序的入口，
原因是 Cargo 有一个惯例：src/main.rs 是二进制包的根文件，该二进制包的==包名==跟所属 ==Package== 相同，在这里都是 my-project，所有的代码执行都从该文件中的 fn main() 函数开始。


--------------
库 Package

创建一个库类型的 Package：
```sh
$ cargo new my-lib --lib
     Created library `my-lib` package
$ ls my-lib
Cargo.toml
src
$ ls my-lib/src
lib.rs
```
库类型的 Package 只能作为三方库被其它项目引用，而不能独立运行，只有之前的二进制 Package 才可以运行。


只要你牢记 Package 是一个项目工程，
而包只是一个编译单元，
基本上也就不会混淆这个两个概念了：src/main.rs 和 src/lib.rs 都是编译单元，因此它们都是包。


#### 典型的 Package 结构

如果一个 Package 同时拥有 src/main.rs 和 src/lib.rs，那就意味着它包含两个包：库包和二进制包，这两个包名也都是 my-project —— 都与 Package 同名。

一个真实项目中典型的 Package，会包含多个二进制包，这些包文件被放在 src/bin 目录下，每一个文件都是独立的二进制包，同时也会包含一个库包，该包只能存在一个 src/lib.rs：

```text
.
├── Cargo.toml
├── Cargo.lock
├── src
│   ├── main.rs
│   ├── lib.rs
│   └── bin
│       └── main1.rs
│       └── main2.rs
├── tests
│   └── some_integration_tests.rs
├── benches
│   └── simple_bench.rs
└── examples
    └── simple_example.rs
```

- 唯一库包：src/lib.rs
- 默认二进制包：src/main.rs，编译后生成的可执行文件与 Package 同名
- 其余二进制包：src/bin/main1.rs 和 src/bin/main2.rs，它们会分别生成一个文件同名的二进制可执行文件
- 集成测试文件：tests 目录下
- 基准性能测试 benchmark 文件：benches 目录下
- 项目示例：examples 目录下


-----------------

#### 模块 Module

Rust 的代码构成单元：模块

使用模块可以将包中的代码按照功能性进行重组，最终实现更好的可读性及易用性。
同时，我们还能非常灵活地去控制代码的可见性，进一步强化 Rust 的安全性。

----------
> 创建嵌套模块

`cargo new --lib restaurant` 创建一个小餐馆
将以下代码放入 src/lib.rs 中：
```rust
// 餐厅前厅，用于吃饭
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

以上的代码创建了三个模块，有几点需要注意的：
- 使用 mod 关键字来创建新模块，后面紧跟着模块名称
- 模块可以嵌套，这里嵌套的原因是招待客人和服务都发生在前厅，因此我们的代码模拟了真实场景
- 模块中可以定义各种 Rust 类型，例如函数、结构体、枚举、特征等
- 所有模块均定义在同一个文件中


---------------
模块树

src/main.rs 和 src/lib.rs 被称为包根(crate root)

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```


-------------
##### 用路径引用模块

想要调用一个函数，就需要知道它的路径，在 Rust 中，这种路径有两种形式：
- 绝对路径，从包根开始，路径名以包名或者 crate 作为开头
- 相对路径，从当前模块开始，以 self，super 或当前模块的标识符作为开头


```rust
// src/lib.rs
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

当代码被挪动位置时，尽量减少引用路径的修改

如果不确定哪个好，你可以考虑优先使用绝对路径，因为调用的地方和定义的地方往往是分离的，而定义的地方较少会变动。

。。如果 被导入的模块 和 自己这个模块 在一起， 如果 一起移动到 其他地方，那么 相对路径不需要修改。
。。如果 自己 被移动了， 那么 绝对路劲不需要修改。

。。导入其他模块， 基本上 就代表 自己是 实现。  实现文件 移动的可能性较大， 
。。被导入的模块，基本代表 定义， 定义文件 的位置一般 经过 深思熟虑，移动的可能性不大， 而且，我们也知道 一旦移动，所有导入它的地方都可能需要改，所以 更加不可能移动 定义。
。。还有上面提到了， 定义 和 调用 的地方是分开的， 所以 不太可能 一起移动。
。。所以 用 绝对路径。
。。嵌套包 的 内部包 可以用相对路径 super 调用 外层包。 所以 super 应该优先于 绝对路径


#### 代码可见性

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```
会编译失败： module `hosting` is private

为何 front_of_house 模块就可以访问？因为它和 eat_at_restaurant 同属于一个包根作用域内

默认情况下，所有的类型都是私有化的，包括函数、方法、结构体、枚举、常量，是的，就连模块本身也是私有化的。

父模块完全无法访问子模块中的私有项，但是子模块却可以访问父模块、父父..模块的私有项


-----------
> pub

Go 语言中的首字母大写。 就是 public

```rust
mod front_of_house {
    pub mod hosting {
        fn add_to_waitlist() {}
    }
}

/*--- snip ----*/
```
失败： function `add_to_waitlist` is private


```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

/*--- snip ----*/
```
ok


------------
> super

相对路径有三种方式开始：self、super和 crate或者模块名

super 代表的是父模块为开始的引用方式，非常类似于文件系统中的 .. 语法：../a/b 文件

```rust
fn serve_order() {}

// 厨房模块
mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();
    }

    fn cook_order() {}
}
```

------------
> self

self 其实就是引用自身模块中的项
都调用同一模块中的内容，区别在于之前章节中直接通过名称调用即可，而 self，你得多此一举

```rust
fn serve_order() {
    self::back_of_house::cook_order()
}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        crate::serve_order();
    }

    pub fn cook_order() {}
}
```
是的，多此一举，因为完全可以直接调用 back_of_house，但是 self 还有一个大用处，在下一节中我们会讲。



#### 结构体和枚举的可见性
这两个家伙的成员字段拥有完全不同的可见性：
- 将结构体设置为 pub，但它的所有字段依然是==私有==的
- 将枚举设置为 pub，它的所有字段也将==对外可见==

原因在于，枚举和结构体的使用方式不一样。如果枚举的成员对外不可见，那该枚举将一点用都没有，因此枚举成员的可见性自动跟枚举可见性保持一致，这样可以简化用户的使用。


----------------
> 模块与文件分离

把 front_of_house 前厅分离出来，放入一个单独的文件中 src/front_of_house.rs：
```rust
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```


将以下代码留在 src/lib.rs 中：
```rust
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```

- mod front_of_house; 告诉 Rust 从另一个和模块 front_of_house 同名的==文件==中加载该模块的内容
- 使用绝对路径的方式来引用 hosting 模块：crate::front_of_house::hosting;


---------------
我们可以创建一个目录 front_of_house，然后在文件夹里创建一个 hosting.rs 文件，hosting.rs 文件现在就剩下：
`pub fn add_to_waitlist() {}`
编译失败： file not found for module `front_of_house`

如果需要将文件夹作为一个模块，我们需要进行显示指定暴露哪些子模块。按照上述的报错信息，我们有两种方法：
- 在 front_of_house 目录里创建一个 mod.rs，如果你使用的 rustc 版本 1.30 之前，这是唯一的方法。
- 在 front_of_house 同级目录里创建一个与模块（目录）同名的 rs 文件 front_of_house.rs，在新版本里，更建议使用这样的命名方式来避免项目中存在大量同名的 mod.rs 文件（ Python 点了个 踩）。

。。。？


-----------------

#### use

使用 use 关键字把路径提前引入到当前作用域中，随后的调用就可以省略该路径，极大地简化了代码。


> 绝对路径引入模块

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```

> 相对路径引入模块中的函数
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use front_of_house::hosting::add_to_waitlist;

pub fn eat_at_restaurant() {
    add_to_waitlist();
    add_to_waitlist();
    add_to_waitlist();
}
```


--------------
> 引入模块还是函数

从使用简洁性来说，引入函数自然是更甚一筹，但是在某些时候，引入模块会更好：
- 需要引入同一个模块的多个函数
- 作用域中存在同名函数

建议：优先使用最细粒度(引入函数、结构体等)的引用方式，如果引起了某种麻烦(例如前面两种情况)，再使用引入模块的方式。


-----------
遇到同名的情况该如何处理

> 模块::函数

```rust
use std::fmt;
use std::io;

fn function1() -> fmt::Result {
    // --snip--
}

fn function2() -> io::Result<()> {
    // --snip--
}
```

> as 别名引用

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
```


------------------

#### 引入项再导出 pub use

当外部的模块项 A 被引入到当前模块中时，它的可见性自动被设置为私有的，如果你希望允许其它外部代码引用我们的模块项 A，那么可以对它进行再导出：

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;         // pub

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```
这里 use 代表引入 hosting 模块到当前作用域，pub 表示将该引入的内容再度设置为可见。

当你希望将内部的实现细节隐藏起来或者按照某个目的组织代码时，可以使用 pub use 再导出，
例如统一使用一个模块来提供对外的 API，那该模块就可以引入其它模块中的 API，然后进行再导出，最终对于用户来说，所有的 API 都是由一个模块统一提供的。


-----------------

引入第三方包中的模块

- 修改 Cargo.toml 文件，在 [dependencies] 区域添加一行：rand = "0.8.3"
- 此时，如果你用的是 VSCode 和 rust-analyzer 插件，该插件会自动拉取该库，你可能需要等它完成后，再进行下一步（VSCode 左下角有提示）

```rust
use rand::Rng;

fn main() {
    let secret_number = rand::thread_rng().gen_range(1..101);
}
```

Rust 社区已经为我们贡献了大量高质量的第三方包，你可以在 crates.io 或者 lib.rs 中检索和使用，从目前来说查找包更推荐 lib.rs，搜索功能更强大，内容展示也更加合理，但是下载依赖包还是得用crates.io。


--------------
使用 {} 简化引入方式

```rust
use std::collections::HashMap;
use std::collections::BTreeMap;
use std::collections::HashSet;

use std::cmp::Ordering;
use std::io;
```
```rust
use std::collections::{HashMap,BTreeMap,HashSet};
use std::{cmp::Ordering, io};
```

-----------
```rust
use std::io;
use std::io::Write;
```
`use std::io::{self, Write};`           // self

------
self，它在模块中的两个用途：
- use self::xxx，表示加载当前模块中的 xxx。此时 self 可省略
- use xxx::{self, yyy}，表示，加载当前路径下模块 xxx 本身，以及模块 xxx 下的 yyy


-------------
使用 * 引入模块下的所有项

`use std::collections::*;`

当使用 * 来引入的时候要格外小心，因为你很难知道到底哪些被引入到了当前作用域中，有哪些会和你自己程序中的名称相冲突：
```rust
use std::collections::*;

struct HashMap;
fn main() {
   let mut v =  HashMap::new();
   v.insert("a", 1);
}
```
编译失败，没有new方法。
对于编译器来说，本地同名类型的优先级更高。



-----------

#### 受限的可见性

https://course.rs/basic/crate-module/use.html#%E5%8F%97%E9%99%90%E7%9A%84%E5%8F%AF%E8%A7%81%E6%80%A7

可见性：控制了模块中哪些内容可以被外部看见

控制哪些人能看，这就是 Rust 提供的受限可见性。


在 Rust 中，包是一个模块树，我们可以通过 pub(crate) item; 这种方式来实现：item 虽然是对外可见的，但是只在当前包内可见，外部包无法引用到该 item。

。。？ 可见，但无法引用。。可见了干嘛？。
。。这个描述也有点不对， 应该是 当前包内可以引用吧。

如果我们想要让某一项可以在整个包中都可以被使用，那么有两种办法：
- 在包根中定义一个非 pub 类型的 X(父模块的项对子模块都是可见的，因此包根中的项对模块树上的所有模块都可见)
- 在子模块中定义一个 pub 类型的 Y，同时通过 use 将其引入到包根

```rust
mod a {
    pub mod b {
        pub fn c() {
            println!("{:?}",crate::X);
        }

        #[derive(Debug)]
        pub struct Y;
    }
}

#[derive(Debug)]
struct X;
use a::b::Y;
fn d() {
    println!("{:?}",Y);
}
```

----------

有时我们会遇到这两种方法都不太好用的时候。例如希望对于某些特定的模块可见，但是对于其他模块又不可见：
```rust
// 目标：`a` 导出 `I`、`bar` and `foo`，其他的不导出
pub mod a {
    pub const I: i32 = 3;

    fn semisecret(x: i32) -> i32 {
        use self::b::c::J;
        x + J
    }

    pub fn bar(z: i32) -> i32 {
        semisecret(I) * z
    }
    pub fn foo(y: i32) -> i32 {
        semisecret(I) + y
    }

    mod b {
        mod c {
            const J: i32 = 4;
        }
    }
}
```
这段代码会报错，因为与父模块中的项对子模块可见相反，子模块中的项对父模块是不可见的。这里 semisecret 方法中，a -> b -> c 形成了父子模块链，那 c 中的 J 自然对 a 模块不可见。

如果使用之前的可见性方式，那么想保持 J 私有，同时让 a 继续使用 semisecret 函数的办法是将该函数移动到 c 模块中，然后用 pub use 将 semisecret 函数进行再导出：
```rust
pub mod a {
    pub const I: i32 = 3;

    use self::b::semisecret;

    pub fn bar(z: i32) -> i32 {
        semisecret(I) * z
    }
    pub fn foo(y: i32) -> i32 {
        semisecret(I) + y
    }

    mod b {
        pub use self::c::semisecret;
        mod c {
            const J: i32 = 4;
            pub fn semisecret(x: i32) -> i32 {
                x + J
            }
        }
    }
}
```
这段代码说实话问题不大，但是有些破坏了我们之前的逻辑，如果想保持代码逻辑，同时又只让 J 在 a 内可见该怎么办？

```rust
pub mod a {
    pub const I: i32 = 3;

    fn semisecret(x: i32) -> i32 {
        use self::b::c::J;
        x + J
    }

    pub fn bar(z: i32) -> i32 {
        semisecret(I) * z
    }
    pub fn foo(y: i32) -> i32 {
        semisecret(I) + y
    }

    mod b {
        pub(in crate::a) mod c {                // pub(in )
            pub(in crate::a) const J: i32 = 4;
        }
    }
}
```
通过 pub(in crate::a) 的方式，我们指定了模块 c 和常量 J 的可见范围都只是 a 模块中，a 之外的模块是完全访问不到它们的。


----------
###### 限制可见性语法

pub(crate) 或 pub(in crate::a) 就是限制可见性语法，前者是限制在整个包内可见，后者是通过绝对路径，限制在包内的某个模块内可见，总结一下：

- pub 意味着可见性无任何限制
- pub(crate) 表示在当前包可见
- pub(self) 在当前模块可见
- pub(super) 在父模块可见
- `pub(in <path>)` 表示在某个路径代表的模块中可见，其中 path 必须是父模块或者祖先模块




### 注释和文档

在 Rust 中，注释分为三类：
- 代码注释，用于说明某一块代码的功能，读者往往是同一个项目的协作开发者
- 文档注释，支持 ==Markdown==，对项目描述、公共 API 等用户关心的功能进行介绍，同时还能提供示例代码，目标读者往往是想要了解你项目的人
- 包和模块注释，严格来说这也是文档注释中的一种，它主要用于说明当前包和模块的功能，方便用户迅速了解一个项目

-----------
代码注释
- 行注释 //
- 块注释/* ..... */


----------
文档注释

当查看一个 crates.io 上的包时，往往需要通过它提供的文档来浏览相关的功能特性、使用方式，这种文档就是通过文档注释实现的。

Rust 提供了 cargo doc 的命令，可以用于把这些文档注释转换成 HTML 网页文件，最终展示给用户浏览，这样用户就知道这个包是做什么的以及该如何使用。

- 文档行注释 ///
- 文档块注释 /** ... */


- 文档注释需要位于 lib 类型的包中，例如 src/lib.rs 中
- 文档注释可以使用 markdown语法！例如 # Examples 的标题，以及代码块高亮
- 被注释的对象需要使用 pub 对外可见，记住：文档注释是给用户看的，内部实现细节不应该被暴露出去


```rust
/// `add_one` 将指定值加1
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

```rust
/** `add_two` 将指定值加2


\`\`\`
let arg = 5;
let answer = my_crate::add_two(arg);

assert_eq!(7, answer);
\`\`\`
*/
pub fn add_two(x: i32) -> i32 {
    x + 2
}
```

。。注释里面就是 ``` ， 斜杠，我加的。



---------------
##### cargo doc

cargo doc 可以直接生成 HTML 文件，放入target/doc目录下。
cargo doc --open 命令，可以在生成文档后，自动在浏览器中打开网页




----------
常用文档标题

除了`# Examples` ，还有一些常用的，你可以在项目中酌情使用
- Panics：函数可能会出现的异常状况，这样调用函数的人就可以提前规避
- Errors：描述可能出现的错误及什么情况会导致错误，有助于调用者针对不同的错误采取不同的处理方式
- Safety：如果函数使用 unsafe 代码，那么调用者就需要注意一些使用条件，以确保 unsafe 代码块的正常工作



---------
包和模块级别的注释

这些注释要添加到包、模块的最上方！

- 行注释 //! 
- 块注释 /*! ... */

```rust
/*! lib包是world_hello二进制包的依赖包，
 里面包含了compute等有用模块 */

pub mod compute;
```

```rust
//! 计算一些你口算算不出来的复杂算术题


/// `add_one`将指定值加1
///
```

运行 cargo doc --open 查看下效果

----------
#### 文档测试(Doc Test)

Rust 允许我们在文档注释中写单元测试用例

```rust
/// `add_one` 将指定值加1
///
/// # Examples11
///
/// ```
/// let arg = 5;
/// let answer = world_hello::compute::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```
以上的注释不仅仅是文档，还可以作为单元测试的用例运行
cargo test


-------
```rust
/// # Panics
///
/// The function panics if the second argument is zero.
///
/// ```rust
/// // panics on division by zero
/// world_hello::compute::div(10, 0);
/// ```
pub fn div(a: i32, b: i32) -> i32 {
    if b == 0 {
        panic!("Divide-by-zero error");
    }

    a / b
}
```
以上测试运行后会 panic：
如果想要通过这种测试，可以添加 should_panic：

```rust
/// # Panics
///
/// The function panics if the second argument is zero.
///
/// ```rust,should_panic
/// // panics on division by zero
/// world_hello::compute::div(10, 0);
/// ```
```
通过 should_panic，告诉 Rust 我们这个用例会导致 panic，这样测试用例就能顺利通过。



-----------

```rust
/// \`\`\`
/// # // 使用#开头的行会在文档中被隐藏起来，但是依然会在文档测试中运行
/// # fn try_main() -> Result<(), String> {
/// let res = world_hello::compute::try_div(10, 0)?;
/// # Ok(()) // returning from try_main
/// # }
/// # fn main() {
/// #    try_main().unwrap();
/// #
/// # }
/// \`\`\`
pub fn try_div(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("Divide-by-zero"))
    } else {
        Ok(a / b)
    }
}
```
。。\ 我加的。

我们使用 # 将不想让用户看到的内容隐藏起来，但是又不影响测试用例的运行，最终用户将只能看到那行没有隐藏的 `let res = world_hello::compute::try_div(10, 0)?;`



---------

#### 文档注释中的代码跳转


跳转到标准库

```rust
/// `add_one` 返回一个[`Option`]类型
pub fn add_one(x: i32) -> Option<i32> {
    Some(x + 1)
}
```
此处的 `[Option]` 就是一个链接，指向了标准库中的 Option 枚举类型，有两种方式可以进行跳转:
- 在 IDE 中，使用 Command + 鼠标左键(macOS)，CTRL + 鼠标左键(Windows)
- 在文档中直接点击链接

--------------

使用路径的方式跳转：
```rust
use std::sync::mpsc::Receiver;

/// [`Receiver<T>`]   [`std::future`].
///
///  [`std::future::Future`] [`Self::recv()`].
pub struct AsyncReceiver<T> {
    sender: Receiver<T>,
}

impl<T> AsyncReceiver<T> {
    pub async fn recv() -> T {
        unimplemented!()
    }
}
```

-----------
使用完整路径跳转到指定项
```rust
pub mod a {
    /// `add_one` 返回一个[`Option`]类型
    /// 跳转到[`crate::MySpecialFormatter`]
    pub fn add_one(x: i32) -> Option<i32> {
        Some(x + 1)
    }
}

pub struct MySpecialFormatter;
```

----------
同名项的跳转

如果遇到同名项，可以使用标示类型的方式进行跳转：

```rust
/// 跳转到结构体  [`Foo`](struct@Foo)
pub struct Bar;

/// 跳转到同名函数 [`Foo`](fn@Foo)
pub struct Foo {}

/// 跳转到同名宏 [`foo!`]
pub fn Foo() {}

#[macro_export]
macro_rules! foo {
  () => {}
}
```

-------------
#### 文档搜索别名

```rust
#[doc(alias = "x")]
#[doc(alias = "big")]
pub struct BigX;

#[doc(alias("y", "big"))]
pub struct BigY;
```


---------
### 格式化输出

```rust
println!("Hello");                 // => "Hello"
println!("Hello, {}!", "world");   // => "Hello, world!"
println!("The number is {}", 1);   // => "The number is 1"
println!("{:?}", (3, 4));          // => "(3, 4)"
println!("{value}", value=4);      // => "4"
println!("{} {}", 1, 2);           // => "1 2"
println!("{:04}", 42);             // => "0042" with leading zeros
```


- print! 将格式化文本输出到标准输出，不带换行符
- println! 同上，但是在行的末尾添加换行符
- format! 将格式化文本输出到 String 字符串

最常用的是 println! 及 format!，前者常用来调试输出，后者常用来生成格式化的字符串：


输出到标准错误输出：
- eprint!
- eprintln!


---------

#### {} vs {:?}

与 {} 类似，{:?} 也是占位符：
- {} 适用于实现了 std::fmt::Display 特征的类型，用来以更优雅、更友好的方式格式化文本，例如展示给用户
- {:?} 适用于实现了 std::fmt::Debug 特征的类型，用于调试场景

代码需要调试时，使用 {:?}，剩下的场景，选择 {}。

-----------
> Debug 特征

大多数 Rust 类型都实现了 Debug 特征或者支持派生该特征：
```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8
}

fn main() {
    let i = 3.1415926;
    let s = String::from("hello");
    let v = vec![1, 2, 3];
    let p = Person{name: "sunface".to_string(), age: 18};
    println!("{:?}, {:?}, {:?}, {:?}", i, s, v, p);
}
```

对于数值、字符串、数组，可以直接使用 {:?} 进行输出，但是对于结构体，需要派生Debug特征后，才能进行输出，总之很简单。


---------------

#### {:?} {:#?}

> Display 特征

实现了 Display 特征的 Rust 类型并没有那么多，往往需要我们自定义想要的格式化方式：

```rust
let i = 3.1415926;
let s = String::from("hello");
let v = vec![1, 2, 3];
let p = Person {
    name: "sunface".to_string(),
    age: 18,
};
println!("{}, {}, {}, {}", i, s, v, p);
```

v 和 p 都无法通过编译，因为没有实现 Display 特征，但是你又不能像派生 Debug 一般派生 Display，只能另寻他法：

- 使用 =={:?} 或 {:#?}==
- 为自定义类型实现 Display 特征
- 使用 newtype 为外部类型实现 Display 特征

----------------
{:#?} 与 {:?} 几乎一样，唯一的区别在于它能更优美地输出内容：

```text
// {:?}
[1, 2, 3], Person { name: "sunface", age: 18 }

// {:#?}
[
    1,
    2,
    3,
], Person {
    name: "sunface",
}
```

----------
如果你的类型是定义在当前作用域中的，那么可以为其实现 Display 特征，即可用于格式化输出：

```rust
struct Person {
    name: String,
    age: u8,
}

use std::fmt;
impl fmt::Display for Person {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(
            f,
            "大佬在上，请受我一拜，小弟姓名{}，年芳{}，家里无田又无车，生活苦哈哈",
            self.name, self.age
        )
    }
}
fn main() {
    let p = Person {
        name: "sunface".to_string(),
        age: 18,
    };
    println!("{}", p);
}
```

----------------
为外部类型实现 Display 特征

在 Rust 中，无法直接为外部类型实现外部特征，但是可以使用newtype解决此问题：

```rust
struct Array(Vec<i32>);

use std::fmt;
impl fmt::Display for Array {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "数组是：{:?}", self.0)
    }
}
fn main() {
    let arr = Array(vec![1, 2, 3]);
    println!("{}", arr);
}
```


----------------

位置参数

```rust
fn main() {
    println!("{}{}", 1, 2); // =>"12"
    println!("{1}{0}", 1, 2); // =>"21"
    // => Alice, this is Bob. Bob, this is Alice
    println!("{0}, this is {1}. {1}, this is {0}", "Alice", "Bob");
    println!("{1}{}{0}{}", 1, 2); // => 2112
}
```

-----------
具名参数

```rust
fn main() {
    println!("{argument}", argument = "test"); // => "test"
    println!("{name} {}", 1, name = 2); // => "2 1"
    println!("{a} {c} {b}", a = "a", b = 'b', c = 3); // => "a 3 b"
}
```

带名称的参数必须放在不带名称参数的后面
下面代码将报错：
`println!("{abc} {1}", abc = "def", 2);`


--------------
#### 格式化参数

输出浮点数的小数点后两位：
```rust
fn main() {
    let v = 3.1415926;
    // Display => 3.14
    println!("{:.2}", v);
    // Debug => 3.14
    println!("{:.2?}", v);
}
```

----------
宽度


字符串填充
字符串格式化默认使用空格进行填充，并且进行左对齐。

```rust
fn main() {
    //-----------------------------------
    // 以下全部输出 "Hello x    !"
    // 为"x"后面填充空格，补齐宽度5
    println!("Hello {:5}!", "x");
    // 使用参数5来指定宽度
    println!("Hello {:1$}!", "x", 5);
    // 使用x作为占位符输出内容，同时使用5作为宽度
    println!("Hello {1:0$}!", 5, "x");
    // 使用有名称的参数作为宽度
    println!("Hello {:width$}!", "x", width = 5);
    //-----------------------------------

    // 使用参数5为参数x指定宽度，同时在结尾输出参数5 => Hello x    !5
    println!("Hello {:1$}!{}", "x", 5);
}
```

---------------
数字填充:符号和 0

数字格式化默认也是使用空格进行填充，但与字符串左对齐不同的是，数字是右对齐。
```rust
fn main() {
    // 宽度是5 => Hello     5!
    println!("Hello {:5}!", 5);
    // 显式的输出正号 => Hello +5!
    println!("Hello {:+}!", 5);
    // 宽度5，使用0进行填充 => Hello 00005!
    println!("Hello {:05}!", 5);
    // 负号也要占用一位宽度 => Hello -0005!
    println!("Hello {:05}!", -5);
}
```

--------

对齐

```rust
fn main() {
    // 以下全部都会补齐5个字符的长度
    // 左对齐 => Hello x    !
    println!("Hello {:<5}!", "x");
    // 右对齐 => Hello     x!
    println!("Hello {:>5}!", "x");
    // 居中对齐 => Hello   x  !
    println!("Hello {:^5}!", "x");

    // 对齐并使用指定符号填充 => Hello x&&&&!
    // 指定符号填充的前提条件是必须有对齐字符
    println!("Hello {:&<5}!", "x");
}
```

---------------------
精度
精度可以用于控制浮点数的精度或者字符串的长度

```rust
fn main() {
    let v = 3.1415926;
    // 保留小数点后两位 => 3.14
    println!("{:.2}", v);
    // 带符号保留小数点后两位 => +3.14
    println!("{:+.2}", v);
    // 不带小数 => 3
    println!("{:.0}", v);
    // 通过参数来设定精度 => 3.1416，相当于{:.4}
    println!("{:.1$}", v, 4);

    let s = "hi我是Sunface孙飞";
    // 保留字符串前三个字符 => hi我
    println!("{:.3}", s);
    // {:.*}接收两个参数，第一个是精度，第二个是被格式化的值 => Hello abc!
    println!("Hello {:.*}!", 3, "abcdefg");
}
```

----------
进制
可以使用 # 号来控制数字的进制输出：
- #b, 二进制
- #o, 八进制
- #x, 小写十六进制
- #X, 大写十六进制
- x, 不带前缀的小写十六进制

```rust
fn main() {
    // 二进制 => 0b11011!
    println!("{:#b}!", 27);
    // 八进制 => 0o33!
    println!("{:#o}!", 27);
    // 十进制 => 27!
    println!("{}!", 27);
    // 小写十六进制 => 0x1b!
    println!("{:#x}!", 27);
    // 大写十六进制 => 0x1B!
    println!("{:#X}!", 27);

    // 不带前缀的十六进制 => 1b!
    println!("{:x}!", 27);

    // 使用0填充二进制，宽度为10 => 0b00011011!
    println!("{:#010b}!", 27);
}
```

----------
指数

```rust
fn main() {
    println!("{:2e}", 1000000000); // => 1e9
    println!("{:2E}", 1000000000); // => 1E9
}
```

------------
指针地址

```rust
let v= vec![1, 2, 3];
println!("{:p}", v.as_ptr()) // => 0x600002324050
```

-----------
转义
输出 {和}

```rust
fn main() {
    // "{{" 转义为 '{'   "}}" 转义为 '}'   "\"" 转义为 '"'
    // => Hello "{World}" 
    println!(" Hello \"{{World}}\" ");

    // 下面代码会报错，因为占位符{}只有一个右括号}，左括号被转义成字符串的内容
    // println!(" {{ Hello } ");
    // 也不可使用 '\' 来转义 "{}"
    // println!(" \{ Hello \} ")
}
```

----------
在格式化字符串时捕获环境中的值（Rust 1.58 新增）

想要输出一个函数的返回值，你需要这么做：
```rust
fn get_person() -> String {
    String::from("sunface")
}
fn main() {
    let p = get_person();
    println!("Hello, {}!", p);                // implicit position
    println!("Hello, {0}!", p);               // explicit index
    println!("Hello, {person}!", person = p);
}
```

而在 1.58 后，我们可以这么写：
```rust
fn get_person() -> String {
    String::from("sunface")
}
fn main() {
    let person = get_person();
    println!("Hello, {person}!");
}
```

甚至还可以将环境中的值用于格式化参数:
```rust
let (width, precision) = get_format();
for (name, score) in get_scores() {
  println!("{name}: {score:width$.precision$}");
}
```

它只能捕获普通的变量，对于更复杂的类型（例如表达式），可以先将它赋值给一个变量或使用以前的 name = expression 形式的格式化参数。 

```rust
fn get_person() -> String {
    String::from("sunface")
}
fn main() {
    let person = get_person();
    panic!("Hello, {person}!");
}
```


## ch3 入门实战：文件搜索工具

https://course.rs/basic-practice/intro.html






## ch4 高级进阶

### 生命周期

。。。
1. 每一个引用参数都会获得独自的生命周期
2. 若只有一个输入生命周期(函数参数中只有一个引用类型)，那么该生命周期会被赋给所有的输出生命周期
3. 若存在多个输入生命周期，且其中一个是 &self 或 &mut self，则 &self 的生命周期被赋给所有的输出生命周期
。。。。。。

------------
不太聪明的生命周期检查

```rust
#[derive(Debug)]
struct Foo;

impl Foo {
    fn mutate_and_share(&mut self) -> &Self {
        &*self
    }
    fn share(&self) {}
}

fn main() {
    let mut foo = Foo;
    let loan = foo.mutate_and_share();
    foo.share();
    println!("{:?}", loan);
}
```
编译失败
```text
error[E0502]: cannot borrow `foo` as immutable because it is also borrowed as mutable
  --> src/main.rs:12:5
   |
11 |     let loan = foo.mutate_and_share();
   |                ---------------------- mutable borrow occurs here
12 |     foo.share();
   |     ^^^^^^^^^^^ immutable borrow occurs here
13 |     println!("{:?}", loan);
   |                      ---- mutable borrow later used here
```

编译器的提示在这里其实有些难以理解，因为可变借用仅在 mutate_and_share 方法内部有效，
出了该方法后，就只有返回的不可变借用，
因此，按理来说可变借用不应该在 main 的作用范围内存在。

对于这个反直觉的事情，让我们用生命周期来解释下，可能你就很好理解了：
```rust
struct Foo;

impl Foo {
    fn mutate_and_share<'a>(&'a mut self) -> &'a Self {
        &'a *self
    }
    fn share<'a>(&'a self) {}
}

fn main() {
    'b: {
        let mut foo: Foo = Foo;
        'c: {
            let loan: &'c Foo = Foo::mutate_and_share::<'c>(&'c mut foo);
            'd: {
                Foo::share::<'d>(&'d foo);
            }
            println!("{:?}", loan);
        }
    }
}
```
以上是模拟了编译器的生命周期标注后的代码，可以注意到 &mut foo 和 loan 的生命周期都是 'c。

还记得生命周期消除规则中的第三条吗？因为该规则，导致了 mutate_and_share 方法中，参数 &mut self 和返回值 &self 的生命周期是相同的，因此，若返回值的生命周期在 main 函数有效，那 &mut self 的借用也是在 main 函数有效。

。。
就是，生命周期消除的 第三条规则，导致 入参 和 出参 的生命周期一样。
loan 接收值，所以 生命周期c 扩散到了 main中。 所以 入参 (&mut ) 的 生命周期 也扩散到 main中。并且 和 loan 的生命周期 一样。
。。

总结下：&mut self 借用的生命周期和 loan 的生命周期相同，将持续到 println 结束。而在此期间 foo.share() 又进行了一次不可变 &foo 借用，违背了可变借用与不可变借用不能同时存在的规则，最终导致了编译错误。

上述代码实际上完全是正确的，但是因为生命周期系统的“粗糙实现”，导致了编译错误，目前来说，遇到这种生命周期系统不够聪明导致的编译错误，我们也没有太好的办法，只能修改代码去满足它的需求，并期待以后它会更聪明。


-------
生命周期不太聪明的 另一个例子

```rust
#![allow(unused)]
fn main() {
    use std::collections::HashMap;
    use std::hash::Hash;
    fn get_default<'m, K, V>(map: &'m mut HashMap<K, V>, key: K) -> &'m mut V
    where
        K: Clone + Eq + Hash,
        V: Default,
    {
        match map.get_mut(&key) {
            Some(value) => value,
            None => {
                map.insert(key.clone(), V::default());
                map.get_mut(&key).unwrap()
            }
        }
    }
}
```
这段代码不能通过编译的原因是编译器未能精确地判断出某个可变借用不再需要，反而谨慎的给该借用安排了一个很大的作用域，结果导致后续的借用失败：
```text
error[E0499]: cannot borrow `*map` as mutable more than once at a time
  --> src/main.rs:13:17
   |
5  |       fn get_default<'m, K, V>(map: &'m mut HashMap<K, V>, key: K) -> &'m mut V
   |                      -- lifetime `'m` defined here
...
10 |           match map.get_mut(&key) {
   |           -     ----------------- first mutable borrow occurs here
   |  _________|
   | |
11 | |             Some(value) => value,
12 | |             None => {
13 | |                 map.insert(key.clone(), V::default());
   | |                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ second mutable borrow occurs here
14 | |                 map.get_mut(&key).unwrap()
15 | |             }
16 | |         }
   | |_________- returning this value requires that `*map` is borrowed for `'m`
```

分析代码可知在 match map.get_mut(&key) 方法调用完成后，对 map 的可变借用就可以结束了。但从报错看来，编译器不太聪明，它认为该借用会持续到整个 match 语句块的结束(第 16 行处)，这便造成了后续借用的失败。

。。match 的 语句 的生命周期 会持续到 match 结束。 所以导致 match 内部 不能再使用。
。。但是，为什么官方会让 生命周期 持续到match 结束？ 防止幻读( 是不可重复读。 幻读是增加一行， 不可重复读是 修改值)？


类似的例子还有很多，由于篇幅有限，就不在这里一一列举，如果大家想要阅读更多的类似代码，可以看看 Rust 代码鉴赏 一书。
https://github.com/sunface/rust-codes
。。里面没有这种 代码的。。


-------------------
#### 无界生命周期

不安全代码(unsafe)经常会凭空产生引用或生命周期，这些生命周期被称为是 无界(unbound) 的。

无界生命周期往往是在解引用一个裸指针(裸指针 raw pointer)时产生的，换句话说，它是凭空产生的，因为输入参数根本就没有这个生命周期：

```rust
fn f<'a, T>(x: *const T) -> &'a T {
    unsafe {
        &*x
    }
}
```

上述代码中，参数 x 是一个裸指针，它并没有任何生命周期，然后通过 unsafe 操作后，它被进行了解引用，变成了一个 Rust 的标准引用类型，该类型必须要有生命周期，也就是 'a。

可以看出 'a 是凭空产生的，因此它是无界生命周期。这种生命周期由于没有受到任何约束，因此它想要多大就多大，这实际上比 'static 要强大。例如 &'static &'a T 是无效类型，但是无界生命周期 &'unbounded &'a T 会被视为 &'a &'a T 从而通过编译检查

我们在实际应用中，要尽量避免这种无界生命周期。最简单的避免无界生命周期的方式就是在函数声明中运用生命周期消除规则。若一个输出生命周期被消除了，那么必定因为有一个输入生命周期与之对应。
。。就是手动指定 返回值的生命周期 应该和 哪个入参一致。

-------------
生命周期约束 HRTB

生命周期约束跟特征约束类似，都是通过形如 'a: 'b 的语法，来说明两个生命周期的长短关系。

> 'a: 'b

假设有两个引用 &'a i32 和 &'b i32，它们的生命周期分别是 'a 和 'b，若 'a >= 'b，则可以定义 'a:'b，表示 'a 至少要活得跟 'b 一样久。

```rust
struct DoubleRef<'a,'b:'a, T> {
    r: &'a T,
    s: &'b T
}
```
上述代码定义一个结构体，它拥有两个引用字段，类型都是泛型 T，每个引用都拥有自己的生命周期，由于我们使用了生命周期约束 'b: 'a，因此 'b 必须活得比 'a 久，也就是结构体中的 s 字段引用的值必须要比 r 字段引用的值活得要久。


> T: 'a

表示类型 T 必须比 'a 活得要久：
```rust
struct Ref<'a, T: 'a> {
    r: &'a T
}
```

。。生命周期 和生命周期比， 生命周期和 类型 比。。。。
。。好像有点点道理。  use std::collection::HashMap， 引入了 type， 这个type 必须比 上面的 Ref 结构 要久， 不然 就不知道 类型了。。
。。但是，你在 没有 HashMap 的地方 使用 HashMap， 编译也是失败吧。 编译的时候 肯定要知道 这个是什么类型啊。

因为结构体字段 r 引用了 T，因此 r 的生命周期 'a 必须要比 T 的生命周期更短(被引用者的生命周期必须要比引用长)。

在 Rust 1.30 版本之前，该写法是必须的，但是从 1.31 版本开始，编译器可以自动推导 T: 'a 类型的约束，因此我们只需这样写即可：
```rust
struct Ref<'a, T> {
    r: &'a T
}
```


使用了生命周期约束的综合例子：
```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a: 'b, 'b> ImportantExcerpt<'a> {
    fn announce_and_return_part(&'a self, announcement: &'b str) -> &'b str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```
上面的例子中必须添加约束 'a: 'b 后，才能成功编译，因为 self.part 的生命周期与 self的生命周期一致，将 &'a 类型的生命周期强行转换为 &'b 类型，会报错，只有在 'a >= 'b 的情况下，'a 才能转换成 'b。


#### 闭包函数的消除规则

```rust
fn fn_elision(x: &i32) -> &i32 { x }
let closure_slision = |x: &i32| -> &i32 { x };
```
```text
error: lifetime may not live long enough
  --> src/main.rs:39:39
   |
39 |     let closure = |x: &i32| -> &i32 { x }; // fails
   |                       -        -      ^ returning this value requires that `'1` must outlive `'2`
   |                       |        |
   |                       |        let's call the lifetime of this reference `'2`
   |                       let's call the lifetime of this reference `'1`
```

两个一模一样功能的函数，一个正常编译，一个却报错，
错误原因是编译器无法推测 返回的 引用 和传入的 引用 谁活得更久！

真的是非常奇怪的错误，学过上一节的读者应该都记得这样一条生命周期消除规则：如果函数参数中只有一个引用类型，那该引用的生命周期会被自动分配给所有的返回引用。我们当前的情况完美符合， function 函数的顺利编译通过，就充分说明了问题。

先给出一个结论：**这个问题，可能很难被解决，建议大家遇到后，还是老老实实用正常的函数，不要秀闭包了。**
。。不要用闭包
。。remember, no closure!


对于函数的生命周期而言，它的消除规则之所以能生效是因为它的生命周期完全体现在签名的引用类型上，在函数体中无需任何体现：

`fn fn_elision(x: &i32) -> &i32 {..}`
因此编译器可以做各种编译优化，也很容易根据参数和返回值进行生命周期的分析，最终得出消除规则。

可是闭包，并没有函数那么简单，它的生命周期分散在参数和闭包函数体中(主要是它没有确切的返回值签名)：
`let closure_slision = |x: &i32| -> &i32 { x };`

编译器就必须深入到闭包函数体中，去分析和推测生命周期，复杂度因此急剧提升：试想一下，编译器该如何从复杂的上下文中分析出参数引用的生命周期和闭包体中生命周期的关系？

。。？ 不清楚， rust的闭包规则， 上面的 闭包 是写了 返回值 &i32 的。 不过闭包应该确实可以省略。 C++都可以省略， C++。。 auto。。
。。但是为什么 不规定 : 闭包必须有返回值呢？
。。完全可以把 闭包 当做 方法处理啊。

由于上述原因(当然，实际情况复杂的多)，Rust 语言开发者目前其实是有意针对函数和闭包实现了==两种不同的生命周期消除规则==。

##### 用 Fn 特征解决闭包生命周期
之前我们提到了很难解决，但是并没有完全堵死(论文字的艺术- , -) 这不 @Ykong1337 同学就带了一个解决方法，为他点赞!
```rust
fn main() {
   let closure_slision = fun(|x: &i32| -> &i32 { x });
   assert_eq!(*closure_slision(&45), 45);
   // Passed !
}

fn fun<T, F: Fn(&T) -> &T>(f: F) -> F {
   f
}
```
。。？ 为了闭包， 写了一个方法？ 不过，可以通用的。 
。。 就是 显式声明 F 的类型是 一个方法，接收 &T，返回 &T 。 这样就把 生命周期 挂到 闭包上了。


--------------
#### NLL (Non-Lexical Lifetime)

引用的生命周期正常来说应该从借用开始一直持续到作用域结束，但是这种规则会让多引用共存的情况变得更复杂
从 1.31 版本引入 NLL 后，就变成了：==引用的生命周期从借用处开始，一直持续到最后一次使用的地方==。

。。rust 1.30 ， 2018-10的



------------
#### Reborrow再借用

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    fn move_to(&mut self, x: i32, y: i32) {
        self.x = x;
        self.y = y;
    }
}

fn main() {
    let mut p = Point { x: 0, y: 0 };
    let r = &mut p;
    let rr: &Point = &*r;

    println!("{:?}", rr);
    r.move_to(10, 10);
    println!("{:?}", r);
}
```
以上代码，大家可能会觉得可变引用 r 和不可变引用 rr 同时存在会报错吧？
但是事实上并不会，原因在于 rr 是对 r 的==再借用==。

对于再借用而言，rr 再借用时==不会破坏借用规则==，但是你==不能在它的生命周期内再使用原来的借用 r==，来看看对上段代码的分析：

```rust
fn main() {
    let mut p = Point { x: 0, y: 0 };
    let r = &mut p;
    // reborrow! 此时对`r`的再借用不会导致跟上面的借用冲突
    let rr: &Point = &*r;

    // 再借用`rr`最后一次使用发生在这里，在它的生命周期中，我们并没有使用原来的借用`r`，因此不会报错
    println!("{:?}", rr);

    // 再借用结束后，才去使用原来的借用`r`
    r.move_to(10, 10);
    println!("{:?}", r);
}
```

-----------

```rust
use std::vec::Vec;
fn read_length(strings: &mut Vec<String>) -> usize {
   strings.len()
}
```
函数体内对参数的二次借用也是典型的 Reborrow 场景。
。。？ 这个例子？ 不对吧。


----------
#### 生命周期消除规则补充
介绍几个常见的消除规则：

- impl 块消除
- 生命周期约束消除

----------
impl块消除
```rust
impl<'a> Reader for BufReader<'a> {
    // methods go here
    // impl内部实际上没有用到'a
}
```
如果你以前写的impl块长上面这样，同时在 impl 内部的方法中，根本就没有用到 'a，那就可以写成下面的代码形式。
```rust
impl Reader for BufReader<'_> {
    // methods go here
}
```
'_ 生命周期表示 BufReader 有一个==不使用的生命周期==，我们可以忽略它，无需为它创建一个名称。

生命周期参数也是类型的一部分，因此 BufReader<'a> 是一个完整的类型，在实现它的时候，你不能把 'a 给丢了！

-------------
生命周期约束消除
```rust
// Rust 2015
struct Ref<'a, T: 'a> {
    field: &'a T
}

// Rust 2018
struct Ref<'a, T> {
    field: &'a T
}
```

------------
例子

```rust
struct Interface<'a> {
    manager: &'a mut Manager<'a>
}

impl<'a> Interface<'a> {
    pub fn noop(self) {
        println!("interface consumed");
    }
}

struct Manager<'a> {
    text: &'a str
}

struct List<'a> {
    manager: Manager<'a>,
}

impl<'a> List<'a> {
    pub fn get_interface(&'a mut self) -> Interface {
        Interface {
            manager: &mut self.manager
        }
    }
}

fn main() {
    let mut list = List {
        manager: Manager {
            text: "hello"
        }
    };

    list.get_interface().noop();

    println!("Interface should be dropped here and the borrow released");

    // 下面的调用会失败，因为同时有不可变/可变借用
    // 但是Interface在之前调用完成后就应该被释放了
    use_list(&list);
}

fn use_list(list: &List) {
    println!("{}", list.manager.text);
}
```

```text
error[E0502]: cannot borrow `list` as immutable because it is also borrowed as mutable // `list`无法被借用，因为已经被可变借用
  --> src/main.rs:40:14
   |
34 |     list.get_interface().noop();
   |     ---- mutable borrow occurs here // 可变借用发生在这里
...
40 |     use_list(&list);
   |              ^^^^^
   |              |
   |              immutable borrow occurs here // 新的不可变借用发生在这
   |              mutable borrow later used here // 可变借用在这里结束
```

这是因为我们在 get_interface 方法中声明的 lifetime 有问题，该方法的参数的生命周期是 'a，而 List 的生命周期也是 'a，说明该方法至少活得跟 List 一样久，再回到 main 函数中，list 可以活到 main 函数的结束，因此 list.get_interface() 借用的可变引用也会活到 main 函数的结束，在此期间，自然无法再进行借用了。

要解决这个问题，我们需要为 get_interface 方法的参数给予一个不同于 List<'a> 的生命周期 'b
```rust
impl<'a> List<'a> {
    pub fn get_interface<'b>(&'b mut self) -> Interface<'b, 'a>
    where 'a: 'b {
        Interface {
            manager: &mut self.manager
        }
    }
}
```

-----------

#### &'static, T:'static

'static 在 Rust 中是相当常见的，例如字符串字面值就具有 'static 生命周期:
```rust
fn main() {
  let mark_twain: &str = "Samuel Clemens";
  print_author(mark_twain);
}
fn print_author(author: &'static str) {
  println!("{}", author);
}
```
特征对象的生命周期也是 'static


除了 &'static 的用法外，我们在另外一种场景中也可以见到 'static 的使用:
```rust
use std::fmt::Display;
fn main() {
    let mark_twain = "Samuel Clemens";
    print(&mark_twain);
}

fn print<T: Display + 'static>(message: &T) {
    println!("{}", message);
}
```
在这里，很明显 'static 是作为生命周期约束来使用了。 那么问题来了， &'static 和 T: 'static 的用法到底有何区别？



##### &'static

&'static 对于生命周期有着非常强的要求：一个引用必须要活得跟剩下的程序一样久，才能被标注为 &'static。

对于字符串字面量来说，它直接被打包到二进制文件中，永远不会被 drop，因此它能跟程序活得一样久，自然它的生命周期是 'static。

但是，&'static 生命周期==针对的仅仅是引用==，而不是持有该引用的变量，对于变量来说，还是要遵循相应的作用域规则 :
```rust
use std::{slice::from_raw_parts, str::from_utf8_unchecked};

fn get_memory_location() -> (usize, usize) {
  // “Hello World” 是字符串字面量，因此它的生命周期是 `'static`.
  // 但持有它的变量 `string` 的生命周期就不一样了，它完全取决于变量作用域，对于该例子来说，也就是当前的函数范围
  let string = "Hello World!";
  let pointer = string.as_ptr() as usize;
  let length = string.len();
  (pointer, length)
  // `string` 在这里被 drop 释放
  // 虽然变量被释放，无法再被访问，但是数据依然还会继续存活
}

fn get_str_at_location(pointer: usize, length: usize) -> &'static str {
  // 使用裸指针需要 `unsafe{}` 语句块
  unsafe { from_utf8_unchecked(from_raw_parts(pointer as *const u8, length)) }
}

fn main() {
  let (pointer, length) = get_memory_location();
  let message = get_str_at_location(pointer, length);
  println!(
    "The {} bytes at 0x{:X} stored: {}",
    length, pointer, message
  );
  // 如果大家想知道为何处理裸指针需要 `unsafe`，可以试着反注释以下代码
  // let message = get_str_at_location(1000, 10);
}
```
- &'static 的引用确实可以和程序活得一样久，因为我们通过 get_str_at_location 函数直接取到了对应的字符串
- 持有 &'static 引用的变量，它的生命周期受到作用域的限制，大家务必不要搞混了


##### T: 'static
这种形式的约束就有些复杂了。

首先，在以下两种情况下，T: 'static 与 &'static 有相同的约束：T 必须活得和程序一样久。

```rust
use std::fmt::Debug;

fn print_it<T: Debug + 'static>( input: T) {
    println!( "'static value passed in is: {:?}", input );
}

fn print_it1( input: impl Debug + 'static ) {
    println!( "'static value passed in is: {:?}", input );
}

fn main() {
    let i = 5;

    print_it(&i);
    print_it1(&i);
}
```
以上代码会报错，原因很简单: &i 的生命周期无法满足 'static 的约束，如果大家将 i 修改为常量，那自然一切 OK。


```rust
use std::fmt::Debug;

fn print_it<T: Debug + 'static>( input: &T) {
    println!( "'static value passed in is: {:?}", input );
}

fn main() {
    let i = 5;

    print_it(&i);
}
```
这段代码竟然不报错了！原因在于我们约束的是 T，但是使用的却是它的引用 &T，换而言之，我们根本没有直接使用 T，因此编译器就没有去检查 T 的生命周期约束！它只要确保 &T 的生命周期符合规则即可，在上面代码中，它自然是符合的。


------------

```rust
use std::fmt::Display;

fn main() {
  let r1;
  let r2;
  {
    static STATIC_EXAMPLE: i32 = 42;
    r1 = &STATIC_EXAMPLE;
    let x = "&'static str";
    r2 = x;
    // r1 和 r2 持有的数据都是 'static 的，因此在花括号结束后，并不会被释放
  }

// 用C++试了下，{} 里面的复制，应该是 复制构造，所以不会有问题。
// 用指针的话，可以看到，{} 内 和外， 地址值 是一样的， 但是 里面能 *p 打印出值， 外门 *p 打印空白。

  println!("&'static i32: {}", r1); // -> 42
  println!("&'static str: {}", r2); // -> &'static str

  let r3: &str;

  {
    let s1 = "String".to_string();

    // s1 虽然没有 'static 生命周期，但是它依然可以满足 T: 'static 的约束
    // 充分说明这个约束是多么的弱。。
    static_bound(&s1);

    // s1 是 String 类型，没有 'static 的生命周期，因此下面代码会报错
    r3 = &s1;

    // s1 在这里被 drop
  }
  println!("{}", r3);
}

fn static_bound<T: Display + 'static>(t: &T) {
  println!("{}", t);
}
```


---------
> static 到底针对谁？

大家有没有想过，到底是 &'static 这个引用还是该引用指向的数据活得跟程序一样久呢？

答案是引用指向的数据，而引用本身是要遵循其作用域范围的，我们来简单验证下：
```rust
fn main() {
    {
        let static_string = "I'm in read-only memory";
        println!("static_string: {}", static_string);

        // 当 `static_string` 超出作用域时，该引用不能再被使用，但是数据依然会存在于 binary 所占用的内存中
    }

    println!("static_string reference remains alive: {}", static_string);
}
```

作为经验之谈，可以这么来:
- 如果你需要添加 &'static 来让代码工作，那很可能是设计上出问题了
- 如果你希望满足和取悦编译器，那就使用 T: 'static，很多时候它都能解决问题



### 函数式编程：闭包，迭代器

- 使用函数作为参数进行传递
- 使用函数作为函数返回值
- 将函数赋值给变量


这里主要关注的是函数式特性：
- 闭包 Closure
- 迭代器 Iterator
- 模式匹配
- 枚举

这些函数式特性可以让代码的可读性和易写性大幅提升


#### 闭包

闭包是一种匿名函数，它可以赋值给变量也可以作为参数传递给其它函数，不同于函数的是，它允许捕获调用者作用域中的值

```rust
fn main() {
   let x = 1;
   let sum = |y| x + y;

    assert_eq!(3, sum(2));
}
```

闭包 sum，它拥有一个入参 y，同时捕获了作用域中的 x 的值，因此调用 sum(2) 意味着将 2（参数 y）跟 1（x）进行相加,最终返回它们的和：3。

sum 非常符合闭包的定义：可以赋值给变量，允许捕获调用者作用域中的值。


```rust
fn workout(intensity: u32, random_number: u32) {
    let action = || {
        println!("muuuu.....");
        thread::sleep(Duration::from_secs(2));
        intensity
    };
```

闭包的形式定义：
```rust
|param1, param2,...| {
    语句1;
    语句2;
    返回表达式
}
```

`|param1| 返回表达式`

----------

函数往往会作为 API 提供给你的用户，因此你的用户必须在使用时知道传入参数的类型和返回值类型。必须手动为函数的所有参数和返回值指定类型

闭包并不会作为 API 对外提供，因此它可以享受编译器的类型推导能力，无需标注参数和返回值的类型。

```rust
let sum = |x: i32, y: i32| -> i32 {
    x + y
}

let sum = |x, y| x + y
```

同一个功能的函数和闭包实现形式：
```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

类型推导很好用，但是它不是泛型，当编译器推导出一种类型后，它就会一直使用该类型
```rust
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5);
```
在 s 中，编译器为 x 推导出类型 String，但是紧接着 n 试图用 5 这个整型去调用闭包，跟编译器之前推导的 String 类型不符，因此报错


---------
> 结构体中的闭包

我们要实现一个简易缓存，功能是获取一个值，然后将其缓存起来，那么可以这样设计：
- 一个闭包用于获取值
- 一个变量，用于存储该值

可以使用结构体来代表缓存对象，最终设计如下：
```rust
struct Cacher<T>
where
    T: Fn(u32) -> u32,
{
    query: T,
    value: Option<u32>,
}
```

`Fn(u32) -> u32` 是一个特征，用来表示 T 是一个闭包类型


```rust
impl<T> Cacher<T>
where
    T: Fn(u32) -> u32,
{
    fn new(query: T) -> Cacher<T> {
        Cacher {
            query,
            value: None,
        }
    }

    // 先查询缓存值 `self.value`，若不存在，则调用 `query` 加载
    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.query)(arg);
                self.value = Some(v);
                v
            }
        }
    }
}
```


---------
> 闭包对内存的影响

当闭包从环境中捕获一个值时，会分配内存去存储这些值。
对于有些场景来说，这种额外的内存分配会成为一种负担。
与之相比，函数就不会去捕获这些环境值，因此定义和使用函数不会拥有这种内存负担。


#### 三种 Fn 特征

闭包捕获变量有三种途径，恰好对应函数参数的三种传入方式：==转移所有权、可变借用、不可变借用==，因此相应的 Fn 特征也有三种：

FnOnce
FnMut
Fn

##### FnOnce
- FnOnce，该类型的闭包会拿走被捕获变量的所有权。Once 顾名思义，说明该闭包只能运行一次
```rust
fn fn_once<F>(func: F)
where
    F: FnOnce(usize) -> bool,
{
    println!("{}", func(3));
    println!("{}", func(4));
}

fn main() {
    let x = vec![1, 2, 3];
    fn_once(|z|{z == x.len()})
}
```

```text
error[E0382]: use of moved value: `func`
 --> src\main.rs:6:20
  |
1 | fn fn_once<F>(func: F)
  |               ---- move occurs because `func` has type `F`, which does not implement the `Copy` trait
                  // 因为`func`的类型是没有实现`Copy`特性的 `F`，所以发生了所有权的转移
...
5 |     println!("{}", func(3));
  |                    ------- `func` moved due to this call // 转移在这
6 |     println!("{}", func(4));
  |                    ^^^^ value used here after move // 转移后再次用
  |
```

这里面有一个很重要的提示，因为 F 没有实现 Copy 特征，所以会报错，那么我们添加一个约束，试试实现了 Copy 的闭包：
```rust
fn fn_once<F>(func: F)
where
    F: FnOnce(usize) -> bool + Copy,// 改动在这里
{
    println!("{}", func(3));
    println!("{}", func(4));
}

fn main() {
    let x = vec![1, 2, 3];
    fn_once(|z|{z == x.len()})
}
```
上面代码中，func 的类型 F 实现了 Copy 特征，调用时使用的将是它的拷贝，所以并没有发生所有权的转移。

。。所以之前是把 lambda 的 所有权转出去了。 不是。
。。 lambda 复制了一份。。。。
。。
。。 上面说明了 FnOnce 会获得 被捕获变量的所有权
。。 但是为什么， FnOnce 加上Copy 就可以 执行2次？  x 变量 被捕获 2次了啊。并且每次都是 获得了所有权的啊。
。。 而且下面就是 move。 和 FnOnce 有点 重复了。 不，没有重复， FnOnce 是类型， 下面的move 直接是 lambda
。。 看 上面的报错， 写清了 func 被 move 了2次。 
。。 所以 fnOnce 是有 2个 概念： 1是 它会获得 捕获变量的 所有权， 2是 它自己的所有权会发生移动 ？ 。。 但是 1 的话， 为什么 Copy 后， 可以 捕获 2次？


##### move
如果你想强制闭包==取得捕获变量的所有权==，可以在参数列表前添加 ==move== 关键字，这种用法通常用于闭包的生命周期大于捕获变量的生命周期时，例如将闭包返回或移入其他线程。
```rust
use std::thread;
let v = vec![1, 2, 3];
let handle = thread::spawn(move || {
    println!("Here's a vector: {:?}", v);
});
handle.join().unwrap();
```


##### FnMut
以可变借用的方式捕获了环境中的值，因此可以修改该值：

```rust
fn main() {
    let mut s = String::new();

    let mut update_string =  |str| s.push_str(str); // let mut. 不然异常
    update_string("hello");

    println!("{:?}",s);
}
```



##### Fn
以不可变借用的方式捕获环境中的值 

```rust
fn main() {
    let s = "hello, ".to_string();

    let update_string =  |str| println!("{},{}",s,str);     // s 必须不能 mut， s实现mut就变成了 FnMut

    exec(update_string);

    println!("{:?}",s);
}

fn exec<'a, F: Fn(String) -> ()>(f: F)  {
    f("world".to_string())
}
```

-------------

##### move 和 Fn

使用了 move 的闭包依然可能实现了 Fn 或 FnMut 特征。
一个闭包实现了哪种 Fn 特征取决于该闭包如何使用被捕获的变量，而不是取决于闭包如何捕获它们。move 本身强调的就是后者，闭包如何捕获变量：

```rust
fn main() {
    let s = String::new();

    let update_string =  move || println!("{}",s);

    exec(update_string);
}

fn exec<F: FnOnce()>(f: F)  {
    f()
}
```

---------------

三种 Fn 的关系

实际上，一个闭包并不仅仅实现某一种 Fn 特征，规则如下：
- 所有的闭包都自动实现了 FnOnce 特征，因此任何一个闭包都至少可以被调用一次
- 没有移出所捕获变量的所有权的闭包自动实现了 FnMut 特征
- 不需要对捕获变量进行改变的闭包自动实现了 Fn 特征


#### 闭包作为函数返回值

```rust
fn factory() -> Fn(i32) -> i32 {
    let num = 5;

    |x| x + num
}

let f = factory();

let answer = f(1);
assert_eq!(6, answer);
```
Rust 要求函数的参数和返回类型，必须有固定的内存大小
绝大部分类型都有固定的大小，但是不包括特征，因为特征类似接口，对于编译器来说，无法知道它后面藏的真实类型是什么，因为也无法得知具体的大小。

```text
help: use `impl Fn(i32) -> i32` as the return type, as all return paths are of type `[closure@src/main.rs:11:5: 11:21]`, which implements `Fn(i32) -> i32`
  |
8 | fn factory<T>() -> impl Fn(i32) -> i32 {
```

impl Trait 可以用来返回一个实现了指定特征的类型，那么这里 impl Fn(i32) -> i32 的返回值形式，说明我们要返回一个闭包类型，它实现了 Fn(i32) -> i32 特征。

但是，在特征那一章，我们提到过，impl Trait 的返回方式有一个非常大的局限，就是你只能返回同样的类型，例如：

```rust
fn factory(x:i32) -> impl Fn(i32) -> i32 {

    let num = 5;

    if x > 1{
        move |x| x + num
    } else {
        move |x| x - num
    }
}
```

if 和 else 分支中返回了不同的闭包类型，这就很奇怪了，明明这两个闭包长的一样的，
好在细心的读者应该回想起来，本章节前面咱们有提到：就算签名一样的闭包，类型也是不同的，
因此在这种情况下，就无法再使用 impl Trait 的方式去返回闭包。

`= help: consider boxing your closure and/or using it as a trait object`

可以用特征对象！只需要用 Box 的方式即可实现：
```rust
fn factory(x:i32) -> Box<dyn Fn(i32) -> i32> {
    let num = 5;

    if x > 1{
        Box::new(move |x| x + num)
    } else {
        Box::new(move |x| x - num)
    }
}
```





#### 迭代器

```rust
let arr = [1, 2, 3];
for v in arr {
    println!("{}",v);
}


for i in 1..10 {
    println!("{}", i);
}
```

可以显式的把数组转换成迭代器：
```rust
let arr = [1, 2, 3];
for v in arr.into_iter() {
    println!("{}", v);
}
```

-------

lazy init

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();        // 不会发生任何迭代行为

for val in v1_iter {
    println!("{}", val);
}
```

---------
next

迭代器之所以成为迭代器，就是因为实现了 Iterator 特征，
要实现该特征，最主要的就是实现其中的 next 方法，该方法控制如何从集合中取值，最终返回值的类型是关联类型 Item。

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // 省略其余有默认实现的方法
}
```

```rust
fn main() {
    let arr = [1, 2, 3];
    let mut arr_iter = arr.into_iter();

    assert_eq!(arr_iter.next(), Some(1));
    assert_eq!(arr_iter.next(), Some(2));
    assert_eq!(arr_iter.next(), Some(3));
    assert_eq!(arr_iter.next(), None);
}
```

next 方法返回的是 Option 类型
手动迭代必须将迭代器声明为 mut 可变，因为调用 next 会改变迭代器其中的状态数据


--------------
IntoIterator 特征

迭代器自身也实现了 IntoIterator
```rust
impl<I: Iterator> IntoIterator for I {
    type Item = I::Item;
    type IntoIter = I;

    #[inline]
    fn into_iter(self) -> I {
        self
    }
}
```
可以写出：
```rust
fn main() {
    let values = vec![1, 2, 3];

    for v in values.into_iter().into_iter().into_iter() {
        println!("{}",v)
    }
}
```

##### into_iter, iter, iter_mut

- into_iter 会夺走所有权
- iter 是借用
- iter_mut 是可变借用

```rust
fn main() {
    let values = vec![1, 2, 3];

    for v in values.into_iter() {
        println!("{}", v)
    }

    // 下面的代码将报错，因为 values 的所有权在上面 `for` 循环中已经被转移走
    // println!("{:?}",values);

    let values = vec![1, 2, 3];
    let _values_iter = values.iter();

    // 不会报错，因为 values_iter 只是借用了 values 中的元素
    println!("{:?}", values);

    let mut values = vec![1, 2, 3];
    // 对 values 中的元素进行可变借用
    let mut values_iter_mut = values.iter_mut();

    // 取出第一个元素，并修改为0
    if let Some(v) = values_iter_mut.next() {
        *v = 0;
    }

    // 输出[0, 2, 3]
    println!("{:?}", values);
}
```

- .iter() 方法实现的迭代器，调用 next 方法返回的类型是 Some(&T)
- .iter_mut() 方法实现的迭代器，调用 next 方法返回的类型是 Some(&mut T)，因此在 if let Some(v) = values_iter_mut.next() 中，v 的类型是 &mut i32，最终我们可以通过 *v = 0 的方式修改其值


----------
##### Iterator 和 IntoIterator 的区别

这两个其实还蛮容易搞混的，但我们只需要记住，
- Iterator 就是==迭代器特征==，只有实现了它才能称为迭代器，才能调用 next。
- IntoIterator 强调的是某一个类型如果实现了该特征，它可以通过 ==into_iter，iter 等方法==变成一个迭代器。



##### ==迭代器的消费者==

- 消费者适配器

比如sum，拿走迭代器的`所有权`，然后通过不断调用 next 方法对里面的元素进行求和

```rust
fn main() {
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    let total: i32 = v1_iter.sum();       // <<<<<<<

    assert_eq!(total, 6);

    // v1_iter 是借用了 v1，因此 v1 可以照常使用
    println!("{:?}",v1);

    // 以下代码会报错，因为 `sum` 拿到了迭代器 `v1_iter` 的所有权
    // println!("{:?}",v1_iter);
}
```

------------

- 迭代器适配器

既然消费者适配器是消费掉迭代器，然后返回一个值。那么迭代器适配器，顾名思义，会`返回一个新的迭代器`，这是实现链式方法调用的关键：v.iter().map().filter()...。

与消费者适配器不同，迭代器适配器是惰性的，意味着你==需要一个消费者适配器来收尾==，最终将迭代器转换成一个具体的值：

。。java的 stream。。

```rust
let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
```

collect 方法，该方法就是一个消费者适配器，使用它可以将一个迭代器中的元素收集到指定类型中，
这里我们为 v2 标注了 Vec<_> 类型，就是为了告诉 collect：请把迭代器中的元素消费掉，然后把值收集成 Vec<_> 类型，至于为何使用 _，因为编译器会帮我们自动推导。

map 会对迭代器中的每一个值进行一系列操作，然后把该值转换成另外一个新值，该操作是通过闭包 |x| x + 1 来完成：最终迭代器中的每个值都增加了 1，从 [1, 2, 3] 变为 [2, 3, 4]。


```rust
use std::collections::HashMap;
fn main() {
    let names = ["sunface", "sunfei"];
    let ages = [18, 18];
    let folks: HashMap<_, _> = names.into_iter().zip(ages.into_iter()).collect();

    println!("{:?}",folks);
}
```

zip 是一个迭代器适配器，它的作用就是将两个迭代器的内容压缩到一起，形成 `Iterator<Item=(ValueFromA, ValueFromB)>` 这样的新的迭代器，在此处就是形如 [(name1, age1), (name2, age2)] 的迭代器。


-----------
闭包作为适配器参数

##### 。。跳


### 深入类型

#### 类型转换

##### as 转换

`i8::MAX`

```rust
fn main() {
   let a = 3.1 as i8;
   let b = 100_i8 as i32;
   let c = 'a' as u8; // 将字符'a'转换为整数，97

   println!("{},{},{}",a,b,c)
}
```

内存地址转为指针
```rust
let mut values: [i32; 2] = [1, 2];
let p1: *mut i32 = values.as_mut_ptr();
let first_address = p1 as usize; // 将p1内存地址转换为一个整数
let second_address = first_address + 4; // 4 == std::mem::size_of::<i32>()，i32类型占用4个字节，因此将内存地址 + 4
let p2 = second_address as *mut i32; // 访问该地址指向的下一个整数p2
unsafe {
    *p2 += 1;
}
assert_eq!(values[1], 3);
```


##### TryInto 转换

一些场景中，使用 as 关键字会有比较大的限制

```rust
use std::convert::TryInto;          // 可以删除，rust默认导入的。

fn main() {
   let a: u8 = 10;
   let b: u16 = 1500;

   let b_: u8 = b.try_into().unwrap();

   if a < b_ {
     println!("Ten is less than one hundred.");
   }
}
```

引入了 std::convert::TryInto 特征，但是却没有使用它

原因在于如果你要使用一个特征的方法，那么你需要引入该特征到当前的作用域中，我们在上面用到了 try_into 方法，因此需要引入对应的特征。


> try_into 转换会捕获大类型向小类型转换时导致的溢出错误

```rust
fn main() {
    let b: i16 = 1500;

    let b_: u8 = match b.try_into() {
        Ok(b1) => b1,
        Err(e) => {
            println!("{:?}", e.to_string());
            0
        }
    };
}
```


##### 通用类型转换，(as,TryInto只能应用到数值类型)

强制类型转换

在某些情况下，类型是可以进行==隐式强制转换==的，虽然这些转换弱化了 Rust 的类型系统，但是它们的存在是为了让 Rust 在大多数场景可以工作(说白了，帮助用户省事)，而不是报各种类型上的编译错误。

首先，在匹配特征时，不会做任何强制转换(除了方法)。一个类型 T 可以强制转换为 U，不代表 impl T 可以强制转换为 impl U，例如下面的代码就无法通过编译检查：
```rust
trait Trait {}

fn foo<X: Trait>(t: X) {}

impl<'a> Trait for &'a i32 {}

fn main() {
    let t: &mut i32 = &mut 0;
    foo(t);
}
```
the trait `Trait` is not implemented for `&mut i32`

&i32 实现了特征 Trait， &mut i32 可以转换为 &i32，但是 &mut i32 依然无法作为 Trait 来使用。


-------------
点操作符

方法调用的 点操作符，会发生类型转换，比如：自动引用、自动解引用，强制类型转换直到类型能匹配等。


##### 。。跳。

#### newtype

简单来说，就是使用元组结构体的方式将已有的类型包裹起来：==struct Meters(u32);==，那么此处 Meters 就是一个 newtype。

- 自定义类型可以让我们给出更有意义和可读性的类型名，例如与其使用 u32 作为距离的单位类型，我们可以使用 Meters，它的可读性要好得多
- 对于某些场景，只有 newtype 可以很好地解决
- 隐藏内部类型的细节


第二点的场景： 为外部类型实现外部特征

如果在外部类型上实现外部特征必须使用 newtype 的方式，否则你就得遵循孤儿规则：要为类型 A 实现特征 T，那么 A 或者 T 必须至少有一个在当前的作用范围内。

如果想使用 println!("{}", v) 的方式去格式化输出一个动态数组 Vec，以期给用户提供更加清晰可读的内容，那么就需要为 Vec 实现 Display 特征，但是这里有一个问题： Vec 类型定义在标准库中，Display 亦然，这时就可以祭出大杀器 newtype 来解决：

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```



#### 类型别名。。跳

`type Meters = u32`

- 类型别名仅仅是别名，只是为了让可读性更好，并不是全新的类型，newtype 才是！
- 类型别名无法实现为外部类型实现外部特征等功能，而 newtype 可以


#### 。。跳

### 智能指针。。跳

智能指针往往是基于结构体实现，它与我们自定义的结构体最大的区别在于它实现了 Deref 和 Drop 特征：
- Deref 可以让智能指针像引用那样工作，这样你就可以写出同时支持智能指针和引用的代码，例如 *T
- Drop 允许你指定智能指针超出作用域后自动执行的代码，例如做一些数据清除等收尾工作

智能指针在 Rust 中很常见，我们在本章不会全部讲解，而是挑选几个最常用、最有代表性的进行讲解：
- Box<T>，可以将值分配到堆上
- Rc<T>，引用计数类型，允许多所有权存在
- Ref<T> 和 RefMut<T>，允许将借用规则检查从编译期移动到运行期进行


### 循环引用与自引用。。跳

weak

### ==多线程并发编程==

不同语言对于线程的实现可能大相径庭：
- 由于操作系统提供了创建线程的 API，因此部分语言会直接调用该 API 来创建线程，因此最终程序内的线程数和该程序占用的操作系统线程数相等，一般称之为1:1 线程模型，例如 ==Rust==。
- 还有些语言在内部实现了自己的线程模型（绿色线程、协程），程序内部的 M 个线程最后会以某种映射方式使用 N 个操作系统线程去运行，因此称之为M:N 线程模型，其中 M 和 N 并没有特定的彼此限制关系。一个典型的代表就是 Go 语言。
- 还有些语言使用了 Actor 模型，基于消息传递进行并发，例如 Erlang 语言。

绿色线程/协程的实现会显著增大运行时的大小，因此 Rust 只在标准库中提供了 1:1 的线程模型，
如果你愿意牺牲一些性能来换取更精确的线程控制以及更小的线程上下文切换成本，那么可以选择 Rust 中的 M:N 模型，这些模型由三方库提供了实现，例如大名鼎鼎的 tokio。

------------
由于多线程的代码是同时运行的，因此我们无法保证线程间的执行顺序，这会导致一些问题：
- 竞态条件(race conditions)，多个线程以非一致性的顺序同时访问数据资源
- 死锁(deadlocks)，两个线程都想使用某个资源，但是又都在等待对方释放资源后才能使用，结果最终都无法继续执行
- 一些因为多线程导致的很隐晦的 BUG，难以复现和解决

#### 创建线程 thread::spawn

```rust
use std::thread;
use std::time::Dueation;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("number {} from thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("number {} from main!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

- 线程内部的代码使用闭包来执行
- main 线程一旦结束，程序就立刻结束，因此需要保持它的存活，直到其它子线程完成自己的任务
- thread::sleep 会让当前线程休眠指定的时间，随后其它线程会被调度运行（上一节并发与并行中有简单介绍过），因此就算你的电脑只有一个 CPU 核心，该程序也会表现的如同多 CPU 核心一般，这就是并发！


-----------------
等待子线程结束

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..5 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```


---------
线程闭包中使用move

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```
线程的启动时间点和结束时间点是不确定的，因此存在一种可能，当主线程执行完， v 被释放掉时，新的线程很可能还没有结束甚至还没有被创建成功，此时新线程对 v 的引用立刻就不再合法！

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();

    // 下面代码会报错borrow of moved value: `v`
    // println!("{:?}",v);
}
```

在系统编程中，操作系统提供了直接杀死线程的接口，简单粗暴，
但是 Rust 并没有提供这样的接口，
原因在于，粗暴地终止一个线程可能会导致资源没有释放、状态混乱等不可预期的结果，
一向以安全自称的 Rust，自然不会砸自己的饭碗。

那么 Rust 中线程是如何结束的呢？答案很简单：线程的代码执行完，线程就会自动结束。但是如果线程中的代码不会执行完呢？那么情况可以分为两种进行讨论：
- 线程的任务是一个循环 IO 读取，任务流程类似：IO 阻塞，等待读取新的数据 -> 读到数据，处理完成 -> 继续阻塞等待 ··· -> 收到 socket 关闭的信号 -> 结束线程，在此过程中，绝大部分时间线程都处于阻塞的状态，因此虽然看上去是循环，CPU 占用其实很小，也是网络服务中最最常见的模型
- 线程的任务是一个循环，里面没有任何阻塞，包括休眠这种操作也没有，此时 CPU 很不幸的会被跑满，而且你如果没有设置终止条件，该线程将持续跑满一个 CPU 核心，并且不会被终止，直到 main 线程的结束

模拟第二种情况
```rust
use std::thread;
use std::time::Duration;
fn main() {
    // 创建一个线程A
    let new_thread = thread::spawn(move || {
        // 再创建一个线程B
        thread::spawn(move || {
            loop {
                println!("I am a new thread.");
            }
        })
    });

    // 等待新创建的线程执行完成
    new_thread.join().unwrap();
    println!("Child thread is finish!");

    // 睡眠一段时间，看子线程创建的子线程是否还在运行
    thread::sleep(Duration::from_millis(100));
}
```
main 线程创建了一个新的线程 A，同时该新线程又创建了一个新的线程 B，可以看到 A 线程在创建完 B 线程后就立即结束了，而 B 线程则在不停地循环输出。


#### 多线程性能

据不精确估算，创建一个线程大概需要 0.24 毫秒，随着线程的变多，这个值会变得更大


当任务是 CPU 密集型时，让线程数等于 CPU 核心数是最好的。

当你的任务大部分时间都处于阻塞状态时，就可以考虑增多线程数量，这样当某个线程处于阻塞状态时，会被切走，进而运行其它的线程，典型就是网络 IO 操作

事实上，对于这种网络 IO 情况，一般都不再使用多线程的方式了，毕竟操作系统的线程数是有限的，意味着并发数也很容易达到上限，而且过多的线程也会导致线程上下文切换的代价过大，使用 async/await 的 M:N 并发模型，就没有这个烦恼。


----------
一个无锁实现(CAS)的 Hashmap 在多线程下的使用：

```rust
for i in 0..num_threads {
    let ht = Arc::clone(&ht);

    let handle = thread::spawn(move || {
        for j in 0..adds_per_thread {
            let key = thread_rng().gen::<u32>();
            let value = thread_rng().gen::<u32>();
            ht.set_item(key, value);
        }
    });

    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}
```

既然是无锁实现了，那么锁的开销应该几乎没有，性能会随着线程数的增加接近线性增长，但是真的是这样吗？

下图是该代码在 48 核机器上的运行结果：

。略，反正就是 吞吐量上升，然后下降。 

从图上可以明显的看出：吞吐并不是线性增长，尤其从 16 核开始，甚至开始肉眼可见的下降，这是为什么呢？
大概的原因：
- 虽然是无锁，但是内部是 CAS 实现，大量线程的同时访问，会让 CAS 重试次数大幅增加
- 线程过多时，CPU 缓存的命中率会显著下降，同时多个线程竞争一个 CPU Cache-line 的情况也会经常发生
- 大量读写可能会让内存带宽也成为瓶颈
- 读和写不一样，无锁数据结构的读往往可以很好地线性增长，但是写不行，因为写竞争太大

多线程的开销往往是在锁、数据竞争、缓存失效上，这些限制了现代化软件系统随着 CPU 核心的增多性能也线性增加的野心。


#### 线程屏障(Barrier)

在 Rust 中，可以使用 Barrier 让多个线程都执行到某个点后，才继续一起往后执行：

```rust
use std::sync::{Arc, Barrier};
use std::thread;

fn main() {
    let mut handles = Vec::with_capacity(6);
    let barrier = Arc::new(Barrier::new(6));

    for _ in 0..6 {
        let b = barrier.clone();
        handles.push(thread::spawn(move|| {
            println!("before wait");
            b.wait();
            println!("after wait");
        }));
    }

    for handle in handles {
        handle.join().unwrap();
    }
}
```


#### 线程局部变量(Thread Local Variable)

线程局部变量在一些场景下非常有用，而 Rust 通过标准库和三方库对此进行了支持。

标准库 thread_local

使用 thread_local 宏可以初始化线程局部变量，然后在线程内部使用该变量的 with 方法获取变量值：

```rust
use std::cell::RefCell;
use std::thread;

thread_local!(static FOO: RefCell<u32> = RefCell::new(1));

FOO.with(|f| {
    assert_eq!(*f.borrow(), 1);
    *f.borrow_mut() = 2;
});

// 每个线程开始时都会拿到线程局部变量的FOO的初始值
let t = thread::spawn(move|| {
    FOO.with(|f| {                      // FOO.with
        assert_eq!(*f.borrow(), 1);
        *f.borrow_mut() = 3;
    });
});

// 等待线程完成
t.join().unwrap();

// 尽管子线程中修改为了3，我们在这里依然拥有main线程中的局部值：2
FOO.with(|f| {
    assert_eq!(*f.borrow(), 2);
});
```

FOO 即是我们创建的线程局部变量，每个新的线程访问它时，都会使用它的初始值作为开始，各个线程中的 FOO 值彼此互不干扰。
注意 FOO 使用 static 声明为生命周期为 'static 的静态变量。

线程中对 FOO 的使用是通过借用的方式，但是若我们需要每个线程独自获取它的拷贝，最后进行汇总，就有些强人所难了。


在结构体中使用线程局部变量：
```rust
use std::cell::RefCell;

struct Foo;
impl Foo {
    thread_local! {
        static FOO: RefCell<usize> = RefCell::new(0);
    }
}

fn main() {
    Foo::FOO.with(|x| println!("{:?}", x));
}
```

通过引用的方式使用它:
```rust
use std::cell::RefCell;
use std::thread::LocalKey;

thread_local! {
    static FOO: RefCell<usize> = RefCell::new(0);
}
struct Bar {
    foo: &'static LocalKey<RefCell<usize>>,
}
impl Bar {
    fn constructor() -> Self {
        Self {
            foo: &FOO,
        }
    }
}
```


三方库 thread-local
https://github.com/Amanieu/thread_local-rs

。。Arc的全名就是:Atomically Reference Counted,即原子引用计数。

```rust
use thread_local::ThreadLocal;
use std::sync::Arc;
use std::cell::Cell;
use std::thread;

let tls = Arc::new(ThreadLocal::new());

// 创建多个线程
for _ in 0..5 {
    let tls2 = tls.clone();
    thread::spawn(move || {
        // 将计数器加1
        let cell = tls2.get_or(|| Cell::new(0));
        cell.set(cell.get() + 1);
    }).join().unwrap();
}

// 一旦所有子线程结束，收集它们的线程局部变量中的计数器值，然后进行求和
let tls = Arc::try_unwrap(tls).unwrap();
let total = tls.into_iter().fold(0, |x, y| x + y.get());

// 和为5
assert_eq!(total, 5);
```



#### 用条件控制线程的挂起和执行

条件变量(Condition Variables)经常和 Mutex 一起使用，可以让线程挂起，直到某个条件发生后再继续执行：

```rust
use std::thread;
use std::sync::{Arc, Mutex, Condvar};

fn main() {
    let pair = Arc::new((Mutex::new(false), Condvar::new()));
    let pair2 = pair.clone();

    thread::spawn(move|| {
        let (lock, cvar) = &*pair2;
        let mut started = lock.lock().unwrap();
        println!("changing started");
        *started = true;
        cvar.notify_one();
    });

    let (lock, cvar) = &*pair;
    let mut started = lock.lock().unwrap();
    while !*started {
        started = cvar.wait(started).unwrap();
    }

    println!("started changed");
}
```

上述代码流程如下：
1. main 线程首先进入 while 循环，调用 wait 方法挂起等待子线程的通知，并释放了锁 started
2. 子线程获取到锁，并将其修改为 true，然后调用条件变量的 notify_one 方法来通知主线程继续执行




#### 只被调用一次的函数

我们会需要某个函数在多线程环境下只被调用一次，
例如初始化全局变量，无论是哪个线程先调用函数来初始化，都会保证全局变量只会被初始化一次，随后的其它线程调用就会忽略该函数：

```rust
use std::thread;
use std::sync::Once;

static mut VAL: usize = 0;
static INIT: Once = Once::new();

fn main() {
    let handle1 = thread::spawn(move || {
        INIT.call_once(|| {
            unsafe {
                VAL = 1;
            }
        });
    });

    let handle2 = thread::spawn(move || {
        INIT.call_once(|| {
            unsafe {
                VAL = 2;
            }
        });
    });

    handle1.join().unwrap();
    handle2.join().unwrap();

    println!("{}", unsafe { VAL });
}
```


#### 线程间的消息传递

在多线程间有多种方式可以共享、传递数据，最常用的方式就是==通过消息传递==或者==将锁和Arc联合==使用，
而对于前者，在编程界还有一个大名鼎鼎的Actor线程模型为其背书，典型的有 Erlang 语言，还有 Go 语言中很经典的一句话：
> Do not communicate by sharing memory; instead, share memory by communicating

而对于后者，我们将在下一节中进行讲述。

。。有点道理，而且和rust很像。
。。通过 共享内存交流的话， 没人知道 现在 有多少根线程 在 读写 这块内存， 所以 必须加锁。
。。通过 交流 来共享内存的话， 我(发起交流方) 肯定持有了 这块内存， 我把这块内存 发给 接受者，那么接受者 就拥有了 这块内存的 所有权。  。。 应该是这样的。。。但是，下面的 消息通道，多对多，还是会导致 内存被共享啊。

##### 消息通道
与 Go ==语言内置==的chan不同，Rust 是在==标准库==里提供了消息通道(channel)，你可以将其想象成一场直播，多个主播联合起来在搞一场直播，最终内容通过通道传输给屏幕前的我们，其中主播被称之为发送者，观众被称之为接收者，显而易见的是：一个通道应该支持==多个发送者和接收者==。

但是，在实际使用中，我们需要使用不同的库来满足诸如：多发送者 -> 单接收者，多发送者 -> 多接收者等场景形式，此时一个标准库显然就不够了，不过别急，让我们先从标准库讲起。


##### 多发送者，单接受者
`std::sync::mpsc`

`mpsc`是`multiple producer, single consumer`的缩写
当然，支持多个发送者也意味着支持单个发送者

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    // 创建一个消息通道, 返回一个元组：(发送者，接收者)
    let (tx, rx) = mpsc::channel();

    // 创建线程，并发送消息
    thread::spawn(move || {
        // 发送一个数字1, send方法返回Result<T,E>，通过unwrap进行快速错误处理
        tx.send(1).unwrap();

        // 下面代码将报错，因为编译器自动推导出通道传递的值是i32类型，那么Option<i32>类型将产生不匹配错误
        // tx.send(Some(1)).unwrap()
    });

    // 在主线程中接收子线程发送的消息并输出
    println!("receive {}", rx.recv().unwrap());
}
```


- tx,rx对应发送者和接收者，它们的类型由编译器自动推导: tx.send(1)发送了整数，因此它们分别是`mpsc::Sender<i32>`和`mpsc::Receiver<i32>`类型，需要注意，由于内部是泛型实现，一旦类型被推导确定，该通道就只能传递对应类型的值, 例如此例中==非i32类型==的值将导致==编译错误==
- 接收消息的操作rx.recv()会==阻塞==当前线程，直到读取到值，或者通道被关闭
- 需要使用move将tx的所有权转移到子线程的闭包中


-------------
不阻塞的 try_recv 方法

不会阻塞线程，当通道中没有消息时，它会立刻返回一个错误

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        tx.send(1).unwrap();
    });

    println!("receive {:?}", rx.try_recv());
}
```


##### 传输具有所有权的数据

使用通道来传输数据，一样要遵循 Rust 的所有权规则：
- 若值的类型实现了Copy特征，则直接复制一份该值，然后传输过去，例如之前的i32类型
- 若值没有实现Copy，则它的所有权会被转移给接收端，在发送端继续使用该值将报错

第二种情况
```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let s = String::from("我，飞走咯!");
        tx.send(s).unwrap();
        println!("val is {}", s);   // error: value borrowed here after move
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```


-----------
##### 使用 for 进行循环接收
`for msg in rx {`

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {                // <<<<<
        println!("Got: {}", received);
    }
}
```

--------
##### 使用多发送者

由于子线程会拿走发送者的所有权，因此我们必须对发送者进行克隆，然后让每个线程拿走它的一份拷贝:

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();
    let tx1 = tx.clone();
    thread::spawn(move || {
        tx.send(String::from("hi from raw tx")).unwrap();
    });

    thread::spawn(move || {
        tx1.send(String::from("hi from cloned tx")).unwrap();
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

- 需要所有的发送者都被drop掉后，接收者rx才会收到错误，进而跳出for循环，最终结束主线程
- 这里虽然用了clone但是并不会影响性能，因为它并不在热点代码路径中，仅仅会被执行一次
- 由于两个子线程谁先创建完成是未知的，因此哪条消息先发送也是未知的，最终主线程的输出顺序也不确定


##### 消息顺序

上述第三点的消息顺序仅仅是因为线程创建引起的，并不代表通道中的消息是无序的，对于通道而言，消息的发送顺序和接收顺序是一致的，满足FIFO原则(先进先出)。



##### 同步和异步通道
Rust 标准库的mpsc通道其实分为两种类型：同步和异步。

异步通道
之前我们使用的都是异步通道：无论接收者是否正在接收消息，消息发送者在发送消息时都不会阻塞:

同步通道
与异步通道相反，同步通道发送消息是阻塞的，只有在消息被接收后才解除阻塞，例如：
```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;
fn main() {
    let (tx, rx)= mpsc::sync_channel(0);            // <<<<< sync_channel, channel

    let handle = thread::spawn(move || {
        println!("发送之前");
        tx.send(1).unwrap();
        println!("发送之后");
    });

    println!("睡眠之前");
    thread::sleep(Duration::from_secs(3));
    println!("睡眠之后");

    println!("receive {}", rx.recv().unwrap());
    handle.join().unwrap();
}
```

我们传递了一个参数0: mpsc::sync_channel(0);，这是什么意思呢？

该值可以用来指定同步通道的消息缓存条数，
当你设定为N时，发送者就可以无阻塞的往通道中发送N条消息，
当消息缓冲队列满了后，新的消息发送将被阻塞(如果没有接收者消费缓冲队列中的消息，那么第N+1条消息就将触发发送阻塞)。


-----------
所有发送者被drop或者所有接收者被drop后，通道会自动关闭。
这件事是在编译期实现的，完全没有运行期性能损耗


------------
##### 传输多种类型的数据

如果你想要传输多种类型的数据，可以为每个类型创建一个通道，你也可以使用枚举类型来实现：

```rust
use std::sync::mpsc::{self, Receiver, Sender};

enum Fruit {
    Apple(u8),
    Orange(String)
}

fn main() {
    let (tx, rx): (Sender<Fruit>, Receiver<Fruit>) = mpsc::channel();

    tx.send(Fruit::Orange("sweet".to_string())).unwrap();
    tx.send(Fruit::Apple(2)).unwrap();

    for _ in 0..2 {
        match rx.recv().unwrap() {
            Fruit::Apple(count) => println!("received {} apples", count),
            Fruit::Orange(flavor) => println!("received {} oranges", flavor),
        }
    }
}
```
枚举类型还能让我们带上想要传输的数据，
但是有一点需要注意，Rust 会按照枚举中占用内存最大的那个成员进行内存对齐，
这意味着就算你传输的是枚举中占用内存最小的成员，它占用的内存依然和最大的成员相同, 因此会造成内存上的浪费。


-----------
新手遇到的坑

```rust
use std::sync::mpsc;
fn main() {

    use std::thread;

    let (send, recv) = mpsc::channel();
    let num_threads = 3;
    for i in 0..num_threads {
        let thread_send = send.clone();
        thread::spawn(move || {
            thread_send.send(i).unwrap();
            println!("thread {:?} finished", i);
        });
    }

    // 在这里drop send...

    for x in recv {
        println!("Got: {}", x);
    }
    println!("finished iterating");
}
```
运行后主线程会一直阻塞，最后一行打印输出也不会被执行
因为send本身直到main函数的结束才会被drop。

通道关闭的两个条件：发送者全部drop或接收者被drop，
要结束for循环显然是要求发送者全部drop，但是由于send自身没有被drop，会导致该循环永远无法结束，最终主线程会一直阻塞。

解决办法很简单，drop掉send即可：在代码中的注释下面添加一行drop(send);。



---------------
##### mpmc 更好的性能
如果你需要 mpmc(多发送者，多接收者)或者需要更高的性能，可以考虑第三方库:
- crossbeam-channel, 老牌强库，功能较全，性能较强，之前是独立的库，但是后面合并到了crossbeam主仓库中
- flume, 官方给出的性能数据某些场景要比 crossbeam 更好些



#### 线程同步：锁，Condvar，信号量


共享内存可以说是同步的灵魂，因为消息传递的底层实际上也是通过共享内存来实现，两者的区别如下：
- 共享内存相对消息传递能节省多次内存拷贝的成本
- 共享内存的实现简洁的多
- 共享内存的锁竞争更多

消息传递适用的场景很多，我们下面列出了几个主要的使用场景:
- 需要可靠和简单的(简单不等于简洁)实现时
- 需要模拟现实世界，例如用消息去通知某个目标执行相应的操作时
- 需要一个任务处理流水线(管道)时，等等

而使用共享内存(并发原语)的场景往往就比较简单粗暴：需要==简洁的实现==以及==更高的性能==时。

总之，消息传递类似一个单所有权的系统：一个值同时只能有一个所有者，如果另一个线程需要该值的所有权，需要将所有权通过消息传递进行转移。
而共享内存类似于一个多所有权的系统：多个线程可以同时访问同一个值。


##### Mutex

Mutex让多个线程并发的访问同一个值变成了排队访问：同一时间，只允许一个线程A访问该值，其它线程需要等待A访问完成后才能继续。


----------
单线程中使用

```rust
use std::sync::Mutex;

fn main() {
    // 使用`Mutex`结构体的关联函数创建新的互斥锁实例
    let m = Mutex::new(5);

    {
        // 获取锁，然后deref为`m`的引用
        // lock返回的是Result
        let mut num = m.lock().unwrap();
        *num = 6;
        // 锁自动被drop
    }

    println!("m = {:?}", m);
}
```
和Box类似，==数据被Mutex所拥有==，要访问内部的数据，需要使用方法m.lock()向m申请一个锁, 该方法会阻塞当前线程，直到获取到锁，因此当多个线程同时访问该数据时，只有一个线程能获取到锁，其它线程只能阻塞着等待，这样就保证了数据能被安全的修改！

m.lock()方法也有可能报错，例如当前正在持有锁的线程panic了。在这种情况下，其它线程不可能再获得锁，因此lock方法会返回一个错误。

m.lock明明返回一个锁，怎么就变成我们的num数值了？聪明的读者可能会想到智能指针，没错，因为`Mutex<T>`是一个智能指针，准确的说是m.lock()返回一个智能指针`MutexGuard<T>`:
- 它实现了Deref特征，会被自动解引用后获得一个引用类型，该引用指向Mutex内部的数据
- 它还实现了Drop特征，在超出作用域后，自动释放锁，以便其它线程能继续获取锁

如果释放锁，你需要做的仅仅是做好锁的作用域管理

-------------
##### 多线程中使用 Mutex

`Rc<T>`无法在线程中传输，因为它没有实现Send特征

多线程安全的 `Arc<T>`

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```


`Rc<T>/RefCell<T>`用于单线程内部可变性， 
`Arc<T>/Mutex<T>`用于多线程内部可变性。


------------
死锁

单线程死锁
```rust
use std::sync::Mutex;

fn main() {
    let data = Mutex::new(0);
    let d1 = data.lock();
    let d2 = data.lock();
} // d1锁在此处释放
```

-------
多线程死锁

```rust
use std::{sync::{Mutex, MutexGuard}, thread};
use std::thread::sleep;
use std::time::Duration;

use lazy_static::lazy_static;
lazy_static! {
    static ref MUTEX1: Mutex<i64> = Mutex::new(0);
    static ref MUTEX2: Mutex<i64> = Mutex::new(0);
}

fn main() {
    // 存放子线程的句柄
    let mut children = vec![];
    for i_thread in 0..2 {
        children.push(thread::spawn(move || {
            for _ in 0..1 {
                // 线程1
                if i_thread % 2 == 0 {
                    // 锁住MUTEX1
                    let guard: MutexGuard<i64> = MUTEX1.lock().unwrap();

                    println!("线程 {} 锁住了MUTEX1，接着准备去锁MUTEX2 !", i_thread);

                    // 当前线程睡眠一小会儿，等待线程2锁住MUTEX2
                    sleep(Duration::from_millis(10));

                    // 去锁MUTEX2
                    let guard = MUTEX2.lock().unwrap();
                // 线程2
                } else {
                    // 锁住MUTEX2
                    let _guard = MUTEX2.lock().unwrap();

                    println!("线程 {} 锁住了MUTEX2, 准备去锁MUTEX1", i_thread);

                    let _guard = MUTEX1.lock().unwrap();
                }
            }
        }));
    }

    // 等子线程完成
    for child in children {
        let _ = child.join();
    }

    println!("死锁没有发生");
}
```


--------
try_lock

try_lock会尝试去获取一次锁，如果无法获取会返回一个错误，因此不会发生阻塞

```rust
    if i_thread % 2 == 0 {
        // 锁住MUTEX1
        let guard: MutexGuard<i64> = MUTEX1.lock().unwrap();

        println!("线程 {} 锁住了MUTEX1，接着准备去锁MUTEX2 !", i_thread);

        // 当前线程睡眠一小会儿，等待线程2锁住MUTEX2
        sleep(Duration::from_millis(10));

        // 去锁MUTEX2
        let guard = MUTEX2.try_lock();
        println!("线程 {} 获取 MUTEX2 锁的结果: {:?}", i_thread, guard);
    // 线程2
    } else {
        // 锁住MUTEX2
        let _guard = MUTEX2.lock().unwrap();

        println!("线程 {} 锁住了MUTEX2, 准备去锁MUTEX1", i_thread);
        sleep(Duration::from_millis(10));
        let guard = MUTEX1.try_lock();
        println!("线程 {} 获取 MUTEX1 锁的结果: {:?}", i_thread, guard);
    }
```

当try_lock失败时，会报出一个错误:Err("WouldBlock")，接着线程中的剩余代码会继续执行，不会被阻塞。


------------
##### 读写锁 RwLock

```rust
use std::sync::RwLock;

fn main() {
    let lock = RwLock::new(5);

    // 同一时间允许多个读
    {
        let r1 = lock.read().unwrap();
        let r2 = lock.read().unwrap();
        assert_eq!(*r1, 5);
        assert_eq!(*r2, 5);
    } // 读锁在此处被drop

    // 同一时间只允许一个写
    {
        let mut w = lock.write().unwrap();
        *w += 1;
        assert_eq!(*w, 6);

        // 以下代码会阻塞发生死锁，因为读和写不允许同时存在
        // 写锁w直到该语句块结束才被释放，因此下面的读锁依然处于`w`的作用域中
        // let r1 = lock.read();
        // println!("{:?}",r1);
    }// 写锁在此处被drop
}
```
我们也可以使用try_write和try_read来尝试进行一次写/读，若失败则返回错误:
`Err("WouldBlock")`


RwLock虽然看上去貌似提供了高并发读取的能力，但这个不能说明它的性能比Mutex高，事实上Mutex性能要好不少，后者唯一的问题也仅仅在于不能并发读取。

一个常见的、错误的使用RwLock的场景就是使用HashMap进行简单读写，因为HashMap的读和写都非常快，RwLock的复杂实现和相对低的性能反而会导致整体性能的降低，因此一般来说更适合使用Mutex。

如果你要使用RwLock要确保满足以下两个条件：并发读，且需要对读到的资源进行"长时间"的操作


------------
##### 第三方库提供的lock
标准库在设计时总会存在取舍，因为往往性能并不是最好的，如果你追求性能，可以使用三方库提供的并发原语:
- parking_lot, 功能更完善、稳定，社区较为活跃，star 较多，更新较为活跃
- spin, 在多数场景中性能比parking_lot高一点，最近没怎么更新



-------------
##### 条件变量Condvar控制线程的同步

Mutex用于解决资源安全访问的问题，但是我们还需要一个手段来解决资源访问顺序的问题。

条件变量(Condition Variables)，它经常和Mutex一起使用，可以让线程挂起，直到某个条件发生后再继续执行

```rust
use std::sync::{Arc,Mutex,Condvar};
use std::thread::{spawn,sleep};
use std::time::Duration;

fn main() {
    let flag = Arc::new(Mutex::new(false));
    let cond = Arc::new(Condvar::new());
    let cflag = flag.clone();
    let ccond = cond.clone();

    let hdl = spawn(move || {
        let mut lock = cflag.lock().unwrap();
        let mut counter = 0;

        while counter < 3 {
            while !*lock {
                // wait方法会接收一个MutexGuard<'a, T>，且它会自动地暂时释放这个锁，使其他线程可以拿到锁并进行数据更新。
                // 同时当前线程在此处会被阻塞，直到被其他地方notify后，它会将原本的MutexGuard<'a, T>还给我们，即重新获取到了锁，同时唤醒了此线程。
                lock = ccond.wait(lock).unwrap();
            }
            
            *lock = false;

            counter += 1;
            println!("inner counter: {}", counter);
        }
    });

    let mut counter = 0;
    loop {
        sleep(Duration::from_millis(1000));
        *flag.lock().unwrap() = true;
        counter += 1;
        if counter > 3 {
            break;
        }
        println!("outside counter: {}", counter);
        cond.notify_one();
    }
    hdl.join().unwrap();
    println!("{:?}", flag);
}
```



------------
##### Semaphore

本来 Rust 在标准库中有提供一个信号量实现, 但是由于各种原因这个库现在已经不再推荐使用了，因此我们推荐使用tokio中提供的Semaphore实现: tokio::sync::Semaphore。

```rust
use std::sync::Arc;
use tokio::sync::Semaphore;

#[tokio::main]
async fn main() {
    let semaphore = Arc::new(Semaphore::new(3));
    let mut join_handles = Vec::new();

    for _ in 0..5 {
        let permit = semaphore.clone().acquire_owned().await.unwrap();
        join_handles.push(tokio::spawn(async move {
            //
            // 在这里执行任务...
            //
            drop(permit);
        }));
    }

    for handle in join_handles {
        handle.await.unwrap();
    }
}
```

-------------
#### Atomic原子类型与内存顺序

从 Rust1.34 版本后，就正式支持原子类型。

Mutex用起来简单，但是无法并发读，
RwLock可以并发读，但是使用场景较为受限且性能不够，
那么有没有一种全能性选手呢？ 欢迎我们的Atomic闪亮登场。

==在多核 CPU 下，当某个 CPU 核心开始运行原子操作时，会先暂停其它 CPU 内核对内存的操作，以保证原子操作不会被其它 CPU 内核所干扰。==

原子操作是通过指令提供的支持，因此它的性能相比锁和消息传递会好很多。
相比较于锁而言，原子类型不需要开发者处理加锁和释放锁的问题，同时支持修改，读取等操作，还具备较高的并发性能，几乎所有的语言都支持原子类型。

可以看出原子类型是无锁类型，但是无锁不代表无需等待，因为原子类型内部使用了CAS循环，当大量的冲突发生时，该等待还是得等待！但是总归比锁要好。


#### 使用 Atomic 作为全局变量

原子类型的一个常用场景，就是作为全局变量来使用:

```rust
use std::ops::Sub;
use std::sync::atomic::{AtomicU64, Ordering};
use std::thread::{self, JoinHandle};
use std::time::Instant;

const N_TIMES: u64 = 10000000;
const N_THREADS: usize = 10;

static R: AtomicU64 = AtomicU64::new(0);

fn add_n_times(n: u64) -> JoinHandle<()> {
    thread::spawn(move || {
        for _ in 0..n {
            R.fetch_add(1, Ordering::Relaxed);
        }
    })
}

fn main() {
    let s = Instant::now();
    let mut threads = Vec::with_capacity(N_THREADS);

    for _ in 0..N_THREADS {
        threads.push(add_n_times(N_TIMES));
    }

    for thread in threads {
        thread.join().unwrap();
    }

    assert_eq!(N_TIMES * N_THREADS as u64, R.load(Ordering::Relaxed));

    println!("{:?}",Instant::now().sub(s));
}
```

当然以上代码的功能其实也可以通过Mutex来实现，但是后者的强大功能是建立在额外的性能损耗基础上的，因此性能会逊色不少:
```text
Atomic实现：673ms
Mutex实现: 1136ms
```

可以看到Atomic实现会比Mutex快41%，实际上在复杂场景下还能更快(甚至达到 4 倍的性能差距)！

和Mutex一样，Atomic的值具有内部可变性，你无需将其声明为mut


`Ordering::Relaxed`用来控制 原子操作的 内存顺序。

#### 内存顺序

内存顺序是指 CPU 在访问内存时的顺序，该顺序可能受以下因素的影响：
- 代码中的先后顺序
- 编译器优化导致在编译阶段发生改变(内存重排序 reordering)
- 运行阶段因 CPU 的缓存机制导致顺序被打乱


编译器优化是指：
```rust
static mut x = 0;
static mut y = 0;
fn main() {
    unsafe {
        x = 1;
        y = 3;
        x = 2;
    }
}
```
由于 2次x 之间没有人调用x，所以会直接删除 x=1。 这样，其他线程 永远不会看到x=1了。


CPU缓存

下面就是2根线程的执行，结果有各种可能。
```rust
initial state: X = 0, Y = 1

THREAD Main     THREAD A
X = 1;          if X == 1 {
Y = 3;              Y *= 2;
X = 2;          }
```

y的结果：
- 3
- 6
- 2，A读取y的值1，main开始执行y=3, main执行完y=3，A执行y=1*2。
- 2，main线程执行了 y=3，但是还没有同步到其他CPU缓存中，导致 y=1*2 = 2


可能出现Y = 2，因为Main线程中的X和Y被同步到其它 CPU 缓存中的顺序未必一致。
```rust
initial state: X = 0, Y = 1

THREAD Main     THREAD A
X = 1;          if X == 2 {
Y = 3;              Y *= 2;
X = 2;          }
```


##### 限定内存顺序的 5 个规则

Ordering 有5个成员：
- Relaxed，最宽松，对编译器和CPU不做任何限制，可以乱序
- Release，设定内存屏障，保证它之前的操作 永远在它之前，但是 它后面的操作可能被重排到它之前
- Acquire，保证它之后的访问 永远在它之后，但是它之前的操作 却有可能被重排到它后面，往往和Release在不同线程中联合使用。
- AcqRel，release+acquire，
- SeqCst，顺序一致性，是AcqRel 的加强版，

。。想了下，应该是 AcqRel 是 对这个对象的  之前的对它的 操作 肯定在之前， 之后的对它的 操作肯定在之后。 ( 就是 非操作这个对象的 操作 可能被重排序掉)
。。SeqCst 应该是， 对这个对象 之前的 所有操作(不管是对谁的) 都在前面， 之后的。。
。。应该是。不然想不出 SeqCst 怎么加强 AcqRel 了。


-----------
内存屏障例子

原则上，Acquire用于读取，而Release用于写入。但是由于有些原子操作同时拥有读取和写入的功能，此时就需要使用AcqRel来设置内存顺序了。在内存屏障中被写入的数据，都可以被其它线程读取到，不会有 CPU 缓存的问题。

下面我们以Release和Acquire为例，使用它们构筑出一对内存屏障，防止编译器和 CPU 将屏障前(Release)和屏障后(Acquire)中的数据操作重新排在屏障围成的范围之外:


```rust
use std::thread::{self, JoinHandle};
use std::sync::atomic::{Ordering, AtomicBool};

static mut DATA: u64 = 0;
static READY: AtomicBool = AtomicBool::new(false);

fn reset() {
    unsafe {
        DATA = 0;
    }
    READY.store(false, Ordering::Relaxed);
}

fn producer() -> JoinHandle<()> {
    thread::spawn(move || {
        unsafe {
            DATA = 100;                                 // A
        }
        READY.store(true, Ordering::Release);           // B: 内存屏障 ↑
    })
}

fn consumer() -> JoinHandle<()> {
    thread::spawn(move || {
        while !READY.load(Ordering::Acquire) {}         // C: 内存屏障 ↓

        assert_eq!(100, unsafe { DATA });               // D
    })
}


fn main() {
    loop {
        reset();

        let t_producer = producer();
        let t_consumer = consumer();

        t_producer.join().unwrap();
        t_consumer.join().unwrap();
    }
}
```


内存顺序的选择
- 不知道怎么选择时，优先使用SeqCst，虽然会稍微减慢速度，但是慢一点也比出现错误好
- 多线程只计数fetch_add而不使用该值触发其他逻辑分支的简单使用场景，可以使用Relaxed 参考 Which std::sync::atomic::Ordering to use? (https://stackoverflow.com/questions/30407121/which-stdsyncatomicordering-to-use)


-------------
#### 多线程中使用 Atomic

```rust
use std::sync::Arc;
use std::sync::atomic::{AtomicUsize, Ordering};
use std::{hint, thread};

fn main() {
    let spinlock = Arc::new(AtomicUsize::new(1));

    let spinlock_clone = Arc::clone(&spinlock);
    let thread = thread::spawn(move|| {
        spinlock_clone.store(0, Ordering::SeqCst);
    });

    // 等待其它线程释放锁
    while spinlock.load(Ordering::SeqCst) != 0 {
        hint::spin_loop();
    }

    if let Err(panic) = thread.join() {
        println!("Thread had an error: {:?}", panic);
    }
}
```


----------
那么原子类型既然这么全能，它可以替代锁吗？答案是不行：
- 对于复杂的场景下，锁的使用简单粗暴，不容易有坑
- std::sync::atomic包中仅提供了数值类型的原子操作：AtomicBool, AtomicIsize, AtomicUsize, AtomicI8, AtomicU16等，而锁可以应用于各种类型
- 在有些情况下，必须使用锁来配合，例如上一章节中使用Mutex配合Condvar


------------
事实上，Atomic虽然对于用户不太常用，但是对于高性能库的开发者、标准库开发者都非常常用，它是并发原语的基石，除此之外，还有一些场景适用：
- 无锁(lock free)数据结构
- 全局变量，例如全局自增 ID, 在后续章节会介绍
- 跨线程计数器，例如可以用于统计指标


#### 基于Send 和 Sync的线程安全

为何 Rc、RefCell 和裸指针不可以在多线程间使用？如何让裸指针可以在多线程使用？我们一起来探寻下这些问题的答案。

```rust
// Rc源码片段
impl<T: ?Sized> !marker::Send for Rc<T> {}
impl<T: ?Sized> !marker::Sync for Rc<T> {}

// Arc源码片段
unsafe impl<T: ?Sized + Sync + Send> Send for Arc<T> {}
unsafe impl<T: ?Sized + Sync + Send> Sync for Arc<T> {}
```
Send和Sync是在线程间安全使用一个值的关键。


Send和Sync是 Rust 安全并发的重中之重，但是实际上它们只是标记特征(marker trait，该特征未定义任何行为，因此非常适合用于标记), 来看看它们的作用：
- 实现Send的类型可以在线程间安全的传递其所有权
- 实现Sync的类型可以在线程间安全的共享(通过引用)

还有一个潜在的依赖：一个类型要在线程间安全的共享的前提是，指向它的引用必须能在线程间传递。因为如果引用都不能被传递，我们就无法在多个线程间使用引用去访问同一个数据了。
由上可知，若类型 T 的引用&T是Send，则T是Sync。


。。tiao


- 实现Send的类型可以在线程间安全的传递其所有权, 实现Sync的类型可以在线程间安全的共享(通过引用)
- 绝大部分类型都实现了Send和Sync，常见的未实现的有：裸指针、Cell、RefCell、Rc 等
- 可以为自定义类型实现Send和Sync，但是需要unsafe代码块
- 可以为部分 Rust 中的类型实现Send、Sync，但是需要使用newtype，例如文中的裸指针例子



### 全局变量

首先，有一点可以肯定，全局变量的生命周期肯定是'static，但是不代表它需要用static来声明，
例如常量、字符串字面值等无需使用static进行声明，原因是它们已经被打包到二进制可执行文件中。


---------
#### 编译期初始化

我们大多数使用的全局变量都只需要在编译期初始化即可，例如静态配置、计数器、状态值等等。

-------
静态常量

全局常量可以在程序任何一部分使用，当然，如果它是定义在某个模块中，你需要引入对应的模块才能使用。常量，顾名思义它是不可变的，很适合用作静态配置：
```rust
const MAX_ID: usize =  usize::MAX / 2;
fn main() {
   println!("用户ID允许的最大值是{}",MAX_ID);
}
```

常量与普通变量的区别
- 关键字是const而不是let
- 定义常量必须指明类型（如 i32）不能省略
- 定义常量时变量的命名规则一般是全部大写
- 常量可以在任意作用域进行定义，其生命周期贯穿整个程序的生命周期。编译时编译器会尽可能将其内联到代码中，所以在不同地方对同一常量的引用并==不能保证引用到相同的内存地址==
- 常量的赋值只能是常量表达式/数学表达式，也就是说必须是在编译期就能计算出的值，如果需要在运行时才能得出结果的值比如函数，则不能赋值给常量表达式
- 对于变量出现重复的定义(绑定)会发生变量遮盖，后面定义的变量会遮住前面定义的变量，常量则不允许出现重复的定义


----------------
静态变量

静态变量允许声明一个全局的变量，常用于全局数据统计，例如我们希望用一个变量来统计程序当前的总请求数：

```rust
static mut REQUEST_RECV: usize = 0;
fn main() {
   unsafe {
        REQUEST_RECV += 1;
        assert_eq!(REQUEST_RECV, 1);
   }
}
```
Rust 要求必须使用unsafe语句块才能访问和修改static变量，因为这种使用方式往往并不安全，其实编译器是对的，当在多线程中同时去修改时，会不可避免的遇到脏数据。

只有在同一线程内或者不在乎数据的准确性时，才应该使用全局静态变量。

静态变量和常量的区别
- 静态变量不会被内联，在整个程序中，静态变量只有一个实例，所有的引用都会指向同一个地址
- 存储在静态变量中的值必须要实现 Sync trait


---------
原子类型

想要全局计数器、状态控制等功能，又想要线程安全的实现，原子类型是非常好的办法。
```rust
use std::sync::atomic::{AtomicUsize, Ordering};
static REQUEST_RECV: AtomicUsize  = AtomicUsize::new(0);
fn main() {
    for _ in 0..100 {
        REQUEST_RECV.fetch_add(1, Ordering::Relaxed);
    }

    println!("当前用户请求数{:?}",REQUEST_RECV);
}
```



------
#### 运行期初始化

以上的静态初始化有一个致命的问题：无法用函数进行静态初始化，例如你如果想声明一个全局的Mutex锁：
```rust
use std::sync::Mutex;
static NAMES: Mutex<String> = Mutex::new(String::from("Sunface, Jack, Allen"));

fn main() {
    let v = NAMES.lock().unwrap();
    println!("{}",v);
}
```
运行后报错如下：
```rust
error[E0015]: calls in statics are limited to constant functions, tuple structs and tuple variants
 --> src/main.rs:3:42
  |
3 | static NAMES: Mutex<String> = Mutex::new(String::from("sunface"));
```

##### lazy_static
lazy_static是社区提供的非常强大的宏，用于懒初始化静态变量，之前的静态变量都是在编译期初始化的，因此无法使用函数调用进行赋值，而lazy_static允许我们在运行期初始化静态变量！

```rust
use std::sync::Mutex;
use lazy_static::lazy_static;
lazy_static! {
    static ref NAMES: Mutex<String> = Mutex::new(String::from("Sunface, Jack, Allen"));
}

fn main() {
    let mut v = NAMES.lock().unwrap();
    v.push_str(", Myth");
    println!("{}",v);
}
```
当然，使用lazy_static在每次访问静态变量时，会有轻微的性能损失，因为其内部实现用了一个底层的并发原语std::sync::Once，在每次访问该变量时，程序都会执行一次原子指令用于确认静态变量的初始化是否完成。

lazy_static宏，匹配的是static ref，所以定义的静态变量都是不可变引用

可能有读者会问，为何需要在运行期初始化一个静态变量，除了上面的全局锁，你会遇到最常见的场景就是：一个全局的动态配置，它在程序开始后，才加载数据进行初始化，最终可以让各个线程直接访问使用


一个使用lazy_static实现全局缓存的例子:
```rust
use lazy_static::lazy_static;
use std::collections::HashMap;

lazy_static! {
    static ref HASHMAP: HashMap<u32, &'static str> = {
        let mut m = HashMap::new();
        m.insert(0, "foo");
        m.insert(1, "bar");
        m.insert(2, "baz");
        m
    };
}

fn main() {
    // 首次访问`HASHMAP`的同时对其进行初始化
    println!("The entry for `0` is \"{}\".", HASHMAP.get(&0).unwrap());

    // 后续的访问仅仅获取值，再不会进行任何初始化操作
    println!("The entry for `1` is \"{}\".", HASHMAP.get(&1).unwrap());
}
```

----------
##### Box::leak

Box::leak方法，它可以将一个变量从内存中泄漏(听上去怪怪的，竟然做主动内存泄漏)，然后将其变为'static生命周期，最终该变量将和程序活得一样久，因此可以赋值给全局静态变量CONFIG。

```rust
#[derive(Debug)]
struct Config {
    a: String,
    b: String
}
static mut CONFIG: Option<&mut Config> = None;

fn main() {
    let c = Box::new(Config {
        a: "A".to_string(),
        b: "B".to_string(),
    });

    unsafe {
        // 将`c`从内存中泄漏，变成`'static`生命周期
        CONFIG = Some(Box::leak(c));
        println!("{:?}", CONFIG);
    }
}
```

---------------
##### 从函数中返回全局变量

报错这里就不展示了，跟之前大同小异，还是生命周期引起的，那么该如何解决呢？依然可以用Box::leak:
```rust
#[derive(Debug)]
struct Config {
    a: String,
    b: String,
}
static mut CONFIG: Option<&mut Config> = None;

fn init() -> Option<&'static mut Config> {
    let c = Box::new(Config {
        a: "A".to_string(),
        b: "B".to_string(),
    });

    Some(Box::leak(c))
}


fn main() {
    unsafe {
        CONFIG = init();

        println!("{:?}", CONFIG)
    }
}
```




-----------
#### 标准库中的 OnceCell

在 Rust 标准库中提供了实验性的 lazy::OnceCell 和 lazy::SyncOnceCell (在 Rust 1.70.0版本及以上的标准库中，替换为稳定的 cell::OnceCell 和 sync::OnceLock )两种 Cell ，前者用于单线程，后者用于多线程，
==它们用来存储堆上的信息，并且具有最 多只能赋值一次的特性。==
如实现一个多线程的日志组件 Logger：


--------

简单来说，全局变量可以分为两种：
- 编译期初始化的全局变量，const创建常量，static创建静态变量，Atomic创建原子类型
- 运行期初始化的全局变量，lazy_static用于懒初始化，Box::leak利用内存泄漏将一个变量的生命周期变为'static








### 错误处理

如何对 Result ( Option ) 做进一步的处理，以及如何定义自己的错误类型


#### or() 和 and()
跟布尔关系的与/或很像，这两个方法会对两个表达式做逻辑组合，最终返回 Option / Result。
- or()，表达式按照顺序求值，若任何一个表达式的结果是 Some 或 Ok，则该值会立刻返回
- and()，若两个表达式的结果都是 Some 或 Ok，则第二个表达式中的值被返回。若任何一个的结果是 None 或 Err ，则立刻返回。


#### or_else() 和 and_then()
它们跟 or() 和 and() 类似，唯一的区别在于，它们的第二个表达式是一个闭包。

#### filter
filter 用于对 Option 进行过滤：


#### map() 和 map_err()
map 可以将 Some 或 Ok 中的值映射为另一个：
如果你想要将 Err 中的值进行改变， map 就无能为力了，此时我们需要用 map_err


#### map_or() 和 map_or_else()
map_or 在 map 的基础上提供了一个默认值:
map_or_else 与 map_or 类似，但是它是通过一个闭包来提供默认值:


#### ok_or() and ok_or_else()
将 Option 类型转换为 Result 类型。其中 ok_or 接收一个默认的 Err 参数:
ok_or_else 接收一个闭包作为 Err 参数:


#### 自定义错误类型

```rust
use std::fmt::{Debug, Display};

pub trait Error: Debug + Display {
    fn source(&self) -> Option<&(Error + 'static)> { ... }
}
```
当自定义类型实现该特征后，该类型就可以作为 Err 来使用
实际上，自定义错误类型只需要实现 Debug 和 Display 特征即可，source 方法是可选的，而 Debug 特征往往也无需手动实现，可以直接通过 derive 来派生


------
简单的错误

```rust
use std::fmt;

// AppError 是自定义错误类型，它可以是当前包中定义的任何类型，在这里为了简化，我们使用了单元结构体作为例子。
// 为 AppError 自动派生 Debug 特征
#[derive(Debug)]
struct AppError;

// 为 AppError 实现 std::fmt::Display 特征
impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "An Error Occurred, Please Try Again!") // user-facing output
    }
}

// 一个示例函数用于产生 AppError 错误
fn produce_error() -> Result<(), AppError> {
    Err(AppError)
}

fn main(){
    match produce_error() {
        Err(e) => eprintln!("{}", e),
        _ => println!("No error"),
    }

    eprintln!("{:?}", produce_error()); // Err({ file: src/main.rs, line: 17 })
}
```

事实上，实现 Debug 和 Display 特征并不是作为 Err 使用的必要条件，大家可以把这两个特征实现和相应使用去除，然后看看代码会否报错。既然如此，我们为何要为自定义类型实现这两个特征呢？原因有二:
- 错误得打印输出后，才能有实际用处，而打印输出就需要实现这两个特征
- 可以将自定义错误转换成 Box<dyn std::error:Error> 特征对象，在后面的归一化不同错误类型部分，我们会详细介绍


----------
更详尽的错误

```rust
use std::fmt;

struct AppError {
    code: usize,
    message: String,
}

// 根据错误码显示不同的错误信息
impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let err_msg = match self.code {
            404 => "Sorry, Can not find the Page!",
            _ => "Sorry, something is wrong! Please Try Again!",
        };

        write!(f, "{}", err_msg)
    }
}

impl fmt::Debug for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(
            f,
            "AppError {{ code: {}, message: {} }}",
            self.code, self.message
        )
    }
}

fn produce_error() -> Result<(), AppError> {
    Err(AppError {
        code: 404,
        message: String::from("Page not found"),
    })
}

fn main() {
    match produce_error() {
        Err(e) => eprintln!("{}", e), // 抱歉，未找到指定的页面!
        _ => println!("No error"),
    }

    eprintln!("{:?}", produce_error()); // Err(AppError { code: 404, message: Page not found })

    eprintln!("{:#?}", produce_error());
    // Err(
    //     AppError { code: 404, message: Page not found }
    // )
}
```



------------
#### 错误转换 From 特征

#### 归一化不同的错误类型

#### 简化错误处理

。。跳





### ==unsafe rust==

unsafe 能赋予我们 5 种超能力，这些能力在安全的 Rust 代码中是无法获取的：
- 解引用裸指针
- 调用一个 unsafe 或外部的函数
- 访问或修改一个可变的静态变量
- 实现一个 unsafe 特征
- 访问 union 中的字段

unsafe 并不能绕过 Rust 的借用检查，也不能关闭任何 Rust 的安全检查规则，例如当你在 unsafe 中使用引用时，该有的检查一样都不会少。

因此 unsafe 能给大家提供的也仅仅是之前的 5 种超能力，在使用这 5 种能力时，编译器才不会进行内存安全方面的检查，最典型的就是使用裸指针(引用和裸指针有很大的区别)。



------------

https://course.rs/advance/unsafe/superpowers.html

讲解了上面提到的5中 unsafe 用法。
还有一些与其他编程语言交互的方法。

。。跳


----------
#### 内联汇编

Rust 提供了 asm! 宏，可以让大家在 Rust 代码中嵌入汇编代码，对于一些极致高性能或者底层的场景还是非常有用的，例如操作系统内核开发。但通常来说，大家并不应该在自己的项目中使用到该项技术，它为极客而生！




### Macro宏编程

大家应该熟悉宏的使用场景，但是不要滥用，当你真的需要时，再回来查看本章了解实现细节，这才是最完美的使用方式。
https://course.rs/advance/macro.html


println!
vec!
assert_eq!

宏的参数可以使用 ()、[] 以及 {}

```rust
fn main() {
    println!("aaaa");
    println!["aaaa"];
    println!{"aaaa"}
}
```


在 Rust 中宏分为两大类：声明式宏( declarative macros ) macro_rules! 和三种过程宏( procedural macros ):
- #[derive]，在之前多次见到的派生宏，可以为目标结构体或枚举派生指定的代码，例如 Debug 特征
- 类属性宏(Attribute-like macro)，用于为目标添加自定义的属性
- 类函数宏(Function-like macro)，看上去就像是函数调用


元编程
从根本上来说，宏是通过一种代码来生成另一种代码，如果大家熟悉元编程，就会发现两者的共同点。
在附录 D中讲到的 derive 属性，就会自动为结构体派生出相应特征所需的代码，例如 `#[derive(Debug)]`，还有熟悉的 println! 和 vec!，所有的这些宏都会展开成相应的代码，且很可能是长得多的代码。


可变参数
Rust 的函数签名是固定的：定义了两个参数，就必须传入两个参数
而宏就可以拥有可变数量的参数


宏展开
由于宏会被展开成其它代码，且这个展开过程是发生在编译器对代码进行解释之前。因此，宏可以为指定的类型实现某个特征：先将宏展开成实现特征的代码后，再被编译。


-------
声明式宏 macro_rules!

来看看该如何使用它来实现 vec!，以下是一个简化实现：
```rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```


。。。生成代码的代码， 比C的 #define 强100倍。
。。在 编译前，根据信息，动态 生成 代码，然后替换，然后 编译。
。。 不是 文本替换。 是 动态生成。


假设我们在开发一个 web 框架，当用户通过 HTTP GET 请求访问 / 根路径时，使用 index 函数为其提供服务:
```rust
#[route(GET, "/")]
fn index() {
```
这里的 `#[route]` 属性就是一个过程宏，它的定义函数大概如下：
```rust
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```




### async/await异步编程

看展示的图片，actix框架比 gin快很多。可能3-10倍？

https://course.rs/advance/async/getting-started.html

#### async简介
async 是 Rust 选择的异步编程模型

。。信息太多了，不可能复制


async vs 其它并发模型
- OS 线程,  最简单，不和 业务代码耦合。 缺点：难以线程间同步，上下文切换代价大。对于IO密集场景来说，即使使用线程池，也无法满足。 rust原生支持线程级的并发编程。
- 事件驱动(Event driven), 事件驱动 通常和 回调 一起。 性能相当好，最大的问题就是 回调地狱：非线性的控制流和结果处理 导致 数据流向 和 错误传播 难以掌控。 可维护性，可读性低。
- 协程(Coroutines) ，可能是目前最火的， go的协程 非常优秀。 和线程类似，无需改变编程模型； 和async 也类似，可以支持 大量任务 并发运行。  协程抽象层次过高，用户无法掌握底层细节，这对于 系统编程语言 和 自定义异步运行时 是难以接受的。
- actor 模型， erlang 的杀手锏。 将所有 并发计算 分割成 一个个小的单元，这些单元 被称为 actor，单元之间通过消息传递的方式进行通信和数据传递，跟分布式系统的设计理念非常相像。一旦遇到流控制、失败重试等场景时，就会变得不太好用
- async/await， 性能高，支持底层编程， 和线程协程一样 无需过多的改变 编程模型。 但是， async的内部实现机制 过于复杂，比线程，协程 更难 理解和使用。


Rust 经过权衡取舍后，最终选择了同时提供多线程编程和 async 编程:
- 前者通过标准库实现，当你无需那么高的并发时，例如需要并行计算时，可以选择它，优点是线程内的代码执行效率更高、实现更直观更简单，这块内容已经在多线程章节进行过深入讲解，不再赘述
- 后者通过语言特性 + 标准库 + 三方库的方式实现，在你需要高并发、异步 I/O 时，选择它就对了


- Future 在 Rust 中是惰性的，只有在被轮询(poll)时才会运行
- Async 在 Rust 中使用开销是零，意味着只有你能看到的代码(自己的代码)才有性能损耗
- Rust 没有内置异步调用所必需的运行时，但有第三方的，如 tokio
- 运行时同时支持单线程和多线程


------------
OS 线程非常适合少量任务并发，因为线程的创建和上下文切换是非常昂贵的，甚至于空闲的线程都会消耗系统资源。
对于长时间运行的 CPU 密集型任务，例如并行计算，使用线程将更有优势。
这种密集任务往往会让所在的线程持续运行，任何不必要的线程切换都会带来性能损耗，因此高并发反而在此时成为了一种多余。同时你所创建的线程数应该等于 CPU 核心数，充分利用 CPU 的并行能力，甚至还可以将==线程绑定到 CPU 核心==上，进一步减少线程上下文切换。


高并发更适合 IO 密集型任务，例如 web 服务器、数据库连接等等网络服务，因为这些任务绝大部分时间都处于等待状态
使用async，既可以有效的降低 CPU 和内存的负担，又可以让大量的任务并发的运行，一个任务==一旦处于IO或者其他等待(阻塞)状态==，就会==被立刻切走并执行另一个任务==，而这里的任务切换的性能开销要远远低于使用多线程时的线程上下文切换。


事实上, async 底层也是基于线程实现，但是它基于线程封装了一个运行时，可以将多个任务映射到少量线程上，然后将线程切换变成了任务切换，后者仅仅是内存中的访问，因此要高效的多



- 有大量 IO 任务需要并发运行时，选 async 模型
- 有部分 IO 任务需要并发运行时，选多线程，如果想要降低线程创建和销毁的开销，可以使用线程池
- 有大量 CPU 密集任务需要并行运行时，例如并行计算，选多线程模型，且让线程数等于或者稍大于 CPU 核心数
- 无所谓时，统一选多线程


async 和多线程的性能对比
|操作	|async	|线程|
|--|--|--|
|创建	|0.3 微秒	|17 微秒|
|线程切换	|0.2 微秒	|1.7 微秒|



----------------
例子：并发下载文件，多线程版
```rust
fn get_two_sites() {
    // 创建两个新线程执行任务
    let thread_one = thread::spawn(|| download("https://course.rs"));
    let thread_two = thread::spawn(|| download("https://fancy.rs"));

    // 等待两个线程的完成
    thread_one.join().expect("thread one panicked");
    thread_two.join().expect("thread two panicked");
}
```

async
```rust
async fn get_two_sites_async() {
    // 创建两个不同的`future`，你可以把`future`理解为未来某个时刻会被执行的计划任务
    // 当两个`future`被同时执行后，它们将并发的去下载目标页面
    let future_one = download_async("https://www.foo.com");
    let future_two = download_async("https://www.bar.com");

    // 同时运行两个`future`，直至完成
    join!(future_one, future_two);
}
```

。。java里的 Future 还是 线程池的。

事实上，async 和多线程并不是二选一，在同一应用中，可以根据情况两者一起使用


----------------
#### Async Rust 当前的进展
Rust 语言的 async 目前还没有达到多线程的成熟度，其中一部分内容还在不断进化中，当然，这并不影响我们在生产级项目中使用，因为社区中还有 tokio 这种大杀器。

使用 async 时，你会遇到好的，也会遇到不好的，例如：
- 收获卓越的性能
- 会经常跟进阶语言特性打交道，例如生命周期等，这些家伙可不好对付
- 一些兼容性问题，例如同步和异步代码、不同的异步运行时( tokio 与 async-std )
- 更昂贵的维护成本，原因是 async 和社区开发的运行时依然在不停的进化



async 的底层实现非常复杂，且会导致编译后文件体积显著增加

要完整的使用 async 异步编程，你需要依赖以下特性和外部库:
- 所必须的特征(例如 Future )、类型和函数，由标准库提供实现
- 关键字 async/await 由 Rust 语言提供，并进行了编译器层面的支持
- 众多实用的类型、宏和函数由官方开发的 futures 包提供(不是标准库)，它们可以用于任何 async 应用中。
- async 代码的执行、IO 操作、任务创建和调度等等复杂功能由社区的 async 运行时提供，例如 tokio 和 async-std



async 中的编译错误和运行时错误跟之前没啥区别，但是依然有以下几点值得注意：
- 编译错误，由于 async 编程时需要经常使用复杂的语言特性，例如生命周期和Pin，因此相关的错误可能会出现的更加频繁
- 运行时错误，编译器会为每一个async函数生成状态机，这会导致在栈跟踪时会包含这些状态机的细节，同时还包含了运行时对函数的调用，因此，栈跟踪记录(例如 panic 时)将变得更加难以解读
- 一些隐蔽的错误也可能发生，例如在一个 async 上下文中去调用一个阻塞的函数，或者没有正确的实现 Future 特征都有可能导致这种错误。这种错误可能会悄无声息的通过编译检查甚至有时候会通过单元测试。好在一旦你深入学习并掌握了本章的内容和 async 原理，可以有效的降低遇到这些错误的概率



#### async/.await 简单入门
async/.await 是 Rust 内置的语言特性，可以让我们用同步的方式去编写异步的代码。

==通过 async 标记的语法块会被转换成实现了Future特征的状态机==。 
与同步调用阻塞当前线程不同，当Future执行并遇到阻塞时，它会让出当前线程的控制权，这样其它的Future就可以在该线程中运行，这种方式完全不会导致当前线程的阻塞。


1. 引入 futures 包
```text
[dependencies]
futures = "0.3"
```

2. 使用 async fn 语法来创建一个异步函数:

。。没有写返回值类型，是因为 默认规定死 Future 了吗？

```rust
async fn do_something() {
    println!("go go go !");
}
```

异步函数的返回值是一个 Future，若直接调用该函数，不会输出任何结果，因为 Future 还未被执行:
```rust
fn main() {
    do_something();
}
```
上面不会打印 go go go

。。 ==block_on阻塞当前线程==
使用一个执行器( executor ):
```rust
// `block_on`会阻塞当前线程直到指定的`Future`执行完成，这种阻塞当前线程以等待任务完成的方式较为简单、粗暴，
// 好在其它运行时的执行器(executor)会提供更加复杂的行为，例如将多个`future`调度到同一个线程上执行。
use futures::executor::block_on;

async fn hello_world() {
    println!("hello, world!");
}

fn main() {
    let future = hello_world(); // 返回一个Future, 因此不会打印任何输出
    block_on(future); // 执行`Future`并等待其运行完成，此时"hello, world!"会被打印输出
}
```

。。这不就是 线程池？ 那么就是底层实现的 逻辑不同。 但是看起来 就是 线程池。Future版的线程池。


-------------
使用.await

如果你要在一个async fn函数中去调用另一个async fn并等待其完成后再执行后续的代码，该如何做？
```rust
use futures::executor::block_on;

async fn hello_world() {
    hello_cat();
    println!("hello, world!");
}

async fn hello_cat() {
    println!("hello, kitty!");
}
fn main() {
    let future = hello_world();
    block_on(future);
}
```
上面的代码会 warning，因为没有人调用 hello_cat。
也会给出提示：futures do nothing unless you `.await` or poll them
即 两种解决方法：使用.await语法或者对Future进行轮询(poll)。后者较为复杂

```rust
use futures::executor::block_on;

async fn hello_world() {
    hello_cat().await;     // -----------
    println!("hello, world!");
}

async fn hello_cat() {
    println!("hello, kitty!");
}
fn main() {
    let future = hello_world();
    block_on(future);
}
```
输出的顺序跟代码定义的顺序完全符合，因此，我们在上面代码中使用同步的代码顺序实现了异步的执行效果，非常简单、高效，而且很好理解，未来也绝对不会有回调地狱的发生。

。。？输出顺序==定义的顺序，为什么说 异步的效果？

总之，在async fn函数中使用.await可以等待另一个异步调用的完成。但是与block_on不同，==.await并不会阻塞当前的线程==，而是异步的等待Future A的完成，在等待的过程中，该线程还可以继续执行其它的Future B，最终实现了并发处理的效果。



#### 底层探秘: Future 执行器与任务调度

Future 特征是 Rust 异步编程的核心，毕竟异步函数是异步编程的核心，而 Future 恰恰是异步函数的返回值和被执行的关键。

首先，来给出 Future 的定义：它是一个能产出值的异步计算(虽然该值可能为空，例如 () )。光看这个定义，可能会觉得很空洞，我们来看看一个简化版的 Future 特征:
```rust
trait SimpleFuture {
    type Output;
    fn poll(&mut self, wake: fn()) -> Poll<Self::Output>;
}

enum Poll<T> {
    Ready(T),
    Pending,
}
```

在上一章中，我们提到过 Future 需要被执行器poll(轮询)后才能运行，诺，这里 poll 就来了，通过==调用该方法，可以推进 Future 的进一步执行，直到被切走为止==( 这里不好理解，但是你只需要知道 ==Future 并不能保证在一次 poll 中就被执行完==，后面会详解介绍)。

若在当前 poll 中， Future 可以被完成，则会返回 Poll::Ready(result) ，反之则返回 Poll::Pending， 并且安排一个 wake 函数：当未来 Future 准备好进一步执行时， 该函数会被调用，然后管理该 Future 的执行器(例如上一章节中的block_on函数)会再次调用 poll 方法，此时 Future 就可以继续执行了。


。状态机


使用 Waker 来唤醒任务

构建一个定时器

执行器 Executor

执行器和系统 IO

怎么才能知道 socket 中的数据已经可以被读取了？
通过操作系统提供的 IO 多路复用机制来完成，例如 Linux 中的 epoll，FreeBSD 和 macOS 中的 kqueue ，Windows 中的 IOCP, Fuchisa中的 ports 等


#### 定海神针 Pin 和 Unpin

Pin 可以防止一个类型在内存中被移动

```rust
pub struct Pin<P> {
    pointer: P,
}
```

Unpin 是 特征


Pin把值固定到栈上

把值固定到堆上



#### async/await 和 Stream 流处理

async/.await 是 Rust 语法的一部分，它在遇到阻塞操作时( 例如 IO )会让出当前线程的所有权而不是阻塞当前线程，这样就允许当前线程继续去执行其它代码，最终实现并发。

有两种方式可以使用 async： async fn 用于声明函数，async { ... } 用于声明语句块，它们会返回一个实现 Future 特征的值:
```rust
// `foo()`返回一个`Future<Output = u8>`,
// 当调用`foo().await`时，该`Future`将被运行，当调用结束后我们将获取到一个`u8`值
async fn foo() -> u8 { 5 }

fn bar() -> impl Future<Output = u8> {
    // 下面的`async`语句块返回`Future<Output = u8>`
    async {
        let x: u8 = foo().await;
        x + 5
    }
}
```
async 是懒惰的，直到被执行器 poll 或者 .await 后才会开始运行，其中后者是最常用的运行 Future 的方法。 当 .await 被调用时，它会尝试运行 Future 直到完成，但是若该 Future 进入阻塞，那就会让出当前线程的控制权。当 Future 后面准备再一次被运行时(例如从 socket 中读取到了数据)，执行器会得到通知，并再次运行该 Future ，如此循环，直到完成。


##### async 的生命周期

async fn 函数如果拥有引用类型的参数，那它返回的 Future 的生命周期就会被这些参数的生命周期所限制:
```rust
async fn foo(x: &u8) -> u8 { *x }

// 上面的函数跟下面的函数是等价的:
fn foo_expanded<'a>(x: &'a u8) -> impl Future<Output = u8> + 'a {
    async move { *x }
}
```
意味着 async fn 函数返回的 Future 必须满足以下条件: 当 x 依然有效时， 该 Future 就必须继续等待( .await ), 也就是说 x 必须比 Future 活得更久。

在一般情况下，在函数调用后就立即 .await 不会存在任何问题，例如foo(&x).await。但是，若 Future 被先存起来或发送到另一个任务或者线程，就可能存在问题了:
```rust
use std::future::Future;
fn bad() -> impl Future<Output = u8> {
    let x = 5;
    borrow_x(&x) // ERROR: `x` does not live long enough
}

async fn borrow_x(x: &u8) -> u8 { *x }
```

其中一个常用的解决方法就是将具有引用参数的 async fn 函数转变成一个具有 'static 生命周期的 Future 。 以上解决方法可以通过将参数和对 async fn 的调用放在同一个 async 语句块来实现:
```rust
use std::future::Future;

async fn borrow_x(x: &u8) -> u8 { *x }

fn good() -> impl Future<Output = u8> {
    async {
        let x = 5;
        borrow_x(&x).await
    }
}
```
如上所示，通过将参数移动到 async 语句块内， 我们将它的生命周期扩展到 'static， 并跟返回的 Future 保持了一致。



##### async move
async 允许我们使用 move 关键字来将环境中变量的所有权转移到语句块内

```rust
// 多个不同的 `async` 语句块可以访问同一个本地变量，只要它们在该变量的作用域内执行
async fn blocks() {
    let my_string = "foo".to_string();

    let future_one = async {
        // ...
        println!("{my_string}");
    };

    let future_two = async {
        // ...
        println!("{my_string}");
    };

    // 运行两个 Future 直到完成
    let ((), ()) = futures::join!(future_one, future_two);
}



// 由于 `async move` 会捕获环境中的变量，因此只有一个 `async move` 语句块可以访问该变量，
// 但是它也有非常明显的好处： 变量可以转移到返回的 Future 中，不再受借用生命周期的限制
fn move_block() -> impl Future<Output = ()> {
    let my_string = "foo".to_string();
    async move {
        // ...
        println!("{my_string}");
    }
}
```

---------------
当.await 遇见多线程执行器



##### Stream 流处理
Stream 特征类似于 Future 特征，但是前者在完成前可以生成多个值，这种行为跟标准库中的 Iterator 特征倒是颇为相似。

```rust
trait Stream {
    // Stream生成的值的类型
    type Item;

    // 尝试去解析Stream中的下一个值,
    // 若无数据，返回`Poll::Pending`, 若有数据，返回 `Poll::Ready(Some(x))`, `Stream`完成则返回 `Poll::Ready(None)`
    fn poll_next(self: Pin<&mut Self>, cx: &mut Context<'_>)
        -> Poll<Option<Self::Item>>;
}
```

。。跳。。



#### 使用 join! 和 select! 同时运行多个 Future

```rust
use futures::join;

async fn enjoy_book_and_music() -> (Book, Music) {
    let book_fut = enjoy_book();
    let music_fut = enjoy_music();
    join!(book_fut, music_fut)
}
```

##### try_join!
由于 join! 必须等待它管理的所有 Future 完成后才能完成，如果你希望在某一个 Future 报错后就立即停止所有 Future 的执行，可以使用 try_join!，特别是当 Future 返回 Result 时:


##### select!
join! 只有等所有 Future 结束后，才能集中处理结果，如果你想同时等待多个 Future ，且任何一个 Future 结束后，都可以立即被处理，可以考虑使用 futures::select!:

```rust
use futures::{
    future::FutureExt, // for `.fuse()`
    pin_mut,
    select,
};

async fn task_one() { /* ... */ }
async fn task_two() { /* ... */ }

async fn race_tasks() {
    let t1 = task_one().fuse();
    let t2 = task_two().fuse();

    pin_mut!(t1, t2);

    select! {
        () = t1 => println!("任务1率先完成"),
        () = t2 => println!("任务2率先完成"),
    }
}
```

##### default 和 complete
select!还支持 default 和 complete 分支:
- complete 分支当所有的 Future 和 Stream 完成后才会被执行，它往往配合 loop 使用，loop 用于循环完成所有的 Future
- default 分支，若没有任何 Future 或 Stream 处于 Ready 状态， 则该分支会被立即执行

```rust
use futures::future;
use futures::select;
pub fn main() {
    let mut a_fut = future::ready(4);
    let mut b_fut = future::ready(6);
    let mut total = 0;

    loop {
        select! {
            a = a_fut => total += a,
            b = b_fut => total += b,
            complete => break,
            default => panic!(), // 该分支永远不会运行，因为 `Future` 会先运行，然后是 `complete`
        };
    }
    assert_eq!(total, 10);
}
```


##### 跟 Unpin 和 FusedFuture 进行交互

##### 在 select 循环中并发


#### 一些疑难问题的解决办法
https://course.rs/advance/async/pain-points-and-workarounds.html


##### 在 async 语句块中使用 ?
async 语句块和 async fn 最大的区别就是前者无法显式的声明返回值，在大多数时候这都不是问题，但是当配合 ? 一起使用时，问题就有所不同:
```rust
async fn foo() -> Result<u8, String> {
    Ok(1)
}
async fn bar() -> Result<u8, String> {
    Ok(1)
}
pub fn main() {
    let fut = async {
        foo().await?;           // <<< ??;;
        bar().await?;
        Ok(())
    };
}
```

显式类型
```rust
let fut = async {
    foo().await?;
    bar().await?;
    Ok::<(), String>(()) // 在这一行进行显式的类型注释
};
```


##### async 函数和 Send 特征
事实上，未实现 Send 特征的变量可以出现在 async fn 语句块中:

即使上面的 foo 返回的 Future 是 Send， 但是在它内部短暂的使用 NotSend 依然是安全的，原因在于它的作用域并没有影响到 .await，下面来试试声明一个变量，然后让 .await 的调用处于变量的作用域中试试

可以将变量声明在语句块内，当语句块结束时，变量会自动被 Drop，这个规则可以帮助我们解决很多借用冲突问题，特别是在 NLL 出来之前。
```rust
async fn foo() {
    {
        let x = NotSend::default();
    }
    bar().await;
}
```

##### 递归使用 async fn
```rust
// foo函数:
async fn foo() {
    step_one().await;
    step_two().await;
}
// 会被编译成类似下面的类型：
enum Foo {
    First(StepOne),
    Second(StepTwo),
}

// 因此 recursive 函数
async fn recursive() {
    recursive().await;
    recursive().await;
}

// 会生成类似以下的类型
enum Recursive {
    First(Recursive),
    Second(Recursive),
}
```
```text
error[E0733]: recursion in an `async fn` requires boxing
 --> src/lib.rs:1:22
  |
1 | async fn recursive() {
  |                      ^ an `async fn` cannot invoke itself directly
  |
  = note: a recursive `async fn` must be rewritten to return a boxed future.
```

```rust
use futures::future::{BoxFuture, FutureExt};

fn recursive() -> BoxFuture<'static, ()> {
    async move {
        recursive().await;
        recursive().await;
    }.boxed()
}
```


##### 在特征中使用 async
在目前版本中，我们还无法在特征中定义 async fn 函数，不过大家也不用担心，目前已经有计划在未来移除这个限制了。

```rust
#[async_trait]      // <<<<
impl Advertisement for Modal {
    async fn run(&self) {
        self.render_fullscreen().await;
        for _ in 0..4u16 {
            remind_user_to_join_mailing_list().await;
        }
        self.hide_for_now().await;
    }
}
```






3



























方法前缀	性能开销	所有权改变
as_	Free	borrowed -> borrowed
to_	Expensive	borrowed -> borrowed
borrowed -> owned (non-Copy types)
owned -> owned (Copy types)
into_	Variable	owned -> owned (non-Copy types)














------------------------
------------------------
# google rust course (。速览)

https://github.com/google/comprehensive-rust


组成
- rustc - rust compiler， 后端使用了 LLVM
- cargo - rust dependency manager and build tool
- rustup - rust toolchain installer and updater. (install/update rustc and cargo)


## rustc cargo command
`rustc --version`
`cargo --version`

-----

`cargo new project01`
`cargo run`
`cargo check`: use `cargo build` to compile without running it. output to `target/debug/`
`cargo build --release`: generate a release build, in `target/release/`

整形溢出，
通过 `overflow-checks` 标记设置，true 溢出就 panic。 false：溢出就wrap-around
- `cargo build`  默认 wrap-around
- `cargo build --relwase`  默认 panic

数组越界检查
无法关闭，unsafe中依然会进行检查。
`unsafe` + `slice::get_unchecked` ，可以 不检查


## 概念和关键字

句法：
- 变量
- scalar 标量类型，数据类型标准量
- 复合类型
- 枚举
- struct
- reference
- function
- method

控制:
- if
- if let
- while
- while let
- break
- continue

模式匹配：
- 解构 enum，struct，array


## compile time gurantess

- 无 未初始化变量
- 无 内存泄漏(mostly)
- 无 double-free
- 无 use-after-free
- 无 NULL pointer
- 无 忘记释放 已加锁的 mutex
- 无 线程间数据竞争
- 无 非法iter

rust可以产生内存泄漏，如：
- Box::leak， 用来获得 运行时初始化，运行时获得size的 static 变量
- std::mem::forget， 让 编译器 忘记一个值，即 析构器不会执行
- 意外地 使用 Rc 或 Arc 形成了 循环引用
- rust 无法防止 无限填充一个集合。


## runtime guarantee
运行时 没有 未定义的行为
- 检查数组是否越界
- 定义整形溢出 (panic 或 wrap-around)

整形溢出，
通过 `overflow-checks` 标记设置，true 溢出就 panic。 false：溢出就wrap-around
- `cargo build`  默认 wrap-around
- `cargo build --relwase`  默认 panic

数组越界检查
无法关闭，unsafe中依然会进行检查。
`unsafe` + `slice::get_unchecked` ，可以 不检查


## modern feature

language feature
- enum, pattern match
- generic 泛型
- no overhead FFI(foreign function interface)， 调用其他语言的函数 没有额外开销？
- zero-cost abstraction。 0代价抽象

工具
- 优秀的编译器错误提示
- 内置 依赖管理
- 内置 对于测试 的支持
- 优秀的 language server protocol 支持


0开销抽象： 不需要为 高级编程语言结构 支付 内存和CPU。例如， 使用 for 循环和 使用 .iter().fold() 的 代价时一样的。

Rust enum 是 Algebraic Data Types， 也被称为 Sum types. 允许类型系统 使用 `Option<T>` 和 `Result<T, E>` 来表达。
。。sum types:  和类型， Sum Type是一种可以容纳不同类型值的数据结构，只能同时拥有其中一种类型的值。

`rust-analyzer` is a well supported LSP implementation used in major IDEs and text editors.



## 基础语法

// 111
/* 111 */

### scalar type

|name|type|literal|
|--|--|--|
|signed integer|i8,i16,i32,i64,i128,isize|-10,0,1_000,123_i64|
|unsigned interger|u8,u16,u32,u64,u128,usize|0,123,10_u16|
|floating point number|f32,f64|3.14,-10.0e20,2_f32|
|string|&str|"foo","two\nline"|
|unicode scalar value|char|'a', 'α', '∞'|
|boolean|bool|true,false|

type宽度
iN,uN,fN,  N bits wide
isize, usize,   pointer的宽度
char, 32bit
bool, 8bit

string还可以使用 r，而不是 转移字符： r"\n" == "\\n"
在两侧加上 相同数量的 # ， 可以不转义 双引号
```rust
fn main() {
    println!(r#"<a href="link.html">link</a>"#);
    println!("<a href=\"link.html\">link</a>");
}
```

byte string 可以让你创建一个 `&[u8]`
```rust
fn main() {
    println!("{:?}", b"abc");
    println!("{:?}", &[97, 98, 99]);
}
```

下划线可以移除
1_000 == 10_00 == 1000
`123_i32 == 123i32`


### compound type
|name|type|literal|
|--|--|--|
|array|[T; N]|[20,30,40], [0; 3]|
|tuple|(), (T,), (T1, T2) ...| (), ('x', ), ('x', 1.2), ...|


```rust
fn main() {
    let mut a: [i8; 10] = [42; 10];
    a[5] = 0;
    println!("a: {a:?}");
}
```

```rust
fn main() {
    let t: (i8, bool) = (7, true);
    println!("t.0: {}", t.0);
    println!("t.1: {}", t.1);
}
```


Adding `#`, eg `{a:#?}`, invokes a “pretty printing” format


空tuple `()`， 被称为 unit type， 值只有一个。
通常用来，表明 方法 或 表达式 没有返回值。
你可以认为它是 void


### reference

```rust
fn main() {
    let mut x: i32 = 10;
    let ref_x: &mut i32 = &mut x;
    *ref_x = 20;
    println!("x: {x}");
}
```

需要 解引用 才能对它进行赋值操作。 类似于 C++的指针
有些场合，rust 会自动 解引用，如 `ref_x.count_ones()`
引用声明为 mut， 那么可以 绑定到不同的 值。

`let mut ref_x: &i32` `let ref_x: &mut i32`
前者是 mut ref，可以绑定到不同的 不可变 i32 变量。
后者是 不可变ref，绑定到 mut i32 变量


#### dangling(悬垂) reference

rust 静止 悬垂ref
```rust
fn main() {
    let ref_x: &i32;
    {
        let x: i32 = 10;
        ref_x = &x;
    }
    println!("ref_x: {ref_x}");
}
```


### slice

```rust
fn main() {
    let mut a: [i32; 6] = [10, 20, 30, 40, 50, 60];
    println!("a: {a:?}");

    let s: &[i32] = &a[2..4];

    println!("s: {s:?}");
}
```

如果在 println! 之前 修改 a[3] 的值会发生什么？
为了内存安全，你不能修改 a[3]。
只能在 创建 s 之前 或 println! 之后  修改a[3]


如果 slice 从 下标0 开始，可以省略。  `&a[0..a.len()]` == `&a[..a.len()]`
如果 一直到最后一个位置， 那么 也可以省略  `&a[2..a.len()]` == `&a[2..]`
可以两边都省略，表明包含全部， `&a[..]`

切片总是 从 其他对象 借用者， 所以 生命周期要 匹配。


#### String vs str

```rust
fn main() {
    let s1: &str = "World";
    println!("s1: {s1}");

    let mut s2: String = String::from("Hello ");
    println!("s2: {s2}");
    s2.push_str(s1);
    println!("s2: {s2}");
    
    let s3: &str = &s2[6..];
    println!("s3: {s3}");
}
```

- &str: an immutable reference to a string slice.
- String: a mutable string buffer.

&str 是 slice， 是 UTF-8编码的 字符数据 的 不可变 ref，保存在 内存块中。
字符串常量 ("hello") 被保存在 代码的二进制中。
String 是 byte vector 的封装， 比如 `Vec<T>`， String有 所有权。

String::from(), 从 字符串常量 创建一个 String对象。
String::new()， 创建一个 空的 String对象，后续可以 push(), push_str()。
format!() 宏，可以用来 从动态值 创建 String对象。 它的格式 和 println! 一样。

通过 & 和 范围， 可以从 String对象 借到 `&str` 切片，

对于C++开发人员： 可以认为： 
`&str` == `const char *`, 除了 &str 始终指向了内存中 有效的 字符串。
`String` == `std::string`， 除了 rust的 String 只能保存 UTF-8 编码的 byte， 且不会使用 small-string 优化。

。。sso， small string optimization,  当字符串较短时，保存在栈中。。当分配大小小于16个字节时候,从栈上进行分配,而如果大于等于16个字节,则在堆上进行。



### function

```rust
fn main() {
    print_fizzbuzz_to(20);
}

fn is_divisible(n: u32, divisor: u32) -> bool {
    if divisor == 0 {
        return false;
    }
    n % divisor == 0
}

fn fizzbuzz(n: u32) -> String {
    let fizz = if is_divisible(n, 3) { "fizz" } else { "" };
    let buzz = if is_divisible(n, 5) { "buzz" } else { "" };
    if fizz.is_empty() && buzz.is_empty() {
        return format!("{n}");
    }
    format!("{fizz}{buzz}")
}

fn print_fizzbuzz_to(n: u32) {
    for i in 1..=n {
        println!("{}", fizzbuzz(i));
    }
}
```

没有声明 返回值，编译器默认 `-> ()`


#### RustDoc
Rust中所有的 language item 都可以使用  `///` 来添加文档。

```rust
/// Determine whether the first argument is divisible by the second argument.
///
/// If the second argument is zero, the result is false.
///
/// # Example
/// ```
/// assert!(is_divisible_by(42, 2));
/// ```
fn is_divisible_by(lhs: u32, rhs: u32) -> bool {
    if rhs == 0 {
        return false;  // Corner case, early return
    }
    lhs % rhs == 0     // The last expression in a block is the return value
}
```

内容被认为是 markdown.
使用 `rustdoc` 工具可以导出。


#### method (带self)

有 self 的是 method。

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn inc_width(&mut self, delta: u32) {       // 。。&mut self
        self.width += delta;
    }
}

fn main() {
    let mut rect = Rectangle { width: 10, height: 5 };
    println!("old area: {}", rect.area());
    rect.inc_width(5);
    println!("new area: {}", rect.area());
}
```

fn new(width: u32, height: u32) -> Rectangle {
    Rectangle { width, height }
}

Rectangle::square(width: u32)



#### function overload (。rust不支持)
不支持 overload
不支持 型参默认值 。 有时，可以使用宏来完成。

支持范型

```rust
fn pick_one<T>(a: T, b: T) -> T {
    if std::process::id() % 2 == 0 { a } else { b }
}

fn main() {
    println!("coin toss: {}", pick_one("heads", "tails"));
    println!("cash prize: {}", pick_one(500, 1000));
}
```

使用范型时， 标准库的 `Into<T>` 可以为 参数类型 提供 一种受限的 多态。



### 测试
类型转换，i8 转为 i16
2种方法
`i16::from(x)`
`x.into()`




## 控制流

### block

rust中的 block 包含了 一些表达式， 每个块 都有 一个值 和一个类型，来自 block的 最后一个 表达式。
如果最后一个表达式是 ; 结尾，那么 返回值和类型是 `()`

。。expression, statement



### if (语句 或==表达式==)

```rust
fn main() {
    let mut x = 10;
    if x % 2 == 0 {     // 语句
        x = x / 2;  // ; 语句
    } else {
        x = 3 * x + 1;
    }
}
```

```rust
fn main() {
    let mut x = 10;
    x = if x % 2 == 0 {     // 表达式
        x / 2       // no ;  表达式
    } else {
        3 * x + 1
    };
}
```


### for

#### 。。step_by

```rust
fn main() {
    let v = vec![10, 20, 30];

    for x in v {
        println!("x: {x}");
    }
    
    for i in (0..10).step_by(2) {
        println!("i: {i}");
    }
}
```

修改，需要将 v 设置为 mut， 然后使用 `for x in v.iter_mut()`


### while

和其它语言差不多

```rust
fn main() {
    let mut x = 10;
    while x != 1 {
        x = if x % 2 == 0 {
            x / 2
        } else {
            3 * x + 1
        };
    }
    println!("x: {x}");
}
```


### break, continue

都可以通过 label 来 break/continue 多层。

```rust
fn main() {
    let v = vec![10, 20, 30];
    let mut iter = v.into_iter();
    'outer: while let Some(x) = iter.next() {
        println!("x: {x}");
        let mut i = 0;
        while i < x {
            println!("x: {x}, i: {i}");
            i += 1;
            if i == 3 {
                break 'outer;
            }
        }
    }
}
```

。。'outer  是生命周期吗？ 为什么 ' ？ ， 还是说 随便加的？  就是可以直接 AAA 吗？


### loop

endless loop

必须有 break 或 return

```rust
fn main() {
    let mut x = 10;
    loop {
        x = if x % 2 == 0 {
            x / 2
        } else {
            3 * x + 1
        };
        if x == 1 {
            break;
        }
    }
    println!("x: {x}");
}
```

尝试使用 `break 8` 来break loop。 然后打印出来。
。。？

loop 需要返回一个 non-trivial 值， 因为 它被认为 会至少进入一次 (这点 和 while 和 for 不同)
。。



## variable

### type interface

rust 会基于 变量的使用 来决定 类型

```rust
fn takes_u32(x: u32) {
    println!("u32: {x}");
}

fn takes_i8(y: i8) {
    println!("i8: {y}");
}

fn main() {
    let x = 10;
    let y = 20;

    takes_u32(x);
    takes_i8(y);
    // takes_u32(y);
}
```

```rust
fn main() {
    let mut v = Vec::new();
    v.push((10, false));
    v.push((20, true));
    println!("v: {v:?}");

    let vv = v.iter().collect::<std::collections::HashSet<_>>();
    println!("vv: {vv:?}");
}
```


### static, constanct
2种 生成 全局变量 (运行时 不可move 或 不可重新赋值) 的 方式。

#### const
常量变量 编译期 eval， 值被==内联==到 使用的地方。

。。内链 所以 没有这个对象。

```rust
const DIGEST_SIZE: usize = 3;
const ZERO: Option<u8> = Some(42);

fn compute_digest(text: &str) -> [u8; DIGEST_SIZE] {
    let mut digest = [ZERO.unwrap_or(0); DIGEST_SIZE];
    for (idx, &b) in text.as_bytes().iter().enumerate() {
        digest[idx % DIGEST_SIZE] = digest[idx % DIGEST_SIZE].wrapping_add(b);
    }
    digest
}

fn main() {
    let digest = compute_digest("Hello");
    println!("digest: {digest:?}");
}
```

#### static
静态变量在 程序的整个生命周期中 都存在， 因此不能 move

```rust
static BANNER: &str = "Welcome to RustOS 3.14";

fn main() {
    println!("{BANNER}");
}
```

不会被 内联， 所以有一个 真实的 内存地址。 对于 unsafe 和 embeded 代码 是 有用的。
当一个 全局范围的 值 不需要 object identity 时， const 更好。

由于 static 变量 可以被不同 线程访问，所以 必须是 `Sync` 的。 内部可变性 通过 Mutex, atomic 等 实现。

可以存在 mut的 static， 必须 手工同步， 所以 任何访问 它们的代码 都是 unsafe 的。
会在 unsafe rust 章节看到 mutable static


-------------

const 类似 C++的 constexpr
static 类似 C++的 const 或 全局的可变变量。
static 提供了 object identity: 内存中的 一个地址， 和 具有 内部可变性的类型 所需的状态 (如 `Mutex<T>`)
需要运行时 eval的 constant 不常见，但是 它比 static 更 helpful 和 safer。

使用 `std::thread_local` 宏 创建 thread_local 数据。


|属性|static|const|
|--|--|--|
|有内存地址|yes|no(inlined)|
|生命周期和 程序的一样|yes|no|
|可以mut|yes(unsafe)|no|
|编译时eval|yes(编译时初始化)|yes|
|inline|no|yes|


### scope and shadow

```rust
fn main() {
    let a = 10;
    println!("before: {a}");

    {
        let a = "hello";
        println!("inner scope: {a}");

        let a = true;
        println!("shadowed in inner scope: {a}");
    }

    println!("after: {a}");
}
```


## enum

创建有 不同 variant 的 type。

```rust
fn generate_random_number() -> i32 {
    // Implementation based on https://xkcd.com/221/
    4  // Chosen by fair dice roll. Guaranteed to be random.
}

#[derive(Debug)]
enum CoinFlip {
    Heads,
    Tails,
}

fn flip_coin() -> CoinFlip {
    let random_number = generate_random_number();
    if random_number % 2 == 0 {
        return CoinFlip::Heads;
    } else {
        return CoinFlip::Tails;
    }
}

fn main() {
    println!("You got: {:?}", flip_coin());
}
```



### ==variant payload==

```rust
enum WebEvent {
    PageLoad,                 // Variant without payload
    KeyPress(char),           // Tuple struct variant
    Click { x: i64, y: i64 }, // Full struct variant
}

#[rustfmt::skip]
fn inspect(event: WebEvent) {
    match event {
        WebEvent::PageLoad       => println!("page loaded"),
        WebEvent::KeyPress(c)    => println!("pressed '{c}'"),
        WebEvent::Click { x, y } => println!("clicked at x={x}, y={y}"),
    }
}

fn main() {
    let load = WebEvent::PageLoad;
    let press = WebEvent::KeyPress('x');
    let click = WebEvent::Click { x: 20, y: 80 };

    inspect(load);
    inspect(press);
    inspect(click);
}
```

可以通过 `std::mem::discriminant()` 获得 discriminant(判别式)


### enum size

#### ==type_name==, size_of, align_of


在考虑了 内存对齐后，enum 被 紧密地 打包在一起，

```rust
use std::any::type_name;
use std::mem::{align_of, size_of};

fn dbg_size<T>() {
    println!("{}: size {} bytes, align: {} bytes",
        type_name::<T>(), size_of::<T>(), align_of::<T>());
}

enum Foo {
    A,
    B,
}

fn main() {
    dbg_size::<Foo>();
}
```

https://doc.rust-lang.org/reference/type-layout.html


rust内部使用 一个field(discriminant) 来 保持 enum variant 的追踪
你可以控制 discriminant 
```rust
#[repr(u32)]
enum Bar {
    A,  // 0
    B = 10000,
    C,  // 10001
}

fn main() {
    println!("A: {}", Bar::A as u32);
    println!("B: {}", Bar::B as u32);
    println!("C: {}", Bar::C as u32);
}
```
没有 `repr`, discriminant type 占据 2个byte， 因为 10001 满足2个 byte

尝试其他类型，如
- dbg_size!(bool)
  size 1 byte, align 1 byte
- `dbg_size!(Option<bool>)`
  size 1, align 1
- dbg_size!(&i32)
  size 8, align 8
- `dbg_size!(Option<&i32>)`
  size 8, align 8


rust有几个优化，用来使得 枚举 占用的空间更小
- niche 优化
  rust 会为 enum discriminant 合并没有使用的 bit pattern
- null pointer 优化，对于一些类型， `rust 保证 size_of::<T>()` 等于`size_of::<Option<T>>()`  ； https://doc.rust-lang.org/std/option/#representation


。。
enum通常需要一个标志（discriminant）来区分究竟是哪一个variant。
。。

#### ==bit布局==
```rust
use std::mem::transmute;

macro_rules! dbg_bits {
    ($e:expr, $bit_type:ty) => {
        println!("- {}: {:#x}", stringify!($e), transmute::<_, $bit_type>($e));
    };
}

fn main() {
    unsafe {
        println!("bool:");
        dbg_bits!(false, u8);
        dbg_bits!(true, u8);

        println!("Option<bool>:");
        dbg_bits!(None::<bool>, u8);
        dbg_bits!(Some(false), u8);
        dbg_bits!(Some(true), u8);

        println!("Option<Option<bool>>:");
        dbg_bits!(Some(Some(false)), u8);
        dbg_bits!(Some(Some(true)), u8);
        dbg_bits!(Some(None::<bool>), u8);
        dbg_bits!(None::<Option<bool>>, u8);

        println!("Option<&i32>:");
        dbg_bits!(None::<&i32>, usize);
        dbg_bits!(Some(&0i32), usize);
    }
}
```

更复杂的例子， 当我们把 超过 256个 Option chain一起，会发生什么

```rust
#![recursion_limit = "1000"]

use std::mem::transmute;

macro_rules! dbg_bits {
    ($e:expr, $bit_type:ty) => {
        println!("- {}: {:#x}", stringify!($e), transmute::<_, $bit_type>($e));
    };
}

// Macro to wrap a value in 2^n Some() where n is the number of "@" signs.
// Increasing the recursion limit is required to evaluate this macro.
macro_rules! many_options {
    ($value:expr) => { Some($value) };
    ($value:expr, @) => {
        Some(Some($value))
    };
    ($value:expr, @ $($more:tt)+) => {
        many_options!(many_options!($value, $($more)+), $($more)+)
    };
}

fn main() {
    // TOTALLY UNSAFE. Rust provides no guarantees about the bitwise
    // representation of types.
    unsafe {
        assert_eq!(many_options!(false), Some(false));
        assert_eq!(many_options!(false, @), Some(Some(false)));
        assert_eq!(many_options!(false, @@), Some(Some(Some(Some(false)))));

        println!("Bitwise representation of a chain of 128 Option's.");
        dbg_bits!(many_options!(false, @@@@@@@), u8);
        dbg_bits!(many_options!(true, @@@@@@@), u8);

        println!("Bitwise representation of a chain of 256 Option's.");
        dbg_bits!(many_options!(false, @@@@@@@@), u16);
        dbg_bits!(many_options!(true, @@@@@@@@), u16);

        println!("Bitwise representation of a chain of 257 Option's.");
        dbg_bits!(many_options!(Some(false), @@@@@@@@), u16);
        dbg_bits!(many_options!(Some(true), @@@@@@@@), u16);
        dbg_bits!(many_options!(None::<bool>, @@@@@@@@), u16);
    }
}
```



## novel control flow

novel： 小说，不同的，新颖的

### if let

根据 值来进行 匹配，运行 不同的代码

```rust
fn main() {
    let arg = std::env::args().next();
    if let Some(value) = arg {
        println!("Program name: {value}");
    } else {
        println!("Missing name?");
    }
}
```

和 match 不同， if let 不需要 覆盖所有的 分支。 所以比 match 更 简约

常见的用法是， 处理 Option 的 Some 值

和match不同， if let 不支持  模式匹配 的 guard子句。

#### let else

从 1.65开始，  let else 结构 可以进行 destructuring assignment，或者，当失败时，执行 一个block，它会 (通过 panic,return,break,continue) 终止 正常的控制流 
```rust
fn main() {
    println!("{:?}", second_word_to_upper("foo bar"));
}
 
fn second_word_to_upper(s: &str) -> Option<String> {
    let mut it = s.split(' ');
    let (Some(_), Some(item)) = (it.next(), it.next()) else {
        return None;
    };
    Some(item.to_uppercase())
}
```


### while let

和 if let 类似，只不过这个 会重复 测试 值和pattern

```rust
fn main() {
    let v = vec![10, 20, 30];
    let mut iter = v.into_iter();

    while let Some(x) = iter.next() {
        println!("x: {x}");
    }
}
```

每次调用 next() 时， 会返回一个 `Option<i32>`, 返回 Some(x) 直到 None。 while let 会遍历完 全部 item。

while let 会一直运行，只要 值 能匹配 pattern



### match

用来 匹配 value 到 pattern， 就像 一系列的 if let 表达式

```rust
fn main() {
    match std::env::args().next().as_deref() {
        Some("cat") => println!("Will do cat things"),
        Some("ls")  => println!("Will ls some files"),
        Some("mv")  => println!("Let's move some files"),
        Some("rm")  => println!("Uh, dangerous!"),
        None        => println!("Hmm, no program name?"),
        _           => println!("Unknown program name!"),
    }
}
```

`as_deref()`
- std::env::args().next() returns an `Option<String>`, but we cannot match against String.
- as_deref() transforms an `Option<T>` to `Option<&T::Target>`. In our case, this turns `Option<String>` into `Option<&str>`.
- We can now use pattern matching to match against the &str inside Option.


## pattern match

match 可以 将 1个值 和 模式 进行匹配， 这个比较 是从上往下， 找到 第一个匹配。

模式可以是 简单值， 就像 C++ 的 switch
```rust
fn main() {
    let input = 'x';

    match input {
        'q'                   => println!("Quitting"),
        'a' | 's' | 'w' | 'd' => println!("Moving around"),
        '0'..='9'             => println!("Number input"),
        _                     => println!("Something else"),
    }
}
```

- `|` 就是 逻辑或
- `..` 是range;  1..=5是[1,5],   1..5是[1,5)
- `_` 匹配任何值


### ==destructuring enum==

https://google.github.io/comprehensive-rust/pattern-matching/destructuring-enums.html

pattern 可以用来 绑定 值中 的 一部分。

```rust
enum Result {
    Ok(i32),
    Err(String),
}

fn divide_in_two(n: i32) -> Result {
    if n % 2 == 0 {
        Result::Ok(n / 2)
    } else {
        Result::Err(format!("cannot divide {n} into two equal parts"))
    }
}

fn main() {
    let n = 100;
    match divide_in_two(n) {        // -----------方法
        Result::Ok(half) => println!("{n} divided in two is {half}"),
        Result::Err(msg) => println!("sorry, an error happened: {msg}"),
    }
}
```

解构 Result 值， 
第一个分支， half 绑定了 Ok 这个variant 中的 值。
第二个分支， msg 绑定了 error message。( 。。而不是 Err)




### Destructuring Structs

```rust
struct Foo {
    x: (u32, u32),
    y: u32,
}

#[rustfmt::skip]
fn main() {
    let foo = Foo { x: (1, 2), y: 3 };
    match foo {
        Foo { x: (1, b), y } => println!("x.0 = 1, b = {b}, y = {y}"),
        Foo { y: 2, x: i }   => println!("y = 2, x = {i:?}"),
        Foo { y, .. }        => println!("y = {y}, other fields were ignored"),
    }
}
```


### Destructuring Arrays


```rust
#[rustfmt::skip]
fn main() {
    let triple = [0, -2, 3];
    println!("Tell me about {triple:?}");
    match triple {
        [0, y, z] => println!("First is 0, y = {y}, and z = {z}"),
        [1, ..]   => println!("First is 1 and the rest were ignored"),
        _         => println!("All elements were ignored"),
    }
}
```

slice, &&&&

```rust
fn main() {
    inspect(&[0, -2, 3]);
    inspect(&[0, -2, 3, 4]);
}

#[rustfmt::skip]
fn inspect(slice: &[i32]) {
    println!("Tell me about {slice:?}");
    match slice {
        &[0, y, z] => println!("First is 0, y = {y}, and z = {z}"),
        &[1, ..]   => println!("First is 1 and the rest were ignored"),
        _          => println!("All elements were ignored"),
    }
}
```

patterns [.., b] and [a@..,b]
。。可以试下， 这2个 是 pattern 吗？ 主要是 第二个，  有 @ ..



### ==Match Guard== 增加条件

增加 guard 到 pattern 上。 guard 是 任意的 bool 表达式。 如果 pattern 匹配，那么会 执行 这个 bool 表达式。

。。先 pattern， 然后 bool。

```rust
#[rustfmt::skip]
fn main() {
    let pair = (2, -2);
    println!("Tell me about {pair:?}");
    match pair {
        (x, y) if x == y     => println!("These are twins"),
        (x, y) if x + y == 0 => println!("Antimatter, kaboom!"),
        (x, _) if x % 2 == 1 => println!("The first one is odd"),
        _                    => println!("No correlation..."),
    }
}
```

The condition defined in the guard applies to every expression in a pattern with an |

。。不是 || ?

。。不知道：
(x,y) if x==y | (x,1) if x==1
(x,y) | (x,1) if x==y | x== 1
哪个 能编译。




## 内存管理
以前，内存管理分为2类
- 手工控制： C,C++,Pascal
- 运行时自动控制: Java,Python,Go,Haskell

rust是一种混合
在编译时 通过 正确的内存管理 来获得 完整的控制和安全。

通过 显式的 所有权 来完成。


### stack vs heap

https://google.github.io/comprehensive-rust/memory-management/stack-vs-heap.html

stack： 保存local变量的 连续的内存
- 固定大小：编译时 就知道 大小。
- 极其快，只需要移动 stack pointer
- 便于管理：follow function call
- great memory locality

。。这个 stack 和 数据结构的stack 不一样， 主要是 这里 能移动 指针，  数据结构stack 只能操作 top。

heap： 保存 方法 外的 value
- 动态大小： 运行时 才知道
- 比stack慢一点点： 需要一些 book-keeping
- 不保证 memory locality


### stack and heap example

创建一个 string 对象， 会 
- 在 stack上生成 固定size的 metadata， 
- 在 heap上 生成 动态size 的 数据

```rust
fn main() {
    let s1 = String::from("Hello");
}
```

![1827f0a5f432269e4123f279454ee816.png](resources/d351068bb87b4f9983eefc20382db2a1.png)


```rust
fn main() {
    let mut s1 = String::from("Hello");
    s1.push(' ');
    s1.push_str("world");
    // DON'T DO THIS AT HOME! For educational purposes only.
    // String provides no guarantees about its layout, so this could lead to
    // undefined behavior.
    unsafe {
        let (ptr, capacity, len): (usize, usize, usize) = std::mem::transmute(s1);
        println!("ptr = {ptr:#x}, len = {len}, capacity = {capacity}");
    }
}
```


### manual memory management

你需要自己 申请 和 释放 heap 内存

```c
void foo(size_t n) {
    int* int_array = malloc(n * sizeof(int));
    //
    // ... lots of code
    //
    free(int_array);
}
```
。。举了一个 c 的  malloc  free 的例子，就没了。 就是在 free 之前 有异常，会导致 内存泄漏。

。。估计 rust 没有这个功能，或者是 unsafe 的， 只是用来和 下面进行对比。


### scope-based memory management (RAII)

构造器 和 析构器 可以让你 hook 到 对象的生命周期中。
通过把 指针放到 对象中， 可以在 对象析构的时候 释放内存。 编译器保证 析构的发生，即使出现 异常。

这个被称为 resource acquisition is initialization (RAII) ， 给你 智能指针。

```c++
void say_hello(std::unique_ptr<Person> person) {
  std::cout << "Hello " << person->name << std::endl;
}
```
传递到方法中时，需要使用 移动构造器
```c++
std::unique_ptr<Person> person = find_person("Carla");
say_hello(std::move(person));
```

### garbage collection
- 不需要 显式 申请，释放 内存
- gc 会自动 发现 未使用的内存 并释放。

```java
void sayHello(Person person) {
  System.out.println("Hello " + person.getName());
}
```


### rust memory management

是一个混合
- 和java一样安全，正确，但不需要 gc
- 和c++一样，基于scope，且 编译器强制完全遵守
- rust用户可以对 不同场景选择 正确的抽象，有时 甚至在 运行时没有成本，如C。

rust通过 对 所有权 显式建模 来实现 内存管理。

rust通常通过 RAII wrapper，如 Box，Vec，Rc，Arc。

析构就是 Drop trait



## ==ownership==

所有的变量 都有一个 scope，在这个 scope中，它是有效的，在 这个 scope外使用 是错误的。

```rust
struct Point(i32, i32);

fn main() {
    {
        let p = Point(3, 4);
        println!("x: {}", p.0);
    }
    println!("y: {}", p.1);
}
```

- 在scope 结束的地方， 变量被 drop， 数据被释放
- 析构器 释放资源
- 我们称之为 变量 own value。


### move semantics(语义)

赋值 会 在 变量间 转移 ownership

```rust
fn main() {
    let s1: String = String::from("Hello!");
    let s2: String = s1;
    println!("s2: {s2}");
    // println!("s1: {s1}");
}
```

- s1 到 s2的赋值 ，转移了 所有权
- s1 出scope时， 不会发生任何事情，因为 它 own nothing
- s2 出scope时， 字符串数据 被 free
- 对于一个 value， 任何时候， ==有且只有一个== 变量 own 它。


和c++不同，c++的赋值，默认是 copy， 除非 显式 std::move (且 定义了移动构造器)

只移动了 所有权，

简单的value 可以被 标记为 Copy

rust中， clone 是 显式的 (通过 使用 clone)。


### move string

https://google.github.io/comprehensive-rust/ownership/moved-strings-rust.html

```rust
fn main() {
    let s1: String = String::from("Rust");
    let s2: String = s1;
}
```

- s1 的 heap 数据，被 s2 重用了
- s1 出scope时，不会发生任何事情

。看图， 是 stack中 s1,s2 都指向了 同一个heap的地址。


#### double free in modern c++

modern c++ 使用不同的方式 解决下面的问题。

```C++
std::string s1 = "Cpp";
std::string s2 = s1;  // Duplicate the data in s1.
```

- s1的 heap 的数据 被 复制了一份 给s2, s2拥有了一份独立的数据的 所有权
- 当s1,s2 出scope时， 都  会 free 内存。

。看图，就是， stack 中，s1,s2 分别指向 不同的 heap中的 地址


- C++中 `=` 是 copy
- `s2 = std::move(s1)`, 不会发生 内存的 申请。 执行后， s1 处于 有效但未指定的 状态。 和rust不同， c++中可以继续使用 s1.
- 和rust不同， C++中的 "=" 可以根据被复制或移动的类型 运行任意的代码



### move in function call

当你传递 value 到 function，value 被赋值到 函数参数上 ， 这会 ==转移所有权==。

```rust
fn say_hello(name: String) {
    println!("Hello {name}")
}

fn main() {
    let name = String::from("Alice");
    say_hello(name);
    // say_hello(name);
}
```

- 第一次调用 say_hello 时， main 放弃了 name 的 所有权， 所以 后续不能再使用了。
- name 的 heap内存 会在 ==say_hello 的 结束处== free
- 修改代码，传递 ref (&name)， main可以 维持 所有权。
- 或者， main 可以传递 name的clone (name.clone())
- rust==默认使用 move语义==，并且==需要 显式 clone==，使得 很难 无意中 创建 copy。


### copy and clone
虽然默认使用 move语义， 但是 有些type 默认是 copy。

```rust
fn main() {
    let x = 42;
    let y = x;
    println!("x: {x}");
    println!("y: {y}");
}
```
这些类型 实现了 ==Copy trait==

你可以让你自己的类 使用 copy 语义
```rust
#[derive(Copy, Clone, Debug)]
struct Point(i32, i32);

fn main() {
    let p1 = Point(3, 4);
    let p2 = p1;
    println!("p1: {p1:?}");
    println!("p2: {p2:?}");
}
```

- 在赋值后，p1,p2 有各自的数据
- 我们可以使用 p1.clone() 来 显式 copy数据


#### copy trait vs clone trait
- copy 是 对内存区域的 按bit复制， 不是 所有 对象都 可以 copy
- copy 不允许 自定义逻辑 ( 不像 C++的 复制构造器)
- copy 对于实现了 Drop trait 的 类型 无效
- clone 是一个更一般的操作， 通过实现 Clone trait 来 自定义操作


`derive`：在编译时 生产代码，在上面的例子中， 生成了 Copy 和 Clone trait的 默认实现。

1. 在 `struct Point` 中增加 `String`, 编译会失败，因为 String 没有 Copy trait
2. 如果移除 derive 中的 Copy， 编译时 println! 会报错。
3. println! 中使用 显式 clone，可以运行。


### borrow

调用方法时， 除了 转移 所有权， 还可以让 function borrow the value。

。。ref

```rust
#[derive(Debug)]
struct Point(i32, i32);

fn add(p1: &Point, p2: &Point) -> Point {
    Point(p1.0 + p2.0, p1.1 + p2.1)
}

fn main() {
    let p1 = Point(3, 4);
    let p2 = Point(10, 20);
    let p3 = add(&p1, &p2);
    println!("{p1:?} + {p2:?} = {p3:?}");
}
```

- add 方法 borrow 2个 point， 并且 返回一个 新的 point
- 调用者 依然 有 实参值的 所有权。


```rust
#[derive(Debug)]
struct Point(i32, i32);

fn add(p1: &Point, p2: &Point) -> Point {
    let p = Point(p1.0 + p2.0, p1.1 + p2.1);
    println!("&p.0: {:p}", &p.0);
    p
}

pub fn main() {
    let p1 = Point(3, 4);
    let p2 = Point(10, 20);
    let p3 = add(&p1, &p2);
    println!("&p3.0: {:p}", &p3.0);
    println!("{p1:?} + {p2:?} = {p3:?}");
}
```

- 从add 方法中 返回值 是cheap的，因为 编译器可以 消除 copy操作。
  如上的代码中，增加了打印 stack 地址， 在 DEBUG 优化级别， 2个地址是不同的， 在 RELEASE级别，2个地址是相同的。
- rust编译器可以执行 RVO (return value optimization)
- 在C++ 中， 复制省略 必须在 语言规范中定义， 因为 构造器 可以有 副作用。 在RUST中，这不是问题。如果没有发生RVO，那么rust 始终是 `memcpy` 复制，简单且高效


#### shared and unique borrow

对于 borrow 有限制
- 你可以有一个或多个 &T
- 或者 你 只有 一个 &mut T 值

```rust
fn main() {
    let mut a: i32 = 10;
    let b: &i32 = &a;

    {
        let c: &mut i32 = &mut a;
        *c = 20;
    }

    println!("a: {a}");
    println!("b: {b}");
}
```
- 上面 编译失败，因为 同一时间， a被c borrow 为 &mut, 又被b borrwo 为 & 
- 把 println("{b}") 放到 let c 所在的 scope 前， 编译成功
- 因为在移动 println后， 编译器 意识到 b 只有在 c的scope前 被使用了， 所以 b,c 的scope 没有重叠。这个 borrow checker 的功能 被成为 "non-lexical lifetimes"



### lifetime

被借用 的值 有如下的 生命周期
- 生命周期可以是 隐式的： `add(p1: &Point, p2: &Point) -> Point`
- 可以显式：`&'a Point`, `&'document str`
- `&'a Point` 读作： 一个至少在 生命周期 a 中有效的 被借用的 Point。
- 生命周期 通常 由 编译器 推导： 你不能 自己定义 生命周期
  - lifetime annotation 创建了 约束， 编译器会 确认 是否存在一个 有效的 解决方案
- 函数的 参数 和返回值的 生命周期 必须全部指定，但是 rust 允许 在大多数情况下，通过一些简单的规则来消除 生命周期。


### lifetime in function call

除了 借用参数， 函数还可以返回一个 借用的 value。

```rust
#[derive(Debug)]
struct Point(i32, i32);

fn left_most<'a>(p1: &'a Point, p2: &'a Point) -> &'a Point {
    if p1.0 < p2.0 { p1 } else { p2 }
}

fn main() {
    let p1: Point = Point(10, 10);
    let p2: Point = Point(20, 20);
    let p3: &Point = left_most(&p1, &p2);
    println!("p3: {p3:?}");
}
```

- `'a` 是一个泛型参数， 由编译器推断
- 生命周期 从 `'` 开始， `'a` 是一个典型的默认名字
- `&'a Point` 被读为：至少在生命周期 a 中有效的 借用的 Point。
  - "至少" 是重要的，当 参数 有不同的 生命周期时。



```rust
#[derive(Debug)]
struct Point(i32, i32);

fn left_most<'a>(p1: &'a Point, p2: &'a Point) -> &'a Point {
    if p1.0 < p2.0 { p1 } else { p2 }
}

fn main() {
    let p1: Point = Point(10, 10);
    let p3: &Point;
    {
        let p2: Point = Point(20, 20);
        p3 = left_most(&p1, &p2);
    }
    println!("p3: {p3:?}");
}
```
编译失败，因为 p3 比 p2 活的长


重置代码，修改：
`fn left_most<'a, 'b>(p1: &'a Point, p2: &'a Point) -> &'b Point`
编译失败，因为 'a 和 'b 的 关系不清楚。


另一种解释的方法
- 对2个值的 2个ref 被 函数借用，函数返回 另一个 ref
- 返回值 必须来自 2个input之一 (或来自 一个 全局变量)
- 到底是哪个？ 编译器需要确定。来确保 返回的 ref 不会比 输入的ref 的生命周期 更长。



### lifetime in data structure

如果 数据类型 保存了 借用的值， 那么必须 标注 生命周期

```rust
#[derive(Debug)]
struct Highlight<'doc>(&'doc str);

fn erase(text: String) {
    println!("Bye {text}!");
}

fn main() {
    let text = String::from("The quick brown fox jumps over the lazy dog.");
    let fox = Highlight(&text[4..19]);
    let dog = Highlight(&text[35..43]);
    // erase(text);
    println!("{fox:?}");
    println!("{dog:?}");
}
```

- struct HighLight 的标注 强制 要求所有包含 &str的 数据 至少 活的 和 HighLight的 实例一样长。
- 在 fox 或 dog 的 生命周期 结束前 消费掉 text， borrow checker 会抛出error
- 具有 借用数据的 type 强制 用户 保留 原数据。 这对于 创建 轻量级的view 很有用， 但是通常来说，会更难使用
- 如果可能的花，让 数据结构 直接拥有 数据
- 内部有多个ref 的 struct 可能有多个 生存周期注释；如果除了 结构本身的 生存周期外，还需要 描述 ref 之间的 生命周期关系，那么 就需要 多个生命周期了。这些是 高级用例。



## struct

```rust
struct Person {
    name: String,
    age: u8,
}

fn main() {
    let mut peter = Person {
        name: String::from("Peter"),
        age: 27,
    };
    println!("{} is {} years old", peter.name, peter.age);
    
    peter.age = 28;
    println!("{} is {} years old", peter.name, peter.age);
    
    let jackie = Person {
        name: String::from("Jackie"),
        ..peter
    };
    println!("{} is {} years old", jackie.name, jackie.age);
}
```

- struct 和 c/c++ 类似
  - 定义type 不需要 typedef， 这点和 C++类似， 和 C不同。
  - struct之间没有继承关系， 这点和 C++不同。
- 方法被定义在 `impl` 块中。
- 语法 `..peter`， 允许我们从 旧数据中 复制，而不需要 显式写明。 它必须是最后一个元素。

。。C也可以不使用 typedef 的。 C++可以使用 typedef。


### tuple struct

如果 field的名字 不重要，那么可以使用 tuple struct

```rust
struct Point(i32, i32);

fn main() {
    let p = Point(17, 23);
    println!("({}, {})", p.0, p.1);
}
```


#### newtype
经常用于 单个field 封装， 被称为 newtype

```rust
struct PoundsOfForce(f64);
struct Newtons(f64);

fn compute_thruster_force() -> PoundsOfForce {
    todo!("Ask a rocket scientist at NASA")
}

fn set_thruster_force(force: Newtons) {
    // ...
}

fn main() {
    let force = compute_thruster_force();
    set_thruster_force(force);
}
```

- newtype 是对 基本类型中的 值 的附加信息进行编码的好方法，如
  - 这个数字代表的 单位，在上面的例子中 是 牛顿
  - 这个值在创建时 通过了一些 校验， 所以 不再需要 每次使用时 校验。
- 通过访问 newtype 的 唯一的字段 将 f64 加上去。
  - rust 不喜欢 不精确的数字，比如 自动展开，或者 使用 布尔值 作为整数。
  - 运算符重载 在后续介绍。




### field shorthand syntax

如果已经 为变量设置了 名字，那么你可以 使用 shorthand 来 创建 struct

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

impl Person {
    fn new(name: String, age: u8) -> Person {
        Person { name, age }
    }
}

fn main() {
    let peter = Person::new(String::from("Peter"), 27);
    println!("{peter:?}");
}
```

- new 函数可以使用 Self 来替换 2个 Person （。。返回值，方法体）
```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}
impl Person {
    fn new(name: String, age: u8) -> Self {
        Self { name, age }
    }
}
```

- 为struct 实现 Default trait。  定义一些属性，其它的属性 使用 默认值
```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}
impl Default for Person {
    fn default() -> Person {
        Person {
            name: "Bot".to_string(),
            age: 0,
        }
    }
}
fn create_default() {
    let tmp = Person {
        ..Person::default()     // 默认值
    };
    let tmp = Person {
        name: "Sam".to_string(),    // 自定义
        ..Person::default()         // 默认值
    };
}
```

- method 被定义在 impl 块中
- 使用 struct update 语法 使用 peter 定义一个 新的结构体，注意，peter 变量 不能再被访问。
- 使用 `{:#?}` 打印 结构体时 要求 实现 Debug。


## method

通过 impl 块 关联 方法到 你的type。

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

impl Person {
    fn say_hello(&self) {
        println!("Hello, my name is {}", self.name);
    }
}

fn main() {
    let peter = Person {
        name: String::from("Peter"),
        age: 27,
    };
    peter.say_hello();
}
```


- It can be helpful to introduce methods by comparing them to functions.
  - Methods are called on an instance of a type (such as a struct or enum), the first parameter represents the instance as self.
  - Developers may choose to use methods to take advantage of method receiver syntax and to help keep them more organized. By using methods we can keep all the implementation code in one predictable place.

- Point out the use of the keyword self, a method receiver.
  - Show that it is an abbreviated term for self: Self and perhaps show how the struct name could also be used.
  - Explain that Self is a type alias for the type the impl block is in and can be used elsewhere in the block.
  - Note how self is used like other structs and dot notation can be used to refer to individual fields.
  - This might be a good time to demonstrate how the &self differs from self by modifying the code and trying to run say_hello twice.

- We describe the distinction between method receivers next.




### method receiver

&self 意味着 method immut 借用 对象。 有其它类型的 receiver
- &self， 通过 shared，immutable ref 从 caller 那里 获得 对象。 调用完成后，对象还可以继续使用
- &mut self， 通过 unique， mut ref 从 caller 那里获得 对象， 调用完后，还可以继续使用
- self， 从 caller 获得 对象的 所有权。 当method 结束时，对象会被 drop， 除非 所有权 被明确地 传递。 拥有所有权 不意味着 mut。
- mut self， 和上面一样，而且 method 可以修改 对象
- no receiver， struct 的 ==static== method， 通常用于 按约定称为 new 构造器。

除了 self 的变体， 还有一些 特殊的 wrapper 类型， 也可以 作为 receiver， 比如 `Box<Self>`


### example

不同的 receiver

```rust
#[derive(Debug)]
struct Race {
    name: String,
    laps: Vec<i32>,
}

impl Race {
    fn new(name: &str) -> Race {  // No receiver, a static method
        Race { name: String::from(name), laps: Vec::new() }
    }

    fn add_lap(&mut self, lap: i32) {  // Exclusive borrowed read-write access to self
        self.laps.push(lap);
    }

    fn print_laps(&self) {  // Shared and read-only borrowed access to self
        println!("Recorded {} laps for {}:", self.laps.len(), self.name);
        for (idx, lap) in self.laps.iter().enumerate() {
            println!("Lap {idx}: {lap} sec");
        }
    }

    fn finish(self) {  // Exclusive ownership of self
        let total = self.laps.iter().sum::<i32>();
        println!("Race {} is finished, total lap time: {}", self.name, total);
    }
}

fn main() {
    let mut race = Race::new("Monaco Grand Prix");
    race.add_lap(70);
    race.add_lap(68);
    race.print_laps();
    race.add_lap(71);
    race.print_laps();
    race.finish();
    // race.add_lap(42);
}
```



## ==standard library==

- Option, Result
- String
- Vec
- HashMap
- Box, an owned pointer for heap-allocated data.
- Rc, a shared reference-counted pointer for heap-allocated data.

。。unique_ptr, shared_ptr ?


rust的 标准库分为3层： core, alloc, std
- core 包含了最基本的类型 和 function, 它们不依赖于 libc, allocator，甚至不依赖于 操作系统。
- alloc 包含了 需要 全局heap allocator 的类型， 如 Vec, Box, Arc
- 嵌入式 Rust app，通常 只 使用 core， 有时 使用 alloc。


### Option, Result

https://google.github.io/comprehensive-rust/std/option-result.html

这个类型代表可选的数据

```rust
fn main() {
    let numbers = vec![10, 20, 30];
    let first: Option<&i8> = numbers.first();
    println!("first: {first:?}");

    let arr: Result<[i8; 3], Vec<i8>> = numbers.try_into();
    println!("arr: {arr:?}");
}
```

- `Option<&T>` 比起 `&T` 有 0空间开销 (。。就是不需要额外的空间)
- `Result` 是 实现 错误处理的 标准类型
- `try_into` 尝试转换 vector 到 固定size的数组
  - 如果 vector 有正确的size， 那么返回 Result::Ok，并且带有 转换后的数组
  - 否则，返回 Result::Err 并带有 原始的 vector



### String

String 是一个 标准的 在heap上申请空间的 可增长的 UTF-8 字符串buffer

```rust
fn main() {
    let mut s1 = String::new();
    s1.push_str("Hello");
    println!("s1: len = {}, capacity = {}", s1.len(), s1.capacity());

    let mut s2 = String::with_capacity(s1.len() + 1);
    s2.push_str(&s1);
    s2.push('!');
    println!("s2: len = {}, capacity = {}", s2.len(), s2.capacity());

    let s3 = String::from("🇨🇭");
    println!("s3: len = {}, number of chars = {}", s3.len(),
             s3.chars().count());
}
```

String 实现了 `Deref<Target = str>`, 这意味着 可以在 String 上调用 所有 str 的方法。

- `String::chars` 返回 实际字符的 iter。
- 当人们讨论字符串的时候，可能是 &str，也可能是 String
- 当类型实现 `Deref<Target = T>`，编译器可以让你 调用 T 的method。 编写并比较 `let s3 = s1.deref();`, `let s3 = &*s1;`
- String 被实现为 byte的 vector 的封装，vector支持的许多操作 在 String 上也支持，但是有一些额外的保证。
- index 一个 String 的不同方式：
  - 需要character，使用 s3.chars().nth(i).unwrap()
  - 需要substring，使用 `s3[0..4]`



### Vec

Vec 是一个 大小可变的 创建在heap 上的 buffer。

```rust
fn main() {
    let mut v1 = Vec::new();
    v1.push(42);
    println!("v1: len = {}, capacity = {}", v1.len(), v1.capacity());

    let mut v2 = Vec::with_capacity(v1.len() + 1);
    v2.extend(v1.iter());
    v2.push(9999);
    println!("v2: len = {}, capacity = {}", v2.len(), v2.capacity());

    // Canonical macro to initialize a vector with elements.
    let mut v3 = vec![0, 0, 1, 2, 3, 4];

    // Retain only the even elements.
    v3.retain(|x| x % 2 == 0);
    println!("{v3:?}");

    // Remove consecutive duplicates.
    v3.dedup();
    println!("{v3:?}");
}
```

Vec 实现了 `Deref<Target = [T]>`, 意味着你可以 在 Vec上 调用 slice method。


- `Vec<T>` 是泛型，可以不指定 T， 通过Rust type推导， T 在 第一次调用 push 时 确定。
- `vec![...]` 是一个 宏，用来代替 Vec::new()， 它支持 初始化时 增加元素。
- `[]` 操作，可能越界，导致 panic， 可以使用 get()，返回 Option

便利 vector，修改值: `for e in &mut v { *e += 50; }`


### HashMap

HashMap 可以防止 HashDos 攻击

```rust
use std::collections::HashMap;

fn main() {
    let mut page_counts = HashMap::new();
    page_counts.insert("Adventures of Huckleberry Finn".to_string(), 207);
    page_counts.insert("Grimms' Fairy Tales".to_string(), 751);
    page_counts.insert("Pride and Prejudice".to_string(), 303);

    if !page_counts.contains_key("Les Misérables") {
        println!("We know about {} books, but not Les Misérables.",
                 page_counts.len());
    }

    for book in ["Pride and Prejudice", "Alice's Adventure in Wonderland"] {
        match page_counts.get(book) {
            Some(count) => println!("{book}: {count} pages"),
            None => println!("{book} is unknown.")
        }
    }

    // Use the .entry() method to insert a value if nothing is found.
    for book in ["Pride and Prejudice", "Alice's Adventure in Wonderland"] {
        let page_count: &mut i32 = page_counts.entry(book.to_string()).or_insert(0);
        *page_count += 1;
    }

    println!("{page_counts:#?}");
}
```

下面的代码，第一行 查询 某本书 是否在 hashmap中，不存在 就返回 默认值。
第二行，如果书不存在，会插入 数据。
```rust
  let pc1 = page_counts
      .get("Harry Potter and the Sorcerer's Stone")
      .unwrap_or(&336);
  let pc2 = page_counts
      .entry("The Hunger Games".to_string())
      .or_insert(374);
```

没有 hashmap! 宏来 新建并初始化 一些值 到 HashMap
从1.56起， HashMap 实现了 `From<[(K, V); N]>` 来 用 字面量数组 初始化 HashMap
```rust
  let page_counts = HashMap::from([
    ("Harry Potter and the Sorcerer's Stone".to_string(), 336),
    ("The Hunger Games".to_string(), 374),
  ]);
```

HashMap 可以从 任何 包含k-v tuple 的 Iterator 中 构造。


使用 String ，不使用 &str 可以让例子 更简单
尝试移除 to_string()， 看下能否编译成功。 或者 你认为哪里会有问题。

`std::collections::hash_map::Keys`
`keys()`


### Box

https://google.github.io/comprehensive-rust/std/box.html

Box 是 拥有 heap上的数据 所有权的 指针。

```rust
fn main() {
    let five = Box::new(5);
    println!("five: {}", *five);
}
```
。。stack 上的 five， 指向了 heap 的 5

`Box<T>` 实现了 `Deref<Target = T>`， 使得 所有 T的方法 可以直接用到 `Box<T>` 上。


- Box 类似 C++的 std::unique_ptr， 不过 Box 保证 非null
- 在上面的例子中，可以省略 println 中的 *， 因为 Deref的缘故。
- Box 是有用的，当你：
  - 有一个类型， 这个类型在 编译时 无法确定 实际大小，但是 编译器需要知道实际大小
  - 要 转移 大数据 的 所有权。 为了避免 在 stack上复制大量 数据， 使用Box 在 heap上保存数据，并且 转移所有权 只需要转移 指针。


#### recursive data type

递归的数据类型，或 动态size的数据类型， 需要使用 Box

```rust
#[derive(Debug)]
enum List<T> {
    Cons(T, Box<List<T>>),
    Nil,
}

fn main() {
    let list: List<i32> = List::Cons(1, Box::new(List::Cons(2, Box::new(List::Nil))));
    println!("{list:?}");
}
```

![dbecda964539184ca3b2aed919414378.png](resources/894f24c3d1e3489ab5a4a919d14ec6c6.png)

- 如果不使用Box，我们尝试 在 List中 嵌入一个 List，编译器无法计算出 这个 struct的 size。
- Box 解决了这个问题，因为 Box 是固定大小的，指向了 heap中的 下一个List。
- 如果没有 Box， 编译器会报错： `Recursive with indirection`, 这意味着你 需要使用 Box 或 类似的 引用，而不是直接存value。


#### Niche优化

```rust
#[derive(Debug)]
enum List<T> {
    Cons(T, Box<List<T>>),
    Nil,
}

fn main() {
    let list: List<i32> = List::Cons(1, Box::new(List::Cons(2, Box::new(List::Nil))));
    println!("{list:?}");
}
```

Box 不可能为null，所以 指针始终是有效的，非null的。这允许 编译器 优化 内存布局

![91e3a9118589b029350422ee5d8870f4.png](resources/34af7a920a5d4a48b40771561bc4f706.png)


。。把 Cons 优化掉了。但是 不太理解 Cons 有什么作用。
。。Cons 是一个 tuple struct，保存了 值 和下一个 List 的Box，优化后 就没有 Cons了。
。。只是内存中没有， class metadata 还是应该有 Cons 的。不然 没办法 list.0, list.1 之类的。



### Rc

Rc 是一个 reference-counted shared pointer。当你想要从多个地方 指向 相同的数据时， 你使用这个。

```rust
use std::rc::Rc;

fn main() {
    let mut a = Rc::new(10);
    let mut b = Rc::clone(&a);

    println!("a: {a}");
    println!("b: {b}");
}
```

- 如果想在 多线程中使用，查看 `Arc` `Mutex`
- 你可以 将 shared pointer 降级为 Weak pointer，来创建 可以drop 的 循环


- Rc 确保 它包含的值是 有效的。
- 类似 C++的 std::shared_ptr
- Rc::clone is cheap, 它创建一个指针 指向相同的地址，并且 增加 ref count，不会 deep clone，所以当你有 性能问题时，不需要检查 Rc。
- make_mut 在必要(clone-on-write) 时 clone 内部值，并返回一个 mut ref
- 使用 Rc::strong_count 来检查 ref count
- Rc::downgrade 给你 一个 weakly reference-counted object，来创建 可以 drop的 cycle ( 类似和 RefCell 的组合)



### Cell, RefCell

这2个实现了 内部可变性：在不可变的上下文中 修改值

Cell 通常用于 简单类型，它需要 copy或 move 值。
复杂类型使用 RefCell，它在 运行时 追踪 shared 和 exclusive ref，如果误用就 panic。

```rust
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug, Default)]
struct Node {
    value: i64,
    children: Vec<Rc<RefCell<Node>>>,
}

impl Node {
    fn new(value: i64) -> Rc<RefCell<Node>> {
        Rc::new(RefCell::new(Node { value, ..Node::default() }))
    }

    fn sum(&self) -> i64 {
        self.value + self.children.iter().map(|c| c.borrow().sum()).sum::<i64>()
    }
}

fn main() {
    let root = Node::new(1);
    root.borrow_mut().children.push(Node::new(5));
    let subtree = Node::new(10);
    subtree.borrow_mut().children.push(Node::new(11));
    subtree.borrow_mut().children.push(Node::new(12));
    root.borrow_mut().children.push(subtree);

    println!("graph: {root:#?}");
    println!("graph sum: {}", root.borrow().sum());
}
```

- 如果在上面的例子中，我们使用 Cell，而不是 RefCell，我们不得不 将 Node从 Rc中 move出来，然后 push，然后 move回去。 这是 安全的，因为 始终只有 一个，un-ref的 value，但是 代价较大。
- 要对Node 进行任意操作，你必须调用 一个 RefCell 方法，通常是 borrow 或 borrow_mut。
- 演示一个 运行时panic， 增加一个 `fn inc(&mut self)` 来增加 self.value，并且在 children 上 调用 inc， 当出现 ref loop (。就是root变成 child的 child) 的时候 会 panic，`thread 'main' panicked at 'already borrowed: BorrowMutError'`



## ==module==

我们已经看到了 impl 可以让我们 namepsace function to a type。
类似的， mod 可以让我们 namespace type 和 function

```rust
mod foo {
    pub fn do_something() {
        println!("In the foo module");
    }
}

mod bar {
    pub fn do_something() {
        println!("In the bar module");
    }
}

fn main() {
    foo::do_something();
    bar::do_something();
}
```

- package 提供了 功能 和 包含了 Cargo.file，这个文件描述了 如何构建一个 多个crate的 bundle
- crate 是 module 组成的树， binary crate 生成 可执行文件，library crate 编译为 库
- module 定义了 organization， scope


### 可见性

模块提供了 隐私边界
- 默认下，模块中item 是 private
- parent 和 sibling (兄弟姐妹) 中的item 总是可见的
- 即，如果 在模块foo 中，item A 可见， 那么 item A 对于 foo的所有 子模块 都可见。

。。本章最后， 祖先模块， 子模块， 都必须可见。 有点。
。。主要是  可见， 是我被可见，还是 它被可见。
。。 应该是 我可见。 即 对方被看见。
。。 所以是 我能看到 父模块 和 兄弟姐妹 模块 里面的内容。

```rust
mod outer {
    fn private() {
        println!("outer::private");
    }

    pub fn public() {           // pub
        println!("outer::public");
    }

    mod inner {
        fn private() {
            println!("outer::inner::private");
        }

        pub fn public() {
            println!("outer::inner::public");
            super::private();
        }
    }
}

fn main() {
    outer::public();
}
```

- `pub` 来使得 模块 公开 。。 公开mod 还是 公开 fn ？ 上面的例子 只有 fn。 不知道mod可以不可以

可以使用 `pub(...)` 来指定 public 可见性的 scope
- 查看 https://doc.rust-lang.org/reference/visibility-and-privacy.html#pubin-path-pubcrate-pubsuper-and-pubself
- 配置 pub(crate) 是一个 常用模式
- 比较少见的是，你可以 给 某个path 可见性
- 任何情况下，visibility must be granted to an ancestor module (and all of its descendants).

。。its descendants 应该是指 兄弟姐妹模块
。。ancestor module 应该是指 parent模块， 就是 只高一级， 而不是 追溯上 最顶上的 root。
。。当然，parent可见， 那么 parent的parent 自然也可见了。



### path

path被如下解析
- 作为 相对路径
  - foo 或 self::foo ： 指向 当前模块 中的 foo
  - super::foo : 指向 父模块 中的 foo

- 作为 绝对路径
  - crate::foo ： 指向 当前 crate 的root 的 foo
  - bar::foo ： 指向 `bar` crate的 foo

通过 `use` 可以 将 其他模块的 符号(symbol) 导入 进来。
```rust
use std::collections::HashSet;
use std::mem::transmute;
```


### filesystem hierarchy

不带 内容的 模块 可以告诉 rust 从 另一个文件中 查找它
`mod garden;`
这告诉 rust，gardon 模块可以在 src/garden.rs 中找到。
类似， garden::vegetables 模块，可以在 src/garden/vegetables.rs 中找到。

crate的 root在：
- src/lib.rs     对于library crate
- src/main.rs    对于binary crate


定义在文件中的 module 也可以 文档化，通过 "inner doc comments"。
```rust
//! This module implements the garden, including a highly performant germination
//! implementation.

// Re-export types from this module.
pub use seeds::SeedPacket;
pub use garden::Garden;

/// Sow the given seed packets.
pub fn sow(seeds: Vec<SeedPacket>) { todo!() }

/// Harvest the produce in the garden that is ready.
pub fn harvest(garden: &mut Garden) { todo!() }
```


- rust2018之前，module需要位于 module_name/mod.rs， 而不是 module_name.rs。现在依然可以这样。
- 修改成 module_name.rs 是因为 mod.rs 文件太多，且 IDE 无法 直接区分它们。
- 更深的嵌套 可以使用 文件夹，即使 主module 是一个文件
```rust
src/
├── main.rs
├── top_module.rs
└── top_module/
    └── sub_module.rs
```
- 通过编译器指示，可以指定 module 的位置
```rust
#[path = "some/path.rs"]
mod some_module;
```
这是有用的，比如，你想将 测试模块放在 some_module.test.rs 中。(类似 Go 中的约定)



## ==generic==

rust支持 泛型， 可以让你 抽象 type用到的 算法 或 数据结构。


### generic data type
你可以使用 泛型 来 抽象 具体的field type

```rust
#[derive(Debug)]
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
    println!("{integer:?} and {float:?}");
}
```


### generic method

可以在 impl 块 声明 一个 泛型 类型。

```rust
#[derive(Debug)]
struct Point<T>(T, T);

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.0  // + 10
    }

    // fn set_x(&mut self, x: T)
}

fn main() {
    let p = Point(5, 10);
    println!("p.x = {}", p.x());
}
```

Q：为什么 `impl<T> Point<T> {  }` 中需要 2个 T？
- 这是因为 它是 泛型类型的 泛型实现部分。它们是独立的。
- 它意味着 这些方法 为 任意T 定义。
- 可以写出： `impl Point<u32> {  }`
  - Point 依然是 泛型，你可以使用 `Point<f64>` , 但是 这个 块中的 方法 只能被 `Point<u32>` 使用



### Monomorphization (单态化)

在调用点处，泛型代码 转换为 非泛型代码
```rust
fn main() {
    let integer = Some(5);
    let float = Some(5.0);
}
```
行为就像你编写了下面的代码
```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

这是一个 0代价抽象

## trait

使用 trait 抽象 类型， trait 类似于 接口

```rust
struct Dog { name: String, age: i8 }
struct Cat { lives: i8 } // No name needed, cats won't respond anyway.

trait Pet {
    fn talk(&self) -> String;
}

impl Pet for Dog {
    fn talk(&self) -> String { format!("Woof, my name is {}!", self.name) }
}

impl Pet for Cat {
    fn talk(&self) -> String { String::from("Miau!") }
}

fn greet<P: Pet>(pet: &P) {
    println!("Oh you're a cutie! What's your name? {}", pet.talk());
}

fn main() {
    let captain_floof = Cat { lives: 9 };
    let fido = Dog { name: String::from("Fido"), age: 5 };

    greet(&captain_floof);
    greet(&fido);
}
```

### trait objects

trait对象 允许 不同type 的值

```rust
struct Dog { name: String, age: i8 }
struct Cat { lives: i8 } // No name needed, cats won't respond anyway.

trait Pet {
    fn talk(&self) -> String;
}

impl Pet for Dog {
    fn talk(&self) -> String { format!("Woof, my name is {}!", self.name) }
}

impl Pet for Cat {
    fn talk(&self) -> String { String::from("Miau!") }
}

fn main() {
    let pets: Vec<Box<dyn Pet>> = vec![
        Box::new(Cat { lives: 9 }),
        Box::new(Dog { name: String::from("Fido"), age: 5 }),
    ];
    for pet in pets {
        println!("Hello, who are you? {}", pet.talk());
    }
}
```

pets的内存布局：

![3cd42548a2b9878ad5ad1159cf3743f6.png](resources/a44bc14c97b2485ab1b0e26eee6b31bf.png)

。。name 的后面的 4,4 是什么？


- 实现 某个trait 的 不同 type 可以有着不同的 size，这使得 没有办法 `Vec<dyn Pet>`。 。。 vec的元素的大小 必须相同，且在 编译时确定。 所以没有办法这样写。 只能加一个 Box
- pets 放在 stack， vector data 存储在 heap。 vector的元素 是 胖指针(fat pointer)
  - fat pointer 是 双宽度指针，由2 部分组成：
    - 指向 真实类型的 指针
    - 指向 Pet 的impl 的 vtable (virtual method table) 
  - Dog 有 name 和 age 属性， Cat有lives 属性。
- 比较下面的代码的输出
```rust
    println!("{} {}", std::mem::size_of::<Dog>(), std::mem::size_of::<Cat>());
    println!("{} {}", std::mem::size_of::<&Dog>(), std::mem::size_of::<&Cat>());
    println!("{}", std::mem::size_of::<&dyn Pet>());
    println!("{}", std::mem::size_of::<Box<dyn Pet>>());
```
### ==上面的code==


### deriving triat

derive 宏，可以为 数据结构 impl 指定的 trait， 自动生成代码。

```rust
#[derive(Debug, Clone, PartialEq, Eq, Default)]
struct Player {
    name: String,
    strength: u8,
    hit_points: u8,
}

fn main() {
    let p1 = Player::default();
    let p2 = p1.clone();
    println!("Is {:?}\nequal to {:?}?\nThe answer is {}!", &p1, &p2,
             if p1 == p2 { "yes" } else { "no" });
}
```

。。这宏， 内部很复杂的吧。


### default method

trait 可以包含 方法impl

```rust
trait Equals {
    fn equals(&self, other: &Self) -> bool;
    fn not_equals(&self, other: &Self) -> bool {
        !self.equals(other)
    }
}

#[derive(Debug)]
struct Centimeter(i16);

impl Equals for Centimeter {
    fn equals(&self, other: &Centimeter) -> bool {
        self.0 == other.0
    }
}

fn main() {
    let a = Centimeter(10);
    let b = Centimeter(20);
    println!("{a:?} equals {b:?}: {}", a.equals(&b));
    println!("{a:?} not_equals {b:?}: {}", a.not_equals(&b));
}
```

- trait 可以impl 方法，作为方法的默认实现。 方法的默认实现 中可以调用 其他 没有默认实现的 方法。
- 移动 not_equals 方法到 新trait `NotEquals`
- 使得 Equals 成为 NotEquals 的 super trait
```rust
trait NotEquals: Equals {
    fn not_equals(&self, other: &Self) -> bool {
        !self.equals(other)
    }
}
```
- 为 Equals对象 提供 NotEquals 的一揽子实现
```rust
trait NotEquals {
    fn not_equals(&self, other: &Self) -> bool;
}

impl<T> NotEquals for T where T: Equals {
    fn not_equals(&self, other: &Self) -> bool {
        !self.equals(other)
    }
}
```
通过一揽子实现， 不再要求 Equals 作为 NotEquals 的 super trait。

### ==上面的code==


### trait bound

使用 泛型时，你经常希望 type 实现了 某些 trait， 这样，你可以调用 这个 trait的 方法。

你可以通过 `T: Trait` or `impl Trant` 来实现

```rust
fn duplicate<T: Clone>(a: T) -> (T, T) {    // 。。可以 fn duplicate(a: impl Clone) - 。。好像不行，返回值没有办法写。 不可能 (impl Clone, impl Clone) 吧。 看下一章，还真可以。。
    (a.clone(), a.clone())
}

// Syntactic sugar for:
//   fn add_42_millions<T: Into<i32>>(x: T) -> i32 {
fn add_42_millions(x: impl Into<i32>) -> i32 {
    x.into() + 42_000_000
}

// struct NotClonable;

fn main() {
    let foo = String::from("foo");
    let pair = duplicate(foo);
    println!("{pair:?}");

    let many = add_42_millions(42_i8);
    println!("{many}");
    let many_more = add_42_millions(10_000_000);
    println!("{many_more}");
}
```
### ==上面的code==
。。还有下面

- where 子句
```rust
fn duplicate<T>(a: T) -> (T, T)
where
    T: Clone,
{
    (a.clone(), a.clone())
}
```

- where 子句整理了 函数签名。
- 它有额外的 功能，使得它更加强大
  - `:` 的左侧可以是任意的type， 比如 `Option<T>`



### impl trait

和 trait bound 类似， impl Trait 语法可以用于 输入参数 和 返回值

```rust
use std::fmt::Display;

fn get_x(name: impl Display) -> impl Display {
    format!("Hello {name}")
}

fn main() {
    let x = get_x("foo");
    println!("{x}");
}
```

- impl Trait 可以用在 你在无法为类型命名的时候。


- impl Trait 在不同位置 的 含义有一点点 区别。
  - 对于 参数， impl Trait， 就像 带有trait bound 的 匿名泛型参数。
  - 对于 返回值，它意味着，返回值是一些 实现了这个trait的 具体的type，但是没有具体name。如果 你不希望 在 public api中 暴露具体类型，那么这很有用。


上面的例子中 使用了 impl Display 2次， 但==不代表 2次都是 相同的 type==。
如果我们使用 `T: Display`， 那么 强制要求 ==输入的type 和 return type 必须一样==


## ==重要的trait==

- Iterator, IntoIterator， 用于 for循环
- From, Into ， 用来转换值
- Read, Write， 用于 IO
- Add, Mul,... ，  用来 operator overload
- Drop， 定义 析构器
- Default， 构造 类型的 默认实例

Fn, FnMut, FnOnce


### Iterator

在自己的类型上 实现 Iterator trait

```rust
struct Fibonacci {
    curr: u32,
    next: u32,
}

impl Iterator for Fibonacci {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        let new_next = self.curr + self.next;
        self.curr = self.next;
        self.next = new_next;
        Some(self.curr)
    }
}

fn main() {
    let fib = Fibonacci { curr: 0, next: 1 };
    for (i, n) in fib.enumerate().take(5) {
        println!("fib({i}): {n}");
    }
}
```

- Iterator 实现了 集合上的 公共的 编程操作。
- IntoIterator 使得 loop工作。 所有的 集合类型(`Vec<T>`) 以及集合类型的ref (`&Vec<T>, &[T]`) 都实现了它。 Range 也实现了它，这就是为什么你可以 `for i in some_vec{..}` ，但是 不能 `some_vec.next()`



### FromIterator

FromIterator 可以让你 从 Iterator 构造 集合

```rust
fn main() {
    let primes = vec![2, 3, 5, 7];
    let prime_squares = primes
        .into_iter()
        .map(|prime| prime * prime)
        .collect::<Vec<_>>();
    println!("prime_squares: {prime_squares:?}");
}
```

Iterator 实现了 `fn collect<B>(self) -> B where B: FromIterator<Self::Item>, Self: Sized`

也有实现可以让你 转换 `Iterator<Item = Result<V, E>>` 到 `Result<Vec<V>, E>`


### From and Into

实现 From, Into 的 type 进行 类型转换 更方便
```rust
fn main() {
    let s = String::from("hello");
    let addr = std::net::Ipv4Addr::from([127, 0, 0, 1]);
    let one = i16::from(true);
    let bigger = i32::from(123i16);
    println!("{s}, {addr}, {one}, {bigger}");
}
```

当 From 实现的时候， Into 自动实现
```rust
fn main() {
    let s: String = "hello".into();
    let addr: std::net::Ipv4Addr = [127, 0, 0, 1].into();
    let one: i16 = true.into();
    let bigger: i32 = 123i16.into();
    println!("{s}, {addr}, {one}, {bigger}");
}
```

当 声明 一个 函数输入参数类型 为 "任何能转为String 的类型"，应该使用 `Into`， 这样，你的方法会接收 所有实现了 From (。有From，自动就有 Into) 的类型， 以及 所有 实现了 Into (。但没有实现 From) 的类型。


### Read，Write

https://google.github.io/comprehensive-rust/traits/read-write.html

使用 Read, BufRead， 你可以抽象 u8 source

```rust
use std::io::{BufRead, BufReader, Read, Result};

fn count_lines<R: Read>(reader: R) -> usize {
    let buf_reader = BufReader::new(reader);
    buf_reader.lines().count()
}

fn main() -> Result<()> {
    let slice: &[u8] = b"foo\nbar\nbaz\n";
    println!("lines in slice: {}", count_lines(slice));

    let file = std::fs::File::open(std::env::current_exe()?)?;
    println!("lines in file: {}", count_lines(file));
    Ok(())
}
```

类似，Write 可以抽象 u8 sink

```rust
use std::io::{Result, Write};

fn log<W: Write>(writer: &mut W, msg: &str) -> Result<()> {
    writer.write_all(msg.as_bytes())?;
    writer.write_all("\n".as_bytes())
}

fn main() -> Result<()> {
    let mut buffer = Vec::new();
    log(&mut buffer, "Hello")?;
    log(&mut buffer, "World")?;
    println!("Logged: {:?}", buffer);
    Ok(())
}
```

### Drop

https://google.github.io/comprehensive-rust/traits/drop.html

实现 Drop，可以在 出scope时 自动 实现的执行代码。

```rust
struct Droppable {
    name: &'static str,
}

impl Drop for Droppable {
    fn drop(&mut self) {
        println!("Dropping {}", self.name);
    }
}

fn main() {
    let a = Droppable { name: "a" };
    {
        let b = Droppable { name: "b" };
        {
            let c = Droppable { name: "c" };
            let d = Droppable { name: "d" };
            println!("Exiting block B");
        }
        println!("Exiting block A");
    }
    drop(a);            // .. 手动drop
    println!("Exiting main");
}
```

- `std::mem::drop` 不同于 `std::ops::Drop::drop`
- 出scope时自动 drop
- 当值drop时，如果实现了 `std::ops::Drop`，那么它的 `Drop::drop` 方法被调用。
- 它的所以field 也会被 drop，无论是否实现了 `Drop`
- `std::mem::drop` 是一个 接收任何值 的 空方法。这个方法获得 值的 所有权，所以在 方法退出时 触发drop，所以 它是一个方便的方法 来 显式 提前drop 值。
  - 在 drop时 有时需要作一些事情：释放锁，关闭文件 等


为什么 `Drop::drop` 不是获得 `self`？
如果这样做的话，在scope结束时，调用`std::mem::drop`，会导致再次调用`Drop::drop`，会 stack overflow
。。？既然 scope结束时 还会 调用 std::mem::drop，那么 我实现Drop，提前drop 有什么意义？提前释放部分资源？ 而且，drop的时候 还要判断 是否已经被释放了 (是否null) 。 不过无论如何，都应该判断。
。。可以观察下，代码，手动drop后，出scope是否还会 drop。 应该会的，不然这条没有意义。

尝试使用`a.drop()` 代替 `drop(a)`


### Default

提供type的默认value

```rust
#[derive(Debug, Default)]
struct Derived {
    x: u32,
    y: String,
    z: Implemented,
}

// ----------
#[derive(Debug)]
struct Implemented(String);
impl Default for Implemented {
    fn default() -> Self {
        Self("John Smith".into())
    }
}
// ----------

fn main() {
    let default_struct = Derived::default();
    println!("{default_struct:#?}");

    let almost_default_struct = Derived {
        y: "Y is set!".into(),
        ..Derived::default()
    };
    println!("{almost_default_struct:#?}");

    let nothing: Option<Derived> = None;
    println!("{:#?}", nothing.unwrap_or_default());
}
```

- 可以直接impl，或通过 `#[derive(Default)]` 实现
- derive 的实现 会为 所有 field 设置 它们的 默认值
  - 这意味着，struct中的所有 field 的 type 都必须实现了 `Default`
- 标准rust类型 通常实现了 Default
- 部分struct copy 可以很好地利用 default 值。 (。。 估计是指 `..Derived::default()`)
- rust标准库 知道 type可以实现 `Default`， 所以提供了 一些方便的方法来使用它。
- `..` 被称为 struct update syntax


### Add, Mul, ...

运算符重载，通过 `std::ops` 中的 trait 来实现

```rust
#[derive(Debug, Copy, Clone)]
struct Point { x: i32, y: i32 }

impl std::ops::Add for Point {      // ------
    type Output = Self;

    fn add(self, other: Self) -> Self {
        Self {x: self.x + other.x, y: self.y + other.y}
    }
}

fn main() {
    let p1 = Point { x: 10, y: 20 };
    let p2 = Point { x: 100, y: 200 };
    println!("{:?} + {:?} = {:?}", p1, p2, p1 + p2);
}
```


- 你可以为`&Point`实现`add`。 什么情况下，这 (。。为ref实现add) 是有用的？
  - Add:add 消费了 self，如果类型 T 没有实现 Copy，你需要考虑为 &T 也实现 运算符重载，这可以避免 在调用点的 不必要的 clone。
。。我记得默认是 传递所有权， 所以 调用的时候 需要 手动 clone 一个副本。 所以 
。。不， Copy 没有任何方法，只是告诉 编译器： 复制 stack的数据，并且 `=` 从默认的 传递所有权，变成了 传递副本。 。 Clone 需要程序员自己手动clone，并且方法自己实现，所以可以实现 deep clone。
。。所以一旦 实现了 Copy，那么 fn add(self, a: Self) -> Self , 2个形参 都是 副本？

### ==上面的code==

- 为什么 Output (。。impl Add for Point 的第一行 ) 是一个 关联type？ 可以变成 函数的类型参数？
  - 函数类型参数 是被 调用者 控制的，但是 关联type 是 实现trait的人 控制的。

- 可以为 2个不同类型 实现 `Add`，比如 `impl Add<(i32, i32)> for Point`
。。那么 Add 后面不写， 就是 默认 Point ？



### ==Fn, FnMut, FnOnce==

闭包， lambda表达式 的type 无法命名，但是 它们可以实现 特殊的 Fn, FnMut, FnOnce。

```rust

// 外面的lambda 并没有实现 FnOnce 啊， 这个是 额外附加的？ 就像 C++的 外面是 int， 形参是 const int ？ 。 限制了 这个lambda 在内部 只能使用一次？
fn apply_with_log(func: impl FnOnce(i32) -> i32, input: i32) -> i32 {
    println!("Calling function on {input}");
    func(input)
}

fn main() {
    let add_3 = |x| x + 3;
    println!("add_3: {}", apply_with_log(add_3, 10));
    println!("add_3: {}", apply_with_log(add_3, 20));

    let mut v = Vec::new();
    let mut accumulate = |x: i32| {
        v.push(x);
        v.iter().sum::<i32>()
    };
    println!("accumulate: {}", apply_with_log(&mut accumulate, 4));
    println!("accumulate: {}", apply_with_log(&mut accumulate, 5));

    let multiply_sum = |x| x * v.into_iter().sum::<i32>();
    println!("multiply_sum: {}", apply_with_log(multiply_sum, 3));
}
```

- Fn 不会 消费 也不会 修改 捕获的值。 Fn 可以不捕获任何东西。 可以被 并发地 多次调用。
- FnMut 可能修改 捕获的值。 可以多次调用，但不能 并发
- FnOnce 只调用一次， 可以消费捕获的值

FnMut 是 FnOnce的子类型，Fn 是 FnMut 和 FnOnce 的子类型。
例如，可以在 FnOnce被调用的地方 使用 FnMut，  在 FnMut 或 FnOnce 被调用的地方，调用 Fn。
。。这个和 面向对象的多态 正好相反啊。
oop是 实参类型 必然是 形参类型 ( 就像 方法接收Person，内部输出 Person.name, 实参可以是 Student )，
。这里是 形参类型 必然是 实参类型， FnOnce 必然是 Fn， 形参 必然是 实参。  感觉更像是 const int 和 int 的区别。  并不是 subtype。


编译器 根据闭包捕获的值，推断 Copy (比如 add_3) 和 Clone (比如 multiply_sum)。

默认下， 闭包 会 尽量捕获 ref。  `move` 使得 闭包 捕获 value
```rust
fn make_greeter(prefix: String) -> impl Fn(&str) {
    return       move       |name| println!("{} {}", prefix, name)
}

fn main() {
    let hi = make_greeter("Hi".to_string());
    hi("there");
}
```

### ===code: simple gui===

https://google.github.io/comprehensive-rust/exercises/day-3/simple-gui.html



## error handling

- Functions that can have errors list this in their return type,
- There are no exceptions.

### panic

运行时如果发生错误，rust会触发 panic
```rust
fn main() {
    let v = vec![10, 20, 30];
    println!("v[100]: {}", v[100]);
}
```

- panic 是 未覆盖的，未期望的 error， 代表了 bug
- 可以使用 无panic的 api， 比如 Vec::get 

。。无panic 就意味着，返回  Option


#### catch stack unwind
默认下，panic 会导致 stack unwind(展开)， unwind可以被捕获

```rust
use std::panic;

fn main() {
    let result = panic::catch_unwind(|| {
        "No problem here!"
    });
    println!("{result:?}");

    let result = panic::catch_unwind(|| {
        panic!("oh no!");
    });
    println!("{result:?}");
}
```

- 这在 服务器中 有用，即使单个 request 出问题， 整个服务器还是 继续 允许。
- 如果在 Cargo.toml 中设置 panic == 'abort'， 那么这个(。。应该是指 catch_unwind)不会起效。


### structured error handling (Result)

编码时，已经考虑到错误，那么使用 Result。
```rust
use std::fs;
use std::io::Read;

fn main() {
    let file = fs::File::open("diary.txt");
    match file {
        Ok(mut file) => {
            let mut contents = String::new();
            file.read_to_string(&mut contents);
            println!("Dear diary: {contents}");
        },
        Err(err) => {
            println!("The diary could not be opened: {err}");
        }
    }
}
```

- 就像 Option， 成功的值 被放在 Result 中，强制 开发者 抽取值。这也支持 error check。如果 不会发生 error，那么直接 unwrap() 或 expect() 。
- Result 的doc，值得阅读，包含了 方便的方法 和函数 来支持 函数式编程。


### 使用 ? 来扩散error

try operator `?` ，用来返回 error 到 caller 。简化代码

```rust
match some_expression {
    Ok(value) => value,         // 抽取了。 下面返回 Err，所以 整个方法是返回 Option的？
    Err(err) => return Err(err),
}
```
等价于
`some_expression?`


一个返回 `Result<T, Err>` 的方法中，可以对返回 `Result<AnyT, Err>` 的调用 应用 `?`， 不能对 返回 `Option<AnyT>` 或 `Result<T, OtherErr>` 的 调用 应用 `?`， 除非 OtherErr 实现了 `From<Err>`

一个返回 `Option<T>` 的方法中， 只能对 返回`Option<AnyT>` 的调用 应用 `?`

。。？ 但是 无论 Result 还是 Option， T 都是确定的吧， 不可能一个分支返回 i32, 一个分支返回 u32 啊。  所以 为什么 是 AnyT ？

你可以使用不同的 Option 和 Result 方法 将 不兼容的 类型 转换为 其他类型， 如 Option::ok_or, Result::ok, Result::err



#### convert error type

`?` 比之前的要复杂一些

`expression?`
就像
```rust
match expression {
    Ok(value) => value,
    Err(err)  => return Err(From::from(err)),
}
```

`From::from` 意味着 我们尝试 转换 错误的类型。

。。这个 from 好像没有指定 转换成什么啊。   默认转成 返回值的 error类型？
。。from 就不存在，是一个 虚指。。看下面


##### example

```rust
use std::error::Error;
use std::fmt::{self, Display, Formatter};
use std::fs::{self, File};
use std::io::{self, Read};

#[derive(Debug)]
enum ReadUsernameError {
    IoError(io::Error),
    EmptyUsername(String),
}

impl Error for ReadUsernameError {}

impl Display for ReadUsernameError {
    fn fmt(&self, f: &mut Formatter) -> fmt::Result {
        match self {
            Self::IoError(e) => write!(f, "IO error: {e}"),
            Self::EmptyUsername(filename) => write!(f, "Found no username in {filename}"),
        }
    }
}

impl From<io::Error> for ReadUsernameError {
    fn from(err: io::Error) -> ReadUsernameError {
        ReadUsernameError::IoError(err)
    }
}

fn read_username(path: &str) -> Result<String, ReadUsernameError> {
    let mut username = String::with_capacity(100);
    File::open(path)?.read_to_string(&mut username)?;
    if username.is_empty() {
        return Err(ReadUsernameError::EmptyUsername(String::from(path)));
    }
    Ok(username)
}

fn main() {
    //fs::write("config.dat", "").unwrap();
    let username = read_username("config.dat");
    println!("username or error: {username:?}");
}
```

#### derive Error

第三方依赖： thiserror (https://docs.rs/thiserror/latest/thiserror/)
是一个方便的方法 来创建 类似 上一章 的 error

```rust
use std::{fs, io};
use std::io::Read;
use thiserror::Error;

#[derive(Debug, Error)]
enum ReadUsernameError {
    #[error("Could not read: {0}")]
    IoError(#[from] io::Error),
    #[error("Found no username in {0}")]
    EmptyUsername(String),
}

fn read_username(path: &str) -> Result<String, ReadUsernameError> {
    let mut username = String::new();
    fs::File::open(path)?.read_to_string(&mut username)?;
    if username.is_empty() {
        return Err(ReadUsernameError::EmptyUsername(String::from(path)));
    }
    Ok(username)
}

fn main() {
    //fs::write("config.dat", "").unwrap();
    match read_username("config.dat") {
        Ok(username) => println!("Username: {username}"),
        Err(err)     => println!("Error: {err}"),
    }
}
```

thiserror的 
- derive 宏， 自动实现 std::error::Error.
- error 宏 实现 Display
- from 宏 实现 From

对 struct 也有效。

。。？ derive 不是 rust的宏吗？ 它怎么 覆盖掉 rust的 宏的？  是同名 就 默认覆盖？

thiserror 不会影响你的 公共API， 所以 适合作为库


#### dynamic error type
有时， 我们允许 任意的error type 返回， 且 不想写 自己的 enum 来 覆盖 所有的 分支。
`std::error::Error` 使得这变简单。

```rust
use std::fs;
use std::io::Read;
use thiserror::Error;
use std::error::Error;

#[derive(Clone, Debug, Eq, Error, PartialEq)]
#[error("Found no username in {0}")]
struct EmptyUsernameError(String);

fn read_username(path: &str) -> Result<String, Box<dyn Error>> {
    let mut username = String::new();
    fs::File::open(path)?.read_to_string(&mut username)?;
    if username.is_empty() {
        return Err(EmptyUsernameError(String::from(path)).into());
    }
    Ok(username)
}

fn main() {
    //fs::write("config.dat", "").unwrap();
    match read_username("config.dat") {
        Ok(username) => println!("Username: {username}"),
        Err(err)     => println!("Error: {err}"),
    }
}
```

这节约了代码，但是放弃了 清晰的error处理分支。
通常来说，在 库的 公共API中 使用 `Box<dyn Error>` 不是一个 好主意。
在代码中，你只是想要 display error message 的话， 这 是一个不错的 选项。


#### add context to error

anyhow (https://docs.rs/anyhow/)
可以帮助你 增加 上下文信息 到 error 中， 这样 可以减少 自定义的 error type

```rust
use std::{fs, io};
use std::io::Read;
use anyhow::{Context, Result, bail};

fn read_username(path: &str) -> Result<String> {
    let mut username = String::with_capacity(100);
    fs::File::open(path)
        .with_context(|| format!("Failed to open {path}"))?
        .read_to_string(&mut username)
        .context("Failed to read")?;
    if username.is_empty() {
        bail!("Found no username in {path}");
    }
    Ok(username)
}

fn main() {
    //fs::write("config.dat", "").unwrap();
    match read_username("config.dat") {
        Ok(username) => println!("Username: {username}"),
        Err(err)     => println!("Error: {err:?}"),
    }
}
```

- `anyhow::Result<V>` 是 `Result<V, anyhow::Error>` 的 类型别名
- `anyhow::Error` 基本上是 `Box<dyn Error>` 的封装。 所以 对于 public api，这不是一个 好的选择。 但在 app 中应用很多。
- 在必要的时候，可以提取 内部的 真实错误类型 来检测
- `anyhow::Result<T>` 提供的功能 对于 Go开发者 可能很熟悉，因为 和 Go中 (T, error) 类似。



## test

rust 和 cargo 有一个简单的 单元测试框架

- 通过你的代码 支持 单元测试
- 通过 tests/ 目录 支持 集成测试


### unit test (`#[test]`)

`#[test]`

```rust
fn first_word(text: &str) -> &str {
    match text.find(' ') {
        Some(idx) => &text[..idx],
        None => &text,
    }
}

#[test]
fn test_empty() {
    assert_eq!(first_word(""), "");
}

#[test]
fn test_single_word() {
    assert_eq!(first_word("Hello"), "Hello");
}

#[test]
fn test_multiple_words() {
    assert_eq!(first_word("Hello World"), "Hello");
}
```

`cargo test` 来找到 和运行单元测试。


### test module (`#[cfg(test)]`)

单元测试 经常 放到一个 嵌入的 模块中，

```rust
fn helper(a: &str, b: &str) -> String {
    format!("{a} {b}")
}

pub fn main() {
    println!("{}", helper("Hello", "World"));
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_helper() {
        assert_eq!(helper("foo", "bar"), "foo bar");
    }
}
```

- 这样可以测试 private 方法
- `#[cfg(test)]` 表示，只有 `cargo test` 时 才激活代码


### documentation test

rust 原生支持 documentation test

```rust
/// Shortens a string to the given length.
///
/// ```
/// # use playground::shorten_string;
/// assert_eq!(shorten_string("Hello World", 5), "Hello");
/// assert_eq!(shorten_string("Hello World", 20), "Hello World");
/// ```
pub fn shorten_string(s: &str, length: usize) -> &str {
    &s[..std::cmp::min(length, s.len())]
}
```

- `///` 中的 代码 会被 rust 看到
- cargo test 时，这些代码 会被 编译 和执行
- `#` 可以隐藏代码

。。应该是 /// 中的 3个` 包围的 才会被当错代码吧。
。。 就是上面的 第一行 Shorten xxx ，不可能编译啊。



### integration test

如果要将 library 当作 client 进行测试， 使用 集成测试

在 `tests/` 下 创建 .rs 文件
```rust
use my_library::init;

#[test]
fn test_init() {
    assert!(init().is_ok());
}
```

这里的测试，只能访问 你的 crate的 public api


#### useful crate for write test

- googletest, https://docs.rs/googletest
- proptest, https://docs.rs/proptest
- retest, https://docs.rs/rstest


## unsafe rust

rust 分为 2部分
- safe rust， 内存安全，没有 未定义信为
- unsafe rust， 可能触发 未定义行为。

unsafe code 通常是 small 和 隔离的， 它的正确性 需要 文档记录。 它通常 被 包裹在 safe的抽象层 中。

unsafe rust 可以有一些 新能力
- de-ref 原生指针
- 访问 和 修改 可变的 static 变量
- 访问 union 属性
- 调用 unsafe 的函数，包括 `extern` 函数
- 实现 unsafe 的特征

更详细的看： https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html



### de-ref raw pointer

创建指针 是安全的， 但是 解指针 需要 unsafe

```rust
fn main() {
    let mut num = 5;

    let r1 = &mut num as *mut i32;
    let r2 = r1 as *const i32;

    // Safe because r1 and r2 were obtained from references and so are
    // guaranteed to be non-null and properly aligned, the objects underlying
    // the references from which they were obtained are live throughout the
    // whole unsafe block, and they are not accessed either through the
    // references or concurrently through any other pointers.
    unsafe {
        println!("r1 is: {}", *r1);
        *r1 = 10;
        println!("r2 is: {}", *r2);
    }
}
```

de-ref pointer，说明 pointer 必须是 有效的：
- 非null
- 指针指向的是 一个对象 ( 。而不是对象中间)
- 对象 没有被 析构
- 不能并发
- 如果 指针 是通过 ref 转换过来的，那么 对象必须live，且不能 用其他 ref 访问内存。


### mutable static variable

读取 immut static var 是安全的

```rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("HELLO_WORLD: {HELLO_WORLD}");
}
```

但是，由于存在 数据竞争， 读写 mut static var 是不安全的。
```rust
static mut COUNTER: u32 = 0;

fn add_to_counter(inc: u32) {
    unsafe { COUNTER += inc; }  // Potential data race!
}

fn main() {
    add_to_counter(42);

    unsafe { println!("COUNTER: {COUNTER}"); }  // Potential data race!
}
```

使用 mut static 是 坏主意。 有时，在 低级别的 no_std 代码中 有意义， 比如 实现 heap分配器 或 调用 C API。



### union

union 类似 enum，但是 你需要自己 追踪 有效的 field

```rust
#[repr(C)]
union MyUnion {
    i: u8,
    b: bool,
}

fn main() {
    let u = MyUnion { i: 42 };
    println!("int: {}", unsafe { u.i });
    println!("bool: {}", unsafe { u.b });  // Undefined behavior!
}
```

union 很少很少使用， 基本用 enum。 union只有在和 C API 交互的时候 会用到。

如果你想要将 字节 重新解释为 其他类型，那么你应该使用 `std::mem::transmute` 或 使用 safe wrapper (如 `zerocopy` crate)


### call unsafe func

func 或 method 可以被标记为 unsafe。

```rust
fn main() {
    let emojis = "🗻∈🌏";

    // Safe because the indices are in the correct order, within the bounds of
    // the string slice, and lie on UTF-8 sequence boundaries.
    unsafe {
        println!("emoji: {}", emojis.get_unchecked(0..4));
        println!("emoji: {}", emojis.get_unchecked(4..7));
        println!("emoji: {}", emojis.get_unchecked(7..11));
    }

    println!("char count: {}", count_chars(unsafe { emojis.get_unchecked(0..7) }));

    // Not upholding the UTF-8 encoding requirement breaks memory safety!
    // println!("emoji: {}", unsafe { emojis.get_unchecked(0..3) });
    // println!("char count: {}", count_chars(unsafe { emojis.get_unchecked(0..3) }));
}

fn count_chars(s: &str) -> usize {
    s.chars().count()
}
```

#### write unsafe func

```rust
/// Swaps the values pointed to by the given pointers.
///
/// # Safety
///
/// The pointers must be valid and properly aligned.
unsafe fn swap(a: *mut u8, b: *mut u8) {
    let temp = *a;
    *a = *b;
    *b = temp;
}

fn main() {
    let mut a = 42;
    let mut b = 66;

    // Safe because ...
    unsafe {
        swap(&mut a, &mut b);
    }

    println!("a = {}, b = {}", a, b);
}
```

在 unsafe fn 中 写 unsafe code， 可以不用 unsafe { } 。 可以通过 `#[deny(unsafe_op_in_unsafe_fn)]` 来禁止 (。即必须写 unsafe{ })

。。这个 #[deny ] 放哪里？ 。。 main 还是 unsafe fn 上？



#### call external code

通过 unsafe 调用 其他语言的代码

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        // Undefined behavior if abs misbehaves.
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

无法保证 外部函数 满足 rust的内存模型， 所以 要用 unsafe。
。。不过 其他语言 确实不满足。。

可用的ABI ， application binary interface：
https://doc.rust-lang.org/reference/items/external-blocks.html

。。C++ 好少。 就一个。 而且名字也怪。



### impl unsafe trait

和func 一样， 你可以标记 特征 为 unsafe ， 如果 实现 必须 保证 某些条件 以确保 不会触发 未定义行为。

例如， `zerocopy` crate 有一个 unsafe trait， 类似：
```rust
use std::mem::size_of_val;
use std::slice;

/// ...
/// # Safety
/// The type must have a defined representation and no padding.
pub unsafe trait AsBytes {
    fn as_bytes(&self) -> &[u8] {
        unsafe {
            slice::from_raw_parts(self as *const Self as *const u8, size_of_val(self))
        }
    }
}

// Safe because u32 has a defined representation and no padding.
unsafe impl AsBytes for u32 {}
```


### ==code，调用linux system call==

https://google.github.io/comprehensive-rust/exercises/day-3/safe-ffi-wrapper.html

```rust
    extern "C" {
        pub fn opendir(s: *const c_char) -> *mut DIR;

        #[cfg(not(all(target_os = "macos", target_arch = "x86_64")))]
        pub fn readdir(s: *mut DIR) -> *const dirent;

        // See https://github.com/rust-lang/libc/issues/414 and the section on
        // _DARWIN_FEATURE_64_BIT_INODE in the macOS man page for stat(2).
        //
        // "Platforms that existed before these updates were available" refers
        // to macOS (as opposed to iOS / wearOS / etc.) on Intel and PowerPC.
        #[cfg(all(target_os = "macos", target_arch = "x86_64"))]
        #[link_name = "readdir$INODE64"]
        pub fn readdir(s: *mut DIR) -> *const dirent;

        pub fn closedir(s: *mut DIR) -> c_int;
    }
```



## android

。。android， java kotlin。。
。。这里的android 估计指 android操作系统

Rust is supported for native platform development on Android
you can write new operating system services in Rust, as well as extending existing services.

。。对， native platform

。。跳


## bare metal

Bare metal is a computer system without a base operating system (OS) or installed applications.

。。没有os，那就是 单片机了？嗯，百度 "bare metal  单片机" 可以有相关联的页面
。。单片机的话， 汽车芯片 应该是 最火的。
。。但是单片机 新兴的语言 应该是 zig



## ==concurrency==

rust 支持 带有mutex，channel的 OS thread 进行并发

## thread

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("Count in thread: {i}!");
            thread::sleep(Duration::from_millis(5));
        }
    });

    for i in 1..5 {
        println!("Main thread: {i}");
        thread::sleep(Duration::from_millis(5));
    }
}
```

- 都是 守护线程， main线程 不会等待它们结束
- 线程panic 是相互独立的。
  - panic可以带有 payload，可以通过 `downcast_ref` 解开

。。守护线程：优先级低，不能让app继续运行 ( 所有非守护线程结束时，app结束 )。 app结束时，守护线程 被结束。


- 在达到10之前就结束了， 因为 main线程 不会等待它
- 使用 `let handle = thread::spawn(..)` ，`handle.join()` 来等待 线程结束
- 在 线程中 触发panic， 不会影响到 main 线程
- 使用 `Result` 从 `handle.join()` 接收返回值，可以 access 到 panic。( 这里用到了 `Any`)


### scoped thread (从环境中借用值)

普通线程 不能从 环境中 borrow 值
```rust
use std::thread;

fn foo() {
    let s = String::from("Hello");
    thread::spawn(|| {
        println!("Length: {}", s.len());
    });
}

fn main() {
    foo();
}
```

可以通过 scoped thread 来实现
```rust
use std::thread;

fn main() {
    let s = String::from("Hello");

    thread::scope(|scope| {
        scope.spawn(|| {
            println!("Length: {}", s.len());
        });
    });
}
```

这样做的原因是：当 thread::scope 结束时， 所有线程 都被保证 是 join，所以他们可以 返回 借用的值。
rust借用规则起效：你可以 只在一根线程 中 mut借用 ，也可以在多根线程中 immut借用

。。原因，无法理解。。The reason for that is that when the thread::scope function completes, all the threads are guaranteed to be joined, so they can return borrowed data.


## channel
rust channel 有2部分， `Sender<T>` , `Receiver<T>`。 这2部分 通过 channel 连接。
你只能看到 end-point， (。。看不到 channel)

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    tx.send(10).unwrap();
    tx.send(20).unwrap();

    println!("Received: {:?}", rx.recv());
    println!("Received: {:?}", rx.recv());

    let tx2 = tx.clone();
    tx2.send(30).unwrap();
    println!("Received: {:?}", rx.recv());
}
```

### unbounded channel

通过 `mpsc::channel()` 获得 unbouned(无限), 异步 的 channel

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let thread_id = thread::current().id();
        for i in 1..10 {
            tx.send(format!("Message {i}")).unwrap();
            println!("{thread_id:?}: sent Message {i}");
        }
        println!("{thread_id:?}: done");
    });
    thread::sleep(Duration::from_millis(100));

    for msg in rx.iter() {
        println!("Main: got {msg}");
    }
}
```


### bounded channel

使用 bounded(synchronous) channel， `send` 可以在 当前线程 block

。。channel 容量满了，所以 send 会 阻塞

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::sync_channel(3);

    thread::spawn(move || {
        let thread_id = thread::current().id();
        for i in 1..10 {
            tx.send(format!("Message {i}")).unwrap();
            println!("{thread_id:?}: sent Message {i}");
        }
        println!("{thread_id:?}: done");
    });
    thread::sleep(Duration::from_millis(100));

    for msg in rx.iter() {
        println!("Main: got {msg}");
    }
}
```

- 调用 send 会阻塞 线程， 除非 channel 有空间 可以放 新消息。 如果没有线程读取数据， send 线程 会无限等待。
- 当 receiver drop完后， channel 就close。
- 如果 channel 已经close， 调用 send 会 error (所以 send return `Result`)。 
- 如果 size 为0, 那么这个channel 被称为 "rendezvous channel"。 send 会阻塞线程，直到 另一个线程 调用 read。 。。 就是channel里 不存东西。



## Send, Sync

rust 怎么知道 如何 在 线程间 禁止 shared access ？ 答案是 2个 trait：
- Send：T 是 Send 的， 如果 T 可以安全地 在 线程间 move
- Sync：T 是 Sync 的， 如果 &T 可以安全地 在线程间 move

Send，Sync 是 unsafe trait。
如果你的type 只包含 Send 和 Sync 类型， 那么 编译器 自动 derive Send 和 Sync。

你也可以手动实现。

- 可以将 这些trait 当作 type是否 线程安全 的 标志
- 它们可以和 普通的trait 一样 用于 泛型 约束。



### Send

https://google.github.io/comprehensive-rust/concurrency/send-sync/send.html

如果可以安全地 move T类型的value 到另一个线程，那么这个 T 是 `Send`

移动所有权到另一个线程的 副作用是 会执行 析构器。所以问题就是，什么时候 你可以 在线程中 申请value，在另一个线程中 析构它。

。。但是 move 应该是 不析构的 move 吧。

### Sync

如果 可以同时 多线程 access(访问，存取) T类型的值，那么 T 就是 `Sync`
更准确地说，它的定义是： `T is Sync if and only if &T is Send`

。。access：访问，存取(计算机文件);  。 所以有没有 写的意思？
。。有， random access memory ...
。。read access， write access，有这2个单词的。
。。
。。而且， 只有读取，没有写入的话， 任何值 都是可以 并发的。。。 并发读是不会出现任何问题的， 只有 当有线程 写 的时候， 读写冲突，写写冲突，读读不会冲突


if a type is Sync it means that it can be shared across multiple threads without the risk of data races or other synchronization issues, so it is safe to move it to another thread. 
。。所以 access 还包括 构造，析构 的意思。


### Example


> Send + Sync

遇到的大多数类型是 Send + Sync
```text
i8, f32, bool, char, &str, …
(T1, T2), [T; N], &[T], struct { x: T }, …
String, Option<T>, Vec<T>, Box<T>, …
Arc<T>: Explicitly thread-safe via atomic reference count.
Mutex<T>: Explicitly thread-safe via internal locking.
AtomicBool, AtomicU8, …: Uses special atomic instructions.
```


> Send + !Sync

这些类型可以移动到其他线程，但是它们不是线程安全的。通常是因为 内部可变性
```text
mpsc::Sender<T>
mpsc::Receiver<T>
Cell<T>
RefCell<T>
```


> !Send + Sync

线程安全，但是不能移动到 其他线程

```text
MutexGuard<T: Sync>: Uses OS level primitives which must be deallocated on the thread which created them.
```



> !Send + !Sync

不是线程安全，且不能移动到其他线程

```text
Rc<T>: each Rc<T> has a reference to an RcBox<T>, which contains a non-atomic reference count.
*const T, *mut T: Rust assumes raw pointers may have special concurrency considerations.
```



## shared state

rust 使用 type system 来 强制 共享数据的 synchronization，主要通过 2个类型：
- `Arc<T>`, atomic reference counted T: 处理线程间的共享， 当最后一个ref drop时， 析构值
- `Mutex<T>`, 确保 对T值 的 独占access


### Arc

`Arc<T>` 通过 `Arc::clone` ，可以进行 shared read-only access

。。共享只读，还需要操作？  只读的东西，天然就是 线程安全的啊。  应该是 生命周期，所有权的 问题，导致 需要 clone。。 但是 没有mut ref 的时候， 可以同时存在 多个 immut ref 吧。 直接 immut ref 不行吗？ 非要 Arc？
。。这个得试下，主要是 看 编 译 器 报 什 么 错 误。
。。难道， 多个 immut ref 是指 单线程中 可以有 多个？ 估计是的。

### ==上面的code==

```rust
use std::thread;
use std::sync::Arc;

fn main() {
    let v = Arc::new(vec![10, 20, 30]);
    let mut handles = Vec::new();
    for _ in 1..5 {
        let v = Arc::clone(&v);
        handles.push(thread::spawn(move || {
            let thread_id = thread::current().id();
            println!("{thread_id:?}: {v:?}");
        }));
    }

    handles.into_iter().for_each(|h| h.join().unwrap());
    println!("v: {v:?}");
}
```

- Arc：atomic reference counted， Rc的线程安全版本， 通过 atomic operation 来完成。
- `Arc<T>` 实现了 Clone， 而不是 T 实现的。 `Arc<T>` 也会实现 Send 和 Sync，如果 T 都实现了这2个。
- Arc::clone() 的代价是 执行的 原子操作， 之后 使用 T 是没有 额外的消耗的。
- 小心 循环ref， Arc 没有 垃圾回收 来侦测它们。
  - std::sync::Weak 有助于 循环ref


### Mutex

`Mutex<T>` 确保 互斥，并且 允许在 只读接口 后面 对 T 进行 mut access

```rust
use std::sync::Mutex;

fn main() {
    let v = Mutex::new(vec![10, 20, 30]);
    println!("v: {:?}", v.lock().unwrap());

    {
        let mut guard = v.lock().unwrap();
        guard.push(40);
    }

    println!("v: {:?}", v.lock().unwrap());
}
```

。。Mutex 在什么地方  触发 unlock ？
。。应该是 出 scope的时候，就是 main的 最后一行之后。  中间 虽然有3次 lock， 但是 只有第一次 可能发生 竞争， 后2次，因为 已经 lock 了，所以 不会 竞争。  应该。。


注意，我们有 `impl<T: Send> Sync for Mutex<T>` 的一揽子实现 。 https://doc.rust-lang.org/std/sync/struct.Mutex.html#impl-Sync-for-Mutex%3CT%3E


- Mutex 就像 只有一个元素的 集合， 元素就是 受保护的数据
  - 在 access 受保护的数据前，不要忘记 获得 mutex
- 你可以 通过获得lock 从 `&Mutex<T>` 中获得 `&mut T`， `MutexGuard` 确保 `&mut T` 的范围不会 超过 lock的范围。
- `Mutex<T>` 实现了 `Send` 和 `Sync`， iff (if and only if)  T 实现了 `Send`
- read-write lock : `RwLock`
- 为什么 lock() 返回 Result ？
  - 如果 持有 Mutex 的线程 panic，那么 Mutex会变成 "poisoned" (中毒，败坏)，表明 protected data 可能处于 不一致状态。 对 posioned Mutex 调用 lock()，会失败，并赶回 PoisonError。你可以在出错时，对error 调用 into_inner() 来恢复数据。


。。what is mutexguard

### Example

Arc, Mutex

```rust
use std::thread;
// use std::sync::{Arc, Mutex};

fn main() {
    let v = vec![10, 20, 30];
    let handle = thread::spawn(|| {
        v.push(10);
    });
    v.push(1000);

    handle.join().unwrap();
    println!("v: {v:?}");
}
```

可能的解决方案

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let v = Arc::new(Mutex::new(vec![10, 20, 30]));

    let v2 = Arc::clone(&v);
    let handle = thread::spawn(move || {
        let mut v2 = v2.lock().unwrap();
        v2.push(10);
    });

    {
        let mut v = v.lock().unwrap();
        v.push(1000);
    }

    handle.join().unwrap();

    println!("v: {v:?}");
}
```

- `v` 被封装在 Arc 和 Mutex 中， 因为它们的关注点是正交的。
  - 将 `Mutex` 放到 `Arc` 中 是一个常见模式，来在线程间共享 可变的状态。
- `v: Arc<_>` 需要 clone 为 v2，在 move到另一个线程之前。 注意在 lambda 中增加了 move
- 引入区块是为了 尽可能缩小 LockGuard 的范围。

。。what is lockguard ？


### 哲学家就餐，==爬虫==

https://google.github.io/comprehensive-rust/exercises/concurrency/link-checker.html



## ==async rust==

async 是 并发模型：执行一个task 直到 block，然后切换 另一个task，来实现并发。
这个模型允许 执行 在 固定数量的 线程上 执行 大量task。这是因为每个任务的开销通常非常低，并且操作系统提供了 有效的方法 来识别 可处理的 IO。

rust的 异步操作 基于 "future", 它代表了 可能在未来 完成的 工作。future 会被 轮询，直到 它们完成。

future 被 async runtime 轮询， 并且有几个 不同的 runtime 可用。

比较
- python 有一个类似的模型在它的 `asyncio`。 但是，它的 `Future` 是基于 回调的，不是 轮询。 async python编程 需要 一个 "loop"，类似 rust的 运行时
- JavaScript 的 `Promise` 类似，但是 也是 基于 回调的。 language runtime 实现了 event loop，所以 许多 Promise 的 detail 被 隐藏了。


### async, await

在高层上，async rust code 看起来 非常像 "normal" sequential code

```rust
use futures::executor::block_on;

async fn count_to(count: i32) {
    for i in 1..=count {
        println!("Count is: {i}!");
    }
}

async fn async_main(count: i32) {
    count_to(count).await;
}

fn main() {
    block_on(async_main(10));
}
```

- 上面是一个简单的例子，展示了语法。 没有长时间运行的操作，没有 real concurrency
- async 的 return type 是什么？
  - 使用 `let future: () = async_main(10)`
- `async` 是 语法糖。 编译器会 将 return type 替换为 future
- 如果没有 向编译器 说明 如何使用 返回的 future 的话，不能使得 main方法 async。
- 你需要一个 ==executor 来 run async code==，`block_on` 阻塞了 当前线程，直到 future 完成。
- `.await` 异步等待 另一个操作的完成。 和 block_on 不同， await不会阻塞当前线程。
- `.await` 只能在 async 方法/块 内部使用。



### Future (trait)

Future 是一个 trait， 实现这个trait的对象 代表 一个操作可能还没有完成。
future可以被 轮询，poll方法 返回 Poll对象。

https://doc.rust-lang.org/std/future/trait.Future.html

https://doc.rust-lang.org/std/task/enum.Poll.html


```rust
use std::pin::Pin;
use std::task::Context;

pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

async func 返回 `impl Future`。 你也可以为你自己的type 实现 Future (但不推荐)。
例如，从 `tokio::spawn` 返回的 `JoinHandle` 实现了 Future 来 允许 join。

`.await` 关键字，应用到 Future 上，使得 当前 async func pause，直到 Future ready，然后 eval 出它的 output


- 上面的代码就是 Future， Poll的源码， 可以查看 之前的 url 来获得详细信息
- 我们没有用到 `Pin` 和 `Context`， 因为 我们 着重于 编写 async 代码， 而不是 构建 新的async 原语
  - Context， 允许 当event 发生时， Future可以让自己 重新schedule， 等待 poll
  - Pin，确保 内存中 Future 没有 move，这样的话，指向 Future 的指针 依然有效。这需要 允许 ref 在 `.await` 之后 依然有效。



### ==runtime==

runtime 提供以下支持：  异步执行operation (a reactor)， 负责执行 future (an executor)。
rust 没有内置的 runtime，一些可选项：
- Tokio: 高性能， 完善的生态系统，比如 HTTP使用Hyper，gRPC使用Tonic
- async-std: 旨在成为 "std for async"， 在 `async::task` 中包含了 一个基础的 runtime
- smol: 简单，轻量级

https://tokio.rs/
https://async.rs/
https://docs.rs/smol/latest/smol/

一些大型app 有它们自己的 runtime，比如 Fuchsia

https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/lib/fuchsia-async/src/lib.rs


- future 是 "inert"(迟钝，惰性) , 它们不会作任何事情 (甚至不会 启动 IO操作)，除非 有 executor poll 它们。


#### tokio

tokio 提供：
- 多线程 runtime，用于 执行 异步code
- 标准库 的一个 异步版本
- 一个庞大的 library的 生态环境

```rust
use tokio::time;

async fn count_to(count: i32) {
    for i in 1..=count {
        println!("Count in task: {i}!");
        time::sleep(time::Duration::from_millis(5)).await;
    }
}

#[tokio::main]
async fn main() {
    tokio::spawn(count_to(10));

    for i in 1..5 {
        println!("Main task: {i}");
        time::sleep(time::Duration::from_millis(5)).await;
    }
}
```

- 通过 `tokio::main` 宏， 可以使得 main 方法 async
- `spawn` 创建一个 新的，并发的 task
- spawn 接收 Future， 所以 count_to 后面不需要 .await

- 为什么 count_to (通常) 不会到 10？这是 异步取消的一个例子。 tokio::spawn 返回一个 handle，可以等待它完成。
- 尝试在 spawn 中 使用 count_to(10).await
- 尝试等待 tokio::spawn 中返回的 task



### task

rust 有 task 系统， 是 由轻量级线程组成

一个 task 有一个 顶级的 future，executor 会 poll 这个 future。 这个 顶级 future 可能有 多个 内嵌的 future，由 顶级future的 poll 方法 轮询。

使用 task 进行并发，可能会 poll 多个 child future。

```rust
use tokio::io::{self, AsyncReadExt, AsyncWriteExt};
use tokio::net::TcpListener;

#[tokio::main]
async fn main() -> io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:6142").await?;
    println!("listening on port 6142");

    loop {
        let (mut socket, addr) = listener.accept().await?;

        println!("connection from {addr:?}");

        tokio::spawn(async move {
            if let Err(e) = socket.write_all(b"Who are you?\n").await {
                println!("socket error: {e:?}");
                return;
            }

            let mut buf = vec![0; 1024];
            let reply = match socket.read(&mut buf).await {
                Ok(n) => {
                    let name = std::str::from_utf8(&buf[..n]).unwrap().trim();
                    format!("Thanks for dialing in, {name}!\n")
                }
                Err(e) => {
                    println!("socket error: {e:?}");
                    return;
                }
            };

            if let Err(e) = socket.write_all(reply.as_bytes()).await {
                println!("socket error: {e:?}");
            }
        });
    }
}
```

- 复制代码，放到 main.rs 中，并运行
- 尝试使用 nc 或 telnet  来 建立 tcp 连接。
    Ask students to visualize what the state of the example server would be with a few connected clients. What tasks exist? What are their Futures?

    This is the first time we’ve seen an async block. This is similar to a closure, but does not take any arguments. Its return value is a Future, similar to an async fn.

    Refactor the async block into a function, and improve the error handling using ?.



### async channel

一些 crate 支持 异步管道， 比如 tokio

```rust
use tokio::sync::mpsc::{self, Receiver};

async fn ping_handler(mut input: Receiver<()>) {
    let mut count: usize = 0;

    while let Some(_) = input.recv().await {
        count += 1;
        println!("Received {count} pings so far.");
    }

    println!("ping_handler complete");
}

#[tokio::main]
async fn main() {
    let (sender, receiver) = mpsc::channel(32);
    let ping_handler_task = tokio::spawn(ping_handler(receiver));
    for i in 0..10 {
        sender.send(()).await.expect("Failed to send ping.");
        println!("Sent {} pings so far.", i + 1);
    }

    drop(sender);
    ping_handler_task.await.expect("Something went wrong in ping handler task.");
}
```

- channel size 改成 3, 查看如何影响执行
- 尝试移除 std::mem::drop。
- `Flume` crate 有 实现了 sync和async，send和recv 的 channel。 可以被 IO 和 CPU都高压力的 复杂 app 方便地使用
- 使用 异步channel 的更可取的是 将它们与其他 future 组合到一起，从而 创建 复杂的 控制流。


## future control flow

future 可以组合到一起，来完成 并发流。
- Join
- Select

### join

join操作会等待，直到 所有的future ready，并返回 它们的 result的 集合。
类似于 JS中的 `Promise.all` 或 python中的 `asyncio.gather`

```rust
use anyhow::Result;
use futures::future;
use reqwest;
use std::collections::HashMap;

async fn size_of_page(url: &str) -> Result<usize> {
    let resp = reqwest::get(url).await?;
    Ok(resp.text().await?.len())
}

#[tokio::main]
async fn main() {
    let urls: [&str; 4] = [
        "https://google.com",
        "https://httpbin.org/ip",
        "https://play.rust-lang.org/",
        "BAD_URL",
    ];
    let futures_iter = urls.into_iter().map(size_of_page);
    let results = future::join_all(futures_iter).await;
    let page_sizes_dict: HashMap<&str, Result<usize>> =
        urls.into_iter().zip(results.into_iter()).collect();
    println!("{:?}", page_sizes_dict);
}
```


### select

select操作，等待， 直到有任意一个 future ready， 并且 返回 那个 future的 result。
类似 JS的 `Promise.race` , python的 `asyncio.wait(task_set, return_when=asyncio.FIRST_COMPLETED)`

```rust
use tokio::sync::mpsc::{self, Receiver};
use tokio::time::{sleep, Duration};

#[derive(Debug, PartialEq)]
enum Animal {
    Cat { name: String },
    Dog { name: String },
}

async fn first_animal_to_finish_race(
    mut cat_rcv: Receiver<String>,
    mut dog_rcv: Receiver<String>,
) -> Option<Animal> {
    tokio::select! {
        cat_name = cat_rcv.recv() => Some(Animal::Cat { name: cat_name? }),
        dog_name = dog_rcv.recv() => Some(Animal::Dog { name: dog_name? })
    }
}

#[tokio::main]
async fn main() {
    let (cat_sender, cat_receiver) = mpsc::channel(32);
    let (dog_sender, dog_receiver) = mpsc::channel(32);
    tokio::spawn(async move {
        sleep(Duration::from_millis(500)).await;
        cat_sender
            .send(String::from("Felix"))
            .await
            .expect("Failed to send cat.");
    });
    tokio::spawn(async move {
        sleep(Duration::from_millis(50)).await;
        dog_sender
            .send(String::from("Rex"))
            .await
            .expect("Failed to send dog.");
    });

    let winner = first_animal_to_finish_race(cat_receiver, dog_receiver)
        .await
        .expect("Failed to receive winner");

    println!("Winner is {winner:?}");
}
```


## async/await 的陷阱

async/await 提供了 方便高效 的 并发异步编程 的抽象。 但是 async/await 模型 也有 陷阱和不足：
- Blocking the Executor
- Pin
- Async Traits
- Cancellation

### blocking the executor

大部分 async runtime 都 只 允许 并发执行 IO任务。
这意味着 CPU blocking task 会 block executor
一个简单的解决方案是 尽可能使用 async 等效 方法

```rust
use futures::future::join_all;
use std::time::Instant;

async fn sleep_ms(start: &Instant, id: u64, duration_ms: u64) {
    std::thread::sleep(std::time::Duration::from_millis(duration_ms));
    println!(
        "future {id} slept for {duration_ms}ms, finished after {}ms",
        start.elapsed().as_millis()
    );
}

#[tokio::main(flavor = "current_thread")]
async fn main() {
    let start = Instant::now();
    let sleep_futures = (1..=10).map(|t| sleep_ms(&start, t, t * 10));
    join_all(sleep_futures).await;
}
```

- 运行代码，会发现 sleep 是 连续的，而不是 并发的。
- "current_thread" flavor 将所有的 task 放到一个 thread。 使得效果更明显。
- 把 `std::thread::sleep` 换成 `tokio::time::sleep` 
- 另一个fix 是 `tokio::task::spawn_blocking`， 会 spawn 一个 真实的线程，并 转移 它的 handle 到 future ，不会 block executor
- 不能将 task 认为时 OS线程。 它们不是 1比1 的， 大部分 executor 会在 一个 OS线程上 执行 许多task。 当通过 FFI 和 其他库 进行交互时，会有问题： 那个库 可能依赖于 thread-local 存储，或 映射到 指定的 OS线程 ( 比如，CUDA)。 这种情况下 使用 `tokio::task::spawn_blocking`
- 小心地 使用 sync mutex。 在 `.await` 上持有 mutex，可能导致 另一个task 阻塞，并且该task 可能在 同一个thread 上执行



### Pin

当你等待 future，所有的 local 变量 ( 通常保存在 stack frame中)  会保存到 当前异步块的 Future 中， 如果你的 future 有 指针 指向stack， 那么这些 指针 是无效的。

因此，你必须 确保， future 中指向 的 地址 没有被修改。 这就是 pin 的功能。

```rust
use tokio::sync::{mpsc, oneshot};
use tokio::task::spawn;
use tokio::time::{sleep, Duration};

// A work item. In this case, just sleep for the given time and respond
// with a message on the `respond_on` channel.
#[derive(Debug)]
struct Work {
    input: u32,
    respond_on: oneshot::Sender<u32>,
}

// A worker which listens for work on a queue and performs it.
async fn worker(mut work_queue: mpsc::Receiver<Work>) {
    let mut iterations = 0;
    loop {
        tokio::select! {
            Some(work) = work_queue.recv() => {
                sleep(Duration::from_millis(10)).await; // Pretend to work.
                work.respond_on
                    .send(work.input * 1000)
                    .expect("failed to send response");
                iterations += 1;
            }
            // TODO: report number of iterations every 100ms
        }
    }
}

// A requester which requests work and waits for it to complete.
async fn do_work(work_queue: &mpsc::Sender<Work>, input: u32) -> u32 {
    let (tx, rx) = oneshot::channel();
    work_queue
        .send(Work {
            input,
            respond_on: tx,
        })
        .await
        .expect("failed to send on work queue");
    rx.await.expect("failed waiting for response")
}

#[tokio::main]
async fn main() {
    let (tx, rx) = mpsc::channel(10);
    spawn(worker(rx));
    for i in 0..100 {
        let resp = do_work(&tx, i).await;
        println!("work result for iteration {i}: {resp}");
    }
}
```
https://google.github.io/comprehensive-rust/async/pitfalls/pin.html




### async trait

在 stable(稳定，持久的) channel 中 还不支持 trait 中的 async method

`async_trait` crate 提供了一种 解决方案

```rust
use async_trait::async_trait;
use std::time::Instant;
use tokio::time::{sleep, Duration};

#[async_trait]
trait Sleeper {
    async fn sleep(&self);
}

struct FixedSleeper {
    sleep_ms: u64,
}

#[async_trait]
impl Sleeper for FixedSleeper {
    async fn sleep(&self) {
        sleep(Duration::from_millis(self.sleep_ms)).await;
    }
}

async fn run_all_sleepers_multiple_times(sleepers: Vec<Box<dyn Sleeper>>, n_times: usize) {
    for _ in 0..n_times {
        println!("running all sleepers..");
        for sleeper in &sleepers {
            let start = Instant::now();
            sleeper.sleep().await;
            println!("slept for {}ms", start.elapsed().as_millis());
        }
    }
}

#[tokio::main]
async fn main() {
    let sleepers: Vec<Box<dyn Sleeper>> = vec![
        Box::new(FixedSleeper { sleep_ms: 50 }),
        Box::new(FixedSleeper { sleep_ms: 100 }),
    ];
    run_all_sleepers_multiple_times(sleepers, 5).await;
}
```

- `async_trait` 容易使用， 但是注意，它使用了 heap allocation 来完成功能，heap allocation 有性能开销



### cancellation

drop 一个 future， 意味着 这个 future 不能在被 poll。 这被称为  cancellation，它可能发生在 任何 await 点。
在 future 被cancel 时， 要确保 系统工作正常。 比如，不能 死锁，不能丢失数据


```rust
use std::io::{self, ErrorKind};
use std::time::Duration;
use tokio::io::{AsyncReadExt, AsyncWriteExt, DuplexStream};

struct LinesReader {
    stream: DuplexStream,
}

impl LinesReader {
    fn new(stream: DuplexStream) -> Self {
        Self { stream }
    }

    async fn next(&mut self) -> io::Result<Option<String>> {
        let mut bytes = Vec::new();
        let mut buf = [0];
        while self.stream.read(&mut buf[..]).await? != 0 {
            bytes.push(buf[0]);
            if buf[0] == b'\n' {
                break;
            }
        }
        if bytes.is_empty() {
            return Ok(None)
        }
        let s = String::from_utf8(bytes)
            .map_err(|_| io::Error::new(ErrorKind::InvalidData, "not UTF-8"))?;
        Ok(Some(s))
    }
}

async fn slow_copy(source: String, mut dest: DuplexStream) -> std::io::Result<()> {
    for b in source.bytes() {
        dest.write_u8(b).await?;
        tokio::time::sleep(Duration::from_millis(10)).await
    }
    Ok(())
}

#[tokio::main]
async fn main() -> std::io::Result<()> {
    let (client, server) = tokio::io::duplex(5);
    let handle = tokio::spawn(slow_copy("hi\nthere\n".to_owned(), client));

    let mut lines = LinesReader::new(server);
    let mut interval = tokio::time::interval(Duration::from_millis(60));
    loop {
        tokio::select! {
            _ = interval.tick() => println!("tick!"),
            line = lines.next() => if let Some(l) = line? {
                print!("{}", l)
            } else {
                break
            },
        }
    }
    handle.await.unwrap()?;
    Ok(())
}
```

- 编译器不会帮助 cancellation safety. 你需要阅读 api doc，思考你的 async fn 维持了 什么状态。

https://google.github.io/comprehensive-rust/async/pitfalls/cancellation.html



## ==其他的rust教程==

https://google.github.io/comprehensive-rust/other-resources.html

十几个

官方
https://doc.rust-lang.org/book/
https://doc.rust-lang.org/rust-by-example/
https://doc.rust-lang.org/std/
https://doc.rust-lang.org/reference/

https://doc.rust-lang.org/nomicon/
https://rust-lang.github.io/async-book/
https://doc.rust-lang.org/stable/embedded-book/


---
非官方
http://cliffle.com/p/dangerust/
https://docs.opentitan.org/doc/ug/rust_for_c/
https://overexact.com/rust-for-professionals/
https://exercism.org/tracks/rust
https://ferrous-systems.github.io/teaching-material/index.html
https://docs.microsoft.com/en-us/shows/beginners-series-to-rust/ and https://docs.microsoft.com/en-us/learn/paths/rust-first-steps/   ==microsoft==
https://rust-unofficial.github.io/too-many-lists/


查看更多书
https://lborb.github.io/book/



## solution

之前 exercises 的答案

2023-11-15 17:14

---

