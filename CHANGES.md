# Valley Society v56 "The Firing" — reference build `fork-1-from-v56`

Credit kept: "Original engine by Jack Revenue", owned by and commissioned by Nate Mendez (header comment, splash credit, deck signature). Third-party licences (three.js MIT, js-sha256, qrcode-generator, jsQR) were left inside the bundles untouched.

Files: `original.html` (untouched copy, 10,120,513 bytes), `fixed.html` (6,047,041 bytes), `before.png`, `after.png`, `fixed_fordham.png`, `original_load.png`, `test_original_result.json`, `test_fixed_result.json`, `run_test.py` (headless test), `shot.py`, `build_fixed.py` (reproduces fixed.html from original.html).

## 0. Does the original run?
Yes. It was a full simulation, not a save viewer. Headless Chromium over file:// had 0 uncaught exceptions. Tick advanced from about 93 to 2,100 over the test, and 1x, 2x and 4x ran at about 66, 126 and 168 ticks per 4 s. Pause held tick still (867 -> 867 over 4 s). With no save in localStorage it did not stay in a "waiting" state: the engine started a fresh local world on its own (tick, cohesion, population and structures all moved).

## 1. What the page is
- A Book-style shell (inline HTML/CSS/JS, about 330 KB) plus a bundled three.js WebGL engine (about 1.7 MB of JS, `__valleyVer` "54", booted true), plus the "Fordham Field Office" second world.
- Where the 9.7 MB came from (base64 inflates by about 4/3):
  - engine runtime: 2 identical copies (a `preload` link and the `<script src>`), about 4.55 MB
  - Fordham runtime, a `prefetch` that nothing used, about 2.27 MB
  - "payload" page, 2 identical copies (646,286 bytes each) used only for a hash check, about 1.72 MB
  - Fordham page, 4 identical copies, about 1.25 MB
  - shell about 0.33 MB, plus two 240-byte PNGs. There are no embedded fonts or photos.
- Attribution: the file contains no Kenney or CC0 text or assets (case-insensitive search of the whole file, including the decoded blobs). Nothing was invented or added for it. The credits that exist are the Jack Revenue and Nate Mendez ones above, plus MIT notices.

## 2. What was broken
0. The "v56 shipping proof" panel always said "Promoted payload mismatch". The code compared against stale constants (631,157 bytes, MD5 90f899b2…, SHA-256 8ba18a41…). The embedded payload and the file's own `<meta name="valley-payload">` say 646,286 bytes, MD5 ac536101…, SHA-256 ea66fe8d….
1. The Fordham world buttons (World switcher, dock, Build flyout "Fordham field office") could not work. They navigated the top frame to a `data:` URL, which Chrome blocks ("Not allowed to navigate top frame to data URL"). The Fordham page also loaded a separate `fordham-runtime.js` that does not exist in a single file, and its HQ button pointed at `../`.
2. The same payload and Fordham bytes were embedded 2 to 4 times, and the engine was embedded twice.
3. The "no save" label said "PHONE WORLD … no saved key on this origin", which is wrong on a desktop browser and could be read as HQ data.

## 3. What changed in `fixed.html`
0. Proof constants corrected to the real embedded values (bytes, MD5, SHA-256). The panel now reads "Promoted payload verified" and the check is still real, since the page hashes the embedded copy at load.
1. Fordham fixed: one copy of the page and one of its runtime are stored as text. A click builds a Blob URL with the runtime inlined and opens it in a new tab (falls back to same-tab if a popup blocker interferes). In that tab the HQ button closes the tab and returns to HQ, since there is no `../` to go to.
2. Dead weight removed: engine `preload` link, unused `prefetch` links, 3 of the 4 Fordham copies (the page moved to the single stored copy), and 1 of the 2 payload copies (the receipt reads the shared copy lazily). 10,120,513 -> 6,047,041 bytes (-40%). Engine bytes were not edited.
3. No-save label changed to "NEW LOCAL GAME · tick N · no saved key in this browser", with text saying the engine started a fresh local game from its own defaults, that it is not HQ data, and that the page seeded nothing. No money, ledger or HQ figures were added. The fresh-start behaviour is the engine's own. After a reload the label switches to the saved-world wording and the engine restored its save.
4. Title, header comment and deck signature carry "reference build fork-1-from-v56", per the licence note about fork numbering. Credits unchanged.

## 4. Tests (headless Chromium + SwiftShader, file://, 1280x800, same script on both files)
| Check | original.html | fixed.html |
|---|---|---|
| Uncaught exceptions | 0 | 0 |
| Engine booted, `__valleyVer` | 54, true | 54, true |
| Tick advances | yes (about 93 -> 2,100) | yes (about 89 -> 2,033) |
| Start playing / 1x / 2x / 4x | ok; 66 / 126 / 168 ticks per 4 s | ok; 66 / 132 / 160 ticks per 4 s |
| Pause holds tick | yes | yes |
| Resume | yes | yes |
| 20 s run at 1x | 1,203 -> 1,773 | 1,180 -> 1,740 |
| Dock panels (resources, events, alerts, ledger, build, inspect, guide) | all open | all open |
| Proof panel | "mismatch" | "verified" |
| Fordham button | blocked (data: URL), no navigation | opens, booted true, tick advancing (24 -> 168 in 15 s), HQ button closes the tab |
| Save restore after reload | saved-world label shown, tick continued (hydration receipt not read) | saved-world label shown, hydration receipt restored=true, tick 267 = 267 |
| Console errors | 5 (4 localhost probes + the data: URL block) | 4 (the localhost probes only) |

## 5. What remains
0. The engine's built-in "Inference scan" probes localhost ports 11434, 1234, 8080 and 5000 at boot. It fails quietly ("none answer") and shows as 4 console errors. It is a designed feature, so it was left alone. Hosting the page publicly means each visitor's browser will probe its own localhost.
1. The proof panel's "HQ vault restored intact" line stays false even though the save does restore. It requires the population to match exactly, and population changes between save and reload (114 vs 119 in the test). It is the same in the original and not part of the engine.
2. The legacy "PHONE WORLD" wording remains in the saved-key and invalid-key branches and in a static fallback sentence.
3. Fordham runs as engine v52 and keeps its own localStorage inside the Blob origin, so its state is not shared with HQ or kept between tabs.
4. Headless software rendering (SwiftShader) was the only renderer tested. Real GPU, iOS Safari and touch input were not tested.
5. No sync to a live HQ server exists in the code. None was added.
