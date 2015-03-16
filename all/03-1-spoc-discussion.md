# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy systemm分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
  + 采分点：说明四种算法的优点和缺点
  - 答案没有涉及如下3点；（0分）
  - 正确描述了二种分配算法的优势和劣势（1分）
  - 正确描述了四种分配算法的优势和劣势（2分）
  - 除上述两点外，进一步描述了一种更有效的分配算法（3分）
 ```
- [x]  

> 最优匹配：优势：避免大的分区被拆分，从而减少外碎片的尺寸。   劣势：释放分区较为复杂，因为合并时是去找地址临近的；另外剩下的边角料也大小少了，导致外碎片增多。</br>
最差匹配：优势：匹配的速度快，对中等尺寸分配较多的效果较好，剩下的空间仍然可以被利用，减少了外碎片。
劣势：释放分区也较为复杂，而且由于每次分配都是找最大块，导致大块的空间减少，以致以后可能不能分配大的内存块。</br>
最先匹配：优势：释放分区的合并花费时间较少，把大的空闲块留到了高地址，因此后面申请大块时总能找着。
劣势：由于每次都是从前往后查找分配，导致低地址的外碎片较多，当分配大些的块时不得不从前往后依次查找，造成时间开销越来越大。</br>
buddy system：优势：比较好地折中了合并和分配块的位置碎片的问题。
劣势：仍然有外碎片、内碎片。  
另外一种连续内存分配算法slab算法：类似于“对象池”，通过分析评估申请释放内存造成的碎片，算法用于同种类型的对象分配连续内存空间，但只适用于内存。


## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)

---
>2012011392做的是0：最优匹配。方法如下：  
>维护一个大的字节数组作为内存，然后还维护两个链表，分别是空闲链表和已申请的空间链表，分别记录内存的申请和空闲情况，避免在内存块上置标志位，使得合并也容易些。代码见下：
主体：memoryManager.h  
\#ifndef MEMORYMANAGER_H  
\#define MEMORYMANAGER_H  
  
\#define N 1000  
typedef unsigned char uint8;  
  
struct Block  
{  
	int address;  
	int size;  
	Block(int a,int s)  
	:address(a),size(s){}  
};  
  
struct Node  
{  
	Block block;  
	Node* prev;  
	Node* next;  
	Node(const Block& b=Block(0,0),Node* p=NULL,Node* n=NULL)  
		:block(b),prev(p),next(n){}  
};  
  
class List  
{  
 private:  
	Node* head;  
	Node* tail;  
	int size;  
 public:  
	List()  
	{  
		size=0;  
		head=new Node();  
		tail=new Node();  
		head->next=tail;  
		tail->prev=head;  
	}  
	  
	~List()  
	{  
		while(!empty())  
			erase(begin());  
		delete head;  
		delete tail;  
	}  
	  
	Node* begin() const  
	{  
		return head->next;  
	}  
	  
	Node* end() const  
	{  
		return tail;  
	}  
	  
	Node* insert(Node* p,const Block x)				//insert before p  
	{  
		 size++;  
                 return (p->prev=p->prev->next=new Node(x,p->prev,p));  
	}  
	  
	Node* insert(const Block x)							//maintain the sorted List  
	{  
		for(Node* p=begin();p!=tail;p=p->next)  
		{  
			if(x.size <= p->block.size)  
			{  
				return insert(p,x);  
			}  
		}  
		return insert(end(),x);  
	}  
	  
	Node* erase(Node* p)  
	{  
		p->prev->next=p->next;  
		p->next->prev=p->prev;  
		Node* q=p->next;  
		size--;  
		delete p;  
		return q;  
	}  
	
	inline bool empty() const  
	{  
		return size==0;  
	}  
};  
  
class MemoryManager  
{  
 public:  
	MemoryManager()  
	{  
		memset(memory,0,sizeof(uint8)*N);  
		freeList=new List;  
		freeList->insert(Block(0,N));  
		(freeList->end())->block.address=N;			//the tail of the list, size=0  
		busyList=new List;  
	}  
	~MemoryManager()  
	{  
		delete freeList;  
		delete busyList;  
	}  
	void* memAlloc(int size)  
	{  
		for(Node* p=freeList->begin();p!=freeList->end();p=p->next)  
		{  
			if(p->block.size >= size)  
			{  
				Block b=p->block;  
				busyList->insert(Block(b.address,size));  
				uint8* rp=memory+b.address;  
				b.address+=size;  
				b.size=p->block.address+p->block.size-b.address;  
				freeList->erase(p);  
				freeList->insert(b);  
				return (void*)rp;  
			}  
		}  
		return NULL;  
	}  
	bool memFree(void* bp)  
	{  
		int addr=(uint8*)bp - memory;  
		  
		for(Node* p=busyList->begin();p!=busyList->end();p=p->next)  
		{  
			if(p->block.address == addr)  
			{  
				Block b=p->block;				//merge  
				for(Node* q=freeList->begin();q!=freeList->end();q=q->next)  
				{  
					if(q->block.address+q->block.size == b.address)  
					{  
						b.address=q->block.address;  
						b.size=p->block.address+p->block.size-b.address;  
						freeList->erase(q);  
						break;  
					}  
				}  
				for(Node* r=freeList->begin();r!=freeList->end();r=r->next)  
				{  
					if(p->block.address+p->block.size == r->block.address)  
					{  
						b.size=r->block.address + r->block.size-b.address;  
						freeList->erase(r);  
						break;  
					}  
				}  
				busyList->erase(p);  
				freeList->insert(b);  
				return true;  
			}  
		}  
		return false;  
	}  
	  
	void print()  
	{  
		std::cout<<"\nfreeList\n";  
		for(Node* p=freeList->begin();p!=freeList->end();p=p->next)  
		{  
			std::cout<<"("<<p->block.address<<","<<(p->block.address+p->block.size)<<") ";  
		}  
		std::cout<<"\nbusyList:\n";  
		for(Node* p=busyList->begin();p!=busyList->end();p=p->next)  
		{  
			std::cout<<"("<<p->block.address<<","<<(p->block.address+p->block.size)<<") ";  
		}  
		std::cout<<"\n-------------";  
	}  
	
 private:  
	uint8 memory[N];  
	List* freeList;  
	List* busyList;  
};  
  
\#endif  
  
cpp文件bestMatching.cpp给出了一些测试样例：  
\#include <iostream>  
\#include "memoryManager.h"  
using namespace std;  
int main()  
{  
	MemoryManager mgr;  
	int* A=(int*)mgr.memAlloc(100);  
	mgr.print();  
	char* B=(char*)mgr.memAlloc(2);  
	mgr.print();  
	  
	double* C=(double*)mgr.memAlloc(sizeof(double)*50);  
	mgr.print();  
  	
	int* D=(int*)mgr.memAlloc(sizeof(int)*50);  
	mgr.print();  
	  
	int* E=(int*)mgr.memAlloc(600);  
	cout<<E<<endl;  
	mgr.print();  
	  
	mgr.memFree(A);  
	mgr.print();  
	mgr.memFree(C);  
	mgr.print();  
	mgr.memFree(B);  
	mgr.print();  
	return 0;  
}  
  
以下是测试结果：
freeList  
(100,1000)  
busyList:  
(0,100)  
-------------  
freeList  
(102,1000)  
busyList:  
(100,102) (0,100)  
-------------  
freeList  
(502,1000)  
busyList:  
(100,102) (0,100) (102,502)  
-------------  
freeList  
(702,1000)  
busyList:  
(100,102) (0,100) (502,702) (102,502)  
-------------0  
  
freeList  
(702,1000)  
busyList:  
(100,102) (0,100) (502,702) (102,502)  
-------------  
freeList  
(0,100) (702,1000)  
busyList:  
(100,102) (502,702) (102,502)  
-------------  
freeList  
(0,100) (702,1000) (102,502)  
busyList:  
(100,102) (502,702)  
-------------  
freeList  
(702,1000) (0,502)  
busyList:  
(502,702)  
-------------  
  
  

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

- [x]  

>  http://en.wikipedia.org/wiki/Memory_management_unit

L1和L2高速缓存有什么区别？

- [x]  

>  http://superuser.com/questions/196143/where-exactly-l1-l2-and-l3-caches-located-in-computer
>  Where exactly L1, L2 and L3 Caches located in computer?

>  http://en.wikipedia.org/wiki/CPU_cache
>  CPU cache

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  

为什么最先匹配会越用越慢？

- [x]  

>  

为什么最差匹配会的外碎片少？

- [x]  

>  

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  

### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  



