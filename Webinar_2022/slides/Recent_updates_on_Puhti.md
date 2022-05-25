---
theme: csc-2019
lang: en
---

# Webinar: Recent updates on Puhti for biousers {.title}
<div class="column">
![](https://mirrors.creativecommons.org/presskit/buttons/88x31/png/by-sa.png)
</div>
<div class="column">
<small>
All material (C) 2020-2022 by CSC -IT Center for Science Ltd.
This work is licensed under a **Creative Commons Attribution-ShareAlike** 4.0
Unported License, [http://creativecommons.org/licenses/by-sa/4.0/](http://creativecommons.org/licenses/by-sa/4.0/)
</small>
</div>

# Todays topics
- Tykky Container wrapper
- Puhti web interface
- Short topics


# Easy way to install software - Tykky container wrapper {.title}

# Containers in a nutshell
- Containers are a way to package software with its dependencies (libraries, etc)
- Popular container engines include Docker, Singularity, Shifter
- Singularity is the most popular in HPC environments
  - Singularity can run most Docker containers

# Container benefits: Ease of installation
- Easy installation of complex software packages
  - Single-command installation if a ready container exists
  - Many developers distribute their software as containers
  - Third-party created containers available for many more
    - E.g. all packages in Bioconda repository also available 
    as containers 

# Container benefits: Environment isolation
- Containers use host system kernel, but can have their own Bins/Libs layer
  - Can be a different Linux distribution that the host
  - Can solve some incompatibilities
  - Less likely to be effected by changes in the host system
  
# Container benefits: Enviroment reproducibility
- Analysis environment can be saved as a whole
  - Useful with e.g. Python, where updating underlaying 
  libraries (Numpy etc) can lead to differences in behavior  
- Sharing with collaborators easy (single file)

# Container benefits: Portability
- Many containers can run as-is on any system with container support
  - Same container could run in Puhti, Mahti, LUMI, your own computer...
- Software requiring MPI will need suitable container for each system
  - Most bioscience applications are thread-based
- Sofware requiring GPU more portable than MPI, but may require modifications

# Container complications
- Building containers from scratch can be a bit tricky
  - Some operations require root acces on build system
- Running containerized applications requires special commands
- Care needed to make host system files visible inside the containers
  - Using `singularity_wrapper` command takes care most of this

# Tykky container wrapper
- Provides an easy way to do containerised installations
- Creates wrappers for commands, so no special commands needed:
    Instead of e.g.

     ```
     singularity_wrapper exec myimage.sif myprog <options>
     ```
     just:
    
     ```
     myprog <options>
     ```

# Using Tykky to install a Conda package
- "Bare" Conda installations should be avoided
  - Installations can have tens of thousands of files
  - Leads to performance issues on Lustre
- Basic command to install into folder "MyEnv":

  ```
  mkdir MyEnv
  conda-containerize new --prefix MyEnv env.yml
  ```

# Environment file
- Typically provided with the software distribution
- Can be exported from an existing Conda environment
  
  ```
  conda env export -n <target_env_name> > env.yaml 
  ```

- Or written by hand
  
  ```
  channels:
    - conda-forge
  dependencies:
    - python=3.8.8
    - scipy
    - nglview
  ```

# Using Tykky to install Python packages
- Basic command for pip -based installations:

  ```
  mkdir MyEnv
  pip-containerize new --prefix MyEnv req.txt
  ```

  - `req.txt`: pip requirements file
- By default uses the Python in current `$PATH` as base
  - Depends on loaded modules
  - Can not be a container-based installation
  - Using option `--slim` starts with a minimal Python installation

# Using Tykky to install Python packages, continued
- For Python software with no pip package available 
  - First create basic environment with `conda-containerize new`
  - Add installation command to a file, e.g. "post.txt"
  - Then use `conda-containerize update`

  ```
  conda-containerize update MyEnv --post-install post.txt 
  ```
  
    post.txt:
  
    ```
    python setup.py install
    ```

# Using Tykky to create wrapper for existing container
- Basic command:

  ```
  wrap-container -w </path/inside/container> <container> --prefix <install_dir> 
  ```

  - `-w` the directory/directories inside the container where the executables are
  - `container` can be a path to a container image file or an URL
  
  example:

  ```
  mkdir bbmap
  wrap-container -w /usr/local/bin docker://quay.io/biocontainers/bbmap:38.96--h5c4e2a8_1 \
  --prefix bbmap 
  ```

# More information in Docs

https://docs.csc.fi/computing/containers/tykky/

# Short topics {.title}

# Course: CSC Computing Environment
- Self learning course
  - https://ssl.eventilla.com/event/CSCCompEnvSelfLearn
  - Aimed at beginner to intermediate users
   - Also some advanced topics covered
- Materials available also directly:
  - https://csc-training.github.io/csc-env-eff/
  - Tutorials written to be self-standing introductions to topics
- Course is meant to complement CSC Docs pages

# OS updates
- Operating systems on CSC supercomputers will be updated to
  Red Hat Enterprice Linux (RHEL) 8
  - Mahti updated in May 2022 
  - Puhti update planned for Spetember 2022
- Software installed by CSC will be re-installed by CSC
- Software installed by you
  - Software installed as binaries or containers will probably be ok
  - Software compiled by you needs to be re-compiled

# Take home message {.title}

# Take home message
- Wrap Conda installations with Tykky
- Puhti web interface easy way to run graphical applications, Jupyter notebooks, Rstudio etc
- OS update coming on Puhti