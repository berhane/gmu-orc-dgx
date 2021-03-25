---
description: Having an NVIDIA's GPU Cloud Registry account has a lot of benefits
---

# Using NGC

While an NGC account is not necessary to use the DGX, it does provide a lot of capabilities.

## Getting an NVIDIA GPU Cloud Account:

The NVIDIA GPU Cloud \(NGC\) provides simple access to GPU-optimized software for deep learning, machine learning and high-performance computing \(HPC\). An NGC account grants you access to these tools as well as the ability to set up a private registry to manage your customized software.

Sign up for an NGC account at [https://ngc.nvidia.com](https://ngc.nvidia.com)

## Accessing the NGC registry:

Once you have an NGC account, you are able to access containers and instructions on running them using Singularity or Docker. The access is provided through the NGC web interface or using the NGC CLI.

Because we use Singularity containers to run most applications on the server, you need to pull down Docker containers from NVIDIA GPU Cloud \(NGC\) and convert them to Singularity format \(SIF\) for execution. This can be done either through the NGC web portal or the NGC CLI.

### Using the NGC Web Portal:

The list of available Containers can be found [here](https://ngc.nvidia.com/catalog/containers). You can search through the registry and download and run the containers as described there.

### Using NGC CLI to interact with NGC:

Alternatively, you can access the same information through a command line \(CLI\) environment. Once you’ve created an NGC account and have logged in, you’ll be able to generate the [NGC API key](https://docs.nvidia.com/ngc/#generating-api-key) required to pull all NGC containers. You can find more information describing how to obtain and use your NGC API key [here](https://docs.nvidia.com/ngc/#generating-api-key).

### Setting up NGC Credentials on Hopper:

{% hint style="info" %}
Users are currently set up to use our ORC's NGC account to pull down containers from NGC and interact with them. The steps below are provided if you want to sign up for your own NGC account and set up your personal registry.
{% endhint %}

To access your own NGC account, you must set the NGC container registry authentication credentials.  
This is easily accomplished by setting and adding the following environment variables in your `~/.bashrc`:

```bash
$ export SINGULARITY_DOCKER_USERNAME='$oauthtoken' 
$ export SINGULARITY_DOCKER_PASSWORD=<NGC API key>
```

## NGC commands:

To interact with NGC using the CLI, you can run the following command to see the possible options:

```bash
$ ngc registry image –help

image: 
       {info,list,pull,push,remove,update} 

info    Display information about an image
           repository or tagged image. 
list    List container images accessible by the user. 
pull    Pull a container image from the NGC image registry. If no tag is provided, 'latest' is assumed. 
push     Push a container image to the NGC image registry. If no tag is provided, 'latest' is assumed. 
remove    Remove an image repository or specific image with an image tag. 
update     Update image repository metadata
```

```bash
$ ngc registry image list 
$ ngc registry image list | grep -i <container_name> 
$ ngc registry image info nvcr.io/<container_name>:<containter_tag>
```

This example demonstrates how to search and pull down a GROMACS image:

```bash
$ ngc registry image list|grep -i gromacs 

| GROMACS | hpc/gromac | 2020.2 | 275.47 MB | Sep 24, | unlocked|
```

```bash
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
```

```bash
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

To look through your NGC setup and diagnose any issues with your account, you can run

```bash
  $ ngc diag all
```

## Pulling Docker Containers and Building Singularity containers:

Once you select a Docker container to use, you can pull it down and convert it to Singularity image format with the following command.

```bash
$ singularity build <container_name>_<container_version/tag>.sif docker://nvcr.io/<hpc>/<container_name><container_version/tag>
```

Here is an example for preparing a GROMACS Singularity container:

```bash
$ singularity build gromacs-2020_2.sif docker://nvcr.io/hpc/gromacs:2020.2
```

Please note that we have adapted the following convention on naming Singularity image files.

* we use SIF instead of SIMG for the file extension 
* we name containers as `<container_name>_<container_version/tag>.sif` 

Also note that you can pull the containers from NGC, DockerHub or any other source, but we encourage using ones from the NGC registry if one is available.



