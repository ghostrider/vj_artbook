# Risograph Overprint and Misregistration — design

## Look

Any footage comes out as a cheap, gorgeous two- or three-colour print: fluorescent pink, blue and yellow ink on warm off-white paper, never a pure RGB pixel. Each ink is its own flat "plate" built from coarse halftone dots or sandy grain. Where plates overlap the inks multiply into new colours (pink over blue gives a deep violet-blue, pink over yellow a hot red-orange, yellow over blue a grassy green). The plates are never quite in register. Each one sits a few pixels off and a fraction of a degree rotated, so edges get coloured fringes and thin slivers of bare paper. The image runs at 8 to 12 "printed frames" per second, and every frame lands slightly differently, so the picture breathes and jitters like the scanned-print animations of Julia Schimautz and Hiromu Oka (see research.md). The mood is zine, gig poster and children's book: warm, tactile, a bit loud, clearly handmade. On the kick the plates fly apart for a moment and snap back. On a phrase change the ink set swaps.

Design rules: **(1) inks only subtract.** Every plate multiplies onto paper, so the image can never be brighter than the paper. **(2) Misregistration must vary**, between plates, between frames and across the frame, otherwise it reads as a cheap RGB-split preset (research.md, "boring risks").

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Paper | flat paper colour (`#F4EFE4` default) plus a low-contrast fibre/noise texture | Normal | 100 % | the base white; switch between off-white, newsprint and kraft stock |
| 2 | Source | any clip, camera, Spout/NDI feed from UE or TouchDesigner | not visible directly, feeds the separation | — | high-contrast footage with big shapes works best; busy detail turns to mush at coarse dot sizes |
| 3 | Print clock | sample-and-hold of the source at `print_fps` (8–12 fps) | inside the effect | — | the "every frame was printed" feel; bypass for smooth video |
| 4 | Separation | source → 1 to 3 density maps (tonal bands or channel matrix) | inside the effect | — | decides which ink goes where |
| 5 | Plate 1 (usually Blue `#0078BF` or Black) | density → halftone/grain screen, own offset, rotation, jitter | Multiply onto layer 1 | `ink_1` 60–100 % | the "key" plate that carries the drawing |
| 6 | Plate 2 (Fluorescent Pink `#FF48B0`) | as above, different screen angle | Multiply | `ink_2` 60–100 % | |
| 7 | Plate 3 (Yellow `#FFE800`) | as above | Multiply | `ink_3` 50–100 % | yellow is weak on its own; carries the overlaps |
| 8 | Ink texture | speckle, density mottling, drum run-out gradient | inside each plate (modulates coverage) | — | uneven ink makes it feel printed |
| 9 | Scan overlay (optional) | real scanned blank Riso sheet (fibre, dust, ink pooling) | Multiply | 30–70 % | Oka-style physical texture; loop 4–8 scanned frames at `print_fps` |
| 10 | Output | at wall-native resolution, no scaling after layer 7 | — | — | no bloom, no sharpen after the print |

Layers 3–8 are one effect (one ISF / post-process material) in the custom paths. In the native Resolume path each plate becomes its own group (see Build recipes).

## Techniques

**1. Print clock (stepped time and per-print randomness)**
1. `frame_id = floor(time * print_fps)`, with `print_fps` 6–15, default 10. At 120 BPM, 8 fps equals four prints per beat, which locks nicely.
2. When `frame_id` changes, copy the current source frame into a hold buffer; between changes, keep showing the hold buffer. This gives the stop-motion cadence.
3. Seed every random value of the frame from `frame_id`: `r = hash(frame_id * 7.13 + plate_index * 1.7)`. All plates jump at once, but each by a different amount, just as each scanned sheet lands differently on the scanner bed.
4. With `print_fps = 0` (off) the source runs smoothly and jitter falls back to a slow noise drift.

**2. Colour separation into ink densities**
Two modes, switchable:
1. **Tonal (posterise) mode**, the poster look: `Y = luma(rgb)` (Rec.709 weights). Plate 1 (dark ink) density `d1 = smoothstep(t1 + s, t1 - s, Y)` covers the shadows; plate 2 `d2 = 1 - smoothstep(t2 - s, t2 + s, Y)` covers shadows and mid-tones (so it overprints plate 1 in the darks); plate 3 `d3 = 1 - smoothstep(t3 - s, t3 + s, Y)` covers everything below the highlights. Defaults `t1 = 0.30, t2 = 0.55, t3 = 0.80`, softness `s = 0.06`. The stacking means deep shadows get all three inks (darkest overprint), highlights get bare paper.
2. **Channel mode**, which keeps the source's colours: treat the inks as CMY-like. `cmy = 1 - rgb`. A 3×3 "plate matrix" maps cmy to the three densities. With Blue/Pink/Yellow inks, start with blue ≈ cyan, pink ≈ magenta, yellow ≈ yellow, i.e. identity. If plate 1 is Black instead of Blue, use under-colour removal: `k = min(c, m, y)`, black plate `d1 = k`, colour plates `d_i = cmy_i - ucr * k` with `ucr` 0.5–1.0.
3. Both modes then apply a per-plate curve: `d = clamp((d - lift) * gain, 0, 1)` with `gain` 1–2. Riso prints look best with plenty of solid ink and bare paper, not mushy 50 % everywhere.

**3. Screen: halftone dots or grain**
1. **Halftone:** per plate, rotate UV into a dot grid with angle `a_i` and cell size `cell` (8–24 px at 1080p). Angles 15°, 75°, 45° for three plates follow the classic CMYK screen angles that keep moiré down ([Stefan Gustavson, WebGL halftone tutorial](https://itn-web.it.liu.se/~stegu76/webglshadertutorial/shadertutorial.html); [Maxime Heckel, Shades of Halftone](https://blog.maximeheckel.com/posts/shades-of-halftone/)).
2. In each cell, sample the density at the cell centre (so the dot is one colour), radius `r = cell * 0.5 * 1.414 * sqrt(d)` so that dot *area* follows density (Heckel's sqrt coverage rule). Coverage = `1 - smoothstep(r - aa, r + aa, dist)`, `aa` = one pixel (use `fwidth`), as in Gustavson's antialiased dots.
3. Rough the dot edge with value noise (amplitude 0.1–0.3 of the radius, frequency about 3× the cell rate) so dots look stencilled, not vector-perfect (Gustavson adds noise to the edges in the same way).
4. **Grain (stochastic) mode**, Riso's "grain touch": coverage = `step(n, d)` where `n` is blue noise or a per-pixel hash in 0..1, scaled up by `grain_size` (1–4 px). It gives the sandy gradient seen in Schimautz's work. Re-seed `n` per print frame for a boiling texture, or keep it fixed for a calm "same master" look.
5. `screen_mix` (0..1) blends halftone and grain coverage so you can fade between them.

**4. Misregistration (the soul of the look)**
Per plate `i`, the UV used for sampling both the density and the screen is moved by:
1. **Static offset** `o_i`: a fixed direction per plate (e.g. plate 1 at 0°, plate 2 at 130°, plate 3 at 250°) times `offset` (0–24 px at 1080p, default 4 px).
2. **Rotation** `rot_i`: ±0.0–0.8° around a point slightly off centre, `pivot = (0.5, 0.5) + 0.2 * (r_i - 0.5)`. Small rotation makes the misregistration change across the frame (tight at the pivot, wider at the edges), so it is not a uniform shift.
3. **Per-print jitter** `j_i = (hash2(frame_id, i) - 0.5) * jitter`, `jitter` 0–6 px. New every print frame.
4. **Paper stretch / regional wobble** (optional): add `warp * fbm(uv * 1.5 + i * 3.1)` with `warp` 0–3 px, so one corner sits tight and another drifts, like paper that grew in humidity.
5. **Kick kick-out**: `offset_total = offset + kick_env * kick_amount` (kick_amount 8–40 px), direction per plate stays fixed, so the plates fly apart radially and settle back.

**5. Ink texture and overprint**
1. Coverage `cov_i` from technique 3 is multiplied by an ink-density field: `ink_i * (1 - mottle * fbm(uv * 4 + i))`, `mottle` 0–0.35, plus a sparse speckle (random pixels at 0.5–2 % that drop to zero ink, the small white pinholes of real prints).
2. **Drum run-out**: multiply by `1 - runout * smoothstep(0, 1, uv.x_along_feed)`, a ramp along the feed direction (top to bottom), `runout` 0–0.8. Ink fades towards the tail of the sheet like a drying drum; slam back to 0 on the drop.
3. **Overprint:** `colour = paper`, then for each plate `colour *= mix(1, ink_colour_i, cov_i)`. Multiply is the standard screen stand-in for translucent ink (the Riso Club example does the same, research.md).
4. **Trapping/ink spread** (optional): blur coverage by 0.5–1 px before multiplying, so edges are soft and slightly swollen.

## Shaders

Render order: P0 (hold) → P1 (separation) → P2 (plates + overprint). All at output resolution except P1, which can run at half resolution when using halftone (the dots sample cell centres anyway).

**P0 — print-clock hold**
- Purpose: stepped stop-motion cadence.
- Inputs: source texture; its own previous output (ping-pong or a persistent buffer); uniforms `print_fps`, `time`.
- Per pixel: if `floor(time * print_fps)` differs from last frame's value (passed as a uniform or bool `new_print`), output the source; otherwise output the previous buffer.
- Output: RGBA8 at source resolution. Bypass when `print_fps = 0`.

**P1 — separation**
- Purpose: three ink densities in one texture.
- Inputs: P0 output; uniforms `sep_mode` (0 tonal, 1 channel), `t1, t2, t3, s`, plate matrix (3×3), `gain`, `lift`.
- Per pixel: luma or cmy (technique 2), per-plate curve, write `d1, d2, d3` into R, G, B.
- Output: RGBA8 (or RGBA16F) density map. Important: P2 samples this at *displaced* UVs per plate, which is why separation is a separate pass.

**P2 — plates and overprint**
- Purpose: the whole visible look.
- Inputs: P1 density map; optional scan texture; uniforms `frame_id`, `offset`, `rot_max`, `jitter`, `warp`, `kick_env`, `kick_amount`, `cell`, `angles[3]`, `screen_mix`, `grain_size`, `mottle`, `runout`, `ink_colour[3]`, `ink[3]` (opacity), `plates` (1–3), `paper_colour`.
- Per pixel:
  1. Start with `col = paper_colour`, multiplied by paper fibre noise (±3 % value) or the scan texture.
  2. For each active plate `i`: build `uv_i` = rotate around pivot by `rot_i`, then add `o_i + j_i + warp_i + kick term` (technique 4).
  3. Sample `d_i` from P1 at `uv_i` (at the cell centre for halftone, at the pixel for grain).
  4. Coverage: halftone dot and grain threshold, mixed by `screen_mix`; noise-rough edges.
  5. Modulate by mottle, speckle, run-out and `ink[i]`.
  6. `col *= mix(1, ink_colour_i, cov_i)`.
  7. Optional: multiply the scan overlay.
- Output: RGBA8 at wall resolution, alpha 1. This is the last pass.

## Color scheme

Ink hexes are the standard Riso ink colour list as collected in the risoinks R package (derived from Stencil's colour list) ([risoinks, color.R](https://rdrr.io/github/anfederico/risoinks/src/R/color.R)). On screen they are approximations; fluorescent inks glow in print in a way no RGB value can show.

| role | name | hex | notes |
|------|------|-----|-------|
| base | paper off-white | `#F4EFE4` | default stock, ~95 % value, warm |
| base alt | newsprint | `#E8E2D0` | greyer, lower contrast |
| base alt | kraft | `#C9A982` | brown; inks get darker and muddier, great for a break |
| plate 1 (key) | Blue | `#0078BF` | carries the drawing |
| plate 2 (accent) | Fluorescent Pink | `#FF48B0` | the signature Riso colour |
| plate 3 (highlight) | Yellow | `#FFE800` | lifts overlaps into orange and green |
| alt key | Black | `#000000` | for a darker, comic look |
| alt | Teal | `#00838A` | Schimautz-style set with orange |
| alt | Orange | `#FF6C2F` | |
| alt | Bright Red | `#F15060` | |
| alt | Medium Blue | `#3255A4` | softer than Blue |
| alt | Green | `#00A95C` | |
| alt | Purple | `#765BA7` | |
| alt | Sunflower | `#FFB511` | warmer yellow, reads better on kraft |

Overprint results (multiply, before paper): Pink × Blue ≈ `#002284` (deep blue-violet), Pink × Yellow ≈ `#FF4100` (red-orange), Blue × Yellow ≈ `#006D00` (green), all three ≈ near-black navy. These are your "free" extra colours.

**Ink sets** (switch per phrase, never fade between them, swap on a bar line):
- A "Classic": Blue / Fluorescent Pink / Yellow on off-white.
- B "Schimautz": Teal / Orange / Yellow on off-white (research.md names yellow, orange, bright red, teal and black).
- C "Comic": Black / Bright Red / Yellow on newsprint.
- D "Two-colour": Fluorescent Pink / Medium Blue, plate 3 off, on kraft for breaks.

- Mapping: no gradient ramp. Colour is only paper × ink coverage. Hue rotation is not used, because it breaks the "real ink" look; change inks by swapping sets.
- LED walls: paper is the brightest value on screen. At `#F4EFE4` a full-paper frame is bright but not white. If it is too bright on the wall, darken paper to `#E0DACD` rather than lowering ink opacity. Black level comes only from overprint; with no black plate the darkest value is ~`#0A1030`, which looks good and keeps LED blacks from crushing detail. Fluorescent pink is very saturated on LEDs; reduce `ink_2` to 85 % if the wall clips it.

## Blending and compositing

- **Plate blending: Multiply, always.** It is subtractive, order-independent (so plate order does not matter), and never brightens, which is how translucent ink behaves. Avoid Screen/Add between plates; it turns the print into a light show.
- **Paper:** Normal, at the bottom. A scan overlay goes on top in Multiply at 30–70 %.
- **Position in the chain:** as a final look on a group or the composition, so unrelated clips share one print. Or per layer, so only some sources get printed and others stay clean (cut between print and clean on phrases, research.md).
- **Feedback:** if any, put it before the effect. Feeding the print back through itself blurs the dots into mud.
- **Post after the print:** none, or at most a 0.3–0.5 px blur for ink spread. No bloom (inks don't glow on paper), no chromatic aberration (the misregistration is your aberration), no extra digital grain (the screen is the grain). Sharpen destroys the halftone edges.

## Motion and timing

- **Print clock:** 8–12 fps stepped; on a 120 BPM track, `print_fps = BPM / 15` gives 8 fps (4 prints per beat). During builds, double it to 16 fps.
- **Per print frame:** jitter re-rolls (2–4 px), grain re-seeds if boiling is on.
- **Per beat (kick):** `kick_env` attack 0–10 ms, release 150–250 ms, exponential ease-out, plates fly out by `kick_amount` 12–24 px and settle.
- **Per bar:** halftone `angles` rotate by +7.5° per bar (stepped) or the static offset direction rotates 90° per bar, so the fringe colours move around the shapes.
- **Per phrase (8/16 bars):** ink set swap (A → B → C), or paper stock swap.
- **Build-up:** over 8–16 bars ramp `runout` from 0 to 0.6 (ink dries up), `jitter` from 2 to 6 px, `cell` from 12 to 24 px (coarser, louder dots).
- **Drop:** on beat 1, `runout` → 0, `offset` jumps to 20 px for one beat then eases to 4 px over 1 bar, `screen_mix` → halftone, plates 1→3.
- **Break:** `plates` = 2, ink set D on kraft, `print_fps` 6, `jitter` 1 px, grain mode, slow `warp` 2 px. Calm and papery.

## Parameters

| name | range | default | what it does on stage | suggested MIDI / BPM mapping |
|------|-------|---------|-----------------------|------------------------------|
| `offset` (misregister) | 0–40 px (at 1080p) | 4 | how far the plates sit apart; small = subtle fringes, large = three ghost images | KeyStep 37 knob 1; kick envelope adds `kick_amount` |
| `jitter` | 0–8 px | 2.5 | per-print random shake; the "hand-printed" breathing | knob 2; hi-hat energy adds +1–2 px |
| `dot_size` (`cell`) | 4–32 px | 12 | fine print versus loud pop-art dots | knob 3; bass level ±4 px, slow smoothing |
| `grain` (`screen_mix`) | 0 halftone – 1 grain | 0.3 | clean dots versus sandy Riso grain | knob 4 or fader |
| `ink_set` | 0–3 (A–D) | 0 | swaps all three inks and the paper at once | BeatStep Pro pads 1–4, quantised to the bar |
| `print_fps` | 0 (off), 6–16 fps | 10 | stop-motion cadence; 0 = smooth video | pad toggle off/on; or derived from BPM (BPM/15) |

Set-and-forget: `rot_max` 0.3°, `warp` 1 px, `mottle` 0.15, speckle 1 %, separation tonal with `t1/t2/t3` 0.30/0.55/0.80. Optional extra knobs: `runout` (0–0.8), `plates` (1–3, pad).

## Build recipes

### Resolume Arena

**Path A — native, no custom code (tonal separation, three plate groups).** Every plate is a group with two layers, built so that "ink where the threshold is black, paper where it is white":
1. Composition bottom layer: solid colour clip `#F4EFE4` (paper), Normal, 100 %.
2. Group "Plate Blue", blend mode **Multiply**, opacity = `ink_1`. Inside it:
   - Layer a: solid colour clip `#0078BF`.
   - Layer b: the source clip (or a Spout/NDI source) with Resolume's threshold-type effect (Threshold, level 0.30) so it gives pure black/white, then **Transform** with Position X/Y offset of e.g. +4 / 0 px and Rotation +0.2°. Blend mode **Screen** over layer a. Screen of ink and a black/white mask gives ink where the mask is black and white where it is white, which is exactly the plate.
3. Group "Plate Pink", Multiply, same structure, threshold 0.55, ink `#FF48B0`, Transform offset −3 / +3 px, Rotation −0.15°.
4. Group "Plate Yellow", Multiply, threshold 0.80, ink `#FFE800`, Transform offset 0 / −4 px.
5. Texture: add the built-in **PolkaDot** effect or a noise/grain effect to layer b before the threshold, so the threshold breaks into dots/grain. Check which dot and noise effects your Resolume version ships; the 7.x effect reference lists PolkaDot, Transform and Hue Rotate among others ([Resolume effects reference 7.7](https://resolume.com/support/en/7.7/effects)).
6. Misregistration on the beat: on each plate's Transform, animate Position with the BPM-synced parameter animation (e.g. Random, 1/4 beat, range ±3 px) for jitter, and map the kick (Resolume audio FFT parameter link) to Position X with a short release for the kick-out.
7. Print clock: put a frame-rate / strobe-hold style effect on the source if available, or set the source clip's speed and use a BPM-synced "freeze" sample-and-hold; otherwise skip it in the native path.
8. Limits: dots are not screen-angled per plate and grain is digital noise, so it looks like "posterised overprint" more than real Riso. Good enough for a quick look, and it runs on any machine.

**Path B — ISF effect (recommended).** One ISF filter "RisoPrint" containing P0–P2 as ISF passes (`PASSES` with `PERSISTENT: true` for the hold buffer) ([ISF multipass reference](https://docs.isf.video/ref_multipass.html), [Resolume ISF support](https://resolume.com/support/en/isf)). Expose inputs: `offset`, `jitter`, `dot_size`, `grain`, `print_fps`, `ink_set` (long, 0–3), `plates` (long, 1–3), `runout`, `kick` (float 0–1, fed by Resolume audio FFT or OSC), and three colour inputs for custom inks plus a `paper` colour. Since ISF has `TIME` but no beat clock, either feed `print_fps` from a BPM-synced parameter or drive a `print_trigger` event input from the clip's BPM sync. Place it as the last effect on the composition or on a group.

### Resolume Wire

Patch "RisoPrint" as an effect:
- **Input**: Texture In (source); Float inputs `offset`, `jitter`, `dot_size`, `grain`, `runout`, `kick`; Integer `ink_set`, `plates`; Float `print_fps`.
- **Print clock**: Time → multiply by `print_fps` → Floor → Change detector (compare with a one-frame delay) → trigger. Trigger → Sample and Hold of the texture (a feedback/texture-delay node gated by the trigger) → `held`.
- **Random per print**: `frame_id` → per-plate Random/Noise node seeded by `frame_id + i` → Vec2 → scale by `jitter`.
- **Separation**: GLSL shader node (or Luma + three Smoothstep/Threshold nodes) → three density textures, or one RGB density texture.
- **Plates**: three branches; each = 2D Transform node (translate = static offset + jitter + kick × direction, rotate = `rot_i`) → GLSL halftone/grain node with its angle → Colorize to ink hex.
- **Overprint**: Blend nodes in Multiply: paper colour → × plate 1 → × plate 2 → × plate 3 → optional × scan texture → Texture Out.
- **Ink sets**: an Array/Switch node indexed by `ink_set` selects three colours plus paper.
- **Audio**: Wire's FFT node (low band) → envelope follower (attack 5 ms, release 200 ms) → `kick` ([Resolume Wire FFT](https://resolume.com/support/en/wire-fft)). Expose all inputs on the patch so they show in Arena's effect panel and can be MIDI mapped.

### Unreal Engine 5.7

**Approach:** a post-process material (Material Domain: Post Process, Blendable Location: After Tonemapping) on an unbound Post Process Volume, so the whole render gets printed. Do the hold and separation in a Blueprint-driven render target pipeline if you want the print clock.
- **Print clock:** Blueprint timer at `1 / print_fps` → `Draw Material to Render Target` copies SceneTexture:PostProcessInput0 (via a small copy material, or capture with a Scene Capture 2D) into `RT_Hold` (RGBA8, output resolution). It also increments an integer `FrameID` that goes into a **Material Parameter Collection** `MPC_Riso`.
- **Material graph (M_RisoPrint):**
  1. `Texture Sample (RT_Hold)` or `SceneTexture:PostProcessInput0` when the clock is off.
  2. Separation: `Dot` with (0.2126, 0.7152, 0.0722) → three `SmoothStep` nodes with `t1/t2/t3` (or `1 - RGB` and a 3×3 matrix with three `Dot` nodes for channel mode).
  3. Per plate: `ScreenPosition` (ViewportUV) → `CustomRotator` (angle `rot_i`, centre pivot) → `Add` (offset vec2 from MPC: static + jitter + kick × direction, in pixels × `ViewSize` inverse). Jitter from a hash of `FrameID + i` (a small Custom node or `Noise` node with `FrameID` as a coordinate).
  4. Halftone: rotate UV by the screen angle → multiply by `ViewSize / cell` → `Frac` → `Distance` to (0.5, 0.5) → compare with `sqrt(d) * 0.707` via `SmoothStep` (width from `DDX/DDY` length). Grain: `Noise` (Fast Gradient or Value, scale from `grain_size`) → `Step` against `d`. `Lerp` between them with `screen_mix`.
  5. Overprint: `Lerp(1, ink_i, cov_i)` three times, `Multiply` chain onto `paper`, then × optional `Texture Sample (T_Scan)`.
  6. Output to Emissive Color.
- **Remote Control:** expose the MPC scalars `Offset`, `Jitter`, `DotSize`, `Grain`, `Runout`, `Kick`, `PrintFPS`, `Plates` and an integer `InkSet` (the Blueprint turns it into four vector params) in a Remote Control Preset; bind to OSC and MIDI (KeyStep knobs CC 1–4, BeatStep Pro pads for `InkSet`).
- **Output:** Spout (e.g. the Off World Live plugin) to Resolume on the same machine, NDI for other machines. Decide where the print happens. If UE prints, send it over Spout only. NDI's chroma subsampling smears fine halftone dots and fringes, so over NDI send the clean render and print on the receiving side.

### Godot

- **Scene:** source (`VideoStreamPlayer`, a camera/Spout `TextureRect`, or a 3D scene in a `SubViewport`) → `SubViewport` "Hold" with `render_target_update_mode = UPDATE_ONCE`, re-armed from a script timer at `1 / print_fps` (the print clock, cheap and exact) ([Godot SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)).
- **Print:** a full-screen `ColorRect` on a top `CanvasLayer` with a `canvas_item` shader that does P1 and P2 in one pass (separation is cheap enough to recompute per plate sample) ([Godot canvas_item shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/canvas_item_shader.html)).
- **Uniforms:** `sampler2D held`, `float offset_px`, `float jitter_px`, `float cell_px`, `float screen_mix`, `float runout`, `float kick`, `int frame_id`, `int plates`, `vec3 ink[3]` (`source_color` hint), `vec3 paper`, `vec3 thresholds`, `vec3 angles_deg`, optional `sampler2D scan`.
- **Script:** increments `frame_id` per print, handles ink-set switching (an Array of Color presets), and feeds `kick` from an `AudioEffectSpectrumAnalyzer` low band.

## Audio reactivity

| band | drives | smoothing | threshold / notes |
|------|--------|-----------|-------------------|
| Kick, 40–120 Hz | `kick` → plate kick-out (`kick_amount` 12–24 px) | attack 0–10 ms, release 150–250 ms | onset when the band rises > 6 dB above its 1 s average; ignore new onsets for 100 ms |
| Bass, 60–250 Hz (sustained level) | `dot_size` ± 4 px, or `ink` opacity 70–100 % | attack 80 ms, release 400 ms | slow, so dots swell with the bassline and don't flicker |
| Mids, 500 Hz–2 kHz | plate 2 offset only (variation "plates by audio band") | attack 20 ms, release 200 ms | snares/vocals shake the pink plate on its own |
| Hi-hats, 6–12 kHz | `jitter` +0–2 px, and re-seed grain | attack 5 ms, release 80 ms | only matters at print-frame boundaries, so it naturally quantises to `print_fps` |
| Overall loudness (RMS, 8-bar average) | `runout` inverse: quiet sections run dry | attack 2 s, release 4 s | the drop refills the drum |

For "plates by audio band" (research.md), map kick → plate 1, mids → plate 2, hats → plate 3 so each ink dances to its own instrument.

## Performance tips

- **Cost:** one full-screen pass with 3 density samples, 3 noise lookups and a few smoothsteps per pixel; under 0.5 ms at 1080p on a mid-range GPU. The hold pass is a copy. The native Resolume path costs three groups × (threshold + transform) plus three solid layers; still light, but it triples source decoding if you use three copies of a video clip. Use one source in a layer and route it into the groups (Resolume layer routing / a Spout loop) instead.
- **Resolution:** set `offset`, `jitter` and `cell` in pixels relative to 1080p and scale by `output_height / 1080`, so the look is the same on a 4K wall and a preview. On LED walls with a coarse pixel pitch, keep `cell` ≥ 3 LED pixels or the dots alias into moiré with the LED grid; switch to grain mode on very low-resolution walls.
- **Odd aspect ratios:** compute halftone cells in pixels (not normalised UV), otherwise dots stretch into ellipses on 32:9 or tall columns. Put the rotation pivot in pixel space for the same reason.
- **Moiré:** keep the three screen angles 30° apart (15/45/75). With two plates use 15° and 75°. Avoid 0° and 90° near the LED grid's orientation.
- **Scaling after the effect** (Resolume output scaling, LED processor scaling) blurs dots and causes moiré; render at the wall's native resolution.
- **Codecs/NDI:** halftone dots and 1 px fringes are worst-case for compression; print after transport.
- **Flicker safety:** at `print_fps` ≥ 12 with full-frame ink-set swaps on every beat, the wall can strobe hard. Keep set swaps to bar lines.

## Further references

- [Stefan Gustavson, WebGL halftone shader tutorial](https://itn-web.it.liu.se/~stegu76/webglshadertutorial/shadertutorial.html): step-by-step antialiased halftone dots, rotated screens, noise-roughened edges and CMYK screen angles.
- [Maxime Heckel, Shades of Halftone (2026)](https://blog.maximeheckel.com/posts/shades-of-halftone/): grid, rotation, sqrt coverage for dot area, CMYK layers with subtractive blending.
- [risoinks colour list (R package source)](https://rdrr.io/github/anfederico/risoinks/src/R/color.R): hex approximations of the standard Riso ink colours used above.
- [ISF multipass and persistent buffers](https://docs.isf.video/ref_multipass.html): how to build the hold buffer and the passes as one Resolume ISF effect.
