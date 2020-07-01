### firecracker
与容器和传统虚拟机都不同，综合两者有点，既要安全隔离，也要性能开销。主要用于无服务和容器应用。</br></br>
firecracker轻量级虚拟机，在microVM内仅单独运行一个容器或者仅运行一个应用Pod，不同应用之间实现了虚拟化技别的隔离。每个应用不再共享OS内核。 </br>
每个MicroVM都以KVM进程的方式运行。
firecracker使用内存安全的Rust语言编写

firecracker提升了容器的隔离性和安全性。</br>
firecracker移除了PCI总线，取消了VGA显示，音频、视频、USB外设都不支持，没有BIOS，也不做任何指令模拟，不是一台完整的虚拟计算机。</br>
虚拟机内使用的OS也是定制的精简Linux，所以其启动步骤和加载项远少于传统虚拟机，启动非常快。现在可以在125ms内启动虚拟机，每秒150台，内存开销小于5MB，并行运行4000台。</br>
docker在LXC的基础上进行了改进，提供了一个更高层的控制工具。
- 跨主机部署：docker 定义了镜像格式，使同一个镜像文件在任何可运行 docker 的机器中运行。LXC程序的执行依赖于机器的特定配置。
- 自动构建、版本管理、组件重用、工具生态链等等

</br></br>
AWS lambda，开始时用LXC隔离功能，用虚拟化隔离用户，一个用户可以在一个VM里运行多个功能应用。

### 容器原理
容器的本质是一个进程，容器壁虚拟化启动快很多，内存开销也小很多。主要利用Linux的Cgroups技术对进程进行资源限制。
比如cgroups可以限制进程的CPU核数、特定的内存大小，如果资源超过限制则暂停或杀死。cgroup一个很完整的控制功能，可以限制CPU、内存、磁盘。</br>
在/sys/fs/cgroup/cpu/docker目录中，linux会为每个容器创建一个cgroup目录，以容器的长ID命名。</br>
</br></br>
Docker利用namespace机制来进行容器间的隔离。
Linux中有6种namespace，分别对应6种资源：Mount， UTS， IPC， PID，Network和User
- Mount namespace：让容器看上去拥有整个文件系统。
- UTS namespace：让容器拥有自己的hostname。容器的hostname默认是它的短ID，可以通过-h或 --hostname参数来设置。
- IPC namespace：让容器拥有自己的共享内存和信号量，实现进程间通信。
- Network：让容器拥有自己独立的网卡、ip、路由等资源。
- User namespace：让容器能够管理自己的用户，host不能看到容器中创建的用户。
- PID namespace：容器拥有独立的进程号
</br></br>
容器的问题：主机上的所有容器共享一个操作系统内核，依赖内核中的隔离机制，一个容器可能需要依赖数百个syscall。</br>
容器开发工程师，可以通过限制系统调用来提高容器的安全性。


### LXC(Linux Container)
Linux容器，将应用软件打包成一个软件容器，内含软件本身代码，以及需要的操作系统核心和库。LXC与Docker类似，也是利用了Linux Cgroups和namespace功能为软件提供一个独立的环境。</br></br>
使应用程序看到的操作系统环境被区隔成独立区间，包括进程树，网络，用户id，以及挂载的文件系统。 最初的Docker也是基于LXC实现的。</br></br>
用Linux的chroot命令为容器提供一个自己的根目录。



