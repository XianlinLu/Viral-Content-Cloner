# QA and Fallbacks

Load this reference during validation or after any node failure.

## QA gates

### Evidence gate

- Every uploaded reference video is bound to a Lumina built-in Video node, not an unrelated node type.
- The native Video node is visibly present on the Canvas, fully loaded, and exposes a playable preview or readable video metadata.
- The node's bound source matches the user's original upload; a filename, URL label, or node title alone is not proof.
- Source is readable and its duration/aspect ratio are known or marked unknown.
- Frame or direct-video evidence covers beginning, middle, ending, and short reveals.
- Transcript is aligned to time or explicitly unavailable.
- Analysis contains no unsupported detail presented as fact.

### Prompt gate

- Mode and `@` references agree.
- Clone mode uses no replacement materials.
- Product/full swap includes the product-fidelity lock.
- Dialogue fits the shot duration.
- Shot prompts do not contradict the global continuity prompt.

### Visual gate

- Every required character has one dedicated asset node, and every character asset depicts exactly one person.
- No character asset contains a couple, crowd, second identity, background person, reflected person, portrait, poster, or screen showing another human.
- The current asset version for every downstream character has explicit user confirmation.
- Person identity, age, face, hair, body, and clothing remain consistent when they should.
- Product color, shape, material, pattern, and logo placement do not drift.
- Scene layout, prop placement, time of day, lighting direction, and palette stay stable.
- Hands, object interactions, readable text, and screen content are plausible.
- Camera angle/movement and aspect ratio match the shot plan.

### Timeline/audio gate

- Each character's audio choice is recorded. Requested audio is generated only after asset confirmation; unrequested audio has no generation node.
- Every requested character-audio node receives a verified edge from the matching approved character asset node and represents only that character.
- Multi-character shots receive separate character-reference edges rather than a combined character asset.
- No missing, duplicated, overlapping, or out-of-order shot.
- Dialogue and visible speaker are synchronized where required.
- Music does not mask speech; effects occur at the correct action.
- Subtitles match the transcript, remain inside the safe area, and do not cover product details.
- Master duration and export settings match the user's request.

## Retry policy

Retry a failed stage at most twice automatically:

1. first retry: same evidence with a corrected parameter or stricter prompt;
2. second retry: simpler compatible component/model or reduced scope;
3. then stop that branch, preserve successful nodes, and report degradation.

Do not rerun successful upstream nodes unless the evidence itself is wrong. Regenerate only the failed shot when continuity permits.

## Recovery table

| Failure | Detection | Recovery | Degraded result |
|---|---|---|---|
| wrong source node type | uploaded video appears as Image, Audio, Text, Note, File, generated-video, placeholder, or unrelated Component | remove or ignore the incorrect node, create the Lumina built-in Video node, bind the original upload, and re-verify ingest | no downstream analysis until corrected |
| native Video node not loaded or not visible | node exists but source is empty, load is pending/failed, preview and metadata are absent, or the node is off-canvas/not rendered | retry binding/loading once, bring the native node into the visible source group, then verify creation, load, and visibility separately | preserve upload handle; stop that video's branch if still unresolved |
| source cannot load | ingest node fails | ask for re-upload, accessible URL, or supported encode | no analysis; keep graph placeholder |
| visual sampler missing | no connected component | use direct video understanding with timestamp requirement | lower temporal precision |
| visual analysis fails | error or invalid JSON | retry once; switch exact -> rewrite | flat viral-logic analysis |
| all vision analysis fails | both modes fail | preserve source, frames, transcript, and schema template | manual analysis required |
| no audio track | inspector reports none | skip transcription normally | visual-only analysis |
| transcription fails | error/timeout | retry, then use burned-in captions if clearly readable | dialogue may be incomplete |
| replacement reference missing | resolved mode needs absent input | fall back to clone only if that still meets user intent; otherwise pause that branch | no requested swap |
| character asset contains multiple people | visual inspection fails | tighten the solo-character prompt and regenerate only that character | stop its dependent audio and video branches |
| character approval missing | current asset version is pending/revise | display that asset separately and request explicit confirmation | no dependent audio or video generation |
| character audio edge missing or mismatched | canvas edge absent or wrong source character | reconnect the approved matching asset and rerun only that audio node | omit that character's audio until verified |
| anchor generation drifts | identity/product mismatch | tighten anchor prompt, reuse accepted seed/reference, enable face or reference protection | stop before clips if unresolved |
| one clip fails | generation node error | regenerate only that clip; simplify motion before model substitution | sequence with labeled gap if unresolved |
| composition fails | timeline/export error | retain ordered clips and audio/subtitle assets | no final master |
| export unavailable | no save/export tool | leave verified preview node and exact export settings | preview only |

## Required degraded-output notice

Use this structure:

```text
Status: partial
Missing: {artifact or capability}
Cause: {observed failure}
Quality impact: {specific consequence; avoid fake percentages}
Recovery: connect/run {component or user action}
Last successful node: {stable node name}
```

## Batch failure handling

Mark every item `completed`, `partial`, or `failed`. Continue unrelated items. The final batch report must list:

- source name;
- mode;
- last successful node;
- missing artifact;
- retry count;
- final preview/export location when available.

## Completion rule

Do not say “finished” until the promised execution-mode output exists:

- `analyze`: parsed analysis + transcript status + prompt package;
- `build`: connected graph with visible parameters and missing-tool labels;
- `auto`: previewable/exportable master plus QA report.

No mode is complete if an uploaded reference video lacks its visible, loaded Lumina built-in Video source node.

For character-based `auto` runs, completion also requires a passing ledger from `short-drama-production.md`. A final export does not override a failed character, approval, audio, or edge gate.
