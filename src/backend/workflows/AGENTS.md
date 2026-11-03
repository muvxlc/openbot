# `src/backend/workflows`

Automation workflow engine, trigger matching, step fibers, and execution run stores.
Earned file: score 13, distinct domain (15 files, Effect Context.Service workflow engine).

## OVERVIEW
Automation workflow runtime managing event-driven triggers (GitHub, Discord, cron, webhooks) and step execution fibers.

## WHERE TO LOOK
| Component | Primary Files | Role |
|---|---|---|
| Workflow Runtime | `workflows.ts` | `Workflows` `Context.Service`, pass execution, step fibers |
| Definition Store | `workflow-store.ts` | Workflow persistence, versions, revision conflict handling |
| Run Store | `workflow-run-store.ts` | Run execution states, step status transitions, step outputs |
| GitHub Delivery Store | `workflow-github-store.ts` | Delivery deduplication, loop guards, waiting deliveries |
| Trigger Scheduler | `workflow-triggers.ts` | Cron schedule evaluation, webhook receipt payload prep |
| GitHub Integration | `workflow-github-api.ts`, `github-matching.ts` | GitHub event predicate matching and API actions |

## CONVENTIONS
- **Effect Context.Service:** Runtime implemented as `class Workflows extends Context.Service<Workflows, WorkflowsShape>()`.
- **Direct row writes:** `workflow-store.ts` and `workflow-run-store.ts` write directly to SQLite rather than through `dispatch` (audit logs and step text are immutable).
- **Conditional state transitions:** Run step updates must check the preceding status to prevent duplicate settlements.
- **Scoped transactions:** Multi-table workflow mutations wrap with `workflowTransaction(database, work)`.
- **Concurrency control:** Fiber execution coordinates via dual semaphores: one pass lock and one generation lock.
- **Error types:** Operations return `Effect.Effect<..., WorkflowFailed>` with `WorkflowStoreError` on store failures.

## ANTI-PATTERNS
- Never edit released workflow migration SQL strings (`WORKFLOW_SCHEMA_SQL`, `V46_`, `V50_`).
- Do not allow concurrent passes to settle the same step twice (rely on conditional status checks).
- Do not inflate `workflows.ts`; extract step evaluators, prompt generators, and matching logic.
- Do not emit unredacted GitHub or webhook payload secrets into step execution traces.
