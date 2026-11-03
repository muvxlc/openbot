# Native Modules Knowledge Base

<!-- Earned: Distinct Domain (Score: 12, Local Expo Native Modules) -->

## OVERVIEW
Local Expo native modules bridging iOS (Swift/ObjC) and Android (Kotlin/JNI) platform capabilities.

## STRUCTURE
```
modules/
├── android-notifications/ # FCM message decryption and local notification posting
├── app-lock/              # Monotonic clock away-tracking and PBKDF2 PIN derivation
├── composer-paste/        # UITextView runtime swizzling for multiline paste interception
├── quick-actions/         # iOS home screen quick actions and Android app shortcuts
├── share-extension/       # iOS Share Target keychain exchange and Siri intent donation
└── speech/                # sherpa-onnx STT/TTS engine, CoreAudio/AudioRecord PCM streams
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| On-device speech & TTS | `speech/` | `SpeechModule.swift`, `SpeechModule.kt`, sherpa-onnx integration |
| App lock & PIN auth | `app-lock/` | Monotonic clock timing (`mach_continuous_time`), PBKDF2 |
| Composer paste swizzling | `composer-paste/` | `UITextView.paste(_:)` swizzling for multiline text fields |
| App shortcuts | `quick-actions/` | `QuickActionsAppDelegateSubscriber`, Android vector XML |
| iOS share target | `share-extension/` | `INSendMessageIntent` donations, App Group keychain token |
| Encrypted FCM push | `android-notifications/` | `OpenBotMessagingService`, AES/HMAC payload unsealing |

## CONVENTIONS
- **Expo Module Configs**: Each module defines `expo-module.config.json` listing target platforms, module classes, and app delegate subscribers.
- **Dynamic Linking & Checksums**: Remote native frameworks must verify sha256 checksums (`verifySherpaOnnx` Gradle task and `OpenBotSpeech.podspec` curl check).
- **Time Measurements**: Use monotonic clocks (`mach_continuous_time()`, `SystemClock.elapsedRealtime()`) to avoid negative time deltas or clock tampering.
- **Shared Keychain & App Groups**: Share session auth tokens between main app and extensions using App Group `group.run.openbot` and shared keychain access group.

## ANTI-PATTERNS
- **No Expo AV For Raw Audio**: Do not route streaming PCM speech capture or playback through Expo AV; use low-level AudioEngine/AudioTrack in `speech`.
- **No Unsafe Swizzle Fallbacks**: Never alter `UITextView` selectors without checking `canPerformAction` or handling unreadable clipboard items.
- **No Retained Shortcut Intents**: Clear Android launcher shortcut intents immediately after consumption so subsequent resumes do not repeat the action.
