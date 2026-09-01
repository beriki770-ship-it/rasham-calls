# HANDOFF — Rasham Calls

_Verified: 2026-09-01._

Hardened personal fork of **ShizuCallRecorder** (kitsumed/ShizuCallRecorder, GPL-3.0).
Records both sides of a phone call on a non-rooted phone via **Shizuku** (ADB-shell
privilege) + a bundled **scrcpy-server**. Everything stays on-device — no INTERNET.

Repo: `beriki770-ship-it/rasham-calls` · app id `com.beri.rashamcalls` · "Rasham Calls".

## State
Builds **green in CI**; a debug APK is published as release **v0.1** and manifest-verified.
**Not yet runtime-tested on a device.** The capture/detection path was left untouched, so
it should behave like upstream, but nobody has installed v0.1 and recorded a call yet.

Verified on the built APK (not just source): no `INTERNET`; `READ_CONTACTS`,
`READ_CALL_LOG`, `QUERY_ALL_PACKAGES`, `SYSTEM_ALERT_WINDOW` all absent;
`assets/scrcpy-server` present; app id `com.beri.rashamcalls`.

## What changed this session (2026-09-01)
Forked upstream, then hardened (commit d6cdcbf + CI fix ccb9c64):
- Dropped 4 permissions → contacts ignore-filter, recording overlay, and the **whole
  PhoneState detection branch** removed; **InCallService is now the only detection mode**,
  `minSdk 30 → 31`.
- Deleted 7 files: ContactSelectionDialog, ContactPickerViewModel, RecordingOverlayController,
  RecordingOverlay, PhoneState{Receiver,SessionManager,TemporaryCache}.
- Defaults flipped: auto-record incoming + outgoing = **true** (AppPreferences `DefaultsValue`
  L50/51). Audio source was already `voice-call` (both sides).
- Rebranded: applicationId `com.beri.rashamcalls`, app_name "Rasham Calls" (namespace/Kotlin
  packages left as `com.kitsumed.shizucallrecorder` — do NOT rename, it's wired into the
  Shizuku provider authority).
- Added `.github/workflows/build.yml` (CI build, no local toolchain needed).

## What's left
1. **Install v0.1, grant Shizuku + perms, make an earpiece test call, confirm both sides
    record.** This is the only unverified step.
2. Optional Phase 2 (not security, just cleanup): remove Sponsor/donate, Debug screen,
   about-libraries, language/theme selectors. See the upstream trim map — these were
   deliberately deferred.
3. Residual (harmless, warnings only): 3 debug buttons are now no-ops (their PhoneState
   backend was deleted); a few unused string resources and dead detection-mode callbacks
   remain. No `allWarningsAsErrors`, so they don't fail the build.

## Build / release
No Android toolchain on the dev PC — **build in CI**:
```
git push            # triggers .github/workflows/build.yml
gh run download -n rasham-calls-debug-apk -D dist
```
Rebuild without a code change: `gh workflow run "Build APK"`.

## Gotchas
- **CI needs `SKIP_SIGNING: "true"`** (already set in the workflow). The fork's `ci-release`
  signing config throws at configuration time on a missing `KEYSTORE_FILE` otherwise —
  even for `assembleDebug`.
- **Bleeding-edge toolchain**: AGP 9.2.1 / Gradle 9.4.1 / Kotlin 2.3.21 / compileSdk 36 /
  JDK 21. CI installs `platforms;android-36` + `build-tools;36.0.0`.
- **scrcpy-server is downloaded at build time** by a Gradle task (SHA256-pinned) — the CI
  runner needs network. It ends up at `assets/scrcpy-server` and is SHA-validated before
  every recording; do not strip it.
- **Runtime needs Shizuku running** (Wireless-debugging start) AND OnePlus/Oppo needs
  Developer options → "Disable system optimization" ON (renamed "Disable permission
  monitoring"). Loudspeaker drops the far side — record in earpiece mode.
- **GPL-3.0**: if this is ever distributed, source must stay public + upstream credited.
- APK is ~62 MB (scrcpy-server + libphonenumber metadata) — too big to send over chat;
  distribute via the GitHub release.

## Key files (KEEP — the capture path)
`services/recording/*`, `services/shell/*` (ShellAudioPipeline, ShellService),
`integrations/shizuku/*`, `integrations/scrcpy/*`, `aidl/**/IShellService.aidl`,
`services/callDetection/incall/InCallService.kt`, `system/storage/SafHelper.kt`.
