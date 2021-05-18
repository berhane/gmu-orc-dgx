---
description: Running GROMACS on Hopper
---

# GROMACS

Here's a sample batch submission file to run one of the examples provided by NVIDIA:

{% embed url="https://gitlab.com/NVHPC/ngc-examples" %}

The sample run is available at `/groups/ORC-VAST/app-tests/gromacs`

Please update **NGPUS**, NTASKS**, NMOPT**

```bash
#!/bin/bash
#SBATCH --partition=gpuq                    # the DGX only belongs in the 'gpu'  partition
#SBATCH --qos=gpu                          # need to select 'gpu' QoS
#SBATCH --job-name=jgromacs-__NGPUS__
#SBATCH --output=%x.%j
#SBATCH --nodes=1
#SBATCH --ntasks=__NTASKS__                # up to 128, but make sute ntasks x cpus-per-task < 128
#SBATCH --cpus-per-task=__NOMPT__          # up to 128; but make sute ntasks x cpus-per-task < 128
#SBATCH --gres=gpu:A100.40gb:__NGPUS__          # up to 8; only request what you need
#SBATCH --mem-per-cpu=3500M                # memory per CORE; total memory is 1 PB (1,000,000 MB)
#SBATCH --export=ALL
#SBATCH --time=0-01:00:00                  # set to 1hr; please choose carefully

set echo
umask 0022

#-----------------------------------------------------
# Download data
#-----------------------------------------------------
# Create a directory on the host to work within
mkdir -p ./work
cd ./work

# Download benchmark data
DATA_SET=water_GMX50_bare
wget -c ftp://ftp.gromacs.org/pub/benchmarks/${DATA_SET}.tar.gz
tar xf ${DATA_SET}.tar.gz

# Change to the benchmark directory
cd ./water-cut1.0_GMX50_bare/1536

# to see ID and state of GPUs assigned
nvidia-smi

#-----------------------------------------------------
# Determine GPU and CPU resources to use
#-----------------------------------------------------
# parse out number of GPUs and CPU cores reserved for your job
env | grep -i slurm
GPU_COUNT=`echo $SLURM_JOB_GPUS | tr "," " " | wc -w`
N_CORES=${SLURM_NTASKS}

# Set OMP_NUM_THREADS
# GROMACS suggests 2-6
# SLURM_CPUS_PER_TASK is set to the value of -c, but only if -c is explicitly set
# please note that ntasks x cpus-per-task <= 128
if [ -n "$SLURM_CPUS_PER_TASK" ]; then
  omp_threads=$SLURM_CPUS_PER_TASK
else
  omp_threads=1
fi
export OMP_NUM_THREADS=$omp_threads


#-----------------------------------------------------
# Set up container
#-----------------------------------------------------
SINGULARITY_BASE=/containers/dgx/Containers
CONTAINER=${SINGULARITY_BASE}/gromacs/gromacs-2020_2.sif

# Singularity will mount the host PWD to /host_pwd in the container
SINGULARITY_RUN="singularity run --nv -B ${PWD}:/host_pwd --pwd /host_pwd"

#-----------------------------------------------------
# Run container
#-----------------------------------------------------
# Prepare benchmark data
${SINGULARITY_RUN} ${CONTAINER} gmx grompp -f pme.mdp

# Run benchmark
${SINGULARITY_RUN} ${CONTAINER} gmx mdrun \
                 -ntmpi ${GPU_COUNT} \
                 -nb gpu \
                 -ntomp ${OMP_NUM_THREADS} \
                 -pin on \
                 -v \
                 -noconfout \
                 -nsteps 5000 \
                 -s topol.tpr
```

