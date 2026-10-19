# PerfBoost: Universal Debian/Ubuntu System & Process Accelerator

**PerfBoost** transforms your Debian-based system so that **any process** running on the machine utilizes available CPU cores, RAM, I/O bandwidth, and network resources to perform faster.

---

## What It Does For All System Processes

### 1. Global Multi-Threaded Environment (`/etc/profile.d/99-perfboost-env.sh`)
Every shell, script, and background job automatically inherits multi-core parallel settings:
- **Build Systems:** `MAKEFLAGS="-j$(nproc)"`, `CMAKE_BUILD_PARALLEL_LEVEL="$(nproc)"`, `CARGO_BUILD_JOBS="$(nproc)"`, `NINJA_STATUS`.
- **Math & Scientific Computing:** `OMP_NUM_THREADS`, `MKL_NUM_THREADS`, `OPENBLAS_NUM_THREADS`, `NUMEXPR_NUM_THREADS` (speeds up Python NumPy, SciPy, PyTorch, C++ OpenMP).
- **Compression:** `XZ_OPT="-T0"` and `ZSTD_NBTHREADS="0"` force multi-threaded compression across tools like `tar` and `dpkg`.

### 2. Multi-Core Drop-in Replacements
- Automatically installs `pigz` and `pbzip2` and symlinks them over single-threaded `gzip` and `bzip2`.
- Any command or archive operation (`tar -czf`, backups, log rotations) instantly utilizes **all CPU cores** instead of just one.

### 3. Maximum Hardware Clock Speed (CPU Governor)
- Sets CPU scaling governor to `performance` across all cores.
- Tunes energy-performance policy (`x86_energy_perf_policy`) to remove latency penalties.

### 4. Kernel, Memory, & I/O Throughput Tuning (`/etc/sysctl.d/99-perfboost.conf`)
- **RAM Swappiness (`vm.swappiness = 10`):** Prioritizes fast physical RAM over slow disk swap.
- **VFS Cache Pressure (`vm.vfs_cache_pressure = 50`):** Keeps filesystem directories and inode caches hot in memory.
- **Process Scheduler (`kernel.sched_autogroup_enabled = 1`):** Isolates foreground desktop/terminal tasks from heavy background work.
- **Scheduler Cache Affinity (`kernel.sched_migration_cost_ns = 5000000`):** Prevents needless thread migration across CPU cores, maximizing L1/L2/L3 cache hits.
- **High-Capacity File Descriptors (`fs.file-max = 2097152`, `limits.conf = 524288`):** Prevents bottlenecks in multi-threaded programs and servers.
- **Network Stack:** Enables TCP BBR congestion control, window scaling, and fast open.

### 5. High-Speed ZRAM Memory Expansion
- Configures in-RAM compressed swap with `zstd` (50% RAM), allowing processes to fit 1.5x–2x more data in memory without touching slow disk swap.

### 6. Process Auto-Booster Daemon (`perfboost.service`)
- A lightweight background daemon monitors running tasks every few seconds.
- Automatically grants elevated CPU priority (`nice -n -5`) and high I/O priority (`ionice -c 2 -n 0`) to active compilers (`gcc`, `clang`, `rustc`), build engines (`make`, `cmake`, `ninja`), runtimes (`python3`, `node`), media encoders (`ffmpeg`, `blender`), and containers (`docker`, `podman`).

### 7. Universal Command Wrapper (`perfboost run <command>`)
Run **any** arbitrary command boosted on demand:
```bash
perfboost run <command> [args...]
```
- Grants immediate CPU core affinity across all detected cores (`taskset`).
- Elevates CPU scheduling priority (`nice`).
- Elevates disk I/O scheduling priority (`ionice`).
- Injects full multi-threading runtime parameters.

---

## Quick Usage Guide

### 1. One-Time System Setup
Run the setup once to apply all system optimizations and enable the daemon:
```bash
sudo ./perfboost.sh setup
```

### 2. Check System Status
View current governor, swappiness, ZRAM status, and optimizer service:
```bash
./perfboost.sh status
```

### 3. Run Any Command Accelerated
```bash
# Compile code using all cores and max priority
perfboost run make

# Transcode video with maximum CPU and disk throughput
perfboost run ffmpeg -i input.mov -c:v libx264 output.mp4

# Run heavy Python / Machine Learning scripts
perfboost run python3 train.py
```

### 4. Package Acceleration (Apt & Pip)
```bash
# Upgrade all system packages using multi-connection parallel downloads
sudo perfboost apt-upgrade

# Install apt packages
sudo perfboost apt-install build-essential git cmake

# Install pip packages in parallel (load-capped)
perfboost pip-install torch numpy pandas flask
```

### 5. Complete Reversible Uninstall
Revert every configuration file, remove symlinks, and restore original system defaults:
```bash
sudo ./perfboost.sh --uninstall
```
