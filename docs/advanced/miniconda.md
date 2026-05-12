# Virtual Environments with Miniconda

## Why Miniconda?

Python programs depend on many libraries, and different projects often require incompatible versions. Miniconda lets you create isolated **environments** — each with its own Python version and packages — without interfering with other projects or the system Python.

On G2, Miniconda is available as a module and is the **recommended way** to manage Python software.

## Key Recommendations

- **Use `/groups/<pi-name>`** for your environments — this lets group members share them
- **Do not install in `~/scratch`** — scratch is purged regularly
- **Use `-p <path>` when creating environments** to install in a specific directory
- **Avoid using `pip` unless `conda` cannot install the package** — mixing conda and pip can break environments
- If you must use `pip`, only do so inside a conda environment after conda packages are installed

## Getting Started

### Load Miniconda

```bash
module load miniconda
```

### Initialize Conda (First Time Only)

```bash
conda init bash
source ~/.bashrc
```

You only need to do this once. After initialization, `conda activate` will work in future sessions.

## Creating Environments

### In Your Group Directory (Recommended)

```bash
module load miniconda

# Create environment at a specific path
conda create -p /groups/<pi-name>/envs/myenv python=3.12

# Activate
conda activate /groups/<pi-name>/envs/myenv
```

All group members can then activate and use this environment.

### In Your Home Directory

```bash
conda create -n myenv python=3.12
conda activate myenv
```

This installs to `~/.conda/envs/myenv`. Keep in mind your home directory has a 50 GB quota.

### Specifying Python Version

```bash
conda create -p /groups/<pi-name>/envs/py311 python=3.11
conda create -p /groups/<pi-name>/envs/py312 python=3.12
```

## Installing Packages

```bash
conda activate /groups/<pi-name>/envs/myenv

# Install from default channels
conda install numpy scipy matplotlib

# Install from conda-forge
conda install -c conda-forge tensorflow

# Install PyTorch (GPU version)
conda install pytorch torchvision torchaudio pytorch-cuda=12.4 -c pytorch -c nvidia

# Install multiple packages at once
conda install numpy pandas scikit-learn jupyter

# List installed packages
conda list

# Remove a package
conda remove scipy
```

### Using pip (When Necessary)

```bash
# Only after conda packages are installed
pip install --upgrade MDAnalysis
pip install some_package_not_in_conda
```

## Using Environments in Job Scripts

Always load Miniconda and activate your environment in your batch script:

```bash
#!/bin/bash
#SBATCH --job-name=python_job
#SBATCH --output=python_%j.out
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --mem=16G
#SBATCH --time=4:00:00

module purge
module load miniconda

conda activate /groups/<pi-name>/envs/myenv

python my_analysis.py
```

For GPU jobs, also load CUDA before Miniconda:

```bash
module purge
module load cuda/12.4
module load miniconda

conda activate /groups/<pi-name>/envs/pytorch_env

python train.py
```

## Managing Environments

```bash
# List all environments
conda env list

# Activate an environment
conda activate /groups/<pi-name>/envs/myenv

# Deactivate
conda deactivate

# Remove an environment
conda env remove -p /groups/<pi-name>/envs/myenv

# Clone an environment
conda create -p /groups/<pi-name>/envs/myenv_v2 --clone /groups/<pi-name>/envs/myenv
```

## Exporting and Sharing Environments

```bash
# Export environment to a YAML file
conda env export > environment.yml

# Recreate from YAML
conda env create -p /groups/<pi-name>/envs/restored_env -f environment.yml
```

This is useful for sharing exact reproducible environments with collaborators or for documentation.

## Installing Popular Packages

### PyTorch (GPU)

```bash
conda activate /groups/<pi-name>/envs/pytorch_env
conda install pytorch torchvision torchaudio pytorch-cuda=12.4 -c pytorch -c nvidia
python -c "import torch; print(torch.cuda.is_available())"
```

### TensorFlow

```bash
conda create -p /groups/<pi-name>/envs/tf_env python=3.11
conda activate /groups/<pi-name>/envs/tf_env
conda install -c conda-forge tensorflow
```

### Data Science Stack

```bash
conda create -p /groups/<pi-name>/envs/datascience python=3.12
conda activate /groups/<pi-name>/envs/datascience
conda install numpy pandas matplotlib seaborn scikit-learn scipy jupyter
```

### Bioinformatics

```bash
conda create -p /groups/<pi-name>/envs/bio python=3.11
conda activate /groups/<pi-name>/envs/bio
conda install -c bioconda -c conda-forge biopython samtools bcftools
```

## Troubleshooting

### Conda Activate Fails

If `conda activate` says "conda command not found":
```bash
module load miniconda
conda init bash
source ~/.bashrc
```

### Package Conflicts

If conda cannot resolve dependencies:
```bash
# Use mamba for faster, better dependency resolution
conda install -c conda-forge mamba
mamba install conflicting_package
```

### Environment Takes Too Much Space

Check usage:
```bash
du -sh /groups/<pi-name>/envs/myenv
```

Remove unused packages:
```bash
conda clean --all
```

### Wrong Python Version Active

```bash
which python
python --version
conda activate /groups/<pi-name>/envs/myenv
which python
```

## Conda Cheat Sheet

```bash
module load miniconda                          # Load conda
conda create -p /groups/group/envs/myenv python=3.12  # Create environment
conda activate /groups/group/envs/myenv        # Activate
conda install numpy scipy                      # Install packages
conda install -c conda-forge package           # From conda-forge
conda list                                     # List packages
conda deactivate                               # Deactivate
conda env list                                 # List all environments
conda env export > env.yml                     # Export environment
conda env create -f env.yml                    # Recreate from file
conda remove package                           # Remove a package
conda env remove -p /groups/group/envs/myenv   # Delete environment
```

## Next Steps

- [Python optimization →](python-optimization.md)
- [Running Python jobs →](../running-programs/common-programs.md)
- [Containers for complex dependencies →](containers.md)

## Need Help?

- **Email**: [circ-assist@utdallas.edu](mailto:circ-assist@utdallas.edu)
- **HPC Services**: [hpc.utdallas.edu/services](https://hpc.utdallas.edu/services)
