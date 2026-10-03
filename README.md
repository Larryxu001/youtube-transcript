# YouTube Transcript

Organize a YouTube channel's video list, captions, and basic engagement data into a local HTML archive: play the YouTube video on the left and read the transcript on the right. When YouTube captions are unavailable, use Groq or OpenAI's Whisper service to generate timestamped AI captions automatically, with support for more than 99 languages, including Chinese and English.

This tool does not download video files or retain intermediate audio, SRT, VTT, or TXT files long-term. It saves only local HTML, a small incremental state file, and an optional CSV list.

## Features

- Fetch a YouTube channel's historical video list.
- Prefer manually created captions, then YouTube's automatic captions.
- When YouTube captions are unavailable, automatically extract low-bitrate audio and call Groq or OpenAI Whisper to generate AI captions, with automatic language detection and support for more than 99 languages, including Chinese and English.
- Process long videos with adaptive chunking, parallel recognition, automatic audio-source switching, and resumable progress; delete temporary audio after processing.
- Generate an `index.html` listing for each channel.
- Generate a separate HTML page for each video.
- Click a caption timestamp to jump to that point in the video.
- Generate pages even for videos without captions, with video playback still available on the left.
- Display publication time, view count, like count, and comment count.
- Generate `videos.csv` for easy use in Excel, Numbers, or Google Sheets.
- Update incrementally: automatically skip archived videos.
- Check the YouTube connection before starting a task and distinguish timeouts, DNS, TLS, proxy errors, rate limits, login verification, regional restrictions, private videos, and unavailable videos.
- Retry temporary network errors a limited number of times instead of remaining stuck on the reading status indefinitely.
- Add failed videos to a retry queue and prioritize them the next time the same channel runs, without fetching successful videos again.
- Automatically pause a channel when rate limits, login verification, or repeated network failures are detected, avoiding further request pressure.
- Use macOS/Windows system proxy settings by default, with support for entering a manual HTTP/HTTPS/SOCKS proxy address in the app.
- Provide a native macOS `.app` and a Windows graphical app: enter a channel and video count, then view live progress and left-aligned logs without opening a command window. Tasks support pause, resume, and stop.
- Support Chinese channel names and percent-encoded links copied from a browser, such as `%E6...`.
- Let you choose and remember the archive location; app upgrades do not overwrite existing data.
- Open results through the main app; newer versions no longer generate a `.command`, `.bat`, or small viewer for every channel.
- Accept channel URLs directly, without requiring manual configuration-file edits.

## Installation

On macOS, download the DMG and drag “YouTube Transcript” into Applications. Release builds include a standalone backend, yt-dlp, and FFmpeg; you do not need to install Python or Homebrew or use Terminal.

The DMG uses the traditional Mac installation layout: the app is on the left, and `Applications` on the right is a shortcut to the system Applications folder. Drag the app from left to right to install it. After the first launch from Applications, the app reminds you to eject the installation disk and delete the downloaded DMG; it does not delete user files automatically.

The current free version is not notarized through the Apple Developer program, so Gatekeeper may block its first launch. In Finder, Control-click the app and choose Open. If it is still blocked, go to System Settings → Privacy & Security and select Open Anyway near the bottom. No Terminal commands are needed.

Only developers running the source directly need Python 3, `yt-dlp`, and FFmpeg. The app does not download or save video files. AI captions temporarily process audio only, which is cleaned up automatically after success or failure.

## Quickest start

For a beginner-friendly three-step guide, see [QUICKSTART.md](QUICKSTART.md).

If you prefer not to enter commands:

- macOS: Double-click `YouTube Transcript.app`. Its native interface lets you paste a channel URL, set the number of videos for this run, choose an archive location, and see live progress. You can pause, resume, or stop without opening Terminal.
- Windows: Download `YouTube-Transcript-2.3.1.1-Windows-x64-Setup.exe` from the website, complete the installation wizard, and open “YouTube Transcript” from the Start menu. You do not need to install Python, yt-dlp, or FFmpeg separately.

Paste a YouTube creator's channel URL, such as `https://www.youtube.com/@handle`. On the first run, the default location is the `YouTube 字幕学习档案` folder in Documents. You can also select an existing archive directory in the app. When fetching finishes, the page for the channel just processed opens.

If you enable automatic AI captions when YouTube captions are unavailable, select Groq (the default, optimized for speed) or OpenAI Whisper and paste the corresponding API key. Both support more than 99 languages, including Chinese and English, and detect the language automatically. The Get API Key control opens the provider's official platform. Keys for different services are encrypted and saved separately, and are not written to caption archives or logs. Disable this option if you only want YouTube's existing captions and playback pages for videos without captions.

After pasting an API key, click Save Key; the Saved status confirms success. macOS uses the system Keychain, while Windows uses DPAPI encryption for the current account. If macOS search shows two apps with the same name, the installation disk is usually still mounted. Eject it, delete the downloaded DMG, and remove the old app named `开始抓取 YouTube 字幕` from Applications. This does not affect caption archives.

The Network Proxy field can usually remain empty because the app reads system settings automatically. If your browser can open YouTube but the app cannot read a channel, enter the local proxy address shown by your proxy software, such as `http://127.0.0.1:PORT`. The app does not remember proxies containing a username and password, and logs hide authentication details.

If you have fetched this creator before, enter the same channel URL again to continue. The program skips completed videos and continues with older ones.

A count of `50` or `100` is suitable for everyday use. Enter `0` to attempt all videos not yet archived in this run. For large channels, smaller batches are still recommended to reduce temporary YouTube rate limits.

If you prefer the command line, you can run a single Python file:

```bash
python3 youtube_caption.py --interactive
```

Open the results after fetching:

```bash
python3 youtube_caption.py --open --output archive
```

You can also pass a channel URL directly:

```bash
python3 youtube_caption.py --channel "https://www.youtube.com/@handle" --output archive
```

On Windows, replace `python3` with:

```powershell
py -3 youtube_caption.py --channel "https://www.youtube.com/@handle" --output archive
```

## Build the native macOS app

Regular users do not need to build the app. After changing the interface or backend, developers can run this for everyday debugging:

```bash
./macos/build_macos_app.sh
```

The debug app is generated at `.build/Products.noindex/YouTube Transcript.app`. Spotlight does not list this directory as another installed copy. Regular users should still install from the DMG into Applications.

Generate a universal DMG for website distribution:

```bash
./macos/build_release_macos.sh
```

Release builds support both Apple Silicon and Intel Macs. They include the backend, yt-dlp, and universal LGPL audio components built from a pinned official FFmpeg source release. Artifacts and SHA-256 checksum files are written to `release/`. The build machine needs a one-time setup of project-local PyInstaller and dmgbuild environments and the official `yt-dlp_macos`. If FFmpeg is missing, the script verifies the official source and builds it automatically. Regular users do not need these tools.

To view an archive, open the main app and click Open Results.

## Configure multiple channels

Copy the example configuration:

```bash
cp channels.example.json channels.json
```

Edit `channels.json`:

```json
{
  "channels": [
    {
      "url": "https://www.youtube.com/@handle",
      "batch_size": 50,
      "delay_seconds": 2,
      "timezone": "Asia/Shanghai",
      "languages": ["zh-Hans", "zh-Hant", "zh.*", "en.*", "en"]
    }
  ]
}
```

Then run:

```bash
python3 youtube_caption.py --config channels.json --output archive
```

You can paste a channel homepage URL directly, such as `https://www.youtube.com/@handle`; the program automatically converts it to `/videos`. The `name` field is optional and is only needed to customize the local folder name.

## Common commands

## Build the Windows installer

Regular users do not need to build the app. Developers can install Python 3.11 and Inno Setup 6 on Windows 10/11 x64, then run:

```powershell
.\build_windows_exe.ps1
```

The script creates an isolated environment, downloads the official `yt-dlp.exe`, packages the windowless graphical app and backend separately, and generates a standard installer and SHA-256 checksum:

```text
release\windows\YouTube-Transcript-2.3.1.1-Windows-x64-Setup.exe
release\windows\YouTube-Transcript-2.3.1.1-Windows-x64-Setup.exe.sha256
```

You can also push the code to GitHub, manually run the `Build Windows installer` workflow, and download the same installer from its build artifacts. PyInstaller cannot generate Windows executables directly on macOS, so the final EXE must be built on a Windows machine or Windows GitHub Actions runner.

The installer installs for the current user and does not require administrator privileges. Program files are stored in the user's application directory, and caption archives default to the `YouTube 字幕学习档案` folder in Documents. Uninstalling or upgrading the app does not delete caption archives. The current free version does not have commercial code signing, so Microsoft Defender SmartScreen may show a warning when installing on another computer for the first time.

Test with a small batch:

```bash
python3 youtube_caption.py --channel "https://www.youtube.com/@handle" --output archive --limit 3
```

Process at most 100 new videos per run:

```bash
python3 youtube_caption.py --channel "https://www.youtube.com/@handle" --output archive --limit 100
```

Recheck videos that previously had no captions:

```bash
python3 youtube_caption.py --config channels.json --output archive --retry-missing
```

Backfill only publication times, view counts, like counts, and comment counts for existing videos without adding new ones:

```bash
python3 youtube_caption.py --config channels.json --output archive --limit 0 --metadata-limit 100
```

## Output structure

```text
archive/
├── index.html
└── Channel Name/
    ├── index.html
    ├── videos.csv
    ├── .caption-archive.json
    ├── .caption-retry.json (present only when videos are awaiting retry)
    ├── .ai-checkpoints/ (retained when an AI task is interrupted; cleaned up after success)
    ├── .ai-reports/ (AI processing times and model details; no audio or keys)
    └── 2026-08-21_videoId.html
```

Keep `.caption-archive.json`, which stores incremental state. It is not an intermediate video or caption file.

`.caption-retry.json` stores failed video IDs, error types, and retry counts. It contains no video, audio, captions, or passwords. Run the same channel again after the network recovers to prioritize retries; the file is deleted automatically when all retries succeed.

`.ai-checkpoints` stores only successfully returned recognition results for individual chunks so processing can resume. It does not store audio or API keys. A video's checkpoint is deleted automatically when the entire video finishes.

## What happens during network problems

- YouTube also fails in the browser: the app retries a limited number of times and shows a clear message; existing archives remain unaffected.
- YouTube works in the browser but the app fails: first confirm the system proxy is enabled. If it still fails, enter the local proxy address in Network Proxy.
- YouTube returns 429: the app immediately stops the current channel and saves progress. Waiting 30–60 minutes is recommended.
- YouTube requests sign-in or bot verification: the app does not increase request frequency or attempt to bypass verification.
- One video is private, deleted, or region-restricted: only that video is added to the retry queue, and the others continue.
- Three consecutive videos encounter connection errors: the app pauses the channel instead of repeatedly making requests while offline.

## Why results need the launcher

Double-clicking local HTML opens it with the browser's `file://` protocol, where the embedded YouTube player often fails. Open Results in the main app starts a local-only HTTP service at `127.0.0.1`, allowing the browser to embed the YouTube player normally.

## Rate-limit recommendations

A first run on a large channel may include hundreds of videos. Recommended settings:

- Set `batch_size` or `--limit` to 50–100.
- Keep `delay_seconds` at 2 seconds or higher.
- If YouTube requests sign-in, bot verification, or reports restricted access, pause for a while. Do not increase concurrency or shorten the delay.

## Use with an AI assistant

You can ask Codex, Claude, Cursor, or another assistant to run it like this:

```text
Please use this project to fetch the complete video list and captions from https://www.youtube.com/@handle.
Do not download videos by default. Write to archive, generate HTML pages, channel listings, and videos.csv, and use incremental updates.
```

If your assistant supports skills, you can install `skills/youtube-caption-archive/SKILL.md` in its skills directory.

## License

MIT License. Respect YouTube's terms of service and content creators' rights. This tool only organizes captions and metadata accessible from public pages. It does not bypass payment, login, copyright, or regional restrictions.
