# House 3D model – project notes

Interactive 3D study model of the entrance floor (קומת כניסה) of our house, built with three.js r128.
Live site: https://talizkmail-ui.github.io/rt/ (GitHub Pages from `main` / root).

## Files
- `index.html` – the whole app: HTML, CSS and JS in one file. Two images are embedded as base64 data URIs: the plan underlay (`PLAN_IMG`) and the natural-stone texture (`stoneImg`, cropped from a photo of the real wall).
- three.js and its add-ons (OrbitControls, Sky, EffectComposer + SAOPass and their shaders) load from jsDelivr (`three@0.128.0`). Keep versions pinned.

## Rendering (photo-real look)
- Linear workflow: `outputEncoding = sRGB`, ACES filmic tone mapping (exposure 0.8). `M()` converts hex colours from sRGB, and every colour texture sets `encoding = sRGBEncoding` – do the same for new materials and textures, or they look washed out.
- Light: `THREE.Sky` with the sun direction `SUN_DIR`, baked through PMREM into `scene.environment` for reflections and fill light. `ENV` scales how strong that sky light is relative to the sun.
- Textures are procedural canvases (plaster, porcelain tile, concrete, pavers, grass). `worldUV(material, metres)` maps them in world space, so they keep the same scale on any box size.
- Post-processing: `composer` = render → SAO ambient occlusion → gamma. The loop calls `composer.render()`, not `renderer.render()`. SAO's depth-texture mode is broken in r128, so keep `SAOPass(scene,camera,false,true)` and the camera near plane at 0.5.

## Sources
- Sheet 4 – entrance floor plan (scale traced from the 1390 cm overall width).
- Sheets 8–9 – elevations. Floor ±0.00 = 39.50, walls to +3.35, roof edge +3.85, lower floor −3.30, window heads +2.20 (sill +1.10), large glazing heads +2.80, front door +2.40, garage +2.45.
- Owner photos and feedback. **Where the owner's description differs from the drawings, the owner wins** – the drawings show the design, the owner knows what was built.

## Coordinate system
- All model data is in **centimetres**; the scene is in metres (`box()` divides by 100).
- Origin = outer NW corner of the kitchen. **x** runs right on the plan sheet, **y** runs down the sheet, **z** is height. Three.js mapping: X = x, Y = z, Z = y.
- The top of the plan sheet is the **front / entrance side** (east in the elevations).

## How the code is organised
- `mat` – shared materials. `mat.ext` = light-grey exterior plaster, `mat.int` = white interior.
- `W` – wall list. `ext(x1,y1,x2,y2,thickness,openings,opts)` / `int(...)` for axis-aligned walls.
- Openings: `G(s,e,sill,top)` = window (glass + black frame + roller shutter if exterior), `D(s,e,top)` = door hole. Extra flags: `leaf:'black'|'garage'|'door'|'glassdoor'`, `transom`, `noShutter`.
- Wall opts: `h` (wall height), `stone:'-x'|'+x'|'-z'|'+z'` (outer face clad in stone), `stoneTop` (stone only below this height).
- `buildWalls()` rebuilds all walls; it's called on load and when the wall height, cut, roof or stone image changes. Exterior faces get plaster vs. white using `inside(x,y)` against the `INSIDE` footprint rectangles.
- Roller shutters: state lives in `shutState` (keyed per opening), animated in the render loop; clicking a window toggles it.
- Fixed items (floors, kitchen, bathrooms, stairs, railings) are added once to `groups.fixed`; outdoor items to `groups.outdoor`.
- Room labels: the `rooms` array, `[english, hebrew, x, y, outdoorFlag]`.

## Decisions confirmed by the owner
- Stairs go **down** to the lower floor (U-shape, 19 risers, 3.30 m), first flight along the study wall. Oak treads.
- Stair railings: black steel with expanded-metal mesh, sloped to follow each flight; no wall between flights.
- Stair window: full height, down to the terrace floor. Interior window from the stairwell to the study.
- Stone cladding (natural stone from the photo): the wall around the front door (incl. the guest WC window), the garage wall with the side door (below the long window only), and the stair side of the study wall. The kitchen-window front wall is plain light-grey plaster, not stone.
- Garage: closed, walls to +2.45, no roof, one black car door on the front, black side door with a long window strip above it running the length of the wall.
- All interior walls white. Exterior plaster light grey. Black window frames, black outside doors, aluminium + glass utility door to the outside. Black roller shutters on all exterior windows.
- Kitchen: green fronts, white marble-look worktop, U-shape with oak-veneer peninsula back, waterfall end, wine fridge; tall wall = open oak niche, oven tower, 2 × 60 cm fridges, tall cabinet; tall units to 2.70, drywall above to the ceiling, flush return wall closing the side.
- Roof: 50 cm fascia band, 80 cm overhang at the front, open slatted roof (4 beams) over the entrance passage.
- No gap between the utility room and bedroom 2; utility door opens from the hall.
- No window from the front bedroom onto the entry passage (its wall at x = 777 is blank).
- The roof is hidden by default; the Roof button turns it on.

## Open questions
- Where the garage's second regular door is.
- Where the kitchen cooktop is (not visible in the kitchen renders).
- Lower floor and sloping site are not modelled yet (needs the lower-floor plan).
