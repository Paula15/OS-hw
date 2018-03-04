# OS作业2

## 思考题

- **你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？**
  1. 硬件支持：
     - CP0寄存器：提供多个寄存器，用于保存CPU的当前状态。切换进程、重填TLB等，都需要CP0的参与。
     - 通用寄存器：程序运行需要。
     - MMU：需要保证不同进程之间虚拟地址空间独立，其中可能集成TLB功能用于快速查询。
  2. 特权指令（如MIPS）：
     - MFC0、MTC0：读写CP0。
     - TLBWI、TLBWR：写TLB表项。
     - ERET：异常返回，因为新建进程时用到`fork`函数属于系统调用，会触发SYSCALL异常；虚存管理所用TLB可能触发TLB MISS异常。
- **你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？**
  1. 区别：
     - 实模式：无虚拟地址转换机制，不区分用户程序和系统程序的权限，无内存访问保护。
     - 保护模式：有虚拟地址转换机制（寻址空间为$2^{32}=4G$），支持多任务，有特权检查，物理内存地址不能直接被程序访问（需由操统地址映射）
  2. 含义：
     - 物理地址：是处理器提交到总线上用于访问计算机系统中的内存和外设的最终地址。
     - 逻辑地址：在有地址变换功能的计算机中，访问指令给出的地址。是相对当前进程数据段的地址。只有在Intel实模式下，逻辑地址才和物理地址相等。
     - 线性地址：逻辑地址到物理地址变换之间的中间层，处理器通过段（Segment）机制控制下的形成的地址空间。逻辑地址 + 段的基地址 = 线性地址。
- **理解list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）**
- **对于如下的代码段，请说明":"后面的数字是什么含义**
```c
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;              // segment selector
    unsigned gd_args : 5;             // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;             // reserved(should be zero I guess)
    unsigned gd_type : 4;             // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;              // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;       // high bits of offset in segment
 };
```

代表该field占据多少位：参考资料[Bit Field](http://en.cppreference.com/w/cpp/language/bit_field)。

实验：

```c
#include <stdio.h>
#include <memory.h>

/* struct gatedesc ... */

int main() {
    struct gatedesc a;
    memset(&a, -1, sizeof(a));
    printf("%u %u %u %u %u %u %u %u %u\n",
        a.gd_off_15_0,
        a.gd_ss,
        a.gd_args,
        a.gd_rsv1,
        a.gd_type,
        a.gd_s,
        a.gd_dpl,
        a.gd_p,
        a.gd_off_31_16);
    return 0;
}

/* 输出：65535 65535 31 7 15 1 3 1 65535 */
```

- 对于如下的代码段，

```c
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr = 8;
SETGATE(intr, 1, 2, 3, 0);
```
**请问执行上述指令后， `intr`的值是多少？**

由于`gd_off_15_0`和`gd_ss`两个field一共为32位，且位于结构体的低位，而unsigned类型也只有32位，故实际上只需看这2个field即可。

执行`SETGATE`后，`gd_off_15_0`为0x0003，`gd_ss`为0x0002，故`intr`的值应为0x20003。

## 课堂实践练习

**请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。**

```asm
# vectors.S sends all traps here.
.text
.globl __alltraps
__alltraps:
    # push registers to build a trap frame
    # therefore make the stack look like a struct trapframe
    pushl %ds
    pushl %es
    pushl %fs
    pushl %gs
    pushal

    # load GD_KDATA into %ds and %es to set up data segments for kernel
    movl $GD_KDATA, %eax
    movw %ax, %ds
    movw %ax, %es

    # push %esp to pass a pointer to the trapframe as an argument to trap()
    pushl %esp

    # call trap(tf), where tf=%esp
    call trap

    # pop the pushed stack pointer
    popl %esp
```

首先将多个寄存器压栈，并设置一些重要寄存器，以形成`trapframe`结构。

其次用栈上的`trapframe`结构体地址（也即栈指针`%esp`）作参数调用`trap`函数，完成异常处理。

最后弹出`%esp`，调用结束。

参考资料：

  - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
  - [x86汇编指令集](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
  - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
  - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
  - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)


**宏定义和引用在内核代码中很常用。请枚举ucore中宏定义的用途，并举例描述其含义。**

1. 复杂数据结构中的数据访问/数据类型转换：

   ```c
   #define SETGATE(gate, istrap, sel, off, dpl) {            \
       (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
       (gate).gd_ss = (sel);                                \
       (gate).gd_args = 0;                                    \
       (gate).gd_rsv1 = 0;                                    \
       (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
       (gate).gd_s = 0;                                    \
       (gate).gd_dpl = (dpl);                                \
       (gate).gd_p = 1;                                    \
       (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
   }
   ```

2. 常用功能的封装。

   ​