# Ellipse Converter: source code of its open-source libraries

[Ellipse Converter](https://ellipseapps.pages.dev/converter/) for Android is built on open-source
libraries, several of them under the GNU Lesser General Public License (LGPL): FFmpeg, FFmpegKit,
LAME, FriBidi, libiconv, libheif and libde265. This repository publishes the complete
corresponding source of those libraries as they are built into each released version of the app,
together with the scripts that build them and a record of every change made to them.

The app itself is not open source. This repository covers only the libraries it uses.

## Downloads

| App version | Version code | Source |
|---|---|---|
| 0.9.0 (shown as 0.9.0-testers in the testing build) | 9 | [Release v0.9.0](https://github.com/ellipseapps/ellipse-converter-source/releases/tag/v0.9.0) |
| 0.8.0 (shown as 0.8.0-testers in the testing build) | 8 | [Release v0.8.0](https://github.com/ellipseapps/ellipse-converter-source/releases/tag/v0.8.0) |
| 0.7.0 (shown as 0.7.0-testers in the testing build) | 7 | [Release v0.7.0](https://github.com/ellipseapps/ellipse-converter-source/releases/tag/v0.7.0) |
| 0.6.0 (shown as 0.6.0-testers in the testing build) | 6 | [Release v0.6.0](https://github.com/ellipseapps/ellipse-converter-source/releases/tag/v0.6.0) |
| 0.5.0 (shown as 0.5.0-testers in the testing build) | 5 | [Release v0.5.0](https://github.com/ellipseapps/ellipse-converter-source/releases/tag/v0.5.0) |
| 0.4.0 (shown as 0.4.0-testers in the testing build) | 4 | [Release v0.4.0](https://github.com/ellipseapps/ellipse-converter-source/releases/tag/v0.4.0) |
| 0.3.0 (shown as 0.3.0-testers in the testing build) | 3 | [Release v0.3.0](https://github.com/ellipseapps/ellipse-converter-source/releases/tag/v0.3.0) |
| 0.2.0 (shown as 0.2.0-testers in the testing build) | 2 | [Release v0.2.0](https://github.com/ellipseapps/ellipse-converter-source/releases/tag/v0.2.0) |

Each release holds:

| File | Contents |
|---|---|
| `native-corresponding-source.tar.gz` | FFmpegKitNext 9.0.0 with FFmpeg 9.0.1 and every library built into it (LAME, Opus, libvpx, libaom, OpenH264, Kvazaar, libwebp, libass, FreeType, HarfBuzz, FriBidi, fontconfig, libiconv, zimg and their dependencies), as configured and built, including generated configuration. |
| `libheif-<version>-source.tar.xz`, `libde265-<version>-source.tar.xz` | The HEIF image libraries. |
| `image-adapter-source.tar.xz` | The small JNI adapter that links the app to libheif, with its build script. |
| `rebuild-support.tar.xz` | The build scripts, the lock files that pin every source revision and build flag, the Kvazaar patch and the list of packaged notices. |
| `build-records.tar.xz` | The build flags, source revisions and checksums of the built libraries, and diffs of every difference between the sources used and their upstream revisions. |
| `SHA256SUMS.txt` | Checksums of the files above. |

When a version's libraries are unchanged, its release links to the earlier release's
`native-corresponding-source.tar.gz`, the same file byte for byte, instead of attaching a second
copy; its notes give the link and the checksum.

## Changes we made

- 2026-09-19: FFmpegKitNext's Android library build (`android/ffmpeg-kit-next-android-lib/build.gradle`)
  targets minimum API 26 instead of 24.
- 2026-09-19: FFmpegKitNext's Kvazaar build script (`scripts/android/kvazaar.sh`) applies
  `kvazaar-rdcost-lifetime.patch`, which fixes the teardown of Kvazaar's optional
  rate/distortion training state, including on repeated HEIC alpha encodes.

`build-records/upstream-build.diff` shows both changes in full. The diffs in
`build-records/native-source-patches/` show every difference between the sources that were built
and their upstream revisions; apart from the Kvazaar patch, FFmpegKitNext's own build scripts make
them.

## Building

The libraries are built on Linux for `arm64-v8a`, `armeabi-v7a` and `x86_64` at API 26, with the
Android NDK r27d (27.3.13750724). You need JDK 17, the Android SDK with platform 34, Build Tools
35.0.0, CMake 3.22.1 and that NDK, plus autoconf, automake, libtool, pkg-config, gettext, bison,
meson, ninja, nasm, yasm, gperf and doxygen.

1. Unpack `rebuild-support.tar.xz` into an empty directory. It holds `scripts/`, `third_party/` and
   the adapter's `engine/image/src/main/cpp/`.
2. Run `./scripts/build_ffmpeg.sh`. It checks out FFmpegKitNext at the locked revision into
   `.cache/ffmpeg-kit-next`, stages the Kvazaar patch and runs FFmpegKitNext's `android.sh` with the
   locked arguments. The result is an Android library (AAR) under `artifacts/native/maven`.
3. Run `python3 scripts/build_image_codecs.py`. It builds libde265, libheif and the adapter into
   `artifacts/image-native/jni/<abi>/`.

To build from the archives here instead of downloading, unpack `native-corresponding-source.tar.gz`
and run its `android.sh` with the arguments recorded in `third_party/ffmpeg-kit-next.lock.json`, and
build the HEIF archives with CMake using the flags in `third_party/image-codecs.lock.json`.
The archives carry no Git metadata; initialise a repository in a source directory if a script asks
for one.

## Replacing the libraries in the app

The libraries are separate shared libraries (`lib/<abi>/lib*.so`) inside the app, and the app does
not check its own signature or reject replaced libraries. To run the app with a library you
modified:

1. Build the library for your phone's ABI as above, keeping its interface compatible.
2. Copy the app's APKs from your phone: `adb shell pm path app.ellipse.converter`, then `adb pull`
   each path. The native libraries are in the split for your ABI, for example
   `split_config.arm64_v8a.apk`.
3. Replace the `.so` files in that split, stored uncompressed, and align it with
   `zipalign -P 16 -f 4`.
4. Sign every APK with your own key (`apksigner sign`), uninstall the copy from Google Play (its
   signature differs) and install yours with `adb install-multiple`.

From 0.3.0, FFmpegKit's Java classes keep their own names inside the app's `classes*.dex`, so a
modified FFmpegKit can replace them as well as its native libraries.

Features that depend on Google Play, such as the subscription, may not work in a copy signed with
another key.

## Licences

Each library's licence is inside its archive (for example FFmpeg's `COPYING.LGPLv2.1` and
`COPYING.LGPLv3`, and libheif's `COPYING`), and the app shows all of them under Settings →
Open-source licenses. The build scripts, lock files and adapter published here are provided under
the GNU Lesser General Public License, version 3 or (at your option) any later version.

## Written offer

For at least three years after we last distribute a version of the app, we will also provide the
corresponding source for that version, on a physical medium or by download, for no more than our
cost of doing so. Write to ellipseapps@gmail.com.
