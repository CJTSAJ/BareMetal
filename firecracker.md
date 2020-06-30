
firecracker轻量级虚拟机，在microVM内仅单独运行一个容器或者仅运行一个应用Pod，不同应用之间实现了虚拟化技别的隔离。每个应用不再共享OS内核。</br>
每个MicroVM都以KVM进程的方式运行。
firecracker使用内存安全的Rust语言编写

firecracker提升了容器的隔离性和安全性。</br>
firecracker移除了PCI总线，取消了VGA显示等等硬件模拟，不是一台完整的虚拟计算机。</br>
虚拟机内使用的OS也是定制的精简Linux，所以其启动步骤和加载项远少于传统虚拟机，启动非常快。现在可以在125ms内启动虚拟机，每秒150台，内存开销小于5MB，并行运行4000台。</br>
