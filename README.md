# Packaging BacTrack as an Android APK

This folder wraps `backtrackv7_4_1.html` in Capacitor so it runs from a
`https://` origin inside the app instead of `file://`, which should fix the
phone-browser connection problems. It also starts a foreground service so the
sync and photo-upload timers keep running while the screen is locked.

## One-time setup (no local Android Studio needed)

1. Create a new **empty public** GitHub repo, e.g. `bactrack-apk`.
2. Upload everything in this folder to it, keeping the folder structure
   (`.github/workflows/build-apk.yml` must stay at that exact path).
3. GitHub → your repo → **Actions** tab → you should see "Build BacTrack APK" →
   click **Run workflow**.
4. Wait a few minutes, open the finished run, download the `bactrack-debug-apk`
   artifact, unzip it — that's your `app-debug.apk`.
5. Copy it to your phone and tap it to install (allow "install unknown apps"
   for whichever app you copied it with).

Whenever you want to ship a new version of the HTML file: replace
`www/index.html` with the new one, push, run the workflow again, reinstall.

## Why a foreground service

Android suspends a locked app's background work by default. The foreground
service keeps BacTrack classified as active, with a permanent low-priority
notification ("Keeping your tasks and photos in sync"), so the existing
Firestore listeners and the photo-retry timer keep running and your photos
upload the moment you're back on Wi-Fi — no need to open the app.

If you'd rather not have the permanent notification, delete the
`ForegroundService` block in `www/index.html`'s boot function and the
`@capawesome-team/...` line in `package.json` — the app will then behave like
a normal Android app and only sync while it's open or briefly backgrounded.

## What I have not been able to test

I don't have a phone or Android Studio here, so none of this has run on real
hardware. The known risk is that Android may still throttle the WebView's
JavaScript timers even inside a foreground service on some phone brands
(Xiaomi, Samsung and others add their own battery managers on top of
Android's). If photos still don't upload while locked after this, the fix is
disabling battery optimization for BacTrack in the phone's Settings → Apps →
Battery.
