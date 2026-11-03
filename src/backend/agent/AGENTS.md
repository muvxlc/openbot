# `src/backend/agent`

Agent execution runtime, turn lifecycle, attention management, and tool routing.
Earned file: score 18, high complexity (96 files, 34k LOC, core agent orchestration).

## OVERVIEW
Coordinates conversation turns, agent attention, memory compaction, prompt generation, tool dispatch, and provider client execution.

## WHERE TO LOOK
| Subsystem | Primary Files | Role |
|---|---|---|
| Service Composer | `agent-service.ts` | Composes attention, drain scheduler, turns, and tools |
| Turn Lifecycle | `turn-lifecycle.ts`, `thread-lifecycle.ts` | Step execution, state machine, and response streaming |
| Attention & Drain | `attention-registry.ts`, `drain-scheduler.ts` | Turn queue, focus management, and delivery throttling |
| Tool Dispatch | `openbot-tools.ts`, `data-tools.ts` | Tool catalogs, routing, parameter decoding, schemas |
| Memory & Compaction | `agent-memories.ts`, `context-compaction.ts` | Memory retrieval, token compaction, conversation context |
| Provider Protocol | `acp-client.ts`, `provider-models.ts` | Provider sessions, model speed tiers, ACP integration |
| Runtime Gates | `duplication-gate.ts`, `usage-limit-gate.ts` | Duplicate turn prevention, usage and quota gating |

## CONVENTIONS
- **One class per file:** Kebab-case naming, `<Name>Options` + `<Name>`, private fields using `#`, constructor assigns only. No barrel exports.
- **Effect 4 services:** Typed workflows with `Effect.fn` and `Effect.gen`, schemas via `@effect/schema`, tagged errors via `Schema.TaggedError`.
- **Direct module imports:** Consumers import specific files under `./agent/<module>`; no `index.ts` barrel.
- **Localized strings:** User-visible text and status messages must resolve through `sourceText` from `@openbot/i18n/source`.
- **Identity stability:** Maintain persistent agent identity, workspace, and thread state across model/provider switches and app restarts.

## ANTI-PATTERNS
- Do not mix Promise APIs inside Effect workflows; do not use legacy Effect v3 compatibility aliases.
- Do not add new domain concerns to central hotspots (`agent-service.ts`, `drain-scheduler.ts`, `attention-registry.ts`); extract to focused helper files.
- Do not log raw prompt content containing credentials or personal tokens.
- Do not bypass tool parameter schemas when exposing new tools to the model.
