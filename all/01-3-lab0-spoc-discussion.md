# lab0 SPOC思考题

## 个人思考题

---

能否读懂ucore中的AT&T格式的X86-32汇编语言？请列出你不理解的汇编语言。
- [x]  

>  http://www.imada.sdu.dk/Courses/DM18/Litteratur/IntelnATT.htm
全部都在《汇编语言程序设计》中学过，都是看得懂的。

虽然学过计算机原理和x86汇编（根据THU-CS的课程设置），但对ucore中涉及的哪些硬件设计或功能细节不够了解？
- [x]  

>我对实模式和保护模式不够了解，当时学习汇编语言和计算机原理对这个也没有深究。对分段和分页，还停留在理论，没有真正动手实现过，因此也不是很熟悉。   


哪些困难（请分优先级）会阻碍你自主完成lab实验？
- [x]  

>实验环境的熟悉>理论算法的理解>C语言的技巧。   

如何把一个在gdb中或执行过程中出现的物理/线性地址与你写的代码源码位置对应起来？
- [x]  

>可以在代码源码处设置断点，这样gdb运行到断点处，就会显示其运行的内存地址。

了解函数调用栈对lab实验有何帮助？
- [x]  

>了解函数调用栈，可以对进程内存管理、调度有较好的认识。

你希望从lab中学到什么知识？
- [x]  

>学习运用C语言编写系统的方法技巧，能够自己实现操作系统中的算法,加深对理论的理解，提高动手能力。

---

## 小组讨论题

---

搭建好实验环境，请描述碰到的困难和解决的过程。
- [x]  

>问题：VirtualBox里竟然没有64位选项。
解决过程：上网了解到可能是PC虚拟化没有开，于是在bios配置中将Virtualization改为enabled，就出现了64位的。

熟悉基本的git命令行操作命令，从github上
的 http://www.github.com/chyyuu/ucore_lab 下载
ucore lab实验
- [x]  

> 首先创建一个目录mkdir test；  
然后cd test；  
然后，用git clone http://www.github.com/chyyuu/ucore_lab即可从源上下载文件到test中。

尝试用qemu+gdb（or ECLIPSE-CDT）调试lab1
- [x]   

> make clean清除make生成的文件；  
然后make即可编译程序；  
用make qemu即可运行程序。  
至于gdb，可以make debug；  
然后就可以出现gdb，在其中可以设端点break，以及print查看变量值。

对于如下的代码段，请说明”：“后面的数字是什么含义
```
/* Gate descriptors for interrupts and traps */
struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
};
```

- [x]  

> 指的是数据所占的位数。

对于如下的代码段，
```
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
intr=8;
SETGATE(intr, 0,1,2,3);
```
请问执行上述指令后， intr的值是多少？

- [x]  

>65538 

请分析 [list.h](https://github.com/chyyuu/ucore_lab/blob/master/labcodes/lab2/libs/list.h)内容中大致的含义，并能include这个文件，利用其结构和功能编写一个数据结构链表操作的小C程序
- [x]  

> 

---

## 开放思考题

---

是否愿意挑战大实验（大实验内容来源于你的想法或老师列好的题目，需要与老师协商确定，需完成基本lab，但可不参加闭卷考试），如果有，可直接给老师email或课后面谈。
- [x]  

>  

---
