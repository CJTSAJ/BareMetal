## Reference
https://zhuanlan.zhihu.com/p/136767162 </br>
http://bestwode.com/sma/430348.html

Cascade Glacier Version 3 Board，支持OVS offload到该硬件，有两个核，**一个ARM核(HPS)和一个FPGA核**。ARM核上没有PCI总线，所以没有设备驱动。ARM核运行Wind River Linux 9 kernel 4.8，该系统内有OVS模块，FPGA镜像存储在EPCQ Flash上，直接从该Flash上启动，可以通过HPS更新该镜像。ARM系统用于管理网卡，包括镜像更新，配置等等。最**主要的用途是用于运行OVS控制功能和slow path**。
![](https://github.com/CJTSAJ/BareMetal/blob/master/picture/%E6%99%BA%E8%83%BD%E7%BD%91%E5%8D%A1.png)

两种模式
- Virtualized environment
- Bare metal：在该模式下，OVS的slow path运行在ARM核上

硬件要求
- PCIe 3.0 x8 or x16 slot
- Operating temperature: 0 °C to 50 °C
- Operating system dependency：CentOS 7.4(Kernel 3.10.0-693)，其它版本可能可以，但是还没有试过。

- **ifc**：FPGA Device Driver，通过跟文件系统，用户可以配置VF的数量。
- **ifc_pm**：Port Management on ARM

### 启动
用USB连接后，运行在ARM上的Linux自动开始运行DHCP客户端，用户可以通过ssh登录网卡。HPS和FPGA分别从各自的Flash启动镜像。ARM核有两个镜像，如果一个镜像坏了就用另一个镜像。

### 软核处理器
基于FPGA的SOC片上系统设计技术，是使用FPGA的逻辑和资源搭建的一个软核CPU系统，由于是**使用FPGA的通用逻辑搭建的CPU**，
因此具有一定的灵活性，用户可以根据自己的需求对CPU进行定制裁剪，增加一些专用功能，例如除法或浮点运算单元，
用于提升CPU在某些专用运算方面的性能，或者删除一些在系统里面使用不到的功能，以节约逻辑资源。

而且，如果单个的软核CPU无法满足用户需求，**可以添加多个CPU软核，搭建多核系统**，通过多核CPU协同工作，让系统拥有更加灵活便捷的控制能力。

### 硬核处理器(HPS, Hardware Processor System)
由于软核CPU是使用FPGA的通用逻辑资源搭建的，相较使用经过布**局布线优化的硬核处理器**来说，**软核处理器够运行的最高实时钟主频要低一些**。
因此**SOPC方案(软核)仅适用于对于数处理器整体性能要求不高的应用**。所以，各大FPGA厂家推出了SoC FPGA技术，是在芯片设计之初，就在内部的硬件电路上添加了硬核处理器，
是纯硬件实现的，不会消耗FPGA的逻辑资源，硬核处理器和FPGA逻辑在一定程度上是相互独立的，简单的说，**就是SoC FPGA就是把一块ARM处理器和一块FPGA芯片封装成了一个芯片**。


### 区别和联系
一般来说，硬核处理器的性能要远远高于软核处理器。另外，硬核处理器除了CPU部分，还集成了各种高性能外设，如MMU、DDR3控制器、Nand FLASH控制器等，
**可以运行成熟的Linux操作系统和应用程序**，提供统一的系统API，降低开发者的软件开发难度。

虽然SoC FPGA芯片上既包含了有ARM，又包含了有FPGA，但是**两者一定程度上是相互独立的**，SoC芯片上的ARM处理器核并非是包含于FPGA逻辑单元内部的，FPGA和ARM（HPS）处理器只是封装到同一个芯片中，
JTAG接口、电源引脚和外设的接口引脚都是独立的，因此，如果使用SoC FPGA芯片进行设计，即使不使用到片上的ARM处理器，**ARM处理器部分占用的芯片资源也无法释放出来，不能用作通用的FPGA资源**。

### DPDK
传统数据包处理方式存在以下两个难题
- 每来一次报文，就触发一次中断。大量数据的到来会导致频繁的中断触发，系统承受不了如此频繁的中断。
- 数据包需要从内核态拷贝到用户态，当数据量巨大时，这个拷贝操作会大幅的降低数据包处理性能。

核心技术
- 通过UIO技术将报文拷贝到应用空间处理。DPDK针对Intel网卡实现了基于轮询方式的**PMD（Poll Mode Drivers）驱动**，该驱动由API、用户空间运行的驱动程序构成，该驱动**使用无中断方式直接操作网卡的接收和发送队列**，减少了报文在用户空间和应用空间的多次拷贝。
- 通过**大页内存，降低cache miss**，提高命中率，进而cpu访问速度
- 通过CPU亲和性，绑定网卡和线程到固定的core，减少cpu任务切换
- 通过无锁队列，减少资源竞争

### vDPA(vhost datapath acceleration)
vDPA是vhost datapath acceleration的缩写，意为vhost数据路径加速

### OVS(Open VSwitch)
虚拟交换机，绿色虚线内组成的就是一个虚拟网络了。其**虚拟机之间的信息交换都通过虚拟交换机**。</br>
![](https://img-blog.csdn.net/20140917210046025)

虽然在内核实现可以缩短网络数据包在操作系统的路径，但是在内核进行程序开发和更新也更加困难，**以OpenVSwitch的更新速度，完全在内核实现将变得不切实际**。其次，完全按照OpenFlow pipeline去处理网络包，势必要消耗大量CPU，进而降低网络性能。所以Openflow实现在用户态。

OpenVSwitch主要由三个部分组成：
- ovsdb-server：保存OpenVSwitch的持久化数据
- ovs-vswitchd：运行在用户空间的转发程序，接收SDN控制器下发的OpenFlow规则。并且通知OVS内核模块该如何处理网络数据包。
- OVS内核模块：运行在内核空间的转发程序，根据ovs-vswitchd的指示，处理网络数据包

在SDN中所处的位置 </br>
![](http://image-store1.oss-cn-hangzhou.aliyuncs.com/18-9-28/40575247.jpg)

网络数据的转发，都是由位于内核空间的OVS datapath完成。用户空间和内核空间的信息是怎么同步的？对于一个网络数据流，**第一个数据包到达OVS datapath，这个时候的datapath没有转发信息**，并不知道怎么完成转发。接下来**OVS datapath会查询位于用户空间的ovs-vswitchd进程**。ovs-vswitchd进程因为有OpenFlow信息，可以根据OpenFlow规则完成match-action操作，也就是从一堆OpenFlow规则里面匹配网络数据包对应的规则，根据这些规则里的action实现转发。

这样第一个数据包就完成了转发。与此同时，ovs-vswitchd会通过netlink向OVS datapath写入一条（也有可能是多条）转发规则,同一个网络数据流的后继网络数据包到达OVS datapath时，因为**已经有了转发规则，datapath就可以直接完成转发，不再需要向ovs-vswitchd查询**。</br>
![](https://pic1.zhimg.com/80/v2-41387cfa250047521687a2536f5ef2d8_hd.jpg)

- slow path: 通过ovs-vswitchd查找OpenFlow实现转发的路径称为slow-path
- fast path: 通过OVS datapath直接转发的路径称为fast-path

### Zero Copy
zero copy技术就是减少不必要的内核缓冲区跟用户缓冲区间的拷贝，从而减少CPU的开销和内核态切换开销，达到性能的提升。</br>
![](https://upload-images.jianshu.io/upload_images/207235-0c63dbf565386423.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/512/format/webp)
![](https://upload-images.jianshu.io/upload_images/207235-bc756e05a212b2ef.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/429/format/webp)
