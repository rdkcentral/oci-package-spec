# Package Format Specification
**Version: 1.2.0**

## Versioning

The RALF Specification follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Each released
spec version is identified as `MAJOR.MINOR.PATCH` and a package MUST declare the version it was authored
against via the `specVersion` metadata field (see [metadata.md](metadata.md#specversion)).

| Component | Bump triggers                                                                                                                                |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| MAJOR     | Backwards-incompatible change. A field becomes required, is removed, or is renamed. A media type is removed/renamed. Manifest or signature payload shape changes. |
| MINOR     | Backwards-compatible additive change. A new optional field, optional media type, or optional annotation is introduced.                       |
| PATCH     | Editorial change only. Typos, clarifications, example fixes. No normative behaviour change.                                                  |

### Schema Compatibility

The JSON schema is published under MAJOR-versioned paths: `schema/v1/package.schema.json`,
`schema/v2/package.schema.json`, etc. The schema for a given MAJOR is updated in place across MINOR/PATCH
releases. A validator implementing MAJOR `N` MUST accept any payload whose declared `specVersion` has
MAJOR `N`, regardless of its MINOR/PATCH value. Unknown OPTIONAL fields introduced in later MINOR
releases are validated against the same schema path; validators MAY ignore semantic content they do not
understand but MUST NOT reject the payload solely on that basis.

The canonical schema URL tracks `main` on GitHub and resolves at:
`https://raw.githubusercontent.com/rdkcentral/oci-package-spec/main/schema/v1/package.schema.json`

For a pinned, immutable copy of the schema as it shipped with a specific spec version, use the
release asset URL (see below).

### Releases

Releases are tagged on `main` as annotated tags of the form `vMAJOR.MINOR.PATCH` (pre-releases use
`v1.1.0-rc.1`). Each release publishes:
- a GitHub Release with notes extracted from `CHANGELOG.md`;
- the schema file as a release asset (`package.schema-vMAJOR.MINOR.PATCH.json`);
- a tarball of the spec documents (`ralf-spec-MAJOR.MINOR.PATCH.tar.gz`).

See [RELEASING.md](RELEASING.md) for the maintainer runbook.

## Description

### Overview

The OCI Package Format Specification defines how to bundle binary (eg. application, runtime, resources) content as an OCI Artifacts. A compatible Package consists of package payload, and package metadata.

### Layers

The content layer always consists of the package data.

The config layer consists of a JSON-formatted string, which contains package metadata described by [Package Metadata Specification](metadata.md).

### Format

The OCI Package Format Spec consists of two layers bundled together:

A layer specifying package metadata configuration for the target package (application, runtime, etc.)
A layer containing the package data itself

The **content** layer also has a defined media type, depending on payload:

| Media Types                                                 | Description                                                                                                                                                                                   |
| ----------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| application/vnd.oci.empty.v1+json                           | Empty data. No content layer file, see [empty descriptor](https://github.com/opencontainers/image-spec/blob/main/manifest.md#guidance-for-an-empty-descriptor).                               |
| application/vnd.rdk.package.content.layer.v1.tar            | Package data as a tarball                                                                                                                                                                     |
| application/vnd.rdk.package.content.layer.v1.tar+gzip       | Package data as gzipped tarball                                                                                                                                                               |
| application/vnd.rdk.package.content.layer.v1.zip            | Package data as zip file. This would be the same as the traditional W3C widget zip contents, but without the config.xml and signature1.xml file you'd typically have in a traditional widget. |
| application/vnd.rdk.package.content.layer.v1.erofs.lz4+dmverity    | EROFS image (lz4hc-compressed) with appended dm-verity hash tree. Requires dm-verity root-hash / salt / offset annotations. See [EROFS Layer Specification](erofs_layer.md).                  |
| application/vnd.rdk.package.content.layer.v1.erofs.zstd+dmverity   | EROFS image (zstd-compressed) with appended dm-verity hash tree. Requires `CONFIG_EROFS_FS_ZIP_ZSTD` (Linux 6.x). Same annotations as above.                                                  |
| application/vnd.rdk.package.content.layer.v1.erofs.nocmpr+dmverity | EROFS image (uncompressed) with appended dm-verity hash tree. Same annotations as above.                                                                                                      |
| application/vnd.rdk.package.content.layer.v1.erofs+dmverity        | **Deprecated.** Umbrella type produced by pre-spec tooling. Readers MAY accept it and MUST treat it as `…erofs.lz4+dmverity`. New producers MUST NOT emit it.                                  |
| application/vnd.rdk.package.content.layer.v1.erofs.lz4+dmverity+encrypted    | EROFS image (lz4hc-compressed) with appended dm-verity hash tree, stored inside a LUKS2/dm-crypt encrypted container. Requires encryption key(jwe) annotations.           |


Each layer is associated with its own Media Type, which is stored in the OCI Descriptor for that layer:

| Component Name       | Type                     | Key Media Types                                                                                                                                                                                                              | Description                                                                                                                                                                                                                                                                   |
| -------------------- | ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Package OCI Artifact | JSON Object              | application/vnd.rdk.package.config.v1+json                                                                                                                                                                                   | Custom package metadata - [package.json](metadata.json)                                                                                                                                                                                                                       |
|                      | JSON Object                     | application/vnd.oci.empty.v1+json                                                                                                                                                                                            | _Application_<br><br> Is for packages that consist only out of metadata (eg. web apps) - config layer and hence have no real content layer - payload data. As defined by the OCI Image Specification, the corresponding content layer then consists solely of an empty JSON object, being {}                           |
|                      | binary data (byte array) | application/vnd.rdk.package.content.layer.v1.tar<br>application/vnd.rdk.package.content.layer.v1.tar+gzip<br>application/vnd.rdk.package.content.layer.v1.zip<br>application/vnd.rdk.package.content.layer.v1.erofs.lz4+dmverity<br>application/vnd.rdk.package.content.layer.v1.erofs.zstd+dmverity<br>application/vnd.rdk.package.content.layer.v1.erofs.nocmpr+dmverity<br>application/vnd.rdk.package.content.layer.v1.erofs+dmverity _(deprecated; accept as erofs.lz4+dmverity)_<br>application/vnd.rdk.package.content.layer.v1.erofs.lz4+dmverity+encrypted | _Runtime_<br><br>Consists of specific binaries, resources, configurations and shared libraries that build up final runtime, eg.<br><br>- rdkbrowser<br>- cobalt<br>- flutter<br>- luna<br><br>_Application_<br><br>Contains application binary, resources, shared libraries, etc. |

OCI Artifact Manifest also contains an artifactType always set to:

`application/vnd.rdk.package+type`

### Example

#### Application Package - Browser Test Tool 

##### Package Manifest

The following manifest provide an example of the OCI Artifact descriptors for an Application Package stored according to the specification:

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "application/vnd.rdk.package+type",
  "config": {
    "mediaType": "application/vnd.rdk.package.config.v1+json",
    "digest": "sha256:<digest>",
    "size": <size>,
    "annotations": {
      "org.opencontainers.image.title": "package.json"
    }
  },
  "layers": [
    {
      "mediaType": "application/vnd.rdk.package.content.layer.v1.tar+gzip",
      "digest": "sha256:<digest>",
      "size": <size>,
      "annotations": {
        "org.opencontainers.image.title": "package.tar.gz"
      }
    }
  ]
}
```

##### Package Metadata

Browser Test Tool example application package metadata.

```json
{
  "id": "com.sky.browser_test_tool",
  "specVersion": "1.1.0",
  "version": "4.3.5",
  "name": "Browser Test Tool",
  "packageType": "application",
  "packageSpecifier": "html",
  "entryPoint": ".",
  "dependencies": {
    "com.sky.rdkbrowser": ">=2.7.2"
  },
  "permissions": [
    "urn:rdk:permission:internet",
    "urn:rdk:permission:firebolt",
    "urn:rdk:permission:thunder",
    "urn:entos:permission:as-access",
    "urn:entos:permission:as-player"
  ],
  "configuration": {
    "urn:rdk:config:platform": {
      "architecture": "arm",
      "variant": "v7",
      "os": "linux"
    },
    "urn:rdk:config:memory": {
      "system": "450M",
      "gpu": "200M"
    },
    "urn:rdk:config:storage": {
      "maxLocalStorage": "32M"
    },
    "urn:rdk:config:drm-support": [
      "com.widevine.alpha",
      "com.microsoft.playready"
    ]
  }
}
```

##### Package Data

Browser Test Tool example application package.

```
/package
├── icon.png
├── index.html
├── override.js
├── rdk.config
```

#### Application Package - web application with no package data

##### Package Manifest

The following manifest provides an example of the OCI Artifact descriptors for an Application Package stored according to the specification:

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "application/vnd.rdk.package+type",
  "config": {
    "mediaType": "application/vnd.rdk.package.config.v1+json",
    "digest": "sha256:<digest>",
    "size": <size>,
    "annotations": {
      "org.opencontainers.image.title": "package.json"
    }
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.empty.v1+json",
      "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
      "size": 2
    }
  ]
}
```

##### Package Metadata

Web application with no package data example application package metadata.

```json
{
  "id": "com.rdkcentral.wiki",
  "specVersion": "1.1.0",
  "version": "0.1.0",
  "name": "RDK Central Wiki",
  "packageType": "application",
  "packageSpecifier": "html",
  "entryPoint": "https://wiki.rdkcentral.com/",
  "dependencies": {
    "com.sky.rdkbrowser": ">=2.7.2"
  },
  "permissions": [
    "urn:rdk:permission:internet",
    "urn:rdk:permission:firebolt"
  ],
  "configuration": {
    "urn:rdk:config:platform": {
      "architecture": "arm",
      "variant": "v7",
      "os": "linux"
    },
    "urn:rdk:config:memory": {
      "system": "450M",
      "gpu": "200M"
    },
    "urn:rdk:config:storage": {
      "maxLocalStorage": "32M"
    }
  }
}
```

#### Runtime Package - rdkbrowser

##### Package Manifest

The following manifest provide an example of the OCI Artifact descriptors for an Runtime Package stored according to the specification:

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "application/vnd.rdk.package+type",
  "config": {
    "mediaType": "application/vnd.rdk.package.config.v1+json",
    "digest": "sha256:<digest>",
    "size": <size>,
    "annotations": {
      "org.opencontainers.image.title": "package.json"
    }
  },
  "layers": [
    {
      "mediaType": "application/vnd.rdk.package.content.layer.v1.tar+gzip",
      "digest": "sha256:<digest>",
      "size": <size>,
      "annotations": {
        "org.opencontainers.image.title": "runtime.tar.gz"
      }
    }
  ]
}
```

##### Package Metadata

Browser example runtime package metadata.

```json
{
  "id": "com.sky.rdkbrowser",
  "specVersion": "1.1.0",
  "version": "2.7.2",
  "name": "com.sky.rdkbrowser",
  "packageType": "runtime",
  "packageSpecifier": "html",
  "entryPoint": "SkyBrowserLauncher",
  "dependencies": {
    "com.rdk.base.system": ">=1.1.0"
  },
  "configuration": {
    "urn:rdk:config:runtime": {
      "supportedApplicationTypes": [
        {
          "type": "html",
          "args": {
            "userAgent": "RDK/WPE"
          }
        }
      ]
    }
  }
}
```

##### Package Data

Browser example runtime package.

```
/runtime
├── SkyBrowserLauncher
├── fonts
│ ├── Glametrix.otf
│ ├── GlametrixBold.otf
│ ├── GlametrixLight.otf
│ ├── NotoEmoji-Regular.woff2
│ ...
│ └── noto-sans-kr-v27-korean-regular.woff2
├── fonts.conf
├── icon.png
├── qtvirtualkeyboard
│ ├── 5.15
│ │ └── SkyVirtualKeyboard
│ └── README.md
├── signature1.xml
└── wpewebkit
├── extensions
│ ├── libAAMPExtension.so
│ ├── ...
│ └── libWpeQueryExtension.so
└── libSkyWebKitBackend.so
```

## Signing

Packages MAY be signed using the Cosign "simple signing" format. This allows for offline verification of packages without reliance on an OCI registry.
The signature follows [Cosign Signature Specification](https://github.com/sigstore/cosign/blob/main/specs/SIGNATURE_SPEC.md).

### Signature Artifact

A signed package consists of two OCI manifests:

1. The **Target Manifest**: The actual package manifest (application or runtime).
2. The **Signature Manifest**: A separate manifest containing the signature data.

The Signature Manifest MUST be associated with the Target Manifest via the `org.opencontainers.image.ref.name` annotation. The value of this annotation MUST follow the pattern:
`sha256-<target_manifest_digest>.sig`

### Signature Structure

The Signature Manifest consists of a single layer containing the signed payload.

#### Signature Layer

The signature layer MUST use the media type:
`application/vnd.dev.cosign.simplesigning.v1+json`

The layer descriptor contains the following annotations:

| Annotation Key                       | Requirement | Description                                                    |
| ------------------------------------ | ----------- | -------------------------------------------------------------- |
| `dev.cosignproject.cosign/signature` | Required    | The base64-encoded signature of the layer's content (payload). |
| `dev.sigstore.cosign/certificate`    | Optional    | The PEM-encoded X.509 certificate used for signing.            |
| `dev.sigstore.cosign/chain`          | Optional    | The PEM-encoded certificate chain.                             |

`dev.sigstore.cosign/certificate` and `dev.sigstore.cosign/chain` are not required. It is possible to just sign with public / private key, rather than PKI certificate chains.

#### Signature Payload

The content of the signature layer (the blob) is a JSON object with the following structure:

```json
{
  "critical": {
    "identity": {
      "docker-reference": "<reference>"
    },
    "image": {
      "docker-manifest-digest": "<sha256_digest_of_target_manifest>"
    },
    "type": "cosign container image signature"
  },
  "optional": null
}
```

`docker-reference` is a string that identifies the signed artifact. It does not
have to follow any specific format, but it is recommended to use a format that clearly indicates the artifact being
signed (e.g., `com.sky.rdkbrowser:2.7.2`).

### Key Generation (Informative)

The following commands demonstrate how to generate a self-signed certificate and key pair using OpenSSL for testing purposes.

**1. Generate Root Private Key**

```bash
openssl genrsa -out rootCA.key 4096
```

**2. Create Self-Signed Certificate**

```bash
openssl req -x509 -new -nodes -key rootCA.key -sha512 -days 3650 -out rootCA.pem
```

**3. Generate PKCS#12 Certificate**

```bash
openssl pkcs12 -export -in ./rootCA.pem -inkey ./rootCA.key -out <path_to_new_pkcs12>
```

### Example: Signed Package

#### Index (index.json)

The index links the target package and its signature.

```json
{
  "schemaVersion": 2,
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:4161227e2a40097d5e00150b6027e86ea78a03f3668d41cf240173dc9b199614",
      "size": 469,
      "annotations": {
        "org.opencontainers.image.ref.name": "test"
      }
    },
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:f85bf2af0b25c1e0774ecc283e192fddc4dafbb33ef5ec47a863c1f1880fc48f",
      "size": 9299,
      "annotations": {
        "org.opencontainers.image.ref.name": "sha256-4161227e2a40097d5e00150b6027e86ea78a03f3668d41cf240173dc9b199614.sig"
      }
    }
  ]
}
```

#### Signature Manifest

(Digest: `sha256:f85bf2af0b25c1e0774ecc283e192fddc4dafbb33ef5ec47a863c1f1880fc48f`)

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "size": 342,
    "digest": "sha256:5ba08d0c9572e6bcec408cec3116751b3724e8d7c3c5d5d7d74453085d6ef223"
  },
  "layers": [
    {
      "mediaType": "application/vnd.dev.cosign.simplesigning.v1+json",
      "size": 243,
      "digest": "sha256:f55379abf8de5f1abedab0cd5399c1a1c7a3d60636fe654e3f03779f8ed42fc4",
      "annotations": {
        "dev.cosignproject.cosign/signature": "R/E7xo8xv1O3uVQVqXNXDp8AXqynRe5ksIywmeWoIr47AMgQWq9uoSMWDaBF+D3KPulAhYpk4LTmTWnwO3b9Xx3IeZA0ZRUHWjfZZy+4DMMUFDgxRT6sl7j7KNvcfQkGSqL7W71ulsjESFDFRKgaNP8VQfCbf6LiZLOPiA4Q+HD3qVlx4uNPVfGbziMC0m+T28EeY8Z2vyggc/9ZC5SbA944S0PRD5CDasXVNFDXEo6KjSjd4czIRjcsw6OLakd7hOiVeIAdYxqq0bc8I35RN/Q4qwB7gIrj5eQQAH8OdgEDNNGBgmhgonVqil4+jhTr/b+9vfPmljl2sk2eBF8PD2E0jravPbY8+bbr8OwEMwSbl0uPGu/+ZU14xRpzbUHz8L9+rge+v9yoLejXlwcL8BdFBevFAoIaQCVf0w2E+AqeMRm9wnk8o+yOGyyCyRW1IT0LhewkEyL5TklEHQJx/j67Y8Ga5ghb5ZEEWnQltCrwBm9QD3w3k+u9mscpEeyO5JrhkSvJptZw6fLtsxh9YNJdRRsdrN+PxfUxWfNr9R7AuPlb+QyUg8Nzk/ABDFTp/DiV3BmvCS1V55/ucOBWJOIghT55QEnk7dlwn+rl92amtnBQ+KFRmoJjAmBESozfu5T8H0Ehj/Z46fMBoRwcPyxkm0rfTR/r/lHRU3uku0Y=",
        "dev.sigstore.cosign/certificate": "-----BEGIN CERTIFICATE-----\nMIIFhjCCA26gAwIBAgICEAAwDQYJKoZIhvcNAQELBQAwQTELMAkGA1UEBhMCR0Ix\n...\n-----END CERTIFICATE-----
",
        "dev.sigstore.cosign/chain": "-----BEGIN CERTIFICATE-----\nMIIFRjCCAy6gAwIBAgICEAAwDQYJKoZIhvcNAQELBQAwOTELMAkGA1UEBhMCR0Ix\n...\n-----END CERTIFICATE-----
"
      }
    }
  ]
}
```

#### Signature Layer Payload

(Digest: `sha256:f55379abf8de5f1abedab0cd5399c1a1c7a3d60636fe654e3f03779f8ed42fc4`)

```json
{
  "critical": {
    "identity": {
      "docker-reference": "com.sky.rdkbrowser:2.7.2"
    },
    "image": {
      "docker-manifest-digest": "sha256:4161227e2a40097d5e00150b6027e86ea78a03f3668d41cf240173dc9b199614"
    },
    "type": "cosign container image signature"
  },
  "optional": null
}
```


## Encryption

Packages MAY be encrypted using dm-crypt+LUKS for block-level encryption of OCI content layers in BOLT/RALF bundles.

### Encryption Principles

1. Encryption applies to content layers only; config layers and signature layer remain unencrypted.
2. MediaType of the content layer signal encrypted content via +encrypted suffix to base mediaType.
3. Symmetric Master Key is generated by [bolt-tool](https://github.com/rdkcentral/bolt-tools/blob/main/bolt/docs/make.md), wrapped with the recipient's public key, and delivered via OCI manifest annotations as a JWE token.
4. Encryption updates the OCI package manifest (content layer digest, size, mediaType and annotations). Therefore, if package signing is used, the encrypted manifest MUST be signed after encryption. Cosign signatures bind to the final encrypted manifest digest.
5. dm-crypt + LUKS performs only the block-level encryption of the content blob; all JWE wrapping, annotation injection, and manifest update are performed by bolt-tool. 


### Encryption Architecture

Package Structure with dm-crypt+LUKS Encryption: A complete encrypted bolt package consists of,

**1. Config Layer (Unencrypted)**:

* Package metadata (`application/vnd.rdk.package.config.v1+json`)
* Remains plaintext for access control and version detection

**2. Content Layer (Encrypted — LUKS2 container)**: 

* Original content blob encrypted into a LUKS2 container
* The LUKS2 container file replaces the original content layer blob in the OCI layout
* LUKS2 container structure: [LUKS2 Header (~4–16 MB)] | [AES-XTS Encrypted Content Data]
* mediaType appended with +encrypted suffix
* Digest replaced with sha256 of the LUKS2 container blob
* Size replaced with size of the LUKS2 container blob

**3. Signature Manifest (Unencrypted — if cosigned)**

* References the encrypted package manifest digest
* Signature validation occurs against the encrypted package manifest
* Content-layer decryption occurs only after successful signature verification

**4. Encryption Metadata (Content Layer Manifest Annotations — injected by bolt-tool)**

* JWE-wrapped master key (`org.opencontainers.image.enc.keys.jwe`)

### LUKS2 Container Structure

**Physical On-disk Layout**

```
Offset 0
├─────────────────────────────────────────────────────────┤
│  PRIMARY HEADER  (4 KiB)                                │
├─────────────────────────────────────────────────────────┤
│  PRIMARY JSON METADATA AREA                             |
|           - Keyslots definitions                        |
|           - segments definitions                        |
|           - digests                                     |
|           - tokens                                      |
|           - config                                      |
├─────────────────────────────────────────────────────────┤
│  SECONDARY HEADER (4 KiB)                               |              
├─────────────────────────────────────────────────────────┤
│  SECONDARY JSON METADATA AREA                           | 
|           - Backup copy of JSON metadata                │
├─────────────────────────────────────────────────────────┤
│  KEYSLOTS AREA                                          |
|                                                         |
|             Slot 0                                      |
|             Slot 1                                      |
|             ...                                         |
|             Slot 31                                     | 
|  Encrypted Volume Key Material                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  DATA SEGMENT                                           │
│                                                         |  
|  AES-XTS sector-encrypted data                          |
|  ext4 / xfs / erofs / squashfs / raw image              │
│                                                         │
└─────────────────────────────────────────────────────────┘

```

### Properties

This section describes about the **Required** and **Optional** properties used to encrypt OCI packages.

**1. mediaType (Required)**

* **Type**: String
* **Description**:
    * The mediaType property of the Content Layer, MUST append +encrypted suffix to base mediaType, to signal encrypted payloads
    * The .bolt package has multiple layers like config, content and signature layers. Whereas we will be encrypting only the content layer of the package and the respective layer mediaType property MUST append +encrypted to signal the encrypted payload
* **Pattern**: <base-media-type>+encrypted
* **Example**:

    ```json
    "mediaType": "application/vnd.rdk.package.content.layer.v1.erofs.lz4+dmverity+encrypted"
    ```

**2. digest (Required)**

* **Type**: String (sha256 digest of encrypted blob)
* **Description**: SHA-256 digest of the LUKS2 container blob (not the original plaintext). The LUKS2 container is larger than the original content due to the LUKS2 header overhead (~16MB)
* **Format**: sha256:<hex-encoded-hash>
* **Example**:

    ```json
    "digest": "sha256:a069714e5aac375e9fe89c6ba743dc96a49a3d2ebbeff01390529379d071223b"
    ```

**3. size (Required)**
* **Type**: Integer
* **Description**: Size in bytes of the LUKS2 container blob. Always larger than the original content by at least the LUKS2 header size (typically 4–16 MB depending on --luks2-metadata-size and --luks2-keyslots-size configuration).
* **Example**:

    ```json
    "size": 31219712
    ```

**4. encryptionKey (Required)**
* **Type**: String (base64-encoded JWE General JSON serialization)
* **Description**:
    * JWE (JSON Web Encryption) token containing the dm-crypt/LUKS master key wrapped with the recipient's RSA public key
    * The master key is the 512-bit raw key used to open the LUKS2 container (passed via --master-key-file to cryptsetup)
    * Stored as an annotation on the Content Layer of the Package Manifest
* **JWE Header**:

    ```json
    {
      "alg": "RSA-OAEP-256",
      "enc": "A256GCM",
      "kid": "recipient-public-key-id-1"
    }
    ```
* **JWE Payload**: Raw 512-bit (64-byte) LUKS master key bytes
* **Example**:

    ```json
    "annotations": {
      "org.opencontainers.image.enc.keys.jwe": "eyJhbGciOiJSU0EtT0FFUC0yNTYiLCJlbmMiOiJBMjU2R0NNIn0..."
    }
    ```

**5. dmCryptCipher (Optional)**
* **Type**: String
* **Description**:
    * The dm-crypt cipher suite used to encrypt the LUKS2 content blob
    * Extracted from the LUKS2 header after cryptsetup luksFormat and injected by bolt-tool
    * Format: <cipher>-<mode>-<iv_generation> (Linux dm-crypt cipher string)
* **Supported Values**:
    * aes-xts-plain64 - AES-256 in XTS mode (recommended)
    * serpent-xts-plain64 - Serpent-256 in XTS mode (alternative)
* **Example**:

    ```json
    "annotations": {
      "org.opencontainers.image.dmcrypt.cipher": "aes-xts-plain64"
    }
    ```

**6. dmCryptSalt (Optional)**
* **Type**: String (base64-encoded)
* **Description**:
    * The PBKDF salt extracted from LUKS2 keyslot 0 after cryptsetup luksFormat
    * Injected by bolt-tool after extracting from cryptsetup luksDump
    * Although the salt is also embedded in the LUKS2 header (inside the encrypted blob), the annotation provides,
      * Early algorithm verification before downloading the full blob
      * Cross-validation at decryption time
    * Base64-encoded binary salt (32 bytes / 256 bits)
* **Example**:

    ```json
    "annotations": {
      "org.opencontainers.image.dmcrypt.salt": "dGVzdF9zYWx0X2V4YW1wbGVfc2FsdF92YWx1ZWhlcmU="
    }
    ```

**7. dmCryptKeySize (Optional)**
* **Type**: String (integer as string, in bits)
* **Description**: Key size passed to cryptsetup luksFormat --key-size. For AES-XTS this is 512 (2× 256-bit XTS keys). Informational.
* **Example**:

    ```json
    "annotations": {
      "org.opencontainers.image.dmcrypt.keysize": "512"
    }
    ```

**8. dmCryptType (Optional)**
* **Type**: String
* **Description**: LUKS format version. Always luks2 for new packages. Informational.
* **Example**:

    ```json
    "annotations": {
      "org.opencontainers.image.dmcrypt.type": "luks2"
    }
    ```

### Storage and Representation

* The mediaType of the Content Layer MUST append +encrypted suffix to base mediaType
* The JWE-wrapped master key is base64-encoded and stored as `org.opencontainers.image.enc.keys.jwe` annotation on the Content Layer
* The Content Layer digest and size reference the LUKS2 container blob

#### Layer Descriptor with dm-crypt + LUKS Encryption

```json
{
  "mediaType": "application/vnd.rdk.package.content.layer.v1.erofs+lz4+dmverity+encrypted",
  "digest": "sha256:a069714e5aac375e9fe89c6ba743dc96a49a3d2ebbeff01390529379d071223b",
  "size": 31219712,
  "annotations": {
    "org.rdk.package.content.dmverity.roothash": "e8bbfa9a6f06ad0507f57b6a998c329430e47b2ff0fdcac17f5b22b72e8797ca",
    "org.rdk.package.content.dmverity.offset": "4096",
    "org.rdk.package.content.dmverity.salt": "33e130b3d76367806d458cd40464a7c6e713f15d36ba8b703364613a598ae760",
    "org.opencontainers.image.enc.keys.jwe": "eyJhbGciOiJSU0EtT0FFUC0yNTYiLCJlbmMiOiJBMjU2R0NNIiwia2lkIjoicmVjaXBpZW50LWtleS0xIn0",
    "org.opencontainers.image.dmcrypt.cipher": "aes-xts-plain64",
    "org.opencontainers.image.dmcrypt.salt": "dGVzdF9zYWx0X2V4YW1wbGVfc2FsdF92YWx1ZWhlcmU=",
    "org.opencontainers.image.dmcrypt.keysize": "512",
    "org.opencontainers.image.dmcrypt.type": "luks2"
  }
}  
```

#### Encrypted Package Manifest

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "application/vnd.rdk.package+type",
  "config": {
    "mediaType": "application/vnd.rdk.package.config.v1+json",
    "digest": "sha256:e65677a8132ccf679fddc80aeda30177b4301168bf7c9acd2d34b7d66427d9e5",
    "size": 414,
    "annotations": {
      "org.opencontainers.image.title": "package.json"
    }
  },
  "layers": [
    {
      "mediaType": "application/vnd.rdk.package.content.layer.v1.erofs+lz4+dmverity+encrypted",
      "digest": "sha256:a069714e5aac375e9fe89c6ba743dc96a49a3d2ebbeff01390529379d071223b",
      "size": 31219712,
      "annotations": {
        "org.rdk.package.content.dmverity.roothash": "<hex-dmverity-root-hash>",
        "org.rdk.package.content.dmverity.offset": "<decimal-dmverity-offset>",
        "org.rdk.package.content.dmverity.salt": "<hex-dmverity-salt>",
        "org.opencontainers.image.enc.keys.jwe": "<JWE-general-JSON-token>",
        "org.opencontainers.image.dmcrypt.cipher": "aes-xts-plain64",
        "org.opencontainers.image.dmcrypt.salt": "<base64-luks-keyslot-salt>",
        "org.opencontainers.image.dmcrypt.keysize": "512",
        "org.opencontainers.image.dmcrypt.type": "luks2"
      }
    }
  ]
}
```

### Encryption Mechanism

Encryption is applied ONLY to the Content blob of the Target Manifest (Plaintext Package Manifest), not to the Signature Manifest.

Target Manifest (Plain Package)

  ├── Config layer    (unencrypted metadata)
  
  └── Content layer   (plaintext blob → replaced with LUKS2 container)

Signature Manifest (Attached to Encrypted Target)

  ├── References: sha256-<encrypted_manifest_digest>.sig

  ├── Contains signature of encrypted package manifest

  └── Never encrypted

[bolt-tool](https://github.com/rdkcentral/bolt-tools/blob/main/bolt/docs/make.md) is the sole orchestrator. dm-crypt + LUKS only performs block encryption:
* **bolt-tool** generates the 512-bit master key (random, in-process)
* **bolt-tool** invokes **cryptsetup luksFormat** to create the LUKS2 container
* **bolt-tool** invokes **cryptsetup luksOpen**  to mount the container and writes the content layer blob into it
* **bolt-tool** invokes **cryptsetup luksDump** to extract salt and cipher for annotation
* **bolt-tool** performs wrapping of the master key and constructs the JWE token
* **bolt-tool** updates the OCI manifest with new digest, size, mediaType, and all annotations
* The master key exists only in process memory during encryption; it is never persisted to disk in plaintext


### Signing Order

When both encryption and signing mechanism are used, MUST perform operations in the following order:

1. Generate OCI package manifest.
2. Encrypt the content layer.
3. Replace the content layer digest, size, mediaType, and encryption annotations.
4. Generate the final encrypted package manifest.
5. Generate the cosign signature over the encrypted package manifest.

When a signature manifest is present, signature verification MUST succeed before content-layer decryption is attempted.



### Signing and Encryption Workflow
Refer to [Encryption wiki](https://wiki.rdkcentral.com/pages/viewpage.action?pageId=502400306&spaceKey=WG&title=Encryption%2BFormat%2Busing%2Bdm-crypt%2BLUKS)

### End-to-End Encryption Flow
Refer to [Encryption Wiki](https://wiki.rdkcentral.com/pages/viewpage.action?pageId=502400306&spaceKey=WG&title=Encryption%2BFormat%2Busing%2Bdm-crypt%2BLUKS)

    




