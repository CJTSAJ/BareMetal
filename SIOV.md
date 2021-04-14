## new in hardware
### new in device
- ADI(assignable device interface):最小的硬件单元，只能通过mmio映射
- IMS(interrupt message storage): MSI-x仅支持2048个中断

### new in cpu
- intel VT-d scalable mode

## new software achitechture in linux



## PCIe ATS(Address Translation Services)
### 1. motivation
iotlb会被多个IO设备同时访问，这种集中式的iotlb会影响系统的性能

ATS的思想：每个PCIe设备都有自己的ATC(Address Translation Cache)，无需查找iotlb，减小iotlb的压力，提高访存性能。

### 2. detail
![](http://liujunming.top/images/2019/11/9.PNG)

详情见[Address Translation Services, Revision 1.1](https://composter.com.ua/documents/ats_r1.1_26Jan09.pdf) p11 to p12

## VT-d scalable mode
[reference](https://www.cnblogs.com/haiyonghao/p/14440767.html)
### 1. legacy mode
![](https://img2020.cnblogs.com/blog/2262526/202102/2262526-20210228212927791-160641719.png)

### 2. scalable mode
![](https://img2020.cnblogs.com/blog/2262526/202102/2262526-20210228212906550-354955558.png)


- VEB(Virtual Edge Bridge): 
- VEPA(virtual Ethernet port aggregator)


- mailbox: 有时候VF driver需要和PF driver通信；由mailbox buffer和mailbox register组成，buffer用来写信息，register用来同步和通知；当VF分配给一个VM时，其中一个VF资源就是mailbox，这个mailbox，VF 和 PF都可以访问，
