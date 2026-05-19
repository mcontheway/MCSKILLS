---
name: dependency-audit
description: Audit module dependencies and detect high coupling or cycles using CodeGraph.
---

Use this skill when you need to understand how modules depend on each other, identify tight coupling or detect circular dependencies.

Goal: Provide insights into module relationships so you can plan refactoring, enforce architectural boundaries or improve maintainability.

Preferred tools:
1. **CodeGraph MCP tools**: `codegraph_status`, `codegraph_files`, `codegraph_search`, `codegraph_node`, `codegraph_callers`, `codegraph_callees`, `codegraph_explore`. CodeGraph does not yet expose a dedicated dependency graph query, but node metadata and caller/callee queries help identify imports and references.
2. **Fallback**: CodeGraph CLI commands: `codegraph status`, `codegraph files`, `codegraph query`.
3. **If neither MCP nor CLI is available**, use manual inspection; state that graph-backed dependency analysis is unavailable.

Workflow:
1. Confirm the graph index is current with `codegraph_status`.
2. Identify the modules or packages under audit (e.g. `app`, `domain`, `infra`). Use `codegraph_files` to list directories and submodules.
3. For each module, use `codegraph_search` to find exports and their locations.
4. For each export, call `codegraph_node`, `codegraph_callers` and `codegraph_callees` to inspect incoming and outgoing references.
5. Use `codegraph_explore` only when module boundaries are unclear after the targeted queries.
6. Summarise dependencies:
   - **Inbound dependencies**: which modules depend on this module?
   - **Outbound dependencies**: which modules this module depends on?
7. Look for signs of high coupling: modules that import many symbols from each other, or modules with both high inbound and high outbound dependency counts.
8. Attempt to detect circular dependencies by examining mutual imports. When node data shows that module A imports module B and module B imports module A, flag it as a cycle.
9. Present findings in a table or bulleted list, highlighting:
   - Modules audited
   - Their inbound and outbound dependencies
   - Potential cycles
   - Recommendations to decouple or apply dependency inversion

Rules:
- Be explicit that the current CodeGraph API does not expose a dedicated full dependency graph endpoint; your analysis is based on available metadata and may miss dynamic or indirect imports.
- Encourage incremental improvements rather than wholesale rewrites.
