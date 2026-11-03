# Mobile Subtree Knowledge Base

<!-- Earned: Subtree Root (Score: 17) -->

## OVERVIEW
Expo and React Native mobile application for iOS and Android, built with HeroUI Native and Expo Router.

## STRUCTURE
```
apps/mobile/
├── modules/     # Local Expo native modules (Swift/Kotlin/C++)
├── src/app/     # Expo Router file-system routes and layout stacks
├── src/features/# Modular feature slices (chat, servers, agents, workspace)
└── src/shared/  # Shared design tokens, text, haptics, and UI primitives
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Route screens & navigation | `src/app/` | File-system routing; thin wrappers delegating to features |
| Native stack & modal layout | `src/app/(app)/_layout.tsx` | Root stack controller, modal sheet options, app providers |
| Feature logic & screens | `src/features/` | Feature slices; deep imports, no root barrel files |
| Native platform modules | `modules/` | Swift, Kotlin, sherpa-onnx STT/TTS, AppLock, FCM service |
| UI tokens & shared widgets | `src/shared/` | HeroUI aliases, `SheetScrollView`, haptics, platform utils |
| App branding & assets | `assets/` | App icons, splash screens, animations |

## CONVENTIONS
- **Design System**: Use HeroUI Native components with theme aliases in `global.css`. Treat navigation chrome as the exception: use native Expo Router and `@expo/ui` primitives (`Stack`, `NativeTabs`) for native iOS Liquid Glass and Android conventions.
- **Interface Text**: Always render text via catalog keys (`useText()`, `currentText()`) from `@/shared/lib/text` with `mobile.` prefix.
- **Sheets**: Use native `formSheet` with fixed detents (`[0.85]` for forms, `[1]` for search). Screens must use `SheetScrollView` (`flex: 1`, `alwaysBounceVertical={false}`). Place save actions in header via `SheetSaveAction`; avoid duplicate in-content submit buttons.
- **Package Management**: Always install packages using `bunx expo install <pkg>` to resolve SDK-compatible versions.

## ANTI-PATTERNS
- **No Fake Chrome**: Do not emulate native headers, tab bars, or sheets with custom blur, gradients, or `GlassView`.
- **No Manual Native Edits**: Never edit `ios/` or `android/` directly when generated via Continuous Native Generation; configure via `app.json` and plugins.
- **Verification Limits**: Do not run full-workspace or parallel typechecks locally; run single scoped checks (`bun run lint`, relevant test file). EAS builds and simulator runs require explicit user authorization.

## COMMANDS
```bash
bun run start          # Start Expo development server
bun run lint           # Biome check on mobile src
bun run typecheck      # Scoped mobile typecheck (codegen + type-checker)
bun run doctor         # Check dependency and config health
```
