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
