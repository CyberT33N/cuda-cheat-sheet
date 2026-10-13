# CUDA Cheat Sheet — Docs

Orientation map for everything CUDA-related: installation, toolchain, driver
stack, and GPU-computing practice on this repository's scope (NVIDIA CUDA
platform: toolkit, `nvcc`, runtime/driver model).

## Structure

```
docs/
└── install/
    └── linux/
        └── ubuntu/
            └── ubuntu-25.10.md
```

- `install/` — how-to guides for getting the CUDA toolkit + driver stack onto
  a machine (Diátaxis how-to type; not a tutorial).
- OS parent folders (`linux/`, future: `windows/`, `macos/`), then distro
  (`ubuntu/`, future: `debian/`, `fedora/`), then one file per release
  (`ubuntu-25.10.md`).

## Naming convention (binding for this repo)

- English content, lowercase kebab-case paths,
  pattern `^[a-z0-9]+(-[a-z0-9]+)*(\.[a-z0-9]+)*$`.
- Version tokens keep their dot (`ubuntu-25.10.md`, not `ubuntu-2510.md`) —
  same rule as the `ai-cheat-sheet` model-catalog conventions.
- Every new platform (e.g. `rocm/`, `opencl/`) becomes a sibling repository
  under `gpu-computing/`, never a sub-folder of `cuda-cheat-sheet/`.

## Extension checklist for future agents

1. New OS → `docs/install/<os>/README`-free is fine until 2+ distros exist;
   add `index.md` then.
2. New distro → `docs/install/<os>/<distro>/`.
3. New release guide → `<distro>-<version>.md` with the verification-state
   block (sources + dates; CUDA repo availability changes per release).
4. Facts must be re-anchored against official documentation before writing —
   NVIDIA ships apt repos only for Ubuntu LTS releases; interim releases
   (25.x) need conda/runfile paths. Verify the repo index again before
   documenting a newer release.

## Verification state

- 2026-10 structure pass: NVIDIA CUDA repos index (Ubuntu entries: 12.04→24.04
  LTS, 26.04 LTS — no interim 25.x), CUDA Installation Guide for Linux
  (sections: Package Manager / Driver / Runfile / Conda / Pip Wheels).
- Legend: **verified** = confirmed against official source in this pass.
