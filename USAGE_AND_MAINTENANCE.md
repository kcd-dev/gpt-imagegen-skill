# gpt-imagegen-skill 使用与维护指南

## 测试日期
2026-05-13

## 文档版本
v1.0.0

## 1. 目标

本仓库用于把 `imagegen` system skill 以开源形态交付，供 Codex、Claude Code 或其他 agent runtime 复用。

## 2. 依赖要求

- Python 3.10+
- `openai`
- `pillow`（透明背景后处理需要）
- 有效的 `OPENAI_API_KEY`
- 若接第三方兼容平台，需要 `OPENAI_BASE_URL`

安装：

```bash
uv pip install openai pillow
```

## 3. 使用方式

### 3.1 作为 Skill 使用

1. 让 agent 读取 `SKILL.md`
2. 普通生图请求默认使用内建图像工具
3. 用户显式要求 CLI / API / 模型控制时，按 `references/cli.md` 执行

### 3.2 作为 CLI 使用

#### 生成

```bash
python scripts/image_gen.py generate \
  --prompt "A minimal product shot of a ceramic mug" \
  --size 1024x1024 \
  --out output/imagegen/mug.png
```

#### 编辑

```bash
python scripts/image_gen.py edit \
  --image input.png \
  --prompt "Change only the background to warm sunset tones" \
  --out output/imagegen/mug-edit.png
```

#### 批量

```bash
python scripts/image_gen.py generate-batch \
  --input tmp/imagegen/prompts.jsonl \
  --out-dir output/imagegen/batch
```

## 4. 常见场景

### 场景 A：普通单图生成
- 推荐：默认 `gpt-image-2`
- 典型参数：`--size 1024x1024 --quality high`

### 场景 B：多图批量出稿
- 推荐：`generate-batch`
- 注意：每个 distinct asset 应单独一条 prompt，而不是靠 `n` 代替

### 场景 C：透明背景商品图
- 默认：先生成纯色背景图
- 后处理：`remove_chroma_key.py`
- 若用户明确要求 native transparency：再显式切 `gpt-image-1.5`

### 场景 D：接第三方 OpenAI 兼容网关
- 设置 `OPENAI_BASE_URL`
- 先用 dry-run 检查参数
- 再用真实请求验证延迟、错误码和产物格式

## 5. 维护原则

### 5.1 不要静默降级模型
- 不要把 built-in 或 `gpt-image-2` 悄悄改成 `gpt-image-1.5`
- 如果要切换，是显式产品决策，不是隐藏 fallback

### 5.2 不要把仓库变成私有环境快照
- 不提交真实 key
- 不提交私有 gateway 域名凭证
- 不在 README 写死个人机器路径

### 5.3 保持文档和脚本同步
以下任一改动，都必须同步更新：
- 默认模型改动
- 透明背景策略改动
- CLI 参数新增/删除
- 输出目录约定改动

对应应回写的文件：
- `SKILL.md`
- `references/cli.md`
- `references/image-api.md`
- `CHANGELOG.md`

## 6. 验收建议

最低验收：

```bash
python3 -m py_compile scripts/image_gen.py scripts/remove_chroma_key.py
python3 scripts/image_gen.py generate --prompt "test" --out output/imagegen/test.png --dry-run
python3 scripts/remove_chroma_key.py --help
```

真实业务验收建议至少覆盖：
- 一次 `generate`
- 若仓库声称支持透明图，再跑一次真实抠图或真实透明 fallback
- 若仓库声称支持兼容网关，再跑一次非官方 `OPENAI_BASE_URL`

## 7. 故障排查

### `OPENAI_API_KEY is not set`
设置环境变量后重试。

### `ModuleNotFoundError: openai`
安装依赖：

```bash
uv pip install openai
```

### `Pillow is required`
安装依赖：

```bash
uv pip install pillow
```

### 输出文件已存在
默认不会覆盖，显式使用 `--force`。

### 兼容网关返回 502 / 503 / timeout
先区分：
- 是网关问题
- 是上游模型问题
- 是客户端超时过短

不要把单次失败直接判成“模型不可用”。

## 8. 发布维护建议

- 每次 release 前至少跑一轮 dry-run
- 若修改默认行为，更新 README 与 CHANGELOG
- 若修改对外口径，优先保证 README 中文首页、README_EN、README_ZH 一致
