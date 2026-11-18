# Whitney Digital Harmonics — design

## Look

A black field holding a few hundred small, saturated dots. At the start of a cycle all the dots stand in one clean figure: a straight ray from the centre, a single bright point, or a regular polygon. Then they begin to move. Each dot turns at its own speed, and the speeds are whole-number multiples of one base rate (dot 1 at 1x, dot 2 at 2x, dot 3 at 3x and so on). This is Whitney's differential motion from *Digital Harmony* ([book text](https://archive.org/stream/DigitalHarmony_201611/Digital%20Harmony_djvu.txt)). The ray curls into a spiral, the spiral splits into 2, 3, 5 and 7 arms, the arms dissolve into a shimmering rosette cloud, and then the field snaps back into a figure for a moment before it scatters again. The colour follows *Permutations* (1966), which was shot from a black-and-white display and coloured later on an optical printer ([Visual Music Archive](https://visualmusicarchive.org/works/w/permutations.html)). Here that means flat, saturated dye colours on true black, with a slight RGB misregistration. Each dot draws a short comet tail, so you see the whole field turning. The mood is calm, precise and hypnotic. The whole show is tension and release: the dots scatter like a dissonance and then snap into a figure like a chord resolving, timed to land on the downbeat. A second mode follows *Matrix III* (1972): dots run on Lissajous paths that are mirrored across the frame ([Light Cone](https://lightcone.org/en/film-1555-matrix-iii)).

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #000000, optional radial vignette to #05040A at corners | Normal | 100 % | keep true black for LED contrast |
| 2 | Phase clock (hidden) | CPU/Wire/Blueprint logic: global phase φ, snap/resolve state, consonance value c | — | — | pass P0, data only |
| 3 | Harmonic field | analytic per-pixel dots + comet arcs (spiral / ring / RDTD modes), or N-loop for Lissajous mode | Add over 1 | 100 % | pass P1, HDR float |
| 4 | Bloom trails (optional) | feedback buffer, decay 0.90–0.97, zoom 1.000–1.004, rotate 0–0.15°/frame | replaces 3 output | — | pass P2, the *Arabesque*-style "blooming" variant |
| 5 | Tone map + colour | soft clip, consonance whitening, optical-printer RGB offset | — | — | pass P3 |
| 6 | Post | Bloom, grain | Add (bloom) | 100 % | built-in effects |
| 7 | Over footage (optional) | the whole stack over a video layer | Add or Screen, or as luma mask | 50–100 % | see compositing |

## Techniques

**1. Harmonic differential motion (the core)**
1. N dots, index k = 1…N (N = 24–360, default 120). One global phase φ, measured in turns.
2. Angle of dot k: `θ_k = 2π · m · k · φ + θ_rot`. Here m is the voice multiplier (1 by default) and θ_rot is a slow global rotation. Dot k moves k times faster than dot 1. This is the whole-number speed ladder that the book calls harmonic resonance ([Digital Harmony](https://archive.org/stream/DigitalHarmony_201611/Digital%20Harmony_djvu.txt)).
3. When φ = 0 or any whole number, every dot is at θ_rot, so the figure is fully aligned. When φ = p/q (a reduced fraction), the dots fall onto only q distinct angles, so the field shows q arms or a q-gon. Small q gives a strong, simple figure, like a consonant interval: 1/2 gives 2 arms, 1/3 gives 3, 2/5 gives 5. Large q or an irrational φ looks like dissonance: a smeared cloud.
4. Close to a fraction, at φ = p/q + ε, dot k drifts off the clean figure by k·ε turns. The outermost dot drifts by N·ε. The figure reads as "resolved" while N·ε < 0.02 turns. With larger N the resolution moments are sharper and shorter, so N works as a "focus" control.
5. Consonance value: `d = min over q = 1…Q of |φ·q − round(φ·q)| / q`, with Q = 8, then `c = exp(−(N·d / 0.02)²)`. c is 1 at a clean figure and 0 in the cloud. It drives brightness, whitening and an OSC output that other layers can follow.

**2. Placement modes (where dot k sits)**
1. **Spiral arm** (default, the classic Whitney look): `r_k = R · (r0 + (1 − r0) · k / N)`, with r0 = 0.08 and R = 0.46 · min(width, height). At φ = 0 the dots form a single ray. Then the ray winds into a spiral.
2. **Ring**: all dots at radius R. At φ = 0 they collapse to one point, and at p/q they form a regular q-gon. Use it for very hard, graphic resolutions.
3. **Symmetry order S** (1–12): each dot is drawn S times, at θ_k + 2π·s/S. This makes rosettes and keeps the field balanced on wide walls.
4. **RDTD-style radius differential**: add `ρ · sin(2π · m_r · k · φ)` to r_k, with ρ = 0–0.15 R and m_r = 1–4. The radius now also moves harmonically, which gives petal rosettes. This follows the idea behind the name of Whitney-Reed RDTD (radius differential, theta differential) ([Wikipedia](https://en.wikipedia.org/wiki/John_Whitney_(animator))). The exact formulas of that program are not documented in the sources.
5. **Lissajous mode (*Matrix III*)**: `x_k = R · sin(2π · a · k · φ + π/2)`, `y_k = R · sin(2π · b · k · φ)`, with a:b taken from the ratio set. The result is mirrored into four quadrants (`x → |x|`, `y → |y|`, then reflected), because the film applies one Lissajous figure symmetrically over the frame ([Light Cone](https://lightcone.org/en/film-1555-matrix-iii)). Draw small shapes instead of dots: triangle, square or hexagon SDFs of size 0.6–1.2 % of screen height.

**3. Phase drivers (tension and release)**
1. **Breath** (default): `φ(t) = S_bloom · ease(tri(beats / (4 · bars)))`, where tri runs 0 → 1 → 0 over the cycle and ease is a cubic in-out. The field leaves the aligned state, blooms out to φ = S_bloom and comes back, so it is fully aligned again on the cycle downbeat. S_bloom between 0.002 and 0.02 gives gentle spirals only. Between 0.1 and 0.5 the field passes through many p/q figures on the way out and again on the way back.
2. **Sweep**: `φ += ν · dt`, with ν = 0.0005–0.01 turns/s. The figures arrive whenever they arrive, Farey-sequence style. Use it for ambient breaks.
3. **Resolve** (trigger): find the nearest fraction p/q with q ≤ Q_snap (Q_snap = 2–8). Glide φ there over 1 beat with an exponential ease-out, hold for `hold` beats (0–4), then return to the active driver. Fire it on the kick of the drop, on a pad, or automatically on bar 1 of each phrase.
4. **Detune drift**: replace m with m + δ (δ = 0–0.01). Resolutions then precess slightly and never repeat exactly. This is research variation 2.

**4. Speed limit and anti-strobe**
1. The outermost dot moves `N · m · Δφ` turns per frame. Above about 0.1 turns per frame you get wagon-wheel strobing.
2. Clamp dφ per frame to `0.1 / (N · m)` in P0. If the driver asks for more, the motion lags slightly. That is better than aliasing.
3. The analytic comet arcs (technique 5) act as motion blur, so moderate speeds still look smooth at 50 or 60 fps.

**5. Analytic comet arcs (trails without feedback)**
1. The tail of dot k is the arc it travelled in the last τ seconds: from `θ_k(φ(t − τ))` to `θ_k(φ(t))`, with τ = 0.05–1.5 s. P0 passes φ_prev = φ(t − τ) as a uniform, so the tails are frame-rate independent.
2. The arc's angular length is `L_k = 2π · m · k · (φ − φ_prev)`. Brightness along the arc fades as `exp(−3 · s)`, where s = 0 at the head and 1 at the tail end.
3. Energy normalisation, as in a scope beam: tail brightness ∝ `1 / max(1, |L_k| · r_k / dotSize)`. Fast outer dots become faint threads and slow inner dots stay bright.
4. If |L_k| ≥ 2π/S, the tail fills the whole ring. Draw it as a faint full ring at the normalised brightness. Concentric rings then appear in the cloud phase, which is a nice side effect.

**6. Chord mapper (multi-voice)**
1. Up to 3 voices j, each with its own multiplier m_j, colour and radial band (voice 1: 0.08–0.45 R, voice 2: 0.45–0.75 R, voice 3: 0.75–1.0 R), all sharing the same φ.
2. Interval to ratio: octave 1:2, fifth 2:3, fourth 3:4, major third 4:5, minor third 5:6. These are the low-order Pythagorean ratios Whitney carries into motion ([Digital Harmony](https://archive.org/stream/DigitalHarmony_201611/Digital%20Harmony_djvu.txt)). A held KeyStep chord sets m_j from the intervals above the lowest note, so a major triad gives m = 4, 5, 6 and a minor triad gives 10, 12, 15.
3. All voices align together only at whole-number φ. Partial alignments appear where each voice's own figure resolves, so major and minor chords show visibly different interference figures. Lower the bloom S_bloom when the multipliers are large, because the speed limit (technique 4) then scales with the largest m.

## Shaders

**P0 — Phase clock** (CPU / Wire logic / Blueprint, not a shader)
- Inputs: BPM, beat position, driver mode, S_bloom, ν, resolve trigger, Q_snap, hold, detune, kick envelope.
- Per frame: advance φ with the active driver; apply the snap glide when triggered; clamp dφ (technique 4); store φ_prev from a 2-second ring buffer of φ samples, read at t − τ; compute c (technique 1.5).
- Output: uniforms φ, φ_prev, c, voice multipliers m_1…m_3, plus c as an OSC/MIDI CC for other layers.

**P1 — Harmonic field** (fragment, RGBA16F, 1.0× output resolution; 0.75× on weak GPUs)
- Inputs: φ, φ_prev, N, mode, S (symmetry), m_j, R, r0, ρ, m_r, dotSize σ (0.25–0.8 % of screen height), tail τ, palette uniforms, c, pulse.
- Per pixel, spiral or ring mode, with O(1) cost per voice:
  1. `p = (fragCoord − centre) / min(width, height)`, aspect-safe. Polar `(ρp, α)`.
  2. Spiral: continuous index `k* = (ρp / R − r0) / (1 − r0) · N`. Test only `k = floor(k*) − 1 … floor(k*) + 2`, clamped to 1…N. Ring mode: all dots share one radius, so test the dot whose angle is nearest. Because θ_k = 2π m k φ, solve `k ≈ (α − θ_rot) / (2π m φ) + j / (m φ)` for the few j that land in 1…N. Above N = 200 in ring mode, switch to the N-loop or sprites.
  3. For each candidate k: `δ = wrap(α − θ_k)` into [−π/S, π/S] (symmetry fold). Distance `d² = (ρp − r_k)² + (r_k · δ)²`. Head: `exp(−d² / 2σ²) · pulseGain`.
  4. Tail: if δ lies on the trailing side within |L_k|, add `tailGain · exp(−3s) · exp(−(ρp − r_k)² / 2(0.6σ)²) · norm_k` (technique 5).
  5. Colour per dot from the palette by `u = k / N` (see Color scheme), multiplied by intensity, then summed over candidates and voices.
- Lissajous mode: loop over k = 1…N (N ≤ 128 in ISF) with the quadrant mirror applied to p first, so a quarter of the work covers the whole frame. Shape SDF per dot: triangle, square or hexagon chosen by k mod 3.
- Output: linear HDR colour (values above 1 allowed).

**P2 — Bloom trails** (optional, ping-pong pair, RGBA16F, 0.5× resolution)
- Inputs: P1, previous P2.
- Per pixel: sample the previous frame at `uv' = centre + rotate(uv − centre, −rot) / zoom`, multiply by decay (0.90–0.97), add P1 · 0.6. Use 16-bit float, because 8-bit buffers leave a grey residue that never fades to black.
- Output: the trail buffer, upscaled bilinear and added under P1 in P3. Use for *Arabesque*-like blooming. Keep it off for the crisp *Permutations* look.

**P3 — Tone map and colour** (fragment, output resolution)
- Inputs: P1 (+P2), c, misregistration amount.
- Per pixel: (1) optical-printer offset: read R at uv + o, G at uv, B at uv − o, with o = 0–2 px along a fixed diagonal; (2) soft clip `col = 1 − exp(−col · exposure)` with exposure 1.2–2.5; (3) consonance whitening `col = mix(col, highlight, 0.35 · c · luma(col))`; (4) black clamp: subtract 0.01, then clamp to ≥ 0.
- Output: LDR to the post chain.

Render order: P0 → P1 → (P2 reads P1 and its own previous frame) → P3 → Bloom → grain → output.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base | #000000 | field |
| shadow | #05040A | corner vignette only |
| accent 1 | #FF2E88 | magenta, inner dots / voice 1 |
| accent 2 | #FFB000 | amber, middle / voice 2 |
| accent 3 | #19D3FF | cyan, outer / voice 3 |
| accent 4 | #7CFF4F | green, palette option |
| accent 5 | #8A5CFF | violet, palette option |
| highlight | #FFF6E0 | warm white for resolution flashes |

- **Mapping**: gradient ramp by dot index `u = k / N` through magenta → amber → cyan (default "Printer" palette). A spiral then shows a rainbow along each arm, and an aligned ray reads as one continuous gradient. Alternative "Dye" palette: hue stepped by `k mod 3` (magenta, amber, cyan), closer to the *Permutations* dot colours. In chord mode each voice gets one solid accent.
- **Over time**: rotate the ramp offset by +1/3 per phrase (8 bars) so the colours cycle without drifting continuously. On a resolution, c whitens the cores toward #FFF6E0, so consonance reads as light.
- **Music**: hi-hat onsets step the ramp offset by 0.02 (a brief colour shimmer). On the drop, swap the palette.
- **LED walls**: true black (no lift), saturation at 85–95 % because fully saturated magenta and cyan clip harshly on many LED processors, and peak luma about 0.9 before bloom. The contrast lives in the dots, not the field.

## Blending and compositing

- Dots and tails are summed additively inside P1. Overlaps at the resolution moments therefore get brighter, which is exactly when they should.
- Layer 3 over the background: Add. Over footage: Add for light-on-image, or Screen at 70 % if the footage is bright.
- As a matte: send the luma of P3 (before colour) as the mask of a footage layer, so the video shows only through the dots and arcs. The figure then reveals the image at each resolution. In Arena the quick version is the Whitney layer above the footage in Multiply. For a clean matte, do the masking inside the Wire patch, with the footage as a second texture input.
- Feedback mix (P2): 0.6 of P1 added into the loop, decay 0.93 default. Never above 0.97 on LED walls: the trails smear into a grey haze.
- Post chain in order: optical-printer RGB offset (in P3) → soft clip → Bloom (threshold 0.6, intensity 0.8–1.4, radius medium) → grain 2–3 % monochrome → output. No sharpen, because it rings around small dots. No extra chromatic aberration beyond the 0–2 px offset.

## Motion and timing

- **Cycle length**: Breath cycle = 1, 2 or 4 bars (default 2 bars). Full alignment always lands on the cycle downbeat.
- **Global rotation** θ_rot: 1 turn per 16–64 bars. It is barely noticeable but keeps the aligned ray from always pointing the same way.
- **Kick**: dot size pulse +40 %, attack 5 ms, release 150 ms. It does not move φ, so the harmonic logic stays clean.
- **Build-up**: raise S_bloom from 0.01 to 0.3 over 8–16 bars. The field passes more and more complex figures and ends in the cloud. Raise N step by step (60 → 240) to sharpen the moments.
- **Drop**: fire Resolve with Q_snap = 1 (snap to full alignment) on the first downbeat, hold 1 beat, then jump the palette and switch to Breath with a 1-bar cycle at S_bloom 0.05. Tight, fast blooms.
- **Break**: switch to Sweep, ν = 0.001, tail τ 1.2 s, P2 on at decay 0.95. Slow, drifting rosettes.
- **Phrase**: every 8 bars, step the ratio set or symmetry order (S: 1 → 3 → 5 → 6). Ease every parameter jump over 1 beat, except snaps.

## Parameters

| name | range | default | what it does on stage | suggested mapping |
|------|-------|---------|-----------------------|-------------------|
| bloom | 0.002–0.5 | 0.03 | how far the field travels away from alignment per cycle (S_bloom) | BeatStep Pro knob 1, or ramp over a build-up |
| focus | 24–360 dots | 120 | dot count N; higher means sharper and shorter resolutions | knob 2, stepped (24, 60, 120, 240, 360) |
| ratio | octave / fifth / fourth / M3 / m3 / chord | octave | voice multipliers; chord mode reads KeyStep notes | KeyStep 37 held notes (chord mode) or pads 1–5 |
| symmetry | 1–12 | 1 | rosette order S | knob 3, stepped; step per phrase |
| tail | 0.05–1.5 s | 0.3 s | comet arc length τ | knob 4; long on breaks |
| resolve | trigger | — | snap to the nearest clean figure, hold, release | BeatStep Pro pad, or auto on bar 1 of a phrase |
| cycle | 1 / 2 / 4 bars | 2 | Breath cycle length, BPM-locked | knob 5 or BPM clock divider |

## Build recipes

### Resolume Arena

- Arena has no built-in harmonic dot generator, so the field is a custom ISF source built in Wire (see below) on layer 2, blended Add over a black layer 1. Resolume documents ISF in the context of Wire ([Resolume ISF](https://resolume.com/support/en/isf)), so test loading in your Arena version before the gig.
- Parameter control: link `bloom` to the clip's BPM-synced parameter animation in "ramp" style over 8 or 16 bars for build-ups. Map `resolve` to a MIDI pad. Link the size pulse to Arena's audio analysis (Low band) with its attack/release controls.
- Post on layer 2: Bloom (in Resolume since 7.2 ([Resolume blog](https://resolume.com/blog/19880))), then a light grain effect from your set. For trails without the P2 shader, Resolume's FeedbackPro (Arena 7.18.2+) with scale 100.2 % and a small hue shift gives a similar bloom ([Resolume blog](https://www.resolume.com/blog/23113)).
- External route: run the UE or Godot build and bring it in as a Spout source on layer 2 (Add). Send c back via OSC to Arena to drive, for example, the opacity of a footage layer.
- Matte use: footage on layer 1, Whitney layer on layer 2 in Multiply for a quick version. For a true luma matte, build it into the Wire patch with the footage as a texture input.

### Resolume Wire

- **Clock (P0)**: `BPM` / beat-position node → divide by (4 · bars) → fract → triangle → cubic ease → multiply by `bloom` → φ (Breath). Parallel: time × ν accumulator (Sweep). A `Switch` selects the driver. Resolve: on the trigger, store the current φ, compute the target fraction in a small expression chain (loop over q is easier as an ISF 1×1 pass, or precompute targets for Q_snap ≤ 4 with a few `Round` nodes) and `Smooth` toward it.
- **φ_prev**: a delay of φ by τ seconds. If your Wire version has no delay node, approximate it with `φ_prev = φ − dφ/dt · τ` (derivative from a one-frame delayed value). This is exact for Sweep and close enough for Breath.
- **Audio**: audio analysis low band → envelope (attack 5 ms, release 150 ms) → `pulse`. High band → onset → ramp-offset step.
- **MIDI**: KeyStep 37 notes → sort the held notes → intervals above the lowest note → lookup table to m_1…m_3.
- **Render**: ISF node "WhitneyField" (P1 + P3 in one pass) → optional feedback via Wire's own feedback node (P2; multipass ISF support varies by Wire version, so do not rely on it) → Bloom → output. Wire has no vertex shaders, so the Lissajous mode stays an N-loop in the fragment shader, with N ≤ 128.
- **Exposed parameters**: bloom, focus, ratio, symmetry, tail, resolve, cycle, palette, mode (spiral / ring / RDTD / Lissajous).

### Unreal Engine 5.7

- **Niagara emitter `NE_Harmonics`**: GPU Compute Sim, Spawn Burst Instantaneous N (User parameter, maximum 1024) at t = 0, infinite lifetime. Particle index k = ExecutionIndex + 1.
- **Update (Scratch Pad module)**: read User.Phase, User.PhasePrev, User.Mult (Vector for 3 voices), User.Mode, User.Symmetry. Compute θ_k and r_k (technique 2), and set Position in a local plane (X/Y, or tilt the emitter for 3D). For symmetry S > 1, spawn N·S particles and use `k = index mod N`, `s = index div N`.
- **3D variant** (research variation 3): give each voice a Z offset (0, 40, 80 cm) and dolly the CineCamera through the axis on the resolution moment, with the dolly timed so the camera passes the aligned ray on the downbeat.
- **Renderers**: Sprite Renderer, unlit additive material, colour from a Curve Linear Color sampled at k/N. Tails: a Ribbon Renderer with Ribbon ID = k and a short lifetime is expensive at N > 300; better to keep the sprites and add the analytic tail as a second sprite stretched along the velocity (Sprite Alignment = Velocity Aligned, length ∝ |L_k| · r_k).
- **Post-process volume**: Bloom intensity 1.0–2.0 (threshold about 1.0 in HDR), Film Grain intensity 0.1–0.2, Chromatic Aberration 0.1–0.3 as the optical-printer offset, exposure locked (Manual) so the black stays black.
- **Phase clock**: a Blueprint actor computes φ, φ_prev and c every tick from BPM and the active driver, and pushes them to Niagara User parameters.
- **Remote Control**: expose bloom, focus, ratio (int), symmetry, tail, cycle, resolve (bool pulse) and BPM in a Remote Control Preset, then bind to OSC (from Arena or a phone) and to MIDI (BeatStep Pro knobs CC, KeyStep notes for chord mode).
- **Output**: Spout (or NDI for a second machine) into Arena layer 2, blended Add.

### Godot

- **2D route**: a `ColorRect` filling the viewport with a CanvasItem shader that implements P1 + P3. Uniforms: `phase`, `phase_prev`, `consonance`, `dot_count`, `mult` (vec3), `symmetry`, `radius`, `r0`, `rho`, `m_r`, `dot_size`, `tail_gain`, `pulse`, `mode`, `palette` (`sampler2D` gradient texture), `misreg`.
- **Trails (P2)**: a `SubViewport` (HDR 2D enabled, 0.5× size, clear mode Never) containing a `TextureRect` of its own previous frame with the decay/zoom shader plus the field shader in Add. Display it through a `ViewportTexture`.
- **Lissajous or 3D**: `GPUParticles3D` with amount = N and a particle process shader that sets each particle's position from INDEX and the phase uniforms (no physics). Or use `MultiMeshInstance2D` with per-instance positions updated from GDScript for N ≤ 500.
- **Clock**: GDScript autoload for φ, φ_prev and c, with MIDI input through Godot's `InputEventMIDI` (KeyStep notes, BeatStep CCs).
- **Post**: `WorldEnvironment` glow (2D HDR enabled), glow intensity 0.8–1.2, blend mode Additive.
- **Output**: Spout from Godot needs a third-party plugin. Otherwise capture the window in Arena or use NDI through an addon.

## Audio reactivity

| band | drives | smoothing | threshold |
|------|--------|-----------|-----------|
| kick (40–120 Hz) | dot size pulse +40 % | attack 5 ms, release 150 ms | onset > +6 dB over 1 s average |
| bass (60–250 Hz, RMS) | radius R ± 4 % | attack 40 ms, release 400 ms | none, continuous |
| snare / mid (1–3 kHz) | tail τ +30 % per hit | attack 10 ms, release 300 ms | onset > +5 dB |
| hats (6–12 kHz) | ramp offset step +0.02 | none (stepped) | onset, 60 ms hold-off |
| track energy (full-band RMS over 4 bars) | bloom upper limit | 4-bar average | — |

- Do not drive φ directly from audio. The tension and release depends on φ staying smooth and rule-bound. Audio only changes size, radius, tail and colour, and triggers Resolve.
- Optional "follow the band": run a pitch tracker on the bass or lead (or use KeyStep MIDI) to select the ratio, so the figure follows the harmony.

## Performance tips

- P1 in spiral mode tests 4 candidates per voice per symmetry fold, so it costs about 12–20 Gaussian evaluations per pixel for 3 voices. That is cheap even at 4K. Ring mode with large N and Lissajous mode loop over N. Keep N ≤ 128 in ISF and render at 0.75× there.
- P2 at 0.5× resolution is enough, because trails are soft anyway.
- Size dots in pixels of the shorter screen side (`σ` from min(width, height)), so they stay round and equally sized on 32:9 walls. Centre the figure in the middle of the wall's main viewing area, not the pixel centre, if the processor maps an odd canvas.
- On very wide walls (> 3:1), use S = 2 or 4 with two figure centres (left and right thirds) instead of one huge figure. A single centred figure leaves the edges empty.
- Dots smaller than 2 LED pixels flicker on low-pitch walls. Set a minimum dot radius of 1.5 physical pixels and lower N instead.
- Strobing: respect the speed clamp. At 50 Hz LED refresh, wagon-wheel effects start earlier than at 60.
- Use 16-bit float for P1 and P2. In 8 bit the faint tails band and the feedback never reaches black.

## Further references

- [Digital Harmony, full text (archive.org)](https://archive.org/stream/DigitalHarmony_201611/Digital%20Harmony_djvu.txt): Whitney's own explanation of differential motion and harmonic resonance, the basis for techniques 1 and 6.
- [Matrix III analysis (Motion Design Archive)](https://luckydumpling.github.io/motion-archive/entries/john-whitney-matrix/): film embed and notes linking the work to Processing and TouchDesigner practice.
- [Lissajous curve (Wikipedia)](https://en.wikipedia.org/wiki/Lissajous_curve): how frequency ratio and phase shape the figures used in the *Matrix III* mode.
- [ISF JSON reference](https://docs.isf.video/ref_json.html): inputs and pass definitions for building the WhitneyField ISF node.
