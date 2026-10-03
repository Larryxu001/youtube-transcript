# Cloud Caption MVP

This directory is only for comparing the Groq and VideoCaptioner transcription workflows. It does not modify the production `archive/` or save temporary audio in the caption archive.

## Test video

The recommended default is a video in the current archive that has been confirmed to have no YouTube captions:

```text
https://www.youtube.com/watch?v=jpoMabs9t4s
```

## Step 1: Run the lightweight Groq workflow

For the macOS graphical MVP, double-click `运行 Groq MVP.app`, paste the complete key, and click Start MVP Test. You can select the option to show the input for verification. The key remains only in the subprocess's memory for that run; it is not saved to the Keychain or a file.

You can also set an API key temporarily in your local terminal. Do not put keys in scripts, screenshots, GitHub, or chat histories:

```bash
export GROQ_API_KEY='Your Groq API Key'
cd "/Users/larryxu/Documents/Youtube Caption"
python3 mvp/cloud_asr_mvp.py \
  --video-url 'https://www.youtube.com/watch?v=jpoMabs9t4s'
```

Output is written to `mvp/results/<video-ID>/<run-time>/`:

- `groq_raw_chunks.json`: raw Groq responses for auditing;
- `groq_segments.json`: normalized timestamps;
- `groq_transcript.srt`;
- `groq_transcript.vtt`;
- `groq_transcript.txt`;
- `groq_report.json`: duration, speed, chunk count, and estimated cost.

Default behavior:

- Fetch only YouTube's audio stream, without downloading video;
- Convert to 16 kHz, mono, 48 kbps MP3;
- Automatically split audio longer than 10 minutes, with 10 seconds of overlap between adjacent chunks;
- Use `whisper-large-v3-turbo`;
- Delete temporary audio after success, failure, or cancellation.

Add `--keep-audio` only when VideoCaptioner needs exactly the same input audio for a comparison. Delete `comparison_audio.mp3` after the comparison.

## Step 2: Compare with VideoCaptioner

VideoCaptioner is pinned to the upstream version examined in this review:

```text
commit 95842ecb5618c0b6a548a336bdfb0eb859bdb501
```

The comparison requires a separate isolated Python 3.10–3.12 environment and many dependencies. Complete the lightweight Groq workflow first to confirm that the API, audio, and timestamps work, then install and run VideoCaptioner. This avoids downloading unrelated large dependencies before the key has been verified.

Both workflows must use the same `comparison_audio.mp3`, Groq model, language, and prompt for the comparison.

## Offline tests

```bash
cd "/Users/larryxu/Documents/Youtube Caption/mvp"
python3 -m unittest -v test_cloud_asr_mvp.py
```

## Next-round end-to-end speed experiment

`next_round_mvp.py` tests two independent paths in sequence without modifying the production caption archive:

1. By default, use the `yt-dlp → ffmpeg → Groq` pipeline. The first chunk adapts between 2 and 5 minutes based on measured audio speed, subsequent chunks are 10 minutes each, and concurrency is limited to two;
2. Keep `--try-direct-url` as a diagnostic option. In testing, Groq received a 302 when reading YouTube's temporary URLs, so the production path does not spend time trying it;
3. Automatically speed-test up to four low-bitrate audio-only sources, falling back in order if the preferred source fails;
4. Atomically save a text checkpoint immediately after each successful Groq chunk. After a pause, unexpected exit, or network failure, the next run does not call Groq again for successful chunks. Delete checkpoints automatically when the entire video finishes.

It requests only segment timestamps and records both time to first captions and total end-to-end time:

```bash
cd "/Users/larryxu/Documents/Youtube Caption"
python3 mvp/next_round_mvp.py
```

The API key is read from the environment variable first; if none is set, it is read from the previously described macOS Keychain item. Temporary audio chunks are deleted as soon as Groq confirms completion. Temporary YouTube audio URLs are not written to the results.

Recorded live verification:

- Automatic source selection chose `249-drc` at about 50.6 kbps. For a 14-minute, 16-second video, the first captions appeared in 5.8 seconds and the full result finished in 8.1 seconds;
- On checkpoint resume, the first chunk explicitly skipped Groq, and only the remaining second chunk was recognized;
- After deliberately selecting a nonexistent audio format, the program automatically switched to an available source and completed;
- Checkpoints and temporary audio were both deleted automatically after the successful test.
