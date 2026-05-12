# Contributing

欢迎为 `gpt-imagegen-skill` 提交改进。

## 开始前

1. 先阅读 `README.md`、`SKILL.md`、`ARCHITECTURE.md`
2. 确认你的改动不会泄露真实密钥、网关地址凭证或私有运行态
3. 若改动影响 CLI 行为，请同步更新 `references/cli.md` 与 `references/image-api.md`

## 推荐流程

1. Fork 或新分支开发
2. 保持单一主题提交
3. 本地运行：

```bash
python3 -m py_compile scripts/image_gen.py scripts/remove_chroma_key.py
python3 scripts/image_gen.py generate --prompt "test" --out output/imagegen/test.png --dry-run
```

4. 更新文档
5. 提交 PR，说明：
   - 改动背景
   - 改动范围
   - 验证方式
   - 是否影响现有 Skill 用户

## 可以贡献的方向

- 更好的 prompt 参考
- 更多 OpenAI 兼容平台验证经验
- 更清晰的透明背景后处理参数建议
- 文档翻译和示例增强
- CI / packaging 优化

## 不建议的改动

- 静默改变默认模型或 fallback 逻辑
- 提交任何真实 API key / token / cookie
- 把本仓库改造成与 Skill 无关的大而全应用
