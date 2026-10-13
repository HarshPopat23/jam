# fix(qr): clean up cancelled camera startup

After creating the upstream issue, add `Fixes #<issue-number>` to the PR description.

## What does this PR do?

Closing the QR scanner while camera acquisition is pending can leave its stream live or start a fallback camera after cancellation. This change gives each scanner session its own startup/stop lifecycle, discards stale callbacks, and cleans up that same instance after delayed acquisition.

The scanner owns its reader DOM node and retains it off-screen during pending acquisition/rendering so html5-qrcode can finish with nonzero layout dimensions. Cleanup handles the interval between `start()` resolving and the video `playing` event, and shares one stop promise across competing close/decode paths.

Scope: `QrScannerDialog.tsx` and its existing test file. Public props, BIP21 parsing, UI layout, and dependencies are unchanged.

### Validation

- Two targeted regressions fail on upstream `31df001`: no stop after late startup, and fallback attempted after closing.
- All 24 focused scanner tests pass after the fix.
- Full unit + Storybook suite: **1402 tests in 248 files passed**.
- Coverage: statements 91.46%, branches 84.22%, functions 89.01%, lines 92.05%.
- Lint, formatting, build, and `git diff --check` pass.
- Chromium fake-camera check with the actual component and html5-qrcode: close during delayed acquisition leaves a `live` video track on base; the fix ends it and removes the temporary reader.

Local validation used Node 26.7.0 / npm 11.19.0 and an uncommitted executable override for installed Chromium 134.0.6998.35. Physical-camera and live-wallet flows were not tested. Production test configuration is unchanged.

## Visual Demo

### Image Demo

These screenshots use actual browser media-track diagnostics in a temporary harness; no physical camera or wallet data is involved.

| Before: closed dialog, track live | After: closed dialog, track ended |
| --- | --- |
| ![Before](https://raw.githubusercontent.com/HarshPopat23/jam/8b42abf769c0ac302f03aab8a02bd43f23f9dde7/qr-scanner-cancellation/02-before-closed-late-start.png) | ![After](https://raw.githubusercontent.com/HarshPopat23/jam/8b42abf769c0ac302f03aab8a02bd43f23f9dde7/qr-scanner-cancellation/03-after-closed-cleanup.png) |

Captured focused test output (verbatim saved output displayed in a read-only page):

![Regressions fail on base](https://raw.githubusercontent.com/HarshPopat23/jam/8b42abf769c0ac302f03aab8a02bd43f23f9dde7/qr-scanner-cancellation/04-before-regression-failures.png)

![Focused suite passes after fix](https://raw.githubusercontent.com/HarshPopat23/jam/8b42abf769c0ac302f03aab8a02bd43f23f9dde7/qr-scanner-cancellation/05-after-regression-passes.png)

[Original screenshots and evidence notes](https://github.com/HarshPopat23/jam/tree/8b42abf769c0ac302f03aab8a02bd43f23f9dde7/qr-scanner-cancellation).