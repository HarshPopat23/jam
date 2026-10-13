# QR scanner cancellation evidence

Source fix: https://github.com/HarshPopat23/jam/commit/be8a9800aff660650bdd203f71e10c824aebbcb2
Base: joinmarket-webui/jam devel @ 31df00194cffd8f47684ad4bb84fdda2a3434819.

These are genuine screenshots captured from a local browser harness running the real QrScannerDialog and html5-qrcode 2.3.8. Camera acquisition was held by a controlled getUserMedia delay. Chromium 134.0.6998.35 used its fake-camera device; no physical camera, wallet, or personal camera frames were used.

## Browser steps
1. Open the scanner and wait until getUserMedia is pending.
2. Close the dialog.
3. Release the delayed acquisition.
4. Inspect the actual MediaStreamTrack.readyState retained by the harness.

Unmodified base: dialog closed, track remains live. The removed reader can also trigger html5-qrcode's minimum qrbox-size error.
Fixed implementation: dialog closed, track ends and the temporary reader is removed.
Diagnostics are development-only instrumentation that reads actual browser media-track state, not a mocked stop-call counter.

## Images
- 01-before-pending-start.png: unmodified base while acquisition is pending.
- 02-before-closed-late-start.png: unmodified base after closing and releasing acquisition; actual track is live.
- 03-after-closed-cleanup.png: fixed working tree after the same sequence; actual track is ended.
- 04-before-regression-failures.png: saved, verbatim Vitest output displayed in a read-only local page; the two targeted regressions fail on base. This is test output, not a live camera observation.
- 05-after-regression-passes.png: the same kind of saved test-output display; all 24 focused cases pass after the fix.

## Validation
Node 26.7.0; npm 11.19.0. Focused suite: 24 passed. Full unit and Storybook suite: 1402 tests in 248 files passed.
Coverage: statements 91.46%, branches 84.22%, functions 89.01%, lines 92.05%. Local browser tests used the installed Chromium 134 executable through an uncommitted config override; the production Vitest configuration was not changed.

Evidence is on this separate branch so the upstream PR contains only the scanner and its tests. Static images document observed states; regression tests establish the cancellation sequence.