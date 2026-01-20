# 附录B 配置文件模板

## B.1 作业调度系统配置

### B.1.1 Slurm 现代配置 (23.02+)
#### slurm.conf (核心配置)
```ini
# /etc/slurm/slurm.conf
# 集群基础信息
ClusterName=hpc-ai-cluster
ControlMachine=slurm-ctl-01
ControlAddr=10.10.1.1
BackupController=slurm-ctl-02
BackupAddr=10.10.1.2

# 调度策略 (AI/HPC 混合负载优化)
SchedulerType=sched/backfill
SelectType=select/cons_tres
SelectTypeParameters=CR_Core_Memory,CR_ONE_TASK_PER_CORE

# 优先级与抢占
PriorityType=priority/multifactor
PriorityDecayHalfLife=14-0
PreemptType=preempt/qos
PreemptMode=REQUEUE

# 节点定义 (H100 GPU 节点示例)
NodeName=gpu-h100-[01-32] CPUs=128 RealMemory=2097152 Sockets=2 CoresPerSocket=64 ThreadsPerCore=1 Gres=gpu:h100:8 State=UNKNOWN Feature=MyFeature

# 分区定义
PartitionName=debug Nodes=gpu-h100-[01-02] Default=NO MaxTime=02:00:00 State=UP QoS=debug
PartitionName=gpu_h100 Nodes=gpu-h100-[03-32] Default=YES MaxTime=7-00:00:00 State=UP

# 记账与日志
JobAcctGatherType=jobacct_gather/linux
JobAcctGatherFrequency=30
SlurmctldLogFile=/var/log/slurm/slurmctld.log
SlurmdLogFile=/var/log/slurm/slurmd.log
```

#### gres.conf (GPU资源)
```ini
# /etc/slurm/gres.conf
# 自动检测 (推荐用于 NVIDIA GPU)
AutoDetect=nvml
```

#### cgroup.conf (资源隔离)
```ini
# /etc/slurm/cgroup.conf
CgroupPlugin=cgroup/v2
ConstrainCores=yes
ConstrainDevices=yes
ConstrainRAMSpace=yes
ConstrainSwapSpace=yes
ConstrainKmemSpace=no  # 避免内核内存泄漏问题
```

## B.2 容器与 AI 环境配置

### B.2.1 Enroot 配置 (NVIDIA 容器)
#### enroot.conf
```ini
# /etc/enroot/enroot.conf
# 存储路径 (建议挂载高性能大容量存储)
ENROOT_RUNTIME_PATH /mnt/nvme/enroot/runtime
ENROOT_CACHE_PATH   /mnt/nvme/enroot/cache
ENROOT_DATA_PATH    /mnt/nvme/enroot/data

# 挂载配置
ENROOT_MOUNT_HOME   yes
ENROOT_RESTRICT_DEV yes
```

### B.2.2 PyTorch 分布式训练启动脚本
#### torch_launch.sh
```bash
#!/bin/bash
# SBATCH --job-name=llama3-train
# SBATCH --nodes=4
# SBATCH --gpus-per-node=8
# SBATCH --ntasks-per-node=1  # PyTorch 通常每节点启动一个主进程
# SBATCH --cpus-per-task=96

# 环境设置
export NCCL_DEBUG=INFO
export NCCL_IB_HCA=^mlx5_0:1
export OMP_NUM_THREADS=1

# 获取主节点 IP
MASTER_ADDR=$(scontrol show hostnames $SLURM_JOB_NODELIST | head -n 1)
MASTER_PORT=29500

# 启动训练
srun torchrun \
    --nnodes=$SLURM_JOB_NUM_NODELIST \
    --nproc_per_node=8 \
    --rdzv_id=$SLURM_JOB_ID \
    --rdzv_backend=c10d \
    --rdzv_endpoint=$MASTER_ADDR:$MASTER_PORT \
    train.py --config config.yaml
```

## B.3 监控系统配置 (Prometheus/Grafana)

### B.3.1 Prometheus 采集配置
#### prometheus.yml
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  # 节点基础监控 (Node Exporter)
  - job_name: 'hpc_nodes'
    file_sd_configs:
      - files:
        - '/etc/prometheus/targets/nodes/*.yml'

  # NVIDIA GPU 监控 (DCGM Exporter)
  - job_name: 'gpu_dcgm'
    scrape_interval: 10s
    static_configs:
      - targets: ['compute-01:9400', 'compute-02:9400']

  # Slurm 监控
  - job_name: 'slurm_exporter'
    static_configs:
      - targets: ['slurm-ctl:9341']
```

### B.3.2 Grafana 告警规则 (JSON Model)
#### alert_rules.yaml (Prometheus Rule)
```yaml
groups:
- name: HPC_Alerts
  rules:
  # GPU Xid 错误告警
  - alert: GPUXidError
    expr: DCGM_FI_DEV_XID_ERRORS > 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "GPU Xid Error detected on {{ $labels.instance }}"

  # 节点宕机告警
  - alert: NodeDown
    expr: up{job="hpc_nodes"} == 0
    for: 3m
    labels:
      severity: critical
    annotations:
      summary: "Node {{ $labels.instance }} is down"
```

## B.4 存储系统配置

### B.4.1 Lustre 客户端配置
#### /etc/modprobe.d/lustre.conf
```bash
# 网络超时优化 (适应大规模集群)
options lnet networks="o2ib0(ib0)" config_on_load=1
options ptlrpc at_min=30 at_max=600
```
#### 挂载脚本
```bash
mount -t lustre -o localflock,lazystatfs 10.10.1.100@o2ib:/lustre /mnt/lustre
```

## B.5 网络配置 (InfiniBand)

### B.5.1 IPoIB 配置 (RedHat/Rocky)
#### /etc/sysconfig/network-scripts/ifcfg-ib0
```bash
DEVICE=ib0
TYPE=InfiniBand
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.100.1
PREFIX=24
CONNECTED_MODE=yes  # 开启 Connected Mode 以支持更大的 MTU
MTU=65520           # IPoIB 最大 MTU
```