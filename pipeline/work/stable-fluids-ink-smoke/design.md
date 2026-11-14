# Stable Fluids Ink and Smoke — design

## Look

Black water. On each kick a punch of coloured ink shoots into the frame from one of eight emitters on a ring. It rolls up into a mushroom-shaped vortex pair, sheds thin curling filaments and slowly fades back into the dark. Only three inks are used, deep cobalt, hot magenta and amber, plus a warm white where they pile up. Where the inks cross they layer like stained glass instead of turning into brown soup. A soft relief light from the top left makes the plumes look thick and wet, and a light bloom makes the dense cores glow. Hi-hats twist the edges into fine curls. On the drop all eight emitters fire inward at once and collide in the centre. On a break the ink rises slowly like smoke (buoyancy mode) and thins out until the wall is almost empty. The mood is calm, liquid and physical: every swirl clearly comes from a hit. The reference is Pavel Dobryakov's browser sim ([GitHub](https://github.com/PavelDoGreat/WebGL-Fluid-Simulation)), with a stricter palette and beat-driven emitters instead of the mouse. The "ink from a gesture" idea and the rising and sinking blobs echo the ink and lava-lamp modes of Karl Sims' *Flow* ([karlsims.com/flow](https://karlsims.com/flow.html)).

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #000000, or a very dark #03040A in "water" mode | Normal | 100 % | empty space is a design element: keep at least 40 % of the wall dark |
| 2 | Obstacle mask (hidden, optional) | luma of a logo, type or video silhouette, threshold 0.5 | — | — | solid cells: the flow splits around them (Technique 6) |
| 3 | Velocity sim (hidden) | RG float grid, ping-pong: forces, curl, vorticity, divergence, pressure Jacobi, gradient subtract, advection | — | — | the physics. It is never shown directly |
| 4 | Dye sim (hidden) | RGBA float grid at a higher resolution, advected by layer 3 and faded by dissipation | — | — | R, G, B = amounts of ink A, B, C, not screen colours |
| 5 | Ink display | dye → palette matrix → tone curve → relief shading | Normal over 1 | 100 % | the main visible layer |
| 6 | Displacement target (optional) | another clip warped by the velocity field | Normal, or Add under the ink | 0–100 % | variation: "stable-fluids displacement" |
| 7 | Post | bloom, black crush, grain | see compositing | — | bloom carries the "dense core" glow |

## Techniques

**1. Semi-Lagrangian advection (moving everything)**
1. Each cell asks where its content was one time step ago: `p = uv − dt · v(uv) · texel`. Here v is in sim cells per second and `texel = 1/gridSize`.
2. It reads the field there with bilinear filtering and writes that value. This backward trace is stable for any dt. That is why the method suits real time ([Wikipedia: Fluid animation](https://en.wikipedia.org/wiki/Fluid_animation)), and why it fits fragment programs on the GPU ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)).
3. The velocity field advects itself (self-advection). The dye is advected by the velocity, sampled at the dye pixel's uv, so dye and velocity can have different resolutions.
4. Dissipation in the same pass: `result = sample / (1 + d · dt)`, the form used in Dobryakov's advection shader ([script.js](https://raw.githubusercontent.com/PavelDoGreat/WebGL-Fluid-Simulation/master/script.js)). Dye d = 1.0 gives about a 1 s fade. Velocity d = 0.2 lets motion coast for several seconds.
5. Clamp dt to 0–1/30 s. A dropped frame then cannot throw ink across the wall.

**2. Forces: splats, emitters, buoyancy**
1. A splat is a Gaussian added to a field: `w = exp(−d² / r)`, with `d = (uv − c)` and the x component multiplied by the aspect ratio so splats stay round. Dobryakov uses `r = SPLAT_RADIUS / 100` (default 0.25 / 100) and scales the radius by the aspect on wide canvases ([script.js](https://raw.githubusercontent.com/PavelDoGreat/WebGL-Fluid-Simulation/master/script.js)). GPU Gems also injects user force as a 2D Gaussian splat into the velocity texture ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)).
2. Velocity splat: `v += w · dir · force`. Force is in screen heights per second (default 1.5, range 0.3–4), converted to cells per second by multiplying with the grid height.
3. Dye splat: `dye += w · inkVector · amount`, where inkVector is (1,0,0), (0,1,0) or (0,0,1) for ink A/B/C, amount 0.3–1.0. Dobryakov keeps splat colour brightness low (HSV value × 0.15) so that overlaps do not clip ([script.js](https://raw.githubusercontent.com/PavelDoGreat/WebGL-Fluid-Simulation/master/script.js)). Here the tone curve in the display pass does that job.
4. Ring emitters: 8 points on a circle of radius 0.32 screen heights around the centre. Each fires inward (or tangentially, for a spin), selected by `beatCount mod 8`. On the drop all 8 fire inward together.
5. Buoyancy (smoke mode): `v.y += buoyancy · (dyeSum − 0.05)`, where dyeSum = R + G + B of the dye at that cell. Dense ink rises. A negative buoyancy makes ink sink. With ink A positive and ink C negative, the plumes separate like the lava-lamp mode in *Flow* ([karlsims.com/flow](https://karlsims.com/flow.html)).

**3. Projection: making the flow incompressible**
1. Divergence per cell: `div = 0.5 · ((vR.x − vL.x) + (vT.y − vB.y))`, from the 4 neighbours.
2. Pressure solve with Jacobi iterations: `p' = (pL + pR + pB + pT − div) / 4`, repeated N times with a ping-pong pair of textures.
3. Gradient subtract: `v −= 0.5 · (pR − pL, pT − pB)`. The flow is now divergence-free, so it swirls instead of piling up ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)).
4. Iteration count: GPU Gems suggests 40–80 and warns that below 20 the error becomes noticeable ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)). Dobryakov gets away with 20 by warm-starting: before iterating he multiplies last frame's pressure by `PRESSURE = 0.8` instead of clearing it ([script.js](https://raw.githubusercontent.com/PavelDoGreat/WebGL-Fluid-Simulation/master/script.js)). This design defaults to 24 iterations with a 0.8 warm start, and allows 40 when the GPU has headroom.
5. Keep N even so the result always lands in the same buffer, P_A, which is also next frame's warm start.

**4. Vorticity confinement (keeping the curls)**
1. Curl per cell, as a scalar in 2D: `ω = 0.5 · ((vR.y − vL.y) − (vT.x − vB.x))`.
2. Direction towards stronger swirl: `g = (|ωT| − |ωB|, |ωR| − |ωL|)`. Normalise it with a small epsilon (1e−5) added to the length.
3. Force `f = swirl · ω · g`, with the y component negated so it acts perpendicular to the gradient, as in Dobryakov's vorticity shader. Then `v += f · dt`, clamped to ±1000 cells/s ([script.js](https://raw.githubusercontent.com/PavelDoGreat/WebGL-Fluid-Simulation/master/script.js)).
4. The method comes from Fedkiw et al. (2001) and is not part of Stam's original scheme ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)). Dobryakov's default `CURL = 30` (on a 128-cell grid) is the reference. Here the range is 0–60, 0 gives soft, blurry plumes and 60 gives tight, nervous curls.

**5. Boundaries**
1. Walls (default): sample pressure with clamp-to-edge addressing. The neighbour outside the grid then equals the edge value, which gives the pure Neumann pressure condition from GPU Gems for free. In gradient subtract, set the velocity component normal to the wall to 0 in the outermost cell row and column (free slip). GPU Gems uses no-slip, which negates the whole velocity at the wall ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)). Free slip lets the ink slide along the LED frame instead of sticking.
2. Wrap mode (360° rooms, LED strips that close into a ring): use repeat addressing on all sim textures for x and walls only for y.

**6. Obstacles (variation "obstacle dance")**
1. Mask texture M at velocity resolution: 1 = solid. Feed a blurred logo or silhouette through a threshold at 0.5.
2. Jacobi: if a neighbour is solid, use the centre pressure in its place (Neumann at the obstacle).
3. Gradient subtract and velocity advection: velocity = 0 inside solids. Dye advection: dye × (1 − M), so ink never sits inside the logo.
4. GPU Gems notes that arbitrary boundaries cost extra per-cell branching and are slower ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)). Expect about +20 % on the velocity passes. Moving obstacles push no fluid in this simple version. They only block it. For "push", add a velocity splat along the silhouette's frame-to-frame motion.

## Shaders

Render order per frame (Dobryakov's step order, with forces first ([script.js](https://raw.githubusercontent.com/PavelDoGreat/WebGL-Fluid-Simulation/master/script.js))):
S1 forces → S2 curl → S3 vorticity → S4 divergence → S5 pressure warm start → S6 Jacobi × N → S7 gradient subtract → S8 advect velocity → S9 advect dye (+ dye splats) → S10 display → S11 post.

Grid sizes for a 1920×1080 output: **velocity grid 320×180** (default; 160×90 for big lazy plumes, 480×270 for fine detail) and **dye grid 1280×720** (Dobryakov defaults: SIM 128, DYE 1024 on the short or long side ([script.js](https://raw.githubusercontent.com/PavelDoGreat/WebGL-Fluid-Simulation/master/script.js))). Curl, divergence and pressure share the velocity grid. Cells stay square, so the grids follow the output aspect.

| buffer | size | format | filter | persistent across frames |
|--------|------|--------|--------|--------------------------|
| VEL (ping-pong V0/V1) | velocity grid | RG16F (RGBA16F if RG is not renderable) | linear | yes |
| CURL | velocity grid | R16F | nearest | no |
| DIV | velocity grid | R16F | nearest | no |
| P (ping-pong P_A/P_B) | velocity grid | R16F | nearest | yes (warm start) |
| DYE (ping-pong D0/D1) | dye grid | RGBA16F | linear | yes |
| OUT | output | RGBA16F before bloom, RGBA8 after | — | no |

Dobryakov requests RGBA16F / RG16F / R16F and filters dye and velocity linearly, but divergence, curl and pressure with nearest ([script.js](https://raw.githubusercontent.com/PavelDoGreat/WebGL-Fluid-Simulation/master/script.js)). 8-bit buffers do not work: signed velocities and pressure need a float format.

**S1 — Forces** (velocity grid)
- Inputs: VEL read, emitter list (up to 8 positions, directions and strengths this frame), `force`, `radius`, `buoyancy`, DYE read (for buoyancy), `dt`.
- Per pixel: sum the Gaussian velocity splats → add buoyancy from the dye sum → write.
- Output: VEL write. Swap.

**S2 — Curl** (velocity grid)
- Inputs: VEL. Per pixel: 4 neighbour fetches → scalar ω. Output: CURL.

**S3 — Vorticity** (velocity grid)
- Inputs: VEL, CURL, `swirl`, `dt`. Per pixel: 5 CURL fetches → force (Technique 4) → `v += f · dt`, clamp. Output: VEL write. Swap.

**S4 — Divergence** (velocity grid)
- Inputs: VEL, obstacle mask. Per pixel: 4 neighbours (solid or out-of-grid neighbours count as the mirrored centre velocity, so walls do not leak) → div. Output: DIV.

**S5 — Pressure warm start** (velocity grid)
- Inputs: P_A. Per pixel: `p = 0.8 · p` (`warm` 0–1, 0 = clear every frame). Output: P_B. This pass and S6 also form the ping-pong.

**S6 — Jacobi × N** (velocity grid, N = 24 default, 20–40, even)
- Inputs: P (current), DIV, mask. Per pixel: 4 P fetches + 1 DIV → `p' = (ΣpN − div) / 4`. Output: the other P buffer. With S5 writing P_B, the iterations alternate B→A, A→B, … and an even N ends in P_A.

**S7 — Gradient subtract** (velocity grid)
- Inputs: VEL, P_A, mask. Per pixel: `v −= 0.5 · ∇p`, zero the wall-normal component at the edges, v = 0 in solids. Output: VEL write. Swap.

**S8 — Advect velocity** (velocity grid)
- Inputs: VEL, `dt`, `drag` (velocity dissipation). Per pixel: backtrace, bilinear sample, divide by `(1 + drag · dt)`. Output: VEL write. Swap. This is the state for next frame.

**S9 — Advect dye + dye splats** (dye grid)
- Inputs: DYE read, VEL (bilinear, sampled at the dye pixel uv), `dt`, `trail` (dye dissipation), dye splat list, mask.
- Per pixel: backtrace with the velocity (in velocity-grid texel units) → sample DYE → divide by `(1 + trail · dt)` → add dye splats → × (1 − mask) → clamp to 0–8.
- Output: DYE write. Swap. Persistent.

**S10 — Display** (output resolution)
- Inputs: DYE (bilinear upscale), palette colours, `exposure`, `relief`.
- Per pixel: (1) `c = R·inkA + G·inkB + B·inkC` (a 3×3 palette matrix). (2) Tone curve `c = 1 − exp(−c · exposure)`, exposure 1.5, so piled-up ink rolls off into colour instead of clipping. Then lerp towards the highlight #FFF1D6 by `smoothstep(2.0, 4.0, R+G+B)`. (3) Relief: height h = R+G+B, normal from central differences × relief (0–3), light from (−0.5, 0.7, 0.5), `c *= 0.75 + 0.25 · diffuse`. Dobryakov's sim has a comparable SHADING switch ([script.js](https://raw.githubusercontent.com/PavelDoGreat/WebGL-Fluid-Simulation/master/script.js)).
- Output: OUT (RGBA16F).

**S11 — Post** (output; bloom chain at 1/4 resolution)
- Bloom threshold 0.6, soft knee 0.7, intensity 0.8. These are Dobryakov's defaults, which use an 8-step bloom chain at 256 px ([script.js](https://raw.githubusercontent.com/PavelDoGreat/WebGL-Fluid-Simulation/master/script.js)). Here use 5 downsample steps from 480×270.
- Black crush `c = max(c − 0.015, 0) / 0.985`, then grain 1 % monochrome.

Licence note: Dobryakov's sim is MIT licensed ([GitHub](https://github.com/PavelDoGreat/WebGL-Fluid-Simulation)). This design only restates the method and his default values. If a sketch ports his GLSL directly (for example into an ISF file), it must keep his copyright notice and the MIT licence text in the file header.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base / black | #000000 | background, dye = 0 |
| water (optional) | #03040A | barely lifted black for "ink in water" |
| ink A (base) | #1F4BFF | cobalt, the bass and kick ink, most of the volume |
| ink B (accent) | #FF2D8A | hot magenta, snare and the drop |
| ink C (accent 2) | #FFA21F | amber, hats and highlights in the swirls |
| highlight | #FFF1D6 | where total ink density exceeds about 3 |
| shadow (relief) | multiply 0.75 | the dark sides of the plumes, from the relief light |

- Mapping: the dye stores ink amounts, not colours. The palette matrix in S10 turns them into colour, so palettes change instantly and without re-simulation. Overlaps add up in a controlled way (cobalt + magenta → violet, magenta + amber → coral), and the tone curve stops them from clipping to grey-white. This is the research's "palette lock" fix against rainbow mud.
- Over time: per phrase (8 bars), rotate the three ink hues together by up to ±25°. On the drop, swap ink A and B for one phrase (the wall turns magenta-dominant). Never more than 3 inks at once.
- Mono smoke mode (break): all three inks map to #D8DCE6 at 60 % brightness, with buoyancy on. The image is grey smoke on black.
- LED walls: true black at dye < 0.02 (after the crush). Peak white about 85 %. Cobalt #1F4BFF is dark in luminance, so it can fill large areas without blinding. Magenta and amber should cover under 25 % of the wall. The dye's gradients are smooth, so the grain is needed against banding on 8-bit processors.

## Blending and compositing

- Ink display in Normal over black. The tone-mapped dye is opaque. For an overlay over video, output alpha = `1 − exp(−(R+G+B) · 1.5)` with premultiplied colour, and put the layer in Normal over the clip (smoke over footage).
- As a matte: total ink density as a luma mask on a video layer. The video only appears inside the plumes.
- Displacement variation: the velocity field (S8 output, scaled by 0.002 screen units per cell/s) offsets the UVs of another clip. Use the clip as dye if it should smear, or warp it fresh every frame if it should only ripple.
- Feedback: the sim is the feedback loop. Its decay is `trail` (dye) and `drag` (velocity). Do not add a host feedback effect on top, because that doubles the smearing and the image goes muddy.
- Post chain order: display tone curve → bloom → black crush → grain.

## Motion and timing

- What moves: the emitter impulses, the flow they create, vorticity curls, and buoyant drift. Nothing moves without forces, and an idle sim settles to black within `trail`-dependent seconds.
- Base groove (128 BPM): every beat one ring emitter fires inward with ink A (force 1.5, radius 0.0025, amount 0.6). The emitter index advances by 3 each beat (3 is coprime to 8), so the hits spread around the ring. Every bar the downbeat emitter uses ink B.
- Splat envelope: a hit applies its force over 3 frames (attack about 50 ms), not as a single-frame spike. This gives rounder mushrooms and fewer pressure artefacts.
- Bar / phrase: every 4 bars the emitters switch from inward to tangential (all clockwise) for one bar, and the whole field spins. Every 8 bars the palette rotates (see colour).
- Build-up: over the last 8 bars, swirl rises from 30 to 55, trail falls from 1.0 to 0.5 (longer trails, the wall fills), and the emitter rate doubles to 1/8 notes in the last bar.
- Drop: on the downbeat all 8 emitters fire inward with force 3.5, inks alternating A/B. The central collision throws ink outward within one beat. For the next 4 bars use radius × 1.5 and drag 0.1.
- Break: buoyancy goes to +40 over 2 bars, the mono smoke palette fades in over 1 bar, trail rises to 2.0 (fast fade) and emitters fire only every 2 bars from the bottom edge. Uli can paint smoke by hand with the KeyStep (pitch = x position, mod strip = force).
- Clear-on-phrase (optional): a "flush" trigger sets trail to 8 for 0.5 s. The wall empties quickly without a hard cut.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| punch | 0.3–4 screen heights/s | 1.5 | strength of each emitter hit: small puffs to violent mushrooms | kick envelope × knob 1 (BeatStep Pro) |
| trail | 0.2–4 (dye dissipation, per s) | 1.0 | how long ink stays: 0.2 fills the wall, 4 leaves only short flashes | knob 2; "flush" pad sets 8 for 0.5 s |
| swirl | 0–60 (vorticity) | 30 | soft plumes ↔ tight nervous curls | knob 3; hat envelope adds +15 |
| drag | 0.05–2 (velocity dissipation, per s) | 0.2 | how long motion coasts after a hit (thick honey ↔ thin water) | knob 4 |
| buoyancy | −40 to +40 cells/s² per unit ink | 0 | smoke rising (+) or ink sinking (−) | KeyStep mod strip, or a switch for the break macro |
| ink | 0–1 | 0 | palette rotation and A/B swap (0–0.5 rotates hue ±25°, above 0.5 swaps A and B) | knob 5; jumps on phrase boundaries |

Performance controls, not faders: emitter mode (inward / tangential / all-fire), reset (clears VEL, P and DYE), flush, pressure iterations (24/32/40, a setup value, not a live fader).

## Build recipes

### Resolume Arena

- This design assumes no native fluid solver in Arena. It needs either an ISF multipass effect (route 1) or an external sim sent in by Spout (route 2).
- Route 1, ISF multipass effect:
  - ISF passes can set `TARGET`, `PERSISTENT`, `FLOAT` (32-bit per channel) and `WIDTH`/`HEIGHT` as expressions of `$WIDTH`/`$HEIGHT`. Persistent buffers are resized, and so reset, when the render size changes ([ISF JSON reference](https://docs.isf.video/ref_json.html), [ISF multipass](https://docs.isf.video/ref_multipass.html)). The shader branches on `PASSINDEX`. `FRAMEINDEX == 0` triggers the init, and `TIMEDELTA` gives dt ([ISF variables](https://docs.isf.video/ref_variables.html)).
  - Pass list (N = 20, 29 passes): 0 forces → `velF`; 1 curl → `curlB`; 2 vorticity → `velV`; 3 divergence → `divB`; 4 warm start from persistent `pA` → `pB`; 5–24 Jacobi alternating `pB → pA`, `pA → pB`, … (an even N ends in `pA`); 25 gradient subtract → `velP`; 26 advect `velP` → persistent `vel`; 27 copy persistent `dye` → `dyeOld`; 28 advect `dyeOld` + dye splats → persistent `dye`; then the display pass with no target. Every sim buffer needs `FLOAT: true`. Velocity buffers use `$WIDTH/6`, `$HEIGHT/6` (320×180 at 1080p). Dye buffers use `$WIDTH/1.5`, `$HEIGHT/1.5`.
  - The dye copy pass (27) is there because the ISF pages do not say whether a pass may read neighbouring pixels from the persistent buffer it is writing. The copy avoids relying on that. It costs one cheap pass.
  - ISF has no loops over passes, so the iteration count is fixed by the JSON. Make 20, 32 and 40 variants. Inputs: floats `punch`, `trail`, `swirl`, `drag`, `buoyancy`, `ink`, `beatPhase` (0–1); a long `emitterIndex` or a float `beatCount`; `point2D` hand splat; `event` reset and flush; `color` inks A/B/C. Emitter hits come from `beatPhase`: the force envelope is `exp(−beatPhase · 12)`, so each beat gives a short push without needing extra state. Drive `beatPhase` with a BPM-synced sawtooth in Arena's parameter animation and `punch` from the audio analysis on the Low band.
  - Version caveat: Resolume documents ISF only in its Wire pages ([Resolume ISF](https://resolume.com/support/en/isf)), and multipass with persistent and FLOAT buffers depends on the version. Pre-gig test: fire one splat with drag 0.05. If it stops dead, shows stair-steps, or the velocity only moves in one direction (negative values clipped), the buffers are 8-bit. The fluid cannot be rescued with 8-bit packing at these pass counts, so switch to route 2.
- Route 2, external sim (most reliable): run the sim in UE or Godot (below), or in TouchDesigner, where the same splat/curl/vorticity pipeline is shown in a tutorial ([threedashes, YouTube](https://www.youtube.com/watch?v=2k6H5Qa_fCE)). Bring it into Arena as a Spout source. Send alpha (premultiplied) if the ink is used over clips.
- Layer setup: layer 1 is an optional clip, layer 2 the fluid (ISF or Spout) in Normal. Bloom is Arena's Bloom on the composition, with values as in S11, if the source does not bloom itself.

### Resolume Wire

- State lives in the ISF node: the 29-pass JSON from route 1 runs inside Wire's ISF node, and the node owns the persistent buffers. Wire has no vertex shaders (as of version 7.22) ([Resolume ISF](https://resolume.com/support/en/isf)). This design does not need them, because every pass is a full-screen fragment pass.
- Node outline: Effect input (optional obstacle or source video) → Threshold / luma → Resize to velocity-grid size → ISF node `stable-fluids` → Output. Run the same input in parallel through Blur (2 % radius) before the threshold, so silhouette edges do not create pressure noise.
- Control side: BPM / clock → phase (0–1 per beat) → `beatPhase`; beat counter → `beatCount`. Audio analysis Low → envelope (attack 10 ms, release 200 ms) → `punch` multiplier. High → envelope (5 ms / 120 ms) → added to `swirl`. A bar counter mod 4 → emitter mode.
- Exposed parameters: punch, trail, swirl, drag, buoyancy, ink, reset, flush. Node names for the envelope, counter and phase differ between Wire versions, so check the node browser. Do the 8-bit test from route 1 first.

### Unreal Engine 5.7

- Route A, render-target ping-pong (deterministic, recommended for this exact look):
  - Render targets (Texture Render Target 2D, mips off, clamp addressing): `RT_V0/RT_V1` 320×180 RG16f (or RGBA16f), linear filter; `RT_Curl`, `RT_Div` 320×180 R16f, nearest; `RT_P0/RT_P1` 320×180 R16f, nearest; `RT_D0/RT_D1` 1280×720 RGBA16f, linear; `RT_Out` 1920×1080 RGBA8.
  - One Unlit material per pass (`M_FluidForces`, `M_Curl`, `M_Vorticity`, `M_Divergence`, `M_PressureWarm`, `M_Jacobi`, `M_GradSub`, `M_Advect` used for both velocity and dye with a `Dissipation` parameter, `M_Display`). Each has a Texture Object parameter for its inputs, a `Texel` vector parameter and 4–5 Texture Sample nodes at offset UVs, or one Custom node.
  - Blueprint on Tick: call Draw Material to Render Target for S1–S10 in order, swapping the RT references after each write. For S6 use a For Loop with N iterations (an int variable). The draws queue on the render thread in call order, so 30+ draws per tick are fine. Reset: Clear Render Target 2D on all sim RTs.
  - Emitters: the Blueprint keeps a small array (max 8) of splat positions, directions and inks for this frame and writes them into the MID as vector parameters (`Splat0` … `Splat7`, xy = pos, zw = dir; plus `SplatInk0–7`).
- Route B, Niagara Fluids 2D gas: Epic's Niagara Fluids include grid-based 2D gas. It stores fields in Grid 2D Collections and runs the solver as simulation stages, where a stage can repeat with Num Iterations and the iterative pressure solve trades cost for accuracy ([Epic, fluid simulation overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/fluid-simulation-in-unreal-engine---overview)). Start from a 2D gas template in the New Niagara System menu, then replace the source with beat emitters (user parameters for position, force and ink) and render the density into a Render Target. This route is faster to start, but the look is less controllable than route A.
- Output: Spout (via a Spout plugin) or NDI of `RT_Out` into Arena. Or show `RT_Out` on a full-screen plane in front of a fixed camera.
- Remote Control: a Material Parameter Collection `MPC_Fluid` (`Punch`, `Trail`, `Swirl`, `Drag`, `Buoyancy`, `Ink`, `InkA/B/C` colours). Blueprint variables `PressureIterations`, `EmitterMode` and functions `Reset`, `Flush`, `FireAll` go into a Remote Control Preset, bound to OSC (from Wire or Arena) and MIDI (BeatStep Pro knobs 1–5, KeyStep notes → hand splat x, mod strip → buoyancy).

### Godot

- Godot 4.x, Forward+ renderer. Compute shaders need a RenderingDevice-based renderer (Forward+ or Mobile), and mobile drivers are unreliable ([Godot docs, compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)).
- State: the same buffers as the table in Shaders, created on the main RenderingDevice as storage images (16-bit float RGBA or RG formats). Advection reads through a sampler with linear filtering, and writes go through image stores.
- One compute shader per pass (or one shader with a `pass` push constant), with 8×8 workgroups: 40×23 groups for the velocity grid, 160×90 for the dye grid. The script records S1–S9 every frame in order, with N Jacobi dispatches, swapping bindings after each write. Do not call `sync()` every frame. The docs recommend waiting 2–3 frames, and splitting long work into several dispatches avoids a Windows TDR ([Godot docs, compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)).
- Display: wrap the final dye image in a `Texture2DRD` via `texture_rd_rid` ([Godot docs, Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html)) and draw it on a full-screen ColorRect with a `canvas_item` shader that does S10. Bloom comes from a WorldEnvironment glow, or from a SubViewport with `use_hdr_2d = true` (RGBA16 framebuffer ([Godot docs, Viewport](https://docs.godotengine.org/en/stable/classes/class_viewport.html))).
- No clean Compatibility fallback: S6 needs 20+ ordered passes per frame. A SubViewport chain (`UPDATE_ONCE`, `CLEAR_MODE_NEVER` ([Godot docs, SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html))) can only give one ordered step per frame, which is too few for the pressure solve.
- Uniforms / push constants to expose: `punch`, `trail`, `swirl`, `drag`, `buoyancy`, `ink`, `ink_a/b/c`, `exposure`, `relief`, `warm`, `pressure_iterations`, and the splat array (up to 8 × vec4 + ink index). Audio comes from AudioEffectSpectrumAnalyzer on the music bus. Output to Arena via a Spout GDExtension if available, otherwise NDI.

## Audio reactivity

| band | drives | smoothing | threshold / scaling |
|------|--------|-----------|---------------------|
| kick, 40–120 Hz | fires the next ring emitter (ink A), force = punch × hit strength | attack 10 ms, release 200 ms; force applied over 3 frames on the rising edge | trigger at 0.6 of the running peak (2 s window), retrigger blocked 120 ms |
| bass, 60–250 Hz | splat radius × (1 to 2) and drag −30 % | attack 80 ms, release 800 ms | linear above a 0.2 floor |
| snare / clap, 1–4 kHz | a mirrored emitter pair with ink B | attack 5 ms, release 150 ms | trigger at 0.5 |
| hats, 6–12 kHz | swirl +15 and tiny ink-C splats (radius × 0.3) at random edge points | attack 5 ms, release 120 ms | trigger at 0.5, max 4 per beat |
| RMS loudness | trail from 1.5 (quiet) to 0.6 (loud), so loud sections fill the wall | attack 500 ms, release 4 s | clamp to the trail fader range |

- Do not modulate pressure iterations or grid size with audio, because changes reallocate buffers or make the solve jitter. Never drive buoyancy from fast audio either, because the whole field lurches.

## Performance tips

- Cost (320×180 velocity grid, 1280×720 dye, 24 Jacobi iterations): the velocity grid is 57.6 k cells. All velocity passes together are about 30 passes × 5 fetches, under 9 M fetches. Dye advection is 0.9 M pixels × 2 fetches. Display at 1080p is about 5 fetches per pixel. In total this is well under 1 ms on a current desktop GPU. Bloom is the next largest cost.
- The pressure solve is the main sim cost ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)), but at 320×180 even 40 iterations are cheap. In ISF the cost is per-pass host overhead, not fetches, so 29 passes can cost more than the maths. Keep N at 20 in ISF.
- Resolution choice: the velocity grid sets the size of the swirls, the dye grid sets the sharpness of the filaments. Raise the dye resolution first. Doubling the velocity grid quadruples the cells and needs about twice the Jacobi iterations for the same quality, because the pressure needs more steps to cross the grid.
- Odd aspect ratios: keep cells square. For a 5760×1080 strip use a 960×180 velocity grid and a 3840×720 dye grid, and use the splat aspect correction, otherwise splats turn into ellipses. Pressure has to cross a long grid, so give strips 32–40 iterations, or the flow compresses at the far ends.
- Big LED walls: large pixel pitch hides fine filaments, so lower the dye grid (960×540) and raise the splat radius instead.
- Pitfalls: 8-bit buffers (dead or one-directional flow); odd Jacobi counts without tracking the current buffer (pressure flickers); forgetting the dt clamp (ink explodes after a hitch); no dissipation (grey soup, the research's main failure); too many inks (mud); mipmaps on sim textures (blurred, unstable sampling).

## Further references

- [NVIDIA GPU Gems, ch. 38: Fast Fluid Dynamics Simulation on the GPU](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu) — the GPU version of Stam's method: the pass order, Jacobi counts, boundaries and vorticity confinement.
- [Pavel Dobryakov, WebGL Fluid Simulation](https://github.com/PavelDoGreat/WebGL-Fluid-Simulation) (MIT, [live demo](https://paveldogreat.github.io/WebGL-Fluid-Simulation/)) — the reference look. Its `script.js` holds the defaults, formats and the warm-started pressure used in this design.
- [ISF Multi-Pass and Persistent Buffer Reference](https://docs.isf.video/ref_multipass.html) — PERSISTENT and FLOAT buffers, the basis of the Arena/Wire route.
- [Epic, Fluid Simulation in Unreal Engine overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/fluid-simulation-in-unreal-engine---overview) — Niagara Fluids 2D gas, Grid 2D Collections and simulation stages with Num Iterations.
