---
theme: csc-2019
lang: en
---

# Basic usage of Apptainer containers {.title}

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


# Apptainer on CSC supercomputers

- Apptainer available in Puhti and Mahti, SingularityCE on LUMI
- Apptainer jobs are treated just as any other jobs
- Users can run their own containers
  - No need to load a module. Apptainer in `$PATH` by default.
- Some CSC software installations are provided as containers (e.g. R, Python environments, many bio tools)
  - In many cases the containers are "hidden" behing wrapper scripts
  - See the software pages in Docs CSC for details


# Running Apptainer containers: Basic syntax

- Execute a command in the container
  - `apptainer exec [exec options...] <container> <command>`
- Run the default action (runscript) of the container
  - Defined when the container is built
  - `apptainer run [run options...] <container>`
- Open a shell in the container
  - `apptainer shell [shell options...] <container>`


# File system

- Containers have their own internal file system (FS)
  - The internal FS is read-only when executed with user-level rights
- To access host directories, they need to be mapped to container directories
  - This can be done with the `--bind (-B)` option
  - E.g., to map the host directory `/scratch/project_2001234` to `/data` directory inside the container:
    ```
    --bind /scratch/project_2001234:/data
    ```
  - The target directory inside the container does not need to exist, it is created if necessary
  - More than one directory can be mapped


# File system

- Basic syntax: `/host/dir:/container/dir`
- If path is same in host and container it can be specified only once
  ```
  --bind /scratch:/scratch
  ```

  and 
  
  ```
  --bind /scratch
  ```
   do the same


# File system

- Multiple paths can be separated by comma or given as separate options:
  ```
  --bind /projapp,/scratch
  ```
  and 
  ```
  --bind /projappl --bind /scratch
  ```
  do the same 


# File system

- Current working directory (`$PWD`) bound by default
- Some host system directories also bound
  - Can be controlled with `--no-mount` option
- Also possible to set `$APPTAINER_BIND` environment variable
  
  ```
  export APPTAINER_BIND="/host/dir:/container/dir"
  ```


# `apptainer_wrapper`

- Running containers with `apptainer_wrapper` takes care of the most common `--bind` commands automatically
  - Binds: /users, /projappl, /scratch, /appl/data
  - Binds `$TMPDIR` if set and directory exists
  - Binds `$LOCAL_SCRATCH` if set
- For containers that need specific binding it's best to use `apptainer exec --bind`


# `apptainer_wrapper`
- You need to set the `$SING_IMAGE` environment variable to point to the correct Apptainer image file

  ```bash
  export SING_IMAGE=/path/to/container.sif
  apptainer_wrapper exec myprog <options>
  ```


# `apptainer_wrapper`

- Any other options can be set by setting `$SING_FLAGS`
  - Any additional `--bind` statements
  - Add `--nv` to use host CUDA
  - Adding `--cleanev` if necessary


# Environment variables

- Most environment variables from the host are inherited by the container
  - Can be prevented if necessary by adding the option `--cleanenv`
- Environment variables can be set specifically inside the container by setting on the host `$APPTAINERENV_variablename`.
  - E.g., to set `$TEST` in a container, set `$APPTAINERENV_TEST` on the host


# Temporary files

- Default location for Apptainer temporary files is `~/.apptainer`
  - Note: These are used by Apptainer itself, not by applications inside the container
- Since home directory is quite small, it's best to direct the files to some other location
  - Especially `.apptainer/cache` can fill up fast if you pull/build containers
    - Can be safely emptied
      
        ```bash
        apptainer cache clean
        ```

- You can set other location by setting environment variables `$APPTAINER_CACHEDIR` and `$APPTAINER_TMPDIR`


# Running containers

- Containers can be run just as any other software
  - Batch job scripts same for containerized and non-containerized sofware
- When running as a batch job, it is best to use `apptainer exec`
  - Depending on the runscript `apptainer run` can cause complications, i.e. 
  variables not inherited as expected, etc
- If not using `apptainer_wrapper`, remember to bind all necessary directories


# Special resources: GPU

- To Use GPU:
  - Reserve it in the batch job script and use gpu-enabled partition:
    
    ```
    #SBATCH --partition=gpu
    #SBATCH --gres=gpu:v100:<number_of_gpus_per_node>
    ```

  - Also add option `--nv`
    - Allows container to use host driver stack, CUDA, etc
  - Depending on the container you may also need to set `APPTAINERENV_LD_LIBRARY_PATH`
    - In case of CSC installed software, check the Docs
  - Run with `srun`:
    `srun apptainer exec <options>`


# Special resources: NVMe

- To use fast local NVMe disk: 
  - Reserve it in the batch job script:
  
    ```
    #SBATCH --gres=nvme:<local_storage_space_per_node>
    ```

- `$LOCAL_SCRATCH` and `$TMPDIR` are set by the system if NVMe resources allocated for the job 
  - If using `apptainer_wrapper`, `$LOCAL_SCRATCH` and `$TMPDIR` are bound automatically
  - If not, it has to be bound, e.g. `--bind $LOCAL_SCRATCH,$TMPDIR` or `--bind /run`
  - It may be necessary to bind `$TMPDIR` to /tmp: `--bind $TMPDIR:/tmp`


# SquashFS

- Lustre is very inefficient in handling large number of files in single directory
- When using Apptainer containerized tools you can use SquashFS to package files
  - Single file on Host filesystem
  - Can be mounted as a directory inside the container
  
  ```
  mksquashfs my_dataset my_dataset.sqfs
  apptainer exec --bind my_dataset.sqfs:/data:image-src=/ image.sif myprog
  ```

- See instructions in [Docs](https://docs.csc.fi/computing/containers/run-existing/#mounting-datasets-with-squashfs)


# Example batch job script

 ```
#!/bin/bash
#SBATCH --job-name=example
#SBATCH --account=project_12345
#SBATCH --partition=small
#SBATCH --time=02:00:00
#SBATCH --ntasks=1
#SBATCH --mem-per-cpu=4000

export SING_IMAGE=/scratch/project_12345/image.sif
export SING_FLAGS="--bind /scratch/project_12345/my_reference:/reference $SING_FLAGS"
apptainer_wrapper exec myprog -i input -o output
```

# Some recurring problems

- Conflicts with host system
  - Typical causes include `$PYTHONPATH` or `$PERL5LIB` set on host, etc.
    - Usually due to other modules being loaded or changes to `.bashrc`
    - Inherited by the container, but interpreted as paths inide the container
  - Adding option `--cleanenv` prevents host environment variables from being inherited


# Some recurring problems

- Locale related problems
  - Typically due to host default locale not being available in the container
  - Try `--cleanenv`
  - Try setting locale to "C" before running the container
  
   ```
   export LC_ALL=C
   ``` 
- Should be solved when building the container, but not an easy option when using a ready container


# Some recurring problems

- Graphics/X11 related problems
  - Seems to happen mainly with Python
  - If you get X11 related error message, try unsetting `$DISPLAY`
  
    ```
    unset DISPLAY
    ```

# Some recurring problems

- Applications trying to write to container FS
  - Container FS read-only at runtime, so this will cause an error
  - See if there is command line option to set the directory the application wants to write to
  - Try binding the directory to a writable directory on host

# Some recurring problems

- Error message about not being able to mount `/tmp`
  - Usually solved by defining `$TMPDIR`

# Some recurring problems

- Errors related to command `which`
  - Some container images use BusyBox instead of GNU coreutils
  - Often `--cleanenv` will help
    - e.g. error "unknown option --tty-only" caused by inheriting system alias settings from host
