# ZephVideoCompresser WIP

A native macOS app that compresses videos locally using **AVFoundation** — no
ffmpeg, no uploads, no accounts. Pick a video, choose a resolution / codec /
optional size cap, and save the compressed file. `The Name is Intentional LoL`

## Features
- Drag-and-drop or file picker to load any video AVFoundation can open (`.mp4`, `.mov`, `.m4v`, …).
- Resolution presets: Original / 1080p / 720p / 480p / 360p (longest-side scaling, aspect-preserving).
- Codec: **HEVC (H.265)** or **H.264**.
- Optional **max file-size** cap — the engine derives a bitrate that fits your budget.
- Saves via a Save dialog every time (default name `<name>-compressed.mp4`).
- Before/after size and % saved, with Reveal / Open in Finder.

## Build

Requires Xcode Command Line Tools (Swift 5.7+). No Homebrew or ffmpeg needed.

```bash
cd ZephVideoCompresser
./build.sh
```

This compiles a release build, assembles `ZephVideoCompresser.app`, and ad-hoc
codesigns it. Then:

```bash
open ZephVideoCompresser.app
```

> The app is ad-hoc signed (no Apple Developer ID), so it opens fine locally.
> If macOS quarantines it, run `xattr -dr com.apple.quarantine ZephVideoCompresser.app`
> once, or right-click → Open.

## Command line

`zephctl` is bundled for headless use / power users:

```bash
# Inspect a file
zephctl info "movie.mov"

# Compress
zephctl compress "movie.mov" out.mp4 --preset 720p --codec hevc
zephctl compress "movie.mov" out.mp4 --preset 1080p --codec h264 --max-size 25MB
```

## How it works

Compression uses `AVAssetReader` + `AVAssetWriter`:
- A `AVMutableVideoComposition` scales the source (preserving aspect ratio and
  applying the source's rotation) and the chosen codec (`HEVC`/`H.264`) and
  average bitrate are set on the writer input.
- Audio is re-encoded to AAC for broad compatibility.
- When a target size is set, the bitrate is derived from `size ÷ duration` so the
  output aims for that budget.

## Limitations
- **Target size is approximate.** `AVVideoAverageBitRateKey` is a soft hint — the
  encoder has a minimum bitrate per resolution, so a very small target at a high
  resolution (e.g. 25 MB at 1080p on detailed screen recordings) may land larger
  than requested. Lowering the resolution lets small targets be met (a 25 MB cap
  at 480p typically lands near/under 25 MB). This is an AVFoundation constraint,
  not a bug.
- `AVAssetReader`/`Writer` can't decode every container (some `.mkv`/exotic
  codecs); common `.mp4`/`.mov`/`.m4v` work. Unsupported inputs surface a clear error.
- Notarization is out of scope (ad-hoc signed only).
- Ships with the default app icon; drop a `ZephVideoCompresser.icns` into
  `ZephVideoCompresser.app/Contents/Resources/` to customize.

