# Native Runtime Package

Public installation and integration guide for the compiled detection runtime.

## Overview

The native runtime provides GPU-accelerated model loading and image detection for the host application. It is delivered as a platform-specific Python extension and is used through the supplied `movement_tracking.py` adapter.

Do not load `detection.dll` or `detection.so` directly with `ctypes`, C, C++, or C#. The adapter selects the correct file and provides the supported Python interface.

## Supported systems

| System | Architecture | Python | Runtime file |
|---|---|---|---|
| Windows 10 or 11 | x86-64 | CPython 3.11, 64-bit | `detection.dll` |
| Linux | x86-64 | CPython 3.10, 64-bit | `detection.so` |

An NVIDIA GPU, a compatible NVIDIA driver, and the Python dependencies supplied for the host application are required.

## Main package files

| File | Purpose |
|---|---|
| `movement_tracking.py` | Public Python adapter used by the host application |
| `native/windows-x86_64/detection.dll` | Windows native runtime |
| `native/linux-x86_64/detection.so` | Linux native runtime |
| `native/detection_settings.xml` | Runtime detection configuration |
| `native/movement_tracking_manifest.json` | Release metadata used during runtime loading |
| `models/model.engine` | Fixed TensorRT detection model loaded at application startup |

Keep the supplied directory layout and all additional release files unchanged. Files from different releases must not be mixed.

## Install

1. Stop the host application completely.
2. Copy the complete supplied runtime package into the host application's root directory.
3. Preserve all filenames and subfolders exactly.
4. Confirm that the TensorRT model is present at `models/model.engine`.
5. Activate the supported Python environment for the operating system.
6. Start the host application with its normal launcher or service.
7. Wait until model loading reports completion before starting live detection.

The first model load can take longer while the GPU runtime initializes. A successful startup should identify the model, initialize the execution provider, warm up the engine, and report the runtime as ready.

## Python integration

Import the adapter, not the native binary:

```python
import movement_tracking
```

Load the fixed production model and display startup progress:

```python
import movement_tracking


def show_progress(percent, message):
    print(f"{percent:3d}% | {message}")


runtime = movement_tracking.load_runtime_model(show_progress)
print(movement_tracking.status())
```

The host application normally performs this step once during startup. Repeated loading is unnecessary.

## Detect an image

Frames are supplied as NumPy/OpenCV images in BGR order:

```python
import cv2
import movement_tracking

frame = cv2.imread("sample.png")
if frame is None:
    raise RuntimeError("The sample image could not be read")

result = movement_tracking.detect(
    frame,
    confidence=0.05,
    iou=0.45,
    mode="balanced",
    encode_preview=True,
)

for item in result.get("detections", []):
    print(item)

preview = movement_tracking.render_result_image()
```

Use `encode_preview=False` when the caller needs detection data only. The host application may apply class-specific thresholds and movement rules in addition to the values shown in this basic example.

## Detection modes

| Mode | Recommended use |
|---|---|
| `fast` | Lowest latency when throughput is the priority |
| `balanced` | General production use |
| `accuracy` | More detailed processing when latency is less important |

Use lower-case mode names. Start with `balanced` unless the deployment has been performance-tested with another mode.

## Runtime status

Call `movement_tracking.status()` to obtain the current runtime state. Typical information includes whether the model is loaded, the active model path, the selected device, and current runtime availability. Treat the returned object as diagnostic data; do not modify it.

## Stop safely

Use the normal service stop command or press `Ctrl+C` once in the server console. Allow the shutdown sequence to finish. The host application should:

1. Stop live detection and reject new frames.
2. Wait for active work to finish.
3. Unload the model and release GPU resources.
4. Release the camera.
5. Stop the application service.

For a direct Python integration, release the model in a `finally` block:

```python
try:
    # Run the application workload.
    pass
finally:
    movement_tracking.unload_model()
```

Do not terminate the process or power off the computer while shutdown is still in progress.

## Update

1. Stop live detection and close the host application.
2. Back up the currently installed runtime package.
3. Replace the supplied adapter, native folder, and related release files as one matching set.
4. Replace `models/model.engine` only when a compatible production model is intentionally being deployed.
5. Restart the application and confirm that model loading completes.
6. Run one image test and one controlled live-camera test before returning to production.

## Verification checklist

- The operating system, architecture, and Python version match the table above.
- The correct platform binary exists in the supplied native folder.
- `models/model.engine` exists and is a compatible TensorRT detection engine.
- The NVIDIA driver can see the installed GPU.
- Startup reaches the ready state without an exception.
- A test image returns a valid result object.
- Live detection starts and stops normally.
- `Ctrl+C` completes the full shutdown sequence.

## Troubleshooting

| Problem | Action |
|---|---|
| Runtime file is missing | Restore the complete supplied package and preserve its directory structure. |
| Python import fails | Confirm the exact supported CPython version and 64-bit architecture. |
| Release validation fails | Reinstall one complete matching release. Do not combine old and new runtime files. |
| `models/model.engine` is missing | Place the compatible production engine at the required fixed path. |
| Model is not a YOLO detection engine | Deploy the correct TensorRT detection engine built for this runtime and GPU environment. |
| CUDA or TensorRT initialization fails | Check the NVIDIA driver, CUDA/TensorRT dependencies, and GPU compatibility. |
| Detection returns no results | Verify the image, model classes, confidence threshold, and detection settings. |
| Address or port is already in use | Stop the existing application instance before starting another one. |
| Shutdown appears slow | Wait for active detection work and GPU cleanup to complete before forcing termination. |

When requesting support, provide the operating system, Python version, GPU model, NVIDIA driver version, runtime release identifier, and the exact startup error. Do not send credentials, private keys, datasets, or unrelated system files.

## Distribution

Distribute the approved native runtime, adapter, required configuration, public documentation, and published file hashes as one release. Do not rename or modify runtime files, and do not distribute private source code, build tooling, debug symbols, generated build files, or internal engineering documentation.
