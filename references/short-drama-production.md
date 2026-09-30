# Short-Drama Production Gates

Load this reference before any `auto` run that generates people, character speech, storyboards, or video. The gates below are execution requirements, not suggestions.

## State order

```text
confirmed analysis or screenplay
→ cast and scene extraction
→ one solo character asset node per character
→ explicit user confirmation of every required character asset
→ optional one-character audio branches
→ storyboard table and shot plan
→ scene or shot video generation
→ timeline QA and export
```

Never skip forward because a tool call succeeded. Tool success proves only that an artifact exists; it does not equal user approval.

## Production ledger

Maintain a visible Text or structured-data node with at least:

```yaml
interaction_language: language tag or name
content_language: language tag or name
screenplay_version: integer
characters:
  character_id:
    display_name: string
    profile_version: integer
    asset_node: stable node name
    asset_version: integer
    asset_status: pending | confirmed | revise | failed
    asset_edge_status: pending | connected | failed
    audio_requested: true | false
    audio_node: stable node name | none
    audio_version: integer | none
    audio_status: not_requested | pending | confirmed | revise | failed
    audio_edge_status: not_required | pending | connected | failed
scenes:
  scene_id:
    screenplay_version: integer
    storyboard_version: integer
    character_asset_versions: map of character_id to asset_version
    character_audio_versions: map of character_id to audio_version or none
    video_status: pending | confirmed | revise | stale | failed
```

Only the latest explicitly confirmed asset version may feed downstream nodes.

## 1. Lock story intent and extract the cast

Use the analyzed reference and selected `exact` or `rewrite` strategy to lock the screenplay or production outline. Extract each character who appears in a generated shot or supplies character-specific speech. Give each one a stable `character_id` and profile containing visible identity anchors, costume, palette, role, emotional baseline, and continuity constraints.

Separate source evidence from production choices. Do not infer sensitive traits or a real person's private identity.

## 2. Enforce one character per asset node

Create exactly one image-generation node for each character:

```text
06_character_<character_id>
```

Each output must depict exactly one character identity. A node may include several views of that same person only when the layout keeps one identity consistent. It must not include:

- a second character, crowd, group portrait, couple, or ensemble;
- another person in a mirror, screen, photo, poster, painting, or background;
- a split layout containing different identities;
- a combined cast sheet;
- a character and an unrelated human model used as a scale reference.

Use prompts such as `solo character, exactly one person, no other human figure, reflection, portrait, poster, or screen`. Inspect the returned image. If more than one person is visible or identity is ambiguous, mark the node `revise` and regenerate only that character.

Never represent two characters with one asset node. Never crop a group asset and pretend the original node is a valid solo asset.

## 3. Connect and verify each asset branch

Connect each character profile or prompt node directly to its own image-generation node. Record the real source node, target node, port mapping when exposed, and connection result. A character asset is eligible for review only when:

1. the edge exists on the Canvas;
2. the generation node returned a real image;
3. the image passed the one-person inspection;
4. the artifact is previewable.

Do not claim a node is connected merely because its prompt mentions an `@` reference.

## 4. Mandatory character approval gate

Show every character asset separately with its name or `character_id`, profile summary, node name, and current version. Ask the user in `interaction_language` to confirm or request changes for each character.

Stop the run after presenting the assets. Do not generate character audio, dialogue, storyboards that depend on unapproved identities, or character video while any required `asset_status` is not `confirmed`.

Approval must be explicit and version-specific. Silence, continuing the conversation, approving the script, or approving a different character does not count. If one character is rejected, regenerate only that character, increment its version, invalidate its former approval, and return to the same gate.

## 5. Optional character audio branch

Resolve `audio_requested` for every character from the user's instruction or a localized confirmation question. Do not assume that all characters need audio.

For `audio_requested: true`:

1. require the current character asset version to be `confirmed`;
2. create exactly one corresponding node named `07_character_audio_<character_id>`;
3. connect `06_character_<character_id>` directly to that audio-generation node as the identity/reference input;
4. generate only that character's requested audition, dialogue, or voice asset in `content_language`;
5. verify the real edge, correct source character, returned audio artifact, duration, and previewability;
6. record the audio version and status before it is used downstream.

For `audio_requested: false`, do not create, run, or bill an audio-generation node for that character. Set `audio_node: none`, `audio_status: not_requested`, and `audio_edge_status: not_required`. Do not leave a placeholder that can be mistaken for a generated result.

Never connect character A's image to character B's audio node. One audio node must not represent multiple characters. If a scene mixes several speakers, keep character audio sources separate until an ordered dialogue-mix node.

## 6. Storyboard and video generation

After the approval and optional-audio gates are satisfied, create a storyboard table with:

```text
shot id | local duration | framing and movement | character ids and asset versions | action | dialogue/audio source | environment and props | transition
```

Generate in story order. Every video node receives only current confirmed dependencies. For a shot containing multiple characters, connect each character's separate approved asset node to the video node through its own named input edge, such as `@character_hero` and `@character_rival`. The shot output may show multiple people; the upstream character asset nodes may not.

Use a local timeline beginning at `00:00` inside every video-generation call. Keep global cumulative time only in the storyboard or ledger. Preserve successful upstream assets and regenerate only a failed or stale shot when possible.

## 7. Change propagation

- A profile or appearance change increments that character's asset version and resets its status to `pending`.
- A changed asset invalidates its audio and every dependent storyboard/video node.
- A changed audio choice or audio version invalidates only dependent dialogue mixes and video nodes.
- A screenplay change re-extracts affected characters and scenes, then marks their downstream nodes stale.
- Do not mix old and new versions in the same shot.
- Rebuild stale dependencies in order and re-enter every applicable user-confirmation gate.

## Completion checks

The character branch passes only when:

- number of required characters equals number of separate character asset nodes;
- every character asset contains exactly one person;
- each current asset version has explicit user confirmation;
- every requested audio node is downstream of the matching confirmed image node through a verified Canvas edge;
- no unrequested character audio node exists;
- every multi-character shot uses separate incoming character references;
- every delivered artifact came from a real successful node;
- all visible interaction follows `interaction_language`, while dialogue, narration, subtitles, and speech follow `content_language`.
