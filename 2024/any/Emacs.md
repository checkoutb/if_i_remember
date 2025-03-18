Emacs

Org-Mode

[[toc]]

---


# tutorial (软件自带)

C - ch: control + ch
M - ch: alt + ch
。。meta 和 edit键基本碰不到了。
。如果没有meta，edit，alt， 那么可以使用 点击+释放ESC 来代替。

## C-x C-c (关闭emacs)

## C-g

## C-x k 退出tutorial

## screen移动

|命令|Desc|
|--|--|
|C-v|下一页/屏幕|
|M-v|上一页/屏幕|
|C-l|光标所在行 变成屏幕的中间，最上，最下|

pageup, pagedown


## cursor control

```text
     C-p
C-b   I   C-f
     C-n
```

P revious line
B ackword
F orward
N ext line

M-f 前进一个单词
M-b 后退一个单词

M 和 C 同时按， M 起效。

C-a 行首 begin
C-e 行尾 end

M-a 上一句/段头
M-e 下一句/段尾


M-< 整个文本的头
M-> 整个文本的尾
。。这个得 shift


## repeat command

```text
C-u 8 C-f

C-u 数字 命令
```

一些命令不能和 C-u 配合
C-v, M-v
    比如 C-u 8 C-v 只能 8行，而不是8个 屏幕。


## 中断命令

C-g 终止现在输入到一半的命令

当Emacs 没有响应的时候，也可以使用这个 C-g

。。估计是 长任务，所以 响应慢， 所以可以中断 执行中的命令



## disabeled command

有些高级命令(影响很大) 默认不启用的。
会出弹框，让你选择是否激活，激活一次，等。


## windows

可以多窗口

C-x 1 回到单窗口，kill 其他的

```text
C-u 0 C-l   光标定位到第一行后，开新窗口，不会导致文本的移动。

C-h k C-f  开新窗口

```



## editor


### insert

换行，默认会带上一行的前缀空格。

C-u 8 *
可以输入8个 *


### delete

#### delete
。。好像只有这2个是 delete，其他都是 kill。
退格键， 删除前面的一个char
C-d， 删除后面的一个char

#### kill
M-退格键， 删除前面的一个 word
M-d, 删除后面的一个 word

C-k, 当前位置 到行末， 全部删除
M-k, 当前位置 到句末， 全部删除
。。类似 C-e, M-e 的 行末，句末。


C-space 移动光标 C-w 删除选中的文本



C-u 2 C-k 删除2行，
2次 C-k 只删除一行。 第一次是kill 到行尾，第二次是kill 换行。 所以只删除了 一行。


kill 和 delete 不同，kill掉的内容 可以通过 C-y，shift-insert 粘贴出来。

连续多次 C-k 的内容，可以被 C-y 一次性全部粘贴出来。

M-y 可以在 所有的 C-y 的内容 中循环。
。。描述不太对，
。。这个功能真的6。



## undo

C-/

C-_

C-x u



## files

find file  
C-x C-f  
。。然后会在最下面出现 输入框。  
。。输入文件名后，如果没有这个文件，会新建。  

C-g 中断上面的 Cx Cf


save file  
C-x C-s



## buffer

使用 Cx Cf 打开一个文件后，原来的文件还在emacs中。  
通过再次 Cx Cf 切换回去。

Emacs 在 buffer 中存储 文件的内容。  

list buffer
C-x C-b

。。C-x 1 退出 buffer list

切换buffer  
C-x b

save buffer  
C-x s  
每个修改过的buffer 都会 问你。


## 扩展命令集

Emacs的命令围绕 X 命令。有2种格式  
- C-x 字符eXtend， 后续输入一个char
- M-x 具名eXtend， 后续输入一个名字


暂时退出Emacs  
C-z  
恢复  
fg 或 %emacs

一些已经学习过的 C-x 命令
|命令|Desc|
|--|--|
|C-x C-f|find file|
|C-x C-s|save file|
|C-x s|save some buffers|
|C-x C-b|list buffers|
|C-x b|switch buffer|
|C-x C-c|quit emacs|
|C-x 1|delete all but one window|
|C-x u|undo|


具名 eXtended 命令 使用频率低，或者 只在某些模式下可用。  
比如 replace-string 命令， 输入 M-x， 然后 repl s, 然后 tab，可以自动补全。 然后输入 2个 字符串。

。。
repl s tab, 回车，不过文档上没有tab，直接回车。确实可以直接回车。  
输入 被修改的字符串 回车  
输入 修改后的字符串 回车  



## auto save

自动保存的文件名是 原文件 前面后面 各加一个 # 。 当你保存时，emacs会删除临时文件。

电脑崩溃后，你可以 `M-x recover-this-file 回车` 来恢复

## echo area
如果emacs发现你输入 多char 的命令 慢，它会展示它们 到你的 平面的底部。  
echo area 包含了 screen 的 最底行。


## mode line
在 echo area 上方的 line 称为 mode line  
类似：
`-:**- TUTORIAL 62% L725 (Fundamental)`

这里包含了 emacs 和正在编辑的文本 的状态  
- 文件名
- 当前位置百分比 (Top == 0%, Bot == 100%， All表示所有内容一个屏幕显示完)
- 当前行号
- 开头的 星号 表示你进行了修改。
- 括号内的部分 是 当前的 editing mode。 默认是 Fundamental

切换成text模式  
M-x text-mode 回车

当前mode的 doc  
C-h m

自动填充模式  
M-x auto-fill-mode 回车。  
。。不过没有效果啊。



## searching

C-s  
输入字符串后，继续 C-s, 会移动到下一个。

C-r  
往前搜

## 多窗口

新建窗口， 不移动焦点  
C-x 2

滚动底部窗口(就是焦点在本窗口，滚动另一个窗口的内容)  
C-M-v

切换窗口  
C-x o

。。C-x 1 关闭其他窗口

C-x 4 C-f  随便输入文件名， 新建窗口，并移动 焦点到新窗口   

C-x 3 是在右半部分新建窗口。  2和4 是在 下半部分


## multiple frames

frame 是 多个窗口的集合。 共用 menu， scroll bar, echo area 等。  
emace的 frame 像其他应用的 window。  

图形界面，多个 frame 可以同时展现。  
文本界面，同时只能有一个。

新建frame  
C-x 5 2  
。。新的应用。

关闭  
C-x 5 0  
。。这个是，在哪个frame 中执行，就关闭哪个


## recirsove editing levels

有时，你会进入 recursive editing level。你可以看到 模式的 外面多了一层 []。   
要退出，只能 连续3次 Esc。 C-g 没用，它是用来中断命令的，无法用来退出。

M-x 进入 minibuffer， 然后3次 Esc退出。


## more help

C-h

单行  
C-h c C-p

新窗口显示  
C-h k C-p

C-h f

C-h a

C-h i


## more feature

手册  
C-h r




---




# emacs 常用插件，网站

2024-05-13 22:18

内容来自：https://zhuanlan.zhihu.com/p/468098391


emacs 网站：
- https://www.emacswiki.org/emacs?interface=zh-cn
- https://planet.emacslife.com/
- http://xahlee.info/emacs/emacs/blog.html
- https://www.masteringemacs.org/


## 管理插件

Emacs有自带的包管理器 package.el ，MELPA是常用的上游软件源，速度慢的话可以用国内的镜像服务器。

use-package 
https://github.com/jwiegley/use-package
4.4k      last year
use-package给出了一种看起来比较有条理的配置管理方案，能涵盖大多数配置。
use-package不一定总是好的，我建议新手尽早建立适合一个==自己的包管理机制==

leaf.el
https://github.com/conao3/leaf.el  
500       10 months ago
是use-package的后继者，它兼容use-package的语法，并且提供了更复杂的、可定制的功能。
emacs-29之后，use-package并入了emacs内建包


## 插件

all-the-icons
1.4k      4 months ago
https://github.com/domtronn/all-the-icons.el

文件的 icon 改成更现代化。



sr-speedbar
https://github.com/emacsorphanage/sr-speedbar
30        8 years ago
speedbar是emacs自带的一个侧边栏，可以用来浏览文件啥的。但是令人感到困扰是，它与emacs居然不出现在一个窗口内
sr-speedbar已经很久没有更新过了，它存在一些需要手动调节的问题，在Emacswiki上有详细的FAQ。

### 1
https://github.com/orgs/emacsorphanage/repositories?type=all   这里有很多 emacs 库




dashboard
https://github.com/emacs-dashboard/emacs-dashboard
1.3k      3 days ago
定制一个看起来相对比较顺眼的启动界面

有一些与之搭配的插件例如dashboard-ls，dashboard-hackernews，允许你在dashboard中加入更多的元素



eshell-toggle
https://github.com/4da/eshell-toggle
35        last month
添加了emacs特性的shell程序



popwin
https://github.com/emacsorphanage/popwin
500       5 days ago
对于panel工作流，这个包提供了一个一般的解决方案。
它实现了一套产生和控制panel的接口，我们只需要把对应的buffer放进popwin制造的panel里就可以了。
你可以把任何buffer融进panel化的工作流



async
https://github.com/jwiegley/emacs-async
800       2 months age
async给emacs提供了elisp层面的异步支持，当然这个异步只是在elisp层面自己做调度而已


avy
https://github.com/abo-abo/avy
1.7k      last year
ace系列的扛把子。灵感来自vim-easymotion，通过按下尽可能少的键移动光标的奇技淫巧。
avy的实现使用了de Bruijn sequence分配键位


### 2
https://github.com/abo-abo?tab=repositories  abo-abo 就是 Oleh Krehel 

avy还有一些衍生品，例如ace-window ace-link还有ace-jump-buffer之类的，在此就不一一介绍了。
它们大都出自abo-abo（Oleh Krehel）之手，此人主导的项目几乎盖了emacs牛比插件集合的半壁江山。

我不推荐使用ace-isearch，avy只能搜索当前buffer的可见区域，swiper对付大文件的时候会卡成一片，最简单的isearch反而是最灵活的。因此我倾向于将这些工具独立而不是捆绑使用。




ace-mc
https://github.com/mm--/ace-mc
51        5 years ago
基于ace-jump和multiple-cursors的多光标编辑功能，它可以快速地在你跳转到的位置插入多个光标。



multiple-cursors
https://github.com/magnars/multiple-cursors.el
2.2k      3 months ago
在多光标编辑上提供了更多的功能，比如按字词选取，多行插入，标记点插入等等。
配合all这个可以搜出正则匹配行的工具，勉强是一个emacs中进行多点编辑的完善方案



iedit
https://github.com/victorhge/iedit
400       2 years ago
在2022年，emacs终于迎来了真正的多点编辑，只要按下`C-;`就可以进入多点编辑模式。
不必为此抛弃multiple-cursors，因为它们可以优势互补。


auctex
https://www.gnu.org/software/auctex/

AUCTeX的存在让emacs一举成为了科研狗的最强写作工具。

AUCTeX可以做到在行内自动预览和折叠公式块，还能给不是公式的东西加一大堆语法高亮。
而且，不要忘了emacs主打的是文本编辑，补全，lint，snippet这些统统都可以开起来。



company
http://company-mode.github.io/
https://github.com/company-mode/company-mode
2.2k      2 weeks ago

Emacs有两大开箱即用的代码自动补全框架，一个是auto-complete，另一个是company。
后者的维护和开发更积极一些，而且视觉效果也更好。

company有一堆后端，如何搭配这些后端是个问题。
老道的用户可能会自己组合搭配出合适的补全后端，
但是，userguide里面也说了，保持原样就可以，company会在合适的点自动帮你选择合适的后端。
但无论如何，多个补全后端是很难相容的。

company-posframe是一个很不错的前端，基于posframe。
company-box是另一个漂亮的前端。
company-quickhelp提供实时展示文档的功能。
https://github.com/tumashu/company-posframe
120       2 years ago
https://github.com/sebastiencs/company-box
560       2 months ago
https://github.com/company-mode/company-quickhelp
370       7 months ago

It comes with several back-ends such as Elisp, Clang, Semantic, Ispell, CMake, BBDB, Yasnippet, Dabbrev, Etags, Gtags, Files, Keywords and a few others.




auto-dim-other-buffers
https://github.com/mina86/auto-dim-other-buffers.el
90        2 years ago
你还在为自己正在编辑哪个frame发愁吗？
这个包可以很好地帮助您找到当前被聚焦的frame，即使它们显示的是同一个buffer。
不过，如果您想在半透明的vte中继续享受您的半透明主题，只dim前景色，不要dim背景色。



dimmer.el
https://github.com/gonewest818/dimmer.el
250       2 years ago
与auto-dim-other-buffers相比，dimmer提供了更多层次的text dim effect。




context-coloring
https://github.com/jacksonrayhamilton/context-coloring
52        2 months ago
绝大多数词法高亮的实现只是正则识别，另一些包诸如color-identifiers-mode，工作方式则是给tag染色，这些在函数式编程中并不太够用，一个很重要的理由是某些变量的绑定具有上下文依赖，这时前两种高亮方法都并不能帮助我们通过高亮识别这些名字。
唯一的解决方案是做点语法分析，考虑到我们看到的范围不大，这并不会消耗太多资源。它与elisp-mode自带高亮的兼容不太好，不过放在js2-mode里用还是很香的。



counsel
https://github.com/abo-abo/swiper
2.3k      2 weeks ago
Ivy，counsel和swiper放在同一个仓库。如果有人问emacs有什么杀手级的应用，那么榜上是一定少不了它们的。
- Ivy, a generic completion mechanism for Emacs.
- Counsel, a collection of Ivy-enhanced versions of common Emacs commands.
- Swiper, an Ivy-enhanced alternative to Isearch.


counsel则是ivy的进一步封装，提供了更多更广义的操作，包括文件搜索和代码补全啥的。



dired

dired-mode是emacs很容易劝退新手的一个东西，至少我是被它劝退过——它甚至都不支持自动同步文件状态！它是emacs内置的文件浏览器

用dired-single和dired-sidebar（不推荐）这两个插件可以把它打造成一个侧边栏文件浏览器......我的评价是，不如sr-speedbar加上几行代码来的简单直接，而且speedbar还有ecb集成。
dired-single这个包还是值得一装的，因为使用dired模式浏览文件目录可能会创建出一堆夹在中间的多余buffer，而这些buffer常常乱成一团

https://github.com/crocket/dired-single
已经跑路了，这人仓库已经没有任何东西了。


dirvish
https://github.com/alexluigit/dirvish
760       last year
相比起dired，绝对是超现实的体验。
dirvish给了emacs一个文本编辑器该有的文件浏览器，同时最大限度地利用了dired已有的模块。
尽管不像treemacs那样对lsp有良好的原始支持，但是以扩展形式加入更多的功能是可能的。
例如，很棒的侧栏（dirvish-side），版本控制信息（dirvish-vc），多媒体即时预览（dirvish-media）等等。
目前dirvish还存在一些bug，但是它正处于积极的开发和维护中。
。。感觉不怎么维护了，issue好多bug.. diy!



flycheck
https://www.flycheck.org/en/latest/
https://github.com/flycheck/flycheck
2.4k      last month

关于lint，emacs内置了flymake，但是从体验上来说，flycheck显然是更友好的

flycheck开箱即用，而且与用户之间有更多的交互。
flymake需要用户自己做更多配置（它甚至在你保存文件之前不能自动更新！）。而且它目前第三方包的数量也不如flycheck多。
顺带一提，就算你不开lsp-mode，只用flycheck搭配lsp-ui的效果还是很香的。




diff-hl
https://github.com/dgutov/diff-hl
880       last week
总之就是非常好看，给每行标上diff颜色的价值在频繁改动的小项目里也许没有那么大，但是这个，搭配上暗色主题简直太漂亮了。
在稍大一点的项目的维护中，可视的diff也是非常实用（其实vscode里早就有这个功能了）。
不过如果是特大的项目，会不会有效率问题？



ein
https://github.com/millejoh/emacs-ipython-notebook
1.5k      9 months ago
不知道是什么原因，搞AI的做实验开始流行使用Jupyer Notebook连接云服务器。
可是，它的前端只有浏览器，如果不加点插件的话未免也太弱了。
如果我要去把浏览器打造成IDE也是很匪夷所思的事情。
难道真的就必须用那些庞大的IDE才行嘛？不，emacs告诉你：我就是浏览器。

自从有了ein，我觉得我可以接受Jupyter了，但是我更想直接用无敌的org-mode，那样才爽翻哩！



eldoc-overlay
https://repo.or.cz/eldoc-overlay.git
          13 months ago
一直以来，在minibuffer里的eldoc信息总让人觉得憋的慌。
特别是在使用mini-modeline的情况下，只有半行的eldoc简直没法看。
怎么才能把eldoc从这个minibuffer里面解救出来呢？
eldoc-box是个不错的方案，它将文档信息显示在一个弹出窗口中，不过这个功能似乎与company-quickhelp有点重复了。
eldoc-overlay则是将信息显示于sideline中，尽管这可能不如box来得灵活，但是看上去很酷，满满的黑客范儿。




esup
https://github.com/jschaf/esup
400       2 years ago
想要提高emacs的启动性能，我们需要一个强大又易用的profiler，esup就是一个这样的插件。



gited
https://github.com/calancha/Gited
18        5 years ago
尽管dired不好用，但是可以浏览git仓库分支的功能还是可以接受滴。配合dired-git-info-mode可以显示文件状态信息，不过后者对付大型仓库容易卡。



highlight-parentheses.el
https://sr.ht/~tsdh/highlight-parentheses.el/
          1 month ago
能力包括但不限于自动匹配并高亮多重括号，包括但不限于lisp程序员的福音。



highlight-blocks
https://github.com/Fanael/highlight-blocks
15        5 years ago
有了这个插件就再也不用担心漏掉层层嵌套的括号了。



magit
https://github.com/magit/magit
6.4k      3 days ago
这个包并没有变成built-in，但是，一名开发者怎么可能离得了这个呢？
magit（magical git interface）是emacs版的git操作界面，你会清晰地感受到比起bash或是widget，emacs在交互性上带来了怎样的变革。



mini-modeline
https://github.com/kiennq/emacs-mini-modeline
200       last year
节约emacs上的modeline占用的空间是个大问题。因为每一个frame都有一个自带的modeline，对于只有12寸屏幕的用户来说，如何利用好多窗口是很让人头疼的问题。

mini-modeline是一个非常modern的设计，缺点是，如果你启用了过多的minor-mode，它保留的长度可能显示不完，一个美观的解决方案是，用符号图标替代默认的mode名称缩写。
dim.el (https://github.com/alezost/dim.el) 就是一个很好的定制方案。

mini-modeline目前与doom-modeline是不相容的，这对颜值党来说有点小小遗憾，尽管这不算是什么难题。




mini-frame
https://github.com/muffinmad/emacs-mini-frame
310       last year
mini-modeline存在的缺陷还有其他更新奇的解决方案。就像之前提到的ivy存在的小问题，把东西挤在minibuffer里不总是方便的，但是创建新的buffer又太混乱。
既然如此，为什么不把minibuffer搬到前台来呢？这就是神奇的mini-frame，它给了我们一个近乎完美的ivy。

ivy-posframe (https://github.com/tumashu/ivy-posframe) 是一个类似的包，不过它只是面向ivy的，目的与mini-frame并不同。
不过，因为它使用的是posframe包，运行反而比mini-frame要稳定，对posframe的客制化也要简单许多。




smartparens
https://github.com/Fuco1/smartparens
1.8k      last month
Emacs中是否有一个实现了像vim-surround那样的功能包呢？
尽管emacs里有evil-mode，但是为了吃桃把桃林子搬到家里来是不太合适的。
遗憾的是，似乎每个包都不是那么让人满意。
Emacs自带的electirc-pairs功能十分有限，像wrap-region这样的包提供的功能又不是那么符合我们的期望。

smartparens的内容相比vim-surround更为充实，它针对不同的使用环境做了特化。如介绍中所描述的，它“试图为所有上述的特性提供一个超集”。



separedit.el
https://github.com/twlz0ne/separedit.el
135       4 month ago
其理念的先进只能用ultimate来形容。
Indirect edit everywhere！这或许将启发下一种编辑模式，我们只是向register中挂载各种模式串，然后进入indirect edit，配合多点编辑，这将大大减少花在写regex上的开支，而且可以处理更复杂的语法。

如果我们的编辑器可以识别括号的层次，何必让程序员去操心括号在哪里呢？
诚然，separedit并不是第一个这样子搞的包，有这种编辑模式的编辑器也不是第一个。
但是站在巨人肩膀上的它为我们展现了一种强大的编辑模式！
而这种编辑模式与emacs本身的特点是如此的契合。
假如能再搭配regex-search和all.el这一类的包，它势必将成为emacs的==神器中的神器==。
尽管目前这个包只是个鸡肋包，但是未来可期。
。。别耍我哦



rainbow-mode
https://elpa.gnu.org/packages/rainbow-mode.html
最近一个版本是 2024-3-31, 再上一个版本是 2020-7-28
前端表示这是全世界最好的minor-mode，而且它甚至还能在命令行终端下使用（只是颜色少一些）。

### 3
https://elpa.gnu.org/packages/index.html
ELPA(Emacs Lisp Package Archive)是 Emacs 的扩展仓库





lsp-ui
https://github.com/emacs-lsp/lsp-ui
1k        5 days ago
让emacs IDE焕发第二春的项目。
与依赖tag的cedet相比，lsp采用的是client-server的设计，连接到通用的language server，而不是emacs专用的后端或者数据库。
不过，如果只是日常编辑而不是开发，用内置的cedet或许来的更简便。

总之，它把你能想到的UI元素都给做了。装了lsp-mode简直是买椟还珠......（顺带一提，lsp-mode里面目前还有很多与其它包兼容性上的bug）

https://github.com/emacs-lsp/lsp-mode
4.7k      3 days ago

emacs29内嵌了eglot，大概是eglot对内建包的亲和性好一些吧。尽管如此，相对lsp而言eglot的功能还是太少了。但是它居然可以使用lsp-ui提供的大部分功能。
eglot
https://github.com/joaotavora/eglot
2.2k      5 months ago
。。这个要的，有 rust， c++ 等


json-navigator
https://github.com/DamienCassou/json-navigator
150       8 months ago
香，实在是香。但是开大文件会卡，不可避。




yami-imenu
https://github.com/knu/yaml-imenu.el
10        2 years ago
不是内建的包，但感觉比内建的json-mode好用很多。yaml-imenu很像swiper，但它按各级字段查找而不是单纯做匹配。

在这里建议尝试 yaml-pro (https://github.com/zkry/yaml-pro ) ，这个包提供了更为齐全的yaml编辑工具集，你可以用它取代原来的yaml-mode包。yaml-pro包含类似yaml-imenu的子项搜索功能，除此之外还有许许多多犀利的功能。




nhexml-mode
http://elpa.gnu.org/packages/nhexl-mode.html

二进制打开文件
这绝对是cracker们会喜欢的一个模式。




rime
https://github.com/DogLooksGood/emacs-rime
462       5 months ago
emacs对 rime 输入法的支持
。应该是台湾的，但是好像 挺好的。 开源的，不会被记录 广告。
https://blog.csdn.net/CoderLvJie/article/details/133886693
这篇吹的挺厉害的： 总之,RIME为程序员量身定制了一个高效、智能、可个性化的输入法解决方案。它的开源自由,配置化能力,和强大扩展性,让输入编程变得飞快而愉悦。
。我的输入法是什么。。不知道，应该是系统自带的，不知道是什么

pyim
https://github.com/tumashu/pyim
880       5 days ago
pyim 是一个 Emacs 中文输入法，支持全拼，双拼，五笔，仓颉 和 Rime 等




vlf
https://elpa.gnu.org/packages/vlf.html
emacs缺乏对大文件的支持，事实上，哪怕这个文件不是很大（比如只有几k），emacs都处理不好超长行的显示问题，而且这个问题至今（2022.10）没有解决……
vlf让我们可以轻松打开一个几十G的文件，而且它还是兼容nhexl-mode的。



yasnippet-snippets
https://github.com/AndreaCrotti/yasnippet-snippets
1.1k      last week
所谓snippet就是写代码时用的模板，比如说，我想写一个lambda语句，那么只要输入lam，然后调用yas-expand（默认是按下`<tab>`键）就可以得到

yasnippet是emacs自带的snippet引擎，这个包是官方收录的一些snippets，开箱即用，适用范围非常广。
尽管常言道，自己定义的snippet最香，但是常人如我，在写代码的时候是很少会分出精力去打磨几份snippet的。
如果我希望实现一些自动化操作，为什么不是录一个宏或者写点elisp语句呢？

auto-yasnippet 是一个很棒的尝试，将snippet的展开与宏结合起来得到可以原地定制的snippet。
https://github.com/abo-abo/auto-yasnippet
243       last year



### 关于 org-mode

org-mode可谓是emacs中的unicorn。对很多人来说 emacs = org-mode editor。
我本人也是重度org用户，用org写代码，画图表，写文档，写计划，写日志，写博客，写论文，写笔记，如果你看到这些功能就能联想起一组又一组对应的包和代码，那么你大概已经会玩了。

org唯一不太好的一点就是用org的人太少了
目前唯一支撑我用org-mode的理由，大概只剩下写proof draft可以偷个懒，以及我不用ms的系统这两条了

仅仅是开发中出现的写作需求的话，markdown+扩展就能通吃

至于org深度中毒的用户，大概用一个16位寄存器就能数得过来




---
---

# 专业 Emacs 入门
https://zhuanlan.zhihu.com/p/385214753
共10章

- 专业 Emacs 入门（一）（本文）
- 专业 Emacs 入门（二）：基础操作
- 专业 Emacs 入门（三）：多文件与模式
- 专业 Emacs 入门（四）：基本配置
- 专业 Emacs 入门（五）：插件篇——功能优化类
- 专业 Emacs 入门（六）：插件篇——功能增强类
- 专业 Emacs 入门（七）：插件篇——编程开发类
- 专业 Emacs 入门（八）：外观主题
- 专业 Emacs 入门（九）：实用小技巧
- 专业 Emacs 入门（十）：笔记系统 org-mode


## emacs 插件大全

https://github.com/emacs-tw/awesome-emacs


## 快捷键pdf

https://www.gnu.org/software/emacs/refcards/index.html


## introduction elisp
https://www.gnu.org/software/emacs/manual/html_node/eintr/



## stackexchange
https://emacs.stackexchange.com/


## emacs wiki
https://www.emacswiki.org/emacs?interface=zh-cn


## emacs manual (not elisp manual)

https://www.gnu.org/software/emacs/manual/html_node/emacs/index.html



对于不想花过多时间配置的读者，可以使用下一小节中提到的 “Emacs 发行版”。

本教程所讨论的 GNU Emacs 是最原始的 Emacs，需要从 0 开始配置。
部分用户会觉得这样过于枯燥，于是有一些 “Emacs 发行版”，预装了很多插件。
这样的 Emacs 有两个：Doom Emacs 和 Spacemacs，它们尤其对 Vim 转到 Emacs 的用户比较友好，因为它们预装了 evil 插件，可以在 Emacs 上使用 Vi 的操作。
但本文只讲解原生 Emacs，也被称为 Vanilla Emacs。



# 专业 Emacs 入门（二）：基础操作
https://zhuanlan.zhihu.com/p/403076883

启动 emacs
`emacs`

非图形界面的 emacs
`emacs -nw`

后面接 文件名，可以直接打开相应文件

在 emacs tutorial 中，输入 `M-x` `help-with-tutorial-spec-language` 可以切换语言(输入 chinese即可)


## 功能键

5个
- Control -> ctrl
- Meta    -> alt / option(mac)
- Shift   -> shift
- Super   -> win / command(mac)
- Hyper   -> 已消失

emacs中快捷键绝大多数都是 control，meta， 其中大部分 只用 control

`C-a` 表示按下Control，按下a，松开 Control和 a

`C-x b` 按下control，按下a，松开 control，a， 按b，松开b

`C-S-<mouse-1>` 按下control 和 shift，鼠标左键点击，松开control，shift

特别的，Meta 有2种按法，如 `M-x`
- 按住Meta，按 x，松开Meta，x
- 按Esc，松开Esc，按x，松开x



Emacs 是通过命令进行交互的。
而所谓命令，就是 Emacs 中使用 Elisp 语言定义的一些函数

即使是最最简单的“将光标上移一行”，也对应着命令 previous-line 。
一切操作都对应一个命令，而快捷键的本质是在调用这些命令。


对 Emacs 输入命令需要先按下 M-x，此时你会看到 Emacs 最下面的空行上出现了 "M-x "，然后等待你的输入，随后你便可以==输入一个函数名==。
`M-x`快捷键可以说是==最重要的一个快捷键==了，只要有它，即使你忘记了其它快捷键，也可以输入函数名进行调用。

命令名的传统是有连字符连接的多个有意义的英文单词。
在输入时==可以用空格代替连字符==。也可以使用 `<tab>` 键自动补全。 

那么此时，读者一定知道前文所说的选择教程语言的命令：`M-x` `help-with-tutorial-spec-language` 应该如何操作了。 


## 基础快捷键

`C-h t` 松开C，h之后再按t， 打开内置教程

`C-x C-c` 退出程序

`C-g` 放弃当前操作 (比如，按了部分快捷键，输入部分命令，被卡住也可以 C-g)

`C-z` 挂起emacs，回到 命令行， 之后 fg 返回emacs。 emacs启动需要几秒钟，所以 频繁退出启动 不方便。 如果安装了 evil 插件，C-z 会被占用，需要 `C-x C-z`


## 移动鼠标

方向键的上下左右
`C-p`,`C-n`,`C-b`,`C-f`

`M-b` 左移一个单词
`M-f` 右移一个单词

`C-a` 光标移到 行首 (不管代码的缩进，所以无法直接编辑)
`C-e` 光标移到 行尾

`M-m` 句首
`M-e` 句尾

`M-<` 文件头，需要shift
`M->` 文件尾

`M-r` 按一次，移到窗口中间行， 2次，窗口最上面一行， 3次，窗口最下面一行

可以玩 Emacs 内的贪吃蛇游戏来锻炼对方向键的熟练度。
按 M-` 调用 tmm-menubar ，按 t 选择 Tools，按 g 选择 Games，按 s 选择 Snake


## 编辑

`DEL`, `backspace` 删除左侧字符
`C-d` 删除右侧字符

`M-d` 剪切右侧 整个词
`M-<DEL>` 剪切左侧 整个词

`M-k` 剪切右侧 直到句子结尾
`C-k` 剪切右侧 直到 行结尾

`C-SPC(空格)` emacs最下面空行显示 Mark set, 移动鼠标，可以看到 半透明选择框。这和 鼠标选择 是一样的。

`M-w` 复制选中区域
`C-w` 剪切选中区域

`C-y`，`M-y` emacs内部维护了一个 环形 "剪贴板历史"，`C-y` 会将最近一次移除的内容插入回来。 继续 `M-y` 会得到 倒数第二次的内容， 再一次 `M-y` 会得到倒数第三次的内容
可以通过插件 counsel 辅助这个过程

`C-/`, `C-_`, `C-x u` 撤销刚刚的操作，如果是字符，则撤销刚才的所有字符操作。

`C-g` + 撤销， redo，就是把上次的撤销 撤销掉。
对于历史记录也维护成了一个环，C-g会改变 环的移动方向

## 标记与跳转

`C-SPC` ，连续2次， 移动光标到其他位置，`C-x C-SPC` 或 `C-u C-SPC` 可以跳会刚刚的位置。

`M-g` 加行号， 跳转到 指定的行


## 重复操作

emacs 可以将 一个命令 重复执行 任意次数。
`C-u`， 数字， 命令快捷键。 `C-u 12 C-n` 表示向下 12行。 不加数字，默认4
这个用法的本质是对本来的 C-n 命令传递了一个数字参数。 
也可以按下 meta 键的同时输入数字，或同时按下 C-M- 输入数字，同样可以实现数字传参。
在图形界面 Emacs 中，也可以只按下 ctrl 同时输入数字。


## 页面移动

`C-v` 向下翻滚一页
`M-v` 向上翻滚一页
`C-l` 第一次，移动页面使得光标 位于窗口中央。 2次，移动页面使得光标 在窗口最上面，3次，移动页面使得光标 在最下面，4次 和 1次一样。

## 搜索文本

`C-s` 输入要搜索的文本，从光标位置向下搜索。光标移动到第一个匹配的位置。
- 再一次`C-s`，跳到下一个匹配位置。 
- `回车`，停留在当前位置，退出搜索
- `C-g`, 放弃搜索，回到 搜索前的位置

`C-r` 从光标位置向前搜索。用法和 `C-s` 一致

## 其他小操作
- `C-t` 交换光标左右字符
- `M-t` 交换光标前后 词
- `C-x C-t` 交换光标所在行 和 上一行。

`C-o` 光标所在行 下方创建一个新行
`C-x C-o` 将光标所在前后 所有连续空行 变成一个空行

`M-l` 光标后 词 变成小写
`M-u` 变成大写
`M-c` 变成首字母大写

`C-x C-=` 放大字号
`C-x C--` 缩小字号
`C-x C-0` 重置字号


## 获得帮助

内置了多种 获取帮助的方式。前缀都是 `C-h`
`C-h c` 输入想查询的快捷键，简要描述
`C-h k` 输入想查询的快捷键，详细描述
`C-h f` 输入函数名
`C-h v` 输入变量名
`C-h a` 输入关键字，列出包含 这一关键字的 命令
`C-h d` 输入关键字，列出包含 这一关键字的符号的 文档

`C-h ?` 列出上述功能，以及其他的帮助功能。


## 上述命令总结

|操作描述|快捷键|命令名|
|--|--|--|
|输入命令|M-x|execute-extended-command|
|退出程序|C-x C-c|save-buffers-kill-terminal|
|放弃当前输入|C-g|keyboard-quit|
|光标向上一行（方向键上）|C-p|previous-line|
|光标向下一行（方向键下）|C-n|next-line|
|光标向左一个字符（方向键左）|C-b|backward-char|
|光标向右一个字符（方向键右）|C-f|forward-char|
|光标向左移动一个词|M-b|backward-word|
|光标向右移动一个词|M-f|forward-word|
|光标移至行首|C-a|move-beginning-of-line|
|光标移至行尾|C-e|move-end-of-line|
|光标移动到一行缩进的开头|M-m|back-to-indentation|
|光标移至句首|M-a|backward-sentence|
|光标移至句尾|M-e|forward-sentence|
|光标移至文件开头|M-<|beginning-of-buffer|
|光标移至文件结尾|M->|end-of-buffer|
|光标移动至窗口的中间、最上、最下|M-r|move-to-window-line-top-bottom|
|删除光标右侧字符|C-d|delete-char|
|移除光标右侧词|M-d|kill-word|
|移除光标左侧词|M-|backward-kill-word|
|移除右侧直到句子结尾|M-k|kill-sentence|
|移除右侧直到行尾|C-k|kill-line|
|设置标记以选择区域|C-SPC|set-mark-command|
|复制区域|M-w|kill-region-save|
|移除区域|C-w|kill-region|
|插入已移除文本|C-y|yank|
|插入历史移除文本|M-y|yank-pop|
|撤回|C-/ 或 C-_ 或 C-x u|undo|
|跳转到上一标记|C-x C-SPC 或 C-u C-SPC|pop-global-mark|
|跳转到行号|M-g M-g|goto-line|
|重复|C-u|universal-argument|
|向下一页|C-v|scroll-up-command|
|向上一页|M-v|scroll-down-command|
|移动页面使得光标在中央/最上方/最下方|C-l|recenter-top-bottom|
|向后搜索|C-s|isearch-forward|
|向前搜索|C-r|isearch-backward|
|交换前后字符|C-t|transpose-chars|
|交换前后词|M-t|transpose-words|
|交换前后两行|C-x C-t|transpose-lines|
|在下方新建一行|C-o|open-line|
|删除连续空行为一个空行|C-x C-o|delete-blank-lines|
|将后面的词变为小写|M-l|downcase-word|
|将后面的词变为大写|M-u|upcase-word|
|将后面的词变为首字母大写|M-c|capitalize-word|
|放大字号|C-x C-=|text-scale-adjust|
|缩小字号|C-x C--|text-scale-adjust|
|重置字号|C-x C-0|text-scale-adjust|
|简要描述快捷键功能|C-h c|describe-key-briefly|
|描述快捷键功能|C-h k|describe-key|
|描述函数功能|C-h f|describe-function|
|描述变量|C-h v|describe-variable|
|列出含某一关键词的命令|C-h a|apropos-command|
|列出含某一关键词的符号的文档|C-h d|apropos-documentation|
|帮助的帮助|C-h ?|help-for-help|



# 专业 Emacs 入门（三）：多文件与模式

https://zhuanlan.zhihu.com/p/409364725

本篇介绍 Emacs 的界面术语、如何管理多个文件，如何分割显示等


## 界面术语

- Frame
  如果使用图形界面打开 Emacs，那么整个程序窗口 就是 Frame，如果打开了多个窗口，就有 多个 Frame,
  如果使用终端打开 emacs，那么 emacs 所占据的整个终端的界面 被称为 Frame
- Menu bar
  菜单栏，在 Frame的最上方。
  默认包括 File，Edit 等。在终端中不能用鼠标时，需要用 `menu-bar-open` 命令打开，对应的快捷键是 `f10`。 还有一个 M-` ，对应命令 tmm-menu, 可以从下方展开互动界面打开菜单。
- Tool bar
  工具栏，只有 图形界面时 可用。
  就是那些工具图标，由于 功能很基本，所以 笔者 会关闭工具栏(在配置文件中加入 `tool-bar-mode -1`).
- Echo Area
  界面下方的 一行 就是 "回显区"，用来打印各种简短的消息。
- Window
  tool bar 之下，echo area 之上的 区域。
  这个和平常使用电脑时的窗口 不同， 平常的窗口 在emacs中是 frame。
- Mode line
  window最下方的 灰色行。
  显示当前 buffer 的一些信息， 大概包括了 文件编码，是否修改，当前 buffer 名，光标所在位置 占全文百分比，行号， 内容可以自定义
- Scroll bar
  window的最右侧的滚动条
  在emacs中 根本不需要滚动条，所以 可以关闭。
- Cursor
  光标， 可以换成 小竖线
- Point
  光标所在的位置。 区别是，光标只有一个，但是 Point是 针对buffer的，每个 buffer 都有一个 point。

## 文件与buffer

之前是在命令行中使用 `emacs <filename>` 来打开文件。

`C-x C-f` 在echo area 会出现 "Find file: ", 输入文件路径(可以 tab 补全)，就可以 在 emacs中打开一个文件。 如果 要新建文件，输入一个 不存在的文件名即可以。 插件 ivy, helm 可以辅助这一过程

`C-x C-s` 保存文件
`C-x `

当你使用 C-x C-f 打开第二个文件时，第一个文件会"消失"。
事实上，所有打开的文件都会被放入一个被称为 Buffer 的对象中，
当打开了第二个文件时，第一个文件所在的 Buffer 会切入后台，而第二个文件的 Buffer 会占据当前的 Window。
Buffer 的名字显示在 Mode line 中间，通常是文件名本身。

Emacs 也可以用这个方式==打开目录==（文件夹），会显示出目录内的文件（此即 Linux 的设计理念，一切皆为文件，即使是目录也本质上是一个文件），可以用光标选择想打开的文件。


### buffer的切换

3种方法

`C-x b` 输入buffer的名字，然后回车 即可。 如果不输入直接回车，会跳转到 默认的buffer中。
。。不是默认，是 和最近的 那个交换。 所以一直执行 的话，就是 2个 buffer 互换。

`C-x C-b` 会弹出一个 window，名为 buffer list， 列出所有当前打开的 buffer。 
其中，以 星号 开头结尾的 buffer 是 emacs 的。
开头如果是 %，表明这个 buffer 被修改过 且 没有保存。
如果光标不在 buffer list中，可以用 `C-x o` 切换过去。
。。这个怎么退出。就是新开了一个 窗口，怎么关闭这个窗口。。下面的 C-x 1

通过光标选择buffer。 在 buffer list 中，还可以：
- `?` 显示帮助
- `q` 退出
- `d` 标记一个buffer 打算关闭
- `s` 标记一个buffer 打算保存
- `u` 取消标记
- `x` 执行刚刚标记过的 删除 和 保存操作。
这里上移下移，不需要 C，直接 p 和 n 即可

`C-<mouse-1>` (左键) 用鼠标菜单 切换buffer


### 文件备份

使用emacs打开文件后，会发现 目录下 多一个 文件，文件的名字是 打开的文件名 + ~ 。
如 打开 names.txt 会增加一个 names.txt~
这是 emacs 的备份机制。 可以关闭 或设置 文件数量上线。

。。我的没有(至少没有自动启用这个功能)。

## 多window

- `C-x 2` 上下分割出2个 window
- `C-x 3` 左右分割出2个window
- `C-x 0` 关闭光标所在的 window
- `C-x 1` 只保留光标所在的window，关闭其他的
- `C-x o` 将光标切到下一个window

分割后，默认会把当前的 Buffer 也显示到新的 Window，即显示了两个一样的 Window。
再次强调一下，Buffer 对应真正打开的文件，而 Window 是把 Buffer 显示出来的元件，所以一个文件只会开一个 Buffer，但可以有多个 Window 显示。
于是，在新的 Window 里用 `C-x C-f` 打开另一个文件即可看到两个文件了，当然也可以正常用上面所说的 Buffer 切换。

开一个新的窗口并打开新的文件这个需求很常见
如果只有以上快捷键，需要先 `C-x 3` 分割出一个窗口，`C-x o` 切换到新窗口，`C-x C-f` 打开新文件，过于繁琐
所以 Emacs 提供了一个==快捷键==：
`C-x 4 f` 来达到“在另一个窗口打开新的文件，如果只有一个窗口就分割成两个”的效果。
。。那如果已经有2个窗口了，是不会打开新窗口的？

`C-x 4 b` 表示在另一个窗口切换到另一 Buffer，如果只有一个窗口就分割成两个

`C-x 4 d` 表示 在另一个窗口打开目录，如果只有一个窗口就分割成两个

可以总结出 C-x 4 为前缀时，就表达“在另一个窗口做……“。

`C-M-v` 2窗口时，如果光标在第一个窗口，而希望第二个窗口向下翻页，就可以使用这个。
`C-M-S-v` 第二个窗口向上翻页


## 多Frame

`C-x 5 2` 打开新 frame
`C-x 5 f` 在新的frame 打开文件

C-x 5 和 C-x 4 类似，前者在 frame间操作， 后者在 window间操作。


## 模式(mode)

Emacs 的核心要素之一就是模式（mode）。一个模式就对应着一组环境，不同模式可以分别进行配置，应对不同的场景。

emacs mode 分为2类，主模式(major mode)，次模式(minor mode)

### 主模式

主模式默认根据 Buffer 的文件类型来选择，一个 Buffer 只能对应一个主模式。
例如，Emacs 发现你打开了 .cpp 为后缀的文件，就会把 Buffer 自动设置成 `c++-mode`

最直观的区别是 Emacs 为不同语言的源码提供了不同的语法高亮。 主模式的名字会显示在 Mode line 上。

我们也可以手动切换主模式，只需要按下 `M-x` ，输入相应的模式名称即可。通常来说其实我们不需要手动设置。

最基本的主模式是 Fundamental mode，就是没有进行任何配置的模式 。

### 次模式

同一个 Buffer 可以有多个次模式，次模式可以进一步调整、增加一些配置。
通常来说，插件都是靠次模式来起作用的。
当我们安装插件时，插件的官网会提示如何设置这个插件，其中大多都会使用次模式。

https://www.gnu.org/software/emacs/manual/html_node/emacs/Minor-Modes.html


### mode hook

每一个主模式都对应着一个 Mode hook，hook 是挂钩的意思，Mode hook 的作用就是当启动一个主模式时，==自动执行==一些已经“挂钩”到这个主模式的==函数或次模式==。
由此，我们可以自由地向一个主模式上挂上各种功能，在启动这个主模式时就可以自动跟随着一起启动。

Mode hook 的名字通常就是“主模式名-hook”

例如，我们希望在主模式“文本文件模式” text-mode 时启动次模式“检查拼写” flyspell-mode ，我们就可以这样写配置：
`(add-hook 'text-mode-hook 'flyspell-mode)`



## 目录（文件夹）操作, Dired

Dired，即 Directory Editor，是 Emacs 自带的用以处理目录和文件的功能

常见的操作例如删除文件、将文件从一处拷贝至另一处，更高级的操作如对比两个文件的异同、更改权限、链接文件等等，都可以通过 Dired 实现。

`C-x C-f` 输入 文件夹，就进入 Dired。
`C-x d` 或 `M-x dired` 是更标准的进入 dired 的方式

`C-x C-j` 已打开文件时，打开文件所在的目录

`h` 帮助

Dired 的基本操作是
- 通过光标 上下移动 (不需要C，直接 p 和 n)
- 按下 一个命令快捷键 来对该文件 调用命令。 
- 需要批量操作，只需要按 m 就可以选择， 按 u 取消选择。 
- 批量删除时，按 d 标记删除， 按 x 执行删除

例子，我们想要将 a.txt 和 b.txt 文件挪到 subdir 中，
- 首先我们可以对 subdir 按下 i 来展开这个子目录， 
- 随后对两个文本文件按下 m 标记， 
- 然后按下 R（Rename） ，
- 在回显区输入 ~/Code/Emacs/Test/subdir/，
- 按下回车。

移动文件的本质就是重命名（Rename），所以 Dired 里没有所谓的”移动“这个操作，而只有重命名。



|操作描述|快捷键|命令名|
|--|--|--|
|下拉菜单栏|`<f10>`|menu-bar-open|
|互动菜单栏|M-`|tmm-menubar|
|打开文件|C-x C-f|find-file|
|保存文件|C-x C-s|save-buffer|
|打开并只读文件|C-x C-r|find-file-read-only|
|打开另一相近文件|C-x C-v|find-alternate-file|
|只读模式|C-x C-q|read-only-mode|
|切换到 Buffer|C-x b|switch-to-buffer|
|列出 Buffer|C-x C-b|list-buffers|
|关闭 Buffer|C-x k|kill-buffer|
|鼠标列出 Buffer|C-mouse-1|mouse-buffer-menu|
|上下分割出 Window|C-x 2|split-window-below|
|左右分割出 Window|C-x 3|split-window-right|
|关闭当前 Window|C-x 0|delete-window|
|只保留当前 Window|C-x 1|delete-other-windows|
|切换到另一 Window|C-x o|other-window|
|在另一 Window 中打开文件|C-x 4 f|find-file-other-window|
|在另一 Window 中切换 Buffer|C-x 4 b|switch-to-buffer-other-window|
|在另一 Window 中打开目录|C-x 4 d|dired-other-window|
|创建新的 Frame|C-x 5 2|make-frame-command|
|在另一 Frame 中打开文件|C-x 5 f|find-file-other-frame|
|让另一 Window 向下翻页|C-M-v|scroll-other-window|
|让另一 Window 向上翻页|C-M-S-v|scroll-other-window-down|
|打开 Dired|C-x d|dired|
|打开文件所在目录|C-x C-j|dired-jump|



# 专业 Emacs 入门（四）：基本配置

https://zhuanlan.zhihu.com/p/432552171

本文主要内容归纳：
- 简单的 Emacs Lisp 语言知识——让你在配置时游刃有余
- 配置文件的知识——模块化的写法
- 一些观点——最好不要完全使用并依赖大牛的配置，把配置掌握在自己手中
- 一些基础配置——开启部分 Emacs 功能
- 介绍插件的安装并用 use-package 管理插件——管理成本低、逻辑更清晰
- 快捷键、变量的设置——迈出你的自定义脚步


## Emacs Lisp

### 语法简介

```lisp
(defun ivy-set-prompt (caller prompt-fn)
  (setq ivy--prompts-list
        (plist-put ivy--prompts-list caller prompt-fn)))
```

Lisp 就是 “List Processing“ 的缩写

Lisp 语言的核心就是列表（List）。
在 Lisp 中，每一对小括号表达了一个列表，列表元素用空格分隔

在执行 Lisp 时，会把列表的==第一个元素作为函数名==，后面的元素都是==函数的参数==。
元素可以是一个“词”，也可以是另一个列表。
可以类比 Shell 命令的写法，也是第一个词是命令名，而后面的是命令的参数。

2+3+4 在 Lisp 中写为：
`(+ 2 3 4)`

4+(3-2) 写为：
`(+ 4 (- 3 2))`

定义函数就用 `defun` 关键字， 
设置变量的值用 `setq` 关键字。

上面展示的那段 Emacs Lisp 代码可以约等于如下 C/C++ 语言代码
```C
void ivy_set_prompt(CallerType caller, FnType prompt_fn) {
    ivy__prompts_list = plist_put(ivy__prompts_list, caller, prompt_fn);
}
```

Lisp 的变量名可以包含许多字符，所以如果看到了一些奇奇怪怪的名字，不要感到惊讶，就是变量名。

一些常见关键字如 
- let 为一组变量圈出一个作用域、
- if/when/unless 表示条件语句。 
- t 表示 true，
- nil 表示空（相当于 C 语言的 NULL、Python 的 None）。 
- 单引号表达了后面的元素不进行执行而直接返回它本身：
  `'(Tom Amy John)`
  当括号前面有一个单引号时，表达了一个包含了三个元素的“数组”，而不是在执行一个叫 Tom 的函数
  `'ivy-set-prompt`
  表示把 ivy-set-prompt 这个函数作为一个对象传递给其它部分，也没有执行这个函数。 
  ==反引号（`）==backquote（或称 backtick、grave accent）在 Lisp 中也有含义，和单引号类似

Emacs Lisp 源码文件的后缀名是 .el

分号（;）以后的内容都是注释。

由于 Lisp 的整个语言结构就是列表的嵌套，所以它设定了一个非常强大的宏系统，可以用代码生成代码，甚至定义出一个与之前不太一样的新语言，常被称为方言

## Scratch buffer
Emacs 总会有一个默认打开的 Buffer 叫 `*scratch*`，它是一个用来写一些临时“草稿”代码的

例如想测试一下一段 Emacs Lisp 运行的结果，就可以先在 `*scratch*` 里写一下，然后调用 `M-x eval-buffer` 等命令


## 配置文件

配置文件是一个包含了 Emacs Lisp 源码的文件，描述了 Emacs 应当以什么样的方式启动。
在 Emacs 启动的时候会执行其中的代码，可以理解为启动时刻运行的脚本。

当启动 Emacs 时，Emacs 会自动依次寻找以下几个文件之一作为配置文件（一旦找到了其中之一，就不会继续寻找顺序靠后的其它配置文件了）：
- ~/.emacs
- ~/.emacs.el
- ~/.emacs.d/init.el
- ~/.config/emacs/init.el

。。我本地 4个地方都没有

自定义config位置， https://www.gnu.org/software/emacs/manual/html_node/emacs/Find-Init.html

随着我们需要的功能越来越复杂，配置源码会越来越长，我们会希望能够分多个源文件进行不同功能的管理。
所以使用 `~/.emacs.d/init.el` 作为配置文件是最为常见的。 我们可以将其它各种源文件都放置在 `~/.emacs.d` 目录下，方便管理。 


## 基础设置

笔者最开始借鉴的就是 Steve Purcell 的配置。Steve Purcell 是 MELPA 的维护者
https://github.com/purcell/emacs.d
6.8k      3 weeks ago
读者可以 clone 下来后把文件夹命名为 ~/.emacs.d


基本结构
使用 Emacs 打开 ~/.emacs.d/init.el 文件

根据 Emacs Lisp 的规范，所有的源码文件的开头最好写好 docstring

按照习惯，三个分号开头的注释表示“节”，两个分号开头的注释表示“段落”
`;;; Code:` 后面就开始 Emacs Lisp 的代码了。同时，文件的结尾要是：`;;; init.el ends here`

Steve Purcell 的配置的前 34 行几乎可以照抄，除了其中一行 (require 'init-benchmarking)中使用了他定义在 ~/.emacs.d/lisp/init_benchmarking.el 中的逻辑来测量启动时间，读者酌情加入 
。。现在 19行

涉及一些基本的启动要素，例如检查版本、设定源码加载路径、通过修改垃圾回收的内存上限来提高 Emacs 启动速度等等
“==设定源码加载路径” 这句代码是指将 ~/.emacs.d/lisp/ 目录==作为==源码加载路径==，这样你可以将功能需求拆分成多个文件放置在这个目录中，供 init.el 使用。 
。。就是 `(add-to-list 'load-path (expand-file-name "lisp" user-emacs-directory))`  这句

`(require 'xxx)`，这个可以理解为“导入并执行”，基本类似于 Python 的 import。
也就是导入刚刚说的放置在了 ~/.emacs.d/lisp/ 目录下的某个源码文件，并运行了其中的代码使得内部的设置和函数定义生效。

`(interactive)` 这句代码意为“让这个函数可以通过 M-x 手动调用，否则按下 M-x 时会发现找不到 hello-world 这个命令。 
没有 (interactive) 的函数就是指不对用户直接暴露的函数，是用于内部调用的。


### 最开始的配置

对于一个刚打开的“白板”编辑器来说，有不少功能是我们亟需开启的，在此做简要归纳：
```lisp
(setq confirm-kill-emacs #'yes-or-no-p)      ; 在关闭 Emacs 前询问是否确认关闭，防止误触
(electric-pair-mode t)                       ; 自动补全括号
(add-hook 'prog-mode-hook #'show-paren-mode) ; 编程模式下，光标在括号上时高亮另一个括号
(column-number-mode t)                       ; 在 Mode line 上显示列号
(global-auto-revert-mode t)                  ; 当另一程序修改了文件时，让 Emacs 及时刷新 Buffer
(delete-selection-mode t)                    ; 选中文本后输入文本会替换文本（更符合我们习惯了的其它编辑器的逻辑）
(setq inhibit-startup-message t)             ; 关闭启动 Emacs 时的欢迎界面
(setq make-backup-files nil)                 ; 关闭文件自动备份
(add-hook 'prog-mode-hook #'hs-minor-mode)   ; 编程模式下，可以折叠代码块
(global-display-line-numbers-mode 1)         ; 在 Window 显示行号
(tool-bar-mode -1)                           ; （熟练后可选）关闭 Tool bar
(when (display-graphic-p) (toggle-scroll-bar -1)) ; 图形界面时关闭滚动条

(savehist-mode 1)                            ; （可选）打开 Buffer 历史记录保存
(setq display-line-numbers-type 'relative)   ; （可选）显示相对行号
(add-to-list 'default-frame-alist '(width . 90))  ; （可选）设定启动图形界面时的初始 Frame 宽度（字符数）
(add-to-list 'default-frame-alist '(height . 55)) ; （可选）设定启动图形界面时的初始 Frame 高度（字符数）
```




### 配置快捷键

首先介绍一下如何配置全局的快捷键：

`(global-set-key (kbd <KEY>) <FUNCTION>)`

其中 `<KEY>` 和 `<FUNCTION>` 替换为你想要设置的快捷键和功能。
例如一个常见设置是修改回车键为“新起一行并做缩进”：
`(global-set-key (kbd "RET") 'newline-and-indent)`

其它设置示例：

```lisp
(global-set-key (kbd "M-w") 'kill-region)              ; 交换 M-w 和 C-w，M-w 为剪切
(global-set-key (kbd "C-w") 'kill-ring-save)           ; 交换 M-w 和 C-w，C-w 为复制
(global-set-key (kbd "C-a") 'back-to-indentation)      ; 交换 C-a 和 M-m，C-a 为到缩进后的行首
(global-set-key (kbd "M-m") 'move-beginning-of-line)   ; 交换 C-a 和 M-m，M-m 为到真正的行首
(global-set-key (kbd "C-c '") 'comment-or-uncomment-region) ; 为选中的代码加注释/去注释


;; 自定义两个函数
;; Faster move cursor
(defun next-ten-lines()
  "Move cursor to next 10 lines."
  (interactive)
  (next-line 10))

(defun previous-ten-lines()
  "Move cursor to previous 10 lines."
  (interactive)
  (previous-line 10))
;; 绑定到快捷键
(global-set-key (kbd "M-n") 'next-ten-lines)            ; 光标向下移动 10 行
(global-set-key (kbd "M-p") 'previous-ten-lines)        ; 光标向上移动 10 行
```



### MELPA

Emacs 的插件都被放在了一些固定的仓库网站上

Emacs 最大的插件仓库就是 MELPA 了，也就是上文提到的 Steve Purcell 所维护的项目。
此外也有一个默认仓库 GNU ELPA。

MELPA 的官网有直接介绍如何配置
```lisp
(require 'package)
(add-to-list 'package-archives '("melpa" . "https://melpa.org/packages/") t)
(package-initialize)
```

国内网络,有如下两个方案。

使用代理
```lisp
(setq gnutls-algorithm-priority "NORMAL:-VERS-TLS1.3") ; 不加这一句可能有问题，建议读者尝试一下
(setq url-proxy-services '(("no_proxy" . "^\\(192\\.168\\..*\\)")
                           ("http" . "<代理 IP>:<代理端口号>")
			   ("https" . "<代理 IP>:<代理端口号>")))
```

使用国内镜像
腾讯
```lisp
(require 'package)
(setq package-archives '(("gnu"   . "http://mirrors.cloud.tencent.com/elpa/gnu/")
                         ("melpa" . "http://mirrors.cloud.tencent.com/elpa/melpa/")))
(package-initialize)
```

。。腾讯的不是最新的， 要用清华的。 比如
- ssh-deploy, 腾讯是 3.1.13, 清华是 3.1.16
- js2-mode, 腾讯是 20210906, 清华是 20231224

腾讯的应该是 在2021年打了一个包，后续没有更新。

清华
https://mirrors.tuna.tsinghua.edu.cn/help/elpa/


随后重启 Emacs 后，输入命令 `package-list-packages` 就可以==列出来仓库中的所有插件==，可以选中相应的插件，会弹出介绍的界面和安装按钮。
此外，还可以直接通过命令 `package-install` ，按下回车后，输入插件名就可以安装相应插件。

默认情况下，插件会被安装到 ~/.emacs.d/elpa/ 目录下。

想要删除已安装的插件，输入命令 `package-delete` ，然后输入已安装的插件名即可。


### 插件设置(use-package)

不同插件的使用、配置常常不同，一一配置会使得配置文件很乱，且不易管理，并且缺少一些自动化的配置机制

Steve Purcell 的配置中，他在 init-elpa.el 中定义了一些辅助函数 require-package 等实现了插件的自动安装。 

笔者则使用了一个更为方便的插件 use-package 来进行管理。
Emacs 29.1 已经集成了 use-package

Emacs 28 及以下版本，需要先安装 use-package。
输入命令 package-install 按下回车后输入 "use-package"，回车。
安装后在 init.el 较靠前的位置（或其它你认为合适的文件中）写上：
```lisp
(eval-when-compile
  (require 'use-package))
```

这样，我们就在启动 Emacs 的时候首先加载 use-package 插件。随后我们再使用 use-package 插件来管理所有其它插件。

use-package 官网提供了一些教程，其使用方法很简单，假设我们希望使用一个叫 foo 的插件：
```lisp
(use-package foo
  :init                  ; 在加载插件前执行一些命令
  (setq foo-variable t)
  :config                ; 在加载插件后执行一些命令
  (foo-mode 1))
```
冒号开头的词是 use-package 的一些设置关键词。

也可以轻松地设定上一篇教程中提到的模式的 hook。例如，我们希望在编程模式 prog-mode 下使用代码语法检查工具 flycheck，只需要使用 :hook 进行设置： 
```lisp
(use-package flycheck
  :ensure t
  :hook                        ; 为模式设置 hook
  (prog-mode . flycheck-mode))
```

### 配置变量

除了使用配置文件，Emacs 还提供了一个更为方便的办法管理一些变量（customizable variables），或称用户选项（user options）。

`M-x` 输入 customize，回车确认

变量是分组（group）管理的，只需要点进去寻找或搜索相关的变量就可以进行设置。
对于每一个变量，点左侧的箭头展开内容，可以看到有的变量是 Toggle 按钮表示可以设定 true/false，有的则是取值列表，可以设定值。
修改后，State 会显示已编辑。
最后点击上方的 Apply 就是应用更改。
点击 Revert 就可以放弃更改等。
按 q 退出。 

当设置了变量后，事实上 Emacs 会自动将一些配置代码加入到 init.el 中，或是加入到自定义的文件中（比如 Steve Purcell 就自定义了这个文件）。 

使用 Emacs 的过程中也可以==临时修改某个变量的值==，`M-x set-variable` 就可以输入变量名、回车、输入值、回车。还可以用 `C-h v` 输入变量名来==查看变量==的含义。


最简单让配置生效的办法就是重启 Emacs。

事实上 Emacs Lisp 语言是逐句执行的。
所以例如我们新加入了一段配置，我们便可以直接选中这部分代码，然后按下 M-x eval-region ，表达了“运行选中的这部分代码”的含义，这样这段代码立刻就会生效了。
。。从后面看来就是， 直接 执行 新加的配置， 下面的 ivy 中有安装的详细步骤

当然，还有 M-x eval-buffer 可以直接重新执行一下当前 Buffer 的所有代码

最后，配置文件的结尾要有一句：
```lisp
(provide 'init)

;;; init.el ends here
```



# 专业 Emacs 入门（五）：插件篇——功能优化类

https://zhuanlan.zhihu.com/p/441612281

大体分为如下几类：
- 功能优化类：对 Emacs 自身的一些不够完美的功能进行替换，解决一些痛点，提高操作便利性
- 功能增强类：大大提升 Emacs 体验与效率
- 编程类：和编程相关的插件配置
- 外观类：配置颜色、主题、屏保等。
- 有趣类：实用性不高但很有趣的插件


例如马上要介绍的插件 ivy，和它相关的命令就都是 ivy-* 这样的命名。
所以读者在安装 ivy 后可以首先 M-x ivy- 然后就可以看到==一系列以此为前缀的命令==，
想要查询其中==某一个的功能==例如 ivy-push-view，那就先 C-g 回到正常 Buffer 内，然后输入我们第二篇教程就介绍的命令 `C-h f ivy-push-view` 就可以显示这个命令的介绍了。 

## ivy
https://github.com/abo-abo/swiper

有个评论： 官网说了安装consel才会把其他三个一起安装，只安装ivy按照前面的配置会找不到consel相关函数


emacs功能是全的，但是使用的友好度差了点。ivy 就为 Emacs 带来了使用体验上的巨大提升

打开 ivy 的链接，会发现其实这个仓库名为 swiper，里面包含了 ivy、counsel 和 swiper 三部分。
它们三个分别加强了 Emacs 的三个方面：补全系统、部分常用命令、搜索功能。


1. 那么只要我们的配置文件中写好了相应的仓库地址，就可以通过输入 `M-x package-install <RET> ivy <RET>` 来安装插件了

2. 更方便的方式来管理插件—— use-packag，可以直接在配置文件（例如 ~/.emacs.d/init.el）中写上：
     ```lisp
     (use-package ivy
     :ensure t)
     ```
  重启，或直接执行这段代码：
  - 选中这段代码，
  - `M-x eval-region` 或 光标在 括号内时 按下 `C-M-x` (eval-defun 执行函数) 。 use-package 就自动帮你安装好 ivy 了


主页上提供了完整的配置文档，同时也提供了一个简单的配置
有的插件会提供 use-package 的写法，而有的不会

```lisp
;; ---- 执行了一个函数启动 ivy mode ----
(ivy-mode)
;; ---- 设置两个变量为 True，还有一个可选的 ---
(setq ivy-use-virtual-buffers t)
(setq enable-recursive-minibuffers t)
;; enable this if you want `swiper' to use it
;; (setq search-default-mode #'char-fold-to-regexp)
;; ---- 绑定快捷键 ----
(global-set-key "C-s" 'swiper)
(global-set-key (kbd "C-c C-r") 'ivy-resume)
(global-set-key (kbd "<f6>") 'ivy-resume)
(global-set-key (kbd "M-x") 'counsel-M-x)
(global-set-key (kbd "C-x C-f") 'counsel-find-file)
(global-set-key (kbd "<f1> f") 'counsel-describe-function)
(global-set-key (kbd "<f1> v") 'counsel-describe-variable)
(global-set-key (kbd "<f1> o") 'counsel-describe-symbol)
(global-set-key (kbd "<f1> l") 'counsel-find-library)
(global-set-key (kbd "<f2> i") 'counsel-info-lookup-symbol)
(global-set-key (kbd "<f2> u") 'counsel-unicode-char)
(global-set-key (kbd "C-c g") 'counsel-git)
(global-set-key (kbd "C-c j") 'counsel-git-grep)
(global-set-key (kbd "C-c k") 'counsel-ag)
(global-set-key (kbd "C-x l") 'counsel-locate)
(global-set-key (kbd "C-S-o") 'counsel-rhythmbox)
(define-key minibuffer-local-map (kbd "C-r") 'counsel-minibuffer-history)
```

use-package 中 
- :init 标签表示加载插件前执行的逻辑（如，启动 ivy-mode），
- :config 表示加载后执行的逻辑（如，设置一些变量，这显然是加载插件后的逻辑，因此写在 :config 后面）。
至于绑定快捷键，只需要照抄一下。最后一行的配置涉及到了 keymap，keymap 就是指在某一模式下的键盘输入事件的集合

```lisp
(use-package counsel
  :ensure t)

(use-package ivy
  :ensure t
  :init
  (ivy-mode 1)
  (counsel-mode 1)
  :config
  (setq ivy-use-virtual-buffers t)
  (setq search-default-mode #'char-fold-to-regexp)
  (setq ivy-count-format "(%d/%d) ")
  :bind
  (("C-s" . 'swiper)
   ("C-x b" . 'ivy-switch-buffer)
   ("C-c v" . 'ivy-push-view)
   ("C-c s" . 'ivy-switch-view)
   ("C-c V" . 'ivy-pop-view)
   ("C-x C-@" . 'counsel-mark-ring); 在某些终端上 C-x C-SPC 会被映射为 C-x C-@，比如在 macOS 上，所以要手动设置
   ("C-x C-SPC" . 'counsel-mark-ring)
   :map minibuffer-local-map
   ("C-r" . counsel-minibuffer-history)))
```

直接使用 use-package 来安装 ivy、counsel 和 swiper 可能存在一些依赖问题，笔者暂未解决。对此可以手动安装，例如 `M-x package-install counsel`。


### 同类插件

和 ivy 功能基本类似的插件有 helm 和 ido ， 读者可以自己选择自己更为喜爱的。

Helm is an Emacs framework for incremental completions and narrowing selections.

https://github.com/emacs-helm/helm


## amx
https://github.com/DarwinAwardWinner/amx

记录我们每次调用 M-x 时输入的命令历史，然后每次将最常用的显示在前面

```lisp
(use-package amx
  :ensure t
  :init (amx-mode))
```


## ace-window
https://github.com/abo-abo/ace-window

使用 Emacs 多窗口时，window 超过 3 个后就很难使用 C-x o 进行切换了。
ace-window 对 C-x o 重新绑定，使用时可以为每个 window 编个号，用编号进行跳转。

```lisp
(use-package ace-window
  :ensure t
  :bind (("C-x o" . 'ace-window)))
```


## mwin
https://github.com/alezost/mwim.el

按一次 C-a 时移动到代码文字开头，再按一次则是移动到整行的行首，如此反复。

当本行代码结尾有注释时，第一次按 C-e 将光标移动到代码尾部、注释之前。再按一次则是移动到整行的行尾。


## undo-tree
https://www.emacswiki.org/emacs/UndoTree

所有的编辑都会成为 undo-tree 上的一个节点。当按下 C-x u 时就会显示出这棵树。当你撤销时，只需要向上寻找历史节点

undo-tree 被放在了 GNU ELPA 上，并不是 MELPA，所以读者如果用了国内镜像，一定要把 GNU ELPA 加入到包管理链接中，详见上一篇教程的 MELPA 章节。

配置方面，简单的使用只需要如下配置。
其默认会为每个文件生成一个隐藏文件用来保存之前的历史记录，这对项目是个污染。因此，最后的 :custom 中设置了变量 undo-tree-auto-save-history 为空，就是关闭了这个保存历史记录的功能。
此外，也可以将所有的 undo-tree 历史记录归纳到一个专门的自定义文件夹，需要通过变量 undo-tree-history-directory-alist 来设置，读者详见文档

```lisp
(use-package undo-tree
  :ensure t
  :init (global-undo-tree-mode)
  :custom
  (undo-tree-auto-save-history nil))
```


## smart-mode-line（可选）
https://github.com/Malabarba/smart-mode-line

一个让 mode line 更加漂亮、方便管理的插件，可以自动做一些模式的隐藏等等，也可以选择多种主题。具体读者可以自行探索。这里给个基础配置：

```lisp
(use-package smart-mode-line
  :ensure t
  :init (sml/setup))
```

## good-scroll （可选）

https://github.com/io12/good-scroll.el

在现代图形界面操作系统中，光标在上下移动、翻页的时候 Emacs 会直接刷新界面，滚动时也是按行滚动，比较粗糙。
good-scroll 提供了平滑滚动，并且支持变速滚动，更加顺手。

Emacs 29 集成了 pixel-scroll-precision-mode，只需配置 (pixel-scroll-precision-mode t) 即可。

非Emacs 29
```lisp
(use-package good-scroll
  :ensure t
  :if window-system          ; 在图形化界面时才使用这个插件
  :init (good-scroll-mode))
```



# 专业 Emacs 入门（六）：插件篇——功能增强类

https://zhuanlan.zhihu.com/p/450512406

有几个插件的篇幅较长，原因在于其配置和使用相对复杂和特别，但绝对是效率利器，需要读者静下心学习。


## book mark
https://www.emacswiki.org/emacs/BookMarks

Emacs 自带的功能。

在当前光标位置打上一个书签，之后可以随时跳转回来。主要涉及到如下四个命令：
- `C-x r m` 设置书签，可以自定义名字，默认文件名
- `C-x r b` 跳转到书签
- `C-x r l` 列出所有书签
- `M-x bookmark-delete` 删除书签


## ivy view

ivy 插件的额外功能——ivy view

它与 Book mark 的区别是直接将当前 Frame 中的 Window 的状态都进行保存，然后状态间切换

- `ivy-push-view` 保存当前窗口状态
- `ivy-switch-view` 切换窗口状态
- `ivy-pop-view` 删除保存了的窗口状态


## which-key（可选）
https://github.com/justbur/emacs-which-key

。。已经在gnu-elpa 中了。

实用小工具，专门针对 Emacs 快捷键多而杂的问题，安装后，当按下部分快捷键前缀时，它会通过 minibuffer 提示你都有哪些可以按的快捷键及其命令名。


## avy
https://github.com/abo-abo/avy

极力推荐读者使用，熟练后效率可以成倍提升

最基本的功能就是无需鼠标的快速光标跳转

可以让你在不操作光标的情况下，快速对文本进行复制、剪切、粘贴，大大提高了围绕光标的操作的效率

### 操作逻辑抽象

很多 Emacs 命令的操作抽象：Filter、Select、Act，翻译过来就是“筛选”、“选择”、“行动”

### 基本用法

avy 就是在这个模式上做出了文章，其核心思想就是将三者进行解耦合，让这个模式支持更多的应用场景

```lisp
(use-package avy
  :ensure t
  :bind
  (("C-j C-SPC" . avy-goto-char-timer)))
```fo
读者也可以选择使用 avy 中示例的 M-j 作为快捷键。 


加入后让光标在这个作用域内，按下 C-M-x 让配置生效 。
此时，就可以使用 avy 了 。假如我们有如下的 Python 代码：
```python
def hello_world():
    print("Hello, world!")

def hello_coder():
    print("Hello, coder!")

def hello_emacser():
    print("Hello, emacser!")

...
```

那么如果此时我们的光标在其它位置，例如一个下面的代码中，而希望将光标移动到 hello_emacser 函数的位置，我们要么需要使用鼠标，要么需要用向上键一行一行地缓慢挪动，似乎都不够完美。
有了 avy，我们就可以按下我们刚刚设定好的快捷键 C-j C-SPC 调用 avy-goto-char-timer，此时 avy 在等待你输入“目标位置的部分文本”，例如这个场景下，就输入 ：“h“、“he” 、”hel“ 等等都可以（要快一点输入），此时，原本是 "h" 的位置会被替换为一个高亮了的其它字母，那么我们希望跳转到 hello_emacser 的函数定义处，对应了字母 g，于是我们紧接着按 g，就会发现光标成功移动到了 hello_emacser 的开头。


### 进阶用法
我们写着写着希望换一种写法，对代码进行重构，把上面的三个 print 语句合并在一个函数中，那么一行行分别去选中、复制、回来、粘贴肯定是很麻烦的。
于是，我们按下 C-j C-SPC，输入 p ( print 的开头字母），此时按下 ?，下面的 Minibuffer 就会显示出可以选择的其它行动，我们希望使用 Y: yank-line，于是输入 Y（注意大小写），然后按下我们目标 print 语句的标签，例如 a，这时，a 对应的语句就完全被复制到了当前光标所在的位置。
我们的光标没有丝毫的移动，就成功将一行代码复制了过来。

读者可以自行试试其它命令的效果，例如 kill-stay 可以“隔空剪切文本”、teleport 可以“把远处的文本传送到当前位置“等等。注意我们中途按下的 ? 只是对大家的提示 ，如果熟练记住了几个常用的“行动”，就可以直接省略按 ? 了 。

另外，记得尝试一下命令 avy-copy-line、avy-move-line、avy-copy-region 和 avy-move-region， 可以以行为单位进行操作，光标都不用动，就可以快速复制、剪切一行乃至一段文字，效率非常高。

读者会发现自己的 avy 中并没有笔者的 e: embark 这一项。事实上，这些可以选择的命令也是我们可以自定义的，这里的embark 就是添加了使用 embark 插件的一个“行动”，在下文会进一步介绍。 

如果读者不喜欢这种输多个字符然后等待 avy 生效这样的方式，也可以把快捷键调用的命令改为例如 avy-goto-word-1，这时就可以通过输某个词的首字母就可以直接触发 avy 的筛选。 


## marginalia
https://github.com/minad/marginalia

一个为 Emacs minibuffer 中的选项添加注解的插件。



## embark（可选）
https://github.com/oantolin/embark

有的时候，我们按下命令、选择了文件后，可能又后悔了，想要对相同的文件输入另一个命令
我们可以在按下 C-x k 、输入了部分文件名选中文件后 ，按下 C-. 触发 embark-act，这时按下 o 就可以在另一个新的窗口打开这个 buffer 了。我们无需放弃命令重新输入，而是继续输入就好了。

由于 ivy 和 avy 的使用其实已经大大加强了 Emacs 对文本和 Buffer 的操作能力，相对而言 embark 并没有那么必须，所以这里笔者标记为了可选。 

直接使用官网提供的配置就好（官网中强烈建议 embark 和 marginalia 一同使用，所以最好先装好 marginalia）：

如果想要让 avy 也支持 embark，需要在 avy 的配置中添加 :config：
```lisp
(use-package avy
  :ensure t
  :config
  (defun avy-action-embark (pt)
	(unwind-protect
		(save-excursion
          (goto-char pt)
          (embark-act))
      (select-window
       (cdr (ring-ref avy-ring 0))))
	t)
  (setf (alist-get ?e avy-dispatch-alist) 'avy-action-embark)
  :bind
  (("C-j C-SPC" . avy-goto-char-timer)))
```


## hydra
https://github.com/abo-abo/hydra

hydra 进一步解决了 Emacs 的复杂的命令如何组织的问题

hydra 主要功能是把一组特定场景的命令组织到一起， 通过简单按键来进行调用。这个思路和 Vim 的各种 mode 是类似的

```lisp
(use-package undo-tree
  :ensure t
  :init (global-undo-tree-mode)
  :after hydra
  :bind ("C-x C-h u" . hydra-undo-tree/body)
  :hydra (hydra-undo-tree (:hint nil)
  "
  _p_: undo  _n_: redo _s_: save _l_: load   "
  ("p"   undo-tree-undo)
  ("n"   undo-tree-redo)
  ("s"   undo-tree-save-history)
  ("l"   undo-tree-load-history)
  ("u"   undo-tree-visualize "visualize" :color blue)
  ("q"   nil "quit" :color blue)))
```

按下 C-x C-h u 来调用 hydra-undo-tree/body 这个命令，它会在 minibuffer 中显示出我们配置中的字符串
通过选择 p、n、s、l 和 u 来分别触发五个 undo-tree 的命令了

具体来说，我们通过 :hydra 标签可以声明这样一组命令，起了个名字，在上面的例子中就是 hydra-undo-tree，被称为 hydra-awesome，习惯以 hydra- 开头。
想要调出这一组，需要输入的是就是加上 /body，上例中也就是 hydra-undo-tree/body，当然你可以选择直接调其中的某个命令例如 hydra-undo-tree/undo-tree-undo。

一个值得注意的小细节是每个提示词的颜色，有的为红色，有的为蓝色。
事实上颜色是有相对应的设置的，
- 红色的表示按过了之后依然可以继续按，不会退出 hydra ；
- 蓝色表示按了一次就会退出 hydra



## multiple-cursors
https://github.com/magnars/multiple-cursors.el

多光标编辑可是编辑器的必备需求。这个插件提供了多种生成多光标的方式。

- 连续多行
  `C-SPC` 触发一次 set-mark, 然后让光标向下移动， 再输入 `M-x mc/edit-lines` 生成连续多行光标
- 编辑多处同一段文本
  选中文本，输入命令mc/mark-next-like-this、mc/mark-previous-like-this、 mc/mark-all-like-this，看名字就知道，分别可以标记下一个词、上一个词、所有词。还可以用 mc/skip-to-next-like-this 和 mc/skip-to-previous-like-this 跳过一部分。
- 鼠标点击选择
  见配置，将 mc/toggle-cursor-on-click 绑定到某个键位。笔者使用的是 Ctrl+ Shift + 鼠标左键。
```lisp
(use-package multiple-cursors
  :bind
  ("C-S-<mouse-1>" . mc/toggle-cursor-on-click)) 
```


要使用 hydra 形成一组快捷键

```lisp
(use-package multiple-cursors
  :ensure t
  :after hydra
  :bind
  (("C-x C-h m" . hydra-multiple-cursors/body)
   ("C-S-<mouse-1>" . mc/toggle-cursor-on-click))
  :hydra (hydra-multiple-cursors
		  (:hint nil)
		  "
Up^^             Down^^           Miscellaneous           % 2(mc/num-cursors) cursor%s(if (> (mc/num-cursors) 1) \"s\" \"\")
------------------------------------------------------------------
 [_p_]   Prev     [_n_]   Next     [_l_] Edit lines  [_0_] Insert numbers
 [_P_]   Skip     [_N_]   Skip     [_a_] Mark all    [_A_] Insert letters
 [_M-p_] Unmark   [_M-n_] Unmark   [_s_] Search      [_q_] Quit
 [_|_] Align with input CHAR       [Click] Cursor at point"
		  ("l" mc/edit-lines :exit t)
		  ("a" mc/mark-all-like-this :exit t)
		  ("n" mc/mark-next-like-this)
		  ("N" mc/skip-to-next-like-this)
		  ("M-n" mc/unmark-next-like-this)
		  ("p" mc/mark-previous-like-this)
		  ("P" mc/skip-to-previous-like-this)
		  ("M-p" mc/unmark-previous-like-this)
		  ("|" mc/vertical-align)
		  ("s" mc/mark-all-in-region-regexp :exit t)
		  ("0" mc/insert-numbers :exit t)
		  ("A" mc/insert-letters :exit t)
		  ("<mouse-1>" mc/add-cursor-on-click)
		  ;; Help with click recognition in this hydra
		  ("<down-mouse-1>" ignore)
		  ("<drag-mouse-1>" ignore)
		  ("q" nil)))
```
可以使用 C-x C-h m 来列出所有的命令，然后选择即可


## dashboard

https://github.com/emacs-dashboard/emacs-dashboard

dashboard 就是一个新的欢迎界面
可以列出最近打开的项目、最近打开的文件等等
按下 p 或 r 就可以快速 跳转到相应小结里
还可以列出来标记过的书签、org-mode （Emacs 自带的一个强大的笔记系统）日程、自定义控件等。

```lisp
 (use-package dashboard
  :ensure t
  :config
  (setq dashboard-banner-logo-title "Welcome to Emacs!") ;; 个性签名，随读者喜好设置
  ;; (setq dashboard-projects-backend 'projectile) ;; 读者可以暂时注释掉这一行，等安装了 projectile 后再使用
  (setq dashboard-startup-banner 'official) ;; 也可以自定义图片
  (setq dashboard-items '((recents  . 5)   ;; 显示多少个最近文件
			  (bookmarks . 5)  ;; 显示多少个最近书签
			  (projects . 10))) ;; 显示多少个最近项目
  (dashboard-setup-startup-hook))
```


## tiny
https://github.com/abo-abo/tiny

tiny 可以实现一个方便的序号宏展开。举个小例子就一目了然了：我们想要定义一组函数分别名为 int fun01、int fun02、 …… 、int fun10，正常我们只能一个个手敲，但有了 tiny，我们可以输入一个这样的简单语法：
`m1\n10|int func%02d ()`

```lisp
(use-package tiny
  :ensure t
  ;; 可选绑定快捷键，笔者个人感觉不绑定快捷键也无妨
  :bind
  ("C-;" . tiny-expand))
```

## highlight-symbol
https://github.com/nschum/highlight-symbol.el

高亮出当前 Buffer 中所有的、与光标所在处的符号相同的符号。

虽然在后面我们使用一些其他插件时也会捎带有类似功能，但它可以同时高亮很多字符，便于阅读代码等。

```lisp
(use-package highlight-symbol
  :ensure t
  :init (highlight-symbol-mode)
  :bind ("<f3>" . highlight-symbol)) ;; 按下 F3 键就可高亮当前符号
```

## rainbow-delimiters

https://github.com/Fanael/rainbow-delimiters

用不同颜色标记多级括号，方便看清代码块

```lisp
(use-package rainbow-delimiters
  :ensure t
  :hook (prog-mode . rainbow-delimiters-mode))
```


## evil（为 Vim 用户）
https://github.com/emacs-evil/evil
3.2k

```lisp
(use-package evil
  :ensure t
  :init (evil-mode))
```

evil 可以使用 C-z 切换 Emacs 按键模式和 Vim 按键模式。当然在终端中，这会覆盖掉挂起功能，想要挂起可以按 C-x C-z。 



# 专业 Emacs 入门（七）：插件篇——编程开发类

https://zhuanlan.zhihu.com/p/467681146

编辑器最基本需要三大方面的功能：

- 对编程进行辅助：自动补全、语法检查、代码跳转等。
- 项目管理、编译、运行、调试、版本控制等。
- 与相关开发工具结合等。

## 自动补全

### company-mode

Emacs 最广为使用的补全插件便是 company-mode

http://company-mode.github.io/

```lisp
(use-package company
  :ensure t
  :init (global-company-mode)
  :config
  (setq company-minimum-prefix-length 1) ; 只需敲 1 个字母就开始进行自动补全
  (setq company-tooltip-align-annotations t)
  (setq company-idle-delay 0.0)
  (setq company-show-numbers t) ;; 给选项编号 (按快捷键 M-1、M-2 等等来进行选择).
  (setq company-selection-wrap-around t)
  (setq company-transformers '(company-sort-by-occurrence))) ; 根据选择的频率进行排序，读者如果不喜欢可以去掉
```

输入前缀即可弹出自动补全。用 M-p 和 M-n 上下选择， Meta 键 + 一个数字选择相应标号的备选项。

此外，如果读者使用图形界面，可以再安装一个 company-box 用以显示图标：

```lisp
(use-package company-box
  :ensure t
  :if window-system
  :hook (company-mode . company-box-mode))
```


### TabNine - AI 自动补全

https://github.com/TommyX12/company-tabnine
https://www.tabnine.com/

```lisp
(use-package company-tabnine
  :ensure t
  :init (add-to-list 'company-backends #'company-tabnine))
```

随后，重启 Emacs，输入命令：`M-x company-tabnine-install-binary`，来安装 TabNine 的后台程序。之后就可以正常使用了。

AI 插件会导致偶尔的高 CPU 占用，如果电脑硬件性能不佳，建议尽量关闭后台训练功能，或者是直接使用传统补全插件就好。


### 代码片段模板 yasnippet

https://github.com/joaotavora/yasnippet


```lisp
(use-package yasnippet
  :ensure t
  :hook
  (prog-mode . yas-minor-mode)
  :config
  (yas-reload-all)
  ;; add company-yasnippet to company-backends
  (defun company-mode/backend-with-yas (backend)
    (if (and (listp backend) (member 'company-yasnippet backend))
	backend
      (append (if (consp backend) backend (list backend))
              '(:with company-yasnippet))))
  (setq company-backends (mapcar #'company-mode/backend-with-yas company-backends))
  ;; unbind <TAB> completion
  (define-key yas-minor-mode-map [(tab)]        nil)
  (define-key yas-minor-mode-map (kbd "TAB")    nil)
  (define-key yas-minor-mode-map (kbd "<tab>")  nil)
  :bind
  (:map yas-minor-mode-map ("S-<tab>" . yas-expand)))

(use-package yasnippet-snippets
  :ensure t
  :after yasnippet)
```

这里笔者的配置略微复杂，主要有两个考虑：
- 默认情况，yasnippet 当光标位于一个可展开的字符串后面时，按下 tab 键会自动展开。然而，tab 键在 Emacs 中本身就有着一个对齐缩进的作用，当我们写代码时，光标不论在一行代码的哪里，只要我们按下 tab，就会将这行代码缩进到最合理的位置。于是这会造成一个误判.于是解绑了 `<tab>` 键，换成了 `S-<tab>`（也就是 `Shift+<tab>`）。
- 我们希望 yasnippet 可以与自动补全协同，让自动补全也为我们补全 yasnippet 代码片段。默认情况，我们需要显式调用 company-yasnippet 来查看专属 yasnippet 的自动补全。

use-package 所安装的 yasnippet-snippets 是一个模板集合，里面包含了常见的模板。读者也可以自定义模板，参考 yasnippet 的主页即可。


### hippie 文本展开

dabbbrev 与之功能相似，二者都是 Emacs 自带的功能。默认 M-/ 键就被绑定到了 dabbrev-expand 函数。笔者选择使用前者，替换掉了这个快捷键：
`(global-set-key (kbd "M-/") 'hippie-expand)`

当我们输入几个字符前缀，然后按下 M-/ 调用 hippie-expand 函数，它会根据前缀匹配后面的内容。
不同于普通的代码补全，它的补全还包括了文件名、elisp 函数名等。
它的补全规则不包含语法分析，而是纯文本补全，很适用于我们写一些重复性的相似代码/其它文本的场合。


## 语法检查

语法检查有两个主流插件，一个是 flycheck，一个是 flymake，笔者个人感觉 flycheck 使用的人更多。

https://www.flycheck.org/en/latest/

https://www.emacswiki.org/emacs/FlyMake

flycheck 是一个对接 Emacs 和语法检查程序（也就是 linting）的插件。官网提供的方案是全局开启 flycheck
```lisp
(use-package flycheck
  :ensure t
  :init (global-flycheck-mode))
```

笔者认为全局开启略有不妥。我们可以使用 hook 来指定什么情况下启动 flycheck，例如只在编程时使用：
```lisp
(use-package flycheck
  :ensure t
  :config
  (setq truncate-lines nil) ; 如果单行信息很长会自动换行
  :hook
  (prog-mode . flycheck-mode))
```

flycheck已就绪，特定语言的语法检查程序要如何安装呢？flycheck 的主页上已为你列好各种编程语言对应的语法检查程序列表，其中包含了相应的链接。
https://www.flycheck.org/en/latest/languages.html#flycheck-languages

flycheck 对 Windows 没有官方支持


## 代码分析

通过语法分析来进行诸如寻找函数和变量的定义、引用等等与编程语言本身相关的功能

C/C++ 开发会用一些传统的 CEDET，gtag 等插件，Python 则用 elpy。

这类插件由于和编辑器直接耦合，完全由插件开发者控制，更新维护不一定及时，bug 也会较多。
因此，笔者推荐使用微软为 VSCode 设计的 Language Server Protocol (LSP) 来完成这一任务。
随着 LSP 发展壮大，它已成为一个开放的、统一的标准了，可以供 Emacs 使用。

LSP 将代码分析解耦合为前端（即编辑器提供的功能，语言无关）和后端（语法分析，语言相关）两部分，二者通过一个规定好的协议来通信，也就是 LSP 协议

为 Emacs 提供 LSP 的插件有 lsp 和 eglot，相对来说 lsp 功能更丰富.
Emacs 29 默认集成了 eglot，读者也可考虑尝试。

lsp 应当与下文介绍的 projectile 一起使用
https://emacs-lsp.github.io/lsp-mode/

这里的 lsp 只是前端部分， 后端需要单独安装，有的时候可以直接通过命令 lsp-install-server 让 lsp 为你自动安装，或者也可以手动安装你偏爱的 LSP server

根据官网，就有了一个基础配置
```lisp
(use-package lsp-mode
  :ensure t
  :init
  ;; set prefix for lsp-command-keymap (few alternatives - "C-l", "C-c l")
  (setq lsp-keymap-prefix "C-c l"
	lsp-file-watch-threshold 500)
  :hook 
  (lsp-mode . lsp-enable-which-key-integration) ; which-key integration
  :commands (lsp lsp-deferred)
  :config
  (setq lsp-completion-provider :none) ;; 阻止 lsp 重新设置 company-backend 而覆盖我们 yasnippet 的设置
  (setq lsp-headerline-breadcrumb-enable t)
  :bind
  ("C-c l s" . lsp-ivy-workspace-symbol)) ;; 可快速搜索工作区内的符号（类名、函数名、变量名等）
```

如果希望有更现代的图形化的支持，例如光标停留在一个变量或者函数时，显示相关的定义缩略信息、文档注释等，那么我们可以再安装一个 lsp-ui 插件

```lisp
(use-package lsp-ui
  :ensure t
  :config
  (define-key lsp-ui-mode-map [remap xref-find-definitions] #'lsp-ui-peek-find-definitions)
  (define-key lsp-ui-mode-map [remap xref-find-references] #'lsp-ui-peek-find-references)
  (setq lsp-ui-doc-position 'top))
```

同时，lsp 还能和我们之前安装过的 ivy 进行协作，利用 ivy 辅助 lsp。
```lisp
(use-package lsp-ivy
  :ensure t
  :after (lsp-mode))
```

这样我们可以通过命令 lsp-ivy-workspace-symbol 来搜索当前工作区的符号。 


## 代码调试

https://emacs-lsp.github.io/dap-mode/

与代码分析类似，微软设计 VSCode 时，对调试器也进行了前后端分离的设计，称为 Debug Adapter Protocol。Emacs 中可以使用 dap-mode 作为客户端。

对各种语言的配置
https://emacs-lsp.github.io/dap-mode/page/configuration/
。。没有cpp


```lisp
(use-package dap-mode
  :ensure t
  :after hydra lsp-mode
  :commands dap-debug
  :custom
  (dap-auto-configure-mode t)
  :config
  (dap-ui-mode 1)
  :hydra
  (hydra-dap-mode
   (:color pink :hint nil :foreign-keys run)
   "
^Stepping^          ^Switch^                 ^Breakpoints^         ^Debug^                     ^Eval
^^^^^^^^----------------------------------------------------------------------------------------------------------------
_n_: Next           _ss_: Session            _bb_: Toggle          _dd_: Debug                 _ee_: Eval
_i_: Step in        _st_: Thread             _bd_: Delete          _dr_: Debug recent          _er_: Eval region
_o_: Step out       _sf_: Stack frame        _ba_: Add             _dl_: Debug last            _es_: Eval thing at point
_c_: Continue       _su_: Up stack frame     _bc_: Set condition   _de_: Edit debug template   _ea_: Add expression.
_r_: Restart frame  _sd_: Down stack frame   _bh_: Set hit count   _ds_: Debug restart
_Q_: Disconnect     _sl_: List locals        _bl_: Set log message
                  _sb_: List breakpoints
                  _sS_: List sessions
"
   ("n" dap-next)
   ("i" dap-step-in)
   ("o" dap-step-out)
   ("c" dap-continue)
   ("r" dap-restart-frame)
   ("ss" dap-switch-session)
   ("st" dap-switch-thread)
   ("sf" dap-switch-stack-frame)
   ("su" dap-up-stack-frame)
   ("sd" dap-down-stack-frame)
   ("sl" dap-ui-locals)
   ("sb" dap-ui-breakpoints)
   ("sS" dap-ui-sessions)
   ("bb" dap-breakpoint-toggle)
   ("ba" dap-breakpoint-add)
   ("bd" dap-breakpoint-delete)
   ("bc" dap-breakpoint-condition)
   ("bh" dap-breakpoint-hit-condition)
   ("bl" dap-breakpoint-log-message)
   ("dd" dap-debug)
   ("dr" dap-debug-recent)
   ("ds" dap-debug-restart)
   ("dl" dap-debug-last)
   ("de" dap-debug-edit-template)
   ("ee" dap-eval)
   ("ea" dap-ui-expressions-add)
   ("er" dap-eval-region)
   ("es" dap-eval-thing-at-point)
   ("q" nil "quit" :color blue)
   ("Q" dap-disconnect :color red)))
```




## 项目管理

事实上，上述的 lsp 插件还需要配合 projectile 插件使用。原因在于，目前为止，Emacs 只是在对文件进行操作，而没有项目的概念。我们实际的开发一定是以项目为单位的，lsp 的符号查找应当也是在项目范围的。projectile 就是为 Emacs 提供了项目管理的插件。

https://docs.projectile.mx/projectile/projects.html

```lisp
(use-package projectile
  :ensure t
  :bind (("C-c p" . projectile-command-map))
  :config
  (setq projectile-mode-line "Projectile")
  (setq projectile-track-known-projects-automatically nil))

(use-package counsel-projectile
  :ensure t
  :after (projectile)
  :init (counsel-projectile-mode))
```

当同时配好 projectile 和 lsp 后，我们如果打开一个项目内的文件，lsp 会提示你让你确认一下 projectile 推测出的项目的根目录，它会以此为范围做代码分析。

配置好上述的 counsel-projectile 后，我们还会拥有 counsel 和 projectile 的协作功能。
例如，我们可以使用快捷键 C-c p p 调用 counsel-projectile-switch-project 来选择你曾经打开过的项目；
再如，我们可以使用快捷键 C-c p f 来调用 counsel-projectile-find-file 快速打开一个项目内的文件。
它利用 counsel 的搜索功能，可以模糊查找，也不必输入完整的路径，比正常 C-x C-f 要快速方便许多。 


## 局部变量

Emacs 自然也可以对项目进行特别的配置。
Emacs 有一个配置文件，就类似于 VSCode 的 .vscode/settings.json ，名为 .dir-locals.el。
这个文件是一种特殊的语法，用于保存一些变量在这个项目下的取值。

举个例子，如果我们的项目需要使用 clang 编译器的 c++11 标准做语法检查，应当如下操作：
- 输入命令 M-x add-dir-local-variable 在操作结束后，会自动创建 .dir-locals.el 文件。
- 首先它会让我们选择一下我们的这个变量是哪个 major mode 的变量，我们选择 c++-mode。
- 随后，输入我们想要设置的变量 flycheck-clang-language-standard，输入回车确认。
- 最后输入我们要设置的值 "c++11"（注意双引号表示字符串）。
- 此时命令完成我们会跳转到 .dir-locals.el 文件的 buffer，内容如下：
```lisp
;;; Directory Local Variables
;;; For more information see (info "(emacs) Directory Variables")

((c++-mode . ((flycheck-clang-language-standard . "c++11"))))
```

按下 C-x C-s 保存这个文件。下次打开这个项目的文件时，会提示你是否要应用这些变量的自定义值（为了安全性），按 y 即可生效。 

如果使用的是 gcc 编译器，将变量名中的 clang 替换成 gcc 就好。



## 环境变量

https://github.com/purcell/exec-path-from-shell

特别的，尤其是在 macOS 上，有时候我们可能会遇到一些在终端中的可执行文件放到 Emacs 图形界面下不能使用的问题。
原因就在于图形界面的环境变量没有被正确设置。
例如在图形界面使用 lsp 插件写 Python 程序并使用了 conda 虚拟环境，可能会提示你它找不到任何 language server，原因就在于 lsp 直接调用了 pyright 命令但是它没有在基本环境变量里。 

```lisp
(use-package exec-path-from-shell
  :if (memq window-system '(mac ns))
  :ensure t
  :init
  (setq exec-path-from-shell-arguments nil)
  (exec-path-from-shell-initialize))
```

注意，完成这一任务会调用 Shell 进程，速度很慢，而如果遇到日常大家 export PATH=$PATH:path/to/bin 这种字符串拼接写法时会更慢，大大拖慢 Emacs 启动速度。
根据 Purcell 的建议，这里的配置就只针对 macOS 上使用了图形界面的情况才启动这个插件的功能，并且读者应当尽量做到：
- 通过把 PATH 变量的设定放置在 ~/.profile、~/.bash_profile、~/.zshenv 而不是通通放入 ~/.bashrc 、~/.zshrc 。
- 不要使用 PATH 变量字符串拼接，而直接赋值。


## 版本管理

https://magit.vc/

magit 是 Emacs 内部的 git 管理工具，提供了对 git 方便的调用和显示。magit 几乎无需配置。
```lisp
(use-package magit
  :ensure t)
```

安装好后，在一个 git 仓库中，我们可以使用 C-x g 调用 magit-status 查看状态，相当于 git status。


## 语言相关配置

### c,cpp

编译

Emacs 本身就有一个 compile 函数可以用来编译 C/C++ 文件。
打开一个 C/C++ 项目，使用默认配置调用 `M-x compile`，它会提示 make -k 来进行编译，也就是会默认我们是一个 make 项目。 
如果我们没有 Makefile，例如我们在做算法题，只是想直接单独编译一个源文件，那么也可以直接手动输入命令：
`g++ prog.cpp -g -o exec`

事实上，这个默认值是一个 Emacs 字符串变量，如果我们希望在这个项目里使用一个固定的编译命令，就可以利用上文“局部变量”小节中提到的办法自定义局部变量 compile-command 为我们想要的编译命令。 
如果我们在之前的基础上设置，会得到如下的 .dir-locals.el 文件：

```lisp
;;; Directory Local Variables
;;; For more information see (info "(emacs) Directory Variables")

((c++-mode . ((compile-command . "g++ main.cpp -g -o exec")
	      (flycheck-clang-language-standard . "c++11"))))
```

但是显然，这里我们写死了源代码文件 main.cpp，对于一些特定小项目是可以的， 但是对于刷算法题这种需求却并不好用，因为我们不同的题目是放在不同的源文件中，每个都单独可编译。
为此，我们需要让 Emacs 自己为编译命令填写当前的源码文件（这里其实就是我们手动实现一个类似 VSCode 的 Code Runner 插件）。

首先我们可以在 init.el （或者是任何你自己定义的 elisp 文件中）定义两个函数：
```lisp
(defun file-name-only ()
  "Get the current buffer file name without directory."
  (file-name-nondirectory (buffer-name)))

(defun file-name-only-noext ()
  "Get the currennt buffer file name without directory and extension."
  (file-name-sans-extension (file-name-only)))
```

前者可以获得当前所在 buffer 的文件名，后者则得到了文件名除去后缀名的名字（用来做可执行文件名）。

随后我们设置 compile-command 为：
`(concat "clang++ -g " (file-name-only) " -o " (file-name-only-noext))`

其中 concat 是一个字符串拼接函数，可以理解为：
`"clang++ -g " + (file-name-only) + " -o " + (file-name-only-noext)`

最终得到这样的 .dir-locals.el：
```lisp
((c++-mode . ((compile-command . (concat "clang++ -g "
				  (file-name-only)
                                  " -o "
				  (file-name-only-noext)))
	      (flycheck-clang-language-standard . "c++11"))))
```

这样当我们调用 compile 命令时，可以利用字符串拼接自动补全当前所在的源文件并编译成相应的可执行文件。
注意其中的 -g 选项是用于 debug 的选项 。
读者也可以直接修改 .dir-locals.el 文件，只不过括号比较多需要注意不要出错。


---
LSP

前文提到，LSP 需要针对每个语言有一个后端程序提供分析功能。对于 C/C++ 笔者所使用的是 llvm 下的 clangd 工具，读者还可以选择 ccls。
安装可以参照它的官网
https://clangd.llvm.org/installation



对于项目需要一些特殊的编译选项，例如自定义头文件、库等，需要参照 clangd 的文档(https://clangd.llvm.org/installation#project-setup) 进行设置，
简单来说，就是在项目目录下创建一个 compile_flags.txt 文件，把编译选项写在里面就好。
写好后，clangd 在分析代码时就会使用这些选项了。 

对于 CMake 项目，CMake 可以生成 compile_commands.json 文件给 clangd 使用，参考 clangd 的文档。此外，也可以使用 cmake-ide 插件（但就不走 lsp 插件了）。对于 CMake 语法则可以安装 cmake-language-server。
https://github.com/atilaneves/cmake-ide
https://github.com/regen100/cmake-language-server

C/C++ 配置如下，其中 c-toggle-hungry-state 函数是为了在按下删除键时尽可能删除多余空白字符，例如缩进的空白、空行等，会自动删除到一个非空白字符，读者可以根据需要开启：
```lisp
(use-package c++-mode
  :functions 			; suppress warnings
  c-toggle-hungry-state
  :hook
  (c-mode . lsp-deferred)
  (c++-mode . lsp-deferred)
  (c++-mode . c-toggle-hungry-state))
```


---
调试

Emacs 本身可以直接使用 M-x gdb 利用 gdb 进行调试。

那么我们更希望使用 dap-mode， 笔者选择使用 llvm 下的 lldb-vscode 工具做后端进行调试。安装 llvm 后应该已经安装好了这个工具，否则手动安装就好。

https://emacs-lsp.github.io/dap-mode/page/configuration/#native-debug-gdblldb

```lisp
(use-package dap-lldb
  :after dap-mode
  :custom
  (dap-lldb-debug-program '("/usr/local/opt/llvm/bin/lldb-vscode"))
  ;; ask user for executable to debug if not specified explicitly (c++)
  (dap-lldb-debugged-program-function
    (lambda () (read-file-name "Select file to debug: "))))
```
最后的链接命令可以改成你觉得合适的其它路径，只要在环境变量中就好。安装好后，就可以直接使用 lldb。macOS 上 dap-mode 使用 gdb 会有问题，笔者也暂未解决。



### rust

Rust 得益于其本身完善的工具链，相比之下要简单很多，直接安装 rust-mode 和 cargo 插件，M-x cargo-process-run 就可以执行 cargo run。其余命令也都以 cargo- 为前缀，可以参考官方主页。 

https://github.com/rust-lang/rust-mode
https://github.com/kwrooijen/cargo.el


```lisp
(use-package rust-mode
  :ensure t
  :functions dap-register-debug-template
  :bind
  ("C-c C-c" . rust-run)
  :hook
  (rust-mode . lsp-deferred)
  :config
  ;; debug
  (require 'dap-gdb-lldb)
  (dap-register-debug-template "Rust::LLDB Run Configuration"
                               (list :type "lldb"
				     :request "launch"
			             :name "rust-lldb::Run"
				     :gdbpath "rust-lldb"
				     :target nil
				     :cwd nil)))

(use-package cargo
  :ensure t
  :hook
  (rust-mode . cargo-minor-mode))
```

LSP 方面，笔者使用了 rust-analyzer，其安装方法在主页上有详细说明。这里 debug 我使用的是 rust-lldb，因为 macOS 上使用 rust-gdb 有一些问题。事实上，rust 本质就是使用 lldb，所以和 C/C++ 的情况一样。另外，这篇博客有介绍更多详细的 Rust 相关配置。相信现阶段能使用 Rust 的都是有些编程经验的人，所以这里就不做赘述了。
https://github.com/rust-lang/rust-analyzer
https://rust-analyzer.github.io/manual.html#installation
https://robert.kra.hn/posts/rust-emacs-setup/


### python

Python 的运行主要考虑与它的 REPL 的配合以及虚拟环境的切换。
前者是 Emacs 自带的基础功能，后者可以安装插件 pyvenv 进行管理。
笔者平日使用 miniconda 做 Python 的环境管理（F.Y.I., miniconda 是 anaconda 的最精简版），如果你使用 anaconda，只需要改个名字；如果你使用 Python 本身的 virtualenv，pyvenv 插件更是直接支持。

https://github.com/jorgenschaefer/pyvenv

```lisp
(use-package python
  :defer t
  :mode ("\\.py\\'" . python-mode)
  :interpreter ("python3" . python-mode)
  :config
  ;; for debug
  (require 'dap-python))

(use-package pyvenv
  :ensure t
  :config
  (setenv "WORKON_HOME" (expand-file-name "~/miniconda3/envs"))
  ;; (setq python-shell-interpreter "python3")  ; （可选）更改解释器名字
  (pyvenv-mode t)
  ;; （可选）如果希望启动后激活 miniconda 的 base 环境，就使用如下的 hook
  ;; :hook
  ;; (python-mode . (lambda () (pyvenv-workon "..")))
)
```

读者需要把这里的 ~/miniconda3 路径换成自己的路径，一般默认或者在用户目录下，形如 ~/anaconda3；
或者在根目录下，形如 /anaconda3。使用 virtualenv 的话则不需要这行配置。

有了 pyvenv，我们打开 Python 项目，如果想要切换环境，就输入命令 `M-x pyvenv-workon`

想要使用 REPL，首先打开一个 Python 文件，然后使用快捷键 C-c C-p 调用命令 run-python，启动 Python 解释器，Emacs 会弹出一个名为 `*Python*` 的 buffer，就是 Python 的 REPL。
随后我们光标放在 Python 文件的 buffer 中，可以选中一部分代码，或者不选表示全部代码，按 C-c C-c 调用命令 python-shell-send-buffer，把选中的代码发送到 REPL 中，此时 `*Python* buffer` 中就会相应地执行了这些代码。


LSP

Python 的 language server 比较多，包括 python-language-server（pyls）、Jedi、Microsoft Python Language Server 和 Pyright。pyls 普遍评价是比较慢，所以不推荐使用。后三个读者可以根据自己喜好选择。笔者选择了评价较好的 Pyright。可以通过 pip3 install pyright 来手动安装。Emacs 中的相应配置如下：
```lisp
(use-package lsp-pyright
  :ensure t
  :config
  :hook
  (python-mode . (lambda ()
		  (require 'lsp-pyright)
		  (lsp-deferred))))
```


调试
使用 dap-mode 调试前需先手动通过 pip 安装 python3 -m pip install ptvsd。



## 工作区管理

对于涉及多个项目的更复杂的任务，我们需要一个工作区（workspace）来进行管理和切换。treemacs 为我们提供了这样的功能。如果读者不需要工作区管理这样复杂的功能，而只是想要一个树形文件结构显示，那么可以考虑使用 neotree。

https://github.com/Alexander-Miller/treemacs
2k        yesterday

https://github.com/jaypei/emacs-neotree
1.5k      x months ago

```lisp
(use-package treemacs
  :ensure t
  :defer t
  :config
  (treemacs-tag-follow-mode)
  :bind
  (:map global-map
        ("M-0"       . treemacs-select-window)
        ("C-x t 1"   . treemacs-delete-other-windows)
        ("C-x t t"   . treemacs)
        ("C-x t B"   . treemacs-bookmark)
        ;; ("C-x t C-t" . treemacs-find-file)
        ("C-x t M-t" . treemacs-find-tag))
  (:map treemacs-mode-map
	("/" . treemacs-advanced-helpful-hydra)))

(use-package treemacs-projectile
  :ensure t
  :after (treemacs projectile))

(use-package lsp-treemacs
  :ensure t
  :after (treemacs lsp))
```

配置好后，我们可以使用 C-x t t 调出 treemacs，用 M-0 在我们的代码 Buffer 和 treemacs 边栏之间切换。
当我们光标进入 treemacs 边栏时，可以按问号 ? 来调出帮助。


我们可以使用 treemacs-create-workspace 来创建一个新的工作区。它会在你当前光标所在处打出一个小窗提示你输入 “Workspace name”，起个合适的名字。创建好后，我们在 treemacs 边栏中输入 C-c C-p a 调用 treemacs-add-project-to-workspace 来添加一个项目路径，用 C-c C-p d 去除工作区选中的项目。这些都可以在 ? 显示的帮助中找到。想要切换工作区时，使用 treemacs-switch-workspace 命令。 

由于我们安装了 treemacs-projectile，我们也可以使用 treemacs-projectile 命令将我们 projectile 当中保存过的项目直接导入到 treemacs 中。

此外，我们还可以调用 treemacs-edit-workspaces 来通过配置文件来修改我们的工作区设置，修改完成后用 treemacs-finish-edit 命令或 C-c C-c 结束编辑。它会自动检查语法。（ps：所使用的语法是 Emacs org-mode，一个 Emacs 内部的笔记系统，可以近似理解为高级 markdown，后续教程会做介绍）。

treemacs 不仅能够显示文件，还会显示其中的符号，包括函数定义、结构体定义等等。配置中的 (treemacs-tag-follow-mode) 就是希望 treemacs 始终跟随着我们代码光标所在位置进行移动。

最后，? 显示出的帮助是 treemacs-common-helpful-hydra，我们可以按 / 或调用 treemacs-advanced-helpful-hydra 来显示出更多帮助，包括了对文件的操作和对工作区的操作。

treemacs 正常情况不会根据我们打开的 Buffer 不同而切换项目或工作区。其原因在于有时候我们只是打开一些额外的文件，并不希望工作区也进行转变。而大多数情况，我们也确实并不需要频繁切换工作区，所以我们主动去调用 treemacs-switch-workspace 就好。如果读者觉得这样过于繁琐，也可以开启一个 treemacs-project-follow-mode，它会根据我们的文件跳转到相应的项目中。 或者直接使用本节开头提到的 neotree 插件。

treemacs 的边栏是一个固定在 Buffer 内的 frame， 所以操作起来有些特殊。


## 终端

`M-x shell` 即可打开你的默认 Shell

使用 Shell 主要注意操作上，我们不再使用方向键来回滚历史命令，而是通过 M-p 和 M-n 来翻看命令。

此外 Emacs 还特别打造了 eshell 作为 Emacs 的专用 Shell，但笔者几乎没有使用所以只做简要介绍。
https://www.gnu.org/software/emacs/manual/html_node/eshell/index.html


## 远程访问

Emacs 本身有一个 tramp 功能可以实现远程 ssh 访问。无需任何配置，使用 C-x C-f 打开文件时，删除掉所有路径，只留下一个斜杠 /，然后输入成 /ssh:user@address:，后面接远程服务器上的目录，就可以访问到相应的远程服务器的相应文件或目录。如果在 ~/.ssh/config 中有配置 Host 名称，也可以直接输入名称 /ssh:hostname:。 

但是，虽然可以远程编辑，却并不能继续使用以上大部分功能，因为这些插件都是安装在本地的，而源代码和项目都在远程服务器上。VSCode 对此的解决办法是可以在远程服务器也安装一遍这些插件。对 Emacs 来说，其实只要把 ~/.emacs.d 目录复制到远程服务器上，然后在上面安装一个 Emacs，就可以直接在终端中使用远程的 Emacs 了，毕竟我们用 Emacs 本来就不需要鼠标。这样可能比远程访问来得更加简单好用。




# 专业 Emacs 入门（八）：外观主题

https://zhuanlan.zhihu.com/p/598809889


## Face

https://www.emacswiki.org/emacs/Face#face

Emacs 中掌管显示的专用名词是 Face，例如对文字来说，其字体、字号、颜色、背景都称为 Face。


`M-x customize-face` 然后输入相应的 Face 名称即可自定义

`M-x list-faces-display` 就可以显示当前界面下所有的 Face 的名字及颜色
我们想更改光标的颜色，可以输入 "cursor"


除了使用M-x list-faces-display 列出当前界面所有的 Face 外，
还可以使用快捷键 `C-u C-x =` 调用带前缀参数的 what-cursor-position 命令，它会在新的窗口中显示光标所在位置的界面信息，其中包括了 Face。

更通用的设置方式则是直接使用 Customization 进行搜索查找。
强调一下，Customization 本身并不只包含关于 Face 的设置，各种变量都可以用它更改。
进入 Customization 对首页可以输入命令 `M-x customize`，可以点选其中的 “Face” 进入关于 Face 的设置


## 配置 Custom 文件

当读者保存了上述自定义配置后，默认会在你的初始化文件（如 ~/.emacs.d/init.el）的末尾添加一段代码

部分读者可能会提出，Custom 如此修改初始化文件，把初始化文件弄的不美观了；
或者如果用户有时候在本机使用图形界面 Emacs，有时候在服务器上使用命令行 Emacs，
二者希望进行不同的 Custom 设置但又不想维护两组 Emacs 配置，该如何操作呢？

这段配置可以单独放在一个文件中，比如建一个 ~/.emacs.d/custom.el 文件，把上方的代码块完全剪切到其中，然后在配置文件如 ~/.emacs.d/init.el 中写入：
```lisp
(setq custom-file "~/.emacs.d/custom.el")
(load custom-file)
```

读者更可以利用前面所学的模块化配置的方法，新建一个 ~/.emacs.d/lisp/init-theme.el 文件，把这两句代码写进去后，在 ~/.emacs.d/init.el 中 require 导入。 


多个场景下的 Custom 配置，可以参考如下 init-theme.el 的配置
```lisp
(setq custom-nw-file (expand-file-name "custom-nw.el" user-emacs-directory))
(setq custom-gui-file (expand-file-name "custom-gui.el" user-emacs-directory))

(if (display-graphic-p)
	(progn
	  (setq custom-file custom-gui-file)
          ; (add-to-list 'default-frame-alist '(ns-appearance . dark)) ; macOS 下让窗口使用暗色主题
	  ;; other settings
        )
  (progn
	(setq custom-file custom-nw-file)
        ;; other settings
	))

(load custom-file)
```

## 配置主题

Emacs 的主题非常多，读者只需要在搜索引擎中搜索 “Emacs 主题”，或在国外搜索引擎中搜索 “Emacs themes” 就能得到非常多的结果。比较全的网站有 Emacs Themes Gallery 和 GitHub Topics：emacs-theme。

https://emacsthemes.com/

https://draculatheme.com/emacs

https://github.com/topics/emacs-theme


笔者选择了 doom-monokai-octagon 主题：
```lisp
(use-package doom-themes
  :ensure t
  :config
  ;; Global settings (defaults)
  (setq doom-themes-enable-bold nil    ; if nil, bold is universally disabled
	doom-themes-enable-italic t) ; if nil, italics is universally disabled
  (load-theme 'doom-monokai-octagon t)
  (doom-themes-treemacs-config))
```

最好搭配下文的 all-the-icons 一同使用。 


## 配置图标

图形界面的 Emacs 可以安装一个 all-the-icons 插件来为 Emacs 提供图标的支持，对于命令行 Emacs 来说如果终端能够支持这些图标字体，也是可以使用的

https://github.com/domtronn/all-the-icons.el

```lisp
(use-package all-the-icons
  :if (display-graphic-p))
```

重启 Emacs 后安装字体：`M-x all-the-icons-install-fonts`



## mode-line

mode-line 的显示非常见仁见智，有的人喜欢花哨的，有的人喜欢简约的。
笔者属于后者，所以没有配置很复杂的图形显示。
mode-line 可以使用 powerline、spaceline 等。
powerline 在 zsh 里比较流行，spaceline 则是从 Spacemacs 衍生出来的。

https://github.com/milkypostman/powerline

https://github.com/TheBB/spaceline

https://github.com/Malabarba/smart-mode-line

这里再次提一下在《专业 Emacs 入门（五）：插件篇——功能优化类》里提及的插件 smart-mode-line 的额外配置。
mode-line 由于会显示出当前的所有次模式（minor mode），很混乱，而 smart-mode-line 可以规整 mode-line， 但用起来还是会发现经常一些不太重要的 minor mode 被显示出来，重要的反而被排在了后面。
这时候我们可以隐藏一部分 minor mode。

隐藏 minor mode 实际上是由另一个插件 rich-minority 来完成的，但安装了 smart-mode-line 之后已经被一起安装好了。还有一个类似功能的插件是 diminish，读者可以自行选择其中之一。

https://github.com/myrjola/diminish.el

https://github.com/Malabarba/rich-minority


```lisp
(use-package smart-mode-line
  :ensure t
  :init
  ; (setq sml/no-confirm-load-theme t)  ; avoid asking when startup
  (sml/setup)
  :config
  (setq rm-blacklist
    (format "^ \\(%s\\)$"
      (mapconcat #'identity
        '("Projectile.*" "company.*" "Google"
	  "Undo-Tree" "counsel" "ivy" "yas" "WK")
         "\\|"))))
```
就是添加了一个 config，把一些不想被显示的 minor mode 加入黑名单

读者只需要修改其中的那个列表：
```lisp
'("Projectile.*" "company.*" "Google"
  "Undo-Tree" "counsel" "ivy" "yas" "WK")
```


# 专业 Emacs 入门（九）：实用小技巧


https://zhuanlan.zhihu.com/p/599202074


## 笔记

### markdown

笔者对 Emacs 中的 Markdown 的态度是轻度使用，不希望费力配置，甚至不想记太多操作，所以本文只介绍 Emacs 自带的功能 markdown-mode。

无需任何配置，打开 .md 文件就会自动使用 markdown-mode，Emacs 会提供一些语法高亮，还有一些语法的快捷键。

例如插入代码块可以输入命令 `M-x markdown-insert-gfm-code-block` 然后输入想要输入的编程语言名字即可，其快捷键是 `C-c C-s C`。所有相关命令都是以 markdown-insert- 做前缀的


此外还有一些方便跳转的快捷键，例如：
- `C-c C-n` 调用 markdown-outline-next 可以快速跳到下节，C-c C-u 调用 markdown-outline-up 跳到上一节。
- `C-c C-f` 调用 markdown-outline-next-same-level 跳到下一个同级章节，C-c C-b 调用 markdown-outline-previous-same-level 跳到上一个同级章节。 

预览 Markdown，可以输入 `M-x markdown-live-preview-mode` 或输入快捷键 `C-c C-c l` 就可以在另一个窗口里预览 。

如果确实认为自带的功能很简陋，读者可以尝试 Wiki 中提及的 impatient-mode、livedown-mode 和 realtime-preview。
impatient-mode 根据介绍是一个基于 Emacs Lisp 的为 HTML 提供预览的工具，但可以稍微改造一下用于 Markdown 的预览。 
其它两个模式分别需要依赖 Node.js 和 Ruby 这类外部工具，读者按需选择就好。 


### LaTeX, pdf

LaTeX 是一个功能极其强大的排版系统，一般来说日常无需使用如此重量级的工具，笔者仅在写一些比较重要的报告和论文时才会使用。
如果没有这类需求可直接略过本节。

pdf
https://github.com/vedang/pdf-tools#keybindings-for-navigating-pdf-documents


### 中文支持

这里主要是指对中文内容提供两点支持：
- 中文分词：
  Emacs 可以做到 M-b、M-f 来按中文词进行光标移动。
  这种支持对于快速编辑中文、保持 Emacs 的操作一致性非常重要。
  默认情况下，对一段中文文本按下 M-b/M-f 会直接跳到上一个/下一个标点符号，失去了以词为单位移动的意义。
- 中英文符号的自定义切换。
  如果读者写过带公式、代码块的中文 Markdown/LaTeX，一定会发现有一件事非常非常令人厌烦，就是中英文输入法的切换。
  在输入内容和标点时需要中文字符，但在输入符号时需要英文字符，经常写起来会被这些切换搞得头脑混乱。
  虽然可以利用编辑器的快捷键缓解问题，但不具有通用性，不尽如人意。

解决分词问题笔者使用了 emacs-chinese-word-segmentation 工具。该工具借助 jieba 分词实现了分词功能

https://github.com/kanglmf/emacs-chinese-word-segmentation

笔者建议在 ~/.emacs.d/chinese/ 等目录下 clone 该项目，然后按照其主页的命令使用 make 进行编译。
随后把其主页的配置内容放到合适的配置文件中去。
在想要使用这个中文分词时，手动调用 cns-mode 就可以了。
如果想要在某个特定项目中使用，那么可以加入到局部变量 .dir-locals.el 中，如：
`((markdown-mode . ((eval . (cns-mode)))))`

Emacs 还有一个 pyim 插件，是一个内部中文输入法，能支持五笔、双拼等，安装了它后，也可以获得分词功能。但是 pyim 体验一般，而且毕竟是独立于系统的输入法的，在 Emacs 内外切换两个输入法不是很舒适，用户完全可以使用外部的选择众多的功能强大的输入法搭配 cns-mode，因此笔者并不推荐 pyim。

而解决输入法切换问题，可直接利用 Emacs 的 keyboard-translate 功能，该功能可以实现把一个符号映射成另一个符号。笔者针对 Markdown 使用了如下设置：

https://ftp.gnu.org/old-gnu/Manuals/emacs-20.7/html_node/emacs_457.html

```lisp
(add-hook 'markdown-mode-hook (lambda ()
			        (keyboard-translate ?¥ ?$)
				(keyboard-translate ?· ?`)
				(keyboard-translate ?～ ?~)
                                (keyboard-translate ?、 ?\\)
				(keyboard-translate ?｜ ?、)
				(keyboard-translate ?\「 ?{)
				(keyboard-translate ?\」 ?})
				(keyboard-translate ?\《 ?<)
				(keyboard-translate ?\》 ?>)))
```


### org-mode

非常强大的笔记模式，在 Emacs 中使用的体验、功能的强大远超 Markdown。
org-mode 的主要缺点是一方面需要一定的学习成本，另一方面只能在 Emacs 中使用，不方便直接分享。


## 计算器

写程序也是偶尔需要计算器的，例如算一算字节数、传输速率、做一做十六进制转化等。Emacs 内置了一个非常强大的计算器：calc。除了常见普通计算器的功能，还包括：

https://www.gnu.org/software/emacs/manual/html_node/calc/index.html

- 复数运算
- 2～36 进制运算
- 质因数分解
- 线性代数、矩阵运算
- 微积分
- 方程求解
- 逆波兰表达式
- 与 GNUPLOT 对接画图


`M-x calc` 启动计算器。 按 `h T` 可以打开教程。 

默认使用 RPN（Reverse Polish Notation，逆波兰表达式或称后缀表达式）

例如计算 时需要依次输入：2、3、4、*、+

按 backspace 可以移除栈顶元素，按 TAB 可以交换栈顶的两个元素，按 `M-TAB` 可以把第三个元素拿出来重新放入栈顶，U 可以撤销操作。很多运算符也都有含义，例如 & 取倒数 ，^ 取幂等。 

想要输入十六进制的数，例如 0xFE，只需要输入 16#fe ，其它进制也一样，N 进制就输入 N# 开头，后接数字即可。 N 的取值是 2～36（笔者推测之所以止步于三十六进制是因为 10 个数字+26 个字母共计 36 个字符）。

运算结果默认显示为十进制，想要显示为 N 进制，输入 d r N。 此外，还有快捷输入：
- 显示为十进制：d 0
- 显示为二进制：d 2
- 显示为十六进制：d 6
- 显示为八进制：d 8


## Emacs server

由于 Emacs 的启动较慢，在命令行使用 Emacs 时，频繁启动关闭非常不方便。
一种方式是启动后按 C-z 挂起 Emacs 回到 Shell，之后再输入命令 fg （Foreground 的缩写）回到 Emacs。

另一种方式。Emacs 为用户提供了一个 client-server 模式，用户可以设置开机启动 Emacs server 的守护进程，然后在需要使用 Emacs 编辑文件时，通过 Emacs client 打开 Emacs。由于 Emacs 已在后台启动，client 可以秒开 Emacs。

手动启动，在 Shell 中输入：
`$ emacs --daemon`

之后想要打开文件 test.c 时，输入：
`$ emacsclient test.c`


## ChatGPT

chatgpt-shell 可以在 Emacs 内通过交互式界面与 ChatGPT 通信。其使用方式非常简单，只需要根据其主页导入插件并设置 OpenAI key：

然而这里直接把密钥写进配置文件并不是一个好选择，因为大家通常会在云端备份自己的 Emacs 配置，会暴露自己的密钥。所以更推荐使用密码管理器，例如 Unix 平台上可以使用 pass，将密钥保存到 pass 中并进行这样的配置：


## google-this

在 Emacs 内选中一段文本，按下 C-c / t 触发 google-this，即可搜索这段文本。注意这个插件一般是在本地机器才会使用，不适合在服务器上用。

https://github.com/Malabarba/emacs-google-this

```lisp
(use-package google-this
  :ensure t
  :init
  (google-this-mode)) 
```




# 专业 Emacs 入门（十）：笔记系统 org-mode

https://zhuanlan.zhihu.com/p/633047823


org-mode 是 Emacs 中非常强大的笔记模式，它的语法与 Markdown 类似，但 Emacs 为其赋予了很多功能，使用体验极佳，例如：

https://orgmode.org/

- 章节折叠、跳转
- 方便的多状态、多层级 Todo List
- 丰富多样的链接引用功能
- 表格操作（甚至可以像 Excel 一样输入公式自动计算）
- 插入代码
- 导出成各种格式（Markdown、HTML、LaTeX 等）
- 依附于 Emacs 生态， 有着连贯的使用体验


## 基础结构

https://orgmode.org/manual/index.html

org-mode 的文档非常简短，建议读者大致过一遍，至少看看目录

只要打开后缀名为 .org 的文件就会自动启动 org-mode。 语法和 Markdown 类似，只不过星号（*）开头是节，或可以说是标题：

通过按 `<TAB>` 和 `S-<TAB>` 可以对小节进行折叠

下面则是一个链接（可以不止是链接外部网页，也可以是本地文件）：
`[[https://zhihu.com][知乎]]`

#+ 开头的是该文件的属性，例如下面定义了 foo 属性的值为 bar。属性主要用于为 org-mode 提供高级功能。
`#+foo:bar`

org-mode 的配置自由度非常高，每个人都有自己的偏好，不复杂但可能会很长。
因此建议根据基本配置中的内容将 org-mode 相关配置单独放到一个文件中（例如 ~/.emacs.d/lisp/init-org.el）。

一般来说我们希望把自己的笔记整理到一起，变量 org-directory 表示了用户希望把笔记放到哪里，默认是 ~/org/ 目录，可以先进行如下设置：
`(setq org-directory (file-truename "~/org/"))`


## Zettelkasten 笔记法

一个很流行的笔记方式。
大概的思想就是说，我们以前组织文件、书籍往往是按照内容组织成树形结构，自上而下有一级一级的文件夹，将知识分门别类归纳其中。
但我们在学习的过程中，很难在一开始就把知识整理成体系，而是先零散学习，之后随着知识面的增加逐渐形成体系。
因此，相比于自上而下的笔记方式，自下而上的方式更符合人的学习过程。
此外，知识本身并不是个树形结构，知识之间有很多复杂的关联，应该形成一个知识网络，树形分门别类的方式限制了知识间的关联。
简而言之，树形结构的组织方式更像是一个专业的学科综述，它不适合作为我们日常的笔记方法。

Zettelkasten 笔记法就是一个自下而上的知识网络的方式。
我们在学习新事物时，先不要去管它到底应该属于哪个分支，而是记下来再说。
而等到我们意识到这部分知识和我们已有的知识的关联时，就通过链接的方式与我们已有的知识网络连接起来，由此我们的知识网络就增加了一个新的节点。
通过链接的方式，我们可以逐渐组织我们自己的知识网络，没有任何结构性的限制。
一些常见笔记软件例如 Notion 也融入了这种方法。

那么在 Emacs 中，org-roam 就提供了这样的功能。以下是一个基本配置：

https://www.orgroam.com/

```lisp
(use-package org-roam
   :ensure t
   :after org
   :init
   (setq org-roam-v2-ack t) ;; Acknowledge V2 upgrade
   :config
   (org-roam-setup)
   :custom
   (org-roam-directory (concat org-directory "roam/")) ; 设置 org-roam 目录
   :bind
   (("C-c n f" . org-roam-node-find)
    (:map org-mode-map
          (("C-c n i" . org-roam-node-insert)
           ("C-c n o" . org-id-get-create)
           ("C-c n t" . org-roam-tag-add)
           ("C-c n a" . org-roam-alias-add)
           ("C-c n l" . org-roam-buffer-toggle)))))
```

在 org-roam 中，一个文件就是一个节点（node）。
我们通过它提供的函数来新建文件，即按 `C-c n f` 调用 org-roam-node-find 来新建或打开一个笔记文件。
新建时会让我们输入一个标题，这个后面随时可改，org-roam 会在我们指定的 org-roam-directory 里（这里设定的是 ~/org/roam/ 目录）新建一个带时间戳的文件作为实际存放这个笔记的文件，并在属性中绑定一个唯一的 ID，以防止标题重名。
我们并不需要手动去访问这个目录里的文件，只需要用 C-c n f 来打开文件即可。
org-roam 会自动和 ivy 或 helm 协作，提供补全功能。

打开了一个笔记文件后，可以按 `C-c n i` 调用 org-roam-node-insert 来关联另一个节点。
此外也可以对文件起别名、添加标签等等。
其余功能都可以在文档中找到。
对一个链接可以按 `C-c C-o` 调用 org-open-at-point 打开。

此外，对于新建的笔记，我们可以自定义它的模板，例如笔者希望能记录每个笔记最后一次修改的时间，这里写了较多的辅助函数（pv/ 前缀的函数），但是逻辑就是在文件保存时（before-save）绑定一个 Hook（pv/org-set-last-modified），可以把当前的时间记录下来：

```lisp
(use-package org-roam
   :ensure t
   :after org
   :init
   (setq org-roam-v2-ack t) ;; Acknowledge V2 upgrade
   :config
   (org-roam-setup)
   ;;--------------------------
   ;; Handling file properties for ‘LAST_MODIFIED’
   ;;--------------------------
   (defun pv/org-find-time-file-property (property &optional anywhere)
     "Return the position of the time file PROPERTY if it exists.

When ANYWHERE is non-nil, search beyond the preamble."
     (save-excursion
       (goto-char (point-min))
       (let ((first-heading
              (save-excursion
                (re-search-forward org-outline-regexp-bol nil t))))
         (when (re-search-forward (format "^#\\+%s:" property)
                                  (if anywhere nil first-heading)
                                  t)
           (point)))))

   (defun pv/org-has-time-file-property-p (property &optional anywhere)
     "Return the position of time file PROPERTY if it is defined.

As a special case, return -1 if the time file PROPERTY exists but
is not defined."
     (when-let ((pos (pv/org-find-time-file-property property anywhere)))
       (save-excursion
         (goto-char pos)
         (if (and (looking-at-p " ")
                  (progn (forward-char)
                         (org-at-timestamp-p 'lax)))
             pos
           -1))))
   (defun pv/org-set-time-file-property (property &optional anywhere pos)
    "Set the time file PROPERTY in the preamble.

When ANYWHERE is non-nil, search beyond the preamble.

If the position of the file PROPERTY has already been computed,
it can be passed in POS."
    (when-let ((pos (or pos
                        (pv/org-find-time-file-property property))))
      (save-excursion
        (goto-char pos)
        (if (looking-at-p " ")
            (forward-char)
          (insert " "))
        (delete-region (point) (line-end-position))
        (let* ((now (format-time-string "[%Y-%m-%d %a %H:%M]")))
          (insert now)))))

  (defun pv/org-set-last-modified ()
    "Update the LAST_MODIFIED file property in the preamble."
    (when (derived-mode-p 'org-mode)
      (pv/org-set-time-file-property "last_modified")))
   :hook
   (before-save . pv/org-set-last-modified) ; 保存文件时调用
   :custom
   (org-roam-directory (concat org-directory "roam/"))  ; 设置 org-roam 目录
   ;; 自定义默认模板
   (org-roam-capture-templates
    '(("d" "default" plain "%?"
       :if-new
       (file+head "${slug}-%<%Y%m%d%H%M%S>.org"
                  "#+title: ${title}\n#+date: %u\n#+last_modified: \n\n")
       :immediate-finish t)))
   :bind (("C-c n f" . org-roam-node-find)
          (:map org-mode-map
            (("C-c n i" . org-roam-node-insert)
            ("C-c n o" . org-id-get-create)
            ("C-c n t" . org-roam-tag-add)
            ("C-c n a" . org-roam-alias-add)
            ("C-c n l" . org-roam-buffer-toggle)))))
```

笔者这里只有一个默认模板，如果用户设定了多个模板，在新建文件时就会询问用户想要用哪个模板新建，十分方便。


## 任务管理

org-mode 另一个用途是可以管理自己的任务清单。在毫无自定义配置时，写起来大概是这样的：

任务管理是要搭配 org-agenda 一同使用。
实际的使用方式是，用户在固定的一个或几个文件中写自己的任务清单，然后调用 org-agenda 命令，它会为我们整理一个漂亮的==日程安排==。
其中可以选择很多种方式展示，例如过滤掉已完成的任务，按 Deadline 顺序显示等等。

可以看到，org-mode 结合 org-agenda 后，不仅仅是一个简单的 todo-list，还是一个日历，还可以具备多状态、多优先级、多标签。


### 配置

笔者的配置主要来自于这篇博客和这篇博客。
在 :init 中，笔者设置了 pv/org-agenda-files 为目录 ~/org/Agenda/，这表示“该目录下的所有文件都是用于写 todo-list 的，在使用 org-agenda 时需要对它们进行读取”。
随后新建一个文件 ~/org/Agenda/agenda.org 用来写自己的 todo-list，调用 C-c a d 就可以展示类似上图的任务列表视图。

http://doc.norang.ca/org-mode.html

https://blog.aaronbieber.com/2016/09/24/an-agenda-for-life-with-org-mode.html


```lisp
(setq org-directory (file-truename "~/org/"))
(setq pv/org-agenda-files `(,(concat org-directory "Agenda/")))

(use-package org
  :init
  (require 'org-indent)
  :config
  (defun pv/init-org-hook ()
    (setq truncate-lines nil)
    (org-toggle-pretty-entities)) ; display LaTeX symbols
  (defun pv/org-skip-subtree-if-priority (priority)
    "Skip an agenda subtree if it has a priority of PRIORITY.

PRIORITY may be one of the characters ?A, ?B, or ?C."
    (let ((subtree-end (save-excursion (org-end-of-subtree t)))
          (pri-value (* 1000 (- org-lowest-priority priority)))
          (pri-current (org-get-priority (thing-at-point 'line t))))
      (if (= pri-value pri-current)
          subtree-end
        nil)))
  (defun pv/org-skip-subtree-if-habit ()
    "Skip an agenda entry if it has a STYLE property equal to \"habit\"."
    (let ((subtree-end (save-excursion (org-end-of-subtree t))))
      (if (string= (org-entry-get nil "STYLE") "habit")
          subtree-end
        nil)))
  :hook
  (org-mode . pv/init-org-hook)
  :custom
  (org-hide-leading-stars t "clearer way to display")
  (org-startup-with-inline-images t "always display inline image")
  (org-image-actual-width 600 "set width of image when displaying")
  (org-outline-path-complete-in-steps nil)
  (org-todo-keywords
   (quote ((sequence "TODO(t)" "NEXT(n)" "|" "DONE(d)")
           (sequence "WAITING(w@/!)" "|" "CANCELLED(c@/!)" "MEETING"))))
  (org-todo-keyword-faces
   (quote (("TODO" :foreground "goldenrod1" :weight bold)
           ("NEXT" :foreground "DodgerBlue1" :weight bold)
           ("DONE" :foreground "SpringGreen2" :weight bold)
           ("WAITING" :foreground "LightSalmon1" :weight bold)
           ("CANCELLED" :foreground "LavenderBlush4" :weight bold)
           ("MEETING" :foreground "IndianRed1" :weight bold))))
  (org-todo-state-tags-triggers
   (quote (("CANCELLED" ("CANCELLED" . t))
           ("WAITING" ("WAITING" . t))
           (done ("WAITING"))
           ("TODO" ("WAITING") ("CANCELLED"))
           ("NEXT" ("WAITING") ("CANCELLED"))
           ("DONE" ("WAITING") ("CANCELLED")))))
  (org-adapt-indentation t)
  (org-agenda-files pv/org-agenda-files)
  ;; Do not dim blocked tasks
  (org-agenda-dim-blocked-tasks nil)
  ;; compact the block agenda view
  (org-agenda-compact-blocks t)
  (org-agenda-span 7)
  (org-agenda-start-day "-2d")
  (org-agenda-start-on-weekday nil)
  (org-agenda-tags-column -86) ; default value auto has issues
  ;; Custom agenda command definitions
  (org-agenda-custom-commands
   (quote (("d" "Daily agenda and all TODOs"
            ((tags "PRIORITY=\"A\""
                   ((org-agenda-skip-function '(org-agenda-skip-entry-if 'todo 'done))
                    (org-agenda-overriding-header "High-priority unfinished tasks:")))
             (agenda "" ((org-agenda-ndays 1)))
             (alltodo ""
                      ((org-agenda-skip-function '(or (pv/org-skip-subtree-if-habit)
                                                      (pv/org-skip-subtree-if-priority ?A)
                                                      (org-agenda-skip-if nil '(scheduled deadline))))
                       (org-agenda-overriding-header "ALL normal priority tasks:"))))
            ((org-agenda-compact-blocks t)))
           ("p" "Projects"
            ((agenda "" nil)
             (tags "REFILE"
                   ((org-agenda-overriding-header "Tasks to Refile")
                    (org-tags-match-list-sublevels nil)))
             (tags-todo "-CANCELLED/!"
                        ((org-agenda-overriding-header "Stuck Projects")
                         (org-agenda-skip-function 'bh/skip-non-stuck-projects)
                         (org-agenda-sorting-strategy
                          '(category-keep))))
             (tags-todo "-CANCELLED/!NEXT"
                        ((org-agenda-overriding-header (concat "Project Next Tasks"
                                                               (if bh/hide-scheduled-and-waiting-next-tasks
                                                                   ""
                                                                 " (including WAITING and SCHEDULED tasks)")))
                         (org-agenda-skip-function 'bh/skip-projects-and-habits-and-single-tasks)
                         (org-tags-match-list-sublevels t)
                         (org-agenda-todo-ignore-scheduled bh/hide-scheduled-and-waiting-next-tasks)
                         (org-agenda-todo-ignore-deadlines bh/hide-scheduled-and-waiting-next-tasks)
                         (org-agenda-todo-ignore-with-date bh/hide-scheduled-and-waiting-next-tasks)
                         (org-agenda-sorting-strategy
                          '(todo-state-down effort-up category-keep))))
             (tags "-REFILE/"
                   ((org-agenda-overriding-header "Tasks to Archive")
                    (org-agenda-skip-function 'bh/skip-non-archivable-tasks)
                    (org-tags-match-list-sublevels nil))))
            nil))))
  :bind
  (("C-c a" . 'org-agenda)
   :map org-mode-map
   ("C-c C-q" . counsel-org-tag)))
```

值得一提的是，对一个 TODO 可以用 org-deadline 来指定 deadline，用 org-schedule 规划时间。例如：
```text
* TODO 学习 Emacs 
  DEADLINE: <2023-05-20 Sat>
```

其中在日历中选择日期时，需要按 Shift+方向键。对于重复性任务如每周例会，则手动加上 "+7d" 表示每 7 天一个周期，具体参见文档：
```text
* MEETING 例会
  SCHEDULED: <2023-05-25 Thu 18:30 +7d>
```

### 快速添加任务

先快速把任务记下来，回头再整理。这个功能叫 org-capture

可将如下配置合并到上面的配置中：

```lisp
;; ...
(setq pv/org-refile-file (concat org-directory "refile.org"))

(use-package org
  :custom
  ;; ...
  (org-capture-templates
   (quote (("t" "todo" entry (file pv/org-refile-file)
            "* TODO %?\n%U\n%a\n" :clock-in t :clock-resume t)
           ("r" "respond" entry (file pv/org-refile-file)
            "* NEXT Respond to %:from on %:subject\nSCHEDULED: %t\n%U\n%a\n" :clock-in t :clock-resume t :immediate-finish t)
           ("n" "note" entry (file pv/org-refile-file)
            "* %? :NOTE:\n%U\n%a\n" :clock-in t :clock-resume t)
           ("w" "org-protocol" entry (file pv/org-refile-file)
            "* TODO Review %c\n%U\n" :immediate-finish t)
           ("m" "Meeting" entry (file pv/org-refile-file)
            "* MEETING with %? :MEETING:\n%U" :clock-in t :clock-resume t))))
  (org-refile-targets (quote ((nil :maxlevel . 9)
                                 (org-agenda-files :maxlevel . 9))))
  ;; Use full outline paths for refile targets - we file directly with IDO
  (org-refile-use-outline-path 'file)
    ;; Allow refile to create parent tasks with confirmation
  (org-refile-allow-creating-parent-nodes (quote confirm))
  (org-cite-global-bibliography pv/org-bibtex-files)
  :bind
  ;; ...
  (("C-c a" . 'org-agenda)
   ("C-c c" . 'org-capture)
   :map org-mode-map
   ("C-c C-q" . counsel-org-tag)))
```

当我们使用 C-c c 调用 org-capture 时，会提示我们想要添加的是什么任务。
按 C-c C-c 保存到 ~/org/refile.org 中，也就是临时存储。
等有空时，我们可以打开 ~/org/refile.org 文件，对着一个任务输入 C-c C-w 调用 org-refile，此时会提示你希望把这个任务插入到哪个文件的哪个项目的位置里。


## 论文笔记

结合以上工具，org-mode 可以对论文进行笔记并关联 BibTeX。
主要配置就是要告知 Emacs 我们的 BibTeX 文件在哪里，并设定一个存放论文 PDF 的仓库。
这里，笔者配置了 ~/org/References 为仓库，~/org/References/reference.bib 为一个总的 BibTeX。
二者都是列表，笔者可以根据需要设置多个仓库和多个 BibTeX 文件。笔者还使用了 org-ref 插件辅助管理。

。。跳





# 模板/快速输入

插件是 yasnippet。

模板路径 `.emacs.d/elpa/yasnippet-snippets-20241207.2221/snippets`

C++注意： `cc-mode`, `c++-mode`, `c-mode`, `c-lang-mode`。








# 21天学会Emacs

https://book.emacs-china.org/

配套的视频: https://www.bilibili.com/video/BV12P4y1j7EL/


## 改键

Windows
使用 SharpKeys:
- 将 左Alt键 和 windows键 交换
- Caps Lock 改成 Ctrl 键

Emacs 默认 Alt键就是 Meta，而 windows键 我们改成 super键，可以通过下面的elisp代码来完成
`(setq w32-apps-modifier 'super)       ; 通过SharpKeys改成了 Application`

---

Mac
只需要将 大小写键 改成 Ctrl键。
其他的可以通过 elisp修改
```lisp
(setq mac-option-modifier 'meta
      mac-command-modifier 'super)
```

---

### 常见符号(M s S C)的含义

Meta， 统一修改到 Mac的option， Win的 windows键
super， 统一改到 Mac 的command， Win 的 左Alt
Shift， 不用修改
Ctrl， 统一改成 Caps Lock键

。。我的Caps Lock 有个灯。。 而且 左Ctrl感觉还可以。。

## 光标的移动

- C-f， 前移一个字符  forward
- C-b， 后移一个字符  backward
- C-p， 上移一行  previous
- C-n， 下移一行  next
- C-a， 移到行首  ahead
- C-e， 移到行尾  end

我们可以把 Mac下的 复制，粘贴，剪切，全选 等命令 移植到 Emacs中
```js
(global-set-key (kbd "s-a") 'mark-whole-buffer) ;;对应Windows上面的Ctrl-a 全选
(global-set-key (kbd "s-c") 'kill-ring-save) ;;对应Windows上面的Ctrl-c 复制
(global-set-key (kbd "s-s") 'save-buffer) ;; 对应Windows上面的Ctrl-s 保存
(global-set-key (kbd "s-v") 'yank) ;对应Windows上面的Ctrl-v 粘贴
(global-set-key (kbd "s-z") 'undo) ;对应Windows上面的Ctrol-z 撤销
(global-set-key (kbd "s-x") 'kill-region) ;对应Windows上面的Ctrol-x 剪切
```


## 2. 打造属于你自己的笔记本

Emacs功能很强大，但是部分功能默认情况下没有开启，比如: 显示行号: `M-x linum-mode`

下面3种方法在学习Emacs的过程中 非常重要
- C-h k， 寻找快捷键的帮助信息
- C-h v， 寻找变量的帮助信息
- C-h f， 寻找函数的帮助信息

---

给windows 的邮件菜单 添加 OpenWithEmacs功能
```text
[HKEY_CLASSES_ROOT\*\shell]
[HKEY_CLASSES_ROOT\*\shell\openwemacs]
@="&Edit with Emacs"
[HKEY_CLASSES_ROOT\*\shell\openwemacs\command]
@="C:\\emax64\\bin\\emacsclientw.exe -n \"%1\""
[HKEY_CLASSES_ROOT\Directory\shell\openwemacs]
@="Edit &with Emacs"
[HKEY_CLASSES_ROOT\Directory\shell\openwemacs\command]
@="C:\\emax64\\bin\\emacsclientw.exe -n \"%1\""
```

要使用这个功能，Emacs需要开启 Server Mode: `(server-mode 1)`



## 3. Elisp基础，Org基础，包管理器

务必完成 https://learnxinyminutes.com/zh-cn/elisp/

elisp 是函数式语言，所以全部功能由 函数来实现

简单的例子
```js
;; 2 + 2
(+ 2 2)

;; 2 + 3 * 4
(+ 2 (* 3 4))

;; 定义变量
(setq name "username")
(message name) ; -> "username"

;; 定义函数
(defun func ()
  (message "Hello, %s" name))

;; 执行函数
(func) ; -> Hello, username

;; 设置快捷键
(global-set-key (kbd "<f1>") 'func)

;; 使函数可直接被调用可添加 (interactive)
(defun func ()
  (interactive)
  (message "Hello, %s" name))
```

---

Emacs 配置文件默认保存在 `~/.emacs.d/init.el` 中。 也可能保存在 `~/.emacs`中，后续会介绍区别
如果 希望把配置放在 init.el中，那么需要 删除 .emacs文件

开始配置之前，先来区别 Major mode 和 Minor mode 的区别。
- Major Mode 通常是 定义对于 一种文件类型的 核心规则， 例如 语法高亮，缩进，快捷键绑定 等。
- Minor Mode 是 Major Mode所提供的核心功能以外的 (辅助)功能。 例如下面提到的 `tool-bar-mode` 和 `linum-mode` 都是 Minor Mode


简单来说就是，一种文件类型同时`只能存在一种 Major Mode` 但是它可以同时激活`一种或多种 Minor Mode`。
如果你希望知道当前的模式信息，可以使用 `C-h m` 来显示当前所有开启 的全部 Minor Mode 的信息。 


### 简单的编辑器自定义

复制下面的内容 到 `init.el` 中

```js
;; 关闭工具栏，tool-bar-mode 即为一个 Minor Mode
(tool-bar-mode -1)

;; 关闭文件滑动控件
(scroll-bar-mode -1)

;; 显示行号
(global-linum-mode 1)

;; 更改光标的样式（不能生效，解决方案见第二集）
(setq cursor-type 'bar)

(icomplete-mode 1)


;; 快速打开配置文件
(defun open-init-file()
  (interactive)
  (find-file "~/.emacs.d/init.el"))

;; 这一行代码，将函数 open-init-file 绑定到 <f2> 键上
(global-set-key (kbd "<f2>") 'open-init-file)
```

需要 重启 编辑器或者 重新加载 配置文件 才能生效。

重新加载 需要在 当前配置文件中，使用 `M-x load-file` 双击2次 回车确认 默认文件名， 或使用 `M-x eval-buffer` 去执行 当前缓冲区中的 所有 Lisp命令。 也可以使用 `C-x C-e` 来执行某一行的Lisp代码。


### 插件管理

使用 默认的插件管理器(菜单栏`Options -> Manage Emacs Packages`) 来安装 Company 插件， 它是一个 代码补全插件。
使用下面的配置 将 Company-mode 在全局激活
```js
;; 开启全局 Company 补全
(global-company-mode 1)

;; company mode 默认选择上一条和下一条候选项命令 M-n M-p
(define-key company-active-map (kbd "C-n") 'company-select-next)
(define-key company-active-map (kbd "C-p") 'company-select-previous)
```


### Org-mode

简单的使用，它可以 列出提纲，并方便地使用 `tab` 键 来对其 进行 展开和关闭。
`C-c C-t` 可以将一个 条目 转换成 一个待办事件

```js
* 为一级标题
** 为二级标题
*** 为三级标题并以此类推
```

## 4. 增强Emacs补全

关于 lexical binding
```js
;;在文件最开头添加地个 文件作用域的变量设置，设置变量的绑定方式
;; -*- lexical-binding: t -*-
(let ((x 1))    ; x is lexically bound.
  (+ x 3))
⇒ 4

(defun getx ()
  x)            ; x is used free in this function.

(let ((x 1))    ; x is lexically bound.
  (getx))
;;error→ Symbol's value as variable is void: x
```
更多细节，阅读 官方文档: https://www.gnu.org/software/emacs/manual/html_node/elisp/Lexical-Binding.html


```js
;; 更改显示字体大小 16pt
;; http://stackoverflow.com/questions/294664/how-to-set-the-font-size-in-emacs
(set-face-attribute 'default nil :height 160);;

;;让鼠标滚动更好用
(setq mouse-wheel-scroll-amount '(1 ((shift) . 1) ((control) . nil)))
(setq mouse-wheel-progressive-speed nil)
```

### 配置gnu 和 melpa镜像

最常用的是 MELPA (Milkypostman's Emacs Lisp Package Archive)， 有 3000+ 插件。
添加源后，就可以使用 `M-x package-list-packages` 来查看 melpa上所有的插件。
在表单中
- i 标记安装
- d 标记删除
- U 更新
- x 确认
- u 撤销标记操作。


复制下面的代码 到 配置文件顶端，从而直接使用 melpa作为 插件源
```js
(require 'package)
(setq package-archives '(("gnu"   . "http://elpa.zilongshanren.com/gnu/")

                         ("melpa" . "http://elpa.zilongshanren.com/melpa/")))
(package-initialize)

;;防止反复调用 package-refresh-contents 会影响加载速度
(when (not package-archive-contents)
  (package-refresh-contents))

;;modeline上显示我的所有的按键和执行的命令
(package-install 'keycast)
(keycast-mode t)
```

。。不知道 为什么 gnu 和 melpa 是 分开的， 主要是 gun前面有 '， 难道这个 ' 是指 到下个括号全部?
。但是视频确实 是分开的。

---

增强 minibuffer 补全: vertico 和 Orderless
```js
(package-install 'vertico)
(vertico-mode t)

(package-install 'orderless)
(setq completion-styles '(orderless))
```

---

增强 minibuffer 的 annotation: marginalia
```js
(package-install 'marginalia)
(marginalia-mode t)
```

---

minibuffer action 和 自适应的 context menu: embark
```js
(package-install 'embark)
(global-set-key (kbd "C-;") 'embark-act)
(setq prefix-help-command 'embark-prefix-help-command)
```

---

增强文件内搜索 和跳转函数定义: consult
```js
(package-install 'consult)
;;replace swiper
(global-set-key (kbd "C-s") 'consult-line)
;;consult-imenu
```



## 5. 手动安装插件和使用外部程序

如果要深入学习 elisp，那么可以看 https://www.gnu.org/software/emacs/manual/html_mono/eintr.html  。 也可以 `M-x info` 然后选择 Emacs Lisp Intro

我们先解决之前的问题:
- 在对象是一个缓冲区局部变量(buffer-local variable) 的时候，比如下面的 `cursor-type`，我们需要区分 `setq` `setq-default`: setq 是 设置 当前buffer中的变量值， setq-default是 全局变量的值。 下面是一个例子，用于设置 光标样式
```js
(setq-default cursor-type 'bar)
(show-paren-mode t)

;;另外一件安装插件的方法
(add-to-list 'load-path (expand-file-name "~/.emacs.d/awesome-tab/"))

(require 'awesome-tab)

(awesome-tab-mode t)

(defun awesome-tab-buffer-groups ()
  "`awesome-tab-buffer-groups' control buffers' group rules.
Group awesome-tab with mode if buffer is derived from `eshell-mode' `emacs-lisp-mode' `dired-mode' `org-mode' `magit-mode'.
All buffer name start with * will group to \"Emacs\".
Other buffer group by `awesome-tab-get-group-name' with project name."
  (list
   (cond
    ((or (string-equal "*" (substring (buffer-name) 0 1))
         (memq major-mode '(magit-process-mode
                            magit-status-mode
                            magit-diff-mode
                            magit-log-mode
                            magit-file-mode
                            magit-blob-mode
                            magit-blame-mode)))
     "Emacs")
    ((derived-mode-p 'eshell-mode)
     "EShell")
    ((derived-mode-p 'dired-mode)
     "Dired")
    ((memq major-mode '(org-mode org-agenda-mode diary-mode))
     "OrgMode")
    ((derived-mode-p 'eaf-mode)
     "EAF")
    (t
     (awesome-tab-get-group-name (current-buffer))))))
```


- 其次 它使用到了 `quote`，就是之前遇到的 `'` 。 因为它非常常用，所以有 简写的方法
```js
;; 下面两行的效果完全相同的
(quote foo)
'foo
```
`quote` 的意思是 不要执行后面的内容，返回 它本身
```js
(print '(+ 1 1)) ;; -> (+ 1 1)
(print (+ 1 1))  ;; -> 2
```

---

通常 我们的配置文件 和 项目文件 都适用 版本控制系统。所以 自动生成 备份文件 就有点多余。
我们可以禁止 emacs 自动生成备份文件，例如 `init.el~` , `~` 是自动生成的文件 的后缀。
`(setq make-backup-files nil)`

---

使用下面的配置来加入最近打开过的文件的选项 让我们更快捷地在GUI的菜单中 打开最近编辑过的文件
```js
(require 'recentf)
(recentf-mode 1)
(setq recentf-max-menu-item 10)

;; 这个快捷键绑定可以用之后的插件 counsel 代替
;; (global-set-key (kbd "C-x C-r") 'recentf-open-files)
```

`require` 的意思 是从 文件加载特性

---

使用下面的配置文件 将删除功能 配置成 和其他GUI的编辑器一样， 即当你选中一段文字后 输入一个 字符会 替换掉 你选中的文字
`(delete-selection-mode 1)`

---

下面的函数可以让你 找到 不同函数，变量，快捷键 所定义的文件位置
- `C-h C-f` , function
- `C-h C-v` , variable
- `C-h C-k` , key

---

使用外部命令行工具

下面安装 emax， 配置 emacs加载路径

。。emax 是 Emacs的 非官方的 windows版

```js
(progn
  (defvar emax-root (concat (expand-file-name "~") "/emax"))
  (defvar emax-bin (concat emax-root "/bin"))
  (defvar emax-bin64 (concat emax-root "/bin64"))

  (setq exec-path (cons emax-bin exec-path))
  (setenv "PATH" (concat emax-bin ";" (getenv "PATH")))

  (setq exec-path (cons emax-bin64 exec-path))
  (setenv "PATH" (concat emax-bin64 ";" (getenv "PATH")))

  (setq emacsd-bin (concat user-emacs-directory "bin"))
  (setq exec-path (cons  emacsd-bin exec-path))
  (setenv "PATH" (concat emacsd-bin  ";" (getenv "PATH")))

  ;;可选安装msys64
  ;;下载地址: http://repo.msys2.org/mingw/sources/
  (setenv "PATH" (concat "C:\\msys64\\usr\\bin;C:\\msys64\\mingw64\\bin;" (getenv "PATH")))

  ;; (dolist (dir '("~/emax/" "~/emax/bin/" "~/emax/bin64/" "~/emax/lisp/" "~/emax/elpa/"))
  ;;   (add-to-list 'load-path dir))
  )
```


使用 `M-x Shell` 来学习命令行操作。
。。这个是 模拟shell的buffer


## 6. Emacs作为超级前端

使用 Emacs打开文件管理器

。。这个估计要看视频，不知道用来干嘛的， 而且应该是 windows的

```js
(shell-command-to-string "explorer.exe C:\\")

(shell-command-to-string "explorer.exe ~/.emacs.d")

(shell-command-to-string
 (encode-coding-string
  (replace-regexp-in-string "/" "\\\\"
                            (format "explorer.exe %s" (expand-file-name "~/.emacs.d")))
  'gbk))

(defun consult-directory-externally (file)
  "Open FILE externally using the default application of the system."
  (interactive "fOpen externally: ")
  (if (and (eq system-type 'windows-nt)
           (fboundp 'w32-shell-execute))
      (shell-command-to-string (encode-coding-string (replace-regexp-in-string "/" "\\\\" (format "explorer.exe %s" (file-name-directory (expand-file-name file)))) 'gbk))
    (call-process (pcase system-type
                    ('darwin "open")
                    ('cygwin "cygstart")
                    (_ "xdg-open"))
                  nil 0 nil
                  (file-name-directory (expand-file-name file)))))

(define-key embark-file-map (kbd "E") #'consult-directory-externally)
;;打开当前文件的目录
(defun my-open-current-directory ()
  (interactive)
  (consult-directory-externally default-directory))
```

---

批量搜索替换: embark，consult

```js
(package-install 'embark-consult)
(package-install 'wgrep)
(setq wgrep-auto-save-buffer t)

(eval-after-load
    'consult
  '(eval-after-load
       'embark
     '(progn
        (require 'embark-consult)
        (add-hook
         'embark-collect-mode-hook
         #'consult-preview-at-point-mode))))

(define-key minibuffer-local-map (kbd "C-c C-e") 'embark-export-write)

;;使用ripgrep来进行搜索
;;consult-ripgrep

;;everyting
;;consult-locate
;; 配置搜索中文
(progn
  (setq consult-locate-args (encode-coding-string "es.exe -i -p -r" 'gbk))
  (add-to-list 'process-coding-system-alist '("es" gbk . gbk))
  )
(eval-after-load 'consult
  (progn
    (setq
     consult-narrow-key "<"
     consult-line-numbers-widen t
     consult-async-min-input 2
     consult-async-refresh-delay  0.15
     consult-async-input-throttle 0.2
     consult-async-input-debounce 0.1)
    ))
```

---

使用拼音进行搜索

```js
(package-install 'pyim)

(defun eh-orderless-regexp (orig_func component)
  (let ((result (funcall orig_func component)))
    (pyim-cregexp-build result)))


(defun toggle-chinese-search ()
  (interactive)
  (if (not (advice-member-p #'eh-orderless-regexp 'orderless-regexp))
      (advice-add 'orderless-regexp :around #'eh-orderless-regexp)
    (advice-remove 'orderless-regexp #'eh-orderless-regexp)))

(defun disable-py-search (&optional args)
  (if (advice-member-p #'eh-orderless-regexp 'orderless-regexp)
      (advice-remove 'orderless-regexp #'eh-orderless-regexp)))

;; (advice-add 'exit-minibuffer :after #'disable-py-search)
(add-hook 'minibuffer-exit-hook 'disable-py-search)

(global-set-key (kbd "s-p") 'toggle-chinese-search)
```

---

高亮当前行
`(global-hl-line-mode 1)`

---

安装主题
`(package-install 'monokai-theme)`

打开编辑器时 加载主题
`(load-theme 'monokai 1)`

---

使用 `M-x customize-group` 后选择 对应的插件名称，可以进入 可视化选项区 对 指定插件 做自定义设置。  选择 save for future session 后， 刚做的配置 会被保存到 init.el 中。

各个插件的 安装 和 使用，通常 可以在 官方页面 (github 或 项目中的 README.md) 找到。


## 7. 模块化配置文件管理

使用多个文件 存储 配置文件

不同配置 放到不同文件中，使其模块化，让后续维护更简单。
下面是 目前的 `~/.emacs.d/` 中的目录
```shell
├── auto-save-list # 自动生成的保存数据
├── elpa           # 下载的插件目录
├── init.el        # 我们的配置文件
└── recentf        # 最近访问的文件列表
```

通常我们只保存 配置文件，并对其 进行版本控制， 其他的插件 在第一次使用 编辑器时 通过网络下载， 当然你也可以 将全部配置文件 版本控制 来保证 最稳定的 环境

---

`custom.el`
```js
(setq custom-file (expand-file-name "~/.emacs.d/custom.el"))
(load custom-file 'no-error 'no-message)
```

现在，我们希望将配置文件分为下面几个模块
```js
init-packages.el        # 插件管理
init-ui.el              # 视觉层配置
init-better-defaults.el # 增强内置功能
init-keybindings.el     # 快捷键绑定
init-org.el             # Org 模式相关的全部设定
custom.el              # 存放使用编辑器接口产生的配置信息
```

目录结构
```shell
├── init.el
└── lisp
    ├── custom.el
    ├── init-better-defaults.el
    ├── init-keybindings.el
    ├── init-packages.el
    ├── init-ui.el
    └── init-org.el
```

`init.el` 是配置文件的入口， 所以 当我们需要使用这些模块时， 需要在 init.el 中 引用 需要加载的模块。
下面以 `init-packages.el` 为例
`~/.emacs.d/lisp/init-packages.el` 的内容
```js
(require 'package)
(setq package-archives '(("gnu"   . "http://elpa.zilongshanren.com/gnu/")

                         ("melpa" . "http://elpa.zilongshanren.com/melpa/")))
(package-initialize)

;;防止反复调用 package-refresh-contents 会影响加载速度
(when (not package-archive-contents)
  (package-refresh-contents))

;; 文件末尾
(provide 'init-packages)
```

`~/.emacs.d/init.el`中
```js
(add-to-list 'load-path "~/.emacs.d/lisp/")

;; Package Management
;; -----------------------------------------------------------------
(require 'init-packages)
```

模块化很简单
1. 将某部分配置文件移动到 独立的文件， 并在末尾 添加 `(provide '模块名)`
2. 在入口文件中 添加该文件，并且 引用配置


推荐2个配置
- https://github.com/condy0919/.emacs.d
- https://github.com/seagle0128/.emacs.d 


### 使用Org来管理配置文件

org-mode中，你可以直接开启新的 buffer 直接用 相应的 Major Mode 来 编辑代码块中的内容。
在代码块中 `C-c '` 会直接打开 对应模式的缓冲区。

使用 `<s` 然后 tab 可以直接插入代码块的代码片段。
```js
(require 'org-tempo)  ;开启easy template

;; 禁用左尖括号
(setq electric-pair-inhibit-predicate
      `(lambda (c)
         (if (char-equal c ?\<) t (,electric-pair-inhibit-predicate c))))

(add-hook 'org-mode-hook
          (lambda ()
            (setq-local electric-pair-inhibit-predicate
                        `(lambda (c)
                           (if (char-equal c ?\<) t (,electric-pair-inhibit-predicate c))))))
```

将下面的代码放入 init.el中
```js
(require 'org-install)
(require 'ob-tangle)
(org-babel-load-file (expand-file-name "zilongshanren.org" user-emacs-directory))
```

之后，我们只需要将 所有配置文件放入 Org模式的 代码块中即可， 并使用 目录结构来表述你的 配置文件， 再把它 保存在 与 入口文件相同的目录中 即可 (文件名为 `org-file-name.org`)
emacs会提取其中的配置 并使其生效。

使用org-mode 进行管理的 范例
https://github.com/trev-dev/emacs



## 8. macro和use-package

更好的默认设置

下面的代码可以使 emacs自动加载外部修改过的文件
`(global-auto-revert-mode 1)`

下面 可以关闭自动保存文件。 (之前是关闭 自动备份)
`(setq auto-save-default nil)`

关闭警告音
`(setq ring-bell-function 'ignore)`

对yes 和no 取别名，这样就只需要输入 y 或 n
`(fset 'yes-or-no-p 'y-or-n-p)`


### Macro

什么是宏

```js
(setq my-var 1)
(setq my-var (1+ my-var))

(defmacro inc (var)
  (list 'setq var (list '1+ var)))

(macroexpand '(inc my-var))

(inc my-var)

;;; pass by value
(defun inc-2 (var)
  (setq var (1+ var)))


(inc-2 my-var)

;; Backquote allows you to:

;; quote a list, and
;; selectively evaluate elements of the list (with comma ,), or:
;; splice (eval & spread) the element with ,@

`(a list of ,(+ 2 3) elements)

(setq some-list '(2 3))

`(1 ,@some-list 4 ,@some-list)
```

### use-package

安装
`(package-install 'use-package) `

use-package 是一个宏，它能让你 将一个包的 require 和 它的相关的初始化等配置 组织在一起， 避免 同一个包的 配置 散落在不同文件中

更多信息: https://github.com/jwiegley/use-package

---

一些简单用法

更安全的 require
emacs中，引入包，通常使用
`(require 'package-name)`

但是，当 package-name 不在 load-path 中时，会报错。
使用 use-package 可以避免
`(use-package package-name)`
上面的代码会扩展为
```js
(if
    (not
     (require 'package-name nil 't))
    (ignore
     (message
      (format "Cannot load %s" 'package-name))))
```
use-package 使用ignore 来避免抛出错误，这样 ，当包不存在时， emacs 也能正常启动。

---

将配置集中
当引入包时，可能需要 定义一些与这个包 相关的变量
```js
(use-package package-name
  :init
  (setq my-var1 "xxx")
  :config
  (progn
    (setq my-var2 "xxx")
    (setq my-var3 "xxx")
    )
  )
```
init 后的代码 会在 包的require 之前执行，如果这段代码出错，那么跳过 包的 require。
config 后的代码 在 包的 require之后执行。

init，config之后 只能 单个表达式， 如果需要多个语句，那么使用 `progn`

---

autoload
需要时才加载
```js
(use-package package-name
  :commands
  (global-company-mode)
  :defer t
  )
```
commands 让 package 延迟加载。
defer 也可以让 package-name 延迟加载。

---

键绑定
之前，通过 `global-key-bind` 或 `define-key` 来 绑定键

```js
(use-package company
  :bind (:map company-active-map
              ("C-n" . 'company-select-next)
              ("C-p" . 'company-select-previous))
  :init
  (global-company-mode t)
  :config
  (setq company-minimum-prefix-length 1)
  (setq company-idle-delay 0))
```

---

use-package 的好处
- 让 相关配置更集中，避免 分散导致的 维护困难
- 完善的错误处理


## 9. use-package更多设置

安装restart-emacs
```js
(use-package restart-emacs
  :ensure t)

;;emacs --debug-init
```
删除 company 这个包，看看会出现什么

---

### 自动安装

```js
(require 'use-package-ensure)
(setq use-package-always-ensure t)

;; Setup `use-package'
(unless (package-installed-p 'use-package)
  (package-refresh-contents)
  (package-install 'use-package))
```
这个选项会 自动安装包，但是不会自动更新包 到最新版本

`[]` 清理所有的 package-install

---

### 保存光标历史，记住上个命令

```js
(use-package savehist
  :ensure nil
  :hook (after-init . savehist-mode)
  :init (setq enable-recursive-minibuffers t ; Allow commands in minibuffers
              history-length 1000
              savehist-additional-variables '(mark-ring
                                              global-mark-ring
                                              search-ring
                                              regexp-search-ring
                                              extended-command-history)
              savehist-autosave-interval 300)
  )

(use-package saveplace
  :ensure nil
  :hook (after-init . save-place-mode))
```

### 显示文件行，列，大小，美化状态栏

```js
(use-package simple
  :ensure nil
  :hook (after-init . size-indication-mode)
  :init
  (progn
    (setq column-number-mode t)
    ))

;;modeline上显示我的所有的按键和执行的命令
(package-install 'keycast)
(add-to-list 'global-mode-string '("" keycast-mode-line))
(keycast-mode t)

;; 这里的执行顺序非常重要，doom-modeline-mode 的激活时机一定要在设置global-mode-string 之后‘
(use-package doom-modeline
  :ensure t

  :init
  (doom-modeline-mode t))
```


## 10. orgmode基础

查看orgmode版本号: `C-h v org-version`

安装最新版
```js
;; 安装org 之前，一定要配置 use-package 不使用内置的org 版本，可以使用本段代码最后面的代码，具体位置可以参考视频
(use-package org
  :pin melpa
  :ensure t)

(use-package org-contrib
  :pin nongnu
  )

;; 添加新的 nongnu 的源
("nongnu" . "http://elpa.zilongshanren.com/nongnu/")

;;; 这个配置一定要配置在 use-package 的初始化之前，否则无法正常安装
(assq-delete-all 'org package--builtins)
(assq-delete-all 'org package--builtin-versions)
```

---

### org todo

```js
(setq org-todo-keywords
      (quote ((sequence "TODO(t)" "STARTED(s)" "|" "DONE(d!/!)")
              (sequence "WAITING(w@/!)" "SOMEDAY(S)" "|" "CANCELLED(c@/!)" "MEETING(m)" "PHONE(p)"))))

;;fix doom modeline
:custom-face
(mode-line ((t (:height 0.9))))
(mode-line-inactive ((t (:height 0.9))))

(require 'org-checklist)
;; need repeat task and properties
(setq org-log-done t)
(setq org-log-into-drawer t)
```

### org agenda

```js
;; C-c C-s schedule
;; C-c C-d deadline
(global-set-key (kbd "C-c a") 'org-agenda)
(setq org-agenda-files '("~/gtd.org"))
(setq org-agenda-span 'day)
```


### org capture

```js
(setq org-capture-templates
      '(("t" "Todo" entry (file+headline "~/gtd.org" "Workspace")
         "* TODO [#B] %?\n  %i\n %U"
         :empty-lines 1)))

(global-set-key (kbd "C-c r") 'org-capture)
```

### org effect

在 agenda view中，`e` 键可以设置 effort， `_` 可以过滤指定 effort 的 heading

### org tags

agent view中，`:` 按照 tag名称来过滤 todo 的 heading  (这里的 快捷键可以通过 `M-x` 来查看)

### org priority

agenda view中
```js
(setq org-agenda-custom-commands
      '(("c" "重要且紧急的事"
         ((tags-todo "+PRIORITY=\"A\"")))
        ;; ...other commands here
        ))
```

---

参考: https://www.cnblogs.com/Open_Source/archive/2011/07/17/2108747.html



## 11. 使用 ox-huge 写博客

。。

## 12. 使用eglot 编写，运行，调试 C/C++ 代码

安装 Emacs29
显示行号:
```js
                                        ;  (global-linum-mode 1)
(global-display-line-numbers-mode t)    ;修改成这个来显示行号，性能更好
```

安装 mysys2

下载 https://www.msys2.org/

安装 C C++必要工具
```js
pacman -Syu
pacman -S mingw-w64-x86_64-gcc mingw-w64-x86_64-gdb make mingw-w64-x86_64-clang mingw-w64-x86_64-clang-tools-extra
```
。。windows 的。。

设置环境变量
```js
C:\msys64\mingw64\bin
C:\msys64\usr\bin
```

安装eglot (Emacs29 自带eglot)
```js
(require 'eglot)
(add-to-list 'eglot-server-programs '((c++-mode c-mode) "clangd"))
(add-hook 'c-mode-hook #'eglot-ensure)
(add-hook 'c++-mode-hook #'eglot-ensure)
```

---

### 一键运行代码

安装 quickrun

```js
(use-package quickrun
  :ensure t
  :commands (quickrun)
  :init
  (quickrun-add-command "c++/c1z"
    '((:command . "g++")
      (:exec . ("%c -std=c++1z %o -o %e %s"
                "%e %a"))
      (:remove . ("%e")))
    :default "c++"))
```

自定义快捷键
`(global-set-key (kbd "<f5>") 'quickrun)`


---

### 使用GDB调试

运行 `M-x compile`, 输入  `g++ -g -o test.o test.cpp`
`M-x gud-gdb`, 输入 `gdb ./test.o`

常用调试命令
|命令|功能|
|--|--|
|list|显示源码|
|break|增加断点: break main, break 12(行号)|
|info|查看断点 或 局部变量信息: info breakpoints，info locals|
|run|开始调试|
|next|类似step over|
|step|跳转到函数内部|
|continue|继续运行到下一个断点|
|quit|退出调试|
|watch|内存断点|
|display|类似IDE中的watch功能|
|break 11 if xxx|条件断点|


## 13. 使用Evil: vim按键模拟

。。


## 14. packages推荐

some minor fix
```js
;; make c-j/c-k work in vertico selection
(define-key vertico-map (kbd "C-j") 'vertico-next)
(define-key vertico-map (kbd "C-k") 'vertico-previous)

;; make consult-ripgrep work
(add-to-list 'process-coding-system-alist 
             '("[rR][gG]" . (utf-8-dos . windows-1251-dos)))
```

---

多光标操作 iedit 和 evil-multiedit

```js
(use-package iedit
  :ensure t
  :init
  (setq iedit-toggle-key-default nil)
  :config
  (define-key iedit-mode-keymap (kbd "M-h") 'iedit-restrict-function)
  (define-key iedit-mode-keymap (kbd "M-i") 'iedit-restrict-current-line))

(use-package evil-multiedit
  :ensure t
  :commands (evil-multiedit-default-keybinds)
  :init
  (evil-multiedit-default-keybinds))
```

---

expand-region
```js
(use-package expand-region
  :config
  (defadvice er/prepare-for-more-expansions-internal
      (around helm-ag/prepare-for-more-expansions-internal activate)
    ad-do-it
    (let ((new-msg (concat (car ad-return-value)
                           ", H to highlight in buffers"
                           ", / to search in project, "
                           "e iedit mode in functions"
                           "f to search in files, "
                           "b to search in opened buffers"))
          (new-bindings (cdr ad-return-value)))
      (cl-pushnew
       '("H" (lambda ()
               (interactive)
               (call-interactively
                'zilongshanren/highlight-dwim)))
       new-bindings)
      (cl-pushnew
       '("/" (lambda ()
               (interactive)
               (call-interactively
                'my/search-project-for-symbol-at-point)))
       new-bindings)
      (cl-pushnew
       '("e" (lambda ()
               (interactive)
               (call-interactively
                'evil-multiedit-match-all)))
       new-bindings)
      (cl-pushnew
       '("f" (lambda ()
               (interactive)
               (call-interactively
                'find-file)))
       new-bindings)
      (cl-pushnew
       '("b" (lambda ()
               (interactive)
               (call-interactively
                'consult-line)))
       new-bindings)
      (setq ad-return-value (cons new-msg new-bindings)))))
```

添加快捷键，让标记和搜索功能更方便

```js
;;;###autoload
(defun my/search-project-for-symbol-at-point ()
  (interactive)
  (if (use-region-p)
      (progn
        (consult-ripgrep (project-root (project-current))
                         (buffer-substring (region-beginning) (region-end))))))

(global-definer
  "hc" 'zilongshanren/clearn-highlight
  "hH" 'zilongshanren/highlight-dwim
  "v" 'er/expand-region
  )
```

---

interactive replace

```js
(defun zilongshanren/evil-quick-replace (beg end )
  (interactive "r")
  (when (evil-visual-state-p)
    (evil-exit-visual-state)
    (let ((selection (regexp-quote (buffer-substring-no-properties beg end))))
      (setq command-string (format "%%s /%s//g" selection))
      (minibuffer-with-setup-hook
          (lambda () (backward-char 2))
        (evil-ex command-string)))))

(define-key evil-visual-state-map (kbd "C-r") 'zilongshanren/evil-quick-replace)
```

---

安装 quelpa 插件

```js
(use-package quelpa)

(unless (package-installed-p 'quelpa-use-package)
  (quelpa
   '(quelpa-use-package
     :fetcher git
     :url "https://github.com/quelpa/quelpa-use-package.git")))

(use-package quelpa-use-package
  :init
  (setq quelpa-use-package-inhibit-loading-quelpa t)
  :demand t)
```

---

symbol-overlay ＆ highlight-global

```js
(defun zilongshanren/highlight-dwim ()
  (interactive)
  (if (use-region-p)
      (progn
        (highlight-frame-toggle)
        (deactivate-mark))
    (symbol-overlay-put)))

(defun zilongshanren/clearn-highlight ()
  (interactive)
  (clear-highlight-frame)
  (symbol-overlay-remove-all))

(use-package symbol-overlay
  :config
  (define-key symbol-overlay-map (kbd "h") 'nil))

(use-package highlight-global
  :ensure nil
  :commands (highlight-frame-toggle)
  :quelpa (highlight-global :fetcher github :repo "glen-dai/highlight-global")
  :config
  (progn
    (setq-default highlight-faces
                  '(('hi-red-b . 0)
                    ('hi-aquamarine . 0)
                    ('hi-pink . 0)
                    ('hi-blue-b . 0)))))
```


## 15. 使用Treesit + wglot 打造现代编程IDE

安装 Emacs29

安装 treesit-auto插件
https://github.com/renzmann/treesit-auto
```js
(use-package treesit-auto
  :demand t
  :config
  (setq treesit-auto-install 'prompt)
  (global-treesit-auto-mode))
```

配置 fontlock level
`(setq treesit-font-lock-level 4)`


跳转函数列表 consult-imenu
```js
(+general-global-menu! "search" "s"
  "j" 'consult-imenu
  "p" 'consult-ripgrep
  "k" 'consult-keep-lines
  "f" 'consult-focus-lines)
```

---

查找定义和引用
使用 ctrl-o 返回
```js
xref-find-references
xref-find-definitions
```

---

添加 snippets 支持
```js
(use-package yasnippet
  :ensure t
  :hook ((prog-mode . yas-minor-mode)
         (org-mode . yas-minor-mode))
  :init
  :config
  (progn
    (setq hippie-expand-try-functions-list
          '(yas/hippie-try-expand
            try-complete-file-name-partially
            try-expand-all-abbrevs
            try-expand-dabbrev
            try-expand-dabbrev-all-buffers
            try-expand-dabbrev-from-kill
            try-complete-lisp-symbol-partially
            try-complete-lisp-symbol))))

(use-package yasnippet-snippets
  :ensure t
  :after yasnippet)
```

---

在头文件和 源文件之间跳转
`ff-find-related-file`


## 16. 窗口管理

Evil的窗口选择操作
`C-w h/j/k/l`

推荐插件
window numbering
```js
(use-package window-numbering
  :init
  :hook (after-init . window-numbering-mode))
```

es-windows
```js
(use-package es-windows
  :ensure t)
```

buffer move
```js
(use-package buffer-move
  :ensure t)
```

resize windows
```js
(use-package resize-window
  :ensure t
  :init
  (defvar resize-window-dispatch-alist
    '((?n resize-window--enlarge-down " Resize - Expand down" t)
      (?p resize-window--enlarge-up " Resize - Expand up" t)
      (?f resize-window--enlarge-horizontally " Resize - horizontally" t)
      (?b resize-window--shrink-horizontally " Resize - shrink horizontally" t)
      (?r resize-window--reset-windows " Resize - reset window layout" nil)
      (?w resize-window--cycle-window-positive " Resize - cycle window" nil)
      (?W resize-window--cycle-window-negative " Resize - cycle window" nil)
      (?2 split-window-below " Split window horizontally" nil)
      (?3 split-window-right " Slit window vertically" nil)
      (?0 resize-window--delete-window " Delete window" nil)
      (?K resize-window--kill-other-windows " Kill other windows (save state)" nil)
      (?y resize-window--restore-windows " (when state) Restore window configuration" nil)
      (?? resize-window--display-menu " Resize - display menu" nil))
    "List of actions for `resize-window-dispatch-default.
Main data structure of the dispatcher with the form:
\(char function documentation match-capitals\)"))
```

winner(builtin)
```js
(use-package winner
  :ensure nil
  :commands (winner-undo winner-redo)
  :hook (after-init . winner-mode)
  :init (setq winner-boring-buffers '("*Completions*"
                                      "*Compile-Log*"
                                      "*inferior-lisp*"
                                      "*Fuzzy Completions*"
                                      "*Apropos*"
                                      "*Help*"
                                      "*cvs*"
                                      "*Buffer List*"
                                      "*Ibuffer*"
                                      "*esh command on file*")))
```


popper
```js
(use-package popper
  :defines popper-echo-dispatch-actions
  :commands popper-group-by-directory
  :bind (:map popper-mode-map
              ("s-`" . popper-toggle-latest)
              ("s-o"   . popper-cycle)
              ("M-`" . popper-toggle-type))
  :hook (emacs-startup . popper-mode)
  :init
  (setq popper-reference-buffers
        '("\\*Messages\\*"
          "Output\\*$" "\\*Pp Eval Output\\*$"
          "\\*Compile-Log\\*"
          "\\*Completions\\*"
          "\\*Warnings\\*"
          "\\*Flymake diagnostics.*\\*"
          "\\*Async Shell Command\\*"
          "\\*Apropos\\*"
          "\\*Backtrace\\*"
          "\\*prodigy\\*"
          "\\*Calendar\\*"
          "\\*Embark Actions\\*"
          "\\*Finder\\*"
          "\\*Kill Ring\\*"
          "\\*Embark Export:.*\\*"
          "\\*Edit Annotation.*\\*"
          "\\*Flutter\\*"
          bookmark-bmenu-mode
          lsp-bridge-ref-mode
          comint-mode
          compilation-mode
          help-mode helpful-mode
          tabulated-list-mode
          Buffer-menu-mode
          occur-mode
          gnus-article-mode devdocs-mode
          grep-mode occur-mode rg-mode deadgrep-mode ag-mode pt-mode
          ivy-occur-mode ivy-occur-grep-mode
          process-menu-mode list-environment-mode cargo-process-mode
          youdao-dictionary-mode osx-dictionary-mode fanyi-mode

          "^\\*eshell.*\\*.*$" eshell-mode
          "^\\*shell.*\\*.*$"  shell-mode
          "^\\*terminal.*\\*.*$" term-mode
          "^\\*vterm.*\\*.*$"  vterm-mode

          "\\*DAP Templates\\*$" dap-server-log-mode
          "\\*ELP Profiling Restuls\\*" profiler-report-mode
          "\\*Flycheck errors\\*$" " \\*Flycheck checker\\*$"
          "\\*Paradox Report\\*$" "\\*package update results\\*$" "\\*Package-Lint\\*$"
          "\\*[Wo]*Man.*\\*$"
          "\\*ert\\*$" overseer-buffer-mode
          "\\*gud-debug\\*$"
          "\\*lsp-help\\*$" "\\*lsp session\\*$"
          "\\*quickrun\\*$"
          "\\*tldr\\*$"
          "\\*vc-.*\\*$"
          "\\*eldoc\\*"
          "^\\*elfeed-entry\\*$"
          "^\\*macro expansion\\**"

          "\\*Agenda Commands\\*" "\\*Org Select\\*" "\\*Capture\\*" "^CAPTURE-.*\\.org*"
          "\\*Gofmt Errors\\*$" "\\*Go Test\\*$" godoc-mode
          "\\*docker-containers\\*" "\\*docker-images\\*" "\\*docker-networks\\*" "\\*docker-volumes\\*"
          "\\*prolog\\*" inferior-python-mode inf-ruby-mode swift-repl-mode
          "\\*rustfmt\\*$" rustic-compilation-mode rustic-cargo-clippy-mode
          rustic-cargo-outdated-mode rustic-cargo-test-moed))

  (when (display-grayscale-p)
    (setq popper-mode-line
          '(:eval
            (concat
             (propertize " " 'face 'mode-line-emphasis)
             (all-the-icons-octicon "pin" :height 0.9 :v-adjust 0.0 :face 'mode-line-emphasis)
             (propertize " " 'face 'mode-line-emphasis)))))

  (setq popper-echo-dispatch-actions t)
  (setq popper-group-function nil)
  :config
  (popper-echo-mode 1)

  (with-no-warnings
    (defun my-popper-fit-window-height (win)
      "Determine the height of popup window WIN by fitting it to the buffer's content."
      (fit-window-to-buffer
       win
       (floor (frame-height) 3)
       (floor (frame-height) 3)))
    (setq popper-window-height #'my-popper-fit-window-height)

    (defun popper-close-window-hack (&rest _)
      "Close popper window via `C-g'."
      ;; `C-g' can deactivate region
      (when (and (called-interactively-p 'interactive)
                 (not (region-active-p))
                 popper-open-popup-alist)
        (let ((window (caar popper-open-popup-alist)))
          (when (window-live-p window)
            (delete-window window)))))
    (advice-add #'keyboard-quit :before #'popper-close-window-hack)))
```


按键绑定
```js
(global-definer
  ;; 这里是其他的快捷键
  "0" 'select-window-0
  "1" 'select-window-1
  "2" 'select-window-2
  "3" 'select-window-3
  "4" 'select-window-4
  "5" 'select-window-5)

(+general-global-menu! "window" "w"
  "/" 'split-window-right
  "-" 'split-window-below
  "m" 'delete-other-windows
  "u" 'winner-undo
  "z" 'winner-redo
  "w" 'esw/select-window
  "s" 'esw/swap-two-windows
  "d" 'esw/delete-window
  "=" 'balance-windows-area
  "r" 'esw/move-window
  "x" 'resize-window
  "H" 'buf-move-left
  "L" 'buf-move-right
  "J" 'buf-move-down
  "K" 'buf-move-up)
```


## 17. 工作区间管理

。。都是配置，要看效果，需要看 视频

使用内置的 tab-bar-mode
```js
(use-package tab-bar
  :ensure nil
  :init
  (tab-bar-mode t)
  (setq tab-bar-new-tab-choice "*scratch*") ;; buffer to show in new tabs
  (setq tab-bar-close-button-show nil)      ;; hide tab close / X button
  (setq tab-bar-show 1)                     ;; hide bar if <= 1 tabs open
  (setq tab-bar-format '(tab-bar-format-tabs tab-bar-separator))

  (custom-set-faces
   '(tab-bar ((t (:inherit mode-line))))
   '(tab-bar-tab ((t (:inherit mode-line :foreground "#993644"))))
   '(tab-bar-tab-inactive ((t (:inherit mode-line-inactive :foreground "black")))))

  (defvar ct/circle-numbers-alist
    '((0 . "⓪")
      (1 . "①")
      (2 . "②")
      (3 . "③")
      (4 . "④")
      (5 . "⑤")
      (6 . "⑥")
      (7 . "⑦")
      (8 . "⑧")
      (9 . "⑨"))
    "Alist of integers to strings of circled unicode numbers.")

  (defun ct/tab-bar-tab-name-format-default (tab i)
    (let ((current-p (eq (car tab) 'current-tab))
          (tab-num (if (and tab-bar-tab-hints (< i 10))
                       (alist-get i ct/circle-numbers-alist) "")))
      (propertize
       (concat tab-num
               " "
               (alist-get 'name tab)
               (or (and tab-bar-close-button-show
                        (not (eq tab-bar-close-button-show
                                 (if current-p 'non-selected 'selected)))
                        tab-bar-close-button)
                   "")
               " ")
       'face (funcall tab-bar-tab-face-function tab))))
  (setq tab-bar-tab-name-format-function #'ct/tab-bar-tab-name-format-default)
  (setq tab-bar-tab-hints t))
```


工作区间
```js
(use-package tabspaces
  ;; use this next line only if you also use straight, otherwise ignore it.
  :hook (after-init . tabspaces-mode) ;; use this only if you want the minor-mode loaded at startup.
  :commands (tabspaces-switch-or-create-workspace
             tabspaces-open-or-create-project-and-workspace)
  :custom
  (tabspaces-use-filtered-buffers-as-default t)
  (tabspaces-default-tab "Default")
  (tabspaces-remove-to-default t)
  (tabspaces-include-buffers '("*scratch*"))
  ;; maybe slow
  (tabspaces-session t)
  (tabspaces-session-auto-restore t)
  :config
  ;; Filter Buffers for Consult-Buffer

  (with-eval-after-load 'consult
    ;; hide full buffer list (still available with "b" prefix)
    (consult-customize consult--source-buffer :hidden nil :default nil)
    ;; set consult-workspace buffer list
    (defvar consult--source-workspace
      (list :name "Workspace Buffers"
            :narrow ?w
            :history 'buffer-name-history
            :category 'buffer
            :state #'consult--buffer-state
            :default t
            :items (lambda () (consult--buffer-query
                               :predicate #'tabspaces--local-buffer-p
                               :sort 'visibility
                               :as #'buffer-name)))

      "Set workspace buffer list for consult-buffer.")
    (add-to-list 'consult-buffer-sources 'consult--source-workspace)))
```

按键绑定
```js
(+general-global-menu! "layout" "l"
  "l" 'tabspaces-switch-or-create-workspace
  "L" 'tabspaces-restore-session
  "p" 'tabspaces-open-or-create-project-and-workspace
  "f" 'tabspaces-project-switch-project-open-file
  "s" 'tabspaces-save-session
  "B" 'tabspaces-switch-buffer-and-tab
  "b" 'tabspaces-switch-to-buffer
  "R" 'tab-rename
  "TAB" 'tab-bar-switch-to-recent-tab
  "r" 'tabspaces-remove-current-buffer
  "k" 'tabspaces-close-workspace)
```



## 18. org-mode进阶

org-download
```js
(use-package org-download
    :ensure t
    :demand t
    :after org
    :config
    (add-hook 'dired-mode-hook 'org-download-enable)
    (setq org-download-screenshot-method "powershell -c Add-Type -AssemblyName System.Windows.Forms;$image = [Windows.Forms.Clipboard]::GetImage();$image.Save('%s', [System.Drawing.Imaging.ImageFormat]::Png)")
    (defun org-download-annotate-default (link)
      "Annotate LINK with the time of download."
      (make-string 0 ?\s))

    (setq-default org-download-heading-lvl nil
                  org-download-image-dir "./img"
                  ;; org-download-screenshot-method "screencapture -i %s"
                  org-download-screenshot-file (expand-file-name "screenshot.jpg" temporary-file-directory)))
```

选择图片尺寸
`#+ATTR_HTML: :width 1000px`

---

Org Protocol

配置:
1. 启动 emacsclient
2. 生成 org-protocol.reg
```js
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\org-protocol]
"URL Protocol"=""
@="URL:Org Protocol"

[HKEY_CLASSES_ROOT\org-protocol\shell]

[HKEY_CLASSES_ROOT\org-protocol\shell\open]

[HKEY_CLASSES_ROOT\org-protocol\shell\open\command]
@="\"C:\\Program Files\\Emacs\\emacs-29.1\\bin\\emacsclientw.exe\"  \"%1\""
```

3. 增加org template
```js
(setq org-agenda-file-note (expand-file-name "~/notes.org"))
(setq org-capture-templates
      '(
        ("x" "Web Collections" entry
         (file+headline org-agenda-file-note "Web")
         "* %U %:annotation\n\n%:initial\n\n%?")
        ))
```

4. add chrome bookmark
```js
javascript:location.href='org-protocol://capture?template=x&url=%27+encodeURIComponent(location.href)+%27&title=%27+encodeURIComponent(document.title)+%27&body=%27+encodeURIComponent(function(){var html = "";var sel = window.getSelection();if (sel.rangeCount) {var container = document.createElement("div");for (var i = 0, len = sel.rangeCount; i < len; ++i) {container.appendChild(sel.getRangeAt(i).cloneContents());}html = container.innerHTML;}var dataDom = document.createElement(%27div%27);dataDom.innerHTML = html;dataDom.querySelectorAll(%27a%27).forEach(function(item, idx) {console.log(%27find a link%27);var url = new URL(item.href, window.location.href).href;var content = item.innerText;item.innerText = %27[[%27+url+%27][%27+content+%27]]%27;});[%27p%27, %27h1%27, %27h2%27, %27h3%27, %27h4%27].forEach(function(tag, idx){dataDom.querySelectorAll(tag).forEach(function(item, index) {var content = item.innerHTML.trim();if (content.length > 0) {item.innerHTML = content + %27&#13;&#10;';}});});return dataDom.innerText.trim();}())
```

5. return follow link
`(org-return-follows-link t)`

---

spell checking
。。。

---

翻译字典
。。。


## 19. ==Org Roam==

。。基于 org mode 的 知识管理系统。 主要功能:
- 非层次化笔记
- 双向链接
- 图形化视图
- 实时搜索
- 版本控制


安装
在HOME新建一个 org文件夹，用来保存 org roam笔记
```js
(use-package org-roam
  :ensure t
  :custom
  (org-roam-directory (file-truename "~/org"))
  :bind (("C-c n l" . org-roam-buffer-toggle)
         ("C-c n f" . org-roam-node-find)
         ("C-c n g" . org-roam-graph)
         ("C-c n i" . org-roam-node-insert)
         ("C-c n c" . org-roam-capture)
         ;; Dailies
         ("C-c n j" . org-roam-dailies-capture-today))
  :config
  ;; If you're using a vertical completion framework, you might want a more informative completion interface
  (setq org-roam-node-display-template (concat "${title:*} " (propertize "${tags:10}" 'face 'org-tag)))
  (org-roam-db-autosync-mode)
  ;; If using org-roam-protocol
  (require 'org-roam-protocol))
```

### 基本使用

安装 corfu 来进行补全
```js
(use-package corfu
  :init
  (progn
    (setq corfu-auto t)
    (setq corfu-cycle t)
    (setq corfu-quit-at-boundary t)
    (setq corfu-quit-no-match t)
    (setq corfu-preview-current nil)
    (setq corfu-min-width 80)
    (setq corfu-max-width 100)
    (setq corfu-auto-delay 0.2)
    (setq corfu-auto-prefix 1)
    (setq corfu-on-exact-match nil)
    (global-corfu-mode)
    ))
```

新建笔记和查找笔记
`C-c n f`

建立笔记引用
`[[]]`

查找引用
`org-return-follows-link t c-c n l`

已有的headline转为一个节点
`org-id-get-create refile node`

---

### 从github安装插件

```js
(require 'cl-lib)
(require 'use-package-core)

(cl-defun slot/vc-install (&key (fetcher "github") repo name rev backend)
  (let* ((url (format "https://www.%s.com/%s" fetcher repo))
         (iname (when name (intern name)))
         (package-name (or iname (intern (file-name-base repo)))))
    (unless (package-installed-p package-name)
      (package-vc-install url iname rev backend))))

(defvar package-vc-use-package-keyword :vc)

(defun package-vc-use-package-set-keyword ()
  (unless (member package-vc-use-package-keyword use-package-keywords)
    (setq use-package-keywords
          (let* ((pos (cl-position :unless use-package-keywords))
                 (head (cl-subseq use-package-keywords 0 (+ 1 pos)))
                 (tail (nthcdr (+ 1 pos) use-package-keywords)))
            (append head (list package-vc-use-package-keyword) tail)))))

(defun use-package-normalize/:vc (name-symbol keyword args)
  (let ((arg (car args)))
    (pcase arg
      ((or `nil `t) (list name-symbol))
      ((pred symbolp) args)
      ((pred listp) (cond
                     ((listp (car arg)) arg)
                     ((string-match "^:" (symbol-name (car arg))) (cons name-symbol arg))
                     ((symbolp (car arg)) args)))
      (_ nil))))

(defun use-package-handler/:vc (name-symbol keyword args rest state)
  (let ((body (use-package-process-keywords name-symbol rest state)))
    ;; This happens at macro expansion time, not when the expanded code is
    ;; compiled or evaluated.
    (if args
        (use-package-concat
         `((unless (package-installed-p ',(pcase (car args)
                                            ((pred symbolp) (car args))
                                            ((pred listp) (car (car args)))))
             (apply #'slot/vc-install ',(cdr args))))
         body)
      body)))

(defun package-vc-use-package-override-:ensure (func name-symbol keyword ensure rest state)
  (let ((ensure (if (plist-member rest :vc)
                    nil
                  ensure)))
    (funcall func name-symbol keyword ensure rest state)))

(defun package-vc-use-package-activate-advice ()
  (advice-add
   'use-package-handler/:ensure
   :around
   #'package-vc-use-package-override-:ensure))

(defun package-vc-use-package-deactivate-advice ()
  (advice-remove
   'use-package-handler/:ensure
   #'package-vc-use-package-override-:ensure))

;; register keyword on require
(package-vc-use-package-set-keyword)
```

---

### org roam UI

```js
(use-package org-roam-ui
  :vc (:fetcher "github" :repo "org-roam/org-roam-ui"))
```

- 鼠标操作
- 可视化
- 实时显示链接


## 20. Emacs配置问题排查

配置有问题，无法启动怎么办
`emacs-debug -init`

不能安装包，提示 签名错误
`(setq package-check-signature nil)`


## 21. 优化性能，借鉴其他人的配置

如何度量启动性能
```js
(use-package benchmark-init
  :ensure t
  :demand t
  :config
  ;; To disable collection of benchmark data after init is done.
  (add-hook 'after-init-hook 'benchmark-init/deactivate))
```


use-packages 哪些操作可以优化性能
https://systemcrafters.net/emacs-from-scratch/cut-start-up-time-in-half/
介绍了
- 度量 emacs-startup-hook
- use-package 的一些 延迟加载的 方法
  - :hook
  - :bind
  - :commands
  - :mode
  - :after
  - :defer
- with-eval-after-load
- 垃圾回收的配置
  - gc-cons-threshold


### 可借鉴的配置

社区的配置: spacemacs， doomemacs

个人配置
https://github.com/purcell/emacs.d
https://github.com/bbatsov/prelude
https://github.com/bbatsov/prelude
https://github.com/seagle0128/.emacs.d
https://github.com/zilongshanren/emacs.d 


小技巧
```js
(defun spacemacs/alternate-buffer (&optional window)
  "Switch back and forth between current and last buffer in the
current window.
If `spacemacs-layouts-restrict-spc-tab' is `t' then this only switches between
the current layouts buffers."
  (interactive)
  (cl-destructuring-bind (buf start pos)
      (if (bound-and-true-p spacemacs-layouts-restrict-spc-tab)
          (let ((buffer-list (persp-buffer-list))
                (my-buffer (window-buffer window)))
            ;; find buffer of the same persp in window
            (seq-find (lambda (it) ;; predicate
                        (and (not (eq (car it) my-buffer))
                             (member (car it) buffer-list)))
                      (window-prev-buffers)
                      ;; default if found none
                      (list nil nil nil)))
        (or (cl-find (window-buffer window) (window-prev-buffers)
                     :key #'car :test-not #'eq)
            (list (other-buffer) nil nil)))
    (if (not buf)
        (message "Last buffer not found.")
      (set-window-buffer-start-and-point window buf start pos))))
```

按键绑定
`"TAB" 'spacemacs/alternate-buffer`




---

# Emacs高手修炼手册

https://zhuanlan.zhihu.com/p/655059463

https://github.com/cabins/emacs.d

涉及
- 字体
- 主题
- Emacs29 内置 use-package

Emacs29内置的部分包:
- use-package
- global-auto-revert-mode， 文件内容自动加载到最新内容
- auto-save-visited-mode， 自动保存
- delete-selection-mode， 在编辑(删除文字)时有用
- fido-vertical-mode， minibuffer变成垂直 (非常推荐)
- flyspell-mode， 拼写检查，依赖系统安装 aspell， win慎用，疑似有性能问题
- global-hl-line-mode， 高亮当前行，建议 图形化(display-graphic-p)时使用，命令行下 体验不好
- recentf-mode， 打开最近的文件
- repeat-mode， 多次撤销只需要一次 `C-x u`， 之后只按 u即可


## 第三方包

company
自动补全
```js
(use-package company :ensure t :defer t
  :hook (after-init . global-company-mode))
```
如果你只喜欢在编程的时候启用，可以将:hook一行改成:hook (prog-mode . company-mode)。
默认输入第三个字母的时候开始出现补全选项。我最早会改出1个，但后来发现，完全没有必要，默认3个就很好，毕竟int def这样的关键字，输入比补全更快。

---

exec-path-from-shell
帮助读取你的环境变量， 不然很多的时候，我们会出现找不到我们〔明明就在那里〕的一些命令。
```js
;; Settings for exec-path-from-shell
;; fix the PATH environment variable issue
(use-package exec-path-from-shell
  :ensure t
  :when (or (memq window-system '(mac ns x))
        (unless cabins--os-win
          (daemonp)))
  :init (exec-path-from-shell-initialize))
```

format-all
格式化工具
```js
;; great for programmers
(use-package format-all :ensure t :defer t
  ;; 开启保存时自动格式化
  :hook (prog-mode . format-all-mode)
  ;; 绑定一个手动格式化的快捷键
  :bind ("C-c f" . #'format-all-region-or-buffer))
```

---

move-dup
将代码行 上移，下移。
内置的 transpose-line 也能完成类似的功能。
```js
(use-package move-dup :ensure t :defer t
  :hook (after-init . global-move-dup-mode))
```

---

which-key
提示接下来可能用到的快捷键
```js
(use-package which-key :ensure t :defer t
  :hook (after-init . which-key-mode))
```


## 编程

先启用几个 内置包，来增强 编程模式

```js
(add-hook 'prog-mode-hook 'column-number-mode) ;在ModeLine显示列号
(add-hook 'prog-mode-hook 'display-line-numbers-mode) ;显示行号
(add-hook 'prog-mode-hook 'electric-pair-mode) ;括号的配对
(add-hook 'prog-mode-hook 'flymake-mode) ;错误的提示
(add-hook 'prog-mode-hook 'hs-minor-mode) ;代码的折叠
(add-hook 'prog-mode-hook 'prettify-symbols-mode) ;会将lambda等符号美化为λ
```

听从 flymake的官方建议，给 错误跳转绑定 2个快捷键 (我天天用)
```js
(global-set-key (kbd "M-n") #'flymake-goto-next-error)
(global-set-key (kbd "M-p") #'flymake-goto-prev-error)
```


treesit
Emacs29开始，内置了 treesit。 它是一个解析器生成工具 和 增量解析库。可以为源文件 构建一个 具体的语法树， 并在 源文件被编辑时 高效地更新语法树。
29开始，也内置了很多 `-ts-mode`， 如，go-ts-mode,rust-ts-mode...， 这样 配合 eglot 就不需要第三方的包(go-mode,rust-mode)了。
```js
(use-package treesit
  :when (and (fboundp 'treesit-available-p)
         (treesit-available-p))
  :config (setq treesit-font-lock-level 4)
  :init
  (setq treesit-language-source-alist
    '((bash       . ("https://github.com/tree-sitter/tree-sitter-bash"))
      (c          . ("https://github.com/tree-sitter/tree-sitter-c"))
      (cpp        . ("https://github.com/tree-sitter/tree-sitter-cpp"))
      (css        . ("https://github.com/tree-sitter/tree-sitter-css"))
      (cmake      . ("https://github.com/uyha/tree-sitter-cmake"))
      (csharp     . ("https://github.com/tree-sitter/tree-sitter-c-sharp.git"))
      (dockerfile . ("https://github.com/camdencheek/tree-sitter-dockerfile"))
      (elisp      . ("https://github.com/Wilfred/tree-sitter-elisp"))
      (go         . ("https://github.com/tree-sitter/tree-sitter-go"))
      (gomod      . ("https://github.com/camdencheek/tree-sitter-go-mod.git"))
      (html       . ("https://github.com/tree-sitter/tree-sitter-html"))
      (java       . ("https://github.com/tree-sitter/tree-sitter-java.git"))
      (javascript . ("https://github.com/tree-sitter/tree-sitter-javascript"))
      (json       . ("https://github.com/tree-sitter/tree-sitter-json"))
      (lua        . ("https://github.com/Azganoth/tree-sitter-lua"))
      (make       . ("https://github.com/alemuller/tree-sitter-make"))
      (markdown   . ("https://github.com/MDeiml/tree-sitter-markdown" nil "tree-sitter-markdown/src"))
      (ocaml      . ("https://github.com/tree-sitter/tree-sitter-ocaml" nil "ocaml/src"))
      (org        . ("https://github.com/milisims/tree-sitter-org"))
      (python     . ("https://github.com/tree-sitter/tree-sitter-python"))
      (php        . ("https://github.com/tree-sitter/tree-sitter-php"))
      (typescript . ("https://github.com/tree-sitter/tree-sitter-typescript" nil "typescript/src"))
      (tsx        . ("https://github.com/tree-sitter/tree-sitter-typescript" nil "tsx/src"))
      (ruby       . ("https://github.com/tree-sitter/tree-sitter-ruby"))
      (rust       . ("https://github.com/tree-sitter/tree-sitter-rust"))
      (sql        . ("https://github.com/m-novikov/tree-sitter-sql"))
      (vue        . ("https://github.com/merico-dev/tree-sitter-vue"))
      (yaml       . ("https://github.com/ikatyang/tree-sitter-yaml"))
      (toml       . ("https://github.com/tree-sitter/tree-sitter-toml"))
      (zig        . ("https://github.com/GrayJack/tree-sitter-zig"))))
  (add-to-list 'major-mode-remap-alist '(sh-mode         . bash-ts-mode))
  (add-to-list 'major-mode-remap-alist '(c-mode          . c-ts-mode))
  (add-to-list 'major-mode-remap-alist '(c++-mode        . c++-ts-mode))
  (add-to-list 'major-mode-remap-alist '(c-or-c++-mode   . c-or-c++-ts-mode))
  (add-to-list 'major-mode-remap-alist '(css-mode        . css-ts-mode))
  (add-to-list 'major-mode-remap-alist '(js-mode         . js-ts-mode))
  (add-to-list 'major-mode-remap-alist '(java-mode       . java-ts-mode))
  (add-to-list 'major-mode-remap-alist '(js-json-mode    . json-ts-mode))
  (add-to-list 'major-mode-remap-alist '(makefile-mode   . cmake-ts-mode))
  (add-to-list 'major-mode-remap-alist '(python-mode     . python-ts-mode))
  (add-to-list 'major-mode-remap-alist '(ruby-mode       . ruby-ts-mode))
  (add-to-list 'major-mode-remap-alist '(conf-toml-mode  . toml-ts-mode))
  (add-to-list 'auto-mode-alist '("\\(?:Dockerfile\\(?:\\..*\\)?\\|\\.[Dd]ockerfile\\)\\'" . dockerfile-ts-mode))
  (add-to-list 'auto-mode-alist '("\\.go\\'" . go-ts-mode))
  (add-to-list 'auto-mode-alist '("/go\\.mod\\'" . go-mod-ts-mode))
  (add-to-list 'auto-mode-alist '("\\.rs\\'" . rust-ts-mode))
  (add-to-list 'auto-mode-alist '("\\.ts\\'" . typescript-ts-mode))
  (add-to-list 'auto-mode-alist '("\\.y[a]?ml\\'" . yaml-ts-mode)))
```

eglot
```js
(use-package eglot
  :hook (prog-mode . eglot-ensure)
  :bind ("C-c e f" . eglot-format))
```
bind是绑定了一个 格式化的快捷键，但是 有 format-all，所以不是很必须。


---

可选第三方包
有一些特殊的需求，如 显示 protobuf 或 quickrun 等，那么可以配置
```js
(use-package protobuf-mode :ensure t :defer t)
(use-package quickrun :ensure t :defer t)
(use-package restclient :ensure t :defer t
  :mode (("\\.http\\'" . restclient-mode)))
```






---


# now, it's my show time.

2024-05-15 10:22


1. 仓库

4个地方(大约1414行)都没有 emacs的 启动配置。
有 `~/.emacs.d`
所以 新建了一个 文件在 `~/.emacs.d/init.el`
复制了 配置，有效。


ivy or helm ?
看下来，helm 的功能很强，但是 速度慢。 而且 强的过头了， 很多用不到的功能。








2. 主题

估计用 https://draculatheme.com/emacs
其他的，git上的图 看不了，局域网有点烦人啊。

不了，还是用 https://github.com/bbatsov/zenburn-emacs   相信 第一名 的含金量。

德古拉 颜色很亮/跳(和背景高对比，白色，绿色，(粉)红色)，但是 注释的颜色太淡了(和背景色有点接近)。


最终： https://github.com/oneKelvinSmith/monokai-emacs
zenburn 对比度太低了，估计会很累。
monokai 是排名最高的 全黑背景，  其他的 是 灰色的背景








3. c++,rust






inti.el 已经上传到 github， my_emacs_lisp/init.el

2024-05-15 21:29




2025-3-3


https://zhuanlan.zhihu.com/p/476668873
eglot 代替 LSP

emacs29 内置了 eglot 。。**怎么启动?**

3点修改
- eldoc 高度限制为 3 行，太大了影响阅读代码
- 修改高亮『当前变量』的字体，默认的不是很明显
- 增加 Rust 宏展开的命令，lsp-mode 默认支持，这里给出了 eglot 的实现

```js
(use-package eglot
  :defer t
  :commands (eglot-ensure my/rust-expand-macro)
  :config
  (progn
    (setq eldoc-echo-area-use-multiline-p 3
          eldoc-echo-area-display-truncation-message nil)
    (set-face-attribute 'eglot-highlight-symbol-face nil
                        :background "#b3d7ff")

    (defun my/rust-expand-macro ()                    ; 我可以不要这个
      "Expand macro at point, same as `lsp-rust-analyzer-expand-macro'.
https://rust-analyzer.github.io/manual.html#expand-macro-recursively"
      (interactive)
      (jsonrpc-async-request
       (eglot--current-server-or-lose)
       :rust-analyzer/expandMacro (eglot--TextDocumentPositionParams)
       :error-fn (lambda (msg) (error "Macro expand failed, msg:%s." msg))
       :success-fn
       (lambda (expanded-macro)
         (cl-destructuring-bind (name format expansion result) expanded-macro
           (let* ((pr (eglot--current-project))
                  (buf (get-buffer-create (format "*rust macro expansion %s*" (cdr pr)))))
             (with-current-buffer buf
               (let ((inhibit-read-only t))
                 (erase-buffer)
                 (insert result)
                 (rust-mode)))
             (switch-to-buffer-other-window buf))))))
    ))
```

2025-3-3


https://github.com/cabins/emacs.d
https://github.com/condy0919/.emacs.d


