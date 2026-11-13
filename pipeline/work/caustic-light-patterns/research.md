# Caustic Light Patterns — research

## Origin
- A caustic is the envelope of light rays reflected or refracted by a curved surface, or its projection onto another surface. The word comes from Greek for "burnt" ([Wikipedia](https://en.wikipedia.org/wiki/Caustic_(optics))). Rainbows are caustics too (same source).
- The page does not name who first described caustics, so no discoverer or date is claimed here.
- Computer graphics: an early animated example is Mark Watt's 30-second "Water Caustics" (SIGGRAPH 1991 Electronic Theatre, Digital Pictures) ([SIGGRAPH History Archive](https://history.siggraph.org/?p=65616)). Real-time water caustics are covered in GPU Gems (Guardado, 2004), and games often use pre-computed textures instead of physical simulation ([Wikipedia](https://en.wikipedia.org/wiki/Caustic_(optics))).
- Inverse "caustic engineering" (design a surface that casts a chosen image) was formalised with optimal transport, e.g. Schwartzburg et al. 2014 ([Wikipedia](https://en.wikipedia.org/wiki/Caustic_(optics))).

## Key works and artists
- **River is... (Jussi Ängeslevä, 2012, Gwangju):** a 23 m² chrome sculpture, computationally designed and CNC-milled, frozen from a moment of the Yeongsan River. Two moving lights make its reflections spell words on the wall ([artist page](https://angesleva.iki.fi/art/river-is/)).
- **Caustic Rays (Loop.pH, Swarovski Kristallwelten Store Wien):** curved reflective optical filters plus programmable LEDs recreate shifting water-light reflections over an escalator ([Swarovski](https://www.kristallwelten.swarovski.com/en/artworks/caustic-rays)).
- **Naturally Focused (Alison Stott, reported Nov 2025):** a hand-blown glass lens in gyroscopic rings that projects caustics, inspired by Michael Berry's work on light singularities ([Physics World](https://physicsworld.com/a/caustic-light-patterns-inspire-new-glass-artwork/)).

## How it works
- A wavy surface acts like many small curved lenses. Refracted rays bunch up at the edges of regions where they cross, so those boundaries are very bright and sharp. The floor pattern is a 2D slice through moving 3D "caustic sheets" ([Atmospheric Optics](https://atoptics.co.uk/fz776.htm)).
- Caustics stay sharp when the distance changes, unlike a lens focus. Catastrophe theory says only seven stable elementary forms exist, so lines meet at cusps in familiar ways (same source).
- Real-time fake used in shaders: warp coordinates in a loop with time-driven sin/cos, accumulate `1/length(...)`, then raise the result to a power to sharpen thin bright lines. Parameters are iteration count, speed, tint and intensity ([Godot port of a Shadertoy effect](https://godotshaders.com/shader/water-caustic/)).
- Another cheap fake: distorted smooth Voronoi noise sampled in world xz, weighted by how much water the light crossed ([Moana Shadertoy making-of](https://wallisc.github.io/rendering/2020/12/08/Making-Of-Moana-the-shadertoy.html)).

## Visual language
- Thin, bright, branching filaments with brighter patches between them. They flicker and drift, never repeat, but always look organised (cells, cusps, crossing nets) ([Atmospheric Optics](https://atoptics.co.uk/fz776.htm)).
- Typically a cool tinted base (the Godot default tint is teal) with near-white lines. Chrome and glass versions give hard, graphic line art that can carry text or images ([artist page](https://angesleva.iki.fi/art/river-is/)).
- Reads as architecture and water: good as a calm texture on walls, floors and mapped surfaces.

## Live potential
Suggestions from the technique, not from sources:
- **Kick:** pulse intensity or briefly widen the line sharpness exponent; or kick adds a wave impulse that spreads outward.
- **Bass:** modulate wave amplitude (more amplitude gives more crossing and tighter filaments).
- **Hi-hats:** jitter small high-frequency ripples or sparkle on filament crossings.
- **BPM:** lock time/speed to a multiple of tempo; reverse or freeze on breaks.
- **Faders/MIDI:** speed, scale, iteration count, tint, intensity, second layer offset.
- **Layering:** additive blend over dark footage, use as a lighting pass on 3D or mapped geometry, or as a luma mask or displacement source for other clips.
- **Boring when:** it loops at one constant speed, one scale, one colour with no accent. It needs dynamics and a second scale.

## Variations
1. Two caustic layers at different scales and speeds with parallax, for depth.
2. Beat-driven impulse ripples (a height field feeding the caustic), so hits are visible as rings.
3. Chrome-sculpture style: caustic that resolves into a hidden word or logo on a bar drop (after River is...).
4. Prismatic split: offset R, G, B caustics for coloured fringes.
5. Mapped use: caustic as light on projection-mapped architecture, driven via Spout from Resolume or UE.

## Facts
- Rippling caustics from water waves are made by crossing refracted rays, and rainbows are caustics ([Wikipedia](https://en.wikipedia.org/wiki/Caustic_(optics))).
- Catastrophe theory limits stable caustics to seven elementary forms ([Atmospheric Optics](https://atoptics.co.uk/fz776.htm)).
- The Atmospheric Optics example was photographed in only 30–50 cm of sea water ([Atmospheric Optics](https://atoptics.co.uk/fz776.htm)).
- A 1991 caustics animation ran just 30 seconds on Silicon Graphics hardware ([SIGGRAPH History Archive](https://history.siggraph.org/?p=65616)).
- A hidden-text chrome sculpture used 23 m² of CNC-milled surface ([artist page](https://angesleva.iki.fi/art/river-is/)).
- Shadertoy shaders default to CC BY-NC-SA 3.0, so check the licence before reusing code ([Godot Shaders](https://godotshaders.com/shader/water-caustic/)).

## Visual references
- [River is... — Jussi Ängeslevä](https://angesleva.iki.fi/art/river-is/) — chrome sculpture projecting caustic words onto a wall, documentation images
- [Caustic Rays — Loop.pH / Swarovski](https://www.kristallwelten.swarovski.com/en/artworks/caustic-rays) — ceiling caustic light installation
- [Naturally Focused — Physics World](https://physicsworld.com/a/caustic-light-patterns-inspire-new-glass-artwork/) — glass lens art projecting caustics
- [Water Caustic — Godot Shaders](https://godotshaders.com/shader/water-caustic/) — code for a cheap animated caustic with speed and tint (port of Shadertoy MdlXz8, which I could not open)
- [High-contrast Computational Caustic Design (YouTube, channel GCM EPFL)](https://www.youtube.com/watch?v=R00IvqcI9jU) — SIGGRAPH 2014 video of surfaces engineered to cast images as caustics; title and channel confirmed via oEmbed, content not watched

## Sources
- https://en.wikipedia.org/wiki/Caustic_(optics)
- https://atoptics.co.uk/fz776.htm
- https://angesleva.iki.fi/art/river-is/
- https://www.kristallwelten.swarovski.com/en/artworks/caustic-rays
- https://physicsworld.com/a/caustic-light-patterns-inspire-new-glass-artwork/
- https://history.siggraph.org/?p=65616
- https://godotshaders.com/shader/water-caustic/
- https://wallisc.github.io/rendering/2020/12/08/Making-Of-Moana-the-shadertoy.html
- https://www.youtube.com/oembed?url=https://www.youtube.com/watch?v=R00IvqcI9jU&format=json
