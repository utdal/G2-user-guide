# Scratch Space

## What is Scratch?

Scratch (`~/scratch`) is G2's very high-performance storage system, designed specifically for the heavy I/O demands of running batch jobs. It is up to **10× faster** than the Home or Group directories for large I/O operations.

| Property | Value |
|----------|-------|
| Path | `~/scratch` |
| Quota | 30 TB (soft limit) |
| Backup | **Never** |
| Purge policy | Files are deleted regularly |
| Access | Private to your account |

!!! danger "Scratch is not backed up and is purged regularly"
    Never store data you cannot afford to lose in scratch. Always copy important results to your Home or Group directory after a job completes.

## Why Use Scratch?

Running programs have much heavier I/O needs than file browsing or editing. When a job reads and writes large amounts of data:

- Scratch handles this at full speed — it is purpose-built for high-throughput I/O
- Home (`~`) and Group (`/groups/<pi-name>`) directories share bandwidth across all users and are not optimized for this workload
- Using Scratch for job I/O can dramatically reduce your job's wall time

## Scratch Workflow

![Scratch data lifecycle — stage data in before the job, process on scratch during the job, copy important results to Home/Group after, then clean up scratch.](../images/scratch-data-lifecycle.png)

For any job with significant I/O, follow this four-step pattern inside your batch script:

```
1. Copy data IN at the start
2. Read from and write to Scratch during the job
3. Copy data OUT at the end
4. Clean up before leaving
```

### Example Batch Script with Scratch

```bash
#!/bin/bash
#SBATCH --job-name=my_simulation
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16
#SBATCH --mem=64G
#SBATCH --time=4:00:00
#SBATCH --partition=cpu-preempt
#SBATCH --output=sim_output.txt

# 1. Copy data into Scratch
WORKDIR=~/scratch/sim_${SLURM_JOB_ID}
mkdir -p $WORKDIR
cp ~/projects/sim/input.dat $WORKDIR/
cp ~/projects/sim/run_sim     $WORKDIR/
cd $WORKDIR

# Load required modules
module load gnu12

# 2. Run your program — reads/writes happen in Scratch
./run_sim input.dat results.dat

# 3. Copy results back to persistent storage
cp results.dat ~/projects/sim/results_$(date +%Y%m%d_%H%M%S).dat

# 4. Clean up Scratch
cd ~
rm -rf $WORKDIR
```

`$SLURM_JOB_ID` is the unique SLURM job ID, creating a unique working directory per job to avoid conflicts when running multiple jobs simultaneously.

## Scratch Storage Management Policy

Overall scratch space is shared across all users, so it is important to keep it clean. G2 enforces a **scratch purge policy**:

- Files in `~/scratch` that have not been accessed for an extended period are **automatically deleted**
- The purge policy details are available at [hpc.utdallas.edu/policies-and-guidelines/](https://hpc.utdallas.edu/policies-and-guidelines/)

**Best practices**:

- Use scratch only during batch jobs, then delete afterwards
- Do not use scratch for long-term storage — use `~` or `/groups/<pi-name>` instead
- Do not install software in `~/scratch`

## Checking Scratch Usage

```bash
# Current usage
du -sh ~/scratch

# List large files
find ~/scratch -size +1G -ls

# Find old files (not accessed in 14+ days)
find ~/scratch -atime +14 -type f
```

## Next Steps

- [Storage overview →](storage.md)
- [Submit jobs that use Scratch →](../running-programs/slurm.md)

## Need Help?

- **Email**: [circ-assist@utdallas.edu](mailto:circ-assist@utdallas.edu)
- **Storage policy**: [hpc.utdallas.edu/policies-and-guidelines/](https://hpc.utdallas.edu/policies-and-guidelines/)
