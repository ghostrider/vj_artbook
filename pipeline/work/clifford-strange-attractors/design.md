# Clifford Strange Attractors — design

## Look

A single luminous form floats on near-black. It looks like smoke, silk, or a folded sheet of glowing gauze. Thin bright filaments pile up into hot, caustic-like seams, and wide veils of faint mist fade out between them. Brightness follows the log of how often the orbit passes through each pixel. Dense seams burn white-amber, mid-density veils glow teal and ember, and the sparse edges fall to indigo and then black. The shape never sits still. It slowly breathes along a closed loop in parameter space, and on bar or phrase changes it stretches, folds and re-forms into a new figure. Kicks make it flare and sharpen for a moment. The reference images are the density renders on [Paul Bourke's Clifford page](https://paulbourke.net/fractals/clifford/), with colour driven by step length as in [Ricky Reusser's WebGL notebook](https://rreusser.github.io/notebooks/clifford-and-de-jong-attractors), set in motion like the parameter sweeps by [Sekara](https://vedransekara.github.io/2019/10/01/temporal_attractors.html). The mood is hypnotic, organic and slightly cosmic: a long-exposure photograph of something alive.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #04030A | Normal | 100 % | just above black, so LED modules do not look dead |
| 2 | Density accumulation | GPU orbit particles, hit histogram (hits + summed step length), temporal decay | (internal buffer) | — | the core; not shown directly |
| 3 | Attractor colour | log-density tone map × gradient ramp indexed by mean step length | Add | 100 % | main image |
| 4 | Ghost echo | the same colour layer through a feedback loop: zoom 1.004, rotate 0.05°/frame, decay 0.90 | Add | 25 % | gives smoky trails during morphs; 0 % on hard cuts |
| 5 | Hot core | threshold of layer 3 luma above 0.85, blurred 6 px, tinted #FFE9C7 | Add | 40 % | makes the dense seams read as caustics |
| 6 | Post | bloom, then grain, then vignette | see compositing | — | order matters |

## Techniques

**1. The map (Clifford, with optional de Jong blend)**
1. Clifford: `x' = sin(a·y) + c·cos(a·x)`, `y' = sin(b·x) + d·cos(b·y)` ([Bourke](https://paulbourke.net/fractals/clifford/)).
2. de Jong: `x' = sin(a2·y) − cos(b2·x)`, `y' = sin(c2·x) − cos(d2·y)` ([Bourke, de Jong](https://paulbourke.net/fractals/peterdejong/)). It has its own parameter set (a2..d2).
3. Blend: `p' = mix(clifford(p), dejong(p), family)`, with family in 0..1. In-between values are a new hybrid map, not a crossfade of images, so the shape morphs instead of dissolving. Keep family at 0 or 1 most of the time and sweep it only over 2–4 bars.
4. Bounds (for framing): a Clifford orbit always stays inside `|x| ≤ 1 + |c|`, `|y| ≤ 1 + |d|`. A de Jong orbit stays inside `|x|, |y| ≤ 2`. Use these to auto-fit the form to the screen (technique 4).

**2. Orbit particles (state that persists between frames)**
1. Keep N particles (default 1,048,576 = a 1024 × 1024 state texture or buffer). Each particle holds its position (x, y) and a warm-up counter.
2. Each frame every particle applies the map k times (default 16, range 4–64). Every step is splatted into the histogram (technique 3), so one frame adds N·k hits (16.8 M by default).
3. Because the state persists, a parameter change does not restart anything. The points fall onto the new attractor within about 10–20 steps (one frame at k = 16), so morphs look continuous.
4. Respawn: each frame re-seed 0.5 % of the particles (picked by hash(id, frame)) at a random point in [−2, 2]². Also re-seed any particle that is NaN or outside 4× the bounds. A re-seeded particle runs 12 warm-up steps without splatting, to skip the transient path that would draw stray lines.
5. Why respawn at all: at some parameter values the orbit collapses to a few fixed points. Without re-seeding, the particles would stay trapped there after the parameters move on.

**3. Density histogram with colour channel**
1. Two accumulators per pixel: `H` = hit count, `S` = sum of step lengths `|p' − p|`, following Reusser's colour-by-last-step idea ([notebook](https://rreusser.github.io/notebooks/clifford-and-de-jong-attractors)). Bourke notes that the per-hit increment can carry other attributes such as curvature ([Bourke](https://paulbourke.net/fractals/clifford/)).
2. Compute path: `atomicAdd` into two uint storage buffers at output resolution. Increment H by 1, and S by `round(step · 64)`.
3. Rasteriser path (Niagara, Godot particles): draw one additive point per particle per frame into an RGBA16F target with colour `(1, step, 0, 0)`. Additive blending sums them, which gives the same H and S.
4. Temporal resolve: `A = A · decay + new` into a float accumulation buffer, then clear the new-hit buffers. Decay 0.80 gives a crisp shape that follows morphs at once. 0.97 gives a silky long exposure, but morphs smear.
5. Normalise so the look does not depend on particle count or resolution: `h = A.H · pixelCount / (N · k · (1 / (1 − decay)))`. With this, the mean density over the whole frame is about 1.

**4. Framing and morph**
1. Fit: `scale = fitFactor / max(1 + |c|, 1 + |d|)`, scaled by the short side of the output. fitFactor is 0.9 for full frame. Smooth scale with a 1 s release so the image does not pump when c or d moves.
2. Presets: Bourke's sets, for example (−1.4, 1.6, 1.0, 0.7), (1.6, −0.6, −1.2, 1.6), (1.7, 1.7, 0.6, 1.2), (1.5, −1.8, 1.6, 0.9), (−1.7, 1.3, −0.1, −1.2), (−1.7, 1.8, −1.9, −0.4), (−1.8, −2.0, −0.5, −0.9) ([Bourke](https://paulbourke.net/fractals/clifford/)).
3. Morph between presets: interpolate (a, b, c, d) linearly with a smootherstep ease (`e = t³(t(6t − 15) + 10)`). Linear paths can pass through collapsed regions where the form shrinks to dots. Audition every preset pair in rehearsal and keep a list of "safe pairs".
4. Breathing loop: `a = a0 + r·cos(2πφ)`, `b = b0 + r·sin(2πφ)`, `c = c0 + 0.5r·sin(2πφ)`, `d = d0 + 0.5r·cos(2πφ)`, with φ = bars / loopLength. A closed loop in parameter space repeats seamlessly, as in gumeo's moving attractor ([post](https://gumeo.github.io/post/visualizing-strange-attractors)). r is 0.02–0.15. Above about 0.2 the shape often breaks.

**5. Tone map and colour**
1. Lightness: `L = log2(1 + h · gain) / log2(1 + peak · gain)`, clamped to 0..1. Default gain 1, peak 40 (range 8–200). Then `L = pow(L, 0.85)`.
2. Colour index: `t = (A.S / A.H) / stepMax`, with `stepMax = 2·sqrt((1+|c|)² + (1+|d|)²)`. Clamp t to 0..1, then `t = fract(t · 1.5 + hueShift)`.
3. `rgb = ramp(t) · L`, then mix toward the highlight colour by `smoothstep(0.8, 1.0, L)`, so the densest seams turn white-hot whatever their hue.
4. Pixels with H below 0.5 (normalised) are left black. This kills single-hit speckle in the far mist.

## Shaders

Render order per frame: P1 iterate + splat, P2 resolve/decay, P3 colour, P4 echo feedback, P5 composite + post.

**P1 — Iterate and splat (compute shader, or particle sim + additive point draw)**
- Purpose: advance the orbits and count hits.
- Inputs: particle state buffer (RG = position, B = warm-up counter), uniforms a, b, c, d, a2, b2, c2, d2, family, k, scale, offset, frame seed, respawn rate.
- Per particle: optional re-seed; loop k times: apply the map, compute the step length, convert to pixel coordinates `px = (p · scale + 0.5) · resolution`, and if the warm-up counter is 0 and the point is on screen, atomicAdd H += 1 and S += step·64. Then write the new state.
- Output: uint buffers H and S (output resolution), updated state buffer. Workgroup 256. For 1 M particles, dispatch 4096 groups.
- Rasteriser variant: the particle update does one step per frame (k = 1), and N goes up to 2–4 M to make up for it. Points go to an RGBA16F target with additive blending.

**P2 — Resolve and decay (fragment or compute, ping-pong)**
- Inputs: H, S (new hits), previous accumulation A_prev, decay.
- Per pixel: `A = A_prev · decay + (H, S/64)`. Clear H and S to 0 (in the same compute pass, or with a buffer clear).
- Output: RG32F at output resolution, two buffers swapped each frame. RG16F is enough when decay ≤ 0.95.

**P3 — Colour (fragment)**
- Inputs: A, ramp texture (256 × 1 RGBA8, or 5 colour uniforms), gain, peak, hueShift, highlight colour, flash.
- Per pixel: normalise h, compute L and t as in technique 5, look up the ramp, mix in the highlight, multiply by `1 + flash`.
- Output: RGBA16F at output resolution.

**P4 — Echo feedback (fragment, ping-pong)**
- Inputs: P3, previous echo buffer, echo zoom 1.004, echo rotate 0.05°, echo decay 0.90.
- Per pixel: sample the previous echo at `(uv − 0.5) · R(θ) / zoom + 0.5`, multiply by decay, then add P3 · 0.15.
- Output: RGBA16F. Half resolution is fine because the echo is soft anyway.

**P5 — Composite and post**
- Background + P3 (Add) + P4 · echo amount (Add) + hot core (threshold P3 luma at 0.85, 6 px blur, Add 40 %), then bloom, grain and vignette. Clamp or tonemap to 8-bit for Spout/NDI.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base / black | #04030A | background, black level |
| shadow | #1B0F3A | ramp stop 0.0: sparse mist, indigo |
| mid | #0E7C86 | ramp stop 0.35: veils, deep teal |
| accent | #FF6B35 | ramp stop 0.7: fast folds, ember |
| warm | #FFB347 | ramp stop 0.9 |
| highlight | #FFE9C7 | densest seams and kick flash, white-hot cream |

- Mapping: a 5-stop gradient ramp indexed by mean step length (t), times log density (L). Slow areas of the orbit (short steps) read cool, fast areas (long jumps) read warm, which gives every form a two-tone depth without any 3D.
- Alternative palettes for phrase changes. "Ice": #050814, #1C2E6B, #3FA9F5, #B8F3FF, #FFFFFF. "Acid": #06040A, #2B0B3F, #C2185B, #FFD23F, #F7FFF0.
- Over time: hueShift drifts 0.02 per bar, so the colours slide along the form. The palette swaps on 16-bar phrase boundaries with a 1-beat crossfade of the ramp texture.
- Music: mid-band energy adds up to ±0.08 to hueShift. A kick sets flash to 0.4 (decay 150 ms).
- LED walls: keep black at #04030A. Cap L at 0.92 before the highlight mix, so the white-hot seams do not clip into flat blocks. The log mapping already compresses the range, so do not add extra contrast curves. On high-pitch walls, raise peak (for example to 80) so the thin filaments do not vanish.

## Blending and compositing

- All light layers are **Add**: this is light on black, and overlapping veils should brighten, not muddy.
- **Accumulation decay** (P2) is the main "trail" control: 0.85 default, 0.6 on drops, 0.97 on breaks.
- **Echo feedback** (P4): decay 0.90, zoom 1.004, rotation 0.05° per frame, mixed at 25 %. Set it to 0 on hard cuts so the old form does not ghost into the new one.
- **Masks**: none needed. For a matte, the L channel alone (grey) works as a Luma Key over video in Resolume, so video shows through the smoke.
- **Post chain in order**: bloom (threshold 0.7, intensity 1.3, radius large) → film grain 2 % monochrome (hides the per-frame noise in sparse areas) → vignette 0.3 → optional sharpen 0.15 on walls with pitch above 4 mm. Skip chromatic aberration by default. On a drop, a 1-beat chromatic split of 3 px works as an accent.

## Motion and timing

- **Breathing loop**: loopLength 16 bars (range 4–64), radius r 0.06. It never stops, so the form is never static, which is the main risk named in the research.
- **Preset morph**: on every 8th bar, morph to the next safe preset over 2 bars with smootherstep. In builds, shorten it to 1 bar.
- **Family sweep**: once per 32-bar phrase, sweep family 0 → 1 → 0 over 4 bars (Clifford to de Jong hybrid and back).
- **Kick**: flash 0.4 (150 ms release). Decay dips to 0.65 for 120 ms so the form looks sharper on the hit. Parameter nudge: d += 0.02, released over 300 ms. It is small enough not to break the shape.
- **Build**: r grows from 0.06 to 0.15 over 8 bars, hueShift speeds up 4×, echo rises to 45 %.
- **Drop**: hard cut. Preset jump with morph time 0, echo 0, decay 0.6, flash 1.0 on the hit. Then the breathing loop resumes at r = 0.1 with loopLength 4 bars, so it moves faster.
- **Break**: decay 0.97 (long exposure), r 0.03, loopLength 32 bars, bloom +0.3, palette "Ice". Slow rotation of the whole form, 2° per bar.
- **Rotation**: 0–3° per second, global, applied in P1 when converting to pixel coordinates (no resampling cost).

## Parameters

| name | range | default | what it does on stage | suggested MIDI / BPM mapping |
|------|-------|---------|----------------------|------------------------------|
| shape | preset index 0–7 | 0 (−1.4, 1.6, 1.0, 0.7) | picks the base figure; jumps or morphs to it | BeatStep Pro pads 1–8; morph by default, snap while a "cut" pad is held |
| drift | 0–0.2 (loop radius r) | 0.06 | how much the form breathes and twists | KeyStep knob; bass level adds up to +0.04 |
| morph | 0–4 bars | 2 bars | time to travel to a new preset | knob; 0 on drops |
| trails | 0.6–0.97 (decay) | 0.85 | crisp shape vs. silky long exposure | fader; kick dips it to 0.65 |
| glow | peak 8–200 (inverted) + flash | peak 40 | brightness and how much of the faint mist shows | fader; kick sets flash |
| family | 0–1 | 0 | Clifford → de Jong hybrid | fader, slow moves only; BPM-synced 4-bar sweep per phrase |

## Build recipes

### Resolume Arena

Arena cannot scatter points, so the attractor itself is rendered in an engine and Arena composes and colours it.
- **Source**: a Spout (or NDI) source from UE5, Godot, TouchDesigner or a small Rust/wgpu app. Send a two-channel "data" frame: R = L (log density, already tone-mapped, 0..1), G = t (colour index), B = 0. Sending tone-mapped L keeps 8-bit Spout transport usable. Raw counts would band.
- **Colour**: a custom ISF effect built and exported from Wire (ISF runs through Wire's ISF node per [Resolume's ISF page](https://resolume.com/support/en/isf)). It reads R and G, looks up the 5-stop ramp, and applies hueShift, highlight mix and flash. Expose hueShift, palette index (0–2) and flash.
- **Layer 1**: Solid Color #04030A. **Layer 2**: the Spout source with the ISF colour effect, Add, 100 %. **Layer 3**: a duplicate of layer 2 with Blur (6 px) and the built-in Levels effect pulling the input black up to 85 %, tinted cream, Add 40 % (hot core). Arena's own feedback/trail effects can stand in for P4 on a third copy at 25 %.
- **Composition effects**: Bloom (threshold about 0.7, intensity about 1.3), then Vignette.
- **Control**: the engine receives a, b, c, d, family, decay and flash over OSC. Arena's audio analysis and BPM clock can send OSC out to the engine, or a MIDI controller can talk to the engine directly. The colour and post stay in Arena.

### Resolume Wire

Wire has no vertex shaders ([Resolume ISF](https://resolume.com/support/en/isf)), and multipass ISF support is version-dependent. So a full histogram is not possible inside Wire. There are two options:
- **A, recommended: Wire as colour and control patch.** `Spout Receiver` (engine data frame) → ISF node "attractor-colour" (ramp lookup, highlight, flash) → `Blend` Add with a feedback branch (Wire's feedback node, `Transform` zoom 1.004, `Multiply` 0.90) → blur + add for bloom → output. The control side: `BPM` / bar counter → `Counter` (wrap 8) → array lookup of preset a..d. Then `LFO` (sine and cosine, period = loopLength bars) × drift added to a and b, a `Smooth` node for the morph, and an OSC Out node sending a, b, c, d, family and decay to the engine. Exposed: shape, drift, morph, trails, glow, family, palette.
- **B, "sketch mode" inside Wire only.** It is honestly thin, but it works as a calm line-drawing variant. A single-pass ISF in which every pixel re-computes the same short orbit (K = 96 points, seeded from the frame number) and sums a Gaussian splat `exp(−d² / σ²)` with σ = 1.2 px around each orbit point. Feed this into a Wire feedback loop with decay 0.985, so about 6,000 points build up into fine glowing threads. Render it at 960 × 540 and upscale. The cost is K iterations per pixel, so keep K ≤ 128.

### Unreal Engine 5.7

- **Niagara system `NS_Clifford`**: GPU Compute Sim, Fixed Bounds ±300 cm. Emitter spawn: Spawn Burst Instantaneous 2,097,152 (1 M on laptops), infinite lifetime.
- **Particle Spawn**: a Scratch Pad sets Position to a random point in [−2, 2]² (in attractor units, stored in a custom `Particles.Orbit` Vector2 attribute), with warm-up 12.
- **Particle Update**: a Scratch Pad with one Custom HLSL node applies the map once to `Orbit` (using User.A … User.D, User.A2 … User.D2, User.Family), writes `Particles.Step` = step length, handles respawn (0.5 % chance per frame from a random float, plus a NaN/out-of-bounds check), and sets Position = Orbit × User.Scale (cm) on the camera plane. No forces and no Solve Forces needed, because the position is set directly.
- **Sprite Renderer**: material `M_OrbitPoint`, unlit, Additive, Emissive = (1, Step, 0) × 0.01. Sprite size about 1.5 px at the output resolution under an orthographic camera.
- **Accumulation and colour**: a post-process material `PP_AttractorTone` (Blendable Location: Replacing the Tonemapper) reads SceneTexture PostProcessInput0. It computes h = R × norm, t = G / R, L = log2(1 + h·gain) / log2(1 + peak·gain), and the ramp from a Curve Atlas row or a 256 × 1 texture. For temporal decay above what the frame gives, use a SceneCapture2D into an RGBA16F render target plus a Blueprint `Draw Material to Render Target` ping-pong (`A = A·decay + new`), and display the result on a full-screen plane.
- **Disable**: TSR/TAA (anti-aliasing method None) and motion blur, which smear and ghost the point cloud. Exposure manual at a fixed EV, because auto-exposure fights the flash.
- **Post-process volume**: Bloom intensity 1.3, threshold 0.7. Film Grain 0.02, Vignette 0.3.
- **Remote Control preset** (OSC and MIDI protocols): User.A, User.B, User.C, User.D, User.Family, User.Drift, User.Decay, User.Gain, User.Peak, User.HueShift, User.Flash. Do the preset morph and the breathing loop in a Blueprint (Timeline + sin/cos), driven by an OSC BPM/bar counter.
- **Output**: Spout through a third-party plugin (for example Off World Live's toolkit), or NDI through the NDI SDK for Unreal. Render at the LED processor's native resolution.

### Godot

- **Best path (Forward+ or Mobile renderer)**: a compute shader (GLSL 450, `#[compute]`) as described in the [Godot compute shader docs](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html). Run it on the main RenderingDevice (`RenderingServer.get_rendering_device()`), not a local one, so no CPU readback is needed. Bindings: particle state SSBO (vec2 + uint per particle), H and S uint SSBOs (width × height), uniform block for a..d, a2..d2, family, k, scale, seed. Pass 1: iterate + `atomicAdd`. Pass 2: resolve into an RG32F image with decay, then clear H and S. Show the image through a `Texture2DRD` on a full-screen `TextureRect`. Split dispatches if a frame takes long, because the docs warn about Windows TDR timeouts.
- **Colour**: the `TextureRect` gets a `canvas_item` shader with uniforms `gain, peak, hue_shift, flash, highlight_color` and `sampler2D ramp` (a `GradientTexture1D`, 5 stops from the palette).
- **Fallback (Compatibility renderer)**: `GPUParticles2D`, amount 1,048,576, explosiveness 1, a `particles` shader that applies one map step per frame to a position stored in `CUSTOM.xy`, with `COLOR = vec4(1, step, 0, 1)` and a 1 px texture. Render it into a `SubViewport` with HDR 2D enabled and an additive `CanvasItemMaterial`. A second `SubViewport` does the `A·decay + new` feedback.
- **Echo and post**: a third `SubViewport` for P4. `WorldEnvironment` glow intensity 1.3, HDR threshold 0.7.
- **Control**: `InputEventMIDI` for faders and pads, an OSC addon for Resolume clock input, and a community Spout addon or NDI for output.

## Audio reactivity

| band | drives | smoothing | threshold |
|------|--------|-----------|-----------|
| kick, 40–100 Hz onset | flash 0.4; decay dip to 0.65; d += 0.02 | attack 2 ms, release 150 ms (flash), 300 ms (nudge) | onset at +6 dB over the 1 s average |
| bass, 60–250 Hz level | drift r + up to 0.04 | attack 80 ms, release 600 ms | none, continuous |
| mids, 300 Hz–2 kHz | hueShift ± 0.08 | attack 150 ms, release 1 s | none |
| hi-hats, 6–12 kHz | gain +20 % (more mist sparkles in) | attack 1 ms, release 80 ms | gate at −30 dBFS |
| bar / phrase clock | preset morph every 8 bars, family sweep per 32 bars, palette per 16 bars | — | — |

Only the kick is fast. Parameter changes from audio stay under ±0.05, because the attractor's shape is sensitive and big audio jumps make it collapse or flicker. Big changes always come from presets.

## Performance tips

- **P1 cost**: 1 M particles × 16 steps = 16.8 M map evaluations (4 trig each) and 16.8 M atomics per frame. Expect about 1–3 ms on a current desktop GPU. Atomic contention in the brightest seams is the real limit. If it hurts, lower k and raise N.
- **Density budget**: aim for about 4–8 hits per output pixel per frame before decay (`N·k ≈ 4–8 × pixel count`). 1080p: 1 M × 8. 4K: 2 M × 16. Below that, the mist turns to visible speckle. Grain and decay hide some of it.
- **Rasteriser path**: 2–4 M additive sprites cost more than compute (overdraw and blending). Keep sprites at 1–2 px and use the RGBA16F target.
- **Precision**: float32 orbits are fine live. Reusser notes that single precision is not quite adequate for perfect renders and adds a CPU mode for final stills ([notebook](https://rreusser.github.io/notebooks/clifford-and-de-jong-attractors)), so do not expect print-quality smoothness from a 60 fps render.
- **Spout/NDI**: send tone-mapped L and t (8-bit), never raw counts.
- **Odd aspect ratios**: the attractor fits in a roughly square box. On 32:9 strips, either render two instances side by side (second preset, or the same preset mirrored in x), or scale the fit to the long side and let the form crop. Cropping often looks better, because the dense seams fill the strip.
- **Pitfalls**: drift radius above 0.2 (the shape collapses to dots); unaudited preset pairs; auto-exposure and TAA in UE; decay above 0.97 during fast morphs (smear); framing that follows c and d without smoothing (pumping zoom).

## Further references

- [Paul Bourke: Clifford Attractors](https://paulbourke.net/fractals/clifford/): equations, seven parameter sets, the density-histogram method.
- [Paul Bourke: Peter de Jong Attractors](https://paulbourke.net/fractals/peterdejong/): the sister map and nine parameter sets for the family blend.
- [Ricky Reusser: Clifford and de Jong Attractors](https://rreusser.github.io/notebooks/clifford-and-de-jong-attractors): real-time WebGL histogram with colour from step length and log lightness.
- [gumeo: Visualizing Strange Attractors](https://gumeo.github.io/post/visualizing-strange-attractors): closed loops in parameter space for seamless animation.
