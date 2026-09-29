# Media Analysis on Lumina Canvas

Load this reference during ingest, frame/audio evidence extraction, or batch processing.

## Visual sampling

Default short-form policy:

- sample at 3 frames per second;
- cap at 36 frames;
- keep the first and last meaningful frames;
- when the cap would be exceeded, distribute samples over the full duration instead of truncating the ending;
- record timestamps to at least 0.1 second when the tool exposes them.

When a contact-sheet component exists:

- arrange 4 columns x 3 rows;
- use 360 x 640 thumbnails for vertical sources or an equivalent aspect-preserving fit for other ratios;
- pad rather than crop when crop would remove action or text;
- label each frame with a high-contrast timestamp;
- use at most three sheets for the default 36-frame budget.

When Lumina offers direct video understanding but no frame sampler, pass the video directly and require the analyzer to cite timestamps. Mark `sampling_method: direct_video` in the output.

## Shot-boundary correction

Uniform sampling can miss very short cuts. Add a frame near a detected cut, title card, product reveal, or reversal when doing so stays within the frame budget. Prefer evidence coverage over duplicate near-identical frames.

## Audio and speech

Branch audio analysis from the source in parallel with visual sampling when the tools allow it.

Transcription output should include:

- utterance start/end;
- speaker label or `unknown_speaker`;
- verbatim text, including meaningful fillers;
- confidence when exposed;
- audible emotion, pause, or action marker only when supported by sound or image.

Do not rewrite advertising claims, slang, or grammar during transcription. Normalize them later only in the generated prompt, and preserve a verbatim transcript separately.

If the video has music or sound effects but no speech, return:

```json
{
  "has_speech": false,
  "transcript": "",
  "background_sounds": [],
  "music": "observable description or unknown"
}
```

## Input quality checks

Record these facts before analysis:

- source name and stable canvas node name;
- duration and aspect ratio;
- resolution;
- visible watermark or burned-in captions;
- likely language;
- audio presence;
- obvious corruption, black frames, or missing ending.

Warn when the source is below 720p, heavily compressed, cropped, or dominated by overlays. Do not estimate exact fabric, small text, or identity details that are not resolved in the evidence.

## Batch policy

Process videos independently. For each item:

1. preflight;
2. visual/audio evidence extraction;
3. analysis;
4. prompt package;
5. optional generation;
6. QA and status.

Limit concurrency to the current workspace's stable capacity. If the canvas cannot isolate errors cleanly, run items sequentially. Shared product or person anchors may fan out into multiple video groups only when the user explicitly wants that reuse.
