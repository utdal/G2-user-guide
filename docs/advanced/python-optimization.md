# Accelerating Python Programs

## Why Python is Slow — and How to Fix It

Python is an interpreted language. Compared to compiled languages (C, Fortran), pure Python code runs 10–100× slower for CPU-bound numerical work. On an HPC cluster this gap matters.

The key insight is that **Python's slow parts can be replaced or bypassed**:

- NumPy, SciPy, and similar libraries call fast C/Fortran code under the hood
- GPU-accelerated libraries (PyTorch, CuPy) run on GPU CUDA cores
- Python's GIL prevents true thread parallelism, but multiprocessing and MPI work around it
- Tools like Numba can JIT-compile Python loops to near-C speed

## Python Parallelism Options

| Method | Scope | When to Use |
|--------|-------|-------------|
| C-backed libraries (NumPy, SciPy) | Implicit threading | Replace Python loops with vectorized calls |
| `multiprocessing.Pool` | Multiple CPU cores (one node) | CPU-bound tasks, no GIL restriction |
| `mpi4py` | Multiple nodes | Distributed memory parallel |
| PyTorch / CuPy / Numba | GPU | Data-parallel, matrix ops, deep learning |
| Launcher / Array jobs | Many independent tasks | Parameter sweeps, batch processing |

## Using NumPy for Vectorization

Replacing explicit Python loops with NumPy operations is the most impactful single optimization for numerical code.

```python
import numpy as np

# Slow: Python loop
result = []
for i in range(1000000):
    result.append(i ** 2)

# Fast: NumPy vectorized (10–100× faster)
x = np.arange(1000000)
result = x ** 2
```

NumPy calls OpenMP-threaded C code internally. For linear algebra (dot products, matrix solves), NumPy uses BLAS/LAPACK which can exploit all cores on a node without any extra code.

## Multiprocessing (One Node)

Python's `multiprocessing` module bypasses the GIL by using separate processes instead of threads:

```python
from multiprocessing import Pool

def process_file(filename):
    # CPU-bound work here
    data = load(filename)
    result = analyze(data)
    return result

if __name__ == '__main__':
    files = ['input_1.dat', 'input_2.dat', ..., 'input_100.dat']
    with Pool(processes=16) as pool:
        results = pool.map(process_file, files)
```

**SLURM script** (request the same number of CPUs):

```bash
#!/bin/bash
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16
#SBATCH --mem=32G
#SBATCH --time=4:00:00

module purge
module load miniconda
conda activate /groups/mygroup/envs/analysis

python my_parallel.py
```

## MPI with mpi4py (Multiple Nodes)

For work that must span multiple nodes, use MPI from Python:

```python
from mpi4py import MPI
import numpy as np

comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

# Each rank processes a different chunk
chunk_size = 10000 // size
start = rank * chunk_size
end = start + chunk_size

local_data = np.arange(start, end, dtype=float)
local_result = np.sum(local_data ** 2)

# Gather results on rank 0
total = comm.reduce(local_result, op=MPI.SUM, root=0)

if rank == 0:
    print(f"Total: {total}")
```

**SLURM script**:

```bash
#!/bin/bash
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=16
#SBATCH --mem=64G
#SBATCH --time=4:00:00

module purge
module load miniconda
module load openmpi4
conda activate /groups/mygroup/envs/mpi

mpirun -np $SLURM_NTASKS python my_mpi_script.py
```

## GPU Acceleration with PyTorch

PyTorch runs computations on GPU automatically when you move tensors to the device:

```python
import torch

device = 'cuda' if torch.cuda.is_available() else 'cpu'
print(f"Using: {device}")

# Move data to GPU
x = torch.randn(10000, 10000, device=device)
y = torch.randn(10000, 10000, device=device)

# Matrix multiply runs on GPU cores
result = torch.matmul(x, y)
```

**SLURM script**:

```bash
#!/bin/bash
#SBATCH --partition=gpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --gres=gpu:1
#SBATCH --mem=32G
#SBATCH --time=8:00:00

module purge
module load cuda/12.4
module load miniconda
conda activate /groups/mygroup/envs/pytorch

python train_model.py
```

## GPU Acceleration with CuPy

CuPy is a drop-in NumPy replacement that runs on GPU:

```python
import cupy as cp

# Same as NumPy, but runs on GPU
x = cp.arange(1000000)
result = x ** 2

# Transfer back to CPU if needed
result_cpu = cp.asnumpy(result)
```

Install: `conda install -c conda-forge cupy`

## JIT Compilation with Numba

Numba compiles Python functions to machine code at runtime:

```python
from numba import jit
import numpy as np

@jit(nopython=True)
def sum_squares(arr):
    total = 0.0
    for x in arr:
        total += x * x
    return total

data = np.random.random(10_000_000)
result = sum_squares(data)   # compiled on first call, fast thereafter
```

For GPU with Numba:

```python
from numba import cuda
import numpy as np

@cuda.jit
def add_kernel(a, b, out):
    i = cuda.grid(1)
    if i < a.shape[0]:
        out[i] = a[i] + b[i]

n = 1_000_000
a = np.ones(n, dtype=np.float32)
b = np.ones(n, dtype=np.float32)
out = np.zeros(n, dtype=np.float32)

threads_per_block = 256
blocks = (n + threads_per_block - 1) // threads_per_block
add_kernel[blocks, threads_per_block](a, b, out)
```

## High Throughput with Launcher

For independent Python tasks (parameter sweeps, batch file processing), use Launcher rather than Python-level parallelism:

```
# tasklist.txt
python process.py input_1.dat output_1.dat
python process.py input_2.dat output_2.dat
...
python process.py input_500.dat output_500.dat
```

See [Launcher Guide](../running-programs/launcher.md) for the full job script.

## Profiling Your Python Code

Before optimizing, find where time is actually spent:

```python
import cProfile
import pstats

with cProfile.Profile() as pr:
    my_function()

stats = pstats.Stats(pr)
stats.sort_stats('cumulative')
stats.print_stats(20)   # top 20 functions by time
```

Or from the command line:
```bash
python -m cProfile -s cumulative my_script.py | head -30
```

For line-by-line profiling, install `line_profiler`:
```bash
conda install line_profiler
kernprof -l -v my_script.py
```

## Summary

1. **First**: Replace Python loops with NumPy/SciPy vectorized operations
2. **CPU-bound, one node**: Use `multiprocessing.Pool`
3. **CPU-bound, many nodes**: Use `mpi4py`
4. **GPU workloads**: Use PyTorch, CuPy, or Numba
5. **Many independent tasks**: Use Launcher or SLURM array jobs

## Next Steps

- [Miniconda for environment management →](miniconda.md)
- [Launcher for parallel tasks →](../running-programs/launcher.md)
- [Containers for complex dependencies →](containers.md)

## Need Help?

- **Email**: [circ-assist@utdallas.edu](mailto:circ-assist@utdallas.edu)
- **HPC Services**: [hpc.utdallas.edu/services](https://hpc.utdallas.edu/services)
