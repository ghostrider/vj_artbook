# SDF Raymarched Morphing Forms — research

## Origin
- Sphere tracing, the core of SDF raymarching, is credited to John C. Hart, in a June 1995 paper on antialiased ray tracing of implicit surfaces ([Wikipedia: Ray marching](https://en.wikipedia.org/wiki/Ray_marching)). The demoscene and Inigo Quilez (co-creator of Shadertoy) later made it popular, per the same article.
- A signed distance function (SDF) returns the distance from any point to a shape's surface. The sign tells you whether the point is inside or outside ([Wikipedia: Signed distance function](https://en.wikipedia.org/wiki/Signed_distance_function)).
- In live AV, evvvvil's 2017 vvvv "Generative Show" with OddJohn used raymarched distance fields and credits Inigo Quilez's distance-field work ([vvvv blog, 5 Mar 2017](https://legacy.vvvv.org/blog/evvvvil-oddjohn-generative-show)).

## Key works and artists
- **Inigo Quilez**: his Shadertoy and YouTube live-coding sessions (e.g. "Sphere Gears", parts 1 and 2) show an SDF scene built from scratch ([Part 1](https://www.youtube.com/watch?v=sl9x19EnKng), [Part 2](https://www.youtube.com/watch?v=bdICU2uvOdU)).
- **evvvvil + OddJohn, Generative Show (2017)**: a live AV set. The visuals were raymarched distance fields plus other generative layers, and Resolume is among the listed tools ([vvvv blog](https://legacy.vvvv.org/blog/evvvvil-oddjohn-generative-show)).
- **Marching.js (Charlie Roberts)**: a JavaScript library that compiles GLSL raymarchers, with an API terse enough for live coding performance ([GitHub](https://github.com/charlieroberts/marching)).
- **Sebastian Lague, "Coding Adventure: Ray Marching"**: a tutorial video for building the technique ([YouTube](https://www.youtube.com/watch?v=Cp5WWtMoeKg)).
- **Trippasch, MandelBox Audio Visualization**: a Unity raymarcher driven by audio ([YouTube](https://www.youtube.com/watch?v=A9TlDTLxBmk)).

## How it works
- Each pixel casts a ray. At every step the scene SDF says how far the nearest surface is, so the ray can safely jump that far. It stops when the distance falls below a small threshold ([Wikipedia: Ray marching](https://en.wikipedia.org/wiki/Ray_marching)).
- Shapes are combined with boolean operators and modulo (domain repetition) to build complex surfaces ([Wikipedia: Ray marching](https://en.wikipedia.org/wiki/Ray_marching)).
- Morphing between two shapes is a blend of their distance values. Smooth union (smin) rounds the joint instead of leaving a crease. Smin gives a slightly under-estimated distance in the blend zone, so the step size should be reduced a little to avoid artifacts.
- The method suits morphing, soft shadows, repetition and algorithmic scenes, but it brings little over normal ray tracing for simple scenes ([Wikipedia: Ray marching](https://en.wikipedia.org/wiki/Ray_marching)).

## Visual language
- Smooth, liquid joins between spheres, boxes and tori. The surfaces look like clay, metal or glass, depending on the shading.
- Endless repetition of a cell along a corridor, with the camera flying through it.
- Twisted and bent space that makes straight forms spiral.
- Rhythm comes from the blend radius pumping, forms swelling and splitting, and camera cuts. Palettes can stay near-monochrome with one accent, with ambient occlusion and fog giving depth.
- Fractal forms (Mandelbox-like) fit the same renderer, per the audio demo above.

## Live potential
Suggested mappings (design ideas, not sourced):
- **Kick**: pulse the blend factor k or the sphere radii. A short decaying envelope works better than raw level.
- **Bass**: warp amplitude, or a slow twist of the space.
- **Hi-hats**: fine displacement, a color flash, or a step in the repetition cell offset.
- **BPM**: lock the morph cycle and camera path to bars. Phase-lock to a clock, not to time.
- **MIDI faders**: shape select, repetition spacing, twist, camera speed, palette. A KeyStep or BeatStep Pro can step through shape sets.
- **Layering**: output with an alpha or depth mask for Resolume, or share via Spout/NDI. Dark backgrounds let it stack well with other layers.
- **Boring risks**: a single blob that only grows and shrinks, static camera, uniform grey shading, no contrast changes over minutes. Cost matters: heavy raymarch loops need a step cap and a resolution scale on a big screen.

## Variations
1. **Kick-blob chain**: 3 to 6 spheres orbiting on a BPM-locked path, smooth-merged, k driven by the kick.
2. **Repeated corridor**: domain repetition with per-cell shape morph, camera on a tempo-synced rail.
3. **Twist and melt**: a stack of primitives with bend and twist controlled by faders, with hi-hats adding noise displacement.
4. **Mask mode**: render the SDF as a silhouette or depth mask to cut other video layers in Resolume.
5. **Sequenced shape bank**: MIDI notes select the primitive pairs, so a keyboard plays the morph targets.

## Facts
- Sphere tracing dates from 1995 ([Wikipedia: Ray marching](https://en.wikipedia.org/wiki/Ray_marching)).
- A true SDF has gradient length 1 almost everywhere (the eikonal property) ([Wikipedia: Signed distance function](https://en.wikipedia.org/wiki/Signed_distance_function)).
- Smooth operations break that property, so the marcher must take more cautious steps.
- Raymarched live sets are not new: evvvvil's 2017 show played them live with the Resolume-adjacent toolchain ([vvvv blog](https://legacy.vvvv.org/blog/evvvvil-oddjohn-generative-show)).
- Raymarchers can be live-coded in the browser through JavaScript that compiles to GLSL ([Marching.js](https://github.com/charlieroberts/marching)).

## Visual references
- [Quilez, Live Coding "Sphere Gears" Part 1](https://www.youtube.com/watch?v=sl9x19EnKng) — video by Inigo Quilez (channel: Inigo Quilez), an SDF scene built live from scratch.
- [Quilez, Live Coding "Sphere Gears" Part 2](https://www.youtube.com/watch?v=bdICU2uvOdU) — continuation of the same session (channel: Inigo Quilez).
- [Sebastian Lague, Coding Adventure: Ray Marching](https://www.youtube.com/watch?v=Cp5WWtMoeKg) — tutorial with morphing and repeated SDF forms (channel: Sebastian Lague). I confirmed the title and channel, not the content.
- [MandelBox Audio Visualization, Unity Demo](https://www.youtube.com/watch?v=A9TlDTLxBmk) — audio-driven raymarched fractal (channel: Trippasch). I confirmed the title and channel, not the content.
- [evvvvil + OddJohn Generative Show](https://legacy.vvvv.org/blog/evvvvil-oddjohn-generative-show) — project page for a live raymarching AV set. The page has no video link.

## Sources
- [Wikipedia: Ray marching](https://en.wikipedia.org/wiki/Ray_marching)
- [Wikipedia: Signed distance function](https://en.wikipedia.org/wiki/Signed_distance_function)
- [vvvv blog: evvvvil + OddJohn Generative Show](https://legacy.vvvv.org/blog/evvvvil-oddjohn-generative-show)
- [Marching.js on GitHub](https://github.com/charlieroberts/marching)
- YouTube oEmbed checks for the four videos above
