# Lumina Tool Contracts

Load this reference during preflight, canvas construction, or tool failure diagnosis.

Lumina Agent can call a workflow only after it is packaged as a Component and connected to the Agent as a Tool. The names below are capability contracts, not required product names. Map them to the strongest components available in the current workspace.

## Required capability map

| Contract | Required input | Required output | Notes |
|---|---|---|---|
| `inspect_video` | video | metadata + audio presence | Duration, dimensions, aspect ratio, frame rate when available |
| `sample_frames` | video + sampling config | timestamped frames or contact sheets | Preserve source timestamps and ordering |
| `transcribe_audio` | video/audio + language hint | transcript + timestamps | Speaker labels and emotion are optional but useful |
| `analyze_reference` | frames/video + transcript | schema-valid JSON | Must accept both evidence branches together |
| `generate_image` | prompt + optional references | image | Character mode must return one solo identity per node |
| `generate_character_audio` | one approved character image + speech/voice settings | one character audio artifact | Optional; create only when requested for that character |
| `generate_video` | prompt + optional image/audio/video refs | clip | Prefer models with reference and first/last-frame control |
| `compose_timeline` | ordered clips + audio/subtitles | master video | Must preserve requested aspect ratio and timing |
| `preview_or_export` | media | visible preview or downloadable file | Success must be observable |

`auto` requires every contract from ingest through export. `analyze` needs only the first four. `build` may create placeholders for unavailable downstream contracts, but each placeholder must be visibly labeled `MISSING TOOL`.

## Agent configuration

- Use a vision-capable main model for exact analysis.
- Set a deterministic or low-temperature reasoning configuration for extraction and schema writing.
- Allow enough max iterations for the requested branches, but do not increase it merely to hide repeated failures.
- Give every tool a description that states required fields, accepted ranges, and output types.
- Prefer stable node names so prompts can reference `@person_ref`, `@product_ref`, `@scene_anchor`, and `@shot_prompt` instead of positional inputs.

## Suggested tool descriptions

### Reference analyzer

> Accepts a reference video or timestamped frame set plus an optional transcript. Returns evidence-grounded JSON using the selected exact or rewrite schema. It does not generate media.

### Anchor generator

> Generates a single character, product, or scene anchor image from a prompt and named references. Keep the requested aspect ratio and return an image that can feed downstream video nodes.

For character assets, configure a separate generator node for every `character_id`. The returned asset must show exactly one person. Do not use one generator node to emit a combined cast sheet.

### Character audio generator

> Accepts exactly one explicitly approved character image and that character's requested speech or voice settings. Returns one audio artifact for that character. It must not run when the user opted out of character audio.

### Shot generator

> Generates one shot from a shot prompt and named anchor inputs. Required: duration and aspect ratio. Optional: first frame, last frame, reference video, reference audio, seed. Returns one video clip.

### Timeline composer

> Concatenates ordered clips, mixes dialogue/music/effects, adds subtitles, and exports a master without changing the target aspect ratio.

## Graph invariants

- Keep each video's branches inside one labeled group.
- Never connect a generated clip back into the evidence-analysis branch.
- Preserve the original source node even after derived frames or audio exist.
- Keep replacement materials on a separate branch so mode changes do not require re-ingest.
- Connect only the references required by the resolved replacement mode.
- Keep one character asset per node and one optional character-audio node per character. Character asset nodes never contain two or more people.
- Connect each approved character image directly to its matching character-audio node before running audio. Verify the real Canvas edge and returned artifact; a textual `@` mention is not proof of connection.
- Multi-character video nodes receive separate named edges from separately approved character asset nodes.
- A green/successful node is necessary but not sufficient; preview the semantic result before marking QA complete.
