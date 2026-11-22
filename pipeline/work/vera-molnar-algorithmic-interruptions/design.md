# Vera Molnar Algorithmic Interruptions — design

## Look

A dense, even field of short straight lines fills the screen, one line per cell of a square grid (25 x 25 by default, as in *Interruptions*, 1968 ([Toledo Museum](https://infiniteimages.toledomuseum.org/?p=811))). In a calm state they all point the same way and sit end to end, like hatching on graph paper. Then the rule loosens: every line tilts a little, a few tilt a lot, and soft-edged holes open where whole patches of lines are missing, as if the paper had worn away. Lines next to a hole lean further than lines in the dense areas, which re-codes of the piece note in the original ([Re-Code, Medium](https://medium.com/@xuetingzhang313/re-code-vera-moln%C3%A1rs-interruptions-b6dc9765eae5)). On a kick the whole grid snaps into a new broken state and then heals back to order over a few beats. A second rule set swaps the lines for square outlines with jittered corners, after *Interruptions à recouvrements* and *Structure de quadrilatères* ([CMU IDeATe](https://courses.ideate.cmu.edu/15-104/f2018/2018/10/05/vera-molnars-generative-art)). For LED walls the plotter look is inverted to pale ink on near-black, with the original black-on-paper as a daylight preset. The mood is dry and exact: long stretches of order broken by short ruptures, Molnar's "disturbed equilibrium" ([Morgan](https://www.themorgan.org/morganmobile/order-and-disorder/molnar)).

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Ground | flat colour, #0B0B0C (stage) or #F1ECE2 (paper preset) | Normal | 100 % | the "paper"; never pure black on LED |
| 2 | Paper grain | static value noise, 1 px scale, ±2 % luminance | Overlay | 30 % | stops large flat areas from looking like a dead panel; off on small walls |
| 3 | Ghost grid | the same generator with disorder = 0 and interruptions = 0 | Add (stage) / Multiply (paper) | 8–12 % | a faint memory of perfect order behind the broken grid; makes the deviations readable |
| 4 | Main grid | per-cell line or square SDF generator (pass P1) | Normal over ground (generator outputs alpha) | 100 % | the piece itself |
| 5 | Accent pen | same generator, filtered to cells whose hash > 1 − accentShare | Add (stage) / Multiply (paper) | 100 % | 0–5 % of cells drawn in a second "pen" colour; off by default |
| 6 | Footage window (optional) | live clip or camera, masked by the interruption mask (holes) | Normal, masked | 0–100 % | the image shows through where the grid breaks |
| 7 | Post | 1-pixel sharpen off, no bloom by default; optional soft glow on the accent pen only | see compositing | — | plotter look needs crisp, flat lines |

## Techniques

**1. Cell grid with constant square cells on any aspect ratio**
1. Work in pixels: `pitch = shortSide / density`, where density is the cell count across the short side (8 to 64, default 25).
2. `g = (fragCoord - centre) / pitch`; cell id `c = floor(g)`, local position `f = fract(g) - 0.5`.
3. The grid is centred and extends past the long side, so a 32:9 wall gets square cells and more columns, not stretched lines.
4. Optional margin: cells whose centre lies outside a centred square of `density x density` cells are blank, which gives the framed, sheet-of-paper composition of the 1968 drawing.

**2. Per-cell rule evaluation (the "machine imaginaire")**
1. Per cell, compute four hashes from `c` and the seed: `h0..h3 = hash(c + seed * k)`. Use an integer hash or the fract-sine hash from [The Book of Shaders, ch. 10](https://thebookofshaders.com/10/); on GPUs with poor sin precision at large inputs, prefer an integer hash.
2. Dominant orientation `theta0` (0 = vertical, the default; π/2 = horizontal; the re-code notes both appear in the series).
3. Local disorder `d = disorder * field(c, t)` (technique 4) plus the gap boost from technique 3.
4. Angle `theta = theta0 + (h0 - 0.5) * 2 * d * maxAngle`, maxAngle = 90°. Use a biased distribution so most cells stay near the rule: `(h0 - 0.5)` raised to an odd power via `sign(x) * abs(x)^1.6`.
5. Displacement (rule "displace"): `offset = (h1, h2) - 0.5` times `d * 0.4 * pitch`.
6. Length `L = lengthRatio * pitch` (default 1.0, so vertical neighbours touch at their endpoints).

**3. Interruptions (clustered erasure)**
1. Evaluate a low-frequency value-noise fbm at the cell centre: `n = fbm(c * gapScale + seedOffset)`, gapScale 0.08–0.2 cells⁻¹, 3 octaves. Clustered noise rather than per-cell coin flips, as in the Perlin-noise re-code ([Medium](https://medium.com/@xuetingzhang313/re-code-vera-moln%C3%A1rs-interruptions-b6dc9765eae5)).
2. Threshold: a cell is erased when `n > 1 - interrupt`. interrupt 0 = no holes, 0.15 = the 1968 density, 0.5 = ruins.
3. Add a per-cell hash test so the hole edge is ragged: erased if `n + (h3 - 0.5) * 0.12 > 1 - interrupt`. This gives the "abraded paper" edge rather than a smooth cloud.
4. Gap boost: `d += gapBoost * smoothstep(1 - interrupt - 0.25, 1 - interrupt, n)`, gapBoost 0.6. Lines near a hole lean more.
5. Export the erasure value as a separate mask channel (for layer 6 and for keying).

**4. Disorder field: order that breaks and heals**
`field(c, t)` decides where and when the rule is broken. Three modes, crossfadable:
1. **Global**: field = 1 everywhere.
2. **Sweep**: a band moving across the grid, `field = exp(-((dot(c, dir) - pos) / width)^2)`, with pos running from one edge to the other in 1 bar, width 3–6 cells. Chaos travels through and order is restored behind it.
3. **Heal**: on each trigger, `env = exp(-(t - tKick) / heal)`; per cell a delay `h1 * 0.5 * heal` makes cells heal one by one, not all at once: `field = env evaluated at (t - tKick - delay)`, clamped.
4. **Footage**: field = luminance of a video input sampled at the cell centre (one texel per cell, from a mip level or a downscaled copy), so the image appears as a pattern of tilted lines.
5. Final per-cell disorder is `base + kickAmount * field`, so a small base disorder (0.05–0.15) always keeps the grid alive.

**5. Seed crossfade instead of hard cuts**
1. Keep two seeds, A and B. A trigger (kick, pad or bar) sets A = B, B = new seed, `s = 0`.
2. `s` ramps 0 to 1 over `snap` time (0 = hard cut, default 60 ms, up to 1 beat), smoothstep ease.
3. Angles are mixed by shortest angular distance; displacement and length are lerped; erasure uses `mix(nA, nB, s)` so holes slide and morph instead of popping.
4. With snap = 0 the grid cuts like a new plotter sheet, which is the strongest drop effect.

**6. Anti-aliased line and square rendering**
1. Because a line may be longer than the cell or displaced, test the own cell and its 8 neighbours (3 x 3) per pixel.
2. For each neighbour compute its segment from its centre, angle and length, and the distance from the pixel to the segment (standard segment SDF, as in [Inigo Quilez, 2D distance functions](https://iquilezles.org/articles/distfunctions2d/)).
3. Coverage: `ink = 1 - smoothstep(w/2 - 0.75, w/2 + 0.75, dpx)`, w = line weight in pixels (default 2 px at 1080p, scale with `outputHeight / 1080`). Take the max over neighbours.
4. Rule "overlap": draw a square outline per cell instead, side `0.8 * pitch * (1 + (h2 - 0.5) * d)`, rotated by theta, each corner moved by `(hash - 0.5) * d * 0.3 * pitch` for the quadrilateral look; distance = min distance to the four edges.
5. Rule blend 0..1 crossfades line ink and square ink; at 0.5 both are visible and overlap, which reads like *Interruptions à recouvrements*.
6. Clamp constraints: lengthRatio ≤ 1.6 and displacement ≤ 0.4 pitch keep every line inside the 3 x 3 search, so no line gets clipped at cell borders.

## Shaders

Render order per frame: P0 (optional) footage downscale, P1 grid generator, P2 composite and post. No feedback buffers are needed; all state (seeds, kick times) lives in uniforms.

**P0 — Footage disorder map (optional)**
- Purpose: one luminance value per cell for the footage mode.
- Inputs: video texture.
- Per pixel: sample the video with linear filtering, `lum = dot(rgb, (0.2126, 0.7152, 0.0722))`, optional gamma 0.7 to lift shadows.
- Output: R8 texture at the grid resolution (e.g. 25 x 14 for 16:9 at density 25). Rebuild only on density change. Cost: negligible.

**P1 — Grid generator**
- Purpose: the whole image of lines or squares.
- Inputs: uniforms `density, disorder, kickAmount, interrupt, gapScale, gapBoost, lengthRatio, weight, rule, theta0, seedA, seedB, snapMix, fieldMode, sweepPos, sweepDir, sweepWidth, tKick, heal, time, margin`; optional P0 texture.
- Per pixel:
  1. Compute g, c, f (technique 1).
  2. Loop over the 3 x 3 neighbours `cn = c + (i, j)`.
  3. For each: hashes for seed A and B, disorder field, erasure test for A and B mixed by snapMix (technique 5). Skip erased cells (still record the own cell's erasure for the mask).
  4. Build segment or square, compute pixel distance, coverage (technique 6), keep max ink and the id of the winning cell.
  5. Accent flag: winning cell hash h3 > 1 − accentShare.
- Output: RGBA8 at output resolution. R = ink, G = accent ink, B = erasure mask of the own cell (soft, 0..1, smoothstep over 0.1 around the threshold), A = ink. Cost: 9 segment tests per pixel plus 9 small fbm calls; fine at 4K on a mid-range GPU. Optimisation: evaluate fbm only once per cell by moving it into P0's grid texture when footage mode is off.

**P2 — Composite and post**
- Purpose: colour, ghost grid, grain, footage window.
- Inputs: P1 output, a second P1 output for the ghost grid (or the same pass with disorder forced to 0 in a second evaluation inside P1, writing to another channel), palette uniforms, footage texture.
- Per pixel:
  1. `col = ground`.
  2. Footage window: `col = mix(col, footage, mask * windowOpacity)`.
  3. Ghost: stage mode `col += ghostInk * inkColour * 0.1`; paper mode multiply.
  4. Main ink: `col = mix(col, inkColour, R)`; accent `col = mix(col, accentColour, G)`.
  5. Grain: `col += (noise(fragCoord) - 0.5) * 0.02`.
- Output: final RGBA8, output resolution.

## Color scheme

| role | stage (LED) | paper preset | notes |
|------|-------------|--------------|-------|
| ground / base | #0B0B0C | #F1ECE2 | off-black keeps the wall from looking switched off; warm off-white plotter paper |
| ink / main | #ECE6D8 | #141414 | warm white, not #FFFFFF, to avoid harsh glare at full wall brightness |
| ghost | ink at 8–12 % | ink at 10 % via Multiply | memory of the rule |
| accent pen 1 | #E0442E (red) | #C8321E | pen-plotter red |
| accent pen 2 | #2F5BD3 (blue) | #1F44A8 | |
| accent pen 3 | #2E9E5B (green) | #1E7A44 | |
| salmon preset | ground #E59A82, ink #FFF6EE | — | after the white-on-salmon *Structure de quadrilatères* ([CMU IDeATe](https://courses.ideate.cmu.edu/15-104/f2018/2018/10/05/vera-molnars-generative-art)) |

Mapping: no gradients. Colour is a fixed pair per scene; the only change is which pen the accent cells use. Advance the accent pen every 4 bars or on a pad. On the drop, invert the stage pair for one beat (ground #ECE6D8, ink #0B0B0C), then return; this is the only flash in the piece. Contrast: keep ground at 3–5 % and ink at 85–92 % of wall maximum; lines are thin, so brightness can stay high without blinding. Do not use pure saturated pens for the whole grid on LED: thin red lines at 2 px break up on many walls' subpixel layouts.

## Blending and compositing

- Main grid over ground with Normal (alpha = ink): lines must cover, not add, so overlapping lines do not get brighter. Brighter crossings would read as "glow", which is foreign to plotter ink.
- Ghost grid in Add at 10 % on stage, Multiply on paper: a quiet reference so the eye can see how far each line left its rule.
- Accent pen in Add on stage (a slight lift where it crosses white ink is acceptable) and Multiply on paper (ink over ink darkens, like a real second pen pass).
- Footage window: P1's B channel used as a luma matte for the footage layer. Soft edge of 0.1 in mask units; the holes become windows.
- Mask export: the same B channel can be sent alone over Spout/NDI as a stencil for other layers or other machines.
- Post chain, in order: none for the main build. Optional: Gaussian blur 6 px on the accent channel only, added back at 40 % (pen glow); film grain 2 %. No chromatic aberration, no bloom on the main ink, no feedback: they destroy the plotter precision that is the whole point.

## Motion and timing

- **Base state**: disorder 0.08, interrupt 0.12, nothing moves except a very slow drift of the gap noise (`seedOffset += 0.02 cells per second`), so holes creep like erosion.
- **Per beat**: nothing by default. Hi-hats can flip single cells (see audio).
- **Per kick**: trigger heal mode, kickAmount 0.6, heal 1.5 beats; snap 60 ms. The grid shatters and repairs, cell by cell.
- **Per bar**: new seed B with snap = 1 beat, so the composition changes every bar without a cut.
- **Per phrase (8 or 16 bars)**: step density through 13, 25, 37, 49 or toggle theta0 vertical/horizontal, with snap 2 beats.
- **Build-up**: ramp base disorder 0.08 to 0.5 and interrupt 0.12 to 0.35 over the last 8 bars; switch field mode to sweep with sweep speed 1 bar, then 2 beats, then 1 beat.
- **Drop**: snap = 0, new seed, rule jump to squares (rule 1.0) or 0.5 overlap, invert colours for 1 beat, then disorder falls back to 0.15 over 2 bars with an exponential ease-out. The fall back to order is the payoff.
- **Break**: disorder 0, interrupt 0, ghost off: a perfect grid. Then let interrupt rise slowly (0 to 0.25 over 16 bars) so holes appear one by one. Silence is order.
- Speed ranges: snap 0–1000 ms, heal 0.25–8 beats, sweep 0.5–4 bars per crossing.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| disorder | 0–1 | 0.08 | how far lines leave the rule (angle, displacement, square size) | BeatStep Pro knob 1; kick adds kickAmount on top |
| interrupt | 0–0.6 | 0.12 | amount of erased area; 0 = full sheet, 0.5 = ruins | BeatStep Pro knob 2; slow ramp in breaks |
| density | 8–64 cells | 25 | grid cells across the short side | KeyStep 37 keys: one octave mapped to 8, 13, 19, 25, 31, 37, 49, 64; change on phrase |
| rule | 0–1 | 0 | 0 = lines, 0.5 = overlap, 1 = jittered squares | KeyStep mod strip or knob 3 |
| heal | 0.25–8 beats | 1.5 | how long a kick's damage takes to repair | knob 4, in beats synced to Arena BPM |
| weight | 1–6 px at 1080p | 2 | line thickness; thicker for far audiences | knob 5 |

Triggers (pads, not faders): `reseed` (new seed B), `shatter` (kick envelope manually), `invert` (one-beat colour flip), `orient` (toggle vertical/horizontal). Map to BeatStep Pro drum pads.

## Build recipes

### Resolume Arena

- The generator is a custom shader, so it lives in a Wire patch (see below) that uses the ISF node, exported as a Wire source and loaded as a clip in Arena. Resolume documents ISF only for Wire ([Resolume: ISF in Wire](https://resolume.com/support/en/isf)), so treat Wire as the host.
- **Layer 1**: Solid Color source #0B0B0C (or skip it and let the Wire patch draw the ground).
- **Layer 2**: Wire source "Molnar Grid" with its exposed parameters: disorder, interrupt, density, rule, heal, weight, reseed, shatter. Blend mode Alpha, opacity 100 %.
- **Layer 3**: same Wire source, second instance, disorder 0, interrupt 0, opacity 10 %, blend Add: the ghost grid. On a big wall, use the in-patch ghost instead to save a pass.
- **Layer 4 (optional)**: footage clip with a Mask effect fed by the Wire patch's mask output (send it as a separate Wire source in mask-only mode, then use that layer as a luma mask for the footage layer).
- **Parameter animation**: on disorder, use the Audio tab, low band, for kick hits; on reseed, use BPM sync (1 bar). On heal, set the value in beats.
- **Built-in effects**: Invert RGB on the composition, triggered by a pad and set to last one beat, for the drop flash. Avoid Bloom, Blur and feedback effects on this layer.
- **Output**: Spout or NDI output of the layer or composition to send the stencil to UE or TouchDesigner.

### Resolume Wire

- **ISF node** (P1 + P2 in one ISF with two PASSES, or a single pass if no footage): inputs as float INPUTS with MIN/MAX (see [ISF Multi-Pass Reference](https://docs.isf.video/ref_multipass.html) for PASSES and TARGET).
- **Time and BPM**: use the patch's time and the BPM from Arena's transport to make `time` and `beatPhase`. Convert heal and snap from beats to seconds with `seconds = beats * 60 / BPM`.
- **Seed logic**: an event input "reseed" drives a counter; the counter value is seed B, the previous value is seed A. A ramp from 0 to 1 over snap seconds restarts on each event and goes to snapMix.
- **Kick logic**: event input "shatter" (or an audio-threshold event) stores the current time as tKick.
- **Sweep**: a 1-bar sawtooth from the BPM phase, scaled to the grid width in cells, goes to sweepPos.
- **Exposed parameters** (on the patch's outer interface): disorder, interrupt, density (int), rule, heal, weight, fieldMode (int 0–3), paletteIndex (int), accentShare, reseed (event), shatter (event), invert (event).
- **Outputs**: main video out; second out with only the mask channel in grey.

### Unreal Engine 5.7

- **Material route (recommended)**: an unlit material (Material Domain User Interface or Post Process, or a Surface material on a screen-filling plane) with one **Custom** node holding the 3 x 3 neighbour loop, segment distance and hash. Feed it from **ScalarParameter** nodes (disorder, interrupt, density, rule, heal, weight, snapMix, seedA, seedB, tKick) and **VectorParameter** nodes for colours. Screen position from **ScreenPosition** or **TextureCoordinate** times the view size in pixels.
- Put all scalars in a **Material Parameter Collection** so Blueprints and Remote Control can drive them in one place.
- **Seed and kick logic** in a Blueprint actor: on a reseed event, copy seed B to seed A, write a new seed B, then drive snapMix with a Timeline (length = snap seconds). On shatter, write the current game time to tKick.
- **Render to texture**: draw the material into a Render Target with **Draw Material to Render Target** each frame, at output resolution, then use it on stage geometry or send it out.
- **Niagara alternative (3D)**: one emitter, **Spawn Burst Instantaneous** with density² particles, **Grid Location** module for the layout, sprite or mesh renderer with a thin box mesh as the line. Per-particle angle from a hash of Execution Index plus seed in a Scratch Pad module; set **Mesh Orientation** or sprite rotation from it; kill or scale to 0 for erased cells. This lets the grid tilt in depth (lines rotate around X as well as Z) for a 3D take on the same rule.
- **Remote Control**: expose the MPC scalars and the reseed/shatter Blueprint functions in a Remote Control Preset; bind OSC addresses (e.g. `/molnar/disorder`) and MIDI CCs from the BeatStep Pro via the Remote Control protocol bindings.
- **Output**: Spout (via a Spout plugin for UE) or NDI (NDI SDK for Unreal) from the render target, so Resolume gets the grid as a source or a mask.

### Godot

- **Node setup**: a ColorRect filling the viewport with a ShaderMaterial, `shader_type canvas_item`. Use `FRAGCOORD` and the viewport size for the pixel-based grid.
- **Uniforms**: `density` (int), `disorder`, `interrupt`, `gap_scale`, `gap_boost`, `length_ratio`, `weight`, `rule`, `theta0`, `seed_a`, `seed_b`, `snap_mix`, `field_mode` (int), `t_kick`, `heal`, `sweep_pos`, colour uniforms with `source_color` hints, optional `sampler2D footage`.
- **Logic** in a GDScript on the ColorRect: a Tween for snap_mix on reseed, time since kick for heal.
- **Audio**: an AudioEffectSpectrumAnalyzer on the music bus; read magnitude for 40–120 Hz each frame and fire shatter on threshold crossings.
- **Mask output**: render the same shader in a SubViewport with a `mask_only` uniform and use its ViewportTexture as a mask for footage.
- No feedback SubViewport is needed.

## Audio reactivity

| band | drives | smoothing | threshold / notes |
|------|--------|-----------|-------------------|
| Kick, 40–120 Hz | shatter trigger (sets tKick, kickAmount 0.6) | attack 0 ms, retrigger lockout 120 ms | onset when the band rises above 1.4 x its 1 s running average |
| Bass, 60–250 Hz, sustained | base disorder +0 to +0.2 | attack 150 ms, release 800 ms | sustained bass loosens the grid without kicks |
| Mids, 500 Hz–2 kHz | interrupt +0 to +0.08 | attack 300 ms, release 2 s | holes grow slowly with dense mids, never jump |
| Hats, 6–12 kHz | single-cell flips: on each onset, erase or restore 1–3 random cells for 1/16 note | attack 0, hold 1/16 note | sparse flicker; cap at 3 cells so it stays an accent |
| Overall loudness | nothing | — | keep the image from pumping with volume; Molnar is about rules, not loudness |

Without audio analysis: drive shatter from the kick track of the BeatStep Pro (gate to MIDI note) and reseed from Arena's BPM, 1 per bar.

## Performance tips

- P1 is cheap: 9 segment distance tests plus hashes per pixel. The fbm for the gap mask is the most expensive part; compute it per cell (in a small grid texture) instead of per pixel x 9 when density is fixed.
- Render at native output resolution. Do not upscale: thin lines become soft and lose the plotter look. On 4K or wider walls, set weight from output height (2 px at 1080p, 4 px at 2160p).
- Moiré: at density 64 on a low-resolution LED wall (pixel pitch 3.9 mm or more), lines a few pixels apart beat against the LED grid. Keep pitch above 12 wall pixels: `density ≤ shortSide / 12`.
- Odd aspect ratios: the grid is defined on the short side, so a 4:1 strip gets 25 rows and 100 columns. Use the margin option to frame a square sheet in the centre if the wall is very wide, or set density on the long side for vertical strips.
- Lines at near-horizontal angles alias most; the 0.75 px smoothstep already handles it. Do not drop it to save cost.
- The ghost grid as a second Arena layer doubles cost; prefer computing it in the same pass.
- Integer hashes give identical patterns on all GPUs; fract-sine hashes can differ between vendors, which matters if two machines must show the same seed.

## Further references

- [The Book of Shaders, ch. 10: Generative designs, random](https://thebookofshaders.com/10/) — per-cell random values with `floor()` and `fract()`, the 2D random from `dot()`, Truchet tiles and the 10 PRINT maze: the core of the per-cell rule.
- [Inigo Quilez, 2D distance functions](https://iquilezles.org/articles/distfunctions2d/) — the segment and box distance functions behind anti-aliased lines and squares.
- [Re-Code: Vera Molnár's Interruptions (Medium)](https://medium.com/@xuetingzhang313/re-code-vera-moln%C3%A1rs-interruptions-b6dc9765eae5) — observations on the original (same-length lines, dominant direction, larger deviation near gaps) and a Perlin-noise gap mask.
- [CMU 60-212 Interruptions gallery](https://ems.andrew.cmu.edu/2018_60212f/deliverables/due-9-7/interruptions-gallery) — 22 student p5.js re-codes of *Interruptions*, some animated; useful for comparing gap and angle strategies.
