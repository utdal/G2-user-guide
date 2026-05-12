# Running Common Scientific Programs

## Overview

This page shows how to run frequently used programs on G2, including the modules to load and example job scripts.

## Python

See the [Miniconda Guide](../advanced/miniconda.md) for setting up environments. Once your environment is ready:

```bash
#!/bin/bash
#SBATCH --job-name=python_job
#SBATCH --output=python_%j.out
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --mem=8G
#SBATCH --time=2:00:00

module purge
module load miniconda
conda activate /groups/mygroup/envs/myenv

python my_analysis.py --input data.csv --output results.txt
```

For GPU-accelerated Python (PyTorch, TensorFlow):

```bash
#!/bin/bash
#SBATCH --job-name=pytorch_train
#SBATCH --output=train_%j.out
#SBATCH --partition=gpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --gres=gpu:1
#SBATCH --mem=32G
#SBATCH --time=12:00:00

module purge
module load cuda/12.4
module load miniconda
conda activate /groups/mygroup/envs/pytorch

python train.py
```

## MATLAB

```bash
#!/bin/bash
#SBATCH --job-name=matlab_sim
#SBATCH --output=matlab_%j.out
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=16G
#SBATCH --time=12:00:00

module purge
module load matlab/r2024b

matlab -nodisplay -nosplash -r "run('simulation.m'); exit;"
```

For MATLAB with parallel toolbox, set the thread count to match your allocation:

```matlab
% Inside simulation.m
parpool('local', str2num(getenv('SLURM_NTASKS_PER_NODE')));
```

## Gaussian 16

Gaussian requires specific environment settings. Use the module to handle this automatically:

```bash
#!/bin/bash
#SBATCH --job-name=gaussian_run
#SBATCH --output=gaussian_%j.out
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16
#SBATCH --mem=32G
#SBATCH --time=24:00:00

module purge
module load gaussian/16

# Gaussian reads number of processors from the .com file
# Add %NProcShared=16 to your .com file
g16 myjob.com
```

In your `.com` file:
```
%NProcShared=16
%Mem=28GB
# B3LYP/6-31G* opt

My molecule

...
```

## GROMACS

```bash
#!/bin/bash
#SBATCH --job-name=gromacs_md
#SBATCH --output=gromacs_%j.out
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=32
#SBATCH --mem=64G
#SBATCH --time=24:00:00

module purge
module load gnu12
module load openmpi4
module load gromacs

# Run GROMACS molecular dynamics
gmx mdrun -v -deffnm md -ntmpi 1 -ntomp 32
```

For GPU-accelerated GROMACS:

```bash
#SBATCH --gres=gpu:1
#SBATCH --partition=gpu-preempt

module purge
module unload gnu12
module load cuda/12.4
module load gromacs

gmx mdrun -v -deffnm md -ntmpi 1 -ntomp 4 -gpu_id 0
```

## AMBER

```bash
#!/bin/bash
#SBATCH --job-name=amber_md
#SBATCH --output=amber_%j.out
#SBATCH --partition=gpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --gres=gpu:1
#SBATCH --mem=16G
#SBATCH --time=24:00:00

module purge
module unload gnu12
module load cuda/12.4
module load amber

# GPU-accelerated AMBER
pmemd.cuda -O -i mdin -o mdout -p prmtop -c inpcrd -r restrt -x mdcrd
```

## NAMD

```bash
#!/bin/bash
#SBATCH --job-name=namd_run
#SBATCH --output=namd_%j.out
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16
#SBATCH --mem=32G
#SBATCH --time=24:00:00

module purge
module load namd/3.0.1

namd3 +p16 config.namd > namd_output.log
```

## ORCA

```bash
#!/bin/bash
#SBATCH --job-name=orca_calc
#SBATCH --output=orca_%j.out
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mem=32G
#SBATCH --time=12:00:00

module purge
module load gnu12
module load openmpi4
module load orca

# ORCA must be called with the full path when using MPI
$(which orca) myjob.inp > myjob.out
```

In your ORCA input file, set parallelism:
```
%pal nprocs 8 end
%maxcore 3500
```

## Ansys Fluent

```bash
#!/bin/bash
#SBATCH --job-name=fluent_cfd
#SBATCH --output=fluent_%j.out
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=32
#SBATCH --mem=128G
#SBATCH --time=48:00:00

module purge
module load ansys2025R1/fluent

fluent 3ddp -g -t32 -i myjob.jou
```

## R

```bash
#!/bin/bash
#SBATCH --job-name=r_analysis
#SBATCH --output=r_%j.out
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --mem=8G
#SBATCH --time=4:00:00

module purge
module load gnu14
module load R/4.5.0

Rscript my_analysis.R
```

## Ollama (Local LLM Inference)

Ollama lets you run large language models on G2's GPUs:

```bash
#!/bin/bash
#SBATCH --job-name=ollama_infer
#SBATCH --output=ollama_%j.out
#SBATCH --partition=gpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --gres=gpu:1
#SBATCH --mem=32G
#SBATCH --time=2:00:00

module purge
module load ollama

# Start Ollama server in background
ollama serve &
sleep 5

# Run inference
ollama run llama3 "Summarize the following text: ..."
```

## Compiled Code (C, C++, Fortran)

See [Parallelism Models](parallelism.md) for full compile commands. Quick reference:

```bash
# Serial C
gcc -O2 -o myprogram myprogram.c

# OpenMP (multithreaded)
gcc -fopenmp -O2 -o myprogram_mt myprogram_mt.c

# MPI
mpicc -O2 -o myprogram_mpi myprogram_mpi.c

# CUDA (GPU)
module unload gnu12
module load cuda/12.4
nvcc -O2 -o myprogram_cuda myprogram_cuda.cu
```

## Next Steps

- [Parallelism models →](parallelism.md)
- [High-throughput with Launcher →](launcher.md)
- [Python environments with Miniconda →](../advanced/miniconda.md)

## Need Help?

- **Application support**: [circ-assist@utdallas.edu](mailto:circ-assist@utdallas.edu)
- **HPC Services**: [hpc.utdallas.edu/services](https://hpc.utdallas.edu/services)
