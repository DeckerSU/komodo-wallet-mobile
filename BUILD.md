# Build Guide

## Prerequisites

This project requires **Flutter 2.8.1** and is managed via [FVM (Flutter Version Manager)](https://fvm.app/).
Do **not** use the system-wide Flutter installation — the SDK constraint in `pubspec.yaml` is `>=2.8.1 <3.0.0`,
which is incompatible with Dart 3.x.

Install FVM and the required Flutter version:

```bash
dart pub global activate fvm
fvm install 2.8.1
fvm use 2.8.1
```

## KDF (MM2) Library

The app embeds the [Komodo DeFi Framework](https://github.com/KomodoPlatform/komodo-defi-framework)
as a prebuilt static library (`libmm2.a`). This library must be present **before** running the Flutter
build. It is placed into:

| ABI | Path |
|-----|------|
| `arm64-v8a` | `android/app/src/main/cpp/libs/arm64-v8a/libmm2.a` |
| `armeabi-v7a` | `android/app/src/main/cpp/libs/armeabi-v7a/libmm2.a` |
| iOS | `ios/libmm2.a` |

CMake (`android/app/src/main/cpp/CMakeLists.txt`) links the static library into `libmm2-lib.so`
via a JNI wrapper (`mm2_native.cpp`), which is then loaded at runtime by `MainActivity.java`.

### Option 1: Download prebuilt binaries (recommended)

The easiest way. Downloads release artifacts from GitHub based on the tag configured in
`.docker/build_config.json` (currently `v2.3.0-beta`):

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r .docker/requirements.txt
python .docker/update_api.py --force
```

### Option 2: Docker multi-stage build

Builds the KDF library from source and then builds the APK inside Docker:

```bash
# Stage 1: compile libmm2.a from Komodo DeFi Framework source (Rust nightly)
docker build -f .docker/kdf-android-build.dockerfile -t kdf-android-build .

# Stage 2: copy compiled binaries and build the Flutter APK
docker build -f .docker/android-apk-build.dockerfile -t android-apk-build .
```

The Rust build step inside Docker compiles:
```bash
cargo rustc --target=aarch64-linux-android    --lib --release --crate-type=staticlib --package mm2_bin_lib
cargo rustc --target=armv7-linux-androideabi  --lib --release --crate-type=staticlib --package mm2_bin_lib
```

The output `libkdflib.a` is renamed to `libmm2.a` and copied into the paths listed above.

### Option 3: Devcontainer / Codespaces

The devcontainer setup (`.devcontainer/dev-setup.sh`) clones komodo-defi-framework into `/kdf`
and compiles `libmm2.a` from source automatically on container creation.

## Building the APK

Once `libmm2.a` is in place:

```bash
fvm flutter build apk
```

For a debug build:

```bash
fvm flutter build apk --debug
```
