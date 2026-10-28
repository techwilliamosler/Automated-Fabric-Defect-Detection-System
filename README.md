# Native Runtime Package

Public installation and update instructions.

## Purpose

This package provides the compiled runtime used by the host application. It is not a standalone program and must not be loaded directly with `ctypes`, C, C++, or C#.

## Supported systems

| System | Architecture | Python | Runtime file |
|---|---|---|---|
| Windows 10/11 | x86-64 | CPython 3.11, 64-bit | `detection.dll` |
| Linux | x86-64 | CPython 3.10 | `detection.so` |

An NVIDIA GPU, compatible driver, and the dependencies supplied for the host application are required.

## Required package files

Keep the supplied folder layout unchanged:

```text
native/
  windows-x86_64/detection.dll
  linux-x86_64/detection.so
  movement_tracking_manifest.json
  SystemCheckscum.xml
  detection_settings.xml
```

The runtime binary, signed manifest, and XML files must belong to the same approved release.

## Install

1. Stop the host application completely.
2. Copy the complete supplied `native` folder into the host application, preserving every filename and subfolder.
3. Confirm that the production model supplied for the installation is present as `models/model.engine`.
4. Start the host application using its normal launcher or service.
5. Wait for the startup check and model loading to finish before using live operation.

The host application automatically selects the correct runtime file. Do not import or open the DLL or SO directly.

## Update

1. Stop live operation and close the host application.
2. Back up the currently installed `native` folder.
3. Replace the DLL, SO, manifest, and XML files with the complete new release set.
4. Start the host application and confirm that startup completes normally.

Do not combine files from different releases.

## Renew SystemCheckscum

When an authorized replacement `SystemCheckscum.xml` is provided:

1. Stop the host application.
2. Replace only `native/SystemCheckscum.xml`.
3. Start the host application again.

Do not edit the XML file manually.

## Stop safely

Use the normal service stop command or press `Ctrl+C` in the server console. Allow the shutdown sequence to finish before closing the console or powering off the computer.

## Troubleshooting

| Problem | Action |
|---|---|
| Native runtime validation fails | Reinstall the complete approved native release. Do not mix old and new files. |
| Runtime file is reported missing | Confirm that the supplied folder layout and filenames were preserved. |
| Python compatibility error | Use CPython 3.11 on Windows or CPython 3.10 on Linux. |
| Model cannot be loaded | Confirm that `models/model.engine`, the NVIDIA driver, and GPU dependencies are installed. |
| Address or port is already in use | Stop the other running instance before starting the application again. |
| SystemCheckscum failure | Install the current authorized `SystemCheckscum.xml` and restart the application. |

## Public distribution rules

- Distribute only the approved runtime package and this public documentation.
- Do not rename, modify, patch, inspect, or repackage the runtime files.
- Do not distribute private source code, build tools, signing material, test reports, generated C files, debug symbols, or internal documentation.
- Verify file hashes against the release information provided with the package.
