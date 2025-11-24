---
theme: csc-2019
lang: en
---

# Introduction to Apptainer containers {.title}

<div class="column">
![](https://mirrors.creativecommons.org/presskit/buttons/88x31/png/by-sa.png)
</div>
<div class="column">
<small>
All materials (c) 2020-2025 by CSC – IT Center for Science Ltd.
This work is licensed under a **Creative Commons Attribution-ShareAlike** 4.0
Unported License, [http://creativecommons.org/licenses/by-sa/4.0/](http://creativecommons.org/licenses/by-sa/4.0/)
</small>
</div>


# The one-slide lecture

- Some software packages on the CSC supercomputers are installed as containers
  - May cause some changes to usage
  - See the instructions in Docs CSC for each software for details
- Containers provide an easy way to install software
  - A single-command installation if a suitable Docker or Apptainer/Singularity container exists


# Containers

- Containers are a way of packaging software and their dependencies (libraries, etc.)
- Popular container engines include Docker, Apptainer (open source fork of Singularity), Shifter, Podman
- Docker probably the most popular overall, but not suitable for HPC environments
- Singularity/Apptainer is most popular in HPC environments


# Apptainer vs. Singularity
- The open source project formerly called Singularity is now called Apptainer (since March 2022)
- Sylabs continues to develop commercially oriented products such as SingularityCE and SingularityPro
- Apptainer 1.0 was forked from Singularity 3.9.5 with some [changes](https://github.com/apptainer/apptainer/releases/tag/v1.0.0)
- Singularity and Apptainer are currently very similar, but they are now developed independently and will probably continue to diverge
- Puhti and Mahti have Apptainer. LUMI has SingularityCE


# Apptainer/Singularity differences
- All SINGULARITY_ and SINGULARITYENV_ variables work for now, but you will get a warning
  - There are currently corresponding APPTAINER_ and APPTAINERENV_ variables, but the naming may change in the future
  - If both SINGULARITY_/SINGULARITYENV_ and APPTAINER_/APPTAINERENV_ are defined for a given variable, APPTAINER values will be used.
- On CSC machines commands `singularity` and `singularity_wrapper` are symlinked to `apptainer` and `apptainer_wrapper` respectively

TLDR: Most older singularity instructions work currently as-is, but this may change as time goes on.


# Containers vs. virtual machines (1/2)

<div style="text-align:center"><img src="./img/containers-fig1.png" /></div>


# Containers vs. virtual machines (2/2)

- Virtual machines can run a totally different OS than the host computer (e.g. Windows on a Linux host or vice versa)
- Containers share the kernel with the host, but they can have their own libraries
  - They can run, e.g., a different Linux distribution than the host


# Benefits of containers: Ease of installation

- Containers are becoming a popular way of distributing software
  - A single-command installation from existing image
  - More portable since all dependencies are included
- Limited root privileges inside the container if the build system supports it
  - Package managers (yum, apt, etc.) can be utilized even when not available on the target system
  - Some containers need full root access in to build


# Benefits of containers: Environment isolation

- Containers use the host system kernel, but they can have their own bins/libs layer
  - Can be a different Linux distribution than the host
  - Can solve some incompatibilities
  - Less likely to be affected by changes in the host system


# Benefits of containers: Enviroment reproducibility

- Environment can be saved as a whole
  - Useful with, e.g., Python, where updating underlying packages (NumPy, etc.) can lead to differences in the behavior  
- Sharing with collaborators is easy (a single file)


# Apptainer in a nutshell

- Containers can be run with user-level rights
  - But: building new containers requires root access or 
  `--fakeroot` option available on the build system
- Minimal performance overhead
- Supports MPI
  - Requires containers tailored to the host system
- Can use host driver stack (Nvidia/CUDA)
  - Add the option `--nv`
- Can import and run Docker containers
  - Running Docker directly would require root privileges


# Using Docker containers with Apptainer

- You can pull and convert a Docker container image with normal user rights:
  - `apptainer pull docker://<address>:<tag>`
  - `apptainer build <image> docker://<address>:<tag>`
- For example:
  - `apptainer pull docker://nvcr.io/nvidia/pytorch:19.10-py3`
- More information in our documentation:
  - [Running Apptainer containers](https://docs.csc.fi/computing/containers/run-existing/)
  - [Creating Apptainer containers](https://docs.csc.fi/computing/containers/creating/)
  - [Using Tykky to create Apptainer containers](https://docs.csc.fi/computing/containers/tykky/)
  - [Apptainer containers on LUMI](https://docs.lumi-supercomputer.eu/software/containers/singularity/)


# Apptainer containers as an installation method

- Apptainer is a good option in cases where the installation is otherwise problematic:
  - Complex installations with many dependencies/files
  - Obsolete dependencies incompatible with the host environment
    - Still needs to be kernel-compatible
  - Native use of Conda has been deprecated on CSC supercomputers
    - Conda environments can be installed inside containers
    - Container wrapper tool [Tykky](https://docs.csc.fi/computing/containers/tykky/) provided for this
- Should be considered even when other methods exist


# Just a random example (FASTX-toolkit)

- Tested installation methods:
  - Native: 47 files, total size 1.9 MB
  - Conda: 27464 files, total size  1.1 GB
  - Apptainer: 1 file, total size 339 MB
- Containers are not the solution for everything, but they do have their uses