---
description: Ways to run NAMD on Hopper
---

# NAMD

You can run different versions of NAMD on the Hopper cluster and the DGX A100 GPU server that is integrated to it.

If you want to take advantage of the GPUs, you can eiither use the Singularity containers we provide at `/containers/dgx/Containers` or the native application we provision using modules.

However, you can run your simulations using the CUDA-enabled NAMD 2.14 module.

If you prefer to use CPUs only, you can load up the NAMD 2.14 module as you would on Argo.

The two approaches, namely running native NAMD and containterized NAMD, are outlined below.

## Using Native Application

### 2.14 GPU enabled

* 2.14 GPU-enabled \(`/opt/sw/other/apps/namd/2.14/NAMD_2.14_Linux-x86_64-verbs-smp-CUDA`\)
  * module = `namd/2.14-verbs-smp-cuda` 
  * sample run - `/groups/ORC-VAST/app-tests/NAMD/native/2.14-CUDA-enabled`
  * sample batch submission file - `/groups/ORC-VAST/app-tests/NAMD/native/2.14-CUDA-enabled/run.slurm`

### 2.14 CPU-only

* 2.14 cpu-only \(`/opt/sw/other/apps/namd/2.14/NAMD_2.14_Linux-x86_64-verbs-smp`\)
  * module - `namd/2.14-verbs-smp`
  * example directory - `/groups/ORC-VAST/app-tests/NAMD/native/2.14-CPUonly`
  * sample batch submission file - `/groups/ORC-VAST/app-tests/NAMD/native/2.14-CPU-only/run.slurm`

## Using Containerized Application

We currently provide the following GPU/CUDA-enabled containers from NVIDIA

### 3.0-alpha GPU enabled

* 3.0-alpha 
  * Container - `/containers/dgx/Containers/namd/namd_3.0-alpha3-singlenode.sif`
  * example directory - `/groups/ORC-VAST/app-tests/NAMD/containerized/3.0`
  * sample batch submission file - `/groups/ORC-VAST/app-tests/NAMD/containerized/3.0/run*gpu*cores.slurm`

### 2.13 GPU enabled

* 2.13 
  * Container - `/containers/dgx/Containers/namd/namd_2.13-singlenode.sif`
  * example directory - `/groups/ORC-VAST/app-tests/NAMD/containerized/2.13`
  * sample batch submission file - `/groups/ORC-VAST/app-tests/NAMD/containerized/2.13/run*gpu*cores.slurm`

## Benchmarks

### Timings

#### Containerized GPU

```text

NAMD/3.0-alpha]  grep -m 1 "Benchmark time" *log |sort -n -k 4 |column -t
1gpus-1cores.log:Info:   Benchmark  time:  1   CPUs  0.0014479   s/step  119.345  ns/day  0  MB  memory
1gpus-2cores.log:Info:   Benchmark  time:  2   CPUs  0.00138185  s/step  125.05   ns/day  0  MB  memory
1gpus-4cores.log:Info:   Benchmark  time:  4   CPUs  0.00136058  s/step  127.005  ns/day  0  MB  memory
1gpus-8cores.log:Info:   Benchmark  time:  8   CPUs  0.00134193  s/step  128.77   ns/day  0  MB  memory
1gpus-16cores.log:Info:  Benchmark  time:  16  CPUs  0.0013593   s/step  127.124  ns/day  0  MB  memory
1gpus-32cores.log:Info:  Benchmark  time:  32  CPUs  0.00142356  s/step  121.386  ns/day  0  MB  memory
-
-
NAMD/2.13]  grep -m 1 "Benchmark time" *log |sort -n -k 4 |column -t
1gpus-1cores.log:Info:   Benchmark  time:  1   CPUs  0.0129561   s/step  0.149955   days/ns  531.312  MB  memory
1gpus-2cores.log:Info:   Benchmark  time:  2   CPUs  0.00804309  s/step  0.0930913  days/ns  534.602  MB  memory
1gpus-4cores.log:Info:   Benchmark  time:  4   CPUs  0.00537852  s/step  0.0622514  days/ns  540.238  MB  memory
1gpus-8cores.log:Info:   Benchmark  time:  8   CPUs  0.0042346   s/step  0.0490116  days/ns  547.203  MB  memory
1gpus-16cores.log:Info:  Benchmark  time:  16  CPUs  0.00265262  s/step  0.0307016  days/ns  566.164  MB  memory
1gpus-32cores.log:Info:  Benchmark  time:  32  CPUs  0.0025912   s/step  0.0299907  days/ns  596.684  MB  memory
```

#### Native GPU

```text

NAMD/2.14-CUDA-enabled]  grep -m 1 "Benchmark time" *log |sort -n -k 4|column -t
1gpus-8cores.log:Info:   Benchmark  time:  8   CPUs  0.0034427   s/step  0.039846   days/ns  582.902  MB  memory
2gpus-8cores.log:Info:   Benchmark  time:  8   CPUs  0.00342256  s/step  0.0396129  days/ns  878.598  MB  memory
1gpus-16cores.log:Info:  Benchmark  time:  16  CPUs  0.00267346  s/step  0.0309429  days/ns  594.891  MB  memory
2gpus-16cores.log:Info:  Benchmark  time:  16  CPUs  0.0021852   s/step  0.0252917  days/ns  912.77   MB  memory
1gpus-32cores.log:Info:  Benchmark  time:  32  CPUs  0.00239043  s/step  0.027667   days/ns  611.156  MB  memory
2gpus-32cores.log:Info:  Benchmark  time:  32  CPUs  0.00192482  s/step  0.022278   days/ns  941.094  MB  memory
```

#### CPU-only

```text
NAMD/2.14-CPUonly]  grep -m 1 "Benchmark time" *log |sort -n -k 4 |column -t
1cores.log:Info:   Benchmark  time:  1   CPUs  0.486551   s/step  5.63137   days/ns  435.906  MB  memory
2cores.log:Info:   Benchmark  time:  2   CPUs  0.247587   s/step  2.86559   days/ns  461.664  MB  memory
4cores.log:Info:   Benchmark  time:  4   CPUs  0.133817   s/step  1.54881   days/ns  675.102  MB  memory
8cores.log:Info:   Benchmark  time:  8   CPUs  0.0682242  s/step  0.789632  days/ns  740.727  MB  memory
16cores.log:Info:  Benchmark  time:  16  CPUs  0.0364464  s/step  0.421833  days/ns  1299.59  MB  memory
32cores.log:Info:  Benchmark  time:  32  CPUs  0.0202405  s/step  0.234265  days/ns  2446.89  MB  memory
48cores.log:Info:  Benchmark  time:  48  CPUs  0.0141087  s/step  0.163296  days/ns  3602.52  MB  memory
```

### Conclusions

* GP-GPU runs are more than 1-2 orders of magnitude faster than CPU-only runs. However, the speedup from using multiple GPUs is small, at least for the APOA1 example.
* NAMD 3.0 offloads almost all computations to the GPU while NAMD 2.1\[3/4\] split the computations between the CPU and GPU
* NAMD 3.0 is 2-3 times faster than NAMD 2.1\[3/4\]
* If using NAMD 3.0, there is no reason to request many CPU cores.

## Caveats

* the NAMD 3.0-alpha container has impressive performance, but it is unclear if it is suitable for production work \([http://www.ks.uiuc.edu/Research/namd/alpha/3.0alpha/](http://www.ks.uiuc.edu/Research/namd/alpha/3.0alpha/)\)
* we have not tested multi-GPU or multi-node runs using the containers or native applications. We'll add such tests and examples later
* since the DGX is the only server with GPUs on Hopper, we will need to take extra steps to make sure it is used equitably. Please refrain from reserving more GPUs than you truly need.
* we realize the native \(2.14\) and containerized apps \(2.13, 3.0\) are different versions. 2.13 and 3.0 are the only ones available as containers. Please let us know if you want anything other than 2.14 as a native application.

