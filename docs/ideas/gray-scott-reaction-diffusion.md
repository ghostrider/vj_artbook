---
tags:
  - biology
  - simulation
  - shader
  - organic
---

# Gray-Scott Reaction-Diffusion

> Two imaginary chemicals spread and react on a grid, and the wall grows spots, worms and coral that split like living cells.

![Gray-Scott reaction-diffusion simulation with parameters varying radially: spots in the centre, labyrinth stripes at the edge (only 400x400 px)](https://upload.wikimedia.org/wikipedia/commons/a/a3/Gray-Scott_Lotus_%282125994225%29.jpg)
*Gray-Scott reaction-diffusion simulation with parameters varying radially: spots in the centre, labyrinth stripes at the edge (only 400x400 px). Image: [Jonathan Lidbeck](https://commons.wikimedia.org/wiki/File:Gray-Scott_Lotus_(2125994225).jpg), [CC BY 2.0](https://creativecommons.org/licenses/by/2.0).*

## The idea

Every cell on the grid holds two amounts, A and B. Each step, both spread to their neighbours, A faster than B. Then the reaction runs: where two Bs meet one A, the A turns into B. A is fed in at rate f, and B is removed at rate k ([Karl Sims](https://www.karlsims.com/rd.html)). Two numbers, f and k, decide whether you get spots, stripes or labyrinths.

The pattern comes from edges. Convex edges of a B region receive A from outside and grow. Concave edges starve and shrink. A spot can swell and split like a cell ([Karl Sims](https://www.karlsims.com/rd.html)). The roots go back to Alan Turing's 1952 paper on morphogenesis: a state that is stable in a well-mixed system can become unstable once diffusion is added, and that gives patterns ([Wikipedia](https://en.wikipedia.org/wiki/Reaction%E2%80%93diffusion_system)).

On a club screen it works because it is alive and tunable. The output is soft-edged, high-contrast blobs that grow, divide and settle. Colour is free, since the simulation only gives one scalar field. The risk is stillness: left alone, the pattern settles into something static. So the design keeps injecting chemical on the beat and keeps moving f and k.

## Origins and references

- Alan Turing, 1952, "The chemical basis of morphogenesis": diffusion can destabilise a stable mixture ([Wikipedia](https://en.wikipedia.org/wiki/Reaction%E2%80%93diffusion_system)).
- P. Gray and S. K. Scott, 1984: autocatalytic reactions in a stirred tank reactor, the origin of the model. They called the k term "decay" or "poisoning" ([mrob.com xmorphia](https://mrob.com/pub/comp/xmorphia)).
- John Pearson, 1993, "Complex patterns in a simple system": began a classification of the patterns across f and k ([mrob.com xmorphia](https://mrob.com/pub/comp/xmorphia)).
- Lee et al., *Nature*, 1994: self-replicating spots reported in a real chemical experiment ([Wikipedia](https://en.wikipedia.org/wiki/Reaction%E2%80%93diffusion_system)).
- Karl Sims: the standard practical explainer for graphics people, with parameters and extensions (orientation, style map, flow, scale) ([karlsims.com](https://www.karlsims.com/rd.html)).
- Robert Munafo, "xmorphia": a map of the Gray-Scott parameter space after Pearson ([mrob.com](https://mrob.com/pub/comp/xmorphia)).
- Simon Gladman, Gray-Scott solved in VEX in Houdini 15.5 (Nov 2016) ([SideFX](https://www.sidefx.com/gallery/reaction-diffusion-in-houdini/)).
- Hamy, "C10ud": an audio-driven Gray-Scott visual. Beats found offline fire a handler that inserts new pools of B ([hamy.xyz](https://hamy.xyz/blog/building-a-reaction-diffusion-visualizer-in-p5js)).
- Pao Olea, TouchDesigner tutorial (April 2025): builds reaction-diffusion, then adds audio reactivity ([Derivative](https://derivative.ca/community-post/tutorial/reaction-diffusion-touchdesigner-tutorial-19/71701)).

## Visual language

Think of a wet surface under a microscope. Soft blue-green blobs sit on near-black ink. They grow, push out fingers, split and close into labyrinths and coral. The field is lit like a gel: each blob has a rounded, shiny top with a small warm-white highlight. Fronts that are still growing glow orange at the edge, so the eye always finds where things are happening.

The palette borrows from bromothymol blue, the pH indicator Pearson matched his pattern colours to ([mrob.com](https://mrob.com/pub/comp/xmorphia)): deep indigo, teal, and an acid yellow at the crests.

| role | hex | use |
|------|-----|-----|
| base | #03050C | empty background |
| shadow | #0B2E4F | blob flanks |
| body | #22A699 | main blob colour |
| highlight | #F2D64B | crests |
| specular | #FFF6D8 | gel highlight |
| accent | #FF5A36 | growth glow on active fronts |

Motion is the growth itself: seeds open into rings, rings break into spots, spots divide. The mood is microscopic, biological and slow-breathing. On the drop the wall is reseeded and the look jumps from mitosis (splitting spots) to coral (branching growth). On a break the kill rate rises and the pattern dissolves back to black.

## See it

- [Karl Sims: Reaction-Diffusion Tutorial](https://www.karlsims.com/rd.html) — explainer with example pattern images and the parameter map.
- [Munafo xmorphia](https://mrob.com/pub/comp/xmorphia) — Gray-Scott parameter-space atlas after Pearson.
- [piellardj reaction-diffusion-webgl](https://github.com/piellardj/reaction-diffusion-webgl) — real-time GPU sim with a live demo link, uniform and image-driven modes.
- [SideFX gallery: Reaction Diffusion in Houdini](https://www.sidefx.com/gallery/reaction-diffusion-in-houdini/) — Simon Gladman's animated Gray-Scott in VEX.
- [Coding Challenge #13: Reaction Diffusion Algorithm in p5.js](https://www.youtube.com/watch?v=BV9ny785UNc) (YouTube, The Coding Train) — video tutorial that builds the sim live.
- [Fun with Reaction-Diffusion in Houdini, Dan Wills, SIGGRAPH Asia 2019](https://www.youtube.com/watch?v=K_7TkoIkFhk) (YouTube, Houdini) — talk on extending Gray-Scott in several ways.

## Code design

The design is a ping-pong simulation of two floating-point textures, run several steps per frame, then a shading pass that turns the scalar field into a gel look. Sim resolution is half the output (960×540 for 1080p); bilinear upscaling of a smooth field looks clean.

**The update.** A starts at 1, B at 0, with small seeded areas of B at 1. The Laplacian uses Sims' 3×3 kernel: centre −1, edges 0.2, corners 0.05. With D_A = 1.0, D_B = 0.5 and dt = 1.0: `A' = A + (D_A·lapA − A·B² + f·(1 − A))·dt` and `B' = B + (D_B·lapB + A·B² − (k + f)·B)·dt` ([Karl Sims](https://www.karlsims.com/rd.html)). Clamp both to 0..1.

**Speed is steps, not dt.** With this kernel, D_A = 1 and dt = 1 are already at the edge of what explicit Euler tolerates. Raising either gives checkerboard noise. Instead run N integer steps per frame: default 12, from 1 (almost frozen, for breaks) to 32 (the drop). At 60 fps that is 720 steps per second. The design estimates a seeded droplet becomes a visible ring in about one beat at 128 BPM. These step counts are untested estimates, so tune them on the real rig. Lock the output frame rate, because growth halves if the frame rate does.

**Pattern size.** A `scale` factor of 0.3–1.0 multiplies only the reaction and feed/kill terms. Diffusion is untouched, so stability holds. Lower scale gives bigger, slower blobs ([Karl Sims](https://www.karlsims.com/rd.html) names scale as a variant).

**Style map.** A video or noise layer sets f and k per pixel, mixing mitosis (f 0.0367, k 0.0649) for dark areas and coral (f 0.0545, k 0.062) for bright ones. This follows [piellardj](https://github.com/piellardj/reaction-diffusion-webgl), which blends two presets by image luminance. The field traces the video shapes only over minutes, so use slow footage or a blurred camera.

**Injection.** Kicks add a Gaussian disc of B at a position hashed from the beat counter. Each bar adds a small sprinkle, and a held gate paints B under a brush. In UE and Godot a watchdog reads mean B and reseeds if the field is dead, or raises k if it is saturated.

**Shading.** B goes through a contrast window (lo 0.08, hi 0.32), a four-stop ramp, normals from central differences, and a Blinn specular. The last sim step also writes the growth `max(B' − B, 0)` into a third channel for the orange glow. Post: black crush, bloom, 1.5 % grain.

=== "Resolume Wire"
    - Resolume has no built-in reaction-diffusion effect. The route is an ISF multipass effect, which Wire can host. The ISF spec lets each pass set `TARGET`, `PERSISTENT` (kept across frames) and `FLOAT` (32-bit per channel) ([ISF JSON reference](https://docs.isf.video/ref_json.html), [ISF multipass](https://docs.isf.video/ref_multipass.html)).
    - Two persistent float buffers, `simA` and `simB`, at half the output size. Passes alternate between them, N even (default 12). The last pass is the shade pass at full size. The shader branches on `PASSINDEX` and runs the init when `FRAMEINDEX == 0` ([ISF variables](https://docs.isf.video/ref_variables.html)).
    - The step count is fixed by the number of passes in the JSON. For a live speed control, write 32 passes and let a `speed` input make later passes copy their input unchanged. Copy passes still cost a full-buffer copy each.
    - Node outline: effect input (style map) → resize to half → ISF node → output. A blur on the style-map input, radius 2–3 %, keeps video edges from imprinting pixel noise.
    - Control side: beat node → trigger → envelope (instant attack, 100 ms release) → `inject`. A beat counter, hashed, gives the seed position. A slow smoothed square LFO, one cycle per 8 bars, drives `morph`. A low-mid audio band drives `flow`. Node names differ between Wire versions, so check the node browser.
    - **Unconfirmed: float buffers.** Resolume documents ISF only inside Wire, and multipass and persistent-buffer support depends on the version. Whether the host honours `FLOAT` is not documented by Resolume. Pre-gig test: run the mitosis preset for 30 seconds. If spots freeze, die instantly or show stair-stepped edges, the buffer is 8-bit. The fallback is to pack each value as 16-bit fixed point across two 8-bit channels, as [piellardj](https://github.com/piellardj/reaction-diffusion-webgl) does on WebGL 1. Sample packed buffers only at texel centres.
    - Most reliable alternative: run the sim in UE or Godot and send it into Arena as a Spout source, so Arena only composites.

=== "Unreal Engine 5.7"
    - **Route A, recommended: Blueprint ping-pong.** Two render targets `RT_S0` and `RT_S1`, 960×540, RGBA16f or RGBA32f, mips off, address Wrap, filter Nearest. One 1920×1080 RGBA8 target for output.
    - `M_RDStep` is an unlit surface material with a Texture Object parameter `State`, nine texture samples (or one Custom node with a 9-tap loop), then Laplacian, reaction, f/k from a Lerp on style-map luma, scale, inject disc, growth channel and Clamp.
    - Blueprint on Tick: a For Loop from 0 to Steps − 1. Set the dynamic material's `State` to the current read target, call Draw Material to Render Target into the other, then swap. Use an even N. Init draws `M_RDInit` into `RT_S0` on BeginPlay and on reset. `M_RDShade` reads the final target and draws into the output target.
    - **Route B: Niagara.** A Grid2D Collection with attributes A and B, 960×540. A simulation stage "Init" runs on reset, and a stage "Step" iterates over the grid with Num Iterations set to steps per frame. Epic's fluid docs describe Grid2D Collections as the grid storage and Num Iterations as the way to repeat a stage ([Epic, fluid simulation overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/fluid-simulation-in-unreal-engine---overview)). Check in your version whether Num Iterations can be bound to a user parameter for live speed changes. If not, use Route A.
    - Watchdog: mip-reduce a small copy of the state, read the 1×1 mip every 30 frames, not every frame, and reseed when mean B < 0.002.
    - Output via Spout or NDI into Arena. Remote Control: a Material Parameter Collection with `Morph`, `KillOffset`, `Scale`, `Inject`, `Flow`, `Glow`, seed and brush position. Add `Steps`, `Reset` and `RingReseed` to a Remote Control Preset and bind OSC and MIDI.

=== "GODOT"
    - Godot 4.x with Forward+ (compute needs a RenderingDevice-based renderer, Forward+ or Mobile) ([Godot docs, compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)).
    - **Route A, compute, recommended.** Two RGBA16F or RGBA32F storage images, 960×540. A GLSL compute shader runs one step per dispatch with 8×8 workgroups. Every frame the script dispatches Steps times, alternating image bindings. Wrap the final image in a `Texture2DRD` through `texture_rd_rid` ([Godot docs, Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html)) and show it on a full-screen ColorRect with a `canvas_item` shader that does the shading. Do not call `sync()` every frame ([Godot docs, compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)).
    - **Route B, no compute, one step per frame.** Two SubViewports at 480×270 with `use_hdr_2d = true` ([Godot docs, Viewport](https://docs.godotengine.org/en/stable/classes/class_viewport.html)), each holding a ColorRect with the step shader that samples the other's ViewportTexture. A script alternates `UPDATE_ONCE` between them ([Godot docs, SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)). That gives 60 steps per second, so use the lower resolution and a scale of 0.5–0.7.
    - Audio comes from an AudioEffectSpectrumAnalyzer on the music bus. Output to Arena via a Spout GDExtension if available, or NDI via screen capture.

## On stage

**Audio reactivity.**

| band | drives | smoothing |
|------|--------|-----------|
| kick, 40–120 Hz | one seed disc per hit, applied once on the rising edge | attack instant, release 100 ms; trigger at 0.6 of the running peak, retrigger blocked 150 ms |
| bass, 60–250 Hz | flow 0 → 0.4 | attack 80 ms, release 600 ms |
| snare / clap, 1–4 kHz | seed radius ×1.5, or a mirrored second seed | attack 5 ms, release 150 ms |
| hats, 6–12 kHz | tiny sprinkle, 1–2 seeds, max 4 per beat | attack 5 ms, release 60 ms |
| RMS loudness | speed 8 → 24 steps (even integers), glow 0.3 → 1 | attack 300 ms, release 3 s |

Do not drive f or k from fast audio. Every small change re-shapes the whole wall at once and reads as flicker. Only morph and kill offset move, slowly, on bar boundaries.

**MIDI mapping.**

| control | parameter | range |
|---------|-----------|-------|
| BeatStep Pro knob 1 | morph (0 = mitosis, 1 = coral) | 0–1 |
| BeatStep Pro knob 2 | kill offset, positive starves the pattern | −0.006 to +0.008 |
| BeatStep Pro knob 3 | speed, steps per frame | 1–32, even |
| BeatStep Pro knob 4 | scale, low = big slow blobs | 0.3–1.0 |
| KeyStep mod strip | flow | 0–0.6 cells per step |
| KeyStep XY + gate | brush position and paint | — |
| Pads | reset, ring reseed, manual seed, drop = 24 steps | — |

**Arc of a set.**

- Base: mitosis, 12 steps, scale 0.8, flow 0.1. A kick seed every beat, a sprinkle every bar.
- Every 4 bars, crossfade f/k between mitosis and coral over one bar. Jumping f/k makes the whole field flash.
- Build-up: steps rise from 8 to 24 over the last 4 bars, scale drops from 0.9 to 0.6, flow rises to 0.4.
- Drop: on the downbeat add a ring reseed to the existing state, never clearing it, so the wall never goes black. Switch to coral over 2 beats, steps 24, glow 1.
- Break: kill rises to 0.068 over 2 bars, steps drop to 4, and only the brush paints. Return k to the preset one bar before the next drop.
- Never leave the field without injection for more than 8 bars.

**Layering.** Use the field as a luma mask on a video clip, or multiply it over the clip in Arena. For a refractive look, displace the clip by the field's gradient. Post order: crush, bloom, grain. On LED walls the crush keeps blacks fully off; keep body luminance moderate and let only crests and speculars go bright.

**Performance.** The sim is cheap: 12 steps at 960×540 is about 56 M texture fetches per frame, well under 1 ms on a current desktop GPU, and two RGBA16F targets take about 8 MB. The ISF route is heavier, since every step is a full pass with host overhead, so stay at 12–16 passes. Keep sim cells square: for a 5760×1080 strip use a 1440×270 sim, not a stretched 16:9 one. Wrap addressing lets patterns cross the seam. Pitfalls: 8-bit buffers, raising dt or D, mipmaps on the state, odd step counts without tracking the current buffer, and an empty field after a long break.

## Variations

- Image-driven f/k: video or camera brightness sets local f/k, so the sim approximates the picture ([piellardj](https://github.com/piellardj/reaction-diffusion-webgl)).
- Directional diffusion (Sims' orientation option) for combed, wood-grain or fingerprint textures ([Karl Sims](https://www.karlsims.com/rd.html)).
- Flow: advect the chemicals along a noise or optical-flow field, from 0 to 0.6 cells per step. Above about 0.8 the pattern smears ([Karl Sims](https://www.karlsims.com/rd.html)).
- Three coupled sims, one per colour channel, with different f/k each, added together as in piellardj's colour mode. Costs three times the sim.
- Alternate drop palette: rotate the ramp hue by +150° and switch the accent to #3DF5FF for one phrase.
- Run the sim on a mesh surface, the Houdini route ([SideFX](https://www.sidefx.com/gallery/reaction-diffusion-in-houdini/)), or as a UE5 Niagara grid lit as a bump map.

## Did you know?

- Pearson's pattern colours were matched to the pH indicator bromothymol blue used in experiments ([mrob.com](https://mrob.com/pub/comp/xmorphia)).
- Gray and Scott's system was a chemical stirred tank, with F meaning feed or flow of pure U ([mrob.com](https://mrob.com/pub/comp/xmorphia)).
- Self-replicating spots were seen in a real chemical experiment, reported in 1994 ([Wikipedia](https://en.wikipedia.org/wiki/Reaction%E2%80%93diffusion_system)).
- WebGL 1 has no 16-bit float texture, so one browser version packs each value into two 8-bit channels ([piellardj](https://github.com/piellardj/reaction-diffusion-webgl)).

## Sources

- [Karl Sims, Reaction-Diffusion Tutorial](https://www.karlsims.com/rd.html)
- [Wikipedia, Reaction-diffusion system](https://en.wikipedia.org/wiki/Reaction%E2%80%93diffusion_system)
- [Munafo, xmorphia](https://mrob.com/pub/comp/xmorphia)
- [Hamy, Building a Reaction Diffusion Audio Visualizer](https://hamy.xyz/blog/building-a-reaction-diffusion-visualizer-in-p5js)
- [Derivative, Reaction Diffusion TouchDesigner tutorial](https://derivative.ca/community-post/tutorial/reaction-diffusion-touchdesigner-tutorial-19/71701)
- [SideFX gallery, Reaction Diffusion in Houdini](https://www.sidefx.com/gallery/reaction-diffusion-in-houdini/)
- [piellardj, reaction-diffusion-webgl](https://github.com/piellardj/reaction-diffusion-webgl)
- [The Coding Train, challenge 13](https://thecodingtrain.com/challenges/13-reaction-diffusion)
- [Coding Challenge #13 on YouTube](https://www.youtube.com/watch?v=BV9ny785UNc)
- [Dan Wills, Fun with Reaction-Diffusion in Houdini, SIGGRAPH Asia 2019](https://www.youtube.com/watch?v=K_7TkoIkFhk)
- [ISF JSON reference](https://docs.isf.video/ref_json.html)
- [ISF multipass and persistent buffers](https://docs.isf.video/ref_multipass.html)
- [ISF variables](https://docs.isf.video/ref_variables.html)
- [Epic, Fluid Simulation in Unreal Engine overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/fluid-simulation-in-unreal-engine---overview)
- [Godot docs, compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)
- [Godot docs, Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html)
- [Godot docs, Viewport](https://docs.godotengine.org/en/stable/classes/class_viewport.html)
- [Godot docs, SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)
