# RALF (RDK Application Layer Format) Specification

This repository contains the **RALF (RDK Application Layer Format) Specification**, which defines a standard for packaging applications, runtimes, and other resources as OCI (Open Container Initiative) artifacts for RDK devices.

RALF packages are distributed as single files with the `.ralf` extension (ZIP or Tarball) containing a standard OCI Image Layout.

## Specification Documents

*   **[RALF Format Specification (format.md)](format.md):** This document describes the structure of a RALF package (`.ralf`), including the file format (ZIP/Tar), internal OCI layout, layers, media types, and the **Cosign-based signing** mechanism.

*   **[Package Metadata Specification (metadata.md)](metadata.md):** This document defines the metadata that can be included in a package's config layer. This metadata provides essential information such as package name, version, dependencies, permissions, and resource requirements.

*   **[Package Metadata JSON Schema (package.schema.json)](schema/package.schema.json):** This file provides a JSON schema for validating the package metadata.

## Key Features

*   **OCI Standard**: Built on open standards (OCI Image Spec, OCI Image Layout).
*   **Single File Distribution**: Packages are bundled as `.ralf` files (ZIP or Tarball).
*   **Secure**: Supports offline verification using Cosign-style signatures.
*   **Flexible**: Supports various payloads including Tarballs and EROFS images with dm-verity.

## Contributing

Contributions to the RALF Specification are welcome. If you have any suggestions or improvements, please feel free to open an issue or submit a pull request.
