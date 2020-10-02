***三家公司的异构计算*** </br>
Amazon offers Elastic Inference on top of GPUs, </br>
Microsoft Azure exposes FPGAaccelerated ML services </br>
Google runs AutoML service on TPUs </br></br></br>

在两种智能网卡上实现了lynx：基于FPGA的智能网卡和基于ARM处理器的智能网卡
- FPGA-based NIC：在FPGA中主要实现了两个逻辑。一个网络服务器，用来处理外界的链接；第二个逻辑是通过PCIe与外界的加速器通信
- ARM Based NIC：


# vpp OVS-DPDK性能测试
SR-IOV8个VNF时吞吐量达到峰值，九个以后吞吐量开始下降。主要是因为网卡缓存溢出导致性能下降

