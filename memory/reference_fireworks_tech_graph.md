---
name: fireworks-tech-graph skill
description: SVG+PNG technical diagram skill installed from yizhiyanhua-ai/fireworks-tech-graph; 7 styles, 14 diagram types, AI/Agent patterns; Windows rsvg-convert replaced by Playwright wrapper
type: reference
originSessionId: 85b62cf5-e098-4ce2-b282-fb41489b8bb7
---
## fireworks-tech-graph Skill

- **Source**: `yizhiyanhua-ai/fireworks-tech-graph` (GitHub, 1.1k stars)
- **Install**: `NO_PROXY=github.com npx skills add yizhiyanhua-ai/fireworks-tech-graph --force -g -y`
- **Location**: `~/.claude/skills/fireworks-tech-graph/SKILL.md` (symlink from `~/.agents/skills/`)

### Capabilities

- 7 visual styles: Flat Icon, Dark Terminal, Blueprint, Notion Clean, Glassmorphism, Claude Official, OpenAI Official
- 14 diagram types: Full UML + AI/Agent domain (RAG, Agent Architecture, Memory, Multi-Agent, Tool Call)
- 40+ product icons: OpenAI, Anthropic, Pinecone, Kafka, etc.
- Semantic shapes + arrow colors for domain concepts

### Windows Compatibility

- `rsvg-convert` NOT available on Windows (no native package)
- **SVG generation**: Works perfectly (pure text output)
- **PNG export**: Use `scripts/svg2png.py` (Playwright-based wrapper)
  ```bash
  NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe scripts/svg2png.py input.svg -o output.png -w 1920
  ```
- cairosvg installed but cairocffi cannot find `cairo-2.dll` on Windows (DLL naming mismatch)

### Trigger Phrases

`画图`, `帮我画`, `生成图`, `做个图`, `架构图`, `流程图`, `可视化一下`, `出图`, `generate diagram`, `draw diagram`, `visualize`

### Conflict with Existing Tools

- **Mermaid** (`mmdc`): For inline markdown diagrams, still use mermaid
- **fireworks-tech-graph**: For standalone publication-quality SVG diagrams
- **No conflict**: Different use cases (mermaid = inline docs; fireworks = standalone publication figures)
