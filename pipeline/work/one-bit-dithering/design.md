# One-Bit Dithering — design

## Look
Any video input, whether a camera, a clip, a UE render over Spout or a feedback layer, becomes a field of hard square pixels in exactly two colours. Every grey is faked by the density of dots. Flat areas turn into neat cross-hatch grids (Bayer) or fine stipple grain (blue noise), and gradients such as vignettes, glows and skin become slowly crawling dot patterns. The reference is the 1984 Macintosh screen (512x342, black on white, see [Beyond Loom](https://beyondloom.com/blog/dither.html)) and Lucas Pope's *Return of the Obra Dinn*: mostly solid blacks and whites with clear outlines, and dither only in the transitions ([PlayStation Blog](https://blog.playstation.com/archive/2019/10/17/lucas-pope-on-return-of-the-obra-dinns-art-style)). The mood is graphic, cold and printed, like an engraving that moves. On stage the pattern itself is the instrument. The kick pumps the image toward white, the bass makes the cells breathe, hi-hats make the grain sparkle, and section changes swap the ink and paper colours.

## Layer stack
| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid paper colour (default `#F2F2EC`) | Normal | 100 % | only visible where the source has alpha |
| 2 | Source | any clip, camera, Spout/NDI input or UE render | Normal | 100 % | high-contrast footage works best. Avoid flat mid-grey |
| 3 | Tone prep | luma, levels, contrast, gamma, bias pump | effect on layer 2 | 100 % | the place where the music pushes brightness |
| 4 | Outline (optional) | Sobel edges on the cell-res luma, forced to ink | inside the dither shader | 0–100 % (Edge) | gives Pope-style legibility on big walls |
| 5 | **Dither** | cell downsample, threshold against Bayer or blue noise, 1-bit output | effect on layer 2 (last in chain) | 100 % | core pass, see Shaders |
| 6 | Duotone | maps 0/1 to ink/paper hex colours | inside the dither shader | 100 % | palette preset per song section |
| 7 | Stencil variant (optional) | 1-bit result used as luma matte for a second clip | Mask / Luma Key on layer above | 100 % | variation 3: the dither cuts holes into another clip |
| 8 | Post | nothing that resamples. No blur, bloom or scaling after the dither | – | – | anything that filters destroys the pixels. See Performance |

Rule: the dither is always the **last** effect before output. If you put it inside a feedback loop, the loop re-dithers its own pattern and you get noise mush. Feedback goes before it, in layer 2.

## Techniques

### 1. Cell quantisation (pixel size)
1. Pick a cell size `px` in output pixels (1–16, integer).
2. Work on a virtual grid: `cell = floor(fragCoord / px)`, cell centre `uv_c = (cell + 0.5) * px / resolution`.
3. Sample the source at `uv_c`. For stable tone, use the box average of the cell instead of one tap (a mip level `log2(px)` or a 2x2 tap average). This stops single bright pixels in the source from flickering whole cells.
4. All later steps (threshold, edges) run per cell, not per output pixel, so the pattern stays chunky.

### 2. Tone preparation
1. Luma: `L = dot(rgb, (0.2126, 0.7152, 0.0722))`, done in linear or display space consistently. Display space (after tonemapping) gives the expected look.
2. Levels: `L = (L - black) / (white - black)`, clamped. Defaults are black 0.05 and white 0.95.
3. Contrast around 0.5: `L = (L - 0.5) * contrast + 0.5`, then gamma `L = pow(L, gamma)` (0.6–1.6).
4. Bias pump: `L = L + bias`, where `bias` is the threshold fader plus the kick envelope.
5. Why: dither only shows its texture in midtones. Contrast decides how much of the frame is solid (graphic) and how much is patterned (textured). Pope's lesson was to keep most areas solid and dither the transitions.

### 3. Ordered dithering (Bayer)
1. Threshold map of size N (2, 4 or 8). Value for cell `(x mod N, y mod N)`. Either use a small lookup texture (8x8, R8, nearest, no mips, no sRGB, wrap) or the standard recursive definition: `M(2n) = [4M(n), 4M(n)+2; 4M(n)+3, 4M(n)+1]`, starting from `M(1)=0`.
2. Normalise: `t = (M + 0.5) / N²`, which gives values strictly in (0, 1).
3. Output `bit = L > t ? 1 : 0`.
4. Bayer 2x2 gives coarse checkers with 5 tone levels. 4x4 gives 17 levels and the classic hatch. 8x8 gives 65 levels and smooth but visible cross-grid banding ([Wikipedia: Ordered dithering](https://en.wikipedia.org/wiki/Ordered_dithering)).
5. Why: per-pixel and neighbour-free, so it is cheap and stable in motion.

### 4. Blue-noise thresholding
1. Load a tileable blue-noise texture: Christoph Peters' CC0 set, 64x64 or 128x128 single-channel "LDR_LLL1" style ([Moments in Graphics](https://momentsingraphics.de/BlueNoise.html)). Import it linear, nearest, wrap, no mips.
2. `t = noise(cell / texSize + offset)`, then `bit = L > t`.
3. Static `offset = 0` gives a calm, screen-locked grain. Animated offset gives sparkle: on each trigger, `offset += (0.618034, 0.754878)` mod 1 (golden-ratio steps), so each new pattern is uncorrelated with the last.
4. Why: there is no visible grid and the quality is close to error diffusion, but it stays per-pixel ([Wronski](https://bartwronski.com/2016/10/30/dithering-part-three-real-world-2d-quantization-dithering/)).

### 5. Error diffusion (Atkinson / Floyd–Steinberg), offline only
1. These kernels are sequential: each pixel depends on the error from earlier pixels. They do not map to a single fragment pass, and on moving video they shimmer across the whole frame.
2. For the live chain, do not use them. Use blue noise as the stand-in.
3. If you want the authentic Atkinson worm/contrast look, pre-render clips offline (CPU, e.g. Processing, like the [Evil Mad Scientist sketch](https://www.evilmadscientist.com/2012/dithering/)) at the final cell resolution, encode them losslessly or as DXV at native size, and play them back undithered (no dither effect on that layer).
4. To fake the Atkinson punch live, use blue noise with contrast 1.6–2.0. Atkinson's look comes largely from discarding 1/4 of the error, which blows out near-black and near-white ([Wikipedia: Atkinson dithering](https://en.wikipedia.org/wiki/Atkinson_dithering)).

### 6. Outlines
1. On the cell-res luma, compute a Sobel gradient magnitude `g` from the 3x3 neighbouring cells, each sampled at the cell centre.
2. `edge = smoothstep(edgeLo, edgeLo + 0.1, g)`, with edgeLo 0.15–0.4.
3. Where `edge * Edge > 0.5`, force `bit = 0` (ink).
4. Why: at large cell sizes the dither stops blending for the eye, and outlines keep shapes readable (Pope's approach).

### 7. Pattern locking (motion stability)
1. Screen-locked pattern (default): the threshold map is fixed to the screen, so moving content "swims through a dirty window". This is stable and calm.
2. Scroll-locked: add a camera or content offset to `cell` before the threshold lookup, so the pattern travels with a pan. Use it for big lateral camera moves to avoid pixel strobing ([Pope devlog](https://dukope.itch.io/mars-after-midnight/devlog/285964/working-in-one-bit)).
3. World-locked (UE only): see the UE recipe.

## Shaders

**Render order:** Source → (feedback, keys, colour effects) → Pass A Dither → output. There is no ping-pong and no history buffer. The whole effect is stateless, which is why it holds up live.

### Pass A: Dither1Bit (single pass, the only one you really need)
- **Purpose:** turn the input into a duotone 1-bit image at cell resolution.
- **Inputs:** `inputImage` (source), `blueNoise` (64² or 128² CC0 texture), uniforms `cellSize` (float, rounded to int), `pattern` (int 0–4: 0 hard threshold, 1 Bayer2, 2 Bayer4, 3 Bayer8, 4 blue noise), `bias`, `contrast`, `gamma`, `edge`, `noiseOffset` (vec2), `scroll` (vec2, in cells), `invert` (bool), `inkColor`, `paperColor`.
- **Per pixel:**
  1. `cell = floor(fragCoord / cellSize)`, then `uv_c = (cell + 0.5) * cellSize / RENDERSIZE`.
  2. Sample the source at `uv_c`, using the 4 taps at ±0.25 cell averaged. Compute luma.
  3. Apply levels, contrast, gamma and bias (Technique 2).
  4. Get the threshold `t` for `cell + scroll` from the selected pattern (Technique 3 or 4). Pattern 0 uses `t = 0.5`.
  5. `bit = L > t`. If `invert`, then `bit = 1 - bit`.
  6. Edge: 8 more taps at neighbouring cell centres, Sobel, then force ink (Technique 6). Skip the taps when `edge == 0`.
  7. `out = mix(inkColor, paperColor, bit)`, alpha 1.
- **Output:** RGBA8 at full output resolution, with hard cell edges. No filtering anywhere after this pass.
- **Cost:** 4–13 texture taps per pixel. Trivial on any GPU.

### Optional Pass 0: Cell downsample (UE / Godot multi-pass version)
- Render the source into a buffer of size `ceil(W / cellSize) x ceil(H / cellSize)` with bilinear or box filtering (a proper area average). Then Pass A reads that buffer with nearest sampling and skips step 2. This gives better tone stability for large cells and noisy cameras, and costs one extra small buffer.

## Color scheme
| preset | ink (shadow) | paper (base) | use |
|--------|-------------|--------------|-----|
| Mac | `#0B0B0B` | `#F2F2EC` | default, intro, the classic look |
| Phosphor | `#00140A` | `#33FF88` | techno, cold sections |
| Riso | `#1B1F6B` | `#FF5E7E` | warm or pop sections |
| Sodium | `#140800` | `#FFB347` | breakdowns, night streets |
| Blueprint | `#F2F2EC` | `#0A2A8A` | inverted look, white ink on blue |
| Accent flash | `#000000` | `#FFFFFF` | drop only, 1–2 bars, pure contrast |

- **Mapping:** a strict two-colour lookup, `mix(ink, paper, bit)`. No gradient, no hue shift inside a frame. That is the rule of the look.
- **Over time:** switch presets on phrase boundaries (every 16 or 32 bars) as a hard cut, never a fade. A crossfade creates a third colour and breaks the 1-bit idea. The only exception is a 1-beat "invert" flash.
- **Ink from video (variation 2):** replace `inkColor` with a sample of a second texture at `uv_c`, for example a slow colour gradient clip. Paper stays flat.
- **LED black level and contrast:** keep ink at true or near black (≤ `#0B0B0B`). LED walls have deep blacks, so this is the strength. Keep paper below full white (`#F2F2EC`, or limit wall brightness to about 60–70 %). Large full-white fields on a wall are blinding and draw maximum power. Mostly-paper frames (high bias) need extra care: cap `bias` so that no more than about 70 % of pixels are paper for longer than one bar.

## Blending and compositing
- **Chain on one Resolume layer:** Source clip → (optional Levels / colour) → Dither1Bit. Normal blend to the composition. Dithering must come after everything that blurs, scales or blends.
- **Stencil (variation 3):** layer N is the clip you want to reveal. Layer N+1 is the dithered source set to a mask or luma-key mode (paper = opaque, ink = transparent), so the dot pattern of one clip cuts holes into another. Set ink and paper to pure `#000000` / `#FFFFFF` for clean keying.
- **Two dithers stacked:** two layers with different patterns (Bayer4 and blue noise) in Difference blend. Overlapping 1-bit patterns XOR into new textures and stay 1-bit if both use pure black and white. A good effect for breaks.
- **Feedback:** feed back the source before the dither, never the dithered output. Feeding back the 1-bit output (mix < 0.3) only works as a deliberate "crawl" glitch.
- **Post chain:** none. No bloom, blur, chromatic aberration, grain or sharpen after the dither. The dither already is the grain. If the composition is scaled to the output (LED processor canvas ≠ composition size), use integer scaling and nearest filtering only.

## Motion and timing
- **What moves:** the source moves. The pattern is static unless you animate it. The stage motion comes from four controls.
  - **Bias pump (kick):** `bias += 0.35` on the kick. Attack 0 ms, exponential release over 1/8 to 1/4 beat (about 60–120 ms at 128 BPM). The image flashes to paper and falls back.
  - **Cell breathing (bass):** cell size steps between integer values (2 → 3 → 4). Always quantised. Continuous fractional values make the grid jump and shimmer. Change on beats only.
  - **Sparkle (hats):** step `noiseOffset` by the golden-ratio vector on every 1/8 or 1/16 note, or on hi-hat onsets. Blue noise only. With Bayer, step `scroll` by 1 cell instead.
  - **Pattern swap (bars):** cycle Bayer2 → Bayer4 → Bayer8 → blue noise on bar boundaries (variation 1). Switching is instant, since it is per frame and stateless.
- **Scroll drift:** `scroll += speed * beats`, 0–4 cells per beat, linear (no easing; the pixel steps are the easing).
- **Build-up:** cell size grows from 2 to 8 over 8–16 bars, contrast drops from 1.4 to 1.0 (more pattern, less solid), and sparkle rate doubles every 4 bars.
- **Drop:** hard cut to cell size 2–3, contrast 1.8, Accent flash palette for 1 bar, then the section palette. The kick pump is at full depth.
- **Break:** the hard-threshold pattern (0) with high contrast, so the frame becomes pure silhouettes, or Bayer8 with large cells (10–16) and a slow scroll for a calm graphic texture. The sparkle is off.

## Parameters
| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| Cell | 1–16 px (integer) | 3 | pixel size: fine grain to chunky Mac blocks | BeatStep Pro knob 1, quantised. Bass step on beats |
| Threshold | −0.5 … +0.5 | 0.0 | pushes the frame toward ink or paper | KeyStep 37 mod strip or a fader. Kick envelope added on top |
| Contrast | 0.5 … 3.0 | 1.4 | solid graphic (high) vs. full pattern (low) | BeatStep Pro knob 2 |
| Pattern | 0–4 (Threshold, Bayer2/4/8, Blue noise) | 4 | dither character | BeatStep Pro pads 1–5 (note → value). Auto-cycle per bar |
| Sparkle | 0 … 1 (rate: off, 1/4, 1/8, 1/16) | 0 | grain animation speed | knob 3, or hi-hat onset trigger |
| Edge | 0 … 1 | 0.3 | outline strength for readability | knob 4 |
| Palette / Invert | preset 0–5 / toggle | Mac / off | ink and paper colours. Invert flash | pads 9–14 for presets, pad 16 momentary invert |

## Build recipes

### Resolume Arena
- **Core effect:** build Dither1Bit as a Wire patch (next section), save it as an effect and drop it as the **last** effect on any layer or on the composition. For composition-wide use, put it on the composition effects so everything is dithered together.
- **Layer 1:** a solid colour clip in the paper colour (only matters for sources with alpha).
- **Layer 2:** source clip / Spout input from UE / NDI input. Effect chain: Levels (or another built-in brightness/contrast effect) for rough trim → Dither1Bit.
- **Layer 3 (stencil variant):** a second clip with Dither1Bit set to pure black and white, used as a mask for the layer below.
- **Animation:** set Threshold to audio FFT (low band) with the parameter's audio settings, Pattern to BPM-synced step animation (1 bar per step), and Sparkle offset to BPM sync 1/8. Map Cell, Contrast and Edge via Shortcuts → MIDI to BeatStep Pro.
- **Output:** in Advanced Output, keep the slice at native resolution, or at an integer multiple of the composition. Any non-integer scaling smears the dither cells.
- **Pre-rendered Atkinson clips:** load them as DXV at native size on their own layer, with no dither effect.

### Resolume Wire
Node outline:
1. **Input texture** (effect input) → **Shader** node (GLSL/ISF shader holding Pass A) → **Output texture**.
2. **Texture** / file-loader node with the blue-noise PNG → shader input `blueNoise`. Set the sampler to nearest + repeat.
3. Exposed **parameters** (float sliders / int selector / colour pickers) → shader uniforms: Cell, Threshold, Contrast, Gamma, Pattern, Edge, Invert, Ink, Paper.
4. **Bias pump:** a trigger input (exposed, for Arena's audio or beat sync) → envelope/decay logic (attack 0, release ~100 ms) → add to Threshold → `bias` uniform.
5. **Sparkle:** a counter that increments on a trigger (beat division or exposed trigger) → multiply by (0.618034, 0.754878) → fract → `noiseOffset`.
6. **Cell:** round the parameter to an integer before it reaches the shader.
7. **Palette:** an int selector → a lookup of 6 colour pairs → `inkColor`, `paperColor`.
Save as an effect and expose the 7 stage parameters so they appear in Arena's effect panel.

### Unreal Engine 5.7
- **Post Process Material** (Material Domain: Post Process, Blendable Location: *Scene Color After Tonemapping*, so the dither sees display-space values and comes after bloom, film grain and AA). Add it to a Post Process Volume (Infinite Extent) → Rendering Features → Post Process Materials. Also disable TSR/TAA sharpening artefacts if they show. In practice the dither runs after AA, so it stays crisp.
- **Material graph:**
  - `ScreenPosition` (ViewportUV) × `ViewSize` → pixel coords → divide by `Cell` → `Floor` → cell index.
  - Cell index + 0.5 → × `Cell` / `ViewSize` → UV → `SceneTexture: PostProcessInput0` (Color) → `Dot` with (0.2126, 0.7152, 0.0722) → luma.
  - Levels / contrast / gamma / bias with `Subtract`, `Multiply`, `Power` and `Saturate` nodes.
  - Threshold: `Texture Sample` of an 8x8 Bayer texture or of the blue-noise texture (texture group UI or a custom group with Filter *Nearest*, no mips, sRGB off, Compression *Grayscale/R8*), UV = (cell index + scroll) / texSize + noiseOffset. For Bayer2/4, sample the 8x8 texture with (cell mod N) and rescale, or keep three small textures and `Switch` on Pattern.
  - `If` / `Step` (luma > t) → `Lerp` (Ink, Paper) → Emissive Color.
- **Parameters:** put all stage values in a **Material Parameter Collection** (Cell, Threshold, Contrast, Gamma, Pattern, Edge, Invert, Ink, Paper, NoiseOffset, Scroll). Expose the MPC values in **Remote Control** presets → bind to OSC addresses (e.g. `/dither/cell`) and to MIDI CCs from the BeatStep Pro. Drive Sparkle and Pump from Blueprint (a timeline or a beat event from OSC) that writes the MPC.
- **World-locked variant (Pope-style):** instead of screen UV, sample the blue noise triplanar in world space inside the object materials (WorldPosition / scale), or in the post material by reconstructing world position from SceneDepth. The pattern sticks to geometry and no longer swims when the camera moves.
- **Niagara:** not needed for the effect. Niagara particles (small unlit sprites) dither well because they make strong luma gradients.
- **Output:** send the final view to Resolume via a Spout sender (e.g. the Off World Live toolkit or a Spout plugin) or via NDI. Prefer Spout locally. NDI's compression smears 1-bit patterns (see Performance). Alternatively, send UE **undithered** over Spout and dither in Resolume, so one effect serves all sources.

### Godot
- **Scene:** source (`VideoStreamPlayer`, a `TextureRect` fed by a camera/Spout addon, or a 3D `SubViewport`) → full-screen `ColorRect` on a top `CanvasLayer` with a `canvas_item` shader.
- **Shader:** read the screen via `uniform sampler2D screen_tex : hint_screen_texture, filter_linear_mipmap` (mips allow the cell box average via LOD `log2(cell)`). `uniform sampler2D blue_noise : filter_nearest, repeat_enable`. Pass A steps as above, using `FRAGCOORD.xy` and `SCREEN_PIXEL_SIZE`.
- **Uniforms to expose:** `cell_size` (int hint_range 1–16), `bias`, `contrast`, `gamma`, `pattern` (int), `edge`, `noise_offset` (vec2), `scroll` (vec2), `invert`, `ink_color` / `paper_color` (`source_color`).
- **Optional Pass 0:** render the source into a small `SubViewport` sized `window / cell` (with linear filtering), then show it in a `TextureRect` with nearest filtering and run the threshold shader on it. This is cheaper and stabler.
- **Control:** a GDScript sets the uniforms from MIDI (`InputEventMIDI`) and from an audio bus `AudioEffectSpectrumAnalyzer` (bands below). In Project Settings, set the default texture filter to Nearest and use the `viewport` stretch mode with integer scaling.

## Audio reactivity
| band | drives | smoothing | threshold / notes |
|------|--------|-----------|-------------------|
| Kick, 40–120 Hz onset | Threshold bias pump +0.2…+0.4 | attack 0 ms, release 80–150 ms | onset when the band rises > 6 dB above its 500 ms average. Cap the total bias at +0.5 |
| Sub/bass energy, 30–80 Hz RMS | Cell size (2 ↔ 5) | release 400 ms, then quantise to an integer, applied on the next beat only | prevents grid jitter |
| Hi-hats, 6–12 kHz onset | Sparkle step (noise offset) | none (trigger) | min gap 60 ms so 1/32 rolls don't turn into white noise |
| Mids, 300 Hz–2 kHz RMS | Edge 0.2 → 0.8 | attack 50 ms, release 300 ms | keeps vocals and leads "drawn" |
| Bar / phrase clock | Pattern cycle, palette cut | – | from Resolume BPM / Ableton Link. BeatStep Pro clock as the fallback |

## Performance tips
- **GPU cost:** Pass A costs 4–13 taps per pixel, well under 0.2 ms at 4K on any current GPU. The optional downsample pass is negligible. This effect is never the bottleneck.
- **Resolution is everything:** run the dither at the **native output pixel grid** of the LED wall or projector. If the processor or scaler resizes the image afterwards, cells become uneven (some 3 px, some 4 px) and moiré appears. Use integer scaling only, or set the composition to the exact canvas.
- **Cell size vs. viewing distance:** on LED walls with a 2.6–3.9 mm pitch seen from 10 m or more, 1 px dither reads as flat grey noise. Use cell 2–4 and Bayer4/blue noise. On a 1080p projection, cell 2–3 reads "Mac". Cell 1 is only for close screens.
- **Odd aspect ratios / multiple slices:** compute the cells in **composition** pixel space, not per slice, so the grid is continuous across slices. If slices are mapped at different scales, the dither breaks. Dither per output in that case.
- **Compression:** 1-bit patterns are worst-case content for video codecs. NDI and H.264/H.265 recordings smear them into grey blocks, and DXV is heavy but holds. Send undithered video over NDI and dither at the receiving end. Pope chose random patterns partly because they compress better in video ([Game Developer](https://www.gamedeveloper.com/design/lucas-pope-and-the-rise-of-the-1-bit-dither-punk-aesthetic)).
- **Flicker and health:** a fast sparkle (1/16) on full-screen fine dither can be uncomfortable. Pope's first fullscreen dither made some playtesters ill ([PC Gamer](https://www.pcgamer.com/lucas-pope-on-the-challenge-of-creating-obra-dinns-1-bit-aesthetic)). Limit full-frame sparkle to short passages, avoid 1-beat ink/paper inversions above about 3 Hz, and follow your venue's photosensitivity rules.
- **Pitfalls:** sRGB-flagged or mip-mapped threshold textures give wrong thresholds and blur. Filtering after the dither (scaling, bloom, AA in UE before Blendable "After Tonemapping") softens the pixels. Mid-grey, low-contrast sources produce a uniform checkerboard, so always trim with Contrast first.

## Further references
- [Moments in Graphics: Free blue noise textures](https://momentsingraphics.de/BlueNoise.html): the CC0 blue-noise textures used in Pass A, with comparisons to Bayer.
- [Bart Wronski: Dithering part three, real-world 2D quantization dithering](https://bartwronski.com/2016/10/30/dithering-part-three-real-world-2d-quantization-dithering/): Bayer versus blue noise on the GPU, and why blue noise wins.
- [Wikipedia: Ordered dithering](https://en.wikipedia.org/wiki/Ordered_dithering): Bayer matrix construction and normalisation.
- [Lucas Pope: Working in One Bit](https://dukope.itch.io/mars-after-midnight/devlog/285964/working-in-one-bit): pixel strobing on moving 1-bit images and how to keep patterns stable.
