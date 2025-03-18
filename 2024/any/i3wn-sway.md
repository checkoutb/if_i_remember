
2024-05-27 19:32


1. 改键
  交换 caps lock 和 win，这就是看 caps lock 的灯了，估计这个灯是 物理灯，所以挺麻烦的。(就是，软件灯的话，应该可以 按下 caps lock 才亮，所以 交换按键后， 按 win ，才会亮灯。但感觉不太现实。)
2. 如何通过代码的方式 启动app，设置app的所在窗口 和大小布局
  绑定到 Fn + F5 或者直接 F1
  反正没有找到 如何 设置 app的大小，所属窗口。
3. sway-bar 怎么增加功能。
  它支持什么，完全没有提及，感觉是不可变的。除非换一个 bar，但是也不太可能支持自定义。最多就是提供一些模块，然后自定义开启。 
  反正现在的 sway-bar，我感觉是 完全不可配置的。 至少找不到关于配置的内容。
  其他bar 的配置： https://github.com/Alexays/Waybar/wiki/Configuration ，它可以在 bar 上绑定 鼠标的功能，就是鼠标单击可以执行一些命令，二十个。
  `gsettings set org.gnome.desktop.interface gtk-theme Adwaita-dark` 不清楚这个命令有没有用。 或者可以装 gnome-tweaks ，没用， sway 对标 gnome， wayland 对标 x11 ， 所以 有 sway，没 gnome， 它们是同一个范畴的。
4. darkmode
  dnf install darkman .这个用 go 写的。 https://gitlab.com/WhyNotHugo/darkman
  https://wiki.archlinux.org/title/Dark_mode_switching 中第一个就是 darkman。 还有个 https://github.com/oskarsh/Yin-Yang

https://github.com/pedroscaff/swaywsr   
rust的，修改workspace的名字。 代码很短，使用了 swap 的IPC


[[toc]]

---
---




# sway

https://github.com/swaywm/sway/wiki

配置的模板文件是 `/etc/sway/config`， 需要复制为 `~/.config/sway/config`


## 改键

可以改键。 这种功能是 wm 的功能吗？ 这种 改键 会影响 其他软件的输入吗？
。不是，改键是 xkb 的功能 。X Keyboard Extension。 键盘驱动程序。

比如
- caps lock 设置为 空格 `xkb_options caps:escape`
- 交换 Esc 和 caps lock `xkb_options caps:swapescape`
- caps lock 设置为 ctrl `xkb_options ctrl:nocaps`
- caps lock 设置为 额外的ctrl `xkb_options caps:ctrl_modifier`
- 交换 左alt 和 super, 且设置 caps lock 为 Esc `xkb_options altwin:swap_lalt_lwin,caps:escape`
- 菜单键作为额外的super `xkb_options altwin:menu_win`

触发多个选项时，需要用 逗号连接 `xkb_options caps:escape,altwin:swap_lalt_lwin`，不然 只有最后一个生效


加载 自定义的 xkb keymap 文件
```text
# or input <identifier>
input "type:keyboard" {
    xkb_file ~/.config/xkb/custom
}
```

键盘重复，延迟，次数
```text
# or input <identifier>
input "type:keyboard" {
    repeat_delay 500
    repeat_rate 5
}
```

`man 5 sway-input`


### 我的改建


。。感觉可以 交换下 caps lock 与 win键
。win键 大拇指 太累了。
。。但是 caps lock 有个灯。太亮了。

---

还有 F1 - F12 可以绑定一些 app
或许应该用 `Fn + F5` 这种。 可以自己写app，一键开启 终端，ff，emacs，code，joplin，并分好 窗口。  但是 没有看到 sway 怎么通过 代码的方式 移动/设置 app




## 壁纸

`man 1 swaybg`

如果安装了 swaybg， 那么可以换壁纸。
。默认安装了。

使用 output 命令的 bg 选项
`output HDMI-A-1 bg ~/wallpaper.png stretch`

。。HDMI-A-1 怎么来的？
。output 命令是  sway 配置文件中的 output命令 ， 不是 命令行的



## swaybar 配置

`man 5 sway-bar`

支持大多数 i3 bar 的功能。 配置方法相同。
增加一个 bar 节到你的配置中
```text
bar {
    # ...bar options..
}
```

---

## 注释 # 

注释是 # 开头 ， 命令和 注释不能在同一行， 因为有 #RRGGBB 这种颜色配置。

如果注释的最后 是 \， 那么 下一行也是注释(即使没有 # 开头)， 所以注释中 不要使用 \ 结尾



## 截屏，录像

查看插件列表了解更多

截屏，使用 grim， 或 Grimshot,shotman,flameshot,swappy

录像使用 wf-recorder


## 程序加载器

bemenu

sway-launcher-desktop

wofi

yofi

tofi


## 不能进行屏幕缩放 



## 设置环境变量

设置环境变量 依赖于 sway是怎么启动的

sway 会从 启动sway的进程 继承环境变量。你需要在那里设置 变量

一些可能的启动者
- login manager
- login shell
- user service





还有一些 FAQ










# sway 的 插件

https://github.com/swaywm/sway/wiki/Useful-add-ons-for-sway

就记录下 大类

login manager
launcher，   应用启动器
menu，
display/output
图片查看器， imv,mvi,swayimg
视频播放， mpv
notification, mako, fnott, dunst, wayherb, swaync, salut
workspace
overview
layout，
screenshot
brightness，调整屏幕亮度
Gamma，屏幕颜色的gamma值
colorscheme， darkman，深色模式
wallpaper，壁纸
Bars & panels
Bar content generators
Widget 小装置
Keyboard/Input
Input Method Editors
On-screen keyboards
idle timeout
locking
terminal
VNC/RDP/SPICE
Remote/recording
Misc. Scripts
Development







# 以下为 i3-wm

# 快捷键

## 鼠标

悬浮在哪里， 哪个窗口就是 焦点。当然可以通过 mod + jkl; 来移动， 并且只有当 鼠标 再次进入 焦点窗口时， 鼠标才能 通过悬浮 移动 焦点


## mod
- a, forcus parent
- s, stacked layout
- d, dmenu，运行软件
- f, 当前软件全屏
- j, 焦点移到左边窗口
- k, 下
- l, 上
- ;, 右侧
- w, tabbed layout，一个屏幕展示一个窗口，通过上方的标签切换
- e, default layout，一个屏幕展示所有窗口
- r, resize mode
- v, split vert
- h, split horiz
- space, focus floating/tilling
- return, 打开cmd
- 1/2/3, 切换不同的桌面


## mod + shift
- j, 窗口向左移动
- k, 下
- l, 上
- ;, 右
- space, toggle tilling/floating
- q, 关闭窗口
- e, 退出 i3wm
- r, 重启 i3wm
- 1/2/3,  将焦点窗口移动到另一个桌面



## 组合

mod + a， mod + s， 会把当前屏幕的窗口 组合到一起，
但是不知道怎么拆分。    // mod + e
也不知道怎么 合并任意 n个，这里只能合并一个屏幕，当然可以将 不需要的 放到其他桌面， 合并，然后再放回来。

主要是那几个快捷键 功能不知道具体是什么。






# user guide

https://i3wm.org/docs/userguide.html


在 打开新窗口 前， mod + v，下个窗口就是 水平切分， mod + h，下个窗口就是 垂直切分


2.2
split container 有3 种布局
- 按垂直/水平 split， mod + e, 默认也是这个
- stack，每个标题占一行, mod + s
- tabbed，所有标题在一行上。mod + w


2.4
你可以为 常用的应用 配置一个 快捷键。


2.6
workspace 可以 将窗口 分组
默认情况下，你在 第一个 工作空间，
`mod + 数字`
多屏幕也可以用

`mod + shift + 数字` 移动到某个工作空间

`mod + r` 就是 调整 大小，就是 mod + r 然后 方向键
不过一般直接用鼠标即可


2.11 floating
你可以自己控制 窗口的位置和大小 (。。就是最普遍的模式，不过总要有个应用占据整个屏幕。。)。
。。只能控制 一个窗口

这个模式 对于 save as 之类的窗口 特别适用。

`mod + shift + space`


2.12 通过鼠标拖动 tilling container
从4.21开始。
。。但是我4.23 拖不动。



3
所有信息都以 tree 的形式保存


3.3
focus parent

。以当前窗口作为 起始节点，然后 一次 mod + a，就 向上一级。
。新建节点也是在当前节点的右侧/下方 新建的



4
配置i3

为了修改i3的配置，复制 /etc/i3/config 到 ~/.i3/config ( 或 ~/.config/i3/config , 如果你喜欢 XDG目录格式)， 然后编辑
。。安装系统的时候选择 win作为mod，它会自动创建到 ~/.config/i3/config

也可以使用 wized (向导。。应该就是 安装系统的时候的那个，很简单的)，使用的时候 必须保证 ~/.i3/config 不存在。

从4.0开始，有了一种 新的 配置的format。  i3 会通过一些关键词 自动侦测 配置的版本。 也可以显式指定 `# i3 config file (v4)`



从 4.20 开始，可以在配置文件中 导入 其他的配置文件
`include <pattern>`

```text
include ~/.config/i3/assignments.conf
include $HOME/.config/i3/assignments.conf   # 。环境变量
include ~/.config/i3/config.d/*.conf        # 。。通配符
include ~/.config/i3/`hostname`.conf        # 。。hostname 是一个 命令，通过 `` 变成普通string
include ~/.config/i3/config     # 。不会导致死循环，会报错
include assignments.conf        # 。相对路径
```

`i3 --moreversion`
获得所有的加载的 配置文件

。。变量 在 所有配置 文件中共享，但是必须 include 后 才可以使用。 好像不是，下面的 挺难理解的。
。感觉意思是  被 include 的文件 可以使用 外层 文件中 定义的 变量？
。。所以 和 C++ 的 include 是相反的？

- You can define a variable and use it within an included file.
- You cannot use (in the parent file) a variable that was defined within an included file.


有一个技术限制：变量的扩展 在 处理include语句 之前

从概念上来说， included 文件 只能 增加配置，不能 修改/撤销 已有的配置。 比如，你只能增加 新的 快捷键，不能 覆盖 或 溢出 已有的快捷键



`# xxx` 是注释


4.3
字体
i3 支持所有 X的 和 免费的 字体 来影响 窗口的 title

如果无法打开字体，会 报错， 并返回 默认字体

语法
```text
font <X core font description>
font pango:<family list> [<style options>] <size>
```

例子
```text
font -misc-fixed-medium-r-normal--13-120-75-75-C-70-iso10646-1
font pango:DejaVu Sans Mono 10
font pango:DejaVu Sans Mono, Terminus Bold Semi-Condensed 11
font pango:Terminus 11px
```


## 4.4 键盘绑定

将 命令 绑定到 某个按键上

可以绑定到 keycodes 或 keysyms
- keysyms (key symbol) 是 符号的 描述， 是你在 Xmodmap 中 使用的 remap key的符号。 通过 `xmodmap -pke` 来获得 map 。 要通过输入key 来获得 keysym，你可以通过 `xev`

- keycodes 不需要 有 符号。但你使用 其他布局时，不会修改 含义

。。就是 一个是 字符，比如说 a， 不同的键盘布局，a 的位置不同， 但是 只要是 a字符，那么就是 同一个快捷键
。keycodes 就是 某个特定的 实体 键， 不管 布局是什么， 始终都是  这个 实体键。


我的建议是：如果你 经常改变 键盘布局，且 你希望 将 绑定 保持在 某些 物理键上，那么 适用 keycodes。
如果你 换 键盘布局，且 想要一个 简洁的 配置文件，那么适用 keysyms

一些工具 (如 import，xdotool) 无法对 KeyPress 事件作出响应， 因为 键盘/指针 仍然 被占用着
这种场景下， 可以适用 `--release` 标记。

语法
```text
bindsym [--release] [<Group>+][<Modifiers>+]<keysym> command
bindcode [--release] [<Group>+][<Modifiers>+]<keycode> command
```

```text
bindsym $mod+f fullscreen toggle    # 全屏

bindsym $mod+Shift+r restart    # 重启i3

bindcode 214 exec --no-startup-id /home/michael/toggle_beamer.sh    # 笔记本特定的热键

bindsym --release $mod+x exec --no-startup-id xdotool key --clearmodifiers ctrl+v   # 按下 mod + x 时模拟 ctrl + v

bindsym --release $mod+x exec --no-startup-id import /tmp/latest-screenshot.png # mod + x 是截图
```

可用的 modifiers
mod1 - mod5， shift， ctrl

当使用 多个 键盘布局时， 你可以指定 哪个 键盘布局 有效


4.5
鼠标绑定

语法
`bindsym [--release] [--border] [--whole-window] [--exclude-titlebar] [<Modifiers>+]button<n> command`

默认情况下，只有当你 点击 窗口的 标题栏时 才执行。
当指定 `--release`， 当鼠标 到达 标题栏 时 就执行

`--whole-window`， 窗口的 任意一处 被点击时 就执行binding (。而不是 只有 标题栏) 。
`--border`， 当 边框被点击时 执行
`--exclude-titlebar`， keybind时， 标题栏 不被 考虑。


```text
bindsym --release button2 kill  # 中键 kill 窗口

bindsym --whole-window $mod+button2 kill    # 窗口的任意部分 被 mod + 中键 就 kill 窗口

bindsym button3 floating toggle # 右键触发 floating
bindsym $mod+button3 floating toggle

bindsym button9 move left   # 鼠标侧键 移动窗口
bindsym button8 move right
```



4.6
绑定模式

你可以 使用不同的绑定模式 来 获得 多组绑定集合。
。就是多种配置。 不，是 emacs 的那种 组合键 (C-x C-x 按2次 才是一个完整的 功能)。

语法
```text
# config directive
mode [--pango_markup] <name>

# command
mode <name>
```

```text
# Press $mod+o followed by either f, t, Escape or Return to launch firefox,
# thunderbird or return to the default mode, respectively.
set $mode_launcher Launch: [f]irefox [t]hunderbird          # 这个是 提示吗？
bindsym $mod+o mode "$mode_launcher"

mode "$mode_launcher" {
    bindsym f exec firefox
    bindsym t exec thunderbird

    bindsym Escape mode "default"
    bindsym Return mode "default"
}
```



4.7
floating modifier

。。。?
效果应该是， 当你 按住 mod1 的时候， 你鼠标 右键 点中 某个 window，就可以 拖动它。 会出现 阴影的。

语法： `floating_modifier <Modifier>`

例子： `floating_modifier Mod1`



4.8
约束 floating 窗口大小

-1 是 不限制

语法 + 例子
```text
floating_minimum_size <width> x <height>
floating_maximum_size <width> x <height>

floating_minimum_size 75 x 50
floating_maximum_size -1 x -1
```


新workspace 的方向

默认情况下，如果 显示器 宽 > 高，那么 水平布局。 如果 宽 < 高， 那么 垂直布局

`default_orientation horizontal|vertical|auto`

`default_orientation vertical`


新容器的 布局

`workspace_layout default|stacking|tabbed`

`workspace_layout tabbed`


4.11
窗口标题的对齐
默认靠左

`title_align left|center|right`



新窗口的 默认的 border style

The default is normal

default_floating_border 只对那些 以 floating window 开始的 window 有效 ， 对那些 变成 floating window 的无效

在 split 布局中，适用 pixel 会消除 title bar
normal 可以 在不动 title bar的情况下 修改 边宽宽度

在 stack 和 tabbed 布局中， title bar 始终可见，不能通过 配置修改。

语法
```text
default_border normal|none|pixel
default_border normal|pixel <px>
default_floating_border normal|none|pixel
default_floating_border normal|pixel <px>
```

new_window, new_float 已经不建议适用了，以后会移除。 。。不知道是什么，这里并没有啊。

例子
`default_border pixel`

normal，pixel 支持 可选的 board宽度 像素

```text
# The same as default_border none
default_border pixel 0

# A 3 px border
default_border pixel 3
```



4.13
隐藏 和显示器边 相交的 边

默认是 none。

smart：workspace只有一个 窗口时，隐藏，  有多个窗口时，有边。

`hide_edge_borders none|vertical|horizontal|both|smart|smart_no_gaps`

`hide_edge_borders vertical`



对指定窗口的 任意命令 (for_window)

通过 for_window，你可以 让 i3 执行 任意 命令，当 i3 遇到 窗口时。
可以用于 设置 窗口 为 floating 或 修改 border style。

`for_window <criteria> <command>`

```text
# enable floating mode for all XTerm windows
for_window [class="XTerm"] floating enable

# Make all urxvts use a 1-pixel border:
for_window [class="urxvt"] border pixel 1

# A less useful, but rather funny example:
# makes the window floating as soon as I change
# directory to ~/work
for_window [title="x200: ~/work"] floating enable
```

criteria 查看 command criteria
只有运行时 command 可用。



4.15
打开窗口时 不要 focus

新窗口出现时，默认会focus。

`no_focus <criteria>`

`no_focus [window_role="pop-up"]`

criteria 查看 command criteris


## 4.16 变量


`set $<name> <value>`

```text
set $m Mod1
bindsym $m+Shift+r restart
```

变量会被直接 替换。
不会 递归替换

如果你需要 更动态的配置， 需要使用 xresources 脚本


4.17
x resources

可以通过 x resource database 来创建 变量。

定义一个资源 会从 资源数据库中 加载它，并赋值到 指定的变量。
逐个执行，并且值必须符合 i3的规范
必须指定一个 fallback，以便在 无法从 database 加载时 调用

`set_from_resource $<name> <resource_name> <fallback>`

```text
# The ~/.Xresources should contain a line such as
#     *color0: #121212
# and must be loaded properly, e.g., by using
#     xrdb ~/.Xresources
# This value is picked up on by other applications (e.g., the URxvt terminal
# emulator) and can be used in i3 like this:
set_from_resource $black i3wm.color0 #000000
```



4.18
在指定的 workspace 中 自动 put client 



。。气死了， x11 都要被淘汰了。
而且看 github，sway的 star 比 i3 多。。擦。
。。搞得又想装 sway 了，擦。



跑了。




# 目录

```text

1. Default keybindings
2. Using i3
    2.1. Opening terminals and moving around
    2.2. Changing the container layout
    2.3. Toggling fullscreen mode for a window
    2.4. Opening other applications
    2.5. Closing windows
    2.6. Using workspaces
    2.7. Moving windows to workspaces
    2.8. Resizing
    2.9. Restarting i3 inplace
    2.10. Exiting i3
    2.11. Floating
    2.12. Moving tiling containers with the mouse
3. Tree
    3.1. The tree consists of Containers
    3.2. Orientation and Split Containers
    3.3. Focus parent
    3.4. Implicit containers
4. Configuring i3
    4.1. Include directive
    4.2. Comments
    4.3. Fonts
    4.4. Keyboard bindings
    4.5. Mouse bindings
    4.6. Binding modes
    4.7. The floating modifier
    4.8. Constraining floating window size
    4.9. Orientation for new workspaces
    4.10. Layout mode for new containers
    4.11. Window title alignment
    4.12. Default border style for new windows
    4.13. Hiding borders adjacent to the screen edges
    4.14. Arbitrary commands for specific windows (for_window)
    4.15. Don’t focus window upon opening
    4.16. Variables
    4.17. X resources
    4.18. Automatically putting clients on specific workspaces
    4.19. Automatically starting applications on i3 startup
    4.20. Automatically putting workspaces on specific screens
    4.21. Changing colors
    4.22. Interprocess communication
    4.23. Focus follows mouse
    4.24. Mouse warping
    4.25. Popups during fullscreen mode
    4.26. Focus wrapping
    4.27. Forcing Xinerama
    4.28. Automatic back-and-forth when switching to the current workspace
    4.29. Delaying urgency hint reset on workspace change
    4.30. Focus on window activation
    4.31. Drawing marks on window decoration
    4.32. Line continuation
    4.33. Tiling drag
    4.34. Gaps
5. Configuring i3bar
    5.1. i3bar command
    5.2. Statusline command
    5.3. Workspace buttons command
    5.4. Display mode
    5.5. Mouse button commands
    5.6. Bar ID
    5.7. Position
    5.8. Output(s)
    5.9. Tray output
    5.10. Tray padding
    5.11. Font
    5.12. Custom separator symbol
    5.13. Workspace buttons
    5.14. Minimal width for workspace buttons
    5.15. Strip workspace numbers/name
    5.16. Binding Mode indicator
    5.17. Colors
    5.18. Transparency
    5.19. Padding
6. List of commands
    6.1. Executing applications (exec)
    6.2. Splitting containers
    6.3. Manipulating layout
    6.4. Focusing containers
    6.5. Moving containers
    6.6. Swapping containers
    6.7. Sticky floating windows
    6.8. Changing (named) workspaces/moving to workspaces
    6.9. Moving workspaces to a different screen
    6.10. Moving containers/workspaces to RandR outputs
    6.11. Moving containers/windows to marks
    6.12. Resizing containers/windows
    6.13. Jumping to specific windows
    6.14. VIM-like marks (mark/goto)
    6.15. Window title format
    6.16. Window title icon
    6.17. Changing border style
    6.18. Enabling shared memory logging
    6.19. Enabling debug logging
    6.20. Reloading/Restarting/Exiting
    6.21. Scratchpad
    6.22. Nop
    6.23. i3bar control
    6.24. Changing gaps
7. Multiple monitors
    7.1. Configuring your monitors
    7.2. Interesting configuration for multi-monitor environments
8. i3 and the rest of your software world
    8.1. Displaying a status line
    8.2. Giving presentations (multi-monitor)
    8.3. High-resolution displays (aka HIDPI displays)
```






