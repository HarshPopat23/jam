# QR scanner: closing during camera startup can leave a live stream

**Expected behavior**

Closing or unmounting the QR scanner should clean up camera acquisition that completes afterward and suppress further camera fallback attempts.

**Actual behavior**

In `QrScannerDialog`, cleanup only stops a scanner when `isScanning` is already true, then clears the reference. Closing while `start()` is pending leaves no late-success cleanup. A late rejection also attempts the front-camera fallback after the dialog has closed.

Reproduced with the actual component and html5-qrcode 2.3.8 in Chromium using its fake-camera device and a controlled delay in `getUserMedia`: after closing and releasing acquisition, the actual video track remains `live`.

Removing the reader during acquisition also matters: the library can try to render into a detached, zero-sized viewfinder and raise its minimum-qrbox-size error. A fix must preserve enough of the reader lifecycle to complete startup and release the stream. `start()` can additionally resolve before the video `playing` event sets `isScanning`.

Both Send and Sweep use this shared scanner.

**Steps to reproduce the problem**

1. In a local test browser, open the scanner and hold camera acquisition pending after the 300 ms initialization delay.
2. Close the dialog before acquisition completes.
3. Release the delayed acquisition.
4. Inspect the acquired MediaStream's video track: it remains `live` although the dialog is closed.

For a deterministic component reproduction, defer the scanner's `start()` promise, render `open=false`, then resolve it: the original code makes zero stop calls. Rejecting the pending start instead produces a second start call for the fallback camera.

**Specifications**

- Version: 2.0.0-beta.4; upstream `devel` at `31df00194cffd8f47684ad4bb84fdda2a3434819`.
- Platform: Linux; Node 26.7.0; npm 11.19.0.
- Browser: Chromium 134.0.6998.35, fake-camera device. Physical camera and live-wallet flows were not tested.

**Additional context**

- [Affected component](https://github.com/joinmarket-webui/jam/blob/31df00194cffd8f47684ad4bb84fdda2a3434819/src/components/ui/QrScannerDialog.tsx).
- Related #1530 adds stories, and #1099 handles uploaded-image scanning; neither addresses cancelled startup.
- This is a camera privacy/resource cleanup issue; no fund-loss or video-transmission claim is made.
- [Evidence and reproduction notes](https://github.com/HarshPopat23/jam/tree/8b42abf769c0ac302f03aab8a02bd43f23f9dde7/qr-scanner-cancellation).

Before: dialog closed; actual fake-camera track remains live.

![Before: closed dialog, live camera track](https://raw.githubusercontent.com/HarshPopat23/jam/8b42abf769c0ac302f03aab8a02bd43f23f9dde7/qr-scanner-cancellation/02-before-closed-late-start.png)

A prepared fix ends the same track after the same cancellation sequence:

![After: closed dialog, ended camera track](https://raw.githubusercontent.com/HarshPopat23/jam/8b42abf769c0ac302f03aab8a02bd43f23f9dde7/qr-scanner-cancellation/03-after-closed-cleanup.png)