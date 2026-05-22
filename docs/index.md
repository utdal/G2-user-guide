# Ganymede 2 (G2) HPC Cluster User Guide

## Introduction

Welcome to Ganymede 2 (G2), a High Performance Computing (HPC) cluster at UT Dallas. This guide provides essential information for new and existing users to get started and make the most of the system.

## What is Ganymede 2?

Ganymede 2 is an HPC cluster consisting of **126 nodes** organized into **26 condos**, owned and operated by individual research groups. In total it provides roughly **8,000 cores**, **116 GPUs**, and **58 TB of RAM**, interconnected via **HDR100 InfiniBand** for fast MPI communication.

Users access shared nodes through the `cpu-preempt` and `gpu-preempt` partitions, in addition to any condo they belong to.

![Ganymede 2 Cluster Diagram](images/Ganymede2.png)

For the full hardware breakdown — node types, per-condo specs, and GPU inventory — see the [Hardware Overview](getting-started/hardware.md).

---

## Quick Start Guide

<div class="grid cards" markdown>

-   **Get Started**

    ---

    Request an account and learn how to log in to G2

    [Request Account](getting-started/account-request.md)

-   **SLURM Job Scheduler**

    ---

    Learn to submit and manage jobs on the cluster

    [SLURM Guide](running-programs/slurm.md)

-   **Storage & Data**

    ---

    Understand directories, quotas, and data transfer

    [Storage Guide](getting-started/storage.md)

-   **Get Help**

    ---

    Contact support and find answers to common questions

    [Support](support/getting-help.md)

</div>

---

## Documentation Overview

### Getting Started
New to G2? Start here to set up your account, log in, and understand the storage system.

- [How to Request an Account](getting-started/account-request.md)
- [How to Log In to the System](getting-started/login.md)
- [SSH Key Authentication](getting-started/ssh-keys.md)
- [Storage and Data Transfer](getting-started/storage.md)
- [Scratch Space](getting-started/scratch-space.md)
- [Hardware Overview](getting-started/hardware.md)

### Working on G2
Learn the essential tools and commands for working on the cluster.

- [Linux Commands Crash Course](working-on-g2/linux-commands.md)
- [Module System](working-on-g2/modules.md)
- [Available Software and Compilers](working-on-g2/software.md)

### Running Programs
Master job submission and execution on G2.

- [SLURM Job Scheduler](running-programs/slurm.md)
- [Monitoring Jobs and Cluster State](running-programs/advanced-slurm.md)
- [Running Common Scientific Programs](running-programs/common-programs.md)
- [Parallelism Models](running-programs/parallelism.md)
- [High Throughput Processing with Launcher](running-programs/launcher.md)

### Advanced Topics
Optimize your workflows with advanced techniques.

- [Containers on G2](advanced/containers.md)
- [Accelerating Python](advanced/python-optimization.md)
- [Virtual Environments with Miniconda](advanced/miniconda.md)

### Support & FAQ
Get help and find answers to common questions.

- [Getting Help](support/getting-help.md)
- [Frequently Asked Questions](support/faq.md)

---

## Important Links

- **HPC Services**: [hpc.utdallas.edu/services](https://hpc.utdallas.edu/services)
- **G2 Resources**: [hpc.utdallas.edu/systems-resources/ganymede2/](https://hpc.utdallas.edu/systems-resources/ganymede2/)
- **Open OnDemand**: [g2-ood.circ.utdallas.edu](https://g2-ood.circ.utdallas.edu/)
- **Support Email**: [circ-assist@utdallas.edu](mailto:circ-assist@utdallas.edu)
- **Orientation Slides**: Available at [hpc.utdallas.edu/systems-resources/ganymede2/](https://hpc.utdallas.edu/systems-resources/ganymede2/)

---

*For the most current information, always refer to the official HPC documentation at hpc.utdallas.edu.*
