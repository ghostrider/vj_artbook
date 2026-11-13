# SDF Raymarched Morphing Forms — design

## Look

A dark void holds three to six soft forms (spheres, rounded boxes, tori) that orbit each other and melt together like liquid graphite. Where two forms meet, the join swells into a smooth neck. On every kick the necks get fatter and the cluster gulps together, then pulls apart. The surfaces read as satin metal: grey-blue bodies, crisp white speculars, deep ambient occlusion in the creases, and one hot orange rim light on the edges facing away from the camera. Far parts fade into a blue-black fog. Two more scenes use the same renderer: an endless **corridor** of repeated cells the camera flies through on a tempo-locked rail, each cell morphing between a box and a sphere on its own phase; and a **twist** scene, a stack of primitives wrung around the vertical axis. The mood is cool and sculptural, with a physical pulse on the beat. The research lists live raymarched sets such as evvvvil and OddJohn's 2017 show ([vvvv blog](https://legacy.vvvv.org/blog/evvvvil-oddjohn-generative-show)); the build-from-scratch process is shown in Inigo Quilez's "Sphere Gears" live-coding videos ([Part 1](https://www.youtube.com/watch?v=sl9x19EnKng)).

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | vertical gradient #07080B (top) to #0B0E14 (bottom), equal to the fog colour at the horizon | Normal | 100 % | fog fades into this exact colour, so forms dissolve without a visible edge |
| 2 | SDF render | raymarched scene (pass P1), rendered at 50 % scale, colour + coverage alpha | Normal (premultiplied alpha) | 100 % | the main layer; the alpha lets it sit over video too |
| 3 | Upscale + edge fix | bilinear upscale to output plus light sharpen (pass P2) | replaces 2 | 100 % | hides the render-scale stair steps |
| 4 | Ghost trails | feedback of layer 3, decay 0.75, slight zoom 1.003 | Add (inside the feedback buffer) | 0–40 % | off in normal playback; 25–40 % in breaks for a smeared, dreamy melt |
| 5 | Rim glow | bright-pass (threshold 0.7) of layer 3, blurred | Add | 60 % | makes the orange rim and speculars bloom on the LED wall |
| 6 | Mask out (optional) | coverage alpha or depth as greyscale (pass P1 alpha) | used as a Luma Mask on other layers | 100 % | research variation 4: the forms cut other video |
| 7 | Post | grain 2 %, vignette 0.2, clamp | see compositing | — | order below |

## Techniques

**1. Sphere tracing (the march)**
1. Per pixel: `uv = (fragCoord * 2 - res) / res.y` (height-normalised, so a wider screen shows more scene instead of stretching).
2. Camera: ray origin `ro` on the camera path, look-at target `ta`; build a basis (forward `f = normalize(ta - ro)`, right, up) and `rd = normalize(uv.x * r + uv.y * u + 1.6 * f)`. The 1.6 is the focal length (about 64° vertical FOV); range 1.2 (wide) to 2.5 (tele).
3. Loop: `t = 0`; repeat up to MAX_STEPS: `d = map(ro + rd * t)`; if `d < eps * t` → hit; `t += d * stepScale`; if `t > MAX_DIST` → miss. The idea of stepping by the SDF value and stopping at a threshold is the sphere tracing described on [Wikipedia](https://en.wikipedia.org/wiki/Ray_marching) and in [Jamie Wong's tutorial](https://jamie-wong.com/2016/07/15/ray-marching-signed-distance-functions/).
4. Values: MAX_STEPS 80 (range 48–128), `eps = 0.0008` relative to t (a hit tolerance that grows with distance, so far objects stop early), MAX_DIST 30 units (blob scene) / 60 units (corridor), `stepScale` 0.9 (see technique 3 for when to lower it).
5. A ray that runs out of steps without a hit is treated as a miss and gets fog colour. This avoids black speckles at grazing edges.

**2. The scene function `map(p)` and morphing**
1. Primitives (all in local space, after moving p): sphere `length(p) - r`; rounded box `length(max(abs(p) - b, 0)) - rr`; torus `length(vec2(length(p.xz) - R, p.y)) - r`. Moving or rotating a shape means transforming p by the inverse, as Jamie Wong's article shows for rigid transforms.
2. Smooth union (the liquid join): `h = clamp(0.5 + 0.5 * (b - a) / k, 0, 1)`, `d = mix(b, a, h) - k * h * (1 - h)`. `k` is the blend radius in scene units: 0 = hard union, 0.3 = soft neck, 1.0 = everything melts into one blob.
3. Shape morph: `d = mix(dA, dB, morph)` between two primitives at the same position, for example sphere → rounded box → torus as morph runs 0 → 1 → 2. The research's sequenced shape bank uses this with MIDI notes picking A and B.
4. Kick-blob chain (scene A): N = 5 spheres, radius 0.45, on a Lissajous orbit `pos_i = (1.3 sin(φ + i·2π/N), 0.6 sin(2φ + i), 1.3 cos(φ + i·2π/N))` where φ is the bar phase × 2π. Unrolled smooth union over all N. One centre sphere, radius 0.6, anchors the cluster.
5. Hard CSG (`min`, `max`, `max(a, -b)`) is available for cutouts, e.g. subtract a thin slab to slice the blob on hi-hats ([Jamie Wong](https://jamie-wong.com/2016/07/15/ray-marching-signed-distance-functions/)).

**3. Space warps: twist, bend, displacement**
1. Twist around y: rotate p.xz by angle `twist * p.y` before evaluating the shapes. `twist` in radians per unit, range 0–3.
2. Bend: rotate p.xy by angle `bend * p.x`, range 0–1.5.
3. Noise-like ripples: `d += amp * sin(f * p.x) * sin(f * p.y) * sin(f * p.z)`, f = 6–12, amp 0–0.06.
4. All three, and smooth union itself, make `map` return a distance that is no longer exact (a true SDF has gradient length 1 almost everywhere, per [Wikipedia: Signed distance function](https://en.wikipedia.org/wiki/Signed_distance_function)). The marcher then overshoots and you see holes or ripples at silhouettes.
5. Fix: `stepScale = 0.9 / (1 + 0.35 * twist + 0.5 * bend + 6 * amp)`, clamped to 0.45–0.9, and raise MAX_STEPS by the same factor (cap 128). Lowering the step scale costs frame time, so the twist fader is also the cost fader.

**4. Domain repetition (scene B: corridor)**
1. Cell size `c` (density parameter, 1.5–4 units). `id = floor((p + c/2) / c)`, `q = mod(p + c/2, c) - c/2`. Taking the modulus of the coordinates tiles one shape through space ([Wikipedia: Ray marching](https://en.wikipedia.org/wiki/Ray_marching)).
2. Repeat only in x and z (a corridor floor and ceiling) or in all three axes (a lattice). Leave y unrepeated for the corridor and add two planes at y = ±1.4 as walls.
3. Per-cell variation: `h = hash(id)` in 0..1. Per-cell morph = `0.5 + 0.5 sin(beatPhase · 2π + h · 6.28)`, so the cells ripple in a travelling wave rather than all at once.
4. Guard: the shape must stay inside its cell. Keep the largest radius below 0.4 c, otherwise the distance to the neighbour cell is ignored and the marcher clips through it.
5. Camera on a rail along z: `ro.z = speed · beatsElapsed · c`. Wrap `ro.z` with `mod(ro.z, c · 64)` every time it passes a multiple of 64 cells. Without the wrap, 32-bit floats lose precision after a long set (tens of thousands of units) and the image starts to shimmer.

**5. Shading: normal, AO, rim, fog**
1. Normal: tetrahedral gradient with 4 `map` calls, offsets `e = 0.0005 · t` on the vectors (1,−1,−1), (−1,−1,1), (−1,1,−1), (1,1,1). It does the same job as the central-difference normal in [Jamie Wong's tutorial](https://jamie-wong.com/2016/07/15/ray-marching-signed-distance-functions/) with 4 calls instead of 6.
2. Ambient occlusion: 5 samples along the normal at distances 0.02, 0.08, 0.16, 0.24, 0.32; `ao = 1 - 3 · Σ (dist_i − map(pos + n·dist_i)) · 0.7^i`, clamped 0–1. This gives the dark creases inside the smooth-union necks.
3. Lighting: key light from upper left `L = normalize(−0.5, 0.8, 0.4)`, diffuse `max(dot(n, L), 0)`, Blinn specular with exponent 64; soft fill from below at 15 %.
4. Rim: `rim = pow(1 − max(dot(n, −rd), 0), 3)` times the accent colour. Combined with the dark body this is where the orange edge comes from.
5. Fog: `col = mix(col, fogCol, 1 − exp(−0.06 · t))` (corridor 0.035). Optional soft shadow (16 steps) only in scene A and only at High quality.

**6. Feedback ghosting (breaks only)**
1. Buffer F at render-scale resolution.
2. `F = F_prev(uv scaled toward centre by 1.003) · decay + current`, decay 0.75 (range 0.5–0.9).
3. Mix F into the output at 0–40 %. In drops set it to 0 so the forms stay sharp.

## Shaders

Render order per frame: P1 raymarch → P2 upscale → P3 feedback (optional) → P4 glow → P5 composite/post.

**P1 — Raymarch**
- Purpose: render the scene.
- Inputs: uniforms `scene` (0 blob, 1 corridor, 2 twist), `blend k`, `morph`, `twist`, `bend`, `ripple amp`, `density c`, `barPhase`, `beatsElapsed`, `pulse` (kick envelope 0–1), `camera dist`, `quality` (0 low, 1 high), palette colours, `renderRes`.
- Per pixel: (1) build ray (technique 1); (2) **bounding sphere test** for scene A and C: intersect the ray with a sphere of radius 3.2 around the origin analytically; if it misses, write fog colour and alpha 0 and stop; otherwise start t at the entry point; (3) march; (4) on hit: normal, AO, lighting, rim, fog; (5) write colour, alpha = 1 on hit, 0 on miss (fog fade also fades alpha: `alpha = exp(−0.06 · t)`).
- Output: RGBA16F at **render scale × output**. Default scale 0.5. Optionally write linear depth `t / MAX_DIST` to a second target (R16F) for the mask mode.

**P2 — Upscale and sharpen**
- Inputs: P1 texture (bilinear filtering).
- Per pixel: sample bilinear; unsharp mask with a 5-tap cross, amount 0.3. At 0.5 scale this restores most of the silhouette crispness.
- Output: RGBA16F at output resolution. Skip this pass and let the host scale (Arena layer scaling, Godot TextureRect) if you need the milliseconds.

**P3 — Feedback (ping-pong)**
- Inputs: previous F, P1 output, decay, zoom.
- Per pixel: technique 6. Two RGBA16F buffers at render-scale resolution, swapped each frame.
- Output: F.

**P4 — Glow**
- Inputs: P2 (or P1).
- Bright-pass at threshold 0.7 → downsample to 1/4 output resolution → separable Gaussian blur, 9 taps each way, radius ~ 1 % of output height → output.
- Output: RGBA16F at 1/4 resolution.

**P5 — Composite and post**
- Inputs: background gradient, P2, P3, P4.
- Per pixel: background; over it P2 (premultiplied alpha); add P3 × trailMix; add P4 × 0.6; grain 2 % (monochrome, per frame seed); vignette 0.2; clamp to 0–1 for 8-bit Spout/NDI.
- Output: RGBA8 at output resolution.

## Color scheme

| role | hex | use |
|------|-----|-----|
| black / fog far | #07080B | background top, far fog |
| fog near horizon | #0B0E14 | background bottom, fog colour |
| shadow body | #1C1F26 | albedo in AO-dark creases (albedo × ao) |
| base body | #5A6170 | main albedo of the forms, satin graphite |
| highlight | #F2F4F7 | specular, kick flash |
| accent rim | #FF5A1F | rim light, hot orange |
| alt accent | #3DDCFF | swap-in rim for a second section, ice cyan |
| alt accent 2 | #C6FF3D | rare third rim for the peak drop, acid green |

- Mapping: shading is mostly value, not hue. Body colour = mix(#1C1F26, #5A6170, ao · diffuse), plus white spec, plus rim × accent. The palette stays near monochrome with a single accent, as the research suggests.
- Corridor scene: per-cell tint picks between body and accent with the cell hash; only 1 in 8 cells (`h > 0.875`) gets the accent as albedo, so the corridor has sparse orange beacons.
- Over time: accent switches per 16-bar phrase from the list (orange → cyan → orange), and acid green only on the peak drop. Accent swaps cut on the bar downbeat, never fade (a fade passes through muddy mixed tones).
- Music: kick lifts rim intensity from 0.8 to 1.6 with the pulse envelope; hi-hats add a 40 ms specular flash (spec × 1.5).
- LED walls: black level #07080B, not pure 0, so dark modules do not look switched off; keep body albedo under 40 % brightness so the speculars and rim have headroom. Peak (spec + rim + glow) should clip only in a few pixels. Contrast is high; if the wall's processor has gamma 2.2+, lift the body to #66707F so the AO creases do not crush.

## Blending and compositing

- **SDF over background: Normal with premultiplied alpha.** The form is a solid object, not light, so it must occlude. The alpha also allows placing it over a video layer.
- **Glow and trails: Add.** Glow is light; Add keeps the rim bright where it overlaps the body.
- **Feedback mix**: decay 0.75, mix 0 % in drops, 25–40 % in breaks. Above decay 0.9 the forms smear into grey soup, because the dark bodies also accumulate.
- **Mask mode**: output P1 alpha (or depth) alone. In Resolume, put it on a layer above a video layer and use it as that layer's luma mask: the video shows only inside the forms. Depth used as a matte gives a soft falloff (near = white, far = black) for fog-like video blends.
- **Post chain in order**: upscale + sharpen 0.3 → glow (threshold 0.7, 60 %) → grain 2 % → vignette 0.2 → clamp. No chromatic aberration by default: it fights the clean satin look. Allowed as a one-beat hit on the drop (offset 2 px), then off.

## Motion and timing

- **Clock**: everything runs from the host's beat clock (bar phase 0–1, beats elapsed), never from wall-clock time. Then the orbit stays locked after tempo changes.
- **Scene A orbit**: one full orbit per 4 bars (range 1–16 bars). Camera orbits the cluster at radius 5 (`camera dist`, range 3–9), one turn per 16 bars, height bobbing ±0.6 units on a sine per 8 bars.
- **Kick pulse**: k jumps from base (0.25) to base + 0.35 · pulse, and every sphere radius grows by 0.08 · pulse. Envelope attack 5 ms, release 180 ms with an exponential ease-out. This is the "gulp".
- **Morph**: the shape morph eases with smoothstep over 1 bar (range 1 beat to 4 bars) when a new pair is chosen; a BeatStep Pro pad or a KeyStep note triggers the next pair.
- **Corridor**: camera speed 1 cell per beat (range 0.25–4). Per-cell morph wave tied to beat phase. Camera roll ±8° on a sine per 8 bars.
- **Twist scene**: twist eases between 0 and the fader value over 2 bars; ripple amp follows hi-hats.
- **Build** (last 8 bars before a drop): blend k ramps from 0.25 to 0.9 (everything merges into one blob), camera dist closes from 5 to 3.2, ripple amp climbs to 0.05, fog thickens (0.06 → 0.12).
- **Drop**: hard cut on the downbeat. Scene switch (A → B or B → A), k snaps back to 0.15 so the forms burst apart, accent swaps, chromatic offset for one beat, trails 0 %.
- **Break**: camera speed halves, trails to 30 %, rim drops to 0.5, the forms drift with k near 0.6. Optional slow scene C at low twist.

## Parameters

| name | range | default | what it does on stage | suggested MIDI / BPM mapping |
|------|-------|---------|----------------------|------------------------------|
| blend | 0–1.2 (k, scene units) | 0.25 | how liquid the joins are: 0 = separate hard forms, 1 = one melted blob | KeyStep knob; kick envelope adds +0.35 |
| morph | 0–2 | 0 | morphs sphere → rounded box → torus; in the corridor it biases all cells | BeatStep Pro pads 1–8 pick target pairs; eased over 1 bar |
| twist | 0–3 rad/unit | 0 | wrings space around the vertical axis; also lowers step scale (cost) | fader; bass envelope adds up to +0.4 |
| density | 1.5–4 (cell size) / 3–6 (blob count) | 2.5 / 5 | corridor cell spacing or number of blobs in the cluster | knob, stepped; change on bar lines only |
| speed | 0–4 (cells or orbits per beat-scaled unit) | 1 | camera travel and orbit speed, BPM-locked multiples | knob quantised to 0.25, 0.5, 1, 2, 4 |
| pulse | 0–1 | 0.6 | how hard the kick pumps blend and radii | kick envelope × this knob |

Plus a `scene` selector (0/1/2, pads) and a `quality` toggle (low/high: steps 56/96, AO 3/5 samples, shadow off/on).

## Build recipes

### Resolume Arena

Arena runs the raymarcher as a Wire patch loaded as a source; ISF support is documented for Wire, which has its own ISF node ([Resolume ISF support](https://resolume.com/support/en/isf)).
- **Layer 1**: Solid Color or a gradient source in #07080B / #0B0E14.
- **Layer 2**: the Wire patch (P1 + P2 as one single-pass ISF) as a source, Normal blend, 100 %. Expose blend, morph, twist, density, speed, pulse, scene, quality as Wire inputs; they appear as clip parameters in Arena.
- **Render scale**: make the Wire patch output 1920 × 1080 (or 1280 × 720 on weak GPUs) and let Arena scale the clip to fill a 4K composition. This is the simplest way to get the 0.5 scale without multipass.
- **Effects on layer 2**: Bloom (threshold about 0.7, moderate intensity), then a light Sharpen if the upscale looks soft.
- **Ghost trails**: do the feedback inside the Wire patch (see Wire) rather than in Arena, so it runs at the patch's lower resolution.
- **Dashboard**: map blend, twist, speed to KeyStep knobs via MIDI shortcuts; pads on the BeatStep Pro to scene and morph-pair presets. Speed should use BPM-synced values only.
- **Audio**: Arena's audio analysis on the low band drives pulse; on the twist parameter use a smoothed bass band.
- **Mask mode**: a second instance of the patch with `scene` output set to alpha-only (white forms on black), placed above a video layer and used as its luma mask.
- **External engine**: if the patch is too heavy, render in UE5 or Godot and bring it in via Spout or NDI on layer 2.

### Resolume Wire

Node outline:
- `BPM` / transport → bar phase and beat counter → `Multiply` by speed → camera phase input. Keep a separate beat counter for `beatsElapsed` and wrap it (mod 64 · cell count) to protect float precision.
- `Audio Analysis` (FFT) → low band → envelope (attack 5 ms, release 180 ms) → `Multiply` by pulse → ISF `pulse` input. Bass band → slower envelope (attack 60 ms, release 500 ms) → twist offset.
- MIDI or Arena parameters → ISF float inputs (with MIN/MAX in the ISF JSON header, as the Resolume page describes).
- `ISF` node with the raymarch shader (single pass: march, shade, fog, alpha) at 1920 × 1080 → feedback loop (previous frame × decay, Add current, Wire's own feedback / previous-frame node) → `Mix` by trail amount → blur-and-add glow (bright threshold, downscale, blur, Add) → output.
- The ISF spec allows multiple passes and persistent buffers with sizes like a fraction of `$WIDTH` ([ISF JSON reference](https://docs.isf.video/ref_json.html)). Wire's support for multipass ISF has varied by version, so keep the raymarch single-pass and build feedback and glow from Wire nodes. Wire has no vertex shaders (version 7.22, per the Resolume page), which does not matter here: everything is a fragment pass.
- Exposed parameters: blend, morph, twist, density, speed, pulse, scene, quality, trail amount, accent colour.

### Unreal Engine 5.7

- **Raymarch material**: a Material (Domain: Surface, Unlit, or Post Process) built around one **Custom** node holding the march loop, `map`, normal, AO and shading in HLSL; inputs ScreenUV, aspect, and every parameter. Epic notes that a Custom node blocks constant folding and can cost more instructions than built-in nodes ([UE docs: Custom material expressions](https://dev.epicgames.com/documentation/en-us/unreal-engine/custom-material-expressions-in-unreal-engine)); for a loop this is still the only practical route.
- **Resolution control**: render the material into a **Render Target** (RGBA16F, 1920 × 1080 for a 4K wall) with *Draw Material to Render Target* each tick from a Blueprint, then show the render target full screen (a UMG Image or a post-process material that samples it, bilinear). This keeps the expensive pass at a fixed size independent of the viewport.
- **Parameters**: a Material Parameter Collection `MPC_SDF` with scalars Blend, Morph, Twist, Density, Speed, Pulse, Scene, Quality, BarPhase, BeatsElapsed and vectors AccentColor, BodyColor. Expose them in a **Remote Control Preset**, bound to OSC (BarPhase and Pulse sent from Resolume or an audio analyser) and MIDI (KeyStep / BeatStep Pro).
- **Optional Niagara layer**: a GPU emitter with 20–50k tiny additive sprites orbiting on the same Lissajous path (reading the same BarPhase from a User parameter), as glowing dust around the blobs. They are not occluded by the raymarched forms unless you compare against the depth render target, so keep them sparse and dim.
- **Post-process volume**: Bloom intensity around 1, threshold 0.7; manual exposure (no eye adaptation, or the kick flash pumps the whole image); Film Grain low; Vignette 0.2.
- **Output**: send the final frame via a Spout plugin for Unreal or the NDI SDK for Unreal into Resolume at the LED processor's native resolution.

### Godot

- **Raymarch**: a `SubViewport` (size = render scale × output, e.g. 1920 × 1080, update mode Always, HDR 2D on) containing a full-rect `ColorRect` with a `canvas_item` shader. Build uv from `FRAGCOORD.xy` and `1.0 / SCREEN_PIXEL_SIZE` (the built-ins listed in the [Godot canvas_item reference](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/canvas_item_shader.html)), march in `fragment()`.
- **Display**: a `TextureRect` with a `ViewportTexture` from that SubViewport, stretch to the window, texture filter Linear. A second shader on the TextureRect does the sharpen (P2).
- **Feedback**: a second `SubViewport` whose `ColorRect` shader reads its own previous frame through a `ViewportTexture` (one frame of delay) times decay plus the raymarch texture.
- **Uniforms** (`hint_range` for each): blend_k, morph, twist, bend, ripple, density, speed, pulse, scene (int), quality (int), bar_phase, beats_elapsed, accent_color, body_color. Drive bar_phase from a script that integrates BPM, not `TIME`.
- **Glow**: `WorldEnvironment` with glow enabled (canvas background mode for 2D), threshold 0.7.
- **Control**: `InputEventMIDI` in `_input()` maps CC numbers to the uniforms; OSC via a community addon if needed.

## Audio reactivity

| band | drives | smoothing | threshold |
|------|--------|-----------|-----------|
| kick, 40–100 Hz onset | pulse envelope → blend k (+0.35) and radii (+0.08); rim × 2 | attack 5 ms, release 180 ms | onset at +6 dB over the 1 s average |
| bass, 60–250 Hz level | twist offset (0–0.4) and camera bob | attack 60 ms, release 500 ms | none, continuous |
| mids, 300 Hz–2 kHz | ripple amp 0–0.03 (surface shimmer) | attack 80 ms, release 600 ms | gate below −36 dBFS |
| hi-hats, 6–12 kHz | specular flash (×1.5, 40 ms) and corridor cell-offset step | attack 1 ms, release 60 ms | gate at −30 dBFS |
| full-band loudness, 4 s average | fog density (louder = clearer, quieter = foggier) | attack 1 s, release 3 s | none |

Use envelopes, not raw levels (the research flags raw level as boring and jittery). Only the kick moves geometry fast; everything else breathes.

## Performance tips

Raymarching cost scales with pixels × steps × cost of `map`. Rough cost per pixel in `map` calls: march 40–96 (most rays exit early in scene A), normal 4, AO 5, shadow 16 (high only) → about 60–120 calls for hit pixels, 0 for pixels rejected by the bounding sphere.

Estimated budget (one `map` for 6 smooth-unioned spheres is roughly 60–80 GPU instructions; these are order-of-magnitude estimates from that count, not benchmarks; measure on your machine):

| output | render scale | pixels marched | scene A (blobs, ~35 % of pixels hit) | scene B (corridor, ~100 % hit) |
|--------|-------------|----------------|-------------------------------------|-------------------------------|
| 1920 × 1080 | 1.0 | 2.07 M | ~2–4 ms mid-range GPU | ~6–10 ms |
| 3840 × 2160 | 0.5 | 2.07 M | ~2–4 ms | ~6–10 ms |
| 3840 × 2160 | 1.0 | 8.3 M | ~8–15 ms | ~25–40 ms (misses 60 fps) |
| 7680 × 1080 (wide strip) | 0.5 | 2.07 M | ~2–4 ms | ~6–10 ms |
| 7680 × 2160 | 0.5 | 4.15 M | ~4–8 ms | ~12–20 ms |

- **Render scale** is the main lever: 0.5 on any output above 2.5 M pixels, 0.67 for 1080p walls with a fine pitch, 1.0 only for small outputs. LED walls with pitch 3.9 mm and up hide 0.5 scale completely.
- **Step cap**: 80 is the sweet spot. Below 48 the twist scene shows holes; above 128 the extra steps go into grazing rays that hit nothing.
- **Relative epsilon** (`eps · t`) saves many steps at distance and costs nothing visible.
- **Bounding sphere** in scenes A and C skips 50–70 % of pixels in wide shots. The corridor cannot use it, so it is the most expensive scene; give it fog 0.035+ and MAX_DIST 40 so far rays stop early.
- **Quality toggle**: drop the soft shadow first (saves ~15 %), then AO to 3 samples, then steps to 56.
- **Unroll** fixed blob counts (compile-time N = 3, 5, 6 variants) instead of a dynamic loop over a uniform; dynamic loops inside `map` multiply the cost of every step.
- **Float precision**: wrap the camera position and beat counter (technique 4 step 5); a corridor that has been flying for an hour otherwise shimmers.
- **Odd aspect ratios**: height-normalised uv means a 32:9 strip sees a wider slice of the scene, so the blob cluster can look small. Raise the focal length to 2.2–2.5 on ultra-wide outputs, or run two cameras side by side. Never stretch the render: the spheres turn into ovals.
- **Smooth union and warps** need the lower step scale (technique 3); if you see holes at silhouettes during twist, that is overstepping, not too few steps.
- **Mask mode** only needs alpha: skip AO, lighting and shadows and stop at the first hit, which is about 3× cheaper.

## Further references

- [Jamie Wong: Ray Marching and Signed Distance Functions](https://jamie-wong.com/2016/07/15/ray-marching-signed-distance-functions/) — step-by-step tutorial: sphere tracing loop, normals from the SDF gradient, CSG, Phong lighting, camera matrix.
- [Wikipedia: Ray marching](https://en.wikipedia.org/wiki/Ray_marching) — sphere tracing definition, domain repetition by modulus, strengths of SDF ray marching.
- [ISF JSON reference](https://docs.isf.video/ref_json.html) — PASSES, PERSISTENT and FLOAT buffers and size expressions, for hosts that support multipass ISF.
- [Sebastian Lague: Coding Adventure: Ray Marching](https://www.youtube.com/watch?v=Cp5WWtMoeKg) — video tutorial on building a raymarcher (from the research); [Marching.js](https://github.com/charlieroberts/marching) — a JavaScript library that compiles GLSL ray marchers, with an API made for live coding.
