# WarionAI Android app - what's here and what's next

## Getting an APK (I can't compile one from this chat - no Android SDK access)

**Easiest: GitHub Actions (no local setup)**
1. Push this folder to a new GitHub repo.
2. The included workflow (`.github/workflows/build-apk.yml`) builds a debug
   APK automatically and attaches it to the run - go to Actions tab -> the
   run -> Artifacts -> `WarionAI-debug-apk`.

**Or: Android Studio**
1. Install Android Studio, open this folder.
2. Let it sync Gradle (downloads the wrapper + SDK bits automatically).
3. Build > Build APK(s), or just Run on a connected phone.

## What's implemented

- **CameraX capture** in rolling ~4.5s segments (`camera/CameraController.kt`).
- **Auto-detect a delivery**: `camera/MotionDetector.kt` watches for a burst
  of frame motion (the bowling action) after a quiet baseline, then the
  segment is extended ~1.1s and kept; segments with no trigger are
  discarded automatically - this is what satisfies "no random [clips]
  shown."
- **Gallery storage, no cloud**: kept clips are saved via MediaStore to
  `Movies/WarionAI` on the phone (`camera/GallerySaver.kt`).
- **Take DRS flow**: `ui/screens/DrsScreen.kt` uploads the clip to the
  engine, shows a processing state, plays the annotated replay, shows the
  verdict, and has a Yes/No correction control that posts to the engine's
  `/feedback` endpoint - this is the "feed deliveries back in" loop you
  asked for, wired end to end.
- **Settings screen** to type/save/test the engine's LAN address.
- No login anywhere, no analytics, no ads.

## Known gaps - please read before assuming this matches FullTrack AI

1. **The auto-detect trigger is a simple motion heuristic, not a trained
   model.** It will definitely false-trigger on things that aren't a
   bowl (someone walking past camera, a hand movement) and can miss a
   very fast delivery in low light. A TFLite classifier trained on your
   own footage is a straightforward swap-in (same call site in
   `CameraController.kt`) once you have labeled clips from the engine's
   data pipeline.
2. **Not yet tested on a real device** - I don't have Android hardware or
   emulator network access in this environment. CameraX's Recorder +
   ImageAnalysis + Preview combination is a supported concurrent-use-case
   set, but please test on your actual phone before relying on it, and
   check Logcat if recording doesn't start (some devices are picky about
   video encoder availability).
3. **No stump/pitch calibration step yet** - the engine currently assumes
   camera geometry; a "tap the crease and stumps" step in the app would
   make speed/length accurate rather than estimated (see engine README).
4. **No offline/on-device DRS** - "Take DRS" requires the phone and
   laptop on the same Wi-Fi network the whole time, as you described.

## Suggested next session
- Test the camera + auto-detect on a real phone, tune `MotionDetector`'s
  threshold/cooldown against real bowling footage.
- Add the crease/stump calibration screen.
- Start collecting + labeling real deliveries through the engine's
  feedback loop and run the first `train.py` pass.
