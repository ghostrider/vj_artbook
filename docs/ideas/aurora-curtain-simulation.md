---
tags:
  - nature
  - light
  - atmosphere
  - shader
---

# Aurora Curtain Simulation

> A slow green sheet of light with a razor hem, a violet flash on every kick and a red crown that remembers the whole build.

![Green aurora curtain and beams over Nordmo, Bjerkvik, Norway (portrait photo)](https://upload.wikimedia.org/wikipedia/commons/thumb/f/fb/Northern_lights_curtain_over_Nordmo_in_Bjerkvik%2C_Narvik%2C_Nordland%2C_Norway%2C_2023_September.jpg/1280px-Northern_lights_curtain_over_Nordmo_in_Bjerkvik%2C_Narvik%2C_Nordland%2C_Norway%2C_2023_September.jpg)
*Green aurora curtain and beams over Nordmo, Bjerkvik, Norway. Image: [Ximonic (Simo Räsänen)](https://commons.wikimedia.org/wiki/File:Northern_lights_curtain_over_Nordmo_in_Bjerkvik,_Narvik,_Nordland,_Norway,_2023_September.jpg), [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0).*

## The idea

An aurora is a tall curtain of light in the upper atmosphere. Electrons and protons from the solar wind are guided by Earth's magnetic field into the air. They excite atoms and molecules, which give the energy back as light ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)). The light does not travel. Atoms stay put. Shifting field conditions send particles elsewhere, so the pattern seems to flow ([Discover](https://www.discovermagazine.com/real-time-footage-of-aurora-shows-them-dancing-and-shimmering-13807)).

On a club screen this is a vast, quiet backdrop. The image is mostly black, so it layers additively under silhouettes, type or line work. The danger is a plain slow green wash. The cure is structure: rays, folds, a palette shift or a storm event.

This chapter goes one step further than "noise with a green gradient". It simulates the emission physics. Each gas has its own colour, altitude and delay. Those three facts become three layers, three colours and three decay times. The music then excites the sky the way the sun does: fast things flash, slow things build.

## Origins and references

- Pierre Gassendi's 1649 account of a 1621 display gave the aurora borealis its name, a term Galileo had used in 1619 ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)).
- Kristian Birkeland shot an electron beam at a magnetised sphere, a "terrella", and got rings of light at the poles. He proposed field-aligned currents in 1908. Satellite data confirmed them in 1967 ([Wikipedia: Kristian Birkeland](https://en.wikipedia.org/wiki/Kristian_Birkeland)).
- Carl Størmer triangulated more than 12,000 auroras from 1911. None was below 70 km, the height maximum was near 100 km ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)).
- Lawlor and Genetti, GPU volume rendering of aurora in 2011: a 2D electron-flux footprint times a height-dependent deposition function ([PDF](https://www.cs.uaf.edu/~olawlor/papers/2010/aurora/lawlor_aurora_2010.pdf)).
- Project Aurora by Ginny Ruffner: 34,560 LEDs, 20 ft by 10 ft, refreshed 20 times per second ([Nordic Museum](https://nordicmuseum.org/exhibitions/project-aurora)).
- Mytherrella by Dorin Cucicov, ISEA2023: a StyleGAN trained on all-sky aurora images, shaped by visitors ([ISEA archive](https://www.isea-symposium-archives.org/?p=75676)).
- Night Curtains: a browser simulator with a storm level from KP 1 to 9 ([Night Curtains](https://arcade.pirillo.com/night-curtains.html)).
- Gaia Sky aurora renderer: a volume ray-caster that stretches hand-drawn footprints vertically ([Toni Sagristà](https://tonisagrista.com/blog/2024/rendering-aurorae-nebulae/)).

## Visual language

Tall vertical curtains. A bright, sharp lower edge and a soft fade upward. Folds and rays run along the vertical. The colour goes by height: violet or pink hem, green body, red crown. The ground is almost black: a silhouette horizon and still stars.

Real-time footage shimmers and dances, and the stars hold still ([Discover](https://www.discovermagazine.com/real-time-footage-of-aurora-shows-them-dancing-and-shimmering-13807)). Most aurora video is time-lapse, so a calm real-time look feels fresh. Hem waves take 20 to 120 seconds per cycle. Never sync them to the beat. The slowness is the scale reference.

The sharp hem exists because atomic oxygen becomes scarce below about 100 km ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)). Where a fold is seen edge-on, it brightens into a hot seam.

Pulsating auroras vary on roughly 2 to 20 second periods. A quiet arc can turn into an active display within minutes, a substorm ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)). That is the drop.

### Physics to design

| Gas and line | Altitude | Delay (emission lifetime) | Layer in the patch | Colour | Decay in the patch |
|---|---|---|---|---|---|
| Molecular nitrogen, about 428 nm | lowest: the hem | much faster than oxygen | N2 channel, hem band | violet-blue `#5B3BFF` (pink variant `#FF5FB0`) | none: flashes and stops in the same frame |
| Atomic oxygen, 557.7 nm | middle: the body | about 0.7 s | green channel, the rays | green `#3DFF8F` | fast release, tau 0.7 s x `lifetime` |
| Atomic oxygen, 630.0 nm | highest: the crown | up to about two minutes | red channel, diffuse | red `#E8263A` | slow integrator, tau 4 to 30 s (default 12 s) |

Source for gases, wavelengths, lifetimes and order: [Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora). Sources disagree on the green altitude range (compare [Night Curtains](https://arcade.pirillo.com/night-curtains.html)), so the band edges are a stage choice, not a measurement. The real two minutes for red is too slow for a set, so `lifetime` scales it down.

## See it

- [Dance of the Spirits: Incredible Real Time Northern Lights from Jan 24 Solar Storm](https://www.youtube.com/watch?v=SkiNUMKKsgM) — real-time solar-storm display, shimmering curtains in motion (YouTube, channel ingenioustv by Alister Chapman).
- [Friday the 13th Aurora](https://vimeo.com/238450470) — aurora film by Alan Dyer (Vimeo).
- [Project Aurora, National Nordic Museum](https://nordicmuseum.org/exhibitions/project-aurora) — LED wall of moving aurora sheets.
- [Night Curtains](https://arcade.pirillo.com/night-curtains.html) — live procedural aurora simulator with storm level and palette controls.
- [Mytherrella, ISEA2023](https://www.isea-symposium-archives.org/?p=75676) — interactive generative aurora installation.

## Code design

The patch is a 2D curtain built from a footprint curve, in four passes. P0 computes per-column curtain data (hem line, fold coordinate, edge-on gain) for up to three curtains. P1 computes the excitation rate per species: nitrogen, green oxygen, red oxygen. P2 is a persistent float buffer that gives each species its own delay. P3 turns species into colour, adds sky and stars, sharpens the hem, applies a hue-preserving tonemap and dithers. Bloom comes after, in the host.

The hem line is a sum of two slow sines. Altitude `a` is measured from the hem. A vertical deposition profile falls off quickly below the hem and fades upward, the same split as Lawlor and Genetti's footprint times deposition ([PDF](https://www.cs.uaf.edu/~olawlor/papers/2010/aurora/lawlor_aurora_2010.pdf)). The fold coordinate `s` bends the sheet. When fold strength passes 1, the curtain folds back over itself, and an edge-on gain lights the fold as a seam. Rays are anisotropic fbm noise (about 50:1) along `s`, sharpened by a power, each with its own top.

The colour is a fixed vector per species, weighted by emission. It is not a gradient ramp. Palette swaps and band reversal are one remap.

Do not smooth twice. The emission delays in P2 are the smoothing for the light. Keep the audio envelopes fast and let green and red do their own slow work.

=== "Resolume Wire"
    - Use one multipass ISF node: P0 `cols` (FLOAT, height 3), P1 `excite` (half size), P2 `glow` (PERSISTENT, FLOAT), P3 to the output ([ISF multipass reference](https://docs.isf.video/ref_multipass.html)). TIMEDELTA is a built-in ISF uniform ([ISF variables](https://docs.isf.video/ref_variables.html)).
    - Inputs as floats with MIN/MAX: storm, energy, rays, height, lifetime, crown, plus `kick`, `hat`, `palette`, `edgeLock` and a `clear` event ([Resolume ISF](https://resolume.com/support/en/isf)).
    - Check your version: the Resolume ISF page does not cover multipass or persistent buffers. Test first that `glow` keeps its state across frames. Also check what your Wire version calls the Time and clock nodes.
    - Fallback if persistence fails: two single-pass ISF nodes, "Green" and "Red", each in its own Wire feedback loop with mix 0.976 (green) and 0.998 (red). A third node colourises. If the loop is 8 bit, subtract a 1/255 floor per frame on the red.
    - Wire has no vertex shaders, which does not matter here. The volumetric route is not worth it in Wire.
    - In Arena: the Wire patch goes on an Add layer over a sky layer. Silhouette PNG with alpha on top in Normal. Bloom at threshold about 0.6, intensity about 1.2. Keep the envelopes in Wire, not in Arena's analysis.
    - Without a shader: real-time aurora footage on an Add layer, levels pushed to crush noise, a blurred duplicate at 30 % for glow. You lose the species timing.

=== "Unreal Engine 5.7"
    - Route A, 2D (same as ISF): two RGBA16f half-resolution render targets plus an excite target. `Draw Material to Render Target` runs excite, then afterglow, swapping the two glow targets each frame. A final unlit material colourises. Parameters sit in a Material Parameter Collection.
    - Route B, ribbons (cheap parallax): one Niagara emitter per curtain, 300 to 600 particles along x, a Ribbon Renderer with width equal to curtain height and the ribbon standing vertical. Check the facing mode name in 5.7. The material is unlit, additive, two-sided, and UV.v reads the species weights from a 256 x 1 lookup. Delays come from Blueprint envelopes, not per pixel.
    - Route C, volumetric: an inverted box with a Custom node ray-marching about 48 steps with per-pixel jitter. March at reduced resolution and let TSR clean up the jitter.
    - Control: a Remote Control Preset exposing the MPC scalars and Niagara user parameters, bound to OSC from the audio envelopes and to MIDI from the BeatStep Pro. Output to Arena through Spout or NDI.

=== "GODOT"
    - 2D route (Godot 4): a half-size `SubViewport` with HDR 2D holds a `ColorRect` with the P0 + P1 shader. Two more half-size SubViewports, GlowA and GlowB, run P2 and swap each frame from a script. A full-screen `ColorRect` runs P3, followed by `WorldEnvironment` glow for bloom.
    - Uniforms: storm, energy, rays, height, lifetime, crown, kick, hat, three palette colours (`source_color`), edge_lock, hem_softness. Pass delta from `_process(delta)`.
    - Volumetric route: start from the CC0 aurora sky shader with 30 jittered steps ([Godot Shaders](https://godotshaders.com/shader/aurora-sky-shader/)). Replace its gradient with the three-species lookup and drive brightness from the kick and red envelopes in GDScript.
    - Audio: `AudioEffectSpectrumAnalyzer` on the music bus, with attack and release envelopes in script. Output through Spout or NDI, or capture the window in Arena.

## On stage

**Audio reactivity**

| band | drives | smoothing |
|---|---|---|
| 40 to 120 Hz onset (kick) | excitation pulse on the near curtain: violet hem flash, green with its 0.7 s tail | attack 2 ms, release 120 ms; trigger above 1.4 x the 1 s mean |
| 40 to 160 Hz RMS | `energy` (hem depth, violet width) and `height` by +-0.1 | attack 80 ms, release 400 ms |
| 300 Hz to 2 kHz RMS | sustained input to the red integrator (crown) | attack 1 s, release 4 s, then tau in P2 |
| 6 to 12 kHz onset (hats) | per-ray shimmer, +-15 % brightness noise | attack 1 ms, release 50 ms |
| BPM | pulsation period 8 or 16 beats, ray flip every 32 beats, drop ramp of 1 bar | none |

One kick barely shows in red, because red integrates. A long build turns the crown red, and it lingers through the break.

**Controls (BeatStep Pro, KeyStep 37)**

| parameter | range | default | mapping |
|---|---|---|---|
| storm | 0 to 1 | 0.2 | BeatStep Pro knob 1; drop pad ramps to 1 over 1 bar |
| energy | 0 to 1 | 0.3 | bass RMS, smoothed; knob 2 as offset |
| rays | 6 to 120 per frame height | 40 | knob 3 |
| height | 0.3 to 0.75 | 0.5 | bass level +-0.1; knob 4 |
| lifetime | 0.5 to 2 | 1 | knob 5; double it in breaks |
| crown | 0 to 2 | 1 | KeyStep mod strip; slow fade-ins only |

**Arrangement**

- Drop: storm 0 to 1 over one bar with a cubic ease-out, plus a one-shot `energy` of +0.3. Mid and far curtains fade in over 2 beats. Curtains multiply, fold over, rays sharpen and race along the arc, and the hem drops and turns violet.
- Break: storm falls to 0.1 over 2 bars. Rays give way to dim patches pulsing every 8 or 16 beats (3.75 s or 7.5 s at 128 BPM). The red crown from the build lingers.
- Trigger the drop by hand, not automatically. A loud breakdown would cause false triggers.
- Crossfade palettes over 4 to 8 bars, never on the beat.
- Speed x 4 gives the familiar time-lapse sweep. Use it only as a transition.
- Layer under silhouettes, type or line work. The aurora is mostly black, so an Add layer on top always reads.

**LED walls**

- Green LEDs are efficient, so a full green field is harsh. Cap green at about 80 % and let bloom add glow. Keep about 80 % of the frame below 5 % luminance.
- Keep each ray at least 3 LED pixels wide: `rays <= wall_height_px / 6`. Favour fewer, wider rays on low-resolution walls.
- Define ray density per frame height, so wide strips get more rays, not stretched ones.
- Slow green fades on dark backgrounds band badly on 8-bit processors. Keep the dither on and test the black level, because some processors crush the first 2 to 3 % to black.

**Performance**

- Float buffers are required for red. With a decay factor of 0.9986, an 8-bit value of 10/255 rounds back to 10/255, so the ghost never fades. Use FLOAT or 16f buffers. In an 8-bit fallback, subtract a 1/255 floor per frame, or update the red only every 8th frame.
- Always scale decay by TIMEDELTA, so a frame drop on stage does not change the timing.
- The 2D route is cheap. The main cost is P1 (three curtains, 3-octave fbm), so turning off mid and far curtains at low storm saves about 60 % of it. These are estimates; measure on your machine.
- The volumetric route is only worth it if the camera moves through or under the aurora. Lawlor and Genetti measured 60 fps at 720p on a GTX 280 with distance-field stepping, against +350 % cost for naive stepping ([PDF](https://www.cs.uaf.edu/~olawlor/papers/2010/aurora/lawlor_aurora_2010.pdf)). For a fixed frontal LED view, the 2D route looks the same.

## Variations

- Non-physical palettes in the same order: "solar" with hem `#29E0FF`, body `#FF2BD6`, crown `#FFB52E`. "Ice" with hem `#FFFFFF`, body `#59C8FF`, crown `#3B4BFF`.
- Reverse the stack, or let the three colour bands drift out of sync.
- Curtains over a city or architecture silhouette, via a mask from another layer.
- Terrella view: a glowing ring around a magnetised sphere, after Birkeland ([Wikipedia: Kristian Birkeland](https://en.wikipedia.org/wiki/Kristian_Birkeland)).
- Pixel-LED version with sheets on a low-res grid, as in Project Aurora ([Nordic Museum](https://nordicmuseum.org/exhibitions/project-aurora)).
- Physical delay mode for green: softer and more correct, but kicks look weaker than in the default stage mode.

## Did you know?

- Red oxygen light takes up to about two minutes to be emitted, green about 0.7 s ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)).
- Størmer's survey of more than 12,000 auroras found none below 70 km ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)).
- Birkeland's theory was dismissed in his lifetime and confirmed by satellite data in 1967 ([Wikipedia: Kristian Birkeland](https://en.wikipedia.org/wiki/Kristian_Birkeland)).
- Most aurora videos are time-lapse, because real movement is usually slow. Real-time clips look very different ([Discover](https://www.discovermagazine.com/real-time-footage-of-aurora-shows-them-dancing-and-shimmering-13807)).

## Sources

- [Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)
- [Wikipedia: Kristian Birkeland](https://en.wikipedia.org/wiki/Kristian_Birkeland)
- [Toni Sagristà: Rendering aurorae and nebulae](https://tonisagrista.com/blog/2024/rendering-aurorae-nebulae/)
- [National Nordic Museum: Project Aurora](https://nordicmuseum.org/exhibitions/project-aurora)
- [ISEA archive: Mytherrella](https://www.isea-symposium-archives.org/?p=75676)
- [Night Curtains](https://arcade.pirillo.com/night-curtains.html)
- [Discover Magazine](https://www.discovermagazine.com/real-time-footage-of-aurora-shows-them-dancing-and-shimmering-13807)
- [Lawlor & Genetti, Interactive Volume Rendering Aurora on the GPU (2011)](https://www.cs.uaf.edu/~olawlor/papers/2010/aurora/lawlor_aurora_2010.pdf)
- [Godot Shaders: Aurora sky shader](https://godotshaders.com/shader/aurora-sky-shader/)
- [ISF Multi-Pass and Persistent Buffer Reference](https://docs.isf.video/ref_multipass.html)
- [ISF variables](https://docs.isf.video/ref_variables.html)
- [Resolume ISF](https://resolume.com/support/en/isf)
- [Commons file page](https://commons.wikimedia.org/wiki/File:Northern_lights_curtain_over_Nordmo_in_Bjerkvik,_Narvik,_Nordland,_Norway,_2023_September.jpg)
