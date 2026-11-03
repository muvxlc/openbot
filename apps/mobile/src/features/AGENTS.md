# Feature Slices Knowledge Base

<!-- Earned: Distinct Domain (Score: 14, Modular Feature Architecture) -->

## OVERVIEW
Self-contained business feature slices encapsulating domain models, screens, contexts, and queries.

## STRUCTURE
```
features/
├── agents/      # Agent configuration, avatar thumbnails, activity loops
├── auth/        # Session authentication, token refresh, and login screens
├── chat/        # Virtualized message list, composer, TeX math, markdown
├── servers/     # Server connections, provider settings, drawer navigation
├── workspace/   # Core workspace state machine, query keys, host request sync
└── ...          # 17 additional domain feature slices (voice, search, etc.)
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Workspace state & sync | `workspace/context/` | `MobileWorkspaceProvider`, `useMobileWorkspaceContext` |
| Query cache invalidation | `workspace/model/query-keys.ts` | Global `queryKeys`, `QueryScope` definitions |
| Chat message list & composer | `chat/components/` | Virtualized message feed, paste targets, markdown/math |
| Session auth & challenges | `auth/context/`, `auth/api/` | `MobileSessionProvider`, token exchange, session recovery |
| Server drawer & providers | `servers/components/`, `screens/` | Navigation drawer, connector preferences, provider admin |
| Agent avatars & activity | `agents/components/`, `model/` | `BloubAvatar`, activity state reducers, idle frames |

## CONVENTIONS
- **Strict Deep Imports**: Zero barrel or root `index.ts` files inside feature roots; import concrete file paths directly (`@/features/<feature>/<layer>/<file>`).
- **Internal Layering**: Organize feature directories consistently into `components/`, `model/`, `screens/`, `context/`, and `api/`.
- **Platform Separation**: Split diverging native logic via file suffixes (`.ios.tsx`, `.android.tsx`, `.native.tsx`), not inline runtime conditionals.
- **State Coordination**: Pair TanStack React Query (`queryKeys`) for remote server state with scoped Context (`MobileWorkspaceProvider`) for live host state.

## ANTI-PATTERNS
- **No Barrel Re-exports**: Never create `index.ts` at the root of a feature folder.
- **No Sensitive Chat Inputs**: Never collect credentials, tokens, or PINs in the chat composer; route through dedicated secure form sheets.
- **Local Haptic Alignment**: Trigger haptic feedback solely upon local audio/haptic playback completion, never on streaming chunk arrivals.
- **Non-Preemptive Turn Timers**: Remote host turn timers must serve only as lost-connection fallbacks, never cancelling ongoing streaming turns prematurely.
