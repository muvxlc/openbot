# `src/renderer/stories`

Storybook visual catalog and testing fixtures for renderer and shared components (score 12, distinct domain).

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Story definitions | `*.stories.tsx` | 163 component stories discovering via `.storybook/main.ts` |
| Shared test fixtures | `fixtures.ts`, `*-fixtures.ts` | Static fixtures re-exporting `src/preview/fixtures` |
| Mock OpenBot IPC | `mock-openbot.ts` | Re-exports `src/preview/mock-openbot` for stories |
| Support helpers | `*-story-support.tsx`, `*-stage.tsx` | Specialized stage wrappers and story harness |
| Story stylesheets | `*-story.css` | Isolated styles imported directly into stories |
| Binary/mock assets | `assets/` | Mock images/SVGs exempt from production image rules |

## CONVENTIONS

- SolidJS Storybook (`storybook-solidjs-vite` using `@solidjs/web` JSX import source).
- Uniform story shape: `satisfies Meta<typeof Component>`, default export, and `StoryObj<typeof meta>`.
- File naming: PascalCase `<Subject>.stories.tsx` for stories; kebab-case for helpers (`*-fixtures.ts`, `*-story-support.tsx`, `*-stage.tsx`).
- Action callbacks use `fn()` from `storybook/test`.
- Hard-coded UI text is permitted in stories (`no-hardcoded-ui-text` excludes `**/*.stories.tsx`).

### Fixtures & Mock Patterns
- Always import mock APIs from `./mock-openbot` and fixtures from `./fixtures` or local `*-fixtures.ts`. Never bypass shims to import `src/preview/*` directly.
- Instantiate IPC harnesses via `createMockOpenBot()`; configure deterministic mock responses rather than emulating full backend services.
- Keep test fixtures static, pure, and immutable. Do not generate nondeterministic fixture data (e.g., dynamic timestamps or random IDs) in story renders.

### Component Isolation Requirements
- Fully decouple components from Electron desktop runtime: bind `window.openbot` through mock controller before mounting stories.
- Wrap complex components in dedicated support stages (`*-story-support.tsx`, `OpenBotPlayground`) providing mock provider hierarchy.
- Confine visual styles to `*-story.css` or component wrapper containers; never mutate root cascading classes or global theme variables outside story scope.

## ANTI-PATTERNS

- Never add story `play` functions (`no-story-play.grit` enforced by Biome and `check:ui`).
- Never use `data-testid`, `className`, `any`, `@ts-ignore`, or `@ts-expect-error`.
- Never run unit tests against this directory; test rules relax here and stories run via Storybook only.
- Never import live Electron IPC, node builtins, or backend services (`src/main`, `src/backend`, `src/preload`).
- Never allow stories to depend on external network endpoints, filesystem state, or unmocked globals.
