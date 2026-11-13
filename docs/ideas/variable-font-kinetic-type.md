---
tags:
  - typography
  - text
  - motion
  - design
---

# Variable Font Kinetic Typography

> One typeface, one oversized word, and letters that swell, squash and lean on the beat without ever swapping a font.

![Grid of the letter B interpolated along two axes (width and weight; labels in Swedish), showing how a variable font moves between masters](https://upload.wikimedia.org/wikipedia/commons/thumb/6/68/Interpolation_variable_fonts.png/1280px-Interpolation_variable_fonts.png)
*Grid of the letter B interpolated along two axes (width and weight; labels in Swedish), showing how a variable font moves between masters. Image: [Krisduck](https://commons.wikimedia.org/wiki/File:Interpolation_variable_fonts.png), [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0).*

## The idea
A variable font stores a few extreme designs and the rules to blend them. Each axis, such as weight or width, has a minimum and a maximum, and every value in between is valid ([variablefonts.io](https://variablefonts.io/about-variable-fonts/)). That makes a letter a continuous object. You can turn it from hairline to black with a fader.

On a club screen this is a typographic instrument. Set one word huge, white on black, cropped at the edges. Let the kick punch the weight, let the bass breathe the width, and send a slow wave through the letters so each one swells a little after its neighbour. The word stays the same typeface and only changes body. That separates it from a font-switch strobe.

Legibility is optional in VJing. Oversized and cropped, type reads as texture, and as a matte it becomes a window onto other video.

## Origins and references
- **OpenType 1.8, 2016.** Announced on 14 September 2016 at ATypI in Warsaw by Google, Microsoft, Apple and Adobe together, reviving Apple's TrueType GX variations ([Google Open Source Blog](https://opensource.googleblog.com/2016/09/introducing-opentype-font-variations.html)). The motive was efficiency, many weights in one file. Animation was a side effect of continuous axes ([CSS-IRL](https://css-irl.info/variable-font-animation-with-css-and-splitting-js/)).
- **Kablammo** (Vectro Type Foundry for Google; Travis Kochel, Lizy Gershenzon). A Morph axis from 0 to 60 moves the letter shapes through four named states: Zoink, Bloop, Splat, Eek ([Google Design](https://design.google/library/kablammo-the-dancing-font-from-outer-space)).
- **COLLINS x San Francisco Symphony.** The identity uses variable type so each character changes form in reaction to sound and music ([Typeroom](https://www.typeroom.eu/collins-san-francisco-symphony-variable-type)).
- **Animated pixelated variable fonts** (Toshi Omagari, 2017). Short film clips play by sliding one axis ([tosche.net](https://tosche.net/AnimatedPixelatedVF/)).
- **Variable Time** (studio Pointer with Celine Hurka). Open-source tool for type in motion, with axes driven by volume, pitch or clarity ([Variable Time](https://variabletime.pointer.click/)).
- **Type Physics+** (type.today). Words bounce, orbit or collapse while axes animate with the motion ([Type Physics+](https://typephysics.type.today/)).
- **Gluk's audio visualizers** (2025). A variable colour font and a second variable font used as audio visualizers ([TypeDrawers](https://typedrawers.com/discussion/comment/70367)).

## Visual language
Letters that breathe, swell, squash and lean, with no jumps. Per-letter phase offsets create a wave through the word, like a crowd doing the wave. Weight and width changes reflow the line, so the text feels elastic.

Classical axes feel editorial and calm. Morph-style fonts such as Kablammo feel playful and cartoonish, and on the drop the letters wobble through cartoon shapes.

The palette is flat and graphic. Warm white (#F4F1EA) type on true black (#000000). The kick flashes orange (#FF4A1C), echo trails and outline ghosts are cyan (#2FD3F0). Verses are monochrome and held. The build adds the cool echo. The drop brings the orange flash and lets the letters act as windows onto colour video.

Position stays fixed, or changes by hard cut once per phrase. Only the axis values move, which keeps the motion clearly typographic.

## See it
- [Kablammo, Google Design](https://design.google/library/kablammo-the-dancing-font-from-outer-space) — morphing letterforms of the Morph axis.
- [Animated pixelated variable fonts](https://tosche.net/AnimatedPixelatedVF/) — film clips played by one font axis.
- [Type Physics+](https://typephysics.type.today/) — interactive words bouncing with axes animating.
- [Variable Time](https://variabletime.pointer.click/) — open-source type-in-motion tool with audio mapping.
- [Variable, color font as audio visualizer (YouTube, Gluk)](https://youtu.be/qAvhR4Owsxk) — variable colour font reacting to audio.
- [Variable font (WiktoriV) as audio visualizer (YouTube, Gluk)](https://youtu.be/biNWMPaUkYA) — second audio visualizer test.

## Code design
The core is one axis-driving model. Each axis gets a normalised value `a` from 0 to 1, mapped to the font's real range with `mix(min, max, a)`. For letter `i` of `n`:

`a_i = clamp(base + depth * w_i + punch * env_kick + jitter_i, 0, 1)`

The wave term is `w_i = 0.5 + 0.5 * sin(2π * (beatPhase * rate - i * spread / n))`. `rate = 0.25` gives one cycle per bar. The kick envelope uses a 10 ms attack and a 250 ms exponential release, with a per-letter delay of 0 to 40 ms so the punch ripples through the word. A `freeze` switch latches all values, because constant motion gets boring.

### Tooling reality
No VJ tool in this stack was found to expose variable font axes natively.

- **Resolume Arena / Wire:** no axis control found in Arena's text sources. Resolume's ISF documentation does not cover fonts ([Resolume ISF](https://resolume.com/support/en/isf)).
- **Unreal Engine 5.7:** no axis control found. A 2024 forum feature request says the feature is missing and has no answer from Epic ([UE forum](https://forums.unrealengine.com/t/feature-request-variable-fonts/1897780)). UE 5.5 and newer render UI text as SDF/MSDF and add a *Font Signed Distance* material node. That helps with a fake weight, but the docs never mention variable fonts ([Epic docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/using-signed-distance-field-text-rendering-in-unreal-engine)).
- **Godot 4:** full native support. A `FontVariation` resource takes OpenType axis coordinates in `variation_opentype`, and `Font.get_supported_variation_list()` returns each axis as min, max and default ([FontVariation](https://docs.godotengine.org/en/stable/classes/class_fontvariation.html), [Font](https://docs.godotengine.org/en/stable/classes/class_font.html), [Using fonts](https://docs.godotengine.org/en/stable/tutorials/ui/gui_using_fonts.html)).
- **Browsers:** full native support through CSS `font-variation-settings` and the standard properties `font-weight`, `font-stretch` and `font-style` ([MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Fonts/Variable_fonts)).

That gives four routes, and the design works with all of them.

| route | true variable outlines? | per-letter control | runs inside | best for |
|-------|------------------------|--------------------|-------------|----------|
| **A. Live renderer to Spout/NDI** | yes, live | yes | Godot 4 app (Spout) or browser page in OBS (NDI) | the full effect, any text typed live |
| **B. Axis-sweep MSDF atlas + shader** | yes, sampled at K steps and interpolated | yes | ISF (Arena/Wire), UE material, Godot shader | self-contained in one tool, fixed charset |
| **C. Axis-sweep clip, scrubbed** | yes, pre-rendered | whole word only | Arena clip transport | fastest set-up, one word per clip |
| **D. SDF pseudo-weight** | no, distance offset only | yes | UE 5.5+ UMG text, any SDF shader | quick fallback for any static font |

**Route B, the portable core.** Offline, generate K MSDF atlases of the same charset, one per axis step, with msdf-atlas-gen. Use `-varfont "font.ttf?wght=<v>"` for each step value, plus `-type mtsdf -uniformgrid -uniformcols 8 -uniformcell 128 128 -uniformorigin on -pxrange 8 -size 96 -yorigin top` ([msdf-atlas-gen](https://github.com/Chlumsky/msdf-atlas-gen)). The default is K = 16 weight steps and a 40-glyph charset (A–Z, 0–9, `! ? . -`), tiled into a 4096×2560 RGBA8 super-atlas (40 MB). In the shader, each pixel finds its letter slot, samples the same cell in two neighbouring step atlases, takes the median of RGB for each, and mixes by the axis fraction. Neighbouring instances are close, so the interpolated distance reads as an in-between master. Morph axes that change topology, such as Kablammo's 0–60 Morph, need denser sampling (K = 61) or a short cross-fade. The default layout is "slot": each letter is centred in an equal-width slot, so no advance table is needed.

**Route D, pseudo-weight.** Shift the SDF threshold: `coverage = clamp((d - 0.5 + bold) * pxRangeScreen + 0.5, 0, 1)`, with `bold` from -0.15 to +0.25. This is not a weight axis. Counters fill, stems keep their length and corners round off. It reads as weight only when it changes fast, so use it for the kick punch and use a real axis for the slow wave.

**Shader chain.** P1 draws the type. P2 is echo feedback: `q = (uv - 0.5) / zoom + 0.5`, `out = max(P1.r, prev(q) * decay)`, at half resolution. P3 composites type, echo and the fill video seen through the type. P4 is post: black crush, glow from the SDF alpha, and 1.5 % grain only on non-black pixels so the black stays off on LEDs.

**Layer stack.**

| # | layer | blend | opacity |
|---|-------|-------|---------|
| 1 | Black solid #000000 | Normal | 100 % |
| 2 | Fill video (optional), seen only through the type matte | Normal | 100 % |
| 3 | Type source: white type on black plus coverage | used as matte | — |
| 4 | Type matte: layer 3 cuts layer 2 in luma or alpha mode | mask | 0–100 % |
| 5 | Type plain, tinted with the palette colour | Add | 100 % |
| 6 | Echo feedback: previous output ×1.015 scale, ×0.82 fade | Lighten (max) | 0–60 % |
| 7 | Outline ghost: thin outline at a lower weight, route B/D only | Add | 40 % |
| 8 | Post: black crush, bloom, grain | — | — |

Use Lighten, not Add, for the echo. Add blows overlapping trails to white and loses the letter shapes. Above a decay of 0.9 the word smears into a block.

=== "Resolume Wire"
    Patch it as an **effect**, applied in Arena to the atlas clip (route B). The effect input is the atlas, so the ISF needs no image input of its own.

    - Nodes: Texture input (the atlas) → ISF node `VarType` (P1, single pass, loaded through the node's Fragment Shader attribute, [Resolume ISF](https://resolume.com/support/en/isf)) → mix stage for colour (P3) → Output.
    - Controls: a BPM/beat phase node feeds `beatPhase`. A low-band audio analysis through an envelope (attack 10 ms, release 250 ms) feeds `env`. A mid-low band feeds width. A list of 8 integers holds the word, and a counter advancing every 4th beat picks the word, one per bar.
    - Exposed parameters: weight, punch, wave, spread, width, echo, word index, freeze.
    - Wire has no vertex shaders, so the type cannot sit on per-letter quads. Everything runs per pixel in the fragment shader, which is why the slot layout is used. Multipass ISF depends on the Wire version, so do the echo with Wire's own feedback or delay node and check node names in your node browser.
    - In Arena, route A is the recommended way to get the full effect. Run the Godot renderer and receive it as a Spout source. Or send a browser page from OBS through DistroAV's NDI output ([DistroAV](https://github.com/DistroAV/DistroAV)) and receive it as an NDI source ([Resolume sources](https://resolume.com/support/en/sources)).
    - Route C: sweep clips per word, with BPM-synced ping-pong over 1 beat, or the clip position driven over OSC, e.g. `/composition/layers/3/clips/1/transport/position` ([Resolume OSC](https://resolume.com/support/en/osc)). There is no per-letter phase, so prefer route B for the wave.

=== "Unreal Engine 5.7"
    No native variable-font axes were found. Use route B as the main path and route D for punches.

    - **Material `M_VarType`** (User Interface domain, on a fullscreen UMG Image, or Post Process): TexCoord → slot math (Multiply by n, Floor, Frac) → glyph index from a small `T_Word` texture (n×1, R8, written by Blueprint) → cell UVs → two Texture Sample nodes on the atlas (Linear, no mips, Clamp) → median of RGB → Lerp by axis fraction → coverage → Final Color and Opacity.
    - **Per-letter axis values:** write them into an n×1 R16F texture each tick, or evaluate the wave formula in the material from Material Parameter Collection scalars (`Base`, `Depth`, `Spread`, `Rate`, `BeatPhase`, `Env`).
    - **Physics variant:** a Niagara system with one particle per letter and a sprite material. Glyph index, axis value and phase go in through Dynamic Material Parameter. Use Point Attraction Force and Collision against a box the size of the wall, and drive the axis from particle speed, so fast letters get heavy.
    - **Pseudo-weight on any UI text (route D):** enable *Enable Distance Field Font Rasterization* in Project Settings > Engine > User Interface > UMG Fonts, enable distance field rendering on the font face, and build a font material with *Font Signed Distance* plus a `Bold` offset. Growing beyond the glyph needs *Material Is Stencil* on the text, plus an outline of similar width ([Epic docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/using-signed-distance-field-text-rendering-in-unreal-engine)).
    - **Live route A inside UE:** receive the Godot or browser output as an NDI or Spout texture and use it as a matte or emissive texture.
    - **Remote Control:** a preset exposing the MPC scalars `Base`, `Punch`, `Depth`, `Spread`, `Width`, `Echo`, `FillMix`, `Freeze`, plus a Blueprint function `NextWord`. Bind to OSC and MIDI. Output via Spout or NDI to Arena.

=== "GODOT"
    Godot is the one tool with real variable-font axes, so it is the best home for route A.

    - **Version and output.** Godot 4.5 for Spout. The spout-gd bindings target 4.5, are Windows only, and provide a `SpoutViewport` node ([spout-gd](https://github.com/you-win/spout-gd)). Scene: `SpoutViewport` (1920×1080, sender name `VarType`) → Node2D `TypeRig`.
    - **Font import.** Import the variable .ttf with MSDF on and MSDF Pixel Range 8. The pixel range should be at least twice any outline size ([Using fonts](https://docs.godotengine.org/en/stable/tutorials/ui/gui_using_fonts.html)).
    - **Variation bank.** Build an `Array[FontVariation]` of 64 entries per axis at `_ready()`, from the min and max in `get_supported_variation_list()`. Quantising caps the number of glyph caches, because a cache exists per variation. That limit is a design caution, not a documented one. Draw every variation once off-screen in the first frame, because MSDF glyphs are rasterised on the CPU the first time.
    - **Drawing.** In `_draw()`, draw letter by letter with `draw_char`, picking the font from the bank by `round(a_i * 63)`, and `draw_set_transform` for per-letter scale or rotation ([CanvasItem](https://docs.godotengine.org/en/stable/classes/class_canvasitem.html)). Call `queue_redraw()` every frame. Advance `x` by `get_string_size(char)` of the chosen variation and centre the word, so weight and width reflow the line.
    - **Route B in Godot.** A `ColorRect` with a `canvas_item` shader (P1), the super-atlas as `sampler2D` with `filter_linear` and `repeat_disable`, and the echo through two SubViewports in ping-pong.
    - **Fake slant.** If the font has no `slnt` axis, `FontVariation.variation_transform` gives a synthetic slant. Avoid `variation_embolden` with MSDF, because emboldened outlines can self-intersect and break MSDF ([FontVariation](https://docs.godotengine.org/en/stable/classes/class_fontvariation.html)).
    - **Self-intersecting outlines** break MSDF mode, which is common with Google Fonts downloads. Turn MSDF off and fix the size per word, or get the font from the designer's site.
    - **Input.** MIDI from the BeatStep Pro through Godot's MIDI input events. Audio from `AudioEffectSpectrumAnalyzer` on a capture bus (kick band 40–120 Hz, bass 60–250 Hz).
    - **Browser alternative.** One `<span>` per letter, with CSS custom properties `--wght` and `--wdth` inside `font-variation-settings`, the pattern MDN recommends ([MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Fonts/Variable_fonts)). Splitting text into spans follows the [CSS-IRL](https://css-irl.info/variable-font-animation-with-css-and-splitting-js/) article. Set `font-synthesis: none` so the browser adds no fake bold.

### Test before a gig
- Whether your Arena build feeds a clip effect at the atlas's native resolution. If it scales the input to composition size, the cells shrink. Then make the atlas exactly composition size (1920×1080 with 96 px cells, fewer steps).
- Whether W and M at maximum weight fit the 128 px cell. If not, use 160 px cells (5120×3200 atlas).
- The node names and multipass support in your Wire version.
- Latency of the Spout or NDI link against the kick. NDI adds encode and network delay. Offset the audio envelope if the type lags.
- Your font's axis defaults. If the default is not at the minimum, `a = 0` will not look as expected, so always map from the real min and max.
- Self-intersecting outlines in MSDF mode, before the font goes on the set list.
- Kablammo-style contextual alternates: live renderers change repeated letters, but a route B atlas holds one glyph per character.
- Thin weights on the real wall. Under wght 300 they vanish on coarse walls, since a stroke needs at least 3 LED pixels. Raise `base` until it holds.
- A wide strip such as 5760×1080: slot layout stretches letters. Keep the glyph aspect fixed from the box height, or repeat the word 3× with a phase offset of 1/3.

## On stage
**Audio reactivity.**

| band | drives | smoothing |
|------|--------|-----------|
| kick, 40–120 Hz | weight punch (or Morph on morph fonts) | attack 10 ms, release 250 ms; trigger at 0.6 of the 2 s peak, 120 ms lock |
| bass, 60–250 Hz | width, ×0.4 above a 0.2 floor | attack 60 ms, release 400 ms |
| hats, 6–12 kHz | slant jitter ±0.05 or a 2-frame bold flicker | attack 5 ms, release 60 ms |
| spectrum, log-spaced | letter i = band i, the word as equaliser | per band `s = max(new, s * 0.88)` |
| RMS loudness | base weight drift 0.2 → 0.5 | attack 300 ms, release 3 s |

Use the spectrum mapping only in a dedicated section, because it drowns the wave. Variable Time uses the same model of volume, pitch and clarity with min/max and smoothing ([Variable Time](https://variabletime.pointer.click/)).

**MIDI mapping (BeatStep Pro, KeyStep 37).**

| control | parameter | range |
|---------|-----------|-------|
| BeatStep knob 1 | weight | 0–1, default 0.35 |
| knob 2 | punch (kick envelope × punch) | 0–1, default 0.6 |
| knob 3 | wave depth | 0–1, default 0.3 |
| knob 4 | spread, wave cycles across the word | 0–3, default 1 |
| knob 5 | width (bass rides on top) | 0–1, default 0.5 |
| knob 6 | echo | 0–0.6, default 0 |
| KeyStep mod strip | live wave swells | — |
| BeatStep pads | next word, freeze, outline mode, fill-window on/off | toggles |

Wave rate follows the host BPM, and a pad toggles ×2.

**Arc of a set.**
- Verses: one word per bar, changed on the downbeat, never mid-bar. Monochrome, calm wave, held.
- Build (last 4 bars): `depth` rises 0.1 → 0.8, `rate` doubles every bar (0.25 → 2), echo rises 0 → 0.4.
- Drop: hard cut to `base` 0.2 so the first kick punches to black. Fill mix at 1, orange flash on, echo 0.5. With a morph font, drive Morph from the kick instead of weight.
- Break: freeze, then release one letter per beat from left to right. Switch to outline mode and fade the echo over 2 bars.

**Tips.**
- Never run the same mapping for more than 16 bars. Alternate wave-only, punch-only and hold.
- Pure amplitude mapping feels mechanical. Vary rhythm: hold, then hit.
- If the text must be read, keep `spread` ≤ 1, `punch` ≤ 0.5 and echo off.
- LED walls: keep the type peak around 92 %. A heavy full-height word can light 40–50 % of the wall at near-white, so cap it at 80 % in the drop if power or glare matters.
- Keep the type crisp. No sharpening and no chromatic aberration by default. An optional 2 px RGB split on the kick only.
- Route B is cheap, under 0.3 ms at 1080p. Keep mipmaps off on every SDF atlas, since they cause grey fringes between steps.
- In the browser route, changing `wdth` reflows layout every frame. Stay under about 30 spans and animate `wght` or `GRAD` where possible, since GRAD changes stroke weight without changing width ([MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Fonts/Variable_fonts)).

## Variations
1. Lyric or keyword strobe: one word per beat, weight punches on the kick.
2. Per-letter phase wave locked to BPM, travelling along the word.
3. Morph-axis fonts (Kablammo-style) as characters that act on the drops.
4. Text as mask or window over live video, axes driven by faders.
5. Physics typography: letters bounce and collide, axes track velocity.
6. Pixel-axis font: play a looping clip as a one-axis font, after Omagari, for a typographic video layer.

## Did you know?
- Four companies announced OpenType 1.8 variable fonts at once, on 14 September 2016 ([Google Open Source Blog](https://opensource.googleblog.com/2016/09/introducing-opentype-font-variations.html)).
- Only five axes are registered (`wght`, `wdth`, `ital`, `slnt`, `opsz`). Everything else is custom ([variablefonts.io](https://variablefonts.io/about-variable-fonts/)).
- Kablammo took about a year and a half from idea to final glyphs ([Google Design](https://design.google/library/kablammo-the-dancing-font-from-outer-space)).
- Variable Time's MP4 export is capped at 1920x1080 total pixels ([Variable Time](https://variabletime.pointer.click/)).

## Sources
- [Google Open Source Blog: Introducing OpenType Font Variations](https://opensource.googleblog.com/2016/09/introducing-opentype-font-variations.html)
- [variablefonts.io](https://variablefonts.io/about-variable-fonts/)
- [Google Design: Kablammo](https://design.google/library/kablammo-the-dancing-font-from-outer-space)
- [Typeroom: COLLINS x San Francisco Symphony](https://www.typeroom.eu/collins-san-francisco-symphony-variable-type)
- [TypeDrawers: Gluk](https://typedrawers.com/discussion/comment/70367)
- [Type Physics+](https://typephysics.type.today/)
- [Variable Time](https://variabletime.pointer.click/)
- [Animated pixelated variable fonts](https://tosche.net/AnimatedPixelatedVF/)
- [CSS-IRL: variable font animation](https://css-irl.info/variable-font-animation-with-css-and-splitting-js/)
- [Derivative forum: variable typography](https://forum.derivative.ca/t/variable-typography/11167)
- [Resolume ISF](https://resolume.com/support/en/isf)
- [Resolume OSC](https://resolume.com/support/en/osc)
- [Resolume sources](https://resolume.com/support/en/sources)
- [UE forum: feature request, variable fonts](https://forums.unrealengine.com/t/feature-request-variable-fonts/1897780)
- [Epic docs: SDF text rendering](https://dev.epicgames.com/documentation/en-us/unreal-engine/using-signed-distance-field-text-rendering-in-unreal-engine)
- [Godot: FontVariation](https://docs.godotengine.org/en/stable/classes/class_fontvariation.html)
- [Godot: Font](https://docs.godotengine.org/en/stable/classes/class_font.html)
- [Godot: Using fonts](https://docs.godotengine.org/en/stable/tutorials/ui/gui_using_fonts.html)
- [Godot: CanvasItem](https://docs.godotengine.org/en/stable/classes/class_canvasitem.html)
- [MDN: Variable fonts guide](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Fonts/Variable_fonts)
- [msdf-atlas-gen](https://github.com/Chlumsky/msdf-atlas-gen)
- [DistroAV](https://github.com/DistroAV/DistroAV)
- [spout-gd](https://github.com/you-win/spout-gd)
