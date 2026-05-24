# JupyterLab on Ganymede2

There are two ways to run JupyterLab on the cluster. Choose based on your workflow:

| Method | Best for |
|---|---|
| **Open OnDemand** | Most users — no terminal setup, survives browser close, easy reconnect |
| **SSH port forwarding** | Full control over a specific conda environment, GPU work with custom builds, or advanced multi-session workflows |

Both methods run JupyterLab on a **compute node** — never on the login node.

---

## Method 1 — JupyterLab via Open OnDemand (Recommended)

Open OnDemand launches JupyterLab for you through a web form. No SSH tunneling required.

### Step 1 — Open the portal
Navigate to [https://g2-ood.circ.utdallas.edu/](https://g2-ood.circ.utdallas.edu/) and log in with your UT Dallas NetID.

> **Off-campus users:** connect to the UT Dallas VPN before accessing the portal.

### Step 2 — Request a JupyterLab session

1. Click **Interactive Apps** in the top menu bar.
2. Select **Jupyter Lab**.
3. Fill in the resource form:

   | Field | Typical value | Notes |
   |---|---|---|
   | Number of hours | 4 | Max 48 |
   | Number of cores | 2–4 | Scale up for parallel work |
   | Memory (GB) | 8–16 | |
   | Partition | `gpu-preempt` | Use for GPU |
   | GPU | 1 *(optional)* | Only for GPU partitions |
