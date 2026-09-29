# Lumina Viral Content Cloner

面向 **Lumina Canvas Agent** 的参考视频拆解与复刻 Skill。它把参考视频变成一条可追溯的 Lumina 画布工作流：

```text
参考视频
  ├─ 视觉采样 / 时间戳证据 ─┐
  ├─ 音频 / 对白转录 ──────┼→ 结构化分析 → Prompt 包 → 锚点 → 分镜视频 → QA → 导出
  └─ 人物 / 商品替换素材 ──┘
```

本项目参考并重新设计自 [dingtom336-gif/viral-video-replicator](https://github.com/dingtom336-gif/viral-video-replicator)（参考提交：`dde67e0`）。原项目面向本地 FFmpeg、火山方舟 Vision、TOS 和 Seed-ASR；本版本改为 Lumina 的 Agent、Node、Component、`@` 引用与画布执行模型。

## 能做什么

- 单条或批量拆解参考视频；
- `exact` 精确复刻与 `rewrite` 爆款逻辑改写；
- 纯复刻、换人物、换商品/服装、人物与商品全换；
- 输出结构化分析、时间轴对白、全局 Prompt 和逐镜 Prompt；
- 自动搭建可追溯画布，或继续生成、合成与导出成片；
- 在组件缺失或节点失败时保留可用结果并明确降级。

## 与原 Skill 的主要差异

| 原始实现 | Lumina 版本 |
|---|---|
| 本地 FFmpeg/ffprobe 抽帧 | Lumina 视频理解或帧采样 Component |
| TOS 上传 + Seed-ASR API | 连接到 Agent 的转录 Component |
| 火山方舟 API Key + Vision 模型 ID | Lumina Agent 主模型与分析 Component |
| 输出 Seedance 复刻 Prompt + SOP | 输出 Prompt 包、节点图、可选完整生成与导出 |
| `@图片1` / `@图片2` 位置引用 | `@person_ref` / `@product_ref` 等稳定节点名 |
| 本地错误降级 | 画布节点级重试、局部重生成与可见缺口 |

## 安装到 Lumina Canvas Agent

Lumina 的 Agent 只有在工作流被封装为 **Component** 并作为 **Tool** 连接后，才能调用实际能力。建议按以下方式配置：

1. 在 Lumina 新建或打开一个 Canvas。
2. 添加 Agent 节点。
3. 将 [SKILL.md](./SKILL.md) 的正文作为 Agent 的系统指令，或导入到团队的 Skill 管理方式中。
4. 给 Agent 连接能覆盖这些能力的 Components：
   - 视频读取/元数据检查；
   - 帧采样或视频理解；
   - 音频转录；
   - 图片生成/编辑；
   - 视频生成；
   - 时间线合成与音频/字幕；
   - 预览/保存/导出。
5. 参考 [工具契约](./references/tool-contracts.md) 为每个 Tool 写清参数、输入类型与输出类型。
6. 设置足够的 Agent Max Iterations，先用短视频跑通一次，再保存为云端工作流或 AI App。

Lumina 官方文档强调：工作流必须先封装为 Component 并连接到 Agent，Agent 才能使用它；Canvas 支持文本、图片、视频、音频与多模态节点。[Lumina Canvas 文档](https://seedancelumina.com/guide)

## 使用方式

直接对已配置好的 Agent 说：

```text
用这个参考视频做 exact 分析，先只搭建画布，不生成成片。
```

```text
复刻这个视频，把人物替换成 @person_ref，其他节奏、服装、场景和镜头保持不变，自动生成并导出 9:16 版本。
```

```text
批量分析这 5 条视频，提取共同的前 3 秒钩子与转化结构，不生成视频。
```

### 三种执行模式

| 模式 | 适合请求 | 最终交付 |
|---|---|---|
| `analyze` | “分析/拆解” | 分析 JSON、转录、Prompt 包 |
| `build` | “搭建画布/工作流” | 已连接并填好参数的 Canvas 图 |
| `auto` | “生成/复刻/出片” | 分析、画布、逐镜生成、QA、导出 |

### 四种替换模式

| 模式 | 素材 | 替换内容 |
|---|---|---|
| `clone` | 无 | 不替换 |
| `person_swap` | 人物参考 | 人脸/身材身份 |
| `product_swap` | 商品或服装参考 | 商品/服装 |
| `full_swap` | 人物 + 商品参考 | 人物与商品 |

## 文件结构

```text
.
├── SKILL.md
├── README.md
├── LICENSE
├── agents/
│   └── openai.yaml
└── references/
    ├── tool-contracts.md
    ├── media-analysis.md
    ├── analysis-schemas.md
    ├── prompt-assembly.md
    └── qa-and-fallbacks.md
```

## 设计原则

- **证据优先**：只描述实际帧、视频和音频支持的内容。
- **可追溯**：源视频、证据、Prompt、锚点、镜头与导出都保留在画布。
- **局部重试**：单镜失败只重跑该镜，不反复消耗成功节点。
- **算力克制**：只有 `auto` 才自动生成；`build` 停在可运行工作流。
- **诚实降级**：缺工具、缺音频或导出失败时，明确最后成功节点与恢复方法。

## 验证

项目可用 Skill Creator 的验证器检查：

```bash
python /path/to/skill-creator/scripts/quick_validate.py .
```

验证器能检查 frontmatter、命名与脚手架残留；实际使用前仍应在 Lumina 中用一条短视频做端到端测试。

## License

MIT。详见 [LICENSE](./LICENSE)。

---

## English

This repository adapts the complete reverse-engineering workflow of `viral-video-replicator` for Lumina Canvas Agent. It replaces local FFmpeg/API plumbing with connected Lumina Components, named `@` references, traceable node groups, execution modes, shot-level generation, QA gates, and honest fallbacks. Start with [SKILL.md](./SKILL.md) and connect tools according to [tool-contracts.md](./references/tool-contracts.md).
