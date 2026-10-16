# Local Whisper backends

Use an existing working installation where possible. Otherwise create an isolated Python environment in the working directory; avoid modifying system Python. The examples use `uv`, but a standard `python3 -m venv .venv` environment also works.

Install `ffmpeg` through the platform package manager if missing. Install `yt-dlp` only when downloading a remote source requires it. The first model load may download several GB of weights; subsequent inference is local. Do not switch to a hosted transcription service without the user's instruction.

## Inspect media

```bash
ffprobe -v error -show_entries format=duration:stream=codec_type,codec_name -of json "source.mp4"
ffmpeg -i "source.mp4" -vn -ac 1 -ar 16000 "audio.wav"
```

Whisper can read common audio/video formats through FFmpeg, so extraction is optional. Use a new working directory to avoid overwriting files.

## Apple Silicon: MLX Whisper

```bash
uv venv .venv
uv pip install --python .venv/bin/python mlx-whisper
.venv/bin/mlx_whisper "audio.wav" \
  --model mlx-community/whisper-large-v3-mlx \
  --task transcribe \
  --condition-on-previous-text False \
  --output-dir decode-run-01 \
  --output-name transcript \
  --output-format all
```

Add `--language en` or `--language zh` when the language is known; omit it for automatic detection. MLX uses hyphenated flags. Check the installed version's `--help` before adding optional flags.

The MLX CLI may catch a transcription exception, print `Skipping ...`, and still exit successfully. Inspect its logs and verify newly generated, nonempty JSON/text files before treating the run as complete.

## Other hardware: OpenAI Whisper

```bash
uv venv .venv
uv pip install --python .venv/bin/python openai-whisper
.venv/bin/whisper "audio.wav" \
  --model large-v3 \
  --task transcribe \
  --condition_on_previous_text False \
  --output_dir decode-run-01 \
  --output_format all
```

OpenAI Whisper uses underscore-separated flags. Add a known `--language` as above. Use `--fp16 False` for CPU execution when needed. CPU transcription with `large-v3` may be slow; choose a smaller model if resources require it and disclose the change.

## Validate the decoding result

Use JSON segments to inspect start/end times, repeated neighboring text, `compression_ratio` when present, and large gaps. A compression ratio above Whisper's usual decoding threshold of 2.4 is a signal for review, not proof of hallucination. Listen to suspicious spans and distinguish actual repeated speech from decoding loops.

Derive the readable transcript from the verified segments while retaining global timestamps. The agent produces `summary-zh.md` from the entire verified transcript; neither CLI generates the Chinese summary.

## Upstream references

- [MLX Whisper setup and CLI](https://github.com/ml-explore/mlx-examples/tree/main/whisper)
- [MLX Whisper CLI implementation](https://github.com/ml-explore/mlx-examples/blob/main/whisper/mlx_whisper/cli.py)
- [OpenAI Whisper setup and usage](https://github.com/openai/whisper)
