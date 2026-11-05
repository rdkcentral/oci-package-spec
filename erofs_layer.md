# EROFS Layer Specification for OCI Packages

EROFS (Enhanced Read-Only File System) is the immutable content-layer format used by RALF packages that must support direct loopback mount, dm-verity integrity, and zero-copy random access on resource-constrained targets (STB, IoT).

## 1. Overview

EROFS uses fixed-size output compression: variable input fits into fixed physical blocks. Compared to SquashFS this minimises I/O amplification and resident memory at the cost of slightly larger images, which matches the runtime profile of RDK devices.

When dm-verity is used, the hash tree is appended immediately after the EROFS image. The root hash, salt and hash-tree offset are carried as OCI descriptor annotations (see [section 6](#6-dm-verity-annotations)). If the layer is encrypted with LUKS2, see [Offset Semantics](#offset-semantics) for how the offset is interpreted.

## 2. Media Types

The content-layer media type encodes the compressor so headend and tooling can reject or transcode packages a target kernel cannot mount.

| Media Type | Compressor | Notes |
| :--- | :--- | :--- |
| `application/vnd.rdk.package.content.layer.v1.erofs.lz4+dmverity`  | `lz4hc` (encode) / `lz4` (decode) | Baseline. Supported on every EROFS-enabled kernel. |
| `application/vnd.rdk.package.content.layer.v1.erofs.zstd+dmverity` | `zstd` | Requires `CONFIG_EROFS_FS_ZIP_ZSTD` (Linux 6.x). |
| `application/vnd.rdk.package.content.layer.v1.erofs.nocmpr+dmverity` | none | Uncompressed. Smallest CPU cost, largest image. |
| `application/vnd.rdk.package.content.layer.v1.erofs.lz4+dmverity+encrypted` | `lz4hc` (encode) / `lz4` (decode) | LZ4-compressed EROFS + dm-verity representation stored inside a LUKS2/dm-crypt encrypted container. dm-verity metadata describes the decrypted EROFS + dm-verity representation. Requires the encrypted-layer kernel options in [section 4](#4-kernel-configuration-requirements).|
| `application/vnd.rdk.package.content.layer.v1.erofs.zstd+dmverity+encrypted` | `zstd` |  Zstd-compressed EROFS + dm-verity representation stored inside a LUKS2/dm-crypt encrypted container. Requires `CONFIG_EROFS_FS_ZIP_ZSTD` (Linux 6.x) and the encrypted-layer kernel options in [section 4](#4-kernel-configuration-requirements). dm-verity metadata describes the decrypted representation. |
| `application/vnd.rdk.package.content.layer.v1.erofs.nocmpr+dmverity+encrypted` | none | Uncompressed EROFS + dm-verity representation stored inside a LUKS2/dm-crypt encrypted container. Requires the encrypted-layer kernel options in [section 4](#4-kernel-configuration-requirements). dm-verity metadata describes the decrypted representation. |

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
| `CONFIG_EROFS_FS_ZIP=y` | Any compressed layer (includes LZ4/LZ4HC decode), encrypted or not.|
| `CONFIG_EROFS_FS_ZIP_ZSTD=y` | `erofs.zstd+dmverity` and `erofs.zstd+dmverity+encrypted` layers. Linux 6.x. |
| `CONFIG_BLK_DEV_DM=y` | Device-mapper core, required by dm-verity and dm-crypt (normally selected automatically by the options below). |
| `CONFIG_DM_VERITY=y` | Verified mount of the appended hash tree, for all layers including encrypted ones. |
| `CONFIG_CRYPTO_SHA256=y` | SHA-256 hashing used by dm-verity. | 

Additional options for `+encrypted` layers, which are opened as a dm-crypt mapping before dm-verity is applied:

| Symbol | Required for |
| :--- | :--- |
| `CONFIG_DM_CRYPT=y` | All `+encrypted` layers. Establishes the decrypted dm-crypt mapping over the LUKS2 data segment. |
| `CONFIG_CRYPTO_XTS=y` | All `+encrypted` layers. XTS mode is used by both supported ciphers (`aes-xts-plain64`, `serpent-xts-plain64`). |
| `CONFIG_CRYPTO_AES=y` | Layers encrypted with `aes-xts-plain64` (recommended cipher). An architecture-optimised AES implementation is RECOMMENDED on constrained targets. |
| `CONFIG_CRYPTO_SERPENT=y` | Layers encrypted with `serpent-xts-plain64` only. |

A device that accepts encrypted layers MUST support `aes-xts-plain64`. Support for `serpent-xts-plain64` is OPTIONAL. A consumer MUST reject a layer encrypted with a cipher its kernel does not provide, as reported by the LUKS2 header or the `org.rdk.package.content.dmcrypt.cipher` annotation. 

Unencrypted `erofs.nocmpr+dmverity` layers need only `CONFIG_EROFS_FS` plus `CONFIG_DM_VERITY` (with the options it depends on). `erofs.nocmpr+dmverity+encrypted` layers additionally need the encrypted-layer options above.




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
| `org.rdk.package.content.dmverity.offset` | Yes | Byte offset of the dm-verity hash tree (= size of the EROFS image). decimal. See [Offset Semantics](#offset-semantics). |

### Offset Semantics

For an unencrypted EROFS layer, the plaintext EROFS + dm-verity representation is the OCI layer blob itself, so `org.rdk.package.content.dmverity.offset` is relative to the beginning of the OCI layer blob.

For an encrypted EROFS layer, the OCI layer blob is a LUKS2 container. The offset is relative to the beginning of the decrypted LUKS2 data segment and MUST NOT include the LUKS2 header, metadata, keyslot or other container overhead. 

The root hash, salt and offset are generated from the plaintext EROFS + dm-verity representation, before any encryption. Encryption does not change them

**Verification:** 

For an unencrypted layer, dm-verity runs directly on the layer blob. For an encrypted layer, dm-verity runs on the decrypted dm-crypt mapping, so the processing order for an encrypted EROFS layer is:
1. Open the LUKS2 container.
2. Establish the decrypted dm-crypt mapping.
3. Apply the dm-verity configuration to the decrypted content using the descriptor root hash, salt and offset.
4. Mount the verified EROFS filesystem.

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

For encrypted layers, see [Offset Semantics](#offset-semantics).