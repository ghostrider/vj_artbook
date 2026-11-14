# Oscilloscope XY Vector Art — design

## Look

A single glowing line on pure black, drawn in one continuous stroke like the beam of an analogue scope in XY mode. The default colour is green phosphor: the core of the line burns almost white, the edges fall off into saturated green, and a soft halo blooms around it. Where the beam slows down, turns a corner or retraces itself, the line gets brighter. Where it moves fast it thins out into a faint thread, which is the scope's own brightness logic ([woscope](https://m1el.github.io/woscope-how/)). The shapes run from Lissajous knots (two sines at a ratio such as 3:2, slowly precessing) to spinning wireframe cubes and one-stroke logos. Every shape leaves a short afterglow tail, so motion reads as smeared light rather than frames. The mood is lab equipment turned instrument: cold, precise and hypnotic, with the warmth of glowing phosphor. On the kick the figure jumps outward and snaps back. On the drop the frequency ratio steps to a new figure and the afterglow stretches into long trails.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #000000, optional faint graticule (10 × 8 grid, #0A2A12) | Normal | 100 % | the graticule is off by default on LED walls |
| 2 | XY signal (hidden) | either a stereo sample ring buffer (real audio) or generated oscillators | — | — | pass P0; this is data, not image |
| 3 | Beam | segment-integrated Gaussian beam, history fade by sample age | Add over layer 1 | 100 % | pass P1; the main look |
| 4 | Phosphor accumulation (optional) | feedback buffer: `accum = accum · exp(−dt/τ_fb) + beam` | replaces layer 3 output | — | pass P2; only for long smears (> 300 ms) that the history window cannot cover |
| 5 | Tone map / colour | intensity → phosphor ramp, saturation to white | — | — | pass P3 |
| 6 | Post | bloom, optional CRT curvature and vignette, grain | see compositing | — | bloom carries the glow on big screens |
| 7 | Over content (optional) | whole stack on top of footage | Add or Screen | 60–100 % | or used as a luma mask for another layer |

## Techniques

**1. XY mapping (the scope itself)**
1. One stereo sample pair `(L, R)` is one point: `x = L`, `y = R`, both in −1…1. On a real scope in XY mode the left channel moves the beam horizontally and the right channel moves it vertically (research, via IEEE Spectrum).
2. Screen position: `p = centre + (x, y) · scale · 0.45 · min(width, height)`. Use the shorter side so a circle stays round on any aspect ratio.
3. Consecutive points are joined by straight segments. The line between them is what the beam "draws".
4. Goniometer mode for normal music: ordinary stereo music is mostly mono content, so `L ≈ R` and raw XY gives a 45° diagonal smear. Rotate by 45° to get the mid/side display: `x = (L − R) / √2`, `y = (L + R) / √2`. Mono content becomes a vertical line and stereo width spreads it sideways. Use raw XY only for tracks composed for the scope.

**2. Generated XY signals (no audio needed)**
1. Lissajous: `x = A · sin(a·ω·t + δ)`, `y = B · sin(b·ω·t)`. A rational ratio a/b gives a closed, still figure; an irrational ratio never closes and seems to rotate; δ changes the apparent viewing angle ([Wikipedia](https://en.wikipedia.org/wiki/Lissajous_curve)).
2. Do not use an irrational ratio to get rotation, because the figure also smears. Use a rational ratio plus a small detune `Δ` (0–2 Hz) on the x oscillator: `x = A · sin(2π(a·f0 + Δ)·t + δ)`. The figure stays crisp and rotates at Δ turns per second.
3. Wireframe shapes: store a one-stroke path (for example a cube's 12 edges walked as one path with retraced edges, about 16 vertices) as a small vertex list. A path parameter `s` runs 0…1 along the list at `f_draw` laps per second. Rotate the 3D points by yaw/pitch, project with `p = xy / (1 + z · 0.4)`, and keep `z` for brightness (front edges brighter).
4. Logos and text as one stroke: export the path from osci-render or any SVG tool as a point list. The retrace segments between letters stay visible, which is part of the look.
5. Morphing: crossfade two point sets with the same point count, `p = mix(shapeA, shapeB, m)`, with `m` eased over 1 beat.

**3. Beam integral (why the line looks like a scope)**
Taken from the [woscope write-up](https://m1el.github.io/woscope-how/).
1. The beam spot is a Gaussian with spread σ (woscope uses σ = beam size / 4).
2. The beam travels along a segment of length `l` in one sample period. Integrating the Gaussian over that motion gives a closed form. In segment-local coordinates (segment from (0,0) to (l,0), pixel at (px, py)): `I = (1 / 2l) · exp(−py² / 2σ²) · [erf(px / (√2σ)) − erf((px − l) / (√2σ))]`.
3. The `1/l` term is the physics: the same energy spread over a longer path means fast motion is dim and lingering is bright. Clamp `l` to at least σ·0.5 so stationary points do not blow up (woscope switches to a plain Gaussian for near-zero lengths).
4. Adjacent segments sum to a seamless line, so joins need no special handling. All segments are added up (additive blend).
5. Any cheap `erf` approximation is fine visually; the error is far below what bloom hides.

**4. Persistence (phosphor afterglow)**
1. Main method, history fade: draw more history than one frame and dim each segment by its age: `w = exp(−age / τ)`, with `age = (newest sample index − i) / sampleRate` in seconds. woscope fades by sample index in the same spirit.
2. Draw a window of `≈ 3τ`. With τ = 30 ms that is 90 ms of signal, about 4300 samples at 48 kHz.
3. Because the fade is in signal time, not in frames, the afterglow looks the same at 50, 60 or 120 fps.
4. Long smears (τ > 300 ms) would need too many segments. For those, add the feedback buffer (layer 4): `accum = accum · exp(−dt / τ_fb) + beam_this_frame`, with `dt` the real frame time. This buffer must be 16-bit float. In 8 bit, small decay factors get stuck at a grey residue that never reaches black.
5. Real phosphor glows hotter on the first pass. A short τ (10–20 ms) for the core plus a longer, dimmer feedback tail (`τ_fb` 0.3–1.5 s at 25 % weight) reads like a real P31-style screen.

**5. Sample rate versus frame rate**
1. At 48 kHz and 60 fps, 800 new sample pairs arrive per frame (3200 at the 192 kHz Fenderson used, per the research). Drawing only one point per frame, for example the latest sample, gives noise, not a shape. Every frame must draw the whole block of samples since the last frame, plus the history window.
2. Audio arrives in driver blocks (256–1024 frames) that do not line up with video frames. Keep a ring buffer of 8192 stereo frames on the CPU, upload it all each frame with the write-head index, and always draw a fixed window that ends at the write head. Whether one frame got 512 or 1024 new samples, the window is continuous and does not jitter.
3. Generated signals use a virtual sample rate. Evaluate the oscillators at `t_i = t_frame − (N − 1 − i) · dt_s`, with `dt_s = window / N`. The curve is then a function of time, not of frame count, so it plays at the same speed on any machine.
4. Segment budget: you need about 40 segments per oscillation cycle for a smooth curve. Audio-rate oscillators (f0 = 100 Hz) in a 90 ms window give 9 cycles, so about 400 segments. Higher f0 or a longer window means more segments. If the budget is tight, lower f0 into the visual range (f0 = window⁻¹, about 1 cycle per window). The ratio defines the shape, not the absolute frequency.
5. Decimation and interpolation: for real audio on the per-pixel route (Wire), take every 2nd to 4th sample (12–24 kHz effective). Sharp corners round off a little, which is close to a real CRT. For the opposite problem (too few points, kinked curves), woscope suggests sinc interpolation; linear or Catmull-Rom upsampling ×2 is the cheap version.

**6. Phosphor tone mapping**
1. Accumulated intensity `I` (0…∞) → `e = 1 − exp(−I · exposure)` (soft saturation, exposure 1–4).
2. Colour: `c = phosphor · e + white · smoothstep(0.7, 1.0, e) · 0.8`. Low intensity stays saturated green and the hot core turns whitish, as in woscope's green with small red and blue parts.
3. Optional depth or age tint: older segments shift 10–20 % toward the shadow colour so the tail cools.

## Shaders

Render order per frame: P0 signal → P1 beam → (P2 feedback) → P3 tone map → P4 post.

**P0 — Signal**
- Purpose: produce the point list for this frame.
- Real audio: the CPU writes the stereo ring buffer into a texture of 8192 × 1, RG32F (R = L, G = R), plus uniforms `head` (write index) and `sampleRate`. With ISF, an `audio` input arrives as an image with one row per channel, one sample per column, centred on 0.5 ([ISF spec](https://docs.isf.video/ref_json.html)), so map with `s = (texel − 0.5) · 2`.
- Generated: no texture. The point is computed inside P1 from the segment index (Techniques 2).
- Output: texture or nothing; resolution = sample count, not screen.

**P1a — Beam, geometry route (UE Niagara, Godot MultiMesh)**
- One instanced quad per segment, N = 1024–4096 instances, additive blend.
- Vertex stage: fetch points `p_i`, `p_{i+1}` (from the ring texture at `(head − N + i) mod 8192`, or from the generator). Build a quad covering the segment plus a 3σ margin on all sides: along axis `d = normalize(p_{i+1} − p_i)` (fallback (1,0) for very short segments), normal `n = (−d.y, d.x)`. Pass segment-local coordinates `(px, py)` and `l` to the pixel stage.
- Pixel stage: beam integral (Techniques 3, step 2) × age weight `exp(−age/τ)` × `intensity` × optional depth weight `0.4 + 0.6 · (1 − z)`. Output into a float target.
- Output: RGBA16F at output resolution. This is woscope's method and the most accurate.

**P1b — Beam, per-pixel route (Wire ISF, Godot canvas_item, single pass)**
- No vertex shaders in Wire ([Resolume ISF docs](https://resolume.com/support/en/isf)), so each pixel loops over the segments.
- Inputs: generated parameters (`ratio_a`, `ratio_b`, `detune`, `phase`, `f0`, `window`, `tau`, `width`, `intensity`, `scale`, `TIME`, `RENDERSIZE`), or an `audio` image if the host fills it.
- Per pixel: (1) map the pixel to signal space; (2) for i = 0…N−1 compute `p_i`, `p_{i+1}`, reject quickly if the pixel lies outside the segment's bounding box grown by 3σ, otherwise add the beam integral × age weight; (3) write the summed intensity to R. Reuse `p_{i+1}` as the next `p_i` so each point is computed once per pixel.
- N = 256–512. Render at half resolution (960×540) and let the bilinear upscale and bloom hide it. The beam is soft, so the loss is barely visible.
- Output: R16F (or RGBA8 if no float target; then do the tone map in the same pass).

**P2 — Phosphor feedback (optional, needs a persistent buffer)**
- Inputs: P1 output, previous `accum` (ping-pong, RGBA16F), uniforms `tau_fb` (0.3–1.5 s), `fb_mix` (0–0.4), `dt`.
- Per pixel: `accum = accum_prev · exp(−dt / tau_fb) + P1 · fb_mix`; final intensity `I = P1 + accum`.
- ISF: a pass with `PERSISTENT` and `FLOAT` set ([ISF spec](https://docs.isf.video/ref_json.html)). Support for this in Wire depends on the version, so test before relying on it.

**P3 — Tone map / colour**
- Inputs: intensity `I`, palette uniforms, `exposure`, `hue` (Laposky mode).
- Per pixel: Techniques 6, then black clamp `c = max(c − 0.01, 0)`.
- Output: RGBA8 at output resolution.

**P4 — Post**
- Bloom: threshold 0.35, intensity 1.4, wide radius (about 1.5 % of frame height) plus a tight one (0.3 %) if the tool allows two. The line is thin, so the threshold must be low.
- CRT option (small screens only): barrel distortion `uv' = uv + (uv − 0.5) · |uv − 0.5|² · 0.08`, vignette 0.25.
- Grain 1 % monochrome.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base / black | #000000 | background, must stay true 0 |
| shadow | #0A2A12 | graticule, cooled tail of old segments |
| phosphor (base) | #33FF66 | main beam colour, P31-style green |
| highlight / core | #E8FFF0 | saturated core, hot retrace points |
| accent amber | #FFB000 | alternate "amber scope" palette for breaks |
| accent cyan | #29E6FF | drop palette |
| accent magenta | #FF2E88 | second drop colour in Laposky mode |

- Mapping: intensity ramp shadow → phosphor → core via the saturation formula (Techniques 6). It is not a hue gradient: one hue whose intensity runs into white.
- Laposky mode (variation from the research, after his rotating colour filters): `phosphor = hsv(hue0 + t · 0.05 + age · 0.3, 0.85, 1)`. The colour turns slowly and the tail lags in hue, like a filter wheel turning in front of the screen.
- Over time: green for most of the set, amber on breaks, cyan on drops, Laposky mode for one phrase at the peak.
- LED walls: the background must be exactly 0 (black clamp), or the bloom haze lifts the whole wall. Cap the core at 90 % white. A single 1-px line is invisible from 20 m, so keep the beam at 2–4 px and let bloom spread it.

## Blending and compositing

- Beam segments add to each other (additive, ONE/ONE or SRC_ALPHA/ONE as in woscope). Retrace and slow spots build brightness by physics, not by a separate effect.
- Beam over background or footage: Add. Use Screen over bright footage to avoid clipping.
- As a matte: the beam's luma masks another clip, so footage shows only through the line. In Arena use the layer as a mask; in Wire use a mask node on the luma.
- Feedback mix: 0–0.4 of the beam into the accumulation buffer. Never feed the bloomed image back, because the halo would grow every frame until the screen is washed out.
- Post chain in order: tone map → black clamp → bloom → CRT curvature (optional) → grain.

## Motion and timing

- The signal is the motion. Base f0 is 40–400 Hz in audio mode, or 0.5–5 cycles per window in visual mode.
- Rotation: detune Δ 0–2 Hz. BPM lock: Δ = BPM / 60 / 4 gives one full turn per bar.
- Ratio steps (variation 2 of the research): on each downbeat of a phrase, step a:b through 1:1 → 1:2 → 2:3 → 3:4 → 3:5 → 4:5. Ease the switch by morphing over 1 beat (Techniques 2, step 5); a hard switch on the drop.
- Phase δ: LFO 0 → 2π over 2 bars, so the figure "turns over".
- Wireframe mode: yaw 1 turn per 2 bars, pitch sine ±25° over 8 bars, `f_draw` 30–120 laps per second (above 30 the shape stops flickering).
- Kick: scale +15 %, attack 5 ms, release 180 ms exponential.
- Build-up: τ 20 → 120 ms over 8 bars (trails grow), detune 0 → 1 Hz, bloom 1.0 → 1.8.
- Drop: ratio jumps (hard cut), palette to cyan, feedback on (fb_mix 0.3, τ_fb 0.8 s), width +1 px for 1 bar.
- Break: amber palette, a single ellipse (1:1, δ breathing slowly) or the goniometer view of the live music, τ 15 ms, feedback off. Very sparse, which makes the next drop hit harder.
- Change the shape family (Lissajous / wireframe / logo / live audio) at least every 16 bars. The research names a static ellipse as the main boredom risk.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| ratio | index 0–5 into 1:1, 1:2, 2:3, 3:4, 3:5, 4:5 | 2 (2:3) | which Lissajous figure | BeatStep Pro pads 1–6, or step per phrase |
| spin | detune 0–2 Hz | 0.25 | rotation speed of the figure | knob 1; BPM-lock switch (1 turn per bar) |
| afterglow | τ 5–300 ms (+ feedback on above 300) | 30 ms | trail length, from crisp line to long smear | knob 2; KeyStep mod strip for live sweeps |
| beam | width 1–6 px | 2.5 | line thickness and softness (σ) | knob 3; hats add +0.5 px |
| punch | scale 0.5–1.3 | 0.85 | figure size, kick bump on top | knob 4; kick envelope +0.15 |
| source | 0 generated / 1 wireframe / 2 live XY / 3 goniometer | 0 | which signal draws the picture | pads 7–8 cycle |

Intensity, exposure, palette and f0 are set once at soundcheck.

## Build recipes

### Resolume Arena

- I found no built-in XY scope source in Arena, so there are three routes:
  1. Generated figure: load the Wire-made effect/source (see Wire) on layer 2, blend Add over layer 1. Arena's audio analysis drives `punch` (Low band) and `beam` (High band) through the parameter's audio link, with Arena's own attack/release controls.
  2. External render: run the UE or Godot build (or the osci-render app with its software oscilloscope window screen-captured) and bring it in as a Spout or NDI source on layer 2, blend Add.
  3. Real scope: point a camera at a real analogue scope fed by an audio interface, and bring the camera in through a capture card. This is the most authentic look. Add a small Blur and a Levels black clamp to remove the camera's grey floor.
- Post on layer 2: Bloom (added in Resolume 7.2 together with Pixel Blur; [Resolume blog](https://resolume.com/blog/19880)). Raise it until the line glows without a grey wash.
- Persistence: keep it inside the shader (history fade). Do not count on a built-in trails effect.
- Custom ISF: the single-pass P1b + P3 combined is the candidate. Resolume documents ISF only in the context of Wire, so test loading in your Arena version before the gig.

### Resolume Wire

- No vertex shaders ([Resolume ISF docs](https://resolume.com/support/en/isf)), so the beam is the per-pixel route P1b inside an ISF node.
- Chain: parameters → ISF node "XYBeam" (P1b + P3 in one pass, rendered at 960×540) → Resize to output → Bloom → output. The single pass needs no buffers.
- Live stereo audio: the ISF spec defines an `audio` input type (one row per channel) ([ISF spec](https://docs.isf.video/ref_json.html)). Resolume's ISF page does not say whether Wire fills it, so test it. If it works, set `MAX` to 512 to decimate and use the goniometer rotation for normal music. If it does not, use generated signals and drive them with Wire's audio analysis values.
- Feedback tail (P2) as a second pass with `PERSISTENT` + `FLOAT` only if your Wire version supports multipass ISF. Otherwise skip it; the history fade alone covers τ up to about 300 ms.
- Control side: audio analysis (low band) → envelope (attack 5 ms, release 180 ms) → `punch`. BPM/beat → counter per phrase → `ratio`. BPM → `spin` when locked. LFO 2 bars → `phase`.
- Exposed parameters: ratio, spin, afterglow, beam, punch, source, palette (0 green / 1 amber / 2 cyan / 3 Laposky).

### Unreal Engine 5.7

Main route: a Niagara GPU sprite emitter, one sprite per beam segment (P1a).

- Emitter `NE_XYBeam`: GPU Compute Sim, Spawn Burst Instantaneous N = 2048 at t = 0, infinite lifetime (no Kill). Each particle is a segment with index `i = ExecutionIndex`.
- Update (Scratch Pad module): compute `p_i` and `p_{i+1}`.
  - Generated: Techniques 2 formulas, with `t_i = Engine.Time − (N − 1 − i) · Window / N` and User parameters `Ratio`, `Detune`, `Phase`, `F0`, `Window`, `Scale`, `Yaw`, `Pitch`.
  - Live audio: Niagara has an Audio Oscilloscope data interface that reads a submix ([UE forum](https://forums.unrealengine.com/t/niagara-oscilloscope-attribute-wont-package/472063)). I could not open Epic's API page to confirm how it exposes the left and right channels separately, so test that in 5.7 before building on it. The same thread reports packaged builds hanging on launch with this interface, with `[Audio] UseAudioThread=false` in DefaultEngine.ini as a user workaround. Test a packaged build, not just the editor. Fallback: feed the stereo ring buffer as a texture written from C++ and sample it with a texture data interface.
- Set Position = midpoint, sprite size = (l + 6σ, 6σ), and alignment along `d`. Sprite Renderer: Alignment = Custom Alignment with `SpriteAlignment = d`, Facing = camera. Pass `l` and `age` to the material with a Dynamic Parameter.
- Material `M_Beam`: Unlit, Additive. From the sprite UV, rebuild `(px, py)` in segment space, apply the beam integral × `exp(−age/τ)` × `Intensity` (default 6, so bloom picks it up), and multiply by the phosphor colour (or tone map in post).
- Camera: orthographic CineCamera (or a plane facing a perspective camera) so the XY plane is undistorted.
- Persistence tail (P2): SceneCapture2D of the beam into `RT_Beam` (RGBA16F), then each tick Draw Material to Render Target `RT_Accum = RT_Prev · exp(−dt/τ_fb) + RT_Beam · fb_mix` with two targets swapped. Display it on a screen-filling plane or in a post-process material.
- Post Process Volume (unbound): Bloom 1.4, threshold 0.35; manual exposure (Min = Max = 1); Vignette 0; AA off or FXAA (TSR smears fast thin lines into ghosts that look like fake persistence).
- Remote Control: a Remote Control Preset with the actor's exposed variables (`Ratio`, `Detune`, `Tau`, `Width`, `Scale`, `Source`, `Palette`). A Blueprint pushes them into the Niagara User parameters every tick. Bind them to OSC and MIDI (BeatStep Pro).
- Output: Spout or NDI sender plugin to Arena; the beam layer goes in Add mode.

### Godot

- Godot 4. Audio in: put an AudioEffectCapture on the music bus (or on a bus fed by an AudioStreamMicrophone for line-in, with audio input enabled in the project settings). `get_buffer(frames)` returns a PackedVector2Array of stereo frames, x = left, y = right, in −1…1. Check `get_frames_available()` first. Set `buffer_length` to about 0.05 s for low latency; it cannot change once the effect is initialised ([Godot AudioEffectCapture](https://docs.godotengine.org/en/stable/classes/class_audioeffectcapture.html)).
- Each `_process`: read all available frames into an 8192-frame CPU ring (PackedFloat32Array), write it to an Image (FORMAT_RGF, 8192 × 1) with `set_data`, update an ImageTexture, and pass the `head` index and the sample rate as uniforms. Do not call `set_pixel` per sample in GDScript.
- Beam (P1a): a MultiMeshInstance2D with a 1 × 1 quad mesh and instance count N = 2048, all instance transforms identity, inside a SubViewport (output size, HDR 2D on so the target is float). In the canvas_item shader, `render_mode blend_add, unshaded`. In `vertex()`, use `INSTANCE_ID` and `VERTEX_ID` to fetch the two points and place the quad corners ([Godot canvas_item shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/canvas_item_shader.html)). In `fragment()`, compute the beam integral × age weight. Generated mode computes the points from `INSTANCE_ID` and TIME, with no texture.
- If segments vanish at screen edges, culling is using the 1 × 1 mesh bounds. Enlarge the bounds or skip culling.
- Feedback (P2): a second SubViewport with clear mode Never, or two viewports in ping-pong. A full-screen ColorRect shader does `prev · exp(−dt/τ_fb) + beam · fb_mix`. Use HDR 2D, or the trail never fades to black.
- Final: a ColorRect with the P3 tone-map shader reading the viewport texture. WorldEnvironment glow (2D glow needs HDR 2D) or a custom two-tap bloom.
- Quick-test route: Line2D with the points updated per frame. It works but gives a uniform-brightness line with no beam physics.
- Uniforms to expose: `ratio_a`, `ratio_b`, `detune`, `phase`, `tau`, `width`, `intensity`, `scale`, `source`, `palette`, `fb_mix`. Map them from MIDI via Godot's InputEventMIDI, or from OSC through an addon.

### Note on osci-render

[osci-render](https://github.com/jameshball/osci-render) turns SVG, .obj, text, Lua scripts and Blender scenes into XY audio, with MIDI control and a software oscilloscope, as a standalone app or VST3/AU plugin. Its source is GPLv3, and its name, logos and branding are not covered by that licence. Using the app or plugin live to generate the XY signal (routed by a virtual audio cable into Godot or UE, or into a real scope) carries no obligation for your visuals. If you copy or port its code into a shader or tool you distribute, that work has to be released under GPLv3 too. This design is built from the published math and the woscope explanation, not from osci-render code.

## Audio reactivity

| band | drives | smoothing | threshold / scaling |
|------|--------|-----------|---------------------|
| kick, 40–120 Hz | punch: scale +0.15 | attack 5 ms, release 180 ms | trigger at 0.6 of the running peak (2 s window), retrigger blocked for 120 ms |
| bass, 60–250 Hz | amplitude A/B (0.7 → 1.0), or ratio step on strong hits in build-ups | attack 60 ms, release 500 ms | linear above a 0.2 floor |
| mids, 500 Hz–2 kHz | phase δ nudge ±0.3 rad | attack 50 ms, release 400 ms | continuous |
| hats, 6–12 kHz | beam width +0.5 px, XY jitter 0–0.008 (hash noise per segment) | attack 3 ms, release 60 ms | trigger at 0.5; jitter off in logo mode above 0.004 (text becomes unreadable) |
| RMS loudness | intensity 0.7 → 1.2 and bloom | attack 300 ms, release 2 s | drop sections only |
| raw stereo waveform | the picture itself (live XY / goniometer source) | none: the signal is drawn sample-exact | normalise with a slow peak follower (release 3 s), so quiet tracks still fill the screen |
| beat clock | ratio per phrase, spin lock per bar, palette per phrase | — | quantise to downbeats |

## Performance tips

- Geometry route (P1a): 2048–4096 quads with a few erf calls per covered pixel. The cost is overdraw: retraced and dense figures stack hundreds of additive quads on the same pixels. Keep σ small (width ≤ 6 px), since the quad area grows with σ.
- Per-pixel route (P1b): cost = pixels × N. At 960×540 with N = 512 that is about 265 million segment tests per frame, which only a desktop GPU handles comfortably. The bounding-box early-out skips most of the erf work. Measure in your Wire build. Fall back to N = 256 or 640×360 on laptops.
- Never draw only one sample per frame (sample rate versus frame rate, Techniques 5). Never decay by a per-frame factor without `dt`, or the afterglow length changes with frame rate.
- Accumulation and feedback buffers must be 16-bit float. 8-bit buffers leave a permanent grey ghost of the figure on LED walls.
- LED walls: minimum beam width 2 LED pixels. On low-resolution walls (≤ 1080 px tall at a large pitch) use 3–4 px and more bloom. Thin, fast, dim segments disappear first. Raise exposure rather than intensity so the core does not clip.
- Odd aspect ratios: scale by the shorter side so figures stay round. On ultra-wide strips (for example 5760×1080), place three figures side by side with phase offsets of 2π/3, rather than stretching one figure.
- Latency: the audio-to-picture delay is the capture buffer plus one frame. Keep the capture at ≤ 50 ms so kicks and figures stay in sync with the PA.
- Pitfalls: a mono track in raw XY (only a diagonal line, so use goniometer mode), DC offset in the signal (the figure drifts off-centre, so subtract a slow average), TSR/TAA ghosting in UE, and packaged-build hangs with the Niagara audio interface (test outside the editor).

## Further references

- [woscope: how it works](https://m1el.github.io/woscope-how/) — the Gaussian beam integral with erf, the 1/l brightness rule, additive blending and sample-index persistence used in this design.
- [ISF JSON reference](https://docs.isf.video/ref_json.html) — the `audio` input layout (row per channel, centred on 0.5) and the `PASSES` / `PERSISTENT` / `FLOAT` keys for the feedback pass.
- [Godot AudioEffectCapture](https://docs.godotengine.org/en/stable/classes/class_audioeffectcapture.html) — stereo frames as PackedVector2Array, ring buffer length and availability checks.
- [osci-render](https://github.com/jameshball/osci-render) — GPLv3 tool for making XY audio from SVG, 3D models, text and Lua, with a software oscilloscope.
