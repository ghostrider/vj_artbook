---
tags:
  - demoscene
  - retro
  - shader
  - plasma
---

# Demoscene Classic Effects

> Plasma, copper bars, rotozoomer and tunnel: four sine tricks from the 90s that lock to a beat without effort.

![Concentric rainbow rings of a chromatic plasma gradient](https://images.unsplash.com/photo-1791960466884-3a956d57b023?w=1400)
*Similar look, not the original work. Photo: [Plasmanica Generator](https://unsplash.com/photos/vibrant-abstract-concentric-color-gradient-qs62YnigrXE) on [Unsplash](https://unsplash.com/license).*

## The idea

Demo effects are small real-time graphics tricks. Their ancestors are early display hacks: the "Bouncing Ball" on the Whirlwind computer (early 1950s) and "munching squares" on the PDP-1 (around 1962) ([Wikipedia: Demo effect](https://en.wikipedia.org/wiki/Demo_effect)). The demoscene took off in the 1980s on the Commodore 64, Atari ST and Amiga, where coders reverse-engineered custom hardware to get new looks. In the MS-DOS era PCs had no special chip, so effects became pixel-level software rendering ([same source](https://en.wikipedia.org/wiki/Demo_effect)).

Today general-purpose 3D engines and procedural content matter more in demos than these effects, so they now read as a retro style. On a club screen that is the point. Saturated palettes, hard pixels and low colour counts say "90s" in one frame.

The rhythm is always periodic and sine-based. Feed beat phase into the sines and one period equals one bar. The effects are also cheap shaders, so the whole chain costs almost nothing on the GPU.

## Origins and references

- **Bret Mulvey, "Plasma" (1988)**: a VGA demo released on CompuServe. It used the diamond-square algorithm and cycled the hardware palette ([Wikipedia: Plasma effect](https://en.wikipedia.org/wiki/Plasma_effect)).
- **Raster / copper bars**: seen on the Atari 2600 and Atari 8-bit, then in C64, Amiga, Atari ST and Amstrad CPC demos. "Copper bar" comes from the Amiga's Copper coprocessor ([Wikipedia: Raster bar](https://en.wikipedia.org/wiki/Raster_bar)).
- **Sanity**: an Amiga rotozoomer built from pre-rendered planar bitmaps and copper effects ([Wikipedia: Demo effect](https://en.wikipedia.org/wiki/Demo_effect)).
- **Kefrens, Megademo 8, "Transformer" part (coded by Promax)**: first appearance of the see-through "glenz" vector look, a name given by Photon ([Wikipedia: Demo effect](https://en.wikipedia.org/wiki/Demo_effect)).
- **Bonz, "Sines and Cosines for Fun and Profit" (Hugi 24)**: a coder's tutorial with formulas for plasma, interference, Lissajous paths, shadebobs, rotozoom, starfield and wormhole ([Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)).
- **Trixter, "Demos Explained" graphics glossary** (archived 2000): defines plasma, tunnel, copper and rotating bitmap ([scheib.net mirror](https://www.scheib.net/play/demos/what/trixter/demo_graphics.html)).
- **Burger, Paulovic, Hasan, "Realtime Visualization Methods in the Demoscene" (CESCG 2002)**: an academic overview of copper, fire, rotozoomer and voxel landscapes ([CESCG](https://old.cescg.org/CESCG-2002/BBurger)).

## Visual language

Plasma is a field of smooth, sine-warped blobs in saturated hue bands. Copper bars are horizontal metallic gradient bands, dark at the edge and white-hot in the core, bobbing on sine paths and crossing each other. The rotozoomer is a tiled texture that spins and breathes in zoom. The tunnel is a fast forward flight through rings.

Hard pixel grids and chunky pixels sell the look. Quantise the palette to 16 steps so colour reads as cycling, not as a smooth gradient. Keep the black true black.

Everything moves in sines and everything locks to the beat. The mood is cheerful and mechanical.

## See it

- [Demos Explained: graphics terms (Trixter)](https://www.scheib.net/play/demos/what/trixter/demo_graphics.html) — glossary with period screenshots of plasma, tunnel and copper
- [Wikipedia: Demo effect](https://en.wikipedia.org/wiki/Demo_effect) — overview list of effects and their history
- [Wikipedia: Raster bar](https://en.wikipedia.org/wiki/Raster_bar) — explanation of raster/copper bars
- [Wikipedia: Plasma effect](https://en.wikipedia.org/wiki/Plasma_effect) — explanation of plasma
- [Realtime Visualization Methods in the Demoscene](https://old.cescg.org/CESCG-2002/BBurger) — paper with demo effect overview and the authors' own demos

## Code design

Three formula effects stack into one image, and a fourth is held back for the drop. At the back, a plasma quantised to a 16-step palette. Across it, five to eight copper bars with a 9-step shade ramp. In the middle, a rotozoomed tile (checker, logo or live feed) behind a mask. On the drop, a hard cut to a tunnel. A pixelate pass (about 320x180), a posterise pass and light scanlines finish the frame. Render the generators at the low internal resolution and upscale with nearest-neighbour. That gives the pixel look for free.

The core maths, after Bonz ([Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)):

- **Plasma**: sum four cosines with different frequencies, speeds and phases. Never give two terms the same value. Map the sum to 0..1, add a palette offset, quantise to 16 steps, look up the palette. Most of the motion comes from the offset, as in the 1988 original ([Wikipedia](https://en.wikipedia.org/wiki/Plasma_effect)).
- **Copper bars**: each bar has a sine-driven centre y. Distance to the centre picks one of 9 shade steps ([Wikipedia: Raster bar](https://en.wikipedia.org/wiki/Raster_bar)). Sort by a fake depth, or use Add for glowing crossings.
- **Rotozoomer**: rotate and scale each pixel back into a tiled texture, `k = 1.5 + sin(t)` for the zoom. Sample with repeat wrap and nearest filtering.
- **Tunnel**: polar coordinates around a Lissajous-moved centre, `v = depth / r + speed * t`. Fade to black at the vanishing point.
- **Shadebobs (optional)**: soft dots on Lissajous paths into a feedback buffer, decay 0.92.

Layer stack, bottom to top: black solid, plasma, tunnel (scene B), rotozoomer, copper bars, optional shadebobs, then the post chain pixelate, posterise, scanlines.

Palettes (selected by `paletteId`): Amiga rainbow (default), copper metal (blue-white 9-step bar ramp, hue-rotated per bar) and fire / C64 warm. Use nearest lookup, no interpolation. On an LED wall keep black at #000000 and cap flat plasma areas at 80 % value so the white bar cores stay the peak.

Seven parameters go on the controller: `cycle`, `warp`, `bars`, `spin`, `pulse`, `chunk`, `scene`.

=== "Resolume Wire"
    - Inputs: a beat phase / BPM node from the host, the seven parameters exposed, and an FFT node for the kick band.
    - Plasma: UV node into four branches of Multiply (frequency) and Add (time x speed), then Cos, sum, scale to 0..1, add the palette offset, Fract, Floor(x*16)/16, then a Sample Texture of a 16x1 palette image.
    - Bars: a loop or a GLSL/ISF shader node over N bars (cap at 12).
    - Rotozoom: Transform 2D (rotation from accumulated beat angle, scale from zoom) into Sample Texture with repeat wrap and nearest filter.
    - Tunnel: a GLSL shader node. Polar maths in nodes is messy.
    - Composite: Blend nodes (Normal, Multiply, Add), render at 320x180, resize with nearest filtering, Posterize, scanline Multiply.

=== "Unreal Engine 5.7"
    - One Unlit material on a full-screen plane or Render Target, with Custom HLSL nodes for plasma, bars, roto and tunnel. The 16x1 palette texture uses Nearest filtering.
    - Pixelation: render to a 320x180 Render Target with Nearest filtering and display it upscaled, or snap UVs in the post material with `floor(uv * grid) / grid`.
    - Niagara variant for 3D copper bars: a Ribbon renderer with 8 ribbons and a sine on Z per ribbon index, colour from a Color Curve with the 9-step ramp. Shadebobs as an additive Sprite emitter on Lissajous paths.
    - Expose the parameters through a Material Parameter Collection and a Remote Control preset. Bind KeyStep / BeatStep CCs in the Remote Control MIDI protocol, or use OSC addresses like `/demo/warp`. Drive `BeatPhase` from the OSC clock.
    - Output by Spout or NDI to Resolume. Spout from Resolume can also feed the rotozoom tile.

=== "GODOT"
    - A full-screen `ColorRect` inside a 320x180 `SubViewport`, shown through a `SubViewportContainer` with nearest texture filter. The pixelation comes for free.
    - A CanvasItem shader with uniforms `beat_phase`, `cycle`, `warp`, `bars`, `spin`, `pulse`, `scene`, plus nearest-filtered, repeating `palette` and `tile` samplers.
    - Shadebobs: a second SubViewport that never clears (or a ping-pong pair) that reads the previous frame times the decay, plus additive GPUParticles2D.
    - Scanlines and posterise in a final shader on the upscaled TextureRect.
    - Drive `beat_phase` from the AudioStreamPlayer position and BPM, or from OSC / `InputEventMIDI`.

## On stage

**Audio reactivity.** Use beat phase from the clock for all periodic motion. Audio only adds accents. Purely audio-driven motion looks jittery against sine-based effects.

- **Kick (40-120 Hz)**: envelope follower, attack 0-5 ms, release 120-150 ms. Drives `pulse`: bar height +40 %, roto zoom +0.3, tunnel speed +50 %.
- **Bass (60-250 Hz)**: slow follower, attack 80 ms, release 400 ms. Modulates `warp` by about 20 % and bar amplitude.
- **Hi-hats (6-12 kHz)**: onset detection. Each onset adds one palette step and a small scanline flicker.
- **Overall level**: slow release, maps to plasma brightness between 60 and 80 %.

**MIDI mapping (KeyStep 37 / BeatStep Pro).**

| Control | Parameter |
|---------|-----------|
| BeatStep Pro knob | `cycle` (0-4 steps per beat), hi-hat adds +1 |
| KeyStep mod strip or fader | `warp` (0.3-3.0) |
| KeyStep keys | `bars`, one bar per semitone from C |
| BeatStep Pro knob | `spin` (0-90 degrees per beat), a pad flips the sign |
| BeatStep Pro knob | `pulse` (0-1), driven by the kick envelope |
| Fader | `chunk`, a pad for 160x90 on breaks |
| BeatStep Pro pad | `scene`, a hard cut quantised to the bar |

**Layering and transitions.**

- Use the stack as a colourful matte or mask for other clips. Add or Multiply over video also works. A Spout texture from UE or TouchDesigner can feed the rotozoom tile.
- The scene A to B switch is a hard cut on the beat (true demo style) or a one-beat crossfade. In Resolume, put scene A and B in two columns and set clip trigger quantisation to 1 bar.
- Break: freeze palette cycling, drop bars to 2, run at half speed, go chunkier at 160x90. Drop: cut to the tunnel, switch palette, return to 320x180, raise bars to 8.
- Every 16 bars, change the palette or swap the rotozoom tile.

**Performance tips.**

- Avoid bilinear filtering anywhere in the chain. One bilinear upscale ruins the look on LED.
- Use an integer upscale factor (1920 / 320 = 6). Non-integer factors give uneven pixel widths that shimmer in motion.
- For odd aspect ratios such as 3840x960, fix the grid height (for example 120 rows), derive the width from the aspect and normalise coordinates by `res.y`.
- Fast palette cycling can flicker on low-refresh LED processors. Clamp to 30 steps per second.
- Shadebob feedback: use a 16-bit float buffer, or subtract 1/255 per frame in 8-bit, so the decay does not stick at a grey floor.
- A static palette, constant speed and a full-screen plasma with no contrast get boring on a big screen. Limit the palette, keep hard edges, change on the tempo and combine two effects.

## Variations

1. Palette from a live source: sample colours from the camera or another Resolume layer instead of a fixed gradient.
2. 3D copper bars as sine-moved ribbons in UE Niagara or a depth-aware shader.
3. Plasma as a displacement map: warp a video clip with the plasma value instead of colouring.
4. A rotozoomer that tiles your own logo or a live feed.
5. Glenz or shadebob trails with a feedback loop for light-painting trails. Bonz describes shadebobs as bitmaps added along a path with a history of positions ([Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)).
6. Kefrens-style bar: one vertical bar whose x follows a sine per scanline, with no clear between lines.

## Did you know?

- The first plasma demo dates to 1988 and used palette cycling, not per-pixel recalculation ([Wikipedia](https://en.wikipedia.org/wiki/Plasma_effect)).
- "Copper bar" comes from the Amiga's Copper coprocessor, which changes colours per scanline with little CPU work. Raster bars were already possible on the Atari 2600 ([Wikipedia](https://en.wikipedia.org/wiki/Raster_bar)).
- The names "glenz" and "blenk" were coined by Photon ([Wikipedia](https://en.wikipedia.org/wiki/Demo_effect)).
- A Lissajous curve with equal frequencies gives a circle. Unequal ones give the wandering paths used for bars and bobs ([Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)).
- Demoparty categories use hard size limits such as 64 KB, 4 KB or 256 bytes, which is why effects are so formula-based ([CESCG paper](https://old.cescg.org/CESCG-2002/BBurger)).

## Sources

- [Wikipedia: Demo effect](https://en.wikipedia.org/wiki/Demo_effect)
- [Wikipedia: Plasma effect](https://en.wikipedia.org/wiki/Plasma_effect)
- [Wikipedia: Raster bar](https://en.wikipedia.org/wiki/Raster_bar)
- [Hugi 24: Bonz, Sines and Cosines for Fun and Profit](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)
- [Trixter, Demos Explained: graphics](https://www.scheib.net/play/demos/what/trixter/demo_graphics.html)
- [CESCG 2002: Realtime Visualization Methods in the Demoscene](https://old.cescg.org/CESCG-2002/BBurger)
- [Resolume REST API docs](https://resolume.com/docs/restapi/)
