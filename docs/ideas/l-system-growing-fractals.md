---
tags:
  - fractal
  - plants
  - grammar
  - generative
---

# L-System Growing Fractals

> A few rewriting rules, one bar at a time: a glowing plant forks, sways and collapses back into its root on the beat.

![The classic L-system fractal plant (25 degree branching) drawn as line branches](https://upload.wikimedia.org/wikipedia/commons/thumb/4/44/Fractal-plant.svg/1280px-Fractal-plant.svg.png)
*The classic L-system fractal plant (25 degree branching) drawn as line branches. Image: [Sakurambo](https://commons.wikimedia.org/wiki/File:Fractal-plant.svg), [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/).*

## The idea

An L-system is a grammar. It has an alphabet, a start string (the axiom) and rewriting rules. In every step all symbols are replaced at once ([Wikipedia: L-system](https://en.wikipedia.org/wiki/L-system)). To draw the result, a "turtle" reads the string: draw a line, turn by an angle, save or restore position with brackets. The brackets make branches.

Aristid Lindenmayer, a Hungarian theoretical biologist and botanist at Utrecht, introduced the system in 1968 to describe how simple multicellular organisms grow. Drawing was not the goal ([Wikipedia: L-system](https://en.wikipedia.org/wiki/L-system)). In 1990 Przemyslaw Prusinkiewicz and Lindenmayer published *The Algorithmic Beauty of Plants*, which covers tree modelling, phyllotaxis, plant organs and animation ([Wikipedia: ABOP](https://en.wikipedia.org/wiki/The_Algorithmic_Beauty_of_Plants)).

On a club screen the grammar gives you a natural clock. Each iteration is a generation of twigs, so each generation can be one beat or one bar. A tree that finishes a depth every bar, sways between hits, and collapses on a phrase change keeps the room oriented in time. It looks alive and it counts for you.

## Origins and references

- Aristid Lindenmayer, 1968: L-systems for cell development in simple organisms ([Wikipedia: L-system](https://en.wikipedia.org/wiki/L-system)).
- Prusinkiewicz and Lindenmayer, *The Algorithmic Beauty of Plants* (Springer, 1990): out of print, free online ([algorithmicbotany.org](http://algorithmicbotany.org/papers/#abop)).
- Fractint: the fractal program that popularised turtle-graphics drawing of L-systems ([Wikipedia: L-system](https://en.wikipedia.org/wiki/L-system)).
- Michael Hansmeyer, "L-Systems in Architecture" (2003): stochastic, parametric and modular L-systems for architectural form, output as Maya scripts ([project page](https://www.michael-hansmeyer.com/l-systems)).
- MilkITt, "unnamed tree ∞/∞ 3d": a 20-minute animation that grows a different tree each time, shared on the Grasshopper3D forum by Stefan K ([Grasshopper3D](https://grasshopper3d.com/photo/albums/growing-tree-animation)).
- Paul Bourke, L-System User Notes: turtle symbol table and many tested rules ([Bourke](https://paulbourke.net/fractals/lsys/)).

## Visual language

Thin line trees and ferns branch into ever finer twigs. Self-similarity is strict, or slightly jittered when the rules are stochastic. The same grammar with a different angle gives other species: the dragon, Koch and Sierpinski curves read as geometric lace or circuitry.

Motion is growth outward from the root, then sway. Rhythm comes from generations.

The design: a near-black screen (#030605) and a thin glowing line that rises from the bottom edge. Old wood is deep teal (#0B2A24), main branches mid green (#1FA67A), the finest twigs mint (#7CFFB2). Fresh growth flashes white-mint (#F2FFE8) and cools over one beat, so the audience can see which twigs are new. Magenta sparks (#FF4FA3) bloom at tips on hi-hats and are the only warm colour. Two other palettes: "Ember" for bushes, "Circuit" (blues and cyan) for dragon, Koch and Sierpinski.

Nothing pops. New twigs unfold out of their parents with eased growth, and the camera pulls back to keep the plant in frame. The mood is a bioluminescent plant in time-lapse.

## See it

- [Computerphile: L Systems, Creating Plants from Simple Rules](https://www.youtube.com/watch?v=puwhf-404Xc) — explainer video showing plants grown from rewriting rules (channel: Computerphile).
- [unnamed tree ∞/∞ 3d](https://youtu.be/7_vKj9J7i9g) — excerpt of a 20-minute growing-tree animation (author: MilkITt).
- [Grasshopper3D: growing tree animation](https://grasshopper3d.com/photo/albums/growing-tree-animation) — page for the exhibition animation of generated growing trees.
- [Hansmeyer: L-Systems in Architecture](https://www.michael-hansmeyer.com/l-systems) — Sierpinski, stochastic and parametric structures as images.
- [Commons: Dragon trees](https://commons.wikimedia.org/wiki/File:Dragon_trees.jpg) — fractal trees from a varied dragon-curve L-system.
- [Commons: L-system Tree05](https://commons.wikimedia.org/wiki/File:L-system_Tree05.jpg) — a rendered L-system tree.

## Code design

The grammar runs on the CPU, only when something changes. Each symbol carries a birth generation and a parent index, so a new generation can grow out of the old one instead of replacing it. The turtle turns the string into a segment list (positions, depth, path distance from the root, birth generation). The GPU draws one instanced quad per segment. The reveal, the generation easing and the sway all happen in the vertex stage.

Key techniques from the design:

- **Birth tags.** Stochastic choices use `hash(seed, symbolIndex, n)`, not a running random generator. The same seed gives the same tree, and the old branches stay put when a generation is added.
- **Budget first.** Symbol counts follow a linear recurrence, so compute them before building any string.
- **Smooth growth.** The stage is fractional, `gen = n + f`, with smootherstep easing. New twigs take length `L · e`. For edge-replacement curves (dragon, Koch) each vertex stores a rest position, so the dragon folds out of the previous dragon. TouchDesigner's LSystem SOP offers the same idea with Continuous Length and Angles ([Derivative: LSystem SOP](https://docs.derivative.ca/LSystem_SOP)).
- **Reveal by path distance.** A growth front sweeps from the root along the branches, GPU only. Branches of the same order grow together.
- **Camera fit and sway.** The bounding box is eased with a 400 ms damper. Sway rotates vertices around the root, so the trunk barely moves and the tips swing.

Segment counts per generation (drawing symbols before merging runs). These were derived from the rule recurrences, not looked up. Check them in your own tool.

| species | rules and angle | gen 4 | gen 5 | gen 6 | gen 7 | gen 8 | recommended maxGen |
|---------|-----------------|-------|-------|-------|-------|-------|--------------------|
| fractal plant | `-X`; `X → F+[[X]-X]-F[-FX]+X`, `F → FF`; 25° | 360 | 1,488 | 6,048 | 24,384 | 97,920 | 7 (8 for reveal only) |
| binary tree | `0`; `1 → 11`, `0 → 1[0]0`; 45° | 48 | 112 | 256 | 576 | 1,280 | 12 (4096 leaves) |
| dragon | `F`; `F → F+G`, `G → F−G`; 90° | 16 | 32 | 64 | 128 | 256 | 15 |
| Koch (square) | `F`; `F → F+F−F−F+F`; 90° | 625 | 3,125 | 15,625 | 78,125 | 390,625 | 6 (7 for reveal only) |
| Sierpinski | `F−G−G`; `F → F−G+F+G−F`, `G → GG`; 120° | 243 | 729 | 2,187 | 6,561 | 19,683 | 9 |

The rules come from [Wikipedia: L-system](https://en.wikipedia.org/wiki/L-system). A weed rule (22.5°) comes from [Bourke](https://paulbourke.net/fractals/lsys/).

=== "Resolume Wire"
    Wire has no vertex shaders: the [Resolume ISF page](https://resolume.com/support/en/isf) says Wire (version 7.22) does not support them because it works in 2D. Do not try string rewriting in Wire.

    - **Option A, recommended.** Wire as receiver and control patch. A `Spout Receiver` takes the engine frame, a `Blend` node adds a feedback branch (scale 1.003, multiply 0.88) as afterglow, then bloom and output. The control side counts bars (counter, wrap 16), adds one generation per bar, collapses on bar 16 and sends gen, grow, species and seed to the engine over OSC.
    - **Option B, Wire-only fold-tree.** A single-pass ISF shader for the symmetric binary tree. Per pixel it loops over the depth, takes the distance to the trunk segment, mirrors `p.x = abs(p.x)`, rotates and rescales by 0.72. Growth draws segment i up to `clamp(grow · depth − i, 0, 1)`. A branch id adds jitter. It is cheap but limited to binary trees, and it is not a real L-system. Use it as a calm variant.

=== "Unreal Engine 5.7"
    - A C++ `ULSystemComponent` holds the rules (a DataAsset per species), rewrites with birth tags, runs the turtle in an async task and fills the segment data.
    - **Flat, recommended for VJ.** A GPU Niagara system takes the segment array, computes the rest/true lerp, reveal and sway in a Scratch Pad, and renders sprites aligned along each segment with an additive unlit material. A second emitter makes tips and sparks.
    - **3D hero tree.** Spline mesh chains for fewer than about 300 branches. For larger trees, a Dynamic Mesh tube with World Position Offset for the reveal.
    - Post: bloom 1.4 with threshold 0.6, film grain 0.02, vignette 0.25. Anti-aliasing off or FXAA, because TSR smears thin moving lines. Motion blur off, manual exposure. An orthographic CineCamera follows the damped bounding box.
    - Remote Control preset exposes Grow, Depth, Spread, Sway, Jitter, Species, Seed, Flash and EchoAmount over OSC and MIDI. Output by Spout (a third-party plugin) or NDI.

=== "GODOT"
    - A `Node2D` script holds the rules as `Resource` files. GDScript is enough for rewriting on change up to about 20k segments. Use C# or a GDExtension for per-frame re-turtles.
    - Lines: a `MultiMeshInstance2D` with a quad mesh. Set `use_colors` and `use_custom_data` before `instance_count`, as the docs require ([Godot MultiMesh](https://docs.godotengine.org/en/stable/classes/class_multimesh.html)). A `canvas_item` shader reads the per-instance data, applies reveal, ease and sway, and draws an anti-aliased core with a halo.
    - Tips and sparks: `GPUParticles2D`, additive, pulsed on hi-hats.
    - Echo: two `SubViewport` nodes ping-pong at half resolution (zoom 1.003, decay 0.88). Post: `WorldEnvironment` glow, intensity 1.4.
    - Control: `InputEventMIDI` for faders and pads, an OSC addon for the Resolume clock, Spout or NDI out.

## On stage

**Audio.** Geometry changes come from the clock, never from raw audio. Audio-driven generation jumps pop, which is the main failure here. Audio touches brightness, width, colour and small angles.

| band | drives | smoothing |
|------|--------|-----------|
| kick, 40–100 Hz onset | flash 0.35, width pulse +25 % | attack 2 ms, release 180 ms |
| bass, 60–250 Hz | branch angle ± 3° (or extra sway) | attack 100 ms, release 700 ms |
| mids, 300 Hz–2 kHz | up to +0.1 on the colour ramp, more tip colour | attack 150 ms, release 1 s |
| hi-hats, 6–12 kHz onset | 32 sparks at random tips, seed +1 if jitter > 0 | spark life 400 ms, max 8 per beat |
| bar clock | +1 generation per bar, collapse and new species every 16 bars | none |

**Clock.** One generation per bar: the growth eases in over the first beat and the plant rests for three beats while it sways. Default phrase of 16 bars: bar 1 the seedling rises, bars 2–7 one generation per bar, bars 8–15 hold with breathing and sparks, bar 16 collapse (grow 1 to 0 over 2 beats) and a new species.

**Controls.**

| parameter | what it does | mapping |
|-----------|--------------|---------|
| grow | growth front from the root | fader; auto 0 to 1 on phrase start |
| depth | generation, eased between integers | KeyStep knob; BPM clock; pad = "next generation now" |
| spread | branch angle, tight spire to wide fan | knob; bass adds ± 3° |
| sway | wind on the canopy | knob; LFO 0.25 Hz |
| jitter | stochastic weight and angle noise | knob; hi-hat increments seed |
| species | rules, angle, palette | BeatStep Pro pads 1–6; held "cut" pad = hard switch |

**Layering.** All light layers use Add: crossing branches should brighten. Add a feedback echo at 30 percent (50 percent on breaks, 0 on hard cuts and during collapse, or the old species ghosts into the new one). Use the plant, in grey before post, as a Luma Key or Track Matte over video in Resolume. For a mask, use thick lines and skip the halo.

**Moments.**

- Drop: hard cut to a new species at maxGen minus 1, grow 0 to 1 in one beat, echo off, flash 1.0, then one generation per 2 beats.
- Break: collapse to generation 3, regrow one generation per 2 bars, echo 50 percent, sway 6°, Ember or Circuit palette.

**Tips.**

- LED walls: lines thinner than 1.5 output pixels flicker while swaying. Raise the width instead of adding contrast. If the dark trunk disappears, lift it to #12403A.
- Render at the wall's native resolution. A downscaled 4K render loses the twigs.
- A tree is tall. On 32:9 strips, plant a forest of 3 to 5 trees, or lay a dragon curve horizontally. Do not stretch one tree.
- Pitfalls: building a string before checking its count, a running random generator for stochastic rules, TSR/TAA on thin lines, feedback left on during species changes, generation steps without easing.

## Variations

- **Beat-grown forest.** Several L-systems with different angles, each depth tied to a different drum voice.
- **Angle morph.** Slide the angle from 25° to 90° over 8 bars, so the skeleton changes from organic to crystalline. It needs a per-frame re-turtle, so keep the generation at 6 or lower.
- **Stochastic seeds on hi-hats.** Each hit re-rolls branches for a shimmering canopy that never repeats.
- **3D growth in Unreal.** Bake the turtle path to splines or instanced meshes and grow along the spline with a mask.
- **Wind and decay.** Reverse the process on breakdowns: branches retract.

## Did you know?

- L-systems apply all rules in parallel at each step, unlike standard formal grammars, which apply one rule at a time ([Wikipedia: L-system](https://en.wikipedia.org/wiki/L-system)).
- Lindenmayer's algae system (A → AB, B → A) produces strings whose lengths are Fibonacci numbers ([Wikipedia: L-system](https://en.wikipedia.org/wiki/L-system)).
- *The Algorithmic Beauty of Plants* is free as a PDF, 17 MB in high quality or 4 MB in low ([algorithmicbotany.org](http://algorithmicbotany.org/papers/#abop)).
- Hansmeyer made his 2003 L-system forms with a custom Excel/VBA program that output Maya scripts ([project page](https://www.michael-hansmeyer.com/l-systems)).

## Sources

- [Wikipedia: L-system](https://en.wikipedia.org/wiki/L-system)
- [Wikipedia: The Algorithmic Beauty of Plants](https://en.wikipedia.org/wiki/The_Algorithmic_Beauty_of_Plants)
- [algorithmicbotany.org: ABOP](http://algorithmicbotany.org/papers/#abop)
- [Hansmeyer: L-Systems in Architecture](https://www.michael-hansmeyer.com/l-systems)
- [Grasshopper3D growing tree animation](https://grasshopper3d.com/photo/albums/growing-tree-animation)
- [Computerphile video](https://www.youtube.com/watch?v=puwhf-404Xc)
- [unnamed tree ∞/∞ 3d (MilkITt)](https://youtu.be/7_vKj9J7i9g)
- [Commons: Fractal-plant.svg](https://commons.wikimedia.org/wiki/File:Fractal-plant.svg)
- [Commons: Dragon trees](https://commons.wikimedia.org/wiki/File:Dragon_trees.jpg)
- [Commons: L-system Tree05](https://commons.wikimedia.org/wiki/File:L-system_Tree05.jpg)
- [Paul Bourke: L-System User Notes](https://paulbourke.net/fractals/lsys/)
- [Derivative: LSystem SOP](https://docs.derivative.ca/LSystem_SOP)
- [Resolume ISF](https://resolume.com/support/en/isf)
- [Godot MultiMesh](https://docs.godotengine.org/en/stable/classes/class_multimesh.html)
