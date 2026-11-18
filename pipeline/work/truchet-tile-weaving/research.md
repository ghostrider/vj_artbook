# Truchet Tile Weaving — research

## Origin
Sébastien Truchet, a French priest and polymath, described square tiles split along the diagonal into two contrasting triangles in a 1704 memoir, "Mémoire sur les combinaisons" ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)). Cyril Stanley Smith popularised them in 1987 and introduced the quarter-circle variant, in which two arcs join the midpoints of adjacent sides. That variant had already appeared in the games Trax and the Black Path Game ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)). Christopher Carlson extended Smith's idea in 2018 to tiles scaled by powers of 1/2, which gives multi-scale patterns ([Bridges 2018](https://archive.bridgesmathart.org/2018/bridges2018-39.html)).

## Key works and artists
- Cyril Stanley Smith, 1987: the quarter-circle tile that produces continuous lines dividing the plane into two-colourable regions ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile), [Bridges 2018](https://archive.bridgesmathart.org/2018/bridges2018-39.html)).
- Christopher Carlson, "Multi-Scale Truchet Patterns", Bridges 2018, pp. 39-44. It adds a "winged" tile and an infinite set of tiles scaled by 1/2 ([Bridges 2018](https://archive.bridgesmathart.org/2018/bridges2018-39.html)).
- Lionel Radisson (makio135), "Multi-scale Truchet patterns", an Observable notebook dated 24 March 2021 that implements the multi-scale idea ([Observable](https://observablehq.com/@makio135/multi-scale-truchet-patterns)).
- Nick Montfort calls the one-line Commodore 64 BASIC program that generates the maze version "a found poem" ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)).
- The Art of Code, "Shader Coding: Truchet Tiling Explained!" (2019), a shader tutorial that builds vine-like Truchet patterns in Shadertoy ([YouTube](https://www.youtube.com/watch?v=2R7h76GoIJM)).

## How it works
The tile is a square whose pattern is not rotationally symmetric, so its orientation carries information ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)). The quarter-circle tile has two orientations. A shader hashes each cell's integer coordinate to a random value and uses it to choose the orientation. Because the arcs always meet at edge midpoints, the lines continue across cell borders and form closed loops and long paths. A random square Truchet tiling is equivalent to bond percolation on a square lattice at the critical probability p = 1/2 ([Chalkdust](https://chalkdustmagazine.com/features/when-truchet-met-chladni/)). That is why the paths look neither too short nor too long. In the multi-scale version, tiles at different sizes (powers of 1/2) are combined in one pattern ([Bridges 2018](https://archive.bridgesmathart.org/2018/bridges2018-39.html)).

## Visual language
Continuous black-and-white arcs, loops, knots and maze corridors. A strict grid gives a calm rhythm, and the random orientation gives organic flow. Line thickness and colour can be changed freely. Tile types are black-and-white, with square tiles in two kinds and hexagonal tiles in three ([Chalkdust](https://chalkdustmagazine.com/features/when-truchet-met-chladni/)).

## Live potential
These are design ideas, not sourced claims.
- Kick: flip a random subset of tiles, or advance the seed, so the weave re-knots on the beat.
- Bass: line thickness or tile-scale pulse. Hi-hats: thin flashes along a single path.
- Wave: rotate tiles with a phase offset based on distance from a point, so a ripple re-weaves the pattern.
- BPM: step the hash seed once per bar. MIDI faders: grid density, line width, flip probability (0.5 gives the percolation look), multi-scale depth.
- Layering: works well as a mask or luma matte for video, or as a feedback source. A flat static version gets boring on a big screen, so keep scale and probability moving.

## Variations
- Flip probability away from 0.5: bias toward long diagonal rivers or tight closed rings.
- Multi-scale subdivision triggered by kick: cells split in half on hits and merge back.
- Tile pairs mixed with the diagonal or labyrinth tile ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)).
- Hexagonal Truchet tiles for a softer, non-square look ([Chalkdust](https://chalkdustmagazine.com/features/when-truchet-met-chladni/)).
- Use the pattern as a displacement or texture source in Unreal via Spout.

## Facts
- Truchet described the tiles in 1704 ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)).
- A random square Truchet tiling is bond percolation at p = 1/2 ([Chalkdust](https://chalkdustmagazine.com/features/when-truchet-met-chladni/)).
- In 2002 Bogomolny and Schmit conjectured that random Chladni figures statistically match random Truchet tilings at large scales ([Chalkdust](https://chalkdustmagazine.com/features/when-truchet-met-chladni/)).
- The classic 10 PRINT one-liner on the Commodore 64 draws the labyrinth version ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)).
- Carlson's multi-scale tiles use powers of 1/2 ([Bridges 2018](https://archive.bridgesmathart.org/2018/bridges2018-39.html)).
- Truchet died in 1729, long before Chladni (born 1756) ([Chalkdust](https://chalkdustmagazine.com/features/when-truchet-met-chladni/)).

## Visual references
- [Shader Coding: Truchet Tiling Explained!](https://www.youtube.com/watch?v=2R7h76GoIJM) — video tutorial by The Art of Code (YouTube); builds vine-like Truchet patterns in a shader.
- [Multi-scale Truchet patterns](https://observablehq.com/@makio135/multi-scale-truchet-patterns) — Observable notebook by Lionel Radisson (makio135), multi-scale variant.
- [Truchet tile (Wikipedia)](https://en.wikipedia.org/wiki/Truchet_tile) — tile variants and example tilings.
- [Multi-Scale Truchet Patterns (Bridges 2018)](https://archive.bridgesmathart.org/2018/bridges2018-39.html) — paper by Christopher Carlson with figures.

## Lead image
NONE. I could not open a Wikimedia Commons file page or the Commons API for a free-licensed Truchet image, so no licence could be checked.

## Sources
- [Truchet tile, Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)
- [When Truchet met Chladni, Chalkdust](https://chalkdustmagazine.com/features/when-truchet-met-chladni/)
- [Multi-Scale Truchet Patterns, Bridges 2018](https://archive.bridgesmathart.org/2018/bridges2018-39.html)
- [Multi-scale Truchet patterns, Observable](https://observablehq.com/@makio135/multi-scale-truchet-patterns)
- [Shader Coding: Truchet Tiling Explained!, YouTube](https://www.youtube.com/watch?v=2R7h76GoIJM)
