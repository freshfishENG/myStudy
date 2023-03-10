# 内存管理
现代操作系统的一大特性: 多任务，并行化。
带来问题: 如何优雅的调度内存？
![][image-1]

能存容量是有限的, 能用多久？解决方式 : 虚拟内存!
## CPU主流工作模式
**CPU工作模式：是指CPU的寻址方式、寄存器大小、指令用法和内存布局等概念的集合。**
### 实模式和保护模式(以x86系列cpu为例)
### 实模式 :
8086时代CPU唯一的工作模式，所谓”实”, 代表程序中所有的内存地址都是真实地址, 下图实模式内存寻址的流程
![][image-2]

段地址由段基址+段内偏移地址组成

![][image-3]

**最大寻址空间 ： 2^(16  + 4) = 1M, 最大分段 2^16 = 64K**
为什么段地址只有16位? 因为寄存器16位，但是地址总线宽度20位, 如何做到匹配？20位的物理地址高四位取决于端地址的高四位, 低16位取决偏移量。
![][image-4]

注意 : 在保护模式下, cs会存储当前cpu的状态(内核/用户)
存在问题: 1、内存每段大小不固定, 产生碎片后管理问题。2、物理地址没有对用户透明, 可以直接操作物理地址, 安全性问题
### 保护模式 (分页几乎取代分段)
寻址流程
![][image-5]
#### 重新认识段
如何通过段定位到一个线性地址 ？段基地址(下图中的**Base Address**, 32位) + 偏移offset(32位)
1. 段描述符(每个8字节)
 ![][image-6] ![][image-7]
- Limit : 该段的大小
-  Base : 该段映射的线性地址中第一个线性地址
- G : 粒度。0，该段大小以字节为单位，否则4096字节
- Type : 段的类型特征
- DPL : 该段描述符的访问权限。(0 : 内核态；3 : 用户态)
- P : 段是否在主存。(linux永远为1, 因为linux不会把段放到磁盘)
- D/B : 代码/数据
- AVL : linux忽略
2. 如何快速定位到段描述符
	所有的段描述符存储在内存中某块(全局描述表(GDT))中, 局部描述符表(LDT)已被linux抛弃, 不做介绍 。GDT本质就是一个数组，数组每个元素符即是一个段描述符。
- 段选择符
 ![][image-8]
    - index : 13位，即需要段描述符在GDT中的下标。因为0不用, 所以一个段表中最多存储2^13 - 1个段描述符
    - TI : 1位, GDT/LDT, linux永远是GDT
    - RPL : 2位, 当前段选择器符的权限(linux中仅有0 : 内核态, 3 : 用户态)
    每个段寄存器存储当前cpu所用的段选择符, 由index x 8 + offset = 段描述符在GDT中的位置。
    每次去内存拿段描述符，性能问题 ？如下图，针对每个段寄存器，有一个对应的对用户透明的寄存器，存储段描述符。当段描述符与cpu需要的不对应时，才去内存拿。
![][image-9]
  
3. 分段单元总结

	![][image-10]

	1、linux中段的具体实现 (仅在80x86结构下linux才需要使用分段)

![][image-11]

1、由上图发现，当linux位于保护模式下，无论是内核态还是用户态，线性地址=逻辑地址，因为线性地址的起始位置都为0，只跟用户设置的**虚拟地址**有关。

2、事实上，在用户态或者内核态时，linux根本不需要特地去设置寄存器的段选择符，直接将linux内置的相关的宏装载到寄存器中。

**当然在linux启动阶段，还处在实模式下还会有几种段，有兴趣的可以自己看看**

[image-1]:	pic/1.png
[image-2]:	pic/2.png
[image-3]:	pic/3.png
[image-4]:	pic/4.png
[image-5]:	pic/5.png
[image-6]:	pic/6.png
[image-7]:	pic/7.png
[image-8]:	pic/8.png
[image-9]:	pic/9.png
[image-10]:	pic/10.png
[image-11]:	pic/11.png