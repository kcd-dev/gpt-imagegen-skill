# gpt-imagegen-skill

> 一个可开源复用的 Codex / Claude Code / 通用代理图片生成 Skill：默认走内建生图能力，必要时降级到 OpenAI 兼容 CLI，支持 `gpt-image-2`、编辑、多图批量、以及透明背景后处理。

[![License: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-green.svg)](./LICENSE)
![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)
![Skill](https://img.shields.io/badge/type-imagegen--skill-purple.svg)

[中文](./README.md) | [README_ZH](./README_ZH.md) | [English](./README_EN.md)

## ✨ 它是什么

`gpt-imagegen-skill` 是从实际可用的 `imagegen` system skill 中提炼出来的开源版本，目标不是“再造一个图像 SDK”，而是把**代理如何正确调用图像模型**这层经验沉淀下来：

- 默认优先使用代理自带的原生生图能力
- 用户明确要求 CLI / API / 模型参数时，再切到 `scripts/image_gen.py`
- 默认模型为 `gpt-image-2`
- 透明背景需求优先走色键抠图；真正原生透明再显式切到 `gpt-image-1.5`
- 支持 `generate` / `edit` / `generate-batch`
- 兼容 `OPENAI_BASE_URL`，可接 OpenAI 兼容网关

## ✅ 适合谁

- 想给 Codex / Claude Code / 自定义 agent 增加稳定图片生成能力的人
- 想复用一套现成 prompt 规范、CLI、参考文档的人
- 想在 OpenAI 官方与 OpenAI 兼容网关之间切换的人
- 想要一个“能直接落地”的 Skill，而不是只有几段 prompt 模板的人

## ❌ 不适合谁

- 只想直接调单个 HTTP 接口，不需要 Skill / 工作流封装的人
- 主要做 SVG / Icon / 前端原生矢量图，而不是位图生图的人
- 想把这个仓库当成完整 Web 应用或图床服务的人

## ✨ 核心特性

- 🧭 **双模式设计**：默认内建工具模式 + 显式 CLI fallback 模式
- 🖼️ **生成/编辑/批量**：覆盖常见图片工作流
- 🔍 **透明背景守卫**：区分色键抠图与模型原生透明
- 🔌 **OpenAI 兼容**：支持 `OPENAI_API_KEY` + `OPENAI_BASE_URL`
- 📚 **文档完整**：包含架构、API 速查、prompt 参考、维护指南
- 🧪 **可验证**：CLI 支持 dry-run，便于本地验证命令和参数

## 📁 项目结构

```text
gpt-imagegen-skill/
├── SKILL.md
├── scripts/
│   ├── image_gen.py
│   └── remove_chroma_key.py
├── references/
│   ├── cli.md
│   ├── image-api.md
│   ├── prompting.md
│   ├── sample-prompts.md
│   └── codex-network.md
├── agents/
│   └── openai.yaml
├── assets/
├── ARCHITECTURE.md
├── USAGE_AND_MAINTENANCE.md
├── CONTRIBUTING.md
├── CHANGELOG.md
└── TODOS.md
```

## 🚀 快速开始

### 1）直接作为 Skill 使用

把本仓库放到你的 Skill 根目录后，让代理读取 `SKILL.md` 即可。

### 2）只用 CLI

```bash
export CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"
export IMAGE_GEN="$PWD/scripts/image_gen.py"
export OPENAI_API_KEY="<your_key>"
# 可选：接 OpenAI 兼容网关
export OPENAI_BASE_URL="https://sub-lb.tap365.org/v1"
```

安装依赖：

```bash
uv pip install openai pillow
```

Dry-run：

```bash
python "$IMAGE_GEN" generate \
  --prompt "White background, simple blue letter O icon, clean vector style" \
  --size 1024x1024 \
  --out output/imagegen/demo.png \
  --dry-run
```

真实生成：

```bash
python "$IMAGE_GEN" generate \
  --prompt "A polished landing-page hero image of a matte ceramic mug on a stone surface" \
  --size 1024x1024 \
  --quality high \
  --out output/imagegen/mug-hero.png
```

### 3）透明背景后处理

```bash
python scripts/remove_chroma_key.py \
  --input output/imagegen/source.png \
  --out output/imagegen/final-transparent.png \
  --auto-key border \
  --soft-matte \
  --transparent-threshold 12 \
  --opaque-threshold 220 \
  --despill
```

## ⚙️ 配置

### 必需环境变量

- `OPENAI_API_KEY`：真实 API 调用必需

### 可选环境变量

- `OPENAI_BASE_URL`：接入 OpenAI 兼容网关时设置
- `CODEX_HOME`：若你要复用默认 Codex Skill 路径时可设置

## 📚 详细文档

- [ARCHITECTURE.md](./ARCHITECTURE.md)
- [USAGE_AND_MAINTENANCE.md](./USAGE_AND_MAINTENANCE.md)
- [references/cli.md](./references/cli.md)
- [references/image-api.md](./references/image-api.md)
- [references/prompting.md](./references/prompting.md)
- [references/sample-prompts.md](./references/sample-prompts.md)

## 🛠️ 开发

### 本地检查

```bash
python3 -m py_compile scripts/image_gen.py scripts/remove_chroma_key.py
python3 scripts/image_gen.py generate --prompt "test" --out output/imagegen/test.png --dry-run
python3 scripts/remove_chroma_key.py --help
```

### CI 目标

- 脚本可编译
- CLI 可进入 dry-run
- 文档结构齐全

## 🤝 贡献

欢迎提交 Issue / PR。开始前请先阅读 [CONTRIBUTING.md](./CONTRIBUTING.md)。

## ❓ FAQ

### 1. 它和直接调用 OpenAI Image API 有什么区别？
它多了一层**代理工作流约束**：什么时候用内建工具，什么时候切 CLI，什么时候不能静默降级模型，什么时候要做透明背景后处理。

### 2. 为什么默认是 `gpt-image-2`，但透明背景又提到 `gpt-image-1.5`？
因为 `gpt-image-2` 适合作为默认高质量模型，但原生透明背景并不是它的默认能力路径；所以 true transparency 被明确放进 fallback 流程里。

### 3. 可以接第三方 OpenAI 兼容平台吗？
可以。只要平台兼容标准 Images API，并通过 `OPENAI_BASE_URL` 指向对应入口即可。

### 4. 这个仓库会不会暴露你的私有 key？
不会。仓库只包含 Skill、脚本和文档，不包含任何真实密钥。

### 5. 什么时候不该用这个 Skill？
当需求本质上是 SVG、Logo 系统、前端原生图形、或完全自定义业务后端时，不建议强行套这套生图 Skill。

## ⭐ Star History

[![Star History Chart](https://api.star-history.com/svg?repos=kcd-dev/gpt-imagegen-skill&type=Date)](https://www.star-history.com/#kcd-dev/gpt-imagegen-skill&Date)

> 说明：GitHub 远端仓库名与本地工作目录现已统一为 `gpt-imagegen-skill`。

## 📄 许可证

Apache-2.0，详见 [LICENSE](./LICENSE)。
