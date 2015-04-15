# lab4 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 总体介绍

(1) ucore的线程控制块数据结构是什么？
 proc_struct 
</br>
### 关键数据结构

(2) 如何知道ucore的两个线程同在一个进程？
看parent是否是同一个。
</br>
(3) context和trapframe分别在什么时候用到？
进程切换时使用。
</br>
(4) 用户态或内核态下的中断处理有什么区别？在trapframe中有什么体现？

### 执行流程

(5) do_fork中的内核线程执行的第一条指令是什么？它是如何过渡到内核线程对应的函数的？
```
tf.tf_eip = (uint32_t) kernel_thread_entry;
/kern-ucore/arch/i386/init/entry.S
/kern/process/entry.S
```
pushl %edx 
</br>
(6)内核线程的堆栈初始化在哪？
```
tf和context中的esp
```
setup_kstack 
</br>
(7)fork()父子进程的返回值是不同的。这在源代码中的体现中哪？

(8)内核线程initproc的第一次执行流程是什么样的？能跟踪出来吗？

## 小组练习与思考题

(1)(spoc) 理解内核线程的生命周期。

> 需写练习报告和简单编码，完成后放到git server 对应的git repo中

### 掌握知识点
1. 内核线程的启动、运行、就绪、等待、退出
2. 内核线程的管理与简单调度
3. 内核线程的切换过程

### 练习用的[lab4 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab4/lab4-spoc-discuss)


请完成如下练习，完成代码填写，并形成spoc练习报告

### 1. 分析并描述创建分配进程的过程

> 注意 state、pid、cr3，context，trapframe的含义
在kern_init里调用了proc_init初始化了进程表，在proc_init里创建了两个内核线程，仔细看这个函数，发现先是建立了一个内核进程，设置state为PROC_RUNNABLE，之后两次调用kernel_thread创建两个内核线程，其中init_main被调用，而schedule在init_main调用，schedule调用了proc_run，在proc_run里调用load_esp0切换内核堆栈，调用lcr3切换页表，再调用switch_to切换进程。

### 练习2：分析并描述新创建的内核线程是如何分配资源的

> 注意 理解对kstack, trapframe, context等的初始化
在kernel_thread里调用do_fork，do_fork主要实现了内核线程的资源分配，通过alloc_proc分配TCB，setup_kstack为子线程分配内核堆栈，然后copy_thread创建trapframe和context，再用list_add_before将TCB插入链表，最后调用wake_up唤醒它。

当前进程中唯一，操作系统的整个生命周期不唯一，在get_pid中会循环使用pid，耗尽会等待

### 练习3：阅读代码，在现有基础上再增加一个内核线程，并通过增加cprintf函数到ucore代码中
能够把进程的生命周期和调度动态执行过程完整地展现出来
> 
```
struct proc_struct *initproc3 = NULL;
void
proc_init(void) {
    int i;

    list_init(&proc_list);

    if ((idleproc = alloc_proc()) == NULL) {
        panic("cannot alloc idleproc.\n");
    }

    idleproc->pid = 0;
    idleproc->state = PROC_RUNNABLE;
    idleproc->kstack = (uintptr_t)bootstack;
    idleproc->need_resched = 1;
    set_proc_name(idleproc, "idle");
    nr_process ++;

    current = idleproc;

    int pid1= kernel_thread(init_main, "init main1: Hello world!!", 0);
    int pid2= kernel_thread(init_main, "init main2: Hello world!!", 0);
    int pid3= kernel_thread(init_main, "init main3: Hello worldx!", 0);
    if (pid1 <= 0 || pid2<=0 || pid3<=0) {
        panic("create kernel thread init_main1 or 2 or 3 failed.\n");
    }

    initproc1 = find_proc(pid1);
	initproc2 = find_proc(pid2);
    initproc3 = find_proc(pid3);
    set_proc_name(initproc1, "init1");
	set_proc_name(initproc2, "init2");
    set_proc_name(initproc3, "init3");
    cprintf("proc_init:: Created kernel thread init_main--> pid: %d, name: %s\n",initproc1->pid, initproc1->name);
	cprintf("proc_init:: Created kernel thread init_main--> pid: %d, name: %s\n",initproc2->pid, initproc2->name);
    cprintf("proc_init:: Created kernel thread init_main--> pid: %d, name: %s\n",initproc3->pid, initproc3->name);
    assert(idleproc != NULL && idleproc->pid == 0);
}
```
### 练习4 （非必须，有空就做）：增加可以睡眠的内核线程，睡眠的条件和唤醒的条件可自行设计，并给出测试用例，并在spoc练习报告中给出设计实现说明

### 扩展练习1: 进一步裁剪本练习中的代码，比如去掉页表的管理，只保留段机制，中断，内核线程切换，print功能。看看代码规模会小到什么程度。


