# How to Log In to the System

## Overview

This guide covers the different methods to access the G2 HPC cluster.

```
  Your laptop / workstation             Ganymede 2 HPC Cluster
  ┌──────────────────────┐              ┌──────────────────────────────────────────┐
  │                      │   SSH        │  ┌──────────────────────────────────────┐│
  │  $ ssh netID@        │─────────────►│  │     Login Node  (g2-l-01)            ││
  │    ganymede2.utd…    │              │  └────────────────┬─────────────────────┘│
  │                      │   HTTPS      │                   │  sbatch / srun        │
  │  Browser:            │─────────────►│                   ▼                      │
  │  g2-ood.circ.utd…    │              │  ┌──────────────────────────────────────┐│
  │                      │              │  │          Compute Nodes               ││
  └──────────────────────┘              │  │  cpu-preempt │ gpu-preempt │ condos  ││
                                        │  └──────────────────────────────────────┘│
                                        └──────────────────────────────────────────┘
```

## Prerequisites: Network Access

Your computer must be connected to the UTD campus network via one of:

- **Wired Ethernet** on campus
- **WiFi** connected to CometNet
- **VPN** connection to campus — see [UTD VPN setup](https://atlas.utdallas.edu/TDClient/30/Portal/Requests/ServiceDet?ID=167) if off-campus

## SSH Login (Command Line)

### Basic Login

The primary method to access G2 is through SSH (Secure Shell).

**Linux/Mac**

Open your terminal and run:
```bash
ssh netID@ganymede2.utdallas.edu
```

Replace `netID` with your UTD NetID.

**Windows (PowerShell or Windows Terminal)**

```bash
ssh netID@ganymede2.utdallas.edu
```

**Recommended terminal programs**:

- Windows: Windows Terminal, MobaXterm, or PuTTY
- Mac: Terminal (pre-installed) or Ghostty
- Linux: terminal (pre-installed) or xterm

### First-Time Login

When logging in for the first time:

1. You'll see a message about host authenticity:
   ```
   The authenticity of host 'ganymede2.utdallas.edu' can't be established.
   Are you sure you want to continue connecting (yes/no)?
   ```

2. Type `yes` and press Enter

3. Enter your UTD NetID password when prompted (characters won't appear as you type)

4. You should see a welcome message and disk quota summary followed by the G2 command prompt

**Login Node Prompt**

After successful login, you'll be on a login node. The prompt will look like:
```
[netID@g2-l-01 ~]$
```

![Screenshot of a terminal window showing the SSH login sequence — host key confirmation prompt, password prompt, and the G2 welcome banner followed by the login node prompt.](../images/screenshot-login-terminal.png)

## Open OnDemand (Web Interface)

Access G2 through your web browser without needing SSH:

1. Navigate to: [https://g2-ood.circ.utdallas.edu/](https://g2-ood.circ.utdallas.edu/)
2. Log in with your UTD NetID and password
3. You'll see the Open OnDemand dashboard

![Screenshot of the Open OnDemand dashboard showing the top navigation bar with Files, Jobs, Clusters, and Interactive Apps menus.](../images/screenshot-ood-dashboard.png)

### Features Available

- **Files**: Browse, upload, and download files
- **Shell Access**: Web-based terminal
- **Interactive Apps**: Launch Jupyter, MATLAB, and other GUI programs
- **Job Composer**: Create and submit SLURM jobs through a GUI
- **Active Jobs**: Monitor your running jobs

!!! tip
    Open OnDemand is ideal if you prefer a graphical interface or are on a network that restricts SSH.

## Data Transfer

Transfer files between your computer and G2 using `scp` or `rsync`:

```bash
# Upload a file to G2
scp myfile.txt netID@ganymede2.utdallas.edu:~/work

# Download a file from G2
scp netID@ganymede2.utdallas.edu:~/work/results.txt ./
```

For large or resumable transfers, use `rsync`:
```bash
rsync -P myfile.txt netID@ganymede2.utdallas.edu:~/scratch/
```

The `-P` flag preserves partially downloaded files and shows progress.

GUI clients such as **FileZilla** also work with G2 over SFTP (`ganymede2.utdallas.edu`, port 22).

See [Storage and Data Transfer](storage.md) for full details.

!!! tip "Skip typing your password every time"
    Set up [SSH key authentication](ssh-keys.md) to log in without a password and enable automation workflows.

## Login Node Best Practices

!!! warning "Do not run computations on login nodes"
    Login nodes are shared among all users. Running intensive jobs on login nodes degrades performance for everyone.

**Acceptable on login nodes**:

- Editing files
- Compiling code
- Submitting and managing SLURM jobs
- Transferring small files
- Light testing

**Not acceptable on login nodes**:

- Running simulations or large data processing
- Long-running computations
- Memory-intensive operations

For computational work, use [compute nodes via SLURM](../running-programs/slurm.md).

## Keeping Your Session Alive

If your connection frequently drops, use `tmux` to keep work running:

```bash
# Start a tmux session
tmux new -s mysession

# Reconnect after disconnection
tmux attach -t mysession
```

## Quick Reference

| Purpose | Command |
|---------|---------|
| Basic login | `ssh netID@ganymede2.utdallas.edu` |
| Login with X11 forwarding | `ssh -X netID@ganymede2.utdallas.edu` |
| Keep connection alive | `ssh -o ServerAliveInterval=60 netID@ganymede2.utdallas.edu` |

## Next Steps

1. [Set up SSH key authentication →](ssh-keys.md)
2. [Explore storage options →](storage.md)
3. [Learn basic Linux commands →](../working-on-g2/linux-commands.md)
4. [Submit your first job →](../running-programs/slurm.md)

## Need Help?

- **Email**: [circ-assist@utdallas.edu](mailto:circ-assist@utdallas.edu)
- **HPC Services**: [hpc.utdallas.edu/services](https://hpc.utdallas.edu/services)
