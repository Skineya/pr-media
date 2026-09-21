# SKI-237 — runtime verification

What was actually observed on a running app, and how.

## Setup

- Dedicated headless iPhone 17 / iOS 27.0 simulator (no Simulator GUI on the
  verification machine, so nothing can tap).
- `skineya-dev`, Debug, built from this branch merged **locally only** with
  `alexandrjacko/debug-simulator-autologin` (PR #12). That merge exists purely
  to get past the auth wall; it is not part of this branch.
- Signed in for real via `-autologin-email` / `-autologin-password` launch
  arguments against `develop.skineya.app`.
- All emails, bearer tokens and APNs hex tokens are redacted from the logs in
  `logs/`.

## Screenshots

| File | What it shows |
| --- | --- |
| `01-launch-auth-gate.png` | The auth gate that blocked every earlier attempt (kept from the original commit). |
| `02-launch-permission-prompt.png` | First launch after auto-login: the system notification prompt over the signed-in Home screen. |
| `03-home-after-autologin.png` | Home, signed in, permission already determined. |
| `04-profile-push-toggle-authorized.png` | Profile › Preferences › **Push Notifications**, authorization granted → toggle on. |
| `05-profile-push-toggle-denied.png` | Same row with authorization denied → `fetchData()` renders the toggle off. |
| `06-notifications-denied-alert.png` | **The new denied-permission alert.** Flipping the toggle on while iOS has recorded a denial reverts the switch and offers *Open Settings*. |
| `07-push-banner-foreground.png` | A `simctl push` alert presented while the app is frontmost (proves the `UNUserNotificationCenter` delegate is live). |
| `08-push-banner-app-not-running.png` | The same push delivered with the app terminated: banner plus badge. |

## Two things had to be faked, and neither of them is app behaviour

Both are disclosed here because they are the only reason these screenshots
exist at all.

1. **Answering the system permission prompt.** `simctl privacy` has no
   `notifications` service and there is no GUI to tap *Allow* or *Don't Allow*.
   The authorization state was written straight into the simulator's
   `Library/BulletinBoard/VersionedSectionInfo.plist` — the same
   `BBSectionInfo` archive SpringBoard itself writes, with
   `authorizationStatus` set to `2` (authorized) or `1` (denied) — with the
   device shut down, then booted. The app still reads its status through the
   real `UNUserNotificationCenter.notificationSettings()`.

2. **Reaching Profile and flipping the switch.** A temporary, DEBUG-only
   harness mounted `ProfileView` as the window root, scrolled to the
   preferences card, then set `viewModel.notificationsEnabled = true` and
   called `viewModel.saveProfile()` — exactly what tapping the row and then
   *Save* does. Everything downstream of that (`applyNotificationsToggle()` →
   `refreshAuthorizationStatus()` → `.denied` → revert + alert) is the real,
   unmodified code from this branch. The harness was reverted before this
   commit and is not in the diff.

## Not verified

- **Cold-launch notification tap.** Nothing can tap a notification here, so
  "the delegate is installed before `didFinishLaunching` returns, therefore a
  cold-launch tap routes" is still argued from code.
- **Silent push.** `simctl push` rejects `content-available`-only payloads, and
  a combined alert + `content-available` payload is injected into the
  notification centre rather than delivered through the APNs background path,
  so `didReceiveRemoteNotification` is never entered. See
  `logs/b4-push-delivery.txt`.

  The `UIBackgroundModes` gap this turned up **is** fixed on this branch —
  `logs/b9-remote-notification-background-mode.txt` has the before/after from
  the built products plus the vanished launch warning. That removes what would
  have blocked silent push on a real device; whether the handler then routes
  correctly is still only argued from the code.
- **`unregister()` on logout / account deletion.** Confirmed by reading the
  code that it makes no network call (the endpoint enum has only `.register`),
  so there is nothing that can throw; not exercised at runtime.
