# CLAUDE.md

Kotlin Multiplatform Material You recurring-expense tracker.
Upstream: https://github.com/DennisBauer/RecurringExpenseTracker

**Canonical agent instructions:** [AGENTS.md](./AGENTS.md) — read it first and keep both files aligned when updating guidance.

## Quick commands

```bash
./gradlew ktlintCheck
./gradlew ktlintFormat
./gradlew :shared:allTests
./gradlew :app:assembleDebug
./gradlew build
```

Java 21. CI runs `ktlintCheck` then `build`, and rejects new Kotlin warnings outside `.github/compiler-warnings-baseline.txt`.

## Orientation

- Product logic/UI lives in `:shared` (`commonMain`). Android shell (widgets, alarms, biometric, backup) in `:app`.
- DI: Koin — `sharedModule` + `platformModule` + `appModule`.
- DB: Room 3 in `RecurringExpenseDatabase` (v11); migrations + `shared/schemas/` must update together.
- Prefs: KSafe. Exchange rates: bundled JSON (offline; no phone-home).
- Tests: `shared/src/commonTest` and `jvmTest` (migrations).

## Working rules

- Match existing patterns; minimal diffs; no drive-by refactors or dependency bumps.
- Run ktlint + relevant tests before considering work done.
- Negative expense amounts are a feature (income / net funds).
- Do not add analytics/tracking or network sync for user data.
- Prefer upstream issues/PRs; translations via Weblate when possible.
