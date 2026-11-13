# Moire Interference Layers — research

## Origin
- The word comes from *moire*, a rippled "watered" silk made by pressing two damp layers of fabric together. French used it from the 17th century, and the adjective *moire* is attested from at least 1823 ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)).
- The modern science was popularised by Gerald Oster and Yasunori Nishijima in *Scientific American*, May 1963. Their key point is that for near-parallel lines, a small shift of one layer becomes a large shift of the pattern ([SciAm blog](https://www.scientificamerican.com/blog/sa-visual/art-and-science-of-the-moire/)).
- Oster (1918–1993) was a biophysicist who became an artist. He showed at Howard Wise Gallery in 1965 and 1966 and was in MoMA's *The Responsive Eye* (1965), the show that defined Op Art ([SciAm blog](https://www.scientificamerican.com/blog/sa-visual/art-and-science-of-the-moire/)).

## Key works and artists
- **Gerald Oster, *Oster's Magic Moires* (1965) and *Moires and Phosphenes* (1966)**: art made directly from overlaid line screens ([SciAm blog](https://www.scientificamerican.com/blog/sa-visual/art-and-science-of-the-moire/)).
- **Philippe Decrauzat, *On Cover* series (from 2011)**: paintings inspired by the May 1963 SciAm moire cover. Shown at Elizabeth Dee Gallery, Sept–Oct 2014 ([SciAm blog](https://www.scientificamerican.com/blog/sa-visual/art-and-science-of-the-moire/)).
- **Roger Penrose demonstration (Royal Institution)**: rhombus patterns on acetate, rotated and shifted to show the moire spacing changing ([Rosetta mirror page](https://rosetta.to/u/theroyalinstitution/incredible-moir-patterns)).
- **David C. Roy, kinetic sculptures *Shimmer* and *Illusion***: moire made with rotating wheels ([Wood That Works](https://woodthatworks.com/blog/2011/4/14/moire-patterns-in-kinetic-design.html)).

## How it works
- Two layers multiply (light must pass through both). Multiplying two similar spatial frequencies creates a difference frequency, and that is the visible band ([Michael Bach](https://michaelbach.de/ot/lum-moire1/)).
- For two sine gratings with frequencies k1 and k2, the envelope has frequency (k1 - k2)/2. This is the same math as an audio beat tone ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)).
- For identical patterns rotated by angle a, the band spacing is D = p / (2 sin(a/2)), about p/a for small angles. Tiny rotation gives huge bands ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)).
- In shader terms: draw two line, dot or ring fields, then multiply, difference or XOR them. Rotation, scale and offset are the natural parameters.

## Visual language
- Black-and-white fine lines give swimming, breathing bands, curved fans and "ghost" shapes that are not in either layer.
- Concentric rings against rings give curved dark bands that drift (Michael Bach's demo moves the rings at slightly different X/Y speeds so the cycle takes a long time to repeat ([Michael Bach](https://michaelbach.de/ot/lum-moire1/))).
- Dot and hexagonal grids give rosette and lattice textures. Colour comes from blend modes such as Multiply, Screen, Difference and XOR ([Moire Explore](https://nateroe.github.io/moire/index.html)).
- Rhythm: slow drift is hypnotic; a small knob move causes a global ripple.

## Live potential
- **Kick**: short rotation impulse (a fraction of a degree) on one layer. Since D ~ p/a, this swings the whole band pattern. Or a quick pitch change on one layer.
- **Bass / sustained**: map the level to line pitch difference, which makes bands widen or tighten.
- **Hi-hats**: toggle blend mode, flip line thickness, or add a brief phase jump.
- **BPM**: sync one layer's rotation to a full turn per bar and the other to a ratio (for example 2:3), so patterns lock and resolve on downbeats. Moire Explore uses two-keyframe animation between states, which is a good model for beat-quantised morphs ([Moire Explore](https://nateroe.github.io/moire/index.html)).
- **MIDI faders**: layer count (2–8), rotation offset, scale, thickness, blend mode, wave distortion amount.
- **Layering**: use as a mask or displacement map over video, or as a luma key. Multiplying it over other material gives the interference for free. High-contrast output also works on LED walls.
- **Boring risk**: static, evenly paced rotation; pure black and white for a whole set; no depth. Also, fine lines alias on projectors and cameras (the same moire problem), so test at the real resolution.
- Shader-side tip: layered scrolling line and circle patterns, treated as an intersection rather than a union, give cheap motion and spinning effects. The workflow is trial and error with a contact sheet of shapes ([realtimevfx thread](https://realtimevfx.com/t/interference-patterns-for-motion/2671)).

## Variations
1. **Wave-warped grids**: distort one grid with up to a handful of sine waves so bands become organic, as in Moire Explore ([Moire Explore](https://nateroe.github.io/moire/index.html)).
2. **Video as one layer**: use live camera or clip luminance to modulate line width in one layer, so the image emerges through the moire.
3. **Audio spectrum as grating**: draw one layer's lines from FFT bands, so the beat of the music is literally the moire (an audio-beat analogy from the same math).
4. **Rings from tracked points**: ring centres follow MIDI notes or LFOs, so chords make interference rosettes.
5. **Three-layer colour split**: put R, G and B on separately rotated grids for chromatic moire.
6. **Print the physical version**: acetate overlays in front of the projector, as in Penrose's demo.

## Facts
- The moire effect shares its math with acoustic beats ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)).
- Interlaced TV and cameras filming screens create unwanted moire. Banknotes use fine patterns that moire when scanned ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)).
- Twisted bilayer graphene forms a moire pattern, and at a "magic angle" it shows superconductivity ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)).
- Moire fringes magnify displacement, which is why they are used to measure strain ([Wikipedia](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern), [SciAm blog](https://www.scientificamerican.com/blog/sa-visual/art-and-science-of-the-moire/)).
- Oster did not draw the 1963 article's illustrations; the cover is credited to Joan Starwood and Photo-Lettering ([SciAm blog](https://www.scientificamerican.com/blog/sa-visual/art-and-science-of-the-moire/)).

## Visual references
- [Incredible Moire Patterns! (YouTube)](https://www.youtube.com/watch?v=cvWF_Q5-Kt8) — Roger Penrose demonstrates rotating and shifting overlaid patterns; channel The Royal Institution (title and channel confirmed via oEmbed).
- [Moire Explore](https://nateroe.github.io/moire/index.html) — interactive tool: 2–8 rectangular or hexagonal grid layers, blend modes, wave distortion, keyframe animation.
- [Michael Bach, Moire Patterns](https://michaelbach.de/ot/lum-moire1/) — interactive concentric-ring moire you can drag and recolour.
- [SciAm: Art and Science of the Moire](https://www.scientificamerican.com/blog/sa-visual/art-and-science-of-the-moire/) — the 1963 cover, Oster's art and Decrauzat's paintings.
- [Wood That Works, Moire in Kinetic Design](https://woodthatworks.com/blog/2011/4/14/moire-patterns-in-kinetic-design.html) — kinetic moire sculptures by David C. Roy.

## Sources
- [Wikipedia: Moire pattern](https://en.wikipedia.org/wiki/Moir%C3%A9_pattern)
- [Scientific American blog: Art and Science of the Moire](https://www.scientificamerican.com/blog/sa-visual/art-and-science-of-the-moire/)
- [Michael Bach: Moire Patterns](https://michaelbach.de/ot/lum-moire1/)
- [Wood That Works](https://woodthatworks.com/blog/2011/4/14/moire-patterns-in-kinetic-design.html)
- [Rosetta mirror of Royal Institution video](https://rosetta.to/u/theroyalinstitution/incredible-moir-patterns)
- [Moire Explore](https://nateroe.github.io/moire/index.html)
- [realtimevfx: Interference patterns for motion](https://realtimevfx.com/t/interference-patterns-for-motion/2671)
