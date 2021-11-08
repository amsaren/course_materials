---
topic: singularity
title: Exercise - Creating singularity containers
---

# Building Singularity containers in sandbox mode

In this tutorial we create a Singularity container in sandbox mode.

This is an extra exercise. It can not be run in Puhti. You will need  access
to a computer or a virtual machine where you have sudo rights and that has
Singularity 3.x installed.

When building a container in sandbox mode, the container image is created
as a writable directory tree that corresponds to the internal file system of
the container. The installation is then done interactively using `singularity shell`.

Using the sandbox mode can be a bit more intuive and faster than using definition
files, especially if you have to iterate some parts (finding correct library versions 
etc). The problem is that a container created this way is not easily reproducible.
It is a good practise to keep track of the commands you use and write a definition
file afterwrds.

In this tutorial we focus on using Singularity. In our course 
[Using CSC HPC Environment Efficiently](https://csc-training.github.io/csc-env-eff/)
we have a tutorial [Installing a simple C code from source](https://github.com/csc-training/csc-env-eff/blob/master/_hands-on/installing/installing_hands-on_mcl.md) where you can find more information on the installation
process itself.

In this tutorial we only cover the basics. Detailed instructions can be found
in the [Singularity manual](https://sylabs.io/guides/3.7/user-guide).

## Sandbox mode

One way to create a Singularity container is to do it in so-called sandbox
mode. Instead of an image file, we create a directory structure
representing the file system of the container. 

To start with, we create a basic container from a definition file. To choose
a suitable Linux distribution, you should check the documentation of the
software you wish to install. If the the developers provide installation 
intructions for a specific distribution, it is usually easiest to start with that.

In this case there is no specific reason to choose one distribution over another,
but from past experience I know the software installs without problems in CentOS,
so we start with that.

We start from a very bare-bones definition file. Copy the following lines to
a file called `centos.def`.
```text
Bootstrap: yum
OSVersion: 7
MirrorURL: http://mirror.centos.org/centos-%{OSVERSION}/%{OSVERSION}/os/$basearch/
Include: yum
```
We then use that definition file to build the container:
```bash
sudo singularity build --sandbox mcl centos.def
```
Note that instead of an image file, we created a directory called `mcl`. If
you need to include some reference files etc, you can copy them to correct subfolder.

We can then open a shell in the container. We need the container file system 
to be writable, so we include option `--writable`:
```bash
sudo singularity shell --writable mcl
```
The command prompt should now be `singularity>`

If there is a need to make the container as small as possible, we should only
install the dependencies we need. A smaller container will load faster, so this
can be a considertaion in workflows where the same container is started repeatedly
(e.g. once for each input file). Usually the size is not that critical, so we may
opt more for ease of use. 

In this case we install application group "Development tools" that includes 
most of the components we need (C, C++, make), but also a lot of things we 
don't actually need in this case.

Notice that unlike in the CSC machine, we are able to use the package mangement 
tools (in this case `yum`). This will often make installing libraries and other 
dendencies easier.

Also notice that it is not necessary to use `sudo` inside the container.

```bash
yum group install "Development Tools" -y
yum install wget -y
```
We are now ready to install the software. 

Download and extract the distribuition package:
```bash
wget https://micans.org/mcl/src/mcl-latest.tar.gz
tar xf mcl-latest.tar.gz
cd mcl-14-137
```
Run configure:
```bash
./configure
```
Check the output of `configure` and install any missing dependencies
(In this case there should not be any.)

And finally run `make`:
```bash
make
make install
```

We can now test the application to see it works:
```bash
mcl --version
```
If everything works we can clean up:
```bash
cd ..
rm -rf mcl-*
```
We can also add a runscript:
```bash
echo 'exec /bin/bash "$@"' >> /singularity
```
We can now exit the container:
```bash
exit
```
In order to run the container without sudo rights we need to build
a production image from the sandbox:

```bash
sudo singularity build  mcl.sif mcl
```
We can now test it. Note that we no longer need `sudo`:
```bash
singularity exec mcl.sif mcl --version
```

## Definition file

The above method is applicable as-is if you intend the
container to be only used by you and your close collaborators.
However, if you plan to distribute it wider, it's best to write
a definition file for it. That way the other users can see
what is in the container and they can, if they so choose, easily 
rebuild the production image.

A definition file will also make it easier to modify and re-use 
the container later. For example, software update can often be done
simply by modifying the version number in the definition file and
re-building the image.

To write the definition file we can start from the original 
bare-bones file and add various sections to it as necessary.

Command `history` can be useful to keep track of the actual installation 
commands, but if you need try several different thing, keep track of what 
finally worked.

We have a separate tutorial on [Creating Singularity containers from definition files](.\building_containers_from_def_file.md),
so we will not go into details here.

Check the [example definition file](../Files/mcl.def) for this exercise.