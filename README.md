# Alexandria VR

A cozy, walkable 3D library you can enter on desktop or in a VR headset (WebXR) [Not currently optimized for mobile]. Walk to the bookcase, touch a shelf, see its covers float out in front of you, touch a cover, and read. Built with Three.js r128 and GSAP; no build step is needed to run it. Prototype is testable here: https://vincentkovar.github.io/alexandri-vr/

<img width="260" alt="Bookcase view" src="https://github.com/user-attachments/assets/c7e7de02-0260-4492-be82-2246a8e93486" />
<img width="260" alt="Books fanned from one shelf" src="https://github.com/user-attachments/assets/1521bcde-bf19-4c65-bb6c-77788e81002b" />
<img width="260" alt="Flippable preview pages" src="https://github.com/user-attachments/assets/a6ab471d-92e2-4d0e-9f6e-b5c3d885f529" />

## Goals

**Goal:** a first-time viewer, with no coaching, walks to a bookcase, touches a shelf, sees covers fan out into a floating display, touches one, and reads, in **under 60 seconds**, on desktop and on Quest 2.

**Current scope:** one cozy room, one interaction flow, 12 Project Gutenberg books with real covers, VR teleport.

**Out of scope (iteration 3):** Calibre export, 300+ books, multiple bookcases, search, full-length reading, multiple rooms or moods.

**Future iterations:** Branded multimedia stores to purchase digital content (books, podcasts, etc)

The full iteration 2 spec is in [docs/iteration-2-spec.md](docs/iteration-2-spec.md). Where the project is going (personal library and branded storefront) and the phased roadmap are in [docs/PRD.md](docs/PRD.md).

## Stretch Goals

**Look and feel:** go grand!


<img width="450" alt="library-baroque" src="https://github.com/user-attachments/assets/f7567cac-99a3-477c-ab89-5fb0b2fe7e11" />

**Make it personal**


<img width="450" alt="cyberpunk-library" src="https://github.com/user-attachments/assets/a50f02b1-8ba4-40b0-8748-6c184563f527" />


## Run it locally

The page is a single `index.html`. It loads Three.js and GSAP from CDNs, so it needs an internet connection.

**Serve it over HTTP.** The covers are loaded as WebGL textures from `covers/`, which browsers generally block or fail to resolve when the page is opened straight from disk (`file://`). In that case the generated fallback covers appear instead. All testing of the real covers was done over HTTP.

```bash
# Option A: any static server
python3 -m http.server 8000
# then open http://localhost:8000
```

```bash
# Option B: the project's Vite setup
npm install
npm run dev
# then open http://localhost:3000
```

`npm install` previously failed with a peer-dependency conflict (`esbuild ^0.25` vs Vite 8). The pin is now `^0.28.0`. With that change, install, the dev server, `npx tsc --noEmit` and the unit tests all worked in a clean copy of the repo.

Unit tests (data and pagination logic only, not the 3D scene):

```bash
npx tsx --test src/library/reader-logic.test.ts
```

## Controls

| | Desktop | VR (Quest) |
|---|---|---|
| Move | Orbit Cam (drag to orbit), or FPS Walk with WASD | Push a thumbstick forward to aim, release to teleport |
| Touch a shelf | Click it: its books float into an arc in front of you | Trigger |
| Open a book | Click a floating cover | Trigger on a cover |
| Put books back | Click the shelf again, or empty space | Same |

## Project layout

```
index.html              the whole app (scene, room, interaction, reader, VR)
covers/<gutenbergId>.jpg  local cover art (gutenberg.org sends no CORS headers)
images/                 style reference photos (not used in the scene)
docs/                   iteration specs
src/library/            typed, unit-tested book data + pagination logic (mirror of
                        the data in index.html; index.html does not import it)
```

## Iteration 2: requirement status

| Req | What | Status |
|---|---|---|
| R1 | One interaction: touch shelf, books float into an arc, touch cover to read | Done, verified on desktop |
| R2 | Real covers with generated fallback | Done, 12 of 12 verified over HTTP |
| R3 | Cozy room | Done, verified on desktop. World Labs Marble backdrop (stretch) not attempted |
| R4 | Remove atmosphere presets | Done |
| R5 | Fix plaques | Superseded: the shelf labels were removed entirely (they did not match the books on each shelf) |
| R6 | VR teleport | Built, logic tested on desktop with a simulated controller. **Not tested on a headset** |
| R7 | Housekeeping | Done (this commit) |

## Acceptance tests (run 2026-11-10)

| # | Test | Result |
|---|---|---|
| 1 | 360° from spawn never shows a black void | **Pass.** 36 views (12 headings × 3 pitches): at most 0.9% near-black pixels in any view |
| 2 | Plaque labels fully readable | **No longer applies.** Labels removed after iteration 2 (see below) |
| 3 | At least 10 of 12 image covers | **Pass.** 12 of 12 |
| 4 | Shelf, then cover, then open in two actions | **Pass.** Two real clicks opened a book; no extra buttons |
| 5 | Teleport to each bookcase and open a book in a Quest browser | **Not run.** Needs a headset |
| 6 | Cold-start stranger completes the flow in under 60 s | **Not run.** Needs a person |
| 7 | 72 fps on Quest 2 (OVR Metrics) | **Not run.** Needs a headset. Desktop frame: 123 draw calls, 5 lights, 1 shadow-casting light |

## Iteration log

### Iteration 1: rated 1.5 / 5

Root causes found in review: redundant shelf controls, atmosphere presets that only changed light and fog colors, clipped plaque labels, text-only covers, mostly black space (no background, no front wall, near-black ceiling), and no way to move in VR.

### Iteration 2 (2026-11-10): make the room real, simplify the interaction

Built in the spec's order, one commit per step: plaques and presets, interaction, covers, room, VR teleport, housekeeping.

**What was cut, and why**

- **Atmosphere presets and dock row (R4).** They changed only colors, so they did not make anything feel different. One finished room replaces them. Also removed from `src/library/reader-logic.ts` and its test.
- **Dock shelf buttons and `data-tier` handlers (R1).** Touching a shelf is now the only way to select it.
- **Auto-fan of the top shelf at startup.** The room now opens with every book at rest.
- **Moonlight directional light and the desk lamp's shadow cube map (R3).** To stay at five lights with one shadow caster for Quest 2. The window is an emissive dusk view instead.
- **The old 11 × 16 m room with a 5.2 m ceiling (R3).** Replaced by an 8 × 7.8 m closed room with a 3.4 m ceiling.
- **React scaffold (R7).** `src/App.tsx`, `src/main.tsx` and `src/index.css` were empty stubs that nothing referenced, so they were deleted.
- **`.aistudio/` screenshots (R7).** Removed, and `.aistudio/` is now in `.gitignore`.
- **Not built:** the World Labs Marble backdrop (optional stretch) and VR snap-turn (you can turn physically).

**Decisions and deviations from the spec**

- **Covers are stored locally (R2).** gutenberg.org sends no `Access-Control-Allow-Origin` header, so its images cannot be used as WebGL textures. This is the spec's fallback. The generated leather cover is still the placeholder and the fallback.
- **Books widened from 0.24 m to 0.28 m.** Gutenberg covers are about 0.65–0.72 wide-to-tall; the old book face (0.57) forced cropping that cut off titles.
- **The fan arc adapts to the screen.** It starts about 1 m from the viewer and backs off up to 1.6 m so all four covers fit within 80% of the horizontal field of view without overlapping.
- **Click versus drag.** A drag that ends in a click no longer counts as a click, so orbiting does not collapse the fan.
- **Brass is not metallic.** With no environment map, metallic materials rendered black, so brass uses low metalness.
- **The camera is clamped inside the room on desktop,** since the room is now smaller. In VR, teleport targets are limited to floor space in front of the bookcase and clear of furniture.
- **Dependency fix.** `esbuild` bumped to `^0.28.0` in `package.json` so `npm install` works.

**Fixes after first hands-on use (2026-11-10)**

- **FPS Walk could get stuck.** The mode switched on before the browser granted mouse lock, and a failed lock left clicks forced to the screen centre (so only the centre shelf and Dracula could be picked). The mode now turns on only after the lock succeeds. Where mouse lock is unavailable (embedded browsers, some iframes) it falls back to drag-to-look with a normal pointer.
- **Real mouse lock broke clicks and the reader.** With a genuine pointer lock the browser delivers clicks to the page body, which the click handler ignored, and the reader's buttons cannot be clicked while the mouse is captured. Clicks are now accepted while locked, opening a book releases the mouse (FPS stays on, with a normal cursor), and mouse picking is ignored while in a headset session.
- **Walk speed.** Movement was tied to frame rate (about 17 m/s on a fast display). It is now 2.4 m/s at any frame rate.
- **Walking into the bookcase.** The desktop camera can no longer pass through the bookcase.
- **Reader drop cap.** The drop cap no longer overlaps the third line, and a chapter heading is drawn as a heading instead of receiving the drop cap. Pages that open with a date or number (Dracula) get no drop cap.

**Changes after the prototype review (2026-11-10)**

- **Shelf labels removed.** "Victorian Novels", "Gothic & Speculative" and "World Classics" did not describe the books on those shelves, so the plaques are gone. Shelf grouping returns in a later phase, driven by real metadata (see `docs/PRD.md`).
- **Fuller pages.** Each page held about one paragraph (about 60 words). Each book now carries about 2,000 to 3,300 words of its real opening from Project Gutenberg (`books/excerpts.js`), paginated at runtime to fill each page (about 250 to 350 words). If that file is missing the old short pages are used.

**Planned next: flat reader (decided 2026-11-10, not built yet)**

Today the opened book is a tilted 3D object, which skews the pages and the text. The plan is for an opened book to read like iPad or Apple Books: flat to the screen, squared to the viewer.

- **Flat, square-on pages.** A short 3D transition brings the book from the shelf to the viewer ("semi-3D"), then it settles flat with no tilt or perspective skew. Two pages side by side on wide windows, one page on narrow ones.
- **No overlap with the menus.** The book is sized to fit the free area between the reader menu at the top and the dock at the bottom, with a margin, and re-fits when the window is resized or the dock is collapsed.
- **Remove the 3D "Return to shelf" plate above the book.** It duplicates the Return button in the top menu. Esc still closes the reader. **Do not remove it for VR until a VR replacement exists** (see the concerns below).
- **Flat page-turn animation** in place of the 3D flip.

**Concerns for the VR version of the flat reader**

1. **The menu is HTML, which a headset cannot show.** In an immersive session only 3D objects are visible. Prev, Next, View on Gutenberg and Return exist only in the HTML menu, so today the 3D plate above the book (and clicking the pages) is the only way to close and flip in VR. Removing the plate would strand VR users. Add VR-native controls first: for example a controller button to close, thumbstick left/right to flip, and small 3D buttons below the page (at least about 4 cm so a controller ray can hit them). Show them only in a headset.
2. **Where the flat page lives.** A page locked to the head is uncomfortable. Place it in the world once when the book opens (about 1.0 to 1.5 m away, facing the viewer, at a comfortable height for seated or standing), and offer a way to re-summon it after the viewer moves or teleports.
3. **Legibility.** Flat, square-on text should be sharper than the tilted version, but the current 768 by 1024 page texture at 19 px is still unproven in a headset. Expect to need a higher-resolution texture or SDF text, a larger font, and a one-page mode in VR. Test on a Quest 2 before choosing.
4. **Two sizing systems.** On desktop the book fits the browser window; in VR there is no window, so it needs a fixed physical size in metres. Plan for both modes in one layout function.
5. **One reader, not two.** Avoid an HTML reader on desktop and a textured-plane reader in VR that drift apart. Use the textured-plane reader everywhere and keep HTML only for the desktop menu.
6. **Page-turn cost.** Redrawing larger page textures on every flip can cause a frame hitch at 72 fps. Pre-render the next and previous pages.
7. **Less sense of a physical book.** A flat page loses depth cues. Keep the opening transition, a soft shadow and a hint of page edge so it still feels like a book.
8. **Dimmed room.** Reading mode dims the room lights. Keep the page unlit so its text stays crisp and independent of the room lighting, and check how the dimming looks in the headset.

See `docs/PRD.md` (section 7.3 and Phase 0b).

**Known issues and open items**

- **Excerpts, not full books.** Each book is an opening excerpt of 7 to 12 pages. Full-length reading is a later phase.

- **Quest 2 is untested:** frame rate, teleport on device (the thumbstick is assumed to be `axes[2]`/`axes[3]`), and how the arc line looks in the headset.
- **Blank first load:** the iteration 1 screenshot `r1-desktop.png` showed an empty canvas. It could not be reproduced in this iteration (every load rendered); the screenshot is gone with `.aistudio/`. Worth checking on a clean browser profile.
- **Reference photos.** `images/NY-rose-readingroom.png` has now been uploaded to the repo (it was an empty file). The photos are style guidance only and are not used in the scene.
- **`package.json` is still the AI Studio scaffold** (name `react-example`; React, Tailwind, Express and `@google/genai` dependencies). `index.html` uses none of them. `.env.example`, `metadata.json` and `vite.config.ts` also carry AI Studio and Gemini text. Pruning these would be a good small cleanup.
- **`src/library/reader-logic.ts` is not used by `index.html`.** It is a typed copy of the book data, kept because it has unit tests.
- **Cover art:** the covers come from Project Gutenberg's generated cover pages. Check their terms before publishing the project publicly.
