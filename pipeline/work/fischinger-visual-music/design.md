# Fischinger Visual Music — design

## Look

A dark, slightly warm black screen, like a projected film gate. Out of it, on the music, simple shapes appear: a large disc on the kick, a long bar sliding in on the snare, pin-point dots on the hi-hats, a thin arc or spiral that climbs with the melody. Each shape is a separate voice with its own size, speed and depth. Shapes do not cut in. They spring into place with a small physical overshoot, sway as if hung on thread, cast a soft shadow onto the black, and then recede into depth and fade. Edges are not vector-perfect: they wobble very slightly and "boil" at 12 frames per second, like hand-drawn or cut-paper animation. Between phrases there are real rests: the screen empties to near black. The set moves through three looks: **Studie** (silver-white on black, light only), **Poem** (flat saturated paper shapes with shadows, after the hanging cut-outs of *An Optical Poem*, [TCM](https://www.tcm.com/articles/246892/an-optical-poem)), and **Lumigraph** (a second performer draws coloured light that only shows where it crosses a thin virtual light sheet, after the instrument described by [CVM](https://centerforvisualmusic.org/Fischinger/Lumigraph.htm)). This is an original homage in Fischinger's visual language, not a recreation of any of his films. The mood is precise, playful and musical: the eye should read the sync first, and the shapes second.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Gate | solid #0A0908, plus a very slow luminance flicker (±1.5 %) and a soft vignette | Normal | 100 % | the "film" black; never pure #000 on LED |
| 2 | Paint-build (optional) | accumulation buffer: stamps of past shapes at 4–8 % opacity, reset every N bars | Normal | 0–60 % | after *Motion Painting No. 1*; off by default |
| 3 | Shadows | soft offset copy of every shape's SDF, blur grows with depth | Multiply (paper mode) / off (light mode) | 45 % | computed inside the Voices pass, not a separate blur |
| 4 | Voices | 4 voices × 2 live shapes each (bass discs, snare bars, hat dots, melody arcs), 2D SDFs | Normal "over" (paper) ↔ Add (light), crossfaded by `paper` | 100 % | the core layer |
| 5 | Staff (optional) | 5 thin horizontal lines that scroll right-to-left at 1 bar per screen width, with note ticks | Add | 20 % | the "score view" variation; shows the sync |
| 6 | Lumigraph sheet | light-sheet drawing: input stamps masked by a thin depth band, coloured by 4 palette slots | Add | 0–100 % | fades in only in Lumigraph mode |
| 7 | Lumigraph persistence | feedback of layer 6, decay 0.90–0.97 per frame | Add (inside the feedback buffer) | 80 % | gives the after-glow of a moving hand |
| 8 | Post | gate weave, grain, soft focus, bloom (light mode only), vignette | see compositing | — | order below |

## Techniques

**1. Voices as 2D signed distance fields**
1. Work in aspect-correct space: `p = (uv - 0.5) * vec2(aspect, 1)`, so y runs -0.5..0.5 and circles stay round on any wall.
2. One SDF per shape type: disc `d = length(q) - r`; bar = box with half-size (w, h); dot = small disc; arc = ring `abs(length(q) - r) - t` cut by an angle range; spiral = ring whose radius grows with angle (`r = r0 + k·angle`, check the two nearest turns). Building shapes from distance plus smoothstep is covered in [The Book of Shaders, Shapes](https://thebookofshaders.com/07/).
3. `q = rotate(p - centre, angle) / scale`; multiply the resulting distance back by scale so edges keep one pixel width.
4. Coverage: `a = 1 - smoothstep(-px, px, d)` with `px` = 1.5 output pixels, converted to p-space by `1 / outputHeight`.
5. Composite the 8 live shapes back to front in a fixed voice order (bass far, melody, snare, hats near). With so few shapes no sorting is needed.

**2. Event envelopes with physical spring**
1. Each shape has an age `t` in seconds since its trigger and a lifetime `L` in beats (bass 2 beats, snare 1, hats 1/2, melody until next note + 1/2 beat).
2. Attack uses a damped spring, not a linear fade: `s(t) = 1 - exp(-ζ·ω·t) · cos(ω·t)`, with ω = 2π · 3 Hz and ζ = 0.45. This gives about 8 % overshoot, like a paper shape jerked on its thread.
3. Scale = `s(t)`; opacity = `min(1, t / 0.03 s)`. The first frame is the hit; there is no fade-in longer than 30 ms, or the sync gets soft.
4. Release: over the last 40 % of L, push the shape back in depth (`z` from 0 to 1 with ease-in quadratic) and fade opacity to 0.
5. Different voices use different ω: bass 2 Hz (heavy), snare 4 Hz, hats 8 Hz (snappy), melody 3 Hz. That difference is what makes them read as instruments.

**3. Depth, parallax and hanging-paper sway**
1. Each shape has depth `z` (0 near, 1 far). Apparent scale `k = 1 / (1 + 1.5·z)`; position is pulled toward the vanishing point `vp` (default screen centre): `centre' = vp + (centre - vp)·k`.
2. Sway: `angle = A·sin(2π·f·T + seed)`, A = 2–6°, f = 0.15–0.35 Hz, and a vertical bob of 0.3 % screen height at the same frequency, phase-shifted 90°. Each shape gets its own seed so they never move in lockstep.
3. Shadow: sample the same SDF at `p - lightDir · (0.01 + 0.03·z)` and use a soft edge width `px_shadow = 2 px + 40 px · z`. Shapes further from the "wall" cast larger, softer shadows. lightDir default (0.6, -0.8).
4. In paper mode shadows multiply the gate black up to 45 % darker; on a #0A0908 background this is subtle, so shadows mostly show when they fall on other shapes. That is the point: overlap reads as layered paper.
5. Optional slow dolly on bars 1 of each phrase: `vp` drifts 2 % of screen width over 16 bars.

**4. Hand-made edge ("boil")**
1. Stepped time: `Tb = floor(T · 12) / 12` (animation "on twos" at 24 fps).
2. Edge wobble: `d += wobble · noise(angleAroundCentre · 3 + seed, Tb · 7)` for discs and arcs, and `noise(p · 20 + seed, Tb)` along bar edges. Wobble 0.002–0.008 of screen height.
3. Because the noise only changes 12 times per second, the edge shimmers like redrawn frames rather than flowing. This adds the softness the research says a digital version has to put in on purpose.
4. Keep wobble at 0 on hats (dots read as clean ticks) and highest on melody arcs (they read as drawn lines, after the freehand line of *Motion Painting*, [Visual Music Archive](https://visualmusicarchive.org/works/w/motion-painting-no-1.html)).

**5. Score-driven triggering (sparse, with rests)**
1. A per-voice step pattern of 16 steps per bar works as the "graphic score" (CVM shows such notated plans, [CVM Process](https://www.centerforvisualmusic.org/Fischinger/OFProcess.htm)). Audio or MIDI onsets fire a shape only if the current step is enabled in the pattern.
2. A `density` fader is a probability gate on top: at 0.3 roughly every third allowed hit fires. Hash the bar and step index for the coin flip, so a repeated bar repeats its pattern (musical, not random noise).
3. Position comes from a hash of (voice, hit count) mapped into a voice zone: bass discs in the centre third, snare bars on left/right thirds, hats anywhere in the top half, melody height = pitch.
4. Melody voice: MIDI note → y = map(note, low, high, -0.4, 0.4), velocity → radius 0.04–0.16, x advances by 1/16 screen per note and wraps (like reading a score).
5. Rests: if input RMS stays below -40 dBFS (or no MIDI note) for 1 bar, all live shapes go to release, and no new spawns happen until the next onset. Silence becomes black.

**6. Lumigraph light sheet**
1. Input is a "performer field" H(x, y) with a depth value per pixel. Two sources: (a) a depth camera (Kinect, Orbbec, RealSense) aimed at the second performer; (b) without a depth camera, a pen, touch tablet or TouchOSC XY pad, where pressure (or a fader) stands in for depth.
2. The sheet: `visible = smoothstep(z0 - w, z0, depth) · (1 - smoothstep(z0, z0 + w, depth))`. Default sheet depth z0 = 1.2 m from the camera, thickness w = 4 cm. Only the part of a hand that pierces the sheet lights up, as on the real instrument ([CVM Lumigraph](https://centerforvisualmusic.org/Fischinger/Lumigraph.htm)).
3. With a 2D pen: stamp a soft disc (radius 0.01–0.04 screen height) along the stroke, with pressure above a threshold acting as "in the sheet".
4. Colour = weighted mix of 4 palette slots, the 4 "colour wheels". Each slot weight comes from one fader or knob controlled by the second player (the "colour ropes"). Normalise the weights; with all at 0 the light is cream white.
5. Feed the stamp into a persistence buffer (technique 7). Fingertip circles then leave glowing rings, and a flat object pressed through the sheet leaves a solid shape.

**7. Persistence and paint-build feedback**
1. Lumigraph buffer: `B = B_prev · decay + stamp`, decay 0.93 by default (≈ 0.5 s half-life at 60 fps). Slight blur in the feedback (1 px box) so trails soften as they fade.
2. Paint-build buffer: on each shape's release, stamp the shape at 6 % opacity into a non-decaying buffer. The canvas slowly fills with layered ghosts of past hits.
3. Reset the paint-build buffer on the bar counter (every 8, 16 or 32 bars) with a 1-beat fade to black, not a hard clear.

## Shaders

Render order per frame: P0 Voices, P1 Lumigraph stamp, P2 Lumigraph persistence, P3 Paint-build (optional), P4 Composite + post.

**P0 — Voices (single pass, the core)**
- Purpose: draw all live shapes, their shadows and the optional staff in one pass.
- Inputs (uniforms): per voice v ∈ {bass, snare, hat, melody} and slot j ∈ {current, previous}: `age_vj` (s), `seed_vj` (hit count), `posHint_vj` (melody: pitch and velocity; others: unused); global `time`, `bpm`, `beatPhase`, `paper` (0..1), `depth`, `sway`, `wobble`, palette colours, `lightDir`, `vp`, `staffAmount`.
- Per pixel:
  1. Build aspect-correct p; compute stepped time Tb.
  2. For each of the 8 slots with age < lifetime: derive centre from hash(seed) and voice zone; spring scale from age; depth z from release progress; sway angle; apply parallax.
  3. Evaluate the SDF with wobble; coverage `a` with 1.5 px edge.
  4. Shadow coverage from the same SDF at the offset position with the depth-dependent edge width.
  5. Accumulate back to front: `col = mix(col, shapeCol, a)` for paper; `col += shapeCol · a` for light; final `col = mix(lightCol, paperCol, paper)`. Shadows multiply only in the paper branch.
  6. Staff: 5 lines `1 - smoothstep(0, 1 px, abs(fract(...)))`, scrolled by beat phase, at staffAmount.
- Output: RGBA8 or RGBA16F at output resolution; alpha = total shape coverage (useful as a matte).
- Two slots per voice is a deliberate limit: it makes the pass stateless and single-pass (works as one ISF in Wire). Engines with a real pool (UE5, Godot) can use 8–16 slots per voice.

**P1 — Lumigraph stamp**
- Purpose: turn the performer input into light inside the sheet.
- Inputs: depth texture (depth camera, via Spout/NDI or engine plugin) or pen position/pressure uniforms; `z0`, `w`, 4 slot colours, 4 slot weights.
- Per pixel: depth mode, apply the sheet band from technique 6 and colour it; pen mode, distance from pixel to the segment between last and current pen position, soft disc of radius r. Multiply by the mixed colour.
- Output: RGBA16F at half output resolution (the light is soft, half res is invisible).

**P2 — Lumigraph persistence (ping-pong)**
- Inputs: previous buffer, P1, `decay`.
- Per pixel: 5-tap cross blur of previous (1 px), times decay, plus stamp; clamp at 3.0.
- Output: RGBA16F, half resolution, two buffers swapped each frame (in ISF: a `PERSISTENT` pass target, see [ISF JSON reference](https://docs.isf.video/ref_json.html)).

**P3 — Paint-build (optional, persistent)**
- Inputs: previous buffer, P0 alpha and colour, `releaseStamp` event, `reset` fade value.
- Per pixel: `B = B_prev · (1 - resetFade) + stamp · 0.06` when a shape releases.
- Output: RGBA8 is enough (it does not decay, so no banding from repeated multiplies), output resolution.

**P4 — Composite + post**
- Inputs: P0, P2 (bilinear upscaled), P3, gate colour, `time`.
- Per pixel, in order: gate weave (shift uv by `0.0006 · noise(Tb)` in x and y, stepped at 12 fps); gate black + P3 · paintAmount; over P0 (blend as in P0); add P2 · lumigraphAmount; soft focus (only for light mode: 1-tap mix with a 2 px blur, 25 %); bloom (light mode only, threshold 0.7, intensity 0.8); monochrome grain 4 % stepped at 24 fps; flicker ±1.5 %; vignette 0.3.
- Output: 8-bit output resolution for Spout/NDI.

## Color scheme

Three palettes, one per look. All flat colours: no gradients inside a shape.

**Studie (black and white, after the Studies)**

| role | hex | use |
|------|-----|-----|
| base | #0A0908 | gate black |
| main | #ECE7DC | silver white, all shapes |
| mid | #8E8A82 | far shapes (z > 0.6) and staff lines |
| shadow | #000000 | not used, light mode only |

**Poem (flat paper colour)**

| role | hex | use |
|------|-----|-----|
| base | #0B0C14 | blue-black gate |
| bass | #D9412B | vermilion discs |
| snare | #2643B8 | cobalt bars |
| hats | #F2E6C9 | cream dots |
| melody | #E9B33C | ochre arcs and spirals |
| accent | #1E8C82 | teal, swapped into one voice per phrase |
| shadow | #000000 at 45 % | multiply shadow |

**Lumigraph (four "colour wheels")**

| slot | hex |
|------|-----|
| 1 | #FF3B2F red |
| 2 | #FFC23A amber |
| 3 | #2FB6FF sky |
| 4 | #7A3BFF violet |
| none | #F5EFE2 cream white |

- Mapping: each voice owns one flat colour. Depth darkens a shape by `mix(col, base, 0.5·z)` rather than changing hue, so recession reads as distance.
- Over time: per phrase (16 bars), rotate the accent: one voice takes the teal for the whole phrase. The look itself (Studie → Poem → Lumigraph) changes only by hand or every 64 bars, with a 1-bar crossfade.
- Music: the colour stays fixed while a shape lives. Never colour-cycle a living shape; Fischinger's sync is in shape and motion, and hue cycling muddies it.
- LED walls: black at #0A0908 / #0B0C14 keeps modules just above off. Peak cream #ECE7DC (about 92 % luminance) prevents large discs from clipping and dimming the wall on power-limited processors. Large flat discs draw a lot of current: cap any single shape at 35 % of screen area, or lower its opacity to 85 %.

## Blending and compositing

- **Paper mode (Normal over + Multiply shadow)**: overlapping shapes cover each other like cut paper, and shadows give real layering. Use it for the Poem look.
- **Light mode (Add)**: overlaps get brighter, like projected light; use it for Studie and Lumigraph. The `paper` parameter crossfades the two results, so morphing between them is a stage gesture in itself.
- **Mattes**: P0 alpha is a clean matte. Use it as a mask to show other video only inside the shapes, a direct Fischinger-shaped window onto other material.
- **Over other material**: in light mode the output is mostly black, so it sits on top of other layers in Add or Screen without a key.
- **Feedback mix**: Lumigraph decay 0.93 (0.85–0.97). Paint-build 6 % stamps, reset every 16 bars. No feedback on the voice layer itself: the sync must stay crisp.
- **Post chain in order**: gate weave → composite → soft focus (light mode 25 %) → bloom (light mode, threshold 0.7, intensity 0.8) → grain 4 % monochrome, stepped → flicker ±1.5 % → vignette 0.3. No chromatic aberration and no sharpening: they fight the film feel.

## Motion and timing

- **Spawns are frame-exact**: a shape is at full opacity on the first frame after the onset. Everything soft happens after the hit (spring settle, sway, recede), never before.
- **Lifetimes in beats** (at 124 BPM, 1 beat = 0.48 s): bass 2 beats, snare 1, hats 1/2, melody until next note + 1/2 beat. Tie them to BPM, not seconds.
- **Spring frequencies**: bass 2 Hz, melody 3 Hz, snare 4 Hz, hats 8 Hz, damping 0.45. Range ζ 0.3 (bouncy) to 0.9 (dead).
- **Sway**: 0.15–0.35 Hz, 2–6°; free-running, not beat-synced, so the picture breathes against the grid.
- **Snare bars travel**: a bar enters from a screen edge and travels 30 % of the width over its lifetime with ease-out cubic; direction alternates per hit.
- **Bars and phrases**: every bar, the step pattern can advance (pattern A/B/C/D); every phrase, the accent colour moves to another voice.
- **Vocabulary that grows over a set** ("chapters", advanced by a pad or every 32 bars): 1, bass discs only, Studie; 2, add snare bars; 3, add hats and the staff; 4, Poem palette with depth and shadow; 5, melody arcs and spirals; 6, Lumigraph opens; 7, paint-build on. Going back a chapter is allowed and is a strong break gesture.
- **Build**: density climbs from 0.3 to 1.0 over 8 bars, depth goes to 0 (everything near and flat), sway falls to 0. The picture gets tense and front-facing.
- **Drop**: one huge bass disc on the downbeat (radius 0.45, spring ζ 0.3 so it rings), screen flashes to the disc colour for one frame, then density 0.6 with full depth and shadow.
- **Break**: hats and snare off; one melody voice draws slow spirals; Lumigraph performer takes the lead. Rests are allowed to last a full bar of black.
- **Silent rhythm mode (after *Radio Dynamics*)**: an internal clock at a set BPM drives the step patterns without audio, so the visual rhythm leads and the musician follows a click or the bar counter.

## Parameters

| name | range | default | what it does on stage | suggested MIDI / BPM mapping |
|------|-------|---------|----------------------|------------------------------|
| density | 0–1 | 0.5 | probability that an allowed hit spawns a shape; low = sparse, rests | BeatStep Pro knob 1; automate up through builds |
| depth | 0–1 | 0.6 | how far shapes recede and how big their shadows get; 0 = flat graphic | knob 2; to 0 on builds, 1 after the drop |
| paper | 0–1 | 0 (Studie) | crossfades light (Add) and paper (over + shadow) | fader; snap via pad to 0 or 1 |
| sway | 0–1 (maps to 0–6°) | 0.4 | hanging-thread movement and wobble together; 0 = rigid machine | knob 3; mid-band level adds +0.2 |
| chapter | 1–7 | 1 | which voices and looks are active | BeatStep Pro pads 1–7, or auto every 32 bars |
| sheet | 0–1 | 0 | Lumigraph amount and sheet thickness (2–10 cm); 0 = off | second player: fader for sheet, 4 knobs for colour slots |

## Build recipes

### Resolume Arena

Resolume documents ISF as a Wire feature, with its own nodes and float, bool, color and event inputs ([Resolume ISF support](https://resolume.com/support/en/isf)). So the custom parts live in a Wire patch that Arena then plays as a source.
- **Layer 1**: solid colour source at #0A0908 (or the Poem base).
- **Layer 2**: the Wire patch "Fischinger Voices" (P0 + P4 merged) as a source. Expose density, depth, paper, sway, chapter, and the onset events. Blend Normal.
- **Layer 3**: the Lumigraph: either the Wire Lumigraph patch (P1 + P2) or a Spout/NDI input from UE5, Godot or TouchDesigner with the depth camera. Blend Add, opacity on a fader.
- **Layer 4 (optional)**: other video with the Voices alpha as a mask, for "window" moments.
- **Effects**: keep post inside the Wire patch so the order is fixed. In Arena, use only the built-in **Transform** effect on Layer 2 for a slow position drift, if wanted.
- **Beat triggers**: Arena's parameter animation can switch effects "to the beat" ([Resolume effects](https://resolume.com/support/en/effects)); use BPM-synced animation for the chapter auto-advance and the staff scroll. Map BeatStep Pro pads via Arena's MIDI shortcuts to the chapter and onset events.

### Resolume Wire

- **Audio in**: `Spectrum In` delivers 1024 bands, about 23 Hz each at 48 kHz ([Resolume Wire FFT](https://resolume.com/support/en/wire-fft)). Sum bins 2–4 (≈ 45–115 Hz) for kick, bins 7–11 plus 90–200 for snare, bins 300–520 for hats. For each band: average over 1 s, compare to the instant value (+6 dB) → onset trigger, with a 90 ms hold-off.
- **Score gate**: BPM/beat clock → step counter (wrap 16) → array lookup of the voice pattern → AND with onset → probability compare against `density` with a hash of bar·16 + step.
- **Per-voice slot logic**: on a gated onset, copy current age/seed to the "previous" slot, reset current age to 0, increment seed. Age increments by frame delta time. This gives the 8 `age_vj` and `seed_vj` floats.
- **MIDI in**: KeyStep 37 note → melody pitch/velocity floats and a melody onset event.
- **ISF node "Voices"**: single-pass P0 + P4, inputs as above (all floats, colours as color inputs).
- **ISF node "Lumigraph"**: P1 + P2. Multipass and persistent buffers are defined by the ISF spec ([ISF JSON reference](https://docs.isf.video/ref_json.html)), but support in Wire varies by version. If your Wire version does not keep a persistent buffer, do the decay with Wire's own feedback node and keep the ISF single-pass. Wire has no vertex shaders, so all drawing is fragment-only, which this design already assumes.
- **Blend** Voices and Lumigraph (Add) → output. Exposed parameters: density, depth, paper, sway, chapter, sheet, 4 colour-slot weights.

### Unreal Engine 5.7

- **Voices as Niagara**: GPU or CPU emitter per voice, sprite renderer, spawn burst of 1 particle per onset event from a User parameter (`User.KickTrigger` etc., incremented by OSC). Particle attributes: age, seed, voice, z. Lifetime from BPM: `Lifetime = beats · 60 / User.BPM`.
- **Modules**: Initialize Particle (position from a hash of seed inside the voice zone, sprite size), a Scratch Pad module for the spring scale (formula from technique 2) and sway rotation, Scale Sprite Size, Sprite Rotation Rate replaced by the scripted angle, Color with depth darkening. Pool sizes of 16 per voice are fine.
- **Sprite material** (unlit, Masked for paper or Additive for light, or two renderers): SDF per shape type from a `ShapeType` dynamic parameter; Custom node or a Material Function `MF_FischingerShape` for disc, box, ring, spiral; edge = 1 − SmoothStep with DDX/DDY for one-pixel width; wobble from a Noise node fed with stepped time `floor(Time·12)/12`.
- **Shadows**: a second sprite renderer in the same emitter with an offset of `lightDir · (1 + 3·z)` cm toward the back plane, black, Translucent, opacity 0.45, edge softness growing with z. Cheaper and more controllable than real lights.
- **Camera**: orthographic for Studie; perspective 30° for Poem so z is real depth.
- **Lumigraph**: depth camera via its UE plugin or Spout/NDI in → Material with the sheet band on depth → render target; persistence with two Canvas Render Targets ping-ponged by a Blueprint each tick (`Draw Material to Render Target` with prev · decay + stamp).
- **Post**: a post-process material in the Post Process Volume's Blendables for gate weave, grain and flicker; place it After Tonemapping (the default, cheapest) as described in [UE post-process materials](https://dev.epicgames.com/documentation/en-us/unreal-engine/post-process-materials-in-unreal-engine). Bloom on for light mode (intensity 0.8, threshold 0.7). Manual exposure, otherwise big discs pump the image.
- **Remote Control**: put density, depth, paper, sway, chapter, sheet, the 4 colour weights, BPM and the 4 onset triggers in a Remote Control Preset and bind them with the OSC and MIDI protocol plugins ([UE Remote Control protocols](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-protocols-in-unreal-engine)). Send onsets from Resolume or an audio analyser via OSC.
- **Output**: Spout or NDI to Resolume (third-party Spout plugin or the NDI SDK for Unreal), at the LED processor's native resolution.

### Godot

- **Voices**: one full-screen `ColorRect` with a `canvas_item` shader implementing P0; uniforms as listed in P0 (arrays of 16 floats per voice are fine in Godot, so use 8 slots per voice instead of 2). A GDScript keeps the slot ring buffers and sets the uniforms every frame.
- **Light mode**: a second ColorRect with the same shader and `render_mode blend_add`, crossfaded with `modulate.a` by `paper` ([Godot CanvasItem shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/canvas_item_shader.html)).
- **Lumigraph persistence**: a `SubViewport` with `render_target_clear_mode` = Never and `render_target_update_mode` = Always ([Godot SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)); draw a black ColorRect at alpha `1 - decay` first, then the stamp with blend_add. For the exact decay, use two SubViewports in ping-pong with a ColorRect shader reading the other's `ViewportTexture`.
- **Paint-build**: another SubViewport with clear mode Never; reset by setting clear mode to Once.
- **Post**: final ColorRect shader for weave, grain, flicker and vignette; `WorldEnvironment` glow for light mode.
- **Input**: `InputEventMIDI` for KeyStep notes and BeatStep knobs; an audio bus with `AudioEffectSpectrumAnalyzer` for band energies; OSC via an addon if driven from Resolume.

## Audio reactivity

| band | drives | smoothing | threshold |
|------|--------|-----------|-----------|
| kick, 45–115 Hz | bass disc spawn (gated by score and density); drop disc | attack 2 ms, release 150 ms | onset at +6 dB over the 1 s average, 90 ms hold-off |
| snare / clap, 150–250 Hz + 2–5 kHz | snare bar spawn | attack 2 ms, release 120 ms | onset +5 dB, both bands must fire within 20 ms |
| hi-hat, 7–12 kHz | hat dot spawn, 1/2-beat life, opacity 60 % | attack 1 ms, release 50 ms | onset +4 dB, max 8 per beat |
| mids, 300 Hz–2 kHz level | sway amount +0 to +0.2, wobble | attack 150 ms, release 1 s | none, continuous |
| full RMS | rest detection (silence → release all) | 1-bar window | below -40 dBFS for 1 bar |
| MIDI notes (KeyStep 37) | melody arcs: pitch → height, velocity → size | none | note on |

Keep everything except onsets slow. The fast part of this look is the hit itself; continuous modulation of size or colour by levels destroys the score-like clarity.

## Performance tips

- **P0** is cheap: 8 SDFs plus 8 shadow SDFs and noise per pixel, about 30 noise calls at most. 4K at 60 fps is fine on any discrete GPU. With 16 slots per voice in Godot or the shader version, skip dead slots early (age > lifetime) with a uniform-driven loop bound.
- **Lumigraph** at half resolution: two RGBA16F buffers, negligible. Depth cameras add 30–60 ms latency; compensate by keeping the sheet thin and the decay long, so the lag reads as after-glow.
- **Paint-build**: RGBA8 at output resolution; avoid 8-bit for decaying feedback (banding), it is fine here because nothing decays.
- **Odd aspect ratios**: all coordinates are height-normalised, so circles stay round. On 32:9 or wider strips, widen the voice zones by aspect, and spawn snare bars horizontally only; give the melody x-advance 1/32 of the width.
- **Low-res LED (P5+ pitch)**: set edge width to 2.5 px, turn wobble down (it turns into flicker) and grain off.
- **Power and brightness**: large flat discs at full white can trip power limits on some walls; keep the 35 % area cap and peak at #ECE7DC.
- **Pitfalls**: fading shapes in (kills sync); every voice firing on every hit (boring, the research's main warning); same spring on all voices; colour cycling on live shapes; auto-exposure in UE pumping on big discs.

## Further references

- [CVM: The Lumigraph](https://centerforvisualmusic.org/Fischinger/Lumigraph.htm) — how the light sheet, colour wheels and two-player setup worked; the basis for the Lumigraph mode.
- [CVM: Process](https://www.centerforvisualmusic.org/Fischinger/OFProcess.htm) — graphic notated scores used to plan animation; the model for the score-gated triggering.
- [The Book of Shaders: Shapes](https://thebookofshaders.com/07/) — distance-field circles, boxes and polar shapes with smoothstep edges, the base of the Voices pass.
- [ISF JSON reference](https://docs.isf.video/ref_json.html) — PASSES, TARGET and PERSISTENT buffers for the multipass Lumigraph in ISF hosts.
