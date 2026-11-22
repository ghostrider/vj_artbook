---
tags:
  - simulation
  - artificial-life
  - cellular-automata
  - shader
  - organic
---

# Lenia Continuous Artificial Life

> Soft translucent creatures glide across dark water, and every kick drops a new handful of life into it.

## The idea

Lenia is a family of cellular automata made by Bert Wang-Chak Chan, an independent researcher in Hong Kong. It generalises Conway's Game of Life to continuous space, time and state. Life itself is a special case of discrete Lenia ([Wikipedia](https://en.wikipedia.org/wiki/Lenia)). The main paper reports more than 400 species in 18 families, many found by interactive evolutionary search ([arXiv 1812.05433](https://arxiv.org/abs/1812.05433)).

The rule is short. Each pixel sums its neighbours through a ring-shaped kernel. A bell-shaped growth function then decides whether the pixel grows or decays. The result is added to the state in small steps ([arXiv 1812.05433](https://ar5iv.labs.arxiv.org/html/1812.05433)). From that, discs with fuzzy rims and ring-like insides appear. They glide, rotate, pulse, split and bounce off each other.

On a club screen it reads like microscope footage of plankton. The motion is slow, breathing and biological, not on the grid. Beats do not flash. They show up as births and lurches. The state is a single grey channel, so colour is entirely your choice.

## Origins and references

- **Lenia (2018 onward), Bert Chan.** The original system, with a browser version and Python, MATLAB and R/Jupyter code under the MIT licence ([GitHub](https://github.com/Chakazul/Lenia)). Named species include Orbium (a disc-shaped glider), Scutium (a shield-shaped, omnidirectional mover), Paraptera, Gyrorbium and Hexadentium ([arXiv 1812.05433](https://arxiv.org/abs/1812.05433)). A MATLAB v2.0 dates from 2017 on the author's timeline ([Lenia portal](https://chakazul.github.io/lenia.html)).
- **Recognition.** Winner of the 2018 Virtual Creatures Contest at GECCO (Kyoto), honorable mention in the ALIFE Art Award 2018 (Tokyo), and ISAL Outstanding Publication of 2019 ([Wikipedia](https://en.wikipedia.org/wiki/Lenia)).
- **Lenia: Expanded Universe (2020).** Multi-channel and multi-kernel worlds with self-replication, emission, growth by ingestion and "virtual eukaryotes" with cell-type differentiation ([arXiv 2005.03742](https://arxiv.org/abs/2005.03742)).
- **Flow-Lenia (2022/23).** Plantec, Hamon, Etcheverry, Oudeyer, Moulin-Frier and Chan add mass conservation and localised parameters, so many different creatures coexist and mix in one world ([arXiv 2212.07906](https://arxiv.org/abs/2212.07906)).
- **Art uses.** Chan's portal lists a VRChat "Lenia Room", the film "Thoughts on Life", a virtual creature module for Artbreeder, and Interspecifics' "Codex Virtualis" shown at Ars Electronica ([Lenia portal](https://chakazul.github.io/lenia.html)). The Interspecifics project pages do not name Lenia, so treat that link as claimed by Chan only ([Interspecifics](https://interspecifics.cc/work/category/bacteria/)).

## Visual language

A dark-field microscope slide fills the wall. Near-black water, and in it soft, translucent discs with fuzzy rims and faint concentric rings. The main species is Orbium. Each creature shows its heading: the growing front edge glows amber, the decaying tail fades to magenta. Faint ghost trails mark where it has been.

Motion is gliding, swelling, splitting. Kicks drop clusters of "primordial soup" that either pull together into creatures or dissolve within a second or two. Bass makes the population fat or lean.

Colour comes from a four-stop ramp on the state value. The default palette is dark-field teal: body #127C7A, inner rings #63D6B4, core #EAF8EE, front #FF9F43, tail #B83A78, background #02070B. On the drop it crossfades to bioluminescent violet (body #7B3FE4, rings #C9A2FF). Rhythm is slow and organic. Keep the highlight core below 92 % brightness for LED walls, and let the contrast come from dark water against glowing bodies.

## See it

- [Lenia web demo](https://chakazul.github.io/Lenia/JavaScript/Lenia.html) — interactive browser simulation with species list (Orbium, Scutium), brush and GIF export.
- [Lenia - Mathematical Life Forms](https://youtu.be/iE46jKYcI4Y) — the author's video of Lenia creatures in motion.
- [Lenia: Expanded Universe 1080p](https://youtu.be/HT49wpyux-k) — companion video for multi-channel, 3D and self-replication results.
- [Lenia portal](https://chakazul.github.io/lenia.html) — hub with demos, videos, Shadertoy and VRChat links.
- [Lenia GitHub repo](https://github.com/Chakazul/Lenia) — code and screencap folder.

## Code design

The sim is a ping-pong float buffer at 640×360, upscaled and shaded at full resolution. Per step: potential `U = Σ K · A` over a normalised ring kernel, growth `G = 2·exp(−(U − mu)² / (2σ²)) − 1`, update `A' = clamp(A + dt·G, 0, 1)` ([arXiv 1812.05433](https://ar5iv.labs.arxiv.org/html/1812.05433)). The default Orbium genes are R = 13, dt = 0.1, mu = 0.15, sigma = 0.015, taken from the [CAX Lenia example](https://maxencefaldor.github.io/cax/examples/20_lenia/). Other species need their own values from Chan's code ([GitHub](https://github.com/Chakazul/Lenia)).

The pass order per frame: kernel bake (only on change), inject and N Lenia steps, trail, watchdog (every 30 frames), shade, post. The state buffer needs RGBA16F at minimum. At 8 bit the small `dt·G` steps round away and the sim dies.

Three supporting ideas keep it alive on stage:

- **Injection and watchdog.** A kick seeds a noise disc of radius 1.2·R, a pad stamps a known creature, and a brush paints soup. If mean state stays below 0.0005 for 2 s, the sim auto-seeds 8 discs. If it rises above 0.25, mu is raised briefly.
- **Speed.** Do not raise dt to go faster. Use 1 to 4 steps per frame. A kick may lift dt from 0.1 to 0.16 for 120 ms as a forward lurch.
- **Growth-aware shading.** The growth channel marks where a body grows (front) and decays (tail). Add a rim from the state gradient to bring out the inner rings, and a trail of `max(trail · 0.97, A)` under the body.

=== "Resolume Wire"
    Arena has no Lenia effect, so the options are an ISF multipass effect loaded in Wire, or an external sim over Spout.

    - **Route 1, ISF multipass.** Persistent FLOAT buffers `s0`, `s1` at a third of the output size, plus `trail` ([ISF multipass](https://docs.isf.video/ref_multipass.html)). Passes: step s0 to s1, step s1 to s0, trail, then the shade pass. Initialise on `FRAMEINDEX == 0` with 12 soup discs. GLSL loops need constant bounds, so hard-code R = 13 and compute the normaliser once per frame in a 1×1 pass.
    - **Precision check.** Resolume does not document whether `FLOAT` is honoured. Seed Orbium parameters and run for 30 s. If everything dies at once or never moves, the buffers are 8-bit. Use Route 2.
    - **Route 2, Spout.** Run the sim in UE or Godot and receive it in Wire. Keep only the shading (palette, glow) in a single-pass ISF node.
    - **Node outline.** Input (food clip) → Blur → Resize to sim size → ISF node → Output.
    - **Control.** Beat node → envelope (instant attack, 120 ms release) → `dt` lurch and `seed`. A beat counter hashed in the shader gives `seedPos`. An LFO at one cycle per 16 bars, ±0.004, feeds `mu`. Exposed: life, growth, tolerance, seed, glow, paletteMix, reset. Node names vary between Wire versions.

=== "Unreal Engine 5.7"
    - **Route A, Blueprint ping-pong (recommended).** `RT_S0` and `RT_S1`: 640×360, RGBA16f, Wrap, Nearest, no mips. `RT_Trail` R16f, `RT_Kernel` 31×31 R32f, `RT_Out` 1920×1080.
    - `M_LeniaKernel` writes the ring weights. The normaliser is computed in Blueprint on change and passed as a scalar.
    - `M_LeniaStep` is one Custom node (HLSL) with the double loop over the kernel and state, then growth, food, injection and clamp. Output (A, G, U, 0).
    - Blueprint Tick loops `Steps` times, drawing into the other RT and swapping. Then trail, then `M_LeniaShade` into `RT_Out`.
    - **Route B, Niagara.** A Grid2D Collection with attributes A and G and a simulation stage with `Steps` iterations. Use it if you want particles in the same system: a GPU emitter samples `RT_S0` and spawns amber sparks where growth and state are both high.
    - **Output and control.** Spout or NDI from `RT_Out` into Arena. A Material Parameter Collection `MPC_Lenia` holds Mu, Sigma, Dt, Food, Seed, brush, Glow and PaletteMix, with Blueprint functions `Reset`, `Reseed` and `Stamp(index)`. Bind via Remote Control over OSC and MIDI: BeatStep Pro knobs 1 to 4, pads 1 to 8 as stamps, KeyStep strip to Glow.

=== "GODOT"
    - Godot 4.x with Forward+, since compute needs a RenderingDevice renderer ([Godot docs, compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)).
    - **Compute route (recommended).** Two RGBA16F storage images at 640×360 and one kernel storage buffer filled from GDScript on change. Workgroup 16×16. Each group loads a (16 + 2R)² tile into shared memory, then each thread runs its tap loop from there. This is the biggest GPU speed-up for Lenia. Dispatch `steps` times per frame, alternating bindings.
    - Show the result with `Texture2DRD` ([Godot docs, Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html)) on a full-screen ColorRect whose `canvas_item` shader does the shading.
    - **Fallback.** SubViewport ping-pong at 320×180 with `use_hdr_2d`, 1 step per frame, R = 8. Expect slower life.
    - Audio comes from AudioEffectSpectrumAnalyzer on the music bus. `steps` and `R` belong to the script.

## On stage

**Audio.** Move only mu with audio, slowly. Never drive sigma from audio. It is the most sensitive gene and fast changes wipe out the population.

| band | drives | smoothing |
|------|--------|-----------|
| kick, 40–120 Hz | soup seed plus dt lurch 0.1 to 0.16 | instant attack, 120 ms release, retrigger blocked 200 ms |
| bass, 60–250 Hz | mu offset 0 to −0.004, bodies swell | attack 150 ms, release 1.2 s |
| snare, 1–4 kHz | mirrored second seed | attack 5 ms, release 150 ms |
| hats, 6–12 kHz | tiny brush dab, radius 0.3·R | attack 5 ms, release 60 ms, max 2 per beat |
| RMS loudness | glow 0.3 to 1, bloom | attack 300 ms, release 3 s |

**KeyStep 37 / BeatStep Pro.**

| control | parameter | range |
|---------|-----------|-------|
| BeatStep knob 1 | life (steps per frame), quantised to the bar | 1–4 |
| BeatStep knob 2 | growth, mu offset | −0.01 to +0.01 |
| BeatStep knob 3 | tolerance, sigma offset, small moves only | −0.003 to +0.003 |
| BeatStep knob 4 | seed strength | 0–1 |
| BeatStep pads 1–8 | stamp a species | — |
| KeyStep knob | size R, breaks only (rebakes the kernel and resets the world) | 8–15 |
| KeyStep mod strip | glow | 0–1 |

Species live in narrow regions of parameter space. Outside them, creatures die or fill the screen, so clamp the MIDI ranges hard ([arXiv 1812.05433](https://arxiv.org/abs/1812.05433)).

**Structure.**

- Every 4 bars, stamp one known creature at an edge, heading inwards. Over 16 bars, let mu drift on a sine of ±0.004 so the population breathes.
- Drop: reseed 10 soup discs in a ring around the centre, run 4 steps per frame for 8 bars, crossfade palette A to B over 1 bar, add a beat of bloom.
- Break: 1 step per frame, sigma −0.001 so creatures shrink and some die, long trails, seeds off. Keep the watchdog on so the wall never goes empty.
- Change speed on bar lines only, eased by alternating step counts (2, 3, 2, 3).

**Layering.** Output as a grey mask or a gradient-mapped colour layer through Spout or NDI. Two ways over footage: Lenia as the luma mask of a clip, so the clip shows only inside the bodies, or Lenia in Screen at 60–80 % over dark footage. Avoid Add over bright footage. Post chain: black crush at 3 %, bloom, 1 px chromatic aberration, 3 % grain. No sharpening.

**Performance.** The cost is the convolution: about 530 non-zero taps per cell at R = 13, or about 245 M fetches per frame at 2 steps and 640×360. That is about 50 times a Gray-Scott step, so budget it first. Levers, in order: shared-memory tiling, lower sim resolution, smaller R, FFT convolution (used in Chan's Python version, [GitHub](https://github.com/Chakazul/Lenia), but hard to build in UE or ISF). Lock the output fps, because speed is counted in steps per frame. For wide strips keep cells square, for example 1280×240 for 5760×1080. Wrap addressing hides the seam and lets gliders circle a room.

**Pitfalls.** Changing R mid-set, raising dt for speed, an unnormalised kernel, and too much food all break it.

## Variations

- **Beat-seeded ecology.** Each drum voice stamps a different species, so kick, snare and hats compete as visible populations.
- **Footage as food.** Feed blurred clip luma into the field to lower mu locally. Creatures gather on bright shapes. Use slow footage only, and keep the effect small or bright areas fill solid.
- **Lenia as displacement.** Use the blurred field as a height map to refract a second clip.
- **Multi-channel colour.** A 3-channel Lenia from Expanded Universe makes RGB interacting species ([arXiv 2005.03742](https://arxiv.org/abs/2005.03742)).
- **Flow-Lenia mixing.** Coexisting species with mass conservation, so beats move matter instead of creating it ([arXiv 2212.07906](https://arxiv.org/abs/2212.07906)).

## Did you know?

- Conway's Game of Life is a special case of discrete Lenia, with a square kernel and a rectangular growth rule ([Wikipedia](https://en.wikipedia.org/wiki/Lenia)).
- The multi-channel architecture resembles a recurrent residual convolutional network, which links Lenia to neural cellular automata ([arXiv 2005.03742](https://arxiv.org/abs/2005.03742)).
- In 3D Lenia, stable creatures are often spherical and sometimes arrange internal voids like Platonic solids ([arXiv 2005.03742](https://arxiv.org/abs/2005.03742)).
- Some Lenia creatures keep their boundaries when they collide, and some self-replicate ([arXiv 2005.03742](https://arxiv.org/abs/2005.03742)).

## Sources

- [arXiv 1812.05433, Lenia: Biology of Artificial Life](https://arxiv.org/abs/1812.05433)
- [ar5iv version](https://ar5iv.labs.arxiv.org/html/1812.05433)
- [arXiv 2005.03742, Lenia and Expanded Universe](https://arxiv.org/abs/2005.03742)
- [arXiv 2212.07906, Flow-Lenia](https://arxiv.org/abs/2212.07906)
- [Wikipedia: Lenia](https://en.wikipedia.org/wiki/Lenia)
- [Lenia portal](https://chakazul.github.io/lenia.html)
- [GitHub Chakazul/Lenia](https://github.com/Chakazul/Lenia)
- [Lenia web demo](https://chakazul.github.io/Lenia/JavaScript/Lenia.html)
- [ALIFE encyclopedia: Lenia](https://alife.org/encyclopedia/software-platforms/lenia/)
- [Interspecifics Codex Virtualis](https://interspecifics.cc/work/category/bacteria/)
- [CAX Lenia example](https://maxencefaldor.github.io/cax/examples/20_lenia/)
- [From Conway to Lenia tutorial notebook](https://colab.research.google.com/github/OpenLenia/Lenia-Tutorial/blob/main/Tutorial_From_Conway_to_Lenia.ipynb)
- [Bert Chan's shaders on Shadertoy](https://www.shadertoy.com/user/chakazul)
- [ISF multipass reference](https://docs.isf.video/ref_multipass.html)
- [Godot docs, compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)
- [Godot docs, Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html)
