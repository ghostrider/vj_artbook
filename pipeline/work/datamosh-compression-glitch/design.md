# Datamosh Compression Glitch — design

## Look

A clean clip plays, the kick hits, the scene cuts, but the old picture stays. A dancer's face from the last shot is dragged across the frame by the movement of the new one: whole 16-pixel squares of skin and jacket slide sideways, stretch into stripes at the edge of the frame and pile up where the new motion stops. Where the new clip moves fast, colour pulls away from shapes and runs ahead of them. Where nothing moves, the old image hangs on like a ghost. This is the look of Murata's *Monster Movie*, where the source keeps falling apart and rebuilding itself ([EAI](https://www.eai.org/titles/monster-movie/video-intro)), and of the 2009 music videos that took it mainstream ([Motionographer](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/)). On a held note one movement repeats and grows: a head turn becomes a blooming smear that swallows the frame (P-frame repeat). On the drop the whole thing snaps back to a sharp clean picture, a fresh "keyframe", and the next mosh starts from there. The mood is digital decay: hard block edges right next to soft running colour, a few blocks of decode-error green, and blacks that stay black. Two routes give this look. **Route A** is real datamosh: clips moshed offline by deleting I-frames or repeating P-frames, baked and played as ordinary clips. **Route B** is a real-time emulation: motion vectors from the live source push the blocks of a held frame around, so any input can be moshed and the kick can trigger it.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #000000 | Normal | 100 % | only shows when wet is pulled down or blocks smear off-screen with edge mode "black" |
| 2 | Clean source A (dry) | live camera, NDI/Spout feed or DXV clip, high contrast, lots of motion | Normal | 100 % | the "keyframe" image; layer 4 snaps back to this |
| 3 | Route A: baked mosh clips | DXV clips rendered offline (I-frame removal transitions, P-frame repeat blooms) | Normal over layer 2 | 100 % while playing | triggered on the bar; when the clip ends, cut to the clean target clip on layer 2 |
| 4 | Route B: live mosh (wet) | held frame in a ping-pong buffer, pushed each frame by block motion vectors | Normal over layer 2 | 0–100 % (wet fader), default 100 % | main real-time look; Shaders P0–P4 |
| 5 | Motion source B (hidden) | second live source whose motion drives layer 4 (optional) | — (not visible) | — | variation 1 from the research: image of A, motion of B |
| 6 | Block glitch accents | random intra-refresh blocks and decode-error green blocks | inside layer 4 | refresh 0–0.15 | short bursts on hi-hats |
| 7 | Post | black crush, slight sharpen, no bloom by default | see compositing | — | blur and bloom kill the block edges, which are the whole point |

Use either layer 3 or layer 4 per song section. Both on at once turns into mush.

## Techniques

**1. Why moshing happens (codec model)**
1. A codec stores a full picture (I-frame), then P-frames that only hold motion vectors per block plus a small correction (residual) ([Spotlight FX](https://spotlightfx.com/blog/what-is-datamoshing)).
2. The decoder builds each frame as "previous frame, blocks moved by their vectors, plus residual".
3. Delete the I-frame at a cut and the decoder applies the new shot's vectors to the old shot's last picture: that is the transition mosh.
4. Repeat one P-frame many times and the same vectors are applied again and again: the motion compounds into a bloom.
5. Route B copies exactly this model in shaders: a stored picture (the "decoded frame"), a block motion field, an optional residual, and a keyframe trigger.

**2. Route A, offline mosh (real datamosh)**
1. Encode source clips without B-frames so every frame after the I-frame depends only on the previous one. The open-source [datamoshing](https://github.com/tiberiuiancu/datamoshing) scripts do this with ffmpeg into AVI with `-bf 0`, a 10000k bitrate and lossless quality settings ([mosh.py](https://raw.githubusercontent.com/tiberiuiancu/datamoshing/master/mosh.py)).
2. Put clip A and clip B back to back in one encode. Remove the I-frames from the cut onwards (`mosh.py` with start and end frame): B's motion now runs on A's last picture.
3. For blooms, use the P-frame repeat mode (`-d n` repeats a block of n P-frames over the following frames). n = 1–4 gives a steady push in one direction; n = 15–30 gives a rhythmic looping smear.
4. For finer control, edit the motion vectors themselves with FFglitch ([ffglitch.org](https://ffglitch.org/)). Its motion-vector averaging script replaces each frame's vectors with the mean of the last N frames, which turns jitter into smooth liquid pushes ([FFglitch tutorial](https://ffglitch.org/2024/06/mv_avg_010.html)). The same repo wraps FFglitch for motion transfer, adding one clip's vectors to another ([datamoshing](https://github.com/tiberiuiancu/datamoshing)). Manual P-frame copy and paste also works in Avidemux 2.7.0+ ([Spotlight FX](https://spotlightfx.com/blog/what-is-datamoshing)).
5. Bake the result: decode the broken file once (the script re-encodes to H.264 at CRF 18) and convert it to DXV in Alley with "DXV Normal Quality, No Alpha" ([Resolume: Alley](https://resolume.com/article/81)). Never give Arena the broken AVI. The mosh only exists in the decoder, and every decode can turn out different. The Kanye video team needed about 45–50 renders for that reason ([Motionographer](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/)).
6. Render many versions per transition and keep the best 5–10. Trim each to a whole number of bars (4 bars at 128 BPM = 7.5 s).

**3. Route B, block motion estimation (codec-like vectors)**
1. Work on luma at quarter resolution (480×270 for a 1080p source), current frame `L` and previous frame `Lp`.
2. One motion vector per 16×16 full-res macroblock = 4×4 quarter-res pixels. The motion texture is 120×68 for 1080p.
3. For each block, test candidate offsets `o` in a search window of ±6 quarter-res pixels (±24 full-res px per frame, about 1440 px/s at 60 fps). Cost = sum over the 16 pixels of `|L(p) - Lp(p - o)|` (SAD).
4. Choose the lowest cost, but **prefer zero**: keep `o = 0` unless `cost(o) < 0.9 · cost(0)` and `cost(0) > 0.02 · 16`. This copies what encoders do in flat areas. Without it, sky and black backgrounds fill with random vectors and turn into noise.
5. Store `mv = o · 4` in full-res pixels per frame. This means "content at `p - mv` in the previous frame is now at `p`."
6. Cheaper alternative: coarse-to-fine search (step 2 over ±6, then ±1 around the best), 49 + 9 = 58 candidates instead of 169.

**4. Route B alternative: gradient optical flow (liquid, not blocky)**
1. Per quarter-res pixel: `It = L - Lp`, spatial gradient `g = (dL/dx, dL/dy)` from central differences.
2. `flow = -It · g / (|g|² + λ)`, λ = 0.01–0.05 (luma 0..1). This is a one-step, Lucas-Kanade-style estimate. It is cheap and smooth but only good for small motions.
3. Blur the flow with a 5×5 box. Clamp its length to 24 px.
4. Andrew Benson's 2009 Jitter/GLSL patch used optical-flow distortion like this for a liquid datamosh look ([CDM](https://cdm.link/2009/03/liquidify-video-live-optical-flow-glsl-datamosh-technique/)), and Buck added block structure on top of a flow-driven feedback loop to make it angular ([Buck](https://buck.co/sandbox/moshing)). The SIGGRAPH Asia 2023 poster by Careaga, Reddy and Aksoy shows that accurate optical flow applied to mismatched content gives moshing without codec blocks ([Datamoshing with Optical Flow](https://yaksoy.github.io/datamosh)).
5. With external flow (TouchDesigner Optical Flow TOP, Nvidia RTX 3000 or newer, Windows only), R and G hold motion in screen widths per second, and G is scaled the same way, so 1.0 in G is about 1.77 screen heights in 16:9 ([Derivative](https://derivative.ca/UserGuide/Optical_Flow_TOP)). Convert with `mv_px = (R, -G) · width / fps`. TouchDesigner's y axis points up, so G is negated.

**5. Route B, mosh accumulation (the held frame)**
1. Keep a persistent full-res buffer `M` (the "decoded picture"). Each frame: `M_new(p) = M_old(p - mvBlock(p))`.
2. `mvBlock(p)` is the vector of the block that contains `p`. Sample the motion texture at the block centre with nearest filtering, so all pixels of a block move together. That gives the hard macroblock edges. A larger `block` (32 or 64 px) samples the vector at the centre of the larger cell.
3. Sample `M_old` with texel-snapped coordinates (`floor(p - mv) + 0.5`) for crisp blocks. Bilinear sampling smooths the edges but blurs a little more every frame. Expose it as `melt` 0–1 (mix of the two samples).
4. Chroma slip: take luma from `M_old(p - mv)` and chroma from `M_old(p - mv · chroma)`, chroma = 1.0–1.6. Colour runs ahead of shapes as in real moshes.
5. Residual (optional, 0–0.3): `M_new += (cur - prev) · residual`. Small values let the new clip "draw in" its edges as the mosh goes on.
6. Intra refresh: per block `hash(blockId, frame) < refresh` → that block takes the current source. A low rate (0.002) slowly cleans the picture, like codec intra blocks. Bursts (0.05–0.15 for 2 frames) give the "blocks popping in" look.
7. Keyframe event: `M_new = keySource` for one frame. That is the I-frame: a clean snap and the start of a new mosh.
8. Edge mode: clamp (blocks pulled from the edge become stripes, the classic look) or black.

**6. Hold = P-frame repeat (bloom)**
1. When `hold` turns on, copy the current motion texture into `MV_held` and stop updating it.
2. While held, apply `MV_held · gain` every frame. The same push repeats and compounds, like repeated P-frames.
3. A smoothing option, inspired by FFglitch's averaging: `MV_avg = mix(MV_avg, MV_new, 1 / tail)`, tail = 4–30 frames. Use `MV_avg` instead of the raw vectors for long liquid pushes ([FFglitch tutorial](https://ffglitch.org/2024/06/mv_avg_010.html)).
4. Releasing hold on the drop fires a keyframe, so the bloom snaps back to clean.

## Shaders

Route B only; Route A has no shaders at show time. Render order per frame: P0 → P1 → P2 → P3 → P4. Two ping-pong pairs: luma history (P0) and mosh buffer (P3); `MV_held` / `MV_avg` are persistent too.

**P0 — Luma downsample**
- Purpose: cheap input for motion estimation.
- Inputs: motion source (B, or A if no B), uniform none.
- Per pixel: 2×2 box average of the source down to quarter res, `Y = 0.2126 R + 0.7152 G + 0.0722 B`. Write to `L`; the previous `L` becomes `Lp` (swap).
- Output: 480×270, R8 or R16F (R16F avoids SAD banding in dark footage). 2 buffers.

**P1 — Motion estimation**
- Purpose: one vector per macroblock.
- Inputs: `L`, `Lp`; uniforms `search` (6), `zeroBias` (0.9), `flatThresh` (0.02).
- Per pixel (each pixel = one 16×16 block): block-matching search (Technique 3) or gradient flow (Technique 4, then sample at block centre). Output `mv` in full-res px/frame in RG, best SAD in B (confidence; useful for masks).
- Output: 120×68 RG16F (block-match) or 480×270 RG16F (gradient flow). With TouchDesigner, P0 and P1 are replaced by the Optical Flow TOP and the conversion in Technique 4.5.

**P2 — Motion shaping**
- Purpose: gain, hold, smoothing, audio push, jitter.
- Inputs: P1, `MV_held`, `MV_avg`; uniforms `gain` (0–4), `hold` (bool), `tail`, `push` (vec2, px/frame), `jitter` (0–1), `time`.
- Per pixel: `v = hold ? MV_held : (tail > 1 ? MV_avg : mv)`, `v = v · gain + push`; jitter adds `±jitter · 8 px · hash(block, beat)` on a random 20 % of blocks. Update `MV_avg`; copy to `MV_held` on the rising edge of hold.
- Output: same res as P1, RG16F, persistent.

**P3 — Mosh update (ping-pong)**
- Purpose: move the held picture.
- Inputs: `M_old` (previous P3 output), current source A (key source), current and previous source (for residual), P2; uniforms `block` (16/32/64 px), `melt`, `chroma`, `residual`, `refresh`, `drip` (0–0.05), `keyframe` (bool, one frame), `edgeMode`, `frame`.
- Per pixel: (1) `b = floor(p / block)`, `v = P2` sampled at block centre, nearest; (2) two taps of `M_old` at `p - v` and `p - v · chroma`, snapped or bilinear by `melt`, recombined as luma + chroma; (3) `+ (cur - prev) · residual`; (4) intra refresh by block hash; (5) `mix(c, cur, drip)`; (6) if keyframe → `c = keySource`; (7) clamp 0..1.
- Output: full output res (1920×1080 or wall canvas), RGBA8, 2 buffers (8.3 MB each at 1080p). Do not use RGBA16F; 8-bit quantization actually helps keep blocks crisp.

**P4 — Composite and post**
- Inputs: P3, dry source; uniforms `wet`, `greenBlocks` (0–1).
- Per pixel: optional decode-error blocks: if `hash(b, frame/2) < greenBlocks · 0.01` → `c = #008700` (the RGB of an all-zero YCbCr block, like a corrupt macroblock). `out = mix(dry, c, wet)`. Black crush `max(c - 0.03, 0) / 0.97`.
- Output: output res, to the layer.

## Color scheme

The picture colour comes from the sources. The design adds decode-error accents and a stage grade.

| role | hex | use |
|------|-----|-----|
| base / black | #000000 | background, crushed blacks, edge mode "black" |
| shadow | #0B0B12 | floor of the grade; dark mosh areas never float above this |
| accent: decode green | #008700 | corrupt-block bursts (what zeroed YCbCr data decodes to) |
| accent: chroma magenta | #FF2BD6 | optional tint on chroma slip for the stylised version |
| accent: chroma cyan | #00E5FF | partner of magenta for chroma slip split |
| highlight | #F2F2F2 | peak white limit after grade |

- Mapping: no ramp or LUT on the main image. The mosh only moves existing colour. In the stylised version, the chroma-slipped sample is pushed toward magenta (`c.rb += 0.15 · stylise`) and the luma sample toward cyan. That reads as a codec colour split without being RGB-shift.
- Over time: chroma gain 1.0 in verses, 1.4 in drops. Green blocks only on fills and the last beat before a drop.
- LED walls: crush blacks fully (held frames from footage often sit at 5–10 % grey and pile up into a grey haze). Peak white 95 %. Long moshes average toward mid-tones and lose contrast, so add contrast 1.15 in the grade and fire a keyframe at least every 8 bars to bring back real blacks.

## Blending and compositing

- Wet over dry in **Normal**. Both are full images; Add or Screen would double the static parts and wash out the blocks.
- For "mosh only where it moves", use the P1 confidence channel (SAD) or the motion length as a matte: wet alpha = `smoothstep(0.5, 3.0, |mv|)` in px/frame, so still regions show the dry image. Good on cameras pointed at a static stage.
- Route A clips run at 100 % Normal over the clean layer; the cut back to clean is a hard cut on the downbeat, never a crossfade (a crossfade looks like a broken transition, not a mosh).
- No feedback mix in the usual sense: the mosh buffer **is** the feedback, at 100 %. Its "decay" is `drip` (0–0.05) and `refresh`.
- Post chain in order: black crush → contrast 1.15 → optional sharpen (unsharp, radius 1 px, amount 0.3, only for upscaled sources) → nothing else. No bloom, no blur, no grain: grain is moved by the mosh too and smears into a dirty film.

## Motion and timing

- What moves: blocks of the held picture, driven by the motion of the source. The design never invents motion except for the audio `push` and the jitter.
- Speed: source motion × `gain`. gain 1 = faithful mosh, 2–3 = exaggerated, 4 = explosive, use for one beat only.
- Keyframes on the grid: by default a keyframe fires on beat 1 of every 2nd bar (every 3.75 s at 128 BPM). On a hard kick section switch to every kick, but then lower gain to 0.6 so each mosh only lasts a beat.
- Route A on the grid: set the baked clips to BPM sync with their real length in bars (4 bars = 16 beats). Trigger them on the downbeat; the target clean clip is triggered on the downbeat where the mosh clip ends.
- Build-up: gain 1 → 2.5 over 8 bars, refresh 0.002 → 0, keyframes stop for the last 4 bars so the picture decays completely, green blocks rise on the last bar.
- Drop: keyframe on the downbeat to the new source (clean snap), then hold on for 2 beats with gain 2 (bloom), release, normal moshing at gain 1.2 with keyframes every bar.
- Break: switch to gradient flow (liquid), melt 0.7, tail 20, drip 0.02: slow melting instead of blocks. Or use Route A blooms on a clean loop.
- Hold timing: while a pad or sustained note holds, `hold` is on. Cap it at 4 bars, then force a keyframe, because longer blooms all end up as the same coloured soup.
- Never let one mosh run longer than 8 bars without a keyframe or refresh burst (the research names constant full-strength smear as the main way this effect gets boring).

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| mosh (gain) | 0–4 | 1.0 | how hard the motion pushes the held blocks; 0 = frozen picture | BeatStep Pro knob 1; bass envelope adds up to +1 |
| hold | off/on | off | P-frame repeat: freezes the vectors and lets them compound into a bloom | KeyStep sustain or a BeatStep pad (gate); release fires keyframe |
| keyframe | trigger | — | clean snap to the source, start of a new mosh | kick (every n-th, n = 1/2/4/8 on a knob) or pad 1; auto on bar 1 every 2 bars |
| block | 16 / 32 / 64 px | 16 | macroblock size; bigger = chunkier, more readable on big walls | knob 2 (stepped); hats jitter one step up for 1 frame |
| refresh | 0–0.15 | 0.002 | rate of blocks snapping back to the live image; bursts look like codec errors | knob 3; hats burst to 0.08 for 2 frames |
| source swap | A / B | A | which source the keyframe takes its picture from; motion always comes from the other | pad 2 toggles, quantised to the next beat |

`melt`, `chroma`, `residual`, `drip`, `greenBlocks` are set per song and not needed live. Wet is the layer opacity.

## Build recipes

### Resolume Arena

**Route A (baked clips), the robust gig option**
- Prepare: moshed transitions and blooms rendered offline (Technique 2), baked to DXV Normal Quality, No Alpha in Alley ([Resolume: Alley](https://resolume.com/article/81)). DXV is Resolume's codec that is decoded on the GPU ([Resolume DXV](https://www.resolume.com/software/codec)), so many layers of baked moshes stay cheap.
- Deck: column per song section; layer 1 holds the clean clips, layer 2 holds the mosh clips whose first frame matches the clean clip of layer 1 that came before them. Name clips `A>B_mosh_4bar_v3` so you know which clean clip must follow.
- Clip settings: transport BPM sync, length in beats as rendered, so the mosh lands on the bar at any tempo. Trigger on the downbeat (quantised clip launching).
- Playing: trigger the mosh clip on layer 2 → at its end trigger the clean target on layer 1 and clear layer 2. On the BeatStep Pro put mosh clips on pads, clean clips on the next row.
- Blooms (P-frame repeat clips) can loop on layer 2 over any clean layer at 100 %, or at 40–60 % Normal for a ghost layer.

**Route B (real time) in Arena**
- Option 1, Wire effect: build the Wire patch below, export it, and load it in Arena as an effect on the source layer. All parameters appear in the effect panel and can be mapped to MIDI or Arena's audio analysis.
- Option 2, external app over Spout: run [huff](https://schwwaaa.github.io/huff/) (paid, its datamosh pass displaces tiles from a 60-frame ring buffer, MIDI CC/notes, OSC on UDP 9000, Spout out on Windows) and bring it into Arena as a Spout source. The page says Spout goes through the CPU and is reliable at 720p/30 fps, so treat it as a 720p layer.
- Option 3, TouchDesigner over Spout: Optical Flow TOP for the vectors (Technique 4.5), the P2–P4 logic as GLSL TOPs with a Feedback TOP for the ping-pong, Spout Out TOP to Arena. Highest quality flow, but it needs an RTX 3000+ GPU ([Derivative](https://derivative.ca/UserGuide/Optical_Flow_TOP)).
- Custom ISF fits as the core of Option 1. Resolume documents ISF only inside Wire, where it has its own nodes; Wire 7.22 does not support vertex shaders ([Resolume: ISF](https://resolume.com/support/en/isf)). This design only uses fragment passes, so that limit does not matter. Multipass and persistent buffers in ISF depend on the Wire version: test before the gig that a persistent buffer keeps its content between frames. If it does not, use the Wire Buffer node for the ping-pong (below).

### Resolume Wire

- Node outline: Texture input (effect input, source A) → [optional second input or Spout source as motion source B] → ISF node "P0 luma" (quarter res) → ISF node "P1 motion" (also gets the previous luma) → ISF node "P2 shape" → ISF node "P3 mosh" → ISF node "P4 composite" → Output.
- Previous-frame textures: since 7.11 the **Buffer** and **Ring Buffer** nodes accept textures, which makes delay and feedback patches possible in Wire; long delays use a lot of VRAM ([Resolume 7.11](https://resolume.com/blog/21445)). Use a Buffer node with a one-frame delay for `Lp` (previous luma), one for `M_old` (P3 output fed back into P3), and one for `MV_held`/`MV_avg`. Only one frame each, so VRAM is small (about 8.3 MB at 1080p RGBA8 for the mosh buffer).
- If multipass ISF works in your Wire build, P0–P4 can live in one ISF with persistent buffers; else keep one ISF per node as above.
- Control side: BPM/beat node → bar counter → keyframe every n bars; audio analysis (low band) → envelope → kick keyframe and gain; (high band) → refresh burst and block jitter. Exact names of envelope, counter and audio nodes differ between Wire versions; check the node browser.
- Exposed parameters: mosh, hold, keyframe (event input), block, refresh, source swap, plus wet. ISF event inputs are supported in Wire ([Resolume: ISF](https://resolume.com/support/en/isf)) and fit keyframe and hold edges.
- A search found no built-in optical flow node in Wire, which is why P1 is a custom shader.

### Unreal Engine 5.7

- Use: mosh a rendered 3D scene (variation 4: move the camera to make the motion) or a Media Texture / NDI input.
- Render targets: `RT_Luma_A/B` 480×270 RTF R16F, `RT_MV` 120×68 RTF RG16F, `RT_MVHeld` 120×68 RG16F, `RT_Mosh_A/B` 1920×1080 RTF RGBA8, all mips off, filter nearest for `RT_MV`, bilinear for the rest. Ping-pong pairs alternate each frame by a Blueprint bool.
- Scene input: Scene Capture 2D (Capture Source "Final Color (LDR) in RGB") into `RT_Scene`, or the Media Texture.
- Passes (Blueprint on Tick, in order): Draw Material to Render Target with `M_Luma` → `M_Motion` (block search with Custom node, or the gradient flow in standard nodes) → `M_Shape` → `M_Mosh` (reads `RT_Mosh` of the other parity) → `M_Composite` into `RT_Out`. Materials in the Unlit domain; the logic per pass is as in Shaders.
- Optional: SceneTexture Velocity in a post-process material can replace P0–P1 for camera and object motion in a 3D scene. Check its encoding and sign in your build before relying on it. A commercial alternative is Petyu's Datamosh plugin, a post-process shader with Render Target and Screen modes, depth/stencil masking and packages listed up to UE 5.7 ([itch.io](https://petyu.itch.io/uedatamosh)).
- Niagara: optional, not part of the core. A GPU sprite system sampling `RT_MV` can spawn small square particles on high-motion blocks for extra "debris".
- Remote Control: a Material Parameter Collection `MPC_Mosh` with `Gain`, `Hold`, `Block`, `Refresh`, `Chroma`, `Melt`, `Wet`; Keyframe and SourceSwap as Blueprint functions. Expose them in a Remote Control Preset and bind to OSC and MIDI (BeatStep Pro pads for Keyframe/Hold).
- Output: `RT_Out` through the Spout or NDI plugin to Arena, at 1920×1080 60 fps.

### Godot

- Godot 4, Compatibility or Forward+ renderer, 2D only.
- SubViewports, all `render_target_update_mode = UPDATE_ALWAYS` ([Godot docs](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)):
  - `Luma` (480×270): TextureRect with the source and a luma `canvas_item` shader.
  - `LumaPrev` (480×270): TextureRect showing `Luma`'s ViewportTexture (one-frame-late copy).
  - `Motion` (120×68): ColorRect with the block-match shader reading `Luma` and `LumaPrev`.
  - `MotionHeld` (120×68, `render_target_clear_mode = CLEAR_MODE_NEVER`): a TextureRect showing `Motion`'s texture. The script sets its update mode to `UPDATE_DISABLED` while hold is on, so it keeps the last vectors at no cost, and back to `UPDATE_ALWAYS` on release.
  - `Mosh` (1920×1080, `CLEAR_MODE_NEVER`) and `MoshPrev` (1920×1080): `Mosh` runs the P3 shader reading `MoshPrev`; `MoshPrev` shows `Mosh`'s texture. This gives a one-frame feedback loop without reading a viewport into itself.
- Godot does not guarantee the draw order of viewports, so the copy may lag one extra frame. That only scales the mosh speed slightly; compensate with gain.
- Final: a full-screen ColorRect with the P4 shader.
- Source: VideoStreamPlayer (Theora) or CameraTexture.
- Uniforms to expose: `gain`, `hold`, `keyframe`, `block`, `refresh`, `melt`, `chroma`, `residual`, `drip`, `wet`; `frame` set from script. Audio: AudioEffectSpectrumAnalyzer on the music bus.

## Audio reactivity

| band | drives | smoothing | threshold / scaling |
|------|--------|-----------|---------------------|
| kick, 40–120 Hz | keyframe trigger (every n-th kick) | attack 5 ms, release 150 ms | trigger at 0.6 of the running peak (2 s window), retrigger blocked for 200 ms |
| bass, 60–250 Hz | gain +0 to +1 and `push` along a fixed direction (default left, 0–6 px/frame) | attack 40 ms, release 500 ms | linear above a 0.25 floor |
| sustain (bass or pad energy staying above 0.5 for > 1 beat) | hold on, release on drop | — | hold off when the energy falls below 0.3 |
| hats, 6–12 kHz | refresh burst 0.08 for 2 frames, block jitter one size up for 1 frame | attack 2 ms, release 60 ms | trigger at 0.5 |
| snare, 150–300 Hz + 2–5 kHz | green blocks 0 → 1 for 3 frames (fills only) | attack 2 ms, release 100 ms | trigger at 0.7, enable per song section |

Kick keyframes and the bar-grid keyframes are mutually exclusive: choose one per section, or the image is never allowed to decay.

## Performance tips

- Route A costs nothing beyond normal DXV playback. Do the heavy work at home and keep many variants.
- Route B cost per frame at 1080p (desktop GPU estimates, measure on your machine):
  - P0: 1 pass at 480×270, trivial.
  - P1 block-match full search: 8160 blocks × 169 candidates × 32 taps ≈ 44 M texture reads, about 1–2 ms. Coarse-to-fine cuts it to about 15 M. Gradient flow is about 1 M reads.
  - P2: trivial at 120×68.
  - P3: 2–3 taps per output pixel at 1080p, about 0.2 ms.
  - VRAM: under 25 MB in total (two 1080p RGBA8 mosh buffers, small luma and vector buffers).
- Resolution: motion at quarter res is enough (the blocks are 16 px anyway). Keep the mosh buffer at output res, otherwise block edges get soft when scaled.
- LED walls: a 16 px block on a low-res wall canvas (e.g. 1152×648) is huge. Scale block size with the canvas: `block ≈ canvasHeight / 68` as the default, then 2× and 4×. On large outdoor walls with coarse pixel pitch use 32–64 so the blocks read from a distance.
- Odd aspect ratios: keep blocks square in pixels. The motion texture is `ceil(width / 16) × ceil(height / 16)`, e.g. 360×68 for 5760×1080. Keep the search radius in pixels, not in uv, or vertical motion is under-searched on wide strips.
- Frame rate: vectors are in px per frame. If the output drops from 60 to 30 fps, each step moves twice as far and the mosh looks twice as fast. External flow (TouchDesigner) is in screen widths per second, so convert with the real measured fps.
- Pitfalls: low-contrast or static sources give no motion and nothing happens; noisy cameras (high ISO) give random vectors, so raise `flatThresh`; bilinear `melt` = 1 for long periods blurs the picture to mush; forgetting the zero bias makes black areas fizz. Feedback through Spout adds one frame of latency per hop, so keep Route B inside one app when possible.

## Further references

- [Datamoshing with Optical Flow (Careaga, Reddy, Aksoy, SIGGRAPH Asia 2023 Posters)](https://yaksoy.github.io/datamosh) — moshing with accurate optical flow instead of codec data; transitions and loops from the same principle as Route B.
- [FFglitch: averaging motion vectors](https://ffglitch.org/2024/06/mv_avg_010.html) — the official tutorial for scripting motion vectors offline; the basis for the `tail` smoothing idea.
- [tiberiuiancu/datamoshing](https://github.com/tiberiuiancu/datamoshing) — open-source scripts for I-frame removal, P-frame repeat, vector scripts and motion transfer (Route A).
- [Notch: Motion Datamosh](https://manual.notch.one/0.9.23/en/docs/nodes/post-fx/stylisation/motion-datamosh/) — a production node with velocity input, hold time and velocity amount; a useful model for which controls a live mosh needs.
