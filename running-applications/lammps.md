---
description: Running LAMMPS using as a container and native application
---

# LAMMPS

## Running as a container

One can use the highly-optimized LAMMPS containers from NGC \([https://ngc.nvidia.com/catalog/containers/nvidia:lammps](https://ngc.nvidia.com/catalog/containers/nvidia:lammps)\) for single- or multi-GPU runs as follows. 

The containers can currently be found at `$SINGULARITY_BASE/containers/dgx/Containers/lammps`

The sample tests are at all located at `/groups/ORC-VAST/app-tests/lammps`.

```
$ tree -rf /groups/ORC-VAST/app-tests/lammps/dgx/containerized

├── /groups/ORC-VAST/app-tests/lammps/dgx/containerized/10Feb2021
└── /groups/ORC-VAST/app-tests/lammps/dgx/containerized/29Oct2020
```

### Batch submission file

A typical batch submission file \(named `run.slurm` \)  would like this:

{% code title="run.slurm" %}
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

### Input files

You would need this input file to run this test. You can copy 

{% code title="in.lj.tzt" %}
```bash
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
{% endcode %}

### Benchmarks

For this particular example, the benchmarks indicate that the code scales well up 4 GPUs. Also, the performance of the latest container \(20Feb2021\) is marginally better than the previous one \(29Oct2020\).  

#### 29Oct2020 

```bash
$ grep -i performance *lammps

log-1gpus-1cores.lammps:Performance: 4304.617 tau/day, 9.964 timesteps/s 
log-2gpus-2cores.lammps:Performance: 6647.065 tau/day, 15.387 timesteps/s 
log-4gpus-4cores.lammps:Performance: 10345.111 tau/day, 23.947 timesteps/s 
log-8gpus-8cores.lammps:Performance: 10983.890 tau/day, 25.426 timesteps/s
```

#### 10Feb2021

```bash
$ grep -i performance *lammps

log-1gpus-1cores.lammps:Performance: 4484.309 tau/day, 10.380 timesteps/s
log-2gpus-2cores.lammps:Performance: 6857.607 tau/day, 15.874 timesteps/s
log-4gpus-4cores.lammps:Performance: 10636.647 tau/day, 24.622 timesteps/s
log-8gpus-8cores.lammps:Performance: 12251.451 tau/day, 28.360 timesteps/s
```

#### Comparison with CPU-only Runs

It is always informative to see how much GPU acceleration speeds up calculations. For that purpose, we compared the above benchmarks with those run on nodes with CPUs only.

#### Intel20-IMPI20 version

```bash
log-1nodes-48cores-1thr_per_core.lammps:Performance: 499.450 tau/day, 1.156 timesteps/s
log-2nodes-192cores-1thr_per_core.lammps:Performance: 1908.361 tau/day, 4.418 timesteps/s 
log-4nodes-432cores-1thr_per_core.lammps:Performance: 4327.999 tau/day, 10.019 timesteps/s 
```

#### GNU9-OpenMPI4 

```bash
$ grep -i performance log-gpus-*

log-1node-1thr_per_core.lammps:Performance: 479.698 tau/day, 1.110 timesteps/s
log-4nodes-1thr_per_core.lammps:Performance: 1824.944 tau/day, 4.224 timesteps/s
log-10nodes-480cores-1thr_per_core.lammps:Performance: 4622.468 tau/day, 10.700 timesteps/s
```

### Conclusions

* GPU-optimized LAMMPS container runs very well on our DGX A100
* The GPU code scales well with the number of GPUs used, but it will depend heavily on the size of the simulation
* The two GPU-accelerated containers we tested perform about the same
* 1 NVIDIA A100 GPU performs as well as 9-10 nodes \(dual Intel Cascade Lake CPUs with 48 cores per server\) combined
* Native applications built using the GNU9+OpenMPI4 and Intel20+IMPI20 perform equally well. However, the way the jobs are launched is slightly different, so users are encouraged to see the examples at `/groups/ORC-VAST/app-tests/lammps`.

```bash
$ tree -d /groups/ORC-VAST/app-tests/lammps

/groups/ORC-VAST/app-tests/lammps
├── dgx
│   └── containerized
│       ├── 10Feb2021
│       └── 29Oct2020
└── hopper
    ├── containerized
    │   └── 29Oct2020
    │       ├── multinode
    │       │   └── other
    │       ├── single-node-hybrid
    │       ├── single-node-mpi
    │       └── single-node-omp
    └── native
        └── 21Jul2020
            ├── gnu9-openmpi4
            │   ├── large-example
            │   ├── multi-node
            │   ├── single-node-mpi
            └── intel20-impi20
                ├── large-example
                ├── multi-node
                └── single-node-mpi

```

