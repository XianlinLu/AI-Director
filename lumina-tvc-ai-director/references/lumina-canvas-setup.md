# Lumina Canvas Setup

这份说明用于把 `lumina-tvc-ai-director/SKILL.md` 配置为 Lumina Canvas 的 Agent System Instructions，并为 Agent 连接可用组件。

## 导入包格式

导入时只选择 `lumina-tvc-ai-director` 文件夹，并保持 `SKILL.md` 位于这个文件夹的根目录。

包内规则：

- 允许扩展名：`.md`、`.txt`、`.json`、`.yaml`、`.yml`。
- 扩展名必须小写；唯一例外是入口文件的固定名称 `SKILL.md` 中的大写文件名部分。
- 文件和文件夹名只使用英文字母、数字、下划线和连字符，且不超过 64 个字符。
- 不要加入 `.skillignore`、`.gitignore`、`.DS_Store`、无扩展名的 `LICENSE`、脚本、图片、压缩包或本地 Git 元数据。
- 不要把整个本地 Git 工作目录作为 Skill 文件夹上传。

本 Skill 目录只包含受支持的 Markdown 文件，不需要 `.skillignore`。如果打包工具自动生成 `.skillignore`，请在上传前从待导入文件夹中移除它。

## 推荐画布拓扑

```text
Brief / 品牌规范 / 客户反馈（Text） ─┐
产品 / 人物 / 场景 / 风格（Image） ──┼─> TVC AI Director Agent ─┬─> Show Text
参考片 / 粗剪（Video） ──────────────┤                         ├─> Image Generation ─> Preview / Save Image
音乐 / VO / 同期声（Audio） ─────────┘                         ├─> Video Generation ─> Preview / Save Video
                                                            ├─> Multimodal Review
                                                            └─> Composite Image
```

不要为了凑齐拓扑而连接不存在的组件。最小可用配置是：文本输入、Agent、Show Text。只有连接了生成组件，Agent 才能真正生成媒体。

## Agent 配置

1. 新建或打开 Canvas，放置 Agent 节点。
2. 将 `SKILL.md` 全文粘贴到 **System Instructions**。YAML frontmatter 可保留；它只是说明 Skill 名称和用途。
3. 把具体项目 Brief 放入 **Task Prompt**，或通过 Text/String 节点使用 `@` 引用。
4. 按任务连接工具组件。Tool Description 必须写清参数含义、必填项、允许范围和输出类型。
5. Max Iterations 设为足够完成“判断阶段 → 生成/调用 → 复核 → 输出交接”的步数。若只做 Brief 诊断，不需要为媒体生成预留迭代。
6. 先用一个小 Brief 测试门槛是否生效，再测试单张图、单镜视频和完整多镜头流程。

## 语言行为配置

`SKILL.md` 会在每轮开始时自动确定 `WORKING_LANGUAGE`，不需要为中文、英文或日文建立不同 Agent。Task Prompt 不指定语言时，Agent 使用当前用户请求的主语言；明确指定语言时，以指定语言为准。

语言锁定覆盖可见思考/执行摘要、阶段判断、Brief Gate、工具说明、生成参数中的自然语言、表格、复核、错误、重试和交接。品牌原文、客户原话、节点别名、固定参数和代码不会被误译。

## 输入节点命名

建议使用稳定的英文别名，便于 Prompt 中引用：

| 输入 | 推荐别名 | 说明 |
| --- | --- | --- |
| 客户 Brief | `@brief` | 项目目标、渠道、时长、预算、交付 |
| 品牌规范 | `@brand_guide` | 品牌色、语气、Logo、禁用项 |
| 产品主参考 | `@product_reference` | 包装、Logo、材质、比例的唯一事实源 |
| 人物参考 | `@character_reference` | 脸、发型、妆容、服装、身体比例 |
| 服装参考 | `@wardrobe_reference` | 服装版型、材质、颜色、配饰 |
| 场景参考 | `@scene_reference` | 空间、时间、美术、道具、天气 |
| 风格参考 | `@style_reference` | 只控制摄影气质、光影、色彩、质感 |
| 参考片/粗剪 | `@video_reference` | 镜头节奏、动作或待审内容 |
| 音乐/VO | `@audio_reference` | 节拍、旁白、同期声或音色参考 |
| 客户反馈 | `@client_feedback` | 原始反馈，不要预先改写 |

同类输入有多个时使用编号，如 `@product_reference_01`、`@product_reference_02`。每次引用都说明它控制什么以及不能控制什么。

## Tool Description 模板

### 图像生成

```markdown
用途：根据文本 Prompt 和可选参考图生成一张 TVC 视觉资产。
必填：prompt。
可选：reference_images、aspect_ratio、resolution、seed，以及组件实际暴露的参数。
输入约束：Prompt 必须明确主体、动作、环境、构图、镜头、光线、色板、风格和负面约束；有产品时必须连接产品参考。
输出：单张图片。生成后必须交给预览/保存或多模态复核。
```

### 视频生成

```markdown
用途：根据 Prompt 和可选图片/视频/音频参考生成单个 TVC 镜头。
必填：prompt；时长与画幅必须由 Prompt 或组件参数明确。
可选：reference_media、seed、resolution、camera_fixed、generate_audio、return_last_frame，以及组件实际暴露的参数。
输入约束：一个调用只负责一个可控镜头；复杂镜头需分段描述动作和结束帧。
输出：单段视频；若支持返回尾帧，可将尾帧传给下一镜头维持连续性。
```

### 多模态理解/复核

```markdown
用途：读取图片、视频或音频中的可观察事实，并按指定验收标准检查。
必填：待检查媒体、检查目标。
输出：事实描述、通过/不通过、具体问题、重生成或后期修正建议。
不得推断不可见的品牌事实或把主观风格判断写成客观事实。
```

### 图片合成

```markdown
用途：将已批准的图片和文字组合成提案页、对比板、KV 或产品包装后期合成。
必填：输入图片、画布尺寸/画幅、图层位置或布局规则。
可选：字体、透明度、混合方式、圆角和文字样式。
输出：合成图片。不得用合成组件伪造未批准的 Logo、声明或包装文字。
```

### 保存/预览

```markdown
用途：显示或保存已通过复核的媒体结果。
必填：媒体输入。
命名：项目_镜号_用途_v版本，例如 `TeaLaunch_S03_hero_v02`。
输出：可被下游引用的媒体节点或文件。
```

## 测试用例

### 1. 门槛测试

```text
我要做一条冰红茶广告，只有这张产品图。
```

预期：Agent 只做 Brief 诊断并提出 10 个 Required Brief Gate 问题，不直接生成创意或画面。

### 2. 单镜生成测试

```text
Brief 已确认。请为分镜 03 生成 4 秒 9:16 图生视频，引用 @product_reference 和 @character_reference；只调用一次视频生成，完成后检查包装、脸、手和结束帧。
```

预期：Agent 建立引用职责、生成单镜 Prompt、调用已连接视频组件、复核并输出版本与返工建议。

### 3. 客户反馈测试

```text
请分析 @client_feedback 和 @video_reference。客户说“不够高级，产品也不突出”，先解释真实关注点，再给出修改动作；不要直接重生成。
```

预期：Agent 进入 review 模式，区分内部判断与客户回复，并把反馈翻译为剪辑/Prompt/画面动作。

### 4. 英文语言链测试

```text
Create a 15-second vertical skincare commercial. Start by checking whether the brief is complete, then plan the production workflow. Keep every visible planning step, tool explanation, table, prompt, review note, and handoff in English.
```

预期：除品牌原文、固定参数和节点别名外，画布全部可见过程与交付均为英文，不出现中文阶段标题或中文检查信息。

### 5. 日文语言链测试

```text
20秒の横型飲料CMを企画してください。まずブリーフの不足情報を確認し、その後の制作工程を設計してください。
```

预期：可见思考/执行摘要、提问、组件调用说明、表格标题、Prompt、检查和交接均为自然日文；英文仅用于必要的固定技术词。

## 常见问题

- Agent 跳过提问直接生成：检查 System Instructions 是否完整，Task Prompt 是否写了“可使用默认值”。
- Agent 说已生成但没有媒体：检查生成组件是否真的连接，并在系统指令中保留“不得虚构工具调用结果”。
- 产品包装漂移：把产品图设为唯一 `@product_reference`，在 Tool Description 和每镜 Prompt 中同时锁定包装与 Logo。
- 多镜风格不一致：先生成全片一致性块，再按同场景/同人物/同产品批次生成；可用上一镜尾帧作为下一镜参考。
- Prompt 过长被截断：保留引用、动作链、镜头、结束帧和负面约束，删除重复形容词与不影响画面的背景说明。
