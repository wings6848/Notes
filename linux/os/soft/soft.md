# 软件

既然我们的操作系统是一个软件，那么计算机是如何加载并运行这个“软件”呢？
换句话说，在计算机加电之后，发生了什么事情？

以Linux系统为例，它的启动过程，我们可以大致分为以下几个部分：

1. CPU加电瞬间将`CS:IP`置为`BIOS`的入口地址。
2. CPU执行跳转指令，将控制权交给`BIOS`。
3. `BIOS`进行`上电自检(POST)`，初始化设备等流程，最后加载操作系统。
4. `BIOS`将控制权交给操作系统，至此，启动过程完成。

> 在`i386`处理器架构下，CPU加电瞬间会将`CS:IP`置为`0XF000:0XFFF0`，也就代表着`BIOS`的入口地址是`0XFFFF0`。

参考文章：

1. [计算机启动过程](http://blog.kongfy.com/2014/03/%E8%AF%91%E8%AE%A1%E7%AE%97%E6%9C%BA%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B-how-computers-boot-up/)

## BIOS

`BIOS`即`Basic Input Output System`(基本输入输出系统)，可以认为`BIOS`是计算机启动时加载的第一个软件。

`BIOS`的主要功能包括：

1. `POST`：`Power Of Self Test`（上电自检）检测CPU各寄存器、计时芯片、中断芯片，`DMA`控制器等
2. `Initial`：枚举设备，初始化寄存器，分配中断、IO端口、`DMA`资源等。
3. `Setup`：进行系统设置，例如开机时选择进入`BIOS`的设置界面。
4. `Runtime Program`：将常驻程序库（Runtime Program）常驻于某一段内存，供操作系统或应用程序调用，如：`INT 10h、INT 13h、INT 15h`（即我们常说的BIOS中断）。
5. `启动自举程序`：在`POST`过程结束后，将调用`INT 19h`，启动自举程序，自举程序将读取引导记录，装载操作系统。

> DMA：即`Direct Memory Access`（直接内存存取），是一种不经过CPU，而直接从内存存取数据的数据交换模式。

## Bootloader

简单来说，BootLoader就是在操作系统内核运行之前运行的一段小程序。
通过这段小程序，我们可以初始化硬件设备，建立内存空间的映射图，
从而将系统的软硬件带到一个合适的状态，以便为最终调用操作系统内核准备好正确的环境。

当机器引导他的操作系统时，BIOS会读取引导介质上最前面的512字节（即人们所知的 主引导记录（master boot record，MBR）），
为了能达到启动内核的目的，BootLoader必须具备以下功能：

1. 初始化RAM

    Bootloader必须设置和初始化RAM，为调用Linux内核做好准备。
    初始化RAM的任务包括设置CPU的控制寄存器参数，以便能正常使用RAM以及检测RAM大小等。

2. 初始化串口端口

    在Linux的启动过程中有着非常重要的作用，它是Linux内核和用户交互的方式之一。

3. 检测处理器类型

    Bootloader在调用Linux内核前必须检测系统的处理器类型，并将其保存到某个常量中提供给Linux内核。
    Linux内核在启动过程中会根据该处理器类型调用相应的初始化程序。

4. 设置Linux启动参数

    Bootloader在执行过程中必须设置和初始化Linux的内核启动参数。

5. 调用Linux内核映像

    Bootloader完成的最后一项工作便是调用Linux内核。
    一般的嵌入式系统都是将Linux内核拷贝到RAM中，然后跳转到RAM中去执行。

参考文章：

1. [嵌入式系统 BootLoader](https://www.ibm.com/developerworks/cn/linux/l-btloader/index.html)
2. [了解LILO与GRUB](https://www.ibm.com/developerworks/cn/linux/l-bootload.html)
3. [Linux系统下的bootloader](https://blog.csdn.net/xiewenhao12/article/details/52924079)

## Kernel

操作系统为了能够更好的管理计算机，并且对应用程序提供便捷的服务，在操作系统发展的过程中，出现了以下四个抽象：

1. 中断（Interrupt）
2. 进程（Process）
3. 虚存（Virtual Memory）
4. 文件（File）