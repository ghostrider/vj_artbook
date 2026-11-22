# VHS Tape Degradation — design

## Look
Any source (live camera, Unreal via Spout, clips) looks like a worn rental tape playing on a tired deck. Colour is soft and smeared to the right of every edge, while brightness stays sharper but still low-res, with a faint bright halo from deck sharpening. Each scanline wobbles a pixel or two. Now and then a noisy tracking band rolls down the frame and tears the picture sideways. The bottom 3 % of the frame is always a skewed, noisy head-switching strip. White dropout flecks flash on single lines, and saturated edges crawl with tiny checkerboard dots and rainbow fringes. Blacks are lifted to a milky grey and highlights bloom softly. The mood is nostalgic and a little haunted, like a memory of a 1990s night out that is decaying while you watch it (compare the faux home video of Chase & Status "Blind Faith" and the found tape of Leckey's *Fiorucci Made Me Hardcore*, see research.md). The key rule is that it never sits still. A "wear" macro moves the tape from fresh to chewed over a set, so it does not read as a stock overlay.

## Layer stack
Bottom to top. In practice layers 2–9 are one multi-pass effect placed last in the chain.

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|--------------------|-----------|---------|-------|
| 1 | Source | any footage, camera, UE via Spout/NDI | Normal | 100 % | the effect is a treatment, so the source should be bright and saturated |
| 2 | Downsample | rescale to the tape work buffer (720×576 or 640×480 equivalent width) | — | — | the low-res base everything else works on |
| 3 | Time base | per-line horizontal jitter, tracking band displacement, head-switch skew | — | — | UV displacement only, no colour change |
| 4 | YIQ split | RGB to Y (luma) + I/Q (chroma) | — | — | blur and shift luma and chroma separately |
| 5 | Luma path | horizontal low-pass, then unsharp "deck sharpen" halo | — | — | soft but edgy luma |
| 6 | Chroma path | wide horizontal blur, delay (shift right), saturation loss | — | — | the colour bleed |
| 7 | Composite crosstalk | dot crawl + rainbow from a subcarrier pattern on edges | Add (into Y and I/Q) | 0–30 % | NTSC only, optional |
| 8 | Noise + dropouts | line-stretched luma noise, white dropout streaks, tracking-band snow | Add / Screen | 5–60 % (wear) | |
| 9 | Ghost | copy shifted 6–12 px right | Add | 8–20 % | multipath / ringing echo |
| 10 | Generation feedback (optional) | output fed back into layer 2 | Normal mix | 0–70 % | copy-of-a-copy decay |
| 11 | Output | upscale to wall res, scanline mask, vignette, grade | Multiply (scanlines) | 10–25 % | lifted black, soft clip |

## Techniques

**1. Low-res tape buffer and YIQ split** (from VHS bandwidth: luma up to ~3 MHz, chroma ~300 kHz, about 240 TV lines; [Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS))
1. Render the source into a buffer about 640 px wide (4:3) or 854 px (16:9) at 480 or 576 lines. Vertical stays at line count; tape smearing is horizontal.
2. Convert to YIQ: `Y = 0.299R + 0.587G + 0.114B`, `I = 0.596R − 0.274G − 0.322B`, `Q = 0.211R − 0.523G + 0.312B`.
3. Process Y and I/Q in separate horizontal-only filters. VHS damage is 1D along the scanline, so never blur vertically, except for a 1-line chroma average (step 4).
4. Average chroma over 2 lines (`IQ = mix(IQ[y], IQ[y−1], 0.5)`). PAL decoders and VHS comb filters smear colour vertically too, and this gives the soft blocky colour.

**2. Luma softening plus deck sharpen**
1. Horizontal box or Gaussian blur on Y, radius 1–3 px in the work buffer (`lumaSoft`).
2. Unsharp mask: `Y' = Y_blur + sharpen * (Y_blur − blur(Y_blur, 4 px))`, sharpen 0.4–1.2.
3. The overshoot gives the bright and dark halos next to edges that VCR "picture" knobs produced. Clamp Y to 0..1.1 so that halos can slightly clip.

**3. Chroma bleed and delay**
1. Wide horizontal blur on I and Q: 12–40 px at the work buffer, i.e. ~10× the luma radius, matching the ~1:10 bandwidth ratio.
2. Make the blur one-sided (sample only to the left, so colour trails to the right). This reads as "tape" much more than a symmetric blur.
3. Shift chroma right by `chromaDelay` 2–6 px.
4. Scale I/Q by saturation 0.6–0.9. Add a slow drift to the I/Q phase (`hueRot = 0.05 * sin(t * 0.3)` rad) for the colour wander of a bad deck.

**4. Time-base error: line jitter, tracking band, head switching** (tracking error definition: heads not following the tracks; [AV Artifact Atlas](https://www.avartifactatlas.com/artifacts/tracking_error.html); head switching: [Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS))
1. Line jitter: per scanline `dx = jitter * (noise1D(line * 0.15 + t * 3) − 0.5) * 4 px` plus white noise `± jitter * 0.5 px`. Use the line index, not uv.y, so it snaps to lines.
2. Tracking band: band centre `yb = fract(t / bandPeriod)` scrolling downwards (top to bottom in 1 bar at default). Weight `w = smoothstep(bandWidth, 0, |uv.y − yb|)` with bandWidth 0.03–0.12 of frame height. Inside: `dx += w * tracking * (noise(line, t*30) − 0.5) * 60 px`, and add snow (step 6) scaled by w.
3. Head switch: lines in the bottom `hsHeight` (2–4 % of height). Offset grows towards the bottom: `dx += hsSkew * pow(k, 1.5) * 40 px` where `k` 0..1 across the strip. Add strong noise and a 1–3 line black or white edge at the top of the strip. Bottom-edge flicker per frame.
4. Vertical roll (rare, on "chew"): `uv.y += roll * fract(t*2)`, with the black vertical blanking bar (8 lines) visible as it scrolls.
5. Apply all dx to the sampling uv in the time-base pass, before the YIQ processing, so bleed and halos follow the torn lines like on a real tape.

**5. Dot crawl and rainbows** (composite luma/chroma crosstalk; [Wikipedia: Dot crawl](https://en.wikipedia.org/wiki/Dot_crawl))
1. Subcarrier phase per pixel: `phi = x * fsc + line * π + frame * π`, with fsc about π/2 rad per work-buffer pixel (4 samples per cycle).
2. Chroma to luma (dot crawl): `Y += crawl * edgeC * (I cos phi + Q sin phi)`, where `edgeC` is the horizontal gradient magnitude of chroma, so dots appear only on colour edges.
3. Luma to chroma (rainbow): `I += rainbow * hpY * cos phi`, `Q += rainbow * hpY * sin phi`, with hpY = high-pass of Y (fine detail). The result is rainbow moiré on stripes and text.
4. Since the phase flips each frame and line, the pattern crawls upwards at frame rate. That movement is what sells it.

**6. Noise and dropouts**
1. Luma noise stretched along lines: sample white noise at `(x * 0.25, line)`, so grains are 4 px wide and 1 line tall. Amplitude `noise * 0.08`.
2. Dropouts: per line per frame, `hash(line, frame) > 1 − dropRate * 0.01` turns on a streak. Its start x is random, its length 10–120 px. It is bright (Y 0.9–1.0, desaturated) with a 3 px dark tail. dropRate 0–5 % of lines.
3. Tracking snow: in the band weight w, replace up to 70 % of the pixel with noise `mix(Y, noise, w * tracking * 0.7)`.

**7. Generation loss (feedback)**
1. Ping-pong two work buffers: A = this frame's output, B = last frame's output.
2. Before the time-base pass: `in = mix(source, B, generations * 0.7)`. Every frame, B is degraded again, so bleed and halos stack into copy-of-a-copy mush.
3. Keep the mix ≤ 0.7, or the image burns into a static smear. Add a 0.5 px right shift per generation so trails drift instead of freezing.

## Shaders
Render order per frame. Work buffer is W×H (default 854×480 for 16:9, 640×480 for 4:3; use 720×576 for PAL feel).

| pass | purpose | inputs | per pixel | output |
|------|---------|--------|-----------|--------|
| P0 Downsample + feed | low-res base, generation mix | source texture, B (prev output), `generations` | bilinear sample source at work res; `mix(src, B, generations*0.7)` with B sampled at uv − (0.5 px, 0) | RGBA16F W×H |
| P1 Time base | line jitter, tracking band, head switch, roll | P0, `time`, `jitter`, `tracking`, `bandPeriod`, `bandWidth`, `hsHeight`, `chew`, `frame` | compute line index; sum dx from technique 4; sample P0 at `uv + (dx/W, roll)`; output also band weight w and head-switch mask k in alpha or a second target | RGBA16F W×H (+ R8 mask) |
| P2 Encode + luma | YIQ split, luma blur and sharpen | P1, `lumaSoft`, `sharpen` | RGB to YIQ; 7–9 tap horizontal blur on Y; unsharp mask; write Y, I, Q | RGBA16F W×H |
| P3 Chroma | bleed, delay, vertical chroma average, saturation, hue drift | P2, `bleed`, `chromaDelay`, `saturation`, `time` | one-sided horizontal blur on I/Q, 16–24 taps spread over bleed px (or two passes of 12); shift by chromaDelay; average with line above; rotate IQ by hueRot; scale | RGBA16F W×H |
| P4 Composite + damage | dot crawl, rainbow, noise, dropouts, snow, ghost, decode | P3, P1 mask, `crawl`, `rainbow`, `noise`, `dropRate`, `ghost`, `frame` | technique 5 crosstalk; add noise and dropouts; snow by w; ghost `Y += ghost * Y(x − 8 px)`; YIQ to RGB; grade (see colour) | RGBA16F W×H, copied to B for next frame |
| P5 Output | upscale to wall, scanlines, vignette | P4, `scanlines`, output res | bicubic or bilinear upscale (horizontal can stay soft); scanline multiply `1 − scanlines * 0.5 * (1 − cos(2π * uv.y * H))`; vignette 0.15; optional 4:3 pillarbox | output res |

Ping-pong: A/B swap after P4. Clear B on clip change, or a ghost of the previous clip stays (sometimes this is a nice feature: map a "clear tape" button).

## Color scheme
The colour comes from the source, so the palette is a grade, not a fixed set.

| role | hex | use |
|------|-----|-----|
| Black level (shadow) | `#141216` | lifted, slightly violet black: `rgb = mix(#141216, rgb, ...)` floor at about 7 % |
| Base midtone bias | `#8A7F86` | slight magenta-grey cast in mids from tape chroma drift |
| Highlight | `#F2EEE4` | soft clip, warm off-white; no pure #FFFFFF except dropouts |
| Dropout flash | `#FFFFFF` | the only full white in the frame |
| Accent fringe A | `#FF3E7A` | red/magenta side of chroma bleed (I+) |
| Accent fringe B | `#2EE6D6` | cyan side (I−) |

- Mapping: work in YIQ. Saturation 0.75, I/Q rotation ±0.05 rad drift, and the grade as `Y = 0.07 + Y * 0.88` with a soft shoulder above 0.85 (`Y = 0.85 + (Y − 0.85) * 0.4`).
- Optional "warm tape" tint: I += 0.02 (towards orange). "Cold tape": Q += 0.02.
- Over time, the wear macro lowers saturation from 0.9 to 0.55 and raises the black floor from 5 % to 10 %. The image fades like an old tape.
- LED walls: lifted blacks on LED read as bright grey and kill contrast in a dark room. Keep the floor at 5–7 % on LED (higher is fine on projection), and pull overall output to 85 % so dropout flashes do not blind the front row.

## Blending and compositing
- The effect is one chain placed **last**, so every layer below inherits the tape. In Resolume, put it on the Composition (or on a group at the top).
- Inside the chain, tracking and bleed happen *after* time-base displacement, so colour smears follow the tears.
- Ghost: Add at 8–20 %, 6–12 px right. Above 25 % it looks like a stereo error, not tape.
- Generation feedback mix 0–0.7. Default 0.
- Selective tape: a mask (Resolume layer mask, Wire shape, UE stencil) lerps between the clean source and the taped output: `out = mix(clean, tape, mask)`. Feather the mask edge horizontally 20 px so the border also looks smeared.
- Post chain order: upscale, then scanlines (Multiply 10–25 %), then a light bloom (threshold 0.8, intensity 0.3, radius 1 % of height, for highlight glow), then vignette 0.15, then **no** extra grain (the noise is already in P4), then **no** sharpen at output (sharpening is inside the luma path).

## Motion and timing
- Line jitter: always on, low (0.1–0.3), changes every frame. Never zero, or the image looks digital.
- Tracking band: scrolls top to bottom, one pass per bar at default (`bandPeriod = 60/BPM*4` s). Range half a bar to 4 bars. Ease: linear scroll, but band strength `tracking` breathes with a slow sine over 8 bars (0.2 to 0.6).
- Hue drift: 0.3 rad/s sine, very slow.
- Head-switch strip: constant, noise flickers per frame; skew pumps 1.0 to 1.5 on each beat with a 120 ms decay.
- Kick: a short horizontal tear. `jitter` spikes to 1.0, attack 0, release 150 ms.
- Snare / clap: one dropout burst (dropRate to 8 % for 2 frames).
- **Chew** (pad, one-shot, 0.5–1 bar): tracking to 1.0, roll 1 frame-height scroll, band width 0.3, saturation 0.3, then ease out over 1 beat (exponential). Use it on the transition into the drop.
- **Drop**: fire chew on the last beat before the drop, then hit the drop *clean-ish* (wear −30 %) for 1 bar, then let wear creep back. Contrast sells the drop.
- **Break**: wear up slowly (+0.2 over 8 bars), generations to 0.4, pause-frame freeze (hold B, set source mix 0) on the one for a "paused tape" look with heavy jitter and a static head-switch strip.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| wear | 0–1 | 0.35 | macro: scales noise, dropRate, bleed, saturation loss, black lift together; fresh rental to chewed tape | KeyStep 37 mod strip or a fader; slow automation across the set |
| tracking | 0–1 | 0.3 | strength of the rolling tracking band and its snow and tear | BeatStep Pro knob 1; band period locked to 1 bar |
| bleed | 0–40 px (work res) | 18 | chroma smear width and delay | BeatStep Pro knob 2; bass envelope adds +0–15 |
| jitter | 0–1 | 0.2 | per-line wobble and kick tears | kick envelope (attack 0, release 150 ms) on top of the knob value |
| generations | 0–0.7 | 0 | copy-of-a-copy feedback decay | fader; raise in breaks |
| chew | trigger | off | one-shot tape-eating glitch for 0.5–1 bar | BeatStep Pro pad (note), or the last beat of every 16-bar phrase |

Internal (not on stage, set per venue): lumaSoft 2 px, sharpen 0.7, chromaDelay 3 px, crawl 0.15, rainbow 0.1, hsHeight 3 %, ghost 0.12, scanlines 0.15.

## Build recipes

### Resolume Arena
- **Quick path (no custom code):** on the Composition, stack in this order: **Reducto** (lo-fi colour and resolution reduction), **Shifty** (horizontal line distortion, small amount, smoothing on), **VHSifyer** (scanlines, slight distortion, chromatic aberration), **Blur** (low), **Vignette**. Add **Screen Shake** with its trigger synced to BPM as the kick tear. Resolume introduced Reducto, Shifty, VHSifyer and Screen Shake as built-in effects ([Resolume blog: 9 new effects](https://www.resolume.com/blog/23113)). This gets roughly 60 % of the look. It has no one-sided chroma bleed or tracking band.
- **Full path:** one custom effect "VHS Tape" containing P0–P5. Resolume reads ISF shaders ([Resolume: ISF](https://resolume.com/support/en/isf)). ISF supports multi-pass and persistent buffers, so P0–P4 map to PASSES, with B as a `PERSISTENT` buffer. Expose the six stage parameters as ISF inputs. They then appear as effect sliders you can map with right-click > MIDI map.
- BPM: set the tracking band period with the parameter's BPM sync (Timeline mode with "beats" duration) or by feeding `bandPeriod` from a Wire BPM node.
- Audio: on `bleed` and `jitter`, use the parameter's audio FFT option (bass for bleed, low for jitter) with Gain and Fall as in Audio reactivity.
- Chew: map a BeatStep Pro pad to a Dashboard button. It triggers a short clip on a top layer containing only the effect with chew = 1, or a Wire trigger input.
- Output: Advanced Output to the LED wall; also Spout out if UE/TouchDesigner should take the taped image.

### Resolume Wire
Build as an Effect patch (effect input, effect output).
- Nodes: **Effect Input (texture)**, then a **Resize/Render to texture** at work res, then **GLSL Shader** node (P1 time base), then **GLSL Shader** (P2), then **GLSL Shader** (P3), then **GLSL Shader** (P4), then **Resize** to output res, then **GLSL Shader** (P5), then **Effect Output**.
- Feedback: a **Feedback** node takes P4 output and returns last frame's texture into P0's mix input.
- Time and beat: a **Time** node into all shaders, a **BPM / Beat** node into `bandPeriod` and into an **Envelope** (AD, attack 0 ms, decay 150 ms) for jitter spikes. A **Trigger** input for chew drives an envelope with 0 ms attack and 1-beat release.
- Expose as patch inputs: wear, tracking, bleed, jitter, generations, chew, plus a "clear tape" trigger that resets the Feedback node.
- The wear macro: one float input that feeds **Map/Remap** nodes: noise 0.02–0.15, dropRate 0–5 %, bleed +0–20 px, saturation 0.9–0.55, black floor 0.05–0.10.
- Node names vary between Wire versions. Use the equivalents in your build; the connection order above is what matters.

### Unreal Engine 5.7
- **Post Process Material**, Material Domain Post Process, Blendable Location *After Tonemapping* (so grading is in display space). Put it in a Post Process Volume (Infinite Extent) as a Blendable.
- Inputs: SceneTexture:PostProcessInput0. Do the time-base dx (technique 4) with **ScreenPosition**, **Floor** (line index = floor(uv.y × 480)), **Noise** or a tiled noise texture, then **Add** to UV.
- Horizontal blurs (luma, one-sided chroma): a **Custom** node loop of 16 taps along x on PostProcessInput0, using the displaced UV. Do the YIQ conversion with a 3×3 matrix multiply (three **Dot** nodes).
- Damage: noise texture (Texture Sample with tiled 256² white noise, UV scaled ×(0.25, 1) per line), **Time** for the frame index (Floor(Time × 30)).
- Generation feedback: a Blueprint uses **Draw Material to Render Target** to write the result into RT_A each tick and reads RT_B. Swap them, and lerp the post material input with RT_B.
- Parameters: a **Material Parameter Collection** `MPC_VHS` with Wear, Tracking, Bleed, Jitter, Generations, Chew, BandPeriod. Expose them to **Remote Control** (Remote Control Preset, OSC and MIDI protocols bound to the same fields): BeatStep Pro CCs for knobs, a pad note to Chew. Feed beat envelopes over OSC from Resolume or a BPM clock.
- Output: Spout out via the Off World Live plugin, or NDI via the NDI SDK for Unreal, into Resolume. Alternatively run the scene clean into Resolume and apply the tape there, which keeps one tape treatment over all sources.

### Godot
- Godot 4: **CanvasLayer** (top), then a full-rect **ColorRect** with a `canvas_item` shader reading `hint_screen_texture` for a single-pass version (P1–P4 folded into one shader, blur taps inline).
- Multi-pass: chain **SubViewport**s, one per pass (P1, P2/P3, P4) at the work res (854×480), each with a ColorRect + shader reading the previous SubViewport's **ViewportTexture**. Set the last one to `render_target_update_mode = Always`.
- Feedback: a second SubViewport holds last frame's P4 (`render_target_clear_mode = Never`), fed back as `prev_tex`.
- Final: a TextureRect at window size with `texture_filter = Linear`, and the P5 scanline shader.
- Uniforms: `wear`, `tracking`, `bleed`, `jitter`, `generations`, `chew`, `band_period`, `time` (or use built-in TIME). Drive them from a GDScript reading MIDI via `InputEventMIDI`.
- The [Godot Shaders VHS Glitch](https://godotshaders.com/shader/vhs-glitch/) page is a starting point for the bar shift and channel offset part.

## Audio reactivity
| band | drives | smoothing | threshold |
|------|--------|-----------|-----------|
| Kick (40–100 Hz onset) | jitter spike to 1.0, head-switch skew ×1.5 | attack 0 ms, release 150 ms | onset at −12 dB relative to running peak |
| Sub / bass (30–120 Hz RMS) | bleed +0–15 px, slow line wobble amplitude | attack 80 ms, release 400 ms | none (continuous), gain so a full bass line reaches +10 px |
| Snare / clap (150–400 Hz + 2–5 kHz onset) | dropout burst (dropRate 8 % for 2 frames) | instant, 2 frames hold | onset only |
| Hi-hats (6–12 kHz RMS) | noise amount +0–0.05, crawl +0–0.15 | attack 20 ms, release 120 ms | gate at −30 dB so silence stays clean |
| Overall loudness (slow) | wear offset ±0.1 | attack 2 s, release 6 s | none |

Do not drive tracking from audio directly. It becomes nervous noise. Keep it on bars and the chew pad.

## Performance tips
- The work buffer is small (854×480 ≈ 0.4 MP), so P1–P4 cost little even with 24-tap blurs. Total under 1 ms on a mid GPU. Only P5 runs at wall res and it is a single tap plus math.
- Do not run the blurs at 4K. Besides wasting GPU, it destroys the look, because bleed in px must be relative to ~480 lines. If you must work at higher res, scale every px value by `H / 480`.
- Use 16-bit float buffers so halos (Y > 1) and negative I/Q survive between passes. 8-bit clips chroma and makes bleed banded.
- Odd aspect ratios (very wide LED strips, 5760×540): keep the work buffer at 480 lines and scale its width to the aspect. Line jitter then still reads as scanlines. On tall portrait walls, use 640×N with N up to 1136, but lower line jitter, as it gets busy.
- Scanlines on LED: the scanline period must not beat with the LED pixel pitch. If you see moiré, set scanline count to an integer divisor of the wall's pixel height or turn scanlines off. LED pixels already give structure.
- Dropout white flashes on huge walls are intense: cap the dropout luma at 0.9 and the overall output at 85 %.
- Clear the feedback buffer on clip changes, or old content leaks for seconds.

## Further references
- [ntsc-rs](https://github.com/valadaptive/ntsc-rs): open-source Rust NTSC/VHS artefact emulator (standalone, OpenFX, After Effects/Premiere). Good for reading pass order and parameter ranges, or for pre-rendering clips.
- [TD-VHShader](https://github.com/ogrew/td-vhshader): TouchDesigner project combining 12 VHS shaders, useful as a module-by-module reference.
- [Godot Shaders: VHS Glitch](https://godotshaders.com/shader/vhs-glitch/): bar shift, line noise and channel offset in one shader.
- [AV Artifact Atlas: Tracking error](https://www.avartifactatlas.com/artifacts/tracking_error.html): what real tracking errors look like, for matching the band.
