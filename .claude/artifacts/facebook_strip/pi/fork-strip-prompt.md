You are the Developer for the Flutter plugin repo `appinio_social_share` (current working directory = the fork repo `flutter_packages`). Task: remove the Facebook SDK dependency from the plugin entirely, while keeping every other share target untouched and keeping the Dart API 100% unchanged.

CONTEXT
- The app that consumes this fork is removing the Facebook SDK completely. This fork exists for iOS fixes the upstream package lacks (WhatsApp image share, X share, iPad crash fix) — those must keep working EXACTLY as they are.
- The plugin's only Facebook SDK uses are: iOS `shareToFacebookPost` (FBSDK ShareDialog) and Android `shareToFacebook` (FacebookSdk + ShareDialog). The iOS stories share (`shareToFacebookStory`) is pasteboard + URL-scheme based and SDK-free — keep its behavior unchanged.

STEPS
1. `git checkout -b no-facebook-sdk` (from current `main` HEAD). This is the ONLY git state change you may make. Do NOT push.
2. `ios/appinio_social_share.podspec`: remove the `s.dependency 'FBSDKCoreKit', '18.0.2'` and `s.dependency 'FBSDKShareKit', '18.0.2'` lines. Nothing else changes in the podspec.
3. `ios/Classes/ShareUtil.swift`:
   - Rewrite `shareToFacebookPost(args:result:)` WITHOUT FBSDK: load the UIImages from `argImagePaths`, build the share items (images + the message text), and present a `UIActivityViewController` on the root view controller — the same system share menu the plugin's system share uses. Mirror the iPad popover handling that the existing system-share path uses (the repo has a commit "Fix: iPad crashes when shareToSystem" — follow that pattern). Fire-and-forget semantics like today: return `SUCCESS` once the controller is presented. Facebook may not be installed — the system share menu handles that natively, no canOpenURL gate here.
   - Remove the `import FBSDKCoreKit` / `import FBSDKShareKit`, and remove the `SharingDelegate` conformance and its callback methods IF (and only if) they are now unused. Keep `shareToFacebookStory` byte-for-byte identical behavior (it must not need any SDK class).
   - Keep `getInstalledApps` and its scheme mapping unchanged (it is URL-scheme based, SDK-free).
4. `ios/Classes/SwiftAppinioSocialSharePlugin.swift`: remove FBSDK imports; keep the `FACEBOOK` and `FACEBOOK_STORIES` cases and the method-channel method names; adapt the `shareToFacebookPost` call signature if it loses the delegate parameter.
5. `android/build.gradle`: remove `implementation 'com.facebook.android:facebook-share:17.0.0'`.
6. `android/src/main/java/com/appinio/socialshare/appinio_social_share/utils/SocialShareUtil.java`:
   - Rewrite `shareToFacebook(filePaths, text, activity, result)` WITHOUT the SDK: build the content Uri via the existing FileProvider pattern, `Intent(ACTION_SEND)`, type `image/*`, `EXTRA_STREAM`, `FLAG_GRANT_READ_URI_PERMISSION`, `setPackage("com.facebook.katana")`. If the intent does not resolve (Facebook not installed), return the SAME not-installed result convention the other Android share methods in this class use, so the app's existing not-installed fallback keeps working.
   - Inspect `shareToFacebookStory`: if it uses `com.facebook.*` SDK classes, rewrite it as the closest intent-based equivalent (same target package, FileProvider Uris); if it is already intent-based, leave it alone.
   - Remove `FacebookSdk.fullyInitialize()` / `setApplicationId`, the `CallbackManager` field and all `FacebookCallback` plumbing, and every `com.facebook.*` import.
7. `android/.../AppinioSocialSharePlugin.java`: remove any now-dead callbackManager wiring (e.g. onActivityResult forwarding that existed only for the FB dialog). Keep all other method routing unchanged.
8. Dart files (`lib/`): NO changes. Public API must stay identical (same method names, same signatures, same return strings).

VERIFY
- `grep -rn "FBSDK\|FacebookSdk\|com.facebook.share\|ShareDialog\|CallbackManager\|facebook-share" ios/ android/src android/build.gradle ios/appinio_social_share.podspec` → only acceptable remaining hits: `com.facebook.katana` package strings, `facebook-stories`/`fb-messenger` scheme strings, the `FACEBOOK`/`FACEBOOK_STORIES` case constants and Dart-facing method names, and `com.facebook.sharedSticker.*` pasteboard keys.
- `cd packages/appinio_social_share && dart analyze` → 0 errors.
- Real compile proof happens later in the app build; you cannot build iOS/Android here.

HARD RULES
- Touch ONLY: the podspec, the 2 iOS Swift class files, `android/build.gradle`, the 2 Android Java files. Do NOT touch `example/`, other packages in this repo, or any `lib/` Dart file.
- Git: only the one `git checkout -b no-facebook-sdk`. No add/commit/push (the mediator commits).
- No comments except a one-line WHY for a non-obvious constraint (e.g. why the share menu has no canOpenURL gate).

FINAL OUTPUT (print exactly):
STATUS: DONE | FAILED
FILES_MODIFIED:
CHECKS: <grep result summary>, <dart analyze result>
DEVIATIONS: <or "none">
NOTES: <anything the reviewer must know>
