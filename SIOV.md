## new in hardware
### new in device
- ADI(assignable device interface):最小的硬件单元，只能通过mmio映射
- IMS(interrupt message storage): MSI-x仅支持2048个中断

### new in cpu
- intel VT-d scalable mode

## new software achitechture in linux


## CPU 两级地址翻译
内存虚拟化现在一般采用两级地址翻译，EPT。</br>
但是两级地址翻译会导致多次内存 访问，因为GVA->GPA的页表，每个页表地址是GPA，而GPA需要翻译成HPA，所以开销比较大。</br>

CR3和EPT都有TLB，会加速这一过程

## iotlb
对于CPU的TLB是不共享的，即一个多个处理器，每个核都有自己的tlb，所以如果对同一个核切换不同的进程会导致刷TLB，但是切换同一个进程的不同线程不同刷tlb。但是对于**iommu的iotlb是共享资源**，

## device iotlb
PCIe ATS的宗旨就是让每个设备都有自己的cache，这样可以加速DMA的地址翻译过程；


当PCIe Device的ATC无法完成地址映射时，此刻就需要PCIe Device发送ATS Request给TA。TA完成地址映射后，会将结果返还给PCIe Device，这样，PCIe Device中的ATC就有地址映射项了。


当TA中对内存地址更改之后，会发送**ATS Invalidate Request**给PCIe Device，Device会取消该映射项，并将结果返还给TA。

![](http://file.elecfans.com/web1/M00/BB/5C/o4YBAF6qf5qAZk5AAAIWxCSIwTI240.png)

## 热迁移脏页记录
KVM脏页统计离不开**硬件支持**，它依赖Intel PML(Page Modification Logging)特性。该特性的主要功能是记录虚机写内存页的行为并将内存页的地址GPA记录下来。

- PML
  - Accessed and Dirty flags：是EPTP字段中位于bit 6的一个标志位，当设置此标志位后，它告诉CPU**每当使用EPT查询HPA时，将页结构存放的表项中的Accessed位(bit 8)置1，对于指向物理页的页表项，当往指向的物理页中写数据时，将它的Dirty位(bit 9)置1**。
  - PML Buffer：一块内存区域，用来存放上一次开启PML特性之后，CPU写过的物理页的地址，大小为4K，可以存放512条GPA。KVM就是通过这个区域来跟踪内存脏页。Buffer满了会VM Exit，KVM保存buffer的内容，然后将buffer清空


## devlink
- devlink-info：enable device driver to report device information in a standard
- devlink-region：access driver defined address regions
- devlink-params: 设置设备参数，比如是否enable SR-IOV，是否enable region的snapshot
- devlink-flash: allow updating device firmware
  - example: $ devlink dev flash pci/0000:05:00.0 file flash-boot.bin


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
- FDIR(flow director)


- mailbox: 有时候VF driver需要和PF driver通信；由mailbox buffer和mailbox register组成，buffer用来写信息，register用来同步和通知；当VF分配给一个VM时，其中一个VF资源就是mailbox，这个mailbox，VF 和 PF都可以访问。所有的vdev的mailbox都映射在pf的0x02000000处，一个vdev的mailbox占4k空间


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

### 4. ice_vdcm_mdev_open
分配VSI
ice_vdcm_mdev_open()->ice_siov.c: ice_adi_vsi_setup()->ice_lib.c: ice_vsi_setup()->ice_lib.c: ice_vsi_alloc()->ice_lib.c: ice_vsi_set_num_qs()->ice_lib.c：ice_vsi_set_num_desc()

- struct ice_vsi *ice_adi_vsi_setup(struct ice_pf *pf, u32 pasid_id): 为一个ADI设置VSI
- struct ice_vsi *ice_vsi_setup(struct ice_pf *pf, struct ice_port_info *pi, enum ice_vsi_type vsi_type, u16 vf_id, u32 pasid_id)： 分配VSI struct以及queue资源 
- static struct ice_vsi *ice_vsi_alloc(struct ice_pf *pf, enum ice_vsi_type vsi_type, u16 __always_unused vf_id): 分配PF中的下一个VSI(pf->next_vsi是一个整数index)
- static void ice_vsi_set_num_qs(struct ice_vsi *vsi, u16 __always_unused vf_id): 设置queue, descriptor和vector的数量，目前queue和vector硬编码为1，
- static void ice_vsi_set_num_desc(struct ice_vsi *vsi)：为这个VSI的queue设置descriptor数量，目前都为default值，rx为2048，tx为256个descriptor；(descriptor需要guest去设置地址)


- ice_set_ringparam((struct net_device *netdev, struct ethtool_ringparam *ring)：guest通过ice_ethtool_ops调用，设置ring的参数，其中包括每个descriptor的信息

struct kobject是组成设备device、驱动driver、总线bus、class的基本结构。相当于基类


rings

devm_kcalloc(struct device * dev, size_t n, size_t size, gfp_t flags): 具有资源管理的 kzalloc()，分配的内存与设备相关联，当设备从系统中detach时，会自动释放这部分内存，n位元素数量，size位元素大小

struct ice_vsi
  - alloc_txq
  - alloc_rxq：分配的数据队列的数量，目前硬编码为1
  - rx_rings
  - tx_rings : 
  - num_q_vectors: 一个queue，对应一个q_vectors(interrupt vector)

struct vdcm
  - u8 *vconfig : vdev的配置空间


### 读取VSI硬件信息
- ice_update_eth_stats(struct ice_vsi *vsi)


### 中断缩写
- ITR(Interrupt Throttling)
- INTRL(Interrupt Rate Limiting)
- PBA(Pending Bit Array): 中断进来对应的bit设置为1，中断发送到pcie后，清零
- QTX_TAIL寄存器存的是指向tx buffer的指针，支持2048个，2048*8=16KB，对于QRX_TAIL同理

### 为mdev创建属性
mdev的子设备会继承父设备的supported_type_groups，经验证对子设备做写操作，callback function传入的kobj和vdev是相同的地址(应该都是父设备) </br>
可以通过mdev_attr_groups添加新的属性，对通过该方法添加的属性进行操作，传进来的参数kobj每个mdev是不同的，但是dev参数相同属性是一样的

以下三个宏定义都会为MDEV创建一个属性文件，属性变量名为mdev_type_attr_\*\*name
- MDEV_TYPE_ATTR_WO(name)
- MDEV_TYPE_ATTR_RO(name)
- MDEV_TYPE_ATTR_RW(name)

#### sysfs 创建属性文件函数
- __kernfs_create_file: 在目录下创建一个文件

#### sysfs写属性文件逻辑
fs/kernfs/file.c:kernfs_fop_write_iter()->(ops->write())->sysfs_kf_write()->(ops->store())

ice_vdcm_probe()->mdev_register_device(pdev->dev, &ice_vdcm_mdev_ops)

- ice_vdcm_mdev_ops
  - .supported_type_groups  = ice_vdcm_mdev_type_groups

有很多组attr_groups，每一个group包含不同的属性，这些属性会在sysfs目录下创建文件
- ice_vdcm.c:
  - static struct attribute *mdev_types_attrs


### question
enum ice_vsi_type (no defined?)
