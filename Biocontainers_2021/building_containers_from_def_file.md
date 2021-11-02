---
topic: singularity
title: Tutorial - Creating singularity containers from definition files
---

# Creating singularity containers from definition files

In this tutorial we create a Singularity container from a definition file.

In most case it is preferable to build containers in a system where you have root access.
This could be your own machine or a suitable virtual machine. This kind of setup gives you 
the most options and makes troubleshooting the easiest.

In this tutorial, however, we will be usig the [SyLabs Remote Builder](https://cloud.sylabs.io/builder) 
service instad. This way of building has some limitations, but cam be run without root 
access on host system as long as you can log into the Syslabs service.

We will be building a container for [MACS3](https://github.com/macs3-project/MACS) software.

## Preliminaries

Go to Sylabs [Singularity Container Services](https://cloud.sylabs.io/home) page and sign in.
You should be able to use your Google, GitHub, GitLab or Microsoft account to log in.

Go to [Account Management](https://cloud.sylabs.io/auth) and create a new access token.

Save the token, as we will need it later in the tutorial and it can not be displayed again
once you close the browser window.

## Create a definition file

Log into Puhti.

Create an empty definition file. You can use a text editor of your choice, e.g. `nano`:
```bash
nano macs3.def
```
💬 In nano you can use `ctrl + o` to save and `ctrl + x` to exit.


## Selecting the base image

You typically start by selecting a suitable base image. This will have the basic operating system,
file system etc for your image. These can be selected from e.g Singularity conatiner library, Docker 
Hub or Singularity Hub. The possible sources and options are discussed in more detail in the lecture.

Selection of base image depends on the software. If, for example, the software developer provide 
installation instructions for a certain version of Ubuntu, it's probably easiest to start with that.

If you plan to build an image with MPI support, it is easiesto start with something close to the host 
system. In case of Puhti this would be CentOs 7.

In this case we are installing a Python software, so choice of operating systems ois not that critical. 
In this case we'll start with Ubuntu 20.04 definition from Docker Hub.

Add the following lines to the definition file:

```text
Bootstrap: docker
From: ubuntu: 20.04
```




```text
```
```bash
```