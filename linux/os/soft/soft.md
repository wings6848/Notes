# 软件

既然我们的操作系统是一个软件，那么计算机是如何加载并运行这个“软件”呢？
换句话说，在计算机加电之后，发生了什么事情？

以Linux系统为例，它的启动过程，我们可以大致分为以下几个部分：

1. CPU加电瞬间将`CS:IP`置为`BIOS`的入口地址。
2. CPU执行跳转指令，将控制权交给`BIOS`。
3. `BIOS`进行`上电自检(POST)`，初始化设备等流程，最后加载操作系统。
4. `BIOS`将控制权交给操作系统，至此，启动过程完成。

> 在`i386`处理器架构下，CPU加电瞬间会将`CS:IP`置为`0XF000:0XFFF0`，也就代表着`BIOS`的入口地址是`0XFFFF0`。

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

## Kernel

操作系统为了能够更好的管理计算机，并且对应用程序提供便捷的服务，在操作系统发展的过程中，出现了以下四个抽象：

1. 中断（Interrupt）
2. 进程（Process）
3. 虚存（Virtual Memory）
4. 文件（File）