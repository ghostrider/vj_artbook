---
tags:
  - math
  - attractors
  - particles
  - generative
---

# Clifford Strange Attractors

> Two lines of trigonometry, iterated a few million times, turn into a glowing veil that never sits still.

## The idea
The Clifford attractor is a two-dimensional iterated map. Take a point, feed it through two equations, take the result, repeat. Four numbers (a, b, c, d) decide the shape. The equations are `x' = sin(a·y) + c·cos(a·x)` and `y' = sin(b·x) + d·cos(b·y)` ([Bourke](https://paulbourke.net/fractals/clifford/)).

Drawing the points one by one looks thin. The trick is to count instead: every pass through a pixel raises a counter on a large grid, and the counts become brightness through a gradient map. Bourke notes that many more iterations are needed than you would expect for smooth gradients ([Bourke](https://paulbourke.net/fractals/clifford/)). The map is only 2D, yet the result often reads as a 3D structure seen flat ([Bourke, 3D](https://paulbourke.net/fractals/attract3d/index.html)).

On a club screen this gives silky filaments and smoky veils, with bright caustic-like seams where density piles up. Because the shape is a long-run orbit, animation means nudging the four parameters in tiny steps. The form then stretches, folds and collapses into new figures ([Sekara](https://vedransekara.github.io/2019/10/01/temporal_attractors.html)). Four faders, or four slow LFOs, are a complete instrument.

## Origins and references
- [Paul Bourke, Clifford Attractors](https://paulbourke.net/fractals/clifford/) (2004): the map, attributed to Cliff Pickover, with graphics, parameter sets and source code from Paul Richards.
- [Paul Bourke, Peter de Jong Attractors](https://paulbourke.net/fractals/peterdejong/): the sister map, published in Scientific American "Computer Recreations", July 1987.
- [Paul Bourke, Attractors in 3D for VR](https://paulbourke.net/fractals/attract3d/index.html) (2021): the sine/cosine idea in 3D, with roughly 1,000 to 10,000 million points binned into equirectangular histograms.
- [Ricky Reusser, Clifford and de Jong Attractors](https://rreusser.github.io/notebooks/clifford-and-de-jong-attractors) (2025): interactive WebGL notebook. Points accumulate on a texture as a histogram. Colour comes from the distance of the last step, lightness from the log of the hit count.
- [Dirk Brockmann, Stranger Things](https://www.complexity-explorables.org/explorables/stranger-things/) (2018): sliders for Clifford, Bedhead, Fractal Dream and Gumowski-Mira maps.
- [orbita2d/clifford](https://github.com/orbita2d/clifford): Python renderer that writes animated GIF and MP4 frame sequences.

## Visual language
One luminous form floats on near-black, like smoke, silk or folded gauze. Thin bright filaments pile into hot seams. Wide veils of faint mist fade out between them.

Brightness follows the log of the hit count, which gives a soft glow. Dense seams burn white-amber, mid-density veils glow teal and ember, sparse edges fall to indigo and then black. Colour is indexed by mean step length: slow parts of the orbit read cool, fast parts read warm. That gives every form a two-tone depth without any 3D.

Motion is a continuous morph. The shape breathes along a closed loop in parameter space, and on bar or phrase changes it re-forms into a new figure. Good starting parameters are a=-1.4, b=1.6, c=1.0, d=0.7 ([Bourke](https://paulbourke.net/fractals/clifford/)). The mood is hypnotic and slightly cosmic: a long-exposure photograph of something alive.

## See it
- [Bourke, Clifford Attractors](https://paulbourke.net/fractals/clifford/) — still renders and parameter sets
- [Reusser, Clifford and de Jong notebook](https://rreusser.github.io/notebooks/clifford-and-de-jong-attractors) — interactive real-time WebGL render
- [Brockmann, Stranger Things](https://www.complexity-explorables.org/explorables/stranger-things/) — sliders for four attractor maps
- [Sekara, Strange Attractors Part II](https://vedransekara.github.io/2019/10/01/temporal_attractors.html) — parameter-sweep GIF animations
- [orbita2d gallery repo](https://github.com/orbita2d/clifford) — frame-sequence animations of the attractor
- [Evolution of a Clifford Attractor](https://www.youtube.com/watch?v=ZU1EitsgVC8) (YouTube, Jeremy A. Owen) — attractor evolving as parameters change
- [Clifford attractor tutorial Max/MSP/Jitter](https://www.youtube.com/watch?v=Xqlmd3i24VU) (YouTube, Ryan Dunn) — real-time build in a visual patching tool
- [Timothy Tan, Clifford Attractor Improv](https://www.youtube.com/watch?v=qeIUPP9B3bM) (YouTube, timbretan) — improvised performance with the attractor

## Code design
The core is the same everywhere. Keep about a million orbit particles alive between frames. Each frame, every particle applies the map several times (default 16) and adds a hit to a pixel histogram. A second accumulator sums step lengths, so colour can follow Reusser's colour-by-step idea. Because particle state persists, a parameter change restarts nothing: the points fall onto the new attractor within one frame or two, and morphs look continuous. A small share of particles (0.5 %) is re-seeded each frame, so none stay trapped when an orbit collapses to fixed points.

A decaying accumulation buffer (`A = A·decay + new`) is the trail control. A tone map turns density into lightness with a log curve, and a 5-stop ramp indexed by mean step length gives the colour. A feedback echo (zoom 1.004, 0.05° rotation, decay 0.90) adds smoky trails during morphs. A hot-core layer, thresholded and blurred, makes the seams read as caustics. Bloom, grain and vignette finish it, in that order.

Optional extras: a blend of the Clifford and de Jong update rules (`mix(clifford(p), dejong(p), family)`), which gives hybrid shapes rather than an image crossfade, and a breathing loop `a = a0 + r·cos(2πφ)` and similar, which repeats seamlessly. The loop idea follows [gumeo](https://gumeo.github.io/post/visualizing-strange-attractors). Keep r between 0.02 and 0.15. Above about 0.2 the shape often breaks. Presets come from Bourke, for example (−1.4, 1.6, 1.0, 0.7) and (1.6, −0.6, −1.2, 1.6). Morph between them with a smootherstep ease, and audition every pair in rehearsal.

=== "Resolume Wire"
    Wire has no vertex shaders ([Resolume ISF](https://resolume.com/support/en/isf)), and multipass ISF support depends on the version. A full histogram is therefore not realistic inside Wire. The design offers two options.

    **A (recommended): Wire as colour and control patch.** An engine (UE5, Godot, TouchDesigner or a small Rust app) renders the density and sends it over Spout as a data frame: R = tone-mapped log density, G = colour index. In Wire:

    - `Spout Receiver` into an ISF node "attractor-colour" (ramp lookup, highlight mix, flash).
    - `Blend` (Add) with a feedback branch: `Transform` zoom 1.004, `Multiply` 0.90.
    - Blur plus add for bloom, then output.
    - Control side: BPM and bar counter, a `Counter` wrapping at 8, an array lookup of preset a..d. Sine and cosine `LFO`s (period = loop length in bars) times drift are added to a and b. A `Smooth` node handles the morph. An OSC Out node sends a, b, c, d, family and decay to the engine.
    - Exposed: shape, drift, morph, trails, glow, family, palette.

    Sending tone-mapped values keeps 8-bit transport usable. Raw counts would band.

    **B: sketch mode inside Wire only.** It is thin, but it works as a calm line-drawing variant. A single-pass ISF in which every pixel recomputes the same short orbit (K = 96 points, seeded from the frame number) and sums a Gaussian splat `exp(−d² / σ²)` with σ = 1.2 px around each orbit point. Feed this into a Wire feedback loop with decay 0.985, so about 6,000 points build into fine glowing threads. Render at 960 × 540 and upscale. Cost is K iterations per pixel, so keep K ≤ 128.

    The design also sketches an Arena-side stack for the engine's Spout feed: a solid #04030A layer, the Spout source with the colour effect (Add), a blurred and levelled duplicate tinted cream as hot core (Add, 40 %), then Bloom and Vignette on the composition.

=== "Unreal Engine 5.7"
    This is a build plan from the designer's own knowledge, not a tested result.

    - **Niagara system `NS_Clifford`**: GPU Compute Sim, fixed bounds ±300 cm. Spawn Burst Instantaneous 2,097,152 particles (1 M on laptops), infinite lifetime.
    - **Particle Spawn**: a Scratch Pad sets a random point in [−2, 2]² in a custom `Particles.Orbit` Vector2 attribute, with warm-up 12.
    - **Particle Update**: a Scratch Pad with one Custom HLSL node applies the map once to `Orbit` (User.A to User.D, User.A2 to User.D2, User.Family). It writes `Particles.Step`, handles respawn (0.5 % per frame, plus NaN and out-of-bounds checks) and sets Position = Orbit × User.Scale on the camera plane. No forces are needed.
    - **Sprite Renderer**: unlit additive material `M_OrbitPoint`, Emissive = (1, Step, 0) × 0.01, sprites about 1.5 px under an orthographic camera.
    - **Tone and colour**: post-process material `PP_AttractorTone` (Replacing the Tonemapper) reads PostProcessInput0. It computes h = R × norm, t = G / R, then L = log2(1 + h·gain) / log2(1 + peak·gain), and looks up the ramp from a Curve Atlas row or a 256 × 1 texture. For decay beyond what one frame gives, capture with a SceneCapture2D into an RGBA16F render target and ping-pong with a Blueprint `Draw Material to Render Target`.
    - **Disable** TSR/TAA and motion blur, which smear the point cloud. Fix exposure manually, because auto-exposure fights the flash.
    - **Post-process volume**: bloom intensity 1.3, threshold 0.7, film grain 0.02, vignette 0.3.
    - **Remote Control preset** (OSC and MIDI): User.A to User.D, Family, Drift, Decay, Gain, Peak, HueShift, Flash. Do the preset morph and the breathing loop in a Blueprint (Timeline plus sin/cos), driven by an OSC bar counter.
    - **Output**: Spout through a third-party plugin (for example Off World Live's toolkit) or NDI through the NDI SDK for Unreal, at the LED processor's native resolution.

=== "GODOT"
    This is a build plan from the designer's own knowledge, not a tested result.

    **Best path (Forward+ or Mobile renderer):** a GLSL 450 compute shader as described in the [Godot compute shader docs](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html).

    - Run it on the main RenderingDevice (`RenderingServer.get_rendering_device()`), so no CPU readback is needed.
    - Bindings: a particle state SSBO (vec2 + uint per particle), uint SSBOs H and S at output size, and a uniform block for a..d, a2..d2, family, k, scale and seed.
    - Pass 1 iterates and does `atomicAdd`. Pass 2 resolves into an RG32F image with decay, then clears H and S.
    - Show the image through a `Texture2DRD` on a full-screen `TextureRect`. Split dispatches if a frame runs long, since the docs warn about Windows TDR timeouts.
    - Colour: a `canvas_item` shader on the `TextureRect` with uniforms `gain`, `peak`, `hue_shift`, `flash`, `highlight_color` and a `GradientTexture1D` ramp with 5 stops.

    **Fallback (Compatibility renderer):** `GPUParticles2D` with 1,048,576 particles and explosiveness 1. A `particles` shader applies one map step per frame to a position stored in `CUSTOM.xy`, with `COLOR = vec4(1, step, 0, 1)` and a 1 px texture. Render into a `SubViewport` with HDR 2D and an additive `CanvasItemMaterial`. A second `SubViewport` does the `A·decay + new` feedback, and a third does the echo.

    Post: `WorldEnvironment` glow intensity 1.3, HDR threshold 0.7. Control: `InputEventMIDI` for faders and pads, an OSC addon for Resolume clock input, and a community Spout addon or NDI for output.

## On stage
**Audio reactivity.** Only the kick is fast. Keep audio-driven parameter changes under ±0.05, because the shape is sensitive and big jumps make it collapse or flicker. Big changes come from presets.

| band | drives | smoothing |
|------|--------|-----------|
| Kick, 40–100 Hz onset | flash 0.4, decay dip to 0.65, d += 0.02 | attack 2 ms, release 150 ms (flash), 300 ms (nudge) |
| Bass, 60–250 Hz | drift radius up to +0.04 | attack 80 ms, release 600 ms |
| Mids, 300 Hz–2 kHz | hueShift ±0.08 | attack 150 ms, release 1 s |
| Hi-hats, 6–12 kHz | gain +20 %, more mist sparkles in | attack 1 ms, release 80 ms |
| Bar / phrase clock | preset morph every 8 bars, family sweep per 32 bars, palette per 16 bars | none |

**MIDI mapping (KeyStep 37 / BeatStep Pro).**

| control | parameter | notes |
|---------|-----------|-------|
| BeatStep Pro pads 1–8 | shape (preset 0–7) | morphs by default, snaps while a "cut" pad is held |
| KeyStep knob | drift, 0–0.2 | bass level adds up to +0.04 |
| Knob | morph time, 0–4 bars | set to 0 on drops |
| Fader | trails (decay 0.6–0.97) | kick dips it to 0.65 |
| Fader | glow (peak 8–200, inverted) | kick sets flash |
| Fader | family, 0–1 | slow moves only |

**Song structure.**

- **Breathing loop**: 16 bars, radius 0.06. It never stops, so the form is never static.
- **Preset morph**: every 8th bar, over 2 bars. Shorten to 1 bar in builds.
- **Build**: radius grows from 0.06 to 0.15 over 8 bars, hueShift runs 4× faster, echo rises to 45 %.
- **Drop**: hard cut. Preset jump with morph time 0, echo 0, decay 0.6, flash 1.0 on the hit. Then the loop resumes at radius 0.1 with a 4-bar length.
- **Break**: decay 0.97 for a long exposure, radius 0.03, 32-bar loop, bloom +0.3, the "Ice" palette, slow rotation of 2° per bar.

**Layering.** All light layers are Add: light on black, overlapping veils should brighten. The density grey alone works as a Luma Key over video in Resolume, so footage shows through the smoke. Set the echo to 0 on hard cuts so the old form does not ghost into the new one. A 1-beat chromatic split of 3 px works as a drop accent.

**Palette.** Base #04030A, ramp #1B0F3A, #0E7C86, #FF6B35, #FFB347, highlight #FFE9C7. Alternatives: "Ice" (#050814, #1C2E6B, #3FA9F5, #B8F3FF, #FFFFFF) and "Acid" (#06040A, #2B0B3F, #C2185B, #FFD23F, #F7FFF0). Swap on 16-bar phrase boundaries with a 1-beat ramp crossfade.

**Performance and walls.**

- Aim for roughly 4–8 hits per output pixel per frame before decay. At 1080p that is 1 M particles × 8 steps. At 4K, 2 M × 16. Below that, the mist turns to speckle. Grain and decay hide some of it.
- Expect about 1–3 ms for the iteration pass on a current desktop GPU (the designer's estimate). Atomic contention in the brightest seams is the real limit: lower the step count and raise the particle count.
- Float32 is fine live. Reusser notes single precision is not fully accurate and offers a CPU mode for final renders ([notebook](https://rreusser.github.io/notebooks/clifford-and-de-jong-attractors)).
- LED walls: keep black at #04030A, cap L at 0.92 before the highlight mix, and add no extra contrast curves. On high-pitch walls raise peak (for example 80) so thin filaments do not vanish.
- Ultra-wide strips (32:9): crop rather than letterbox. Dense seams fill the strip, or render two instances side by side.
- Pitfalls: drift above 0.2, unaudited preset pairs, auto-exposure and TAA in UE, decay above 0.97 during fast morphs, framing that follows c and d without smoothing.

## Variations
1. **Clifford to de Jong hybrid.** Blend the two update rules, not the images ([Bourke, de Jong](https://paulbourke.net/fractals/peterdejong/)). Sweep it over 2–4 bars once per phrase.
2. **Closed-loop parameter paths.** A circle in 4D parameter space makes a seamless loop ([Sekara](https://vedransekara.github.io/2019/10/01/temporal_attractors.html)).
3. **Colour by velocity or curvature.** Bourke notes the increment can be a function of curvature ([Bourke](https://paulbourke.net/fractals/clifford/)).
4. **Feedback trails.** Feed the density back with a slow decay and a slight zoom.
5. **3D and dome.** Extend to 3D and project onto a rotating camera or a dome, following [Bourke's VR work](https://paulbourke.net/fractals/attract3d/index.html).

## Did you know?
- Bourke's VR renders bin 1,000 to 10,000 million points per image ([Bourke](https://paulbourke.net/fractals/attract3d/index.html)).
- Brockmann's explorable iterates the map 200,000 times per display. Clifford and Fractal Dream have four parameters, Bedhead and Gumowski-Mira two ([explorable](https://www.complexity-explorables.org/explorables/stranger-things/)).
- The de Jong map was published in 1987. Bourke's Clifford write-up dates from 2004 ([de Jong](https://paulbourke.net/fractals/peterdejong/), [Clifford](https://paulbourke.net/fractals/clifford/)).
- Sekara's colour animation varies each of a, b, c, d in its own panel ([post](https://vedransekara.github.io/2019/10/01/temporal_attractors.html)).

## Sources
- <https://paulbourke.net/fractals/clifford/>
- <https://paulbourke.net/fractals/peterdejong/>
- <https://paulbourke.net/fractals/attract3d/index.html>
- <https://rreusser.github.io/notebooks/clifford-and-de-jong-attractors>
- <https://www.complexity-explorables.org/explorables/stranger-things/>
- <https://vedransekara.github.io/2019/10/01/temporal_attractors.html>
- <https://github.com/orbita2d/clifford>
- <https://gumeo.github.io/post/visualizing-strange-attractors>
- <https://resolume.com/support/en/isf>
- <https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html>
- <https://www.youtube.com/watch?v=ZU1EitsgVC8>
- <https://www.youtube.com/watch?v=Xqlmd3i24VU>
- <https://www.youtube.com/watch?v=qeIUPP9B3bM>
