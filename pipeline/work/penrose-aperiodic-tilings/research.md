# Penrose Aperiodic Tilings — research

## Origin
- Roger Penrose worked out these non-repeating tilings in the 1970s. His first version (P1, 1974) used six pentagon-based tiles with matching rules. He later cut it down to two shapes: kites and darts (P2) or two rhombi (P3). Robert Ammann found the rhombus version independently in 1976. ([Wikipedia: Penrose tiling](https://en.wikipedia.org/wiki/Penrose_tiling))
- Martin Gardner popularised them in Scientific American in January 1977. The roots go back to Kepler (1619) and to Hao Wang and Robert Berger's 1960s work on aperiodic tile sets. ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling))
- N. G. de Bruijn gave two constructions in 1981: the multigrid or pentagrid method and cut-and-project from a 5D cubic lattice. ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling), [Quasicrystal](https://en.wikipedia.org/wiki/Quasicrystal))
- Physics followed: Dan Shechtman saw ten-fold electron diffraction in an Al-Mn alloy in 1982, published in 1984, and won the 2011 Nobel Prize in Chemistry. ([Wikipedia: Quasicrystal](https://en.wikipedia.org/wiki/Quasicrystal))

## Key works and artists
- **Max Cooper, "Penrose Tiling" (2019, album *Yearning for the Infinite*)**: visuals by Jessica In. They show a Penrose tiling growing in spinning trails that look random at first and slowly reveal their structure, with several scales layered to show self-similarity. The track uses only prime-number loop lengths, so no bar repeats exactly. The visuals were made to cover the roof, walls and floor of a room. ([Creative Commission](https://creative-commission.com/project/max-cooper/music-video/jessin-0), [Partyflock](https://partyflock.nl/video/194348:muziekvideo-Penrose-Tiling-Max-Cooper))
- **Girih patterns and Darb-i Imam shrine, Isfahan (1453)**: Peter Lu and Paul Steinhardt (2007) argued that Penrose-like tiling underlies medieval Islamic girih decoration. ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling))
- **Architecture**: a terrazzo Penrose floor at Miami University (1979), the Oxford Mathematics building entrance, a Helsinki pedestrian street (2014) and the Salesforce Transit Center, San Francisco (2018). ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling))
- **Clark Richert, Drop City (1970)**: used similar rhombs before Penrose published. ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling))

## How it works
- **Substitution (inflate/deflate)**: every tile is cut into smaller tiles of the same kinds, so the tiling is self-similar. The kite-to-dart ratio tends to the golden ratio (phi). ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling))
- **Pentagrid (de Bruijn)**: take five families of parallel lines, each rotated 72 degrees from the last. Every place where two lines cross becomes one rhombus, oriented across those two lines. Neighbouring tiles with parallel edges form "ribbons" that follow the lines. ([Minutephysics via Rosetta](https://rosetta.to/u/minutephysics/why-penrose-tiles-never-repeat), [Wikipedia: Pentagrid](https://en.wikipedia.org/wiki/Pentagrid))
- **Why it never repeats**: along a ribbon the ratio of wide to thin tiles is phi, which is irrational. About 16.18 wide tiles per 10 thin ones on average. ([Minutephysics via Rosetta](https://rosetta.to/u/minutephysics/why-penrose-tiles-never-repeat))
- **Cut-and-project**: a flat 2D slice through a 5D cubic lattice, projected down. Rotating or shifting the slice changes the tiling smoothly, which makes it a natural animation parameter. ([Wikipedia: Quasicrystal](https://en.wikipedia.org/wiki/Quasicrystal))
- Grids with 7, 8 or 10 line families give related quasi-periodic tilings. ([Minutephysics via Rosetta](https://rosetta.to/u/minutephysics/why-penrose-tiles-never-repeat))

## Visual language
- Two rhombus shapes (36/144 and 72/108 degrees) or kites and darts, with local five-fold stars and decagons that never line up globally. At most one point of exact global five-fold symmetry exists. ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling))
- It looks ordered but is never periodic, so the eye keeps finding patterns that dissolve. Strong in high-contrast line, two-colour fill, or glowing edges on black.
- Motion comes from growth and subdivision (Cooper's video), colour by ribbon family or by tile type, and rotating or sweeping the grid.

## Live potential
(Suggestions, not sourced claims.)
- **Kick**: step the inflation level by one, or pulse tile scale or outlines. A crossfade between levels hides the pop.
- **Bass**: slide the pentagrid offsets or the 5D slice angle so the whole tiling reshuffles continuously. Tiles flip between rhombus types smoothly.
- **Hi-hats**: flash random tiles or one ribbon family, or spark vertices.
- **BPM**: rotate the whole field by a fixed angle per bar; step through the five line families one per beat.
- **MIDI faders**: grid offset, line spacing, number of families (5, 7, 8, 10), colour mode, line width.
- **Layering**: works well as a luma or alpha mask over video, or as a feedback source. Use a monochrome version over textures.
- **Boring when**: a static tiling slowly zoomed. It needs constant structural change or a deep zoom into self-similar levels.

## Variations
1. **Pentagrid shader**: compute tiles per pixel from line crossings in GLSL, animate the five offsets with LFOs so the tiling "breathes" in real time.
2. **Beat-stepped inflation**: kick advances the subdivision level in a Cooper-style growth with trails.
3. **n-fold switch**: fader morphs from 5 to 7, 8 or 10 line families.
4. **Prime-loop rhythm**: five or more tile layers with prime-length loops, so the whole mix never repeats, echoing Cooper's track.
5. **Quasicrystal diffraction view**: show the ten-fold diffraction pattern next to the tiling, from the Shechtman story.

## Facts
- Penrose tilings have local pentagonal symmetry but never repeat; at most one centre of global five-fold symmetry. ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling))
- A Penrose-like tiling was found in girih patterns at the 1453 Darb-i Imam shrine. ([Wikipedia](https://en.wikipedia.org/wiki/Penrose_tiling))
- de Bruijn showed Penrose tilings are 2D slices of 5D hypercubic structures. ([Wikipedia: Quasicrystal](https://en.wikipedia.org/wiki/Quasicrystal))
- Shechtman waited about two years to publish; Linus Pauling dismissed his result. ([Wikipedia: Quasicrystal](https://en.wikipedia.org/wiki/Quasicrystal), [Veritasium summary](https://summify.io/discover/the-infinite-pattern-that-never-repeats-48sCx-))
- Max Cooper made every loop length in his Penrose track a prime number. ([Creative Commission](https://creative-commission.com/project/max-cooper/music-video/jessin-0))

## Visual references
- [Max Cooper — Penrose Tiling (Creative Commission)](https://creative-commission.com/project/max-cooper/music-video/jessin-0) — project page for the Jessica In music video: growing tiling with spinning trails (no video embedded; I could not confirm a YouTube URL)
- [Veritasium — The Infinite Pattern That Never Repeats (YouTube)](https://www.youtube.com/watch?v=48sCx-wBs34) — explainer with animated tilings and quasicrystals; channel Veritasium, title confirmed via oEmbed
- [minutephysics — Why Penrose Tiles Never Repeat (YouTube)](https://www.youtube.com/watch?v=-eqdj63nEr4) — pentagrid and ribbon animation; channel minutephysics, title confirmed via oEmbed
- [Penrose Tiling (Rhombi), Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Penrose_Tiling_(Rhombi).svg) — thick/thin rhombus tiling with five-fold symmetry

## Sources
- [Wikipedia: Penrose tiling](https://en.wikipedia.org/wiki/Penrose_tiling)
- [Wikipedia: Pentagrid](https://en.wikipedia.org/wiki/Pentagrid)
- [Wikipedia: Quasicrystal](https://en.wikipedia.org/wiki/Quasicrystal)
- [Minutephysics, Why Penrose Tiles Never Repeat (transcript page)](https://rosetta.to/u/minutephysics/why-penrose-tiles-never-repeat)
- [Veritasium summary](https://summify.io/discover/the-infinite-pattern-that-never-repeats-48sCx-)
- [Creative Commission: Max Cooper, Penrose Tiling](https://creative-commission.com/project/max-cooper/music-video/jessin-0)
- [Partyflock: Penrose Tiling](https://partyflock.nl/video/194348:muziekvideo-Penrose-Tiling-Max-Cooper)
- [Wikimedia Commons: Category Penrose tilings](https://commons.wikimedia.org/wiki/Category:Penrose_tilings)
