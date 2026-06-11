# Golf Swing Tracker

Live skeleton tracking, swing-state recognition, measurements, and AI coaching —
running in the browser. Live at https://mikesqairpants.github.io/swing-tracker/

## What it does (by milestone)

**Milestone 1 — capture engine.** The camera tracks your body as a skeleton
(Google MediaPipe, fully on-device). A state machine recognizes ADDRESS, then
records the swing into a rolling video buffer. When the swing finishes, a
look-back analysis finds the exact frames for TAKEAWAY, TOP, IMPACT, and
FINISH and shows them as labeled snapshots.

**Milestone 2 — measurement layer.** Ten down-the-line measurements computed
from the checkpoint skeletons (spine angle, knee flex, head rise at impact,
hip thrust, balance, and more), each shown against a target range.

**Milestone 3 — AI coaching.** "Get AI coaching" sends the checkpoint photos
and measurements to Claude with the 31-point rubric prompt (from
`05_SWING_ANALYSIS_PROMPT.md`). Requires your own Anthropic API key, entered
under the ⚙ settings button — the key is stored only in your browser and sent
only to Anthropic. (The production architecture moves this to a backend later.)

**Milestone 4 pieces.** The verdict can be read aloud (🔊, built-in browser
voice). Sessions are saved on-device and recent ones are fed back to the coach
so it tracks progress instead of re-diagnosing the same fault.

**Video upload.** The "Upload video" button runs a recorded swing clip through
the exact same pipeline — useful for analyzing good swings to calibrate the
gold standard, or your own swings filmed earlier. The clip should include a
second of stillness at address so the detector can arm itself.

## Run it locally

```
npm install
npm run dev
```

Open the printed Local https address (accept the self-signed certificate
warning) and allow camera access. For a phone on the same wifi, open the
Network address. NOTE: run from a local disk, not Google Drive — npm breaks
on Drive's virtual filesystem.

## Deploy an update

```
npm run build
```

Then upload everything inside `dist/` (flat) to the GitHub repo
(`mikesqairpants/swing-tracker`) via Add file → Upload files, and commit.
GitHub Pages redeploys automatically in ~1 minute. Bump `APP_VERSION` in
`src/App.jsx` first so the header shows which build is live.

## Tuning

- Swing detection thresholds: top of `src/swing/swingDetector.js`
  (every constant is commented — stillness, takeaway trigger, impact delay…).
- Measurement target ranges: `TARGETS` in `src/swing/measurements.js`
  (replace with gold-standard numbers when available).
- Skeleton colors/sizes: top of `src/tracking/skeleton.js`.
- Smoothing: `SMOOTHING_ALPHA` in `src/tracking/smoothing.js`.
- Coaching model/length: top of `src/coaching/coachingEngine.js`.
- Voice rate/pitch: top of `src/coaching/voice.js`.

## How the code is organized

```
src/
  App.jsx                    the screen: wires everything together
  styles.css                 appearance only
  camera/camera.js           camera permission + preview (Capacitor-swappable)
  tracking/poseTracker.js    MediaPipe Pose Landmarker wrapper
  tracking/smoothing.js      jitter smoothing
  tracking/skeleton.js       draws the stick figure
  swing/swingDetector.js     swing state machine + look-back analysis + ALL thresholds
  swing/frameBuffer.js       rolling video memory (last ~10s)
  swing/measurements.js      Milestone 2: skeletons -> numbers
  coaching/coachingEngine.js Milestone 3: Claude coaching (BYOK)
  coaching/voice.js          reads the verdict aloud
  coaching/history.js        on-device session memory
```
