---
id: framework.react-native
title: React Native Application Constitution
category: framework
version: 1.0.0
status: active
owners:
  - mobile-platform
applies_to:
  - react-native
  - ios
  - android
depends_on:
  - language.typescript
  - framework.react
  - universal.security
  - universal.testing
---

# React Native Application Constitution

This constitution defines the engineering rules for building Entgra mobile applications with bare React Native on iOS and Android.

It is intentionally opinionated. These rules optimize for a single TypeScript codebase, the React Native New Architecture, MVVM screen structure, both platforms treated as first-class, and releases that cannot be hot-fixed without a store review.

## Scope

This constitution applies to React Native applications and their committed native projects:

- JavaScript/TypeScript source under `src/` and the app entry (`index.js`).
- Native projects under `ios/` and `android/`.
- Maestro end-to-end flows under `e2e/`.
- Build, signing, and release lanes under `fastlane/`.

It builds on, and does not repeat, the universal modules and `language.typescript` / `framework.react`. Where this module is stricter than those modules, this module wins for React Native projects.

Project-specific choices (versions, bundle IDs, OTA, styling approach, budgets) are recorded in the project's `PROJECT_INSTRUCTIONS.md`. See [Project-Specific Decisions](#project-specific-decisions).

## Core Principle

A React Native change MUST work on both iOS and Android, keep Views free of business logic and data access, validate every value that crosses into the app, and be safe to ship through an app store, because a mobile defect cannot be rolled back as quickly as a server deployment.

Keywords: MUST / MUST NOT are hard rules and block merge. SHOULD / SHOULD NOT are strong defaults; a deviation needs a one-line justification in the PR or a code comment.

## Platform and Runtime

- Projects MUST use bare React Native (React Native Community CLI, latest stable at project start).
- Expo MUST NOT be used: no `expo`, `expo-*`, or `@expo/*` packages, no Expo Router, EAS, `expo prebuild`, or `install-expo-modules`.
- The New Architecture (Fabric + TurboModules) MUST be enabled, with the Hermes engine. Every native library MUST support the New Architecture.
- Every feature MUST be built and verified on both iOS and Android before it is considered done.
- Platform-specific code uses `Platform.select` for small differences and `.ios.tsx` / `.android.tsx` files for large ones. Inline `Platform.OS === ...` branching spread across a component SHOULD NOT be used.
- Apps MUST respect platform conventions: Android back button and gesture, safe areas and notches, dynamic type, and dark mode.

## Default Technology Stack

Unless a project records a different choice in its `PROJECT_INSTRUCTIONS.md`, projects use:

| Concern | Default | Notes |
|---|---|---|
| Framework | Bare React Native (Community CLI) | No Expo |
| Architecture | New Architecture + Hermes | Libraries MUST support it |
| Language | TypeScript (strict); Kotlin (Android) and Swift / Obj-C++ (iOS) for native code | No new Java code |
| Navigation | React Navigation (native stack) | Typed route params |
| Server state | TanStack Query | All network data goes through it |
| Client state | Zustand | Redux Toolkit only when chosen; see [Redux](#redux-when-chosen) |
| Forms | React Hook Form + Zod resolver | |
| Validation | Zod | API, storage, deep links, push payloads, env |
| Lists | FlashList | `FlatList` acceptable for short, static lists |
| Animation | Reanimated + Gesture Handler | No core `Animated` API for new code |
| Images | `@d11/react-native-fast-image` | Core `Image` for simple, static cases |
| Secure storage | `react-native-keychain` | iOS Keychain / Android Keystore |
| General storage | `react-native-mmkv` | Never for secrets |
| Environment config | `react-native-config` | Values are bundled, never secrets |
| Splash screen / app icon | `react-native-bootsplash` | |
| Styling | `StyleSheet.create` + design tokens, or NativeWind when chosen | One approach per project |
| i18n | `i18next` / `react-i18next` + `react-native-localize` | |
| Unit / component tests | Jest + React Native Testing Library | Network mocked with MSW |
| E2E tests | Maestro | See [E2E Testing with Maestro](#e2e-testing-with-maestro) |
| Lint / format | ESLint (`@react-native/eslint-config`) + Prettier | |
| Errors and monitoring | Sentry (`@sentry/react-native`) | |
| Builds and releases | Fastlane in CI | Fastlane `match` (iOS), CI-stored keystore (Android) |
| OTA updates | None by default | Any OTA solution needs explicit approval |
| iOS dependencies | CocoaPods via Bundler (`bundle exec pod install`) | Commit `Podfile.lock` and `Gemfile.lock` |
| Package manager | One per project; commit the lockfile | |

Rules:

- A second library MUST NOT be introduced for a concern already covered above (for example, no Redux alongside Zustand, no Axios alongside a fetch wrapper).
- Before adding a native module, check the [React Native Directory](https://reactnative.directory) for New Architecture support, recent maintenance, and support on both platforms. Record the justification in the PR as required by `universal.dependency-management`.

## TypeScript and Boundary Validation

These rules tighten `language.typescript` for mobile code:

- `tsconfig.json` MUST enable `"strict": true`, `noUncheckedIndexedAccess`, and `noImplicitOverride`.
- `any` MUST NOT be used. Use `unknown` and narrow it. `// @ts-ignore` MUST NOT be used; `// @ts-expect-error` is allowed only with a reason comment.
- Data from outside the app (API responses, deep-link params, storage reads, push payloads, environment values) MUST be validated at the boundary with a Zod schema before use. Types are derived with `z.infer`, not written twice.
- Non-null assertions (`!`) SHOULD NOT be used outside tests.

Rationale: a runtime crash on mobile needs a store release to fix. The compiler and boundary validation catch it first.

## Architecture: MVVM

Every screen is split into three layers inside a feature-first structure:

| Layer | Responsibility | Lives in |
|---|---|---|
| View | Renders UI and forwards user input. No business logic, no data access. | `features/<feature>/screens/`, `features/<feature>/components/`, `src/components/ui/` |
| ViewModel | Screen state and logic: calls the Model, shapes data, handles user actions, decides navigation. No JSX, no styles. | `features/<feature>/viewmodels/` |
| Model | Data and business rules: API services, Zod schemas, query/mutation hooks, stores, pure domain logic. Knows nothing about screens. | `features/<feature>/api/`, `features/<feature>/model/`, `features/<feature>/store/`, `src/lib/` |

Dependencies MUST point one way: View → ViewModel → Model.

- Views MAY import their ViewModel, other View components, `components/ui`, theme tokens, `useTranslation`, and types. Views MUST NOT import from `api/`, `store/`, `model/` services, `src/lib/api`, the query client, or store hooks.
- ViewModels MAY import Model hooks, store selectors and actions, services, typed navigation hooks, and shared hooks. ViewModels MUST NOT import View components, return JSX, or contain styles.
- Model code MUST NOT import Views, ViewModels, navigation, or theme. Services, schemas, and domain rules are plain TypeScript with no React imports; only query/mutation hooks and stores use React APIs.
- These boundaries MUST be enforced with ESLint (`no-restricted-imports` or `eslint-plugin-boundaries`).

ViewModel contract:

```tsx
// features/auth/viewmodels/useLoginViewModel.ts
type LoginViewModel = {
  state: {
    email: string;
    isSubmitting: boolean;
    errorMessage: string | null;
    canSubmit: boolean;
  };
  actions: {
    changeEmail: (value: string) => void;
    submit: () => void;
  };
};

export function useLoginViewModel(): LoginViewModel {
  // calls Model hooks (e.g. useLoginMutation), derives UI-ready state, defines actions
}

// features/auth/screens/LoginScreen.tsx
export function LoginScreen() {
  const { state, actions } = useLoginViewModel();
  return <LoginForm {...state} onChangeEmail={actions.changeEmail} onSubmit={actions.submit} />;
}
```

- Each screen MUST have one ViewModel named `use<Screen>ViewModel` in `viewmodels/use<Screen>ViewModel.ts`, with an explicitly typed return value.
- A ViewModel returns `{ state, actions }`. `state` is UI-ready: formatted dates and numbers, flags such as `isLoading` / `isEmpty` / `canSubmit`, and user-facing error messages. The View does no data transformation.
- `actions` are named by user intent (`submit`, `retry`, `selectItem`), not as setters (`setX`).
- The screen reads route params and passes them to the ViewModel as arguments (`useOrderDetailViewModel(orderId)`).
- A large reusable component with its own logic MAY have a colocated `use<Component>ViewModel`. Simple presentational components take props only.
- Purely visual state (an accordion open/closed, a focused input, animation values) MAY stay in the View. Anything that affects data or business rules goes in the ViewModel.
- A ViewModel that grows beyond about 150 lines SHOULD be split into smaller hooks composed inside it.

Data flows one way: the Model provides data, the ViewModel shapes it into `state`, the View renders it. User input calls an `action`, the ViewModel calls the Model, and updated data flows back down.

## Project Structure

```
android/                  # native Android project (committed, owned by us)
ios/                      # native iOS project (committed, owned by us)
index.js                  # app entry: registers the root component only
src/
  app/
    App.tsx               # providers: query client, navigation, theme, error boundary
  navigation/             # navigators, route param types, linking config
  features/
    <feature>/
      screens/            # View: read params, call the ViewModel, render
      components/         # View: presentational UI used only by this feature
      viewmodels/         # ViewModel: use<Screen>ViewModel hooks
      api/                # Model: service functions, query/mutation hooks, query key factory
      model/              # Model: Zod schemas, types, pure business rules
      store/              # Model: feature-scoped Zustand store or Redux slice
      index.ts            # public API of the feature
  components/ui/          # View: shared design-system primitives
  lib/                    # app-wide infra: api client, query client, storage, i18n, logger
  native/                 # TS specs and wrappers for our Turbo Modules / Fabric components
  hooks/                  # truly shared hooks
  theme/                  # tokens: colors, spacing, typography, radii
  config/                 # env parsing (react-native-config values validated with Zod)
  types/                  # global types only
assets/
e2e/                      # Maestro flows
fastlane/                 # build, sign, and release lanes
```

- Screens MUST stay thin: read params, call their ViewModel, render components.
- Navigation param lists MUST be typed (`RootStackParamList` and similar), and `useNavigation` / `useRoute` MUST be used with those types.
- Features MUST NOT import another feature's internals, only its `index.ts`.
- Use path aliases (`@/features/...`, `@/components/...`) instead of deep relative imports.
- Naming: components `PascalCase.tsx`, hooks `useCamelCase.ts`, utilities `camelCase.ts`, constants `SCREAMING_SNAKE_CASE`. One component per file.

## React and Components

These rules add React Native specifics to `framework.react`:

- Use function components and hooks only. Class components are allowed only for error boundaries.
- `eslint-plugin-react-hooks` MUST be enabled with `rules-of-hooks` and `exhaustive-deps` set to error. Do not silence `exhaustive-deps`; fix the dependencies or restructure the code.
- Components and hooks MUST be pure. Never mutate props, state, or query data.
- The app root runs inside `<React.StrictMode>` in development.
- All text MUST be inside `<Text>`. Conditional rendering MUST use a ternary or an explicit boolean (`items.length > 0 && ...`), never `count && <X />`, because rendering a bare `0` or `''` crashes React Native.
- List keys MUST be stable and unique IDs from the data. Never use the array index for lists that can reorder, insert, or delete.
- Do not store derived values in state, and do not copy props into state. Reset state with a `key` when an identity changes.
- `useEffect` is only for synchronizing with external systems (native modules, `AppState`, `Keyboard`, `NetInfo`, `Linking`, timers, subscriptions). Do not use effects to fetch data, derive state, or respond to user events.
- Every effect that subscribes, listens, or starts a timer MUST return a cleanup function.
- Use `useFocusEffect` for work that must run when a screen gains focus, not on mount.
- Context is for low-frequency, app-wide values (theme, auth session, locale). Rapidly changing state belongs in store selectors.
- On React 19+, pass `ref` as a normal prop; do not add new `forwardRef` wrappers.
- Use named exports. Never define a component inside another component. Event handler props are named `onXxx`; their implementations `handleXxx`.
- Components SHOULD stay under about 200 lines.

## UI, Styling, and Accessibility

- Colors, spacing, and font sizes MUST come from theme tokens. Light and dark mode MUST both be supported.
- User-facing strings MUST use i18n keys.
- Use `SafeAreaView` / `useSafeAreaInsets` from `react-native-safe-area-context`, never the core `SafeAreaView`.
- Every async screen MUST handle loading, empty, error, and success states.
- Forms MUST remain usable with the keyboard open (`KeyboardAvoidingView` or `react-native-keyboard-controller`).

Accessibility (MUST):

- Every touchable has `accessibilityRole`, and an `accessibilityLabel` when it has no visible text.
- Minimum touch target is 44×44 pt (iOS) and 48×48 dp (Android); use `hitSlop` when the visual is smaller.
- Text respects system font scaling, and layouts do not break at the largest dynamic type size.
- Color contrast meets WCAG AA (4.5:1 for body text). Information is never conveyed by color alone.
- Screens are checked with VoiceOver and TalkBack before release.

## State and Data

- Server state MUST live in TanStack Query, not in Zustand, Context, or `useState`. Query data MUST NOT be copied into a store.
- Query keys MUST come from a per-feature key factory (for example, `userKeys.detail(id)`).
- Mutations invalidate or update the affected queries. Optimistic updates MUST have a rollback path.
- Client state starts as local `useState` and moves to Zustand only when distant screens need it. Read stores through selectors.
- A single API client in `src/lib/api` handles base URL, auth headers, token refresh, timeouts, and error normalization. Features MUST NOT call `fetch` directly.
- Every API response MUST be parsed with its Zod schema. A schema mismatch is reported to Sentry, not swallowed.
- Network errors map to typed, user-friendly errors. Raw error messages and stack traces MUST NOT be shown to users.
- The app MUST handle loss of connectivity gracefully: detect it with `@react-native-community/netinfo`, show the state, and retry. Persist the query cache when the product needs offline reads.

## Redux (When Chosen)

Redux is allowed only when the project records it in `PROJECT_INSTRUCTIONS.md`. When used, these rules replace the Zustand guidance above.

- Use Redux Toolkit only. Legacy Redux (`createStore`, hand-written action constants, `switch` reducers) MUST NOT be used. Redux replaces Zustand; never use both.
- One store, created with `configureStore` in `src/app/store.ts`, exporting `RootState` and `AppDispatch`.
- Use typed hooks only (`useAppDispatch`, `useAppSelector`). Only ViewModels read from or dispatch to the store.
- Server state still belongs in TanStack Query. If the project chooses RTK Query instead, it is used for all server state; never both.
- One slice per feature with `createSlice` in `features/<feature>/store/<feature>Slice.ts`. Reducers MUST be pure: no API calls, random values, `Date.now()`, dispatching, or navigation.
- Name actions as events (`loginSucceeded`), not setters. Normalize collections with `createEntityAdapter`.
- State MUST be serializable. Keep `serializableCheck` and `immutableCheck` enabled in development.
- Read state through selectors defined next to the slice. Derived data uses `createSelector`; a selector MUST NOT return a new object or array on every call.
- Async logic uses `createAsyncThunk` or the listener middleware, with typed errors via `rejectWithValue`. redux-saga and redux-observable need explicit approval.
- Persistence uses `redux-persist` with an MMKV adapter, a whitelist of slices, and `version` + `migrate`. Tokens, credentials, and PII MUST NOT be persisted through Redux.
- Reducers and selectors are tested as pure functions. ViewModel tests use a real store via a shared `renderWithProviders` helper.

## Native Projects

The `ios/` and `android/` folders are source code we own and are reviewed with the same rigor as TypeScript.

- Native changes MUST be deliberate and explained in the PR: what changed in Gradle, Podfile, `Info.plist`, `AndroidManifest.xml`, or entitlements, and why.
- A feature that needs native code MUST be implemented for both platforms, or the platform gap MUST be recorded in `PROJECT_INSTRUCTIONS.md`.
- Custom native modules MUST be Turbo Modules or Fabric components with a TypeScript Codegen spec in `src/native/`. No legacy bridge modules for new code.
- Build environments MUST be reproducible across machines and CI: pin Node (`.nvmrc`), Ruby (`.ruby-version` + `Gemfile`), JDK, Xcode, Android Gradle Plugin, and Kotlin versions.
- Build variants: Android product flavors and iOS schemes/configurations for `development`, `staging`, and `production`, each with its own bundle ID / application ID and `.env` file. `.env.*` files are provided by CI; only `.env.example` is committed.
- Signing material (keystores, `.p12`, provisioning profiles, production `google-services.json` / `GoogleService-Info.plist`) MUST NOT be committed. Use Fastlane `match` and CI secrets.
- Permissions in `Info.plist` (with usage descriptions) and `AndroidManifest.xml` MUST be the minimum needed, and every new permission is called out in the PR.
- Android release builds enable R8/ProGuard with maintained keep rules.
- After dependency changes, run `bundle exec pod install`. Never hand-edit `Pods/`, `build/`, or Codegen output.
- Gradle and CocoaPods dependencies use pinned versions; no dynamic `+` versions in Gradle.

## License Headers

Every source file we author MUST start with the Entgra license header. This covers `.ts`, `.tsx`, `.js`, `.kt`, `.swift`, `.m`, `.mm`, `.h`, `.java`, `.gradle(.kts)`, shell and Ruby scripts, and YAML we write (Maestro flows, CI workflows).

```ts
/*
 * Copyright (c) <YEAR>, Entgra (Pvt) Ltd. (https://www.entgra.io) All Rights Reserved.
 *
 * Entgra (Pvt) Ltd. licenses this file to you under the Apache License,
 * Version 2.0 (the "License"); you may not use this file except
 * in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing,
 * software distributed under the License is distributed on an
 * "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
 * KIND, either express or implied. See the License for the
 * specific language governing permissions and limitations
 * under the License.
 */
```

- `<YEAR>` is the year the file was created. When a file changes in a later year, update it to a range (`2025 - 2026`). Never remove the original year.
- Use the comment syntax of the language: `/* */` for TS/JS/Kotlin/Swift/Obj-C/Java/Gradle, `#` for shell/Ruby/YAML, `<!-- -->` for XML.
- Do not add the header to generated files (`Pods/`, `build/`, Codegen output), lockfiles, JSON, Markdown, or third-party code. Never change another party's copyright header.
- CI MUST enforce headers: an ESLint header rule for JS/TS and a header-check script for native and script files.

## Performance

- Long or dynamic lists MUST use FlashList with stable keys. Large lists MUST NOT be rendered with `ScrollView` + `.map()`.
- Use `React.memo`, `useMemo`, and `useCallback` where profiling or list rendering justifies it, not by reflex. When the React Compiler is enabled, rely on it and do not add manual memoization.
- Animations run on the UI thread via Reanimated worklets. No JS-driven animation loops.
- Images are sized correctly and cached. Multi-megabyte assets MUST NOT be bundled.
- Avoid chatty JS↔native calls in hot paths; batch on the native side or use JSI-based modules.
- Defer heavy work that delays first paint.
- `console.log` MUST NOT be committed; use the project logger, which is silenced in production.
- Projects set budgets in `PROJECT_INSTRUCTIONS.md` for cold start time, JS bundle size, and 60 fps on a mid-range Android device. Regressions SHOULD be caught before merge.

## Security and Privacy

These rules apply `universal.security` to mobile clients:

- Secrets, tokens, and credentials MUST be stored only via `react-native-keychain`. Never in AsyncStorage, MMKV, persisted Zustand/Redux state, `SharedPreferences` / `UserDefaults`, or plain files.
- The client bundle and native binary MUST NOT contain secrets. Everything in `react-native-config` `.env` files, `strings.xml`, and `Info.plist` is extractable.
- Android production builds disable `android:allowBackup` and `usesCleartextTraffic` unless justified, and set `android:exported` explicitly and minimally.
- iOS App Transport Security exceptions MUST NOT be added for production domains.
- Certificate pinning is decided per project in `PROJECT_INSTRUCTIONS.md`.
- PII, tokens, and full request/response bodies MUST NOT be logged or sent to analytics or Sentry.
- Deep links and push payloads are untrusted input: validate with Zod before navigating or acting on them.
- WebViews MUST allow-list origins, disable JavaScript unless needed, never inject untrusted content, and never expose native bridges to arbitrary pages.
- Permissions are requested just in time, with a clear explanation, and denial is handled gracefully.
- Sensitive screens SHOULD prevent screenshots and hide content in the app switcher where the product requires it.
- JS source maps, dSYMs, and ProGuard mapping files are uploaded to Sentry, not shipped in the app.

## Testing

These rules apply `universal.testing` to each MVVM layer:

- Model: services, schemas, business rules, and stores tested as plain functions, with the network mocked by MSW.
- ViewModel: every ViewModel is tested with `renderHook` for initial state, each action, and loading/error/empty paths, against real Model hooks with the network mocked.
- View: screens are tested with their ViewModel mocked, checking that each `state` renders correctly and user input calls the right `action`.
- Components are tested with React Native Testing Library, querying by role, label, or text. `testID` is a last resort except for Maestro selectors.
- Network is mocked at the network layer, not by mocking individual hooks.
- Our own native modules have JUnit / XCTest coverage for non-trivial logic.
- Tests control timers and dates (`jest.useFakeTimers`, fixed clocks).
- Critical journeys (login, onboarding, primary flows, payments) MUST have Maestro coverage.
- The coverage target for `src/features/**` and `src/lib/**` is set in `PROJECT_INSTRUCTIONS.md` (suggested ≥ 80% lines for logic).

## E2E Testing with Maestro

Structure:

```
e2e/
  config.yaml             # workspace config: which flows to run, default tags
  flows/
    <feature>/            # one folder per feature, mirroring src/features/
      login-success.yaml
      login-invalid-password.yaml
  subflows/               # reusable steps: login, logout, permission dialogs
  scripts/                # JS helpers for runScript (test data setup)
```

Example flow:

```yaml
appId: ${APP_ID}
tags:
  - smoke
  - auth
---
- launchApp:
    clearState: true
- runFlow:
    file: ../../subflows/login.yaml
    env:
      USERNAME: ${TEST_USERNAME}
      PASSWORD: ${TEST_PASSWORD}
- extendedWaitUntil:
    visible:
      id: "home.screen"
    timeout: 15000
- assertVisible:
    id: "home.welcomeBanner"
```

- One flow per user journey, named `<action>-<outcome>.yaml`. A change to a critical journey MUST add or update its flow in the same PR.
- Shared steps live in `subflows/` and are reused with `runFlow`, never copy-pasted.
- `appId` MUST come from the `APP_ID` variable. Bundle and application IDs MUST NOT be hard-coded.
- Every flow MUST be independent: start with `launchApp: clearState: true` and create its own state. Flows MUST NOT depend on run order.
- Elements a flow uses MUST have a `testID` named `<feature>.<element>` and be selected with `id:`. Selecting by visible text is allowed only for system UI and permission dialogs.
- Fixed waits MUST NOT be used. Use `extendedWaitUntil` or `waitForAnimationToEnd`.
- Every flow MUST end with an assertion of the expected outcome.
- Cover the happy path and key failure paths (invalid input, permission denied, no network).
- Platform differences use conditional subflows (`when: platform: Android`), not duplicated flows.
- Flows run in a fixed locale and region.
- Credentials come from CI secrets via `-e`. Real user accounts MUST NOT appear in YAML. Flows run against a test backend or mock server, never production data.

Tags:

| Tag | Meaning | Runs |
|---|---|---|
| `smoke` | Core journeys: launch, login, primary flow | Every PR (Android emulator) |
| `regression` | Full journey coverage | Nightly and before every release, Android and iOS |
| `android-only` / `ios-only` | Platform-specific flows | Matching platform only |
| `flaky` | Quarantined | Never gates; MUST have a linked issue and be fixed within two weeks |

- CI runs `maestro test e2e/flows --include-tags smoke --format junit --output e2e-report.xml`. Release gates run `regression` on both platforms against release builds.
- The Maestro CLI version is pinned in CI. On failure, CI uploads the JUnit report, screenshots, and recordings.
- A flaky flow is fixed or quarantined, never retried until green or deleted to make a build pass. Removing a flow needs tech lead approval.

## Errors and Observability

These rules apply `universal.observability` to mobile clients:

- The app root and each top-level navigator or feature stack MUST be wrapped in an error boundary with a recoverable fallback UI.
- Unhandled promise rejections and native crashes are captured by Sentry, tagged with release and build number.
- User-facing error messages are human, actionable, and localized.
- Analytics events have a typed schema and contain no PII.

## Builds, Releases, and Upgrades

- CI MUST pass before merge: `tsc --noEmit`, ESLint with zero warnings, Prettier check, unit/component tests, license header check, Maestro `smoke` flows, and Android and iOS builds whenever `android/`, `ios/`, or native dependencies change.
- UI PRs include screenshots or recordings from both iOS and Android.
- The app version follows SemVer; build numbers increase monotonically and are set by the release lane, not by hand.
- Builds, signing, and store uploads MUST run through Fastlane lanes in CI, never from a developer machine for production.
- Store releases use staged rollouts (Play staged rollout, App Store phased release).
- If an OTA solution is approved, it is for JS and asset changes only. Any native change (new native module, permission, React Native upgrade, Gradle or Podfile change) MUST ship as a full store build, and OTA bundles MUST target a specific native binary version.
- React Native SHOULD stay within two minor versions of latest stable. Upgrades happen on a dedicated branch using the [React Native Upgrade Helper](https://react-native-community.github.io/upgrade-helper/), followed by the full E2E suite on both platforms.
- This module SHOULD be reviewed at every React Native upgrade.

## AI Coding Agents

In addition to `universal.ai-assisted-development`, AI coding agents working in React Native projects MUST:

- Read this module, `PROJECT_INSTRUCTIONS.md`, and the relevant existing code before proposing changes.
- Follow MVVM: new screens get a View, a `use<Screen>ViewModel`, and Model code in the right folders.
- Verify library APIs against the versions in `package.json` and their installed types. React Native and library APIs change between versions.
- Not add dependencies without asking, and never suggest Expo packages or Expo-only APIs.
- Keep edits to `ios/` and `android/` minimal, make them for both platforms, list them in the summary, and never touch generated files or signing configuration.
- Run type-check, lint, and relevant tests before declaring work done, and report failures with output.
- Never weaken safeguards: no disabled lint rules, skipped tests, `@ts-ignore`, loosened `tsconfig`, or bypassed validation.
- Add the Entgra license header to every new source file.
- Raise a conflict with this constitution instead of silently working around it.

## Project-Specific Decisions

Each React Native project records these in `PROJECT_INSTRUCTIONS.md` at project start. Shared modules are not edited for a single project's convenience.

| Decision | Value |
|---|---|
| React Native version | |
| Node / Ruby / JDK / Xcode versions | |
| Minimum iOS / Android `minSdkVersion` / `targetSdkVersion` | |
| Bundle ID / application ID per environment | |
| OTA updates (none / approved solution) | |
| Styling (StyleSheet + tokens / NativeWind) | |
| React Compiler enabled? | |
| Client state (Zustand / Redux Toolkit) and server state (TanStack Query / RTK Query) | |
| Package manager | |
| Auth method (OAuth/OIDC, SSO, custom) | |
| Certificate pinning required? | |
| Offline requirements | |
| Supported languages / RTL | |
| Coverage target | |
| Maestro test backend and test accounts | |
| Performance budgets (cold start, bundle size) | |
| MDM / enterprise distribution requirements | |
| Other deviations from this constitution (with reason) | |

## React Native PR Checklist

- [ ] Works on iOS and Android, in a release build if native code changed.
- [ ] Loading, empty, error, and offline states handled.
- [ ] Light and dark mode correct; strings localized; no hard-coded colors or sizes.
- [ ] Accessibility roles and labels set; works with the largest text size.
- [ ] MVVM respected: View renders only, ViewModel returns `{ state, actions }`, no View imports from the Model.
- [ ] Strict types, no `any`; boundary data validated with Zod.
- [ ] No hooks-lint errors; every subscribing effect cleans up.
- [ ] Linked accepted requirements: issue or specification stating the problem and affected users, expected behavior and acceptance criteria, error, offline, and permission cases, and iOS/Android differences, with who accepted it and which version (`universal.engineering-principles`).
- [ ] Tests added: unit/component tests, and a Maestro flow added or updated for critical journeys.
- [ ] Tests executed: linked CI run or test report naming the commands or procedures run, the commit or app build, the iOS/Android versions and device or simulator used, which checks passed, failed, were skipped, or could not run, and remaining limitations with an owner (`universal.testing`).
- [ ] Native, permission, and dependency changes listed in the PR description.
- [ ] No secrets, PII, or `console.log` added.
- [ ] Every new source file has the Entgra license header.
- [ ] Documentation impact: linked updated specifications, API contracts, setup instructions, or release guidance, or the reason no update is needed (`universal.documentation`).
- [ ] Independent human reviewer accepted the code and test evidence.
- [ ] Team lead accepted the delivered user behavior, with stakeholder acceptance where required.
- [ ] Release owner approved the exact app build being released, including its store or MDM rollout and recovery plan.
- [ ] Each approval records the approver, decision, date, build/version, and evidence; AI reviews and CI do not replace it, and material changes after approval are re-approved (`universal.engineering-principles`).

## Verification

Reviewers check both-platform evidence, MVVM import boundaries, boundary validation, async and offline state handling, accessibility, secure storage choices, native and permission changes, license headers, unit and Maestro coverage for changed journeys, CI results for type-check, lint, tests, and native builds, linked requirements, evidence of tests executed on named iOS and Android targets, documentation impact, and recorded human approvals.
