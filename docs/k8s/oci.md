---
title: oci
layout: default
parent: k8s
has_children: false
---

## 1. 介绍

- OCI（Open Container Initiative 开放容器协议），旨在推动**容器运行时**和**容器镜像格式**的开放标准化。

- Runtime Specification（运行时规范）：定义了容器的生命周期管理，包括创建、启动、停止和销毁容器等操作。最常见的容器运行时实现是OCI Runtime（也称为runc）。
- Image Specification（镜像规范）：定义了容器镜像的结构和格式，包括镜像的分层结构、元数据和配置等信息。常见的镜像格式是OCI Image（也称为OCI Image Format）。

## 2. OCI Runtime 规范

- OCI运行时文件系统包主要包括以下两部分:
    - config.json：这是必需的配置文件，存放于文件系统包的根目录下。OCI运行时规范对Linux、Windows、Solaris和虚拟机4种平台的运行时做了相应的配置规范。
    - 容器的根文件系统：容器启动后进程所使用的根文件系统，由 config.json 中的root.path属性确定该文件系统的路径，通常是“rootfs/”。

- OCI定义了容器的生命周期中四个基本的状态: creating, created, running, stopped

### 2.1 rootfs
- 挂载点:
    Linux内核在启动时，会首先静态生成rootfs文件系统的相关结构，用于提供最原始的挂载点和根目录，不过这个时候，新生成的根目录下暂时还不包含任何文件。后面内核会基于rootfs构建一个临时根文件系统，并完成早期用户空间的初始化。
- 文件系统:
    定义了文件树以`/`为根，对应的文件放在指定的目录，比如`/etc`下存放配置文件，`/bin`下存放二进制文件

### 2.2 OCI Runtime 规范——运行时配置（Linux）

在Linux平台下的配置，容器Runtime配置主要围绕元数据、资源隔离、资源管理、用户进程几个维度展开：元数据主要包括：

- oci的版本信息： ociVersion
- 容器运行的根文件系统（root filesystem）路径和读写权限。
- hostname配置。
- 用户配置

### 2.3 OCI Runtime 规范——资源隔离（namespace）

OCI支持Linux内核支持的7种类型：
- pid: 保证用户进程只能看到所在容器内的其它进程。
- network：使容器拥有自已的网络栈。
- mount: 使容器拥有隔离的mount表。
- ipc: 使容器内的进程拥有系统级的IPC资源隔离。
- uts: 容器可以使用自已的hostname和domainname。
- user: 使得容器可以对主机和容器内的用户和用户组进行映射。
- cgroup：使得容器拥有独立的cgroup视图。


### 2.4  OCI Runtime 规范——资源管理

- mount：根据用户的需求，顺序对用户的挂载配置项进行挂载操作。每个挂载项包含基本的source, destination配置项。
- rlimit： cpu，mem等的限制。

### 2.5 OCI Runtime 规范——用户进程 

用户进程即process配置项，主要包括环境变量、安全、权限控制、OOM管理等内容。
当然还有最重要的用户进程的配置。对Linux来讲，还要求容器内的proc, sysfs, devpts, tmpfs这四个文件系统必须可用。

### 2.6 OCI Runtime 规范——容器标准包（Bundle）

容器标准包包含了容器运行的所有环境依赖，它是保证容器运行一致性的基础。一个标准的容器标准包包含所需要加载和启动容器的所有信息。包含两部分内容：

- config.json： 即前文所述的容器运行时配置内容。
- root filesystem: 即前文所述的root.path所代表的位置。

## 3. OCI Image规范

该规范包含manifest, image index 和 filesystem layers三部分内容。

- manifest: 对于指定架构和OS的容器镜像， manifest定义了它所依赖的相关配置信息和对应的layer镜像层信息。
- image index: 比manifest更高层的抽象，包含了额外的配置信息。
- filesystem layer: 给出了如何将容器的文件系统进行序列化，如何创建和使用这些layer。我们知道容器的启动速度可达秒级。主要的原因是我们常见的aufs, devicemapper等均采用了COW(copy on write)的技术，使得相同镜像的不同容器实例可以共享bundle，write（修改）的数据也是在layter中。

### 3.1 overlay2

- **mount**, mount [-fnrsvw] [-t fstype] [-o options] device mountpoint
    - fstype: ext2、ext3、ext4、xfs、btrfs、vfat、sysfs、proc、nfs和cifs。 
    - 这里是用的**overlay2**

- overlay2主要由merged、lowerdir、upperdir、workdir组成。 对外统一展示为merged，uperdir和lower的同名文件会被upperdir覆盖
    - lowerdir:lower曾是只读的，基于copy-on-write,写时复制，所以在mount后的文件的时候原来的文件不会变


## cgroup

- 伪文件系统：
    - /proc 内存数据的映射的临时文件
    - /sys 操作系统产生的，一般是加载的时候产生的临时文件

- 一切设备都是文件：
    - 存储设备和其他设备




- cgroup 主要由两部分组成 - 核心和控制器,cgroups 形成一个树结构，系统中的每个进程都属于 到一个且只有一个 cgroup 中,可以迁移进程 添加到另一个 cgroup 中。流程迁移不会影响 already 现有后代进程。


task(进程)->cgroup（把进程放入）->subsystem

- 可以操作的对象基本只有cpu，内存，还有设备和挂起
- tasks和cgroup.procs有什么区别呢？按照官方文档的描述，将pid写入cgroup.procs，则该pid所在的线程组及该pid的子进程等都会自动加入到cgroup中。将pid写入tasks，则只限制该pid。

- **subsystem**
    - cpuset： 对task独立分配cpu和内存
    - cpu,cpuacct
    - blkio： 块设备(磁盘，U盘)
    - memory：内存使用量限制，并且生成资源报告
    - devices：开启或者关闭对设备的访问，比如鼠标键盘（去除块设备）
    - freezer：挂起和恢复设备资源
    - net_cls,net_prio： 标记网络数据包
    - perf_event：性能测试
    - hugetlb：没有启用
    - pids： 限制单独PID编号
    - rdma：

