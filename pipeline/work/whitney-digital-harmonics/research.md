# Whitney Digital Harmonics — research

## Origin
John Whitney Sr. (1917-1995) was an American animator, composer and inventor, often called a pioneer of computer animation ([Wikipedia](https://en.wikipedia.org/wiki/John_Whitney_(animator))). In the late 1950s he converted a WWII M-5 anti-aircraft gun director (later adding an M-7) into an analog animation machine about twelve feet high, with design templates on rotating tables photographed by a rotating camera ([Socks-Studio](https://socks-studio.com/2010/11/25/john-whitney-inventor-and-father-of-computer-animation/)). He worked on the 1958 *Vertigo* titles with Saul Bass, founded Motion Graphics Inc. in 1960 and collected effects in *Catalog* (1961) ([Wikipedia](https://en.wikipedia.org/wiki/John_Whitney_(animator))).

In 1966 IBM made him its first artist in residence. Between 1966 and 1969 Jack Citron programmed IBM 360 computers for him. His first computer film, *Homage to Rameau*, took its name from Rameau's treatise on harmony, which pushed him toward the ideas in his 1980 book *Digital Harmony* (McGraw-Hill) ([Visual Music Archive](https://visualmusicarchive.org/reading/r/digital-harmony.html)). The book argues for the "complementarity" of music and visual art and includes code examples (same page).

## Key works and artists
- *Permutations* (1966): made on an IBM 360 with an IBM 2250 display, programmed in GRAF and FORTRAN. Colour was added later with an optical printer. Colored dots form evolving patterns, each point moving at its own speed ([Visual Music Archive](https://visualmusicarchive.org/works/w/permutations.html)).
- *Matrix III* (1972, 11 min): shapes (triangles, squares, hexagons) follow invisible paths based on a classic Lissajous figure applied symmetrically to the frame. Music by Terry Riley, from *Rainbow in Curved Air* ([Light Cone](https://lightcone.org/en/film-1555-matrix-iii)).
- *Arabesque* (1975): described as the peak of his digital films, with blooming colour forms ([Wikipedia](https://en.wikipedia.org/wiki/John_Whitney_(animator))).
- Whitney-Reed RDTD (Radius-Differential Theta Differential): his later audio-visual composition program, used in works such as *Moon Drum* (1989-1995) ([Wikipedia](https://en.wikipedia.org/wiki/John_Whitney_(animator))).
- Book: *Digital Harmony* (1980), full text on [archive.org](https://archive.org/stream/DigitalHarmony_201611/Digital%20Harmony_djvu.txt).

## How it works
In the book, each graphic element gets its own speed, and the speeds step up in whole-number multiples: base rate, 2x, 3x and so on. Each element also has its own direction. When every element obeys a consistent rule, with no random drift, the field forms and reforms recognisable patterns. Whitney calls this harmonic resonance ([Digital Harmony text](https://archive.org/stream/DigitalHarmony_201611/Digital%20Harmony_djvu.txt)).

Order shows most clearly at resonant moments, especially when speed ratios are whole numbers. Then the motion resolves like a dissonance resolving into consonance. Whitney carries the low-order Pythagorean ratios (octave, fifth, fourth, third) into visual motion and hypothesises that ratio, interference and resonance laws work in vision as in music (same source).

Practical reading: N points on a ring, point k at angle = k * base_phase * t (or with a radius that also varies). At t = 0 and at periodic moments they align into a shape. In between they smear into spirals and rosettes.

## Visual language
- Many small dots or short lines, usually on a dark field, with saturated colour added.
- Slow bloom, scatter, then snap back into a clean figure. The tension and release is the whole show.
- Radial symmetry, spirals, rosettes, Lissajous-like weaves.
- Rhythm comes from the cycle time of the ratio, not from cuts.

## Live potential
Ideas below are design suggestions, not claims from sources.
- Kick: pulse the phase offset or trigger a "resolution" (reset to aligned state). Bass: ring radius or point size. Hi-hats: flicker, trail refresh or brief colour step.
- MIDI faders: ratio set (1:2, 2:3, 3:4, 4:5), point count, trail length, symmetry order. A key on the KeyStep could pick the ratio from the played interval (octave = 1:2, fifth = 2:3, fourth = 3:4), so the visual literally follows the chord.
- BPM: set the full cycle (return to alignment) to 1, 2 or 4 bars, so resolution lands on the downbeat.
- Layering: it works as additive points over video or feedback. Use it as a luma/alpha mask over footage. Send via Spout to Resolume or UE (Niagara particles as the points).
- Boring risk: if the ratio never changes, or the pattern is never allowed to dissolve. Needs tension phases. Also avoid too-uniform colour and static point size on a big screen.

## Variations
1. Chord mapper: MIDI notes set the ratios per ring, so a major chord shows a different figure than a minor one.
2. Detune drift: slightly off-integer ratios, so the resolution moment slowly precesses and never repeats exactly.
3. 3D rings in UE Niagara, with the camera dollying through the aligned moment.
4. Video-sampling: each point samples colour from a live input at its position.
5. Multi-ring polyrhythm: concentric rings with different ratios, aligned only every N bars.

## Facts
- Whitney's analog computer began as a WWII anti-aircraft gun director ([Socks-Studio](https://socks-studio.com/2010/11/25/john-whitney-inventor-and-father-of-computer-animation/)).
- IBM gave him its first artist-in-residence position, in 1966 ([Wikipedia](https://en.wikipedia.org/wiki/John_Whitney_(animator))).
- *Homage to Rameau* was named after Rameau's harmony treatise ([Visual Music Archive](https://visualmusicarchive.org/reading/r/digital-harmony.html)).
- *Matrix III* is built on one Lissajous figure applied symmetrically over the frame ([Light Cone](https://lightcone.org/en/film-1555-matrix-iii)).
- *Permutations* was shot from a black-and-white 2250 display, with colour added afterward in an optical printer ([Visual Music Archive](https://visualmusicarchive.org/works/w/permutations.html)).
- The book includes code examples ([Visual Music Archive](https://visualmusicarchive.org/reading/r/digital-harmony.html)).

## Visual references
- [Permutations (Visual Music Archive)](https://visualmusicarchive.org/works/w/permutations.html) — page on the 1966 film of coloured dot patterns.
- [Matrix III (Light Cone)](https://lightcone.org/en/film-1555-matrix-iii) — film record with the Lissajous description.
- [Matrix III analysis (Motion Design Archive)](https://luckydumpling.github.io/motion-archive/entries/john-whitney-matrix/) — embeds the film, notes ties to Processing and TouchDesigner.
- [Digital Harmony full text](https://archive.org/stream/DigitalHarmony_201611/Digital%20Harmony_djvu.txt) — the book with the theory.
- [Whitney overview (Socks-Studio)](https://socks-studio.com/2010/11/25/john-whitney-inventor-and-father-of-computer-animation/) — machine and film images.

## Lead image
NONE. Wikimedia Commons could not be opened in this session, so no free image was confirmed.

## Sources
- https://en.wikipedia.org/wiki/John_Whitney_(animator)
- https://visualmusicarchive.org/reading/r/digital-harmony.html
- https://visualmusicarchive.org/works/w/permutations.html
- https://lightcone.org/en/film-1555-matrix-iii
- https://archive.org/stream/DigitalHarmony_201611/Digital%20Harmony_djvu.txt
- https://socks-studio.com/2010/11/25/john-whitney-inventor-and-father-of-computer-animation/
- https://luckydumpling.github.io/motion-archive/entries/john-whitney-matrix/
