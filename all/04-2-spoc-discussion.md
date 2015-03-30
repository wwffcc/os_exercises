#lec9 虚存置换算法spoc练习

## 个人思考题
1. 置换算法的功能？

2. 全局和局部置换算法的不同？

3. 最优算法、先进先出算法和LRU算法的思路？

4. 时钟置换算法的思路？

5. LFU算法的思路？

6. 什么是Belady现象？

7. 几种局部置换算法的相关性：什么地方是相似的？什么地方是不同的？为什么有这种相似或不同？

8. 什么是工作集？

9. 什么是常驻集？

10. 工作集算法的思路？

11. 缺页率算法的思路？

12. 什么是虚拟内存管理的抖动现象？

13. 操作系统负载控制的最佳状态是什么状态？

## 小组思考题目

----
(1)（spoc）请证明为何LRU算法不会出现belady现象


(2)（spoc）根据你的`学号 mod 4`的结果值，确定选择四种替换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试。请参考如python代码或独自实现。
 - [页置换算法实现的参考实例](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab3/page-replacement-policy.py)
 
 > 代码（LRU.cpp):
 
 ```
    #include <stdio.h>

    #define N 5

    struct Page{
        int val;
        Page* prev;
        Page* next;
        Page(int v=-1 ,Page* p=NULL, Page* n=NULL)
            :val(v),prev(p),next(n){}
    };

    Page* p_header;
    Page* p_tail;
    int size = 0;

    Page* insert(Page* ptr, int val)
    {
         return ptr->prev =ptr->prev->next = new Page(val,ptr->prev,ptr);
    }

    Page* erase(Page* ptr)
    {
        ptr->prev->next = ptr->next;
        ptr->next->prev = ptr->prev;
        Page* n = ptr->next;
        delete ptr;
        return n;
    }

    void swap(Page* p1,Page* p2)
    {
        int val = p1->val;
        p1->val=p2->val;
        p2->val=val;
    }

    void LRU(int val)
    {
        Page* ptr;
        for(ptr = p_header->next;ptr!=p_tail;ptr=ptr->next)
            if(ptr->val == val)
                break;
        if(ptr != p_tail)
        {
            for(Page* p=ptr;p!=p_tail->prev;p=p->next)        
                swap(p,p->next);
        }
        else
        {
            if(size!=N)
            {
                insert(p_tail,val);
                size++;
            }
            else
            {
               erase(p_header->next);
               insert(p_tail,val);
            }
        }
    }

    void printList()
    {
        for(Page* p=p_header->next ;p!=p_tail; p=p->next)
            printf("%d ",p->val);
        printf("\n");
    }

    int main()
    {
        p_header = new Page;
        p_tail = new Page;
        p_header->next=p_tail;
        p_tail->prev=p_header;   
        size=0;
        int d;
        while(true)
        {
            scanf("%d",&d);
            if(d==-1)
                return 0;
            LRU(d);
            printList();
        }
        return 0;
    }
 ```
 
 测试结果：
 ```
    1
    1 
    2
    1 2 
    3
    1 2 3 
    4
    1 2 3 4 
    5
    1 2 3 4 5 
    6
    2 3 4 5 6 
    3
    2 4 5 6 3 
    5
    2 4 6 3 5 
    2
    4 6 3 5 2 
    9
    6 3 5 2 9 
    3
    6 5 2 9 3 
    4
    5 2 9 3 4 
    1
    2 9 3 4 1 
    2
    9 3 4 1 2 
    1
    9 3 4 2 1 

 ```
 
## 扩展思考题
（1）了解LIRS页置换算法的设计思路，尝试用高级语言实现其基本思路。此算法是江松博士（导师：张晓东博士）设计完成的，非常不错！

参考信息：

 - [LIRS conf paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang02_LIRS.pdf)
 - [LIRS journal paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang05_LIRS.pdf)
 - [LIRS-replacement ppt1](http://dragonstar.ict.ac.cn/course_09/XD_Zhang/(6)-LIRS-replacement.pdf)
 - [LIRS-replacement ppt2](http://www.ece.eng.wayne.edu/~sjiang/Projects/LIRS/sig02.ppt)
