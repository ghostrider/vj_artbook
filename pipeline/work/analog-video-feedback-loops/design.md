# Analog Video Feedback Loops — design

## Look

A black frame with a small bright seed, a logo, a line of text, a dancer from a booth camera or a Niagara burst, falls into itself. Each frame holds a slightly smaller, slightly turned, slightly recoloured copy of the last one, so the seed becomes a tunnel of nested frames that spirals toward the centre. Each nesting step shifts the hue a little, so the rings run through the colour wheel. Lens blur and a soft brightness ceiling turn hard edges into glowing filaments, and at the right zoom/rotation pair the image locks into a lattice of bright spots with 5-, 6- or 8-fold symmetry, like the octagon still on [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Video_feedback_octagon_pattern.jpg). The image is never still. It grows, locks into a pattern, then mutates when a knob moves, which is how the camera-at-monitor rigs of the Vasulkas, Skip Sweeney and Doctor Who's "howlround" titles behaved ([Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)). There are two flavours. **Analog**: soft, chromatic fringes, slight grain, slow response. **Digital**: crisp, two-transform folds that sharpen into fractal forms, like Dave Blair's beam-splitter device ([The Light Herder](https://www.thelightherder.com/)). On stage both flavours can come from the same chain: an emulated GPU loop, or a real camera aimed at a screen, with the computer acting as the knob board.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #000000 | Normal | 100 % | true black for LED walls; the loop's frame border falls back to this |
| 2 | Seed source | clip, text, logo, booth camera (NDI/Spout) or Niagara render, luma-keyed | — (fed into P1, also optionally visible) | seed 0–1 | the energy input; without it the loop dies to black |
| 3 | Seed shaper | luma key (threshold 0.35, soft 0.1) + optional edge/outline extract | inside P1 | — | thin bright shapes on black make the best seeds |
| 4 | Feedback buffer | ping-pong pair A/B, RGBA16F, loop resolution (default 1920×1080) | — (not visible) | — | holds the previous loop frame; see P1 |
| 5 | Loop pass | inverse transform (zoom, spin, offset), optics (blur, chroma split), colour (hue step, saturation), gain + soft clip + floor, seed add | written to buffer | — | the whole effect lives here |
| 6 | Second transform (fold) | optional mirrored/second affine copy of the previous frame summed in P1 | Add inside P1, weights 0.5/0.5 | fold 0–1 | the beam-splitter mode: fractal folds |
| 7 | Auto-gain meter | mean luma of the buffer (mip chain or 16×16 reduce), smoothed | — | — | drives the gain correction in the next frame (P2) |
| 8 | Output colouriser (optional) | luma of the loop through a palette ramp, **outside** the loop | Normal over 5, mix 0–1 | 0 % default | "keyer and colouriser" look ([Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)) without destabilising the loop |
| 9 | Post | black crush, light bloom, grain, optional unsharp mask | see compositing | — | keep it light: the loop already glows |
| alt 2–5 | Camera loop | real camera aimed at a monitor, captured and sent back out to that monitor | replaces 4–5 | — | see technique 6 |

## Techniques

**1. The loop equation**
1. Every frame: `out = softclip( floor_cut( gain · colour( optics( T(prev) ) ) ) + seed · S )`, where `prev` is last frame's `out`, `T` is the geometric transform and `S` is the keyed seed. This is the "iterated functional equation" view of video feedback that Crutchfield gave as one of his two models ([UC Davis](https://csc.ucdavis.edu/~chaos/chaos/pubs/stdvf-title.html)).
2. Store `out` in buffer A, then read it as `prev` next frame while writing B, then swap. Never read and write the same texture in one pass.
3. The loop delay is the frame time. All "per frame" amounts must be scaled by `dt` (below) so the look doesn't change between 50 and 60 fps or when a frame drops.

**2. Geometric transform (the camera pose)**
1. `p = uv - centre`, `p.x *= aspect` (so rotation stays round on any aspect).
2. Sample position: `q = R(-θ) · p / s + offset`, then `q.x /= aspect`, `uvPrev = q + centre`. The inverse is used because each output pixel asks where it came from.
3. `s < 1` (0.95–0.995 per frame): each generation shrinks, giving a tunnel that recedes to the centre (camera zoomed out). `s > 1` (1.005–1.05): the image blooms outward from the centre (camera zoomed in).
4. `θ` per frame 0–3°: spiral arms. Zoom and rotation together set the symmetry of the spot lattices ([Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)). Good starting pairs are s 0.97 with θ 45°/n per frame for n-fold looks; sweep slowly from there.
5. Outside the 0–1 range, sample black (clamp-to-border). That gives the monitor-bezel frame-in-frame look. Mirror-repeat is an alternative that makes kaleidoscopic edges.
6. Frame-rate independence: with `s_sec` and `θ_sec` given per second, per-frame `s = s_sec^dt` and `θ = θ_sec · dt`. The same applies to the hue step and the decay.

**3. Optics and colour (what the lens and monitor knobs do)**
1. Lens blur: 5-tap cross or 9-tap box, radius 0.5–2 px at loop resolution. This is the diffusion term. Without it, edges alias into digital noise. Bilinear sampling already adds about 0.5 px per pass.
2. Chroma split: sample R at `s·(1 + ca)` and B at `s·(1 − ca)`, with `ca` 0–0.004. Each generation separates the channels a little more, which gives rainbow fringes like a cheap CRT and lens.
3. Hue step: rotate the colour around the grey axis (YIQ rotation, or a 3×3 rotation about (1,1,1)/√3) by `hueStep` 0–6° per frame. Every nesting ring gets a new hue.
4. Saturation 0.9–1.15 per frame. Above 1.0 it climbs to pure hues, below 1.0 the loop goes pastel and then grey.
5. Contrast around a pivot: `c = (c − 0.25) · contrast + 0.25`, contrast 1.0–1.3. This is the "reaction" term. Bright areas grow brighter up to the ceiling and dim areas sink, so filaments and spots form instead of a smooth fog. Together with the blur it gives the reaction-diffusion behaviour of Crutchfield's second model ([UC Davis](https://csc.ucdavis.edu/~chaos/chaos/pubs/stdvf-title.html)).

**4. Stability: no white-out, no fade to black**
A loop with total gain above 1 runs to white, and below 1 it runs to black. Real rigs are saved by the monitor's finite brightness and the camera's limits. Rebuild those limits on purpose:
1. **Soft ceiling.** Replace hard clipping with a knee: below 0.8 the value passes through, above it the curve bends toward 0.95 (for example `knee + (1−knee)·(1 − exp(−(c−knee)/(1−knee)))` per channel). Values can never pass 0.95, and saturated regions keep their colour instead of turning flat #FFFFFF.
2. **Decay before gain.** Multiply by `decay` (0.90–0.995 per frame, `decay^dt`-scaled) and then by `gain`. With both near 1 the loop sits on the edge where patterns live. The stage knob is the product `gain·decay`, centred at 1.0.
3. **Auto-gain (AGC).** Each frame, read the mean luma `m` of the buffer from the 1×1 mip or a 16×16 reduce. Target `m*` = 0.18–0.30. Correction `k = clamp(m*/max(m, 0.002), 0.85, 1.15)`, smoothed with an attack of about 0.3 s and a release of about 1.5 s. It multiplies the gain in the next frame. This is the digital version of a camera's auto-exposure, and the main guard against both failure modes. An AGC strength 0–1 blends it in. At 0 the performer has the raw edge of chaos.
4. **Floor cut.** Subtract 0.5/255–1.5/255 per frame (then clamp at 0) so dim residue reaches true black instead of a grey haze. This is important on LED walls, where grey 3/255 is visibly on.
5. **Precision.** Use a 16-bit float buffer. In 8 bit, a decay of 0.98 can never lower values below about 25/255, because each step rounds back up. The result is a permanent muddy ghost, and the loop never clears.
6. **Seed floor.** Keep a minimum seed of about 0.05 (a thin ring or the keyed source) so a loop that has decayed always restarts, as Blair restarts his rig from an outside source or by turning up a screen's brightness ([Hackaday](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/)). A panic button (MIDI pad) sets `prev` to black for one frame to clear a blown-out state.

**5. Fold, cross-feed and delay variations**
1. **Two-transform fold (beam splitter).** `T(prev)` becomes `0.5·T1(prev) + 0.5·T2(mirror(prev))`, where T2 has its own scale (0.5–0.7), rotation and offset. Two shrinking copies summed every frame make an iterated function system, and the loop converges toward fractal shapes. It is the digital stand-in for 50/50 half-mirrored glass between two monitors ([The Light Herder](https://www.thelightherder.com/)). Keep the weights summing to about 1 and let the AGC handle the rest.
2. **Cross-feed.** Run loops A and B with different transforms and add `x · B` into A and `x · A` into B (x 0–0.3). With 2–3 loops this gives the "two structures influence each other" behaviour ([Hackaday](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/)).
3. **Frame delay.** Read `prev` from N frames ago (N 1–30) out of a ring of buffers. Echo rings appear spaced by N. Blair's device has delay units of up to 30 frames ([Hackaday](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/)). Run the ring at reduced resolution (below).

**6. Real camera-at-screen setup for the stage**
1. **Path:** camera (HDMI/SDI, manual exposure, manual focus, zoom lens) → capture card or NDI → Resolume layer → a dedicated **loop monitor** (a 24–32" LCD on a second output) → the camera films that monitor. The audience sees the Resolume composition on the main wall, not the loop monitor.
2. **Hood it.** Put the camera and monitor in a black box or under black molton. Stage light on the monitor adds a constant offset every pass and pushes the loop to white, and wash and strobe make it pump.
3. **Pose:** camera on a tripod with a ball head. The camera's distance from the monitor sets `s`, and rolling the camera (or the monitor on a turntable) sets `θ`. Start with the monitor just filling about 90 % of the frame and 5–15° of roll. Blair moves camera and monitors by hand, and his rotating monitor sits on a belt-driven dowel ([The Light Herder](https://www.thelightherder.com/)).
4. **Gain staging:** lock the camera's auto exposure, auto white balance and auto focus first. Set the monitor's contrast to about 70 % and brightness to about 45 %. The camera's iris or ND is the coarse gain, and the Resolume Levels/Brightness on the camera layer is the fine gain. To fight white-out: close the iris by half a stop, lower contrast, or lower the layer gain. To fight black: open the iris, or inject the seed (a phone torch, a hand, or a Resolume seed layer over the camera layer).
5. **Digital knobs in the loop:** Hue Rotate, saturation and Transform on the camera layer act as the monitor's hue, saturation, brightness and contrast knobs, mapped to the BeatStep Pro encoders. The analog look stays, but the knobs become MIDI-mappable.
6. **Moiré and latency:** defocus the lens slightly so the camera does not resolve the monitor's pixel grid; the defocus is also the lens-blur term. A digital camera, capture and display chain adds several frames of latency, so the loop responds in steps, not instantly ([Hackaday](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/)). Treat that as the loop's delay N. A hybrid also works: run the captured camera image through the digital AGC and soft ceiling (P1–P2) before sending it back to the monitor, for an analog look with digital safety.

## Shaders

Render order per frame: P0 seed → P1 loop (reads buffer A, writes B) → P2 meter (reads B) → P3 output/post (reads B) → swap A/B. The AGC value from P2 is used by P1 in the **next** frame. That one-frame lag is fine and avoids a CPU readback stall.

**P0 — Seed**
- Purpose: turn any source into thin bright seed shapes.
- Inputs: source texture, `keyThreshold` (0.2–0.6), `keySoft` (0.05–0.2), `outline` 0/1, `seedHue`.
- Per pixel: luma `L = dot(rgb, (0.2126, 0.7152, 0.0722))`; key `k = smoothstep(thr, thr + soft, L)`. If outline is on, take the gradient length of `k` from four taps for an outline-only seed. Output `rgb · k` (or tinted by `seedHue`).
- Output: RGBA16F at loop resolution, or half resolution for heavy sources.

**P1 — Loop**
- Purpose: one generation of feedback.
- Inputs: `prev` (buffer A, linear filtering, border colour black), `seed` (P0), uniforms `zoom`, `spin`, `offset`, `centre`, `aspect`, `blur`, `ca`, `hueStep`, `sat`, `contrast`, `gain`, `decay`, `agcK`, `agcAmt`, `floor`, `seedAmt`, `fold`, `T2` parameters, `dt`.
- Per pixel, step by step:
  1. Build `uvPrev` from the inverse transform (technique 2). If `fold > 0`, also build `uvPrev2` from T2 with the mirrored `p.x`.
  2. Sample `prev` at `uvPrev` with a 5-tap blur of radius `blur`, and sample R and B at chroma-offset scales. Repeat at `uvPrev2` and mix `(1 − fold/2)·A + (fold/2)·B`.
  3. Hue rotate by `hueStep·dt·60`, then saturation, then contrast around 0.25.
  4. Multiply by `gain · decay^(dt·60) · mix(1, agcK, agcAmt)`.
  5. Subtract `floor`, clamp at 0.
  6. Add the seed: `c += seed · seedAmt` (Add). Use Screen (`1 − (1−c)(1−s)`) for a softer seed.
  7. Soft ceiling (technique 4.1).
- Output: buffer B, RGBA16F, loop resolution (default 1920×1080, or 1280×720 for the analog look; lower resolution blurs faster per generation).

**P2 — Meter**
- Purpose: mean luma for the AGC.
- Inputs: buffer B.
- Per pixel: generate mips of B, or run a 16×16 reduce pass that averages blocks. Take the single value and apply the attack/release smoothing as a 1×1 persistent target: `agc = mix(agcPrev, k, rate)` with rate `1 − exp(−dt/τ)`, τ 0.3 s when rising and 1.5 s when falling.
- Output: 1×1 R16F `agcK`.

**P3 — Output and post**
- Purpose: what the audience sees.
- Inputs: buffer B, palette ramp, `colorise` 0–1, `blackCrush`, `bloom`, `grain`.
- Per pixel: optional colourise `mix(c, ramp(luma(c)), colorise)`. Apply the black crush: `c = max(c − 2/255, 0) · 255/253`. Add a light bloom (threshold 0.75, intensity 0.3–0.6, radius 8–16 px at 1080p) and grain at 0.02–0.04. An optional unsharp mask (amount 0.3, radius 1 px) brings back crisp digital edges. Sharpening goes **only** here, never inside P1, because inside the loop it amplifies every generation and blows out.
- Output: canvas resolution.

Buffers: A/B ping-pong at loop resolution, both RGBA16F, never cleared except on the panic button. Delay variation: a ring of N RGBA16F buffers at 960×540 (8.3 MB each, about 250 MB for 30). Cross-feed: a second A/B pair.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base | #000000 | background, frame border, floor-cut target |
| shadow | #0B0420 | deep indigo the colouriser ramp starts from |
| accent 1 | #FF2BD6 | magenta, mid-ramp, and the default seed tint |
| accent 2 | #19E3FF | cyan, the complement two-thirds of the way round the hue wheel |
| warm | #FFB000 | amber for "analog" scenes and Doctor Who-style monochrome howlround |
| highlight | #FFF4E0 | warm white, the soft-ceiling top (never pure #FFFFFF) |

- **Inside the loop**, colour comes from `hueStep`. At 2° per frame and 60 fps the hue wheel turns once every 3 s, so rings 10 generations apart differ by 20°. A step of 0 with a tinted seed gives monochrome (amber or cyan) howlround.
- **Outside the loop**, the colouriser maps luma through a 4-stop ramp #0B0420 → #FF2BD6 → #19E3FF → #FFF4E0. This keeps the loop's dynamics independent of the palette, so changing palettes never destabilises it.
- **Over time:** slow drift of `hueStep` per phrase; on the drop, flip its sign (the rings reverse direction around the wheel).
- **LED walls:** keep the floor cut on so blacks stay at 0, cap the peak with the soft ceiling at about 0.95 and the AGC target at 0.18–0.25 (walls are very bright, and a mean above 0.3 reads as a white wash). Use moderate contrast in post: the loop already has strong local contrast.

## Blending and compositing

- Seed into loop: **Add** (punchy, can saturate) or **Screen** (soft, never above 1). Default Screen with seed 0.4.
- Fold copies: plain weighted sum inside P1, weights summing to 1.
- Cross-feed: Add at x 0–0.3, then the shared soft ceiling.
- Loop over background: Normal. The loop can also be used as a **Luma matte** for another layer, or as a displacement source for another effect, since the research suggests running it as a mask or displacement map.
- Optional dry seed on top: Add at 20–50 % so the "now" shape stays readable inside the tunnel.
- Post order: colourise → black crush → bloom → (unsharp) → grain → output.

## Motion and timing

- **Base state:** zoom 0.985/frame (receding), spin 0.4°/frame, hue step 1.5°/frame, gain·decay 1.0 with AGC 0.7.
- **Sweeps:** move zoom and spin in slow sweeps of 8–16 bars with ease-in-out (sine). Rare hits are better than constant motion, since a single fixed setting locks into one static pattern.
- **BPM lock:** spin as a full turn of the spiral per 16 beats. At 128 BPM, 16 beats last 7.5 s, which is 450 frames at 60 fps, so θ = 0.8°/frame. The hue cycle is one wheel turn per bar or per 4 bars.
- **Kick:** zoom pulse. For one beat, `s` jumps toward 1.03 (bloom outward), then decays exponentially back to base with τ ≈ 120 ms. With `s` below 1 the image pumps out and falls back in.
- **Snare / clap:** seed flash. `seedAmt` goes to 1.0 for 2 frames, so a new ring of the seed enters the tunnel.
- **Hats:** offset jitter of ±0.002 uv for 1 frame. It seeds fine detail without moving the whole image.
- **Break:** AGC target down to 0.1, seed off, spin 0. The tunnel slowly fades to embers but never quite dies, because the seed floor stays on.
- **Build:** ramp contrast 1.0 → 1.25 and fold 0 → 0.6 over the last 8 bars. Filaments sharpen into fractal folds.
- **Drop:** flip the spin and hue directions, jump zoom to 1.01 (bloom) for one bar, then return to receding. Optionally switch from the digital loop to the camera loop with a 1-beat crossfade.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| zoom | 0.94 – 1.05 per frame | 0.985 | below 1: tunnel into the centre; above 1: bloom outward | BeatStep Pro encoder 1; kick adds +0.04 pulse |
| spin | −3° – +3° per frame | 0.4° | spiral arms and lattice symmetry | encoder 2; BPM-locked option (turn per 16 beats) |
| hue drift | −6° – +6° per frame | 1.5° | rainbow rings; 0 for monochrome howlround | encoder 3; sign flip on drop pad |
| feedback | 0.9 – 1.1 (gain·decay) | 1.0 | below 1 trails fade, above 1 patterns grow; AGC keeps it in range | KeyStep 37 mod strip or fader; never auto-modulated |
| seed | 0 – 1 | 0.4 | how much new image enters the loop | pad velocity (snare flash); fader for level |
| fold | 0 – 1 | 0 | blends in the second mirrored transform for fractal forms | encoder 4; ramp over a build |

Setup parameters (not for live riding): AGC strength 0.7, AGC target 0.22, blur 1.0 px, chroma 0.0015, contrast 1.1, floor 1/255, colourise 0.

## Build recipes

### Resolume Arena

- **No-code route (Arena 7.18.2+): FeedbackPro.** Resolume's FeedbackPro effect luma-keys the footage and then lets you move, scale and hue-shift the feedback ([Resolume blog](https://www.resolume.com/blog/23113)). Layer 1: seed clip (text, logo, booth NDI). Put FeedbackPro on the layer. Set scale just under 100 % for a tunnel, add a small rotation if your version has it, and a small hue shift. Map the scale, move and hue controls to the BeatStep Pro.
- **Spout round trip (works on any Arena that sends and receives Spout).** Turn on a Spout output of the composition. On the bottom layer, add a Spout source that receives Arena's own output. That layer is `prev`. On it, stack **Transform** (scale 97–99 %, rotation 0.2–2°), **Hue Rotate** (small amount), then a Levels/Brightness-type effect for gain, and set layer opacity to 90–98 % for decay. Put the seed layer above it in Add or Screen. Test this on your version: depending on routing it adds one or more frames of delay (which is fine), and the Spout path is 8 bit, so add a small black floor against ghosting (technique 4.5).
- **Custom effect (best quality):** build the P0–P3 loop as a Wire patch with an ISF node (below) and load it in Arena as an effect on the seed layer. Resolume documents ISF only inside Wire ([Resolume ISF](https://resolume.com/support/en/isf)), so the custom shader goes through Wire.
- **Camera loop:** camera capture on its own layer. Use Advanced Output to send that layer (or a slice) to the hooded loop monitor, and the main composition to the LED wall. Put Hue Rotate and Transform on the camera layer as the knob board.
- Post on the composition: small bloom and grain from your effect set. Keep **Kaleidoscope** off the loop itself (Resolume calls it a heavy effect ([Resolume 7.7 effects](https://resolume.com/support/en/7.7/effects))); fold mode gives the symmetry instead.

### Resolume Wire

- **Nodes:** seed input (video in) → ISF node `Seed` (P0) → ISF node `Loop` (P1) → output, plus ISF `Post` (P3).
- **Loop state:** in the `Loop` ISF, declare a PERSISTENT pass target with FLOAT true, and read the previous state by the target's name ([ISF multipass reference](https://docs.isf.video/ref_multipass.html)). The AGC can be a second persistent pass sized 1×1 (`WIDTH`/`HEIGHT` expressions), averaging a coarse grid of taps of the loop buffer (for example 16×9 taps) instead of a true mip chain.
- **Version caveat:** Resolume's ISF page does not cover multipass or persistent buffers, and support depends on your Wire version. Test the persistent buffer first. If it does not keep its state, fall back to the Spout round trip above with the ISF node doing only the per-pass maths. Wire has no vertex shaders ([Resolume ISF](https://resolume.com/support/en/isf)). Not needed here: everything is fragment work.
- **Exposed parameters:** ISF float INPUTS with MIN/MAX (zoom 0.94–1.05, spin −3–3, hue −6–6, feedback 0.9–1.1, seed 0–1, fold 0–1) plus an event input for the panic clear. Publish them to the Arena effect panel and map them to MIDI there. Feed BPM and an audio envelope into the zoom-pulse and seed-flash inputs.

### Unreal Engine 5.7

- **Render targets:** RT_A, RT_B (RTF RGBA16f, 1920×1080, clamp/border addressing), RT_Seed, and a 1×1 RT_AGC.
- **Blueprint actor `BP_FeedbackLoop`:** on Tick, render M_Seed → RT_Seed, then M_Loop (texture param `Prev` = current A) → B, then M_Meter → RT_AGC, then swap the A/B references. Use the `Draw Material to Render Target` node (Kismet Rendering Library). Set the actor's tick group to after the seed scene capture.
- **M_Loop (Unlit, User Interface or post-style material for RT drawing):** TexCoord → subtract centre → multiply X by aspect → **CustomRotator** (angle = spin) → divide by zoom → back to UV → several TextureSample taps of `Prev` for blur and chroma split → hue rotation (the HueShift material function) → saturation (Desaturation node with a negative or positive fraction) → contrast/gain/floor maths → Add seed → soft-knee curve built from Exp nodes → Emissive.
- **Seed:** a SceneCapture2D (Final Color LDR, black background) looking at a Niagara system (sprite burst on kick, ribbon spirals) writes RT_Seed. That makes Niagara the seed generator.
- **AGC:** enable mip generation on the render target if your setup allows it and sample the lowest mip; otherwise a small M_Meter that averages a 16×9 grid of taps into the 1×1 target, with smoothing done in Blueprint (FInterp To with a rising/falling speed).
- **Remote Control:** put zoom, spin, hue, feedback, seed and fold in a Material Parameter Collection, add them to a Remote Control preset, and bind them through the OSC and MIDI protocol bindings to the BeatStep Pro. Add a Blueprint function `PanicClear` (Clear Render Target 2D on A and B) to the preset.
- **Output:** show RT_B on a full-screen widget or a plane in front of a cine camera. Send it over Spout or NDI with the plugin you already use, so Resolume can take it as a layer source.

### Godot

- **Scene:** `SubViewport LoopA` (size 1920×1080, `render_target_update_mode = UPDATE_ALWAYS`, `render_target_clear_mode = CLEAR_MODE_NEVER` ([Godot SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)), HDR 2D on for float precision) contains a full-rect `ColorRect` with the P1 canvas_item shader. `SubViewport LoopB` (same settings) contains a `TextureRect` showing LoopA's ViewportTexture. LoopA's shader reads LoopB's ViewportTexture as `prev`. That makes the A → B → A ping-pong, with one or two frames of loop delay.
- **Seed:** a third SubViewport with the seed scene (Label, Sprite2D logo, a GPUParticles2D burst), or a VideoStreamPlayer for clips. Its texture goes to the `seed` uniform.
- **AGC:** a fourth SubViewport at 16×9 with a shader averaging taps of LoopA. Read it back only every few frames (`get_image()` is a CPU stall), or skip the readback and do the smoothing in a 1×1 SubViewport that feeds itself.
- **Uniforms to expose:** `zoom`, `spin`, `hue_step`, `feedback`, `seed_amt`, `fold`, `agc_amt`, `blur`, `chroma`, `floor`. Drive them from a script reading MIDI (Godot's MIDI input events) and an audio bus spectrum analyzer.
- **Output:** a final `TextureRect` with the P3 post shader in the main window. Use a Spout or NDI add-on if you need to feed Resolume.

## Audio reactivity

| band | drives | shaping |
|------|--------|---------|
| kick (40–120 Hz) onset | zoom pulse +0.02 to +0.04 | threshold at −18 dB relative to running peak; attack instant, release τ 120 ms |
| bass envelope (60–250 Hz) | spin speed ×(1 + 0.5·env) | attack 50 ms, release 400 ms |
| snare/clap (1–4 kHz) onset | seed flash to 1.0 for 2 frames | threshold with 150 ms hold-off so rolls don't flood the loop |
| hats (8–14 kHz) | offset jitter ±0.002 uv | attack 5 ms, release 60 ms; scaled by a "sparkle" amount |
| full-band RMS | hue step ×(0.5 – 1.5) | release 1–2 s; slow colour speed-up on loud passages |

Never drive `feedback` (gain·decay) from audio. Pumping the gain fights the AGC and causes white flashes. Use zoom pulses and seed flashes for impact.

## Performance tips

- **Cost:** P1 is one full-screen pass with about 5–10 texture taps (blur, chroma, fold doubles it). At 1080p this is well under 1 ms on a mid-range GPU. P0 and P3 are similar, and P2 is negligible. The loop is cheap; frame-time stability matters more than raw cost.
- **Fixed loop resolution:** run the loop at a fixed internal size (1920×1080 or a direct match of the LED processor's pixel map) and scale only once at the output. Every extra rescale inside the loop adds blur per generation, and the image goes to mush.
- **Frame drops change the look:** use the `dt` scaling for zoom, spin, hue and decay, and avoid VSync hitches (lock to 60 or 50 fps to match the LED processor).
- **Odd aspect ratios:** always aspect-correct the rotation, or very wide walls (for example 5760×1080) shear the spiral into ellipses. On extreme widths, consider 2–3 loops side by side with cross-feed instead of one wide tunnel, or place the centre off-axis.
- **8-bit paths (Spout, NDI, capture cards, Resolume layers):** expect sticky ghosts. Raise the floor cut to 1–2/255 and keep decay at or below 0.97.
- **Camera rig:** latency in a capture → render → display chain is several frames. Keep the chain the same during a show (same capture card, same output) or the loop speed changes. The hood and locked exposure matter more than any shader setting.
- **Flashing:** a high zoom with a large hue step can strobe. Keep the hue step under about 4°/frame at full brightness and follow venue flash rules.

## Further references

- [The Light Herder](https://www.thelightherder.com/) — Dave Blair's camera-at-screen feedback devices: monitors with analog knobs, 50/50 beam-splitter glass, Roland keying, and which physical parameters he adjusts; the model for the stage rig and the fold mode.
- [Hackaday: Analog Optical Feedback Generates 4K Fractals](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/) — the knob board (Teensy over RS-232), the up-to-30-frame delay units, and seeding from an outside source.
- [ISF Multi-Pass and Persistent Buffer Reference](https://docs.isf.video/ref_multipass.html) — PASSES, TARGET, PERSISTENT and FLOAT buffers, the basis of the Wire route.
- [Resolume: ISF in Wire](https://resolume.com/support/en/isf) — the ISF node, float inputs with MIN/MAX, and the note that Wire has no vertex shader support.
