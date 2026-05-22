# Frequently Asked Questions

## Account and Access

**Q: How do I get an account on G2?**

Visit [hpc.utdallas.edu/services](https://hpc.utdallas.edu/services), locate Ganymede 2, and follow the account request link. If unsure, email [circ-assist@utdallas.edu](mailto:circ-assist@utdallas.edu).

**Q: I can't connect to G2. What should I check?**

1. Are you on the UTD campus network (wired Ethernet, CometNet WiFi, or VPN)?
2. Is your NetID password correct?
3. Can you ping `ganymede2.utdallas.edu`?

Off-campus users must connect via [UTD VPN](https://atlas.utdallas.edu/TDClient/30/Portal/Requests/ServiceDet?ID=167) first.

**Q: What is my username?**

Your UTD NetID (same as your email prefix).

---

## Storage

**Q: Where should I put my data?**

| Use case | Location |
|----------|----------|
| Job scripts, small inputs | `~` (home, 50 GB) |
| Large datasets, shared group software | `/groups/<pi-name>` (1–20 TB) |
| Heavy I/O during jobs | `~/scratch` (30 TB, not backed up) |

**Q: How do I check my storage quota?**

```bash
mfsgetquota -H ~
mfsgetquota -H /groups/<pi-name>
```

**Q: My home quota is full. What should I do?**

Move large files to `/groups/<pi-name>` or `~/scratch`. Clean up old job outputs. For a permanent quota increase, open a ticket at the HPC Services page.

**Q: Are files in scratch automatically deleted?**

Yes. Scratch is purged regularly. Do not store important data there long-term — copy results to `~` or `/groups/<pi-name>` after jobs finish. See the [Scratch Space](../getting-started/scratch-space.md) page and the [Policies page](https://hpc.utdallas.edu/policies-and-guidelines/) for details.

**Q: Is my data backed up?**

Home (`~`) and group (`/groups/<pi-name>`) directories are backed up daily. Scratch (`~/scratch`) is **never backed up**.

---

## Jobs and SLURM

**Q: What partitions can I use?**

All users have access to `cpu-preempt` (all CPU nodes) and `gpu-preempt` (all GPU nodes). If your PI owns a condo, you can also use that condo's partition with higher priority.

**Q: My job is stuck in the queue. Why?**

Run `squeue -j <jobid> -l` and check the `REASON` column:

- `Resources`: No idle nodes match your request — wait or reduce your resource request
- `Priority`: Higher-priority jobs are ahead — wait
- `QOSMaxCpuPerUserLimit`: You've hit a per-user CPU cap — reduce your request

**Q: What is preemption?**

Jobs in `cpu-preempt` and `gpu-preempt` may be killed when the condo owner of those nodes submits jobs. Add `#SBATCH --requeue` to your script to automatically restart on another node.

**Q: How long can my jobs run?**

- `cpu-preempt` and `gpu-preempt`: up to 7 days
- `g1mig` condo: up to 2 days
- Other condos: varies — check with `sinfo` or `scontrol show partition <name>`

**Q: Can I run jobs on more than one node?**

Yes. Use `--nodes=N` in your job script. For MPI jobs, SLURM automatically coordinates the launch across nodes.

**Q: How do I request a GPU?**

Add `--gres=gpu:1` to your `#SBATCH` directives and use `--partition=gpu-preempt` (or your condo's GPU partition).

**Q: How do I see why my job failed?**

```bash
cat myjob_<jobid>.err           # Error file
sacct -j <jobid> --format=State,ExitCode,MaxRSS,Elapsed
```

Exit code 137 means out of memory. `TIMEOUT` means wall time exceeded.

---

## Software and Modules

**Q: How do I find available software?**

```bash
module avail
module avail python
module avail | grep -i tensorflow
```

**Q: How do I use CUDA/GPU programs?**

```bash
module unload gnu12
module load cuda/12.4
nvcc -o myprogram myprogram.cu
```

Request a GPU in your job script with `--gres=gpu:1` and use a GPU partition.

**Q: Can I install my own software?**

Yes, in several ways:
- **Python packages**: Use Miniconda — see [Miniconda Guide](../advanced/miniconda.md)
- **R packages**: `install.packages()` in R
- **Other software**: Install to `~/bin/` or `/groups/<pi-name>/software/`
- **Containers**: Use Apptainer for complex environments — see [Containers](../advanced/containers.md)
- **System-wide installation**: Open a ticket at the HPC Services page

**Q: How do I set up a Python environment?**

```bash
module load miniconda
conda create -p /groups/<pi-name>/envs/myenv python=3.12
conda activate /groups/<pi-name>/envs/myenv
conda install numpy scipy matplotlib
```

See the [Miniconda Guide](../advanced/miniconda.md) for full details.

**Q: What Python packages are available without installing anything?**

The base Miniconda installation is available with `module load miniconda`. For any specific packages, create a Conda environment.

---

## Performance

**Q: My job is slow. How can I speed it up?**

Common fixes:
- Use Scratch for heavy I/O (`~/scratch` is up to 10× faster than home)
- Use more cores if your program supports parallelism
- Verify your program is actually using all requested CPUs (`top` inside an interactive job)
- For Python: use NumPy vectorization, multiprocessing, or GPU acceleration — see [Python Optimization](../advanced/python-optimization.md)

**Q: How do I run a GPU-accelerated Python program (PyTorch, TensorFlow)?**

1. Create a Conda environment with PyTorch (CUDA variant)
2. Load `cuda/12.4` and `miniconda` modules
3. Submit to `gpu-preempt` with `--gres=gpu:1`

See [Common Programs](../running-programs/common-programs.md) for example job scripts.

**Q: How do I run many independent tasks efficiently?**

Use [Launcher](../running-programs/launcher.md) or SLURM array jobs. Both let you run hundreds of tasks within a single job allocation without submitting individual jobs.

---

## Open OnDemand

**Q: What is Open OnDemand?**

A web interface to G2 at [g2-ood.circ.utdallas.edu](https://g2-ood.circ.utdallas.edu/). It provides file browsing, a web terminal, and the ability to launch interactive applications without SSH.

**Q: Can I run Jupyter on G2?**

Yes, through Open OnDemand. Log in, click **Interactive Apps**, and launch a Jupyter session. Resources are allocated via SLURM.

---

## Need Help?

- **Email**: [circ-assist@utdallas.edu](mailto:circ-assist@utdallas.edu)
- **HPC Services**: [hpc.utdallas.edu/services](https://hpc.utdallas.edu/services)
