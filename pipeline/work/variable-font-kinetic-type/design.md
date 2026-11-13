# Variable Font Kinetic Type — design

## Look

One oversized word fills the wall, set in heavy white type on pure black, letters touching the edges or cropped. The letters do not move around. They change shape: on every kick the weight punches from hairline to black and relaxes over half a beat, the width breathes with the bass, and between hits a slow wave runs through the word so each letter swells a little after its neighbour, like a crowd doing the wave. Because the font interpolates between its masters, nothing jumps or swaps. The type stays the same typeface and simply gets fatter, narrower or more slanted, which is what separates this from a font-switch strobe (research). With a morph-axis font such as Kablammo the letters go further and wobble through cartoon shapes on the drop (research). The mood is graphic and editorial in the verses (monochrome, calm wave, held) and loud in the drop (kick punches, a short feedback echo, and the type used as a window onto colour video). Reference images in words: the per-character audio response of the COLLINS identity for the San Francisco Symphony, and the tools Variable Time and Type Physics+, all listed in the research.

## Tooling reality (read first)

No VJ tool in this book's stack was found to expose variable font axes natively:

- **Resolume Arena / Wire:** the research found no axis control in Arena's text sources, and Resolume's ISF documentation does not cover fonts at all ([Resolume ISF](https://resolume.com/support/en/isf)).
- **Unreal Engine 5.7:** no variable font axis control was found. A 2024 forum feature request says the feature is missing, and the thread has no answer from Epic ([UE forum](https://forums.unrealengine.com/t/feature-request-variable-fonts/1897780)). UE 5.5 and newer can render UI text as SDF/MSDF and adds a *Font Signed Distance* material node. That node is useful for a fake weight, but the SDF docs never mention variable fonts ([Epic docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/using-signed-distance-field-text-rendering-in-unreal-engine)).
- **Godot 4:** has full native support. A `FontVariation` resource takes OpenType axis coordinates in `variation_opentype`, and `Font.get_supported_variation_list()` returns each axis as min/max/default ([FontVariation](https://docs.godotengine.org/en/stable/classes/class_fontvariation.html), [Font](https://docs.godotengine.org/en/stable/classes/class_font.html), [Using fonts](https://docs.godotengine.org/en/stable/tutorials/ui/gui_using_fonts.html)).
- **Browsers:** have full native support through CSS `font-variation-settings` and the standard properties `font-weight`, `font-stretch` and `font-style` ([MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Fonts/Variable_fonts)).

That gives four realistic routes. The design below works with all four.

| route | true variable outlines? | per-letter control | runs inside | best for |
|-------|------------------------|--------------------|-------------|----------|
| **A. Live renderer → Spout/NDI** | yes, live | yes | Godot 4 app (Spout) or browser page in OBS (NDI) | the full effect, any text typed live |
| **B. Axis-sweep MSDF atlas + shader** | yes, sampled at K steps and interpolated | yes | ISF (Arena/Wire), UE material, Godot shader | self-contained in one tool, fixed charset |
| **C. Axis-sweep clip, scrubbed** | yes, pre-rendered | whole word only | Arena clip transport | fastest set-up, one word per clip |
| **D. SDF pseudo-weight** | no, distance offset only (dilate/erode) | yes | UE 5.5+ UMG text, any SDF shader | quick fallback for any static font |

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #000000 | Normal | 100 % | true black for LEDs |
| 2 | Fill video (optional) | colour clip, live camera or a generator from another chapter | Normal | 100 % | only visible through the type matte (layer 4) |
| 3 | Type source | white type on black (RGB) plus coverage (A), output of route A, B or C | — (used as a matte) | — | 1920×1080 or wall canvas; the core of the effect |
| 4 | Type matte | layer 3 cuts layer 2 (type as a window) | Arena: layer 2 masked by layer 3 in luma or alpha mode | 0–100 % (fill mix) | at 0 % you see plain white type; at 100 % the letters are windows onto video |
| 5 | Type plain | layer 3 tinted with the palette colour | Add over 1/4 | 100 % | carries the kick flash colour |
| 6 | Echo feedback | previous output scaled ×1.015 around the centre, faded ×0.82 | Lighten (max) with the current type | 0–60 % | drops and fills only; trails show where the letters were heavier a moment ago |
| 7 | Outline ghost (optional) | the same SDF read as a thin outline at a lower axis value (e.g. weight − 300) | Add | 40 % | double-contour look on breaks; route B/D only |
| 8 | Post | black crush → bloom → grain | see compositing | — | keep the type edges crisp |

## Techniques

**1. Axis driving model (all routes)**
1. Each axis (wght, wdth, slnt, or a custom axis such as Kablammo's MORF) gets a normalised control value `a` from 0 to 1, mapped to the font's real range: `value = mix(min, max, a)`. In Godot, read min and max from `get_supported_variation_list()`. In CSS the registered ranges are wght 1–1000, wdth as a percentage and slnt −90 to 90 ([MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Fonts/Variable_fonts)).
2. Per letter `i` of `n`: `a_i = clamp(base + depth · w_i + punch · env_kick + jitter_i, 0, 1)`.
3. The wave term is `w_i = 0.5 + 0.5 · sin(2π · (beatPhase · rate − i · spread / n))`, with `beatPhase` counting beats from the host BPM clock. `spread = 1` places one full wave across the word, and `rate = 0.25` gives one cycle per bar.
4. `env_kick` is a 0..1 envelope: attack 10 ms, release 250 ms exponential. Add `delay_i = i · stagger` (0–40 ms) so the punch ripples through the word instead of hitting every letter at once.
5. Holds: a `freeze` switch latches all `a_i`. Pure continuous motion gets boring (research), so freeze for a bar, then release on the next kick.

**2. Route A: live variable font renderer**
1. Godot 4: import the variable font with *Multichannel Signed Distance Field* enabled, so large sizes are crisp and size changes need no re-rasterisation ([Using fonts](https://docs.godotengine.org/en/stable/tutorials/ui/gui_using_fonts.html)).
2. At load, pre-build a bank of `FontVariation` resources, one per quantised axis step (64 steps per axis, or 16×16 for two axes). Quantising caps how many glyph caches exist. A cache RID exists per variation (`find_variation()`, [Font](https://docs.godotengine.org/en/stable/classes/class_font.html)), so setting free floats every frame could build an unbounded number of caches. That limit is my design caution, not a documented one.
3. In a Node2D `_draw()`, draw letter by letter with `draw_char(font, pos, char, size, modulate)`. Pick the font from the bank by `round(a_i · 63)`, and use `draw_set_transform` for optional per-letter scale or rotation ([CanvasItem](https://docs.godotengine.org/en/stable/classes/class_canvasitem.html)). Call `queue_redraw()` every frame.
4. Layout: advance `x` by `get_string_size(char)` of the chosen variation ([Font](https://docs.godotengine.org/en/stable/classes/class_font.html)), then centre the whole word. This is "flow" layout: weight and width changes reflow the word, which makes the text elastic.
5. Browser alternative: put each letter in its own `<span>` and set CSS custom properties per span (`--wght`, `--wdth`) inside `font-variation-settings`. MDN recommends this pattern so one axis can change without redeclaring the others ([MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Fonts/Variable_fonts)). Splitting a word into spans for per-letter animation follows the CSS-IRL article cited in the research. Set `font-synthesis: none` so the browser does not add a fake bold on top (MDN).

**3. Route B: axis-sweep MSDF atlas (the portable core)**
1. Offline, generate K MSDF atlases of the same charset, one per axis step, with msdf-atlas-gen. Use `-varfont "font.ttf?wght=<v>"` for each step value `v`, plus `-type mtsdf -uniformgrid -uniformcols 8 -uniformcell 128 128 -uniformorigin on -pxrange 8 -size 96 -yorigin top`. Add `-json` to get each glyph's `advance` ([msdf-atlas-gen](https://github.com/Chlumsky/msdf-atlas-gen)). With `-uniformorigin on` the glyph origin sits at the same point in every cell, so cells from neighbouring steps line up.
2. Default: K = 16 steps (wght 100–900 in steps of about 53) and a 40-glyph charset (A–Z, 0–9, `! ? . -`) in 8×5 cells, so each step atlas is 1024×640. Tile the 16 step atlases into a 4×4 super-atlas of 4096×2560 RGBA8 (40 MB). Before generating everything, check that W and M at maximum weight fit the 128 px cell. If they don't, use 160 px cells (5120×3200 super-atlas).
3. Per pixel: find the letter slot and glyph, then compute `s = a_i · (K − 1)`, `k0 = floor(s)`, `f = s − k0`. Sample the cell in step atlases `k0` and `k0 + 1` at the same cell-local uv, take `d = median(r, g, b)` of each, and interpolate `d = mix(d0, d1, f)`. Coverage is `clamp((d − 0.5) · pxRangeScreen + 0.5, 0, 1)`, with `pxRangeScreen = pxrange · (onscreenGlyphSize / 128)`.
4. Why it works: variable-font outlines move continuously along an axis, so neighbouring instances are close and the interpolated distance looks like an in-between master. At about 50 weight units per step no ghosting shows. Morph axes that change topology (Kablammo MORF 0–60) need denser sampling: use one step per integer (K = 61) or accept a short cross-fade.
5. A second axis (e.g. wdth) multiplies K. Use 8×8 = 64 steps, sample 4 neighbours (bilinear in axis space), and drop the charset to the letters of the words you need.
6. Layout: "slot" by default, where each letter is centred in an equal-width slot (`slotW = canvasW / n`), so there is no reflow and no advance table. "Flow" layout reads advances from a K×glyphs table (from the JSON) and is only used in UE and Godot, where uniform arrays are easy.

**4. Route C: scrubbed sweep clip**
1. Render one word with route A while the axis sweeps 0 → 1 over 240 frames, white on black. Export as DXV with alpha at 1920×1080.
2. In Arena, clip position = axis value. Ping-pong BPM sync over 1 beat gives breathing. For audio scrubbing, drive the clip's transport position over OSC, e.g. `/composition/layers/3/clips/1/transport/position` ([Resolume OSC](https://resolume.com/support/en/osc)).
3. There is no per-letter phase. For a wave, render one clip per letter and place them on separate layers with their own offsets. That gets heavy fast, so prefer route B for the wave.

**5. Route D: SDF pseudo-weight**
1. Any SDF text (UE 5.5+ UMG text with distance-field rasterisation enabled, Godot MSDF, or the route B atlas with K = 1) can be dilated or eroded by shifting the threshold: `coverage = clamp((d − 0.5 + bold) · pxRangeScreen + 0.5, 0, 1)`, with `bold` between −0.15 and +0.25.
2. This is not a weight axis. Counters fill up, stems stay the same length and corners round off. It reads as weight only when it changes fast, as on a kick punch, so use it for the punch and use a real axis (A/B) for the slow wave.
3. In UE, the *Font Signed Distance* node outputs Signed Distance, Smooth Signed Distance, Pixel Distance Factor and Implicit Opacity. Growing beyond the glyph needs *Material Is Stencil* on the text, plus an outline of similar width to widen the distance range ([Epic docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/using-signed-distance-field-text-rendering-in-unreal-engine)).

## Shaders

Render order per frame: P1 type → P2 echo (ping-pong) → P3 composite → P4 post. Under route A, P1 runs in Godot or the browser and P2–P4 run in the host (Arena or UE).

**P1 — Variable type (route B)**
- Purpose: draw the word, with each letter at its own axis value.
- Inputs: `atlas` (super-atlas, linear filter, **no mipmaps**, clamp), uniforms `glyphIdx[0..11]` (int per letter, −1 = space), `n`, `K`, `axis[0..11]` (the `a_i` computed on the CPU or in the host's parameter system), `pxrange`, `cellPx`, `atlasCols`, `stepCols`, `bold` (route D offset), `outline` (0 = fill, >0 = stroke width), `slant` (0–0.3), `marginX`, `marginY`.
- If the host cannot compute per-letter values (ISF), pass `base`, `depth`, `spread`, `rate`, `beatPhase`, `env` and evaluate the wave formula from Technique 1 per letter inside the shader.
- Per pixel:
  1. `uv` into a text box: `p = (uv − margin) / (1 − 2·margin)`, then `slot = floor(p.x · n)`, `local = (fract(p.x · n), p.y)`.
  2. Fake slant (needed for fonts without slnt): `local.x += (local.y − 0.5) · slant`.
  3. Map the slot box to the glyph cell. Glyph height fills the box, `cellUV = local` adjusted by the slot's aspect (`slotW / boxH`) so letters are not stretched.
  4. Compute the cell coordinates of glyph `g` in step atlases `k0` and `k0+1`: `step origin = (k mod stepCols, floor(k / stepCols)) · stepAtlasSize`, `cell origin = (g mod 8, floor(g / 8)) · cellPx`. Clamp the cell uv 1 px inside the cell.
  5. Two taps, a median per tap, a mix by `f`, then the coverage formula with `bold`. Outline mode uses `cov = band(d, 0.5 − outline, 0.5 + outline)`, built from two smoothsteps.
  6. Glow term from the MTSDF alpha (true SDF): `glow = exp(−max(0.5 − dA, 0) · 12)`, used by P4.
- Output: RGBA8 at output resolution (1920×1080). R = coverage, G = glow, B = letter index / 16 (for per-letter colour), A = coverage.

**P2 — Echo feedback (ping-pong)**
- Purpose: short trails of past letter shapes.
- Inputs: P1, `prev` (last P2 result), uniforms `zoom` (1.0–1.03), `decay` (0.7–0.9), `echoMix`.
- Per pixel: `q = (uv − 0.5) / zoom + 0.5`, `e = prev(q) · decay`, `out = max(P1.r, e)`.
- Output: R16F at half resolution (960×540) to keep trails soft and cheap. Two buffers swap every frame, and the output is upscaled bilinear in P3.

**P3 — Composite**
- Inputs: P1, P2, `fill` (video texture), palette uniforms, `fillMix`, `echoMix`, `flash` (kick envelope).
- Per pixel: `typeCol = mix(colType, colAccent, flash · step-per-letter)`, `plain = P1.r · typeCol`, `window = P1.r · fill.rgb`, `c = mix(plain, window, fillMix)`, `c = max(c, P2 · colEcho · echoMix)`.
- Output: RGBA8, output resolution.

**P4 — Post**
- Black crush: `c = max(c − 0.02, 0) / 0.98`.
- Bloom: threshold 0.85, intensity 0.5, radius 0.8 % of frame height. Alternatively add `P1.g · colGlow · 0.4` (the SDF glow), which is cheaper and does not bleed into the fill video.
- Grain: 1.5 % monochrome, only on non-black pixels (`grain · step(0.02, luma)`), so the black stays off on LEDs.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base | #000000 | background, always true black |
| type | #F4F1EA | warm white for the letters (pure #FFFFFF glares on LED walls) |
| accent / kick | #FF4A1C | flash colour on the kick punch, per letter with the stagger |
| cool accent | #2FD3F0 | echo trails and the outline ghost |
| shadow | #17151C | outline ghost at low intensity, breaks only |
| highlight | #FFF3C4 | SDF glow tint in the drop |

- Mapping: flat colour, not a gradient. The type is graphic, and colour comes either from the accent flash or from the fill video seen through the type matte.
- Over time: verses are monochrome (type only). The build adds the cool echo. The drop brings the orange kick flash and `fillMix` at 70–100 %. Rotate accent pairs per phrase (8 bars) between orange/cyan and #E83FFF/#C6FF3D if the show needs more variety.
- LED walls: background at 0, type peak at about 92 % (#F4F1EA is already below full). Heavy weights on a full-height word can light 40–50 % of the wall at near-white, so cap the type at 80 % in the drop if power or glare matters. Thin weights (wght < 300) disappear on coarse walls: a stroke needs at least 3 LED pixels, so raise `base` until it does.

## Blending and compositing

- Type over black in **Add** (layer 5): black is neutral, and it stacks cleanly with the echo.
- Type as a window (layer 4): in Arena, use layer 3 as a mask for layer 2 in luma (or alpha) mode. Use Multiply only if the type source has no alpha. The window look sells the "type as texture" idea from the research and hides the fact that the shapes repeat.
- Echo with **Lighten (max)**, not Add. Add would blow the overlapping trails to white and lose the letter shapes.
- Feedback mix 0–0.6, decay 0.82 by default. Above 0.9 the word smears into a block and stops reading as type.
- Post order: crush → bloom (or SDF glow) → grain. No chromatic aberration by default; on type it reads as a mistake at large sizes. Optionally use 2 px RGB split on the kick only.
- No sharpen: SDF edges are already analytic, and sharpening adds halos on black.

## Motion and timing

- What moves: axis values only. Position stays fixed or changes by hard cut once per phrase, which keeps the motion clearly typographic.
- Wave: `rate` 0.25 (1 cycle/bar) by default, range 0.125–1. `spread` 1 by default, range 0–3. Linear phase; the sine does the easing.
- Kick punch on weight: +0.6 axis units, attack 10 ms, release 250 ms (about half a beat at 128 BPM), stagger 15 ms per letter.
- Bass on width: 0–0.4 axis units, attack 60 ms, release 400 ms. Hats: slant jitter ±0.05 or a 2-frame `bold` flicker.
- Words: one word per bar in verses (lyric or keyword strobe, research variation 1). Change on the downbeat, never mid-bar.
- Build (last 4 bars): `depth` rises 0.1 → 0.8, `rate` doubles every bar (0.25 → 2), `echoMix` rises 0 → 0.4.
- Drop: hard cut to `base` 0.2 (thin) so the first kick punches to black, `fillMix` 1, accent flash on, echo 0.5. With a morph font, drive MORF from the kick instead of wght.
- Break: `freeze` on, then release one letter at a time per beat (left to right), switch to outline mode (`outline` 0.04) and fade the echo out over 2 bars.
- Never run the same mapping for more than 16 bars. Alternate wave-only, punch-only and hold.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| weight | 0–1 (font wght min–max) | 0.35 | base weight of the whole word | BeatStep Pro knob 1 |
| punch | 0–1 | 0.6 | how hard the kick slams the weight | knob 2; kick envelope × punch |
| wave | 0–1 (depth) | 0.3 | how strongly the per-letter wave swells | knob 3; KeyStep mod strip for live swells |
| spread | 0–3 | 1 | how many wave cycles across the word (0 = all letters together) | knob 4 |
| width | 0–1 (wdth min–max) | 0.5 | narrow to wide; bass rides on top | knob 5; bass envelope ×0.4 |
| echo | 0–0.6 | 0 | feedback trail amount | knob 6; pad toggles 0 / 0.5 |

Pads on the BeatStep Pro: next word, freeze, outline mode, fill-window on/off. Wave rate follows the host BPM; a pad toggles ×2.

## Build recipes

### Resolume Arena

- **Route A (recommended for the full effect):** run the Godot renderer (see Godot recipe) and receive it in Arena as a Spout source. Alternatively, open the browser page as an OBS Browser Source and send it with DistroAV's NDI Output or NDI Filter ([DistroAV](https://github.com/DistroAV/DistroAV)), then receive it as an NDI source in Arena ([Resolume sources](https://resolume.com/support/en/sources)). Control: Arena sends OSC on beat and BPM, or the renderer takes MIDI directly from the BeatStep Pro.
- **Layer set-up:** L1 black solid; L2 fill video; L3 type source (Spout/NDI), used as a mask for L2 in luma mode with the window on; L4 the same type source in Add with a Colorize effect at #F4F1EA (layer opacity = plain type amount); L5 an echo layer, either Arena's Feedback effect on L4 (decay about 0.8, Lighten if the effect offers it) or the ISF P2 below. Post on the composition: Bloom (threshold 0.85, low intensity) and grain from any noise effect at 1–2 %.
- **Route B as a custom ISF effect:** load the 4096×2560 super-atlas as a clip on L3 and apply the type ISF (P1 + P3 colour) as a **clip effect**, so the effect's input image is the atlas. Uniforms for the letters (`glyphIdx`) are float inputs 0–39, one per letter slot (up to 8 letters). `base`, `punch`, `depth`, `spread`, `rate` and `slant` are float inputs; `beatPhase` comes from Arena's BPM-synced parameter animation (sawtooth over 1 beat); `env` comes from audio analysis (Low band). Test whether your Arena build feeds the clip effect at the atlas's native resolution. If it scales the input to composition size, the cells shrink: compensate by making the atlas exactly the composition size (1920×1080 with 96 px cells, fewer steps).
- ISF caveats: Resolume's ISF documentation lists float, bool, colour and event inputs and does not cover image inputs or multipass ([Resolume ISF](https://resolume.com/support/en/isf)). Keep P1 single-pass and use Arena's own Feedback for the echo.
- **Route C:** sweep clips per word in a column. Use BPM sync ping-pong over 1 beat, or OSC transport position from an external audio tool ([Resolume OSC](https://resolume.com/support/en/osc)). Trigger words with a pad.

### Resolume Wire

- Patch as an **effect**, applied in Arena to the atlas clip. The input texture is the atlas (route B), so there is no need for an image input inside the ISF.
- Nodes: Texture input (effect input = atlas) → ISF node `VarType` (P1, single pass; fragment shader loaded via the node's Fragment Shader attribute, [Resolume ISF](https://resolume.com/support/en/isf)) → mix stage for colour (P3) → Output.
- Control nodes: a BPM/beat phase node → `beatPhase`; an audio analysis low band → envelope (attack 10 ms, release 250 ms) → `env`; a mid-low band → width. A small array/list of 8 integers for the word, selected by a counter that advances on every 4th beat (one word per bar).
- Exposed parameters: weight, punch, wave, spread, width, echo, word index, freeze.
- Wire has no vertex shaders, so the type cannot be laid out on per-letter quads. Everything happens per pixel in the fragment shader, which is why the slot layout is used ([Resolume ISF](https://resolume.com/support/en/isf)). Multipass ISF depends on the Wire version: do the echo with Wire's own feedback/delay node instead of a second ISF pass, and check node names in your node browser.

### Unreal Engine 5.7

- No native variable-font axes were found (see Tooling reality). Use route B as the main path and route D for punches.
- **Material `M_VarType`** (User Interface domain, on a fullscreen UMG Image, or Post Process for a camera view): TexCoord → slot math (Multiply by n, Floor, Frac) → glyph index from a small `T_Word` texture (n×1, R8, one glyph index per texel, written by Blueprint) → cell UVs (Fmod, Floor, Divide by atlas size) → two Texture Sample nodes on `T_AxisAtlas` (sampler Linear, no mips, Clamp) → per sample, a median of RGB (Min/Max nodes) → Lerp by axis fraction → coverage (Subtract 0.5, Multiply by screen pixel range, Saturate) → Final Color and Opacity.
- **Per-letter axis values:** write them into a second n×1 texture `T_Axis` (R16F) each tick from Blueprint, or evaluate the wave formula in the material from MPC scalars (`Base`, `Depth`, `Spread`, `Rate`, `BeatPhase`, `Env`).
- **Physics variant (research variation 5):** a Niagara system with one particle per letter (Spawn Burst Instantaneous, count = n), Sprite Renderer with `M_VarType_Sprite`. Glyph index, axis value and phase go in through Dynamic Material Parameter (index 0: x = glyph, y = axis, z = phase). Use Point Attraction Force plus Collision against a box the size of the wall, and drive the axis from particle speed: `axis = saturate(length(Velocity) / 800)`, so fast letters get heavy.
- **Pseudo-weight on any UI text (route D):** enable *Enable Distance Field Font Rasterization* in Project Settings > Engine > User Interface > UMG Fonts, enable distance field rendering on the font face, and build a font material with *Font Signed Distance* plus an offset parameter `Bold` ([Epic docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/using-signed-distance-field-text-rendering-in-unreal-engine)).
- **Live route A inside UE:** receive the Godot/browser output as an NDI or Spout texture and use it as a matte or an emissive texture on 3D geometry.
- **Remote Control:** a preset with MPC `MPC_VarType` scalars `Base`, `Punch`, `Depth`, `Spread`, `Width`, `Echo`, `FillMix`, `Freeze`, plus a Blueprint function `NextWord`. Bind to OSC and MIDI (BeatStep Pro). Output via Spout or NDI to Arena.

### Godot

- **Godot 4.5 for Spout.** The Spout bindings spout-gd target 4.5, are Windows only, and provide a `SpoutViewport` node that sends its content at runtime ([spout-gd](https://github.com/you-win/spout-gd)).
- **Scene:** `SpoutViewport` (1920×1080, sender name `VarType`, transparent bg off) → Node2D `TypeRig` with the `_draw()` logic from Technique 2. Font: the variable .ttf imported with MSDF on and MSDF Pixel Range 8. Docs: the pixel range should be at least twice any outline size ([Using fonts](https://docs.godotengine.org/en/stable/tutorials/ui/gui_using_fonts.html)).
- **Variation bank:** `Array[FontVariation]` of 64 entries per axis, built at `_ready()` from `get_supported_variation_list()` min/max. Warm up by drawing every variation once off-screen in the first frame, so the drop does not stall on glyph generation (MSDF glyphs are rasterised on the CPU the first time; [Using fonts](https://docs.godotengine.org/en/stable/tutorials/ui/gui_using_fonts.html)).
- **Fallback for fonts with self-intersecting outlines:** these break in MSDF mode (common with Google Fonts downloads, [Using fonts](https://docs.godotengine.org/en/stable/tutorials/ui/gui_using_fonts.html)). Turn MSDF off and fix font size per word, or get the font from the designer's site.
- **Route B in Godot:** a `ColorRect` with a `canvas_item` shader (P1), the super-atlas as `sampler2D` with `filter_linear` and `repeat_disable`, and P2 via two SubViewports in ping-pong.
- **Uniforms/properties to expose:** `weight`, `punch`, `wave`, `spread`, `width`, `slant`, `echo`, `word_list`. Input: MIDI from the BeatStep Pro through Godot's MIDI input events; audio from `AudioEffectSpectrumAnalyzer` on a capture bus (kick band 40–120 Hz, bass 60–250 Hz).
- **Fake slant:** if the font has no slnt axis, `FontVariation.variation_transform = Transform2D(1, slant, 0, 1, 0, 0)` gives a synthetic slant. Avoid `variation_embolden` with MSDF, because emboldened outlines can self-intersect and break MSDF ([FontVariation](https://docs.godotengine.org/en/stable/classes/class_fontvariation.html)).

## Audio reactivity

| band | drives | smoothing | threshold / scaling |
|------|--------|-----------|---------------------|
| kick, 40–120 Hz | weight punch (or MORF on morph fonts) | attack 10 ms, release 250 ms | trigger at 0.6 of the 2 s running peak, retrigger lock 120 ms |
| bass, 60–250 Hz | width | attack 60 ms, release 400 ms | linear above a 0.2 floor, ×0.4 |
| hats, 6–12 kHz | slant jitter ±0.05 or bold flicker | attack 5 ms, release 60 ms | trigger at 0.5 |
| spectrum, n bands (log spaced) | per-letter weight: letter i = band i (word as equaliser, as in Gluk's visualisers from the research) | per band `s = max(new, s · 0.88)` | only in a dedicated section; it drowns the wave otherwise |
| RMS loudness | `base` weight drift 0.2 → 0.5 | attack 300 ms, release 3 s | slow, so loud passages get heavier overall |

Variable Time uses the same model of volume, pitch or clarity mapped to axes with min/max and smoothing (see research), which suggests it reads well.

## Performance tips

- **Route B is cheap:** two atlas taps per pixel (four for two axes) plus a few ALU ops. Well under 0.3 ms at 1080p; the 4096×2560 RGBA8 atlas costs 40 MB of VRAM.
- **Mipmaps off** on every SDF atlas. Mips blend neighbouring cells and the distance values, which causes grey fringes and bleeding between steps. Also keep a 1 px inset inside each cell.
- **Pixel range:** with `-pxrange 8` and 128 px cells the edge stays sharp up to about 16× magnification. A 1080 px tall glyph from a 96 px em is roughly 11×, which is fine. For walls taller than 2000 px, regenerate at 192 px cells.
- **Route A (Godot):** CPU cost grows with the number of distinct variations used per frame. Quantised banks (64 steps) keep it bounded. Spout adds a GPU copy, while NDI adds encode and network latency, so measure it in rehearsal against the kick and offset the audio envelope if the type lags.
- **Browser route:** changing wdth reflows layout on the CPU every frame. Stay under about 30 spans and animate wght or GRAD where possible; MDN notes that GRAD changes stroke weight without changing width, so there is no reflow ([MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Fonts/Variable_fonts)).
- **Odd aspect ratios:** for a 5760×1080 strip, the slot layout makes each letter of a 6-letter word 960 px wide and the letters get stretched. Instead, keep glyph aspect fixed from the box height and centre the word, or repeat the word 3× across the strip with a phase offset of 1/3 per repeat for a continuous wave. On tall portrait walls, stack letters vertically (slot along y).
- **Legibility vs. texture:** the full effect needs no legibility. If text must be read (lyrics, names), keep `spread` ≤ 1, `punch` ≤ 0.5 and echo off.
- **Pitfalls:** fonts whose axis defaults are not at the min end (`get_supported_variation_list()` returns the default) make `a = 0` look different from what you expect, so always map from the real min/max. Morph fonts with contextual alternates (Kablammo) change repeated letters in live renderers but not in a route B atlas, which holds one glyph per character.

## Further references

- [msdf-atlas-gen (Chlumsky)](https://github.com/Chlumsky/msdf-atlas-gen) — the atlas generator for route B; supports variable fonts via `-varfont` and a uniform grid with fixed glyph origins.
- [Godot docs: Using fonts](https://docs.godotengine.org/en/stable/tutorials/ui/gui_using_fonts.html) — variable fonts with FontVariation, MSDF import options and their limits.
- [Epic docs: Signed distance field text rendering](https://dev.epicgames.com/documentation/en-us/unreal-engine/using-signed-distance-field-text-rendering-in-unreal-engine) — UE 5.5+ SDF/MSDF UI text and the Font Signed Distance material node.
- [MDN: Variable fonts guide](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Fonts/Variable_fonts) — registered axes, `font-variation-settings` and the custom-property pattern for per-letter control in the browser route.
