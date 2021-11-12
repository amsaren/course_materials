---
topic: singularity
title: Tutorial - Creating singularity containers from definition files
---

# Creating Singularity containers from definition files

In this tutorial we create a Singularity container from a definition file.

There are many ways to build Singularity containers. While writing and using definition files 
many not be the most intuitive one, it has many benefits: Definition files are easily 
upgradeabla and reusable. They are the prefereable way to to distribute containers (as opposed
to "black box" image files) and are necessary when uploadin your images to most repositories.

In most cases it is preferable to build containers in a system where you have root access.
This could be your own machine or a suitable virtual machine. This kind of setup gives you 
the most options, and makes troubleshooting the easiest.

In this tutorial, however, we will be using the [Sylabs Remote Builder](https://cloud.sylabs.io/builder) 
service instead. This way of building has some limitations, but can be run without root 
access on host system, as long as you can log into the Sylabs service.

We will be building a container for [MACS3](https://github.com/macs3-project/MACS) software.


## 1. Remote Builder

### 1.1 Create an Access Token and login

Go to SyLabs [Singularity Container Services](https://cloud.sylabs.io/home) page and sign in.
You should be able to use your Google, GitHub, GitLab or Microsoft account to log in.

Go to [Account Management](https://cloud.sylabs.io/auth) and create a new Access Token.

Save the token, as it can not be displayed again once you close the browser window.

Log into Puhti.

You will first need to authenticate yourself with the Remote Builder site.

Give this command in Puhti command line: 
```bash
singularity remote login
```
When prompted, paste the Access Token you created earlier and press `enter`.

💬The Access Token is saved in `$HOME/.singularity/remote.yaml`. Login is valid as 
long as the file exists and the Access Token is valid, so you don't need to login every time.

### 1.2 Building an image

Let's try building an image.

Download an example definition file:
```bash
wget https://raw.githubusercontent.com/amsaren/course_materials/main/Biocontainers_2021/Files/tutorial-1.def
```
The basic command to use on a system where you have root access would be:
```bash
sudo singularity build test.sif tutorial-1.def
```
This will not work on Puhti, so we will use [Sylabs Remote Builder](https://cloud.sylabs.io/builder) 
service.

The build command is as above, but we omit `sudo` and add option `--remote`.
```bash
singularity build --remote test.sif tutorial-1.def
```
Alternatively you can upload or paste the definition file to the web interface at Sylabs,
and then use `singularity pull` to download the image.

Finally you can test your new image:
```bash
singularity run test.sif
```

## 2. Creating a definition file

Create an empty definition file. You can use a text editor of your choice, e.g. `nano`:
```bash
nano macs3.def
```
💬 In nano you can use `ctrl + o` to save and `ctrl + x` to exit.

A definition file will have a header defining the base image, and number of sections defining
different aspects of the container. All sections are optional. We will discuss some of the most 
used options here. Please see the [Singularity documentation](https://sylabs.io/guides/3.8/user-guide/definition_files.html) 
for a full list and detailed descriptions.

### 2.1 Selecting the base image

You typically start by selecting a suitable base image. This will have the basic operating system,
file system etc for your image. These can be selected from e.g Singularity container library, Docker 
Hub or Singularity Hub. The possible sources and options are discussed in more detail in the lecture.

Selection of base image depends on the software. If, for example, the software developer provide 
installation instructions for a certain Linux distribution, it's probably easiest to start with that.

If you plan to build an image with MPI support, it is easiesto start with something close to the host 
system. In case of Puhti this would be CentOs 7.

In this case we are installing a Python software, so choice of operating systems is not that critical. 
In this case we'll start with Ubuntu 20.04 definition from Docker Hub.

Add the following lines to the definition file:

```text
Bootstrap: docker
From: ubuntu: 20.04
```

### 2.2 Including files

When building an image on a local computer, you can include files form host file system. These are defined
in the `%files` section. The format is `/path/in/host /path/in/container`, so to include file `myfile.txt`
from the current directory to directory `/some/path` inside the container, you would add:
```text
%files
  myfile.txt /some/path/myfile.txt
```
When using Remote Builder this won't work (local files are not available at the remote build server), so if 
you need to include files make them available from net and download the from inside the container using 
`git`, `wget`, `curl` or similar in the `%post` section. 

💬These tools are often not included in the base image, so you will need to install them first.

Including files from the local system can be considered more secure, but downloadin everything will allow
other people to build the container without needing files only available in your system.

MACS3 does not need any extra files, so in this case we can just omit the whole section.


### 2.3 Installing software

All commands that should be run inside the container after the base image is running, are included in section `%post`.
These typically include all installation commands.

As these commands are executed inside the container, we usually have access to package manager softwares for each 
distribution, e.g. `apt` for Ubunbtu (depends on the base image used). All commands are executed with root rights by 
default, so there is no need to prefix the commands with `sudo`.

Many base images are very barebones to keep the images small. For example you often have to start by installing the 
compiler environment, file transfer tools and other tools you may need. A packet manager is usually included.

The base image we chose does not include Python, so we will start by installing it. It does include packet manager `apt`,
so we can use it to install Python. 

Add the following lines to the definition file:

```text
%post
  # Install python
  apt update
  apt install python3 -y
  apt install pip -y
```
💬Indentation is optional, but improves readability. 
💬Comments will make the definition file easier to read for others (and also for yourself after some time has passed).

We will experiment with different installation methods to test some of the options in definition files.

#### 2.3.1 Installing from PyPI

MACS3 is available in PyPI, so the easiest way to install it is by using `pip`.

Add the following lines to the definition file:
```text
  # Install MACS3
  pip install MACS3
```
You can now try to build it:
```bash
singularity build --remote macs3.sif macs3.def
```
If the build finishes, try it:
```bash
singularity exec macs3.sif macs3 --help
```

#### 2.3.2 Installing from source code

Since MACS3 is still under development, we might prefer to install from source code instead to include any
changes not yet available in PyPI. 

For this we also need to install git and python3-devel packages.

Comment out or delete the pip command, and add the following to the definition file:
```text
  apt install python3-devel -y
  apt install git -y
  git clone https://github.com/macs3-project/MACS.git -recurse-submodules
  cd MACS
  pip install -r requirements.txt
  python setup.py install
```
You can now try to build it:
```bash
singularity build --remote macs3.sif macs3.def
```
If the build finishes, try it:
```bash
singularity exec macs3.sif macs3 --help
```

#### 2.3.3 Setting up the environment

Any commands that set up the environment go to the `%environment` section.

The default `$PATH` for a Singularity environment is
```text
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

The default installation location for software is usually one of the folders in the default 
`$PATH`, and thus we don't need to adjust the environment. 

Sometimes however, especially when using Conda or some software specific installations scripts, the 
installation may end up in some other location.

To test this, let's edit the install command above a bit to install to folder /app/macs instead:

```text
  python setup.py install --prefix=/app/macs
```

We will now need to adjust `$PATH` and `$PYTHONPATH`:
```text
%environment
  export PATH=/app/macs/bin:$PATH
  export PYTHONPATH=/app/macs/lib/python3.8/site-packages
```
💬These can also be set at runtime in the host system, but it's more user friendly to set them in container.

In the finished container image these commands are stored in script `/environment`.

You can try this definition file as above:

```bash
singularity build --remote macs3.sif macs3.def
```
If the build finishes, try it:
```bash
singularity exec macs3.sif macs3 --help
```

## 3. Adding addition metadata

If you are building the container image just for yourself, you may be happy with an image with
just the basic functionality.

If you plan of sharing the image with others, it is helpfull to add some additional metada to the
definition file.

### 3.1 Adding a runscript

Runscript is a list of commands that will be executed when the container is run with `singularity run` or when the container 
is run as an executable. It is defined in section `%runscript`.

In the finished container image these commands are stored in script `/singularity`.

For containers that run a service, e.g. a web server or similar, the runscript usually starts the service.

For application containers, like the MACS3 container we are building, it's up to the author to decide what to put 
here. One option is to make the container run any command line options as a command. This will make `singularity run` 
behave similarily to `singularity exec`. You can also choose to leave it empty or e.g. make it print out usge help.

```text
%runscript
  exec "$@"
```
💬In a HPC system, especially with batch jobs, it's best to always use `singularity exec` to run.

The runscript of an existeing container can be checked with:
```bash
singularity inspect --runscript macs3.sif
```

### 3.2 Adding information about the container

Singularity automatically adds some metadata to the container. This includes Singularity version number, build date, base 
image etc. If you wish to add some additional metadata, e.g your contact information, you do it in section `%labels`. This 
is advisable, especially if you wish to distribute the container.

```text
%labels
  Maintainer my.address@example.net
  Version v1.0
```
This information can be checked with command:
```bash
singularity inspect --labels macs3.sif
```

### 3.3 Adding help text

It is also possible to add usage information. This goes to section `%help` and can be viewed with
command:
```bash
singularity run-help macs3.sif
```

```text
%help
  This is a container for MACS3 software
  https://github.com/macs3-project/MACS
  MACS3 is distributed under BSD 3-Clause License
  Usage:
  singularity exec macs3.sif macs3 --help
```
Try adding some additional metadata to the definition file and rebuild.
```bash
singularity build --remote macs3.sif macs3.def
```
You can check the information you added:
```bash
singularity inspect --runscript macs3.sif
singularity inspect --labes macs3.sif
singularity run-help macs3.sif
```

We have provided some [example definition files](https://github.com/amsaren/course_materials/tree/main/Biocontainers_2021/Files),
including the various versions of this tutorial. 