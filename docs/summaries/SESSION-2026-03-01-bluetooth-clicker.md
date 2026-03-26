# Session Handoff - 2026-03-01

**Project:** volleyball-scorer
**Status:** Bluetooth clicker support implemented, hardware on order. Waiting for delivery to test/calibrate.

## Context
Single-file PWA (`index.html`) — Dutch-language volleyball score tracker with voice recognition (nl-NL) and tap-to-score. One person scores for both teams (they are also a player).

## Decisions Made

- **Voice as secondary, clicker as primary** — reviewer flagged voice-only in a gym as wrong architecture
- **Joystick remote ordered** — AliExpress item 1005008785411249, €3.03, 4-way rocker + large button. Expected keys: `ArrowLeft`, `ArrowRight`, `ArrowUp`, `ArrowDown`, `Enter`, `VolumeUp`. Mapping: left=team A, right=team B, bottom=undo.
- **Calibration approach** — no hard-coded key codes; setup screen has 4 "press to assign" slots so whatever the remote sends gets mapped at runtime
- **Long-press undo as fallback** — if no dedicated undo key is assigned, holding a team button for 500ms triggers undo (with progress bar animation)
- **4 key slots** — team A, team B, undo, read score (all optional/independent)

## Changes Made This Session

All in `index.html`:

1. **Wake lock fix** (line ~660): stores handle in `wakeLock` var, re-acquires on `release` event (catches OS screen-dim without page-hide). Added `if (wakeLock && !wakeLock.released) return` guard.
2. **`undoPoint()` empty case** (line ~445): replaced `speak('Niets om ongedaan te maken')` with `setLastAction()` + rapid vibrate — avoids blocking voice recognition for a non-critical message.
3. **`visibilitychange` guard**: added `&& !state.gameOver` so wake lock isn't re-acquired after game ends.
4. **Clicker calibration UI** (setup screen): 4 rows with "press to assign" buttons and key-code badges. Duplicate key detection with error feedback.
5. **Short/long press**: `keydown` starts timer, `keyup` checks duration. Long-press animation via CSS `::after` progress bar (`--hold-duration` CSS var synced to `HOLD_MS = 500`).
6. **Dedicated undo/score keys**: when `clickerKeys.undo` is set, long-press is disabled and undo fires on `keydown` immediately. `readScore()` likewise on `keydown`.

## What Remains

- [ ] **Test with actual hardware** — verify key codes the joystick remote sends on Android Chrome. `ArrowLeft`/`ArrowRight` are expected but not confirmed. VolumeUp may or may not reach the browser (system intercept risk).
- [ ] **Sleep/wake behaviour** — AB Shutter style remotes need a wake-up press after idle. May need UX note or auto-ping workaround.
- [ ] **Voice recognition** — current state is functional with continuous mode, all 5 alternatives iterated. Reviewer's remaining concerns (SpeechGrammarList was already absent) are addressed.

## Key Files
- `projects/volleyball-scorer/index.html` — entire app (single file, ~720 lines)

## Known Voice Issues (from reviewer, partially addressed)
- Confidence threshold: removed ✓
- Continuous mode: on ✓
- Iterate all 5 alternatives: implemented ✓
- SpeechGrammarList: not present ✓
- Race condition (onend + synthesis onend): mitigated with shared `restartHandle` ✓
- Wake lock re-acquisition: fixed ✓
