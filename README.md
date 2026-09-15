# AI Director for Lumina Canvas

[中文](#中文介绍) · [English](#english-introduction)

一个为 **Lumina Canvas Agent** 适配的 TVC AI 导演 Skill：从 Brief 诊断、创意提报、导演阐述、PPM、分镜、AI 图像/视频 Prompt，到生成统筹、剪辑、客户审片与交付，统一在一个画布总控 Agent 中完成。

This repository provides a **Lumina Canvas Agent–ready TVC AI Director skill** that coordinates the complete advertising-film workflow—from brief diagnosis and creative development to treatment, PPM, storyboards, AI image/video prompts, generation management, editing, client review, and delivery.

**AI Director for Lumina Canvas** 的核心作用，是把真实 TVC 团队分散在策略、创意、导演、制片、AI 生成、剪辑和审片中的判断，组织成一个可执行、可复核、可持续交接的画布 Agent 工作流。

## 中文介绍

### 它是什么

这不是一个“帮我写广告 Prompt”的单点提示词，而是一套运行在 Lumina Canvas 里的 TVC 制作控制系统。

Skill 以一个总控 Agent 协调九种专业工作模式：

```text
brief -> concept -> treatment -> ppm -> storyboard
      -> ai prompt -> ai production -> editing -> review
```

Agent 会先识别当前阶段和缺失信息，再决定是输出策略、调用图像/视频组件、组织生成批次，还是解析客户反馈。所有阶段都使用统一交接格式，因此上游结果可以直接进入下游，不需要反复解释项目背景。

### 为什么适配 Lumina Canvas

Lumina Canvas 使用节点、连接、组件和 Agent 组织多模态工作流。Agent 可以接收文本、图片、视频和音频，并把已连接的组件当作工具调用；System Instructions 决定它如何理解任务、选择工具和输出结果。

这个 Skill 围绕 Lumina 画布原生机制设计：

- 将多个 `$tvc-*` 子 Skill 路由改为单 Agent 内部阶段路由。
- 使用 `@brief`、`@product_reference`、`@character_reference`、`@scene_reference` 等节点引用保持上下文和素材连续性。
- 将“工具可用时生成图片/视频”改成“只调用当前已连接的 Lumina 组件”，避免虚构执行结果。
- 为图像生成、视频生成、多模态复核、图片合成、预览和保存提供了可直接复制的 Tool Description。
- 内置 10 项 Required Brief Gate、创意提案八段结构、导演视觉开发、PPM、分镜四步法、图片 Prompt 八层结构、Seedance-style 视频 Prompt、连续性管理、客户反馈翻译和全局质量门槛。
- 在没有电子表格组件时输出 Markdown/TSV；有对应组件时再执行工作簿导出。

### 原创实现重点

本仓库的原创实现重点不是再做一个通用内容机器人，而是赋予 Lumina Agent 一套“导演总控”职责：

- **先判断，再制作**：双层路由门槛阻止 Agent 在关键信息缺失时盲目生成。
- **一个 Agent，九种岗位能力**：策略、创意、导演、制片、分镜、Prompt、AI 制作、剪辑和审片共享同一项目上下文。
- **画布素材有明确职责**：每个 `@` 引用都声明控制范围，降低人物、包装、Logo、场景和风格漂移。
- **生成与验收形成闭环**：每次媒体生成都有调用前条件、结果复核、拒绝理由和重生成策略。
- **从创意语言落到生产语言**：把“高级一点”“更有冲击”等模糊表达翻译成镜头、光影、节奏、声音和 Prompt 动作。
- **能力缺失时安全降级**：未连接组件时交付可执行 Prompt 与清单，不虚构已生成或已保存的结果。

### 核心能力

| 阶段 | 能力 | 主要产物 |
| --- | --- | --- |
| Brief | 诊断信息缺口，先确认画幅并守住前置门槛 | 策略 Brief、待确认项、创意领地 |
| Concept | 解释为什么这样拍，而不是提前堆镜头 | 客户提案、洞察、Creative Idea、Story Outline、KV |
| Treatment | 把批准的创意变成导演语言和视觉证据 | 导演阐述、mood frame、人物/场景/产品设定 |
| PPM | 把创意和执行选择变成可开会确认的制作包 | PPM、物料、风险、客户确认项 |
| Storyboard | 用导演级方法拆解情绪、叙事和镜头 | 精确时间码分镜表、高潮与结尾记忆点 |
| AI Prompt | 将每镜转换为可执行的图像和视频生成指令 | 图片 Prompt、Seedance-style 视频 Prompt、负面约束 |
| Production | 管理参考素材、批次、版本、连续性和返工 | 镜头状态表、版本规则、重生成策略 |
| Editing | 规划前 3 秒、节奏、音乐、字幕、packshot 和版本 | 剪辑时间线、声音设计、交付清单 |
| Review | 把“不够高级”等模糊反馈翻译成制作动作 | 修改矩阵、优先级、客户回复建议 |

### 最重要的门槛

当用户只提供产品图、产品名、粗略品类或一句需求时，Agent 不会直接生成创意、分镜或视频。它必须先确认：

1. 横屏、竖屏，还是横竖都要。
2. 投放渠道。
3. 成片时长。
4. 参考风格。
5. 已有表达内容。
6. 品牌 Slogan。
7. 禁用词。
8. 目标人群。
9. 是否允许真人演员。
10. 是否承担电商转化目标。

只有这些问题得到回答，或用户明确接受已列出的默认值后，Agent 才会继续下游工作。

### 在 Lumina 中使用

1. 在 Lumina Canvas 中创建一个 Agent 节点。
2. 将 [`lumina-tvc-ai-director/SKILL.md`](lumina-tvc-ai-director/SKILL.md) 全文粘贴到 Agent 的 **System Instructions**。
3. 将具体项目需求写入 **Task Prompt**；也可以复制 [`task-prompt-template.md`](lumina-tvc-ai-director/assets/templates/task-prompt-template.md)，再用 `@` 绑定画布上的输入节点。
4. 按需要连接文本生成、图像生成、视频生成、多模态理解、图片合成、预览与保存组件。
5. 参考 [`lumina-canvas-setup.md`](lumina-tvc-ai-director/references/lumina-canvas-setup.md) 配置节点命名、Tool Description、验收和测试。

### Lumina 导入格式规范

为避免 “Some documents need to be processed / documents have not been uploaded” 报错，请遵守：

- 只导入 `lumina-tvc-ai-director` 文件夹，不要直接上传包含开发文件的本地 Git 仓库目录。
- Skill 包内只放 `.md`、`.txt`、`.json`、`.yaml` 或 `.yml` 文件。
- 不要放入 `.skillignore`、`.gitignore`、`.DS_Store`、无扩展名文件、脚本、图片、ZIP 或其他格式。
- 文件与文件夹名只能使用英文字母、数字、下划线 `_` 和连字符 `-`，长度不超过 64 个字符。
- 扩展名必须是小写；入口文件必须准确命名为 `SKILL.md`。
- `SKILL.md` 必须位于导入文件夹根目录，并且保留开头的 YAML frontmatter。

本仓库的 `lumina-tvc-ai-director` 文件夹已按以上规则整理，内部只有受支持的 `.md` 文件。截图所示的 `.skillignore` 不属于本 Skill，也不应加入导入包。

最小可用拓扑：

```text
Text / Image / Video / Audio inputs
                │
                ▼
      Lumina TVC AI Director Agent
        ├── Show Text
        ├── Image Generation ── Preview / Save Image
        ├── Video Generation ── Preview / Save Video
        ├── Multimodal Review
        └── Composite Image
```

没有连接媒体生成组件时，Skill 仍可完成 Brief、策略、创意、导演阐述、PPM、分镜、Prompt、制作统筹、剪辑和审片分析，但只会输出执行所需的文字、表格和 Prompt，不会声称已经生成媒体。

### 示例 Task Prompt

```text
请读取 @brief、@brand_guide 和 @product_reference。
先判断信息是否通过 Brief Gate；如果不足，只提出必须确认的问题。
如果完整，请生成一条 15 秒 9:16 产品 TVC 的创意路线、导演阐述和带时间码分镜，
再为每个镜头生成图片 Prompt 与 Seedance-style 视频 Prompt。
只有在引用素材和 Prompt 都完整时才调用已连接的生成组件；每次生成后检查包装、Logo、人物、手部和连续性。
```

### 文件结构

```text
.
├── README.md
├── LICENSE.md
└── lumina-tvc-ai-director/
    ├── SKILL.md
    ├── assets/templates/
    │   ├── task-prompt-template.md
    │   └── tvc-output-templates.md
    └── references/
        └── lumina-canvas-setup.md
```

### 设计边界

- 本仓库提供的是 Agent System Instructions、画布连接建议和工作流方法，不包含可直接导入 Lumina 的私有画布文件。
- Lumina 中实际可用的模型、参数和组件以你当前账号与画布界面为准；Skill 不硬编码会随产品更新变化的模型名称。
- 品牌声明、法律、版权、肖像、产品包装和客户审批仍需由项目负责人确认。

## English Introduction

### What it is

This is not a one-shot “write me an ad prompt” preset. It is a TVC production control system designed to run as the central Agent in a Lumina Canvas workflow.

One central Agent coordinates nine specialist operating modes:

```text
brief -> concept -> treatment -> ppm -> storyboard
      -> ai prompt -> ai production -> editing -> review
```

The Agent identifies the current production stage, checks whether critical inputs are missing, and then decides whether to develop strategy, call connected image/video components, organize generation batches, or interpret client feedback. A shared handoff format keeps context intact across every stage.

### Lumina-native design

- Uses internal stage routing suitable for a single Lumina Agent.
- Uses `@` references for briefs, products, characters, scenes, style frames, videos, and audio on the canvas.
- Calls only components actually connected to the Agent and never claims that media was generated or saved when the capability is absent.
- Includes copy-ready tool descriptions for image generation, video generation, multimodal review, image composition, preview, and save components.
- Includes a ten-question brief gate, eight-section proposal logic, visual treatment process, PPM coverage, four-step storyboard method, eight-layer image prompting, Seedance-style video prompting, continuity management, feedback translation, and final quality checks.
- Falls back to Markdown tables or TSV when no spreadsheet component is available.

### Original contribution

The original implementation in this repository gives a Lumina Agent a focused **director-control** role instead of turning it into another generic content assistant:

- **Decide before producing:** two routing gates prevent premature generation when critical inputs are missing.
- **One Agent, nine disciplines:** strategy, creative, directing, production, storyboarding, prompting, AI generation, editing, and review share the same project context.
- **Explicit canvas-reference contracts:** every `@` reference declares what it controls, reducing drift in characters, packaging, logos, locations, and style.
- **Closed-loop generation QA:** every media call has preconditions, acceptance checks, rejection reasons, and regeneration guidance.
- **Creative language becomes production action:** vague notes such as “more premium” or “more impactful” are translated into camera, lighting, pacing, sound, and prompt changes.
- **Safe capability fallback:** when a component is missing, the Agent returns executable prompts and action lists instead of fabricating tool results.

### Capabilities

| Stage | What the Agent does | Primary output |
| --- | --- | --- |
| Brief | Diagnoses missing inputs and confirms aspect ratio first | Strategic brief, open questions, creative territories |
| Concept | Explains why the film should be made this way | Client proposal, insight, idea, story outline, KV direction |
| Treatment | Turns an approved idea into directing choices and visual evidence | Director treatment, mood frames, character/scene/product studies |
| PPM | Converts approved choices into a production-ready meeting pack | PPM, materials, risks, approval items |
| Storyboard | Designs emotion, narrative, visual language, and exact timing | Timecoded shot table, climax, closing memory beat |
| AI Prompt | Converts each shot into executable image and video instructions | Image prompts, Seedance-style video prompts, negative constraints |
| Production | Manages references, batches, versions, continuity, and regeneration | Shot status table, version rules, retry strategy |
| Editing | Plans the opening hook, pacing, music, supers, packshot, and versions | Edit timeline, sound plan, delivery checklist |
| Review | Translates vague client comments into craft decisions | Revision matrix, priorities, client reply language |

### Use it in Lumina

1. Create an Agent node in Lumina Canvas.
2. Paste the full contents of [`lumina-tvc-ai-director/SKILL.md`](lumina-tvc-ai-director/SKILL.md) into **System Instructions**.
3. Put the current project request in **Task Prompt**, or copy [`task-prompt-template.md`](lumina-tvc-ai-director/assets/templates/task-prompt-template.md) and bind canvas inputs with `@` references.
4. Connect the text, image, video, multimodal, composition, preview, and save components your workflow actually needs.
5. Follow [`lumina-canvas-setup.md`](lumina-tvc-ai-director/references/lumina-canvas-setup.md) for node aliases, tool descriptions, acceptance checks, and test cases.

### Lumina import-safe format

To avoid “Some documents need to be processed / documents have not been uploaded” errors:

- Import the `lumina-tvc-ai-director` folder only; do not upload a local Git working directory containing development artifacts.
- Keep only `.md`, `.txt`, `.json`, `.yaml`, or `.yml` files inside the skill package.
- Do not include `.skillignore`, `.gitignore`, `.DS_Store`, extensionless files, scripts, images, ZIP files, or other unsupported formats.
- Use only letters, numbers, underscores, and hyphens in file and folder names, with a maximum length of 64 characters.
- Keep extensions lowercase. The entry file must be named exactly `SKILL.md`.
- Place `SKILL.md` at the root of the imported folder and preserve its YAML frontmatter.

The `lumina-tvc-ai-director` folder in this repository already follows these rules and contains only supported Markdown files. The `.skillignore` shown in the error screenshot is not part of this Skill and should not be added to the import package.

Without connected media-generation components, the Agent can still deliver strategy, proposals, treatments, PPMs, storyboards, prompts, production plans, edit plans, and review analysis. It will return executable text and tables instead of pretending to have generated media.

### Scope and limitations

- The repository provides Agent System Instructions and canvas setup guidance; it does not contain a private Lumina canvas export.
- Available models, parameters, and components depend on the current Lumina account and UI. The Skill intentionally avoids hard-coding volatile model names.
- Brand claims, legal clearance, copyright, likeness rights, packaging accuracy, and client approvals remain human responsibilities.

## Lumina references

- Lumina introduction: [BytePlus documentation](https://docs.byteplus.com/en/docs/seedream/lumina-introduction-page)
- Lumina Canvas guide: [BytePlus documentation](https://docs.byteplus.com/en/docs/seedream/lumina-canvas-user-guide)

## License

MIT. See [LICENSE.md](LICENSE.md).
