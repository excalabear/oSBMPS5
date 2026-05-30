<div align="center">
  <img src="assets/osbm/osbmPixelArt-8x.png" alt="OpenStarbound Mobile pixel art banner" width="760">
  <h1>OpenStarbound Mobile</h1>
  <p><strong>A mobile-focused port of OpenStarbound for Android and iOS devices.</strong></p>
  <p>
    <a href="https://github.com/RohanBhattacharyya/oSBM/releases">
      <img alt="Latest release" src="https://img.shields.io/github/v/release/RohanBhattacharyya/oSBM?include_prereleases&label=release&style=for-the-badge&color=00d1b2">
    </a>
    <a href="https://github.com/RohanBhattacharyya/oSBM/actions/workflows/android-manual.yml">
      <img alt="Android workflow" src="https://img.shields.io/github/actions/workflow/status/RohanBhattacharyya/oSBM/android-manual.yml?branch=main&label=android%20apk&style=for-the-badge&logo=android&logoColor=white">
    </a>
    <a href="https://github.com/RohanBhattacharyya/oSBM/actions/workflows/ios-manual.yml">
      <img alt="iOS workflow" src="https://img.shields.io/github/actions/workflow/status/RohanBhattacharyya/oSBM/ios-manual.yml?branch=main&label=ios%20ipa&style=for-the-badge&logo=apple&logoColor=white">
    </a>
  </p>
  <p>
    <img alt="Android ARM64" src="https://img.shields.io/badge/Android-ARM64-3ddc84?style=flat-square&logo=android&logoColor=white">
    <img alt="iOS ARM64" src="https://img.shields.io/badge/iOS-ARM64-000000?style=flat-square&logo=apple&logoColor=white">
    <img alt="CMake 3.23 or newer" src="https://img.shields.io/badge/CMake-3.23%2B-064f8c?style=flat-square&logo=cmake&logoColor=white">
    <img alt="Java 17" src="https://img.shields.io/badge/Java-17-f89820?style=flat-square&logo=openjdk&logoColor=white">
  </p>
</div>

You **must** own a copy of Starbound. Base game assets are not provided for very obvious reasons.

The code is worked on whenever I have the spare time. Contributions and issues are welcome!

## Installation
### Download the [latest release](https://github.com/RohanBhattacharyya/oSBM/releases).
The latest stable release is recommended, as the nightly build can have bugs.

At the moment, you must have a copy of the game assets (**packed.pak**) to upload to the app.


## Building

<details>
<summary>Android (ARM64 APK)</summary>

#### Requirements
* CMake 3.23+
* Ninja
* Java 17
* Android SDK with:
  * `platform-tools`
  * `platforms;android-35`
  * `build-tools;35.0.0`
  * `ndk;26.3.11579264`
* vcpkg (set `VCPKG_ROOT`)

#### Environment variables
Set these before configuring:

```bash
export VCPKG_ROOT=/absolute/path/to/vcpkg
export ANDROID_SDK_ROOT=/absolute/path/to/android-sdk
export ANDROID_NDK_HOME="$ANDROID_SDK_ROOT/ndk/26.3.11579264"
```

#### Configure and build
From the repository `source/` directory:

```bash
cmake --preset android-arm64-release
cmake --build --preset android-arm64-release --target android_apk --parallel
```

Debug build:

```bash
cmake --preset android-arm64-debug
cmake --build --preset android-arm64-debug --target android_apk --parallel
```

#### Build outputs
* Release APK: `source/application/mobile/android/host/app/build/outputs/apk/release/app-release.apk`
* Debug APK: `source/application/mobile/android/host/app/build/outputs/apk/debug/app-debug.apk`
</details>
<details>
<summary>iOS (ARM64)</summary>

#### Requirements
* macOS (Xcode 15+ recommended)
* Xcode command line tools
* CMake 3.23+
* vcpkg (set `VCPKG_ROOT`)

#### Option A: compile-only check (preset)
From `source/`:

```bash
cmake --preset ios-arm64-debug
cmake --build --preset ios-arm64-debug --target starbound --parallel
```

#### Option B: build unsigned `.ipa` locally (matches CI workflow)
From repository root:

```bash
cmake -S source -B build/ios-arm64-ipa \
  -G Xcode \
  -DCMAKE_TOOLCHAIN_FILE="$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake" \
  -DVCPKG_OVERLAY_TRIPLETS="$PWD/triplets" \
  -DCMAKE_SYSTEM_NAME=iOS \
  -DCMAKE_OSX_SYSROOT=iphoneos \
  -DCMAKE_OSX_ARCHITECTURES=arm64 \
  -DCMAKE_BUILD_TYPE=Release \
  -DVCPKG_TARGET_TRIPLET=arm64-ios-mixed \
  -DSTAR_ENABLE_STEAM_INTEGRATION=OFF \
  -DSTAR_ENABLE_DISCORD_INTEGRATION=OFF \
  -DCMAKE_XCODE_ATTRIBUTE_CODE_SIGNING_ALLOWED=NO \
  -DCMAKE_XCODE_ATTRIBUTE_CODE_SIGNING_REQUIRED=NO \
  -DCMAKE_XCODE_ATTRIBUTE_CODE_SIGNING_IDENTITY=""
```

Build and package the unsigned `.ipa` in one step using the `ipa` CMake target:

```bash
cmake --build build/ios-arm64-ipa --config Release --parallel --target ipa
```

The `ipa` target:
1. Builds `starbound` (and all dependencies) if out of date
2. Deletes any previous `ipa/Payload/` directory and unsigned `.ipa`
3. Copies the full current app bundle (binary + assets + Info.plist) into `ipa/Payload/OpenStarbound.app/`
4. Zips the result into the final `.ipa`

Do not package a manually staged `Payload` directory. Re-run the `ipa` target
whenever you need a fresh installable build; it is the only step that writes the
final IPA payload.

Resulting file:
* `build/ios-arm64-ipa/ipa/OpenStarbound-iOS-Release-unsigned.ipa`
</details>
<details>
<summary>CI (manual workflows)</summary>

If you prefer GitHub Actions artifacts instead of local packaging:
* Android manual workflow: `.github/workflows/android-manual.yml`
* iOS manual workflow: `.github/workflows/ios-manual.yml`
</details>
