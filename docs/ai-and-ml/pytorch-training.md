# PyTorch Training Jobs

This page covers three GPU training patterns, from a single GPU to many nodes. Each section includes a complete, ready-to-submit SLURM script and a working training script.

---

## Prerequisites

- A Conda environment with PyTorch installed. See [GPU Computing on G2](index.md) for setup instructions.
- Familiarity with SLURM job submission. See [SLURM Job Scheduler](../running-programs/slurm.md).

The training scripts on this page use ResNet-18 on CIFAR-10 as a concrete, reproducible example. Replace the model, dataset, and loss function with your own.

---

## Single GPU Training

The simplest case: one job, one GPU.

### SLURM script

```bash
#!/bin/bash
#SBATCH -J train_single_gpu
#SBATCH -o logs/train_%j.out
#SBATCH -e logs/train_%j.err
#SBATCH -p gpu-preempt                  
#SBATCH -N 1
#SBATCH --ntasks-per-node=1
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=8        # feed the DataLoader with enough CPU workers
#SBATCH --mem=64GB
#SBATCH -t 4:00:00
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user=netID@utdallas.edu

mkdir -p logs

module purge
module load gnu/14.1.0
module load miniconda
module load cuda/12.8

conda activate gpu-env

srun python train_single.py \
    --epochs 20 \
    --batch_size 256 \
    --lr 0.1 \
    --output_dir ./results
```

### Training script — `train_single.py`

```python
#!/usr/bin/env python3
import argparse, json, time
from pathlib import Path
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
from torchvision import datasets, transforms, models

def get_dataloaders(batch_size, num_workers=8):
    transform_train = transforms.Compose([
        transforms.RandomCrop(32, padding=4),
        transforms.RandomHorizontalFlip(),
        transforms.ToTensor(),
        transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2023, 0.1994, 0.2010)),
    ])
    transform_val = transforms.Compose([
        transforms.ToTensor(),
        transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2023, 0.1994, 0.2010)),
    ])
    train_set = datasets.CIFAR10('./data', train=True,  download=True, transform=transform_train)
    val_set   = datasets.CIFAR10('./data', train=False, download=True, transform=transform_val)
    train_loader = DataLoader(train_set, batch_size=batch_size, shuffle=True,
                              num_workers=num_workers, pin_memory=True)
    val_loader   = DataLoader(val_set,   batch_size=batch_size, shuffle=False,
                              num_workers=num_workers, pin_memory=True)
    return train_loader, val_loader

def build_model(num_classes=10):
    model = models.resnet18(weights=None)
    model.fc = nn.Linear(512, num_classes)
    return model

def train_epoch(model, loader, criterion, optimizer, device):
    model.train()
    total_loss, total_samples = 0.0, 0
    t0 = time.time()
    for images, labels in loader:
        images, labels = images.to(device), labels.to(device)
        optimizer.zero_grad()
        loss = criterion(model(images), labels)
        loss.backward()
        optimizer.step()
        total_loss += loss.item() * images.size(0)
        total_samples += images.size(0)
    elapsed = time.time() - t0
    return total_loss / total_samples, total_samples / elapsed

def validate(model, loader, device):
    model.eval()
    correct, total = 0, 0
    with torch.no_grad():
        for images, labels in loader:
            images, labels = images.to(device), labels.to(device)
            _, predicted = torch.max(model(images), 1)
            correct += (predicted == labels).sum().item()
            total += labels.size(0)
    return 100.0 * correct / total

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument('--epochs',     type=int,   default=20)
    parser.add_argument('--batch_size', type=int,   default=256)
    parser.add_argument('--lr',         type=float, default=0.1)
    parser.add_argument('--output_dir', type=str,   default='./results')
    args = parser.parse_args()

    Path(args.output_dir).mkdir(parents=True, exist_ok=True)
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    print(f"Device: {device} — {torch.cuda.get_device_name(0)}")
    print(f"GPU memory: {torch.cuda.get_device_properties(0).total_memory / 1e9:.1f} GB")

    model     = build_model().to(device)
    criterion = nn.CrossEntropyLoss()
    optimizer = optim.SGD(model.parameters(), lr=args.lr, momentum=0.9, weight_decay=5e-4)
    scheduler = optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=args.epochs)
    train_loader, val_loader = get_dataloaders(args.batch_size)

    history = []
    for epoch in range(1, args.epochs + 1):
        loss, throughput = train_epoch(model, train_loader, criterion, optimizer, device)
        accuracy = validate(model, val_loader, device)
        scheduler.step()
        history.append({'epoch': epoch, 'loss': loss,
                        'accuracy': accuracy, 'throughput': throughput})
        print(f"Epoch {epoch:3d} | loss {loss:.4f} | acc {accuracy:.2f}% "
              f"| {throughput:.0f} samples/sec")

    torch.save(model.state_dict(), f'{args.output_dir}/model.pt')
    with open(f'{args.output_dir}/results.json', 'w') as f:
        json.dump(history, f, indent=2)

if __name__ == '__main__':
    main()
```

### Key options to tune

| Option | Guideline |
|---|---|
| `--gres=gpu:1` | Start with one GPU; add more only if it's actually too slow |
| `--cpus-per-task` | Set to 4–8; each CPU worker prefetches batches while the GPU trains |
| `--mem` | A safe rule: 4× the GPU VRAM (e.g., 64 GB for a 16 GB GPU) |
| `batch_size` | Larger batches use more GPU memory but increase throughput; start at 128–256 |

---

## Related Pages

- [GPU Computing on G2](index.md) — GPU partitions, environment setup
- [Common Scientific Programs](../running-programs/common-programs.md) — job scripts for other frameworks
- [Containers](../advanced/containers.md) — running vLLM, AlphaFold, Stable Diffusion in containers
