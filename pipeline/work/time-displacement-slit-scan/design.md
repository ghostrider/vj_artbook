# Time-Displacement Slit-Scan — design

## Look

A live camera or clip, but time no longer runs evenly across the frame. The edge where the scan starts shows "now", and every step across the image reaches further into the past, up to about two seconds back. Static things (stage, truss, a seated drummer) stay sharp. Anything that moves turns into ribbons: a hand becomes a fan of fingers, a turning head melts into a multi-angled smear like Rozin's *Time Scan Mirror* ([Rozin](http://www.smoothware.com/danny/timescan.html)), and dancers stretch and fold into each other as in Oliphant's *Slithering* ([VICE](https://www.vice.com/en/article/post-processing-technique-really-makes-these-dancers-slither-video/)). On the drop the map switches to a radial tunnel: the centre is now and the edges are the past, so moving light pours outward like the Stargate in *2001* ([Wikipedia](https://en.wikipedia.org/wiki/Slit-scan_photography)). Each kick pushes the time depth up and lets it relax, so the image smears on the hit and snaps back between hits. The mood can be natural and uncanny (no tint) or a psychedelic colour river (age tint on, cyan for now, magenta and violet for the past).

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #000000 | Normal | 100 % | the effect covers the full frame; the background only shows when dry/wet is pulled down |
| 2 | Source (dry) | live camera, NDI/Spout feed or clip with clear motion | Normal | 100 % | the same source feeds layer 3; keep it as the dry signal |
| 3 | Frame history | ring buffer of the last N frames (default N = 128 at 960×540), stored as a texture atlas or a Ring Buffer collection | — (not visible) | — | written every output frame; see Shaders P0 |
| 4 | Offset map | grayscale 0..1 per pixel: linear ramp, radial, luma, noise or spectrum | — (not visible) | — | generated inline in the read shader; can be a separate input texture |
| 5 | Slit-scan read (wet) | per-pixel read from the history at `head - map·depth·(N-1)`, two-tap time interpolation | Normal over layer 2 | 0–100 % (dry/wet fader), default 100 % | main look |
| 6 | Age tint | colour ramp keyed by the per-pixel delay | Multiply-mix inside layer 5 (`mix(src, src·ramp, tint)`) | tint 0–0.6 | off for portraits, on for Stargate rivers |
| 7 | Self-feed (optional) | output mixed back into what gets written to the history | inside P0 | feed 0–0.5 | streaks that persist and drift; variation 3 from the research |
| 8 | Post | black crush, light bloom, grain | see compositing | — | keep it light; the smear already softens the image |

## Techniques

**1. Video as a stack: x, y and time**
1. Treat the last N frames as a 3D block. Normal playback reads one slice at constant time; slit-scan reads a slanted or curved slice ([research](https://flong.com/archive/texts/lists/slit_scan/index.html)).
2. Store the block as N 2D frames in a ring: a write pointer `head` advances by 1 each output frame, `head = frameCount mod N`. Nothing is copied or shifted, only the pointer moves.
3. Each output pixel picks its own delay in frames, `d`, from the offset map, and reads frame `head - d`.
4. N sets the maximum delay: N = 128 at 60 fps is 2.13 s, which covers one bar at 128 BPM (1.875 s, 112.5 frames). N = 64 is enough for one beat-scale smear and halves memory.

**2. Ring buffer as a texture atlas (portable storage)**
1. Not every tool can write into a slice of a texture array, but all of them can draw into a region of one large 2D render target. So the default storage is an atlas: a grid of `cols × rows` tiles, one frame per tile.
2. Default: tile 960×540, grid 16×8 = 128 frames, atlas 15360×4320. It fits the usual 16384 px maximum texture size of D3D11/D3D12 GPUs.
3. Tile index `i` maps to tile position `(i mod cols, floor(i / cols))`.
4. Write: draw the current source frame scaled into tile `head` only. All other tiles keep their content (render target never cleared).
5. Read: `uvAtlas = (tile + inset(uv)) / (cols, rows)`, where `inset` clamps uv to `[0.5/tileW, 1 - 0.5/tileW]` (and the same for height) so bilinear filtering never bleeds into the neighbour tile. No mipmaps on the atlas.

**3. Offset map (the shape of time)**
All modes produce `m` in 0..1 (0 = now, 1 = oldest). Use `p = uv - 0.5`, with `p.x *= aspect` for round shapes.
1. Linear: `m = dot(p, (cos θ, sin θ)) / extent + 0.5`, where `extent` is the half-length of the frame along θ. θ = 90° gives a vertical sweep (top now, bottom past), θ = 0° the classic left-to-right slit.
2. Radial tunnel: `m = length(p) / length((0.5·aspect, 0.5))`. Invert (`1 - m`) for "edges now, centre past".
3. Luma: `m = luminance(current source)`, so bright parts lag behind dark parts (variation 5).
4. Noise: `m = fbm(uv · 3 + t · 0.05)` remapped to 0..1, 4 octaves; slow liquid ripples.
5. Spectrum: `m = spectrum(x)` from a 64- or 128-bin audio texture, smoothed, so the sound shape becomes the time shape (variation 4).
6. Shared shaping on top: wobble `m += wobble · sin(6.283 · (m · 3 + t · 0.5))`, clamp; bands `m = floor(m · bands) / bands` when bands ≥ 2 (hard stepped slits, the "Video Streamer" look); softness is the smoothstep between bands.

**4. Delay and time interpolation**
1. `d = m · depth · (N - 1)` in frames, float.
2. Beat-stepped mode (variation 2): `d = round(d / framesPerBeat) · framesPerBeat`, clamped to N - 1. With `framesPerBeat = 60 · 60 / BPM`, 28 frames at 128 BPM, so the bottom of the screen shows exactly one, two or three beats ago.
3. `f = head - d`, `i0 = floor(f)`, `a = f - i0`. Read tiles `i0` and `i0 - 1` (both wrapped with `((i mod N) + N) mod N`) and `mix` by `a`. Without this second tap a smooth ramp shows visible horizontal steps, because neighbouring rows jump whole frames.
4. Never read tiles not yet written: keep a `filled` counter for the first N frames after load and clamp `d ≤ filled - 1`, otherwise the start shows black or garbage tiles.

**5. Self-feeding scan (optional)**
1. What gets written into the history is `mix(source, previousOutput, feed)` instead of the raw source.
2. The output is then partly made of older outputs, so streaks compound and drift along the map direction.
3. Keep feed ≤ 0.5. Above that the image turns into mush within a few seconds (the research names too much depth as the main way the effect fails on a big screen).

## Shaders

Render order per frame: P0 write → P1 read + map + tint → P2 composite → P3 post. One frame of the buffer is written before it is read, so `head` already shows "now".

**P0 — History write**
- Purpose: put the current frame into the ring.
- Inputs: source texture, previous output (only for feed), uniforms `head`, `cols`, `rows`, `feed`.
- Per pixel: only the pixels of tile `head` are touched. Sample source at the tile-local uv; if feed > 0, `c = mix(src, prevOut, feed)`. Write RGBA8.
- Output: the atlas (persistent, never cleared). Resolution: atlas size, but only one tile (960×540) is actually written when the tool supports drawing into a sub-rectangle (UE canvas, Godot SubViewport). In ISF the pass runs over the whole atlas and copies the other tiles from itself, see Arena recipe.

**P1 — Slit-scan read**
- Purpose: build the time-displaced image.
- Inputs: atlas, current source (for luma map), optional spectrum texture (1D, 128×1, R16F) and optional custom map texture; uniforms `head`, `filled`, `N`, `cols`, `rows`, `depth`, `angle`, `mapMode`, `bands`, `wobble`, `beatStep`, `framesPerBeat`, `tint`, `time`.
- Per pixel: (1) compute `m` from the selected map mode; (2) apply wobble and bands; (3) `d = m · depth · (N - 1)`, optionally beat-stepped, clamp to `filled - 1`; (4) two-tap read from tiles `i0` and `i0 - 1`, mix by fraction; (5) age tint: `ramp(m · depth)` from the palette, `c = mix(c, c · ramp · 1.4, tint)`.
- Output: RGBA8 at output resolution (1920×1080 or the wall's canvas). Upscaling the 960×540 history happens here through bilinear sampling.

**P2 — Composite**
- Purpose: dry/wet and feed tap.
- Inputs: P1, dry source; uniform `dryWet`.
- Per pixel: `out = mix(dry, wet, dryWet)`. Store a copy as `prevOut` for P0 next frame (ping-pong: one extra 1080p RGBA8 target, 8.3 MB).
- Output: output resolution.

**P3 — Post**
- Black crush: `c = max(c - 0.02, 0) / 0.98`, contrast 1.1 around 0.5.
- Bloom: threshold 0.8, intensity 0.6, radius ~1 % of frame height. Only enough to make light sources in the smear glow.
- Grain: 2 %, monochrome, refreshed per frame. Hides 8-bit banding in long smooth smears.

## Color scheme

The picture colour comes from the source. The design adds an optional age tint and a stage grade.

| role | hex | use |
|------|-----|-----|
| base / black | #000000 | background, crushed blacks |
| shadow | #1B0F2E | deep violet floor for the oldest part of the ramp |
| accent "now" | #2DE2E6 | ramp at m = 0 (fresh image) |
| accent "past" | #FF3D7F | ramp at m = 0.55 |
| oldest | #6A2C91 | ramp at m = 1 |
| highlight | #FFE8A3 | bloom tint for blown highlights in the tunnel mode |

- Mapping: a 3-stop gradient ramp keyed by the per-pixel delay `m · depth`, multiplied into the image (not replaced), so faces and shapes stay readable. `tint = 0` gives the uncanny natural version; `tint = 0.4–0.6` the Stargate / Doctor Who colour river.
- Over time: rotate the ramp hue by 30° per phrase (8 bars) on stage, or hold it fixed for a whole song. On the drop jump tint from 0 to 0.5 in one beat.
- LED walls: blacks fully off after the crush (static backgrounds in the source often sit at 5–10 % grey, which reads as a haze on LEDs). Peak white at 90 %; the smear spreads bright objects over many pixels and can push wall power and glare high in tunnel mode.

## Blending and compositing

- Wet over dry in Normal with an opacity fader (dry/wet). Normal, not Add, because both layers are full images; Add would double the static background.
- For an "echo" look, put the wet layer in Lighten over the dry: the live subject stays solid and only bright trails show the past. Good with dancers lit on a dark stage.
- No mask by default. Optional: a soft luma key of the source (threshold 0.15, softness 0.1) as a matte on the wet layer, so smears happen only on the subject and the dark background stays clean.
- Feed amount 0–0.5, mixed before the write (P0), never after the read.
- Post chain order: black crush → bloom → grain. Grain last so it is not bloomed.

## Motion and timing

- What moves: the map (angle, mode, wobble) and the depth. The source supplies the actual motion; the effect only reshapes it.
- Angle: slow rotation, one full turn per 4 bars, linear. On hi-hats an optional 7.5° tick step instead of the smooth rotation.
- Depth: base 0.35. Kick punch +0.35, attack 10 ms, release 300 ms exponential (about half a beat at 128 BPM), so the image smears on the hit and relaxes in between.
- Wobble: driven by bass, 0–0.2, slow phase speed 0.5 Hz.
- Beat-stepped mode: switch on for a whole phrase; bands = 4 with beat steps gives four horizontal strips showing now, 1, 2 and 3 beats ago, which reads clearly on a big wall.
- Build-up: depth ramps from 0.2 to 0.7 over the last 4 bars, bands rise 2 → 16.
- Drop: cut to radial tunnel, depth 0.8, tint 0.5, angle rotation stops; kick punch stays on.
- Break: depth eases to 0.1 over 2 bars, map freezes, dry/wet to 50 %, feed 0.3 so leftover streaks drift slowly.
- Never hold one fixed map for more than about 16 bars; the research names a static map as the main reason this effect gets boring.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| depth | 0–1 (0–2.1 s at N=128, 60 fps) | 0.35 | how far into the past the far edge of the map reaches | BeatStep Pro knob 1; kick envelope added on top |
| angle | 0–360° | 90° | direction of the time sweep (linear mode) | knob 2; or LFO one turn per 4 bars |
| shape | 0–4 stepped (linear, radial, luma, noise, spectrum) | 0 | which map shapes time | BeatStep pads 1–5 |
| bands | 0 (smooth) – 32 | 0 | hard stepped slits instead of a smooth ramp; with beat-step, one strip per beat | knob 3; pad 6 toggles beat-step |
| wobble | 0–0.3 | 0.05 | ripples the time offsets | knob 4; bass envelope |
| feed | 0–0.5 | 0 | self-feeding streaks that persist | KeyStep mod strip |

Dry/wet is the layer opacity fader in every tool and does not need its own parameter.

## Build recipes

### Resolume Arena

- Arena has no built-in effect that stores a multi-frame history and reads it per pixel (a search found none; third-party FFGL plugins such as [BeatScanner](https://tombass.gumroad.com/l/beatscanner) and [Slice Displace](https://rexaffects.gumroad.com/l/gmtca) exist but were not tested for this book). Arena's Feedback and Trails effects only blend into one buffer, so they cannot do real slit-scan.
- Practical route 1, Wire effect: build the patch below in Wire and load it in Arena as an effect on the source layer. History lives in Wire's memory, see Wire recipe.
- Practical route 2, custom ISF effect (two passes):
  - Pass 1: persistent buffer `history`, `WIDTH = $WIDTH * 4`, `HEIGHT = $HEIGHT * 4` for a 4×4 grid (16 frames at full output size) or a smaller tile size with more tiles. Per pixel: if the pixel lies in tile `FRAMEINDEX mod N`, write the input image; else copy the previous value of `history` at the same pixel.
  - Pass 2: the read shader P1 sampling `history`.
  - Storage cost: the whole atlas is read and written every frame, so it is much heavier than the single-tile write in UE or Godot. Stay at tile 480×270, grid 8×8 = 64 frames (atlas 3840×2160, 33 MB) for ISF.
  - Caveat from the previous chapter: Resolume documents ISF in the context of Wire, and multipass / persistent-buffer support depends on the version. Test the ISF in your Arena build before the gig; if the persistent buffer does not keep its content, use the Wire route.
- Layer set-up: layer 1 dry source, layer 2 same source with the slit-scan effect, opacity = dry/wet. Map depth, angle, bands to the BeatStep Pro via Arena's MIDI shortcuts; use Arena's audio analysis (Low band) for the kick punch on depth.
- Use the source at the composition frame rate (60 fps); the delay is counted in output frames.

### Resolume Wire

- Storage option A, Ring Buffer: the Ring Buffer node keeps the last X frames as a collection; split the collection into separate outputs with the Split node. Check the maximum buffer size in your Wire version before the gig: a later changelog limits Ring Buffer sizes to prevent freezing, so do not plan for 128 full-HD frames.
  - Banded build: Ring Buffer size 16 at 960×540 (resize before the buffer) → Split → 16 outputs. Each output is masked to one band of the offset map (band k: `m` between k/16 and (k+1)/16) and the 16 masked layers are composited. This gives the stepped "bands = 16" look with built-in nodes. Smooth ramps are not possible this way.
- Storage option B, ISF atlas: put the two-pass ISF from the Arena recipe in Wire's ISF shader node. Gives the full smooth version, same caveats.
- Node outline (option B): Texture input (effect input) → Resize to 960×540 → ISF slit-scan (pass 1 atlas, pass 2 read) → Mix with the original input by dry/wet → Output.
- Control side: BPM / beat node → envelope (attack 10 ms, release 300 ms) → add to depth; an LFO at 1/4 bar rate → angle; audio band node (low) → wobble.
- Exposed parameters: depth, angle, shape, bands, wobble, feed, dry/wet. Exact node names for envelope and LFO differ between Wire versions; check the node browser.

### Unreal Engine 5.7

- Source: a Media Player + Media Texture (camera or file), or an NDI receiver texture.
- History storage: `RT_History`, a Texture Render Target 2D, 15360×4320, RTF RGBA8, mips off, clear colour black, filter bilinear. 128 tiles of 960×540, about 265 MB.
- Write (Blueprint, on Tick, before rendering): compute `head = frameCount mod 128`, tile origin `(head mod 16 · 960, floor(head / 16) · 540)`. Call Begin Draw Canvas to Render Target on `RT_History`, Draw Texture (Media Texture, Screen Position = tile origin, Screen Size = 960×540, Blend Mode Opaque), End Draw Canvas to Render Target ([UE docs](https://dev.epicgames.com/documentation/unreal-engine/BlueprintAPI/Rendering/BeginDrawCanvastoRenderTarget)). Do not call Clear Render Target 2D, the other tiles must survive. Only one tile is touched per frame, so the write is cheap. For feed > 0, draw a material that mixes the Media Texture and `RT_Out` instead of the raw texture.
- Read material `M_SlitScan` (User Interface or Post Process domain): TexCoord → map logic in standard material nodes, no Custom node needed (Dot, Length, Noise, Floor for bands) → delay `d` → two atlas UVs (Fmod for wrap, Floor/Frac for tile row/col, clamp inset) → two Texture Sample nodes on `RT_History` → Lerp by fraction → age tint via a Curve Atlas Row or 3-colour Lerp → Emissive / Final Color.
- Output: render the material into `RT_Out` (1920×1080) with Draw Material to Render Target and send it with Spout or NDI, or apply it as a Post Process Material on a fullscreen camera view.
- Remote Control: expose Material Parameter Collection `MPC_SlitScan` scalars `Depth`, `Angle`, `Shape`, `Bands`, `Wobble`, `Feed`, `DryWet`, `Tint` in a Remote Control Preset, bind to OSC and MIDI (BeatStep Pro). `Head` and `Filled` are set by the Blueprint, not exposed.
- Niagara is not needed. Optional extra: a Niagara system sampling `RT_History` for sparkle particles on bright smear pixels; not part of the core.

### Godot

- Godot 4. Source: VideoStreamPlayer (Theora .ogv) or a CameraTexture, rendered at 960×540.
- History: a SubViewport `History` sized 15360×4320, `render_target_clear_mode = CLEAR_MODE_NEVER`, `render_target_update_mode = UPDATE_ALWAYS` ([Godot docs](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)), transparent background off, 2D only. Inside: one TextureRect showing the source, size 960×540. A script moves it each frame to the tile origin of `head`. Because the viewport never clears, older tiles stay.
- If 15360 px wide viewports fail on the target GPU, use 64 tiles (8×8, atlas 7680×4320).
- Read: a full-screen ColorRect with a `canvas_item` shader that samples `History`'s ViewportTexture (filter linear, no mipmaps) and does P1 + P2.
- Feed: a second SubViewport at 1920×1080 holds the previous output; the TextureRect in `History` uses a small shader mixing source and that texture by `feed`.
- Uniforms to expose: `depth`, `angle`, `shape` (int), `bands`, `wobble`, `feed`, `tint`, `dry_wet`; `head`, `filled`, `cols`, `rows` set from the script. Drive them from an AudioEffectSpectrumAnalyzer on the music bus (low band for kick, 60–250 Hz for wobble).

## Audio reactivity

| band | drives | smoothing | threshold / scaling |
|------|--------|-----------|---------------------|
| kick, 40–120 Hz | depth punch +0.35 | attack 10 ms, release 300 ms | trigger at 0.6 of the band's running peak (2 s window), retrigger blocked for 120 ms |
| bass, 60–250 Hz | wobble 0–0.2 | attack 50 ms, release 400 ms | linear above a 0.2 floor |
| hats, 6–12 kHz | angle tick 7.5° or band jitter ±1 band | attack 5 ms, release 80 ms | trigger at 0.5 |
| full spectrum, 128 bins | map in spectrum mode | per bin `s = max(new, s · 0.85)` | log-scaled frequency axis so the bass does not fill the whole map |
| RMS loudness | tint 0 → 0.5 | attack 200 ms, release 2 s | only in drop sections |

In beat-stepped mode the kick punch is turned off, because the beat grid itself already pulses the image.

## Performance tips

- VRAM is the cost, not shader math. RGBA8 per frame: 1920×1080 = 8.3 MB, 960×540 = 2.1 MB, 480×270 = 0.5 MB.

| history | 64 frames | 128 frames | 256 frames |
|---------|-----------|------------|------------|
| 1920×1080 RGBA8 | 531 MB | 1.06 GB | 2.12 GB |
| 960×540 RGBA8 | 133 MB | 265 MB | 531 MB |
| 480×270 RGBA8 | 33 MB | 66 MB | 133 MB |

  RGBA16F doubles every figure; do not use float for the history.
- Recommended default: 960×540 × 128 frames (265 MB). The smear hides the lower resolution; the dry layer keeps the sharp full-res image where nothing moves.
- Atlas limits: most D3D11/D3D12 GPUs cap 2D textures at 16384 px per side. 1080p tiles allow at most 8 per row (15360 px) and 15 rows = 120 frames, 1 GB; avoid that.
- Write cost: UE canvas and Godot SubViewport write one tile per frame (cheap). The ISF route copies the whole atlas each frame: at 7680×4320 RGBA8 that is 133 MB read + written per frame, about 16 GB/s at 60 fps. Fine on a desktop GPU, too much on laptops, hence the smaller ISF default.
- Read cost: two texture taps per output pixel plus map math, well under 0.5 ms at 1080p.
- Odd aspect ratios: give tiles the output aspect. A 5760×1080 strip uses tiles 1440×270 (same pixel count as 960×540), grid 10×13 = 130 frames, atlas 14400×3510. In linear mode compute `extent` with the real aspect, or the ramp only covers part of a wide wall.
- Frame rate: delay is counted in frames. If the output drops from 60 to 30 fps, all delays double in seconds and beat-stepped bands drift off the beat. Lock the output fps, or compute `framesPerBeat` from measured fps.
- Pitfalls: mipmaps or missing tile insets cause visible seams (thin lines of a neighbouring frame) at band edges; sources with little motion show nothing; very high depth plus feed turns into mush.

## Further references

- [Slit Scan using the Time Machine TOP (Derivative)](https://learn.derivative.ca/courses/200-intermediate/lessons/202-tops-intermediate/topic/slit-scan-using-the-time-machine-top/) — official TouchDesigner lesson pairing a Texture 3D TOP (history) with a Time Machine TOP (per-pixel time offset map), with an example file; the same model as this design.
- [Levin, An Informal Catalogue of Slit-Scan Video Artworks and Research](https://flong.com/archive/texts/lists/slit_scan/index.html) — over 90 works for map and framing ideas.
- [Begin Draw Canvas to Render Target (Epic docs)](https://dev.epicgames.com/documentation/unreal-engine/BlueprintAPI/Rendering/BeginDrawCanvastoRenderTarget) — the Blueprint node used to write one tile per frame into the UE atlas.
