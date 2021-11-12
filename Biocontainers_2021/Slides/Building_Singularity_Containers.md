---
theme: csc-2019
lang: en
---

# Building Singularity Containers {.title}

<div class="column">
![](https://mirrors.creativecommons.org/presskit/buttons/88x31/png/by-sa.png)
</div>
<div class="column">
<small>
All material (C) 2020-2021 by CSC -IT Center for Science Ltd.
This work is licensed under a **Creative Commons Attribution-ShareAlike** 4.0
Unported License, [http://creativecommons.org/licenses/by-sa/4.0/](http://creativecommons.org/licenses/by-sa/4.0/)
</small>
</div>

# Building Singularity Containers
- There two main ways to build Singularity containers
  - Building using a definition file
  - Building interactively using the sandbox mode

# Pros and cons: Definition files
- Pros:
  - Definition files reusable
    - E.g. If your definition file clones the latest version from git, updating a software can 
    typically be done by just re-building 
    - Or: If you e.g. have a conda template definition, you can just change the `conda install` commands
    - Etc
  - Most containers can be built with `--remote` option (no root access needed).
- Cons:
  - Can be a bit cumbersome if you have to try many things (e.g. installing missing libraries)
   
# Pros and cons: Sandbox mode
- Pros: 
  - Easier to try different things
  - Any additional files can be copied directly to correct directory
- Cons:
  - Results not easily reproducible or reusable
    - e.g. to update a software you'll have to start from scratch again
  - Always needs root access

# Base images
- In both cases you start with a base OS image and add
your applications and dependencies
- You start with an existing container (local file or in a registry) or by
specifying a bootstrap agent and source
- Usually you'll want to start with a barebones image with bare essentials
  - This way you can avoid any possible conflicts from the start

# Base image selection
- Select a base image that best suits your needs
  - E.g. if you have installation instructions for a certain Linux distro, start with that
  - If you need MPI, it's best to start with an image that is close to the host system
    - E.g for Puhti use CentOs 7
- If there are no specific reasons to choose a particular distro, just pick the one you are most 
comfortable working with.

# Base image sources
- Base images can be found e.g. 
  - [Sylabs Container Library](https://cloud.sylabs.io/library)
  - [DockerHub](https://hub.docker.com/)
  - [SingularityHub](https://datasets.datalad.org)
    - Note that since early 2021 SingularityHub is no longer maintained and is read-only. 
    For new images it's best to use other sources, e.g. DockerHub

# Definition files
- A definition file is a set of intructions that can be used to build a container
- A definition file contains a header and optional sections
  - Only header is compulsory
- A full description can be found in [Singularity documentation](https://sylabs.io/guides/3.8/user-guide/definition_files.html)

# Definition file: Header
- Header is compulsory and needs to be in the beginning of the file
- It defines the `Bootstrap agent` and `source` used to build the base OS
  - There are different bootstrap agents for different sources
  - Syntax depends on the bootstrap agent

# Header example 1
- Example using "library" bootstrap agent and an image from Sylabs Container Library:

  ```
  Bootstrap: library
  From: debian:7
  ```

# Header example 2
- Example using "docker" bootstrap agent and an image from DockerHub:

  ```
  Bootstrap: docker
  From: ubuntu:20.04
  ```

# Header example 3
- Example using "yum" bootsratp agent and distrubution files from CentOs site:

  ```
  Bootstrap: yum
  OSVersion: 7
  MirrorURL: http://mirror.centos.org/centos-%{OSVERSION}/%{OSVERSION}/os/$basearch/
  Include: yum
  ```

# Optional sections
- There are sections to define the environment variables, installation commands, metadata etc
- All sections are optional and can be in any order
- Section labels start with %
  - Available section labels: %setup, %files, %environment, %post, %runscript, %startscript, %test, %labels, %help
- See Singularity documentation for full descriptions.

# %setup
- Commands to be run `in the host` after the base container is running
- Care should be taken as these command are run with root rights
  - Check especially if you got the definition file from the net
- Rarely needed and should be avoided

# %files
- Can be used to add files from the host system to the container
- Syntax is: `/path/in/host /path/in/container`

  ```
  %files
    myfile /app/data/myfile
  ```

# %environment
- Can be used to set any environment variables, e.g `$PATH`
- Will be stored in file `/environment` inside the container

  ```
  %environment
    export PATH=/app/bin:$PATH
  ```

# %post
- Any commands to be run `in the container` after base container is running
- Typically includes any installation commands
  - Note that base images typically don't include any programming languages, compilers etc, 
  so start by installing what you need.

  ```
  %post
    apt install python
    pip install numpy
  ```

# %runscript
- Defines the runscript
- Will be stored in file `/singularity` inside the container
- Executed when container run with:
  ```
  singularity run myprog.sif
  ```
  or
  ```
  ./myprog.sif
  ```

# %startscript
- Similar to the `%runscript` section
- Executed when run with:
  ```
  singularity start instance example.sif
  ```  
- Mainly used for containers that run services etc. Not that relevant on HPC environment

# %test
- Runs at the very end of the build process to validate the container using a method of your choice
- Can be run with:
  ```
  singularity test example.sif
  ```

# %labels
- Can be used to add metadata, e.g. contact information
  
   ```
  %labels
    Maintainer my.address@example.net
  ```
- Can be seen with:
  ```
  singularity inspect --labels example.sif
  ```

# %help
- Add usage instructions etc

  ```
  %help
    Usage:
    singularity exec myprog.sif myprog --help
  ```
- Can be seen with:
  ```
  singularity run-help example.sif
  ```

# Example definition file
```
Bootstrap: docker
From: ubuntu:20.04
%runscript
  echo "This is what happens when you run the container..."
%files
  hello_world /usr/bin/hello_world
  hello2  /found/me/hello2
```

# Sandbox mode 1/4
- Build a basic container in sandbox mode (`--sandbox`)
  - You can start with a simple definition file (just the header) or
  from a bootstrap agent: 
    ```
    sudo singularity build --sandbox myprog library://centos:7.7
    ```
  - Creates a folder structure instead of an image file
  - You can copy files directly to correct subfolder

# Sandbox mode 2/4
- Open a shell in the container and install software
  - To open in writable mode use option `--writable` 
    ```
    sudo singularity shell --writable myprog
    ```
  - Depending on base image system, package managers can be used to install 
    libraries and dependencies (`apt install` , `yum install` etc)
  - Installation as per software developer instructions
  
# Sandbox mode 3/4
- Build a production image from the sandbox
  ```
  sudo singuluraity build myprog.sif myprog
  ```

# Sandbox mode 4/4 (optional)
- Make a definition file and build a production image from it
  - Helps with updating and re-using containers
  - Also necesary if you wish to distribute your container wider
- Production image can be transferred to e.g. Puhti and run with user rights
