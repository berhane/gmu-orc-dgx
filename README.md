---
description: Temporary documentation for GMU ORC's DGX A100 Server
---

# Getting Started

## Hardware Specs

You can learn more about NVIDIA DGX A100 systems here:

[https://www.nvidia.com/en-us/data-center/dgx-a100/](https://www.nvidia.com/en-us/data-center/dgx-a100/)

|  |  |
| :--- | :--- |
| **GPUs** | 8x NVIDIA A100 Tensor Core GPUs |
|  | 320 GB total memory |
| **Performance** | 5 petaFLOPS AI |
|  | 10 petaFLOPS INT8 |
| **CPU** | Dual AMD Rome 7742, |
|  | 128/256 cores/threads total, 2.25 GHz \(base\), 3.4 GHz \(max boost\) |
| **System Memory** | 1TB |
| **NVIDIA NVSwitches** | 6 |
| **Networking** | 8x Single-Port Mellanox ConnectX-6 VPI 200Gb/s HDR InfiniBand |
|  | 1x Dual-Port Mellanox ConnectX-6 VPI |
|  | 10/25/50/100/200Gb/s Ethernet |
| **Storage** | OS: 2x 1.92TB M.2 NVME drives |
|  | Internal Storage: 15TB |
|  | \(4x 3.84TB\) U.2 NVME drives |
| **Base OS** | Ubuntu 20.04 LTS |

## Getting Access

The DGX server is a part of the new Hopper cluster. Users would need to log into Hopper to submit jobs to the DGX.

Log into the Hopper cluster with:

```bash
$ ssh <username>@hopper.orc.gmu.edu.
```

You can log into the DGX only if you only have an active job on the DGX: 

* you have submitted a SLURM batch job \(using `sbatch`\) and it is actively running on the DGX, or 
* you have an active SLURM interactive session \(using `salloc`\) on the DGX

Depending on why you want access to the DGX, you can take these two approaches.

### For Quick Compiling and Testing

Because the DGX has a different OS \(Ubuntu 20.04 LTS\) and CPU architecture \(AMD EPYC Zen2\), you would likely need to recompile your code on the DGX itself and run quick tests before submitting any production runs. For that purpose, you can request a small interactive session via SLURM:

* If you don't need a GPU, you can request 1 core :

```bash
$  salloc -p gpuq -q gpu -n 1 -t 0-01:00:00 
```

* If you need a GPU, you can request a GPU along with CPU cores

```bash
$ salloc -p gpuq -q gpu -n 1 --gres=gpu:A100:1 -t 0-01:00:00   
```

This will log you into the DGX as soon as the requested resource is available:

```bash
$ salloc -p gpuq -q gpu -n 1 -t 0-01:00:00

salloc: Granted job allocation 5562
salloc: Waiting for resource configuration
salloc: Nodes dgx-a100-01 are ready for job

$user@dgx-a100-01:~$
```

### For Production Calculations

For production runs, you can submit your job batch or interactive job through SLURM and `ssh` into the DGX if necessary.

```bash
$ ssh dgx-a100-01
```

Otherwise, your connection attempt will be declined with a message like this:

_Access denied by pam\_slurm\_adopt: you have no active jobs on this node  
Connection closed by server on port \#\#_

## Running Jobs

The DGX runs Ubuntu 20.04 LTS. You can run calculations on it by submitting jobs via SLURM in batch or interactive mode from Hopper.

Both containerized and native applications are supported. You can run

* containerized applications using Singularity containers you build or ones we provide
* native applications you have compiled or those we provision using Lmod modules

These two approaches are described below.

## Running Containerized Applications

We provide a growing list of Singularity containers in a shared location. You are also welcome to pull and run your own Singularity containers.

### Using Shared Containers

Containers and examples available for all users can be found on at `/containers/dgx/Containers` and `/containers/dgx/Examples`. The environmental variable `$SINGULARITY_BASE` points to `/containers/dgx/Containers`

Currently available containers can be viewed with:

```bash
$ tree /containers/dgx/Containers

/containers/dgx/Containers
├── autodock
│   └── autodock_2020.06.sif
├── caffe
│   └── caffe_20.03-py3.sif
├── digits
│   └── digits_21.02-tensorflow-py3.sif
├── gamess
│   └── gamess_17.09-r2-libcchem.sif
├── gromacs
│   └── gromacs-2020_2.sif
├── lammps
│   ├── lammps_10Feb2021.sif
│   └── lammps_29Oct2020.sif
├── namd
│   ├── namd_2.13-multinode.sif
│   ├── namd_2.13-singlenode.sif
│   └── namd_3.0-alpha3-singlenode.sif
├── ngc-preflightcheck
│   └── ngc-preflightcheck_20.11.sif
├── nvidia-hpc-benchmarks
│   └── hpc-benchmarks_20.10-hpl.sif
├── pytorch
│   └── pytorch_21.02-py3.sif
├── quantum_espresso
│   └── quantum_espresso_v6.7.sif
└── tensorflow
    ├── tensorflow_21.02-tf1-py3.sif
    ├── tensorflow_21.02-tf2-py3.sif
    ├── tensorflow_21.04-tf1-py3.sif
    └── tensorflow_21.04-tf2-py3.sif
```

We encourage using these shared containers because they are optimized by NVIDIA to run well on the DGX. Sharing containers also saves storage space. Please let us know if you want us to add particular containers.

### Building your Own Containers

Modern containers come from many registries \(Dockerhub, NGC, SingularityHub, Biocontainers, ... etc \) and in different formats \(Docker, Singularity, OCI\) and runtimes \(Docker, Singularity, CharlieCloud, ...\).

{% hint style="warning" %}
Please keep in mind that you can not build or run Docker containers directly on Hopper or the DGX. You would need to pull and convert Docker containers to Singularity format and run the Singularity containers.
{% endhint %}

We use Docker containers pulled from [NVIDIA GPU Cloud \(NGC\)](https://ngc.nvidia.com) catalog in the examples below, but the same steps apply to containers from other sources. The NVIDIA GPU Cloud \(NGC\) provides simple access to GPU-optimized software for deep learning, data science and high-performance computing \(HPC\). An NGC account grants you access to these tools as well as the ability to set up a private registry to manage your customized software. However, it is not absolutely necessary that you have an NGC account. Please see the link below for more:

{% page-ref page="using-ngc.md" %}

#### NGC commands:

This example below demonstrates how to search and pull down a GROMACS image using the NGC CLI:

```bash
$ ngc registry image list 
$ ngc registry image list | grep -i <container_name> 
$ ngc registry image info nvcr.io/<container_name>:<containter_tag>
$ ngc registry image list|grep -i gromacs 

| GROMACS | hpc/gromac | 2020.2 | 275.47 MB | Sep 24, | unlocked|

$ ngc registry image info nvcr.io:hpc/gromacs 

-------------------------------------------------- 
 Image Repository Information  
 Name: GROMACS  
 Short Description: GROMACS is a popular molecular dynamics application used to simulate proteins and lipids.  
 Built By: KTH Royal Institute of Technology 
 Publisher: KTH Royal Institute of Technology 
 Multinode Support: False 
 Multi-Arch Support: True 
 Logo: https://assets.nvidiagrid.net/ngc/logos/ISV-OSS-Non-Nvidia-Publishing-Gromacs.png 
 Labels: Covid-19, HPC, Healthcare, High Performance Computing, Supercomputing, arm64, x86_64 
 Public: Yes 
 Last Updated: Sep 24, 2020 
 Latest Image Size: 275.47 MB 
 Latest Tag: 2020.2 
 Tags: 
  2020.2 
  2020 
  2020.2-arm64 
  2020.2-x86_64 
  2018.2 
  2016.4
  
$ ngc registry image info nvcr.io/hpc/gromacs:2020.2 

-------------------------------------------------- 
 Image Information 
 Name: hpc/gromacs:2020.2 
 Architecture: amd64 
 Schema Version: 1 
 Image Size: 275.47 MB 
 Last Updated: Jun 22, 2020 
--------------------------------------------------
```

#### Pulling Docker containers and building Singularity containers:

Once you select a Docker container to use, you need to pull it down and convert it to a Singularity image format with the following command. You would need to load `singularity` module first.

```bash
$ module load singularity
```

```bash
$ singularity build <container_name>_<container_version/tag>.sif docker://nvcr.io/<hpc>/<container_name><container_version/tag>
```

Here is an example for preparing a GROMACS Singularity container:

```bash
$ module load singularity
$ singularity build gromacs-2020_2.sif docker://nvcr.io/hpc/gromacs:2020.2
```

Please note that we have adapted the following convention on naming Singularity image files.

* we use SIF instead of SIMG for the file extension 
* we name containers as `<container_name>_<container_version/tag>.sif` 

Also note that you can pull the containers from NGC, DockerHub or any other source, but we encourage using ones from the NGC registry if one is available because they are optimized for NVIDIA GPUs.

## Running Native Applications

If you want to run native GPU-capable applications, you can run them much like you have on Argo.

* load up the module for the GPU-capable application/version
* run the application

We currently have a limited set of native applications that have been tested on the DGX. That will increase over time.

{% hint style="warning" %}
The DGX is very different from Argo and Hopper in terms of OS, CPU and GPU architecture as well as the software stack running on it. Therefore, you would generally need to recompile your code on the DGX itself using the software stack built for the DGX. Please email orchelp@gmu.edu if you need help.
{% endhint %}

| **System** | **Argo** | **Hopper** | **DGX** |
| :--- | :--- | :--- | :--- |
| **OS** | CentOS 7.8 | CentOS 8.3 | Ubuntu 20.04 |
| **CPU** | Intel  | Intel | AMD |
| **GPUs** | K80, V100 | - | A100 |
| **NVIDIA  driver version** | 440.x-455.y | - | 450.x |

To access modules built for the DGX, first load into the DGX by creating a short interactive session:

```bash
$ salloc -p gpuq -q gpu -n 1 -t 0-01:00:00

salloc: Granted job allocation 5562
salloc: Waiting for resource configuration
salloc: Nodes dgx-a100-01 are ready for job

$
```

You should see a `hosts/dgx` module loaded and other  modules that are available to you:

```text

$ module avail
...
----- GNU-9.3.0 ---------
   openmpi/4.0.4-ev    python/3.7.6-tf    python/3.8.6-mf (L,D)

----- Independent ---------
   cuda/10.2.89    cuda/11.0.2    cuda/11.2.1 (D)    gnu9/9.3.0    intel/2020.2    orca/4.2.1    singularity/3.7.1

----- Core ---------
   hosts/dgx (L)    lmod    settarg    use.own
...
```

## Scheduling SLURM Jobs

Once you have a native or containerized application, you can run it through SLURM either interactively or using batch submission scripts. Both approaches are discussed below. To run jobs on the DGX, you would need

You can run a native or containerized application through SLURM either interactively or using batch submission scripts. Both approaches are discussed below. To run jobs on the DGX, you would need

* to have a SLURM account on Hopper AND
* be eligible to use the 'gpu' Quality-of-Service \(QoS\) 

The DGX is part of the ‘gpuq’ partition.

```bash
$ sinfo -o "%12P %5D %14F %8z %10m %.11l %15N %G" 

PARTITION    NODES NODES(A/I/O/T) S:C:T    MEMORY       TIMELIMIT NODELIST        GRES
debug        3     0/3/0/3        2:24:1   180000         1:00:00 hop[043-045]    (null)
interactive  3     0/3/0/3        2:24:1   180000        12:00:00 hop[043-045]    (null)
contrib      42    6/36/0/42      2:24:1   180000      6-00:00:00 hop[001-042]    (null)
normal*      25    21/4/0/25      2:24:1   180000      3-00:00:00 hop[046-070]    (null)
gpuq         1     0/1/0/1        8:16:1   1024000     2-00:00:00 dgx-a100-01     gpu:A100-40g:6,gpu:1g.5gb:9,gpu:2g.10gb:1,gpu:3g.20gb:1
orc-test     70    27/43/0/70     2:24:1   180000      1-00:00:00 hop[001-070]    (null)
```

The GPU list shows 6x A100 GPUs as well as 9x 1g.5gb, 1x 2g.10gb and 1x 3g.20gb resources. The latter three types of resources are a product of a partitioning scheme called Multi-Instance GPU (MIG).  

### GPU partitioning

The DGX A100 has 8 NVIDIA Tesla A100 GPUs which can be further partitioned into smaller slices to optimize access and utilization. For example, each GPU can be sliced into as many as 7 instances when enabled to operate in [MIG (Multi-Instance GPU)](https://docs.nvidia.com/datacenter/tesla/mig-user-guide/index.html) mode. 

![MIG-mode](https://docs.nvidia.com/datacenter/tesla/mig-user-guide/graphics/gpu-mig-overview.jpg)

GPU Instance Profiles on A100 Profile 

| Name 	| Fraction of Memory |	Fraction of SMs |	Hardware Units |	L2 Cache Size |	Number of Instances Available |
| :---  | :---               | :----            | :--------        | :--------        | :------------------- 
| MIG 1g.5gb  |	1/8 |	1/7 |	0 NVDECs |	1/8 |	7 |
| MIG 2g.10gb |	2/8 |	2/7 |	1 NVDECs |	2/8 |	3 |
| MIG 3g.20gb |	4/8 |	3/7 |	2 NVDECs |	4/8 |	2 |
| MIG 4g.20gb | 4/8 |	4/7 |	2 NVDECs |	4/8 |	1 |
| MIG 7g.40gb |	Full |	7/7 |	5 NVDECs |	Full |	1 |

Our DGX is currently partitioned such that six of the 8 A100 GPUs (GPU ID 0-5) are not partitioned while the last two (GPU ID 6-7) are partitioned into slices of different sizes.

| GPU ID | Size  | GRES name 
| :--- | :--- | :---- |
| 0  | Full A100 | A100-40g  
| 1  | Full A100 | A100-40g 
| 2  | Full A100 | A100-40g
| 3  | Full A100 | A100-40g
| 4  | Full A100 | A100-40g
| 5  | Full A100 | A100-40g
| 6  | 1/7 A100  | 1g.5gb
|    | 1/7 A100  | 1g.5gb
|    | 1/7 A100  | 1g.5gb
|    | 1/7 A100  | 1g.5gb
|    | 1/7 A100  | 1g.5gb
|    | 1/7 A100  | 1g.5gb
|    | 1/7 A100  | 1g.5gb
| 7  | 1/7 A100  | 1g.5gb
|    | 1/7 A100  | 1g.5gb
|    | 2/7 A100 | 2g.10gb
|    | 4/7 A100 | 3g.20gb

The way the GPUs are partitioned will likely change over time to optimize utilization.

The best way to take advantage of MIG operation is to analyze the demands of your job and determine which GPU size is available and suitable for it. For example, if your simulation uses very small memory, you would be better off using a 1g.5gb slice and leaving the bigger partitions to jobs that need more GPU memory. Another consideration for machine learning jobs is the difference in demands of training and inference tasks. Training tasks are more compute and memory intensive, this they are a better for for a full GPU or large partition while inference tasks would run sufficiently on smaller slices. 

### Interactive Mode

You can request an interactive access the DGX A100 server through SLLURM as follows:

```bash
$ salloc -p gpuq -q gpu --gres=gpu:A100:1 -t 0-01:00:00 

salloc: Granted job allocation 2185 
salloc: Waiting for resource configuration 
salloc: Nodes dgx-a100-01 are ready for job

$ 
```

Once your reservation is available, you will be logged into the DGX automatically:

```bash
$ hostname -s
dgx-a100-01
```

To run the container while connected:

```bash
$ singularity run [ --nv] [other_options] <container_name>_<container_version/tag>.sif <command>
```

As an example, the following command runs a Python script using Tensorflow container

```bash
$ singularity run --nv -B ${PWD}:/host_pwd --pwd /host_pwd /containers/dgx/Containers/tensorflow/tensorflow_21.02-tf1-py3.sif python test_single_gpu.py
```

You can run on any one or more GPUs. The GPUs are indexed 0-7. Since this is a shared resource, we encourage you to monitor the GPUs usage and selectively submit to idle GPU\(s\) when running jobs interactively. For example, the output of `nvidia-smi` command suggests that there GPUs indexed 0,1,2 are being actively used, and you should run your jobs on one of the other GPUs.

```text
$ nvidia-smi

Thu Mar 15 10:58:08 2021
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 450.102.04   Driver Version: 450.102.04   CUDA Version: 11.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  A100-SXM4-40GB      On   | 00000000:07:00.0 Off |                    0 |
| N/A   29C    P0    52W / 400W |      0MiB / 40537MiB |      0%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+
|   1  A100-SXM4-40GB      On   | 00000000:0F:00.0 Off |                    0 |
| N/A   30C    P0    54W / 400W |      0MiB / 40537MiB |      0%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+
|   2  A100-SXM4-40GB      On   | 00000000:47:00.0 Off |                    0 |
| N/A   29C    P0    53W / 400W |      0MiB / 40537MiB |      0%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+
|   3  A100-SXM4-40GB      On   | 00000000:4E:00.0 Off |                    0 |
| N/A   29C    P0    55W / 400W |      0MiB / 40537MiB |      0%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+
|   4  A100-SXM4-40GB      On   | 00000000:87:00.0 Off |                    0 |
| N/A   33C    P0    53W / 400W |      0MiB / 40537MiB |      0%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+
|   5  A100-SXM4-40GB      On   | 00000000:90:00.0 Off |                    0 |
| N/A   31C    P0    51W / 400W |      0MiB / 40537MiB |      0%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+
|   6  A100-SXM4-40GB      On   | 00000000:B7:00.0 Off |                   On |
| N/A   31C    P0    46W / 400W |     25MiB / 40537MiB |     N/A      Default |
|                               |                      |              Enabled |
+-------------------------------+----------------------+----------------------+
|   7  A100-SXM4-40GB      On   | 00000000:BD:00.0 Off |                   On |
| N/A   31C    P0    42W / 400W |     25MiB / 40537MiB |     N/A      Default |
|                               |                      |              Enabled |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| MIG devices:                                                                |
+------------------+----------------------+-----------+-----------------------+
| GPU  GI  CI  MIG |         Memory-Usage |        Vol|         Shared        |
|      ID  ID  Dev |           BAR1-Usage | SM     Unc| CE  ENC  DEC  OFA  JPG|
|                  |                      |        ECC|                       |
|==================+======================+===========+=======================|
|  6    7   0   0  |      3MiB /  4864MiB | 14      0 |  1   0    0    0    0 |
|                  |      0MiB /  8191MiB |           |                       |
+------------------+----------------------+-----------+-----------------------+
|  6    8   0   1  |      3MiB /  4864MiB | 14      0 |  1   0    0    0    0 |
|                  |      0MiB /  8191MiB |           |                       |
+------------------+----------------------+-----------+-----------------------+
|  6    9   0   2  |      3MiB /  4864MiB | 14      0 |  1   0    0    0    0 |
|                  |      0MiB /  8191MiB |           |                       |
+------------------+----------------------+-----------+-----------------------+
|  6   10   0   3  |      3MiB /  4864MiB | 14      0 |  1   0    0    0    0 |
|                  |      0MiB /  8191MiB |           |                       |
+------------------+----------------------+-----------+-----------------------+
|  6   11   0   4  |      3MiB /  4864MiB | 14      0 |  1   0    0    0    0 |
|                  |      0MiB /  8191MiB |           |                       |
+------------------+----------------------+-----------+-----------------------+
|  6   12   0   5  |      3MiB /  4864MiB | 14      0 |  1   0    0    0    0 |
|                  |      0MiB /  8191MiB |           |                       |
+------------------+----------------------+-----------+-----------------------+
|  6   13   0   6  |      3MiB /  4864MiB | 14      0 |  1   0    0    0    0 |
|                  |      0MiB /  8191MiB |           |                       |
+------------------+----------------------+-----------+-----------------------+
|  7    1   0   0  |     11MiB / 20096MiB | 42      0 |  3   0    2    0    0 |
|                  |      0MiB / 32767MiB |           |                       |
+------------------+----------------------+-----------+-----------------------+
|  7    5   0   1  |      7MiB /  9984MiB | 28      0 |  2   0    1    0    0 |
|                  |      0MiB / 16383MiB |           |                       |
+------------------+----------------------+-----------+-----------------------+
|  7   13   0   2  |      3MiB /  4864MiB | 14      0 |  1   0    0    0    0 |
|                  |      0MiB /  8191MiB |           |                       |
+------------------+----------------------+-----------+-----------------------+
|  7   14   0   3  |      3MiB /  4864MiB | 14      0 |  1   0    0    0    0 |
|                  |      0MiB /  8191MiB |           |                       |
+------------------+----------------------+-----------+-----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  running processes found                                                    |
|  0 App1  1%                                                                 |
|  1 App2  12%                                                                |
|  2 App3  90%                                                                |
+-----------------------------------------------------------------------------+
```

To select particular GPU\(s\), you can use the `SINGULARITYENV_CUDA_VISIBLE_DEVICES` environmental variable. For example, you can select the 1st and 3rd GPU by setting

```bash
$ SINGULARITYENV_CUDA_VISIBLE_DEVICES=0,2
```

SLURM specifies the GPU indices assigned to your job to the `SLURM_JOB_GPUS` environmental variable. So you can set

```text
$ SINGULARITYENV_CUDA_VISIBLE_DEVICES=${SLURM_JOB_GPUS}
```

For example, the following commands will run on any number of GPU assigned to you:

```bash
$ SINGULARITYENV_CUDA_VISIBLE_DEVICES=${SLURM_JOB_GPUS} 

$ singularity run --nv -B ${PWD}:/host_pwd --pwd /host_pwd /containers/dgx/Containers/tensorflow/tensorflow_21.02-tf1-py3.sif python test_single_gpu.pyUseful tools for monitoring the GPU usage
```

While you are on the server, you can use these tools to monitor the GPU usage:

* `nvidia-smi`  -  has lots of option to set and monitor GPUs

  `nvitop -m` - great for live monitoring GPU usage, complete with color coding

* `nvtop` - ncurses-based  gpu monitoring similar to '`htop`' 

**Please remember to log out of the DGX A100 server when you finish running your interactive job.**

### Batch Mode

Below is a sample SLURM batch submission file you can use as an example to submit your jobs. Save the information into a file \(say `run.slurm`\), and submit it by entering `sbatch run.slurm`. Please update `<N_CPU_CORES>`, `<MEM_PER_CORE>` and `<N_GPUs>` to reflect the number of CPU cores and GPUs you need. Please note that the DGX has 128 CPU cores, 8 GPUs and 1TB of system memory.

```bash
#!/bin/bash 
#SBATCH --partition=gpuq 
#SBATCH --qos=gpu 
#SBATCH --job-name=jmultigpu_basics 
#SBATCH --output=jmultigpu_basics.%j 
#SBATCH --nodes=1 
#SBATCH --ntasks-per-node=<N_CPU_CORES> 
#SBATCH --gres=gpu:A100:<N_GPUs> 
#SBATCH --mem-per-cpu=<MEM_PER_CORE>  
#SBATCH --export=ALL 
#SBATCH --time=0-01:00:00 

set echo 
umask 0022 
nvidia-smi 
env|grep -i slurm

SINGULARITY_BASE=/containers/dgx/Containers 
CONTAINER=${SINGULARITY_BASE}/tensorflow/tensorflow_21.02-tf1-py3.sif 
SINGULARITY_RUN="singularity run --nv -B ${PWD}:/host_pwd --pwd /host_pwd" 

SCRIPT=multigpu_basics.py 
${SINGULARITY_RUN} ${CONTAINER} python ${SCRIPT} | tee ${SCRIPT}.log
```

We encourage the use of environmental variables to make the job submission file cleaner and easily reusable.

The syntax for running different containers varies depending on the application. Please check the [NGC page](https://ngc.nvidia.com/catalog/containers) for more instructions on running these containers using Singularity.

## Storage Locations

Currently, these locations have been designated for storing shared and user-specific containers.

* Containers
  * Shared:`/containers/dgx/Containers`
  * User-specific:`/containers/dgx/UserContainers/$USER`
* Examples
  * Containerized applications:`/containers/dgx/Examples`
  * Native applications: `/groups/ORC-VAST/app-tests`

## Sample Runs

We provide some sample calculations to facilitate setting up and running calculations:

* examples demonstrating how you run different applications to`/containers/dgx/Examples`
* examples on running native and containerized applications is available here:`/groups/ORC-VAST/app-tests`
* The examples at [https://gitlab.com/NVHPC/ngc-examples](https://gitlab.com/NVHPC/ngc-examples) are helpful. For many applications, there are no instructions on running the containers using Singularity, but you should be able to build one from the Docker image and run it. 

