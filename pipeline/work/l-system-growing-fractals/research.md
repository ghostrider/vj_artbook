# L-System Growing Fractals — research

## Origin
- Aristid Lindenmayer, a Hungarian theoretical biologist and botanist at Utrecht, introduced L-systems in 1968 to describe how simple multicellular organisms (such as cyanobacteria filaments) grow. Later work extended them to higher plants ([Wikipedia: L-system](https://en.wikipedia.org/wiki/L-system)).
- In 1990 Przemyslaw Prusinkiewicz and Lindenmayer published *The Algorithmic Beauty of Plants* (Springer). It is out of print but free online. It covers tree modelling, phyllotaxis, plant organs and animation ([Wikipedia: ABOP](https://en.wikipedia.org/wiki/The_Algorithmic_Beauty_of_Plants), [algorithmicbotany.org](http://algorithmicbotany.org/papers/#abop)).
- Drawing L-systems with turtle graphics (Logo-style) became common; the fractal program Fractint used this approach ([Wikipedia: L-system](https://en.wikipedia.org/wiki/L-system)).

## Key works and artists
- *The Algorithmic Beauty of Plants* (1990): the reference book, with computer-generated plant images made from simple rules ([Wikipedia](https://en.wikipedia.org/wiki/The_Algorithmic_Beauty_of_Plants)). Free PDF: [algorithmicbotany.org](http://algorithmicbotany.org/papers/#abop).
- Michael Hansmeyer, "L-Systems in Architecture" (2003): uses stochastic, parametric and modular L-systems to generate architectural form and trusses, output as Maya scripts ([project page](https://www.michael-hansmeyer.com/l-systems)).
- "unnamed tree ∞/∞ 3d" by MilkITt: a 20-minute animation that grows a different tree each time, made for an exhibition and shared on the Grasshopper3D forum by Stefan K ([Grasshopper3D album](https://grasshopper3d.com/photo/albums/growing-tree-animation)).
- Computerphile, "L Systems: Creating Plants from Simple Rules": a clear explainer of the grammar and its plant models ([YouTube](https://www.youtube.com/watch?v=puwhf-404Xc)).

## How it works
- An L-system has an alphabet, an axiom (the start string), and rewriting rules. In each step every symbol is replaced at once (parallel rewriting) ([Wikipedia](https://en.wikipedia.org/wiki/L-system)).
- Lindenmayer's algae example: axiom A, rules A -> AB, B -> A. String lengths follow the Fibonacci sequence ([Wikipedia](https://en.wikipedia.org/wiki/L-system)).
- To draw, the string is read by a "turtle". Symbols mean draw a line, turn by an angle, or save and restore position with brackets `[` and `]`. Brackets create branches ([Wikipedia](https://en.wikipedia.org/wiki/L-system)).
- Variants: context-sensitive rules (neighbours matter), stochastic rules (random choice among rules), parametric rules (symbols carry numbers such as length and angle) ([Wikipedia](https://en.wikipedia.org/wiki/L-system), [Hansmeyer](https://www.michael-hansmeyer.com/l-systems)).
- Recursive rules give self-similar, fractal-like forms: Koch curve, Sierpinski triangle, dragon curve, fractal plant with a 25 degree angle ([Wikipedia](https://en.wikipedia.org/wiki/L-system)).

## Visual language
- Thin line trees and ferns that branch into ever finer twigs. Strict self-similarity, symmetric or slightly jittered when stochastic.
- Space-filling curves (dragon, Koch, Sierpinski) read as geometric lace or circuitry; the same grammar gives organic or crystalline looks depending on the angle.
- Motion is natural: growth outward from a root, then sway. Rhythm comes from generations (each iteration is a beat of growth).
- Strongest on dark backgrounds with glowing line, or as a mask/texture source.

## Live potential
- Kick: add one iteration (a new generation of twigs), or pulse line width and brightness. Resetting depth on the downbeat of a bar gives a "grow in, collapse out" phrase.
- Bass: modulate the branch angle (a few degrees) so the whole canopy breathes; low-frequency sway of the trunk.
- Hi-hats: spawn leaves or sparks at branch tips, or jitter stochastic rules.
- BPM: tie growth time of one generation to a beat division, so the tree finishes a depth every bar.
- MIDI faders: iteration depth, angle, segment length decay, line width, rule choice (switch between plant, dragon, Koch), colour, random seed.
- Layering: add-blend over feedback or slit-scan, use as a mask for video, or feed as a Spout/NDI source into Resolume with several trees at different depths.
- Boring on a big screen when: one static tree at fixed depth, uniform thin lines, no variation between generations, or hard jumps in depth that look like popping. Growth needs easing, and rules need to change over time.

## Variations
- Beat-grown forest: several L-systems with different angles, each depth tied to a different drum voice.
- Angle morph: slowly interpolate between plant (25 degrees) and dragon/Koch style rules so the same skeleton changes species.
- Stochastic seeds on hi-hats: each hit re-rolls branches for a shimmering, never-repeating canopy.
- 3D growth in Unreal Niagara: bake the turtle path to splines or instanced meshes and grow along the spline with a mask.
- Wind and decay: reverse the process (rewriting back, or branches retracting) on breakdowns.

## Facts
- L-systems apply all rules in parallel each step, unlike standard formal grammars that apply one rule at a time ([Wikipedia](https://en.wikipedia.org/wiki/L-system)).
- Lindenmayer's algae system produces strings whose lengths are Fibonacci numbers ([Wikipedia](https://en.wikipedia.org/wiki/L-system)).
- The original purpose was biology: modelling cell development in simple organisms, not drawing ([Wikipedia](https://en.wikipedia.org/wiki/L-system)).
- *The Algorithmic Beauty of Plants* is free as a PDF (17 MB high quality, 4 MB low quality) ([algorithmicbotany.org](http://algorithmicbotany.org/papers/#abop)).
- Hansmeyer produced his 2003 L-system forms from a custom Excel/VBA program that output Maya scripts ([project page](https://www.michael-hansmeyer.com/l-systems)).
- Reviewers noted that the book's plant images came from relatively simple algorithms ([Wikipedia: ABOP](https://en.wikipedia.org/wiki/The_Algorithmic_Beauty_of_Plants)).

## Visual references
- [Computerphile: L Systems, Creating Plants from Simple Rules](https://www.youtube.com/watch?v=puwhf-404Xc) — explainer video showing plants grown from rewriting rules (channel: Computerphile).
- [unnamed tree ∞/∞ 3d](https://youtu.be/7_vKj9J7i9g) — excerpt of a 20-minute growing-tree animation (author: MilkITt; title confirmed via oEmbed).
- [Grasshopper3D: growing tree animation](https://grasshopper3d.com/photo/albums/growing-tree-animation) — page for the exhibition animation of generated growing trees.
- [Hansmeyer: L-Systems in Architecture](https://www.michael-hansmeyer.com/l-systems) — Sierpinski, stochastic and parametric structures as images.
- [Commons: Dragon trees](https://commons.wikimedia.org/wiki/File:Dragon_trees.jpg) — fractal trees from a varied dragon-curve L-system.
- [Commons: L-system Tree05](https://commons.wikimedia.org/wiki/File:L-system_Tree05.jpg) — a rendered L-system tree.

## Lead image
- file_page: https://commons.wikimedia.org/wiki/File:Fractal-plant.svg
- image_url: https://upload.wikimedia.org/wikipedia/commons/thumb/4/44/Fractal-plant.svg/1280px-Fractal-plant.svg.png
- shows: the classic L-system fractal plant (25 degree branching) drawn as line branches
- author: Sakurambo
- license: CC BY-SA 3.0
- license_url: https://creativecommons.org/licenses/by-sa/3.0/

## Sources
- [Wikipedia: L-system](https://en.wikipedia.org/wiki/L-system)
- [Wikipedia: The Algorithmic Beauty of Plants](https://en.wikipedia.org/wiki/The_Algorithmic_Beauty_of_Plants)
- [algorithmicbotany.org: ABOP](http://algorithmicbotany.org/papers/#abop)
- [Hansmeyer: L-Systems in Architecture](https://www.michael-hansmeyer.com/l-systems)
- [Grasshopper3D growing tree animation](https://grasshopper3d.com/photo/albums/growing-tree-animation)
- [Computerphile video](https://www.youtube.com/watch?v=puwhf-404Xc)
- [Commons API: Fractal-plant.svg, Dragon_trees.jpg, L-system_Tree05.jpg](https://commons.wikimedia.org/wiki/File:Fractal-plant.svg)
