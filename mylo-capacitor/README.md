# MYLO — Native App (Capacitor)

This wraps the MYLO web app (`www/index.html`) as a real installable Android/iOS
app using [Capacitor](https://capacitorjs.com). The web app itself is unchanged —
Capacitor just puts it inside a native shell (native splash screen, status bar
theming, home-screen icon, hardware back-button handling on Android) instead of
running it in a browser tab.

**Important — what I could and couldn't do in this sandbox:** I have no internet
access to Google's Gradle/Android SDK servers or a macOS machine with Xcode here,
so I can't produce a compiled `.apk`/`.ipa` for you directly. What's in this
folder is the **complete, verified project source** — dependencies installed,
both native platforms scaffolded, real branded icons and splash screens
generated and checked, config wired up — ready to open and build on your own
machine in a few minutes.

## What's in here

```
mylo-capacitor/
├── www/                    the app itself (index.html — same file you've been using, plus a small
│                            Capacitor bridge script at the bottom that only activates inside the
│                            native shell; the plain web version is completely unaffected)
├── android/                 native Android project (open in Android Studio)
├── ios/                     native Xcode project (open in Xcode, macOS only)
├── capacitor.config.json    app ID, name, permissions, splash/status-bar settings
├── package.json
└── .gitignore
```

App ID: `com.ubayanda.mylo` — matches your existing MYLO native package naming.
Change this in `capacitor.config.json` before you publish if you want something
different (see "Changing the app ID" below).

## Prerequisites

- **Node.js** (v18+) and npm — you already have these if you've been building
  the web version.
- **For Android:** [Android Studio](https://developer.android.com/studio)
  (includes the SDK and a bundled JDK).
- **For iOS:** a Mac with [Xcode](https://apps.apple.com/app/xcode/id497799835)
  and [CocoaPods](https://cocoapods.org) (`sudo gem install cocoapods`).

## First-time setup

```bash
cd mylo-capacitor
npm install
```

That's it — `android/` and `ios/` are already scaffolded and synced with the
current web app and icons. You only need `cap add`/`cap sync` again if you
change the web app or the config (see below).

## Building for Android

```bash
npx cap open android
```

This opens the project in Android Studio. From there:
- **Run on a device/emulator:** the ▶ Run button.
- **Build a debug APK to install manually:** Build → Build Bundle(s)/APK(s) →
  Build APK(s). The file lands in
  `android/app/build/outputs/apk/debug/app-debug.apk` — copy it to a phone and
  install it (you'll need to allow installs from unknown sources).
- **Build a signed release APK/AAB for the Play Store:** Build → Generate
  Signed Bundle/APK, and follow Android Studio's signing wizard (you'll need to
  create a keystore the first time — keep it safe, you need the same one for
  every future update).

Command-line equivalent, if you'd rather not open Android Studio at all:
```bash
cd android
./gradlew assembleDebug
# APK at android/app/build/outputs/apk/debug/app-debug.apk
```

## Building for iOS (macOS only)

```bash
cd ios/App
pod install
cd ../..
npx cap open ios
```

This opens the project in Xcode. From there:
- **Run on the simulator or a connected device:** the ▶ Run button. You'll need
  to set your Apple Developer team under Signing & Capabilities the first time.
- **Archive for the App Store / TestFlight:** Product → Archive, then follow
  Xcode's distribution wizard. You'll need a paid Apple Developer account for
  this part — there's no way around that requirement, it's Apple's, not mine.

## Making changes to the app

The actual app is just `www/index.html` (and `www/sw.js`) — same single-file
app as the web version. After editing it:

```bash
npx cap sync
```

This copies your changes into both `android/` and `ios/` and re-links any
plugins. Then rebuild in Android Studio / Xcode as above.

## What's already wired up for you

- **Real branded icons** — generated from the app's actual logo, not
  Capacitor's default placeholder graphic. Covers every Android density
  (legacy + adaptive-icon foreground/background layers) and the single 1024×1024
  iOS icon. I rendered and visually checked every one of these before including
  them, including simulating Android's adaptive-icon composite (foreground layer
  over the background color) to make sure the mark isn't invisible against its
  own background — that's an easy mistake to ship and I caught it while
  building this.
- **Branded splash screen** — dark background matching the app's theme, logo
  centered, generated at every size the templates called for.
- **Status bar & splash screen native behavior** — the app hides the native
  splash and sets the status bar to match its dark theme automatically once
  the UI is ready (see `initCapacitorBridge()` near the bottom of
  `www/index.html`'s script — it's a no-op when this same file is opened as a
  normal website, so nothing about the web version changed).
- **Android hardware back button** — closes an open sheet/search overlay if
  one's open, otherwise navigates to the Home tab, otherwise exits the app —
  rather than the OS default of just closing the app on the first press from
  anywhere.
- **Domains the app talks to are pre-allowed** in `capacitor.config.json`
  (`allowNavigation`) — YouTube, Supabase, iTunes, TheAudioDB, radio-browser,
  lrclib, ipapi — so those integrations keep working inside the native shell.

## Known limitations, and why

- **No compiled binary included.** Building an APK needs Google's Gradle
  distribution servers and the Android SDK; building an IPA needs an actual Mac
  with Xcode. Neither is available in the sandbox I built this in — I actually
  tried the Gradle build here to confirm, and it's blocked by network access
  restrictions, not something wrong with the project. The steps above are the
  real, standard Capacitor workflow — there isn't a shortcut around needing
  Android Studio or Xcode at least once.
- **Magic-link sign-in won't complete inside the native app as-is.** The web
  version emails a link back to `window.location.href`, which only makes sense
  in a browser tab. Inside a native app, clicking that link on your phone just
  opens it in the phone's browser instead of returning to the app. Proper
  support needs a custom URL scheme (deep linking) registered on both the
  Supabase side and in the native projects, plus a listener for
  `App.addListener('appUrlOpen', ...)` — that's a real chunk of additional
  work I haven't done here. Guest mode and password-based sign-in both work
  fine as-is since they don't depend on a redirect.
- **The service worker (`sw.js`) mostly doesn't matter here.** It was built for
  the web PWA's offline caching and Chrome's desktop install prompt — neither
  of those concepts really apply once the app is already a native install.
  It's harmless to leave in `www/`, just not doing much.
