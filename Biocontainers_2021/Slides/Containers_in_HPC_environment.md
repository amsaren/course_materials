---
theme: csc-2019
lang: en
---

# Using Containers in HPC environments {.title}

# Basics 1/2
- Singularity available both in login nodes and compute nodes
  - Only light tasks (`singularity inspect` etc) should be done in the login nodes
  - All actual computation should be done in the compute nodes, i.e.  as a batch 
  job or in an interactive session

# Basics 2/2
- Default location for Singularity temporary files is `~/.singularity`
- Since home directory is quite small it's best to direct the files to other location
  - Especially `.singularity/cache` can fill up fast if you pull/build containers
- You can set other location by setting environment variables 
`$SINGULARITY_CACHEDIR` and `$SINGULARITY_TMPDIR`



