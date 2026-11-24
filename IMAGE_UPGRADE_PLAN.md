# AjoKoe — photographic traffic-situation images

Status 24 Nov 2026: **all 105 situations have a photo** (77 unique images; 28 situations reuse an image that shows the same scene). Merged into `main` and live.

## 1. How illustrations worked before

- All 105 traffic situations (`SIT`, ids s1–s105) were drawn live as SVG by `Scene({spec, reveal})` from a small scene spec. No image files existed.
- The same component is used in the situation drill (`SituationRun`), the mock exam (`Mock`) and the mock review (`MockReview`).
- Question text, options and explanations are plain HTML and were not touched.

## 2. What changed

- `SituationVisual({spec, reveal, lang})` wraps `Scene`. If `SIT_PHOTO[id]` exists it shows the photo; otherwise, or if the file fails to load, it falls back to the original SVG. Quiz interface, questions, answers and explanations are unchanged.
- Files: `img/situations/<id>-v1-1920.webp` and `<id>-v1-960.webp` (16:9), served with `srcset` + `sizes`. Fixed `width`/`height` plus `aspect-ratio` stop layout shift. The next situation's photo is preloaded.
- Reuse: a situation that shows the same scene points at another image (`{ f: "s10" }`), so it downloads nothing extra.
- Reveal rings are stored per image in percent (`hi: [{x, y, rx, ry}]`) and were placed by hand on each photo.
- Every photo has bilingual alt text (Finnish + English).
- `img/situations/manifest.json` records each image's asset id, Canva media id, scene, edits, file sizes and QC date.
- Review aid: `index.html#sit=s49,s53` starts the drill with exactly those items.
- Weight: 1920 px files are 63–204 KB (average 127 KB); 960 px files are 25–85 KB (average 52 KB). All 154 files total 14.2 MB, but only the current image and the next one ever load.

## 3. Production workflow (repeatable)

1. Write the scene prompt from the question itself: driver's-eye view, Finland, right-hand traffic, exact road-user positions, season and light, Finnish markings (yellow centre lines, white edge lines), no text or logos.
2. Generate with Canva `generate-image` (16:9), place on a 1920×1080 page (one image per design), export JPEG q92.
3. Transfer: this workspace cannot reach Canva's download host, so a temporary GitHub Actions workflow downloaded the exports. **It has now been removed from the branch.**
4. **Signs are always replaced with official artwork.** Every visible sign is swapped for the matching SVG from Väylävirasto's CC0 library (github.com/finnishtransportagency/liikennemerkit), scaled to the original sign, exposure-matched, softened and given grain.
5. Retouch markings and artefacts (direction of give-way triangles, line colours, seams, misspelt lettering).
6. QC against the checklist, then encode WebP (q76 at 1920, q74 at 960).
7. Add the `SIT_PHOTO` entry, rebuild, smoke-test, and screenshot the answered state to check the rings.

Checklist applied to every image: professional and sharp; realistic; Finnish environment; signs and markings correct; road-user positions legal and realistic for right-hand traffic; shows the scenario in the question; works on mobile and desktop.

## 4. QC summary

- 77 images, all passed.
- 16 needed regenerating at least once. Rejected outputs included a car driving in the wrong lane, a wrong-way car on a motorway's opposite carriageway, a double-parked car, a tram facing the driver, a deer instead of an elk, and signs at a junction that has to be unsigned.
- 30 had signs replaced with official artwork. The AI's own sign pictograms were rarely right.
- 12 needed marking fixes: give-way triangles pointing the wrong way, white centre lines on two-way roads, wrong-way turn arrows, a half-width zebra crossing, and stray dashes.
- Other fixes: "POLISI" and "AMBULANS" relettered as POLIISI and AMBULANSSI; traffic-light and indicator lamp states set to match the question; a pedestrian moved off the carriageway onto the pavement; faint vertical sky artefacts removed.
- 23 images needed no retouching at all.

| Image | Also used for | Scene | Problems fixed |
|---|---|---|---|
| s1 | — | Town crossroads, straight-ahead arrow in our lane, oncoming red lorry | regenerated twice: crossroads, lane arrow and red lorry (the explanation mentions both) |
| s2 | — | Zebra crossing, pedestrian approaching from the right pavement | E1 crossing sign replaced with official artwork |
| s3 | s57 | Slow car ahead, single solid centre line | regenerated once: first version changed to a dashed line ahead; two stray white lane marks removed |
| s4 | s39 | Unmarked junction, red car approaching from the right side street | first export came back as an unrelated image; re-exported |
| s6 | s70, s76 | Snowy rural road, car ahead in our lane | none needed |
| s7 | s97 | Tram stopped without a boarding island, passengers crossing our lane to the pavement | regenerated once: first version showed the tram facing us |
| s8 | — | Ambulance with blue lights behind (rear-view mirror), green light ahead | bonnet lettering corrected to AMBULANSSI (AI wrote AMBULANS) |
| s10 | s63 | Residential road past a school and playground, 30 km/h sign on the right | C32 30 km/h sign replaced with official artwork |
| s11 | s103 | Residential street, pedestrian stepping out between parked cars | none needed |
| s13 | s72 | Rural road in dense fog, dipped headlights | regenerated once: first version had the car straddling the centre line |
| s14 | — | Same crossroads, yellow only (flashing yellow mode) | lamp state edited from the s15 base (yellow only, pedestrian signal dark); flashing is stated in the question text |
| s15 | — | Signalled crossroads, red light, stop line | none needed |
| s16 | s75 | Bend ahead around a rock face, solid centre line | none needed |
| s17 | — | T-junction with give-way sign and give-way line, car on the through road | give-way triangles flipped to point at the driver (AI drew them pointing away); AI give-way sign checked and kept (correct B5) |
| s18 | s62, s82 | Motorway right lane, car merging from the on-ramp on the right | none needed |
| s19 | — | Ball rolling into a city street from a playground on the left | none needed |
| s20 | s59 | Car ahead at the centre line signalling left to turn, room on the right | regenerated once; right indicator recoloured to a plain tail light (AI showed both indicators lit) |
| s21 | s88 | City shopping street in spring, no-stopping sign on the right kerb | C37 no-stopping sign replaced with official artwork |
| s22 | s74 | Night, full beam on, oncoming car | regenerated once: car position |
| s23 | s51 | Person with a white cane waiting at a zebra crossing | zebra stripes extended across the opposite lane (AI drew only half a crossing); E1 crossing sign replaced with official artwork |
| s24 | s65 | Paved residential living street with planters and bicycles, living-street sign on the right | AI portrait-format sign replaced with official E24 living-street sign at its true proportions |
| s25 | s73, s105 | Heavy rain, standing water across our lane | regenerated once: car position and water placement |
| s26 | s33 | Priority road sign, white car waiting on the side road on the right | first export came back as an unrelated image; re-exported; AI priority-road sign checked and kept (correct B1) |
| s27 | s95 | Town street with 50 km/h sign, bus signalling left to leave a stop | C32 50 km/h sign replaced with official artwork; right-hand amber lamps turned off so only the left indicator shows |
| s28 | s99 | Rural road approaching an unguarded level crossing | A26 warning sign and A29.1 crossbuck replaced with official artwork |
| s29 | s54 | Cyclist ahead, solid centre line, oncoming car | none needed |
| s30 | — | Two lanes: left lane left-turn arrow, our lane straight-ahead arrow | left-lane arrow repainted as a left-turn arrow in lane perspective (AI drew it pointing right) |
| s31 | — | Unmarked crossroads, white car approaching from the left | none needed |
| s32 | — | STOP sign and stop line at a crossroads, car approaching from the right behind a hedge | white centre line recoloured yellow (Finnish two-way road); AI STOP sign checked and kept (correct B6) |
| s34 | — | Waiting to turn left (left-turn arrow), oncoming car going straight | lane arrow mirrored to a left-turn arrow (AI drew a right-turn arrow); wrong-way arrow beyond the junction removed |
| s35 | — | End of a living street meeting a main road, pedestrian on the pavement across the exit, end-of-living-street sign | pedestrian moved from the main-road carriageway onto the pavement across the exit (AI had them walking in the road); AI sign replaced with official E25 end-of-living-street sign; first generation rejected: car on the main road drove in the wrong lane |
| s36 | — | Unsigned T-junction, car approaching from the right on the crossing street | stray road markings at the junction removed |
| s37 | — | Van parked on our side of a narrow road, oncoming car | none needed |
| s38 | — | Unsigned crossroads, tram on the crossing street from the left | regenerated once: first version had a give-way sign and marking that would change the rule tested |
| s40 | — | End-of-priority-road sign, car approaching the next junction from the right | B2 end-of-priority sign replaced with official artwork (AI drew a striped variant) |
| s41 | s12 | Roundabout entry, give-way line, car circulating left to right | D2 roundabout sign replaced with official artwork (AI arrows were garbled) |
| s42 | — | Empty single-lane roundabout entry in a suburb, give-way line and B5 + D2 signs | give-way line repainted: AI drew alternating triangles; now every triangle points at the approaching driver; B5 give-way and D2 roundabout signs replaced with official artwork |
| s43 | — | Circulating in a roundabout, a pedestrian crossing the zebra on our exit road | stray direction arrow removed from the exit road |
| s44 | — | Roundabout entry, cyclist circulating | D2 roundabout sign replaced with official artwork; AI give-way sign checked and kept (correct B5) |
| s45 | — | Same crossroads, red and yellow lit together | lamp state edited from the s15 base (red + yellow) |
| s46 | — | Green light, left-turn arrow, oncoming car going straight | none needed |
| s47 | — | Turning right on green, pedestrian on the crossing of the side street | none needed |
| s48 | s101 | Same crossroads, green light | lamp state edited from the s15 base (green) |
| s49 | — | City street, zebra crossing, pedestrian stepping on from the left | E1 crossing sign replaced with official Väylävirasto artwork |
| s50 | — | Two lanes, SUV stopped at the crossing in the next lane, pedestrian crossing in front of it | generated after Canva cooldown |
| s52 | — | Child alone at the left kerb | none needed |
| s53 | s5 | City street, right-hand cycle lane, cyclist going straight past a side street on the right | none needed |
| s55 | — | Pedestrian stepping off a refuge island into our half of the crossing | E1 crossing sign replaced with official artwork |
| s56 | — | Person walking a bicycle across a zebra crossing | E1 crossing sign replaced with official artwork |
| s58 | — | Dashed centre line, clear straight road, car ahead | none needed |
| s60 | — | Car ahead entering a blind left bend, solid centre line | first generation failed on the server; retried |
| s61 | — | Car ahead signalling left to overtake a lorry | regenerated once: first version showed no clear left signal |
| s64 | — | Rural main road, 80 km/h speed-limit sign | C32 80 km/h sign replaced with official artwork |
| s66 | — | Entrance to a narrow cobbled one-way street in an old town centre, no-entry signs on both kerbs | both AI signs replaced with official C17 no-entry signs; faint vertical line artefact removed from the sky |
| s67 | — | Two-lane road at the edge of town, car ahead in our lane, no-overtaking sign on the right | AI sign replaced with official C28 no-overtaking sign |
| s68 | s80 | Country road, 'other danger' warning sign | A33 sign composited (AI had drawn a level-crossing sign) |
| s69 | — | Traffic-signals warning sign, signals ahead | A23 sign replaced with official artwork |
| s71 | — | Snowy rural road, slippery-road warning sign | A13 slippery-road sign replaced with official artwork (AI pictogram was non-Finnish); vertical seam in sky retouched |
| s77 | s9 | Forest road at dusk, elk warning sign | A20.1 elk sign replaced with official artwork (AI drew a deer); duplicated centre-line dash removed |
| s78 | — | Lapland main road, reindeer warning sign, reindeer on the right verge | A20.2 reindeer sign replaced with official artwork |
| s79 | — | Elk (moose) crossing the road in our lane | regenerated once: first animal looked like a deer, not an elk |
| s81 | — | Joining the motorway from the acceleration lane, traffic passing on the left | regenerated once: first version was not on the slip road |
| s83 | — | Motorway queue in both lanes, empty hard shoulder on the right | regenerated once: first version had a car on the left shoulder and a narrow hard shoulder |
| s84 | — | Two-lane Finnish motorway in summer, yellow left edge line, white dashed lane line, hard shoulder on the right | dark corner artefact removed from the sky; first generation rejected: a car on the opposite carriageway was shown from behind (driving the wrong way) |
| s85 | — | Motorway at speed, car ahead in our lane | none needed |
| s86 | — | Start of the motorway, motorway sign, no speed-limit sign | E15 motorway sign replaced with official artwork |
| s87 | — | Residential street in autumn, no-parking sign on the right | C38 no-parking sign replaced with official artwork; faint vertical line artefact removed from the sky |
| s89 | — | Autumn town street approaching a zebra crossing, empty kerb before the crossing, woman with pushchair waiting | AI sign replaced with official E1 pedestrian-crossing sign; yellow dashed line along the parked cars recoloured white (yellow would read as a second centre line); car badge softened |
| s90 | — | Town-centre street with cars parked in single file at the kerb, P sign with parking-disc panel | P sign and panel replaced with official E2 + H19.2 parking-disc panel (30 min); centre line recoloured from white to yellow; two people walking in the carriageway far ahead removed; first generation rejected: a car was double-parked and the bay markings were garbled |
| s91 | — | Empty bus stop at the kerb | bus-stop sign replaced with official E6 (AI drew a round blue sign, which is not a Finnish bus-stop sign) |
| s92 | — | Cars parked along the right kerb before a junction | none needed |
| s93 | s94 | Stopped at a red light, police car with blue lights behind (rear-view mirror) | bonnet lettering corrected to POLIISI (AI wrote POLISI) |
| s96 | — | Rural 80 km/h road, coach signalling to leave a lay-by stop | regenerated once: first version was a town street; C32 80 km/h sign replaced with official artwork |
| s98 | — | Guarded level crossing, barrier down, red lights on | A29.1 crossbucks (2) replaced with official artwork |
| s100 | — | Road works, traffic controller signalling stop, temporary light green | none needed |
| s102 | — | Residential street, ball rolling out between parked cars | export page taken from a Canva layout variant of the same image |
| s104 | — | Oncoming car drifting over the centre line towards us | none needed |

## 5. Flagged for your decision (not changed)

- **s66 wording (fixed 24 Nov with your approval).** The explanation said the no-entry sign has "a white bar"; the official Finnish C17 *Kielletty ajosuunta* sign has a **yellow** bar. It now reads "a horizontal bar". The general theory question "A red circle with a single white horizontal bar means:" (Road signs) has the same wording and was left as it is.
- **Finnish mode.** Situation questions and explanations are still English-only in Finnish mode (the earlier Stage B translation item).
- **s79.** The old SVG scene drew a child, but the question is about an elk. The photo shows an elk, which matches the question.
- **Canva clutter.** Each image left a generated asset and a one-page design in your Canva account (around a hundred of each, counting rejected attempts). They can be deleted; the website doesn't use them.
- **Licensing.** The photos were AI-generated in Canva under your Canva account's terms. The sign artwork is CC0. The same `SIT_PHOTO` slot takes any 16:9 image if you ever switch to licensed stock.

## 6. Going live

`image-upgrade-samples` was fast-forwarded into `main` on 24 Nov 2026. To roll back, revert the merge range on `main`; the SVG scenes return automatically when `SIT_PHOTO` is empty.
