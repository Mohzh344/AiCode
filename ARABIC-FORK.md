# AiCode AR — Arabic fork

Independent build of [AiCode](https://github.com/jieapi/AiCode) maintained in this
repository (`Mohzh344/AiCode`). It adds a full Arabic interface, proper RTL support and a
chat-only RTL switch, and it is shipped under its own applicationId and signing key so it
can live next to the upstream app.

## What this fork changes

### 1. Arabic locale (complete)
* `app/src/main/res/values-ar/strings.xml` — all **2232** UI strings translated from the
  Simplified-Chinese source into Modern Standard Arabic, same keys, same order.
* `app/src/main/res/xml/locales_config.xml` — `ar` registered for per-app language.
* `core/util/LanguageRegistry.kt` — `العربية` added to the in-app language picker.
* Android already had `android:supportsRtl="true"`, so the whole app mirrors correctly in
  Arabic; CodeMirror-style editors and code blocks stay LTR (see below).

Translation rules used: format placeholders (`%1$s`, `%2$d`) and escapes (`\n`, `\'`,
`\@`) preserved verbatim, technical terms (SSH, MCP, Git, Token, API, …) kept in Latin
script, brand names never transliterated.

### 2. Chat-only RTL switch
Settings → **General → Reply → "Right-to-left chat layout"**
(`settings_chat_rtl`). When on, the chat panel — message list, bubbles, input bar and the
dialogs opened from it — is laid out right-to-left *even if the app itself is in English*.
When off, the chat simply follows the app language. Code blocks inside messages always
stay LTR so code is never mirrored.

* storage: `GeneralSettingsRepository.chatRtlFlow` (`chat_rtl` preference)
* wiring: `SettingsViewModel.chatRtl` / `AIAgentViewModel.chatRtl`
* applied in `AIChatPanel` via `LocalLayoutDirection`, and re-set to LTR for fenced code
  blocks in `MarkdownContent.kt`

### 3. Independent release identity
* `applicationIdSuffix = ".ar"` on the release build type → **`com.aicode.ar`**, installs
  side by side with the upstream `com.aicode` app.
* Update check, About page and the star prompt point at this repository's GitHub Releases.
* Signed with this fork's own keystore (`app/aicode.jks`, git-ignored; the CI copy lives in
  the `AICODE_KEYSTORE_BASE64` / `AICODE_KEYSTORE_PASSWORD` / `AICODE_KEY_ALIAS` /
  `AICODE_KEY_PASSWORD` secrets). Because the signing key differs from upstream, Android
  will not update an app across the two — that is what the `.ar` suffix avoids.

## Building

Releases are built by GitHub Actions (`.github/workflows/android-release.yml`):

```bash
git tag vX.Y.Z-ar.N && git push origin vX.Y.Z-ar.N
```

Three signed APKs are published on the release page:

| asset | ABI | note |
|-------|-----|------|
| `aicode-arm64-*.apk` | arm64-v8a | phone build, the one to install |
| `aicode-universal-*.apk` | arm64-v8a + x86_64 | largest |
| `aicode-x86_64-*.apk` | x86_64 | emulator / Chromebook |

Run the plain CI check locally with `./gradlew :app:testUniversalDebugUnitTest`.

## Known upstream strings that are still Chinese in English/Arabic mode

These live in Kotlin literals rather than `strings.xml`, so they are not covered by the
resource translation yet (tracked as follow-up work):

* `feature/agent/presentation/run/AgentRunManager.kt` — the goal/goal-status texts and the
  usage table injected into the chat (`| 项目 | 今日 | 累计 |`, `已设定目标:…`, `当前目标(…):…`).
* a handful of plugin / backup / log-viewer helpers
  (`ProjectToolsDrawer`, `BackupManagerImpl`, `PluginContent`, crash-report reasons).
* AI request logs and the container-side helper scripts (developer-facing only).

Fix for these means moving the literals into `strings.xml` with `en`/`ar` values — the
resource plumbing for Arabic and RTL is already in place.

## Credits

Upstream project: <https://github.com/jieapi/AiCode> (GPL-3.0). This fork keeps the same
licence; all changes above are original to this repository.
