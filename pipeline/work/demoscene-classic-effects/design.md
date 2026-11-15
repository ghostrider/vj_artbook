# Demoscene Classic Effects — design

## Look
A hard, bright 90s demo screen built from three formula effects stacked into one image. At the back a chunky plasma flows in saturated hue bands, quantised to a 16-step palette so it reads as cycling colour, not a smooth gradient. Across it, five to eight metallic copper bars, each a 9-step shaded band from dark to white-hot core, bob on sine paths and cross each other. In the middle a rotozoomed tile (a checker, a logo or a live feed) spins and breathes in zoom, shown through a mask or as a centre panel. On the drop everything flips to a texture-mapped tunnel flown at speed. The whole frame goes through a pixelate pass (320×200 feel) and a light scanline. The mood is cheerful and mechanical: everything moves in sines, everything locks to the beat. There is no single reference image; think of the Trixter "Demos Explained" screenshots of plasma, tunnel and copper ([scheib.net](https://www.scheib.net/play/demos/what/trixter/demo_graphics.html)).

## Layer stack
| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #000000 | Normal | 100 % | true black so bars and plasma pop on LED |
| 2 | Plasma | 4-cosine sum → 16-step palette lookup, palette offset cycles with beat | Normal | 100 % | the "floor" of the image; can be dimmed to 60 % under bars |
| 3 | Tunnel (scene B) | polar lookup tunnel, checker or logo texture | Normal (crossfade with 2–4) | 0 % / 100 % | switched in on drop, scene A otherwise |
| 4 | Rotozoomer | rotated + scaled tiled texture, wrap = repeat | Normal through mask, or Multiply over plasma | 100 % | mask: centre rectangle 60 % wide, or the plasma value > 0.5 as a matte |
| 5 | Copper bars | N bars, centre y = sine per bar, 9-step shade gradient | Add (or Normal with back-to-front sort) | 100 % | Add gives the glowing overlap; Normal with sort gives the "true" Amiga look |
| 6 | Shadebob trail (optional) | soft dots on Lissajous paths into a feedback buffer | Add | 70 % | feedback decay 0.92 per frame |
| 7 | Post: pixelate | snap UV to a 320×180 grid (or 160×90 for "chunky") | — | 100 % | nearest-neighbour, keeps the hard pixel edge |
| 8 | Post: posterise / palette snap | quantise each channel to 8 levels | — | 100 % | hides banding, sells the low colour count |
| 9 | Post: scanlines | darken every 2nd output-pixel row of the low-res grid by 25 % | Multiply | 30–60 % | flicker ±5 % on hi-hat |

## Techniques

### Plasma (after Bonz, Hugi 24)
1. Normalise pixel coordinates: `p = (fragCoord - 0.5 * res) / res.y`, so the pattern is not stretched on wide walls.
2. Sum four cosines with different frequencies, speeds and phases: `v = cos(a*p.x + s1*t) + cos(b*p.y + s2*t) + cos(c*(p.x+p.y) + s3*t) + cos(d*length(p - c0(t)) + s4*t)`, where `c0(t)` is a slow Lissajous centre. Bonz's rule: never give two terms the same value, or the pattern goes regular ([Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)).
3. Default frequencies a=3.1, b=4.7, c=2.3, d=6.0; speeds s1..s4 = 0.31, 0.47, 0.23, 0.61 (in beats, see Motion).
4. Map `v` from [-4, 4] to [0, 1]: `u = v / 8 + 0.5`.
5. Palette cycling: `idx = fract(u + paletteOffset)`, then quantise `idx = floor(idx * 16) / 16` and look up a 16-entry ramp. The motion comes mostly from `paletteOffset`, as in the 1988 original, which cycled the hardware palette ([Wikipedia](https://en.wikipedia.org/wiki/Plasma_effect)).

### Copper bars
1. Work in normalised y (0 at top, 1 at bottom of the screen).
2. For bar i of N: centre `cy_i = 0.5 + amp * sin(t * speed + i * spread)`, with spread = 0.55 rad and amp = 0.35 by default.
3. Distance `d = abs(y - cy_i) / halfHeight` (halfHeight 0.035). If d < 1, shade index `k = floor((1 - d) * 9)` into a 9-step ramp from dark edge to white core: this is the "several gradated colours" that make a metallic bar ([Wikipedia: Raster bar](https://en.wikipedia.org/wiki/Raster_bar)).
4. Each bar gets its own hue (rainbow across i) but shares the shade profile.
5. Draw order: sort by `cos(t * speed + i * spread)` (fake depth) so bars going "back" are drawn first; or simply Add for the glow variant.
6. Kefrens-style variant: replace horizontal bars with one vertical bar whose x follows a sine per scanline, and do not clear between lines (needs the feedback buffer).

### Rotozoomer
1. Centre coordinates: `q = p - centre`.
2. Rotate and scale back into texture space: `uv.x = (q.x cos a - q.y sin a) / k`, `uv.y = (q.x sin a + q.y cos a) / k` ([Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)).
3. Zoom `k = base + depth * sin(t)`, base 1.5, depth 1.0 (Bonz's `1.5 + sin(t)`); add kick pulse on top.
4. Sample with `fract(uv)` (repeat wrap), nearest filtering for the hard texel look.
5. Rotation `a` advances by a fixed angle per beat (default 22.5°), eased over the beat.

### Tunnel
1. Polar coordinates from the (Lissajous-moved) centre: `r = length(q)`, `phi = atan(q.y, q.x)`.
2. Texture lookup: `u = phi / (2π) + twist * t`, `v = depth / r + speed * t` (depth constant 0.3). This is the screen-to-texture table of the classic tunnel, computed live instead of stored ([Wikipedia: Demo effect](https://en.wikipedia.org/wiki/Demo_effect)).
3. Shade by distance: multiply the colour by `clamp(r * 2.5, 0, 1)` so the vanishing point fades to black.
4. Move the centre on a Lissajous path `(0.15 sin(1.0 t), 0.1 sin(1.5 t))`; unequal frequencies give the wandering path, equal ones a circle ([Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)).

### Shadebobs (optional)
1. Each frame, add 3–6 soft discs (radius 0.04, gaussian falloff) at Lissajous positions into a feedback buffer.
2. Buffer = previous × decay (0.92) + new discs; clamp.
3. Map buffer brightness through the same 16-step palette, so overlapping trails climb the ramp like the original additive bobs ([Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)).

## Shaders
Render order: A → B → C → D → E. All generator passes run at the low internal resolution; only E runs at output resolution.

| pass | purpose | inputs | per pixel | output / res |
|------|---------|--------|-----------|--------------|
| A Plasma | background colour field | uniforms `beatPhase`, `freqA..D`, `paletteOffset`, `paletteId`; 16×1 palette texture (or 3 hex sets as uniforms) | normalise p → 4-cosine sum → map to 0..1 → add offset, fract, quantise 16 → palette lookup | RGBA8, 320×180 |
| B Tunnel | drop scene | `beatPhase`, `tunnelSpeed`, `twist`, tile texture | centre on Lissajous → polar → u,v lookup with repeat → distance shade | RGBA8, 320×180 |
| C Rotozoom + mask | middle layer | `angle`, `zoom`, tile texture (checker, logo, Spout input), plasma from A for the matte | rotate/scale → fract → nearest sample; alpha = centre rect or step(0.5, plasmaValue) | RGBA8, 320×180 |
| D Bars + bobs | foreground | `barCount`, `barAmp`, `barSpeed`, `barHeight`; feedback buffer (ping-pong) for bobs | loop over N ≤ 12 bars → shade index → colour; bobs: read prev buffer × decay + discs, write new | RGBA8 (bobs RGBA16F ping-pong), 320×180 |
| E Composite + post | final image | A–D, `sceneMix`, `pixelSize`, `scanlineAmt` | mix A/C/D (scene A) vs B (scene B) by `sceneMix` → nearest upscale → posterise 8 levels → scanline multiply on low-res row parity | output res (e.g. 1920×1080 or wall native) |

Rendering at 320×180 and upscaling with nearest-neighbour gives the pixel look for free and makes the whole chain nearly free on the GPU.

## Color scheme
Three switchable 16-entry palettes (selected by `paletteId`):

- **Amiga rainbow** (default): #000000 (shadow), #2B0F54, #6B1FB1 (base), #C71585, #FF2E63 (accent), #FF7F11, #FFD319 (highlight), #FFFFFF (core), then mirrored back down so the cycle has no seam.
- **Copper metal** (bar shade ramp, 9 steps): #0A0A1A, #1E2A5A, #2F4F9F, #4F7FDF, #8FB7FF, #CFE3FF, #FFFFFF core, mirrored. Per-bar hue: rotate this ramp in HSV by `i / N`.
- **Fire / C64 warm**: #000000, #3B0A00, #8B1A00, #D4430B, #FF8A00, #FFD27F, #FFFFFF.

Mapping: palette texture lookup with nearest filtering (no interpolation, that is the point). Over time: `paletteOffset` advances 1/16 per beat (one palette step per beat, a full cycle every 4 bars); a hi-hat can add a single extra step. On the drop, switch `paletteId` and invert the cycle direction.

LED wall: keep black at #000000, not lifted; limit the brightest flat areas (plasma) to 80 % value so the white bar cores remain the peak. Contrast comes from the hard steps, so do not add bloom on top of the plasma.

## Blending and compositing
- Plasma is Normal at the bottom; it is opaque and fills the frame.
- Rotozoomer sits on plasma either inside a centre mask (clean, "demo part" look) or Multiplied (tile becomes a darkening pattern in the plasma colours). The plasma-threshold matte (`step(0.5, plasma)`) makes the tile appear inside the plasma blobs; good for breakdowns.
- Bars on top in Add for glowing crossings, or Normal with depth sort for the faithful look. Add bars over a plasma at 100 % will clip to white; dim plasma to 60 % in that mode.
- Shadebob feedback: decay 0.90–0.96 (0.92 default); mix into the composite with Add at 70 %.
- Scene switch A ↔ B: hard cut on the beat (true demo style), or a 1-beat crossfade.
- Post chain order: pixelate (nearest) → posterise 8 levels → scanlines (Multiply 40 %) → optional 1 px chromatic shift of R/B on kick only. No blur, no bloom on the chunky version; optional soft bloom (threshold 0.85, intensity 0.6) for a "modern" variant without pixelation.

## Motion and timing
- Global time `t = beatPhase * 2π / 4`, so one sine period = one bar (4 beats) at default speed multipliers. All speeds are multiples of 1/4, 1/2, 1, 2 of that.
- Plasma: term speeds 0.23–0.61 per bar, slow; real motion from palette cycling (1 step per beat).
- Copper bars: one full bob cycle per bar; on kick, bar halfHeight jumps +40 % and eases back over 120 ms (exponential).
- Rotozoomer: rotation +22.5° per beat with ease-out over 60 % of the beat; zoom breathes one cycle per 2 bars; kick adds +0.3 to `k` decaying over 150 ms.
- Tunnel: base speed 0.5 texture repeats per beat; drop raises to 2.0; twist 0.05 rev per bar.
- Phrase (16 bars): change palette, or swap rotozoom tile.
- Break: freeze palette cycling, bars to N=2, slow everything to 1/2 speed, drop pixelation to 160×90 (chunkier) for tension.
- Drop: hard cut to tunnel scene, palette switch, pixelSize back to 320×180, bar count up to 8.

## Parameters
| name | range | default | what it does on stage | suggested mapping |
|------|-------|---------|-----------------------|-------------------|
| `cycle` (palette speed) | 0–4 steps/beat | 1 | how fast colour flows through plasma and bars | BeatStep Pro knob; hi-hat adds +1 step |
| `warp` (plasma freq scale) | 0.3–3.0 | 1.0 | multiplies a,b,c,d: big soft blobs ↔ fine busy interference | KeyStep mod strip or fader; bass envelope ±20 % |
| `bars` (bar count) | 0–12 | 6 | number of copper bars | KeyStep keys (C = 0 … one bar per semitone) or knob |
| `spin` (roto rotation per beat) | 0–90° | 22.5° | rotozoomer angle step per beat | knob; sign flip on a pad |
| `pulse` (kick amount) | 0–1 | 0.5 | how much kick swells bars, zoom and tunnel speed | knob; kick envelope drives it |
| `chunk` (pixel size) | 1–8 output px (or grid 640→80 wide) | grid 320 | from clean to very chunky | fader; pad for 160×90 on breaks |
| `scene` | 0–1 | 0 | plasma/roto/bars ↔ tunnel | BeatStep Pro pad, hard cut quantised to the bar |

## Build recipes

### Resolume Arena
- Layer 1 (bottom): a Solid Color source #000000.
- Layer 2: Plasma. Either the built-in Generator "Plasma" if your version has it, or a custom ISF generator implementing pass A (preferred, gives palette quantise and beat phase). Map palette speed to the clip's BPM-sync parameter via Resolume's parameter animation (Timeline → BPM sync).
- Layer 3: Rotozoomer. Put a checker or logo clip (or a Spout source from UE/TD) here and use a custom ISF effect implementing pass C. Without ISF: the built-in Transform effect, Rotation set to BPM Sync (one turn per 4 bars) and Scale animated with a sine LFO 100–250 %, plus Tile effect for the repeat. Blend Multiply or use a Mask clip above it.
- Layer 4: Copper bars as an ISF generator (pass D), blend Add.
- Layer 5 (or a second deck/column): Tunnel ISF generator (pass B). Use Columns: column 1 = scene A, column 2 = scene B, trigger with a BeatStep Pad; set Composition → Clip Trigger quantisation to 1 bar.
- Composition effects, in order: Pixelate (built-in, cell size 6 for 1920 width ≈ 320 cells), Posterize (levels 8), then a custom scanline ISF or a static scanline PNG on the top layer in Multiply at 40 %.
- Audio: on the bars layer, link `pulse` to Audio FFT low band via the parameter's Audio dashboard; attack 0, release 150 ms.
- MIDI: Shortcuts → MIDI map the parameters in the table to KeyStep/BeatStep. REST API ([docs](https://resolume.com/docs/restapi/)) can switch columns from an external sequencer if needed.

### Resolume Wire
- Inputs: `Beat Phase`/`BPM` node from Resolume host (Time node with BPM), exposed parameters `cycle`, `warp`, `bars`, `spin`, `pulse`, `chunk`, `scene`; an Audio input or FFT node for the kick band.
- Plasma: a UV/Texture Coordinates node → four branches of Multiply (freq) + Add (time × speed) → Cos → Add all → Multiply 0.125 + 0.5 → Add `paletteOffset` (Fract) → Floor(x*16)/16 → Gradient/Ramp or Sample Texture (16×1 palette image) → texture out.
- Bars: a Loop/For node over N (or a GLSL/ISF shader node with pass D) → y distance → step ramp → colour.
- Rotozoom: Transform 2D (rotate by accumulated beat angle, scale by zoom) → Sample Texture with Repeat wrap, nearest filter.
- Tunnel: a GLSL shader node with pass B (polar maths in nodes is messy).
- Composite: Blend nodes (Normal, Multiply, Add) as in the stack → Render at 320×180 → Resize with nearest filtering → Posterize → scanline Multiply → output. Expose all seven parameters on the patch so they appear as Resolume effect/source parameters.

### Unreal Engine 5.7
- Material approach (main): one Unlit Post Process Material or a Material on a full-screen plane/Render Target. Custom HLSL nodes for plasma, bars, roto and tunnel (or node graph: TexCoord → Rotator node for roto, VectorToRadialValue node for tunnel polar coords, Cosine nodes for plasma). Palette as a 16×1 texture with Nearest filter and Clamp/Wrap.
- Pixelation: render into a Render Target 320×180 (Texture Filter = Nearest) via a Scene Capture or Draw Material to Render Target in a Blueprint tick, then display upscaled; or in the post material: `uv = floor(uv * grid) / grid`.
- Niagara variant (3D copper bars): a Ribbon renderer emitter, 8 ribbons, Spawn Burst Instantaneous, positions set in Update Particle with a sine on Z per ribbon index; colour from a Color Curve with the 9-step ramp. Shadebobs: Sprite emitter with additive material on Lissajous paths, trail via spawn rate 120/s and lifetime 0.6 s.
- Expose to Remote Control preset: Material Parameter Collection scalars `Cycle`, `Warp`, `Bars`, `Spin`, `Pulse`, `Chunk`, `Scene`, plus `BeatPhase` (driven from OSC by Resolume/Ableton clock). Bind MIDI CCs from KeyStep/BeatStep in the Remote Control MIDI protocol, OSC addresses like `/demo/warp`.
- Output: Spout output (Off World Live or similar Spout plugin) or NDI to Resolume; send at 1920×1080 after the nearest upscale, or send 320×180 and upscale in Resolume with filtering off. Also accept a Spout texture from Resolume as the rotozoom tile.

### Godot
- One `ColorRect` full screen inside a `SubViewport` of size 320×180; `SubViewportContainer` with stretch on and `texture_filter = nearest` on the container (or a TextureRect showing the viewport texture with Nearest filter). This gives pixelation for free.
- CanvasItem shader on the ColorRect: uniforms `beat_phase`, `cycle`, `warp`, `bars` (int), `spin`, `pulse`, `scene`, `palette : sampler2D` with `filter_nearest, repeat_enable`, `tile : sampler2D` with `filter_nearest, repeat_enable`.
- Shadebobs: a second SubViewport with `render_target_clear_mode = never` (or ping-pong two viewports) whose ColorRect reads the previous frame and multiplies by decay; GPUParticles2D with additive CanvasItemMaterial for the bobs.
- Scanlines/posterise: a final CanvasItem shader on the upscaled TextureRect.
- Drive `beat_phase` from an AudioStreamPlayer's playback position and BPM, or from OSC/MIDI input (Godot has `InputEventMIDI`).

## Audio reactivity
- **Kick (40–120 Hz)**: envelope follower, attack 0–5 ms, release 120–150 ms, threshold −18 dB relative to the band's running peak. Drives `pulse`: bar height +40 %, roto zoom +0.3, tunnel speed +50 %.
- **Bass (60–250 Hz) slow**: attack 80 ms, release 400 ms. Modulates `warp` ±20 % and bar amplitude ±0.1.
- **Hi-hats (6–12 kHz)**: onset detect, threshold above a 1 s running average × 1.5. Each onset = +1 palette step and scanline flicker ±5 %.
- **Overall RMS**: release 1 s, maps to plasma brightness 60–80 %.
- Prefer beat phase from the clock for all periodic motion; audio only adds accents. Pure audio-driven motion looks jittery against sine-based effects.

## Performance tips
- All four generators at 320×180 cost almost nothing (under 0.1 ms each on any modern GPU). The output pass at 4K is the only real cost, and it is one texture read.
- The bar loop runs per pixel over N; cap N at 12 and early-out when outside all bars.
- Avoid bilinear filtering anywhere in the chain; one bilinear upscale ruins the look and softens edges on LED.
- Odd aspect ratios (e.g. 4:1 LED strips, 3840×960): keep the grid height fixed (e.g. 120 rows) and derive width from aspect, so pixels stay square. Normalise plasma/roto coordinates by `res.y` so patterns do not stretch. For very wide walls, raise plasma frequencies' x term or repeat the tunnel per section.
- Ensure the upscale is an integer factor of the internal grid (1920/320 = 6); non-integer factors give uneven pixel widths that shimmer when things move. On odd LED resolutions choose the grid as output divided by an integer.
- Palette cycling at full saturation can flicker on low-refresh LED processors; keep at least 2 frames per palette step (at 60 fps and fast cycle values, clamp to 30 steps/s).
- Feedback buffer for shadebobs: use 16-bit float to avoid the decay getting stuck at a grey floor; or subtract a small constant (1/255) each frame in 8-bit.

## Further references
- [Hugi 24: Bonz, "Sines and Cosines for Fun and Profit"](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm) — the formulas for plasma, rotozoom, tunnel, Lissajous paths and shadebobs
- [Wikipedia: Plasma effect](https://en.wikipedia.org/wiki/Plasma_effect) and [Wikipedia: Raster bar](https://en.wikipedia.org/wiki/Raster_bar) — history and the palette-cycling / per-scanline-colour principle
- [CESCG 2002: Realtime Visualization Methods in the Demoscene](https://old.cescg.org/CESCG-2002/BBurger) — overview of copper, fire, rotozoomer, voxel landscapes
- [Trixter, Demos Explained: graphics](https://www.scheib.net/play/demos/what/trixter/demo_graphics.html) — period screenshots of plasma, tunnel and copper
