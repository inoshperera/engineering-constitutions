# Project-specific instructions

- This project is a bare React Native application (no Expo) with the New Architecture and Hermes enabled.
- Follow the shared `framework.react-native` constitution module for architecture, native, testing, and release work.
- Every feature must be verified on both iOS and Android before review.
- Run `tsc --noEmit`, lint, unit tests, and the Maestro `smoke` flows before submitting changes.

## Project-specific decisions

| Decision | Value |
|---|---|
| React Native version | 0.82.x |
| Node / Ruby / JDK / Xcode versions | See `.nvmrc`, `.ruby-version`, `PROJECT_SETUP.md` |
| Minimum iOS / Android `minSdkVersion` / `targetSdkVersion` | iOS 15.1 / 24 / 35 |
| Bundle ID / application ID per environment | `io.entgra.sample.dev`, `io.entgra.sample.staging`, `io.entgra.sample` |
| OTA updates | None |
| Styling | StyleSheet + design tokens |
| React Compiler enabled? | No |
| Client state / server state | Zustand / TanStack Query |
| Package manager | npm |
| Auth method | OAuth 2.0 / OIDC |
| Certificate pinning required? | No |
| Offline requirements | Read-only cache for the last loaded data |
| Supported languages / RTL | English; no RTL |
| Coverage target | 80% lines for `src/features/**` and `src/lib/**` |
| Maestro test backend and test accounts | Staging mock server; accounts from CI secrets |
| Performance budgets | Cold start < 2 s on a mid-range Android device |
| MDM / enterprise distribution requirements | None |
| Other deviations | None |
