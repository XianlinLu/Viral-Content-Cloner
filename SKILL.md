---
name: lumina-viral-content-cloner
description: Reverse-engineer one or more reference videos inside Lumina Canvas, build an auditable node graph, and optionally generate a faithful or adapted remake with face, body, product, or full material replacement. Use for requests such as “复刻这个视频”, “分析爆款视频”, “拆解参考片”, “批量复刻”, or “build this reference as a Lumina workflow”. Do not use for unrelated editing or from-scratch ideation without a reference video.
license: MIT
metadata:
  version: "1.1.0"
  compatibility: "Lumina Canvas Agent"
---

# Lumina Viral Content Cloner

Reverse-engineer reference videos and turn the findings into an executable, traceable Lumina Canvas workflow. Preserve the reference video's observable structure; never invent unseen facts or claim that a node ran when it did not.

## Scope

Use this skill when the user supplies at least one reference video or an accessible video URL and wants analysis, a reusable canvas workflow, or a generated remake.

Do not activate for:

- video creation with no reference; use a from-scratch creative workflow;
- simple trimming, resizing, captioning, or format conversion;
- requests that only ask for a written review with no replication intent.

## Language Routing

Maintain `interaction_language` and `content_language` separately.

- Resolve `interaction_language` from the user's newest direct instruction. Ignore language found only in attachments, quoted source material, code, metadata, proper names, or tool output. If the message mixes languages, obey an explicit language request; otherwise use the language carrying the latest substantive request; otherwise retain the established conversation language.
- Use `interaction_language` for every visible question, option, heading, recommendation, node-status note, confirmation request, error, QA report, and delivery summary.
- Resolve `content_language` from an explicit request first; otherwise retain the language of source dialogue in `exact`; otherwise use `interaction_language`. Dialogue, narration, subtitles, and character speech use `content_language`.
- When a connected component requires a different prompt language, translate only the hidden tool parameter. Do not switch the visible conversation language.

## Lumina Runtime Contract

Lumina Agent can call only Components connected to it as Tools. Before promising execution, inventory the connected tools and map them to these capabilities:

1. load or inspect video;
2. sample frames or understand video visually;
3. extract/transcribe speech when present;
4. generate or edit images;
5. generate video clips;
6. compose/edit audio and video;
7. preview, save, or export results.

Read [references/tool-contracts.md](references/tool-contracts.md) when configuring or diagnosing the Agent's tools. If a required capability is absent, either build the remaining graph and label the missing edge, or stop at the last honest deliverable. Never simulate a successful tool call.

## Entry Clarification

Infer answers already supplied. Ask only for missing choices that materially change the graph.

1. **Input count:** single or batch.
2. **Execution mode:**
   - `analyze`: return structured analysis, transcript, and prompt package;
   - `build`: create and wire the reusable Canvas workflow without running costly generation;
   - `auto`: run analysis, generation, QA, and export.
3. **Replication strategy:**
   - `exact`: preserve observable timing, composition, action, dialogue, and sound;
   - `rewrite`: preserve hook, pacing, and viral logic while allowing a new narrative or product treatment.
4. **Replacement materials:** none, person reference, garment/product reference, or both.
5. **Output:** aspect ratio, target duration, language/subtitles, and resolution when the user specified them.

If the user says “分析/拆解”, default to `analyze`. If they say “搭建画布/工作流”, default to `build`. If they explicitly ask to “生成/复刻/出片”, default to `auto`.

## Mandatory Recommendations

Before execution, surface only recommendations relevant to the current input:

- Prefer 720p or higher reference video. Low-resolution sources lose texture, text, and fine motion.
- Use the strongest connected vision-capable model for `exact`; use a faster text/vision model for `rewrite` when cost or latency matters.
- Keep speech transcription enabled when dialogue drives the hook. A visual-only remake must be labeled as missing dialogue evidence.
- For product replacement, provide a clean, well-lit reference showing the full product. For identity replacement, provide a consented front or three-quarter portrait.
- In `auto`, generation can consume credits. Run only the branches the user requested and reuse approved anchors instead of regenerating every node.

## Replacement Modes

| Mode | Inputs | Prompt references | Replace | Preserve |
|---|---|---|---|---|
| `clone` | reference video only | no material `@` reference in generation prompt | nothing | person, wardrobe/product, scene, actions, dialogue, camera |
| `person_swap` | person reference | `@person_ref` | face/body identity | wardrobe/product, scene, actions, dialogue, camera |
| `product_swap` | product or garment reference | `@product_ref` | wardrobe/product | person, scene, actions, dialogue, camera |
| `full_swap` | product + person references | `@product_ref`, `@person_ref` | person and wardrobe/product | scene, actions, dialogue, camera |

Resolve automatically:

```text
person_ref + product_ref -> full_swap
product_ref only          -> product_swap
person_ref only           -> person_swap
neither                   -> clone
```

Use node names, not ordinal assumptions such as “image 1”, whenever Canvas exposes stable names. Do not attach unused references.

## Core Workflow

### 0. Preflight

- Confirm the source is readable and within Lumina's current upload/runtime limits.
- Record duration, resolution, aspect ratio, frame rate when exposed, audio-track presence, and input count.
- Inventory connected tools and select `analyze`, `build`, or `auto`.
- Create a visible status ledger: `queued -> analyzing -> ready -> generating -> validating -> completed/failed`.

### 1. Ingest and branch

Place each source video in its own labeled group. Branch it into visual sampling and audio transcription so independent work can run concurrently when the connected tools support it.

Read [references/media-analysis.md](references/media-analysis.md) for frame selection, contact sheets, audio handling, and batch behavior.

### 2. Evidence extraction

- Sample at 3 fps, capped at 36 representative frames for ordinary short-form inputs.
- Arrange up to 12 frames per 4x3 timestamped contact sheet when a contact-sheet component exists.
- Preserve exact timestamps and source ordering.
- Extract speech with word or utterance timestamps. Preserve fillers, pauses, emphasis, and speaker changes when observable.
- Treat music-only or silent audio as normal, not an error.

### 3. Structured analysis

Analyze frames together with the transcript. Use `exact` for nested production details and `rewrite` for flattened viral logic.

Read [references/analysis-schemas.md](references/analysis-schemas.md) and return valid JSON plus a readable summary. Describe only what the evidence supports. Mark uncertain fields as `unknown` with a short reason.

### 4. Canvas graph assembly

Build a left-to-right graph with visible groups:

```text
INPUTS
  -> VISUAL EVIDENCE ----\
  -> AUDIO EVIDENCE ------> ANALYSIS -> PROMPT PACKAGE -> GENERATION -> QA -> EXPORT
  -> MATERIAL REFERENCES-/
```

Keep source, evidence, prompts, anchors, generated clips, and final exports on the canvas. Name nodes with stable prefixes such as `01_source`, `02_frames`, `03_transcript`, `04_analysis`, `05_prompt`, `06_anchor`, `07_clip`, `08_qa`, `09_export`.

When people appear in the planned output, replace the generic anchor stage with the gated production branch below. A multi-character shot may consume several separately approved character nodes, but a character asset node may never contain or represent more than one person.

In `build`, stop after the graph is wired and parameters are populated. In `auto`, continue.

### 5. Prompt assembly

Build one global continuity prompt and one shot prompt per timeline segment. Generalize clothing-specific actions or dialogue before product/full replacement so the new product is not forced to perform impossible features.

Read [references/prompt-assembly.md](references/prompt-assembly.md) for the four builders, continuity locks, action generalization, and Canvas `@` references.

### 6. Generation

For `auto` only:

1. Lock the analyzed or rewritten screenplay, then extract the cast, scenes, dialogue order, shots, continuity, and target duration.
2. Create exactly one character asset node per character. Each asset must contain one person only. Never place two or more people, a group portrait, a contact sheet of different identities, or multiple characters inside one character asset node.
3. Connect each character prompt/profile node to its own image-generation node, run it, verify a real successful output and a valid edge, then show every character asset separately to the user.
4. Stop at the character approval gate. Silence, an earlier broad approval, or node success is not approval. Continue only after the user explicitly confirms each required character asset; regenerate only rejected characters and ask again.
5. After character approval, follow the user's audio choice per character. If audio is requested, create one corresponding character-audio node, use that character's approved image as its identity input, connect the approved asset node to the audio node, and verify that the edge and audio output both succeeded. If audio is not requested, do not create or run a character-audio node.
6. Generate product and scene anchors, a storyboard table, and shot prompts from only confirmed dependencies. Multi-character shots receive one separate incoming edge from each approved character asset; never merge them into a combined character-reference asset.
7. Generate scene or shot video nodes in story order, using local zero-based timing inside each generation call. Reuse confirmed assets, audio, seeds where supported, aspect ratio, and style locks. Preserve successful upstream nodes when one shot fails.
8. Compose only verified outputs in timestamp order; add requested dialogue, music, effects, and subtitles, then export the requested master with every intermediate node traceable.

Read [references/short-drama-production.md](references/short-drama-production.md) before any media generation involving characters. Its state ledger, approval gate, audio branching, edge verification, and invalidation rules are mandatory.

Do not silently substitute an unavailable model. Choose the closest connected component and record the substitution in the run summary.

### 7. Validation

Read [references/qa-and-fallbacks.md](references/qa-and-fallbacks.md). Before declaring completion, verify:

- analysis JSON parses and includes all required fields;
- prompt `@` references match the resolved mode;
- every character asset contains exactly one person and has its own node;
- every character used downstream has explicit user approval recorded against the current asset version;
- requested character audio has one verified node and successful edge from the matching approved character asset;
- opted-out character audio has no generated audio node or dangling audio edge;
- `clone` has no replacement-material reference;
- product/full swap contains an explicit product-fidelity lock;
- spoken content is present when speech evidence exists;
- character, product, scene, action, timing, camera, text, and audio remain consistent;
- the timeline has no gap, overlap, duplicate shot, or unintended aspect-ratio change;
- export node visibly succeeded and the result is previewable/downloadable;
- the replacement summary accurately states what changed and what stayed fixed.

Fix failed checks before delivery when a safe retry remains. Limit automatic retries to two per failing stage, then degrade honestly.

## Batch Mode

Process each reference in an isolated group with its own configuration, evidence, prompts, status, and export. Reuse shared person/product anchors only when the user intends the same identity or product across videos.

Batch completion is partial-success tolerant: continue after an item fails, then report totals and the last successful stage for every item. Never let one low-quality source contaminate another video's analysis.

## Deliverables

Return only artifacts supported by the chosen execution mode:

- structured analysis JSON and readable summary;
- timestamped transcript or an explicit `no speech detected` note;
- global prompt and per-shot prompts;
- mode-specific upload/connection instructions;
- replacement summary;
- Canvas graph summary with node/component names;
- QA report and any degraded-mode warning;
- in `auto`, final preview/export and retained intermediate nodes.

## Failure Policy

Prefer a truthful partial result over an empty result. Every degraded output must state:

1. what is missing;
2. why it is missing;
3. the expected quality impact;
4. the exact node or user action that can restore full quality.

Never fabricate frames, transcripts, analysis, tool success, or exported media. Use the recovery table in [references/qa-and-fallbacks.md](references/qa-and-fallbacks.md).

## Domain Role

The references define evidence extraction, schemas, prompt assembly, tool contracts, and recovery paths. They assist execution; they are not cached results. Always operate on the user's actual media and current connected Lumina tools.
