# Lumina Viral Content Cloner

[English](#english) | [中文](#中文)

## English

**Lumina Viral Content Cloner is an original, purpose-built Skill for Lumina Canvas Agent.** It turns reference videos into traceable, reusable Canvas workflows that can analyze creative structure, assemble generation prompts, replace people or products, generate shot-level clips, validate continuity, and export finished media.

```text
Reference video
  ├─ Visual samples / timestamp evidence ─┐
  ├─ Audio / dialogue transcript ─────────┼→ Analysis → Script lock → Solo character nodes → User approval
  └─ Person / product references ─────────┘
                                                        ↓
                                  Optional character audio → Storyboard → Video → QA → Export
```

The Skill is designed around Lumina's native Agent, Node, Component, Tool, multimodal input, named `@` reference, and visual workflow concepts. Every stage remains visible on the Canvas so a creator can inspect, rerun, replace, or reuse it independently.

### Capabilities

- Analyze one reference video or a batch of videos.
- Use `exact` mode for close reconstruction or `rewrite` mode for transferable viral structure.
- Support pure cloning, person replacement, product/garment replacement, and full replacement.
- Produce structured analysis, timestamped dialogue, a global continuity prompt, and shot prompts.
- Automatically match all visible interaction to the user's language while routing dialogue, subtitles, narration, and speech through a separately controlled content language.
- Use a gated short-drama production flow: confirmed story, cast extraction, solo character assets, explicit user approval, optional character audio, storyboard, video, QA, and export.
- Enforce one character asset per node and exactly one person per character asset. Multi-character shots connect several approved solo nodes instead of using a combined cast asset.
- Generate character audio only when requested, only after that character is approved, and only through a verified edge from the matching character image node.
- Build a reusable Canvas graph without generating media, or continue through generation and export.
- Preserve successful nodes and provide explicit recovery paths when a component is unavailable or fails.

### Lumina-native architecture

| Design area | Lumina implementation |
|---|---|
| Media evidence | Video-understanding or frame-sampling Components |
| Speech evidence | A transcription Component connected as an Agent Tool |
| Reasoning | Vision-capable Agent model and structured analysis Component |
| Generation control | Named inputs such as `@person_ref`, `@product_ref`, and `@scene_anchor` |
| Continuity | Shared anchors, stable prompts, optional fixed seeds, and shot-level QA |
| Character control | One solo character per node, versioned approvals, and separate named edges into multi-character shots |
| Optional character audio | One approved character image → one matching audio node; omitted completely when not requested |
| Recovery | Node-level retries, isolated regeneration, visible missing-tool placeholders |
| Delivery | Preview, composition, subtitle/audio, save, and export Components |

### Install in Lumina Canvas Agent

Lumina Agent can call a workflow only after it has been packaged as a **Component** and connected to the Agent as a **Tool**.

1. Create or open a Lumina Canvas.
2. Add an Agent node.
3. Use the body of [SKILL.md](./SKILL.md) as the Agent's system instructions, or import it through your team's Skill workflow.
4. Connect Components that provide:
   - video loading and metadata inspection;
   - frame sampling or direct video understanding;
   - speech transcription;
   - image generation or editing;
   - video generation;
   - timeline, audio, and subtitle composition;
   - preview, save, and export.
5. Use [tool-contracts.md](./references/tool-contracts.md) to describe each Tool's parameters, accepted inputs, and outputs.
6. Set enough Agent Max Iterations for the requested execution mode, test with a short reference video, and save the verified workflow for reuse.

For platform concepts and current Component behavior, see the [Lumina Canvas documentation](https://seedancelumina.com/guide).

### Usage examples

```text
Analyze this reference video in exact mode. Build the Canvas workflow but do not generate the final video yet.
```

```text
Recreate this video with @person_ref. Preserve the original pacing, wardrobe, scene, dialogue, and camera, then generate and export a 9:16 version.
```

```text
Analyze these five videos as a batch. Extract their shared first-three-second hook and conversion structure without generating media.
```

#### Execution modes

| Mode | Best for | Deliverable |
|---|---|---|
| `analyze` | Analysis and reverse engineering | Analysis JSON, transcript status, prompt package |
| `build` | Workflow construction | Connected and parameterized Canvas graph |
| `auto` | End-to-end production | Analysis, graph, shot generation, QA, final export |

#### Replacement modes

| Mode | Material inputs | Replaced content |
|---|---|---|
| `clone` | None | Nothing |
| `person_swap` | Person reference | Face and body identity |
| `product_swap` | Product or garment reference | Product or clothing |
| `full_swap` | Person + product references | Person and product |

### Repository structure

```text
.
├── SKILL.md
├── README.md
├── LICENSE.md
├── agents/
│   └── openai.yaml
└── references/
    ├── tool-contracts.md
    ├── media-analysis.md
    ├── analysis-schemas.md
    ├── prompt-assembly.md
    ├── short-drama-production.md
    └── qa-and-fallbacks.md
```

### Original design principles

- **Evidence first:** describe only what the supplied video frames and audio support.
- **Canvas traceability:** keep sources, evidence, prompts, anchors, clips, QA, and exports visible.
- **Human approval gates:** never treat node success or silence as character approval.
- **Single-character assets:** one person, one asset, one node, with separate edges into every dependent generator.
- **Local retries:** regenerate only the failed stage or shot instead of repeating successful work.
- **Compute awareness:** only `auto` proceeds into generation; `build` stops at a runnable workflow.
- **Honest degradation:** identify what is missing, why it failed, its impact, and the recovery action.

### Validation

Validate the package with Skill Creator's validator:

```bash
python /path/to/skill-creator/scripts/quick_validate.py .
```

The validator checks frontmatter, naming, and unfinished scaffold markers. Before production use, also run one short reference video end to end in Lumina.

### License

MIT. See [LICENSE.md](./LICENSE.md).

---

## 中文

**Lumina Viral Content Cloner 是一套为 Lumina Canvas Agent 原创设计、从零构建的 Skill。** 它能够把参考视频转换成可追溯、可复用的 Lumina 画布工作流，完成创意结构分析、生成提示词组装、人物或商品替换、逐镜生成、一致性验证与最终导出。

```text
参考视频
  ├─ 视觉采样 / 时间戳证据 ─┐
  ├─ 音频 / 对白转录 ──────┼→ 结构化分析 → 剧本锁定 → 独立人物节点 → 用户确认
  └─ 人物 / 商品替换素材 ──┘
                                                        ↓
                                  可选人物音频 → 分镜表 → 视频生成 → QA → 导出
```

整套 Skill 围绕 Lumina 原生的 Agent、Node、Component、Tool、多模态输入、命名式 `@` 引用和可视化工作流进行设计。每个阶段都会保留在画布中，方便创作者独立检查、重跑、替换与复用。

### 核心能力

- 分析单条参考视频或批量视频。
- 使用 `exact` 模式进行精确复刻，或使用 `rewrite` 模式提取可迁移的爆款结构。
- 支持纯复刻、替换人物、替换商品/服装以及人物与商品全替换。
- 输出结构化分析、时间轴对白、全局一致性 Prompt 和逐镜 Prompt。
- 自动适配用户当前使用的语言，同时将对白、字幕、旁白和语音作为独立的内容语言进行控制。
- 使用带硬门槛的短剧生产流程：确认故事、拆分角色、生成独立人物素材、用户逐一确认、按需生成人物音频、制作分镜、生成视频、QA 与导出。
- 强制一个人物素材对应一个节点，且每张人物素材只能出现一个人物。多人镜头必须分别连接多个已确认的单人人物节点，禁止使用合并人物素材。
- 仅在用户需要时生成人物音频；必须先确认对应人物图，再由该人物图通过真实连线连接到专属音频节点。
- 可以只搭建可复用画布，也可以继续执行生成、合成与导出。
- 当组件缺失或节点失败时，保留成功结果并提供明确的恢复路径。

### Lumina 原生架构

| 设计环节 | Lumina 实现方式 |
|---|---|
| 媒体证据 | 视频理解或帧采样 Component |
| 语音证据 | 作为 Agent Tool 连接的转录 Component |
| 分析推理 | 具备视觉能力的 Agent 模型与结构化分析 Component |
| 生成控制 | `@person_ref`、`@product_ref`、`@scene_anchor` 等命名输入 |
| 一致性控制 | 共享锚点、稳定 Prompt、可选固定 Seed 与逐镜 QA |
| 人物控制 | 一人一节点、版本化确认，多人镜头分别接入独立人物节点 |
| 可选人物音频 | 一个已确认人物图连接一个对应音频节点；用户不需要时完全不生成 |
| 故障恢复 | 节点级重试、局部重生成与可见的缺失工具占位 |
| 结果交付 | 预览、合成、音频/字幕、保存与导出 Component |

### 安装到 Lumina Canvas Agent

Lumina Agent 只有在工作流被封装成 **Component** 并作为 **Tool** 连接后，才能调用实际能力。

1. 新建或打开一个 Lumina Canvas。
2. 添加 Agent 节点。
3. 将 [SKILL.md](./SKILL.md) 的正文作为 Agent 系统指令，或通过团队的 Skill 工作流导入。
4. 为 Agent 连接能够提供以下能力的 Components：
   - 视频读取与元数据检查；
   - 帧采样或直接视频理解；
   - 语音转录；
   - 图片生成或编辑；
   - 视频生成；
   - 时间线、音频与字幕合成；
   - 预览、保存与导出。
5. 参考 [tool-contracts.md](./references/tool-contracts.md) 描述每个 Tool 的参数、输入类型和输出类型。
6. 根据执行模式设置足够的 Agent Max Iterations，先用短参考视频完成端到端测试，再保存为可复用工作流。

平台概念与当前 Component 行为可参考 [Lumina Canvas 文档](https://seedancelumina.com/guide)。

### 使用示例

```text
用 exact 模式分析这个参考视频，只搭建画布工作流，暂时不要生成最终视频。
```

```text
复刻这个视频，把人物替换成 @person_ref，保留原视频的节奏、服装、场景、对白和镜头，然后生成并导出 9:16 版本。
```

```text
批量分析这五条视频，提取共同的前三秒钩子与转化结构，不生成媒体内容。
```

#### 三种执行模式

| 模式 | 适合场景 | 最终交付 |
|---|---|---|
| `analyze` | 分析与逆向拆解 | 分析 JSON、转录状态、Prompt 包 |
| `build` | 搭建工作流 | 已连接并配置参数的 Canvas 图 |
| `auto` | 端到端制作 | 分析、画布、逐镜生成、QA、最终导出 |

#### 四种替换模式

| 模式 | 输入素材 | 替换内容 |
|---|---|---|
| `clone` | 无 | 不替换 |
| `person_swap` | 人物参考 | 人脸与身材身份 |
| `product_swap` | 商品或服装参考 | 商品或服装 |
| `full_swap` | 人物 + 商品参考 | 人物与商品 |

### 仓库结构

```text
.
├── SKILL.md
├── README.md
├── LICENSE.md
├── agents/
│   └── openai.yaml
└── references/
    ├── tool-contracts.md
    ├── media-analysis.md
    ├── analysis-schemas.md
    ├── prompt-assembly.md
    ├── short-drama-production.md
    └── qa-and-fallbacks.md
```

### 原创设计原则

- **证据优先**：只描述用户提供的视频画面与音频真正支持的内容。
- **画布可追溯**：源素材、证据、Prompt、锚点、镜头、QA 和导出全部保留在画布中。
- **人工确认门槛**：节点成功或用户沉默都不能代替人物素材确认。
- **单人人物资产**：一个人物、一份素材、一个节点，并通过独立连线接入所有下游生成节点。
- **局部重试**：只重生成失败阶段或镜头，不重复消耗已经成功的节点。
- **算力克制**：只有 `auto` 会进入生成阶段；`build` 停在可运行工作流。
- **诚实降级**：明确缺少什么、失败原因、质量影响与恢复动作。

### 验证

可以使用 Skill Creator 验证器检查 Skill 包：

```bash
python /path/to/skill-creator/scripts/quick_validate.py .
```

验证器会检查 frontmatter、命名和未完成的脚手架标记。正式使用前，还应在 Lumina 中使用一条短参考视频完成端到端测试。

### 许可证

MIT，详见 [LICENSE.md](./LICENSE.md)。
