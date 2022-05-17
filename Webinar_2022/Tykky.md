---
theme: csc-2019
lang: en
---

# Easy way to install software - Tykky container wrapper {.title}

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

# Containers in a nutshell
- Containers are a way to package software with its dependencies (libraries, etc)
- Popular container engines include Docker, Singularity, Shifter
- Singularity is the most popular in HPC environments
  - Singularity can run most Docker containers

# Container benefits: Ease of installation
- Easy installation of complex softwar packages
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
  - Most bioscience applications are thread-base, so this is not aproblem

# Container complications
- Building containers from scratch can be a bit tricky
- Running containerized applications requires special commands
- Care needed to make files visible inside the containers
  - Using `singularity_wrapper` command takes care some of this

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
- Basic command:
 ```
  conda-containerize new --prefix <install_dir> env.yml
 ```

# Environment file
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
  pip-containerize new --prefix <install_dir> req.txt
  ```
  - `req.txt` pip requirements file
- By default uses the Python in current `$PATH`as base
  - Can not be a conatiner-based installation
  - Using option `--slim` starts with a minimal Python installation


# Using Tykky to create wrapper for existing conatiner
- Basic command:
  ```
  wrap-container -w </path/inside/container> <container> --prefix <install_dir> 
  ```
  - `-w` the directory/directories inside the container where the executables are
  - `container` can be a path toa conatiner image file or an URL
  

# More information in Docs

https://docs.csc.fi/computing/containers/tykky/



```
export PATH="<install_dir>/bin:$PATH"
```




```
```