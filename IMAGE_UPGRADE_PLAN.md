# AjoKoe — photographic traffic-situation images

Status 23 Nov 2026: **phase 1 (5 samples) built and awaiting approval.** Nothing on `main` has changed.

## 1. How illustrations worked before

- All 105 traffic situations (`SIT`, ids s1–s105) are drawn live as SVG by `Scene({spec, reveal})` from a small scene spec (road type, weather, signs, road users). No image files exist.
- The same component is used in three places: the situation drill (`SituationRun`), the full mock exam (`Mock`) and the mock review (`MockReview`).
- After an answer, red reveal rings are drawn from `hi: [{x, d, r}]`, which are road-perspective coordinates, not image coordinates.
- Question text, options and explanations are plain HTML and were not touched.

## 2. What phase 1 changes

- New `SituationVisual({spec, reveal, lang})` wraps `Scene`. If `SIT_PHOTO[id]` exists it shows the photo; otherwise, or if the file fails to load, it falls back to the original SVG. The quiz interface, questions, answers and explanations are unchanged.
- Files: `img/situations/<id>-v1-1920.webp` and `<id>-v1-960.webp` (16:9), delivered with `srcset` + `sizes`, so desktop at 1× loads the 960 version and phones/tablets with high-density screens load the 1920 version. Fixed `width`/`height` plus `aspect-ratio` stop layout shift.
- The next situation's photo is preloaded while the current one is answered.
- Reveal rings for photos are stored per image in percent (`hi: [{x, y, rx, ry}]`), positioned by hand on each image.
- Each photo has bilingual alt text.
- Review aid: `index.html#sit=s49,s53,s41,s71,s77` starts the drill with exactly those items.
- `img/situations/manifest.json` records asset id, Canva media id, edits and QC date for every image.

## 3. Production workflow (repeatable)

1. Write a scene prompt from the question itself: driver's-eye view, Finland, right-hand traffic, exact road-user positions, season and light, Finnish markings (yellow centre lines, white edge lines), no text or logos.
2. Generate with Canva `generate-image` (16:9, about 2720×1536).
3. Place on a 1920×1080 page and export JPEG q92.
4. Transfer: this workspace cannot reach Canva's download host, so a temporary GitHub Actions workflow on the staging branch downloads the exports and commits them; the workflow is removed afterwards.
5. **Signs are always replaced with official artwork.** The AI got 4 of 5 sign pictograms wrong. Every visible sign is swapped for the matching SVG from Väylävirasto's CC0 library (github.com/finnishtransportagency/liikennemerkit), scaled to the original sign's border, exposure-matched, softened and given grain.
6. Retouch any artefacts (seams, duplicated markings).
7. QC against the checklist below, then encode WebP (q76 at 1920, q74 at 960).
8. Add the `SIT_PHOTO` entry (version, alt text, reveal rings), rebuild, and screenshot at 1440×900, 820×1180 and 390×844.

## 4. QC results for the five samples

| Id | Scene | Problems found | Fix | Result |
|---|---|---|---|---|
| s49 | Zebra crossing, pedestrian from the left | Crossing-sign figure not to spec | Official E1 composited | Pass |
| s53 | Right turn across a cycle lane, cyclist going straight | None (left-side parking is legal in built-up areas since 2020) | — | Pass |
| s41 | Roundabout entry, car circulating | Roundabout-sign arrows garbled (mixed directions) | Official D2 composited | Pass |
| s71 | Snowy road, slippery-road warning | Non-Finnish slippery-road pictogram; vertical seam in the sky | Official A13; seam retouched | Pass |
| s77 | Dusk, elk warning | Sign showed a deer (closer to A20.2); duplicated centre-line dash | Official A20.1; dash removed | Pass |

Checklist applied to each image: professional and sharp; realistic; Finnish environment; signs and markings correct; road-user positions realistic; shows the scenario in the question; works on mobile and desktop.

## 5. Flagged for decision (not changed)

- **s79** asks about an elk crossing, but its SVG scene draws a child. The photo version will show an elk unless you say otherwise.
- Questions and explanations in the situation drill are still English-only in Finnish mode (the earlier Stage B translation item).
- Licensing: the photos are AI-generated in Canva under your Canva account's terms. The sign artwork is CC0. If you'd rather use a licensed stock library, the same `SIT_PHOTO` slot takes any 16:9 image.

## 6. Plan for the remaining 100 situations (after approval)

- Batches of about 15, grouped by environment (city, rural, winter, motorway, night/fog, roundabouts and junctions) so lighting and style stay consistent within a group.
- Expect the AI to struggle with traffic-controller hand signals (s100), level-crossing barriers and lights (s28, s98), motorway merges and hard shoulders (s81, s83), and exact lane arrows. Those get extra regeneration rounds, or keep the SVG scene if no photo passes QC.
- Every batch gets the same QC table and screenshots before it's merged.
- Weight: about 25–175 KB per image. Only the current situation's image loads (plus the next one, preloaded), so a 50-item mock exam spreads roughly 3–8 MB across the whole session.
