# Package Metadata

This specification describes the metadata that is stored in a package file.

## Full Example

```json
{
  "id": "com.sky.myapp",
  "version": "1.2.3",
  "versionName": "1.2.3-beta",
  "title": "My Application",
  "description": "A comprehensive example of package metadata.",
  "icons": [
    {
      "src": "icon/low-res.png",
      "sizes": "48x48"
    },
    {
      "src": "maskable_icon.png",
      "sizes": "48x48",
      "type": "image/png"
    }
  ],
  "type": "application/html",
  "entryPoint": "web/index.html",
  "dependencies": {
    "rdk.browser.wpe": ">=1.1.0"
  },
  "permissions": [
    "urn:rdk:permission.internet",
    "urn:rdk:permission:firebolt",
    "urn:rdk:permission:thunder",
    "urn:rdk:permission:mediarite",
    "urn:rdk:permission:rialto",
    "urn:rdk:permission:airplay",
    "urn:rdk:permission:compositor",
    "urn:rdk:permission:game-controller",
    "urn:entos:permission:timeshift-buffer",
    "urn:entos:permission:read-external-storage",
    "urn:entos:permission:write-external-storage",
    "urn:entos:permission:display-overlay",
    "urn:entos:permission:home-app",
    "urn:entos:permissionr:as-access",
    "urn:entos:permissior:as-player"
  ],
  "settings": {
    "urn:rdk:settings:log-levels": ["error", "warning", "info"],
    "urn:entos:settings:parent-package-id": "com.sky.parentapp",
    "unt:entos:settings:sky-live-app": true,
    "urn:rdk:settings:dial": {
      "appNames": ["MyMediaApp", "MediaRemote"],
      "corsDomains": ["http://example.com", "https://my-media.com"],
      "originHeaderRequired": true
    },
    "urn:rdk:settings:lifecycle-states": {
      "version": "v2.0",
      "states": ["Initializing", "Active", "Suspended"]
    },
    "urn:rdk:settings:timeouts": {
      "startupSeconds": 60,
      "watchdogSeconds": 30
    }
  },
  "requirements": {
    "urn:rdk:requirement:platform": {
      "architecture": "arm",
      "variant": "v7",
      "os": "linux"
    },
    "urn:rdk:requirement:input-handling": {
      "keyCapture": ["play", "pause", "stop", "fastForward", "rewind"],
      "keyMonitor": ["volumeUp", "volumeDown", "mute"]
    },
    "urn:rdk:requirement:display-info": {
      "virtualSize": 1080,
      "refreshRate": 60,
      "pictureMode": "hdr10"
    },
    "urn:rdk.requirement.audio-info": {
      "soundMode": "surround",
      "soundScene": "cinema",
      "soundLevel": -5
    },
    "urn:rdk:requirement:memory": {
      "system": "256M",
      "gpu": "128M"
    },
    "urn:rdk:requirement:storage": "200M",
    "urn:rdk:requirement:network": {
      "public": [
        {
          "name": "media-stream",
          "port": 8080,
          "protocol": "tcp"
        }
      ],
      "exported": [
        {
          "name": "my-app-service",
          "port": 9000,
          "protocol": "udp"
        }
      ],
      "imported": [
        {
          "name": "system-service",
          "port": 7000,
          "protocol": "tcp"
        }
      ],
      "multicast": {
        "address": "224.0.0.1",
        "port": 1900
      }
    },
    "urn:rdk:requirement:drm-support": [
      "com.widevine.alpha",
      "com.microsoft.playready"
    ]
  }
}
```

## Metadata Available per Package Type

| Metadata                      | Runtime  | Application | Service  |
| ----------------------------- | -------- | ----------- | -------- |
| [id](#id)                     | Required | Required    | Required |
| [version](#version)           | Required | Required    | Required |
| [versionName](#versionName)   | Optional | Optional    | Optional |
| [title](#title)               | Optional | Optional    | Optional |
| [icon](#icon)                 | Optional | Optional    | Optional |
| [type](#type)                 | Required | Required    | Required |
| [entryPoint](#entryPoint)     | Required | Required    | Required |
| [dependencies](#dependencies) | Optional | Optional    | Optional |
| [permissions](#permissions)   | Disabled | Optional    | Optional |
| [settings](#settings)         | Optional | Optional    | Optional |
| [requirements](#requirements) | Optional | Optional    | Optional |
| [platform](#platform)         | Optional | Optional    | Optional |

## id

The package `id` field is a unique identifier for the package. It is a string that is unique within the package
repository. Typically, this is represented as a reverse domain name, e.g. `com.example.myapp`, but is not mandatory.
The `id` MUST comply with the following rules:

- Cannot be an empty string.
- May only contain dots (`.`), dashes (`-`), underscore (`_`) and alphanumeric characters (a-z, A-Z, 0-9).
- In additional the following restrictions apply:
  - The first and last characters must be alphanumeric.
  - Double dots (`..`) are not allowed anywhere in the string.

_Examples_

```json
"id": "com.example.myapp"
```

```json
"id": "Netflix"
```

```json
"id": "rdk.browser.wpe"
```

## version

The package version, this must consist of 1 to 4 numbers separated by periods (`.`). Numbers must not
include a leading zero. For example, `2.01` is not allowed; however, `0.2`, `2.0.1`, and `2.10` are allowed.

_Examples_

```json
"version": "1.0.0"
```

```json
"version": "0.1.20"
```

## versionName

Optional human-readable package version string, that represents the version of the package. There are no
restrictions on the version name, except that it if present it must not be an empty string. Typically, this
is represented as a [Semantic Versioning](https://semver.org), but is not mandatory.

If `versionName` is not present, then the `version` field is used as the version name.

_Examples_

```json
"versionName": "1.0.1-beta"
```

## title

The title is a user readable string that represents the title of the package. It is not mandatory, and currently not
used by the EntOS platform.

There are no restrictions on the title except that if present it must not be an empty string.

## icon

The icon is optional and currently only used in debug environments on the EntOS platform. The icon consists of an array
of paths to a files within the package that represents the icons. The paths are relative to the root of the package.
First element point to default icon.

_Examples_

```json
"icon": [
  "icon.png",
  "icon-1920x1024"
]
```

## type

A package can have one of the following category types:

- `runtime/` - A package that contains a runtime for an application.
- `application/` - A package that contains an application. An application is optionally combined with a runtime to form
  a runnable application.
- `service/` - A package that contains a service, which is an application that runs in the background. Likewise for
  applications, a service may be combined with a runtime to form a runnable service.
- `resource/` - A package that contains resources, eg. certificates, translations, ML model.

This field has double purpose, it's simply identifies the type of the package and it is used for associating an app
or service package with a runtime package.

Depending on the `type` some of the device capabilities are implicitly requested, eg. `application` needs to render
graphics, which is not the case for `service` or `resource`.

### runtime

For `runtime/` packages this represents the type of the runtime; this is used by the system when an app or service is
started to determine which runtime to use. The runtime type is a string that is formatted like a mime type, it should
start with `runtime/` e.g. `runtime/html`.

_Examples_

```json
"type": "runtime/html"
```

```json
"type": "runtime/cobalt"
```

```json
"type": "runtime/python"
```

### application/service/resource

For `application/` and `service/` packages this field is used to describe which runtime the app or service requires to run,
either it is a downloadable runtime package or the runtime is the part of device firmware. As with `runtime/` packages,
this is a string formatted like a mime type, but should started with `application/`, e.g. `application/html`.
The type `resource/` indicates that package does not require runtime, its just need to be extracted somewher in FS.

_Examples_

```json
"type": "application/html"
```

```json
"type": "application/cobalt"
```

```json
"type": "service/system"
```

```json
"type": "resource/data"
```

or

```json
"type": "resource/model"
```

Currently, to match an app with a service, the system just strips off the `application/` or `runtime/` prefix and compares
the remaining strings. This is subject to change in the future.

## entryPoint

Every package has an entry point, this is a string and expected to be a path to a file within the package. The
interpretation of this path is up to the system / runtime that is using the package. Typically, for runtime packages
this will be the path to the `init` executable or script to run in a container. For `application` and `service` packages
this path may be passed to the runtime `init` to start the app or service.

The entry point path is relative to the root of the package.

_Examples_

```json
"entryPoint": "SkyBrowserLauncher"
```

```json
"entryPoint": "bin/NetflixApp"
```

```json
"entryPoint": "web/index.html"
```

## dependencies

Dependencies are specified in a simple object that maps a package name to a version range.
The version range is a string which has one or more space-separated descriptors.
Dependencies can also be identified with a tarball or git URL.

See [semver](https://semver.org/) for more details about specifying version ranges.

- `version` Must match version exactly
- `>version` Must be greater than version
- `=version` etc
- `<version`
- `<=version`
- `~version` Approximately equivalent to version
- `^version` Compatible with version
- `1.2.x` 1.2.0, 1.2.1, etc., but not 1.3.0
- `http://...` See 'URLs as Dependencies' below
- `*` Matches any version
- `""` (just an empty string) Same as `*`
- `version1 - version2` Same as `>=version1` `<=version2`.
- `range1 || range2` Passes if either range1 or range2 are satisfied.
- `path/path/path` See 'Local Paths' below

_Examples_

```json
"dependencies": {
  "rdk.browser.wpe": ">=1.1.0",
}
```

### URLs as Dependencies

You may specify a tarball URL in place of a version range.

This tarball will be downloaded and installed locally at install time.

### Local Paths

You can provide a path to a local directory that contains a package.

_Examples_

```json
"dependencies": {
  "foo-bar-relative": "../foo/bar",
  "foo-bar-absolute": "/foo/bar"
}
```

This feature is helpful for local offline development and creating tests that require installing where you don't want to hit an external server.

## permissions

These array represents permissions that an `application` or `service` package is requesting. Runtime
packages do not request permissions, but the apps or services that run within the runtime may request them.

| Permission \*)                                                                              |
| ------------------------------------------------------------------------------------------- |
| [urn:rdk:permission:internet](#urn:rdk:permission:internet)                                 |
| [urn:rdk:permission:firebolt](#urn:rdk:permission:firebolt)                                 |
| [urn:rdk:permission:thunder](#urn:rdk:permission:thunder)                                   |
| [urn:rdk:permission:mediarite](#urn:rdk:permission:mediarite)                               |
| [urn:rdk:permission:rialto](#urn:rdk:permission:rialto)                                     |
| [urn:rdk:permission:airplay](#urn:rdk:permission:airplay)                                   |
| [urn:rdk:permission:compositor](#urn:rdk:permission:compositor)                             |
| [urn:rdk:permission:game-controller](#urn:rdk:permission:game-controller)                   |
| [urn:entos:permission:timeshift-buffer](#urn:entos:permission:timeshift-buffer)             |
| [urn:entos:permission:read-external-storage](#urn:entos:permission:read-external-storage)   |
| [urn:entos:permission:write-external-storage](#urn:entos:permission:write-external-storage) |
| [urn:entos:permission:display-overlay](#urn:entos:permission:display-overlay)               |
| [urn:entos:permission:home-app](#urn:entos:permission:home-app)                             |
| [urn:entos:permission:as-access](#urn:entos:permission:as-access)                           |
| [urn:entos:permission:as-player](#urn:entos:permission:as-player)                           |

\*) List is extensible

- ### urn:rdk:permission:internet

_Internet Privilege_

The app or service simply requires internet access, or more specifically access to the external network.

- ### urn:rdk:permission:firebolt

_Firebolt Privilege_

The app or service requires access to the Firebolt system. This means on startup the app or service will receive a
firebolt session and can interact with the Firebolt system.

Individual firebolt permissions are not currently supported at this level of the package metadata. Currently, an
additional _firebolt manifest_ file can be stored within the package and used to specify the firebolt permissions.
However, this is subject to change.

- ### urn:rdk:permission:thunder

_Thunder Privilege_

The app or service requires access to Thunder services.

As with firebolt, individual thunder permissions are not currently supported at this level of the package metadata.

- ### urn:rdk:permission:mediarite

_Mediarite Privilege_

The app or service requires access to the Mediarite tuner sub-system.

- ### urn:rdk:permission:rialto

_Rialto Privilege_

The app or service requires access to the Rialto media player interface. Meaning at start up the app or service will
have a Rialto session created for it and can interact with the Rialto system.

- ### urn:rdk:permission:airplay

_AirPlay Privilege_

The app or service requires access to the AirPlay sub-system.

- ### urn:rdk:permission:compositor

_Compositor Privilege_

The app or service is requesting access to the window manager / compositor interface. This allows the app or service
to control the composition of apps on the screen.

- ### urn:rdk:permission:game-controller

_GameController Privilege_

The app is requesting access to any connected game controllers.

- ### urn:entos:permission:timeshift-buffer

_Time Shift Buffer Privilege_

The app or service requires access to the time shift buffer.

- ### urn:entos:permission:read-external-storage

_Read External Storage Privilege_

The app or service is requesting access to read from external storage devices. This is typically a USB device, but in
the future could include things like local network storage.

- ### urn:entos:permission:write-external-storage

_Write External Storage Privilege_

The app or service is requesting access to write to external storage devices.

- ### urn:entos:permission:display-overlay

_Display Overlay Privilege_

The app or service is requesting access to display graphical overlays. This allows an app or service to draw over the
top of all other apps, and if the overlay is a notification type, also capture input events.

This privilege is used by the Sky EPG to display things like the voice search notification, or volume controls,
picture settings, etc.

- ### urn:entos:permission:home-app

_Home App Privilege_

The app is requesting to be the home app. It's system dependant what this means, but in the EntOS case this means it
is the app that is launched when the box boots up, it's responsible for launching other apps, and is the fallback app
when the user exits an app.

See [Home App Capability](https://wiki.at.sky/display/AAI/Home+App+Capability+-+What+it+Means+in+AppService) (Sky
internal) for more information.

- ### urn:entos:permission:as-access

_AS Access Privilege_

There are 5 levels of AS access that an app or service can request. Different levels allow varying access to the
AS system.

See [AS Set Menus](https://www.stb.bskyb.com/confluence/display/2016/AS+Set+Menus) (Sky internal) for more information.

- ### urn:entos:permission:as-player

_AS Player Privilege_

This privilege grants the app access to the wayland APIs to control the AS player surface properties.

> [!NOTE]
> This privilege is only available on platforms that support the ASPlayer. And is considered a legacy requirement,
> going forward expect all video surfaces to be handled in a more generic way.

    > [!NOTE]
    > This doesn't control access to the ASPlayer REST API, this is controlled by the [AS Access Privilege](#as-access-privilege).

_Examples_

```json
"permissions": [
  "urn:rdk:permission.internet",
  "urn:rdk:permission:firebolt",
  "urn:rdk:permission:thunder",
  "urn:rdk:permission:mediarite",
  "urn:rdk:permission:rialto",
  "urn:rdk:permission:airplay",
  "urn:rdk:permission:compositor",
  "urn:rdk:permission:game-controller",
  "urn:entos:permission:timeshift-buffer",
  "urn:entos:permission:read-external-storage",
  "urn:entos:permission:write-external-storage",
  "urn:entos:permission:display-overlay",
  "urn:entos:permission:home-app",
  "urn:entos:permission:as-access",
  "urn:entos:permission:as-player"
]
```

## settings

Settings object consist package specific settings.

| Settings \*)                                                                  | Runtime  | Application | Service  |
| ----------------------------------------------------------------------------- | -------- | ----------- | -------- |
| [urn:rdk:settings:log-levels](#urn:rdk:settings:log-levels)                   | Disabled | Optional    | Optional |
| [urn:entos:settings:parent-package-id](#urn:entos:settings:parent-package-id) | Disabled | Optional    | Disabled |
| [urn:entos:settings:sky-live-app](#urn:rdk:settings:sky-live-app)             | Disabled | Optional    | Disabled |
| [urn:rdk:settings:dial](#urn:rdk:settings:dial)                               | Disabled | Optional    | Disabled |
| [urn:rdk:settings:lifecycle-states](#urn:rdk:settings:lifecycle-states)       | Disabled | Optional    | Optional |
| [urn:rdk:settings:timeouts](#urn:rdk:settings:timeouts)                       | Disabled | Optional    | Optional |

\*) List is extensible

### urn:rdk:settings:log-levels

The logging levels that the system will capture in the system log for the app or service. This is optional and
restricted, it allows for certain apps to be more verbose in their production logging than others.

This is typically ignored on debug builds where all logs are captured, but on release builds this can be used to set
the log level for the app or service.

The levels are:

- `fatal` - Fatal messages are captured.
- `error` - Error messages are captured.
- `warning` - Warning messages are captured.
- `milestone` - Milestone messages are captured.
- `info` - Info messages are captured.
- `debug` - Debug messages are captured.
  These are bitfields and can be combined, e.g. `error | warning | milestone`.

_Object Schema_

```json
"urn:rdk:setting:log-levels": {
  "description": "Logging levels.",
  "type": "array",
  "items": {
    "type": "string",
    "enum": ["fatal", "error", "warning", "milestone", "info", "debug"]
  }
}
```

_Examples_

```json
"settings": {
  "urn:rdk:settings:log-levels": [ "error", "warning", "milestone" ]
}
```

### urn:entos:settings:parent-package-id

If the application is a child of another application, then this will return the parent package `id`.

See [Parent and Child Apps capability](https://www.stb.bskyb.com/confluence/display/~TDI03/Parent+and+Child+Apps+capability)
(Sky internal) for more information.

_Object Schema_

```json
"urn:entos:settings:parent-package-id": {
  "description": "Parent package ID.",
  "type": "string"
}
```

### urn:entos:settings:sky-live-app

Boolean flag to indicate if the app is a Sky Live app. This is used by the system to determine how to handle the app.
If this flag is set then additional metadata is expected in the package to define the Sky Live usage, see
[cherry.config](https://wiki.at.sky/display/CHY/Cherry+-+Lightning+App+Configuration#CherryLightningAppConfiguration-cherry.config)
(Sky internal) for more information.

_Object Schema_

```json
"urn:entos:settings:sky-live-app": {
  "description": "Sky Live app flag.",
  "type": "boolean"
}
```

_Examples_

```json
"settings": {
  "urn:entos:settings:sky-live-app": true
}
```

### urn:rdk:settings:dial

If the app supports DIAL then this field contains the details for the DIAL service.

- #### DIAL App Name(s)
  This is an optional list of DIAL app names that the app supports. If not specified then the package `id` is used as
  the DIAL app name.
- #### CORS Domains
  This is an optional list of CORS domains that the app supports. If present the DIAL service will only accept
  requests if the origin matches one of the domains in the list.
- #### Origin Header Required
  Flag that indicates the DIAL service requires the `Origin` header to be present in the request for the given app.
  The DIAL specification allows for it to be optional, but some apps (YouTube) require it is present to pass their
  certification testing.

_Object Schema_

```json
"urn:rdk:settings:dial": {
  "description": "DIAL service details.",
  "type": "object",
  "properties": {
    "appNames": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "corsDomains": {
      "type": "array",
      "items": {
        "type": "string",
        "format": "uri"
      }
    },
    "originHeaderRequired": {
      "type": "boolean"
    }
  }
}
```

_Examples_

```json
"settings": {
  "urn:rdk:settings:dial": {
    "appNames": [ "uk.co.bbc.iPlayer", "uk.co.bbc.Sport", "uk.co.bbc.News" ],
    "corsDomains": [ "http://example.com", "https://bbc.co.uk" ],
    "originHeaderRequired": true
  }
}
```

### urn:rdk:settings:lifecycle-states

Set of the supported lifecycle states for the app or service. This is used as a hint to the system, for example if
the app doesn't support deep sleep / hibernation then the system can shut down the app or service before entering deep
sleep.

App or service needs to declare whether it supports Lifecycle 1 or 2 (via `version` field) and set of corresponding supported states.

Another example is if the app doesn't support lifecycle, then it would likely only support the `foreground` and
`background` states.

_Object Schema_

```json
"urn:rdk:settings:lifecycle-states": {
  "description": "Set of the supported lifecycle states for the app or service.",
  "type": "object",
  "properties": {
    "version": {
      "description": "Version of the lifecycle states.",
      "type": "string",
      "enum": ["v1.5", "v2.0"]
    },
    "states": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Array of lifecycle states.  The allowed values depend on the lifecycleVersion."
    }
  },
  "required": ["version", "states"],
  "additionalProperties": false,
  "allOf": [
    {
      "if": {
        "properties": {
          "version": {
            "const": "v1.5"
          }
        }
      },
      "then": {
        "properties": {
          "states": {
            "items": {
              "enum": [
                "initializing",
                "inactive",
                "foreground",
                "background",
                "unloading",
                "suspended"
              ]
            }
          }
        }
      }
    },
    {
      "if": {
        "properties": {
          "version": {
            "const": "v2.0"
          }
        }
      },
      "then": {
        "properties": {
          "states": {
            "items": {
              "enum": [
                "Hibernated",
                "Active",
                "Paused",
                "Suspended",
                "Initializing"
              ]
            }
          }
        }
      }
    }
  ]
}
```

_Examples_

```json
"settings": {
  {
    "urn:rdk:settings:lifecycle-states": {
      "version": "v1.5",
      "states": ["inactive", "foreground", "background"]
    }
  }
}
```

```json
"settings": {
  "urn:rdk:settings:lifecycle-states": {
    "version": "v2.0",
    "states": ["Initializing", "Active", "Suspended"]
  }
}
```

### urn:rdk:settings:timeouts

Defines the timeouts for the app or service startup and watchdog.

- #### Startup Timeout
  The time in seconds that the system will wait for the app or service to signal it has started before terminating it.
- #### Watchdog Timeout
  The time in seconds that the system will wait for the app or service to signal the watchdog before terminating it.
  The watchdog is not started until the first signal is received from the app or service.

_Object Schema_

```json
"urn:rdk:settings:timeouts": {
  "description": "Startup and watchdog timeouts.",
  "type": "object",
  "properties": {
    "startupSeconds": { "type": "integer" },
    "watchdogSeconds": { "type": "integer" }
  }
}

```

_Examples_

```json
"settings": {
  "urn:rdk:settings:timeouts": {
    "startupTimeoutSeconds": 60,
    "watchdogTimeoutSeconds": 30
  }
}
```

## requirements

Requirements object consist package specific HW/Platform requirements.

| Requirement \*)                                                           | Runtime  | Application | Service  |
| ------------------------------------------------------------------------- | -------- | ----------- | -------- |
| [urn:rdk:requirement:platform](#urn:rdk:requirement:platform)             | Disabled | Optional    | Optional |
| [urn:rdk:requirement:input-handling](#urn:rdk:requirement:input-handling) | Disabled | Optional    | Optional |
| [urn:rdk:requirement:display-info](#urn:rdk:requirement:display-info)     | Disabled | Optional    | Optional |
| [urn:rdk:requirement:audio-info](#urn:rdk:requirement:audio-info)         | Disabled | Optional    | Optional |
| [urn:rdk:requirement:network](#urn:rdk:requirement:network)               | Disabled | Optional    | Optional |
| [urn:rdk:requirement:memory](#urn:rdk:requirement:memory)                 | Disabled | Optional    | Optional |
| [urn:rdk:requirement:storage](#urn:rdk:requirement:storage)               | Disabled | Optional    | Optional |
| [urn:rdk:requirement:drm-support](#urn:rdk:requirement:drm-support)       | Disabled | Optional    | Optional |

\*) List is extensible

### urn:rdk:requirement:platform

This OPTIONAL object specifies the platform on which the package is intended to run.
This object is complaint with corresponding object in [OCI Index](https://github.com/opencontainers/image-spec/blob/main/image-index.md#image-index-property-descriptions) and [OCI Image Config](https://github.com/opencontainers/image-spec/blob/main/config.md#properties).

In RDK environment the values that will be used:

- `architecture` - `arm`, `arm64`
- `variant` - `v7`, `v8`, ...
- `os` - `linux`

- #### architecture REQUIRED

  The CPU architecture which the binaries in this package are built to run on.
  Configurations SHOULD use, and implementations SHOULD understand.

  Choices for `architecture` are:

  - `amd64` (64-bit x86, the most mature port),
  - `386` (32-bit x86), arm (32-bit ARM),
  - `arm64` (64-bit ARM),
  - `ppc64le` (PowerPC 64-bit, little-endian),
  - `ppc64` (PowerPC 64-bit, big-endian),
  - `mips64le` (MIPS 64-bit, little-endian),
  - `mips64` (MIPS 64-bit, big-endian),
  - `mipsle` (MIPS 32-bit, little-endian),
  - `mips` (MIPS 32-bit, big-endian),
  - `s390x` (IBM System z 64-bit, big-endian),
  - `wasm` (WebAssembly 32-bit),
  - `riscv64` (RISC-V).

- #### variant OPTIONAL

  The variant of the specified CPU architecture. Configurations SHOULD use, and implementations SHOULD understand,
  variant values listed in the Platform Variants table.

  | ISA/ABI    | architecture | variant           |
  | ---------- | ------------ | ----------------- |
  | ARM 32-bit | arm          | v6 v7, v8         |
  | ARM 64-bit | arm64        | v8, v8.1, …       |
  | POWER8+    | ppc64le      | power8, power9, … |
  | RISC-V     | riscv64      | rva20u64, …       |
  | x86-64     | amd64        | v1, v2, v3, …     |

- #### os REQUIRED

  The name of the operating system which the package is built to run on. Configurations SHOULD use, and implementations
  SHOULD understand.

  Choices for `os` are:

  - `android`,
  - `darwin`,
  - `dragonfly`,
  - `freebsd`,
  - `illumos`,
  - `ios`,
  - `js`,
  - `linux`,
  - `netbsd`,
  - `openbsd`,
  - `plan9`,
  - `solaris`,
  - `wasip1`,
  - `windows`.

_Object Schema_

```json
"urn:rdk:requirement:platform": {
  "description": "Specifies the platform on which the package is intended to run.",
  "type": "object",
  "properties": {
    "architecture": {
      "description": "The CPU architecture of the platform. Examples: 'amd64', 'arm', 'ppc64le'.",
      "type": "string"
    },
    "variant": {
      "description": "The CPU architecture variant. This field is OPTIONAL. Examples: 'v7' for ARM, 'z' for s390x.",
      "type": "string"
    },
    "os": {
      "description": "The operating system of the platform. Examples: 'linux', 'windows', 'darwin'.",
      "type": "string"
    }
  },
  "required": ["architecture", "os"]
}
```

_Examples_

```json
"requirements": {
  "urn:rdk:requirement:platform": {
    "architecture": "arm",
    "variant": "v7",
    "os": "linux"
  }
}

```

### urn:rdk:requirement:input-handling

The input handling configuration for the app. This is optional and allows the app to capture or be notified of certain
input events.

- #### System Key Capture Set
  There is a set of _system_ keys that the system will always capture and pass to the _home_ app, regardless of the
  currently focused app. This configuration allows the app to capture these keys instead of the _home_ app.
- #### System Key Monitor Set
  Similar to the _capture_ set, but in this case the monitored key will be sent to both the app AND the _home_ app.

_Object Schema_

```json
"urn:rdk:requirement:input-handling": {
  "description": "Input handling configuration.",
  "type": "object",
  "properties": {
      "keyCapture": { "type": "array", "items": { "type": "string" } },
      "keyMonitor": { "type": "array", "items": { "type": "string" } }
  }
}
```

_Examples_

```json
"requirements": {
  "urn:rdk:requirement:input-handling": {
    "keyCapture": [ "search", "voice" ],
    "keyMonitor": [ "volume+", "volume-" ]
  }
}
```

### urn:rdk:requirement:display-info

The requested display details for the app. Apps can request different display sizes and refresh rates to best match
their content. All fields are optional.

- #### Display Size
  The display size requested by the app, e.g. `1080`, `720`. Note that this typically only controls the virtual
  display resolution for the app, the wayland output display size. The actual final composition and / or HDMI
  display size is controlled by the system.
- #### Display Refresh Rate
  The display refresh rate requested by the app, e.g. `60`, `50`. Unlike the `displaySize` this does control the
  actual output refresh rate and the system will attempt to match the requested rate when the app is running.
  This may be overridden by the system if it is matching refresh rates against video content.
- #### Picture Mode
  The picture mode requested by the app, e.g. `HDR`, `SDR`. The actual mode is platform specific and may be
  a predefined mode e.g. `game`, `dynamic`, `movie`, etc.

_Object Schema_

```json
"urn:rdk:requirement:display-info": {
  "description": "Display details.",
  "type": "object",
  "properties": {
    "virtualSize": { "type": "integer" },
    "refreshRate": { "type": "integer" },
    "pictureMode": { "type": "string" }
  }
}
```

_Examples_

```json
"requirements": {
  "urn:rdk:requirement:display-info": {
    "virtualSize": 720,
    "refreshRate": 60,
    "pictureMode": "game"
  }
}
```

### urn:rdk:requirement:audio-info

The requested audio details for the app. Apps can request different audio settings to best match their content.
All fields are optional.

- #### Sound Mode
  The optional name of the sound mode to set, e.g. `kids`, `surround`, `dolby`.
- #### Sound Scene
  The optional name of the sound scene to set, e.g. `sports`, `cinema`, `music`.
- #### Sound Level
  Loudness adjustment value for the app. This is a value between -100 and 100, where 0 is the default level.
  See [App-specific loudness adjustment](https://www.stb.bskyb.com/confluence/display/2016/App-specific+loudness+adjustment)
  (Sky internal) for more information.

_Object Schema_

```json
"urn:rdk:requirement:audio-info": {
  "description": "Audio details.",
  "type": "object",
  "properties": {
    "soundMode": { "type": "string" },
    "soundScene": { "type": "string" },
    "soundLevel": {
      "type": "integer",
      "minimum": -100,
      "maximum": 100
    }
  }
}
```

_Examples_

```json
"requierements": {
  "urn:rdk:requirement:audio-info": {
    "soundMode": "kids",
    "soundScene": "cinema",
    "soundLevel": -10
  }
}
```

### urn:rdk:requirement:network

And app or service can request access to or expose a network services. Network services are divided into three
categories:

- #### Public
  Network services that an app or service exposes outside the device.
- #### Exported
  Service that an app or service exposes to other apps or services on the device.
- #### Imported
  Network services supplied by another app or service that the current app requires access to.

`Exported` and `Imported` services are related, if an app exports a service then another app can import that service,
but the `port` and `protocol` must match.

In practical terms, these settings correspond to firewall rules that are applied to the app or service in the container.
It's up to the app or service to actually listen on the ports and handle the network traffic.

> [!NOTE]
> Currently this configuration uses fixed ports defined by the app or service. In the future, we'll look at adding
> options for dynamic port assignment.

_Object Schema_

```json
"urn:rdk:requirement:network": {
  "description": "Network services configuration.",
  "type": "object",
  "properties": {
    "public": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string" },
          "port": { "type": "integer" },
          "protocol": { "type": "string" }
        },
        "required": ["name", "port", "protocol"]
      }
    },
    "exported": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string" },
          "port": { "type": "integer" },
          "protocol": { "type": "string" }
        },
        "required": ["name", "port", "protocol"]
      }
    },
    "imported": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string" },
          "port": { "type": "integer" },
          "protocol": { "type": "string" }
        },
        "required": ["name", "port", "protocol"]
      }
    }
  }
}
```

_Examples_

```json
"requirements": {
  "urn:rdk:requirement:network": {
    "public": [
      {
        "name": "netflix-mdx",
        "port": 8009,
        "protocol": "tcp"
      }
    ],
    "exported": [
      {
        "name": "com.example.myapp.service"
        "port": 1234,
        "protocol": "tcp"
      }
    ],
    "imported": [
      {
        "name": "com.example.someotherapp.service"
        "port": 4567,
        "protocol": "tcp"
      }
    ]
  }
}
```

### urn:rdk:requirement:memory

The requested memory quota for the app or service. This is optional and used as a hint to the system about the required
memory usage. The system may use this to determine when to run the app or service, or to apply limits on the memory
usage.

- #### System Memory
  The amount of system memory required by the app or service.
- #### GPU Memory
  The amount of GPU memory required by the app or service.

_Object Schema_

```json
"urn:rdk:requirement:memory": {
  "description": "Memory quota. Value can be specified with G, M, or B suffix. If no suffix is provided, the value is assumed to be in bytes.",
  "type": "object",
  "properties": {
    "system": {
      "type": "string",
      "pattern": "^\\d+[GMB]?$"
    },
    "gpu": {
      "type": "string",
      "pattern": "^\\d+[GMB]?$"
    }
  }
}
```

_Examples_

```json
"requirements": {
  "urn:rdk:requirement:memory": {
    "system": "256M",
    "gpu": "128M"
  }
}
```

### urn:rdk:requirement:storage

The requested storage quota for the app or service (in MB). This is optional and used as a hint to the system about the
amount of storage for the app or service.

_Object Schema_

```json
"urn:rdk:requirement:storage": {
  "description": "Storage quota. Value can be specified with G, M, or B suffix. If no suffix is provided, the value is assumed to be in bytes.",
    "type": "string",
    "pattern": "^\\d+[GMB]?$",
}
```

_Examples_

```json
"requirements": {
  "urn:rdk:requirement:storage": "32M"
}
```

### urn:rdk:requirement:drm-support

A set of DRM systems that the app or service supports / requires. This is optional and used as a hint to the system about the DRM systems that the app or service requires.

#### Note

Currently each DRM type is listed as a free from string, but in the future this may be changed to a more structured definition of DRM system, and likely will follow EME [Key System][https://w3c.github.io/encrypted-media/#dfn-key-system] specification.

_Object Schema_

```json
"urn:rdk:requirement:drm-support": {
  "description": "Supported DRM systems.",
  "type": "array",
  "items": { "type": "string" }
}

```

_Examples_

```json
"requirements": {
  "urn:rdk:requirement:drm-support": [ "org.w3.clearkey", "com.microsoft.playready" ]
}
```
