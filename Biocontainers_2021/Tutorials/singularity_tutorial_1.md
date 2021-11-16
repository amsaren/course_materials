---
topic: singularity
title: Tutorial - Singularity introduction start (essential)
---

# Singularity tutorial

💬 In this tutorial we get familiar with the basic usage of Singularity containers. 

1. To run these exercises in Puhti, use `sinteractive`.
```bash
sinteractive --account project_xxxx   # Change the xxxx for the project number
```

## Getting started

2. Download a test container image from allas.
    ```bash
    wget  https://a3s.fi/saren-2001659-pub/tutorial.sif
    ls -lh tutorial.sif
    ```
3. As you can see, the container is a single file. In this case the container is very bare-bones, and thus quite small, about 50 MB. 
    - Actual application containers are typically larger, since they also contain the software installation, and may in some cases include some reference data etc.

## Basic usage

💬 There are three basic ways to run software in a Singularity container:

### Singularity exec
1. To execute a command inside the container, the command is `singularity exec`.
    ```bash
    singularity exec tutorial.sif hello_world
    ```
2. Compare the outputs of the following commands:
    ```bash
    cat /etc/*release
    singularity exec tutorial.sif cat /etc/*release
    ```
    - The first command is run in the host. The second command is run inside the container.

💭 The tutorial container is based on Ubuntu 18.04. The host and the container use the same kernel, but the rest of the system can vary. 
- That means a container can be based on a different Linux distribution than the host (as long as they are kernel-compatible), but can't run a totally different OS like Windows or macOS.

### Singularity run
💬 When containers are created, a standard action, called the `runscript` is defined. Depending on the container, it may simply print out a message, or it may launch a program or service inside the container. 

💭 If you are using a container created by somebody else, you will need to check the documentation provided by the creator for details.

1. In our test container it prints out a simple message:
    ```bash
    singularity run tutorial.sif
    ```
2. Give the container image execute rights and you can also run it directly:
    ```bash
    chmod u+x tutorial.sif
    ./tutorial.sif
    ```
3. You can see the actual script with command:
    ```bash
    singularity inspect --runscript tutorial.sif
    ```

### Singularity shell

1. Open a shell inside the container. 
    ```bash
    singularity shell tutorial.sif
    ```
2. You can see the command prompt change to `Singularity>`. You can now run any software inside the container interactively:
    ```bash
    hello_world
    ```
3. Exit the container with:
    ```bash
    exit
    ```

### Getting help

You can check if the creator of the container has included any help on the usage:
```bash
singularity run-help tutorial.sif
``` 

Adding this help is, however, optional, and in many containers it is missing. In those case yoy could 
check e.g. the developers web site for details.

For software installed by CSC, check [Docs CSC](https://docs.csc.fi/apps/alpha/).

## More information

💬 This tutorial is meant as a brief introduction to get you started.

☝🏻 When searching the internet for instruction, pay attention that the instructions are for the same version of Singularity that you are using. There has been some command syntax changes etc. between the versions, so older instructions may not work with copy-paste.

💡 For authoritative instructions see [Singularity documentation](https://sylabs.io/docs/).
