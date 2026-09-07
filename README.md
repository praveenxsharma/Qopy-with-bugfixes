<div align="center">

<img align="center" width="128px" src="src-tauri/icons/icon.png" />
<h1 align="center"><b>Qopy</b></h1>

A fixed clipboard manager for Windows.

<a href="https://github.com/praveenxsharma/Qopy/releases">
  <img src="./public/windows.png"> Windows (x64) — latest release
</a>

</div>

## What is Qopy

Qopy is a clipboard manager designed as a simple alternative to the standard clipboard on Windows. This fork is focused on fixing and hardening the Windows experience.

## What's different from the original

This fork ([praveenxsharma/Qopy](https://github.com/praveenxsharma/Qopy)) addresses several issues in the upstream ([0PandaDEV/Qopy](https://github.com/0PandaDEV/Qopy)):

| Fix | Details |
|-----|---------|
| **Panics on startup** | `tauri-plugin-apps-rs` panic when App Info was missing; prevented the window from opening on first launch or after a fresh install. |
| **File paste (Ctrl+V) broken** | `write_files` never matched `"file"` content type on Windows — the Rust match arm only handled the array case. Now `write_and_paste` accepts `"file"` and `"files"` via the plugin's `write_files_uris`. |
| **Image section empty** | Images were the oldest items (newest image was item #51 of 6719). The app only loaded the 50 newest items, so the Image filter always returned nothing. Fixed by pushing the content-type filter down to the SQL query so the DB returns filtered chunks directly. |
| **Actions menu clipped** | The menu was positioned using a size measured during the enter transition (`scale(0.95)`), so clamping used a ~5%-too-small height and the bottom of the menu got cut off at the window edge. Uses layout dimensions (`offsetWidth`/`offsetHeight`) instead. |
| **Flickering toggle** | Opening the Actions menu flickered (mousedown-outside closed it, then the button's click re-opened it). The trigger button is now excluded from the outside-click handler. |
| **Missing pinned/title on reload** | `processHistoryRows` now carries `pinned` and `title` from the DB row into the client-side item. |

## Compatibility & testing

> [!IMPORTANT]
> Only the **Windows x86_64** build is tested and supported for now. **No other build** (Windows arm64, Linux, macOS) is tested. If you hit a problem on any of them, please report it — but expect that it may not reproduce or be fixed.

Supported:
- Windows 10 / 11 — x86_64 (x64) only

Not tested (please report issues, but no guarantees):
- Windows on ARM (arm64)
- Linux (deb / rpm / AppImage)
- macOS (Silicon / Intel)

## Reporting issues

If you find any issue, please create one at [github.com/praveenxsharma/Qopy/issues](https://github.com/praveenxsharma/Qopy/issues/) with the following info:

### Required — always include

- **Qopy version** (bottom of the app window, or check the installed package: `Qopy-<version>_x64.msi`)
- **Windows version** — e.g. `Windows 11 24H2` / `Windows 10 22H2` (see Win+R → `winver`)
- **Architecture** — x64, arm64 (`open System` → Device specs → System type)
- **Qopy source** — installed MSI from releases, or self-built from `main` (give the commit hash if self-built)
- **Steps to reproduce** — numbered, starting from fresh state
- **Expected vs actual behavior** — what you expected, and what actually happened
- **Screenshots or screen recording** — where useful

### Recommended — helps a lot

- **Copy of the log file** — located at:
  - `%APPDATA%\net.pandadev.qopy\logs\app.log`
  - Older logs are rotated alongside it (`app.YYYY-MM-DD.log`); include them too if present.
- **Copy of the database** (for corruption/data-loss issues only) — located at:
  - `%APPDATA%\net.pandadev.qopy\data.db`
  - **Do not** include this publicly — attach it only if an issue is requesting it, and be aware it contains your clipboard history (credentials, etc.). Redact or scrub it first if possible.
- **Keyboard layout/language** — if the issue involves hotkeys (default: `Windows+V`), the global hotkey, or typing/pasting, include your active keyboard layout (e.g. `US QWERTY`, `French AZERTY`, `Devanagari INSCRIPT`) and locale (`en-US`, `hi-IN`, …). Hotkey capture is input-locale sensitive.
- **Clipboard manager / conflict info** — if you use another clipboard manager or the built-in `Windows+V` clipboard history alongside Qopy, mention it.
- **Antivirus / DLP / corporate software** — if the issue is paste-related (files failing to paste, Qt/WinUI apps, etc.), note any AV, endpoint protection, or remote-desktop software running (e.g. Defender, Crowdstrike, TeamViewer RDP) — these commonly block synthetic `Ctrl+V`.
- **App behavior on empty/trailing cases** — for empty-history, missing-image, or blank-filter issues: roughly how many entries you had, and whether the history was recently cleared.
- **Does it reproduce after reboot?** — state whether a restart of Qopy, or of Windows, changes anything.

### When creating the issue

1. Press `Win+V` → check the version, then `winver` for the OS build.
2. Try to reproduce once more on the **latest release** before filing (old installs silently keep previous versions).
3. Paste the log file contents (or a link to a [gist](https://gist.github.com)) rather than a screenshot of text.
4. **Privacy check** — Qopy stores your real clipboard history (passwords, tokens, etc.) in `data.db`. Never attach it publicly; if the issue requires it, switch to a fresh/empty history first or scrub sensitive entries before sharing.

## Local development

You'll need [Rust](https://rustup.rs/) and [bun](https://bun.sh/) installed.

```sh
git clone https://github.com/praveenxsharma/Qopy.git
cd Qopy
bun i
bun dev
```

## Building for production

```sh
bun build
```

> [!NOTE]
> The build will fail at the end because it cannot detect a signing key, but the installer files are generated regardless. Find them in `src-tauri/target/release/bundle`.

## License

Qopy is licensed under AGPL-3. See the [LICENSE file](./LICENSE) for more information.
