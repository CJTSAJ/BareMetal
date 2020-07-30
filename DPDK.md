## Reference
https://www.jianshu.com/p/8c6b056f73ce </br>

## DPDK
传统数据包处理方式存在以下两个难题
- 每来一次报文，就触发一次中断。大量数据的到来会导致频繁的中断触发，系统承受不了如此频繁的中断。
- 数据包需要从内核态拷贝到用户态，当数据量巨大时，这个拷贝操作会大幅的降低数据包处理性能。

核心技术
- 通过UIO技术将报文拷贝到应用空间处理。DPDK针对Intel网卡实现了基于轮询方式的**PMD（Poll Mode Drivers）驱动**，该驱动由API、用户空间运行的驱动程序构成，该驱动**使用无中断方式直接操作网卡的接收和发送队列**，减少了报文在用户空间和应用空间的多次拷贝。
- 通过**大页内存，降低cache miss**，提高命中率，进而cpu访问速度
- 通过CPU亲和性，绑定网卡和线程到固定的core，减少cpu任务切换
- 通过无锁队列，减少资源竞争

## OVS(Open VSwitch)
虚拟交换机，绿色虚线内组成的就是一个虚拟网络了。其**虚拟机之间的信息交换都通过虚拟交换机**。
![](https://img-blog.csdn.net/20140917210046025)

## Zero Copy
zero copy技术就是减少不必要的内核缓冲区跟用户缓冲区间的拷贝，从而减少CPU的开销和内核态切换开销，达到性能的提升。</br>
![](https://upload-images.jianshu.io/upload_images/207235-0c63dbf565386423.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/512/format/webp)
![](https://upload-images.jianshu.io/upload_images/207235-bc756e05a212b2ef.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/429/format/webp)
