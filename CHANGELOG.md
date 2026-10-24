# Changelog

## 0.3.3-gfn

- Add a configurable Escape proxy (`keyboard.escProxy`, default `CapsLock`).
  Caps Lock / ScrollLock / Pause / ContextMenu sends a synthetic Escape without
  a real Esc, so the browser does not release pointer lock.
- Key-up safety timer (250 ms) so a consumed key-up cannot leave Escape stuck.
- Panel shows the proxy key and how many synthetic Escapes were sent.

## 0.3.2-gfn

Input fixes on top of Julianbjrk/gfn-tizen-desktop v0.3.1.

- Try to register `F1`–`F12` / `KEY_F1`–`KEY_F12` with Tizen's input API.
- Cancel the TV hotkey default for F keys that reach the page, without stopping
  them from reaching the game.
- Add `Ctrl/Alt + 1..0,-,=` → `F1..F12` remap, toggled with the yellow button.
  Key-up is delivered even if Ctrl/Alt is released first.
- Re-request pointer lock on the next keypress or click in the video after Esc.
- Cancel TV Back (`keyCode 10009`) so the TV does not take focus, when delivered.
- Log window blur, visibility changes and pointer-lock transitions.
- Extend the blue diagnostics panel with F-key/remap/re-lock counters, the last
  key (`key`, `code`, `keyCode`, trusted) and the Tizen registration result.
- Green now toggles both the pointer-lock keeper and the Esc auto-relock.

Documentation: 1440p120 confirmed on an S95C.
