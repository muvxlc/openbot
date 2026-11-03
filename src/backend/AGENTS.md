# `src/backend`

Subtree root: SQLite database, domain stores, provider clients, and agent orchestration.
Earned file: score 18, subtree root.

## OVERVIEW
Core desktop backend services: local SQLite persistence, migration pipeline, provider adapters, and agent execution.

## WHERE TO LOOK
| Domain | Path | Role |
|---|---|---|
| Agents | `src/backend/agent/` | Turn lifecycle, tools, memory compaction, prompt builders |
| Database | `src/backend/database/` | SQLite query/projection controllers and row decoders |
| Messaging | `src/backend/messaging/` | Slack, Discord, and Telegram drivers and transports |
| Workflows | `src/backend/workflows/` | Effect-based automation runtime and trigger matching |
| Core Stores | `openbot-database.ts`, `mailbox-store.ts` | Central DB facade, mailbox state, attachment ops |
| Host Process | `agent-data/agent-database-host.ts` | Isolated SQLite process for agent-executed SQL |

## CONVENTIONS
- **One class per file:** Kebab-case filenames, `<Name>Options` + `<Name>`, `readonly #` fields, assignment-only constructors. No barrel files.
- **Effect 4 services:** Named `Effect.fn`, `Effect.gen`, `Context.Service`, explicit layers, and `Schema.TaggedError`.
- **Database migrations:** Never edit or delete a shipped migration (including frozen v8 baseline). Append contiguous versions, update `openbot-database-schema-history.json` and `LATEST_SCHEMA_SQL` in the same change.
- **Two build paths:** `createLatestDatabase` execs `LATEST_SCHEMA_SQL` and marks migrations applied without running them; upgrades run `MIGRATIONS`. Divergence caught by `openbot-database-schema-parity.test.ts`.
- **Migration safety:** Every migration is irreversible production data work: preserve all user data, keep schema change and `schema_migrations` marker in one transaction, roll back on error, restore FK enforcement in `finally`, run integrity checks. No automatic full-DB backups.
- **Store boundaries:** `mailbox-store.ts` owns mailbox DB writes. `attachment-files.ts` owns file ops and must never import `MailboxStore` or write DB state. Apply staged attachments to mailbox only after atomic conversation/mailbox commit.
- **Transaction ownership:** `dispatch` and `deleteEventsAndReceipt` open transactions only if none open. `ThreadReplay.rebuildThreadProjection` and `persistConversationAndMailbox` open unconditionally.
- **Controller handles:** Controllers read `core.connection` per use; never cache connection across `initialize()`/`close()`.
- **Host process isolation:** `agent-data/agent-database-host.ts` runs in a dedicated Electron `utilityProcess` (not a worker thread) to bound runaway statements with SIGKILL. Runtime imports limited strictly to `node:*` and `effect` (repo types only).

## ANTI-PATTERNS
- Never edit or delete released migrations or weaken `openbot-database-schema-parity.test.ts`.
- Never touch `apps/auth-api/migrations/` from here (Cloudflare D1, owned by `apps/auth-api/AGENTS.md`).
- Never add new concerns to large files (`agent-service.ts`, `openbot-database.ts`); extract on modification.
- Never open independent transactions inside projection controllers (triggers nested `BEGIN` errors).
- Never expose raw `Error` stack or file paths in agent database failure values.
