---
layout: page
title: "Tutorial: Bridging Interactive and Batch computing with Containers in HPC environments"
permalink: /resources/containers-in-hpc/
toc: true
---

* TOC
{:toc}

This tutorial explores how containers can bridge the gap between interactive development environments and batch computing systems, enabling seamless workflows from development to production in HPC environment. The examples are tailored toward use in UVA's HPC environment although the concepts apply generally.

We build on these foundational concepts:

- Basic familiarity with command-line interfaces
- Basic understanding of notebooks and the JupyterLab environment
- Understanding of container concepts, see <a href="https://uvads.github.io/container-basics/">Container Basics</a> and <a href="/S4RC/resources/container-registries/">Container Registries</a>

It is assumed that you have access to an HPC system. HPC acess and user accounts are typically tied to an **allocation**, which is a grant of compute resources that allows you to submit and run jobs on the HPC cluster. At UVA, faculty can request allocations through the Research Computing, see <a href="https://www.rc.virginia.edu/userinfo/hpc/">UVA Research Computing</a>. Postdocs, staff and students can be sponsored through a faculty allocation.

## Overview

Containers package application code/executables and all their dependencies needed to run them. They provide lightweight operating system-level virtualization (as opposed to hardware virtualization provided by virtual machines) and offer portability of applications across different environments. Several container projects are specifically targeted at HPC environments, addressing the unique requirements of high-performance computing systems.

When working with code it is helpful to distinguish *interactive* vs *batch (non-interactive)* workloads and *development* vs *production* phases in a software's life cycle.

| | Development | Production |
| --- | --- | --- |
| **Interactive** | Prototyping (e.g. Jupyter notebooks, RStudio) | Data exploration and live analysis (e.g. Jupyter notebooks, RStudio) |
| **Batch** | Testing scripts and analysis at small scale (e.g. scheduled jobs) | full scale analysis (e.g., scheduled jobs) |

**Development Phase**

There is some tension between the fixed nature of container content and the necessary fluidity of program code as it evolves during the development process. A convenient approach is to containerize all dependencies and mount folders with the evolving code base into the container instance at runtime.

Eventually, the version controlled code base should be included in a new container image based on the image with all its dependencies as base layer. This will become the production image.

**Production Phase**

In the production phase, containers are deployed with the finalized code base included in the image. Both interactive and batch jobs in an HPC environment run on resources allocated by a scheduler, which manages compute node access and resource allocation. The containerized application can be executed consistently across different nodes, ensuring reproducibility and portability of the computational workflow.

## Containers in HPC environments

Traditional Docker requires root-level daemon access, which is typically restricted in HPC environments for security reasons. Several container frameworks have been developed to address this limitation:

<div style="display: flex; align-items: center; margin-bottom: 1em;">
<div style="margin-right: 1em;">
<img src="{{ site.baseurl }}/assets/images/containers-in-hpc/apptainer-small.png" alt="Apptainer logo" style="height: 80px;">
</div>
<div style="flex: 1;">
  <a href="https://apptainer.org/">Apptainer</a> (formerly Singularity): Initially developed at Lawrence Berkeley National Laboratory, now managed through its own foundation. The most widely adopted HPC container solution with support for direct container execution.
</div>
</div>

<div style="display: flex; align-items: center; margin-bottom: 1em;">
<div style="margin-right: 1em;">
<img src="{{ site.baseurl }}/assets/images/containers-in-hpc/Podman-logo-small.png" alt="Podman logo" style="height: 80px;">
</div>
<div style="flex: 1;">
  <a href="https://podman.io/">Podman</a>: Developed by Red Hat. A daemonless container engine that provides a Docker-compatible command-line interface, making it easy to transition from Docker workflows. The `podman-hpc` variant is optimized for HPC environments.
</div>
</div>

<div style="display: flex; align-items: center; margin-bottom: 1em;">
<div style="margin-right: 1em;">
<img src="{{ site.baseurl }}/assets/images/containers-in-hpc/shifter-small.png" alt="Shifter logo" style="height: 80px;">
</div>
<div style="flex: 1;">
  <a href="https://github.com/NERSC/shifter">Shifter</a>: Developed at NERSC (National Energy Research Scientific Computing Center). Optimized for large-scale HPC deployments with integration into HPC schedulers and filesystems.
</div>
</div>

<div style="display: flex; align-items: center; margin-bottom: 1em;">
<div style="margin-right: 1em;">
<img src="{{ site.baseurl }}/assets/images/containers-in-hpc/charliecloud-small.png" alt="CharlieCloud logo" style="height: 80px;">
</div>
<div style="flex: 1;">
  <a href="https://charliecloud.io/">CharlieCloud</a>: Developed by Los Alamos National Laboratory. A minimal container runtime with a small footprint, designed for simplicity and ease of deployment.
</div>
</div>

These frameworks are <a href="https://opencontainers.org/">OCI (Open Container Initiative)</a> compatible and can wrap around Docker images at runtime, allowing you to use existing Docker images without modification. This compatibility means you can develop containers using Docker on your local machine and then run them on HPC systems using these user-space frameworks. These frameworks allow users to run containers in HPC environments without requiring administrative privileges, making them suitable for shared computing resources.

**Relevant for Parallel Computing and DL/ML Workloads:**

All of the above frameworks support MPI for parallel computing workloads and provide abstractions that handle GPU hardware access on the host, making them suitable for both traditional HPC workloads and deep learning/machine learning applications that require GPU acceleration.

## Interactive Code Development in JupyterLab on an HPC System

JupyterLab is an interactive development environment that provides a web-based interface for working with notebooks, code, and data. It offers a portal that allows users to select a specific app/code environment for their interactive computing sessions. A code environment is defined by its software packages, which are isolated across different environments. These environments are defined and referred to as **kernels**. Each kernel provides a specific set of software packages and dependencies. This isolation allows you to work with different programming languages, libraries, and tools within the same JupyterLab interface.

![Diagram showing the relationship between HPC systems, JupyterLab, and containerized kernels, illustrating how containers enable interactive development environments on HPC infrastructure]({{ site.baseurl }}/assets/images/containers-in-hpc/HPC-JupyterLab-Containers.png)

You can define your own kernels backed by custom environments, including container images. Here we use it specifically for containerized Python environments, but the concept extends to R, Julia and other supported apps in the Jupyter ecosystem.

JupyterLab searches for kernels in the following order:

1. **User-level directories**: `~/.local/share/jupyter/kernels` (or `~/Library/Jupyter/kernels` on macOS) - kernels specific to your user account
2. **System-wide directories**: `/usr/local/share/jupyter/kernels` or `/usr/share/jupyter/kernels` - kernels available to all users
3. **Environment-specific directories**: `share/jupyter/kernels` within active conda/virtual environments

User-level kernels take precedence over system-wide kernels, allowing you to customize your kernel selection without affecting other users. When creating custom kernels, they are typically placed in `~/.local/share/jupyter/kernels`.

In the following steps, we will explore how to define your own kernels backed by custom container images of your choosing. These containerized kernels can then be used for both interactive work in JupyterLab and non-interactive batch job submissions, providing a consistent environment across your workflow.

### 1. Get the Container Image

### 2. Check that ipykernel is installed inside the image
   
### 3. Creating a new kernel for Pytorch 2.7

#### 3a Set up a directory for the kernel definition
```
KERNEL_DIR=-"~/.local/share/jupyter/kernels/pytorch-2.7"
cd $KERNEL_DIR
```

Inside this directory we will set up two files, `kernel.json` anf `init.sh`.

#### 3b The Kernel spec file 

`kernel.json`
```
{
    "argv": [
        /home/mst3k/.local/share/jupyter/pytorch-2.7/init.sh,
        "-f",
        "{connection_file}"
    ],
    "display_name": "PyTorch 2.7",
    "language": "python",
    "metadata": {
       "debugger": true
    }
} 
```

<ins>Notes:</ins>
- You cannot use env variables and shell expansion for the `argv` arguments. `~` or `$HOME` will not work to indicate the path to the script.
- We use the `init.sh` script to bundle a few commands, see next step. You can change the name of this launch script
- You can customize `display_name` as you wish. That's the name that will show up in the JupyterLab UI.  
- On UVA's HPC system the connection file will be injected at runtime by the JupyterLab instance. You don't need to define that.

**Launch script:** `init.sh`
```
#!/usr/bin/env bash
. /etc/profile.d/00-modulepath.sh
. /etc/profile.d/rci.sh
nvidia-modprobe -u -c=0

module purge
module load apptainer
apptainer exec --nv /home/mst3k/pytorch-2.7.sif python -m ipykernel $@
```

<ins>Notes:</ins>
- Lines 2-4 ensure proper setup in case NVIDIA GPUs will be used
- Note the execution of `python -m ipykernel $@` inside the container, including the `$@` which will shell expand command line args you may want to pass on. (Not used here).

Content of `~/.local/share/jupyter/kernels/

#### 3c Testing the new kernel

## Running a non-interactive Job

## Tips & Tricks

### Turn Your Image Into an Executable

One powerful feature of Apptainer is the ability to turn container images into executable commands. This is particularly useful for bioinformatics tools and other command-line applications that are distributed as Docker images.

**Example: Creating a `samtools` executable**

Many bioinformatics Docker images define their entry point to run a specific tool. For example, a Dockerfile for samtools might look like this:

```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y samtools

ENTRYPOINT ["samtools"]
CMD ["--help"]
```

To turn this Docker image into an executable:

```bash
module load apptainer
apptainer pull samtools-latest.sif docker://staphb/samtools:latest
```

This creates a file named `samtools_latest.sif`. You can then rename it to match the command name:

```bash
mv samtools-latest.sif samtools
chmod +x samtools  # if not already executable
```

Now you can use `samtools` directly as a command:

```bash
samtools view input.bam
samtools sort input.bam -o sorted.bam
samtools index sorted.bam
```

The container will automatically execute the `ENTRYPOINT` or `CMD` defined in the Dockerfile, passing your command-line arguments to the tool inside the container. This approach allows you to use containerized tools as if they were natively installed, while maintaining the reproducibility and isolation benefits of containers.


## References

Kurtzer, G. M., Sochat, V., & Bauer, M. W. (2017). Singularity: Scientific containers for mobility of compute. *PLoS ONE*, 12(5), e0177459. <a href="https://doi.org/10.1371/journal.pone.0177459">https://doi.org/10.1371/journal.pone.0177459</a> (Apptainer, formerly Singularity)

Priedhorsky, R., Randles, T., & Olivier, S. L. (2017). Charliecloud: Unprivileged containers for user-friendly HPC. In *Proceedings of the International Conference for High Performance Computing, Networking, Storage and Analysis* (SC '17), Article 50. Association for Computing Machinery. <a href="https://doi.org/10.1145/3126908.3126925">https://doi.org/10.1145/3126908.3126925</a>

Gerhardt, L., Bhimji, W., Canon, S., Fasel, M., Jacobsen, D., Mustafa, M., ... & Yates, B. (2017). Shifter: Containers for HPC. In *Cray User Group Conference* (CUG '17). <a href="https://cug.org/proceedings/cug2017_proceedings/includes/files/pap115s2-file1.pdf">https://cug.org/proceedings/cug2017_proceedings/includes/files/pap115s2-file1.pdf</a>

Walsh, D. (2023). *Podman in Action*. Manning Publications. <a href="https://www.manning.com/books/podman-in-action">https://www.manning.com/books/podman-in-action</a>

Sun, R., & Siller, K. (2024). HPC Container Management at the University of Virginia. In *Practice and Experience in Advanced Research Computing 2024: Human Powered Computing* (PEARC '24), Article 73. Association for Computing Machinery. <a href="https://doi.org/10.1145/3626203.3670568">https://doi.org/10.1145/3626203.3670568</a>

**Additional Resources:**

- <a href="https://apptainer.org/">Apptainer</a> - User Guide and Documentation. Apptainer a Series of LF Projects LLC.
- <a href="https://podman.io/">Podman</a> - Podman Documentation. Red Hat, Inc.
- <a href="https://github.com/NERSC/shifter">Shifter</a> - Shifter Container System. National Energy Research Scientific Computing Center (NERSC).
- <a href="https://charliecloud.io/">CharlieCloud</a> - CharlieCloud Documentation. Los Alamos National Laboratory.