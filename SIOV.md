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
- VSI(virtual station interface):一个vNIC有一个VSI，并且通过VSI连接VEB


- mailbox: 有时候VF driver需要和PF driver通信；由mailbox buffer和mailbox register组成，buffer用来写信息，register用来同步和通知；当VF分配给一个VM时，其中一个VF资源就是mailbox，这个mailbox，VF 和 PF都可以访问，


###   PCIe
- PCIe ATU(Address translation unit): 负责把cpu域的物理地址转换到PCI域的总线地址

- 一个PCIe系统可以有256条bus，每条bus有32个device，每个device最多8个function，所以操作系统需要为设备预留 256*32*8*4KB=256MB空间

- 上电的时候，系统吧PCIe设备开放的空间映射到内存空间，CPU访问对应的内存空间，RC(root complex)检查该地址，如果是设备，则触发产生TLP，去读取或者写入PCIe配置空间；一个设备可能有若干个内部空间需要映射到内存空间，设备出厂时将这些空间的大小属性和都写在BAR寄存器里，上电后，系统读取这些bar，分别为其分配对应的系统内存空间，并把相应的内存基地址写会BAR


## ice.ko代码阅读
### 1. VDCM模块初始化
ice_module_init()->pci_register_driver()->ice_probe()->ice_vdcm_probe()
- pci_register_driver()：根据vendor_id和device_id(在pci_driver结构id_table中)去匹配设备，**系统在bus总线的数据结构中维护两个结构体struct kset drivers和struct kset devices**，一个方驱动，一个放设备信息。driver和device attach后，后调用driver probe函数

### 2. ice.ko注册parent_dev
ice_vdcm_probe()->mdev_register_device()：设备驱动向内核mdev模块注册parent mdev

### 3. 向mdev目录create mdev
mdev_register_device(&pdev->dev, &ice_vdcm_mdev_ops)注册父设备时，ice_vdcm_mdev_ops中create与ice_vdcm_mdev_create()绑定。所以create调用ice_vdcm_mdev_create(struct kobject *kobj, struct mdev_device *mdev)

ice_vdcm_mdev_create()->ice_vdcm_create_config_space(ivdm)初始化vdev配置空间


struct kobject是组成设备device、驱动driver、总线bus、class的基本结构。相当于基类


devm_kcalloc(struct device * dev, size_t n, size_t size, gfp_t flags): 具有资源管理的 kzalloc()，分配的内存与设备相关联，当设备从系统中detach时，会自动释放这部分内存，n位元素数量，size位元素大小

struct vdcm
  - u8 *vconfig : vdev的配置空间
