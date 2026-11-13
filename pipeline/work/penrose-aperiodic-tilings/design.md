# Penrose Aperiodic Tilings — design

## Look

A floor of rhombi with no edges, filling the whole frame. Every tile is a thick or thin rhombus outlined in thin cool white on near-black. The fills are dim: deep indigo for thick tiles and old gold for thin ones. Local five-pointed stars and decagons keep appearing, but they never line up into a repeating grid. The floor is never still. The five hidden grid offsets drift, so small groups of tiles flip in place ("phason flips"), and the pattern shimmers like a crystal re-deciding its structure. On snares a teal band sweeps along one ribbon, the wavy chain of tiles that share an edge direction. Hi-hats spark the vertices. On the kick, each tile splits into its golden-ratio children for a beat, which shows the self-similarity. For the second scene the camera falls into the centre of a ten-fold sun, with spinning trails behind the edges. This follows the Jessica In video for Max Cooper's "Penrose Tiling": trails that look random at first, slowly reveal the structure, and stack several scales on top of each other ([Creative Commission](https://creative-commission.com/project/max-cooper/music-video/jessin-0)). The mood is crystalline, mathematical and hypnotic: order you can feel but never pin down.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #05060B | Normal | 100 % | just above black, so LED modules never read as dead |
| 2 | Tile fill | pentagrid shader (P1), fill channel: thick/thin colour, per-tile brightness jitter ±10 % | Add | 35 % | keep it dim; the edges carry the image |
| 3 | Ribbon sweep | P1 ribbon channel: Gaussian band over ribbon index of one family | Add | 80 % | one family at a time, family steps per bar |
| 4 | Edges | P1 edge channel, constant pixel width, anti-aliased | Add | 100 % | main image |
| 5 | Vertex sparks | P1 spark channel: radial dot on tile corners, gated by hash and hi-hat | Add | 70 % | 1–3 % of vertices lit at a time |
| 6 | Growth scene | Robinson-triangle deflation shader (P2): edges at level L plus level L+1 ghost | Add | 0–100 % (scene fader) | crossfades with layers 2–5, same palette |
| 7 | Trails | feedback (P3) of layers 4 + 6 edges, rotate + zoom per frame | Add | 50 % | Cooper-style spinning trails; 0 % in the breathing scene by default |
| 8 | Matte (optional) | P1 edges in white only, used as luma matte for a video layer | Mask / Luma Key | 100 % | video seen only through the tile edges or only inside thin tiles |
| 9 | Post | bloom, grain, vignette | see compositing | — | order below |

## Techniques

**1. Pentagrid tiling per pixel (de Bruijn dual method)**
The definitions below follow [Le Bruyn's summary of de Bruijn's pentagrids](https://lievenlebruyn.github.io/neverendingbooks/de-bruijns-pentagrids). Family j holds the points x where `x·e_j + γ_j` is an integer, with `e_j = (cos 2πj/5, sin 2πj/5)`. The offsets satisfy `Σγ_j = 0`. The grid coordinates are `K_j(x) = ceil(x·e_j + γ_j)`, and a mesh region becomes the tiling vertex `V = Σ K_j e_j`. Every crossing of two grid lines becomes one rhombus. The shader inverts this per pixel:
1. **Pixel to tile space.** `p = rot(θ) · (fragCoord − 0.5·res) / res.y · scale`, where scale is the number of tile edges across the screen height (default 14).
2. **Guess the grid point.** On average `Σ(z·e_j)e_j = (N/2)·z` (N families) and the ceil adds about 0.5 per family. So the grid point behind p is roughly `z0 = (2/N)·(p − c)` with `c = Σ(γ_j + 0.5)·e_j`. This estimate is my own derivation. It is off by less than one line spacing, which makes a ±1 search window enough.
3. **Search crossings.** For each pair of families j<k (10 pairs for N=5), let `a_j = z0·e_j + γ_j` and `a_k = z0·e_k + γ_k`. Try the integer lines `n_j ∈ round(a_j) + {−1,0,1}` and `n_k ∈ round(a_k) + {−1,0,1}`, which gives 9 candidates per pair. Solve the 2×2 system for the crossing z*. The inverse matrix per pair is constant per frame, so precompute it.
4. **Build the rhombus.** For every other family i, `K_i = ceil(z*·e_i + γ_i)`. The base corner is `V0 = Σ_{i≠j,k} K_i e_i + n_j e_j + n_k e_k`, and the four corners are `V0 + {0, e_j, e_k, e_j+e_k}`.
5. **Inside test.** Solve `p − V0 = a·e_j + b·e_k`. If a and b both lie in 0..1, the pixel is in this tile. Keep (j, k, n_j, n_k, a, b) and stop at the first hit. If nothing hits, output magenta while debugging. Holes mean the search window or the z0 estimate is wrong.
6. **Read out.** Tile shape class is `d = min(|j−k|, N−|j−k|)`. For N=5, d=1 gives a thick rhombus (72°/108°) and d=2 a thin one (36°/144°). The edge distance in tile units is `sin(angle(e_j,e_k)) · min(a, 1−a, b, 1−b)`. The tile ID is `hash(j, k, n_j, n_k)`. The tile sits on ribbon n_j of family j and ribbon n_k of family k.

Why this method: it is stateless, has no CPU geometry and runs in one fragment pass. That makes it a fit for an ISF node in Wire, which only offers 2D fragment shaders ([Resolume ISF support](https://resolume.com/support/en/isf)). The grid (dualisation) method is one of the standard construction routes, alongside projection and inflation, reviewed by [Grimm and Schreiber](https://arxiv.org/abs/cond-mat/9903010).

**2. Breathing and phason flow (animating the offsets)**
1. Write the offsets as `γ_j = base_j + breath·sin(2π·f_j·t + φ_j) + u·e_{2j mod 5}`. Use incommensurate LFO rates `f_j` = 0.031, 0.047, 0.059, 0.071, 0.083 Hz.
2. Subtract the mean of γ every frame, so `Σγ_j = 0` holds and the result stays a Penrose tiling.
3. The term `u·e_{2j}` comes from a 2D "phason" vector u. Moving u is the cut-and-project idea of sliding the slice through 5D ([Wikipedia: Quasicrystal](https://en.wikipedia.org/wiki/Quasicrystal)). The tiles do not slide; small hexagonal groups of three tiles flip instead. The 5D framing here is my reading of the projection method; the visible flips are what you check on screen.
4. Pure translation is a different term: adding `v·e_j` to all γ pans the floor. Use it for camera drift, not for flips.
5. Default base offsets: (0.21, −0.34, 0.08, 0.27, −0.22). Never set all offsets equal: that is a singular grid, where more than two lines meet at one point, and it leaves holes. Le Bruyn notes that almost all offset choices are regular.

**3. n-fold switch**
1. Use N line families. Odd N takes directions `2πj/N`; even N takes `πj/N`, otherwise lines would come in parallel pairs. N=4 gives an 8-fold rhombus tiling, N=6 a 12-fold one, N=7 a 14-fold one. N=3 gives the periodic "stacked cubes" pattern, which works as a deliberate "order" contrast.
2. In `c = Σ(γ_j + 0.5)e_j` and `z0 = (2/N)(p − c)` the formulas stay the same; only the loop bounds change.
3. The number of shape classes is floor(N/2). Colour them from a ramp (see Color scheme).
4. Switch N on the bar line with a 1-beat crossfade between two renders, or as a hard cut on a drop. N cannot morph continuously.

**4. Robinson-triangle deflation per pixel (growth scene)**
This uses the subdivision rules from [Preshing](https://preshing.com/20110831/penrose-tiling-explained/), run per pixel the way the [shadcn Penrose shader](https://www.shadcn.io/shaders/penrose-tiling-shader) describes: each pixel keeps only the child triangle that contains it, for about 10 steps.
1. **Start wheel.** Ten "red" triangles (A at the origin, 36° apex) with B and C on the unit circle at angles (2i−1)π/10 and (2i+1)π/10; swap B and C for even i. Pick the wheel triangle from the pixel's angle.
2. **Red (A,B,C).** `P = A + (B−A)/φ`. The children are red (C,P,B) and blue (P,C,A). Test the pixel's side of line C–P.
3. **Blue (A,B,C).** `Q = B + (A−B)/φ` and `R = B + (C−B)/φ`. The children are blue (R,C,A), blue (Q,R,B) and red (R,Q,A). Test line R–A first, then line Q–R.
4. **Edges.** Draw only sides C–A and A–B, never the base B–C, as Preshing does. Two mirrored triangles then read as one rhombus. Store the edge distance at level L (bright) and level L+1 (ghost, weight g).
5. **Endless zoom.** Scale the view by `φ^u`, with u looping from 0 to 2. The shadcn page notes that after two deflation steps the floor matches its start mirrored. So mirror x at each wrap and the loop has no seam. Add the iteration count so tile size stays between about 20 and 120 px.

**5. Spinning trails (feedback)**
1. Buffer T at output resolution.
2. `T = T_prev(sampled at rot(ω)·(uv−0.5)/zoomF + 0.5) · decay + edges_now`.
3. Defaults are ω = 0.35° per frame, zoomF = 1.004 and decay = 0.90. Edges leave curved comet trails that spiral from the centre; when ω returns to 0 the trails collapse into the clean tiling. That is the "random first, structure later" reveal.

## Shaders

Render order per frame: P1 pentagrid, P2 growth, P3 feedback, P4 composite, P5 post. P1 and P2 are independent and can run in parallel or be skipped when their layer opacity is 0.

**P1 — Pentagrid tiling (single pass)**
- Purpose: tiles, edges, ribbons and vertex sparks for the main scene.
- Inputs: uniforms `N` (3–9), `gamma[9]` (computed on the CPU or node side with the mean already removed), `scale`, `rot`, `pan`, `lineWidthPx`, `ribbonFamily`, `ribbonPos`, `ribbonWidth`, `sparkSeed`, `sparkAmount`, `kickFlash`, `time`.
- Per pixel:
  1. Compute p and z0 (technique 1, steps 1–2).
  2. Loop over the pairs and the 3×3 candidates, inside-test, exit early on the first hit.
  3. Edge mask: `edgePx = edgeDist / pixelSize`, where pixelSize = scale/res.y; `edge = 1 − smoothstep(w−0.75, w+0.75, edgePx)`; add the halo `exp(−edgePx/5)·0.2`.
  4. Fill: shape class, times `0.9 + 0.2·hash(tileID)`.
  5. Ribbon: if j or k equals ribbonFamily, take that tile's n for the family and compute `exp(−((n − ribbonPos)/ribbonWidth)²)`.
  6. Spark: distance to the nearest of the 4 corners in px gives `exp(−d²/8)`, gated by `hash(cornerVertex + sparkSeed) < sparkAmount`. Hash the vertex position, not the tile, so all tiles that share a corner light together.
- Output: RGBA16F at output resolution. R = edge, G = fill class / (floor(N/2)), B = ribbon, A = spark. In Wire, put the palette into this pass and output colour directly (single pass, no data texture).

**P2 — Robinson growth (single pass)**
- Purpose: self-similar zoom with child edges fading in.
- Inputs: `zoomPhase u` (0–2), `levels` (8–12), `ghost g` (0–1), `rot`, `lineWidthPx`.
- Per pixel: run technique 4. Keep the edge distance at the last level and the level before it, and the triangle colour at the last level. Output `edge = max(edge_L, g·edge_{L+1})`.
- Output: RGBA16F at output resolution; same channel layout as P1 (no ribbons; sparks optional).

**P3 — Trail feedback (ping-pong)**
- Inputs: previous T, edge channels of P1 and P2 mixed by the scene fader, ω, zoomF, decay.
- Per pixel: technique 5. Clamp to 4.0.
- Output: two RGBA16F buffers swapped each frame. Half resolution is fine on 8K canvases.

**P4 — Composite**
- Inputs: P1, P2, P3, palette, scene fader, layer opacities.
- Per pixel: start from the background, then add fill·0.35, ribbon colour·ribbon, edge colour·edge, spark colour·spark, and trails·0.5. Mix P1 and P2 by the scene fader before adding.
- Output: RGBA16F.

**P5 — Post**
- Bloom (threshold 0.7, intensity 1.3, medium radius), monochrome grain 2 %, vignette 0.2. Then clamp to 8-bit for Spout or NDI.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base / black | #05060B | background |
| edge | #E8F1FF | tile outlines, cool white |
| thick fill | #1B2A6B | indigo, at 35 % or less |
| thin fill | #C9962E | old gold, at 35 % or less |
| accent | #35E0D0 | ribbon sweep, teal |
| highlight | #FFFFFF | vertex sparks, kick edge flash |
| shadow | #0B0F24 | ghost level and trails tint (trails are multiplied toward this) |

- **Mapping.** Fill is flat per shape class. For N above 5, the extra classes run along a fixed ramp #1B2A6B → #3B2F7A → #7A3B6B → #C9962E. Edges are a flat colour with brightness from the mask. Trails are tinted halfway from the edge colour to #0B0F24, so they read as afterglow and not as new geometry.
- **Alternative palette "Girih"**, for phrases that suit tilework, after the Isfahan link in the research: base #070A0C, edge #F2E6C9, thick #0E6E6E, thin #1A2A4F, accent #D9B26F.
- **Over time.** Change palette on 16-bar phrase boundaries, crossfading over 1 bar. Keep the edge colour near white so the structure stays readable.
- **Music.** Kick lifts the edges to #FFFFFF for 60 ms. Bass level scales fill opacity from 0.2 to 0.45.
- **LED walls.** Keep the black at #05060B. Cap the edges at 85 % before bloom so 1 px lines do not clip into blocky white. Fills above 45 % flatten the image into a quilt; keep contrast high.

## Blending and compositing

- Everything above the background is **Add**: light on black, so overlaps of edge, halo, ribbon and spark brighten instead of muddying.
- **Scene fader** (P1 ↔ P2): a linear mix before the trails. Both scenes share edge colour and width, so the crossfade reads as one structure turning into another.
- **Feedback.** Decay 0.90 (range 0.80–0.96). Above 0.96 the trails never clear and the tiling disappears into a disc of light.
- **Mattes.** Use the P1 edge channel as a luma matte: the video shows only along the edges, which looks like a stained-glass leading. Or use the thin-tile class as an alpha matte: the video shows through about 38 % of the floor, in scattered thin rhombi. That figure comes from the research: 16.18 thick tiles to 10 thin ones, so 10/26.18 of the tiles are thin.
- **Post chain in order.** Bloom (threshold 0.7, intensity 1.3), then grain 2 % monochrome, then vignette 0.2, then optional sharpen 0.2 for walls with a pitch above 4 mm. Skip chromatic aberration: it doubles the thin edges and breaks the crystal feel.

## Motion and timing

- **Breath.** The γ LFOs run continuously at 0.03–0.08 Hz with amplitude 0.05–0.4: slow flips everywhere, never periodic. Around 0.1 there are 2–3 flips per second on screen at scale 14.
- **Phason flow.** u moves at 0–0.5 units/s. Low speed gives rare, isolated flips. Above 0.3 units/s whole areas re-tile every second: a "storm".
- **Rotation.** 1 full turn per 64 bars in the breathing scene. In the growth scene, step 36° per phrase, because 36° keeps the ten-fold star aligned with itself.
- **Ribbon sweep.** On every snare (beats 2 and 4) ribbonPos advances by 1. ribbonFamily steps j → j+1 every bar, so over 5 bars every edge direction gets its turn. Ease ribbonPos with an exponential ease-out over 120 ms.
- **Kick.** Edge flash, 60 ms, and in the growth scene the ghost weight g jumps to 1 and decays over 1 beat. Every 4th kick (the downbeat) advances the zoom phase u by 1, a full φ step, eased with ease-in-out over 1 beat.
- **Bars and phrases.** Pan direction changes every 8 bars. N stays at 5 in verses; build sections go 5 → 7, and the drop cuts to 5 or 4.
- **Build.** Breath rises from 0.05 to 0.4 over 8 bars, phason flow ramps to 0.4, trails rise from 0 to 50 % and ω to 0.8°/frame. The floor dissolves into spinning noise.
- **Drop.** Hard cut. Set breath and flow to 0 and trails to 0, and cut to the growth scene at a fresh zoom phase with a white edge flash. The tiling snaps sharp: structure revealed.
- **Break.** Show only edges at 40 %, no fills. The ribbon sweep slows to one step per bar and scale drifts from 14 to 40 over the whole break (a zoom out into density).

## Parameters

| name | range | default | what it does on stage | suggested MIDI / BPM mapping |
|------|-------|---------|----------------------|------------------------------|
| scale | 4–60 tile edges across height | 14 | big readable tiles or a dense crystal texture | KeyStep knob 1, smoothed 200 ms |
| flow | 0–0.5 units/s (phason speed) | 0.05 | how fast tiles flip and re-tile; high = storm | bass envelope plus fader; BeatStep Pro knob |
| breath | 0–0.4 (γ LFO amplitude) | 0.1 | slow always-on shimmer of flips | fader; automate up in builds |
| families | 3, 4, 5, 6, 7, 9 | 5 | symmetry: 6-, 8-, 10-, 12-, 14-, 18-fold looks | BeatStep Pro pads 1–6, quantised to the bar |
| ribbon | ribbonPos step + width 0.5–4 | width 1.5 | teal wave travelling along one family | snare trigger steps it; knob sets width |
| growth | scene fader 0–1, ghost g 0–1 | 0 / 0.3 | swaps to the self-similar zoom; ghost shows the next level of tiles | crossfader for scene; kick envelope on ghost |

Secondary: edge width 0.5–4 px (default 1.5), trails amount 0–0.8, rotation speed.

## Build recipes

### Resolume Arena

Resolume documents ISF only inside Wire ([Resolume ISF support](https://resolume.com/support/en/isf)). So build the shaders in Wire and bring them into Arena as Wire patches.
- **Layer 1:** Solid Color source #05060B.
- **Layer 2:** the "Pentagrid" Wire patch as a source (P1 with the palette inside, output in colour). Expose scale, flow, breath, families, ribbon width, edge width and a ribbon-step trigger. Blend Add, 100 %.
- **Layer 3:** the "Penrose Growth" Wire patch as a source (P2). Blend Add, opacity on the Arena crossfader or a dashboard knob (the scene fader).
- **Trails:** do them inside the Growth patch (see Wire), or add Arena's own feedback-style effect on layer 3 if your version has one. Check its rotate and zoom options before the show.
- **Composition effects:** Bloom, then a grain or noise effect at a low amount, then a vignette.
- **BPM.** Map the ribbon-step trigger and the growth zoom step to Arena's BPM-synced parameter animation (step per beat or per bar). Map flow and edge flash to Arena's audio analysis: low band for flow, kick for the flash.
- **Matte use:** put a video layer above the Pentagrid layer with the Pentagrid patch in "white edges only" mode as its mask (luma), so the video shows through the edges.

### Resolume Wire

Patch "Pentagrid" (single-pass ISF, no multipass needed):
- `Time` → Math nodes for the five LFO sines (rates from technique 2) → multiply by `breath` → add `base_j` → add the phason term (`flow` integrated over time into u, dotted with e_{2j}) → subtract the mean → five float inputs gamma0..gamma4 of the ISF node. For N above 5, pass gamma5..gamma8 too; the shader ignores the unused ones.
- `BPM` / beat-clock node → counter for ribbonFamily (wrap N, step per bar) and ribbonPos (step on snare or every 2 beats).
- `Audio Analysis` → low band → envelope (attack 10 ms, release 400 ms) → added to `flow`. Kick onset → envelope (attack 1 ms, release 60 ms) → `kickFlash`.
- ISF node P1 → Bloom-style chain (Blur → Add over the original) → output.
- Exposed parameters: scale, flow, breath, families, ribbonWidth, edgeWidth, palette index.

Patch "Penrose Growth":
- `Time` and BPM → zoom phase u (advance 1 per bar, eased) → ISF node P2.
- ISF output → Wire's feedback loop with a Transform node (rotate ω, scale zoomF) and a multiply by decay. Keep feedback in Wire nodes: multipass and persistent buffers in ISF depend on the Wire version, and Resolume's ISF page does not cover them.
- Exposed: ghost, levels, trails amount, ω, decay.
- Wire has no vertex shaders, so the instanced-tile approach (UE5 and Godot below) does not apply here.

### Unreal Engine 5.7

Two routes; the second is the one that only UE can do well.
- **A, fragment route.** A post-process material, or an unlit material on a screen plane, with a Custom node holding the P1 loop in HLSL. Material Parameter Collection `MPC_Penrose` holds Gamma0..8, Scale, N, RibbonFamily, RibbonPos, EdgeWidth and KickFlash. A Blueprint or the Remote Control preset updates the MPC every tick: LFOs, phason u and mean removal on the game thread.
- **B, Niagara tile instances (smooth flips, 3D).** One GPU emitter.
  - Spawn Burst Instantaneous `pairs × (2M+1)²` particles; M=20 with N=5 gives 16,810. Each particle decodes its ID into (j, k, n_j, n_k).
  - Update: a Scratch Pad module computes the crossing z*, the K_i and the tile centre `V0 + (e_j+e_k)/2`. It sets a target position and orientation, angle(e_j) plus the shear choice, and MeshIndex = thick or thin.
  - Move the particle toward the target with a critically damped spring of about 120 ms. A phason flip then becomes a visible slide and turn of three tiles instead of a pop.
  - Mesh Renderer with two static meshes: thick and thin rhombus, unit edge, 0.05 thick extrusion.
  - Material: unlit emissive, edge glow from vertex UVs, fill from a per-particle colour.
  - Kick pushes Position.Z by `0.3·kick·hash` for a 3D ripple. Kill particles outside the view bounds by setting Scale to 0 (keep the IDs stable, do not kill).
- **Remote Control.** Create a Remote Control preset exposing Scale, Flow, Breath, N, RibbonWidth, Growth, Ghost and EdgeWidth. Bind OSC routes (for example `/penrose/flow`) and MIDI CCs from the KeyStep and BeatStep Pro.
- **Growth scene:** P2 as a second post-process material blended by Growth. Trails come from a render-target ping-pong in a Blueprint (Draw Material to Render Target, two RTs, swap each tick).
- **Post-process volume:** Bloom 1.3, threshold 0.7; manual exposure, so the flashes do not pump; Film Grain 0.02; Vignette 0.2.
- **Output:** Spout through a Spout plugin for UE, or NDI through the NDI SDK for Unreal, at the LED processor's native resolution.

### Godot

- **Pentagrid:** a `ColorRect` filling the viewport with a `canvas_item` shader holding P1. Uniforms: `int families`, `float gamma[9]`, `float scale`, `float rot`, `vec2 pan`, `float edge_px`, `int ribbon_family`, `float ribbon_pos`, `float ribbon_width`, `float spark_amount`, `float kick_flash`, plus palette `vec3`s. A GDScript `_process` computes the LFOs, phason and mean removal and sets `gamma`.
- **Smooth-flip variant:** a `MultiMeshInstance2D` with one instance per (j, k, n_j, n_k). GDScript computes the centres on the CPU (16,810 instances is fine) and lerps each instance transform toward its target. Use two MultiMeshes, one for thick and one for thin rhombus meshes.
- **Growth:** a second `ColorRect` with the P2 shader in its own `SubViewport`.
- **Trails:** two `SubViewport`s ping-ponged. A `ColorRect` shader samples the previous frame's `ViewportTexture` with rotation and zoom, multiplies by decay and adds the current edges.
- **Post:** `WorldEnvironment` with glow enabled (intensity 1.3, HDR threshold 0.7).
- **Control:** `InputEventMIDI` maps CCs and notes to the uniforms. Output through a community Spout or NDI addon if one is available for your Godot version.

## Audio reactivity

| band | drives | smoothing | threshold |
|------|--------|-----------|-----------|
| kick, 40–100 Hz onsets | edge flash; ghost level g (growth scene); every 4th onset advances zoom phase | attack 1 ms, release 60 ms (flash) / 1 beat (ghost) | onset at +6 dB over the 1 s average |
| bass, 60–250 Hz level | flow (phason speed) +0 to +0.3; fill opacity 0.2–0.45 | attack 80 ms, release 600 ms | none, continuous |
| snare, 1.5–4 kHz onsets | ribbon step (ribbonPos +1) | step, eased over 120 ms | onset at +5 dB |
| hi-hat, 6–12 kHz | vertex spark amount 0–3 %, new spark seed per onset | attack 1 ms, release 80 ms | gate at −30 dBFS |
| mids, 300 Hz–2 kHz energy | breath 0.05–0.25 | attack 200 ms, release 1.5 s | none |

Flips are the most "busy" element. Never drive flow from a fast envelope, or the floor turns to static. The kick gets the fast reactions; the tiling structure moves on slow envelopes.

## Performance tips

- **P1 cost:** pairs × 9 candidates, each needing N dot products and one 2×2 solve. N=5 means 10 pairs and at most 90 candidates; the early exit roughly halves the average. N=9 has 36 pairs, about 3.6× the work. On a desktop GPU N=5 at 4K is light (single-digit ms, an estimate, not benchmarked). On laptops render at half resolution and upscale only if edges are 2 px or wider.
- **Tiny tiles:** when tiles fall below about 6 px (scale above 60 on 1080 px height), edges alias into moiré. Fade the edge mask by `smoothstep(4, 10, tilePx)` and let the fill carry the image.
- **Precision:** at large pan values, float32 loses precision in `x·e_j`. Wrap pan by subtracting whole translation steps (keep |p| below about 10⁴ tile units).
- **Singular grids:** equal γ values or γ that pass exactly through a triple crossing give a one-frame hole. Mean removal plus the irregular base offsets avoids this in practice. If a hole flickers, nudge base_0 by 0.003.
- **Odd aspect ratios:** the tiling is infinite and has no period, so any aspect works without seams. This is a real advantage on strips and odd LED shapes. For multi-screen mappings compute p from the global canvas pixel, not per output, so tiles continue across screens.
- **P2:** 10–12 iterations of two side tests: very cheap. Keep enough levels that the smallest drawn tiles stay above 6 px, or the zoom shows mush in the centre.
- **Feedback:** RGBA16F, two buffers. On 8K canvases run it at half resolution; trails are soft anyway.
- **Pitfalls:** fills too bright (quilt look); chromatic aberration on 1 px edges; auto-exposure in UE; changing N mid-bar without a crossfade (the whole floor pops).

## Further references

- [Lieven Le Bruyn: de Bruijn's pentagrids](https://lievenlebruyn.github.io/neverendingbooks/de-bruijns-pentagrids) — grid families, the γ sum condition, K_j and the vertex map used in technique 1.
- [Jeff Preshing: Penrose Tiling Explained](https://preshing.com/20110831/penrose-tiling-explained/) — Robinson triangle subdivision rules and the ten-triangle start wheel used in technique 4.
- [shadcn.io: Penrose tiling shader](https://www.shadcn.io/shaders/penrose-tiling-shader) — per-pixel substitution in a fragment shader, plus a seamless golden-ratio zoom loop with the mirrored match after two steps.
- [byewokko/penrose](https://github.com/byewokko/penrose) — Python implementation of de Bruijn's multigrid method; a CPU reference for the grid-to-tile logic in the instanced (UE5 / Godot) route.
- [Grimm and Schreiber: Aperiodic Tilings on the Computer](https://arxiv.org/abs/cond-mat/9903010) — review of the projection, inflation and grid methods.
