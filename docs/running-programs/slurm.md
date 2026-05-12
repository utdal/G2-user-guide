# SLURM Job Scheduler

## What is a Job Scheduler?

SLURM (Simple Linux Utility for Resource Management) is the job scheduler on G2. All compute work must go through SLURM — it allocates nodes to your jobs, runs them when resources are available, and returns output when they finish.

**Never run computationally intensive work directly on login nodes.**

## Ways to Run Programs on G2

There are three ways to run programs:

```
  On login node:   1. Interactively (light work only)

  On compute nodes via SLURM:
                   2. Batch job (sbatch) — recommended
                   3. Interactive session (salloc + srun)
```

### 1. Interactive on Login Node

After SSH login, you are on a login node. This is fine for:

- Editing files
- Compiling code
- Submitting and monitoring jobs
- Light testing

Do not run simulations or data processing here.

### 2. Batch Jobs (Recommended)

Submit a job script with `sbatch`. SLURM runs it on a compute node when resources are available, without requiring you to be online.

### 3. Interactive on Compute Nodes

Use `salloc` to request resources, then `srun` to open a shell on the allocated node:

```bash
# Step 1: Request resources
salloc -p cpu-preempt --mem=30G --ntasks-per-node=4

# Step 2: Start interactive session on the allocated node
srun --pty bash -l
```

Your prompt changes to show the compute node name (e.g., `[netID@c-09-04 ~]$`). Use `exit` when done to release the allocation.

**Common salloc options**:

| Option | Description | Example |
|--------|-------------|---------|
| `-p` | Partition | `-p cpu-preempt` |
| `--mem` | Memory per node | `--mem=30G` |
| `--ntasks-per-node` | CPUs (tasks) per node | `--ntasks-per-node=8` |
| `--nodes` | Number of nodes | `--nodes=1` |
| `-t` | Time limit | `-t 2:00:00` |
| `--gres` | Generic resources (GPUs) | `--gres=gpu:1` |

## G2 Partitions

![G2 cluster partitions — how the 126 nodes are divided across cpu-preempt, gpu-preempt, and individual condo partitions.](../images/slurm-partitions.png)

G2 nodes are organized into **partitions** (also called queues). Choose the partition that matches your job's resource needs.

### Shared Partitions (All Users)

| Partition | Time Limit | Resources | Notes |
|-----------|------------|-----------|-------|
| `cpu-preempt` | 7 days | All CPU compute nodes | Jobs may be preempted by condo owners |
| `gpu-preempt` | 7 days | All GPU compute nodes | Jobs may be preempted by condo owners |

### Condo Partitions (Group Members Only)

Condo owners and their group members have exclusive access to their condo's partition. Examples:

| Partition | Time Limit | Resources |
|-----------|------------|-----------|
| `g1mig` | 2 days | 8 CPU nodes (512 cores, 6.1 TB RAM) |
| `coskunuzer` | 7 days | 2 CPU nodes + 1 H100 GPU node |
| `rotea` | 7 days | 2 CPU nodes + 1 H100 GPU node |
| `chang` | varies | varies |

Run `sinfo` to see all current partitions and their states.

### Job Priority and Preemption

- **Condo partitions**: Only condo owners and their group members can submit jobs here. These jobs run at high priority and are never preempted.
- **cpu-preempt / gpu-preempt**: Any user can submit here. Jobs run first-come, first-served when idle nodes are available. However, when a condo owner submits a job to their own partition, any `cpu-preempt` or `gpu-preempt` job running on those nodes may be **killed**.
  - Add `--requeue` to your script to automatically restart a preempted job on another node.

This design maximizes cluster utilization: condo nodes are available to all users when idle, but condo owners always get priority on their own hardware.

## Batch Job Scripts

A batch job script is a shell script with special `#SBATCH` directives at the top.

### Basic Template

```bash
#!/bin/bash
#SBATCH --job-name=myjob          # Job name
#SBATCH --output=myjob_%j.txt     # Output file (%j = job ID)
#SBATCH --error=myjob_%j.err      # Error file
#SBATCH --partition=cpu-preempt   # Partition
#SBATCH --nodes=1                 # Number of nodes
#SBATCH --ntasks-per-node=8       # CPUs (tasks) per node
#SBATCH --mem=30G                 # Memory per node
#SBATCH --time=3:30:00            # Wall time limit (hh:mm:ss)

# Load required modules
module purge
module load gnu12

# Run your program
./myprogram
```

### SBATCH Directives Reference

| Directive | Description | Example |
|-----------|-------------|---------|
| `--job-name` | Job name | `#SBATCH --job-name=crunch` |
| `--output` | Standard output file | `#SBATCH --output=out_%j.txt` |
| `--error` | Standard error file | `#SBATCH --error=err_%j.txt` |
| `--partition` | Partition/queue | `#SBATCH --partition=cpu-preempt` |
| `--nodes` | Number of nodes | `#SBATCH --nodes=2` |
| `--ntasks-per-node` | CPUs per node | `#SBATCH --ntasks-per-node=64` |
| `--mem` | Memory per node | `#SBATCH --mem=64G` |
| `--time` | Wall time limit | `#SBATCH --time=12:00:00` |
| `--gres` | GPUs | `#SBATCH --gres=gpu:1` |
| `--requeue` | Restart if preempted | `#SBATCH --requeue` |
| `--mail-type` | Email notifications | `#SBATCH --mail-type=END,FAIL` |
| `--mail-user` | Email address | `#SBATCH --mail-user=you@utdallas.edu` |

### Submitting a Job

```bash
sbatch my_job.sh
```

You'll receive a job ID:
```
Submitted batch job 8378
```

## Example Job Scripts

### CPU Job

```bash
#!/bin/bash
#SBATCH --job-name=cpu_sim
#SBATCH --output=cpu_sim_%j.out
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16
#SBATCH --mem=64G
#SBATCH --time=8:00:00

module purge
module load gnu12

./my_simulation
```

### GPU Job

```bash
#!/bin/bash
#SBATCH --job-name=gpu_train
#SBATCH --output=gpu_train_%j.out
#SBATCH --partition=gpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=6
#SBATCH --gres=gpu:1
#SBATCH --mem=30G
#SBATCH --time=3:30:00

module purge
module load cuda/12.4
module load miniconda

conda activate /groups/mygroup/envs/pytorch

python train_model.py
```

### Job with Scratch I/O

```bash
#!/bin/bash
#SBATCH --job-name=sim_model_a
#SBATCH --output=sim_model_a_%j.out
#SBATCH --partition=rotea
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=6
#SBATCH --gres=gpu:1
#SBATCH --mem=30G
#SBATCH --time=3:30:00

SRC=/groups/mygroup/simulation/

# Copy data into Scratch
cd ~/scratch
mkdir batch_job_a
cd batch_job_a
cp $SRC/sim .
cp $SRC/properties_db .
cp $SRC/model_a .

# Load environment and run
module purge
module load miniconda
conda activate /groups/mygroup/envs/sim

./sim model_a results_a

# Copy results back
cp model_a_updated $SRC
cp results_a $SRC

# Clean up Scratch
cd ..
rm -rf batch_job_a
```

### Python Job

```bash
#!/bin/bash
#SBATCH --job-name=python_analysis
#SBATCH --output=python_%j.out
#SBATCH --partition=cpu-preempt
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --mem=8G
#SBATCH --time=2:00:00

module purge
module load miniconda
conda activate /groups/mygroup/envs/myenv

python analyze_data.py --input data.csv --output results.txt
```

### MATLAB Job

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

### Array Job

Run the same script many times with different inputs:

```bash
#!/bin/bash
#SBATCH --job-name=array_job
#SBATCH --output=array_%A_%a.out
#SBATCH --partition=cpu-preempt
#SBATCH --array=1-20
#SBATCH --ntasks-per-node=2
#SBATCH --mem=4G
#SBATCH --time=1:00:00

# $SLURM_ARRAY_TASK_ID is 1..20 for each task
python process.py input_${SLURM_ARRAY_TASK_ID}.dat
```

## How a Batch Job Runs

```
  1. sbatch crunch_job.sh          (on login node)
  2. SLURM allocates resources and
     starts crunch_job.sh on a compute node
  3. Output is written to crunch_job.sh.out
     in the submission directory
```

You can monitor job progress by watching the output file:
```bash
tail -f crunch_out.txt
```

## Monitoring Jobs

```bash
# View your queued and running jobs
squeue --me

# View all jobs in a partition
squeue -p cpu-preempt

# Detailed job information
scontrol show job 8378

# Job accounting (after completion)
sacct -j 8378
sacct -j 8378 --format=JobID,JobName,State,Start,End,Elapsed,MaxRSS
```

**squeue output columns**:

| Column | Meaning |
|--------|---------|
| JOBID | Job identifier |
| PARTITION | Partition name |
| NAME | Job name |
| USER | Username |
| ST | State: R=running, PD=pending, CG=completing |
| TIME | Elapsed time |
| NODES | Number of nodes |
| NODELIST(REASON) | Nodes allocated, or reason for pending |

## Managing Jobs

```bash
# Cancel a job
scancel 8378

# Cancel all your jobs
scancel -u $USER

# Hold a job (prevent it from starting)
scontrol hold 8378

# Release a held job
scontrol release 8378
```

## Checking Partition Resources

```bash
# View all partitions
sinfo

# Detailed partition information
scontrol show partition cpu-preempt

# Node details
scontrol show node c-08-01
```

## Job Dependencies

Chain jobs so that one starts only after another completes:

```bash
# Submit first job
job1=$(sbatch --parsable job1.sh)

# Submit second job dependent on first
sbatch --dependency=afterok:$job1 job2.sh
```

**Dependency types**:

- `afterok`: Start only if previous job succeeded
- `afternotok`: Start only if previous job failed
- `afterany`: Start after previous job ends (any status)

## Troubleshooting

### Job Pending for a Long Time

```bash
squeue -u $USER -l
```

**Common reasons**:

- `Resources`: No idle nodes available; wait in queue
- `Priority`: Other jobs have higher priority
- `QOSMaxCpuPerUserLimit`: Exceeded per-user CPU limit — reduce your request
- `AssocGrpMemLimit`: Exceeded memory limit

### Job Failed Immediately

```bash
cat myjob_8378.err
```

Common causes: wrong module, missing input file, incorrect path, syntax error in script.

### Out of Memory

Job killed with exit code 137, or "Killed" in the error file. Increase `--mem`:

```bash
#SBATCH --mem=64G
```

### Job Times Out

Increase `--time` (up to the partition maximum). Check partition limits:

```bash
sinfo -o "%P %l"
```

## Best Practices

1. **Right-size your resource request** — over-requesting delays your job (no idle nodes to fit it); under-requesting causes failures
2. **Test with small jobs first** — use short time limits during development
3. **Use Scratch for heavy I/O** — see [Scratch Space](../getting-started/scratch-space.md)
4. **Add `--requeue`** if submitting to `cpu-preempt` or `gpu-preempt` to survive preemption
5. **Load modules explicitly** in your script rather than relying on `.bashrc`
6. **Clean up old output files** regularly

## SLURM Environment Variables

Inside a running job, these are automatically set:

| Variable | Value |
|----------|-------|
| `$SLURM_JOB_ID` | Job ID |
| `$SLURM_ARRAY_TASK_ID` | Array task index |
| `$SLURM_CPUS_PER_TASK` | CPUs allocated per task |
| `$SLURM_MEM_PER_NODE` | Memory per node |
| `$SLURM_SUBMIT_DIR` | Directory where sbatch was run |
| `$SLURM_NODELIST` | List of allocated nodes |
| `$SLURM_NTASKS` | Total number of tasks |
| `$SLURM_NNODES` | Number of nodes |

## Next Steps

- [Monitor jobs and cluster state →](advanced-slurm.md)
- [Run common programs →](common-programs.md)
- [Parallelism models →](parallelism.md)
- [High-throughput computing with Launcher →](launcher.md)

## Need Help?

- **Email**: [circ-assist@utdallas.edu](mailto:circ-assist@utdallas.edu)
- **HPC Services**: [hpc.utdallas.edu/services](https://hpc.utdallas.edu/services)
