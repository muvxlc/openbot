# `src/backend/database`

SQLite controllers, conversation persistence, query indices, and projection maintenance.
Earned file: score 12, distinct domain (17 query/projection controllers, row mapping).

## OVERVIEW
Dedicated SQLite controllers managing conversation history, thread replay, mailbox projections, and row decoders.

## WHERE TO LOOK
| Controller | File | Role |
|---|---|---|
| Conversation Writer | `conversation-writer.ts` | Message persistence, thread event writes, mutations |
| Conversation Queries | `conversation-queries.ts` | Thread message lookups, pagination, and order keys |
| Mailbox Projection | `mailbox-projection.ts` | Projecting event log state into mailbox tables |
| Row Decoders | `database-rows.ts` | Row mapping and SQLite type conversions |
| Thread Replay | `thread-replay.ts` | Rebuilding thread projections from raw event history |
| Agent Roster & Usage | `agent-roster.ts`, `agent-usage.ts` | Tracking agent metadata, sessions, and token cost |

## CONVENTIONS
- **No facade imports:** Controller classes never import the `OpenBotDatabase` facade; they receive `DatabaseCore` via options.
- **Dynamic connection access:** Always read `core.connection` on each call; never cache connection handles across queries.
- **Centralized decoding:** SQLite row decoders and SQL statements live in domain query modules and `database-rows.ts`.
- **Append-only log writes:** Write projection rows via `database.dispatch`; keep event payloads minimal (identifiers only).
- **Class shape:** One class per file, kebab-case, constructors assign dependencies only.
- **Store ownership:** Each store documents the specific tables it owns in header comments.

## ANTI-PATTERNS
- Never start independent transactions inside projection controllers; nested `BEGIN` causes SQLite failures.
- Never resolve database paths inside controllers (delegated to configuration and facade).
- Never expose raw `Error` stack or absolute filesystem paths in database failure objects.
- Do not rely on worker thread termination to interrupt blocked `sqlite3_step` statements.
- Do not yield asynchronous work while SQLite authorizer state is active.
