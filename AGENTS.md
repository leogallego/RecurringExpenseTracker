# Recurring Expense Tracker — Agent Guide

Material You recurring-expense tracker. Kotlin Multiplatform + Compose Multiplatform.
Upstream: https://github.com/DennisBauer/RecurringExpenseTracker (GPL-3.0).
This clone’s `origin` is typically a personal fork; contribute via PR to upstream `main`.

## Stack

| Piece | Detail |
|-------|--------|
| Language | Kotlin 2.4.x, Java 21 toolchain |
| UI | Compose Multiplatform / Material 3 |
| Modules | `:shared` (KMP), `:app` (Android shell) |
| DI | Koin (`sharedModule` + `platformModule` + Android `appModule`) |
| DB | Room 3 (`RecurringExpenseDatabase`, version 11), schemas in `shared/schemas/` |
| Prefs | KSafe (`KSafeUserPreferencesRepository`) |
| Targets | Android (primary), JVM desktop, iOS (local only) |
| Package | `de.dbauer.expensetracker` / `.shared` |
| App ID | `de.dbauer.expensetracker` — version in `app/build.gradle.kts` |

## Commands

```bash
./gradlew ktlintCheck          # required by CI; run before commit
./gradlew ktlintFormat         # auto-fix style
./gradlew :shared:allTests     # unit tests (commonTest + jvmTest)
./gradlew :app:assembleDebug   # Android debug APK
./gradlew build                # full build (what CI runs after ktlint)
./gradlew :shared:run          # desktop JVM app
```

Java **21** required. Open as a normal Android/Gradle project in Android Studio / IDEA.

## Architecture

```
app/          Android-only: Application, MainActivity, widgets (Glance),
              notification receivers/alarms, biometric lock, backup/restore
shared/       Almost all product code
  commonMain/   UI, ViewModels, repository, Room entities/DAO, domain
  androidMain/  expect/actual (DB builder, theme, currency, platformModule)
  jvmMain/      Desktop entry (`Main.kt`) + actuals
  iosMain/      iOS entry (`MainViewController`) + actuals
  commonTest/   Most unit tests
  jvmTest/      Room migration + legacy prefs migration tests
iosApp/       Xcode wrapper around shared framework
fastlane/     Store metadata / screenshots
.github/      PR pipeline, release, compiler-warning baseline
```

Layering inside `shared`:

- `ui/` — Compose screens (home, edit expense, upcoming, settings, tags, about)
- `viewmodel/` — screen state
- `model/database/` — Room + `ExpenseRepository` / `IExpenseRepository`
- `model/datastore/` — user preferences
- `model/` — date math, exchange rates, notifications manager
- `data/` — domain models (`RecurringExpenseData`, currencies, tags)
- `di/Module.kt` — Koin wiring; platform bits via `expect val platformModule`

## Where to change what

| Task | Start here |
|------|------------|
| Expense CRUD / recurrence math | `ExpenseRepository`, `DateTimeCalculator`, `RecurringExpenseData` |
| Upcoming payments | `UpcomingPaymentsViewModel`, `UpcomingPaymentsExpander` |
| Reminders / notification *logic* | `ExpenseNotificationManager` (shared); Android delivery in `app/.../notification/` |
| Settings / default currency | `SettingsViewModel`, `IUserPreferencesRepository` |
| Tags | `TagsScreenViewModel`, tag entities/DAO |
| Home widget | `app/.../widget/` (Android only) |
| Biometrics / backup | `app/.../security/`, `DatabaseBackupRestore` |
| Strings / i18n | `shared/.../composeResources/values*/`; community translations via Weblate |
| Exchange rates | Bundled JSON under compose resources; updater script `.github/scripts/update_exchange_rates.py` |
| DB schema | Entities + migrations in `RecurringExpenseDatabase.kt`; export schema under `shared/schemas/` |

## Code style

- `.editorconfig` + ktlint (+ Compose rules). Max line length **115** for `.kt`.
- Trailing commas preferred; star imports discouraged (high threshold).
- Material 2 disallowed in Compose except icons.
- Prefer existing patterns: ViewModel + repository interfaces, fakes for previews/tests (`FakeExpenseRepository`, `previewModule`).
- Match surrounding naming/packages; do not invent parallel architecture.

## Testing & CI

PR pipeline (`.github/workflows/pr_pipeline.yml`):

1. `./gradlew ktlintCheck`
2. `./gradlew build`
3. Compiler warnings must stay within `.github/compiler-warnings-baseline.txt` (see `.github/check-compiler-warnings.sh`)

Write/extend tests next to existing ones in `commonTest` / `jvmTest`. Prefer kotlin-test + coroutines-test. DB migration tests belong in `jvmTest`.

## Gotchas

- **Privacy / offline**: No analytics, no remote expense sync. Exchange rates are **bundled resources**, not fetched at runtime. Do not add network telemetry.
- **Room migrations**: Bump `@Database(version)`, add `Migration`, and update schema JSON under `shared/schemas/`. Destructive downgrade is allowed; upgrades must migrate.
- **expect/actual**: Platform APIs use expect in `commonMain` with actuals per target. Add all platforms when introducing new expects.
- **Android-only features** (widget, AlarmManager notifications, biometric, SAF backup) stay in `:app`, not `commonMain`.
- **Negative amounts** are intentional (income / net-funds tracking). Don’t “fix” by clamping to positive.
- **iOS CI is disabled**; desktop + Android are what CI builds.
- **Signing**: Release signing uses CI env secrets; local release falls back to debug keystore.
- **Renovate** manages dependency bumps on upstream — avoid drive-by version churn in feature PRs.

## Contribution workflow

1. Track upstream: `git remote add upstream git@github.com:DennisBauer/RecurringExpenseTracker.git` (if missing).
2. **Fork-only files:** `AGENTS.md` / `CLAUDE.md` live on this fork’s `main` for multi-device sync. Do **not** include them in upstream PRs — always cut contribution branches from `upstream/main` (`git checkout -b my-fix upstream/main`), not from fork `main`.
3. Keep PRs focused; run `ktlintCheck` + relevant tests locally.
4. Prefer linking an upstream issue for bugs/features.
5. Translations: Weblate preferred over hand-editing every locale unless you own the string change in `values/`.
6. Sync fork: `git checkout main && git fetch upstream && git merge upstream/main` (docs stay on fork `main`).

## Out of scope for casual changes

- Do not commit secrets, keystores, or `local.properties`.
- Do not expand privacy surface (tracking SDKs, crash reporters that phone home) without explicit product decision.
- Do not rewrite module layout or replace Koin/Room/Compose without a clear, agreed reason.
