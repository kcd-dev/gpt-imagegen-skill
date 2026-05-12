# gpt-imagegen-skill

> An open-source Codex / Claude Code / agent-ready image generation skill that prefers the built-in image tool first and falls back to an OpenAI-compatible CLI only when explicitly needed.

[![License: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-green.svg)](./LICENSE)
![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)
![Skill](https://img.shields.io/badge/type-imagegen--skill-purple.svg)

[中文](./README.md) | [README_ZH](./README_ZH.md) | [English](./README_EN.md)

## What it is

`gpt-imagegen-skill` is an open-source extraction of a production-usable system skill for image generation. The goal is not to wrap yet another SDK, but to preserve the practical execution rules an agent needs:

- prefer the built-in image generation tool by default
- switch to `scripts/image_gen.py` only when the user explicitly asks for CLI / API / model-level controls
- default to `gpt-image-2`
- use chroma-key removal first for transparent output, and only use `gpt-image-1.5` for true native transparency when explicitly confirmed
- support `generate`, `edit`, and `generate-batch`
- support `OPENAI_BASE_URL` for OpenAI-compatible gateways

## Core features

- Dual-mode workflow: built-in tool first, explicit CLI fallback second
- Generation, editing, and batch generation
- Transparent-background guardrails
- OpenAI-compatible gateway support
- Reusable prompt references and maintenance docs
- Dry-run capable CLI for local verification

## Quick start

```bash
export IMAGE_GEN="$PWD/scripts/image_gen.py"
export OPENAI_API_KEY="<your_key>"
export OPENAI_BASE_URL="https://api.openai.com/v1"  # optional
uv pip install openai pillow

python "$IMAGE_GEN" generate \
  --prompt "A cozy alpine cabin at dawn" \
  --size 1024x1024 \
  --out output/imagegen/alpine-cabin.png
```

## Documentation

- [ARCHITECTURE.md](./ARCHITECTURE.md)
- [USAGE_AND_MAINTENANCE.md](./USAGE_AND_MAINTENANCE.md)
- [CONTRIBUTING.md](./CONTRIBUTING.md)
- [references/cli.md](./references/cli.md)
- [references/image-api.md](./references/image-api.md)

## FAQ

### When should I use this instead of calling the Image API directly?
Use this when you want reusable agent behavior and execution policy, not just a raw HTTP request.

### Does it work with third-party OpenAI-compatible gateways?
Yes. Set `OPENAI_BASE_URL` to your gateway base URL.

### Does this repository contain any private API keys?
No. It ships only the skill, scripts, and documentation.

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=kcd-dev/gpt-imagegen-skill&type=Date)](https://www.star-history.com/#kcd-dev/gpt-imagegen-skill&Date)

## License

Apache-2.0. See [LICENSE](./LICENSE).
