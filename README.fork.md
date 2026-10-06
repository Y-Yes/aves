# Aves — patched fork (`Y-Yes/aves`)

Fork of [deckerst/aves](https://github.com/deckerst/aves) with a fix for
[issue #1040](https://github.com/deckerst/aves/issues/1040): video audio through the mpv backend
is muffled, mono and too quiet on Pixel-class devices.

Not intended to be merged upstream. Builds are published on this fork's
[Releases](https://github.com/Y-Yes/aves/releases) page.

## Background

`audiotrack` is the mpv audio output that sounds right on these devices; the bundled default,
`opensles`, downmixes to mono and is quiet. Using `audiotrack` only became safe once mpv allowed
several JNI instances of it (`46fe3cded0`, cf
[media-kit#1061](https://github.com/media-kit/media-kit/issues/1061)), and that fix is not in the
libmpv revision that media-kit pins. So this can only be fixed by rebuilding that dependency chain
from the bottom up, which is probably why the issue has been stale for a while. This is my own
build of it for personal use, not an attempt to get it upstreamed:

1. **[Y-Yes/libmpv-android-video-build](https://github.com/Y-Yes/libmpv-android-video-build)** —
   fork of media-kit's libmpv build with that fix backported. Its CI publishes the rebuilt
   `default-<abi>.jar` files in the **`vnext`** release.

2. **[Y-Yes/media-kit](https://github.com/Y-Yes/media-kit)** — fork whose
   `libs/android/media_kit_libs_android_video/android/build.gradle` `filesToDownload` list
   points at those jars instead of the upstream media-kit release.

3. **This fork** — `dependency_overrides` redirects `media_kit_libs_android_video` to the fork
   above, and the `ao` property is set before media is opened.

## Changes relative to upstream

| file | change |
|---|---|
| `pubspec.yaml` | `dependency_overrides` for `media_kit_libs_android_video` → `Y-Yes/media-kit` |
| `flavors/pubspec_{play,izzy,libre}.yaml` | the same override (the flavor scripts copy these over `pubspec.yaml`, so it must be present there too) |
| `pubspec.lock`, `flavors/pubspec_*.lock` | git-dependency entries whose `resolved-ref` pins the fork commit |
| `plugins/aves_video_mpv/lib/src/controller_video.dart` | Android-only `setProperty('ao', 'audiotrack,opensles')` before `open()` |
| `plugins/aves_video_mpv/lib/src/controller_audio.dart` | the same, plus the `dart:io` import |
| `.github/workflows/release.yml` | release body text |
| `pubspec.yaml` (`version:`) | build number bump |

Rebasing onto upstream only touches those files.

## Installing

The `libre` flavor is the one for de-Googled ROMs (LineageOS, GrapheneOS, …). It has the same
applicationId as F-Droid's build but a different signing key, so uninstall the original Aves
first — export your data with Settings → Export beforehand — then install the modified APK from
this fork's Releases page.

To get updates, add this repo to [Obtainium](https://github.com/ImranR98/Obtainium) with an asset
filter of `app-libre-arm64-v8a-release.apk`.
