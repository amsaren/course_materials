---
topic: containers
title: Tutorial - Running applications in containers
---

# Apptainer tutorial

💬 In this tutorial, we will get familiar with the basic usage of Apptainer (previously Singularity) containers.

In normal testing it is best to use [interactive sessions](https://docs.csc.fi/computing/running/interactive-usage/). However, during courses Puhti `ìnteractive` partition can get congested, so these exercises have been designed to be light so you can run them in the login node.

## Getting started

Log into Puhti.

Download a test container image from Allas:

```bash
wget  https://a3s.fi/saren-2001659-pub/tutorial.sif
ls -lh tutorial.sif
```

The file we downloaded is a container image. It contains all the software and data of the container in a single file. In this case, the container is very bare-bones and thus quite small, about 50 MB.
    - Actual application containers are typically larger since they also contain the software installation and may in some cases include reference data, etc.

## Basic usage

💬 There are three basic ways to run software in an Apptainer container:

### Apptainer `exec`

To execute a command inside the container, use `apptainer exec`:

```bash
apptainer exec tutorial.sif hello_world
```

Compare the outputs of the following commands:

```bash
cat /etc/*release
apptainer exec tutorial.sif cat /etc/*release
```

- The first command is run on the host, the second command is run inside the container.

💭 The tutorial container is based on Ubuntu 18.04. The host and the container use the same kernel, but the rest of the system can vary.

- This means that a container can be based on a different Linux distribution than the host (as long as they are kernel-compatible), but it can't run a totally different OS, such as Windows or macOS.

#### Apptainer `exec` in batch jobs

💡 `apptainer exec` is the run method you will typically use in batch job scripts.

Create a file called `test.sh`:

```bash
module load nano    # The compute nodes do not have nano available by default
nano test.sh
```

Copy the following contents into the file and replace `<project>` with your actual CSC project, e.g. `project_2001234`:

```bash
#!/bin/bash
#SBATCH --job-name=test           # Name of the job visible in the queue.
#SBATCH --account=<project>       # Choose the billing project. Has to be defined!
#SBATCH --partition=test          # Job queues: test, interactive, small, large, longrun, hugemem, hugemem_longrun
#SBATCH --time=00:01:00           # Maximum duration of the job. Max: depends of the partition. 
#SBATCH --mem=1G                  # How much RAM is reserved for job per node.
#SBATCH --ntasks=1                # Number of tasks. Max: depends on partition.
#SBATCH --cpus-per-task=1         # How many processors work on one task. Max: Number of CPUs per node.

apptainer exec tutorial.sif hello_world
```

Submit the job to the queue with:

```bash
sbatch test.sh
```

💡 For more information on batch jobs, see [Docs CSC](https://docs.csc.fi/computing/running/getting-started/).

### Apptainer `run`

💬 When containers are created, a standard action called the `runscript` is defined. Depending on the container, it may simply print out a message, or it may launch a program or service inside the container.

💭 If you are using a container created by someone else, you will need to check the documentation provided by the creator for details.

In our test container the `runscript` prints out a simple message:

```bash
apptainer run tutorial.sif
```

Give the container image execute rights so that you can run it directly:

```bash
chmod u+x tutorial.sif
./tutorial.sif
```

You can see the actual script with the command:

```bash
apptainer inspect --runscript tutorial.sif
```

### Apptainer `shell`

Open a shell inside the container:

```bash
apptainer shell tutorial.sif
```

Notice that the command prompt changed. You can now run any software inside the container interactively:

```bash
hello_world
```

Exit the container with:

```bash
exit
```

## Using files

💬 Apptainer containers have their own internal file system that is separated from the host file system.

- The internal file system is always read-only when the container is run with normal user privileges.

💭 In most real use cases you might want to access the host file system to read and write files.

- To do this, you must bind a writable directory on the host file system to the internal file system of the container.

💡 This is done using the command line argument `--bind` (or `-B`). The basic syntax is `--bind /path/inside/host:/path/inside/container`.

💭 Some remarks:  
➖ The bind path does not need to exist inside the container – it is created if necessary.  
➖ More than one bind pair can be specified.  
➖ The option is available for all run methods described in the previous tutorial.  


Try listing the contents of your project directory (edit the path as needed) from inside the container without `--bind`:

```bash
export SCRATCH=/scratch/<project>/$USER    # replace <project> with your CSC project, e.g. project_2001234
apptainer exec tutorial.sif ls $SCRATCH
```

The container cannot see the host directory, so you will get a `No such file or directory` error.

Try binding the host directory `/scratch` to the directory `/scratch` inside the container:

```bash
apptainer exec --bind $SCRATCH:/scratch tutorial.sif ls /scratch
# or
apptainer exec --bind /scratch:/scratch tutorial.sif ls $SCRATCH
```

This time, the host directory is linked to the container directory and the command shows what the container sees inside `/scratch`.

💡 You can use `--bind` to set the container, for example, to find input data or configuration files from a certain directory.

Bind the host directory specified in `$SCRATCH` to a directory called `/input`:

```bash
apptainer exec --bind $SCRATCH:/input tutorial.sif ls /input
```

### Using the `apptainer_wrapper`

If you use the wrapper script `apptainer_wrapper`, it will automatically take care of the most common bind use cases.

You just need to set a `$SING_IMAGE` environment variable to point to the correct Apptainer image file:

```bash
export SING_IMAGE=$PWD/tutorial.sif
apptainer_wrapper exec ls $SCRATCH
```

💡 Note that the image file name is not needed in the `apptainer_wrapper` command since `$SING_IMAGE` is set.

‼️ Since some modules set `$SING_IMAGE` when loaded, it is a good idea to start with `module purge` to make sure the correct image is used if you plan to use `apptainer_wrapper`.

## Environment variables

💬 Some software may require environment variables to be set, e.g., to point to some reference data or a configuration file.

💬 Most environment variables set on the host are inherited by the container.

☝🏻 Sometimes this may be undesired, in which case the command line option `--cleanenv` can be used to prevent the host environment from being inherited by the container.

💬 To set an environment variable specifically inside the container, you can set an environment variable `$APPTAINERENV_XXX` (where `XXX` is the variable name) on the host before invoking the container.

Set some test variables:

```bash
export TEST1="value1"
export APPTAINERENV_TEST2="value2"
```

Compare the outputs of:

```bash
env | grep TEST
apptainer exec tutorial.sif env | grep TEST
apptainer exec --cleanenv tutorial.sif env | grep TEST
```

- The first command is run on the host and we see `$TEST1` and `$APPTAINERENV_TEST2`.
- The second command is run inside the container and we see `$TEST1` (inherited from the host) and `$TEST2` (specifically set inside the container by setting `$APPTAINERENV_TEST2` on the host).
- The third command is also run inside the container, but this time we omitted the host environment variables so we only see `$TEST2`.

Note that any command-line variables on the host are substituted by their values when passed to the container:

```bash
apptainer exec tutorial.sif echo $TEST1
apptainer exec tutorial.sif echo $TEST2
```

- The first line prints the value set on the host.
- The second line results in an empty output because a variable called `$TEST2` has not been set on host. It was `APPTAINERENV_TEST2="value2"`, remember?

## Exploring containers

💬 Our test container includes the program `hello2`, but it has not been added to the `$PATH` variable.

One way to find it is to try running `find` inside the container:

```bash
apptainer exec tutorial.sif find / -type f -name "hello2" 2>/dev/null
```

You can now run it by providing the full path:

```bash
apptainer exec tutorial.sif /found/me/hello2
```

Or you could add it to `$PATH` inside the container:

```bash
export APPTAINERENV_PREPEND_PATH=/found/me
apptainer exec tutorial.sif hello2
```

💡 If you can't locate the desired binary with `find`, you can always use `apptainer shell` to explore the container.

## More information

💬 This tutorial is meant as a brief introduction to get you started.

☝🏻 When searching online for instructions, pay attention that the instructions are for the same version of Apptainer as you are using. There has been some command syntax changes etc. between versions so older instructions may not work as copy/paste. Also note that Apptainer is formerly known as Singularity.

💡 For more detailed instructions, see the [Apptainer documentation](https://
