---
description: Please ask other questions as they would help future users
---

# FAQ

## General

### What kinds of workloads is the DGX intended for?

The DGX is intended mainly for GPU jobs, particularly those which can take advantage of 

* the NVIDIA Tesla A100 GPU's tensor core engine and large memory \(40GB/GPU\) 
* the NVLink and NVSwitch communications enabling multi-GPU jobs across as many as 8 GPUs

### How can I use the DGX responsibly?

Since the DGX is currently the only server with GPUs on Hopper, users cooperation in ensuring its equitable usage is greatly appreciated.

Generally, the following guidelines will ensure the GPUs equitable usage.

* Benchmark your code to determine the best mix of CPU cores and GPUs to use. As you do, here are a few things to keep in mind:
  * Some codes do not scale well on multiple GPUs and running them on more than one GPU would be wasteful. 
  * Some utilize CPU cores and GPUs actively while others offload all the work to the GPU and only need one CPU core for system tasks
  * Some codes run as well on older generation GPUs as they do on the DGX's A100 GPUs. In that case, you should consider running them on Argo. Please note that your /home, /projects, and /scratch directories are mounted on Argo, Hopper and the DGX to ensure you can run calculations on any of them seamlessly
* Only request resources you need
* If you are using the DGX in interactive mode, please log out and release the resources as soon as you are done
* Use the DGX for GPU jobs only.

### What is new about the DGX compared to GPU nodes on Argo?

From a hardware standpoint, the DGX has the latest generation GPUs \(NVIDIA Tesla A100s\) compared to slightly older generations \(V100, K80\) on Argo. It also has 8 GPUs interconnected with NVLink and NVSwitch fabric compared to 4-8-way GPUs communicating over PCIe bus. So, it should allow you to run GPU jobs faster and with greater parallelization. It also has powerful AMD Rome CPUs, 1TB of memory, and much faster memory bandwidth.

From a software standpoint, it will run native and containerized applications compiled with the latest NVIDIA libraries and drivers.

### What's the best way to get help?

Please submit a support ticket by emailing orchelp@gmu.edu if you need help. Please note that the DGX is currently in the testing phase an we aim to get it ready for the production phase using your input.

## Containers

### Do I have to use containers on the DGX?

No, you do not. If you have a native application that runs on A100 GPUs properly, you can run it as is. We emphasize containers here primarily because they save you/us from the pain of compiling GPU-enabled code.

### Is there a performance penalty for using containers?

Containers mostly run as efficiently as native applications, but the overhead of running containers can result in slightly lower performance \(&lt;5%\)

### Do I really need an NGC account?

Not necessarily. You would not need an NGC account if 

* you have no intention of using containers, OR
* you only plan to use containers we provide centrally OR
* you plan to get containers from sources other than NGC

You would only need an NGC account if you plan to pull Docker containers from NVIDIA's NGC catalog. However, there are a lot of reasons to sign up for an NGC account because it provides

* access to containers that are optimized for NVIDIA GPUs
* lots of documentation on running them
* lots of datasets, workflows for HPC, data science, machine learning, deep learning, ... etc

### Can I run Docker containers?

Yes, you can run Docker containers as long as you convert them to Singularity format.

However, you can not build the Docker containers on Hopper. The best approach is to 

* build the Docker containers on your local computer or virtual machine
* push the Docker container to DockerHub or any other container registry
* pull and convert it to Singurity on Hopper
* run the Singularity container

### Where is the best place to store containers?

Containers can get large and you also need fast access to them when you are running your calculations. To enable the efficient use of containers, we have designated `/containers/dgx/UserContainers/$USER` as a place where users can build and store their containers.

You can reference this space using the environmental variable `$SINGULARITY_BASE_USER`

The shared containers are at `/containers/dgx/Containers` and you can reference that directory using the  environmental variable `$SINGULARITY_BASE`

