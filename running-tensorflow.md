---
description: Using containers
---

# Running TensorFlow

These examples demonstrate how to run the TensorFlow from NGC on the DGX using SLURM

{% embed url="https://ngc.nvidia.com/catalog/containers/nvidia:tensorflow" %}

## Single GPU Run

You can find it at `/containers/dgx/Examples/Tensorflow/21.02-tf1-py3/1-single-GPU-example`

```bash
#!/bin/bash
#SBATCH --partition=gpuq                    # the DGX only belongs in the 'gpu'  partition
#SBATCH --qos=gpu                          # need to select 'gpu' QoS
#SBATCH --job-name=single-gpu
#SBATCH --output=jsingle-gpu.%j
#SBATCH --nodes=1
#SBATCH --ntasks=1                # up to 128; note that multithreading is enabled
#SBATCH --gres=gpu:a100:1          # up to 8; only request what you need
#SBATCH --mem-per-cpu=3500M                # memory per CORE; total memory is 1 TB (1,000,000 MB)
#SBATCH --export=ALL
#SBATCH --time=0-01:00:00                  # set to 1hr; please choose carefully

set echo
umask 0027

# to see ID and state of GPUs assigned
nvidia-smi

SINGULARITY_BASE=/containers/dgx/Containers
CONTAINER=${SINGULARITY_BASE}/tensorflow/tensorflow_21.02-tf1-py3.sif
SINGULARITY_RUN="singularity run --nv -B ${PWD}:/host_pwd --pwd /host_pwd"
SCRIPT=test_single_gpu.py

${SINGULARITY_RUN} ${CONTAINER} python ${SCRIPT} | tee ${SCRIPT}.log
```

## Multi-GPU Run

You can find this example at `/containers/dgx/Examples/Tensorflow/21.02-tf1-py3/2-multi-GPU-example`

```bash
#!/bin/bash
#SBATCH --partition=gpuq                    # the DGX only belongs in the 'gpu'  partition
#SBATCH --qos=gpu                          # need to select 'gpu' QoS
#SBATCH --job-name=jmultigpu-2
#SBATCH --output=jmultigpu-2.%j
#SBATCH --nodes=1
#SBATCH --ntasks=8                # up to 128; note that multithreading is enabled
#SBATCH --gres=gpu:a100:2          # up to 8; only request what you need
#SBATCH --mem-per-cpu=3500M                # memory per CORE; total memory is 1 TB (1,000,000 MB)
#SBATCH --export=ALL
#SBATCH --time=0-01:00:00                  # set to 1hr; please choose carefully

set echo
umask 0027

# to see ID and state of GPUs assigned
nvidia-smi

# parse out number of GPUs and CPU cores assigned to your job
env | grep -i slurm
N_GPUS=`echo $SLURM_JOB_GPUS | tr "," " " | wc -w`
N_CORES=${SLURM_NTASKS}

# set up the calculation
SINGULARITY_BASE=/containers/dgx/Containers
CONTAINER=${SINGULARITY_BASE}/tensorflow/tensorflow_21.02-tf1-py3.sif
SINGULARITY_RUN="singularity run --nv -B ${PWD}:/host_pwd --pwd /host_pwd"

# run the calculation
SCRIPT=multigpu_basics.py
${SINGULARITY_RUN} ${CONTAINER} python ${SCRIPT} | tee ${N_GPUS}g-${N_CORES}c-${SCRIPT}.log

SCRIPT=multigpu_cnn.py
${SINGULARITY_RUN} ${CONTAINER} python ${SCRIPT} | tee ${N_GPUS}g-${N_CORES}c-${SCRIPT}.log
```

