# EROFS Layer Specification for OCI Packages

EROFS (Enhanced Read-Only File System) is the immutable content-layer format used by RALF packages that must support direct loopback mount, dm-verity integrity, and zero-copy random access on resource-constrained targets (STB, IoT).

## 1. Overview

EROFS uses fixed-size output compression: variable input fits into fixed physical blocks. Compared to SquashFS this minimises I/O amplification and resident memory at the cost of slightly larger images, which matches the runtime profile of RDK devices.

When dm-verity is appended, the hash tree is written immediately after the EROFS image in the same blob. The root hash, salt and hash-tree offset are carried as OCI descriptor annotations (see section 6).

## 2. Media Types

The content-layer media type encodes the compressor so headend and tooling can reject or transcode packages a target kernel cannot mount.

| Media Type | Compressor | Notes |
| :--- | :--- | :--- |
| `application/vnd.rdk.package.content.layer.v1.erofs.lz4+dmverity`  | `lz4hc` (encode) / `lz4` (decode) | Baseline. Supported on every EROFS-enabled kernel. |
| `application/vnd.rdk.package.content.layer.v1.erofs.zstd+dmverity` | `zstd` | Requires `CONFIG_EROFS_FS_ZIP_ZSTD` (Linux 6.x). |
| `application/vnd.rdk.package.content.layer.v1.erofs.nocmpr+dmverity` | none | Uncompressed. Smallest CPU cost, largest image. |

The legacy umbrella type `application/vnd.rdk.package.content.layer.v1.erofs+dmverity` is deprecated and MUST NOT be produced by new tooling. Readers MAY accept it for backwards compatibility and treat it as `erofs.lz4+dmverity`.

## 3. Technical Configuration

Parameters below are normative for RALF EROFS layers. Tooling MUST set them when invoking `erofs-utils`.

| Parameter | Value | Description |
| :--- | :--- | :--- |
| Block size (`-b`) | `4096` | 4 KiB, matches typical page size. |
| Pcluster size (`-C`) | `4096` | Single-page pcluster. Zero read amplification. |
| Compressor (`-z`) | `lz4hc`, `zstd`, or omitted | Determined by media type. `lz4hc` gives better ratio than `lz4` with identical decode cost. |
| UID / GID | `0` / `0` | All entries owned by root. |
| Inline data | enabled | Small regular files inlined to save seeks. |
| Reproducibility | `--ignore-mtime` | Deterministic builds. |
| Extended attributes | filtered (`^xattr-name-filter`) | The normative `mkfs.erofs` examples configure xattr name filtering rather than an explicit global xattr disable. Userspace extractors do not support xattrs yet, and the tar source produced by tooling contains none. |
| Tail packing | disabled | Not yet supported by readers. |
| Dedup / fragments | disabled | Not yet supported by readers. |

Minimum `erofs-utils` version: **1.8.10**.

## 4. Kernel Configuration Requirements

| Symbol | Required for |
| :--- | :--- |
| `CONFIG_EROFS_FS=y` | All EROFS layers. |
| `CONFIG_EROFS_FS_ZIP=y` | Any compressed layer (includes LZ4/LZ4HC decode). |
| `CONFIG_EROFS_FS_ZIP_ZSTD=y` | `erofs.zstd+dmverity` layers. Linux 6.x. |
| `CONFIG_DM_VERITY=y` | Verified mount of the appended hash tree. |

`erofs.nocmpr` layers need only `CONFIG_EROFS_FS` plus `CONFIG_DM_VERITY`.

## 5. Feature & Compatibility Matrix

Minimum supported kernel for RALF: **Linux 5.4**. The legacy v0 on-disk format used in pre-5.4 Android backports is out of scope.

| Feature | Linux 5.4 | Linux 5.15 (LTS) | Linux 6.12+ |
| :--- | :--- | :--- | :--- |
| Status | Mainline | LTS | Modern |
| On-disk format | Compact / extended | Compact / extended | Compact / extended |
| In-place decompression | Yes | Yes | Yes |
| Big pclusters | No | Yes (5.13+) | Yes |
| Zstd support | No | No | Yes (6.x) |

## 6. dm-verity Annotations

Every EROFS content-layer descriptor (any compressor variant) MUST carry the following annotations describing the appended hash tree. Values are lowercase hex strings except `offset`, which is a decimal byte count.

| Annotation Key | Required | Description |
| :--- | :--- | :--- |
| `org.rdk.package.content.dmverity.roothash` | Yes | Hex-encoded SHA-256 root hash of the verity tree. |
| `org.rdk.package.content.dmverity.salt` | Yes | Hex-encoded salt fed to every hash block (≤ 256 bytes). |
| `org.rdk.package.content.dmverity.offset` | Yes | Byte offset of the hash tree inside the layer blob (= size of the EROFS image). |

Verification: mount the first `offset` bytes as the EROFS data device and use the remaining bytes as the verity hash device with the supplied root hash and salt.

## 7. Build Commands (erofs-utils)

Tooling builds an intermediate tarball, then invokes `mkfs.erofs --tar` to produce the image, then appends the dm-verity hash tree, then publishes the layer with the annotations from section 6.

### LZ4 (`erofs.lz4+dmverity`)

```bash
mkfs.erofs \
    -b4096 \
    -C4096 \
    --ignore-mtime \
    -E ^dedupe,^all-fragments,inline_data,^legacy-compress,^xattr-name-filter,^ztailpacking \
    -zlz4hc \
    --all-root \
    --tar=f \
    output.erofs /source/base.tar
```

### Zstd (`erofs.zstd+dmverity`)

```bash
mkfs.erofs \
    -b4096 \
    -C4096 \
    --ignore-mtime \
    -E ^dedupe,^all-fragments,inline_data,^legacy-compress,^xattr-name-filter,^ztailpacking \
    -zzstd \
    --all-root \
    --tar=f \
    output.erofs /source/base.tar
```

### Uncompressed (`erofs.nocmpr+dmverity`)

```bash
mkfs.erofs \
    -b4096 \
    -C4096 \
    --ignore-mtime \
    -E ^dedupe,^all-fragments,inline_data,^legacy-compress,^xattr-name-filter,^ztailpacking \
    --all-root \
    --tar=f \
    output.erofs /source/base.tar
```

After `mkfs.erofs` completes, append the dm-verity hash tree to `output.erofs` (e.g. via `veritysetup format` with `--hash-offset=$(stat -c%s output.erofs)` against the same file, or an equivalent in-process implementation) and record the resulting root hash, salt and offset as descriptor annotations.
