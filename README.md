# FastMVS

GPU-accelerated Multi-View Stereo pipeline for dense point cloud reconstruction from large-scale aerial and UAV imagery.

FastMVS takes a COLMAP workspace as input and produces dense block point clouds for downstream meshing, inspection, and georeferencing workflows. It is designed for production photogrammetry jobs where throughput matters: on a single NVIDIA GPU, FastMVS can process thousands of aerial images in minutes instead of hours.

![FastMVS dense point cloud](figures/pointcloud_3930.jpg)

## Highlights

- **Minute-level throughput**: 3,930-image DJI-Nadir UAV survey processed in 44.6 minutes on one RTX 5090 D.
- **Large speedup vs. COLMAP PatchMatch**: more than 44x faster on the full 3,930-image production pipeline benchmark.
- **COLMAP-compatible input**: works with COLMAP-format workspaces from COLMAP or other SfM tools.
- **CUDA-accelerated depth generation**: uses GPU acceleration for Semi-Global Matching stereo matching.
- **Production-oriented output**: generates dense block point clouds ready for downstream reconstruction workflows.

## Pipeline

FastMVS runs in three stages:

1. **Depth Generation** - CUDA-accelerated SGM stereo matching for per-pair depth maps.
2. **Depth Fusion** - geometric consistency filtering and per-reference depth fusion.
3. **Point Cloud Generation** - multi-view fusion into dense block point clouds.

## Requirements

- Linux x86_64: Ubuntu 22.04 or Ubuntu 24.04
- Windows x64
- NVIDIA GPU with CUDA 12.x
- At least 8 GB GPU VRAM
- COLMAP workspace with sparse reconstruction and undistorted images

## Installation

### Ubuntu

Download the matching `.deb` package from the latest release, then install it locally:

```bash
# Ubuntu 22.04
sudo apt install ./fastmvs_0.1.3-1.ubuntu22.04_amd64.deb

# Ubuntu 24.04
sudo apt install ./fastmvs_0.1.3-1.ubuntu24.04_amd64.deb
```

After installation, the `fast_mvs` executable is available from `/usr/bin`.

### Windows

Download the Windows CUDA archive from the latest release, extract it, then either:

- add the extracted `bin` directory to `PATH`, or
- run `fast_mvs.exe` directly from the extracted folder.

## Quick Start

```bash
fast_mvs \
  -i /path/to/colmap_workspace \
  -o /path/to/output \
  -s 0.5 \
  --mode balanced
```

### Key Parameters

| Parameter | Description | Default |
| --- | --- | --- |
| `-i` | COLMAP project root directory | required |
| `-o` | Output directory | required |
| `--sparse` | Sparse model directory; auto-detects `sparse/0` or `sparse/` when omitted | auto |
| `--images` | Images directory; auto-detected when omitted | auto |
| `-s` | Image scale factor; `0.5` is recommended for aerial surveys | `0.5` |
| `--steps` | Pipeline steps to run, such as `all` or `1,2,3` | `all` |
| `--gpu-workers` | Parallel Step 1 processes; `0` means auto based on VRAM, `1` means serial | `0` |
| `--mode` | Fusion mode: `strict`, `balanced`, or `relaxed` | `balanced` |

### Fusion Modes

| Mode | Use Case |
| --- | --- |
| `strict` | Higher precision with fewer, cleaner points. Use when noise tolerance is low. |
| `balanced` | Default trade-off between precision and completeness. Suitable for most UAV surveys. |
| `relaxed` | Higher recall and denser output. Use when completeness is more important. |

## Benchmarks

Benchmarks below use the same machine unless noted: Intel Core Ultra 7 270K Plus, 30 GiB RAM, NVIDIA RTX 5090 D 24 GB, CUDA 12.8.

### Speed vs. COLMAP PatchMatch

| Scenario | Images | FastMVS Full Pipeline | COLMAP PatchMatch | Speedup |
| --- | ---: | ---: | ---: | ---: |
| Oblique-5Cam UAV survey | 1,535 | 28.4 min | >13 h estimated | >27x |
| DJI-Nadir UAV survey | 3,930 | 44.6 min | >33 h estimated | >44x |
| ISPRS 8K aerial block | 1,260 | ~35 min estimated | - | - |

![FastMVS scaling across datasets](figures/fastmvs_scaling.png)

### Accuracy

| Dataset | Metric | Result |
| --- | --- | ---: |
| ETH3D training set | F-score at 2 cm | 0.343 |
| ETH3D training set | Precision at 2 cm | 0.852 |
| ETH3D training set | Recall at 2 cm | 0.234 |
| DTU dataset | Overall at 2 mm | 0.55 mm |
| DTU dataset | Accuracy | 0.46 mm |
| DTU dataset | Completeness | 0.64 mm |

FastMVS is optimized for UAV surveys with high image overlap and forward or oblique flight patterns. Academic benchmarks such as ETH3D and DTU use capture patterns that differ from typical aerial production workflows, so their completeness characteristics are not always representative of the target use case.

## Downloads

Latest release: **v0.1.3**

- [Ubuntu 22.04 amd64 `.deb`](https://github.com/huluoboge/FastMVS/releases/download/v0.1.3/fastmvs_0.1.3-1.ubuntu22.04_amd64.deb)
- [Ubuntu 24.04 amd64 `.deb`](https://github.com/huluoboge/FastMVS/releases/download/v0.1.3/fastmvs_0.1.3-1.ubuntu24.04_amd64.deb)
- [Windows x64 CUDA 12.8 `.zip`](https://github.com/huluoboge/FastMVS/releases/download/v0.1.3/fastmvs-0.1.3-windows-cuda12.8-protect-x64.zip)
- [All releases](https://github.com/huluoboge/FastMVS/releases)

## Repository

This repository hosts the public FastMVS website and release information. FastMVS itself is proprietary software and the source code is not publicly available.

## Citation

If you use FastMVS in research or production work, please cite:

```bibtex
@software{fastmvs2026,
  title   = {FastMVS: GPU-Accelerated Multi-View Stereo Pipeline},
  author  = {Yang Hu},
  year    = {2026},
  url     = {https://fastmvs.github.io}
}
```

## Contact

For early access, licensing, or commercial inquiries, contact [huyang0909@gmail.com](mailto:huyang0909@gmail.com).
