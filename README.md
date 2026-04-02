# Prompt Optimizer Studio

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

Browser-based prompt editor for building structured LLM prompts from reusable tagged blocks such as `<role>`, `<task>`, `<context>`, and `<constraints>`.

It is designed for people who want more control than a plain textarea: you can compose prompts block by block, reorder sections, apply reasoning scaffolds, estimate token cost, and export the final result as a clean prompt file.

Live demo: https://yucheng-wei.github.io/Prompt-Optimizer-Studio/

## Why this project exists

Large prompts often become hard to maintain once they mix persona, task instructions, examples, constraints, and code snippets in one long paragraph. This project treats prompts as modular parts, then assembles them into a consistent final output.

That makes it easier to:

- rearrange prompt structure without rewriting everything
- apply formatting strategies consistently
- optimize or translate only one section at a time
- compare different reasoning scaffolds on the same base prompt

## Features

### Structured block editor

Create, edit, reorder, and delete blocks with dedicated wrappers:

- `<role>` for system persona or high-level behavior
- `<task>` for the main instruction
- `<context>` for references, documents, examples, or data
- `<constraints>` for output requirements and guardrails
- `<code_context>` for source code snippets
- plain text blocks when tagging is unnecessary

### Prompt assembly controls

- live final prompt preview
- compact whitespace mode to reduce unnecessary blank lines
- one-click copy
- export as `.txt` or `.md`

### Research-inspired formatting toggles

- Prompt Repetition
  Repeats task blocks at the end of the prompt to reinforce instruction following.
- Chain-of-Thought variants
  Supports classic step-by-step prompting, agentic plan/check/execute scaffolding, and table-first reasoning.
- Society of Thought scaffold
  Adds a multi-perspective reasoning pattern with verifier, strategist, and auditor-style roles inside a thinking scaffold.

### AI helpers

Optional AI actions can be triggered per block:

- translate a block to English
- rewrite a block for clarity and structure
- generate a role block from a task description

The current implementation supports:

- browser-direct SiliconFlow mode
- proxy mode for safer deployment
- local-only mode with AI helpers disabled

## Research basis

This project exposes several prompt-formatting ideas that have been discussed in recent prompting literature:

1. Prompt Repetition  
   Leviathan et al. (2025)  
   https://arxiv.org/abs/2512.14982

2. Zero-shot Chain-of-Thought prompting  
   Kojima et al. (2022)  
   https://arxiv.org/abs/2205.11916

3. Society of Thought  
   Kim et al. (2026)  
   https://arxiv.org/abs/2601.10825

This tool does not claim to reproduce paper results exactly. It provides practical UI switches that let you test related formatting patterns in your own workflow.

## Privacy

- prompt editing happens locally in the browser
- custom presets are stored in `localStorage`
- API keys are not persisted to browser storage in direct mode; they stay in current tab memory only
- data leaves your machine only when you explicitly use an AI helper that sends a request to a model endpoint

If you want production-grade safety, use a backend proxy instead of exposing provider access from the front end.

## Tech stack

- Vue 3 via CDN
- single-file frontend in `index.html`
- custom utility-style CSS
- browser `localStorage`

## Project structure

```text
.
├── index.html
└── README.md
```

## Local usage

This project is a static frontend and can be opened directly in a browser.

```bash
open index.html
```

Or serve it with any static file server:

```bash
python -m http.server 8000
```

Then open `http://localhost:8000`.

## Deployment

The app is suitable for static hosting, including:

- GitHub Pages
- Cloudflare Pages
- Netlify
- Vercel static deployment

For public deployment, proxy AI requests through your own backend if you do not want browser clients to interact with model endpoints directly.

## License

MIT © 2026 Yu-Cheng Wei
