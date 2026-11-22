# Lenia Continuous Artificial Life — design

## Look

A dark-field microscope slide fills the wall. Soft, translucent creatures drift across near-black water. They are discs with fuzzy rims and faint concentric rings inside, like plankton or amoebae under a lens. The main species is Orbium, the disc-shaped glider from Chan's paper ([arXiv 1812.05433](https://arxiv.org/abs/1812.05433)). Each creature shows its own direction: the growing front edge glows amber and the decaying tail fades to magenta, so even a single glider reads as alive and on its way somewhere. Faint ghost trails show where creatures have been. Kicks drop new clusters of "primordial soup" that either pull together into new creatures or dissolve. Bass makes the whole population swell and shrink. On the drop the wall is reseeded and the palette changes from teal to violet. On a break time slows down, creatures shrink, and some die. The mood is slow, breathing and biological. The motion is organic, not on the grid. Beats show up as births and lurches, not as flashes.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #02070B | Normal | 100 % | where A < 0.02; crushed to true LED black in post |
| 2 | Food map (hidden, optional) | blurred clip or camera luma | — | — | lowers mu locally so creatures gather on bright shapes (Technique 5) |
| 3 | Kernel LUT (hidden) | normalised kernel weights, baked once per parameter change | — | — | P0; 27×27 R32F for R = 13 |
| 4 | Sim state (hidden) | ping-pong RGBA16F: R = state A, G = growth G(U), B = potential U, A = unused | — | — | P1, 1–4 steps per frame |
| 5 | Inject (inside sim) | noise-soup discs, species stamps, brush | inside P1 (first step of the frame) | — | keeps the world from dying or settling |
| 6 | Trail buffer (hidden) | persistent `trail = max(trail · decay, A)` | — | — | P2; ghost paths |
| 7 | Shaded Lenia | A → ramp, growth-front tint, rim light, trail underlay | Normal | 100 % | main visible layer (P3) |
| 8 | Video under (optional) | any clip, seen through the creatures | Lenia as luma mask, or Lenia in Screen over the clip at 60–80 % | 0–100 % | variation: creatures as windows |
| 9 | Post | black crush, bloom, 1 px chromatic aberration, grain | see compositing | — | light; the field is already soft |

## Techniques

**1. Lenia update (the core)**
1. The state is one float A per cell, 0..1. Each step has three stages: potential, growth and update ([arXiv 1812.05433](https://ar5iv.labs.arxiv.org/html/1812.05433)).
2. Potential `U = Σ K(n) · A(x + n)` over all offsets n with |n| ≤ R. K is a normalised ring kernel (Technique 2).
3. Growth `G = 2 · exp(−(U − mu)² / (2 · sigma²)) − 1`. G is +1 when U is exactly mu and close to −1 far from it ([arXiv 1812.05433](https://ar5iv.labs.arxiv.org/html/1812.05433)).
4. Update `A' = clamp(A + dt · G, 0, 1)`, with dt = 1/T.
5. Orbium "genes": R = 13, T = 10 (dt = 0.1), mu = 0.15, sigma = 0.015, ring heights beta = [1] (these values are used in the [CAX Lenia example](https://maxencefaldor.github.io/cax/examples/20_lenia/)). This is the default world. Other species from Chan's catalogue need their own R, T, mu, sigma and beta values, which are listed in his code and web demo ([GitHub](https://github.com/Chakazul/Lenia)).
6. Why it looks alive: cells with "just enough" neighbours grow and others decay. A disc then grows on one side and shrinks on the other, so it glides. The decaying side is where G < 0 inside the body. Technique 6 makes that visible.

**2. Ring kernel**
1. For each offset n, `r = |n| / R` (0..1, zero outside the disc).
2. Shell: with B rings and heights beta, `Ks = beta[floor(B·r)] · Kc(fract(B·r))`. The core is exponential, `Kc(q) = exp(4 − 1 / (q · (1 − q)))` for 0 < q < 1 ([arXiv 1812.05433](https://ar5iv.labs.arxiv.org/html/1812.05433), α = 4).
3. Normalise: `K = Ks / ΣKs`, so U stays in 0..1 and mu keeps its meaning at any R.
4. Bake K into a (2R+1)² float texture (P0) whenever R, B or beta change. It costs nothing at runtime and keeps exp() out of the hot loop.
5. Rank 1 (Orbium) gives one soft ring. Rank 2–3 with beta like (1, 2/3, 1/3), which appears in the paper's figures, gives creatures with more internal rings and a more complex outline.

**3. Speed without breaking the creature**
1. dt is part of the species. Orbium is tuned for dt = 0.1. Do not raise it to speed up the world. Above about 0.2 the creatures get coarse and tend to die or explode. That is a guideline from using it; test your own species.
2. Speed = steps per frame. The default is 2 steps per frame at 60 fps (120 steps/s). The range on stage is 1 (calm) to 4 (the drop).
3. A beat "lurch" is the one exception: dt goes from 0.1 to 0.16 for 120 ms on a kick and then eases back. Short and small enough to survive, visible as a forward jolt.
4. Lock the output fps. Speed is counted in steps per frame, so a frame drop slows the life down.

**4. Injection and the watchdog (anti-death, anti-mush)**
1. The research names two failures: a lone glider that just drifts, and worlds that die or fill the screen.
2. Soup seed (kick): a disc with radius 1.2·R cells at a hashed position (hash of the beat counter). Inside it, `A = max(A, noise(cell) · 0.9 · inject)` with per-cell white noise. At Orbium parameters, random patches often condense into gliders or vanish within about 1–2 s. Both outcomes look good.
3. Species stamp (pad): copy a small A-only texture of a known creature into the field at the brush position, with a rotation of 0/90/180/270°. Build the stamp library from your own sim: freeze, crop a 32×32 area around a healthy creature and save it as a float EXR. Chan's web demo can also load named species to look at first ([web demo](https://chakazul.github.io/Lenia/JavaScript/Lenia.html)).
4. Brush: an XY position (KeyStep or OSC) paints soup with radius 0.5·R while a gate is held.
5. Watchdog: mean A from a mip or reduction of the state (P4). If mean A < 0.0005 for 2 s (dead), auto-seed 8 soup discs. If mean A > 0.25 (screen filling), raise mu by +0.004 for 4 bars, which makes growth harder, and then ease it back.

**5. Footage as food**
1. Food map `f = blur(luma(clip))`, at sim resolution, 2–3 % blur radius.
2. Local growth centre `mu_local = mu − food · f · 0.01`. Bright areas grow more easily, so creatures swell and gather there. Dark areas stay as they are.
3. Keep `food · 0.01` ≤ 0.012 (about 8 % of mu). More than that and bright areas fill solid.
4. Slow footage only (silhouettes, slow pans). The creatures react over seconds, not frames.

**6. Growth-aware shading**
1. The sim already has G in its G channel, so the render knows where each body is growing (G > 0) and dying (G < 0).
2. Front tint: `front = clamp(G, 0, 1) · smoothstep(0.03, 0.25, A)`. Rear tint: `rear = clamp(−G, 0, 1) · smoothstep(0.05, 0.5, A)`.
3. Rim light: `rim = |∇A| · rimGain` from central differences, with rimGain 6–10. This brings out the inner rings like phase contrast.
4. Trail: P2 keeps `trail = max(trail · 0.97, A)` per frame (half-life about 0.4 s at 60 fps). It is drawn at 15–25 % under the body.

## Shaders

Render order per frame: P0 (only on change) → inject + P1 × N steps (ping-pong) → P2 trail → P4 watchdog (every 30 frames) → P3 shade → post.

**P0 Kernel bake**
- Purpose: precompute the normalised kernel.
- Inputs: R (int, 6–15), rank B (1–3), beta[3].
- Per texel (offset n = texel − R): r = |n|/R. If r ≥ 1 the weight is 0, otherwise Ks as in Technique 2. A second tiny pass (or the CPU) sums all Ks. The final texture stores Ks / sum.
- Output: `kernelTex`, (2R+1)² R32F, Nearest.

**P1 Lenia step (ping-pong S0 ↔ S1)**
- Purpose: one Lenia time step.
- Inputs: `stateTex` (previous), `kernelTex`, `foodTex` (optional), uniforms `mu, sigma, dt, R, food, inject, seedPos, seedOn, brushPos, brushOn, stampTex, stampPos, stampOn, frameSeed, isFirstStep`.
- Per cell:
  1. Read A.
  2. If isFirstStep: apply the soup seed, brush and stamp to A (Technique 4). The injected value feeds into this step's convolution only through the next step, which is fine.
  3. Loop dy, dx from −R to R. Skip zero kernel weights (corners). `U += kernel(dx, dy) · state(cell + (dx, dy)).R`. Sample at texel centres with wrap addressing (torus world).
  4. `mu_l = mu − food · foodTex · 0.01`; `G = 2·exp(−(U − mu_l)² / (2σ²)) − 1`.
  5. `A' = clamp(A + dt·G, 0, 1)`. Write (A', G, U, 0).
- Resolution: 640×360 default (16:9), 960×540 high. Format RGBA16F at minimum. 8-bit kills Lenia because dt·G steps of 0.1 × small values round away.
- Output: the other state buffer. Steps per frame is an integer, the same every frame. Track which buffer is current.

**P2 Trail**
- Inputs: current state, `trailTex` (persistent, its own ping-pong pair or read-modify-write), `trailDecay` 0.90–0.99.
- Per pixel: `trail' = max(trail · trailDecay, A)`.
- Output: `trailTex`, sim resolution, R16F.

**P3 Shade (full output resolution)**
- Inputs: state (bilinear, upscaled), `trailTex`, palette uniforms, `frontAmt`, `rearAmt`, `rimGain`, `trailAmt`, `paletteMix`.
- Per pixel:
  1. A, G from state. `v = smoothstep(0.02, 0.9, A)`.
  2. Body colour = 4-stop ramp on v (Color scheme). `paletteMix` blends palette A → palette B.
  3. Add front tint `amber · front · frontAmt` and rear tint `magenta · rear · rearAmt`.
  4. Rim: sample A at ±1 sim texel, `rim = length(∇A) · rimGain`, add `ringMint · rim · 0.6`.
  5. Underlay: `col = max(col, trailColor · trail · trailAmt)`.
  6. Floor: if A < 0.02 and trail < 0.03, output exactly the background colour.
- Output: RGBA8 to the comp or to Spout/NDI.

**P4 Watchdog (cheap)**
- Downsample the state to 64×36, then to 1×1 by mips or a reduction. Read the mean A every 30 frames (CPU readback stalls if done every frame). Logic as in Technique 4.5.

## Color scheme

Palette A, "dark-field teal" (default):

| hex | role |
|-----|------|
| #02070B | background / black level |
| #0A2633 | shadow, thin body edge (v 0–0.25) |
| #127C7A | body base (v 0.25–0.6) |
| #63D6B4 | inner rings, rim light (v 0.6–0.85) |
| #EAF8EE | highlight core (v 0.85–1) |
| #FF9F43 | accent: growing front |
| #B83A78 | accent: decaying tail |
| #1B4E5C | trail colour |

Palette B, "bioluminescent violet" (drop): #07020E background, #2A1450 shadow, #7B3FE4 body, #C9A2FF rings, #FFF1FB core, front #FFD166, tail #3FC1C9, trail #2E1B5E.

- Mapping: a 4-stop gradient ramp on `v`, linear between stops. Front and tail tints are additive on top. `paletteMix` crossfades A → B over 1 bar on the drop, and back over 4 bars after it.
- Slow drift: hue rotate the body stops ±8° on a 32-bar sine. The accents stay fixed so direction stays readable.
- LED walls: background ≤ #02070B, and the post crush sets anything below 3 % luma to 0, so empty water is truly off. Keep the highlight core below 92 % brightness. Large Orbium cores at full white are glaring on high-nit panels. Contrast comes from dark water against glowing bodies, not from pure white.

## Blending and compositing

- Inside P3 the tints are additive and the trail uses max (Lighten). Max keeps trails from brightening bodies they overlap, so the creature stays the brightest thing.
- Over footage: (a) creatures as windows. Put the Lenia luma as the mask of a clip layer, so the clip shows only inside the bodies; the clip sits on top of a dark background. (b) Lenia in Screen at 60–80 % over dark footage, so creatures glow over it. Do not use Add over bright footage, because it washes out.
- Displacement variation: use A (blurred 3 px) as a height map to refract a second clip, with displacement strength 8–20 px at 1080p.
- Post chain in order: black crush (levels, input black 3 %) → Bloom (threshold 0.65, intensity 0.8, radius 2 % of height) → chromatic aberration 1 px at 1080p (adds a lens feel, more blurs the rings) → film grain 3 %, monochrome, in Overlay. No sharpening: Lenia must stay soft.

## Motion and timing

- Creature speed: fixed by the species. Orbium glides steadily, a few cells per second at the default speed. The global speed fader (steps per frame 1–4) is the main tempo control. Change it on bar lines only, eased over 1 beat by alternating step counts (for example 2,3,2,3).
- Beats: kick = soup seed plus a dt lurch (0.1 → 0.16, 120 ms, ease-out). Snare = mirrored second seed across the centre. Hats = a single tiny brush dab, radius 0.3·R.
- Bars: every 4 bars, stamp one known creature at a hashed edge position, heading inwards (choose the rotation so it points to the centre).
- Phrases (16 bars): mu drifts on a sine, ±0.004 around the species value, period 16 bars. The population breathes between fatter and leaner bodies.
- Drop: reseed 10 soup discs in a ring at 35 % of the frame height around the centre, speed 4 steps per frame for 8 bars, palette A → B over 1 bar, bloom intensity +0.4 for 1 beat.
- Break: speed 1 step per frame, sigma −0.001 (creatures shrink, some die), trail decay 0.99 (long ghosts), seeds off. The watchdog stays on so the wall never goes fully empty.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| life (speed) | 1–4 steps/frame | 2 | how fast the world lives; 1 is slow-motion, 4 is frantic | BeatStep Pro knob 1, quantised to bar |
| growth (mu offset) | −0.01 … +0.01 | 0 | negative: bodies swell and merge; positive: bodies thin out and die | knob 2, plus 16-bar LFO ±0.004 |
| tolerance (sigma offset) | −0.003 … +0.003 | 0 | wider: blobby, aggressive growth; narrower: fragile, sharp creatures | knob 3 (small moves only) |
| seed | 0–1 | 0.8 | strength of kick soup seeds; 0 = no births | kick trigger × knob 4; BeatStep pads 1–8 = stamp species |
| size (R) | 8–15 cells | 13 | creature size on screen and GPU cost; rebakes the kernel and resets the world | KeyStep knob, changed only on breaks |
| glow | 0–1 | 0.5 | front/tail tint, rim and bloom together | KeyStep mod strip; RMS envelope |

The mu and sigma offsets are small because Lenia species live in narrow regions of parameter space. Outside them creatures die or fill the screen ([arXiv 1812.05433](https://arxiv.org/abs/1812.05433)). Clamp the MIDI ranges hard.

## Build recipes

### Resolume Arena

- Arena has no Lenia or custom-rule CA effect. Use a custom ISF effect or an external sim over Spout.
- Route 1, ISF multipass effect (load it in Wire):
  - Persistent FLOAT buffers `s0`, `s1` (`WIDTH = $WIDTH/3`, `HEIGHT = $HEIGHT/3`, so 640×360 at 1080p) and `trail` ([ISF multipass](https://docs.isf.video/ref_multipass.html)). The PASSES array is: step s0→s1, step s1→s0 (2 steps per frame), trail, then the final shade pass at full size. Initialise on `FRAMEINDEX == 0` with 12 soup discs.
  - Bake the kernel in-shader instead of in P0. GLSL loops in ISF need constant bounds, so loop −15..15 and skip offsets with |n| > R. Computing Kc per tap is costly. With R fixed at 13, hard-code the bounds and compute the normaliser once per frame in a 1×1 persistent pass.
  - Inputs: `mu`, `sigma`, `dt`, `life` (copy-through passes, as in the Gray-Scott chapter design), `seed` (event), `seedPos` (point2D), `brushPos`, `brushOn`, `glow`, `paletteMix`, `reset`. Map `seed` to Arena's audio analysis (Low band) and `paletteMix` to a BPM-synced clip parameter.
  - Precision: Resolume does not document whether `FLOAT` is honoured. Test: seed and run Orbium parameters for 30 s. If everything dies at once or never moves, the buffers are 8-bit, so use Route 2.
- Route 2 (recommended for 900+ taps per cell): run the sim in UE or Godot and bring it in as a Spout source on layer 2. Arena then composites only: layer 1 a clip, layer 2 Lenia (Screen, 70 %), or layer 2 as a mask for layer 1. Composition effects: Levels-style black crush, then Arena's Bloom (threshold 0.65, intensity 0.8).

### Resolume Wire

- Node outline: Input (food clip) → Blur (2–3 %) → Resize to sim size → ISF node (Lenia multipass, Route 1) → Output. If you use the Spout route, swap the ISF node for a Spout receiver and keep only the shading in an ISF single-pass node (P3), so palette and glow stay in Wire.
- Control: a beat/BPM node → trigger → envelope (instant attack, 120 ms release) → `dt` lurch and `seed`. A beat counter → hash in the shader → `seedPos`. An LFO at 1 cycle per 16 bars, sine, ±0.004 → added to `mu`. An audio low band → envelope (attack 80 ms, release 800 ms) → small `mu` offset.
- Exposed: life, growth, tolerance, seed, glow, paletteMix, reset. Node names for envelope and LFO vary between Wire versions, so check the node browser.

### Unreal Engine 5.7

- Route A (recommended): Blueprint ping-pong with render targets.
  - `RT_S0`, `RT_S1`: Texture Render Target 2D, 640×360, RTF RGBA16f, address Wrap, filter Nearest, no mips. `RT_Trail` R16f. `RT_Kernel` 31×31 R32f. `RT_Out` 1920×1080 RGBA8.
  - `M_LeniaKernel` (Unlit, Emissive): writes Ks per texel. The normaliser is computed in Blueprint (a loop over (2R+1)² in a function, run on change) and passed as a scalar.
  - `M_LeniaStep`: one Custom node (HLSL) with the double loop over `RT_Kernel` and `State` via Texture2DSample at texel centres. Then the growth exp, food Lerp, inject disc (Distance to `SeedPos` → noise from a hash of the cell plus `FrameSeed` → max into A, gated by `IsFirstStep`), and Clamp. Output (A, G, U, 0) to Emissive.
  - Blueprint Tick: a For Loop over `Steps`. Each iteration does Draw Material to Render Target into the other RT, then swaps. Then trail, then `M_LeniaShade` into `RT_Out`.
- Route B, Niagara: Grid2D Collection 640×360 with attributes A, G. Simulation stage "Step" with Num Iterations = steps. A custom HLSL neighbour loop reads the previous A, using a second attribute copy so you never read and write the same attribute in one stage. Use it only if you want particles in the same system: a CPU-light GPU emitter samples `RT_S0` and spawns sparks where G > 0.8 and A > 0.3 (the growing fronts). Sprite size 2–4 px, lifetime 0.6 s, additive, colour #FF9F43.
- Output: a Spout sender (via a Spout plugin) or NDI from `RT_Out` into Arena. Alternatively draw `M_LeniaShade` as a post-process material in a full-screen camera.
- Remote Control: Material Parameter Collection `MPC_Lenia` (`Mu`, `Sigma`, `Dt`, `Food`, `Seed`, `SeedPosX/Y`, `BrushX/Y`, `BrushOn`, `Glow`, `PaletteMix`) plus Blueprint variables `Steps` (int) and `R` (int), and functions `Reset`, `Reseed`, `Stamp(index)`. Bind OSC from Arena/Wire, BeatStep Pro knobs 1–4 and pads 1–8 (stamps), KeyStep strip → Glow.

### Godot

- Godot 4.x, Forward+ (compute needs a RenderingDevice renderer, [Godot docs, compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)).
- Compute route (recommended): two RGBA16F storage images 640×360, one R32F kernel buffer (storage buffer of (2R+1)² floats, filled from GDScript on change). Workgroup 16×16. Each group loads a (16 + 2R)² tile (42×42 for R = 13) into shared memory once, barriers, then each thread runs its 729-tap loop from shared memory. This is the single biggest speed-up for Lenia on the GPU. Dispatch `steps` times per frame, alternating bindings. Wrap the result with `Texture2DRD` ([Godot docs, Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html)) on a full-screen ColorRect whose `canvas_item` shader does P3.
- Fallback without compute: SubViewport ping-pong at 320×180 with `use_hdr_2d = true`, 1 step per frame, R = 8 (scale mu-tuned species accordingly and expect slower life).
- Uniforms: `mu`, `sigma`, `dt`, `food`, `seed_pos`, `seed_on`, `brush_pos`, `brush_on`, `stamp_index`, `glow`, `palette_mix`, `trail_decay`, `rim_gain`; `steps` and `R` belong to the script. Audio from AudioEffectSpectrumAnalyzer on the music bus.

## Audio reactivity

| band | drives | smoothing | threshold / scaling |
|------|--------|-----------|---------------------|
| kick, 40–120 Hz | soup seed + dt lurch 0.1 → 0.16 | attack instant, release 120 ms; one seed per rising edge | trigger at 0.6 of the running peak (2 s window), retrigger blocked 200 ms |
| bass, 60–250 Hz | mu offset 0 → −0.004 (bodies swell) | attack 150 ms, release 1.2 s | linear above a 0.25 floor |
| snare / clap, 1–4 kHz | mirrored seed | attack 5 ms, release 150 ms | trigger at 0.5 |
| hats, 6–12 kHz | brush dab radius 0.3·R, random position | attack 5 ms, release 60 ms | max 2 per beat |
| RMS loudness | glow 0.3 → 1, bloom intensity | attack 300 ms, release 3 s | — |

- Never drive sigma from audio. It is the most sensitive gene and fast changes wipe out the population. Only mu moves with audio, slowly and in small steps.

## Performance tips

- The cost is the convolution: about 530 non-zero taps per cell at R = 13. At 640×360 that is about 122 M taps per step, so 2 steps per frame is about 245 M fetches. That is fine on a current desktop GPU, but it is about 50× a Gray-Scott step. Budget it before the rest of the show.
- Levers in order: (1) shared-memory tiling (compute only), (2) a lower sim resolution: 640×360 upscaled bilinearly looks clean because the field is soft, (3) a smaller R (8–10). Cost scales with R², but creatures get more pixelated inside and some species no longer work at small R, (4) FFT convolution, which makes the cost independent of R. Chan's Python version uses it ([GitHub](https://github.com/Chakazul/Lenia)), but it is not trivial to build in UE materials or ISF.
- Creature size on screen is about 2·R cells. At 640×360 with R = 13 an Orbium is about 7 % of the frame height. For a big wall where you want a few large creatures, use a lower sim resolution (480×270) rather than a bigger R.
- Odd aspect ratios: keep cells square. For a 5760×1080 strip use 1280×240. Wrap addressing makes the left/right seam invisible and lets gliders circle a 360° room.
- Precision: RGBA16F minimum. RGBA8 kills the sim. The watchdog readback must not run every frame.
- Pitfalls: changing R mid-set (the world resets or every creature dies; do it only on breaks), dt raised for speed, mismatched step counts between frames, forgetting to normalise the kernel (mu then means nothing), and too much food (the wall fills).

## Further references

- [Chakazul/Lenia on GitHub](https://github.com/Chakazul/Lenia) — Chan's MIT-licensed code (Python with GPU/FFT, JavaScript, MATLAB) with the species catalogue.
- ["From Conway to Lenia" tutorial notebook](https://colab.research.google.com/github/OpenLenia/Lenia-Tutorial/blob/main/Tutorial_From_Conway_to_Lenia.ipynb) — step-by-step build from Game of Life to Lenia, linked from the [Lenia portal](https://chakazul.github.io/lenia.html).
- [CAX Lenia example](https://maxencefaldor.github.io/cax/examples/20_lenia/) — a compact implementation with the Orbium parameters used here (R 13, T 10, mu 0.15, sigma 0.015).
- [Bert Chan's shaders on Shadertoy](https://www.shadertoy.com/user/chakazul) — the author's GPU fragment-shader versions of Lenia, linked from his portal; the closest model for the ISF route.
