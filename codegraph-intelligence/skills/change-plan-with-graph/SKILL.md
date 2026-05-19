---
name: change-plan-with-graph
description: Generate a graph-backed implementation plan before modifying repository code.
---

Use this skill whenever the user asks to implement, refactor, rename, remove, migrate or significantly modify code.

Goal: Force a pre-implementation planning phase that leverages CodeGraph to reduce risk and improve clarity.

Preferred tools:
1. **CodeGraph MCP tools**: `codegraph_status`, `codegraph_context`, `codegraph_explore`, `codegraph_search`, `codegraph_node`, `codegraph_callers`, `codegraph_callees`, `codegraph_impact`.
2. **Fallback**: CodeGraph CLI commands: `codegraph status`, `codegraph query`, `codegraph context`, `codegraph affected`.
3. **If neither MCP nor CLI is available**, state that graph-backed planning is unavailable and proceed with extra caution.

Workflow:
1. Summarise the user's goal in a single sentence to anchor the task.
2. Call `codegraph_status` to ensure the graph index is available and fresh. If not, ask the user to initialise or sync it.
3. Build a task context using `codegraph_context` (e.g. "implement rate limiting for authenticated routes"). Limit nodes to a manageable number.
4. If the target area is unfamiliar, use `codegraph_search` to find concrete anchors and then one bounded `codegraph_explore` call for a subsystem survey.
5. Identify relevant symbols and modules using `codegraph_search`. Use `codegraph_node` to confirm details. Resolve ambiguities.
6. Run `impact-analysis` on the key symbols to understand who calls them, who they call and what tests are affected.
7. Use `test-target-finder` to determine which tests will need to run.
8. Draft a short implementation plan that includes:
   - High-level steps (analysis, code changes, tests, validation).
   - Files or modules to edit.
   - Supporting tasks (e.g. update documentation).
   - Estimated risks and unknowns.
9. Proceed after the plan for low-risk technical changes. Ask the user before editing only when the plan affects product behavior, cost, time, security, privacy, data, permissions or other externally visible outcomes.

Rules:
- Never edit files until a graph-backed plan has been produced. User confirmation is required only for the higher-risk cases above.
- Prefer CodeGraph findings; if using CLI fallback, clearly state that results may be incomplete.
- Encourage the user to ask clarifying questions or adjust the plan before execution.
