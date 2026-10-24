# gfn-tizen-desktop

A single-file [TizenBrew](https://github.com/reisxd/TizenBrew) site-modification
module that runs the GeForce NOW web client (`play.geforcenow.com`) on a Samsung
Tizen TV while presenting itself as desktop Chrome on Windows.

This fork adds input fixes for TV keyboards: F-key diagnostics/capture/remap and
Esc pointer-lock recovery. The base module is
[`Julianbjrk/gfn-tizen-desktop`](https://github.com/Julianbjrk/gfn-tizen-desktop)
v0.3.1 (MIT).

Unofficial and for personal use. Not affiliated with NVIDIA or Samsung. Changing
how the browser identifies itself may conflict with the GeForce NOW terms of use.
Use at your own risk.

## Fork changes: v0.3.2-gfn

- **F keys.** Tries to register `F1`–`F12` / `KEY_F1`–`KEY_F12` with Tizen's
  input API. If an F key reaches the page, its TV hotkey default is cancelled
  without stopping the key from reaching the game.
- **F-key remap.** `Ctrl/Alt + 1..0,-,=` sends `F1..F12` to the GFN client.
  **Yellow** toggles it. This is for games whose actions are bound to F keys
  that the TV consumes.
- **Esc recovery.** Esc is left alone so the game receives it. After the browser
  releases pointer lock, the next keypress or click in the video requests it
  again, so you should not need a separate click.
- **TV Back.** `keyCode 10009` is cancelled and logged so the TV does not take
  focus, if the key is delivered to the page.
- **Diagnostics.** The blue panel now shows the last key (`key`, `code`,
  `keyCode`, trusted), how many F keys arrived, the remap count, the re-lock
  count, the Tizen registration result, and window blur/visibility events.
- **Green** now toggles both the pointer-lock keeper and the Esc auto-relock.

1440p120 is confirmed on an S95C with this module. 4K is conditional: it depends
on WebRTC H.265/AV1, hardware decode, ~45 Mbps and NVIDIA's browser entitlement.

## Install

1. Install TizenBrew on the TV.
2. In TizenBrew open **Module Manager → Add Module → GitHub** and enter:

   ```
   tngranados/gfn-tizen-desktop@v0.3.2-gfn
   ```

   TizenBrew's GitHub type prepends `gh/`; the full module ID becomes
   `gh/tngranados/gfn-tizen-desktop@v0.3.2-gfn`. Use the tag, not `@main`;
   jsDelivr may serve a cached copy of `main`.
3. Launch **GFN Desktop** from the TizenBrew dashboard.

Mouse support needs TizenBrew to be packaged with
`<tizen:setting pointing-device-support="enable"/>`; Samsung disables the
pointer for apps that do not declare it (2021+ models). If a mouse does not
appear, patch TizenBrew's `config.xml` and reinstall it.

## Controls

| Button | Action |
| --- | --- |
| **blue** | show/hide the diagnostics panel |
| **red** | 4K override (relaunch, then pick 3840×2160 in GFN) |
| **green** | pointer-lock keeper + Esc auto-relock on/off |
| **yellow** | F-key remap on/off |

## F keys: what to expect

Press **blue** and look at:

```
Tangentbord:    F1–F12 0 · omap 3 · relås 2 · 0 ok / 24 fel · omap PÅ (gul)
Tangent:        F1 code=F1 keyCode=112
```

- `F1–F12 0` after pressing F1 means the TV consumed the key before the page.
  No Tizen app can recover a key it never receives.
- `omap` counts synthetic F keys sent by `Ctrl/Alt + 1..0,-,=`. If the game does
  not react, GFN may be ignoring untrusted (`isTrusted === false`) events.
- The reliable fallback for TV-consumed F keys is a keyboard with onboard
  remapping (QMK/VIA), a HID remapper dongle, or rebinding the action in-game.

The panel is Swedish; `Tangentbord` is the keyboard line, `Tangent` is the last
key, `relås` is re-lock.

## Esc: what to expect

- The browser always exits pointer lock on Esc; that is not preventable.
- The next keypress or click in the video calls `requestPointerLock()` again.
  `relås` in the panel increments when it succeeds.
- If a game menu does not want the pointer re-locked, press **green** to turn the
  lock assistance off (this also disables the drag fix).
- `window blur`, visibility changes and pointer-lock transitions are logged, so
  the panel shows whether the TV moved focus at the OS level.

## Upstream features

- **Starts first.** Loads `play.geforcenow.com/robots.txt`, applies the identity,
  then loads GFN's `index.html` in the same window.
- **Desktop identity.** Overrides `navigator.userAgent`, `userAgentData`
  (including `getHighEntropyValues`), `platform`, `vendor`, `maxTouchPoints` and
  `plugins`, spoofs GFN's blob workers, answers `(hover)`/`(pointer)` media
  queries like a desktop and hides Samsung's globals (`tizen`, `webapis`,
  `TizenTVApiInfo`, …). The real Chromium version is kept.
- **Pointer-lock keeper.** GFN re-requests pointer lock whenever the game shows
  or hides its cursor. Samsung's engine turns that into unlock + relock and
  releases the held mouse button. While the stream already holds the lock, the
  module answers "locked" itself.
- **4K override.** Hooks GFN's webpack runtime and calls
  `ConfigureRagnarokSettings` with `codeclist=<H265|AV1>,H264&force4kbrowser=1`.
  Then pick *Custom → 3840×2160* in GFN. Whether it is offered also depends on
  NVIDIA's server entitlement.
- **Diagnostics.** Live `RTCPeerConnection.getStats()`: resolution, fps, codec,
  RTT, jitter buffer, decode time, drops; SDK modes; mouse and gamepad state;
  spoofed identity; hidden Tizen traces; de-duplicated log.

## Configuration

Edit the `CONFIG` object at the top of `inject.js`:

| Key | Default | Purpose |
| --- | --- | --- |
| `spoof` | `true` | Present as Chrome on Windows |
| `hideTizenGlobals` | `true` | Hide `tizen`, `webapis`, `TizenTVApiInfo` and similar globals |
| `spoofWorkers` | `true` | Same identity inside blob workers |
| `spoofPointerMedia` | `true` | Desktop answers for `(hover)` and `(pointer)` media queries |
| `bootViaRobots` | `true` | Load GFN from `robots.txt` after the identity is in place |
| `mouse.keepLock` | `true` | Answer GFN's re-lock requests while the stream already holds pointer lock |
| `mouse.blockNativeDrag` | `true` | Suppress drag-and-drop and text selection while streaming |
| `keyboard.registerFKeys` | `true` | Ask Tizen to route F1–F12 to the app |
| `keyboard.captureFKeys` | `true` | `preventDefault()` on F1–F12 so TV hotkeys do not fire |
| `keyboard.remap` | `true` | `Ctrl/Alt + 1..0,-,=` → `F1..F12` |
| `keyboard.relockOnInput` | `true` | Re-lock pointer lock on the next key/click after Esc |
| `force4k.preferred` | `['H265', 'AV1']` | Codec order for the 4K override |
| `overlay.autoShowMs` | `20000` | Show the panel this long after page load (`0` = off) |

## License

MIT. The base module is Copyright (c) 2026 Julianbjrk; fork modifications are
Copyright (c) 2026 tngranados. See [LICENSE](LICENSE).
