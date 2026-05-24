# VSCode on Ganymede2

## Overview

Visual Studio Code can run on Ganymede2 through **Open OnDemand**, which launches a browser-based VSCode (`code-server`) on a compute node with full CPU, memory, and GPU resources.

!!! warning "Use Open OnDemand, not Remote-SSH"
    The VSCode **Remote-SSH** extension connects to a **login node**, which is limited to ~8 GB of RAM and is not meant for running code or development tools. Indexing a project, opening large files, or running extensions there can crash your session and degrade the login node for everyone. Always use the Open OnDemand method below.

## Why VSCode on Ganymede2?

- Full IDE experience (syntax highlighting, IntelliSense, search) on the cluster
- Integrated terminal for loading modules and submitting jobs
- File browser for the Juno filesystem
- Built-in Git integration and Jupyter notebook support
- No X11 forwarding required

## Launching VSCode via Open OnDemand

1. Open your browser and go to [https://g2-ood.circ.utdallas.edu/](https://g2-ood.circ.utdallas.edu/) (connect to the UT Dallas VPN first if off-campus).
2. Log in with your UT Dallas NetID.
3. Click **Interactive Apps** → **VSCode Server**.
4. Fill in the resource form:
   - **Number of hours** — how long you need (e.g. 4)
   - **Number of cores** — typically 2–4 for interactive work
   - **Memory (GB)** — e.g. 8–16
   - **Partition** — `cpu-preempt` for CPU work, `gpu-preempt` for GPU work
5. Click **Launch** and wait for the status to reach **Running**.
6. Click **Connect to VSCode**.
