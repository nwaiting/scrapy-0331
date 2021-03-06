## linux - 进程线程内存共享
- **概述：**
>       进程和线程关系：
>           Linux内核把所谓的线程都当做进程来实现，线程仅仅被视为一个与其他进程共享资源的进程。！！！
>           一个进程的地址空间与另一个进程的地址空间使用相同的内存地址（虚拟内存的地址空间），是否共享地址空间几乎是进程和Linux中的线程间本质上的唯一区别，
>           线程对内核来说仅仅是一个共享特定资源的进程而已。
>
>       进程中创建的子线程，它会获取一部分进程的堆栈空间，作为其名义上的独立私有空间。（为什么是名义上的？）
>       由于这些属于同一进程的线程，只要其他线程获取了你私有堆栈上某些数据的指针，其他线程便可以自由访问你名义上的私有空间数据。
>       （多进程一般不容易获取，因为不同的进程，相同的虚拟地址，基本不可能映射到相同的物理地址）
>

- **进程内存释放**
>       内核释放物理页面是通过先释放虚拟内存，找到其对应的物理页面，然后将其全部释放。！！！！
>
>

- **进程和线程共享：**
>       共享全局变量
>       堆
>       进程的文件
>
>

- **进程栈：**
>       进程栈是用户态栈，在32位机器上，虚拟地址空间大小为4G。虽然内核和用户进程占了这么大的地址空间，但是并不意味着它们使用了这么多物理内存，仅表示可支配内存。
>       这些虚拟地址通过页表(PageTable)映射到物理内存，页表由操作系统维护。
>
>       栈区大小：
>           进程虚拟地址空间中的栈区，就是进程栈，进程栈的初始化大小是由编译器和链接器计算出来的，但是栈的实时大小不是固定的，
>           内核会根据入栈情况进程动态增长（就是添加新的页表）。栈的最大小智RLIMIT_STACK一般为8M
>       动态栈增长是唯一一种访问未映射内存区域而被允许的情况，其他任何对未映射内存区域的访问都会触发页错误。
>

- **线程栈：**
>       线程栈是从进程的地址空间中map出来的一块内存区域，原则上是线程私有的。
>       线程栈是使用mmap系统调用分配的空间，mmap其实和堆一样，可以说他们都是动态内存分配，但是严格来说mmap区域并不属于堆区。
>       为什么需要单独的线程栈：
>           当内核唤醒进程的时候，需要回复进程的上下文，就是进程栈
>           为了防止主进程和子线程出入栈不混乱
>
>

- **进程内核栈：**
>       就是为了多进程在并发执行时，都需要进入内核态，那么不同的进程栈可以隔离进程的内核栈操作
>
>

- **多线程哪些需要加锁：**
>       多线程调用需要保护时，主要针对全局变量和静态变量，函数内的局部变量不会受到影响。
>       可重入函数要求不能使用无保护的全局变量。！！！
>
>       一组并发线程运行在一个进程的上下文，每个进程都有它独立的线程上下文，如：栈、程序计数器、线程ID、条件码等
>       每个线程和其他线程一起共享除此之外的进程上下文的剩余部分,如：整个用户的虚拟地址空间、打开的文件集合。！！！
>       寄存器不共享，虚拟存储器不共享。！！
>
>
>

- **Linux进程内存分布：**
>       Kernel space
>       Stack
>       mmap：
>           1、内存映射是一种高效的文件IO方式，因而被用于装载动态共享库。
>           2、内核将硬盘文件的内容直接映射到内存，任何应用程序都可荣国Linux的mmap()系统调用请求这种映射。
>           3、用户也可以创建匿名内存映射，改映射没有对应的文件，可用于存放程序数据
>               Linux中，通过malloc()请求一大块内存，C运行库将创建一个匿名内存映射，而不是使用堆内存。大块内存意味着比阈值MMAP_THRESHOLD还大，默认128K，
>               可通过mallopt()调整。
>           mmap映射向下扩展，堆向上扩展，两者相对扩展，直到用尽虚拟地址空间。
>           程序可申请的最大堆空间是多大？？
>               一共有两处，一处是从bss段到0X40000000，约不到1G的空间；一处是从共享库到栈的空间，约不到2G，这两块空间大小取决于栈、共享库大小和数量。
>               故32位Linux的2.6以上的Kernel，malloc()申请的最大内存理论值在2.9G左右
>       Heap
>       BSS
>       Data segment
>       Text
>
>
>
>
>
>
>
>
>
>
>
>
>

- **待续：**
>       参考：https://github.com/Durant35/durant35.github.io/issues/24  Linux-进程-线程-内存
>            http://durant35.github.io/2017/10/29/VM1_UserSpaceSegments/    虚拟内存[01] 用户内存空间的各个段分布
>
>
>
>
>
>
>
>
>
>
>
