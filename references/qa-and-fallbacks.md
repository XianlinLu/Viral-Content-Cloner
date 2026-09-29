# QA and Fallbacks

Load this reference during validation or after any node failure.

## QA gates

### Evidence gate

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

- Person identity, age, face, hair, body, and clothing remain consistent when they should.
- Product color, shape, material, pattern, and logo placement do not drift.
- Scene layout, prop placement, time of day, lighting direction, and palette stay stable.
- Hands, object interactions, readable text, and screen content are plausible.
- Camera angle/movement and aspect ratio match the shot plan.

### Timeline/audio gate

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
| source cannot load | ingest node fails | ask for re-upload, accessible URL, or supported encode | no analysis; keep graph placeholder |
| visual sampler missing | no connected component | use direct video understanding with timestamp requirement | lower temporal precision |
| visual analysis fails | error or invalid JSON | retry once; switch exact -> rewrite | flat viral-logic analysis |
| all vision analysis fails | both modes fail | preserve source, frames, transcript, and schema template | manual analysis required |
| no audio track | inspector reports none | skip transcription normally | visual-only analysis |
| transcription fails | error/timeout | retry, then use burned-in captions if clearly readable | dialogue may be incomplete |
| replacement reference missing | resolved mode needs absent input | fall back to clone only if that still meets user intent; otherwise pause that branch | no requested swap |
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
