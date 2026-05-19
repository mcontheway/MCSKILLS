---
name: codegraph-context-bootstrap
description: Build repository context using CodeGraph before starting tasks.
---

Use this skill whenever starting a new coding session, exploring an unfamiliar project, or before performing significant changes.

Goal: Establish a concise, graph-backed context for the current task without scanning the entire repository.

Preferred tools:
1. **CodeGraph MCP tools**: `codegraph_status`, `codegraph_context`, `codegraph_search`, `codegraph_files`, `codegraph_explore`. These return structured results that Claude or Codex can interpret directly.
2. **Fallback**: CodeGraph CLI commands: `codegraph status`, `codegraph sync`, `codegraph query`, `codegraph context`, `codegraph files`. Use these when MCP tools are unavailable; be prepared to parse text output.
3. **If neither MCP nor CLI is available**, state that graph-backed context cannot be established and proceed cautiously with minimal file reading.

Workflow:
1. Call `codegraph_status` (or `codegraph status .`) to check if a graph index exists and is fresh. If missing or stale, request the user run `codegraph init -i` or `codegraph sync`.
2. Use `codegraph_context` to build a focused context for the current task (for example, "refactor auth module"), limiting the number of nodes to a manageable size (e.g. 20).
3. Supplement with `codegraph_search` for specific keywords, classes, functions or services relevant to the task. Disambiguate by module, file path or symbol kind when multiple matches appear.
4. For broad unfamiliar areas, use `codegraph_explore` after search has identified concrete symbols, filenames or short code terms. Avoid natural-language-only explore queries.
5. Use `codegraph_files` to understand the repository structure and identify important directories and modules.
6. Summarise the context: key files, modules, classes, functions, entry points, external integrations, and potential risks. Use lists or tables for clarity.
7. Only after building context should you read source files or edit code.

Rules:
- Do not read large numbers of files without first using CodeGraph to narrow down the scope.
- Keep the main session lightweight: prefer `codegraph_context` for focused work and reserve `codegraph_explore` for genuine surveys.
- Clearly state when the context is incomplete or when CodeGraph analysis is unavailable.
