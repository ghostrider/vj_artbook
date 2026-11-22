# Saul Bass Cut-Paper Titles — design

## Look

A flat, matte poster that moves in snaps. Everything on the screen is one of four flat colours: a warm grey or orange ground, near-black "ink", off-white "paper", and one hot accent. Thick paper bars slam in from the edges on the kick and stop dead. Type slides in on them, or appears through gaps between them. In the middle sits one bold icon made of angular cut-out pieces: an eye, a speaker cone, a key, a jagged zigzag. On a snare it breaks along straight scissor cuts and the pieces jump outward. On the drop it snaps back together. Every edge is slightly rough, like hand-cut card, and the roughness changes 8 to 12 times a second, so the whole frame "boils" like stop-motion paper under a rostrum camera. Nothing is eased, there are no gradients and no glow. The mood is tense, graphic and witty, like the opening titles of a 1950s thriller played on a club wall. This is an original piece built on the rule set described in the research (few shapes, two or three colours, one motion path per object, timing locked to the counts; [Art of the Title: Psycho](https://www.artofthetitle.com/title/psycho/), [Art of the Title: The Man with the Golden Arm](https://www.artofthetitle.com/title/the-man-with-the-golden-arm/)). Do not trace or recreate any Bass poster, title card or logo. All icons are built from primitives, and the palettes below are our own.

Design rule: **one idea per moment.** At any time only one thing moves: the bars, the icon, the type or the transition. The other parts hold still.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Ground | flat palette slot 0 | (inside P1) | 100 % | swaps on a hard cut, never fades |
| 2 | Bars | 1–9 rectangles, each with a lane, entry side and stepped slide; torn-edge SDF | (inside P1) | — | the rhythm section of the look |
| 3 | Notches | small squares stepping along one bar edge on 1/16 hats | (inside P1) | — | off by default |
| 4 | Icon pieces | icon SDF from P0, split by 3–5 straight cut lines into up to 32 pieces, each offset and rotated | (inside P1) | — | assemble / shatter |
| 5 | Type | text texture (white on transparent) as a matte: revealed by bars, knocked out of bars, or sliced | (inside P1) | — | from a Text Block, Wire text, UE render target or Godot SubViewport |
| 6 | Spiral wipe | two-colour Archimedean band that grows from the centre for 1 bar | (inside P1) | — | transition between scenes only |
| 7 | Paper finish (optional) | fibre grain multiply ±2 %, offset hard shadow 0–3 px at 12 % | (inside P2) | 0–100 % | default off; flat is the style |
| 8 | Video fill (optional) | camera or clip shown inside one palette slot (for example only inside the icon) | (inside P1, `fill` image input) | 100 % | the "cut-out window" variation |
| 9 | Output | 1 px anti-aliasing via screen-space derivative, black floor, white cap | (inside P2) | — | no bloom, blur, grain or chromatic aberration |

P0 renders the icon distance field into a texture. P1 composes layers 1–6 and 8 by painter's order and writes a palette index, not a colour. P2 resolves index to colour and applies the finish.

## Techniques

**1. Stepped "boil" clock (the paper feel)**
1. All motion uses quantised time: `tb = floor(time * fps) / fps`, with fps 6, 8 or 12 (12 is "on twos" at 24 fps). The boil seed is `seed = floor(time * fps)`.
2. Positions are not stepped by the beat directly. They move linearly during short slides, then the result is sampled at `tb`. A 0.25 s slide at 12 fps is 3 visible positions, which reads as cut paper pushed by hand.
3. fps = 0 switches the boil off (clean vector look, still with snapping motion).
4. Why: the research names on/off, step-like motion as the core of the language. A continuous 60 fps glide reads as modern motion graphics, not as paper.

**2. Rough hand-cut edges (SDF displacement)**
1. Every shape is a signed distance in output pixels (`d < 0` is inside). Primitives: box, triangle, circle, polygon, from [Inigo Quilez: 2D distance functions](https://iquilezles.org/articles/distfunctions2d/).
2. Displace the edge: `d' = d + rough * amp * (n(p * f + seed * 17.3) - 0.5)`, where `n` is 2-octave value noise ([The Book of Shaders: Noise](https://thebookofshaders.com/11/)). Scissor cut: f = 1/40 px, amp up to 3 px (long, slight wobble). Torn: second octave at f = 1/6 px, amp up to 6 px (fibrous).
3. Use a different noise offset per object (`objectId * 31.7`) so bars do not boil in sync.
4. Coverage = `clamp(0.5 - d' / fwidth(d'), 0, 1)`, which is a 1 px edge. Do not soften further: LED walls turn soft edges into seams.

**3. Bar sequencer (slam in, hold, slam out)**
1. Divide the frame into lanes: 6 horizontal lanes and 8 vertical lanes in units of wall height `H`, so 3:1 walls get more vertical lanes, not wider bars.
2. Each bar i stores `{lane, orientation, side (L/R/U/D), length 0.3–1.0 of span, thickness 0.04–0.14 H, t0}`. On trigger k the bar `i = k mod bars` gets new values from `hash(k, seedBase)`. Bars that share a lane are rejected and re-hashed, so bars never cross (a Psycho-like constraint, applied to new shapes).
3. Position: `x = mix(offscreen, target, clamp((tb - t0) / slide, 0, 1))`, with slide 1/8 beat by default. Linear, no ease. Optional overshoot: one boil frame at 104 % target, then 100 %.
4. Exit: on the next trigger for the same bar, it continues out to the opposite side (or back where it came from on even k). This gives the "on, off, left, right, up, down" vocabulary.

**4. Icon assembly and scissor shatter**
1. Build an icon as a union or subtraction of 3–8 primitives (see icon set below). Draw it into the P0 SDF texture once per icon change, not per frame.
2. Cut it with K = 3–5 random straight lines through the icon's bounding box (K = 5 gives up to 32 pieces). The piece ID of a point is its sign pattern against the K lines, so `id = sum(step(0, dot(p - a_j, n_j)) << j)`.
3. Each piece gets its own offset `o_id = shatter * spread * dir_id` (dir = direction from the icon centre to the piece centroid plus ±20° hash) and rotation `r_id = shatter * ±25°`. The values are rounded to the boil clock.
4. Per pixel, loop over the piece IDs: undo piece id's offset and rotation, sample the icon SDF, compute the sign pattern, and if it equals id and is inside, the pixel is that piece. First hit wins. Cost is 32 texture reads, cheap.
5. `shatter` is changed in steps (0, 0.25, 0.5, 1) by beat, never as a smooth fader, so pieces jump like stop-motion.

**5. Type as matte**
1. Text arrives as a texture. Resolution should be at least the wall height, with letters at 8–20 % of H. Use an open-licence condensed grotesque or your own hand-cut letters scanned as a texture.
2. Three modes: **reveal**, where type shows only where a bar covers it (in ink, on paper bars); **knockout**, where type is cut out of the bars and shows the ground; **slice**, where the text is cut into 3–7 horizontal strips that slide in from alternating sides and lock together on the beat.
3. Run the type alpha through the same edge displacement (sample the texture at `uv + rough * noise offset`, max 2 px) so type boils with the paper.

**6. Spiral wipe (scene hand-off)**
1. Polar coordinates around the centre: `r = length(p) / H`, `a = atan(p.y, p.x) / 2π`.
2. Band: `s = fract(r * turns - a - tb * spin)`, colour = slot A where `s < 0.5`, else slot B. turns = 3–6, spin = 0.5 turns per beat.
3. Mask the spiral by a growing disc `r < grow`, with grow stepping 0 → 1.2 over one bar. Under the disc the old scene is hidden; when the disc shrinks, the new scene is there.
4. It is a generic geometric spiral in two flat colours. Keep it as a transition only, not as a held scene.

**Icon set (original, from primitives)**
- *Eye*: a vesica (two intersected circles) minus a smaller circle, plus a filled pupil circle in accent.
- *Speaker*: a trapezoid cone and a box magnet, with three concentric arcs as separate pieces.
- *Key*: a ring (circle minus circle), a long box shaft, and two notched teeth boxes.
- *Zigzag*: a 6-vertex jagged polygon, thick, rotated about 12°.
- *Hand-cut star*: 7 triangles around a centre, uneven lengths.

## Shaders

**P0 — Icon SDF** (on icon change only)
- Purpose: bake the current icon distance field.
- Inputs: `icon` (int), `iconScale` (0.3–0.8 H), `iconAspect`.
- Per pixel: compute the union/subtraction of the primitives in icon space and store the signed distance in pixels (clamped to ±64).
- Output: R16F texture, 1024 × 1024 (square, independent of wall aspect). Sampled bilinear.

**P1 — Compose** (every frame, full output resolution)
- Purpose: paint layers 1–6 and 8 as palette indices.
- Inputs: `iconSDF` (P0), `typeTex`, `fill` (optional video), uniforms `time`, `fps`, `beatPhase`, `beatCount`, `trigCount`, `trigTime`, `bars`, `slide`, `rough`, `shatter`, `cuts`, `typeMode`, `wipe` (0–1), `seedBase`, `roles` (a 4-entry role-to-slot map).
- Per pixel, in painter's order:
  1. `p` = pixel coordinates centred, in px; `H` = output height.
  2. index = role GROUND.
  3. For each bar (≤ 9): compute the rough box SDF at its stepped position; if covered, index = role PAPER, objectId = bar.
  4. Notches if enabled: small boxes along the most recent bar's edge, `count = floor(beatPhase * 16)`.
  5. Icon pieces (technique 4); if hit, index = role INK, or ACCENT for the pupil or one "hero" piece. If `fill` is active, flag the pixel to show video instead.
  6. Type (technique 5), according to `typeMode`.
  7. Spiral wipe on top when `wipe > 0`.
- Output: RGBA8 with R = slot index of the top object, G = slot index below it, B = edge coverage (for mixing the two at the 1 px edge), A = 1 where a non-ground object sits (used by P2's shadow). Single-pass option: resolve the palette directly in P1 (mix the two slot colours by coverage) and skip P2 when the finish is off. This is the recommended path for Resolume.

**P2 — Finish** (optional, full resolution)
- Inputs: P1, `palette[4]`, `grain` (0–0.02), `shadow` (0–3 px), `blackFloor`, `whiteCap`.
- Per pixel: look up the colour; if `shadow > 0`, sample P1 at `p + (shadow, -shadow)` and darken by 12 % where that sample is a non-ground object and this pixel is ground; multiply by `1 + grain * (fibreNoise(p * 0.7) - 0.5)` with a static seed; clamp to `[blackFloor, whiteCap]`.
- Output: final RGBA, alpha = 1 (or alpha = 0 on ground in "matte mode", see compositing).

Render order: P0 (event) → P1 → P2. No ping-pong, no feedback: all state comes from counters and hashes, so the look is deterministic and restarts cleanly.

## Color scheme

Four slots per palette, assigned to roles. Hard cuts only, never a cross-fade between palettes.

| palette | ground (0) | ink (1) | paper (2) | accent (3) | use |
|---------|-----------|---------|-----------|-----------|-----|
| Grey Trial | #8C8A84 | #151413 | #ECE5D5 | #C8442B | default, calm, verse |
| Poster Heat | #E0622A | #161210 | #F0E8D6 | #2E5A86 | build, high energy |
| Night Cut | #121211 | #ECE5D5 | #E8B13A | #D23A2A | dark club, low ambient |
| Teal Card | #2D6B67 | #141414 | #EEE5D1 | #E0A126 | breaks, cooler sets |
| Strict B/W | #101010 | #E8E8E8 | #E8E8E8 | #101010 | drop hits, maximal contrast |

- Mapping: roles → slots through `roles`. **Invert** swaps ground and paper, so a light poster turns into a dark one in one frame (snare or MIDI pad). **Accent hop** moves the accent role to a different piece or bar each bar.
- Over time: palette changes on phrase boundaries (every 8 or 16 bars) by default. Strict B/W is reserved for drops so it keeps its punch.
- LED walls: black floor #10–#15 (pure #000 looks like a hole and hides the wall seams poorly), white cap about #ECE5D5, never #FFFFFF. Full white bars over a large area at 100 % brightness blind the room. Run the wall at 50–70 % master brightness and keep these values. The accent should stay saturated but not pure (no #FF0000), so it does not clip on LED drivers.
- No gradients, no hue rotation, no tweening of colours inside a palette.

## Blending and compositing

- Inside the shader everything is opaque painter's order. Flat paper does not blend, so no blend modes are used between bars, pieces and type.
- **Over video** (research "layering"): output a matte mode where the ground has alpha 0. Put the layer above live video in Alpha (normal) blend: the video shows through like a photo behind cut card. Alternatively, put the full poster above the video in **Multiply**: the paper becomes a window and the ink stays black. Avoid Add and Screen, which wash out the flat colours.
- **Video inside the icon**: use the `fill` input, so video appears only inside one role. Treat the video first with a hard threshold or two-tone posterise in the same ink and paper colours so it stays flat.
- **Difference** with a second Cut-Paper layer (other seed, other bars) gives sudden overlapping inversions. Use it only for drops, for 1–2 bars.
- Post chain: none. If the venue camera feed needs it, the only allowed step is a 1 px sharpen on the final output after scaling. No bloom, no blur, no chromatic aberration, no film grain beyond P2's 2 %.

## Motion and timing

- Base unit: the beat. Bar slides last 1/8 beat (about 0.25 s at 120 BPM). Hold for at least 1 beat before anything else moves.
- **Kick**: one bar slams (next bar in sequence). At 4/4 with bars = 4, the screen rebuilds once per bar.
- **Snare / clap**: icon shatter steps up one level, or invert (choose one per section, not both).
- **Hats**: notches step along an edge, 1/16. Only in sections with sparse kicks.
- **Phrase (8/16 bars)**: palette change and icon change, always hidden behind a spiral wipe or a full-frame bar slam (one bar covers the frame for one boil frame, then leaves).
- **Build**: over the last 4 bars, shatter steps 0 → 0.25 → 0.5 → 1, rough goes 0.3 → 0.8, boil goes 8 → 12 fps, bars 3 → 7.
- **Drop**: hard cut to Strict B/W, icon reassembled (shatter 0) in one frame, four bars slam in from all four sides at the same time, then **hold the frame still for one full bar**. The stillness after the noise is the impact.
- **Break**: Grey Trial, one bar, one line of type in slice mode, boil 6 fps, rough 0.2, icon hidden or whole.
- Easing: none, except the optional 1-frame overshoot. Everything else is linear motion sampled by the boil clock.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| slam | trigger, plus rate 1/4, 1/2, 1, 2 beats | trigger on kick | moves the next bar in or out | BeatStep Pro drum pads 1–4 force a side (L/R/U/D); kick onset as auto trigger |
| shatter | 0, 0.25, 0.5, 1 (stepped) | 0 | breaks the icon along scissor cuts | KeyStep pads or knob quantised to 4 steps; snare advances one step |
| rough | 0–1 | 0.35 | edge from clean vector to torn card | BeatStep Pro knob; rises on builds |
| boil | 0, 6, 8, 12 fps | 8 | paper jitter rate | 4-position knob or note; 12 on builds, 6 on breaks |
| bars | 1–9 | 4 | how many bars are in play | knob; scale with section energy |
| palette | 0–4, plus invert toggle | 0 | hard cut to another poster palette | KeyStep notes C–E select palettes; F = invert; auto on phrase |

Secondary (exposed but rarely played): `typeMode` (reveal, knockout, slice), `icon` (0–4), `cuts` (3–5), `wipe` trigger, `seedBase` (new layout family).

## Build recipes

### Resolume Arena
- **Layer 1 (optional)**: live video or clip, Normal.
- **Layer 2, the look**: the Wire patch below as a **source** (P1 with palette resolve, single pass). Blend Alpha in matte mode over layer 1, or opaque. Publish slam, shatter, rough, boil, bars, palette, invert, typeMode, icon, wipe. Resolume documents ISF inside Wire through its ISF node ([Resolume ISF support](https://resolume.com/support/en/isf)), so the custom shader runs in Wire.
- **Type layer**: a Text Block source on its own layer, white on transparent, routed into the patch as the `typeTex` input (Wire texture input or Spout round trip). Its text can be changed live from the Arena clip.
- **No-code fallback** (rougher): a Solid Color source as ground; 3–4 layers of a Solid Color clip, each cropped to a bar with Transform (scale X 100 %, Y 8–12 %), with BPM-synced position animation set to jump positions; a Text Block layer above. The boil and the shatter need the custom shader.
- Mapping: map the published parameters with Arena's MIDI shortcuts to the BeatStep Pro and KeyStep. Use the kick audio analysis (Arena's FFT on the layer or composition) on `slam`.

### Resolume Wire
- **Clock**: beat-clock / BPM source → beat phase and beat count; counter driven by the slam trigger (MIDI note or audio onset) → `trigCount`, and a sample-and-hold of time at the trigger → `trigTime`. Node names differ between Wire versions; use the beat-clock, counter and sample-and-hold nodes from the node browser.
- **P0**: ISF node (icon SDF) at fixed 1024 × 1024, only re-rendered when `icon` or `iconScale` changes (gate its render, or accept the low per-frame cost).
- **P1**: ISF node at the composition resolution, inputs: P0 texture, text texture, optional video texture, and all floats. Palette as four colour inputs plus `palette` index logic in Wire (a switch/select per slot).
- **Steps**: quantise `shatter` and `boil` in Wire math (round to the nearest step value) before the ISF node, so the fader can be played freely but still snaps.
- **Exposed**: the six stage parameters plus typeMode, icon, wipe, seedBase. Wire does not support vertex shaders ([Resolume ISF support](https://resolume.com/support/en/isf)). Not needed: all passes are fragment-only.

### Unreal Engine 5.7
- **Approach A (shader, recommended)**: a full-screen post-process or User Interface material that reproduces P1. Material nodes: Time → Multiply(fps) → Floor → Divide(fps) for `tb`; Custom node (HLSL) for the bar loop, line cuts and piece loop; Noise node or a tiling value-noise texture for edge displacement; TextureSample of a render target for the icon SDF and one for the type. Draw the icon SDF into the render target once with Draw Material to Render Target in Blueprint.
- **Approach B (actors)**: real flat pieces as planes or extruded static meshes with an Unlit material, seen by an orthographic Cine Camera. Animate them in Sequencer with **Constant** key interpolation (stepped keys) for the boil look. The torn edge is a Masked material with an opacity mask from a noise texture, its UV offset changed by the stepped time.
- **Type**: Text Render Actor or UMG text captured by a Scene Capture 2D into a render target, then used as the type texture.
- **Niagara (optional)**: a burst of small triangle mesh particles (Mesh Renderer, Unlit flat material in the ink or accent colour, Add Velocity in Cone, Drag) on the drop. Set the system's tick to a low fixed rate if your version offers it, or quantise positions in the material, so the confetti boils too.
- **Remote Control**: put slam (trigger count), shatter, rough, boil, bars, palette and invert in a Material Parameter Collection and expose it in a Remote Control Preset; map them to OSC from Arena/Wire or to MIDI from the BeatStep Pro ([Remote Control protocols](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-protocols-in-unreal-engine)).
- **Output**: native wall resolution, sent to Arena over Spout or NDI with the plugin you already use.

### Godot
- **Scene**: a full-rect ColorRect with a ShaderMaterial (`shader_type canvas_item`) for P1 plus palette resolve.
- **Icon SDF**: a SubViewport (1024 × 1024, update mode Once, re-triggered on icon change) with its own ColorRect running P0; pass the ViewportTexture as `icon_sdf`.
- **Type**: a second SubViewport (transparent background) with a Label, as `type_tex`.
- **Uniforms**: `fps`, `bars`, `slide`, `rough`, `shatter`, `cuts`, `type_mode`, `wipe`, `seed_base`, `trig_count`, `trig_time`, `beat_phase`, `palette` (vec4 array of 4 colours), `invert`.
- **Clock and audio**: a GDScript keeps BPM and beat phase; an AudioEffectSpectrumAnalyzer on the input bus reads the kick band with `get_magnitude_for_frequency_range` and raises `trig_count` on an onset. MIDI via `InputEventMIDI`.

## Audio reactivity

| band | drives | detection | smoothing |
|------|--------|-----------|-----------|
| 40–120 Hz (kick) | slam trigger | onset: energy > 1.6 × 300 ms running mean, re-arm after 120 ms | none (trigger), optionally quantised to the nearest 1/8 beat |
| 150–400 Hz envelope | `bars` target (within the fader range) | RMS | attack 200 ms, release 2 s, then rounded to an integer |
| 1–4 kHz (snare / clap) | shatter step or invert | onset, threshold 1.5 × mean, re-arm 200 ms | none |
| 8–16 kHz (hats) | notch step | onset, re-arm 60 ms | none |
| full-band RMS | `rough` offset (+0 … +0.3) | RMS | attack 80 ms, release 600 ms, then sampled at the boil clock |

Rule: audio may only trigger steps or set targets. It never drives positions continuously, because a smoothly shaking bar breaks the style. On dense tracks, switch audio off and play the triggers from the BeatStep Pro; the look is at its best when the cuts feel composed "to the counts".

## Performance tips

- P1 cost: 9 bar SDFs + up to 32 piece tests (one texture read each) + 2-octave noise per object. Light at 1080p; at 4K keep cuts ≤ 4 (16 pieces) on older GPUs. P0 is close to free because it runs only on icon changes.
- Run P1 at native wall resolution. Never upscale flat shapes: scaled edges turn soft and the 1 px AA becomes a 2–3 px blur.
- Express all sizes in units of output height `H` or in output pixels (edge amp, shadow), not in UV. Otherwise roughness looks different on a 3:1 wall and on a 16:9 screen.
- Odd aspect ratios: lanes are counted in H units, so wide walls get more vertical lanes. Centre the icon on the main viewing axis, not on the geometric centre of an L-shaped or split wall. For multi-panel walls, run one icon per panel with different seeds.
- Large white areas are the main risk: cap white, limit paper bars to about 40 % of the screen in the brightest palettes, and use Night Cut in dark rooms.
- The boil changes every pixel on an edge 8–12 times a second. That is fine on LED walls. On NDI or low-bitrate streams it costs bitrate, so lower `rough` for streamed output.
- 8-bit everywhere is fine (flat colours, no gradients), except P0's SDF, which needs 16-bit float.

## Further references

- [Inigo Quilez: 2D distance functions](https://iquilezles.org/articles/distfunctions2d/) — box, triangle, circle and polygon SDFs used to build the original icons and bars.
- [The Book of Shaders: Noise](https://thebookofshaders.com/11/) — value and gradient noise, and noise used to modulate shapes, which is the basis for the torn edges.
- [Art of the Title: Psycho](https://www.artofthetitle.com/title/psycho/) and [Art of the Title: Anatomy of a Murder](https://www.artofthetitle.com/title/anatomy-of-a-murder) — analysis of the bar and cut-out rule sets this design abstracts (study the rules, do not copy the frames).
- [Resolume: ISF in Wire](https://resolume.com/support/en/isf) — the ISF node, JSON inputs, and the note that Wire has no vertex shader support.
