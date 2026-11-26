# Hosting plan: Valley Society reference build (fork-1-from-v56)

Status: prepared, NOT published. Publishing waits on Nate's explicit yes.

## Hosted copy
- `hosted.html` (6,046,301 bytes) = `fixed.html` with one change inside the engine bundle:
  `INFERENCE_PORTS` set to `[]`, so the boot "Inference scan" probes no localhost ports (11434, 1234, 8080, 5000).
  The browser-local Chrome AI check (no network) is untouched. The hash-checked payload copy was left byte-identical, so the proof panel still reads "verified".
- `fixed.html` and `original.html` untouched (see SHA256SUMS).

## Test (headless Chromium, same run_test.py as CHANGES.md, result in test_hosted_result.json)
- 0 page errors, 0 console errors, 0 failed requests (fixed.html had 4, all localhost probes).
- Engine v54 booted; tick 90 -> 2,055; 1x/2x/4x 66/120/160 ticks per 4 s; pause holds; all 7 dock panels open.
- Proof panel: verified, 646,286 bytes. Fordham opens in a Blob tab (v52, booted, no errors), HQ button closes it.

## Proposed hosting (free, static)
1. New public GitHub repo (e.g. `valley-society-ref`) on an account Nate picks.
2. Commit `hosted.html` as `index.html`, plus CHANGES.md and this plan. Only the hosted copy goes public.
3. Enable GitHub Pages from `main` / root. URL shape: `https://<account>.github.io/valley-society-ref/`.
4. 6 MB is well under Pages limits (1 GB site, 100 MB file).
5. Link shared only with the agents doing red/blue-team work. Not posted on GLP forum or anywhere else.

Alternative if no GitHub: Netlify Drop or Cloudflare Pages (both free, also public).

## Notes for visitors
- Saves live in each visitor's own browser localStorage; nothing syncs to HQ.
- Anyone with the link can view it; Pages has no password option.
