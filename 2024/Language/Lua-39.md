Lua-39
2021年8月27日
11:00


- WowLua
- DevTools



[toc]


https://cloudwu.github.io/lua53doc/contents.html

已使用 Microsoft OneNote 2016 创建。


https://www.runoob.com/manual/lua53doc/


--------------------

# 网址

https://www.lua.org/manual/5.4/



# wow

2024-9-1

`nil`

## 基本类型

`print(type(5))`

- number
- string 。"" '' 都可以
- boolean
- function
- table (等于 C++的 数组与hash的混合)
- thread
- userdata


## 赋值

`x,y=3,5`

左侧多，则多余的是 nil


## 比较运算符

```lua
==
<
>
<=
>=
~=   // 不等于
```

运算符2侧的类型必须相同。

==，~= 适用于所有值
大于小于，只用于 数值 和 字符串



## 字符串(连接，和数值转换)

字符串是不可修改的。

`..` 用来连接 2个字符串。


`print("Number " .. 4)`
可以自动将 4 转为 字符串。


`x = tostring(14)` 显式转换

tostring 可以将 任意输入参数 转换为 字符串

`x = tonumber("1234")`

`x = tonumber("1e4")`


`x = 'asdasd " asdasd'`
`x = "asdasd ' asdasd"`

还可以使用转义


### 方括号

`x = [[this is a string ' "]]`

`x = [==[this is a ]][[]] string]==]`


### string长度

`x = #'string'`

`x = string.len('string')`





## bool 运算符

and,or,not

一些奇怪的特性
`print(true and 4)` 返回4
`print(true and "hi")` 返回hi


不是 false，nil 的都是 true


### 三目

`print(false and "hi" or "hello")`



## 变量

默认全局，除非 do end 中有 local




## 函数


```lua
hello = function()
    print("hi")
end
```

```text
> hi = function()
>> print("asd hi")
>> end
> hi
function: 00000000006f5f40
> hi()
asd hi
```

另一种定义方式
```lua
function hello()
    print("hi")
end
```


### local函数

```lua
local function hi()
    print("z")
end

local hi = function()
    print(1)
end
```


### 形参，返回值

```lua
convert1 = function(celsius)
    local a = (celsius * 1.8) + 32
    return a
end
```


函数也可以使用 ==,~= 来比较



## if

```lua
if (num == 7) then // 小括号不是必须的
    print("777")
end
```

下面都是有效的条件
- name
- type(name) == "string"
- (not anonymous_flag) and type(name) == "string"


```lua
if xxx then
    xxx
elseif uuu then
    yyy
else
    zzz
end
```

```lua
function greeting(name)
    if (type(name) == "string") then
        print("hi "..name)
    elseif (type(name) == "nil") then
        print("hi friend")
    else
        error("invalid name was entered")
    end
end
```


```lua
function greeting(name)
    if (type(name) == "string") then
        print("hi "..name)
    else
        if (type(name) == "nil") then
            print("hi friend")
        else    
            error("invalid name was entered")
        end
    end
end
```




## while

```lua
while xxx do
    yyy
end
```

```lua
function factorial(num)
    local a = 1
    while (num > 1) do 
        a = a * num
        num = num - 1
    end
    return a
end
```



### repeat/until

是 while 的一个变种

```lua
repeat
    yyy
until xxx
```

。。就是 do while


## for

```lua
for my_var = start_value, end_value, step_value do
    xxx
end
```

不提供 step_value时，默认 1

my_var 默认 local

==等于 end_value 时 也会执行==


```lua
for i = 3,1,-1 do
    print(i)
end
```

```lua
do
    local i = 3
    while (i >= 1) do
        print(i)
        i = i - 1
    end
end
```


start_value, stop_value, step_value 在起始时 计算一次。

```lua
upper = 3
for i=1,upper do
    print(i)
    upper = upper + 1
end
```
这个输出 1 2 3，不是 死循环。


```lua
i = 15
for i=1,3 do print(i) end   // 1 2 3 // 这个i是local 和上一行的不是同一个东西。
print(i)        // 15
```


获得控制变量的值, 需要额外的变量
```lua
upper = 10
do 
    local max
    for i = 1, upper do
        max = i
    end
    print(max)  // 10
end
```



## table

`a = {}`

表达式 `{}` 用于创建一个表，这里创建的是 空表

添加数据
```lua
a["asd"] = "zzzz"
a["qwe"] = "vvv"
```

输出
`a["asd"]`

不存在就是 nil

清除数据就是 手工设置为 nil
`a["asd"] = nil`



`a["asd"] = "aaaa"`
等价于
`a.asd = "aaaa"`  // 不能 a.123  即，不能加数字。 只能 string

只有当 关键字 以 字母 或 下划线开头时， 才可以 `a.xxx`



### 创建有内容的表

```lua
mytable = {
    ["asd"] = "asdz",
    ["qwe"] = "qqq",  // 这个 逗号 可加可不加，一般加
}
```

由于可以使用 点标识符索引，所以可以简化
```lua
mytable = {
    asd = "asdf",
    qwe = "qqq",
}
```


### table当做数组

```lua
tb = {
    "asd","asdw","zxc",
}
```

等价于
```lua
tb = {
    [1] = "asd",
    [2] = "asdw",
    [3] = "zxc",
}
```

。。数组从 1 开始


#### 数组长度

`#tb`

只计算元素个数
```lua
tb1 = {
    "111",
    "222",
    ["zz"] = "asd",
    ["vv"] = "qqq",
    "333",
}
print(#tb1)     // 3

for i = 1, #tb1 do
    print(tb1[i])       // 输出 111,222,333
end
```


#### 添加元素到数组

`tb[#tb+1] = "asd"`   // 书上没有+1，但是必须要的。


`table.insert(tb, [pos, ] value)`

pos不写，就是 尾部插入


#### 删除元素

`value = table.remove(tb [, pos])`

pos不写，就是 删除尾部元素


#### 排序

`table.sort(tb [, comp])`


### ==用名称空间使用表==

前面介绍的
- table.insert()
- table.remove()
- table.sort()

当函数以这种方式进行分组时，它们就被称为 名称空间中的一部分。  
本例中，table 就是 命名空间

命名空间提供了一种 对相关函数的逻辑分组， 其实现 和 lua中的表 一样简单。  
因为表支持函数值，所以 函数可以使用下面的方法进行访问
- table["insert"]()
- table["remove"]()
- table["sort"]()

新建一个库就是 编写函数，新建表，在表中设置 关键字-函数



---

创建命名空间

`util = {}`

添加已有的函数
`util.convert = convert1`

访问
`util.convert(1)`

新建函数并添加到命名空间
```lua
function util.factorial(num)
    local t = 1
    for i = 1, num do
        t = t * i
    end
    return t
end
```
等价于
```lua
util.factotial = function(num)
    xxx
end
```


### 表的面向对象编程

oop中，数据被描述成对象，它包含方法。 方法通过对象执行动作，或直接作用于对象上。


非 oop 的计数器
```lua
do 
    counter = 0

    function counter_get()
        return counter
    end

    function counter_inc()
        counter = counter + 1
    end
end
```


oop的计数器

```lua
counter = {
    count = 0
}

function counter.get(self)
    return self.count
end

function counter.inc(self)
    self.count = self.count + 1
end
```

```lua
print(counter.get(counter))
counter.inc(counter)
```

。。单例？ 怎么new一个？


创建一个新计数器
```lua
counter2 = {
    count = 15,
    get = counter.get,
    inc = counter.inc,
}
```

。。。那我得知道 counter 提供的方法。 确实，你都要用这个类了，你肯定要知道 提供的方法。 而且这里可以裁剪。
。。不算很oop啊， 没有继承。


#### 使用冒号调用对象方法

`对象:方法`

对象会作为方法的第一个参数。

`counter:get()` == `counter.get(counter)`

。。所以 counter.get 的 counter 是对象，而不是 类定义。


#### 使用冒号定义函数

`function tb:method()` == `function tb.method(self)`

```lua
counter = {
    counter = 0
}

function counter:get() 
    return self.count
end

function counter:inc()
    self.count = self.count + 1
end
```


#### 更好的计数器

```lua
do
    local function get(self)
        return self.count
    end

    local function inc(self)
        self.count = self.count + 1
    end

    function new_counter(value)
        if type(value) ~= "number" then
            value = 0
        end

        local obj = {
            count = value,
            get = get,
            inc = inc,
        }
        return obj
    end
end
```

这里提供了一个 new_counter 的全局函数， 参数是 计数器的初始值， 方法返回一个 新对象。

这是 一个典型的 工厂函数。

```lua
counter = new_counter()
print(counter:get())
```


## 元表

使用元表对 表进行扩展

lua中每个表 都可以 附带一个 元表(metatable)。

元表是 二级表格，提供了一些 关于 表应该如何被处理的附加信息。


---

元表就是一个简单的表，它存储了 和它有联系的 表的一些附加信息。它们能够被传递，能够附加到很多表，并且随时可以更改。  
为了重新定义表的行为，你必须创建一个元表，然后使用 setmetatable 将其附加到一个 表对象。

setmetatable有2个参数， tbl，mt  
一个返回值，是 tbl 的值， 将新建表 直接传递给 setmetatable时 有很有用。

下面的代码 建立了一些表，并给每个表 附上了 相同的元表
```lua
tb1 = {"aaaa"}
tb2 = {"bbbb"}
tb3 = {}
mt = {}
setmetatable(tb1, mt)
setmetatable(tb2, mt)
setmetatable(tb3, mt)
```

使用 getmetatable 来检查 元表是否已经成功附加。

`print(getmetatable(tb1) == mt)`  返回true，说明 元表已经被设置了。


### 元方法 metamethod

元方法就是一个函数，它存储在 元表中，且有 指定的 键

元方法有很多，参数各不相同。 每个元方法都是 双下划线开头。  

下面是 wow中 常用的 元方法
- __add
- __mul
- __div
- __sub
- __unm  取负/非
- __tostring
- __concat  连接
- __index  对表中不存在的键进行检索时的行为。。？ 应该是检索吧，下面的新建。 不，确实是 找不到所需的key的时候 调用这个方法，看下面的例子
- __newindex  用表中未预先设置的键检索时的行为 。。 当新建key时 调用这个方法。


---

__add
__sub
__mul
__div

定义了 基本运算

每个算术元方法 都有2个参数，并且 可以返回任何你想返回的值，但是记住
- 操作的结果可能作为 一个更大的算术表达式的一部分
- 如果从元方法 返回一个 非数字元素，就应该确保它能进行算术运算
- 如果返回 nil， 会 破坏它所在的 任何算术表达式，应该避免这种情况


下面的方法 定义了2个表的加运算，新表的开始是第一个表的数组部分，紧接着是第二个表的数组部分。

```lua
function mt.__add(a, b)
    local result = setmetatable({}, mt)

    for i=1,#a do
        table.insert(result, a[i])
    end

    for i=1,#b do
        table.insert(result, b[i])
    end

    return result
end
```

---

__unm

返回 取负值后的结果， 本例中，是 逆序

```lua
function mt.__unm(a)
    local result = setmetatable({}, mt)

    for i=#a,1,-1 do
        table.insert(result, a[i])
    end

    return result
end
```

`um = -tb1`


---

__tostring

有意义的输出

```lua
function mt.__tostring(tb)
    local result = "["

    for i=1,#tb do
        if i > 1 then 
            result = result .. ', '
        end
        result = result .. tostring(tb[i])
    end

    result = result .. "]"

    return result
end
```


---

__concat

连接表

```lua
mt.__concat = mt.__add
print(tb1 .. tb2)
```


---

__index

在后备表中浏览  
一般情况下，没有找到key时 返回 nil。 但有时，返回其他值更有意义。 __index 就是用于这种场景

下面是 当key 没有找到时的 操作
- 表 使用 不存在的 key 进行索引
- 如果 表的 __index 元表项 是一个表，那么 就返回这个表中 key 的值 (不存在就是 nil)
- 如果 表的 __index 是一个函数，那么会返回 这个函数的结果，函数的 参数 是key


```lua
race = {
    ["night"] = "nacht"
}
mt = {}
setmetatable(race, mt)

en_race = {
    ["human"] = "human",
    ["night"] = "night",
}
mt.__index = en_race
```

```lua
print(race["night"])    // nacht
print(race["human"])    // human
```


使用函数

除了使用表 作为 __index外，还可以使用函数

```lua
function mt.__index(tbl, key)
    print("attempt to access key '" .. key .. "' in " .. tostring(tbl))
    return nil
end
```



---

__newindex

引出新的键

3个参数
- tbl
- key
- value

设置新的键时，函数能通知你，它甚至可以防止 键被重复设置。

设置这种元方法后，它会对这种分配做出响应

下面的例子 允许你设置 除 "banana" 外的任何 索引值

```lua
function mt.__newindex(tbl, key, value)
    if key == "banana" then
        error("cannot set this")
    else
        rawset(tbl, key, value)
    end
end
```


---

旁路元表

在编写 __index, __newindex 的函数时， 获取/设置 值 时 必须忽略 元表。

- v = rawget(tbl, key)
- rawset(tbl, key)





## ch5 高级函数和控制结构


### 多值返回

return语句可以返回多个值

例子，将 十六进制的颜色 (#FFCC99) 转为 RGB

```lua
function ConvertHexToRGB(hex)
    local red = string.sub(hex, 1, 2)
    local green = string.sub(hex, 3, 4)
    local blue = string.sub(hex, 5, 6)

    red = tonumber(red, 16) / 255
    green = tonumber(green, 16) / 255
    blue = tonumber(blue, 16) / 255

    return red, green, blue
end
```


接收多个返回值

`var1, var2, var3, var4 = my_fun()`

如果返回值 多于变量数，则 多余的返回值被 忽略


#### 返回值丢失

多值返回 会出现一些奇怪的情况

```lua
print(ConvertHexToRGB("FFFFFF")) // 1,1,1
print(ConvertHexToRGB("FFFFFF"), "a string") // 1, a string
```

==规则==： 当函数调用所得的多个返回值是 另一个函数的 最后一个参数，或 多个指派表达式中的 最后一个参数时， 所有的返回值 会被传入或使用。 否则只有 第一个返回值 被使用或指定。



#### wow中的多返回值

一些api 返回多个值，比如 GetRaidRosterInfo 函数，输入是 角色的团队索引(一个数字)，输出是
- 角色名称
- 角色在团队中的级别
- 角色所属子群
- 角色等级
- 角色职业(本地化表示)
- 角色职业(大写的英文表示)
- 角色所在区域名
- 角色是否在线
- 角色是否死亡
- 角色是否是主T或主治疗
- 角色是否是物品分配人员


#### 获得指定返回值

- 使用哑变量
- 使用select()语句


哑变量

`local _, g = ConvertHexToRGB("FFFFFF")`

。。第三个返回值，由于 变量只有2个，所以自然就丢弃了。


select

```lua
print(select("#", "a", "b", "c"))   // 3
print(select(1, "a", "b", "c"))   // a,b,c
print(select(3, "a", "b", "c"))  // c
```

。。估计是 `a = select(2, GetRaidRosterInfo(123))` , 直接获得第二个返回值。 select返回 第二个及以后的， 但是 前面只有一个参数， 所以 第二个后面的 就忽略了。



### 变长形参列表

```lua
function test_print(...)
    print("test string", ...)
end
```

省略号 可以作为最后一个参数出现。接收任意数目的参数

在方法内部，可以用下面的形式 使用 省略号
- `print(...)`  参数传入到另一个函数
- `local var1, var2, var3 = ...`  给变量指派参数
- `local tbl = {...}`  使用参数新建表


```lua
function newtable(...)
    return {...}
end
```
```lua
tbl = newtable("1","2","3")
for i=1,#tbl do
    print(i, tbl[i])
end
```


#### 结合select()使用

```lua
function printargs(...)
    local cnt = select("#", ...)
    for i=1,cnt do
        local arg = select(i, ...)
        print(i, arg)
    end
end
```
这样可以避免 创建新表，还允许nil作为输入

。。看书上例子， nil会中断表。就是下面的。
```lua
function test1(...)
    local tbl = {...}
    for i=1,#tbl do
        print(i, tbl[i])
    end
end

function test2(...)
    for i=1,select("#", ...) do
        print(i, select(i, ...))
    end
end

test1("a","b",nil,"d")  // 1,a 2,b  // 没有nil及之后的
test2("a","b",nil,"d")  // 1,a 2,b 3,nil 4,d
```


### 泛型for循环和迭代器

```lua
for val1, val2, val3, ... in <expression> do
    xxx
end
```

泛型for循环 每次循环中 返回多个变量。  
expression应当返回下面的3个值
- 一个在每次循环中都可以被调用的迭代器函数
- 一个状态值，在下一次调用时都被迭代器函数所使用
- 一个初始值，用来初始化迭代器


lua提供了很多迭代器，可以满足大部分需求

---

#### 遍历表的数组部分

```lua
tbl = {"a","b","c"}
for idx, value in ipairs(tbl) do
    print(idx, value)  // 1,a  2,b  3,c
end
```

ipairs 接受一个表作为输入，返回遍历一个表的数组部分所需的信息，包括 迭代器函数本身。  
每次调用迭代器都返回 元素的序号 和 元素本身。  

`print(ipairs(tbl))`  function: 0x303333, table: 0x554433, 0  // 分别是，迭代器函数，状态(在这里就是传入的表)，迭代器的初始值
`print(tbl)` table: 0x554433

---

#### 遍历完整的表(数组+hash)

```lua
tbl = {"a","b",["qq"]="qq",["zz"]="zz"}
for k,v in pairs(tbl) do
    print(k, v)
end
```

pairs 的顺序是 随机的。  
ipairs 是顺序遍历

使用 pairs时，必须保证 不添加元素。 因为内部使用 next函数。


---

#### 清除表中所有元素

```lua
for k,v in pairs(tbl) do
    tbl[k] = nil
end
```


---

#### 其他迭代器

string.gmatch() 可以通过 lua 模式匹配 产生一个 匹配string 的迭代器。
```lua
for word in string.gmatch("there are some words", "%S+") do
    print(word)  // 一行一个单词
end

for char in string.gmatch("hello!", ".") do
    print(char)  --// 一行一个字符
end
```


### 数组排序

内置的 table.sort 能以默认的方式 对数字和字符串 进行排序。  
也可以自定义 比较函数

样本
```lua
guild = {}

table.insert(guild, {
    name = "z",
    class = "rogue",
    level = 70,
})

table.insert(guild, {
    name = "b",
    class = "mega",
    level = 30,
})
```

默认情况下，列表的顺序 和 插入顺序一致。
`for idx,value in ipairs(guild) do print(idx, value.name) end`

直接打印 value，会是 table:0x333333 这种信息。


`table.sort(guild)` 会报错，因为 lua不知道如何比较


```lua
function sortName(a, b)
    return a.name < b.name
end

table.sort(guild, sortName)

for idx, val in ipairs(guild) do
    print(idx, value.name)
end
```



## ch6 lua标准库

方括号表示参数可选

### 表table

`table.concat(table [, sep [, i [, j]]])`  
将表的数据项连接到一起，可选的分割字符串sep，和 起止下标。  
sep默认 空字符串， i默认1， j默认表的长度

```lua
tbl = {"a", "b", "c"}
print(table.concat(tbl, ":"))
print(table.concat(tbl, nil))
print(table.concat(tbl, "\n", 2, 3))
```

---

`table.insert(table, [pos,] value)`

pos默认 表的长度+1
会将 >=pos 的元素后移。

---

`table.maxn(table)`

返回最大正索引。 。。。 5.1有， 5.4没有了。  bing了 "wow lua version" ，官方5.1版本的子集。

当表不包含 正数 的索引时，返回0.  

。。是索引，hash的key，不是 数组的下标。

需要遍历一遍。

---

`table.remove(table [, pos])`

移除元素  
pos 默认 表的长度  
会将 >pos的元素 前移一位。

---

`table.sort(table [, comp])`

对表的 数组部分 进行排序

不稳定的排序。 (相同值 sort后， 位置无法保持)



### math

- math.abs(x)， 绝对值
- math.ceil(x)， >= x的最小整数
- math.deg(x)， 弧度x 的角度
- math.exp(x)， e^x
- math.floor(x)， <= x的最大整数
- math.fmod(x, y)， x%y
- math.log(x)
- math.log10(x)
- math.max(x,y,z,...)
- math.min(x,y,z,...)
- math.modf(x)， 返回2个值，x的整数部分，x的小数部分
- math.pi
- math.pow(x,y)
- math.rad(x)， 角度x 的弧度
- math.random([m [, n]])， 默认`[0, 1)`, 只有m时，`[1, m]`, m+n时，`[m, n]`  。。边界是这样的
- math.randomseed(x)， 设置随机数种子， 相同的种子，产生的随机数序列是一样的
- math.sqrt(x)， x^0.5



### string

- string.len(s)， 嵌套的多个0字符会被计数为1，所以 "a\000bv\000" 的长度是5
- string.lower(s)， 返回拷贝，全小写。 大写字母的定义取决于当前的区域设置
- string.rep(s,n)， 连续 n 个 s 拼接
- string.reverse(s)， 逆序
- string.sub(s, i [, j])， 返回子串，从i开始到j， j默认-1，即字符串的长度。 sub(s, -i) 返回 s 的长度为i的后缀
- string.upper(s)， 返回副本，所有小写字母被转换为大写


#### 格式化

string.format(format_string, ...)

支持的格式化有：%c, %d,%i, %o, %u, x, X, e, E, f, g,G, q, s

q 是接收一个字符串，使得它可以安全地被 lua解释器读取。

% 和 指示符 之间有几个选项
- +,-， 使得数字的输出带上 正负号
- 填充字符， 在指定宽度的输出中 填充多余的空位
- -， 表明 左对齐，右对齐，默认右对齐，添加 - 后变成左对齐  。。 怎么和 正负号区分？
- 数字，指定最小宽度
- 数字，指定 浮点数最多 保留几位小数。 string时 按这个数字截断
- 类型指定符，表示参数应当看做什么类型


`string.format("%%c:%c", 83)`  S


wow有一个额外的 选项，可以从参数表中选择参数

数字$

`print(string.format(%2$d, %1$d, $d, 13, 17))`  17, 13, 13


### 模式匹配

。。RE

|类别|匹配|
|--|--|
|x|字符x|
|.|任意字符|
|%a|任意字母|
|%c|任意控制字符|
|%d|任意数字|
|%l|任意小写字母|
|%p|任意括号字符|
|%s|任意空白字符|
|%u|任意大写字母|
|%w|任意数字或字母|
|%x|任意16进制数字|
|%z|0字符|
|%x|字符x本身，这是转义特殊字符的标准方法|
|[set]|在set中的字符|
|[^set]|字符的补集|


模式选项

- `*` 0个或多个
- `+` 1个或多个
- `-` 0个或多个，尽可能小的匹配
- `?` 0个或1个
- `%n` n在1-9之间，第n个捕获的字符串
- `%bxy` x和y不同字符，以x开头，y结束的字符串，x和y匹配平衡，即 x+1，y-1，第一个使得值为0的y就是最后一个元素，即 `%b()` 就是一个合格的括号表达式


模式捕获

当一个闭合的小括号中的模式匹配成功的时候，这个小括号中的模式就可以被存储(捕获)以备后用

捕获按照它们左侧的小括号来编号，  
`(a*(.)%w(%s*))` 中，`a*(.)%w(%s*)` 被存储在 第一个捕获中(编号1)，`.` 是编号2的捕获， `%s*` 是第三个

空捕获 `()` 捕获当前字符串的位置(一个数字)，例如，如果在 字符串 flaaap 中应用模式 `()aa()` ，就会有2个捕获，分别是 位置3 和位置5 

。。这个就是 官方文档中的，但是还是不太理解 flaaap 的3 5 的来源。



模式锚点

`^` 匹配字符串的开头  
`$` 匹配字符串的结尾



### 模式匹配函数

lua提供了4个函数来接收 模式字符串

- string.gmatch(s, pattern)， 返回一个迭代器，每次调用该迭代器 都会返回 字符串s 中 pattern模式的 下一个捕获
  `s = "hello world"   for w in string.gmatch(s, "%a+") do print(w) end`  
  模式需要匹配每一次出现，因此模式不能有太多的锚点
- string.gsub(s, pattern, repl [, n])， 返回s 的一个副本，所有( 或前n个 )模式出现的地方 被替换为 repl， repl可以是 
  - 字符串
  - `%n` n为1-9,  表示 第n个捕获的子串的值。 %0 表示整个匹配， %%表示一个 百分号
  - 表，这个表会在每次匹配中查询，并使用第一个捕获作为关键字。如果没有任何捕获，那么 完全匹配被用作关键字
  - 函数，在所有出现匹配的地方被调用，并且所有捕获的子串会 依次作为参数 传递进来。 如果没有捕获，那么整个字符串被传入
    - 如果 表 或函数 的返回值 是 string 或数字，那么会进行替换，如果是 false 或nil，则不会替换。
- string.match(s, pattern [, init])， 在s中寻找 第一个匹配。找到就返回，否则返回 nil。如果 模式找不到任何捕获，那么返回 整个字符串。   `Looks for the first match of pattern in the string s. If it finds one, then match returns the captures from the pattern; otherwise it returns nil. If pattern specifies no captures, then the whole match is returned.` ..确实是这样的，但是无法理解， nil 和 返回整个字符串，即 找不到匹配 和 没有任何捕获 之间有任何的差异吗？
  init指定了 开始搜索的地方，默认1，也可以取负值
- string.find(s, pattern [, init [, plain]])
  plain 默认false， 如果true，那么关闭 模式匹配，只是 单纯的 字符串匹配。


### wow

wow有额外的函数
- strsplit(sep, str)
  str中每次出现 sep，就拆分为独立的字符串
- strjoin(sep, ...)
  sep 连接 ... 字符串
- strconcat(...)
  直接连接
- getglobal(name)
  获取该名称的全局变量，`greek1,greek2 = "a","b"  for i=1,3 do print(getglobal("greek"..i)) end`
- setglobal(name, value)
- debugstack([start [, count1 [, count2]]])， start 栈轨迹深度，默认1， 输出栈顶的元素个数，默认12， 输出栈底的元素个数，默认10



#### 函数别名

wow 中，很多库函数 有 更短的别名

。。就是把 math，string，table 3个库的方法， 可以不带 前缀 math,string,table




## ch7 学习xml


转义

```text
&  &amp;
<  &lt;
>  &gt;
"  &quot;
'  &apos;
```


wow的xml模式在 安装目录的 Blizzard intergace Data (enUS)/FrameXML/UI.xsd   
。。现在没有这个了。 UI.xsd 都没有。 。。 xsd 都没有。




## ch8 wow编程概述

p109  
默认情况下，wow不会显示错误，需要手工打开
- esc
- interface
- 在第二组选择框中，也就是屏幕的中央，选中显示lua错误的选项

。。我 esc - 插件， 里面没有 error 之类的东西。 有个 !DevErrorFrame 不知道是不是。


回车，输入 `/run message("hi")` 回车。   。。 可以， 屏幕中间弹出一个框 里面的内容是 hi， 可以点击 确定按钮

。应该没有问题，书上说，如果没有 弹框的话，说明 没有 选择 显示lua错误。

wow中
`/run print("hi")` 可以。  但 书上说，会报错，因为没有全局 print 函数。  改了好多啊。

。。`/run error("hi")`  不行，没有任何输出。



---

wow 中有2种主要的方法用于显示输出
- message， 输出窗口是受限的，太长的字符串会被 截断。 这个函数是 弹出窗口，很不方便， 一般只有 写代码 或 调试代码时 才用到
- AddMessage， 向聊天窗口发送文本信息。 向用户显示信息的 主要方法


默认情况下，wow有 7个聊天窗口，每个聊天窗口都有一个函数，用来在 聊天窗口中显示消息。  
默认情况下， `ChatFrame1` 是主要窗口，所以一般将它用于主输出窗口  
`/run ChatFrame1:AddMessage("hi")`

。可以， ChatFrame4 是 我新建的 聊天框， 1是默认的聊天框 。 其他的数字 不知道输出到哪里去了。

message，AddMessage 接收字符串，所以不能输出 表格值。

可以 `/run message(tostring(ChatFrame1))`   。可以，框中显示 `table: 0x123123` 。 试了下， 1-10 都有地址， 0 或 大于10 的，是 nil



### wow中lua编辑器


- TinyPad
- Omnibus
- WowLua


。。第二个已经没有了，第一个在网易有爱中的描述中，lua的功能很少了。 所以使用了 第三个。 而且书上也是用这个的


### 插件和脚本的局限

设计插件时，必须记住的约束
- wow的lua实现中 没有任何形式的 文件输入输出。 插件可以通过保存变量来存储数据，但是它们只有在 游戏启动时才能加载，游戏关闭或 界面重载时 才能保存
- 插件不能 智能地 指向目标。 例如，无法 自动将当前生命值最少的 团队成员 设置为目标
- 不能 智能地 为你选择一个法术
- 不能控制 角色的移动

创建插件时，一个通用的规则是， 为玩家 提供尽可能多的信息， 从而 可以让他们做出正确的选择， 但是，必须由 玩家做出决定 并 单击按钮。 这意味着，你可以 突出一个 特定目标，或者 在法术栏 突出 应该使用的法术， 但是 最后的决定 必须是 玩家做出的。




p114

介绍了一个 World of Warcraft Interface AddOn Kit

。。但是似乎没用了，直接bing能搜到， 下载，是一个 exe，打开会 报错：找不到 魔兽世界安装目录。而且是 08年的， 估计 和现在的 架构完全不同了，所以删除了。   
.. https://wowpedia.fandom.com/wiki/Interface_AddOn_Kit 这里下载的 exe
。。又看了下，确实， 从 4.0.1 开始 过期了。  又给了下面的地址，
。 https://wowpedia.fandom.com/wiki/Viewing_Blizzard%27s_interface_code   。。但是不会用。。这里有3个链接， 似乎可以。
- https://github.com/Gethe/wow-ui-source
- https://www.townlong-yak.com/framexml/live    这个最好。
- https://github.com/Gethe/wow-ui-textures



。。还下载到一个 Wow_Interface_zhTW， 里面应该是一些 wow 提供的 api？ 我看是 2010年的，估计还有用。
..确实应该还有用，解压后在 FrameXML中 有 ChatFrame.lua， 里面能搜到 ChatFrame1 
..  https://addonstudio.org/wiki/WoW:Interface_AddOn_Kit  最下面 下载了 TW 的


https://www.wowinterface.com/addons.php   似乎没什么用了，就是一些插件，但是都用 course 了。


## ch9 插件解析

Interface/AddOns 用于存放插件

### 暴雪官方插件

很多功能是通过独立的插件实现的，这些插件只有在 客户端有需求时 才加载。

比如拍卖行界面，只有玩家访问 拍卖师时，游戏才会加载 Blizzard_AuctionUI 插件。

插件与触发条件
- Blizzard_AuctionUI， 与拍卖师交互时
- Blizzard_BattlefieldMiniMap， shift+战场或jjc的图标 或 打开战场小地图
- Blizzard_BindingUI， 主菜单打开 快捷键设置
- Blizzard_CombatText， 在高级插件选项中 启动 blizzard floating combat text 功能
- Blizzard_CraftUI， 打开 会用到CraftUI的事件 (如 附魔，宠物训练等)
- Blizzard_GMSurveyUI， 和GM交流，他让你做一个调查
- Blizzard_InspectUI， 观察其他玩家
- Blizzard_ItemSocketingUI， shift+右键 点击 item  。。就是发送到聊天框
- Blizzard_MacroUI， 打开宏编辑
- Blizzard_RaidUI， 参加 raid
- Blizzard_TalentUI， 打开天赋
- Blizzard_TradeSkillUI， 打开需要使用 TradeSkill 的商业技能
- Blizzard_TrainerUI， 与职业训练师交谈

每个blizzard插件 在目录中都有一个 .pub 的唯一文件。 代表可信任的插件



### 自定义插件

和 官方插件同目录。

它们放在由 插件命名的 单独子目录中， 包含了 插件所需的所有文件。


#### 内容表格文件 toc

必须包含，必须以 插件 文件夹的名字 命名

主要提供一些重要的信息(如 标题，描述，作者) 来帮助 加载 特定的插件文件。

.toc 示例如下

```text
## Interface: 20300
## Title: My_Addon_name
## Notes: this is a simple addon

MyAddon.xml
MyAddon.lua
```

`##` 开头的行 包含一些 与各种元数据有关的指令。  
然后是 要加载的文件的列表  

Title 是 插件选择窗口中 展现的名字。
Note 是 鼠标在插件名字上悬停后的 详细描述


`## Interface` 为用户提供了一种基本的版本控制。 一般是客户端的版本号(如 2.3.0 的客户端 就是20300)  
后面的数字 被 客户端用来 判断插件的状态。
- out of date， 表明 客户端的版本 比 `## Interface` 的版本 更新，插件加载后可能有错误， 是一个警告， 用户可以选择 加载过期插件 就可以了。
- incompatible， 用户界面有很大的改变，客户端 不能加载这个插件，需要重新下载。

插件可以对 单个角色 或 所有角色开启 (在 人物选择界面)


`## Title: MyAddonName`  
插件选择窗口中 列出的插件是 通过 `## Title` 进行分类和显示的， 而不是 toc文件和目录。  

title 默认值 就是 插件目录的名字  

title 可以用于本地化。 title后面加 - 和 区域码即可，如 `## Title-koKR: xxx`


`## Notes: xxx`  
对 title 值 的描述  
让用户知道 插件可以做什么 等信息  
也可以 本地化

title 和 notes 可以使用 颜色代码 来突出显示。


`## Dependencies` `## RequiredDeps: AddonA, AddonB`

本插件依赖于什么插件。


`## OptionalDeps: AddonA, AdonB`

本插件和其他插件有交互，但不是 硬从属。

这里的名字 必须和 .toc 文件名 和 所给插件的目录名 匹配


`## LoadOnDemand: 1` 或0

懒加载 or 急加载

默认0 (急加载)


`## LoadsWith: AddonA, AddonB`

当 AddonA 或 AddonB 被加载后， 也会加载 本插件


`## DefaultState: enabled` disabled

插件的默认状态， 是否启用。


`## LoadManager: AddonA`

如果 AddonA 可用，则 本插件 LoD (load on demand，按需加载)。

。。不，应该是： 由 AddonA 来决定 本插件是否 加载。

AddonLoader 是一个简单的插件， 被一些其他插件用做 LoadManager， 其他插件的开发者 将 条件 保存到 toc 文件中，AddonLoader 根据这些条件，判断 插件的加载时机。


`## SavedVariables: VarName`

在会话中，插件要保存信息 只能通过定义一个lua 变量，并在 toc文件中 列出。  
==关闭游戏的时候，系统会将这个 值 保存到 文件中，启动时，读取。==  
变量类型可以是 字符串，数字 或 表格

`## SavedVariablesPerCharacter: VarName`

每个角色 保存 变量值 到文件，并在启动时 加载


非标准元数据

`## Auther: xxx`
`## Version: 1.0`
`# X-xxx`


#### lua脚本文件

toc列出了 每个lua文件(包括 子目录中的文件)。  
这些文件 是按照 toc中的顺序 进行加载的  
可以在文件层面上 定义局部变量，这些变量 对其他文件 不可见

lua更多地用于 定义插件的行为， 而不是 界面， 当然 它也可以用来 动态创建窗口 (ch16)


#### xml文件

。似乎不需要在 toc文件中 列出？但是。。
。。原文的翻译是真的看不懂：".toc文件可以从插件中下载任何.xml文件"

xml文件被解析和加载时， 使用 UI.xsd 进行校验。

xml可以使用 `<Script file="xxx.lua">` 来加载 lua文件。  

每个xml 文件 只有一个 顶级元素 `<Ui>`

在 Logs/FrameXML.log 中有日志。

#### 媒体文件

音乐  
例如 有个文件： `Interface/Addons/MySound/mysound.mp3`  
在游戏中 `/run PlaySoundFile("Interface\\Addons\\MySound\\mysound.mp3")` 就可以 播放。

wow 可以识别 mp3 和 wav


图形  
窗口纹理， wow 只支持2种 图形格式， 且 必须满足
- 文件的高度，宽度 大于等于 2像素，小于(。。等于？) 512像素
- 高度，宽度 必须是 2的幂， 但不必相同

BLP格式，暴雪自己的，没有官方的转换工具

TGA格式， 比较简单的图像格式，用于存储 带有透明度信息的 彩色图像，不能无损压缩。


### 本地化

- deDE  德语
- enUS  美国英语
- enGB  英国英语
- esES  西班牙语
- frFR  法语
- koKR  韩语
- zhCN  简体
- zhTW  繁体


插件的作者应该
- 提供一个 常量集，或 翻译列表，而不要仅使用字符串
- 本地化信息 写入到 readme.txt ，或 自己的官网上
- 提供一些注释 来帮助翻译， 比如 speed 有多种含义。


本地化实现步骤
1. 为你翻译的语言添加一个 新的本地化文件。比如，Localization.enUS.lua 。 并把这个文件 放到 toc的==顶部==，保证 首先加载
2. 创建一个 包含基本串的全局表， 假设插件名字是 MyAddon，那么在 Localization.enUS.lua 中创建一个 MyAddonLocalization 的全局表。 向这个文件 添加基础翻译 有2种方式， 使用完整字符串， 使用标记
  - 使用完整字符串，我们 既使用 完整字符串作为 键，又作为值
    ```lua
    MyAddonLocalization = {}
    MyAddonLocalization["Frames have been locked"] = "Frames have been locked"
    MyAddonLocalization["Frames have been unlocked"] = "Frames have been unlocked"
    ```
    它可以 用于较短的字符串，但是用于 较长字符串 键 会变得特别冗余
  - 使用标记
    我们可以使用 较短的字符串 或标记 来代替完整的 字符串作为 表的键。
    ```lua
    MyAddonLocalization = {}
    MyAddonLocalization["FRAMES_LOCK"] = "Frames have been locked"
    MyAddonLocalization["FRAMES_UNLOCK"] = "Frames have been unlocked"
    ```
3. 使用本地化表。 已经确保 首先加载 本地化文件，所以你可以使用 表 而不是 简单地写入字符串。例如，你可以通过 下面的方式 来显式 lock 信息
    `ChatFrame1:AddMessage(MyAddonLocalization["Frames have been locked"])` 或 `ChatFrame1:AddMessage(MyAddonLocalization["FRAMES_LOCK"])`

    还可以进一步对 全局表简化
    ```lua
    local L = MyAddonLocalization
    ChatFrame1:AddMessage(L["FRAMES_LOCK"])
    ```
    如果你使用 标记 的方法，甚至可以
    `ChatFrame1:AddMessage(L.FRAMES_LOCK)`
4. 添加新的本地化， 将下列代码添加到 Localization.deDE.lua，且保证 toc中，这个文件 在 ==xx.enUS.lua 之后==， 主插件之前 加载
   ```lua
   if GetLocale() == "deDe" then
        if not MyAddonLocalization then
            MyAddonLocalization = {}
        end
        MyAddonLocalization["FRAMES_LOCKED"] = "德语xxx"
        MyAddonLocalization["FRAMED_UNLOCKED"] = "deyu xxx"
   end
   ```


### 创建插件框架


为插件命名  
插件必须位于 Interface/AddOns 中。  

`Interface/AddOns/WowXMLExample` ， 接下来 2章 中的 所有文件都位于这个目录中


---

创建 .toc 文件

```text
## Interface: 20300
## Title: WowXMLExample
## Description: example

WowXMLExample.xml
```

Interface 号 应该查询 FrameXML.toc 来获得 最新版本


---

创建框架 .xml 我呢件

将下列代码复制到 WowXMLExample.xml 中

```xml
<Ui xmlns="http://www.blizzard.com/wow/ui"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.blizzard.com/wow/ui..\FrameXML\UI.xsd">

</Ui>
```

。。现在已经没有 FrameXML 了， 需要看下其他的插件。


---

### 使用外部库


附录E p878 列出了 主要的库和框架，及详细信息


## ch10 在xml中创建窗体


### `<UI>`

创建的用户界面对象 主要有3种
- 纹理(texture)， 纹理 在游戏中被用于 显示一些图形，颜色，颜色渐变效果。纹理使用 `<Texture>` 标签创建
- 字体字符串(FontString)， 用于 显示 指定字体，尺寸，颜色的 文本。可以进一步进行设定，以 空心，阴影，或其他标准效果来显式。`<FontString>` 标签
- 窗体(Frame)， frame是一个 容器，拥有 纹理 和 字体字符串。 frame有一些特定的子类型，如 按钮，编辑框，卷轴信息。 frame可以嵌套。 `<Frame>` 标签


---

#### 为对象命名

每个基本用户界面对象 都可以通过 name属性 得到一个名字， 这个名字允许 你接下来 在xml 或 lua 中， 对此对象进行引用。  
将窗体命名，是一个 好的习惯  

。。不知道默认是 什么？ ""吗？ or nil


---

#### 指定父对象

除了名字，每个对象还可以得到一个 父对象。 为界面定义一个层次化结构
- 当父对象被隐藏，它的所有子窗体也被隐藏。
- 窗体的缩放程度 由 自身的尺寸 乘以 父窗体的缩放程度 得到。
- 窗体的透明程度 由 自身的透明程度 乘以 父窗体的透明程度 得到。

暴雪的许多默认窗体 继承自 UIParent，它可以通过命令隐藏用户界面，并提供 缩放。  

当创建自己的窗体时，最好使得它们 最终继承 UIParent，以保证 它们和其他窗体一样，随用户的设置而改变。

3种方式 设置 父子关系
- 完全使用xml属性 指定父对象
- 使用xml 元素层次结构来定义关系
- 直接对一个对象调用 SetParent() 方法


---

完全使用xml
```xml
<Frame name="myFrame" parent="UIParent">
</Frame>
```

---

xml元素层次
```xml
<Frame name="myFrame" parent="UIParent">
    <Layers>
        <Layer level="BACKGROUND">
            <FontString name="MyText"/>
            <Texture name="MyGraphic">
        </Layer>
    <Layers>
</Frame>
```

Layers 用于组织所有的层次  
Layer 声明一个指定的层，在该层上声明了 字体字符串 和 纹理  
上面的例子是 创建一个 包含 字体字符串和纹理的 背景层 的窗体。由于这2个图形元素 在 窗体内声明，所以它们都继承自 myFrame

---

SetParent()

加载窗体后，可以 MyFrame:SetParent(Minimap) 来将 Minimap 设置为 MyFrame 的 父窗体。   
参数必须是 窗体对象，不能是 字符串


命名窗体的时候，可以通过 `$parent` 来包含父对象的名字。例如，下面的 窗体的名字是 UIParentMyParent，
`<Frame name="$parentMyFrame" parent="UIParent">`

这种技巧，对 纹理和 fontstring 都可以。


#### 设置对象尺寸

`<Size>` 可以设置 绝对尺寸，相对尺寸


绝对尺寸

宽100像素，高50像素

```xml
<Frame name="MyFrame">
    <Size x="100" y="50"/>
</Frame>
```
或
```xml
<Frame name="MyFrame">
    <Size>
        <AbsDimension x="100" y="50"/>
    </Size>
</Frame>
```

---

绝对尺寸

父窗体尺寸的百分比

```xml
<Frame name="myFrame1">
    <Size x="100" y="50"/>
</Frame>
<Frame name="myFrame2" parent="MyFrame1">
    <Size>
        <RelDimension x="0.5" y="0.5"/>
    </Size>
</Frame>
```

默认UI中，很少看到这种方法。



#### 锚定对象

所有对象的放置 通过 一系列的 锚点(anchor)完成。  
锚点 将窗体的一个位置 粘贴在 另一个窗体的 某个位置上。

可用锚点类型
左上 TOPLEFT， 上 TOP， 右上 TOPRIGHT
左 LEFT， 中 CENTER， 右 RIGHT
左下 BOTTOMLEFT， 下 BOTTOM， 右下 BOTTOMRIGHT

所有的锚点在 `<Anchors>` 中定义， 使用 `<Anchor>` 及以下属性
- point， 被锚定的点
- relativeTo， 被锚定的点所在的窗体
- relativePoint， 附加到 relativeTo 窗体上的点。 可以不指定，默认是 被锚定的点


`<Anchor>` 还包含 `<Offset>`， 可以设置 绝对 或相对 偏移量 ( 方式和 `<Size>` 相同)。

下面的代码，创建 MyFrame，并将其锚定在 UIParent 的中间

```xml
<Frame name="MyFrame" parent="UIParent">
    <Size x="100" y="100"/>
    <Anchors>
        <Anchor point="CENTER" relativePoint="CENTER" relativeTo="UIParent">
            <Offset x="0" y="0"/>
        </Anchor>
    </Anchors>
</Frame>
```

下面的代码，创建 MyFrame2，其位于 MyFrame1 的右面。
```xml
<Frame name="MyFrame2" parent="UIParent">
    <Size x="50" y="50"/>
    <Anchors>
        <Anchor point="TOPLEFT" relativePoint="TOPRIGHT" relativeTo="MyFrame"/>
    </Anchors>
</Frame>
```


粘性锚点

窗口的锚定 是永久的，即 移动后，依然锚定着。


---

#### 窗体和图形元素分层

wow的用户界面系统 提供了一个 非常规整的方法，用于将图形元素放置于 另一个的上层。  
它允许创建一个 完全覆盖于另一个窗体上的 窗体，或者由分布于各层的图形元素来得到最终的图像。

有3个级别的 分层方式： 窗体层，窗体等级，纹理/字符串层(图形层)

---

窗体层  
窗体分层的最简单方法是 窗体层  

简单来说，给定窗体层 中的 所有窗体 将比 更低窗体层中的 所有窗体更晚呈现， 比 更高层窗体层中的 所有窗体更早呈现。

。。更早呈现，是指 在下层，还是 在表层？。。


从低到高，所有可能的窗体层
- BACKGROUND， 所有的窗体不会与 鼠标进行交互， 本层中 所有窗体不会 收到 鼠标事件，除非 窗体等级大于1
- LOW， 被默认用户界面用于 增益状态窗口，耐久度窗口，团队界面，宠物窗体
- MEDIUM， UIParent 及其所有 子窗体 的 默认窗体等级
- HIGH， 被默认用户界面 用于 动作按钮，教程窗体，界面错误，警告窗体
- DIALOG， 用于任何对话类型的窗体，它们弹出并等待用户的响应
- FULLSCREEN， 本层应当包含所有的全屏窗体，如世界地图或用户界面选项
- FULLSCREEN_DIALOG， 位于 FULLSCREEN 之上的一个对话层，用于 对话交互 及下拉菜单
- TOOLTIP， 最高窗体层，用于 鼠标提示


在指定的层 绘制一个 窗体很容易
```xml
<Frame name="myFrame" frameStrata="HIGH">
</Frame>
```


---

窗体等级

在一个层中，每个窗体具有一个 窗体等级，来决定 呈现的顺序(从低到高)   
设置比较繁琐。 但有时，是实现一些指定层次类型的 唯一途径。  

当 FrameA 包含子窗体 FrameB时， FrameB 的等级 比 FrameA 高一级。

。。高的在 表层。

窗体等级 通过 frameLevel属性设置  

窗体可以使用 toplevel属性， true时，当窗体被单击时，它会自动升级为 给定层中的 最高窗体等级， 在最表层显示。  
当窗体可移动时，这个方法很有用，它可以将 被移动的窗体置于 最表层。

```xml
<Frame name="myframe" frameStrata="HIGH" frameLevel="5" toplevel="true">
</Frame>
```


---

图形层

所有的 Textures 和 FontStrings 必须 属于一个 给定窗体的 Layers 元素。  
这些 layer 允许 纹理和字体字符串 进行更加规整的分层


图形层的描述
- BACKGROUND， 窗体的背景应当放置于此
- BORDER， background之上，其他层之下的 图形框(border) 或 插画(artwork) 放在这里
- ARTWORK， 窗体的插画，非功能性的插画
- OVERLAY， 窗体的功能性部分，如 按钮，文本框 等
- HIGHLIGHT， 鼠标悬停的 提示。为了使 highlight有效，窗体的 enable Mouse属性 应设置为 true

不同的层次通过 `<Layer>` 元素 及 level 属性创建
```xml
<Frame name="myFrame">
    <Layers>
        <Layer level="BACKGROUND">
        </Layer>
        <Layer level="HIGHLIGHT">
        </Layer>
    </Layers>
</Frame>
```

#### 一般属性

除了基本的 命名，尺寸，定位 外，还有 一些属性 经常用于 3种基本用户界面的对象。

- Name， 被创建对象的名字。 对象可以匿名， 但一旦有了名字，就可以 把名字当做全局变量 在lua中访问。名字字符串中可以包含 `$parent`。   。。这个不就是 name 吗？
- Inherits， 接收一个 逗号分开的 窗体列表，用于继承的模板名
- Virtual， 布尔值，窗体是否是 虚模板。 虚模板指 没有实际的对象被创建。
- setAllPoints， 布尔值， 窗体是否应当被粘附到 其父窗体的所有点上，这样 窗体 和父窗体有相同的 尺寸和位置
- Hidden， 布尔值，窗体 是否默认 隐藏


### 创建纹理

纹理 是客户端为 用户界面 渲染的二维图形元素。  
可以是 从磁盘加载的图形文件， 带有透明度的 固定颜色值， 从一个颜色到另一个颜色的渐变。  
都使用 `<Texture>` 标记。

纹理 元素拥有下面的 属性，以及之前 列出的 一般属性
- file， 纹理文件的路径
- alphaMode， 对多个纹理进行分层时，使用的混合模式，可以是： DISABLE, BLEND, ALPHAKEY, ADD, MOD


#### 添加颜色 Color

`<Color>` 4个属性，r,g,b,a,  红绿蓝 透明度， 都是 0.0 - 1.0 之间

```xml
<Frame name="MyFrame">
    <Layers>
        <Layer level="BACKGROUND" setAllPointes="true">
            <Color r="1.0" g="0.1" b="0.1" a="1.0"/>
        </Layer>
    </Layers>
</Frame>
```
上面的代码 创建一个窗体，窗体仅包含 红色纹理

#### 添加渐变效果 Gradient

`<Gradient>` 设置 最小颜色，最大颜色，并定义一个 渐变效果。  

。。不是动态渐变，就是一张图， 上面是 最小颜色，下面是最大颜色， 中间是慢慢过渡的。

Gradient 有一个 可选属性 orientation， 可以是 HORIZONTAL 或 VERTICAL，默认是 垂直渐变(HORIZONTAL)

Gradient 必须包含 2个元素 `<MinColor>`, `<MaxColor>`, 都是 ColorType 类型  

单个 Gradient ==不会渐变，必须==和 Color 结合使用。 从color获得颜色值 乘以 当前渐变值 得到 最终屏幕上的颜色。

最简单的实现是  
`<Color r="1.0" g="1.0" b="1.0" a="1.0"/>`  
这保证了 你的渐变值 从 MinColor开始，到 MaxColor 结束。 由于每个 值都是 1.0，所以 相乘后，不会改变 颜色值。

```xml
<Ui xmlns="http://www.blizzard.com/wow/ui/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.blizzard.com/wow/ui/..\FrameXML\UI.xsd">
<Frame name="GradientTest" parent="UIParent">
    <Size x="200" y="200"/>
    <Anchors>
        <Anchors point="CENTER" relativePoint="CENTER" relativeTo="UIParent"/>
    </Anchors>
    <Layers>
        <Layer level="BACKGROUND">
            <Texture name="$parentHorizontal">
                <Size x="200" y="100"/>
                <Anchors>
                    <Anchor point="TOPLEFT" relativePoint="TOPLEFT"/>
                </Anchors>
                <Color r="1.0" g="1.0" b="0.0" a="1.0"/>
                <Gradient orientation="HORIZONTAL">
                    <MinColor r="1.0" g="0.0" b="0.0" a="1.0"/>
                    <MaxColor r="0.0" g="0.0" b="0.0" a="1.0"/>
                </Gradient>
            </Texture>
            <Texture name="$parentVertical">
                <Size x="200" y="100"/>
                <Anchors>
                    <Anchor point="BOTTOMLEFT" relativePoint="BOTTOMLEFT"/>
                </Anchors>
                <Color r="1.0" g="1.0" b="1.0" a="1.0"/>
                <Gradient orientation="VERTICAL">
                    <MinColor r="0.0" g="0.0" b="0.0" a="1.0"/>
                    <MaxColor r="1.0" g="1.0" b="0.0" a="1.0"/>
                </Gradient>
            </Texture>
        </Layer>
    </Layers>
</Frame>
</Ui>
```


#### 添加图形元素

下面的代码，在屏幕中间创建一个 100 * 100 的窗体，展示 暗言术痛 的图标

```xml
<Ui xmlns="http://www.blizzard.com/wow/ui/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.blizzard.com/wow/ui/..\FrameXML\UI.xsd">
<Frame name="GraphicTest" parent="UIParent">
    <Size x="100" y="100"/>
    <Anchors>
        <Anchor point="CENTER" relativePoint="CENTER" relativeTo="UIParent"/>
    </Anchors>
    <Layers>
        <Layer level="BACKGROUND">
            <Texture name="$parentPainIcon" file="Interface\Icons\Spell_Shadow_ShadowWordPain" setAllPoints="true">
            </Texture>
        </Layer>
    </Layers>
</Frame>
</Ui>
```



### 使用 fontstring创建文本

向窗体中添加文本，需要在 窗体中创建一个 FontString对象，并将其锚定到窗体上。

FontString 元素必须在 Layer中。  

除了一般属性外，FontString 还有下列的属性
- font， 字体文件路径
- bytes， 一个正数，用来指定 FontString 最多可以显示的字符数
- text， 显示的文本
- spacing， 如果字体字符串 具有多行，则设定 行间隙， 单位是 像素
- outline， 轮廓类型，必须是： NONE，NORMAL，THICK
- monochrome， 布尔值， 字体是否以单色(灰度) 显示
- nonspacewrap， 布尔值，没有空格的长字符串是否应该 扭曲或截断。
- justifyV， 使用 TOP，MIDDLE，BOTTOM 来指定 文本的 垂直修正
- justifyH， 使用 LEFT，CENTER，RIGHT 来指定文本的 水平修正
- maxLines， 显示的最大行数


#### 使用模板

一些可用的模板， 提取自 Fonts.xml 文件
- GameFontNormal
- GameFontNormalSmall
- GameFontHighlightSmallOutline
- GameFontNormalHuge
- ChatFontNormal
- ChatFontSmall
- DialogButtonNormalText
- GameTooltipText
- WorldMapTextFont
- CombatTextFont

等。。。这个要自己去 Fonts.xml 中找。 。但是 没有这个文件。。

使用的例子
```xml
<Ui xmlns="xxxxxxxxxx">
<Frame name="FontTest" parent="UIParent">
    <Size x="200" y="50"/>
    <Anchors>
        <Anchor point="CENTER" relativePoint="CENTER"/>
    </Anchors>
    <Layers>
        <Layer level="OVERLAY">
            <FontString setAllPoints="true" justifyV="MIDDLE" justifyH="CENTER" inherits="GameFontNormalHuge" text="Hello!"/>
        </Layer>
    </Layers>
</Frame>
</Ui>
```


#### 进一步自定义

字体字符串 显示的 定制
- `<FontHeight>`， 使用 val 属性 或 `<AbsValue>` `<RelValue>`， 以绝对值 或 相对值 来指定 字体字符串的高度
- `<Color>`， 改变 fontstring的颜色，使用 rgba指定
- `<Shadow>`， 增加阴影效果。 阴影的颜色和位置 通过 `<Color>` `<Offset>` 来指定，2个都是必选



### 其他窗体类型

p141 / 123 , 有图

。。下面的在 ch31 描述了 api


除了 Frame 还有几种窗体， 这些窗体 都可以由 基本的 Frame 组成。

按钮 Button

复选旋钮 CheckButton

颜色选择 ColorSelect   。。 就是 调整 聊天框背景的那种

编辑框 Editbox

游戏工具提示 GameTooltip  。。 鼠标悬停后的 框

消息窗体 MessageFrame  。。 "法术没有准备好"

小地图 Minimap  。。 右上角那个

模型 Model   三维模型， 旋转，缩放功能。

滚动信息窗体 ScrollingMessageFrame    。聊天窗体

滚动窗体 ScrollFrame    。任务文本过长时，好友窗口

简单的HTML窗体 SimpleHTML

滑动器 Slider   。 声音 0% - 100%

状态栏 StatusBar 。。 武器技能

飞行路线窗体



## ch11 向xml窗体中添加行为

可用的窗体脚本 和方法的 完整列表，参阅 ch31


游戏通过 窗体脚本 与用户交互


### 窗体脚本

每个窗体都有特定的行为，它们可以实现响应，从而使得界面具有 交互性。  
例如  按钮可以对 单击事件作为响应， 编辑框对 字符输入做出响应。 `<OnClick>` `<OnValueChanged>`


### 游戏事件

指 客户端把 信息发送到 用户界面。 例如，单位的 生命值改变时， 会触发 UNIT_HEALTH 事件。

为了响应 游戏事件， 窗体必须使用 :RegisterEvent() 来注册 监听的事件。 这会使得 客户端在每次事件触发时 为 窗体调用 `<OnEvent>` 脚本

窗体可以 注册 任意多的事件，但是 只能有一个 `<OnEvent>` 脚本。  游戏将 事件名称 发送到 该处理模块， 模块根据不同的事件 进行不同的响应

### 使用脚本响应窗体事件 `<Scripts>`

为了交互， 你可以定义 被调用的窗体脚本，以响应用户交互。 可以使用 xml来创建这些脚本， 方式是 将一个 `<Scripts>` 加入到 窗体定义中，并对 每种类型的 行为 定义一个处理脚本。


#### `<OnEnter>` `<OnLeave>`

任何 具有 enableMouse 属性的 窗体都可以使用 自定义的脚本 对 OnEnter OnLeave 进行响应


在 WowXMLExample.xml 中 添加
```xml
<Frame name="EnterLeaveTest" parent="UIParent">
    <Size x="100" y="100"/>
    <Anchors>
        <Anchor point="CENTER" relativePoint="CENTER" relativeTo="UIParent"/>
    </Anchors>
    <Layers>
        <Layer level="BACKGROUND">
            <Texture name="$parentIcon" file="Interface\Icons\Spell_Shadow_ShadowWordPain" setAllPoints="true"/>
        </Layer>
    </Layers>
    <Scripts>
        <OnEnter>
            ChatFrame1:AddMessage("++ Enter : " .. self:GetName())
        </OnEnter>
        <OnLeave>
            ChatFrame1:AddMessage("-- leave : " .. self:GetName())
        </OnLeave>
    </Scripts>
</Frame>
```

元素内的 处理模块 是 函数形式的 lua代码。  

每个处理模块 都会收到一个 self变量， 它是第一个参数，即 窗体对象本身。


#### `<OnLoad>`

xml 定义的每个窗体都可以有一个 `<OnLoad>` 脚本， 并在 窗体最初创建时 执行。  
通常用于 注册事件 和 其他的初始化行为。

下面的xml可以用于注册 UNIT_HEALTH 事件
```xml
<Scripts>
    <OnLoad>
        self:RegisterEvent("UNIT_HEALTH")
    </OnLoad>
</Scripts>
```


#### `<OnEvent>`

为了实际地进行一些操作 来响应 游戏事件， 必须有一个 `<OnEvent>` 脚本 来处理这些响应。

OnEvent 脚本 会收到： 窗体对象，事件名称(一个字符串)， 一系列的参数(不同事件 不同参数)。

OnEvent 中的任何代码 都将放入下面的 函数定义中，并被设置为 OnEvent脚本。
```lua
function OnEvent(self, event, ...)
    if event == "UNIT_HEALTH" then
        -- xxx
    elseif event == "UNIT_MANA" then
        -- xxx
    end
end
```

这个处理模块 负责 ==响应所有的==游戏事件。

ch30 有 事件列表

下面的代码加入到 WowXMLExample.xml 中
```xml
<Frame name="UnitHealthTest">
    <Scripts>
        <OnLoad>
            self:RegisterEvent("UNIT_HEALTH")
        </OnLoad>
        <OnEvent>
            local unit = ...
            ChatFrame1:AddMessage("UNIT_HEALTH - unit=" .. unit)
        </OnEvent>
</Frame>
```

UNIT_HEALTH 事件 仅有一个参数 (生命值发生改变的单位)。   
我们将 ... 指定到局部变量，这样 更容易输出



#### `<OnClick>`

按钮，复选按钮， 使用 OnClick 脚本 响应 对 按钮的单击， 函数有3个参数
- self， 被单击的按钮
- button， 进行单击的鼠标按键
- down， 鼠标单击的方向


wow 可以识别 5种 鼠标按键， 左键，右键，中键，按键4，按键5 。 LeftButton, RightButton, MiddleButton, Button4, Button5

游戏可以区分 单击的 向下部分，向上部分。

默认情况下，按键仅接受  LeftButtonUp 事件。 可以使用 `:RegisterForClicks()` 来改变

为了注册 左击，右击，并且同时 接收 up，down， 可以在 OnLoad中调用下面的代码
`self:RegisterForClicks("LeftButtonUp", "LeftButtonDown", "RightButtonUp", "RightButtonDown")`

注册一个按键 最简单的方法是 在 RegisterForClicks 中使用 AnyUp， AnyDown 虚拟按键。

下面的代码放到 WowXMLExample.xml 的Ui 中
```xml
<Button name="OnClickTest" parent="UIParent">
    <Size x="100" y="100"/>
    <Anchors>
        <Anchor point="CENTER" relativePoint="CENTER" relativeTo="UIParent"/>
    </Anchors>
    <Layers>
        <Layer level="BACKGROUND">
            <Texture name="$parentIcon" file="Interface\Icons\Spell_Shadow_ShadowWordPain" setAllPoints="true"/>
        </Layer>
    </Layers>
    <Scripts>
        <OnLoad>
            self:RegisterForClicks("AnyUp", "AnyDown")
        </OnLoad>
        <OnClick>
            ChatFrame1:AddMessage(self:GetName() .. " - button: " .. tostring(button) .. " down: " .. tostring(down))
        </OnClick>
    </Scripts>
</Button>
```


#### `<OnUpdate>`

特殊的脚本  

在每次wow客户端 重新绘制 窗体时调用。  
因此，其仅在窗体被显示(但玩家不一定看到)时触发。


### 11.3 可用的窗体脚本


|脚本|描述|适用的窗体|
|--|--|--|
|OnChar(self, text)|窗体的enableMouse为true，所有的键盘输入将使用这个脚本发送到窗体中，第二个参数是 键盘输入的文本|所有窗体类型|
|OnClick(self, button, down)|单击按钮，按钮仅接受注册过的单击事件(:RegisterForClicks())，默认，只注册了LeftButtonUp|按钮，复选框|
|OnDoubleClick(self, button)|双击按钮，必须将按钮注册到 鼠标的 Up 部分 (down无法触发该脚本)|按钮，复选按钮|
|OnDragStart(self, button), OnDragStop(self, button)|拖拽事件，通过 self:RegisterForDrag()注册，参数是 鼠标键名称(如 LeftButton，Button4)的列表。用户按住并移动鼠标时触发 OnDragStart，释放鼠标时 OnDragStop|所有窗体类型|
|OnEnter(self, motion), OnLeave(self, motion)|窗体被设定为接收鼠标事件，这2个函数用来 处理这些事件。第二个参数 表明 动作是否由鼠标移动引起|所有窗体类型|
|OnHide(self), OnShow(self)|在窗体 被显示 或被隐藏时 调用。 一个隐藏的窗体可以继续接收事件，这可能不是我们所希望的，因此可以使用 OnHide 取消 注册的事件，并在窗体重新显示时 使用OnShow 重新注册事件|所有窗体类型|
|PreClick(self, button, down), PostClick(self, button, down)|并非所有的情况都可以使用 OnClick，因此提供了 这2个 来提供灵活性，它们的参数 和 OnClick的一样|按钮，复选按钮|




### 使用窗体方法改变窗体

每个带名称的窗体都可以通过 lua脚本 进行访问。

#### 常用方法

xml中，窗体，纹理，字体字符串 有 属性 及 子元素。  


- Object:GetAlpha()， 获得 透明度
- Object:GetName()， 获得 对象的名称，没有则返回 nil
- Object:GetObjectType()， 获得 对象类型的字符串
- Object:IsObjectType("type")， 对象是 给定的类型 则返回1， 否则返回 nil
- Object:SetAlpha(alpha)， 设置透明度， 0.0-1.0之间的值
- Object:Show()， 显示对象，这可能不会使其显示到屏幕上，例如，如果它没有被放置，或 父对象 被影藏
- Object:Hide()， 影藏对象
- Object:IsShown()， 对象正在被显示，返回1，否则 nil， 不一定意味着 屏幕上可见
- Object:IsVisible()， 如果 对象，及其 父对象，祖父对象 等 均被显示，返回1

有些函数 仅对 部分窗体有用，所有的方法查询 31章


### 创建并使用模板

wow提供了几百个不同的模板。


#### 新建xml模板

在WowXMLExample.xml 的 Ui 中
```xml
<Button name="IconTestTemplate" virtual="true">
    <Size x="32" y="32"/>
    <Layers>
        <Layer level="OVERLAY">
            <Texture name="$parentIcon" file="Interface\Icons\Spell_Shadow_ShadowWordPain" setAppPoints="true"/>
        </Layer>
    </Layers>
    <Scripts>
        <OnLoad>
            self.Icon = getGlobal(self:GetName() .. "Icon")
        </OnLoad>
        <OnEnter>
            self.Icon:SetDesaturated(true)
        </OnEnter>
        <OnLeave>
            self.Icon:SetDesaturated(nil)
        </OnLeave>
        <OnClick>
            ChatFrame1:AddMessage("you click on " .. self:GetName())
        </OnClick>
    </Scripts>
</Button>
```

鼠标移动上去，灰度化，即黑白， 移走后恢复

。。不清楚 Icon 是什么。最初的值是怎么来的？


#### 使用xml模板

WowXMLExample.xml 中，在上面的定义之后的 某处添加下面的

```xml
<Button name="IconTest1" inherits="IconTestTemplate" parent="UIParent">
    <Anchors>
        <Anchor point="BUTTOMRIGHT" relativePoint="CENTER" relativeTo="UIParent"/>
    </Anchors>
</Button>
<Button name="IconTest2" inherits="IconTestTemplate" parent="UIParent">
    <Anchors>
        <Anchor point="BUTTOMLEFT" relativePoint="CENTER" relativeTo="UIParent"/>
    </Anchors>
</Button>
<Button name="IconTest3" inherits="IconTestTemplate" parent="UIParent">
    <Anchors>
        <Anchor point="TOPLEFT" relativePoint="CENTER" relativeTo="UIParent"/>
    </Anchors>
</Button>
<Button name="IconTest4" inherits="IconTestTemplate" parent="UIParent">
    <Anchors>
        <Anchor point="TOPRIGHT" relativePoint="CENTER" relativeTo="UIParent"/>
    </Anchors>
</Button>
```

这会在屏幕中央创建一个 包含4个按钮的方格， 按钮的图标是 暗言术痛  
单击 会输出

新窗体可以 覆盖 继承的值


### 使用默认UI工具集模板

wow提供了几百个 用户界面模板， 这里介绍几个最常用的

- UIPanelButtonTemplate， 游戏按钮
- UIPanelButtonTemplate2， 游戏按钮，尺寸更易调整
- UIPanelCloseButton， 关闭按钮 (右上的X)
- InputBoxTemplate， 聊天窗口，也可以作为文本项
- UICheckButtonTemplate， 复选框
- TabButtonTemplate， 窗口顶部或底部，带有标签的按钮
- GameMenuButtonTemplate， Esc的菜单
- UIRadioButtonTemplate， 单选框
- OptionSliderTemplate， 滑动器(声音0-100)




## ch12 第一个插件 CombatTracker


### 需求

- 需要可以拖拽和移动
- 进入战斗时，保存当前时间 并显示 战斗中 状态
- 当用户攻击时，更新 战斗总时长，伤害总量，秒伤
- 战斗结束后，显示 统计结果
- 该窗体也是一个按钮，因此可以点击，点击后输出到屏幕


1. 找准游戏事件

要确定 为了完成这些功能，需要使用什么游戏事件。 这一步比较困难。  

CombatTracker 会涉及 3个事件

- PLAYER_REGEN_DISABLED， 进入战斗后，除了魔法，天赋，装备 外，无法 回复生命，所以这个事件 表明 战斗中。 无参数
- PLAYER_REGEN_ENABLED， 玩家正常回复生命，说明 脱离战斗。 无参数
- UNIT_COMBAT， 战斗中，只要玩家关注的单位 的生命值有所变化，这个事件就会触发。 参数：
  - unit， 发生改变的单位的标识符
  - action， 战斗的行为类型，如 HEAL, DODGE, BLOCK, WOUND, MISS, PARRY, RESIST
  - modifier， 如果行为是 攻击，那么可能是 一般攻击，重击，致命一击，分别是 GLANCING, CURSHING, CRITICAL。
  - damage， 收到的 伤害值 或 恢复值
  - damage type， 伤害种类
    - 0， 物理
    - 1， 神圣
    - 2， 火
    - 3， 自然
    - 4， 冰
    - 5， 暗影
    - 6， 奥术


有个 DevTools 插件可以用来跟踪事件。 `/dtevents`


### 插件架构

大多数插件是 单个lua文件 或 xml 即可。  

但是，我们希望 窗口的设计 和 插件行为 分开。

1. 在 AddOns中创建目录 CombatTracker
2. 在这个目录中 创建 CombatTracker.toc，包含如下内容
   ```text
    ## Interface: 20300
    ## Title: CombatTracker
    ## Description: xxxxxxx
    
    CombatTracker.lua
    CombatTracker.xml
   ```
3. 创建 CombatTracker.xml， CombatTracker.lua ， 2个空文件

重启wow 后，插件列表中可以看到 插件



### xml窗体

```xml
<Ui xmlns="xxxxxxxx">
<Button name="CombatTrackerFrame" parent="UIParent" enableMouse="true" movable="true" frameStrata="LOW">
    <Size x="175" y="40"/>
    <Anchors>
        <Anchor point="TOP" relativePoint="BOTTOM">
    </Anchors>
</Button>
</Ui>
```
























