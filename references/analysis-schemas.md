# Reference Analysis Schemas

Load this reference when converting visual/audio evidence into structured analysis.

## Exact mode

Use for close replication and same-category replacement. Return valid JSON with these fields:

```json
{
  "person": {
    "gender_presentation": "",
    "age_range": "",
    "face": "",
    "skin_tone": "",
    "hair": "",
    "build": "",
    "makeup": ""
  },
  "product_or_clothing": {
    "type": "",
    "color": "",
    "pattern": "",
    "material_look": "",
    "shape_or_neckline": "",
    "sleeve_or_structure": "",
    "length_or_size": "",
    "fit": "",
    "details": "",
    "accessories": ""
  },
  "scene": {
    "location": "",
    "background_objects": "",
    "floor": "",
    "wall": "",
    "lighting_source": "",
    "color_temperature": "",
    "overall_tone": ""
  },
  "actions": [
    {"start": 0.0, "end": 1.5, "subject": "", "action": "", "hand_or_direction": ""}
  ],
  "dialogue": [
    {"start": 0.0, "end": 1.5, "speaker": "", "text": "", "action_marker": ""}
  ],
  "camera": {
    "movement_type": "",
    "orientation": "",
    "timeline": []
  },
  "audio": {
    "has_speech": false,
    "speech_style": "",
    "background_sounds": "",
    "music": "",
    "overall_audio_mix": ""
  },
  "video_type": "",
  "duration_seconds": {"actual": 0, "recommended": 0},
  "people_count": 0,
  "evidence": {
    "sampling_method": "",
    "frame_count": 0,
    "transcript_available": false,
    "uncertainties": []
  }
}
```

Requirements:

- action nodes every 1–2 seconds where meaningful;
- specify left/right, entrance/exit direction, and object interaction when visible;
- align dialogue with the action timeline;
- preserve fillers and spoken brand/product claims in the transcript;
- use `unknown` instead of guessing;
- separate observation from interpretation in uncertainty notes.

## Rewrite mode

Use for extracting viral logic across different subjects or categories. Return:

```json
{
  "hook": "",
  "audience_promise": "",
  "gender_presentation": "",
  "scene": "",
  "product_or_clothing": "",
  "actions": [],
  "dialogue": [],
  "camera": "",
  "dialogue_style": "",
  "viral_structure": [
    {"beat": "hook|proof|reveal|objection|payoff|cta", "start": 0, "end": 0, "function": ""}
  ],
  "video_type": "",
  "has_speech": false,
  "duration_seconds": 0,
  "evidence": {"uncertainties": []}
}
```

## Analyzer instruction

Use this system instruction in the analysis node:

> You are a reference-video reverse-engineering specialist. Extract only details supported by the supplied video frames and transcript. The result will drive a Lumina Canvas workflow. Do not beautify, infer hidden facts, or merge different timestamps. Return schema-valid JSON, preserve timing, and mark uncertainty explicitly.

For rewrite mode, add:

> Separate the transferable engagement mechanism from subject-specific details. Preserve the hook, beat timing, proof order, and payoff logic without copying unsupported claims or protected brand assets.

## Evidence rules

- Burned-in text may be recorded as visible text; do not assume it is true.
- A transcript is evidence of speech, not evidence that a product claim is factual.
- Infer camera movement only from multiple frames or direct video evidence.
- Do not infer identity, ethnicity, medical state, or other sensitive attributes beyond what the task strictly requires.
- When visual and audio evidence conflict, record both and flag the conflict.
