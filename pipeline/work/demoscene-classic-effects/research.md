# Demoscene Classic Effects — research

## Origin
Demo effects are small real-time graphics tricks that grew out of early display hacks: the "Bouncing Ball" on the Whirlwind computer (early 1950s) and "munching squares" on the PDP-1 (around 1962) are named as early ancestors ([Wikipedia: Demo effect](https://en.wikipedia.org/wiki/Demo_effect)). The demoscene itself took off in the 1980s on the Commodore 64, Atari ST and Amiga, where coders reverse-engineered custom hardware for new effects. In the MS-DOS era of the 1990s, PCs had no special chip, so effects became pixel-level software rendering, and many chunky-pixel effects were written first on PC or Falcon030 ([same source](https://en.wikipedia.org/wiki/Demo_effect)).

Specific roots:
- **Plasma**: Bret Mulvey's "Plasma" VGA demo (1988, released on CompuServe) used the diamond-square algorithm and cycled the hardware palette. Demoscene coders adopted it heavily in the early 1990s ([Wikipedia: Plasma effect](https://en.wikipedia.org/wiki/Plasma_effect)).
- **Raster / copper bars**: seen on the Atari 2600 and Atari 8-bit, then in C64, Amiga, Atari ST and Amstrad CPC demos. "Copper bar" comes from the Amiga's Copper coprocessor ([Wikipedia: Raster bar](https://en.wikipedia.org/wiki/Raster_bar)).
- **Rotozoomer**: Sanity implemented one on the Amiga using pre-rendered planar bitmaps and copper effects ([Wikipedia: Demo effect](https://en.wikipedia.org/wiki/Demo_effect)).
- Today general-purpose 3D engines and procedural content matter more in demos than these effects ([same](https://en.wikipedia.org/wiki/Demo_effect)), which is why they now read as a retro style.

## Key works and artists
- **Bret Mulvey, "Plasma" (1988)**: the origin of the plasma effect ([Wikipedia](https://en.wikipedia.org/wiki/Plasma_effect)).
- **Kefrens, Megademo 8, "Transformer" part (coded by Promax)**: first appearance of the "glenz" see-through vector look, a name given by Photon ([Wikipedia](https://en.wikipedia.org/wiki/Demo_effect)). Kefrens bars are also a listed effect.
- **Sanity**: Amiga rotozoomer ([Wikipedia](https://en.wikipedia.org/wiki/Demo_effect)).
- **Bonz, "Sines and Cosines for Fun and Profit" (Hugi 24)**: a coder-written tutorial with the formulas for plasma, interference, Lissajous trajectories, shadebobs, rotozoom, starfield and wormhole ([Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)).
- **Trixter, "Demos Explained" graphics glossary** (archived 2000): defines plasma, tunnel, copper and rotating bitmap, with screenshots credited to Delusion/Sonic-PC and Show/Majic 12 ([scheib.net mirror](https://www.scheib.net/play/demos/what/trixter/demo_graphics.html)).
- **Burger, Paulovic, Hasan, "Realtime Visualization Methods in the Demoscene" (CESCG 2002)**: an academic overview covering copper, fire, rotozoomer, voxel landscapes and more, with the authors' own demos Dream (1999) and Expiration (2000) ([CESCG](https://old.cescg.org/CESCG-2002/BBurger)).

## How it works
- **Plasma**: sum several cosines whose arguments depend on x, y and time, scale to 0–255, and map through a palette. Bonz's form is four cosines, each with its own frequency, speed and phase; he warns against giving two parameters the same value ([Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)). Rotating the palette over time gives the flowing motion ([Wikipedia](https://en.wikipedia.org/wiki/Plasma_effect)).
- **Raster / copper bars**: change a colour table entry per scanline, so different lines show different colours. Several gradated colours in sequence make a metallic bar ([Wikipedia: Raster bar](https://en.wikipedia.org/wiki/Raster_bar)). In a shader this is just colour as a function of y plus sine-moved bar centres.
- **Rotozoomer**: for each screen pixel, rotate and scale its coordinates back into a tiled texture: `x = (x' cos a − y' sin a)/k`, `y = (x' sin a + y' cos a)/k`, with zoom such as `k = 1.5 + sin(t)` ([Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)).
- **Tunnel / wormhole**: draw circles at decreasing depth with centres on a Lissajous path, projected with `d/z` ([Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)). Texture-mapped tunnels are usually done with a screen-to-texture lookup table ([Wikipedia: Demo effect](https://en.wikipedia.org/wiki/Demo_effect)).
- **Interference**: add two textures sampled at cosine-driven offsets ([Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)).

## Visual language
Saturated gradient palettes cycling through hue; smooth sine-warped blobs (plasma); horizontal metallic gradient bands that bob in sine paths (copper); tiled textures spinning and breathing in zoom (rotozoomer); fast forward flight through rings (tunnel). Hard pixel grids, low colour counts and big chunky pixels sell the 90s feel. The rhythm is always periodic and sine-based, which is why it locks to BPM so easily.

## Live potential
- **Kick**: pulse the zoom `k` of the rotozoomer, kick bar thickness or brightness, a tunnel speed jump.
- **Bass**: modulate the plasma frequency parameters or amplitude of bar sine paths.
- **Hi-hats**: palette step or palette-cycle speed, scanline flicker, small bar-count increments.
- **BPM**: use beat phase as the `t` in all sines so one period equals one bar or beat; rotation angle advances by a fixed amount per beat.
- **MIDI faders (KeyStep/BeatStep)**: frequencies a–l of the plasma, bar count, palette selection, pixelation level, zoom, rotation speed.
- **Layering**: use as a colourful matte or mask for other clips in Resolume; blend modes (Add, Multiply) over video; feed through a Spout texture from UE or TouchDesigner. Pixelate or posterise as a final pass for the retro look.
- **Boring on a big screen**: a single static palette, constant speed, and a full-screen plasma with no contrast. Fix with a limited palette, hard edges, tempo-synced changes and combining two effects.

## Variations
1. Palette from a live source: sample colours from the camera or from another Resolume layer instead of a fixed gradient.
2. 3D copper bars: bars as sine-moved ribbons in UE Niagara or a depth-aware shader.
3. Plasma as displacement map for video (warp a clip with the plasma value), not as colour.
4. Rotozoomer using the VJ's own logo or a live feed as the tiled texture.
5. Glenz or shadebob trails with a feedback loop for light-painting trails (Bonz describes shadebobs as bitmaps added along a path with a history of positions, [Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)).

## Facts
- The first plasma demo dates to 1988 and used palette cycling, not per-pixel recalculation ([Wikipedia](https://en.wikipedia.org/wiki/Plasma_effect)).
- "Copper bar" comes from the Amiga's Copper coprocessor, which changes colours per scanline with little CPU work ([Wikipedia](https://en.wikipedia.org/wiki/Raster_bar)).
- Raster bars were already possible on the Atari 2600 ([Wikipedia](https://en.wikipedia.org/wiki/Raster_bar)).
- The names "glenz" and "blenk" were coined by Photon ([Wikipedia](https://en.wikipedia.org/wiki/Demo_effect)).
- A Lissajous curve with equal frequencies a = b gives a circle; unequal ones give the classic wandering paths used for bars and bobs ([Hugi](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)).
- Demoparty categories use hard size limits such as 64 KB, 4 KB or 256 bytes, which is why effects are so formula-based ([CESCG paper](https://old.cescg.org/CESCG-2002/BBurger)).

## Visual references
- [Demos Explained: graphics terms (Trixter)](https://www.scheib.net/play/demos/what/trixter/demo_graphics.html) — glossary with period screenshots of plasma, tunnel and copper
- [Wikipedia: Demo effect](https://en.wikipedia.org/wiki/Demo_effect) — overview list of effects and their history
- [Wikipedia: Raster bar](https://en.wikipedia.org/wiki/Raster_bar) — explanation of raster/copper bars
- [Wikipedia: Plasma effect](https://en.wikipedia.org/wiki/Plasma_effect) — explanation of plasma
- [Realtime Visualization Methods in the Demoscene](https://old.cescg.org/CESCG-2002/BBurger) — paper with demo effect overview and their own demos

## Lead image
- kind: similar
- page_url: https://unsplash.com/photos/vibrant-abstract-concentric-color-gradient-qs62YnigrXE
- image_url: https://images.unsplash.com/photo-1791960466884-3a956d57b023?w=1400
- shows: "Abstract chromatic plasma gradient": concentric rings of red, orange, green and cyan, close to a palette-cycled plasma or tunnel colour ramp (smooth, not pixelated)
- photographer: Plasmanica Generator (https://unsplash.com/@plasmanicagenerator)
- license: Unsplash License
- note: no free Commons image could be verified (Commons blocked). No screenshot of a discussed demo hosted by its own creator was found: Kefrens and Sanity screens exist only on mirrors (exotica.org.uk Ami Sector One), the Trixter glossary is a mirror with screenshots of other groups' demos, and the CESCG paper's Fig. 1 shows other groups' demos.

## Sources
- [Wikipedia: Demo effect](https://en.wikipedia.org/wiki/Demo_effect)
- [Wikipedia: Plasma effect](https://en.wikipedia.org/wiki/Plasma_effect)
- [Wikipedia: Raster bar](https://en.wikipedia.org/wiki/Raster_bar)
- [Hugi 24: Bonz, Sines and Cosines for Fun and Profit](https://hugi.scene.org/online/coding/hugi%2024%20-%20coding%20graphics%20bonz%20sines%20and%20cosines%20for%20fun%20and%20profit.htm)
- [Trixter, Demos Explained: graphics](https://www.scheib.net/play/demos/what/trixter/demo_graphics.html)
- [CESCG 2002: Realtime Visualization Methods in the Demoscene](https://old.cescg.org/CESCG-2002/BBurger)
