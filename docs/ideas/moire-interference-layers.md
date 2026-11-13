---
tags:
  - op-art
  - optics
  - interference
  - geometry
---

# Moire Interference Layers

> Two fields of fine lines, one tiny twist, and the whole wall starts to swim.

![Two overlapping sets of concentric circles whose interference forms ellipse and hyperbola moire fringes](https://upload.wikimedia.org/wikipedia/commons/thumb/0/05/Moire_Circles.svg/1280px-Moire_Circles.svg.png)
*Two overlapping sets of concentric circles whose interference forms ellipse and hyperbola moire fringes. Image: [SharkD](https://commons.wikimedia.org/wiki/File:Moire_Circles.svg), [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0).*

## The idea

Lay one fine grating over another and slow, wide bands appear that are drawn in neither layer. They are the difference between the two patterns. Multiply two similar spatial frequencies and you get a difference frequency, and that is the band you see ([Michael Bach](https://michaelbach.de/ot/lum-moire1/)). It is the same math as the beat tone of two close pitches in audio ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)).

The word comes from *moire*, a rippled "watered" silk made by pressing two damp layers of fabric together ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)). The science reached a wide audience through Gerald Oster and Yasunori Nishijima in *Scientific American*, May 1963. Their key point: for near-parallel lines, a small shift of one layer becomes a large shift of the pattern ([SciAm blog](https://www.scientificamerican.com/blog/sa-visual/art-and-science-of-the-moire/)).

That lever is the reason it works on a club screen. A fraction of a degree on one knob sweeps bands across the entire wall. It is cheap to render, it is high contrast, and it is hypnotic when it drifts slowly. It also has a dark twin: the same effect appears when a camera films a screen, and there it is a defect. This chapter uses the first and avoids the second.

## Origins and references

- **Gerald Oster**, *Oster's Magic Moires* (1965) and *Moires and Phosphenes* (1966). Oster (1918–1993) was a biophysicist who became an artist. He showed at Howard Wise Gallery and was in MoMA's *The Responsive Eye* (1965) ([SciAm blog](https://www.scientificamerican.com/blog/sa-visual/art-and-science-of-the-moire/)).
- **Philippe Decrauzat**, *On Cover* series (from 2011): paintings inspired by the May 1963 SciAm moire cover ([SciAm blog](https://www.scientificamerican.com/blog/sa-visual/art-and-science-of-the-moire/)).
- **Roger Penrose**, Royal Institution demonstration: patterns on acetate, rotated and shifted so the moire spacing visibly changes ([Rosetta mirror](https://rosetta.to/u/theroyalinstitution/incredible-moir-patterns)).
- **David C. Roy**, kinetic sculptures *Shimmer* and *Illusion*: moire made with rotating wheels ([Wood That Works](https://woodthatworks.com/blog/2011/4/14/moire-patterns-in-kinetic-design.html)).
- **Michael Bach**, interactive ring-on-ring demo ([michaelbach.de](https://michaelbach.de/ot/lum-moire1/)).
- **Moire Explore**, an interactive tool with stacked grids, blend modes and wave distortion ([Moire Explore](https://nateroe.github.io/moire/index.html)).

## Visual language

Picture the screen filled edge to edge with very fine, even grey-white lines on near black. Up close they read as a flat grey haze. Over it, wide soft bands of light and dark swim through the frame: fans, curved ribs, ghost circles and rosettes.

- **Lines** give swimming, breathing bands and curved fans.
- **Concentric rings** against rings give curved dark bands that drift. Bach's demo moves the rings at slightly different X and Y speeds, so the cycle takes a long time to repeat ([Michael Bach](https://michaelbach.de/ot/lum-moire1/)).
- **Dot and hexagonal grids** give rosette and lattice textures ([Moire Explore](https://nateroe.github.io/moire/index.html)).
- **Blend modes** (Multiply, Screen, Difference, XOR) decide the color and character of the overlap.

Motion is the point. Slow drift is hypnotic. A small input causes a global ripple. On a kick the bands snap tighter and relax. Color stays at band scale: a cold cyan-to-navy ramp over a pale, neutral carrier. The mood is calm, exact and slightly dizzying.

## See it

- [Incredible Moire Patterns! (YouTube, The Royal Institution)](https://www.youtube.com/watch?v=cvWF_Q5-Kt8) — Roger Penrose rotates and shifts overlaid patterns.
- [Moire Explore](https://nateroe.github.io/moire/index.html) — interactive tool: 2–8 rectangular or hexagonal grid layers, blend modes, wave distortion, keyframe animation.
- [Michael Bach, Moire Patterns](https://michaelbach.de/ot/lum-moire1/) — interactive concentric-ring moire you can drag and recolor.
- [SciAm: Art and Science of the Moire](https://www.scientificamerican.com/blog/sa-visual/art-and-science-of-the-moire/) — the 1963 cover, Oster's art and Decrauzat's paintings.
- [Wood That Works, Moire in Kinetic Design](https://woodthatworks.com/blog/2011/4/14/moire-patterns-in-kinetic-design.html) — kinetic moire sculptures by David C. Roy.

## Code design

The design rule: **all moire is computed, none is sampled.** Every grating is band-limited to the pixel grid it is drawn on. The only interference the audience sees is the one between our own layers, not between our layers and the LED pixels, the LED processor's scaler or a camera.

Everything lives in one single-pass fragment shader. Each layer is a phase field: work in centered output pixels, rotate, scale and offset per layer, then take `0.5 + 0.5 cos(2π φ)`. Use sine, not hard bars, because a square wave has harmonics that alias first. Layer B is layer A twisted by a small angle, detuned in pitch and optionally warped by 2–4 sine waves. The layers are combined like transparencies: Multiply (default, dark bands on a lit field), Difference or XOR. An optional layer C of rings from a moving centre adds rosettes.

Spacing is predictable. For a twist α between equal gratings, band spacing is `D = p / (2 sin(α/2))`, about `p/α` ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)). With p = 10 px, 1.5° gives about 382 px and 1.8° about 318 px. That is the kick lever. A pitch difference (p vs p + δp) gives bands every `p(p + δp)/δp`, about 343 px for δp = 0.3 px. That is the bass lever.

The shader also computes the band field directly, `0.5 + 0.5 cos(2π (φA − φB))`. It has no fine detail and never aliases. A `carrier` parameter crossfades between the full fine lines and this smooth band field. Anti-aliasing follows the filtered-cosine idea from [Inigo Quilez: band limiting](https://iquilezles.org/articles/bandlimiting/); for hard-edged lines or dots, see [Inigo Quilez: filterable procedurals](https://iquilezles.org/articles/filterableprocedurals/).

=== "Resolume Wire"
    Resolume documents ISF only inside Wire ([Resolume ISF support](https://resolume.com/support/en/isf)). Build the shader as a Wire patch with one ISF node and export it to Arena as a source. Wire supports ISF as of version 7.22 and no vertex shaders; this shader is fragment-only, so it fits.

    - Inputs on the ISF node: twist, detune, period, thickness, carrier, warp, warpPhase, mode, combine, ringCentre, palette colors.
    - Beat phase through a cosine ease drives the twist envelope. `Audio Analysis` low band into an `Envelope` (attack 100 ms, release 400 ms) adds the kick. Bass level, smoothed, drives detune. Mid level drives warp.
    - A bar `Counter` (wrap 16) picks mode, combine and accent color per phrase.
    - Keep the ISF single-pass. Do the optional ghost feedback with Wire's own feedback node on the band field only.
    - Output at the composition resolution. In Arena, leave the moire layer's Scale at 100 %, Rotation at 0 and Position at 0: Arena transforms resample the image, and a resampled fine grating is an accidental moire.
    - Without Wire: two layers of a pre-rendered fine-line loop at exactly 1:1 pixels, the top one in Multiply and rotated a fraction of a degree. It works, but the rotation resamples. Use periods of 8 px or more and expect some crawl.

=== "Unreal Engine 5.7"
    - A post-process material (Blendable Location: After Tonemapping) or an unlit material on a full-screen plane. Pixel coordinates come from ScreenPosition × ViewSize, rotations from Sin/Cos, gratings from `Cos`, and a Custom node holds the band-limit factor `sin(πw)/(πw)` with `w = 1/period`.
    - A Material Parameter Collection `MPC_Moire` holds Twist, Detune, Period, Carrier, Warp, Mode, Combine, RingCentre, Accent, driven from Blueprint (beat clock, envelopes).
    - Optional Niagara system with 1–3 particles follows MIDI notes or LFOs and writes ring centers to the MPC each tick. It is only a control source; nothing is rendered.
    - Post-process volume: Anti-aliasing off (or TSR disabled for the view), because temporal AA jitters sub-pixel lines and smears the carrier. Screen Percentage 100, motion blur off, manual exposure, bloom off or very low.
    - Remote Control preset on the MPC scalars, with OSC and MIDI bindings. Render at the LED processor's native resolution. Output via Spout (third-party plugin) or NDI.

=== "GODOT"
    - A `ColorRect` filling the viewport with a `canvas_item` shader. `FRAGCOORD.xy` gives pixel coordinates. Uniforms: `twist_deg`, `detune`, `period_px`, `carrier`, `warp`, `mode`, `combine`, `ring_centre`, `accent`, `beat_phase`, each with `hint_range`. Band-limit with `w = 1.0 / period_px`, or `fwidth(phase)` when warped.
    - Viewport size equals wall resolution, stretch mode `disabled` (or `canvas_items` with integer scale only). MSAA and FXAA off: they do nothing for a shader-drawn pattern, and FXAA blurs the carrier unevenly.
    - Ghost feedback: a half-size `SubViewport` outputting only the band field, clear mode never, with a black `ColorRect` at alpha 0.30 drawn first each frame (decay about 0.70).
    - `InputEventMIDI` for knobs and pads. `AudioEffectSpectrumAnalyzer` on the bus, low and mid bands read per frame and smoothed in script.

## On stage

!!! warning "LED walls, projectors and cameras: the rules that make it work"
    A fine grating collides with the LED grid, the processor's scaler, the camera sensor and the encoder. Unplanned moire is what you get when these rules are broken.

    1. **Render at the native pixel map, 1:1.** No render-then-scale. A 1920 × 1080 render scaled onto a 1728 × 972 wall resamples every fine line and adds a second moire against the scaler grid. In Arena, map the output slice 1:1; switch scaling off on the LED processor.
    2. **Minimum period 4 output px** for any single grating. Below that, the Multiply sum term (about twice the line frequency) aliases into slow fake bands. Where scaling is unavoidable, use 8 px or more, measured after scaling.
    3. **Cameras live (broadcast, recording, streaming):** carrier at 0.3 or lower, or period 8 px or more. Keep lines off exactly 0° and 90°: rotate the common frame by 3 to 7° so they do not align with the LED rows. Do a test shot through the actual camera and lens at the planned framing. Photographing screens is a classic source of unwanted moire ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)).
    4. **Keep absolute rotation slow.** A pixel at radius r moves `r · Δθ` per frame; if that nears half a period, the lines strobe or run backwards. For a 3840 px wall and p = 10 px, that is about 3.9°/s at 60 fps. Hold common rotation to 0–3°/s. Relative twist can move fast, because band motion is not limited by the carrier. Never step twist in a single frame: ease kicks over 80–120 ms.
    5. **Keep fine lines neutral in color.** Colored fine lines add chroma aliasing on LED subpixels and on cameras. Color lives at band scale only.
    6. **Leave out of the output path:** TAA/TSR/FXAA, sharpen, grain above 2 %, chromatic aberration on the carrier, keystone and scaling. Projector keystone resamples too: do it only with a low carrier, or accept a larger period.
    7. **Coarse walls (P3.9 and up, seen close):** a 4 px period reads as stripes, not haze. Use 6–10 px so the bands still dominate.

**Audio reactivity.** Because band spacing goes as p/α, depth must be small. A full kick is a fraction of a degree. Too deep a mapping becomes constant full-screen flicker.

| band | drives | smoothing |
|------|--------|-----------|
| Kick, 40–100 Hz onset | twist impulse +0.3° | attack 100 ms, release 400 ms |
| Bass, 60–250 Hz level | detune 0–6 %, ramp midpoint 0.5 to 0.35 | attack 50 ms, release 500 ms |
| Mid, 300 Hz–2 kHz | warp 0.05–0.6 | attack 150 ms, release 1 s |
| Hi-hat, 6–12 kHz onset | B phase jump of exactly half a period (bands invert), or XOR for a 1/16 note; max one per 1/8 note | instant, hold 1/16 |
| Loudness | carrier 0.4 to 1.0 | attack 300 ms, release 2 s |

**MIDI (KeyStep 37 / BeatStep Pro).**

| control | parameter |
|---------|-----------|
| KeyStep knob, fine steps (0.02°) | twist |
| Fader | detune, period, carrier |
| Knob | warp |
| BeatStep Pro pads | mode and combine, quantized to the bar |
| KeyStep notes (up to 3) | ring centers in the break; held chords build rosettes |

**Arrangement.**

- **Every bar:** twist eases from 0 on the downbeat to its target by beat 2 and back toward 0. With twist and detune both 0 the gratings coincide, the bands open to infinity and the screen becomes an even field. The pattern blooms out of flat and collapses on every bar. Run grating B's phase at 2 cycles per bar and grating C's at 3, so the pattern realigns each bar without fast rotation.
- **Every beat:** B's phase steps by a quarter period, eased over 60 ms. Bands walk across the screen.
- **Phrase (16 bars):** switch mode (lines, rings, dots, hex) and accent color (cyan, magenta, amber) at the resolve, when the field is flattest and the cut is invisible.
- **Build (8 bars):** period shrinks from 14 px to the floor, warp rises 0 to 1, carrier rises to 1. Dense, nervous, shimmering.
- **Drop:** hard cut to rings, Difference, period 12 px, twist 3°, ring center onto a new LFO path, 80 ms flash on the band field.
- **Break:** carrier to 0.2, soft bands only, slow detune drift, rotation stopped.

**Layering.** Send the band field (carrier 0) as a luma key or Multiply over video. The video gets swimming dark bands without fine lines, so it stays safe on camera. Multiplying it over other material gives the interference for free. On LED walls keep black at #050506 and peaks at about 90 %; a 50 % grey haze from a fine carrier pulls a lot of power and looks flat, so lower the carrier on breaks.

**Pitfalls.** Static, evenly paced rotation; pure black and white for a whole set; no depth. Bloom over a full-brightness carrier washes everything to grey, so bloom the band field only.

## Variations

1. **Wave-warped grids.** Distort one grid with a few sine waves so bands become organic ([Moire Explore](https://nateroe.github.io/moire/index.html)).
2. **Video as one layer.** Let camera or clip luminance modulate line width in one layer, so the image emerges through the moire.
3. **Audio spectrum as grating.** Draw one layer's lines from FFT bands, so the beat of the music is literally the moire.
4. **Rings from tracked points.** Ring centers follow MIDI notes or LFOs, and chords make interference rosettes.
5. **Three-layer color split.** Put R, G and B on separately rotated grids (twist offsets of about −0.15°, 0°, +0.15° against a shared A) for chromatic moire. The bands split into rainbow fringes while the carrier stays roughly white.
6. **Physical version.** Acetate overlays in front of the projector, as in Penrose's demo.

## Did you know?

- Moire shares its math with acoustic beats ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)).
- Interlaced TV and cameras filming screens create unwanted moire, and banknotes use fine patterns that moire when scanned ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)).
- Twisted bilayer graphene forms a moire pattern, and at a "magic angle" it shows superconductivity ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)).
- Moire fringes magnify displacement, which is why they are used to measure strain ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern), [SciAm blog](https://www.scientificamerican.com/blog/sa-visual/art-and-science-of-the-moire/)).

## Sources

- [Wikipedia: Moire pattern](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)
- [Scientific American blog: Art and Science of the Moire](https://www.scientificamerican.com/blog/sa-visual/art-and-science-of-the-moire/)
- [Michael Bach: Moire Patterns](https://michaelbach.de/ot/lum-moire1/)
- [Wood That Works](https://woodthatworks.com/blog/2011/4/14/moire-patterns-in-kinetic-design.html)
- [Rosetta mirror of Royal Institution video](https://rosetta.to/u/theroyalinstitution/incredible-moir-patterns)
- [Moire Explore](https://nateroe.github.io/moire/index.html)
- [realtimevfx: Interference patterns for motion](https://realtimevfx.com/t/interference-patterns-for-motion/2671)
- [Resolume ISF support](https://resolume.com/support/en/isf)
- [Inigo Quilez: band limiting](https://iquilezles.org/articles/bandlimiting/)
- [Inigo Quilez: filterable procedurals](https://iquilezles.org/articles/filterableprocedurals/)
