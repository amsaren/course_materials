---
theme: csc-2019
lang: en
---

# Using Containers in CSC HPC environment {.title}

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

# Basics 1/2
- Singularity available in Puhti and Mahti
  - available both in login nodes and compute nodes
  - Only light tasks (`singularity run-help` etc) should be done in the login nodes
  - All actual computation should be done in the compute nodes, i.e.  as a batch 
  job or in an interactive session (`sinteractive`)

# Basics 2/2
- Default location for Singularity temporary files is `~/.singularity`
- Since home directory is quite small, it's best to direct the files to some other location
  - Especially `.singularity/cache` can fill up fast if you pull/build containers
    - Can be safely emptied
- You can set other location by setting environment variables `$SINGULARITY_CACHEDIR` and `$SINGULARITY_TMPDIR`

# Running containers
- Containers can be run jus as any other software
  - Batch job scripts same for containerized and non-containerized sofware
- When running as a batch job, it is best to use `singularity exec`
  - Depending on the runscript `singularity run` can cause problems, i.e. 
  environment not inherited correctly, etc
- If not using `singularity_wrapper`, remember to bind all necessary directories

# Singularity_wrapper
- `singularity_wrapper` takes care of most common binds
  - Binds: /users, /projappl, /scratch, /appl/data
  - Binds `$TMPDIR`if set and directory exists
  - Binds `$LOCAL_SCRATCH` if set.
- Uses `$SING_IMAGE` as image file if set
  - Some modules set `$SING_IMAGE`. If you have problems do `module purge` or `unset SING_IMAGE`
- Any other options can be set by setting `$SING_FLAGS`

# Special resources: GPU
- To Use GPU:
  - Reserve it in the batch job script: `--gres=gpu:v100:<number_of_gpus_per_node>`
  - For some containers you need to add `--nv`
    - Allows container to use host driver stack, cuda, etc
    - Some containers include cuda etc, and don't nee `--nv`
    - In case of CSC installed software, check the Docs

# Special resources: NVMe
- To use fast local NVMe disk: 
  - Reserve it in the batch job script: `#SBATCH --gres=nvme:<local_storage_space_per_node>` 
  - If using `singularity_wrapper` `$LOCAL_SCRATCH` is bound automatically. 
  - If not, it has to be bound, e.g. `--bind $LOCAL_SCRATCH:$LOCAL_SCRATCH`

# SquashFS
- Lustre is very inefficient in handling large number of files in single directory
- When using Singularity containerised tools you can use SquashFS to package files
  - Single file on Host filesystem
  - Can be mounted as a directory inside the container
- See instructions in [Docs](https://docs.csc.fi/computing/containers/run-existing/#mounting-datasets-with-squashfs)