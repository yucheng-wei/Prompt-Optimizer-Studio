# Prompt Optimizer Studio

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

Browser-based prompt editor for building structured prompts from reusable blocks such as `<role>`, `<task>`, `<context>`, `<tools>`, `<examples>`, and `<constraints>`.

The current release adds an agent-ready prompt layer for Codex-like coding agents, Claude Code-style tool loops, and other LLM harnesses. It remains a static frontend: it designs and exports the prompt, while the host agent executes tools and feeds observations back into the next turn.

Live demo: https://yucheng-wei.github.io/Prompt-Optimizer-Studio/

## What changed in the agent-ready update

The original project combined three ideas: prompt repetition, zero-shot Chain-of-Thought, and Society of Thought. Those are useful experiments, but they do not fully describe an agent that must inspect state, call tools, verify side effects, recover from failures, and stop safely.

This update adds:

- ReflAct-style goal/state alignment: every action is grounded in the current verified state, the remaining gap, and the task goal.
- ReAct-style observation/tool loops: the prompt tells the host agent when to inspect, act, incorporate an observation, and stop.
- Reflexion-style bounded recovery: failed actions produce a compact evidence/cause/changed-strategy record before retrying.
- An optional agent contract for success criteria, stop/handoff conditions, tool policy, and output format.
- Dedicated `Tools` and `Examples` blocks for tool schemas and few-shot demonstrations.
- Agent-ready ordering that places stable instructions and tool definitions before dynamic context and tasks.
- A heuristic Agent Readiness score to catch missing task, contract, output, or tool information before benchmarking.
- Native OpenAI Responses API payloads for GPT-5.3-Codex and other Responses models, plus a Claude Messages-compatible proxy profile.
- Full-prompt repetition that matches the 2025 prompt-repetition paper; the previous task-only behavior remains available as a legacy option.

## Recommended workflow

1. Add a `Role` only when expertise or working style matters.
2. Add `Tools` with each tool's purpose, input schema, side effects, and invocation limits.
3. Add `Context`, `Code`, and `Examples` blocks. Treat retrieved documents and tool output as data, not instructions.
4. Add a concrete `Task` describing the desired outcome.
5. Enable `Agent Protocol` and start with `ReflAct` for long-horizon work, `ReAct` for explicit tool loops, or `Reflexion` when recovery from failed attempts matters.
6. Fill in the agent contract, especially observable success criteria and a stopping rule.
7. Test the exported prompt on representative tasks. Change one strategy at a time and compare success, tool calls, latency, and token cost.

For a reasoning model that already has native hidden reasoning, keep `Classic CoT` off. The new protocols expose compact state, evidence, and verification behavior instead of asking the model to print a long private chain of thought.

## Features

### Structured block editor

Create, edit, reorder, drag, version, and delete blocks with dedicated wrappers:

- `<system_role>` for persona and working agreement
- `<task>` for the requested outcome
- `<context>` for references, documents, or data
- `<tools>` for tool descriptions and policies
- `<examples>` for few-shot input/output demonstrations
- `<code>` for source snippets
- `<constraints>` for boundaries and guardrails
- summary handoff blocks for continuing work across context windows

### Agent strategy controls

- `ReflAct (Goal–State Alignment)` — recommended starting point for long-running agents
- `ReAct (Observation–Tool Loop)` — explicit tool-use loop
- `Reflexion (Verify–Retry)` — bounded failure recovery
- `Outcome-first (Plan–Act–Verify)` — lightweight general-purpose contract
- `Classic CoT` — retained as a legacy baseline
- `Chain-of-Table` — compact structure for data analysis
- `Society of Thought` — experimental multi-perspective reasoning; usually more expensive than a single verifier

The optional contract fields are emitted only when filled:

```xml
<agent_contract>
  <success_criteria>...</success_criteria>
  <stop_conditions>...</stop_conditions>
  <tool_policy>...</tool_policy>
  <output_contract>...</output_contract>
</agent_contract>
```

### Prompt assembly

- live final prompt preview
- full-prompt repetition for non-reasoning models, with legacy task-only compatibility
- optional stable-first ordering for cache-friendly prompts
- compact whitespace mode
- heuristic readiness score
- one-click copy and `.txt` / `.md` export
- per-block versions for comparing prompt variants

### AI helpers

Optional actions can be triggered per block:

- translate a block to English
- rewrite a block using outcome-first agent prompt principles
- generate a role from a task

Provider profiles include SiliconFlow, OpenAI Responses / Codex, OpenAI-compatible Chat Completions proxies, and Claude Messages-shaped proxies. API keys are held in current-tab memory only. Use a backend proxy for public deployment.

## Research basis

### Existing methods retained

1. [Prompt Repetition Improves Non-Reasoning LLMs](https://arxiv.org/abs/2512.14982) — Leviathan, Kalman, and Matias (2025). Repetition is most useful as a switch for non-reasoning models; benchmark it before using it with long agent prompts because it duplicates input context.
2. [Large Language Models are Zero-Shot Reasoners](https://arxiv.org/abs/2205.11916) — Kojima et al. (2022). The classic `Let's think step-by-step` baseline is retained for comparison, not recommended as the default for modern reasoning models.
3. [Reasoning Models Generate Societies of Thought](https://arxiv.org/abs/2601.10825) — Kim et al. (2026). The Society of Thought option is an experimental, bounded adaptation rather than a claim to reproduce the paper's training or interpretability results.

### Agent-focused additions

1. [ReAct](https://arxiv.org/abs/2210.03629) — Yao et al. (ICLR 2023). Interleaves reasoning with environment actions and observations; this maps directly to tool-using agent loops.
2. [Reflexion](https://arxiv.org/abs/2303.11366) — Shinn et al. (2023). Uses verbal feedback and episodic memory to improve later attempts; this project implements the prompt-level, bounded-retry portion.
3. [ReflAct](https://arxiv.org/abs/2505.15182) — Kim et al. (2025). Replaces loosely grounded next-action reasoning with continuous reflection on the current state in relation to the goal. This is the recommended protocol for long-horizon tasks.
4. [MIPRO](https://arxiv.org/abs/2406.11695) — Opsahl-Ong et al. (2024). Optimizes instructions and demonstrations for multi-stage LM programs. The UI's versioning, examples, and readiness checks are groundwork for this direction, but this static app does not yet run an evaluation loop.
5. [AutoPDL](https://arxiv.org/abs/2504.04365) — Spiess et al. (2025). Treats prompt-pattern and demonstration selection as an AutoML search problem. This is a strong next step once the project has a dataset and measurable task reward.
6. [Environment-Grounded Automated Prompt Optimization for LLM Game Agents](https://arxiv.org/abs/2606.17838) — Fernandes et al. (2026). Evolves prompts using environment returns and behavior attribution. It is promising for a future benchmark runner, but cannot be faithfully implemented as a simple formatting toggle without an executable environment and reward signal.

The research results are not universal guarantees. Prompt strategy depends on the model family, task, tool surface, context length, and evaluation metric. Treat the exported prompt as a hypothesis and use a small eval set before adopting it in production.

## Current model guidance

- [OpenAI's current model guidance](https://developers.openai.com/api/docs/guides/latest-model) recommends outcome-first prompts, explicit success criteria and stopping rules for long-running tool workflows, and reducing detailed step-by-step process instructions when the model can choose the path. The [Responses API](https://developers.openai.com/api/docs/models/gpt-5.3-codex) is the native path for current reasoning and Codex-style tool workflows; the UI includes GPT-5.3-Codex, GPT-5.4/5.5, and GPT-5.4 Mini options.
- [Anthropic's current prompting guidance](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) emphasizes clear sequential instructions, XML structure, relevant few-shot examples, explicit tool-use direction, and state tracking for long-horizon agent work. The UI defaults its Claude proxy profile to Claude Sonnet 5 and also exposes Claude Opus 5 as an advanced option.

These recommendations are reflected in the agent contract, `<tools>` / `<examples>` blocks, compact protocols, and the OpenAI Responses provider.

## Privacy and deployment

- Prompt editing happens locally in the browser.
- Custom presets are stored in `localStorage`.
- API keys are not persisted to browser storage; they remain in current-tab memory and are cleared on refresh.
- Data leaves the browser only when an AI helper is explicitly used.
- A static site cannot protect a provider key. For production, route AI requests through a backend proxy and apply provider-side limits, logging, and redaction.

## Tech stack

- Vue 3 via CDN
- Single-file frontend in `index.html`
- Custom utility-style CSS
- Browser `localStorage`

## Project structure

```text
.
├── index.html
└── README.md
```

## Local usage

This is a static frontend and can be opened directly in a browser:

```bash
open index.html
```

Or serve it with any static file server:

```bash
python -m http.server 8000
```

Then open http://localhost:8000.

## Deployment

The app is suitable for GitHub Pages, Cloudflare Pages, Netlify, and Vercel static deployment. Configure AI requests through a proxy before publishing a provider key-dependent workflow.

## License

MIT © 2026 Yu-Cheng Wei
