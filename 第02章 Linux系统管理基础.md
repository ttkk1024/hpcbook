# 第2章 Linux系统管理基础

## 2.1 Linux系统安装与配置

### 系统选择与规划

#### 发行版选择
对于HPC环境，推荐使用以下Linux发行版：

**企业级选择：**
- **CentOS Stream 9 / RHEL 9**：企业级标准
- **Ubuntu 22.04/24.04 LTS**：AI 和深度学习领域常用
- **openEuler**：国内 HPC 环境常用

**HPC专用选择：**
- **Rocky Linux 9 / AlmaLinux 9**：RHEL 的最佳开源替代
- **OpenHPC**：提供完整的 HPC 软件栈 (基于 EL9/OpenSUSE)

#### 硬件要求规划 (2025参考)

**管理节点配置：**
```
CPU: 32-64核，支持虚拟化
内存: 128-256GB
存储: 2TB NVMe SSD (系统) + 20TB HDD (镜像/数据)
网络: 双25GbE网卡 (做了Bonding)
```

**计算节点配置：**
```
CPU: 64-192核 (双路 EPYC/Xeon)，高主频
内存: 256GB-1TB (推荐每核心配比 4GB+)
存储: 1TB NVMe SSD (本地暂存)
网络: 200/400Gbps InfiniBand/ROCE (计算网) + 25GbE (管理网)
```

**存储节点配置：**
```
CPU: 16-32核
内存: 64-128GB
存储: PB级并行存储阵列 (HDD + NVMe Cache)
网络: 100/200/400GbE/IB 网卡
```

### 批量安装准备

#### PXE/UEFI 网络安装环境搭建 (EL9)

```bash
# 1. 安装基础服务 (dnf 替代 yum)
dnf install dhcp-server tftp-server httpd syslinux shim shim-x64 grub2-efi-x64

# 2. 配置DHCP服务 (/etc/dhcp/dhcpd.conf)
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    next-server 192.168.1.10;  # TFTP服务器IP
    
    # UEFI 支持判断
    if option arch = 00:07 {
        filename "grubx64.efi";
    } else {
        filename "pxelinux.0";
    }
}

# 3. 启动服务
systemctl enable --now dhcpd tftp httpd
```

#### Kickstart自动化安装 (EL9 语法)

```bash
# 创建Kickstart配置文件 (/var/www/html/ks.cfg)
#version=RHEL9

# System authorization information
authselect --useshadow --passalgo=sha512

# Use network installation
url --url="http://192.168.1.10/rocky9"

# Use text mode install
text

# System language
lang en_US.UTF-8
keyboard --vckeymap=us --xlayouts='us'

# Network information
network --bootproto=dhcp --device=link --activate

# Root password
rootpw --iscrypted $6$...

# System services
services --enabled="chronyd,sshd"

# System timezone
timezone Asia/Shanghai --isUtc

# Disk partitioning information
zerombr
clearpart --all --initlabel
# 推荐使用 LVM
part /boot/efi --fstype="efi" --size=600
part /boot --fstype="xfs" --size=1024
part pv.01 --size=1 --grow
volgroup rl pv.01
logvol / --fstype="xfs" --name=root --vgname=rl --size=51200
logvol /home --fstype="xfs" --name=home --vgname=rl --size=1 --grow
logvol swap --fstype="swap" --name=swap --vgname=rl --size=8192

%packages
@^minimal-environment
@development
chrony
wget
curl
# 常用运维工具
vim
net-tools
bind-utils
nfsv4-client-utils
%end

%post
echo "nameserver 223.5.5.5" > /etc/resolv.conf
systemctl enable --now chronyd
%end
```

#### 系统镜像配置

```bash
# 1. 挂载ISO镜像
mount -o loop Rocky-9.3-x86_64-dvd.iso /mnt

# 2. 复制安装文件
mkdir -p /var/www/html/rocky9
cp -r /mnt/* /var/www/html/rocky9/

# 3. 处理 SELinux (如果开启)
restorecon -Rv /var/www/html/
```

### 系统优化配置

#### 内核参数优化

```bash
# 编辑 /etc/sysctl.conf
# 网络优化
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 5000
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.tcp_fin_timeout = 30

# 内存优化
vm.swappiness = 1
vm.dirty_ratio = 15
vm.dirty_background_ratio = 5

# 文件系统优化
fs.file-max = 1000000
kernel.pid_max = 4194303

# 应用配置
sysctl -p
```

#### 文件系统选择

**XFS文件系统**（推荐用于HPC）：
```bash
# 创建XFS文件系统
mkfs.xfs /dev/sda1

# 挂载选项优化
echo "/dev/sda1 /data xfs defaults,noatime,logbsize=256k 0 0" >> /etc/fstab

# 挂载
mount -a
```

**ext4文件系统**：
```bash
# 创建ext4文件系统
mkfs.ext4 -E stride=16,stripe-width=64 /dev/sda1

# 挂载选项
echo "/dev/sda1 /data ext4 defaults,noatime,data=writeback 0 0" >> /etc/fstab
```

## 2.2 用户与权限管理

### 用户账户管理

#### 批量用户创建

```bash
# 1. 准备用户列表文件 (users.txt)
# 格式：username:uid:gid:fullname
researcher1:1001:1001:Dr. Zhang
researcher2:1002:1001:Dr. Li
student1:1003:1002:Student Wang

# 2. 批量创建用户脚本
#!/bin/bash
while IFS=':' read -r username uid gid fullname; do
    useradd -u $uid -g $gid -c "$fullname" -m -s /bin/bash $username
    echo "$username:defaultpass" | chpasswd
    echo "Created user: $username"
done < users.txt

# 3. 设置用户组
groupadd hpc-users
groupadd hpc-admins
```

#### 用户环境配置

```bash
# 1. 配置默认shell环境
echo 'export PATH=/opt/mpi/bin:$PATH' >> /etc/bashrc
echo 'export LD_LIBRARY_PATH=/opt/mpi/lib:$LD_LIBRARY_PATH' >> /etc/bashrc
echo 'module load mpi' >> /etc/bashrc

# 2. 配置umask (默认权限)
echo 'umask 022' >> /etc/bashrc

# 3. 配置历史记录
echo 'HISTSIZE=10000' >> /etc/bashrc
echo 'HISTTIMEFORMAT="%F %T "' >> /etc/bashrc
```

### 权限管理

#### 文件权限设置

```bash
# 1. 设置项目目录权限
mkdir -p /data/projects
chmod 775 /data/projects
chown root:hpc-users /data/projects

# 2. 设置用户主目录权限
find /home -type d -exec chmod 755 {} \;
find /home -type f -exec chmod 644 {} \;
find /home -name ".*" -type f -exec chmod 644 {} \;

# 3. 设置特殊权限
chmod u+s /usr/bin/passwd  # SUID
chmod g+s /data/projects   # SGID
chmod +t /tmp              # Sticky bit
```

#### ACL权限管理

```bash
# 1. 启用ACL支持
mount -o remount,acl /data

# 2. 设置ACL权限
setfacl -m u:researcher1:rwx /data/shared
setfacl -m g:hpc-users:rx /data/shared
setfacl -m d:u:researcher1:rwx /data/shared  # 默认ACL

# 3. 查看ACL权限
getfacl /data/shared

# 4. 删除ACL权限
setfacl -b /data/shared
```

#### sudo权限配置

```bash
# 编辑 /etc/sudoers (推荐使用 visudo)
# 或者在 /etc/sudoers.d/ 下创建文件

# 允许 hpc-admins 组无密码 sudo
%hpc-admins ALL=(ALL) NOPASSWD: ALL

# 精细化权限控制
researcher1 ALL=(root) /usr/sbin/parted, /usr/bin/dnf

# 日志审计
Defaults logfile=/var/log/sudo.log
Defaults log_input, log_output
```

### 认证与安全

#### SSH密钥认证

```bash
# 1. 生成SSH密钥对
ssh-keygen -t rsa -b 4096 -f ~/.ssh/hpc_key

# 2. 配置SSH服务 (/etc/ssh/sshd_config)
Port 22
Protocol 2
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

# 3. 分发公钥到计算节点
ssh-copy-id -i ~/.ssh/hpc_key.pub user@compute-node1

# 4. 测试连接
ssh -i ~/.ssh/hpc_key user@compute-node1
```

#### 集中式身份认证 (FreeIPA / LDAP)

现代 HPC 环境推荐使用 FreeIPA 代替纯 LDAP 配置，管理更简单且包含 Kerberos。

```bash
# 1. 安装 FreeIPA 客户端 (EL9)
dnf install ipa-client

# 2. 加入域 (交互式)
ipa-client-install --mkhomedir --enable-dns-updates

# 3. 验证身份
kinit researcher1
id researcher1
```

若仍需手动配置 LDAP (EL9 使用 sssd):

```bash
# 1. 安装 sssd
dnf install sssd sssd-tools openldap-clients

# 2. 配置 authselect (关键步骤)
authselect select sssd with-mkhomedir --force

# 3. 配置 SSSD (/etc/sssd/sssd.conf)
[domain/hpc]
id_provider = ldap
auth_provider = ldap
ldap_uri = ldap://ldap-server.hpc.edu.cn
ldap_search_base = dc=hpc,dc=edu,dc=cn
...
```

## 2.3 文件系统管理

### 存储设备管理

#### 磁盘分区与LVM

```bash
# 1. 查看磁盘信息
lsblk
fdisk -l

# 2. 创建LVM卷组
pvcreate /dev/sdb /dev/sdc
vgcreate vg_data /dev/sdb /dev/sdc

# 3. 创建逻辑卷
lvcreate -L 1T -n lv_home vg_data
lvcreate -L 2T -n lv_projects vg_data

# 4. 创建文件系统
mkfs.xfs /dev/vg_data/lv_home
mkfs.xfs /dev/vg_data/lv_projects

# 5. 挂载文件系统
echo "/dev/vg_data/lv_home /home xfs defaults 0 0" >> /etc/fstab
echo "/dev/vg_data/lv_projects /data/projects xfs defaults 0 0" >> /etc/fstab
mount -a
```

#### RAID配置

```bash
# 1. 软件RAID配置
mdadm --create /dev/md0 --level=10 --raid-devices=4 /dev/sd[b-e]
mkfs.xfs /dev/md0
mount /dev/md0 /data/raid10

# 2. RAID监控
mdadm --detail /dev/md0
cat /proc/mdstat

# 3. 硬件RAID管理 (使用厂商工具)
# MegaCLI, storcli, hpssacli 等
```

### 文件系统优化

#### XFS文件系统调优

```bash
# 1. 创建优化的XFS文件系统
mkfs.xfs -f -l size=128m,lazy-count=1 -d agcount=32 /dev/sdb1

# 2. 挂载选项优化
mount -o noatime,logbsize=256k,largeio /dev/sdb1 /data

# 3. XFS参数调整
xfs_growfs /data  # 扩展文件系统
xfs_info /data    # 查看文件系统信息
```

#### 并行文件系统客户端

```bash
# 1. Lustre客户端配置
modprobe lustre
mount -t lustre 192.168.1.10@tcp:/lustre /mnt/lustre

# 2. GPFS客户端配置
mmstartup -a
mmmount /dev/gpfs0

# 3. BeeGFS客户端配置
systemctl start beegfs-client
systemctl enable beegfs-client
```

### 存储性能监控

#### I/O性能测试

```bash
# 1. 使用fio测试I/O性能
fio --name=test --ioengine=libaio --rw=read --bs=4k --size=1G \
   --numjobs=4 --runtime=60 --time_based --filename=/dev/sdb1

# 2. 使用dd测试顺序读写
dd if=/dev/zero of=/data/test bs=1M count=1000 oflag=direct
dd if=/data/test of=/dev/null bs=1M iflag=direct

# 3. 监控I/O统计
iostat -x 1
iotop -o
```

#### 存储空间管理

```bash
# 1. 查看磁盘使用情况
df -h
du -sh /data/projects/*

# 2. 设置磁盘配额
quotacheck -cu /home
quotaon /home
edquota -u username

# 3. 监控存储使用
#!/bin/bash
df -h | awk 'NR>1 {print $5 " " $6}' | while read output; do
    usage=$(echo $output | awk '{print $1}' | cut -d'%' -f1)
    partition=$(echo $output | awk '{print $2}')
    if [ $usage -ge 80 ]; then
        echo "Warning: $partition is $usage% full"
    fi
done
```

## 2.4 网络配置与管理

### 网络接口配置

#### 多网卡绑定 (使用 nmcli)

RHEL/CentOS 7 之后的系统推荐使用 NetworkManager (`nmcli`) 进行配置。

```bash
# 1. 创建 Bond 接口 (mode 4 = 802.3ad LACP)
nmcli con add type bond con-name bond0 ifname bond0 mode 802.3ad ipv4.method manual ipv4.addresses 192.168.1.10/24 ipv4.gateway 192.168.1.1

# 2. 添加从接口 (Slaves)
nmcli con add type ethernet slave-type bond con-name bond0-port1 ifname eth0 master bond0
nmcli con add type ethernet slave-type bond con-name bond0-port2 ifname eth1 master bond0

# 3. 激活连接
nmcli con up bond0
```

#### InfiniBand配置

```bash
# 1. 安装OFED驱动
./mlnxofedinstall --all

# 2. 配置IPoIB
# /etc/sysconfig/network-scripts/ifcfg-ib0
DEVICE=ib0
TYPE=InfiniBand
ONBOOT=yes
BOOTPROTO=static
IPADDR=10.1.1.10
NETMASK=255.255.255.0
MTU=65520

# 3. 测试InfiniBand连接
ibstat
ibping remote-node
```

### 网络性能优化

#### TCP参数调优

```bash
# 1. 网络缓冲区优化
echo 'net.core.rmem_max = 134217728' >> /etc/sysctl.conf
echo 'net.core.wmem_max = 134217728' >> /etc/sysctl.conf
echo 'net.ipv4.tcp_rmem = 4096 87380 134217728' >> /etc/sysctl.conf
echo 'net.ipv4.tcp_wmem = 4096 65536 134217728' >> /etc/sysctl.conf

# 2. 连接队列优化
echo 'net.core.somaxconn = 65535' >> /etc/sysctl.conf
echo 'net.core.netdev_max_backlog = 5000' >> /etc/sysctl.conf

# 3. 应用配置
sysctl -p
```

#### 网络接口调优

```bash
# 1. 禁用不必要的网络功能
ethtool -K eth0 gro off
ethtool -K eth0 tso off
ethtool -K eth0 lro off

# 2. 调整中断亲和性
echo 2 > /proc/irq/16/smp_affinity
echo 4 > /proc/irq/17/smp_affinity

# 3. 配置RSS (Receive Side Scaling)
ethtool -L eth0 combined 8
```

### 网络监控与故障排除

#### 网络性能测试

```bash
# 1. 带宽测试
iperf3 -s  # 服务端
iperf3 -c server-ip  # 客户端

# 2. 延迟测试
ping -c 100 -i 0.1 server-ip

# 3. 网络质量测试
ntttcp -s  # 发送端
ntttcp -r  # 接收端
```

#### 网络故障诊断

```bash
# 1. 现代网络工具替换
ping -c 3 gateway-ip
tracepath target-host   # 替代 traceroute

# 2. 端口检查 (ss 替代 netstat)
ss -tuln
ss -tip

# 3. 接口详细信息
ip -d link show bond0
ethtool -S eth0         # 查看网卡统计(丢包/错误)

# 4. 邻居表 (ARP)
ip neigh show
```

## 2.5 系统监控与性能调优

### 系统监控工具

#### 基础监控命令

```bash
# 1. CPU使用率监控
top
htop
sar -u 1 10

# 2. 内存使用监控
free -h
vmstat 1 10
sar -r 1 10

# 3. 磁盘I/O监控
iostat -x 1
iotop
sar -d 1 10

# 4. 网络监控
iftop
nload
sar -n DEV 1 10
```

#### 基础监控工具 (Modern)

```bash
# 1. 增强型终端监控
htop    # 交互式进程查看
btop    # 现代化资源监控 UI (推荐)
dstat   # 多功能系统资源统计 (dnf install dstat)

# 2. 历史数据记录 (pcp / sysstat)
dnf install pcp-system-tools
pmchart # 图形化分析
```

#### 企业级监控平台 (Prometheus + Grafana)

HPC 集群通常采用分布式监控方案：

1.  **Node Exporter** (部署在所有节点):
    - 收集 CPU/Mem/Disk/Net 指标
    - 暴露 metrics 给 Prometheus
    
2.  **Prometheus** (部署在管理节点):
    - 时序数据库，拉取数据
    - 强大的 PromQL 查询语言

3.  **Grafana** (部署在管理节点):
    - 可视化大屏
    - 预警通知


### 性能分析工具

#### CPU性能分析

```bash
# 1. 使用perf分析CPU使用
perf top -p pid
perf record -p pid
perf report

# 2. 使用 eBPF 工具 (bcc-tools) 分析
# dnf install bcc-tools
/usr/share/bcc/tools/execsnoop  # 监控进程执行
/usr/share/bcc/tools/biolatency # 块设备 I/O 延迟分布
/usr/share/bcc/tools/profile    # CPU 采样剖析
```

#### 内存分析

```bash
# 1. 内存使用分析
cat /proc/meminfo
cat /proc/slabinfo
cat /proc/vmallocinfo

# 2. 内存泄漏检测
valgrind --tool=memcheck --leak-check=full ./program

# 3. 内存映射分析
cat /proc/pid/maps
pmap pid
```

### 系统调优实践

#### 内核参数调优

```bash
# 1. 内存管理优化
echo 'vm.swappiness = 1' >> /etc/sysctl.conf
echo 'vm.dirty_ratio = 15' >> /etc/sysctl.conf
echo 'vm.dirty_background_ratio = 5' >> /etc/sysctl.conf

# 2. 进程调度优化
echo 'kernel.sched_min_granularity_ns = 10000000' >> /etc/sysctl.conf
echo 'kernel.sched_wakeup_granularity_ns = 15000000' >> /etc/sysctl.conf

# 3. 文件系统优化
echo 'fs.file-max = 1000000' >> /etc/sysctl.conf
echo 'kernel.pid_max = 4194303' >> /etc/sysctl.conf
```

#### 服务优化

```bash
# 1. 禁用不必要的服务
systemctl disable postfix
systemctl disable avahi-daemon
systemctl disable cups

# 2. 优化SSH服务
# /etc/ssh/sshd_config
UseDNS no
GSSAPIAuthentication no

# 3. 优化NTP服务
systemctl enable chronyd
chronyc sources -v
```

#### 文件系统优化

```bash
# 1. 挂载选项优化
echo "/dev/sda1 /data xfs defaults,noatime,logbsize=256k 0 0" >> /etc/fstab

# 2. 文件系统参数调优
tune2fs -o journal_data_writeback /dev/sda1  # ext4
xfs_io -c "setattr -S sync" /data/file       # XFS

# 3. 目录结构优化
mkdir -p /data/{home,projects,scratch,archive}
chmod 755 /data/{home,projects}
chmod 777 /data/scratch
```

### 性能基准测试

#### CPU基准测试

```bash
# 1. 使用sysbench测试CPU
sysbench cpu --cpu-max-prime=20000 run

# 2. 使用Linpack测试
./xhpl

# 3. 使用SPEC CPU测试
./runspec --config=myconfig --rate=base cpu2017
```

#### 内存基准测试

```bash
# 1. 使用stream测试内存带宽
./stream_c.exe

# 2. 使用mbw测试内存拷贝
mbw -t 10 1000

# 3. 使用memtest测试内存稳定性
memtest86+
```

#### 存储基准测试

```bash
# 1. 使用fio测试存储性能
fio --name=randread --ioengine=libaio --rw=randread --bs=4k --size=1G \
   --numjobs=4 --runtime=60 --time_based --filename=/dev/sdb1

# 2. 使用iozone测试文件系统
iozone -a -g 1g -n 1m -N -i 0 -i 1 -i 2

# 3. 使用dd测试顺序读写
dd if=/dev/zero of=/data/test bs=1M count=1000 oflag=direct
dd if=/data/test of=/dev/null bs=1M iflag=direct
```

## 本章小结

Linux系统管理是HPC运维工程师的核心技能。本章详细介绍了：

1. **系统安装与配置**：批量安装、自动化部署、系统优化
2. **用户与权限管理**：批量用户管理、权限控制、安全认证
3. **文件系统管理**：存储设备管理、文件系统优化、性能监控
4. **网络配置与管理**：多网卡配置、网络优化、故障诊断
5. **系统监控与性能调优**：监控工具使用、性能分析、系统优化

掌握这些技能，能够确保HPC系统的稳定运行和高效管理。在实际工作中，需要根据具体的硬件配置和应用需求，灵活运用这些技术和方法。