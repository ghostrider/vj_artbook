---
tags:
  - fluids
  - simulation
  - shader
  - flow
---

# Stable Fluids Ink and Smoke

> Black water, three inks, and a plume of colour that rolls up into a vortex every time the kick lands.

![Landsat 7 view of clouds forming a von Karman vortex street behind an island; a real-world example of the paired vortices a stable-fluids sim produces behind obstacles](https://upload.wikimedia.org/wikipedia/commons/2/2c/Vortex-street-1.jpg)
*A real cloud vortex street behind an island, seen by Landsat 7 in 1999. Not a simulation: the same paired swirls appear behind obstacles in a stable-fluids sim. Image: [Robert Cahalan, NASA/GSFC](https://commons.wikimedia.org/wiki/File:Vortex-street-1.jpg), [Public domain (NASA)](https://commons.wikimedia.org/wiki/Template:PD-USGov).*

## The idea

A 2D grid holds a velocity for every cell and a dye that rides along on it. Each frame the sim pushes the flow with forces, makes it incompressible, and moves the dye. Incompressible matters: the flow swirls instead of piling up. The result looks like ink in water or smoke in still air.

The method is Jos Stam's "Stable Fluids" (1999). It traces each cell backward along the velocity to find where its content came from. This is stable even at large time steps, which is why it became the standard for real-time fluids ([Wikipedia: Fluid animation](https://en.wikipedia.org/wiki/Fluid_animation)). Mark J. Harris moved it to the GPU in 2004 ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)).

On a club screen it works because every swirl has a visible cause. A hit goes in, a mushroom comes out, and it fades slowly. The motion is continuous and organic, and it reads from the back of the room. The risk is mud: a wall that fills with mixed colour. This chapter answers it with three inks, strong dissipation and plenty of empty black.

## Origins and references

- Jos Stam, "Stable Fluids" (1999): semi-Lagrangian advection with implicit viscosity. Stam later received a 2008 Academy Award for technical achievement for the Maya Fluid Effects system ([Wikipedia: Jos Stam](https://en.wikipedia.org/wiki/Jos_Stam)).
- Mark J. Harris, "Fast Fluid Dynamics Simulation on the GPU" (GPU Gems, 2004): the pass order, Jacobi counts and boundaries ([NVIDIA GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)).
- Fedkiw et al. (2001): vorticity confinement, which keeps small swirls alive. It is not part of Stam's original.
- Pavel Dobryakov, WebGL Fluid Simulation: a browser GPU sim, MIT licence, the reference for the coloured-ink-splat look ([GitHub](https://github.com/PavelDoGreat/WebGL-Fluid-Simulation)).
- Karl Sims, *Flow* (2018): an installation where a Kinect tracks visitors and their gestures drive fluid. Modes include ink from the hands, layered fluids and a lava lamp ([karlsims.com/flow](https://karlsims.com/flow.html)).

## Visual language

Soft, saturated plumes on black. Each hit rolls up into a mushroom-shaped vortex pair, sheds thin curling filaments and fades. Only three inks are used: cobalt, hot magenta and amber, plus a warm white where they pile up. Where they cross they layer like stained glass. A soft relief light from the top left makes the plumes look thick and wet. A light bloom makes dense cores glow.

Rhythm comes from the emitters. Eight points sit on a ring. Each kick fires one of them inward. Hi-hats twist the edges into fine curls. On the drop all eight fire at once and collide in the centre. On a break the ink rises slowly like smoke and thins until the wall is almost empty. Keep at least 40 % of the wall dark.

| role | hex | use |
|------|-----|-----|
| base / black | #000000 | background, dye = 0 |
| ink A | #1F4BFF | cobalt, bass and kick ink, most of the volume |
| ink B | #FF2D8A | hot magenta, snare and the drop |
| ink C | #FFA21F | amber, hats and highlights |
| highlight | #FFF1D6 | where total ink density exceeds about 3 |

## See it

- [WebGL Fluid Simulation repository](https://github.com/PavelDoGreat/WebGL-Fluid-Simulation) — README with screenshot and link to the live browser demo
- [Karl Sims: Flow](https://karlsims.com/flow.html) — installation page describing ink, lava-lamp and paint-swirl modes
- [Navier-Stokes (Stable Fluids, Jos Stam), Encre](https://www.youtube.com/watch?v=mWj9rx5FC9g) — a Stable Fluids ink simulation; channel Spectron
- [TouchDesigner _06 Fluid Simulation](https://www.youtube.com/watch?v=2k6H5Qa_fCE) — TouchDesigner tutorial building a splat/curl/vorticity fluid; channel threedashes

## Code design

The dye stores ink amounts, not colours. A palette matrix in the display pass turns R, G, B into the three inks, so palettes change instantly without re-simulating. Sim buffers must be float: signed velocities and pressure do not survive 8 bits.

**Pass order per frame** (Dobryakov's step order, with forces first):

1. S1 forces: Gaussian velocity splats from the emitters, plus buoyancy.
2. S2 curl.
3. S3 vorticity confinement.
4. S4 divergence.
5. S5 pressure warm start: `p = 0.8 · p` instead of clearing.
6. S6 Jacobi, N times (24 default, keep N even so the result lands in P_A).
7. S7 gradient subtract, with free-slip walls.
8. S8 advect velocity, with drag.
9. S9 advect dye, add dye splats, apply trail dissipation.
10. S10 display: palette matrix, tone curve, relief shading.
11. S11 post: bloom, black crush, grain.

**Buffers** for a 1920x1080 output:

| buffer | size | format | filter | persistent |
|--------|------|--------|--------|------------|
| VEL (ping-pong V0/V1) | 320x180 | RG16F (RGBA16F if RG is not renderable) | linear | yes |
| CURL | 320x180 | R16F | nearest | no |
| DIV | 320x180 | R16F | nearest | no |
| P (ping-pong P_A/P_B) | 320x180 | R16F | nearest | yes (warm start) |
| DYE (ping-pong D0/D1) | 1280x720 | RGBA16F | linear | yes |
| OUT | output | RGBA16F before bloom, RGBA8 after | none | no |

Rules that matter: clamp dt to 0 to 1/30 s so a dropped frame cannot throw ink across the wall. Dissipation lives in the advection pass as `result = sample / (1 + d · dt)`. Do not add a host feedback effect on top of the sim, because it doubles the smearing.

**Licence note.** Dobryakov's sim is MIT licensed ([GitHub](https://github.com/PavelDoGreat/WebGL-Fluid-Simulation)). This design only restates the method and his default values. If a sketch ports his GLSL directly, for example into an ISF file, it must keep his copyright notice and the MIT licence text in the file header.

=== "Resolume Wire"
    The sim runs as one ISF multipass effect, either inside Wire's ISF node or on a layer in Arena. ISF passes can set `TARGET`, `PERSISTENT`, `FLOAT` and sizes as expressions of `$WIDTH`/`$HEIGHT`. Persistent buffers are resized, and so reset, when the render size changes ([ISF JSON reference](https://docs.isf.video/ref_json.html), [ISF multipass](https://docs.isf.video/ref_multipass.html)). The shader branches on `PASSINDEX`, `FRAMEINDEX == 0` triggers the init and `TIMEDELTA` gives dt ([ISF variables](https://docs.isf.video/ref_variables.html)).

    Pass list (N = 20, 29 passes):

    - 0 forces to `velF`
    - 1 curl to `curlB`
    - 2 vorticity to `velV`
    - 3 divergence to `divB`
    - 4 warm start from persistent `pA` to `pB`
    - 5 to 24 Jacobi, alternating `pB` to `pA`, `pA` to `pB`; an even N ends in `pA`
    - 25 gradient subtract to `velP`
    - 26 advect `velP` to persistent `vel`
    - 27 copy persistent `dye` to `dyeOld`
    - 28 advect `dyeOld` plus dye splats to persistent `dye`
    - display pass with no target

    Every sim buffer needs `FLOAT: true`. Velocity buffers use `$WIDTH/6` by `$HEIGHT/6`, dye buffers `$WIDTH/1.5` by `$HEIGHT/1.5`. The dye copy (pass 27) avoids relying on reading neighbours from a persistent buffer that is being written, which the ISF pages do not specify. ISF has no loops over passes, so make 20, 32 and 40 variants.

    Inputs: floats `punch`, `trail`, `swirl`, `drag`, `buoyancy`, `ink`, `beatPhase` (0 to 1), a `beatCount`, a `point2D` hand splat, `event` reset and flush, and `color` for the three inks. The force envelope is `exp(-beatPhase · 12)`, so each beat gives a short push with no extra state. Drive `beatPhase` with a BPM-synced sawtooth.

    **Pre-gig 8-bit buffer test.** Resolume documents ISF only in its Wire pages ([Resolume ISF](https://resolume.com/support/en/isf)), and multipass with persistent and FLOAT buffers depends on the version. Fire one splat with drag 0.05. If the motion stops dead, shows stair-steps, or only moves in one direction (negative values clipped), the buffers are 8-bit. Do not try to rescue it with 8-bit packing at these pass counts. Switch to an external sim sent in by Spout (the Unreal or Godot tabs, or TouchDesigner, where the same pipeline is shown in a tutorial by [threedashes](https://www.youtube.com/watch?v=2k6H5Qa_fCE)).

    In Wire there are no vertex shaders (as of version 7.22), which this design does not need ([Resolume ISF](https://resolume.com/support/en/isf)). Node outline: Effect input (optional obstacle or video) to Threshold / luma to Resize to velocity-grid size to the ISF node to Output. Blur the input by 2 % before the threshold so silhouette edges do not create pressure noise. Control side: BPM clock to phase to `beatPhase`; beat counter to `beatCount`; Low band envelope (attack 10 ms, release 200 ms) to `punch`; High band envelope (5 ms / 120 ms) added to `swirl`; bar counter mod 4 to emitter mode. Node names differ between Wire versions, so check the node browser.

=== "Unreal Engine 5.7"
    Route A, render-target ping-pong, is the deterministic choice for this exact look.

    - Render targets (mips off, clamp addressing): `RT_V0/RT_V1` 320x180 RG16f, linear; `RT_Curl`, `RT_Div` 320x180 R16f, nearest; `RT_P0/RT_P1` 320x180 R16f, nearest; `RT_D0/RT_D1` 1280x720 RGBA16f, linear; `RT_Out` 1920x1080 RGBA8.
    - One Unlit material per pass: `M_FluidForces`, `M_Curl`, `M_Vorticity`, `M_Divergence`, `M_PressureWarm`, `M_Jacobi`, `M_GradSub`, `M_Advect` (used for velocity and dye with a `Dissipation` parameter), `M_Display`.
    - A Blueprint on Tick calls Draw Material to Render Target for S1 to S10 in order and swaps the RT references after each write. S6 is a For Loop with N iterations. Reset is Clear Render Target 2D on all sim RTs.
    - Emitters: the Blueprint keeps up to 8 splats (position, direction, ink) per frame and writes them to the MID as `Splat0` to `Splat7` and `SplatInk0` to `SplatInk7`.

    Route B, Niagara Fluids 2D gas. Niagara Fluids store fields in Grid 2D Collections and run the solver as simulation stages that can repeat with Num Iterations ([Epic, fluid simulation overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/fluid-simulation-in-unreal-engine---overview)). Start from a 2D gas template, replace the source with beat emitters and render density into a render target. It starts faster, but the look is less controllable.

    Output `RT_Out` by Spout or NDI into Arena, or show it on a full-screen plane. Remote Control: a Material Parameter Collection `MPC_Fluid` (`Punch`, `Trail`, `Swirl`, `Drag`, `Buoyancy`, `Ink`, `InkA/B/C`) plus Blueprint functions `Reset`, `Flush`, `FireAll` in a Remote Control Preset, bound to OSC and MIDI.

=== "GODOT"
    Godot 4.x with the Forward+ renderer. Compute shaders need a RenderingDevice-based renderer (Forward+ or Mobile), and mobile drivers are unreliable ([Godot docs, compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)).

    - State: the buffers from the table above, created on the main RenderingDevice as storage images in 16-bit float formats. Advection reads through a linear sampler and writes with image stores.
    - One compute shader per pass, or one shader with a `pass` push constant, in 8x8 workgroups: 40x23 groups for the velocity grid, 160x90 for the dye grid. The script records S1 to S9 each frame with N Jacobi dispatches, swapping bindings after each write.
    - Do not call `sync()` every frame. The docs recommend waiting 2 to 3 frames, and splitting long work into several dispatches avoids a Windows TDR ([Godot docs, compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)).
    - Display: wrap the dye image in a `Texture2DRD` via `texture_rd_rid` ([Godot docs, Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html)) and draw it on a full-screen ColorRect with a `canvas_item` shader that does S10. Bloom comes from a WorldEnvironment glow, or from a SubViewport with `use_hdr_2d = true` ([Godot docs, Viewport](https://docs.godotengine.org/en/stable/classes/class_viewport.html)).
    - There is no clean Compatibility fallback. The pressure solve needs 20+ ordered passes per frame, and a SubViewport chain (`UPDATE_ONCE`, `CLEAR_MODE_NEVER`, see [Godot docs, SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)) gives one ordered step per frame.
    - Expose `punch`, `trail`, `swirl`, `drag`, `buoyancy`, `ink`, `ink_a/b/c`, `exposure`, `relief`, `warm`, `pressure_iterations` and the splat array. Audio comes from AudioEffectSpectrumAnalyzer. Send to Arena by Spout GDExtension if available, otherwise NDI.

## On stage

**Audio reactivity**

| band | drives | smoothing |
|------|--------|-----------|
| kick, 40-120 Hz | fires the next ring emitter (ink A) | attack 10 ms, release 200 ms; trigger at 0.6 of the running peak, retrigger blocked 120 ms |
| bass, 60-250 Hz | splat radius x 1 to 2, drag -30 % | attack 80 ms, release 800 ms |
| snare / clap, 1-4 kHz | a mirrored emitter pair with ink B | attack 5 ms, release 150 ms |
| hats, 6-12 kHz | swirl +15, tiny ink-C splats at random edges, max 4 per beat | attack 5 ms, release 120 ms |
| RMS loudness | trail from 1.5 (quiet) to 0.6 (loud) | attack 500 ms, release 4 s |

Do not modulate pressure iterations or grid size with audio: buffers reallocate or the solve jitters. Never drive buoyancy from fast audio, or the whole field lurches.

**Mapping.** BeatStep Pro knobs 1 to 5: punch, trail, swirl, drag, ink (palette rotation, with an A/B swap above 0.5). KeyStep mod strip: buoyancy, or force for hand-painted smoke. KeyStep notes: pitch sets the x position of a hand splat. Pads: reset, flush (trail 8 for 0.5 s), fire-all, emitter mode (inward, tangential, all-fire).

**Arrangement at 128 BPM.**

- Groove: each beat one ring emitter fires inward with ink A. The index advances by 3 each beat, so hits spread around the ring. Every downbeat uses ink B. A hit applies its force over 3 frames for rounder mushrooms.
- Every 4 bars: tangential emitters for one bar, the whole field spins. Every 8 bars: rotate the ink hues by up to 25 degrees.
- Build-up: swirl 30 to 55, trail 1.0 to 0.5, emitter rate doubles in the last bar.
- Drop: all 8 emitters fire inward, force 3.5, inks alternating A/B. For 4 bars use radius x 1.5 and drag 0.1.
- Break: buoyancy to +40, grey smoke palette, trail 2.0, one emitter every 2 bars from the bottom edge.

**Layering.** Output alpha as `1 - exp(-(R+G+B) · 1.5)` with premultiplied colour and put the layer in Normal over footage. Or use total ink density as a luma matte so the video only shows inside the plumes.

**Performance.** At 320x180 velocity and 1280x720 dye with 24 iterations, the sim is well under 1 ms on a current desktop GPU. The pressure solve is the main sim cost ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)). In ISF the cost is host overhead per pass, so keep N at 20. Raise dye resolution before velocity resolution. For a 5760x1080 strip use 960x180 velocity and 3840x720 dye, and 32 to 40 iterations. Pitfalls: 8-bit buffers, odd Jacobi counts, a missing dt clamp, no dissipation, too many inks, mipmaps on sim textures.

## Variations

- **Tempo-locked emitters**: a ring of splat points on a beat grid, so the fluid becomes a rhythm score.
- **Obstacle dance**: a logo, type or video silhouette as solid boundary. The flow splits around it. Expect about +20 % on the velocity passes, since arbitrary boundaries cost extra per-cell branching ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)).
- **Palette lock**: two or three inks with hue shifts on a fader, no rainbow mud.
- **Stable-fluids displacement**: use the velocity field to warp other layers (video, type) instead of showing dye.
- **Gravity and buoyancy**: smoke rising or ink sinking on a switch. With ink A rising and ink C sinking, the plumes separate like the lava-lamp mode in Karl Sims' *Flow* ([karlsims.com/flow](https://karlsims.com/flow.html)).
- **Wrap mode**: repeat addressing in x for 360-degree rooms and LED rings.

## Did you know?

- Stam's scheme is stable for any time step ([Wikipedia: Fluid animation](https://en.wikipedia.org/wiki/Fluid_animation)).
- Backward tracing was chosen over forward-moving particles because it is stable and fits fragment programs on the GPU ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)).
- The pressure solve typically takes 40 to 80 Jacobi iterations per frame ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)).
- Karl Sims' Flow runs in C++, OpenCL and OpenGL on a Linux PC with an NVIDIA card ([karlsims.com/flow](https://karlsims.com/flow.html)).

## Sources

- [Wikipedia: Fluid animation](https://en.wikipedia.org/wiki/Fluid_animation)
- [Wikipedia: Jos Stam](https://en.wikipedia.org/wiki/Jos_Stam)
- [NVIDIA GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)
- [PavelDoGreat/WebGL-Fluid-Simulation](https://github.com/PavelDoGreat/WebGL-Fluid-Simulation) ([script.js](https://raw.githubusercontent.com/PavelDoGreat/WebGL-Fluid-Simulation/master/script.js), [live demo](https://paveldogreat.github.io/WebGL-Fluid-Simulation/))
- [Karl Sims: Flow](https://karlsims.com/flow.html)
- [Navier-Stokes (Stable Fluids, Jos Stam), Encre](https://www.youtube.com/watch?v=mWj9rx5FC9g)
- [TouchDesigner _06 Fluid Simulation](https://www.youtube.com/watch?v=2k6H5Qa_fCE)
- [Commons: Vortex-street-1.jpg](https://commons.wikimedia.org/wiki/File:Vortex-street-1.jpg)
- [Resolume ISF](https://resolume.com/support/en/isf)
- [ISF JSON reference](https://docs.isf.video/ref_json.html), [ISF multipass](https://docs.isf.video/ref_multipass.html), [ISF variables](https://docs.isf.video/ref_variables.html)
- [Epic, Fluid Simulation in Unreal Engine overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/fluid-simulation-in-unreal-engine---overview)
- [Godot docs, compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)
- [Godot docs, Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html)
- [Godot docs, Viewport](https://docs.godotengine.org/en/stable/classes/class_viewport.html)
- [Godot docs, SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)
