---
topic: singularity
title: Tutorial - Creating singularity containers from definition files
---

# Creating singularity containers from definition files

In this tutorial we create a Singularity container from a definition file.

There are many ways to build Singularity containers. While using definition files 
many not be the most intuitive one, it has many benefits: Definition files are easily 
upgradeabla and reusable. They are prefereable way to to distribute containers (as opposed
to "black box" image files) and are necessary when uploadin your images to most repositories.

In most case it is preferable to build containers in a system where you have root access.
This could be your own machine or a suitable virtual machine. This kind of setup gives you 
the most options, and makes troubleshooting the easiest.

In this tutorial, however, we will be usig the [SyLabs Remote Builder](https://cloud.sylabs.io/builder) 
service instead. This way of building has some limitations, but can be run without root 
access on host system, as long as you can log into the Syslabs service.

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

## Including files

When building an image on a local computer, you can include files form hos file system using `%files` header:
```text
%files
  myfile.txt /some/path/myfile.txt
```
When using Remote Builder this won't work, so if you need to include files make them available from net and 
download the from inside the container using `git`.`wget`,`curl` on similar in the `%post` section. Note that
these tools are often not included in the base image, so you will need to install them first.

Luckily MACS3 does not need any extra files included, so we can skipt this.

Unneeded sections can be omitted or left empty.

## Installing software

All commands that should be run after the base image is running, including typically any installation commands, are 
included in `%post`

As these commands are executed inside the container, we usually have access to package manager softwares for each 
distribution, e.g. `apt` for Ubunbtu (depends on the base image used). All commands are executed with root rights by 
default, so there is no need to prefix the commands with `sudo`.

Many base images are very barebones to keep the images small. For example the on we chose does not include Python, so 
we will start by installing it.

Add the following lines to the definition file:

```text
%post
  # Install python
  apt update
  apt install python3 -y
  apt install pip -y
  ln -s /usr/bin/python3 /usr/bin/python
```
Indentation is optional, but improves readability. Comments will make the definition file easier to read for others
(and also for yourself after some time has passed).

Easiest way to install MACS3 in by using Pypi:
```text
  # Install MACS3
  pip install MACS3
```

Since MACS3 is still under development, we might prefer to install from source instead. For this we also need to install git
and python3-devel packages.
```text
  # Install MACS3
  apt install python3-devel -y
  apt install git -y
  git clone https://github.com/macs3-project/MACS.git -recurse-submodules
  cd MACS
  pip install -r requirements.txt
  python setup.py install
```


## Setting up the environment

Any commands that set up the environment go to the `%environment` section.

The default `$PATH` for a Singularity environment is
```text
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

If your installation goes anywhere else, you should adjust the `$PATH`,

For example if we had chosen to install MACS3 in non-standard location in the last section:

```text
python setup.py install --prefix=/app/macs
```

We would need to add:
```text
%environment

  export PATH=/app/macs/bin:$PATH
  export PYTHONPATH=/app/macs/lib/python3.8/site-packages
```
If we installed with pip or omitted the `--prefix` option we can again omit this section or leave it empty.


# Adding a runscript



```tetx
%runscript

  exec echo "$@"
```

## Adding information about the container

If you plan to distribute the container, it is advisable to add some information about it into the container
itself. this information can be checked with command:

```text
%labels
  Maintainer Ari-Matti Saren, CSC
  Version v1.0

%help
  This is a a container for MACS3 software
  https://github.com/macs3-project/MACS
  MACS3 is distributed under BSD 3-Clause License
```






```text
```
```bash
```