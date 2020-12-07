以VC6为例，VC10小区块的操作已经潜到操作系统  
[main函数的前后操作]  
main()是c++的接入函数  
mainCRTStartup():gcc也有类似操作，c runtime   
_heap_init():  
crt_heap:先要4096的堆大小初始化crt_heap  
sbh_heap:再初始化16个sbh_heap的Header，用来做内存管理(crt_heap)      
[Header结构]  
_ioinit():  
1.根据dbg与否，来选择用malloc或是malloc_dbg  
2.再malloc256字节，n_size（？作用），第一次malloc    
3._heap_alloc_dbg():  
计算block_size:再n_size基础上，dbg模式需要附加一些header(_crtMemberBlockHeader和gap4字节，检查是否越界)，
4._heap_alloc_base():   
根据计算好的block_size来分配内存  
小于1016+(8)的小区块需要单独做处理  
5.sbh_alloc_block():  
添加上下cookie，并对齐16  
6.sbh_alloc_new_region():  
开始分配内存。用于管理内存大概16k  
7.sbh_alloc_new_group():  

[malloc内存分配]  
tagRegion:  
bitvec是usigned int ，
管理哪些区块是否存在链表中  

tagGroup：  
有64个listhead组成双向链表  
程序申请内存时
内存不够，group从虚拟空间中取一块（1/32,32k），按页（4k）分成八块，并且形成一个双向链表，再挂到group的双向链表中。  
给与程序，八页分配完再申请新的。 

【tag entry】  
保留：为了对齐16倍数  
黄色：标识页面的开始和结束，阻隔器，方便回收页面回收，防止切割时越界  
4080：标识这块的大小  

页表的切割  
[页表的切割]
64条链表，管理最大的内存块为大于1k（最后一条链表）。  
第一条是16个字节，后面依次32，48，64...,直到>1k
被切割的页表cookie调整来标识哪些内存被切割除去，上图切割130的大小，红色部分被切割除去，0x00000ec0是被切割后剩下的大小。  
0x00000131是130+1,加1表示已经被使用（因为切割大小一定是16的倍数，末尾一定是0，所以用来标识是否被使用）   



