# pipr

**pipr** is a universal shell tool for piping any video or livestream (YouTube, Twitch, Kick, Odysee, YouNow, etc – see [yt-dlp’s supported sites](https://github.com/yt-dlp/yt-dlp/blob/master/supportedsites.md)) to an RTMP endpoint using `yt-dlp` and `ffmpeg`.  
It auto-detects the source domain, extracts the best stream URL, and streams with error handling and automatic fallback to re-encoding if needed. All actions and errors are logged in `pipr.log`.

## Features

- **Supports any site supported by yt-dlp!** (YouTube, Twitch, Kick, Odysee, YouNow, and [hundreds more](https://github.com/yt-dlp/yt-dlp/blob/master/supportedsites.md))
- **Auto-detects source domain** and logs accordingly
- **Gets stream source** via `yt-dlp` (`--get-url`)
- **Streams to RTMP endpoint** with `ffmpeg`
- **Tries codec copy first**, then automatically falls back to re-encoding for compatibility
- **Robust logging** to both terminal and `pipr.log`
- **Error handling** for missing dependencies, invalid URLs, and streaming failures
- **Verbose mode** (`-v` flag) streams detailed yt-dlp and ffmpeg output to your terminal

## Requirements

- `yt-dlp`
- `ffmpeg`
- Bash (compatible with most Linux/macOS systems)

## Usage

```bash
# Make it executable:
chmod +x pipr

# Usage for any video/livestream URL from a supported site:
./pipr "https://www.youtube.com/watch?v=XXXXX" "rtmp://your-endpoint/live/stream_key"
./pipr "https://www.twitch.tv/<channelname>" "rtmp://your-endpoint/live/stream_key"
./pipr "https://kick.com/<channelname>" "rtmp://your-endpoint/live/stream_key"
./pipr -v "https://www.example.com/video" "rtmp://your-endpoint/live/stream_key"
```

## 💸 Support pipr

This project is free and open source.  
If you'd like to support pipr and tip the developer, you can donate via:

- **Credit Card:** [https://tip.m60.live](https://tip.m60.live)
- **Crypto (Monero, Bitcoin, & more):** [https://xmrchat.com/m60](https://xmrchat.com/m60)

Thank you for supporting open source!

## Logging

- **Default mode:** Only script log messages appear in your terminal.  
  All raw yt-dlp and ffmpeg output goes to the log file (`pipr.log`).
- **Verbose mode (`-v`):** All yt-dlp and ffmpeg output is streamed to terminal (in addition to logging).
- All major actions and errors are printed to your terminal *and* saved in `pipr.log`.

## Notes

- By default, only script log messages are shown in your terminal; all tool output goes to the log file.
- In verbose mode, you see the live output from both yt-dlp and ffmpeg.
- If your RTMP endpoint doesn't accept the source video/audio codecs, pipr will automatically fall back to re-encoding.
- If you want to change the log filename, adjust the `LOGFILE` variable in the script.

## License

MIT License © 2025 [m60](https://github.com/m60irl).  
See [LICENSE](./LICENSE) for full terms.

---

**pipr**: The easiest way to pipe ANY supported livestreams or videos anywhere!
