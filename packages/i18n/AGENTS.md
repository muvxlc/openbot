# @openbot/i18n

## OVERVIEW
Typed localization runtime, catalog assembly, and source-text fallback across desktop, web, and mobile (Score: 17, package boundary & distinct localization domain).

## WHERE TO LOOK
| Area | File / Directory | Purpose |
| --- | --- | --- |
| Desktop Entry | `src/index.ts` | Package root, `translateFor`, `AppTranslate`, `AppTextKey` |
| Mobile Entry | `src/mobile.ts` | Mobile runtime, excludes desktop catalogs from Hermes bundle |
| Source Text | `src/source-text.ts` | `sourceText`, `localizeSourceText`, IPC/wire template reverse matching |
| Core Engine | `src/message.ts` | Type-safe param substitution, CLDR plural resolution, fallback chain |
| Locales | `src/locale.ts` | `resolveLocale`, `formatLocale`, 13 supported `TRANSLATED_LOCALES` |
| Formatters | `src/format.ts` | `createFormat`, `AppFormat` date/number/list formatting |
| Lazy Loader | `src/lazy-catalogs.ts` | Web dynamic chunk loading for non-English locales |
| English Source | `src/messages/en/` | Canonical message catalogs partitioned by functional area |
| Translations | `src/messages/<locale>/` | Translated area catalogs (`de`, `es`, `fr`, `it`, `ja`, `ko`, `nl`, `pl`, `pt`, `ru`, `tr`, `zh`) |

## CONVENTIONS
- English is canonical source of truth; translations may omit keys to fall back to English automatically.
- Area isolation: Message keys must follow `<area>.<name>` prefix. English modules must call `defineMessages("<area>", { ... })`.
- Plural messages: Require `{count}` placeholder and explicit plural object shape (`{ other: "...", one: "..." }`).
- Cross-process boundaries: IPC endpoints, main process, and team backends emit English text via `sourceText("key", params)`; UI reverses and localizes via template matching.
- Aggregators: `index.ts`, `mobile.ts`, `shared.ts`, and `source.ts` in locale folders only assemble modules; never place raw messages directly in aggregators.
- Subpath exports: Import via `@openbot/i18n`, `@openbot/i18n/mobile`, `@openbot/i18n/source`, `@openbot/i18n/languages`, or `@openbot/i18n/format`.

## ANTI-PATTERNS
- Never rebuild translator instances in render loops; reuse per-locale cached instances from `translateFor` or `mobileTranslateFor`.
- Never translate protocol values, provider identifiers, analytics event names, raw log lines, or LLM agent prompts.
- Never add a key outside its owning area prefix or duplicate a key across multiple area modules.
- Never accept plain string translation where English defines a plural object, or vice-versa.
- Never accept ambiguous source text templates matching multiple keys; `bun run i18n:check` fails on collisions.
- Never pass a `t` prop down component trees; consume text hooks directly (`useText()` in `@openbot/ui/text` or `@/shared/lib/text`).

## COMMANDS
```bash
bun run i18n:check                           # Validate prefix conventions, missing keys, and template collisions
bun run --filter @openbot/i18n typecheck    # Validate package TypeScript compilation
```
