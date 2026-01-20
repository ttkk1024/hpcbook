# 附录C 性能测试工具

## C.1 基准测试工具

### C.1.1 综合计算性能 (HPL/HPCG)

#### HPL (High Performance Linpack) AI 优化版
**HPL.dat 配置文件 (针对大规模集群优化)**
```bash
# /opt/hpl/bin/Linux/HPL.dat
HPLinpack benchmark input file
Innovative Computing Laboratory, University of Tennessee
HPL.out      output file name (if any)
6            device out (6=stdout,7=stderr,file)
1            # of problems sizes (N)
200000       Ns  # N值需根据总内存调整 (N * N * 8 < Total Memory * 0.9)
1            # of NBs
256          NBs # H100/A100 推荐较大块大小 (256/512)
0            PMAP process mapping (0=Row-,1=Column-major)
1            # of process grids (P x Q)
8            Ps  # P * Q = 总进程数 (建议 P <= Q)
16           Qs
16.0         threshold
1            # of panel fact
2            PFACTs (0=left, 1=Crout, 2=Right)
1            # of recursive stopping criterium
4            NBMINs (>= 1)
1            # of panels in recursion
2            NDIVs
1            # of recursive panel fact.
2            RFACTs (0=left, 1=Crout, 2=Right)
1            # of broadcast
1            BCASTs (0=1rg,1=1rM,2=2rg,3=2rM,4=Lng,5=LnM)
1            # of lookahead depth
1            DEPTHs (>=0)
2            SWAP (0=bin-exch,1=long,2=mix)
64           swapping threshold
0            L1 in (0=transposed,1=no-transposed) form
0            U  in (0=transposed,1=no-transposed) form
1            Equilibration (0=no,1=yes)
8            memory alignment in double (> 0)
```

**NVIDIA HPL (xhpl_cuda) 运行脚本**
```bash
#!/bin/bash
# /opt/hpl/run_hpl_cuda.sh
# 使用 NVIDIA 优化的 HPL 容器运行

# 环境变量
export OMP_NUM_THREADS=8
export OMP_PROC_BIND=TRUE
export OMP_PLACES=cores

# 启动容器运行
srun --mpi=pmix --gpus-per-node=8 --ntasks-per-node=8 \
    apptainer run --nv \
    docker://nvcr.io/nvidia/hpc-benchmarks:24.03 \
    ./xhpl_cuda
```

### C.1.2 内存子系统 (STREAM/Intel MLC)

#### STREAM (GPU 版)
**BabelStream 运行脚本**
```bash
#!/bin/bash
# /opt/stream/run_babelstream.sh
# 测试 GPU HBM3 内存带宽

# 编译 (需 CUDA 环境)
nvcc -O3 -std=c++11 -arch=sm_90 main.cu -o babelstream_cuda

# 运行
./babelstream_cuda --arraysize 400000000
```

#### Intel Memory Latency Checker (MLC)
**运行脚本**
```bash
#!/bin/bash
# 测试 DDR5 内存延迟与带宽矩阵
# --peak_injection_bandwidth: 测试峰值注入带宽
# --idle_latency: 测试空闲延迟

mlc --peak_injection_bandwidth
mlc --idle_latency
mlc --latency_matrix
```

## C.2 存储性能测试工具

### C.2.1 综合 I/O (IO500/FIO)

#### IO500 (HPC 存储排名标准)
**config.ini 配置文件**
```ini
# /opt/io500/config.ini
[global]
datadir = /mnt/lustre/io500_data
timestamp_datadir = TRUE
resultdir = ./results
timestamp_resultdir = TRUE
api = POSIX # 或 MPIIO

[ior-easy]
API = POSIX
blockSize = 1m
transferSize = 1m
filePerProc = TRUE

[ior-hard]
API = POSIX
segmentCount = 1000000

[mdtest-easy]
API = POSIX
n = 1000000

[mdtest-hard]
API = POSIX
n = 1000000
filesPerDir = 10000
```

**运行脚本**
```bash
#!/bin/bash
# 需 MPI 环境
mpirun -np 128 ./io500 config.ini
```

#### FIO (Flash I/O Tester)
**AI 训练负载模拟脚本**
```bash
#!/bin/bash
# /opt/fio/ai_load_sim.fio
# 模拟 PyTorch DataLoader 的随机小文件读取模式

[global]
ioengine=libaio
direct=1
buffered=0
size=100G
numjobs=16
runtime=300
group_reporting
directory=/mnt/nvme/data

[ai-random-read]
# 模拟图像读取 (如 ImageNet)
rw=randread
bs=128k # 对应典型图片大小
iodepth=64
```

## C.3 网络性能测试工具

### C.3.1 互连网络 (OSU Micro-Benchmarks)

**OSU 运行脚本 (InfiniBand/ROCE)**
```bash
#!/bin/bash
# /opt/osu/run_osu.sh

# 1. 基础延迟与带宽
mpirun -np 2 --map-by node ./osu_latency
mpirun -np 2 --map-by node ./osu_bw
mpirun -np 2 --map-by node ./osu_bibw # 双向带宽

# 2. 全互连模式 (All-to-All) - 模拟分布式训练通信压力
# -np 应为集群总核心数或 GPU 数
mpirun -np 64 ./osu_alltoall
mpirun -np 64 ./osu_allreduce
```

### C.3.2 NCCL 性能测试 (AI 专属)

**NCCL Tests 运行脚本**
```bash
#!/bin/bash
# /opt/nccl-tests/run_nccl_perf.sh
# 测试 GPU 间通信带宽 (NVLink + IB)

# 编译 nccl-tests (通常包含在 NVIDIA 容器中)
# 运行 AllReduce 性能扫描
# -b: 开始大小 8 字节
# -e: 结束大小 1GB
# -f: 倍增因子 2
# -g: 每节点 GPU 数 8

srun --mpi=pmix --gpus-per-node=8 --ntasks-per-node=8 \
    /usr/local/bin/all_reduce_perf -b 8 -e 1G -f 2 -g 8
```

## C.4 AI 算力基准测试 (MLPerf)

### C.4.1 MLPerf Training
**运行脚本模板**
```bash
#!/bin/bash
# /opt/mlperf/run_bert.sh
# 运行 BERT 模型训练基准测试

# 拉取 MLPerf 容器
apptainer pull docker://mlperf/training:3.1-nvidia

# 启动训练
srun --gpus-per-node=8 --ntasks-per-node=1 \
    apptainer run --nv training.sif \
    ./run_and_time.sh
```

### C.4.2 MLPerf Inference
**运行脚本模板**
```bash
#!/bin/bash
# /opt/mlperf/run_resnet.sh
# 运行 ResNet50 推理基准测试

python3 -m mlperf_inference.runner \
    --model resnet50 \
    --scenario Offline \
    --accelerator-type gpu
```