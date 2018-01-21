# 基于KVM和Docker的虚拟化数据存储管理

## 引言

- 虚拟化技术概览（包括KVM、Xen、Hyper-V）

- 容器化技术概览（包括LXC、Cgroup、Docker）

  > LXC provides a “normal” OS environment that supports all the features and capabilities available in the Linux environment. Behaves very much like a traditional VM and thus offers a lower barrier to entry and does not require
  > changes to the application being deployed.
  >
  > An LXC container
  > • is a folder in /var/lib/lxc
  > • is described by a small config file
  > • requires sysadmins “elbow grease”
  > A Docker container
  > • packs an application and its dependencies
  > • contains everything it needs to run: code, runtime ...
  > • applications works regardless from the environment
  >
  > LXC is used for complete Operating Systems packaging, Docker is used for applications
  > deployment

- 虚拟化技术和容器化技术的异同，并提出使用容器化技术弥补虚拟化技术中的不足（动态数据存储，虚拟化技术提供给用户的是一个和实际生产环境一样的操作系统环境，主要用于搭建某些应用或环境，数据存储应当独立于虚拟化的存储并且可以动态挂载到任何需要的地方，但是传统的虚拟化平台通常是使用块存储实现，不够将数据独立开来，而Docker可以借用Minio这种类S3存储将远程的数据安全挂载到虚拟机中。）

  ```
  异同：
  1、性能：虚拟化技术是在硬件层面实现的虚拟化，需要有虚拟机管理应用和虚拟机操作系统层，容器是在操作系统层面实现的虚拟化，共享宿主机的操作系统内核，更加轻量级。
  2、容器不需要启动一个完整的操作系统，但是虚拟机在启动时必须加载完整的操作系统并且要虚拟化系统资源，所以容器的启动速度远远快于虚拟机。


  ```

  ​

  > A container:
  > • uses the host kernel
  > • can’t boot a different OS
  > • can’t have it’s own modules
  > • it’s a bunch of processes visible on the host machine (VMs are totally opaque)
  > Linux container’s building bricks are Control Groups (cgroups):
  > • Each subsystem has hierarchy (Memory, CPU, Block I/O ...)
  > • Hierarchies are indipendent
  > and namespaces:
  > • Provides processes with their system own view
  > • Each process is in one namespace of each type (pid, net, mnt ...)
  > Cgroups limits “how much” you can use, Namespace limits “what you can see” (and use)

## 技术选型

- 底层虚拟机化技术的选择（最后选择KVM）
- 对象存储、S3协议、Minio（最后选择Docker部署Minio，主要是为了适应多租户模式和平台统一管理）
- 其他方式的存储（NFS、webdav、CIFS，为什么不选这些）

## 平台管理技术模型/架构

- 总体架构图
- 单机模式的模型
  - 从本地扩展出单用户多Bucket（默认即是如此）
- 多机模式的模型
  - 从远程服务器中扩展出多用户多Bucket（如何实现）
  - 带宽资源占用的解决、I/O过大造成的远程服务器崩溃的解决办法

## 平台实现

- 根据REST API原则设计的具体API内容
- 实现后的一部分截图

## 结语

- 本管理设计满足了我们的实际生产需求，对于在如今越来越庞大的数据集，将要用到多种复杂的虚拟机搭建的环境中，比如Tensorflow、Spark等多种计算框架中。
- 虚拟机本身的硬盘空间永远是不会得到提升的，只需要能够容纳最小的计算环境或者应用环境的资源就可以，具体的数据存储应当远离环境而独立开来，这样服务能够实现最大的有效性和安全性。