# 附录A 常用命令速查

## Linux系统管理

### 基础命令

#### 文件和目录操作
```bash
# 文件操作
ls [选项] [路径]           # 列出目录内容
ls -l                      # 详细列表格式
ls -lh                     # 人类可读格式
ls -la                     # 显示隐藏文件
cp [选项] 源文件 目标文件   # 复制文件
cp -r 源目录 目标目录       # 递归复制目录
mv [选项] 源文件 目标文件   # 移动/重命名文件
rm [选项] 文件             # 删除文件
rm -r 目录                 # 递归删除目录
mkdir [选项] 目录名        # 创建目录
mkdir -p 路径              # 创建多级目录
rmdir 目录                 # 删除空目录
touch 文件名               # 创建空文件或更新时间戳
cat 文件                   # 显示文件内容
head -n 文件               # 显示文件前n行
tail -n 文件               # 显示文件后n行
tail -f 文件               # 实时显示文件末尾（日志监控）
grep [选项] 模式 文件      # 搜索文件内容
grep -r 模式 目录          # 递归搜索目录
find [路径] [表达式]       # 查找文件
find . -name "*.log"       # 查找特定扩展名文件
chmod [选项] 权限 文件     # 修改文件权限
chmod 755 文件             # 设置权限为rwxr-xr-x
chown [选项] 用户:组 文件  # 修改文件所有者
chown user:group file      # 设置用户和组

# 文件查看和编辑
cat 文件                   # 显示完整文件内容
more 文件                  # 分页显示文件
less 文件                  # 可滚动的分页显示
head -20 文件              # 显示前20行
tail -20 文件              # 显示后20行
tail -f 文件               # 实时监控文件变化
grep "pattern" 文件        # 搜索文件中的模式
sed 's/old/new/g' 文件     # 替换文本
awk '{print $1}' 文件      # 提取列
```

#### 系统信息
```bash
# 系统状态
uname -a                   # 显示系统信息
cat /etc/os-release        # 显示发行版信息
cat /proc/cpuinfo          # 显示CPU信息
cat /proc/meminfo          # 显示内存信息
lscpu                      # CPU详细信息
free -h                    # 内存使用情况（人类可读）
df -h                      # 磁盘使用情况
du -sh 目录                # 目录大小
ps [选项]                  # 显示进程信息
ps aux                     # 显示所有进程
top                        # 实时进程监控
htop                       # 增强的进程监控
uptime                     # 系统运行时间
who                        # 当前登录用户
w                          # 用户和系统负载
date                       # 显示日期时间
cal                        # 显示日历
```

#### 网络命令
```bash
# 网络诊断
ifconfig                   # 显示网络接口（旧版）
ip addr show               # 显示网络接口（新版）
ip route show              # 显示路由表
ping [选项] 主机           # 测试网络连通性
ping -c 4 google.com       # 发送4个ping包
netstat [选项]             # 显示网络连接
netstat -tuln              # 显示监听端口
ss [选项]                  # 现代网络状态工具
ss -tuln                   # 显示监听端口
traceroute 主机            # 路由跟踪
nslookup 域名              # DNS查询
dig 域名                   # 详细的DNS查询
hostname                   # 显示主机名
hostname -I                # 显示IP地址
```

#### 系统管理
```bash
# 服务管理
systemctl start 服务名     # 启动服务
systemctl stop 服务名      # 停止服务
systemctl restart 服务名   # 重启服务
systemctl status 服务名    # 查看服务状态
systemctl enable 服务名    # 设置开机启动
systemctl disable 服务名   # 禁止开机启动
systemctl list-units       # 列出所有运行的服务

# 软件包管理
yum install 软件包         # 安装软件包（CentOS/RHEL）
yum remove 软件包          # 卸载软件包
yum update 软件包          # 更新软件包
yum list installed         # 列出已安装的包
yum search 关键词          # 搜索软件包

apt update                 # 更新包列表（Ubuntu/Debian）
apt install 软件包         # 安装软件包
apt remove 软件包          # 卸载软件包
apt upgrade                # 升级所有包
apt search 关键词          # 搜索软件包

# 用户管理
useradd 用户名             # 创建用户
userdel 用户名             # 删除用户
passwd 用户名              # 修改密码
usermod [选项] 用户名      # 修改用户属性
id 用户名                  # 查看用户ID信息
groups 用户名              # 查看用户组
whoami                     # 显示当前用户
su 用户名                  # 切换用户
sudo 命令                  # 以管理员权限执行
```

### 进程和资源管理

#### 进程监控
```bash
# 进程查看
ps aux                     # 显示所有进程
ps aux | grep 进程名       # 查找特定进程
ps -ef                     # 标准格式显示进程
top                        # 实时进程监控
htop                       # 增强的进程监控工具
jobs                       # 显示后台作业
bg %作业号                 # 将作业放到后台
fg %作业号                 # 将作业调到前台

# 进程控制
kill PID                   # 终止进程
kill -9 PID                # 强制终止进程
killall 进程名             # 终止所有同名进程
pkill 模式                 # 根据模式终止进程
nohup 命令 &               # 后台运行不挂断
Ctrl+Z                     # 暂停当前进程
Ctrl+C                     # 终止当前进程
Ctrl+D                     # 退出shell
```

#### 资源监控
```bash
# CPU和内存
top                        # 实时资源使用
htop                       # 增强的资源监控
vmstat [选项]              # 虚拟内存统计
vmstat 1                   # 每秒更新一次
iostat [选项]              # I/O统计
iostat -x 1                # 详细I/O统计
sar [选项]                 # 系统活动报告
sar -u 1                   # CPU使用率
sar -r 1                   # 内存使用率
sar -d 1                   # 磁盘I/O

# 磁盘和I/O
df -h                      # 文件系统使用情况
du -sh 目录                # 目录大小
iotop                      # I/O实时监控
lsof [选项]                # 列出打开的文件
lsof -i                      # 列出网络连接
fuser 文件                 # 显示使用文件的进程
```

### 网络和安全

#### 网络配置
```bash
# 接口配置
ip addr add IP/掩码 dev 接口 # 添加IP地址
ip addr del IP/掩码 dev 接口 # 删除IP地址
ip link set 接口 up         # 启用接口
ip link set 接口 down       # 禁用接口
ip route add 网络 via 网关  # 添加路由
ip route del 网络           # 删除路由

# 防火墙
firewall-cmd --list-all     # 显示防火墙状态
firewall-cmd --add-port=端口/tcp --permanent  # 添加端口
firewall-cmd --reload       # 重新加载防火墙
iptables -L                 # 列出iptables规则
iptables -A INPUT -p tcp --dport 端口 -j ACCEPT # 添加规则

# SSH
ssh 用户@主机               # SSH连接
ssh -p 端口 用户@主机       # 指定端口连接
scp 文件 用户@主机:路径     # 复制文件到远程
scp -r 目录 用户@主机:路径   # 递归复制目录
rsync [选项] 源 目标        # 同步文件
rsync -avz 源 目标          # 压缩同步
```

## HPC专用命令

### 作业调度系统

#### SLURM命令
```bash
# 作业提交
sbatch [选项] 脚本         # 提交作业
sbatch --job-name=作业名 脚本 # 指定作业名
sbatch --partition=分区 脚本 # 指定分区
sbatch --nodes=N --ntasks-per-node=M 脚本 # 指定资源

# 交互式作业
salloc [选项]              # 分配资源
salloc --nodes=1 --ntasks=32 --time=1:00:00
srun [选项] 命令           # 运行作业
srun --pty /bin/bash       # 获取交互式shell

# 作业监控
squeue                     # 查看队列
squeue -u 用户名           # 查看用户作业
squeue -p 分区名           # 查看分区作业
squeue -l                  # 详细信息
scontrol show job 作业ID   # 作业详细信息
scontrol show node 节点名  # 节点信息

# 作业管理
scancel 作业ID             # 取消作业
scancel -u 用户名          # 取消用户所有作业
scancel -t pending         # 取消排队中的作业
scontrol hold 作业ID       # 挂起作业
scontrol release 作业ID    # 恢复作业
scontrol update jobid=作业ID TimeLimit=新时间 # 修改时间

# 资源查询
sinfo                      # 查看分区状态
sinfo -N -l                # 节点详细信息
scontrol show config       # 显示配置
scontrol ping              # 测试连接

# 性能统计
sacct                      # 作业会计信息
sacct -u 用户名            # 用户统计
sacct -j 作业ID --format=JobID,JobName,Partition,Elapsed,CPUTime # 格式化输出
sreport                    # 生成报告
```

#### PBS Pro命令
```bash
# 作业提交
qsub [选项] 脚本          # 提交作业
qsub -N 作业名 脚本       # 指定作业名
qsub -q 队列名 脚本       # 指定队列
qsub -l nodes=N:ppn=M 脚本 # 指定资源

# 交互式作业
qsub -I [选项]            # 交互式作业
qsub -I -q 队列名 -l walltime=1:00:00

# 作业监控
qstat                     # 查看队列
qstat -u 用户名           # 查看用户作业
qstat -q 队列名           # 查看队列信息
qstat -f 作业ID           # 作业详细信息
qstat -B                  # 服务器信息

# 作业管理
qdel 作业ID               # 删除作业
qdel -p 作业ID            # 暂停删除
qsig -s SUSPEND 作业ID    # 挂起作业
qsig -s CONTINUE 作业ID   # 恢复作业
qalter 作业ID -l walltime=新时间 # 修改时间

# 资源查询
pbsnodes                  # 节点信息
pbsnodes -a               # 所有节点
pbsnodes -l               # 节点负载
qmgr -c "list node"       # 列出节点
qmgr -c "list queue"      # 列出队列
```

#### LSF命令
```bash
# 作业提交
bsub [选项] 命令          # 提交作业
bsub -J 作业名 命令       # 指定作业名
bsub -q 队列名 命令       # 指定队列
bsub -n N -W 时间 命令    # 指定资源

# 交互式作业
bsub -I [选项] 命令       # 交互式作业
bsub -I -q 队列名 -n 32 -W 1:00:00

# 作业监控
bjobs                     # 查看作业
bjobs -u 用户名           # 查看用户作业
bjobs -q 队列名           # 查看队列作业
bjobs -l 作业ID           # 作业详细信息

# 作业管理
bkill 作业ID              # 终止作业
bkill -9 作业ID           # 强制终止
bstop 作业ID              # 挂起作业
bresume 作业ID            # 恢复作业
bmod -q 队列 作业ID       # 修改队列

# 资源查询
bhosts                    # 主机信息
bhosts -l                 # 详细主机信息
bqueues                   # 队列信息
bqueues -l                # 详细队列信息
```

### 并行计算

#### MPI命令
```bash
# OpenMPI
mpirun [选项] 程序        # 运行MPI程序
mpirun -np 32 ./程序      # 指定进程数
mpirun -hostfile 主机文件 ./程序 # 指定主机
mpirun -map-by node ./程序 # 按节点映射

# Intel MPI
mpiexec [选项] 程序       # 运行MPI程序
mpiexec -n 32 ./程序      # 指定进程数
mpiexec -ppn 8 ./程序     # 每节点进程数

# MPICH
mpiexec [选项] 程序       # 运行MPI程序
mpiexec -n 32 ./程序      # 指定进程数
```

#### OpenMP环境变量
```bash
export OMP_NUM_THREADS=32  # 设置线程数
export OMP_PROC_BIND=TRUE  # 绑定线程到核心
export OMP_PLACES=cores    # 线程放置策略
export KMP_AFFINITY=compact # Intel线程绑定
export GOMP_CPU_AFFINITY="0-31" # GCC线程绑定
```

### 性能分析

#### 基础性能工具
```bash
# CPU性能
perf top                  # 实时性能分析
perf record 程序          # 记录性能数据
perf report               # 生成报告
oprofile                  # OProfile性能分析
valgrind 程序             # 内存和性能分析
valgrind --tool=memcheck 程序 # 内存检查

# I/O性能
fio [选项]                # I/O基准测试
fio --name=test --ioengine=libaio --rw=read --bs=4k --size=1G --numjobs=4
ioping [选项] 设备        # I/O延迟测试
dd if=/dev/zero of=测试文件 bs=1M count=1000 # 简单I/O测试

# 网络性能
iperf3 -s                 # 服务端
iperf3 -c 服务器IP        # 客户端
ntttcp -s                 # NTTTCP服务端
ntttcp -r                 # NTTTCP接收端

# 系统监控
sar -u 1 10               # CPU使用率
sar -r 1 10               # 内存使用率
sar -d 1 10               # 磁盘I/O
sar -n DEV 1 10           # 网络接口
```

#### HPC专用工具
```bash
# 性能基准测试
linpack                   # LINPACK基准测试
stream                    # 内存带宽测试
hpl                       # 高性能Linpack
iozone                    # 文件系统基准测试
imb                       # MPI基准测试

# 作业性能分析
sacct -j 作业ID --format=JobID,JobName,Partition,Elapsed,MaxRSS,MaxVMSize
sreport cluster Allocated -t hour
```

### 存储系统

#### Lustre文件系统
```bash
# 客户端操作
mount -t lustre MDS节点:/文件系统 /挂载点 # 挂载
umount /挂载点            # 卸载
lfs df                    # 显示磁盘使用
lfs df -h                 # 人类可读格式
lfs getstripe 文件        # 查看条带化信息
lfs setstripe -c 4 -S 1m 文件 # 设置条带化
lfs find /路径 -type f    # 查找文件
lfs migrate 文件          # 迁移文件

# 服务端操作
lctl get_param mds.*      # 查看MDS参数
lctl get_param oss.*      # 查看OSS参数
lctl get_param osc.*      # 查看客户端参数
lctl set_param 参数=值    # 设置参数
```

#### GPFS文件系统
```bash
# 基本操作
mmlsfs all                # 列出文件系统
mmdf 文件系统             # 显示磁盘使用
mmgetstate -a             # 查看节点状态
mmstartup                 # 启动GPFS
mmshutdown                # 关闭GPFS

# NSD操作
mmlsnsd                   # 列出NSD
mmcrnsd -F 配置文件       # 创建NSD
mmchfs 文件系统 -B 1m     # 修改条带大小
```

#### 本地存储
```bash
# LVM管理
pvcreate 设备             # 创建物理卷
vgcreate 卷组 物理卷      # 创建卷组
lvcreate -L 大小 -n 逻辑卷 卷组 # 创建逻辑卷
mkfs.xfs 设备             # 创建XFS文件系统
mount 设备 挂载点         # 挂载文件系统

# RAID管理
mdadm --create /dev/md0 --level=10 --raid-devices=4 设备列表 # 创建RAID
mdadm --detail /dev/md0   # 查看RAID状态
cat /proc/mdstat          # 查看RAID统计

# 磁盘工具
fdisk -l                  # 分区表信息
parted -l                 # 分区信息
lsblk                     # 块设备列表
blkid                     # 设备UUID
hdparm -tT 设备           # 磁盘性能测试
```

### 网络配置

#### InfiniBand
```bash
# 基本信息
ibstat                    # InfiniBand状态
ibhosts                   # InfiniBand主机
iblinkinfo                # InfiniBand链路信息
ibv_devinfo               # 设备信息
ibv_devices               # 设备列表

# 性能测试
ibping 远程节点           # Ping测试
ib_send_bw 远程节点       # 带宽测试
ib_read_bw 远程节点       # 读带宽测试
ib_write_bw 远程节点      # 写带宽测试

# 配置管理
opensm                    # 子网管理器
ibnetdiscover             # 网络发现
```

#### 网络绑定
```bash
# 创建bond接口
modprobe bonding          # 加载模块
echo +bond0 > /sys/class/net/bonding_masters # 添加bond
echo active-backup > /sys/class/net/bond0/bonding/mode # 设置模式
echo +eth0 > /sys/class/net/bond0/bonding/slaves # 添加从设备
echo +eth1 > /sys/class/net/bond0/bonding/slaves # 添加从设备

# 配置文件方式
# /etc/sysconfig/network-scripts/ifcfg-bond0
DEVICE=bond0
TYPE=Bond
BONDING_MASTER=yes
BOOTPROTO=static
IPADDR=192.168.1.10
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
ONBOOT=yes
BONDING_OPTS="mode=4 miimon=100 lacp_rate=fast"
```

## 系统管理脚本

### 常用管理脚本

#### 节点状态检查脚本
```bash
#!/bin/bash
# check_nodes.sh - 检查计算节点状态

nodes="compute-001 compute-002 compute-003 compute-004"

for node in $nodes; do
    echo "=== 检查节点: $node ==="

    # 检查SSH连接
    if ssh -o ConnectTimeout=5 $node "hostname" > /dev/null 2>&1; then
        echo "✓ SSH连接正常"
    else
        echo "✗ SSH连接失败"
        continue
    fi

    # 检查CPU使用率
    cpu_usage=$(ssh $node "top -bn1 | grep 'Cpu(s)' | awk '{print \$2}' | cut -d'%' -f1")
    echo "CPU使用率: ${cpu_usage}%"

    # 检查内存使用率
    mem_usage=$(ssh $node "free | grep Mem | awk '{printf \"%.2f\", \$3/\$2 * 100.0}'")
    echo "内存使用率: ${mem_usage}%"

    # 检查磁盘使用率
    disk_usage=$(ssh $node "df -h / | tail -1 | awk '{print \$5}' | sed 's/%//'")
    echo "磁盘使用率: ${disk_usage}%"

    # 检查服务状态
    services="slurmd sshd"
    for service in $services; do
        status=$(ssh $node "systemctl is-active $service")
        echo "$service状态: $status"
    done

    echo ""
done
```

#### 批量软件安装脚本
```bash
#!/bin/bash
# install_software.sh - 批量安装软件

nodes="compute-001 compute-002 compute-003 compute-004"
packages="openmpi gcc gcc-c++ gcc-gfortran"

for node in $nodes; do
    echo "=== 在 $node 上安装软件 ==="
    ssh $node "yum update -y && yum install -y $packages"
    ssh $node "systemctl enable sshd && systemctl start sshd"
    echo "✓ $node 安装完成"
    echo ""
done
```

#### 作业提交模板
```bash
#!/bin/bash
# job_template.sh - 作业提交模板

#SBATCH --job-name=test_job
#SBATCH --output=output_%j.log
#SBATCH --error=error_%j.log
#SBATCH --partition=compute
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=32
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=user@hpc.edu.cn

# 加载模块
module load openmpi/4.1.4
module load gcc/11.2.0

# 设置环境变量
export OMP_NUM_THREADS=4
export OMP_PROC_BIND=TRUE

# 进入工作目录
cd $SLURM_SUBMIT_DIR

# 运行程序
echo "开始运行作业: $SLURM_JOB_NAME"
echo "作业ID: $SLURM_JOB_ID"
echo "节点列表: $SLURM_JOB_NODELIST"
echo "当前时间: $(date)"

# MPI程序
mpirun -np $SLURM_NTASKS ./my_mpi_program

# OpenMP程序
# ./my_openmp_program

# 混合程序
# mpirun -np $SLURM_NNODES ./my_hybrid_program

echo "作业完成: $(date)"
```

#### 性能监控脚本
```bash
#!/bin/bash
# monitor_performance.sh - 性能监控脚本

duration=3600  # 监控1小时
interval=60    # 每60秒采样一次

echo "开始性能监控，持续时间: ${duration}秒"
echo "采样间隔: ${interval}秒"
echo "时间,CPU使用率(%),内存使用率(%),负载1分钟,负载5分钟,负载15分钟" > performance_log.csv

for ((i=0; i<duration; i+=interval)); do
    timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    cpu_usage=$(top -bn1 | grep 'Cpu(s)' | awk '{print $2}' | cut -d'%' -f1)
    mem_usage=$(free | grep Mem | awk '{printf "%.2f", $3/$2 * 100.0}')
    load_avg=$(uptime | awk -F'load average:' '{print $2}' | sed 's/ //g')

    echo "$timestamp,$cpu_usage,$mem_usage,$load_avg" >> performance_log.csv
    echo "[$timestamp] CPU: ${cpu_usage}%, Memory: ${mem_usage}%, Load: $load_avg"

    sleep $interval
done

echo "性能监控完成，数据保存在 performance_log.csv"
```

## 故障排除

### 常见问题解决

#### SSH连接问题
```bash
# 检查SSH服务状态
systemctl status sshd
systemctl start sshd

# 检查防火墙
firewall-cmd --list-ports
firewall-cmd --add-port=22/tcp --permanent
firewall-cmd --reload

# 检查SSH配置
cat /etc/ssh/sshd_config
systemctl restart sshd

# 测试SSH连接
ssh -v 用户@主机
```

#### 网络连接问题
```bash
# 检查网络接口
ip addr show
ip route show

# 检查DNS配置
cat /etc/resolv.conf
nslookup google.com

# 检查路由
route -n
traceroute 目标主机

# 检查防火墙
iptables -L
firewall-cmd --list-all
```

#### 存储系统问题
```bash
# Lustre问题
lctl get_param mds.* | grep -i error
lctl get_param oss.* | grep -i error
dmesg | grep -i lustre

# GPFS问题
mmgetstate -a
mmlsfs all
df -h

# 本地存储问题
df -h
fsck 设备
mount -o remount /
```

#### 作业调度问题
```bash
# SLURM问题
systemctl status slurmctld
systemctl status slurmd
scontrol show config
tail -f /var/log/slurm/slurmctld.log

# PBS问题
systemctl status pbs_server
systemctl status pbs_mom
qstat -Q
tail -f /var/spool/pbs/server_logs/current

# LSF问题
lsadmin showconf
bhosts
lsid
```

### 日志文件位置

#### 系统日志
```bash
/var/log/messages           # 系统消息日志
/var/log/syslog             # 系统日志（Ubuntu）
/var/log/dmesg              # 内核日志
/var/log/auth.log           # 认证日志（Ubuntu）
/var/log/secure             # 安全日志（CentOS）
/var/log/yum.log            # YUM安装日志
/var/log/anaconda/          # 安装日志
```

#### HPC系统日志
```bash
# SLURM日志
/var/log/slurm/slurmctld.log
/var/log/slurm/slurmd.log
/var/log/slurm/slurmdbd.log

# PBS日志
/var/spool/pbs/server_logs/
/var/spool/pbs/mom_logs/
/var/spool/pbs/sched_logs/

# LSF日志
/opt/lsf/9.1/linux2.6-glibc2.3-x86_64/work/cluster1/logs/

# Lustre日志
/var/log/lustre/
dmesg | grep -i lustre

# MPI日志
/tmp/openmpi-sessions-*
/opt/intel/oneapi/mpi/*/logs/
```

#### 应用日志
```bash
# 用户作业日志
$HOME/slurm-%j.out
$HOME/slurm-%j.err
/var/spool/slurmd/job%d/

# 系统服务日志
/var/log/httpd/
/var/log/mysql/
/var/log/postfix/
/var/log/cron/
```

## 快捷命令集合

### 日常管理命令
```bash
# 快速检查系统状态
alias sysinfo='uname -a && free -h && df -h && uptime'

# 快速查看作业状态
alias qstat='squeue -u $USER'
alias jstat='squeue -u $USER -o "%.18i %.9P %.8j %.8u %.2t %.10M %.6D %R"'

# 快速查看节点状态
alias nstat='sinfo -N -l'

# 快速监控资源使用
alias mon='watch -n 1 "echo \"=== CPU ===\" && top -bn1 | grep \"Cpu(s)\" && echo \"=== Memory ===\" && free -h && echo \"=== Disk ===\" && df -h"'

# 快速文件搜索
alias ffind='find . -name'
alias ggrep='grep -r --include="*.c" --include="*.h"'

# 快速网络测试
alias pingtest='ping -c 4'
alias speedtest='iperf3 -c'
```

### 批量操作命令
```bash
# 批量执行命令
for node in compute-{001..010}; do ssh $node "命令"; done

# 批量文件传输
for node in compute-{001..010}; do scp 文件 $node:/目标路径; done

# 批量检查状态
for node in compute-{001..010}; do echo "=== $node ==="; ssh $node "命令"; done

# 并行执行（使用GNU parallel）
parallel ssh {} "命令" ::: compute-{001..010}
```

这个附录提供了HPC运维工程师日常工作中最常用的命令和脚本，建议打印或保存为快速参考手册使用。