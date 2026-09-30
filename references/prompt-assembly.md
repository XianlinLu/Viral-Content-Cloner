# Lumina Prompt Assembly

Load this reference after analysis and before image/video generation.

## Prompt package

Create:

1. `global_continuity_prompt` — invariant person, product, scene, lighting, palette, realism/style, aspect ratio, and prohibited drift;
2. `shot_prompts[]` — one prompt per timeline segment;
3. `negative_prompt` — observable failure risks, not a generic keyword dump;
4. `audio_plan` — dialogue, voice, sound effects, music, and mix priorities;
5. `replacement_summary` — mode plus preserved/replaced fields.

Keep each model-facing shot prompt concise enough for the selected video component. If a component has a prompt limit, store the full prompt in a Text node and a shortened, semantically equivalent version in the generation node.

## Builders

### Clone

No replacement-material `@` references.

```text
{person description}, wearing/using {product description}, in {scene}.
From {start}s to {end}s: {action and dialogue}.
Camera: {framing, angle, motion}. Lighting and palette: {look}.
Audio: {speech, effects, music}. Preserve the source timing and composition.
```

### Person swap

```text
The person identity, face, and body proportions follow @person_ref.
Preserve {original product/clothing}, {scene}, {actions}, {dialogue}, and {camera}.
From {start}s to {end}s: {shot-specific action}.
Keep identity, hairstyle, and body proportions consistent across every shot.
```

### Product swap

```text
{original person description} wears/uses @product_ref.
Preserve {scene}, {generalized actions}, {generalized dialogue}, and {camera}.
Do not alter the product's pattern, color, material, shape, logo placement, or construction.
From {start}s to {end}s: {shot-specific action}.
```

### Full swap

```text
The person follows @person_ref and wears/uses @product_ref.
Preserve {scene}, {generalized actions}, {generalized dialogue}, and {camera}.
Keep identity stable. Do not alter the product's pattern, color, material, shape, logo placement, or construction.
From {start}s to {end}s: {shot-specific action}.
```

## Action generalization for product replacement

Replace source-product terms only when needed to make the action valid for the new product.

| Source-specific | General form |
|---|---|
| skirt hem / 裙摆 | garment edge / 衣角 |
| dress / 连衣裙 | garment / 服装 |
| trouser leg / 裤腿 | lower garment / 下装 |
| chiffon, silk, denim, knit | material / 面料 |
| floral, stripe, plaid | pattern / 图案 |
| A-line, slim, oversized | silhouette / 版型 |
| fishtail or umbrella hem | lower edge / 下摆 |

Do not generalize a functional demonstration if the replacement lacks that feature. Rewrite the action around an observable feature of `@product_ref`, or mark the shot for user review.

## Canvas reference rules

- Use named references: `@source_video`, `@person_ref`, `@product_ref`, `@scene_anchor`, `@first_frame`, `@last_frame`, `@dialogue_audio`.
- For a cast, use one stable name per approved solo-character node, such as `@character_hero` and `@character_rival`. Never use a combined cast image as a character asset reference.
- A reference must be connected to the node that uses it.
- Do not mention a disconnected or nonexistent `@` input.
- Reuse the same anchors across shots. Do not regenerate identity or product anchors inside every shot branch.
- A multi-character shot must receive a separate Canvas edge for each character reference. Keep optional character audio on matching per-character branches until the dialogue-mix stage.
- Keep one canonical Text node for the global continuity prompt and fan it out to shot nodes.

## Shot prompt structure

Every shot should state:

- exact start/end and duration;
- subject count and stable identity;
- scene and fixed spatial relationships;
- action, hand/direction, and object interaction;
- dialogue line or `no speech`;
- framing, angle, and motion;
- lighting/color/style;
- audio event;
- transition in/out only when required.

Avoid contradictory camera instructions, impossible action density, and long dialogue that cannot fit the shot duration.

## Replacement summary

```json
{
  "mode": "full_swap",
  "person_replaced": true,
  "body_replaced": true,
  "product_replaced": true,
  "preserved": ["scene", "actions", "dialogue", "camera", "audio_structure"],
  "references": {
    "person": "@person_ref",
    "product": "@product_ref"
  }
}
```

## Canvas SOP by mode

- `clone`: connect source evidence and prompt nodes; do not connect person/product materials.
- `person_swap`: add and connect one consented person reference.
- `product_swap`: add and connect one clean product reference.
- `full_swap`: connect both named references to anchor and shot-generation components.

In all modes, connect final clips to the timeline composer in timestamp order, then connect dialogue, effects, music, and subtitles before export.
