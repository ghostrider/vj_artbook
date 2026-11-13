# Gray-Scott Reaction-Diffusion — design

## Look

A living, wet surface fills the wall. Soft-edged blue-green blobs sit on near-black ink. They grow, push out fingers, split like cells and close into labyrinths and coral. The model is Gray-Scott with Karl Sims' parameters ([Karl Sims](https://www.karlsims.com/rd.html)). The field is lit like a gel: each blob has a rounded, shiny top with a small warm-white highlight. Fronts that are still growing glow orange at the edge, so the eye always finds where things are happening. Each kick drops a fresh droplet of "B" somewhere on the wall, and over a few beats it opens into a ring and then breaks into spots. On the drop the whole wall is reseeded and the parameters jump from mitosis (splitting spots) to coral (branching growth). On a break the kill rate rises and the pattern slowly dissolves back to black. The colours borrow from bromothymol blue, the pH indicator Pearson matched his pattern colours to (research): deep indigo, teal and an acid yellow at the peaks. The mood is microscopic, biological and slow-breathing, never static.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #03050C | Normal | 100 % | where B ≈ 0; on LED walls this is fully off after the crush |
| 2 | Style map (hidden) | video, camera or noise; luma 0..1 picks local f/k between two presets | — | — | optional; same input as the effect's source in Arena |
| 3 | Sim state (hidden) | two float textures A/B, ping-pong, N steps per frame | — | — | the only persistent state; see Shaders P0–P1 |
| 4 | Inject (inside sim) | soft discs of B added on kicks, brush and sprinkle | inside P1 (first step of the frame) | — | drives "never static" |
| 5 | Shaded RD | B → contrast window → ramp, plus normal-mapped gel lighting and growth glow | Normal | 100 % | main visible layer (P2) |
| 6 | Video under (optional) | any clip, revealed by the RD field | RD as luma mask on the clip, or RD in Multiply over the clip | 0–100 % | variation: RD as mask / matte |
| 7 | Post | black crush, bloom, grain | see compositing | — | light; the field is already soft |

## Techniques

**1. Gray-Scott update (the core)**
1. Each cell stores two amounts: A (food, starts at 1) and B (catalyst, starts at 0, seeded at 1 in small areas) ([Karl Sims](https://www.karlsims.com/rd.html)).
2. Laplacian from a 3×3 kernel: centre −1, the 4 edge neighbours 0.2, the 4 corners 0.05 ([Karl Sims](https://www.karlsims.com/rd.html)). `lapA = Σ w·A(neighbour) − A`, the same for B.
3. Reaction term `r = A·B·B`.
4. `A' = A + (D_A·lapA − r + f·(1 − A))·dt` and `B' = B + (D_B·lapB + r − (k + f)·B)·dt`, with D_A = 1.0, D_B = 0.5, dt = 1.0 ([Karl Sims](https://www.karlsims.com/rd.html)).
5. Clamp both to 0..1 after the step. This is a safety net against float drift and the brush overshooting.
6. Why it works: convex B edges get A from outside and grow, concave edges starve. That is why spots swell, split and settle (research).

**2. Stability and speed: more steps, not a bigger dt**
1. With this kernel, D_A = 1 and dt = 1 are already at the edge of what explicit Euler tolerates. Never raise dt or D to "speed up". The image explodes into checkerboard noise.
2. Speed = sim steps per frame. Default 12 steps per frame at 60 fps = 720 steps per second. A seeded droplet then becomes a visible ring within about one beat at 128 BPM, and a split spot within 2–4 beats.
3. Range on stage: 1 step (almost frozen, for breaks) to 32 steps (fast growth, for the drop). Below about 4 steps the pattern looks static.
4. Steps must be an integer and the same every frame. Tie them to the frame, not to TIMEDELTA, and lock the output frame rate.

**3. Pattern size: reaction scale**
1. Sims names "scale" as a variant: slowing the reaction relative to diffusion makes the features larger ([Karl Sims](https://www.karlsims.com/rd.html)).
2. Implement it as `s` (0.3–1.0) multiplied onto the reaction and feed/kill terms only: `A' = A + (D_A·lapA + s·(−r + f(1 − A)))`, and the same for B. Diffusion is left alone, so stability is untouched.
3. Lower `s` gives bigger, slower blobs. It is the "zoom" fader. The other zoom is the sim resolution (half resolution doubles the feature size on screen).

**4. Style map: f and k per pixel**
1. Sims notes that f and k can vary across the grid ([Karl Sims](https://www.karlsims.com/rd.html)). piellardj interpolates f/k between a "white" preset and a "black" preset using the image luminance `0.21R + 0.72G + 0.07B` ([piellardj](https://github.com/piellardj/reaction-diffusion-webgl)).
2. `l = luma(styleMap(uv))`, `f = mix(fDark, fBright, l·mapAmt + (1 − mapAmt)·0.5)`, and the same for k.
3. Presets (from Sims): mitosis f 0.0367 / k 0.0649, coral f 0.0545 / k 0.062, default f 0.055 / k 0.062. The interesting window is roughly f 0.01–0.1, k 0.045–0.07 ([Karl Sims](https://www.karlsims.com/rd.html)). Use mitosis for dark areas and coral for bright ones, so a performer's silhouette grows coral and the background splits into cells.
4. Over minutes the field traces the video shapes. It does not follow fast motion, so use slow footage or a heavily blurred camera.

**5. Injection (anti-stagnation)**
1. The research names a settled, static pattern as the main failure. Something must always be adding B or moving f/k.
2. Kick seed: add a Gaussian disc, `B += inject · exp(−d²/r²)` and `A −= 0.5 · inject · exp(−d²/r²)`. Radius r = 1.5 % of the sim height, centre from a hash of the beat counter. This follows the C10ud beat-seeded approach cited in the research.
3. Sprinkle: on each bar, 4–8 small seeds (r = 0.5 %) at random positions.
4. Brush: an XY position (KeyStep or OSC) paints B continuously while a gate is held.
5. Watchdog (UE/Godot only): a 1×1 mip or reduction of mean B. If mean B < 0.002 (dead) for 2 s, auto-reseed 20 spots. If it is > 0.35 (saturated mush), raise k by 0.003 for 4 bars.

**6. Flow (optional motion on top of growth)**
1. Sims lists flow, where chemicals move across the grid, as a variant ([Karl Sims](https://www.karlsims.com/rd.html)).
2. Per step, sample the state at `uv − flow·v(uv)/simSize` instead of `uv`, where `v` is a slow curl-noise or fbm vector field (scale 2–3 across the frame, drift 0.02/s). This is semi-Lagrangian advection, with the Laplacian taken around the advected point.
3. `flow` from 0 to 0.6 cells per step. Above about 0.8 cells per step the pattern smears and loses its shape.

**7. Gel shading**
1. `v = clamp((B − lo) / (hi − lo), 0, 1)`, with lo 0.08 and hi 0.32 by default. B rarely reaches 1 in a settled pattern, so without this window the image is dim and flat.
2. Normal from central differences of B: `n = normalize(−dBdx·h, −dBdy·h, 1)`, height h = 12 (range 4–30).
3. Light direction normalize(−0.5, 0.6, 0.6). `diffuse = max(dot(n, L), 0)`, Blinn specular with power 48, specular strength 0.6.
4. Growth glow: the last sim step of each frame also writes `g = max(B' − B, 0) · 400` into a third channel. Fronts that are still growing light up in the accent colour, and the growth itself becomes the visible motion.

## Shaders

Render order per frame: P0 (only on reset) → P1 × N steps (ping-pong) → P2 shade → P3 post. The sim runs at half output resolution by default.

**P0 — Init / reset**
- Purpose: start state.
- Inputs: `seedMode` (spots, ring, noise, style map), `time`.
- Per pixel: A = 1, B = 0 ([Karl Sims](https://www.karlsims.com/rd.html)). Then add B = 1 inside 20 random discs (r = 1 % height), or inside a centred ring (radius 0.3, width 0.01), or wherever noise > 0.75.
- Output: state texture A.
- When it runs: on load (frame 0), on a reset trigger, and on a drop reseed. On a drop, reseed by adding to the existing state rather than clearing, so the wall never goes black.

**P1 — Sim step (runs N times per frame)**
- Purpose: one Gray-Scott step.
- Inputs: previous state (texture, RG = A,B, B channel = growth), style map, uniforms `feed`, `kill`, `fDark/kDark/fBright/kBright`, `mapAmt`, `scale`, `flow`, `inject`, `seedPos`, `seedRadius`, `brushPos`, `brushOn`, `stepIndex`, `texel = 1/simSize`.
- Per pixel: (1) if flow > 0, offset the sample centre by the flow vector. (2) Fetch the centre and 8 neighbours at texel offsets with wrap (repeat) addressing, so edges do not pin the pattern. (3) Compute the Laplacians, then `r = A·B²`, then local f and k from the style map. (4) Apply the update with scale `s` and clamp. (5) Only when `stepIndex == 0`: add the inject disc and the brush. (6) Only when `stepIndex == N − 1`: write growth `max(B' − Bprev, 0)` into channel 3. Otherwise pass it through.
- Output: the other state texture (ping-pong).
- Format: RG16F at minimum, RGBA16F or RGBA32F preferred. 8-bit is not enough. piellardj had to pack each 16-bit value into two 8-bit channels on WebGL 1 for exactly this reason ([piellardj](https://github.com/piellardj/reaction-diffusion-webgl)). Use nearest or texel-centre sampling for the stencil.
- Resolution: 960×540 for a 1080p output (default), 1280×720 for finer detail, 480×270 for big chunky blobs.
- Ping-pong: textures S0 and S1. Step i reads S(i mod 2) and writes S((i + 1) mod 2). Use an even N so the frame always ends in S0. Never read and write the same texture in one draw.

**P2 — Shade**
- Purpose: turn the scalar field into the image.
- Inputs: final state (bilinear sampling, which upscales it to output size), uniforms `lo`, `hi`, `height`, `light`, `glow`, palette colours.
- Per pixel: v from the contrast window → 4-stop ramp → `c = ramp(v) · (0.35 + 0.65·diffuse) + spec · #FFF6D8 · 0.6 · v` → `c += glow · g · #FF5A36`.
- Output: RGBA8 (or RGBA16F before bloom) at output resolution.

**P3 — Post**
- Black crush `c = max(c − 0.02, 0) / 0.98`.
- Bloom: threshold 0.75, intensity 0.8, radius 1.5 % of the frame height. It catches the speculars and the growth glow.
- Grain 1.5 %, monochrome, refreshed each frame. It hides banding in the smooth gradients on 8-bit LED processors.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base / black | #03050C | v = 0, empty A-rich background |
| shadow | #0B2E4F | ramp at v = 0.35, blob flanks |
| body | #22A699 | ramp at v = 0.7, main blob colour (bromothymol green-teal) |
| highlight | #F2D64B | ramp at v = 1, blob crests (acid yellow) |
| specular | #FFF6D8 | gel highlight |
| accent | #FF5A36 | growth glow on active fronts |

- Mapping: a 4-stop gradient ramp indexed by the windowed B value, modulated by the diffuse light. Colour carries no simulation meaning, so any ramp works (research). Keep the lowest stop black so empty space stays dark.
- Alternate drop palette: rotate the ramp hue by +150° (teal turns into magenta, yellow into blue-violet) and switch the accent to #3DF5FF for one phrase.
- RGB triple mode (variation): three sims, one per channel, with different f/k each, composited additively as piellardj's colour mode does ([piellardj](https://github.com/piellardj/reaction-diffusion-webgl)). This costs 3× the sim but gives overlapping colour cells without a ramp.
- Over time: on each phrase (8 bars), shift the contrast window lo/hi by ±0.02 so the blob thickness breathes. Hold the hue for a whole track.
- LED walls: blacks fully off after the crush. Peak white about 90 %. The coral preset can cover 50 %+ of the wall in the body colour, so keep body luminance moderate (#22A699 is about 50 %) and let only the crests and speculars go bright.

## Blending and compositing

- Shaded RD over the background in Normal. The field is a full opaque image.
- As a matte over video: use v (before lighting) as a luma mask on a video layer. Cells reveal the footage, and the edges are soft because B is smooth. In Arena, put the RD layer above the clip with blend mode Multiply, or route v as a mask (see the Arena recipe).
- As displacement: the RD gradient (dBdx, dBdy) × 0.02 displaces a video layer, which gives a refractive gel over the footage. The normal is already computed in P2, so expose it as a second output if the tool allows.
- There is no separate feedback mix. The simulation itself is the feedback loop, and its "decay" is the kill rate k.
- Post chain order: crush → bloom → grain.

## Motion and timing

- What moves: the growth itself (speed = steps per frame), the injected seeds, slow flow, and the f/k morph.
- Base: mitosis preset, 12 steps per frame, scale 0.8, flow 0.1.
- Beat: a kick seed on every beat (inject 0.6, attack instant, applied once). Every bar, a sprinkle of 4 seeds.
- Bar / phrase: every 4 bars, crossfade f/k from mitosis to coral or back over one bar (linear in f and k, not instant), because jumping f/k makes the whole field flash. Every 8 bars, shift the palette window.
- Build-up: steps rise from 8 to 24 over the last 4 bars, scale drops from 0.9 to 0.6 (blobs swell), flow rises to 0.4.
- Drop: on the downbeat, run a ring reseed added to the existing state, switch to coral over 2 beats, set steps to 24 and glow to 1. The ring bursts outward into branches within 1–2 bars.
- Break: kill rises to 0.068 over 2 bars, so the pattern starves and shrinks (spots die first), and steps drop to 4. Then hold, with only the brush active (Uli paints with the KeyStep XY). Return k to the preset 1 bar before the next drop.
- Never leave the field without injection for more than 8 bars.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| morph | 0–1 (0 = mitosis f 0.0367 k 0.0649, 1 = coral f 0.0545 k 0.062) | 0 | slides between splitting cells and branching coral | BeatStep Pro knob 1; flip every 4 bars by sequencer |
| kill offset | −0.006 to +0.008 added to k | 0 | starve (+) or overgrow (−); + dissolves the pattern for breaks | knob 2; macro ramp on breaks |
| speed | 1–32 steps per frame (integer, even) | 12 | how fast the field grows | knob 3; pad for "drop" = 24 |
| scale | 0.3–1.0 | 0.8 | pattern size (low = big slow blobs) | knob 4; slow LFO 1 cycle per 16 bars |
| inject | 0–1 | 0.6 | strength of the beat seeds | kick envelope; pad = manual seed |
| flow | 0–0.6 cells per step | 0.1 | drifts the pattern like liquid | KeyStep mod strip; bass envelope |

Extra performance controls, not faders: reset (pad), ring reseed (pad), brush XY + gate (KeyStep), and glow 0–1 (layer-level).

## Build recipes

### Resolume Arena

- Arena has no built-in reaction-diffusion effect, and its Feedback-type effects cannot run a custom update rule. You need either a custom ISF effect or an external sim sent in by Spout.
- Route 1, ISF multipass effect (also loadable in Wire):
  - The ISF spec lets each pass set `TARGET`, `PERSISTENT` (kept across frames) and `FLOAT` (32-bit per channel), with `WIDTH`/`HEIGHT` given as expressions of `$WIDTH`/`$HEIGHT` ([ISF JSON reference](https://docs.isf.video/ref_json.html), [ISF multipass](https://docs.isf.video/ref_multipass.html)).
  - Two persistent float buffers, `simA` and `simB`, with `WIDTH = $WIDTH/2` and `HEIGHT = $HEIGHT/2`. The PASSES array alternates: pass 0 reads simA and writes simB, pass 1 reads simB and writes simA, and so on for N passes (N even, default 12). The last pass is the shade pass P2 with no target, at full size. The shader branches on `PASSINDEX`, and on `FRAMEINDEX == 0` it runs the init ([ISF variables](https://docs.isf.video/ref_variables.html)).
  - Steps per frame are fixed by the number of passes in the JSON. For a live speed control, write 32 passes and let a `speed` input make passes with index ≥ speed copy their input unchanged (keep the copy count even). Unused passes still cost a full-buffer copy each.
  - Inputs: `inputImage` (style map), floats `morph`, `killOffset`, `scale`, `inject`, `flow`, `glow`, `point2D` brush, `bool` reset. Arena can drive `inject` from its audio analysis (Low band) and `morph` from a BPM-synced LFO.
  - Persistent buffers are resized when the output resolution changes ([ISF JSON reference](https://docs.isf.video/ref_json.html)). Expect a reset when you change the composition size.
- Precision caveat: Resolume documents ISF only inside Wire, and multipass / persistent-buffer support depends on the version. Whether the host honours `FLOAT` is not documented by Resolume. Test before the gig: run the mitosis preset for 30 s. If spots freeze, die instantly or show stair-stepped edges, the buffer is 8-bit. Fallback inside ISF: pack A into R+G and B into B+A as 16-bit fixed point (high byte `floor(v·255)/255`, low byte the remainder), as piellardj does on WebGL 1 ([piellardj](https://github.com/piellardj/reaction-diffusion-webgl)). Sample packed buffers only at texel centres.
- Route 2, external sim (most reliable): run the sim in UE or Godot (below) and bring it into Arena as a Spout source on layer 2. Arena then only does compositing: the RD layer in Multiply over a clip on layer 1, or used as a mask.
- Layer setup: layer 1 is a video clip (optional), layer 2 the RD effect or Spout source. Bloom is Arena's own Bloom effect on the composition (threshold and intensity as in P3).

### Resolume Wire

- State lives inside the ISF node: Wire's ISF node hosts the same multipass JSON from Route 1, and the persistent buffers are owned by the node. The same version and FLOAT caveats apply, so do the 30 s mitosis test in Wire first.
- Node outline: Effect input texture (style map) → Resize to half output size → ISF node (RD multipass: init, N alternating steps, shade) → Output. In parallel, a Blur on the input (radius 2–3 %) before the ISF style-map input, so the video edges do not imprint pixel noise.
- Control side: BPM / beat node → trigger → envelope (instant attack, 100 ms release) → `inject`. A counter of beats → hash → seed position (or pass the beat count and hash inside the shader). An LFO at 1 cycle per 8 bars, square-shaped and smoothed → `morph`. An audio band (low-mid) → `flow`.
- Exposed parameters: morph, kill offset, speed, scale, inject, flow, glow, reset. Wire has no vertex shaders, which this design does not need. Exact names of the envelope, counter and LFO nodes differ between Wire versions, so check the node browser.

### Unreal Engine 5.7

- Route A (deterministic, recommended): Blueprint ping-pong with render targets.
  - `RT_S0`, `RT_S1`: Texture Render Target 2D, 960×540, format RTF RGBA16f (or RGBA32f), mips off, address Wrap, filter Nearest. `RT_Out`: 1920×1080 RGBA8 for output.
  - `M_RDStep`: material domain Surface, Unlit, output to Emissive. Texture Object parameter `State`. TexCoord plus offsets of `1/960, 1/540` feed 9 Texture Sample nodes (or one Custom node with the 9-tap loop). Then the Laplacian, reaction, f/k from a Lerp on the style-map luma, scale, inject disc (Distance to `SeedPos` → Exp of the negative squared distance over r² → multiply by `Inject` × `IsFirstStep`), growth into B when `IsLastStep`, and Clamp.
  - Blueprint on Tick: a For Loop from 0 to Steps − 1. Set the MID's `State` to the current read RT and the flags `IsFirstStep`/`IsLastStep`, call Draw Material to Render Target into the other RT, then swap the references. Draws are queued in order on the render thread, so N steps in one tick are fine. Init: draw `M_RDInit` into `RT_S0` on BeginPlay and on Reset.
  - `M_RDShade` reads the final RT: window, ramp (3 Lerps or a Curve Atlas Row), normal from 4 offset samples, Blinn specular, growth glow. Draw it into `RT_Out`.
- Route B, Niagara: a Grid2D Collection with attributes A and B, 960×540. Simulation stage "Init" runs on reset; stage "Step" iterates over the grid with Num Iterations = steps per frame. Epic's fluid docs describe Grid2D Collections as the grid storage and Num Iterations as the way to repeat a stage before moving on ([Epic, fluid simulation overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/fluid-simulation-in-unreal-engine---overview)). Write the grid to a Render Target 2D data interface in a final stage for shading. Check in your version whether Num Iterations can be bound to a user parameter for live speed changes. If not, use Route A.
- Watchdog: generate mips on a small copy (128×72) of the state, read the 1×1 mip with Read Render Target Pixel every 30 frames (not every frame, because it stalls), and auto-reseed when mean B < 0.002.
- Output: Spout (via a Spout plugin) or NDI from `RT_Out` into Arena, or apply `M_RDShade` as a post-process material on a full-screen camera.
- Remote Control: a Material Parameter Collection `MPC_RD` with `Morph`, `KillOffset`, `Scale`, `Inject`, `Flow`, `Glow`, `SeedPosX/Y`, `BrushX/Y`, `BrushOn`. Add a Blueprint variable `Steps` (int) and the functions `Reset` and `RingReseed` to a Remote Control Preset, then bind OSC (from Arena / Wire) and MIDI (BeatStep Pro knobs 1–4, KeyStep strip).

### Godot

- Godot 4.x, Forward+ renderer (compute needs a RenderingDevice-based renderer, Forward+ or Mobile ([Godot docs, compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html))).
- Route A, compute (recommended): two RGBA16F (or RGBA32F) storage images, 960×540. A GLSL compute shader does one step per dispatch with 8×8 workgroups (120×68 groups). Every frame the script dispatches Steps times, alternating the image bindings, from the render thread on the main RenderingDevice. Wrap the final image in a `Texture2DRD` through its `texture_rd_rid` property ([Godot docs, Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html)) and show it on a full-screen ColorRect with a `canvas_item` shader that does P2. Do not call `sync()` every frame ([Godot docs, compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)).
- Route B, no compute (Compatibility renderer fallback, 1 step per frame): two SubViewports `S0`, `S1`, each 480×270 with `use_hdr_2d = true` (RGBA16 framebuffer ([Godot docs, Viewport](https://docs.godotengine.org/en/stable/classes/class_viewport.html))), each holding a ColorRect with the step shader that samples the other viewport's ViewportTexture. A script alternates `render_target_update_mode = UPDATE_ONCE` between S0 and S1 each frame ([Godot docs, SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)), so exactly one step runs per frame in a defined order. This gives 60 steps per second, so use the lower resolution (bigger features grow visibly faster) and scale 0.5–0.7.
- Uniforms to expose: `feed_dark/kill_dark/feed_bright/kill_bright`, `morph`, `kill_offset`, `scale`, `inject`, `seed_pos`, `flow`, `brush_pos`, `brush_on`, `glow`, `lo`, `hi`, `height`; `steps` and `step_index` are set by the script. Audio comes from an AudioEffectSpectrumAnalyzer on the music bus. Output to Arena via a Spout GDExtension if available, or screen capture through NDI.

## Audio reactivity

| band | drives | smoothing | threshold / scaling |
|------|--------|-----------|---------------------|
| kick, 40–120 Hz | inject seed (one disc per hit) | attack instant, release 100 ms; the seed is applied once on the rising edge | trigger at 0.6 of the running peak (2 s window), retrigger blocked 150 ms |
| bass, 60–250 Hz | flow 0 → 0.4 | attack 80 ms, release 600 ms | linear above a 0.2 floor |
| snare / clap, 1–4 kHz | seed radius ×1.5 for that hit, or a second seed mirrored across the centre | attack 5 ms, release 150 ms | trigger at 0.5 |
| hats, 6–12 kHz | tiny sprinkle (r = 0.3 %, 1–2 seeds) | attack 5 ms, release 60 ms | trigger at 0.5, max 4 per beat |
| RMS loudness | speed 8 → 24 steps, glow 0.3 → 1 | attack 300 ms, release 3 s | quantise speed to even integers |

- Do not drive f or k directly from fast audio. Every small change re-shapes the whole wall at once and reads as flicker. Only morph and kill offset move, slowly, on bar boundaries.

## Performance tips

- Sim cost per step: 9 texture fetches per cell. At 960×540 that is 4.7 M fetches per step, and 12 steps per frame is about 56 M fetches, well under 1 ms on a current desktop GPU. Shading costs about 5 fetches per output pixel.
- VRAM is tiny: two 960×540 RGBA16F targets take 8.3 MB together.
- The ISF route is heavier than it looks: each step is a full pass with host overhead, and unused "copy" passes for live speed still cost. Stay at 12–16 passes and half resolution.
- Resolution choice: the feature size is fixed in cells (about 8–14 cells per spot at the default presets), so the sim resolution sets the on-screen size. Half output resolution is the sweet spot, since bilinear upscaling of a smooth field looks clean. Full resolution makes very fine detail that vanishes at audience distance on LED walls with large pixel pitch.
- Odd aspect ratios: keep sim cells square. For a 5760×1080 strip use a 1440×270 sim (16:3). Never stretch a 16:9 sim onto it, because the spots turn into ellipses. Use wrap addressing so patterns flow across the left/right seam (useful for 360° wraps).
- Frame rate: speed is counted in steps per frame. If the output drops from 60 to 30 fps, the growth halves. Lock the fps.
- Pitfalls: 8-bit buffers (pattern dies or bands), raising dt or D for speed (checkerboard blow-up), sampling the state with mipmaps on, odd step counts in a ping-pong without tracking which buffer is current (the image flickers between two states), and an empty field after a long break (always keep a sprinkle or a watchdog).

## Further references

- [Karl Sims, Reaction-Diffusion Tutorial](https://www.karlsims.com/rd.html) — the parameters, kernel, presets and the extensions (orientation, style map, flow, scale) used in this design.
- [piellardj, reaction-diffusion-webgl](https://github.com/piellardj/reaction-diffusion-webgl) — a real-time GPU version with image-driven f/k, the RGB three-sim mode and the 8-bit packing trick; it has a live demo link.
- [ISF Multi-Pass and Persistent Buffer Reference](https://docs.isf.video/ref_multipass.html) — how PERSISTENT and FLOAT buffers work in ISF, the basis of the Arena/Wire route.
- [Epic, Fluid Simulation in Unreal Engine overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/fluid-simulation-in-unreal-engine---overview) — Grid2D Collections and simulation stages with Num Iterations, the Niagara route.
