
Jeff Erickson



[[toc]]

---
---



# ch0 Introduction


explicit, precise, unambiguous, mechanically-executable


algos(ἄλγος), meaning “pain”.




# ch1 Recursion

## 1.1 reduction 减少

将一个问题 X reduce 为另一个问题 Y，意味着 为 X 写一个算法，这个算法中 用到了 Y的算法作为 黑盒或子例程。
关键点是，X 的算法的正确性 不依赖于 Y 算法的具体实现。我们 唯一能做的就是 假设 黑盒能正确地处理 Y 。 我们并不关系 黑盒内部的 处理逻辑。


### 俄罗斯大数速算技术 Russian Peasant Multiplication

一种古老的数学运算方法,可以在不使用乘法表和计算器的情况下快速完成大数乘法运算

将2个数 分别除以2，然后 较小的数 取整减半，较大的数 取整翻倍， 循环操作，直到 一个数变为1。

常用于 实现 RSA 加密和解密算法。计算成本分析。网络负载平衡。


。。 下面来自这个网站，http://www.cut-the-knot.org/  R.I.P

如 64 * 61
= 32 * 122
= 16 * 244
= 8 * 488
= 4 * 976
= 2 * 1952
= 1 * 3904  余1 + 3904
3904 = 64 * 61

85 * 18 = 余1 + 18
42 * 36 = 
21 * 72 = 余1 + 72
10 * 144 = 
5 * 288 = 余1 + 288
2 * 576 =
1 * 1152 余1 + 1152

1152 + 288 + 72 + 18 = 1530 = 85 * 18

85 = 01010101
= 2^6 + 2^4 + 2^2 + 2^0
= 64 + 16 + 4 + 1

85 * 18 = (64 + 16 + 4 + 1) * 18
。。快速幂的做法。。。 6


### Huntington-Hill algorithm

baidu了下，又看到 cut the knot 了。

seat 分配。是那种 参议院 的 不同州 的 席位的 分配
1941.11.15 由 罗斯福 签署 入法。

已知 席位总数 y，需要分为 x 组：
1. 计算除数 D = y / x
2. 使用 d 来修改 D， d可能是负数，使得 每个州获得 (州人数) / (D + d) 个席位，这些席位 加起来就是 y

。。？ 感觉不对啊。 d 到底是什么？ d应该是要 遍历的，从1 - 无穷大。 来让 所有州的席位加起来 等于 y
。。不过，有可能 不同的 d，都可以加起来等于y，但是 席位分配不同？  
。没有， 不同的 d ，那么必然 有大小， 而 随着 d 变大， (人数)/(d+D) 是变小的，所以 不可能存在 多个 d，使得 sum{ (人数)/(d+D) } 相同的情况下，  有 2套 分配方案。

。还有，这个人数 是 万为单位？ 还是 个？ 或 千？ 百万？

列举了2 种 round
- webster's method，当 A 在 L 和 L+1 之间时，向上 还是 向下 round，取决于 A 是否 大于 或小于 (L + (L + 1)) / 2 。。 这个好像就是 四舍五入？ 但是 没有说等于 的时候 会怎么取啊。。
- method of equal proportions,  等于 sqrt(L * (L + 1)) 。。。。 ？ 这个 取整 不是必然等于 L 吗？
。。懂了。这里 通过 L 计算出来的值 是 cutoff 值， 就是 小于这个 就往下，大于就往上，  sqrt(4*5) = 4.47   sqrt(100 * 101)=100.4987

这里用的是 第二种 round。
。第二种 没有 相等的 可能。 6.   因为 sqrt(n*(n+1))， 除了n=0的时候 可以得到0， 其他时候 都是 无理数， 无理数不能写成 分数。 所以 (人数)/(d+D) 是 有理数， sqrt 是无理数， 不可能相等。    n=0 说明 值x 是 `[0,1)` 的， cutoff 是0，x 必然 大于等于 cutoff，。。好吧， 人数不可能为0， 所以 x 必然!=0， 所以必然 大于0 ，所以 round 后必然 为 1 .这样人数再少的州 也可以至少有一个 席位。

666

---



递归基于归纳法(induction hypothesis)

递归 减小 问题规模 直到一个base case， 这个base case 可以通过 其他方法来解决 (不然 会死循环)


tower of Hanoi

merge sort

quick sort



## ==1.7 recursion trees==

p31

递归树 用来 分析，可视化，分治问题


比如一个 分治算法，花费 O(f(n)) 时间 用于 非递归的work， 然后 进行 r次 递归调用，每次调用的 问题规模为 n/c。 不考虑固定系数，这个 算法的 运行时间的 增长公式 是 `T(n) = r * T(n/c) + f(n)`

递归树的 root 的值是f(n)，且有 r个children，每个 children 是 `T(n/c)` 子树的 root。

所有 这个递归树是 一个 完全 r叉树， 深度为 d 的 节点的 值是 `f(n/(c^d))`。 如下所示

![bf0da3ef18f2fe7041f1c57ecd01e444.png](../_resources/bf0da3ef18f2fe7041f1c57ecd01e444.png)


T(n) 是树中所有节点的值的sum。  第i层 有 `r^i` 个节点，每个节点 `f(n/c^i)` 所以有下面的公式。

![e93ccd57ce240c319404c9d8f89f1440.png](../_resources/e93ccd57ce240c319404c9d8f89f1440.png)

L是树的深度， base case n0=1，意味着 L = logn/logc, 因为 n/c^L = n0 = 1。

。。log以c 为底的 n 

树的 叶子个数 是 `r^L = r^(logn / logc) = n ^ (logr / logc)`， 所以最下面一层的 和是 n^(logr/logc) * f(1) = O(n ^ (logr/logc)) , 因为 f(1) = O(1)


level-by-level series 有3种情况
- 减少，如果指数下降，即每层常数倍变小，那么 `T(n) = O(f(n))`，此时，sum 受递归树的root的值 影响
- 不变，`T(n) = O(f(n)*L) = O(f(n)logn)`
- 增加，如果指数增长，即每层常数倍变大，那么`T(n) = O(n^(logr/logc))`，此时，sum 受 叶子节点数 影响

![54a43f64122586c0e1617f0d603be96a.png](../_resources/54a43f64122586c0e1617f0d603be96a.png)


---

`T(n) = 2T(n/2) + O(n)`  `T(n) = O(nlogn)`

---

如果 我们实现 一种快速排序，每次 pivot都是 1/3，那么
`T(n) <= T(n/3) + T(2n/3) + O(n)`
我们可以画几层递归树，很容易就发现， 所有的 层的 sum 最多是 n， 且 树最多 logn/log(3/2) 层，所以 `T(n) = O(nlogn)`


快速排序的 最坏情况是
`T(n) = T(n+1) + T(1) + O(n)`
是一颗完全不平衡的 递归树，
我们可以观察到: 每层的值最大时 n，且 最多n层， 所以 `T(n)=O(n^2)`


![3da53d4a478f1508ff25908c2c176767.png](../_resources/3da53d4a478f1508ff25908c2c176767.png)


---


## 1.8 linear time selection p53

线性时间 找到 无序数组的 中位数

1970年发现，使用 quickselect 的变种 可以 O(n) 找到 第k小的 元素。

quickselect 是1960年发现的。

---

quickselect

一般的 quickselect 算法选择一个 pivot(支点) 元素，和 quicksort一样 拆分数组，然后 递归  kth元素所在的 数组。

这个算法有 2个特点
- 和 quicksort 一样， 正确性 和 支点的选择无关。
- 即使我们只考虑 中位数 ( k = n/2 的特例)， 它也不会比 kth 简单。


最差时间复杂度是 O(n^2) ，即 T(n) = T(n-1) + O(n) , 即 T(n) = O(n^2)


good pivots

如果我们可以选择一个 好的 pivot，那么可以避免 最差情况。  
一个好的 pivot，可以让 问题规模以 等比(几何)级数 下降。 即 T(n) = T(an) + O(n), a < 1  

等比序列的和 是 S = a1 * (1-q^n) / (1-q)。因为q小于1，所以 是一个常数项。  
所以 T(n) = T(an) + O(n) = O(n)

Blum-Floyd-Pratt-Rivest-Targan (1970年 quick select的 5个人) 算法 可以得到一个 好的 pivot， 通过 对 输入数组 进行 特别筛选，然后 递归 来计算得到。

我们将 输入数组 分为 ceil(n/5) 个块，每个块 包含5个元素，最后一个块 用 无穷大 补满 5个元素。   
brute force 计算 每个块的 中位数，收集起来，形成数组 M   
然后 递归计算 M 的 中位数， 继续 按5个元素 组成一个块。 如果数组长度 < 25，则直接 brute force 计算中位数。(。。搜索pivot 和 搜索目标 都是这个逻辑( < 25 直接brute force))  
最终，我们得到 一个 块中位数的 中位数。  

![9d81d8ac29ad9e89ede8f40243989b25.png](../_resources/9d81d8ac29ad9e89ede8f40243989b25.png)


。mom ： median of median

momselect 为了2个目的 而使用递归
- 第一次是 为了选择一个 pivot
- 第二次是 在 pivot 的某侧 搜索 目标 。。?

the first time to choose a pivot element, 
and the second time to search through the entries on one side of that pivot

。。确实，每次进入方法的时候，都要 先 生成 数组M，然后对 M 计算 中位数， 然后才是对 数组A 计算中位数。  
。。当然 数组A 可能就是 上一层递归的 M


analysis

为什么快？  
第一个关键是 中位数的 中位数 是一个 好的 pivot。   
mom 大于 n/5/2 即 n/10 的 block median。 每个 block median 大于 它的block 中的 2个元素。 所以 mom 大于 3/10 的元素。    
同样，mom 也小于 3/10 的元素。 这样，在 最差的情况下， 第二次 递归时 最多需要搜索 7/10 的数据。

![8281fd633cb84085f9e16631b98c0a6c.png](../_resources/8281fd633cb84085f9e16631b98c0a6c.png)

![acb53d43664da94ad40a1fc37b793446.png](../_resources/acb53d43664da94ad40a1fc37b793446.png)


所以 mom 是一个 好的pivot， 但是 我们的算法 仍然使用 2次 递归，而不是一次递归。 如何证明 线性时间？  
第二个关键是 2次递归的 子问题的 总size 是 常数因子地 小于 原数组大小。  
最差情况是： T(n) = T(n/5) + T(7n/10) + O(n)  
下次递归最多 是 9/10 。 所以 指数下降，得到 T(n) = O(n)

![e4372a1925b05566a0eac80f288ca84a.png](../_resources/e4372a1925b05566a0eac80f288ca84a.png)

。。 block 3 好像没有用。无法降低 问题规模


sanity checking

此时，很多 学生会问： magic constant 5.   
为什么选择 5？  
因为 5 是 最小的 在递归树中可以 指数下降 的奇数 (block size)。  

隐藏在 O(n) 后面的 常数 非常大。 5个元素的 中位数 需要最多 6次比较， 所以我们需要 6n/5 次比较 来 建立 递归子问题。   
之前的 原始的分组 需要n-1 次比较， 但 现在，我们知道 3/10 的元素 大于 pivot ， 3/10 的元素 小于pivot，所以 只需要 2n/5 次比较。 因此，一个 更精确的 最差时间复杂度是 T(n) = T(n/5) + T(7n/10) + 8n/5.  
上界是 8n/5 * [1 + (9/10) + (9/10)^2 + ..] = 8n/5 * 10 = 16n

。。我以为只是计算下 中位数的 中位数，没有想到 还要 修改 数组，让 中位数的中位数 在 正确的下标上， 不然 没有办法 2n/5 次比较的。
。。 1 + 0.9 + 0.9^2 + .. = 1 * (1 - 0.9^n) / (1-0.9)， n 无穷大，就是 10.

实践中，中位数的中位数 并不会 如 最差时间复杂度 那样慢。 但是 在中等规模的 输入下，它依然 比 普通的quicksort 慢。  

事实上，实践中，正确的方法是 均匀随机选择 元素，然后计算 pivot



## 1.9 fast multiplication

上一章 介绍了 2种 古典算法，用来计算 2个数的乘，这2个算法都是 O(n^2) 的  
the grade-school lattice algorithm  
the Egyptian peasant algorithm

一个更高效的方法，通过 将 数字 拆分为 2半：  
(10^m * a + b)(10^m * c + d) = 10^2m * a * c + 10^m * (bc+ad) + bd

这样就可以 分治。分别计算 ac,bc,ad,bd

```
split_multi(x, y, n):
    if n=1
        return x*y
    
    m = (n+1)/2
    a = x/10^m, b = x mod 10^m
    c = y/10^m, d = y mod 10^m

    e = split_multi(a, c, m)
    f = split_multi(b, d, m)
    g = split_multi(b, c, m)
    h = split_multi(a, d, m)

    return 10^2m * e + 10^m * (g+h) + f
```

时间复杂度 `T(n) = 4T((n+1)/2) + O(n)`

`T(n) = O(n^(log4/log2)) = O(n^2)`


---

1960，Karatsuba 发现 `ac + bd - (a-b)(c-d) = bc + ad`
所以 bc+ad 可以从 2个乘法+1个加法 变成 1个乘法+4个加法。

这就是 Karatsuba算法
```
fast_multi(x, y, n):
    if n = 1
        return x * y
    
    m = (n+1) / 2
    a = x/10^m, b = x%10^m
    c = y/10^m, d = y%10^m

    e = fast_multi(a, c, m)
    f = fast_multi(b, d, m)
    g = fast_multi(a-b, c-d, m)

    return 10^2m * e + 10^m * (e+f-g) + f
```

`T(n) <= 3T((n+1)/2) + O(n)`
`T(n) = O(n^(log3/log2)) = O(n^1.58496)`

---

我们可以更进一步，将 大数 分的段更多。
Toom发现: 将大数分为 k 段，每段 n/k 个数字， 计算乘积 只需要 2k-1 个递归乘法。
Cook发现: 对于任何给定的 k， Toom-Cook算法可以 `O(n^(1+1/(lgk)))`

利用 这种分治策略， Gauss 发现了 Fast Fourier transform
。。。高斯，  快速傅里叶。

basic FFT是 `O(nlogn)`， 通过 FFT 可以改进 大数乘法。
1971， Arnold Schonhage，Volker Strassen，第一个FFT改进的 大数乘法，`O(n logn loglogn)`
2019，David Harvey，Joris van der Hoeven ， `O(nlogn)`



## 1.10 Exponentiation 幂

给定 数字a 和 正数n，计算 a^n

最原始的做法是 对 a 乘以 n-1次a。

a可以是 整数，浮点数，有理数， 甚至是 ==矩阵==。


---

通过 分治可以加速
```
pingala_power(a, n):
    if n = 1
        return a

    x = pingala_power(a, n/2);   // a = ( a^(n/2) )^2
    if n is even
        return x * x
    else
        return x * x * a
```

`T(n) <= T(n/2) + 2`
`T(n) = O(logn)`

---

另一种 分治方法 的思想来源于 上一章的 Egyptian peasant multiplication 算法(。。。没有)

```
peasant_power(a, n):
    if n = 1
        return a
    
    if n is even
        return peasant_power(a^2, n/2)   // a = ( a^2 )^(n/2)
    else
        return peasant_power(a^2, n/2) * a
```

`O(logn)`




# ch2 backtracking

。。原路返回， 回溯

71

本章讨论 另一种递归策略: backtracking

backtracking 逐步构建solution。 当需要决定哪个选择更好时，会 递归所有的选项，然后选择最合适的。


## 2.1 N Queens

典型的 backtracking问题 是 N皇后问题。  在 N*N 的棋盘上，放置N个皇后，有多少种方法。

```
place_queue(Q[1..n], r):
    if r = n + 1
        print Q[1..n]
    else
        for j = 1 to n
            legal = true
            for i = 1 to r - 1
                if (Q[i] == j) or (Q[i] == j + r - i) or (Q[i] == j - r + i)
                    legal = false
            if legal
                Q[r] = j
                place_queue(Q[1..n], r + 1)
```

。dfs


## 2.2 game trees

2个玩家的游戏，n* n 的棋盘，每个玩家有 n个 token， 游戏是 移动这些token 从原位置 到 对面。
2个人交替执行，移动一个格子，或 跳过一个已占用的格子。无法跳过2个格子。
谁先全部到达对面，谁先赢

![0a04fb0ef6b0554fb40f81d71d8d7561.png](../_resources/0a04fb0ef6b0554fb40f81d71d8d7561.png)

第一次遇到，可能没有 怎么玩的思路。
但是，这是一个 backtracking。每次最多3种可能

游戏的状态 包括 所有棋子的位置 和 当前的玩家。

![b7509c3c38463feb60de696d5d23e02f.png](../_resources/b7509c3c38463feb60de696d5d23e02f.png)

通过观察这个game tree，我们可以定义 状态是 好还是 差
- good，当前玩家可以赢 或 移动后 对于对方是 bad
- bad，当前玩家已经输了 或 每个移动，都会导致对方 good

即， 非叶子节点是good == 它有一个bad child。
非叶子节点是bad == 它的child都是good。

只要一旦处于 good状态，那么 就意味着 必然可以赢。

下面的代码用于判断 game state 是 good 还是 bad， 核心是 dfs。
对这个算法 进行稍微改动 就可以用来 find good move

```
playgame(X, player):
    if player has won in state X
        return Good
    if player has lost in state X
        return Bad
    
    for all legal move, x->y
        if playgame(Y, ~player) == Bad
            return Good
    return Bad
```

all game-playing program 都基于这种 简单的 backtracking策略。 但是，大部分游戏的 state非常多，所以 不可能遍历 整个game tree。
要借助 启发式策略 来 裁剪(prune) game tree


## 2.3 subset sum

给定一个 正整数集合X， 目标T， 询问: 是否存在X的 某个集合，它的 sum是 T。

2个不重要的边界条件， `T为0`，必然true，因为空集。 `T<0` 必然false，因为全是正整数


集合 X 中任取一个元素 x， 那么 X的子集的sum == T 等价于 下面的必然有一条为真
1. X的子集包含x，且sum为T
2. X的子集不包含x，且sum为T

对于第一种情况，X \ {x} 这个集合 中必然存在一个 子集，它 的sum是 T-x。
第二种情况，X \ {x} 这个集合 中必然存在一个 子集，它的sum是 T

所以，我们可以将 subset_sum(X, T) 这个问题 简化为 subset_sum(X \ {x}, T-x) 和 subset_sum(X \ {x}, T)

```
subset_sum(X, T):
    if T == 0
        return true
    else if T < 0 or X is_empty
        return false
    else 
        x = random choose element
        with = subset_sum(X \ {x}, T - x)
        wout = subset_sum(X \ {x}, T)
        return with or wout
```

正确性
通过 归纳法。
如果T是0，那么 空集的sum == T，所以 true
否则 如果T是负数 或X为空集，不可能==T，所以 false
否则 就根据 X \ {x} 与 T， T-x 的结果来作为 本次的结果。

分析
77
为了分析这个算法，我们需要对 细节 更进一步描述。
假设 输入集合X 就是一个 数组 `X[1..n]`

之前的算法是 从 X 中随机选择一个元素。  现在，我们指定 每次选择最后一个元素。 所以 X \ {x} 就是 `X[1..n-1]`。 复制数组太浪费了，所以传递 数组的地址 和 前缀的长度。

```
subset_sum(X, i, T):  // X[1..i]'s subset's sum == T ?
    if T == 0
        return true
    else if T < 0 or i == 0
        return false
    else
        with = subset_sum(X, i-1, T-x[i])
        wout = subset_sum(X, i-1, T)

        return with || wout
```

`T(n) <= 2T(n-1) + O(1)`

`T(n) = O(2^n)`

---

variant

经过一些小修改，可以处理 subset_sum 的数个变种。

下面是一个算法，展示了， 如何构造 X的子集，使得子集sum为T。
O(2^n)

```
construct_subset(X, i, T):
    if T == 0
        return {}
    if T < 0 or n == 0
        return None
    
    y = construct_subset(X, i-1, T)
    if y != None
        return y
    
    y = construct_subset(X, i-1, T-X[i])
    if y != None
        return y + {X[i]}
    
    return None
```


其他的可以通过 backtracking 解决的问题 都有 这个属性: 同样的递归策略 可以用来解决 该问题的 其他变种。
所以，我们可以先解决 最简单的变种，然后 解决该问题。



## 2.4 general pattern

backtracking算法 通常用来 构造 一系列的决定， 目标是 构造一个递归定义的结构 来满足 指定的条件。
通常来说，目标结构 是一个 序列，比如
- 在 N-皇后问题中，目标是 皇后位置的序列， 每行一个皇后，并且不会互相攻击。  对于每一行来说，算法需要 决定 皇后的位置
- game tree问题中，目标是 合法移动的序列，每次移动都是 最好的选择。 对于每个 game state，算法决定 最好的移动
- subset_sum问题中，目标是 部分元素的序列，它们有指定的sum。 对于每个元素，算法 决定 是否包含到 输出序列中。


在 backtracking算法的 每次 递归call中，我们必须 做出 exactly 一个决定，并且 我们的选择 必须 和 之前的决定 具有 一致性。
每次递归中，既要考虑 输入数据中 还未处理过的部分，还需要 考虑 已经做出的决定的 摘要。 这个摘要 要尽可能小。比如
- N皇后，我们 除了传递 空行的数量，还必须传递 已经放置好的 皇后。 这里，我们必须 记住 之前的决定的 所有细节。 。。。。。。如果传递 不可以的位置，那么就是一种 摘要。
- game tree问题， 我们传递 当前的game state，及 player的标识， 不需要 记住 之前的决定， 因为 从一个 给定的state，就可以计算谁可以赢，不需要了解 进入这个state的 move。
- subset_sum，传递 剩余的整数 和 需要的 target ( 原target 减去 已选择的元素的sum)， 之前选择了 什么元素，并不重要。


当我们设计 递归backtracking算法时，要弄清 我们需要 之前的决定的 哪些信息
如果这些信息 是 nontrivial， 我们的递归算法 可能必须要解决一个 更一般化的问题。

最后，一旦我们弄清 我们真正需要解决的 递归问题， 我们 通过 recursive brute force 来解决该问题: 尝试 下一个选择的 所有的 可能。 让递归处理 剩余部分。 不要跳过任何 看起来很蠢的选择。 try everything，之后再想办法 加速。


## 2.5 text segmentation (interpunctio verborum)

`@see 3.3`


连续文本分割成单词
。。就是 句读之不知。

假设我们有一个 方法， is_word，可以知道 字符串是否是 单词。

和 subset_sum 一样，输入的结构是 序列， 不过这次是 字母 而不是 数字。
我们要 从左往右 消费 字符。

类似地，输出结构是 word的序列。

当我们执行到一半: `blue stem unit rebot | hearthandsaturnspin`
我们可以组成 he，hear，heart，hearth， 但不知道选哪个。 我们只能 尝试所有的 可能性。然后递归，如果 递归返回 成功，那么 我们也成功。

我们现在有一个 简单的 backtracking策略: 选择第一个输出的单词，然后 递归分解剩余部分
要获得 完整的 递归算法，我们需要一个 base case， 我们的递归 中止，当 到达 字符串的 end时。

```
splittable(A[1..n]):
    if n == 0
        return true
    for i = 1 to n
        if is_word(A[1 .. i])
            if splittable(A[i+1 .. n])
                return true
    return false
```


---

index formulation

实践中，传递数组非常慢。 我们需要一个压缩方法 来 描述我们的 递归的子问题。 我们可以把 输入数组 当做 全局变量。传递 下标 而不是 子数组

对于 字符串拆分问题， 所有 递归调用的 参数 都是 原始数组的 后缀数组。 所以我们可以将 原始数组 作为 全局变量， 然后 重定义 我们的递归问题为: 
给定下标i，找到 `A[i..n]` 的一个 拆分

为了描述我们的算法，我们需要2个 bool函数
- 对于2个下标i,j ，is_word(i, j) 返回 该下标范围中的 子数组 是否是 单词。
- 对于下标i， splittable(i) ，如果 后缀`A[i..n]` 可以切分，那么返回 true。

这个算法和之前的一样，只是修改了一些符号
```
splittable(i):
    if i > n
        return true
    for j = i to n
        if is_word(i, j)
            if splittable(j + 1)
                return true
    
    return false
```

---

分析

![04fe4b81174ac5537c2f7575ec70d778.png](../_resources/04fe4b81174ac5537c2f7575ec70d778.png)


![08dda52556404577bdcf247dbcce9cdc.png](../_resources/08dda52556404577bdcf247dbcce9cdc.png)

最终可以化简为 `T(n) = 2T(n-1) + a`
`T(n) = O(2^n)`

---

variant

现在 我们有了 基础的 递归pattern。我们可以用来处理 一些变种。

如果字符串有多种拆分方法， 我们需要找到 最好的拆分。
如果字符串没有办法拆分，那么要返回 能找到的最好的拆分，而不是 返回失败。

为了满足这些需求，假设 我们有一个方法 score，接收 字符串，返回 数值， 比如 更长的 更常用的 word 的 score 更高。  我们要找一个 score sum最大的 拆分。

![96af25828c2401499136e9e9ffa041ee.png](../_resources/96af25828c2401499136e9e9ffa041ee.png)



## ==2.6 longest increasing subsequence==

给定序列S， subseq 是 删除0或多个元素，保持原顺序的 序列。

连续的 subseq 就是 subarray


最长上升子序列
最简单的做法是， 对于每个下标，决定 是否包含在 最终的子序列中。 如果当前值 小于等于 subseq的最后一个元素，那么肯定不选。 如果 大于，那么 选 或 不选 2个分支， brute force

关键的问题: 我们需要 记住 之前的决定的 哪些信息? 
如果我们确信 已选择的 subseq 是 上升的，那么 只需要 记录 最后一个元素的值 即可。

所以 递归 问题 变成了: 给定一个整数prev 和一个数组A，找到 A的 最长的上升subseq，且 这个 subseq 中的 元素 都大于 prev。

递归 通常需要 base case。 当到达数组end时，递归中止。

```
LIS(prev, A[1..n]):
    if n == 0
        return 0
    else if A[1] <= prev
        return LIS(prev, A[2..n])
    else
        skip = LIS(prev, A[2..n])
        take = LIS(A[1], A[2..n]) + 1
        return max(skip, take)
```

可以，但是 传递数组 是昂贵的。

prev必然是某个下标的值，所以下面的 i 代替

给定2个下标i,j， `i<j`， 在`A[j..n]`中找到 最长增长subseq，且 每个元素 都大于 `A[i]`


```
LIS(i, j):
    if j > n
        return 0
    else if A[i] >= A[j]
        return LIS(i, j + 1)
    else 
        skip = LIS(i, j + 1)
        take = LIS(j, j + 1) + 1
        return max(skip, take)
```

`T(n) <= 2T(n-1) + O(1)`
`T(n) = O(2^n)`


## 2.7 LIS, take 2

我们可以考虑 构造输出，每次构造一个 元素。

考虑 `输出序列的 下一个 元素在哪里`， 而不是  `A[i] 是否是 输出序列中的元素`

。。就是 原先是: `A[i]` 取 或 不取 
。改成，取哪个 。即 枚举哪些。 即， 必然取，只不过 取哪个 需要 本次遍历。

给定下标i，找到 `A[i..n]`中的 LIS，且 这个 LIS以 `A[i]` 开头

```
LIS(i):
    best = 0
    for j = i+1 to n
        if A[j] > A[i]
            best = max(best, LIS(j))
    return 1 + best
```


### LIS网上

https://blog.csdn.net/Deeven123/article/details/82930354

尝试为每个长度构造一个 LIS，如果相同长度，那么最后的元素 小的 胜出。

以 3,5,6,2,4,5,7 为里
初始时， 所有长度的 LIS 都是 空集。

1. 3
   LIS只有一个，`[3]`
2. 5
   LIS有2个，`[3]`, `[3,5]`
3. 6
   `3`, `3,5`, `3,5,6`
4. 2
   `2`, `3,5`, `3,5,6`
5. 4
   `2`, `2,4` 或 `3,4`, `3,5,6`
6. 5
   `2`, `3,4`, `3,4,5`     也可能是 `2,4`, `2,4,5`
7. 7
   `2`, `3,4`, `3,4,5`, `3,4,5,7`

最终LIS的长度是 4

思路: 每添加 新元素时，将这个元素 和 已有的 所有列表中的 尾元素 进行比较，如果 它大于 该尾元素，则 该列表 + 这个元素，组成 一个 长度为 sz1+1 的 列表，并和 sz1+1的 列表进行 比较。

。。看代码，只需要 保存 长度+尾元素， 不需要 前面的元素。。。

..但是这个 二分是什么意思?  似乎不是上面描述的那种。。
。。不过似乎是 之前我认为的 那种 LIS。 就是 nlogn的， 对每个元素，二分搜索 找到 可能的 下标位置。 这种方法应该是 正确的。 但是 书上没有提到。

```python
def LIS(nums):
    tails = [0 for _ in range(len(nums))]
    max_len = 0

    for x in nums:
        i, j = i, max_len
        while i < j:
            m = (i + j) // 2
            if tails[m] < x:
                i = m + 1
            else:
                j = m
        tails[i] = x
        max_len = max(i + 1, max_len)

    return max_len
```




## 2.8 optimal binary search tree

递归backtracking + 分治

二叉搜索树 中 成功的搜索的 运行时间 和 找到的节点的 祖先节点的 个数 有关。
所以，最差的 搜索时间是 树的 深度。
因此，要 最小化 最差搜索时间，就 需要 最小化 树的 深度。

理想的树是 完美平衡的

二叉搜索树有许多应用， 最重要的是 最小化 搜索的 total cost， 而不是 最小化 单个搜索的 最差情况。

假设 x 被搜索的次数 远多余 y，如果 构造一棵树，将 x 的深度 浅于 y， 那么 可以节约 时间。
。。splay tree。。

如果 有些节点 被访问次数 远多于 其他结点，那么 完美平衡二叉树 不是 最好的选择。

这种场景 导致了 下面的问题， 假设 我们有一个 有序的 保存了 key的 数组 `A[1..n]`， 和一个 与key对应的 保存了 访问频率的 数组 `f[1..n]`。
我们的任务 是 构造 二叉搜索树，来 最小化 搜索时间。

![4f9321195939386f5efa590e19dcf40d.png](../_resources/4f9321195939386f5efa590e19dcf40d.png)

假设 下标r 所在的 节点 是 子树的root，那么 对于该子树的 所有访问 都会 经过该root，所以 有cost。。 所以下面的 `连加f[i]` 就是 这个代价。

![115a91d76929944053b0c85b39619e3f.png](../_resources/115a91d76929944053b0c85b39619e3f.png)

![f068b34e59fe8243857b5c96894402e2.png](../_resources/f068b34e59fe8243857b5c96894402e2.png)

![796ee85db80790cc8d1a58ac5f45d841.png](../_resources/796ee85db80790cc8d1a58ac5f45d841.png)


---

分析

![94a49a2d219573a9b3321a9885d9087f.png](../_resources/94a49a2d219573a9b3321a9885d9087f.png)


![1c99ad4ead9e7d2f7baf554f63857f50.png](../_resources/1c99ad4ead9e7d2f7baf554f63857f50.png)

![b179e88c906670ae8d356971ab40deb0.png](../_resources/b179e88c906670ae8d356971ab40deb0.png)

![3d1a6e204681ce5aee2b9b4a0575b43b.png](../_resources/3d1a6e204681ce5aee2b9b4a0575b43b.png)






# ch3 dynamic programming


## 3.1 fibonacci

backtracking的代码
```
fibo(n):
    if n == 0
        return 0
    else if n == 1
        return 1
    else 
        return fibo(n-1) + fibo(n-2)
```

`O(((sqrt(5) + 1) / 2) ^ n)` = `O(1.61803 ^ n)`


---

memo: remember everything

```
memFibo(n):
    if n == 0
        return 0
    else if n == 1
        return 1
    else 
        if F[n] is undefined
            F[n] = memFibo(n-1) + memFibo(n-2)
        return F[n]
```

`O(n)`


---

dynamic programming: fill deliberately

我们已经看到 F数组 如何填充， 现在我们可以 通过简单的循环 而不是 递归 来填充数组

```
fibo(n):
    F[0] = 0
    F[1] = 1
    for i = 2 to n
        F[i] = F[i - 1] + F[i - 2]
    return F[n]
```

---

don't remember everything

在许多 dp算法中，不是必须 保留所有结果的。

```
fibo(n):
    prev = 1
    curr = 0
    for i = 1 to n
        next = curr + prev
        prev = curr
        curr = next
    
    return curr
```



## 3.2 aside: even faster fibonacci


![bfadd2cf51bcb54c12fe46897e82e547.png](../_resources/bfadd2cf51bcb54c12fe46897e82e547.png)


![b69f9ca6b859b2b8e5afb68ad9e2b2dd.png](../_resources/b69f9ca6b859b2b8e5afb68ad9e2b2dd.png)


。。矩阵可以 快速幂。


![0425ceceaf361ee7d381427d08d0df3e.png](../_resources/0425ceceaf361ee7d381427d08d0df3e.png)


![08b98b2581605beebdc89db3a692ab4f.png](../_resources/08b98b2581605beebdc89db3a692ab4f.png)


`O(logn)`




## 3.3 interpunctio verborum redux

`@see 2.5`

。。在使用拉丁语时，在单词之间插入连字符的规则

考虑上一章的 分词 问题， 有字符串 `A[1..n]` 和一个 is_word，我们想要知道 A是否可以 拆分成 word 的序列。

我们通过 定义 返回 后缀`A[i..n]`是否可以拆分为 word的序列的 splittable(i) 方法 来 解决该问题。

对于给定的 `A[1..n]`，一共有 n种可能 调用 splittable(i)， 一共有 n^2 种可能 调用 is_word(i,j)

每个递归的子问题 可以被 `[1,n+1]` 中的 一个整数 来标识，所以可以 memo。

```
splittable(A[1..n]):
    splitTable[n + 1] = {true}

    for i = n to 1
        splitTable[i] = false
        for j = i to n
            if is_word(i, j) and splitTable[j + 1]
                splitTable[i] = true
    
    return splitTable[1]
```

`O(n^2)` 调用 is_word

。原先是 `O(2^n)`



## ==3.4 the pattern: smart recursion==

简单来说， dp 就是 没有重复的 递归。

dp保存 子问题的 solution，通常 使用数组保存。

有2种独立的方法 来进行dp
1. 递归定义问题的表达式。 即 使用 子问题的结果 来 解答 当前问题
   1. specification，用文字 递归地 描述 你想解决的问题
   2. solution，对于整个问题 给出一个 清晰的 递归公式或算法。
2. 自底向上构建 问题的重现 的解决方案。 写一个算法 从 base case 出发，直到 最终的solution
   1. 识别子问题， 2次递归，有什么不同?
   2. 选择memo的数据结构， 保存上一步的子问题的结果
   3. 识别依赖， 除了base case，子问题还依赖其他的子问题吗?
   4. 找到合适的推导顺序， 在依赖的子问题解决完后 再解决子问题
   5. 分析空间和时间复杂度， 独立子问题的数量 决定了 memo算法的空间复杂度。
   6. 写出算法


## 3.5 warning: greed is stupid

107

我们可以绕过所有的 重复子问题，memo， 通过 greedy算法 来解决问题。

对于 backtracking算法， greedy算法直接 构建结果，而不需要使用 递归子问题。

能通过 greedy 就正确 解决的 问题很少。 

对于大部分应该通过 backtracking 或 dp 来解决 的问题， 很多学生的 第一反应 就是 使用 greedy。

即使 greedy可以解决问题，你也应该使用 backtracking 或 dp 算法。 因为后者 更 productive。


## ==3.6 Longest Increasing Subsequence==

上一章，计算 LIS时， 有2个 递归backtracking 算法。  都是 O(2^n)。

可以通过 dp 来加速

---

first recurrence: is this next?

我们的第一个 backtracking算法 是 `LIS(i, j)`， 这个方法的返回是 `A[j..n]` 中 每个元素都大于 `A[i]` 的 LIS 。

![37e1ba4637f02acd96ab262baa38a9c4.png](../_resources/37e1ba4637f02acd96ab262baa38a9c4.png)

我们还需要增加 `A[0] = INT_MIN`

递归子问题 可以通过 i,j 来唯一标识， 所以 有 `O(n^2)` 个子问题。 我们可以通过 二维数组 来 memo。 每个子问题可以 `O(1)` 解决，所以 最终是 `O(n^2)`

memo的填充顺序 并不明显，我们只能说 `LIS(i,j)` 在 `LIS(i, j+1)` 和 `LIS(j, j+1)` 之后 填充。
但是这些信息 已经足够 告知我们 一个 有效的 推导顺序。
当我们在 二维数组上 填写子问题的结果时， 该子问题依赖的 那些子问题 必须已经 解决。

```
LIS(A[1..n]):
    A[0] = INT_MIN
    for i = 0 to n
        LISarr[i, n+1] = 0
    for j = n to 1
        for i = 0 to j-1
            keep = 1 + LISarr[j, j+1]  // 需要的都是 j+1 列的，所以 i可以从0开始。
            skip = LISarr[i, j+1]
            if A[i] >= A[j]
                LISarr[i, j] = skip
            else
                LISarr[i, j] = max(keep, skip)
    return LISarr[0, 1]
```

由于只使用了 j+1 列，所以 可以使用 2个一维数组， 空间复杂度为 O(n)


---

second recurrence: what's next?

在我们的 第二个 backtracking算法中 `LIS(i)`， 返回 `A[i..n]` 中 以`A[i]` 为起始的 LIS 的长度。

![042850e21c634111c8fc2ad49ce1d269.png](../_resources/042850e21c634111c8fc2ad49ce1d269.png)

这里，我们假设 空集的 max 是0。这样 base case 可以自动递归计算。
还需要增加 `A[0] = INT_MIN`

这个例子中， ==子问题 通过 下标i 来标识。 所以我们可以 使用 一维数组 进行 memo。==

`LIS(i)` 只依赖了 `LIS(j), j>i`。 所以可以通过 下标降序 来遍历。
我们 需要 所有 大于 i 的 下标的 LIS值， 但不关注具体顺序。

```
LIS(A[1..n]):
    A[0] = INT_MIN
    for i = n to 0
        LISarr[i] = 1
        for j = i+1 to n
            if A[j] > A[i] and 1+LISarr[j] > LISarr[i]
                LISarr[i] = 1 + LISarr[j]
    return LISarr[0] - 1
```

时间 `O(n^2)`
空间 `O(n)`

。。。看了代码，似乎记得了。。


---

## 3.7 edit distance

2个字符串的 edit distance 是 最小的操作次数(插入，删除，代替) 使得 一个字符串变成另一个字符串。

`food -> mood -> mond -> moned -> money` food 和 money 的 编辑距离是 4

Valdimir Levenshtein

f o o   d
m o n e y
上下差了4个
对于长的字符串，很难发现 最短的编辑距离

。。所以是 最长公共子串? 好像确实是这样计算的。


---

recursive structure

要通过dp 解决 这个问题，我们首先需要 递归定义问题。

假设我们已经有 2个字符串的 最短编辑距离，那么 如果我们移除 last column，剩余的 column 仍然是 剩余 前缀的 最短编辑距离。

证明: 如果 前缀有一个更短的编辑距离，那么 加上后缀后， 会得到一个 更短的编辑距离。

。。last column 是什么? 是指 2个字符串都移除 x个后缀? 还是可以 一个字符串移除x个，另一个移除0个?
。。而且这个证明 感觉 不靠谱啊。

从右往左计算，对于输入 `A[1..m]` `B[1..n]`，定义 `Edit(i,j)`是 `A[1..i]` `B[1..j]` 的最短编辑距离。 所以 `Edit(m,n)` 就是最终答案。

---

recurrence

i 和 j 都是正数时，`A[1..i]`，`B[1..j]` 的操作 有3种选择
- insert，字符串A在该位置是空的，所以需要 insert，所以 距离是 `edit(i, j-1) + 1` 。。。就是 不选择A的字符，选择B的字符， 所以 需要一次insert， 并且 是 (i, j-1)。 因为 j 被选择了。 就是 insert 的就是 `B[j]` 这个字符。。 。。 不过 如果 选择A，不选择B呢，这里似乎没有说到。。。不，就是下面的 delete，  所以任何insert 都可以改为 delete?  不不不，我们是 要把 A 转为 B，所以 不选A，选B时 ( 表明A中没有，但是B中有)，需要 insert，， 选A，不选B (说明A中有，B中没有)，需要delete。
- delete，A中有，B中没有，需要删除，`edit(i-1, j) + 1`
- substitute，A，B都有，但是不同，所以需要 转换， `edit(i-1, j-1) + 1`, 如果相同，那么 不需要+1

当i=0 或 j=0时， 递归终止。 `edit(0, j) = j`  `edit(i, 0)=i`。 如果B是空白，那么A要全部删除。 如果A是空白，那么A需要insert成B。

![679ddbb0be8a5667aaf3796c28149924.png](../_resources/679ddbb0be8a5667aaf3796c28149924.png)


---

dynamic programming

现在我们已经有了 递归方程， 我们可以 将它转为 dp算法
- subproblem，每个递归子问题 通过 2个下标 i,j 标识
- memo structure，我们可以使用 二维数组 来保存所有的 edit(i,j) 的值
- dependency，`edit(i,j)` 依赖了 `edit(i-1,j-1)`,`edit(i-1,j)`,`edit(i,j-1)`。。。分别是 转换，删除，添加
- evaluation order，填充二维数组的数据时，使用 最常用的 从上往下，每行从左到右 的顺序，那么在填充时， 它所依赖的 项 已经填充了。
- space and time，memo结构使用了 O(mn)。 我们可以在 O(1) 时间内 计算 `edit[i,j]` 所以 算法时间复杂度是 O(mn)


```
edit_distance(A[1..m], B[1..n]):
    for j = 0 to n
        edit_arr[0, j] = j
    for i = 1 to m
        edit_arr[i, 0] = i

        for j = 1 to n
            ins = edit_arr[i, j-1] + 1
            del = edit_arr[i-1, j] + 1
            if A[i] == B[j]
                rep = edit_arr[i-1, j-1]
            else
                rep = edit_arr[i-1, j-1] + 1
            edit_arr[i, j] = min(ins, del, rep)
    return edit_arr[m, n]
```


## ==3.8 subset sum==

上一章中，已经有了 subset sum 的 递归版本。

我们假设 `SS(i, t)`， 返回 true，如果 `X[i..n]`中有 subset 的 sum 是 t。

![cba65fe08cbd1fd6907999d716e9ee6a.png](../_resources/cba65fe08cbd1fd6907999d716e9ee6a.png)
。这里要注意，通过 `t-X[i]` 小于0 ，那么不能选的。

我们可以转换这个 递归 为 dp
- subproblem，每个子问题可以通过 一个 下标i 和 一个小于T，大于等于0的 整数t 来表示。
- data structure，memo使用二维数组`S[1..n+1, 0..T]`
- evaluation order，每个`S[i,t]` 依赖 最多2个其他项，且这2个项是 `S[i+1, ?]`。所以可以 从最后一行向上 进行计算。
- space and time，memo是 `O(nT)` 空间。 如果 `S[i+1, t]`， `S[i+1, t-X[i]]` 都已经计算出来，那么 O(1)计算出 `S[i,t]`， 所以 时间是 `O(nT)`


```
subset_sum(X[1..n], T):
    S[n+1, 0] = {true}
    for t = 1 to T
        S[n+1, t] = false

    for i = n to 1
        S[i, 0] = true
        for t = 1 to X[i]-1
            S[i, t] = S[i+1, t]  // 子集中没有 X[i]， t是 subset sum，由于 t 小于 X[i]，所以 必然不能 把 X[i] 加入到 子集中。
        for t = X[i] to T
            S[i, t] = S[i+1, t] || S[i+1, t-X[i]]  // 这里可以将 X[i] 添加到子集。
    
    return S[1, T]
```

时间 O(nT)，之前是 O(2^n)



## 3.9 optimal binary search tree

上一章中 讨论的 optimal binary search tree问题。
输入是 搜索的key 的有序数组 `A[1..n]`， 及 对应的 搜索次数 `f[1..n]`

我们的目标是 构建一个 二叉搜索树，使得 执行这些搜索的 总cost 最小。

假设 方法 `cost(i, k)` 返回 子数组`A[i..k]` 的 optimal search tree 的 total search time。

![bca1fb715852b8aa7373df232a348e0e.png](../_resources/bca1fb715852b8aa7373df232a348e0e.png)

。。`连加f[i]` 是指，该子树中的 所有请求 都会经过 子树的root，所以 需要代价。
。。然后 后面就是 min，就是 无论选哪个节点作为root，`连加f[i]` 的值不会变， 所以 只需要 后面的 相加后 最小，那么就 选择该节点作为 root。


对于 每对i,k, 定义 `F(i,k)` 是 `A[i..k]` 中所有key 的 访问次数。
即 `F(i, k) = 连加f[j], j从i到k`

通过递归，上面的公式就写成

![c9b52f0f6a2f589724f6540814af7163.png](../_resources/c9b52f0f6a2f589724f6540814af7163.png)

我们可以在 `O(n^2)` 内 计算 `F(i, k)`

```
init_f(f[1..n]):
    for i = 1 to n
        F[i, i-1] = 0
        for k = i to n
            F[i, k] = F[i, k-1] + f[k]
```

这样，可以把 之前 截图中的 OptCost 中的 `连加f[i]` 替换为 `F[i, k]`


- 子问题，每个子问题可以通过 2个整数 i,k 来 标识
- memo，二维数组 `arr[1..n+1, 0..n]` 保存 所有的 `OptCost(i, j)` 的计算结果
- 依赖，`OptCost[i, k]` 依赖了 `OptCost[i, j-1]` 和 `OptCost[j+1, k]`，`j从i到k`, 即，依赖了 左侧 和 下方。
![232d0cdc9afffb66282f0befe90dfdf8.png](../_resources/232d0cdc9afffb66282f0befe90dfdf8.png)
- 推导顺序，至少有3种不同的顺序。
![779d32ace9ed7a31adf167f2c0f04274.png](../_resources/779d32ace9ed7a31adf167f2c0f04274.png)
![9325510d5ca46d88cc71400cca1bb8bc.png](../_resources/9325510d5ca46d88cc71400cca1bb8bc.png)
- 时间 空间，O(n^2) 的空间， O(n^3) 的时间



## 3.10 dynamic programming on trees

到目前为止，我们的 dp的例子 都是 使用 多维数组 保存 递归子问题的 结果。

图的 independent set 是一个 点的子集，这个子集中的点 没有 边 (直接) 相连。 要在 图中找到 最大 independent set 并不容易。 这是一个 NP问题。 第12章中有进一步介绍。

在一些 经典的 图中， 最大independent set 可以很快找到。 比如，n个点的 tree，我们可以在 O(n) 时间找到 最大independent set

假设有一颗树 T， 点w 是 点v 的后代(descendant)，如果 从w到 root的 path 中包含了 v。

对于 T 中的 任意顶点 v，`MIS(v)` 代表了 以v为 root的 sub-tree的 最大independent set。
在这个 subtree中 不包含 root v 的 所有independent set 都是 v的子节点 作为 root的 subtree 的 independent set 的 union。 换句话说，包含 v 的 independent set 必须不能包含 v的子节点， 因为 包含 v的 孙子节点。
因此， `MIS` 遵循下面的 递归
![9b5d2bc5a0e7445e342427d53665fd84.png](../_resources/9b5d2bc5a0e7445e342427d53665fd84.png)

。。就是 要么 子节点的连加， 要么 root+孙子节点的连加

我们应该用什么 数据结构 来memo? 最自然的选择 就是 树T 它自身。 对于 T中的每个 节点，我们 将 `MIS(v)` 的值保存到 节点的 新属性上。

顺序，v的子问题 依赖了 v的儿子节点的子问题 和 v的孙子节点的子问题。
我们必须在 访问完 节点后，再访问 节点后 父节点， 所以 需要 post-order 遍历。

运行时间， 每个节点的 非递归的时间消耗 主要和 它的 儿子节点数量， 孙子节点数量 有关。  对于不同的节点，这2个 值(儿子节点数量，孙子节点数量) 相差很大。
但是我们可以这样分析: 每个节点 在 它的父节点，爷爷节点 上 进行了 cost。每个节点 最多 1个父节点，1个 爷爷节点， 所以 时间 是 `O(n)`

```
tree_mis(v):

    skipv = 0
    for v's child w
        skipv = skipv + tree_mis(w);

    keepv = 1
    for v's grandchild x
        keepv = keepv + x.MIS   // x.MIS 就是保存了 x的 tree_mis(x)。
    
    v.MIS = max{keepv, skipv}
    return v.MIS
```


通过定义 T的 2个方法，我们可以 得到 更简单的 线性时间的算法。
- `MISyes(v)`，v为root的subtree中 包含v的 最大independent set 的 size
- `MISno(v)`，v为root的 subtree中 不包含 v 的 最大independent set 的size

这样，我们就得到了 这2个方法的 递归定义

![9195f0f7446f1b7f3daf4f1c3dde29a8.png](../_resources/9195f0f7446f1b7f3daf4f1c3dde29a8.png)

我们可以增加 memo， 并且 后缀遍历 是 O(n)

```
tree_mis(v):
    v.MISno = 0
    v.MISyes = 1

    for v's child w
        v.MISno = v.MISno + tree_mis(w)
        v.MISyes = v.MISyes + w.MISno
    
    return max{v.MISyes, v.MISno}
```



# ch4 greedy algorithm


## 4.1 store file on tape

我们有 n个文件，需要 保存到 磁带上。  将来，用户 希望从 磁带行读取这些文件。

从磁带上读取 文件， 不同于 从 磁盘上读取。 从磁带上读取 必须 跳过 前面的 所有文件，这需要消耗 一段时间。

`L[1..n]` 保存了 每个文件的 长度。 访问 第k个文件的 代价是 `cost(k) = 累加L[i], i = 1 to k`

cost 表明: 要读取文件 k， 必须把前面的 文件 也扫描一遍。

搜索 随机文件的 期望cost是 
![bef134db88ebf99103631b054537b624.png](../_resources/bef134db88ebf99103631b054537b624.png)
。。cost的连加 / 文件个数。

如果我们修改 磁带上 文件的顺序，那么 也会 修改 访问文件的 cost，一些变得更贵，一些变得更便宜。

。。似乎就是 越小的文件 越前面。。。毕竟 greedy

如果我们希望 预期cost 最小，那么应该用什么顺序 保存到 磁带?
答案看起来很清楚: 按size 升序。

。。证明，类似 冒泡排序，相邻的 2个文件，如果前面的 文件 size较大，那么交换后， 前面文件的cost会降低， 其他不变，那么 预期也就 下降了。
。。比如2个文件 `[5,3]`，那么访问的 代价是 (5+8)/2, 如果 交换，变成 `[3,5]`，访问的代价是 (3+5)/2, 代价降低了。 。。 代价应该是 (0+5)/2, (0+3)/2, 依然降低了。
。。任取2个文件，如果前面的 大于 后面的，那么 进行 swap，可以知道 这个区间内的 所有文件的 cost 都下降了。。。 比如 `[5,2,3,4]` 选择了 5和3， 之前的代价是 `[0, 5, 5+2, 5+2+3]` ,就是 prefix sum， 5和3 swap后 `[3,2,5,4]` 前缀和 `[0, 3, 3+2, 3+2+5]` 。  对于 2来说，之前是 5， 之后是 3， 代价下降了。
。。对，不包含当前元素的 prefix sum，可以知道 第一个元素会出现 n-1次，第二个元素出现 n-2次，倒数第二个元素 出现1次，最后一个元素 不出现， 所以 肯定是 值越小，越放前面，这样 sum才最小。

---

如果每个文件 有各自的 访问次数，那么 怎么排序 才能让 总cost 最小?

我们之前已经证明了，如果 访问次数相同，那么 按 size 升序。

如果 文件的 size都相同，但 访问次数不同，那么应该按照 访问次数 降序。

如果 size 和 访问次数都是 不同的， 那么应该 按照 `size / 次数` 的值 进行 升序
。确实，size越大，越往后， 次数越多，越往前。

证明: 如果 有2个 相邻的 文件 a,b，a在前面，且a的 size/次数 > b。如果 我们交换 a b，那么 访问a的cost 增加了 `L[b]`， 访问 b 的 cost 减少了 `L[a]`。 分别乘上 它们的 次数 就获得了 `L[b]F[a] - L[a]F[b]`, 由于 a的 L/F 大于 b， 所以 `LbFa - LaFb` 小于0。 所以 交换后， cost 降低了。


---

## ==4.2 scheduling classes==

161

课程在 每周的同一天， 不同的课程 有不同的 开始时间和结束时间。 你要注册尽可能多的课程，但注册的课程不能重叠。
简单来说，你有2个数组 `S[1..n]`， `F[1..n]` 代表课程的 开始 和 结束时间， 数据确保 `0<=S[i]<=F[i]<=M`， 你的任务是 选择尽可能多的 下标，且不重叠。

。。这种是 dp吧? 就是 按 S 排序，然后判断 小于开始时间 有多少个不重叠课程。 。就是 上面的 3.8 的代码，   。。 或者是 mono stack?
。。贪婪怎么贪? 选择 持续时间最短的? 不行的。

这个问题 可以通过 递归 来解决， 你可以选择 课程1，或不选择课程1.  
选择课程1的话，你需要 计算出 结束时间 < 课程1的开始的 子集的 选课结果， 还有 开始时间 > 课程1的结束 的 子集的 选课结果
不选课程1，那么 就 对 剩余的课程 递归。
上面的算法 需要 `O(n^3)`

更好，更简单的算法:  我们肯定希望 第一门课程 越早结束越好。 这就是 贪婪 的策略。
按 结束时间 升序，遍历，发现 不冲突时，就选择。
排序后的遍历时 O(n), 所以 时间复杂度 是 排序的复杂度 `O(nlogn)`

```
greedy_schedule(S[1..n], F[1..n]):
    sort F, S by F
    count = 1
    X[count] = 1
    for i = 2 to n
        if S[i] > F[X[count]]
            count = count + 1
            X[count] = i
    return X[1..count]
```

上面的算法是 找到一个可行的方案，所以我们要证明的是: 至少有一个 最大 无冲突 选课 包含了 第一个结束的 课程。
证明，如果一个 最大无冲突选课 没有包含 最先结束的课程， 那么我们可以把 最先结束的课程 和 最大无冲突选课 中的 第一个结束的 课程 交换， 得到的 选课 依然是 最大无冲突的。


要证明: 贪心的就是最优的
假设贪心得到 `g1,g2..gk`， 最优的是 `g1,g2,..,gj-1,cj,cj+1..cm`,  由于 gj 是 开始时间大于 gj-1的 最早结束的， 所以gj 必然 小于 cj 所以 `g1,g2..gj-1,gj,cj+1..cm` 也是 无冲突的。


## ==4.3 general pattern==

对于 贪婪的 证明步骤 一般是
1. 假设有一个 最优解， 它和 贪心解 不同
2. 找到 这2个解的 第一个不同之处
3. 将 该位置的 贪心解 放到 最优解 的相同位置， 不会导致 最优解 变差。



## 4.4 huffman code

二进制码，是指 由 0 和 1 组成的 字符串。

二进制码 是 prefix-free 的， 如果 所有的 code 都不是 其他 code 的 前缀。

ascii码 是 prefix-free的，   。。估计是 英文字母 的 ascii。 不，ascii 都是 7bit， 所以 不存在 谁是谁的前缀。

prefix-free 的 二进制码 可以 通过 二叉树 可视化，并且 编码后的值保存在 叶子中。
二叉树的 深度 就是 二进制码的长度。
左子树0，右子树1

假设 我们希望将 一条消息 编码，越短越好。 即，给定 字母的出现次数 `f[1..n]`，我们要 创建一个 prefix free 的 二进制码， 来使得 编码后的 总长度 最小。

这个问题和之前的 optimizing binary search tree 一样。 但是 optimization problem不同， 因为 code tree 不要求 保持key的顺序。

Huffman发现了 贪心算法 来 创建这样的 code。
`Huffman: merge 2 least frequent letter and recurse`

Huffman的算法 最好通过例子来讲解，一段文字，为了简单，剔除 空格，引号，逗号 等字符后，得到下面的 次数统计。

![5ae0c0d4309b0961854cead4771bb27e.png](../_resources/5ae0c0d4309b0961854cead4771bb27e.png)

Huffman算法 选出2个 最少出现的，`Z`，`D`。 merge成一个新的字符，出现次数相加。 这个 新的字符 是 code tree的 一个 中间节点， Z 和 D 是它的 child。  然后继续递归。

19次 merge 后， 20个字母 都被merge到一起。 merge的记录 就是 我们的  code tree。
算法中 如果 出现次数都最少，那么可以 随机选择，所以 可能生成不同的 huffman tree。
下面是一种， `A`的编码是 `101000`， `S`是`111`

![4917fc8be0a1e1ad224675109f870802.png](../_resources/4917fc8be0a1e1ad224675109f870802.png)

。。左子树0， 右子树1


引理: 假设x，y是 2个出现次数最少的 字符，那么 最优code tree中， x和y 是相邻的兄弟节点。
证明: 我们证明一个 更强的结论: 存在一个 最优code tree，其中 x和y是相邻的，且 它们是 最深的叶子节点。
假设 T 是 最优code tree，并且 深度是 d，由于 T是 满二叉树，所以 深度d 至少有2个 叶子节点，且 是兄弟节点。假设这2个点不是 x，y，而是 a和b。
T' 是 交换 x 和a 后的 code tree， delta = d - depth(x)， 这个swap 使得 x的深度 增加 delta， 使得 a的深度 减少 delta， 所以 `cost(T') = cost(T) + delta * (f[x] - f[a])`。
我们之前假设 x 是 最少次数的 字母之一， 而 a 没有提到， 所以 `f[x] <= f[a]`， 所以 `cost(T') <= cost(T)`。 而且由于 之前假设 T是 最优code tree，所以 `cost(T') >= cost(T)`, 所以 `cost(T') == cost(T)`。 T' 也是一个 最优code tree
同理，交换 y 和 b， 也可以获得 一个 最优 code tree



引理: 每个huffman code 是 最优 prefix-free binary code
证明: 如果只有1个或2个字符，直接就是 最优prefix free binary code
如果大于三个，且 前2个是 出现次数最小的， 那么就分为 `f[1]`, `f[2]`, `f[3...n+1], f[n+1]=f[1]+f[2]`,  将这个 树的 n+1 节点 替换为 1 和 2. 得到的就是 `f[1..n]` 的 最优prefix-free binary code。


---

为了加速 huffman code， 我们使用 pri_que 来保存 字符出现次数。
我们可以用 3个数组 `L, R, P` 来表达 tree 的 每个节点的 left，right，parent。
最终 code tree的 叶子节点的 下标是 1到 n。 root节点的下标是 2n - 1。
对于 pri_que一共有: 2n-1次 insert， 2n-2次 extractMin。 如果使用 二叉堆 来作为 pri_que，那么 每次操作都是 O(logn)， 所以 最终 `O(nlogn)`

```
build_buffman(f[1..n]):
    for i = 1 to n
        L[i] = 0; R[i] = 0
        insert(i, f[i])
    for i = n to 2n - 1
        x = extractMin()
        y = extractMin()
        f[i] = f[x] + f[y]
        insert(i, f[i])
        L[i] = x; P[x] = i;
        R[i] = y; P[y] = i;
    
    P[2n-1] = 0
```

下面是 将 消息 编码，解码

```
huffman_encode(A[1..k]):
    m = 1
    for i = 1 to k
        huffman_encode_one(A[i])

huffman_encode_one(x):
    if x < 2n - 1
        huffman_encode_one(P[x])
        if x = L[P[x]]
            B[m] = 0
        else
            B[m] = 1
        m = m + 1
```


```
huffman_decode(B[1..m]):
    k = 1
    v = 2n - 1
    for i = 1 to m
        if B[i] = 0
            v = L[v]
        else 
            v = R[v]
        if L[v] = 0     // 左子树为空，说明是叶子节点，可以得到字符
            A[k] = v
            k = k + 1
            v = 2n - 1
```


## 4.5 stable matching

稳定匹配

在美国，新医生 必须实习。
医生提交一个 排名列表，里面是 他愿意去的医院。
医院也会提交一个 排名列表，里面是 医院愿意接收的 医生。
NRMP 会计算出 医生 和 医院的 匹配关系， 匹配是 unstable 如果 医生a 和 医院B 是更好的选择:
- 医生a 被匹配到了 医院A，但是 a 更喜欢 医院B
- 医院B 被匹配了 医生b，但是 B更喜欢 a

这种情况下 `(a, B)` 是 unstable pair。
NRMP的目标是 stable match， 即 没有 unstable pair。

进一步简化后， 有相同数量的 医生 和 医院， 每个 医院有一个 实习名额。 每个 医生 对 所有医院进行 排名， 每个医院 对 所有医生进行了 排名。  我们要找到 匹配关系。

---

Gale-Shapley算法 一直循环执行下面的操作，直到 匹配成功
- 随机选一个 未匹配的 医院 A，将它的 实习 发给 A最满意的 且 还没有拒绝过 A 的 医生 a
- 如果 a 没有匹配，那么 a 暂时 接受 A的 offer。 如果 a有匹配，但是 a更喜欢 A，那么拒绝 手中的 匹配，并 暂时 接受 A， 否则，a 拒绝 A

上面的算法: 医院 发出 offer 是 贪心的，  医生接受offer 也是 贪心的。  医生可以拒绝手中的offer 接受更好的 offer， 使得整个算法 可行。

---

运行时间

O(n^2)


正确性
。。。

最优性
。。。




# ch5 基础的图算法

## 概念

图是 pair 的集合。

pair的 对象 被称为 点/节点
pair的关系 被称为 边/弧

图 由 点集，边集 组成。

`{u, v}` 代表 u和v 之间有 无向边
`(u, v)` 代表 u 指向 v 的 有向边

0 <= 边的数量 <= V取2 (即V*(V-1) )

没有自环 和 重复边的 图 是  simple graph
non-simple graph 有时被称为 multigraphs

节点的 degree(度) 是 该节点的 邻居数

有向边`(u, v)`， u是v的前驱， v是u的后继。
in-degree 是 前驱的数量， out-degree 是后继的数量。

子图，就是 点是 原点集的 子集， 且 边 是原边集的 子集  (子集可以等于原集合)。
proper subgraph， 是 非原图的 子图

无向图的 一个walk 是 点的序列。这些点 是相邻的。
一个 walk 被称为 path，如果 它访问每个节点 最多一次。

v 是 从 u reachable，如果 图包含一个 u和v 之间的 walk
无向图是 connected，如果 每个 点 都可以 从其他点 reachable。
每个无向图 都有 一个或多个 component(连通分量)， component 是最大连通子图。

walk 是 closed，如果 起始 和 结束 是 同一个点。
cycle 是 closed 且 经过每个点最多一次 的 walk
无向图是 acyclic， 如果 没有子图是 cycle。
acyclic graph 也被称为 forests(森林)
tree 是 连通的 无环图， 或者说 是 森林的一个 连通分量。

spanning tree (生成树) 是 一个子图，这个子图 是 tree 且 包含 原图的所有 顶点。

图 包含 生成树，当且仅当 图是 连通的。

spanning forest 是 spanning tree的 集合。 原图的 每个连通分量 一个 spanning tree


有向图的定义 有些不同
directed walk， 是一个 顶点的序列， 前后2个顶点 是 有 有向边存在，且方向是 前面指向后面。
directed path, directed cycle 的定义 类似。

reachable，说明有 directed walk。
有向图 是 strongly connected， 如果 每个点 都可以从 其他 点 可达。

如果不包含 directed cycle，那么图是 acyclic

DAG: directed acyclic graph。

---

## 5.3 representation and example

图的最简单的表达 就是 把它们画出来。

图是 planar (平面的)，如果 画出来后，没有 边 相交， 这样的图 也称为 embedding

图 有不同的画法，所以要注意区分。

intersection graph
![48c90c1c8d1e0d9ca1a83c72305c55a7.png](../_resources/48c90c1c8d1e0d9ca1a83c72305c55a7.png)

下面的图是上面的图的 另一种 表达方式。
![362f7e750207e38a94eede26f2d3b38c.png](../_resources/362f7e750207e38a94eede26f2d3b38c.png)

另外一个 例子是 递归算法的 dependency graph

Fibonacci的依赖图
![c7351113f55db3845d9bd268997a4cf2.png](../_resources/c7351113f55db3845d9bd268997a4cf2.png)


第三章的 编辑距离 的 递归式 和 依赖图

![2ffe6e3c6691b88a0f370c7691a5b73e.png](../_resources/2ffe6e3c6691b88a0f370c7691a5b73e.png)

![b4883b7ff34794204c010f08c50eed37.png](../_resources/b4883b7ff34794204c010f08c50eed37.png)


---

另一个例子是 game的 configuration graph

![b4f1fafe8420a3b0b19d174b4435b402.png](../_resources/b4f1fafe8420a3b0b19d174b4435b402.png)


---

finite-state automata， 有限状态自动机。 是 formal language theory 中的概念，可以通过 带标签的 有向图 来表达。
`M=(Σ,Q,s,A,δ)`
194


## 5.4 Data structures

通常，图使用 2种表达之一， 邻接链表，邻接矩阵， adjacency list， adjacency matrix

这2个 都是 通过 顶点进行索引的 数组。 所以要求 顶点是 1 到 v


adjacency list

保存图 最常用的 数据结构 是 adjacency list。

标准的做法是 单链表， 但是也可以 使用 二叉搜索树， hash table。来保存 相邻的顶点。

邻接链表的 另一种常见实现 是 adjacency array。


adjacency matrix

二维bool数组，`[a][b]` 为true，说明 a b 有边相连

O(1) 知道 2个点是否 直接相连。
O(V) 获得 点的 所有邻接点。
需要 O(V^2) 的空间。


comparison

![041a3d81d25943f6e1e5438633bc24f5.png](../_resources/041a3d81d25943f6e1e5438633bc24f5.png)

使用hash的 邻接链表 和 邻接矩阵 有相同的速度，且 空间更少。

但是 邻接链表，邻接矩阵 更加符合 人的认知，并且 速度足够了。

本书中 默认 邻接链表。


## 5.5 whatever-first search

199

图 最常见的问题 是 可达性。
- 图中 哪些点 可以 从 点s 可达。
- 点s，点v，是否存在路径相连

从现在开始，我们只考虑 无向图。

对于无向图，哪些点 从点s 可达 这个问题 等价于 求s的 连通分量。

最简单的 可达算法，是 dfs。 dfs可以递归 也可以 循环。
```
recursive_dfs(v):
    if v is unmarked
        mark v
        for w in v's neighbour
            recursive_dfs(w)
```

```
iterative_dfs(s):
    stack_push(s)
    while stack is not empty
        v = stack_pop()
        if v is unmarked
            mark v
            for w in v's neighbour
                stack_push(w)
```

。。如果用 queue的话，就是 bfs 了。

dfs是 被我称为 whatever-first search 的 图遍历算法 的 重要一员。

图遍历算法 中最重要的就是 有个 bag，可以放进去，也可以取出来。  。。。这里说了 bag 存 候选边，但是 代码里 存的是 点。。 后续 analysis 的时候 也说 bag 存 边。。。
```
whatever_first_search(s):
    put s into bag
    while bag is not empty
        take v from bag
        if v is unmarked
            mark v
            for w in v's neighbour
                put w into bag
```

稍作修改，就可以 获得 路径。 我们将 前驱称为 parent

```
whatever_first_search(s):
    put (-1, s) into bag
    while bag is not empty
        take (p, v) from bag
        if v is unmarked
            mark v
            parent(v) = p    // 前驱
            for w in v's neighbour
                put(v, w) into bag
```

上面的 (v, parent(v)) 组成的 图 是 生成树 (spanning tree)  。。不是最小生成树。。不过 有权图 才有 最小生成树。

证明: 
1. 所有从s 可达的点 都被找到。  。。。证明好长，都是文字。。

2. 是tree。除s外 每个点只有一个 parent，所以 边的数量是 点的数量-1 。所以不可能有环。

---

analysis

图遍历算法 的运行时间 取决于 我们的 bag， 如果 bag的 insert 和 delete 的时间是 T。
循环 会尝试遍历 所有 顶点，所以 最多 V 次。
每条边 都会被 存放到 bag 中 2次，所以 会执行 2E 次 。。。。。之前也确实说 bag 存放边，但是 bag 应该存放 点啊。有点无法理解。

所以时间是 `O(V + ET)`， 如果 使用邻接矩阵保存图，那么会 `O(V^2 + ET)`



## 5.6 important variants


stack: depth-first

如果我们使用 stack 来实现 bag，那么会得到 dfs。
stack的 insert 和 delete 是 `O(1)`的， 所以 算法是 `O(V+E)`

用这个算法 获得的 生成树 被称为 depth-first spanning tree

树的形状 取决于 起始点 和 邻居点 的存放顺序。

通常来说， depth-first spanning tree 是 long 和 skinny  ( 长和瘦， 就是树很深，每层 节点很少)

ch6 会讨论 dfs 的一些 重要属性 和 应用


---

queue: breadth-first

使用 queue 实现 bag， 就获得了 另一种图的遍历算法: bfs

queue的 insert 和 delete 是 O(1)， 所以 `O(V+E)`

bfs 生成的 breadth-first spanning tree 是 从s开始 到每个点的 shortest path。

ch8 讨论 最短路径。

breadth-first spanning tree的形状依赖于 开始顶点 和 邻居顶点的顺序。

通常来说， breadth-first spanning tree 是 short 和 bushy


---

priority queue: best-first

如果我们使用 pri_que 实现bag。  我们会获得 best-first search。

由于 pri_que 保存了 每条边的 副本， insert，delete 需要 `O(logE)`。 所以 best-first search 是 `O(V + ElogE)`

如果 图是 有权无向图， best-first search 会构造 ==minimum spanning tree== 。

如果 权 都是 唯一的，那么 mini spanning tree 的形状也是唯一的。

这个算法 通常称为 ==Prim algorithm== ， ch7 讨论。

---

将 path的 length 定义为 path上边的 权的 sum， 那么 使用 best-first search 可以 计算出 最短路径。 被称为 ==Dijkstra algorithm== ，ch8 讨论

---

将 path的 width 定义为 path上的 最小的权。

对 Dijkstra 进行修改 就可以 计算 从s出发 到 所有可达点的 widest path。
widest path 也被称为 bottleneck shortest path。

每个 marked 点v 保存了 一个值 width(v)。
最开始时， 设置 width(s) = INT_MAX。 对于其他点， 设置 `parent(v) = p` 时 也设置 `width(v) = min(width(p), w(p->v))`。 保存到 pri_que的时候 使用 `min(width(v), w(v->w))` 作为 优先级。
==widest path 用于 计算 最大流==， ch10讨论

---

Disconnected graphs

whatever-first search， 只访问 从某个点出发 可达的 点。

要访问 所有点，需要外面加一层包装
```
wfs_all(G):
    for all v
        unmark v  // 不要问，这里不是已经访问全部节点了吗?
    for all v
        if v is unmarked
            whatever_first_search(v)
```

计算 连通分量个数
```
count_components(G):
    count = 0
    for all v
        unmark v
    for all v
        if v is unmarked
            count = count + 1
            whatever_first_search(v)
    return count
```

再修改一下，可以计算 每个节点 所处的 连通分量

```
count_and_label(G):
    count = 0;
    for all v
        unmark v
    for all v
        if v is unmarked
            count = count + 1
            label_one(v, count)
    return count

label_one(v, count):
    put v into bag
    while bag is not empty
        take v from bag
        if v is unmarked
            mark v
            comp(v) = count   // 指定v 的 连通分量
            for each edge vw
                put w into bag
```


wfs_all 会 mark 每个点 因此， put 每条边 到 bag 一次。 从bag 拿去 每条边 一次， 所以 运行时间是 `O(V + ET)`， T是 bag的操作的时间。 如果使用 dfs 或 bfs，那么 需要 `O(V+E)`


whatever_first_search 计算了 分量的 spanning tree， 所以使用 wfs_all 可以计算 整张图的 spanning forest。
使用 将 边的权 作为 优先级的 pri_que 的 best-first search 可以计算 minimum-weight spanning forest， 时间是 `O(V + ElogE)`

---

### directed graphs

whatever-first search 很容易就可以 适配 有向图。

唯一的修改点是， mark 点时，将 所有 出边的点 放入bag
事实上，如果使用 标准的 邻接链表 或 邻接矩阵， 完全不需要修改 代码

```
whatever_first_search(s):
    put s into bag
    while bag is not empty
        take v from bag
        if v in unmarked
            mark v
            for each edge v->w   // v 指向 w
                put w into bag
```

之前已经证明了， 算法可以 mark 所有 从 s 可达的 点。 且 `parent(v)->v` 定义了 一个 有根树。

但 我们无法获得 spanning tree，因为 有向图的 可达性 没有 对称性。


---

whatever_first_search 定义了 从s可达的 点的 spanning tree。 通过 bag的不同实现，我们可以获得 
- depth-first spanning tree, 
- breadth-first spanning tree, 
- minimum-weight directed spanning tree
- shortest-path tree
- widest-path tree


## 5.7 graph reduction: flood fill

whatever-first search 的 最早的应用 是 1950年。
pixel map 是一个 二维数组。。。。

![e29d8a9310ce4572277a79f17d24f815.png](../_resources/e29d8a9310ce4572277a79f17d24f815.png)

flood-fill 问题 可以 降为 可达性问题。

我们可以定义一个无向图，每个 pixel 就是一个 点， 相邻的 pixel 有 边。 这样就把 问题转为 可达性问题

对于 n*n 的 pixel map， 图最多有 n^2 个点，2n^2 条边， 所以 whatever-first search 的 时间复杂度是 `O(V+E) = O(n^2)`

这个例子 展示了 reduction 的本质，不是 直接处理flood-fill，而是使用 已有的算法 作为 黑盒子程序。 。。。。

我们还有2个方法可以加速
- 实现时，不需要构建 图，而是直接使用 pixel map 当做 标准邻接矩阵。
- 进一步分析 显示， 运行时间 和 被fill的 pixel的数量 成正比。



# ch6 depth-first search

dfs可以描述为 使用stack的whatever-first search， 但是 多数实现 是 递归的

```
dfs(v):
    if v in unmarked
        mark v
        for each edge v-w
            dfs(w)
```

我们可以加速算法，通过 递归前 检查 node是否是mark。 这个改动可以 确保 对每个 点 只调用一个 dfs
我们可以进一步修改，来计算 边 点 的有用的信息，通过 2个 黑盒子程序: preVisit，postVisit

```
dfs(v):
    mark v
    preVisit(v)
    for each edge v-w
        if w is unmarked   // 先检查，再dfs
            parent(w) = v
            dfs(w)
    postVisit(v)
```


---

有向图中 从 v 可以到达w 定义为， 图包含一个 有向path，起点是v 终点是w。

reach(v) 是 从v 可达的 点的集合(包括v子集)， 那么只需要 unmark所有点，然后 调用 dfs(v)， 被mark的点 就是 reach(v) 中的点。

---

无向图中 可达性 是 对称的: v可以到达w  当且仅当 w可以到达v。
所以 unmark 所有点后， 调用 dfs(v) 遍历 v的 整个连通分量，`parent(v)=w` 这个关系 就是 这个连通分量的 一个 spanning tree

---

有向图 更加微秒。
从不同点出发，会得到 不同的 树。
而且 dfs 后的 reach(v) 无法用来生成 spanning tree


![486892aeafacb4d62922d541b409c37f.png](../_resources/486892aeafacb4d62922d541b409c37f.png)


---

我们可以在外面加一层wrapper function 来 遍历 整个图，而不是 某个连通分量

在 dfs_all 中可以增加 preprocess 来处理 dfs中 用到的 preVisit，postVisit 方法所需要的 东西

```
dfs_all(G):
    preprocess(G)

    for all v
        unmark v
    for all v
        if v is unmarked
            dfs(v)
```
```
dfs_all(G):
    preprocess(G)

    add vertex s
    for all v
        add edge s->v    // ? 这是什么操作?
        unmark v
    dfs(s)
```
根据原文: 这2个 都是 dfs 的标准的 wrapper function。 但是 第二个是什么意思? 下面有说明

如果我们可以修改 图，我们可以增加一个 新的 source vertex s， 并且 从s 指向 图中所有的 点。 然后 只需要 调用 一次 dfs。 不过生成的 spanning tree 不是 原图的

---

对于无向图，上章可以看到 很容易就 适配 dfs_all 来计算图的 连通分量个数。  parent(v) 是 图的 spanning forest

对于有向图， dfs_all 可能找到 很多 连通分量，但是 实际上 图是 连通的， 这取决于 图的 数据结构 及 wrapper function中的遍历顺序。


## 6.1 preorder and postorder

都是通过 dfs 来实现的。

```
dfs_all(G):
    clock = 0
    for all v
        unmark v
    for all v
        if v is unmarked
            clock = dfs(v, clock)

dfs(v, clock):
    mark v
    clock++
    v.pre = clock

    for each edge v->w
        if w is unmarked
            w.parent = v
            clock = dfs(w, clock)
    
    clock++
    v.post = clock

    return clock
```

通过 点的 pre 和 post 属性，可以 判断 一个点的运行期 是否包含 另一个点。

在 dfs_all 遍历全部后， v.pre 的顺序 就是 前缀顺序， v.post 的顺序就是 后缀顺序。

有多种前缀顺序，后缀顺序， 因为 不同的 邻居节点访问顺序

在后续中， v.pre 是 v的starting time ( when v start)，  v.post 是v的 finishing time ( when v finish)。  start 和 finish 的 interval 是 v的 active interval


---

classify vertices and edges

在 dfs_all 执行过程中，图的 每个 v 都是 下列状态中的一种
- new， 如果 dfs(v) 还没有被调用， 即 clock < v.pre
- active, 如果 dfs(v) 被调用了 但还没有return， 即 v.pre <= clock < v.post
- finished, dfs(v) 已经 return，即 v.post <= clock

由于 start，finish time 代表了 stack的 push 和pop， 所以 v 是 active的 当且仅当 v 在 stack中。


![63044110fcbc2c7fb1a45216441de400.png](../_resources/63044110fcbc2c7fb1a45216441de400.png)


---

图的 边 有4种状态
以边 u->v 为例
- 如果 dfs(u) 开始时 v是 new，那么 在 dfs(u) 期间 会调用 dfs(v)。 这种情况下， u是 v 的 祖先， u.pre < v.pre < v.post < u.post
  - 如果 dfs(u) 直接调用 dfs(v)，那么 u = v.parent，且 u->v 是 ==tree edge==
  - 否则 u->v 是 ==forward edge==
- 如果 dfs(u) 开始时， v是 active， 那么 v已经在 递归栈中的，这意味着 v.pre < u.pre < u.post < v.post. 图必须包含 v到 u 的 有向路径。 满足这个条件的 边 是 ==back edges==
- 如果 dfu(u) 开始时， v是 finished， 那么 v.post < u.pre。 满足这个条件的 边 是 ==cross edges==
- 最后， 第四种顺序 u.post < v.pre 是不可能的。

![cc6f8d8bab88b69619342a3652adf3f8.png](../_resources/cc6f8d8bab88b69619342a3652adf3f8.png)


下面的推理 讲述了 depth-first forest中的 祖先和 后代

推理6.1，对于有向图的 一个 depth-first traversal， 对于图中的 任意2点 u和v，下面的 条件是 等价的
- u 是 v在 depth-first forest 中的 祖先
- u.pre <= v.pre < v.post <= u.post
- 在dfs(v)刚调用完时，u还是active
- 在dfs(u)调用前，有一个 u到 v 的路径，该路径上所有 点(包括 u 和 v) 都是 new

证明: 。。。。。


## 6.2 detecting cycles

DAG (directed acyclic graph) 是 无 有向环 的 有向图。

DAG 中 任何没有 入度的 点 被称为 source
任何没有 出度的点 称为 sink
孤立的点 既是 source 又是 sink

每个 DAG 有 至少一个 source，一个sink

---

如果 有边u->v 且 u.post < v.post， 说明 图 包含一个 从 v 到 u 的 有向路径。 所以存在一个 ==经过 u->v的 有向环==。 我们可以 `O(V+E)` 判断 有向图 是不是 DAG: 计算 点的 postorder，然后 检查每条边。

或者，我们 保存 每个点的状态， 如果 发现 一条边 连到了 active状态的点，那么 ==说明不是DAG， 立即返回 false==。 这个算法也是 `O(V+E)`

```
is_acyclic(G):
    for all v
        v.status = new
    for all v
        if v.status == new
            if is_acyclic_dfs(v) == false
                return false
    return true

is_acyclic_dfs(v):
    v.status = active
    for each edge v->w
        if w.status = active
            return false
        else if w.status = new
            if is_acyclic_dfs(w) == false
                return false
    v.status = finished
    return true
```



## 6.3 topological sort

有向图的 拓扑顺序， 是 点的 顺序关系， 如果有边 u->v，那么 u和v 的 拓扑关系就是 u - v。

简单来说， 拓扑顺序，就是 把 所有点 按照某种顺序 放在 x 轴上，使得 所有的边 都是 从 左侧 指向 右侧。

如果图 有 有向环，则 不可能有 拓扑顺序。

---

考虑 有向图的 一种 postordering， 我们之前的分析 表明 对于任意 u->v, 如果 u.post < v.post ，那么 图 存在一个 v->u 的 有向路径， 因为存在一个 经过 u->v 的 有向环。
即，==如果 图是 无环的，那么 对于每条边 u->v， u.post > v.post==。 这也意味着 ==每个DAG 都有一个 拓扑顺序==， 且 ==图的 postordering 的 反转 就是 图的 拓扑排序==

![e42ee0e3c75458d407d4eff9038046bc.png](../_resources/e42ee0e3c75458d407d4eff9038046bc.png)

在一些特殊的 数据结构中，我们需要 拓扑顺序。

我们可以通过 数组保存 postorder，然后反转 ( 或者 反序保存 postorder)， 就可以得到 拓扑顺序，所以 `O(V+E)`

---

implicit topological sort

将 拓扑顺序 保存到 指定的数据结构 通常是 过犹不及的。

拓扑排序的 绝大部分应用， 我们的目标 不是 点的 有序list。 而是 我们要 按照 拓扑顺序 在每个点上 执行 一些 计算。 对于这种应用，我们完全不需要 记录下 拓扑顺序

```
// explicit(显式) topo sort
topo_sort(G):
    for all v
        v.status = new
    clock = V
    for all v
        if v.status = new
            clock = topo_dfs(v, clock)
    return S[1..V]

topo_dfs(v, clock):
    v.status = active
    for each edge v->w
        if w.status = new
            clock = topo_dfs(v, clock)
        else if w.status = active
            fail gracefully
    v.status = finished
    S[clock] = v
    clock = clock - 1
    return clock
```

---

如果我们要 以 反topo序 来处理 DAG，那么 只需要 在 递归dfs的 最后 处理 每个 点。
因为 topo序 == reversed postorder

```
post_process(G):
    for all v
        v.status = new
    for all v
        if v is unmarked
            post_process_dfs(v)

post_process_dfs(v):
    v.status = active
    for each edge v->w
        if w.status = new
            post_process_dfs(w)
        else if w.status = active
            fail gracefully
    v.status = finished
    process(v)
```


如果我们已经知道 图是 无环的，那么可以经一步简化， 只需要 mark，不需要 状态

```
post_process_dag(G):
    for all v
        unmark v
    for all v
        if v is unmarked
            post_process_dag_dfs(s)

post_process_dag_dfs(v):
    mark v
    for each edge v->w
        if w is unmarked
            post_process_dag_dfs(w)
    
    process(v)
```

这是 标准的 dfs。

由于 DAG 上 后续遍历 是非常常见的，所以 有时会简写
```
post_process_dag(G):
    for all v in postorder
        process(v)
```

例如，之前的 显式topo sort的代码可以写成
```
topo_sort(G):
    clock = V
    for all v in postorder
        S[clock] = v
        clock--
    return S[1..v]
```


要以 topo序 处理 DAG，我们可以 记录topo序，然后一个 for循环。
或者 对 图进行反转，然后 dfs。 图的反转是指 所有 v->w 的边 变为 w->v
图的反转的 topo 是 图的topo的反转

有向图的 反转 的 时间复杂度是 O(V+E).


## 6.4 memo and dp

topo sort 算法 是一个 宽泛的 dp算法 的模型。

递归的 依赖图 是 每个递归子问题作为一个 点，然后 从 本问题 指向 每个 递归子问题 都有一条边。
依赖图 必须无环， 不然 递归无法终止

使用 memo 进行 递归 等于 在 依赖图 上进行 dfs。
依赖图的 点 是 marked 如果 对应的子问题 已经 解决。
preVisit， postVisit 是 真正 计算逻辑 的 proxy


更进一步，使用 dp 推导 递归  等同于  以反topo序 对 依赖图 的 每个 子问题进行 计算。
反topo序 使得 在 问题依赖的子问题解决后 处理 该问题。
所以， 每个 dp 算法 等价于 该dp的依赖图的 后续遍历。


```
memo(x):
    if value[x] is undefined
        init value[x]

    for all subproblem y of x
        memo(y)
        update value[x] based on value[y]
    finalize value[x]
```

```
dfs(v):
    if v is unmarked
        mark v
        preVisit(x)
        for all edge v->w
            dfs(w)
        postVisit(x)
```

```
dp(G):
    for all subproblem x in postorder
        init value[x]
        for all subproblem y of x
            update value[x] based on value[y]
        finalize value[x]
```


大多数 dp 和 topo sort 有一些微小区别。
1. 大多数dp算法中， 依赖图 是 隐式的， 点，边 并不直接存储在内存中。
2. 大多数dp 的递归 有高度结构化的 依赖图 。。。。


---

dp in DAG

我们可以使用 dfs 来 构建 dp。

比如，longest path问题， 在 有向 加权 图 中 找到 点s 到 点t 的 max total weight 的路径。

在一般的 有向图中， longest path 是 NP问题。
但，如果 图是 无环的，那么 就很简单， 可以在 线性时间 计算出来:
对于 目标点t， 对 任何点v， 使得 LLP(v) 表示 v到 t 的 Longest Path。 如果图是 DAG，那么 满足递归

![b0fa92c3b6bd2ee98a3a2f5a356b4b22.png](../_resources/b0fa92c3b6bd2ee98a3a2f5a356b4b22.png)

`l(v->w)` 表示 v->w 的权重。
空集的max 是 INT_MIN， 如果 v是 sink 且不是 t，那么 LLP(v) = INT_MIN

这个递归的 依赖图 是 图 自身: 子问题LLP(v) 依赖 子问题LLP(w) 当且仅当 v->w 是图的边。

`O(V+E)` 在图上 从s出发 执行 dfs 。
算法需要 memo LLP的值

```
longest_path(v, t):
    if v == t
        return 0
    if v.LLP is undefined
        v.LLP = INT_MIN
        for each edge v->w
            v.LLP = max{v.LLP, l(v->w) + longest_path(w, t)}
    return v.LLP
```

原则上，我们可以 通过 topo sort 转换 带memo的递归算法 为 dp算法
```
longest_path(s, t):
    for each node v in postorder
        if v = t
            v.LLP = 0
        else
            v.LLP = INT_MIN
            for each edge v->w
                v.LLP = max(v.LLP, l(v->w) + w.LLP)
    return s.LLP
```

这2个算法 是完全相同的， 第一个算法的 递归 和 第二个算法的for 都代表了 dfs。

---

求 optimal sequence 的 dp问题 基本都可以转为 在 DAG上求 optimal path。
比如 test segmentation，subset sum，LIS，edit distance 都可以转为 求 DAG 上的 最长 或最短 路径。  DAG是 是 递归的 依赖图。

但是 树形 的 dp问题，如 optimal binary search tree, max independent set， 不能 转为 在DAG上求 optimal path


## 6.5 strong connectivity 强连通分量

让我们回到 有向图的 connectivity 的定义。

在 有向图中，点u 可以 到达另一个 点v，如果 图包含一条 从 u 到v 的有向路。

`reach(u)` 代表 所有 u可以到达的点 的集合。

点u 和 v 是 ==strongly connected==，如果 u可以到达v， v 也可以到达 u

有向图 是 strongly connected， 当且仅当 所有的点 两两都是 强连通的。

strongly connected components, 简称 strong components

图 的 strong component 是一个 最大 strongly connected subgraph。

有向图是 strongly connected  当且仅当 图 只有一个 强连通分量

==图是 DAG 当且仅当 图的 每个强连通分量 只包含一个点。==

---

strong component graph scc(G) 是 从 图G 生成的 有向图， 它将 每个 强连通分量 合并为 一个 点，且删除 多余的边。

strong component graph 有时也称为 meta-graph 或 condensation

并不难证明 scc(G) 总是一个 DAG。

可以`O(V+E)` 计算出 点 v 所在的 强连通分量
1. 通过 whatever-first search 计算出 reach(v)
2. 然后 计算 `reach^-1(v) = {u | v 属于 reach(u)}`， 通过 搜索 图的 反转图。
3. v的强连通分量就是 `reach(v)` 和 `reach^-1(v)` 的 交集。


在上面的算法 外面 封装一层，就可以获得 图的 所有 强连通分量。
`O(VE)`， 最多有 V个 强连通分量， 每个最多需要 O(E) 来处理。

我们可以 `O(V+E)` 来判断 是否 每个强连通分量 都是 只有一个 点。


## 6.6 strong components in linear time

有多种算法 可以在 `O(V+E)` 计算出 strong component。 这些算法 都基于下面的 观察

Lemma 6.2，对任意有向图进行 depth-first traversal， 图的每个 强连通分量 包含且只包含 1个 节点，该节点在 连通分量中 没有 parent。( 这个节点 的parent 可能在 其他强连通分量中，或者 该节点 没有 parent)
。。应该是 强连通分量 中 每个 点 肯定有 一个parent， 但是 dfs时， 进入这个连通分量时，肯定从 外部进入的， 所以 这个 parent 不在 强连通分量中。  就是 parent 是 dfs时 的 parent。
。。但好像也不是，看不懂了。。 看了 proof， parent 确实是 dfs 的 parent。
```
fix a depth-first traversal of any directed graph G. each strong component C of G contains exactly one node that does not have a parent in C. (either this node has a parent in another strong component, or it has no parent)
```

证明: 。。。。。。。


上面的lemma 暗示: 有向图的 每个 strong component 定义了 图的 depth-first forest 的 中的 一棵 连通的 子树。  特别地，对于 任意 strong component C，C中 最早被访问的 点 是 C中所有点的 ==lowest common ancestor==. 我们称这个点 是 C的 root


---

设 C为 图G中的 任意 strong component，且 C是 scc(G) 的一个 sink， 我们称 C 为 sink component。
等价地， C是 sink component，如果 C中 所有点的 可达点 组成的集合 正好是 C。

我们可以 找到 图中 所有 strong component，通过 重复 寻找 sink component 中的 点 v，找到v 可达的所有点，然后 移除 sink component， 直到 没有 点。 
这不是一个算法， 因为 它没有说清 如何 找到 sink component中的 点

```
strong_component(G):
    count = 0
    while G is not empty
        C = 空集
        count = count + 1
        v = sink component 的 任意一个 点
        for all w in reach(v)
            w.label = count
            add w to C
        从G中移除C 和 它的 入边。
```

---

### Kosaraju

Kosaraju and Sharir's Algorithm

找到 sink component 中的 点 似乎有点难。

但是，找到 source component 中的 点 很容易 ， 图中的 强连通分量 相当于 scc(G) 中的 source

Lemma 6.3， 图的任意后续遍历 的 最后一个点 处于 图的 source component 中。

证明: 。。。。。。v是后缀遍历的 最后一个点，说明 DFS(v) 是 dfs_all 的最后一次的 调用。  所以 v 是 dfs的 某个tree的 root。 x.post > v.pre 的x都是 v的 后代。 所以 v是 强连通分量C的 root 。。。。反证有一部分是 反证。

容易证明: rev(scc(G)) == scc(rev(G)). 所以 rev(G) 的 后续遍历的 最后一个 点 是 处于 原图G的 sink component 的。。 。。?

结合上面的一切，我们可以发现 下面的算法， 它可以 O(V+E) 计算 和 标记 任意有向图的 强连通分量。

Kosaraju-Sharir 算法 由 2部分组成。 
第一部分 对 rev(G) 执行 dfs， 后序遍历 中 将 所有 点 压栈。
第二部分 对 原图 执行 whatever-first traversal，使用 栈中的 顺序。
这个算法 使用 点所在的 强连通分量的 root 来 标记每个点

`O(V+E)` 计算出 scc(G) ( 强连通图， strong component graph)


![8bcd9023cac4d0a39e0f0bde0f0f52fe.png](../_resources/8bcd9023cac4d0a39e0f0bde0f0f52fe.png)


![3a262858cb42ada5da3e33c31e24dfd4.png](../_resources/3a262858cb42ada5da3e33c31e24dfd4.png)


### Tarjan algorithm

242

更早，更微秒的 线性时间算法。

Tarjan算法 识别 图的 source component，"delete" it, 然后 递归 搜索 剩余的 strong component。 但是，整个计算 只需需要 一次 dfs。

对于 有向图的 任意一种 depth-first traversal。 对于每个 点v，`low(v)` 是 从v 可以通过 tree edge组成的 path 后跟着 最多一个 non-tree edge 可达的 所有点 的最小的 starting time 。 
Trivially(平凡地), `low(v) <= v.pre`，因为 v 可以 经过 0个tree edge 跟着0个non-tree edge 到达它自身 。
Trajan 发现 sink component 可以 用这个 `low` 方式 识别出来。

Lemma 6.4  点v 是 图的 某个 sink component 的root  当且仅当 `low(v)=v.pre` 且 对于v的所有 后代w， `low(w) < w.pre`

证明: 。。。。

通过 dfs 计算 每个点的 `low(v)` 很容易。

Lemma 6.4 表明，在 执行 findLow 之后， 我们可以 通过一个 全局 whatever-first search `O(V+E)` 识别 每个 sink component。
然后用 `O(V+E)` 额外时间 mark 和 delete 这些 sink component。 然后 递归。
最差情况下，整个算法 ==需要 V次迭代==， 每次 只移除一个 点， `O(VE)`


```
findLow(G):
    clock = 0
    for all v
        unmark v
        for all v    // 真的是这个 缩进。文字描述也是这样的。
            if v is unmarked
                findLowDfs(v)

findLowDfs(v):
    mark v
    clock = clock + 1
    v.pre = clock
    v.low = v.pre

    for each edge v->w
        if w is unmarked
            findLowDfs(w)
            v.low = min(v.low, w.low)
        else
            v.low = min(v.low, w.pre)
```


为了加速这个算法，Tarjan算法 维护了 一个 点的 辅助stack。
当我们开始一个 新的 点v时， push 该点 到 stack。 当我们结束 点v时， 我们比较 v.low 和 v.pre。 当我们 第一次发现 v.low = v.pre 时， 我们知道:
- 点v 是 sink component 的root
- sink component 的 所有点 都 连续地出现在 辅助stack的 top
- 辅助stack中，该 sink component 的 最深的点是 v。

这样，我们可以 识别 sink component 的所有点， 通过 从 辅助栈 pop出来，直到 把 v pop出来。

我们 可以 删除 sink component中的点，然后 对 剩余的 图 继续计算 强连通分量， 不过这很浪费， 因为我们可以 在 访问v之前 重复 逐字地 所有的计算。 (because we would repeat verbatim all computation done before visiting v)

我们 标识 sink component 中的每个 点， 将v 作为 它所在的 强连通分量的 root。 然后在 后续的 dfs 中 忽略 被标识的 点。
这个修改 导致 `low(v)` 的定义变成:  v所在的 可以通过 tree edge 后跟 最多一个 non-tree edge 组成的 path 访问的 强连通分量 中的 所有点的 最小 starting time。
。smallest starting time among all vertices in the same strong component as v that v can reach by a path of tree edges followed by at most one non-tree edge.....
正确性 很简单， 因为 忽视 已标记的 点， 等同于 删除它们

下面是 完整的 Tarjan 算法， 对 之前的 findLow 做了必要的修改(红色部分)。
算法的 运行时间 分为2部分， 每个点 被push到 S一次，从 S pop一次，所以 辅助栈 消耗的时间是 `O(V)`。 如果我们忽略 辅助栈的 代码， 其余的代码 是一个 标准dfs。 我们可以 `O(V+E)` 运行。

![e8d73bad9bcaac95b2ced70cc4722674.png](../_resources/e8d73bad9bcaac95b2ced70cc4722674.png)



# ch7 minimum spanning trss

假设我们有一个 连通的，无向的，有权重的 图。 即 `G=(V,E)` 且有一个 函数w 将 E映射为 实数 作为 权，权可以是 正数，负数，0.

本章描述 寻找G 的 最小生成树的 方法。 即，spanning Tree 有着最小的 边权的连加。


## 7.1 distinct edge weights

257

一个图 可以有 多个 min spanning tree。 比如，如果 边的权 都是1，那么图的 所有 spanning tree 都是 min 的。

Lemma 7.1  如果连通图 中所有边的 权 都是 distinct的，那么 只有一个 min spanning tree

证明 。。。。。。 反证，如果 权都是 distinct，有2个 不同的 min span tree，那么 取 第二棵tree - 第一棵tree 的边 中 最小的边， 然后这个 边 加入到 第一棵树， 会形成一个环， 由于 边的权都是 distinct的，。。。。。证明很麻烦。。
。。我想的: 1. 2个min span tree 必然可以形成 环，那么 这个环上 都是 distinct的，只有一种删除边的方法。  和 2个min span tree 冲突
。2. T - T' 中最小的边e， T' - T 中最小的边e'    如果 e 小于 e'， 那么把 e添加到 T' 中 。。。这不对。
。2. T中最小边e， T'最小边e'， 由于 。。不对。。


如果我们已经有一个 可以在 distinct 权 的图上使用的 算法，那么也可以 将这个算法 运行到 有 相同权的 图上，只要 我们增加一些 一致性方法 来打破平衡。 一致性方法 比如 下面的 方法，给予 2条边 计算 哪条更小。
```
shortest_edge(i,j, k,l)
    if w(i,j) < w(k,l)      return (i,j)
    if w(i,j) > w(k,l)      return (k,l)
    if min(i,j) < min(k,l)  return (i,j)
    if min(i,j) > min(k,l)  return (k,l)
    if max(i,j) < max(k,l)  return (i,j)
    
    return (k,l)    // max(i,j)>max(k,l)
```

通过上面的方法，我们可以确保 所有的 图 的边 的权 都是 distinct的。
根据lemma 7.1 + 上面的方法, 可以确保，所有图 都只有一个 唯一的 min span tree
所以接下来，只考虑 min span tree


## 7.2 the only min span tree algorithm

有许多算法来计算 min span tree。 但是 它们的原理都一样。 类似 图的遍历，但是 使用 whatever-first search 的 不同变种。

一般的 min span tree 算法 维护了 图G 的 一个 无环子图F， 这个F 被称为 intermediate spanning forest。 在任何时候， 这个无环子图F 都满足: F是 原图G 的 min span tree 的 子图。

初始时，F 包含 V个 单节点的 树。算法 会 通过 增加边来 连接 F中的树。 算法停止时， F就是 一个 spanning tree， 且 是 min span tree。
当然，我们必须 仔细计算 添加 哪条边。

在算法的任何时候，非 intermediate spanning forest F 中的边 有3种状态
- useless， 不是F中的边，但是 边的 2个端点 是 在 F的 某个连通分量中。
- safe， 边的2端 分为属于F的 2个连通分量 中  权最小的 边。
- 其他边 是 undecided。


Lemma 7.2 Prim算法: G的 min span tree 包含 所有 safe edge

证明: 。。。。。证明 更强的论点: 对于图 的点 的 任何 subset S， 图的min span tree 必然包含 只有一个 点在 S中的 边中 权最小的边。


Lemma 7.3  min span tree 不包含 useless edge

证明: 加入 任何的 useless edge，都会导致 环。


我们的 min span tree 的算法 就是 不停的 将 safe 边 加入到 森林F 中。 如果 F 不是 连通的，那么必然 存在 一条 safe 边， 因为 图G 是 连通的。

每当往 F 中添加 新边时， 一些 undecided 的边 变成 safe，其他 undecided 边 变成 useless。
算法 必须说清 每次迭代时 增加哪个safe edge，并且 如何找到 这些 edge。


## 7.3 Boruvka's algorithm

最早，最简单的 min span tree 算法 在 1926年由 Boruvka 发现。1938 Choquet，1951 Lukaszewicz，1961 Sollin 分别重新发现。

这些算法 总结为一句: `add all safe edge and recurse`
下面的算法 使用率 p204 第5章中的 countAndLabel 算法。 。。 本文搜 `count_and_label` 。。但是 逻辑并不匹配啊。。代码逻辑不匹配，但是 大方向对的。
```
Boruvka(V, E):
    F = (V, 空集)
    count = countAndLabel(F)
    while count > 1
        addAllSafeEdges(E, F, count)
        count = countAndLabel(F)
    return F
```

。。addAllSafeEdges  有点复杂。。不抄了。。p262
..书里也说了 Boruvka 太复杂了。

![2b6d01dda342c02f1b7ed67183e8d8bb.png](../_resources/2b6d01dda342c02f1b7ed67183e8d8bb.png)

`O(ElogV)`

。不过 它也有优点。
- 大部分情况下，比 `O(ElogV)`快。
- 它的一个变种 对于 某些图 可以 `O(E)`。 包括: 图可以画出来 且 边不交叉。
- 可以并行化，在每次迭代时， F 的每个连通分量 可以 由 一个线程 来处理。
- 目前有些 min span tree 算法 比 本书中介绍的 要快， 这些算法 是 Boruvka 的 变种。


## 7.4 Jarnik(Prim) algorithm

Prim算法中， intermediate forest F 只有一个 nontrivial component T。 其他的分量都是 单个点。
初始时，T 只有一个点， 重复下面的 操作，直到 T 延伸到 整个树
`repeatedly add T's safe edge to T`

。。T初始时是任意一个端点， 每次 把 2端分别属于 T 和 非T 的 所有边中 权最小的 边 添加到 T 中。

![dc33a19c4899b0d25827f4f83c53a325.png](../_resources/dc33a19c4899b0d25827f4f83c53a325.png)

为了实现 Prim算法，我们需要将 T 的所有临边 保存到 pri_que中。 从 pri_que中获得边时，先判断 边的 2端 是否都是T， 如果不是，那么久 加入到 T，并且 增加新的 临边。

Prim是 best-first search 的一个 变种。

`O(ElogE) = O(ElogV)`

---

improve prim algorithm

通过 使用更复杂的 pri_que ( Fibonacci heap ) 来 优化prim算法。

Fibonacci heap 和 binary heap一样，支持 标准 pri_que操作: insert，extractMin，decreaseKey。
但是 binary heap的 每个操作 需要 `O(logn)`， 而 Fibonacci heap 的 insert，decreaseKey 是 常数时间， extractMin 依然是 O(logn)

。。Fibonacci heap 很复杂。。 而且 log21亿 = 31.

为了应用 Fibonacci heap 我们将 保存 图的点 而不是边 到 pri_que。 点v 的 优先级是  v 和 T 之间的 边的 min weight， 如果没有这样的边，就是 INT_MAX。
我们可以在 开始时 插入所有的点到 pri_que， 当我们往T中 新增边时，降低 一些邻居 点的 优先级。

我们将算法分为2部分， 初始化pri_que， 主算法。
对于每个 点v，我们维护 优先级`priority(v)` 和 一条边`edge(v)` ，使得 `w(edge(v)) == priority(v)`

```
Jarnik(V, E, s):
    JarnikInit(V, E, s)
    JarnikLoop(V, E, s)

JarnikInit(V, E, s):
    for each v in V-{s}
        if vs 属于 E     // 说明 v和s 有 直接边 相连。
            edge(v) = vs
            priority(v) = w(vs)
        else
            edge(v) = NULL
            priority(v) = INT_MAX
        insert(v)

JarnikLoop(V, E, s):
    T = ({s}, 空集)
    for i = 1 to |V| - 1
        v = extractMin      // 只有init的时候 往pri_que中insert，而且保存的是 点，所以不需要判断 边的2端 是否已经都是 T中。
        add v and edge(v) to T
        for each neighbour u of v
            if u 不属于 T and priority(u) > w(uv)
                edge(u) = uv
                decreaseKey(u, w(uv))
```

insert 和 extractMin 调用了 `O(V)` ， decreaseKey 调用了 `O(E)`。
所以如果使用 Fibonacci heap，`O(E + VlogV)`， 比 Boruvka 快，除非 `E = O(V)`


但是 这个优化后的算法 实际使用中 很难 比 binary heap的 快，除非 图很大 且 稠密。
因为 Fibonacci heap 很复杂， 时间复杂度的 常数 很大。


## 7.5 Kruskal algorithm

`scan all edges by increasing weight; if an edge is safe, add it to F`

![b8c506642ad9bc62fb9528dcf9f17a13.png](../_resources/b8c506642ad9bc62fb9528dcf9f17a13.png)

最简单的方法就是 扫描所有边，按权重升序。`O(ElogE)`

由于 按权重 从低到高，所以 如果 边的 2端 属于 不同 分量，那么 边是 safe 的。

和 Boruvka 算法一样，F中的每个 点 需要知道 它属于 F的哪个 连通分量。
和 Boruvka 不同的是， 增加边到 F 后，不需要 重新计算 label。

Kruskal 用到了 uf / disjoint set。

```
kruskal(V, E):
    sort E by increasing weight
    F = (V, 空集)
    for each v in V
        makeSet(v)
    for i = 1 to |E|
        uv = ith lightest edge in E
        if find(u) != find(v)
            union(u, v)
            add uv to F
    return F
```

`O(ElogV)`




# ch8 shortest path

加权有向图 `G = (V, E, w)` 和 2个点， 我们希望找出 这2个点之间的 最短路径。

现实中，就是 从城市A 到 城市B，怎么样最快到达。


## 8.1 shortest path tree

所有的 计算 2点间最短路径的算法 都是 基于 single source shortest path (SSSP) 问题: 找到从 s出发的 到达 其他所有点的 最短路径。 这类问题 通常通过 寻找 以s 为root的 shortest path tree 来解决。

不难看出，如果 shortest path 是 unique，那么 它们组成 一棵树， 因为 shortest path 的 任何 subpath 也是 最短路径。如果 到某些点 有多条 最短路，那么我们可以选一条，使得 path的union 是树。 如果2个最短路径有交叉，那么可以 断开 交叉点 之前的 某条边。

最短路径树 和 min span tree 都是 optimal spanning tree， 但是 它们非常不同。
最短路径树 是 rooted，directed。
min span tree 是 unrooted，undirected。
最短路径tree 通常用于 有向图
min span tree 用于 无向图。
如果 权是 distinct，那么 min span tree 只有一棵。 但是 最短路径tree 是 每个点 一棵。


## 8.2 negative edges 负权边

大多数的 最短路径问题， 边的 权 是 距离，或长度 或时间， 所以 天然是 非负的。

一部分问题是 带负数的，比如 cost。

负权边 对于 最短路径始终是一个问题，因为 如果存在 负权环，那么 就没有 最短路径。

本章只考虑 有向图， 稍微修改 就可以让 有向图的算法 应用于 无向图， 只要 没有负权边。

无向图中 处理 负权边 更难， 我们不能 直接 将 无向边 用 2个有向边替换，这会导致 一个负权环。

包含负权边的 无向最短路径 的 subpath 不一定是 shortest path。

从某个点出发的 无向最短路径 的 集合 不一定能构成 一棵树。

对于 含 负权边的 无向图的 讨论 超过了本书的范围。 稍微说一下， 负权边的 无向图的 单个最短路径 的 时间复杂度是 `O(VE + V*V*logV)`， 通过 reduction to maximum weighted matching


## 8.3 the only SSSP algorithm

single source shortest path

与 图遍历 和 min span tree 一样， 许多 SSSP算法 都是 一个 通用算法的 变种。

图中的 每个点v 保存2个值，描述了 从s 到v 的 暂时的 最短路径
- `dist(v)`, s->v 的 暂时的最短路径的 长度， 如果不可达就是INT_MAX
- `pred(v)`, 上面这个 暂时的最短路径中 v的 前驱点， 如果没有 就 NULL

前驱 指针 定义了一个 以s 为root 的 暂时的 最短路径树。 这些指针 就和 图遍历中的 parent指针一样。
在算法的开始时，初始化 距离 和 前驱:
```
initSSSP(s):
    dist(s) = 0
    pred(s) = NULL
    for all v != s
        dist(v) = INT_MAX
        pred(v) = NULL
```

在算法的执行过程中， 边 u->v 是 tense(紧张的，不能松弛的) 如果 `dist(u) + w(u->v) < dist(v)`， 如果 u->v 是tense，那么 暂时的 最短路径 s->v 是不正确的， 因为 s->u->v 更短。 我们可以通过 relax 边 来 纠正。
```
relax(u->v):
    dist(v) = dist(u) + w(u->v)
    pred(v) = u
```

Ford 的 算法的描述: `repeatedly relax tense edges, until there are no more tense edges`

```
Ford_SSSP(s):
    initSSSP(s)
    while there is at least one tense edge
        relax any tense edge
```

FordSSSP 算法停止后， `pred` 就定义了 一个 最短路径树， 且 `dist(v)` 就是 s 到v 的 最短路径距离。
如果 s 无法到达 v ，那么 `dist(v) = INT_MAX`
如果 从s 可以到达 负权环，那么 算法 无法停止。

Ford算法的 正确性 由下面证明
1. 算法执行期间，对于每个 点v， dist(v) 要么是 INT_MAX，要么是 s到v 的距离。
2. 如果图没有 负权环，那么 `dist(v)` 要么是 INT_MAX，要么是 s->v 的距离。
3. 如果图中的 边 都不是 tense，那么 对于每个 点v， dist(v) 就是 `s-> pred(pred(pred(v))) -> pred(pred(v)) -> pred(v) -> v` 的长度。 如果v违反了这个限制，但是 pred(v) 没有违反，那么 pred(v)->v 是 tense
4. 如果图中的边 都不是 tense，那么 `s->..->pred(v)->v` 是 s到v 的 最短路径。 如果v违背了这个显示，但是 前驱u 没有违反，那么 u->v 是tense。 这也意味着 如果图有 负权环，那么 一些边 始终都是 tense的， 算法无法终止

到目前为止，还没有讲， 如何找到 tense 边， 有多条的话 选择哪条进行 relax。 就像 whatever-first search，Ford 也有多种不同的方法。

本章后续 有 4种 Ford算法的 实例。  对于不同的 图， 有不同的 最优实例。

## 8.4 unweighted graphs: breadth-first search

最短路径问题 最简单的case 是 所有的边的 权 都是1， 这样 path的长度 就是 边的数量。

这种情况 可以通过 bfs 解决。

。。确实，bfs 就是 最快到达 该点的 步骤。

```
bfs(s):
    initSSSP(s)
    push(s)
    while queue is not empty
        u = pull()
        for all edge u->v
            if dist(v) > dist(u) + 1   // u->v is tense
                dist(v) = dist(u) + 1  // relax
                pred(v) = u
                push(v)
```

。。这里有一段分析， 挺长的。  p279 - 282
。。使用 特殊符号，来 区分 phase。

`O(V+E)`


## 8.5 directed acyclic graphs: depth-first search

在有向无环图中， 最短路径 很简单， 即时 边带有权，甚至 一些边 是 负权边

图G 是 带权有向图， s是 起始点。 对于 任意 点v，`dist(v)` 是 G中 s 到v 的 最短路径的 长度。

![a82dd7a1b7ebe5bf895bbe6ba006de3f.png](../_resources/a82dd7a1b7ebe5bf895bbe6ba006de3f.png)

上面适用于 所有 有向图，但是 对于 有向无环图，它 只能是 递归的。
如果 图G 有环， 递归会 死循环。
但是 由于 G是 DAG，所以 每次 递归 都是 访问 topo序 中 更早的点。

这个递归的 依赖图 就是 输入的图本身: 子问题 `dist(v)` 依赖于 `dist(u)` 当且仅当 u->v 是 图中的 边。 因此，我们可以 `O(V+E)` 执行 dfs 后序 遍历 G的反转， 即， G的 topo 顺序

```
DAG_SSSP(s):
    for all v in topo order
        if v == s
            dist(v) = 0
        else 
            dist(v) = INT_MAX
            for all edge u->v
                if dist(v) > dist(u) + w(u->v)
                    dist(v) = dist(u) + w(u->v)
```

最终的 dp 算法是 Ford 的一个实例。


```
DAG_SSSP(s):
    init_SSSP(s)
    for all v in topo order
        for all edge u->v
            if u->v is tense
                relax(u->v)
```

DAG_SSSP 不同于 bfs 和其他 Ford的relax 策略。 其他 最短路径算法 考虑 点，它们尝试 松弛 它的 出边。 而 DAG_SSSP 尝试 松弛 每个 入边。

我们可以修改 DAG_SSSP 使得 它松弛 出边。`O(V+E)`
```
Push_DAG_SSSP(s):
    init_SSSP(s)
    for all u in topo order
        for all outgoing edge u->v
            if u->v is tense
                relax(u->v)
```


## 8.6 best-first: Dijkstra Algorithm

使用 pri_que 代替 bfs中的 queue。 点v 的优先级是 它的 暂时的距离 `dist(v)`

在算法的执行期间，边u->v 是tense 的 当且仅当 点u 在 pri_que中 或 是最近从pri_que extract的。
Dijkstra 是 Ford 的 实例。  如果 G中无 负环，这个算法可以正确 计算出 最短路径。

```
Dijkstra(s):
    initSSSP(s)
    insert(s, 0)
    while pri_que is not empty
        u = extractMin()
        for all edge u->v
            if u->v is tense
                relax(u->v)
                if v is in pri_que
                    decreaseKey(v, dist(v))
                else
                    insert(v, dist(v))
```

---

No negative edge

Dijkstra算法 在 图没有 负权边的情况下， 工作良好。

![4166e3bed916636befe3a1792f5a5f0a.png](../_resources/4166e3bed916636befe3a1792f5a5f0a.png)


Lemma 8.3 如果G没有负权边，那么对于所有 i < j, di <= dj    。。。i，j 是出pri_que的顺序。

Lemma 8.4 如果G没有负权边，G的每个点 只从 pri_que extract 最多一次。

Theorem 8.5 如果G没有负权边，当Dijkstra算法结束时，dist(v) 是 s到v 的 最短路径的长度， v可以是任意点

`O(ElogV)`

如果我们确信 如没有 负权边，那么可以 简化 Dijkstra算法， 通过在 init结算 将所有 点 insert到 pri_que， 在 主循环中 调用 decreaseKey

```
no_negative_dijkstra(s):
    init_SSSP(s)
    for all v
        insert(v, dist(v))
    while pri_que is not empty
        u = extractMin()
        for all edge u->v
            if u->v is tense
                relax(u->v)
                decreaseKey(v, dist(v))
```

。。但是 pri_que 没有 decreaseKey 方法啊。


---

negative edge

上面的 no_negative_dijkstra 对 有负权边的 图 无法正确计算出 最短路径。
甚至，即使 边的权重都是 正数， no_negative_dijkstra 也不会比 Dijkstra 快。

当 图有负权边时，算法不再正确无误。 同一个点 可能 extract 多次。 同一条边 可能 relax多次。 升序可能无法计算出 距离。

对于 没有 负权环的 图， Dijkstra 的 时间复杂度 是 指数级别的。 `O(2^V)`

但是 实际应用中， Dijkstra 对于 有负权边的 图 也足够快了。



## 8.7 relax all edge: Bellman-Ford

算法思想: `relax all tense edges, then recurse`

```
bellman_ford(s)
    init_SSSP(s)
    while there is at least one tense edge
        for every edge u->v
            if u->v is tense
                relax(u->v)
```
。。似乎时间复杂度很高

Lemma 8.6 对于每个点v 和 非负整数 i，在 Bellman-Ford 的主循环 i次迭代后，我们有 dist(v) <= dist~<=i~(v)
。。意思是，每次迭代，只会使得 dist 变小?

证明: 。。。。。。


如果图 没有负权环，从s到 任意其他点的 最短路径是 最多 V-1条边的 简单路径。 所以 最多 V-1 次迭代。       。。。 无环的情况下 连接 V个点 只需要 V-1条边。
换句话说，如果 V-1 次迭代后，还有 tense 边，那么说明 ==图有 负权环==。
所以我们可以重写算法，使之更具体
```
bellman_ford(s)
    initSSSP(s)
    repeat V-1 times
        for every edge u->v
            if u->v is tense
                relax(u->v)
    
    for every edge u->v
        if u->v is tense
            return "negative cycle"
```

`O(VE)`
Bellman-Ford 总是足够高效，即使有负权边，即使有负权环。

如果 边的权 都是 非负数的， 那么 Dijkstra更快。  实践中，即使有负权边，Dijkstra 也比 BellmanFord 快。


---

Moore's improvement

Moore的算法 和 Bellman的算法 很类似。
Moore 也是 `O(VE)`，但是 比 Bellman 快一点，因为 它避免检查 哪些 明显 not tense 的 边。

Moore的 带权最短路径算法 是 在 bfs 版本 (本章开头那个 所有边的权为1) 的 基础上 进行了 2个调整。
1. 最内层循环中 使用 `+w(u->v)` 代替 `+1`
2. 在 insert到 queue之前 检查 它是否已经在 queue中

在之前 bfs 的分析中，引入了 token 来 将 算法 分割为 phase。

```
Moore(s):
    init_SSSP(s)
    push(s)
    push(#)  // 作为token，来分割为 phase
    while queue contains at least one vertex
        u = pull()
        if u == #
            push(#)
        else
            for all edges u->v
                if u->v is tense
                    relax(u->v)
                    if v is not already in queue
                        push(v)
```

由于 queue中 不包含重复元素，所以 每phase 迭代 最多检查 V个点。

Moore 可以看做 BellmanFord 使用 queue来维护 tense edge，而不是 暴力测试每个 edge。

Lemma 8.7 对于每个点v 和 非负整数i，在 Moore的 i phase后，我们有 dist(v) <= dist~<=i~(v)

如果 图 没有 负权环，那么 Moore 最多 V-1 phase。 每个 phase 中，每个 点最多检查一次， 所以 每条边 最多松弛一次。
`O(VE)`

实际使用中,Moore通常比 Bellman 快， 因为 它只需要 scan 上个phase中发生变化的 边。

如果有 负权环， Moore算法 无法结束。 当然，和 Bellman 一样，经过修改后， Moore 可以报告 存在负权环。
最简单的修改是: 记录token次数，如果 phase 超过 V-1 次，那么说明 存在 负权环。

---

### DP

Bellman是用过DP来实现 bellman ford 算法的。

递归式

![46fefa973b72f332e055079918867d3e.png](../_resources/46fefa973b72f332e055079918867d3e.png)

不幸的是，如果 图不是 DAG，这个递归式 ==无法起效==。  假设有环 u->v->w->u， 递归 会死循环。

为了能正确递归，我们需要增加一个 额外的参数，它每次 递归时 都会 降低， 当到达0时，说明 无法计算出结果。  Bellman 选择 边的 最大数量 作为这个 额外的参数。

![4bbeb44edd93b57a09b7c6428558ab53.png](../_resources/4bbeb44edd93b57a09b7c6428558ab53.png)

假设==没有 负权环==，所以 我们的目标是 计算 每个点的  dist~<=V-1~(v)。
这里是dp的递归推导: `dist[i,v]` 保存了 dist~<=i-1~(v) 的值。
`O(VE)`

。。有负权环的话，就是 V-1 次迭代后，遍历边，判断 还有没有 tense 边。

```
bellman_ford_dp(s)
    dist[0, s] = 0
    for every v != s
        dist[0, v] = INT_MAX
    for i = 1 to V-1
        for every v
            dist[i, v] = dist[i-1, v]
            for every edge u->v
                if dist[i, v] > dist[i-1, u] + w(u->v)
                    dist[i, v] = dist[i-1, u] + w(u->v)
```

。。p296
```
bellman_ford_dp_2(s)
    dist[0, s] = 0
    for every v != s
        dist[0, v] = INT_MAX
    for i = 1 to V-1
        for every v
            dist[i, v] = dist[i-1, v]
        for every edge u->v                     // 确实是这个缩进了，其他都一样。
            if dist[i, v] > dist[i-1, u] + w(u->v)
                dist[i, v] = dist[i-1, u] + w(u->v)
```


```
bellman_ford_dp_3(s)
    dist[0, s] = 0
    for every v != s
        dist[0, v] = INT_MAX
    for i = 1 to V-1
        for every v
            dist[i, v] = dist[i-1, v]
        for every edge u->v
            if dist[i, v] > dist[i, u] + w(u->v)    // [i,u]
                dist[i, v] = dist[i, u] + w(u->v)  // [i,u] not [i-1,u]
```

```
bellman_ford_final(s)
    dist[s] = 0
    for every v != s
        dist[v] = INT_MAX
    for i = 1 to V-1
        for every edge u->v
            if dist[v] > dist[u] + w(u->v)
                dist[v] = dist[u] + w(u->v)
```

。。但是 我记得的 bellman ford 是 3重循环啊。 `k i j { vvi[i][j] vs. vvi[i][k]+vvi[k][j]}` ... 那个是 Floyd 算法。。。下一章的。。




# ch9 all-pairs shortest path

9.1 introduction

上一章中，讨论了 单源最短路径， 通过 构造 以s为root的 shortest path tree。
shortest path tree 需要在 图的每个点上 保存 2样信息
- `dist(v)`, s->v 的最短路径的长度
- `pred(v)`，s->v 的最短路径中 v的前驱 (倒数第二个点)。

本章，讨论 所有点对间最短路径问题。
对于任意的 2个点 u,v，我们需要计算下面的信息:
- `dist(u, v)`, u->v 的最短路径的长度
- `pred(u, v)`， u->v 的最短路径的 倒数第二个 点

一些 上章中已经看到的 边界条件
- 如果 u 到 v 没有路径，那么 `dist(u, v) = INT_MAX`, `pred(u, v) = NULL`
- 如果 u 到 v 有 负权环，那么 u->v 的 路径是 负数长度，不可能存在，此时 `dist(u,v) = INT_MIN`, `pred(u, v) = NULL`
- 如果 u 不在 负权环上，那么 u到它自身的 最短路径 不包含任何边，此时 `dist(u,u) = 0`, `pred(u,u) = NULL`


点对间最短路径的 输出 是一个 V*V 的 二维数组。


## 9.2 Lots of single sources

最直观的 点对间最短路径 是 将每个点作为source，求 V次 单源最短路径。

```
obvious_APSP(V,E,w):
    for every vertex s
        dist[S, *] = SSSP(V,E,w,s)
```

时间复杂度 依赖于 使用的 SSSP算法。
选择哪个 SSSP，取决于 图的结构 和 边的权
- 如果边是 无权的，bfs即可，`O(VE) = O(V^3)`
- 如果 图 无环，那么按照 topo序 扫描 点， `O(VE) = O(V^3)`
- 如果所有的 边权 是 非负数的，那么 Dijkstra， `O(VElogV) = O(V^3 * logV)`
- 最一般的情况，使用 Bellman-Ford， `O(V^2 * E) = O(V^4)`



## 9.3 ==re-weight==

负权边 是一个很大的问题， 我们可以避免它们吗?

一种简单的idea 是 对 每条边 增加 相同的 权重，使得 所有的边的权 都是 非负数的。
然后可以使用 Dijkstra，而不是 Bellman-Ford。
但不幸的是，这不行。 因为 短的路 增加的 权重更少。

有一种更微秒的 方法来增加权重，
假设 每个点 v 有对应的 price `π(v)`， 它可能是正，可能负，可能0. 我们可以定义一个 新的 weight 函数 w': `w'(u->v) = π(u) + w(u->v) - π(v)`

想象，当我们离开 点 u时， 需要 交 `π(u)` 的税， 进入 点v时，获得 `π(v)`

不难发现，使用w' 和 w 的 最短路径 是相同的。 `w'(u->v) = π(u) + w(u->v) - π(v)`


## 9.4 Johnson's algorithm

Johnson的 所有点对间 最短路径 算法 为 每个点 计算一个 cost π(v)， 这样，每条边 的 新的 weight 都是 非负数的。 然后 使用 Dijkstra算法

首先，假设 图中有一个点s 可以到达 所有其他点。 Johnson算法 使用 Bellman-Ford 计算 从s 到其他点的 最短路径， 然后 使用 price函数 `π(v) = dist(s,v)` re-weight: `w'(u->v) = dist(s,u) + w(u->v) - dist(s,v)`

新的 weight 是 非负数的，因为 Bellman-Ford 停止了。 如果 bellman ford 发现 负权环，那么 Johnson算法 推出，因为 没有 最短路径。

再次声明: 边u->v 是 tense 的，如果 `dist(s,u) + w(u->v) < dist(s,v)`

如果没有 点s 可以到达其他所有点，那么 无论从哪个点 启动 bellman ford， 总有一些 点的 price 是 无限大。 为了避免这个问题，我们==总是 添加一个 s 到 图中==，s 到其他所有点的 权重是0，但 s 只有出边，没有入边。

下面的 Johnson 的伪代码。 算法的 时间复杂度 取决于 Dijkstra算法。 我们花费 `O(VE)` 运行一次 bellman ford， `O(VElogV)`运行了 V次 Dijkstra。 `O(V+E)`执行其他记录。
所以 总体时间 是 `O(VElogV) = O(V^3 * logV)`。 负权边 并不会 拖慢速度

![d718aa84cd4c6802cc6e4917e12e86b8.png](../_resources/d718aa84cd4c6802cc6e4917e12e86b8.png)


## 9.5 Dynamic Programming

我们也可以直接使用 dp， 而不是调用 单源最短路径算法 ， 来 解决 点对间最短路径。

对于稠密图， dp 比 Johnson 更快 更简单。

本章接下来的部分，我们==假设 图 没有 负权环==。

dp的递归式
![009353ae1f20dcdddfeb10a6d2a25657.png](../_resources/009353ae1f20dcdddfeb10a6d2a25657.png)

只有图是 DAG时，才能工作。 有向环 会导致 死循环。

我们可以 通过 引入 额外的 参数 来 打破死循环， 和 bellman ford 中做的一样。
设 `dist(u,v,l)` 表示 u到v的 使用最多 l 条边的 最短路径。 由于2个点间 最多 V-1 条边，所以 `dist(u,v,V-1)` 是 最终的 最短路径距离。
Bellman 的 单源递归式 可以立刻适配这种情况
![43d2641abdccd1bb56c62962b5d2d391.png](../_resources/43d2641abdccd1bb56c62962b5d2d391.png)

将这个递归式 转为dp算法 很简单。
`O(VVE) = O(V^4)`

```
Shimbel_APSP(V,E,w):
    for all vertix u
        for all vertix v
            if u == v
                dist[u,v,0] = 0
            else
                dist[u,v,0] = INT_MAX
    
    for l = 1 to V-1
        for all vertix u
            for all vertix v != u
                dist[u,v,l] = dist[u,v,l-1]
                for all edge x->v
                    if dist[u,v,l] > dist[u,x,l-1] + w(x->v)
                        dist[u,v,l] = dist[u,x,l-1] + w(x->v)
```

和 bellman ford 的变形一样，我们不需要 最内层的 v的循环 或 下标l的迭代
```
all_pair_bellman_ford(V,E,w):
    for all vertex u
        for all vertex v
            if u == v
                dist[u,v] = 0
            else
                dist[u,v] = INT_MAX
    
    for l = 1 to V-1
        for all vertex u
            for all edge x->v    // x->v  不是 u  。。这个应该就是 kij 3层for循环。 不，这里时 ijk 3层循环。
                if dist[u, v] > dist[u, x] + w(x->v)
                    dist[u, v] = dist[u, x] + w(x->v)
```




## 9.6 divide and conquer

315

我们可以做进一步的优化， bellman 的递归将 最短路径 分为 一个更短的路径 + 一条边。
我们可以将 最短路径 在中间节点断开，分为 2个更短的最短路径。这样我们就有了不同的递归式，本次递归要在 l=1的时候停止，而不是 l=0， 因为 只有1条边的path 没有中间节点。为了简化，我们设置 w(v->v) = 0.

![89885b634a4721b4bf1606eb4715ebff.png](../_resources/89885b634a4721b4bf1606eb4715ebff.png)

如上所述，递归只能在 l是 2的次方时 才能工作。这无所谓，因为 `dist(u,v,l)`当l >= V-1 时，依然是正确的。 所以 l 需要等于2^(upper(logV)) < 2V

`O(V^3 logV)`

`dist[u,v,i]` 表示 `dist(u,v,2^i)`

```
FischerMeyerAPSP(V,E,w):
    for all vertex u
        for all vertex v
            dist[u,v,0] = w(u->v)
    for i = 1 to upper(logV)    // l = 2^i
        for all vertex u
            for all vertex v
                dist[u,v,i] = INT_MAX
                for all vertex x
                    if dist[u,v,i] > dist[u,x,i-1] + dist[x,v,i-1]
                        dist[u,v,i] = dist[u,x,i-1] + dist[x,v,u-1]
```

和之前的算法不同，这个算法 不再是 调用V次 SSSP。
最内层的循环也不再relax tense边。
我们可以移除 数组的 最后一维， 和之前一样。 这降低空间复杂度 从 `O(V^3)` 到 `O(V^2)`。

```
LeyzorekAPSP(V,E,w):
    for all vertex u
        for all vertex v
            dist[u,v] = w(u->v)
    for i = 1 to upper(logV)        // l = 2^i
        for all vertex u
            for all vertex v
                for all vertex x
                    if dist[u,v] > dist[u,x] + dist[x,v]
                        dist[u,v] = dist[u,x] + dist[x,v]
```


## 9.7 Funny matrix multiplication

有向图中计算最短路径 和 矩阵中计算次方 有紧密连系。

对比下面的算法
```
MatrixSquare(A):
    for i = 1 to n
        for j = 1 to n
            A'[i,j] = 0
            for k = 1 to n
                A'[i,j] = A'[i,j] + A[i,k]*A[k,j]
```

```
FischerMeyerInnerLoop(D):
    for all vertex u
        for all vertex v
            D'[u,v] = INT_MAX
            for all vertex x
                D'[u,v] = min{D'[u,v], D[u,x] + D[x,v]}
```

区别只是第二个算法使用 加法 而不是乘法， 且使用 min 而不是 累加。

。。。一大堆。。

2018年，最快的 metrix-multiplication算法可以 `O(n^2.372864)`, 但是 无法仿照这些算法 来优化 APSP的dp形式。

有其他优化方法，
1991年的 Zvi Galil， Oded Margalit，发现了 simple randomized algorithm
1992，Raimund Seidel， 无向无权图计算 APSP 时间复杂度为 `O(M(V)logV)`， `M(n) = O(n^2.372864)`

上面的发现，使得 边权较小的 有向图 的 最短路径的时间 是 次立方 (。接近立方)


## 9.8 ==Floyd-Warshall==

(Kleene-Roy-)Floyd-Warshall(-Ingerman)

最快的dp算法 在 最坏情况下 比 Johnson算法 慢 `O(logV)` 倍

最短路径的 另一种公式 移除了这个 倍数。

Warshall 使用了 不同的 第三个参数。

之前是 考虑 有上限数量的边的 path， 现在 考虑 通过指定点的path。  通过是指 进入和离开。

将 点 描述为 1到V 的数字，对于每对 点u和v，和任意的整数r，我们定义 路径 `π(u,v,r)`: 
`π(u,v,r) 是 u到v 的 路径中 点的标号 最大是r 的 最短路径`

`π(u,v,V)` 就是 u到v 的 最短路径。

这些路径 有 递归的结构

- 路径 `π(u,v,0)` 不能 通过任何中间点，所以 是 u->v 这条边。
- 对于 r>0， `π(u,v,r)` 通过 点r 或 不通过
  - 如果 通过 点r，那么 它包含了 u到r 的 子路径，然后连着 r到v 的子路径。 ==这些路径 通过的点 的编号 最多是 r-1==. 而且在这种限制下，这些子路径越短越好。 所以 2个 subpath必然是 `π(u,r,r-1)` 和 `π(r,v,r-1)`
  - 如果没有通过r，那么 通过的 点的 标号 最多是 r-1， 且它必然是 该限制下的 最短路径。所以 `π(u,v,r) = π(u,v,r-1)`

设 `dist(u,v,r)` 是 路径`π(u,v,r)` 的 长度。递归式如下

![4a8b0e2e04117cbcb58e923ae7074e8e.png](../_resources/4a8b0e2e04117cbcb58e923ae7074e8e.png)

我们的目标是 对所有的u,v对 计算 dist(u,v,V)。
`O(V^3)`

```
KleeneAPSP(V,E,w):
    for all vertex u
        for all vertex v
            dist[u,v,0] = w(u->v)
    for r = 1 to V
        for all vertex u
            for all vertex v
                if dist[u,v,r-1] < dist[u,r,r-1] + dist[r,v,r-1]   // 第一个确实是 r-1
                    dist[u,v,r] = dist[u,v,r-1]
                else
                    dist[u,v,r] = dist[u,r,r-1] + dist[r,v,r-1]
```

和之前一样，我们可以移除 最后一维

```
FloydWarshall(V,E,w):
    for all vertex u
        for all vertex v
            dist[u,v] = w(u->v)
    for all vertex r
        for all vertex u
            for all vertex v
                if dist[u,v] > dist[u,r] + dist[r,v]
                    dist[u,v] = dist[u,r] + dist[r,v]
```
。。。kij!!!




# ch10 maximum flows ＆ minimum cut

1950年中期，美空军研究员 Theodore E. Harris 和 美空军将军 Frank S. Ross 编写了一份报告研究了 苏联和东欧的 铁路网络。
铁路网络 被抽象成 44个点的 图，有105条边。 权重 代表了 运输的 速度。
用来计算 苏联 往 东欧运输的 最大速度， 通过移除连接 来破坏网络 的 最小的代价

这是最早的 最大流，最小割 的应用。

对于所有的问题，输入都是 一个有向图 G=(V,E) 和 2个点s，t 代表 source 和target。
最大流问题 就是要计算 s到t 的 最大的运输速度
最小割 要找到 将 s 和 t 断开的 最小的代价

## 10.1 Flows

`(s,t)-flow` 是一个 函数 `f:E->R`， 这个函数 在 除了s,t 外的所有点上 符合下面的约束

`f(u->v) 的连加` == `f(v->w)的连加`    。。。。 就是 入 等于 出。

为了保持简单，我们定义 `f(u->v) = 0` 如果图中 没有 边 u->v 。

flow f 的 ==value== 是 `|f|`。是 点s 的 所有出 减去 点s所有入。

不难证明 |f| 也等于 点t的所有净入。

`∂f(v)` 定义为 点v 的 净出 的总量
除s,t外的 点的 `∂f(v)` 的sum 等于 `∂f(s) + ∂f(t)`

任何 从点 流出的 flow 必然 流入另一个点， 所以 所有点的 `∂f(v)` 的sum 是0， `|f| = ∂f(s) = -∂f(t)`

假设我们有另外一个函数 `c: E->R+`，代表 每条边e 的 非负的 capacity。
我们称flow f 是 ==feasible(可行的)==，如果 每条边 都 : `0<=f(e)<=c(e)`。

通常，我们只考虑 函数c 是固定值 的flow

我们称 flow f 中 ==saturate(饱和的)== 边 e 如果 `f(e) = c(e)`， ==avoid边e== 如果 `f(e) = 0`

maximum flow problem 是 在 给定capacity函数(容量尽可能大) 的 有向图中 计算 `feasible (s,t)-flow`。


## 10.2 cuts

`(s,t)-cut` 是将 所有点 分为 2个集合 S，T。 点 要么属于S，要么属于T

如果我们有 capacity 函数: `c: E->R+`，==cut 的 capacity== 就是 从集合S出发，指向集合T 的 所有边 的 capacity 的sum。 记作 `||S,T||` 。 这个 定义是对称的， 所以 从集合T出发，指向集合S 的边 并不重要。

如果 v->w 没有边，那么 c(v->w) = 0

minimum cut problem 是计算 `(s,t)-cut` 使得 这个 cut的 capacity 尽可能小， 即 `||S,T||` 尽可能小

min cut 是 代价最小的 将 s 到 t 的 所有flow 中断的 方法。

Lemma 10.1 如果 `f` 是任意的 `feasible (s,t)-flow`, `(S,T)`是任意的 `(s,t)-cut`。 `f` 的value <= `(S,T)`的最大capacity。 此外， `|f|=||S,T||` 当且仅当 f saturate 每天S到T的边 且 avoid 每条从T到S的边。

。。f的value 就是 `|f|`， (S,T)的capacity 就是 `||S,T||`
。。saturate: 充分利用，即 运输的量 等于 边的capacity。
。。avoid: 完全不使用，即 运输的量=0

证明: 选择你最喜欢的 flow f， 你最喜欢的 cut (S,T)， 然后应用下面的 不等式
![efb73f305e4779ea18e4f8d523506a3f.png](../_resources/efb73f305e4779ea18e4f8d523506a3f.png)

。。。。。?

这个推理表明: 如果 `|f| == ||S,T||`，那么 f 必然是 最大流，(S,T) 必然是 最小割。


## 10.3 maxflow-mincut theorem

在每个 flow network中，必然有一个 `feasible (s,t)-flow f` 和一个 `(s,t)-cut (S,T)`, 使得 `|f| = ||S,T||`。  这就是 著名的 Maxflow-Mincut Theorem: `在每个 source s, target t 的 flow network中，max (s,t)-flow 的 value 等于 min (s,t)-cut 的 capacity`

证明:  p331 - 333 。。。。。`reduced`,`residual capacity`剩余容量,`residual graph`剩余图， `augmenting path`增广路径


图是 ==reduced==，意味着 任意2个点 u,v 之间 最多一条 edge。  。。就是没有那种 有来有去的，  要么 去，要么 来
如果2个点之间有 多条边，那么 每条边 中间 增加一个节点， 方向不变， 新的边的权 是 原来的 边的权。  。。但是路径上会有2条边。
![b9a2545960f42911df04d678c2f23ff4.png](../_resources/b9a2545960f42911df04d678c2f23ff4.png)

---

设 f 是 feasible (s,t)-flow，我们定义一个 新的 函数 c~f~: V * V -> R， 称为 ==residual capacity==， 如下所示
![88665470e8e8e6fbe3fe93922cb81b7a.png](../_resources/88665470e8e8e6fbe3fe93922cb81b7a.png)
。。注意图里的 u->v,  v->u， 而且之前说过 把图改成 reduced的。所以 2个 if最多生效一个。

边的 residual capacity ==代表== 这条边 还可以 放多少 流量。
因为 f>=0 且 f<=c，所以 residual capacity 总是 非负的。

可能出现 c~f~(u->v) > 0, 即使 u->v 不是 原图中的 边。

我们定义 ==residual graph== G~f~=(V,E~f~)。 E~f~ 是那些 residual capacity为正的 边的集合。
大多数 residual graph 不是 reduced。 特别是，如果 0 < f(u->v) < c(u->v), 那么 residual graph G~f~ 会 同时包含 u->v 和 v->u
。。但是之前不是 已经假设 图是 reduced 的吗?
。。而且看下面的图， 是和 flow f 有关的，  所以 G~f~ 是 图G 上的 确定flow f 之后的 图。

。。就是 ==先有G，然后把G改成 reduced，然后确定一个flow，然后根据flow，确定 C~f~， G~f~。 G~f~是一个新的图，不是G，这个新的图可能 2个点 有2条边，分别来去，所以不是 reduced==

![c7e9ac760456e7056dc98225a9751f3a.png](../_resources/c7e9ac760456e7056dc98225a9751f3a.png)

现在我们有2个case要考虑:  在 residual 图 G~f~ 中 存在 一个 从s到t 的 有向路径 ， 或不存在这样的 有向路径。

首先假设 residual graph G~f~ 包含了 一个 从s 到t 的 有向路径 P， 我们称P为 ==augmenting path==。
设 F 是 P中 所有边的 c~f~(边) 的最小值。 代表了 我们可以通过 P 运输的 最大流量。

我们定义一个新的flow  f': E->R, 如下
![5577b8db26e647277daa3d1fc07a0217.png](../_resources/5577b8db26e647277daa3d1fc07a0217.png)

![ac364ce46506bb4e5c22b3919e624f03.png](../_resources/ac364ce46506bb4e5c22b3919e624f03.png)

我宣称 这个新的 flow f' 在原来的capacity c下是 feasible， 意味着 任何地方 f' >= 0 且 f' <= c。 考虑 原图中的 一条边 u->v, 有3种情况
- 如果 增广路 P 包含 u->v， 那么 `f'(u->v) = f(u->v) + F > f(u->v) >= 0` 。 因为。。。略。
- 如果 增广路 P 包含 u->v 的反向边 v->u，那么 `f'(u->v) = f(u->v) - F < f(u->v) <= c(u->v)` , 因为 。。。 略
- 如果 u->v, v->u 都不在 P中。 那么 `f'(u->v) = f(u->v)`, 因此 `0 <= f'(u->v) <= c(u->v)`
所以 f 真的是 feasible 。

最后，增广路劲中 只有 第一条边 离开 s，这意味着 `|f'| = |f| + F > |f|`。 所以 f' 是 feasible flow 且 它的value (。传输的流量。搜索`|f|`) 比 f 更大。 
我们断定 在 residual graph G~f~ 中有一条 s到t 的路径，所以 f 不是 最大流。

---

假设 剩余图 G~f~ 中没有 s 到t 的 有向路径。
设 S 是 G~f~ 中 从s 可到的 所有点的集合， T 是 V-S。 明显，`(S,T)`是一个 `(s,t)-cut`。
对于 S中的每个点u，T中的每个点v，都有 `c~f~(u->v) = (c(u->v) - f(u->v)) + f(v->u) = 0`

f 的 feasibility 表明 `c(u->v) - f(u->v) >= 0` 和 `f(v->u) >= 0`， 所以 我们必然有 `c(u->v) - f(u->v) = 0` 和 `f(v->u) = 0`。 换句话说，我们的flow f saturate S到T 的每条边，且 avoid T到S的所有边。
现在 Lemma 10.1 表明 `|f| = ||S,T||`，这意味着 f 是 最大流，(S,T) 是 最小割。

证毕!


。。
剩余图，是指 原图 减去一条边的流量后，添加一条反向边 来表示 流量的 反向流动。 这种表示方法可以帮助我们 更好地理解 和优化 网络流问题。
剩余图中的 边权重 表示 当前边的 剩余容量， 反向边的权重 表示 已经通过该边的 流量。

增广路径: 若 P是图中 一条连接 2个未匹配顶点的路径，并且属于M的边 和 不属于M的边 (即 已匹配和 待匹配的边) 在 P上交替出现，则 P 是 相对于M 的 一条增广路径。

。。这些是baidu搜的， 和书上的差好多。
。。


## 10.4 增广路径(augmenting-path)算法

Ford and Fulkerson's augmenting-path algorithm

Ford 和 Fulkerson 的 对 最大流-最小割理论的 证明 立刻就导致了 计算最大流的 算法: 从 zero flow开始，重复 在 residual graph 中 沿着任何 s到t的path  augment(增加,增广) flow ，直到没有 这样的path

这个算法有一个 重要且直接 的推论:
Integrality Theorem，如果 flow network中所有的 capacity 都是 整数，那么 存在一个最大流，它使得 通过每条边的 流量都是 整数

证明: 。。。。


如果 每条边的 capacity 是整数，那么 Ford-Fulkerson 算法 最多执行 |f*| 次迭代后 停止， 这里 f* 是 真正的最大流。 在每次迭代中，我们可以构建 residual graph G~f~ ，然后执行一个 whetever-first search 在 `O(E)` 时间内找到一个 augmenting path。
算法时间复杂度 `O(E|f*|)`

Ford and Fulkerson 的算法 通常很快， 如果 max flow value |f*| 是 small的话，它一直很快。
但是 augmenting path 上没有 更多的约束的话， 在最差情况下 这不是一个 高效的算法。

---

Irrational Capacities 不合理的容量

如果容量不是 整数 会发生什么?
如果我们将 所有容量 乘以 一个常数， 最大流 也会增加相同常数。

如果所有边的 容量是 合理的(rational)，那么 Ford-Fulkerson 算法 可以终止，但是是 指数时间。

但是，当我们允许 不合理capacity， 算法 会 死循环， 不断地 找到 越来越小的 augmenting path。
更糟的是，这个 augmentation 的 无限的序列 不会 向 最大流 收敛。

。。这里举了一个例子，6个点的例子。    略。

![7381b2096bae9bd536e5da6683cf45c8.png](../_resources/7381b2096bae9bd536e5da6683cf45c8.png)

。。这个例子的很多信息。略


## 10.5 combining and decomposing flow

组合 和 分解 flow

336

flow 通常定义为 图的 edge上的 在点上满足约束的 函数。

但是，flow有第二个 更自然，更有用的 特征。

假设 图G，source s，target t。 找到 2个 `(s,t)-flow` f 和g，2个实数 a 和b， 考虑 函数 h: E->R，定义如下
![2c53bac8340e9894927c7d9621f63671.png](../_resources/2c53bac8340e9894927c7d9621f63671.png)

对于每条边 u->v，我们可以 将定义写得更简单: h = af + bg

定义暗示了 h 也是一个 (s,t)-flow, 值 |h| = a|f| + b|g|

更一般地，任何 (s,t)-flow 的 线性组合 都是 一个 (s,t)-flow
。。不考虑 capacity?  那a=1000000，b=100000， 岂不是 flow 的 value 可以无限大?


任何 (s,t)-flow 都可以被 写为 多个flow 的 带权sum。

对于任何 从s 到t 的 有向路径 P，我们定义 对应的 ==path flow== :
![38872681e3e5edeb54cfa93ad23e92a3.png](../_resources/38872681e3e5edeb54cfa93ad23e92a3.png)

上面的定义 暗示 函数 P: E->R  是一个 value为1的 (s,t)-flow。

这里故意重用了 P，使得 P 即是 path 又是 该path上的 ==unit flow==

类似的，对于任何 有向循环 C，我们定义对应的 ==cycle flow== :
![a7c1e48a538f2e43dce09498e8ff34b1.png](../_resources/a7c1e48a538f2e43dce09498e8ff34b1.png)

很容易就证明 C:E->R 是 一个 value 为0 的 (s,t)-flow
。。循环，所以 value 必然0


path flow 和 cycle flow 的 线性组合 是 另一个 flow。 这种 weighted sum 被称为 ==flow decomposition==。 此外， 每个 非负 flow 都有 一个 使用下面的特殊结构的 flow decomposition

==Flow Decomposition Theorem==: 每个 非负 (s,t)-flow f 可以被 写成 directed (s,t)-path==s== 和 directed cycle==s== 的 系数为正数的线性组合。 此外， 一个 directed edge u->v 会在 这些 paths 和 cycles 中至少出现一次， 当且仅当 `f(u->v) > 0`, 而且 paths 和cycles 的总数量 不会超过 网络中的 edge的数量。
。。为什么 u->v 必然出现?  不经过 u->v 的path， 不经过 u->v 的cycle 也可以组合起来啊。

proof: 。。。。337->339 。。 circulations， acyclic(s,t)-flow


## 10.6 Edmonds and Karp's Algorithms

Ford-Fulkerson 的算法 没有 指出 在剩余图中 应该对哪条path 进行增广。 如果 增广路径的选择很差的话，算法的表现会很差。

1970s，Jack Edmonds 和 Richard Karp 提出了2个 选择 增广路的 规则。 使得算法更高效

---

Fattest Augmenting Paths

第一个规则是 一个 贪婪算法
`Choose the augmenting path with largest bottleneck value`

不难发现 有向图的 maximum-bottleneck (s,t)-path 可以 使用 best-first traversal (类似 Jarnik(Prim)的 min span tree算法 或 Dijkstra的 最短路径算法)  在 `O(ElogV)` 时间内计算出来。
算法 以s 为root，每次生长一个点，生长出 一棵有向树T， 通过 重复 添加 离开T 或到T的 edge中 最高capacity的 edge。(repeatedly adding the highest-capacity edge leaving T to T), 直到 T 包含了 一条 s到t 的path。
或者，也可以 仿照 Kruskal算法: 以 capacity 降序，每次添加一条边，直到 有 s到t 的 path。 不过这个 不够效率。

要进行 完成运行时间分析，我们需要 一个 算法停止前的 迭代次数的上界。 实际上，对于 实数 capacity，算法 可以死循环。 对于 整数 capacity，算法的 迭代次数的上界 是 max flow 的value |f*|
。。这里和之前 提到的 实数，整数，应该是指 max flow 的边 是否可以 实数。 因为 图的边的权 完全可以乘以 100000 来转为 整数的，  不， 实数 一般应该指 1/3, 而不是 1.234。

设f是 G中 的任意flow， f' 是 当前剩余图G~f~ 中的 最大flow。 (在算法的开始时， G~f~ = G， f' = f*)。 我们已经证明了 f' 可以分解为 最多 E 个paths 和 cycles。 通过平均，我们可以知道，至少有一个 path 可以 运输 至少 |f'|/E 单位。 所以 G~f~ 中 fattest (s,t)-path 至少可以 运输 |f'|/E 单位。

因此，G~f~ 中 max-bottleneck path 的 增广f 把 G~f~ 的 remaining max flow 的值 增加了 最多 1 - 1/E 倍。
换句话说，G~f~ 的 residual max flow value 指数退化。 在 E * ln|f*| 次 迭代后， G~f~ 中的 最大flow value 最多是 
![ea97caacfa0b3c15aa370f821e5bc507.png](../_resources/ea97caacfa0b3c15aa370f821e5bc507.png)

上面的e是 欧拉常数。 不是 边。
在 E * ln|f*| 次迭代后， residual max flow value 小于 1. 如果所有的 capacity 是 整数，那么 residual max flow value 也是整数，所以 必然是 0.  即 f 是 max flow

我们考虑 整数capacity的 图， Edmonds-Karp 的 "fattest path" 算法 时间复杂度 `O(E^2 * logE * log|f*|)`。  不同于 Ford-Fulkerson 算法，这个时间上界是 输入的size 的 多项式函数。

和 传统的 Ford-Fulkerson算法一样，"fattest path"算法 遇到 实数capacity 时 会 死循环。

但是 我们的分析 显示: 即使 算法 死循环，它 依然维护了一个 越来越接近 max flow 的 flow f


---

Shortest Augmenting Paths

Edmonds-Karp 的第二个 rule 是 受到Ford-Fulkerson 的启发:
`choose the augmenting path with the smallest number of edges`

通过 bfs 在 `O(E)` 找到 最短增广路径。
算法会在 迭代次数的 多项式 之后停止， 和 edge capacity 无关。

多项式 上界的 证明依赖于 对 residual graph 的变化 观测到的2个现象。
设 f~i~ 是 i次增广后的 当前flow，G~i~ 是对应的 residual graph。 f~0~ 是0， G~0~ = G。
对于每个点 v， level~i~(v) 代表 G~i~中 从s 到v 的 无权最短路径长度， 即 v的 level 是 G~i~ 中以 s为出发点的 bfs 的 层数。
如果 G~i~ 中没有 s到v 的path，那么 level~i~(v) = 无穷大

观察到的第一个现象是: 点的 level 随着时间推移 只会 增加。
Lemma 10.2 对于所有点v 和 所有 大于0 的i，都有 level~i~(v) >= level~i-1~(v)

proof: 。。。。。
。片段: 假设 s-..-u-v是 G~f~ 中的 无权最短路径，如果 u-v 不在 G~i-1~中，那么 v-u 必然在 G~i-1~中，且它是 最短路径。


当我们 增广 flow时， 增广路径 中的 bottleneck 边 会从 residual graph 中消失，  增广路径的 resersal 中的一些边 可能重新出现。
观察到的第二个现象是 一条边 无法 太多次 出现 或消失。
Lemma 10.3 在 Edmonds-Karp 最短增广路径算法的 执行期间，每条边 从 剩余图 G~f~ 中 最多消失 V/2 次。

proof: 。。。。。


算法最多执行 EV/2 次迭代，每次迭代需要 O(E) 时间，所以 算法总体是 `O(VE^2)`



## 10.7 Further Progress

最大流算法 远没有结束。
下面是一些 更进一步的研究的 成果。
所有列出的算法 都要 数个迭代 才算出 max flow
大多数算法 有2个变种: 
- 更简单， 每次迭代时 brute force
- 更快，使用复杂的数据结构 来维护 流网络的 spanning tree。 所以每次迭代可以 对数时间

![d68dc79db6bf9d05940678f61d9a6ef6.png](../_resources/d68dc79db6bf9d05940678f61d9a6ef6.png)

已知的最快的 最大流算法 是 2012年的 James Orlin， `O(VE)`， 差不多是 flow decomposition 的复杂度。  Orlin的算法 远超本书的范围

。现在应该是 Rasmus Kyng 的 近乎线性时间。  网上看到是 `m^(1+O(1))`， m是边的数量 。。。这不是 m^2 吗? 但是 说 线性。。


# ch11 application of flow and cut

## 11.1 edge-disjoint paths

max flow 最早的应用之一 是 在 有向图G 中 计算 2个点s,t 之间 的 edge-disjoint path 的最大数量。

G中的 path 的集合 是 edge-disjoint ，如果 G中的每条边 最多出现在 一个 path 中。  点可以出现在多个 path中。

如果我们将 每条边的 capacity 设置为1， 那么 最大流的value 就是 edge-disjoint path 的数量。
flow-decomposition 理论 表明 由 saturated edge 组成的 子图 S 是 多个 edge-disjoint path 和 cycle 的 组合。 从S中提取 path 很简单: 沿着 每条  S中的 s到t 的 有向路径， 从S中移除 该有向路径，然后 递归。

我们也可以转换  包含 k个 edge-disjoint path 的集合 为 一个 flow。 这个flow 的value 就是 k。
因此，max flow 算法 计算了 edge-disjoint path 的最大集合。

如果使用 Orlin算法来计算 max flow，那么我们可以在 `O(VE)` 内计算出 edge-disjoint path， 但是 Orlin 的算法 对于这个 简单应用来说 太难了。
割 ({s}, V-{s}) 的 capacity 最多 V-1, 所以 max flow 的value最多 V-1。
因此 Ford-Fulkerson 的 原始的 augmenting path algorithm 可以 `O(|f*|E) = O(VE)`

相同的算法可以用于 寻找  无 向 图 中的 edge-disjoint paths。
第一步，将每个无向边 u-v 转为 2个有向边 u->v, v->u， 每个都是 unit capacity。
然后对 有向图 使用 Ford-Fulkerson 算法，计算 最大流
对于 无向图中的 边 u-v，如果 最大流 使用了 u->v 和 v->u，那么 可以从 flow 中移除 这2条边，而不会影响 flow 的value。
这样， 最大流 就 使用了 saturate edge 的 一个方向  (。有向图是2个方向，现在只用了一个方向，那么就是 无向图了)。 就可以 提取出 无向 path 


## 11.2 vertex capacities and vertex-disjoint paths

。。11.1 是边 disjoint。 现在是 点

假设 图G 的每个 点 有 capacity，边也有capacity。

这样， 除了s,t 外的 每个 点v，我们要求 进入 v 的 flow ( 等于出v 的flow) 必须 小于 `c(v)`

我们可以在这种限制下 使用 之前的 max flow 算法?

1962 Ford，Fulkerson，提出:  将每个点v 替换为 v~in~, v~out~， vin->vout 的capacity是 `c(v)`, 每个有向边 u->v 替换为 u~out~ -> v~in~，维持原来的 capacity。
转换后图中的 (sout,tin)-flow 就是 原图中的 feasible (s,t)-flow 。
使用 Orlin算法， `O(VE)`

计算 s到t 的 vertex-disjoint path的最大数量， 现在可以 `O(VE)`， 只需要 将每个 点的 capacity设置为1， 然后执行 max flow
![65cc295d5d217d4fcb41fa10324043be.png](../_resources/65cc295d5d217d4fcb41fa10324043be.png)



## 11.3 Bipartite Matching

2部分组成的 匹配

最大流的一个应用 是 在 bipartite graphs 中找到 最大的 matchings

==matching== 是 一个子图，在这个子图中 每个点 的 degree 最多是1。 即， 边的集合，集合 中 没有2条边 共享一个 点。

这里的问题是 找到一个 有着最多数量的 边 的matching

例如，我们有 一个医生的集合，每个医生都有他 想要去的医院列表， 一个医院的集合，每个医院都有 它愿意接收的 医生的集合。 我们的任务时 找到 最多的 医生-医院 匹配，并且 匹配 符合 他们的 意愿。

这个问题就是 找到 在 bipartite 图中找到 最大 matching

。。这个就是之前的  贪婪啊， 就是 按照 医院的意愿 从高到低 给 还没有拒绝过该医院的医生 发送 offer， 如果 医生 还没有确定，或 该医院 比 临时确定的 医院 优先级更高，那么 就 临时 接受offer，并拒绝 之前的offer。
。。不过 ，忘记了 细节。 就是 是医院发完后 下一家医院，还是每家医院发一个，轮流发。 还有，一次性发完的话，多个医生接受了，那怎么处理?

这个问题等价于 在 医生和医院作为 点的 bipartite graph中，找到 最大matching， 并且这里的边 是 医院和医生 都可以接受对方 才会有边。

我们可以 将这个问题 降级为 max flow 问题 来解决它:
设 G 是 bipartite graph，点是 医生+医院，医生，医院之间有边相连。
我们可以创建一个新的 有向图，通过:
1. 将边 从医生指向医院
2. 增加 一个 source点s，它 和每个 医生有边相连
3. 增加一个 target点t，它和 每个医院相连
4. 每条边的 capacity 是1

图中的 每个 matching， 都可以转为 一个flow: 

。。。好繁。。 大约的意思知道。
p335。还有很多。。

![2fc16ee2bb9706b1825cef8970260cd9.png](../_resources/2fc16ee2bb9706b1825cef8970260cd9.png)


## 11.4 tuple selection

bipartite maximum matching 问题是 被我称为 tuple selection 问题的 简单例子。

tuple selection问题 的输入 是 一些有限集合 X1,X2..Xd，每个 代表了 不同的资源。
我们的任务时 选择 d-tuple 的最大的集合，每个 d-tuple 包含了 从每个 Xi 中提取的 一个元素，收到以下的限制
- 对于每个i，Xi 中的 每个元素x 可以出现在 最多 c(x) 个 selected tuple 中。
- 对于每个i，2个分别属于 Xi 和 Xi+1 的元素 x, y 可以出现在 最多 c(x,y) 个 selected tuple 中

c(x), c(x,y) 的上界 要么是 非负整数，要么是 无穷大。

在 max matching 问题中， 我们有 d=2 个资源，每个元素x 的capacity c(x)=1, 每对 (x,y) 的capacity c(x,y) = 1 或 = 0 ，依赖于 xy 是否是一条边。

由于资源是 全序的 (linearly ordered)， 相邻的集合 中的pair元素 才有限制。 所以 tuple selection 问题可以 转为 图G的 max flow问题，图G的定义:
- 图G中，每个集合的元素 作为一个 点，额外还有一个 source 点s，一个 target点 t。 每个点 (除了s和t) 的 capacity 是 c(x)
- 图G中，对于X1中的每个元素w，都有一个 s->w 的边。 对于Xd中的元素w，都有一个 w->t 的边。 且Xi的元素x，Xi+1的元素 y 组成的 pair 也有一条边, 且这条边的 capacity是 c(x,y) ( 一般，如果 c(x,y)=0, 那么可以忽略 边 x->y)

G中 s到t 的 每个 path 就是一个 我们可以select的 d-tuple。

![56750fa508bece342f927cba4d4857a7.png](../_resources/56750fa508bece342f927cba4d4857a7.png)

一般地说，设 f 是 G中的 feasible interger (s,t)-flow。 由于 所有的 capacity 要么是整数，要么无穷大， Flow Decomposition理论 表明 f 是 s到t的 |f| path 的 ==sum==，每个flow携带 一个unit。。(f is the ==sum== of |f| paths from s to t, each carrying exactly one unit of flow)
定义表明 tuple 的最终set 满足 所有 capacity约束。 反之，对于 满足 capacity约束的 任何 k tuple 的集合， 对应的 k个path 的 ==sum== 是 value是k 的feasible integer (s,t)-flow
。。上面的==sum== 应该是 path的 叠加。

而且，我们可以选择 最大数量的 满足给定capacity约束 的tuple， 通过 计算 G中的 max (s,t)-flow f* 和 f* 的一个flow decomposition。 因为 G 中所有 有限的capacity 都是 整数，我们可以 确定 f* 是一个 整数flow， 因此，相当于 包含|f*|个 tuple 的 有效的set


---

Exam Scheduling

学校要求你写一个算法 来 schedule 他们的final exam。
有n门不同的课程，t个可以考试的时间，r个教室。 你需要为 每门课程 指定 考试时间和 教室。
每个教室 的每个可以考试的时间 最多 进行一场考试。
每门课程 只能在 一个教室，一个时间点 进行考试。

每次考试 必须有 一个监考， 一共有 p个监考。 每个监考 同一时间 最多监考一场考试。
每个监考 有不同的 空闲时间，空闲时间才可以监考。
每个监考 总共 监考 不能 超过 5场考试。

问题的输入是 3个数组
- `E[1..n]`， 每个元素代表，第i门课程的 学生人数
- `S[1..r]`， 每个元素代表，第i个教室的 可容纳的参加考试的学生数
- `A[1..t, 1..p]`， `A[k,l]=true` 当且仅当 第l个监考考试 在 第k个时间段 可以进行监考。

设 N = n+r+tp，代表 输入的 total size。 你的任务是 设计一个算法  为每个课程 指定 一个room，一个time slot，一个 监考 用于 考试。  或者 如果无法安排，则 返回 NO

这是一个 标准的 tuple-selection 问题，有4个资源: class, room, time slot, proctor(监考).
要解决这个问题，我们需要构造一个 包含6种类型(4个资源+s+t)的 点， 5种类型(s->class->room->time slot->proctor->t)的边 的 流网络G :
![7dce02191fad76638306344ec2c3647a.png](../_resources/7dce02191fad76638306344ec2c3647a.png)

- 边 s'->c~i~，s到每个class都有边。 边的 capacity都是1 ( 每个class只有 一场考试)
- 边 c~i~->r~j~, 只要 `E[i]<=S[j]`就有边。边的 capacity是无穷大。
- 边 rj -> tk, room 和 time slot 是 全映射， 边的 capacity是1 (因为一个room，一个时间点，最多一场考试)
- tk->pl， 只要`A[l,k]==true` 就有边， 边的 capacity是1， (因为一个监考，一个时间点 只能监考一场考试)
- pl->t', 每个proctor都有边指向 t'， capacity都是5 (因为 监考的总数不能超过5场)

上面用 s',t' 是因为 t代表 time slot 的数量。

每条从 s' 到t' 的path 代表了 一个考试的 唯一的 class,room,time,proctor 的选择， 并且 room可以坐的下，proctor 有空可以监考。
换句话说， 对于每个 有效的 考试， 必然在G中有一条 对应的 path。

所以我们可以 通过 计算 G中的 最大(s',t')-flow 来 知道 考试的 最大数量， 通过 decompositing f* 成 s'到t' 的path 来获得 考试的schedule。
如果 |f*| >= n，我们可以成功schdule，否则 无法schedule。

从给定的输入 通过 brute force 构造G，需要 O(E)。
O(VE) 使用 Ford-Fulkerson(因为 |f*| <= n < V) 或 Orlin 计算 最大流.
flow decomposition 需要 O(VE)
所以最终 `O(VE) = O(N^3)`


## 11.5 Disjoint-Path Covers

有向图G 的 ==path cover== 是 G中 有向path 的 集合， 使得 G中每个 点 至少在 一个 path中。

G的 ==disjoint-path cover== 是一个 path cover， 其中 G的每个点 属于一个 path。

。。11.1，11.2 是 边 最多 出现在一个path中。  这里是 点 至少在一个path， 点属于一个path

每个有向图 都有一个 trivial disjoint-path cover， 它包含了 多个 长度为0 的 path， 但这很无聊。
我们的目标是 找到一个 包含 的path 最少的 disjoint path cover

这个问题是 NP-hard， 一个图有 size为1的 disjoint-path cover，当且仅当 它包含一个 Hamiltonian path， 但 对于 有向无环图，有一个高效的 基于flow 的 算法

要在 有向无环图 G=(V,E) 中解决这个问题，我们构建一个 新的 bipartite graph G'=(V',E'):
- 对于G的每个点v， 在G'中分为 v^b^ 和 v^#^
- 对于G的每条边 u->v， G'中包含 无向边 u^b^-v^#^

![3955dcdbb377ec7752001afd8ae0e987.png](../_resources/3955dcdbb377ec7752001afd8ae0e987.png)

现在我声明 G可以被 k个 disjoint path cover 当且仅当 G' 有一个 size为 V-K 的 matching。

证明: 。。。361

这样，我们可以 通过 计算 G' 的 最大 matching 来 找到 G的 min disjoint-path cover。

`O(VE)`

尽管它的公式是由DAG和 path组成的，但是 它确实是一个 max matching 问题: 我们希望 match 尽可能多的 vertex 到 distinct successor。
cover DAG的 path 的数量  等于 没有 successor的 点的 数量。 ( 并且，每个 bipartite max matching 问题 都是 一个 flow 问题)


---

Minimal Faculty Hiring

学校由于预算减少，需要减少 教师数量， 但是 由于学生已经付了学费，所以 学校必须维持 足够的 教师 来确保 课程目录中的 课程依然可以上课。
某个教师的 课程 不能overlap ，必须有足够的 空闲时间 来让 教师从一个上课地点 走到 下一个上课地点。
为了简化问题，我们假设: 每个教师 有能力教授 所有的课程，而且他们没有休息时间，没有吃饭时间

具体来说，假设 m个不同的地点，n门课程， 输入:
- 数组`C[1..n]` 代表课程， `C[i]` 有3个属性: `C[i].start, C[i].end, C[i].loc` 分别是 课程的 开始时间，结束时间，地点
- 二维数组`T[1..m, 1..m]`， `T[u,v]` 代表 从 location u 走到 location v 需要的时间。

我们希望找到 可以 教授每门课程的 最少的 教师数量。
教师可以教授2门课程 i,j ( 已知: `C[j].start >= C[i].start`)，只要 `C[j].start >= C[i].end + T[C[i].loc, C[j].loc]`

我们可以把这个问题 简化为 disjoint-path cover 问题:
构建 DAG G=(V,E)， 点是 class，边代表 可以由同一个 教师 来上课。

有向边 i->j，说明 同一个教师 可以 教授 课程i 和课程 j。

可以在 O(n^2) 内 brute force 构建 DAG。
然后使用 matching 算法 来找到 G 的 disjoint-path cover。 G中的每个 path 代表了一个教师的 课程安排。
`O(n^2 + VE) = O(n^3)`

尽管问题的描述 由 时间间隔，距离 组成，但这 的确是 max matching 问题 ( 这意味着 确实是一个 max flow 问题).
我们想要 将尽可能多的课程 和 同一位教师 教的下一门课程进行 match。
我们需要的 教师的数量 等于 没有 assigned successor 的 课程的数量。
每个 没有分配后继者的 课程 是 该教师的 最后一门课。


## 11.6 Baseball Elimination

棒球淘汰


问题的抽象的描述: 输入是 2个数组 `W[1..n]` 和 `G[1..n, 1..n]`
`W[i]` 是 队伍i 已经胜利的 次数。
`G[i,j]` 是 队伍i 和队伍j  还 未 进行的比赛次数
我们希望知道 队伍n 是否可以 在 赛季结束时 获得最多的胜场。

1960s，Benjamin Schwartz 发现这个问题可以 建模为 max flow问题。
20年后，xxx 简化了 1960s的 流的公式 ，使之变成 pair selection 问题。 即，我们想要知道 是否存在一种 指定每场比赛的胜者，使得 队伍n 可以 最终获胜 的方法。

设 `R[i] = 对所有j的累加G[i,j]`，代表了 team i 剩余的 比赛次数。
我们会假设 team n 会赢得剩下的 `R[n]` 场比赛的 胜利。 team n 最终可以获胜 当且仅当 其他队伍i 在 剩余的 `R[i]`场中最多赢了 `W[n] + R[n] - W[i]` 场。

由于我们希望 对每场比赛 select 胜者， 我们开始 构建 bipartite 图，它的 节点 代表了 比赛 和 队伍。
我们总共有 C_n取2 个 比赛节点 g~i,j~，(。。已经比赛过的 也是图的节点 。。后续是通过 capacity 来限制的) ( 1 <= i < j < n) ， n-1个 队伍节点 t~i~ (1 <= i < n)。

对于每对i,j, 增加2条边 gij -> ti, gij -> tj, capacity都是 无限大。
增加一个 source s， s 指向 所有的  gij， capacity 是 `G[i,j]`   。。i和j 还没有比赛的场次
增加 target t，所有的 ti 指向 t， capacity是 `W[n] - W[i] + R[n]`

下图是 1996年的， team n 是 Detroit Tigers

![116321e227438c04764b99e185d65e4a.png](../_resources/116321e227438c04764b99e185d65e4a.png)

定理: Team n可以最终获胜 当且仅当 graph 中有 saturate s的所有出边 的 feasible flow

。。不过上图中， s的出边和 是 27，t的入边和是 26， 所以 肯定不会 saturate，所以 team n 最终获胜的 概率是 0 ?
。。定理里是 可以获胜，就是 有 0.00001% 的几率 也是可以。。
。是的，之后书里说到了 27，26， team n 没有获胜的几率。

证明: 。。。。
1. 假设 team n 最终获胜，那么 其他队伍 最多赢 `Wn+Rn-Wi`场。 对于剩余的比赛，每有一方赢，那么就加一个 unit flow `s->gij->ti->t`。 由于 还有`G[i,j]`场，所以 s的所有 出边 都会 saturate。 由于队伍i最多赢 Wn+Rn-Wi 场，所以 flow 是 feasible (。。这个不懂，为什么是 feasible的，就是 Wn+Rn-Wi 可能很小，导致无法 feasible， 不 feasible 是 used <= capacity,所以必然可以 feasible， 但是 怎么证明 s的出边用完的情况下， 这些流量可以通过 ti->t 呢 ?)
2. 设 f 是 feasible flow saturate 了 s 的所有出边， 。。。


flow network 有 O(n^2) 点， O(n^2) 边， 需要 O(n^2) 时间构建。 使用 Orlin算法，可以 `O(VE) = O(n^4)` 计算 最大流

对于baseball elimination问题 这不是最快的， 2001 Kevin Wayne 证明了 通过高效使用 单次 max flow，可以 `O(n^3)` 获得 所 有 无法获胜的 team



## 11.7 Project Selection

假设我们有 n个project的 集合， 一些project 要 在 完成其他project 后才可以开始。
project 和 它们的依赖 构成了 DAG， 点是project， 边 u->v 代表 u依赖v。
每个project 有 profit，完成后可以获得。  一些project 的 profit 是 负的。

我们可以选择 完成 project的 任何 子集(被依赖的project也必须在子集中)。
我们的目标是 获得 尽可能大的 profit。
如果所有project的 profit都是 负数，那么 正确的答案是 不做任何事情。

从高层看来，我们的任务时 将 project 分为 2个集合 S，T， 代表 select 和 turn down(拒绝)
所以，我们把问题建模为 min cut 问题。 但是 什么图? 怎么表达依赖关系? 我们想要 max profit，但是 我们只知道如何找到 min cut。 还有 如何将 负profit 放到 正的capacity中?

要 将 图 转为 流网络，
- 我们增加source s，target t。 
- 对每个 正profit的 project v，建立 边s->v
- 对于 负profit 的 project u，建立 边 u->t

我们可以将 s 当做 profit为0 的 project。
我们按照下面的规则 给edge赋值 capacity
- c(s->v) = profit(v)， 对于每个 正收益的job
- c(u->t) = -profit(u)， 对于每个 负收益的job
- c(u->v) = 无穷大， 对于每个 u对v 的依赖

所有 capacity 都是正的，所以 可以计算出 max cut

现在考虑 图的 (s,t)-cut (S,T)， 对于 代表依赖的边 u->v，如果 u是S，v是T，那么 ||S,T|| = 无穷大。 所以， S是合法的选择 当且仅当 cut (S,T) 的 capacity 是 有限的。

![fc7c43889bf721d9dec9474614f55e38.png](../_resources/fc7c43889bf721d9dec9474614f55e38.png)

13 = 边的2+3+5+3， 15=点的4+6+2+3

事实上，cut 的更小的 capacity 对应着 更大的 profit

选择S中的 project 可以获得 `P-||S,T||` 的利润。 P是所有 正的 profit的 sum。证明如下:
对于 project的 任意子集X，定义3个值
- cost(X) = X中所有负profit的sum = 所有属于X的u的 c(u->t)的sum
- yield(X) = X中所有 正profit的sum = 所有属于X的v的 c(s->v)的sum
- profit(x) = X中所有 profit的sum = yield(X) - cost(X)

所以 P = yield(V) = yield(S) + yield(T)。
因为cut (S,T) 有 有限的 capacity， 所以只有 s->v，u->t 的 边 能 穿过 cut。

每个s->v 代表了 正profit的 project， 每个 u->t代表了 负profit的 project。所以 ||S,T|| = cost(S) + yield(T)
P - ||S,T|| = yield(S) - cost(S) = profit(S)

所以在图中计算 min cut 可以 最大化 总利润。
构建图需要 O(V+E)
计算 min (s,t)-cut 需要 O(VE)
所以最终 `O(VE)`


# ch12 NP-Hardness

## 12.1 A game you can't win

大意是，一个 逻辑电路 连着一盏灯，
你要回答: 这个灯是否能亮?
但是对方会作弊， 就是 电路实际上 是不亮的，所以 你说 亮，那么对方直接给你看 内部实现。  你说 不亮，那么 在你测试完 所有可能的输入前， 对方可以 修改电路 使得 你还没有测到的 输入 会导致 灯亮。
所以 你唯一获胜的方法就是 测试完 2^n 种case，但是这需要很长时间，你一辈子都没有办法测完。

这种问题 无法通过 brute force 来解决， 或许有一个 更聪明的算法，但是还没有被人发现， 也没有办法证明 这种聪明的算法 不存在。


## 12.2 P vs. NP

算法的一个组成条件就是 有效率的， 即 它的 运行时间的 上界 必须是 输入size的 多项式函数: `O(n^c)`，c是常数。n是输入的size。

研究表明，不是所有的算法 都可以 这么快地 解决。
`NP-hard`问题， 是 大多数人 坚信 不能在 多项式时间 内解决的 问题。 但是没有人 能证明 超多项式的 下界。

`decision problem` 是 输出是 bool (YES or NO)的 问题。
decision problem有3类
- ==P==， 可以多项式时间解决的 decision 问题， 或者说 可以快速解决的问题
- ==NP==， 具有下列属性的 decision问题: 如果答案是YES，那么 可以证明: 可以在多项式时间内 检测出来。 或者说，NP 是 那些 如果告诉我们答案是YES，那么可以在 多项式时间内 证明的 问题。
- ==co-NP==， 是 NP的对立面: 如果问题的答案是NO，那么可以在 多项式时间内证明。

。。是的，baidu，NP: 无法多项式时间内找到答案，但是可以 多项式时间验证答案。

。。书上还有一大堆: P!=NP， NP!=co-NP



## 12.3 NP-hard，NP-easy，NP-complete

一个问题 是 ==NP-hard==， 如果 该问题的 多项式时间的算法 会导致 所有NP问题 可以 在多项式时间内解决。

`problem is NP-hard  <=>  if this problem can be solved in polynomial time, then P=NP`


一个问题是 ==NP-complete==， 如果该问题 既是 NP-hard，又是 NP的。
NP-complete 是 NP问题中 最难的。
一个 NP-complete 问题的 多项式时间算法 表明 对所有NP-complete问题 都有一个 多项式时间算法。

![7652822d60302f2479f2a0552120173b.png](../_resources/7652822d60302f2479f2a0552120173b.png)


1971,1973: The Cook-Levin Theorem: Circuit satisfiability is NP-hard


## 12.4 Formal definitions(HC SVNT DRACONES)

lauguage and Turing machines
。语言和 图灵机

Turing reduction / Cook reduction

many-one reductions / Karp reductions

Karp-NP-hard

logarithmic-space

log-space reduction is a polynomial-time reduction


## 12.5 reductions and SAT

## 12.6 3SAT (from CircuitSAT)

3CNF-SAT 简写: 3SAT

## 12.7 Maximum Independent Set (from 3SAT)

## 12.8 The General Pattern

## 12.9 Clique and Vertex Cover(frmo Independent Set)

clique 是 complete graph 的别名。
complete graph 是 每对点 都有edge 直接相连。

max clique 问题是 在 给定图中找到 最大的 complete 子图。

## 12.10 Graph Coloring (from 3SAT)

proper k-coloring 是指 图中 每个点 选择 k个颜色之一 涂色， 使得 每个边的 2个点 都是不同颜色的。

3Color，给定一个图，是否是 proper 3-coloring

3Color 是 NP-hard


## 12.11 Hamiltonian Cycle

汉密尔顿环 是 图中的一个环，它visit 每个点 一次。

不同于 Euler circuit (欧拉回路)， 欧拉回路 是一个 闭环，它经过每个 边一次。 欧拉回路可以 使用 dfs 在 线性时间内解决。

这里有2个不同的证明，来证明 有向图中的 汉密尔顿环问题 是 NP-hard的

---

From Vertex Cover

---

From 3SAT

---

Variants and Extensions


## 12.12 Subset Sum(from Vertex Cover)

。。证明 ch2 的 subset sum 是 NP-hard


## 12.13 Other Useful NP-hard Problems

- planar circuit sat
- 1 in 3sat
- not all equal 3sat
- planar 3sat
- exact 3 dimensional matching or X3M
- partition, 给定包含 n个整数集合S，是否可以 分为2个集合，使得 它们的sum 相同
- 3partition，3n个整数的集合S， 是否可以分为 n个  3元素的集合，每个集合的 sum 相同
- set cover， 给定 集合的集合 S={S1,S2..Sm}，找到 Si 的最小子集，使得 它们包含 S中所有底层元素
- hitting set
- longest path
- steiner tree
- max2sat
- max cut



- minesweeper， from circuit sat
- sudoku， from 3sat
- tetris   from 3partition
- Klondike   from 3sat
- Pac-Man   from hamiltonian cycle
- Super Mario Brothers  from 3sat
- Candy Crush Saga   from a variant of 3sat
- Threes/2048,   from 3sat
- Trainyard,   from dominating set
- Shortest n * n * n Rubik's cube solution,  from 3sat
- cookie clicker,  from  partition or 3partition


## 12.14 ==choosing the right problem==


- 如果问题 询问 如何 赋值bit 到 object， 或 选择 子集， 或 将object分为2个子集，   那么尝试 SAT or Partition
- 如果问题询问 如何 从一个小的固定set 中 赋值 label到 object， 或 将object 分为 几个 子集，    那么 kColor, 3Color
- 如果问题询问  以某种顺序 将 object 排列，  尝试 directed ham cycle, directed ham path, traverling salesman
- 如果问 符合某些约束的 小的子集，  尝试 min vertex cover
- 如果问 符合某些约束的 大的子集，  尝试 max ind set, max clique, max2sat
- 如果问 将输入 分组为 大量的 小集合，  尝试 3partition
- 如果 问题描述中 出现了 数字3，那么  3sat， 3color, x3m, 3partition
- 最后 尝试 3sat, circuit sat


## 12.15 A frivolous real-world example

## 12.16 On beyonod Zebra

polynomial space

exponential time

excelsior!



# end


