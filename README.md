# Roohu ClipForge 2.0 (Android)

Turns long videos into vertical 9:16 clips, **autonomously**. Package `com.roohu.clipforge`.

## What changed in 2.0
- **Auto Create button**: one tap finds the best moments, writes the spoken words as captions, titles each clip, styles it by mood, adds music and saves everything to the Gallery.
- **Offline auto-captions** (Vosk) with real per-word timing. The spoken word pops and lights up. Styles: Pop, Karaoke, Bar, Outlined, or Auto (matches the clip mood).
- **Styled RoohuXKhan badge**: gradient pill, play icon, glowing X, shimmer sweep, slide-in. Position, size and opacity stay adjustable in Settings.
- **No more lag**: export now runs on the phone's hardware encoder (Media3 Transformer) at a constant 30 fps, H.264 + AAC MP4 with a proper index. Clips save to **Movies/RoohuClipForge**, so they show in the Gallery.
- **Built-in music**: if you add no tracks, a small original beat is generated per mood (calm, upbeat, intense). No copyright worries.
- Animations: zoom punch-in, beat pulse, fade in/out, vignette, animated progress bar.

## Build the APK on GitHub (no Android Studio)
1. Create an empty GitHub repository and upload **everything** in this folder, including the hidden `.github` folder.
2. Open **Actions**. The build starts on every push to `main` (or press **Run workflow**). The workflow downloads the two speech models (about 80 MB) by itself, so this takes a few minutes longer than before.
3. When it is green, download **Roohu-ClipForge-debug-apk** under Artifacts and install `app-debug.apk`.

If the build fails, open the failed step and send the first line that starts with `error:` or `What went wrong:`.

## Honest limits
- Offline captions support **English** and **Hindi/Urdu speech** (Urdu speech comes out in Devanagari script, because there is no small Urdu model). Other languages: import an .srt file.
- "Fit with blurred background" is exported as a clean letterbox. Fill mode (default) follows the motion.
- Music ducking under speech is not in the native export yet; music plays at the volume set in Settings.
- Claude title/rating needs claude.ai and is unavailable here; titles come from the first spoken words instead.
- YouTube links are not downloaded (needs a server).
- Debug-signed APK. A Play Store release needs your own signing key.

## 3.0 Milestone 2: reaction-edit look (new)
- Settings > **Edit look**: black frame with two glowing white dividers, a **3D hook title** in the top bar (red + yellow, extruded), and a **zoom punch + light flash + shake on every loud moment** found automatically from the clip's audio (`Energy.java`). Divider glow pulses with the hits.
- Applies to the phone export. The in-app preview stays plain.
- Next: M3 3D captions with bundled free fonts, M4 on-device scene scoring + mood music, M5 optional Gemini key.

## 3.1 Milestone 3: languages, real fonts, 3D look (new)
- **Any spoken language**: Settings > Spoken language is now *Auto-detect*. Paste a free Groq key (console.groq.com, no card) and Whisper detects the language itself (Urdu, Hindi, Arabic, English and 90+ more) with exact word timing. Without a key the app picks between its offline English and Hindi/Urdu models automatically. Right-to-left languages are laid out right-to-left.
- **Real fonts**: the GitHub build now downloads Anton, Bungee, Bangers, Luckiest Guy, Archivo Black, Poppins Black, Bebas Neue, Russo One, Black Ops One and Tajawal (Arabic) into `assets/fonts`. If a download fails the app falls back to a system font and the build still succeeds.
- **3D captions**: extruded letters with outline, gradient face, shine and a glow on the spoken word. Styles: 3D pop, Fire, Gold, Ice, Neon (auto by clip mood). Each caption page springs in.
- **Cinematic GPU effects** (`FxEffect.java`): chromatic aberration, glitch slices, bloom, grade, vignette and film grain, kicking on every loud moment.
- **Name credit**: the tiny logo pill is gone. "ROOHU X KHAN" is now big 3D chrome text (gold / red X / ice) with a moving shine, centred in the bottom bar.

## 3.2 Hardening
- `shared/core/validate.js` is strict now (null/empty/boolean timestamps rejected, bad metadata never throws, text cleaned, style and scores whitelisted, export-settings and file-name sanitising). 28 tests: `node shared/test/validate.test.js` (also runs in CI before the build).
- Cloud captions retry with exponential backoff (`Net.java`) and cache transcripts locally per clip and language (`cacheDir/asr`), so the same clip is never sent twice.
- The Groq key is stored encrypted with an Android Keystore AES-256-GCM key (`SecureStore.java`), not in localStorage.
- Optional release signing through CI secrets `RELEASE_KEYSTORE`, `RELEASE_KEYSTORE_PASSWORD`, `RELEASE_KEY_ALIAS`, `RELEASE_KEY_PASSWORD` (debug builds need none).

## 3.3 Step 1: real export settings
Shape (9:16, 16:9, 1:1, 4:5, 3:4), resolution (480p/720p/1080p), frame rate (24/30/60), bitrate with size estimates and a file-name template. Settings are validated by `validate.js` in the page and again in Java, and are applied to the actual render (`ClipExporter`). The reaction frame is used for portrait shapes only.

## 3.4 Steps 2 and 3
- **Projects**: autosave every 3 s and when the app is closed, a backup copy, versioned records (newer-version projects are kept, never deleted), Continue / Open / Rename / Duplicate / Delete (delete needs a second tap). Reopen a project by picking the same video again; a different video is refused and nothing is overwritten.
- **Caption editor** (new *Captions* tab): edit text and times, split at the playhead, merge, add, delete, shift all lines, import and export `.srt`. Word timings are kept in step so animated captions stay in sync, and edits flow into the exported clips.
- Tests: `node shared/test/validate.test.js`, `projects.test.js`, `subtitles.test.js` (49 in total, run in CI before the build).

## 3.5 Step 4: AI director (optional Gemini)
- Settings > **Free Gemini key**. Tap *Title and rate with AI* in Review, or run Auto Create with a key: each clip gets a title, a hook-strength rating (how strong the opening is, not a view prediction), a one-line reason, a mood (which picks the music) and a caption style.
- Official REST API with the key in the `x-goog-api-key` header, JSON schema output, answers re-validated locally (`validateDirector`), retries with backoff, answers cached on the phone. The newest stable *flash* model is chosen by asking Google which models the key can use, so no model name is hardcoded.
- Privacy: only the caption text of the clips is sent. No video or audio is uploaded. The key is stored encrypted (Android Keystore).
- The old "Claude" button only worked inside a preview window, never in the real APK; it is replaced.
- On-device scoring (audio loudness, motion, punch moments) works with no key and no internet.

## 3.6 Key Save button + Neon story look
- Settings > Groq key and Gemini key now have **Save key** and **Clear** buttons. A status line shows "Saved on this phone ✓ (ends …xxxx)" after a read-back check. Pasting also saves by itself.
- Settings > **Edit look** is now a list: Reaction, **Neon story** (new), Plain.
- Neon story (from the Qatar reference video): full screen, richer colours, the clip title typed letter by letter (yellow first line, glowing green second line), zoom + flash + glitch on loud moments, spoken captions moved lower.
- Not tested on a phone. Not compiled here (no Android SDK). If the GitHub build fails, send the first line with `error:`.

## 3.7 Face cam cutaways (new)
- Settings > **Face cam cutaways**: choose a second video (for example a face reaction). The exported clip opens on the face cam, then cuts between your main video and the face cam. Settings: how many seconds each one shows.
- Sound stays continuous from the main video (face cam sound is not used). Every cut gets a zoom, flash and glitch.
- Phone export only. The editor preview stays plain.
- Not compiled or tested on a phone. It uses `Composition.Builder.setEffects` (Media3 1.4.1). If the build says it cannot find that method, send me the line.

## 3.8 Highlight captions + zoom blur (new)
- Settings > Captions > Style > **Highlight**: all words white, the spoken word lit green, yellow or red with a glow. Sits at the top of the screen (in the Reaction look it stays in the lower area).
- **Zoom blur** now kicks on every loud moment and every face cam cut (`FxEffect.java`), together with the flash and glitch.
- Not compiled or tested on a phone. If the GitHub build fails, send me the first line that starts with `error:`.
