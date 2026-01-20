# 附录D 参考资源列表

## D.1 行业组织与标准
| 组织名称 | 网址 | 描述 | 关键资源 |
| :--- | :--- | :--- | :--- |
| **TOP500** | top500.org | 全球超算排名 | 绿色HPC列表 (Green500), HPL基准 |
| **MLCommons** | mlcommons.org | AI性能基准 | MLPerf Training/Inference 规则与结果 |
| **CNCF** | cncf.io | 云原生基金会 | Kubernetes, Prometheus, Helm 规范 |
| **PyTorch Foundation** | pytorch.org | AI 框架 | 分布式训练文档, 生态系统 |
| **IBTA** | infinibandta.org | InfiniBand 标准 | IB 规范, RoCEv2 标准 |

## D.2 核心技术文档
### D.2.1 调度与管理
- **Slurm Workload Manager**: [slurm.schedmd.com](https://slurm.schedmd.com)
  - 推荐阅读: *Slurm on GPU Clusters*, *Cgroup v2 Support*
- **Apptainer (原 Singularity)**: [apptainer.org](https://apptainer.org)
  - 推荐阅读: *GPU Support*, *MPI Integration*
- **Warewulf 4**: [warewulf.org](https://warewulf.org)
  - 推荐阅读: *Stateless Provisioning*

### D.2.2 AI 与 GPU 计算
- **NVIDIA Developer**: [developer.nvidia.com](https://developer.nvidia.com)
  - 文档: *CUDA Toolkit Documentation*, *DCGM Data Center GPU Manager*
- **Hugging Face**: [huggingface.co](https://huggingface.co)
  - 文档: *Transformers*, *Accelerate*, *Peft* (大模型微调必备)

### D.2.3 存储与网络
- **Lustre File System**: [wiki.lustre.org](https://wiki.lustre.org)
  - 手册: *Lustre Operations Manual*
- **DAOS**: [docs.daos.io](https://docs.daos.io)
  - 指南: *Intel Optane Persistent Memory Storage*

## D.3 推荐书籍 (2025版)
### D.3.1 体系结构与系统
1. **《Computer Architecture: A Quantitative Approach (6th Edition)》**
   - 作者: Hennessy & Patterson
   - 重点: 领域专用架构 (DSA), GPU 架构
2. **《Systems Performance: Enterprise and the Cloud (2nd Edition)》**
   - 作者: Brendan Gregg
   - 重点: eBPF, 性能监测方法论

### D.3.2 现代 HPC 与 AI
1. **《High Performance Computing: Modern Systems and Practices》**
   - 作者: Thomas Sterling 等
   - 重点: Beowulf 集群, 现代并行编程
2. **《Deep Learning Systems》** (在线课程/书)
   - 作者: Tianqi Chen (陈天奇) 等
   - 重点: 深度学习编译栈, 自动微分原理

## D.4 在线学习平台与社区
- **NVIDIA Deep Learning Institute (DLI)**: 提供实战 GPU 编程实验。
- **Coursera - HPC Series**: 普林斯顿大学的 HPC 经典课程。
- **HPCwire / The Next Platform**: 获取超算与 AI 芯片最新资讯的权威媒体。
- **Reddit r/HPC**: 活跃的全球 HPC 系统管理员讨论区。

## D.5 开源工具箱
| 工具分类 | 推荐工具 | 用途 |
| :--- | :--- | :--- |
| **监控** | Prometheus, Grafana, DCGM-Exporter | AI 集群全栈监控 |
| **日志** | Loki, ELK Stack | 分布式日志聚合 |
| **构建** | Spack, EasyBuild | 科学软件自动化构建 |
| **部署** | Ansible, Terraform | 基础设施即代码 (IaC) |
| **网络** | Wireshark, Mellanox NEO | 网络抓包与 IB 管理 |