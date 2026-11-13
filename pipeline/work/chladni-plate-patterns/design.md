# Chladni Plate Patterns — design

## Look

A square plate of deep black fills the centre of the screen. On it, a few hundred thousand pale grains of "sand" form a symmetric figure of nodal lines: a few bold curves at low modes, a dense lattice of diamonds and loops at high modes. Under the sand, a thin amber line traces the exact nodal curve, and the open areas between lines carry a faint signed tint: cold blue where the plate moves up, wine red where it moves down. On a kick the plate is "bowed". The grains jump, smear into a glowing haze, then drift back and settle onto the new figure over one or two beats. The reference is Chladni's 1787 engravings ([Public Domain Review](https://publicdomainreview.org/collection/chladni-figures-1787/)) and the square-plate tables on [Paul Bourke's page](https://paulbourke.net/geometry/chladni), redrawn as light on black. The mood is scientific, calm and exact, with a physical snap on every change.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #050507, plus a 2 px plate border at #1A1A1F | Normal | 100 % | keep LED pixels just above off so the wall does not look dead |
| 2 | Amplitude tint | signed field F mapped to a two-tone ramp, value only up to 18 % | Add | 35 % | gives the plate body; fades to 0 on breaks |
| 3 | Nodal lines | analytic line shader, constant width in pixels | Add | 70 % | resolution-independent; stays sharp on any wall |
| 4 | Sand particles | GPU particles moving down the gradient of abs(F), 1 px point splats | Add | 100 % | main motion payoff; omitted in the Arena-only build |
| 5 | Sand trails | feedback of layer 4, decay 0.82 per frame | Add (inside the feedback buffer) | 60 % | gives haze when grains scatter, nearly invisible when they are still |
| 6 | Plate mask | square matte (short side of the output, centred), 4 px soft edge | Mask (alpha multiply) | 100 % | cuts layers 2–5 to the plate; turn off for full-frame lattice |
| 7 | Post | bloom, then subtle grain, then vignette | see compositing | — | order matters, see below |

## Techniques

**1. Analytic Chladni field (square plate)**
1. Fit a square to the output's short side; `p = (uv - offset) / side`, so p runs 0..1 on the plate.
2. With `a = n·π` and `b = m·π`: `F = cos(a·px)·cos(b·py) + s·cos(b·px)·cos(a·py)`. `s = -1` is the classic formula from [didgety](https://didgety.github.io/blog/2023/chladni_plate_simulation) and [Paul Bourke](https://paulbourke.net/geometry/chladni); `s = +1` gives the second symmetric family. Mixing two waves with weights follows the idea of the [Houdini Chladni node](https://sidefx.com/docs/houdini/nodes/cop/chladni.html).
3. Gradient, computed analytically (cheap and noise-free): `dF/dx = -a·sin(a·px)·cos(b·py) - s·b·sin(b·px)·cos(a·py)`, `dF/dy = -b·cos(a·px)·sin(b·py) - s·a·cos(b·px)·sin(a·py)`.
4. Normalise F by the mode: divide by 2 so it stays in -1..1.
5. Guard: with `s = -1` and `n = m` the field is zero everywhere (Bourke calls n = m uninteresting). The sequencer must never pick n = m in that case; if a fader lands there, add 0.5 to m.

**2. Mode morph (dissolve and re-form)**
1. Keep two mode sets, A (current) and B (target), each (n, m, s).
2. `F = mix(F_A, F_B, e)` and the same for the gradient, where `e` runs 0 to 1 over the settle time with a smoothstep ease.
3. The nodal lines then slide and reconnect continuously instead of cutting.
4. Pitfall: swapping n and m only flips the sign of F (see research). Mixing F(n,m) with F(m,n) passes through zero at e = 0.5 and the whole plate flashes as "all nodal". When the target is the swapped pair, skip the morph or multiply F_B by -1.
5. Non-integer n and m are allowed while a fader moves; they break the square boundary condition but look like in-between figures. Quantise to integers when the fader stops (snap mode).

**3. Sand particles (settling)**
1. Each particle has a position on the plate (0..1) and a velocity.
2. Vibration energy at the particle is `E = abs(F)`. Grains on moving regions get shaken; grains on nodal lines stay put.
3. Force toward the line: `v += -sign(F) · normalize(gradF) · pull · E`. This moves the grain down the gradient of abs(F), as in [didgety](https://didgety.github.io/blog/2023/chladni_plate_simulation) and [luciopaiva/chladni](https://github.com/luciopaiva/chladni).
4. Random jitter: `v += randomUnitVector · jitter · E`. Jitter scaled by E is what makes the physics read right: still grains on lines, nervous grains in open areas.
5. Damping `v *= 0.85` per frame, then `pos += v · dt`. Kick adds a one-frame impulse `shake` to the jitter for every grain, including those on lines.
6. Grains that leave 0..1 respawn at a random position. A tiny density-based repulsion (from [azrlog](https://azrlog.dev/posts/chladni)) is optional; at 262k grains it is not needed, because the jitter already spreads them along the lines.

**4. Constant-width line rendering**
1. Approximate distance to the nodal line as `d = abs(F) / max(length(gradF), 1e-4)`, in plate units.
2. Convert to pixels: `dpx = d · plateSizeInPixels`.
3. `line = 1 - smoothstep(width - 0.75, width + 0.75, dpx)`, with width in pixels (default 1.5).
4. Add a soft halo: `halo = exp(-dpx / 6) · 0.25`. This keeps lines equally thick at low and high modes, unlike a plain `abs(F) < threshold`, which makes fat blobs where the gradient is flat.

**5. Sand trail feedback**
1. Buffer T (same size as the particle splat target).
2. Each frame `T = T · decay + splat`, decay 0.82 by default.
3. On a kick, raise decay to 0.92 for 250 ms so the scatter leaves a glowing smear, then ease back.

## Shaders

Render order per frame: P0 field, P1 particle sim, P2 splat, P3 trail feedback, P4 composite, P5 post.

**P0 — Field pass**
- Purpose: compute F and its gradient once, for both lines and particles.
- Inputs: uniforms nA, mA, sA, nB, mB, sB, morph e, plate rect.
- Per pixel: compute p, F_A and F_B with their analytic gradients, mix with e, write R = F, G = dF/dx, B = dF/dy, A = abs(F).
- Output: RGBA16F, 1024 × 1024 (plate-space, square). 2048² for 4K walls or modes above 12.

**P1 — Particle simulation (ping-pong)**
- Purpose: move grains toward nodal lines.
- Inputs: position/velocity texture (previous frame), field texture P0, uniforms pull, jitter, shake, damping, dt, time seed.
- Per texel (one texel = one grain): read pos (RG) and vel (BA); sample P0 bilinear at pos; apply technique 3 steps 3–5; respawn if outside; write new pos and vel.
- Output: RGBA32F, 512 × 512 = 262,144 grains (256 × 256 for laptops, 1024 × 1024 for big walls). Two textures, swapped each frame.

**P2 — Splat**
- Purpose: draw grains.
- Inputs: P1 output as vertex positions.
- Per grain: one point (1 px, or a 2 × 2 px quad on 4K), brightness `0.35 + 0.65 · clamp(speed · 8, 0, 1)` so moving grains glow brighter. Additive.
- Output: RGBA16F at output resolution (plate area only).
- Note: needs vertex fetch; not available in Wire (no vertex shaders). Use UE5, Godot or TouchDesigner for this pass.

**P3 — Trail feedback (ping-pong)**
- Inputs: previous trail buffer, P2 splat, decay.
- Per pixel: `T = T_prev · decay + splat`, clamp to 4.0.
- Output: RGBA16F at output resolution, two buffers swapped.

**P4 — Composite**
- Inputs: P0 (sampled at plate coordinates), P3, palette uniforms, line width, tint amount.
- Per pixel: background colour; add signed tint (F > 0 toward #1E3A8A, F < 0 toward #6B1A33, scaled by abs(F) and tint amount); add line + halo in line colour; add sand in sand colour from P3; apply plate mask.
- Output: RGBA16F at output resolution.

**P5 — Post**
- Bloom (threshold 0.75, intensity 1.2, radius medium), grain 3 %, vignette 0.25, then tonemap or clamp to 8-bit for Spout/NDI.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base / black | #050507 | background, black level |
| plate edge | #1A1A1F | thin border, barely visible |
| sand | #EDE3D0 | grains, warm bone white |
| line accent | #FFB547 | nodal-line glow, amber |
| amplitude + | #1E3A8A | tint where plate moves up (very dim) |
| amplitude − | #6B1A33 | tint where plate moves down (very dim) |
| highlight | #FFFFFF | kick flash on sand only, 80 ms |

- Mapping: tint is a two-colour signed ramp, `tint = F > 0 ? colA·F : colB·(-F)`, then times tint amount (default 0.35). Lines and sand are flat colours with brightness from the shaders.
- Over time: per phrase (16 bars) rotate the line accent along a short list: #FFB547 amber, #4FD1FF cyan, #B7FF5A acid green. Sand stays warm white so the figure is always readable.
- Music: bass level scales tint amount (0.15 to 0.5). Kick flashes sand to #FFFFFF for 80 ms.
- LED walls: keep the black at #050507, not pure black, to avoid visible dead modules; cap peak line brightness at about 85 % before bloom so 1 px lines do not clip into blocky white on low-pitch walls. Contrast is high by design; do not lift the tint above 0.5 or the figure loses its edges.

## Blending and compositing

- Everything above the background is **Add**: it is light on black, and additive keeps overlaps between line, sand and halo bright rather than muddy.
- The **plate mask** multiplies layers 2–5 so the figure has a physical edge. Off, the lattice tiles across the whole frame (good for wide LED strips; set the plate to the long side and accept that the pattern is non-square).
- **Feedback mix**: trail buffer decay 0.82 (range 0.6–0.95), trail layer at 60 %. Above 0.95 the sand never looks settled.
- **Post chain in order**: bloom (threshold 0.75, intensity 1.2) → grain 3 % monochrome → vignette 0.25 → optional sharpen 0.2 on walls with pixel pitch above 4 mm. No chromatic aberration: it smears the 1 px lines.
- **As a matte for other video** (research variation 4): output the line layer alone in white as a Luma Key source over video, or feed F into a Displace effect.

## Motion and timing

- **Settle time**: grains need 0.5–2 s to settle (pull and jitter control this). Default: target set reached in 1 beat, fully settled in 2 beats at 124 BPM. As Nigel Stanford notes, the tone must stay long enough for the figure to form ([nigelstanford.com](https://nigelstanford.com/cymatics)); so change modes at most once per bar during dense parts.
- **Mode morph**: e eased with smoothstep over 1 beat (range 1/4 beat to 4 bars).
- **Sequencer**: one (n, m, s) set per bar from a list, e.g. (1,2,-1) (1,3,-1) (2,3,-1) (1,4,+1) (2,5,-1) (3,5,-1) (3,7,+1) (4,7,-1). Low modes in intros, high modes in builds.
- **Kick**: shake impulse (grains jump) without changing mode. Every 4th kick (bar downbeat) also starts the morph to the next set.
- **Build**: shake grows from 0 to 1 over the last 8 bars, so the sand never settles; n and m climb.
- **Drop**: hard snap. Morph time 0, shake 1.0 on the drop hit, then settle into a bold low mode (e.g. (1,3)) within 2 beats. Line accent switches colour.
- **Break**: sand fades to 30 %, lines only, with a slow non-integer drift of m (0.05 per second) so the figure breathes.
- Slow camera: optional plate rotation 0–2° per bar, zero during drops.

## Parameters

| name | range | default | what it does on stage | suggested MIDI / BPM mapping |
|------|-------|---------|----------------------|------------------------------|
| mode | 0–7 (index into set list) or n, m 1–12 | (1,3,-1) | picks the figure | BeatStep Pro pads 1–8 pick the set; or two KeyStep knobs for n and m |
| morph | 0–4 bars | 1 beat | how long one figure takes to turn into the next | knob; 0 = snap on drops |
| shake | 0–1 | 0.1 | how hard the grains are thrown; high = haze, low = crisp sand | kick envelope, plus a fader for builds |
| settle | 0.2–3 (pull strength) | 1.0 | how fast grains find the lines | knob; tie to BPM so settle = 2 beats |
| line | 0–4 px | 1.5 | width of the amber nodal line; 0 = sand only | knob; hi-hat adds +0.5 px jitter |
| symmetry (s) | -1 to +1 | -1 | morphs between the two plate families; 0 gives plain checker bands | fader, slow moves only |

## Build recipes

### Resolume Arena

Arena cannot run the particle passes, so the Arena build is the line-and-tint look, with sand faked.
- **Layer 1**: Solid Color source, #050507.
- **Layer 2**: the Chladni field as a single-pass ISF source (P0 and P4 merged: field, lines, tint). ISF loads in Wire through the ISF node ([Resolume ISF support](https://resolume.com/support/en/isf)); export the Wire patch as a source to Arena. Expose mode n, m, s, morph e, line, tint as parameters. Blend: Add, 100 %.
- **Fake sand**: Layer 3, a copy of the Chladni source with line width 3 px, then the built-in **Noise** effect for grain and a **Threshold** to break it into dots, Add at 50 %. This reads as dust at a distance.
- **Effects on the composition**: Bloom (threshold 0.75, intensity about 1.2), then Vignette.
- **Real sand**: run the particle build in UE5, Godot or TouchDesigner and bring it in as a Spout or NDI source on Layer 3, Add.
- **Audio**: Arena's audio analysis (low band) on shake and tint; BPM sync on the mode index via a Dashboard parameter with "BPM sync / step" animation.
- **Matte use**: put the line layer above a video layer, set the video layer's Mask to use the line layer's luma, so the video only shows inside the nodal lines.

### Resolume Wire

Node outline (line-and-tint source; sand via Spout input):
- `Time` → `Smooth` (envelope for e) and `BPM` / beat counter → `Counter` (wrap 8) → `Array` lookup of n, m, s pairs.
- Two lookups (current and next index) → ISF node inputs nA, mA, sA, nB, mB, sB.
- `Audio Analysis` (FFT) → low band → `Envelope` (attack 5 ms, release 250 ms) → shake / tint inputs. Per the Resolume forum, FFT arrays become textures via a ToTexture node if the shader needs the spectrum ([forum](https://www.resolume.com/forum/viewtopic.php?p=91404)).
- ISF node (Chladni field + lines + tint) → `Blend` (Add) with a `Spout Receiver` carrying the particle layer → `Bloom`-style chain (blur + add) → output.
- Exposed parameters: mode index, morph, line, symmetry, tint, shake.
- Multipass ISF support in Wire has been version-dependent according to forum reports; keep the ISF single-pass and do feedback with Wire's own feedback node if needed.

### Unreal Engine 5.7

- **Field**: a Material Function `MF_Chladni` (inputs: p, n, m, s; outputs: F, gradF) built from Cos/Sin/Multiply nodes or one Custom node. Used in both the plate material and Niagara.
- **Plate material** (unlit, translucent Additive, on a plane facing the camera or as a post-process material): `MF_Chladni` twice (A and B), Lerp by morph, then line = 1 − SmoothStep on abs(F)/length(grad) in pixels (use DDX/DDY of p for the pixel size), Emissive = line · LineColor + tint.
- **Niagara**: GPU Compute Sim, Spawn Burst Instantaneous 262,144 (Fixed Bounds set to the plate). Modules: Initialize Particle (random position in the box, sprite size 0.15 cm), a **Scratch Pad** module that evaluates the field and gradient analytically from user parameters and adds the pull and jitter to Velocity, Drag (about 6), Solve Forces and Velocity, a Kill/respawn check (wrap or re-randomise position when outside). Sprite Renderer, unlit additive material with Particle Color from speed.
- **User parameters to expose** (Remote Control preset, OSC and MIDI protocols): User.N, User.M, User.S, User.Morph, User.Shake, User.Pull, User.LineWidth, User.TintAmount. Drive Morph with a timeline or Sequencer; drive Shake from OSC (kick envelope from Resolume or an audio analyser).
- **Post-process volume**: Bloom intensity 1.2, threshold 0.75; Exposure fixed (manual, no eye adaptation, or the flash pumps); Film Grain 0.03; Vignette 0.25.
- **Output**: Spout via a third-party plugin (for example Off World Live's toolkit) or NDI via the NDI SDK for Unreal; render at the LED processor's native resolution.

### Godot

- **Field + lines**: a `ColorRect` filling the viewport with a `canvas_item` shader; uniforms `n_a, m_a, s_a, n_b, m_b, s_b, morph, line_px, tint, line_color, plate_rect`. Use `fwidth(p)` to convert distance to pixels.
- **Sand**: `GPUParticles2D` with amount 262144, lifetime long (one-shot off, explosiveness 1), and a `ShaderMaterial` with `shader_type particles;`. In `process()` read the same field uniforms, compute F and gradient, update `VELOCITY` with pull, jitter and damping, respawn outside the plate. Draw with a 1 px white texture, `CanvasItemMaterial` blend mode Add.
- **Trails**: render the particles into `SubViewport` A. A second `SubViewport` B holds a `ColorRect` whose shader draws B's previous frame (via a `ViewportTexture`, one frame delayed through a third viewport or a copy) times 0.82 plus A. Simpler fallback: `SubViewport` A with clear mode Never plus a full-screen black `ColorRect` at alpha 0.18 drawn first each frame (decay via alpha fade; works in 8-bit, leaves faint ghosts).
- **Post**: `WorldEnvironment` glow (2D: enable in Environment with Canvas background mode), intensity 1.2, HDR threshold 0.75.
- **Control**: map MIDI via `InputEventMIDI` to the uniforms; for Spout, use a community Spout addon if available, otherwise capture via NDI from a screen-capture tool.

## Audio reactivity

| band | drives | smoothing | threshold |
|------|--------|-----------|-----------|
| kick, 40–100 Hz (didgety reads it just below 100 Hz) | shake impulse; every 4th kick advances mode | attack 2 ms, release 180 ms | onset at +6 dB over the 1 s average |
| bass, 60–250 Hz level | tint amount 0.15–0.5 | attack 50 ms, release 400 ms | none, continuous |
| hi-hat, 6–12 kHz | line width +0.5 px jitter, sand sparkle (brightness of fastest grains) | attack 1 ms, release 60 ms | gate at -30 dBFS |
| mid, 300 Hz–2 kHz energy | jitter (base nervousness of grains) 0.02–0.2 | attack 100 ms, release 800 ms | none |

Keep automation slow and threshold-gated: the didgety build reports constant flashing when automation drives too much ([didgety](https://didgety.github.io/blog/2023/chladni_plate_simulation)). Only the kick is fast; everything else breathes.

## Performance tips

- **Field + lines**: one full-screen pass, about 20 trig calls per pixel with morph. Trivial even at 4K. If you skip particles, compute the field inline in the composite pass and drop P0.
- **Particles**: 262k grains cost about 1–2 ms on a mid-range GPU (sim + splat). 1M grains are fine on a desktop GPU; use them on walls wider than 3840 px, or the sand looks thin.
- **Field texture**: sample P0 bilinear at 1024²; at modes above 12 the gradient aliases, so compute the gradient analytically in the particle shader instead of sampling.
- **Trail buffers**: RGBA16F at output resolution, two copies; on 8K canvases render the trail at half resolution and upscale (the haze is soft anyway).
- **Odd aspect ratios**: the plate is square. For a 32:9 strip either show 2–3 plates side by side (tile p with `fract`, mirror every second plate so lines join) or switch the plate mask off and stretch the period only along x.
- **1 px lines on LED**: low-resolution walls (P5+ pitch) lose 1 px detail; set line to 2–3 px and grain splats to 2 × 2 px.
- **Pitfalls**: n = m with s = -1 (blank plate); swapping n and m mid-morph (flash to black); auto-exposure in UE pumping on kicks; feedback decay above 0.95 (never settles).

## Further references

- [didgety: Chladni plate simulation in TouchDesigner](https://didgety.github.io/blog/2023/chladni_plate_simulation) — formula, gradient-driven particles, kick-driven audio version.
- [luciopaiva/chladni](https://github.com/luciopaiva/chladni) — JavaScript particle simulation with a precomputed gradient field; [live demo](https://luciopaiva.com/chladni).
- [Paul Bourke: Chladni plate interference surfaces](https://paulbourke.net/geometry/chladni) — square and circular plate formulas (Bessel functions for circles) with tables of modes.
- [SideFX Houdini: Chladni node](https://sidefx.com/docs/houdini/nodes/cop/chladni.html) — two-wave mix, nodal/absolute/signed outputs, threshold and smoothing as a reference for parameter design.
