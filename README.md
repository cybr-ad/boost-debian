# PerfBoost (`perfbst`): Universal Debian/Ubuntu System & Process Accelerator

**`perfbst`** transforms your Debian-based system (Debian, Ubuntu, Kali, Mint, Pop!_OS, etc.) so that **any process** running on the machine dynamically utilizes available CPU cores, RAM, I/O bandwidth, and network resources to perform faster and better.

Once installed with `sudo ./perfbst setup`, it is available everywhere simply as **`perfbst`**.

---

## Table of Contents
1. [Core Features](#core-features)
2. [Quick Start](#quick-start)
3. [Step-by-Step Testing & Verification Guide](#step-by-step-testing--verification-guide)
4. [Command Reference](#command-reference)
5. [How It Works Under the Hood](#how-it-works-under-the-hood)
6. [Clean Uninstall](#clean-uninstall)

---

## Core Features

- **Global Multi-Threaded Environment:** Automatically exports parallel build flags (`MAKEFLAGS`, `CMAKE`, `CARGO`) and math libraries (`OMP`, `MKL`, `OPENBLAS`) system-wide.
- **Transparent Multi-Core Compression:** Replaces single-core `gzip` and `bzip2` with multi-core drop-ins (`pigz`, `pbzip2`) so any archive or compression process uses **all CPU cores**.
- **Hardware Performance Governor:** Locks CPU governors across all cores to `performance` and configures `x86_energy_perf_policy` for zero frequency-scaling latency.
- **Kernel & Memory Throughput (sysctl):** Prioritizes RAM over disk swap (`swappiness=10`), keeps filesystem directory cache hot (`vfs_cache_pressure=50`), prevents cross-core CPU cache bouncing, and expands file descriptor limits.
- **ZRAM Compressed Memory Expansion:** Dynamically expands usable RAM by up to 2x at full memory speed with in-RAM `zstd` compression.
- **Process Auto-Booster Daemon (`perfbst.service`):** Continuously monitors the system and automatically promotes CPU priority (`nice -n -5`) and I/O priority (`ionice -c 2 -n 0`) for active compilers, runtimes, encoders, and build tools.
- **Universal Command Runner (`perfbst run <cmd>`):** Supercharge any arbitrary command on demand with explicit CPU affinity, highest nice priority, and maximum I/O bandwidth.
- **Accelerated Package Tools:** Parallel segmented apt downloads via `apt-fast`/`aria2` and concurrent, load-throttled pip package installs.
- **Short Global Command:** Usable anywhere in the terminal as **`perfbst`** (or `perfboost`).
- **100% Reversible:** Tracks every modification with full state logging; `--uninstall` cleanly restores factory defaults.

---

## Quick Start

### 1. One-Time Setup:
Make the script executable and run the setup (this installs `perfbst` to `/usr/local/bin`):
```bash
chmod +x perfbst perfboost.sh
sudo ./perfbst setup
```

### 2. Verify System Status:
From anywhere in your system:
```bash
perfbst status
```

### 3. Run Any Command Accelerated:
```bash
perfbst run make
perfbst run ffmpeg -i input.mov output.mp4
perfbst run python3 train.py
```

---

## Step-by-Step Testing & Verification Guide

Follow these steps to test, benchmark, and verify the performance improvements on your system.

### Test 1: Check Baseline vs Optimized System Status
Run the status check before and after running setup:
```bash
perfbst status
```
**Expected Results after setup:**
- **CPU Governor:** `performance` (all cores at maximum clock speed)
- **RAM Swappiness:** `10` (default was 60; keeps data in physical RAM)
- **Parallel Tools:** `pigz` and `pbzip2` show `Installed ✔`
- **Optimizer Daemon:** `active`

---

### Test 2: Multi-Core Compression Benchmark (Before vs. After)
Compare the time taken by default single-threaded gzip versus the multi-core drop-in:
```bash
# 1. Generate a 300MB test file
dd if=/dev/urandom of=test_sample.raw bs=1M count=300

# 2. Benchmark standard single-threaded gzip directly:
time /bin/gzip -k -f test_sample.raw

# 3. Benchmark multi-core accelerated gzip:
time gzip -k -f test_sample.raw

# 4. Clean up test file
rm -f test_sample.raw test_sample.raw.gz
```
> **Observation:** On modern multi-core processors (4–16 cores), `gzip` runs **3x to 8x faster** and engages all CPU cores simultaneously.

---

### Test 3: Test the `perfbst run` Universal Wrapper
Verify that `perfbst run` automatically applies elevated scheduling priority and CPU core affinity:
```bash
# Launch a test command in background with perfbst run
perfbst run sleep 60 &
PID=$!

# Inspect scheduling priority (NI) and CPU affinity mask:
ps -o pid,ni,cls,comm -p $PID
taskset -p $PID

# Terminate test process
kill $PID
```
> **Observation:** The `NI` (nice priority) column will show **`-5`** (or **`-10`** for root) instead of the default unboosted `0`, ensuring the OS gives it priority execution time over other processes.

---

### Test 4: Verify Multi-Threaded Environment Inheritance
Confirm that any terminal or build tool picks up multi-core parallelism flags automatically:
```bash
source /etc/profile.d/99-perfbst-env.sh
echo "MAKEFLAGS               = $MAKEFLAGS"
echo "CMAKE_BUILD_PARALLEL    = $CMAKE_BUILD_PARALLEL_LEVEL"
echo "CARGO_BUILD_JOBS        = $CARGO_BUILD_JOBS"
echo "OMP_NUM_THREADS         = $OMP_NUM_THREADS"
echo "XZ_OPT                  = $XZ_OPT"
```

---

### Test 5: Verify Background Auto-Booster Daemon
Check that the background service is running and actively optimizing processes:
```bash
sudo perfbst daemon status
```
When running heavy workloads (e.g., `make`, `gcc`, `python3`, `ffmpeg`), the daemon detects them within seconds and automatically elevates their CPU and I/O scheduling classes.

---

## Command Reference

| Command | Description | Needs Root? |
|---|---|---|
| `sudo perfbst setup` | Full system-wide optimization & daemon setup | **Yes** |
| `perfbst status` | Display current performance, governor & daemon status | No |
| `perfbst run <command...> [args]` | Execute **any** arbitrary command with maximum CPU/IO priority | No |
| `sudo perfbst daemon {start\|stop\|restart\|status}` | Manage the background process booster daemon | **Yes** |
| `sudo perfbst apt-upgrade` | Upgrade system packages with parallel segmented downloads | **Yes** |
| `sudo perfbst apt-install <pkg...>` | Install apt packages with multi-connection aria2 acceleration | **Yes** |
| `perfbst pip-install <pkg...>` | Install Python packages concurrently with load throttling | No |
| `sudo perfbst --uninstall` | Revert all system changes and restore original defaults | **Yes** |

---

## How It Works Under the Hood

1. **CPU Frequency Scaling (`/sys/devices/system/cpu/cpu*/cpufreq`):**  
   Switches all CPU cores from power-saving/dynamic downclocking to locked `performance` mode.
2. **Kernel VFS & Virtual Memory (`/etc/sysctl.d/99-perfbst.conf`):**  
   `vm.swappiness = 10` discourages page swapping to disk. `vm.vfs_cache_pressure = 50` caches directory trees and file metadata in memory for instantaneous file operations.
3. **Scheduler Optimization (`kernel.sched_autogroup_enabled = 1`):**  
   Ensures interactive shells and desktop tasks do not stutter even when background jobs use 100% of all CPU cores.
4. **Transparent Compression Redirection (`/usr/local/bin`):**  
   Symlinks `pigz` and `pbzip2` before `/bin` in `$PATH`, making system calls to `gzip` and `bzip2` transparently parallel across all CPU cores.
5. **Dynamic Priority daemon (`perfbst.service`):**  
   Runs in the background at nice level `-10`, scanning for CPU-intensive compilation, encoding, and data jobs to renice them to high priority without user intervention.

---

## Clean Uninstall

To completely revert every configuration file, remove symlinks, stop the daemon, and restore distribution defaults:

```bash
sudo perfbst --uninstall
```

All original files backed up in `/var/backups/perfbst` will be restored, and all state tracking records in `/etc/perfbst` will be removed.
