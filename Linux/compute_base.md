# Computer
[TOC]

## 图灵机
图灵思考的问题：
1. 数学问题是否都有明确的答案。
2. 如果有明确答案，是否可以通过有限步骤的计算得到答案。
3. 对于可能在有限步骤计算出来的数学问题，能否有一种机器，它不断地运动，最后停下来的时候，那个数学问题就解决了。



## 晶体管
![](images/compute_base/transistor.png)

> N型MOS管高电平导通  
> P型MOS管低电平导通

### 逻辑电路
### 非门
![](images/compute_base/not-gate.png)

### 非门的工作过程
![](images/compute_base/not-gate-working-process.png)

### 与门
![](images/compute_base/and-gate.png)

> 与非门在实际中更容易实现，一般用 **与非门** 和 **非门** 实现 **与门**

### 与非门工作工程
![](images/compute_base/and-not-gate-working-process.png)

### 或门
![](images/compute_base/or-gate.png)

### 异或门
![](images/compute_base/xor-gate.png)

![](images/compute_base/exclusive-or.png)
上面是用与或非三种门电路实现的一种异或门

### 寄存器
![](images/compute_base/or-1.png)
在或门的输出连接到一个输入端，一旦A或B有一个为1时，输出都是1。
之后无论A，B怎么变化，输出端都是1.

![](images/compute_base/and-1.png)
在这个与门中，先把A，B都设置为1，之后A为0，输出都为0。
跟上面的或门正好相反。

![](images/compute_base/and-or-latch.png)
把之前的与门和或门组合在一起，可以组成一个AND-OR锁存器(AND-OR Latch)[^and-or-latch]
它有2个输入
“设置”输入1，它的输出为1
“复位”输入1，它的输出为0
**如果“设置”和“复位”都是0，整个电路的输出为最后放入的内容**。

| SET  | RESET | OUTPUT |
| :--- | :---- | :----- |
| 1    | 0     | 1      |
| 0    | 0     | 1      |

| SET  | RESET | OUTPUT |
| :--- | :---- | :----- |
| 0    | 1     | 0      |
| 0    | 0     | 0      |

![](images/compute_base/gated-latch.png)
为了易用性，我们希望只有一条输入线，还需要一条线来启用内存。 



## 加法器
### 半加器
![](images/compute_base/add-xor.png)
除了1 + 1需要进位外，所有的末位可以用异或门来表示

![](images/compute_base/add-xor-and.png)
对于 1 + 1，它的进位1，可以用与门来表示[^add-xor-and]

异或门 + 与门
![](images/compute_base/half-adder.png)

### 全加器
![](images/compute_base/add-3.png)
用半加法器计算1 + 1时产生了进位，这个进位会参与下一个计算
所以不是2位，而是3位相加。

![](images/compute_base/full-adder.png)

![](images/compute_base/full-adder-2.png)


### 4bit的加法器
![](images/compute_base/4bit-adder.png)

### 8bit的加法器
![](images/compute_base/8bit-adder.png)


## 计算机结构的简化模型
图片来自： https://www.bilibili.com/video/BV1VE411o7nx?p=4

### 模型机
![](images/compute_base/simple-mode-1.png)

### 存储器
![](images/compute_base/simple-mode-2.png)

### CPU 控制器 
![](images/compute_base/simple-mode-3.png)

#### 控制器的基本组成
![](images/compute_base/simple-mode-4.png)

![](images/compute_base/simple-mode-5.png)

### CPU 运算器
![](images/compute_base/simple-mode-6.png)

#### 运算器的基本组成
![](images/compute_base/simple-mode-7.png)

### CPU的内部总线
![](images/compute_base/simple-mode-8.png)

## 执行指令的过程
举例说明：

![](images/compute_base/execute-1.png)

假设模型机当前状态:
![](images/compute_base/execute-2.png)

### 取指
![](images/compute_base/execute-fetch.gif)

### 译码
![](images/compute_base/execute-decoding.gif)

### 执行
![](images/compute_base/execute-execute.gif)

### 回写
![](images/compute_base/execute-writeback.gif)


程序 = 指令 + 数据

内存中的指令告诉CPU 从哪去取数据


## 总线
总线再某一时刻只能由2个设备使用。 由控制器
CPU里有总的控制器， 各个设备有各个的控制器（像磁盘的SATA控制器）。 
CPU的控制器与各个设备的控制器相互通信
CPU能控制总线

## Linux
### API(application programing interface)
- syscall
- libcall

### ABI(application binary interface)
- ELF

## 终端
### 虚拟终端:
Teletype
Ctrl + ALT + F[1-6]
表示： /dev/tty#

[下图来自这里](https://segmentfault.com/a/1190000009082089)
```
                   +-----------------------------------------+
 | Kernel       |                       |                   |              |                    |              |                |
 | +--------+   | +----------------+    |                   |              |                    |              |                |
 | +----------+ | +-------------------+ | tty1              | <----------> | User processes     |              |                |
 | Keyboard     | --------->            |                   | +--------+   | +----------------+ |              |                |
 | +----------+ |                       | Terminal Emulator | <->          | tty2               | <----------> | User processes |
 | Monitor      | <---------            |                   | +--------+   | +----------------+ |              |                |
 | +----------+ | +-------------------+ | tty3              | <----------> | User processes     |              |                |
 | +--------+   | +----------------+    |                   |              |                    |              |                |
 |              |                       |                   |              |                    |              |                |
                   +-----------------------------------------+
```

键盘、显示器都和内核中的终端模拟器相连，由模拟器决定创建多少tty，比如你在键盘上输入ctrl+alt+F1时，模拟器首先捕获到该输入，然后激活tty1，这样键盘的输入会转发到tty1，而tty1的输出会转发到显示器，同理用输入ctrl+alt+F2，就会切换到tty2。

当模拟器激活tty时如果发现没有进程与之关联，意味着这是第一次打开该tty，于是会启动配置好的进程并和该tty绑定，一般该进程就是负责login的进程。

当切换到tty2后，tty1里面的输出会输出到哪里呢？tty1的输出还是会输出给模拟器，模拟器里会有每个tty的缓存，不过由于模拟器的缓存空间有限，所以下次切回tty1的时候，只能看到最新的输出，以前的输出已经不在了。

### 图形终端:
Ctrl + ALT + F7

### 伪终端(pseudoterminal slave):
使用ssh远程连接
在图形界面下打开命令提示符
表示： /dev/pts/#

### 物理终端（控制台console）:
表示： /dev/console

### 串行终端:
表示： /dev/ttyS#

### X window
```sh
startx &
startx -- :2 &
```

#### X server
X Server：这组程序主要负责的是屏幕画面的绘制与显示。X Server 可以接收来自 X client 的数据，将这些数据绘制呈现为图面在屏幕上。 此外，我们移动鼠标、点击数据、 由键盘输入数据等等，也会透过 X Server 来传达到 X Client 端，而由X Client 来加以运 算出应绘制的数据；


在X Window系统中，图形界面的数据主要是基于X协议的一系列请求和命令，这些请求和命令用于在X Server端创建和管理图形用户界面元素。这些数据可以包括但不限于以下内容：

绘图命令：
线条和形状绘制（如直线、弧线、矩形、多边形等）。
文本渲染命令，用于在界面上显示文本。
图像和位图传输命令，用于将图像数据从客户端传输到服务器。


窗口管理命令：
创建、移动、调整大小、隐藏和显示窗口。
更改窗口属性，如标题、图标、层次关系等。


事件处理：
键盘和鼠标输入事件的发送。
用户界面事件，如按钮点击、菜单选择、拖放操作等。


资源管理：
创建和使用字体、光标、颜色和图形上下文。
分配和释放这些资源。


剪贴板和选择：
处理文本和数据的复制、粘贴操作。
管理多个应用程序或窗口间的数据交换。


窗口缓冲和双缓冲：
管理窗口内容的缓冲，以减少屏幕闪烁和提高性能。
在某些情况下，使用双缓冲技术进行平滑的动画和视觉效果。


X协议中定义的这些数据和命令都是以一种网络传输友好的格式组织的，这样就可以在网络上高效地传输。在X11转发的情况下，这些命令通过SSH连接安全地从远程X Client发送到本地X Server，然后由X Server负责将它们转换成实际的图形输出。


#### X Client
X Client： 这组程序主要负责的是数据的运算。 X Client 在接受到 X Server 传来的数据后 (例如移动鼠标、点击 icon 等动作)，会经由本身的运算而得到鼠标应该要如何移动、点击的结果应该要出现什么样的数据、键盘输入的结果应该要如何呈现等等，然后将这些结果告知 X Server ，让他自行去绘制到屏幕上。

> **Tips**: 鸟哥常常开玩笑的说， X server 就是画布，而 X client 就是手拿画笔的画家。你得要先有画布 (管理好所有可显示的硬件后) 之后画家的想法 (计算出来的绘图数据) 才能够绘制到画布上！  
> **Server是提供"资源"的一方，而Client是使用"资源"的一方。**

在X Window系统中，X客户端通常是一个应用程序，负责以下主要任务：

用户界面创建：X客户端设计和创建用户界面的各个组件，比如窗口、菜单、按钮、文本框等。

事件处理：X客户端监听来自X服务器的事件，这些事件可能是用户的输入（如键盘敲击、鼠标点击）或来自其他系统事件（如窗口系统请求重绘窗口）。

用户交互：X客户端对用户的操作作出响应，并执行相应的逻辑处理，如打开文件、编辑文本或其他应用程序特定的任务。

图形渲染：虽然X客户端不直接渲染像素到屏幕，但它会发送绘图指令给X服务器，告诉服务器如何在屏幕上呈现图形元素。

资源管理：X客户端负责创建和管理它使用的资源，如颜色、字体、光标形状等，并将这些资源的使用与绘图请求相关联。

通信：X客户端通过X协议与X服务器通信，向服务器发送请求，接收服务器的响应和事件通知。

会话管理：X客户端可能参与会话管理，例如，保存和恢复应用程序的状态，以便在用户注销后再次登录时恢复工作环境。


在许多现代的X Window系统实现中，比如GNOME、KDE或其他桌面环境，X客户端通常不直接与X服务器交互，而是通过工具包（如GTK+或Qt）来进行，这些工具包封装了与X服务器通信的复杂性，并提供了更高级别的API来简化图形用户界面的创建和管理。

#### xserver 与 xclient 创建窗口
在X Window系统中，窗口的创建和管理遵循一个明确的客户端-服务器模型，其中X客户端和X服务器各自有不同的职责。

X客户端（X Client）：负责创建窗口的请求。这意味着实际上是由X客户端（如应用程序）来提出创建窗口的请求。X客户端定义窗口的大小、位置以及其他属性，并通过X协议将这些信息发送给X服务器。

X服务器（X Server）：负责实际创建和管理窗口。当X服务器接收到来自X客户端的创建窗口的请求后，服务器将在屏幕上创建窗口，并管理用户与窗口的交互，如键盘和鼠标输入事件的处理与分发。

简而言之，X客户端提出创建窗口的请求，但实际的窗口创建工作是由X服务器完成的。X服务器管理着屏幕、输入设备以及屏幕上显示的所有窗口，而X客户端通过X服务器提供的接口与这些资源进行交互。因此，尽管创建窗口的指令来自X客户端，窗口的实际创建和维护由X服务器负责。


**"请求者 + 模型"（X客户端）和"执行者 + 渲染"（X服务器）的概念描述了两者之间的关系和各自的职责**。这个模型清晰地分离了用户界面的设计和用户交互的处理（客户端的职责）与底层图形渲染和设备管理（服务器的职责）。

X客户端 (X Client)：作为请求者，它发送创建窗口、绘制图形等请求。同时，它也提供了“模型”，即应用程序的逻辑结构和用户界面设计。X客户端定义了用户交互的方式，以及用户界面应该如何响应这些交互。

X服务器 (X Server)：作为执行者，它接收并执行来自X客户端的请求，渲染图形输出到屏幕，并处理用户的输入事件。X服务器处理硬件层面的细节，如绘制像素、管理窗口缓冲区和输入设备等。


#### Window Manager
由于每一支 X client 都是独立存在的程序，因此在图形显示会发生一些迭图的问题 (想象一下每一个 X client 都是一个很自我的画家， 每个画家都不承认对方的存在，都自顾自的在画布上面作画，最后的结果会是如何？)。因此，后来就有一组特殊的 X client 在进行管理所有的其他 X client 程序，这个总管的咚咚就是 Window Manager！


在X Window系统中，窗口管理器（Window Manager）是一个特殊类型的X客户端，负责管理X服务器上显示的窗口的外观和行为。窗口管理器的主要职责包括：

窗口装饰：为窗口添加边框、标题栏、控制按钮（最小化、最大化和关闭按钮）等装饰元素。
窗口布局：控制窗口的位置和大小，以及它们在屏幕上的排列。这可能包括窗口的移动、调整大小、最小化、最大化等。
焦点管理：决定哪个窗口接收键盘和鼠标输入（即窗口的焦点）。
虚拟桌面：提供一种机制来切换不同的工作区或虚拟桌面，以便用户可以在不同的环境间切换。
用户交互：定义用户与窗口及桌面环境交互的方式，如键盘快捷键、鼠标手势等。

窗口管理器不是X Window系统的一个必需组件，但没有窗口管理器，用户就必须通过其他方式（如命令行）来管理窗口，这通常是不方便的。不同的窗口管理器提供不同的功能和用户体验，从而允许用户根据个人喜好和需求选择适合的窗口管理器。例如，有些窗口管理器专注于提供极简的环境，而有些则提供丰富的视觉效果和高级功能。


#### 窗口管理器在以下几个关键时刻介入
窗口创建：当X客户端请求创建一个新的窗口时，X服务器会通知窗口管理器。窗口管理器此时可以决定窗口的初始位置、大小以及是否要提供窗口装饰，如标题栏和边框。

窗口聚焦：当用户尝试与一个窗口交互时，如点击或双击，窗口管理器负责决定哪个窗口应当获得输入焦点。

窗口操作：对于用户执行的窗口操作，如移动、调整大小、最小化、最大化或关闭窗口，窗口管理器负责处理这些请求并反映相应的变化。

窗口重排：当用户在不同应用程序或窗口之间切换时，窗口管理器负责改变窗口的堆叠顺序。

系统或桌面快捷键：用户使用系统快捷键时，窗口管理器会响应这些快捷键并执行相应的操作，如打开启动菜单、切换窗口等。


在双击文件图标的场景中，窗口管理器可能在以下时刻介入：
响应双击事件：用户双击一个文件图标后，窗口管理器确保文件管理器窗口是活跃的，并且在前面显示，以便接收进一步的操作。

打开新窗口：如果双击操作导致一个新应用程序启动，窗口管理器会介入来管理新窗口的创建和显示，包括窗口的装饰、位置和大小。

调整焦点和层次：窗口管理器将确保新打开的窗口获得焦点，并且在必要时调整其他窗口的层次来保持窗口间的合适关系。

窗口管理器在整个用户界面和应用程序窗口生命周期中扮演着非常重要的角色，它确保了用户界面的一致性和直观性。

#### Display Manager
Display Manager (DM)：提供使用者登入的画面以让用户可以藉由图形接口登入。 在使用者登入后，可透过 display manager 的功能去呼叫其他的 Window manager ，让用户在图形接口的登入过程变得更简单。 由于 DM 也是启动一个等待输入账号密码的图形数据，因此 DM 会主动去唤醒一个 X Server 然后在上头加载等待输入的画面就是了。


通常启动图形接口让用户登入的方式中，都是先执行Display Manager 程序， 该程序会主动加载一个 X Server 程序，然后再提供一个等待输入账号密码的接口程序，之后再根据用户的选择去启动所需要的 Window Manager 程序，最后就由用户直接操作 WM 来玩图形接口。

![](images/compute_base/Simple-X-window-system-architecture.png)


#### 例子
在本地 鼠标双击文件 这个过程 xserver 与 xclient 是这样的：
```
用户 -> 双击文件 -> 鼠标事件产生

鼠标事件 -> 被X Server捕获

X Server -> 确定双击事件对应当前拥有焦点的X Client窗口（文件管理器）

X Server -> 将鼠标事件发送给文件管理器（X Client）

文件管理器（X Client） -> 接收事件 -> 解释为“打开文件”命令

文件管理器 -> 确定关联的应用程序（例如，文本编辑器）

文件管理器 -> 启动关联的应用程序（新的X Client）

新的应用程序（X Client） -> 请求X Server打开新窗口

X Server -> 创建并显示新窗口 -> 加载并显示文件内容
```



笔记本电脑上启动一个X Server

通过SSH连接到办公室服务器：
使用带有X11转发的SSH客户端连接到远程服务器。这通常涉及使用-X（或更安全的-Y）参数来启动SSH连接。
```bash
ssh -X [email protected]
```

在远程服务器上运行X Client应用程序：
一旦通过SSH连接到服务器，你可以启动一个图形界面应用程序（例如，gedit），它会作为X Client运行。
```bash
gedit &
```

在笔记本电脑上查看和交互应用程序：
gedit文本编辑器会在你的笔记本电脑上显示它的窗口，尽管它实际上是在远程服务器上运行的。你可以使用它就像它是在本地运行一样。


#### 将图形显示到其他主机
将图像显示到 localhost 这台主机上
```sh
export DISPLAY=localhost:0.0
xeyes
```

- 第一个0: 在(6000+0)这个端口上
- 第二个0: 表示连接到unix socket的路径 `/tmp/.X11-unix/X0`。这个几乎总是0

允许别的用户启动的图形程序将图形显示在当前屏幕上
```sh
xhost +
```

#### 将远程主机上的图像界面显示在本机上
![](images/compute_base/1.png)

在 X client computer上远程连接到 X server computer上面。
```sh
## -Y: enable X11 forwarding
ssh user1@Xserver -Y

xeyes
```

参考 [here](https://www.cyut.edu.tw/~ckhung/b/mi/xintro.php)
```sh
$ xauth list $DISPLAY
media-mc2.example.com:10  MIT-MAGIC-COOKIE-1  0d9d1431e67591364bce868773b1d7c0
```

```sh
$ xauth add media-mc2.example.com:10  MIT-MAGIC-COOKIE-1  0d9d1431e67591364bce868773b1d7c0

$ export DISPLAY=media-mc2.example.com:10
````

VNC
![](images/compute_base/vnc.png)


[^and-or-latch]: https://www.bilibili.com/video/BV1EW411u7th/?p=6&vd_source=5483606993f558f6ec3fbc3230b66c5d
[^add-xor-and]: https://www.youtube.com/watch?v=1I5ZMmrOfnA&list=PL8dPuuaLjXtNlUrzyH5r6jN9ulIgZBpdo&index=6

