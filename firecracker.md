### firecracker
firecracker轻量级虚拟机，在microVM内仅单独运行一个容器或者仅运行一个应用Pod，不同应用之间实现了虚拟化技别的隔离。每个应用不再共享OS内核。</br>
每个MicroVM都以KVM进程的方式运行。
firecracker使用内存安全的Rust语言编写

firecracker提升了容器的隔离性和安全性。</br>
firecracker移除了PCI总线，取消了VGA显示等等硬件模拟，不是一台完整的虚拟计算机。</br>
虚拟机内使用的OS也是定制的精简Linux，所以其启动步骤和加载项远少于传统虚拟机，启动非常快。现在可以在125ms内启动虚拟机，每秒150台，内存开销小于5MB，并行运行4000台。</br>


### 容器原理
容器的本质是一个进程，容器壁虚拟化启动快很多，内存开销也小很多。主要利用Linux的Cgroups技术对进程进行资源限制。
比如cgroups可以限制进程的CPU核数、特定的内存大小，如果资源超过限制则暂停或杀死。cgroup一个很完整的控制功能，可以限制CPU、内存、磁盘。</br>
在/sys/fs/cgroup/cpu/docker目录中，linux会为每个容器创建一个cgroup目录，以容器的长ID命名。</br>
</br>
Docker利用namespace机制来进行容器间的隔离。
Linux中有6种namespace，分别对应6种资源：Mount， UTS， IPC， PID，Network和User
- Mount namespace：让容器看上去拥有整个文件系统。
- UTS namespace：让容器拥有自己的hostname。容器的hostname默认是它的短ID，可以通过-h或 --hostname参数来设置。
- IPC namespace：让容器拥有自己的共享内存和信号量，实现进程间通信。
- Network：让容器拥有自己独立的网卡、ip、路由等资源。
- User namespace：让容器能够管理自己的用户，host不能看到容器中创建的用户。
- PID namespace：容器拥有独立的进程号


