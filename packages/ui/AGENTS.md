# packages/ui

## OVERVIEW
Shared SolidJS 2.0 component primitives, design system tokens, and modular feature UI across desktop and web (earned: score 14, subtree root).

## WHERE TO LOOK
| Path | Purpose |
| --- | --- |
| `src/` | Core component primitives (`button`, `select`, `toast`, `alert`, `switch`, `dialog`) |
| `src/components/` | Shared composite components (`ProviderModelPicker`, `GlobalSearch`, `SettingsPanel`) |
| `src/styles/` | Package stylesheet entry (`primitives.css` exported as `@openbot/ui/styles.css`) |
| `src/features/conversation/` | Chat stream, composer, virtualizer, trace waterfall (see `conversation/AGENTS.md`) |
| `src/features/` | Modular domain UI (`workflows`, `settings`, `sidebar`, `servers`, `account`, `billing`) |
| `src/text.tsx` | UI localization context accessor (`useText()` providing `t`, `format`, `errorMessage`) |
| `src/data.ts` | Shared UI display models, entity wrappers, and data contracts |

## CONVENTIONS
- **SolidJS 2.0 reactivity**: Read props in compute function of `createEffect`, never in its callback (avoids `PRIMITIVE_IN_EFFECT_CALLBACK`).
- **Platform isolation**: Receive all data and actions through typed props and callbacks. Keep platform features in typed content slots or adapters, never environment checks.
- **Subpath exports**: Export primitives and features via explicit package subpaths in `package.json` (`@openbot/ui/button`, `@openbot/ui/features/<domain>/<Component>`).
- **Styling**: Pure CSS with design tokens (`@openbot/brand`). Array class bindings `class={["class-a", condition && "class-b"]}` standard.
- **Localization**: Call `useText()` from `@openbot/ui/text`. Never add a `t` prop to components.

## ANTI-PATTERNS
- NEVER import renderer, main, backend, or preload code, or call `window.openbot` in shared UI.
- NEVER hand-roll `<button>`, `<input>`, `<textarea>`, `<select>` or composite ARIA roles outside primitives (`scripts/ui-foundation-check.ts`).
- NEVER import directly from `@kobalte/core` or `lucide-solid` inside `features/`; use primitives in `src/`.
- NEVER use direct Kobalte namespace aliases like `export const Dialog = DialogPrimitive` in adapters.
- NEVER use hardcoded color hex/rgb literals in inline styles; use palette tokens from `@openbot/brand`.

## COMMANDS
```bash
bun run check:ui           # Enforces design token usage, no native controls, no direct Kobalte imports
bun run lint               # Biome check verifying desktop boundary and style rules
bun run desktop:typecheck  # TypeScript workspace check including renderer and shared contracts
bun run test:ui-blocks     # E2E test suite validating conversation UI block rendering
```

