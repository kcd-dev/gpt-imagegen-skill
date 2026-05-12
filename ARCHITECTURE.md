# gpt-imagegen-skill 架构说明

## 测试日期
2026-05-13

## 文档版本
v1.0.0

## 整体架构

```text
┌──────────────────────────────┐
│ User / Agent Request         │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ SKILL.md decision policy     │
│ - built-in first             │
│ - explicit CLI fallback      │
│ - transparency guardrails    │
└──────────────┬───────────────┘
               │
      ┌────────┴────────┐
      │                 │
      ▼                 ▼
┌───────────────┐  ┌──────────────────┐
│ Built-in tool │  │ scripts/image_   │
│ path          │  │ gen.py CLI       │
└──────┬────────┘  └─────────┬────────┘
       │                      │
       │                      ▼
       │             ┌──────────────────┐
       │             │ OpenAI SDK /     │
       │             │ compatible API   │
       │             └─────────┬────────┘
       │                       │
       ▼                       ▼
┌──────────────────┐   ┌──────────────────┐
│ Generated image  │   │ Generated /      │
│ under Codex path │   │ edited outputs   │
└─────────┬────────┘   └─────────┬────────┘
          │                      │
          └──────────┬───────────┘
                     ▼
          ┌──────────────────────┐
          │ remove_chroma_key.py │
          │ optional postprocess │
          └──────────────────────┘
```

## 核心模块

### 1. `SKILL.md`
- 职责：定义何时使用内建工具、何时进入 CLI fallback、何时禁止静默降级模型。
- 依赖：references 文档、脚本路径、代理运行时。
- 输入/输出：输入为用户图片任务；输出为执行策略与约束。

### 2. `scripts/image_gen.py`
- 职责：显式 CLI 模式下的图片生成 / 编辑 / 批量生成。
- 依赖：`openai` Python SDK、`OPENAI_API_KEY`、可选 `OPENAI_BASE_URL`。
- 输入/输出：输入 prompt / 图像 / 参数；输出图片文件到 `output/imagegen/`。

### 3. `scripts/remove_chroma_key.py`
- 职责：对平面色键背景做 alpha 抠图。
- 依赖：Pillow。
- 输入/输出：输入一张纯色背景位图；输出带 alpha 的 PNG / WebP。

### 4. `references/*`
- 职责：保存 prompt 规范、API 参数参考、CLI 使用说明、网络注意事项。
- 依赖：与脚本、Skill 同步维护。
- 输入/输出：输入为维护者知识；输出为可读文档。

### 5. `agents/openai.yaml`
- 职责：为 agent 平台提供可复用的配置入口。
- 依赖：具体 agent runtime。
- 输入/输出：输入为平台配置；输出为 agent 可挂载的 skill 能力。

## 数据流

1. 用户发起图片任务。
2. `SKILL.md` 先判断：内建模式还是 CLI fallback。
3. 若走内建模式：由代理自身工具执行生图。
4. 若走 CLI fallback：`image_gen.py` 通过 OpenAI SDK 请求图像接口。
5. 若目标是透明背景：先走色键图，再由 `remove_chroma_key.py` 做本地抠图；复杂场景才转为显式 `gpt-image-1.5` 透明 fallback。
6. 最终产物写入项目工作区或默认输出目录。

## 关键设计决策

### 决策 1：built-in first
原因：对普通代理用户最省心，不要求额外配置 key。

### 决策 2：CLI 只在显式场景下使用
原因：CLI 暴露更多参数，也带来更多失败面；因此只在用户明确需要模型/API/透明控制时进入。

### 决策 3：透明背景分成两级路径
原因：`gpt-image-2` 默认质量好，但不应被误用为“天然支持 true transparency”；因此先色键、后原生透明 fallback。

## 扩展与维护

- 若未来新增模型，优先在 `references/image-api.md` 和 `scripts/image_gen.py` 中同步约束。
- 若未来新增 agent 平台适配，优先新增 `agents/*.yaml` 而不是改坏现有 skill 主路径。
- 若未来新增后处理器，应保持“脚本职责单一”，不要把太多图像处理逻辑塞进 `image_gen.py`。
