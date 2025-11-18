# pipr

**pipr** is a universal shell tool for piping any video or livestream (YouTube, Twitch, Kick, Odysee, YouNow, etc – see [yt-dlp’s supported sites](https://github.com/yt-dlp/yt-dlp/blob/master/supportedsites.md)) to an RTMP endpoint (e.g., Restream, Twitch, YouTube Live), using `yt-dlp` and `ffmpeg`. It auto-detects the source, extracts the best stream URL, attempts stream copy first, and falls back to re-encoding if needed. All actions and errors are logged in `pipr.log`, with sensitive endpoints sanitized.

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

# Keep buffering defaults explicitly (values are tunable, defaults preserved):
pipr --analyzeduration 20M --probesize 100M \
     "https://www.youtube.com/watch?v=XXXXX" "rtmp://your-endpoint/live/stream_key"
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

Notes on precedence
- Copy-first: Without `--force-encoding`, the tool tries `-c copy` first. If copy fails, it re-encodes using your flags.
- If both `--keyframe-interval` and `--gop` are provided, `--keyframe-interval` takes precedence.

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
