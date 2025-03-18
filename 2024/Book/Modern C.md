Jens Gustedt

2023-08-26 13:20

[[toc]]

---

C standards in 1989, 1999, 2011, and 2018, commonly referred to as C89, C99, C11, and C17.
you will need at least a compiler that implements most of C99.

In this book, we will mainly refer to ==C17==, as defined in JTC1/SC22/WG14

C and C++ are different: don’t mix them, and don’t mix them up.


Source code
https://gforge.inria.fr/frs/download.php/latestzip/5297/code-latest.zip
。。403 forbidden。。


level 0-3
0 是非常基础
1 是控制结构，数据类型，操作符，函数。
2 是 C语言的核心：指针，C内存模型，二进制接口
3 是 特殊的topic：性能，重用，原子，线程，泛型。

。。这书就按照上面的 分为 4部分， 每部分 几章。
。。所以前几部分，我估计快速过。


# level 0
## ch 1 getting started

。。第一个代码就是盲区。
。。数组。 那么[2] 是 0.0 ？    [4] [3] 顺序。
。。 %zu .. %g 。。。
。。size_t

```C
/* This may look like nonsense, but really is -*- mode: C -*- */
#include <stdlib.h>
#include <stdio.h>

/* The main thing that this program does. */
int main(void) {
  // Declarations
  double A[5] = {
    [0] = 9.0,
    [1] = 2.9,
    [4] = 3.E+25,
    [3] = .00007,
  };

  // Doing some work
  for (size_t i = 0; i < 5; ++i) {
      printf("element %zu is %g, \tits square is %g\n",
             i,
             A[i],
             A[i]*A[i]);
  }

  return EXIT_SUCCESS;
}
```
### 编译命令
`c99 -Wall -o getting-started getting-started.c -lm`

```shell
> clang -Wall -lm -o getting-started getting-started.c
> gcc -std=c99 -Wall -lm -o getting-started getting-started.c
> icc -std=c99 -Wall -lm -o getting-started getting-started.c
```

。。c99 不是1999 的吗？  就这么猛了？
。。前面确实提过 需要 至少 c99 的编译器， 但是也说了 代码是 C17 的啊。 C99 能编译C17 ？
。。而且 我好像只有 gcc 
。。GCC支持C99, 通过 `--std=c99`

- c99 is the compiler program.
- -Wall tells it to warn us about anything that it finds unusual.
- -o getting-started tells it to store the compiler outputC in a file named getting-started.
- getting-started.c names the source fileC, the file that contains the C code we have written. Note that the .c extension at the end of the filename refers to the C programming language.
- -lm tells it to add some standard mathematical functions if necessary; we will need those later on.

- c99 是编译器程序
- -Wall 让编译器 发出所有警告
- -o getting-started 告诉编译器保存 编译后的输出 到名为 getting-started 的文件中
- -lm 让编译器 增加 用到的标准数学函数。 我们后续会用到。


## ch 2 principal structure of a program

printf is provided by stdio.h
size_t and EXIT_SUCCESS come from stdlib.h

```C
int printf(char const format[static 1], ...);
typedef unsigned long size_t;
#define EXIT_SUCCESS 0
```

```shell
> apropos printf
> man printf
> man 3 printf
```

declarations  definition
声明， 定义 是分开的。

Missing elements in initializers default to 0.


call by value
call by reference

C does not implement pass by reference, 
but it has another mechanism to pass the control of a variable to another function: by taking addresses and transmitting pointers.
。。C没有 传引用， 但是可以传指针。



# level 1

## style (厉害的)

==**We bind type modifiers and qualifiers to the left.**==
。。类型修饰符和限定符。
。。确实 * 是修饰/限定 char 的。  6， 第一次知道这个概念。

`char* name;`
`char const* const path_name;`

**==We do not use continued declarations==**
`unsigned const*const a, b;`
b has type unsigned const
the first const goes to the type, and the second const only goes to the declaration of a.

。。？const 还能作为 类型？ 。。 不是 unsigned 是类型，第一个const 描述 unsigned。

==**We use array notation for pointer parameters.**==
。。对于指针参数，使用数组符号。
We do so wherever these assume that the ==pointer can’t be null==

```C
/* These emphasize that the arguments cannot be null. */
size_t strlen(char const string[static 1]);
int main(int argc, char* argv[argc+1]);
/* Compatible declarations for the same functions. */
size_t strlen(const char *string);
int main(int argc, char **argv);
```

。。char const string[static 1]  。。 static 1 ？？？
。。 argc+1， 可以获得 argc 吗？


。。static 1 表示， 数组的长度至少是1 。

#### my_try
static 1， 传一个空字符串，会报错？  编译期报错、  那如果是 输入的字符呢？
用 static 10.
C++有这个吗？
。。 这个好像没有用啊。  加上 -std=c17 也一样。  代码 01

还有，argc + 1。 这样的话，就意味着 必须先知道 argc， 所以是 从前往后的顺序压栈？
如果 换下顺序呢？
C++呢？
。。换下顺序，就编译报错。

还有下面的 function pointer，是 一个值，一个指针， 还是说 等价的？


==**We use function notation for function pointer parameters**==
we do so whenever we know that a ==function pointer can’t be null==

```C
/* This emphasizes that the handler'' argument cannot be null. */
int atexit(void handler(void));
/* Compatible declaration for the same function.              */
int atexit(void (*handler)(void));
```

。。直接 函数名， 而不进行 指针反引用。
。。第一种 是 复制了一个 函数？ 第二种是 共用一个函数？
。。毕竟第一个是 值， 第二个是 指针。
。。好像不是， 这2个 应该等价的。


**We define variables as close to their first use as possible.**
没有 对变量(特别是指针) 的 初始化，带来了 大量的错误。
所以 在声明时 必须初始化。 所以在 第一次用到的地方 声明+初始化。

**We use prefix notation for code blocks**
。。就是 {不能新起一行，   }新起一行。
。 } else { 
。。 do {
。。 } while (false);



## ch 3 Everything is about control

if, for, do, while, switch

#if/#ifdef/#ifndef/#elif/#else/#endif

type generic expressions denoted with the keyword _Generic


<stdbool.h>
The type bool, specified in stdbool.h
Its values are false and true

Don’t compare to 0, false, or true.

Using the truth value directly makes your code clearer and illustrates one of the basic concepts of the C language

All scalars have a truth value.

scalar types include all the numerical types such as size_t, bool, and int that we already encountered, and pointer types


#### my_try

Scalar types used in this book
|level|name|other|category|where|printf|
|--|--|--|--|--|--|
|0|size_t||Unsigned|stddef.h|%zu, %zx|
|0|double||Floating|内置|%e, %f, %g, %a|
|0|signed|int|Signed|内置|%d|
|0|unsigned||Unsigned|内置|%u %x|
|0|bool|_Bool|Unsigned|stdbool.h|%d as 0 or 1|
|1|ptrdiff_t||Signed|stddef.h|%td|
|1|char const*||String|内置|%s|
|1|char||Chararcter|内置|%c|
|1|void*||Pointer|内置|%p|
|2|unsigned char||Unsigned|内置|%hhu, %02hhx|

。。代码 02
。。带 x 的是 16进制。


```C
for (size_t i = 9; i <= 9; --i) {
    something_else(i);
}
```
size_t 是unsigned 的， 所以不会 死循环。


tgmath.h
“tgmath” stands for type generic mathematical functions.


switch 的 case values must be integer constant expressions.
case labels must not jump beyond a variable definition.



## ch 4 Expressing computations

the type size_t represents values in the range [0, SIZE_MAX].

stdint.h provides SIZE_MAX


### all operator

![a048354ca997f394cd138572cb0af48e.png](../_resources/a048354ca997f394cd138572cb0af48e.png)


![83b6066248c25015c8562542fc757ba1.png](../_resources/83b6066248c25015c8562542fc757ba1.png)


![9163eb45a89faa6097469cf950048862.png](../_resources/9163eb45a89faa6097469cf950048862.png)



Arithmetic on size_t implicitly does the computation %(SIZE_MAX+1).

Never modify more than one object in a statement.


false and true are nothing more than fancy names for 0 and 1. 
So, they can be used in arithmetic or for array indexing

```C
size_t c = (a < b) + (a == b) + (a > b);
size_t d = (a <= b) + (a >= b) - 1;
```

```C
   double largeA[N] = { 0 };
   ...
   /* Fill largeA somehow */

   size_t sign[2] = { 0, 0 };
   for (size_t i = 0; i < N; ++i) {
       sign[(largeA[i] < 1.0)] += 1;        // boolean as index
   }
```

iso646.h 中有 标识符 not_eq  可以代替 != ， 不过很少用， 因为这是用在 那些 缺少字符的 计算机平台上的。



三目运算符，计算实数的 sqrt
```C
#include <tgmath.h>

#ifdef __STDC_NO_COMPLEX__
# error "we need complex arithmetic"
#endif

double complex sqrt_real(double x) {
  return (x < 0) ? CMPLX(0, sqrt(-x)) : CMPLX(sqrt(x), 0);
}
```

复数计算，需要 complex.h， 它已经被 tgmath.h 导入了。



&&, ||, ?:, and , evaluate their first operand first.

A[i, j] is not a two-dimensional index for matrix A, but results in A[j].

Don’t use the , operator.

Most operators don’t sequence their operands.
That order may depend on your compiler, on the particular version of that compiler, on compile-time options, or just on the code that surrounds the expression. Don’t rely on any such particular sequencing: it will bite you.

Function calls don’t sequence their argument expressions.

Functions that are called inside expressions should not have side effects.


。。这里直接让给 uf。。


## ch 5 Basic values and data

- Understanding the abstract state machine
- Working with types and values
- Initializing variables
- Using named constants
- Binary representations of types


All values are numbers or translate to numbers.


The state of the program execution is determined by:
- The executable
- The current point of execution
- The data
- Outside intervention, such as IO from the user


All values have a type that is statically determined.

Possible operations on a value are determined by its type.

A value’s type determines the results of all operations.

Generally, all information we need to determine that model is within reach of any C program: the C library headers provide the necessary information through named values (such as SIZE_MAX), operators, and function calls.

A type’s binary representation is observable.

Type determines optimization opportunities.


**Use size_t for sizes, cardinalities, or ordinal numbers.**

Use unsigned for small quantities that can’t be negative.

Use ptrdiff_t for large differences that bear a sign.

使用 double， double_complex。  (。。即不使用 float， float_complex)


|type|header|context of definition|meaning|
|--|--|--|--|
|size_t|stddef.h||type for sizes and cardinalities (大小和基数)|
|ptrdiff_t|stddef.h||type for size differences|
|uintmax_t|stdint.h||max width unsigned integer, preprocessor|
|intmax_t|stdint.h||max witdh signed integer, preprocessor|
|time_t|time.h|time(0), difftime(t1, t0)|calendar time in second since epoch|
|clock_t|time.h|clock()|processor time|

clock_t 的值代表了 处理器的时钟周期，所以单位 比秒 小很多， 使用 `CLOCKS_PER_SEC` 可以转换为 秒。

#### my_try
看下上面的 uintmax_t 是不是 就是 INT_MAX ？
。。ptrdiff_t是C/C++标准库中定义的一个与机器相关的数据类型。ptrdiff_t类型变量通常用来保存两个指针减法操作的结果。ptrdiff_t定义在stddef.h（cstddef）这个文件内。ptrdiff_t通常被定义为long int类型。

。。。。。uintmax_t  是 type。。。


### 5.3 指定值

123 : 整数常量，最常用
077 : 8进制
0xFFF : 16进制
1.6E-13 : 浮点数，科学计数法
0x1.6aP-13 : 十六进制浮点数，  0XhPe 代表了 h * (2^e) 。 h是16进制， e是10进制。
'a' : char
"hello" : 字符串字面量。


Consecutive string literals are concatenated.
```C
puts("first line\n"
    "another line\n"
    "first and "
    "second part of the third line");
```


The rules for numbers are a little bit more complicated.

Numerical literals are never negative.
- -34 or -1.5E-23, the leading sign is not considered part of the number but is the negation operator

Decimal integer constants are signed.
。。10进制真数常量 是有符号的。

To determine the exact type for integer literals, we always have a ==first fit== rule.
- A decimal integer constant has the first of the three signed types that fits it.
。。就是 先匹配 signed short，然后 signed int， 最后 signed long。
。。不知道有没有 signed long long 。
。。这里是 字面量的 类型， 赋给 变量的时候 还要转换的。
。。这里是 10进制

The same value can have different types.
- 0x7FFF has the value 32767 and thus is type signed
- 0x8000 (value 32768 written in hexadecimal) then is an unsigned, and expression -0x8000 again is unsigned
。。8进制 和 16进制， 应该是 先 unsigned short，然后 signed short， 然后 unsigned int, signed int, signed long, unsigned long ？


Don’t use octal or hexadecimal constants to express negative values.
Use decimal constants to express negative values.

Integer constants can be forced to be unsigned or to be a type with minimal width. This is done by appending ==U, L, or LL== to the literal.

1U : unsigned (int)
1L : signed long
1ULL : unsigned long long


int x = 0xFFFFFFFF.  x 是-1.

0, 0x0, and '\0' are all the same value, a 0 of type signed int. 0 has no decimal integer spelling: 0.0 is a decimal spelling for the value 0 but is seen as a floating-point value with type double.


Different literals can have the same value.
对于整数，这条不可能成立。对于浮点数：the constants 0.2 and 0.2000000000000000111 have the same value.

||float|double|long double|
|字面量|0.2F|0.2|0.2L|
|值|0x1.99999AP-3F|0x1.999999999999AP-3|0xC.CCCCCCCCCCCCCCDP-6L|

#### 0.2L is long double


#### 5.3.1 复数常量(跳过了)



### 5.4 隐式转换

-1 : signed int
-1U : unsigned int, 但是 无符号没有负数，所以 -1U 是 最大的unsigned int

#### my_try
printf 下 -1U  。。 赋值给 int， 然后 printf 下， 赋给 usigned int，printf。
prove that -0x80000000 == 0x80000000.
试下 下下 个 C代码块。 就是 INT_MAX + 0.0F == INT_MAX + 1.0F

。。 代码 02
    // -1, 4294967295
    // -1, 4294967295
    // -1, 4294967295
    printf("%d, %u\n", -1U, -1U);
    int a = -1U;
    printf("%d, %u\n", a, a);
    unsigned b2 = -1U;
    printf("%d, %u\n", b2, b2);
。。
。。 代码 02 。确实， float 是harmful,  double harmless   。。 C++也是这个问题。

    float d2 = INT_MAX + 0.0f;
    // 2147483647, 2147483648.000000, 2147483648.000000, 2147483648.000000
    printf("%d, %f, %f, %f\n", INT_MAX, d2, INT_MAX + 0.0f, INT_MAX + 1.0f);

    double d3 = INT_MAX + 0.0;
    // 2147483647.000000 2147483647.000000 2147483648.000000
    printf("%f %f %f\n", d3, INT_MAX + 0.0, INT_MAX + 1.0);
。。 。。 C++也有这问题。


```C
double          a = 1;             // Harmless; value fits type
signed short    b = -1;            // Harmless; value fits type
signed int      c = 0x80000000;    // Dangerous; value too big for type
signed int      d = -0x80000000;   // Dangerous; value too big for type
signed int      e = -2147483648;   // Harmless; value fits type
unsigned short  g = 0x80000000;    // Loses information; has value 0
```


Avoid narrowing conversions.
。。避免 缩小范围 的转换

Don’t use narrow types in arithmetic.
。。不要使用 狭义/小 类型。


```C
1       + 0.0  // Harmless; double
1       + I    // Harmless; complex float
INT_MAX + 0.0F // May lose precision; float
INT_MAX + I    // May lose precision; complex float
INT_MAX + 0.0  // Usually harmless; double
```
on my machine, INT_MAX + 0.0F is the same as INT_MAX + 1.0F and has the value 2147483648

```C
-1    < 0    // True, harmless, same signedness
-1L   < 0    // True, harmless, same signedness
-1U   < 0U   // False, harmless, same signedness

-1    < 0U   // False, dangerous, mixed signedness
-1U   < 0    // False, dangerous, mixed signedness
-1L   < 0U   // Depends, dangerous, same or mixed signedness
-1LL  < 0UL  // Depends, dangerous, same or mixed signedness
```
On platforms where UINT_MAX ≤ LONG_MAX, 0U is converted to 0L, and thus the first result is true. 
On other platforms with LONG_MAX < UINT_MAX, -1L is converted to -1U (that is, UINT_MAX), and thus the first comparison is false.


Avoid operations with operands of different signedness.

==Use unsigned types whenever you can.==


### 5.5 ==Initializers==

All variables should be initialized.
例外：variable-length arrays (VLA) (6.1.3)


scalar 类型的 initizlier 表达式 外面可以放一个 { } ，也可以不放
```C
double a = 6.7;
double b = 2 * a;
double c = {6.7};
double d = {0};
```

其他类型的 initializer 表达式 外面必须有 { }。

```C
double a[] = { 3.4, };
double b[3] = {2*A[0], 4, 22, };
double c[] = {[0] = 4, [3] = 1,};
```


如果你不知道怎么初始化 类型T 的变量，那么 `T a = {0};` 总是可以的。
{0} 对于所有 object type (除了 VLA) 都是 合法的。



### 5.6 named constants

有特殊含义的 常量 都应该 具名。

#### 只读对象

const 修饰的type 就是 只读的。

但，字符串字面量，是只读的，但是没有办法保证不被修改。








































