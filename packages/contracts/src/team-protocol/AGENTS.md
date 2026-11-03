# packages/contracts/src/team-protocol

## OVERVIEW
Frozen multi-version wire protocol definitions, codecs, capability limits, and WebRTC adapters for Team API (earned: score 14, distinct domain).

## WHERE TO LOOK
| Path | Purpose |
| --- | --- |
| `current.ts` | Current baseline capability definitions and route contracts |
| `index.ts` | Protocol version barrel (`TEAM_PROTOCOL_V1`..`V9`) and adapter registry |
| `provider-aware-codec.ts` | Multi-version wire encoder/decoder bridging dynamic model and provider schemas |
| `v1.ts` .. `v9.ts` | Frozen immutable wire protocol specifications per protocol version |
| `v1-adapter.ts` .. `v8-adapter.ts` | Backward-compatibility translation adapters for older client/host versions |
| `*-webrtc-adapter.ts` | WebRTC peer signaling and data channel protocol bridges |
| `fixtures/v*/` | Frozen client and host JSON wire fixtures recording released formats |

## CONVENTIONS
- **Protocol Immutability**: Everything here is a contract shipped in existing builds. Never alter fields, types, or semantics of released protocols (`v1` through `v8`).
- **Protocol vs SemVer**: Never use application SemVer as a wire protocol version. App versions are diagnostic metadata only.
- **Frozen Fixtures**: Keep codec, adapter, and fixtures for each released protocol under `fixtures/v<N>/`. No unit test reads them; inspect codec diffs against them by hand.
- **Capabilities for Extensibility**: Additive changes use capabilities; missing capabilities cleanly disable individual features.
- **128 Capability Limit**: Protocols 7 and 8 cap capability lists at 128 items and are full. Any new capability must go into `TEAM_PROTOCOL_V9_ONLY_CAPABILITIES`.
- **Open Sets**: From protocol 8, provider IDs and auth kinds are open (`OPEN_PROVIDER_ID`). Adding a new provider does not increment protocol versions.
- **Fail Closed**: Malformed known wire payloads fail closed as `protocol_error`. Unknown optional events are ignored.

## ANTI-PATTERNS
- NEVER mutate or remove fields in frozen protocol versions (`v1.ts` through `v8.ts`). Breaking changes require a new protocol version.
- NEVER serialize internal desktop IPC types across the wire protocol boundary.
- NEVER append new capabilities to v7 or v8 capability lists (exceeds 128-element hard limit).
- NEVER remove protocol adapters solely for age, release count, or SemVer distance. Removal requires architectural review, E2E validation in both directions, and changelog guidance.
- NEVER route chat payloads or commands over Signal protocol relays; live messaging travels peer-to-peer over WebRTC data channels or Team HTTP routes.
