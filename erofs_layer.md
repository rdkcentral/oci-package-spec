# EROFS Layer Specification for OCI Packages

The **EROFS (Enhanced Read-Only File System)** is utilized as a high-performance, immutable layer format within the OCI package specification. It is designed to bridge the gap between storage efficiency and high-speed random access, particularly in resource-constrained environments like Set-Top Boxes (STB) and IoT devices.

## 1. Overview
EROFS utilizes a **fixed-sized output compression** strategy, fitting variable amounts of input data into fixed-sized physical blocks on the disk. This design minimizes I/O amplification and memory-resident overhead compared to traditional formats like SquashFS, which focus solely on extreme image size reduction.

## 2. Technical Configuration
The following technical parameters are required for generating OCI-compliant EROFS layers using **erofs-utils**. These settings prioritize low-latency "Time to First Byte" and ensure consistent container ownership.

| Parameter | Value | Description |
| :--- | :--- | :--- |
| **Pcluster Size** (`-C`) | `4096` (4KB) | Matches system page size to ensure zero read amplification and minimal CPU overhead. |
| **Compression**  | `lz4hc` or `zstd` | LZ4HC is preferred for speed; Zstd requires modern kernels (6.10+). |
| **UID/GID Mapping** | `0` (root) | All files are forced to 'root' (UID 0, GID 0) for OCI consistency. |
| **Inline Data** | `Enabled` | Regular files are inlined by default to save disk seeks for small data. |
| **Reproducibility** | `Enabled` | Modification times are ignored (`--ignore-mtime`) to ensure deterministic builds. |
| **XATTR Support** | `Disabled` | Extended attributes are disabled for software extractor compatibility. |

## 3. Kernel Configuration Requirements
To support this specification, the following Linux kernel configuration symbols must be enabled in the target system's `config`:

### Core Filesystem Support
* **`CONFIG_EROFS_FS=y`**: Enables the primary EROFS driver.
* **`CONFIG_EROFS_FS_ZIP=y`**: Required to support compressed images (LZ4/Zstd).

### Decompression Backends
* **`CONFIG_LZ4_DECOMPRESS=y`**: Required for `lz4` or `lz4hc` compressed layers.
* **`CONFIG_ZSTD_DECOMPRESS=y`**: Required for `zstd` compressed layers (available in Linux 6.10+).

## 4. Feature & Compatibility Matrix
The minimum supported version for this specification is **Linux 4.9 (via backport)**.

| Feature / Metric | Linux v4.9 (Backport) | Linux v5.4 (Mainline) | Linux v5.15 (LTS) | Linux v6.12+ |
| :--- | :--- | :--- | :--- | :--- |
| **Status** | Unofficial / Legacy | Official Merge | Feature LTS | Modern |
| **On-Disk Format** | Legacy (v0) | Compact/Extended | Compact/Extended | Compact/Extended |
| **In-Place Decomp.** | No (Usually) | Yes | Yes | Yes |
| **Big Pclusters** | No | No | Yes (5.13+) | Yes |
| **Zstd Support** | No | No | No | Yes (6.10+) |

## 5. Build Commands (erofs-utils)

### Target: Legacy (Linux 4.9 Backport)
Uses the legacy on-disk format for compatibility with older Android-style backports.
```bash
mkfs.erofs \
    -b4096 \
    -C4096 \
    --ignore-mtime \
    -E ^dedupe,^all-fragments,inline_data,^legacy-compress,^xattr-name-filter,^ztailpacking \
    -zlz4 \
    --all-root \
    --tar \
    output.erofs /source/directory/base.tar
```
