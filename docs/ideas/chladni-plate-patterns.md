---
tags:
  - physics
  - cymatics
  - particles
  - shader
---

# Chladni Plate Patterns

> Sand runs from the shaking parts of a plate to the still ones, and the lines it leaves behind turn into a light show.

## The idea

A thin plate has resonant modes. At a mode frequency the plate splits into regions that move up and down, separated by nodal lines that stay still. Sand is thrown off the moving regions and comes to rest on the nodal lines. Change the frequency and the figure dissolves, then forms again as a new one.

Ernst Chladni published the first systematic study in 1787. He drew a bow over a sand-covered metal plate and recorded what the sand did ([Wikipedia](https://en.wikipedia.org/wiki/Ernst_Chladni), [APS News](https://www.aps.org/apsnews/2017/07/first-experiments-chladni-figures)). On a club screen the idea gives you a strict, symmetric image that still has a physical event inside it. A kick scatters the grains. Over a beat or two they drift back and settle on a new set of lines. The settling is the motion payoff.

Two looks are available. The pure line pattern is a shader and stays sharp at any resolution. The particle look adds grains that drift and settle. Use both together, or use the line shader alone when the budget is tight.

## Origins and references

- **Robert Hooke, 1680**: ran a bow over sand-covered metal a century before Chladni. No record shows Chladni knew of it ([Wikipedia](https://en.wikipedia.org/wiki/Ernst_Chladni)).
- **Ernst Chladni, *Entdeckungen über die Theorie des Klanges*, 1787**: 11 plates and 166 figures ([Wikipedia](https://en.wikipedia.org/wiki/Ernst_Chladni), [APS News](https://www.aps.org/apsnews/2017/07/first-experiments-chladni-figures)).
- **Paris Academy, 1808 and 1816**: Chladni showed the figures there, Napoleon proposed a prize for a mathematical explanation, and Sophie Germain won it in 1816. Her solution was incomplete ([Wikipedia](https://en.wikipedia.org/wiki/Ernst_Chladni), [thatsmaths](https://thatsmaths.com/2020/01/16/the-extraordinary-talent-and-superior-genius-of-sophie-germain/)).
- **Hans Jenny, *Kymatic*, first German volume 1967**: coined "cymatics" and worked with powders, pastes and liquids on plates driven by an oscillator. It influenced Alvin Lucier's *Queen of the South* ([Wikipedia: Cymatics](https://en.wikipedia.org/wiki/Cymatics)).
- **Nigel Stanford, "Cymatics"** (album *Solar Echoes*): a music video built on sand on a Chladni plate, ferrofluid and a fire tube reacting to tones ([nigelstanford.com](https://nigelstanford.com/cymatics)).
- **didgety, Chladni plate simulation in TouchDesigner**: a real-time build with a kick-driven audioreactive version ([didgety](https://didgety.github.io/blog/2023/chladni_plate_simulation)).
- **azrlog, Chladni in Bevy**: a plate simulation where the user places oscillators and walls ([azrlog](https://azrlog.dev/posts/chladni)).
- **SideFX Houdini, Chladni node**: mixes two waves and outputs nodal lines, absolute or signed values, with threshold and smoothing ([SideFX docs](https://sidefx.com/docs/houdini/nodes/cop/chladni.html)).

## Visual language

Thin glowing or dusty lines on black. The plate is square and the figure is symmetric. At low modes you get a few bold curves. At high modes you get a dense lattice of diamonds and loops.

Motion lives in the transitions. On a change the lines slide and reconnect, and the grains jump, smear into a haze, and drift onto the new lines over one or two beats. Still grains sit on lines. Nervous grains crowd the open areas. That difference is what makes the physics read.

Color is restrained. Sand is warm bone white (#EDE3D0) on near-black (#050507). The nodal line carries one accent, amber (#FFB547), which can rotate to cyan (#4FD1FF) or acid green (#B7FF5A) per phrase. The open areas carry a very dim signed tint, cold blue where the plate moves up and wine red where it moves down. The mood is scientific, calm and exact, with a snap on every change.

## See it

- [Chladni Figures (1787), Public Domain Review](https://publicdomainreview.org/collection/chladni-figures-1787/) — ten engraved plates from Chladni's original book showing nodal-line figures
- [Chladni plate interference surfaces, Paul Bourke](https://paulbourke.net/geometry/chladni) — grids of nodal-line and amplitude images for square and circular plates, modes 1 to 5
- [Bowing a Chladni plate, Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Bowing_chladni_plate.png) — public-domain 1879 drawing of a plate bowed so that sand forms figures
- [luciopaiva/chladni on GitHub](https://github.com/luciopaiva/chladni) — particle simulation; the README has a screenshot and links a live demo
- [CYMATICS: Science Vs. Music - Nigel Stanford](https://www.youtube.com/watch?v=Q3oItpVa9fs) — the music video with real sand-plate, ferrofluid and fire experiments
- [Good Vibrations: Chladni Plate Demonstration - Lockheed Martin](https://www.youtube.com/watch?v=_uTPiBlcZpg) — a plate demonstration where patterns change as the vibration changes
- [Sand Vibration Patterns - Chladni Plate](https://www.youtube.com/watch?v=YedgubRZva8) — sand on a rectangular metal plate at resonant frequencies

## Code design

The field is analytic. For a square plate, with a = n·π and b = m·π, F = cos(a·x)·cos(b·y) + s·cos(b·x)·cos(a·y). The value s = -1 is the classic formula ([didgety](https://didgety.github.io/blog/2023/chladni_plate_simulation), [Paul Bourke](https://paulbourke.net/geometry/chladni)). The value s = +1 gives a second family. The gradient is also analytic, so it is cheap and noise-free. Two mode sets, A and B, are mixed with an eased morph value e, so the lines slide and reconnect instead of cutting.

Lines get constant pixel width by dividing |F| by the length of its gradient. Sand grains move down the gradient of |F| toward the still lines, with jitter scaled by |F| and damping each frame. A kick adds a one-frame shake. A feedback buffer with decay 0.82 turns the scatter into haze.

Render order: field, particle simulation, splat, trail feedback, composite, post. Everything above the black background is Add blend. Two pitfalls matter. With s = -1 and n = m the plate is blank. Swapping n and m only flips the sign of F, so mixing the pair passes through zero and flashes. Skip the morph for a swapped pair, or multiply the target by -1.

=== "Resolume Wire"
    Wire cannot run the particle passes, because it has no vertex shaders. Build the line-and-tint look as a single-pass ISF source in Wire ([Resolume ISF support](https://resolume.com/support/en/isf)) and export it to Arena.

    - Beat counter into a `Counter` (wrap 8) and an `Array` lookup of (n, m, s) pairs. Two lookups, current and next, feed the ISF inputs.
    - Audio analysis low band into an envelope (attack 5 ms, release 250 ms) drives shake and tint ([Resolume forum](https://www.resolume.com/forum/viewtopic.php?p=91404) for turning FFT arrays into textures).
    - Blend the ISF output with a Spout receiver that carries the particle layer from another tool, then add a blur-and-add bloom chain.
    - Keep the ISF single pass. Multipass support has been version-dependent according to forum reports. Use Wire's own feedback node if you need trails.
    - In Arena, fake sand with a copy of the source at 3 px line width, run through Noise and Threshold, Add at 50 %.

=== "Unreal Engine 5.7"
    - A Material Function `MF_Chladni` returns F and its gradient from p, n, m and s. Use it in a plate material (unlit, additive) and in Niagara.
    - The plate material mixes sets A and B by the morph value. It turns |F| over gradient length into a pixel-width line using DDX/DDY of p.
    - Niagara: a GPU compute emitter with 262,144 grains. A Scratch Pad module evaluates the field and adds pull and jitter to Velocity. Drag is about 6. Grains that leave the plate respawn at a random position. Render with an unlit additive sprite.
    - Expose User.N, User.M, User.S, User.Morph, User.Shake, User.Pull, User.LineWidth and User.TintAmount through a Remote Control preset on OSC and MIDI.
    - Post-process: bloom intensity 1.2, threshold 0.75, fixed manual exposure (auto exposure pumps on kicks), film grain 0.03, vignette 0.25.
    - Output over Spout (for example Off World Live's toolkit) or NDI at the LED processor's native resolution.

=== "GODOT"
    - A full-viewport `ColorRect` with a `canvas_item` shader draws field, lines and tint. Uniforms hold both mode sets, morph, line width and tint. `fwidth(p)` converts distance to pixels.
    - Sand is a `GPUParticles2D` with 262,144 grains and a `shader_type particles;` material. In `process()` it computes F and the gradient, updates `VELOCITY` with pull, jitter and damping, and respawns grains that leave the plate. Draw with a 1 px texture and Add blend.
    - Trails: render particles into a `SubViewport` and feed back its previous frame times 0.82. A simpler fallback clears never and draws a black rect at alpha 0.18 each frame. It works in 8-bit and leaves faint ghosts.
    - Glow through `WorldEnvironment` at intensity 1.2 and HDR threshold 0.75.
    - Map MIDI with `InputEventMIDI` to the uniforms. For Spout, use a community addon if available, or capture via NDI.

## On stage

**Audio reactivity.** Only the kick is fast. Everything else breathes. The didgety build reports constant flashing when automation drives too much ([didgety](https://didgety.github.io/blog/2023/chladni_plate_simulation)), so keep the rest slow and gated.

| Band | Drives | Smoothing |
|------|--------|-----------|
| Kick, 40–100 Hz | shake impulse, every 4th kick advances the mode | attack 2 ms, release 180 ms |
| Bass, 60–250 Hz | tint amount 0.15–0.5 | attack 50 ms, release 400 ms |
| Hi-hat, 6–12 kHz | line width +0.5 px jitter, sparkle on the fastest grains | attack 1 ms, release 60 ms |
| Mid, 300 Hz–2 kHz | grain jitter 0.02–0.2 | attack 100 ms, release 800 ms |

**MIDI.** BeatStep Pro pads 1–8 pick one (n, m, s) set from the list. Two KeyStep 37 knobs can set n and m directly, quantised to integers when the knob stops. Further knobs: morph time (0 snaps on drops), settle (pull strength), line width. A fader sets the symmetry value s, moved slowly. Shake follows the kick envelope, plus a fader for builds.

**Sequencer.** One set per bar, for example (1,2,-1), (1,3,-1), (2,3,-1), (1,4,+1), (2,5,-1), (3,5,-1), (3,7,+1), (4,7,-1). Low modes in intros, high modes in builds. Change modes at most once per bar in dense parts. Nigel Stanford held each note long enough for the figure to form ([nigelstanford.com](https://nigelstanford.com/cymatics)), and the same limit applies to a live set.

**Structure.**

- Build: shake grows from 0 to 1 over the last 8 bars while n and m climb, so the sand never settles.
- Drop: morph time 0, shake 1.0 on the hit, then settle into a bold low mode such as (1,3) within 2 beats. Switch the line accent colour.
- Break: sand down to 30 %, lines only, with a slow non-integer drift of m so the figure breathes.

**Layering.** The line layer works as a Luma Key source over video, or as a mask so the video shows only inside the nodal lines. The field can also feed a Displace effect.

**Tips.**

- Keep the black at #050507, not pure black, so LED walls do not show dead modules.
- Cap peak line brightness near 85 % before bloom. Do not lift the tint above 0.5 or the figure loses its edges.
- Do not add chromatic aberration. It smears 1 px lines.
- On low-resolution walls use 2–3 px lines and 2 × 2 px grains.
- On a 32:9 strip, tile two or three plates side by side and mirror every second one so the lines join.

## Variations

1. **Mode sequencer**: a BeatStep Pro pattern picks (n, m) pairs, and the figure snaps on each step with a settle time tied to BPM.
2. **Non-square plates**: circular or hexagonal boundaries, or a plate shaped like the venue's logo. This needs a numerical solver such as the grid method in [azrlog](https://azrlog.dev/posts/chladni).
3. **Touch the plate**: MIDI notes pin or drive cells, like the finger and bow constraints in [azrlog](https://azrlog.dev/posts/chladni).
4. **Chladni as displacement**: use the field to warp other video or a 3D mesh in UE5.
5. **Stacked plates**: two or three fields at different scales and colors, with one mode set per audio band.

## Did you know?

- Hooke made sand figures in 1680, about a century before Chladni ([Wikipedia](https://en.wikipedia.org/wiki/Ernst_Chladni)).
- Sophie Germain won the Paris Academy's prize in 1816 for work stemming from Chladni's figures. Her solution was incomplete ([thatsmaths](https://thatsmaths.com/2020/01/16/the-extraordinary-talent-and-superior-genius-of-sophie-germain/)).
- The two-cosine formula is symmetric in n and m, so swapping them flips the sign of the field but keeps the same nodal lines ([didgety](https://didgety.github.io/blog/2023/chladni_plate_simulation)).
- Houdini ships a Chladni node since version 20.5 ([SideFX](https://sidefx.com/docs/houdini/nodes/cop/chladni.html)).

## Sources

- [didgety: Chladni plate simulation (TouchDesigner)](https://didgety.github.io/blog/2023/chladni_plate_simulation)
- [azrlog: Chladni (Bevy)](https://azrlog.dev/posts/chladni)
- [SideFX Houdini: Chladni Cymatic Patterns](https://sidefx.com/docs/houdini/nodes/cop/chladni.html)
- [Wikipedia: Ernst Chladni](https://en.wikipedia.org/wiki/Ernst_Chladni)
- [APS News: first experiments on Chladni figures](https://www.aps.org/apsnews/2017/07/first-experiments-chladni-figures)
- [thatsmaths: Sophie Germain](https://thatsmaths.com/2020/01/16/the-extraordinary-talent-and-superior-genius-of-sophie-germain/)
- [Wikipedia: Cymatics](https://en.wikipedia.org/wiki/Cymatics)
- [Nigel Stanford: Cymatics](https://nigelstanford.com/cymatics)
- [Public Domain Review: Chladni Figures](https://publicdomainreview.org/collection/chladni-figures-1787/)
- [Paul Bourke: Chladni plate interference surfaces](https://paulbourke.net/geometry/chladni)
- [Wikimedia Commons: Bowing chladni plate](https://commons.wikimedia.org/wiki/File:Bowing_chladni_plate.png)
- [luciopaiva/chladni](https://github.com/luciopaiva/chladni)
- [luciopaiva: live demo](https://luciopaiva.com/chladni)
- [Resolume: ISF support](https://resolume.com/support/en/isf)
- [Resolume forum: FFT arrays to textures](https://www.resolume.com/forum/viewtopic.php?p=91404)
