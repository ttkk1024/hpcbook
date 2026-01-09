# 附录B 配置文件模板

## B.1 网络配置模板

### B.1.1 InfiniBand网络配置

#### OpenSM配置文件
```bash
# /etc/opensm/opensm.conf
# OpenSM Subnet Manager配置文件

# 基本配置
daemonize yes
use_syslog yes
priority 10
guid_prefix 0x8888
force_guid_prefix no

# 端口配置
guid_routing yes
sl2sc_file /etc/opensm/sl2sc.conf
sc2sc_file /etc/opensm/sc2sc.conf

# 路由算法
routing_engine minhop
enable_permissive_lids no

# QoS配置
enable_qos yes
qos_policy_file /etc/opensm/qos_policy.conf

# 日志配置
log_file /var/log/opensm/opensm.log
log_max_size 100M
log_level INFO

# 高可用配置
ha_enabled yes
ha_partner opensm-backup
```

#### sl2sc.conf配置文件
```bash
# /etc/opensm/sl2sc.conf
# Service Level to Virtual Lane映射

# SL 0-3: 高优先级流量
0 1
1 1
2 1
3 1

# SL 4-7: 低优先级流量
4 2
5 2
6 2
7 2
```

#### sc2sc.conf配置文件
```bash
# /etc/opensm/sc2sc.conf
# Service Class到Service Class的映射

# 同一Service Class内的映射
0 0
1 1
2 2
3 3
4 4
5 5
6 6
7 7
```

### B.1.2 网络接口配置

#### InfiniBand接口配置
```bash
# /etc/sysconfig/network-scripts/ifcfg-ib0
DEVICE=ib0
TYPE=InfiniBand
ONBOOT=yes
BOOTPROTO=static
IPADDR=10.1.1.10
NETMASK=255.255.255.0
MTU=4096
```

#### VLAN配置
```bash
# /etc/sysconfig/network-scripts/ifcfg-eth0.100
DEVICE=eth0.100
VLAN=yes
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.100.10
NETMASK=255.255.255.0
GATEWAY=192.168.100.1
```

### B.1.3 网络性能调优配置

#### sysctl.conf网络优化
```bash
# /etc/sysctl.conf
# 网络性能优化配置

# TCP优化
net.core.rmem_max = 134217728
net.core.wmem_max = 134217728
net.ipv4.tcp_rmem = 4096 87380 134217728
net.ipv4.tcp_wmem = 4096 65536 134217728
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_sack = 1

# 网络队列优化
net.core.netdev_max_backlog = 5000
net.core.dev_weight = 64
net.core.somaxconn = 1024

# InfiniBand优化
net.core.rmem_default = 262144
net.core.wmem_default = 262144

# 禁用不必要的网络功能
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
```

## B.2 存储系统配置模板

### B.2.1 Lustre文件系统配置

#### MDS配置文件
```bash
# /etc/lustre/mds.conf
# Lustre元数据服务器配置

# 基本配置
mdt_index = 0
mdt_mgs_node = mgs-node@o2ib
mdt_mount_opt = "failnode=mgs-node@o2ib"

# 性能优化
mdt_max_mdt = 1024
mdt_max_ost = 16384
mdt_max_easize = 65536

# 网络配置
mdt_nid = "10.1.1.100@o2ib"
mdt_net = "o2ib"
mdt_net_config = "ib0"

# 日志配置
mdt_log_size = 1024
mdt_log_level = "INFO"
```

#### OSS配置文件
```bash
# /etc/lustre/oss.conf
# Lustre对象存储服务器配置

# 基本配置
ost_index = 0
ost_mgs_node = mgs-node@o2ib
ost_mount_opt = "failnode=mgs-node@o2ib"

# 性能优化
ost_max_ost = 16384
ost_max_easize = 65536
ost_max_raid = 32

# 网络配置
ost_nid = "10.1.1.200@o2ib"
ost_net = "o2ib"
ost_net_config = "ib0"

# 磁盘配置
ost_devices = "/dev/sdb1,/dev/sdc1,/dev/sdd1"
ost_raid_level = 5
ost_stripe_size = 1048576
```

#### 客户端配置文件
```bash
# /etc/lustre/client.conf
# Lustre客户端配置

# 基本配置
mdt_mgs_node = mgs-node@o2ib
mount_point = /mnt/lustre
mount_options = "user_xattr,noatime"

# 性能优化
max_read_ahead_mb = 16
max_read_ahead_whole_mb = 32
read_ahead_async = 1

# 网络配置
network_config = "ib0"
network_timeout = 300
```

### B.2.2 RAID配置模板

#### mdadm配置文件
```bash
# /etc/mdadm.conf
# RAID配置文件

# 配置选项
DEVICE /dev/sd[bcdefghijk]1
MAILADDR admin@hpc.edu.cn
PROGRAM /usr/sbin/mdadm --monitor --scan --mail= --program=

# RAID阵列定义
ARRAY /dev/md0 level=raid10 num-devices=8 UUID=12345678:abcd1234:5678abcd:12345678
ARRAY /dev/md1 level=raid6 num-devices=12 UUID=87654321:dcba4321:dcba8765:87654321
```

#### RAID监控脚本
```bash
#!/bin/bash
# /usr/local/bin/raid_monitor.sh
# RAID状态监控脚本

raid_status_check() {
    echo "=== RAID状态检查 ===" > /tmp/raid_status.log
    date >> /tmp/raid_status.log

    # 检查RAID阵列状态
    for raid in /dev/md*; do
        if [ -b "$raid" ]; then
            status=$(cat /proc/mdstat | grep $(basename $raid) | awk '{print $4}')
            echo "$(basename $raid): $status" >> /tmp/raid_status.log

            if [ "$status" != "active" ]; then
                echo "警告：RAID $(basename $raid) 状态异常：$status"
                send_alert "raid_degraded" "$(basename $raid)"
            fi
        fi
    done

    # 检查磁盘状态
    echo "=== 磁盘状态检查 ===" >> /tmp/raid_status.log
    for disk in /dev/sd*; do
        if [ -b "$disk" ]; then
            health=$(smartctl -H $disk 2>/dev/null | grep "SMART overall-health" | awk '{print $6}')
            if [ "$health" != "PASSED" ]; then
                echo "警告：磁盘 $disk 健康状态异常"
                send_alert "disk_failure" "$disk"
            fi
        fi
    done
}

# 发送告警
send_alert() {
    local type=$1
    local device=$2
    local message="RAID故障：$type on $device"

    # 发送邮件
    echo "$message" | mail -s "RAID故障告警" admin@hpc.edu.cn

    # 记录日志
    logger -p local0.error "$message"
}

# 定时执行
raid_status_check
```

### B.2.3 NFS配置模板

#### exports配置文件
```bash
# /etc/exports
# NFS共享配置

# 用户主目录共享
/home 10.1.0.0/16(rw,sync,no_root_squash,no_subtree_check)
/projects 10.1.0.0/16(rw,sync,no_root_squash,no_subtree_check)

# 临时文件共享
/tmp 10.1.0.0/16(rw,sync,no_root_squash,no_subtree_check)

# 只读共享
/opt/software 10.1.0.0/16(ro,sync,no_subtree_check)
```

#### NFS服务配置
```bash
# /etc/nfs.conf
# NFS服务配置

[nfsd]
# NFS服务器配置
threads=16
port=2049
vers2=y
vers3=y
vers4=y
vers4.1=y
vers4.2=y

[mountd]
# 挂载守护进程配置
port=20048
threads=8

[statd]
# 状态守护进程配置
port=32765
outgoing-port=32766

[idmapd]
# ID映射守护进程配置
domain=hpc.edu.cn
```

## B.3 作业调度系统配置模板

### B.3.1 SLURM配置模板

#### slurm.conf配置文件
```bash
# /etc/slurm/slurm.conf
# SLURM作业调度系统配置

# 基本配置
ClusterName=hpc-cluster
ControlMachine=slurm-master
ControlAddr=10.1.1.1
BackupController=slurm-backup
BackupAddr=10.1.1.2

# 节点配置
NodeName=compute-[001-100] Sockets=2 CoresPerSocket=16 ThreadsPerCore=1 RealMemory=262144 State=UNKNOWN
NodeName=gpu-[001-20] Sockets=2 CoresPerSocket=16 ThreadsPerCore=1 Gres=gpu:4 RealMemory=524288 State=UNKNOWN
NodeName=login-[001-02] Sockets=2 CoresPerSocket=16 ThreadsPerCore=1 RealMemory=262144 State=UNKNOWN

# 分区配置
PartitionName=compute Nodes=compute-[001-100] Default=YES MaxTime=72:00:00 State=UP
PartitionName=gpu Nodes=gpu-[001-20] Default=NO MaxTime=48:00:00 State=UP
PartitionName=debug Nodes=compute-[001-05] Default=NO MaxTime=02:00:00 State=UP

# 调度器配置
SchedulerType=sched/backfill
SelectType=select/cons_tres
SelectTypeParameters=CR_Core_Memory
SchedulerParameters=pack,conflict_check

# 作业配置
MaxArraySize=10000
DefMemPerCPU=4000
MaxMemPerCPU=8000
JobCompType=jobcomp/filetxt
JobCompLoc=/var/log/slurm/job_completions.log

# 安全配置
SlurmUser=slurm
SlurmctldPort=6817
SlurmdPort=6818
AuthType=auth/munge
```

#### gres.conf配置文件
```bash
# /etc/slurm/gres.conf
# GPU资源配置

# GPU资源定义
NodeName=gpu-[001-20] Name=gpu Type=tesla-v100 File=/dev/nvidia[0-3]
NodeName=gpu-[021-040] Name=gpu Type=a100 File=/dev/nvidia[0-7]
```

#### cgroup.conf配置文件
```bash
# /etc/slurm/cgroup.conf
# Cgroup资源控制配置

CgroupPlugin=cgroup/v2
ConstrainCores=yes
ConstrainRAMSpace=yes
ConstrainSwapSpace=yes
ConstrainDevices=yes

# CPU控制
CPUFreqGovernor=performance
CPUFreqMin=2500000
CPUFreqMax=3500000

# 内存控制
MemorySwappiness=0
MemoryLimitRAM=yes
MemoryLimitSwap=no

# 设备控制
Devices=allow
```

### B.3.2 PBS Pro配置模板

#### server_priv/acl_hosts
```bash
# /var/spool/pbs/server_priv/acl_hosts
# PBS服务器访问控制

# 允许连接的主机
slurm-master
compute-[001-100]
gpu-[001-20]
login-[001-02]
```

#### server_priv/config
```bash
# /var/spool/pbs/server_priv/config
# PBS服务器配置

# 基本配置
$pbsserver slurm-master
$usecp *:/home /home
$logevent 2047
$loglevel 0

# 调度器配置
$scheduler_iteration 600
$sched_port 15004
$server_privhooks /var/spool/pbs/server_priv/hooks

# 作业配置
$job_start_timeout 300
$job_prio_style fifo
$node_check_rate 150
```

#### sched_priv/sched_config
```bash
# /var/spool/pbs/sched_priv/sched_config
# PBS调度器配置

# 调度策略
load_balancing     true     all    all
preempt_prio       100
preempt_order      susp
preempt_targets    ALL
preempt_sort       time

# 资源限制
default_queue      batch
resources_default.nodes = 1
resources_default.walltime = 01:00:00
```

### B.3.3 LSF配置模板

#### lsf.conf配置文件
```bash
# /opt/lsf/conf/lsf.conf
# LSF系统配置

# LSF环境变量
LSF_SERVER_HOSTS="lsf-master lsf-backup"
LSF_LIM_HOSTS="lsf-master lsf-backup"
LSF_RES_HOSTS="lsf-master lsf-backup"
LSF_ENTITIES="master backup"

# 集群配置
LSF_CLUSTER_NAME=hpc-cluster
LSF_HOSTS="compute-[001-100] gpu-[001-20] login-[001-02]"

# 端口配置
LSF_LIM_PORT=6801
LSF_RES_PORT=6802
LSF_SBD_PORT=6803

# 日志配置
LSF_LOGDIR=/var/log/lsf
LSF_LOG_MASK=LOG_INFO
```

#### lsb.hosts配置文件
```bash
# /opt/lsf/conf/lsb.hosts
# LSF主机配置

# 主机资源定义
begin host
HOST_NAME   model      type    mpw   arch    admin     swp   r1m   ut
compute-001 Xeon       Linux   32    intel   admin     100   0.1   0.1
compute-002 Xeon       Linux   32    intel   admin     100   0.1   0.1
gpu-001     Xeon+GPU   Linux   32    intel   admin     100   0.1   0.1
login-001   Xeon       Linux   32    intel   admin     100   0.1   0.1
end host
```

#### lsb.queues配置文件
```bash
# /opt/lsf/conf/lsb.queues
# LSF队列配置

# 队列定义
Begin Queue
QUEUE_NAME = compute
PRIORITY = 30
NICE = 10
PENALTY = 10
DESCRIPTION = General compute queue
HOSTS = compute-[001-100]
MAX_RUNNING = 1000
NICE = 10
JOB_ACCEPT_INTERVAL = 30
JOB_START_INTERVAL = 30
RESERVE = Y
END Queue

Begin Queue
QUEUE_NAME = gpu
PRIORITY = 50
PENALTY = 20
DESCRIPTION = GPU compute queue
HOSTS = gpu-[001-20]
MAX_RUNNING = 200
NICE = 5
RESERVE = Y
END Queue
```

## B.4 监控系统配置模板

### B.4.1 Nagios配置模板

#### nagios.cfg配置文件
```bash
# /etc/nagios/nagios.cfg
# Nagios主配置文件

# 基本配置
log_file=/var/log/nagios/nagios.log
cfg_file=/etc/nagios/objects/commands.cfg
cfg_file=/etc/nagios/objects/contacts.cfg
cfg_file=/etc/nagios/objects/timeperiods.cfg
cfg_file=/etc/nagios/objects/templates.cfg

# 主机和服务配置
cfg_dir=/etc/nagios/objects/hosts
cfg_dir=/etc/nagios/objects/services

# 性能配置
use_timezone=Asia/Shanghai
status_file=/var/log/nagios/status.dat
object_cache_file=/var/log/nagios/objects.cache
precached_object_file=/var/log/nagios/objects.precache

# 日志配置
log_rotation_method=d
log_archive_path=/var/log/nagios/archives
```

#### hosts.cfg配置文件
```bash
# /etc/nagios/objects/hosts/compute.cfg
# 计算节点监控配置

define host{
    use             generic-host
    host_name       compute-001
    alias           Compute Node 001
    address         10.1.1.101
    check_command   check-host-alive
    max_check_attempts 5
    notification_interval 30
    notification_period 24x7
    contact_groups  admins
}

define hostgroup{
    hostgroup_name  compute-nodes
    alias           Compute Nodes
    members         compute-001,compute-002,compute-003
}
```

#### services.cfg配置文件
```bash
# /etc/nagios/objects/services/compute-services.cfg
# 计算节点服务监控配置

define service{
    use                 generic-service
    hostgroup_name      compute-nodes
    service_description CPU Usage
    check_command       check_nrpe!check_cpu
    max_check_attempts 4
    normal_check_interval 5
    retry_check_interval 1
    notification_interval 30
    notification_period 24x7
}

define service{
    use                 generic-service
    hostgroup_name      compute-nodes
    service_description Memory Usage
    check_command       check_nrpe!check_memory
    max_check_attempts 4
    normal_check_interval 5
    retry_check_interval 1
    notification_interval 30
    notification_period 24x7
}

define service{
    use                 generic-service
    hostgroup_name      compute-nodes
    service_description Disk Usage
    check_command       check_nrpe!check_disk
    max_check_attempts 4
    normal_check_interval 5
    retry_check_interval 1
    notification_interval 30
    notification_period 24x7
}
```

### B.4.2 Zabbix配置模板

#### zabbix_server.conf配置文件
```bash
# /etc/zabbix/zabbix_server.conf
# Zabbix服务器配置

# 数据库配置
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix_password

# 服务器配置
ListenPort=10051
SourceIP=
LogType=file
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=0
PidFile=/var/run/zabbix/zabbix_server.pid

# 性能配置
StartPollers=50
StartPollersUnreachable=10
StartTrappers=20
StartPingers=10
StartDiscoverers=5
StartHTTPPollers=5
StartTimers=10
StartEscalators=10
```

#### zabbix_agentd.conf配置文件
```bash
# /etc/zabbix/zabbix_agentd.conf
# Zabbix客户端配置

# 基本配置
PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=10.1.1.10
ServerActive=10.1.1.10
Hostname=compute-001
HostMetadata=compute-node

# 性能配置
StartAgents=3
StartPollers=5
StartPollersUnreachable=1
StartTrappers=5
StartPingers=1
StartDiscoverers=1

# 自定义监控
UserParameter=cpu.temperature,/usr/local/bin/check_cpu_temp.sh
UserParameter=memory.usage,/usr/local/bin/check_memory_usage.sh
UserParameter=disk.io,/usr/local/bin/check_disk_io.sh
```

### B.4.3 Prometheus配置模板

#### prometheus.yml配置文件
```yaml
# /etc/prometheus/prometheus.yml
# Prometheus配置文件

global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  # Prometheus自身监控
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # 节点监控
  - job_name: 'node-exporter'
    static_configs:
      - targets:
        - 'compute-001:9100'
        - 'compute-002:9100'
        - 'compute-003:9100'

  # GPU监控
  - job_name: 'nvidia-dcgm'
    static_configs:
      - targets:
        - 'compute-001:9400'
        - 'compute-002:9400'

  # 存储监控
  - job_name: 'lustre'
    static_configs:
      - targets:
        - 'mds-001:9101'
        - 'oss-001:9102'
```

#### alert_rules.yml配置文件
```yaml
# /etc/prometheus/alert_rules.yml
# Prometheus告警规则

groups:
  - name: hpc_cluster
    rules:
      # CPU使用率告警
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage detected"
          description: "CPU usage is above 80% for 5 minutes on {{ $labels.instance }}"

      # 内存使用率告警
      - alert: HighMemoryUsage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High memory usage detected"
          description: "Memory usage is above 90% on {{ $labels.instance }}"

      # 磁盘使用率告警
      - alert: HighDiskUsage
        expr: (1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100 > 90
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High disk usage detected"
          description: "Disk usage is above 90% on {{ $labels.instance }}"
```

## B.5 编译器和工具链配置模板

### B.5.1 GCC编译器配置

#### 编译器优化脚本
```bash
#!/bin/bash
# /usr/local/bin/compile_optimized.sh
# 编译器优化脚本

# 设置编译器选项
export CFLAGS="-O3 -march=native -mtune=native -fopenmp"
export CXXFLAGS="-O3 -march=native -mtune=native -fopenmp"
export FFLAGS="-O3 -march=native -mtune=native -fopenmp"
export FCFLAGS="-O3 -march=native -mtune=native -fopenmp"

# 设置链接器选项
export LDFLAGS="-Wl,-rpath,/usr/local/lib64:/usr/local/lib"

# 设置数学库
export LD_LIBRARY_PATH="/usr/local/lib64:/usr/local/lib:$LD_LIBRARY_PATH"

# 编译函数
compile_with_optimization() {
    local source_file=$1
    local output_file=$2

    echo "编译 $source_file 到 $output_file"
    gcc $CFLAGS $LDFLAGS -o $output_file $source_file
    if [ $? -eq 0 ]; then
        echo "编译成功"
    else
        echo "编译失败"
        return 1
    fi
}

# 使用示例
if [ $# -eq 2 ]; then
    compile_with_optimization $1 $2
else
    echo "用法: $0 <源文件> <输出文件>"
fi
```

#### Intel编译器配置
```bash
# /etc/profile.d/intel_compiler.sh
# Intel编译器环境配置

# Intel编译器路径
export INTEL_HOME=/opt/intel/oneapi
export PATH=$INTEL_HOME/compiler/latest/linux/bin:$PATH
export LD_LIBRARY_PATH=$INTEL_HOME/compiler/latest/linux/lib:$LD_LIBRARY_PATH

# Intel MKL配置
export MKLROOT=$INTEL_HOME/mkl/latest
export LD_LIBRARY_PATH=$MKLROOT/lib/intel64:$LD_LIBRARY_PATH

# Intel MPI配置
export I_MPI_ROOT=$INTEL_HOME/mpi/latest
export PATH=$I_MPI_ROOT/bin:$PATH
export LD_LIBRARY_PATH=$I_MPI_ROOT/lib:$LD_LIBRARY_PATH

# Intel VTune配置
export VTUNE_PROFILER_2021_DIR=$INTEL_HOME/vtune/latest
export PATH=$VTUNE_PROFILER_2021_DIR/bin64:$PATH
```

### B.5.2 数学库配置模板

#### BLAS/LAPACK配置
```bash
# /etc/profile.d/math_libs.sh
# 数学库环境配置

# OpenBLAS配置
export OPENBLAS_NUM_THREADS=32
export OPENBLAS_VERBOSE=0
export LD_LIBRARY_PATH="/usr/lib/openblas-base:$LD_LIBRARY_PATH"

# MKL配置
export MKL_NUM_THREADS=32
export MKL_DYNAMIC=TRUE
export MKL_INTERFACE_LAYER=GNU,ILP64
export MKL_THREADING_LAYER=GNU

# 链接选项
export BLAS_LIBS="-lmkl_gf_ilp64 -lmkl_gnu_thread -lmkl_core -lgomp -lpthread -lm -ldl"
export LAPACK_LIBS="-lmkl_gf_ilp64 -lmkl_gnu_thread -lmkl_core -lgomp -lpthread -lm -ldl"
```

#### FFTW配置
```bash
# /etc/profile.d/fftw.sh
# FFTW库环境配置

export FFTW_HOME=/usr/local/fftw
export PATH=$FFTW_HOME/bin:$PATH
export LD_LIBRARY_PATH=$FFTW_HOME/lib:$LD_LIBRARY_PATH
export PKG_CONFIG_PATH=$FFTW_HOME/lib/pkgconfig:$PKG_CONFIG_PATH

# FFTW优化选项
export FFTW_PLAN_OPTIONS="FFTW_MEASURE FFTW_UNALIGNED"
```

## B.6 安全配置模板

### B.6.1 SSH配置模板

#### sshd_config配置文件
```bash
# /etc/ssh/sshd_config
# SSH服务安全配置

# 基本配置
Port 2222
Protocol 2
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

# 认证配置
LoginGraceTime 60
PermitRootLogin no
StrictModes yes
MaxAuthTries 3
MaxSessions 10

# 密钥认证
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
AuthorizedKeysCommand none
AuthorizedKeysCommandUser nobody

# 安全选项
PermitEmptyPasswords no
PasswordAuthentication yes
ChallengeResponseAuthentication no
UsePAM yes

# 网络安全
X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
GatewayPorts no

# 日志配置
SyslogFacility AUTHPRIV
LogLevel INFO

# 用户限制
AllowUsers compute-user@10.1.0.0/16
DenyUsers root
DenyGroups root
```

#### SSH密钥管理脚本
```bash
#!/bin/bash
# /usr/local/bin/manage_ssh_keys.sh
# SSH密钥管理脚本

# 密钥生成
generate_ssh_key() {
    local user=$1
    local key_type=${2:-rsa}
    local key_bits=${3:-4096}

    if [ -z "$user" ]; then
        echo "用法: generate_ssh_key <用户名> [密钥类型] [密钥长度]"
        return 1
    fi

    local home_dir=$(getent passwd $user | cut -d: -f6)
    local ssh_dir="$home_dir/.ssh"

    # 创建.ssh目录
    mkdir -p $ssh_dir
    chmod 700 $ssh_dir

    # 生成密钥
    ssh-keygen -t $key_type -b $key_bits -f $ssh_dir/id_$key_type -N "" -C "$user@hpc.edu.cn"

    # 设置权限
    chmod 600 $ssh_dir/id_$key_type
    chmod 644 $ssh_dir/id_$key_type.pub

    echo "SSH密钥已生成: $ssh_dir/id_$key_type"
}

# 密钥分发
distribute_ssh_key() {
    local source_key=$1
    shift
    local hosts=("$@")

    if [ -z "$source_key" ]; then
        echo "用法: distribute_ssh_key <源密钥> <主机列表...>"
        return 1
    fi

    for host in "${hosts[@]}"; do
        echo "分发密钥到 $host"
        ssh-copy-id -i $source_key.pub $host
    done
}
```

### B.6.2 防火墙配置模板

#### iptables配置脚本
```bash
#!/bin/bash
# /usr/local/bin/setup_firewall.sh
# 防火墙配置脚本

# 清除现有规则
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X

# 设置默认策略
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# 允许本地回环
iptables -A INPUT -i lo -j ACCEPT

# 允许已建立的连接
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# SSH访问（端口2222）
iptables -A INPUT -p tcp --dport 2222 -m state --state NEW -m limit --limit 3/minute --limit-burst 3 -j ACCEPT

# InfiniBand网络
iptables -A INPUT -p tcp --dport 4791 -j ACCEPT
iptables -A INPUT -p udp --dport 4791 -j ACCEPT

# SLURM端口
iptables -A INPUT -p tcp --dport 6817 -j ACCEPT
iptables -A INPUT -p tcp --dport 6818 -j ACCEPT

# NFS端口
iptables -A INPUT -p tcp --dport 111 -j ACCEPT
iptables -A INPUT -p udp --dport 111 -j ACCEPT
iptables -A INPUT -p tcp --dport 2049 -j ACCEPT

# 保存规则
iptables-save > /etc/iptables/rules.v4
```

#### SELinux配置
```bash
# /etc/selinux/config
# SELinux配置文件

# SELinux状态
SELINUX=enforcing
SELINUXTYPE=targeted

# 自定义策略
# 允许NFS挂载
setsebool -P nfs_export_all_ro on
setsebool -P nfs_export_all_rw on

# 允许SLURM服务
semanage port -a -t slurm_port_t -p tcp 6817
semanage port -a -t slurm_port_t -p tcp 6818

# 允许SSH服务
semanage port -a -t ssh_port_t -p tcp 2222
```

## B.7 备份和恢复配置模板

### B.7.1 rsync备份配置

#### 备份脚本
```bash
#!/bin/bash
# /usr/local/bin/cluster_backup.sh
# 集群备份脚本

# 配置变量
BACKUP_DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/daily"
SOURCE_DIRS=("/home" "/opt/software" "/etc/slurm" "/etc/lustre")
EXCLUDE_FILE="/etc/backup_exclude.txt"
LOG_FILE="/var/log/backup_$BACKUP_DATE.log"

# 创建备份目录
mkdir -p $BACKUP_DIR/$BACKUP_DATE

# 执行备份
backup_directories() {
    for dir in "${SOURCE_DIRS[@]}"; do
        if [ -d "$dir" ]; then
            echo "开始备份 $dir" | tee -a $LOG_FILE
            rsync -avz --delete --exclude-from=$EXCLUDE_FILE \
                $dir $BACKUP_DIR/$BACKUP_DATE/ 2>&1 | tee -a $LOG_FILE

            if [ $? -eq 0 ]; then
                echo "备份 $dir 成功" | tee -a $LOG_FILE
            else
                echo "备份 $dir 失败" | tee -a $LOG_FILE
                return 1
            fi
        fi
    done
}

# 压缩备份
compress_backup() {
    echo "开始压缩备份" | tee -a $LOG_FILE
    tar -czf $BACKUP_DIR/backup_$BACKUP_DATE.tar.gz \
        -C $BACKUP_DIR $BACKUP_DATE 2>&1 | tee -a $LOG_FILE

    if [ $? -eq 0 ]; then
        echo "压缩备份成功" | tee -a $LOG_FILE
        rm -rf $BACKUP_DIR/$BACKUP_DATE
    else
        echo "压缩备份失败" | tee -a $LOG_FILE
        return 1
    fi
}

# 主执行流程
main() {
    echo "=== 开始备份 $BACKUP_DATE ===" | tee $LOG_FILE
    backup_directories
    compress_backup
    echo "=== 备份完成 ===" | tee -a $LOG_FILE
}

# 执行备份
main
```

#### 排除文件配置
```bash
# /etc/backup_exclude.txt
# 备份排除文件列表

# 临时文件
*.tmp
*.log
*.cache
/tmp/*
/var/tmp/*

# 大型数据文件
*.raw
*.dat
*.h5
*.nc

# 系统文件
/proc/*
/sys/*
/dev/*
/boot/*
/mnt/*
/media/*

# 网络挂载
/backup/*
/mnt/lustre/*
/nfs/*
```

### B.7.2 Bacula备份配置

#### bacula-dir.conf配置文件
```bash
# /etc/bacula/bacula-dir.conf
# Bacula Director配置

Director {
    Name = hpc-dir
    DIRPort = 9101
    QueryFile = "/etc/bacula/query.sql"
    WorkingDirectory = "/var/spool/bacula"
    PidDirectory = "/var/run/bacula"
    Maximum Concurrent Jobs = 20
    Password = "director_password"
    Messages = Standard
}

# 客户端定义
Client {
    Name = compute-001-fd
    Address = compute-001
    FDPort = 9102
    Catalog = MyCatalog
    Password = "client_password"
    File Retention = 30 days
    Job Retention = 6 months
    AutoPrune = yes
}

# 存储定义
Storage {
    Name = File
    Address = bacula-sd
    SDPort = 9103
    Password = "storage_password"
    Device = FileStorage
    Media Type = File
}

# 文件集定义
FileSet {
    Name = "Full Set"
    Include {
        Options {
            signature = MD5
            compression = GZIP
        }
        File = /home
        File = /opt/software
        File = /etc/slurm
        File = /etc/lustre
    }
    Exclude {
        File = /etc/backup_exclude.txt
    }
}

# 作业定义
Job {
    Name = "DailyBackup"
    Type = Backup
    Client = compute-001-fd
    FileSet = "Full Set"
    Schedule = "DailyCycle"
    Storage = File
    Messages = Standard
    Pool = DailyPool
    Priority = 10
    Write Bootstrap = "/var/spool/bacula/%c.bsr"
}
```

#### bacula-sd.conf配置文件
```bash
# /etc/bacula/bacula-sd.conf
# Bacula Storage Daemon配置

Storage {
    Name = hpc-sd
    SDPort = 9103
    WorkingDirectory = "/var/spool/bacula"
    Pid Directory = "/var/run/bacula"
    Maximum Concurrent Jobs = 20
}

# 设备定义
Device {
    Name = FileStorage
    Media Type = File
    Archive Device = /backup/bacula
    LabelMedia = yes;
    Random Access = Yes;
    AutomaticMount = yes;
    RemovableMedia = no;
    AlwaysOpen = no;
}

# 消息定义
Messages {
    Name = Standard
    director = hpc-dir = all
}
```

## B.8 用户管理配置模板

### B.8.1 LDAP集成配置

#### nslcd.conf配置文件
```bash
# /etc/nslcd.conf
# LDAP名称服务配置

# 基本配置
uid nslcd
gid nslcd
uri ldap://ldap.hpc.edu.cn/
base dc=hpc,dc=edu,dc=cn

# SSL配置
ssl start_tls
tls_reqcert demand
tls_cacertfile /etc/ssl/certs/ca-certificates.crt

# 用户和组映射
map passwd uid uidNumber
map passwd gidNumber gidNumber
map passwd homeDirectory homeDirectory
map passwd loginShell loginShell

map group memberUid memberUid

# 过滤器
filter passwd (objectClass=posixAccount)
filter group (objectClass=posixGroup)

# 缓存配置
bind_timelimit 30
idle_timelimit 3600
```

#### nsswitch.conf配置文件
```bash
# /etc/nsswitch.conf
# 名称服务切换配置

passwd: files ldap
shadow: files ldap
group: files ldap

hosts: files dns
networks: files

protocols: db files
services: db files
ethers: db files
rpc: db files

netgroup: nis
```

### B.8.2 用户环境配置

#### 全局环境变量
```bash
# /etc/profile.d/hpc_env.sh
# HPC环境变量配置

# 路径配置
export PATH="/opt/software/bin:$PATH"
export LD_LIBRARY_PATH="/opt/software/lib:$LD_LIBRARY_PATH"
export MANPATH="/opt/software/share/man:$MANPATH"

# 编译器配置
export CC=gcc
export CXX=g++
export FC=gfortran

# MPI配置
export PATH="/opt/openmpi/bin:$PATH"
export LD_LIBRARY_PATH="/opt/openmpi/lib:$LD_LIBRARY_PATH"

# 作业调度配置
export PATH="/opt/slurm/bin:$PATH"
export PATH="/opt/slurm/sbin:$PATH"

# 软件模块配置
export MODULEPATH="/opt/modules:$MODULEPATH"
```

#### 用户登录脚本
```bash
#!/bin/bash
# /etc/profile.d/user_setup.sh
# 用户环境初始化脚本

# 创建用户目录
setup_user_directories() {
    local home_dir=$HOME
    local user=$USER

    # 创建标准目录
    mkdir -p $home_dir/{work,scratch,software,logs}

    # 设置权限
    chmod 755 $home_dir/work
    chmod 755 $home_dir/scratch
    chmod 755 $home_dir/software
    chmod 755 $home_dir/logs

    # 创建软链接
    ln -sf /mnt/lustre/users/$user $home_dir/lustre
    ln -sf /tmp/$user $home_dir/tmp
}

# 设置别名
setup_aliases() {
    alias ll='ls -alF'
    alias la='ls -A'
    alias l='ls -CF'
    alias grep='grep --color=auto'
    alias df='df -h'
    alias du='du -h'
}

# 加载模块
setup_modules() {
    if [ -f /etc/profile.d/modules.sh ]; then
        source /etc/profile.d/modules.sh
        module load default-environment
    fi
}

# 主函数
main() {
    setup_user_directories
    setup_aliases
    setup_modules
}

# 执行设置
main
```

## B.9 应急响应配置模板

### B.9.1 故障检测脚本

#### 系统健康检查脚本
```bash
#!/bin/bash
# /usr/local/bin/system_health_check.sh
# 系统健康检查脚本

# 配置
LOG_FILE="/var/log/system_health.log"
ALERT_EMAIL="admin@hpc.edu.cn"
CRITICAL_THRESHOLD=90
WARNING_THRESHOLD=80

# 检查CPU使用率
check_cpu_usage() {
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    local cpu_int=${cpu_usage%.*}

    echo "$(date): CPU使用率: $cpu_usage%" >> $LOG_FILE

    if [ $cpu_int -gt $CRITICAL_THRESHOLD ]; then
        send_alert "CRITICAL" "CPU使用率过高: $cpu_usage%"
        return 2
    elif [ $cpu_int -gt $WARNING_THRESHOLD ]; then
        send_alert "WARNING" "CPU使用率偏高: $cpu_usage%"
        return 1
    fi

    return 0
}

# 检查内存使用率
check_memory_usage() {
    local memory_info=$(free | grep Mem)
    local total=$(echo $memory_info | awk '{print $2}')
    local used=$(echo $memory_info | awk '{print $3}')
    local memory_usage=$((used * 100 / total))

    echo "$(date): 内存使用率: $memory_usage%" >> $LOG_FILE

    if [ $memory_usage -gt $CRITICAL_THRESHOLD ]; then
        send_alert "CRITICAL" "内存使用率过高: $memory_usage%"
        return 2
    elif [ $memory_usage -gt $WARNING_THRESHOLD ]; then
        send_alert "WARNING" "内存使用率偏高: $memory_usage%"
        return 1
    fi

    return 0
}

# 检查磁盘使用率
check_disk_usage() {
    local disk_usage=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')

    echo "$(date): 根分区使用率: $disk_usage%" >> $LOG_FILE

    if [ $disk_usage -gt $CRITICAL_THRESHOLD ]; then
        send_alert "CRITICAL" "磁盘使用率过高: $disk_usage%"
        return 2
    elif [ $disk_usage -gt $WARNING_THRESHOLD ]; then
        send_alert "WARNING" "磁盘使用率偏高: $disk_usage%"
        return 1
    fi

    return 0
}

# 检查网络连接
check_network_connectivity() {
    if ! ping -c 1 google.com > /dev/null 2>&1; then
        send_alert "CRITICAL" "网络连接异常"
        return 2
    fi

    return 0
}

# 发送告警
send_alert() {
    local level=$1
    local message=$2
    local full_message="$level: $message ($(hostname))"

    echo "$full_message" >> $LOG_FILE
    echo "$full_message" | mail -s "系统告警: $level" $ALERT_EMAIL
    logger -p local0.error "$full_message"
}

# 主函数
main() {
    echo "=== 系统健康检查开始 $(date) ===" >> $LOG_FILE

    local cpu_status=0
    local memory_status=0
    local disk_status=0
    local network_status=0

    check_cpu_usage
    cpu_status=$?

    check_memory_usage
    memory_status=$?

    check_disk_usage
    disk_status=$?

    check_network_connectivity
    network_status=$?

    local max_status=$((cpu_status > memory_status ? cpu_status : memory_status))
    max_status=$((max_status > disk_status ? max_status : disk_status))
    max_status=$((max_status > network_status ? max_status : network_status))

    if [ $max_status -eq 0 ]; then
        echo "$(date): 系统状态正常" >> $LOG_FILE
    fi

    echo "=== 系统健康检查结束 $(date) ===" >> $LOG_FILE
    return $max_status
}

# 执行检查
main
```

### B.9.2 自动故障恢复脚本

#### 节点故障恢复脚本
```bash
#!/bin/bash
# /usr/local/bin/auto_recovery.sh
# 自动故障恢复脚本

# 配置
SLURM_CONTROLLER="slurm-master"
RECOVERY_LOG="/var/log/auto_recovery.log"
MAX_RECOVERY_ATTEMPTS=3

# 检查节点状态
check_node_status() {
    local node=$1
    local status=$(scontrol show node $node | grep "State=" | cut -d'=' -f2)

    case $status in
        "DOWN"|"FAIL"|"DRAIN")
            return 1
            ;;
        "IDLE"|"ALLOCATED"|"MIXED")
            return 0
            ;;
        *)
            return 2
            ;;
    esac
}

# 重启节点
reboot_node() {
    local node=$1
    local attempt=$2

    echo "$(date): 尝试重启节点 $node (第$attempt次)" >> $RECOVERY_LOG

    # 使用IPMI重启
    ipmitool -H $node -I lanplus -U admin -P password power cycle

    # 等待节点启动
    sleep 120

    # 检查节点状态
    if check_node_status $node; then
        echo "$(date): 节点 $node 重启成功" >> $RECOVERY_LOG
        scontrol update NodeName=$node State=RESUME
        return 0
    else
        echo "$(date): 节点 $node 重启失败" >> $RECOVERY_LOG
        return 1
    fi
}

# 处理故障节点
handle_failed_node() {
    local node=$1

    echo "$(date): 检测到故障节点: $node" >> $RECOVERY_LOG

    # 从调度系统中移除
    scontrol update NodeName=$node State=DOWN Reason="Automatic recovery"

    # 尝试重启
    for attempt in {1..3}; do
        if reboot_node $node $attempt; then
            echo "$(date): 节点 $node 恢复成功" >> $RECOVERY_LOG
            return 0
        fi

        if [ $attempt -lt 3 ]; then
            echo "$(date): 等待300秒后再次尝试" >> $RECOVERY_LOG
            sleep 300
        fi
    done

    # 重启失败，标记为维护状态
    echo "$(date): 节点 $node 重启失败，标记为维护状态" >> $RECOVERY_LOG
    scontrol update NodeName=$node State=DOWN Reason="Hardware failure - manual intervention required"

    # 发送告警邮件
    echo "节点 $node 重启失败，需要人工干预" | mail -s "硬件故障告警" admin@hpc.edu.cn

    return 1
}

# 主函数
main() {
    echo "=== 自动故障恢复开始 $(date) ===" >> $RECOVERY_LOG

    # 检查所有节点
    scontrol show nodes | grep "NodeName=" | while read line; do
        node=$(echo $line | cut -d'=' -f2 | cut -d' ' -f1)

        if ! check_node_status $node; then
            handle_failed_node $node
        fi
    done

    echo "=== 自动故障恢复结束 $(date) ===" >> $RECOVERY_LOG
}

# 定时执行
if [ "$1" = "--daemon" ]; then
    while true; do
        main
        sleep 300  # 每5分钟检查一次
    done
else
    main
fi
```

## 本章小结

本章提供了HPC运维中常用的配置文件模板，涵盖了：

1. **网络配置**：InfiniBand、以太网、VLAN等网络设备的配置
2. **存储系统**：Lustre、RAID、NFS等存储系统的配置
3. **作业调度**：SLURM、PBS Pro、LSF等调度系统的配置
4. **监控系统**：Nagios、Zabbix、Prometheus等监控工具的配置
5. **编译器和工具链**：GCC、Intel编译器、数学库等的配置
6. **安全配置**：SSH、防火墙、SELinux等安全设置
7. **备份和恢复**：rsync、Bacula等备份方案的配置
8. **用户管理**：LDAP集成、用户环境等配置
9. **应急响应**：故障检测和自动恢复的配置

这些配置文件模板为HPC运维工程师提供了标准化的配置参考，可以大大简化系统部署和管理的工作量。在实际使用时，需要根据具体的硬件环境、网络拓扑和业务需求进行相应的调整。