# GPU Computing on Ganymede2

Ganymede2 provides several GPU partitions suited to different workloads, from interactive prototyping to large-scale distributed AI training. This section covers how to set up a GPU environment, select the right partition, and run jobs efficiently.

---
---

## Setting Up a GPU Environment

The recommended approach on Ganymede2 is to create a Conda environment with the GPU libraries you need.

### 1. Load the necessary modules

```bash
module purge
module load gnu/14.1.0
module load miniconda
module load cuda/12.8
```
### 2. Create a Conda environment

```bash
conda create -n gpu-env python=3.11 -y
conda activate gpu-env
```
### 3. Install PyTorch with CUDA support

```bash
# PyTorch with CUDA 12.8
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu126
```

Check that PyTorch sees the GPU after loading your environment on a compute node:

```bash
python -c "import torch; print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0))"
```
### 4. Install other GPU libraries

```bash
# CuPy — GPU-accelerated NumPy-compatible arrays
pip install cupy-cuda12x

# Numba — JIT compilation for GPU kernels
pip install numba

# vLLM — high-throughput LLM inference
pip install vllm
```

See the [Miniconda Guide](../advanced/miniconda.md) for more on managing Conda environments on G2.

---
## Checking GPU Availability Before Submitting

```bash
# See GPU partition status
sinfo -p gpu-preempt

# Detailed view of a specific GPU node
scontrol show node g-01-07
```
Key fields to look for in `scontrol show node`:

- `Gres` — which GPUs are present and their count
- `AllocTRES` vs `CfgTRES` — how many GPUs are currently allocated vs. available
- `State` — `MIXED` means some GPUs are free

```bash
# See how many GPUs are free on each node
sinfo -p gpu-preempt -o "%.20N %.8t %.30G"
```

---

## Quick Sanity Check Job

Before submitting a long training run, verify your environment works with a short test job:

```bash
#!/bin/bash
#SBATCH -J gpu-test
#SBATCH -o gpu_test_%j.out
#SBATCH -p gpu-preempt
#SBATCH -N 1
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=4
#SBATCH --mem=16GB
#SBATCH -t 0:05:00

module purge
module load gnu/14.1.0
module load miniconda
module load cuda/12.8

conda activate gpu-env

python - <<'EOF'
import torch
print(f"PyTorch version: {torch.__version__}")
print(f"CUDA available: {torch.cuda.is_available()}")
print(f"GPU count: {torch.cuda.device_count()}")
for i in range(torch.cuda.device_count()):
    props = torch.cuda.get_device_properties(i)
    print(f"  GPU {i}: {props.name} — {props.total_memory / 1e9:.1f} GB")

# Quick compute test
x = torch.randn(4096, 4096, device='cuda')
y = x @ x.T
print(f"Matrix multiply OK — result shape: {y.shape}")
EOF
```

---

## What's in This Section

- [PyTorch Training Jobs](pytorch-training.md) — single-GPU, multi-GPU DDP, and multi-node distributed training
- [GPU Performance & Monitoring](gpu-performance.md) — profiling, mixed precision, memory optimization
- [AlphaFold 3](alphafold3.md) — protein structure prediction using the shared container and databases

---

## Related Pages

- [Available Software](../working-on-G2/software.md) — full list of GPU-related modules
- [SLURM Job Scheduler](../running-programs/slurm.md) — job scripts, partitions, and scheduling
- [Miniconda](../advanced/miniconda.md) — managing Python environments
- [Containers](../advanced/containers.md) — running containerized GPU workloads (e.g., ColabFold, Stable Diffusion)
