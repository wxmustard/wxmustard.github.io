# 基于KVM和Docker的虚拟化数据存储管理

## 引言

- 虚拟化技术概览（包括KVM、Xen、Hyper-V）

- 容器化技术概览（包括LXC、Cgroup、Docker）

  > LXC provides a “normal” OS environment that supports all the features and capabilities available in the Linux environment. Behaves very much like a traditional VM and thus offers a lower barrier to entry and does not require changes to the application being deployed.
  >
  > An LXC container • is a folder in /var/lib/lxc • is described by a small config file • requires sysadmins “elbow grease” A Docker container • packs an application and its dependencies • contains everything it needs to run: code, runtime ... • applications works regardless from the environment
  >
  > LXC is used for complete Operating Systems packaging, Docker is used for applications deployment

- 虚拟化技术和容器化技术的异同，并提出使用容器化技术弥补虚拟化技术中的不足（动态数据存储，虚拟化技术提供给用户的是一个和实际生产环境一样的操作系统环境，主要用于搭建某些应用或环境，数据存储应当独立于虚拟化的存储并且可以动态挂载到任何需要的地方，但是传统的虚拟化平台通常是使用块存储实现，不够将数据独立开来，而Docker可以借用Minio这种类S3存储将远程的数据安全挂载到虚拟机中。）

  ```
  异同：
  1、性能：虚拟化技术是在硬件层面实现的虚拟化，需要有虚拟机管理应用和虚拟机操作系统层，容器是在操作系统层面实现的虚拟化，共享宿主机的操作系统内核，更加轻量级。
  2、容器不需要启动一个完整的操作系统，但是虚拟机在启动时必须加载完整的操作系统并且要虚拟化系统资源，所以容器的启动速度远远快于虚拟机。
  ```

  ​

  > A container: • uses the host kernel • can’t boot a different OS • can’t have it’s own modules • it’s a bunch of processes visible on the host machine (VMs are totally opaque) Linux container’s building bricks are Control Groups (cgroups): • Each subsystem has hierarchy (Memory, CPU, Block I/O ...) • Hierarchies are indipendent and namespaces: • Provides processes with their system own view • Each process is in one namespace of each type (pid, net, mnt ...) Cgroups limits “how much” you can use, Namespace limits “what you can see” (and use)

## 技术选型

- 底层虚拟机化技术的选择（最后选择KVM）

  ```
  KVM XEN Hyper-v
  ```

- 对象存储、S3协议、Minio（最后选择Docker部署Minio，主要是为了适应多租户模式multi-tenancy technology和平台统一管理）

- 多租户模式的优点：

  - 降低服务的开发成本，将服务所需资源利用最大化
  - 从服务提供角度来讲，开发的一个服务运行时可以同时给多个客户提供使用，并且客户之间的数据和状态保持隔离。
  - 从使用的角度来讲，当不同的客户使用同一个服务时，使用服务完成的业务是互不影响的，好像是在独享服务一样。
  - ​

  ```
  对象存储提供Key-Value（简称K/V）方式的RESTful数据读写接口，并且常以网络服务的形式提供数据的访问。基于Http协议的Restful Web API，通过HTTP请求中的PUT、GET和DELETE对文件进行上传（写入）、下载（读取）和删除。与文件系统相比，以AWS S3和Swift为代表的对象存储有两个显著的特征——REST风格的接口和扁平的数据组织结构。目录树会给存储系统带来很大的开销，对象存储的扁平化数据组织结构由于抛弃了嵌套的文件夹，从而避免了维护庞大的目录树。
  S3的全称是Simple Storage Service，Amazon S3 is a web service that enables you to store data in the cloud.
  minio非结构化数据对象存储服务。
  ```

- 其他方式的存储（NFS、webdav、CIFS，为什么不选这些）


   - NFS：

    > 优点：简单，容易上手和掌握；NFS文件系统内数据是在文件系统之上的，即数据是能看得见的；部署快速，可控，满足需求；数据可靠性高，经久耐用；服务稳定；
    >
    > 缺点：存在单点故障，如果NFS_Server宕机了，所有客户端都无法访问共享目录；  这在后期会通过 负载均衡及高可用 方案弥补；在大数据高并发的场合，NFS效率、性能有限；客户端认证是基于 IP和主机名，权限根据ID识别，安全性一般（用于内网则问题不大）；NFS数据是明文，NFS本身不对数据完整行进行验证；多台Client挂载一个Server时，连接管理维护麻烦。尤其在Server出问题后，所有Client都处于挂掉状态；涉及了同步（实时等待）和异步（解耦）的概念，NFS服务器和客户端相对来说就是耦合度有些高；

  - ​

## 平台管理技术模型/架构

- 总体架构图
- 单机模式的模型
  - 从本地扩展出单用户多Bucket（默认即是如此）
- 多机模式的模型
  - 从远程服务器中扩展出多用户多Bucket（如何实现）

## 平台实现

- 根据REST API原则设计的具体API内容
- 实现后的一部分截图

## 结语

- 本管理设计满足了我们的实际生产需求，对于在如今越来越庞大的数据集，将要用到多种复杂的虚拟机搭建的环境中，比如Tensorflow、Spark等多种计算框架中。
- 虚拟机本身的硬盘空间永远是不会得到提升的，只需要能够容纳最小的计算环境或者应用环境的资源就可以，具体的数据存储应当远离环境而独立开来，这样服务能够实现最大的有效性和安全性。