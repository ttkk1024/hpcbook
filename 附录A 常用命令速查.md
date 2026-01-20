# 附录A 常用命令速查

## A.1 Linux 系统管理

### 基础运维
#### 软件包管理 (DNF/YUM)
```bash
dnf update -y               # 更新所有包
dnf install <package>       # 安装软件包
dnf search <keyword>        # 搜索软件包
dnf history                 # 查看事务历史
dnf clean all               # 清理缓存
dnf info <package>          # 查看包信息
dnf group list              # 列出软件包组
dnf group install "Development Tools" # 安装开发工具组
```

#### 服务管理 (Systemd)
```bash
systemctl status <service>  # 查看服务状态
systemctl start/stop/restart <service> # 启停/重启服务
systemctl enable/disable <service> # 设置开机自启/禁用
systemctl list-units --type=service # 列出活跃的服务
journalctl -u <service> -f  # 实时查看服务日志
journalctl -xe              # 查看最后具体的报错信息
journalctl --disk-usage     # 查看日志占用空间
```

#### 网络配置 (NMCLI/IP)
```bash
# NetworkManager
nmcli device status         # 查看网卡状态
nmcli connection show       # 查看连接配置
nmcli con up/down <eth0>    # 启用/禁用网卡
nmcli con mod <eth0> ipv4.addresses 192.168.1.10/24 # 修改IP
nmcli con mod <eth0> ipv4.gateway 192.168.1.1 # 修改网关
nmcli con mod <eth0> ipv4.dns "8.8.8.8" # 修改DNS

# Modern IP Tools
ip addr show                # 显示IP地址
ip route show               # 显示路由表
ip neigh show               # 显示ARP缓存
ss -tuln                    # 查看监听端口 (替代netstat)
ss -ta                      # 查看所有TCP连接
ethtool <eth0>              # 查看网卡物理状态
```

### 监控与性能
#### 现代监控工具
```bash
htop                        # 交互式进程监控
btop                        # 现代风格的资源监控 (推荐)
dstat -cmdsn                # 综合资源统计
glances                     # 跨平台系统监控
```

#### 磁盘与I/O
```bash
df -hT                      # 查看磁盘空间及文件系统类型
du -sh <dir>                # 查看目录大小
iotop -o                    # 实时显示I/O进程
iostat -xz 1                # 详细磁盘I/O统计
ncdu <dir>                  # 交互式磁盘空间分析 (推荐)
```

## A.2 AI 与 GPU 计算

### NVIDIA GPU 管理
#### 状态监控 (nvidia-smi)
```bash
nvidia-smi                  # 查看GPU基本状态
nvidia-smi -l 1             # 每秒刷新一次
nvidia-smi -q -d MEMORY     # 查看显存详情
nvidia-smi -q -d TEMPERATURE # 查看温度详情
nvidia-smi --query-gpu=timestamp,name,utilization.gpu,memory.used --format=csv -l 1 # 自定义CSV输出 (适合记录)
nvidia-smi topo -m          # 查看拓扑与NVLink连接
```

#### 深度诊断 (DCGM)
```bash
dcgmi discovery -l          # 列出GPU列表
dcgmi diag -r 3             # 运行 Level 3 深度诊断 (约15分钟)
dcgmi stats -g <group> -a   # 记录性能统计
dcgmi health -c             # 检查健康状态 (ECC, PCIe, 热节流)
```

#### 生产环境配置
```bash
nvidia-smi -pm 1            # 启用持久模式 (Persistence Mode)
nvidia-smi -ac 1215,1410    # 锁定应用时钟频率 (Memory,Graphics) - 提升稳定性
nvidia-smi -pl <watts>      # 设置功率限制
nvcc --version              # 查看CUDA编译器版本
```

### 分布式通信 (NCCL/InfiniBand)
```bash
ibstat                      # 查看IB网卡状态
ibv_devinfo                 # 详细IB设备信息
iblinkinfo                  # 查看IB交换机链路速率
ib_write_bw -d mlx5_0 <ip>  # 测试IB写带宽 (RDMA)
/usr/local/bin/all_reduce_perf -b 8 -e 1G -f 2 -g 8 # NCCL AllReduce 性能测试
```

## A.3 HPC 作业调度 (Slurm)

### 用户常用命令
#### 作业提交与管理
```bash
sbatch <script.sh>          # 提交批处理作业
salloc -N 1 -n 4 -t 1:00:00 # 申请交互式资源
srun --pty bash             # 启动交互式Shell (需配合salloc或直接运行)
squeue                      # 查看作业队列
squeue -u <user>            # 查看特定用户的作业
scancel <jobid>             # 取消作业
scancel -u <user>           # 取消该用户所有作业
```

#### 状态查询
```bash
sinfo                       # 查看分区与节点简报
sinfo -N -l                 # 查看节点详细状态
scontrol show job <jobid>   # 查看作业详细信息 (含脚本路径、环境等)
scontrol show node <node>   # 查看节点详细信息 (含CPU/内存/GRES配置)
sacct -j <jobid> --format=JobID,JobName,MaxRSS,Elapsed # 查看历史作业资源使用
```

### 管理员命令
```bash
scontrol update NodeName=<node> State=DRAIN Reason="Maintenance" # 下线节点
scontrol update NodeName=<node> State=RESUME # 上线节点
scontrol update JobId=<jobid> Priority=10000 # 调整作业优先级
sdiag                       # 查看调度器统计信息
sprio                       # 查看作业优先级明细
```

## A.4 容器技术

### Apptainer / Singularity (HPC环境)
```bash
apptainer build <image.sif> <def_file> # 构建镜像
apptainer pull docker://alpine # 从DockerHub拉取
apptainer shell <image.sif>    # 进入容器Shell
apptainer exec --nv <image.sif> python script.py # 挂载GPU运行命令
apptainer run --bind /data:/data <image.sif> # 挂载目录运行
```

### Docker / Podman (云原生环境)
```bash
docker images               # 列出镜像
docker ps -a                # 列出所有容器
docker run --gpus all -it --rm <image> bash # 启动GPU容器
docker exec -it <container_id> bash # 进入运行中的容器
docker logs -f <container_id> # 查看容器日志
docker system prune -a      # 清理所有未使用的镜像和容器
podman run --security-opt=label=disable ... # Podman特有 SELinux 选项
```

### Enroot / Pyxis (NVIDIA优化的容器)
```bash
enroot import docker://nvidia/pytorch:23.12-py3 # 导入镜像
enroot create -n pytorch nvidia+pytorch+23.12-py3.sqsh # 创建容器文件系统
enroot start --rw pytorch   # 启动容器
# Slurm 中使用 Pyxis
srun --container-image=nvidia/pytorch:23.12-py3 python train.py
```

## A.5 存储与数据传输

### 高性能并行存储 (Lustre)
```bash
lfs df -h                   # 查看Lustre容量及Inode
lfs quota -u <user> /mnt/lustre # 查看配额
lfs getstripe <file/dir>    # 查看条带参数
lfs setstripe -c 4 -S 4M <dir> # 设置条带 (4个OST，4MB大小)
lfs find /mnt/lustre -size +1G # 高效查找大文件
```

### 对象存储与云同步 (Rclone)
```bash
rclone config               # 配置云存储 (S3, OSS, Google Drive等)
rclone lsd remote:          # 列出桶
rclone ls remote:bucket     # 列出文件
rclone copy <local_dir> remote:bucket/path -P # 上传 (带进度条)
rclone sync remote:bucket <local_dir> # 同步 (注意：会删除本地差异文件)
rclone mount remote:bucket /mnt/local --daemon # 挂载为本地文件系统
```

### 高速数据传输
```bash
rsync -avP <source> <dest>  # 本地/远程增量同步 (显示进度)
scp -r <source> <user>@<host>:<dest> # SSH复制
bbcp -P 2 -w 4m <source> <dest> # 多流高速复制 (需安装bbcp)
```

## A.6 代码管理与 DevOps (Ansible/Git)

### Git 常用操作
```bash
git clone <repo_url>        # 克隆仓库
git status                  # 查看文件状态
git add .                   # 添加所有变更
git commit -m "msg"         # 提交变更
git push origin main        # 推送到远程
git pull                    # 拉取远程更新
git checkout -b <branch>    # 创建并切换分支
git log --oneline --graph   # 查看简洁提交历史
```

### Ansible 自动化
```bash
# Ad-hoc 命令 (临时任务)
ansible all -m ping         # 测试所有节点连通性
ansible compute -m shell -a "uptime" # 在计算组执行uptime
ansible gpu_nodes -m shell -a "nvidia-smi" # 检查GPU节点状态
ansible all -m copy -a "src=hosts dest=/etc/hosts" # 分发文件
ansible-playbook site.yml   # 运行剧本
```