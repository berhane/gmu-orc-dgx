---
description: A list of issues that need to be addressed
---

# Outstanding Issues

## 03-24-2021 -  Modules

* Need Lmod modules to operate seamlessly between Hopper and the DGX
* Need to break modules into
  * Intel-only x86\_64
  * AMD-only x86\_64
  * Intel- and AMD-compatible  
* Modules were generally organized without AMD architecture in mind.

### Fixed

* \_

### Changed

* \_

## 03-24-2021 -  Singularity on Hopper

* Singularity 3.7.0 on Hopper won't run on AMD machines because Spack has optimized it for Intel 
* use Singularity 3.4.2 until we build Singularity 3.7.0 for AMD  

### Fixed

* \_

### Changed

* \_

## 03-29-2021 -  server to compile CUDA-enabled code

* currently need to compile CUDA-enabled code on the DGX itself, or potentially on Argo, after installing CUDA 11/12 because Hopper's login and compute don't have GPUs or NVIDIA drivers.
* May need to install GPUs on the Hopper's login nodes
* Need to streamline and document that workflow

**Fixed**

* \_

**Changed**

* \_

 



