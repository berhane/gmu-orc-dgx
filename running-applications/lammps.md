---
description: Running LAMMPS using as a container and native application
---

# LAMMPS

## As a container

One can use the highly-optimized LAMMPS containers from NGC \([https://ngc.nvidia.com/catalog/containers/nvidia:lammps](https://ngc.nvidia.com/catalog/containers/nvidia:lammps)\) for single or multiple GPU runs as follows. 

```
$ tree -rf /groups/ORC-VAST/app-tests/lammps/dgx/containerized

├── /groups/ORC-VAST/app-tests/lammps/dgx/containerized/10Feb2021
└── /groups/ORC-VAST/app-tests/lammps/dgx/containerized/29Oct2020
```

A typical batch submission file would like this:

{% code title="$ run.slurm" %}
```bash
#!/bin/bash
#SBATCH --partition=gpuq                    # the DGX only belongs in the 'gpu'  partition
#SBATCH --qos=gpu                          # need to select 'gpu' QoS
#SBATCH --job-name=jlammps
#SBATCH --output=%x.%j
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1                # up to 128, but make sute ntasks x cpus-per-task < 128
#SBATCH --cpus-per-task=1          # up to 128; but make sute ntasks x cpus-per-task < 128
#SBATCH --gres=gpu:a100:1          # up to 8; only request what you need
#SBATCH --mem-per-cpu=35000M                # memory per CORE; total memory is 1 PB (1,000,000 MB)
# SBATCH --mail-user=user@inst.edu
# SBATCH --mail-type=ALL
#SBATCH --export=ALL
#SBATCH --time=0-04:00:00                  # set to 1hr; please choose carefully

set echo
#-----------------------------------------------------
# Example run from NVIDIA NGC
# https://ngc.nvidia.com/catalog/containers/hpc:lammps
# Please feel free to download and run it as follows
#-----------------------------------------------------

#-----------------------------------------------------
# Determine GPU and CPU resources to use
#-----------------------------------------------------
# parse out number of GPUs and CPU cores reserved for your job
env | grep -i slurm
GPU_COUNT=`echo $SLURM_JOB_GPUS | tr "," " " | wc -w`
N_CORES=${SLURM_NTASKS}

# Set OMP_NUM_THREADS
# please note that ntasks x cpus-per-task <= 128
if [ -n "$SLURM_CPUS_PER_TASK" ]; then
  OMP_THREADS=$SLURM_CPUS_PER_TASK
else
  OMP_THREADS=1
fi
export OMP_NUM_THREADS=$OMP_THREADS

#-----------------------------------------------------
# Set up MPI launching
#-----------------------------------------------------
# If parallel, launch with MPI
if [[ "${GPU_COUNT}" > 1 ]] || [[ ${SLURM_NTASKS} > 1 ]]; then
  MPI_LAUNCH="prun"
else
  MPI_LAUNCH=""
fi

#-----------------------------------------------------
# Set up container
#-----------------------------------------------------
SINGULARITY_BASE=/containers/dgx/Containers
CONTAINER=${SINGULARITY_BASE}/lammps/lammps_10Feb2021.sif

# Singularity will mount the host PWD to /host_pwd in the container
SINGULARITY_RUN="singularity run --nv -B ${PWD}:/host_pwd --pwd /host_pwd"

#-----------------------------------------------------
# Run container
#-----------------------------------------------------
# Define input file and run
LMP_INPUT=in.lj.txt
LMP_OUTPUT= log-${GPU_COUNT}gpus-${SLURM_NTASKS}cores-${OMP_NUM_THREADS}thr_percore.lammps
echo "Running Lennard Jones 16x8x16 example on ${GPU_COUNT} GPUS..."
${MPI_LAUNCH} ${SINGULARITY_RUN} ${CONTAINER} lmp \
                 -k on g ${GPU_COUNT} \
                 -sf kk \
                 -pk kokkos cuda/aware on neigh full comm device binsize 2.8 \
                 -var x 16 \
                 -var y 8 \
                 -var z 16 \
                 -in ${LMP_INPUT} \
                 -log ${LMP_OUTPUT}

```
{% endcode %}

You would need this input file to run this test. You can copy 

```bash
$ cat in.lj.txt

# 3d Lennard-Jones melt

variable        x index 1
variable        y index 1
variable        z index 1

variable        xx equal 20*$x
variable        yy equal 20*$y
variable        zz equal 20*$z

units           lj
atom_style      atomic

lattice         fcc 0.8442
region          box block 0 ${xx} 0 ${yy} 0 ${zz}
create_box      1 box
create_atoms    1 box
mass            1 1.0

velocity        all create 1.44 87287 loop geom

pair_style      lj/cut 2.5
pair_coeff      1 1 1.0 1.0 2.5

neighbor        0.3 bin
neigh_modify    delay 0 every 20 check no

fix             1 all nve

run             100
```

