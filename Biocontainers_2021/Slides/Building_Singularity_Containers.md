---
theme: csc-2019
lang: en
---

# Building Singularity Containers {.title}

- There two main ways to build Singularity containers
  - Building interactively using the sandbox mode
  - Building using a definition file
- In both case you start with a base OS image and add
your applications and dependencies

# Base images
- Any existing container (local file or in a registry)
can be used
- Usually you'll want to start with a barebones image with bare essentials
  - This way you eliminate any possible conflicts from the start

  # Base image selection
- Select a base image that best suits your needs
  - E.g. if you have installation instructions for a certain Linux distro, start with that
  - If you need MPI, it's best to start with an image that is close to the host system
    - E.g for Puhti CentOs 7
- If there are no specific reason to choose a specific one, just pick the one you most 
comfortable working with.

# Base image sources
- Base images can be found e.g. 
  - [Sylabs Container Library](https://cloud.sylabs.io/library)
  - [DockerHub](https://hub.docker.com/)
  - [SingularityHub](https://datasets.datalad.org)
    - Note that since early 2021 SingularityHub is no longer maintained and is read-only. 
    For new iamges it's best to use other sources, e.g. DockerHub

# Pros and cons: Sandbox mode
Pros: 
- Easier to try different things
- Any additional files can be copied directly to correct directory
Cons:
- Results not easily reproducible or reusable
  - e.g. to update a software you'll have to start from scratch again
- Always need root access

# Pros and cons: Definition files
Pros:
- Definition files reusable
  - Updating a software often just updateing the version number and re-building
- Some containers can be built with `--remote` option (no root access needed).
Cons:
- Can be a bit cumbersome if you have to try many things
  - Building a conatiner can take a while, and if the failing command is towards 
  the end iteration can take time

# Sandbox mode
- 1. Build a basic container in sandbox mode (`--sandbox`)
    - Uses a folder structure instead of an image file
- 2. Open a shell in the container and install software
  - Depending on base image system, package managers can be used to install 
    libraries and dependencies (`apt install` , `yum install` etc)
  - Installation as per software developer instructions
  
# Sandbox mode
- 3. Build a production image from the sandbox
- 4. (optional) Make a definition file and build a production image from it
  - Mostly necesary if you wish to distribute your container wider
  - Also helps with updating and re-using containers
- Production image can be transferred to e.g. Puhti and run with user rights

# Definition files
- A definition file is a set of intructions that can be used to build a container
- A definition file contains a header and optional sections
  - Only header is compulsory
- A full description can be found in [Singularity documentation](https://sylabs.io/guides/3.8/user-guide/definition_files.html)

# Definition file: Header
- Header is compulsory and needs to be in the beginning of the file
- It defines the `Bootstrap agent`used to build the base OS
  - There are different bootstrap agents for different sources
- It also defines the source to be used 
  - Format depends on the bootstrap agent

# Header example 1
- Example using "library" bootstrap agent and an image from Sylabs Container Library:
Bootstrap: library
From: debian:7

# Header example 3
- Example using "docker" bootstrap agent and an image from DockerHub:
Bootstrap: docker
From: ubuntu:20.04

# Header example 3
- Example using "yum" bootsratp agent and distrubution files from CentOs site:
Bootstrap: yum
OSVersion: 7
MirrorURL: http://mirror.centos.org/centos-%{OSVERSION}/%{OSVERSION}/os/$basearch/
Include: yum

# Optional sections
- There are sections to define the environment variables, installation commands, metadata etc
- All sections are optional and can be in any order
- Section labels start with %
  - Available section labels: %setup, %files, %environment, %post, %runscript, %startscript, %test, %labels, %help
- See tutorial and Singularity documentation for descriptions.

#%setup
- Commands to be run on host after base conatiner base os is running
- Care should be taken as these command are run on host with root rights
- Rarely needed and should be avoided

#%files
- Can be used to add files from the host system to the container
- Syntax is: /path/in/host /path/in/container
%files
  myfile /app/data/myfile

#%environment
- Can be used to set any environment variables, e.g `$PATH`
- Will be stored in file `/environment` inside the container
%environment
  export PATH=/app/bin:$PATH

#%post
- Any commands to be run in the container after base container is up and running
- Typically includes any installation commands
%post
  apt install python
  pip install numpy

#%runscript
- Defines the runscript
- Will be stored in file `/singularity` inside the container

#%startscript
- Similar to the %runscript section
- Execute when run with `singularity start instance example.sif`
- Not that relevant on HPC environment

#%test
- Runs at the very end of the build process to validate the container using a method of your choice
- Can be run with `singularity test example.sif`

#%labels
- Can be used to add metadata, e.g. contact information
- Can be seen with `singularity inspect --labels example.sif`
%labels
Maintainer my.address@example.net

#%help
- Add usage instructions etc
- Can be seen with `singularity run-help example.sif`
%help
  Usage:
  singularity exec myprog --help

# Example definition file
Bootstrap: docker
From: ubuntu:20.04
%runscript
    echo "This is what happens when you run the container..."
%files
    hello_world /usr/bin/hello_world
    hello2  /found/me/hello2
