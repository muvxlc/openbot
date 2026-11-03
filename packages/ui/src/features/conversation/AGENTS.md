# packages/ui/src/features/conversation

## OVERVIEW
Message streaming pipeline, composer editor, chat virtualization, trace timeline, and interactive agent blocks (earned: score 13, distinct domain).

## WHERE TO LOOK
| Path | Purpose |
| --- | --- |
| `DirectConversation.tsx` | Main conversation container and message stream coordinator |
| `createChatVirtualizer.ts` | TanStack virtual core wrapper for high-performance message scrolling |
| `MarkdownMessageText.tsx` | Streaming markdown parser with syntax highlighting, KaTeX math, and mermaid |
| `MessageRendering.tsx` | Message bubble composition, thinking disclosures, reactions, and retry controls |
| `ComposerEditor.tsx` | Multiline chat input handling slash commands, mentions, file drop, and voice morph |
| `send-shortcut.ts` | Platform-specific send modifier key resolution (`Enter` vs `Cmd/Ctrl+Enter`) |
| `TraceWaterfall.tsx` | Execution trace visualization with flame-graph timeline blocks and nested tool calls |
| `ChatActionMarker.tsx` | Tool execution markers, approval prompts, and status indicators |
| `ui-blocks/` | Interactive agent UI blocks (`UiBlockCard`, `UiConfirmBlock`, `UiFormBlock`, `UiChoiceBlock`) |
| `threads/` | Branching conversation threads and reply drawer (`AgentThreadView`, `ThreadTabs`) |
| `conversation.css` | Ordered style manifest importing KaTeX and 47 scoped stylesheet fragments in `styles/` |

## CONVENTIONS
- **Ordered style cascade**: `conversation.css` defines the exact cascade order. Sub-component styles must live in `styles/` and follow declaration sequence.
- **Message virtualization**: Virtualizer state from `createChatVirtualizer` manages dynamic item measurement and scroll anchoring during streaming.
- **Streaming optimization**: Markdown tokens stream incrementally into memoized blocks; avoid re-rendering entire markdown documents on chunk arrival.
- **Send shortcuts**: Use `isSendShortcutKey` and `sendShortcutAriaKey` from `send-shortcut.ts` to respect user keybinding preferences.

## ANTI-PATTERNS
- NEVER import stylesheet fragments from `styles/` individually into consumer components; all imports must flow through `conversation.css`.
- NEVER alter the `@import` declaration order in `conversation.css` without auditing specificity regressions.
- NEVER parse full markdown documents synchronously in effect callbacks; keep streaming chunks memoized.
- NEVER bind raw keyboard event keys directly for message submission; always use `send-shortcut.ts` helpers.
- NEVER mutate virtualizer scroll offset during active gesture scrolling or rubber-banding.
