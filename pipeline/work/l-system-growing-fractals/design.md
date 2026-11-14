# L-System Growing Fractals — design

## Look

On a near-black screen a thin glowing line rises from the bottom edge. It forks, and every fork forks again. Within one bar a bare stalk turns into a fern-like plant whose finest twigs are hair-thin points of light. Old wood glows a deep teal. Fresh growth flashes white-mint at the tips and cools to green over one beat. The whole canopy sways like a weed under water, the branch angle breathes with the bass, and on hi-hats tiny magenta sparks bloom at the tips. Every bar adds a generation. Each new set of twigs grows smoothly out of the old ones and the camera pulls back so the plant stays in frame. Nothing pops. On a phrase change the plant collapses back into its root and a different species grows in its place: the classic 25° fractal plant, a bushy 22.5° weed, or a crystalline dragon or Koch curve that folds out like circuitry. The reference image is the Wikipedia "fractal plant" (`X → F+[[X]-X]-F[-FX]+X`, `F → FF`, 25°) drawn as light on black ([Wikipedia: L-system](https://en.wikipedia.org/wiki/L-system)), animated in the spirit of the developmental models in *The Algorithmic Beauty of Plants*, chapter 6 "Animation of plant development" ([algorithmicbotany.org](http://algorithmicbotany.org/papers/#abop)). The mood is organic, patient and slightly alien: a bioluminescent plant in time-lapse.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #030605 | Normal | 100 % | just above black so LED modules do not look dead |
| 2 | Geometry buffer | CPU rewriting + turtle → segment list (positions, depth, path distance, birth generation) | (data, not shown) | — | rebuilt only when preset, seed, generation or angle changes |
| 3 | Branch lines | instanced quads, one per segment; reveal, generation easing, sway in the vertex stage; anti-aliased line profile in the fragment stage | Add | 100 % | the main image |
| 4 | Growing tips | one soft sprite at every segment that is still growing (0 < frac < 1) | Add | 80 % | makes the "growth front" readable from the back of the room |
| 5 | Sparks / blossoms | short-lived sprites at leaf tips (depth = max), spawned on hi-hats | Add | 60 % | colour #FF4FA3; off in "circuit" species |
| 6 | Sap echo | feedback of layers 3–5: zoom 1.003 toward the root, decay 0.88 | Add | 30 % | soft afterglow along the growth path; 0 % on cuts |
| 7 | Post | bloom → grain → vignette | see compositing | — | order matters |

## Techniques

**1. Rewriting with birth tags (CPU, on change only)**
1. Each symbol is stored as a small record, not a bare char: `{sym, birthGen, parentIndex}`. The axiom gets birthGen 0, parentIndex −1.
2. One generation: walk the string once and replace every symbol by its successor at the same time (parallel rewriting, [Wikipedia](https://en.wikipedia.org/wiki/L-system)). Constants (`+ - [ ]`) copy through unchanged.
3. Age convention: if a successor starts with its own predecessor (e.g. `F → FF`), that first copy inherits the old birthGen. All other symbols in the successor get birthGen = n+1. Every child records the index of the symbol it came from (parentIndex). This tagging is what makes smooth growth possible later.
4. Stochastic rules: several successors per predecessor with weights (for example 0.5 / 0.5, as in the stochastic example on [Wikipedia](https://en.wikipedia.org/wiki/L-system)). Choose with `hash(seed, symbolIndexInParentString, n)`, not with a running random generator. Then the same seed always gives the same tree, and changing generation keeps the old branches where they were.
5. Budget before building: the symbol counts per generation follow a linear recurrence (count vector × rule matrix). Compute the counts for n = 0..12 first, then set `maxGen` = the largest n whose F-count fits the segment budget (see the limits table). Never build a string just to find out it is too big.
6. Keep all generations 0..maxGen in memory. Then stepping down (collapse) is as cheap as stepping up.

**2. Turtle to segment list (CPU)**
1. State: position p, heading θ, depth d (bracket nesting), path distance s (arc length from the root), step length L, width w. `[` pushes the state and `]` pops it. `+` / `−` turn by ±δ. `F` (and `G` where the rules use it) draws. `X` and other non-drawing letters are skipped, as in Bourke's symbol table ([Paul Bourke: L-systems](https://paulbourke.net/fractals/lsys/)).
2. Each `F` emits a segment record: `p0, p1, s0, s1 = s0 + L, depth d, birthGen, parentSeg, tip flag` (tip = no later segment starts at p1). That is 12 floats, 48 bytes.
3. Merge runs: consecutive `F` with no turn and the same birthGen become one segment. In `F → FF` plants this roughly halves the instance count with no visible change.
4. Width taper: `w = w0 · 0.72^d`, with w0 = 6 px at 1080p and a floor of 1.5 px. Brightness taper: `I = 0.85^d`, so the dense canopy does not burn out.
5. Bounding box: track min/max of all points. The camera fit uses it (technique 5).
6. Cost: one pass over the string, O(symbols). Plan about 1 ms per 50k segments in compiled code (C++, Rust, C#). In GDScript or Blueprint, run the turtle only on preset or seed changes, never per frame.

**3. Smooth growth between generations ("continuous length")**
1. The stage parameter is fractional: `gen = n + f`. Display the string of generation n+1 and use `e = smootherstep(f) = f³(f(6f − 15) + 10)`.
2. Branching trees (bracketed rules): re-run the turtle while f moves. Segments with birthGen ≤ n get the full length. Segments born at n+1 get length `L · e`, and turns born at n+1 use `δ · e`. New twigs therefore sprout from their parent, unfold from the parent's direction and lengthen, and the parent stays in place. TouchDesigner's LSystem SOP uses the same idea in its Continuous Length / Continuous Angles options for non-integer generations ([Derivative: LSystem SOP](https://docs.derivative.ca/LSystem_SOP)).
3. Edge-replacement curves (dragon, Koch, Sierpinski): no re-turtle is needed. For every vertex of generation n+1, also store its "rest position", the point at the same fraction along its parent segment of generation n. The vertex stage draws `mix(rest, true, e)`. The dragon then folds out of the previous dragon, and the Koch bumps push out of straight lines. Both positions are computed once on the CPU.
4. Re-turtle runs only during the transition (1 beat by default). Between transitions the geometry is static and fully GPU-driven.
5. Stepping down (collapse) is the same with f running from 1 to 0.

**4. Reveal by path distance (GPU only)**
1. `grow` in 0..1 sweeps a growth front from the root outward along the branches: `front = grow · (sMax + fade)`, with fade = 0.15 · sMax.
2. Per segment: `frac = clamp((front − s0) / (s1 − s0), 0, 1)`. The vertex stage moves p1 to `mix(p0, p1, frac)` and hides the quad when frac = 0.
3. Every branch starts at its fork's s (the stack restores s), so all branches of the same order grow at the same time. That reads as real growth, not as a pen tracing the string.
4. Use it for "grow in" at the start of a phrase (grow 0 → 1 over 1–2 bars) and "collapse out" on a break (1 → 0, ease-in). It needs no CPU work, so it is safe at any segment count.

**5. Camera fit and sway**
1. Fit: `scale = 0.9 · min(W / bboxW, H / bboxH)`, root anchored at the bottom centre (y = 4 % of the height). Interpolate the bbox from generation n to n+1 with the same e, then smooth it with a 400 ms damper. The camera pulls back as the plant grows and never jumps.
2. Sway (vertex stage, no re-turtle): rotate each vertex around the root by `φ = sway · (s / sMax)² · sin(2π·t·0.25 Hz + s · 0.8)`. The trunk barely moves, the tips swing, and the phase travels upward like a wave.
3. Spread breathing (bass): the true branch angle δ ± 3° needs a re-turtle. Use it only where the turtle runs in compiled code. Otherwise fake it with sway.

**6. Fold-tree (fragment-only fallback for Wire)**
1. For the symmetric binary tree (the `0 → 1[0]0` family on [Wikipedia](https://en.wikipedia.org/wiki/L-system)) the distance to the tree can be found per pixel without any string.
2. Per pixel, loop i = 0..depth−1: take the distance to the current trunk segment (0,0)–(0,len) and keep the minimum. Translate by len. Mirror `p.x = abs(p.x)` so both children are covered at once. Rotate by the spread angle and divide by the length ratio 0.72 (multiply the distance back).
3. Growth: segment i is drawn up to `clamp(grow · depth − i, 0, 1)` of its length.
4. Jitter: build a branch id while folding (`id = id · 2 + sign(p.x)`) and add `±jitter · hash(id)` to the angle. Siblings now differ slightly, and the distance stays usable for a glow.
5. Cost is `depth` iterations per pixel (12 → 4096 tips). It is cheap, but limited to binary trees. It is not a real L-system.

### Segment limits per generation

Counts of drawing symbols (F/G) before merging runs. Live budget: about 100k segments drawn and about 30k for per-frame re-turtle in compiled code.

| species | axiom / rules / angle | gen 4 | gen 5 | gen 6 | gen 7 | gen 8 | recommended maxGen |
|---------|----------------------|-------|-------|-------|-------|-------|--------------------|
| fractal plant | `-X`; `X → F+[[X]-X]-F[-FX]+X`, `F → FF`; 25° ([Wikipedia](https://en.wikipedia.org/wiki/L-system)) | 360 | 1,488 | 6,048 | 24,384 | 97,920 | 7 (8 only for reveal, no re-turtle) |
| binary tree | `0`; `1 → 11`, `0 → 1[0]0`; 45° ([Wikipedia](https://en.wikipedia.org/wiki/L-system)) | 48 | 112 | 256 | 576 | 1,280 | 12 (2^12 = 4096 leaves) |
| weed | `F`; `F → FF-[XY]+[XY]`, `X → +FY`, `Y → -FX`; 22.5° ([Bourke](https://paulbourke.net/fractals/lsys/)) | count in-tool | | | | | stop when > 30k |
| dragon | `F`; `F → F+G`, `G → F−G`; 90° ([Wikipedia](https://en.wikipedia.org/wiki/L-system)) | 16 | 32 | 64 | 128 | 256 | 15 (32,768); 16 = 65,536 for reveal only |
| Koch (square) | `F`; `F → F+F−F−F+F`; 90° ([Wikipedia](https://en.wikipedia.org/wiki/L-system)) | 625 | 3,125 | 15,625 | 78,125 | 390,625 | 6 (7 for reveal only) |
| Sierpinski | `F−G−G`; `F → F−G+F+G−F`, `G → GG`; 120° ([Wikipedia](https://en.wikipedia.org/wiki/L-system)) | 243 | 729 | 2,187 | 6,561 | 19,683 | 9 (59,049) |

Plant string length is about 4× the F-count (about 100k symbols at gen 7). Cap strings at 500k symbols. For comparison, Bourke's program limits strings to 300,000 symbols and bracket nesting to 100 levels ([Bourke](https://paulbourke.net/fractals/lsys/)).

## Shaders

Render order per frame: P0 (CPU, only on change) → P1 lines → P2 tips/sparks → P3 echo feedback → P4 composite + post.

**P0 — Rewrite and turtle (CPU, not a shader)**
- Purpose: build the segment buffer. Runs on preset, seed or generation changes, and every frame during a bracketed-tree transition.
- Inputs: rules, axiom, δ, seed, gen = n + f, budget.
- Output: segment buffer (structured buffer, or an RGBA32F texture with 3 texels per segment: `p0.xy p1.xy | rest0.xy rest1.xy | s0 s1 depth birthGen`), instance count, bbox, sMax. Upload to the GPU only the range that changed.

**P1 — Branch lines (vertex + fragment, instanced)**
- Inputs: segment buffer, uniforms grow, e (generation ease), sway, time, scale/offset, w0, palette ramp (256 × 1), flash, bornBeat (time each generation finished).
- Vertex, per instance: read the segment. Positions = `mix(rest, true, e)` for curves. Apply the reveal frac (technique 4) and the sway rotation (technique 5). Expand into a quad: along the segment plus `w/2 + 1.5 px` sideways (margin for AA and halo), with round caps through 0.5·w overlap at each end. Pass `across` (−1..1), depth, birthGen and frac to the fragment stage.
- Fragment: `core = 1 − smoothstep(w/2 − 0.75 px, w/2 + 0.75 px, |across|·halfWidthPx)`, `halo = exp(−(|across|·halfWidthPx / (2w))²) · 0.35`. Colour = `ramp(depth / maxDepth)`, mixed toward the highlight by youth = `exp(−(now − bornBeat) / 1 beat)`. Output `(core + halo) · I · (1 + flash)`.
- Output: RGBA16F at output resolution, cleared each frame. 4× MSAA is optional; the analytic AA usually suffices.

**P2 — Tips and sparks (point sprites, additive)**
- Tips: one sprite per segment with 0 < frac < 1 (reveal) or birthGen = n+1 during a transition, at the current end point. Radius 2.5·w, colour highlight.
- Sparks: a pool of 2,048 sprites. A hi-hat spawns 32 of them at random tip segments (hash(seed, hitCount)), lifetime 0.4 s, scale 0 → 1 → 0, a slight upward drift of 20 px/s, colour #FF4FA3.
- Output: added into the P1 target.

**P3 — Sap echo (fragment, ping-pong)**
- Inputs: P1+P2 target, previous echo buffer, zoom 1.003 centred on the root, decay 0.88.
- Per pixel: sample the previous echo at `(uv − root) / zoom + root`, multiply by decay, add the current frame · 0.2.
- Output: RGBA16F at half resolution (the echo is soft anyway). Two buffers swapped each frame.

**P4 — Composite and post**
- Background #030605 + P1/P2 (Add) + P3 · echo (Add) → bloom → grain → vignette → clamp to 8-bit for Spout/NDI.

**PW — Fold-tree ISF (Wire only, single pass)**
- Inputs: grow, depth (int, 4–12), spread (°), ratio, jitter, sway, time, palette colours.
- Per pixel: technique 6. Keep the minimum distance and the depth index of the closest segment. Colour = ramp(i / depth) · (core from distance and width + halo).
- Output: RGBA at composition resolution. Feedback through Wire's own feedback node.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base / black | #030605 | background, black level |
| shadow | #0B2A24 | ramp 0.0: trunk and old wood, deep teal |
| mid | #1FA67A | ramp 0.5: main branches |
| accent | #7CFFB2 | ramp 1.0: finest twigs |
| highlight | #F2FFE8 | growing tips and fresh growth (youth), kick flash |
| spark | #FF4FA3 | hi-hat blossoms, the only warm colour |

- Mapping: a 3-stop ramp indexed by `depth / maxDepth`, so the plant brightens and cools toward its tips. Youth mixes the highlight over it: every new generation flashes white-mint and settles into the ramp over 1 beat. The viewer can see which twigs are new.
- Alternative palettes, one per species. "Ember" for bushes: #070302, #4A1405, #D9480F, #FFB000, highlight #FFF4D6. "Circuit" for dragon, Koch and Sierpinski: #02040A, #0A2A5E, #1E88E5, #00E5FF, highlight #E8FDFF, sparks off.
- Over time: the palette changes with the species on phrase changes, crossfaded over the collapse. Inside a phrase the ramp shifts by +0.05 per bar of hue (rotation in OKLCH or HSV) so long sets drift.
- Music: a kick sets flash to 0.35 (release 180 ms). Mids add up to +0.1 to the ramp index (more tip colour).
- LED walls: keep black at #030605. The shadow colour #0B2A24 is very dark, so check it on the wall. If the trunk disappears, lift it to #12403A. Cap core brightness at 0.9 before bloom so the trunk does not clip into a flat block. Lines thinner than 1.5 LED pixels flicker while swaying. Raise w0 rather than adding contrast.

## Blending and compositing

- All light layers are **Add**: light on black, and crossing branches should brighten.
- **Brightness taper** `0.85^depth` (P1) is the main guard against the canopy saturating. Space-filling Koch generations at gen 6 need 0.75^depth, or an overall gain of 0.6.
- **Sap echo** (P3): 30 % default, decay 0.88, zoom 1.003. 50 % on breaks, 0 % on hard cuts and during collapse (otherwise the old species ghosts into the new one).
- **Masks**: the line layer before post, in grey, works as a Luma Key or Track Matte over video in Resolume. The video is then seen through a growing tree. For a mask, render w0 at 12 px and skip the halo.
- **Post chain in order**: bloom (threshold 0.6, intensity 1.4, large radius; thin lines need a low threshold to bloom at all) → film grain 2 % monochrome → vignette 0.25 → optional sharpen 0.1 on walls with a pitch above 4 mm. No chromatic aberration by default. On a drop, a 1-beat 2 px RGB split reads like an electric discharge.

## Motion and timing

- **Generation clock**: one generation per bar. f runs from 0 to 1 over the first beat of the bar (smootherstep), and the plant rests for 3 beats while it sways. Builds: 1 generation per 2 beats. Breaks: 1 per 2 bars.
- **Phrase cycle (default 16 bars)**: bar 1 grow 0 → 1 at gen 2 (the seedling rises). Bars 2–7: one generation per bar up to maxGen. Bars 8–15: hold at maxGen, with spread breathing and hi-hat sparks. Bar 16: collapse (grow 1 → 0 over 2 beats, ease-in), then a new species and seed. This is the "grow in, collapse out" phrase from the research.
- **Sway**: 0.25 Hz base, amplitude 2–6°, always on so the plant is never static.
- **Spread breathing**: δ = preset ± 3° from the bass (compiled turtle) or sway +2° (fallback).
- **Kick**: flash 0.35, and the width pulses `w0 · (1 + 0.25·env)`, release 180 ms. No geometry change on every kick: that would pop.
- **Drop**: a hard cut to a new species at maxGen − 1 and grow 0 → 1 in 1 beat (fast reveal), echo 0, flash 1.0. Then one generation per 2 beats until maxGen.
- **Break**: collapse to gen 3, then grow back one generation per 2 bars. Echo 50 %, sway 6°, bloom +0.3, "Ember" or "Circuit" palette.
- **Angle morph (variation)**: over 8 bars, slide δ from 25° to 90° on the plant rules. The skeleton changes from organic to crystalline. It needs a per-frame re-turtle, so keep gen ≤ 6 while morphing.

## Parameters

| name | range | default | what it does on stage | suggested MIDI / BPM mapping |
|------|-------|---------|----------------------|------------------------------|
| grow | 0–1 | 1 | growth front from the root; 0 = only the seed | fader; auto 0 → 1 on phrase start, 1 → 0 on collapse |
| depth | 0–maxGen (fractional) | auto clock | generation, eased between integers | KeyStep knob for manual control; BPM clock +1 per bar; pad = "next generation now" |
| spread | preset ± 15° | 25° (plant) | branch angle: tight spire vs. wide fan | knob; bass adds ± 3° |
| sway | 0–10° | 3° | wind on the canopy, tips move most | knob; LFO 0.25 Hz |
| jitter | 0–1 | 0.2 | stochastic rule weight and angle noise; re-rolls with seed | knob; hi-hat increments seed when jitter > 0 |
| species | preset 0–5 | 0 (fractal plant) | picks rules, angle and palette; switched through collapse | BeatStep Pro pads 1–6; held "cut" pad = hard switch |

## Build recipes

### Resolume Arena

Arena cannot rewrite strings or draw instanced geometry. It composes; the plant comes from an engine or from a Wire patch.
- **Source A (full version)**: a Spout or NDI source from UE5, Godot, TouchDesigner or a Rust/wgpu app that runs P0–P2 (and optionally P3). TouchDesigner is the quickest: the LSystem SOP generates the geometry with fractional Generations and the Continuous Length/Angles toggles ([Derivative: LSystem SOP](https://docs.derivative.ca/LSystem_SOP)). Render it with a Line MAT or the SOP's tube output and send it out with a Syphon Spout Out TOP.
- **Source B (Arena-only)**: the Wire fold-tree patch (below) loaded as a source.
- **Layer 1**: Solid Color #030605. **Layer 2**: plant source, Add, 100 %. **Layer 3**: the same source with Trails (about 0.85 feedback) at 30 %, Add, which stands in for P3. **Layer 4 (optional)**: video with the plant layer used as a luma mask.
- **Composition**: Bloom (low threshold, as above), then a vignette.
- **Control**: Arena's BPM clock and audio analysis go to the engine as OSC (bar counter, kick, bass, hats). The engine owns generation, grow and seed. Faders for grow, spread, sway and jitter go straight from the KeyStep/BeatStep to the engine over MIDI, or through Arena's OSC output.

### Resolume Wire

Wire has no vertex shaders: the Resolume ISF page says Wire (version 7.22) does not support them because it works in 2D ([Resolume ISF](https://resolume.com/support/en/isf)). Multipass ISF support depends on the version. Do not attempt string rewriting in Wire.
- **Option A, recommended: Wire as receiver and control patch.** `Spout Receiver` (engine frame) → `Blend` Add with a feedback branch (feedback node → `Transform` scale 1.003 → `Multiply` 0.88) → blur-and-add bloom → output. Control: BPM / bar counter → counter (wrap 16) → logic for "generation +1 per bar" and "collapse on bar 16" → OSC out to the engine (gen, grow, species, seed). Exposed: grow, depth, spread, sway, jitter, species.
- **Option B, Wire-only fold-tree.** An ISF node with the PW shader (technique 6). Inputs as ISF `float` (grow, spread, ratio, jitter, sway), `long`/`float` depth, `color` ×3 for the ramp, and an `event` for "re-roll" (the ISF page lists float, bool, color and event inputs). Drive grow from a bar-phase ramp so each bar grows one level: `grow = (bar + easedPhase) / depth`. Feed the output through a feedback node (decay 0.9) for afterglow. Use it as a calm binary-tree variant. Real plant rules, the dragon and Koch need Option A.

### Unreal Engine 5.7

- **Where the string lives**: a C++ `ULSystemComponent` on an actor `BP_LSystem`. It holds the rules (a DataAsset per species: axiom, rules with weights, δ, palette), rewrites with birth tags, runs the turtle and fills `TArray<FVector4f>` segment data (3 × Vector4 per segment). Rewriting runs on species, seed or maxGen changes. The turtle runs on transitions in an async task, with the result swapped in on the game thread. Blueprint-only turtles are fine below about 2k segments and only on change.
- **Rendering, option 1 (2D, flat, recommended for VJ)**: Niagara system `NS_LSystem`, GPU Compute Sim. A User parameter of type Niagara Float4 Array is set from C++/Blueprint with the array data-interface functions. Spawn Burst = segment count (re-spawn when the count changes). Particle Update Scratch Pad: read 3 entries by `ExecutionIndex`, compute rest/true lerp (User.GenEase), reveal frac (User.Grow), sway (User.Sway, time), and write Position = midpoint, `Particles.Length`, `Particles.Width`, `Particles.Dir`, `Particles.Depth`, `Particles.Youth`. **Sprite Renderer**: Alignment = Custom Alignment (Dir), Facing = camera, Sprite Size = (Length, Width + 3 px). Material `M_LSysLine` unlit, Additive: the AA core and halo from the sprite UV.x as in P1, colour from a Curve Atlas row indexed by Depth / MaxDepth, Youth mixed toward the highlight. Second emitter for tips and sparks (sprites, additive, hi-hat event via User.SparkBurst).
- **Rendering, option 2 (3D hero tree)**: the research's spline variation. For fewer than about 300 branches, build one `SplineMeshComponent` chain per branch, with a material mask `step(s, Grow·sMax)` from a per-instance custom primitive data value. For larger trees, use a Dynamic Mesh tube with s in UV1 and World Position Offset pulling vertices back to the branch start for the reveal.
- **Post-process volume**: Bloom intensity 1.4, threshold 0.6; Film Grain 0.02; Vignette 0.25. Anti-aliasing None or FXAA (TSR smears thin moving lines), motion blur off, manual exposure.
- **Camera**: an orthographic CineCamera. Blueprint interpolates OrthoWidth and location from the bbox (technique 5, 400 ms damper).
- **Remote Control preset** (OSC and MIDI protocols): Grow, Depth (float), Spread, Sway, Jitter, Species (int), Seed (int), Flash, EchoAmount. A Blueprint timeline handles the generation ease from an OSC bar/beat clock.
- **Output**: Spout via a third-party plugin (for example Off World Live), or NDI via the NDI SDK for Unreal, at the LED processor's native resolution.

### Godot

- **Where the string lives**: a `Node2D` script `LSystem`. GDScript is fine for rewriting and turtling on change up to about 20k segments. Use C# or a GDExtension (Rust via godot-rust works) for per-frame re-turtles during transitions. Store the species as `Resource` files.
- **Lines**: `MultiMeshInstance2D` with a 1 × 1 `QuadMesh`, `transform_format` 2D, `use_colors` and `use_custom_data` set **before** `instance_count` (the docs require this order) ([Godot MultiMesh](https://docs.godotengine.org/en/stable/classes/class_multimesh.html)). Per instance: `set_instance_transform_2d` (true midpoint, rotation, length × width), `set_instance_color` = (rest midpoint delta xy, s0, s1), `set_instance_custom_data` = (depth, birthGen, width, tip). `visible_instance_count` gives a cheap "draw only the first k". A `canvas_item` shader reads `INSTANCE_CUSTOM` in `vertex()`, applies reveal, generation ease and sway, and draws the AA core and halo in `fragment()`. Uniforms: `grow, gen_ease, sway, time_s, w0, flash, max_depth, sampler2D ramp` (a `GradientTexture1D`).
- **Tips/sparks**: `GPUParticles2D`, additive `CanvasItemMaterial`, emission points baked into a texture from the tip list on each generation, amount 2048, `emitting` pulsed on hi-hats.
- **Echo**: `SubViewport` A renders the plant. `SubViewport` B samples its own previous texture and A (zoom 1.003, decay 0.88) for the ping-pong, half resolution, HDR 2D on.
- **Post**: `WorldEnvironment` glow, intensity 1.4, HDR threshold 0.6.
- **Control**: `InputEventMIDI` for faders and pads, an OSC addon for the Resolume clock, and a Spout addon or NDI for output.

## Audio reactivity

| band | drives | smoothing | threshold |
|------|--------|-----------|-----------|
| kick, 40–100 Hz onset | flash 0.35, width pulse +25 % | attack 2 ms, release 180 ms | +6 dB over the 1 s average |
| bass, 60–250 Hz level | spread ± 3° (or sway +2° in fallback) | attack 100 ms, release 700 ms | none, continuous |
| mids, 300 Hz–2 kHz | ramp index + up to 0.1 (more tip colour) | attack 150 ms, release 1 s | none |
| hi-hats, 6–12 kHz onset | 32 sparks at random tips; seed +1 if jitter > 0 | attack 1 ms, spark life 400 ms | gate at −30 dBFS, max 8 per beat |
| bar / phrase clock | +1 generation per bar; collapse and species change every 16 bars | — | — |

Geometry changes come only from the clock, never from raw audio. Audio-driven generation jumps look like popping, which the research names as the main failure. Audio touches brightness, width, colour and small angles only.

## Performance tips

- **GPU**: 100k instanced quads plus AA fragment work is under 1 ms on a current desktop GPU at 1080p. Overdraw in dense canopies matters more than the instance count. Space-filling Koch gen 6 at 4K is the worst case; lower w0 there.
- **CPU**: rewriting is O(symbols) and so is the turtle. The real limit is per-frame re-turtles during transitions: keep the transitioning tree under about 30k segments in compiled code and about 5k in GDScript/Blueprint. Otherwise use reveal (GPU-only) instead of generation easing for the last step.
- **Memory**: 48 bytes per segment (about 5 MB at 100k). Keep all generations up to maxGen ready so collapse and regrow cost nothing.
- **Uploads**: upload only during transitions. Use a persistent buffer and update the changed range, not a new buffer per frame.
- **Thin lines on LED walls**: minimum 1.5 output pixels; sub-pixel lines shimmer while swaying. When the processor downscales (a 4K render to a lower-resolution wall), render at the wall's native resolution instead, or the twigs vanish.
- **Aspect ratios**: a tree is tall. On portrait columns it is perfect. On 32:9 strips, plant a forest of 3–5 trees with different seeds and generation offsets (the "beat-grown forest" variation), or switch to a dragon curve laid horizontally. Do not stretch one tree, because the fit then makes it tiny.
- **Pitfalls**: building a string before checking its count; stochastic choice with a running RNG (the tree reshuffles on every generation); TSR/TAA on thin lines; feedback left on during species changes; generation steps without easing.

## Further references

- [Wikipedia: L-system](https://en.wikipedia.org/wiki/L-system): grammar, parallel rewriting, stochastic and parametric variants, and the plant, dragon, Koch, Sierpinski and binary-tree rules used here.
- [Paul Bourke: L-System User Notes](https://paulbourke.net/fractals/lsys/): turtle symbol table (including width and length-scale symbols) and a large set of tested plant and curve rules.
- [Prusinkiewicz & Lindenmayer: The Algorithmic Beauty of Plants](http://algorithmicbotany.org/papers/#abop): the free PDF; chapter 6 covers animation of plant development.
- [Derivative: LSystem SOP](https://docs.derivative.ca/LSystem_SOP): a ready-made L-system generator with fractional generations and continuous length and angle for smooth growth.
