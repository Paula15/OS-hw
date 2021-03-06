# OS作业1

## 填空题

* 当前常见的操作系统主要用【C++, 汇编】编程语言编写。
* "Operating system"这个单词起源于【Operator】。
* 在计算机系统中，控制和管理【资源】、有效地组织【多道程序】运行的系统软件称作【操作系统】。
* 允许多用户将若干个作业提交给计算机系统集中处理的操作系统称为【批处理】操作系统
* 你了解的当前世界上使用最多的32bit CPU是【ARM】，其上运行最多的操作系统是【Android】。
* 应用程序通过【系统调用】接口获得操作系统的服务。
* 现代操作系统的特征包括【并行性】，【虚拟性】，【异步性】，【共享性】，【持久性】。
* 操作系统内核的架构包括【宏内核】，【微内核】，【外核】。


## 问答题

- **请总结你认为操作系统应该具有的特征有什么？并对其特征进行简要阐述。**

  特征：

  1. 虚拟性：
     - 操作系统将许多资源（如数据、设备等）抽象成易于使用的形式（如文件、由驱动程序驱动的虚拟设备等），提供了统一的接口，方便了编程。
     - 此外，操作系统使用虚拟存储技术，可以使用硬盘对内存的功能加以拓展，而这对用户是透明的。
  2. 并发性：
     - 多个程序通过分时等技术，可以对用户体现出“同时”运行的假象，提高了交互性与调度效率。
  3. 共享性：
     - 同并发性。
     - 分布式系统中的所有资源可供系统中的所有用户共享。
  4. 异步性：
     - 即使程序执行速度、步调、时间均不可预知，作业多次执行的结果应当相同（除非程序执行结果的确应当与时间相关）。
  5. 持久性：
     - 数据、操统本身的代码都可以装入磁盘等永久性存储介质中存储。


- **为什么现在的操作系统基本上用C语言来实现？为什么没有人用python，java来实现操作系统？**
  1. 效率：相同功能的程序，C语言的执行效率通常高于Java，远高于Python。
  2. 依赖性：C程序编译后可直接得机器语言，无需依赖即可运行；而Java依赖于JVM，Python依赖于Python解释器。
  3. 底层性：C语言较为底层，可以较方便地操作指针、内联汇编等。

## 可选练习题

- 请分析并理解[v9\-computer](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)以及模拟v9\-computer的em.c。理解：在v9\-computer中如何实现时钟中断的；v9 computer的CPU指令，关键变量描述有误或不全的情况；在v9\-computer中的跳转相关操作是如何实现的；在v9\-computer中如何设计相应指令，可有效实现函数调用与返回；OS程序被加载到内存的哪个位置,其堆栈是如何设置的；在v9\-computer中如何完成一次内存地址的读写的；在v9\-computer中如何实现分页机制；


- 请编写一个小程序，在v9-cpu下，能够输出字符


- 输入的字符并输出你输入的字符


- 请编写一个小程序，在v9-cpu下，能够产生各种异常/中断


- 请编写一个小程序，在v9-cpu下，能够统计并显示内存大小

