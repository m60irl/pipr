# pipr

**pipr** is a universal shell tool for piping any video or livestream (YouTube, Twitch, Kick, Odysee, YouNow, etc – see [yt-dlp’s supported sites](https://github.com/yt-dlp/yt-dlp/blob/master/supportedsites.md)) to an RTMP endpoint using `yt-dlp` and `ffmpeg`.

It prefers stream copy first for lowest latency/CPU, with an automatic fallback to re-encoding when copy isn’t possible.

## Features

- Supports any site supported by yt-dlp (YouTube, Twitch, Kick, Odysee, YouNow, and hundreds more)
- Auto-detects source domain and logs accordingly
- Extracts source via `yt-dlp` (`--get-url`), with multi-URL handling (separate A/V) and mapping
- Streams to RTMP with `ffmpeg`
- Copy-first strategy with automatic fallback to re-encoding
- Re-encoding controls: encoder, bitrate, preset, audio bitrate/rate, keyframe interval (seconds or frames), GOP, optional FPS alignment
- Input resilience: reconnect flags for HTTP(S) sources with conditional `http_persistent` support (automatically detected)
- Optional real-time pacing for VOD-to-RTMP streaming
- Optional HTTP header propagation from yt-dlp to ffmpeg
- Tunable buffering: `analyzeduration` and `probesize` (defaults preserved: 20M/100M)
- Encoder-specific optimizations for libx264 and NVENC
- Improved rate control for more predictable RTMP output
- Robust logging with sanitizer: masks RTMP endpoints and reduces HTTP(S) URLs to `protocol://host/...`
- yt-dlp pass-through flags for cookies/headers/custom format selection
- Preflight validation: tool version logging and rtmps protocol checking
- Bash 3.2 compatibility (fallback for `mapfile`/`readarray` on older macOS)

## Requirements

- `yt-dlp`
- `ffmpeg`
- Bash 3.2+ (works on Linux/macOS; includes a compatibility fallback for macOS’s default Bash 3.2)
- Optional: `jq` if you use `--propagate-headers`

### Quick install (dependencies)

Examples (adjust to your platform):

- macOS (Homebrew):
  - `brew install yt-dlp ffmpeg jq`
- Debian/Ubuntu:
  - `sudo apt-get update && sudo apt-get install -y yt-dlp ffmpeg jq`
- Arch:
  - `sudo pacman -S yt-dlp ffmpeg jq`

## Installation

- Clone or download this repo
- Make the script executable:
  ```bash
  chmod +x pipr
  ```
- Run directly from the repo root or place it on your `$PATH` (e.g., `/usr/local/bin`)

## Usage

```bash
# Basic usage (copy first, fallback to encode if needed):
pipr "https://www.youtube.com/watch?v=XXXXX" "rtmp://your-endpoint/live/stream_key"

# Verbose mode (shows ffmpeg progress in terminal; sanitized):
pipr -v "https://kick.com/<channelname>" "rtmp://your-endpoint/live/stream_key"

# Force re-encoding and tune video/audio:
pipr --force-encoding --encoder libx264 --bitrate 4500k --preset veryfast \
     --audio-bitrate 160k --audio-rate 48k \
     "https://www.twitch.tv/<channelname>" "rtmp://your-endpoint/live/stream_key"

# Set keyframe interval by seconds (common 2s) and align GOP via FPS:
pipr --force-encoding --encoder libx264 --bitrate 6000k --preset fast \
     --keyframe-interval 2s --fps 60 \
     "https://www.example.com/video" "rtmp://your-endpoint/live/stream_key"

# Stream a pre-recorded video with real-time pacing (VOD to RTMP):
pipr --realtime --force-encoding --encoder libx264 --bitrate 4500k \
     "https://www.example.com/vod.mp4" "rtmp://your-endpoint/live/stream_key"

# Propagate yt-dlp HTTP headers to ffmpeg (requires jq):
pipr --propagate-headers --force-encoding --encoder libx264 --bitrate 4500k \
     "https://www.example.com/video" "rtmp://your-endpoint/live/stream_key"

# Enable file logging with custom location:
pipr --log-file /tmp/my-stream.log \
     "https://www.youtube.com/watch?v=XXXXX" "rtmp://your-endpoint/live/stream_key"

# Enable file logging with default location (pipr.log):
pipr --log-file \
     "https://www.youtube.com/watch?v=XXXXX" "rtmp://your-endpoint/live/stream_key"

# Keep buffering defaults explicitly (values are tunable, defaults preserved):
pipr --analyzeduration 20M --probesize 100M \
     "https://www.youtube.com/watch?v=XXXXX" "rtmp://your-endpoint/live/stream_key"

# yt-dlp: pass cookies file (repeatable --ytdlp-arg)
pipr --ytdlp-arg "--cookies cookies.txt" \
     "https://www.example.com/live" "rtmp://your-endpoint/live/stream_key"

# yt-dlp: add custom headers (repeatable)
pipr --ytdlp-arg "--add-header" --ytdlp-arg "Authorization: Bearer <TOKEN>" \
     "https://www.example.com/live" "rtmp://your-endpoint/live/stream_key"

# yt-dlp: override format selection
# Prefer H.264/AAC where possible for higher chance of copy-first success
pipr --ytdlp-format "bv*[vcodec*=avc1]+ba[acodec*=mp4a]/b[ext=mp4]" \
     "https://www.example.com/v" "rtmp://your-endpoint/live/stream_key"
```

## Flags

- General
  - `-v`: Verbose mode; shows ffmpeg progress and mirrors yt-dlp/ffmpeg output to terminal (and log if file logging is enabled).
  - `--force-encoding`: Skip stream copy and start with re-encoding.
  - `--realtime`: Apply real-time pacing (`-re`) to HTTP/HTTPS inputs for VOD-to-RTMP streaming.
  - `--propagate-headers`: Fetch and forward yt-dlp HTTP headers to ffmpeg. Requires `jq`. If `jq` is not available, a warning is logged and execution continues without header propagation.

- Video encoding
  - `--encoder <name>`: Video encoder (default: `libx264`). Examples: `libx264`, `h264_nvenc`, `hevc_videotoolbox`.
  - `--bitrate <vbitrate>`: Video bitrate (e.g., `4500k`, `5M`). When specified, rate control is tuned with `maxrate = 1.07 * bitrate` and `bufsize = 2 * bitrate`.
  - `--preset <preset>`: Encoder preset (default: `veryfast`). Examples: `veryfast`, `fast`, `medium`.
  - `--gop <frames>`: GOP size (keyframe interval in frames). Applies `-g <frames>` and `-sc_threshold 0`.
  - `--keyframe-interval <value>`: Keyframe interval in seconds or frames; overrides `--gop`.
    - Examples: `2s`, `2.0s` (seconds), or `120` (frames).
    - Seconds-based: applies `-force_key_frames "expr:gte(t,n_forced*seconds)"` and `-sc_threshold 0`.
    - If `--fps` is also set, applies `-g = round(seconds * fps)` for targets that prefer matching GOP.
  - `--fps <rate>`: Optional helper for `--keyframe-interval` in seconds to align `-g`. Example: `--keyframe-interval 2s --fps 60` -> `-g 120`.

- Audio encoding
  - `--audio-bitrate <abitrate>`: Audio bitrate (e.g., `128k`, `160k`).
  - `--audio-rate <arate>`: Audio sample rate (e.g., `44100`, `48k`).

- Buffering and probing
  - `--analyzeduration <value>`: ffmpeg analyzeduration (default: `20M`).
  - `--probesize <value>`: ffmpeg probesize (default: `100M`).

- Logging
  - `--log-file [<path>]`: Enable file logging. Optionally specify a log file path (default: `pipr.log` when enabled). File logging is OFF by default.
  - `--no-log-file`: Explicitly disable file logging (default behavior). Terminal output still shows high-level pipr logs and (in `-v` mode) sanitized tool output.

- yt-dlp pass-through
  - `--ytdlp-format <fmt>`: Override yt-dlp format selector (default: `bestvideo*+bestaudio/best`).
  - `--ytdlp-arg <arg>`: Pass-through argument to yt-dlp. Repeatable.
    - Examples:
      - `--ytdlp-arg "--cookies cookies.txt"`
      - `--ytdlp-arg "--add-header" --ytdlp-arg "Authorization: Bearer <TOKEN>"`

Notes on precedence
- Copy-first: Without `--force-encoding`, the tool tries `-c copy` first. If copy fails, it re-encodes using your flags.
- If both `--keyframe-interval` and `--gop` are provided, `--keyframe-interval` takes precedence.

## Production defaults

To enhance resilience and compatibility for production streaming, pipr applies several defaults automatically:

- Conditional HTTP flag support
  - The `http_persistent` flag is only applied when supported by your ffmpeg build (detected via `ffmpeg -h protocol=http`). This prevents “Option http_persistent not found” failures.

- Conditional throttling
  - The `-re` flag (real-time playback) is applied only to non-HTTP inputs (files, local streams) to throttle reading. For HTTP/HTTPS inputs, `-re` is not used by default.

- Network resilience for HTTP(S) inputs
  - `-rw_timeout 15000000` (15 seconds in microseconds) to prevent infinite stalls on network issues
  - `-http_persistent 0` (when supported) to work better with flaky CDNs that don’t handle persistent connections well
  - Standard reconnect flags: `-reconnect 1 -reconnect_streamed 1 -reconnect_delay_max 5`
  - `-thread_queue_size 1024` before each input to reduce “Too many packets buffered” errors with HLS/DASH

- Stream mapping defaults
  - For single-input sources, pipr explicitly maps video and audio streams (`-map 0:v:0 -map 0:a:0?`) with optional audio handling.
  - For multi-URL cases (separate video/audio), mapping is handled to maintain A/V sync where possible.

- RTMP/FLV compatibility defaults
  - When re-encoding (either forced or as fallback), pipr automatically adds:
    - `-pix_fmt yuv420p` for broad decoder compatibility
    - `-ac 2` for stereo audio output
    - Tuned rate control when `--bitrate` is set: `-maxrate 1.07 * bitrate` and `-bufsize 2 * bitrate`

- Encoder-specific optimizations
  - libx264: `-profile:v high -tune zerolatency` by default. When GOP is set (via `--gop` or `--keyframe-interval`), also adds `-x264-params scenecut=0`.
  - NVENC (e.g., `h264_nvenc`, `hevc_nvenc`): adds `-rc cbr` and `-forced-idr 1` for consistent rate control and IDR enforcement.

- Keyframe and framerate alignment
  - When using `--keyframe-interval` with seconds (e.g., `2s`) AND `--fps` is provided:
    - Output framerate is set with `-r <fps>`
    - Constant framerate mode is enforced with `-vsync cfr`
    - GOP size `-g` is computed as `round(seconds * fps)` for predictable keyframe spacing
    - If `awk` is not available and both seconds and fps are integers, shell arithmetic is used as fallback

- Preflight validation
  - At startup, pipr logs tool versions (`yt-dlp --version` and `ffmpeg -version`).
  - If the RTMP endpoint uses `rtmps://`, pipr verifies that ffmpeg supports the rtmps protocol before starting.

These defaults require no additional flags and do not change the existing CLI behavior.

## Compatibility notes

- Bash 3.2 (macOS): pipr includes a fallback for `mapfile`/`readarray` so it works with the default macOS Bash 3.2 without requiring manual upgrades.
- ffmpeg builds vary:
  - If your build doesn’t support `http_persistent`, pipr will detect and avoid using it.
  - rtmps requires an ffmpeg build with TLS/rtmps support; pipr preflights this and will exit with a clear error if unsupported.

## VOD (Video-On-Demand) usage notes

When streaming pre-recorded videos (VOD) to RTMP:

- Codec compatibility: Stream copy (`-c copy`) often fails for VOD content with VP9/Opus or other non-RTMP-compatible codecs. For better success with copy mode:
  - Use `--ytdlp-format` to prefer H.264/AAC formats:
    ```bash
    pipr --ytdlp-format "bv*[vcodec*=avc1]+ba[acodec*=mp4a]/b[ext=mp4]" ...
    ```
  - Or use `--force-encoding` to re-encode to H.264/AAC:
    ```bash
    pipr --force-encoding --encoder libx264 --bitrate 4500k ...
    ```

- Real-time pacing: By default, HTTP/HTTPS inputs are not throttled with `-re` to minimize latency for live streams. When streaming VOD content, you may want to add `--realtime` to pace the stream:
    ```bash
    pipr --realtime --force-encoding --encoder libx264 --bitrate 4500k \
         "https://www.youtube.com/watch?v=XXXXX" "rtmp://your-endpoint/live/stream_key"
    ```
    Without `--realtime`, pipr will push frames as fast as the network and encoder allow, which may cause buffering issues on some RTMP servers.

- Header propagation: Some CDNs require specific HTTP headers for VOD content. Use `--propagate-headers` (requires `jq`) to automatically forward yt-dlp-derived headers to ffmpeg:
    ```bash
    pipr --propagate-headers --force-encoding --encoder libx264 \
         "https://www.example.com/video" "rtmp://your-endpoint/live/stream_key"
    ```

## Troubleshooting

- “Too many packets buffered”
  - pipr adds `-thread_queue_size 1024` before inputs; if you still see this, consider lowering input bitrate or network jitter, or forcing re-encode with a reasonable `--bitrate`.

- Copy mode fails or no video on RTMP
  - Force encode: `--force-encoding --encoder libx264 --bitrate 4500k`
  - Prefer H.264/AAC via yt-dlp: `--ytdlp-format "bv*[vcodec*=avc1]+ba[acodec*=mp4a]/b[ext=mp4]`

- rtmps not supported
  - Use an ffmpeg build with TLS/rtmps support or switch to `rtmp://`. pipr checks support up-front and will explain the failure.

- Keyframe interval requirements (e.g., some platforms require 2s)
  - Use seconds-based keyframes plus fps alignment:
    ```bash
    pipr --force-encoding --keyframe-interval 2s --fps 60 ...
    ```

## Logging and privacy

- **File logging is OFF by default.** Use `--log-file` to enable logging to a file.
- **Default (no `-v`)**: pipr runs quietly with minimal output. High-level pipr log lines appear in terminal; ffmpeg shows errors only.
- **Verbose (`-v`)**: ffmpeg progress stats are shown, and `yt-dlp`/`ffmpeg` output is mirrored to terminal (and to file if `--log-file` is used).
- **Sanitization**: `pipr` masks RTMP endpoints (e.g., `rtmp://server/app/***abcd`) and reduces HTTP(S) URLs to `protocol://host/...` in logged/echoed tool output. Review logs before sharing.

## 💸 Support pipr

This project is free and open source. If you'd like to support pipr and tip the developer, you can donate via:

- Credit Card: [tip.m60.live](https://tip.m60.live)
- Crypto (Monero, Bitcoin, & more): [xmrchat.com/m60](https://xmrchat.com/m60)

Thank you for supporting open source!

## License

MIT License © 2025 [m60](https://github.com/m60irl). See [LICENSE](./LICENSE) for full terms.