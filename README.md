# pipr

**pipr** is a universal shell tool for piping any video or livestream (YouTube, Twitch, Kick, Odysee, YouNow, etc – see [yt-dlp’s supported sites](https://github.com/yt-dlp/yt-dlp/blob/master/supportedsites.md)) to RTMP endpoints using `yt-dlp` and `ffmpeg`.

## Features

- Supports any site supported by yt-dlp (YouTube, Twitch, Kick, Odysee, YouNow, and hundreds more)
- Auto-detects source domain and logs accordingly
- Extracts source via `yt-dlp` (`--get-url`), with multi-URL handling (separate A/V) and mapping
- Streams to RTMP with `ffmpeg`
- Copy-first strategy with automatic fallback to re-encoding
- Re-encoding controls: encoder, bitrate, preset, audio bitrate/rate, keyframe interval (seconds or frames), GOP, optional FPS alignment
- Input resilience: reconnect flags for HTTP(S) sources
- Tunable buffering: `analyzeduration` and `probesize` (defaults preserved: 20M/100M)
- Robust logging with sanitizer: masks RTMP endpoints and reduces HTTP(S) URLs to `protocol://host/...`
- yt-dlp pass-through flags for cookies/headers/custom format selection

## Requirements

- `yt-dlp`
- `ffmpeg`
- Bash (most Linux/macOS)

## Usage

```bash
# Make it executable (if running from a local checkout):
chmod +x pipr

# Basic usage (copy first, fallback to encode if needed):
pipr "https://www.youtube.com/watch?v=XXXXX" "rtmp://your-endpoint/live/stream_key"

# Verbose mode (mirrors tool output to terminal and log; sanitized):
pipr -v "https://kick.com/<channelname>" "rtmp://your-endpoint/live/stream_key"

# Force re-encoding and tune video/audio:
pipr --force-encoding --encoder libx264 --bitrate 4500k --preset veryfast \
     --audio-bitrate 160k --audio-rate 48k \
     "https://www.twitch.tv/<channelname>" "rtmp://your-endpoint/live/stream_key"

# Set keyframe interval by seconds (common 2s) and align GOP via FPS:
pipr --force-encoding --encoder libx264 --bitrate 6000k --preset fast \
     --keyframe-interval 2s --fps 60 \
     "https://www.example.com/video" "rtmp://your-endpoint/live/stream_key"
# With --fps provided, this sets -g ~120 (2s * 60fps), -r 60, and -vsync cfr for predictable CFR/GOP.

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
  - `-v`: Verbose mode; mirrors yt-dlp/ffmpeg output to terminal and log.
  - `--force-encoding`: Skip stream copy and start with re-encoding.

- Video encoding
  - `--encoder <name>`: Video encoder (default: `libx264`). Examples: `libx264`, `h264_nvenc`, `hevc_videotoolbox`.
  - `--bitrate <vbitrate>`: Video bitrate (e.g., `4500k`). Default: encoder default.
  - `--preset <preset>`: Encoder preset (default: `veryfast`). Examples: `veryfast`, `fast`, `medium`.
  - `--gop <frames>`: GOP size (keyframe interval in frames). Applies `-g <frames>` and `-sc_threshold 0`.
  - `--keyframe-interval <value>`: Keyframe interval in seconds or frames; overrides `--gop`.
    - Examples: `2s`, `2.0s` (seconds), or `120` (frames).
    - Seconds-based: applies `-force_key_frames "expr:gte(t,n_forced*seconds)"` and `-sc_threshold 0`.
    - If `--fps` is also set, applies `-g = round(seconds * fps)` for encoders/targets that prefer matching GOP.
  - `--fps <rate>`: Optional helper for `--keyframe-interval` in seconds to align `-g`. Example: `--keyframe-interval 2s --fps 60` -> `-g 120`.

- Audio encoding
  - `--audio-bitrate <abitrate>`: Audio bitrate (e.g., `128k`, `160k`). Default: encoder default.
  - `--audio-rate <arate>`: Audio sample rate (e.g., `44100`, `48k`). Default: source/default.

- Buffering and probing
  - `--analyzeduration <value>`: ffmpeg analyzeduration (default: `20M`).
  - `--probesize <value>`: ffmpeg probesize (default: `100M`).

- yt-dlp pass-through
  - `--ytdlp-format <fmt>`: Override yt-dlp format selector (default: `bestvideo*+bestaudio/best`).
  - `--ytdlp-arg <arg>`: Pass-through argument to yt-dlp. Repeatable. Examples:
    - `--ytdlp-arg "--cookies cookies.txt"`
    - `--ytdlp-arg "--add-header" --ytdlp-arg "Authorization: Bearer <TOKEN>"`

Notes on precedence
- Copy-first: Without `--force-encoding`, the tool tries `-c copy` first. If copy fails, it re-encodes using your flags.
- If both `--keyframe-interval` and `--gop` are provided, `--keyframe-interval` takes precedence.

## Production defaults

pipr applies production-hardened defaults for reliability and compatibility:

- **Input handling**: `-re` (real-time reading) is applied only to non-HTTP(S) inputs (local files/pipes) to avoid adding latency to live HTTP/HLS streams.
- **Network resilience**: HTTP(S) inputs automatically include `-rw_timeout 15s` (read/write timeout) and `-http_persistent 0` (disable persistent connections) along with reconnect flags for improved stability.
- **Encoding compatibility**: When re-encoding (fallback or `--force-encoding`):
  - `-pix_fmt yuv420p` ensures broad player/decoder compatibility.
  - `-ac 2` (stereo audio) is set by default.
  - When `--bitrate` is specified, `-maxrate` and `-bufsize` are automatically added to stabilize rate control.
- **Predictable GOP/CFR**: When using `--keyframe-interval` in seconds (e.g., `2s`) with `--fps` (e.g., `60`), pipr also sets `-r <fps>` and `-vsync cfr` for constant frame rate and predictable GOP spacing, in addition to computing `-g` (GOP size in frames).
- **Error handling**: The script uses strict mode (`set -euo pipefail -E`) with traps for graceful signal handling (INT/TERM) and error tracing.
- **URL sanitizer robustness**: Fixed to handle trailing punctuation (including `>`, `<`) without shell syntax errors.

These defaults require no CLI changes and are applied automatically based on input type and provided flags.

## Logging and privacy

- Default: Script log lines appear in terminal; raw tool output goes to `pipr.log`.
- Verbose (`-v`): `yt-dlp` and `ffmpeg` output is mirrored to terminal and logged.
- Sanitization: `pipr` masks RTMP endpoints (e.g., `rtmp://server/app/***abcd`) and reduces HTTP(S) URLs to `protocol://host/...` in logged/echoed tool output. Review logs before sharing.

## 💸 Support pipr

This project is free and open source. If you'd like to support pipr and tip the developer, you can donate via:

- Credit Card: [tip.m60.live](https://tip.m60.live)
- Crypto (Monero, Bitcoin, & more): [xmrchat.com/m60](https://xmrchat.com/m60)

Thank you for supporting open source!

## License

MIT License © 2025 [m60](https://github.com/m60irl). See [LICENSE](./LICENSE) for full terms.
