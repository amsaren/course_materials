---
topic: containers
title: Tutorial - Tykky container wrapper
---

# Tykky container wrapper

CSC provides the [Tykky conatiner wrapper tool]https://docs.csc.fi/computing/containers/tykky/) to build container-based installations on CSC supercomputers with normal user rights. It works around the privilege restrictions by starting with a ready-made container and then does all the installations in a SquashFS filesystem image that gets bound at runtime. 

Tykky also creates wrapper scripts for all the executables in specifed folders so in most cases usage is identical to natively installed applications.

Tykky provides commands `conda-containerize`to create manage Conda-based installations, `pip-containerize` for simple pip-based Python installations and `wrap-container` to create wrapper scripts for an existing container.

You can also use `conda-containerize new` to set up the basic environment and then use `conda-containerize update` to run any installation commands.

Many software packages provided by CSC have been installed using Tykky, e.g. R, many Python modules etc.

## Containerizing a Conda environment with Tykky

💬 Let's create a containerized Conda environment using the Tykky wrapper.

Create a folder under your project's `/projappl` directory for the installation, e.g.:

```bash
mkdir -p /projappl/<project>/$USER/tykky-env    # replace <project> with your CSC project, e.g. project_2001234
```

Create an `env.yml` environment file defining the packages to be installed. Using for example `nano`, copy/paste the following contents to the file:

```yaml
channels:
  - conda-forge
dependencies:
  - python=3.10.8
  - scipy
  - pandas
  - nglview
```

Purge your current module environment and load the Tykky module:

```bash
module purge
module load tykky
```

Create and containerize the Conda environment using the `conda-containerize` command:

```bash
conda-containerize new --prefix /projappl/<project>/$USER/tykky-env env.yml    # replace <project> with your CSC project, e.g. project_2001234
```

☝🏻 This process can take several minutes so be patient.


As instructed by Tykky, add the path to the installation `bin` directory to your `$PATH`:

```bash
export PATH="/projappl/<project>/$USER/tykky-env/bin:$PATH"    # replace <project> with your CSC project, e.g. project_2001234
```

💡 Adding this to your `$PATH` allows you to call Python and all other executables installed by Conda in the same way as you had activated a non-containerized Conda environment.

💡 Tykky installation directories typically include many commonly used commands (e.g. python) that can easily cause conflicts with other applications. It's best not to permanently add any Tykky based installations to your `$PATH`, but rather only add them when needed, e.g. by including the export command to your batch job script.

💭 The above Conda installation would create more than 40k files if installed directly on the parallel file system. Containerizing the environment with Tykky decreases this to less than 200, thus avoiding Lustre performance issues.

💬 To modify an existing Tykky-based Conda environment you can use the `update` keyword of `conda-containerize` together with the `--post-install` option to specify a bash script with commands to run to update the installation. [See more details in Docs CSC](https://docs.csc.fi/computing/containers/tykky/).



