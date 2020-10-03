**三家公司的异构计算** </br>
Amazon offers Elastic Inference on top of GPUs, </br>
Microsoft Azure exposes FPGAaccelerated ML services </br>
Google runs AutoML service on TPUs </br></br></br>

在两种智能网卡上实现了lynx：基于FPGA的智能网卡和基于ARM处理器的智能网卡
- FPGA-based NIC：在FPGA中主要实现了两个逻辑。一个网络服务器，用来处理外界的链接；第二个逻辑是通过PCIe与外界的加速器通信
- ARM Based NIC：

**CPU-driven design缺点**
- Accelerator invocation overhead：CPU-GPU之间至少30微妙的延迟。</br>
- Wasteful use of the CPU：加速器管理都是IO密集型操作，所以不需要超标量计算和乱序执行来提升性能。CPU应该用来做更适合CPU计算的应用
- Interference with co-located applications：CPU-only应用回影响加速器的延迟。

**Limitations of the GPU-centric server design**
- 对于GPU来说运行网络服务需要消耗大量的计算资源，这会导致计算密集型的应用吞吐量变小。
- 加速器侧的IO层可能会对额外的硬件造成压力，比如寄存器，在某些情况下会导致加速器性能下降
- 需要CPU参与加速器的网络服务，会受其他运行在CPU上的应用影响性能。

**设计**
SNIC运行了一个通用的网络服务器用来处理网络包；通过RDMA与加速器通信，加速器连在PCIe总线上。CPU不处理网络请求。设计满足一下四点要求。
- 加速器一侧有网络IO：加速器可以收发网络包
- 避免在加速器上运行通用的网络服务：Lynx在加速器上的网络服务很轻量级，最小化占用硬件资源。这个layer可以很轻易在加速器上实现，并且很容易移植到其他加速器。
- offload加速器IO层到SNIC：Lynx将网络处理和极速器管理移动到了SNIC
- 保持可移植性

# vpp OVS-DPDK性能测试
SR-IOV8个VNF时吞吐量达到峰值，九个以后吞吐量开始下降。主要是因为网卡缓存溢出导致性能下降</br>


