### cgroup
在 Linux 上，控制组（CGroup）用于限制分配给进程的资源。
kubelet 和底层容器运行时都需要对接 cgroup 来强制执行为 Pod 和容器管理资源， 这包括为容器化工作负载配置 CPU/内存请求和限制。

Linux 中有两个 cgroup 版本：cgroup v1 和 cgroup v2。cgroup v2 是新一代的 cgroup API。

cgroup v2 对 cgroup v1 进行了多项改进，例如：
- API 中单个统一的层次结构设计
- 更安全的子树委派给容器
- 更新的功能特性， 例如压力阻塞信息（Pressure Stall Information，PSI）
- 跨多个资源的增强资源分配管理和隔离
    -  统一核算不同类型的内存分配（网络内存、内核内存等）
    -  考虑非即时资源变化，例如页面缓存回写

一些 Kubernetes 特性专门使用 cgroup v2 来增强资源管理和隔离。 例如，MemoryQoS 特性改进了内存 QoS 并依赖于 cgroup v2 原语。

#### 版本判断 
- group 版本取决于正在使用的 Linux 发行版和操作系统上配置的默认 cgroup 版本。 要检查你的发行版使用的是哪个 cgroup 版本，请在该节点上运行 `stat -fc %T /sys/fs/cgroup/`  命令：
  - 对于 cgroup v2，输出为 cgroup2fs。
  - 对于 cgroup v1，输出为 tmpfs。 

#### 驱动分类