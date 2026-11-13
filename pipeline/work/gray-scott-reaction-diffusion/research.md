# Gray-Scott Reaction-Diffusion — research

## Origin
- Alan Turing's 1952 paper "The chemical basis of morphogenesis" proposed that a state stable in a well-mixed system can become unstable once diffusion is added, which gives patterns ([Wikipedia](https://en.wikipedia.org/wiki/Reaction%E2%80%93diffusion_system)).
- The Gray-Scott model comes from P. Gray and S. K. Scott's 1984 paper on autocatalytic reactions in a stirred tank reactor. They called the k term "decay" or "poisoning"; later authors say "kill" ([mrob.com xmorphia](https://mrob.com/pub/comp/xmorphia)).
- John Pearson's 1993 paper "Complex patterns in a simple system" began a classification of the patterns the model makes across f and k ([mrob.com xmorphia](https://mrob.com/pub/comp/xmorphia)). Lee et al. reported self-replicating spots experimentally in *Nature* in 1994 ([Wikipedia](https://en.wikipedia.org/wiki/Reaction%E2%80%93diffusion_system)).
- Karl Sims' page is the standard practical introduction for graphics people ([Karl Sims](https://www.karlsims.com/rd.html)).

## Key works and artists
- Karl Sims: the reference explainer with ready parameters and extensions (orientation, style map, flow, scale) ([karlsims.com](https://www.karlsims.com/rd.html)).
- Robert Munafo, "xmorphia": a map of the Gray-Scott parameter space after Pearson ([mrob.com](https://mrob.com/pub/comp/xmorphia)).
- Simon Gladman, Houdini gallery entry (Nov 2016), Gray-Scott solved in VEX in Houdini 15.5 ([SideFX](https://www.sidefx.com/gallery/reaction-diffusion-in-houdini/)).
- Hamy, "C10ud": an audio-driven Gray-Scott music visual. Beats found offline fire a handler that inserts new pools of B ([hamy.xyz](https://hamy.xyz/blog/building-a-reaction-diffusion-visualizer-in-p5js)).
- Pao Olea, two-part TouchDesigner tutorial (April 2025) that builds reaction-diffusion and then adds audio reactivity ([Derivative](https://derivative.ca/community-post/tutorial/reaction-diffusion-touchdesigner-tutorial-19/71701)).

## How it works
- Each cell holds two amounts, A and B. Every step: both spread to neighbours (A faster than B), then the reaction runs where two Bs turn one A into B. A is fed in at rate f, B is removed at rate k ([Karl Sims](https://www.karlsims.com/rd.html)).
- Sims' values: D_A = 1.0, D_B = 0.5, dt = 1.0, Laplacian as a 3x3 kernel (centre -1, edges 0.2, corners 0.05). Start with A = 1 everywhere and a small seeded blob of B ([Karl Sims](https://www.karlsims.com/rd.html)).
- Why patterns appear: convex B edges get A from outside and grow, concave edges starve and shrink. A spot can grow and split like a cell ([Karl Sims](https://www.karlsims.com/rd.html)).
- Presets from Sims: "mitosis" f=0.0367, k=0.0649; "coral growth" f=0.0545, k=0.062. Interesting region: k 0.045 to 0.07, f 0.01 to 0.1 ([Karl Sims](https://www.karlsims.com/rd.html)).

## Visual language
- Organic labyrinths, worms, coral, leopard spots, splitting cells, and travelling waves. Output is soft-edged, high-contrast blobs that grow, divide and settle.
- Colour is free: the simulation gives only a scalar field B. Map it through gradients, normals or lighting for a gel or liquid look.
- Simulation color mode can run three sims in parallel, one per RGB channel ([piellardj WebGL](https://github.com/piellardj/reaction-diffusion-webgl)).
- Risk: left alone it settles into a static pattern, which is dull on a big screen.

## Live potential
- Feed f and kill k on two faders. They move the look between spots, stripes and mitosis. Sims says f and k can vary across the grid with a style map ([Karl Sims](https://www.karlsims.com/rd.html)), so a texture or video can be the map.
- Kick: inject B blobs at a point, as C10ud does on each beat ([hamy.xyz](https://hamy.xyz/blog/building-a-reaction-diffusion-visualizer-in-p5js)). Hi-hats: small random seeds or a small nudge to k.
- Bass: slow LFO on f/k, or a scale change (Sims' scale option) to zoom pattern size.
- BPM: reseed or flip between two preset (f,k) pairs on the bar.
- Mouse/MIDI position can act as a brush that paints B.
- Layering: use the B field as a mask or displacement map over video, or multiply with other layers in Resolume via Spout/NDI.
- Performance: C10ud's CPU version needed up to an hour per second of video in some scenes, so it could not run live ([hamy.xyz](https://hamy.xyz/blog/building-a-reaction-diffusion-visualizer-in-p5js)). A GPU shader is required. A real-time WebGL version exists ([piellardj](https://github.com/piellardj/reaction-diffusion-webgl)).
- Boring if static: always keep something injecting or changing parameters. Several sim steps per frame are needed for speed.

## Variations
- Image-driven f/k: video or camera brightness sets the local f/k, so the sim approximates the picture ([piellardj](https://github.com/piellardj/reaction-diffusion-webgl) does this with images).
- Directional diffusion (Sims' orientation option) for combed, wood-grain or fingerprint textures ([Karl Sims](https://www.karlsims.com/rd.html)).
- Flow: advect chemicals with a noise or video-optical-flow field ([Karl Sims](https://www.karlsims.com/rd.html)).
- Three coupled sims, one per colour channel, with different f/k each.
- Run the sim on a mesh surface (the Houdini route, [SideFX](https://www.sidefx.com/gallery/reaction-diffusion-in-houdini/)) or as a UE5 Niagara grid, then light it as bump/normal map.

## Facts
- Pearson's 1993 map and Munafo's xmorphia site show a whole zoo of patterns from just two parameters ([mrob.com](https://mrob.com/pub/comp/xmorphia)).
- Gray and Scott's original system was a chemical stirred tank, with F meaning feed or flow of pure U ([mrob.com](https://mrob.com/pub/comp/xmorphia)).
- Pearson's colours were matched to the pH indicator bromothymol blue used in experiments ([mrob.com](https://mrob.com/pub/comp/xmorphia)).
- Self-replicating spots were seen in a real chemical experiment, reported in 1994 ([Wikipedia](https://en.wikipedia.org/wiki/Reaction%E2%80%93diffusion_system)).
- WebGL 1 has no 16-bit float texture, so one browser version packs each value into two 8-bit channels ([piellardj](https://github.com/piellardj/reaction-diffusion-webgl)).

## Visual references
- [Karl Sims: Reaction-Diffusion Tutorial](https://www.karlsims.com/rd.html) — explainer with example pattern images and parameter map.
- [Munafo xmorphia](https://mrob.com/pub/comp/xmorphia) — Gray-Scott parameter-space atlas (Pearson).
- [piellardj reaction-diffusion-webgl](https://github.com/piellardj/reaction-diffusion-webgl) — real-time GPU sim with live demo link, uniform and image-driven modes.
- [SideFX gallery: Reaction Diffusion in Houdini](https://www.sidefx.com/gallery/reaction-diffusion-in-houdini/) — Simon Gladman's animated Gray-Scott in VEX.
- [Coding Challenge #13: Reaction Diffusion Algorithm in p5.js](https://www.youtube.com/watch?v=BV9ny785UNc) — video tutorial, channel The Coding Train; builds the sim live.
- [Fun with Reaction-Diffusion in Houdini, Dan Wills, SIGGRAPH Asia 2019](https://www.youtube.com/watch?v=K_7TkoIkFhk) — talk, channel Houdini; Gray-Scott extended in several ways.

## Sources
- [Karl Sims, Reaction-Diffusion Tutorial](https://www.karlsims.com/rd.html)
- [Wikipedia, Reaction-diffusion system](https://en.wikipedia.org/wiki/Reaction%E2%80%93diffusion_system)
- [Munafo, xmorphia](https://mrob.com/pub/comp/xmorphia)
- [Hamy, Building a Reaction Diffusion Audio Visualizer](https://hamy.xyz/blog/building-a-reaction-diffusion-visualizer-in-p5js)
- [Derivative, Reaction Diffusion TouchDesigner tutorial](https://derivative.ca/community-post/tutorial/reaction-diffusion-touchdesigner-tutorial-19/71701)
- [SideFX gallery](https://www.sidefx.com/gallery/reaction-diffusion-in-houdini/)
- [piellardj WebGL](https://github.com/piellardj/reaction-diffusion-webgl)
- [The Coding Train challenge 13](https://thecodingtrain.com/challenges/13-reaction-diffusion)
