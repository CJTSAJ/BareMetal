## 全虚拟化IO设备
KVM 在 IO 虚拟化方面，传统或者默认的方式是使用 QEMU 纯软件的方式来模拟 I/O 设备，包括键盘、鼠标、显示器，硬盘 和 网卡 等。模拟设备可能会使用物理的设备，或者使用纯软件来模拟。模拟设备只存在于软件中。
### 1.1 原理
![](https://pic3.zhimg.com/80/v2-6c53189e37397b2e4ac49ea1491d54e6_720w.jpg)
1. 客户机的设备驱动程序发起 I/O 请求操作请求
2. **KVM 模块中的 I/O 操作捕获代码拦截这次 I/O 请求**
3. 经过处理后将本次 I/O 请求的信息放到 I/O 共享页 （sharing page），并**通知用户空间的 QEMU 程序**。
4. QEMU 程序获得 I/O 操作的具体信息之后，交由硬件模拟代码来模拟出本次 I/O 操作。
5. 完成之后，QEMU 将结果放回 I/O 共享页，并通知 KMV 模块中的 I/O 操作捕获代码。
6. KVM 模块的捕获代码读取 I/O 共享页中的操作结果，并把结果放回客户机。

注意：当客户机通过DMA （Direct Memory Access）访问大块I/O时，QEMU 模拟程序将不会把结果放进共享页中，而是通过内存映射的方式将结果直接写到客户机的内存中共，然后通知KVM模块告诉客户机DMA操作已经完成。

这种方式的优点是可以模拟出各种各样的硬件设备；其缺点是**每次 I/O 操作的路径比较长，需要多次上下文切换**，也需要多次数据复制，所以性能较差。

## VirtIO
在 KVM 中可以使用准虚拟化驱动来提供客户机的I/O 性能。目前 KVM 采用的的是 virtio 这个 Linux 上的设备驱动标准框架，它提供了一种 Host 与 Guest 交互的 IO 框架。
### 2.1 VirtIO架构
KVM/QEMU 的 vitio 实现采用在 Guest OS 内核中安装前端驱动 （Front-end driver）和在 QEMU 中实现后端驱动（Back-end）的方式。前后端驱动通过 vring 直接通信，这就**绕过了经过 KVM 内核模块的过程**，达到提高 I/O 性能的目的。
![](https://pic3.zhimg.com/80/v2-b0eda7b55488a6d1b5dda5a851df1d7e_720w.jpg)

## VirtIO+vhost
vhost技术对virtio-net进行了优化，在内核中加入了vhost-net.ko模块，使得对网络数据可以再内核态得到处理。
![](https://forum.huawei.com/huaweiconnect/data/attachment/forum/201808/25/20180825161343490002.png)

virtio的io路径
1.     guest设置好tx;
2.     kick host;
3.     guest陷出到kvm；
4.     kvm从内核切换到用户态的qemu进程；
5.     qemu将tx数据投递到tap设备；。

vhost的io路径
1.     guest设置好tx;
2.     kick host;
3.     guest陷出到kvm；
4.     vhost-net将tx数据投递到tap设备;

vhost将部分virio驱动的操作从用户态移到内核态，**减少了用户态/内核态切换时间和包的拷贝次数**，从而更进一步的提升了性能

## vhost-user
vhost是为了减少网络数据交换过程中的多次上下文切换，让guset和host kernel直接通信，提高网络性能。然而再大规模使用KVM虚拟化的云计算生产环境中，通常都会使用Open vSwitch或与其类似的SDN方案，以便可以更加灵活管理网络资源。通常在这种情况下，在**宿主机上会运行一个虚拟交换机用户态进程**，这时如果使用vhost作为后端网络罗处理程序，那么主机上**会存在内核态和用户态的切换**。vhost-user就是为了解决这个问题，让**客户机直接与宿主机上的虚拟交换机进程进行数据交换**。
