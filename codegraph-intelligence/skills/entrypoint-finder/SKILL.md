---
name: entrypoint-finder
description: Find application entry points and system startup locations using CodeGraph.
---

Use this skill when you need to locate where an application or service starts execution (e.g. for debugging, instrumentation or understanding control flow).

Goal: Identify the files, functions or scripts that act as entry points into the system, whether they are web servers, background workers, CLI commands or scheduled jobs.

Preferred tools:
1. **CodeGraph MCP tools**: `codegraph_status`, `codegraph_files`, `codegraph_search`, `codegraph_node`, `codegraph_callers`, `codegraph_explore`.
2. **Fallback**: CodeGraph CLI commands: `codegraph status`, `codegraph files`, `codegraph query`.
3. **If neither MCP nor CLI is available**, use conventional heuristics (search for `main`, `server`, etc.) and note that the result may be incomplete.

Workflow:
1. Ensure the graph index is available with `codegraph_status`.
2. Search for common entrypoint patterns using `codegraph_search`:
   - Functions named `main`, `run`, `start`, `bootstrap`.
   - Files like `server.js`, `app.py`, `index.ts`, `cli.rb`.
   - Framework-specific entrypoints (e.g. `manage.py`, `bin/rails`, `cmd/server`, `src/index.ts`).
3. Inspect candidate files and symbols with `codegraph_node` to confirm they are top-level commands or initialisation routines.
4. Use `codegraph_callers` on each candidate to see if anything calls them. True entrypoints usually have no callers and are only referenced from configuration or script definitions.
5. If framework wiring is unclear, use one bounded `codegraph_explore` query anchored to the candidate files or route symbols.
6. Compile a list of confirmed entrypoints, including:
   - Type (web server, CLI command, worker, scheduler)
   - File path
   - Function or class name
   - Description of what it starts
7. Present the list, grouping similar entrypoints together and noting any that might be dead code or unused.

Rules:
- Do not remove or edit entrypoints; this skill is for identification only.
- If the analysis is ambiguous, provide multiple candidates and explain why more investigation is needed.
