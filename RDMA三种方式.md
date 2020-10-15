# RDMA
RDMA（remote direct memory access）中文全称远程内存直接访问技术，RDMA实现需要特定硬件支持，主要有三种：Infiniband、iWARP和RoCE。
RDMA是在网络I/O虚拟化非常完善之后提供一项新的虚拟化I/O方案。虚拟化RDMA实现方式主要有三种：
- 第一种是VMM/Hypervisor通过PCI-Passtrough技术将RDMA物理设备透传给虚拟机，该方式可以让虚拟机RDMA获得最佳性能，但是RDMA物理设备只能被一台虚拟机独自占用；
- 第二种就是基于SR-IOV的PCI-Passthrough，RDMA物理设备支持SR-IOV，VMM/Hypervisor为每个虚拟机分配一个VF，该方式可以让多台虚拟机共享一个RDMA物理设备，虚拟机性能基本等价于RDMA物理设备。 
- 第三种是SoftRoCE，这种方案底层采用聚合以太网络（Converged Ethernet），VMM/Hypervisor为虚拟机提供的是一个全仿真的RDMA虚拟设备。
