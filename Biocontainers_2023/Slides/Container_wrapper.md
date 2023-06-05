---
theme: csc-2019
lang: en
---

# Container wrapper {.title}

<div class="column">
![](https://mirrors.creativecommons.org/presskit/buttons/88x31/png/by-sa.png)
</div>
<div class="column">
<small>
All materials (c) 2020-2023 by CSC – IT Center for Science Ltd.
This work is licensed under a **Creative Commons Attribution-ShareAlike** 4.0
Unported License, [http://creativecommons.org/licenses/by-sa/4.0/](http://creativecommons.org/licenses/by-sa/4.0/)
</small>
</div>

# Container wrapper

- Tool for installing software in containers with normal user rights
- Developed mainly to replace direct Conda installations
- Can also be used for other installations.
- In Puhti and Mahti:

  ```
  module load tykky
  ```

- In Lumi:

  ```
  module load LUMI/22.08 
  module load lumi-container-wrapper
  ```


# Works with user rights

1. Pulls a basic container image
2. Installs user software to a SquashFS image
  - Mounted to `/CSC_CONTAINER`inside the container at run time
  - Path for Conda environment `/CSC_CONTAINER/miniconda/envs/env1`
3. Creates wrapper scripts for all executables in designated folder
  - Can also create wrapper scripts for a ready-made container


# Wrapper scripts

- Can be run like the original executables
- Run the corresponding application inside the container
- Run all the necessary Apptainer commands
  - Automatically bind most common directories (`/users`, `/projappl`, `/scratch`) and the SquashFS image 

# Creating Conda-based installation 1/2

- You should not have any other Tykky based installation in `$PATH` when installing
  - Start with `module purge`
  - Also `unset PYTHONPATH` if necessary
- You will need a environment YAML file
  - Create by hand
  - Sometimes provided by developers
  - Can be retrieved form existing environment
    
    ```  
    conda env export -n <target_env_name> > env.yml
    ```

# Example Environment file

```
channels:
  - conda-forge
dependencies:
  - python=3.8.8
  - scipy
  - nglview
```

# Creating Conda-based installation 2/2

```
conda-containerize new --prefix <install_dir> env.yml
```

- `--mamba` to use Mamba instead of Conda
  - typically faster, but may fail to solve some environments
- `--requirement/-r <file>` requirement file for `pip`


# Updating Conda-based installation

```
conda-containerize update <existing installation> --post-install <file> 
```

Where the post-install file contains any additional installation commands


# Example post-install file

```
conda install -y numpy
pip install requests
cd /some/folder
python setup.py install

```

- [A C++ based example](https://github.com/CSCfi/hpc-container-wrapper/blob/master/examples/fftw.md)


# Creating wrappers for an existing container

```
wrap-container -w /path/inside/container <container> --prefix <install_dir> 
```

- `-w` is the path (or comma separated list of paths) where the executables to be wrapped are. Needs to an absolute path inside the container.


# Running Tykky installed software

- Add installation `bin` directory to `$PATH`

  ```
  export PATH=/projappl/project_12345/someapp/bin:$PATH
  ```

- Tykky installation bin directory often contains common programs (e.g. as python)
  - Can cause conflicts with other software
  - Only add to `$PATH` when actually using


# Common problems

- If running out of memory when installing, try installing in an interactive session, e.g. 

  ```
  sinteractive --account <project> --time 1:00:00 --mem 12000 --tmp 15
  ```

- Most common reason for non memory related errors is problems with the environment YAML file syntax
  - Check e.g. the indentation and syntax for version numbers
- For Conda environment related errors follow normal procedure (a gallon of coffee and a spare keyboard to bang your head on recommended)

# Documentation

- [Puhti, Mahti](https://docs.csc.fi/computing/containers/tykky/)
- [Lumi](https://docs.lumi-supercomputer.eu/software/installing/container-wrapper/)