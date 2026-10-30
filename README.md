# Learning Garden

Offline-first Android prototype for Grades 3–5 in English, Hindi and Marathi. Kotlin, Jetpack Compose Material 3, one `app` module, minSdk 24, compile SDK 37.1 / target SDK 37. No INTERNET or storage permission, no accounts, no leaderboards, no startup network calls.

## Run

Install Android Studio, SDK platform 37.1 and build tools 36+, and use JDK 21 or 25. Set `sdk.dir` in your untracked `local.properties` (or set `ANDROID_HOME`). Dependencies need internet only for the first build.

```sh
./gradlew assembleDebug
./gradlew installDebug
```

Choose a language, create a learner and start the first lesson of a subject. Home is always one tap away. Complete the previous lesson to unlock the next. Switch profiles from Home. Tap Listen to use a locally installed TTS voice; missing voices are silently skipped. All lesson text stays available.

Long-press the app name to open Researcher mode. Demo PIN: **2468**. CSV export uses the Android document picker. It contains local pseudonymous learner UUIDs and event payloads, never learner names. The PIN is an observation-study convenience, not production security.

Settings includes explicit simulated sync. It requires a connected network due to WorkManager constraints but never contacts a server. FakeSyncApi holds data in memory, merges by UUID and deliberately fails every third call. Downloads installs an optional demo by copying bundled English math JSON after confirmation and verifying its checksum. It uses no mobile data. All core language packs are installed with the APK.

## Verify

```sh
python3 tools/validate_content.py
./gradlew assembleDebug assembleRelease testDebugUnitTest lintDebug
python3 tools/report_apk.py
# Start an emulator, then:
./gradlew connectedDebugAndroidTest
```

Instrumentation completes all four question types in all three languages at 1.5× text size, checks bilingual rewards, verifies actual Room close/reopen persistence, and rejects a corrupt pack. `tools/device_smoke.py` additionally verifies a real force-stop on a fresh emulator after `installDebug`. For repeatable UI tests, onboarding is reset by the test; use an emulator without valuable learner data. Airplane mode is compatible with all learning tests. APKs are in `app/build/outputs/apk/`. The release APK is unsigned; use your own signing configuration for distribution. Never distribute a debug signing key as a production key.

## Structure

- `feature/`: Compose screens, ViewModel, event handlers and platform audio/export adapters.
- `domain/`: pure Kotlin content models, repository ports, activity reducer, gamification, merge policy and research calculations.
- `data/`: Room entities and transactional repository, DataStore preferences, bundled content, pack installer and WorkManager fake sync.
- `AppContainer`: manual wiring of ports and implementations; no DI framework.

State flows down from ViewModel; UI actions invoke repository-backed mutations. Room stores the full lesson reducer state, including attempts, feedback, timing, result and survey completion. Every answer and its XP entry commit atomically. Completion is idempotent by run UUID. Per-run progress is separate from best-ever lesson scores. Profiles keep independent sessions. Locale recreation and force-stop reload the durable session; select Continue lesson on Home.

See [assumptions](docs/ASSUMPTIONS.md), [content guide](docs/content-guidelines.md), [phase gates](docs/phase-verification.md), [performance budget](docs/perf-budget.md), [evaluation plan](docs/evaluation-plan.md), and [architecture](docs/architecture.md). Machine-authored content is explicitly unreviewed and needs educator review before classroom use.
