# Clifford Strange Attractors — research

## Origin
The Clifford attractor is a two-dimensional iterated map attributed to Cliff Pickover. Paul Bourke's page (February 2004) gives the graphics, and source code from Paul Richards ([Bourke](https://paulbourke.net/fractals/clifford/)). It belongs to a family with the Peter de Jong attractor, published in Scientific American "Computer Recreations", July 1987 ([Bourke, de Jong](https://paulbourke.net/fractals/peterdejong/)). Bourke has also extended the sine/cosine idea to 3D for VR viewing (April 2021) ([Bourke, 3D](https://paulbourke.net/fractals/attract3d/index.html)).

## Key works and artists
- **Paul Bourke, Clifford and de Jong pages.** Reference equations, parameter sets and the density-histogram rendering method ([Clifford](https://paulbourke.net/fractals/clifford/)).
- **Ricky Reusser, "Clifford and de Jong Attractors" (2025).** Interactive WebGL notebook: a grid of points is iterated and accumulated on a texture as a histogram. Colour comes from the distance of the last step, and lightness from the log of the hit count ([notebook](https://rreusser.github.io/notebooks/clifford-and-de-jong-attractors)).
- **Dirk Brockmann, "Stranger Things" (2018).** Explorable with sliders for Clifford, Bedhead, Fractal Dream and Gumowski-Mira maps. It credits Mike Bostock's Observable Clifford piece as inspiration ([explorable](https://www.complexity-explorables.org/explorables/stranger-things/)).
- **Bourke, Attractors in 3D for VR.** Roughly 1,000 to 10,000 million points are binned into 2D equirectangular histograms ([page](https://paulbourke.net/fractals/attract3d/index.html)).
- **orbita2d/clifford.** Python frame renderer that outputs animated GIF/MP4 sequences ([repo](https://github.com/orbita2d/clifford)).

## How it works
Two equations are iterated: x' = sin(a·y) + c·cos(a·x), y' = sin(b·x) + d·cos(b·y). The four parameters a, b, c, d define each attractor ([Bourke](https://paulbourke.net/fractals/clifford/)). Plotting single points looks thin. Instead each pass through a pixel increments a counter on a large grid, which gives a 2D occupancy histogram. Colour is applied afterwards through a gradient map. Many more iterations than expected are needed for smooth gradients ([Bourke](https://paulbourke.net/fractals/clifford/)). The map is only 2D, although it often looks like a 3D structure projected flat ([Bourke, 3D](https://paulbourke.net/fractals/attract3d/index.html)). Since the pattern is a long-run orbit, animation is done by nudging parameters in tiny steps and rendering each shape as a frame ([Sekara](https://vedransekara.github.io/2019/10/01/temporal_attractors.html)).

## Visual language
Silky, smoky filaments and veils that fold into ribbons and swirls, with sharp bright caustic-like edges where density piles up. Log-density mapping gives soft glow. Colour by step length or by hit count gives luminous gradients (see Reusser's method). Parameter sets such as a=-1.4, b=1.6, c=1.0, d=0.7 are known good starting points ([Bourke](https://paulbourke.net/fractals/clifford/)). Motion is a continuous morph: forms stretch, fold and collapse into new shapes.

## Live potential
(Design suggestions, not sourced claims.)
- Four faders (MIDI CC) map directly onto a, b, c, d. Slow LFOs give flowing morphs. A BPM-synced jump between two stored parameter sets gives a hard cut.
- Kick: short pulse added to one parameter, or a brightness/exposure flash. Bass: slow drift of a and b. Hi-hats: jitter in point size or colour hue.
- The GPU-histogram technique suits Resolume via Spout/NDI, or a Niagara/TouchDesigner compute pass. Output is a monochrome density field, so it layers well with a gradient map and additive blend.
- Boring risks: a static attractor on screen, or a morph so large that the shape collapses to a point or a line. Keep parameter excursions small and use stored presets for big changes.

## Variations
1. Clifford to de Jong crossfade by blending the two update rules ([de Jong](https://paulbourke.net/fractals/peterdejong/)).
2. Closed-loop parameter paths (a circle in 4D parameter space) for seamless loops ([Sekara](https://vedransekara.github.io/2019/10/01/temporal_attractors.html)).
3. Colour by velocity or curvature (Bourke notes the increment can be a function of curvature) ([Bourke](https://paulbourke.net/fractals/clifford/)).
4. Feedback: feed the density texture back with a slow decay and a slight zoom, for trails.
5. 3D extension projected onto a rotating camera or a dome, following Bourke's VR work ([page](https://paulbourke.net/fractals/attract3d/index.html)).

## Facts
- Parameters from a typical set: a=-1.4, b=1.6, c=1.0, d=0.7 ([Bourke](https://paulbourke.net/fractals/clifford/)).
- Bourke's VR renders bin 1,000 to 10,000 million points per image ([Bourke](https://paulbourke.net/fractals/attract3d/index.html)).
- Single-precision GPU accumulation is not fully accurate, so Reusser offers a CPU mode for final renders ([notebook](https://rreusser.github.io/notebooks/clifford-and-de-jong-attractors)).
- Brockmann's explorable iterates the map 200,000 times per display; Clifford and Fractal Dream have four parameters, Bedhead and Gumowski-Mira two ([explorable](https://www.complexity-explorables.org/explorables/stranger-things/)).
- Sekara's colour animation varies each of a, b, c, d in its own panel ([post](https://vedransekara.github.io/2019/10/01/temporal_attractors.html)).
- The de Jong map was published in 1987, the Clifford write-up is from 2004 ([de Jong](https://paulbourke.net/fractals/peterdejong/), [Clifford](https://paulbourke.net/fractals/clifford/)).

## Visual references
- [Bourke, Clifford Attractors](https://paulbourke.net/fractals/clifford/) — still renders and parameter sets
- [Reusser, Clifford and de Jong notebook](https://rreusser.github.io/notebooks/clifford-and-de-jong-attractors) — interactive real-time WebGL render
- [Brockmann, Stranger Things](https://www.complexity-explorables.org/explorables/stranger-things/) — sliders for four attractor maps
- [Sekara, Strange Attractors Part II](https://vedransekara.github.io/2019/10/01/temporal_attractors.html) — parameter-sweep GIF animations
- [orbita2d gallery repo](https://github.com/orbita2d/clifford) — frame-sequence animations of the attractor
- [Evolution of a Clifford Attractor](https://www.youtube.com/watch?v=ZU1EitsgVC8) — channel Jeremy A. Owen; attractor evolving as parameters change
- [Clifford attractor tutorial Max/MSP/Jitter](https://www.youtube.com/watch?v=Xqlmd3i24VU) — channel Ryan Dunn; real-time build in a visual patching tool
- [Timothy Tan, Clifford Attractor Improv](https://www.youtube.com/watch?v=qeIUPP9B3bM) — channel timbretan; improvised performance with the attractor

## Sources
- https://paulbourke.net/fractals/clifford/
- https://paulbourke.net/fractals/peterdejong/
- https://paulbourke.net/fractals/attract3d/index.html
- https://rreusser.github.io/notebooks/clifford-and-de-jong-attractors
- https://www.complexity-explorables.org/explorables/stranger-things/
- https://vedransekara.github.io/2019/10/01/temporal_attractors.html
- https://github.com/orbita2d/clifford
- https://www.youtube.com/oembed?url=https://www.youtube.com/watch?v=ZU1EitsgVC8&format=json
