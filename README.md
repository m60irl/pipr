# pipr

**pipr** is a universal shell tool for piping any video or livestream (YouTube, Twitch, Kick, Odysee, YouNow, etc – see [yt-dlp’s supported sites](https://github.com/yt-dlp/yt-dlp/blob/master/supportedsites.md)) to an RTMP endpoint (e.g., Restream, Twitch, YouTube Live, etc.), using `yt-dlp` and `ffmpeg`.  
It auto-detects the source domain, extracts the best stream URL, and streams with error handling and automatic fallback to re-encoding if needed. All actions and errors are logged in `pipr.log`.

## Features

- Supports any site supported by yt-dlp (YouTube, Twitch, Kick, Odysee, YouNow, and [hundreds more](https://github.com/yt-dlp/yt-dlp/blob/master/supportedsites.md))
- Auto-detects source domain and logs accordingly
- Gets stream source via `yt-dlp` (`--get-url`)
- Streams to RTMP endpoint with `ffmpeg`
- Tries codec copy first, then automatically falls back to re-encoding for compatibility
- Robust logging to both terminal and `pipr.log`
- Error handling for missing dependencies, invalid URLs, and streaming failures
- Verbose mode (`-v` flag) streams detailed `yt-dlp` and `ffmpeg` output to your terminal

## Requirements

- `yt-dlp`
- `ffmpeg`
- Bash (compatible with most Linux/macOS systems)

## Usage

```bash
# Make it executable (if running from a local checkout):
chmod +x pipr

# Usage for any video/livestream URL from a supported site:
pipr "https://www.youtube.com/watch?v=XXXXX" "rtmp://your-endpoint/live/stream_key"
pipr "https://www.twitch.tv/<channelname>" "rtmp://your-endpoint/live/stream_key"
pipr "https://kick.com/<channelname>" "rtmp://your-endpoint/live/stream_key"
pipr -v "https://www.example.com/video" "rtmp://your-endpoint/live/stream_key"
```

## 💸 Support pipr

This project is free and open source.  
If you'd like to support pipr and tip the developer, you can donate via:

- Credit Card: [tip.m60.live](https://tip.m60.live)
- Crypto (Monero, Bitcoin, & more): [xmrchat.com/m60](https://xmrchat.com/m60)

Thank you for supporting open source!

## Logging

- Default mode: Only script log messages appear in your terminal. Raw `yt-dlp` and `ffmpeg` output is written to the log file (`pipr.log`) for debugging.
- Verbose mode (`-v`): All `yt-dlp` and `ffmpeg` output is streamed to terminal (in addition to logging).
- Privacy: pipr does not log the full media URL or full RTMP endpoint.  
  - The media source is logged as its host only (e.g., `example.com`).  
  - The RTMP endpoint is masked in logs (e.g., `rtmp://server/app/***abcd`).
- All major actions and errors are printed to your terminal and saved in `pipr.log`.

## Notes

- If your RTMP endpoint doesn't accept the source video/audio codecs, pipr will automatically fall back to re-encoding.
- If you want to change the log filename, adjust the `LOGFILE` variable in the script.

## License

MIT License © 2025 [m60](https://github.com/m60irl).  
See [LICENSE](./LICENSE) for full terms.

---

**pipr**: The easiest way to pipe ANY supported livestreams or videos anywhere!