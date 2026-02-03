# Package Format Specification

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
| application/vnd.rdk.package.content.layer.v1.erofs+dmverity | Package data as an EROFS file system image with appended dm-verity hash tree. This layer format is required to have annotations that describe the dm-verity root hash, salt and offset.       |

Each layer is associated with its own Media Type, which is stored in the OCI Descriptor for that layer:

| Component Name       | Type                     | Key Media Types                                                                                                                                                                                                              | Description                                                                                                                                                                                                                                                                   |
| -------------------- | ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Package OCI Artifact | JSON Object              | application/vnd.rdk.package.config.v1+json                                                                                                                                                                                   | Custom package metadata - [package.json](metadata.json)                                                                                                                                                                                                                       |
|                      | JSON Object                     | application/vnd.oci.empty.v1+json                                                                                                                                                                                            | _Application_<br><br> Is for packages that consist only out of metadata (eg. web apps) - config layer and hence have no real content layer - payload data. As defined by the OCI Image Specification, the corresponding content layer then consists solely of an empty JSON object, being {}                           |
|                      | binary data (byte array) | application/vnd.rdk.package.content.layer.v1.tar<br>application/vnd.rdk.package.content.layer.v1.tar+gzip<br>application/vnd.rdk.package.content.layer.v1.zip<br>application/vnd.rdk.package.content.layer.v1.erofs+dmverity | _Runtime_<br><br>Consist specific binaries, resources, configurations and shared libraries that build up final runtime, eg.<br><br>- rdkbrowser<br>- cobalt<br>- flutter<br>- luna<br><br>_Application_<br><br>Contains application binary, resources, shared libraries, etc. |

OCI Artifact Manifest also consist artifactType always set to:

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
  "version": "4.3.5",
  "name": "Browser Test Tool",
  "packageType": "application",
  "packageSpecifier": "html",
  "entryPoint": ".",
  "dependencies": {
    "com.sky.rdkbrowser": ">=2.7.2-kirkstone"
  },
  "permissions": [
    "urn:rdk:permission.internet",
    "urn:rdk:permission:firebolt",
    "urn:rdk:permission:thunder",
    "urn:entos:permissionr:as-access",
    "urn:entos:permissior:as-player"
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
  "version": "0.1.0",
  "name": "RDK Central Wiki",
  "packageType": "application",
  "packageSpecifier": "html",
  "entryPoint": "https://wiki.rdkcentral.com/",
  "dependencies": {
    "com.sky.rdkbrowser": ">=2.7.2-kirkstone"
  },
  "permissions": [
    "urn:rdk:permission.internet",
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
  "version": "2.7.2-kirkstone",
  "name": "com.sky.rdkbrowser",
  "packageType": "runtime",
  "packageSpecifier": "html",
  "entryPoint": "SkyBrowserLauncher",
  "dependencies": {
    "com.rdk.base.system": ">=1.1.0"
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
signed (e.g., `com.sky.rdkbrowser:2.7.2-kirkstone`).

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
      "docker-reference": "com.sky.rdkbrowser:2.7.2-kirkstone"
    },
    "image": {
      "docker-manifest-digest": "sha256:4161227e2a40097d5e00150b6027e86ea78a03f3668d41cf240173dc9b199614"
    },
    "type": "cosign container image signature"
  },
  "optional": null
}
```