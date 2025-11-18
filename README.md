# pipr

**pipr** is a universal shell tool for piping any video or livestream (YouTube, Twitch, Kick, Odysee, YouNow, etc – see [yt-dlp’s supported sites](https://github.com/yt-dlp/yt-dlp/blob/master/supportedsites.md)) to an RTMP endpoint (e.g., Restream, Twitch, YouTube Live, etc.), using `yt-dlp` and `ffmpeg`. It auto-detects the source domain, extracts the best stream URL, and streams with error handling and automatic fallback to re-encoding if needed. All actions and errors are logged in `pipr.log`.

## Features

- Supports any site supported by yt-dlp (YouTube, Twitch, Kick, Odysee, YouNow, and [hundreds more](https://github.com/yt-dlp/yt-dlp/blob/master/supportedsites.md))
- Auto-detects source domain and logs accordingly
- Gets stream source via `yt-dlp` (`--get-url`)
- Streams to RTMP endpoint with `ffmpeg`
- Tries codec copy first, then automatically falls back to re-encoding for compatibility
- Force re-encoding option and configurable encoding settings (`--encoder`, `--bitrate`, `--preset`)
- Robust logging to both terminal and `pipr.log`
- Error handling for missing dependencies, invalid URLs, and streaming failures
- Verbose mode (`-v` flag) streams detailed `yt-dlp` and `ffmpeg` output to your terminal and logs it as well

## Requirements

- `yt-dlp`
- `ffmpeg`
- Bash (compatible with most Linux/macOS systems)

## Usage

```bash
# Make it executable (if running from a local checkout):
chmod +x pipr

# Basic usage:
pipr "https://www.youtube.com/watch?v=XXXXX" "rtmp://your-endpoint/live/stream_key"

# Verbose mode:
pipr -v "https://kick.com/<channelname>" "rtmp://your-endpoint/live/stream_key"

# Force re-encoding (skip the stream copy attempt) and set encoder/bitrate/preset:
pipr --force-encoding --encoder libx264 --bitrate 4500k --preset veryfast \
     "https://www.twitch.tv/<channelname>" "rtmp://your-endpoint/live/stream_key"
```

### Encoding flags
- `--force-encoding`: Skip stream copy and start with re-encoding (useful for strict RTMP targets).
- `--encoder <name>`: Video encoder to use when re-encoding (e.g., `libx264`, `h264_nvenc`, `hevc_videotoolbox`). Default: `libx264`.
- `--bitrate <vbitrate>`: Video bitrate (e.g., `4500k`). Default: encoder default.
- `--preset <preset>`: Encoder preset (e.g., `veryfast`, `fast`, `medium`). Default: `veryfast`.

## 💸 Support pipr

This project is free and open source. If you'd like to support pipr and tip the developer, you can donate via:

- Credit Card: [tip.m60.live](https://tip.m60.live)
- Crypto (Monero, Bitcoin, & more): [xmrchat.com/m60](https://xmrchat.com/m60)

Thank you for supporting open source!

## Logging

- Default mode: Only script log messages appear in your terminal. Raw `yt-dlp` and `ffmpeg` output is written to the log file (`pipr.log`) for debugging.
- Verbose mode (`-v`): All `yt-dlp` and `ffmpeg` output is streamed to terminal and also appended to `pipr.log`.
- Privacy: pipr does not log the full media URL or full RTMP endpoint in its own messages.
  - The media source is logged as its host only (e.g., `example.com`).
  - The RTMP endpoint is masked in logs (e.g., `rtmp://server/app/***abcd`).
  - Note: ffmpeg’s raw output may still print full endpoints; consider sanitizing logs if sharing.
- All major actions and errors are printed to your terminal and saved in `pipr.log`.

## Notes

- If your RTMP endpoint doesn't accept the source video/audio codecs, pipr will automatically fall back to re-encoding (or you can force it via `--force-encoding`).
- If you want to change the log filename, adjust the `LOGFILE` variable in the script.

## License

MIT License © 2025 [m60](https://github.com/m60irl). See [LICENSE](./LICENSE) for full terms.

---

**pipr**: The easiest way to pipe ANY supported livestreams or videos anywhere!