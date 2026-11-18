# Truchet Tile Weaving — design

## Look

A dense square grid that you never see as a grid. Over a near-black ground, warm bone-white ribbons run in quarter-circle arcs from cell edge to cell edge and join into long winding paths, closed rings and knots. This is Cyril Stanley Smith's quarter-circle Truchet tile, oriented at random per cell ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)). Each ribbon is shaded like a soft tube, with a darker edge and a bright spine. In "weave" mode some cells hold a crossing tile where one strand passes over the other, which makes the plane read as woven cloth. The two regions the lines cut the plane into are filled in two very dark tones, ink and deep indigo, so the whole tiling also reads as a two-colour maze. Thin orange light dashes travel along the paths like current in a circuit. On every bar the weave re-knots: a wave of tiles turns by 90° from a point on the screen, the paths break for a moment and close up again in new loops. On the drop, cells split into halves and quarters (Carlson's multi-scale idea, [Bridges 2018](https://archive.bridgesmathart.org/2018/bridges2018-39.html)), so the ornament suddenly gets finer and richer. The mood is ornamental, hypnotic and exact: a 1704 ornament run as a live circuit board.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #07070B | Normal | 100 % | just above LED black, so dark tiles still read as "on" |
| 2 | Region fill | two-colour parity of the Truchet regions (#07070B / #141633) | Normal | 100 % | shows the maze; fade the indigo to 0 for a pure line look |
| 3 | Ribbons | Truchet arcs (and crossing tiles in weave mode) as shaded tubes, constant width in cell units | Normal (alpha over 2) | 100 % | main layer; same shader as 2 in practice |
| 4 | Flow dashes | dashes moving along the path coordinate of each ribbon | Add | 80 % | the "current"; hi-hat flashes ride here |
| 5 | Video through matte (optional) | any clip, masked by the ribbon mask of layer 3 | Normal with luma key, or Multiply | 0–100 % | turns the weave into a window onto video |
| 6 | Trail feedback (optional) | previous frame × 0.85, scale 1.003, mixed under the current frame | Add (inside the feedback buffer) | 0–40 % | smears tile rotations into glowing arcs on breaks; off by default |
| 7 | Post | bloom, then chromatic aberration (drop only), then grain | see compositing | — | order matters |

## Techniques

**1. Grid, hash and orientation**
1. Fit the grid to the short side of the output so cells stay square on any aspect: `p = (fragCoord - 0.5 · res) / res.y · density`, density = cells across the short side (4–40).
2. `id = floor(p)`, `gv = fract(p) - 0.5` (local coordinates -0.5..0.5).
3. Random value per cell: `h = hash21(id + seed · 17.13)`. Any stable 2D-to-1D hash works; [The Art of Code's tutorial](https://www.youtube.com/watch?v=2R7h76GoIJM) builds the same structure.
4. Orientation: `flip = h < flipProb ? 1 : 0`; if flip, mirror `gv.x = -gv.x`. Mirroring and rotating by 90° give the same tile, so either works.
5. flipProb = 0.5 is the percolation-critical case, where paths are neither too short nor too long ([Chalkdust](https://chalkdustmagazine.com/features/when-truchet-met-chladni/)). Lower or higher values bias toward long diagonal rivers or small closed rings.

**2. Arc distance and ribbon**
1. Pick the nearer of the two arc corners: `c = (gv.x + gv.y > 0) ? (0.5, 0.5) : (-0.5, -0.5)`.
2. `r = length(gv - c)`, `d = abs(r - 0.5)`: distance to the quarter circle through the two edge midpoints. Arcs meet at edge midpoints, so lines continue across cell borders.
3. Ribbon mask: `m = 1 - smoothstep(w - aa, w + aa, d)`, with w = half width in cell units (0.02–0.30) and `aa = 1.5 · density / res.y` (1.5 px, so edges stay crisp at any density).
4. Cross coordinate `v = (r - 0.5) / w` in -1..1. Tube shading: `shade = sqrt(max(0, 1 - v²))`, then `edge = smoothstep(0.7, 1.0, abs(v))` to darken the rim.

**3. Path coordinate (for flow dashes)**
1. Angle along the arc: `a = atan2(gv.y - c.y, gv.x - c.x)`, remapped to 0..1 over the quarter turn: `u = fract(a / (π/2))`.
2. Neighbouring cells must run in opposite directions or the dashes jump at cell borders. Multiply by a checker sign `k = ((id.x + id.y) mod 2) · 2 - 1` and by the flip sign (as in the Art of Code tutorial).
3. Test it: set flipProb = 0 (all tiles equal) and check that dashes run seamlessly. If they reverse at every border, drop one of the two signs.
4. Dashes: `dash = smoothstep(0.5 - dw, 0.5, fract(u · k · 2 + t · flowSpeed))`, dw = 0.15, masked by `shade`.

**4. Two-colour regions**
1. Smith's quarter-circle tilings split the plane into regions that can be coloured with two colours ([Wikipedia](https://en.wikipedia.org/wiki/Truchet_tile)).
2. Side of the arc: `inside = r < 0.5 ? 1 : 0`.
3. Region parity = `checker XOR flip XOR inside`. Map 0 to #07070B and 1 to #141633.
4. Verify on screen: no region may change colour where it touches a cell border. If it does, drop `flip` from the XOR (that depends on whether you mirror or rotate in technique 1).

**5. Weave mode (crossing tiles)**
1. A second hash `h2 = hash21(id + 91.7 + seed)`; if `h2 < crossProb`, this cell is a crossing tile instead of a quarter-circle tile.
2. A crossing tile has two straight strands through the cell centre: horizontal (distance `abs(gv.y)`) and vertical (distance `abs(gv.x)`). Both meet the neighbouring arcs at edge midpoints, so paths still connect.
3. Over/under: horizontal on top when the checker is even, vertical on top when odd. This alternation is what reads as weaving.
4. Draw the lower strand first, then a soft shadow of the upper strand on it (`shadow = 1 - 0.6 · smoothstep(w · 1.8, w, dTop)`), then the upper strand.
5. crossProb 0.15–0.35 looks woven; above 0.6 it turns into plain checked cloth.

**6. Re-knot wave (beat-synced rotation)**
1. Keep two seeds: seedA (current) and seedB (next). For each cell compute flipA and flipB.
2. Only cells where flipA ≠ flipB change. Each one rotates its gv by `θ = (π/2) · ease(k)` around the cell centre.
3. Per-cell progress `k = clamp((t - t0) · waveSpeed - dist(id, origin) · waveDelay, 0, 1)`, with origin a random point picked on the trigger (or the screen centre). Ease = smoothstep, or easeOutBack for a slight snap.
4. While a tile rotates, its arcs no longer meet the neighbours: the paths break and close again. That break is the visible event. Rotate the region fill along with the tile.
5. When every k = 1, set seedA = seedB and pick a new seedB.

**7. Multi-scale subdivision**
1. For each level-0 cell, hash `hs = hash21(id + 53.1 + seed)`. If `hs < split`, subdivide: `p2 = p · 2`, repeat techniques 1–5 with `id2 = floor(p2)` and `gv2 = fract(p2) - 0.5`.
2. Repeat once more for level 2 (max depth 3 levels; deeper gets lost on LED pixel pitch).
3. Halve the ribbon width at each level, and invert the region colours at each level so the levels stay readable against each other. Carlson's paper uses tiles with "wings" to make the scales join cleanly ([Bridges 2018](https://archive.bridgesmathart.org/2018/bridges2018-39.html)); the plain version here does not join perfectly at scale borders, which reads as a deliberate seam. For the exact Carlson look, follow the paper or [makio135's notebook](https://observablehq.com/@makio135/multi-scale-truchet-patterns).
4. Animate `split` from 0 to 0.5 over 1 beat on the drop, back to 0 over 2 bars on a break.

## Shaders

Render order per frame: S1 Truchet, S2 feedback (optional), S3 composite + post. All passes at output resolution; no simulation buffers needed, the pattern is stateless.

**S1 — Truchet pass (single pass, the core)**
- Purpose: region fill, ribbons, crossings, dashes, re-knot rotation and subdivision in one fragment shader.
- Inputs: uniforms `time, density, flipProb, width, crossProb, split, seedA, seedB, t0, origin (vec2), waveSpeed, waveDelay, flowSpeed, dashGain, fillGain`, colour uniforms (5 colours below).
- Per pixel: (1) aspect-correct p; (2) choose level by technique 7 (up to 3 levels, final level wins); (3) id, gv, flips for seedA and seedB; (4) if they differ, rotate gv by the eased angle of technique 6; (5) crossing or arc tile (technique 5 or 2); (6) region parity → fill colour; (7) ribbon shade over the fill; (8) path coordinate and dashes, added on top; (9) write ribbon mask m to alpha.
- Output: RGBA8 (RGBA16F in UE for bloom headroom); RGB = image, A = ribbon mask (used as matte for layer 5).
- Cost: a handful of hashes, one atan, no loops beyond the 3 levels. Cheap even at 4K.

**S2 — Trail feedback (optional, ping-pong)**
- Inputs: S1 output, previous S2 frame.
- Per pixel: sample previous frame at `uv' = (uv - 0.5) / 1.003 + 0.5`, multiply by decay 0.85, then `max(prev, current)` (Lighten) so the trails never wash out the fresh lines.
- Output: two RGBA16F buffers at output resolution, swapped each frame.

**S3 — Composite + post**
- Inputs: S1 (or S2), optional video texture.
- Per pixel: video matte (`mix(image, video, m · videoMix)`), then bloom (threshold 0.75, two-pass Gaussian at half resolution, intensity 0.8), chromatic aberration 1.5 px radial, only during the drop envelope, then grain 3 %.
- Output: final frame to Spout/NDI or the display.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base (region A) | #07070B | ground and even regions |
| base 2 (region B) | #141633 | odd regions, deep indigo |
| ribbon | #E8E0CC | bone white tube body |
| ribbon rim / shadow | #3A3428 | tube edge and crossing shadow |
| accent | #FF5A1F | flow dashes |
| highlight | #FFF3DD | dash core and hi-hat flashes |
| drop accent | #1FC8FF | swaps in for the accent on the drop, for 4 bars |

- Mapping: fixed palette, no gradient. Ribbon colour = `mix(rim, ribbon, shade)`. Dashes = `accent` with `highlight` at the dash centre.
- Over time: a slow hue drift of the indigo region (±10° per 32 bars) keeps long sets from going stale. On the drop the accent swaps orange → cyan over 1 beat; on a break, fill gain drops to 0 and only ribbons remain.
- LED walls: keep the black floor at #07070B (about 3 %), cap the ribbon at about 90 % luminance (bone, not pure white) to avoid blooming the wall and blinding the front row. Contrast between ribbon and fill should stay above about 10:1; check it on the wall, LED gamma crushes the indigo.

## Blending and compositing

- Fill and ribbons are one shader with alpha-over (Normal), so there are no blend seams between arcs.
- Dashes in Add, so they glow over the ribbon and never darken it.
- Video window: use the ribbon mask (alpha of S1) as a matte. In Arena, put the clip above the Truchet layer and use the Truchet layer as a luma mask (Mask layer or the "Layer Mask" approach), or multiply the video by the ribbon layer for "video inside the strands".
- Feedback: off in the main look; at 20–40 % on breaks, so rotating tiles leave arcs of light.
- Post chain, in order: bloom (threshold 0.75, intensity 0.8, radius medium) → chromatic aberration (drop only, 1.5 px) → grain (3 %) → optional sharpen 0.2 on low-res LED.

## Motion and timing

- Constant motion: flow dashes at 0.25–1.0 path lengths per beat; a slow pan of the whole grid, 0.02–0.1 cells per second, diagonal, so the image never stands still.
- Every bar (default) or every 2 bars: re-knot wave (technique 6) from a random origin, waveSpeed so the wave crosses the screen in 1 beat; ease smoothstep.
- Every phrase (8 or 16 bars): new origin pattern, e.g. centre → corners, or a horizontal wipe (use `id.x` instead of distance).
- Kick: line width pulse +30 %, attack 10 ms, release 180 ms. Optional "kick flips": re-roll 5–10 % of tiles instantly (step a third seed only for cells with a low hash).
- Hi-hats: brief highlight flash on dashes (dashGain ×2 for 60 ms).
- Drop: split 0 → 0.5 over 1 beat, accent to cyan, chromatic aberration on, density jumps up one step (e.g. 10 → 14 cells).
- Break: split back to 0 over 2 bars, fill fades out, feedback up to 35 %, flow slows to 0.25, re-knot every 4 bars.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| density | 4–40 cells (short side) | 10 | zoom: big bold loops to fine lace | KeyStep 37 mod strip or BeatStep Pro knob 1 |
| flip | 0–1 | 0.5 | 0.5 = endless maze; toward 0 or 1 = long diagonal rivers and small rings | knob 2, soft-takeover |
| width | 0.02–0.30 (cell units) | 0.12 | thin wire to fat ribbon | knob 3 + kick envelope +30 % |
| weave | 0–0.6 | 0.2 | share of crossing tiles: maze → woven cloth | knob 4 |
| split | 0–0.6 | 0 | multi-scale subdivision, richer and finer | knob 5; drop macro |
| reknot | trigger | — | fires a rotation wave and moves to the next seed | BeatStep Pro pad or BPM 1 bar; Resolume BPM sync |

## Build recipes

### Resolume Arena

- **Layer 1:** Solid Color source, #07070B.
- **Layer 2:** the S1 Truchet shader as a single-pass ISF source (fill, ribbons, crossings, dashes, rotation wave, subdivision in one pass). Resolume loads ISF shaders ([Resolume ISF support](https://resolume.com/support/en/isf)); if your version does not list ISF in Sources, wrap it in a Wire patch (below) and load that. Expose density, flip, width, weave, split, flowSpeed, seedA, seedB, waveProgress as parameters. Blend Alpha, 100 %.
- **Without a custom shader:** there is no built-in Truchet generator. A rough version: draw one tile image (two quarter arcs on black) in any tool, then use a Tile effect (e.g. 10 × 10). Every copy has the same orientation, so it gives Smith's regular pattern, not the random maze. Treat this as a fallback only.
- **Layer 3 (optional):** a video clip with a Mask, using layer 2 as the luma source, for the video-in-ribbons look.
- **Re-knot:** animate the shader's `waveProgress` 0 → 1 with the parameter's BPM sync (1 bar) in the clip's parameter animation, and step seedA/seedB on the bar. If Arena cannot step the seeds cleanly, do the bookkeeping in Wire.
- **Post on the composition:** Bloom (threshold about 0.75, amount about 0.8), RGB Split/chromatic effect bound to a drop dashboard button, Noise/grain at 3 %. Exact effect names depend on the Arena version.
- **Dashboard:** map the six parameters to BeatStep Pro knobs 1–6 via Arena's MIDI shortcuts; "drop" as one button that drives split, accent and aberration together.

### Resolume Wire

- `BPM` / beat trigger → `Counter` (bars) → on each bar: sample-and-hold a `Random` into seedB, a second `Random` pair into origin x/y, and reset a ramp.
- Ramp: `Time` (delta) × waveSpeed → accumulator, clamped 0..1 → `waveProgress`. When it reaches 1, copy seedB into seedA (a sample-and-hold triggered by the ramp end).
- `Audio Analysis` → low band → `Envelope` (attack 10 ms, release 180 ms) → × 0.3 → add to width. High band → envelope (attack 2 ms, release 60 ms) → dashGain.
- `ISF` node (the Truchet shader) with all inputs above → optional feedback loop (feedback node, decay 0.85, `Blend` Lighten) → bloom chain (threshold, blur, add) → output.
- Exposed parameters: density, flip, width, weave, split, flow, reknot (manual trigger), drop.
- Node names differ between Wire versions; check the node browser for the exact names.

### Unreal Engine 5.7

- **Material:** `M_Truchet`, domain Post Process (screen-filling output) or Surface/Unlit on a plane for 3D use. One Custom node (HLSL) holds techniques 1–7. Parameters come from a **Material Parameter Collection** `MPC_Truchet` (scalars: Density, Flip, Width, Weave, Split, Flow, SeedA, SeedB, T0, OriginX, OriginY, WaveSpeed, DashGain; vectors for the 5 colours). Emissive = the S1 colour; ribbon mask to a second output for opacity masking.
- **Without HLSL:** the Material graph can do the core: `ScreenPosition` or `TexCoord` × Density → `Floor` (id) and `Frac` − 0.5 (gv); a hash from `Dot` + `Sine` + `Frac` on id; `If` on hash < Flip to negate gv.x; `Length` of gv − corner, `Abs`(r − 0.5) and `SmoothStep` for the ribbon. Rotation and subdivision are much easier in the Custom node.
- **3D use:** the ribbon mask as World Position Offset or a heightmap on a subdivided plane gives embossed metal ornament; light it with a moving Rect Light. Or render to a Render Target (RT_Truchet, 2048²) with `Draw Material to Render Target` and use it as a Niagara mask.
- **Niagara (optional):** GPU sprites sample RT_Truchet and spawn only where the mask > 0.5; drive their velocity along the path (tangent = perpendicular to `gv - c`) to make "beads" running along the weave.
- **Timing:** a Blueprint ticks `T += Flow · DeltaSeconds` into the MPC; on a bar event (OSC from Resolume or a MIDI clock count) set SeedB, Origin, T0 and copy SeedA = SeedB when the wave is done.
- **Remote Control:** a preset exposing the MPC scalars, mapped through the OSC and MIDI protocol plugins ([Epic docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-protocols-in-unreal-engine)): BeatStep Pro knobs → Density, Flip, Width, Weave, Split; a pad → Reknot.
- **Post-process volume:** Bloom intensity 0.8, threshold 0.75; manual exposure (no eye adaptation); Film Grain 0.03; Chromatic Aberration 0 normally, 0.5 during the drop.
- **Output:** Spout via a third-party plugin, or NDI via the NDI SDK for Unreal, into Resolume as a source.

### Godot

- A `ColorRect` filling the viewport with a `canvas_item` shader for S1. Uniforms: `density, flip_prob, width, cross_prob, split, seed_a, seed_b, t0, origin (vec2), wave_speed, wave_delay, flow_speed, dash_gain, fill_gain`, and `col_base, col_base2, col_ribbon, col_rim, col_accent, col_highlight` (hint `source_color`). Accumulate time in GDScript `_process(delta)` and set the seeds on beat events.
- Feedback (optional): render into a `SubViewport` (`render_target_clear_mode` Never, HDR 2D on) and draw a second `ColorRect` with a shader that reads the `ViewportTexture` of the previous frame at decay 0.85, combined with `max()`.
- Post: a `WorldEnvironment` with Glow enabled (2D HDR), bloom 0.2, threshold 0.75, or do bloom in a second canvas pass.
- Audio: `AudioEffectSpectrumAnalyzer` on the music bus; read 40–120 Hz for the width pulse and 6–12 kHz for dash flashes.
- MIDI: `InputEventMIDI` maps CC numbers from the BeatStep Pro to the uniforms.

## Audio reactivity

| band | drives | smoothing | threshold |
|------|--------|-----------|-----------|
| sub/kick 40–120 Hz | width +30 %; optional 5–10 % instant flips | attack 10 ms, release 180 ms | onset above about 0.6 of running peak |
| low-mid 200–800 Hz | dash speed ×1–1.5 | attack 80 ms, release 400 ms | none (continuous) |
| hi-hat 6–12 kHz | dashGain ×2 flash | attack 2 ms, release 60 ms | onset only |
| full-band RMS, long average | split and density on the drop | attack 1 s, release 4 s | crossing 0.7 of the phrase maximum |

Prefer BPM sync over audio for the re-knot wave: a re-knot on every detected kick is too busy. Keep audio for width and dashes, and the grid change on bars.

## Performance tips

- S1 is very cheap: no textures, no loops apart from three subdivision levels. 4K at 60 fps on integrated GPUs is realistic. The optional feedback adds two full-res RGBA16F buffers, about 130 MB at 4K.
- Anti-aliasing is the main risk: at high density, ribbons get thinner than 2 px and shimmer when the grid pans. Clamp `width ≥ 1.5 px` in cell units (`width_min = 1.5 · density / res.y`) and limit density so a cell stays at least 24 px wide on the wall.
- Moiré on LED: very regular patterns beat against the LED pixel grid. Avoid density values where cell size is a near multiple of the LED module size, and keep the slow diagonal pan on, which breaks up aliasing.
- Odd aspect ratios (wide strips, towers): fit density to the short side (technique 1) so cells stay square; on a 10:1 strip use 3–5 cells across the short side, otherwise it reads as noise.
- Hash quality: a cheap `sin`-based hash can show patterns at large ids, and the grid pan grows the ids. Wrap the pan with `mod(pan, 1000)` or use an integer hash.
- Flicker: never re-roll all tiles in one frame on a big wall; the rotation wave is the softer way to change everything.

## Further references

- [Shader Coding: Truchet Tiling Explained!](https://www.youtube.com/watch?v=2R7h76GoIJM) — The Art of Code; builds the grid, hash, arc distance and the checker trick for continuous flow in a shader.
- [Multi-Scale Truchet Patterns](https://archive.bridgesmathart.org/2018/bridges2018-39.html) — Christopher Carlson, Bridges 2018; winged tiles and scale-by-1/2 subdivision.
- [Multi-scale Truchet patterns](https://observablehq.com/@makio135/multi-scale-truchet-patterns) — Lionel Radisson (makio135), Observable notebook implementing the multi-scale idea.
- [When Truchet met Chladni](https://chalkdustmagazine.com/features/when-truchet-met-chladni/) — Chalkdust; why flip probability 0.5 gives the percolation look, and hexagonal tiles.
