# `src/renderer`

SolidJS desktop and web client UI layer rendering application shells, features, and platform adapters (score 14, subtree root).

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Shell & mounting | `src/App.tsx`, `src/AppView.tsx`, `src/WorkspaceShell.tsx` | App frame, overlays, lazy views, workspace layout |
| Core contexts | `src/simple-context.tsx`, `src/app-providers.tsx` | `createSimpleContext` pattern, provider hierarchy |
| Feature domains | `src/features/<domain>/` | Domain context, views, logic, and ports |
| Bridge seams | `src/features/*/*-port.ts`, `src/app-port.ts` | IPC bridge interfaces (`window.openbot`) |
| Shared UI & styling | `src/styles/app-shell.css`, `src/styles.css` | Global theme layer, cascade-ordered styles |
| Component sandbox | `stories/` | Storybook catalogs; see `stories/AGENTS.md` |
| Test mocks & fixtures | `src/preview/` | IPC mocks for development and Storybook |

## CONVENTIONS

- SolidJS 2.0 RC (`solid-js@2.0.0-rc.18`, `@solidjs/signals`, `@solidjs/web`, `@kobalte/core@2.0.0-alpha.0`). Read props in `createEffect` compute function, not callbacks. Wrap conditional props in `untrack` when reads must not re-trigger.
- One `createStore` per concern; avoid sprawling `createSignal` calls. Store setters stay private behind named mutations. Do not destructure stores.
- Feature files follow naming: `<domain>-context.tsx`, `<Domain>*.tsx` components, `<domain>-*.ts` logic, `<domain>-port.ts` bridge calls, and `stores/*-store.ts`.
- UI strings use `@openbot/i18n` (`currentText()`, `useText()`). No hard-coded UI text.
- Re-use primitives from `@openbot/ui` and tokens from `packages/brand/src/tokens.css` (`--openbot-*`). Never introduce color or size literals.

## ANTI-PATTERNS

- Never import `src/main`, `src/backend`, or `src/preload` (`biome.json` path restriction).
- Never split `src/styles/app-shell.css` without lifting theme layer first (cascading specificity hazard).
- Never use `console.*`, `data-testid`, `any`, `@ts-ignore`, or component-scoped CSS imports.
- Never write raw `window.openbot` calls in components; route through domain `*-port.ts`.
- Never verify UI with `dist/` or production builds during development (`bun run dev` only).
