# OCI Package Specification

## Quick Summary

**TL;DR:** This specification defines how to package applications, runtimes, and resources as OCI (Open Container Initiative) artifacts for RDK-based devices. It provides:
- **Standardized packaging format** using OCI artifacts with metadata (config) and content (payload) layers
- **Package metadata schema** for defining app/runtime properties (ID, version, dependencies, permissions, resource requirements)
- **Multiple package types** including applications, runtimes, services, and resources
- **Support for various content formats** (tar, tar+gzip, zip, EROFS+dm-verity)

This repository contains the OCI Package Specification, which defines a standard for packaging applications, runtimes, and other resources as OCI (Open Container Initiative) artifacts. This specification is designed to provide a clear and consistent way to bundle and distribute software components.

## Specification Documents

*   **[Package Format Specification (format.md)](format.md):** This document describes the structure of an OCI package, including the different layers and their corresponding media types. It details how to bundle package payloads and metadata into a single OCI artifact.

*   **[Package Metadata Specification (metadata.md)](metadata.md):** This document defines the metadata that can be included in a package. This metadata provides essential information about the package, such as its name, version, dependencies, and required permissions.

*   **[Package Metadata JSON Schema (package.schema.json)](schema/package.schema.json):** This file provides a JSON schema for validating the package metadata.

## Contributing

Contributions to the OCI Package Specification are welcome. If you have any suggestions or improvements, please feel free to open an issue or submit a pull request.