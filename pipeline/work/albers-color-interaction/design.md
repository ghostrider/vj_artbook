# Albers Color Interaction — design

## Look

Flat, opaque fields of colour with razor-straight edges, nothing else: no texture, no glow, no motion blur. The wall is divided into square cells, each one a set of three or four nested squares sitting low in the cell, with a smaller margin below each square than above it (the schema of *Homage to the Square*, [Wikipedia](https://en.wikipedia.org/wiki/Homage_to_the_Square)). Nothing moves smoothly. Instead, colours are **swapped** on the beat, and the audience sees things that are not there: a centre square that "changes" although its pixel value never does, two different squares that suddenly read as the same, a red that turns lighter or darker when thin stripes cross it, a grey screen that fills with a violet ghost after a long yellow hold. The mood is calm, exact and slightly uncanny, like a lecture demonstration turned into a light show. This is an original piece built on Albers' exercises from *Interaction of Color* (one colour as two, two colours as one, afterimage, transparency, Bezold effect), not a copy of any painting. The palettes below are our own.

Design rule: **the illusion is the content.** Every hex value is chosen so the effect works on a real LED wall at audience distance. Layers stay opaque, and colour math happens in OKLab inside one shader. No blend modes touch the colour fields.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Frame / surround | flat off-white #D6D2C8 (or dark #121214 for club mode), outside the square cells | Normal | 100 % | the "white frame" that gives the work a beginning and an end; dimmed for LED walls |
| 2 | Cell grid | wall split into n square cells, side = wall height / rows, pixel-snapped | (inside shader) | — | n = 1 for 1:1, 3 for 3:1 walls, rows × cols for the grid mode |
| 3 | Nested squares | 3–4 levels per cell, colour index per pixel from the schema | (inside shader) | — | the figure/ground carrier for all exercises |
| 4 | Exercise layer | per mode: split ground, stripe pattern (Bezold), overlap region (transparency), fixation dot (afterimage) | (inside shader) | — | only one exercise is active at a time |
| 5 | Palette engine | 4–6 palette slots in OKLCh, eased in OKLab, hard cut or morph | (inside shader + control) | — | see Color scheme |
| 6 | Palette sampler (optional) | 4 colours sampled from a Spout/NDI or camera input, sorted by OKLab L | (separate pass P1) | — | research variation 2 |
| 7 | Output transform | sRGB encode, wall-specific trim (per-channel gain, black lift), ordered dither ±0.5 LSB | (inside shader) | — | the only "post" step |
| 8 | Overlay content (optional) | type, particles or feedback from other chapters | Normal, above, masked to the frame area | 100 % | never over the cells; see compositing |

Layers 2–5 and 7 are one single-pass fragment shader (P0). There is no bloom, grain, blur, sharpen or chromatic aberration anywhere in this piece.

## Techniques

**1. Nested-square schema (the stage)**
1. Pixel coordinates, origin bottom-left: `p = fragCoord`. Cell side `S = floor(H / rows)` px; cell index `c = floor((p - offset) / S)`; local `u = (p - offset - c * S) / S` in 0..1. `offset` centres the cell block on the wall; the leftover is frame.
2. For N levels and step m (default 0.1 of S): square k (k = 0..N-1) has side `1 - 2 k m`, centred in x, bottom edge `k * 0.5 m`, top edge `1 - k * 1.5 m`. With m = 0.1, N = 4 the sides are 1.0, 0.8, 0.6, 0.4 and the inner square ends 0.15 from the bottom and 0.45 from the top. That gives the "smaller margin below" rule.
3. Colour index = the highest k whose square contains the pixel. Snap all edges to whole pixels (`floor(edge * S + 0.5)`) and draw them **without** anti-aliasing. LED walls show a 1 px blend as a soft seam, and Albers' fields need hard edges.
4. `nest` parameter: 3 or 4 levels, plus a 16-cell grid mode (4 × 4 small Homages per square, or 16 × 4 across a 3:1 wall).

**2. One colour looks like two (fixed figure, flipping ground)**
1. The figure colour F is a constant uniform. It is never animated in this mode.
2. Ground G alternates between two colours that sit on opposite sides of F in lightness and hue (palette A1/A2 below).
3. Split mode: left half of the wall uses ground G1, right half G2, with the same F square in both. A thin bridge of F (a bar 2 % of S high) can join the two figures to prove they are one colour. Flip the bridge on and off on a hi-hat accent.
4. Flip mode: one ground for the whole wall, swapped on the beat. The audience sees the centre change; it does not.

**3. Two colours look like one (compensated figures)**
1. Pick two grounds (light-warm, dark-cool). Ground pushes the figure away from itself: a light ground makes a figure look darker, a dark ground makes it look lighter, and a grey on a bright coloured field takes on a slight tint of the contrasting colour ([Wikipedia: Simultaneous contrast](https://en.wikipedia.org/wiki/Simultaneous_contrast)).
2. So the figure on the light-warm ground is set **lighter and warmer** than the figure on the dark-cool ground, by roughly the amount the grounds take away. Starting values are in palette B; then tune by eye on the actual wall from the audience position.
3. The reveal: on the drop, cut both grounds to the frame colour. The two figures suddenly look different. Albers' point lands in one frame.
4. Expose the compensation as `match` (0 = identical figures, 1 = full compensation) so you can find the point where they read as one on that wall that night.

**4. Bezold effect (assimilation)**
1. Fill a cell or the whole wall with a field colour R. Draw interspersed stripes (or a small grid) of colour X across it, with stripe duty 30–40 %.
2. Swap only X (white, black, or a hue) and hold R fixed. With small interspersed areas, colours shift toward their neighbours (assimilation), the opposite of the contrast effect from large adjacent fields ([Wikipedia: Bezold effect](https://en.wikipedia.org/wiki/Bezold_effect)).
3. Stripe width decides which effect you get. Wide stripes give contrast, narrow ones give assimilation. Size them for viewing distance: 1 arcminute at 1 m is about 0.29 mm, so `stripe_mm ≈ 0.29 × distance_m × arcmin`. Aim for 3–8 arcmin at the main audience distance. Example: a P3.9 wall seen from 15 m, 4 arcmin = 17 mm, which is about 4 px.
4. The `stripe` fader runs from assimilation (narrow) to contrast (wide). On stage this is a strong build lever.

**5. Transparency and the illusion of mixture**
1. Two overlapping rectangles in colours A and B (palette D). The overlap gets a third flat colour `O = mixOKLab(A, B, w)`, never a blend mode.
2. Because O is a true perceptual in-between, the eye reads a transparent sheet. A sRGB average reads as a dull, darker patch (see Color scheme for numbers).
3. Sweep w (the `depth` parameter) from 0.2 to 0.8. Which sheet seems to lie on top changes along the sweep. Find the flip point by eye and put it on a beat. Albers' Foundation videos treat transparence and spatial illusion in parts 1–3 ([Albers Foundation](https://albersfoundation.org/alberses/teaching/josef-albers/logic-magic-of-color)).

**6. Afterimage (hold and release)**
1. Hold a saturated pattern (yellow discs on blue, palette E) with a small neutral fixation dot in the centre (0.5 % of wall height, #121214).
2. Hold long. The Wikipedia article's examples use 25–30 s and 5–60 s of staring. The ghost shows the complementary hue of the adapting image ([Wikipedia: Afterimage](https://en.wikipedia.org/wiki/Afterimage)). At 128 BPM, 16 bars ≈ 30 s.
3. On the drop, hard-cut to neutral grey #7F7F7F with the same fixation dot. Hold the grey 2–4 s (1–2 bars), so the audience sees the violet-blue ghost discs on the grey.
4. Large images and a steady eye help (same source). So fill the wall and keep the dot still. Announce nothing; the crowd discovers it.

## Shaders

Render order per frame: P1 (optional palette sampler) → P0 (generator and output transform). There is no feedback buffer.

**P0 — Albers generator (single pass, the whole look)**
- Purpose: cells, nested squares, exercise geometry, palette lookup, OKLab easing, output transform.
- Inputs (uniforms): resolution; mode (0 Homage, 1 One-as-two, 2 Two-as-one, 3 Bezold, 4 Transparency, 5 Afterimage, 6 Grid); nest (3/4); rows; margin m; palette slots P0–P5 as colours (sRGB hex from the UI); palette slots Q0–Q5 (the next palette); morph (0..1 easing position between P and Q, computed on the CPU from the beat clock); match; stripe (px); stripeAngle (0/45/90°); depth w; dot on/off; trimGain RGB; blackLift; dither on/off; optional texture `sampled` (1×4 px from P1).
- Per pixel, step by step:
  1. Cell and local coordinates (technique 1); outside cells is frame colour.
  2. Region index from the schema plus exercise geometry (split side, stripe on/off, overlap, disc, dot).
  3. Look up the region's slot in palette P and in palette Q.
  4. Convert both to OKLab: decode sRGB to linear, multiply by Ottosson's LMS matrix, cube root, then the second matrix ([Björn Ottosson: A perceptual color space for image processing](https://bottosson.github.io/posts/oklab/)). Mix `lab = mix(labP, labQ, morph)`. For transparency, compute O the same way from A and B with w.
  5. Convert back to linear sRGB. If out of gamut, reduce chroma at fixed L and hue (binary search, 6 steps in OKLCh) instead of clipping channels, which would shift the hue.
  6. Output transform: apply `trimGain` per channel and `blackLift` in linear; encode to sRGB; add ordered (Bayer 4 × 4) dither of ±0.5/255 only while morph is between 0 and 1.
- Output: RGBA8 at the native wall resolution, 1:1 mapping.
- Precompute: convert palettes to OKLab on the CPU when a palette is loaded, and pass OKLab values as uniforms, so the shader only does one Lab→sRGB per pixel. This matters only for very large walls; the ALU cost is small either way.

**P1 — Palette sampler (optional, variation 2)**
- Inputs: external texture (Spout/NDI/camera), mip level or a 9 × 9 box pre-blur.
- Per pixel of a 4 × 1 output: sample four fixed regions of the heavily blurred input (quadrant centres); convert to OKLab; sort the four by L (4-element sort network); write the sorted colours. Slot 0 = darkest → outer square, slot 3 = lightest → centre (or reverse for the "advance" reading).
- Smoothing: ease the sampled palette toward the new values with release 1–2 s (CPU side, or Wire's feedback node on the 4 × 1 texture) so camera noise does not flicker the squares.
- Output: 4 × 1 RGBA16F, read by P0 as palette Q when `source = sampled`.

## Color scheme

**Colour space: OKLab for all mixing and easing, OKLCh for authoring.** Why:
- Albers' exercises isolate one variable. "Change hue, keep lightness" and "change lightness, keep hue" only work in a space where L, C and h act as independent perceptual axes. That is OKLab's design goal ([Ottosson](https://bottosson.github.io/posts/oklab/)).
- sRGB averages are too dark and muddy in the middle (rose #C24A6A + teal #2F7F8C: sRGB mid #78647B, OKLab mid #8A6C7B). Yellow #F0C800 + blue #2E3FA0: the sRGB mid is olive #8F8450; the OKLab mid is the near-neutral #878A84 you would expect from complements.
- HSV/HSL lightness is not perceptual, and hue sweeps pass through unrelated hues. CIELAB shifts blue blends toward purple; OKLab stays more even in Ottosson's white-to-blue test (same source). This matters here because two palettes are blue-heavy.
- Easing in OKLCh (polar) is reserved for one case: the `hue push` fader, which rotates h at fixed L and C. Palette morphs use straight OKLab lines, so a swap between complements passes through grey, not through a rainbow.

The L/C/h values below were computed from the hex values with Ottosson's published matrices (L 0..1, C, h in degrees).

**Palette A — One colour as two** (technique 2)

| role | hex | OKLab L / C / h | use |
|------|-----|------------------|-----|
| figure F (fixed) | #8A8270 | 0.61 / 0.03 / 87 | warm grey; never changes |
| ground G1 | #3B2F5C | 0.34 / 0.08 / 294 | deep violet: F reads lighter and yellower |
| ground G2 | #D9B84A | 0.79 / 0.13 / 92 | ochre: F reads darker and greyer-violet |
| alt figure F' | #9A8088 | 0.63 / 0.03 / 358 | rose-grey, second phrase |
| alt G1' | #7A1F2E | 0.39 / 0.13 / 16 | wine: F' reads cooler, greener |
| alt G2' | #3E8C84 | 0.59 / 0.08 / 186 | teal: F' reads pinker |

Rule: F sits between G1 and G2 in L (about 0.25 above and below), and its hue lies on the axis between the two grounds. Low chroma figures (C < 0.04) move the most.

**Palette B — Two colours as one** (technique 3)

| role | hex | OKLab L / C / h | use |
|------|-----|------------------|-----|
| ground light-warm | #E6CF6A | 0.85 / 0.12 / 97 | pushes its figure darker and more violet |
| figure on light ground | #A8946E | 0.68 / 0.06 / 83 | set lighter and yellower to compensate |
| ground dark-cool | #26325A | 0.33 / 0.07 / 269 | pushes its figure lighter and more orange |
| figure on dark ground | #7B7C8E | 0.59 / 0.03 / 283 | set darker and bluer to compensate |
| reveal surround | #7F7F7F | 0.60 / 0 / — | both grounds cut to this on the reveal |

The two figures differ by ΔL ≈ 0.08 and clearly in hue. `match` scales that difference from 0 to 1 in OKLab around their midpoint.

**Palette C — Homage ramps** (Homage and Grid modes, outer → inner)

| ramp | 0 outer | 1 | 2 | 3 inner | OKLab L |
|------|---------|---|---|---------|---------|
| warm | #7A3B22 | #C9702E | #E8B04A | #F2E3B3 | 0.43 → 0.63 → 0.79 → 0.92 |
| cool | #1F3B45 | #4C7A78 | #8FB3A8 | #DDE6E0 | 0.34 → 0.55 → 0.74 → 0.92 |

Light centre on dark surround makes the centre advance. Reverse the slot order (light outside) and the centre recedes. Reordering the same four colours is the cheapest big change in the set: no new colour, a new picture. That is the research's "same palette, different order" point.

**Palette D — Transparency** (technique 5): A rose #C24A6A (L 0.58), B teal #2F7F8C (L 0.55). Overlap in OKLab: w = 0.3 → #A26175, w = 0.5 → #8A6C7B, w = 0.7 → #6F7582.

**Palette E — Afterimage** (technique 6): discs #F0C800 (L 0.84, C 0.17) on #2E3FA0 (L 0.42, C 0.16); release field #7F7F7F; fixation dot #121214.

**Palette F — Bezold** (technique 4): field #B8322A (L 0.52) with stripes #F2EEE6 (L 0.95) vs #141414 (L 0.19). Second set: field #2F5FA8 (L 0.49) with stripes #E6CF6A vs #141414.

**Frame**: #D6D2C8 off-white (stage) or #121214 (club). Pure #FFFFFF frames on LED walls glare and pull the eye away from the cells.

**Over time / with the music**: a palette is a scene. Change exercise and palette per 16-bar phrase, ground swaps per bar or per kick, and hue push and match on faders. Lightness is never audio-modulated continuously. Albers' effects need steady fields, so music drives **when** colours change, not a constant wobble.

**Black level and contrast**: darkest content value is #121214 (L 0.18), not #000000. Most of these effects live in the mid tones (L 0.3–0.85); crushed blacks and clipped whites are not needed, and peak white is #F4F1EA at most.

**How LED walls and projectors shift these colours** (practical guidance, verify on site):
- *LED native gamut*: LED walls are often wider than sRGB. If the processor has no sRGB/Rec.709 gamut mapping enabled, ochres and teals oversaturate and the near-neutral figures (C 0.03) pick up a tint. That tint breaks "one colour as two" because the figure is no longer neutral. Set the processor to Rec.709/sRGB target.
- *White point*: walls are often set cool. A cool white turns #8A8270 toward grey-blue. Trim with `trimGain` until the two halves of the split mode look as different as possible while the bridge bar still looks like one colour.
- *Dimming and bit depth*: dimming in content (dark palettes at 30 %) throws away 8-bit codes, and low-grey steps on LED drivers are coarse and can tint. Dim at the processor (higher internal bit depth) and keep content at designed values.
- *Module and batch variance*: neighbouring cabinets can differ slightly in low greys. Albers figures differ by only ΔL 0.05–0.1. Keep compared figures on the same cabinet row where possible, and run the calibration plate (below) before the show.
- *Viewing angle*: LED colour shifts off-axis, so side audiences see a slightly different palette. Tune `match` from a typical seat, not from FOH centre only.
- *Projectors*: the raised black level (projector black plus room light) lifts dark grounds. #26325A and #3B2F5C turn into grey-blue, and the dark-ground half of each exercise weakens. Raise figure contrast (larger ΔL between ground and figure) and prefer the light-ground exercises. DLP white boost brightens whites more than colours, so saturated grounds look darker relative to white frames; use the dark frame. Coloured screens or walls tint everything; measure with a white field first.
- *Calibration plate*: a hidden mode 7 shows all slots of the active palette as equal chips on #7F7F7F, plus the split-mode test. Tune trims by eye from the audience area. Afterwards do the **hex round-trip test**: pick the output colour from the Spout/NDI feed; #8A8270 must come out as #8A8270. A lighter or darker value means a double or missing sRGB encode.

## Blending and compositing

- **No blend modes between colour fields.** Every visible colour is computed in P0 as an opaque flat value. Add, Screen, Multiply or opacity fades in the host mixer would mix in the host's space, not OKLab, and would change exactly the values the illusion depends on. The research notes the same: keep Albers layers opaque.
- **Transparency is simulated, not blended**: the overlap colour is a third palette entry (technique 5).
- **Palette transitions**: default is a hard cut on the beat. Optional morph in OKLab over 1/8 note to 1 bar, with dither on during the morph. Never crossfade two Arena clips with different palettes; do the morph inside the shader.
- **Masks**: the cell shape can be used as a matte for other content: send `mode = Homage` in white/black as a luma key, so a video plays only inside square 2. Do it on a separate output, not over the live Albers layer.
- **Overlay content**: type or particles from other chapters can sit in the frame area (outside the cells), Normal blend, 100 %. Over the cells they destroy the effect.
- **Post chain**: none, by design. Only P0's output transform: trim → sRGB encode → dither. No bloom (it bleeds colour across hard edges and changes the grounds), no grain, no sharpen, no chromatic aberration, no vignette.
- **Feedback**: none. A feedback trail would turn the hard swap into a smear.

## Motion and timing

- **Nothing glides.** Changes are cuts on musical positions, with holds long enough for the eye to compare (minimum 1 bar, typical 2–4 bars).
- **Beat (quarter)**: in flip mode the ground swaps G1 ↔ G2 on the kick, but only in builds. Swapping every kick for long periods strobes and kills the comparison.
- **Bar**: default ground swap; Bezold stripe colour X swap; Homage slot order rotate by one (outer → inner).
- **Phrase (16 bars)**: change exercise and palette. Suggested set order: Homage warm → One-as-two (A) → Bezold → Two-as-one (B, reveal on the drop) → Transparency → Grid → Afterimage (break, released on the drop).
- **Build (8 bars)**: swap rate doubles every 2 bars (bar → half → quarter → eighth). `stripe` narrows from contrast width to assimilation width, so the field colour seems to drift as the stripes tighten. Grid mode cells go from 1 to 4 to 16.
- **Drop**: one of two "proof" moments. (a) Afterimage release: 16-bar hold → cut to #7F7F7F for 1–2 bars → full-colour Homage. (b) Two-as-one reveal: both grounds cut to grey and the "identical" figures split apart.
- **Break**: Afterimage hold with fixation dot, or one-as-two split held still with the bridge bar on; hi-hats toggle the bridge.
- **Grid waves** (variation 3): cells take the new palette with a delay of `d = cellIndex × (1/16 note)` along x, or by distance from the centre. Each cell still cuts hard; the wave is in the timing, not in a fade.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|----------------------|----------------------------|
| swap | trigger + rate (off, bar, 1/2, 1/4, 1/8) | bar | flips ground G1 ↔ G2 (or rotates slot order) | BeatStep Pro pad = manual swap; rate on a knob, quantised to the beat clock |
| contrast | 0–1 | 0.7 | scales ground-to-figure ΔL in OKLab around the figure L; low = subtle, high = obvious | fader; raise on projectors and for big rooms |
| hue push | −60° to +60° | 0° | rotates the ground hues in OKLCh at fixed L and C; the figure stays | KeyStep knob; never audio-driven |
| match | 0–1 | 1.0 | two-as-one compensation; find where the figures read as one on this wall | knob, set during soundcheck from the audience area |
| stripe | 2–40 px | 4 px | Bezold stripe width: narrow = assimilation, wide = contrast | fader; automate from wide to narrow over a build |
| morph | 0 (cut) to 1 bar | 0 | palette transition time in OKLab | knob; 0 for most of the set, longer in breaks |

Also on pads, not faders: mode (exercise), nest (3/4/grid), palette slot order reverse, fixation dot, afterimage release.

## Build recipes

### Resolume Arena

- **Main source**: P0 as an ISF shader. Resolume documents ISF support inside Wire ([Resolume ISF support](https://resolume.com/support/en/isf)), so build it as a Wire patch with one ISF node and use the patch in Arena as a source. Expose mode, swap, contrast, hue push, match, stripe, morph, and the palette colours.
- **Composition**: size = LED pixel map (for example 3456 × 1152 → three 1152 px cells), Advanced Output slice 1:1, no scaling. Leave the clip at Scale 100 %, Rotation 0, so edges stay on whole pixels.
- **Layers**: Layer 1 the Albers source, Normal, 100 %, nothing above it except overlays masked to the frame area. Do not put Arena effects on this layer, and do not use layer opacity or the crossfader to change palettes.
- **Palettes**: one clip per palette (A–F) with the colour parameters stored in the clip; triggering a clip on the beat is a palette cut. Arena's parameter animation in BPM-sync mode can drive swap; the audio analysis on the low band can trigger swaps in builds.
- **No Wire available**: stack Arena's built-in Solid Color sources as nested layers, each with transform Scale set to the square sizes (100/80/60/40 %) and Position Y shifted down by the margin rule, all at 100 % opacity, Normal. Swaps = clip triggers per layer. This covers Homage, One-as-two and Two-as-one. Bezold, transparency mixes and OKLab morphs need the shader. Precompute overlap colours (palette D) as hex by hand.

### Resolume Wire

- **Clock**: BPM/beat-phase source → bar and phrase counters → mode and palette index (array or switch lookup) and swap trigger with the rate selector. Node names differ between Wire versions; use the beat-clock and counter nodes from the node browser.
- **Audio**: audio analysis low band → envelope (attack 5 ms, release 150 ms) → threshold → swap trigger (gated by `build` toggle). Mid band only for the optional stripe automation.
- **ISF node** with P0: colour inputs for palette slots, float inputs for contrast, hue push, match, stripe, morph position, depth, trims; long/event inputs for mode and nest. Resolume's ISF page states that Wire (version 7.22) does not support vertex shaders; P0 is fragment-only, so that is fine ([Resolume ISF support](https://resolume.com/support/en/isf)). The same page does not cover multipass ISF, so keep P0 single-pass.
- **Palette sampler (P1)**: second ISF node at 4 × 1 output fed by a Spout/NDI input node; smooth with Wire's feedback node; route its output into P0's `sampled` image input.
- **Morph**: compute the morph position in Wire math (beat phase since the swap / morph length, clamped 0..1, smoothstep ease) and pass it to P0. The OKLab mix itself stays in the shader.
- **Exposed parameters**: swap rate, contrast, hue push, match, stripe, morph, mode.

### Unreal Engine 5.7

- **Material**: post-process material with Blendable Location "Replacing the Tonemapper", so the tonemapper, exposure and colour grading do not alter the hex values. Alternative: an unlit material on a full-screen plane captured by a Scene Capture with tonemapping disabled. Build P0 from ScreenPosition × ViewSize (pixel coords), Floor/Frac for cells, comparisons for the square schema, and a Custom node for sRGB↔OKLab (Ottosson's two matrices plus cube root/cube).
- **Parameters**: Material Parameter Collection `MPC_Albers` with Mode, Contrast, HuePush, Match, Stripe, Morph, Depth, and palette slots as vectors (P0–P5, Q0–Q5). A Blueprint holds the palette library (data table of hex values), converts to OKLab once per palette change, and writes the MPC.
- **Niagara**: not needed. The look is flat fields. Optionally use a Niagara grid as a control source for grid-wave timing, but a Blueprint loop over cell delays is simpler.
- **Post-process volume**: bloom 0, vignette 0, film grain 0, auto exposure off (manual), motion blur off, Screen Percentage 100. Then run the hex round-trip test on the Spout output.
- **Remote Control**: preset exposing the MPC scalars; OSC from Resolume for swap triggers and the beat clock; MIDI bindings for contrast, hue push, match, stripe (KeyStep knobs) and mode (BeatStep pads).
- **Output**: native wall resolution; Spout via a third-party plugin (for example Off World Live) or NDI via the NDI SDK for Unreal.

### Godot

- **Main**: a `ColorRect` filling the viewport with a `canvas_item` shader holding P0. Uniforms: `mode` (int), `nest` (int), `rows`, `margin`, palette slots as `vec3` already in OKLab (computed in script), `morph`, `contrast`, `hue_push`, `match_amt`, `stripe_px`, `depth`, `trim_gain`, `black_lift`. Use `FRAGCOORD.xy` for pixel coordinates.
- **Colour handling**: do the linear↔sRGB conversion explicitly in the shader and confirm with the hex round-trip test. Do not rely on uniform colour hints, which may convert implicitly depending on renderer and settings. For authoring, `Color.from_ok_hsl(h, s, l)` and the `ok_hsl_h/s/l` properties are handy for building palettes in GDScript ([Godot Color class](https://docs.godotengine.org/en/stable/classes/class_color.html)); OKHSL is a related space, not OKLab, so convert to OKLab in script before passing to the shader.
- **Palette sampler**: a `SubViewport` of 4 × 1 px with a second `ColorRect` shader sampling the camera or video texture; read it back as a `ViewportTexture` in P0.
- **Window**: viewport = wall resolution, stretch mode disabled, MSAA/FXAA off, no `WorldEnvironment` glow.
- **Control**: `InputEventMIDI` for knobs and pads; beat clock from OSC (a GDScript UDP listener) or from `AudioEffectSpectrumAnalyzer` low-band onsets.

## Audio reactivity

| band | drives | smoothing | threshold |
|------|--------|-----------|-----------|
| kick, 40–100 Hz onset | ground swap trigger (builds only; otherwise quantised to the bar) | attack 5 ms, release 150 ms; max one swap per 1/8 note | +6 dB over 1 s average |
| bass, 60–250 Hz level | nothing continuous; in Homage mode a bar-long average above threshold selects "advance" order (light centre) vs "recede" | 1 bar average | upper third of the set's range |
| mid, 300 Hz–2 kHz | optional: stripe width over a build (wide → narrow) | attack 500 ms, release 2 s | none |
| hi-hat, 6–12 kHz onset | bridge bar on/off in One-as-two; Bezold stripe colour X step | instant, hold 1/16 note | −30 dBFS gate; max one per 1/8 note |
| silence / breakdown detect | arms Afterimage mode; drop onset fires the release | — | full-band level below −24 dBFS for 2 bars |

Audio picks the moments; colour values stay fixed between moments. Continuous audio modulation of lightness or hue would make every field shimmer and erase the comparison the eye needs.

## Performance tips

- **Cost**: P0 is a handful of comparisons per pixel plus one OKLab→sRGB conversion (two 3 × 3 matrices and three cubes), about 40–60 ALU with gamut mapping. Trivial at 4K and 8K. P1 is 4 pixels. This is the cheapest chapter in the book; spend the GPU budget on other layers.
- **Precompute**: convert palettes to OKLab on the CPU per palette change. Then even gamut mapping only runs during morphs.
- **Resolution**: render at the native pixel map and map 1:1. Any scaling softens the hard edges into blended seams, and those seams are new colours.
- **Odd aspect ratios**: squares must stay square. Fit cells to the short side and let the frame take the remainder (16:9 → one 1080 px cell plus two 420 px frame bands, or two rows of 540 px cells, 3 + frame). For 32:9 strips use a row of cells. Never stretch the schema.
- **Pixel snapping**: compute edges in integer pixels; with `margin` 0.1 and S = 1152 px, edges land at 57.6 px, so snap. Unsnapped edges flicker between two columns during morphs and on some processors.
- **Pitfalls**: host-side opacity fades or blend modes on the Albers layer; bloom or glow anywhere in the chain; tonemapper or auto exposure in UE; double sRGB encoding; LED processor gamut mapping off; content-side dimming; holding the afterimage shorter than about 16 bars and expecting a ghost; swapping on every kick for minutes (strobe, comparison lost).
- **Safety**: hard full-wall swaps between very light and very dark grounds at 1/8 note are close to strobe territory. Cap fast swap rates at short bursts (2 bars) and keep ΔL of full-wall swaps under 0.5 at rates above 1/4 note.

## Further references

- [Björn Ottosson: A perceptual color space for image processing](https://bottosson.github.io/posts/oklab/) — the OKLab definition, conversion matrices, OKLCh, and the blending comparison with CIELAB and HSV used to justify the colour space.
- [Albers Foundation: The Logic and Magic of Color](https://albersfoundation.org/alberses/teaching/josef-albers/logic-magic-of-color) — four short videos with Albers' lecture audio over animated plates: colour mixing and transparence, spatial illusions, transparence of texture, and the Bezold effect.
- [Cassiel: HTTS](https://cassiel.com/project/htts) — an algorithmic, ever-evolving Homage to the Square built with Clojure tools for Studio Wayne McGregor and the Albers Foundation. A precedent for generative Albers on stage.
- [AutoAlbers (GitHub)](https://github.com/y-a-v-a/autoalbers) — canvas piece that makes a new Homage every 12 s, about how long people look at a painting; its colour-scheme module (mono, contrast, triade, analogic and more) is a useful model for a palette generator.
- [Wikipedia: Bezold effect](https://en.wikipedia.org/wiki/Bezold_effect) and [Wikipedia: Afterimage](https://en.wikipedia.org/wiki/Afterimage) — assimilation versus contrast, and the hold times and complementary hues behind the afterimage drop.
