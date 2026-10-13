# Install CUDA on Ubuntu 25.10

**The one-sentence situation:** Ubuntu 25.10 is an interim release and NVIDIA
publishes its CUDA apt repository **only for LTS releases** (the repo index
contains `ubuntu2404` and `ubuntu2604`, no `ubuntu2510`) — so the classic
"download the cuda-keyring .deb" tutorial from most blogs does not apply here.
Three lawful paths remain.

Hardware note: RTX 4090 = Ada / SM 8.9 — supported by CUDA 12.x and 13.x
toolkits, so the version choice below is about workflow, not capability.

## Decision matrix

| Path | CUDA version | Touches system apt? | Best for | Risk |
|---|---|---|---|---|
| **1. Conda** (recommended) | current (13.x) | no | dev workstations, per-project pinning | low |
| 2. Ubuntu package `nvidia-cuda-toolkit` | 12.4 (pinned in 25.10) | yes (multiverse) | "just needs nvcc" simplicity | old toolkit |
| 3. Runfile installer | current | yes (manual) | air-gapped / non-conda | unsupported-distro caveats |

**Do not** install the `ubuntu2404` cuda-keyring on 25.10 and **do not** mix
NVIDIA-repo drivers with the toolkit on interim releases — community reports
(Reddit r/Ubuntu, Nov 2025) show that combination breaking; treat as
**unverified** but consistent with the repo-support policy above.

## Step 0 — driver first (all paths)

The CUDA **driver** comes from Ubuntu's own repos on 25.10:

```bash
sudo apt update
sudo ubuntu-drivers list --gpg          # see recommended drivers
sudo ubuntu-drivers install             # installs the recommended one
sudo reboot
nvidia-smi                               # must show the RTX 4090 + driver version
```

CUDA runtime needs a driver at least as new as the toolkit's minimum — if
`nvidia-smi` works after reboot, proceed.

## Path 1 — Conda (recommended on 25.10)

Officially documented in the CUDA Installation Guide for Linux, section
"Conda Installation". Packages live at `anaconda.org/nvidia`.

```bash
conda create -n cuda python -y
conda activate cuda
conda install cuda -c nvidia          # full toolkit incl. nvcc
nvcc --version                        # verify
```

Pin an older release (label syntax from the same guide):

```bash
conda install cuda -c nvidia/label/cuda-12.4.0
```

Notes:
- Install into a **dedicated environment**, not `base` (the guide's own note).
- `nvcc` lands in `$CONDA_PREFIX/bin` — active inside the env only.
- The legacy `cudatoolkit` package is **not** the toolkit (no compiler);
  use the `cuda` package.
- Uninstall: `conda remove cuda`.

## Path 2 — Ubuntu's own package

```bash
sudo apt update
sudo apt install nvidia-cuda-toolkit   # 12.4.131 on 25.10 (multiverse)
nvcc --version
```

- Version is pinned to CUDA **12.4** in the 25.10 archive — fine for
  llama.cpp/CUDA builds, not for anything needing 13.x features.
- Multiverse = unsupported-by-default component.
- Host-compiler mismatch with a very new gcc can require NVIDIA's
  host-compiler compatibility approach (guide section 2.3); if `nvcc` errors
  on gcc version, prefer Path 1.
- Outlook: Canonical and NVIDIA announced (Sept 2025) CUDA distribution in
  Ubuntu repos; the plain `sudo apt install cuda-toolkit` line arrives with
  **26.04 LTS**, not 25.10.

## Path 3 — Runfile installer (fallback)

Documented in the guide, section "Runfile Installation".

```bash
# get the .run URL for Linux x86_64 from
#   https://developer.nvidia.com/cuda-downloads
wget https://developer.download.nvidia.com/compute/cuda/<version>/local_installers/<file>.run
sudo sh cuda_<version>_linux.run
```

- The component selector: install the **toolkit**, leave the **driver**
  component off if the Ubuntu driver already works.
- 25.10 may not appear in the supported-distribution list — the runfile is
  distro-agnostic but you run it at your own risk on unsupported releases;
  re-anchor the official support table before relying on this path.

## Verify the whole stack

```bash
nvidia-smi                # driver sees the GPU
nvcc --version            # toolkit compiler

# minimal compile + run test
cat > /tmp/v.cu <<'EOF'
#include <cstdio>
__global__ void k() { printf("hello from GPU\n"); }
int main() { k<<<1,1>>>(); cudaDeviceSynchronize(); return 0; }
EOF
nvcc /tmp/v.cu -o /tmp/v && /tmp/v
```

If `nvcc` is "command not found" under Conda: the env is not activated
(`conda activate cuda`).

## Common pitfalls

1. Copy-pasting an `ubuntu2404` keyring tutorial onto 25.10 → broken apt
   CUDA packages with no clean removal path.
2. Installing both driver **and** toolkit from NVIDIA packages on an interim
   release (see decision matrix warning).
3. Expecting `nvcc` from the old `cudatoolkit` conda package.
4. Forgetting `sudo ubuntu-drivers install` + reboot before any toolkit path.

## Verification state

- 2026-10 research pass (**verified** against official sources):
  - NVIDIA CUDA Installation Guide for Linux — Ubuntu section, Runfile
    section, Conda section (`conda install cuda -c nvidia`, label syntax,
    dedicated-env note).
  - `developer.download.nvidia.com/compute/cuda/repos/` directory index —
    Ubuntu repos present only for LTS releases (24.04, 26.04); no 25.x.
  - `packages.ubuntu.com` questing — `nvidia-cuda-toolkit` 12.4.131 (multiverse).
  - Ubuntu 26.04 release notes — `cuda-toolkit` apt availability lands in
    26.04; Canonical blog (2025-09) on the CUDA distribution partnership.
- **Unverified** (community evidence only, labeled as such): the 2025-11
  breakage reports for NVIDIA packages on 25.10.
- Re-anchor before updating this file for newer releases — repo availability
  is release-dependent.
