# Package Metadata Specification

This specification describes the metadata that is stored in a package file.

## Full Example

```json
{
  "id": "com.sky.myapp",
  "version": "1.2.3",
  "versionName": "1.2.3-beta",
  "name": "My Application",
  "packageType": "application",
  "packageSpecifier": "html",
  "entryPoint": "web/index.html",
  "entryArgs": ["--some-option=true"],
  "dependencies": {
    "rdk.browser.wpe": ">=1.1.0"
  },
  "permissions": [
    "urn:rdk:permission:home-app",
    "urn:rdk:permission.internet",
    "urn:rdk:permission:firebolt",
    "urn:rdk:permission:thunder",
    "urn:rdk:permission:rialto",
    "urn:rdk:permission:game-controller",
    "urn:rdk:permission:timeshift-buffer",
    "urn:rdk:permission:external-storage::read",
    "urn:rdk:permission:external-storage::write",
    "urn:rdk:permission:display-overlay",
    "urn:rdk:permission:compositor"
  ],
  "configuration": {
    "urn:rdk:config:overrides": {
      "application": {
        "colorTheme": "vivid"
      },
      "runtime": {
        "userAgent": "BBC_requirement_requesting_Operator_Specific_UserAgent",
        "libInject": "oiptv.js",
        "mTls": "/etc/certs/client_operator.cert"
      }
    },
    "urn:rdk:config:log-levels": ["error", "warning", "info"],
    "urn:rdk:config:dial": {
      "appNames": ["MyMediaApp", "MediaRemote"],
      "corsDomains": ["http://example.com", "https://my-media.com"],
      "originHeaderRequired": true
    },
    "urn:rdk:config:application-lifecycle": {
      "supportedNonActiveStates": ["paused", "suspended", "hibernated"],
      "maxSuspendedSystemMemory": "16M",
      "maxTimeToSuspendMemoryState": "10",
      "startupTimeout": 60,
      "watchdogInterval": 30
    },
    "urn:rdk:config:platform": {
      "architecture": "arm",
      "variant": "v7",
      "os": "linux"
    },
    "urn:rdk:config:input-handling": {
      "keyCapture": ["play", "pause", "stop", "fastForward", "rewind"],
      "keyMonitor": ["volumeUp", "volumeDown", "mute"]
    },
    "urn:rdk:config:window": {
      "virtualDisplaySize": 1080
    },
    "urn:rdk:config:memory": {
      "system": "256M",
      "gpu": "128M"
    },
    "urn:rdk:config:storage": {
      "maxLocalStorage": "32M",
      "sharedStorageAppId": "com.sky.myapp2"
    },
    "urn:rdk:config:network": [
      {
        "name": "netflix-mdx",
        "port": 8009,
        "protocol": "tcp",
        "type": "public"
      },
      {
        "name": "com.example.myapp.service",
        "port": 1234,
        "protocol": "tcp",
        "type": "exported"
      },
      {
        "name": "com.example.someotherapp.service",
        "port": 4567,
        "protocol": "tcp",
        "type": "imported"
      }
    ]
  }
}
```

## Metadata Available per Package Type

| Metadata                              | Base     | Runtime  | Application/Service |
| ------------------------------------- | -------- | -------- | ------------------- |
| [id](#id)                             | Required | Required | Required            |
| [version](#version)                   | Required | Required | Required            |
| [versionName](#versionName)           | Optional | Optional | Optional            |
| [name](#name)                         | Optional | Optional | Optional            |
| [packageType](#packageType)           | Required | Required | Required            |
| [packageSpecifier](#packageSpecifier) | Optional | Optional | Optional            |
| [entryPoint](#entryPoint)             | Required | Required | Required            |
| [entryArgs](#entryArgs)               | Optional | Optional | Optional            |
| [dependencies](#dependencies)         | N/A      | Optional | Optional            |
| [permissions](#permissions)           | N/A      | Optional | Optional            |
| [configuration](#configuration)       | N/A      | Optional | Optional            |

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
{
  "id": "com.example.myapp"
}
```

```json
{
  "id": "Netflix"
}
```

```json
{
  "id": "rdk.browser.wpe"
}
```

## version

The package version, this must consist of 1 to 4 numbers separated by periods (`.`). Numbers must not
include a leading zero. For example, `2.01` is not allowed; however, `0.2`, `2.0.1`, and `2.10` are allowed.

_Examples_

```json
{
  "version": "1.0.0"
}
```

```json
{
  "version": "0.1.20"
}
```

## versionName

Optional human-readable package version string, that represents the version of the package. There are no
restrictions on the version name, except that it if present it must not be an empty string. Typically, this
is represented as a [Semantic Versioning](https://semver.org), but is not mandatory.

If `versionName` is not present, then the `version` field is used as the version name.

_Examples_

```json
{
  "versionName": "1.0.1-beta"
}
```

## name

The name is a user readable string that represents the title of the package. It is not mandatory, and currently not
used by the EntOS platform.

There are no restrictions on the name except that if present it must not be an empty string.

_Examples_

```json
{
  "name": "My App"
}
```

## packageType

A `packageType` informs about type of the package and can have one of the following values:

- `base` - A package that contains a base layer for an application.
- `runtime` - A package that contains a runtime for an application.
- `application` - A package that contains an application. An application is optionally combined with a runtime to form
  a runnable application.
- `service` - A package that contains a service, which is an application that runs in the background. Likewise for
  applications, a service may be combined with a runtime to form a runnable service.
- `resource` - A package that contains resources, eg. certificates, translations, ML model.

_Examples_

Package consist application:

```json
{
  "packageType": "application"
}
```

## packageSpecifier

An optional `packageSpecifier` informs about package specifier and can have one of the following values:

- `html` - A package that is (or depends) on html (wpe) runtime.
- `lightnig` - A package that is (or depends) on lightning (wpe) runtime.
- `cobalt` - A package that is (or depends) on cobalt runtime.
- `flutter` - A package that is (or depends) on flutter runtime.
- `system` - A package that should be executed bcontains an application. An application is optionally combined with a runtime to form

New types (and combinations) can be introduced.

Optionally the combination of `packageType` and `packageSpecifier` can serve multiple purposes:

- Identifies the type of the package.
- It is used for associating an app or service package with a runtime package, eg. having

```json
{
  "packageType": "application",
  "packageSpecifier": "html"
}
```

package requires `html` runtime.

- Disovery and filtering based on package type or specifier, eg. only `html` applications.
- Based on type, specifier combination, it can implicitly assume default dependency eg. `"packageSpecifier": "html` -> `dependencies": { "rdk.browser.wpe": ">=1.1.0" }`
- Routing, not all packages needs to by run by Dobby, eg.

```json
{
  "packageType": "service",
  "packageSpecifier": "systemd"
}
```

_Examples_

Package consist HTML Runtime (WPE):

```json
{
  "pacakgeType": "runtime",
  "packageSpecifier": "html"
}
```

Package consist application that relies on HTML Runtime:

```json
{
  "packageType": "application",
  "packageSpecifier": "html"
}
```

## entryPoint

Every package has an entry point, this is a string and expected to be a path to a file within the package. The
interpretation of this path is up to the system / runtime that is using the package. Typically, for runtime packages
this will be the path to the `init` executable or script to run in a container. For `application` and `service` packages
this path may be passed to the runtime `init` to start the app or service.

The entry point path is relative to the root of the package.

_Examples_

```json
{
  "entryPoint": "SkyBrowserLauncher"
}
```

```json
{
  "entryPoint": "bin/NetflixApp"
}
```

```json
{
  "entryPoint": "web/index.html"
}
```

## entryArgs

Optional arguments for the entry point. It is an array of strings that is used together with `entryPoint`.
Empty array or absence of this field means no arguments are passed to the entry point.

_Examples_

```json
{
  "entryArgs": ["--some-option=true"]
}
```

```json
{
  "entryArgs": [
    "--some-option=true",
    "https://ytlr-cert.appspot.com/2021/main.html"
  ]
}
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
{
  "dependencies": {
    "rdk.browser.wpe": ">=1.1.0"
  }
}
```

### URLs as Dependencies

You may specify a tarball URL in place of a version range.

This tarball will be downloaded and installed locally at install time.

### Local Paths

You can provide a path to a local directory that contains a package.

_Examples_

```json
{
  "dependencies": {
    "foo-bar-relative": "../foo/bar",
    "foo-bar-absolute": "/foo/bar"
  }
}
```

This feature is helpful for local offline development and creating tests that require installing where you don't want to hit an external server.

## permissions

These array represents permissions that an `application` or `service` package is requesting. Runtime
packages do not request permissions, but the apps or services that run within the runtime may request them.

| Permission \*)                                                                            |
| ----------------------------------------------------------------------------------------- |
| _RDK_                                                                                     |
| [urn:rdk:permission:home-app](#urn:rdk:permission:home-app)                               |
| [urn:rdk:permission:internet](#urn:rdk:permission:internet)                               |
| [urn:rdk:permission:firebolt](#urn:rdk:permission:firebolt)                               |
| [urn:rdk:permission:thunder](#urn:rdk:permission:thunder)                                 |
| [urn:rdk:permission:rialto](#urn:rdk:permission:rialto)                                   |
| [urn:rdk:permission:game-controller](#urn:rdk:permission:game-controller)                 |
| [urn:rdk:permission:timeshift-buffer](#urn:rdk:permission:timeshift-buffer)               |
| [urn:rdk:permission:external-storage::read](#urn:rdk:permission:external-storage::read)   |
| [urn:rdk:permission:external-storage::write](#urn:rdk:permission:external-storage::write) |
| [urn:rdk:permission:display-overlay](#urn:rdk:permission:display-overlay)                 |
| [urn:rdk:permission:compositor](#urn:rdk:permission:compositor)                           |

\*) List is extensible

### RDK

- ### urn:rdk:permission:home-app

_Home App Permission_

App is requesting to become the home (launcher) app. An app with this privilege will be launched automatically when
the device boots and will appear when the user presses the home button.

This is typically used alongside the `DisplayOverlays` and `Compositor` privileges to allow the app to render
the home screen and control the system UI.

- ### urn:rdk:permission:internet

_Internet Permission_

The app or service simply requires internet access, or more specifically access to the external network.

- ### urn:rdk:permission:firebolt

_Firebolt Permission_

The app or service requires access to the Firebolt API. This privilege grants general access to Firebolt however,
additional Firebolt-specific privileges may be required to use certain APIs.

- ### urn:rdk:permission:thunder

_Thunder Permission_

The app or service requires access to the Thunder API. This privilege grants general access to Thunder; however,
additional Thunder-specific privileges may be required to access certain Thunder APIs.

- ### urn:rdk:permission:rialto

_Rialto Permission_

The app or service requires access to the Rialto API for A/V playback. Meaning at start up the app or service will
have a Rialto session created for it and can interact with the Rialto system.

- ### urn:rdk:permission:game-controller

_GameController Permission_

The app is requesting access to any connected game controller devices.

- ### urn:rdk:permission:timeshift-buffer

_Time Shift Buffer Permission_

The app or service requesting access to the time shift buffer.

- ### urn:rdk:permission:external-storage::read

_Read External Storage Permission_

The app or service is requesting access to be able to read data from attached external storage devices.
Those are typicallyl USB memory sticks.

- ### urn:rdk:permission:external-storage::write

_Write External Storage Permission_

The app or service is requesting access to be able to read and write data from attached external storage devices.
Those are typicallyl USB memory sticks.

- ### urn:rdk:permission:display-overlay

_Display Overlay Permission_

The app or service is requesting privilege to display overlays on the screen. Overlays are typically popups or
other UI elements that are drawn on top of the standard application UI.

- ### urn:rdk:permission:compositor

_Compositor Permission_

The application is requesting access to the Composition API. This allows the app to control the layout and
composition of the screen for all apps. In this sense, it acts like a basic window manager and is typically used
in conjunction with the `HomeApp` privilege to enable launcher or desktop-like functionality.

_Examples_

```json
{
  "permissions": [
    "urn:rdk:permission:home-app",
    "urn:rdk:permission.internet",
    "urn:rdk:permission:firebolt",
    "urn:rdk:permission:thunder",
    "urn:rdk:permission:rialto",
    "urn:rdk:permission:game-controller",
    "urn:rdk:permission:timeshift-buffer",
    "urn:rdk:permission:external-storage::read",
    "urn:rdk:permission:external-storage::write",
    "urn:rdk:permission:display-overlay",
    "urn:rdk:permission:compositor"
  ]
}
```

## configuration

Configuration object consist package specific configuration and settings.

| Configuration \*)                                                             | Base | Runtime | Application/Service |
| ----------------------------------------------------------------------------- | ---- | ------- | ------------------- |
| [urn:rdk:config:overrides](#urn:rdk:config:overrides)                         | N/A  | N/A     | Optional            |
| [urn:rdk:config:log-levels](#urn:rdk:config:log-levels)                       | N/A  | N/A     | Optional            |
| [urn:rdk:config:dial](#urn:rdk:config:dial)                                   | N/A  | N/A     | Optional            |
| [urn:rdk:config:application-lifecycle](#urn:rdk:config:application-lifecycle) | N/A  | N/A     | Optional            |
| [urn:rdk:config:platform](#urn:rdk:config:platform)                           | N/A  | N/A     | Optional            |
| [urn:rdk:config:input-handling](#urn:rdk:config:input-handling)               | N/A  | N/A     | Optional            |
| [urn:rdk:config:network](#urn:rdk:config:network)                             | N/A  | N/A     | Optional            |
| [urn:rdk:config:memory](#urn:rdk:config:memory)                               | N/A  | N/A     | Optional            |
| [urn:rdk:config:storage](#urn:rdk:config:storage)                             | N/A  | N/A     | Optional            |

\*) List is extensible

### urn:rdk:config:overrides

This allows to override defualt configuration of any of your dependencies eg. default web runtime user-agent.

- #### Application

  An undefined JSON blob that allows config values to be used by the application or service package.

- #### Runtime

  An undefined JSON blob that allows config values to be used by the runtime package.

- #### Base
  An undefined JSON blob that allows config values to be used by the base package.

_Object Schema_

```json
{
  "urn:rdk:config:overrides": {
    "description": "Window details.",
    "type": "object",
    "properties": {
      "application": {
        "description": "An undefined JSON Blog that allows to override config values used by the application.",
        "type": "object",
        "additionalProperties": true
      }
    },
    "properties": {
      "runtime": {
        "description": "An undefined JSON Blog that allows to override config values used by the runtime.",
        "type": "object",
        "additionalProperties": true
      }
    },
    "properties": {
      "base": {
        "description": "An undefined JSON Blog that allows to override config values used by the base layer.",
        "type": "object",
        "additionalProperties": true
      }
    }
  }
}
```

_Examples_

```json
{
  "configuration": {
    "urn:rdk:config:overrides": {
      "application": {
        "colorTheme": "vivid"
      },
      "runtime": {
        "userAgent": "BBC_requirement_requesting_Operator_Specific_UserAgent",
        "libInject": "oiptv.js",
        "mTls": "/etc/certs/client_operator.cert"
      }
    }
  }
}
```

### urn:rdk:config:log-levels

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
{
  "urn:rdk:config:log-levels": {
    "description": "Logging levels.",
    "type": "array",
    "items": {
      "type": "string",
      "enum": ["fatal", "error", "warning", "milestone", "info", "debug"]
    }
  }
}
```

_Examples_

```json
{
  "configuration": {
    "urn:rdk:config:log-levels": ["error", "warning", "milestone"]
  }
}
```

### urn:rdk:config:dial

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
{
  "urn:rdk:config:dial": {
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
          "type": "string"
        }
      },
      "originHeaderRequired": {
        "type": "boolean"
      }
    }
  }
}
```

_Examples_

```json
{
  "configuration": {
    "urn:rdk:config:dial": {
      "appNames": ["uk.co.bbc.iPlayer", "uk.co.bbc.Sport", "uk.co.bbc.News"],
      "corsDomains": ["http://example.com", "https://bbc.co.uk"],
      "originHeaderRequired": true
    }
  }
}
```

### urn:rdk:config:application-lifecycle

Set of the lifecycle related configs for the app or service.

- #### Supported Non-active Lifecycle States
  Supported non-active lifecycle states by the application.
  Valid configurations are:
  - `[]`
  - `["paused"]`
  - `["paused","suspended"]`
  - `["paused","suspended","hibernated"]`
- #### Maxiumum Suspended System Memory
  The maximum system memory allowed for the app when suspended.
- #### Maximum Time To Suspendd Memory State
  The maximum time to reduce the system memory usage to its `MaxSuspendedSystemMemory` after entering suspended state.
- #### Startup Timeout
  The time in seconds that the system will wait for the app or service to signal it has started before terminating it.
- #### Watchdog Interval
  The time in seconds that the system will wait for the app or service to signal the watchdog before terminating it.
  The watchdog is not started until the first signal is received from the app or service.

_Object Schema_

```json
{
  "urn:rdk:config:application-lifecycle": {
    "description": "Set of the supported lifecycle parameters for the app or service.",
    "type": "object",
    "properties": {
      "supportedNonActiveStates": {
        "type": "array",
        "items": {
          "type": "string"
        },
        "description": "Array of supported  non-active lifecycle states."
      },
      "maxSuspendedSystemMemory": {
        "type": "string",
        "pattern": "^\\d+[GMB]?$"
      },
      "maxTimeToSuspendMemoryState": { "type": "integer" },
      "startupTimeout": { "type": "integer" },
      "watchdogInteval": { "type": "integer" }
    },
    "required": [
      "supportedNonActiveStates",
      "maxSuspendedSystemMemory",
      "maxTimeToSuspendMemoryState"
    ],
    "additionalProperties": false
  }
}
```

_Examples_

```json
{
  "configuration": {
    "urn:rdk:config:application-lifecycle": {
      "supportedNonActiveStates": ["paused", "suspended", "hibernated"],
      "maxSuspendedSystemMemory": "16M",
      "maxTimeToSuspendMemoryState": "10",
      "startupTimeout": 60,
      "watchdogInterval": 30
    }
  }
}
```

### urn:rdk:config:platform

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
{
  "urn:rdk:config:platform": {
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
}
```

_Examples_

```json
{
  "configuration": {
    "urn:rdk:config:platform": {
      "architecture": "arm",
      "variant": "v7",
      "os": "linux"
    }
  }
}
```

### urn:rdk:config:input-handling

The input handling configuration for the app. This is optional and allows the app to capture or be notified of certain
input events.

- #### Key Intercept Set

  Input keys that an app receives while running, regardless of whether it currently has focus.

- #### Key Capture Set

  Input keys that an app always receives when it has focus, overriding any intercept keys from other apps,
  and preventing those keys from being forwarded to other apps.

- #### Key Monitor Set

  Input keys that an app always receives when it has focus, but those keys are also sent to any app with intercept keys set.

_Object Schema_

```json
{
  "urn:rdk:config:input-handling": {
    "description": "Input handling configuration.",
    "type": "object",
    "properties": {
      "keyIntercept": { "type": "array", "items": { "type": "string" } },
      "keyCapture": { "type": "array", "items": { "type": "string" } },
      "keyMonitor": { "type": "array", "items": { "type": "string" } }
    }
  }
}
```

_Examples_

```json
{
  "configuration": {
    "urn:rdk:config:input-handling": {
      "keyCapture": ["search", "voice"],
      "keyCapture": ["search", "voice"],
      "keyMonitor": ["volume+", "volume-"]
    }
  }
}
```

### urn:rdk:config:window

The requested window details for the app.

- #### Virtual Display Size
  The display size requested by the app, e.g. `1080`, `720`. Note that this typically only controls the virtual
  display resolution for the app, the wayland output display size. The actual final composition and / or HDMI
  display size is controlled by the system.

_Object Schema_

```json
{
  "urn:rdk:config:window": {
    "description": "Window details.",
    "type": "object",
    "properties": {
      "virtualDisplaySize": { "type": "string" }
    }
  }
}
```

_Examples_

```json
{
  "configuration": {
    "urn:rdk:config:window": {
      "virtualDisplaySize": 720
    }
  }
}
```

### urn:rdk:config:network

And app or service can request access to or expose a network services. Network services are divided into three
categories:

- #### public
  Network services that an app or service exposes outside the device.
- #### exported
  Service that an app or service exposes to other apps or services on the device.
- #### imported
  Network services supplied by another app or service that the current app requires access to.

`exported` and `imported` services are related, if an app exports a service then another app can import that service,
but the `port` and `protocol` must match.

In practical terms, these settings correspond to firewall rules that are applied to the app or service in the container.
It's up to the app or service to actually listen on the ports and handle the network traffic.

> [!NOTE]
> Currently this configuration uses fixed ports defined by the app or service. In the future, we'll look at adding
> options for dynamic port assignment.

_Object Schema_

```json
{
  "urn:rdk:config:network": {
    "description": "Network services configuration.",
    "type": "object",
    "properties": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string" },
          "port": { "type": "integer" },
          "protocol": { "type": "string" },
          "type": { "type": "string" }
        },
        "required": ["name", "port", "protocol", "type"]
      }
    }
  }
}
```

_Examples_

```json
{
  "configuration": {
    "urn:rdk:config:network": [
      {
        "name": "netflix-mdx",
        "port": 8009,
        "protocol": "tcp",
        "type": "public"
      },
      {
        "name": "com.example.myapp.service",
        "port": 1234,
        "protocol": "tcp",
        "type": "exported"
      },
      {
        "name": "com.example.someotherapp.service",
        "port": 4567,
        "protocol": "tcp",
        "type": "imported"
      }
    ]
  }
}
```

### urn:rdk:config:memory

The requested memory quota for the app or service. This is optional and used as a hint to the system about the required
memory usage. The system may use this to determine when to run the app or service, or to apply limits on the memory
usage.

- #### System Memory
  The amount of system memory required by the app or service.
- #### GPU Memory
  The amount of GPU memory required by the app or service.

_Object Schema_

```json
{
  "urn:rdk:config:memory": {
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
    },
    "required": ["system", "gpu"]
  }
}
```

_Examples_

```json
{
  "configuration": {
    "urn:rdk:config:memory": {
      "system": "256M",
      "gpu": "128M"
    }
  }
}
```

### urn:rdk:config:storage

The requested storage quota for the app or service (in MB). This is optional and used as a hint to the system about the
amount of storage for the app or service.

- #### Max Local Storage Size
  The maximum size of the local storage associated with the app.
- #### Shared Storage App Id
  The App Id of the app that shared its local storage within this app.

_Object Schema_

```json
{
  "urn:rdk:config:storage": {
    "description": "Storage quota. Value can be specified with G, M, or B suffix. If no suffix is provided, the value is assumed to be in bytes.",
    "type": "object",
    "properties": {
      "maxLocalStorage": {
        "type": "string",
        "pattern": "^\\d+[GMB]?$"
      },
      "sharedStorageAppId": {
        "type": "string",
        "pattern": "^(?!.*\\.\\.)[a-zA-Z0-9](?:[a-zA-Z0-9._-]*[a-zA-Z0-9])?$"
      }
    },
    "required": ["maxLocalStorage"]
  }
}
```

_Examples_

```json
{
  "configuration": {
    "urn:rdk:config:storage": {
      "maxLocalStorage": "32M",
      "sharedStorageAppId": "com.sky.myapp2"
    }
  }
}
```
