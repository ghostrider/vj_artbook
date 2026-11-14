# Stable Fluids Ink and Smoke — research

## Origin
Jos Stam published the "Stable Fluids" method in 1999. It combines semi-Lagrangian advection with implicit viscosity, which gives unconditionally stable behaviour and allows large time steps ([Wikipedia: Fluid animation](https://en.wikipedia.org/wiki/Fluid_animation)). Stam worked at Alias|wavefront (later Autodesk) and received a 2008 Academy Award for technical achievement for the Maya Fluid Effects system ([Wikipedia: Jos Stam](https://en.wikipedia.org/wiki/Jos_Stam)).

Mark J. Harris moved the method to the GPU in "Fast Fluid Dynamics Simulation on the GPU" (GPU Gems, 2004), explicitly based on Stam 1999 ([NVIDIA GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)). Vorticity confinement, the add-on that keeps small swirls alive, comes from Fedkiw et al. (2001) and is not part of Stam's original.

## Key works and artists
- **WebGL Fluid Simulation** by Pavel Dobryakov: a browser GPU fluid sim (works on mobile), MIT licence, built on the GPU Gems chapter plus two open-source fluid projects. The reference for the "coloured ink splats" look ([GitHub](https://github.com/PavelDoGreat/WebGL-Fluid-Simulation)).
- **Flow** by Karl Sims (2018): installation shown at MIT CSAIL, the National Museum of Mathematics and Boston Children's Hospital. A Kinect tracks visitors, whose gestures drive fluid, particle and image-processing effects. Modes include virtual ink squirting from the hands, layered dense/light fluids and a lava lamp ([karlsims.com/flow](https://karlsims.com/flow.html)).

## How it works
Per frame, on a 2D grid of velocity and dye (GPU textures) ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)):
1. **Advection**: every cell looks backward along the velocity to where its content came from and copies it. Stable even at large steps.
2. **Diffusion** (viscosity): solved implicitly, usually with Jacobi iterations.
3. **Forces**: mouse, MIDI or audio add Gaussian "splats" to velocity (and colour to the dye).
4. **Projection**: solve a Poisson equation for pressure (typically 40 to 80 Jacobi iterations) and subtract its gradient, so the flow is divergence-free (incompressible, so it swirls instead of piling up).
5. **Vorticity confinement** (optional): compute curl and add a force that re-injects small-scale swirls lost to numerical damping.

## Visual language
Soft, saturated coloured plumes that curl into mushroom-like vortices and thin filaments, like ink in water or smoke in still air. Motion is continuous and organic, with a clear cause (each splat is a gesture) and a slow fade. Dense colour on black looks strongest. Karl Sims' modes show the range: ink from the hands, lava-lamp blobs, paint-like swirls, ripples reflecting off the screen edges ([karlsims.com/flow](https://karlsims.com/flow.html)).

## Live potential
- **Kick**: a radial or directional velocity impulse at a fixed point (a "punch" of smoke); bass: large, slow, wide splats with low-frequency colour.
- **Hi-hats**: small, fast, thin splats at random edges, or a brief vorticity spike for fine curls.
- **BPM**: place splat emitters on a clock (circle of N emitters triggered per beat); rotate a constant force field.
- **MIDI faders/keys**: viscosity, dissipation (trail length), vorticity strength, splat radius, hue; KeyStep notes map to emitter positions and colours.
- **Layering**: use the dye or velocity field as a displacement map for other clips, or output alpha-friendly smoke over geometry. Feed it a video mask as an obstacle or as a dye source.
- **Boring risks**: a screen that fills with a muddy mix of colours; no dissipation means grey soup. Fix with limited palettes, strong dissipation, clear-on-bar resets and contrast with empty black space.
- Tooling: the same pipeline exists as a TouchDesigner tutorial ([threedashes, YouTube](https://www.youtube.com/watch?v=2k6H5Qa_fCE)), so Spout output into Resolume is practical.

## Variations
- **Tempo-locked emitters**: a ring of splat points firing on a beat grid, so the fluid becomes a rhythm score.
- **Obstacle dance**: moving shapes (logo, typography, video silhouette) as solid boundaries that split the flow.
- **Palette lock**: dye colour from a two- or three-colour scheme with hue shifts on a fader, avoiding rainbow mud.
- **Stable-fluids displacement**: use the velocity field to warp other layers (video, type) instead of showing dye.
- **Gravity / buoyancy mode**: heat or density-driven plumes (smoke rising, ink sinking) toggled by a switch, like the layered-fluid modes in Karl Sims' Flow.

## Facts
- Stam's scheme is stable for any time step, which is why it is the standard for real-time fluids ([Wikipedia: Fluid animation](https://en.wikipedia.org/wiki/Fluid_animation)).
- Backward tracing was chosen over forward-moving particles because it is stable and fits fragment programs on the GPU ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)).
- The pressure solve is the main cost: 40 to 80 Jacobi iterations per frame is typical ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)).
- Vorticity confinement (Fedkiw et al., 2001) is a later addition that restores swirl detail ([GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)).
- Dobryakov's browser sim is MIT licensed and runs on mobile ([GitHub](https://github.com/PavelDoGreat/WebGL-Fluid-Simulation)).
- Karl Sims' Flow runs in C++/OpenCL/OpenGL on a Linux PC with an NVIDIA card ([karlsims.com/flow](https://karlsims.com/flow.html)).

## Visual references
- [WebGL Fluid Simulation repository](https://github.com/PavelDoGreat/WebGL-Fluid-Simulation) — README with screenshot and link to the live browser demo
- [Karl Sims: Flow](https://karlsims.com/flow.html) — installation page describing ink, lava-lamp and paint-swirl modes
- [Navier-Stokes (Stable Fluids, Jos Stam), Encre](https://www.youtube.com/watch?v=mWj9rx5FC9g) — a Stable Fluids ink simulation; channel Spectron
- [TouchDesigner _06 Fluid Simulation](https://www.youtube.com/watch?v=2k6H5Qa_fCE) — TouchDesigner tutorial building a splat/curl/vorticity fluid; channel threedashes

## Lead image
- file_page: https://commons.wikimedia.org/wiki/File:Vortex-street-1.jpg
- image_url: https://upload.wikimedia.org/wikipedia/commons/2/2c/Vortex-street-1.jpg
- shows: Landsat 7 view (1999) of clouds forming a von Kármán vortex street behind an island, the paired swirling vortices a stable-fluids sim produces behind obstacles
- author: Robert Cahalan, NASA/GSFC
- license: Public domain (NASA)
- license_url: https://commons.wikimedia.org/wiki/Template:PD-USGov

## Sources
- [Wikipedia: Fluid animation](https://en.wikipedia.org/wiki/Fluid_animation)
- [Wikipedia: Jos Stam](https://en.wikipedia.org/wiki/Jos_Stam)
- [NVIDIA GPU Gems ch. 38](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu)
- [PavelDoGreat/WebGL-Fluid-Simulation](https://github.com/PavelDoGreat/WebGL-Fluid-Simulation)
- [Karl Sims: Flow](https://karlsims.com/flow.html)
- [Commons: Vortex-street-1.jpg](https://commons.wikimedia.org/wiki/File:Vortex-street-1.jpg)
