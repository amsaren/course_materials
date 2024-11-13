---
theme: csc-2019
lang: en
---

# Building Apptainer container images {.title}

<div class="column">
![](https://mirrors.creativecommons.org/presskit/buttons/88x31/png/by-sa.png)
</div>
<div class="column">
<small>
All materials (c) 2020-2024 by CSC – IT Center for Science Ltd.
This work is licensed under a **Creative Commons Attribution-ShareAlike** 4.0
Unported License, [http://creativecommons.org/licenses/by-sa/4.0/](http://creativecommons.org/licenses/by-sa/4.0/)
</small>
</div>

# Prerequisites

- A system with Apptainer installed
  - can be a virtual machine
  - Many Linux distros have Apptainer available through package management software (apt, yum)

# Privileges

- Building containers requires root access
  - `sudo` and namespaces unavailable in CSC systems due to safety reasons
- Unprivileged building possible with some limitations using [fakeroot](https://apptainer.org/docs/user/latest/fakeroot.html)
  - Available on CSC supercomputers
  - User appears as `root:root` inside the container but has no actual root privileges
- The `build --remote` option has been removed from Apptainer because there is no standard protocol or non-commercial service that supports it

# Building Apptainer Containers

- There are two main ways to build containers
  - Building using a definition file
  - Building interactively using the sandbox mode


# Pros and cons: Definition files

- Pros:
  - Definition files reusable
    - E.g. If your definition file clones the latest version from git, updating a software can 
    typically be done by just re-building 
    - Or: If you e.g. have a conda template definition, you can just change the `conda install` commands
    - Etc
  - Installation steps documented  
- Cons:
  - Can be a bit cumbersome if you have to try many things (e.g. installing missing libraries)


# Pros and cons: Sandbox mode

- Pros: 
  - Easier to try different things
  - Any additional files can be copied directly to correct directory
- Cons:
  - Installation steps not documented
  - Results not easily reproducible or reusable
    - e.g. to update a software you'll have to start from scratch again


# Base images

- In both cases you start with a base OS image and add
your applications and dependencies
- You can start with an existing container (local file or in a registry) or by
specifying a bootstrap agent and source
- Usually you'll want to start with a barebones image with bare essentials
  - This way you can avoid any possible conflicts from the start
  - Also keeps the container image small


# Base image selection

- Select a base image that best suits your needs
  - E.g. if you have installation instructions for a certain Linux distro, start with that
  - If you need MPI, it's best to start with an image that is close to the host system
    - E.g for Puhti and Mahti use a distro based on RHEL8 (e.g. Rocky 8)
    - MPI in the container must be ABI compatible with MPI in host. By choosing similar base distro you minimize the chances of MPI installation problems
- If there are no specific reasons to choose a particular distro, just pick the one you are most 
comfortable working with.


# Base image sources

- Base images can be found e.g. 
  - [DockerHub](https://hub.docker.com/)
  - [Sylabs Cloud Library](https://cloud.sylabs.io/library)
    - Note that `library:` bootstrap agent does not work in Apptainer by default
      - [Restoring pre-Apptainer library behavior](https://apptainer.org/docs/user/latest/endpoint.html#no-default-remote)
  - [SingularityHub](https://datasets.datalad.org/?dir=/shub/)
    - Note that since early 2021 SingularityHub is no longer maintained and is read-only. 
    For new images it's best to use other sources, e.g. DockerHub


# Definition files

- A definition file is a set of intructions that can be used to build a container
- A definition file contains a header and optional sections
  - Only header is compulsory
- A full description can be found in [Apptainer documentation](https://apptainer.org/docs/user/1.0/definition_files.html)
- [CSC GitHub Repo for apptainer definition files](https://github.com/CSCfi/singularity-recipes/)

# Definition file: Header

- Header is compulsory and needs to be in the beginning of the file
- It defines the `bootstrap agent` and `source` used to build the base OS
  - There are different bootstrap agents for different sources
  - Syntax depends on the bootstrap agent


# Header example 1

- Example using "docker" bootstrap agent and an image from DockerHub:

  ```
  Bootstrap: docker
  From: ubuntu:20.04
  ```


# Header example 2

- Example using "yum" bootsrtap agent and distrubution files from CentOs site:

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
  - Standard sections: `%setup, %files, %environment, %post, %runscript, %startscript, %test, %labels, %help`
  - Various `%apps*` sections for [SCIF](https://sci-f.github.io/) Apps support 
- See [Apptainer documentation](https://apptainer.org/docs/user/latest/definition_files.html) for full descriptions.


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
    apt install python3
    pip install numpy
  ```


# %runscript

- Defines the runscript

  ```
  %runscript
    exec myprog "$@"'  # Runs program "myprog" with command line options provided
  ```

- Executed when container run with:

  ```
  apptainer run myprog.sif
  ```

  or

  ```
  ./myprog.sif
  ```


# %startscript

- Similar to the `%runscript` section
- Executed when run with:

  ```
  apptainer start instance example.sif
  ```  

- Mainly used for containers that run services etc. Not that relevant on HPC environment


# %test

- Runs at the very end of the build process to validate the container using a method of your choice
- Can be run with:

  ```
  apptainer test example.sif
  ```


# %labels

- Can be used to add metadata, e.g. contact information
  
   ```
  %labels
    Maintainer my.address@example.net
  ```

- Can be seen with:

  ```
  apptainer inspect --labels example.sif
  ```


# %help

- Add usage instructions etc

  ```
  %help
    Usage:
    apptainer exec myprog.sif myprog --help
  ```

- Can be seen with:
  
  ```
  apptainer run-help example.sif
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

# Building the image

- To build with a definition file:

  ```
  sudo apptainer build myapp.sif myapp.def
  ```

  or:

  ```
  apptainer build --fakeroot myapp.sif myapp.def
  ```
- If `sudo` or namespaces are not available `--fakeroot` is assumed so it can be omitted 

# Sandbox mode 1/4

- Build a basic container in sandbox mode (`--sandbox`)
  - You can start with a simple definition file (just the header) or
  from a bootstrap agent: 

    ```
    apptainer build --fix-perms --sandbox myprog/ docker://ubuntu:22.04
    ```
  - Depending on host file system you may need to specify `--fix-perms`
  - Creates a folder structure instead of an image file
  - You can copy files directly to correct subfolder


# Sandbox mode 2/4

- Open a shell in the container and install software
  - To open in writable mode use option `--writable`

    ```
    apptainer shell --writable myprog
    ```
  
  - Depending on base image system, package managers can be used to install 
    libraries and dependencies (`apt install` , `yum install` etc)
    - If using package managers you may need to specify `--fakeroot`
  - Also `--cleanenv` may be needed 
  - Installation as per software developer instructions
  

# Sandbox mode 3/4

- Build a production image from the sandbox

  ```
  sudo apptainer build myprog.sif myprog
  ```

  - Production image can be run with user rights 


# Sandbox mode 4/4 (optional)

- Make a definition file and re-build a production image from it
  - Helps with updating and re-using containers
  - Also necessary if you wish to distribute your container wider
- If you you plan on doing this, keep track of the commands you use
  - Command `history` can help, but if you try many things, keep track 
  of what worked.


# Best practices 1/2

- Always install packages, programs, data, and files into operating system locations (e.g. not `/home`, `/tmp`, or any other directories that might get commonly binded on).
- Document your container. If your runscript doesn’t supply help, write a `%help` or `%apphelp` section. A good container tells the user how to interact with it.
- If you require any special environment variables to be defined, add them to the `%environment` or `%appenv` sections of the build recipe.


# Best practices 2/2

- Build production containers from a definition file instead of a sandbox that has been manually changed. This ensures the greatest possibility of reproducibility and mitigates the “black box” effect.
