# Rutt-Etra Scan-Line Displacement — design

## Look

A black screen filled with 60 to 200 thin parallel lines of light, like the raster of an old CRT seen from slightly above. Where the source video is bright, the lines bulge upward into ridges, so a face, a dancer or a cloud turns into a contour relief made of light. Dark areas lie flat. The camera is tilted, so the lines bunch together toward the back and the relief reads as terrain. This follows the software emulations that lift each scan line by pixel brightness ([Airtight](https://www.airtightinteractive.com/2011/06/rutt-etra-izer/)). In the default "hidden line" mode, each line hides the lines behind it, so peaks block the ridges further back like a stacked ridgeline plot. In the "wire" mode all lines add up and the relief turns into lace-like glowing tendrils, the look of the Nalepa / Bainbridge pieces made on Etra's own unit ([CDM](https://cdm.link/?p=43365)). The mood is cold and analog: a phosphor-white or cyan line on deep black, with peaks that glow warmer as they rise. On the kick the whole relief jumps up and settles. On the drop the line count halves and the camera snaps to a steeper angle.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #000000 or another clip (when used as an overlay) | Normal | 100 % | the lines layer is pure black outside the lines, so anything can sit below |
| 2 | Displacement source (hidden) | moving video, camera or noise relief, converted to luma, blurred 3–6 px, downscaled to 480×270 | — (not visible) | — | pass P0; the blur is essential, see Techniques 4 |
| 3 | Colour source (optional, hidden) | second clip or camera giving the line colour (variation 4) | — | — | sampled at the same uv as the displacement |
| 4 | Scan-line relief | displaced line mesh (UE, Godot) or the fragment-shader fake (Wire, ISF) | Add over layer 1 (wire mode) or Normal (hidden-line mode) | 100 % | main look; pass P1 |
| 5 | Colouriser (optional) | colour video or gradient | Multiply, grouped with layer 4 only | 100 % | quick way to colour white lines in Arena without touching the shader |
| 6 | Feedback (optional) | the layer-4 output fed back as part of the displacement source | mixed into layer 2 at 0–40 % | — | recursive variation 3; one frame delay |
| 7 | Post | bloom, light blur on lines, black clamp, grain | see compositing | — | bloom carries the "phosphor" feel |

## Techniques

**1. Scan the source into rows (the line grid)**
1. Pick N rows (default 96). Row k samples the source along the horizontal line `v_k = (k + 0.5) / N`, so line spacing is `1 / N` of the source height.
2. Along each row, sample the source at every output pixel (fragment fake) or at M columns (mesh, default M = 256). More columns than about 1 per 4 output pixels gives no visible gain.
3. Height of a point: `h = pow(luma(u, v_k), gamma) * depth`, with `luma = dot(rgb, (0.299, 0.587, 0.114))`. Gamma 1.0–2.5 (default 1.6) keeps dark-grey noise flat and lets only real highlights rise.
4. Bright lifts, dark recedes, as in the Rutt-Etra-Izer ([Chrome Experiments](https://experiments.withgoogle.com/rutt-etra-izer)). An `invert` switch (`h = (1 - luma)^gamma * depth`) gives valleys instead of peaks, useful for dark silhouettes on a bright background.

**2. True 3D line mesh (UE, Godot): vertex displacement**
1. Geometry: N strips, each a row of M thin quads lying along X, 2 vertices per column (bottom and top edge of the line, offset by the line thickness in the up axis).
2. Both vertices of a column share one UV `(column / M, v_k)`, so both edges move by the same height and the line keeps its thickness.
3. The vertex shader samples the blurred source at that UV (explicit mip 0) and adds `h` along the up axis. The camera then shows the relief from any angle; orbiting reveals the depth.
4. Optional skirts for hidden-line mode: under each line strip a second strip runs from the line down to the base plane. Its top vertices get the same `h`, its bottom vertices stay at 0 (vertex colour flag), it is drawn black and opaque, so each ridge blocks what is behind it.

**3. 2.5D fragment-shader fake (Wire, ISF, any 2D tool)**
Wire has no vertex shaders ([Resolume ISF docs](https://resolume.com/support/en/isf)), so the relief is drawn per pixel by searching which displaced lines pass through the pixel.
1. Fixed oblique view, no free rotation. Row k has a screen base height `y0(v) = 0.08 + 0.72 * pow(v, 1 / (1 + persp))` (v = 0 is the front row at the bottom, v = 1 the back row at the top). `persp` 0–1 spreads the front rows and packs the back rows, which reads as a tilt.
2. Back rows are narrower and lower: scale `s(v) = 1 - 0.35 * persp * v`. Screen x of a source point: `x = 0.5 + (u - 0.5) * s(v)`, so for a pixel at x the row samples `u = 0.5 + (x - 0.5) / s(v_k)`. Line screen height: `y_k = y0(v_k) + h(u, v_k) * s(v_k)`.
3. Lines only rise, so only rows whose base lies between `y - depthMax` and `y` can reach the pixel. Invert `y0` to get that row index range: `v = pow((y - 0.08) / 0.72, 1 + persp)`, `k = v * N`. Loop over those rows only, from front to back. Cap the loop at 64 rows.
4. For each candidate row, coverage `c = smoothstep(w + aa, w - aa, |y - y_k| / sqrt(1 + slope²))`, where `w` is half the line width, `aa` = 1 pixel and `slope` is the screen slope of the line from a second tap one pixel to the right. Without the slope term, lines break into dashes on steep ridge flanks.
5. Wire mode: add up the coverage of all rows (additive lace). Hidden-line mode: stop at the first row (front to back) that either covers the pixel (draw it) or lies above the pixel by more than `w` (the pixel is under the ridge, so it stays black). That is the painter's algorithm in one pass, no depth buffer.
6. 2D roll: rotate the pixel coordinates by `roll` around the centre before step 2, so the whole relief can tilt on beats at no cost.

**4. Source conditioning (why the blur matters)**
1. Raw video has per-pixel noise. Displaced by 20 % of screen height, 1 grey level of noise becomes visible jitter, and compression blocks become stair steps.
2. Pre-blur with a Gaussian of 3–6 px at 1080p (or downscale to 480×270 and sample with bilinear filtering, which acts as a blur). Lines become smooth ridges.
3. Add a contrast curve before the gamma: black level 0.08, white level 0.9. Stage video is rarely full range; without this the "flat" areas still wobble.
4. Optional temporal smoothing: `src = mix(prevSrc, src, 0.5)` per frame (one ping-pong buffer) calms camera flicker. Skip it if the source is already a clean clip.

**5. Variations as switches (from the research)**
1. Audio as image: the displacement source is a scrolling spectrum history (each frame writes the newest FFT as the front row and shifts older rows back), so the relief is the sound itself. Front row = now, back row = about 2 s ago.
2. Vertical lines: swap u and v. Radial rings: run the same search in polar coordinates, `r` takes the role of `y`, ring k has base radius `r_k`, the height pushes it outward.
3. Recursive: mix 20–40 % of the previous output into the displacement source (the original was re-scanned by a camera, per the research).
4. Colour from a second input, height from the first (like the RGB-to-XYZ idea in the [VDMX tutorial](https://vdmx.vidvox.net/tutorials/advanced-v002-rutt-etra-techniques)).
5. Beat-locked line count: N steps between 192, 96, 48, 24 per bar (mesh: hide every second row; fake: change N).

## Shaders

Render order per frame: P0 source prep → P1 relief → P2 post. Optional P_fb copies the P1 output for the feedback variation.

**P0 — Source prep**
- Purpose: clean displacement map.
- Inputs: source texture; uniforms `blackLevel` (0.08), `whiteLevel` (0.9), `gamma` (1.6), `invert`, optional `prevOut` and `feedback` (0–0.4), optional `prevSrc` and `temporal` (0–0.7).
- Per pixel: (1) sample source with a 9-tap Gaussian (or read a downscaled copy); (2) luma; (3) `l = clamp((l - black) / (white - black), 0, 1)`; (4) invert if set; (5) `l = pow(l, gamma)`; (6) mix in `luma(prevOut)` by `feedback`; (7) mix with `prevSrc` by `temporal`.
- Output: single-channel (R8 or RGBA8) at 480×270. In audio mode a 256×128 history strip replaces this pass (row 0 = newest FFT, every frame the strip shifts one row; needs a persistent buffer).

**P1a — Relief, mesh version (UE, Godot)**
- Vertex stage: read P0 at the vertex UV with mip 0, `h = l * depth * lineMask * vertexFlag`, offset along up axis. `lineMask` hides rows for the line-count steps (`mod(rowIndex, step) != 0` → mask 0, row is clipped in the pixel stage).
- Pixel stage: colour = palette ramp(l) or colour source at uv; brightness `0.35 + 0.65 * l` so peaks glow; skirts output black.
- Output: render target at output resolution (1920×1080 or the wall canvas), 4× MSAA if the tool offers it (thin geometry aliases badly without it).

**P1b — Relief, fragment fake (Wire, ISF, Godot canvas_item)**
- Inputs: P0 texture, optional colour texture; uniforms `lines` (N), `depth`, `width` (px), `persp`, `roll`, `mode` (wire / hidden), `scroll`, `hueShift`, `RENDERSIZE`.
- Per pixel: (1) rotate by `roll`; (2) compute candidate row range from Techniques 3 step 3; (3) loop front to back: compute `u`, two taps of P0 at `(u, v_k + scroll)` and `(u + 1px, v_k + scroll)`, `y_k`, slope, coverage; (4) accumulate (wire) or stop at first hit / occlusion (hidden); (5) colour from palette ramp by `l` and row depth (`v_k` dims back rows to 55 %, fake fog).
- Output: RGBA8 at output resolution. Single pass, no buffers, so it works in ISF builds without multipass.
- `scroll` adds a constant to v when sampling, so the image flows through the static lines (flyover over a noise relief).

**P2 — Post**
- Black clamp: `c = max(c - 0.015, 0)`; kills the faint haze that dense additive lines produce.
- Bloom: threshold 0.6, intensity 1.2, radius about 0.8 % of frame height. Lines are thin, so a lower threshold than usual is needed for any glow.
- Grain: 1.5 %, monochrome, per frame. Optional horizontal 1-px scanline darkening (every second output row × 0.85) only on small screens; on LED walls it moires with the pixel grid, leave it off.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base / black | #000000 | background, hidden-line skirts |
| shadow | #0B1E3F | low lines, back rows, height 0 |
| base line | #2E7BFF | ramp at height 0.3 |
| accent | #29E6FF | ramp at height 0.6 (phosphor cyan) |
| highlight | #FFF4D6 | ramp at height 1.0, warm white peaks |
| alt accent | #FF2E88 | drop palette: replaces #29E6FF for one phrase |
| mono phosphor | #D8FFE8 | single-colour "original" mode (the original was black and white per the research) |

- Mapping: a 4-stop gradient ramp keyed by normalised height `l`, multiplied by `0.35 + 0.65 * l`. Back rows dimmed to 55 % by `v_k` (fake depth fog).
- Mode 2, source colour: line colour = colour input at the same uv, saturation × 1.3, so a colour video paints its own relief.
- Over time: hue rotate the ramp by 20° per phrase (8 bars); on the drop swap the accent to #FF2E88 for 8 bars, then back.
- LED walls: background must be true 0. In wire mode with N above about 150, overlapping lines add up to a grey wash: lower line brightness to 0.6 or switch to hidden-line mode. Peaks max 90 % white; thin bright lines on LED look sharp but bloom is what makes them readable from distance.

## Blending and compositing

- Lines over content: Add (wire mode) because the background is black and only the light should add. Screen if the content under it is bright and Add clips.
- Hidden-line mode over content: Normal with the layer's black as solid (the skirts must block), or Add if you want the ridge silhouettes to stay transparent.
- As a matte: use the line layer's luma as a mask for another layer (Arena: layer mask / Wire: Mask node on the luma), so another clip shows only through the lines.
- Colouriser: colour video in Multiply over white lines, grouped so it affects only the lines.
- Feedback: 0–0.4 of the previous output into P0 (the displacement), never added onto the final image.
- Post chain order: black clamp → bloom → grain. Grain last so it is not bloomed.

## Motion and timing

- Source motion does most of the work: use moving faces, dancers, clouds, smoke, or a slow fbm noise as relief (`noise(uv * 3 + t * 0.05)`, 4 octaves) when there is no video. A static photo looks dead (the research names this as the main boredom risk).
- Camera (3D tools): orbit yaw ±25° with a sine over 8 bars; tilt 15°–55° (default 30°). Fake: `persp` 0.3–0.8, `roll` ±6°.
- Scroll: 0.02–0.15 source heights per second, sync to 1 source height per 4 bars at 128 BPM (0.133/s) for a steady flyover.
- Depth: base 0.18 (fraction of screen height). Kick punch +0.12, attack 5 ms, release 250 ms exponential.
- Line count: change per bar in a build: 24 → 48 → 96 → 192, one step per bar on the downbeat (variation 5).
- Build-up: gamma falls 2.2 → 1.0 over 8 bars (more of the image rises), bloom intensity 0.8 → 1.6.
- Drop: N halves to 48, mode switches to wire, depth 0.3, tilt jumps to 50° in one beat with ease-out (3D) or `persp` 0.3 → 0.8, accent swaps to #FF2E88.
- Break: depth eases to 0.05 over 2 bars (lines go almost flat, a clean CRT raster), scroll slows to 0.02, hidden-line mode, mono phosphor colour. Bring depth back on the first kick.
- Change source, mode or camera at least every 16 bars.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| depth | 0–0.4 of screen height | 0.18 | how high bright areas lift; 0 = flat raster | BeatStep Pro knob 1; kick envelope added on top |
| lines | 16–240 (stepped 24/48/96/192 in beat mode) | 96 | line density, from bold stripes to fine lace | knob 2; pad toggles bar-stepped halving/doubling |
| tilt | 0–1 (fake: persp; 3D: 15°–55°) | 0.5 | viewing angle; flat front view vs terrain | knob 3; KeyStep mod strip for live sweeps |
| width | 0.75–4 px | 1.5 | line thickness | knob 4; hi-hat envelope +0.5 px |
| scroll | −0.2 … 0.2 source heights/s | 0.05 | image flows through the lines | knob 5, or BPM-locked (1 height per 4 bars) |
| occlude | 0 / 1 (wire / hidden line) | 1 | lace vs solid ridgeline look | BeatStep pad 1 |

Gamma, palette and feedback are set-and-forget values, not live controls.

## Build recipes

### Resolume Arena

- A search found no built-in Rutt-Etra-style line displacement in Arena, and its effects are 2D, so the relief is built in Wire and loaded into Arena as an effect (see Wire recipe), using the fragment fake P1b.
- Layer set-up: layer 1 background content (or black). Layer 2 the source clip (camera, dancer footage, smoke) with effects in this order: Blur (amount small, about 3 px) → the Wire Rutt-Etra effect → Bloom. Layer 2 blend mode Add, opacity 100 %.
- Colour from a video: put layer 2 and a colour clip in one group, colour clip on top in Multiply. Only the lines are coloured; black stays black.
- Recursive variation: on a third layer use a Layer Router source pointing at layer 2, opacity 30 %, blend Add, placed below layer 2 and routed into the effect's source (or simply layered under it for echo ridges). It adds one frame of delay.
- Map depth, lines, tilt, width to the BeatStep Pro via Arena's MIDI shortcuts on the effect parameters. Kick punch: Arena's audio analysis on the Low band → depth, with the parameter's own attack/release.
- Custom ISF: if your Arena build loads ISF effects directly, P1b is a single-pass shader and needs no persistent buffers, so it is the safest custom ISF in this book. Resolume documents ISF only in the context of Wire, so test in your version before the gig.

### Resolume Wire

- No vertex shaders in Wire ([Resolume ISF docs](https://resolume.com/support/en/isf)) and no 3D mesh rendering, so the line mesh cannot be displaced as geometry. The whole relief is the fragment fake P1b inside an ISF node; the "3D" is the per-pixel row search of Techniques 3.
- Node chain: effect Texture input → Resize to 480×270 → Gaussian blur node (radius 2 at that size) → Levels/colour correction for black 0.08 / white 0.9 → ISF node (P1b shader, P0 texture on its image input, optional second image input for colour) → Bloom → effect output.
- Single pass is enough. Only the temporal-smoothing and audio-history variations need a persistent buffer; multipass / persistent-buffer support in Wire's ISF depends on the version, so build those only after testing. For the audio-as-image variant without buffers, use the current spectrum only as a single front row with all other rows reading the same data shifted by `scroll` (it looks like a scrolling landscape, though not a true history).
- Control side: audio analysis node (low band) → envelope (attack 5 ms, release 250 ms) → add to depth; BPM/beat node → counter per bar → `lines` = 24 · 2^(count mod 4); LFO at 1/8 bar rate → `roll` ±6°.
- Exposed parameters (published as effect parameters): depth, lines, tilt, width, scroll, occlude, palette (0–2: ramp / mono / source colour).

### Unreal Engine 5.7

Main route: a static line mesh displaced in the material (World Position Offset). Niagara is an alternative.

- Mesh `SM_ScanLines`: build in Blender (or any DCC) and import. 1000 × 1000 uu footprint in X/Y, 160 rows × 256 segments. Each row: a strip with 2 vertices per column, bottom edge at Z = 0, top edge at Z = 1.2 uu (line thickness), the strip faces −Y (toward the default camera). UV0 = `(col / 256, (row + 0.5) / 160)` on both vertices of a column; UV1.x = row index / 160 for the line-count mask. Vertex colour: R = 1 (line), G = 1 (displace). Skirt strip per row (for hidden-line mode): top vertices R = 0, G = 1, bottom vertices at Z = −2 uu with R = 0, G = 0. About 160 × 257 × 4 ≈ 164k vertices.
- Source: Media Player + Media Texture (or NDI receiver texture). Render a blurred, levelled copy into `RT_Src` (480×270, RGBA8) each tick with Draw Material to Render Target and a small material doing P0. Sampling a full-res video in the vertex shader without the blur makes the lines jitter.
- Material `M_RuttEtra`: Shading Model Unlit; Blend Mode Masked (hidden-line, skirts on) or Additive (wire mode, skirts hidden via a second material slot or a static switch). World Position Offset: Texture Sample (`RT_Src`, UV0, MipValueMode = MipLevel, level 0) → R channel → Power (gamma) → × `Depth` (default 180 uu) → × VertexColor.G → Append (0, 0, value). Emissive: ramp via a Curve Atlas row lookup with the height, × (0.35 + 0.65 · height) × VertexColor.R × `Intensity` (default 4 for bloom). Opacity Mask: line-count mask `fmod(floor(UV1.x · 160), Step) == 0` (Step = 1, 2, 4, 8).
- Component: set Bounds Scale to about 2 on the static mesh component, because WPO moves vertices beyond the original bounds and the renderer still culls by those bounds.
- Camera: CineCameraActor, 35 mm, pitch −30°, distance so the 1000 uu plane fills the frame. Animate yaw ±25° in a Level Sequence or via Remote Control.
- Post Process Volume (unbound): Bloom intensity 1.5, threshold 0.6; Exposure Min = Max = 1 (manual exposure, otherwise auto exposure dims the frame when lines rise); Vignette 0; Film Grain 0.02. Anti-aliasing: TSR smears thin lines when the camera moves; for crisp lines use the forward renderer with MSAA 4×, or keep line thickness ≥ 1.5 px on screen.
- Niagara alternative: GPU emitter, Spawn Burst Instantaneous 160 × 256, Grid Location (X 256, Y 160), a texture-sample step reading `RT_Src` at each particle's grid UV to set the height, Ribbon Renderer with RibbonID = row index and RibbonLinkOrder = column. Easier for per-line effects (each line can have its own delay or noise), but ribbon ordering at this count needs testing; keep the static mesh as the gig version.
- Remote Control: Material Parameter Collection `MPC_RuttEtra` with `Depth`, `Step` (lines), `Gamma`, `Intensity`, `Scroll` (added to UV0.y), `HueShift`; camera yaw/pitch as actor properties. Put them in a Remote Control Preset and bind to OSC and MIDI (BeatStep Pro).
- Output: render the camera view to a render target and send via your Spout or NDI sender plugin to Arena, where the lines layer goes in Add mode.

### Godot

- Godot 4. 3D route: a MeshInstance3D with an ArrayMesh built once by a `@tool` script with SurfaceTool (same layout as the UE mesh: N rows × M columns, thin strips, UV = (col/M, row centre), COLOR.r = line flag, COLOR.g = displace flag, optional skirts). Quick test: PRIMITIVE_LINES instead of strips (always 1 px wide, no width control).
- Spatial shader: `render_mode unshaded, cull_disabled, blend_add` for wire mode, or `unshaded, cull_disabled` (opaque) with skirts for hidden-line mode ([Godot spatial shader docs](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/spatial_shader.html)). In `vertex()`: read `source_tex` at `UV + vec2(0, scroll)` with explicit LOD 0, height as in Techniques 1, `VERTEX.y += h * COLOR.g`. In `fragment()`: `ALBEDO = ramp(h) * COLOR.r * (0.35 + 0.65 * h)`; for line-count stepping, `discard` rows where `mod(row, step) != 0`.
- Source: a SubViewport (480×270) containing a VideoStreamPlayer or camera TextureRect with a small blur/levels canvas_item shader (P0); pass its ViewportTexture to `source_tex` with linear filtering.
- Set `extra_cull_margin` on the MeshInstance3D (about the max displacement) so displaced lines are not culled at frame edges.
- Camera3D at pitch −30°, FOV 40°. WorldEnvironment: background black, Glow on (intensity 1.2, HDR threshold 0.6), tonemap Linear.
- 2D route: a full-screen ColorRect with a `canvas_item` shader doing P1b, identical to the Wire version, for when no 3D scene is wanted.
- Uniforms to expose: `depth`, `lines`/`step`, `width` (fake only), `persp` or camera pitch, `scroll`, `occlude`, `gamma`, `hue_shift`. Drive them from an AudioEffectSpectrumAnalyzer on the music bus.

## Audio reactivity

| band | drives | smoothing | threshold / scaling |
|------|--------|-----------|---------------------|
| kick, 40–120 Hz | depth punch +0.12 | attack 5 ms, release 250 ms | trigger at 0.6 of the running peak (2 s window), retrigger blocked for 120 ms |
| bass, 60–250 Hz | gamma 2.0 → 1.2 (more relief rises) | attack 80 ms, release 600 ms | linear above a 0.2 floor |
| hats, 6–12 kHz | width +0.5 px flicker | attack 3 ms, release 60 ms | trigger at 0.5; off in hidden-line mode with N > 150 (looks like noise) |
| full spectrum, 128 bins | displacement source in audio-as-image mode | per bin `s = max(new, s · 0.85)` | log frequency axis across u, so bass does not fill the width |
| RMS loudness | bloom intensity 0.8 → 1.6 | attack 300 ms, release 2 s | drop sections only |
| beat clock | lines step per bar, yaw sine per 8 bars, palette per phrase | — | quantise to downbeats |

## Performance tips

- Fragment fake cost: about `2 · depth · N + 4` texture taps per pixel (two taps per candidate row). Default 0.18 × 96 → about 39 taps, roughly 0.5 ms at 1080p on a desktop GPU. N = 240 with depth 0.4 → about 196 taps: cap the loop at 64 rows and sample a small P0 (480×270 fits in cache).
- Mesh route: 164k vertices with one texture read each is trivial. The cost is anti-aliasing: thin geometry needs MSAA or it shimmers, especially on slow camera moves.
- Downscale the displacement source. 480×270 is enough for 240 lines; a 4K source sampled directly only adds noise and bandwidth.
- LED walls: line width at least 1 LED pixel, better 2. On a 2.6 mm pitch wall with a 1920×1080 canvas mapped 1:1, 1.5 px lines are fine; on lower-resolution walls (for example 960 px high) set `lines` ≤ 80 or neighbouring lines merge into a grey surface and moire with the LED grid.
- Odd aspect ratios: keep rows horizontal and scale columns with the width. For a 5760×1080 strip use M = 768 in the mesh, and in the fake compute `u` with the real aspect. For tall portrait screens use vertical lines (variation 2) so the lines run the long way.
- Hidden-line mode on the fake: stop the loop early (first hit), so it is cheaper than wire mode.
- Pitfalls: unblurred source (jitter), auto exposure in UE (brightness pumps with the relief), culling of displaced vertices (bounds), PRIMITIVE_LINES / 1-px lines vanishing on downscaled outputs, and a static source.

## Further references

- [Airtight Interactive: Rutt-Etra-Izer](https://www.airtightinteractive.com/2011/06/rutt-etra-izer/) — Felix Turner's write-up of the WebGL emulation: rows scanned top to bottom, line separation, z from brightness.
- [VDMX: Advanced v002 Rutt Etra techniques](https://vdmx.vidvox.net/tutorials/advanced-v002-rutt-etra-techniques) — RGB-to-XYZ displacement gradients, audio/FFT as lookup, crossfading between shapes.
- [Resolume: ISF in Wire](https://resolume.com/support/en/isf) — the ISF node, and the note that Wire does not support vertex shaders, which is why Wire uses the fragment fake.
- [Godot: Spatial shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/spatial_shader.html) — VERTEX, UV and the `unshaded`, `blend_add`, `cull_disabled` render modes used in the Godot recipe.
