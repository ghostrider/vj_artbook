# Hyperbolic Poincare Tilings — design

## Look

A round portal floats in black. Inside it, a curved ornament of heptagons (or squares, or triangles) repeats without end. The cells are large at the centre and shrink toward a hard circular rim, where they turn into a fine dust of arcs. Every edge is a circle arc that meets the rim at a right angle, so the pattern bends like a fish-eye view of a floor that has no end. Cells alternate between two tones, deep teal and warm brass, like Escher's black and white fish in Circle Limit III ([NGA](https://www.nga.gov/artworks/61280-circle-limit-iii)). Thin bone-white edge lines draw the grid, and a faint violet glow sits on the rim. The disk never sits still. It glides one cell per bar, so the viewer seems to fly across the floor while new cells keep pouring in from the rim and old ones sink back into it. On the kick the edges flash. On the drop the tiling type changes ({7,3} becomes {4,6}, for example), and the cells fill with a kaleidoscope of the live video feed. The mood is hypnotic, ornamental and slightly vertiginous: a mathematical mandala that is also a tunnel.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #000000 outside the disk | Normal | 100 % | true black: the disk reads as an object on LED walls |
| 2 | Rim glow | radial ring at Euclidean radius 1, Gaussian width 0.03 disk units, colour #3A1D5C | Add | 60 % | hides the unresolved dust at the rim |
| 3 | Tile fill | P1 fill channel: two-tone parity checker, darkened by fold depth | Normal (inside disk mask) | 100 % | main body of the image |
| 4 | Video-in tiles | P1 texture channel: live feed sampled at the folded point (kaleidoscope per cell) | Multiply over layer 3, or Normal on the drop | 0–100 % (fader `motif`) | off in the intro, full on the drop |
| 5 | Edges | P1 edge channel: polygon edges (and optional triangle mirrors) at constant hyperbolic width, anti-aliased | Add | 100 % | brightness pulses on the kick |
| 6 | Vertex stars | P1 star channel: small radial dots at polygon vertices, gated by a hash of the cell ID | Add | 50 % | hi-hat shimmer; Escher-style stars where tiles meet |
| 7 | Trails (optional) | feedback P2 of layers 5 + 6, zoom 1.0, no rotation, decay 0.85 | Add | 0–40 % | smears the glide into streaks on breaks |
| 8 | Disk matte (optional) | P1 mask channel (white inside, black outside) sent over Spout | used as Luma Key / Mask on another layer | 100 % | for "video only inside the disk" or "video only in odd cells" |
| 9 | Post | bloom, chromatic aberration, grain, vignette | see compositing | — | order below |

## Techniques

**1. Fundamental triangle as three mirrors**
A {p,q} tiling has regular p-gons with q meeting at each vertex. It is hyperbolic when (p−2)(q−2) > 4 ([Scientific American](https://www.scientificamerican.com/blog/roots-of-unity/everything-looks-better-in-the-hyperbolic-plane/)). It is built by reflecting a triangle with angles π/p, π/q and π/2 across its edges, where each reflection in a curved edge is a circle inversion ([westy31.nl](https://westy31.nl/PoincareMeetsMandelbrot/PoincareMeetsMandelbrot.html)).
1. Put the polygon centre at the disk origin. Mirror A is the x-axis. Mirror B is the line through the origin at angle π/p. The angle between them is the π/p corner.
2. Mirror C is a circle orthogonal to the unit circle, with centre (d, 0) and radius r, so d² = r² + 1. Its centre lies on the x-axis, so it meets mirror A at 90°. It must meet mirror B at π/q, which gives `d·sin(π/p) = r·cos(π/q)`.
3. Solve once per tiling (CPU or uniform setup): `r = 1 / sqrt(cos²(π/q) / sin²(π/p) − 1)` and `d = r·cos(π/q) / sin(π/p)`. The square root is real exactly when the tiling is hyperbolic. This derivation is mine; check it against the table below.
4. Useful derived points: edge midpoint at x = d − r; polygon vertex at radius `v = d·cos(π/p) − sqrt(d²·cos²(π/p) − 1)` along angle π/p; centre of the neighbouring polygon at x = 1/d (the inversion of the origin in mirror C).

Reference values (my own calculation; use them to test the shader):

| {p,q} | r | d | edge midpoint | vertex radius v | neighbour centre 1/d |
|---|---|---|---|---|---|
| {7,3} | 1.746 | 2.012 | 0.266 | 0.301 | 0.497 |
| {3,7} | 3.485 | 3.626 | 0.141 | 0.301 | 0.276 |
| {5,4} | 1.495 | 1.799 | 0.304 | 0.398 | 0.556 |
| {4,5} | 1.799 | 2.058 | 0.259 | 0.398 | 0.486 |
| {6,4} | 1.000 | 1.414 | 0.414 | 0.518 | 0.707 |
| {4,6} | 1.414 | 1.732 | 0.318 | 0.518 | 0.577 |
| {8,3} | 1.189 | 1.554 | 0.365 | 0.406 | 0.644 |
| {5,5} | 1.057 | 1.455 | 0.398 | 0.556 | 0.687 |

{6,4} is the tiling in Coxeter's figure that Escher saw in 1957 ([Dunham](https://www.math-art.eu/Documents/pdfs/Dunham.pdf)). {7,3} is the classic heptagon tiling with symmetry group (2,3,7) ([AMS Visual Insight](https://blogs.ams.org/visualinsight/wp-json/wp/v2/posts/946)).

**2. Per-pixel folding (inverse tiling)**
Instead of drawing millions of polygons, every pixel walks back into the fundamental triangle. This is the standard approach for fragment shaders and needs no geometry.
1. Loop up to N times (N = 40 default, 60 at 4K):
   - if y < 0: y = −y, flips += 1 (mirror A)
   - let n = (−sin(π/p), cos(π/p)); if dot(z, n) > 0: z = z − 2·dot(z, n)·n, flips += 1 (mirror B)
   - if |z − c| < r: z = c + r²·(z − c) / |z − c|², flips += 1, inversions += 1, scale *= r² / |z_before − c|² (mirror C)
   - if none of the three fired, break.
2. Output the folded point w, the parity `flips mod 2` (two-tone checker of triangles), and `inversions`, which counts how many polygons you crossed: a rough hyperbolic distance from the centre ("fold depth").
3. For a polygon checker (whole polygons in one colour) use `inversions mod 2` instead of flips. That works for even q; for odd q, such as {7,3}, adjacent polygons cannot be two-coloured, so colour by `inversions` through the palette instead.
4. Pixels that hit the iteration cap are at the rim. Give them the rim colour, not black, so no speckle shows.

**3. Moebius camera (glide and turn)**
Camera moves are isometries of the disk (Moebius transforms) applied to the pixel before folding. This is the standard technique noted in the research; the loop maths below is my own derivation.
1. Disk coordinates: `z = (fragCoord − 0.5·res) / (0.5·res.y) / diskSize`, with diskSize 0.95 so the rim stays on screen.
2. Rotation: `z = e^{iθ}·z`.
3. Translation along the x-axis by hyperbolic distance s: `z = (z + a) / (1 + a·z)` with real `a = tanh(s/2)` (complex maths; for a general direction use `(z + a) / (1 + conj(a)·z)` with complex a, |a| < 1).
4. Seamless one-cell loop: one cell step is s = 2·atanh(1/d). Drive it with bar phase t ∈ [0,1): `a = tanh(t·atanh(1/d))`. For even p this loops by itself. For odd p, also rotate by π·t **before** the translation; at t = 1 the composite (half-turn, then step) is a symmetry of the tiling, so the cut is invisible. (Derivation: the step equals reflection in the y-axis followed by mirror C; the y-axis is a mirror only for even p, and the half-turn repairs that.)
5. Rotation loop: θ = 2π/p per bar (or per phrase) is always a symmetry and keeps the parity colours.
6. Scale factor for the camera, needed for anti-aliasing: `|T′(z)| = (1 − |a|²) / |1 + conj(a)·z|²`. Multiply it into `scale`.

**4. Edges at constant hyperbolic width**
1. Euclidean distance to the polygon edge in folded space: `e = abs(|w − c| − r)`. Distance to the triangle mirrors (optional "fine mesh" mode): `abs(w.y)` and `abs(dot(w, n))`.
2. Hyperbolic distance: `h = 2·e / (1 − |w|²)`. Draw the line where h < lineWidth (default 0.04). Because the width is hyperbolic, lines thin toward the rim exactly like Escher's fish shrink ([Dunham](https://www.math-art.eu/Documents/pdfs/Dunham.pdf)).
3. Anti-alias in screen pixels: screen distance ≈ e / scale × (0.5·res.y·diskSize). Smoothstep over 1 px. When the line gets thinner than 1 px, fade its brightness by its width in pixels instead of drawing it aliased; that kills the moiré at the rim.
4. Vertex stars: distance from w to the vertex point (v·cos(π/p), v·sin(π/p)), same hyperbolic conversion, radius 0.12.

**5. Video and fractals inside the cells**
1. Every cell is a reflected copy of the fundamental triangle, so sampling a texture at w paints a mirrored kaleidoscope into every cell, shrinking toward the rim.
2. Texture UV: `uv = 0.5 + rot(texAngle)·w / (2·v) · texZoom`, sampled with mirrored repeat. texZoom 0.8–2.5.
3. To avoid the mirror look for faces and text, flip uv.x when parity is odd. The seams that remain fall on mirror lines and hide under the edge layer.
4. Fractal variant: feed w (or the unfolded hyperbolic polar coordinates) into a Julia iteration, as in Poincare meets Mandelbrot ([westy31.nl](https://westy31.nl/PoincareMeetsMandelbrot/PoincareMeetsMandelbrot.html)). Keep it to 32 iterations; it multiplies the cost.

## Shaders

Render order: P0 (CPU/uniform setup) → P1 (tiling, full res) → P2 (trails, optional, half res, ping-pong) → P3 (post, full res).

**P0 — tiling constants (per frame, CPU or a 1×1 pass)**
- Purpose: turn p, q into mirror data.
- Inputs: p, q (integers), bar phase, rotation phase.
- Steps: check (p−2)(q−2) > 4, else keep the previous pair. Compute r, d, n, v, 1/d (technique 1). Compute camera a and θ (technique 3).
- Output: uniforms `mirrorC = (d, 0, r)`, `mirrorB = n`, `vertex`, `camA`, `camTheta`.

**P1 — hyperbolic tiling (main pass)**
- Purpose: everything inside the disk.
- Inputs: P0 uniforms; `motifTex` (video, Spout or NDI); floats lineWidth, depthFade, motif, edgeGain, starGain, hueShift; palette colours.
- Per pixel:
  1. Map to disk coordinates; if |z| ≥ 1 output the rim glow only and stop.
  2. Apply the camera (rotate, then translate) and start `scale` with the camera derivative.
  3. Fold (technique 2) → w, flips, inversions, scale.
  4. Fill: colour A or B by parity; multiply by `exp(−depthFade·inversions)` (depthFade 0.05–0.3) so far cells sink into darkness.
  5. Motif: sample motifTex at uv from technique 5, mix with the fill by `motif`.
  6. Edges and stars (technique 4), added on top in the edge colour, times edgeGain.
  7. Soft disk edge: multiply by smoothstep(1.0, 0.985, |z|) to avoid a hard aliased circle.
- Output: RGBA, alpha = disk mask. Optional second output (or a second instance) with edges only in white, for the trails and for masks.

**P2 — trails (optional)**
- Purpose: streaks during the glide on breaks.
- Inputs: P1 edges, previous P2 frame.
- Per pixel: `out = max(edges, prev × decay)` with decay 0.80–0.92; no zoom or rotation, since the glide already moves the image.
- Output: half-res persistent float buffer, upscaled with bilinear filtering.

**P3 — post**
- Bloom threshold 0.75, intensity 1.2, radius medium; chromatic aberration 1–2 px, radial from the disk centre, scaled by |z| so the centre stays sharp; film grain 3 %; no extra vignette (the black surround already frames it).

## Color scheme

| role | hex | use |
|---|---|---|
| base A | #0E3B43 | deep teal, even cells |
| base B | #C8963E | warm brass, odd cells |
| highlight | #F2EBDD | bone white, edges |
| accent | #E0457B | rose, vertex stars and kick flash tint |
| rim | #3A1D5C | violet rim glow |
| shadow / outside | #000000 | outside the disk, and the far depth fade |

- Mapping: parity picks A or B; fold depth darkens both. In "depth mode" (odd q, or on the drop), use a cosine palette `col = 0.5 + 0.5·cos(2π·(inversions/8 + hueShift + (0.0, 0.33, 0.67)))` and then lerp 40 % toward base A, so it stays on palette instead of turning into a rainbow.
- Over time: hueShift drifts 0.01 per bar in the verse; on every phrase A and B swap over 1 beat (the checker inverts, a strong but cheap change).
- LED walls: keep cell fill at most 70 % luminance so the edges (100 %) carry the image. Outside the disk is true black; the darkest cell after depth fade should still be at least 3 % so the disk does not dissolve into the background.

## Blending and compositing

- Fill is Normal inside the disk mask: tiles are opaque surfaces; Add would wash out the two-tone checker.
- Motif video in Multiply over the fill keeps the teal/brass identity in the intro build; switch to Normal on the drop so the video takes over and only the edges stay.
- Edges, stars, rim and trails in Add: light on top of surfaces, and it reads well on LED.
- Masks: the disk alpha (or the parity channel as a 1-bit matte) goes out over Spout to cut another Resolume layer into "only inside the disk" or "only the odd cells".
- Trails at 0–40 %; above that the rim turns to mush.
- Post order: trails mix → bloom → chromatic aberration → grain.

## Motion and timing

- Glide: one cell step per bar (technique 3), linear in bar phase; for a punchier feel, ease with smoothstep over the last beat of the bar so the floor jumps one cell per bar.
- Rotation: 2π/p per phrase (8 bars), linear. Lock the direction to the glide direction for a corkscrew; reverse it on the second half of the phrase for a sway.
- Kick: edgeGain 1.0 → 2.2, decay 120 ms; line width +30 % for the same envelope.
- Snare: optional half-step of glide (an eased push of 0.5 cell) on beat 2 and 4 instead of the steady glide.
- Break: glide slows to one cell per 4 bars, trails up to 35 %, motif off, depthFade high (0.3) so only the centre cells show: a small calm mandala.
- Build: depthFade sinks to 0.05 over 8 bars, so more and more cells come out of the dark toward the rim.
- Drop: on the downbeat, cut (or 1-beat crossfade between two P1 instances) to the next {p,q} pair; motif to 100 %; glide back to one cell per bar or faster (two per bar).

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|---|---|---|---|---|
| tiling | index 0–7 into {7,3}, {3,7}, {5,4}, {4,5}, {6,4}, {4,6}, {8,3}, {5,5} | 0 ({7,3}) | changes the whole ornament | BeatStep Pro pad row (one pad per pair) or KeyStep keys C3–G3; snap, no in-between values |
| glide | 0–4 cells per bar | 1 | flight speed across the floor | KeyStep knob; synced to BPM bar phase |
| turn | −1 to 1 turns of 2π/p per phrase | 0.25 | rotation of the disk | knob, centre detent = stop |
| depth | depthFade 0.3–0.0 | 0.12 | how far toward the rim cells stay visible | fader; automate rising over a build |
| motif | 0–1 | 0 | live video or fractal inside the cells | fader; jump to 1 on drop |
| pulse | 0–1 | 0.6 | how hard the kick flashes the edges | knob; audio drives the envelope, this scales it |

## Build recipes

### Resolume Arena

- Layer 1: Solid Color #000000, or just an empty composition background.
- Layer 2: the Wire patch (below) as a **source**, blend mode Alpha, published params tiling, glide, turn, depth, motif, pulse, lineWidth, hueShift. Resolume documents ISF only inside Wire, through the ISF node ([Resolume ISF support](https://resolume.com/support/en/isf)), so the custom shader runs in Wire.
- Motif input: put the video (camera, NDI source, or a clip) on its own layer, route it into the patch through a Spout or NDI input node, or send the layer via Spout. Hide that layer (opacity 0) or keep it for the "outside disk" region.
- Effects on layer 2, in order: Bloom (threshold ~0.75, amount ~1.2), Chromatic Aberration (small), Noise/grain at 3 %. Check names and ranges in your Arena build.
- Tiling change on the drop: two copies of the source clip on adjacent columns with different `tiling` defaults; trigger the next column with a 1-beat transition time on the layer.
- BPM: set the composition BPM, and drive `glide` phase from the BPM clock inside Wire, so the loop resets on bar boundaries. Use Arena's Audio FFT on `pulse` (low band) as a parameter animation.
- Clip control via OSC/REST: opacity and parameters are reachable through Resolume's REST API ([Resolume REST API](https://resolume.com/docs/restapi/)), useful if UE or a sequencer triggers tiling changes.

### Resolume Wire

- **Clock:** BPM / bar-phase node → `phase` (0–1 per bar) → multiply by glide → fractional part → ISF input `glidePhase`. A second phase per phrase (8 bars) × turn → `turnPhase`. Check what your Wire version calls these nodes.
- **Setup maths:** p and q from a lookup (an Array/Select node indexed by `tiling`) → ISF integer or float inputs. The r, d, v maths is cheap enough to do inside the ISF per pixel (it is per-frame constant, so the GPU compiler hoists most of it); that avoids building it in nodes.
- **ISF node `Tiling` (P1):** image input `motifTex` from a Spout Receiver or NDI node; float INPUTS with MIN/MAX for glidePhase, turnPhase, lineWidth (0.01–0.12), depthFade (0–0.3), motif (0–1), edgeGain (0–3), starGain (0–1), hueShift (0–1), texZoom (0.8–2.5), diskSize (0.6–1.0); color INPUTS for A, B, edge, star, rim; long input for p and q.
- **Trails (P2):** a second ISF node with a PERSISTENT, FLOAT buffer, as in the [ISF multipass reference](https://docs.isf.video/ref_multipass.html). Resolume's ISF page does not cover multipass, so test it on your Wire version; fall back to Wire's own feedback node if needed. Wire has no vertex shaders ([Resolume ISF support](https://resolume.com/support/en/isf)); this effect needs none.
- **Audio:** audio-analysis node → low band → envelope (attack 5 ms, release 120 ms) × pulse → edgeGain = 1 + 1.2·env.
- **Outputs:** main texture to the patch output; optional Spout Sender with the disk mask.
- **Published:** tiling, glide, turn, depth, motif, pulse, lineWidth, hueShift.

### Unreal Engine 5.7

- **Material approach (recommended):** a post-process material or a material on a full-screen plane / dome mesh, Unlit. All of P1 lives in one **Custom** node (HLSL) because the folding loop needs a dynamic loop; the rest is plain material nodes:
  - TexCoord (or ScreenPosition for post-process) → subtract 0.5 → multiply by aspect → divide by diskSize → into the Custom node.
  - Scalar parameters from a **Material Parameter Collection** `MPC_Hyper`: P, Q, GlidePhase, TurnPhase, LineWidth, DepthFade, Motif, EdgeGain, HueShift.
  - Custom node outputs: fill colour, edge mask, disk mask (use additional outputs).
  - Motif: Texture Sample of a Media Texture (camera or video) or a Spout receiver texture, UV from the Custom node.
  - Emissive = fill × (1 − motif) + motifColour × motif + edge × EdgeColour × EdgeGain.
- **Dome option:** the disk maps naturally onto a fisheye dome master; render the material on a plane and capture it with a SceneCapture2D for a fisheye output.
- **Niagara (optional extra):** sparks at the vertices are hard to place in the curved space from Niagara; instead, use a GPU sprite emitter that samples the edge mask render target (Sample Texture module, spawn where mask > 0.5), with short-lived additive sparks on hi-hats.
- **Post-process volume:** Bloom intensity 0.8, threshold 1.0; Film Grain intensity 0.1; disable auto exposure (fixed EV) so the black surround stays black.
- **Blueprint:** compute bar phase from BPM (Event Tick, accumulate delta × BPM / 240), write it to MPC_Hyper. Tiling index → P, Q via a small array.
- **Remote Control:** expose Tiling (int), Glide, Turn, Depth, Motif, Pulse and BPM from the Blueprint or the MPC to a Remote Control Preset; bind them to OSC addresses (e.g. /hyper/glide) and MIDI CCs from the KeyStep 37 / BeatStep Pro pads (tiling).
- **Output:** Spout or NDI sender of the final render (or of the disk mask on a second capture) into Resolume.

### Godot

- **Node setup:** a ColorRect filling the viewport with a CanvasItem `shader_type canvas_item;` material running P1. Use a Control/CanvasLayer root so it scales with the window.
- **Uniforms:** `int p`, `int q`, `float glide_phase`, `float turn_phase`, `float line_width`, `float depth_fade`, `float motif`, `float edge_gain`, `float hue_shift`, `vec4 col_a, col_b, col_edge, col_star, col_rim` (with `source_color` hint), `sampler2D motif_tex` (from a VideoStreamPlayer's texture or a CameraTexture).
- **Trails:** a SubViewport (update mode Always, Clear Mode Never) whose ColorRect draws `max(edges, previous × decay)`; read the previous frame through a ViewportTexture of the same SubViewport (or ping-pong two SubViewports if your Godot version complains about a feedback loop).
- **Script:** compute bar phase from BPM in `_process`, set shader params; map MIDI with `InputEventMIDI` (CC → glide/turn/depth/motif, note → tiling).
- **Particles:** GPUParticles2D for vertex sparks is optional; same caveat as Niagara (sample the edge mask).

## Audio reactivity

| band | drives | smoothing | threshold |
|---|---|---|---|
| kick, 40–120 Hz | edgeGain flash (1 → 2.2) and line width +30 % | attack 5 ms, release 120 ms | trigger above −18 dBFS band level, or use transient onset |
| bass, 60–250 Hz, RMS | turn speed ±20 % around the set value | attack 200 ms, release 800 ms | none, continuous |
| snare / mids, 1–3 kHz | optional half-cell glide push | eased over 1 beat | onset detect, retrigger lockout 200 ms |
| hats, 6–12 kHz | vertex star gate: hash of cell ID lights 5–15 % of stars; also hueShift +0.002 per hit | attack 2 ms, release 60 ms | onset detect |
| full-band loudness | depthFade (louder = cells visible further toward the rim) | attack 1 s, release 3 s | none |

Keep tiling changes manual or on phrase boundaries: audio-driven changes of {p,q} look random.

## Performance tips

- P1 cost grows with the fold count, which grows roughly with the hyperbolic distance from the centre: centre pixels need 1–5 folds, rim pixels 30+. A 40-iteration cap at 1080p is light on any current GPU; at 4K or on a wide LED canvas, render P1 at native resolution only inside the disk's bounding square and let the rest be black.
- Odd aspect ratios: the disk is round, so on a 32:9 or 4:1 LED wall it leaves big black sides. Options: two or three disks side by side (offset camera per disk), the half-plane model (variation 5 in the research) for a wide strip, or the disk as a mask for a wide second layer.
- Do not draw constant-pixel-width lines near the rim; they turn into moiré on LED processors. Use the hyperbolic width plus the sub-pixel fade (technique 4).
- Keep the soft disk edge (1.5 % smoothstep); a hard circle aliases visibly on LED pixels.
- Video-in tiles minify heavily toward the rim. Enable mipmaps on the motif texture (Godot: filter with mipmaps; UE: Media Texture with mips) or rim cells shimmer.
- Large p or q (above about 8) gives very thin, stretched cells near the rim, as the EscherSketch notes warn for polygons ([GitHub](https://www.github.com/looeee/hyperbolic-tiling)); keep the stage list to p, q ≤ 8.
- Float precision: near |z| = 0.999 the inversions lose precision; the iteration cap plus the rim glow hide it. Do not set diskSize above 1.0 to zoom; zoom by translating instead.

## Further references

- [Dunham: Escher's use of the Poincare models](https://www.math-art.eu/Documents/pdfs/Dunham.pdf) — the disk model, circle-arc lines, conformality and Escher's Circle Limit history behind techniques 1 and 4.
- [westy31.nl: Poincare meets Mandelbrot](https://westy31.nl/PoincareMeetsMandelbrot/PoincareMeetsMandelbrot.html) — triangle reflections as circle inversions and fractal content inside a {7,3}-based tiling (technique 5).
- [EscherSketch (looeee/hyperbolic-tiling)](https://www.github.com/looeee/hyperbolic-tiling) — open-source WebGL prototype mapping motif images onto hyperbolic triangles; a working reference for the two-colour fish style.
- [AMS Visual Insight: {7,3} tiling](https://blogs.ams.org/visualinsight/wp-json/wp/v2/posts/946) — the (2,3,7) triangle group and the heptagon tiling used as the default.
