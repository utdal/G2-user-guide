# Parallelism Models

## Introduction

Achieving high performance on G2 requires programs to perform work **simultaneously** on multiple cores. A program that uses only one core at a time — a *serial program* — cannot take advantage of the hundreds of cores available on each node.

This page explains the parallel computing models available on G2 and how to use each one.

## Why Parallelism?

A serial program processes work sequentially on a single core. To go faster, you must split the work across multiple cores or processors running simultaneously — this is called **parallel processing**.

On G2, parallel execution can happen in three ways:

1. **Shared memory (SMP)** — multiple cores on a single node, sharing the same memory
2. **Distributed memory (MPI)** — cores spread across multiple nodes, each with its own memory, communicating via InfiniBand
3. **GPU acceleration** — thousands of small processors on the GPU running in parallel

## Parallel Execution Models

```
Serial         One core, one task at a time

SMP/OpenMP     Multiple cores, one node, shared memory
               ┌────────────────────────────┐
               │ core0 core1 core2 core3    │
               │       shared memory        │
               └────────────────────────────┘

MPI            Multiple cores across multiple nodes
               ┌──────────┐   ┌──────────┐   ┌──────────┐
               │ node 1   │   │ node 2   │   │ node 3   │
               │ rank 0   │◄─►│ rank 1   │◄─►│ rank 2   │
               └──────────┘   └──────────┘   └──────────┘
                        HDR100 InfiniBand

GPU            Thousands of GPU cores on one card
               ┌────────────────────────────┐
               │ CPU + Memory               │
               │  ▲         ▼              │
               │ ┌──────────────────────┐  │
               │ │ GPU (thousands cores)│  │
               │ │ + VRAM               │  │
               │ └──────────────────────┘  │
               └────────────────────────────┘

Hybrid         Combinations of the above
```

## 1. Serial Programs

No parallelism — use for simple programs or testing.

### Compile and Run

```bash
# C
gcc -O2 -o crunch crunch.c
./crunch

# Fortran
gfortran -O2 -o crunch crunch.f90
./crunch

# Python
python crunch.py
```

### SLURM Script

```bash
#!/bin/bash
#SBATCH --job-name=serial_job
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=4G
#SBATCH --time=2:00:00

module purge
module load gnu12

./crunch
```

## 2. Shared Memory Parallel (OpenMP)

Multiple threads share one node's memory. Best for programs that parallelize loops on a single node.

### Compile and Run

```bash
# C with OpenMP
gcc -fopenmp -O2 -o crunch_mt crunch_mt.c
export OMP_NUM_THREADS=4
./crunch_mt

# Fortran with OpenMP
gfortran -fopenmp -O2 -o crunch_mt crunch_mt.f90
export OMP_NUM_THREADS=4
./crunch_mt
```

### SLURM Script

```bash
#!/bin/bash
#SBATCH --job-name=openmp_job
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16       # 16 threads on one node
#SBATCH --mem=32G
#SBATCH --time=4:00:00

module purge
module load gnu12

export OMP_NUM_THREADS=$SLURM_NTASKS_PER_NODE

./crunch_mt
```

### OpenMP Code Example (C)

```c
#include <omp.h>
#include <stdio.h>

int main() {
    #pragma omp parallel
    {
        int tid = omp_get_thread_num();
        printf("Hello from thread %d of %d\n", tid, omp_get_num_threads());
    }
    return 0;
}
```

## 3. Distributed Memory Parallel (MPI)

Multiple processes on multiple nodes communicate by passing messages over InfiniBand. Best for programs that need more cores or memory than one node provides.

!!! warning
    `mpirun` must be run inside a SLURM job — it will not work correctly on login nodes.

### Compile and Run

```bash
# C with MPI
mpicc -O2 -o crunch_mpi crunch_mpi.c
mpirun -np 4 crunch_mpi

# Fortran with MPI
mpif90 -O2 -o crunch_mpi crunch_mpi.f90
mpirun -np 4 crunch_mpi
```

### SLURM Script

```bash
#!/bin/bash
#SBATCH --job-name=mpi_job
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=16       # 16 MPI ranks per node = 64 total
#SBATCH --mem=64G
#SBATCH --time=8:00:00

module purge
module load gnu12
module load openmpi4

mpirun -np $SLURM_NTASKS ./crunch_mpi
```

### MPI Code Example (C)

```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    int rank, size;
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);
    printf("Hello from rank %d of %d\n", rank, size);
    MPI_Finalize();
    return 0;
}
```

## 4. Hybrid (MPI + OpenMP)

Use MPI across nodes and OpenMP within each node. Reduces communication overhead versus pure MPI by keeping per-node work as shared-memory threads.

### Compile and Run

```bash
# C hybrid
mpicc -fopenmp -O2 -o crunch_mpi_mt crunch_mpi_mt.c
export OMP_NUM_THREADS=4
mpirun -np 4 crunch_mpi_mt

# Fortran hybrid
mpif90 -fopenmp -O2 -o crunch_mpi_mt crunch_mpi_mt.f90
export OMP_NUM_THREADS=4
mpirun -np 4 crunch_mpi_mt
```

### SLURM Script

```bash
#!/bin/bash
#SBATCH --job-name=hybrid_job
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=4        # 4 MPI ranks per node
#SBATCH --cpus-per-task=16         # 16 OpenMP threads per rank
#SBATCH --mem=128G
#SBATCH --time=8:00:00

module purge
module load gnu12
module load openmpi4

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

# 4 nodes × 4 ranks × 16 threads = 256 total cores
mpirun -np $SLURM_NTASKS ./crunch_mpi_mt
```

## 5. GPU Accelerated (CUDA)

Offload massively parallel computation to GPU cores. Requires a GPU node.

### Compile and Run

```bash
# Load CUDA (must unload GNU compiler first)
module unload gnu12
module load cuda/12.4

# Compile CUDA C program
nvcc -O2 -o diffusion_cuda diffusion_cuda.cu

# Run (must be on a GPU node)
./diffusion_cuda
```

!!! warning
    Running a CUDA program on a node without a GPU may produce incorrect results. Always request a GPU via `--gres=gpu:1`.

### SLURM Script

```bash
#!/bin/bash
#SBATCH --job-name=cuda_job
#SBATCH --partition=gpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --gres=gpu:1
#SBATCH --mem=16G
#SBATCH --time=4:00:00

module purge
module unload gnu12
module load cuda/12.4

./diffusion_cuda
```

## 6. Hybrid MPI + CUDA

Run MPI across GPU nodes, with CUDA acceleration on each node.

```bash
module unload gnu12
module load cuda/12.4

nvcc -Xcompiler -ccbin=mpicxx hybrid_mpi_cuda.cpp -o hybrid_mpi_cuda

mpirun -n 4 ./hybrid_mpi_cuda
```

## 7. Hybrid OpenMP + MPI + CUDA

Full hybrid: MPI across nodes, OpenMP within nodes, CUDA on GPUs.

```bash
module unload gnu12
module load cuda/12.4

nvcc -Xcompiler "-fopenmp -ccbin=mpicxx" \
  hybrid_openmp_mpi_cuda.cpp -o hybrid_openmp_mpi_cuda

export OMP_NUM_THREADS=4
mpirun -n 4 ./hybrid_openmp_mpi_cuda
```

## 8. High Throughput Computing (HTC)

For embarrassingly parallel workloads — many independent tasks that don't communicate — use [Launcher](launcher.md) or SLURM array jobs instead of MPI.

See [Launcher](launcher.md) for details.

## Choosing the Right Model

```
Is your problem parallel?
├─ No → serial execution
└─ Yes
   ├─ Independent tasks (parameter sweeps, batch processing)?
   │  └─ High-throughput computing (Launcher, array jobs)
   ├─ Fits on one node?
   │  ├─ Yes → OpenMP (shared memory)
   │  └─ No → MPI or Hybrid MPI+OpenMP
   └─ GPU-friendly (matrix ops, deep learning, image processing)?
      └─ GPU (CUDA) or GPU-accelerated libraries (PyTorch, CuPy)
```

| Model | Scope | Communication | When to Use |
|-------|-------|--------------|-------------|
| Serial | 1 core | None | Simple or sequential programs |
| OpenMP | 1 node | Shared memory | Loop parallelism within a node |
| MPI | Many nodes | Message passing | Scale beyond one node |
| Hybrid | Many nodes | MPI + shared memory | Large scale, memory-intensive |
| CUDA | 1+ GPUs | GPU kernel | Data-parallel, deep learning |
| HTC | Many jobs | None | Independent parameter sweeps |

## Python Parallelism

Python needs special handling: its Global Interpreter Lock (GIL) prevents true thread parallelism for CPU-bound work. The workarounds map onto the models above — `multiprocessing` and `mpi4py` for CPU parallelism, and PyTorch/CuPy/Numba for GPU. See [Accelerating Python](../advanced/python-optimization.md) for code examples and guidance.

## Compilers Available on G2

For the full list of compilers, MPI implementations, and their module names, see [Available Software and Compilers](../working-on-g2/software.md).

## Next Steps

- [High-throughput computing with Launcher →](launcher.md)
- [Python acceleration →](../advanced/python-optimization.md)
- [GPU job monitoring →](advanced-slurm.md)

## Need Help?

- **Parallelization advice**: [circ-assist@utdallas.edu](mailto:circ-assist@utdallas.edu)
- **HPC Services**: [hpc.utdallas.edu/services](https://hpc.utdallas.edu/services)
