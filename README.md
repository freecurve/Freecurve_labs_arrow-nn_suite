# Simulation Package with ARROW-NN forcefield

The simulation conda package contains the InterX ARBALEST molecular
dynamics simulation software along with all the necessary database files
to run ARROW-NN molecular simulations. The package supports Slurm and SGE cluster environment, as well as single or multi-GPUs local PC/VM environment. See details in the
documentation and by invoking `arb -h` command.

## System requirements

The software is distributed as a conda package for the linux x86_64
platform. In addition to a 64-bit Linux install, the following 
dependencies must also be installed prior to the package installation:

* conda (anaconda or miniconda are OK)
* rdma-core (called librdmacm by some distros)
* fftw3f

Download TensorFlow library:

```bash
wget https://storage.googleapis.com/tensorflow/libtensorflow/libtensorflow-cpu-linux-x86_64-2.7.0.tar.gz
```

Create directory (name must be exactly that) for the library and extract the archive there:

```bash
mkdir -p /srv/data/permanent/arbalest/TFLIB/libtensorflow-cpu-linux-x86_64-2.7.0
tar xvfz libtensorflow-cpu-linux-x86_64-2.7.0.tar.gz -C /srv/data/permanent/arbalest/TFLIB/libtensorflow-cpu-linux-x86_64-2.7.0
```

Post-processing scripts for computing DDG values require MATLAB > R2017. Otherwise, install pymbar (`conda install pymbar`) to get only BAR analysis for DDG values calculation.

Version of the ARBALEST binary in the package is Cuda GPU enabled, so GPU-enabled hosts (PC/VM) are required. CPU runs are also possible but the NVidia proprietary drivers must be installed in any case. Use the newest version appropriate for your GPU:
    https://www.nvidia.com/Download/index.aspx

Check proper installation of the driver by invoking `nvidia-smi` command.

## Scratch Space Setup

The simulation package needs a dedicated scratch space to store
intermediary simulation outputs. This can be a separate partition or a
folder, and must have at least 20 GB of free space. Perform the following
commands to initialize the scratch space as a folder on your root
partition:

```bash
mkdir -p /state/partition1 && \
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

Activate the conda environment:

```bash
conda activate sim-env
```

Clone the repository in a separate folder:

```bash
git clone <interx_arrow-nn_suite URL> <clone dir>
```

Install dependencies:

```bash
conda install --file <clone dir>/packages.yml -c conda-forge
```

Install the package:

```bash
conda install <clone dir>/simulation-0.56-0.tar.bz2
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

All examples are prepared for GPU runs.

## Solvation energy of monovalent ions in water

This ARBALEST simulation gives NN-corrected solvation free energy of monovalent Na+ and Cl- ions in water. Alchemical TI transition path is the annihilation of the molecule in the solvent box (decoupling of solute molecule in water).

### Configuration Templates

The simulation folder `./INPUT/XML` includes the following ARBALEST configuration file templates:

* `solvation_TEMPLATE.xml`, `solvation_TEMPLATE_short.xml` - ARROW force field with alchemical transition and PIMD 8 beads molecular dynamics (see details in the ARBALEST manual), and NN adjustment.

The solvation template employs the following scheme of simulation for each lambda point (there are 15 points used):

* minimization of the system's initial state
* 10 ps equilibration NVT with Nose-Hoover thermostat
* production PIMD 200 ps (`solvation_TEMPLATE_short.xml`) or 1 ns (`solvation_TEMPLATE.xml`) NPT with Nose-Hoover thermostat and Berendsen barostat, and using two neighboring lambdas window for BAR algorithm.

Periodic boundary condition uses cubic box with 32 Angstroms linear size.

### ARROW-NN Adjustment Parameters

A list of parameter files for NN is in `./INPUT/XML/FF` folder:

* FFNNConfigFloat_NA_pimd_water.xml - for Na+ in water;
* FFNNConfigFloat_CL_pimd_water.xml - for Cl- in water.

Both files contain NN corrections for water in water.

### Simulations

On a cluster, all lambda points are started at once. On a single PC/VM, lambda points are started sequentially (lambda point 0, lambda point 1, etc.).

1. Go to `ions` folder.
2. To make a short CPU run for ions solvation in water box, execute the script:

   ```bash
   ./run_cpu_short.sh
   ```

   Short runs produce 50 ps trajectory.

   Use `run_gpu_short.sh` to run on GPU. It uses the least loaded GPU. If you have several GPUs on your PC and want to run on a specific GPU, use `GPU_ID` environment setting to manually assign it. For example, to run on GPU with ID 2, execute the command:

   ```bash
   GPU_ID=2 ./run_gpu_short.sh
   ```

3. To start production run (1 ns trajectory) on CPU, execute script `./run_cpu.sh`. Use `run_gpu.sh` to run on GPU.

**In production run each lambda point takes at least 4 days on 2080 TI GPU, or more, depending on computational power.**

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

This ARBALEST simulation gives free energy of water solvation in water. Alchemical TI transition path is the annihilation of the molecule in the solvent box (decoupling of solute molecule in water). 
Also water Hvap (heat of vaporization) is calculated separately.

### ARBALEST Configuration Templates

The simulation folder `./INPUT/XML` includes the following Arbalest configuration file templates:

* `solvation_TEMPLATE.xml`, `solvation_TEMPLATE_short.xml` - ARROW force field with alchemical transition and PIMD 8 beads molecular dynamics (see details in the Arbalest manual), and NN adjustment.

The solvation template employs the following scheme of simulation for each lambda point (there are 15 points used):

* minimization of the system's initial state
* 10 ps equilibration NVT with Nose-Hoover thermostat
* production PIMD 50 ps (`solvation_TEMPLATE_short.xml`) or 1 ns (`solvation_TEMPLATE.xml`) NPT with Nose-Hoover thermostat and Berendsen barostat, and using two neighboring lambdas window for BAR algorithm.

Periodic boundary condition uses cubic box with 32 Angstroms linear size.

### ARROW-NN Adjustment Parameters

A list of parameter files for NN is in `./INPUT/XML/FF` folder:

* FFNNConfigFloat_NA_pimd_water.xml - NN corrections for water in water.

### Simulation

On a cluster, all lambda points are started at once. On a single PC/VM, lambda points are started sequentially (lambda point 0, lambda point 1, etc.).

1. Go to `water` folder.
2. To make a short CPU run for water solvation in water box, execute the script:

   ```bash
   ./run_cpu_short.sh
   ```

   Short runs produce 50 ps trajectory.

   Use `run_gpu_short.sh` to run on GPU. It uses the least loaded GPU. If you have several GPUs on your PC and want to run on a specific GPU, use `GPU_ID` environment setting to manually assign it. For example, to run on GPU with ID 2, execute the command:

   ```bash
   GPU_ID=2 ./run_gpu_short.sh
   ```

3. To start production run (1 ns trajectory) on CPU, execute script `./run_cpu.sh`. Use `run_gpu.sh` to run on GPU.

**In production run each lambda point takes at least 3 days on 2080 TI GPU, or more, depending on computational power.**

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

Water Hvap (heat of vaporization) is calculated in this computational experiment.

### ARBALEST Configuration Templates

The simulation folder `./INPUT/XML` includes the following Arbalest configuration file templates:

* `gas.xml` - ARROW force field with 1 ns classical MD of water in vacuum (gas phase for Hvap calculation).
* `gas_PIMD.xml` - ARROW force field with 1 ns PIMD 8 beads of water in vacuum (gas phase for Hvap calculation)
* `water_MD.xml` - ARROW force field with 200 ps classical MD and NN adjustment (for Hvap).
* `water_PIMD8.xml` - ARROW force field with 200 ps PIMD 8 beads molecular dynamics and NN adjustment (for Hvap).

Periodic boundary condition uses cubic box with 32 Angstroms linear size.

### ARROW-NN Adjustment Parameters

A list of parameter files for NN is in `./INPUT/XML/FF` folder:

* FFNNConfigFloat_NA_pimd_water.xml - NN corrections for water in water.

### Simulation

1. Go to `water` folder.

1. Run classical MD on CPU:

   Execute `./run_MD_cpu.sh`. Both gas (5 ns) and liquid phase (200 ps) simulation start.

   To run on GPU, execute `./run_MD_gpu.sh`.

1. Run PIMD on CPU:

   Execute `./run_PIMD_cpu.sh`. Both gas (5 ns) and liquid phase (200 ps) simulation start.

   To run on GPU, execute `./run_PIMD_gpu`.

### Processing results

Get Hvap energy from MD and PIMD trajectory by executing `./calc_hvap.sh` and `./calc_hvap_pimd.sh` respectively.

   Hvap = U_pot(water)/N(water mols in the box) - U_pot(gas) + RT

## Correction for protein-ligand interactions in CDK2 inhibitor mutation

This ARBALEST TI simulation gives NN-corrected free energy of mutation **1h1q -> 1oiy** ligands in CDK2 protein. As previous analysis showed [1], there are some problematic interactions in ARROW2 which have to be corrected to get good agreement with experimental ddG value.

### ARBALEST Configuration Templates

The simulation folder `./INPUT/XML` includes the following Arbalest configuration file templates:

* `Protein_TEMPLATEX_HREX-MPI.xml` - ARROW force field with alchemical transition, lambda replicas exchange, and reservoir molecular dynamics with NN adjustment.

The template employs the following scheme of simulation:

* minimization of the system's initial state
* 100 ps equilibration NVT with Nose-Hoover thermostat
* production HREX run with 800 cycles limited by 120 sec runtime each to get better synchronization between replicas.
* each cycle ends with replica exchanges with randomly selected reservoir state 

Periodic boundary condition uses triclinic water box with linear sizes corresponding to protein with extra 5-10 Angstroms.

More details on reservoir and HREX algorithms can be found in the `RBFE.pdf` tutorial. Files from `examples/Benchmark_Proteins/CDK2` are used.

### ARROW-NN Adjustment Parameters

A list of parameter files for NN is in `./INPUT/XML/FF` folder:

* FFNNConfigFloat.xml - for CDK2 ligands in the protein.

### Production Runs

On a cluster, all lambda replicas are started in MPI mode distributed over cluster nodes. On the single or multi-CPU VM/PC simulation is started also in MPI mode, but cycle for each (or several, depending on number of CPU/GPU) replica is simulated sequentially (lambda point 0, lambda point 1, etc.), then exchange is performed. If you have multi-CPU(GPU) node, specify number of parallel replicas in option `--hrex <number of MPI threads>` in the scripts. Leave `--hrex` empty if you have enough available number of MPI nodes (not less than number of all lambda replicas).

For mutation ddG energy, mutations in both protein and water are simulated.

1. Go to `CDK2` folder.
2. To run mutation in water on CPU, execute the script:

   ```bash
   ./run_water_cpu.sh
   ```

   To run on GPU, execute `./run_water_gpu.sh`.

   To run mutation in protein on CPU, execute the script:

   ```bahs
   ./run_cpu.sh
   ```

   To run on GPU, execute `./run_gpu.sh`.

**Production runs take about 2 days for protein mutation and half a day for water mutation on 2080 TI GPU cluster.**

### Processing Results

Once complete, the results will be available in the ./OUTPUT directory of `CDK2` folder.

To get mutation ddG value from TI and BAR analysis after simulation completion, execute command in `CDK2` folder:

```bash
./analyze_ti_cdk2_mutation.sh
```

First 500 ps of production MD is skipped from analysis by default (`-bt 500` option).

Results will be available in `./dG_cdk2_mutation.txt` file. To get ddG, subtract dG in water value from dG in protein.

**Reference:**

1. Nawrocki G. et al. J. Chem. Theory Comput., 18, p.7751, 2023.
