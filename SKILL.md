---
name: media-transcribe
description: Transcribe podcasts, audio files, or videos with local Whisper and produce a faithful original-language transcript plus a Chinese summary. Use for audio/video URLs or local files when the user asks for transcription, subtitles, or a Chinese summary.
---

# Media Transcribe

Deliver two local documents: an original-language transcript and a Chinese summary. Speech recognition runs locally; the agent writes the summary from the verified transcript. This skill does not write to note apps or knowledge bases, publish files, or archive media to external storage.

## Workflow

### 1. Prepare the source

- Accept a podcast/audio URL, video URL, or local media file. Use the user's output directory when specified; otherwise create `media_transcripts/<source-slug>/` in the current workspace.
- Record the title, canonical URL or local filename, creator/speakers when known, language, duration, and processing date. Infer metadata from the source rather than asking for optional details.
- For URLs, use `yt-dlp` or the publisher's accessible media download. Keep signed download URLs and authentication details out of delivered documents and logs. If access requires a login that is unavailable, request an accessible file or link.
- Keep the source media intact. Use `ffprobe` to confirm an audio stream and its duration. Extract an audio working copy with `ffmpeg` if needed.
- Read available subtitles as terminology clues, checking their language, coverage, timing, and technical names. Default to local Whisper transcription; reuse an existing transcript when the user requests it, and identify its provenance.

### 2. Transcribe locally

Read [references/local-whisper.md](references/local-whisper.md) when selecting or setting up a backend. Reuse a working local installation first.

- Prefer Whisper `large-v3` when resources allow. On Apple Silicon, prefer MLX Whisper with `mlx-community/whisper-large-v3-mlx`; use OpenAI Whisper on other supported hardware. If resources require a smaller model, report the actual model used.
- Use the source language and the `transcribe` task. Whisper's `translate` task produces English and is not how to generate the Chinese summary.
- Disable previous-text conditioning by default to reduce repetition loops in long recordings. A short source-supported vocabulary prompt can help with names; do not seed it with guessed sentences.
- Preserve the raw decoding result, including segment timestamps, as local working evidence. Give retries distinct names so a failed run cannot be mistaken for a fresh output.
- Keep transcript wording close to the audio. Correct a word only when listening or reliable source context supports it; mark unresolved terms as `[听不清，HH:MM:SS]` or `[unclear, HH:MM:SS]`. Do not invent speaker identities or claim automatic diarization.
- For long recordings, report phase changes and progress without dumping the transcript. If processing in chunks is necessary, record offsets, restore global timestamps, and reconcile overlap without losing speech at boundaries.

### 3. Verify before summarizing

- Confirm nonempty outputs, sensible segment timestamps, and coverage of the beginning, middle, and end. Compare the last speech timestamp with the media duration; inspect missing intervals rather than treating an outro or silence as lost speech.
- Check adjacent repeated text, implausible compression, timestamp overlaps, negative durations, and long gaps. Listen to suspicious spans and representative samples, including names, numbers, and technical terms. A successful process exit alone does not establish a complete transcript.
- If repetition loops or major omissions appear, rerun the affected audio with adjusted language/prompt or smaller chunks, then verify again. Record unresolved gaps explicitly; never silently present a partial transcript as complete.
- For videos, inspect representative frames/slides when an important claim depends on visuals. If only audio was inspected, label the summary `仅基于音轨` and leave visual-dependent claims unverified.

### 4. Write the two deliverables

**`transcript-original.md`**

- Include source metadata, the actual backend/model and language, provenance, and coverage limitations.
- Keep the original spoken language, readable paragraphs, and segment or topic timestamps. Label speaker changes only when supported. Preserve the full spoken content instead of rewriting it as an article or translating it into Chinese.

**`summary-zh.md`**

- Write coherent Chinese covering the main thesis, supporting reasoning, useful examples, conclusions, and material caveats. Let the source determine the length and structure.
- Include timestamps for key claims so the reader can return to the recording.
- Attribute opinions and predictions to their speakers; preserve uncertainty, numbers, and qualifications. Separate source claims from any agent analysis. Summarize only material actually reviewed.
- For long sources, read the complete transcript in manageable sections, collect timestamped notes, and synthesize across those notes. Do not summarize just the first chunk or a search excerpt.

Keep media, decoding JSON, logs, and any generated subtitle files as working artifacts. Generate user-facing SRT/VTT only when requested. Retain the source files unless the user asks for cleanup.

## Delivery

Return clickable links to the two deliverables and a brief validation note identifying the actual model, coverage, and any transcription or visual limitations. Do not paste a long transcript into chat or add a knowledge-base integration step.
