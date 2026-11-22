---
tags:
  - look
  - dithering
  - retro
  - pixel
  - video
---

# One-Bit Dithering

> Two colours, hard square pixels, and every grey faked by the density of dots: an engraving that moves.

![Atkinson-dithered image made with the Mirror Dither app](https://live.staticflickr.com/8057/8183886279_b5b4f28938.jpg)
*Atkinson dithering, made with the Mirror Dither app. Image: [Windell Oskay](https://www.flickr.com/photos/oskay/8183886279/), [CC BY 2.0](https://creativecommons.org/licenses/by/2.0/).*

## The idea

Every pixel is compared to a threshold and becomes black or white. Plain thresholding loses all midtones, so dithering spreads the error around. The average over a small area matches the original grey, and the eye does the blending at a distance. Gradients become the star: a vignette or a radial glow turns into a crawling moiré of dots.

The word "dither" appeared in books on analog computation shortly after WWII. Lawrence G. Roberts used it to hide quantization patterns in his 1961 MIT thesis ([Wikipedia: Dither](https://en.wikipedia.org/wiki/Dither)). The 1984 Macintosh had a 512x342 black-and-white screen, and dithered images became a core part of its look ([Beyond Loom](https://beyondloom.com/blog/dither.html)). Lucas Pope carried the look into 3D with *Return of the Obra Dinn*.

On a club screen the pattern itself is the instrument. The kick pumps the image toward white, the bass makes the cells breathe, hi-hats make the grain sparkle, and section changes swap ink and paper. This is a look, not a source. It suits high-contrast footage, camera feeds, 3D renders and feedback layers. Flat mid-grey footage turns into a uniform checkerboard. The effect works best as the last step on a layer or on the whole composition.

## Origins and references

- **Ordered dithering.** Bryce Bayer presented his threshold method for two-level pictures in June 1973 ([Wikipedia: Ordered dithering](https://en.wikipedia.org/wiki/Ordered_dithering)).
- **Error diffusion.** Robert Floyd and Louis Steinberg published their four-neighbour method in 1976. Jarvis-Judice-Ninke, Stucki, Burkes and Sierra followed ([Wikipedia: Floyd–Steinberg](https://en.wikipedia.org/wiki/Floyd%E2%80%93Steinberg_dithering), [Wikipedia: Dither](https://en.wikipedia.org/wiki/Dither)).
- **Bill Atkinson.** Wrote MacPaint and QuickDraw at Apple and designed the Atkinson variant of Floyd–Steinberg ([Wikipedia: Bill Atkinson](https://en.wikipedia.org/wiki/Bill_Atkinson), [Wikipedia: Atkinson dithering](https://en.wikipedia.org/wiki/Atkinson_dithering)).
- **Blue noise.** Robert Ulichney defined it and published the void-and-cluster method in 1993 ([Moments in Graphics](https://momentsingraphics.de/BlueNoise.html)).
- **Return of the Obra Dinn** (Lucas Pope, 2018). A first-person 3D game rendered fully in 1-bit. Pope used dithering "as little as possible" ([PlayStation Blog](https://blog.playstation.com/archive/2019/10/17/lucas-pope-on-return-of-the-obra-dinns-art-style), [PC Gamer](https://www.pcgamer.com/lucas-pope-on-the-challenge-of-creating-obra-dinns-1-bit-aesthetic)).
- **Daniel Temkin, Dither Studies** (prints from 2011/2012). The dither pattern is the subject, with a web-app generator, installations and projections ([Temkin](https://danieltemkin.com/DitherStudies/About/)).
- **Windell Oskay, live Atkinson webcam sketch** (2012). A Processing sketch that dithers webcam video in real time ([Evil Mad Scientist](https://www.evilmadscientist.com/2012/dithering/)).
- **Pope, Mars After Midnight devlog.** On pixel strobing when a dithered image scrolls ([itch.io devlog](https://dukope.itch.io/mars-after-midnight/devlog/285964/working-in-one-bit)).
- Other 1-bit references: Papers, Please, Downwell, Dark Castle, Shadowgate, Deja Vu ([Game Developer](https://www.gamedeveloper.com/design/lucas-pope-and-the-rise-of-the-1-bit-dither-punk-aesthetic)).

![Return of the Obra Dinn: in-game screenshot, rendered in 1-bit](https://obradinn.com/img/shots/Ship-01.png)
*Return of the Obra Dinn, Lucas Pope, 2018. © 3909 LLC. Shown as a quotation for reference; source: [obradinn.com](https://obradinn.com/).*

## Visual language

Pure black and white, no antialiasing. Pixel size is a design choice. Chunky 2-4 px cells read as "Mac". One pixel on a big screen reads as fine grain.

Each algorithm has its own character:

- **Bayer** gives geometric hatch grids and visible banding in gradients. Graphic, print-like.
- **Blue noise** gives organic grain, like film or stipple, with no visible grid ([Moments in Graphics](https://momentsingraphics.de/BlueNoise.html)).
- **Floyd–Steinberg** gives worm-like maze textures.
- **Atkinson** gives high contrast and clean whites, but near-white and near-black areas blow out ([Beyond Loom](https://beyondloom.com/blog/dither.html)).

At large size the pixels stop blending for the eye and the pattern gets messy. Pope used outlines and mostly solid black or white areas for legibility ([PlayStation Blog](https://blog.playstation.com/archive/2019/10/17/lucas-pope-on-return-of-the-obra-dinns-art-style)). Follow his lead: solid shapes, dither only in the transitions, a thin ink outline.

Colour is a strict two-colour lookup, no gradient and no hue shift inside a frame. Presets:

| preset | ink | paper | use |
|--------|-----|-------|-----|
| Mac | `#0B0B0B` | `#F2F2EC` | default, intro |
| Phosphor | `#00140A` | `#33FF88` | techno, cold sections |
| Riso | `#1B1F6B` | `#FF5E7E` | warm or pop sections |
| Sodium | `#140800` | `#FFB347` | breakdowns |
| Blueprint | `#F2F2EC` | `#0A2A8A` | white ink on blue |
| Accent flash | `#000000` | `#FFFFFF` | drop only, 1-2 bars |

## See it

- [Daniel Temkin, Dither Studies](https://danieltemkin.com/DitherStudies/About/) — art series and web app built on dither kernels, shown in installations and projections.
- [Evil Mad Scientist: Atkinson dithering on webcam](https://www.evilmadscientist.com/2012/dithering/) — live webcam turned 1-bit Mac look, with 512x342 window.
- [PlayStation Blog: Pope on Obra Dinn's art style](https://blog.playstation.com/archive/2019/10/17/lucas-pope-on-return-of-the-obra-dinns-art-style) — the 1-bit game look in screenshots and the dithering decisions.
- [PC Gamer: Pope on the 1-bit aesthetic](https://www.pcgamer.com/lucas-pope-on-the-challenge-of-creating-obra-dinns-1-bit-aesthetic) — interview with game stills, smooth versus sharp mode.
- [Game Developer: the 1-bit dither-punk aesthetic](https://www.gamedeveloper.com/design/lucas-pope-and-the-rise-of-the-1-bit-dither-punk-aesthetic) — survey of 1-bit games and Mac art.
- [Moments in Graphics: blue noise textures](https://momentsingraphics.de/BlueNoise.html) — blue-noise textures and dither comparisons.

## Code design

The whole effect is one stateless fragment pass, `Dither1Bit`. It has no history buffer and no ping-pong, which is why it holds up live. Order of work per pixel:

1. Snap to a cell grid of `cellSize` output pixels and average the source over the cell (4 taps).
2. Prepare tone: luma, levels, contrast, gamma, plus a bias that the kick pushes.
3. Get a threshold from the chosen pattern: hard threshold, Bayer 2/4/8 or a blue-noise texture ([Wronski](https://bartwronski.com/2016/10/30/dithering-part-three-real-world-2d-quantization-dithering/)).
4. `bit = luma > threshold`, then optionally force outlines (Sobel on cell-res luma) to ink.
5. `mix(ink, paper, bit)`. Nothing resamples the result afterwards.

Error diffusion (Atkinson, Floyd–Steinberg) is sequential and shimmers on moving video, so it stays offline. Pre-render such clips at final cell resolution and play them undithered. Live, fake the Atkinson punch with blue noise at contrast 1.6-2.0.

The dither is always the **last** effect. Feedback goes before it, never after.

=== "Resolume Wire"
    Build Dither1Bit as a Wire patch and save it as an effect. Drop it last on any layer, or on the composition effects to dither everything.

    1. Input texture goes into a Shader node (GLSL/ISF holding the pass), then to the output texture.
    2. A file-loader node feeds the blue-noise PNG to `blueNoise`. Sampler: nearest, repeat.
    3. Expose parameters as sliders, selectors and colour pickers: Cell, Threshold, Contrast, Gamma, Pattern, Edge, Invert, Ink, Paper.
    4. Bias pump: trigger input, envelope with attack 0 and release about 100 ms, added to Threshold.
    5. Sparkle: a counter on a trigger, multiplied by (0.618034, 0.754878), fract, into `noiseOffset`.
    6. Round Cell to an integer before it reaches the shader.
    7. Palette: an int selector into a lookup of 6 colour pairs.

    In Arena, keep the output slice at native resolution or an integer multiple. Pre-rendered Atkinson clips go on their own layer as DXV at native size, with no effect.

=== "Unreal Engine 5.7"
    A Post Process Material, Blendable Location *Scene Color After Tonemapping*, in an Infinite Extent Post Process Volume. The dither then sees display-space values and comes after bloom, grain and AA.

    - Graph: ViewportUV times ViewSize, divide by Cell, Floor, add 0.5, scale back to UV, sample `SceneTexture: PostProcessInput0`, Dot with (0.2126, 0.7152, 0.0722).
    - Levels, contrast, gamma and bias with Subtract, Multiply, Power, Saturate.
    - Threshold from an 8x8 Bayer or blue-noise texture: Filter Nearest, no mips, sRGB off, R8.
    - `If`/`Step`, then `Lerp(Ink, Paper)` into Emissive Color.
    - All stage values live in a Material Parameter Collection, exposed via Remote Control presets to OSC (for example `/dither/cell`) and to MIDI CCs. Blueprint drives Sparkle and Pump.
    - World-locked variant: sample blue noise triplanar in world space, or rebuild world position from SceneDepth, so the pattern sticks to geometry.
    - Send to Resolume over Spout. NDI compression smears 1-bit patterns. Or send UE undithered and dither in Resolume.

=== "GODOT"
    A `ColorRect` on a top `CanvasLayer` with a `canvas_item` shader, over the source (`VideoStreamPlayer`, camera or Spout texture, or a 3D `SubViewport`).

    - Read `hint_screen_texture` with `filter_linear_mipmap`, so LOD `log2(cell)` gives the cell average. Blue noise: `filter_nearest, repeat_enable`.
    - Uniforms: `cell_size` (int, 1-16), `bias`, `contrast`, `gamma`, `pattern`, `edge`, `noise_offset`, `scroll`, `invert`, `ink_color`, `paper_color`.
    - Optional: render the source into a small `SubViewport` of `window / cell` size, show it with nearest filtering, and threshold that. Cheaper and steadier.
    - GDScript sets uniforms from `InputEventMIDI` and an `AudioEffectSpectrumAnalyzer`. Set default texture filter to Nearest and use `viewport` stretch with integer scaling.

## On stage

**Chain position.** Last. Nothing that blurs, scales, blooms or sharpens may follow it. If the processor scales the canvas, use integer scaling only.

**Audio reactivity.**

| band | drives | notes |
|------|--------|-------|
| Kick, 40-120 Hz onset | Threshold bias +0.2 to +0.4 | attack 0 ms, release 80-150 ms, cap total bias at +0.5 |
| Bass, 30-80 Hz RMS | Cell size 2 to 5 | quantise to integers, change on beats only |
| Hi-hats, 6-12 kHz onset | Sparkle step | min gap 60 ms, so rolls do not become white noise |
| Mids, 300 Hz-2 kHz | Edge 0.2 to 0.8 | keeps leads "drawn" |
| Bar clock | Pattern cycle, palette cut | from Resolume BPM or Ableton Link |

**MIDI.** BeatStep Pro knob 1: Cell (quantised). Knob 2: Contrast. Knob 3: Sparkle. Knob 4: Edge. Pads 1-5: Pattern. Pads 9-14: palette presets. Pad 16: momentary invert. KeyStep 37 mod strip: Threshold, with the kick envelope added on top.

**Set arc.**

- Build-up: cell size grows from 2 to 8 over 8-16 bars, contrast falls from 1.4 to 1.0, sparkle rate doubles every 4 bars.
- Drop: hard cut to cell 2-3, contrast 1.8, Accent flash palette for one bar, then the section palette.
- Break: hard threshold at high contrast for pure silhouettes, or Bayer8 with cells of 10-16 and a slow scroll. Sparkle off.

**Transitions.** Switch palettes on phrase boundaries as a hard cut. A crossfade creates a third colour and breaks the idea. Pattern swaps are instant because the effect is stateless.

**Tips.**

- Run the dither on the native output pixel grid. Uneven cells cause moiré.
- On LED walls at 10 m or more, 1 px dither reads as grey noise. Use cell 2-4 with Bayer4 or blue noise.
- Compute cells in composition space across multiple slices, so the grid stays continuous.
- Keep ink near black and paper below full white. Cap paper to about 70 % of pixels for longer than one bar.
- Fast full-frame sparkle can be uncomfortable. Pope's first fullscreen dither made some playtesters ill ([PC Gamer](https://www.pcgamer.com/lucas-pope-on-the-challenge-of-creating-obra-dinns-1-bit-aesthetic)). Avoid inversions above about 3 Hz and follow venue rules.
- Trim mid-grey sources with Contrast first.

## Variations

1. **Beat-matrix swap.** Cycle Bayer 2x2, 4x4, 8x8 and blue noise on bar boundaries.
2. **Coloured ink.** Replace the ink colour with a sample of a second video layer, such as a slow gradient. Paper stays flat.
3. **Dither as mask.** Use the 1-bit result as a luma matte so one clip's dots cut holes into another.
4. **Two dithers stacked.** Bayer4 and blue noise in Difference blend. Overlapping 1-bit patterns XOR into new textures.
5. **Custom kernels.** Odd kernels or pixel shapes, in the spirit of Temkin's generator ([Temkin](https://danieltemkin.com/DitherStudies/About/)).
6. **Camera- or world-locked pattern.** The pattern travels with a pan or sticks to 3D objects in UE, following Pope ([Pope devlog](https://dukope.itch.io/mars-after-midnight/devlog/285964/working-in-one-bit)).
7. **Audio-reactive error.** Feed the spectrum into diffusion weights so the worm texture reshapes on bass (offline or experimental).

## Did you know?

- Dithering predates computer graphics: random dithering was tried at least as early as 1951 and similar techniques around 1915 ([Wikipedia: Dither](https://en.wikipedia.org/wiki/Dither)).
- Atkinson passes on only three quarters of the error. That is the source of its punchy contrast ([Wikipedia: Atkinson dithering](https://en.wikipedia.org/wiki/Atkinson_dithering)).
- By 2018 Pope raised Obra Dinn's resolution from 640x360 to 800x450 and tied the 3D camera to the 2D pattern to reduce flicker ([PlayStation Blog](https://blog.playstation.com/archive/2019/10/17/lucas-pope-on-return-of-the-obra-dinns-art-style)).
- Christoph Peters released a free database of blue-noise textures under CC0 ([Moments in Graphics](https://momentsingraphics.de/BlueNoise.html)).

## Sources

- [Wikipedia: Dither](https://en.wikipedia.org/wiki/Dither)
- [Wikipedia: Ordered dithering](https://en.wikipedia.org/wiki/Ordered_dithering)
- [Wikipedia: Floyd–Steinberg dithering](https://en.wikipedia.org/wiki/Floyd%E2%80%93Steinberg_dithering)
- [Wikipedia: Atkinson dithering](https://en.wikipedia.org/wiki/Atkinson_dithering)
- [Wikipedia: Bill Atkinson](https://en.wikipedia.org/wiki/Bill_Atkinson)
- [Beyond Loom: Atkinson Dithering](https://beyondloom.com/blog/dither.html)
- [Evil Mad Scientist: dithering](https://www.evilmadscientist.com/2012/dithering/)
- [Moments in Graphics: blue noise](https://momentsingraphics.de/BlueNoise.html)
- [Bart Wronski: Dithering part three](https://bartwronski.com/2016/10/30/dithering-part-three-real-world-2d-quantization-dithering/)
- [PlayStation Blog](https://blog.playstation.com/archive/2019/10/17/lucas-pope-on-return-of-the-obra-dinns-art-style)
- [PC Gamer](https://www.pcgamer.com/lucas-pope-on-the-challenge-of-creating-obra-dinns-1-bit-aesthetic)
- [Game Developer](https://www.gamedeveloper.com/design/lucas-pope-and-the-rise-of-the-1-bit-dither-punk-aesthetic)
- [Pope: Working in One Bit](https://dukope.itch.io/mars-after-midnight/devlog/285964/working-in-one-bit)
- [Daniel Temkin: Dither Studies](https://danieltemkin.com/DitherStudies/About/)
