Kotlin-37
2021年8月27日
11:00

https://www.kotlincn.net/docs/reference/basic-types.html

在 Kotlin 中，所有东西都是对象，在这个意义上讲我们可以在任何变量上调用成员函数与属性

Kotlin 提供了一组表示数字的内置类型。 对于整数，有四种不同大小的类型，因此值的范围也不同

Byte        8        -128        127
Short        16        -32768        32767
Int        32        -2,147,483,648 (-231)        2,147,483,647 (231 - 1)

Long        64        -9,223,372,036,854,775,808 (-263)        9,223,372,036,854,775,807 (263 - 1)

所有以未超出 Int 最大值的整型值初始化的变量都会推断为 Int 类型。如果初始值超过了其最大值，那么推断为 Long 类型。 如需显式指定 Long 型值，请在该值后追加 L 后缀。

val one = 1 // Int
val threeBillion = 3000000000 // Long
val oneLong = 1L // Long
val oneByte: Byte = 1

对于浮点数，Kotlin 提供了 Float 与 Double 类型
类型        大小（比特数）        有效数字比特数        指数比特数        十进制位数
Float        32        24        8        6-7
Double        64        53        11        15-16

。。L，F，f,,, 没有小写的L。。

Kotlin 中的数字没有隐式拓宽转换。 例如，具有 Double 参数的函数只能对 Double 值调用，而不能对 Float、 Int 或者其他数字值调用。

十进制: 123
Long 类型用大写 L 标记: 123L
十六进制: 0x0F
二进制: 0b00001011

不支持八进制

你可以使用下划线使数字常量更易读
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010

在 Java 平台数字是物理存储为 JVM 的原生类型，除非我们需要一个可空的引用（如 Int?）或泛型。 后者情况下会把数字装箱。

注意数字装箱不一定保留同一性

val a: Int = 100
val boxedA: Int? = a
val anotherBoxedA: Int? = a

val b: Int = 10000
val boxedB: Int? = b
val anotherBoxedB: Int? = b

println(boxedA === anotherBoxedA) // true
println(boxedB === anotherBoxedB) // false

。。。还真是，，，估计 -128 到 127 是  单例的，，类似bigDecimal 里的。
、、、这里是 3个等号。。等于 java的 == ，，，  2个==等于java的equals。。。

。。java.Integer.parseInt 返回的是 int。。
。java.Integer.valueOf 返回Integer。
。。而且 valueOf 先parseInt转成int，然后判断 Integer的内部类IntegerCache 是否包含这个数字。。

由于不同的表示方式，较小类型 并不是 较大类型的子类型。 如果 它们是的话，就会 出现下述问题：

// 假想的代码，实际上并不能编译：
val a: Int? = 1 // 一个装箱的 Int (java.lang.Integer)
val b: Long? = a // 隐式转换产生一个装箱的 Long (java.lang.Long)
print(b == a) // 惊！这将输出“false”鉴于 Long 的 equals() 会检测另一个是否也为 Long

因此较小的类型不能隐式转换为较大的类型。 这意味着在不进行显式转换的情况下我们不能把 Byte 型值赋给一个 Int 变量
val b: Byte = 1 // OK, 字面值是静态检测的
val i: Int = b // 错误

我们可以显式转换来拓宽数字
val i: Int = b.toInt() // OK：显式拓宽
print(i)

每个数字类型支持如下的转换:
toByte(): Byte
toShort(): Short
toInt(): Int
toLong(): Long
toFloat(): Float
toDouble(): Double
toChar(): Char

缺乏隐式类型转换很少会引起注意，因为类型会从上下文推断出来，而算术运算会有重载做适当转换，例如：
val l = 1L + 3 // Long + Int => Long

Kotlin支持数字运算的标准集（+ - * / %），运算被定义为相应的类成员（但编译器会将函数调用优化为相应的指令）

请注意，整数间的除法总是返回整数。会丢弃任何小数部分

如需返回浮点类型，请将其中的一个参数显式转换为浮点类型
val x = 5 / 2.toDouble()

对于位运算，没有特殊字符来表示，而只可用中缀方式调用具名函数
val x = (1 shl 2) and 0x000FF000

这是完整的位运算列表（只用于 Int 与 Long）：
shl(bits) – 有符号左移
shr(bits) – 有符号右移
ushr(bits) – 无符号右移
and(bits) – 位与
or(bits) – 位或
xor(bits) – 位异或
inv() – 位非

。。。用的时候 带不带括号都行，但是要注意 优先级。。估计 除非是 单个变量/值，不然都得带个括号。

本节讨论的浮点数操作如下：
相等性检测：a == b 与 a != b
比较操作符：a < b、 a > b、 a <= b、 a >= b
区间实例以及区间检测：a..b、 x in a..b、 x !in a..b

。。val df23 = 1.3 in 1.2F .. 3.1F
。。range的2个 必须同类型。

当其中的操作数 a 与 b 都是静态已知的 Float 或 Double 或者它们对应的可空类型（声明为该类型，或者推断为该类型，或者智能类型转换的结果是该类型），两数字所形成的操作或者区间遵循 IEEE 754 浮点运算标准。

然而，为了支持泛型场景并提供全序支持，当这些操作数并非静态类型为浮点数（例如是 Any、 Comparable<……>、 类型参数）时，这些操作使用为 Float 与 Double 实现的不符合标准的 equals 与 compareTo，这会出现：

认为 NaN 与其自身相等
认为 NaN 比包括正无穷大（POSITIVE_INFINITY）在内的任何其他元素都大
认为 -0.0 小于 0.0

字符用 Char 类型表示。它们不能直接当作数字

字符字面值用单引号括起来: '1'

我们可以显式把字符转换为 Int 数字：
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // 显式转换为数字
}

当需要可空引用时，像数字、字符会被装箱。装箱操作不会保留同一性

布尔用 Boolean 类型表示，它有两个值：true 与 false。
若需要可空引用布尔会被装箱。

内置的布尔运算有：
|| – 短路逻辑或
&& – 短路逻辑与
! - 逻辑非

数组在 Kotlin 中使用 Array 类来表示，它定义了 get 与 set 函数（按照运算符重载约定这会转变为 []）以及 size 属性，以及一些其他有用的成员函数：

class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ……
}

我们可以使用库函数 arrayOf() 来创建一个数组并传递元素值给它，这样 arrayOf(1, 2, 3) 创建了 array [1, 2, 3]。 或者，库函数 arrayOfNulls() 可以用于创建一个指定大小的、所有元素都为空的数组。

另一个选项是用接受数组大小以及一个函数参数的 Array 构造函数，用作参数的函数能够返回给定索引的每个元素初始值
// 创建一个 Array<String> 初始化为 ["0", "1", "4", "9", "16"]
val asc = Array(5) { i -> (i * i).toString() }
asc.forEach { println(it) }

Kotlin 也有无装箱开销的专门的类来表示原生类型数组: ByteArray、 ShortArray、IntArray 等等。这些类与 Array 并没有继承关系，但是它们有同样的方法属性集。它们也都有相应的工厂方法:

val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]

// 大小为 5、值为 [0, 0, 0, 0, 0] 的整型数组
val arr = IntArray(5)

// 例如：用常量初始化数组中的值
// 大小为 5、值为 [42, 42, 42, 42, 42] 的整型数组
val arr = IntArray(5) { 42 }

// 例如：使用 lambda 表达式初始化数组中的值
// 大小为 5、值为 [0, 1, 2, 3, 4] 的整型数组（值初始化为其索引值）
var arr = IntArray(5) { it * 1 }

Kotlin 为无符号整数引入了以下类型：
kotlin.UByte: 无符号 8 比特整数，范围是 0 到 255
kotlin.UShort: 无符号 16 比特整数，范围是 0 到 65535
kotlin.UInt: 无符号 32 比特整数，范围是 0 到 2^32 - 1
kotlin.ULong: 无符号 64 比特整数，范围是 0 到 2^64 - 1
请注意，将类型从无符号类型更改为对应的有符号类型（反之亦然）是二进制不兼容变更
无符号类型自 Kotlin 1.3 起才可用

与原生类型相同，每个无符号类型都有相应的为该类型特化的表示数组的类型：
kotlin.UByteArray: 无符号字节数组
kotlin.UShortArray: 无符号短整型数组
kotlin.UIntArray: 无符号整型数组
kotlin.ULongArray: 无符号长整型数组

此外，区间与数列也支持 UInt 与 ULong（通过这些类 kotlin.ranges.UIntRange、 kotlin.ranges.UIntProgression、 kotlin.ranges.ULongRange、 kotlin.ranges.ULongProgression）

为使无符号整型更易于使用，Kotlin 提供了用后缀标记整型字面值来表示指定无符号类型（类似于 Float/Long）
后缀 u 与 U 将字面值标记为无符号。确切类型会根据预期类型确定。如果没有提供预期的类型，会根据字面值大小选择 UInt 或者 ULong

val b: UByte = 1u  // UByte，已提供预期类型
val s: UShort = 1u // UShort，已提供预期类型
val l: ULong = 1u  // ULong，已提供预期类型

val a1 = 42u // UInt：未提供预期类型，常量适于 UInt
val a2 = 0xFFFF_FFFF_FFFFu // ULong：未提供预期类型，常量不适于 UInt

后缀 uL 与 UL 显式将字面值标记为无符号长整型。
val a = 1UL // ULong，即使未提供预期类型并且常量适于 UInt

字符串用 String 类型表示。字符串是不可变的。 字符串的元素——字符可以使用索引运算符访问: s[i]。 可以用 for 循环迭代字符串:
for (c in str) {
    println(c)
}

可以用 + 操作符连接字符串。这也适用于连接字符串与其他类型的值， 只要表达式中的第一个元素是字符串：
val s = "abc" + 1
println(s + "def")

原始字符串 使用三个引号（"""）分界符括起来，内部没有转义并且可以包含换行以及任何其他字符:
val text = """
    for (c in "foo")
        print(c)
"""

你可以通过 trimMargin() 函数去除前导空格：
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()

默认 | 用作边界前缀，但你可以选择其他字符并作为参数传入，比如 trimMargin(">")。

字符串字面值可以包含模板表达式 ，即一些小段代码，会求值并把结果合并到字符串中。 模板表达式以美元符（$）开头，由一个简单的名字构成: 或者用花括号括起来的任意表达式

原始字符串与转义字符串内部都支持模板。 如果你需要在原始字符串中表示字面值 $ 字符（它不支持反斜杠转义），你可以用下列语法：
val price = """
${'$'}9.99
"""

有多个包会默认导入到每个 Kotlin 文件中：
kotlin.*
kotlin.annotation.*
kotlin.collections.*
kotlin.comparisons.* （自 1.1 起）
kotlin.io.*
kotlin.ranges.*
kotlin.sequences.*
kotlin.text.*

根据目标平台还会导入额外的包：
JVM:
java.lang.*
kotlin.jvm.*
JS:
kotlin.js.*

可以导入一个单独的名字，如.
import org.example.Message // 现在 Message 可以不用限定符访问

也可以导入一个作用域下的所有内容（包、类、对象等）:
import org.example.* // “org.example”中的一切都可访问

如果出现名字冲突，可以使用 as 关键字在本地重命名冲突项来消歧义：
import org.example.Message // Message 可访问
import org.test.Message as testMessage // testMessage 代表“org.test.Message”

关键字 import 并不仅限于导入类；也可用它来导入其他声明：
顶层函数及属性；
在对象声明中声明的函数和属性;
枚举常量。

控制流：if、when、for、while

在 Kotlin 中，if是一个表达式，即它会返回一个值。 因此就不需要三元运算符（条件 ? 然后 : 否则），因为普通的 if 就能胜任这个角色。
。。可以 return if else ,, java 不能这样写。。 所以 kotlin不需要?:这个三目运算符了。

// 传统用法
var max = a
if (a < b) max = b

// With else
var max: Int
if (a > b) {
    max = a
} else {
    max = b
}

// 作为表达式
val max = if (a > b) a else b

if 的分支可以是代码块，最后的表达式作为该块的值：
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}

如果你使用 if 作为表达式而不是语句（例如：返回它的值或者把它赋给变量），该表达式需要有 else 分支。

when 表达式取代了类 C 语言的 switch 语句。其最简单的形式如下
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // 注意这个块
        print("x is neither 1 nor 2")
    }
}

when 将它的参数与所有的分支条件顺序比较，直到某个分支满足条件。

when既可以被当做表达式使用也可以被当做语句使用。如果它被当做表达式， 符合条件的分支的值就是整个表达式的值，如果当做语句使用， 则忽略个别分支的值。（像 if 一样，每一个分支可以是一个代码块，它的值是块中最后的表达式的值。）

如果 when 作为一个表达式使用，则必须有 else 分支，除非编译器能够检测出所有的可能情况都已经覆盖了［例如，对于 枚举（enum）类条目与密封（sealed）类子类型］

如果很多分支需要用相同的方式处理，则可以把多个分支条件放在一起，用逗号分隔：
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}

我们可以用任意表达式（而不只是常量）作为分支条件
when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
}

我们也可以检测一个值在（in）或者不在（!in）一个区间或者集合中：
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}

另一种可能性是检测一个值是（is）或者不是（!is）一个特定类型的值。注意： 由于智能转换，你可以访问该类型的方法与属性而无需任何额外的检测。
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}

when 也可以用来取代 if-else if链。 如果不提供参数，所有的分支条件都是简单的布尔表达式，而当一个分支的条件为真时则执行该分支：
when {
    x.isOdd() -> print("x is odd")
    y.isEven() -> print("y is even")
    else -> print("x+y is even.")
}
。。when后没有()

自 Kotlin 1.3 起，可以使用以下语法将 when 的主语（subject，译注：指 when 所判断的表达式）捕获到变量中：
fun Request.getBody() =
        when (val response = executeRequest()) {
            is Success -> response.body
            is HttpError -> throw HttpException(response.status)
        }
在 when 主语中引入的变量的作用域仅限于 when 主体。

for 循环可以对任何提供迭代器（iterator）的对象进行遍历，这相当于像 C# 这样的语言中的 foreach 循环。语法如下：
for (item in collection) print(item)

循环体可以是一个代码块。
for (item: Int in ints) {
    // ……
}

如上所述，for 可以循环遍历任何提供了迭代器的对象。即：

有一个成员函数或者扩展函数 iterator()，它的返回类型：
有一个成员函数或者扩展函数 next()，并且
有一个成员函数或者扩展函数 hasNext() 返回 Boolean。
这三个函数都需要标记为 operator。

for (i in 1..3) {
    println(i)
}
for (i in 6 downTo 0 step 2) {
    println(i)
}

如果你想要通过索引遍历一个数组或者一个 list，你可以这么做：
for (i in array.indices) {
    println(array[i])
}

或者你可以用库函数 withIndex：
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}

while 与 do..while 照常使用

while (x > 0) {
    x--
}

do {
  val y = retrieveData()
} while (y != null) // y 在此处可见

在循环中 Kotlin 支持传统的 break 与 continue 操作符。

Kotlin 有三种结构化跳转表达式：
return。默认从最直接包围它的函数或者匿名函数返回。
break。终止最直接包围它的循环。
continue。继续下一次最直接包围它的循环。

所有这些表达式都可以用作更大表达式的一部分：
val s = person.name ?: return

这些表达式的类型是 Nothing 类型。

在 Kotlin 中任何表达式都可以用标签（label）来标记。 标签的格式为标识符后跟 @ 符号，例如：abc@、fooBar@都是有效的标签。 要为一个表达式加标签，我们只要在其前加标签即可。

loop@ for (i in 1..100) {
    // ……
}

现在，我们可以用标签限制 break 或者continue：
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (……) break@loop
    }
}

Kotlin 有函数字面量、局部函数和对象表达式。因此 Kotlin 的函数可以被嵌套。 标签限制的 return 允许我们从外层函数返回。 最重要的一个用途就是从 lambda 表达式中返回。回想一下我们这么写的时候：

fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return // 非局部直接返回到 foo() 的调用者
        print(it)
    }
    println("this point is unreachable")
}

这个 return 表达式从最直接包围它的函数即 foo 中返回。 （注意，这种非局部的返回只支持传给内联函数的 lambda 表达式。） 如果我们需要从 lambda 表达式中返回，我们必须给它加标签并用以限制 return。

fun foo() {
    listOf(1, 2, 3, 4, 5).forEach lit@{
        if (it == 3) return@lit // 局部返回到该 lambda 表达式的调用者，即 forEach 循环
        print(it)
    }
    print(" done with explicit label")
}

现在，它只会从 lambda 表达式中返回。通常情况下使用隐式标签更方便。 该标签与接受该 lambda 的函数同名
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return@forEach // 局部返回到该 lambda 表达式的调用者，即 forEach 循环
        print(it)
    }
    print(" done with implicit label")
}

或者，我们用一个匿名函数替代 lambda 表达式。 匿名函数内部的 return 语句将从该匿名函数自身返回
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach(fun(value: Int) {
        if (value == 3) return  // 局部返回到匿名函数的调用者，即 forEach 循环
        print(value)
    })
    print(" done with anonymous function")
}

请注意，前文三个示例中使用的局部返回类似于在常规循环中使用 continue。并没有 break 的直接等价形式，不过可以通过增加另一层嵌套 lambda 表达式并从其中非局部返回来模拟：

。。？return 怎么可能是 continue，，应该是break啊。

fun foo() {
    run loop@{
        listOf(1, 2, 3, 4, 5).forEach {
            if (it == 3) return@loop // 从传入 run 的 lambda 表达式非局部返回
            print(it)
        }
    }
    print(" done with nested loop")
}

当要返一个回值的时候，解析器优先选用标签限制的 return，即
return@a 1
意为“返回 1 到 @a”，而不是“返回一个标签标注的表达式 (@a 1)”。

https://www.kotlincn.net/docs/reference/classes.html

Kotlin 中使用关键字 class 声明类
class Invoice { /*……*/ }

类声明由类名、类头（指定其类型参数、主构造函数等）以及由花括号包围的类体构成。类头与类体都是可选的； 如果一个类没有类体，可以省略花括号。
class Empty

在 Kotlin 中的一个类可以有一个主构造函数以及一个或多个次构造函数。主构造函数是类头的一部分：它跟在类名（与可选的类型参数）后。
class Person constructor(firstName: String) { /*……*/ }

如果主构造函数没有任何注解或者可见性修饰符，可以省略这个 constructor 关键字。
class Person(firstName: String) { /*……*/ }

主构造函数不能包含任何的代码。初始化的代码可以放到以 init 关键字作为前缀的初始化块（initializer blocks）中。

在实例初始化期间，初始化块按照它们出现在类体中的顺序执行，与属性初始化器交织在一起：

class InitOrderDemo(name: String) {
    val firstProperty = "First property: $name".also(::println)

    init {
        println("First initializer block that prints ${name}")
    }

    val secondProperty = "Second property: ${name.length}".also(::println)

    init {
        println("Second initializer block that prints ${name.length}")
    }
}

。。按顺序，， 多个init

请注意，主构造的参数可以在初始化块中使用。它们也可以在类体内声明的属性初始化器中使用：
class Customer(name: String) {
    val customerKey = name.toUpperCase()
}
。。等于真是一个方法了。。 customerKey是一个 局部变量的感觉。。
。。但是 name 不是代表了 一个 局部变量吗。。还是说 只有 data的时候 主构造器参数 就是 类成员？

事实上，声明属性以及从主构造函数初始化属性，Kotlin 有简洁的语法：

class Person(val firstName: String, val lastName: String, var age: Int) { /*……*/ }

与普通属性一样，主构造函数中声明的属性可以是可变的（var）或只读的（val）。

如果构造函数有注解或可见性修饰符，这个 constructor 关键字是必需的，并且这些修饰符在它前面：

class Customer public @Inject constructor(name: String) { /*……*/ }
。。public 在中间。

类也可以声明前缀有 constructor的次构造函数：

class Person {
    var children: MutableList<Person> = mutableListOf<>()
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
。。这是一个  没有形参的 主构造器吧，，

如果类有一个主构造函数，每个次构造函数需要委托给主构造函数， 可以直接委托或者通过别的次构造函数间接委托。委托到同一个类的另一个构造函数用   this   关键字即可：

。。这样说的话，上面没有委托，那就是说 上面 是没有 主构造器了？

class Person(val name: String) {
    var children: MutableList<Person> = mutableListOf<>()
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}

请注意，初始化块中的代码实际上会成为主构造函数的一部分。委托给主构造函数会作为次构造函数的第一条语句，因此所有初始化块与属性初始化器中的代码都会在次构造函数体之前执行。即使 该 类 没 有 主 构 造 函 数 ，这种委托仍会隐式发生，并且仍会执行初始化块：

class Constructors {
    init {
        println("Init block")
    }

    constructor(i: Int) {
        println("Constructor")
    }
}
。。照这里的说法， 这里是有 主构造器的，，但是 次构造器为什么没有 this？ 。。 不，这里说了 该类没有主构造器。。
。。所以 ： init会在 主构造器之前执行， 所以如果 次构造器 this了 主构造器，那么 先执行init，然后主构造器，然后
。。不，主构造器 是和 顺序有关的，是按序执行的。

。。是的，没有主构造器的，因为 new的时候 括号里空，是会报错的。。

。。感觉主构造器没有太大的意义，除了 简写。。因为 主构造器没有 方法体，所以只能靠 类内部的init 或者 变量初始化 来搞。。

如果一个非抽象类没有声明任何（主或次）构造函数，它会有一个生成的不带参数的主构造函数。构造函数的可见性是 public。如果你不希望你的类有一个公有构造函数，你需要声明一个带有非默认可见性的空的主构造函数：

class DontCreateMe private constructor () { /*……*/ }
。。不写 主、次 构造器，就生成一个 默认的 无参构造器。  和java一样。
。。。。。这里的private是修饰 constructor的。。我

在 JVM 上，如果主构造函数的所有的参数都有默认值，编译器会生成 一个额外的无参构造函数，它将使用默认值。这使得 Kotlin 更易于使用像 Jackson 或者 JPA 这样的通过无参构造函数创建类的实例的库。

在 JVM 上，如果主构造函数的所有的参数都有默认值，编译器会生成 一个额外的无参构造函数，它将使用默认值。这使得 Kotlin 更易于使用像 Jackson 或者 JPA 这样的通过无参构造函数创建类的实例的库。

class Customer(val customerName: String = "")
。。主构造器 都有默认值。  不管次构造器。

要创建一个类的实例，我们就像普通函数一样调用构造函数：
val invoice = Invoice()
val customer = Customer("Joe Smith")

类可以包含：
构造函数与初始化块
函数
属性
嵌套类与内部类
对象声明

在 Kotlin 中所有类都有一个共同的超类 Any，这对于没有超类型声明的类是默认超类：
class Example // 从 Any 隐式继承

Any 有三个方法：equals()、 hashCode() 与 toString()。因此，为所有 Kotlin 类都定义了这些方法。

默认情况下，Kotlin 类是最终（final）的：它们不能被继承。 要使一个类可继承，请用 open 关键字标记它。
。。默认 final。。 除非 open修饰
open class Base // 该类开放继承

zzzzz。。找不到 Any这个类，， kotlin runtime也好像没有，，而且 这个 runtime 插件在哪里 也找不到。。

如需声明一个显式的超类型，请在类头中把超类型放到冒号之后：
open class Base(p: Int)
class Derived(p: Int) : Base(p)

如果派生类有一个主构造函数，其基类可以（并且必须） 用派生类主构造函数的参数就地初始化。

如果派生类没有主构造函数，那么每个次构造函数必须使用 super 关键字初始化其基类型，或委托给另一个构造函数做到这一点。 注意，在这种情况下，不同的次构造函数可以调用基类型的不同的构造函数：

class MyView : View {
    constructor(ctx: Context) : super(ctx)
    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
。。子类有主构造器，那么 父类必须 能用 子类主构造器的参数 初始化。
。。等于就是  子类的主构造器 必须 匹配 父类的 某个 构造器啊。
。。 是父类限制 子类，， 这里讲的 像 子类限制父类。。。

我们之前提到过，Kotlin 力求清晰显式。因此，Kotlin 对于可覆盖的成员（我们称之为开放）以及覆盖后的成员需要显式修饰符：

open class Shape {
    open fun draw() { /*……*/ }
    fun fill() { /*……*/ }
}

class Circle() : Shape() {
    override fun draw() { /*……*/ }
}

Circle.draw() 函数上必须加上 override 修饰符。如果没写，编译器将会报错。 如果函数没有标注 open 如 Shape.fill()，那么子类中不允许定义相同签名的函数， 不论加不加 override。将 open 修饰符添加到 final 类（即没有 open 的类）的成员上不起作用。

。。子类重写父类的方法，必须：子类方法写override，父类是open，被重写的方法是open。。 缺一不可。

标记为 override 的成员本身是开放的，也就是说，它可以在子类中覆盖。如果你想禁止再次覆盖，使用 final 关键字：
open class Rectangle() : Shape() {
    final override fun draw() { /*……*/ }
}
。。一旦标记为override，那么就默认是open的。
。。。那如果 子类不是open的， 那还会被子子类覆盖吗？. 不能，ide错误。子类要写open 才能被继承，而且不要求 子类方法写open
。。子类的override方法可以加 open，加不加open，子子类 都可以 override。

属性覆盖与方法覆盖类似；在超类中声明然后在派生类中重新声明的属性必须以 override 开头，并且它们必须具有兼容的类型。 每个声明的属性可以由具有初始化器的属性或者具有 get 方法的属性覆盖。

open class Shape {
    open val vertexCount: Int = 0
}

class Rectangle : Shape() {
    override val vertexCount = 4
}

。。属性覆盖，似乎无意义啊。。。no。。

你也可以用一个 var 属性覆盖一个 val 属性，但反之则不行。 这是允许的，因为一个 val 属性本质上声明了一个 get 方法， 而将其覆盖为 var 只是在子类中额外声明一个 set 方法。

请注意，你可以在主构造函数中使用 override 关键字作为属性声明的一部分。

interface Shape {
    val vertexCount: Int
}

class Rectangle(override val vertexCount: Int = 4) : Shape // 总是有 4 个顶点

class Polygon : Shape {
    override var vertexCount: Int = 0  // 以后可以设置为任何数
}

在构造派生类的新实例的过程中，第一步完成其基类的初始化（在之前只有对基类构造函数参数的求值），因此发生在派生类的初始化逻辑运行之前。
open class Base(val name: String) {

    init { println("Initializing Base") }

    open val size: Int =
        name.length.also { println("Initializing size in Base: $it") }
}

class Derived(
    name: String,
    val lastName: String,
) : Base(name.capitalize().also { println("Argument for Base: $it") }) {

    init { println("Initializing Derived") }

    override val size: Int =

        (super.size + lastName.length).also { println("Initializing size in Derived: $it") }

}

这意味着，基类构造函数执行时，派生类中声明或覆盖的属性都还没有初始化。如果在基类初始化逻辑中（直接或通过另一个覆盖的 open 成员的实现间接）使用了任何一个这种属性，那么都可能导致不正确的行为或运行时故障。设计一个基类时，应该避免在构造函数、属性初始化器以及 init 块中使用 open 成员。

zzzzz。。父类初始化时 用到了 一个属性，这个属性被子类覆盖了，所以 不知道这个属性 到底是个什么了。。。但是这个属性也需要被初始化吧，初始化的是多少？还是说直接删除了？

派生类中的代码可以使用 super 关键字调用其超类的函数与属性访问器的实现

在一个内部类中访问外部类的超类，可以通过由外部类名限定的 super 关键字来实现：super@Outer
class FilledRectangle: Rectangle() {
    fun draw() { /* …… */ }
    val borderColor: String get() = "black"

    inner class Filler {
        fun fill() { /* …… */ }
        fun drawAndFill() {
            super@FilledRectangle.draw() // 调用 Rectangle 的 draw() 实现
            fill()

            println("Drawn a filled rectangle with color ${super@FilledRectangle.borderColor}") // 使用 Rectangle 所实现的 borderColor 的 get()

        }
    }
}

在 Kotlin 中，实现继承由下述规则规定：如果一个类从它的直接超类继承相同成员的多个实现， 它必须覆盖这个成员并提供其自己的实现（也许用继承来的其中之一）。 为了表示采用从哪个超类型继承的实现，我们使用由尖括号中超类型名限定的 super，如 super<Base>：

。。这里还有 接口。。。

open class Rectangle {
    open fun draw() { /* …… */ }
}

interface Polygon {
    fun draw() { /* …… */ } // 接口成员默认就是“open”的
}

class Square() : Rectangle(), Polygon {
    // 编译器要求覆盖 draw()：
    override fun draw() {
        super<Rectangle>.draw() // 调用 Rectangle.draw()
        super<Polygon>.draw() // 调用 Polygon.draw()
    }
}

可以同时继承 Rectangle 与 Polygon， 但是二者都有各自的 draw() 实现，所以我们必须在 Square 中覆盖 draw()， 并提供其自身的实现以消除歧义。

类以及其中的某些成员可以声明为 abstract。 抽象成员在本类中可以不用实现。 需要注意的是，我们并不需要用 open 标注一个抽象类或者函数——因为这不言而喻。

我们可以用一个抽象成员覆盖一个非抽象的开放成员

open class Polygon {
    open fun draw() {}
}

abstract class Rectangle : Polygon() {
    abstract override fun draw()
}

伴生对象
如果你需要写一个可以无需用一个类的实例来调用、但需要访问类内部的函数（例如，工厂方法），你可以把它写成该类内对象声明中的一员。

更具体地讲，如果在你的类内声明了一个伴生对象， 你就可以访问其成员，只是以类名作为限定符。
类内部的对象声明可以用 companion 关键字标记
该伴生对象的成员可通过只使用类名作为限定符来调用
可以省略伴生对象的名称，在这种情况下将使用名称 Companion
class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
}

val instance = MyClass.create()

class MyClass {
    companion object { }
}

val x = MyClass.Companion

Kotlin 类中的属性既可以用关键字 var 声明为可变的，也可以用关键字 val 声明为只读的。
class Address {
    var name: String = "Holmes, Sherlock"
    var street: String = "Baker"
    var city: String = "London"
    var state: String? = null
    var zip: String = "123456"
}

要使用一个属性，只要用名称引用它即可：
fun copyAddress(address: Address): Address {
    val result = Address() // Kotlin 中没有“new”关键字
    result.name = address.name // 将调用访问器
    result.street = address.street
    // ……
    return result
}

声明一个属性的完整语法是
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]

其初始器（initializer）、getter 和 setter 都是可选的。属性类型如果可以从初始器 （或者从其 getter 返回值，如下文所示）中推断出来，也可以省略。

var allByDefault: Int? // 错误：需要显式初始化器，隐含默认 getter 和 setter
var initialized = 1 // 类型 Int、默认 getter 和 setter

一个只读属性的语法和一个可变的属性的语法有两方面的不同：1、只读属性的用 val开始代替var 2、只读属性不允许 setter
val simple: Int? // 类型 Int、默认 getter、必须在构造函数中初始化
val inferredType = 1 // 类型 Int 、默认 getter
。。。因为是val，所以必须有 初始化，所以 就不会报错，，估计是报另一种错。

我们可以为属性定义自定义的访问器。如果我们定义了一个自定义的 getter，那么每次访问该属性时都会调用它 （这让我们可以实现计算出的属性）。以下是一个自定义 getter 的示例：

val isEmpty: Boolean
    get() = this.size == 0

。。话说，hbm能映射kotlin？

如果我们定义了一个自定义的 setter，那么每次给属性赋值时都会调用它。一个自定义的 setter 如下所示：
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // 解析字符串并赋值给其他属性
    }

自 Kotlin 1.1 起，如果可以从 getter 推断出属性类型，则可以省略它：
val isEmpty get() = this.size == 0  // 具有类型 Boolean

如果你需要改变一个访问器的可见性或者对其注解，但是不需要改变默认的实现， 你可以定义访问器而不定义其实现:
var setterVisibility: String = "abc"
    private set // 此 setter 是私有的并且有默认实现
var setterWithAnnotation: Any? = null
    @Inject set // 用 Inject 注解此 setter

幕后字段
在 Kotlin 类中不能直接声明字段。然而，当一个属性需要一个幕后字段时，Kotlin 会自动提供。这个幕后字段可以使用field标识符在访问器中引用：

var counter = 0 // 注意：这个初始器直接为幕后字段赋值
    set(value) {
        if (value >= 0) field = value
    }

。。定义了个counter 类变量，写setter时，如果counter=value，那么又会调用setter方法，会导致stackoverflow。所以用field来代替类变量。

field 标识符只能用在属性的访问器内

如果属性至少一个访问器使用默认实现，或者自定义访问器通过 field 引用幕后字段，将会为该属性生成一个幕后字段。

例如，下面的情况下， 就没有幕后字段：
val isEmpty: Boolean
    get() = this.size == 0

幕后属性
如果你的需求不符合这套“隐式的幕后字段”方案，那么总可以使用 幕后属性（backing property）：

private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap() // 类型参数已推断出
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }

如果只读属性的值在编译期是已知的，那么可以使用 const 修饰符将其标记为编译期常量。 这种属性需要满足以下要求：
位于顶层或者是 object 声明 或 companion object 的一个成员
以 String 或原生类型值初始化
没有自定义 getter

这些属性可以用在注解中：
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"
@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() { …… }

一般地，属性声明为非空类型必须在构造函数中初始化。 然而，这经常不方便。例如：属性可以通过依赖注入来初始化， 或者在单元测试的 setup 方法中初始化。 这种情况下，你不能在构造函数内提供一个非空初始器。 但你仍然想在类体中引用该属性时避免空检测。

为处理这种情况，你可以用 lateinit 修饰符标记该属性：

public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method()  // 直接解引用
    }
}

该修饰符只能用于在类体中的属性（不是在主构造函数中声明的 var 属性，并且仅当该属性没有自定义 getter 或 setter 时），而自 Kotlin 1.2 起，也用于顶层属性与局部变量。该属性或变量必须为非空类型，并且不能是原生类型。

在初始化前访问一个 lateinit 属性会抛出一个特定异常，该异常明确标识该属性被访问及它没有初始化的事实。

要检测一个 lateinit var 是否已经初始化过，请在该属性的引用上使用 .isInitialized：
if (foo::bar.isInitialized) {
    println(foo.bar)
}

此检测仅对可词法级访问的属性可用，即声明位于同一个类型内、位于其中一个外围类型中或者位于相同文件的顶层的属性。

委托属性

最常见的一类属性就是简单地从幕后字段中读取（以及可能的写入）。 另一方面，使用自定义 getter 和 setter 可以实现属性的任何行为。 介于两者之间，属性如何工作有一些常见的模式。一些例子：惰性值、 通过键值从映射读取、访问数据库、访问时通知侦听器等等。

Kotlin 的接口可以既包含抽象方法的声明也包含实现。与抽象类不同的是，接口无法保存状态。它可以有属性但必须声明为抽象或提供访问器实现。

使用关键字 interface 来定义接口
interface MyInterface {
    fun bar()
    fun foo() {
      // 可选的方法体
    }
}

https://www.kotlincn.net/docs/reference/interfaces.html

Kotlin 的接口可以既包含抽象方法的声明也包含实现。与抽象类不同的是，接口无法保存状态。它可以有属性但必须声明为抽象或提供访问器实现。

使用关键字 interface 来定义接口

interface MyInterface {
    fun bar()
    fun foo() {
      // 可选的方法体
    }
}

实现接口
一个类或者对象可以实现一个或多个接口。
class Child : MyInterface {
    override fun bar() {
        // 方法体
    }
}

你可以在接口中定义属性。在接口中声明的属性要么是抽象的，要么提供访问器的实现。在接口中声明的属性不能有幕后字段（backing field），因此接口中声明的访问器不能引用它们。

。。一旦访问的话，直接就 死循环了。

interface MyInterface {
    val prop: Int // 抽象的

    val propertyWithImplementation: String
        get() = "foo"

    fun foo() {
        print(prop)
    }
}

class Child : MyInterface {
    override val prop: Int = 29
}

一个接口可以从其他接口派生，从而既提供基类型成员的实现也声明新的函数与属性。很自然地，实现这样接口的类只需定义所缺少的实现：

interface Named {
    val name: String
}

interface Person : Named {
    val firstName: String
    val lastName: String

    override val name: String get() = "$firstName $lastName"
}

data class Employee(
    // 不必实现“name”
    override val firstName: String,
    override val lastName: String,
    val position: Position
) : Person

实现多个接口时，可能会遇到同一方法继承多个实现的问题。例如
interface A {
    fun foo() { print("A") }
    fun bar()
}

interface B {
    fun foo() { print("B") }
    fun bar() { print("bar") }
}

class C : A {
    override fun bar() { print("bar") }
}

class D : A, B {
    override fun foo() {
        super<A>.foo()
        super<B>.foo()
    }

    override fun bar() {
        super<B>.bar()
    }
}

上例中，接口 A 和 B 都定义了方法 foo() 和 bar()。 两者都实现了 foo(), 但是只有 B 实现了 bar() (bar() 在 A 中没有标记为抽象， 因为在接口中没有方法体时默认为抽象）。因为 C 是一个实现了 A 的具体类，所以必须要重写 bar() 并实现这个抽象方法。

然而，如果我们从 A 和 B 派生 D，我们需要实现我们从多个接口继承的所有方法，并指明 D 应该如何实现它们。这一规则既适用于继承单个实现（bar()）的方法也适用于继承多个实现（foo()）的方法。

https://www.kotlincn.net/docs/reference/fun-interfaces.html

An interface with only one abstract method is called a functional interface, or a Single Abstract Method (SAM) interface.

。。只有一个抽象方法的接口就是 函数式接口，或简单抽象方法接口。

To declare a functional interface in Kotlin, use the *fun* modifier.
fun interface KRunnable {
   fun invoke()
}

For functional interfaces, you can use SAM conversions that help make your code more concise and readable by using lambda expressions.

Instead of creating a class that implements a functional interface manually, you can use a lambda expression. With a SAM conversion, Kotlin can convert any lambda expression whose signature matches the signature of the interface's single method into an instance of a class that implements the interface.

。使用lambda表达式，而不是实现函数式接口。

For example, consider the following Kotlin functional interface:
fun interface IntPredicate {
   fun accept(i: Int): Boolean
}

If you don't use a SAM conversion, you will need to write code like this:
// Creating an instance of a class
val isEven = object : IntPredicate {
   override fun accept(i: Int): Boolean {
       return i % 2 == 0
   }
}

By leveraging Kotlin's SAM conversion, you can write the following equivalent code instead:

// Creating an instance using lambda
val isEven = IntPredicate { it % 2 == 0 }

fun interface IntPredicate {
   fun accept(i: Int): Boolean
}

val isEven = IntPredicate { it % 2 == 0 }

fun main() {
   println("Is 7 even? - ${isEven.accept(7)}")
}

https://www.kotlincn.net/docs/reference/visibility-modifiers.html

类、对象、接口、构造函数、方法、属性和它们的 setter 都可以有 可见性修饰符。 （getter 总是与属性有着相同的可见性。） 在 Kotlin 中有这四个可见性修饰符：private、 protected、 internal 和 public。 如果没有显式指定修饰符的话，默认可见性是 public。

函数、属性和类、对象和接口可以在顶层声明，即直接在包内：
// 文件名：example.kt
package foo

fun baz() { …… }
class Bar { …… }

如果你不指定任何可见性修饰符，默认为 public，这意味着你的声明将随处可见；
如果你声明为 private，它只会在声明它的文件内可见；
如果你声明为 internal，它会在相同模块内随处可见；
protected 不适用于顶层声明。

注意：要使用另一包中可见的顶层声明，仍需将其导入进来。

// 文件名：example.kt
package foo

private fun foo() { …… } // 在 example.kt 内可见

public var bar: Int = 5 // 该属性随处可见
    private set         // setter 只在 example.kt 内可见

internal val baz = 6    // 相同模块内可见

。。类属性private，并不是 类中可见，还是 文件可见。。。
。。但是转为 java ，怎么搞？ java没有这种 可见性啊。。。
。。这里不是类。。。是静态变量。。。

package com.kt;
import kotlin.Metadata;

@Metadata(mv={1, 4, 0}, bv={1, 0, 3}, k=2, d1={""}, d2={"<set-?>", "", "bar", "getBar", "()I", "baz", "getBaz", "foo", ""})

public final class VisibilityKt
{
  private static int bar = 5;

  private static final int baz = 6;

  private static final void foo()
  {
    bar = 10;
  }
  public static final int getBar() {
    return bar;
  }
  public static final int getBaz() {
    return baz;
  }
}
。。同一个文件内 直接 访问属性。。。对吗？好像不是的啊。。还是说优先setter，如果没有，则用属性。
。。静态变量啊。。

package com.kt;
import java.io.PrintStream;
import kotlin.Metadata;
@Metadata(mv={1, 4, 0}, bv={1, 0, 3}, k=2, d1={""}, d2={"main", "", "test"})
public final class Visibility2Kt
{
  public static final void test()
  {
    int i = VisibilityKt.getBar(); int j = 0; System.out.println(i);
  }
  public static final void main()
  {
    test();
  }
}
。。不同文件内，通过 setter，由于bar 没有setter方法，所以。
。。静态变量啊。。。

类和接口
对于类内部声明的成员：
private 意味着只在这个类内部（包含其所有成员）可见；
protected—— 和 private一样 + 在子类中可见。
internal —— 能见到类声明的 本模块内 的任何客户端都可见其 internal 成员；
public —— 能见到类声明的任何客户端都可见其 public 成员。

请注意在 Kotlin 中，外部类不能访问内部类的 private 成员。

如果你覆盖一个 protected 成员并且没有显式指定其可见性，该成员还会是 protected 可见性。

open class Outer {
    private val a = 1
    protected open val b = 2
    internal val c = 3
    val d = 4  // 默认 public

    protected class Nested {
        public val e: Int = 5
    }
}

class Subclass : Outer() {
    // a 不可见
    // b、c、d 可见
    // Nested 和 e 可见

    override val b = 5   // “b”为 protected
}

class Unrelated(o: Outer) {
    // o.a、o.b 不可见
    // o.c 和 o.d 可见（相同模块）
    // Outer.Nested 不可见，Nested::e 也不可见
}

要指定一个类的的主构造函数的可见性，使用以下语法（注意你需要添加一个显式 constructor 关键字）：
class C private constructor(a: Int) { …… }

这里的构造函数是私有的。默认情况下，所有构造函数都是 public，这实际上等于类可见的地方它就可见（即 一个 internal 类的构造函数只能在相同模块内可见).

局部变量、函数和类不能有可见性修饰符。

可见性修饰符 internal 意味着该成员只在相同模块内可见。更具体地说， 一个模块是编译在一起的一套 Kotlin 文件：

一个 IntelliJ IDEA 模块；
一个 Maven 项目；
一个 Gradle 源集（例外是 test 源集可以访问 main 的 internal 声明）；
一次 <kotlinc> Ant 任务执行所编译的一套文件。

Kotlin 能够扩展一个类的新功能而无需继承该类或者使用像装饰者这样的设计模式。 这通过叫做 扩展 的特殊声明完成。 例如，你可以为一个你不能修改的、来自第三方库中的类编写一个新的函数。 这个新增的函数就像那个原始类本来就有的函数一样，可以用普通的方法调用。 这种机制称为 扩展函数 。此外，也有 扩展属性 ， 允许你为一个已经存在的类添加新的属性。

声明一个扩展函数，我们需要用一个 接收者类型 也就是被扩展的类型来作为他的前缀。 下面代码为 MutableList<Int> 添加一个swap 函数：

fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // “this”对应该列表
    this[index1] = this[index2]
    this[index2] = tmp
}

这个 this 关键字在扩展函数内部对应到接收者对象（传过来的在点符号前的对象） 现在，我们对任意 MutableList<Int> 调用该函数了：

当然，这个函数对任何 MutableList<T> 起作用，我们可以泛化它：
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // “this”对应该列表
    this[index1] = this[index2]
    this[index2] = tmp
}

扩展是静态解析的
扩展不能真正的修改他们所扩展的类。通过定义一个扩展，你并没有在一个类中插入新成员， 仅仅是可以通过该类型的变量用点表达式去调用这个新函数。
。。等于是个语法糖啊，吧上面的swap 扩展成 swap(List, int, int)，这样一个static函数？ 是的。

fun String.showq() {
println(this + ", show.")
}
----转换后：
  public static final void showq(@NotNull String $this$showq) {

    Intrinsics.checkNotNullParameter($this$showq, "$this$showq"); String str = $this$showq + ", show."; int i = 0; System.out.println(str);

  }

我们想强调的是扩展函数是静态分发的，即他们不是根据接收者类型的虚方法。 这意味着调用的扩展函数是由函数调用所在的表达式的类型来决定的， 而不是由表达式运行时求值结果决定的。例如：

。。就是不能多态。

open class Shape
class Rectangle: Shape()
fun Shape.getName() = "Shape"
fun Rectangle.getName() = "Rectangle"
fun printClassName(s: Shape) {
    println(s.getName())
}
printClassName(Rectangle())

这个例子会输出 "Shape"，因为调用的扩展函数只取决于参数 s 的声明类型，该类型是 Shape 类。

如果一个类定义有一个成员函数与一个扩展函数，而这两个函数又有相同的接收者类型、 相同的名字，并且都适用给定的参数，这种情况总是取成员函数。 例如：
class Example {
    fun printFunctionType() { println("Class method") }
}
fun Example.printFunctionType() { println("Extension function") }
Example().printFunctionType()

这段代码输出“Class method”。

当然，扩展函数重载同样名字但不同签名成员函数也完全可以：
class Example {
    fun printFunctionType() { println("Class method") }
}
fun Example.printFunctionType(i: Int) { println("Extension function") }
Example().printFunctionType(1)

。。估计是extension function

可空接收者

注意可以为可空的接收者类型定义扩展。这样的扩展可以在对象变量上调用， 即使其值为 null，并且可以在函数体内检测 this == null，这能让你在没有检测 null 的时候调用 Kotlin 中的toString()：检测发生在扩展函数的内部。

fun Any?.toString(): String {
    if (this == null) return "null"
    // 空检测之后，“this”会自动转换为非空类型，所以下面的 toString()
    // 解析为 Any 类的成员函数
    return toString()
}

与函数类似，Kotlin 支持扩展属性：
val <T> List<T>.lastIndex: Int
    get() = size - 1

注意：由于扩展没有实际的将成员插入类中，因此对扩展属性来说幕后字段是无效的。这就是为什么扩展属性不能有初始化器。他们的行为只能由显式提供的 getters/setters 定义。

例如:
val House.number = 1 // 错误：扩展属性不能有初始化器

伴生对象的扩展
如果一个类定义有一个伴生对象 ，你也可以为伴生对象定义扩展函数与属性。就像伴生对象的常规成员一样， 可以只使用类名作为限定符来调用伴生对象的扩展成员：
class MyClass {
    companion object { }  // 将被称为 "Companion"
}
fun MyClass.Companion.printCompanion() { println("companion") }
fun main() {
    MyClass.printCompanion()
}

扩展的作用域
大多数时候我们在顶层定义扩展——直接在包里：
package org.example.declarations
fun List<String>.getLongestString() { /*……*/}

要使用所定义包之外的一个扩展，我们需要在调用方导入它：
package org.example.usage
import org.example.declarations.getLongestString
fun main() {
    val list = listOf("red", "green", "blue")
    list.getLongestString()
}

扩展声明为成员

在一个类内部你可以为另一个类声明扩展。在这样的扩展内部，有多个 隐式接收者 —— 其中的对象成员可以无需通过限定符访问。扩展声明所在的类的实例称为 分发接收者，扩展方法调用所在的接收者类型的实例称为 扩展接收者 。

class Host(val hostname: String) {
    fun printHostname() { print(hostname) }
}

class Connection(val host: Host, val port: Int) {
     fun printPort() { print(port) }

     fun Host.printConnectionString() {
         printHostname()   // 调用 Host.printHostname()
         print(":")
         printPort()   // 调用 Connection.printPort()
     }

     fun connect() {
         /*……*/
         host.printConnectionString()   // 调用扩展函数
     }
}

fun main() {
    Connection(Host("kotl.in"), 443).connect()
    //Host("kotl.in").printConnectionString(443)  // 错误，该扩展函数在 Connection 外不可用
}

对于分发接收者与扩展接收者的成员名字冲突的情况，扩展接收者优先。要引用分发接收者的成员你可以使用 限定的 this 语法。
class Connection {
    fun Host.getConnectionString() {
        toString()         // 调用 Host.toString()
        this@Connection.toString()  // 调用 Connection.toString()
    }
}
。。扩展接受者 是host，Connection是分发接受者。
。。如果在 getConnectionString里 再来一个 扩展，新扩展能访问到Connection吗？

声明为成员的扩展可以声明为 open 并在子类中覆盖。这意味着这些函数的分发对于分发接收者类型是虚拟的，但对于扩展接收者类型是静态的。

扩展的可见性与相同作用域内声明的其他实体的可见性相同。例如：
在文件顶层声明的扩展可以访问同一文件中的其他 private 顶层声明；
如果扩展是在其接收者类型外部声明的，那么该扩展不能访问接收者的 private 成员。

我们经常创建一些只保存数据的类。 在这些类中，一些标准函数往往是从数据机械推导而来的。在 Kotlin 中，这叫做 数据类 并标记为 data：
data class User(val name: String, val age: Int)

编译器自动从主构造函数中声明的所有属性导出以下成员：
equals()/hashCode() 对；
toString() 格式是 "User(name=John, age=42)"；
componentN() 函数 按声明顺序对应于所有属性；
copy() 函数（见下文）。

为了确保生成的代码的一致性以及有意义的行为，数据类必须满足以下要求：
主构造函数需要至少有一个参数；
主构造函数的所有参数需要标记为 val 或 var；
数据类不能是抽象、开放、密封或者内部的；
（在1.1之前）数据类只能实现接口。

。。才发现，之前的class 的 默认构造器 可以不带 val/var。。。
。。但是它们会在 类内部 声明 val或var 变量。。。

此外，成员生成遵循关于成员继承的这些规则：

如果在数据类体中有显式实现 equals()、 hashCode() 或者 toString()，或者这些函数在父类中有 final 实现，那么不会生成这些函数，而会使用现有函数；

如果超类型具有 open 的 componentN() 函数并且返回兼容的类型， 那么会为数据类生成相应的函数，并覆盖超类的实现。如果超类型的这些函数由于签名不兼容或者是 final 而导致无法覆盖，那么会报错；

从一个已具 copy(……) 函数且签名匹配的类型派生一个数据类在 Kotlin 1.2 中已弃用，并且在 Kotlin 1.3 中已禁用。
不允许为 componentN() 以及 copy() 函数提供显式实现。

自 1.1 起，数据类可以扩展其他类

在 JVM 中，如果生成的类需要含有一个无参的构造函数，则所有的属性必须指定默认值。 （参见构造函数）。
data class User(val name: String = "", val age: Int = 0)

在类体中声明的属性
请注意，对于那些自动生成的函数，编译器只使用在主构造函数内部定义的属性。如需在生成的实现中排除一个属性，请将其声明在类体中：
data class Person(val name: String) {
    var age: Int = 0
}

在 toString()、 equals()、 hashCode() 以及 copy() 的实现中只会用到 name 属性，并且只有一个 component 函数 component1()。虽然两个 Person 对象可以有不同的年龄，但它们会视为相等。

复制
在很多情况下，我们需要复制一个对象改变它的一些属性，但其余部分保持不变。 copy() 函数就是为此而生成。对于上文的 User 类，其实现会类似下面这样：
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)

。。那Person可以吗？主要Person只有一个 单形参的 构造器，所以 copy的时候 ，，估计得需要自己写了。。而且 copy 形参 是怎么规定的？是所有类变量，还是 主构造器的变量？

这让我们可以写：
val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)

数据类与解构声明
为数据类生成的 Component 函数 使它们可在解构声明中使用：
val jane = User("Jane", 35)
val (name, age) = jane
println("$name, $age years of age") // 输出 "Jane, 35 years of age"

。。Component函数 是不是指 getter setter？

标准数据类
标准库提供了 Pair 与 Triple。尽管在很多情况下具名数据类是更好的设计选择， 因为它们通过为属性提供有意义的名称使代码更具可读性。

https://www.kotlincn.net/docs/reference/sealed-classes.html

密封类用来表示受限的类继承结构：当一个值为有限几种的类型、而不能有任何其他类型时。在某种意义上，他们是枚举类的扩展：枚举类型的值集合也是受限的，但每个枚举常量只存在一个实例，而密封类的一个子类可以有可包含状态的多个实例。

。。这里的值，是指 该类的对象 只有几种，还是说 该类的类属性 的 类型 只有几种， 结合说的 枚举， 感觉是 前者。。。前者，下面说了构造器private，。。。不，再下面的一个例子。。感觉 密封类 data后的类 可以任意new吧。。

要声明一个密封类，需要在类名前面添加 sealed 修饰符。虽然密封类也可以有子类，但是所有子类都必须在与密封类自身相同的文件中声明。（在 Kotlin 1.1 之前， 该规则更加严格：子类必须嵌套在密封类声明的内部）。

sealed class Expr
data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()
（上文示例使用了 Kotlin 1.1 的一个额外的新功能：数据类扩展包括密封类在内的其他类的可能性。 ）

一个密封类是自身抽象的，它不能直接实例化并可以有抽象（abstract）成员。
密封类不允许有非-private 构造函数（其构造函数默认为 private）。

请注意，扩展密封类子类的类（间接继承者）可以放在任何位置，而无需在同一个文件中。

使用密封类的关键好处在于使用 when 表达式 的时候，如果能够验证语句覆盖了所有情况，就不需要为该语句再添加一个 else 子句了。当然，这只有当你用 when 作为表达式（使用结果）而不是作为语句时才有用。

fun eval(expr: Expr): Double = when(expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    // 不再需要 `else` 子句，因为我们已经覆盖了所有的情况
}
。。NotANumber是一个对象。。。
。。Const，Sum这种的 构造器是私有的吗？，，感觉肯定不是啊。。那上面说的，岂不是指 类属性的类型 限制。。

https://www.kotlincn.net/docs/reference/generics.html

与 Java 类似，Kotlin 中的类也可以有类型参数：
class Box<T>(t: T) {
    var value = t
}

一般来说，要创建这样类的实例，我们需要提供类型参数：
val box: Box<Int> = Box<Int>(1)

但是如果类型参数可以推断出来，例如从构造函数的参数或者从其他途径，允许省略类型参数：
val box = Box(1) // 1 具有类型 Int，所以编译器知道我们说的是 Box<Int>。

型变
Java 类型系统中最棘手的部分之一是通配符类型（参见 Java Generics FAQ）。 而 Kotlin 中没有。

相反，它有两个其他的东西：声明处型变（declaration-site variance）与类型投影（type projections）。

首先，让我们思考为什么 Java 需要那些神秘的通配符。在 《Effective Java》第三版 解释了该问题——第 31 条：利用有限制通配符来提升 API 的灵活性。 首先，Java 中的泛型是不型变的，这意味着 List<String> 并不是 List<Object> 的子类型。 为什么这样？ 如果 List 不是不型变的，它就没比 Java 的数组好到哪去，因为如下代码会通过编译然后导致运行时异常：

// Java
List<String> strs = new ArrayList<String>();
List<Object> objs = strs; // ！！！此处的编译器错误让我们避免了之后的运行时异常
objs.add(1); // 这里我们把一个整数放入一个字符串列表
String s = strs.get(0); // ！！！ ClassCastException：无法将整数转换为字符串

因此，Java 禁止这样的事情以保证运行时的安全。但这样会有一些影响。例如，考虑 Collection 接口中的 addAll() 方法。该方法的签名应该是什么？直觉上，我们会这样：

// Java
interface Collection<E> …… {
  void addAll(Collection<E> items);
}
但随后，我们就无法做到以下简单的事情（这是完全安全）：

// Java
void copyAll(Collection<Object> to, Collection<String> from) {
  to.addAll(from);
  // ！！！对于这种简单声明的 addAll 将不能编译：
  // Collection<String> 不是 Collection<Object> 的子类型
}
（在 Java 中，我们艰难地学到了这个教训，参见《Effective Java》第三版，第 28 条：列表优先于数组）

这就是为什么 addAll() 的实际签名是以下这样：

// Java
interface Collection<E> …… {
  void addAll(Collection<? extends E> items);
}

通配符类型参数 ? extends E 表示此方法接受 E 或者 E 的 一些子类型对象的集合，而不只是 E 自身。 这意味着我们可以安全地从其中（该集合中的元素是 E 的子类的实例）读取 E，但不能写入， 因为我们不知道什么对象符合那个未知的 E 的子类型。 反过来，该限制可以让Collection<String>表示为Collection<? extends Object>的子类型。 简而言之，带 extends 限定（上界）的通配符类型使得类型是协变的（covariant）。

理解为什么这个技巧能够工作的关键相当简单：如果只能从集合中获取项目，那么使用 String 的集合， 并且从其中读取 Object 也没问题 。反过来，如果只能向集合中 放入 项目，就可以用 Object 集合并向其中放入 String：在 Java 中有 List<? super String> 是 List<Object> 的一个超类。

后者称为逆变性（contravariance），并且对于 List <? super String> 你只能调用接受 String 作为参数的方法 （例如，你可以调用 add(String) 或者 set(int, String)），当然如果调用函数返回 List<T> 中的 T，你得到的并非一个 String 而是一个 Object。

Joshua Bloch 称那些你只能从中读取的对象为生产者，并称那些你只能写入的对象为消费者。他建议：“为了灵活性最大化，在表示生产者或消费者的输入参数上使用通配符类型”，并提出了以下助记符：

PECS 代表生产者-Extends、消费者-Super（Producer-Extends, Consumer-Super）。

注意：如果你使用一个生产者对象，如 List<? extends Foo>，在该对象上不允许调用 add() 或 set()。但这并不意味着该对象是不可变的：例如，没有什么阻止你调用 clear()从列表中删除所有项目，因为 clear() 根本无需任何参数。通配符（或其他类型的型变）保证的唯一的事情是类型安全。不可变性完全是另一回事。

声明处型变
假设有一个泛型接口 Source<T>，该接口中不存在任何以 T 作为参数的方法，只是方法返回 T 类型值：
// Java
interface Source<T> {
  T nextT();
}

那么，在 Source <Object> 类型的变量中存储 Source <String> 实例的引用是极为安全的——没有消费者-方法可以调用。但是 Java 并不知道这一点，并且仍然禁止这样操作：

// Java
void demo(Source<String> strs) {
  Source<Object> objects = strs; // ！！！在 Java 中不允许
  // ……
}

为了修正这一点，我们必须声明对象的类型为 Source<? extends Object>，这是毫无意义的，因为我们可以像以前一样在该对象上调用所有相同的方法，所以更复杂的类型并没有带来价值。但编译器并不知道。

在 Kotlin 中，有一种方法向编译器解释这种情况。这称为声明处型变：我们可以标注 Source 的类型参数 T 来确保它仅从 Source<T> 成员中返回（生产），并从不被消费。 为此，我们提供 out 修饰符：

interface Source<out T> {
    fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // 这个没问题，因为 T 是一个 out-参数
    // ……
}
。。String 也是 Any， 任何接受 Any的地方 必然也能接受String，所以 可以 赋值。

一般原则是：当一个类 C 的类型参数 T 被声明为 out 时，它就只能出现在 C 的成员的输出-位置，但回报是 C<Base> 可以安全地作为 C<Derived>的超类。

简而言之，他们说类 C 是在参数 T 上是协变的，或者说 T 是一个协变的类型参数。 你可以认为 C 是 T 的生产者，而不是 T 的消费者。

out修饰符称为型变注解，并且由于它在类型参数声明处提供，所以我们称之为声明处型变。 这与 Java 的使用处型变相反，其类型用途通配符使得类型协变。

另外除了 out，Kotlin 又补充了一个型变注释：in。它使得一个类型参数逆变：只可以被消费而不可以被生产。逆变类型的一个很好的例子是 Comparable：

interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 拥有类型 Double，它是 Number 的子类型
    // 因此，我们可以将 x 赋给类型为 Comparable <Double> 的变量
    val y: Comparable<Double> = x // OK！
}

我们相信 in 和 out 两词是自解释的（因为它们已经在 C# 中成功使用很长时间了）， 因此上面提到的助记符不是真正需要的，并且可以将其改写为更高的目标：

存在性（The Existential） 转换：消费者 in, 生产者 out! :-)

使用处型变：类型投影
将类型参数 T 声明为 out 非常方便，并且能避免使用处子类型化的麻烦，但是有些类实际上不能限制为只返回 T！ 一个很好的例子是 Array：
class Array<T>(val size: Int) {
    fun get(index: Int): T { …… }
    fun set(index: Int, value: T) { …… }
}

该类在 T 上既不能是协变的也不能是逆变的。这造成了一些不灵活性。考虑下述函数：
fun copy(from: Array<Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}

这个函数应该将项目从一个数组复制到另一个数组。让我们尝试在实践中应用它：
val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3) { "" }
copy(ints, any)
//   ^ 其类型为 Array<Int> 但此处期望 Array<Any>

这里我们遇到同样熟悉的问题：Array <T> 在 T 上是不型变的，因此 Array <Int> 和 Array <Any> 都不是另一个的子类型。为什么？ 再次重复，因为 copy 可能做坏事，也就是说，例如它可能尝试写一个 String 到 from， 并且如果我们实际上传递一个 Int 的数组，一段时间后将会抛出一个 ClassCastException 异常。

那么，我们唯一要确保的是 copy() 不会做任何坏事。我们想阻止它写到 from，我们可以：
fun copy(from: Array<out Any>, to: Array<Any>) { …… }

这里发生的事情称为类型投影：我们说from不仅仅是一个数组，而是一个受限制的（投影的）数组：我们只可以调用返回类型为类型参数 T 的方法，如上，这意味着我们只能调用 get()。这就是我们的使用处型变的用法，并且是对应于 Java 的 Array<? extends Object>、 但使用更简单些的方式。

。。一个是 声明类的in out, 这里是 声明形参 的 in out

你也可以使用 in 投影一个类型：
fun fill(dest: Array<in String>, value: String) { …… }

Array<in String> 对应于 Java 的 Array<? super String>，也就是说，你可以传递一个 CharSequence 数组或一个 Object 数组给 fill() 函数。

星投影
有时你想说，你对类型参数一无所知，但仍然希望以安全的方式使用它。 这里的安全方式是定义泛型类型的这种投影，该泛型类型的每个具体实例化将是该投影的子类型。

Kotlin 为此提供了所谓的星投影语法：

对于 Foo <out T : TUpper>，其中 T 是一个具有上界 TUpper 的协变类型参数，Foo <*> 等价于 Foo <out TUpper>。 这意味着当 T 未知时，你可以安全地从 Foo <*> 读取 TUpper 的值。

对于 Foo <in T>，其中 T 是一个逆变类型参数，Foo <*> 等价于 Foo <in Nothing>。 这意味着当 T 未知时，没有什么可以以安全的方式写入 Foo <*>。

对于 Foo <T : TUpper>，其中 T 是一个具有上界 TUpper 的不型变类型参数，Foo<*> 对于读取值时等价于 Foo<out TUpper> 而对于写值时等价于 Foo<in Nothing>。

如果泛型类型具有多个类型参数，则每个类型参数都可以单独投影。 例如，如果类型被声明为 interface Function <in T, out U>，我们可以想象以下星投影：

Function<*, String> 表示 Function<in Nothing, String>；
Function<Int, *> 表示 Function<Int, out Any?>；
Function<*, *> 表示 Function<in Nothing, out Any?>。
注意：星投影非常像 Java 的原始类型，但是安全。

泛型函数
不仅类可以有类型参数。函数也可以有。类型参数要放在函数名称之前：
fun <T> singletonList(item: T): List<T> {
    // ……
}
fun <T> T.basicToString(): String {  // 扩展函数
    // ……
}

要调用泛型函数，在调用处函数名之后指定类型参数即可：
val l = singletonList<Int>(1)
可以省略能够从上下文中推断出来的类型参数，所以以下示例同样适用：
val l = singletonList(1)

泛型约束
能够替换给定类型参数的所有可能类型的集合可以由泛型约束限制。

上界
最常见的约束类型是与 Java 的 extends 关键字对应的 上界：
fun <T : Comparable<T>> sort(list: List<T>) {  …… }

冒号之后指定的类型是上界：只有 Comparable<T> 的子类型可以替代 T。 例如：
sort(listOf(1, 2, 3)) // OK。Int 是 Comparable<Int> 的子类型

sort(listOf(HashMap<Int, String>())) // 错误：HashMap<Int, String> 不是 Comparable<HashMap<Int, String>> 的子类型

默认的上界（如果没有声明）是 Any?。在尖括号中只能指定一个上界。 如果同一类型参数需要多个上界，我们需要一个单独的 where-子句：
fun <T> copyWhenGreater(list: List<T>, threshold: T): List<String>
    where T : CharSequence,
          T : Comparable<T> {
    return list.filter { it > threshold }.map { it.toString() }
}

public static final <T extends CharSequence,  extends Comparable<? super T>> List<String> copyWhenGreater(@NotNull List<? extends T> list, @NotNull T threshold) {

zzzzz。。还能这样写。。。好像不能啊。。
    private class Asdf<T extends CharSequence, extends Comparable<? super T>> {

        public <T extends CharSequence, extends Comparable<? super T>> List<T> copyWhenGreater() {

            return null;
        }
    }

。。都说 , 后面需要一个标识符。放个T不行。。  Syntax error on token ",", Identifier expected after this token

所传递的类型必须同时满足 where 子句的所有条件。在上述示例中，类型 T 必须既实现了 CharSequence 也实现了 Comparable。

。。有泛型了，上面的 投影。。。

类型擦除

Kotlin 为泛型声明用法执行的类型安全检测仅在编译期进行。 运行时泛型类型的实例不保留关于其类型实参的任何信息。 其类型信息称为被擦除。例如，Foo<Bar> 与 Foo<Baz?> 的实例都会被擦除为 Foo<*>。

因此，并没有通用的方法在运行时检测一个泛型类型的实例是否通过指定类型参数所创建 ，并且编译器禁止这种 is 检测。

类型转换为带有具体类型参数的泛型类型，如 foo as List<String> 无法在运行时检测。 当高级程序逻辑隐含了类型转换的类型安全而无法直接通过编译器推断时， 可以使用这种非受检类型转换。编译器会对非受检类型转换发出警告，并且在运行时只对非泛型部分检测（相当于 foo as List<*>）。

泛型函数调用的类型参数也同样只在编译期检测。在函数体内部， 类型参数不能用于类型检测，并且类型转换为类型参数（foo as T）也是非受检的。然而， 内联函数的具体化的类型参数会由调用处内联函数体中的类型实参所代入，因此可以用于类型检测与转换， 与上述泛型类型的实例具有相同限制。

https://www.kotlincn.net/docs/reference/nested-classes.html

类可以嵌套在其他类中：
class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
    }
}
val demo = Outer.Nested().foo() // == 2

标记为 inner 的嵌套类能够访问其外部类的成员。内部类会带有一个对外部类的对象的引用：
class Outer {
    private val bar: Int = 1
    inner class Inner {
        fun foo() = bar
    }
}
val demo = Outer().Inner().foo() // == 1

使用对象表达式创建匿名内部类实例：
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { …… }
    override fun mouseEntered(e: MouseEvent) { …… }
})

对于 JVM 平台, 如果对象是函数式 Java 接口（即具有单个抽象方法的 Java 接口）的实例， 你可以使用带接口类型前缀的lambda表达式创建它：
val listener = ActionListener { println("clicked") }

枚举类的最基本的用法是实现类型安全的枚举：
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}
每个枚举常量都是一个对象。枚举常量用逗号分隔。

因为每一个枚举都是枚举类的实例，所以他们可以是这样初始化过的：
enum class Color(val rgb: Int) {
        RED(0xFF0000),
        GREEN(0x00FF00),
        BLUE(0x0000FF)
}

枚举常量还可以声明其带有相应方法以及覆盖了基类方法的匿名类。
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },
​
    TALKING {
        override fun signal() = WAITING
    };
​
    abstract fun signal(): ProtocolState
}
如果枚举类定义任何成员，那么使用 *分号 将成员定义中的枚举常量定义分隔开。

一个枚举类可以实现接口（但不能从类继承），可以为所有条目提供统一的接口成员实现，也可以在相应匿名类中为每个条目提供各自的实现。只需将接口添加到枚举类声明中即可，如下所示：

enum class IntArithmetics : BinaryOperator<Int>, IntBinaryOperator {
    PLUS {
        override fun apply(t: Int, u: Int): Int = t + u
    },
    TIMES {
        override fun apply(t: Int, u: Int): Int = t * u
    };
​
    override fun applyAsInt(t: Int, u: Int) = apply(t, u)
}

Kotlin 中的枚举类也有合成方法允许列出定义的枚举常量以及通过名称获取枚举常量。这些方法的签名如下（假设枚举类的名称是 EnumClass）：
EnumClass.valueOf(value: String): EnumClass
EnumClass.values(): Array<EnumClass>

如果指定的名称与类中定义的任何枚举常量均不匹配，valueOf() 方法将抛出 IllegalArgumentException 异常。

自 Kotlin 1.1 起，可以使用 enumValues<T>() 与 enumValueOf<T>() 函数以泛型的方式访问枚举类中的常量 ：
enum class RGB { RED, GREEN, BLUE }
inline fun <reified T : Enum<T>> printAllValues() {
    print(enumValues<T>().joinToString { it.name })
}
printAllValues<RGB>() // 输出 RED, GREEN, BLUE

每个枚举常量都具有在枚举类声明中获取其名称与位置的属性：
val name: String
val ordinal: Int
枚举常量还实现了 Comparable 接口， 其中自然顺序是它们在枚举类中定义的顺序。

https://www.kotlincn.net/docs/reference/object-declarations.html
对象表达式与对象声明

有时候，我们需要创建一个对某个类做了轻微改动的类的对象，而不用为之显式声明新的子类。 Kotlin 用对象表达式和对象声明处理这种情况。

要创建一个继承自某个（或某些）类型的匿名类的对象，我们会这么写：
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { /*……*/ }
    override fun mouseEntered(e: MouseEvent) { /*……*/ }
})

如果超类型有一个构造函数，则必须传递适当的构造函数参数给它。 多个超类型可以由跟在冒号后面的逗号分隔的列表指定：
open class A(x: Int) {
    public open val y: Int = x
}
interface B { /*……*/ }
val ab: A = object : A(1), B {
    override val y = 15
}

任何时候，如果我们只需要“一个对象而已”，并不需要特殊超类型，那么我们可以简单地写：
fun foo() {
    val adHoc = object {
        var x: Int = 0
        var y: Int = 0
    }
    print(adHoc.x + adHoc.y)
}

请注意，匿名对象可以用作只在本地和私有作用域中声明的类型。如果你使用匿名对象作为公有函数的返回类型或者用作公有属性的类型，那么该函数或属性的实际类型会是匿名对象声明的超类型，如果你没有声明任何超类型，就会是 Any。在匿名对象中添加的成员将无法访问。

class C {
    // 私有函数，所以其返回类型是匿名对象类型
    private fun foo() = object {
        val x: String = "x"
    }

    // 公有函数，所以其返回类型是 Any
    fun publicFoo() = object {
        val x: String = "x"
    }

    fun bar() {
        val x1 = foo().x        // 没问题
        val x2 = publicFoo().x  // 错误：未能解析的引用“x”
    }
}

对象表达式中的代码可以访问来自包含它的作用域的变量。
fun countClicks(window: JComponent) {
    var clickCount = 0
    var enterCount = 0
    window.addMouseListener(object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            clickCount++
        }
        override fun mouseEntered(e: MouseEvent) {
            enterCount++
        }
    })
    // ……
}

单例模式在一些场景中很有用， 而 Kotlin（继 Scala 之后）使单例声明变得很容易：
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ……
    }
    val allDataProviders: Collection<DataProvider>
        get() = // ……
}
这称为对象声明。并且它总是在 object 关键字后跟一个名称。 就像变量声明一样，对象声明不是一个表达式，不能用在赋值语句的右边。

对象声明的初始化过程是线程安全的并且在首次访问时进行。

如需引用该对象，我们直接使用其名称即可：
DataProviderManager.registerDataProvider(……)

这些对象可以有超类型：
object DefaultListener : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { …… }
    override fun mouseEntered(e: MouseEvent) { …… }
}

注意：对象声明不能在局部作用域（即直接嵌套在函数内部），但是它们可以嵌套到其他对象声明或非内部类中。

伴生对象
类内部的对象声明可以用 companion 关键字标记：
class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
}

该伴生对象的成员可通过只使用类名作为限定符来调用：
val instance = MyClass.create()

可以省略伴生对象的名称，在这种情况下将使用名称 Companion：
class MyClass {
    companion object { }
}
val x = MyClass.Companion

其自身所用的类的名称（不是另一个名称的限定符）可用作对该类的伴生对象 （无论是否具名）的引用：
class MyClass1 {
    companion object Named { }
}
val x = MyClass1
class MyClass2 {
    companion object { }
}
val y = MyClass2

请注意，即使伴生对象的成员看起来像其他语言的静态成员，在运行时他们仍然是真实对象的实例成员，而且，例如还可以实现接口：

interface Factory<T> {
    fun create(): T
}
class MyClass {
    companion object : Factory<MyClass> {
        override fun create(): MyClass = MyClass()
    }
}
val f: Factory<MyClass> = MyClass

。。3种，类名.伴生对象名，，类名.Companion,,,直接一个 类名。
。。如果有多个伴生对象，那就只能 第一种了吧。

当然，在 JVM 平台，如果使用 @JvmStatic 注解，你可以将伴生对象的成员生成为真正的静态方法和字段。更详细信息请参见Java 互操作性一节 。

对象表达式和对象声明之间的语义差异
对象表达式和对象声明之间有一个重要的语义差别：
对象表达式是在使用他们的地方立即执行（及初始化）的；
对象声明是在第一次被访问到时延迟初始化的；
伴生对象的初始化是在相应的类被加载（解析）时，与 Java 静态初始化器的语义相匹配。

https://www.kotlincn.net/docs/reference/type-aliases.html

类型别名为现有类型提供替代名称。 如果类型名称太长，你可以另外引入较短的名称，并使用新的名称替代原类型名。
它有助于缩短较长的泛型类型。 例如，通常缩减集合类型是很有吸引力的：
typealias NodeSet = Set<Network.Node>
typealias FileTable<K> = MutableMap<K, MutableList<File>>

你可以为函数类型提供另外的别名：
typealias MyHandler = (Int, String, Any) -> Unit
typealias Predicate<T> = (T) -> Boolean

你可以为内部类和嵌套类创建新名称：
class A {
    inner class Inner
}
class B {
    inner class Inner
}
typealias AInner = A.Inner
typealias BInner = B.Inner

类型别名不会引入新类型。 它们等效于相应的底层类型。 当你在代码中添加 typealias Predicate<T> 并使用 Predicate<Int> 时，Kotlin 编译器总是把它扩展为 (Int) -> Boolean。 因此，当你需要泛型函数类型时，你可以传递该类型的变量，反之亦然：

typealias Predicate<T> = (T) -> Boolean
​
fun foo(p: Predicate<Int>) = p(42)
​
fun main() {
    val f: (Int) -> Boolean = { it > 0 }
    println(foo(f)) // 输出 "true"
​
    val p: Predicate<Int> = { it > 0 }
    println(listOf(1, -2).filter(p)) // 输出 "[1]"
}

内联类仅在 Kotlin 1.3 之后版本可用

有时候，业务逻辑需要围绕某种类型创建包装器。然而，由于额外的堆内存分配问题，它会引入运行时的性能开销。此外，如果被包装的类型是原生类型，性能的损失是很糟糕的，因为原生类型通常在运行时就进行了大量优化，然而他们的包装器却没有得到任何特殊的处理。

为了解决这类问题，Kotlin 引入了一种被称为 内联类 的特殊类，它通过在类的前面定义一个 inline 修饰符来声明：
inline class Password(val value: String)

内联类必须含有唯一的一个属性在主构造函数中初始化。在运行时，将使用这个唯一属性来表示内联类的实例（关于运行时的内部表达请参阅下文）：
// 不存在 'Password' 类的真实实例对象
// 在运行时，'securePassword' 仅仅包含 'String'
val securePassword = Password("Don't try this in production")

内联类支持普通类中的一些功能。特别是，内联类可以声明属性与函数：
inline class Name(val s: String) {
    val length: Int
        get() = s.length
​    fun greet() {
        println("Hello, $s")
    }
}
​fun main() {
    val name = Name("Kotlin")
    name.greet() // `greet` 方法会作为一个静态方法被调用
    println(name.length) // 属性的 get 方法会作为一个静态方法被调用
}

然而，内联类的成员也有一些限制：
内联类不能含有 init 代码块
内联类不能含有幕后字段
因此，内联类只能含有简单的计算属性（不能含有延迟初始化/委托属性）

内联类允许去继承接口

interface Printable {
    fun prettyPrint(): String
}
inline class Name(val s: String) : Printable {
    override fun prettyPrint(): String = "Let's $s!"
}
fun main() {
    val name = Name("Kotlin")
    println(name.prettyPrint()) // 仍然会作为一个静态方法被调用
}

禁止内联类参与到类的继承关系结构中。这就意味着内联类不能继承其他的类而且必须是 final。

在生成的代码中，Kotlin 编译器为每个内联类保留一个包装器。内联类的实例可以在运行时表示为包装器或者基础类型。这就类似于 Int 可以表示为原生类型 int 或者包装器 Integer。

为了生成性能最优的代码，Kotlin 编译更倾向于使用基础类型而不是包装器。 然而，有时候使用包装器是必要的。一般来说，只要将内联类用作另一种类型，它们就会被装箱。

。。就等于 int 和 Integer，  内联类。    5既是一个int，又是一个Integer，优先int。

interface I
inline class Foo(val i: Int) : I
fun asInline(f: Foo) {}
fun <T> asGeneric(x: T) {}
fun asInterface(i: I) {}
fun asNullable(i: Foo?) {}
fun <T> id(x: T): T = x
fun main() {
    val f = Foo(42)
    asInline(f)    // 拆箱操作: 用作 Foo 本身
    asGeneric(f)   // 装箱操作: 用作泛型类型 T
    asInterface(f) // 装箱操作: 用作类型 I
    asNullable(f)  // 装箱操作: 用作不同于 Foo 的可空类型 Foo?
    // 在下面这里例子中，'f' 首先会被装箱（当它作为参数传递给 'id' 函数时）然后又被拆箱（当它从'id'函数中被返回时）
    // 最后， 'c' 中就包含了被拆箱后的内部表达(也就是 '42')， 和 'f' 一样
    val c = id(f)
}

因为内联类既可以表示为基础类型有可以表示为包装器，引用相等对于内联类而言毫无意义，因此这也是被禁止的。

名字修饰
由于内联类被编译为其基础类型，因此可能会导致各种模糊的错误，例如意想不到的平台签名冲突：
inline class UInt(val x: Int)
// 在 JVM 平台上被表示为'public final void compute(int x)'
fun compute(x: Int) { }
// 同理，在 JVM 平台上也被表示为'public final void compute(int x)'！
fun compute(x: UInt) { }

为了缓解这种问题，一般会通过在函数名后面拼接一些稳定的哈希码来重命名函数。 因此，fun compute(x: UInt) 将会被表示为 public final void compute-<hashcode>(int x)，以此来解决冲突的问题。

请注意在 Java 中 - 是一个 无效的 符号，也就是说在 Java 中不能调用使用内联类作为形参的函数。

。。kotlin的方法名可以compute，不加后缀hashCode吧？，这种能调用吗？感觉不能。。。还有，可以反射出来啊，既然-是无效的，那么说明kotlin生成的class文件中，不可能有-，应该有规律的，那就能获得这个方法了啊。

初看起来，内联类似乎与类型别名非常相似。实际上，两者似乎都引入了一种新的类型，并且都在运行时表示为基础类型。

然而，关键的区别在于类型别名与其基础类型(以及具有相同基础类型的其他类型别名)是 赋值兼容 的，而内联类却不是这样。

换句话说，内联类引入了一个真实的新类型，与类型别名正好相反，类型别名仅仅是为现有的类型取了个新的替代名称(别名)：

typealias NameTypeAlias = String
inline class NameInlineClass(val s: String)

fun acceptString(s: String) {}
fun acceptNameTypeAlias(n: NameTypeAlias) {}
fun acceptNameInlineClass(p: NameInlineClass) {}

fun main() {
    val nameAlias: NameTypeAlias = ""
    val nameInlineClass: NameInlineClass = NameInlineClass("")
    val string: String = ""

    acceptString(nameAlias) // 正确: 传递别名类型的实参替代函数中基础类型的形参
    acceptString(nameInlineClass) // 错误: 不能传递内联类的实参替代函数中基础类型的形参

    // And vice versa:
    acceptNameTypeAlias(string) // 正确: 传递基础类型的实参替代函数中别名类型的形参
    acceptNameInlineClass(string) // 错误: 不能传递基础类型的实参替代函数中内联类类型的形参
}

内联类的设计处于 Alpha 版，这意味着对于未来版本没有兼容性保证。在 Kotlin 1.3+ 中使用内联类时，会报一个警告，提示该特性尚未稳定发布。

如需移除警告，必须通过指定编译器参数 -Xinline-classes 来选择使用这项特性。

https://www.kotlincn.net/docs/reference/delegation.html

委托模式已经证明是实现继承的一个很好的替代方式， 而 Kotlin 可以零样板代码地原生支持它。 Derived 类可以通过将其所有公有成员都委托给指定对象来实现一个接口 Base：

interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main() {
    val b = BaseImpl(10)
    Derived(b).print()
}

Derived 的超类型列表中的 by-子句表示 b 将会在 Derived 中内部存储， 并且编译器将生成转发给 b 的所有 Base 的方法。

覆盖由委托实现的接口成员

覆盖符合预期：编译器会使用 override 覆盖的实现而不是委托对象中的。如果将 override fun printMessage() { print("abc") } 添加到 Derived，那么当调用 printMessage 时程序会输出“abc”而不是“10”：

interface Base {
    fun printMessage()
    fun printMessageLine()
}

class BaseImpl(val x: Int) : Base {
    override fun printMessage() { print(x) }
    override fun printMessageLine() { println(x) }
}

class Derived(b: Base) : Base by b {
    override fun printMessage() { print("abc") }
}

fun main() {
    val b = BaseImpl(10)
    Derived(b).printMessage()
    Derived(b).printMessageLine()
}

但请注意，以这种方式重写的成员不会在委托对象的成员中调用 ，委托对象的成员只能访问其自身对接口成员实现：

interface Base {
    val message: String
    fun print()
}
​
class BaseImpl(val x: Int) : Base {
    override val message = "BaseImpl: x = $x"
    override fun print() { println(message) }
}
​
class Derived(b: Base) : Base by b {
    // 在 b 的 `print` 实现中不会访问到这个属性
    override val message = "Message of Derived"
}
​
fun main() {
    val b = BaseImpl(10)
    val derived = Derived(b)
    derived.print()
    println(derived.message)
}

输出：
BaseImpl: x = 10
Message of Derived

。。可以覆盖方法，不能覆盖属性。

。。第一个输出，是derived.print,调用BaseImpl的print，输出message，虽然Derived重写了message，但是没有覆盖掉BaseImpl的，所以输出还是 BaseImpl中的message。

https://www.kotlincn.net/docs/reference/delegated-properties.html

有一些常见的属性类型，虽然我们可以在每次需要的时候手动实现它们， 但是如果能够为大家把他们只实现一次并放入一个库会更好。例如包括：

延迟属性（lazy properties）: 其值只在首次访问时计算；
可观察属性（observable properties）: 监听器会收到有关此属性变更的通知；
把多个属性储存在一个映射（map）中，而不是每个存在单独的字段中。
为了涵盖这些（以及其他）情况，Kotlin 支持 委托属性:
class Example {
    var p: String by Delegate()
}

为了涵盖这些（以及其他）情况，Kotlin 支持 委托属性:
class Example {
    var p: String by Delegate()
}

语法是： val/var <属性名>: <类型> by <表达式>。在 by 后面的表达式是该 委托， 因为属性对应的 get()（与 set()）会被委托给它的 getValue() 与 setValue() 方法。 属性的委托不必实现任何的接口，但是需要提供一个 getValue() 函数（与 setValue()——对于 var 属性）。 例如:

import kotlin.reflect.KProperty
​
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {

        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}

当我们从委托到一个 Delegate 实例的 p 读取时，将调用 Delegate 中的 getValue() 函数， 所以它第一个参数是读出 p 的对象、第二个参数保存了对 p 自身的描述 （例如你可以取它的名字)。 例如:

val e = Example()
println(e.p)

输出结果：
Example@33a17727, thank you for delegating ‘p’ to me!

类似地，当我们给 p 赋值时，将调用 setValue() 函数。前两个参数相同，第三个参数保存将要被赋予的值：
e.p = "NEW"

输出结果：
NEW has been assigned to ‘p’ in Example@33a17727.

标准委托
Kotlin 标准库为几种有用的委托提供了工厂方法。

延迟属性 Lazy

lazy() 是接受一个 lambda 并返回一个 Lazy <T> 实例的函数，返回的实例可以作为实现延迟属性的委托： 第一次调用 get() 会执行已传递给 lazy() 的 lambda 表达式并记录结果， 后续调用 get() 只是返回记录的结果。

val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}
fun main() {
    println(lazyValue)
    println(lazyValue)
}

默认情况下，对于 lazy 属性的求值是同步锁的（synchronized）：该值只在一个线程中计算，并且所有线程会看到相同的值。如果初始化委托的同步锁不是必需的，这样多个线程可以同时执行，那么将 LazyThreadSafetyMode.PUBLICATION 作为参数传递给 lazy() 函数。 而如果你确定初始化将总是发生在与属性使用位于相同的线程， 那么可以使用 LazyThreadSafetyMode.NONE 模式：它不会有任何线程安全的保证以及相关的开销。

可观察属性 Observable

Delegates.observable() 接受两个参数：初始值与修改时处理程序（handler）。 每当我们给属性赋值时会调用该处理程序（在赋值后执行）。它有三个参数：被赋值的属性、旧值与新值：

import kotlin.properties.Delegates
​
class User {
    var name: String by Delegates.observable("<no name>") {
        prop, old, new ->
        println("$old -> $new")
    }
}
​
fun main() {
    val user = User()
    user.name = "first"
    user.name = "second"
}

如果你想截获赋值并“否决”它们，那么使用 vetoable() 取代 observable()。 在属性被赋新值生效之前会调用传递给 vetoable 的处理程序。

委托给另一个属性

从 Kotlin 1.4 开始，一个属性可以把它的 getter 与 setter 委托给另一个属性。这种委托对于顶层和类的属性（成员和扩展）都可用。该委托属性可以为：

顶层属性
同一个类的成员或扩展属性
另一个类的成员或扩展属性
为将一个属性委托给另一个属性，应在委托名称中使用恰当的 :: 限定符，例如，this::delegate 或 MyClass::delegate。

class MyClass(var memberInt: Int, val anotherClassInstance: ClassWithDelegate) {

    var delegatedToMember: Int by this::memberInt
    var delegatedToTopLevel: Int by ::topLevelInt

    val delegatedToAnotherClass: Int by anotherClassInstance::anotherClassInt
}
var MyClass.extDelegated: Int by ::topLevelInt

这是很有用的，例如，当想要以一种向后兼容的方式重命名一个属性的时候：引入一个新的属性、 使用 @Deprecated 注解来注解旧的属性、并委托其实现。
class MyClass {
   var newName: Int = 0
   @Deprecated("Use 'newName' instead", ReplaceWith("newName"))
   var oldName: Int by this::newName
}
fun main() {
   val myClass = MyClass()
   // 通知：'oldName: Int' is deprecated.
   // Use 'newName' instead
   myClass.oldName = 42
   println(myClass.newName) // 42
}

将属性储存在映射中

一个常见的用例是在一个映射（map）里存储属性的值。 这经常出现在像解析 JSON 或者做其他“动态”事情的应用中。 在这种情况下，你可以使用映射实例自身作为委托来实现委托属性。

class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}
在这个例子中，构造函数接受一个映射参数：
val user = User(mapOf(
    "name" to "John Doe",
    "age"  to 25
))

委托属性会从这个映射中取值（通过字符串键——属性的名称）：
println(user.name) // Prints "John Doe"
println(user.age)  // Prints 25

这也适用于 var 属性，如果把只读的 Map 换成 MutableMap 的话：
class MutableUser(val map: MutableMap<String, Any?>) {
    var name: String by map
    var age: Int     by map
}

局部委托属性
你可以将局部变量声明为委托属性。 例如，你可以使一个局部变量惰性初始化：
fun example(computeFoo: () -> Foo) {
    val memoizedFoo by lazy(computeFoo)
​
    if (someCondition && memoizedFoo.isValid()) {
        memoizedFoo.doSomething()
    }
}
memoizedFoo 变量只会在第一次访问时计算。 如果 someCondition 失败，那么该变量根本不会计算。

https://www.kotlincn.net/docs/reference/delegated-properties.html
。。。gg。。
。。委托太繁了啊。。。

通过定义 provideDelegate 操作符，可以扩展创建属性实现所委托对象的逻辑。 如果 by 右侧所使用的对象将 provideDelegate 定义为成员或扩展函数， 那么会调用该函数来创建属性委托实例。

provideDelegate 的参数与 getValue 相同：

thisRef —— 必须与 属性所有者 类型（对于扩展属性——指被扩展的类型）相同或者是它的超类型；
property —— 必须是类型 KProperty<*> 或其超类型。

请注意，provideDelegate 方法只影响辅助属性的创建，并不会影响为 getter 或 setter 生成的代码。

https://www.kotlincn.net/docs/reference/functions.html

Kotlin 中的函数使用 fun 关键字声明：
fun double(x: Int): Int {
    return 2 * x
}

调用函数使用传统的方法：
val result = double(2)

调用成员函数使用点表示法：
Stream().read() // 创建类 Stream 实例并调用 read()

函数参数使用 Pascal 表示法定义，即 name: type。参数用逗号隔开。 每个参数必须有显式类型：
fun powerOf(number: Int, exponent: Int) { /*……*/ }

函数参数可以有默认值，当省略相应的参数时使用默认值。与其他语言相比，这可以减少重载数量：
fun read(
    b: Array<Byte>,
    off: Int = 0,
    len: Int = b.size,
) { /*……*/ }
声明后 加=默认值

。。还能使用形参的方法。。。就是 默认参数是 一个表达式。。为什么不是一个局部变量。。

覆盖方法总是使用与基类型方法相同的默认参数值。 当覆盖一个带有默认参数值的方法时，必须从签名中省略默认参数值：
open class A {
    open fun foo(i: Int = 10) { /*……*/ }
}
class B : A() {
    override fun foo(i: Int) { /*……*/ }  // 不能有默认值
}
。。默认值会传递给子类，且不能改。

如果一个默认参数在一个无默认值的参数之前，那么该默认值只能通过使用具名参数调用该函数来使用：
fun foo(
    bar: Int = 0,
    baz: Int,
) { /*……*/ }
​
foo(baz = 1) // 使用默认值 bar = 0

如果在默认参数之后的最后一个参数是 lambda 表达式，那么它既可以作为具名参数在括号内传入，也可以在括号外传入：
fun foo(
    bar: Int = 0,
    baz: Int = 1,
    qux: () -> Unit,
) { /*……*/ }
foo(1) { println("hello") }     // 使用默认值 baz = 1
foo(qux = { println("hello") }) // 使用两个默认值 bar = 0 与 baz = 1
foo { println("hello") }        // 使用两个默认值 bar = 0 与 baz = 1

对于 JVM 平台：在调用 Java 函数时不能使用具名参数语法，因为 Java 字节码并不总是保留函数参数的名称。

如果一个函数不返回任何有用的值，它的返回类型是 Unit。Unit 是一种只有一个值——Unit 的类型。这个值不需要显式返回：
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello $name")
    else
        println("Hi there!")
    // `return Unit` 或者 `return` 是可选的
}

Unit 返回类型声明也是可选的。上面的代码等同于：
fun printHello(name: String?) { …… }

当函数返回单个表达式时，可以省略花括号并且在 = 符号之后指定代码体即可：
fun double(x: Int): Int = x * 2

当返回值类型可由编译器推断时，显式声明返回类型是可选的：
fun double(x: Int) = x * 2

显式返回类型

具有块代码体的函数必须始终显式指定返回类型，除非他们旨在返回 Unit，在这种情况下它是可选的。 Kotlin 不推断具有块代码体的函数的返回类型，因为这样的函数在代码体中可能有复杂的控制流，并且返回类型对于读者（有时甚至对于编译器）是不明显的。

可变数量的参数（Varargs）
函数的参数（通常是最后一个）可以用 vararg 修饰符标记：
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}

允许将可变数量的参数传递给函数：
val list = asList(1, 2, 3)

在函数内部，类型 T 的 vararg 参数的可见方式是作为 T 数组，即上例中的 ts 变量具有类型 Array <out T>。

只有一个参数可以标注为 vararg。如果 vararg 参数不是列表中的最后一个参数， 可以使用具名参数语法传递其后的参数的值，或者，如果参数具有函数类型，则通过在括号外部传一个 lambda。

当我们调用 vararg-函数时，我们可以一个接一个地传参，例如 asList(1, 2, 3)，或者，如果我们已经有一个数组并希望将其内容传给该函数，我们使用伸展（spread）操作符（在数组前面加 *）：

val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)

。。感觉是 摸着c++ 写java。。。

中缀表示法
标有 infix 关键字的函数也可以使用中缀表示法（忽略该调用的点与圆括号）调用。中缀函数必须满足以下要求：
它们必须是成员函数或扩展函数；
它们必须只有一个参数；
其参数不得接受可变数量的参数且不能有默认值。

infix fun Int.shl(x: Int): Int { …… }
// 用中缀表示法调用该函数
1 shl 2
// 等同于这样
1.shl(2)

中缀函数调用的优先级低于算术操作符、类型转换以及 rangeTo 操作符。 以下表达式是等价的：
1 shl 2 + 3 等价于 1 shl (2 + 3)
0 until n * 2 等价于 0 until (n * 2)
xs union ys as Set<*> 等价于 xs union (ys as Set<*>)

另一方面，中缀函数调用的优先级高于布尔操作符 && 与 ||、is- 与 in- 检测以及其他一些操作符。这些表达式也是等价的：
a && b xor c 等价于 a && (b xor c)
a xor b in c 等价于 (a xor b) in c
完整的优先级层次结构请参见其语法参考。

请注意，中缀函数总是要求指定接收者与参数。当使用中缀表示法在当前接收者上调用方法时，需要显式使用 this；不能像常规方法调用那样省略。这是确保非模糊解析所必需的。

class MyStringCollection {
    infix fun add(s: String) { /*……*/ }

    fun build() {
        this add "abc"   // 正确
        add("abc")       // 正确
        //add "abc"        // 错误：必须指定接收者
    }
}

在 Kotlin 中函数可以在文件顶层声明，这意味着你不需要像一些语言如 Java、C# 或 Scala 那样需要创建一个类来保存一个函数。此外除了顶层函数，Kotlin 中函数也可以声明在局部作用域、作为成员函数以及扩展函数。

Kotlin 支持局部函数，即一个函数在另一个函数内部：
fun dfs(graph: Graph) {
    fun dfs(current: Vertex, visited: MutableSet<Vertex>) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v, visited)
    }
    dfs(graph.vertices[0], HashSet())
}

局部函数可以访问外部函数（即闭包）的局部变量，所以在上例中，visited 可以是局部变量：
fun dfs(graph: Graph) {
    val visited = HashSet<Vertex>()
    fun dfs(current: Vertex) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v)
    }
    dfs(graph.vertices[0])
}

成员函数是在类或对象内部定义的函数：
class Sample {
    fun foo() { print("Foo") }
}
成员函数以点表示法调用：
Sample().foo() // 创建类 Sample 实例并调用 foo

函数可以有泛型参数，通过在函数名前使用尖括号指定：
fun <T> singletonList(item: T): List<T> { /*……*/ }

内联函数

扩展函数

高阶函数和 Lambda 表达式

尾递归函数

Kotlin 支持一种称为尾递归的函数式编程风格。 这允许一些通常用循环写的算法改用递归函数来写，而无堆栈溢出的风险。 当一个函数用 tailrec 修饰符标记并满足所需的形式时，编译器会优化该递归，留下一个快速而高效的基于循环的版本：

val eps = 1E-10 // "good enough", could be 10^-15

tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (Math.abs(x - Math.cos(x)) < eps) x else findFixPoint(Math.cos(x))
。。会优化。

这段代码计算余弦的不动点（fixpoint of cosine），这是一个数学常数。 它只是重复地从 1.0 开始调用 Math.cos，直到结果不再改变，对于这里指定的 eps 精度会产生 0.7390851332151611 的结果。最终代码相当于这种更传统风格的代码：

val eps = 1E-10 // "good enough", could be 10^-15
private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (Math.abs(x - y) < eps) return x
        x = Math.cos(x)
    }
}

要符合 tailrec 修饰符的条件的话，函数必须将其自身调用作为它执行的最后一个操作。在递归调用后有更多代码时，不能使用尾递归，并且不能用在 try/catch/finally 块中。目前在 Kotlin for JVM 与 Kotlin/Native 中支持尾递归。

。。这，，，还分2种？jvm 和 native ？？？？

https://www.kotlincn.net/docs/reference/lambdas.html
高阶函数与 lambda 表达式

Kotlin 函数都是头等的，这意味着它们可以存储在变量与数据结构中、作为参数传递给其他高阶函数以及从其他高阶函数返回。可以像操作任何其他非函数值一样操作函数。

为促成这点，作为一门静态类型编程语言的 Kotlin 使用一系列函数类型来表示函数并提供一组特定的语言结构，例如 lambda 表达式。

高阶函数是将函数用作参数或返回值的函数。

一个不错的示例是集合的函数式风格的 fold， 它接受一个初始累积值与一个接合函数，并通过将当前累积值与每个集合元素连续接合起来代入累积值来构建返回值：
fun <T, R> Collection<T>.fold(
    initial: R,
    combine: (acc: R, nextElement: T) -> R
): R {
    var accumulator: R = initial
    for (element: T in this) {
        accumulator = combine(accumulator, element)
    }
    return accumulator
}

在上述代码中，参数 combine 具有函数类型 (R, T) -> R，因此 fold 接受一个函数作为参数， 该函数接受类型分别为 R 与 T 的两个参数并返回一个 R 类型的值。 在 for-循环内部调用该函数，然后将其返回值赋值给 accumulator。

为了调用 fold，需要传给它一个函数类型的实例作为参数，而在高阶函数调用处，（下文详述的）lambda 表达 式广泛用于此目的。

val items = listOf(1, 2, 3, 4, 5)
​
// Lambdas 表达式是花括号括起来的代码块。
items.fold(0, {
    // 如果一个 lambda 表达式有参数，前面是参数，后跟“->”
    acc: Int, i: Int ->
    print("acc = $acc, i = $i, ")
    val result = acc + i
    println("result = $result")
    // lambda 表达式中的最后一个表达式是返回值：
    result
})
​
// lambda 表达式的参数类型是可选的，如果能够推断出来的话：
val joinedToString = items.fold("Elements:", { acc, i -> acc + " " + i })
​
// 函数引用也可以用于高阶函数调用：
val product = items.fold(1, Int::times)
​

Kotlin 使用类似 (Int) -> String 的一系列函数类型来处理函数的声明： val onClick: () -> Unit = ……。
这些类型具有与函数签名相对应的特殊表示法，即它们的参数和返回值：

所有函数类型都有一个圆括号括起来的参数类型列表以及一个返回类型：(A, B) -> C 表示接受类型分别为 A 与 B 两个参数并返回一个 C 类型值的函数类型。 参数类型列表可以为空，如 () -> A。Unit 返回类型不可省略。

函数类型可以有一个额外的接收者类型，它在表示法中的点之前指定： 类型 A.(B) -> C 表示可以在 A 的接收者对象上以一个 B 类型参数来调用并返回一个 C 类型值的函数。 带有接收者的函数字面值通常与这些类型一起使用。

挂起函数属于特殊种类的函数类型，它的表示法中有一个 suspend 修饰符 ，例如 suspend () -> Unit 或者 suspend A.(B) -> C。

函数类型表示法可以选择性地包含函数的参数名：(x: Int, y: Int) -> Point。 这些名称可用于表明参数的含义。
如需将函数类型指定为可空，请使用圆括号：((Int, Int) -> Int)?。
函数类型可以使用圆括号进行接合：(Int) -> ((Int) -> Unit)
箭头表示法是右结合的，(Int) -> (Int) -> Unit 与前述示例等价，但不等于 ((Int) -> (Int)) -> Unit。

还可以通过使用类型别名给函数类型起一个别称：
typealias ClickHandler = (Button, ClickEvent) -> Unit

函数类型实例化
有几种方法可以获得函数类型的实例：
使用函数字面值的代码块，采用以下形式之一：
lambda 表达式: { a, b -> a + b },
匿名函数: fun(s: String): Int { return s.toIntOrNull() ?: 0 }
    带有接收者的函数字面值可用作带有接收者的函数类型的值。
使用已有声明的可调用引用：
顶层、局部、成员、扩展函数：::isOdd、 String::toInt，
顶层、成员、扩展属性：List<Int>::size，
构造函数：::Regex
这包括指向特定实例成员的绑定的可调用引用：foo::toString。
使用实现函数类型接口的自定义类的实例：
class IntTransformer: (Int) -> Int {
    override operator fun invoke(x: Int): Int = TODO()
}
val intFunction: (Int) -> Int = IntTransformer()

如果有足够信息，编译器可以推断变量的函数类型：
val a = { i: Int -> i + 1 } // 推断出的类型是 (Int) -> Int

带与不带接收者的函数类型非字面值可以互换，其中接收者可以替代第一个参数，反之亦然。例如，(A, B) -> C 类型的值可以传给或赋值给期待 A.(B) -> C 的地方，反之亦然：

val repeatFun: String.(Int) -> String = { times -> this.repeat(times) }
val twoParameters: (String, Int) -> String = repeatFun // OK
fun runTransformation(f: (String, Int) -> String): String {
    return f("hello", 3)
}
val result = runTransformation(repeatFun) // OK
​
请注意，默认情况下推断出的是没有接收者的函数类型，即使变量是通过扩展函数引用来初始化的。 如需改变这点，请显式指定变量类型。

函数类型实例调用
函数类型的值可以通过其 invoke(……) 操作符调用：f.invoke(x) 或者直接 f(x)。

如果该值具有接收者类型，那么应该将接收者对象作为第一个参数传递。 调用带有接收者的函数类型值的另一个方式是在其前面加上接收者对象， 就好比该值是一个扩展函数：1.foo(2)，

例如：
val stringPlus: (String, String) -> String = String::plus
val intPlus: Int.(Int) -> Int = Int::plus
println(stringPlus.invoke("<-", "->"))
println(stringPlus("Hello, ", "world!"))
println(intPlus.invoke(1, 1))
println(intPlus(1, 2))
println(2.intPlus(3)) // 类扩展调用
​

lambda 表达式与匿名函数是“函数字面值”，即未声明的函数， 但立即做为表达式传递。考虑下面的例子：
max(strings, { a, b -> a.length < b.length })

函数 max 是一个高阶函数，它接受一个函数作为第二个参数。 其第二个参数是一个表达式，它本身是一个函数，即函数字面值，它等价于以下具名函数：
fun compare(a: String, b: String): Boolean = a.length < b.length

Lambda 表达式的完整语法形式如下：
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }

lambda 表达式总是括在花括号中， 完整语法形式的参数声明放在花括号内，并有可选的类型标注， 函数体跟在一个 -> 符号之后。如果推断出的该 lambda 的返回类型不是 Unit，那么该 lambda 主体中的最后一个（或可能是单个） 表达式会视为返回值。

如果我们把所有可选标注都留下，看起来如下：
val sum = { x: Int, y: Int -> x + y }

在 Kotlin 中有一个约定：如果函数的最后一个参数是函数，那么作为相应参数传入的 lambda 表达式可以放在圆括号之外：
val product = items.fold(1) { acc, e -> acc * e }

这种语法也称为拖尾 lambda 表达式。
如果该 lambda 表达式是调用时唯一的参数，那么圆括号可以完全省略：
run { println("...") }

it：单个参数的隐式名称
一个 lambda 表达式只有一个参数是很常见的。
如果编译器自己可以识别出签名，也可以不用声明唯一的参数并忽略 ->。 该参数会隐式声明为 it：
ints.filter { it > 0 } // 这个字面值是“(it: Int) -> Boolean”类型的

我们可以使用限定的返回语法从 lambda 显式返回一个值。 否则，将隐式返回最后一个表达式的值。

因此，以下两个片段是等价的：
ints.filter {
    val shouldFilter = it > 0
    shouldFilter
}
ints.filter {
    val shouldFilter = it > 0
    return@filter shouldFilter
}

这一约定连同在圆括号外传递 lambda 表达式一起支持 LINQ-风格 的代码：
strings.filter { it.length == 5 }.sortedBy { it }.map { it.toUpperCase() }

下划线用于未使用的变量（自 1.1 起）
如果 lambda 表达式的参数未使用，那么可以用下划线取代其名称：
map.forEach { _, value -> println("$value!") }

匿名函数

上面提供的 lambda 表达式语法缺少的一个东西是指定函数的返回类型的能力。在大多数情况下，这是不必要的。因为返回类型可以自动推断出来。然而，如果确实需要显式指定，可以使用另一种语法： 匿名函数 。

fun(x: Int, y: Int): Int = x + y
匿名函数看起来非常像一个常规函数声明，除了其名称省略了。其函数体可以是表达式（如上所示）或代码块：
fun(x: Int, y: Int): Int {
    return x + y
}

参数和返回类型的指定方式与常规函数相同，除了能够从上下文推断出的参数类型可以省略：
ints.filter(fun(item) = item > 0)

匿名函数的返回类型推断机制与正常函数一样：对于具有表达式函数体的匿名函数将自动推断返回类型，而具有代码块函数体的返回类型必须显式指定（或者已假定为 Unit）。

请注意，匿名函数参数总是在括号内传递。 允许将函数留在圆括号外的简写语法仅适用于 lambda 表达式。

Lambda表达式与匿名函数之间的另一个区别是非局部返回的行为。一个不带标签的 return 语句总是在用 fun 关键字声明的函数中返回。这意味着 lambda 表达式中的 return 将从包含它的函数返回，而匿名函数中的 return 将从匿名函数自身返回。

Lambda 表达式或者匿名函数（以及局部函数和对象表达式） 可以访问其 闭包 ，即在外部作用域中声明的变量。 在 lambda 表达式中可以修改闭包中捕获的变量：

var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum)

带有接收者的函数字面值
带有接收者的函数类型，例如 A.(B) -> C，可以用特殊形式的函数字面值实例化—— 带有接收者的函数字面值。

如上所述，Kotlin 提供了调用带有接收者（提供接收者对象）的函数类型实例的能力。

在这样的函数字面值内部，传给调用的接收者对象成为隐式的this，以便访问接收者对象的成员而无需任何额外的限定符，亦可使用 this 表达式 访问接收者对象。

这种行为与扩展函数类似，扩展函数也允许在函数体内部访问接收者对象的成员。

这里有一个带有接收者的函数字面值及其类型的示例，其中在接收者对象上调用了 plus ：
val sum: Int.(Int) -> Int = { other -> plus(other) }

匿名函数语法允许你直接指定函数字面值的接收者类型。 如果你需要使用带接收者的函数类型声明一个变量，并在之后使用它，这将非常有用。
val sum = fun Int.(other: Int): Int = this + other

当接收者类型可以从上下文推断时，lambda 表达式可以用作带接收者的函数字面值。 One of the most important examples of their usage is type-safe builders:

class HTML {
    fun body() { …… }
}
fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()  // 创建接收者对象
    html.init()        // 将该接收者对象传给该 lambda
    return html
}
html {       // 带接收者的 lambda 由此开始
    body()   // 调用该接收者对象的一个方法
}

https://www.kotlincn.net/docs/reference/inline-functions.html

使用高阶函数会带来一些运行时的效率损失：每一个函数都是一个对象，并且会捕获一个闭包。 即那些在函数体内会访问到的变量。 内存分配（对于函数对象和类）和虚拟调用会引入运行时间开销。

但是在许多情况下通过内联化 lambda 表达式可以消除这类的开销。 下述函数是这种情况的很好的例子。即 lock() 函数可以很容易地在调用处内联。 考虑下面的情况：

lock(l) { foo() }

编译器没有为参数创建一个函数对象并生成一个调用。取而代之，编译器可以生成以下代码：
l.lock()
try {
    foo()
}
finally {
    l.unlock()
}
。。估计lock 是 一个api吧。应该是的，就是ReentrantLock 这种。

为了让编译器这么做，我们需要使用 inline 修饰符标记 lock() 函数：
inline fun <T> lock(lock: Lock, body: () -> T): T { …… }

inline 修饰符影响函数本身和传给它的 lambda 表达式：所有这些都将内联到调用处。
。。就是生成的代码变成普通的代码啊。并不是 lambda，那种 运行时 生产函数对象和类。

内联可能导致生成的代码增加；不过如果我们使用得当（即避免内联过大函数），性能上会有所提升，尤其是在循环中的“超多态（megamorphic）”调用处。

如果希望只内联一部分传给内联函数的 lambda 表达式参数，那么可以用 noinline 修饰符标记不希望内联的函数参数：

inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) { …… }

在 Kotlin 中，我们只能对具名或匿名函数使用正常的、非限定的 return 来退出。 这意味着要退出一个 lambda 表达式，我们必须使用一个标签，并且在 lambda 表达式内部禁止使用裸 return，因为 lambda 表达式不能使包含它的函数返回：

fun foo() {
    ordinaryFunction {
        return // 错误：不能使 `foo` 在此处返回
    }
}

但是如果 lambda 表达式传给的函数是内联的，该 return 也可以内联，所以它是允许的：
fun foo() {
    inlined {
        return // OK：该 lambda 表达式是内联的
    }
}

这种返回（位于 lambda 表达式中，但退出包含它的函数）称为非局部返回。 我们习惯了在循环中用这种结构，其内联函数通常包含：
fun hasZeros(ints: List<Int>): Boolean {
    ints.forEach {
        if (it == 0) return true // 从 hasZeros 返回
    }
    return false
}

请注意，一些内联函数可能调用传给它们的不是直接来自函数体、而是来自另一个执行上下文的 lambda 表达式参数，例如来自局部对象或嵌套函数。在这种情况下，该 lambda 表达式中也不允许非局部控制流。为了标识这种情况，该 lambda 表达式参数需要用 crossinline 修饰符标记:

inline fun f(crossinline body: () -> Unit) {
    val f = object: Runnable {
        override fun run() = body()
    }
    // ……
}

break 和 continue 在内联的 lambda 表达式中还不可用，但我们也计划支持它们。

具体化的类型参数
有时候我们需要访问一个作为参数传给我们的一个类型：
fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
    var p = parent
    while (p != null && !clazz.isInstance(p)) {
        p = p.parent
    }
    @Suppress("UNCHECKED_CAST")
    return p as T?
}

在这里我们向上遍历一棵树并且检测每个节点是不是特定的类型。 这都没有问题，但是调用处不是很优雅：
treeNode.findParentOfType(MyTreeNode::class.java)

我们真正想要的只是传一个类型给该函数，即像这样调用它：
treeNode.findParentOfType<MyTreeNode>()

为能够这么做，内联函数支持具体化的类型参数，于是我们可以这样写：
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}

我们使用 reified 修饰符来限定类型参数，现在可以在函数内部访问它了， 几乎就像是一个普通的类一样。由于函数是内联的，不需要反射，正常的操作符如 !is 和 as 现在都能用了。此外，我们还可以按照上面提到的方式调用它：myTree.findParentOfType<MyTreeNodeType>()。

虽然在许多情况下可能不需要反射，但我们仍然可以对一个具体化的类型参数使用它：

inline fun <reified T> membersOf() = T::class.members

fun main(s: Array<String>) {
    println(membersOf<StringBuilder>().joinToString("\n"))
}

普通的函数（未标记为内联函数的）不能有具体化参数。 不具有运行时表示的类型（例如非具体化的类型参数或者类似于Nothing的虚构类型） 不能用作具体化的类型参数的实参。

inline 修饰符可用于没有幕后字段的属性的访问器。 你可以标注独立的属性访问器：
val foo: Foo
    inline get() = Foo()

var bar: Bar
    get() = ……
    inline set(v) { …… }

你也可以标注整个属性，将它的两个访问器都标记为内联：
inline var bar: Bar
    get() = ……
    set(v) { …… }

当一个内联函数是 public 或 protected 而不是 private 或 internal 声明的一部分时，就会认为它是一个模块级的公有 API。可以在其他模块中调用它，并且也可以在调用处内联这样的调用。

这带来了一些由模块做这样变更时导致的二进制兼容的风险——声明一个内联函数但调用它的模块在它修改后并没有重新编译。

为了消除这种由非公有 API 变更引入的不兼容的风险，公有 API 内联函数体内不允许使用非公有声明，即，不允许使用 private 与 internal 声明以及其部件。

一个 internal 声明可以由 @PublishedApi 标注，这会允许它在公有 API 内联函数中使用。当一个 internal 内联函数标记有 @PublishedApi 时，也会像公有函数一样检测其函数体。

https://www.kotlincn.net/docs/reference/collections-overview.html

List 是一个有序集合，可通过索引（反映元素位置的整数）访问元素。元素可以在 list 中出现多次。列表的一个示例是一句话：有一组字、这些字的顺序很重要并且字可以重复。

Set 是唯一元素的集合。它反映了集合（set）的数学抽象：一组无重复的对象。一般来说 set 中元素的顺序并不重要。例如，字母表是字母的集合（set）。

Map（或者字典）是一组键值对。键是唯一的，每个键都刚好映射到一个值。值可以重复。map 对于存储对象之间的逻辑连接非常有用，例如，员工的 ID 与员工的位置。

只读集合类型是型变的。 这意味着，如果类 Rectangle 继承自 Shape，则可以在需要 List <Shape> 的任何地方使用 List <Rectangle>。 换句话说，集合类型与元素类型具有相同的子类型关系。 map 在值（value）类型上是型变的，但在键（key）类型上不是。

List默认使用ArrayList
Set使用 LinkedHashSet
Map使用 LinkedHashMap

创建集合的最常用方法是使用标准库函数
listOf<T>()、
setOf<T>()、
mutableListOf<T>()、
mutableSetOf<T>()。
如果以逗号分隔的集合元素列表作为参数，编译器会自动检测元素类型。创建空集合时，须明确指定类型。
val numbersSet = setOf("one", "two", "three", "four")
val emptySet = mutableSetOf<String>()

Map 也有这样的函数
mapOf() 与 mutableMapOf()。
映射的键和值作为 Pair 对象传递（通常使用中缀函数 to 创建）。
val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 1)

to 符号创建了一个短时存活的 Pair 对象，因此建议仅在性能不重要时才使用它。 为避免过多的内存使用，请使用其他方法。例如，可以创建可写 Map 并使用写入操作填充它。 apply() 函数可以帮助保持初始化流畅。

val numbersMap = mutableMapOf<String, String>().apply { this["one"] = "1"; this["two"] = "2" }

还有用于创建没有任何元素的集合的函数：emptyList()、emptySet() 与 emptyMap()。 创建空集合时，应指定集合将包含的元素类型。
val empty = emptyList<String>()

对于 List，有一个接受 List 的大小与初始化函数的构造函数，该初始化函数根据索引定义元素的值。
val doubled = List(3, { it * 2 })  // 如果你想操作这个集合，应使用 MutableList
println(doubled)

要创建具体类型的集合，例如 ArrayList 或 LinkedList，可以使用这些类型的构造函数。 类似的构造函数对于 Set 与 Map 的各实现中均有提供。

val linkedList = LinkedList<String>(listOf("one", "two", "three"))
val presizedSet = HashSet<Int>(32)

标准库中的集合复制操作创建了具有相同元素引用的 *浅* 复制集合。 因此，对集合元素所做的更改会反映在其所有副本中。

在特定时刻通过集合复制函数，例如toList()、toMutableList()、toSet() 等等。创建了集合的快照。 结果是创建了一个具有相同元素的新集合 如果在源集合中添加或删除元素，则 不会 影响 副本。副本也可以独立于源集合进行更改。

。。集合不同，但是里面的元素 是相同的。

这些函数还可用于将集合转换为其他类型，例如根据 List 构建 Set，反之亦然。
val sourceList = mutableListOf(1, 2, 3)
val copySet = sourceList.toMutableSet()
copySet.add(3)
copySet.add(4)
println(copySet)

集合的初始化可用于限制其可变性。例如，如果构建了一个 MutableList 的 List 引用，当你试图通过此引用修改集合的时候，编译器会抛出错误。
val sourceList = mutableListOf(1, 2, 3)
val referenceList: List<Int> = sourceList
//referenceList.add(4)            // 编译错误
sourceList.add(4)
println(referenceList) // 显示 sourceList 当前状态

。。referenceList 是 List，没有add方法。。。 自动推导推出的是 MutableList，是可以修改的。

可以通过其他集合各种操作的结果来创建集合。例如，过滤列表会创建与过滤器匹配的新元素列表：
val numbers = listOf("one", "two", "three", "four")
val longerThan3 = numbers.filter { it.length > 3 }
println(longerThan3)

映射生成转换结果列表：
val numbers = setOf(1, 2, 3)
println(numbers.map { it * 3 })
println(numbers.mapIndexed { idx, value -> value * idx })

关联生成 Map:
val numbers = listOf("one", "two", "three", "four")
println(numbers.associateWith { it.length })

https://www.kotlincn.net/docs/reference/iterators.html

Iterable<T> 接口的继承者（包括 Set 与 List）可以通过调用 iterator() 函数获得迭代器。 一旦获得迭代器它就指向集合的第一个元素；调用 next() 函数将返回此元素，并将迭代器指向下一个元素（如果下一个元素存在）。 一旦迭代器通过了最后一个元素，它就不能再用于检索元素；也无法重新指向到以前的任何位置。要再次遍历集合，请创建一个新的迭代器。

val numbers = listOf("one", "two", "three", "four")
val numbersIterator = numbers.iterator()
while (numbersIterator.hasNext()) {
    println(numbersIterator.next())
}

遍历 Iterable 集合的另一种方法是众所周知的 for 循环。在集合中使用 for 循环时，将隐式获取迭代器。因此，以下代码与上面的示例等效：
val numbers = listOf("one", "two", "three", "four")
for (item in numbers) {
    println(item)
}

最后，有一个好用的 forEach() 函数，可自动迭代集合并为每个元素执行给定的代码。因此，等效的示例如下所示：
val numbers = listOf("one", "two", "three", "four")
numbers.forEach {
    println(it)
}

对于列表，有一个特殊的迭代器实现： ListIterator 它支持列表双向迭代：正向与反向。 反向迭代由 hasPrevious() 和 previous() 函数实现。 此外， ListIterator 通过 nextIndex() 与 previousIndex() 函数提供有关元素索引的信息。

val numbers = listOf("one", "two", "three", "four")
val listIterator = numbers.listIterator()
while (listIterator.hasNext()) listIterator.next()
println("Iterating backwards:")
while (listIterator.hasPrevious()) {
    print("Index: ${listIterator.previousIndex()}")
    println(", value: ${listIterator.previous()}")
}

具有双向迭代的能力意味着 ListIterator 在到达最后一个元素后仍可以使用。

为了迭代可变集合，于是有了 MutableIterator 来扩展 Iterator 使其具有元素删除函数 remove() 。因此，可以在迭代时从集合中删除元素。

val numbers = mutableListOf("one", "two", "three", "four")
val mutableIterator = numbers.iterator()
mutableIterator.next()
mutableIterator.remove()
println("After removal: $numbers")

除了删除元素， MutableListIterator 还可以在迭代列表时插入和替换元素。
val numbers = mutableListOf("one", "four", "four")
val mutableListIterator = numbers.listIterator()
mutableListIterator.next()
mutableListIterator.add("two")
mutableListIterator.next()
mutableListIterator.set("three")
println(numbers)

Kotlin 可通过调用 kotlin.ranges 包中的 rangeTo() 函数及其操作符形式的 .. 轻松地创建两个值的区间。 通常，rangeTo() 会辅以 in 或 !in 函数。

if (i in 1..4) {  // 等同于 1 <= i && i <= 4
    print(i)
}

整数类型区间（IntRange、LongRange、CharRange）还有一个拓展特性：可以对其进行迭代。 这些区间也是相应整数类型的等差数列。 这种区间通常用于 for 循环中的迭代。

for (i in 1..4) print(i)

要反向迭代数字，请使用 downTo 函数而不是 .. 。
for (i in 4 downTo 1) print(i)

也可以通过任意步长（不一定为 1 ）迭代数字。 这是通过 step 函数完成的。
for (i in 1..8 step 2) print(i)
println()
for (i in 8 downTo 1 step 2) print(i)

要迭代不包含其结束元素的数字区间，请使用 until 函数：
for (i in 1 until 10) {       // i in [1, 10), 10被排除
    print(i)
}

区间从数学意义上定义了一个封闭的间隔：它由两个端点值定义，这两个端点值都包含在该区间内。 区间是为可比较类型定义的：具有顺序，可以定义任意实例是否在两个给定实例之间的区间内。 区间的主要操作是 contains，通常以 in 与 !in 操作符的形式使用。

要为类创建一个区间，请在区间起始值上调用 rangeTo() 函数，并提供结束值作为参数。 rangeTo() 通常以操作符 .. 形式调用。
val versionRange = Version(1, 11)..Version(1, 30)
println(Version(0, 9) in versionRange)
println(Version(1, 20) in versionRange)

。。Version  是 kt提供的？ 还是 可以自己写的。。 怎么比较的？ 类需要 实现 comparable？

整数类型的区间（例如 Int、Long 与 Char）可视为等差数列。 在 Kotlin 中，这些数列由特殊类型定义：IntProgression、LongProgression 与 CharProgression。

数列具有三个基本属性：first 元素、last 元素和一个非零的 step。 首个元素为 first，后续元素是前一个元素加上一个 step。 以确定的步长在数列上进行迭代等效于 Java/JavaScript 中基于索引的 for 循环。

for (int i = first; i <= last; i += step) {
  // ……
}

数列的 last 元素是这样计算的：
    对于正步长：不大于结束值且满足 (last - first) % step == 0 的最大值。
    对于负步长：不小于结束值且满足 (last - first) % step == 0 的最小值。

因此，last 元素并非总与指定的结束值相同。
for (i in 1..9 step 3) print(i) // 最后一个元素是 7

要创建反向迭代的数列，请在定义其区间时使用 downTo 而不是 ..。
for (i in 4 downTo 1) print(i)

数列实现 Iterable<N>，其中 N 分别是 Int、Long 或 Char，因此可以在各种集合函数（如 map、filter 与其他）中使用它们。
println((1..10).filter { it % 2 == 0 })

https://www.kotlincn.net/docs/reference/sequences.html

除了集合之外，Kotlin 标准库还包含另一种容器类型——序列（Sequence<T>）。 序列提供与 Iterable 相同的函数，但实现另一种方法来进行多步骤集合处理。

当 Iterable 的处理包含多个步骤时，它们会优先执行：每个处理步骤完成并返回其结果——中间集合。 在此集合上执行以下步骤。反过来，序列的多步处理在可能的情况下会延迟执行：仅当请求整个处理链的结果时才进行实际计算。

操作执行的顺序也不同：Sequence 对每个元素逐个执行所有处理步骤。 反过来，Iterable 完成整个集合的每个步骤，然后进行下一步。

因此，这些序列可避免生成中间步骤的结果，从而提高了整个集合处理链的性能。 但是，序列的延迟性质增加了一些开销，这些开销在处理较小的集合或进行更简单的计算时可能很重要。 因此，应该同时考虑使用 Sequence 与 Iterable，并确定在哪种情况更适合。

要创建一个序列，请调用 sequenceOf() 函数，列出元素作为其参数。
val numbersSequence = sequenceOf("four", "three", "two", "one")

如果已经有一个 Iterable 对象（例如 List 或 Set），则可以通过调用 asSequence() 从而创建一个序列。
val numbers = listOf("one", "two", "three", "four")
val numbersSequence = numbers.asSequence()

创建序列的另一种方法是通过使用计算其元素的函数来构建序列。 要基于函数构建序列，请以该函数作为参数调用 generateSequence()。 （可选）可以将第一个元素指定为显式值或函数调用的结果。 当提供的函数返回 null 时，序列生成停止。因此，以下示例中的序列是无限的。

val oddNumbers = generateSequence(1) { it + 2 } // `it` 是上一个元素
println(oddNumbers.take(5).toList())
//println(oddNumbers.count())     // 错误：此序列是无限的。

要使用 generateSequence() 创建有限序列，请提供一个函数，该函数在需要的最后一个元素之后返回 null。

val oddNumbersLessThan10 = generateSequence(1) { if (it < 10) it + 2 else null }

println(oddNumbersLessThan10.count())

最后，有一个函数可以逐个或按任意大小的组块生成序列元素——sequence() 函数。 此函数采用一个 lambda 表达式，其中包含 yield() 与 yieldAll() 函数的调用。 它们将一个元素返回给序列使用者，并暂停 sequence() 的执行，直到使用者请求下一个元素。 yield() 使用单个元素作为参数；yieldAll() 中可以采用 Iterable 对象、Iterable 或其他 Sequence。yieldAll() 的 Sequence 参数可以是无限的。 当然，这样的调用必须是最后一个：之后的所有调用都永远不会执行。

val oddNumbers = sequence {
    yield(1)
    yieldAll(listOf(3, 5))
    yieldAll(generateSequence(7) { it + 2 })
}
println(oddNumbers.take(5).toList())

关于序列操作，根据其状态要求可以分为以下几类：

    无状态 操作不需要状态，并且可以独立处理每个元素，例如 map() 或 filter()。 无状态操作还可能需要少量常数个状态来处理元素，例如 take() 与 drop()。

    有状态 操作需要大量状态，通常与序列中元素的数量成比例。

如果序列操作返回延迟生成的另一个序列，则称为 中间序列。 否则，该操作为 末端 操作。 末端操作的示例为 toList() 或 sum()。只能通过末端操作才能检索序列元素。

序列可以多次迭代；但是，某些序列实现可能会约束自己仅迭代一次。其文档中特别提到了这一点。

我们通过一个示例来看 Iterable 与 Sequence 之间的区别。
https://www.kotlincn.net/docs/reference/sequences.html

val words = "The quick brown fox jumps over the lazy dog".split(" ")
val lengthsList = words.filter { println("filter: $it"); it.length > 3 }
    .map { println("length: ${it.length}"); it.length }
    .take(4)

println("Lengths of first 4 words longer than 3 chars:")
println(lengthsList)

。。有图的，iterator是，filter完，然后map完，然后take。

val words = "The quick brown fox jumps over the lazy dog".split(" ")
// 将列表转换为序列
val wordsSequence = words.asSequence()

val lengthsSequence = wordsSequence.filter { println("filter: $it"); it.length > 3 }

    .map { println("length: ${it.length}"); it.length }
    .take(4)

println("Lengths of first 4 words longer than 3 chars")
// 末端操作：以列表形式获取结果。
println(lengthsSequence.toList())

。。sequence是， 第一个单词 filter，map，take 完， 然后第二个单词 filter map take， 然后第三个单词。

集合操作在标准库中以两种方式声明：集合接口的成员函数和扩展函数。

成员函数定义了对于集合类型是必不可少的操作。例如，Collection 包含函数 isEmpty() 来检查其是否为空； List 包含用于对元素进行索引访问的 get()，等等。

创建自己的集合接口实现时，必须实现其成员函数。 为了使新实现的创建更加容易，请使用标准库中集合接口的框架实现：AbstractCollection、AbstractList、AbstractSet、AbstractMap 及其相应可变抽象类。

其他集合操作被声明为扩展函数。这些是过滤、转换、排序和其他集合处理功能。

Kotlin 标准库为集合 转换 提供了一组扩展函数。 这些函数根据提供的转换规则从现有集合中构建新集合。 在此页面中，我们将概述可用的集合转换函数。

映射 转换从另一个集合的元素上的函数结果创建一个集合。 基本的映射函数是 map()。 它将给定的 lambda 函数应用于每个后续元素，并返回 lambda 结果列表。 结果的顺序与元素的原始顺序相同。 如需应用还要用到元素索引作为参数的转换，请使用 mapIndexed()。

val numbers = setOf(1, 2, 3)
println(numbers.map { it * 3 })
println(numbers.mapIndexed { idx, value -> value * idx })

如果转换在某些元素上产生 null 值，则可以通过调用 mapNotNull() 函数取代 map() 或 mapIndexedNotNull() 取代 mapIndexed() 来从结果集中过滤掉 null 值。

转换Map时，有两个选择：转换键，使值保持不变，反之亦然。 要将指定转换应用于键，请使用 mapKeys()；反过来，mapValues() 转换值。 这两个函数都使用将映射条目作为参数的转换，因此可以操作其键与值。

val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
println(numbersMap.mapKeys { it.key.toUpperCase() })
println(numbersMap.mapValues { it.value + it.key.length })

双路合并 转换是根据两个集合中具有相同位置的元素构建配对。 在 Kotlin 标准库中，这是通过 zip() 扩展函数完成的。 在一个集合（或数组）上以另一个集合（或数组）作为参数调用时，zip() 返回 Pair 对象的列表（List）。 接收者集合的元素是这些配对中的第一个元素。 如果集合的大小不同，则 zip() 的结果为较小集合的大小；结果中不包含较大集合的后续元素。 zip() 也可以中缀形式调用 a zip b 。

。。zip.py coming。。。

val colors = listOf("red", "brown", "grey")
val animals = listOf("fox", "bear", "wolf")
println(colors zip animals)
val twoAnimals = listOf("fox", "bear")
println(colors.zip(twoAnimals))

也可以使用带有两个参数的转换函数来调用 zip()：接收者元素和参数元素。 在这种情况下，结果 List 包含在具有相同位置的接收者对和参数元素对上调用的转换函数的返回值。

val colors = listOf("red", "brown", "grey")
val animals = listOf("fox", "bear", "wolf")

println(colors.zip(animals) { color, animal -> "The ${animal.capitalize()} is $color"})

。。就是在上面的基础上增加一个 {} 参数？，，估计这个zip 是 3参数的。感觉是 zip后执行 lambda

当拥有 Pair 的 List 时，可以进行反向转换 unzipping——从这些键值对中构建两个列表：
    第一个列表包含原始列表中每个 Pair 的键。
    第二个列表包含原始列表中每个 Pair 的值。

要分割键值对列表，请调用 unzip()。
val numberPairs = listOf("one" to 1, "two" to 2, "three" to 3, "four" to 4)
println(numberPairs.unzip())
，页面的输出是 ([one, two, three, four], [1, 2, 3, 4])

。。map也是xx to xx。。说明map里保存的是 pair。。抄cpp的啊，

关联 转换允许从集合元素和与其关联的某些值构建 Map。 在不同的关联类型中，元素可以是关联 Map 中的键或值。

基本的关联函数 associateWith() 创建一个 Map，其中原始集合的元素是键，并通过给定的转换函数从中产生值。 如果两个元素相等，则仅最后一个保留在 Map 中。

val numbers = listOf("one", "two", "three", "four")
println(numbers.associateWith { it.length })

为了使用集合元素作为值来构建 Map，有一个函数 associateBy()。 它需要一个函数，该函数根据元素的值返回键。如果两个元素相等，则仅最后一个保留在 Map 中。 还可以使用值转换函数来调用 associateBy()。

val numbers = listOf("one", "two", "three", "four")
println(numbers.associateBy { it.first().toUpperCase() })

println(numbers.associateBy(keySelector = { it.first().toUpperCase() }, valueTransform = { it.length }))

{O=one, T=three, F=four}
{O=3, T=5, F=4}

另一种构建 Map 的方法是使用函数 associate()，其中 Map 键和值都是通过集合元素生成的。 它需要一个 lambda 函数，该函数返回 Pair：键和相应 Map 条目的值。

请注意，associate() 会生成临时的 Pair 对象，这可能会影响性能。 因此，当性能不是很关键或比其他选项更可取时，应使用 associate()。

后者的一个示例：从一个元素一起生成键和相应的值。
val names = listOf("Alice Adams", "Brian Brown", "Clara Campbell")

println(names.associate { name -> parseFullName(name).let { it.lastName to it.firstName } })

此时，首先在一个元素上调用一个转换函数，然后根据该函数结果的属性建立 Pair。

。。associateWith，associateBy, associate

打平
。。这个翻译。。

如需操作嵌套的集合，则可能会发现提供对嵌套集合元素进行打平访问的标准库函数很有用。
第一个函数为 flatten()。可以在一个集合的集合（例如，一个 Set 组成的 List）上调用它。 该函数返回嵌套集合中的所有元素的一个 List。

val numberSets = listOf(setOf(1, 2, 3), setOf(4, 5, 6), setOf(1, 2))
println(numberSets.flatten())

[1, 2, 3, 4, 5, 6, 1, 2]

。。集合类型沿用最外面的集合类型，泛型是 集合内的集合的元素类型，对集合内的每个集合元素 遍历 它的每个元素，然后加入到ans中。
。no，始终返回list。

。。可以混写。。
val nss = listOf(setOf(1,2,3), setOf(4,5,6),setOf(1,2,3),listOf(1,2));
println(nss.flatten())
[1, 2, 3, 4, 5, 6, 1, 2, 3, 1, 2]

。。。
val nsa = setOf(listOf(1,2,3), listOf(3,4,5), setOf(3,2,1));
println(nsa.flatten())
[1, 2, 3, 3, 4, 5, 3, 2, 1]

。。flatten 始终返回 list。

。。怎么根据list 创建set。。。
println(nsa.flatten().toSet())
。。有意思，反过来的。。是list里的方法来转为set。。。不，这个方法是Iterable的方法。。。恐怖如斯。。

另一个函数——flatMap() 提供了一种灵活的方式来处理嵌套的集合。 它需要一个函数将一个集合元素映射到另一个集合。 因此，flatMap() 返回单个列表其中包含所有元素的值。 所以，flatMap() 表现为 map()（以集合作为映射结果）与 flatten() 的连续调用。

val containers = listOf(
    StringContainer(listOf("one", "two", "three")),
    StringContainer(listOf("four", "five", "six")),
    StringContainer(listOf("seven", "eight"))
)
println(containers.flatMap { it.values })

如果需要以可读格式检索集合内容，请使用将集合转换为字符串的函数：joinToString() 与 joinTo()。

joinToString() 根据提供的参数从集合元素构建单个 String。 joinTo() 执行相同的操作，但将结果附加到给定的 Appendable 对象。

当使用默认参数调用时，函数返回的结果类似于在集合上调用 toString()：各元素的字符串表示形式以空格分隔而成的 String。

val numbers = listOf("one", "two", "three", "four")
println(numbers)
println(numbers.joinToString())
val listString = StringBuffer("The list of numbers: ")
numbers.joinTo(listString)
println(listString)

输出：
[one, two, three, four]
one, two, three, four
The list of numbers: one, two, three, four

要构建自定义字符串表示形式，可以在函数参数 separator、prefix 与 postfix中指定其参数。 结果字符串将以 prefix 开头，以 postfix 结尾。除最后一个元素外，separator 将位于每个元素之后。

val numbers = listOf("one", "two", "three", "four")

println(numbers.joinToString(separator = " | ", prefix = "start: ", postfix = ": end"))

输出：
start: one | two | three | four: end

。。prefix是整个输出的prefix，，而不是 每个元素的prefix，，每个元素的prefix可以通过 separator 和prefix 共同实现。好吧，下面的 transform函数 更好。

对于较大的集合，可能需要指定 limit ——将包含在结果中元素的数量。 如果集合大小超出 limit，所有其他元素将被 truncated 参数的单个值替换。

val numbers = (1..100).toList()
println(numbers.joinToString(limit = 10, truncated = "<...>"))
输出
1, 2, 3, 4, 5, 6, 7, 8, 9, 10, <...>

最后，要自定义元素本身的表示形式，请提供 transform 函数。
val numbers = listOf("one", "two", "three", "four")
println(numbers.joinToString { "Element: ${it.toUpperCase()}"})
输出：
Element: ONE, Element: TWO, Element: THREE, Element: FOUR

https://www.kotlincn.net/docs/reference/collection-filtering.html

过滤是最常用的集合处理任务之一。在Kotlin中，过滤条件由 谓词 定义——接受一个集合元素并且返回布尔值的 lambda 表达式：true 说明给定元素与谓词匹配，false 则表示不匹配。

标准库包含了一组让你能够通过单个调用就可以过滤集合的扩展函数。这些函数**不会改变原始集合**，因此它们既可用于可变集合也可用于只读集合。为了操作过滤结果，应该在过滤后将其赋值给变量或链接其他函数。

基本的过滤函数是 filter()。当使用一个谓词来调用时，filter() 返回与其匹配的集合元素。对于 List 和 Set，过滤结果都是一个 List，对 Map 来说结果还是一个 Map。

val numbers = listOf("one", "two", "three", "four")
val longerThan3 = numbers.filter { it.length > 3 }
println(longerThan3)
val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)

val filteredMap = numbersMap.filter { (key, value) -> key.endsWith("1") && value > 10}

println(filteredMap)

filter() 中的谓词只能检查元素的值。如果想在过滤中使用元素在集合中的位置，应该使用 filterIndexed()。它接受一个带有两个参数的谓词：元素的索引和元素的值。

。。map那就是3个函数了？。。不，map没有下标。。所以map没有这种方法。。不不不，LinkedHashMap啊。。。不会写。。

如果想使用否定条件来过滤集合，请使用 filterNot()。它返回一个让谓词产生 false 的元素列表。

还有一些函数能够通过过滤给定类型的元素来缩小元素的类型：

    filterIsInstance() 返回给定类型的集合元素。在一个 List<Any> 上被调用时，filterIsInstance<T>() 返回一个 List<T>，从而让你能够在集合元素上调用 T 类型的函数。

val numbers = listOf(null, 1, "two", 3.0, "four")
println("All String elements in upper case:")
numbers.filterIsInstance<String>().forEach {
    println(it.toUpperCase())
}

filterNotNull() 返回所有的非空元素。在一个 List<T?> 上被调用时，filterNotNull() 返回一个 List<T: Any>，从而让你能够将所有元素视为非空对象。

val numbers = listOf(null, "one", "two", null)
numbers.filterNotNull().forEach {
println(it.length)   // 对可空的 String 来说长度不可用
}

另一个过滤函数 – partition() – 通过一个谓词过滤集合并且将不匹配的元素存放在一个单独的列表中。因此，你得到一个 List 的 Pair 作为返回值：第一个列表包含与谓词匹配的元素并且第二个列表包含原始集合中的所有其他元素。

val numbers = listOf("one", "two", "three", "four")
val (match, rest) = numbers.partition { it.length > 3 }
println(match)
println(rest)

。。多返回。。。

有些函数只是针对集合元素简单地检测一个谓词：
    如果至少有一个元素匹配给定谓词，那么 any() 返回 true。
    如果没有元素与给定谓词匹配，那么 none() 返回 true。

    如果所有元素都匹配给定谓词，那么 all() 返回 true。注意，在一个空集合上使用任何有效的谓词去调用 all() 都会返回 true 。这种行为在逻辑上被称为 vacuous truth。

val numbers = listOf("one", "two", "three", "four")
println(numbers.any { it.endsWith("e") })
println(numbers.none { it.endsWith("a") })
println(numbers.all { it.endsWith("e") })
println(emptyList<Int>().all { it > 5 })   // vacuous truth
输出：
true
true
false
true

any() 和 none() 也可以不带谓词使用：在这种情况下它们只是用来检查集合是否为空。 如果集合中有元素，any() 返回 true，否则返回 false；none() 则相反。

val numbers = listOf("one", "two", "three", "four")
val empty = emptyList<String>()
println(numbers.any())
println(empty.any())
println(numbers.none())
println(empty.none())

true
false
false
true

在 Kotlin 中，为集合定义了 plus (+) 和 minus (-) 操作符。 它们把一个集合作为第一个操作数；第二个操作数可以是一个元素或者是另一个集合。 返回值是一个新的只读集合：

    plus 的结果包含原始集合 和 第二个操作数中的元素。

    minus 的结果包含原始集合中的元素，但第二个操作数中的元素 除外。 如果第二个操作数是一个元素，那么 minus 移除其在原始集合中的 第一次 出现；如果是一个集合，那么移除其元素在原始集合中的 所有 出现。

val numbers = listOf("one", "two", "three", "four")
val plusList = numbers + "five"
val minusList = numbers - listOf("three", "four")
println(plusList)
println(minusList)
输出：
[one, two, three, four, five]
[one, two]

有关 map 的 plus 和 minus 操作符的详细信息，请参见 Map 相关操作。 也为集合定义了广义赋值操作符 plusAssign (+=) 和 minusAssign (-=)。 然而，对于只读集合，它们实际上使用 plus 或者 minus 操作符并尝试将结果赋值给同一变量。 因此，它们仅在由 var 声明的只读集合中可用。 对于可变集合，如果它是一个 val，那么它们会修改集合。更多详细信息请参见集合写操作。

https://www.kotlincn.net/docs/reference/collection-grouping.html

Kotlin 标准库提供用于对集合元素进行分组的扩展函数。 基本函数 groupBy() 使用一个 lambda 函数并返回一个 Map。 在此 Map 中，每个键都是 lambda 结果，而对应的值是返回此结果的元素 List。 例如，可以使用此函数将 String 列表按首字母分组。

还可以使用第二个 lambda 参数（值转换函数）调用 groupBy()。 在带有两个 lambda 的 groupBy() 结果 Map 中，由 keySelector 函数生成的键映射到值转换函数的结果，而不是原始元素。

val numbers = listOf("one", "two", "three", "four", "five")
println(numbers.groupBy { it.first().toUpperCase() })

println(numbers.groupBy(keySelector = { it.first() }, valueTransform = { it.toUpperCase() }))

{O=[one], T=[two, three], F=[four, five]}
{o=[ONE], t=[TWO, THREE], f=[FOUR, FIVE]}

如果要对元素进行分组，然后一次将操作应用于所有分组，请使用 groupingBy() 函数。 它返回一个 Grouping 类型的实例。 通过 Grouping 实例，可以以一种惰性的方式将操作应用于所有组：这些分组实际上是刚好在执行操作前构建的。

即，Grouping 支持以下操作：
    eachCount() 计算每个组中的元素。
    fold() 与 reduce() 对每个组分别执行 fold 与 reduce 操作，作为一个单独的集合并返回结果。

    aggregate() 随后将给定操作应用于每个组中的所有元素并返回结果。 这是对 Grouping 执行任何操作的通用方法。当折叠或缩小不够时，可使用它来实现自定义操作。

val numbers = listOf("one", "two", "three", "four", "five", "six")
println(numbers.groupingBy { it.first() }.eachCount())

{o=1, t=2, f=2, s=1}

https://www.kotlincn.net/docs/reference/collection-parts.html

Kotlin 标准库包含用于取集合的一部分的扩展函数。 这些函数提供了多种方法来选择结果集合的元素：显式列出其位置、指定结果大小等。

Slice
slice() 返回具有给定索引的集合元素列表。 索引既可以是作为区间传入的也可以是作为整数值的集合传入的。

val numbers = listOf("one", "two", "three", "four", "five", "six")
println(numbers.slice(1..3))
println(numbers.slice(0..4 step 2))
println(numbers.slice(setOf(3, 5, 0)))

Take 与 drop

要从头开始获取指定数量的元素，请使用 take() 函数。 要从尾开始获取指定数量的元素，请使用 takeLast()。 当调用的数字大于集合的大小时，两个函数都将返回整个集合。

要从头或从尾去除给定数量的元素，请调用 drop() 或 dropLast() 函数。
val numbers = listOf("one", "two", "three", "four", "five", "six")
println(numbers.take(3))
println(numbers.takeLast(3))
println(numbers.drop(1))
println(numbers.dropLast(5))

还可以使用谓词来定义要获取或去除的元素的数量。 有四个与上述功能相似的函数：
    takeWhile() 是带有谓词的 take()：它将不停获取元素直到排除与谓词匹配的首个元素。如果首个集合元素与谓词匹配，则结果为空。

    takeLastWhile() 与 takeLast() 类似：它从集合末尾获取与谓词匹配的元素区间。区间的首个元素是与谓词不匹配的最后一个元素右边的元素。如果最后一个集合元素与谓词匹配，则结果为空。

    dropWhile() 与具有相同谓词的 takeWhile() 相反：它将首个与谓词不匹配的元素返回到末尾。
    dropLastWhile() 与具有相同谓词的 takeLastWhile() 相反：它返回从开头到最后一个与谓词不匹配的元素。

val numbers = listOf("one", "two", "three", "four", "five", "six")
println(numbers.takeWhile { !it.startsWith('f') })
println(numbers.takeLastWhile { it != "three" })
println(numbers.dropWhile { it.length == 3 })
println(numbers.dropLastWhile { it.contains('i') })

Chunked

要将集合分解为给定大小的“块”，请使用 chunked() 函数。 chunked() 采用一个参数（块的大小），并返回一个 List 其中包含给定大小的 List。 第一个块从第一个元素开始并包含 size 元素，第二个块包含下一个 size 元素，依此类推。 最后一个块的大小可能较小。

val numbers = (0..13).toList()
println(numbers.chunked(3))

[[0, 1, 2], [3, 4, 5], [6, 7, 8], [9, 10, 11], [12, 13]]

还可以立即对返回的块应用转换。 为此，请在调用 chunked() 时将转换作为 lambda 函数提供。 lambda 参数是集合的一块。当通过转换调用 chunked() 时， 这些块是临时的 List，应立即在该 lambda 中使用。

val numbers = (0..13).toList()
println(numbers.chunked(3) { it.sum() })  // `it` 为原始集合的一个块

[3, 12, 21, 30, 25]

可以检索给定大小的集合元素中所有可能区间。 获取它们的函数称为 windowed()：它返回一个元素区间列表，比如通过给定大小的滑动窗口查看集合，则会看到该区间。 与 chunked() 不同，windowed() 返回从每个集合元素开始的元素区间（窗口）。 所有窗口都作为单个 List 的元素返回。

val numbers = listOf("one", "two", "three", "four", "five")
println(numbers.windowed(3))

[[one, two, three], [two, three, four], [three, four, five]]

。。sliding window，，，kuangxi？

windowed() 通过可选参数提供更大的灵活性：

    step 定义两个相邻窗口的第一个元素之间的距离。默认情况下，该值为 1，因此结果包含从所有元素开始的窗口。如果将 step 增加到 2，将只收到以奇数元素开头的窗口：第一个、第三个等。

    partialWindows 包含从集合末尾的元素开始的较小的窗口。例如，如果请求三个元素的窗口，就不能为最后两个元素构建它们。在本例中，启用 partialWindows 将包括两个大小为2与1的列表。

最后，可以立即对返回的区间应用转换。 为此，在调用 windowed() 时将转换作为 lambda 函数提供。

val numbers = (1..10).toList()
println(numbers.windowed(3, step = 2, partialWindows = true))
println(numbers.windowed(3) { it.sum() })

[[1, 2, 3], [3, 4, 5], [5, 6, 7], [7, 8, 9], [9, 10]]
[6, 9, 12, 15, 18, 21, 24, 27]

要构建两个元素的窗口，有一个单独的函数——zipWithNext()。 它创建接收器集合的相邻元素对。 请注意，zipWithNext() 不会将集合分成几对；它为 每个 元素创建除最后一个元素外的对，因此它在 [1, 2, 3, 4] 上的结果为 [[1, 2], [2, 3], [3, 4]]，而不是 [[1, 2]，[3, 4]]。 zipWithNext() 也可以通过转换函数来调用；它应该以接收者集合的两个元素作为参数。

val numbers = listOf("one", "two", "three", "four", "five")
println(numbers.zipWithNext())
println(numbers.zipWithNext() { s1, s2 -> s1.length > s2.length})

https://www.kotlincn.net/docs/reference/collection-elements.html

Kotlin 集合提供了一套从集合中检索单个元素的函数。 此页面描述的函数适用于 list 和 set。

从定义来看，set 并不是有序集合。 但是，Kotlin 中的 Set 按某些顺序存储元素。 这些可以是插入顺序（在 LinkedHashSet 中）、自然排序顺序（在 SortedSet 中）或者其他顺序。 一组元素的顺序也可以是未知的。在这种情况下，元素仍会以某种顺序排序，因此，依赖元素位置的函数仍会返回其结果。 但是，除非调用者知道所使用的 Set 的具体实现，否则这些结果对于调用者是不可预测的。

为了检索特定位置的元素，有一个函数 elementAt()。 用一个整数作为参数来调用它，你会得到给定位置的集合元素。 第一个元素的位置是 0，最后一个元素的位置是 (size - 1)。

elementAt() 对于不提供索引访问或非静态已知提供索引访问的集合很有用。 在使用 List 的情况下，使用索引访问操作符 （get() 或 []）更为习惯。

val numbers = linkedSetOf("one", "two", "three", "four", "five")
println(numbers.elementAt(3))

val numbersSortedSet = sortedSetOf("one", "two", "three", "four")
println(numbersSortedSet.elementAt(0)) // 元素以升序存储

还有一些有用的别名来检索集合的第一个和最后一个元素：first() 和 last()。
val numbers = listOf("one", "two", "three", "four", "five")
println(numbers.first())
println(numbers.last())

为了避免在检索位置不存在的元素时出现异常，请使用 elementAt() 的安全变体：
    当指定位置超出集合范围时，elementAtOrNull() 返回 null。

    elementAtOrElse() 还接受一个 lambda 表达式，该表达式能将一个 Int 参数映射为一个集合元素类型的实例。 当使用一个越界位置来调用时，elementAtOrElse() 返回对给定值调用该 lambda 表达式的结果。

函数 first() 和 last() 还可以让你在集合中搜索与给定谓词匹配的元素。 当你使用测试集合元素的谓词调用 first() 时，你会得到对其调用谓词产生 true 的第一个元素。 反过来，带有一个谓词的 last() 返回与其匹配的最后一个元素。

val numbers = listOf("one", "two", "three", "four", "five", "six")
println(numbers.first { it.length > 3 })
println(numbers.last { it.startsWith("f") })

如果没有元素与谓词匹配，两个函数都会抛异常。 为了避免它们，请改用 firstOrNull() 和 lastOrNull()：如果找不到匹配的元素，它们将返回 null。

或者，如果别名更适合你的情况，那么可以使用别名：
    使用 find() 代替 firstOrNull()
    使用 findLast() 代替 lastOrNull()

val numbers = listOf(1, 2, 3, 4)
println(numbers.find { it % 2 == 0 })
println(numbers.findLast { it % 2 == 0 })

如果需要检索集合的一个随机元素，那么请调用 random() 函数。 你可以不带参数或者使用一个 Random 对象作为随机源来调用它。
val numbers = listOf(1, 2, 3, 4)
println(numbers.random())

On empty collections, random() throws an exception. To receive null instead, use randomOrNull()

如需检查集合中某个元素的存在，可以使用 contains() 函数。 如果存在一个集合元素等于（equals()）函数参数，那么它返回 true。 你可以使用 in 关键字以操作符的形式调用 contains()。

如需一次检查多个实例的存在，可以使用这些实例的集合作为参数调用 containsAll()。

val numbers = listOf("one", "two", "three", "four", "five", "six")
println(numbers.contains("four"))
println("zero" in numbers)

println(numbers.containsAll(listOf("four", "two")))
println(numbers.containsAll(listOf("one", "zero")))

此外，你可以通过调用 isEmpty() 和 isNotEmpty() 来检查集合中是否包含任何元素。
val numbers = listOf("one", "two", "three", "four", "five", "six")
println(numbers.isEmpty())
println(numbers.isNotEmpty())
val empty = emptyList<String>()
println(empty.isEmpty())
println(empty.isNotEmpty())

。。没有containsAny ?

https://www.kotlincn.net/docs/reference/collection-ordering.html

元素的顺序是某些集合类型的一个重要方面。 例如，如果拥有相同元素的两个列表的元素顺序不同，那么这两个列表也不相等。

。。 == 等于 equals。  === 才是 地址等于。。

在 Kotlin 中，可以通过多种方式定义对象的顺序。

首先，有 自然 顺序。它是为 Comparable 接口的继承者定义的。 当没有指定其他顺序时，使用自然顺序为它们排序。

大多数内置类型是可比较的：
    数值类型使用传统的数值顺序：1 大于 0； -3.4f 大于 -5f，以此类推。
    Char 和 String 使用字典顺序： b 大于 a； world 大于 hello。

如需为用户定义的类型定义一个自然顺序，可以让这个类型继承 Comparable。 这需要实现 compareTo() 函数。 compareTo() 必须将另一个具有相同类型的对象作为参数并返回一个整数值来显示哪个对象更大：

    正值表明接收者对象更大。
    负值表明它小于参数。
    0 说明对象相等。

下面是一个类，可用于排序由主版本号和次版本号两部分组成的版本。

class Version(val major: Int, val minor: Int): Comparable<Version> {
    override fun compareTo(other: Version): Int {
        if (this.major != other.major) {
            return this.major - other.major
        } else if (this.minor != other.minor) {
            return this.minor - other.minor
        } else return 0
    }
}

fun main() {
    println(Version(1, 2) > Version(1, 3))
    println(Version(2, 0) > Version(1, 5))
}

自定义 顺序让你可以按自己喜欢的方式对任何类型的实例进行排序。 特别是，你可以为不可比较类型定义顺序，或者为可比较类型定义非自然顺序。 如需为类型定义自定义顺序，可以为其创建一个 Comparator。 Comparator 包含 compare() 函数：它接受一个类的两个实例并返回它们之间比较的整数结果。 如上所述，对结果的解释与 compareTo() 的结果相同。

val lengthComparator = Comparator { str1: String, str2: String -> str1.length - str2.length }

println(listOf("aaa", "bb", "c").sortedWith(lengthComparator))

有了 lengthComparator，你可以按照字符串的长度而不是默认的字典顺序来排列字符串。

定义一个 Comparator 的一种比较简短的方式是标准库中的 compareBy() 函数。 compareBy() 接受一个 lambda 表达式，该表达式从一个实例产生一个 Comparable 值，并将自定义顺序定义为生成值的自然顺序。 使用 compareBy()，上面示例中的长度比较器如下所示：

println(listOf("aaa", "bb", "c").sortedWith(compareBy { it.length }))

基本的函数 sorted() 和 sortedDescending() 返回集合的元素，这些元素按照其自然顺序升序和降序排序。 这些函数适用于 Comparable 元素的集合。

val numbers = listOf("one", "two", "three", "four")
println("Sorted ascending: ${numbers.sorted()}")
println("Sorted descending: ${numbers.sortedDescending()}")

自定义顺序

为了按照自定义顺序排序或者对不可比较对象排序，可以使用函数 sortedBy() 和 sortedByDescending()。 它们接受一个将集合元素映射为 Comparable 值的选择器函数，并以该值的自然顺序对集合排序。

val numbers = listOf("one", "two", "three", "four")
val sortedNumbers = numbers.sortedBy { it.length }
println("Sorted by length ascending: $sortedNumbers")
val sortedByLast = numbers.sortedByDescending { it.last() }
println("Sorted by the last letter descending: $sortedByLast")

如需为集合排序定义自定义顺序，可以提供自己的 Comparator。 为此，调用传入 Comparator 的 sortedWith() 函数。 使用此函数，按照字符串长度排序如下所示：

val numbers = listOf("one", "two", "three", "four")

println("Sorted by length ascending: ${numbers.sortedWith(compareBy { it.length })}")

你可以使用 reversed() 函数以相反的顺序检索集合。
val numbers = listOf("one", "two", "three", "four")
println(numbers.reversed())

reversed() 返回带有元素副本的新集合。 因此，如果你之后改变了原始集合，这并不会影响先前获得的 reversed() 的结果。

另一个反向函数——asReversed()——返回相同集合实例的一个反向视图，因此，如果原始列表不会发生变化，那么它会比 reversed() 更轻量，更合适。

val numbers = listOf("one", "two", "three", "four")
val reversedNumbers = numbers.asReversed()
println(reversedNumbers)

如果原始列表是可变的，那么其所有更改都会反映在其反向视图中，反之亦然。
val numbers = mutableListOf("one", "two", "three", "four")
val reversedNumbers = numbers.asReversed()
println(reversedNumbers)
numbers.add("five")
println(reversedNumbers)

但是，如果列表的可变性未知或者源根本不是一个列表，那么 reversed() 更合适，因为其结果是一个未来不会更改的副本。

最后，shuffled() 函数返回一个包含了以随机顺序排序的集合元素的新的 List。 你可以不带参数或者使用 Random 对象来调用它。
val numbers = listOf("one", "two", "three", "four")
println(numbers.shuffled())

https://www.kotlincn.net/docs/reference/collection-aggregate.html

Kotlin 集合包含用于常用的 聚合操作 （基于集合内容返回单个值的操作）的函数 。 其中大多数是众所周知的，并且其工作方式与在其他语言中相同。
    min() 与 max() 分别返回最小和最大的元素；
    average() 返回数字集合中元素的平均值；
    sum() 返回数字集合中元素的总和；
    count() 返回集合中元素的数量；

val numbers = listOf(6, 42, 10, 4)
println("Count: ${numbers.count()}")
println("Max: ${numbers.max()}")
println("Min: ${numbers.min()}")
println("Average: ${numbers.average()}")
println("Sum: ${numbers.sum()}")

还有一些通过某些选择器函数或自定义 Comparator 来检索最小和最大元素的函数。
    maxBy()/minBy() 接受一个选择器函数并返回使选择器返回最大或最小值的元素。
    maxWith()/minWith() 接受一个 Comparator 对象并且根据此 Comparator 对象返回最大或最小元素。

val numbers = listOf(5, 42, 10, 4)
val min3Remainder = numbers.minBy { it % 3 }
println(min3Remainder)
val strings = listOf("one", "two", "three", "four")
val longestString = strings.maxWith(compareBy { it.length })
println(longestString)

此外，有一些高级的求和函数，它们接受一个函数并返回对所有元素调用此函数的返回值的总和：
    sumBy() 使用对集合元素调用返回 Int 值的函数。
    sumByDouble() 与返回 Double 的函数一起使用。

val numbers = listOf(5, 42, 10, 4)
println(numbers.sumBy { it * 2 })
println(numbers.sumByDouble { it.toDouble() / 2 })

对于更特定的情况，有函数 reduce() 和 fold()，它们依次将所提供的操作应用于集合元素并返回累积的结果。 操作有两个参数：先前的累积值和集合元素。

这两个函数的区别在于：fold() 接受一个初始值并将其用作第一步的累积值，而 reduce() 的第一步则将第一个和第二个元素作为第一步的操作参数。

..accumulate

val numbers = listOf(5, 2, 10, 4)
val sum = numbers.reduce { sum, element -> sum + element }
println(sum)
val sumDoubled = numbers.fold(0) { sum, element -> sum + element * 2 }
println(sumDoubled)

//val sumDoubledReduce = numbers.reduce { sum, element -> sum + element * 2 } //错误：第一个元素在结果中没有加倍

//println(sumDoubledReduce)

如需将函数以相反的顺序应用于元素，可以使用函数 reduceRight() 和 foldRight() 它们的工作方式类似于 fold() 和 reduce()，但从最后一个元素开始，然后再继续到前一个元素。 记住，在使用 foldRight 或 reduceRight 时，操作参数会更改其顺序：第一个参数变为元素，然后第二个参数变为累积值。

val numbers = listOf(5, 2, 10, 4)

val sumDoubledRight = numbers.foldRight(0) { element, sum -> sum + element * 2 }

println(sumDoubledRight)

你还可以使用将元素索引作为参数的操作。 为此，使用函数 reduceIndexed() 和 foldIndexed() 传递元素索引作为操作的第一个参数。

最后，还有将这些操作从右到左应用于集合元素的函数——reduceRightIndexed() 与 foldRightIndexed()。

val numbers = listOf(5, 2, 10, 4)

val sumEven = numbers.foldIndexed(0) { idx, sum, element -> if (idx % 2 == 0) sum + element else sum }

println(sumEven)

val sumEvenRight = numbers.foldRightIndexed(0) { idx, element, sum -> if (idx % 2 == 0) sum + element else sum }

println(sumEvenRight)

All reduce operations throw an exception on empty collections. To receive null instead, use their *OrNull() counterparts:

    reduceOrNull()
    reduceRightOrNull()
    reduceIndexedOrNull()
    reduceRightIndexedOrNull()

https://www.kotlincn.net/docs/reference/collection-write.html

可变集合支持更改集合内容的操作，例如添加或删除元素。 在此页面上，我们将描述实现 MutableCollection 的所有写操作。 有关 List 与 Map 可用的更多特定操作，请分别参见 List 相关操作与 Map 相关操作。

要将单个元素添加到列表或集合，请使用 add() 函数。指定的对象将添加到集合的末尾。
val numbers = mutableListOf(1, 2, 3, 4)
numbers.add(5)
println(numbers)

addAll() 将参数对象的每个元素添加到列表或集合中。参数可以是 Iterable、Sequence 或 Array。 接收者的类型和参数可能不同，例如，你可以将所有内容从 Set 添加到 List。

当在列表上调用时，addAll() 会按照在参数中出现的顺序添加各个新元素。 你也可以调用 addAll() 时指定一个元素位置作为第一参数。 参数集合的第一个元素会被插入到这个位置。 其他元素将跟随在它后面，将接收者元素移到末尾。

val numbers = mutableListOf(1, 2, 5, 6)
numbers.addAll(arrayOf(7, 8))
println(numbers)
numbers.addAll(2, setOf(3, 4))
println(numbers)

你还可以使用 plus 运算符 - plusAssign (+=) 添加元素。 当应用于可变集合时，+= 将第二个操作数(一个元素或另一个集合)追加到集合的末尾。

val numbers = mutableListOf("one", "two")
numbers += "three"
println(numbers)
numbers += listOf("four", "five")
println(numbers)

若要从可变集合中移除元素，请使用 remove() 函数。 remove() 接受元素值，并删除该值的一个匹配项。
val numbers = mutableListOf(1, 2, 3, 4, 3)
numbers.remove(3)                    // 删除了第一个 `3`
println(numbers)
numbers.remove(5)                    // 什么都没删除
println(numbers)

要一次删除多个元素，有以下函数：
    removeAll() 移除参数集合中存在的所有元素。 或者，你可以用谓词作为参数来调用它；在这种情况下，函数移除谓词产生 true 的所有元素。
    retainAll() 与 removeAll() 相反：它移除除参数集合中的元素之外的所有元素。 当与谓词一起使用时，它只留下与之匹配的元素。
    clear() 从列表中移除所有元素并将其置空。

从集合中移除元素的另一种方法是使用 minusAssign (-=) ——原地修改版的 minus 操作符。 minus 操作符。 第二个参数可以是元素类型的单个实例或另一个集合。 右边是单个元素时，-= 会移除它的第一个匹配项。 反过来，如果它是一个集合，那么它的所有元素的每次出现都会删除。 例如，如果列表包含重复的元素，它们将被同时删除。 第二个操作数可以包含集合中不存在的元素。这些元素不会影响操作的执行。

https://www.kotlincn.net/docs/reference/list-operations.html
List 相关操作

List 支持按索引取元素的所有常用操作： elementAt() 、 first() 、 last() 与取单个元素中列出的其他操作。 List 的特点是能通过索引访问特定元素，因此读取元素的最简单方法是按索引检索它。 这是通过 get() 函数或简写语法 [index] 来传递索引参数完成的。

如果 List 长度小于指定的索引，则抛出异常。 另外，还有两个函数能避免此类异常：
    getOrElse() 提供用于计算默认值的函数，如果集合中不存在索引，则返回默认值。
    getOrNull() 返回 null 作为默认值。

除了取集合的一部分中常用的操作， List 还提供 subList() 该函数将指定元素范围的视图作为列表返回。 因此，如果原始集合的元素发生变化，则它在先前创建的子列表中也会发生变化，反之亦然。

val numbers = (0..13).toList()
println(numbers.subList(3, 6))

在任何列表中，都可以使用 indexOf() 或 lastIndexOf() 函数找到元素的位置。 它们返回与列表中给定参数相等的元素的第一个或最后一个位置。 如果没有这样的元素，则两个函数均返回 -1。

还有一对函数接受谓词并搜索与之匹配的元素：
    indexOfFirst() 返回与谓词匹配的第一个元素的索引，如果没有此类元素，则返回 -1。
    indexOfLast() 返回与谓词匹配的最后一个元素的索引，如果没有此类元素，则返回 -1。

要搜索已排序列表中的元素，请调用 binarySearch() 函数，并将该值作为参数传递。 如果存在这样的元素，则函数返回其索引；否则，将返回 (-insertionPoint - 1)，其中 insertionPoint 为应插入此元素的索引，以便列表保持排序。 如果有多个具有给定值的元素，搜索则可以返回其任何索引。

还可以指定要搜索的索引区间：在这种情况下，该函数仅在两个提供的索引之间搜索。
val numbers = mutableListOf("one", "two", "three", "four")
numbers.sort()
println(numbers)
println(numbers.binarySearch("two"))  // 3
println(numbers.binarySearch("z")) // -5
println(numbers.binarySearch("two", 0, 2))  // -3

如果列表元素不是 Comparable，则应提供一个用于二分搜索的 Comparator。 该列表必须根据此 Comparator 以升序排序。来看一个例子：
val productList = listOf(
    Product("WebStorm", 49.0),
    Product("AppCode", 99.0),
    Product("DotTrace", 129.0),
    Product("ReSharper", 149.0))

println(productList.binarySearch(Product("AppCode", 99.0), compareBy<Product> { it.price }.thenBy { it.name }))

当列表使用与自然排序不同的顺序时（例如，对 String 元素不区分大小写的顺序），自定义 Comparator 也很方便。
val colors = listOf("Blue", "green", "ORANGE", "Red", "yellow")
println(colors.binarySearch("RED", String.CASE_INSENSITIVE_ORDER)) // 3

..String.CASE_INSENSITIVE_ORDER

使用 比较 函数的二分搜索无需提供明确的搜索值即可查找元素。 取而代之的是，它使用一个比较函数将元素映射到 Int 值，并搜索函数返回 0 的元素。 该列表必须根据提供的函数以升序排序；换句话说，比较的返回值必须从一个列表元素增长到下一个列表元素。

data class Product(val name: String, val price: Double)

fun priceComparison(product: Product, price: Double) = sign(product.price - price).toInt()

fun main() {
    val productList = listOf(
        Product("WebStorm", 49.0),
        Product("AppCode", 99.0),
        Product("DotTrace", 129.0),
        Product("ReSharper", 149.0))
    println(productList.binarySearch { priceComparison(it, 99.0) })
}

Comparator 与比较函数二分搜索都可以针对列表区间执行。

List 写操作
除了集合写操作中描述的集合修改操作之外，可变列表还支持特定的写操作。 这些操作使用索引来访问元素以扩展列表修改功能。

要将元素添加到列表中的特定位置，请使用 add() 或 addAll() 并提供元素插入的位置作为附加参数。 位置之后的所有元素都将向右移动。
val numbers = mutableListOf("one", "five", "six")
numbers.add(1, "two")
numbers.addAll(2, listOf("three", "four"))
println(numbers)

列表还提供了在指定位置替换元素的函数——set() 及其操作符形式 []。set() 不会更改其他元素的索引。
val numbers = mutableListOf("one", "five", "three")
numbers[1] =  "two"
println(numbers)

fill() 简单地将所有集合元素的值替换为指定值。
val numbers = mutableListOf(1, 2, 3, 4)
numbers.fill(3)
println(numbers)

要从列表中删除指定位置的元素，请使用 removeAt() 函数，并将位置作为参数。 在元素被删除之后出现的所有元素索引将减 1。
val numbers = mutableListOf(1, 2, 3, 4, 3)
numbers.removeAt(1)
println(numbers)

For removing the first and the last element, there are handy shortcuts removeFirst() and removeLast(). Note that on empty lists, they throw an exception. To receive null instead, use removeFirstOrNull() and removeLastOrNull()

在集合排序中，描述了按特定顺序检索集合元素的操作。 对于可变列表，标准库中提供了类似的扩展函数，这些扩展函数可以执行相同的排序操作。 将此类操作应用于列表实例时，它将更改指定实例中元素的顺序。

就地排序函数的名称与应用于只读列表的函数的名称相似，但没有 ed/d 后缀：
    sort* 在所有排序函数的名称中代替 sorted*：sort()、sortDescending()、sortBy() 等等。
    shuffle() 代替 shuffled()。
    reverse() 代替 reversed()。

asReversed() 在可变列表上调用会返回另一个可变列表，该列表是原始列表的反向视图。在该视图中的更改将反映在原始列表中。

https://www.kotlincn.net/docs/reference/set-operations.html

Set 相关操作

Kotlin 集合包中包含 set 常用操作的扩展函数：查找交集、并集或差集。

要将两个集合合并为一个（并集），可使用 union() 函数。也能以中缀形式使用 a union b。 注意，对于有序集合，操作数的顺序很重要：在结果集合中，左侧操作数在前。

要查找两个集合中都存在的元素（交集），请使用 intersect() 。 要查找另一个集合中不存在的集合元素（差集），请使用 subtract() 。 这两个函数也能以中缀形式调用，例如， a intersect b 。

val numbers = setOf("one", "two", "three")
println(numbers union setOf("four", "five"))
println(setOf("four", "five") union numbers)
println(numbers intersect setOf("two", "one"))
println(numbers subtract setOf("three", "four"))
println(numbers subtract setOf("four", "three")) // 相同的输出

注意， List 也支持 Set 操作。 但是，对 List 进行 Set 操作的结果仍然是 Set ，因此将删除所有重复的元素。

https://www.kotlincn.net/docs/reference/map-operations.html
Map 相关操作

要从 Map 中检索值，必须提供其键作为 get() 函数的参数。 还支持简写 [key] 语法。 如果找不到给定的键，则返回 null 。 还有一个函数 getValue() ，它的行为略有不同：如果在 Map 中找不到键，则抛出异常。 此外，还有两个选项可以解决键缺失的问题：

    getOrElse() 与 list 的工作方式相同：对于不存在的键，其值由给定的 lambda 表达式返回。
    getOrDefault() 如果找不到键，则返回指定的默认值。

要对 map 的所有键或所有值执行操作，可以从属性 keys 和 values 中相应地检索它们。 keys 是 Map 中所有键的集合， values 是 Map 中所有值的集合。

val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
println(numbersMap.keys)
println(numbersMap.values)

可以使用 filter() 函数来过滤 map 或其他集合。 对 map 使用 filter() 函数时， Pair 将作为参数的谓词传递给它。 它将使用谓词同时过滤其中的键和值。

val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)

val filteredMap = numbersMap.filter { (key, value) -> key.endsWith("1") && value > 10}

println(filteredMap)

还有两种用于过滤 map 的特定函数：按键或按值。 这两种方式，都有对应的函数： filterKeys() 和 filterValues() 。 两者都将返回一个新 Map ，其中包含与给定谓词相匹配的条目。 filterKeys() 的谓词仅检查元素键， filterValues() 的谓词仅检查值。

val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
val filteredKeysMap = numbersMap.filterKeys { it.endsWith("1") }
val filteredValuesMap = numbersMap.filterValues { it < 10 }
println(filteredKeysMap)
println(filteredValuesMap)

由于需要访问元素的键，plus（+）与 minus（-）运算符对 map 的作用与其他集合不同。 plus 返回包含两个操作数元素的 Map ：左侧的 Map 与右侧的 Pair 或另一个 Map 。 当右侧操作数中有左侧 Map 中已存在的键时，该条目将使用右侧的值。

val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
println(numbersMap + Pair("four", 4))
println(numbersMap + Pair("one", 10))
println(numbersMap + mapOf("five" to 5, "one" to 11))

minus 将根据左侧 Map 条目创建一个新 Map ，右侧操作数带有键的条目将被剔除。 因此，右侧操作数可以是单个键或键的集合： list 、 set 等。

val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
println(numbersMap - "one")
println(numbersMap - listOf("two", "four"))

。。上面是不可变的Map的操作，所以返回一个新的Map

Mutable Map （可变 Map ）提供特定的 Map 写操作。 这些操作使你可以使用键来访问或更改 Map 值。
Map 写操作的一些规则：
    值可以更新。 反过来，键也永远不会改变：添加条目后，键是不变的。
    每个键都有一个与之关联的值。也可以添加和删除整个条目。

要将新的键值对添加到可变 Map ，请使用 put() 。 将新条目放入 LinkedHashMap （Map的默认实现）后，会添加该条目，以便在 Map 迭代时排在最后。 在 Map 类中，新元素的位置由其键顺序定义。

val numbersMap = mutableMapOf("one" to 1, "two" to 2)
numbersMap.put("three", 3)
println(numbersMap)

要一次添加多个条目，请使用 putAll() 。它的参数可以是 Map 或一组 Pair ： Iterable 、 Sequence 或 Array 。
val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
numbersMap.putAll(setOf("four" to 4, "five" to 5))
println(numbersMap)

如果给定键已存在于 Map 中，则 put() 与 putAll() 都将覆盖值。 因此，可以使用它们来更新 Map 条目的值。

还可以使用快速操作符将新条目添加到 Map 。 有两种方式：
    plusAssign （+=） 操作符。
    [] 操作符为 put() 的别名。

val numbersMap = mutableMapOf("one" to 1, "two" to 2)
numbersMap["three"] = 3     // 调用 numbersMap.put("three", 3)
numbersMap += mapOf("four" to 4, "five" to 5)
println(numbersMap)

要从可变 Map 中删除条目，请使用 remove() 函数。 调用 remove() 时，可以传递键或整个键值对。 如果同时指定键和值，则仅当键值都匹配时，才会删除此的元素。

val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
numbersMap.remove("one")
println(numbersMap)
numbersMap.remove("three", 4)            //不会删除任何条目
println(numbersMap)

还可以通过键或值从可变 Map 中删除条目。 在 Map 的 .keys 或 .values 中调用 remove() 并提供键或值来删除条目。 在 .values 中调用时， remove() 仅删除给定值匹配到的的第一个条目。

val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3, "threeAgain" to 3)

numbersMap.keys.remove("one")
println(numbersMap)
numbersMap.values.remove(3)
println(numbersMap)

minusAssign （-=） 操作符也可用于可变 Map 。
val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
numbersMap -= "two"
println(numbersMap)
numbersMap -= "five"             //不会删除任何条目
println(numbersMap)

https://www.kotlincn.net/docs/reference/coroutines/coroutines-guide.html
协程

async 与 await 在 Kotlin 中并不是关键字，甚至都不是标准库的一部分。此外，Kotlin 的 挂起函数 概念为异步操作提供了比 future 与 promise 更安全、更不易出错的抽象。

kotlinx.coroutines 是由 JetBrains 开发的功能丰富的协程库。它包含本指南中涵盖的很多启用高级协程的原语，包括 launch、 async 等等。

本文是关于 kotlinx.coroutines 核心特性的指南，包含一系列示例，并分为不同的主题。

为了使用协程以及按照本指南中的示例演练，需要添加对 kotlinx-coroutines-core 模块的依赖，如项目中的 README 文件所述。

import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch { // 在后台启动一个新的协程并继续
        delay(1000L) // 非阻塞的等待 1 秒钟（默认时间单位是毫秒）
        println("World!") // 在延迟后打印输出
    }
    println("Hello,") // 协程已在等待时主线程还在继续
    Thread.sleep(2000L) // 阻塞主线程 2 秒钟来保证 JVM 存活
}

本质上，协程是轻量级的线程。 它们在某些 CoroutineScope 上下文中与 launch 协程构建器 一起启动。 这里我们在 GlobalScope 中启动了一个新的协程，这意味着新协程的生命周期只受整个应用程序的生命周期限制。

可以将 GlobalScope.launch { …… } 替换为 thread { …… }，并将 delay(……) 替换为 Thread.sleep(……) 达到同样目的。 试试看（不要忘记导入 kotlin.concurrent.thread）。

如果你首先将 GlobalScope.launch 替换为 thread，编译器会报以下错误：

Error: Kotlin: Suspend functions are only allowed to be called from a coroutine or another suspend function

这是因为 delay 是一个特殊的 挂起函数 ，它不会造成线程阻塞，但是会 挂起 协程，并且只能在协程中使用。

桥接阻塞与非阻塞的世界

第一个示例在同一段代码中混用了 非阻塞的 delay(……) 与 阻塞的 Thread.sleep(……)。 这容易让我们记混哪个是阻塞的、哪个是非阻塞的。 让我们显式使用 runBlocking 协程构建器来阻塞：

import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch { // 在后台启动一个新的协程并继续
        delay(1000L)
        println("World!")
    }
    println("Hello,") // 主线程中的代码会立即执行
    runBlocking {     // 但是这个表达式阻塞了主线程
        delay(2000L)  // ……我们延迟 2 秒来保证 JVM 的存活
    }
}

结果是相似的，但是这些代码只使用了非阻塞的函数 delay。 调用了 runBlocking 的主线程会一直 阻塞 直到 runBlocking 内部的协程执行完毕。

这个示例可以使用更合乎惯用法的方式重写，使用 runBlocking 来包装 main 函数的执行：
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> { // 开始执行主协程
    GlobalScope.launch { // 在后台启动一个新的协程并继续
        delay(1000L)
        println("World!")
    }
    println("Hello,") // 主协程在这里会立即执行
    delay(2000L)      // 延迟 2 秒来保证 JVM 存活
}

这里的 runBlocking<Unit> { …… } 作为用来启动顶层主协程的适配器。 我们显式指定了其返回类型 Unit，因为在 Kotlin 中 main 函数必须返回 Unit 类型。

这也是为挂起函数编写单元测试的一种方式：
class MyTest {
    @Test
    fun testMySuspendingFunction() = runBlocking<Unit> {
        // 这里我们可以使用任何喜欢的断言风格来使用挂起函数
    }
}

延迟一段时间来等待另一个协程运行并不是一个好的选择。让我们显式（以非阻塞方式）等待所启动的后台 Job 执行结束：
val job = GlobalScope.launch { // 启动一个新协程并保持对这个作业的引用
    delay(1000L)
    println("World!")
}
println("Hello,")
job.join() // 等待直到子协程执行结束

协程的实际使用还有一些需要改进的地方。

当我们使用GlobalScope.launch时，我们会创建一个顶层协程。虽然它很轻量，但它运行时仍会消耗一些内存资源。如果我们忘记保持对新启动的协程的引用，它还会继续运行。如果协程中的代码挂起了会怎么样（例如，我们错误地延迟了太长时间），如果我们启动了太多的协程并导致内存不足会怎么样？ 必须手动保持对所有已启动协程的引用并 join 之很容易出错。

有一个更好的解决办法。我们可以在代码中使用结构化并发。 我们可以在执行操作所在的指定作用域内启动协程， 而不是像通常使用线程（线程总是全局的）那样在 GlobalScope 中启动。

在我们的示例中，我们使用 runBlocking 协程构建器将 main 函数转换为协程。 包括 runBlocking 在内的每个协程构建器都将 CoroutineScope 的实例添加到其代码块所在的作用域中。 我们可以在这个作用域中启动协程而无需显式 join 之，因为外部协程（示例中的 runBlocking）直到在其作用域中启动的所有协程都执行完毕后才会结束。因此，可以将我们的示例简化为：

import kotlinx.coroutines.*
fun main() = runBlocking { // this: CoroutineScope
    launch { // 在 runBlocking 作用域中启动一个新协程
        delay(1000L)
        println("World!")
    }
    println("Hello,")
}

作用域构建器

除了由不同的构建器提供协程作用域之外，还可以使用 coroutineScope 构建器声明自己的作用域。它会创建一个协程作用域并且在所有已启动子协程执行完毕之前不会结束。

runBlocking 与 coroutineScope 可能看起来很类似，因为它们都会等待其协程体以及所有子协程结束。 主要区别在于，runBlocking 方法会阻塞当前线程来等待， 而 coroutineScope 只是挂起，会释放底层线程用于其他用途。 由于存在这点差异，runBlocking 是常规函数，而 coroutineScope 是挂起函数。

import kotlinx.coroutines.*
fun main() = runBlocking { // this: CoroutineScope
    launch {
        delay(200L)
        println("Task from runBlocking")
    }
    coroutineScope { // 创建一个协程作用域
        launch {
            delay(500L)
            println("Task from nested launch")
        }
        delay(100L)
        println("Task from coroutine scope") // 这一行会在内嵌 launch 之前输出
    }
    println("Coroutine scope is over") // 这一行在内嵌 launch 执行完毕后才输出
}

请注意，（当等待内嵌 launch 时）紧挨“Task from coroutine scope”消息之后， 就会执行并输出“Task from runBlocking”——尽管 coroutineScope 尚未结束。

提取函数重构

我们来将 launch { …… } 内部的代码块提取到独立的函数中。当你对这段代码执行“提取函数”重构时，你会得到一个带有 suspend 修饰符的新函数。 这是你的第一个挂起函数。在协程内部可以像普通函数一样使用挂起函数， 不过其额外特性是，同样可以使用其他挂起函数（如本例中的 delay）来挂起协程的执行。

import kotlinx.coroutines.*
fun main() = runBlocking {
    launch { doWorld() }
    println("Hello,")
}
// 这是你的第一个挂起函数
suspend fun doWorld() {
    delay(1000L)
    println("World!")
}

但是如果提取出的函数包含一个在当前作用域中调用的协程构建器的话，该怎么办？ 在这种情况下，所提取函数上只有 suspend 修饰符是不够的。为 CoroutineScope 写一个 doWorld 扩展方法是其中一种解决方案，但这可能并非总是适用，因为它并没有使 API 更加清晰。 惯用的解决方案是要么显式将 CoroutineScope 作为包含该函数的类的一个字段， 要么当外部类实现了 CoroutineScope 时隐式取得。 作为最后的手段，可以使用 CoroutineScope(coroutineContext)，不过这种方法结构上不安全， 因为你不能再控制该方法执行的作用域。只有私有 API 才能使用这个构建器。

协程很轻量
import kotlinx.coroutines.*
fun main() = runBlocking {
    repeat(100_000) { // 启动大量的协程
        launch {
            delay(5000L)
            print(".")
        }
    }
}

它启动了 10 万个协程，并且在 5 秒钟后，每个协程都输出一个点。
现在，尝试使用线程来实现。会发生什么？（很可能你的代码会产生某种内存不足的错误）

全局协程像守护线程
以下代码在 GlobalScope 中启动了一个长期运行的协程，该协程每秒输出“I'm sleeping”两次，之后在主函数中延迟一段时间后返回。
GlobalScope.launch {
    repeat(1000) { i ->
        println("I'm sleeping $i ...")
        delay(500L)
    }
}
delay(1300L) // 在延迟后退出

你可以运行这个程序并看到它输出了以下三行后终止：
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
在 GlobalScope 中启动的活动协程并不会使进程保活。它们就像守护线程。

取消与超时

https://www.kotlincn.net/docs/reference/coroutines/cancellation-and-timeouts.html

在一个长时间运行的应用程序中，你也许需要对你的后台协程进行细粒度的控制。 比如说，一个用户也许关闭了一个启动了协程的界面，那么现在协程的执行结果已经不再被需要了，这时，它应该是可以被取消的。 该 launch 函数返回了一个可以被用来取消运行中的协程的 Job：

val job = launch {
    repeat(1000) { i ->
        println("job: I'm sleeping $i ...")
        delay(500L)
    }
}
delay(1300L) // 延迟一段时间
println("main: I'm tired of waiting!")
job.cancel() // 取消该作业
job.join() // 等待作业执行结束
println("main: Now I can quit.")

一旦 main 函数调用了 job.cancel，我们在其它的协程中就看不到任何输出，因为它被取消了。 这里也有一个可以使 Job 挂起的函数 cancelAndJoin 它合并了对 cancel 以及 join 的调用。

取消是协作的

协程的取消是 协作 的。一段协程代码必须协作才能被取消。 所有 kotlinx.coroutines 中的挂起函数都是 可被取消的 。它们检查协程的取消， 并在取消时抛出 CancellationException。 然而，如果协程正在执行计算任务，并且没有检查取消的话，那么它是不能被取消的，就如如下示例代码所示：

val startTime = System.currentTimeMillis()
val job = launch(Dispatchers.Default) {
    var nextPrintTime = startTime
    var i = 0
    while (i < 5) { // 一个执行计算的循环，只是为了占用 CPU
        // 每秒打印消息两次
        if (System.currentTimeMillis() >= nextPrintTime) {
            println("job: I'm sleeping ${i++} ...")
            nextPrintTime += 500L
        }
    }
}
delay(1300L) // 等待一段时间
println("main: I'm tired of waiting!")
job.cancelAndJoin() // 取消一个作业并且等待它结束
println("main: Now I can quit.")

运行示例代码，并且我们可以看到它连续打印出了“I'm sleeping”，甚至在调用取消后， 作业仍然执行了五次循环迭代并运行到了它结束为止。

使计算代码可取消

我们有两种方法来使执行计算的代码可以被取消。第一种方法是定期调用挂起函数来检查取消。对于这种目的 yield 是一个好的选择。 另一种方法是显式的检查取消状态。让我们试试第二种方法。

将前一个示例中的 while (i < 5) 替换为 while (isActive) 并重新运行它。

val startTime = System.currentTimeMillis()
val job = launch(Dispatchers.Default) {
    var nextPrintTime = startTime
    var i = 0
    while (isActive) { // 可以被取消的计算循环
        // 每秒打印消息两次
        if (System.currentTimeMillis() >= nextPrintTime) {
            println("job: I'm sleeping ${i++} ...")
            nextPrintTime += 500L
        }
    }
}
delay(1300L) // 等待一段时间
println("main: I'm tired of waiting!")
job.cancelAndJoin() // 取消该作业并等待它结束
println("main: Now I can quit.")

你可以看到，现在循环被取消了。isActive 是一个可以被使用在 CoroutineScope 中的扩展属性。

在 finally 中释放资源

我们通常使用如下的方法处理在被取消时抛出 CancellationException 的可被取消的挂起函数。比如说，try {……} finally {……} 表达式以及 Kotlin 的 use 函数一般在协程被取消的时候执行它们的终结动作：

val job = launch {
    try {
        repeat(1000) { i ->
            println("job: I'm sleeping $i ...")
            delay(500L)
        }
    } finally {
        println("job: I'm running finally")
    }
}
delay(1300L) // 延迟一段时间
println("main: I'm tired of waiting!")
job.cancelAndJoin() // 取消该作业并且等待它结束
println("main: Now I can quit.")

运行不能取消的代码块

在前一个例子中任何尝试在 finally 块中调用挂起函数的行为都会抛出 CancellationException，因为这里持续运行的代码是可以被取消的。通常，这并不是一个问题，所有良好的关闭操作（关闭一个文件、取消一个作业、或是关闭任何一种通信通道）通常都是非阻塞的，并且不会调用任何挂起函数。然而，在真实的案例中，当你需要挂起一个被取消的协程，你可以将相应的代码包装在 withContext(NonCancellable) {……} 中，并使用 withContext 函数以及 NonCancellable 上下文，见如下示例所示：

val job = launch {
    try {
        repeat(1000) { i ->
            println("job: I'm sleeping $i ...")
            delay(500L)
        }
    } finally {
        withContext(NonCancellable) {
            println("job: I'm running finally")
            delay(1000L)

            println("job: And I've just delayed for 1 sec because I'm non-cancellable")

        }
    }
}
delay(1300L) // 延迟一段时间
println("main: I'm tired of waiting!")
job.cancelAndJoin() // 取消该作业并等待它结束
println("main: Now I can quit.")

超时

在实践中绝大多数取消一个协程的理由是它有可能超时。 当你手动追踪一个相关 Job 的引用并启动了一个单独的协程在延迟后取消追踪，这里已经准备好使用 withTimeout 函数来做这件事。 来看看示例代码：

withTimeout(1300L) {
    repeat(1000) { i ->
        println("I'm sleeping $i ...")
        delay(500L)
    }
}

I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...

Exception in thread "main" kotlinx.coroutines.TimeoutCancellationException: Timed out waiting for 1300 ms

withTimeout 抛出了 TimeoutCancellationException，它是 CancellationException 的子类。 我们之前没有在控制台上看到堆栈跟踪信息的打印。这是因为在被取消的协程中 CancellationException 被认为是协程执行结束的正常原因。 然而，在这个示例中我们在 main 函数中正确地使用了 withTimeout。

由于取消只是一个例外，所有的资源都使用常用的方法来关闭。 如果你需要做一些各类使用超时的特别的额外操作，可以使用类似 withTimeout 的 withTimeoutOrNull 函数，并把这些会超时的代码包装在 try {...} catch (e: TimeoutCancellationException) {...} 代码块中，而 withTimeoutOrNull 通过返回 null 来进行超时操作，从而替代抛出一个异常

val result = withTimeoutOrNull(1300L) {
    repeat(1000) { i ->
        println("I'm sleeping $i ...")
        delay(500L)
    }
    "Done" // 在它运行得到结果之前取消它
}
println("Result is $result")

I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
Result is null

The timeout event in withTimeout is asynchronous with respect to the code running in its block and may happen at any time, even right before the return from inside of the timeout block. Keep this in mind if you open or acquire some resource inside the block that needs closing or release outside of the block.

https://www.kotlincn.net/docs/reference/coroutines/composing-suspending-functions.html

在概念上，async 就类似于 launch。它启动了一个单独的协程，这是一个轻量级的线程并与其它所有的协程一起并发的工作。不同之处在于 launch 返回一个 Job 并且不附带任何结果值，而 async 返回一个 Deferred —— 一个轻量级的非阻塞 future， 这代表了一个将会在稍后提供结果的 promise。你可以使用 .await() 在一个延期的值上得到它的最终结果， 但是 Deferred 也是一个 Job，所以如果需要的话，你可以取消它。

val time = measureTimeMillis {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    println("The answer is ${one.await() + two.await()}")
}
println("Completed in $time ms")

可选的，async 可以通过将 start 参数设置为 CoroutineStart.LAZY 而变为惰性的。 在这个模式下，只有结果通过 await 获取的时候协程才会启动，或者在 Job 的 start 函数调用的时候。

val time = measureTimeMillis {
    val one = async(start = CoroutineStart.LAZY) { doSomethingUsefulOne() }
    val two = async(start = CoroutineStart.LAZY) { doSomethingUsefulTwo() }
    // 执行一些计算
    one.start() // 启动第一个
    two.start() // 启动第二个
    println("The answer is ${one.await() + two.await()}")
}
println("Completed in $time ms")

async 风格的函数

我们可以定义异步风格的函数来 异步 的调用 doSomethingUsefulOne 和 doSomethingUsefulTwo 并使用 async 协程建造器并带有一个显式的 GlobalScope 引用。 我们给这样的函数的名称中加上“……Async”后缀来突出表明：事实上，它们只做异步计算并且需要使用延期的值来获得结果。

// somethingUsefulOneAsync 函数的返回值类型是 Deferred<Int>
fun somethingUsefulOneAsync() = GlobalScope.async {
    doSomethingUsefulOne()
}

// somethingUsefulTwoAsync 函数的返回值类型是 Deferred<Int>
fun somethingUsefulTwoAsync() = GlobalScope.async {
    doSomethingUsefulTwo()
}

注意，这些 xxxAsync 函数不是 挂起 函数。它们可以在任何地方使用。 然而，它们总是在调用它们的代码中意味着异步（这里的意思是 并发 ）执行。

// 注意，在这个示例中我们在 `main` 函数的右边没有加上 `runBlocking`
fun main() {
    val time = measureTimeMillis {
        // 我们可以在协程外面启动异步执行
        val one = somethingUsefulOneAsync()
        val two = somethingUsefulTwoAsync()
        // 但是等待结果必须调用其它的挂起或者阻塞
        // 当我们等待结果的时候，这里我们使用 `runBlocking { …… }` 来阻塞主线程
        runBlocking {
            println("The answer is ${one.await() + two.await()}")
        }
    }
    println("Completed in $time ms")
}

这种带有异步函数的编程风格仅供参考，因为这在其它编程语言中是一种受欢迎的风格。在 Kotlin 的协程中使用这种风格是强烈不推荐的， 原因如下所述。

考虑一下如果 val one = somethingUsefulOneAsync() 这一行和 one.await() 表达式这里在代码中有逻辑错误， 并且程序抛出了异常以及程序在操作的过程中中止，将会发生什么。 通常情况下，一个全局的异常处理者会捕获这个异常，将异常打印成日记并报告给开发者，但是反之该程序将会继续执行其它操作。但是这里我们的 somethingUsefulOneAsync 仍然在后台执行， 尽管如此，启动它的那次操作也会被终止。这个程序将不会进行结构化并发，如下一小节所示。

使用 async 的结构化并发

让我们使用使用 async 的并发这一小节的例子并且提取出一个函数并发的调用 doSomethingUsefulOne 与 doSomethingUsefulTwo 并且返回它们两个的结果之和。 由于 async 被定义为了 CoroutineScope 上的扩展，我们需要将它写在作用域内，并且这是 coroutineScope 函数所提供的：

suspend fun concurrentSum(): Int = coroutineScope {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    one.await() + two.await()
}

这种情况下，如果在 concurrentSum 函数内部发生了错误，并且它抛出了一个异常， 所有在作用域中启动的协程都会被取消。

取消始终通过协程的层次结构来进行传递：
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    try {
        failedConcurrentSum()
    } catch(e: ArithmeticException) {
        println("Computation failed with ArithmeticException")
    }
}

suspend fun failedConcurrentSum(): Int = coroutineScope {
    val one = async<Int> {
        try {
            delay(Long.MAX_VALUE) // 模拟一个长时间的运算
            42
        } finally {
            println("First child was cancelled")
        }
    }
    val two = async<Int> {
        println("Second child throws an exception")
        throw ArithmeticException()
    }
    one.await() + two.await()
}

请注意，如果其中一个子协程（即 two）失败，第一个 async 以及等待中的父协程都会被取消：

https://www.kotlincn.net/docs/reference/coroutines/coroutine-context-and-dispatchers.html

协程上下文与调度器

协程总是运行在一些以 CoroutineContext 类型为代表的上下文中，它们被定义在了 Kotlin 的标准库里。

协程上下文是各种不同元素的集合。其中主元素是协程中的 Job， 我们在前面的文档中见过它以及它的调度器，而本文将对它进行介绍。

http://192.168.1.13/svn/CMS/CMS_PRD/branchs/Dev_Master <http://192.168.1.13/svn/CMS/CMS_PRD/branchs/Dev_Master>  /data

package com.kt

import kotlin.text.*
import java.awt.Rectangle
import java.io.File
import java.nio.file.Files
import java.nio.file.Paths
import java.math.BigDecimal
import java.awt.event.MouseEvent

// 不需要 class AAA_MyKT， 有 反而错的。。
fun main()
{
println("hi,world")
println(po)                        // 可以 输出 [2,3]
print(pa);println();

println("asd : ${pa}");
println("asd : $pa");                // both can

//                for (i in 1..100) { …… }  // 闭区间：包含 100
//for (i in 1 until 100) { …… } // 半开区间：不包含 100
//for (x in 2..10 step 2) { …… }
//for (x in 10 downTo 1) { …… }
//if (x in 1..10) { …… }

for (i in 1..10)
{
println(i);
}

val map = mapOf("a" to 1, "b" to 2, "c" to 3)

var am = mapOf("a" to 2, "b" to 3, "c" to 4)

//                map = mapOf("z" to 2)                // Val cannot be reassigned

am = mapOf("z" to 3, "q" to 5)
println(map)
println(am)

println(transform("Blue"))

val result = try {
//        count()
transform("B123lue")
} catch (e: IllegalArgumentException) {
//                        throw IllegalStateException(e)
}

println(result)                                // kotlin.Unit

val im2 = listOf("a","吧","asd")
for (i in im2)
{
print(i + ",,,")
}
println()
for (i in im2.indices)
{
println("asd $i is ${im2[i]}")
}

var i:Int
i = -1
while (++i < im2.size)
{
println("@ $i is ${im2[i]}")
}

val x = 10
val y = 9;
if (x in 1..y+1)                        // true
println("in ")

// .. 2个点 是一个整体。不能空格。。
if (-1 !in 0 .. im2.lastIndex)        // true
println("not in")

if (im2.size !in im2.indices)        // true                // 集合。 上面的 0..9这种也是 集合?还是 一个range类？

println("asdasds not in")

// 有点cpp的味道了。。。 岂不是 能自己 重写  其他类 +-*/ ？
// public final operator fun plus(other: Byte): Int defined in kotlin.Int
for (x in 1..5)
print(x.toString() + ", ")                        // !!!!!
println()
for (x in 1..10 step 2)
print(x.toString() + ".. ")
println()
for (x in 9 downTo 1 step 3)
print(x.toString() + "; ")
println()

//                for (x in 1 upTo 10 step 3)                // meiyou  upTo..

for (im in im2)
println(im)

// 有一个为true后，就退出 when了。   等于 每个case 后 都跟了一个 break
// 这个 和 下面的 transform方法中的 when(color) 是 类似的吗？
// 这个怎么转为 下面的 那种？  好像不行。。
// 不过下面的那种  可以转为 现在这种吧。。
when {
    "as1d" in im2 -> println("asd in a")
"a1" in im2 -> println("a in im2")
'a' in im2 -> println("ch in ")
else -> println("nnnnnnn")                        // 等于default。
}

// keyi de .
// transform 里的 when  应该是 这个的 简写形式吧。
var col:String = "re1d"
when {
col.equals("red") -> println("a1zaaaa")
col.equals("fasd") -> println("a666a1111")
else -> println("zzzz")
}

// ... 这个是 java.awt.Rectangle ，， java的， 还真可以 无缝衔接。。
val dasd = Rectangle(5,2)
println(dasd.toString())

var x32 :Any = 2                        // 类型空 或 :Int 都不行，，空会自动推导为Int的， 下面 is String 是dead code,, 编译失败的

when (x32)
{
is String -> println("aaaa")
is Int -> println("zzz")                // shuchu
else -> println("cvvv")
}

//                var str:String = "d"
//                when (str)
//                        {

//                        .length == 2 -> println("aaa")                        // 不行的。。

//                }

// 不能只写 Map 、Map和泛型 是一起的。
var map1:Map<String, Int> = mapOf("a" to 1, "b" to 2, "c" to 3)

for ((k2,v) in map1)
{
println("$k2 -> $v")
}

// 3个 map["z"] 都有问题。。。
val map3 : Map<String, Int> = mapOf("a" to 2)
//                map3["z"] = 4
println(map3["z"])                // null

var map4 : Map<String, Int> =mapOf("a" to 2)                // Int>=  这里会优先解析为   大于等于。。

//                map4["d"] = 1
println(map4["d"])

val va : Int = 4
val map21 = mapOf("a" to 1, "b" to 2, "c" to 3)
println(map21["key"])
//                map21["key"] = va
println(map21["a"])

// 这个有毒，， eclipse 会报错 stackoverflow。。。。
//                val lzy:  String by lazy {
//                        println("start init ... ")
//                        "asd"
//                }

println("z==========zzzz")
//                println(lzy)

val str312: String = "asd";
str312.spaceToCamelCase();

//---------------------                 singneton.class 反编译出来的。。。而且这个是 单独的class。

//                // Compiled from AAA_MyKT.kt (version 1.6 : 50.0, super bit)
//public final class com.kt.singleton {
//
//  // Field descriptor #6 Ljava/lang/String;
//  @org.jetbrains.annotations.NotNull
//  private static final java.lang.String name;
//
//  // Field descriptor #11 Lcom/kt/singleton;
//  public static final com.kt.singleton INSTANCE;
//
//  // Method descriptor #9 ()Ljava/lang/String;
//  @org.jetbrains.annotations.NotNull
//  public final java.lang.String getName();
//
//  // Method descriptor #13 ()V
//  private singleton();
//}

println(singleton.name)

// 估计上面生成的class文件 是 getName？   在kotlin这个语言中，是没有getName的，只不过翻译成jvm字节码后，加了。。确实，哪写过 cmpxchg，push，move呢。

//                Unresolved reference: getName
//                println(singleton.getName())

a1()

a2()
a3()
a5()

a6()

a7(21)

val ar = arrayOfMinusOnes(4)
println(ar.size)
for (i in ar.indices)
{
// ... string + int 可以，   int + string 不行。。。
println("${ar[i]}  " + i)
}

test2()

//   不行啊，  3个属性 没有办法解析。。。 但是  有这3个属性徳。。  估计要 kotlin的 类吧，， 现在 这个Rectangle 是 java 的类。

//                val myRectangle = Rectangle().apply {
//                        length = 4
//                        breadth = 5
//                        color = 0xFAFAFA
//                }
//                println("asd11===dddddd=")
//                println(myRectangle.toString())
//                println("asd11====")

test31()

asd()

// chg
var a = 1
var b = 2

a = b . also { b = a }                        // a=b  同时  b=a     also是表示后面是一个方法吧。lambda     . also 可以分开

println(a)
println(b)

//                calcTaxes()                // yes  Exception in thread "main" kotlin.NotImplementedError: An operation is not implemented: Waiting for feedback from accounting

}

// 那就 没有 int 这种变量了，只有Integer 了。。
//val b: Boolean? = ……
//if (b == true) {
//    ……
//} else {
//    // `b` 是 false 或者 null
//}

//Kotlin 的标准库有一个 TODO() 函数，该函数总是抛出一个 NotImplementedError。 其返回类型为 Nothing，因此无论预期类型是什么都可以使用它。 还有一个接受原因参数的重载：

fun calcTaxes(): BigDecimal = TODO("Waiting for feedback from accounting")

//  public final class Gson {
//     ……

//     public <T> T fromJson(JsonElement json, Class<T> classOfT) throws JsonSyntaxException {

//     ……

// inline ....

//inline fun <reified T: Any> Gson.fromJson(json: JsonElement): T = this.fromJson(json, T::class.java)

fun asd(){
//        AVScanner.ini                        / 就是从本盘 盘符 开始的。。   这个文件直接 在 c盘下的
val stream = Files.newInputStream(Paths.get("/AVScanner.ini"))

//  workspace\kotlintest\bin     最前面不能加 / ,,
//        val stream = Files.newInputStream(Paths.get("bin/file.txt"))

// 取得是 工程下的 file.txt      xxx/workspace/kotlintest/file.txt
//        val stream = Files.newInputStream(Paths.get("file.txt"))
stream.buffered().reader().use { reader ->
    println(reader.readText())
}
}

//  public static final void test31()
//  {

//    Customer localCustomer1 = new Customer("asddddd", "@@@"); int j = 0; int k = 0; Customer $this$apply = localCustomer1; int $i$a$-apply-AAA_MyKTKt$test31$cus$1 = 0;

//    $this$apply.setName("asd");
//
//    Customer cus = localCustomer1;
//
//    int i = 0; System.out.println(cus);
//  }
fun test31(){
// 怎么像py一样， 指定 形参名。。
val cus = Customer(email="@@@", name = "asddddd").apply{
name="asd"
//                        email = "zzzz"
}
println(cus)
}

// jd-gui 里 main 方法中 代码没有全部翻译出来。。不知道为什么。。  所以另起一个方法。
//  public static final void test2()
//  {
//    int[] md = new int[3];

//    int i = 0; int j = 0; int[] $this$with = md; int $i$a$-with-AAA_MyKTKt$test2$1 = 0;

//    String str1 = "=============="; int m = 0; System.out.println(str1);
//    ArraysKt.fill$default($this$with, -1, 0, 0, 6, null);
//    int k = 0; for (m = md.length; k < m; i++)
//    {

//      String str3 = String.valueOf(md[i]); int n = 0; System.out.println(str3);

//    }
//    int i = md.length; m = 0; System.out.println(i);
//    String str2 = "=============="; m = 0; System.out.println(str2);
//  }
fun test2(){
val md = IntArray(3)
with (md) {
println("==============")
fill(-1)
for (i in md.indices)
{
println("${md[i]}")
}
println(md.size)
println("==============")
}
}

//对一个对象实例调用多个方法 （with） 。 。。 就是 不需要链式， 而且  setter是自动生成的， 这个返回void
// 没有方法实现啊， 要么实现，要么abstract，，一旦abstract，class 也必须abstract， 下面就没有办法 创建了。
//class Turtle {
//    fun penDown()
//    fun penUp()
//    fun turn(degrees: Double)
//    fun forward(pixels: Double)
//}
//
//val myTurtle = Turtle()
//with(myTurtle) { // 画一个 100 像素的正方形
//    penDown()
//    for (i in 1..4) {
//        forward(100.0)
//        turn(90.0)
//    }
//    penUp()
//}

fun theAnswer1() = 42
fun theAnswer2(): Int {
    return 42
}

// ...  minus one，，还真是  负1 。。。
fun arrayOfMinusOnes(size: Int): IntArray {
    return IntArray(size).apply { fill(-1) }
}

fun a7(p : Int)
{
val r = if (p == 1) {
"one"
} else if (p == 2) {"two"}
else "thrww"
println(r)
}

fun a6()
{
val r = try {
getLen(1)
} catch (e : ArithmeticException) {
throw IllegalStateException(e)
}
println("asdasd : " + r)
}

fun a5()
{
val value = 1

        value ?. let {
 println("11not null")
}

//        val mapped = value?.let { transformValue(it) } ?: defaultValue

//        val v = null

//        v  ?: let {                                                // let 报错， unresolved experiso

//                println("is null")
//        }

}

fun a4()
{
val map2 = mapOf("a" to 1)

//        map2.remove("a")                                // 没有remove，，，怎么才能 获得 空map   空List

//        val mainEmail = emails.firstOrNull() ?: ""
}

// tmp开头的全是 红色的。。  估计预先压栈了一些东西。。
//  public static final void a3()
//  {
//    Map map2 = MapsKt.mapOf(TuplesKt.to("a", Integer.valueOf(1)));

//    Integer tmp24_21 = ((Integer)map2.get("b")); if (tmp24_21 != null) tmpTernaryOp = tmp24_21;

//  }
fun a3(){
val map2 = mapOf("a" to 1)
val asd = map2["b"] ?: "is null"
println(asd)                        // is null
}

// ?.   也是一个整体。。。    .不是调用方法符。。  一个点 非null， 2个点 null。
// ?:  是一个整体。
// ... 反编译出来 和 a1 的一样。。。。。。
//  public static final void a2()
//  {
//    File[] f = new File("zzzzqwd").listFiles();
//    if (f != null) tmpTernaryOp = Integer.valueOf(f.length);
//  }
fun a2(){
val f = File("zzzzqwd").listFiles();
println(f ?. size ?: "empty")
}

//... class 用 jd-gui 反编译出来： 。。。 并没有 throw,,直接运行 java，确实空指针。。现在的问题就是 ? 的代码在哪里体现？
//  public static final void a1()
//  {
//    File[] f = new File("asd").listFiles();
//    int i = f.length; int j = 0; System.out.println(i);
//  }
//。。。。 估计上面的是 之前 删掉？ 的 代码。。。
//  public static final void a1()
//  {
//    File[] f = new File("asd").listFiles();
//    if (f != null) tmpTernaryOp = Integer.valueOf(f.length);
//  }
//... 唯一的问题是，，println 哪里去了？
//。。 而且  jd-gui中  tmpTernaryOp = 是红色的，代表了什么意思？   估计代表 下个print的 东西？
// Ternary  三元的，三个一组。
// 好像也是 ? 就是 对应 ?: 这个运算。

fun a1(){
val f = File("asd").listFiles()
println(f?.size)                        // null
        // 不写？,   就是 java.lang.NullPointerException...
}

// 不能是local变量。   object 关键字，所以小写。
object singleton {
val name = "wo"
}
fun String.spaceToCamelCase() {
    println("aa how to do it ?")
}

//单表达式函数与其它惯用法一起使用能简化代码，例如和 when 表达式一起使用：
fun transform2(color: String): Int = when (color) {
    "Red" -> 0
    "Green" -> 1
    "Blue" -> 2
    else -> throw IllegalArgumentException("Invalid color param value")
}

// 返回when 表达式。
fun transform(color: String): Int {
    return when (color) {
        "Red" -> 0
        "Green" -> 1
        "Blue" -> 2
        else -> throw IllegalArgumentException("Invalid color param value")
    }
}

// val , 不是 var，，， 这里val 想表达什么？ 感觉 var更好啊。
// 估计是 形参name 不可变。。。。但是 这个并不是 方法吧，这里应该是 类属性的 描述。
data class Customer(var name: String, val email: String)
// ok, 知道了。。。md
                                    //为 Customer 类提供以下功能：
                                    //所有属性的 getters （对于 var 定义的还有 setters）
                                    //equals()
                                    //hashCode()
                                    //toString()
                                    //copy()

                                    //所有属性的 component1()、 component2()……等等（参见数据类）

// 方法的 形参 都是不写 val / var 的。
// 写val IDE报错，   没有?: 这种三目。。。
//fun max( a: Int,  b: Int) = (a > b ? a : b);

// 不能有 var。
fun max(a: Int, b:Int)=if (a > b) a else b

fun foo(a: Int = 1, b : Int = 2) = a

// {{}} , bushi (())
var list = listOf(-1,2,3,-4)
val po = list.filter{x1->x1>0}

// it 是 固定的 用法，不能 it2 这种自定义的。
//var pe = list.filter{ it2 > 0 }
var pa = list.filter{ it > 0 }

// 返回类型自动推导。
fun sum(a: Int, b:Int) = a+b

fun sum2(a:Int,b:Int) :Int = a+b

// 只有一个变量时，{} 可以不写。
// int 怎么和stirng 连接。。  int怎么转string？
fun prt(a:Int,b:Int) :Unit = print("asd ${a} and $b is ${a+b}")

// Unit 可以省略
fun prrr(a:Int,b:Int) = print(a+b)

val dsw = 2

fun asdf(){
    var df1f : Int = 2
    df1f = 3
}
// 不行了。。
fun dwf() {
    val c: Int  // 如果没有初始值类型不能省略
//    c = 3       // 明确赋值
}

val dd12:Int = 2
//dd12 = 3

//.....在方法里可以 声明 但不 初始化。
// 全局的话，必须 声明时初始化，  。。ok，全局的话，估计 是static， static。 分开写的话，赋值的语义不够清晰。

/*
 /*
 */
 */

var a = 1
// 模板中的简单名称：
val s1 = "a is $a"

//a = 2                        //        Expecting a top level declaration . ... 需要在fun中。

// 模板中的任意表达式：
val s2 = "${s1.replace("is", "was")}, but now is $a"

//当某个变量的值可以为 null 的时候，必须在声明处的类型后添加 ? 来标识该引用可为空。
fun par(s:String) :Int? {

//        return parseInt(s)                                // ... 这个方法需要什么包？    好像是 自定义的。。擦，难道没有 官方的转换方法吗？

print("asd")
return 4
123                        // 编译没有错。。。只是一个 warning
}

fun getLen(s:Any) :Int?
{
if (s is String)
return s.length
// 在离开类型检测分支后，`obj` 仍然是 `Any` 类型
return null
}

// 。。这个是把 =后面的 表达式计算出来，然后 return 出去。。。。。 就是 有默认的 return了， 不需要再 return s.length 了。

fun getLen23(s:Any) :Int? = if (s is String)  s.length else  null

//    // `obj` 在 `&&` 右边自动转换成 `String` 类型
//    if (obj is String && obj.length > 0) {
//      return obj.length
//    }

//
//通常，一个类的内容按以下顺序排列：
//
//属性声明与初始化块
//次构造函数
//方法声明
//伴生对象

//通常不鼓励使用多个词的名称，但是如果确实需要使用多个词，可以将它们连接在一起或使用驼峰风格（org.example.myProject）。

//类与对象的名称以大写字母开头并使用驼峰风格：
open class DeclarationProcessor { /*……*/ }
object EmptyDeclarationProcessor : DeclarationProcessor() { /*……*/ }
// 。。 open 是什么意思。。。
// object 应该是指  EmptyDeclarationProcessor 是一个对象吧，， 但是为什么 大写。。好吧， kotlin的规范。

//函数、属性与局部变量的名称以小写字母开头、使用驼峰风格而不使用下划线：
//例外：用于创建类实例的工厂函数可以与抽象返回类型具有相同的名称
interface Foo { /*……*/ }
class FooImpl : Foo { /*……*/ }
fun Foo(): Foo { return FooImpl() }

//当且仅当在测试中，可以使用反引号括起来的带空格的方法名。 （请注意，Android 运行时目前不支持这样的方法名。）测试代码中也允许方法名使用下划线。
// @Test 是哪个包的，，，junit的不行。。 估计得 kunit。(kotlin unit？？。。)
class MyTestCase {
//     @Test fun `ensure everything works`() { /*...*/ }
//     @Test fun ensureEverythingWorks_onAndroid() { /*...*/ }
}

//常量名称（标有 const 的属性，或者保存不可变数据的没有自定义 get 函数的顶层/对象 val 属性）应该使用大写、下划线分隔的名称：
const val MAX_COUNT = 8
val USER_NAME_FIELD = "UserName"

val mutableCollection: MutableSet<String> = HashSet()

enum class Color { RED, GREEN }

// private val _
//如果一个类有两个概念上相同的属性，一个是公共 API 的一部分，另一个是实现细节，那么使用下划线作为私有属性名称的前缀
//class C {
//    private val _elementList = mutableListOf<Element>()
//
//    val elementList: List<Element>
//         get() = _elementList
//}

//使用 4 个空格缩进。不要使用 tab。
//对于花括号，将左花括号放在结构起始处的行尾，而将右花括号放在与左括结构横向对齐的单独一行

//在 Kotlin 中，分号是可选的，因此换行很重要。语言设计采用 Java 风格的花括号格式，如果尝试使用不同的格式化风格，那么可能会遇到意外的行为。

//在二元操作符左右留空格（a + b）。例外：不要在“range to”操作符（0..i）左右留空格。
//不要在一元运算符左右留空格（a++）
//在控制流关键字（if、 when、 for 以及 while）与相应的左括号之间留空格。
//不要在主构造函数声明、方法声明或者方法调用的左括号之前留空格。

class A(val x: Int)

fun foo(x: Int) {/* . .  */ }

fun bar() {
    foo(1)
}

//绝不在 (、 [ 之后或者 ]、 ) 之前留空格
//绝不在. 或者 ?. 左右留空格：foo.bar().filter { it > 2 }.joinToString(), foo?.bar()

//不要在用于指定类型参数的尖括号前后留空格：class Map<K, V> { …… }

//不要在 :: 前后留空格：Foo::class、 String::length

//不要在用于标记可空类型的 ? 前留空格：String?

//在以下场景中的 : 之前留一个空格：
//当它用于分隔类型与超类型时；
//当委托给一个超类的构造函数或者同一类的另一个构造函数时；
//在 object 关键字之后。
//而当分隔声明与其类型时，不要在 : 之前留空格。
//
//在 : 之后总要留一个空格。

//对于多个接口，应该将超类构造函数调用放在首位，然后将每个接口应放在不同的行中：

//class Person(
//    id: Int,
//    name: String,
//    surname: String
//) : Human(id, name),
//    KotlinMaker { /*……*/ }

//如果一个声明有多个修饰符，请始终按照以下顺序安放
//public / protected / private / internal
//expect / actual
//final / open / abstract / sealed / const
//external
//override
//lateinit
//tailrec
//vararg
//suspend
//inner
//enum / annotation / fun // 在 `fun interface` 中是修饰符
//companion
//inline
//infix
//operator
//data

//将所有注解放在修饰符前
//@Named("Foo")
//private val foo: Foo

//除非你在编写库，否则请省略多余的修饰符（例如 public）。

//当对链式调用换行时，将 . 字符或者 ?. 操作符放在下一行，并带有单倍缩进
//val anchor = owner
//    ?.firstChild!!
//    .siblings(forward = true)
//    .dropWhile { it is PsiComment || it is PsiWhiteSpace }

//如果为 lambda 表达式分配一个标签，那么不要在该标签与左花括号之间留空格
//fun foo() {
//    ints.forEach lit@{
//        // ……
//    }
//}

//在多行的 lambda 表达式中声明参数名时，将参数名放在第一行，后跟箭头与换行符
//appendCommaSeparated(properties) { prop ->
//    val propertyValue = prop.get(obj)  // ……
//}

//如果参数列表太长而无法放在一行上，请将箭头放在单独一行：
//foo {
//   context: Context,
//   environment: Env
//   ->
//   context.configureEnv(environment)
//}

//尽可能省略分号

//字符串模版
//将简单变量传入到字符串模版中时不要使用花括号。只有用到更长表达式时才使用花括号。
//
//println("$name has ${children.size} children")

//优先使用不可变（而不是可变）数据。初始化后未修改的局部变量与属性，总是将其声明为 val 而不是 var 。

//优先声明带有默认参数的函数而不是声明重载函数。
//// 不良
//fun foo() = foo("a")
//fun foo(a: String) { /*……*/ }
//
//// 良好
//fun foo(a: String = "a") { /*……*/ }

//如果有一个在代码库中多次用到的函数类型或者带有类型参数的类型，那么最好为它定义一个类型别名
typealias MouseClickHandler = (Any, MouseEvent) -> Unit
//typealias PersonIndex = Map<String, Person>

//在简短、非嵌套的 lambda 表达式中建议使用 it 用法而不是显式声明参数。而在有参数的嵌套 lambda 表达式中，始终应该显式声明参数。

//避免在 lambda 表达式中使用多个返回到标签。请考虑重新组织这样的 lambda 表达式使其只有单一退出点。 如果这无法做到或者不够清晰，请考虑将 lambda 表达式转换为匿名函数。

//
//不要在 lambda 表达式的最后一条语句中使用返回到标签。

//当一个方法接受多个相同的原生类型参数或者多个 Boolean 类型参数时，请使用具名参数语法， 除非在上下文中的所有参数的含义都已绝对清楚。
//
//drawSquare(x = 10, y = 10, width = 100, height = 100, fill = true)

////优先使用 try、if 与 when 的表达形式
//return if (x) foo() else bar()
//
//return when(x) {
//    0 -> "zero"
//    else -> "nonzero"
//}
////优先选用上述代码而不是
//if (x)
//    return foo()
//else
//    return bar()
//
//when(x) {
//    0 -> return "zero"
//    else -> return "nonzero"
//}

//... return if else。。。。。 而不是 if return  else return，，， 就是 return只写一个。

//二元条件优先使用 if 而不是 when
//如果有三个或多个选项时优先使用 when。

//如果需要在条件语句中用到可空的 Boolean, 使用 if (value == true) 或 if (value == false) 检测

//优先使用高阶函数（filter、map 等）而不是循环。例外：forEach（优先使用常规的 for 循环， 除非 forEach 的接收者是可空的或者 forEach 用做长调用链的一部分。）

//使用 until 函数在一个开区间上循环
//for (i in 0..n - 1) { /*……*/ }  // 不良
//for (i in 0 until n) { /*……*/ }  // 良好
// .. until 不包含 n。

//优先使用字符串模板而不是字符串拼接。
//
//优先使用多行字符串而不是将 \n 转义序列嵌入到常规字符串字面值中。
//
//如需在多行字符串中维护缩进，当生成的字符串不需要任何内部缩进时使用 trimIndent，而需要内部缩进时使用 trimMargin

//在某些情况下，不带参数的函数可与只读属性互换。 虽然语义相似，但是在某种程度上有一些风格上的约定。
//
//底层算法优先使用属性而不是函数：
//
//不会抛异常
//计算开销小（或者在首次运行时缓存）
//如果对象状态没有改变，那么多次调用都会返回相同结果

//如果为一个类声明一个工厂函数，那么不要让它与类自身同名。优先使用独特的名称， 该名称能表明为何该工厂函数的行为与众不同。只有当确实没有特殊的语义时， 才可以使用与该类相同的名称

//class Point(val x: Double, val y: Double) {
//    companion object {
//        fun fromPolar(angle: Double, radius: Double) = Point(...)
//    }
//}
// companion

//返回平台类型表达式的公有函数/方法必须显式声明其 Kotlin 类型
//fun apiCall(): String = MyJavaApi.getProperty("name")

//任何使用平台类型表达式初始化的属性（包级别或类级别）必须显式声明其 Kotlin 类型
//class Person {
//    val name: String = MyJavaApi.getProperty("name")
//}

//使用平台类型表达式初始化的局部值可以有也可以没有类型声明：
//fun main() {
//    val name = MyJavaApi.getProperty("name")
//    println(name)
//}

//Kotlin 提供了一系列用来在给定对象上下文中执行代码块的函数：let、 run、 with、 apply 以及 also。

//在编写库时，建议遵循一组额外的规则以确保 API 的稳定性：
//总是显式指定成员的可见性（以避免将声明意外暴露为公有 API ）
//总是显式指定函数返回类型以及属性类型（以避免当实现改变时意外更改返回类型）
//为所有公有成员提供 KDoc 注释，不需要任何新文档的覆盖成员除外 （以支持为该库生成文档）

已使用 Microsoft OneNote 2016 创建。