# Pulsar Ridgeline Waveforms — design

Design scope: a general, live ridgeline display. It borrows the *technique* of Harold D. Craft's 1970 pulse stack (many 1D traces, offset and occluding each other), not any specific published artwork. Do not trace or reproduce the album cover; link to it only ([Scientific American](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/)). All data is generated live from audio or noise.

## Look

A dark field holds 48 to 96 thin horizontal traces stacked from the bottom of the frame to about two thirds up, slightly in perspective, so the stack reads like a hillside seen from below. The edges of each trace are almost flat and only twitch with noise. The centre band rises into a cluster of sharp peaks. Every trace hides the traces behind it, so the stack looks solid, like a mountain range made of wire. A new trace arrives on every beat at the front and pushes the older ones back and up, so a kick visibly travels into the distance as a ridge. The mood is cold, scientific and tense. On the drop the peaks double in height, the stack tilts towards the viewer and the newest rows flash. In breaks it calms to a slow, breathing noise landscape. Default colour is off-white lines on near-black. Alternative schemes are black on paper white and white on a cyan fill (both have historical precedent, see [cococubed](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml)), and a colour-by-age gradient.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|------------|---------|-------|
| 0 | background | solid `#07080A` (or the scheme's background) | Normal | 100 % | Same value as the ridge fill, so the occlusion is invisible as a shape |
| 1 | optional backplate | video, NDI or Spout feed shown *through* the gaps between ridges | Normal | 0–100 % | Only used in "matte" mode, see Blending |
| 2 | ridgeline render | P2 render pass: ring-buffer history drawn as occluding traces | Normal (opaque fill) or Alpha (transparent fill) | 100 % | Core layer |
| 3 | front-row flash | same pass, extra emissive on the newest 1–4 rows | inside P2 (additive term) | 0–100 % | Hi-hat and drop accent |
| 4 | glow | Bloom, threshold 0.7, intensity 0.6, radius 0.4 % of frame height | Add | 60 % | Keep low, thin lines blur to mush quickly |
| 5 | grain | monochrome noise, amount 0.03, changes every frame | Overlay | 100 % | Adds plotter / film texture; skip on very large LED walls |
| 6 | vignette / matte | vertical gradient that darkens the top 15 % of the frame | Multiply | 100 % | Makes the back rows fade into "distance" |

## Techniques

### 1. Row signal generation (what goes into one trace)
1. Each trace is a 1D array of W samples (default W = 512), x in 0..1 across the frame.
2. Three selectable sources:
   - **Pulse** (procedural, default): `h(x) = env(x) · (sum of 2–4 Gaussian bumps + noise)`. Bump centres sit around `x = 0.5 ± spread · rnd`, widths 0.01–0.04, amplitudes random per row 0.2–1.0. Noise is 1D value or simplex noise, frequency 40–120 across the row, amplitude 0.02–0.08.
   - **Scope**: a snapshot of the live audio waveform (mono, last 1024 samples), resampled to W. Take its absolute value or an envelope so it stays positive.
   - **Spectrum**: an FFT snapshot (128–512 bins, log-scaled x), so frequency becomes x.
3. `env(x)` is a window that keeps edges flat and peaks in a centre band: `env = smoothstep(0, b, x) · smoothstep(1, 1 - b, x)` with b = 0.25–0.4, then optionally sharpened with `pow(env, 2)`. This is what gives the "quiet edges, busy centre" look.
4. Scope mode must be triggered like an oscilloscope: search the first rising zero crossing in the buffer and start the snapshot there. Without it, peaks jump around randomly and there is no stable column.
5. Clamp the final value to 0..1.5 so a hot input never punches a row through the top of the frame.

### 2. Ring-buffer history (why rows travel)
1. Store the last N rows in a texture of W × N texels, one row per texel row (R16F or R32F, single channel).
2. Keep an integer `head`. On each step, write the new row to texel row `head`, then `head = (head + 1) mod N`.
3. The renderer reads "row age i" (0 = newest) at texel row `(head - 1 - i) mod N`. Nothing is copied or shifted, so the cost per step is one row write.
4. The step clock is either the beat (1 row per 1/4, 1/8 or 1/16 note) or free-running (1 row every k frames). Beat-locked steps make the peak column line up with the tempo, as the 1.337 s period does in the pulsar data.
5. Row feedback (variation): before writing, mix the new row with the previous one: `new = mix(new, prev, echo)`, echo 0–0.8. A kick then echoes for several rows.

### 3. Ridgeline layout (where each row sits)
1. Row age i maps to depth `d = i / (N - 1)` (0 front, 1 back).
2. Baseline: `y0(i) = bottom + spacing · i`, or with perspective `y0 = bottom + (top - bottom) · (1 - pow(1 - d, persp))`, persp 1.0–1.8, so back rows bunch up.
3. Horizontal perspective: `x' = 0.5 + (x - 0.5) · mix(1, shrink, d)`, shrink 0.7–1.0, so back rows are narrower ("looking up a hillside").
4. Curve top: `yc(i, x) = y0(i) + amp · ampScale(d) · h_i(x)`. amp is the main height in units of row spacing (default 4, so peaks cover about 4 rows behind them). `ampScale(d) = mix(1, 0.6, d)` keeps distant rows smaller.

### 4. Hidden-line occlusion (why it looks solid)
1. The fill of every row is the background colour and reaches from its curve down to the bottom of the frame.
2. Per pixel, walk rows from the front (i = 0) to the back (i = N - 1).
3. For row i: compute `yc` at this pixel's x (inverse of the x perspective, one texture tap). If the pixel is within half a line width of `yc`, it is a line pixel: output line colour and stop. If the pixel is below `yc`, it is inside that row's fill: output fill colour and stop. Otherwise continue to the next row back.
4. If no row claims the pixel, it is sky: output background (or the backplate in matte mode).
5. Line width must be constant on screen even on steep flanks: use the distance to the curve, `dist = |y - yc| / sqrt(1 + (dyc/dx)²)`, with the slope from a second tap at x + 1 texel. Anti-alias with `alpha = 1 - smoothstep(lw/2 - px, lw/2 + px, dist)`, px = one pixel.
6. This is the front-to-back version of the painter's algorithm used by ridgeline plot tools, where later groups are drawn on top with an opaque fill ([R-bloggers](https://www.r-bloggers.com/2024/06/cp-1919-psr-b191921-dataset-2/), [R Graph Gallery](https://r-graph-gallery.com/ridgeline-plot)). Doing it front to back lets the shader exit early.

### 5. Breaking the grid (so it does not get boring)
1. **Tilt**: rotate the row layout around the frame centre by -8° to +8°, eased over 1 bar.
2. **Polar mode**: map x to angle and row depth to radius, `r = r0 + spacing · i + amp · h`. The front row becomes the outer ring, peaks push outward; occlusion is the same walk with "below the curve" replaced by "outside the radius".
3. **Glitch rows**: on a random 1 in 16 steps, duplicate a row 3–6 times, giving a frozen block.
4. **Age colour**: line colour sampled from a ramp by d (see Color scheme).

## Shaders

Render order per frame: P1 (only on step frames) → P2 → P3 → output. P1's history is the only persistent buffer; no full-frame feedback is needed.

### P1 — row write
- **Purpose**: write one new row into the ring buffer.
- **Inputs**: `history` (W × N, R16F, persistent), `head` (int), `source` (0 pulse, 1 scope, 2 spectrum), audio texture (waveform or FFT, 1 × 512) if used, `seed` (per-step random), `noiseAmt`, `spread`, `echo`, `kickPulse` (0..1).
- **Per pixel**: if the texel row ≠ head, copy the old value (only needed in hosts where the pass rewrites the whole buffer, such as ISF). If it is the head row: compute h(x) from the selected source, multiply by env(x), add `kickPulse · Gaussian(x, 0.5, 0.02) · 1.2` (one tall pulse on the kick), mix with the previous row by echo, clamp.
- **Output**: history, W × N. Resolution: W = 512 (256 is enough on small screens, 1024 for 8K-wide walls). N = 64 default, 32–128 range.

### P2 — ridgeline render
- **Purpose**: draw the stack with occlusion.
- **Inputs**: `history`, `head`, `N`, `amp`, `spacing`/`top`/`bottom`, `persp`, `shrink`, `tilt`, `lineWidth` (px), `flash`, colour uniforms (`bg`, `fill`, `line`, `ramp` texture or 3 stops), `mode` (linear, polar), `fillAlpha`.
- **Per pixel**: (1) map pixel to plot space, apply inverse tilt; (2) front-to-back row walk as in Technique 4, max N iterations, early exit; (3) line colour = base line colour or ramp(d), multiplied by `1 + flash · (1 - smoothstep(0, 4, i))` for the front rows; (4) back-row fade: line brightness `· mix(1, 0.35, d)`; (5) fill colour `fill` with alpha `fillAlpha` (1 = solid hidden-line, 0 = see-through lines).
- **Output**: RGBA at output resolution (1920×1080 or wall native). Alpha = 1 on line and solid fill, `fillAlpha` on fill, 0 in sky (for keying).

### P3 — post
- **Purpose**: bloom, grain, vignette.
- **Inputs**: P2 output, `bloomAmt`, `grainAmt`, `time`.
- **Per pixel**: bright-pass at 0.7 → separable Gaussian blur at half resolution (2 passes) → add back × bloomAmt; add grain `(hash(uv, frame) - 0.5) · grainAmt`; multiply by vertical vignette.
- **Output**: final frame. Use the host's built-in bloom where possible instead of writing this pass.

## Color scheme

| scheme | background / fill | line | accent (flash, front rows) | notes |
|--------|-------------------|------|----------------------------|-------|
| Plotter dark (default) | `#07080A` | `#E8E6DF` | `#FFFFFF` | Warm off-white avoids harsh blue LED white |
| Paper | `#EDEAE2` | `#111111` | `#B3261E` | For daytime or bright rooms; reduce wall brightness to 40 % |
| Survey cyan | `#2A8FB8` | `#F4FBFF` | `#FFFFFF` | White on a cyan fill, as in one historical reprint |
| Age ramp | `#05060A` | ramp front → back `#F2F0EA` → `#58C4DD` → `#2B3A8C` → `#1A1030` | `#FF4D6D` on the kick row | Colour by row age |

- Mapping: line colour = single colour, or `ramp(d)` with d = row age 0..1 (3- or 4-stop gradient ramp, linear in sRGB is fine). In spectrum mode optionally colour by x (frequency) instead.
- Over time: the ramp offset slides by 1 row per step so colours stay attached to rows as they travel. A phrase change (every 8 or 16 bars) can swap schemes with a 1-beat crossfade.
- Music: kick row gets the accent colour for its first 2 steps. Drop: accent flashes the front 4 rows.
- LED wall: black level `#07080A`, not pure black, so the panel does not visibly switch off and the grain still reads. Keep line luminance at or below 90 % (`#E8E6DF`), with bloom providing the rest. The image is about 90 % dark, so overall power draw is low; raise contrast rather than brightness.

## Blending and compositing

- **Solid mode (default)**: P2 opaque over background. Fill = background colour, so occlusion is pure shape.
- **Matte mode**: fillAlpha = 0 in the sky and 1 in the fills, lines white. Put the ridgeline layer above a video layer and use it as a mask (Resolume: layer blend "Mask", or a luma key) so footage appears only inside the mountain, or only in the sky (invert).
- **See-through mode**: fillAlpha 0.2–0.5. Back rows show faintly through front rows; looks like a wireframe hologram. Use with Add blend over dark material.
- **Feedback**: none on the full frame. Only row echo inside P1 (echo 0–0.8). A full-frame feedback trail smears the lines and kills the occlusion; avoid.
- **Post chain order**: P2 → bloom (threshold 0.7, intensity 0.6) → grain (0.03) → vignette (top 15 %, Multiply). No chromatic aberration by default; at most 1 px horizontal on drops. No sharpen (lines are already anti-aliased).

## Motion and timing

- **Row step**: 1 row per 1/8 note default (at 128 BPM, 4.3 rows per second, a 64-row stack holds 8 bars). Options 1/4 (calm, 16 bars visible) and 1/16 (fast, 4 bars visible). Free mode: 1 row every 2–6 frames.
- **Smooth scroll**: between steps, shift all baselines by `frac(stepPhase) · spacing` so rows glide instead of jumping; ease linear. The new row fades in over the first step (alpha 0 → 1) at the front.
- **Peak column**: the pulse centre drifts slowly, `centre = 0.5 + 0.08 · sin(2π · bars / 16)`.
- **Tilt and perspective**: change only on bar boundaries, easing in-out cubic over 1 bar.
- **Drop**: amp × 2 over 1 beat (ease-out expo), tilt snaps to +6°, flash = 1 on the front 4 rows, step switches to 1/16 for 4 bars, then back.
- **Break**: step to 1/4, amp × 0.4, source switches to pure noise (no bumps), bloom down, slow tilt drift ±2° over 8 bars.
- **Build-up**: amp ramps 1 → 1.6 and spread narrows 0.15 → 0.03 over the 8 bars before the drop, so the peaks concentrate into one tall column.

## Parameters

| name | range | default | what it does on stage | suggested mapping |
|------|-------|---------|-----------------------|-------------------|
| height | 0–10 rows | 4 | Peak height relative to row spacing; low = calm hills, high = jagged Alps | KeyStep 37 mod strip or a fader; bass envelope adds up to +2 |
| density | 16–128 rows | 64 | Row count; few rows = bold graphic, many = fine texture | BeatStep Pro encoder, stepped |
| rate | 1/4, 1/8, 1/16, free | 1/8 | How fast rows travel back | BPM sync; BeatStep Pro pad to cycle |
| spread | 0.01–0.4 | 0.12 | Width of the peak band; narrow = one pulsar column, wide = full range | Encoder; automate on build-ups |
| tilt | -10° to +10° | 0° | Rotates the stack | Encoder, eased over 1 bar |
| jitter | 0–0.15 | 0.04 | Noise on every line, the "tension" | Hi-hat band envelope |

Secondary (set before the show): line width 1–4 px (default 2 px at 1080p), persp 1.0–1.8 (1.3), shrink 0.7–1.0 (0.85), echo 0–0.8 (0), source, colour scheme, mode (linear / polar).

## Build recipes

### Resolume Arena
- Arena has no built-in ridgeline generator. Build the effect as a custom ISF source (generator) with two passes, using ISF's persistent float buffers ([ISF JSON reference](https://docs.isf.video/ref_json)):
  - Pass 1: target `history`, `PERSISTENT: true`, `FLOAT: true`, WIDTH 512, HEIGHT 64. Implements P1. Because ISF rewrites the whole buffer each frame, every texel copies itself except the head row.
  - Pass 2: no target (final output). Implements P2.
  - Head and step: compute `step = floor(TIME · bpm / 60 · div)` from a float input `bpm` and a `div` long input; `head = step mod N`. Write only when the stored step (keep it in one spare texel, e.g. the last column of row 0 in a 513-wide buffer) differs from the current step.
  - Audio: ISF defines `audio` and `audioFFT` input types ([ISF primer, audio visualizers](https://docs.isf.video/primer_chapter_8)). If your Arena version does not feed them, use source = pulse and drive `height`, `jitter` and `kick` from Arena's parameter audio analysis (the audio-reactive animation option on each parameter), with the beat taken from Arena's BPM.
- Layer setup: Layer 1 solid colour `#07080A` (or skip, the ISF fills its own background). Layer 2 the ISF ridgeline source. On layer 2 add Bloom (low settings as above) and a gentle Noise / grain effect if available. Map `bpm` to the composition BPM via the parameter's BPM sync, and the six stage parameters to the BeatStep Pro via MIDI shortcuts.
- Matte use: put a video clip on layer 3 above, set layer 3's blend mode to Mask / or use the ridgeline layer as a luma key source, as described in Blending.
- Alternative: run the effect in Unreal or TouchDesigner and bring it in as a Spout source.

### Resolume Wire
- Inputs: the composition audio input → an FFT / audio analysis node (band energies for kick 40–120 Hz, bass 60–250 Hz, hats 6–12 kHz) and a waveform buffer if available. A BPM / transport node supplies beat phase. Exact node names depend on the Wire version; check the node browser.
- Envelopes: kick band → envelope follower (attack 5 ms, release 250 ms) → threshold 0.6 → trigger `kickPulse`. Bass → envelope (attack 40 ms, release 400 ms) → scale 0..2 → add to height. Hats → envelope (attack 5 ms, release 80 ms) → jitter.
- Shader: load the same two-pass ISF into Wire's shader / ISF node, connect the parameters above plus beat phase.
- Post: Wire bloom/blur node → Output. Expose height, density, rate, spread, tilt, jitter, scheme as patch parameters so they appear as normal Arena effect parameters.

### Unreal Engine 5.7
- **History**: `RT_History`, Texture Render Target 2D, 512 × 64, format R16f, no mips, filter bilinear in x (clamp addressing), clear never.
- **Row write (Blueprint)**: on each step event (from a beat clock driven by BPM, or a Quartz clock subdivided to 1/8), compute the new row in Blueprint or in a small material `M_RowWrite` (inputs: audio array as a 1 × 512 texture or noise parameters, seed, kick). Draw it with Begin Draw Canvas to Render Target → Draw Material into the rectangle (0, head, 512, 1) → End Draw Canvas. Only one row is touched; the rest persists.
- **Audio**: Audio Synesthesia (Constant-Q or Loudness analyzers) or a Submix spectrum analysis on the master submix for bands; for scope mode, a Submix envelope/buffer listener feeding a float array. Convert to the 1 × 512 texture via a dynamic texture or by passing up to 16 band values as material parameters (enough for pulse mode).
- **Render, route A (2D, simplest)**: material `M_Ridgeline`, domain Post Process or UI, with a Custom node for the front-to-back row walk (P2). Output through a Composure / camera post-process or render to `RT_Out` 1920×1080 with Draw Material to Render Target.
- **Render, route B (3D fins, for camera moves)**: a static mesh of N vertical "fins", each a strip 256 × 2 segments, placed at z = row index · spacing. Material `M_Fin`, opaque, unlit: World Position Offset raises the top vertices by height sampled from `RT_History` at (u, row) (the row index comes from UV1 or vertex colour, with `head` from a Material Parameter Collection). Base colour = fill; emissive line where the fin's local v is within line width of the top edge. The depth buffer does the occlusion for free, and the camera can fly along the ridges or orbit the stack. For thicker lines use Niagara ribbons reading the same render target (one ribbon per row, 256 particles each, position from Sample Texture).
- Post: Post Process Volume, Bloom intensity 0.6, threshold 0.7; Film Grain intensity 0.1; vignette 0.3; Exposure fixed (Manual, min = max) so the dark field does not auto-brighten.
- **Remote Control**: expose `MPC_Ridgeline` scalars `Height`, `Density`, `Spread`, `Tilt`, `Jitter`, `Flash`, `Scheme`, and the step-rate enum on the Blueprint, in a Remote Control Preset bound to OSC and MIDI (BeatStep Pro encoders, KeyStep mod strip for Height). `Head` stays internal.
- **Output**: Spout sender (to Resolume) or NDI for collaborators; send RGBA so the matte mode works on the receiving side.

### Godot
- Godot 4. Audio: put an AudioEffectSpectrumAnalyzer on the music bus for band energies, and an AudioEffectCapture on the same bus for scope mode (read frames each step, pick the trigger point, resample to W).
- History: an `Image` W × N, format FORMAT_RF, wrapped in an `ImageTexture`. On each step the script writes one row (set_pixel per sample, or build the row as a PackedFloat32Array) and calls `update()` on the texture. At 512 × 64 this is cheap on the CPU. The GPU never needs a feedback viewport.
- Render: full-screen ColorRect with a `canvas_item` shader doing P2. Uniforms: `history` (sampler2D, filter_linear, repeat_disable), `head`, `rows`, `height`, `spread` (used only in CPU row generation), `tilt`, `persp`, `shrink`, `line_width`, `flash`, `fill_alpha`, `bg`, `line_col`, `ramp` (GradientTexture1D), `mode`.
- Post: WorldEnvironment with glow enabled (2D HDR on in project settings), intensity 0.6, threshold 0.7; grain as a second ColorRect with a tiny noise shader.
- 3D variant: MeshInstance3D fins with a spatial shader doing the same vertex displacement as the UE route B.
- Beat clock from a script BPM value; external control via an OSC or MIDI add-on (Godot has built-in MIDI input through `OS.open_midi_inputs()` and InputEventMIDI).

## Audio reactivity

| band | drives | smoothing | threshold / scaling |
|------|--------|-----------|---------------------|
| kick, 40–120 Hz | `kickPulse`: one tall peak (1.2 × height) in the newest row; accent colour on that row | attack 5 ms, release 250 ms | trigger at 0.6 of the running peak (2 s window), retrigger hold 100 ms |
| bass, 60–250 Hz | height +0..2 rows | attack 40 ms, release 400 ms | linear above a 0.2 floor |
| low mids, 250–800 Hz | spread +0..0.1 | attack 100 ms, release 600 ms | linear |
| hats, 6–12 kHz | jitter 0.02–0.15 and flash on front 2 rows | attack 5 ms, release 80 ms | flash trigger at 0.5 |
| RMS loudness | bloom 0.4 → 0.9 | attack 300 ms, release 2 s | only above 0.5 |
| waveform / FFT | the row content itself (scope or spectrum source) | spectrum per bin `s = max(new, s · 0.85)` | log x for spectrum, triggered snapshot for scope |

In pulse mode the audio shapes the procedural row; in scope or spectrum mode the audio *is* the row and kick/bass mappings should be halved to avoid double counting.

## Performance tips

- P1 is negligible: one row write per step (512 texels).
- P2 cost scales with rows × pixels in the worst case: up to N texture taps (two for the slope) per pixel. With early exit most pixels in the lower half stop after a few rows, but sky pixels walk all N rows. Rough estimate (not measured): 64 rows at 1920×1080 is cheap on a desktop GPU; 128 rows at 4K can cost several ms. To bound it, skip rows that cannot reach the pixel: a row whose curve top `y0 + amp_max` is below the pixel can never claim it, so start the walk at the first row with `y0(i) + amp_max ≥ y` and note that the walk always ends at the latest at the first row whose baseline is above the pixel (its fill covers it). This cuts the loop to about `amp / spacing + 2` iterations per pixel in linear mode.
- UE route B (fins) costs vertices, not pixels: 64 fins × 256 × 2 segments ≈ 33 k vertices, trivial.
- Line width: specify in pixels of the output, not UV, or lines vanish on 4K walls and turn fat on small ones. On LED walls with a coarse pitch (above 3 mm), use 3–4 px lines and fewer rows (32–48); 1 px lines alias and shimmer.
- Wide or odd aspect ratios: keep W proportional to output width (e.g. 1024 for 5760 px). The ridgeline reads best when the stack is narrower than the screen; on a 3:1 strip use the centre third for the stack, or tile 3 stacks with offset histories (row read offset 0, N/3, 2N/3).
- Scope mode without a trigger is the most common failure: it looks like random noise. Also watch for DC offset in the waveform (use absolute value or remove the mean).
- Do not use mipmaps on the history texture; the filter blends neighbouring rows and peaks bleed into other rows.

## Further references

- [Wilke: Goodbye joyplots](https://clauswilke.com/blog/2017/09/15/goodbye-joyplots/) — why the technique is called ridgeline plot, from the author of ggridges.
- [R Graph Gallery: ridgeline chart](https://r-graph-gallery.com/ridgeline-plot) — overlap scale and fill choices for the general technique.
- [ISF primer, chapter 8: audio visualizers](https://docs.isf.video/primer_chapter_8) and the [ISF JSON reference](https://docs.isf.video/ref_json) — audio / audioFFT inputs and persistent float passes used in the Arena recipe.
- [Timmes, cococubed: CP 1919 page](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml) — history of Craft's 1970 stack and its colour treatments.
