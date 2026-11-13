# Moire Interference Layers — design

## Look

The screen is filled edge to edge with very fine, even grey-white lines on near black, so fine that up close they read as a flat grey haze. Over it, wide soft bands of light and dark swim slowly through the frame: fans, curved ribs, ghost circles and rosettes that are not drawn in either layer but come out of the overlap. A tiny twist of one layer sweeps the whole band pattern across the wall; on a kick the bands snap tighter and relax. In colour mode the fine carrier stays pale while the large bands pick up a cold cyan-to-navy ramp, and in the chromatic variant R, G and B gratings drift apart into rainbow fringes. References: the Penrose acetate demo and Oster's 1960s line-screen art (both sourced in research.md) and Michael Bach's drifting ring arrays ([michaelbach.de](https://michaelbach.de/ot/lum-moire1/)). The mood is hypnotic Op Art: calm, exact, slightly dizzying, with large global motion from small inputs.

The main design rule: **all moire is computed, none is sampled.** Every grating is band-limited to the pixel grid it is drawn on, so the only interference the audience sees is the one between our layers, not between our layers and the LED pixels, the LED processor's scaler or a broadcast camera.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #050506 | Normal | 100 % | slightly above black so the wall does not look dead in dark bands |
| 2 | Grating A (reference) | band-limited sine grating: lines, rings or dot lattice, period p px | (inside shader) | — | moves only with the common rotation; the "fixed acetate" |
| 3 | Grating B (twisted) | same pattern, rotated by twist, scaled by detune, optional wave warp | Multiply with A (inside shader) | — | carries all beat motion |
| 4 | Grating C (optional) | rings from a moving centre (LFO, MIDI note or tracked point) | Multiply or Difference with A·B (inside shader) | 0–100 % | adds rosettes; off by default |
| 5 | Band field | analytic difference phase of A and B, the moire envelope alone, no fine carrier | (inside shader) | — | used for colour mapping and the camera-safe mode |
| 6 | Carrier / band mix | crossfade carrier product (2–4) against band field (5) | (inside shader, `carrier` parameter) | — | 1 = full fine lines, 0 = smooth bands only |
| 7 | Colour map | band field through a two-tone ramp, carrier stays neutral | (inside shader) | — | see Color scheme |
| 8 | Ghost feedback (optional) | previous output × 0.70 + current | Lighten | 40 % | half resolution; only the band field goes in, never the carrier |
| 9 | Video matte (optional) | output used as luma key or Multiply over a video layer | Multiply | 100 % | research variation "multiply it over other material" |
| 10 | Post | gentle bloom on bands only, vignette | see compositing | — | no sharpen, no grain above 2 %, no chromatic aberration on the carrier |

Layers 2–7 live in one single-pass fragment shader (P0). This is deliberate: it keeps the Wire build to one ISF node and avoids blending fine lines through any resampling step.

## Techniques

**1. Grating as a phase field**
1. Work in output pixels, centred: `p = fragCoord - 0.5 * resolution`. Using pixels instead of 0..1 UVs keeps the line period identical on 16:9, 32:9 or square walls.
2. Each layer i has a transform: rotation θi, scale si, offset oi. `q = rot(θi) * (p - oi) / si`.
3. Phase in cycles: lines `φ = q.x / period`; rings `φ = length(q - c) / period`; dot lattice uses two line phases (0° and 90°, or 0°/60°/120° for hex as in [Moire Explore](https://nateroe.github.io/moire/index.html)).
4. Brightness of a grating: `g = 0.5 + 0.5 * cos(2π φ)`. Sine, not hard bars: a square wave has harmonics at 3×, 5×, 7× the frequency, and those are the first to alias.
5. Optional thickness: `g = smoothstep(1 - t - e, 1 - t + e, g)` with t = line thickness 0..1 and e the edge softness from technique 3, so thin lines stay band-limited.

**2. Interference (the actual moire)**
1. Combine gratings like layered transparencies: Multiply `L = gA * gB` (light passes both, as in [Michael Bach](https://michaelbach.de/ot/lum-moire1/)). Alternatives: Difference `abs(gA - gB)`, soft XOR `gA + gB - 2 gA gB`. Blend options mirror those of [Moire Explore](https://nateroe.github.io/moire/index.html).
2. Why it works: `cos a · cos b = ½ cos(a − b) + ½ cos(a + b)`. The difference term is the slow band; the sum term is a finer carrier at about twice the line frequency ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)).
3. Band spacing for a twist α between equal gratings: `D = p / (2 sin(α/2)) ≈ p/α` ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)). Example: p = 10 px, α = 1.5° gives D ≈ 382 px; α = 1.8° gives D ≈ 318 px. That is the kick lever.
4. Band spacing for a detune (pitch p vs p + δp, no twist): bands repeat every `p(p + δp)/δp`. p = 10 px, δp = 0.3 px gives about 343 px. This is the bass lever.
5. Wave warp (organic bands): add to grating B's phase `φ += Σ ak · sin(2π fk · dot(q, dk) + ωk t)` with 2–4 waves, amplitudes 0–1.5 cycles, wavelengths 300–1500 px. Moire Explore offers up to 15 such waves; 2–4 are enough on stage.

**3. Anti-aliasing: keep the moire intentional**
1. Filter width per pixel: `w = fwidth(φ)` in cycles per pixel. For unwarped lines and rings it is exactly `1/(period·s)`, so you can pass it as a uniform and avoid derivatives.
2. Band-limit each cosine by its box-filtered average: `cos(2πφ) · sinc(π w)` where `sinc(x) = sin(x)/x`. This is the filtered-cosine idea from [Inigo Quilez: band limiting](https://iquilezles.org/articles/bandlimiting/). In practice, fade the cosine's contrast to 0 as w goes from 0.25 to 0.5 cycles/px (period 4 px down to 2 px), so a grating that is too fine turns into flat grey instead of false bands.
3. Hard rule: **minimum period 4 output px** for any single grating. The Multiply sum term then sits at 2 px, which is the Nyquist limit and still clean. Below 4 px the sum term aliases into slow fake bands that you did not design.
4. For hard-edged lines or dots, use an analytically box-filtered grid instead of a sine, as in [Inigo Quilez: filterable procedurals](https://iquilezles.org/articles/filterableprocedurals/) (filtered grid, squares, checkers and XOR, each using the pixel footprint from derivatives).
5. Band field (layer 5): for any two layers, `band = 0.5 + 0.5 cos(2π (φA − φB))`. This is the moire envelope computed directly. It has no fine detail, never aliases, and survives camera capture and any scaler. The `carrier` fader crossfades `L` toward `band`.

**4. Temporal aliasing (wagon-wheel)**
1. A pixel at radius r from the rotation centre moves `r · Δθ` per frame. If that approaches half a period, the carrier strobes or runs backwards.
2. Limit: `r_max · Δθ < p/4` per frame. For a 3840 px wide wall (r_max ≈ 2200 px from centre to corner) and p = 10 px: Δθ < 0.065° per frame, about 3.9°/s at 60 fps.
3. So absolute rotation stays slow (common rotation 0–3°/s). The audience reads the relative twist, which can move fast because band motion is not limited by the carrier.
4. Kick impulses on twist are eased over 80–120 ms (5–7 frames), never a single-frame step. A phase jump of exactly half a period (bands invert) is the one allowed hard cut.

**5. Beat lock and resolve**
1. When twist and detune are both 0, A and B coincide: the bands open to infinity and the screen becomes an even carrier. That is the "resolve".
2. Per bar: twist eases from 0 at the downbeat to its target by beat 2, holds, and returns toward 0 by the next downbeat (cosine ease). The pattern blooms out of a flat field and collapses back on every bar.
3. The research idea of one turn per bar against a 2:3 ratio is mapped to **relative** phase, not absolute rotation: grating B's phase offset runs 2 cycles per bar, grating C's 3 cycles per bar, so their pattern realigns every bar without fast rotation.

## Shaders

Render order per frame: P0 interference, P1 ghost feedback (optional), P2 composite and post.

**P0 — Interference pass (single pass, the whole look)**
- Purpose: draw all gratings, combine, map colour.
- Inputs (uniforms): resolution, time, twist (deg), detune (%), period (px), thickness, warp amount, warp phase, mode (0 lines, 1 rings, 2 dots, 3 hex), combine (0 Multiply, 1 Difference, 2 XOR), carrier (0–1), ring centre C (xy, px), layerC amount, palette colours, beatPhase (0–1 within bar).
- Per pixel: centre coordinates in px; for A, B (and C) build q, phase φ, filter width w; band-limit each cosine (technique 3); apply thickness; combine to L; compute band field from φA − φB; `out = mix(band, L, carrier)`; colour map (carrier part neutral, band part through ramp); clamp peak to 0.9.
- Output: RGBA8 at the **native pixel resolution of the LED wall** (or projector). Never render at a different size and scale.

**P1 — Ghost feedback (optional, ping-pong)**
- Inputs: previous P1 buffer, the band field from P0 (written to a second output or recomputed at half res), decay 0.70.
- Per pixel: `G = max(G_prev · decay, band)`.
- Output: RGBA16F at half resolution, two buffers swapped. Bands are soft, so half res is invisible; the carrier must never pass through this buffer because downsampling fine lines is resampling, which creates exactly the unwanted moire.

**P2 — Composite and post**
- Inputs: P0, P1, optional video texture.
- Per pixel: background; P0; Lighten with P1 × 0.4; optional Multiply over video. Bloom is computed from the band field only (threshold 0.8, intensity 0.6, large radius); vignette 0.2.
- Output: RGBA8 at native resolution, to Spout/NDI or straight to the LED processor.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base / black | #050506 | background, darkest band |
| carrier | #E8E8E2 | fine lines, warm neutral grey-white |
| band shadow | #0B1A2E | dark end of the band ramp, deep navy |
| band accent | #00B8F0 | middle of the band ramp, cyan |
| highlight | #F4F7FF | band peaks, kick flash |
| alt accent 1 | #FF3D7F | phrase palette swap, magenta |
| alt accent 2 | #FFB000 | phrase palette swap, amber |

- Mapping: two-segment gradient ramp on the band field: 0 → #0B1A2E, 0.5 → accent, 1 → #F4F7FF. The carrier is multiplied in as neutral luminance, so colour lives only at band scale. Coloured fine lines would add chroma aliasing on LED subpixel layouts and on cameras.
- Black-and-white mode (default intro): ramp collapsed to #050506 → #E8E8E2. Plain Op Art.
- Chromatic variant (research variation 5): three gratings B_R, B_G, B_B with twist offsets −0.15°, 0°, +0.15° against a shared A; each drives one channel. Bands split into rainbow fringes while the carrier stays roughly white.
- Over time: accent swaps per 16-bar phrase (cyan → magenta → amber). Swap on the downbeat at the "resolve" moment (technique 5), when the field is flattest, so the cut is invisible.
- Music: bass level pushes the ramp midpoint (more accent, less navy) from 0.5 to 0.35.
- LED walls: keep black at #050506 and peaks at 90 %. 50 % grey haze from a fine carrier pulls a lot of power and looks flat, so on breaks lower `carrier` and let the dark bands reach true base.

## Blending and compositing

- **Multiply between gratings** is the physical model (two transparencies). It is the default because it gives dark bands on a lit field, which reads best at a distance. **Difference** gives bright bands on dark and suits dark clubs. **XOR** gives harsher checker-like rosettes; use it for hi-hat accents.
- All combining happens inside P0 with full-precision floats. Do not build the moire by stacking two Arena layers that each contain fine lines if any layer is transformed or scaled by the mixer: every resample of a fine grating is an uncontrolled moire of its own.
- **Ghost feedback**: Lighten at 40 %, decay 0.70 (range 0.5–0.85), band field only.
- **Masks**: optional circular mask (radius 45 % of the short side, 60 px soft edge) for a "lens" look on wide walls; feed it as a soft matte, not a hard edge (hard edges at 1 px on rotating content also crawl).
- **Post chain in order**: band-only bloom (threshold 0.8, intensity 0.6) → vignette 0.2 → output clamp. Leave out sharpen (adds harmonics → aliasing), grain above 2 % (adds pixel-scale noise that beats against the carrier), chromatic aberration on the carrier, and any output-level scaling or keystone.
- **As a matte for video**: send the band field (carrier = 0) as luma key or Multiply over a video layer. The video gets swimming dark bands without fine lines, which keeps video plus moire safe on camera.

## Motion and timing

- **Common rotation**: 0–3°/s for both layers together, sine-eased, direction reversed per phrase. Hard ceiling from technique 4.
- **Twist (relative)**: 0.3°–6°. Default 1.5° (bands ~380 px at p = 10 px). Larger twist = denser, busier bands.
- **Kick**: twist +0.3° impulse, attack 100 ms, release 400 ms. The whole band pattern jumps in and relaxes.
- **Bar**: lock-and-resolve cycle (technique 5). Twist returns to near 0 on each downbeat.
- **Beat (quarter)**: grating B phase offset steps by 0.25 period, eased over 60 ms. Bands walk across the screen in quarter steps.
- **Phrase (16 bars)**: switch mode (lines → rings → dots → hex) and accent colour on the resolve.
- **Build (8 bars)**: period shrinks from 14 px to the floor (4 px, or 8 px in camera mode); warp rises 0 → 1; carrier rises to 1; twist range widens. Dense, nervous, shimmering.
- **Drop**: hard cut to rings mode, combine Difference, period back to 12 px, twist 3°, ring centre C jumps to a new LFO path. Accent flash to #F4F7FF on the band field for 80 ms.
- **Break**: carrier down to 0.2, only soft bands, slow detune drift (0.05 px/s), common rotation stopped. Rings from tracked points (research variation 4): KeyStep notes place up to 3 ring centres, held chords build rosettes.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|----------------------|----------------------------|
| twist | 0–6° | 1.5° | relative rotation of B; small moves sweep the whole band pattern | KeyStep knob, fine resolution (14-bit or 0.02° steps); kick adds +0.3° |
| detune | 0–8 % | 0 % | pitch difference A vs B; bands widen/tighten without rotation | bass envelope, plus a fader |
| period | 4–32 px (floor 8 px in camera mode) | 10 px | line spacing; small = fine shimmer, large = bold graphic | fader; automate down during builds |
| carrier | 0–1 | 0.8 | 1 = fine lines visible, 0 = smooth band field only (camera-safe) | fader; drop to 0.3 when broadcast cameras are live |
| warp | 0–1 | 0.15 | sine distortion of B; straight fans become organic ripples | mid-band envelope, or knob |
| mode / combine | lines, rings, dots, hex × Multiply, Difference, XOR | lines + Multiply | changes the family of ghost shapes | BeatStep Pro pads; quantise changes to the bar |

## Build recipes

### Resolume Arena

- **Main source**: the P0 shader as an ISF source. Resolume documents ISF only inside Wire ([Resolume ISF support](https://resolume.com/support/en/isf)), so build it as a Wire patch with one ISF node and export the patch to Arena as a source. Expose twist, detune, period, carrier, warp, mode, combine as parameters.
- **Layer 1**: Solid Color #050506. **Layer 2**: the Wire moire source, blend Add or Normal, 100 %.
- **Clip transform**: leave Scale at 100 %, Rotation at 0 and Position at 0 on the moire layer. Do rotation inside the shader. Arena's transforms resample the image, which turns fine lines into accidental moire.
- **Composition size** = LED wall pixel map (for example 3456 × 1152), and the Advanced Output slice mapped 1:1 with no scaling. Same rule on the LED processor side: scaling off.
- **Animation**: twist and phase via Arena's parameter animation with BPM sync (one cycle per bar for the resolve); kick impulse from Arena's audio analysis on the low band with a short fall-off.
- **Video matte**: Layer 3 holds video; put the moire source above it in Multiply with carrier = 0 (bands only).
- **Effects**: at most a gentle Bloom on the composition with high threshold, and only if the carrier is low. Skip sharpen and blur-style effects on the moire layer.
- **No Wire available**: two Arena layers of a pre-rendered fine-line loop at exactly 1:1 pixels, top layer in Multiply, rotated by a fraction of a degree. This works but the rotation resamples and aliases; use periods of 8 px or more and expect some crawl.

### Resolume Wire

- `Time` and `BPM` / beat phase nodes → `Math` (cosine ease) → twist envelope; kick from `Audio Analysis` low band → `Envelope` (attack 100 ms, release 400 ms) → add to twist.
- `Audio Analysis` bass level → `Smooth` (attack 50 ms, release 500 ms) → detune. Mid level → warp.
- `Counter` on bars (wrap 16) → phrase index → `Array` lookup for mode, combine and accent colour.
- ISF node with the full P0 shader: inputs twist, detune, period, thickness, carrier, warp, warpPhase, mode (as float or event steps), combine, ringCentre (2 floats), palette colours (color inputs). Resolume's page states Wire supports ISF as of version 7.22 and does not support vertex shaders; P0 is fragment-only, so this fits ([Resolume ISF support](https://resolume.com/support/en/isf)).
- Keep the ISF single-pass. Resolume's page does not cover multipass ISF, so do the optional ghost feedback with Wire's own feedback node on a second ISF output of the band field, not inside the ISF file.
- Output node at the composition resolution. Exposed parameters: twist, detune, period, carrier, warp, mode/combine.

### Unreal Engine 5.7

- **Material**: post-process material (Blendable Location: After Tonemapping, so tonemapping and bloom do not touch the carrier) or an unlit material on a full-screen plane rendered by a Scene Capture. Build P0 with ScreenPosition × ViewSize for pixel coordinates, `CustomRotator` or a 2×2 rotation built from Sin/Cos for each layer, `Cos` for gratings, and a Custom node for the band-limit factor (sin(πw)/(πw) with w = 1/period).
- **Parameters**: a Material Parameter Collection `MPC_Moire` with Twist, Detune, Period, Carrier, Warp, Mode, Combine, RingCentre, Accent. Drive from Blueprint (beat clock, envelopes).
- **Niagara (variation 4)**: a small CPU system with 1–3 particles whose positions follow MIDI notes or LFOs; write their positions into `MPC_Moire.RingCentre` via a Blueprint each tick. Niagara is only a control source here; no particles are rendered.
- **Post-process volume**: Bloom off or intensity 0.3 with threshold 2; Exposure manual; Anti-aliasing **off** for this output or TSR disabled for the view, because temporal AA jitters and resolves sub-pixel lines and smears or crawls the carrier; Screen Percentage 100. Motion blur off.
- **Remote Control**: preset exposing the MPC scalars; OSC and MIDI protocol bindings for twist (KeyStep knob), carrier (fader), mode (BeatStep pads); kick envelope via OSC from Resolume.
- **Output**: render at the LED processor's native resolution; Spout via a third-party plugin (for example Off World Live's toolkit) or NDI via the NDI SDK for Unreal.

### Godot

- **Main**: a `ColorRect` filling the viewport with a `canvas_item` shader. Use `FRAGCOORD.xy` for pixel coordinates; uniforms `twist_deg`, `detune`, `period_px`, `carrier`, `warp`, `mode` (int), `combine` (int), `ring_centre` (vec2), `accent` (`source_color`), `beat_phase`, each with `hint_range`. Band-limit with `w = 1.0 / period_px` (exact) or `fwidth(phase)` when warped.
- **Window/project**: viewport size = wall resolution, stretch mode `disabled` (or `canvas_items` with integer scale only), MSAA and FXAA off for this viewport (they do nothing for a shader-drawn pattern and FXAA blurs the carrier unevenly).
- **Ghost feedback**: a `SubViewport` at half size rendering a second `ColorRect` that outputs only the band field; its clear mode set to never, with a full-screen black `ColorRect` at alpha 0.30 drawn first each frame (decay ≈ 0.70). Display it via a `TextureRect` with `ViewportTexture`, blend mode Add or a Lighten shader at 40 %.
- **Control**: `InputEventMIDI` for knobs and pads; for audio, `AudioEffectSpectrumAnalyzer` on the bus, read low/mid bands per frame and smooth in script.

## Audio reactivity

| band | drives | smoothing | threshold |
|------|--------|-----------|-----------|
| kick, 40–100 Hz onset | twist impulse +0.3° | attack 100 ms (never shorter, see technique 4), release 400 ms | onset at +6 dB over the 1 s average |
| bass, 60–250 Hz level | detune 0–6 %, ramp midpoint 0.5 → 0.35 | attack 50 ms, release 500 ms | none, continuous |
| mid, 300 Hz–2 kHz energy | warp 0.05–0.6 | attack 150 ms, release 1 s | none |
| hi-hat, 6–12 kHz onset | B phase jump of exactly 0.5 period (bands invert) or combine flips to XOR for 1/16 note | instant step, hold 1/16 note | gate at −30 dBFS; max one jump per 1/8 note |
| full-band loudness | carrier 0.4 → 1.0 | attack 300 ms, release 2 s | none |

Because band spacing goes as p/α, the audio depth must be small: full kick depth is a fraction of a degree. Start low; a too-deep mapping turns into constant full-screen flicker.

## Performance tips

- **Cost**: P0 is a few cosines and rotations per layer, roughly 30–50 ALU per pixel for 3 layers. Trivial at 4K and fine at 8K. The optional feedback at half res is the only buffer.
- **Resolution is the whole game**: render at the native pixel map. A 1920 × 1080 render scaled by the LED processor onto a 1728 × 972 wall resamples every fine line and adds a second, unplanned moire with the scaler grid. If scaling is unavoidable, raise the period floor to 8 px or more (measured after scaling) or lower `carrier`.
- **LED pixel pitch**: the shader period is in wall pixels, so it is pitch-independent once mapped 1:1. On coarse walls (P3.9 and above) viewed from close, a 4 px period is 16 mm per line pair and reads as stripes, not haze; use 6–10 px there so the bands still dominate.
- **Camera recording and streaming**: the camera sensor samples the LED grid, the LED grid samples our grating, and the encoder adds its own compression; fine carriers collide with all three. When cameras are live: carrier ≤ 0.3 or period ≥ 8 px, no lines at exactly 0° or 90° (rotate the common frame by 3–7° so carrier lines do not align with the LED rows), and do a test shot through the actual camera and lens at the planned framing before the show. Wikipedia lists photographing screens as a classic source of unwanted moire ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)).
- **Odd aspect ratios**: pixel-unit coordinates keep the period constant; put the rotation centre at the middle of the wall, but for very wide strips (32:9 and wider) r_max grows, so lower the common rotation ceiling (technique 4) proportionally.
- **Projectors**: keystone and warp correction resample the image. Do geometric correction in the media server only if the carrier is low, or accept a larger period.
- **Pitfalls**: period below 4 px (fake bands from the sum term); fast absolute rotation (wagon-wheel); single-frame twist jumps; TSR/TAA or FXAA in the output path; Arena clip transforms on the moire layer; coloured fine lines on LED subpixels; bloom over a full-brightness carrier (washes everything to grey).

## Further references

- [Moire Explore](https://nateroe.github.io/moire/index.html) — interactive tool with 2–8 rectangular or hexagonal grids, ten blend modes, up to 15 wave distortions and two-keyframe animation; a good parameter model and a way to audition patterns before building.
- [Inigo Quilez: band limiting](https://iquilezles.org/articles/bandlimiting/) — filtering a cosine over the pixel footprint (multiply by a sinc), the core of the anti-aliasing step.
- [Inigo Quilez: filterable procedurals](https://iquilezles.org/articles/filterableprocedurals/) — analytically box-filtered grid, squares, checkers and XOR patterns for hard-edged layers.
- [Michael Bach: Moire patterns](https://michaelbach.de/ot/lum-moire1/) — ring-on-ring demo with slightly different X/Y drift, the model for the slow ring mode.
- [Wikipedia: Moire pattern](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern) — band-spacing formulas for rotation and pitch difference, plus notes on screen photography and aliasing.
