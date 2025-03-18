
Rust程序设计(第二版)

2024-04-15 13:21

[[toc]]

---

---

补充材料（代码示例、练习等）可在
https://github.com/ProgrammingRust 下载。




不，写到 language/rust-24中。
不，rust-24 有点卡。。 应该是太多内容了，2万3千多行，48万个字符。。



Rust 用来确保内存安全的那些限制同样能确保 Rust 程序避免产生数据竞争（data race）。
只要数据不可变，你就可以在线程之间自由地共享这些数据。
会发生变化的数据则只能使用同步原语访问。所有传统的并发工具仍然可用：==互斥锁、条件变量、通道、原子等==。Rust 只负责检查你是否正确地使用了它们。

Rust 的生态系统提供了一些超乎于常规并发原语的库，可帮助你在==处理器池==之间均匀分布复杂负载、使用==无锁同步机制（如读取-复制-更新）==等。


C++ 的各种实现会遵循==零开销原则==：没用到的，就没有开销；要用到的，你也无法手写出更好的代码。

如果你准备让程序充分发挥底层机器的能力，那么 Rust 就会为你提供支持。
这门语言设计了一些“高性能”的默认选项，并赋予你==自主控制内存使用==和==处理器算力分配==方式的能力


借助 Rust 的==特型（trait）和泛型==，我们可以创建具有灵活接口的库，将其用在许多
不同的上下文中。



# ch2 Rust 导览

安装 Rust 的最佳方式是使用 rustup。请转到 rustup.rs 网站

。书上是 1.49 。。 2020-12-05 的

rust升级： `rustup update`

cargo, 编译管理器、包管理器和通用工具。
rustc,  编译器, 通常 Cargo 会替我们调用此编译器
rustdoc, Rust 文档工具，从源码中提取注释 形成 html， 通常 cargo会替我们运行

建立工程
`cargo new hello`

`cargo new htllo --vsc none`
可以不创建 git 的东西

可以在包内的任意目录下调用, 来构建和运行程序
`cargo run`

清理生成的文件
`cargo clean`

`cargo test`

---

```rust
fn gcd(mut n: u64, mut m: u64) -> u64 {
    assert!(n != 0 && m != 0);
    while m != 0 {
        if m < n {
            let t = m;
            m = n;
            n = t;
        }
        m = m % n;
    }
    n
}
```

。。mut n: u64,  not : mut



花括号包起来的任意代码块都可以用作表达式。
下面是一个打印了一条信息然后以 x.cos() 作为其值的表达式：
```rust
{
    println!("evaluating cos x");
    x.cos()
}
```


---

src/main.rs 的末尾添加
```rust
#[test]
fn test_gcd() {
    assert_eq!(gcd(14, 15), 1);
    assert_eq!(gcd(2 * 3 * 5 * 11 * 17, 3 * 7 * 11 * 13 * 19), 3 * 11);
}
```

`#[test]` 正常编译时会跳过它

`cargo test`

`#[test]` 标记是属性（attribute）的示例之一



---
处理命令行参数

```rust
use std::str::FromStr;
use std::env;
fn main() {
    let mut numbers = Vec::new();
    for arg in env::args().skip(1) {
        numbers.push(u64::from_str(&arg).expect("error parsing argument"));
    }
    if numbers.len() == 0 {
        eprintln!("Usage: gcd NUMBER ...");
        std::process::exit(1);
    }
    let mut d = numbers[0];
    for m in &numbers[1..] {    // borrow/ref
        d = gcd(d, *m);     // ****m, de-ref
    }
    println!("The greatest common divisor of {:?} is {}",
    numbers, d);
}
```
`cargo run 42 56`

第一个 use 声明将标准库中的 FromStr 特型引入了当前作用域。
特型是可以由类型实现的方法集合。
任何实现了 FromStr 特型的类型都有一个 from_str 方法，该方法会尝试从字符串中解析这个类型的值。
u64 类型实现了 FromStr，所以我们将调用u64::from_str 来解析程序中的命令行参数。
尽管我们从未在程序的其他地方用到 FromStr 这个名字，但仍然要 use（使用）它，因为要想使用某个特型的方法，该特型就必须在作用域内。

。。现在应该不需要了吧？

std::env 模块的 args 函数会返回一个迭代器，此迭代器会按需生成1每个参数

用 Result 的 expect 方法来检查本次解析是否成功。
如果结果是 Err(e)，那么 expect 就会打印出一条包含 e 的消息并直接退出程序。
但如果结果是 Ok(v)，则 expect 会简单地返回 v 本身



---

## 2.5 web server

。。
cargo 下载的库保存在： /home/xxxx/.cargo/registry/src


```text
[dependencies]
actix-web = "4.5.1"
serde = { version = "1.0.198", features = ["derive"]}
```
。。新老版本 内容差太多了。。反正起不动，而且看了下 官方的example， hello world 都很复杂。。

```text
error[E0277]: the trait bound `fn() -> HttpResponse {get_index}: Handler<_>` is not satisfied
   --> src/main.rs:7:45
    |
7   |         App::new().route("/", web::get().to(get_index))
    |                                          -- ^^^^^^^^^ the trait `Handler<_>` is not implemented for fn item `fn() -> HttpResponse {get_index}`
```


。版本退到 1.0.8 


```text
/.cargo/registry/src/index.crates.io-6f17d22bba15001f/actix-http-0.2.11/src/cookie/jar.rs:226:44
    |
226 |             cookie.set_expires(time::now() - Duration::days(365));
    |                                            ^ no implementation for `Tm - TimeDelta`
```
。。这怎么玩？

。。gg

https://github.com/actix/actix-web/issues/3135
有人问了。


`pin chrono to v0.4.29 in your manifest file.`   还好有人也直接说了 代码。。 我用 `chrono="0.4.29"` 没起效，还是报上面的错误。。 仓库里有 `chrono-0.4.38`， 看来 xx="1.1.2" 是 必须 大于等于 1.1.2 , 不是指定。
```text
chrono = { version = "= 0.4.29" }
```
。。也不是啊。 在2.6中 也需要 版本回退。 直接改就可以了。


启动成功
```text
$ cargo run
    Updating crates.io index
  Downloaded chrono v0.4.29
  Downloaded 1 crate (209.7 KB) in 12.47s
   Compiling chrono v0.4.29
   Compiling actix-http v0.2.11
   Compiling awc v0.2.8
   Compiling actix-web v1.0.9
   Compiling ch02_actix_gcd v0.1.0 (/run/media/asdf/239G/my_github/my_rust_code/programming_rust/ch02_actix_gcd)
    Finished dev [unoptimized + debuginfo] target(s) in 24.31s
     Running `target/debug/ch02_actix_gcd`
serving on localhost:3000
```





`#[derive(Deserialize)]`      // 这个注解 要求 serde crate 在程序编译时检查此类型并自动生成代码，以便从 HTML 表单 POST 提交过来的格式化数据中解析出此类型的值



ok 可以网页计算 gcd

。。看了下文件。。  target 中 1.5g 。。。
。。必须 cargo clean




## 2.6 并发

如果使用互斥锁来协调对共享数据结构进行更改的多个线程，那么 Rust 会确保只有持有锁才能访问这些数据，并会在完工后自动释放锁。

如果想在多个线程之间共享只读数据，那么 Rust 能确保你不会意外修改数据。

如果将数据结构的所有权从一个线程转移给另一个线程，那么Rust 能确保你真的放弃了对它的所有访问权限。


曼德博集


666

a = fun()?;

---

Complex 是一种泛型结构体
`Complex<f64>`


```rust
// 泛型函数
/// 把字符串`s`（形如`"400x600"`或`"1.0,0.5"`）解析成一个坐标对
fn parse_pair<T: FromStr>(s: &str, separator: char) -> Option<(T, T)> {
    match s.find(separator) {
        None => None,
        Some(index) => {
            match (T::from_str(&s[..index]), T::from_str(&s[index + 1..])) {
                (Ok(l), Ok(r)) => Some((l, r)),
                _ => None
            }
        }
    }
}
```




let output = File::create(filename)?;
// 等价于
// let output = match File::create(filename) {
//     Ok(f) => f,
//     Err(e) => {
//     return Err(e);
//     }
// };





`let mut pixels = vec![0; bounds.0 * bounds.1];`
`render(&mut pixels, bounds, upper_left, lower_right);`

`fn render(pixels: &mut [u8], bounds: (usize, usize), upper_left: Complex<f64>, lower_right: Complex<f64>)`





`cargo build --release`
`time target/release/mandelbrot mandel.png 4000x3000 -1.20,0.35 -1,0.20`




..换了个 cargo 的镜像。。build 太慢了。 换了好几个

```text
99 | use image::png::PNGEncoder;
   |            ^^^ could not find `png` in `image`
```
`https://docs.rs/image/0.20.1/image/?search=PNGEncoder`

在0.25.1 中是 `image::codecs::png::PngEncoder`
0.20.1 是 `image::png::PNGEncoder`

版本回退。


```text
time target/release/ch02_multithread_mandelbrot mandel.png 4000x3000 -1.20,0.35 -1,0.20

real	0m3.545s
user	0m3.512s
sys	0m0.010s
```


ok 图片和书上一模一样

target 380mb

---
concurreny

```text
$ time target/release/ch02_multithread_mandelbrot mandel.png 4000x3000 -1.20,0.35 -1,0.20

real	0m1.117s
user	0m3.627s
sys	0m0.008s
```



## 2.7 文件系统与命令行工具

读写文件

正则查找


# ch3 基本数据类型

Rust有类型推断

```rust
fn build_vector() -> Vec<i16> {
    let mut v: Vec<i16> = Vec::<i16>::new();
    v.push(10i16);
    v.push(20i16);
    v
}
```
既然已知函数的返回类型，那么显然 v 只能是一个 `Vec<i16>`

```rust
fn build_vector() -> Vec<i16> {
    let mut v = Vec::new();
    v.push(10);
    v.push(20);
    v
}
```

## 类型大全

- i8,i16,i32,i64,i128
  定宽有符号整数
- u8,u16,u32,u64,u128
  定宽无符号整数
- isuze,usize
  与机器字(32bit, 64bit) 一样大的 有符号，无符号 整数
- f32,f64
  单精度，双精度
- bool
  布尔值
- char
  unicode字符，==4字节==
- (char, u8, i32)
  元组，可以混合类型
- ()
  单元，(空元组)
- struct S { x: f32, y: f32}
  具名字段型结构体
- struct T (i32, char)
  元组型结构体
- struct E
  单元型结构体，无字段
- enum Attend { OnTime, Late(u32) }
  枚举，或 代数数据类型
- `Box<Attend>`
  Box：==指向堆中值 的 拥有型指针==
- &i32, &mut i32
  共享ref 和 可变ref， 非拥有型指针。 引用的生命周期不能超过 被引用的目标
- String
  ==utf-8== 字符串，动态分配大小
- &str
  对 str 的引用，指向 utf-8 文本的 非拥有型指针
- [f64; 4], [u8; 256]
  数组
- `Vec<f64>`
  vector
- &[u8], *mut [u8]
  ==对切片(数组或vec的一部分)的引用，包含指针和长度==
- Option<&str>
  可选值，None 或 Some(v)
- `Result<u64, Error>`
  可能失败的操作结果， Ok(v) 或 Err(e)
- &dyn Any, &mut dyn Read
  特型对象，是对任何实现了 一组给定方法的值的 引用
- fn(&str) -> bool
  函数指针
- 闭包


。。Unicode是一种字符编码标准，它定义了字符和数字之间的映射关系；UTF-8是一种用于实现Unicode的编码方式。
。。UTF-8与ASCII编码兼容
。Unicode使用固定长度的编码方式，UTF-8采用可变长度的编码方式
。由于UTF-8的灵活性和存储效率，它在处理包含多种语言的文本时更为合适；Unicode则常用于内存中的字符表示。
。Unicode是内存中字符表示的标准，而UTF-8是将其转换为字节序列以进行存储和传输的一种方式。



Rust 会使用 u8 类型作为==字节值==
从二进制文件或套接字中读取数据时会产生一个 u8 值构成的流

数字加下划线，4_294_967_295
127_u8， 127u8
0x、0o 和 0b 分别表示十六进制字面量、八进制字面量和二进制字面量

尽管数值类型和 char 类型是不同的，但 Rust 确实为 u8 值提供了字节字面量。
与字符字面量类似，b'X' 表示以字符 X 的 ASCII码作为 u8 值。
例如，由于 A 的 ASCII 码是 65，因此字面量b'A' 和 65u8 完全等效。
只有 ASCII 字符才能出现在字节字面量中

对于难于书写或阅读的字符，可以将其编码改为十六进制。
这种字节字面量形如 b'\xHH'，其中 HH 是任意两位十六进制数，表示值为HH 的字节。
例如，你可以将 ASCII 控制字符 escape 的字节字面量写成 b'\x1b'，因为 escape 的 ASCII 码为 27，即十六进制的1B。
只有当你要强调该值表示的是 ASCII 码时，才应该使用 b'\x1b' 而不是简单明了的 27。

```rust
assert_eq!(2_u16.pow(4), 16); // 求幂
assert_eq!((-4_i32).abs(), 4); // 求绝对值
assert_eq!(0b101101_u8.count_ones(), 4); // 求二进制1的个数
```

方法调用的优先级高于一元前缀运算符
`-4_i32.abs()` , 先调用abs，然后 取负

## 检查算法、回绕算法、饱和算法和溢出算法

当整型算术运算溢出时，Rust 在==调试==构建中会出现 ==panic==。
而在==发布==构建中，运算会==回绕==：它生成的值等于“数学意义上正确的结果”对“值类型范围”取模的值。

下面 乘法 溢出时 触发 panic
`i = i.checked_mul(10).expect("multiplication overflowed");`

分为4类
检查运算会返回结果的 Option 值
```rust
// 10与20之和可以表示为u8
assert_eq!(10_u8.checked_add(20), Some(30));
// 很遗憾，100与200之和不能表示为u8
assert_eq!(100_u8.checked_add(200), None);
// 做加法。如果溢出，则会出现panic
let sum = x.checked_add(y).unwrap();
// 奇怪的是，在某种特殊情况下，带符号的除法也会溢出。
// 带符号的n位类型可以表示-2^(n-1)，但不足以表示2^(n-1)
assert_eq!((-128_i8).checked_div(-1), None)
```

回绕运算会返回与“数学意义上正确的结果”对“值类型范围”取模的值相等的值。
```rust
// 第一个结果可以表示为u16，第二个则不能，所以会得到250000 对2^16的模
assert_eq!(100_u16.wrapping_mul(200), 20000);
assert_eq!(500_u16.wrapping_mul(500), 53392);
// 对有符号类型的运算可能会回绕为负值
assert_eq!(500_i16.wrapping_mul(500), -12144);
// 在移位运算中，移位距离会在值的大小范围内回绕，
// 所以在16位类型中移动17位就相当于移动了1位
assert_eq!(5_i16.wrapping_shl(17), 10);
```

饱和运算会返回最接近“数学意义上正确结果”的可表达值。换句话说，结果“紧贴着”该类型可表达的最大值和最小值。

不存在饱和除法 、饱和求余法 或饱和位移法 
```rust
assert_eq!(32760_i16.saturating_add(10), 32767);
assert_eq!((-32760_i16).saturating_sub(10), -32768)
```

溢出运算会返回一个元组 (result, overflowed)，其中result 是函数的回绕版本所返回的内容，而 overflowed 是一个布尔值，指示是否发生过溢出。
```rust
assert_eq!(255_u8.overflowing_sub(2), (253, false));
assert_eq!(255_u8.overflowing_add(2), (1, true));
```
实际应用的移位数是所请求的移位数对类型位宽取模的结果。
```rust
// 移动17位对`u16`来说太大了，而17对16取模就是1
assert_eq!(5_u16.overflowing_shl(17), (10, true));
```


前缀 checked_、wrapping_、saturating_ 或 overflowing_后面可以跟着的运算名
add, sub, mul, div, rem(求余), neg, abs, pow, shl(按位左移), shr

---
浮点数

`31231.925e-4f64`

小数部分可以仅由一个单独的小数点组成，因此 5. 也是有效的浮点常量。

如果它最终发现这两种浮点类型都适合，就会默认选择 f64。

永远不会把整型字面量推断为浮点类型，反之亦然


f32 类型和 f64 类型具有 IEEE 要求的一些特殊值的关联常量
比如 INFINITY（无穷大）、NEG_INFINITY（负无穷大）、NAN（非数值）以及 MIN（最小有限值）和 MAX（最大有限值）：

```rust
assert!((-1. / f32::INFINITY).is_sign_negative());
assert_eq!(-f32::MIN, f32::MAX);
```

提供了完备的数学计算方法
```rust
assert_eq!(5f32.sqrt() * 5f32.sqrt(), 5.); // 按IEEE的规定，它精确等于5.0
assert_eq!((-1.01f64).floor(), -2.0)
```

方法调用的优先级高于前缀运算符，因此在对负值进行方法调用时，请务必正确地加上圆括号

std::f32::consts 模块和 std::f64::consts 模块提供了各种常用的数学常量，比如 E、PI 和 2 的平方根。

与整数一样，通常不必在实际代码中写出浮点字面量的类型后缀，但如果你想这么做，那么将类型放在字面量或函数上就可以：
```rust
println!("{}", (2.0_f64).sqrt());
println!("{}", f64::sqrt(2.0));
```

---

Rust 的 as 运算符可以将 bool 值转换为整型
as 无法进行另一个方向（从数值类型到 bool）的转换

```rust
assert_eq!(false as i32, 0);
assert_eq!(true as i32, 1);
```

Rust 在内存中会使用整字节来表示 bool 值，因此可以创建指向它的指针


---
字符类型 char 会以 32 位值表示单个 Unicode 字符

如果字符的码点在 U+0000 到 U+007F 范围内（也就是说，如果它是从 ASCII 字符集中提取的），就可以把字符写为'\xHH'，其中 HH 是两个十六进制数。

可以将任何 Unicode 字符写为 '\u{HHHHHH}' 形式


char 永远不会是“半代用区”中的码点（0xD800到 0xDFFF 范围内的码点，它们不能单独使用）或 Unicode 码点空间之外的值（大于 0x10FFFF 的值）。

Rust 不会在 char 和任何其他类型之间进行隐式转换。
可以使用as 转换运算符将 char 转换为整型，对于小于 32 位的类型，该字符值的高位会被截断


---

## 3.4 元组

元组是各种类型值的 值对 或 三元组、四元组、五元组等

("Brazil", 1985) 是一个元组，类型是 (&str, i32)

给定一个元组值 t，可以通过 t.0、t.1 等访问其元素。

通常会用元组类型从一个==函数返回多个值==。
`fn split_at(&self, mid: usize) -> (&str, &str);`

### 模式匹配
可以用 ==模式匹配== 语法将返回值的每个元素赋值给不同的变量
```rust
let text = "I see the eigenvalue in thine eye";
let (head, tail) = text.split_at(21);
assert_eq!(head, "I see the eigenvalue ");
assert_eq!(tail, "in thine eye");
```


另一种常用的元组类型是零元组 ()。
传统上，这叫作单元类型，因为此类型只有一个值，写作 ()。
当无法携带任何有意义的值但其上下文仍然要求传入某种类型时，Rust 就会使用单元类型。

==不返回值的函数的返回类型为 ()==
。可以省略函数的返回部分： -> () 


可以在元组的最后一个元素之后跟上一个逗号：类型(&str, i32,) 和 (&str, i32) 是等效的


Rust始终允许在所有能用逗号的地方（函数参数、数组、结构体和枚举定义，等等）添加额外的尾随逗号


有包含单个值的元组。字面量 ("lonely hearts",) 就是一个包含单个字符串的元组，它的类型是(&str,)。
在这里，==值后面的逗号是必需的==，以用于区分 单值元组 和 简单的 括号表达式。


## 3.5 指针类型

在 Java 中，如果 class Rectangle 包含字段 Vector2D upperLeft;，那么 upperLeft 就是对另一个单独创建的Vector2D 对象的引用。
在 Java 中，一个对象永远不会包含其他对象的实际内容。


默认情况下值会嵌套。值 ((0, 0), (1440, 900)) 会存储为 4 个相邻的整数。
如果将它存储在一个局部变量中，则会得到 4 倍于整数宽度的局部变量。
堆中没有分配任何内容。

当 Rust 程序需要让一些值指向其他值时，必须显式使用指针类型

3 种指针类型：引用、Box 和不安全指针。


### 引用

&String 类型的值（读作“ref String”）是对 String 值的引用，&i32 是对 i32 的引用，以此类推。

最简单的方式是将引用视为 Rust 中的基本指针类型。

表达式 &x 会生成一个对 x 的引用，在 Rust 术语中，我们会说它借用了对 x 的引用。

`numbers.push(u64::from_str(&arg).expect("error parsing argument"));`

给定一个引用 r，表达式 *r 会引用 r 指向的值。

它们非常像 C 和 C++ 中的 & 运算符和 * 运算符，并且和 C 中的指针一样，当超出作用域时引用不会自动释放任何资源

Rust 的引用永远不会为空

2种形式
- &T
  不可变的共享引用。你可以同时拥有多个对给定值的共享引用，但它们是只读的
- &mut T
  一个可变的、独占的引用


### Box

在堆中分配值的最简单方式是使用 Box::new

```rust
let t = (12, "eggs");
let b = Box::new(t); // 在堆中分配一个元组
```

t 的类型是 (i32, &str)，所以 b 的类型是 `Box<(i32, &str)>`。
对 Box::new 的调用会分配足够的内存以在堆上容纳此元组。
当 b 超出作用域时，内存会立即被释放，除非 b 已被移动（move），比如返回它。
移动对于 Rust 处理在堆上分配的值的方式至关重要，第 4 章会对此进行详细解释。


### 裸指针

Rust 也有裸指针类型 *mut T 和 *const T。
裸指针实际上和 C++中的指针很像。
使用裸指针是==不安全==的，因为 Rust 不会跟踪它指向的内容。
例如，裸指针可能为空，或者它们可能指向已释放的内存或现在包含不同类型的值。

只能在 unsafe 块中对裸指针解引用


## 3.6 数组，向量，切片

- 类型 `[T; N]`，N个值，类型都是T，编译期已知大小，不能追加或缩小
- 类型 `Vec<T>`，动态分配，可增长/缩小的 T 类型的 值序列。 ==元素存放在堆中==。
- 类型 `&[T]`,`&mut [T]`，称为 T 的 共享切片， T的可变切片。 是对一系列元素的引用，这些元素是 其他值(如 数组或向量)的 一部分。

3种类型都可以 v.len()。 v[i]。

---
### 数组

```rust
let lazy_caterer: [u32; 6] = [1, 2, 4, 7, 11, 16];
let taxonomy = ["Animalia", "Arthropoda", "Insecta"];

let mut sieve = [true; 10000];
```

#### 固定大小缓冲区

声明==固定大小缓冲区==的语法：`[0u8; 1024]`，它是一个1 KB 的缓冲区，用 0 填充。

Rust 没有任何 能定义 未初始化数组的写法。 一般来说，Rust 会确保代码永远无法访问任何种类的未初始化值。

数组的长度是其类型的一部分，并会在编译期固定下来

你在数组上看到的那些实用方法（遍历元素、搜索、排序、填充、过滤等）都是作为==切片==而非数组的方法提供的。
但是 Rust 在搜索各种方法时会==隐式地将对数组的引用转换为切片==，因此可以直接==在数组上调用任何切片方法==：

```rust
let mut chaos = [3, 5, 4, 1, 2];
chaos.sort();
```
sort 方法实际上是在切片上定义的，但由于它是通过引用获取的操作目标，因此 Rust 会隐式地生成一个引用整个数组的 `&mut [i32]` 切片，并将其传给 sort 来进行操作。


### 向量
向量 `Vec<T>` 是一个可调整大小的 T 类型元素的数组，它是在堆上分配的。

创建向量的方法有好几种
- 最简单的是 vec! 宏
```rust
let mut primes = vec![2, 3, 5, 7];
assert_eq!(primes.iter().product::<i32>(), 210);
```

vec! 宏==相当于==调用 Vec::new 来创建一个新的空向量，然后将元素压入其中

- Vec::new
```rust
let mut pal = Vec::new();
pal.push("step");
pal.push("on");
pal.push("no");
pal.push("pets");
assert_eq!(pal, vec!["step", "on", "no", "pets"]);
```

- 从迭代器生成的值构建一个向量
```rust
let v: Vec<i32> = (0..5).collect();
assert_eq!(v, [0, 1, 2, 3, 4]);
```
使用 collect 时，通常要指定类型（正如此处给 v 声明了类型）

与数组一样，可以对向量使用切片的方法
```rust
let mut palindrome = vec!["a man", "a plan", "a canal", "panama"];
palindrome.reverse();
```
reverse 方法实际上是在切片上定义的，但是此调用会隐式地从此向量中借用一个 `&mut [&str]` 切片并在其上调用 reverse。

还有许多其他方法可以构建新向量或扩展现有向量。第 16 章会介绍这些方法。

`Vec<T>` 由 3 个值组成：
- 指向元素在堆中分配的缓冲区（该缓冲区由 `Vec<T>` 创建并拥有）的指针、
- 缓冲区能够存储的元素数量，
- 它现在实际包含的数量（也就是它的长度）。

当缓冲区达到其最大容量时，往向量中添加另一个元素需要分配一个更大的缓冲区，将当前内容复制到其中，更新向量的指针和容量以指向新缓冲区，最后释放旧缓冲区。

如果事先知道向量所需的元素数量，就可以调用 Vec::with_capacity 而不是 Vec::new 来创建一个向量
vec! 宏就使用了这样的技巧，因为它知道最终向量将包含多少个元素


可以在向量中任意位置插入元素和移除元素，不过这些操作会将受影响位置之后的所有元素向前或向后移动，因此如果向量很长就可能很慢
```rust
let mut v = vec![10, 20, 30, 40, 50];
// 在索引为3的元素处插入35
v.insert(3, 35);
assert_eq!(v, [10, 20, 30, 35, 40, 50]);
// 移除索引为1的元素
v.remove(1);
assert_eq!(v, [10, 30, 35, 40, 50]);
```

可以使用 pop 方法移除最后一个元素并将其返回。
更准确地说，从 `Vec<T>` 中弹出一个值会返回 `Option<T>`


可以使用 for 循环遍历向量
```rust
let languages: Vec<String> = std::env::args().skip(1).collect();
for l in languages {
}
```

Vec 仍然是 Rust 中定义的普通类型，而没有内置在语言中


### 切片

切片（写作不指定长度的 `[T]`）是数组或向量中的一个区域。
由于切片可以是任意长度，因此它不能直接存储在变量中或作为函数参数进行传递。
切片==总是通过引用传递==。

对切片的引用是一个胖指针：一个双字值，包括指向切片第一个元素的指针和切片中元素的数量。

```rust
let v: Vec<f64> = vec![0.0, 0.707, 1.0, 0.707];
let a: [f64; 4] = [0.0, -0.707, -1.0, -0.707];
let sv: &[f64] = &v;
let sa: &[f64] = &a;
```

在最后两行中，Rust ==自动==把 `&Vec<f64>` 的引用和 `&[f64; 4]` 的引用转换成了直接指向数据的切片引用。

。。引用 转换为 切片引用

![683d5d2df842e2f8a0c144c8c1165e85.png](../_resources/683d5d2df842e2f8a0c144c8c1165e85.png)


普通引用是指向单个值的非拥有型指针，而对切片的引用是指向内存中一系列连续值的非拥有型指针。
如果要写一个对 数组 或 向量 进行操作 的函数，那么 切片引用 就是不错的选择。

因为此函数以切片引用为参数，所以也可以给它传入向量或数组。
事实上，你以为属于向量或数组的许多方法其实是在切片上定义的，比如会对元素序列进行排序或反转的 sort 方法和 reverse 方法实际上是切片类型 `[T]` 上的方法。

你可以使用范围值对数组或向量进行索引，以获取一个切片的引用，该引用既可以指向数组或向量，也可以指向一个既有切片：

```rust
print(&v[0..2]); // 打印v的前两个元素
print(&a[2..]); // 打印从a[2]开始的元素
print(&sv[1..3]); // 打印v[1]和v[2]
```

由于切片几乎总是出现在引用符号之后，因此通常只将 `&[T]` 或 &str 之类的类型称为“切片”，使用较短的名称来表示更常见的概念



## 3.7 字符串类型

### 字符串字面量

字符串字面量要用双引号括起来，它们使用与 char 字面量相同的反斜杠转义序列
`let speech = "\"Ouch!\" said the well.\n";`


一个字符串可能跨越多行：
```rust
println!("In the room the women come and go,
 Singing of Mount Abora");
```
换行符是字符串的一部分,第 2 行开头的空格也是如此。


如果字符串的一行以反斜杠结尾，那么就会丢弃其后的换行符和前导空格
```rust
println!("It was a bright, cold day in April, and \
 there were four of us—\
 more or less.");
```
这会打印出单行文本。

---

在少数情况下，需要双写字符串中的每一个反斜杠，这让人不胜其烦
Rust 提供了原始字符串。原始字符串用小写字母 r 进行标记。

原始字符串中的所有反斜杠和空白字符都会逐字包含在字符串中。
原始字符串不识别任何转义序列

```rust
let default_win_install_path = r"C:\Program Files\Gorillas";
let pattern = Regex::new(r"\d+(\.\d+)*");
```

不能简单地在双引号前面放置一个反斜杠来包含原始字符串
可以在原始字符串的开头和结尾添加 # 标记：
```rust
println!(r###"
 This raw string started with 'r###"'.
 Therefore it does not end until we reach a quote mark ('"')
 followed immediately by three pound signs ('###'):
"###);
```

。。 `r###"` 需要匹配 `"###`


### 字节串

带有 b 前缀的字符串字面量都是字节串。
这样的==字节串是 u8 值（字节）的切片== 而不是 Unicode 文本：

```rust
let method = b"GET";
assert_eq!(method, &[b'G', b'E', b'T']);
```
method 的类型是 &[u8; 3]

字节串可以使用前面展示过的所有其他的字符串语法：
- 可以跨越多行、
- 可以使用转义序列、
- 可以使用反斜杠来连接行等。
- 不过原始字节串要以 br" 开头。

字节串不能包含任意 Unicode 字符，它们只能使用 ASCII 和 \xHH 转义序列


### 内存中的字符串

Rust 字符串是 Unicode 字符序列，但它们并没有以 char 数组的形式存储在内存中，而是使用了 UTF-8（一种可变宽度编码）的形式。
字符串中的每个 ASCII 字符都会存储在单字节中，而其他字符会占用多字节。

以下代码创建 String 值和 &str 值
```rust
let noodles = "noodles".to_string();
let oodles = &noodles[1..]; // 借用
let poodles = "ಠ_ಠ";  // 预分配的只读内存
```

![52928227932f6731e07ebb24f595d9b8.png](../_resources/52928227932f6731e07ebb24f595d9b8.png)


可以将 String 视为 `Vec<u8>`，它可以保证包含格式良好的 UTF-8，实际上，String 就是这样实现的。

&str（发音为 /stɜːr/ 或 string slice）是对别人拥有的一系列UTF-8 文本的引用，即它“借用”了这个文本。

字符串字面量是指预分配文本的 &str，它通常与程序的机器码一起存储在只读内存区。


String 或 &str 的 .len() 方法会返回其长度。这个长度以 字节 而不是字符为单位

```rust
assert_eq!("ಠ_ಠ".len(), 7);
assert_eq!("ಠ_ಠ".chars().count(), 3);
```

不能修改 &str

`&mut str` 类型确实存在，但它==没什么用==，因为对 UTF-8 的几乎所有操作都会更改其字节总长度，但切片不能重新分配其引用目标的缓冲区。

事实上，&mut str 上唯一可用的操作是make_ascii_uppercase 和 make_ascii_lowercase，根据定义，它们会就地修改文本并且只影响单字节字符。

### 3.7.4 String


&str 非常像 &[T]，是一个指向某些数据的胖指针。而 String 则类似于 `Vec<T>`

`Vec<T>` 与 String 对比

|-|`Vec<T>`|String|
|--|--|--|
|自动释放缓冲区|T|T|
|可增长|T|T|
|类型关联函数 ::new(), ::with_capacity()|T|T|
|.reverse(), .capacity()|T|T|
|.push(), .pop()|T|T|
|范围语法 `v[start..stop]`|返回 &[T]|返回&str|
|自动转换|`Vec<T>`到 &[T]|&String 到 &str|
|继承的方法|来自 &[T]|来自 &str|


与 Vec 一样，每个 String 都在堆上分配了自己的缓冲区，不会与任何其他 String 共享。
当 String 变量超出作用域时，缓冲区将自动释放，除非这个 String 已经被移动。

创建 String 的方法
- .to_string()
  将 &str 转为 String。这会复制此字符串
- .to_owned()
  和 to_string() 一样。 但这种 方法名 也适用于另外一些类型，祥见ch13
- format!()
  工作方式和 println!() 类似，但它返回一个 新的String，而不是写入到标准输出，并且不会自动添加 换行符
  ```rust
  assert_eq!(format!("{}°{:02}′{:02}″N", 24, 5, 23),
    "24°05′23″N".to_string());
  ```
- 字符串的数组，切片，向量 都有2个方法( .concat(), .join(sep)), 从 许多字符串中形成一个新的字符串。


有时要选择是使用 &str 类型还是使用 String 类型。
第 5 章会详细讨论这个问题。
这里仅指出一点：&str 可以引用任意字符串的任意切片，无论它是字符串字面量（存储在可执行文件中）还是String（在运行期分配和释放）。
这意味着如果希望允许调用者传递任何一种字符串，那么 &str 更适合作为函数参数。



### 使用字符串

`==, !=, <, <=, >, >=`

还有其他方法，可以在 str(原始类型) 和 std::str模块 的 在线文档中找到 (或直接看 ch17)

```rust
assert!("peanut".contains("nut"));
assert_eq!("ಠ_ಠ".replace("ಠ", "■"), "■_■");
assert_eq!(" clean\n".trim(), "clean");
for word in "veni, vidi, vici".split(", ") {
  assert!(word.starts_with("v"));
}
```

考虑到 Unicode 的性质，简单的逐字符比较并不总能给出预期的答案。
例如，Rust 字符串 "th\u{e9}" 和 "the\u{301}" 都是 thé（在法语中是“茶”的意思）的有效 Unicode 表示。
Unicode规定它们应该以相同的方式显示和处理，但 Rust 会将它们视为两个完全不同的字符串。
类似地，Rust 的排序运算符（如 <）也使用基于字符码点值的简单字典顺序。


### 其他类似字符串的类型
Rust 保证字符串是有效的 UTF-8。
有时程序确实需要处理并非有效Unicode 的字符串。

Rust 的解决方案是为这些情况提供一些类似字符串的类型。
- 对于Unicode文本，坚持使用 String， &str
- 当使用文件名时，使用 std::path::PathBuf, &Path
- 处理根本不是UTF-8编码的 二进制数据时，使用 `Vec<u8>`, &[u8]
- 使用OS提供的 环境变量名和命令行参数时， OsString, &OsStr
- 和 使用null结尾字符串 的C语言库 互操作时，使用 std::ffi::CString, &CStr



## 3.8 类型别名

`type Bytes = Vec<u8>;`
`fn decode(data: &Bytes) {}`





# ch4 所有权与移动

谈及内存管理，我们希望编程语言能具备两个特点
- 希望内存能在我们选定的时机及时释放，这使我们能控制程序的内存消耗；
- 在对象被释放后，我们绝不希望继续使用指向它的指针，这是未定义行为，会导致崩溃和安全漏洞。


施加这些限制的最终目的是在混沌中建立足够的秩序，以便让 Rust 的编译期检查器有能力验证程序中是否存在内存安全错误：悬空指针、重复释放、使用未初始化的内存等。

这些规则同样构成了 Rust 支持安全并发编程的基础。
使用 Rust 精心设计的线程原语，那些确保代码在正确使用内存的规则，同样可以用于证明代码中不存在数据竞争。

多线程代码中的固有不确定性被隔离到了那些专门设计来处理它们的线程特性（比如互斥锁、消息通道、原子值等）上，而不必出现在普通的内存引用中。


## 所有权

如果你读过大量 C 或 C++ 代码，可能遇到过这样的注释，即某个类的实例拥有它指向的某个其他对象。
通常，拥有对象意味着可以决定何时释放此对象：当销毁拥有者时，它拥有的对象也会随之销毁。

。。？ 难道rust不能？ 只有一层？

假如有如下 C++ 代码：
`std::string s = "frayed knot";`

![6cf054c1b25ac435b2cb9b1d5f63320c.png](../_resources/6cf054c1b25ac435b2cb9b1d5f63320c.png)


std::string 对象本身总是正好有 3 个机器字长，包括
- 指向分配在堆上的缓冲区的指针、
- 缓冲区的总容量（在不得不为字符串分配更大的缓冲区之前，文本可以增长到多大），
- 以及当前持有的文本的长度。

当程序销毁字符串时，字符串的析构函数会释放缓冲区。

以前，一些 C++ 库会在多个 std::string值之间共享同一个缓冲区，通过引用计数来决定何时释放此缓冲区。
但较新版本的 C++ 规范有效地杜绝了这种表示法，所有现代 C++ 库使用的都是这里展示的方法。

你可以创建一个指向std::string 的缓冲区中的字符的指针，但是当字符串被销毁时，你也必须让你的指针失效，并且要确保不再使用它。
拥有者决定被拥有者的生命周期，其他所有人都必须尊重其决定。

这里使用了 std::string 作为 C++ 中所有权的示例：它只是标准库通常遵循的规约，尽管 C++ 鼓励人们都遵循类似的做法，但说到底，如何设计自己的类型还是要由你自己决定。

在 Rust 中，所有权这个概念内置于语言本身，并通过编译期检查强制执行。
每个值都有决定其生命周期的唯一拥有者。当拥有者被释放时，它拥有的值也会同时被释放

==变量==拥有自己的值，当控制流离开声明变量的块时，变量就会被丢弃，因此它的值也会一起被丢弃。

Box 类型是所有权的另一个例子。`Box<T>` 是指向存储在堆上的 T 类型值的指针。
可以调用 Box::new(v) 分配一些堆空间，将值 v 移入其中，并返回一个指向该堆空间的 Box。
因为 Box 拥有它所指向的空间，所以当丢弃 Box 时，也会释放此空间。


```rust
struct Person { name: String, birth: i32 }
let mut composers = Vec::new();
composers.push(Person { name: "Palestrina".to_string(), birth: 1525 });
composers.push(Person { name: "Dowland".to_string(), birth: 1563 });
composers.push(Person { name: "Lully".to_string(), birth: 1632 });
for composer in &composers {
  println!("{}, born {}", composer.name, composer.birth);
}
```


![1ef7a0b01e5699f9085b1ebe6ccd3967.png](../_resources/1ef7a0b01e5699f9085b1ebe6ccd3967.png)


拥有者及其拥有的那些值形成了一棵树：值的拥有者是值的父节点，值拥有的值是值的子节点。
每棵树的总根都是一个变量，当该变量超出作用域时，整棵树都将随之消失

Rust 的单一拥有者规则将禁止任何可能让它们排列得比树结构更复杂的可能性。
Rust 程序中的每一个值都是某棵树的成员，树根是某个变量。

在 Rust 中丢弃一个值的方式就是从所有权树中移除它：
- 或者离开变量的作用域，
- 或者从向量中删除一个元素，
- 或者执行其他类似的操作。


Rust 从几个方面扩展了这种简单的思想。
- 可以将值从一个拥有者==转移==给另一个拥有者。这允许你构建、重新排列和拆除树形结构。
- 像整数、浮点数和字符这样的非常简单的类型，不受所有权规则的约束。这些称为 ==Copy 类型==。
- 标准库提供了==引用计数指针类型== Rc 和 Arc，它们允许值在某些限制下有多个拥有者。
- 可以对值进行“==借用==”（borrow），以获得值的引用。这种引用是非拥有型指针，有着受限的生命周期。



## 4.2 移动

对大多数类型来说，像
- 为变量赋值、
- 将其传给函数
- 从函数返回
这样的操作都不会复制值，而是会移动值


源会把值的所有权转移给目标并变回未初始化状态，改由目标变量来控制值的生命周期。
Rust 程序会以每次只移动一个值的方式建立和拆除复杂的结构。


```python
s = ['udon', 'ramen', 'soba']
t = s
u = s
```
Python 已经将指针从 s 复制到 t 和 u，并将此列表的引用计数更新为 3。
Python 中的赋值开销极低，但因为它创建了对对象的新引用，所以必须维护引用计数才能知道何时可以释放该值。


```C++
using namespace std;
vector<string> s = { "udon", "ramen", "soba" };
vector<string> t = s;
vector<string> u = s;
```
在 C++ 中，把std::vector 赋值给其他元素会生成一个向量的副本，std::string 的行为也类似。
所以当程序执行到这段代码的末尾时，它实际上已经分配了 3 个向量和 9 个字符串



```rust
let s = vec!["udon".to_string(), "ramen".to_string(),
"soba".to_string()];
let t = s;
let u = s;
```

==与 C 和 C++ 一样==，Rust 会将==纯字符串字面量==（如 "udon"）==放在只读内存==中，因此为了与 C++ 示例和 Python 示例进行更清晰的比较，此处调用了 ==to_string 以获取堆上分配的 String 值==。

在执行了 s 的初始化之后，由于 Rust 和 C++ 对向量和字符串使用了类似的表示形式，因此情况看起来就和 C++ 中一样

在 Rust 中，大多数类型的赋值会将值从源转移给目标，而源会回到未初始化状态。因此在初始化 t 之后，==数据归属于t，同时s变成未初始化状态==

初始化语句 let t = s; 将向量的 3 个标头字段从 s 转移给了 t，现在 t 拥有此向量。

执行初始化语句 let u = s; 时会发生什么呢？这会将尚未初始化的值 s 赋给 u。Rust 明智地禁止使用未初始化的值，因此编译器会拒绝此代码并报告如下错误
```text
error: use of moved value: `s`
 |
7 | let s = vec!["udon".to_string(), "ramen".to_string(),
"soba".to_string()];
 | - move occurs because `s` has type `Vec<String>`,
 | which does not implement the `Copy` trait
8 | let t = s;
 | - value moved here
9 | let u = s;
 | ^ value used here after move
```

与 Python 一样，赋值操作开销极低：程序只需将向量的三字标头从一个位置移到另一个位置即可。
与 C++ 一样，所有权始终是明确的：程序不需要引用计数或垃圾回收就能知道何时释放向量元素和字符串内容。

代价是如果需要同时访问它们，就必须显式地要求复制

```rust
let s = vec!["udon".to_string(), "ramen".to_string(),
"soba".to_string()];
let t = s.clone();
let u = s.clone();
```

还可以使用 Rust 的引用计数来实现与 Python 类似的操作，见4.4


---
4.2.1 更多移动类操作

将一个值转移给已初始化的变量，那么 Rust 就会丢弃该变量的先前值

```rust
let mut s = "Govinda".to_string();
s = "Siddhartha".to_string(); // 在这里丢弃了值"Govinda"
```

```rust
let mut s = "Govinda".to_string();
let t = s;
s = "Siddhartha".to_string(); // 这里什么也没有丢弃
```
t 从 s 接手了原始字符串的所有权，所以当给 s 赋值时，它是未初始化状态。这种情况下不会丢弃任何字符串


```rust
struct Person { name: String, birth: i32 }
let mut composers = Vec::new();
composers.push(Person { name: "Palestrina".to_string(), birth: 1525 });
```
这段代码展示了除初始化和赋值之外发生 移动 的几个地方。
- 从函数返回值
  Vec::new()
  所有权从 Vec::new 转移给了变量composers
- 构造新值
  name 字段是用 to_string 的返回值初始化的。
  该结构体拥有这个字符串的所有权。
- 将值传递给函数
  整个 Person 结构体（不是指向它的指针）被传给了向量的push 方法，此方法会将该结构体移动到向量的末尾。
  向量接管了Person 的所有权，因此也间接接手了 name 这个 String 的所有权。

1. 移动的是值本身，而不是 这些值拥有的 堆存储
2. 编译器在生成代码时，可以看穿这一切。在实践中，机器码通常会将值直接存储在它该在的位置


---
4.2.2 移动与控制流

一般性原则是，如果一个变量的值有可能已经移走，并且从那以后尚未明确赋予其新值，那么它就可以被看作是未初始化状态。

如果一个变量在执行了 if 表达式中的条件后仍然有值，那么就可以在这两个分支中使用它：
```rust
let x = vec![10, 20, 30];
if c {
  f(x); // ……可以在这里移动x
} else {
  g(x); // ……也可以在这里移动x
}
h(x); // 错误：只要任何一条路径用过它，x在这里就是未初始化状态
```

出于类似的原因，禁止在循环中进行变量移动
```rust
let x = vec![10, 20, 30];
while f() {
  g(x); // 错误：x已经在第一次迭代中移动出去了，在第二次迭代中，它成了未初始化状态
}
```

除非在下一次迭代中明确赋予 x 一个新值，否则就会出错。
```rust
let mut x = vec![10, 20, 30];
while f() {
  g(x); // 从x移动出去了
  x = h(); // 赋予x一个新值
}
e(x);
```

---
4.2.3 移动与索引内容

移动会令其来源变成未初始化状态，因为目标将获得该值的所有权。

但并非值的每种拥有者都能变成未初始化状态
```rust
// 构建一个由字符串"101"、"102"……"105"组成的向量
let mut v = Vec::new();
for i in 101 .. 106 {
  v.push(i.to_string());
}
// 从向量中随机抽取元素
let third = v[2]; // 错误：不能移动到Vec索引结构之外 // 注意这里写的是 v[2] 而非 &v[2]
let fifth = v[4]; // 这里也一样
```

Rust 需要以某种方式记住向量的第三个元素和第五个元素是未初始化状态，并要跟踪该信息直到向量被丢弃。
通常的解决方案是，让每个向量都携带额外的信息来指示哪些元素是活动的，哪些元素是未初始化的。这显然不是系统编程语言应该做的。
向量应该只是向量，不应该携带额外的信息或状态。
事实上，Rust 会拒绝前面的代码并报告如下错误：

```text
error: cannot move out of index of `Vec<String>`
   |
14 | let third = v[2];
   |             ^^^^
   |             |
   |             move occurs because value has type `String`,
   |             which does not implement the `Copy` trait
   |             help: consider borrowing here: `&v[2]`
```

移动第五个元素时 Rust 也会报告类似的错误。在这条错误消息中，Rust 还建议使用引用，因为你==可能只是想访问该元素==而不是移动它，这通常确实是你想要做的

### 从vec移出元素
但是，如果真想将一个元素移出向量该怎么办呢？
以下是 3 种可能的方法：

每种方法都能将一个元素移出向量，但仍会让向量处于完全填充状态，只是向量可能会变小。

```rust
// 构建一个由字符串"101"、"102"……"105"组成的向量
let mut v = Vec::new();
for i in 101 .. 106 {
  v.push(i.to_string());
}
// 方法一：从向量的末尾弹出一个值：
let fifth = v.pop().expect("vector empty!");
assert_eq!(fifth, "105");
// 方法二：将向量中指定索引处的值与最后一个值互换，并把前者移动出来：
let second = v.swap_remove(1);
assert_eq!(second, "102");
// 方法三：把要取出的值和另一个值互换：
let third = std::mem::replace(&mut v[2], "substitute".to_string());
assert_eq!(third, "103");
// 看看向量中还剩下什么
assert_eq!(v, vec!["101", "104", "substitute"]);
```

像 Vec 这样的集合类型通常也会提供在循环中消耗所有元素的方法

```rust
for mut s in v {
  s.push('!');
  println!("{}", s);
}
```
当我们将向量直接传给循环（如 for ... in v）时，会将向量从 v 中移动出去，让 v 变成未初始化状态。
for 循环的内部机制会获取向量的所有权并将其分解为元素。


如果需要从拥有者中移出一个编译器无法跟踪的值，那么可以考虑将拥有者的类型更改为能动态跟踪自己是否有值的类型。

```rust
struct Person { name: Option<String>, birth: i32 }
let mut composers = Vec::new();
composers.push(Person { name: Some("Palestrina".to_string()), birth: 1525 });
```

不能像下面这样做：
`let first_name = composers[0].name;`

这只会引发与前面一样的“无法移动到索引结构之外”错误。
但是因为已将 name 字段的类型从 String 改成了 `Option<String>`，所以这意味着 None 也是该字段要保存的合法值。因此，可以像下面这样做：

```rust
let first_name = std::mem::replace(&mut composers[0].name, None);
assert_eq!(first_name, Some("Palestrina".to_string()));
assert_eq!(composers[0].name, None);
```

replace 调用会移出 composers[0].name 的值，将 None 留在原处，并将原始值的所有权转移给其调用者。
事实上，这种使用Option 的方式非常普遍，所以该类型专门为此提供了一个 take 方法，以便更清晰地写出上述操作，如下所示：
`let first_name = composers[0].name.take();`

。。就是 使用 None 替换了 name，并把 name的前值返回，赋给 first_name。
。。take 可以接受其他参数吗？


## 4.3 Copy类型，关于移动的例外情况
本章所展示的值移动示例都涉及向量、字符串和其他可能占用大量内存且复制成本高昂的类型。
移动能让这些类型的所有权清晰且赋值开销极低。

但对于像整数或字符这样的简单类型，如此谨小慎微的处理方式确实没什么必要。

大多数类型会被移动，现在该谈谈例外情况了，即那些被 Rust 指定成 Copy 类型的类型。
对 Copy 类型的值进行赋值会复制这个值，而不会移动它。

标准的 Copy 类型包括所有机器整数类型和浮点数类型、char 类型和 bool 类型，以及某些其他类型。
Copy 类型的元组或固定大小的数组本身也是 Copy 类型。

只有那些可以通过==简单地复制位来复制其值==的类型才能作为 Copy 类型。


根据经验，任何在==丢弃值时需要做一些特殊操作的类型都不能是Copy== 类型：
Vec 需要释放自身元素、File 需要关闭自身文件句柄、MutexGuard 需要解锁自身互斥锁

自定义类型，默认情况下，struct 类型和 enum 类型==不是== Copy 类型：

### 创建Copy类型

如果此结构体的==所有字段本身都是 Copy== 类型，那么也可以通过将属性 `#[derive(Copy, Clone)]` 放置在此定义之上来创建 Copy 类型

```rust
#[derive(Copy, Clone)]
struct Label { number: u32 }
```

在 Rust 中，每次==移动==都是==字节级的一对一浅拷贝==，并==让源变成未初始化状态==。
==复制==也是如此，但会==保留源的初始化状态==。

ch11 讲解特型， ch13 讲解 Copy 和 Clone



## 4.4 Rc,Arc：共享所有权

你会希望某个值只要存续到每个人都用完它就好。

Rust 提供了引用计数指针类型 ==Rc 和 Arc [Arc 是原子引用计数(atomic reference count==) 的缩写 ]。


Rc 类型和 Arc 类型非常相似，它们之间唯一的区别是 Arc 可以安全地在线程之间直接共享，而普通 Rc 会使用更快的非线程安全代码来更新其引用计数。

```rust
use std::rc::Rc;
// Rust能推断出所有这些类型，这里写出它们只是为了讲解时清晰
let s: Rc<String> = Rc::new("shirataki".to_string());
let t: Rc<String> = s.clone();
let u: Rc<String> = s.clone();
```

对于任意类型 T，`Rc<T>` 值是指向附带引用计数的在堆上分配的 T 型指针。
克隆一个 `Rc<T>` 值并==不会复制 T==，相反，它只会==创建==另一个指向它的==指针==并==递增引用计数==。
当丢弃最后一个现有 Rc 时，Rust 也会丢弃 String。

。。shared_ptr

可以==直接==在 `Rc<String>` 上使用 String 的任何常用方法

Rc 指针拥有的值是==不可变==的

Rust 的==内存和线程安全==保证的基石是：==确保不会有任何值是既共享又可变的==。

如果有两个引用计数的值是相互指向的，那么其中一个值就会让另一个值的引用计数保持在 0 以上，因此这些值将永远没机会释放

以这种方式在 Rust 中造成值的泄漏也是有可能的，但这种情况非常少见。
只要不在某个时刻让旧值指向新值，就无法建立循环。

这显然要求旧值是可变的。由于 Rc 指针会保证其引用目标不可变，因此通常不可能建立这种循环引用。

但是，Rust 确实提供了创建其他==不可变值中的可变部分==的方法，这称为==内部可变性==，9.11 节会详细介绍。
如果将这些技术与 Rc 指针结合使用，则确实可以建立循环并造成内存泄漏


有时可以通过对某些链接使用==弱引用指针== std::rc::Weak 来==避免==建立 Rc 指针循环。


移动和引用计数指针是缓解所有权树严格性问题的两种途径。
在第 5 章中，我们将研究第三种途径：==借用对值的引用==。
一旦你熟悉了==所有权和借用==这两个概念，就已经翻越了 Rust 学习曲线中==最陡峭==的部分，并为发挥 Rust 的独特优势做好了准备


# ch05 引用

迄今为止，我们看到的所有指针类型（无论是简单的 `Box<T>` 堆指针，还是 String 值和 Vec 值内部的指针）都是==拥有型指针==

Rust 还有一种名为引用（reference）的非拥有型指针，这种指针对引用目标的生命周期毫无影响。
事实上，影响是反过来的：引用的生命周期绝不能超出其引用目标。

为了强调这一点，Rust 把创建对某个值的引用的操作称为借用（borrow）那个值：凡是借用，终须归还。



## 5.1 对值的引用

引用能让你在==不影响其所有权的情况下访问值==。

引用分为以下两种。
- 共享引用
  允许你==读取==但==不能修改==其引用目标。
  表达式 &e 会产生对 e 值的共享引用
  如果 e 的类型为 T，那么 &e 的==类型==就是 &T，读作“ref T”。
  共享引用是 Copy 类型。
- 可变引用
  允许你==读取和修改==值
  一旦一个值拥有了可变引用，就无法再对该值创建其他任何种类的引用了。
  表达式 `&mut e` 会产生一个对 e 值的可变引用，可以将其==类型==写成 `&mut T`，读作“ref mute T”。
  可变引用不是 Copy 类型。


==只要存在对一个值的共享引用，即使是它的拥有者也不能修改它，该值会被锁定==

==如果有某个值的可变引用，那么它就会独占对该值的访问权，在可变引用消失之前，即使拥有者也根本无法使用该值==


```rust
type Table = HashMap<String, Vec<String>>;

fn show(table: &Table) {  // 共享引用，不会获得 table 的所有权，所以 调用 show 方法后，外面还可以继续使用 table
}

fn sort_works(table: &mut Table) {  // 可变引用，sort会修改数据，所以 可变引用
}
```


## 5.2 使用引用

前面的示例展示了引用的一个非常典型的用途：允许函数在不获取所有权的情况下访问或操纵某个结构。

但引用比这要灵活得多

在 C++ 中，引用是通过类型转换隐式创建的，并且是隐式解引用的
在 Rust 中，引用是通过 `& 运算符`显式创建的，同时要用 `* 运算符显式解引用`
要创建可变引用，可以使用 `&mut 运算符`

由于引用在 Rust 中随处可见，因此 `. 运算符`就会按需对其左操作数==隐式解引用==

```rust
struct Anime { name: &'static str, bechdel_pass: bool }
let aria = Anime { name: "Aria: The Animation", bechdel_pass: true};

let anime_ref = &aria;
assert_eq!(anime_ref.name, "Aria: The Animation");

// 与上一句等效，但把解引用过程显式地写了出来
assert_eq!((*anime_ref).name, "Aria: The Animation");
```

```rust
let mut v = vec![1973, 1968];
v.sort(); // 隐式借用对v的可变引用
(&mut v).sort(); // 等效，但是更烦琐
```


---
5.2.2 对引用变量赋值

把引用赋值给某个引用变量会让该变量指向新的地方

```rust
let x = 10;
let y = 20;
let mut r = &x;
if b { r = &y; }
assert!(*r == 10 || *r == 20);
```
引用 r 最初指向 x。但如果 b 为 true，则代码会把它改为指向y

在 C++ 中对引用赋值会将新值存储在其引用目标中而非指向新值。
C++ 的引用一旦完成初始化，就无法再指向别处了。


---
5.2.3 对引用进行引用

Rust 允许对引用进行引用

```rust
struct Point { x: i32, y: i32 }
let point = Point { x: 1000, y: 729 };
let r: &Point = &point;
let rr: &&Point = &r;
let rrr: &&&Point = &rr;
```
类型可以省略，只是为了更清晰


---
5.2.4 比较引用

像 `. 运算符`一样，Rust 的`比较运算符`也能“看穿”==任意数量==的引用

```rust
let x = 10;
let y = 10;
let rx = &x;
let ry = &y;
let rrx = &rx;
let rry = &ry;
assert!(rrx <= rry);
assert!(rrx == rry);
```

。。之前没有说 .运算符 是 任意数量的 引用。


如果你真想知道两个引用是否指向==同一块内存==，可以使用 std::ptr::eq，它会将两者作为==地址==进行比较
```rust
assert!(rx == ry); // 它们引用的目标值相等
assert!(!std::ptr::eq(rx, ry)); // 但所占据的地址（自身的值）不同
```

比较运算符的操作数（包括引用型操作数）必须具有完全相同的类型

```rust
assert!(rx == rrx); // 错误：`&i32`与`&&i32`的类型不匹配
assert!(rx == *rrx); // 这样没问题
```


---
5.2.5 引用永不为空

在 Rust 中，如果需要用一个值来表示对某个“可能不存在”事物的引用，请使用类型 Option<&T>


---
### 5.2.6 借用任意表达式结果值的引用

C 和 C++ 只允许将 & 运算符应用于某些特定种类的表达式，
而 Rust允许借用任意种类的表达式结果值的引用：

。。？应用到表达式？

```rust
fn factorial(n: usize) -> usize {
  (1..n+1).product()
}
let r = &factorial(6);
// 数学运算符可以“看穿”一层引用
assert_eq!(r + &1009, 1729);
```
Rust 会创建一个匿名变量来保存此表达式的值，并让该引用指向它

这个匿名变量的生命周期取决于你对引用做了什么
- let语句中，如果立即将 引用赋值给某个变量(或使其成为立即被赋值的某个结构体或数组的一部分)，那么rust就让匿名变量存在于 let 初始化此变量期间。 上面的示例中，rust 对 r 的引用目标 这样做。
- 否则，匿名变量会存续到 所属封闭语句块的末尾。上面的示例中， 为 保存 1009 而创建的 匿名变量 只存续到 assert_eq! 语句的末尾。

。。看起来，匿名对象 都是 存活到 语句的末尾啊。


---
5.2.7 对切片和特型对象的引用

迄今为止，我们展示的引用全都是简单地址。
但是，Rust 还包括两种 胖指针，即携带某个值地址的双字值，以及要正确使用该值所需的某些额外信息。

对切片的引用就是一个胖指针，携带着此切片的起始地址及其长度

Rust 的另一种胖指针是 特性对象，即对实现了指定特型的值的引用。
特型对象会携带一个值的地址和指向适用于该值的特型实现的指针，以便调用特型的方法



## 5.3 引用安全

从最简单的案例开始，展示 Rust 如何确保在单个函数体内正确使用引用
然后，如何在函数之间传递引用并将它们存储到数据结构中。这需要为函数和数据类型提供 生命周期参数
最后介绍 Rust 提供的一些简写形式，以简化常见的使用模式


---
5.3.1 借用局部变量

非常浅显的案例。你不能 借用 对 局部变量的 引用 并将其 移出 变量的 作用域

```rust
{
  let r;
  {
    let x = 1;
    r = &x;
  }
  assert_eq!(*r, 1); // 错误：试图读取`x`所占用的内存
}
```
会编译失败

x,r 各自有生命周期，在 执行 r=&x 后， 显然 r 的生命周期不能超过 x，不然会 悬垂指针。

如果将 引用存储在 变量 r 中，则引用类型 必须在 变量 r 从初始化 到 最后一次使用的 整个生命周期内都可以访问。


---
5.3.2 将引用作为函数参数

当我们传递对函数的引用时，rust 如何确保 函数能安全地使用它？

```rust
// 这段代码有一系列问题，无法编译
static mut STASH: &i32;
fn f(p: &i32) { STASH = p; }
```

在 Rust 中，全局变量的等价物称为 静态变量，它是在程序启动时就会被创建并一直存续到程序终止时的值。
静态变量 ch8

上面的代码违反了
- 每个静态变量都必须初始化
- 可变静态变量 本质上 不是线程安全的，即使单线程中，也可能成为一些另类 可重入性问题的牺牲品。所以，你只能在 unsafe 中访问 可变静态变量。


```rust
static mut STASH: &i32 = &128;
fn f(p: &i32) { // 仍然不够理想
  unsafe {
    STASH = p;
  }
}
```
上面的 f 签名实际上 是 下面的简写
`fn f<'a>(p: &'a i32) { ... }`

生命周期 'a（读作“tick A”）是 f 的 生命周期参数。

上面的代码编译失败。 因为 STASH是 'static 生命周期， 而 p 是 'a

```text
error: explicit lifetime required in the type of `p`
  |
5 |       STASH = p;
  |
                  ^ lifetime `'static` required
```


下面的代码可以
```rust
static mut STASH: &i32 = &10;
fn f(p: &'static i32) {
  unsafe {
    STASH = p;
  }
}
```

```rust
static WORTH_POINTING_AT: i32 = 1000;
f(&WORTH_POINTING_AT);
```


我们无法编写在全局变量中潜藏一个引用 却不在函数签名中明示该意图的函数。


---

5.3.3 把引用传给函数

ok
```rust
// 这个函数可以简写为fn g(p: &i32)，但这里还是把它的生命周期写出来了
fn g<'a>(p: &'a i32) { ... }
let x = 10;
g(&x);
```

编译错误
```rust
fn f(p: &'static i32) { ... }
let x = 10;
f(&x);
```

---
5.3.4 返回引用

函数通常会接收某个数据结构的引用，然后返回对该结构的某个部分的引用

```rust
// v应该至少有一个元素
fn smallest(v: &[i32]) -> &i32 {
  let mut s = &v[0];
  for r in &v[1..] {
    if *r < *s { s = r; }
  }
  s
}
```

```rust
let s;
{
  let parabola = [9, 4, 1, 0, 1, 4, 9];
  s = smallest(&parabola);
}
assert_eq!(*s, 0); // 错误：指向了已被丢弃的数组的元素
```

ok
```rust
{
  let parabola = [9, 4, 1, 0, 1, 4, 9];
  let s = smallest(&parabola);
  assert_eq!(*s, 0); // 很好：parabola仍然“活着”
}
```


---

### 5.3.5 包含引用的结构体

Rust 对引用的安全约束不会因为我们将引用“藏”在结构体中而神奇地消失

```rust
// 这无法编译
struct S {
  r: &i32
}

let s;
{
  let x = 10;
  s = S { r: &x };
}
assert_eq!(*s.r, 10); // 错误：从已被丢弃的`x`中读取
```

每当一个引用类型出现在另一个类型的定义中时，==必须写出它的生命周期==
```rust
struct S {
  r: &'static i32
}
```
这表示 r 只能引用贯穿程序整个生命周期的 i32 值，这种限制太严格了。

还有一种方法是给类型指定一个生命周期参数 'a 并将其用在 r 上
```rust
struct S<'a> {
  r: &'a i32
}
```
现在 S 类型有了一个生命周期
你存储在 r 中的任何引用的生命周期最好都涵盖 'a，并且'a 必须比存储在 S 中的任何内容的生命周期都要长。

表达式 S { r: &x } 创建了一个新的 S 值，其生命周期为 'a。
==当你将 &x 存储在 r 字段中时，就将 'a 完全限制在了 x 的生命周期内部==


将 具有生命周期参数的 类型 放置在 其他类型 中时也必须 指定 生命周期
```rust
struct D<'a> {
  s: S<'a>
}
```
为 D 提供生命周期参数并将其传给 S




假设我们有一个解析函数，它会接受一个字节切片并返回一个存有解析结果的结构
`fn parse_record<'i>(input: &'i [u8]) -> Record<'i> { ... }`

不用看 Record 类型的定义就可以知道，如果从 parse_record 接收到 Record，那么==它包含的任何引用就必然指向我们传入的输入缓冲区==，而不是其他地方（'static 静态值除外）。

事实上，Rust 要求包含引用的类型都要接受显式生命周期参数就是为了明示这种内部行为。
其实 Rust 原本可以简单地为结构体中的每个引用创建一个不同的生命周期，从而省去把它们写出来的麻烦。
实际上，Rust 的早期版本就是这么做的，但开发人员发现这样会令人困惑：了解“某个值是从另一个值中借用出来的”这一点很有帮助，特别是在处理错误时。

Rust 中的每个类型都有生命周期，包括 i32 和 String。
它们大多数是 'static 的，这意味着这些类型的值可以一直存续下去，例如，`Vec<i32>` 是自包含的，在任何特定变量超出作用域之前都不需要丢弃它。
但是像`Vec<&'a i32>` 这样的类型，其生命周期就必须被 'a 涵盖，也就是说必须在引用目标仍然存续的情况下丢弃它。

---

### 5.3.6 不同的生命周期参数

```rust
struct S<'a> {
  x: &'a i32,
  y: &'a i32
}


let x = 10;
let r;
{
  let y = 20;
  {
    let s = S { x: &x, y: &y };
    r = s.x;
  }
}
println!("{}", r);
```

Rust 会报错说 y 的存活时间不够长

- S 的两个字段是具有相同生命周期 'a 的引用，因此 Rust 必须找到一个同时适合 s.x 和 s.y 的生命周期。
- 赋值 r = s.x，这就要求 'a 涵盖 r 的生命周期。
- 用 &y 初始化 s.y，这就要求 'a 不能长于 y 的生命周期。

这些约束是不可能满足的：没有哪个生命周期比 y 短但比 r 长。


```rust
struct S<'a, 'b> {
  x: &'a i32,
  y: &'b i32
}
```

'a 可以用 r 的生命周期，
而 'b 可以用 s 的生命周期。（虽然 'b 也可以用 y 的生命周期，但 Rust 会尝试选择可行的最小生命周期。）

`fn f<'a, 'b>(r: &'a i32, s: &'b i32) -> &'a i32 { r } // 宽松多了`



---
5.3.7 省略生命周期参数

在最简单的情况下，你可能永远不需要为参数写出生命周期。
Rust 会为需要生命周期的每个地方分配不同的生命周期

```rust
struct S<'a, 'b> {
  x: &'a i32,y: &'b i32
}
fn sum_r_xy(r: &i32, s: S) -> i32 {
  r + s.x + s.y
}
```

`fn sum_r_xy<'a, 'b, 'c>(r: &'a i32, s: S<'b, 'c>) -> i32`


如果确实要返回引用或其他带有生命周期参数的类型，那么针对无歧义的情况，Rust 会尽量采用简单的设计。
如果函数的参数只有一个生命周期，那么 Rust 就会假设返回值具有同样的生命周期
```rust
fn first_third(point: &[i32; 3]) -> (&i32, &i32) {
  (&point[0], &point[2])
}
```

`fn first_third<'a>(point: &'a [i32; 3]) -> (&'a i32, &'a i32)`


如果函数的参数有多个生命周期，那么就没有理由选择某一个生命周期作为返回值的生命周期，Rust 会要求你明确指定生命周期。

如果函数是某个类型的方法，并且具有==引用==类型的 ==self== 参数，那么 Rust 就会假定返回值的生命周期与 self 参数的生命周期相同

。有 非引用的 self 吗？  默认的 self 是怎么样的？


## 5.4 共享和可变

依然可能引入悬空指针

```rust
let v = vec![4, 8, 19, 27, 34, 10];
let r = &v;
let aside = v; // 把向量转移给aside
r[0]; // 错误：这里所用的`v`此刻是未初始化状态
```

对 aside 的赋值会移动向量、让 v 回到未初始化状态，并将 r 变为悬空指针

当然，Rust 会捕获错误
```text
error: cannot move out of `v` because it is borrowed
    |
9   |       let r = &v;
    |                - borrow of `v` occurs here
10  |       let aside = v; // 把向量转移给`aside`
    |           ^^^^^ move out of `v` occurs here
```

在共享引用的整个生命周期中，它引用的目标会保持只读状态，即不能对引用目标赋值或将值移动到别处


---

使用切片的元素来扩展某个向量
```rust
fn extend(vec: &mut Vec<f64>, slice: &[f64]) {
  for elt in slice {
    vec.push(*elt);
  }
}
```

那么可以把向量追加到其自身吗
`extend(&mut wave, &wave);`

别忘了，在往向量中添加元素时，如果它的缓冲区已满，那么就必须分配一个具有更多空间的新缓冲区

vec修改了，但是 slice还是指向原来的位置，造成 悬垂引用

在许多语言中，在==指向集合的同时修改集合要加倍小心==。
在 C++ 中，std::vector 规范会告诫你“重新分配向量缓冲区会令指向序列中各个元素的所有引用、指针和迭代器失效”。
Java 对修改 java.util.Hashtable 对象也有类似的说法。


这类错误特别难以调试，因为它只会偶尔发生。
在测试中，向量可能总是恰好有足够的空间，缓冲区可能永远都不会重新分配，于是这个问题可能永远都没人发现

Rust 会在编译期报告调用 extend 有问题
```text
cannot borrow `wave` as immutable because it is also borrowed as mutable
```

我们既可以借用向量的可变引用，也可以借用其元素的共享引用，但这两种引用的生命周期不能重叠。


共享访问是只读访问

可变访问是独占访问。


![截图 2024-04-22 20-21-47.png](../_resources/截图%202024-04-22%2020-21-47.png)

在这两种情况下，指向引用目标的所有权路径在此引用的生命周期内都无法更改。
对于共享借用，这条路径是只读的；
对于可变借用，这条路径是完全不可访问的。所以程序无法做出任何会使该引用无效的操作。


```rust
let mut x = 10;
let r1 = &x;
let r2 = &x;  // 正确：允许多个共享借用
x += 10;  // 错误：不能赋值给`x`，因为它已被借出
let m = &mut x; // 错误：不能把`x`借入为可变引用，因为它涵盖在已借出的不可变引用的生命周期内
println!("{}, {}, {}", r1, r2, m); // 这些引用是在这里使用的，所以它们的生命周期至少要存续这么长

let mut y = 20;
let m1 = &mut y;
let m2 = &mut y; // 错误：不能多次借入为可变引用
let z = y;  // 错误：不能使用`y`，因为它涵盖在已借出的可变引用的生命周期内
println!("{}, {}, {}", m1, m2, z); // 在这里使用这些引用
```


可以从共享引用中重新借入共享引用
```rust
let mut w = (107, 109);
let r = &w;
let r0 = &r.0;  // 正确：把共享引用重新借入为共享引用
let m1 = &mut r.1;  // 错误：不能把共享引用重新借入为可变
println!("{}", r0);  // 在这里使用r0
```


可以从可变引用中重新借入可变引用
```rust
let mut v = (136, 139);
let m = &mut v;
let m0 = &mut m.0;  // 正确: 从可变引用中借入可变引用
*m0 = 137;
let r1 = &m.1;  // 正确: 从可变引用中借入共享引用，并且不能和m0重叠
v.1;  // 错误：禁止通过其他路径访问
println!("{}", r1); // 可以在这里使用r1
```


要为集合设计出“支持不受限制地在迭代期间修改”的能力是非常困难的，而且往往会导致无法简单高效地实现这些集合。

C++ 的 std::map 承诺插入新条目不会让指向此映射表中其他条目的指针失效，但做出这一承诺的代价是该标准无法提供像 Rust 的 BTreeMap 这样更高效的缓存设计方案，因为后者会在树的每个节点中存储多个条目。

。。？ 你 b tree 和我 rb tree 有什么关系？


两个经典 C++ 错误（无法处理自赋值和使用无效迭代器）本质上是同一种错误。
在这两种情况下，代码都以为自己正在修改一个值，同时在引用另一个值，但实际上两者是同一个值。

。。所以 operator= 的时候 必须先 判断 是否和 this 相同。


## 5.5 应对复杂对象关系


所有程序都由大量复杂关联的对象构成

![截图 2024-04-22 20-39-13.png](../_resources/截图%202024-04-22%2020-39-13.png)


---

Rust 令人着迷的地方之一就在于，其所有权模型就好像是在通向地狱的高速公路上铺设了一条减速带。
在 Rust 中创建循环引用（两个值，每个值都包含指向另一个值的引用）相当困难。
你必须使用智能指针类型（如 Rc）和内部可变性（目前为止本书还未涉及这个主题）。
Rust 更喜欢让指针、所有权和数据流单向通过系统




![截图 2024-04-22 20-39-34.png](../_resources/截图%202024-04-22%2020-39-34.png)


在阅读本章后，你可能会很自然地想要立即编写代码并创建出大量的对象，所有对象之间使用 Rc 智能指针关联起来，最终呈现你熟悉的所有面向对象反模式。
但此刻这还行不通。Rust 的所有权模型会不断给你制造麻烦。解决之道是进行一些前期设计并构建出更好的程序。


Rust 就是要==把你理解程序的痛苦从将来移到现在==。
它确实做到了：Rust 不仅会迫使你理解为什么自己的程序是线程安全的，甚至可能还需要你做一些高级架构设计。





# ch06 表达式


C 语言中 表达式和语句之间有明显的区别，表达式看起来是这样的
`5 * (fahr - 32) / 9`
语句看起来是这样的
```C
for (; begin != end; ++begin) {
  if (*begin == target)
    break;
}
```
表达式有值，而语句没有。


Rust 是所谓的 表达式语言 。这意味着它遵循更古老的传统，可以追溯到 Lisp，在 Lisp 中，表达式能完成所有工作。

C中，if和 switch 是语句，不能生成值，不能在表达式 中间使用。
Rust 中，if 和 match 可以生成值

```rust
pixels[r * bounds.0 + c] =
  match escapes(Complex { re: point.0, im: point.1 }, 255) {
    None => 0,
    Some(count) => 255 - count as u8
  };

////////////////
let status =
  if cpu.temperature <= MAX_TEMP {
    HttpStatus::Ok
  } else {
    HttpStatus::ServerError // 服务程序出错了
  };

////////////////////

println!("Inside the vat, you see {}.",
  match vat.contents {
    Some(brain) => brain.desc(),
    None => "nothing of interest"
  });
```

C 语言中，三元运算符是一个表达式级别的类似 if语句的东西。

这在 Rust 中是多余的：if 表达式足以处理这两种情况。


## 6.2 优先级和结合性

下面是 表达式语法，顺序是 优先级顺序，还有 相关特型

- 数组字面量
  [1, 2, 3]
- 数组重复表达式
  [0; 50]
- 元组
  (6, "aaa")
- 分组
  (2 + 2)
- 块
  { f(); g() }
- 控制流表达式
  if, while, loop, break, continue, return, for, while let, if let, match
  相关特型：std::iter::IntoIterator
- 宏调用
  println!("ok")
- 路径
  std::f64::consts::PI
- 结构体字面量
  Point { x: 0, y: 0}
- 元组字段访问
  pair.0
  Deref, DerefMut
- 结构体字段访问
  point.x
  Deref, DerefMut
- 方法调用
  point.translate(50, 50)
  Deref, DerefMut
- 函数调用
  stdin()
  Fn(Arg0, ...) -> T, FnMut(Arg0, ...) -> T, FnOnce(Arg0, ...) -> T
- 索引
  arr[0]
  Index, IndexMutDeref, DerefMut
- 错误检查
  create_dir("tmp")?
- 逻辑非/按位非
  !ok
  Not
- 取负
  -num
  Neg
- 解引用
  *ptr
  Deref, DerefMut
- 借用
  &val
- 类型转换
  x as u32
- 乘
  n * 2
  Mul
- 除
  n / 2
  Div
- 取余
  n % 2
  Rem
- 加
  n + 1
  Add
- 减
  n - 1
  Sub
- 左移
  `n << 2`
  Shl
- 右移
  `n >> 2`
  Shr
- 按位与
  n & 1
  BitAnd
- 按位异或
  n ^ 1
  BitXor
- 按位或
  n | 1
  BitOr
- 小于
  `n < 1`
  std::cmp::PartialOrd
- 小于等于
  `n <= 1`
  std::cmp::PartialOrd
- 大于
  `n > 1`
  std::cmp::PartialOrd
- 大于等于
  `n >= 1`
  std::cmp::PartialOrd
- 等于
  `n == 1`
  std::cmp::PartialOrd
- 不等于
  `n != 1`
  std::cmp::PartialOrd
- 逻辑与
  x.ok && y.ok
- 逻辑或
  x.ok || y.ok
- 右开区间范围
  start .. stop
- 右闭区间范围
  start ..= stop
- 赋值
  x = val
- 复合赋值
  *=, /=, %=, +=, -=, <<=, >>=, &=, ^=, |=
  DivAssign, RemAssign, AddAssign, SubAssign, ShlAssign, ShrAssign, BitAndAssign, BitXorAssign, BitOrAssign, MulAssign
- 闭包
  `|x, y| x + y`


所有可以链式书写的运算符都是左结合的。也就是说，诸如 a - b - c 之类的操作链会分组为 (a - b) - c
`* / % + - << >> & ^ | && || as`

比较运算符、赋值运算符 和 范围运算符（.. 和 ..=）则根本无法链式书写。




## 6.3 块与分号


块是一种最通用的表达式。一个块生成一个值，并且可以在任何需要值的地方使用

```rust
let display_name = match post.author() {
  Some(author) => author.name(),
  None => {
    let network_info = post.get_network_metadata()?;
    let ip = network_info.client_address();
    ip.to_string()
  }`
};
```


Some(author) => 之后的代码是简单表达式 author.name()，None => 之后的代码是一个块表达式，它们对 Rust 来说没什么不同。
块表达式的值是最后一个表达式 ip.to_string() 的值。

ip.to_string() 方法调用后面没有分号。
大多数 Rust代码行以分号或花括号结尾，就像 C 或 Java 一样。
如果一个块看起来很像 C 代码，在你熟悉的==每个地方都有分号==，那么它就会像 C 的块一样运行，并且==其值为 ()==。
正如第 2 章提到的，当块的最后一行==不带分号==时，就以==最后这个表达式的值==而不是通常的 () 作为块的值。



## 6.4 声明

除了表达式和分号，块还可以包含任意数量的声明。
最常见的是 let声明，它会声明局部变量

`let name: type = expr;`

类型和初始化代码是可选的，分号则是必需的

let 声明可以在不初始化变量的情况下声明变量，然后再用赋值语句来初始化变量
```rust
let name;     // not mut
if user.has_nickname() {
  name = user.nickname();
} else {
  name = generate_unique_name();
  user.register(&name);
}
```

可能偶尔会看到似乎在重新声明现有变量的代码
```rust
for line in file.lines() {
  let line = line?;
  // ...
}
```

这个 let 声明会创建一个不同类型的、新的（第二个）变量。
第一个 line 变量的类型是 `Result<String, io::Error>`。
第二个 line 变量则是 String。
第二个定义会在所处代码块的其余部分代替第一个定义。这叫作 遮蔽（shadowing）

该代码等效于如下内容：
```rust
for line_result in file.lines() {
  let line = line_result?;
  // ...
}
```

块还可以包含 语法项声明（item declaration）。
语法项是指可以在程序或模块中的任意地方出现的声明，比如 fn、struct 或 use。

```rust
use std::io;
use std::cmp::Ordering;
fn show_files() -> io::Result<()> {
  let mut v = vec![];
  // ...
  fn cmp_by_timestamp_then_name(a: &FileInfo, b: &FileInfo) ->
  Ordering {
    a.timestamp.cmp(&b.timestamp)
      // 首先，比较时间戳
      .reverse()
      // 最新的文件优先
      .then(a.path.cmp(&b.path)) // 通过路径做二级比较
  }
  v.sort_by(cmp_by_timestamp_then_name);
  // ...
}
```


## 6.5 if, match

```rust
if condition1 {
  block1
} else if condition2 {
  block2
} else {
  block_n
}
```

每个 condition 都必须是 bool 类型的表达式，不会隐式转换


match 表达式类似于 C 语言中的 switch 语句，但更灵活

```rust
match code {
  0 => println!("OK"),
  1 => println!("Wires Tangled"),
  2 => println!("User Asleep"),
  _ => println!("Unrecognized Error {}", code)
}
```

通配符模式 `_` 会匹配所有内容

将 _ 模式放在其他模式之前意味着它会优先于其他模式。这样一来，其他模式将永远没机会匹配到（编译器会发出警告）

编译器可以使用==跳转表==来优化这种 match，就像 C++ 中的 switch语句一样。
当 match 的每个分支都生成一个常量值时，也会应用与C++ 类似的优化。
在这种情况下，编译器会==构建出这些值的数组==，并将各个 match 项编译为数组访问。
除了边界检查，编译后的代码中根本不存在任何分支


match 的多功能性源于每个分支 => 左侧支持的多种 模式（pattern）

在上面的例子中，每个模式只是一个常量整数。
我们还展示了用以区分两种 Option 值的 match 表达式

```rust
match params.get("name") {
  Some(name) => println!("Hello, {}!", name),
  None => println!("Greetings, stranger.")
}
```

模式可以匹配一系列值，它可以解构元组、可以匹配结构体的各个字段、可以追踪引用、可以借用部分值，等等。
ch10.



```rust
match value {
  pattern => expr,
  ...
}
```
Rust 禁止 执行 未覆盖 所有可能值 的 match 表达式


if 表达式的所有块都必须生成 相同类型 的值
```rust
let suggested_pet = if with_wings { Pet::Buzzard } else { Pet::Hyena };// 正确
let favorite_number = if user.is_hobbit() { "eleventy-one" } else { 9 };// 错误
let best_sports_team = if is_hockey_season() { "Predators" };// 错误，因为7月份的结果是 ()
```

match 表达式的所有分支都必须具有 相同的类型



---
### 6.5.1 if let

```rust
if let pattern = expr {
  block1
} else {
  block2
}
```

给定的 expr 要么==匹配== pattern，这时会运行 block1；
要么无法匹配，这时会运行 block2。
有时这是从 Option 或 Result 中获取数据的好办法

```rust
if let Some(cookie) = request.session_cookie {
  return restore_session(cookie); // return 是 块的return？还是 外面的方法的 return
}
if let Err(err) = show_cheesy_anti_robot_task() {
  log_robot_attempt(err);
  politely_accuse_user_of_being_a_robot();
} else {
  session.mark_as_human();
}
```

凡是 if let 可以做到的，match 同样可以做到。
if let 表达式其实是只有一个模式的 match 表达式的简写形式


---
### 6.5.2 循环
4种

```rust
while condition {
  block
}

while let pattern = expr {
  block
}

loop {
  block
}

for pattern in iterable {
  block
}
```

while 循环或 for 循环的值总是 ()，因此它们的值通常没什么用。
如果指定了一个值，那么loop 表达式就能生成一个值

while let 循环类似于 if let。
在每次循环迭代开始时，expr 的值要么匹配给定的 pattern，这时会运行循环体（block）；
要么==不匹配==，这时会==退出循环==。


`.. 运算符`会生成一个 范围（range），即具有两个字段（start 和 end）的简单结构体。
`0..20` 与 `std::ops::Range { start: 0, end: 20 }` 相同。

各种范围都可以与 for 循环一起使用，因为Range 是一种可迭代类型，它实现了`std::iter::IntoIterator` 特型 (ch 15)


为了与 Rust 的移动语义保持一致，把值用于 for 循环会==消耗该值==

```rust
let strings: Vec<String> = error_messages();
for s in strings { // 在这里，每个String都会转移给s……
  println!("{}", s);
} // ……并在此丢弃
println!("{} error(s)", strings.len()); // 错误：使用了已移动出去的值
```

简单的补救措施是在循环中访问此==集合的引用==
循环变量也会变成对集合中每个条目的引用
```rust
for rs in &strings {
  println!("String {:?} is at address {:p}.", *rs, rs);
}
```
这里 &strings 的类型是 `&Vec<String>`，rs 的类型是&String。

遍历（可迭代对象的）可变引用会为每个元素提供一个可变引用
```rust
for rs in &mut strings { // rs的类型是&mut String
  rs.push('\n'); // 为每个字符串添加一个换行
}
```


## 6.6 循环中的控制流

break

loop 循环体中， break 后面可以 跟一个表达式，该表达式的值就是 loop 的值

```rust
// 对`next_line`的每一次调用，或者返回一个`Some(line)`（这里的`line`是
// 输入中的一行），或者当输入已结束时返回`None`。最终会返回以"answer: "
// 开头的第1行，如果没找到，就返回"answer: nothing"
let answer = loop {
  if let Some(line) = next_line() {
    if line.starts_with("answer: ") {
      break line;
    }
  } else {
    break "answer: nothing";
  }
};
```

loop 中的所有 break 表达式也必须生成具有相同类型的值，这样该类型就会成为这个 loop 本身的类型


continue 表达式会跳转到循环的下一次迭代

---

循环可以带有生命周期标签

标签也可以与 continue 一起使用。

`'search` 是外部for 循环的标签。因此，`break 'search` 会退出这层循环，而不是退出内部循环
```rust
'search:
for room in apartment {
  for spot in room.hiding_spots() {
    if spot.contains(keys) {
      println!("Your keys are {} in the {}.", spot, room);
      break 'search;    // !!!
    }
  }
}
```

break 可以同时具有标签和值表达式
```rust
// 找到此系列中第一个完全平方数的平方根
let sqrt = 'outer: loop {
  let n = next_number();
  for i in 1.. {
    let square = i * i;
    if square == n {
      // 找到了一个平方根
      break 'outer i;   // iiiiii
    }
    if square > n {
      // `n`不是完全平方数，尝试下一个
      break;    // 。。break `for`
    }
  }
};
```


## 6.7 return表达式

return 会==退出当前函数==，并向调用者返回一个值。

不带值的 return 是 return () 的简写

7.2.4 讲解 ?运算符
。。?;



## 6.8 为什么有 loop

rust需要保证 每个分支 都会返回 相同的类型。
但是rust 不会检查 循环条件
所以，下面的代码实际上是 安全的 ，只会从 while 中返回 i32, 但是编译器会 报错

```rust
fn wait_for_process(process: &mut Process) -> i32 {
  while true {
    if process.wait() {
      return process.exit_code();
    }
  }
} // 错误：类型不匹配：期待i32，实际找到了()
```

loop 表达式就是这个问题的“有话直说”式解决方案

---

rust的类型系统也受到控制流的影响

所有分支都必须返回相同的类型，但是在 break 或 return 表达式、无限 loop，或者调用 panic!() 或 std::process::exit() 等多种方式结束的块上强制执行此规则是 不现实 的。
这些表达式的共同点是它们永远都不会以通常的方式结束并生成一个值

在 Rust 中，这些表达式没有正常类型。
不能正常结束的表达式属于一个==特殊类型 !==，并且它们不受“类型必须匹配”这条规则的约束

std::process::exit() 的函数签名中看到 !
`fn exit(code: i32) -> !`

此处的 ! 表示 exit() ==永远不会返回==，它是一个 发散函数（divergent function）

如果此函数正常返回了，那么 Rust 就会认为它能正常返回反而是一个错误。


## 6.9 函数与方法调用

```rust
let x = gcd(1302, 462);// 函数调用
let room = player.location();// 方法调用
```

Rust 通常会在引用和它们所引用的值之间做出明确的区分。
如果将 &i32 传给需要 i32 的函数，则会出现类型错误。
你会注意到 `. 运算符`稍微放宽了这些规则。
在调用 `player.location()` 的方法中，player 可能是
一个 Player、
一个 &Player 类型的引用，
也可能是一个 `Box<Player>` 
或 `Rc<Player>` 类型的智能指针。

`.location()` 方法可以通过值或引用获取 player。同一个 `.location()` 语法适用于所有情况，因为 Rust 的 `. 运算符`会根据需要自动对 player ==解引用或借入== 一个对它的引用。


第三种语法用于调用类型关联函数，比如 Vec::new()：
`let mut numbers = Vec::new(); // 类型关联函数调用`

类似于面向对象语言中的静态方法：
普通方法会在 值 上调用（如 my_vec.len()），
类型关联函数会在 类型 上调用（如Vec::new()）

支持链式方法调用

在函数调用或方法调用中，泛型类型的常用语法 `Vec<T>` 是不起作用的

```rust
return Vec<i32>::with_capacity(1000); // 错误：是某种关于“链式比较”的错误消息
let ramp = (0 .. n).collect<Vec<i32>>(); // 同样的错误
```

问题在于，在表达式中 `< 是小于运算符`


用 `::<T>` 代替 `<T>` 。这样就解决了问题
```rust
return Vec::<i32>::with_capacity(1000); // 正确，改用::<
let ramp = (0 .. n).collect::<Vec<i32>>(); // 正确，改用::<
```

符号 `::<...>` 在 Rust 社区中被亲切地称为 比目鱼（turbofish）

通常可以删掉类型参数，让 Rust 来推断它们



## 6.10 字段与元素

`game.black_pawns   // 结构体字段`
`coords.1   // 元组元素`

如果 . 左边的值是引用或智能指针类型，那么它就会像方法调用一样 自动 解引用。

`pieces[i] // 数组元素`
方括号左侧的值也会自动解引用

从数组或向量中提取切片的写法很直观：
`let second_half = &game_moves[midpoint .. end];`

```rust
..  // RangeFull
a ..  // RangeFrom { start: a }
.. b  // RangeTo { end: b }
a .. b  // Range { start: a, end: b }
```


左值，因为赋值时它们可以出现在左侧


`..=` 运算符会生成 包含结束值 的范围

只有包含起始值的范围才是可迭代的，因为循环必须从某处开始。
但是在数组切片中，这 6 种形式都可以使用。
如果==省略==了范围的起点或末尾，则==默认==为被切片数据的起点或末尾。


---

6.11 引用运算符

地址运算符 & 和 &mut 已在第 5 章中介绍过

一元 * 运算符用于访问引用所指向的值。

当使用 . 运算符访问字段或方法时，Rust 会自动追踪引用，
因此只有想要 读取或写入 引用 所指的整个值 时 才需要用 * 运算符。




---

6.12 算术运算符、按位运算符、比较运算符和逻辑运算符


与在 C 中一样，a % b 会计算向 0 四舍五入的有符号余数或模数。
其结果与左操作数的符号相同。注意，% 既能用于整数，也能用于浮点数

Rust 还继承了 C 的按位整数运算符 &、|、^、<< 和 >>。
但是，Rust 会使用 ! 而不是 ~ 表示按位非
这意味着对于整数 n，不能用 !n 来表示“n 为 0”，而是应该写成 n == 0。


移位总是对有符号整数类型进行符号扩展，对无符号整数类型进行零扩展


与 C 不同，Rust 中 按位 运算的优先级 高于 比较 运算，因此如果编写 `x & BIT != 0`，那么就意味着 `(x & BIT) != 0`

Rust 的比较运算符是 ==、!=、<、<=、> 和 >=，参与比较的两个值必须具有相同的类型。

Rust 还有两个 短路 逻辑运算符 && 和 ||，它们的操作数都必须具有确切的 bool 类型。



---
6.13 赋值

= 运算符用于给 mut 变量及其字段或元素赋值。


如果值是非 Copy 类型的，则赋值会将其 移动 到目标位置。
值的所有权会从源转移给目标。
目标的先前值（如果有的话）将被丢弃。

支持复合赋值：
`total += item.price;`

不支持链式赋值：不能编写 a = b = 3 来将值 3 同时赋给 a 和 b

没有 C 的自增运算符 ++ 和自减运算符 --



---
## 6.14 类型转换

as

- 数值可以从 任意内值数值类型 转换为 其他内置数值类型
  转换为更窄的类型会导致截断。
  转换为更宽类型的有符号整数会进行符号扩展，转换为无符号整数会进行零扩展
  从浮点类型转换为整数类型会向 0 舍入，比如 -1.99 as i32就是 -1。
  如果值太大而无法容纳整数类型，则转换会生成整数类型可以表示的最接近的值，比如 1e6 as u8 就是 255。
- bool，char 类型值，或enum类型的值 可以转为 任何 整数类型
  不允许 反向转换
- 一些涉及不安全指针类型的转换也是允许的。参见 22.8 节。


几个更重要的自动转换。
- &String 类型的值会自动转换为 &str 类型，无须强制转换。
- `&Vec<i32>` 类型的值会自动转换为 `&[i32]`
- `&Box<Chessboard>` 类型的值会自动转换为 &Chessboard。

这些称为 隐式解引用，因为它们适用于所有实现了内置特型 Deref的类型。
==Deref 隐式转换==的目的是使智能指针类型（如 Box）的行为尽可能像其底层值



## 6.15 闭包

轻量级的类似函数的值

闭包通常由一个参数列表组成，在两条竖线之间列出，后跟一个表达式

`let is_even = |x| x % 2 == 0;`

Rust 会推断其参数类型和返回类型。你也可以像写函数一样显式地写出它们。
如果确实指定了返回类型，那么为了语法的完整性，闭包的主体必须是一个块：

```rust
let is_even = |x: u64| -> bool x % 2 == 0; // 错误
let is_even = |x: u64| -> bool { x % 2 == 0 }; // 正确
```
调用闭包和调用函数的语法是一样的：
`assert_eq!(is_even(14), true);`




# ch07 错误处理

两类错误处理：panic 和 Resul

普通错误使用 Result 类型来处理。
Result 通常用以表示由程序==外部的事物引发==的错误，比如错误的输入、网络中断或权限问题。

panic 针对的是另一种错误，即那种 永远不该发生的 错误



## 7.1 panic

遇到下列问题，可以断定程序自身存在问题，会引发 panic
- 数组越界访问；
- 整数除以 0；
- 在恰好为 Err 的 Result 上调用 .expect()；
- 断言失败。

Rust 既可以在发生 panic 时展开调用栈，也可以中止进程。
展开调用栈是默认方案。


7.1.1 展开调用栈

1. 错误消息打印到终端
```text
thread 'main' panicked at 'attempt to divide by zero',
pirates.rs:3780
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```
如果设置了 RUST_BACKTRACE 环境变量，那么就像这条消息中建议的，Rust 也会在这里转储当前的调用栈。

2. 展开调用栈
当前函数使用的任何临时值、局部变量或参数都将按照与创建它们时相反的顺序被丢弃。
丢弃一个值仅仅意味着随后会进行清理
清理了当前函数调用后，我们将继续执行到其调用者中，以相同的方式丢弃其变量和参数

3. 最后，线程退出。
如果 panic 线程是主线程，则整个进程退出（使用非零退出码）


panic 不是崩溃，也不是未定义行为。
它更像是 Java 中的 RuntimeException 或 C++ 中的 ==std::logic_error==


panic 是基于线程的。
一个线程 panic 时，其他线程可以继续做自己的事。
第 19 章会展示父线程如何发现子线程中的 panic 并优雅地处理错误。

还有一种方法可以捕获调用栈展开，让线程“存活”并继续运行。
标准库函数 ==std::panic::catch_unwind()== 可以做到这一点。



7.1.2 中止
在两种情况下 Rust 不会试图展开调用栈。

1. 如果 Rust 在试图清理第一个 panic 时，.drop() 方法触发了第二个 panic，那么这个 panic 就是致命的。
Rust 会停止展开调用栈并中止整个进程

2. 如果使用 `-C panic=abort` 参数进行编译，那么程序中的第一个 panic 会立即中止进程。



## 7.2 Result

Rust 中没有异常。
相反，函数执行失败时会有像下面这样的返回类型：
`fn get_weather(location: LatLng) -> Result<WeatherReport, io::Error>`

每当调用此函数时，Rust 都会要求我们编写某种错误处理代码。
如果不对 Result 执行某些操作，就无法获取 WeatherReport；
如果未使用 Result 值，就会收到编译器警告。


7.2.1 捕获错误

第 2 章中已经展示过 Result 最彻底的处理方式：使用 match 表达式。

```rust
match get_weather(hometown) {
  Ok(report) => {
    display_weather(hometown, &report);
  }
  Err(err) => {
    println!("error querying the weather: {}", err);
    schedule_weather_retry();
  }
}
```
这相当于其他语言中的 ==try/catch==。
如果想直接处理错误而不是将错误传给调用者，就可以使用这种方式。

match 有点儿冗长，因此 `Result<T, E>` 针对一些常见的特定场景提供了多个有用的方法，每个方法在其实现中都有一个 match 表达式。

- result.is_ok()（已成功）和 
- result.is_err()（已出错）
  返回一个 bool，告知此结果是成功了还是出错了。
- result.ok()（成功值）
  以 Option<T> 类型返回成功值（如果有的话）。如果 result是成功的结果，就返回 Some(success_value)；否则，返回None，并丢弃错误值。
- result.err()（错误值）
  以 Option<E> 类型返回错误值（如果有的话）。
- ==result.unwrap_or(fallback)（解包或回退值）==
  如果 result 为成功结果，就返回成功值；否则，返回fallback，丢弃错误值。
  是 .ok() 的一个很好的替代品
- result.unwrap_or_else(fallback_fn)（解包，否则调用）
  传入一个函数或闭包.只有在得到错误结果时才会调用 fallback_fn
- result.unwrap()（解包）
  成功结果，则返回成功值，如果是错误结果，则 panic
- result.expect(message)（期待）
  和 unwrap 一样，但在 panic是 打印 message
- result.as_ref()（转引用）
  将 `Result<T, E>` 转换为 `Result<&T, &E>`。
- result.as_mut()（转可变引用）
  其返回类型是 `Result<&mut T, &mut E>`。

最后这两个方法之所以有用，是因为前面列出的所有其他方法，除了.is_ok() 和 .is_err()，都在**消耗** result。

假设你想调用 result.ok()，但要让 result 保持不可变状态，那么就可以写成 result.as_ref().ok()，它只会借用 result，返回 `Option<&T>` 而非 `Option<T>`。



7.2.2 Result 类型别名

下面似乎忽略了 Result 中的错误类型：
`fn remove_file(path: &Path) -> Result<()>`

这意味着正在使用 Result 的类型别名。

类型别名是类型名称的一种简写形式。
模块通常会定义一个 Result类型的别名，以免重复编写模块中几乎每个函数都要用到的 Error类型。
例如，标准库的 std::io 模块包括下面这行代码：
`pub type Result<T> = result::Result<T, Error>;`

这定义了一个公共类型 `std::io::Result<T>`，它是 `Result<T, E>` 的别名，但将错误类型硬编码为 `std::io::Error`。
实际上，这意味着如果你写下 `use std::io;`，那么 Rust 就会将`io::Result<String>` 当作 `Result<String, io::Error>` 的简写形式。



7.2.3 打印错误

有时处理错误的唯一方法是将其转储到终端并继续执行

`println!("error querying the weather: {}", err);`

标准库定义了几种名称平平无奇的错误类型：std::io::Error、std::fmt::Error、std::str::Utf8Error 等。
它们都实现了一个公共接口，即 `std::error::Error` 特型，这意味着它们都有
以下特性和方法。
`println!()`（打印）

使用格式说明符 `{}` 打印错误通常只会显示一条简短的错误消息。
或者，也可以使用格式说明符 `{:?}`，以获得该错误的 Debug 视图。

```text
// `println!("error: {}", err);`的结果
error: failed to look up address information: No address associated with hostname

// `println!("error: {:?}", err);`的结果
error: Error { repr: Custom(Custom { kind: Other, error: StringError("failed to look up address information: No address associated with hostname") }) }
```

- err.to_string()（转字符串）
  以 String 形式返回错误消息。
- err.source()（错误来源）
  返回导致 err 的底层错误的 Option


如果想确保打印所有可用信息，请使用下面这个函数
```rust
use std::error::Error;
use std::io::{Write, stderr};
/// 把错误消息转储到`stderr`
///
/// 如果在构建此错误消息或将其写入`stderr`期间发生了另一个错误，就忽略新的错
误
fn print_error(mut err: &dyn Error) {
  let _ = writeln!(stderr(), "error: {}", err);
  while let Some(source) = err.source() {
    let _ = writeln!(stderr(), "caused by: {}", source);
    err = source;
  }
}
```

writeln! 宏类似于 println!，但它会将数据写入所选的流。

可以使用 eprintln! 宏做同样的事情，但是如果 eprintln! 中发生了错误，就会 panic。
在 print_error 中，要忽略在写入消息时出现的错误


标准库的这些错误类型不包括调用栈跟踪，但是，当与不稳定版本的Rust 编译器一起使用时，可以使用广受欢迎的 `anyhow` crate 提供的一个现成的错误类型。


---
7.2.4 传播错误
希望错误沿着调用栈向上传播。

Rust 的 ? 运算符可以执行此操作。可以为任何生成 Result 的表达式加上一个 ?，比如将其加在函数调用的结果后面
`let weather = get_weather(hometown)?;`

? 的行为取决于此函数是返回了成功结果还是错误结果。
- 成功结果，那么它会==解包== Result 以获取其中的成功值
- 如果是错误结果，那么它会立即从所在函数返回，将错误结果沿着调用链向上传播。为了确保此操作有效，? ==只能==在==返回类型为Result== 的函数中的 Result 值上使用。

? 运算符并无任何神奇之处。可以使用 match 表达式来表达同样的意图，只是更冗长：
```rust
let weather = match get_weather(hometown) {
  Ok(success_value) => success_value,
  Err(err) => return Err(err)
};
```


在旧式代码中，你可能还会看到 try!() 宏，在 Rust 1.13 引入 ? 运算符之前，这是传播错误的常用方法：
`let weather = try!(get_weather(hometown));`
此宏会扩展为一个 match 表达式，就像之前那段代码一样。


? 的作用也与 Option 类型相似。
在返回 Option 类型的函数中，也可以使用 ? 解包某个值，这样当遇到 None 时就会提前返回。
`let weather = get_weather(hometown).ok()?;`



7.2.5 处理多种 Error 类型

。就是 ? 触发的 Error 可能无法转为 方法声明的 Error类型


1. 通过代码转换
第 2 章中用于创建曼德博集图像文件的 image crate 定义了自己的错误类型 ImageError，并实现了从 io::Error 和其他几种错误类型到 ImageError 的转换。
如果你想采用这种方法，可以试试 `thiserror` crate，它旨在帮助你用几行代码就定义出良好的错误类型。

2. 内置功能
所有标准库中的错误类型都可以转换为类型 `Box<dyn std::error::Error + Send + Sync + 'static>`

`dyn std::error::Error` 表示“任何错误”，
`Send + Sync + 'static` 表示可以安全地在线程之间传递

建议 anyhow

```rust
type GenericError = Box<dyn std::error::Error + Send + Sync +
'static>;
type GenericResult<T> = Result<T, GenericError>;

// -----

let io_error = io::Error::new( // 制作自己的io::Error
    io::ErrorKind::Other, "timed out");
return Err(GenericError::from(io_error)); // 手动转换成GenericError
```

7.2.6 处理"不可能发生" 的错误

`let num = digits.parse::<u64>().unwrap();`
如果我们确定 转换必然成功 (即，必然是数字)，那么可以 unwrap()，如果失败的话，会 panic。  
当然，如果代码返回 GenericResult ，那么可以 ? 操作符



7.2.7 忽略错误

`writeln!(stderr(), "error: {}", err); // 警告：未使用的结果`

`let _ = writeln!(stderr(), "error: {}", err); // 正确，忽略结果`


7.2.8 处理main()中的错误

main() 不能使用 ?，因为它的返回类型不是 Result。 所以它必须处理

处理 main() 中错误的最简单方式是使用 .expect()：
```rust
fn main() {
  calculate_tides().expect("error"); // 责任止于此
}
```

如果 calculate_tides() 返回错误结果，那么 .expect() 方法就会 panic。

也可以指定 main() 的返回类型
```rust
fn main() -> Result<(), TideCalcError> {
  let tides = calculate_tides()?;
  print_tides(tides);
  Ok(())
}
```

自己打印
```rust
fn main() {
  if let Err(err) = calculate_tides() {
    print_error(&err);
    std::process::exit(1);
  }
}
```


### 7.2.9 声明自定义错误类型

```rust
// json/src/error.rs
#[derive(Debug, Clone)]
pub struct JsonError {
  pub message: String,
  pub line: usize,
  pub column: usize,
}
```

这个结构体叫作 json::error::JsonError。当你想引发这种类型的错误时，可以像下面这样写

```rust
return Err(JsonError {
  message: "expected ']' at end of array".to_string(),
  line: current_line,
  column: current_column
});
```


如果你希望达到你的库用户的预期，确保这个错误类型像标准错误类型一样工作，那么还有一点儿额外的工作要做
```rust
use std::fmt;
// 错误应该是可打印的
impl fmt::Display for JsonError {
  fn fmt(&self, f: &mut fmt::Formatter) -> Result<(), fmt::Error> {
    write!(f, "{} ({}:{})", self.message, self.line,
    self.column)
  }
}
// 错误应该实现std::error::Error特型，但使用Error各个方法的默认定义就够了
impl std::error::Error for JsonError { }
```

最常用的一个是 `thiserror` crate，它会帮你完成之前的所有工作，让你像下面这样编写错误定义

```rust
use thiserror::Error;
#[derive(Error, Debug)]
#[error("{message:} ({line:}, {column})")]
pub struct JsonError {
  message: String,
  line: usize,
  column: usize,
}
```
`#[derive(Error)]` 指令会让 thiserror 生成前面展示过的代码，这可以节省大量的时间和精力。



# ch08 crate 与模块

弄清楚 crate 是什么以及它们如何协同工作的最简单途径是，使用带有 --verbose 标志的 cargo build 来构建具有某些依赖项的现有项目。

Cargo.toml 文件中指定了我们想要的每个 crate 的版本


一旦有了源代码，Cargo 就会编译所有的 crate。
它会为项目依赖图中的每个 crate 都运行一次 rustc（Rust 编译器）。
编译库时，Cargo 会使用 `--crate-type lib` 选项。
这会告诉 rustc 不要寻找 main() 函数，而是==生成一个 .rlib 文件==，其中包含一些已编译代码，可用于创建二进制文件和其他 .rlib 文件。


编译程序时，Cargo 会使用 `--crate-type bin`，结果是目标平台的二进制可执行文件，比如 Windows 上的 mandelbrot.exe。


对于每个 rustc 命令，Cargo 都会传入 --extern 选项，给出crate 将使用的每个库的文件名。
这样，当 rustc 看到一行代码（如 use image::png::PNGEncoder）时，就可以确定 image是另一个 crate 的名称。


`cargo build --release` 会生成优化过的程序。
这种程序运行得更快，但它们的编译时间更长、运行期不会检查整数溢出、会跳过 debug_assert!() 断言，并且在 panic 时生成的调用栈跟踪通常不太可靠。



8.1.1 版本

为了在不破坏现有代码的情况下继续演进，Rust 会使用版本。
Cargo.toml 文件顶部的 `[package]` 部分使用下面这样的行来表明自己是用哪个版本的 Rust 编写的
`edition = "2021"`

如果该关键字不存在，则假定为 2015 版


如果你有一个用旧版本的 Rust 编写的 crate，则 cargo fix 命令能帮助你自动将代码升级到新版本。
Rust 版本指南详细解释了cargo fix 命令。


8.1.2 创建配置文件

在 Cargo.toml 文件中放置几个配置设定区段，这些设定会影响 cargo 生成的 rustc 命令行

|命令|使用的Cargo.toml区段|
|--|--|
|cargo build|`[profile.dev]`|
|cargo build --release|`[profile.release]`|
|cargo test|`[profile.text]`|



## 8.2 模块 mod

crate 是关于项目 间 代码共享的，
而模块是关于项目 内 代码组织的。


```rust
mod spores {
  use cells::{Cell, Gene};
  /// 由成年蕨类植物产生的细胞。作为蕨类植物生命周期的一部分，细胞会随风
  /// 传播。一个孢子会长成原叶体（一个完整的独立有机体，最大直径达5毫米），
  /// 原叶体产生的受精卵会长成新的蕨类植物（植物的性别很复杂）
  pub struct Spore {
    ...
  }
  /// 模拟减数分裂产生孢子
  pub fn produce_spore(factory: &mut Sporangium) -> Spore {
    ...
  }
  /// 提取特定孢子中的基因
  pub(crate) fn genes(spore: &Spore) -> Vec<Gene> {
    ...
  }
  /// 混合基因以准备减数分裂（细胞分裂间期的一部分）
  fn recombine(parent: &mut Cell) {
    ...
  }
  ...
}
```


模块是一组语法项的集合，这些语法项具有命名的特性，比如此示例中的 Spore 结构体和 3 个函数。
pub 关键字会使某个语法项声明为公共项，这样它就可以从模块外部访问了。

如果把一个函数标记为 pub(crate)，那么就意味着它可以在这个crate 中的任何地方使用，但不会作为外部接口的一部分公开。
它不能被其他 crate 使用，也不会出现在这个 crate 的文档中


---
### 8.2.1 嵌套模块

```rust
mod plant_structures {    // .. no pub, but it is pub.
  pub mod roots {     // ...pub
  ...
  }
  pub mod stems {
  ...
  }
  pub mod leaves {
  ...
  }
}

```

也可以指定 `pub(super)`，让语法项只对其父模块可见。还可以指定
`pub(in <path>)`，让语法项在特定的父模块及其后代中可见。

```rust
mod plant_structures {
  pub mod roots {
    pub mod products {
      pub(in crate::plant_structures::roots) struct Cytokinin {
        ...
      }
    }
    use products::Cytokinin; // 正确：在`roots`模块中可见
  }
  use roots::products::Cytokinin; // 错误：`Cytokinin`是私有的
}
// 错误：`Cytokinin`是私有的
use plant_structures::roots::products::Cytokinin;
```

通过这种方式，我们可以写出一个完整的程序，把大量代码和完整的模块层次结构以我们想要的任何方式关联起来，并放在同一个源文件中

这种方式写代码相当痛苦，因此还有另一种选择

8.2.2 单独文件中的模块

模块还可以这样写：
`mod spores;`
我们告诉 Rust 编译器，spores 模块保存在一个单独的名为spores.rs 的文件中：

```rust
// spores.rs        // 。。 文件名
/// 由成年蕨类植物产生的细胞……
pub struct Spore {
 ...
}
/// 模拟减数分裂产生孢子
pub fn produce_spore(factory: &mut Sporangium) -> Spore {
 ...
}
/// 提取特定孢子中的基因
pub(crate) fn genes(spore: &Spore) -> Vec<Gene> {
 ...
}
/// 混合基因以准备减数分裂（细胞分裂间期的一部分）
fn recombine(parent: &mut Cell) {
 ...
}
```


==当 Rust 看到 mod spore; 时，会同时检查 `spores.rs` 和 `spores/mod.rs`，如果两个文件都不存在，或者都存在，就会报错==


```text
fern_sim/
├── Cargo.toml
└── src/
    ├── main.rs
    ├── spores.rs
    └── plant_structures/
        ├── mod.rs        // 。。。-----
        ├── leaves.rs
        ├── roots.rs
        └── stems.rs
```

在 main.rs 中，我们声明了 plant_structures 模块：
`pub mod plant_structures;`

这会导致 Rust 加载 plant_structures/mod.rs，该文件声明了 3 个子模块：
```rust
// 在plant_structures/mod.rs中
pub mod roots;
pub mod stems;
pub mod leaves;
```

```rust
fern_sim/
├── Cargo.toml
└── src/
    ├── main.rs
    ├── spores.rs
    └── plant_structures/
        ├── mod.rs
        ├── leaves.rs
        ├── roots.rs
        ├── stems/
        │    ├── phloem.rs
        │    └── xylem.rs
        └── stems.rs
```

```rust
// 在plant_structures/stems.rs中
pub mod xylem;
pub mod phloem;
```

这 3 种选项（
- 模块位于自己的文件中、
- 模块位于自己的带有 mod.rs的目录中，
- 以及模块在自己的文件中，并带有包含子模块的补充目录）
为模块系统提供了足够的灵活性，以支持你可能用到的几乎任何项目结构。





### 8.2.3 路径与导入

`::` 运算符 用于 访问模块中的 各项特性。

```rust
if s1 > s2 {
  std::mem::swap(&mut s1, &mut s2);
}
```

```rust
use std::mem;
if s1 > s2 {
  mem::swap(&mut s1, &mut s2);
}
```

可以通过写 use std::mem::swap; 来导入 swap 函数本身

然而，我们之前的==编写风格==通常被认为是==最好==的：==导入类型、特型和模块==（如 std::mem），然后==使用相对路径==访问其中的==函数、常量和其他成员==。


可以一次导入多个名称：

```rust
use std::collections::{HashMap, HashSet}; // 同时导入两个模块
use std::fs::{self, File}; // 同时导入`std::fs`和`std::fs::File`
use std::io::prelude::*; // 导入所有语法
```

可以使用 as 导入一个语法项，但在本地赋予它一个不同的名称
```rust
use std::io::Result as IOResult;
// 这个返回类型只是`std::io::Result<()>`的另一种写法：
fn save_spore(spore: &Spore) -> IOResult<()>
```

模块 不会 自动从其父模块继承名称


每个模块都会以“白板”开头，并且必须导入它使用的名称


默认情况下，路径是相对于当前模块的
```rust
// in proteins/mod.rs
// 
// 存在：proteins/synthesis.rs
// 从某个子模块导入
use synthesis::synthesize;
```

self 也是当前模块的同义词
```rust
// in proteins/mod.rs
// 从枚举中导入名称，因此可以把赖氨酸写成`Lys`，而不是`AminoAcid::Lys`
use self::AminoAcid::*;
```
或者简单地写成如下形式。
```rust
// 在proteins/mod.rs中
use AminoAcid::*;
```

关键字 super 和 crate 在路径中有着特殊的含义：
- super 指的是父模块，
- crate 指的是当前模块所在的 crate。


子模块可以使用 use super::* 访问其父模块中的私有语法项。


如果有一个与你正使用的 crate 同名的模块，那么引用它们的内容时有一些注意事项。
如果你的程序在其 Cargo.toml 文件中将 image crate 列为依赖项，但还有另一个名为 image 的模块，那么以 image 开头的路径就是有歧义的

```rust
mod image {
  pub struct Sampler {
  ...
  }
}
// 错误：它引用的是我们的`image`模块还是`image crate`？
use image::Pixels;
```

为了解决歧义，Rust 有一种特殊的路径，称为绝对路径，该路径以:: 开头，总会引用外部 crate。
```rust
use ::image::Pixels; // `image crate`中的`Pixels`

use self::image::Sampler; // `image`模块中的`Sampler`
```

self 和 super 类似于 . 和 .. 这样的特殊目录。


8.2.4 标准库预导入

标准库 std 会自动链接到每个项目


还有一些特别的便捷名称（如 Vec 和 Result）会包含在标准库预导入中并自动导入。
Rust 的行为就好像每个模块（包括根模块）都用以下导入语句开头一样：
`use std::prelude::v1::*`

std::prelude::v1 是唯一会自动导入的预导入。

把一个模块命名为 prelude 只是一种约定，旨在告诉用户应该==使用 * 导入它==。


8.2.5 公开 use 声明

虽然 use 声明只是个别名，但也可以公开它们：

```rust
// 在plant_structures/mod.rs中
...
pub use self::leaves::Leaf;
pub use self::roots::Root;
```

这意味着 Leaf 和 Root 是 plant_structures 模块的公共语法项。
它们还是 plant_structures::leaves::Leaf 和plant_structures::roots::Root 的简单别名。



### 8.2.6 公开结构体字段

```rust
pub struct Fern {
  pub roots: RootSet,
  pub stems: StemSet
}
```

结构体的字段，甚至是私有字段，都可以在声明该结构体的整个模块及其子模块中访问。
在模块之外，只能访问公共字段。




### 8.2.7 静态变量与常量

除了函数、类型和嵌套模块，模块还可以定义 常量 和 静态 变 量。

`pub const ROOM_TEMPERATURE: f64 = 20.0; // 摄氏度`

`pub static ROOM_TEMPERATURE: f64 = 68.0; // 华氏度`

没有 mut 常量。静态变量可以标记为 mut

Rust 没有办法强制执行其关于 mut 静态变量的独占访问规则。
因此，mut 静态变量本质上是非线程安全的，安全代码根本不能使用它们

Rust 不鼓励使用全局可变状态。有关备选方案的讨论，请参阅 19.3.11 节。



## 8.3 将程序变成库

Cargo.toml

```rust
[package]
name = "fern_sim"
version = "0.1.0"
authors = ["You <you@example.com>"]
edition = "2021"
```

很容易将这个程序变成库，步骤如下。
1. 文件 src/main.rs 重命名为 src/lib.rs
2. pub 关键字添加到 src/lib.rs 中的语法项上，这些语法项将成为这个库的公共特性。
3. 将 main 函数移动到某个临时文件中。（我们暂时不同管它。）

```rust
// 生成的 src/lib.rs 文件如下所示：
pub struct Fern {
  pub size: f64,
  pub growth_rate: f64
}
impl Fern {
  /// 模拟一株蕨类植物在一天内的生长
  pub fn grow(&mut self) {
    self.size *= 1.0 + self.growth_rate;
  }
}
/// 执行days天内某株蕨类植物的模拟
pub fn run_simulation(fern: &mut Fern, days: usize) {
  for _ in 0 .. days {
    fern.grow();
  }
}
```

不需要更改 Cargo.toml 中的任何内容

这是因为这个最小化的 Cargo.toml 文件只是为了让 Cargo 保持默认行为而已。
默认设定下，cargo build 会查看源目录中的文件并根据文件名确定要构建的内容。
当它发现存在文件 `src/lib.rs` 时，就知道要构建一个库。

`src/lib.rs` 中的代码构成了库的根模块。
其他使用这个库的 crate 只能访问这个根模块的公共语法项



## 8.4 src/bin 目录

将下面这段代码放入名为 src/bin/efern.rs 的文件中：

```rust
use fern_sim::{Fern, run_simulation};

fn main() {
  let mut fern = Fern {
    size: 1.0,
    growth_rate: 0.001
  };
  run_simulation(&mut fern, 1000);
  println!("final fern size: {}", fern.size);
}
```

`cargo build --verbose`
`cargo run --bin efern`

Cargo 会自动将 src/bin 中的 .rs 文件视为应该构建的额外程序。


---

把这个程序放在它自己的独立项目中，再保存到一个完全独立的目录中，然后在它自己的 Cargo.toml 中将 fern_sim 列为依赖项：
```text
[dependencies]
fern_sim = { path = "../fern_sim" }
```


## 8.5 属性

Rust 程序中的任何语法项都可以用属性进行装饰。

属性是 Rust 的通用语法，用于向编译器提供各种指令和建议

```rust
#[allow(non_camel_case_types)]
pub struct git_revspec {
  ...
}
```


条件编译是使用名为 `#[cfg]` 的属性编写的另一项特性：
```rust
// 只有当我们为Android构建时才在项目中包含此模块
#[cfg(target_os = "android")]
mod mobile;
```

常用 cfg 参数
- test
- debug_assertions
- unix
- windows
- target_pointer_width = "64"
- target_arch = "x86_64"
- target_os = "macos"
- feature = "robots"
- not(A)
- all(A,B)
- any(A,B)


`#[inline]`
内联

当在一个 crate 中定义的函数或方法在另一个 crate 中被调用时，Rust 不会将其内联，除非它是泛型的（具有类型参数）或明确标记为 `#[inline]`。

在其他情况下，编译器只会将 `#[inline]` 视为建议

Rust 还支持更坚定的 `#[inline(always)]`（要求函数在每个调用点内联展开）和 `#[inline(never)]`（要求函数永不内联）。



要将属性附着到整个 crate 上，请将其添加到 main.rs 文件或 lib.rs 文件的顶部，放在任何语法项之前，并写成 #!，而不是 #
```rust
// libgit2_sys/lib.rs
#![allow(non_camel_case_types)]
pub struct git_revspec {
  ...
}
pub struct git_error {
  ...
}
```

`#![allow]` 属性会附着到整个 libgit2_sys 包而不仅仅是 struct git_revspec 上

#! 也可以在函数、结构体等内部使用（但 #! 通常只用在文件的开头，以将属性附着到整个模块或 crate 上）。
某些属性始终使用 #! 语法，因为它们只能应用于整个 crate。

`#![feature]` 属性用于启用 Rust 语言和库的不稳定特性，这些特性是实验性的，因此可能有 bug 或者未来可能会被更改或移除。

```rust
#![feature(trace_macros)]
fn main() {
  // 我想知道这个使用assert_eq!的代码替换（展开）后会是什么样子！
  trace_macros!(true);
  assert_eq!(10*10*10 + 9*9*9, 12*12*12 + 1*1*1);
  trace_macros!(false);
}
```


## 8.6 测试与文档

内置了一个简单的单元测试框架

测试是标有 `#[test]` 属性的普通函数

```rust
#[test]
fn math_works() {
  let x: i32 = 1;
  assert!(x.is_positive());
  assert_eq!(x + 1, 2);
}
```

`cargo test` 会运行项目中的所有测试

`cargo test math` 会运行名称中包含 math 的所有测试。



测试通常会使用 assert! 和 assert_eq! 这两个来自 Rust 标准库的宏
assert!和 assert_eq! 会包含在发布构建中

debug_assert! 和 debug_assert_eq! 来编写仅在调试构建中检查的断言。

要测试各种出错情况，请将 `#[should_panic]` 属性添加到你的测试中
```rust
/// 正如我们在第7章中所讲的那样，只有当除以零导致panic时，这个测试才能通过
#[test]
#[allow(unconditional_panic, unused_must_use)]
#[should_panic(expected="divide by zero")]
fn test_divide_by_zero_error() {
  1 / 0; // 应该panic！
}
```
在这个例子中，还需要添加一个 allow 属性，以让编译器允许我们做一些它本可以静态证明而无法触发 panic 的事情，然后才能执行除法并丢弃答案，因为在正常情况下，它会试图阻止这种愚蠢行为。


还可以从测试中返回 `Result<(), E>`。只要错误变体实现了 Debug特型（通常都实现了），你就可以简单地使用 ? 抛弃 Ok 变体以返回Result

```rust
use std::num::ParseIntError;
/// 如果"1024"是一个有效的数值（这里正是如此），那么本测试就会通过
#[test]
fn explicit_radix() -> Result<(), ParseIntError> {
  i32::from_str_radix("1024", 10)?;
  Ok(())
}
```
标有 `#[test]` 的函数是有条件编译的。
普通的 cargo build 或cargo build --release 会跳过测试代码。
但是当你运行 cargo test 时，Cargo 会分两次来构建你的程序：一次以普通方式，一次带着你的测试和已启用的测试工具。
这意味着你的单元测试可以与它们所测试的代码一起使用，按需访问内部实现细节，而且没有运行期成本。
但是，这可能会导致一些警告


当你的测试变得很庞大以至于需要支撑性代码时，应该按照惯例将它们放在 tests 模块中，并使用 `#[cfg]` 属性声明整个模块仅用于测试
```rust
#[cfg(test)] // 只有在测试时才包含此模块
mod tests {
  fn roughly_equal(a: f64, b: f64) -> bool {
    (a - b).abs() < 1e-6
  }
  #[test]
  fn trig_works() {
    use std::f64::consts::PI;
    assert!(roughly_equal(PI.sin(), 0.0));
  }
}
```

Rust 的测试工具会使用 多个线程 同时运行好几个测试，这是 Rust 代码默认线程安全的附带好处之一

要禁用此功能，
请运行单个测试 cargo test testname 或
运行 `cargo test -- --test threads 1`。（第一个 -- 确保 cargo test 将 --test threads

测试工具只会显示失败测试的输出。如果也想展示成功测试的输出，请运行 `cargo test -- --nocapture`


8.6.1 集成测试


### 8.6.2 文档
cargo doc

```text
$ cargo doc --no-deps --open
Documenting fern_sim v0.1.0 (file:///.../fern_sim)
```

- --no-deps
  要求 Cargo 只为 fern_sim 本身生成文档，而不会为它依赖的所有 crate 生成文档。
- --open
  要求 Cargo 随后在浏览器中打开此文档。


文档型注释
当 Rust 看到以 3 个斜杠开头的注释时，会将它们视为 `#[doc]` 属性。
```rust
/// 模拟减数分裂产生孢子
pub fn produce_spore(factory: &mut Sporangium) -> Spore {
 ...
}

#[doc = "模拟减数分裂产生孢子。"]
pub fn produce_spore(factory: &mut Sporangium) -> Spore {
 ...
}
```

以 //! 开头的注释会被视为 `#![doc]` 属性并附着到封闭块一级的特性（通常是模块或 crate）上


Cargo 会查找路径所指的内容，并在相应的文档页面中将其替换为正确的链接。
例如，从如下代码生成的文档会链接到VascularPath、Leaf 和 Root 的文档页面
```rust
/// 创建并返回一个[`VascularPath`]，它表示从给定的[`Root`][r]到
/// 给定的[`Leaf`](leaves::Leaf)之间的营养路径
///
/// [r]: roots::Root
pub fn trace_path(leaf: &leaves::Leaf, root: &roots::Root) ->
VascularPath {
}
```

你还可以添加搜索别名，以便使用内置搜索功能更轻松地查找内容。
在此 crate 的文档中搜索“path”或“route”都能找到 VascularPath
```rust
#[doc(alias = "route")]
pub struct VascularPath {
  ...
}
```

为了处理较长的文档块或者简化工作流，还可以在文档中包含外部文件。
`#![doc = include_str!("../README.md")]`

你可以使用反引号（`）来标出文本中的少量代码。
也可以使用 Markdown 的三重反引号（```）来标记代码块




8.6.3 文档测试

当你在 Rust 库 crate 中运行测试时，Rust 会检查文档中出现的所有代码是否真能如预期般工作。

```rust
use std::ops::Range;

/// 如果两个范围重叠，就返回true
///
/// assert_eq!(ranges::overlap(0..7, 3..10), true);
/// assert_eq!(ranges::overlap(1..5, 101..105), false);
///
/// 如果任何一个范围为空，则它们不会被看作是重叠的
///
/// assert_eq!(ranges::overlap(0..0, 0..10), false);
///
pub fn overlap(r1: Range<usize>, r2: Range<usize>) -> bool {
  r1.start < r1.end && r2.start < r2.end &&
  r1.start < r2.end && r2.start < r1.end
}
```


要告诉 Rust 编译你的示例，但不实际运行它，请使用带有 no_run 注释的多行代码块
```rust
/// 将本地玻璃栽培箱的所有照片上传到在线画廊
///
/// ```no_run
/// let mut session = fern_sim::connect();
/// session.upload_all();
/// ```
pub fn upload_all(&mut self) {
  ...
}
```

如果代码甚至都不希望编译，请改用 ignore 而不是 no_run。


## 8.7 指定依赖项

`image = "0.6.1"`

使用根本没发布在 crates.io 上的依赖项
一种方法是指定 Git 存储库 URL 和修订号
`image = { git = "https://github.com/Piston/image.git", rev = "528f19c" }`

你可以指定要使用的特定版本（rev）、标签（tag）或分支名（branch）

另一种方法是指定一个包含 crate 源代码的目录：
`image = { path = "vendor/image" }`


### 8.7.1 版本

当你在 Cargo.toml 文件中写入类似 `image = "0.13.0"` 这样的内容时，Cargo 会相当宽松地理解它。
它会使用自认为与版本 0.13.0兼容的最新版本的 image。

- 以 0.0 开头的版本号还非常原始，所以 Cargo 永远不会假定它能与任何其他版本兼容。
- 以 0.x（x 不为 0）开头的版本号，可认为与 0.x 系列的版本兼容。前面我们指定了 image 版本为 0.6.1，但如果可用，则Cargo 会使用 0.6.3。
- 一旦项目达到 1.0，只有出现新的主版本号时才会破坏兼容性。因此，如果你要求版本为 2.0.1，那么 Cargo 可能会使用2.17.99，但不会使用 3.0。


|命令|含义|
|--|--|
|image = "=0.10.0" |仅使用确切的版本 0.10.0|
|image = ">=1.0.5"|使用 1.0.5 或更高版本（甚至 2.9，如果其可用的话）|
|image = ">1.0.5 <1.1.9"|使用高于 1.0.5 但低于 1.1.9 的版本|
|image = "<=2.7.10"|使用 2.7.10 或更早的任何版本|

你偶尔会看到的另一种版本规范是使用通配符 *。它会告诉 Cargo任何版本都可以


8.7.2 Cargo.lock

当第一次构建项目时，Cargo 会输出一个 Cargo.lock 文件，以记录它使用的每个crate 的确切版本。
以后的构建都将参考此文件并继续使用相同的版本。
仅当你要求 Cargo 升级时它才会升级到更新版本，方法是手动增加 Cargo.toml 文件中的版本号或运行 cargo update

cargo update 只会升级到与你在 Cargo.toml 中指定的内容兼容的最新版本。
如果你指定了 image = "0.6.1"，并且想要升级到版本 0.10.0，则必须自己在 Cargo.toml 中进行更改。

如果此==项目是可执行==文件，那你就应该==将 Cargo.lock 提交到版本控制==

如果你的项目是一个==普通的 Rust 库==，请==不要费心提交== Cargo.lock。
你的库的下游用户拥有包含其整个依赖图版本信息的 Cargo.lock 文件，他们将忽略这个库的 Cargo.lock 文件。
在极少数情况下，你的项目是一个共享库（比如输出是 .dll 文件、.dylib 文件或 .so 文件），它没有这样的下游 cargo 用户，这时候就应该提交Cargo.lock。


## 8.8 将 crate 发布到 crates.i

`cargo package`

如果你的库要发布在 crates.io 上，那么它的依赖项也应该在 crates.io 上。应该指定版本号而不是 path
可以同时指定一个 path（供你自己在本地构建时优先使用）和一个 version（供所有其他用户使用）：
`image = { path = "vendor/image", version = "0.13.0" }`

crates.io 上有了账户
`cargo login 5j0dV54BjlXBpUUbfIj7G9DvNl1vsWW1`
`cargo publish`


## 8.9 工作空间
```rust
fernsoft/
├── .git/...
├── fern_sim/
│   ├── Cargo.toml
│   ├── Cargo.lock
│   ├── src/...
│   └── target/...
├── fern_img/
│   ├── Cargo.toml
│   ├── Cargo.lock
│   ├── src/...
│   └── target/...
└── fern_video/
    ├── Cargo.toml
    ├── Cargo.lock
    ├── src/...
    └── target/...
```

Cargo 的工作方式是，每个 crate 都有自己的构建目录 target，其中包含该 crate 的所有依赖项的单独构建。这些构建目录是完全独立的


你可以使用 Cargo 工作空间来节省编译时间和磁盘空间。
Cargo 工作空间是一组 crate，它们共享着公共构建目录和 Cargo.lock 文件。


你需要做的就是在存储库的根目录中创建一个 Cargo.toml 文件，并将下面这两行代码放入其中
```rust
[workspace]
members = ["fern_sim", "fern_img", "fern_video"]
```
这里的 fern_sim 等是那些包含你的 crate 的子目录名。这些子目录中所有残存的 Cargo.lock 文件和 target 目录都需要删除。

完成此操作后，任何 crate 中的 cargo build 都会自动在根目录（在本例中为 fernsoft/target）下创建和使用共享构建目录。
命令cargo build --workspace 会构建当前工作空间中的所有crate。
cargo test 和 cargo doc 也能接受 --workspace 选项



8.10

当你在 crates.io 上发布一个开源 crate 时，你的文档会自动渲染并托管在 docs.rs 上，这要归功于 Onur Aslan。


如果你的项目在 GitHub 上，那么 Travis CI 可以在每次推送时构建和测试你的代码。
设置起来非常容易，有关详细信息，请参阅 travis-ci.org。
如果你已经熟悉 Travis，则可以从下面这个.travis.yml 文件开始。
```rust
language: rust
rust:
  - stable
```



# ch09 结构体

Rust 有 3 种结构体类型：
- 具名字段型结构体、
- 元组型结构体
- 单元型结构体。

具名字段型结构体会为每个组件==命名==；
元组型结构体会按组件==出现的顺序==标识它们；
单元型结构体则根本==没有组件==。单元型结构体虽然不常见，但它们比你想象的更有用。


## 9.1 具名字段型结构体

```rust
/// 由8位灰度像素组成的矩形
struct GrayscaleMap {
  pixels: Vec<u8>,
  size: (usize, usize)
}
```

Rust 中的约定是，所有类型（包括结构体）的名称都将每个单词的第一个字母大写（如GrayscaleMap），这称为
大驼峰格式（CamelCase 或 PascalCase）。
字段和方法是小写的，单词之间用下划线分隔，这称为 蛇形格式（snake_case）。


使用结构体表达式构造出此类型的值
```rust
let width = 1024;
let height = 576;
let image = GrayscaleMap {
  pixels: vec![0; width * height],
  size: (width, height)
};
```

从 与字段同名 的 局部变量或参数 填充字段的简写形式
```rust
fn new_map(size: (usize, usize), pixels: Vec<u8>) -> GrayscaleMap
{
  // assert_eq!(pixels.len(), size.0 * size.1);
  GrayscaleMap { pixels, size }
}
```

你可以对某些字段使用 key: value 语法，而对同一结构体表达式中的其他字段使用简写语法

结构体默认情况下是私有的，仅在声明它们的模块及其子模块中可见

结构体的定义前加上 pub
每个字段也可以加 pub (pub 结构的 字段依然是私有的，所以 必须加)
```rust
/// 由8位灰度像素组成的矩形
pub struct GrayscaleMap {
  pub pixels: Vec<u8>,
  pub size: (usize, usize)
}
```

创建具名字段结构体的值时，可以使用另一个相同类型的结构体为省略的那些字段提供值。
在结构体表达式中，如果具名字段后面跟着 `.. EXPR`，则任何未提及的字段都会从 EXPR（必须是相同结构体类型的另一个值）中获取它们的值。


```rust
let mut broom1 = Broom { height: b.height / 2, .. b };
// 主要从`broom1`初始化`broom2`。由于`String`不是`Copy`类型，
// 因此我们显式克隆了`name`
let mut broom2 = Broom { name: broom1.name.clone(), .. broom1 };
```
2个独立的对象


## 9.2 元组型结构体

定义
`struct Bounds(usize, usize);`

新建
`let image_bounds = Bounds(1024, 768);`
这看起来像一个方法调用， 实际上确实是方法调用，在定义这个结构体类型的时候，也隐式定义了一个函数
`fn Bounds(elem0: usize, elem1: usize) -> Bounds { ... }`

使用
`assert_eq!(image_bounds.0 * image_bounds.1, 786432);`

pub
`pub struct Bounds(pub usize, pub usize);`


元组型结构体适用于创造 新类型（newtype），即建立一个只包含 单组件的结构体，以获得更严格的类型检查
`struct Ascii(Vec<u8>);`

将此类型用于 ASCII 字符串比简单地传递 `Vec<u8>` 缓冲区并在注释中解释它们的内容要好得多。



## 9.3 单元型结构体
声明了一个根本没有元素的结构体类型
`struct Onesuch;`

这种类型的值不占用内存，很像单元类型 ()

Rust 既不会在内存中实际存储单元型结构体的值，也不会生成代码来对它们进行操作，因为仅通过值的类型它就能知道关于值的所有信息

空结构体是一种只有一个值的类型
`let o = Onesuch;`



## 9.4 结构体布局

与 C 和 C++ 不同，Rust 没有具体承诺它将如何在内存中对结构体的字段或元素进行排序

Rust确实承诺会将字段的值直接存储在结构体本身的内存块中。
。`Vec<u8>` 是 Vec 指针 + 长度 在栈中， 具体的元素在堆中。


你可以使用 #[repr(C)] 属性要求 Rust 以兼容 C 和 C++ 的方式对结构体进行布局



## 9.5 用 impl 定义方法

impl 块只是 fn 定义的集合，每个定义都会成为块顶部命名的结构体类型上的一个方法。

```rust
/// 字符的先入先出队列
pub struct Queue {
  older: Vec<char>,
  younger: Vec<char>
}
// 较旧的元素，最早进来的在后面
// 较新的元素，最后进来的在后面
impl Queue {
  /// 把字符推入队列的最后
  pub fn push(&mut self, c: char) {
    self.younger.push(c);
  }
  /// 从队列的前面弹出一个字符。如果确实有要弹出的字符，
  /// 就返回`Some(c)`；如果队列为空，则返回`None`
  pub fn pop(&mut self) -> Option<char> {
    if self.older.is_empty() {
      if self.younger.is_empty() {
        return None;
      }
      // 将younger中的元素移到older中，并按照所承诺的顺序排列它们
      use std::mem::swap;
      swap(&mut self.older, &mut self.younger);
      self.older.reverse();
    }
    // 现在older能保证有值了。Vec的pop方法已经
    // 返回一个Option，所以可以放心使用了
    self.older.pop()
  }
}
```

在 impl 块中定义的函数称为 关联函数，因为它们和 特定类型相关联的。
与关联函数相对的是 自由函数，是 未定义在 impl 块中的语法项。


Rust 会将调用关联函数的==结构体值==作为==第一个参数==传给方法，该参数必须具有特殊名称 ==self==。
由于 self 的类型显然就是在 impl 块顶部命名的类型或对该类型的引用，因此 Rust 允许你==省略类型==，并以 `self、&self 或 &mut self` 作为 `self: Queue、self:&Queue 或 self: &mut Queue` 的简写形式。
如果你愿意，也可以使用完整形式，但如前所述，几乎所有 Rust 代码都会使用简写形式。

在 C++ 和 Java 中，"this" 对象的成员可以在方法主体中直接可见，不用加 this. 限定符

Rust 方法中则必须显式使用 self 来引用调用此方法的结构体值

修改self，就使用 &mut self
不修改，就 &self
获得所有权，就 self

有时，像这样通过值或引用获取 self 还是不够的，因此 Rust 还允许通过智能指针类型传递 self。

### 9.5.1 以 Box、Rc 或 Arc 形式传入 self

方法的 self 参数也可以是 `Box<Self>` 类型、`Rc<Self>` 类型或 `Arc<Self>` 类型

这种方法只能在给定的指针类型值上调用。调用该方法会将 指针的所有权 传给它。

&self 和 &mut self 几乎总是 指针类型的 上位替代。
如果某些方法确实需要获取指向 Self 的指针的所有权，并且其调用者手头恰好有这样一个指针，那么使用 指针形式的方法
为此，你必须明确写出 self 的类型，就好像它是普通参数一样

```rust
impl Node {
  fn append_to(self: Rc<Self>, parent: &mut Node) {
    parent.children.push(self);
  }
}
```


### 9.5.2 类型关联函数

给定类型的 impl 块还可以定义根本不以 self 为参数的函数。

这些函数仍然是关联函数，因为它们在 impl 块中，但它们不是方法，因为它们==不接受 self 参数==。
为了将它们与方法区分开来，我们称其为 类型关联函数。

通常用于提供构造函数
```rust
impl Queue {
  pub fn new() -> Queue {
    Queue { older: Vec::new(), younger: Vec::new() }
  }
}
```

要使用此函数，需要写成 Queue::new，即==类型名称 + 双冒号 + 函数名称==
`let mut q = Queue::new();`


一个类型可以有许多独立的 impl 块，但它们必须都在定义该类型的同一个 crate 中

Rust 确实允许你将自己的方法附加到其他类型中，第 11 章会解释具体做法。



## 9.6 关联常量

有些值是与类型而不是该类型的特定实例关联起来的。在 Rust中，这些叫作 关联常量。

关联常量是常量值
通常用于表示指定类型下的常用值

```rust
pub struct Vector2 {
  x: f32,
  y: f32,
}
impl Vector2 {
  const ZERO: Vector2 = Vector2 { x: 0.0, y: 0.0 };
  const UNIT: Vector2 = Vector2 { x: 1.0, y: 0.0 };
}
```
这些值是和类型本身相关联的，你可以在不必引用 Vector2 的任一实例的情况下使用它们。

```rust
let scaled = Vector2::UNIT.scaled_by(2.0);
```

关联常量的类型不必是其所关联的类型，我们可以使用此特性为类型添加 ID 或名称。
```rust
impl Vector2 {
  const NAME: &'static str = "Vector2";
  const ID: u32 = 18;
}
```

## 9.7 泛型结构体

```rust
pub struct Queue<T> {
  older: Vec<T>,
  younger: Vec<T>
}
```
`对于任意元素类型 T，Queue<T> 有两个 Vec<T> 类型的字段。`

```rust
impl<T> Queue<T> {
  pub fn new() -> Queue<T> {
    Queue { older: Vec::new(), younger: Vec::new() }  // Queue 不需要 <T>, rust 可以推导
  }
  pub fn push(&mut self, t: T) {
    self.younger.push(t);
  }
  pub fn is_empty(&self) -> bool {
    self.older.is_empty() && self.younger.is_empty()
  }
  // ...
}
```

每个 impl 块，无论是不是泛型，都会将特殊类型的参数 Self（注意这里是大驼峰 CamelCase）定义为我们要为其添加方法的任意类型。
对前面的代码来说，Self 就应该是 `Queue<T>`，因此我们可以进一步缩写 Queue::new 的定义

```rust
pub fn new() -> Self {
  Queue { older: Vec::new(), younger: Vec::new() }   // Queue 也可以用 Self 代替
}
```

`::<>`显式提供类型参数
`let mut q = Queue::<char>::new();`


## 9.8 带生命周期参数的泛型结构体

如果结构体类型包含引用，则必须为这些引用的生命周期命名
```rust
struct Extrema<'elt> {
  greatest: &'elt i32,
  least: &'elt i32
}
```

```rust
fn find_extrema<'s>(slice: &'s [i32]) -> Extrema<'s> {
  let mut greatest = &slice[0];
  let mut least = &slice[0];

  for i in 1..slice.len() {
    if slice[i] < *least { least = &slice[i]; }
    if slice[i] > *greatest { greatest = &slice[i]; }
  }

  Extrema { greatest, least }
}
```

返回类型的生命周期与参数的生命周期相同是很常见的情况, 所以 如果有一个显而易见的候选者，那么 Rust 就允许我们省略生命周期
```rust
fn find_extrema(slice: &[i32]) -> Extrema {
  ...
}
```

## 9.9 带常量参数的泛型结构体

泛型结构体也可以接受常量值作为参数

```rust
/// N - 1次多项式
struct Polynomial<const N: usize> {
  /// 多项式的系数
  ///
  /// 对于多项式a + bx + cx2 + ... + zxn-1，其第`i`个元素是xi的系数
  coefficients: [f64; N]
}
```

根据这个定义，`Polynomial<3>` 是一个二次多项式


Rust 通常也能为常量参数推断出正确的值

`let sine_poly = Polynomial::new([0.0, 1.0, 0.0, -1.0/6.0, 0.0, 1.0/120.0]);`

由于我们向 Polynomial::new 传递了一个包含 6 个元素的数组，因此 Rust 知道必须构造出一个 Polynomial<6>。

常量泛型参数可以是任意整数类型、char 或 bool。
不允许使用浮点数、枚举和其他类型。

生命周期参数必须排在第一位，然后是类型，接下来是任何 const 值。
```rust
struct LumpOfReferences<'a, T, const N: usize> {
  the_lump: [&'a T; N]
}
```


下面会报错，不能计算，只能单独使用。
```rust
struct Polynomial<const N: usize> {
  coefficients: [f64; N + 1]
}
```
cannot perform const operation using `N`


如果要为 const 泛型参数提供的值不仅仅是字面量或单个标识符，那么就必须将其括在花括号中，就像 `Polynomial<{5 + 1}>` 这样



## 9.10 让结构体类型派生自某些公共特型


对于这些标准特型和其他一些特型，无须手动实现，除非你想要某种自定义行为。
Rust 可以自动为你实现它们，而且结果准确无误。
只需将 `#[derive]` 属性添加到结构体上即可

```rust
#[derive(Copy, Clone, Debug, PartialEq)]
struct Point {
  x: f64,
  y: f64
}
```

这些特型中的每一个都可以为结构体自动实现特型，但==前提==是结构体的==每个字段都实现了该特型==。


第 13 章会详细描述 Rust 的标准特型并解释哪些可用于 `#[derive]`。


## 9.11 内部可变性 Cell/RefCell

我们需要一个不可变值（SpiderRobot 结构体）中的一丁点儿可变数据（一个 File）。这称为 内部可变性

Rust 提供了多种可选方案，本节将讨论两种最直观的类型，即
`Cell<T> 和 RefCell<T>`，它们都在 std::cell 模块中

`Cell<T>` 是一个包含类型 T 的单个私有值的结构体。
Cell ==唯一的特殊之处==在于，即使你对 Cell 本身没有 mut 访问权限，也可以==获取和设置==这个==私有值字段==。

- Cell::new(value)（新建）
- cell.get()（获取）
- cell.set(value)（设置）
  此方法接受一个不可变引用型的 self。
  `fn set(&self, value: T) // 注意：不是 &mut self`
其他方法，看文档



```rust
use std::cell::Cell;
pub struct SpiderRobot {
  // ...
  hardware_error_count: Cell<u32>,
  // ...
}
```
非 mut 方法也可以使用 .get() 方法和 .set() 方法访问 u32
```rust
impl SpiderRobot {
  /// 把错误计数递增1
  pub fn add_hardware_error(&self) {
    let n = self.hardware_error_count.get();
    self.hardware_error_count.set(n + 1);
  }
  /// 如果报告过任何硬件错误，则为true
  pub fn has_hardware_errors(&self) -> bool {
    self.hardware_error_count.get() > 0
  }
}
```


Cell ==不允许==在共享值上调用 mut 方法。
.get() 方法会返回 Cell 中==值的副本==，因此它==仅在 T 实现了 Copy== 特型时才有效。
对于日志记录，我们需要一个可变的 File，但 File 不是 Copy 类型。

在这种情况下，正确的工具是 RefCell。
与 `Cell<T>` 一样，`RefCell<T>` 也是一种泛型类型，它包含类型 T 的单个值。
但与Cell 不同，RefCell ==支持借用对其 T 值的引用==。

- RefCell::new(value)（新建）
- ref_cell.borrow()（借用）
  返回一个 `Ref<T>`，它本质上只是对存储在 ref_cell 中值的共享引用。
  如果该值已经以可变借用 借出，则panic
- ref_cell.borrow_mut()（可变借用）
  返回一个 `RefMut<T>`，它本质上是对 ref_cell 中值的可变引用。
  如果该值已经 借出，则panic
- ref_cell.try_borrow()（尝试借用）
- ref_cell.try_borrow_mut()（尝试可变借用）
  行为与 borrow() 和 borrow_mut() 一样，但会返回一个Result。
  如果该值已经 借出，则Err，而不是 panic

RefCell 也有一些其他的方法，你可以在其文档中进行查找

当你借用一个变量的引用时，Rust 会 编译期 进行检查
RefCell 会使用 运行期 检查强制执行相同的规则


Cell 以及包含它的任意类型都==不是线程安全==的。因此 Rust 不允许多个线程同时访问它们
。。RefCell 呢？应该也是。


# 第 10 章 枚举与模式

Rust 枚举还可以包含数据，甚至是不同类型的数据

Rust 的 `Result<String, io::Error>` 类型就是一个枚举，这样的值要么是包含 String 型的 Ok 值，要么是包含io::Error 的 Err 值。

Rust 枚举更像是 C 的联合体，但不同之处在于它是类型安全的。

只要值可能代表多种事物，枚举就很有用
使用枚举的“代价”是你必须通过模式匹配安全地访问数据

## 10.1 枚举

Rust 中简单的 C 风格枚举很直观
```rust
enum Ordering {
  Less,
  Equal,
  Greater,    // 这 3个值 称为 变体 或 构造器
}
```

```rust
use std::cmp::Ordering;
fn compare(n: i32, m: i32) -> Ordering {
  if n < m {
    Ordering::Less
  } else if n > m {
    Ordering::Greater
  } else {
    Ordering::Equal
  }
}
```
上面的风格更好

```rust
use std::cmp::Ordering::{self, *}; // `*`导入所有子项
fn compare(n: i32, m: i32) -> Ordering {
  if n < m {
    Less
  } else if n > m {
    Greater
  } else {
    Equal
  }
}
```
self : 导入当前模块中声明的枚举的构造器


有时告诉 Rust 要使用 哪几个整数是很有用的

```rust
enum HttpStatus {
  Ok = 200,
  NotModified = 304,
  NotFound = 404,
  ...
}
```

默认，Rust 会从 0 开始帮你分配数值。


默认情况下，Rust 会使用可以容纳它们的最小内置整数类型来存储 C 风格枚举。最适合的是==单字节==

向枚举添加 `#[repr]` 属性来覆盖 Rust 对内存中表示法的默认选择

可以将 C 风格枚举转换为整数
从整数到枚举的反向转换则行不通。

你可以编写自己的“检查完再转换”逻辑
```rust
fn http_status_from_u32(n: u32) -> Option<HttpStatus> {
  match n {
    200 => Some(HttpStatus::Ok),
    304 => Some(HttpStatus::NotModified),
    404 => Some(HttpStatus::NotFound),
    // ...
    _ => None,
  }
}
```
或者借助 enum_primitive crate。它包含一个宏，可以帮你自动生成这类转换代码

与结构体一样，编译器能为你实现 == 运算符等特性，但你必须明确提出要求
```rust
#[derive(Copy, Clone, Debug, PartialEq, Eq)]
enum TimeUnit {
  Seconds, Minutes, Hours, Days, Months, Years,
}
```
枚举可以有方法，就像结构体一样：
```rust
impl TimeUnit {
/// 返回此时间单位的复数名词
fn plural(self) -> &'static str {
    match self {
      TimeUnit::Seconds => "seconds",
      TimeUnit::Minutes => "minutes",
      TimeUnit::Hours => "hours",
      TimeUnit::Days => "days",
      TimeUnit::Months => "months",
      TimeUnit::Years => "years",
    }
  }
  /// 返回此时间单位的单数名词
  fn singular(self) -> &'static str {
    self.plural().trim_end_matches('s')
  }
}
```

---

10.1.1 带数据的枚举

```rust
/// 刻意四舍五入后的时间戳，所以程序会显示“6个月前”
/// 而非“2016年2月9日上午9点49分”
#[derive(Copy, Clone, Debug, PartialEq)]
enum RoughTime {
  InThePast(TimeUnit, u32),
  JustNow,
  InTheFuture(TimeUnit, u32),
}
```

此枚举中的两个变体 InThePast 和 InTheFuture 能接受参数。
这种变体叫作 元组型变体 。

```rust
let four_score_and_seven_years_ago = RoughTime::InThePast(TimeUnit::Years, 4 * 20 + 7);
let three_hours_from_now = RoughTime::InTheFuture(TimeUnit::Hours, 3);
```


枚举还可以有 结构体型变体

```rust
enum Shape {
  Sphere { center: Point3d, radius: f32 },
  Cuboid { corner1: Point3d, corner2: Point3d },
}
let unit_sphere = Shape::Sphere {
  center: ORIGIN,
  radius: 1.0,
};
```

总而言之，Rust 有 3 种枚举变体，这与我们在第 9 章中展示的 3 种结构体相呼应。
- 没有数据的变体对应于单元型结构体。
- 元组型变体的外观和功能很像元组型结构体。
- 结构体型变体具有花括号和具名字段

单个枚举中可以同时有 3 种类型的变体
```rust
enum RelationshipStatus {
  Single,
  InARelationship,
  ItsComplicated(Option<String>),
  ItsExtremelyComplicated {
    car: DifferentialEquation,
    cdr: EarlyModernistPoem,
  },
}
```
枚举的所有构造器和字段都与枚举本身具有相同的可见性


---

10.1.2 内存中的枚举

在内存中，带有数据的枚举会以一个小型整数 标签 加上足以容纳最大变体中所有字段的内存块的格式进行存储。

。以最大变体为准，  c的 union


---

10.1.3 用枚举表示富数据结构



### 10.1.4 泛型枚举

```rust
enum Option<T> {
  None,
  Some(T),
}

enum Result<T, E> {
  Ok(T),
  Err(E),
}
```

一个不太明显的细节是，当类型 T 是引用、Box 或其他智能指针类型时，Rust 可以省掉 `Option<T>` 的标签字段。
由于这些指针类型都不允许为 0，因此 Rust 可以将 `Option<Box<i32>>` 表示为单个机器字：0 表示 None，非零表示 Some 指针。
这能让 Option 类型的值尽量接近于可能为空的 C 或 C++ 指针。
不同之处在于 Rust的类型系统要求你在使用其内容之前检查 Option 是否为 Some。这有效地避免了对空指针解引用

。。这是rust的，和我们无关 。
。。不过，我们如果确定是 Box，引用，智能指针，那么 可以就 不使用 Option 吧。 不，无法表示 None 。 rust内部可以 用0 表示 None，但是 我们没有办法。


```rust
// `T`组成的有序集合
enum BinaryTree<T> {
  Empty,
  NonEmpty(Box<TreeNode<T>>),
}
// BinaryTree的部件
struct TreeNode<T> {
  element: T,
  left: BinaryTree<T>,
  right: BinaryTree<T>,
}
```

```rust
use self::BinaryTree::*;
let jupiter_tree = NonEmpty(Box::new(TreeNode {
  element: "Jupiter",
  left: Empty,
  right: Empty,
}));

let mars_tree = NonEmpty(Box::new(TreeNode {
  element: "Mars",
  left: jupiter_tree,
  right: mercury_tree,
}));

let tree = NonEmpty(Box::new(TreeNode {
  element: "Saturn",
  left: mars_tree,
  right: uranus_tree,
}));
```



只能用一种安全的方式来访问枚举中的数据，即使用模式

## 10.2 模式

```rust
enum RoughTime {
  InThePast(TimeUnit, u32),
  JustNow,
  InTheFuture(TimeUnit, u32),
}
```

Rust 不允许你通过编写 rough_time.0 和 rough_time.1 来直接访问它们，因为毕竟rough_time 也可能是没有字段的

需要一个 match 表达式 获得数据

```rust
fn rough_time_to_english(rt: RoughTime) -> String {
  match rt {
    RoughTime::InThePast(units, count) => format!("{} {} ago", count, units.plural()),
    RoughTime::JustNow => format!("just now"),
    RoughTime::InTheFuture(units, count) => format!("{} {} from now", count, units.plural()),
  }
}
```

模式 就是第 3 行、第 5 行和 第 7 行中出现在 => 符号前面的部分

模式会 消耗 值

添加一个分支来处理 单数
`RoughTime::InTheFuture(unit, 1) => format!("a {} from now", unit.singular()),`
仅当 count 字段恰好为 1 时，才会匹配此分支。
这行新代码必须添加到第 7 行之前。 否则 Rust 编译器将警告发现了 "unreachable pattern"（无法抵达的模式）。


### 模式的所有类型

|模式类型|例子|注意事项|
|--|--|--|
|字面量|100, <br/>"name"|匹配一个确切的值；也允许匹配常量名称|
|范围|0..=10, <br/>'a'..='k', <br/>256..|匹配范围内的任何值，包括可能给定的结束值|
|通配符|_|匹配任何值并忽略它|
|变量|name, <br/>mut count|类似于 _，但会把值移动或复制到新的局部变量中|
|引用变量|ref field, <br/>ref mut field|借用对匹配值的引用，而不是移动或复制它|
|与子模式绑定|val @ 0..99, <br/>ref circle @, <br/>Shape::Circle {..}|使用 @ 左边的变量名，匹配其右边的模式|
|枚举型模式|Some(value), <br/>None, <br/>Pet::Orca||
|元组型模式|(key, value), <br/>(r, g, b)||
|数组型模式|[a,b,c], <br/>[heading,carom,correction]||
|切片型模式|[first,second], <br/>[first,_,third], <br/>[first,..,nth], <br/>[ ]||
|结构体型模式|Color(r,g,b), <br/>Point{x,y}, <br/>Card{suit:Clubs,rank:n}, <br/>Account{id,name,..}||
|引用|&value, <br/>&{k, v}|仅匹配引用值|
|多个模式|'a'|'A', <br/>Some("left"|"right")||
|守卫表达式|x if x*x<=r2|只用在 match 表达式中（不能用在let 语句等处）|



10.2.1 模式中的字面量、变量和通配符

类似 C 语言的 switch 语句

针对整数值的 match

```rust
match meadow.count_rabbits() {
  0 => {} // 无话可说
  1 => println!("A rabbit is nosing around in the clover."),
  n => println!("There are {} rabbits hopping about in the meadow", n),
}
```

其他字面量也可以用作模式，包括布尔值、字符，甚至字符串

```rust
let calendar = match settings.get_string("calendar") {
  "gregorian" => Calendar::Gregorian,
  "chinese" => Calendar::Chinese,
  "ethiopian" => Calendar::Ethiopian,
  other => return parse_error("calendar", other),
};
```
other 就像上个例子中的 n 一样充当了包罗万象的模式

这些模式与 switch 语句中的 default 分支起着相同的作用

`_` 匹配任意值，但不关心匹配到的值

```rust
let caption = match photo.tagged_pet() {
  Pet::Tyrannosaur => "RRRAAAAAHHHHHH",
  Pet::Samoyed => "*dog thoughts*",
  _ => "I'm cute, love me", // 一般性捕获，对任意Pet都生效
};
```

Rust 要求每个 match 表达式都必须处理所有可能的值，因此最后往往需要一个通配符模式。
即使你非常确定其他情况不会发生，也必须至少添加一个后备分支，也许是 panic 的分支。
```rust
// 有很多种形状（Shape），但我们只支持“选中”一些文本框
// 或者矩形区域中的所有内容。不能选择椭圆或梯形
match document.selection() {
  Shape::TextSpan(start, end) => paint_text_selection(start, end),
  Shape::Rectangle(rect) => paint_rect_selection(rect), _ => panic!("unexpected selection type"),
}
```

10.2.2 元组型模式与结构体型模式

元组型模式匹配元组。每当你想要在==单次 match 中获取多条数据==时，元组型模式都非常有用

```rust
fn describe_point(x: i32, y: i32) -> &'static str {
  use std::cmp::Ordering::*;
  match (x.cmp(&0), y.cmp(&0)) {    // 多条数据
    (Equal, Equal) => "at the origin",
    (_, Equal) => "on the x axis",
    (Equal, _) => "on the y axis",
    (Greater, Greater) => "in the first quadrant",
    (Less, Greater) => "in the second quadrant",
    _ => "somewhere else",
  }
}
```


结构体型模式使用花括号，就像结构体表达式一样。结构体型模式包含每个字段的子模式

```rust
match balloon.location {
  Point { x: 0, y: height } => println!("straight up {} meters", height),
  Point { x: x, y: y } => println!("at ({}m, {}m)", x, y),
}
```

`Point { x: x, y: y }` 有一个简写形式：`Point {x, y}`

当我们只关心几个字段时，匹配大型结构体仍然很麻烦
可以使用 .. 告诉 Rust 你不关心任何其他字段。
```rust
Some(Account { name, language, .. }) => language.show_custom_greeting(name),
```



10.2.3 数组型模式与切片型模式


数组型模式通常用于过滤一些特殊情况的值，并且在处理那些不同位置的值具有不同含义的数组时也非常有用。

将 HSL（色相、饱和度和亮度）颜色值转换为 RGB（红色、绿色和蓝色）颜色值
具有零亮度或全亮度的颜色只会是黑色或白色。
可以使用 match 表达式来简单地处理这些情况
```rust
fn hsl_to_rgb(hsl: [u8; 3]) -> [u8; 3] {
  match hsl {
    [_, _, 0] => [0, 0, 0],
    [_, _, 255] => [255, 255, 255],
    ...
  }
}
```

切片型模式也与此相似，但与数组不同，切片具有可变长度，因此切片型模式不仅匹配值，还匹配长度。
.. 在切片型模式中能匹配任意数量的元素。

```rust
fn greet_people(names: &[&str]) {
  match names {
    [] => { println!("Hello, nobody.") },
    [a] => { println!("Hello, {}.", a) },
    [a, b] => { println!("Hello, {} and {}.", a, b) },
    [a, .., b] => { println!("Hello, everyone from {} to {}.", a, b) }
  }
}
```


10.2.4 引用型模式

Rust 模式提供了两种特性来支持引用。
- ref 模式会借用已匹配值的一部分。
- & 模式会匹配引用。

匹配不可复制的值会移动该值。以下代码是无效的
```rust
match account {
  Account { name, language, .. } => {
    ui.greet(&name, &language);
    ui.show_settings(&account); // 错误：借用已移动的值`account`
  }
}
```

如果 name 和 language 都是可复制的值，则 Rust 会复制字段而非移动它们，这时上述代码就是有效的。


需要 借用 而非移动匹配值的模式。ref 关键字就是这样做的

```rust
match account {
Account { ref name, ref language, .. } => {
    ui.greet(name, language);
    ui.show_settings(&account); // 正确
  }
}
```
现在局部变量 name 和 language 是对 account 中相应字段的引用。
由于 account 只是被借入而没有被消耗，因此继续调用它的方法是没问题的

使用 ref mut 来借入可变引用
```rust
match line_result {
  Err(ref err) => log_error(err),
  Ok(ref mut line) => {
    trim_comments(line);
    handle(line);
  }
}
```
模式 Ok(ref mut line) 能匹配任何成功的结果，并借入其成功值的可变引用

---

与 ref 模式相对 的引用型模式是 & 模式。以 & 开头的模式会==匹配引用==

```rust
match sphere.center() {
  &Point3d { x, y, z } => ...
}
```
sphere.center() 会返回对 sphere 中的私有字段的引用，这是 Rust 中的常见模式。
返回的值是 Point3d 的地址。

Rust 在这里会追踪一个指针，我们通常会将追踪指针的操作与 * 运算符而不是 & 运算符联系起来。
但要记住，==模式和表达式是恰恰相反的==。
==表达式 (x, y) 会把两个值放入一个新的元组中==，而==模式 (x, y) 则会匹配一个元组并分解成两个值==。
& 的逻辑也是如此。在==表达式中，& 会创建一个引用==。在==模式中，& 则会匹配一个引用==。


匹配引用时会遵循我们所期望的一切规则。生命周期规则仍然有效。
你不能通过共享引用获得可变访问权限，而且不能将值从引用中移动出去，即使对可变引用也是如此。

如果试图在具有不可复制字段的结构体上这么做，就会出错
你可以使用 ref 模式来借用对部件的引用，但并不拥有它

```rust
match chars.peek() {
  Some(&c) => println!("coming up: {:?}", c),
  None => println!("end of chars"),
}
```

10.2.5 匹配守卫

匹配分支会有一些==额外的条件==，必须满足这些条件才能视为匹配成功

简单地添加 if
```rust
match point_to_hex(click) {
  None => Err("That's not a game space."),
  Some(hex) => {
    if hex == current_hex {
      Err("You are already there! You must click somewhere else")
    } else {
      Ok(hex)
    }
  }
}
```

匹配守卫，分支增加 if
```rust
match point_to_hex(click) {
  None => Err("That's not a game space."),
  Some(hex) if hex == current_hex => Err("You are already there! You must click somewhere else"),
  Some(hex) => Ok(hex)
}
```



10.2.6 匹配多种可能性

```rust
let at_end = match chars.peek() {
  Some(&'\r' | &'\n') | None => true,
  _ => false,
};
```

```rust
match next_char {
  '0'..='9' => self.read_number(),
  'a'..='z' | 'A'..='Z' => self.read_word(),
  ' ' | '\t' | '\n' => self.skip_whitespace(),
  _ => self.handle_punctuation(),
}
```

还允许使用像 x.. 这样的范围型模式，该模式会匹配从 x 到其类型最大值的任何值

目前模式中还==不允许==使用其他的开区间范围（如 0..100 或 ..100）以及无限范围（如 ..）
。。    ..=100 呢？


---

10.2.7 使用@模式绑定

`x @ pattern` 会与给定的 pattern 精确匹配，但成功时，它不会为匹配到的值的各个部分创建变量，而是会创建单个变量 x 并将整个值==移动或复制==到其中

```rust
match self.get_selection() {
  Shape::Rect(top_left, bottom_right) => {
    optimized_paint(&Shape::Rect(top_left, bottom_right))
  }
  other_shape => {
    paint_outline(other_shape.get_outline())
  }
}
```
第一个分支解包出一个 Shape::Rect 值，却只是为了在下一行重建一个相同的 Shape::Rect 值。
像这种代码可以用 @ 模式重写
```rust
  rect @ Shape::Rect(..) => {
    optimized_paint(&rect)
  }
```

@ 模式对于各种范围模式也很有用。

```rust
match chars.next() {
  Some(digit @ '0'..='9') => read_number(digit, chars),
  ...
},
```


### 10.2.8 模式能用在哪里

Rust 不是要将值存储到单个变量中，而是使用模式匹配来==拆分值==。

```rust
// 把结构体解包成3个局部变量……
let Track { album, track_number, title, .. } = song;

// ……解包某个作为函数参数传入的元组
fn distance_to((x, y): (f64, f64)) -> f64 { ... }

// ……迭代某个HashMap上的键和值
for (id, document) in &cache_map {
  println!("Document #{}: {}", id, document.title);
}

// ……自动对闭包参数解引用（当其他代码给你传入引用，
// 而你更想要一个副本时会很有用）
let sum = numbers.fold(0, |a, &num| a + num);
```


模式 `Point3d{ x, y, z }` 会匹配 Point3d 结构体类型的每个可能值，`(x, y)` 会匹配任何一个 (f64, f64) 值对，等等。
这种==始终都可以匹配==的模式在 Rust 中是很特殊的，它们叫作==不可反驳模式==

==可反驳模式==是一种==可能不会匹配==的模式，比如 Ok(x) 不会匹配错误结果，而 '0' ..= '9' 不会匹配字符 'Q'。

可反驳模式可以用在match 的分支中，因为 match 就是为此而设计的


if let 表达式和 while let 表达式中也允许使用可反驳模式，这些模式可用于

```rust
// ……处理只有一个枚举值的特例
if let RoughTime::InTheFuture(_, _) = user.date_of_birth() {
  user.set_time_traveler(true);
}

// ……只有当查表成功时才运行某些代码
if let Some(document) = cache_map.get(&id) {
  return send_cached_response(document);
}

// ……重复尝试某些事，直到成功
while let Err(err) = present_cheesy_anti_robot_task() {
  log_robot_attempt(err); // 让用户再试一次（此用户仍然可能是人类）
}

// ……在某个迭代器上手动循环
while let Some(_) = lines.peek() {
  read_paragraph(&mut lines);
}
```



10.2.9 填充二叉树


## 10.3 大局观

编程就是数据处理。




# ch11 特型与泛型 403


编程领域的伟大发现之一，就是可以编写能对许多不同类型（甚至是尚未发明的类型）的值进行操作的代码。
- `Vec<T>` 是泛型的
- 许多类型有 .write() 方法
这就是所谓的 多态性

Rust 通过两个相关联的特性来支持多态：特型和泛型

特型是 Rust 体系中的接口或抽象基类
泛型是 Rust 中多态的另一种形式。与 C++ 模板一样，泛型函数或泛型类型可以和不同类型的值一起使用



## 11.1 使用特型

特型代表着一种能力，即一个类型能做什么

特型方法有一条值得注意的规则：特型本身必须在作用域内。否则，它的所有方法都是不可见的
。。就是要 use 将 特型 导入进来。 避免 命名冲突。


Clone 和 Iterator 的各个方法在没有任何特殊导入的情况下就能工作，因为默认情况下它们始终在作用域中：它们是==标准库预导入==的一部分，Rust 会把这些名称自动导入每个模块中。

特型方法类似于虚方法。不过，特型方法的调用仍然很快，与任何其他方法调用一样快。

只有通过 &mut dyn Write 调用时才会产生==动态派发==（也叫虚方法调用）的开销


11.1.1 特型对象

使用特型编写多态代码有两种方法：特型对象和泛型。

Rust 不允许 dyn Write 类型的变量, 变量的大小必须是编译期已知的，而那些实现了 Write 的类型可以是任意大小

需要使用 引用
```rust
let mut buf: Vec<u8> = vec![];
let writer: &mut dyn Write = &mut buf; // 正确
```
对特型类型（如 writer）的引用叫作特型对象。

与任何其他引用一样，特型对象指向某个值，它具有生命周期，并且可以是可变或共享的。

不同之处在于，Rust 通常 无法 在 编译期 间知道 引用目标的类型。

Rust 也不支持从特型对象 &mut dyn Write 向下转型回像 `Vec<u8>` 这样的具体类型

在内存中，特型对象是一个胖指针，由指向==值的指针==和指向表示==该值类型的虚表的指针==组成。

虚表只会在编译期生成一次，并由同一类型的所有对象共享

当你调用特型对象的方法时，该语言会自动使用虚表来确定要调用哪个实现

在 C++ 中，虚表指针或 vptr 是作为结构体的一部分存储的，而 Rust 使用的是胖指针方案。
结构体本身只包含自己的字段。这样一来，每个结构体就可以实现几十个特型而不必包含几十个 vptr了。
。。？C++ 也是 类中 增加一个 指针吧？

Rust 在需要时会自动将普通引用转换为特型对象。
Rust 会愉快地将 `Box<File>` 转换为 `Box<dyn Write>`
`let w: Box<dyn Write> = Box::new(local_file);`

和 &mut dyn Write 一样，`Box<dyn Write>` 也是一个胖指针，即包含写入器本身的地址和虚表的地址。其他指针类型（如 `Rc<dyn Write>`）同样如此。

这种转换是创建特型对象的唯一方法。
编译器在这里真正做的事非常简单。
在发生转换的地方，Rust 知道引用目标的真实类型（在本例中为 File），因此它只要加上适当的虚表的地址，把常规指针变成胖指针就可以了。

---

11.1.2 泛型函数与类型参数

```rust
fn say_hello<W: Write>(out: &mut W) -> std::io::Result<()> {
  out.write_all(b"hello world\n")?;
  out.flush()
}
```

```rust
fn say_hello(out: &mut dyn Write) // 普通函数
fn say_hello<W: Write>(out: &mut W) // 泛型函数
```

短语 `<W: Write>` 把函数变成了泛型形式。此短语叫作类型参数

```rust
say_hello(&mut local_file)?; // 调用say_hello::<File>
say_hello(&mut bytes)?; // 调用say_hello::<Vec<u8>>

say_hello::<File>(&mut local_file)?;
```

```rust
// 调用无参数的泛型方法collect<C>()
let v1 = (0 .. 1000).collect(); // 错误：无法推断类型
let v2 = (0 .. 1000).collect::<Vec<i32>>(); // 正确
```


使用+号语法

```rust
use std::hash::Hash;
use std::fmt::Debug;
fn top_ten<T: Debug + Hash + Eq>(values: &Vec<T>) { ... }
```

```rust
/// 在一个大型且分区的数据集上运行查询
/// 参见<http://research.google.com/archive/mapreduce.html>
fn run_query<M: Mapper + Serialize, R: Reducer + Serialize>(data: &DataSet, map: M, reduce: R) -> Results { ... }
```

Rust 使用关键字 where 提供了另一种语法
```rust
fn run_query<M, R>(data: &DataSet, map: M, reduce: R) -> Results
 where M: Mapper + Serialize, R: Reducer + Serialize
{ ... }
```

这种 where 子句也允许用于泛型结构体、枚举、类型别名和方法——任何允许使用限界的地方。

泛型函数可以同时具有生命周期参数和类型参数。生命周期参数要排在前面
```rust
fn nearest<'t, 'c, P>(target: &'t P, candidates: &'c [P]) -> &'c P
 where P: MeasureDistance
```

除了类型和生命周期，泛型函数也可以接受常量参数
```rust
fn dot_product<const N: usize>(a: [f64; N], b: [f64; N]) -> f64 {
  let mut sum = 0.;
  for i in 0..N {
  sum += a[i] * b[i];
  }
  sum
}
```

```rust
// 显式提供`3`作为`N`的值
dot_product::<3>([0.2, 0.4, 0.6], [0., 0., 1.])
// 让Rust推断`N`必然是`2`
dot_product([3., 4.], [-5., 1.])
```

单独的方法也可以是泛型的，即使它并没有定义在泛型类型上
```rust
impl PancakeStack {
  fn push<T: Topping>(&mut self, goop: T) -> PancakeResult<()> {
    goop.pour(&self);
    self.absorb_topping(goop)
  }
}
```


类型别名也可以是泛型的
`type PancakeResult<T> = Result<T, PancakeError>;`


11.1.3 使用哪一个

当你需要一些混合类型值的集合时，特型对象是正确的选择

```rust
struct Salad {
  veggies: Vec<Box<dyn Vegetable>>
}
```

使用特型对象的另一个原因可能是想减少编译后代码的总大小

Rust 可能会不得不多次编译泛型函数，针对用到了它的每种类型各编译一次。
而这可能会使二进制文件变大，这种现象在 C++ 圈子里叫作 代码膨胀。

与特型对象相比，泛型具有 3 个重要优势，因此在 Rust 中，泛型是更常见的选择

第一个优势是速度
泛型函数签名中缺少 dyn 关键字。因为你要在 编译期 指定类型
Rust 直到 运行期 才能知道特型对象指向什么类型的值

第二个优势在于并不是每个特型都能支持特型对象。
特型支持的几个特性（如关联函数）只适用于泛型：它们完全不支持特型对象。

泛型的第三个优势是它很容易同时指定具有多个特型的泛型参数限界，就像我们的 top_ten 函数要求它的 T 参数必须实现 Debug + Hash + Eq 那样。
特型对象不能这样做：Rust 不支持像 &mut (dyn Debug + Hash + Eq) 这样的类型。


## 11.2 定义与实现特型

==定义特型== 很简单，给它一个名字并列出 特型方法的类型签名 即可。

```rust
/// 角色、道具和风景的特型——游戏世界中可在屏幕上看见的任何东西
trait Visible {
  /// 在给定的画布上渲染此对象
  fn draw(&self, canvas: &mut Canvas);
  /// 如果单击(x, y)时应该选中此对象，就返回true
  fn hit_test(&self, x: i32, y: i32) -> bool;
}
```

要==实现特型==，请使用语法 `impl TraitName for Type`

```rust
impl Visible for Broom {
  fn draw(&self, canvas: &mut Canvas) {
    for y in self.y - self.height - 1 .. self.y {
      canvas.write_at(self.x, y, '|');
    }
    canvas.write_at(self.x, self.y, 'M');
  }
  fn hit_test(&self, x: i32, y: i32) -> bool {
    self.x == x
    && self.y - self.height - 1 <= y
    && y <= self.y
  }
}
```
这个 impl 包含 Visible 特型中每个方法的实现，再无其他。


11.2.1 默认方法

标准库的 Write 特型定义中包含了对 write_all 的 默认实现：
```rust
trait Write {
  fn write(&mut self, buf: &[u8]) -> Result<usize>;
  fn flush(&mut self) -> Result<()>;
  fn write_all(&mut self, buf: &[u8]) -> Result<()> {
    let mut bytes_written = 0;
    while bytes_written < buf.len() {
      bytes_written += self.write(&buf[bytes_written..])?;
    }
    Ok(())
  }
  ...
}
```

标准库中对默认方法最引人注目的应用场景是 Iterator 特型，它有一个必要方法 (.next()) 和几十个默认方法


11.2.2 特型与其他人的类型

Rust 允许在任意类型上实现任意特型，但==特型或类型二者必须至少有一个是在当前 crate 中新建的==。

这意味着任何时候如果你想为任意类型添加一个方法，都可以使用特型来完成

这个特殊特型的唯一目的是向现有类型 char 中添加一个方法。这称为 扩展特型。


甚至可以使用一个泛型的 impl 块来一次性向整个类型家族添加扩展特型。这个特型可以在任意类型上实现
```rust
use std::io::{self, Write};
/// 能让你把HTML写入值里的特型
trait WriteHtml {
  fn write_html(&mut self, html: &HtmlDocument) -> io::Result<()>;
}
```

为所有写入器实现特型会令其成为扩展特型，比如为所有 Rust 写入器添加一个方法

```rust
/// 可以把HTML写入任何一个std::io writer
impl<W: Write> WriteHtml for W {
  fn write_html(&mut self, html: &HtmlDocument) -> io::Result<()> {
    ...
  }
}
```

serde 库提供了一个很好的例子，表明在标准类型上实现用户定义的特型非常有用。
serde 是一个序列化库。也就是说，你可以使用serde 将 Rust 数据结构写入磁盘，稍后再重新加载它们。



11.2.3 特型中的 Self
特型可以用关键字 Self 作为类型。

```rust
pub trait Clone {
  fn clone(&self) -> Self;
  ...
}
```
以 Self 作为返回类型意味着 x.clone() 的类型与 x 的类型相同，无论 x 是什么。


```rust
pub trait Spliceable {
  fn splice(&self, other: &Self) -> Self;
}

impl Spliceable for CherryTree {
  fn splice(&self, other: &Self) -> Self {
    ...
  }
}

impl Spliceable for Mammoth {
  fn splice(&self, other: &Self) -> Self {
    ...
  }
}
```



```rust
// 错误：特型`Spliceable`不能用作特型对象
fn splice_anything(left: &dyn Spliceable, right: &dyn Spliceable)
{
  let combo = left.splice(right);   // splice 要求 left 和 right 类型相同，但是现在是 dyn，所以g
  // ...
}
```
Rust 会拒绝此代码，因为它无法对 left.splice(right) 这个调用进行 类型检查。
特型对象 的全部意义恰恰在于其类型要到 运行期才能知道。
Rust 在编译期无从了解 left 和 right 是否为同一类型。


11.2.4 子特型

声明一个特型是另一个特型的扩展

。。继承？ no

```rust
/// 游戏世界中的生物，既可以是玩家，也可以是
/// 其他小精灵、石像鬼、松鼠、食人魔等
trait Creature: Visible {
  fn position(&self) -> (i32, i32);
  fn facing(&self) -> Direction;
  ...
}
```

每个实现了 Creature 的类型也必须实现 Visible 特型


在 Rust 中，子特型==不会继承==其超特型的关联项，如果你想调用超特型的方法，那么仍然要保证每个特型都在作用域内。

Rust 的子特型只是对 Self 类型限界的简写。像下面这样的 Creature 定义与前面的定义完全等效

```rust
trait Creature where Self: Visible {
}
```


11.2.5 类型关联函数

在大多数面向对象语言中，接口不能包含静态方法或构造函数，但特型可以包含类型关联函数，这是 Rust 对静态方法的模拟

```rust
trait StringSet {
  /// 返回一个新建的空集合
  fn new() -> Self;
  /// 返回一个包含`strings`中所有字符串的集合
  fn from_slice(strings: &[&str]) -> Self;
  /// 判断这个集合中是否包含特定的`string`
  fn contains(&self, string: &str) -> bool;
  /// 把一个字符串添加到此集合中
  fn add(&mut self, string: &str);
}
```
。。这也算构造器？ 只是一个 约定俗成的 工厂方法 吧。

```rust
/// 返回`document`中不存在于`wordlist`中的单词集合
fn unknown_words<S: StringSet>(document: &[String], wordlist: &S) -> S {
  let mut unknowns = S::new();
  ...
}
```

特型对象不支持类型关联函数

如果想使用 &dyn StringSet 特型对象，就必须修改此特型，为每个未通过引用接受 self 参数的关联函数加上类型限界 where Self:Sized：

```rust
trait StringSet {
  fn new() -> Self where Self: Sized;
  fn from_slice(strings: &[&str]) -> Self where Self: Sized;
  fn contains(&self, string: &str) -> bool;
  fn add(&mut self, string: &str);
}
```

这个限界告诉 Rust，特型对象不需要支持特定的关联函数 。通过添加这些限界，就能把 StringSet 作为特型对象使用了

虽然特型对象仍不支持关联函数 new 或 from_slice，但你还是可以创建它们并用其调用 .contains() 和 .add()。

。。？


## 11.3 完全限定的方法调用

迄今为止，我们看到的所有调用特型方法的方式都依赖于 Rust 为你补齐了一些缺失的部分

`"hello".to_string()`

Rust 知道 to_string 指的是 ToString 特型的 to_string 方法（我们称之为 str 类型的实现）。
所以这个游戏里有 4 个“玩家”：特型、特型的方法、方法的实现以及调用该实现时传入的值。

有时，你需要一种方式来准确表达你的意思。完全限定的方法调用符合此要求。

方法只是一种特殊的函数。下面两个调用是等效的
```rust
"hello".to_string()
str::to_string("hello")
```
第二种形式看起来很像关联函数调用。
尽管 to_string 方法需要一个 self 参数，但是仍然可以像关联函数一样调用。
只需将 self作为此函数的第一个参数传进去即可。


由于 to_string 是标准 ToString 特型的方法之一，因此你还可以使用另外两种形式：
```rust
ToString::to_string("hello")
<str as ToString>::to_string("hello")
```

所有这 4 种方法调用都会做同样的事情。
大多数情况下，只要写value.method() 就可以了。
其他形式都是==限定方法调用==。它们要指定方法所 关联的类型==或==特型。
最后一种带有尖括号的形式，同时指定了两者，这就是==完全限定的方法调用==


通过完全限定的调用，你可以准确地指出是哪一个方法，这在一些奇怪的情况下会有所帮助。
- 当两个方法具有相同的名称时
```rust
outlaw.draw(); // 错误：画（draw）在屏幕上还是拔出（draw）手枪？
Visible::draw(&outlaw); // 正确：画在屏幕上
HasPistol::draw(&outlaw); // 正确：拔出手枪
```
- 当无法推断 self 参数的类型时
```rust
let zero = 0; // 类型未指定：可能为`i8`、`u8`……
zero.abs(); // 错误：无法在有歧义的数值类型上调用方法`abs`
i64::abs(zero); // 正确
```
- 将函数本身用作函数类型的值时。
```rust
let words: Vec<String> =
  line.split_whitespace() // 迭代器生成&str值
    .map(ToString::to_string) // 正确
    .collect();
```
- 在宏中调用特型方法时



完全限定语法也适用于关联函数


## 11.4 定义类型之间关系的特型

迄今为止，我们看到的每个特型都是独立的：特型是类型可以实现的一组方法。

特型也可以用于==多种类型必须协同工作==的场景中。它们可以描述多个类型之间的关系。
- std::iter::Iterator 特型会为每个迭代器类型与其生成的值的类型建立联系。


11.4.1 关联类型（或迭代器的工作原理）

```rust
pub trait Iterator {
  type Item;
  fn next(&mut self) -> Option<Self::Item>;
  ...
}
```

这个特型的第一个特性（type Item;）是一个 关联类型。
实现了Iterator 的每种类型都必须指定它所生成的条目的类型

第二个特性（next() 方法）在其返回值中使用了关联类型。


### 实现 Iterator
```rust
//（来自标准库中std::env模块的代码）
impl Iterator for Args {
  type Item = String;     // -------------
  fn next(&mut self) -> Option<String> {
    ...
  }
  ...
}
```

泛型代码可以使用关联类型
```rust
/// 遍历迭代器，将值存储在新向量中
fn collect_into_vector<I: Iterator>(iter: I) -> Vec<I::Item> {
  let mut results = Vec::new();
  for value in iter {
    results.push(value);
  }
  results
}
```
在这个函数体中，Rust 为我们推断出了 value 的类型，这固然不错，但我们还必须明确写出 collect_into_vector 的返回类型，而 Item 关联类型是唯一的途径。


```rust
/// 打印出迭代器生成的所有值
fn dump<I>(iter: I) where I: Iterator
{
  for (index, value) in iter.enumerate() {
    println!("{}: {:?}", index, value); // 错误
  }
}
```
value 不一定是可打印的类型

`fn dump<I>(iter: I) where I: Iterator, I::Item: Debug`
或者
`fn dump<I>(iter: I) where I: Iterator<Item=String>`

```rust
fn dump(iter: &mut dyn Iterator<Item=String>) {
for (index, s) in iter.enumerate() {
  println!("{}: {:?}", index, s);
}
}
```

当特型需要包含的不仅仅是方法的时候，关联类型会很有用
关联类型非常适合每个实现都有一个特定相关类型的情况

- 在线程池库中，Task 特型表示一个工作单元，它可以包含一个关联的 Output 类型
- Pattern 特型表示一种搜索字符串的方式，它可以包含一个关联的 Match 类型
- 用于处理关系型数据库的库可能具有 DatabaseConnection特型，其关联类型表示事务、游标、已准备语句等


11.4.2 泛型特型（或运算符重载的工作原理）

Rust 中的乘法使用了以下特型

```rust
/// std::ops::Mul，用于标记支持`*`（乘号）的类型的特型
pub trait Mul<RHS> {
  /// 在应用了`*`运算符后的结果类型
  type Output;
  /// 实现`*`运算符的方法
  fn mul(self, rhs: RHS) -> Self::Output;
}
```

Mul 是一个泛型特型。类型参数 RHS 是右操作数（right-hand side）的缩写。

泛型特型在涉及孤儿规则时会得到特殊豁免：你可以==为外部类型实现外部特型==，只要特型的==类型参数之一是当前 crate 中定义的类型==即可。

真正的 Mul 特型是这样的
`pub trait Mul<RHS=Self> {`

语法 RHS=Self 表示 RHS 默认为 Self。如果我写下 `impl Mul for Complex`，而不指定 Mul 的类型参数，则表示 `impl Mul<Complex> for Complex`。在类型限界中，如果我写下 `where T: Mul`，则表示 `where T: Mul<T>`。


11.4.3 impl Trait

由许多泛型类型组合而成的结果可能会极其凌乱

```rust
use std::iter;
use std::vec::IntoIter;
fn cyclical_zip(v: Vec<u8>, u: Vec<u8>) -> iter::Cycle<iter::Chain<IntoIter<u8>, IntoIter<u8>>> {
  v.into_iter().chain(u.into_iter()).cycle()
}
```

可以很容易地用特型对象替换这个“丑陋的”返回类型
```rust
fn cyclical_zip(v: Vec<u8>, u: Vec<u8>) -> Box<dyn Iterator<Item=u8>> {
  Box::new(v.into_iter().chain(u.into_iter()).cycle())
}
```
然而，在大多数情况下，如果仅仅是为了避免“丑陋的”类型签名，就要在每次调用这个函数时承受动态派发和不可避免的堆分配开销，可不太划算。


Rust 有一个名为 impl Trait 的特性，该特性正是为应对这种情况而设计的。
impl Trait 允许我们“擦除”返回值的类型，仅指定它实现的一个或多个特型，而无须进行动态派发或堆分配

```rust
fn cyclical_zip(v: Vec<u8>, u: Vec<u8>) -> impl Iterator<Item=u8> {
  v.into_iter().chain(u.into_iter()).cycle()
}
```

Rust 不允许特型方法使用 impl Trait 作为返回值。
只有自由函数和关联具体类型的函数才能使用 impl Trait 作为返回值。

impl Trait 也可以用在带有泛型参数的函数中
```rust
fn print<T: Display>(val: T) {
  println!("{}", val);
}

fn print(val: impl Display) {
  println!("{}", val);
}
```

但有一个重要的例外。
使用泛型时允许函数的调用者指定泛型参数的类型，比如 `print::<i32>(42)`，而如果使用 impl Trait 则不能这样做。


### 11.4.4 关联常量

特型也可以有关联常量

```rust
trait Greet {
  const GREETING: &'static str = "Hello";
  fn greet(&self) -> String;
}
```

关联常量在特型中具有特殊的功能。与关联类型和函数一样，你也可以声明它们，但不为其定义值
```rust
trait Float {
  const ZERO: Self;
  const ONE: Self;
}
```

特型的实现者可以定义这些值
```rust
impl Float for f32 {
  const ZERO: f32 = 0.0;
  const ONE: f32 = 1.0;
}
impl Float for f64 {
  const ZERO: f64 = 0.0;
  const ONE: f64 = 1.0;
}
```

使用这些值的泛型代码
```rust
fn add_one<T: Float + Add<Output=T>>(value: T) -> T {
  value + T::ONE
}
```
关联常量不能与特型对象一起使用，因为为了在编译期选择正确的值，编译器会依赖相关实现的类型信息。

即使是没有任何行为的简单特型（如 Float），也可以提供有关类型的足够信息，再结合一些运算符，以实现像斐波那契数列这样常见的数学函数

```rust
fn fib<T: Float + Add<Output=T>>(n: usize) -> T {
  match n {
    0 => T::ZERO,
    1 => T::ONE,
    n => fib::<T>(n - 1) + fib::<T>(n - 2)
  }
}
```


## 11.5 逆向工程求限界

编写泛型代码可能会是一件真正的苦差事

。。一个例子，编译器报错 - 修改 - 报错 - 修改 。。。


这个过程有点儿痛苦，因为标准库中没有那么一个 Number 特型包含我们想要使用的所有运算符和方法。
碰巧的是，有一个名为 num 的流行开源 crate 定义了这样的一个特型。
如果我们能提前知道，就可以将 num 添加到 Cargo.toml


明确写出限界的最重要的优点是限界就这么写在代码和文档中。

在像 Boost 这样的 C++ 库中完整记录参数类型的工作比我们在这里经历的还要艰巨得多


## 11.6 以特型为基础

特型成为 Rust 中最主要的组织特性之一是有充分理由的，因为良好的接口在设计程序或库时尤为重要。





# ch12 运算符重载 449

可以让自己的类型支持算术运算符和其他运算符，只要实现一些内置特型即可。
这叫作运算符重载，其效果跟 C++、C#、Python 和Ruby 中的运算符重载很相似。


|类别|特型|运算符|
|--|--|--|
|一元运算符|std::ops::Neg/Not|`-x, !x`|
|算术运算符|std::ops::Add/Sub/Mul/Div/Rem|`x+y, -, *, /, %`|
|按位运算符|std::ops::BitAnd/BitOr/BitXor/Shl/Shr|`&, |, ^, <<, >>`|
|复合赋值算术运算符|std::ops::AddAssign/SubAssign/<br/>MulAssign/DivAssign/RemAssign|`+=, -=, *=, /=, %=`|
|复合赋值按位运算符|std::ops::BitAndAssign/BitOrAssign/<br/>BitXorAssign/ShlAssign/ShrAssign|`&=, |=, ^=, <<=, >>=`|
|比较|std::cmp::PartialEq/PartialOrd|`==, !=, <, <=, >, >=`|
|索引运算符|std::ops::Index/IndexMut|`x[y], &x[y], x[y]=z, &mut x[y]`|


## 12.1 算术运算符与按位运算符

表达式 a + b 实际上是 a.add(b) 的简写形式
是对标准库中 std::ops::Add 特型的 add 方法的调用

如果试图写出 z.add(c)，就要将 Add 特型引入作用域，以便它的方法在此可见

```rust
use std::ops::Add;
assert_eq!(4.125f32.add(5.75), 9.875);
assert_eq!(10.add(20), 10 + 20);
```

std::ops::Add 的定义
```rust
trait Add<Rhs = Self> {
  type Output;
  fn add(self, rhs: Rhs) -> Self::Output;
}
```

```rust
use std::ops::Add;
impl Add for Complex<i32> {
  type Output = Complex<i32>;
  fn add(self, rhs: Self) -> Self {
    Complex {
      re: self.re + rhs.re,
      im: self.im + rhs.im,
    }
  }
}
```

不必为 `Complex<i32>`、`Complex<f32>`、`Complex<f64>` 等逐个实现 Add。
除了所涉及的类型不一样，所有定义看起来都完全相同，因此我们可以写一个涵盖所有这些定义的泛型实现，只要复数的各个组件本身的类型都支持加法就可以

```rust
use std::ops::Add;
impl<T> Add for Complex<T> where T: Add<Output = T>, {
  type Output = Self;
  fn add(self, rhs: Self) -> Self {
    Complex {
      re: self.re + rhs.re,
      im: self.im + rhs.im,
    }
  }
}
```

通过编写 `where T: Add<Output=T>`，我们将 T 限界到能与自身相加并产生另一个 T 值的类型。


还可以将条件进一步放宽，因为 Add 特型不要求 + 的两个操作数具有相同的类型，也不限制结果类型。
因此，一个尽可能泛化的实现可以让左右操作数独立变化，并生成加法所能生成的任何组件类型的 Complex 值：

```rust
use std::ops::Add;
impl<L, R> Add<Complex<R>> for Complex<L> where  L: Add<R>, {
  type Output = Complex<L::Output>;
  fn add(self, rhs: Complex<R>) -> Self::Output {
    Complex {
      re: self.re + rhs.re,
      im: self.im + rhs.im,
    }
  }
}
```


。。跳。




## 12.5 其他运算符

并非所有运算符都可以在 Rust 中重载。

错误检查运算符 ? 仅适用于 Result 值和 Option 值

逻辑运算符 && 和 || 仅限于 bool 值

.. 运算符和 ..= 运算符总会创建一个表示范围边界的结构体，

& 运算符总是会借用引用，

= 运算符总是会移动值或复制值。

它们都不能重载。


解引用运算符 *val 和用于访问字段和调用方法的点运算符（如val.field 和 val.method()）可以用 Deref 特型和DerefMut 特型进行重载，这将在第 13 章中介绍。

Rust 不支持重载函数调用运算符 f(x)。当你需要一个可调用的值时，通常只需编写一个闭包即可


# ch13 实用工具特型 473

Rust 实用工具特型可分为三大类。
- 语言扩展特型
  Drop、Deref 和 DerefMut, From 和 Into
- 标记特型
  多用作泛型类型变量的限界
  Sized 和 Copy
- 公共词汇特型
  Default、AsRef、AsMut、Borrow 与 BorrowMut、TryFrom 与 TryInto， ToOwned


|特型|描述|
|--|--|
|Drop|析构器。丢弃值时，rust调用该特型的方法|
|Sized|标记类型为 编译器已知的固定大小类型，相对的是 动态大小类型(如切片)|
|Clone|该类型支持克隆值|
|Copy|可以简单地通过对内存进行逐字节复制|
|Deref, DerefMut|智能指针类型|
|Default|合理的"默认值"|
|AsRef, AsMut|用于从另一种类型中借入一种引用类型的转换特型|
|Borrow, BorrowMut|转换特型，类似AsRef, AsMut，但额外保证 一致的哈希，排序，相等性|
|From, Into|将一种类型的值转为另一种类型|
|TryFrom, TryInto|将一种类型的值转为另一种类型，用于可能失败的转换|
|ToOwned|用于将引用转换为拥有型值的转换特型|

还有另一些重要的标准库特型。
第 15 章会介绍 Iterator 和 IntoIterator。
第 16 章会介绍用于计算哈希值的 Hash 特型。
第 19 章会介绍两个用于标记线程安全类型的特型 Send 和 Sync。


## 13.1 Drop

在大多数情况下，Rust 会自动处理丢弃值的工作。
假设你定义了以下类型
```rust
struct Appellation {
  name: String,
  nicknames: Vec<String>
}
```

Appellation 拥有用作字符串内容和向量元素缓冲区的堆存储。
每当 Appellation 被丢弃时，Rust 都会负责清理所有这些内容，无须你进行任何进一步的编码。
但只要你想，也可以通过实现 `std::ops::Drop` 特型来自定义 Rust 该如何丢弃此类型的值：
```rust
trait Drop {
  fn drop(&mut self);
}
```

当一个值被丢弃时，如果它实现了 std::ops::Drop，那么 Rust 就会调用它的 drop 方法，然后像往常一样继续丢弃它的字段或元素拥有的任何值。
这种对 drop 的隐式调用是调用该方法的唯一途径。
如果你试图显式调用该方法，那么 Rust 会将其标记为错误。

Rust 在丢弃某个值的字段或元素之前会先对值本身调用Drop::drop，该方法收到的值仍然是已完全初始化的。
因此，在Appellation 类型的 Drop 实现中可以随意使用其字段
```rust
impl Drop for Appellation {
  fn drop(&mut self) {
    print!("Dropping {}", self.name);
    if !self.nicknames.is_empty() {
      print!(" (AKA {})", self.nicknames.join(", "));
    }
    println!("");
  }
}
```

如果一个类型实现了 Drop，就不能再实现 Copy 特型了。
如果类型是 Copy 类型，就表示简单的逐字节复制足以生成该值的独立副本。
但是，对同一份数据多次调用同一个 drop 方法显然是错误的。

标准库预导入中包含一个丢弃值的函数 drop，但它的定义一点儿也不神奇：
`fn drop<T>(_x: T) { }`





。。跳

`#[derive(Clone)]`
不需要自己实现


# ch14 闭包 512

```rust
/// 按照人口数量对城市进行排序的辅助函数
fn city_population_descending(city: &City) -> i64 {
  -city.population
}
fn sort_cities(cities: &mut Vec<City>) {
  cities.sort_by_key(city_population_descending); // 正确
}
```

将辅助函数写成闭包（匿名函数表达式）则会更简洁
```rust
fn sort_cities(cities: &mut Vec<City>) {
  cities.sort_by_key(|city| -city.population);
}
```

标准库中接受闭包的其他例子。
- 像 map 和 filter 这样的 Iterator 方法，可用于处理序列数据
- 像 thread::spawn 这样的线程 API，会启动一个新的系统线程。并发就是要将工作转移给其他线程，而闭包能方便地表示这些工作单元
- 一些需要根据条件计算默认值的方法，比如 HashMap 条目的or_insert_with 方法。

学习如何将闭包与标准库方法一起使用、闭包如何“捕获”其作用域内的变量、如何编写自己的以闭包作为参数的函数和方法，以及如何存储闭包供以后用作回调。
我们还将解释 Rust 闭包是如何实现的，以及它们为什么比你预想的要快



## 14.1 捕获变量

闭包可以使用属于其所在函数的数据

### 14.1.1 借用值的闭包

```rust
/// 根据任何其他的统计标准排序
fn sort_by_statistic(cities: &mut Vec<City>, stat: Statistic) {
  cities.sort_by_key(|city| -city.get_statistic(stat));
}
```

当 Rust 创建闭包时，会自动借入对 stat 的引用。
这很合理，因为闭包引用了 stat，所以闭包必须包含对 stat 的引用

闭包同样遵循第 5 章中讲过的关于借用和生命周期的规则。
特别是，由于闭包中包含对 stat 的引用，因此 Rust 不会让它的生命周期超出 stat。
因为闭包只会在排序期间使用，所以这个例子是适用的。

简而言之，Rust 会使用生命周期而非垃圾回收来确保安全。


### 14.1.2 move“窃取”值的闭包

```rust
use std::thread;
fn start_sorting_thread(mut cities: Vec<City>, stat: Statistic) -> thread::JoinHandle<Vec<City>> {
  let key_fn = |city: &City| -> i64 { -city.get_statistic(stat) };
  thread::spawn(|| {
    cities.sort_by_key(key_fn);
    cities
  })
}
```

|| 是闭包的空参数列表。

新线程会和调用者并行运行。当闭包返回时，新线程退出。
闭包的返回值会作为 JoinHandle 值发送回调用线程。第 19 章会介绍JoinHandle。

闭包 key_fn 包含对 stat 的引用
Rust 不能保证此引用的安全使用。因此 Rust 会拒绝编译这个程序

这里还有一个问题，因为 cities 也被不安全地共享了。
thread::spawn 创建的新线程无法保证在 cities 和 stat 被销毁之前在函数末尾完成其工作。

这两个问题的解决方案是一样的：要求 Rust 将 cities 和 stat移动到使用它们的闭包中，而不是借入对它们的引用

```rust
fn start_sorting_thread(mut cities: Vec<City>, stat: Statistic) -> thread::JoinHandle<Vec<City>> {
  let key_fn = move |city: &City| -> i64 { -city.get_statistic(stat) };
  thread::spawn(move || {
    cities.sort_by_key(key_fn);
    cities
  })
}
```
唯一的改动是在两个闭包之前都添加了 ==move 关键字==
move 关键字会告诉 Rust，闭包并不是要借入它用到的变量，而是要“窃取”它们。

第一个闭包 key_fn 取得了 stat 的所有权。
第二个闭包则取得了cities 和 key_fn 的所有权。


Rust 为闭包提供了==两种==从封闭作用域中获取数据的方法：==移动和借用==。

如果闭包要移动可复制类型的值（如 i32），那么就会复制该值。

即使我们确实需要在此之后使用 cities，解决方法也很简单：可以要求 Rust 克隆 cities 并将副本存储在另一个变量中。

通过遵循 Rust 的严格规则，我们也收获颇丰，那就是线程安全。



## 14.2 函数与闭包的类型 Fn > fn

函数和闭包都在被当作值使用。自然，这就意味着它们有自己的类型。

```rust
fn city_population_descending(city: &City) -> i64 {
  -city.population
}
```
该函数会接受一个参数（&City）并返回 i64。所以它的类型是 `fn(&City) -> i64`。

你可以像对其他值一样对函数执行各种操作。

```rust
let my_key_fn: fn(&City) -> i64 =
  if user.prefs.by_population {
    city_population_descending
  } else {
    city_monster_attack_risk_descending
  };

cities.sort_by_key(my_key_fn);
```

结构体也可以有函数类型的字段。
像 Vec 这样的泛型类型可以存储大量的函数，只要它们共享同一个 fn 类型即可。
而且函数值占用的空间很小，因为 fn 值就是函数机器码的内存地址，就像 C++ 中的函数指针一样

一个函数还可以将另一个函数作为参数

```rust
/// 给定一份城市列表和一个测试函数，返回有多少个城市通过了测试
fn count_selected_cities(cities: &Vec<City>, test_fn: fn(&City) -> bool) -> usize {
  let mut count = 0;
  for city in cities {
    if test_fn(city) {
      count += 1;
    }
  }
  count
}
/// 测试函数的示例。注意，此函数的类型是`fn(&City) -> bool`，
/// 与`count_selected_cities` 的 `test_fn`参数相同
fn has_monster_attacks(city: &City) -> bool {
  city.monster_attack_risk > 0.0
}
// 有多少个城市存在被怪兽袭击的风险？
let n = count_selected_cities(&my_cities, has_monster_attacks);
```

闭包与函数不是同一种类型
```rust
let limit = preferences.acceptable_monster_risk();
let n = count_selected_cities(&my_cities, |city| city.monster_attack_risk > limit); // 错误：类型不匹配
```

第二个参数会导致类型错误。
为了支持闭包，必须更改这个函数的类型签名。要改成下面这样
```rust
fn count_selected_cities<F>(cities: &Vec<City>, test_fn: F) -> usize where F: Fn(&City) -> bool {
  let mut count = 0;
  for city in cities {
    if test_fn(city) {
      count += 1;
    }
  }
  count
}
```

。。 Fn, fn

```rust
fn(&City) -> bool // fn类型（只接受函数）
Fn(&City) -> bool // Fn特型（既接受函数也接受闭包）
```
-> 和返回类型是可选的，如果省略，则返回类型为 ()


每个闭包都有自己的类型，因为闭包可以包含数据：从封闭作用域中借用或“窃取”的值。
每个闭包都有一个由编译器创建的特殊类型，大到足以容纳这些数据

因为每个闭包都有自己的类型，所以使用闭包的代码通常都应该是泛型的


## 14.3 闭包性能

Rust 中闭包的设计目标是要快：比函数指针还要快，快到甚至可以在对性能敏感的热点代码中使用它们。

如果你熟悉 C++ 的 lambda 表达式，就会发现 Rust 闭包也一样快速而紧凑，但更安全。

在大多数语言中，闭包会在堆中分配内存、进行动态派发以及进行垃圾回收。
因此，创建、调用和收集每一个闭包都会花费一点点额外的CPU 时间。
更糟的是，闭包往往难以内联，而内联是编译器用来消除函数调用开销并实施大量其他优化的关键技术。
总而言之，闭包在这些语言中确实慢到值得手动将它们从节奏紧凑的内层循环中去掉。

Rust 闭包则没有这些性能缺陷。
它们没有垃圾回收。与 Rust 中的其他所有类型一样，除非你将闭包放在 Box、Vec 或其他容器中，否则它们不会被分配到堆上。
由于每个闭包都有不同的类型，因此 Rust 编译器只要知道你正在调用的闭包的类型，就可以内联该闭包的代码。

14.5 节会展示如何使用特型对象在堆中分配闭包并动态调用它们。
虽然这种方法有点儿慢，但仍然和特型对象的其他方法一样快。


## 14.4 闭包与安全

本节将稍微解释一下当闭包丢弃或修改其捕获的值时会发生什么。

```rust
let my_str = "hello".to_string();
let f = || drop(my_str);
```
。。不是说 drop 不能显式调用吗？ 这个是drop 的方法吗？

```rust
f(); // 正确
f(); // 错误：使用了已移动的值
```

值会被消耗掉（移动）的想法是Rust 的核心概念之一，它对闭包与其他语法一视同仁。



### 14.4.2 FnOnce

第一次调用 FnOnce 闭包时，闭包本身也会被消耗掉。
这是因为 Fn 和 FnOnce 这两个特型是这样定义的
```rust
// 无参数的`Fn`特型和`FnOnce`特型的伪代码
trait Fn() -> R {
  fn call(&self) -> R;    // 引用self
}
trait FnOnce() -> R {
  fn call_once(self) -> R;  // 消耗self
}
```

对于 Fn 闭包，closure() 会扩展为closure.call()。此方法会通过引用获取 self，因此闭包不会被移动。

如果闭包只能安全地调用一次，那么 closure() 就会扩展为 closure.call_once()。
该方法会按值获取 self，因此这个闭包就会被消耗掉。


### 14.4.3 FnMut
包含可变数据或可变引用的闭包

FnMut 闭包会通过可变引用来调用

```rust
// `Fn`特型、`FnMut`特型和`FnOnce`特型的伪代码
trait Fn() -> R {
  fn call(&self) -> R;
}
trait FnMut() -> R {
  fn call_mut(&mut self) -> R;
}
trait FnOnce() -> R {
  fn call_once(self) -> R;
}
```

任何需要对值进行可变访问但不会丢弃任何值的闭包都是 FnMut 闭包。

```rust
let mut i = 0;
let incr = || {
  i += 1; // incr借入了对i的一个可变引用
  println!("Ding! i is now: {}", i);
};
call_twice(incr);
```

Rust 闭包的 3 种类别。
- Fn 是可以不受限制地调用任意多次的闭包和函数系列。此最高类别还包括所有 fn 函数。
- FnMut 是本身会被声明为 mut，并且可以多次调用的闭包系列。
- FnOnce 是如果其调用者拥有此闭包，它就只能调用一次的闭包系列。

每个 Fn 都能满足 FnMut 的要求，每个 FnMut 都能满足 FnOnce 的要求。

Fn() 是 FnMut() 的子特型，而 FnMut() 是 FnOnce() 的子特型。

```rust
fn call_twice<F>(mut closure: F) where F: FnMut() {
  closure();
  closure();
}
```



### 14.4.4 对闭包的 Copy 与 Clone

Rust 也能找出哪些闭包可以实现 Copy 和 Clone，哪些则不可以实现

闭包是表示包含它们捕获的变量的值（对于 move 闭包）或对值的引用（对于非 move 闭包）的结构体。
闭包的Copy 规则和 Clone 规则与常规结构体的规则是一样的

一个不修改变量的非 move 闭包只持有共享引用，这些引用既能 Clone 也能 Copy，所以闭包也能 Clone 和 Copy：
```rust
let y = 10;
let add_y = |x| x + y;
let copy_of_add_y = add_y; // 此闭包能`Copy`，所以……
assert_eq!(add_y(copy_of_add_y(22)), 42); // ……可以调用它两次
```

一个会修改值的非 move 闭包在其内部表示中也可以有可变引用。
可变引用既不能 Clone，也不能 Copy，使用它们的闭包同样如此
```rust
let mut x = 0;
let mut add_to_x = |n| { x += n; x };
let copy_of_add_to_x = add_to_x; // 这会进行移动而非复制
assert_eq!(add_to_x(copy_of_add_to_x(1)), 2); // 错误：使用了已移动出去的值
```

对于 move 闭包，规则更简单。
如果 move 闭包捕获的所有内容都能Copy，那它就能 Copy。
如果 move 闭包捕获的所有内容都能Clone，那它就能 Clone
```rust
let mut greeting = String::from("Hello, ");
let greet = move |name| {
  greeting.push_str(name);
  println!("{}", greeting);
};
greet.clone()("Alfred");
greet.clone()("Bruce");
```


## 14.5 回调

```rust
App::new()
  .route("/", web::get().to(get_index))
  .route("/gcd", web::post().to(post_gcd))
```


## 14.6 高效地使用闭包

MVC 框架都会创建 3 个对象：
- 表示该UI 元素状态的模型、
- 负责其外观的视图和
- 处理用户交互的控制器。

每个对象都会直接或通过回调对其他对象中的一个或两个进行引用

每当 3 个对象中的一个对象发生变化时，它会通知其他两个对象，因此所有内容都会及时更新。哪个对象“拥有”其他对象之类的问题永远不会出现。

如果不进行更改，就==无法在 Rust 中实现此模式==。所有权必须明晰，循环引用也必须消除。模型和控制器不能相互直接引用。



有时你可以通过让每个闭包接受它需要的引用作为参数，来解决闭包所有权和生命周期的问题。
有时你可以为系统中的每个事物分配一个编号，并传递这些编号而不是传递引用。
或者你可以实现 MVC 的众多变体之一，其中的对象并非都相互引用。
或者你可以将工具包建模为具有单向数据流的非 MVC 系统，比如 Facebook 的 ==Flux== 架构

如果你试图使用 Rust 闭包来应对复杂对象关系，就会遇到困难，但还有其他选择。在这种情况下，软件工程这门学科似乎更倾向于使用替代方案，因为它们更简单。






# ch15 迭代器 541

Rust 的 for 循环为使用迭代器提供了一种自然的语法

迭代器本身也提供了一组丰富的方法，比如映射（map）、过滤（filter）、连接（join）、收集（collect）等。


```rust
fn triangle(n: i32) -> i32 {
  let mut sum = 0;
  for i in 1..=n {
    sum += i;
  }
  sum
}
```

```rust
fn triangle(n: i32) -> i32 {
  (1..=n).fold(0, |sum, item| sum + item)
}
```

开始运行时以 0 作为总和，fold 会获取 1..=n 生成的每个值，并以总和（sum）跟当前值（item）为参数调用闭包 |sum, item| sum + item。
闭包的返回值会作为新的总和。它返回的最后一个值就是 fold 自身要返回的值


在本章中，我们将解释以下核心知识点。
- Iterator 特型和 IntoIterator 特型
- 一个典型的迭代器流水线通常有 3 个阶段：
  - 从某种“值源”创建迭代器，
  - 通过选择或处理从中流过的值来将一种迭代器适配成另一种迭代器，
  - 然后消耗此迭代器生成的值。
- 如何为自己的类型实现迭代器



```rust
pub fn iter(&self) -> Iter<'_, T> { }     // 传入 引用。
fn into_iter(self) -> IntoIter<T, A> { }  // 传入 值，所以消耗了。
```


## 15.1 Iterator 特型与 IntoIterator 特型

迭代器是实现了 std::iter::Iterator 特型的任意值：

```rust
trait Iterator {
  type Item;
  fn next(&mut self) -> Option<Self::Item>;
  …… // 很多默认方法
}
```

只要可以用某种自然的方式来迭代某种类型，该类型就可以实现std::iter::IntoIterator，其 into_iter 方法会接受一个值并返回一个迭代器

```rust
trait IntoIterator where Self::IntoIter: Iterator<Item=Self::Item>
{
  type Item;
  type IntoIter: Iterator;
  fn into_iter(self) -> Self::IntoIter;
}
```


IntoIter 的类型 是迭代器本身，而 Item 是它生成的值的类型。
任何实现了 IntoIterator 的类型都称为可迭代者，因为你可以随意迭代它。


Rust 的 for 循环会将所有这些部分很好地结合在一起。
每个 for 循环都只是调用 IntoIterator 和 Iterator 中某些方法的简写形式
```rust
let mut iterator = (&v).into_iter();
while let Some(element) = iterator.next() {
  println!("{}", element);
}
```

for 循环会使用 IntoIterator::into_iter 将其操作数 &v 转换为迭代器，然后重复调用 Iterator::next。

迭代器的一些术语
- **迭代器**是实现了 Iterator 的任意类型。
- **可迭代者**是任何实现了 IntoIterator 的类型
- 迭代器能**生成**值
- 迭代器生成的值是**条目**
- 接收迭代器所生成条目的代码是**消费者**


虽然 for 循环总会在其操作数上调用 into_iter，但你也可以直接把迭代器传给 for 循环


## 15.2 创建迭代器

15.2.1 iter 方法与 iter_mut 方法

大多数集合类型提供了 iter（迭代器）方法和 iter_mut（可变迭代器）方法，它们会返回该类型的自然迭代器，为每个条目生成共享引用或可变引用

像 `&[T]` 和 `&mut [T]` 这样的数组切片也有iter 方法和 iter_mut 方法


如果类型有不止一种常用的遍历方式，该类型通常会为每种遍历方式提供一个专门的方法，因为普通的 iter 方法会产生歧义。

&str 字符串切片类型就没有 iter 方法——如果 s 是 &str，
则 s.bytes() 会返回一个能生成 s 中每字节的迭代器，
而 s.chars() 则会将内容解释为 UTF-8 并生成每个 Unicode 字符。



15.2.2 IntoIterator 的实现

如果一个类型实现了 IntoIterator，你也可以自行调用它的`into_iter` 方法，就像 for 循环一样

大多数集合实际上提供了 IntoIterator 的几种实现，用于共享引用（&T）、可变引用（&mut T）和移动（T）。

- 给定一个集合的共享引用，into_iter 会返回一个迭代器，该迭代器会生成对其条目的共享引用。
- 给定对集合的可变引用，into_iter 会返回一个迭代器，该迭代器会生成对其条目的可变引用
- 当按值传递集合时，into_iter 会返回一个迭代器，该迭代器会获取集合的所有权并按值返回这些条目，这些条目的所有权会从集合转移给消费者，原始集合在此过程中已被消耗掉了。


由于 for 循环会将 IntoIterator::into_iter 作为它的操作对象，因此这 3 种实现创建了以下惯用法，用于迭代
- 对集合的共享引用或
- 可变引用，或者
- 消耗该集合并获取其元素的所有权

```rust
for element in &collection { ... }
for element in &mut collection { ... }
for element in collection { ... }
```

并非每种类型都提供了这 3 种实现，比如，HashSet、BTreeSet和 BinaryHeap 不会在可变引用上实现 IntoIterator，因为修改它们的元素可能会违反类型自身的不变性规则, 修改后的值很可能有不同的哈希值，或者相对于其邻居的顺序改变了


总体原则是，迭代应该是高效且可预测的，因此 Rust 不会提供昂贵或可能表现出意外行为的实现

切片实现了 3 个 IntoIterator 变体中的两个，由于切片并不拥有自己的元素，因此不存在“按值”引用的情况

`&[T] 和 &mut[T]` 各自的 into_iter 会分别返回一个迭代器，该迭代器会生成对其元素的共享引用和可变引用



### 15.2.3 from_fn 与 successors

`std::iter::from_fn`（来自fn）就会返回一个迭代器，该迭代器会调用 fn 来生成条目

```rust
use rand::random; // 在Cargo.toml中添加dependencies: rand = "0.7"
use std::iter::from_fn;
// 产生1000条端点均匀分布在区间[0, 1]上的随机线段的长度（这并不是
// `rand_distr` crate中能找到的分布类型，但你可以轻易实现一个）
let lengths: Vec<f64> = from_fn(|| Some((random::<f64>() - random::<f64>()).abs())).take(1000).collect();
```

如果每个条目都依赖于其前一个条目，那么 `std::iter::successors` 函数很实用。

```rust
use num::Complex;
use std::iter::successors;
fn escape_time(c: Complex<f64>, limit: usize) -> Option<usize> {
  let zero = Complex { re: 0.0, im: 0.0 };
  successors(Some(zero), |&z| { Some(z * z + c) })
    .take(limit)
    .enumerate()
    .find(|(_i, z)| z.norm_sqr() > 4.0)
    .map(|(i, _z)| i)
}
```

从零开始，successors（后继者）调用会通过反复对最后一个点求平方再加上参数 c 来生成复平面上的一系列点。


from_fn 方法和 successors 方法非常灵活，你几乎可以将任何对迭代器的使用改写成对其中之一的调用，通过传递复杂的闭包来得到你想要的行为。
但这样做浪费了迭代器提供的机会，即使用常见模式的标准名称来更清晰地表达计算中的数据流。

使用这两个方法之前，请确保你已经熟悉本章中的其他迭代器方法，==通常其他迭代器是更好的选择==



15.2.4 drain 方法

有许多集合类型提供了 drain（抽取）方法。
drain 会接受一个对集合的可变引用，并返回一个迭代器，该迭代器会将每个元素的所有权传给消费者

然而，与按值获取并消耗掉集合的 into_iter() 方法不同，drain 只会借入对集合的可变引用，当迭代器被丢弃时，它会从集合中移除所有剩余元素以清空集合

对于可以按范围索引的类型（如 String、向量和 VecDeque）， drain 方法可指定要移除的元素范围，而不是“抽干”整个序列

```rust
let mut outer = "Earth".to_string();
let inner = String::from_iter(outer.drain(1..4));
assert_eq!(outer, "Eh");
assert_eq!(inner, "art");
```

rust-std-String.drain:
Removes the specified range from the string in bulk, returning all removed characters as an iterator.



15.2.5 其他迭代器源

|类型或特型|表达式|注意事项|
|--|--|--|
|std::ops::Range|1..10, (1..10).step_by(2)||
|std::ops::RangeFrom|1..||
|std::ops::RangeInclusive|1..=10||
|`Option<T>`|Some(10).iter()||
|`Result<T, E>`|Ok("blah").iter()||
|`Vec<T>, &[T]`|v.windows(16),太多了,chunks,chunks_mut,split,split_mut,rsplit,splitn||
|String, &str|s.bytes(),chars(),split_whitespace(),lines,split('/'),matchs(char::is_numeric)||
|std::collections::HashMap,<br/>std::collections::BTreeMap|map.keys(),map.values(),map.values_mut()||
|std::collections::HashSet,BTreeSet|set1.union(set2),set1.intersection(set2)||
|std::sync::mpsc::Receiver|recv.iter()||
|std::io::Read|stream.bytes(),chars()||
|std::io::BufRead|bufStream.lines(),split(0)||
|std::fs::ReadDir|std::fs::read_dir(path)||
|std::net::TcpListener|listener.incoming||
|自由函数|std::iter::empty(),once(5),repeat("#9")||




## 15.3 迭代器适配器

Iterator 特型 提供了大量适配器方法。 这些适配器方法 消耗 某个迭代器并构建一个 实现了特定行为的新迭代器。

从两个最流行的适配器 map 和 filter 开始

截断、跳过、组合、反转、连接、重复

大多数适配器会按值接受 self，这就要求 Self 必须是固定大小的（Sized）（所有常见的迭代器都是这样的）

关于迭代器适配器，有两点需要特别注意。
- 单纯在迭代器上调用适配器并不会消耗任何条目，只会返回一个新的迭代器，新迭代器会根据需要从第一个迭代器中提取条目，以生成自己的条目。
  。。只是返回迭代器，不执行实际的操作，只有collect 时 才会开始 执行
- 迭代器的适配器是一种零成本抽象。由于 map、filter 和其他类似的适配器都是泛型的，因此将它们应用于迭代器就会专门针对所涉及的特定迭代器类型生成特化代码。这意味着 Rust 会有足够的信息将每个迭代器的 next 方法内联到它的消费者中，然后将这一组功能作为一个单元翻译成机器代码
  。。和 手工遍历 的性能相同。


### 15.3.1 map 与 filter

map 适配器能针对迭代器的各个条目调用闭包来帮你转换迭代器

filter 适配器能使用闭包来帮你从迭代器中过滤某些条目，由闭包决定保留和丢弃哪些条目

```rust
fn map<B, F>(self, f: F) -> impl Iterator<Item=B> 
  where Self: Sized, F: FnMut(Self::Item) -> B;
fn filter<P>(self, predicate: P) -> impl Iterator<Item=Self::Item> 
  where Self: Sized, P: FnMut(&Self::Item) -> bool;
```
map 迭代器会按值将每个条目传给闭包，然后将闭包结果的所有权转移给自己的消费者。
filter 迭代器会通过共享引用将每个条目传给闭包，并保留所有权以便再把选定的条目传给自己的消费者。


逐行遍历文本并希望去掉每一行的前导空格和尾随空格。
```rust
let text = "   ponies   \n    giraffes\niguanas\nsquid".to_string();
let v: Vec<&str> = text.lines().map(str::trim).collect();
```

text.lines() 调用会返回一个生成字符串中各行的迭代器。
在该迭代器上调用 map 会返回第二个迭代器，第二个迭代器会对每一行调用 str::trim 并将生成的结果作为自己的条目。
最后，collect 会将这些条目收集到一个向量中。

。。map，迭代器， 确实，因为它有输出，并且输出也是 一个一个的单词， 就是 java的stream。

如果你想将结果中的 iguanas 排除
```rust
let v: Vec<&str> = text.lines().map(str::trim).filter(|s| *s != "iguanas").collect();
```

filter 迭代器的条目类型是 &str，因此闭包参数 s 的类型是 &&str



### 15.3.2 filter_map 与 flat_map

如果想从迭代中删除而不是处理某些条目，或想用零个或多个条目替换单个条目时该怎么办？
filter_map（过滤映射）适配器和 flat_map（展平映射）适配器为你提供了这种灵活性。

。map 后 filter 呢？  就是 filter_map ？

filter_map 适配器与 map 类似，不同之处在于前者允许其闭包将条目转换为新条目（就像 map 那样）==或==从迭代中丢弃该条目。
因此，它有点儿像 filter 和 map 的组合

```rust
fn filter_map<B, F>(self, f: F) -> impl Iterator<Item=B> 
  where Self: Sized, F: FnMut(Self::Item) -> Option<B>;
```

和 map 基本相同，只是返回 `Option<B>` 而不是 B


扫描字符串，以查找可解析为数值且以空格分隔的单词，然后处理该数值，忽略其他单词

```rust
    use std::str::FromStr;
    let text = "1\nfrontd .25    234\n3.14159 estauary.\n";
    
    // filter_map 迭代器会丢弃所有 None 值并为每个Some(v) 生成值 v
    for num in text.split_whitespace().filter_map(|w| f64::from_str(w).ok())
    {
        println!("{:4.2}", num.sqrt());
    }
```

只用 filter 和 map 也可以做同样的事，但略显烦琐

```rust
text.split_whitespace()
  .map(|w| f64::from_str(w))
  .filter(|r| r.is_ok())
  .map(|r| r.unwrap())
```

我们可以将 flat_map 视为与 filter_map 功能类似的适配器，即对 map 的功能延伸，只是现在这个闭包不仅可以像 map 那样返回一个条目，或像 filter_map 那样返回零个或一个条目，还可以==返回任意数量的条目序列==。
也就是说，flat_map 迭代器会生成此闭包返回的序列串联后的结果。

```rust
fn flat_map<U, F>(self, f: F) -> impl Iterator<Item=U::Item>
  where F: FnMut(Self::Item) -> U, U: IntoIterator;
```
返回一个可迭代者
Option 也是一个可迭代者


将国家映射成其主要城市的表。给定一个国家列表，如何遍历这些国家的主要城市呢？

```rust
    use std::collections::HashMap;
    let mut cities = HashMap::new();
    cities.insert("Japan", vec!["Tokyo", "Kyoto"]);
    cities.insert("USA", vec!["Portland", "New York"]);
    cities.insert("Brazil", vec!["Sao Paulo", "Brasilis"]);
    let countries = ["Japan", "USA", /*"CN"*/];     // cities中不存在就报错
    for &city in countries.iter().flat_map(|country| &cities[country]) {
        println!("{}", city);
    }
```


### 15.3.3 flatten

flatten（展平）适配器会串联起迭代器的各个条目，这里假设每个条目本身都是可迭代者

```rust
fn flatten(self) -> impl Iterator<Item=Self::Item::Item>
  where Self::Item: IntoIterator;
```

```rust
use std::collections::BTreeMap;

let mut parks = BTreeMap::new();
parks.insert("Portland", vec!["park 1", "park 2"]);
parks.insert("Kyoto", vec!["park 3", "park 4"]);
parks.insert("Nas", vec!["park 5", "park 6"]);
let all_parks: Vec<_> = parks.values().flatten().cloned().collect();
println!("{:?}", all_parks);
```

底层迭代器的条目必须自行实现 IntoIterator 才能真正形成一个由序列组成的序列


从一个 `Vec<Option<...>>` 中迭代出 Some 的值
```rust
let arr = vec![None, Some("day"), None, Some("one")];
let vi = arr.into_iter().flatten().collect::<Vec<_>>();
println!("{:?}", vi);
```

也可以用 flatten 来迭代 `Option<Vec<...>>` 值：其中 None 的行为等同于空向量


==Result== 也实现了 IntoIterator，其中 Err 表示一个空序列，因此将 flatten 应用于 Result 值的迭代器有效地排除了所有Err 并将它们丢弃
通常不应该忽略代码中的错误


如果你发现自己随手用了 flatten，这个时候你真正需要的可能是flat_map
```rust
fn to_uppercase(&self) -> String {
  self.chars()
    .map(char::to_uppercase)
    .flatten() // 使用flat_map更好
    .collect()
}
```
用 flatten 的原因是 ch.to_uppercase() 返回的不是单个字符，而是会生成一个或多个字符的迭代器

map 和 flatten 的组合非常常见，因此 Iterator 为这种情况提供了 flat_map 适配器。
```rust
fn to_uppercase(&self) -> String {
  self.chars()
    .flat_map(char::to_uppercase)
    .collect()
}
```

### 15.3.4 take 与 take_while

Iterator 特型的 take（取出）适配器和 take_while（当……时取出）适配器的作用是当条目达到一定数量或闭包决定中止时==结束迭代==。

```rust
fn take(self, n: usize) -> impl Iterator<Item=Self::Item>
  where Self: Sized;
fn take_while<P>(self, predicate: P) -> implIterator<Item=Self::Item>
  where Self: Sized, P: FnMut(&Self::Item) -> bool;
```

两者都会接手某个迭代器的所有权并返回一个新的迭代器，新的迭代器会从第一个迭代器中传递条目，并可能提早终止序列。

给定一封电子邮件，其中有一个空行将标题与邮件正文分隔开，如果只想遍历标题就可以使用 take_while

```rust
    let msg = "To: anyone\r\n\
    from: supergo<editor@oreilly.com>\r\n\
    \r\n\
    Did you get any write today?\r\n\
    when will you stop waste?\r\n";

    for header in msg.lines().take_while(|l| !l.is_empty()) {
        println!("{}", header);
    }
```
。。不过，这个不算 标题吧。 这个是 收发件人。
。就是 一直取，直到 行是空的。


### 15.3.5 skip 与 skip_while

与 take 和 take_while 互补的方法：
skip 从迭代开始时就丢弃一定数量的条目，
skip_while 则一直丢弃条目直到闭包终于找到一个可接受的条目为止，然后将剩下的条目按照原样传递出来

```rust
fn skip(self, n: usize) -> impl Iterator<Item=Self::Item>
  where Self: Sized;
fn skip_while<P>(self, predicate: P) -> impl Iterator<Item=Self::Item>
  where Self: Sized, P: FnMut(&Self::Item) -> bool;
```

skip 适配器的常见用途之一是在迭代程序的命令行参数时跳过命令本身的名称
`for arg in std::env::args().skip(1) {`


```rust
    let msg = "To: anyone\r\n\
    from: supergo<editor@oreilly.com>\r\n\
    \r\n\
    Did you get any write today?\r\n\
    when will you stop waste?\r\n";

    for body in msg.lines().skip_while(|l| !l.is_empty()).skip(1) {
        println!("{}", body);
    }
```
空行 是可以通过 l.is_empty() 的， 所以 需要 skip 掉。




### 15.3.6 peekable

允许我们窥视即将生成的下一个条目，而无须实际消耗它

调用 Iterator 特型的 peekable方法可以将任何迭代器变成 peekable 迭代器

```rust
fn peekable(self) -> std::iter::Peekable<Self>
  where Self: Sized;
```


peekable 迭代器有一个额外的方法 peek，该方法会返回一个`Option<&Item>`：
如果底层迭代器已耗尽，那么返回值就为None；
否则为 Some(r)，其中 r 是对下一个条目的共享引用。

调用 peek 会尝试从底层迭代器中提取下一个条目，如果条目存在，就将其缓存，直到下一次调用 next 时给出

如果要从字符流中解析数值，那么在看到数值后面的第一个非数值字符之前是无法确定该数值的结束位置的

```rust
use std::iter::Peekable;
fn parse_number<I>(tokens: &mut Peekable<I>) -> u32 where I: Iterator<Item=char>
{
    let mut n = 0;
    loop {
        match tokens.peek() {
            Some(r) if r.is_digit(10) => {
                n = n * 10 + r.to_digit(10).unwrap();
            }
            _ => return n
        }
        tokens.next();
    }
}
fn peekable_()
{
    let mut chars = "123123,4234234".chars().peekable();
    println!("{}", parse_number(&mut chars));
    println!("{:?}", chars.next());
    println!("{}", parse_number(&mut chars));
    println!("{:?}", chars.next());
    println!("{:?}", chars.next());
}
```
输出：
```text
123123
Some(',')
4234234
None
```



### 15.3.7 fuse

Iterator 特型并没有规定一旦 next 返回 None 之后，再次调用next 方法时应该如何行动。大多数迭代器只是再次返回 None
但也有例外

fuse（保险丝）适配器能接受任何迭代器并生成一个确保在第一次返回 None 后继续返回 None 的迭代器
。。。



### 15.3.8 可逆迭代器与 rev

有的迭代器能够从序列的两端抽取条目，使用 rev（逆转）适配器可以反转此类迭代器

std::iter::DoubleEndedIterator 特型
```rust
trait DoubleEndedIterator: Iterator {
  fn next_back(&mut self) -> Option<Self::Item>;
}
```

```rust
fn rev(self) -> impl Iterator<Item=Self>
  where Self: Sized + DoubleEndedIterator;
```


```rust
let bee = ["head", "thorax", "abdomen"];
let mut it = bee.iter();

println!("{:?}", it.next());
println!("{:?}", it.next_back());
println!("{:?}", it.next());
println!("{:?}", it.next());
println!("{:?}", it.next_back());

let mut it2 = bee.iter().rev();
// println!("{:?}", it.next());
println!("{:?}", it2.next());   // <-
println!("{:?}", it2.next_back());  // ->
println!("{:?}", it2.next());
println!("{:?}", it2.next_back());
```



### 15.3.9 inspect

inspect（探查）适配器为调试迭代器适配器的流水线提供了便利，但在生产代码中用得不多。

inspect 只是对每个条目的共享引用调用闭包，然后传递该条目。闭包不会影响条目，但可以做一些事情，比如打印它们或对它们进行断言。




### 15.3.10 chain

chain（链接）适配器会将一个迭代器追加到另一个迭代器之后。
更准确地说，i1.chain(i2) 会返回一个迭代器，该迭代器从 i1 中提取条目，直到用尽，然后从 i2 中提取条目

```rust
fn chain<U>(self, other: U) -> impl Iterator<Item=Self::Item>
  where Self: Sized, U: IntoIterator<Item=Self::Item>;
```

```rust
let v: Vec<i32> = (1..4).chain([20, 30, 40]).collect();
assert_eq!(v, [1, 2, 3, 20, 30, 40]);
```

chain 迭代器会跟踪这两个底层迭代器是否返回了 None 并按需把其中一个迭代器的 next 和 next_back 重定向到另一个迭代器的next 和 next_back。




### 15.3.11 enumerate

Iterator 特型的 enumerate（枚举）适配器会将运行索引附加到序列中，它接受某个迭代器生成的条目 A, B, C, ... 并返回生成的值对 (0, A), (1, B), (2, C), ...。

`for (i, band) in bands.into_iter().enumerate() {`

```rust
let vi: Vec<i32> = vec![11,22,33,44,55];
for (i, num) in vi.iter().enumerate() {   // vi.into_iter()
    println!("{:?}, {:?}", i, num);
}
println!("{:?}", vi);       // vi.into_iter()，这里会panic
```


### 15.3.12 zip

zip（拉合）适配器会将两个迭代器组合成一个迭代器，新的迭代器会生成值对，每个底层迭代器各提供一个值，就像把拉链的两侧拉合起来一样。
当两个底层迭代器中的任何一个已结束时，拉合后的迭代器就结束了。

可以通过将无尽范围 0.. 与一个迭代器拉合起来获得与enumerate 适配器相同的效果

```rust
let v: Vec<_> = (0..).zip("ABCDE".chars()).collect();
println!("{:?}", v);
```

zip 的参数本身不一定是迭代器，可以是任意可迭代者。

```rust
    use std::iter::repeat;
    let vs = ["one", "two", "three"];
    let vzs: Vec<_> = repeat("going").zip(vs).collect();
    println!("{:?}", vzs);
```


### 15.3.13 by_ref

前面我们一直在将适配器附加到迭代器上。
一旦开始这样做，还能再取下适配器吗？
一般来说是不能，因为适配器会接手底层迭代器的所有权，并且没有提供归还所有权的方法。


迭代器的 by_ref（按引用）方法会借入迭代器的可变引用，便于将各种适配器应用于该引用。
一旦消耗完适配器中的条目，就会丢弃这些适配器，借用也就结束了，然后你就能重新获得对原始迭代器的访问权。


```rust
    let msg = "To: anyone\r\n\
    from: supergo<editor@oreilly.com>\r\n\
    \r\n\
    Did you get any write today?\r\n\
    when will you stop waste?\r\n";

    let mut lines = msg.lines();

    for header in lines.by_ref().take_while(|l| !l.is_empty()) {
        println!("{}", header);
    }

    println!("\nbody:");
    for body in lines {
        println!("{}", body);
    }
```


### 15.3.14 cloned 与 copied

cloned（克隆后）适配器会接受一个生成引用的迭代器，并返回一个会生成从这些引用克隆而来的值的迭代器，就像iter.map(|item| item.clone())。当然，引用目标的类型也必须实现了 Clone

```rust
  let a = ['1','2','3','7'];
  println!("{:?}, {:?}", a.iter().next(), a.iter().cloned().next());
  assert_eq!(a.iter().next(), Some(&'1'));
  assert_eq!(a.iter().next(), Some('1').as_ref());
  assert_eq!(a.iter().cloned().next(), Some(&'1').copied());
  assert_eq!(a.iter().cloned().next(), Some('1'));
```

copied（复制后）适配器的设计思想同样如此，但限制更严格，它要求引用目标的类型必须实现了 Copy。
像 iter.copied() 这样的调用与 iter.map(|r| *r) 大致相同。
由于每个实现了 Copy的类型也必定实现了 Clone，因此 cloned 更通用。
但根据条目类型的不同，clone 调用可能会进行任意次数的分配和复制。
如果你认为由于条目类型很简单，因而永远不会发生内存分配，那么最好使用copied 来让类型检查器帮你验证这种假设。

。。？


### 15.3.15 cycle

cycle（循环）适配器会返回一个迭代器，它会==无限重复底层迭代器生成的序列==。
底层迭代器必须实现 std::clone::Clone，以便cycle 保存其初始状态并且在每次循环重新开始时复用它。

```rust
let dirs = ["North", "East", "South", "West"];
let mut spin = dirs.iter().cycle();

println!("{:?}, {:?}, {:?}", spin.next(), spin.next(), spin.next());
println!("{:?}, {:?}, {:?}", spin.next(), spin.next(), spin.next());
println!("{:?}, {:?}, {:?}", spin.next(), spin.next(), spin.next());
```


```rust
use std::iter::{once, repeat};

let fiz = repeat("").take(2).chain(once("fizz")).cycle();
let buz = repeat("").take(4).chain(once("buzz")).cycle();
let fiz_buz = fiz.zip(buz);

let f_b = (1..20).zip(fiz_buz).map(|tuple| match tuple {
    (i, ("", "")) => i.to_string(),
    (_, (fiz, buz)) => format!("{}{}", fiz, buz),
});

for line in f_b {
    println!("{}", line);
}
```


## 15.4 消耗迭代器

### 15.4.1 简单累加：count、sum 和 product

```rust
use std::io::prelude::*;

let stdin = std::io::stdin();
println!("{}", stdin.lock().lines().count());
```


```rust
use std::io::prelude::*;

let mut t2: u64 = (1..=20).sum();
println!("{}", t2);

t2 = (1..=20).product();
println!("{}", t2);
```

为方便与其他类型一起工作，可以通过实现 `std::iter::Sum` 特型和 `std::iter::Product` 特型来扩展 sum 和 product，这里我们就不展开讲解这些特型了


### 15.4.2 min 与 max

Iterator 上的 min（最小）方法和 max（最大）方法会分别返回迭代器生成的最小条目与最大条目。
迭代器的条目类型必须实现 `std::cmp::Ord`

```rust
let vi: Vec<i32> = vec![-2,1,2,4,-11,-3,5,-3];
println!("{:?}, {:?}", vi.iter().max(), vi.iter().min());
assert_eq!(vi.iter().max(), Some(&5));
```

Rust 的浮点类型 f32 和 f64 仅实现了std::cmp::PartialOrd 而没有实现 std::cmp::Ord，因此不能使用 min 方法和 max 方法来计算浮点数序列中的最小值和最大值


### 15.4.3 max_by 与 min_by

```rust
use std::cmp::Ordering;

fn cmp(lhs: &f64, rhs: &f64) -> Ordering {
    lhs.partial_cmp(rhs).unwrap()
}

fn max_by_min_by()
{
    let num = [1.0, 4.0, 2.0];
    println!("{:?}, {:?}", num.iter().copied().max_by(cmp), num.iter().copied().min_by(cmp));

    let num = [1.0, 2.0, std::f64::NAN];
    // println!("{:?}, {:?}", num.iter().copied().max_by(cmp), num.iter().copied().min_by(cmp));
                        // panic, Option.unwrap on the None
}
```


### 15.4.4 max_by_key 与 min_by_key

使用 Iterator 上的 max_by_key（据键最大）方法和min_by_key（据键最小）方法可以选择最大条目或最小条目，由针对每个条目调用的闭包确定。

这两个函数通常比 max 和 min更有用

```rust
fn min_by_key<B: Ord, F>(self, f: F) -> Option<Self::Item>
  where Self: Sized, F: FnMut(&Self::Item) -> B;
fn max_by_key<B: Ord, F>(self, f: F) -> Option<Self::Item>
  where Self: Sized, F: FnMut(&Self::Item) -> B;
```

```rust
use std::collections::HashMap;

let mut map2 = HashMap::new();
map2.insert("Portland", 555_555);
map2.insert("Fossil", 449);
map2.insert("Greenhorn", 2);
map2.insert("Boring", 7777);
map2.insert("The Dalles", 15_555);

println!("{:?}, {:?}", map2.iter().max_by_key(|&(_name, pop)| pop), map2.iter().min_by_key(|&(_name, pop)| pop));
```



### 15.4.5 对条目序列进行比较 eq,lt...


如果字符串、向量和切片的各个元素是可比较的，我们就可以使用 < 运算符和 == 运算符来对它们进行比较。
但是，比较运算符不能用来比较迭代器，这项工作是由像 eq 和 lt 这样的方法来完成的

```rust
let packed = "Helen of Troy";
let spaced = "Helen   of   Troy";
let obscure = "Helen of Sandusky"; // 好人，只是不出名
assert!(packed != spaced);
assert!(packed.split_whitespace().eq(spaced.split_whitespace()));
// 此断言为真，因为' ' < 'o'
assert!(spaced < obscure);
// 此断言为真，因为'Troy' > 'Sandusky'
assert!(spaced.split_whitespace().gt(obscure.split_whitespace()));
```


迭代器提供的比较方法既包括用于相等比较的 eq 方法和 ne 方法，也包括用于有序比较的 lt 方法、le 方法、gt 方法和 ge 方法。
而 cmp 方法和 partial_cmp 方法的行为类似于 Ord 特型和PartialOrd 特型的相应方法。



### 15.4.6 any 与 all

```rust
let id = "Iterator";
assert!( id.chars().any(char::is_uppercase));
assert!(!id.chars().all(char::is_uppercase));
```


### 15.4.7 position、rposition 和 ExactSizeIterator

position（位置）方法会针对迭代器中的每个条目调用闭包，并返回调用结果为 true 的第一个条目的索引。
确切而言，它会返回关于此索引的 Option：如果闭包没有为任何条目返回 true，则position 返回 None。一旦闭包返回 true，position 就会停止提取条目

。。从开头/左侧 开始找，找到 第一个 满足 lambda 的，或者 None

rposition（右起位置）方法也是一样的，只是从右侧开始搜索
要求 可逆迭代器， 且 确切大小迭代器 (实现了 std::iter::ExactSizeIterator 特型的迭代器)。
最左边的索引值为 0。
。。所以下标 还是正常的下标。


```rust
trait ExactSizeIterator: Iterator {
  fn len(&self) -> usize { ... }
  fn is_empty(&self) -> bool { ... }
}
```
len 方法会返回剩余的条目数，而 is_empty 方法会在迭代完成时返回 true。


```rust
let text = "Xerxes";
println!("{:?}, {:?}", text.chars().position(|c| c == 'e'), text.chars().position(|c| c=='z'));

let btx = b"Xerxes";
println!("{:?}, {:?}", btx.iter().rposition(|&c| c == b'e'), btx.iter().rposition(|&c| c==b'X'));
```



### 15.4.8 fold 与 rfold


fold（折叠）方法是一种非常通用的工具，用于在迭代器生成的整个条目序列上==累积某种结果==。
给定一个初始值（我们称之为 累加器）和一个闭包，fold 会以当前累加器和迭代器中的下一个条目为参数反复调用这个闭包。
闭包返回的值被视为新的累加器，并将其与下一个条目一起传给闭包。
最终，累加器的值就是 fold 本身返回的值。
如果序列为空，则 fold 只返回初始累加器。


```rust
    let a = [4,5,6,7,8,9];

    println!("{:?}", a.iter().fold(0, |n, _| n+1)); // count
    println!("{:?}", a.iter().fold(0, |n, i| n+i)); // sum
    println!("{:?}", a.iter().fold(1, |n, i| n*i)); // production
    println!("{:?}", a.iter().cloned().fold(i32::min_value(), std::cmp::max));  // max
```

```rust
fn fold<A, F>(self, init: A, f: F) -> A
  where Self: Sized, F: FnMut(A, Self::Item) -> A;
```

累加器的值会移动进闭包或者从闭包中移动出来，因此你可以将 fold 与各种非 Copy 的累加器类型一起使用

```rust
let a = ["AAA", "BBB", "CCC", "DDD"];
let t2 = a.iter().fold(String::new(), |s, w| s + w + " ");
println!("{}", t2);
```

rfold（右起折叠）方法与 fold 方法基本相同，但需要一个双端迭代器，并从后往前处理各个条目



### 15.4.9 try_fold 与 try_rfold

try_fold（尝试折叠）方法与 fold 方法基本相同，不过迭代可以提前退出，无须消耗迭代器中的所有值。
传给 try_fold 的闭包返回的值会指出它是应该立即返回，还是继续折叠迭代器的条目

闭包可以返回多种类型的值，根据类型值，try_fold 方法可判断继续折叠的方式。
- 闭包返回 `Result<T, E>`, 可能是因为它执行了 I/O 或其他一些容易出错的操作，Ok继续，Err停止。
- 闭包返回 `Option<T>`，Some继续，None停止
- 闭包还可以返回一个 std::ops::ControlFlow 值，这种类型是一个具有两个变体的枚举，即 Continue(c) 和Break(b)，分别表示使用新的累加器值 c 继续或提前中止迭代


Continue(c) 和 Break(b) 的行为与 Ok(c) 和 Err(b) 完全一样
使用 ControlFlow 而不用 Result 的优点在于，有时候提前退出并不表示出错了，而只是表明提前得出了答案，这种情况下它会让代码可读性更好

```rust
use std::error::Error;
use std::io::prelude::*;
use std::str::FromStr;
fn fun2() -> Result<(), Box<dyn Error>> {
    let stdin = std::io::stdin();           // linux:ctrl+d, win:ctrl+z
    let sum = stdin.lock().lines().try_fold(0, |sum, line| -> Result<u64, Box<dyn Error>> {
        Ok(sum + u64::from_str(&line?.trim())?)
    })?;
    println!("{}", sum);
    Ok(())
}

fn try_fold_try_rfold()
{
    println!(">>>> {:?} <<<<", fun2());
}
```


try_fold 非常灵活，被用来实现 Iterator 的许多其他消费者方法，比如，以下是 all 的一种实现

```rust
fn all<P>(&mut self, mut predicate: P) -> bool where P: FnMut(Self::Item) -> bool, Self: Sized {
  use std::ops::ControlFlow::*;
  self.try_fold((), |_, item| {
    if predicate(item) { Continue(()) } else { Break(()) }
  }) == Continue(())
}
```
无法用普通的 fold 来写，all 的语义承诺了一旦predicate 返回 false 就停止从底层迭代器中消耗条目，但fold 总是会消耗掉整个迭代器


如果你正在实现自己的迭代器类型，就值得研究一下你的迭代器能否比 Iterator 特型的默认定义更高效地实现 try_fold。如果可以为 try_fold 提速，那么基于它构建的所有其他方法都会受益。



### 15.4.10 nth 与 nth_back

nth（第 n 个）方法会接受索引参数 n，从迭代器中跳过 n 个条目，并返回下一个条目，如果序列提前结束了，则返回 None。
调用.nth(0) 等效于 .next()

nth 不会像适配器那样接手迭代器的所有权，因此可以多次调用
```rust
fn nth(&mut self, n: usize) -> Option<Self::Item>
  where Self: Sized;
```

nth_back（倒数第 n 个）方法与 nth 方法很像，只是从双端迭代器的后面往前提取。调用 .nth_back(0) 等效于.next_back()


```rust
let mut sq = (0..10).map(|i| i*i);      // iterator
println!("{:?}, {:?}, {:?}", sq.nth(4), sq.nth(0), sq.nth(7));  // 16, 25, None
```



### 15.4.11 last

返回迭代器生成的最后一个条目，如果为空则返回 None

`fn last(self) -> Option<Self::Item>;`

会从前面开始消耗所有迭代器的条目，即便此迭代器是可逆的也会如此。
如果你有一个可逆迭代器并且不想消耗它的所有条目，应该只写iter.next_back()。




### 15.4.12 find、rfind 和 find_map

find（查找）方法会从迭代器中提取条目，返回第一个由给定闭包回复 true 的条目，如果序列在找到合适的条目之前就结束了则返回None

```rust
fn find<P>(&mut self, predicate: P) -> Option<Self::Item> 
  where Self: Sized, P: FnMut(&Self::Item) -> bool;
```

```rust
assert_eq!(populations.iter().find(|&(_name, &pop)| pop > 1_000_000), None);
assert_eq!(populations.iter().find(|&(_name, &pop)| pop > 500_000), Some((&"Portland", &583_776)));
```

有时候，闭包不仅仅是一个简单的谓词——对每个条目进行逻辑判断并继续向前，它还可能是更复杂的东西，比如生成一个有意义的值。在这种情况下就需要使用 find_map（查找并映射）

```rust
fn find_map<B, F>(&mut self, f: F) -> Option<B> 
  where F: FnMut(Self::Item) -> Option<B>;
```

find_map 和 find 很像，但其闭包不会返回 bool，而是返回某个值的 Option。
find_map 会返回第一个类型为 Some 的 Option。

```rust
let big_city_with_volcano_park = populations.iter().find_map(|(&city, _)| {
  if let Some(park) = find_volcano_park(city, &parks) {
    // find_map会返回下面的值，以便让调用者知道我们找到了哪个公园
    return Some((city, park.name));
  }
  // 拒绝此条目，并继续搜索
  None
});
assert_eq!(big_city_with_volcano_park, Some(("Portland", "Mt. Tabor Park")));
```




### 15.4.13 构建集合：collect 与 FromIterator

`let args: Vec<String> = std::env::args().collect();`

```rust
use std::collections::{HashSet, BTreeSet, LinkedList, HashMap, BTreeMap};

let args: HashSet<String> = std::env::args().collect();
let args: BTreeSet<String> = std::env::args().collect();
let args: LinkedList<String> = std::env::args().collect();

// 只有键–值对才能收集到Map中，因此对于这个例子，
// 要把字符串序列和整数序列拉合在一起
let args: HashMap<String, usize> = std::env::args().zip(0..).collect();
let args: BTreeMap<String, usize> = std::env::args().zip(0..).collect();
```

collect 本身并不知道如何构造出所有这些类型。
相反，如果某些集合类型（如 Vec 或 HashMap）知道如何从迭代器构造自身，就会自行实现 `std::iter::FromIterator`（来自迭代器）特型，而 collect 只是一个便捷的浅层包装而已：

```rust
trait FromIterator<A>: Sized {
  fn from_iter<T: IntoIterator<Item=A>>(iter: T) -> Self;
}
```

向量固然进行了算法优化来保持这种开销尽可能低，但是如果有某种方法可以直接分配一个正确大小的缓冲区作为起点，那就根本没必要调整大小了
这是 Iterator 特型的 size_hint 方法的用武之地
```rust
trait Iterator {
...
  fn size_hint(&self) -> (usize, Option<usize>) {
    (0, None)
  }
}
```
size_hint 方法会返回迭代器要生成的条目数的下限与可选上限，默认返回 0 作为下限而没有指定上限




### 15.4.14 Extend 特型

如果一个类型实现了 std::iter::Extend（扩展）特型，那么它的 extend 方法就能将一些可迭代的条目添加到集合中

```rust
let mut v: Vec<i32> = (0..5).map(|i| 1 << i).collect();
v.extend([31, 57, 99, 163]);
assert_eq!(v, [1, 2, 4, 8, 16, 31, 57, 99, 163]);
```

```rust
trait Extend<A> {
  fn extend<T>(&mut self, iter: T) where T: IntoIterator<Item=A>;
}
```

这与 std::iter::FromIterator 非常相似：后者会创建新集合，而 Extend 会扩展现有集合。事实上，标准库中FromIterator 的好几个实现都只是简单地创建一个新的空集合，然后调用 extend 填充它




### 15.4.15 partition

```rust
let things = ["doorknob", "mushroom", "noodle", "giraffe", "grapefruit"];

let (live, nolive) : (Vec<&str>, Vec<&str>) = things.iter().partition(|name| name.as_bytes()[0] & 1 != 0);

println!("{:?}", live);
println!("{:?}", nolive);
```

```rust
fn partition<B, F>(self, f: F) -> (B, B) 
  where Self: Sized, B: Default + Extend<Self::Item>, F: FnMut(&Self::Item) -> bool;
```



### 15.4.16 for_each 与 try_for_each


```rust
let vs = ["doves", "hens", "birds"];
vs.iter().zip(["turtle", "frensh", "calling"])
    .zip(2..5)
    .rev()
    .map(|((item,kind),quantity)| {
        format!("{} {} {}", quantity, kind, item)
    })
    .for_each(|gift| {
        println!("you receive: {}", gift);
    });
```

这与简单的 for 循环非常相似，你同样可以在其中使用像 break 和 continue 这样的控制结构


```rust
for gift in ["doves", "hens", "birds"].iter()
  .zip(["turtle", "french", "calling"])
  .zip(2..5)
  .rev()
  .map(|((item, kind), quantity)| {
  format!("{} {} {}", quantity, kind, item)
  }) {
  println!("You have received: {}", gift);
}
```
要绑定的模式 gift 最终可能离使用它的循环体很远


如果你的闭包需要容错或提前退出，可以使用 try_for_each（尝试对每一个）



## 15.5 实现自己的迭代器


。。。







# ch16 集合 604

Rust 的集合与其他语言的集合之间的一些系统性差异

首先，移动和借用无处不在。Rust 使用移动来避免对值做深拷贝。这就是 `Vec<T>::push(item)` 方法会按值而非按引用来获取参数的原因。这样值就会移动到向量中

其次，Rust 没有失效型错误，也就是当程序仍持有指向集合内部数据的指针时，集合被重新调整大小或发生其他变化而导致的那种悬空指针错误。

Rust 没有 null，因此在其他语言使用 null 的地方 Rust 会使用 Option

如果你是一位经验丰富的程序员，而且时间有限，那么可以快速浏览本章，但不要错过 16.5.1 节


## 16.1 概述

|集合|描述|c++|java|python|
|--|--|--|--|--|
|`Vec<T>`|变长数组|vector|ArrayList|list|
|`VecDeque<T>`|双端队列(可增长的环形缓冲区)|deque|ArrayDeque|Collections.<br/>deque|
|`LinkedList<T>`|双向链表|list|LinkedList|-|
|`BinaryHead<T> where T: Ord`|最大堆|priority_queue|PriorityQueue|headq|
|`HashMap<K, V> where K: Eq + Hash`|键值hash表|unordered_map|HashMap|dict|
|`BTreeMap<K, V> where K: Ord`|有序键值表|map|TreeMap|-|
|`HashSet<T> where T: Eq + Hash`|无序，hash集|unordered_set|HashSet|set|
|`BTreeSet<T> where T: Ord`|有序集|set|TreeSet|-|


## 16.2 Vec

```rust
let mut numbers: Vec<i32> = vec![];
// 使用给定内容创建一个向量
let words = vec!["step", "on", "no", "pets"];
let mut buffer = vec![0u8; 1024]; // 1024个内容为0的字节
```

```rust
// 获取某个元素的引用
let first_line = &lines[0];

// 获取某个元素的副本
let fifth_number = numbers[4]; // 要求实现了Copy特型
let second_line = lines[1].clone(); // 要求实现了Clone特型

// 获取切片的引用
let my_ref = &buffer[4..12]; 

// 获取切片的副本
let my_copy = buffer[4..12].to_vec(); // 要求实现了Clone特型
```

下面的方法用于访问 向量 或 切片 ( 所有的切片方法 也适用于 数组和向量)
- slice.first()
- slice.last()
- slice.get(index)
- slice.first_mut()  last_mut()  get_mut(index)
- slice.to_vec();


- 遍历 `Vec<T>` 或数组 `[T; N]` 会生成 T 类型的条目。这些元素会逐个从向量或数组中移动出来并被消耗掉。
- 遍历 `&[T; N]、&[T] 或 &Vec<T>` 类型的值（对数组、切片或向量的引用）会生成 &T 类型的条目，即对单个元素的引用，这些元素不会移动出来。
- 遍历 `&mut [T; N]、&mut [T] 或 &mut Vec<T>` 类型的值会生成 &mut T 类型的条目。


容量，长度
- slice.len()
- slice.is_empty()
- Vec::with_capacity(n)
- vec.capacity()
- vec.reserve(n)
- vec.reserve_exact(n)
- vec.shrink_to_fit() 缩小到刚好够
- vec.push(value)
- vec.pop()
- vec.insert(index, value)
- vec.remove(index)
- vec.resize(new_len, value)  长度设置为 new_len，如果增加了长度，则用 value 的副本填充，value 必须实现 Clone
- vec.resize_with(new_len, closure)
- vec.truncate(new_len)
- vec.clear()
- vec.extend(iterable)
- vec.split_off(index) 与 vec.truncate(index) 类似，但此方法会返回一个`Vec<T>`，其中包含从 vec 末尾移除的那些值。此方法就像是.pop() 的多值版本。
- vec.append(&mut vec2)
- vec.drain(range)  抽取
- vec.retain(test) 留下
- vec.dedup() 去重
- vec.dedup_by(same)（根据 same 调用结果去重）
- vec.dedup_by_key(key)（根据 key 属性去重）

下面的2个方法 用于 数组的数组
- slices.concat()（串联）
- slices.join(&separator)（联结）


拆分
- slice.iter(), slice.iter_mut()
- slice.split_at(index), slice.split_at_mut(index)
- slice.split_first(), slice.split_first_mut()
- slice.split_last(), split_last_mut()
- slice.split(is_sep), split_mut(is_sep)
- slice.split_inclusive(is_sep), _mut
- slice.rsplit(is_sep), _mut
- slice.splitn(n, is_sep), _mut  拆分为 n 片
- slice.rsplitn(n, is_sep), _mut
- slice.chunks(n), _mut  分为长度为 n 的块
- slice.chunks_exact(n), _mut
- slice.rchunks_exact(n), _mut
- slice.windows(n)  （滑动窗口）


交换
- slice.swap(i, j)（交换元素）
- slice_a.swap_with_slice(slice_b)
- vec.swap_remove(i)（交换后移除）

填充
- slice.fill(value)（填充）
- slice.fill_with(function)（以 function 回调填充）

排序与搜索
都会执行稳定排序
- slice.sort()（排序）
- slice.sort_by(cmp)（按 cmp 回调排序）
- slice.sort_by_key(key)（按 key 回调排序）

- slice.reverse()（逆转）

- slice.binary_search(&value)（二分搜索）
- slice.binary_search_by(&value, cmp)（按 cmp 回调二分搜索）
- slice.binary_search_by_key(&value, key)（按 key 闭包二分搜索）
这些方法的返回类型是 `Result<usize, usize>`。

- slice.contains(&value)（包含）未排过序的向量中搜索

比较切片

如果类型 T 支持 == 运算符和 != 运算符（PartialEq 特型，参见 12.2 节），那么`数组 [T; N]、切片 [T] 和向量 Vec<T>` 也会支持这两个运算符。

- slice.starts_with(other)（以 other 开头）
- slice.ends_with(other)（以 other 结尾）


随机元素
随机数并未内置在 Rust 标准库中，但在 rand crate 中可以找到它们。rand crate 提供了以下两个方法，用于从数组、切片或向量中获取随机输出

- slice.choose(&mut rng)（随机选取）
- slice.shuffle(&mut rng)（随机洗牌）



## 16.3 VecDeque


Vec 只支持在末尾高效地添加元素和移除元素。当程序需要一个地方来存储“排队等候”的值时，Vec 可能会很慢。

`std::collections::VecDeque<T>` 是一个双端队列
在首端和尾端进行高效的添加操作和移除操作
VecDeque 的实现是一个==环形缓冲区==

- deque.push_front(value)（队首推入）
- deque.push_back(value)（队尾推入）
- deque.pop_front()（队首弹出）
- deque.pop_back()（队尾弹出）
- deque.front()（队首）和 deque.back()（队尾）
- deque.front_mut()（队首，可变版）和 deque.back_mut()（队尾，可变版）


因为双端队列不会将自己的元素存储在连续的内存中，所以它们无法继承切片的所有方法。但是，如果你愿意承受移动内容的开销，则可以通过 VecDeque 提供的如下方法来解决此问题。


- deque.make_contiguous()（变连续）
  获取 &mut self 并将 VecDeque 重新排列到连续的内存中，返回 &mut [T]
- Vec::from(deque)（来自双端队列）
- VecDeque::from(vec)（来自向量）


## 16.4 BinaryHeap

3 个最常用的 BinaryHeap 方法

- heap.push(value)（压入）
- heap.pop()（弹出）
- heap.peek()（窥视）， heap.peek_mut()（窥视，可变版）

BinaryHeap 还支持 Vec 上的部分方法，包括 
- BinaryHeap::new()、
- .len()、
- .is_empty()、
- .capacity()、
- .clear() 和 
- .append(&mut heap2)。



## 16.5 HashMap 与 BTreeMap

Rust 标准库采用了 B 树而不是平衡二叉树，因为 B 树在现代硬件上速度更快。
两相对比，二叉树固然在每次搜索时的比较次数较少，但B 树具有更好的 局部性 (内存访问被分组在一起，而不是分散在整个堆中)。这使得 CPU 缓存未命中的情况更为罕见

创建
- HashMap::new()（新建）和 BTreeMap::new()（新建）
- iter.collect()（收集）
- HashMap::with_capacity(n)（自带容量）

处理键值
- map.len()（长度）
- map.is_empty()（为空？）
- map.contains_key(&key)（包含 key？）
- map.get(&key)（按 key 获取）
- map.get_mut(&key)（按 key 获取，可变版）
- map.insert(key, value)（插入）
- map.extend(iterable)（用 iterable 扩展）
- map.append(&mut map2)（从 map2 追加）
- map.remove(&key)（按 key 移除值）
- map.remove_entry(&key)（按 key 移除条目）
- map.retain(test)（留下）
- map.clear()（清空）

可以使用方括号来查询 Map，比如 `map[&key]`。
也就是说，Map实现了内置特型 Index。
但是，如果==给定 key 还没有条目==（就像越界数组访问），则会出现 ==panic==，因此只有在要查找的条目肯定已填充过时才应使用此语法

.contains_key()、.get()、.get_mut() 和 .remove() 的 key 参数不必具有确切的类型 &K。这些方法对可以从 K 借用来的类型来说是通用的


`BTreeMap<K, V>` 会始终保持其条目是根据键排序的，所以它支持一些额外的操作
- btree_map.split_off(&key)（拆分出）


### 16.5.1 条目entry

HashMap 和 BTreeMap 都有其对应的 Entry（条目）类型。条目的作用旨在消除冗余的 Map 查找。

```rust
// 已经有关于此学生的记录了吗？
if !student_map.contains_key(name) {
  // 没有：创建一个
  student_map.insert(name.to_string(), Student::new());
}
// 现在，肯定存在一条记录了
let record = student_map.get_mut(name).unwrap();
```
会访问 student_map 两次或 3 次


下面这个单行代码等效于上一段代码，但它只会执行一次查找
```rust
let record = student_map.entry(name.to_string()).or_insert_with(Student::new);
```

所有 Entry 值都是由同一个方法创建的。
- map.entry(key)（按 key 取条目）

3 个方法来处理空条目。
- map.entry(key).or_insert(value)（取条目或插入）
- map.entry(key).or_default()（取条目或插入默认值）
- map.entry(key).or_insert_with(default_fn)（取条目或借助 default_fn 插入）

修改现有字段
- map.entry(key).and_modify(closure)（取条目并修改）


几个方法可以对 Map 进行迭代。
- 按值迭代（for (k, v) in map）以生成 (K, V) 对。这会消耗 Map。
- 按共享引用迭代（for (k, v) in &map）以生成 (&K, &V)对。
- 按可变引用迭代（for (k, v) in &mut map）以生成 (&K, &mut V) 对。（同样，无法对存储在 Map 中的键进行可变访问，因为这些条目是通过它们的键进行组织的。）

Map 也有 .iter() 方法和 .iter_mut() 方法，它们会返回针对“条目引用”的迭代器
- map.keys()（所有键的迭代器）
- map.values()（所有值的迭代器）
- map.values_mut()（所有值的可变迭代器）
- map.into_iter()（转为迭代器）、map.into_keys()（转为键迭代器）和 map.into_values()（转为值迭代器）


所有 HashMap 迭代器都会以任意顺序访问 Map 的条目，而BTreeMap 迭代器会按 key 的顺序访问它们。

## 16.6 HashSet 与 BTreeSet

Rust 的两个 Set 类型 `HashSet<T> 和 BTreeSet<T>` 都是通过对 `HashMap<T, ()> 和BTreeMap<T, ()>` 的浅层包装实现的


- HashSet::new()（新建）和 BTreeSet::new()（新建）
- iter.collect()（收集）
- HashSet::with_capacity(n)（自带容量）


- set.len()（长度）
- set.is_empty()（为空？）
- set.contains(&value)（包含）
- set.insert(value)（插入）
  如果新增了一个值，就返回true；如果它先前已是此 set 的成员，则返回 false。
- set.remove(&value)（移除）
  如果移除了一个值，就返回true；如果它之前不是此 set 的成员，则返回 false。
- set.retain(test)（留下）

两个方法可以迭代 Set。
- 按值迭代（for v in set）会生成 Set 的成员并消耗掉此Set。
- 按共享引用（for v in &set）迭代会生成对 Set 成员的共享引用。

不支持通过可变引用迭代 Set。无法获取对存储在 Set 中的值的可变引用。

- set.iter()（迭代器）


- set.get(&value)（取值）
- set.take(&value)（拿出值）
  与 set.remove(&value) 类似，但此方法会返回所移除的值（如果有的话）。返回类型是 `Option<T>`。
- set.replace(value)（替换为）
  与 set.insert(value) 类似，但如果 set 已经包含一个等于 value 的值，那么此方法就会替代并返回原来的值。返回类型是`Option<T>`



- set1.intersection(&set2)（交集）
  &set1 & &set2 会返回一个新 Set，该 Set 是 set1 和set2 的交集
- set1.union(&set2)（并集）
  &set1 | &set2 会返回包含所有这些值的新 Set。它会找出所有存在于 set1 或 set2 中的值。
- set1.difference(&set2)（差集）
  &set1 - &set2 会返回包含所有此类值的新 Set。
- set1.symmetric_difference(&set2)（对称差集，异或）
  返回存在于 set1 或 set2 中但不同时存在于两者中的迭代器。
  &set1 ^ &set2 会返回包含所有此类值的新 Set。



- set1.is_disjoint(set2)（有交集？）
- set1.is_subset(set2)（是子集？）
- set1.is_superset(set2)（是超集？）


Set 还支持使用 == 和 != 进行相等性测试。如果两个 Set 包含完全相同的一组值，那它们就是相等的。


## 16.7 哈希

std::hash::Hash 是可哈希类型的标准库特型。HashMap 的键和HashSet 的元素都必须实现 Hash 和 Eq。

标准库的一个设计原则是，无论将值存储在何处或如何指向它，都应具有相同的哈希码。
因此，
- 引用与其引用的值具有相同的哈希码，而
- Box 与其封装的值也具有相同的哈希码。
- 向量 vec 与包含其所有数据的切片 &vec[..] 具有相同的哈希码。
- String 与具有相同字符的 &str 具有相同的哈希码。


默认情况下，结构体和枚举没有实现 Hash，但可以派生一个实现：
```rust
/// 大英博物馆藏品中某件物品的ID号
#[derive(Clone, PartialEq, Eq, Hash)]
enum MuseumNumber {
  ...
}

impl PartialEq for Artifact {
  fn eq(&self, other: &Artifact) -> bool {
    self.id == other.id
  }
}

impl Eq for Artifact {}

use std::hash::{Hash, Hasher};
impl Hash for Artifact {
  fn hash<H: Hasher>(&self, hasher: &mut H) {
    // 把哈希工作委托给藏品编号
    self.id.hash(hasher);
  }
}
```

`let mut collection = HashSet::<Artifact>::new();`




## 16.8 使用自定义哈希算法

std::hash::BuildHasher 是表示哈希算法初始状态的类型的特型

每个 Hasher 都是单次使用的，就像迭代器一样：用过一次就扔掉了。而 BuildHasher 是可重用的。


## 16.9 在标准集合之外





# ch17 字符串与文本 658

Unicode， ASCII 是字符集，即 123 代表了 哪个字符
UTF-8 是 Unicode 的 编码格式，即 内存中用几个字节表示/怎么表示 123

## 17.1 一些 Unicode 背景知识

Unicode 和 ASCII 对于从 0 到 0x7f 的所有 ASCII 码点是一一对应的
。。 0-127

Unicode 也将0 到 0xff 分配给了与 ISO/IEC 8859-1 字符集相同的字符，这是ASCII 字符集用于西欧语言的 8 位超集
。。0-255
使用 Latin-1 来指代ISO/IEC 8859-1。

由于 Unicode 是 Latin-1 的超集，因此将 Latin-1 转换为 Unicode甚至不需要查表

Rust 的 String 类型和 str 类型表示使用了 UTF-8 编码形式的文本。UTF-8 会将字符编码为 1~4 字节的序列

UTF-8 编码：
1字节： 0xxxxxxx
2字节： 110xxxxx 10xxxxxx
3字节： 1110xxxx 10xxxxxx 10xxxxxx
4字节： 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx

只有任何给定码点的最短编码才被认为是格式良好的，你不能花费 4 字节来编码原本只需要 3 字节的码点

格式良好的 UTF-8 不得对从 0xd800 到 0xdfff 或超过0x10ffff 的数值进行编码：这些数值要么保留用作非字符目的，要么完全超出了 Unicode 的范围

```rust
assert_eq!("うどん: udon".as_bytes(),
  &[0xe3, 0x81, 0x86, // う
  0xe3, 0x81, 0xa9, // ど
  0xe3, 0x82, 0x93, // ん
  0x3a, 0x20, 0x75, 0x64, 0x6f, 0x6e // : udon
  ]);
```

Rust 的 char 类型是一个包含 Unicode 码点的 32 位值。char 保证会落在0~ 0xd7ff 或 0xe000~ 0x10ffff 范围内

字符串切片可以使用 slice.chars() 生成针对其字符的迭代器：
`assert_eq!("カニ".chars().next(), Some('カ'));`


17.2.1 字符分类

char 类型的一些方法可以将字符分入几个常见类别

类型的分类方法
|方法|描述|例子|
|--|--|--|
|ch.is_numeric()|数值，包括Unicode通用类别中的 数值:数字 和 数值:字母，不包括 数值:其他|'⑧'.is_numeric()|
|ch.is_alphabetic|字母，Unicode字符属性 Alphabetic|'其'.is_alphabetic|
|ch.is_alphanumeric()|数值或字母|-|
|ch.is_whitespace()|空白字符，Unicode字符属性 WSpace=Y|'\n'.is_whitespace()|
|ch.is_control|控制字符 Unicode的 其他:控制字符|'\n'.is_control()|


ASCII分类方法
- is_ascii()
- is_ascii_alphabetic()
- is_ascii_digit()
- is_ascii_hexdigit()
- is_ascii_alphanumeric()
- is_ascii_control()
- is_ascii_uppercase()
- is_ascii_lowercase()
- is_ascii_punctuation()
- is_ascii_whitespace()

ASCII 的方法也可以用于 u8 字节类型
```rust
assert!(32u8.is_ascii_whitespace());
assert!(b'9'.is_ascii_digit());
```

这些分类可能存在某些令人吃惊的差异
注意 is_whitespace 和 is_ascii_whitespace 对某些字符的处理不同
```rust
let line_tab = '\u{000b}'; //“行间制表符”，也叫“垂直制表符”
assert_eq!(line_tab.is_whitespace(), true);
assert_eq!(line_tab.is_ascii_whitespace(), false);
```

- ch.to_digit(radix)（转数字）
  返回 Option，只能识别 ASCII 数字

- std::char::from_digit(num, radix)（来自数字）
  自由函数，只要有可能，就可以把 u32 数字值 num 转换为 char
  to_digit 的逆函数

- ch.is_digit(radix)（是数字？）


- ch.is_lowercase()（是小写？）
- ch.is_uppercase()（是大写？）

- ch.to_lowercase()（转小写）
- ch.to_uppercase()（转大写）

Rust 的 as 运算符会将 char 转换为任何整数类型，并抹掉高位



## 17.3 String 与 str 的方法

Rust 的 String 类型和 str 类型会保证自己只包含格式良好的UTF-8

String 可以解引用成 &str，因此在 str 上定义的每个方法都可以直接在 String 上使用

文本处理方法会按字节偏移量索引文本并以 字节 而 不是字符 为单位测量其长度。

实际上，考虑到 Unicode 的性质，按字符索引并不像看起来那么有用，按字节偏移量索引反而更快且更简单。
如果试图使用位于某个字符的 UTF-8 编码中间的字节偏移量，则该方法会发生panic，因此不能通过这种方式引入格式错误的 UTF-8。
。？都Unicode了，肯定要知道多少个字符啊，要字节干什么？而且后面又说了 访问 中间字节，会 panic。。
。。也有一点道理，建立缓冲区，告知对方 需要预留多少内存。  对于人来说，肯定要 char 的个数。 对于机器来说， byte的个数更好。


String 通过封装 `Vec<u8>` 实现，并可以确保向量中的内容永远是格式良好的 UTF-8
可以假设 String 的性能表现始终会和 Vec 保持一致

后续中 slice 是指 &str 或 对某值(如`String或 Rc<String>`) 的解引用


创建字符串值
- String::new()
- String::with_capacity(n)
- str_slice.to_string()
- iter.collect()
- slice.to_owned()

简单探查
- slice.len()
- slice.is_empty()
- slice[range]
  有界的范围、部分有界的范围和无界的范围都可以
  不能索引具有单个位置的字符串切片，比如 slice[i]
- slice.split_at(i)
- slice.is_char_boundary(i)（是字符边界？）


追加、插入文本
- string.push(ch)（压入）
- string.push_str(slice)（压入字符串）
- string.extend(iter)（以 iter 扩展）
- string.insert(i, ch)（插入于）
- string.insert_str(i, slice)（插入字符串于）
- 


String 实现了 Add<&str> 和 AddAssign<&str>，所以你可以编写如下代码：
```rust
let left = "partners".to_string();
let mut right = "crime".to_string();
assert_eq!(left + " in " + &right, "partners in crime");

right += " doesn't pay";
assert_eq!(right, "crime doesn't pay");
```

```rust
let parenthetical = "(" + string + ")";   // error
let parenthetical = "(".to_string() + &string + ")";  // ok
```


移除、替换文本
- string.clear()（清空）
- string.truncate(n)（截断为 n 个）
- string.pop()（弹出）
- string.remove(i)（移除）
- string.drain(range)（抽取）
- string.replace_range(range, replacement)（替换范围）
  切片不必与要替换的范围长度相同

搜索与迭代的约定
- r
  大多数操作会从头到尾处理文本，但名称以 r 开头的操作会==从尾到头==处理
- n
  名称以 n 结尾的迭代器会将自己限定为只取给定数量的匹配项
- _indices
  名称以 _indices 结尾的迭代器会生成通常的迭代值和在此slice 中的字节偏移量组成的值对。


搜索文本的模式

标准库支持 4 种主要的模式。
- 以 char 作为模式意味着要匹配该字符。
- 以 String、&str 或 &&str 作为模式，意味着要匹配等于该模式的子串。
- 以 FnMut(char) -> bool 闭包作为模式，意味着要匹配该闭包返回 true 的单个字符。
- 以 `&[char]`（注意并不是 &str，而是 char 的切片）作为模式，意味着要匹配该列表中出现的任何单个字符。

搜索和替换
- slice.contains(pattern)（包含）
- slice.starts_with(pattern)（以 pattern 开头）
- slice.ends_with(pattern)（以 pattern 结尾）
- slice.find(pattern)（查找）
- slice.rfind(pattern)（右起查找）
- slice.replace(pattern, replacement)（替换）
- slice.replacen(pattern, replacement, n)（替换 n 次）

遍历文本
- slice.chars()（字符迭代器）
- slice.char_indices()（字符及其偏移量迭代器）
- slice.bytes()（字节迭代器）
- slice.lines()（文本行迭代器）
- slice.split(pattern)（拆分）
- slice.rsplit(pattern)（右起拆分）
- slice.split_terminator(pattern)（终结符拆分）
- slice.rsplit_terminator(pattern)（右起终结符拆分）
- slice.splitn(n, pattern)（拆分为 n 片）
- slice.rsplitn(n, pattern)（右起拆分为 n 片）
- slice.split_whitespace()（按空白字符拆分）
- slice.split_ascii_whitespace()（按 ASCII 空白字符拆分）
- slice.matches(pattern)（匹配项）
- slice.match_indices(pattern)（匹配项及其偏移量）
- slice.rmatch_indices(pattern)（右起匹配项及其偏移量）


修剪
- slice.trim()（修剪）
- slice.trim_matches(pattern)（按匹配修剪）
- slice.strip_prefix(pattern)（剥离前缀）
- slice.strip_suffix(pattern)（剥离后缀）


字符串的大小写转换
slice.to_uppercase() 方法和 slice.to_lowercase() 方法会返回一个新分配的字符串
结果的长度可能与 slice 不同


从字符串中解析出其他类型

如果一个类型实现了 std::str::FromStr 特型，那它就提供了一种从字符串切片中解析出值的标准方法
```rust
pub trait FromStr: Sized {
  type Err;
  fn from_str(s: &str) -> Result<Self, Self::Err>;
}
```

所有常见的机器类型都实现了 FromStr

std::net::IpAddr 类型，即包含 IPv4 或 IPv6 互联网地址的enum，同样实现了 FromStr

字符串切片有一个 parse 方法，该方法可以将切片解析为你想要的任何类型——只要它实现了 FromStr
`let address = "fe80::0000:3ea9:f4ff:fe34:7a50".parse::<IpAddr>()?;`


将其他类型转换为字符串

- std::fmt::Display 特型
  该特型允许在 format! 宏的格式中使用 {} 格式说明符
- 如果一个类型实现了 Display，那么标准库就会自动为它实现std::str::ToString 特型
  `assert_eq!(address.to_string(), "fe80::3ea9:f4ff:fe34:7a50");`
- 标准库中的每个公共类型都实现了 std::fmt::Debug，这个特型会接受一个值并将其格式化为对程序员有用的字符串。
  用Debug 生成字符串的最简单方法是使用 format! 宏的 `{:?}` 格式说明符


借用其他类似文本的类型
- 切片和 String 都实现了 `AsRef<str>、AsRef<[u8]>、AsRef<Path> 和 AsRef<OsStr>`。
- 切片和字符串还实现了 `std::borrow::Borrow<str>` 特型。
  HashMap 和 BTreeMap 会借助 Borrow 令 String 很好地用作表中的键


以 UTF-8 格式访问文本
- slice.as_bytes()（用作字节切片）
  把 slice 的字节借入为 &[u8]。不是可变引用
- string.into_bytes()（转为字节切片）
  获取 string 的所有权并按值返回字符串字节的 `Vec<u8>`。
  这是一个开销极低的转换，因为它只是移动了字符串一直用作缓冲区的`Vec<u8>`。



从 UTF-8 数据生成文本
- str::from_utf8(byte_slice)（来自 utf8 切片）
- String::from_utf8(vec)（来自 utf8 向量）
- String::from_utf8_lossy(byte_slice)（来自 utf8，宽松版）
- String::from_utf8_unchecked(vec)（来自 utf8，不检查版）
- str::from_utf8_unchecked(byte_slice)（来自 utf8，不检查版）


推迟分配

更改 get_name 以返回 Cow
```rust
use std::borrow::Cow;
fn get_name() -> Cow<'static, str> {
  std::env::var("USER")
    .map(|v| Cow::Owned(v))
    .unwrap_or(Cow::Borrowed("whoever you are"))
}
```


把字符串当作泛型集合

String 同时实现了 std::default::Default 和 std::iter::Extend：
default 返回空字符串，而 
extend 可以把字符、字符串切片、Cow<..., str> 或字符串追加到一个字符串尾部。

&str 类型也实现了 Default，返回一个空切片。



## 17.4 格式化各种值

- format! 宏会用它来构建 String。
- println! 宏和 print! 宏会将格式化后的文本写入标准输出流。
- writeln! 宏和 write! 宏会将格式化后的文本写入指定的输出流。
- panic! 宏会使用它构建一个信息丰富的异常终止描述。

Rust 格式化工具的设计是开放式的。
你可以通过实现 std::fmt 模块的格式化特型来扩展这些宏以支持自己的类型。
也可以使用format_args! 宏和 std::fmt::Arguments 类型来让自己的函数和宏支持格式化语言。

格式化宏总会借入对其参数的共享引用，但永远不会拥有或修改它们

number of {}: {}
from {1} to {0}
v = {:?}
name = {:?}
{:8.2} km/s
{:20} {:02x} {:02x}
{1:02x} {2:02x} {0}
{lsb:02x} {msb:02x} {insn}    。。参数名称
{:02?}
{:02x?}


。。跳。



{:#?}  格式化，而不是一行

格式化指针
println!("pointers: {:p}, {:p}, {:p}", original, cloned, impostor);

20个字符宽
`format!("{:>20}", content)`

动态宽度
`format!("{:>1$}", content, get_width())`
1$ 就是在告诉 format! 使用第二个参数的值作为宽度



## 17.5 正则表达式

外部的 regex crate 是 Rust 的官方正则表达式库，它提供了通常的搜索函数和匹配函数

Cargo.toml
`regex = "1"`


Regex::new 构造函数会尝试将 &str 解析为正则表达式，并返回一个 Result

```rust
use regex::Regex;

// 注意，使用原始字符串语法r"..."是为了避免一大堆反斜杠
let semver = Regex::new(r"(\d+)\.(\d+)\.(\d+)(-[-.[:alnum:]]*)?")?;
// 简单搜索，返回布尔型结果
let haystack = r#"regex = "0.2.5""#;
assert!(semver.is_match(haystack));
```

Regex::captures 方法会在字符串中搜索第一个匹配项并返回一个regex::Captures 值，其中包含表达式中每个组的匹配信息

```rust
// 可以检索各个捕获组：
let captures = semver.captures(haystack).ok_or("semver regex should have matched")?;
assert_eq!(&captures[0], "0.2.5");
assert_eq!(&captures[1], "0");
assert_eq!(&captures[2], "2");
assert_eq!(&captures[3], "5");
```

```rust
assert_eq!(captures.get(4), None);
assert_eq!(captures.get(3).unwrap().start(), 13);
assert_eq!(captures.get(3).unwrap().end(), 14);
assert_eq!(captures.get(3).unwrap().as_str(), "5");
```

```rust
let haystack = "In the beginning, there was 1.0.0. \
 For a while, we used 1.0.1-beta, \
 but in the end, we settled on 1.2.4.";
let matches: Vec<&str> = semver.find_iter(haystack)
  .map(|match_| match_.as_str())
  .collect();
assert_eq!(matches, vec!["1.0.0", "1.0.1-beta", "1.2.4"]);
```


17.5.2 惰性构建正则表达式值

Regex::new 构造函数的开销可能很高：在速度较快的开发机器上为1200 个字符的正则表达式构造一个 Regex 会花费差不多 1 毫秒时间，即使是一个微不足道的表达式也要花费几微秒时间

最好让Regex 构造远离繁重的计算循环，这就意味着应该只构建一次 Regex，然后重复使用它。

lazy_static crate 提供了一种在首次使用时惰性构造静态值的好办法

```rust
[dependencies]
lazy_static = "1"
```

这个 crate 提供了一个宏来声明这样的变量：

```rust
use lazy_static::lazy_static;
lazy_static! {
  static ref SEMVER: Regex = Regex::new(r"(\d+)\.(\d+)\.(\d+)(-[-.[:alnum:]]*)?").expect("error parsing regex");
}
```
该宏会扩展成名为 SEMVER 的静态变量的声明，但其类型不完全是Regex，而是一个实现了 `Deref<Target=Regex>` 的由宏生成的类型，并公开了与 Regex 相同的全部方法。

```rust
use std::io::BufRead;
let stdin = std::io::stdin();
for line_result in stdin.lock().lines() {
  let line = line_result?;
  if let Some(match_) = SEMVER.find(&line) {
    println!("{}", match_.as_str());
  }
}
```

可以把 lazy_static! 声明放在模块中，甚至可以放在使用 Regex 的函数内部（如果这就是最合适的作用域的话）。


## 17.6 规范化

作为 Rust 的 &str 值或 String 值，"th\u{e9}" 和 "the\u{301}" 是完全不同的。

Unicode 指定了字符串的规范化形式。
每当根据 Unicode规则应将两个字符串视为等同时，它们的规范化形式是逐字符全同的。
当使用 UTF-8 编码时，它们是逐字节全同的。
这意味着可以使用== 来比较规范化后的字符串，可以将它们用作 HashMap 或HashSet 中的键，等等，这样就能获得 Unicode 规定的相等性概念了。

如果未做规范化，则甚至会产生安全隐患。
如果你的网站对用户名在某些情况下做了规范化，但在其他情况下未做规范化，那么最终可能会出现两个名为 bananasflambé 的不同用户，你的一部分代码会将其视为同一用户，但另一部分代码会认为这是两个用户，导致一个人的权限被错误地扩展到另一个人身上。
当然，有很多方法可以避开这种问题，但历史表明也有很多方法不能避开。


Unicode 定义了 4 种规范化形式
回答两个问题
- 第一个问题是：你更喜欢让字符尽可能组合还是尽可能分解？
- 如果两个字符序列表示相同的基础文本，但文本的格式化方式不同，那么你是要将它们视为等同的还是坚持认为有差异？


### 17.6.2 unicode-normalization crate

Rust 的 unicode-normalization crate 提供了一个特型，可以将方法添加到 &str 中，以便将文本转成四种规范化形式中的任何一种

```rust
use unicode_normalization::UnicodeNormalization;
// 不管左边的字符串使用哪种表示形式（无法仅仅通过观察得知），这些断言都成立
assert_eq!("Phở".nfd().collect::<String>(), "Pho\u{31b}\u{309}");
assert_eq!("Phở".nfc().collect::<String>(), "Ph\u{1edf}");

// 左侧使用了"ffi"连字符
assert_eq!("① Di\u{fb03}culty".nfkc().collect::<String>(), "1Difficulty");
```








# ch18 输入与输出 722

Rust 标准库中的输入和输出的特性是围绕 3 个特型组织的，即 Read、BufRead 和 Write。

读取器，缓冲读取器，写入器

- Read
  - Stdin
  - File
  - TcpStream
  - BufRead
    - `BufReader<R>`
    - `Cursor<&[u8]>`
    - StdinLock
- Write
  - Stdout
  - Stderr
  - File
  - TcpStream
  - `Vec<u8>`
  - `BufWriter<W>`


## 18.1 读取器与写入器

从读取器中读取一些字节
- std::fs::File::open(filename) 打开的文件
- std::net::TcpStream，用于通过网络接收数据
- std::io::stdin()，用于从进程的标准输入流中进行读取
- `std::io::Cursor<&[u8]> 值和std::io::Cursor<Vec<u8>> 值`，它们是从已存在于内存中的字节数组或向量中进行“读取”的读取器

写入器则可以向其中写入一些字节
- std::fs::File::create(filename) 打开的文件
- std::net::TcpStream，用于通过网络发送数据
- std::io::stdout() 和 std::io:stderr()，用于写入到终端；
- `Vec<u8>` 也是一个写入器，它的 write 方法会把内容追加到向量；
- `std::io::Cursor<Vec<u8>>`，它与 `Vec<u8>` 很像，但允许读取数据和写入数据，并能在向量中寻找不同的位置
- `std::io::Cursor<&mut [u8]>`，它与`std::io::Cursor<Vec<u8>>` 很像，不过不能增长缓冲区，因为它只是一些现有字节数组的切片。


读取器和写入器都有标准特型（std::io::Read 和 std::io::Write）


下面是一个将所有字节从任意读取器复制到任意写入器的函数：

```rust
use std::io::{self, Read, Write, ErrorKind};
const DEFAULT_BUF_SIZE: usize = 8 * 1024;
pub fn copy<R: ?Sized, W: ?Sized>(reader: &mut R, writer: &mut W) -> io::Result<u64> 
  where R: Read, W: Write
{
  let mut buf = [0; DEFAULT_BUF_SIZE];
  let mut written = 0;
  loop {
    let len = match reader.read(&mut buf) {
      Ok(0) => return Ok(written),
      Ok(len) => len,
      Err(ref e) if e.kind() == ErrorKind::Interrupted => continue,
      Err(e) => return Err(e),
    };
    writer.write_all(&buf[..len])?;
    written += len as u64;
  }
}
```
这是 Rust 标准库中 std::io::copy() 的实现

`use std::io::{self, Read, Write, ErrorKind};`
此处的 self 关键字将 io 声明成了 std::io 模块的别名


18.1.1 读取器

- reader.read(&mut buffer)（读取）
  buffer 参数的类型是 `&mut[u8]`。此方法最多会读取 buffer.len() 字节
- reader.read_to_end(&mut byte_vec)（读至末尾）
  从这个读取器读取所有剩余的输入，将其追加到 `Vec<u8>` 型的byte_vec 中。
  返回 `io::Result<usize>`，即已读取的字节数。
- reader.read_to_string(&mut string)（读取字符串）
- reader.read_exact(&mut buf)（精确读满）
- reader.bytes()（字节迭代器）
- reader.chain(reader2)（串联）
- reader.take(n)（取出 n 个）


18.1.2 缓冲读取器
- reader.read_line(&mut line)（读一行）
- reader.lines()（文本行迭代器）
- reader.read_until(stop_byte, &mut byte_vec)（读到stop_byte 为止）
- reader.split(stop_byte)（根据stop_byte 拆分）

BufRead 还提供了 .fill_buf() 和 .consume(n)，这是一对底层方法，用于直接访问读取器的内部缓冲区。


18.1.3 读取行

用于实现 Unix grep 实用程序的函数，该函数会在多行文本（通常是通过管道从另一条命令输入的文本）中搜索给定字符串：

```rust
use std::io;
use std::io::prelude::*;
fn grep(target: &str) -> io::Result<()> {
  let stdin = io::stdin();
  for line_result in stdin.lock().lines() {
    let line = line_result?;
    if line.contains(target) {
      println!("{}", line);
    }
  }
  Ok(())
}
```

Rust 标准库使用互斥锁保护着 stdin。
因此要调用.lock() 来锁定 stdin 以供当前线程独占使用，这会返回一个实现了 BufRead 的 StdinLock 值。
在循环结束时，StdinLock 会被丢弃，释放互斥锁。


File 不会自动缓冲。
File 实现了 Read 但没实现BufRead。
但是，为 File 或任意无缓冲读取器创建缓冲读取器很容易。
BufReader::new(reader) 就是做这个的。

```rust
// grep——搜索stdin或其他文件，以便用给定的字符串进行逐行匹配
use std::error::Error;
use std::io::{self, BufReader};
use std::io::prelude::*;
use std::fs::File;
use std::path::PathBuf;

fn grep<R>(target: &str, reader: R) -> io::Result<()> 
  where R: BufRead 
{
  for line_result in reader.lines() {
    let line = line_result?;
    if line.contains(target) {
      println!("{}", line);
    }
  }
  Ok(())
}

fn grep_main() -> Result<(), Box<dyn Error>> {
  // 获取命令行参数。第一个参数是要搜索的字符串，其他参数是一些文件名
  let mut args = std::env::args().skip(1);
  let target = match args.next() {
    Some(s) => s,
    None => Err("usage: grep PATTERN FILE...")?
  };
  let files: Vec<PathBuf> = args.map(PathBuf::from).collect();
  if files.is_empty() {
    let stdin = io::stdin();
    grep(&target, stdin.lock())?;
  } else {
    for file in files {
      let f = File::open(file)?;
      grep(&target, BufReader::new(f))?;
    }
  }
  Ok(())
}

fn main() {
  let result = grep_main();
  if let Err(err) = result {
    eprintln!("{}", err);
    std::process::exit(1);
  }
}
```


18.1.4 收集行

有些读取器方法（包括 .lines()）会返回生成 Result 值的迭代器。
当你第一次想要将文件的所有行都收集到一个大型向量中时，就会遇到如何摆脱 Result 的问题
`let lines = reader.lines().collect::<io::Result<Vec<String>>>()?;`
上面可以，是因为 标准库中包含了 Result 对 FromIterator 的实现


18.1.5 写入器

要将输出发送到写入器，请使用 write!() 宏和 writeln!() 宏。
它们和 print!() 和 println!() 类似，但有两点区别：
- 每个 write 宏都接受一个额外的写入器作为第一参数。
- 它们会返回 Result，因此必须处理错误。这就是为什么要在每行末尾使用 ? 运算符。

Write 特型有以下几个方法
- writer.write(&buf)（写入）
- writer.write_all(&buf)（写入全部）
- writer.flush()（刷新缓冲区）

写入器也会在被丢弃时自动关闭。

BufWriter::new(writer) 也会为任意写入器添加缓冲区
```rust
let file = File::create("tmp.txt")?;
let writer = BufWriter::new(file);
```
要设置缓冲区的大小，请使用
BufWriter::with_capacity(size, writer)

请在丢弃带缓冲的写入器之前将它手动 .flush() 一下


18.1.6 文件

- File::open(filename)（打开）
- File::create(filename)（创建）

OpenOptions

```rust
use std::fs::OpenOptions;
let log = OpenOptions::new()
  .append(true) // 如果文件已存在，则追加到末尾
  .open("server.log")?

let file = OpenOptions::new()
  .write(true)
  .create_new(true) // 如果文件已存在，则失败
  .open("new_file.txt")?;
```


18.1.7 寻址

File 还实现了 Seek 特型，这意味着你可以在 File 中“跳来跳去”

```rust
pub trait Seek {
  fn seek(&mut self, pos: SeekFrom) -> io::Result<u64>;
}
pub enum SeekFrom {
  Start(u64),
  End(i64),
  Current(i64)
}
```

可以用 file.seek(SeekFrom::Start(0)) 倒回到开头，还能用 file.seek(SeekFrom::Current(-8)) 回退几字节

在文件中寻址很慢。无论使用的是硬盘还是固态驱动器（SSD），每一次寻址的开销都接近于读取数兆字节的数据。


18.1.8 其他读取器与写入器类型

- io::stdin()（标准输入）
  Stdin 有一个 .lock() 方法，该方法会获取互斥锁并返回io::StdinLock，这是一个带缓冲的读取器，在被丢弃之前会持有互斥锁。
- io::stdout()（标准输出）和 io::stderr()（标准错误）
- `Vec<u8>`（u8 向量）
- Cursor::new(buf)（新建）
- std::net::TcpStream（Tcp 流）
- std::process::Command（命令）
- io::sink()（地漏）
  无操作写入器。所有的写入方法都会返回 Ok，但只是把数据扔掉了。
- io::empty()（空白）
  这是无操作读取器。读取总会成功，但只会返回“输入结束”（EOF）。
- io::repeat(byte)（重复）
  返回一个会无限重复给定字节的读取器。


18.1.9 二进制数据、压缩和序列化

byteorder crate 提供了 ReadBytesExt 特型和WriteBytesExt 特型，为所有读取器和写入器添加了二进制输入和输出的方法

flate2 crate 提供了用于读取和写入 gzip 数据的适配器方法

serde crate 及其关联的格式类 crate（如 serde_json）实现了序列化和反序列化



## 18.2 文件与目录

18.2.1 OsStr 与 Path

操作系统并不会强制要求其文件名是有效的 Unicode

文件名在实践中几乎总是Unicode，但 Rust 必须以某种方式处理==罕见的例外情况==。==这就是==Rust 会有 std::ffi::OsStr 和 OsString 的原因。

OsStr 是一种字符串类型，它是 UTF-8 的超集。
OsStr 的任务是表示当前系统上的所有文件名、命令行参数和环境变量，无论它们是不是有效的 Unicode。


- Path::new(str)（新建）
- path.parent()（父目录）
- path.file_name()（文件名）
- path.is_absolute()（是绝对路径？）
- path.is_relative()（是相对路径？）
- path1.join(path2)（联结）
- path.components()（组件迭代器）
- path.ancestors()（祖先迭代器）
- path.to_str()（转字符串）
- path.to_string_lossy()（转字符串，宽松版）
- path.display()（转显示）


18.2.3 访问文件系统的函数

std::fs 中的一些函数
- create_dir(path)
- create_dir_all(path)
- remove_dir(path)
- remove_dir_all(path)
- remove_file(path)

- `copy(src_path, dest_path) -> Result<u64>`
- rename(src_path, dest_path)
- hard_link(src_path, dest_path)

- `canonicalize(path) -> Result<PathBuf>`
- `metadata(path) -> Result<Metadata>`
- `symlink_metadata(path) -> Result<Metadata>`
- `read_dir(path) -> Result<ReadDir>`
- `read_link(path) -> Result<PathBuf>`

- set_permissions(path, perm)


18.2.4 读取目录

std::fs::read_dir 或 Path 中的等效方法 .read_dir()：

```rust
for entry_result in path.read_dir()? {
  let entry = entry_result?;
  println!("{}", entry.file_name().to_string_lossy());
}
```

entry 的类型是 std::fs::DirEntry，这个结构体提供了数个方法。
- entry.file_name()（文件名）
- entry.path()（路径）
- entry.file_type()（文件类型）
- entry.metadata()（元数据）

特殊目录 . 和 .. 在读取目录时不会列出。


18.2.5 特定于平台的特性

std::os 在标准库中的实际主体如下所示
```rust
//! 特定于操作系统的功能
#[cfg(unix)] pub mod unix;
#[cfg(windows)] pub mod windows;
#[cfg(target_os = "ios")] pub mod ios;
#[cfg(target_os = "linux")] pub mod linux;
#[cfg(target_os = "macos")] pub mod macos;
...
```


## 18.3 网络

关于网络的教程远远超出了本书的范畴

要编写较底层的网络代码，可以使用 std::net 模块，该模块为TCP 网络和 UDP 网络提供了跨平台支持。可以使用 `native_tls` crate 来支持 SSL/TLS。


更高层级的协议由第三方 crate 提供支持。例如，reqwest crate 为 HTTP 客户端提供了一个漂亮的 API。

actix-web 框架为 HTTP 服务器提供了一些高层次抽象

websocket crate 实现了 WebSocket 协议



# ch19 并发 759

如果还没有对多线程技术彻底失望，那么至少也应该对所有多线程代码保持适度的警惕

很多惯用法。系统程序员常用的方法包括以下几种
- 具有单一作业的后台线程，需要定期唤醒执行作业。
- 通过任务队列与客户端通信的通用工作池。
- 管道，数据在其中从一个线程流向下一个线程，每个线程只负责一部分工作
- 数据并行处理，假设（无论对错）整个计算机只进行一次主要的大型计算，将这次计算分成 n 个部分且在 n 个线程上运行，并期望机器的所有 n 个核心都能立即开始工作
- 同步复杂对象关系，其中多个线程可以访问相同的数据，并且使用基于互斥锁等底层原语的临时加锁方案避免了竞争。（Java内置了对此模型的支持，它曾在 20 世纪 90 年代和 21 世纪初非常流行。）
  。。。？ 曾？？？
- 原子化整数操作允许多个核心借助一个机器字大小的字段传递信息来进行通信。（这种惯用法比其他所有方法更难正确使用，除非要交换的数据恰好是整数值，但实际上，数据通常是指针。）


Rust 提供了一种更好的并发处理方式，不是强制所有程序采用单一风格（对系统程序员来说这可算不上什么解决方案），而是安全地支持多种风格

3 种使用 Rust 线程的方法。
- 分叉与合并（fork-join）并行
- 通道
- 共享可变状态


## 19.1 分叉与合并并行

假设我们正在对大量文档进行自然语言处理。可以写这样一个循环：
```rust
fn process_files(filenames: Vec<String>) -> io::Result<()> {
  for document in filenames {
    let text = load(&document)?; // 读取源文件
    let results = process(text); // 计算统计信息
    save(&document, results)?; // 写入输出文件
  }
  Ok(())
}
```

由于每个文档都是单独处理的，因此要想加快任务处理速度，可以将语料库分成多个块并在单独的线程上处理每个块

出于以下几个原因，分叉与合并并行很有吸引力。
- 非常简单。分叉与合并很容易实现
- 避免了瓶颈。分叉与合并中没有对共享资源的锁定。
  任何线程只会在最后一步才不得不等待另一个线程。同时，每个线程都可以自由运行。这有助于降低任务切换开销。
- 这种模式在性能方面的数学模型对程序员来说比较直观
  在最好的情况下，通过启动 4 个线程，我们只花 1/4 的时间就能完成原本的工作。
- 很容易推断出程序是否正确。

分叉与合并的主要缺点是要求工作单元彼此隔离


### 19.1.1 线程启动与联结

函数 std::thread::spawn 会启动一个新线程
```rust
use std::thread;
thread::spawn(|| {
  println!("hello from a child thread");
});
```

它会接受一个参数，即一个 FnOnce 闭包或函数型的参数。Rust 会启动一个新线程来运行该闭包或函数的代码。

使用 spawn 实现了之前的 process_files 函数的并行版本

```rust
use std::{thread, io};
fn process_files_in_parallel(filenames: Vec<String>) -> io::Result<()> {
  // 把工作拆分成几块
  const NTHREADS: usize = 8;
  let worklists = split_vec_into_chunks(filenames, NTHREADS);
  // 分叉：启动一个线程来处理每一个块
  let mut thread_handles = vec![];
  for worklist in worklists {
    thread_handles.push(
      thread::spawn(move || process_files(worklist))
    );
  }
  // 联结：等待所有线程结束
  for handle in thread_handles {
    handle.join().unwrap()?;
  }
  Ok(())
}
```

请注意我们是如何将文件名列表放入工作线程的。
- 在父线程中，通过 for 循环来定义和填充 worklist。
- 一旦创建了 move 闭包，worklist 就会被移动到此闭包中。
- 然后 spawn 会将闭包（内含 worklist 向量）转移给新的子线程。




### 19.1.2 跨线程错误处理

`handle.join().unwrap()?;`

handle.join() 会返回 std::thread::Result

线程之间的边界充当着 panic 的防火墙，panic 不会自动从一个线程传播到依赖它的其他线程

展开（unwrap）thread::Result 之后，我们就用io::Result 上的 ? 运算符显式地将 I/O 错误从子线程传播到了父线程。



### 19.1.3 跨线程共享不可变数据

假设我们正在进行的分析需要一个大型的英语单词和短语的数据库

```rust
// 之前
fn process_files(filenames: Vec<String>)

// 之后
fn process_files(filenames: Vec<String>, glossary: &GigabyteMap)
```
这个 glossary 会很大，所以要通过引用传递它

原子化引用计数
```rust
use std::sync::Arc;
fn process_files_in_parallel(filenames: Vec<String>, glossary: Arc<GigabyteMap>) -> io::Result<()>
{
  ...
  for worklist in worklists {
    // 对.clone()的调用只会克隆Arc并增加引用计数，并不会克隆 GigabyteMap
    let glossary_for_child = glossary.clone();
    thread_handles.push(
      spawn(move || process_files(worklist, &glossary_for_child))
    );
  }
  ...
}
```

### 19.1.4 rayon

标准库的 spawn 函数是一个重要的基础构件，但它并不是专门为分叉与合并的并行而设计的

在第 2 章中，我们使用 crossbeam 库将一些工作拆分为 8 个线程

由 Niko Matsakis 和 Josh Stone 设计的 rayon 库是另一个例子。
它提供了两种运行并发任务的方式：
```rust
use rayon::prelude::*;

// “并行做两件事”
let (v1, v2) = rayon::join(fn1, fn2);

// “并行做N件事”
giant_vector.par_iter().for_each(|value| {
  do_thing_with_value(value);
});
```

rayon::join(fn1, fn2) 只是调用这两个函数并返回两个结果。
.par_iter() 方法会创建 ParallelIterator，这是一个带有 map、filter 和其他方法的值，很像 Rust 的 Iterator。


使用 rayon 的 process_files_in_parallel 版本和一个接受 `Vec<String>` 型而非 &str 型参数的 process_file

```rust
use rayon::prelude::*;
fn process_files_in_parallel(filenames: Vec<String>, glossary: &GigabyteMap) -> io::Result<()>
{
  filenames.par_iter()
    .map(|filename| process_file(filename, glossary)) // 并发执行
    .reduce_with(|r1, r2| {   // 合并结果
      if r1.is_err() { r1 } else { r2 }
    })
    .unwrap_or(Ok(()))
}
```

rayon 使用了一种叫作==工作窃取==的技术来动态平衡线程间的工作负载。

rayon 还支持跨线程共享引用。幕后发生的任何并行处理都能确保在 reduce_with 返回时完成。


### 19.1.5 重温曼德博集

事实上，图像的浅灰色部分（循环会快速退出的地方）比黑色部分（循环会运行整整 255 次迭代的地方）渲染速度要快得多。
虽然我们将整个区域划分成了大小相等的水平条带，但创建了不均等的工作负载


使用 rayon 很容易解决这个问题。
我们可以为输出中的每一行像素启动一个并行任务。
这会创建数百个任务，而 rayon 可以在其线程中分配这些任务。
有了工作窃取机制，任务的规模是无关紧要的。
rayon 会对这些工作进行平衡。

```rust
let mut pixels = vec![0; bounds.0 * bounds.1];
// 把`pixels`拆分成一些水平条带
{
  let bands: Vec<(usize, &mut [u8])> = pixels
    .chunks_mut(bounds.0)
    .enumerate()
    .collect();
  bands.into_par_iter()
    .for_each(|(i, band)| {
      let top = i;
      let band_bounds = (bounds.0, 1);
      let band_upper_left = pixel_to_point(bounds, (0, top),  upper_left, lower_right);
      let band_lower_right = pixel_to_point(bounds, (bounds.0, top + 1), upper_left, lower_right);
      render(band, band_bounds, band_upper_left, band_lower_right);
    });
}
write_image(&args[1], &pixels, bounds).expect("error writing PNG file");
```

速度比以前手动分配工作时快 75%，而且代码更简短


## 19.2 通道 mpsc

通道是一种单向管道，用于将值从一个线程发送到另一个线程。
换句话说，通道是一个线程安全的队列。

sender.send(item) 会将单个值放入通道，
receiver.recv()则会移除一个值。
值的所有权会从发送线程转移给接收线程。

如果通道为空，则 receiver.recv() 会一直阻塞到有值发出为止。


### 19.2.1 发送值

我们将使用通道来构建一个创建倒排索引的并发程序，倒排索引是搜索引擎的关键组成部分之一。
每个搜索引擎都会处理特定的文档集合。倒排索引是记录“哪些词出现在哪里”的数据库。

我们将展示与线程和通道有关的部分代码。完整的程序（参见本书在GitHub 网站上的页面）也不长，大约 1000 行代码。


#### 10g文件找xx
程序使用总共 5 个线程分别执行了不同的任务
读取，索引，内存中合并，写入，文件合并

。。就是那个面试题啊，10g文件找xx的那种。

启动读取文件线程的代码
```rust
use std::{fs, thread};
use std::sync::mpsc;
let (sender, receiver) = mpsc::channel(); // 创建通道
let handle = thread::spawn(move || {
  for filename in documents {
    let text = fs::read_to_string(filename)?;
    if sender.send(text).is_err() { // 将数据移入通道，只复制3个机器字(String结构体的大小)
      break;
    }
  }
  Ok(())
});
```

通道是 std::sync::mpsc 模块的一部分

通道是有类型的。

我们要使用这个通道来发送每个文件的文本，因此sender 和 receiver 的类型分别为 `Sender<String> 和 Receiver<String>`。
固然可以写成 `mpsc::channel::<String>()` 来明确请求一个字符串型通道。
但最好还是让 Rust 的类型推断来解决这个问题。

。。怎么 auto 这么流行 且 推荐。。

send 方法和 recv 方法都会返回 Result，这两种方法只有当通道的另一端已被丢弃时才会失败。


### 19.2.2 接收值

第二个线程来运行调用 receiver.recv() 的循环

```rust
while let Ok(text) = receiver.recv() {
  do_something_with(text);
}
```

但 Receiver 是可迭代的，所以还有更好的写法
```rust
for text in receiver {
  do_something_with(text);
}
```

```rust
// texts 就是 Receiver
fn start_file_indexing_thread(texts: mpsc::Receiver<String>) -> (mpsc::Receiver<InMemoryIndex>, thread::JoinHandle<()>)
{
  let (sender, receiver) = mpsc::channel();
  let handle = thread::spawn(move || {
    for (doc_id, text) in texts.into_iter().enumerate() {
      let index = InMemoryIndex::from_single_document(doc_id, text);
      if sender.send(index).is_err() {
        break;
      }
    }
  });
  (receiver, handle)
}
```

该线程会从一个通道（texts）接收String 值并将 InMemoryIndex 值发送给另一个通道（sender/receiver）。
这个线程的工作是获取第一阶段加载的每个文件，并将每个文档变成一个小型单文件内存倒排索引。

索引文档的所有工作都是由函数 InMemoryIndex::from_single_document 完成的。我们不会在这里展示它的源代码


### 19.2.3 运行管道

其余 3 个阶段的设计也是类似的。
每个阶段都会使用上一阶段创建的Receiver。

我们设定的目标是将所有小索引合并到磁盘上的单个大索引文件中。
最快的方法是将这个任务分为 3 个阶段。
我们不会在这里展示代码，只会展示这 3 个函数的类型签名

首先，合并内存中的索引，直到它们变得“笨重”（第三阶段）
```rust
fn start_in_memory_merge_thread(file_indexes: mpsc::Receiver<InMemoryIndex>) -> (mpsc::Receiver<InMemoryIndex>, thread::JoinHandle<()>)
```

将这些大型索引写入磁盘（第四阶段）
```rust
fn start_index_writer_thread(big_indexes: mpsc::Receiver<InMemoryIndex>, output_dir: &Path) -> (mpsc::Receiver<PathBuf>, thread::JoinHandle<io::Result<()>>)
```

如果有多个大文件，就用基于文件的合并算法合并它们（第五阶段）
```rust
fn merge_index_files(files: mpsc::Receiver<PathBuf>, output_dir: &Path) -> io::Result<()>
```

启动线程和检查错误的代码
```rust
fn run_pipeline(documents: Vec<PathBuf>, output_dir: PathBuf) -> io::Result<()> {
  // 启动管道的所有5个阶段
  let (texts, h1) = start_file_reader_thread(documents);
  let (pints, h2) = start_file_indexing_thread(texts);  // texts 来自上一步
  let (gallons, h3) = start_in_memory_merge_thread(pints);
  let (files, h4) = start_index_writer_thread(gallons, &output_dir);
  let result = merge_index_files(files, &output_dir);
  // 等待这些线程结束，保留它们遇到的任何错误
  let r1 = h1.join().unwrap();
  h2.join().unwrap();
  h3.join().unwrap();
  let r4 = h4.join().unwrap();
  // 返回遇到的第一个错误（如果有的话）（如你所见，h2和h3
  // 不会失败，因为这些线程都是纯粹的内存数据处理）
  r1?;
  r4?;
  result
}
```

管道就像制造业工厂中的装配流水线，其性能受限于最慢阶段的吞吐量。

在这个例子中，测量表明第二阶段是瓶颈。
我们的索引线程使用了 .to_lowercase() 和.is_alphanumeric()，因此它会花费大量时间在 Unicode 表中查找。
对于索引下游的其他阶段，它们大部分时间在Receiver::recv 中休眠，等待输入。

这意味着应该还可以更快。只要解决了这些瓶颈，并行度就会提高。


### 19.2.4 通道的特性与性能 sync_channel

std::sync::mpsc 中的 mpsc 代表多生产者、单消费者

`Sender<T>` 实现了 Clone 特型。要获得具有多个发送者的通道，只需创建一个常规通道并根据需要多次克隆发送者即可。可以将每个Sender 值转移给不同的线程。

`Receiver<T>` 不能被克隆，所以如果需要让多个线程从同一个通道接收值，就需要使用 Mutex

Rust 的通道经过了精心优化。
首次创建通道时，Rust 会使用特殊的“==一次性==”队列实现。
如果只通过此通道发送一个对象，那么开销是最低的。
如果要发送第二个值，Rust 就会==切换到第二种队列实现==。
实际上，第二种实现就是为长期使用而设计的
如果你克隆了Sender，那么 Rust 就必须==回退到第三种实现==，使得多个线程可以安全地同时尝试发送值，这种实现是安全的。

应用程序很容易在通道性能方面犯一个==错误：发送值的速度快于接收值和处理值的速度==。这会导致通道中积压的值不断增长

Unix 使用了一个优雅的技巧来提供一些背压，以迫使超速的发送者放慢速度：Unix 系统上的每个管道都有固定的大小，如果进程试图写入暂时已满的管道，那么系统就会简单地阻塞该进程直到管道中有了空间。这在 Rust 中的等效设计称为同步通道

```rust
use std::sync::mpsc;
let (sender, receiver) = mpsc::sync_channel(1000);
```
创建时可以指定它能容纳多少个值


### 19.2.5 线程安全：Send 与 Sync

我们一直假定所有值都可以在线程之间自由移动和共享。

Rust 完整的线程安全故事取决于两个内置特型，即 std::marker::Send 和 std::marker::Sync

- 实现了 Send 的类型可以安全地按值传给另一个线程。它们可以跨线程移动。
- 实现了 Sync 的类型可以安全地将一个值的不可变引用传给另一个线程。它们可以跨线程共享。

如果结构体或枚举的所有字段都是 Send 的，那它自然是 Send 的；
如果结构体或枚举的所有字段都是 Sync 的，那它自然是 Sync 的。

有些类型是 Send 的但不是 Sync 的。这通常是刻意设计的，就像mpsc::Receiver 一样，它是为了保证 mpsc 通道的接收端一次只能被一个线程使用。

少数既不是 Send 也不是 Sync 的类型大多使用了非线程安全的可变性，比如引用计数智能指针类型 `std::rc::Rc<T>`

Send 和 Sync 会作为函数类型签名中的限界。
当你生成（spawn）一个线程时，传入的闭包必须实现了Send 特型


### 19.2.6 绝大多数迭代器能通过管道传给通道

```rust
use std::thread;
impl<T> OffThreadExt for T where T: Iterator + Send + 'static, T::Item: Send + 'static
{
  fn off_thread(self) -> mpsc::IntoIter<Self::Item> {
    // 创建一个通道把条目从工作线程中传出去
    let (sender, receiver) = mpsc::sync_channel(1024);
    // 把这个迭代器转移给新的工作线程，并在那里运行它
    thread::spawn(move || {
      for item in self {
        if sender.send(item).is_err() {
          break;
        }
      }
    });
    // 返回一个从通道中拉取值的迭代器
    receiver.into_iter()
  }
}
```


### 19.2.7 除管道之外的用法
日志记录
发送请求并要求返回某种响应


## 19.3 共享可变状态


### 19.3.1 什么是互斥锁

C++的一个例子。。
mutex.Acquire() 和 mutex.Release()
大意就是：
```text
void fun(person p)
{
  mutex.Acquire();
  players.push_back(p);

  if (players.size() >= 8)
  {
    vector<Player> p2;
    players.swap(p2);   // .!
    startGame(p2);
  }

  mutex.Release();
}
```

### 19.3.2 `Mutex<T>`

`type PlayerId = u32;`

```rust
const GAME_SIZE: usize = 8;
/// 等候列表永远不会超过GAME_SIZE个玩家
type WaitingList = Vec<PlayerId>;
```

```rust
use std::sync::Mutex;
/// 所有线程都可以共享对这个大型上下文结构体的访问
struct FernEmpireApp {
  ...
  waiting_list: Mutex<WaitingList>,
  ...
}
```

与 C++ 不同，在 Rust 中，受保护的数据存储于 Mutex 内部。
==建立此 Mutex== 的代码如下所示
```rust
use std::sync::Arc;
let app = Arc::new(FernEmpireApp {    // Arc
  ...
  waiting_list: Mutex::new(vec![]),   // Mutex
  ...
});
```

创建新的 Mutex 看起来就像创建新的 Box 或 Arc，但是 Box 和Arc 意味着堆分配，而 Mutex 仅与锁操作有关。

对整个应用程序使用 Arc::new，而仅对受保护的数据使用 Mutex::new。这两个类型经常一起使用，==Arc 用于跨线程共享数据==，而 ==Mutex 用于sync跨线程共享的可变数据==。

使用
```rust
impl FernEmpireApp {
  /// 往下一个游戏的等候列表中添加一个玩家。如果有足够
  /// 的待进入玩家，则立即启动一个新游戏
  fn join_waiting_list(&self, player: PlayerId) {
    // 锁定互斥锁，并授予内部数据的访问权。`guard`的作用域是一个临界区
    let mut guard = self.waiting_list.lock().unwrap();  // 阻塞直到获得互斥锁
    // 现在开始执行游戏逻辑
    guard.push(player);
    if guard.len() == GAME_SIZE {
      let players = guard.split_off(0);
      self.start_game(players);
    }
  }
}
```

.lock() 返回的 `MutexGuard<WaitingList>` 值是 `&mut WaitingList` 的浅层包装

通过 13.5 节讨论过的“隐式解引用”机制，我们可以直接在此守卫上调用 WaitingList 的各种方法

当 guard 被丢弃时，锁就被释放了。这通常会发生在块的末尾，但也可以手动丢弃

```rust
if guard.len() == GAME_SIZE {
  let players = guard.split_off(0);
  drop(guard); // 启动游戏时就不必锁定列表了
  self.start_game(players);
}
```

### 19.3.3 mut 与互斥锁

join_waiting_list 方法并没有通过可变引用获取 self
`fn join_waiting_list(&self, player: PlayerId)`

当你调用底层集合 `Vec<PlayerId>` 的 push 方法时，它确实需要一个可变引用
`pub fn push(&mut self, item: T)`

在 Rust 中，&mut 表示独占访问。普通 & 表示共享访问


我们习惯于把 &mut 访问从父级传到子级，从容器传到内容。
只有一开始你就拥有对 starships 的 &mut 引用 [ 或者你也可能拥有这些 starships（星舰）？如果是这样，那么……恭喜你成了埃隆 • 马斯克。]，才可以在 starships[id].engine 上调用 &mut self 方法。
这是默认设置，因为如果没有对父项的独占访问权，那么 Rust 通常无法确保你对子项拥有独占访问权。

但是 Mutex 有办法确保这一点：锁。
事实上，互斥锁只不过是提供对内部数据的独占（mut）访问的一种方法，即使有许多线程也在共享（非 mut）访问 Mutex 本身时，也能确保一切正常。


Rust 的类型系统会告诉我们 Mutex 在做什么。它在 动态地 强制执行独占访问，而这通常是由 Rust 编译器在编译期间 静态 完成的。



### 19.3.4 为什么互斥锁不是“银弹”

专门使用分叉与合并并行的程序具有确定性，不会死锁。

使用通道的程序几乎同样表现良好。那些专门供管道使用的通道（比如我们的索引构建器）也具有确定性：虽然消息传递的时机可能有所不同，但不会影响输出

安全的 Rust 代码不会引发数据竞争，这是一种特定类型的 bug，其中多个线程会同时读写同一内存，并产生无意义的结果。

使用互斥锁的线程会遇到 Rust 无法为你修复的另一些问题
- 有效的 Rust 程序不会有数据竞争，但仍然可能有其他竞态条件——程序的行为取决于各线程之间的运行时间长短，因此可能每次运行时都不一样。有些竞态条件是良性的，有些则表现为普遍的不稳定性和难以修复的 bug。以==非结构化方式使用互斥锁会引发竞态条件==。你需要确保竞态条件是良性的。
  。。什么是 非结构化方式？。。
- 共享可变状态也会影响程序设计。==通道==作为代码中的==抽象边界==，可以轻松地==拆出彼此隔离的组件==以进行测试，而==互斥锁则==会鼓励一种“只要==再添加一个方法==就行了”的工作方式，这可能会导致彼此有联系的==代码耦合==成一个单体。
- 互斥锁也并不像最初看起来那么简单。19.3.5 节和 19.3.6 节会详解介绍

要尽可能使用更结构化的方法，只在必要时使用 Mutex。
。。估计就是之前的 fork-join, 管道


### 19.3.5 死锁

线程在尝试获取自己正持有的锁时会让自己陷入死锁：

```rust
let mut guard1 = self.waiting_list.lock().unwrap();
let mut guard2 = self.waiting_list.lock().unwrap(); // 死锁
```

Mutex 中的锁并不是递归锁。

最好的保护是保持临界区尽可能小：进入，开始工作，完成后马上离开

通道也有可能陷入死锁。两个线程可能会互相阻塞，每个线程都在等待从另一个线程接收消息，但 良好的程序设计可以让你确信这在实践中不会发生


### 19.3.6 “中毒”的互斥锁

Mutex::lock() 返回 Result 的原因与 JoinHandle::join() 是一样的：如果另一个线程发生 panic，则可以优雅地失败。
当我们编写 handle.join().unwrap() 时，就是在告诉 Rust 将 panic 从一个线程传播到另一个线程。
mutex.lock().unwrap() 惯用法同样如此。

如果线程在持有 Mutex 期间出现 panic，则 Rust 会把 Mutex 标记为已“中毒”。
之后每当试图锁住已“中毒”的 Mutex 时都会得到错误结果。

你编程时一直在维护不变条件。
由于你的程序在未完成其正在执行的操作的情况下发生 panic 并退出临界区，可能更新了受保护数据的某些字段但未更新其他字段，因此==不变条件==现在有可能==已经被破坏==了。

你仍然可以锁定已“中毒”的互斥锁并访问其中的数据，完全强制运行互斥代码。
具体请参阅 PoisonError::into_inner() 的文档。


### 19.3.7 使用互斥锁的多消费者通道

Rust 的通道是多生产者、单一消费者

一个通道只能有一个 Receiver。
如果有一个线程池，则不能让其中的多个线程使用单个 mpsc 通道作为共享工作列表

有一种非常简单的解决方法，只要使用标准库的一点点“能力”就可以。
可以在 Receiver 周围包装一个 Mutex 然后再共享

```rust
pub mod shared_channel {
  use std::sync::{Arc, Mutex};
  use std::sync::mpsc::{channel, Sender, Receiver};

  /// 对`Receiver`的线程安全的包装
  #[derive(Clone)]
  pub struct SharedReceiver<T>(Arc<Mutex<Receiver<T>>>);

  impl<T> Iterator for SharedReceiver<T> {
    type Item = T;
    /// 从已包装的接收者中获取下一个条目
    fn next(&mut self) -> Option<T> {
      let guard = self.0.lock().unwrap();
      guard.recv().ok()
    }
  }

  /// 创建一个新通道，它的接收者可以跨线程共享。这会返回一个发送者和一个
  /// 接收者，就像标准库的 `channel()`，有时可以作为无缝替代品使用
  pub fn shared_channel<T>() -> (Sender<T>, SharedReceiver<T>) {
    let (sender, receiver) = channel();
    (sender, SharedReceiver(Arc::new(Mutex::new(receiver))))
  }
}
```

### 19.3.8 读/写锁（`RwLock<T>`）

std::sync

服务器程序通常都有一些只加载一次且很少更改的配置信息。
大多数线程只会查询此配置，但由于配置可以更改

如果配置没有改变，那么各个线程就不应该轮流查询配置。
这时就可以使用读 / 写锁或 RwLock

互斥锁只有一个 lock 方法，而读 / 写锁有两个，即 read 和 write。

```rust
use std::sync::RwLock;
struct FernEmpireApp {
  ...
  config: RwLock<AppConfig>,
  ...
}
```

```rust
/// 如果应该使用试验性的真菌代码，则为True
fn mushrooms_enabled(&self) -> bool {
  let config_guard = self.config.read().unwrap();
  config_guard.mushrooms_enabled
}
```

```rust
fn reload_config(&self) -> io::Result<()> {
  let new_config = AppConfig::load()?;
  let mut config_guard = self.config.write().unwrap();
  *config_guard = new_config;
  Ok(())
}
```

### 19.3.9 条件变量（Condvar）

std::sync::Condvar

通常线程需要一直等到某个条件变为真。

- 在==关闭服务器==的过程中，主线程可能需要等到所有其他线程都完成后才能退出。
- 当工作线程==无事可做==时，需要一直等待，直到有数据需要处理为止。
- 实现==分布式共识==协议的线程可能要等到一定数量的对等点给出响应为止。

Condvar 中有方法 .wait() 和 .notify_all() .notify_one()，其中 .wait() 会阻塞线程，直到其他线程调用了 .notify_all() 或 .notify_one()。

条件变量是关于受特定 Mutex 保护的某些数据的特定“真或假”条件。因此，Mutex 和 Condvar 是相关的。


当所需条件变为真时，就调用 Condvar::notify_all（或 notify_one）来唤醒所有等待的线程
`self.has_data_condvar.notify_all();`

要进入睡眠状态并等待条件变为真，可以使用 Condvar::wait()
```rust
while !guard.has_data() {
  guard = self.has_data_condvar.wait(guard).unwrap();
}
```

### 19.3.10 原子化类型

std::sync::atomic 模块

- AtomicIsize 和 AtomicUsize 是与单线程 isize 类型和 usize 类型对应的共享整数类型。
- AtomicI8、AtomicI16、AtomicI32、AtomicI64 及其无符号变体（如 AtomicU8）是共享整数类型，对应于单线程中的类型 i8、i16 等。
- AtomicBool 是一个共享的 bool 值。
- `AtomicPtr<T>` 是不安全指针类型 *mut T 的共享值。

```rust
use std::sync::atomic::{AtomicIsize, Ordering};
let atom = AtomicIsize::new(0);
atom.fetch_add(1, Ordering::SeqCst);
```
参数 Ordering::SeqCst 是指内存排序。
只要拿不准，就尽情使用 Ordering::SeqCst 吧


原子化的一个简单用途是中途取消。假设有一个线程正在执行一些长时间运行的计算（如渲染视频），我们希望能异步取消它。
问题在于如何与希望关闭的线程进行通信。可以通过共享的 AtomicBool 来做到这一点

```rust
use std::sync::Arc;
use std::sync::atomic::AtomicBool;
let cancel_flag = Arc::new(AtomicBool::new(false));
let worker_cancel_flag = cancel_flag.clone();

// ///////////////

use std::thread;
use std::sync::atomic::Ordering;
let worker_handle = thread::spawn(move || {
  for pixel in animation.pixels_mut() {
    render(pixel); // 光线跟踪，需要花几微秒时间
    if worker_cancel_flag.load(Ordering::SeqCst) {
      return None;
    }
  }
  Some(animation)
});


// ///////////////


// 取消渲染
cancel_flag.store(true, Ordering::SeqCst);
// 放弃结果，该结果有可能是`None`
worker_handle.join().unwrap();
```

当然，还有其他实现方法。此处的 AtomicBool 可以替换为 `Mutex<bool>` 或通道。主要区别在于==原子化的开销是最低==的。
原子化操作从==不使用系统调用==。
加载或存储通常会编译为==单个 CPU 指令==。

原子化是==内部可变性==的一种形式（就像 Mutex 或 RwLock），因此它们的方法==会通过共享（非 mut）引用获取 self==。
这使得它们作为简单的全局变量时非常有用。


### 19.3.11 全局变量 。const fn

一个全局变量，即一个每当发出数据包时都会递增的计数器
```rust
/// 服务器已成功处理的数据包的数量
static PACKETS_SERVED: usize = 0;
```

这可以正常编译，但有一个问题：PACKETS_SERVED 是不可变的，所以我们永远都不能改变它。

Rust 会尽其所能阻止全局可变状态

用 ==const== 声明的常量当然是==不可变==的。
==默认==情况下，==静态变量==也是==不可变==的，因此无法获得一个mut 引用。
static 固然可以声明为 mut，但访问它是不安全的
。。rust 所有的类型 都是默认不可变的吧。

支持递增 PACKETS_SERVED 并保持其线程安全的最简单方式是让它变成原子化整数
```rust
use std::sync::atomic::AtomicUsize;
static PACKETS_SERVED: AtomicUsize = AtomicUsize::new(0);
```

```rust
use std::sync::atomic::Ordering;
PACKETS_SERVED.fetch_add(1, Ordering::SeqCst);
```

原子化全局变量仅限于简单的整数和布尔值。
不过，要创建任何其他类型的全局变量，就要解决以下两个问题。
- 变量必须以某种方式成为线程安全的，否则它就不能是全局变量。为了安全起见，静态变量必须同时是 Sync 和非 mut 的
- 静态初始化程序只能调用被专门标记为 const 的函数，编译器可以在编译期间对其进行求值。换句话说，它们的输出是确定性的，这些输出只会取决于它们的参数，而不取决于任何其他状态或I/O。 类似于 C++ 的 constexpr。

Atomic 类型（AtomicUsize、AtomicBool 等）的构造函数都是 const 函数


还可以直接在函数的签名前加上 const 来定义自己的 const 函数
const 函数不能以类型而只能以生命周期作为泛型参数，并且不能分配内存或对裸指针进行操作，即使在 unsafe 的块中也是如此

可以创建便捷函数来更轻松地定义 static 和 const 并减少代码重复

```rust
const fn mono_to_rgba(level: u8) -> Color {
  Color {
    red: level,
    green: level,
    blue: level,
    alpha: 0xFF
  }
}
const WHITE: Color = mono_to_rgba(255);
const BLACK: Color = mono_to_rgba(000);
```

如果方法不是 const的， 又需要在全局静态变量中 声明，那么使用 lazy_static crate
通过 lazy_static! 宏定义的变量允许你使用任何喜欢的表达式进行初始化，该表达式会在第一次解引用变量时运行，并保存该值以供后续操作使用。

```rust
use lazy_static::lazy_static;
use std::sync::Mutex;
lazy_static! {
  static ref HOSTNAME: Mutex<String> = Mutex::new(String::new());
}
```
现在 Mutex::new() 已经是 const fn 了。
同样的技术也适用于其他复杂的数据结构，比如 HashMap 和Deque。
对于根本不可变、只是需要进行非平凡初始化 的静态变量，它也非常方便。


“非平凡初始化”是指初始化一个变量或对象时，需要进行一些复杂的计算或操作，而不是简单的赋值或默认初始化。
在计算机编程中，有些变量或对象的初始化需要进行一些计算，比如读取配置文件、解析命令行参数、连接数据库等，这些初始化操作都可以称为“非平凡初始化”。
与之相反，简单的赋值操作或者默认初始化可以称为“平凡初始化”。


使用 lazy_static! 会在每次访问静态数据时产生很小的性能成本。
该实现使用了 ==std::sync::Once==，这是一种专为一次性初始化而设计的底层同步原语。


## 19.4 在 Rust 中编写并发代码的一点儿经验


保持线程近乎处于隔离态可以让 Rust相信你的代码正在做的事是安全的。

Rust 能让你组合多种技术并进行实验。你可以快速迭代


# ch20 异步编程 816

假设你要编写一个聊天服务器。
对于每个网络连接，都会有一些要解析的传入数据包、要组装的传出数据包、要管理的安全参数、要跟踪的聊天组订阅等。
要想同时管理这么多连接，就得进行一定的组织工作

理论上，可以为传入的每个连接启动一个单独的线程：
```rust
use std::{net, thread};

let listener = net::TcpListener::bind(address)?;
for socket_result in listener.incoming() {
  let socket = socket_result?;
  let groups = chat_group_table.clone();
  thread::spawn(|| {
    log_error(serve(socket, groups));
  });
}
```


可以使用 Rust 异步任务 在单个线程或工作线程池中交替执行许多彼此独立的活动。

异步 Rust 代码看上去很像普通的多线程代码，但实际上那些可能导致阻塞的操作（如 I/O 或获取互斥锁）会以略有不同的方式处理。

前面代码的异步版本如下所示

```rust
use async_std::{net, task};

let listener = net::TcpListener::bind(address).await?;
let mut new_connections = listener.incoming();
while let Some(socket_result) = new_connections.next().await {
  let socket = socket_result?;
  let groups = chat_group_table.clone();
  task::spawn(async {
    log_error(serve(socket, groups).await);
  });
}
```

这里用的是 async_std 这个 crate 的网络模块和任务模块，并在可能发生阻塞的调用之后添加了 .await。

但这段代码的整体结构与基于线程的版本无异。


为了展示异步编程的机制，我们会列举一组涵盖所有核心概念的最小语言特性集：
- Future（未来值）、
- 异步函数、
- await 表达式、
- 任务以及 
- block_on 执行器和 
- spawn_local 执行器。

我们会介绍异步块和 spawn 执行器。

Pin 类型


## 20.1 从同步到异步

考虑调用以下（非异步，而是完全传统的）函数时会发生什么

```rust
use std::io::prelude::*;
use std::net;

fn cheapo_request(host: &str, port: u16, path: &str) -> std::io::Result<String>
{
  let mut socket = net::TcpStream::connect((host, port))?;
  let request = format!("GET {} HTTP/1.1\r\nHost: {}\r\n\r\n", path, host);
  socket.write_all(request.as_bytes())?;
  socket.shutdown(net::Shutdown::Write)?;
  let mut response = String::new();
  socket.read_to_string(&mut response)?;
  Ok(response)
}
```

事实上，这个函数在几乎所有时间里都在等待操作系统
当这个函数正在等待系统调用返回时，它的单个线程是阻塞的

为了解决这个问题，就要允许线程在==等待系统调用==完成==期间进行其他工作==


### 20.1.1 Future

Rust 支持异步操作的方法是引入特型 `std::future::Future`

```rust
trait Future {
  type Output;
  // 现在，暂时把`Pin<&mut Self>`当作`&mut Self`
  fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

enum Poll<T> {
  Ready(T),
  Pending,
}
```

Future 代表一个你可以测试其完成情况的操作。
Future 的 ==poll==（轮询）方法从来不会等待操作完成，它总是==立即返回==。
如果==操作已完成==，则 poll 会==返回 Poll::Ready(output)==，其中 output 是它的最终结果。
==否则，它会返回 Pending==。
如果 Future 值得再次轮询，它承诺会通过调用 Context 中提供的==回调函数 waker== 来通知我们。
我们将这种实现方式称为异步编程的“皮纳塔模型”：对于 Future，你唯一能做的就是通过轮询来“敲打”它，直到某个值“掉”出来。


异步版本的 read_to_string 的签名 大致 如下
```rust
fn read_to_string(&mut self, buf: &mut String) -> impl Future<Output = Result<usize>>;
```

异步版本会返回携带 `Result<usize>` 的 Future。
你需要轮询这个 Future，直到从中获得 Ready(result)。

调用这个版本的 read_to_string 并没有实际读取任何内容，它唯一的职责是构建并返回一个 Future，该 Future 会在轮询时完成其真正的工作
这个 Future 必须包含执行调用请求所需的全部信息

事实上，由于 Future 包含self 和 buf 的引用，因此 read_to_string 的正确签名必然是如下形式
```rust
fn read_to_string<'a>(&'a mut self, buf: &'a mut String) -> impl Future<Output = Result<usize>> + 'a
```

这增加了生命周期以表明返回的 Future 的生存期只能与 self 和 buf 借用的值一样长。

==async-std crate== 提供了所有 std 中 I/O 设施的异步版本，包括带有 read_to_string 方法的异步 Read 特型。
。。这个 async-std  2年前就不更新了，估计不怎么用了。  也有可能直接进 std了。。没有具体的说明


Future 特型的一个规则是，一旦 Future 返回了 Poll::Ready，它就会假定自己永远不会再被轮询（poll）。
当某些 Future 被过度轮询时，它们只会永远返回 Poll::Pending，而其他 Future则可能会 panic 或被挂起。

完全没必要一听到轮询就觉得效率低下。
Rust 的异步架构是经过精心设计的，只要你正确实现了基本的 I/O 函数（如 read_to_string），就只会在值得尝试时才轮询 Future

但使用 Future 似乎很具挑战性：当轮询时，如果得到了Poll::Pending，应该做些什么呢？你必须四处寻找这个线程暂时可以做的其他工作，还不能忘记稍后回到这个 Future 并再次轮询它。

好消息是，你大可不必这样做。


### 20.1.2 异步函数与 await 表达式

```rust
use async_std::io::prelude::*;
use async_std::net;

async fn cheapo_request(host: &str, port: u16, path: &str) -> std::io::Result<String>
{
  let mut socket = net::TcpStream::connect((host, port)).await?;
  let request = format!("GET {} HTTP/1.1\r\nHost: {}\r\n\r\n", path, host);
  socket.write_all(request.as_bytes()).await?;
  socket.shutdown(net::Shutdown::Write)?;
  let mut response = String::new();
  socket.read_to_string(&mut response).await?;
  Ok(response)
}
```

除了以下几点，这段程序跟我们的原始版本几乎是每个字母都一样
- 函数以 `async fn` 而不是 fn 开头。
- 使用 async_std crate 的异步版本的 TcpStream::connect、write_all 和 read_to_string。这些都会返回其结果的 Future。
- 每次返回 Future 的调用之后，代码都会 .await。它是语言中内置的特殊语法，用于等待 Future 就绪。

当你调用异步函数时，它会在==函数体开始执行之前立即返回==

当你调用异步函数时，它会在函数体开始执行之前立即返回。显然，调用的最终返回值还没有计算出来，你得到的只是承载它最终值的 Future。

你不需要调整异步函数的返回类型，Rust 会自动把 ==async== fn f(...) -> T 函数的==返回值视为承载 T 的 Future==，而非直接的 T 值。

Future 的特化类型是由编译器根据函数的主体和参数自动生成的。
这种类型没有名字，你只知道它实现了 `Future<Output=R>`，其中 R 是异步函数的返回类型


当你==首次轮询== cheapo_request 返回的 Future 时，会从函数体的==顶部开始==执行，一直运行到 TcpStream::connect 返回的Future 的==第一个 await==。
==await 表达式会轮询 connect 返回的Future==，如果它==尚未就绪==，则向调用者返回 Poll::Pending：程序不能从这个 await 继续向前运行了，直到对这个 Future 的某次轮询返回了 Poll::Ready。
因此，表达式TcpStream::connect(...).await 大致等价于如下内容

```rust
{
  // 注意：这是伪代码，不是有效的Rust
  let connect_future = TcpStream::connect(...);
  'retry_point:
  match connect_future.poll(cx) {
    Poll::Ready(value) => value,
    Poll::Pending => {
      // 安排对`cheapo_request`返回的Future进行
      // 下一次`poll`，以便在'retry_point处恢复执行
      ...
      return Poll::Pending;
    }
  }
}
```

await 表达式会获取 Future 的所有权，然后轮询它。
如果已就绪，那么 Future 的最终值就是 await 表达式的值，然后继续执行。
否则，此 Future 返回 Poll::Pending。

但==至关重要==的是，下一次对 cheapo_request 返回的 Future 进行轮询时==不会再从函数的顶部==开始，而是会在即将轮询connect_future 的==中途时间点恢复==执行函数。
直到 Future 就绪之前，我们都不会继续处理异步函数的其余部分。


在函数中间暂停执行稍后再恢复，这种能力是异步函数所独有的。
当一个普通函数返回时，它的栈帧就永远消失了。
由于 await 表达式依赖于这种恢复能力，因此只能在异步函数中使用它们。


### 20.1.3 从同步代码调用异步函数：block_on

现在调用者的工作是以某种方式进行轮询。
但最终还是得有人实际等待一个值。

可以使用 async_std 的 async_std::task::block_on 函数从普通的同步函数（如 main）调用 cheapo_request，这会接受一个 Future 并轮询，直到它生成一个值

```rust
fn main() -> std::io::Result<()> {
  use async_std::task;
  let response = task::block_on(cheapo_request("example.com", 80, "/"))?;
  println!("{}", response);
  Ok(())
}
```

由于 block_on 是一个会生成异步函数最终值的同步函数，因此可以将其视为==从异步世界到同步世界==的适配器。

block_on 的阻塞式特征意味着我们不应该在异步函数中使用它，因为在值被准备好之前它会一直阻塞整个线程。异步函数中请改用 await

async_std::task::block_on 的价值在于，它知道如何进入休眠直到 Future 真正值得再次轮询时再启动轮询，而不是浪费处理器时间和电池寿命进行数十亿次无结果的 poll 调用。


### 20.1.4 启动异步任务

本章的目标是让线程在等待的同时做其他工作

async_std::task::spawn_local
该函数会接受一个 Future 并将其添加到任务池中，只要正阻塞着 block_on的 Future 还未就绪，就会尝试轮询。
因此，如果你将一堆 Future传给 spawn_local，然后将 block_on 应用于最终结果的Future，那么 block_on 就会在可以向前推进时轮询每个启动（spawn）后的 Future，并行执行整个任务池，直到你想要的结果就绪。

要想在 async-std 中使用 spawn_local，就必须启用该 crate 的 unstable 特性。
`async-std = { version = "1", features = ["unstable"] }`
。。现在还是，https://docs.rs/async-std/1.12.0/async_std/task/fn.spawn_local.html   写了要 unstable only

。。现在估计不怎么用 async-std 了，应该是 tokio ，futures 了
。。 https://github.com/rust-lang/futures-rs    futures 是官方的。
。github 上 tokio 的 issue 多很多。   futures 的第一页 可以看到 2023的，  async-std 的第一页 可以看到 2022的，
。。那就大约了解下吧。 不过 不同的语言/库，应该都差不多。

spawn_local 函数是标准库的 std::thread::spawn 函数的异步模拟，用于启动线程。


异步任务与线程的一个重要区别是：从一个异步任务到另一个异步任务的切换只会出现在 await 表达式处，且只有当等待的 Future 返回了 Poll::Pending 时才会发生。


### 20.1.5 异步块

除了异步函数，Rust 还支持异步块。
普通的块语句会返回其最后一个表达式的值，而异步块会返回其最后一个表达式值的 Future。
可以在异步块中使用 await 表达式。

```rust
let serve_one = async {
  use async_std::net;
  // 监听连接并接受其中一个
  let listener = net::TcpListener::bind("localhost:8087").await?;
  let (mut socket, _addr) = listener.accept().await?;
  // 在`socket`上与客户端对话
  ...
};
```
上述代码会将 serve_one 初始化为一个 Future（当被轮询时），以侦听并处理单个 TCP 连接。



### 20.1.6 从异步块构建异步函数

异步块为我们提供了另一种实现与异步函数相同效果的方式，并且这种方式更加灵活。

```rust
use std::io;
use std::future::Future;

fn cheapo_request<'a>(host: &'a str, port: u16, path: &'a str) -> impl Future<Output = io::Result<String>> + 'a {
  async move {
    ……函数体……
  }
}
```

当你调用这个版本的函数时，它会立即返回异步块返回值的 Future。

由于没有使用 async fn 语法，因此需要在返回类型中写上 impl Future。但就调用者而言，这两个定义是具有相同函数签名的可互换实现。

如果想在调用函数时立即进行一些计算，然后再创建其结果的Future，那么第二种方法会很有用。

```rust
fn cheapo_request(host: &str, port: u16, path: &str) -> impl Future<Output = io::Result<String>> + 'static
{
  let host = host.to_string();
  let path = path.to_string();
  async move {
    ……使用&*host、port和path……
  }
}
```
这个版本允许异步块将 host 和 path 捕获为拥有型 String 值，而不是 &str 引用



### 20.1.7 在线程池中启动异步任务

我们展示的这些示例把几乎所有时间都花在了等待 I/O 上，但某些工作负载主要是 CPU 任务和阻塞的混合体

当计算量繁重到无法仅靠单个 CPU 满足时，可以使用 async_std::task::spawn 在工作线程池中启动 Future，线程池专门用于轮询那些已准备好向前推进的 Future

```rust
use async_std::task;

let mut handles = vec![];
for (host, port, path) in requests {
  handles.push(task::spawn(async move {
    cheapo_request(&host, port, &path).await
  }));
}
...
```

spawn 具有 spawn_local 所没有的一项限制。由于 Future 会被发送到另一个线程运行，因此它必须实现标记特型 Send


20.1.9 长时间运行的计算：yield_now 与 spawn_blocking

tokio crate（本章稍后会用到）自己定义了一组类似的执行器函数。

### 20.1.11 一个真正的异步 HTTP 客户端


## 20.2 异步客户端与服务器

本节的示例是聊天服务器和客户端。真正的聊天系统是很复杂的，涉及从安全、重新连接到隐私和内部审核的各种问题


我们希望能好好处理背压。
也就是说，即使一个客户端的网络连接速度较慢或完全断开连接，也绝不能影响其他客户端按照自己的节奏交换消息。
由于“龟速”客户端不应该让服务器花费无限的内存来保存其不断增长的积压消息，因此我们的服务器应该丢弃那些发给掉队客户端的消息，但也有义务提醒他们其信息流不完整。

```rust
[package]
name = "async-chat"
version = "0.1.0"
authors = ["You <you@example.com>"]
edition = "2021"

[dependencies]
async-std = { version = "1.7", features = ["unstable"] }
tokio = { version = "1.0", features = ["sync"] }
serde = { version = "1.0", features = ["derive", "rc"] }
serde_json = "1.0"
```

tokio crate 是类似于 async-std crate 的另一个异步基础构件集合
tokio 是一个大型 crate，但我们只需要其中的一个组件，因此Cargo.toml 依赖行中的 features = ["sync"] 字段将tokio 缩减为了我们需要的部分


。。跳

。。得直接写代码。


## 20.3 原始 Future 与执行器：Future 什么时候值得再次轮询

当像 block_on 这样的执行器轮询 Future 时，必须传入一个称为唤醒器（waker）的回调。
如果 Future 还没有就绪，那么 Future特型的规则就会要求它必须暂时返回 Poll::Pending，并且如果Future 值得再次轮询，就会安排在那时调用唤醒器。

如果 Future 的值就绪了，就返回它。否则，将Context 中唤醒器的克隆体存储在某处，并返回Poll::Pending。

当 Future 值得再次轮询时，它一定会通过==调用其唤醒器的 wake 方法==通知最后一个轮询它的执行器：

执行器和 Future 会轮流轮询和唤醒：执行器会轮询Future 并进入休眠状态，然后 ==Future 会调用唤醒器==，这样，执行器就会醒来并再次轮询 Future。

异步函数和异步块的 Future 不会处理唤醒器本身，它们只会将自己获得的上下文传给要等待的子 Future，并将保存和调用唤醒器的义务委托给这些子 Future。


20.3.1 调用唤醒器：spawn_blocking


20.3.2 实现 block_on


## 20.4 固定（Pin）

本节首先会展示为什么异步函数调用和异步块的 Future 不能像普通Rust 值那样随意处理；
然后会展示 Pin 如何用作指针的“许可印章”，我们可以依靠这些“盖章指针”来安全地管理此类 Future


### 20.4.1 Future 生命周期的两个阶段


- 第一阶段从刚创建 Future 时开始。因为函数体还没有开始执行，所以它的任何部分都不可能被借用。在这一点上，移动它和移动其他 Rust 值一样安全。
- 第二阶段在第一次轮询 Future 时开始。一旦函数的主体开始执行，它就可以借用对存储在 Future 中的变量的引用，然后等待，保留对 Future 持有的变量的借用。从第一次轮询开始，就必须假设 Future 不能被安全地移动了。


### 20.4.2 固定指针

Pin 类型是指向 Future 的指针的包装器，它限制了指针的用法，以确保 Future 一旦被轮询就不能移动。

这些限制对于不介意被移动的 Future 是可以取消的，但对于需要安全地轮询异步函数和异步块的 Future 必不可少。

这里的指针指的是任何实现了 Deref 或 DerefMut 的类型。
包裹在指针上的 Pin 称为固定指针。
`Pin<&mut T> 和 Pin<Box<T>>` 是典型的固定指针。

```rust
pub struct Pin<P> {
  pointer: P,
}
```

pointer 字段不是 pub 的。这意味着构造或使用 Pin 的唯一方法是借助该类型提供的经过精心设计的方法。

给定一个异步函数或异步块返回的 Future，只有以下几种方法可以获得指向它的固定指针。
- pin! 宏来自 futures-lite crate，它会用新的 `Pin<&mut T>` 类型的变量遮蔽 T 类型的变量。
- 标准库的 Box::pin 构造函数能获取任意类型 T 值的所有权，将其移动到堆中，并返回 `Pin<Box<T>>`。
- `Pin<Box<T>>` 可以实现 `From<Box<T>>`，因此Pin::from(boxed) 会取得 boxed 的所有权，并返回指向堆上同一个 T 的固定过的 Box。

。。std 有个 pin! 宏。https://doc.rust-lang.org/std/pin/macro.pin.html

获得指向这些 Future 的固定指针的每一种方法都需要放弃对Future 的所有权，并且无法再取回。

一旦固定了 Future，如果想轮询它，那么所有 `Pin<pointer to T>` 类型都会有一个 as_mut 方法，该方法会解引用指针并返回poll 所需的 `Pin<&mut T>`。


### 20.4.3 Unpin 特型

并不是所有的 Future 都需要这样小心翼翼地处理。
对于普通类型（如前面提到的 SpawnBlocking 类型）的 Future 的任何手写实现，这些对构造和使用固定指针方面的限制都是不必要的。

这种耐用类型实现了 Unpin 标记特型：
`trait Unpin { }`

Rust 中的几乎所有类型都使用编译器中的特殊支持自动实现了 Unpin。

异步函数和异步块返回的 Future 是这条规则的例外情况。


## 20.5 什么时候要用异步代码

==异步代码比多线程代码更难写。==
你必须使用正确的 I/O 和同步原语，手动分解长时间运行的计算或将它们分拆到其他线程上，并管理多线程代码中不会遇到的其他细节，比如固定。那么异步代码到底有哪些优势呢？

下面是你常会听到的两种说法，但它们经不起仔细推敲。
- “异步代码非常适合 I/O。”这不完全正确。
  如果应用程序正在花费时间等待 I/O，那么把它变成异步形式并不会让 I/O 运行得更快。
- “异步代码比多线程代码更容易编写。”
  在 JavaScript、Python等语言中，这很可能是正确的。
  在这些语言中，程序员使用async/await 作为并发的一种形式：有一个执行线程，并且中断只发生在 await 表达式中，因此通常不需要互斥锁来保持数据一致，只是不要在使用它的中途进行 await。
  当只有在你的明确许可下才可能发生任务切换时，代码会更容易理解。
  但是这个论点不适用于 Rust，因为在 Rust 中线程用起来几乎不怎么麻烦。
  一旦程序编译完成，就不会出现数据竞争。
  非确定性行为仅限于同步特性，比如互斥锁、通道、原子等，它们都是为应对该行为而设计的。
  因此，异步代码并不能帮你更好地了解其他线程何时会影响你，这在所有安全的 Rust 代码中一目了然


异步代码的真正优势是什么呢?
- 异步任务可以使用更少的内存
  在 Linux 上，一个线程的内存使用量==至少为 20 KiB==，包括用户空间和内核空间的内存使用量。(AAA)
  Future 则小得多：我们的聊天服务器的 Future 只有==几百字节==大小，并且随着 Rust 编译器的改进还能变得更小。
- 异步任务的创建速度更快。
  在 Linux 上，创建一个线程大约需要 15 微秒。
  而启动一个异步任务大约需要 300 纳秒，仅为创建线程所花费==时间的约 1/50==。
- 异步任务之间的上下文切换比操作系统线程之间的上下文切换更快
  在 Linux 上这两个操作所需时间分别为 0.2 微秒和 1.7 微秒。 (BBB)
  然而，这些都是最佳情况下的数值：如果切换是由于 I/O 就绪导致的，则这两个操作的时间都会上升到 1.7 微秒。
  线程之间的切换和不同处理器核心上的任务之间的切换大不相同：跨核心的通信非常慢。


AAA: 这包括内核内存，并按为线程分配的物理分页（而非虚拟分页）中尚未分配的数量计算。macOS 和 Windows 上的数量类似。

BBB: Linux 上下文切换也曾处于 0.2 微秒范围内，直到其内核由于处理器安全漏洞而被迫使用较慢的技术。


# ch21 宏 893

宏是一种扩展语言的方式，它能做到单纯用函数无法做到的一些事。
`assert_eq!(gcd(6, 10), 2);`

assert_eq! 宏能做到一些无法用函数做到的事。
一是当断言失败时，assert_eq! 会生成一条错误消息，其中包含断言的==文件名和行号==。
函数无法获取这些信息，而宏可以，因为它们的工作方式完全不同。

在编译期间，在检查类型并生成任何机器码之前，每个宏调用都会被展开
每个宏调用都会被替换成一些 Rust 代码。前面的宏调用展开后大致如下所示
```rust
match (&gcd(6, 10), &2) {
  (left_val, right_val) => {
  if !(*left_val == *right_val) {
    panic!("assertion failed: `(left == right)`, \
    (left: `{:?}`, right: `{:?}`)", left_val, right_val);
  }
  }
}
```
panic! 也是一个宏，它本身可以展开为更多的 Rust 代码

这些代码使用了另外两个宏，即 ==file!() 和 line!()==

Rust 宏采用了完全不同的设计，类似于 Scheme 的 syntax-rules。


## 21.1 宏基础

assert_eq! 宏的部分源代码。

![61c7876d4b7aae9755facd3a97930a5b.png](../_resources/61c7876d4b7aae9755facd3a97930a5b.png)

macro_rules! 是在 Rust 中定义宏的主要方式。
请注意，这个宏定义中的 assert_eq 之后没有 !：
只有调用宏时才要用到 !，定义宏时不用。

有一些宏是内置于编译器中的，比如 file!、line! 和 macro_rules!。

使用 macro_rules! 定义的宏完全借助“模式匹配”方式发挥作用。
宏的主体只是一系列规则

```rust
( pattern1 ) => ( template1 );
( pattern2 ) => ( template2 );
...
```

可以在模式或模板周围随意使用 方括号或花括号 来代替 圆括号 ，这对 Rust 没有影响。
同样，在调用宏时，下面这些都是等效的

```rust
assert_eq!(gcd(6, 10), 2);
assert_eq![gcd(6, 10), 2];
assert_eq!{gcd(6, 10), 2}
```
唯一的区别是 花括号 后面的 分号 通常是可选的。

按照惯例，
在调用 assert_eq! 时使用圆括号，
在调用 vec! 时使用方括号，而
在调用 macro_rules! 时使用花括号。


21.1.1 宏展开的基础

Rust 在编译期间的很早阶段就展开了宏。
编译器会从头到尾阅读你的源代码，定义并展开宏。
你不能在定义宏之前就调用它，因为 Rust 在查看程序的其余部分之前就已经展开了每个宏调用

Rust 会首先将参数与模式进行匹配

宏模式是 Rust 中的一种迷你语言。
它们本质上是用来匹配代码的正则表达式。

`assert_eq!(gcd(6, 10), 2);`

![8460d8cf711658cd5fff705c6b1fc004.png](../_resources/8460d8cf711658cd5fff705c6b1fc004.png)


Rust 会将 $left 和 $right 替换为它在匹配过程中找到的==代码片段==。

在输出模板中包含片段类型（比如写成 $left:expr 而不仅是 $left）是一个常见的错误。
Rust 不会立即检测到这种错误。它会将$left 视为替代品，然后将 :expr 视为模板中的其他内容——要包含在宏输出中的语法标记。
所以宏在被调用之前不会发生错误，然而它将生成实际无法编译的伪输出。

21.1.2 意外后果

简而言之，宏可以做一些令人惊讶的事情。

如果在你编写的宏周围发生了某些奇怪的事，那么很可能就是宏造成的。


### 21.1.3 重复

标准的 vec! 宏有两种形式：

```rust
// 把一个值重复N次
let buffer = vec![0_u8; 1000];

// 由逗号分隔的值列表
let numbers = vec!["udon", "ramen", "soba"];
```
。。在这本书之前，我只会前一种。。。

```rust
macro_rules! vec {
  ($elem:expr ; $n:expr) => {
    ::std::vec::from_elem($elem, $n)
  };
  ( $( $x:expr ),* ) => {
    <[_]>::into_vec(Box::new([ $( $x ),* ]))
  };
  ( $( $x:expr ),+ ,) => {
    vec![ $( $x ),* ]
  };
}
```

- 第一条规则处理像 vec![0u8; 1000] 这样的用法。
- 第二条规则处理 vec!["udon", "ramen", "soba"]。
  `$($x:expr ),*` 模式使用了我们从未见过的一个特性：重复。
  它会匹配 0 个或多个表达式，以逗号分隔。
- 使用 `$( ... ),+ ,` 来匹配带有额外逗号的列表。
  然后在模板中递归地调用 vec!，并剥离额外的逗号。这次会匹配上第二条规则。


|模式|含义|
|--|--|
|`$( ... )*` |匹配0次或多次，没有分隔符|
|`$( ... ),*`|匹配0次或多次，以逗号分隔|
|`$( ... );*`|匹配0次或多次，以分号分隔|
|`$( ... )+` |匹配1次或多次，没有分隔符|
|`$( ... ),+`|匹配1次或多次，以逗号分隔|
|`$( ... );+`|匹配1次或多次，以分号分隔|
|`$( ... )?` |匹配0次或1次，没有分隔符|

代码片段 `$x` 不是单个表达式，而是一个表达式列表。
这条规则的模板也使用了重复语法：
`<[_]>::into_vec(Box::new([ $( $x ),* ]))`

第一个代码片段 `<[_]>` 是用于编写“某物的切片”类型的一种不寻常的方式，它会期待 Rust 推断出元素类型。
那些名称为普通标识符的类型可以不经任何修改直接用在表达式中，但是像 `fn()、&str 或[_]` 这样的特殊类型必须用尖括号括起来



## 21.2 内置宏

file!()（文件名）、line!()（行号）和 column!()（列号）

file!() 会展开为字符串字面量，即当前文件名。
line!() 和 column!() 会展开为 u32 字面量，以给出当前行号和列号（从 1 开始计数）。

如果一个宏调用了另一个宏，后者又调用了别的宏，并且最后一个宏调用了 file!()、line!() 或 column!()，而它们都在不同的文件中，则最终展开之后的结果指示的是第一个宏所在的位置。


stringify!(...tokens...)（代码字符串）
该宏会展开为包含给定语法标记的字符串字面量。
它的参数中的宏调用不会展开，比如 `stringify!(line!())` 会展开为字符串 "line!()"。

concat!(str0, str1, ...)（串联）
串联它的各个参数展开为单个字符串字面量。

cfg!(...)（配置）
该宏会展开为布尔常量，如果当前正构建的配置与圆括号中的条件匹配则为 true。
如果在启用了调试断言的情况下进行编译，则 cfg!(debug_assertions) 为 true。

env!("VAR_NAME")（环境变量）
展开为字符串，即在编译期指定的环境变量的值。如果该变量不存在，则为编译错误。

Cargo 会在编译 crate 时设置几个有实质性内容的环境变量，这样此宏才有价值。例如，要获取 crate 的当前版本字符串，可以这样写：
`let version = env!("CARGO_PKG_VERSION");`


option_env!("VAR_NAME")（可选环境变量）
和 env! 基本一样，不过该宏会返回 `Option<&'static str>`，如果没有设置指定的变量，则返回 None。


以下 3 个内置宏支持从另一个文件中引入代码或数据。
include!("file.rs")（包含代码文件）
该宏会展开为指定文件的内容，这个文件必须是有效的 Rust 代码——表达式或语法项的序列。

include_str!("file.txt")（包含字符串）
该宏会展开为包含指定文件中文本的 &'static str。可以像这样使用它
`const COMPOSITOR_SHADER: &str = include_str!("../resources/compositor.glsl");`
如果文件不存在或不是有效的 UTF-8 格式，你将收到编译错误。


include_bytes!("file.dat")（包含一些字节）
和上一个宏基本相同，不过该宏会把文件视为二进制数据而非UTF-8 文本。结果是 `&'static [u8]`。


Rust 还提供了几个我们未曾涉及的便捷宏。
todo!()（待做）和 unimplemented!()（未实现）

这两个宏的作用相当于 panic!()，但传达了不同的意图。
unimplemented!() 适用于 if 子句、match 分支和其他尚未处理的情况。它总会 panic。
todo!() 大致相同，但传达了这样的想法，即这段代码还没有编写，一些 IDE 会对其进行标记以提请关注。


matches!(value, pattern)（匹配）
将值与模式进行比较，如果匹配就返回 true，否则返回 false。类似于如下写法：
```rust
match value {
  pattern => true,
  _ => false
}
```


## 21.3 调试宏

以下是 3 个有助于解决宏问题的工具。

第一，也是最简单的，可以让 rustc 展示代码在展开所有宏后的样子。
使用 cargo build --verbose 查看 Cargo 如何调用rustc。
复制 rustc 命令行并添加 -Z unstable-options --pretty expanded 选项。
完全展开的代码将转储到终端。很遗憾，只有当代码没有语法错误时才能这样做。

第二，Rust 提供了一个 log_syntax!() 宏，它只会在编译期将自己的参数打印到终端。
你可以将其用于 println! 式调试。此宏需要添加 `#![feature(log_syntax)]` 特性标志。


第三，可以要求 Rust 编译器把所有宏调用记录到终端。
在代码中的某个地方插入 trace_macros!(true);。
从那时起，每当 Rust 展开宏时，它都会打印宏的名称和各个参数




## 21.4 构建 json! 宏

macro_rules! 支持的片段类型
expr, stmt, ty, path, pat, item, block, meta, literal, lifetime, vis, ident, tt


## 21.5 在匹配过程中避免语法错误


## 21.6 超越 macro_rules!

Rust 1.15 引入了一种称为过程宏的独立机制。
过程宏不仅支持扩展#[derive] 属性以处理自定义派生（参见图 21-4），还支持创建自定义属性和像前面讨论过的 macro_rules! 这样的宏。


也许在阅读了所有这些内容后，你开始有点儿讨厌宏了。
那么，还有什么其他选择吗？还有一种方法是==使用构建脚本生成 Rust 代码==。
Cargo 文档展示了如何一步一步地做到这一点。
它涉及编写一个生成所需 Rust 代码的程序，然后向 Cargo.toml 中添加一行，以便在构建过程中运行该程序并使用 include! 将生成的代码放入你的 crate 中







# ch22 不安全代码 929

使用不安全特性的所有要点。
- Rust 的 unsafe 块在普通的、安全的 Rust 代码和使用了不安全特性的代码之间建立了边界。
- 可以将函数标记为 unsafe，提醒调用者这里存在必须遵守的额外契约，以避免未定义行为。
- 裸指针及其方法允许不受限制地访问内存，进而构建 Rust 的类型系统原本会禁止的数据结构。Rust 的引用是安全但受限的，而任何 C 或 C++ 程序员都知道，裸指针是一个强大而锋利的工具。
- 理解未定义行为的定义将帮助你理解为什么它会产生比得到不正确的结果还要严重的后果。
- 不安全特型（unsafe trait）与 unsafe 函数类似，对每个实现而不是每个调用者都强加了必须遵守的契约


## 22.1 不安全因素来自哪里

```shell
$ cat crash.rs
fn main() {
  let mut a: usize = 0;
  let ptr = &mut a as *mut usize;
  unsafe {
    *ptr.offset(3) = 0x7ffff72f484c;
  }
}
$ cargo build
 Compiling unsafe-samples v0.1.0
 Finished debug [unoptimized + debuginfo] target(s) in 0.44s
$ ../../target/debug/crash
crash: Error: .netrc file is readable by others.
crash: Remove password or make file unreadable by others.
Segmentation fault (core dumped)
$
```

这个程序借用了对局部变量 a 的可变引用，将其转换为 `*mut usize` 类型的裸指针，然后使用 offset 方法在内存中又生成了 3 个字的指针。
这恰好是存储 main 的返回地址的地方。这个程序用一个常量覆盖了返回地址，这样从 main 返回的行为就会令人非常惊讶。


## 22.2 不安全块

```rust
unsafe {
  String::from_utf8_unchecked(ascii)
}
```

与普通的 Rust 块一样，unsafe 块的值就是其最终表达式的值，如果没有则为 ()。

unsafe 块解锁了 5 个额外的选项。
- 可以调用 unsafe 函数。
- 可以解引用裸指针。
- 可以访问 union 的各个字段
- 可以访问可变的 static 变量
- 可以访问通过 Rust 的外部函数接口声明的函数和变量。


## 22.3 示例：高效的 ASCII 字符串类型



## 22.4 不安全函数

unsafe 函数看起来就像前面加了 unsafe 关键字的普通函数。
unsafe 函数的主体自动被视为 unsafe 块。

只能在 unsafe 块中调用 unsafe 函数。


## 22.5 不安全块还是不安全函数

推荐的方法是先对函数做一些判定
- 如果能正常编译，但仍可能以导致未定义行为的方式滥用函数，则必须将其标记为 unsafe。
- 否则，函数就是安全的。也就是说，对函数的任何类型良好的调用都不会导致未定义行为。这样的函数不应该标记为 unsafe。

不要仅仅因为函数体中使用了不安全特性就把 安全的函数 标记为unsafe。这会让函数更难使用


## 22.6 未定义行为

```rust
let i = 10;
very_trustworthy(&i); // 共享引用
println!("{}", i * 100);  // 编译器可以认为 i 不会改变，所以直接 改成 1000
```

```rust
fn very_trustworthy(shared: &i32) {
  unsafe {
    // 把共享引用转换成可变指针
    // 这是一种未定义行为
    let mutable = shared as *const i32 as *mut i32;
    *mutable = 20;
  }
}
```

下面是 Rust 判断程序是否“遵纪守法”的规则
- 程序不得读取未初始化的内存。
- 程序不得创建无效的原始值。
  - 引用、Box 值或 fn 指针为空（null）。
  - bool 值非 0 且非 1。
  - enum 值具有无效判别值
  - char 值无效，比如存在半代用区的 Unicode 码点。
  - str 值不是格式良好的 UTF-8。
  - 胖指针具有无效虚表或 slice 长度。
  - never 类型的任何值，可以写作 !，只能用于不会返回的函数
- 程序必须遵守第 5 章中解释过的引用规则。任何引用的生命周期都不能超出其引用目标，共享访问是只读访问，可变访问是独占访问。
- 程序不得对空指针、未正确对齐的指针或悬空指针进行解引用。
- 程序不得使用指针访问与此指针关联的分配区之外的内存
- 程序必须没有数据竞争。
- 程序不得对借助外部函数接口进行的跨语言调用进行栈展开
- 程序必须遵守标准库函数的契约





## 22.7 不安全特型
不安全特型是一种特型，用于表示这里存在某种 Rust 无法检查也无法强制保障的契约。

std::marker::Send 和 std::marker::Sync 是不安全特型的典型示例

这些特型没有定义任何方法，因此你可以用喜欢的任意类型来轻松实现它们。
但它们确实有契约：
Send 要求实现者能安全地转移给另一个线程，而 
Sync 要求实现者能安全地通过共享引用在线程之间共享。





## 22.8 裸指针

你可以使用裸指针来创建 Rust 的受检查指针类型不能创建的各种结构，比如==双向链表或任意对象图==。

裸指针有以下两种类型。
- *mut T 是指向 T 的允许修改其引用目标的裸指针。
- *const T 是指向 T 的只允许读取其引用目标的裸指针。

（没有单纯的 *T 类型，必须始终指定 const 或 mut。）

可以把引用转换成裸指针，并使用 * 运算符对其解引用：
```rust
let mut x = 10;
let ptr_x = &mut x as *mut i32;
let y = Box::new(20);
let ptr_y = &*y as *const i32;
unsafe {
  *ptr_x += *ptr_y;
}
assert_eq!(x, 30);
```

尽管 Rust 会在各种情况下隐式解引用安全指针类型，但对裸指针解引用必须是显式的。


22.8.1 安全地解引用裸指针
以下是安全使用裸指针的一些常识性指南。
- 解引用空指针或悬空指针是未定义行为，引用未初始化的内存或超出作用域的值也一样。
- 解引用未针对其引用目标的类型正确对齐的指针是未定义行为。
- 只有在遵守了第 5 章解释过的引用安全规则（任何引用的生命周期都不能超出其引用目标，共享访问是只读访问，可变访问是独占访问）的前提下，才能从解引用的裸指针中借用值。
- 仅当引用目标是所属类型的格式良好的值时，才能使用裸指针的引用目标。
- 如果想在特定的裸指针上使用 offset 方法和 wrapping_offset 方法，那么该裸指针只能指向原初（original）指针所引用的变量内部的字节或分配在堆上的内存块内部的字节，或者指向上述两个区域之外的第一字节。
- 如果要给裸指针的引用目标赋值，则不得违反引用目标所属的任何类型的不变条件。


22.8.2 示例：RefWithFlag

22.8.3 可空指针
Rust 中的空裸指针是一个零地址，与 C 和 C++ 中一样。
对于任意类型 T，`std::ptr::null<T>` 函数会返回一个 *const T 空指针，
而 `std::ptr::null_mut<T>` 会返回一个 *mut T 空指针。

检查裸指针是否为空有几种方法。
最简单的是 is_null 方法，但as_ref 方法可能更方便。
`as_ref` 方法会接受 `*const T` 指针并返回 `Option<&'a T>`，以便将一个空指针变成 None。同样，
`as_mut` 方法会将 `*mut T` 指针转换为 `Option<&'a mut T>` 值。


22.8.4 类型大小与对齐方式

调用 `std::mem::size_of::<T>()` 会返回类型 T 值的大小（以字节为单位），
而调用 `std::mem::align_of::<T>()` 会返回其所需的对齐方式。例如：

```rust
assert_eq!(std::mem::size_of::<i64>(), 8);
assert_eq!(std::mem::align_of::<(i32, i32)>(), 4);
```

22.8.5 指针运算


22.8.6 移动入和移动出内存


标准库还提供了将值数组从一个内存块移动到另一个内存块的函数。
- std::ptr::copy(src, dst, count)（复制）
- ptr.copy_to(dst, count)（复制到）
- std::ptr::copy_nonoverlapping(src, dst, count)（复制，无重叠版）
- ptr.copy_to_nonoverlapping(dst, count)（复制到，无重叠版）
- read_unaligned（读取，未对齐版）和 write_unaligned（写入，未对齐版）
- read_volatile（读取，易变版）和 write_volatile（写入，易变版）



22.8.7 示例：GapBuffer


22.8.8 不安全代码中的 panic 安全性

当你决定使用不安全代码时，就得考虑 panic 安全性的问题了。


## 22.9 用联合体重新解释内存

联合体是 Rust 最强大的特性之一，用于操纵这些字节并选择如何解释它们

```rust
union FloatOrInt {
  f: f32,
  i: i32,
}
```

```rust
let mut one = FloatOrInt { i: 1 };
assert_eq!(unsafe { one.i }, 0x00_00_00_01);
one.f = 1.0;
assert_eq!(unsafe { one.i }, 0x3F_80_00_00);
```

联合体的大小会由其最大字段决定。


因为不知道该如何丢弃其内容，所以联合体的所有字段都必须是可Copy 的。
但是，如果必须在联合体中存储一个 String，那么也有相应的解决方案，详情请参阅 std::mem::ManuallyDrop 的标准库文档

## 22.10 匹配联合体

在 Rust 联合体上进行匹配和在结构体上匹配类似，但每个模式必须指定一个字段

```rust
unsafe {
  match u {
    SmallOrLarge { s: true } => { println!("boolean true"); }
    SmallOrLarge { l: 2 } => { println!("integer 2"); }
    _ => { println!("something else"); }
  }
}
```

与联合体变体匹配但 不指定值 的 match 分支永远都会成功。如果 u的最后一个写入字段是 u.i，则以下代码将导致未定义行为：
```rust
// 未定义行为！
unsafe {
  match u {
    FloatOrInt { f } => { println!("float {}", f) },
    // 警告：无法抵达的模式
    FloatOrInt { i } => { println!("int {}", i) }
  }
}
```

## 22.11 借用联合体

借用联合体的一个字段就是借用整个联合体

这意味着，按照正常的借用规则，将一个字段作为可变借用会排斥对该字段或其他字段的任何其他借用，而将一个字段作为不可变借用则意味着对任何字段都不能再进行可变借用。


# ch23 外部函数 979




## 23.1 寻找共同的数据表示


int 通常为 32 位长，但可以更长，也可以短至 16 位；C char 可以是有符号的也可以是无符号的。
为了应对这种可变性，Rust 的 std::os::raw 模块定义了一组保证与某些 C 类型具有相同表示形式的 Rust 类型

std::os::raw
- c_short
- c_int
- c_long
- c_longlong
- c_ushort
- c_uint
- c_ulong
- c_longlong
- c_char
- c_schar
- c_uchar
- c_float
- c_double
- `*mut c_void、*const c_void`



要定义与 C 结构体兼容的 Rust 结构体类型，可以使用 `#[repr(C)]` 属性。
将 `#[repr(C)]` 放在结构体定义之上会要求Rust 结构体中各个字段的内存布局和 C 中类似结构体的内存布局一样。

```rust
use std::os::raw::{c_char, c_int};
#[repr(C)]
pub struct git_error {
  pub message: *const c_char,
  pub klass: c_int
}
```
`#[repr(C)]` 属性只会影响结构体本身的布局，而不会影响其各个字段的表示法，因此要匹配 C 结构体，每个字段也必须使用类 C 类型



```rust
#[repr(C)]
#[allow(non_camel_case_types)]
enum git_error_code {
  GIT_OK = 0,
  GIT_ERROR = -1,
  GIT_ENOTFOUND = -3,
  GIT_EEXISTS = -4,
  ...
}
```

通过将 #[repr(C)] 应用于枚举、结构体和联合体类型，并使用match 语句根据标记在更大的结构体中选择一个联合体字段，Rust代码可以与这个结构体进行互操作

```rust
#[repr(C)]
enum Tag {
  Float = 0,
  Int = 1
}
#[repr(C)]
union FloatOrInt {
  f: f32,
  i: i32,
}
#[repr(C)]
struct Value {
  tag: Tag,
  union: FloatOrInt
}

fn is_zero(v: Value) -> bool {
  use self::Tag::*;
  unsafe {
    match v {
      Value { tag: Int, union: FloatOrInt { i: 0 } } => true,
      Value { tag: Float, union: FloatOrInt { f: num } } => (num == 0.0),
      _ => false
  }
  }
}
```


## 23.2 声明外部函数与变量

extern 块会声明要在最终 Rust 可执行文件链接的其他库中定义的函数或变量。


告诉 Rust，C 库的 strlen 函数是这样的
```rust
use std::os::raw::c_char;
extern {
  fn strlen(s: *const c_char) -> usize;
}
```

```rust
use std::ffi::CString;
let rust_str = "I'll be back";
let null_terminated = CString::new(rust_str).unwrap();
unsafe {
  assert_eq!(strlen(null_terminated.as_ptr()), 12);
}
```

POSIX 系统有一个名为environ 的 全局变量，该变量会持有进程的环境变量的值。在 C 中，可以将其声明为如下形式：
`extern char **environ;`

```rust
use std::ffi::CStr;
use std::os::raw::c_char;
extern {
  static environ: *mut *mut c_char;
}
```

要打印环境的第一个元素，可以这样写：
```rust
unsafe {
  if !environ.is_null() && !(*environ).is_null() {
    let var = CStr::from_ptr(*environ);
    println!("first environment variable: {}", var.to_string_lossy())
  }
}
```

## 23.3 使用库中的函数

要使用特定库提供的函数，可以在 extern 块的顶部放置一个 `#[link]` 属性，该属性中的 name 是 Rust 应该链接进可执行文件的库。
例如，下面这个程序会调用 libgit2 的初始化和关闭方法，但不会执行任何其他操作

```rust
use std::os::raw::c_int;

#[link(name = "git2")]
extern {
  pub fn git_libgit2_init() -> c_int;
  pub fn git_libgit2_shutdown() -> c_int;
}
fn main() {
  unsafe {
    git_libgit2_init();
    git_libgit2_shutdown();
  }
}
```

`#[link(name ="git2")]` 属性会在 crate 中留下一个便签，大意是当 Rust 创建最终的可执行文件或共享库时，应该链接到 git2 库

#[link] 属性也适用于库 crate。在构建依赖于其他 crate 的程序时，Cargo 会收集整个依赖图中的 link 注解，并将它们全部包含在最终链接中。

可以通过编写构建脚本来告诉 Rust 在何处搜索库，这是 Cargo 在构建时会编译和运行的 Rust 代码。
构建脚本可以做各种各样的事情，比如动态生成代码、编译 C 代码并将其包含在 crate 中，等等。

将名为 build.rs 的文件添加到 Cargo.toml 文件的所属目录中，其中包含如下内容：
```rust
fn main() {
  println!(r"cargo:rustc-link-search=native=/home/jimb/libgit2-
0.25.1/build");
}
```
这是 Linux 上的正确路径，在 Windows 上，可以将文本 native=后面的路径更改为 C:\ Users\JimB\libgit2-0.25.1\build\Debug。

它不知道运行期在哪里可以找到 共享库。
在 Linux 上，必须设置 LD_LIBRARY_PATH 环境变量
在 macOS 上，则要设置 DYLD_LIBRARY_PATH。
在 Windows 上，必须设置 PATH 环境变量

Cargo 约定，提供 C 库访问能力的 crate 应该命名为 LIB-sys，其中 LIB 是 C 库的名称。-sys crate 应该只包含静态链接库以及带有 extern 块和类型定义的 Rust 模块。

更高级别的接口则属于依赖 -sys crate 的那些 crate。


## 23.4 libgit2 的裸接口

要正确使用 libgit2，需要弄清楚以下两个问题。
- 在 Rust 中使用 libgit2 函数需要做什么？
- 如何围绕 libgit2 函数构建安全的 Rust 接口？

。。跳

## 23.5 libgit2 的安全接口

。。跳



## 23.6 结论

Rust 不是一门简单的语言。它的目标是横跨两个截然不同的世界。它是一门现代编程语言，天生安全，具有闭包和迭代器等便捷特性。但Rust 旨在让你以最低的运行期开销控制运行计算机的原始能力。

Rust 设法用安全代码弥合了大部分差距。它的借用检查器和零成本抽象让你尽可能接近裸机，却不必冒着未定义行为的风险。

它的目标始终是使用不安全特性来构建安全 API



2024-05-05 21:22













