---
title: "Work on the LIP6 GPU Cluster: A Quick Guide"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - LIP6
  - GPU
  - HPC
comments: true
---

Here’s a compact guide to help you work on the LIP6 GPU cluster: from **connecting**, to **running jobs**, to setting up interactive **Jupyter Notebooks**. 

- [LIP6 Convergence Official Instructions](https://front.convergence.lip6.fr/)

## Connecting to LIP6 Clusters

To streamline your SSH connection, add these settings to your `~/.ssh/config`:

```bash
Host barder.lip6.fr
    User <username>
    ForwardAgent yes

Host cluster.lip6.fr
    User <username>
    ProxyJump barder.lip6.fr
```

Then simply connect:

- For CPU cluster:
  ```bash
  ssh cluster.lip6.fr
  ```

- For GPU cluster:
  ```bash
  ssh front.convergence.lip6.fr
  ```

No need to remember complex jump commands anymore!

## Running Jobs on the GPU Cluster

Once connected, you’ll interact with the SLURM job scheduler. Here's the basic flow:

- **Check the queue**  
  ```bash
  squeue
  ```

- **Submit a job**  
  Prepare a `run.sh` script and submit it:
  ```bash
  sbatch run.sh
  ```

- **Cancel a job**  
  ```bash
  scancel <job_id>
  ```

### Example Job Scripts

#### 1. Running a Python Script (`script.sh`)

Here’s a simple example of a job that runs a Python script:

```bash
#!/bin/bash
#SBATCH --job-name=test_script
#SBATCH --nodes=1
#SBATCH --gpus-per-node=1
#SBATCH --time=1440
#SBATCH --mail-type=ALL
#SBATCH --output=%x-%j.out
#SBATCH --error=%x-%j.err

module purge
module load python/anaconda3
eval "$(conda shell.bash hook)"
conda activate myenv
python main.py --test_nbr_workers=0
```

Just modify `main.py` with your actual Python script name.

#### 2. Running a Jupyter Notebook (`notebook.sh`)

Want to start a Jupyter notebook? Here’s the script:

```bash
#!/bin/bash
#SBATCH --job-name=test_jupyter
#SBATCH --nodes=1
#SBATCH --gpus-per-node=1
#SBATCH --time=60
#SBATCH --mail-type=ALL
#SBATCH --output=%x-%j.out
#SBATCH --error=%x-%j.err

module purge
module load python/anaconda3
eval "$(conda shell.bash hook)"
conda activate myenv
jupyter notebook
```

Then on your local machine, use SSH *port forwarding*:

```bash
ssh -J front.convergence.lip6.fr -L 8888:localhost:8888 node01.convergence.lip6.fr
```

Or if you’re inside the cluster front-end or VSCode terminal:

```bash
ssh -L 8888:localhost:8888 node01.convergence.lip6.fr
```

Now you can open `http://localhost:8888` on your browser to access your notebook remotely!

## Interactive Sessions

Need an interactive terminal on a GPU node?

```bash
srun --nodes=1 --ntasks-per-node=1 --time=01:00:00 --pty bash -i
```

This is great for quick testing or troubleshooting before submitting longer jobs.

