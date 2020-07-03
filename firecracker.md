## Firecracker
### 1.1 背景
Amazon AWS为Lambda无服务(serverless)容器应用设计的轻量级虚拟机，容器开发速度快，运营成本低，但业界一直致力于改善容器的安全问题。与容器和传统虚拟机都不同，综合两者有点，**既要安全隔离，也要性能开销**。主要用于无服务和容器应用。firecracker轻量级虚拟机，在microVM内仅单独运行一个容器或者仅运行一个应用Pod，不同应用之间实现了虚拟化技别的隔离。每个应用不再共享OS内核。每个MicroVM都以KVM进程的方式运行。firecracker使用**内存安全的Rust语言编写**，专门为无服务计算场景提供安全高效的运行时

### 1.2 原理
firecracker提升了容器的隔离性和安全性。**firecracker移除了PCI总线，取消了VGA显示，音频、视频、USB外设都不支持，没有BIOS，也不做任何指令模拟**，不是一台完整的虚拟计算机。**虚拟机内使用的OS也是定制的精简Linux**，所以其启动步骤和加载项远少于传统虚拟机，启动非常快。现在可以在125ms内启动虚拟机，每秒150台，内存开销小于5MB，并行运行4000台。</br>

</br></br>
AWS lambda，开始时用LXC隔离功能，用虚拟化隔离用户，一个用户可以在一个VM里运行多个功能应用。Lambda的function是比较小的，所以相对来说虚拟化的开销就比较大。

</br></br>
firecracker使用KVM，但是替换掉了VMM，用安全语言编写了一个VMM(Firecracker)。firecracker大约有50K Rust代码，比QEMU少96%，**Intel的Cloud Hypervisor采取了和Firecracker相似的思路**，甚至共享了很多代码。NEMU则是裁剪了QEMU。firecracker利用了Linux的功能，比如将block IO传给Linux kernel处理，Linux进程调度器和内存管理机制。firecracker可以用ps命令来可以查看所有MicroVM，还有如top，vmstat和kill命令都可以使用。firecracker基于Google的VMM crosvm实现的，复用了一些crosvm的组件。但firecracker比crosvm代码少一半。firecracker提供了有限的设备模拟：网络和block设备、串口和部分i8042。firecracker在网络和block模拟中使用了virtio，virtio block实现用了1400行代码。选择用block设备而不是文件系统，是因为**文件系统代码量大且复杂**。仅向用户提供block IO可以更好地保护主机kernel。
</br></br>
**内部架构**
微虚拟机的创建需要两个组件：jailer和firecrakcer。**Jailer负责利用Linux提供的机制来创建沙箱环境，然后在沙箱环境中启动后者**。后者利用KVM来创建极度精简的虚拟机。
![](https://github.com/CJTSAJ/BareMetal/blob/master/picture/firecracker%20structure.png)
</br></br>
firecrakcer负责创建和管理微虚拟机，提供设备模拟和handle VM exits。每个虚拟机都有用一个shim控制进程，该进程通过TCP/IP socket和Mcro-Manager、per-worker进程通信，
![](https://github.com/CJTSAJ/BareMetal/blob/master/picture/lambda%20worker.png)
**微虚拟机模型**
Firecracker**利用了硬件辅助虚拟化**。从虚拟化的角度看恶意分为如下几个方面：
- CPU/Memory：**利用VT-x进行CPU虚拟化和内存虚拟化**
- 设备模拟：virtio-net、virtio-block、console、keyboard、irqchip、clock source

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
容器开发工程师，可以通过限制系统调用来提高容器的安全性。docker在LXC的基础上进行了改进，提供了一个更高层的控制工具。
- 跨主机部署：docker 定义了镜像格式，使同一个镜像文件在任何可运行 docker 的机器中运行。LXC程序的执行依赖于机器的特定配置。
- 自动构建、版本管理、组件重用、工具生态链等等


### LXC(Linux Container)
Linux容器，将应用软件打包成一个软件容器，内含软件本身代码，以及需要的操作系统核心和库。LXC与Docker类似，也是利用了**Linux Cgroups和namespace**功能为软件提供一个独立的环境。</br></br>
使应用程序看到的操作系统环境被区隔成独立区间，包括进程树，网络，用户id，以及挂载的文件系统。 最初的Docker也是基于LXC实现的。</br></br>
用Linux的chroot命令为容器提供一个自己的根目录。
</br></br>
**seccomp** 是 Linux 内核提供的一种应用程序沙箱机制,seccomp 通过只允许应用程序调用 exit(), sigreturn(), read() 和 write() 四种系统调用来达到沙箱的效果。如果应用程序调用了除了这四种之外的系统调用， kernel 会向进程发送 SIGKILL 信号。
</br></br>
**seccomp-bpf** 是 seccomp 的一个扩展，它可以通过配置来允许应用程序调用其他的系统调用。



### Meltdown和Spectre攻击原理
在现代计算机执行条件分支指令的时候，由于不知道条件结果，所以一般会进行**推测执行**，这种方式可以减少CPU周期浪费，加速执行。</br></br>
现在假设一条指令需要权限才能执行，但是在推测机制下，处理器可以在没有获得权限的情况下推测执行该指令，最后如果权限不够，指令结果被否决。但是在执行指令的过程中，在计算机中留下了痕迹，比如CPU高速缓存中留下了数据。攻击者先flush所有的高速缓存，推测执行恶意代码后，高速缓存中只有私密数据。</br></br>
漏洞来源于硬件，研究人员和CPU设计师已经开始考虑未来应该设计怎样的CPU，才能既保留推测执行功能，又不牺牲安全性。


### Unikernel
在Unikernel的操作系统中，**所有程序与底层的核心驱动都运行在一起**，不存在运行时状态上下文切换的额外开销。将每个应用程序在编译时可以直接指定引入特定的驱动和核心library，相当于每个程序都是自带操作系统的，而且是单独定制裁剪的操作系统。Unikernel 是与某种语言紧密相关的，**一种 unikernel 只能用一种语言写程序**
</br></br>
通过Unikernel系统方式构建出来的应用程序都是可以独立发布和直接运行在虚拟化平台上的，**一个应用服务就是一个操作系统**，从而形成规模极大的操作系统集群。Unikernel系统的运行环境通常是虚拟化的基础设施，而不是那些嵌入式硬件设备。通常来说，Unikernel操作系统既不能兼容Linux或Windows软件，也不能运行Docker容器（目前还没有能运行Docker的Unikernel系统）。Unikernel也是一种不可变的基础设施（Immutable Infrastructure），**Unikernel系统一旦编译完整就不可改变的**，想要对其中的内容进行重新定制的唯一办法就是修改源码然后重新编译。
</br></br>
在Unikernel巨大的性能和安全性优势的背后是其系统构建的繁琐，每次软件的发布都需要经过『编写应用程序』、『选择需要的library』、『将应用程序与系统编译到一起形成系统』这些复杂的重复性操作
</br></br>
**LightVM**：轻量级虚拟机，虚拟机比容器奇动慢、占用内存多。容器隔离安全性不好。lightVM可以在2.3ms内驱动一个虚拟机，Linux启动一个进程需要1ms,两者都小于容器的启动速度。</br></br>
首要是缩小虚拟机的image大小和运行时内存的footprint(内存占用)。通过观察发现，多数VM和Container都是运行单应用的，若将VM的功能裁剪至刚好仅能满足所运行应用的需求，将可以极大地降低VM的内存占用。为此作者尝试了unikernel和Tinyx两种方式来创建适合单应用运行的最小镜像。 - Unikernel：小型虚拟机，其中的简约操作系统（例如MiniOS ）直接链接目标应用程序。 - Tinyx：作者开发的一个工具，用来为指定的应用程序创建一个小型Linux发行版本。

### AWS Lambda
如图，用户请求通过ALB(load balance)转发给Front end，接着转发给Worker manager，然后初始化Worker，Worker准备初始化沙箱执行环境，完成后原路返回给Front end。然后由Front end触发执行函数。</br>
![](https://github.com/CJTSAJ/BareMetal/blob/master/picture/AWS%20lambda.png)
</br>
### Hyper-Threading
hyperthreading技术的关键点就是：当我们在处理器中执行代码时，很多时候处理器并不会使用到全部的计算能力，部分计算能力会处于空闲状态，而hyperthreading技术会更大程度地“压榨”处理器。举个例子，如果一个线程必须要等到一些数据加载到缓存中以后才能继续执行，此时**CPU可以切换到另一个线程去执行**，而不用去处于空闲状态，等待当前线程的IO执行完毕。一个传统的处理器在线程之间切换大约需要20000时钟周期，而一个具有Hyperthreading技术的处理器只需要1个时钟周期，**大大减小了线程之间切换的成本**。**两个逻辑处理器是共享这颗CPU的所有执行资源**。
</br></br>
逻辑核心：Hyper-threading 使操作系统认为处理器的核心数是实际核心数的2倍，因此如果有4个核心的处理器，操作系统会认为处理器有8个核心。
