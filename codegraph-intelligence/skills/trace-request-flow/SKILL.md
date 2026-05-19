---
name: trace-request-flow
description: Trace a request or function call from entry point through middleware, controllers, services, repositories and external calls using CodeGraph.
---

Use this skill when the user asks how a particular request, API call, route or function flows through the system.

Goal: Produce a sequential overview of the call chain, highlighting key abstractions (middleware, controller, service, repository, external integration) and noting asynchronous boundaries.

Preferred tools:
1. **CodeGraph MCP tools**: `codegraph_search`, `codegraph_context`, `codegraph_explore`, `codegraph_node`, `codegraph_callers`, `codegraph_callees`.
2. **Fallback**: CodeGraph CLI commands: `codegraph context` with an appropriate description and `codegraph query` to locate symbols.
3. **If neither MCP nor CLI is available**, proceed with caution using conventional navigation and clearly state that graph-backed flow analysis is unavailable.

Workflow:
1. **Identify the starting point**:
   - For web/API requests, search for the route path or HTTP verb using `codegraph_search` or search for framework router registrations.
   - For function calls, search for the function name.
2. Use `codegraph_context` when the route or function is ambiguous. Use `codegraph_explore` only for broad, unfamiliar flows after search has produced concrete anchors.
3. **Confirm the start node** using `codegraph_node` and note its kind (route handler, controller, CLI command).
4. **Walk down the call chain**:
   - Use `codegraph_callees` to retrieve direct callees for each function in the chain.
   - Distinguish middleware, controller, service and repository layers based on naming conventions, directory structure and file location.
   - Include database or external service calls as separate steps.
5. At each step, record the order of calls and note asynchronous boundaries (promises, awaits, event loops).
6. Present the call chain as an ordered list or table with columns such as level, component, file, description and external effect.
7. Optionally visualise the flow with a textual diagram if it aids comprehension.
8. Summarise key insights: entry point, major abstractions, side effects, error handling points, and external integrations.

Rules:
- When using CLI fallback, rely on `codegraph context` to summarise call chains and disambiguate manually with file reading if necessary.
- Avoid exhaustive traversal; limit depth to what is relevant to the user's question.
- Clearly state when parts of the call graph are inferred rather than directly observed in the graph.
