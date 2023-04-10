# Simulation Package with ARROW-NN forcefield

The simulation conda package contains the InterX ARBALEST molecular
dynamics simulation software along with all the necessary database files
to run ARROW-NN molecular simulations.

## System requirements

The software is distributed as a conda package for the linux x86_64
platform. In addition to a 64-bit linux install, the following 
dependencies must also be installed prior to the package installation:
  * conda (anaconda or miniconda are OK)
  * rdma-core (called librdmacm by some distros)

To compute DDG values and other analysis, install MATLAB. The path to MATLAB binary should be in the $PATH environment variable. Otherwise, install pymbar (`conda install pymbar`) to get only BAR analysis for DDG values calculation.

If you are using GPU acceleration, the NVidia proprietary drivers
must be installed. Use the newest version appropriate for your GPU:
    https://www.nvidia.com/Download/index.aspx
    
## Scratch Space Setup

The simulation package needs a dedicated scratch space to store
intermediary simulation outputs. This can be a separate partition or a
folder, and must have at least 20 GB of free space. Perform the following
commands to initialize the scratch space as a folder on your root
partition:

```bash
$ mkdir -p /state/partition1
$ chmod o+rwx /state/partition1
```

One can also setup scratch directory with a different path 
setting the TMPDIR environment variable. 
For example, this command in .bashrc file (if your default shell is bash) will set scratch directory to /tmp:  

```bash
export TMPDIR=/tmp
```

## Conda Package Install ##

After installing conda, create a new conda environment:

```bash
$ conda create --name sim-env
```

Clone the repository in a separate folder:

```bash
git clone <interx_arrow-nn_suite URL> <dir>
```

Now install the package into that environment by selecting a channel from the cloned repository.

```bash
$ conda install simulation -c <dir>/channel/ -c conda-forge -n sim-env
```

After installing the package, activate the conda environment:

```bash
$ conda activate sim-env
```

Make sure the latest simulation package version has been installed 
( developers will inform about the current version of the package ): 
 
```bash
$ conda list simulation
```

Run tests to check the simulation package integrity:

```bash
$ arb test 
```
