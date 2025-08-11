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
| application/vnd.rdk.package.content.layer.v1.tar            | Package data as a tarball                                                                                                                                                                     |
| application/vnd.rdk.package.content.layer.v1.tar+gzip       | Package data as gzipped tarball                                                                                                                                                               |
| application/vnd.rdk.package.content.layer.v1.zip            | Package data as zip file. This would be the same as the traditional W3C widget zip contents, but without the config.xml and signature1.xml file you'd typically have in a traditional widget. |
| application/vnd.rdk.package.content.layer.v1.erofs+dmverity | Package data as an EROFS file system image with appended dm-verity hash tree. This layer format is required to have annotations that describe the dm-verity root hash, salt and offset.       |

Each layer is associated with its own Media Type, which is stored in the OCI Descriptor for that layer:

| Component Name       | Type                     | Key Media Types                                                                                                                                                                                                              | Description                                                                                                                                                                                                                                                                   |
| -------------------- | ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Package OCI Artifact | JSON Object              | application/vnd.rdk.package.config.v1+json                                                                                                                                                                                   | Custom package metadata - [package.json](metadata.json)                                                                                                                                                                                                                       |
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
    "size": "<size>",
    "annotations": {
      "org.opencontainers.image.title": "package.json"
    }
  },
  "layers": [
    {
      "mediaType": "application/vnd.rdk.package.content.layer.v1.tar+gzip",
      "digest": "sha256:<digest>",
      "size": "<size>",
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
    "size": "<size>",
    "annotations": {
      "org.opencontainers.image.title": "package.json"
    }
  },
  "layers": [
    {
      "mediaType": "application/vnd.rdk.package.content.layer.v1.tar+gzip",
      "digest": "sha256:<digest>",
      "size": "<size>",
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
