# Simulation Package with ARROW-NN forcefield

The simulation conda package contains the InterX ARBALEST molecular
dynamics simulation software along with all the necessary database files
to run ARROW-NN molecular simulations. The package supports Slurm and SGE cluster environment, as well as single or multi-CPUs local PC/VM environment.

## System requirements

The software is distributed as a conda package for the linux x86_64
platform. In addition to a 64-bit linux install, the following 
dependencies must also be installed prior to the package installation:
  * conda (anaconda or miniconda are OK) – version higher than 22.11.0
  * rdma-core (called librdmacm by some distros)

Post-processing scripts for computing DDG values require MATLAB > R2017. Otherwise, install pymbar (`conda install pymbar`) to get only BAR analysis for DDG values calculation.

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
mkdir -p /state/partition1
chmod o+rwx /state/partition1
```

One can also setup scratch directory with a different path setting the TMPDIR environment variable.
For example, this command in .bashrc file (if your default shell is bash) will set scratch directory to /tmp:  

```bash
export TMPDIR=/tmp
```

## Conda Package Install ##

After installing conda, create a new conda environment:

```bash
conda create --name sim-env
```

Clone the repository in a separate folder:

```bash
git clone <interx_arrow-nn_suite URL> <clone dir>
```

Now install the package into that environment by selecting a channel from the cloned repository.

```bash
conda install simulation -c file://<clone dir>/nn/ -c conda-forge -n sim-env
```

After installing the package, activate the conda environment:

```bash
conda activate sim-env
```

Run tests to check the simulation package integrity:

```bash
arb test 
```

## Running ARROW-NN examples

Go to sub-folder

```bash
cd $CONDA_PREFIX/opt/interx/examples/ARROW-NN
```

## Solvation energy of monovalent ions in water

This ARBALEST simulation gives NN-corrected solvation free energy of monovalent Na+ and Cl- ions in water. Alchemical TI transition path is the annihilation of the molecule in the solvent box (decoupling of solute molecule in water).

On a cluster, all lambda points are started at once. Runs on all lambda points on the single PC (not on cluster) are started sequentially (lambda point 0, then lambda point 1, etc.)  and run is started on the least loaded GPU.

1. Go to `ions` folder.
2. To make a short run for ions solvation in water box, execute the script:

   ```bash
   ./run_ions_short.sh
   ```

   Short runs produce 50 ps trajectory.

   If you have several GPUs on your PC and want to run on a specific GPU, use `GPU_ID` environment setting to manually assign it. For example, to run on GPU with ID 2, execute the command:

   ```bash
   GPU_ID=2 ./run_ions_short.sh
   ```

3. To start production run, execute script `./run_ions.sh`. Production run produces 1 ns trajectory.

**Note that in production run each lambda point takes at least 4 days on 2080 TI GPU, or more, depending on computational power.**

### Processing Results

Once complete, the results will be available in the `./OUTPUT` directory of `ions` folder.

To get solvation dG value from TI and BAR analysis after simulation completion, execute command in `ions` folder:

```bash
./analyze_ti_ions.sh
```

First 10 ps of production MD is skipped from analysis by default (`-bt 10` option).

Results will be in `./dG_solv_ions.txt` file.

For long trajectory analysis, run `./analyze_ti_ions_long.sh`. Results will be available in `./dG_solv_ions_longtrr.txt` file.

## Solvation energy of water in water

This ARBALEST simulation gives free energy of water solvation in water. Alchemical TI transition path is the annihilation of the molecule in the solvent box (decoupling of solute molecule in water). Also water Hvap (heat of vaporization) is calculated separately.

On a cluster, all lambda points are started at once. Runs on all lambda points on the single PC (not on cluster) are started sequentially (lambda point 0, then lambda point 1, etc.)  and run is started on the least loaded GPU.

1. Go to `water` folder.
2. To make a short run for water solvation in water box, execute the script:

   ```bash
   ./run_water_short.sh
   ```

   Short runs produce 50 ps trajectory.

   If you have several GPUs on your PC and want to run on a specific GPU, use `GPU_ID` environment setting to manually assign it. For example, to run on GPU with ID 2, execute the command:

   ```bash
   GPU_ID=2 ./run_water_short.sh
   ```

3. To start production run, execute script `./run_water.sh`. Production run produces 1 ns trajectory.

**Note that in production run each lambda point takes at least 3 days on 2080 TI GPU, or more, depending on computational power.**

### Processing Results

Once complete, the results will be available in the `./OUTPUT` directory of `water` folder.

To get solvation dG value from TI and BAR analysis after simulation completion, execute command in `water` folder:

```bash
./analyze_ti_water.sh
```

First 10 ps of production MD is skipped from analysis by default (`-bt 10` option).

Results will be available in `./dG_solv_water.txt` file.

For long trajectory analysis, run `./analyze_ti_water_long.sh`. Results will be available in `./dG_solv_water_longtrr.txt` file.

## Water heat of vaporization

1. Classical MD

   Execute `./run_water_MD.sh` in `water` folder. Both gas (5 ns) and liquid phase (200 ps) simulation start.

2. PIMD

   Execute `./run_water_PIMD.sh` in `water` folder. Both gas (5 ns) and liquid phase (200 ps) simulation start.

3. Get Hvap energy from MD and PIMD trajectory by executing `./calc_hvap.sh` and `./calc_hvap_pimd.sh` respectively.

   Hvap = U_pot(water)/N(water mols in the box) - U_pot(gas) + RT

## Correction for protein-ligand interactions in CDK2 inhibitor mutation

This ARBALEST TI simulation gives NN-corrected free energy of mutation **1h1q -> 1oiy** ligands in CDK2 protein. As previous analysis showed [1], there are some problematic interactions in ARROW2 which have to be corrected to get good agreement with experimental ddG value.

On a cluster, all lambda replicas are started in MPI mode distributed over cluster nodes. Runs on all lambda points on the single PC or multi-CPU VM/PC are started also in MPI mode, but cycle for each (or several, depending on number of CPU/GPU) replica is simulated sequentially (lambda point 0, then lambda point 1, etc.), then exchange is performed. To specify number of parallel replicas, use option `--hrex <N>` in scripts.

For mutation ddG energy, mutations in both protein and water are simulated.

1. Go to `CDK2` folder.
2. To run water mutation, execute the script:

   ```bash
   ./run_cdk2_mutation_water.sh
   ```

   To run protein mutation, execute the script from t:

   ```bahs
   ./run_cdk2_mutation.sh
   ```

Production runs take about 2 days for protein mutation and half a day for water mutation on 2080 TI GPU cluster.

### Processing Results

Once complete, the results will be available in the ./OUTPUT directory of `CDK2` folder.

To get mutation ddG value from TI and BAR analysis after simulation completion, execute command in `CDK2` folder:

```bash
./analyze_ti_cdk2_mutation.sh
```

First 500 ps of production MD is skipped from analysis by default (`-bt 500` option).

Results will be available in `./dG_cdk2_mutation.txt` file. To get ddG, subtract water dG value from protein dG.

Reference:

1. Nawrocki G. et al. J. Chem. Theory Comput., 18, p.7751, 2023.
