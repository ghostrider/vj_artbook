# Chladni Plate Patterns — research

## Origin
- Ernst Chladni published *Entdeckungen über die Theorie des Klanges* in 1787. He drew a bow over a sand-covered metal plate. The sand collects on the nodal lines, where the plate does not move. The book has 11 plates and 166 figures ([Wikipedia](https://en.wikipedia.org/wiki/Ernst_Chladni), [APS News](https://www.aps.org/apsnews/2017/07/first-experiments-chladni-figures)).
- Robert Hooke ran a bow over sand-covered metal in 1680, a century earlier. No record shows Chladni knew of it (same Wikipedia page).
- Chladni showed the figures to the Paris Academy in 1808. Napoleon was in the audience and proposed a prize for a mathematical explanation. Sophie Germain won it in 1816, the first woman to win the Academy's grand prize in mathematics. Her solution was incomplete ([Wikipedia](https://en.wikipedia.org/wiki/Ernst_Chladni), [thatsmaths](https://thatsmaths.com/2020/01/16/the-extraordinary-talent-and-superior-genius-of-sophie-germain/)).
- Hans Jenny (1904–1972) coined "cymatics". He put powders, pastes and liquids on plates driven by an oscillator. His first German volume appeared in 1967 ([Wikipedia: Cymatics](https://en.wikipedia.org/wiki/Cymatics)).

## Key works and artists
- **Nigel Stanford, "Cymatics" (album *Solar Echoes*, page (c) 2014)**: a music video built on sand on a Chladni plate, ferrofluid and a fire (Rubens) tube reacting to tones. The music stays on each note long enough for the shape to form ([nigelstanford.com](https://nigelstanford.com/cymatics)).
- **Hans Jenny, *Kymatic* (first volume 1967)**: the main reference for the idea that sound makes form. It influenced Alvin Lucier's *Queen of the South* ([Wikipedia: Cymatics](https://en.wikipedia.org/wiki/Cymatics)).
- **Real-time TouchDesigner build** by didgety, with a kick-driven audioreactive version ([didgety blog](https://didgety.github.io/blog/2023/chladni_plate_simulation)).
- **Bevy plate simulation** by azrlog, where the user places oscillators and walls ([azrlog.dev](https://azrlog.dev/posts/chladni)).

## How it works
- A thin plate has resonant modes. At a mode frequency the plate splits into regions moving up and down, separated by nodal lines that stay still. Sand is thrown off the moving regions and rests on the nodal lines.
- The usual shortcut for a square plate is the difference of two cosine products, cos(nπx/L)·cos(mπy/L) − cos(mπx/L)·cos(nπy/L). Setting it to zero traces the nodal lines. Integers n and m pick the mode ([didgety](https://didgety.github.io/blog/2023/chladni_plate_simulation)).
- Particle version: turn the field into a velocity or gradient map. Particles move down the gradient of |amplitude| toward the still lines. Respawn particles that leave the bounds ([didgety](https://didgety.github.io/blog/2023/chladni_plate_simulation)). A small repulsion keeps them spread out ([azrlog](https://azrlog.dev/posts/chladni)).
- Physical version: simulate a 128×128 grid where each cell is pulled by its neighbors. Cells can be free, pinned (a finger), or forced to oscillate (a bow). Chladni-like figures then appear without the closed formula ([azrlog](https://azrlog.dev/posts/chladni)).
- No simulation needed for a pattern layer: Houdini's Copernicus Chladni node mixes two waves (a and b) and outputs nodal lines, absolute value or signed value. It has threshold and smoothing controls ([SideFX docs](https://sidefx.com/docs/houdini/nodes/cop/chladni.html)).

## Visual language
- Thin glowing or dusty lines on black, symmetric and often square. Dense lattices at high modes, a few bold curves at low modes.
- Two looks: the pure line pattern (a shader, resolution-independent), and the particle look where grains drift and settle. The settling is the motion payoff.
- Between modes, the figure dissolves and then re-forms. This is where the drama is.
- Color suggestions (my own judgment): one hue for the lines, a second for particle velocity or the signed amplitude.

## Live potential
- **Mode numbers n, m** map to two faders or knobs. Quantize them to integers for snaps, or let them go non-integer for in-between morphs (the Houdini node uses non-integer frequencies).
- **Kick** triggers a mode change or a shake: raise particle energy or add noise to the velocity so the grains scatter, then let them re-settle. The didgety build drives its pulse from the kick, read just below 100 Hz ([didgety](https://didgety.github.io/blog/2023/chladni_plate_simulation)).
- **Hi-hats** can jitter line thickness or add sparkle. Bass level can set the amplitude.
- **BPM**: step through a list of (n, m) pairs per bar. Use the BeatStep Pro to pick the pair from pads.
- **Layering**: the line pattern works as a mask or displacement map for video in Resolume. It also works as an emitter field in Niagara. Run the shader output over Spout.
- **Boring risks**: a static figure for too long, or automation that flashes constantly (didgety notes flashing from automation). Resolution limits show on a big screen if particles are too few, so use many particles or the analytic shader.

## Visual references
- [Chladni Figures (1787), Public Domain Review](https://publicdomainreview.org/collection/chladni-figures-1787/) — ten engraved plates from Chladni's original book showing nodal-line figures
- [Chladni plate interference surfaces, Paul Bourke](https://paulbourke.net/geometry/chladni) — grids of nodal-line and amplitude images for square and circular plates, modes 1 to 5
- [Bowing a Chladni plate, Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Bowing_chladni_plate.png) — public-domain 1879 drawing of a plate bowed so that sand forms figures
- [luciopaiva/chladni on GitHub](https://github.com/luciopaiva/chladni) — particle simulation; the README has a screenshot and links a live demo
- [CYMATICS: Science Vs. Music - Nigel Stanford](https://www.youtube.com/watch?v=Q3oItpVa9fs) — the music video with real sand-plate, ferrofluid and fire experiments (YouTube, Nigel John Stanford)
- [Good Vibrations: Chladni Plate Demonstration - Lockheed Martin](https://www.youtube.com/watch?v=_uTPiBlcZpg) — a plate demonstration where patterns change as the vibration changes (YouTube, Univ. of Kentucky Pigman College of Engineering)
- [Sand Vibration Patterns - Chladni Plate](https://www.youtube.com/watch?v=YedgubRZva8) — sand on a rectangular metal plate at resonant frequencies (YouTube, SMUPhysics)

## Variations
1. **Mode sequencer**: a BeatStep Pro pattern picks (n, m) pairs, and the figure snaps on each step with a settle time tied to BPM.
2. **Non-square plates**: circular or hexagonal boundaries, or a plate shaped like the venue's logo. Needs a numerical solver such as the grid method in [azrlog](https://azrlog.dev/posts/chladni).
3. **Touch the plate**: MIDI notes pin or drive cells, like the finger and bow constraints in [azrlog](https://azrlog.dev/posts/chladni).
4. **Chladni as displacement**: use the field to warp other video or a 3D mesh in UE5.
5. **Stacked plates**: two or three fields at different scales and colors, with one mode set per audio band.

## Facts
- Hooke made sand figures in 1680, about a century before Chladni ([Wikipedia](https://en.wikipedia.org/wiki/Ernst_Chladni)).
- Sophie Germain won the Paris Academy's prize in 1816 for work stemming from Chladni's figures, though her solution was incomplete ([thatsmaths](https://thatsmaths.com/2020/01/16/the-extraordinary-talent-and-superior-genius-of-sophie-germain/)).
- Stanford kept the music on each note long enough for the sand to form its shape, a constraint that applies to live sets too ([nigelstanford.com](https://nigelstanford.com/cymatics)).
- The two-cosine formula is symmetric in n and m, so swapping them flips the sign of the field but keeps the same nodal lines ([didgety](https://didgety.github.io/blog/2023/chladni_plate_simulation); the symmetry follows from the formula).
- Houdini ships a Chladni node since version 20.5 ([SideFX](https://sidefx.com/docs/houdini/nodes/cop/chladni.html)).

## Sources
- [didgety: Chladni plate simulation (TouchDesigner)](https://didgety.github.io/blog/2023/chladni_plate_simulation) (opened, matches)
- [azrlog: Chladni (Bevy)](https://azrlog.dev/posts/chladni) (opened, matches)
- [SideFX Houdini: Chladni Cymatic Patterns](https://sidefx.com/docs/houdini/nodes/cop/chladni.html) (opened, matches)
- [Wikipedia: Ernst Chladni](https://en.wikipedia.org/wiki/Ernst_Chladni)
- [APS News: first experiments on Chladni figures](https://www.aps.org/apsnews/2017/07/first-experiments-chladni-figures)
- [thatsmaths: Sophie Germain](https://thatsmaths.com/2020/01/16/the-extraordinary-talent-and-superior-genius-of-sophie-germain/)
- [Wikipedia: Cymatics](https://en.wikipedia.org/wiki/Cymatics)
- [Nigel Stanford: Cymatics](https://nigelstanford.com/cymatics)
- [Public Domain Review: Chladni Figures](https://publicdomainreview.org/collection/chladni-figures-1787/)
- [Paul Bourke: Chladni plate interference surfaces](https://paulbourke.net/geometry/chladni)
- [Wikimedia Commons: Bowing chladni plate](https://commons.wikimedia.org/wiki/File:Bowing_chladni_plate.png)
- [luciopaiva/chladni](https://github.com/luciopaiva/chladni)
