All checks pass. Final report:

```
STATUS: DONE
FILES_MODIFIED:
- packages/appinio_social_share/ios/appinio_social_share.podspec (removed FBSDKCoreKit 18.0.2 + FBSDKShareKit 18.0.2 deps)
- packages/appinio_social_share/ios/Classes/ShareUtil.swift
- packages/appinio_social_share/ios/Classes/SwiftAppinioSocialSharePlugin.swift
- packages/appinio_social_share/android/build.gradle (removed facebook-share:17.0.0)
- packages/appinio_social_share/android/src/main/java/com/appinio/socialshare/appinio_social_share/utils/SocialShareUtil.java
CHECKS: grep shows zero disallowed hits — only com.facebook.sharedSticker.* pasteboard keys, com.facebook.katana package string, facebook-stories/fb-messenger scheme strings, and FACEBOOK/FACEBOOK_STORIES case constants remain; dart analyze = 0 errors (2 pre-existing info-level lints in untouched example/lib/main.dart, identical on main)
DEVIATIONS: none in behavior; one mechanical signature change beyond shareToFacebookPost — shareImageToWhatsApp(args:result:delegate:) dropped its delegate param because SharingDelegate is an FBSDKShareKit type and the param was never used in the body (iOS WhatsApp image-share behavior is byte-for-byte identical)
NOTES:
1. Branch no-facebook-sdk created from main HEAD (b27b97c); nothing staged/committed/pushed; git status shows only the 5 files above modified.
2. iOS shareToFacebookPost now = system share menu (UIActivityViewController with loaded UIImages + message), mirroring the shareToSystem iPad-popover fix from commit fc4c4c0; fire-and-forget SUCCESS after present, single result call (the old code could double-result on validate error). One-line WHY comment added for the missing canOpenURL gate, per instructions.
3. Removed now-dead SwiftPlugin SharingDelegate conformance, its 3 callback methods, and the flutterResult field (only consumer of those callbacks).
4. Android shareToFacebook: ACTION_SEND + image/* + EXTRA_STREAM (FileProvider uri, first image) + EXTRA_TEXT + FLAG_GRANT_READ_URI_PERMISSION + setPackage("com.facebook.katana"); not-installed → ERROR_APP_NOT_AVAILABLE (the class's existing not-installed convention, also thrown via ActivityNotFoundException catch). Android FB post share now carries ONE image (ACTION_SEND holds a single stream; original SDK path supported multiple) — reviewer should confirm the app passes ≤1 image to shareToFacebook on Android.
5. Android shareToFaceBookStory was already intent-based (com.facebook.stories.ADD_TO_STORY + FileProvider) — untouched. AppinioSocialSharePlugin.java needed NO changes: it never contained callbackManager/onActivityResult wiring (verified by grep). getFacebookAppId kept — it only reads manifest meta-data and is still used by shareTextToPackage/shareToFaceBookStory.
6. pub get side effects: flutter pub get auto-modified analysis_options.yaml (package + example) and example/pubspec.lock during verification; I reverted all three to HEAD. Only gitignored artifacts (.dart_tool/, pubspec.lock) remain from the analyze run. ERROR_CANCELLED constant left declared (public, harmless) though now unused.
7. lib/ Dart files, example/, and all other packages untouched; Dart API byte-identical.
```
