# Boids Murmuration Flocks — design

## Look

One dark, smoky body of thousands of tiny birds moves over the wall. It never holds a shape. It stretches into a ribbon, folds into a ball, thins out until you see single birds, then a dark wave runs through it as the birds turn together. This is the starling look the research describes, after the Commons photo of a real murmuration. The default "Night" look puts pale silver birds on true black. Dense areas glow warm cream, while far, thin areas fade to cool grey-blue. An invisible predator orbits in time with the music. On bar boundaries it dives through the cloud, the flock tears into two or three bodies, and they merge again within a few beats. On the kick the flock pulls together for a moment, like it is taking a breath. On the drop a turning wave runs across the whole cloud with a short warm-orange flash along its front. The alternative "Dusk" look reverses this: near-black birds on a dim violet-to-amber sky gradient, closer to the photo. The mood is calm, huge and alive. The music shows in how the crowd behaves, not in strobing.

## Layer stack

The flock simulation runs in a sim host: Unreal Engine 5.7, TouchDesigner or Godot. It comes into Resolume over Spout (same machine) or NDI (second machine). Resolume does colour, compositing and post. Wire has no vertex shaders ([Resolume ISF](https://resolume.com/support/en/isf)), so it cannot draw thousands of moving points efficiently. Wire only gets the colourize effect and a small fallback flock (Build recipes).

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | Night: solid #000000. Dusk: vertical gradient #141830 → #4A3557 → #B8704A | Normal | 100 % | Dusk sky capped at about 55 % luminance (LED power, glare) |
| 2 | Video bed (optional) | any clip, shown only inside the flock (matte mode) | Normal | 100 % | used with layer 4 in Multiply mode |
| 3 | Agent state | per-boid position and velocity (3D), species and flags. Lives in the sim host as textures, particle attributes or storage buffers | not visible | — | 30 k–300 k boids (estimates, see Build recipes) |
| 4 | Flock render (sim output) | boids drawn as velocity-stretched wedges or point sprites, accumulated into a float density buffer, tonemapped and packed: R = density, G = "event" glow (wave front, predator zone), B = depth (0 = near, 1 = far) | Normal (Night), Multiply (Dusk / matte) | 100 % | Spout/NDI source `Flock`, 1920×1080 RGBA8 |
| 5 | Colourize | Wire/ISF effect on layer 4: depth-tinted ramp on R, accent on G | inside layer 4 | — | see Color scheme |
| 6 | Goal / logo field (optional) | white logo on black, sent from Resolume back to the sim host | not visible here (Spout out → sim) | — | flock briefly forms the shape (Technique 5) |
| 7 | Post | Bloom, light grain | effects on layer 4 | — | bloom threshold 0.75, grain 1 % |

## Techniques

**1. Agent state that persists (ping-pong)**
1. Each boid stores `p` (vec3, world units) and `v` (vec3, units/s), plus `species` and `flags` (e.g. "hit by wave" timer). That is 2 × 16 bytes if packed as two RGBA32F texels or vec4s.
2. All steering reads the *previous* state and writes a *new* buffer (A → B, then swap). Otherwise half the flock would react to neighbours that have already moved this tick, and the result depends on thread order. The [three.js GPGPU birds example](https://raw.githubusercontent.com/mrdoob/three.js/dev/examples/webgl_gpgpu_birds.html) does exactly this with two float textures (`textureVelocity`, `texturePosition`). Velocity updates first, then position, one texel per bird.
3. Positions must be 32-bit float. Half floats jitter visibly once the flock is more than a few hundred units across.
4. Initial state: random points in a sphere of radius 30 u, with velocities along a common direction plus 30 % noise. Otherwise the flock spends its first seconds just agreeing on a heading.
5. Integration uses a fixed `dt = 1/60 s`, with 1–2 substeps chosen from real frame time. The flock's speed then stays the same when the frame rate drops.

**2. Steering rules (Reynolds), with topological neighbours**
1. Neighbourhood: up to `k` nearest flockmates (default 7, after the STARFLAG finding of about seven neighbours, [The Contrarian](https://contrarian.ca/2012/06/20/the-seventh-starling/)). They must lie inside a hard search radius `rMax` and a field of view. Reynolds bounds the neighbourhood by a distance and an angle ([Reynolds](https://www.red3d.com/cwr/boids/)). Field of view: ignore a neighbour if `dot(normalize(pj − p), normalize(v)) < cos(fov/2)`, with fov 270° by default.
2. Separation: `sep = Σ (p − pj) / |p − pj|²` over neighbours closer than `rSep`.
3. Alignment: `ali = mean(vj) − v`.
4. Cohesion: `coh = mean(pj) − p`.
5. Extra forces: `flee` from the predator, `bound` (soft sphere, see 4), `noise` (curl noise of `p · 0.02 + t · 0.05`, magnitude 1), `goal` (Technique 5), `wave` (Technique 4).
6. Combine: `a = wS·sep + wA·ali + wC·coh + wP·flee + wB·bound + wN·noise + wG·goal`, then clamp `|a| ≤ maxForce`. Next `v += a·dt` and clamp speed to `[vMin, vMax]`, then `p += v·dt`. The minimum speed matters: starlings never hover, and a flock without `vMin` collapses into a buzzing ball.

Defaults, with world units chosen so that the flock is about 100 u across: `rMax` 8 u, `rSep` 2 u, `wS` 1.6, `wA` 1.0, `wC` 0.9, `vMin` 8 u/s, `vMax` 16 u/s, `maxForce` 30 u/s². These are starting points to tune by eye, not values from a source.

**3. Neighbour search on the GPU**
The naive version tests every boid against every other boid. Reynolds notes this is O(n²) and that a spatial structure sorted by location brings it close to linear ([Reynolds](https://www.red3d.com/cwr/boids/)). The three.js example is the naive version: it loops over every texel and skips birds outside the zone radius ([source](https://raw.githubusercontent.com/mrdoob/three.js/dev/examples/webgl_gpgpu_birds.html)). That is fine for 1,024 birds, but not for a murmuration. Three methods, in order of preference:
- **A. Uniform grid with counting sort (compute shaders: Godot, UE custom HLSL, TD GLSL POP).** Cell size = `rMax`. (1) Clear the `cellCount` array. (2) Per boid: `cell = floor((p − gridMin) / rMax)`, linear index `c`, then `atomicAdd(cellCount[c], 1)`, and store the returned slot. (3) Prefix sum over `cellCount` gives `cellStart`. (4) Per boid: `sorted[cellStart[c] + slot] = boidIndex`. (5) Steering pass: each boid scans its own and the 26 neighbouring cells, stops after `maxCandidates` (default 48), and keeps the `k` closest in a small register list (insertion sort, k ≤ 16). This is the same sort-then-find-cell-starts idea as the GPU Gems 3 broad-phase chapter. That chapter uses a radix sort and requires cells at least as large as the interaction range ([NVIDIA GPU Gems 3, ch. 32](https://developer.nvidia.com/gpugems/gpugems3/part-v-physics-simulation/chapter-32-broad-phase-collision-detection-cuda)). The grid only needs to cover the bounding sphere. Example: radius 70 u / 8 u cells = 18³ ≈ 5.8 k cells, which is tiny.
- **B. Built-in neighbour grids.** Niagara's Neighbor Grid 3D stores up to `MaxNeighborsPerCell` particle indices per cell ([Epic API](https://dev.epicgames.com/documentation/unreal-engine/API/Plugins/Niagara/UNiagaraDataInterfaceNeighborGri-)). Girardot's public Niagara boids system is built on it ([80.lv](https://80.lv/articles/check-out-a-new-niagara-based-boids-system-set-up-in-ue5-3)). In TouchDesigner, the Neighbor POP writes the indices of nearby points into an array attribute `Nebr`, using a spatial hash with a set number of buckets, a max distance and max neighbours ([Derivative](https://docs.derivative.ca/Neighbor_POP)).
- **C. Persistent kNN cache (fragment-only, for TD GLSL TOP and the Wire fallback; a design proposal, untested).** Fragment passes cannot scatter, so a grid cannot be built. Instead, each boid keeps its current best `k` neighbour indices in an extra texture (e.g. two RGBA32F texels = 8 indices). Each tick it tests (a) its cached `k`, (b) one cached neighbour of each cached neighbour ("neighbours of neighbours"), and (c) 8 random boids from `hash(id, frame)`. It keeps the closest `k` of these ~23 candidates. The cache converges within a few frames and follows the flock as it moves, because neighbours of neighbours are usually close. Cost is constant per boid. Splits work, since the random samples find new neighbours after a merge.

**4. Making it a murmuration, not a swirl (design proposals)**
1. Soft bound: if `|p − centre| > R` (R = 50 u), `bound = −normalize(p − centre) · (|p − centre| − R) · 0.5`. A sphere instead of a box gives no corners and no straight walls in the motion.
2. Predator: a point on a beat-synced Lissajous path around the centre (period 4 bars on x, 3 bars on y, 2 bars on z, radius 0.8 R). Inside `fleeRadius` (default 18 u), `flee = normalize(p − pred) · (1 − d / fleeRadius)²`, `wP` default 4. Also raise `vMax` by 40 % for those boids, like the three.js example raises its speed limit near the predator ([source](https://raw.githubusercontent.com/mrdoob/three.js/dev/examples/webgl_gpgpu_birds.html)). On the bar downbeat (every 1, 2 or 4 bars, selectable), the predator dives: it moves along a straight line through the flock's centre of mass over 1 beat, eased in-out. This makes the split the research asks for.
3. Turn wave (drop, pad): a spherical front grows from a seed point at 60 u/s. A boid crossing the front gets a lateral impulse `wave = cross(v, up) · 6` for 0.15 s and sets its "hit" timer. Alignment carries the turn on, so it looks like a real wave of turning. The hit timer drives the G "event" channel for 0.4 s.
4. Breathing (kick): `wC` jumps ×2.5 with a 250 ms release. The flock contracts and relaxes on each beat.
5. Centre of mass: computed once per frame by a reduction (or from 1,024 sampled boids). The camera and the bound centre follow it with a lag of 2 s, so the flock never leaves frame and the bound never yanks it.

**5. Goal field: the flock draws a shape (variation)**
1. Bake the logo into a 2D signed distance texture (512×512) once in the sim host. Its plane is z = 0 in flock space, scaled to 0.8 of the flock width.
2. With `wG > 0`, each boid steers toward `p − sdf(p.xy) · gradient(sdf)` (the nearest edge point), with z pulled toward 0. Also lower `vMin` to 3 u/s so the birds can hold the outline.
3. Ramp `wG` up over 1 bar to 3, hold, then cut to 0 on the drop. Add a turn wave at the same moment so the logo shatters outward.

**6. Rendering density, not dots**
1. Draw each boid as a 3-vertex wedge pointing along `v`, stretched by speed (length 0.4–1.2 u). At small sizes use a 1–2 px point sprite instead. Wedges show heading, which makes the turning waves readable.
2. Render into a float target (RGBA16F) with additive blending: R += coverage, G += hit timer, B += depth × coverage (normalise later by R). Additive blending is what turns 100 k tiny marks into a smoky volume.
3. Tonemap `d = 1 − exp(−R · exposure)` (exposure default 0.6), take depth `z = B / max(R, ε)`, and pack them for Spout with a black floor of 0.02.
4. Motion trails: prefer the speed-stretched wedge over screen feedback. If you want trails, use a feedback mix of at most 0.6 in the sim host before tonemapping. More than that turns the flock into fog.

## Shaders

Generic pass order per frame, inside the sim host. The tool-specific mapping is in Build recipes. Run S1–S5 once per substep, then R1–R2 once.

**S1 — Grid clear** (method A only)
- Purpose: zero `cellCount`.
- Inputs: none. One thread per cell (about 6 k cells).
- Output: `cellCount` buffer.

**S2 — Bin**
- Purpose: count boids per cell.
- Inputs: state A, `gridMin`, `rMax`, `gridDim`.
- Per boid: compute the cell index, `atomicAdd` on the cell count, store the slot.
- Output: `cellCount`, per-boid `cell` and `slot`.

**S3 — Prefix sum**
- Purpose: turn counts into start offsets.
- Inputs: `cellCount`. With fewer than about 64 k cells, one workgroup doing a two-level scan is enough.
- Output: `cellStart`.

**S4 — Scatter**
- Inputs: per-boid cell and slot, `cellStart`.
- Per boid: `sorted[cellStart[cell] + slot] = id`.
- Output: `sorted` index buffer (N × uint).

**S5 — Steer and integrate** (the main pass; methods B and C replace S1–S4 with their own lookup)
- Inputs: state A, `sorted`, `cellStart`, `cellCount`, uniforms `k`, `rMax`, `rSep`, `fov`, `wS`, `wA`, `wC`, `wP`, `wN`, `wG`, `vMin`, `vMax`, `maxForce`, `predPos`, `fleeRadius`, `waveCentre`, `waveRadius`, `centre`, `R`, `time`, `dt`, and `goalSDF` (texture).
- Per boid: (1) gather candidates from 27 cells, up to `maxCandidates`, and keep the k nearest inside the fov; (2) compute sep, ali and coh; (3) add flee, bound, noise, goal and wave; (4) clamp the force, integrate, clamp speed; (5) update the hit timer.
- Output: state B (N × 2 vec4). Swap A and B afterwards.

**R1 — Draw**
- Inputs: state (current), camera (perspective, 35–50 mm equivalent, slow orbit), `wedgeScale`, `pointSize`.
- Per instance: the vertex stage reads `p` and `v` by instance id, builds a wedge along `v`, and projects it. The fragment stage outputs `(coverage, hit, depth01 · coverage, 0)` with additive blending.
- Output: density RT, RGBA16F, at output resolution (1920×1080) or 0.75×.

**R2 — Grade and pack**
- Inputs: density RT, `exposure`, `flash`.
- Per pixel: `d = 1 − exp(−R · exposure · (1 + flash))`, `g = 1 − exp(−G · 2)`, `z = B / max(R, 1e-4)`, black floor on d, output `(d, g, z)`.
- Output: RGBA8 1920×1080 → Spout `Flock` / NDI.

**C1 — Colourize** (in Resolume, Wire/ISF)
- Inputs: the Spout texture, colours `near`, `mid`, `far`, `accent`, `skyMode` (bool).
- Per pixel: `tint = mix(near, far, z)`, `c = tint · ramp(d)`, `c = mix(c, accent, g · 0.7)`. In Dusk mode, output `1 − d · 0.92` as a Multiply matte (dark birds).
- Output: composition resolution.

**W0 — Wire fallback flock** (one multipass ISF, design proposal, untested)
- Buffer STATE: persistent, float, 32×32 (1,024 boids). Pixels 0–1023 hold `p.xy` and `v.xy` (2D only).
- Buffer KNN: persistent, float, 32×32. Holds 4 cached neighbour indices (method C with k = 4).
- Pass 1 updates KNN, pass 2 steers STATE. Pass 3 is a gather render: each output pixel at 640×360 loops over all 1,024 boids and adds `exp(−dist² / σ²)`. That is about 236 M distance tests per frame (estimate), so it stays at this size.
- Uses the `PASSES`, `TARGET`, `PERSISTENT`, `FLOAT` and `WIDTH`/`HEIGHT` keys from the [ISF multipass spec](https://docs.isf.video/ref_multipass.html). Resolume's ISF page does not mention multipass or persistent buffers, so test whether your Wire version keeps buffer content across frames.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base / black | #000000 | Night background, ramp at d = 0 |
| far / shadow | #2E3644 | birds at z = 1, thin far areas |
| mid | #8D99AC | typical density |
| near / highlight | #EDE6D6 | dense, close parts of the cloud (warm cream) |
| accent | #FF6A3D | turn-wave front and predator zone (G channel) |
| Dusk sky | #141830 → #4A3557 → #B8704A | vertical gradient, top to horizon |
| Dusk birds | #0A0A0E | Multiply matte colour |
| species 2 / 3 | #6FD3FF / #FFC94A | multi-species variation, one tint per flock |

- Mapping: a 3-stop ramp on tonemapped density `d` (0 → base, 0.35 → mid, 1 → near). This is multiplied by a depth tint that blends `near` → `far` over z. Depth tint gives the cloud volume without lighting.
- Over time: the palette stays mostly fixed, since a murmuration reads best as one colour. Each phrase (16 bars), drift the depth tint's far colour ±15° in hue. The accent appears only with events.
- Music: the kick `flash` raises exposure +25 % for 200 ms. The drop's turn wave paints the accent across the cloud for 0.4 s per bird.
- LED walls: true black below d = 0.02 (floor in R2). Cap near/highlight at about 90 %. A dense flock filling half the wall in cream is bright but still far from full-white. In Dusk mode, keep the sky at 55 % or less. A bright sky on a large wall draws far more power than the birds and flattens the contrast.

## Blending and compositing

- Night: the flock layer goes over black in Normal. It is a complete image already.
- Over video or a UE scene: Add (the flock as light) or Screen (softer). Avoid Normal; the black gaps hide the video.
- Dusk / silhouette: the sky gradient goes on layer 1, and layer 4 (C1 in Dusk mode) in Multiply. The birds then darken whatever is underneath, which also works over live camera.
- Matte: video on layer 2, the flock in Multiply in Night mode. The video shows only through the birds, which suits logo footage.
- Mask source: send the R channel to other layers as a luma mask, e.g. in a Wire patch, to reveal one clip through the flock and another outside it.
- Feedback: none in Resolume by default. If the sim host adds trails, it stays at a 0.6 mix at most (Technique 6).
- Post chain in order: black floor (R2) → colourize (C1) → Bloom (threshold 0.75, intensity 0.6, small radius) → grain 1 % monochrome. Grain goes last to hide 8-bit banding in the far-grey haze after Spout.

## Motion and timing

- Base: birds fly at 8–16 u/s in a cloud about 100 u across, so a bird crosses the flock in roughly 6–12 s. The flock's centre of mass drifts slowly. The camera orbits it once per 32 bars at a 40–60 u distance, on a sine path, with a slight dolly of ±10 %.
- Beat: the kick "breath" (`wC` ×2.5, release 250 ms). Hi-hats add a heading jitter of ±4° on 10 % of the boids for one tick, a shimmer (from the research).
- Bar: the predator dives on the downbeat every 1, 2 or 4 bars (pad select). Between dives it orbits on its Lissajous path, so the flock keeps reshaping.
- Phrase (8/16 bars): change `k` between 4 (loose, grainy swarm) and 12 (tight ribbon) with a 2-bar glide. Optionally swap the camera side.
- Build-up (last 8 bars): `vMax` 16 → 22, `wS` 1.6 → 1.0 (denser), `k` 7 → 14, and predator dives every bar, then every half bar in the last 2 bars.
- Drop: a turn wave from the centre with the accent flash, `wS` jumps to 3 for 1 beat (the cloud bursts open), then everything glides back to defaults over 2 bars. Or, if the goal field was built up, release the logo here.
- Break: `vMax` 10, `wN` 2 (wandering), predator parked far away, `k` 4, camera pulls back 30 %. The flock thins into a loose, drifting veil. For a full hold, set `dt` to 0 (time freeze) and keep the camera orbit running, so the frozen cloud turns in 3D.
- Easing: parameter glides use ease-in-out over beats or bars from Ableton Link. Predator dives use ease-in-out over exactly 1 beat.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| tightness (`wC`) | 0–3 | 0.9 | loose veil → tight ball; kick multiplies it ×2.5 | BeatStep Pro knob 1; kick envelope |
| spacing (`wS`, scales `rSep` with it) | 0.5–4 | 1.6 | dense smoke → grainy, airy cloud | knob 2; bass adds +0.8 |
| neighbours (`k`) | 2–16 | 7 | swarm-like noise (low) → long coherent ribbons and waves (high) | knob 3; 2-bar glides per phrase |
| speed (`vMax`, `vMin` = 0.5 × vMax) | 0–24 u/s | 16 | calm drift → fast, nervous flock; 0 = freeze | knob 4; build-up ramp |
| predator | 0–1 | 0.5 | flee strength and radius together (`wP` 0–8, `fleeRadius` 6–30 u); 0 = no predator | knob 5; dive on bar downbeat |
| shape (`wG`) | 0–3 | 0 | pulls the flock into the logo | KeyStep 37 mod strip |

Pads: BeatStep Pro pad 1 = predator dive now, pad 2 = turn wave, pad 3 = burst (`wS` ×3 for 1 beat), pad 4 = reseed (sphere), pads 5–7 = dive every 1/2/4 bars, pad 8 = Night/Dusk toggle (C1 `skyMode`).

## Build recipes

### Resolume Arena

- The sim host sends the packed R2 output as a Spout sender `Flock`, 1920×1080. In Arena, drag it from the Sources browser (Spout) into a clip on layer 4. For a second machine, use NDI instead and expect one or two frames of latency. That is fine for this slow effect.
- Layer 1: a Solid Color source in #000000 (Night). For Dusk, use a gradient clip or a tiny ISF generator in Wire (three colour inputs).
- Colourize: C1 as a Wire effect on the layer 4 clip (next section). Its five `color` inputs and the `skyMode` bool appear in Arena's effect panel. Resolume's ISF inputs include float, bool, color and event ([Resolume ISF](https://resolume.com/support/en/isf)).
- After C1: Arena's Bloom (threshold 0.75, intensity 0.6). Layer 4 blend mode: Normal (Night), Multiply (Dusk/matte), Add (over video).
- Goal field: put a white logo on black in a dedicated layer, route it through Advanced Output to a Spout output `FlockGoal`, and keep it out of the main output slices. The sim host reads it, turns it into an SDF and uses it in Technique 5.
- Control: the flock parameters live in the sim host, so map the BeatStep Pro there. Arena handles opacity, colour and bloom. Share tempo with Ableton Link so predator dives land on Arena's bars.

### Resolume Wire

- Colourize patch: effect texture input → ISF node with C1 (load it under Attributes → Fragment Shader, per [Resolume ISF](https://resolume.com/support/en/isf)) → effect output. Expose `near`, `mid`, `far`, `accent` (color), `skyMode` (bool) and `exposureTrim` (float 0.5–2) as patch parameters.
- Mask patch (optional): two texture inputs (clip A, clip B) plus the flock texture → ISF node with `out = mix(B, A, smoothstep(0.05, 0.3, flock.r))` → output.
- A full flock in Wire is not practical. Wire (7.22) has no vertex shaders, so boids cannot be drawn as instanced geometry. Fragment passes cannot scatter, so a neighbour grid cannot be built either. The only in-Wire option is W0: about 1,024 2D boids (estimate) with a kNN cache and a gather render at 640×360. It works as a sketch or a backup if the sim machine fails, but not as the main visual. It depends on persistent float buffers from the [ISF multipass spec](https://docs.isf.video/ref_multipass.html), which Resolume does not document for Wire, so test it first.

### Unreal Engine 5.7

- Niagara system `NS_Murmuration`, one emitter, Sim Target GPUCompute, fixed bounds (a sphere of radius 80 u scaled to about 8 m), local space off.
- How agent state persists: Emitter Spawn uses Spawn Burst Instantaneous with Count = user parameter `BoidCount` (default 150,000, estimate). Particle Spawn sets `Position` in a sphere and `Velocity` = common direction + noise. No Lifetime kill (set Kill Particles off / infinite lifetime), so particles live until the system resets. Particle attributes are Niagara's persistent per-agent storage.
- How the neighbour search runs: an emitter-level Neighbor Grid 3D data interface (`MaxNeighborsPerCell` e.g. 16, World BBox Size = bound diameter, cells = diameter / `rMax` ≈ 18 per axis; [Epic API](https://dev.epicgames.com/documentation/unreal-engine/API/Plugins/Niagara/UNiagaraDataInterfaceNeighborGri-)). Simulation stages, in order:
  1. `ClearGrid`: iteration source Data Interface (the grid). Reset the per-cell counts.
  2. `PopulateGrid`: iteration source Particles. Each particle computes its cell and writes its execution index into the next free slot of that cell (custom HLSL with the grid's set/count functions).
  3. `Steer`: iteration source Particles. For the 27 cells around the particle, read the stored indices. Fetch the neighbours' `Position` and `Velocity` by index through a Particle Attribute Reader data interface pointing at the same emitter, then run S5. Write `Velocity` and `Position` directly (no Solve Forces and Velocity module, which would add its own drag).
  The grid's HLSL function names have changed between engine versions. Check the Neighbor Grid 3D function list in the 5.7 Niagara HLSL node before writing the module. Girardot's Boids 2.0 for UE 5.3 shows this data interface works for boids ([80.lv](https://80.lv/articles/check-out-a-new-niagara-based-boids-system-set-up-in-ue5-3)).
- Render: a Mesh Renderer with a 3-triangle wedge mesh (or a Sprite Renderer with velocity-aligned facing at small sizes). Material: Unlit, Additive, emissive = `(1, hitTimer, depth01)` × `Opacity`. Render this in a scene capture or a dedicated camera with a black background into a RGBA16F render target, then post-process it to R2 (a post-process material that tonemaps and packs).
- Output: send the packed render target with your Spout or NDI plugin. Or render the flock in-scene with lighting and a sky for a "real" Dusk look.
- Remote Control: a Blueprint actor `BP_FlockControl` with floats `Tightness`, `Spacing`, `Neighbours`, `Speed`, `Predator`, `Shape` and functions `Dive`, `TurnWave`, `Burst`, `Reseed`. Expose them in a Remote Control Preset and bind OSC and MIDI (BeatStep Pro). On change, call Set Niagara Variable (Float/Vector) for `User.*` parameters. The predator path and dive timing run in the Blueprint from the Link or OSC beat clock and are passed in as `User.PredatorPos`.
- Boid count (estimate): 50 k–300 k on a current desktop GPU, limited by `MaxNeighborsPerCell` × 27 reads per particle. Hodgin reports 40,000 GPU boids in his earlier work and over a million in a friend's compute-shader port ([Hodgin](https://roberthodgin.com/project/murmuration)), so the upper end is plausible with lean attributes. Profile with `stat GPU`.

### TouchDesigner (recommended sim host on the Windows VJ machine)

**Route 1: POPs (GPU point operators).** POPs run on the GPU, loops are built with the Feedback POP, and custom compute shaders run in the GLSL POP ([Derivative, Learning About POPs](https://docs.derivative.ca/Learning_About_POPs)).
- How state persists: Feedback POP (Target = the last POP of the loop) holds `P` and a custom `v` attribute from frame to frame ([Feedback POP](https://docs.derivative.ca/Feedback_POP)). Its input is an initial point cloud (sphere scatter); Initialize reseeds it.
- How the neighbour search runs: Neighbor POP after the Feedback POP. Neighbors Type By Distance, Max Neighbors Distance = `rMax`, Max Neighbors = `k` (7–16), Num Hash Buckets ≈ point count. Use Distribution Default for speed. Closest gives a true sorted k-nearest list but is the slowest mode ([Neighbor POP](https://docs.derivative.ca/Neighbor_POP)). It writes `Nebr` (indices) and `NumNebrs`.
- Steering: GLSL POP reads `Nebr`, fetches neighbour `P` and `v` by index, and runs S5 (minus the grid). This is the loop's Target.
- Render: Geometry COMP instancing the POP points (wedge SOP or point sprites), a Constant MAT with additive blending, a Render TOP 1920×1080 in 16-bit float, a GLSL TOP for R2, then Syphon Spout Out TOP `Flock` or NDI Out TOP.
- Count (estimate): 50 k–200 k at 60 fps. The Neighbor POP is the main cost; check it with the Performance Monitor.

**Route 2: GLSL TOPs (works in older TD builds).**
- How state persists: GLSL TOP `boids_update`, 512×512 RGBA32F, 2 outputs (pos and vel; one texel = one boid, 262 k max). Its inputs are Feedback TOPs targeting `boids_update`, so each frame reads last frame's state ([Feedback TOP](https://docs.derivative.ca/Feedback_TOP)). Pulse Reset Pulse to reload an initial-state GLSL TOP.
- How the neighbour search runs: method C (kNN cache) in a third persistent RGBA32F output (`knn`, 4 indices per texel, two texels per boid for k = 8). No scatter is needed, so it suits TOPs.
- Render: instance from the TOP, as in Route 1.
- Count (estimate): 65 k (256×256) comfortably, 262 k (512×512) on a fast GPU.

### Godot

- Godot 4 with Forward+ or Mobile. Compute shaders only run on the RenderingDevice-based renderers ([Godot compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)). Record the work on the main rendering device via `RenderingServer.get_rendering_device()` inside `RenderingServer.call_on_render_thread`, so that results can be shown through Texture2DRD ([Godot Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html)).
- How state persists: two pairs of RGBA32F storage images, 512×512 (`posA`/`velA`, `posB`/`velB`), created once and swapped every tick. Storage images, unlike storage buffers, can also be wrapped in a Texture2DRD and sampled by the render shader. Grid buffers are plain storage buffers: `cellCount` (uint), `cellStart` (uint) and `sorted` (N × uint). Free all of them with `free_rid()` on exit; the docs note RIDs are not freed automatically.
- How the neighbour search runs: method A as four dispatches (S1–S4) and then S5, local size 256 (S3: one workgroup), with barriers between them. Do not call `sync()`, which blocks the CPU according to the compute docs. Keep each dispatch short to avoid Windows TDR resets, which the same page warns about.
- Render: MultiMeshInstance3D with N wedge instances (a dummy transform per instance). In its spatial shader, `vertex()` reads `p` and `v` from the current state's Texture2DRD at `ivec2(INSTANCE_ID % 512, INSTANCE_ID / 512)` and builds the wedge orientation from `v`. Use `render_mode unshaded, blend_add`. Draw it in a SubViewport (HDR on, black clear) and run R2 as a `canvas_item` shader on a TextureRect showing that viewport.
- Uniforms (push constants): `k`, `r_max`, `r_sep`, `fov_cos`, `w_sep`, `w_ali`, `w_coh`, `w_pred`, `w_noise`, `w_goal`, `v_min`, `v_max`, `max_force`, `pred_pos`, `flee_radius`, `wave_centre`, `wave_radius`, `centre`, `bound_r`, `time`, `dt`.
- Output: Godot has no built-in Spout or NDI output. Run it fullscreen on its own output, or use a community addon.
- Count (estimate): 100 k–500 k. This is the leanest path, closest to the compute-shader port Hodgin mentions.

## Audio reactivity

| band | drives | smoothing | threshold / scaling |
|------|--------|-----------|---------------------|
| kick, 40–120 Hz | breath: `wC` ×2.5; flash +25 % exposure | attack 5 ms, release 250 ms | trigger at 0.6 of a 2 s running peak, retrigger lock 120 ms |
| bass, 60–250 Hz | spacing +0.8 (the flock loosens and breathes with sustained bass) | attack 80 ms, release 600 ms | linear above a 0.2 floor |
| snare / clap, 1–3 kHz transient | small local turn wave at a random boid, radius cap 25 u | trigger only | at 0.5, max one per beat |
| hats, 6–12 kHz | heading jitter ±4° on 10 % of boids | attack 5 ms, release 60 ms | at 0.5 |
| RMS loudness | `vMax` +0–30 % | attack 400 ms, release 3 s | slow: the flock gets faster in loud sections, not on every hit |
| beat clock (Link) | predator path phase, dive timing, `k` glides | — | counted in beats, so it stays on the grid when frames drop |

Keep audio away from `k` and `rMax` on short time scales. Rapid neighbour changes look like jitter, not music.

## Performance tips

- Memory is trivial: 300 k boids × 32 B = 9.6 MB per state copy. The cost is neighbour reads. S5 with 48 candidates is about 48 random reads per boid, so 300 k boids ≈ 14 M reads per substep (estimate). Lower `maxCandidates` before you lower the boid count.
- Dense clumps are the worst case: when the flock contracts on a kick, more boids share a cell and the candidate cap kicks in. Keep the cap. Without it, frame time spikes on every beat.
- Sorting the state by cell index every ~30 frames (or using `sorted` for the read order) improves cache hits. It is optional and worth it above about 200 k boids.
- Draw cost: additive wedges with heavy overdraw in dense areas. Render the density RT at 0.75× output and let bloom hide the upscale. On low-resolution LED walls, render at the native wall resolution and switch to 1 px point sprites. Wedges smaller than a pixel only shimmer.
- Odd aspect ratios: use a perspective camera with the wall's aspect and stretch the bound into an ellipsoid along the long axis (e.g. 3 : 1 : 1 for a 5760×1080 strip). The flock then uses the whole width instead of sitting in the middle.
- Boids count versus look: 30–60 k already reads as a murmuration on a 1080p wall. More boids give a smoother, smokier body, not more detail. Raise the count only if the cloud looks dotty at your viewing distance.
- Spout/NDI: send the 8-bit packed image (R2), never raw density. Add grain in Resolume against banding.
- Pitfalls: in-place state updates (order-dependent jitter); no minimum speed (the flock collapses); a box bound (birds pile up in corners); static weights and no predator (the uniform swirl the research warns about); half-float positions; changing `rMax` without rebuilding the grid size (missed neighbours).

## Further references

- [Craig Reynolds, Boids](https://www.red3d.com/cwr/boids/): the three rules, the distance and angle neighbourhood, and the note on O(n²) versus spatial sorting.
- [three.js GPGPU birds source](https://raw.githubusercontent.com/mrdoob/three.js/dev/examples/webgl_gpgpu_birds.html): a compact texture ping-pong flock (1,024 birds, O(n²) loop, mouse predator, wedge birds built in the vertex shader).
- [NVIDIA GPU Gems 3, ch. 32: Broad-Phase Collision Detection with CUDA](https://developer.nvidia.com/gpugems/gpugems3/part-v-physics-simulation/chapter-32-broad-phase-collision-detection-cuda): the uniform grid, sort and cell-start method behind neighbour-search method A.
- [Derivative, Neighbor POP](https://docs.derivative.ca/Neighbor_POP): the built-in GPU neighbour search for the TouchDesigner route.
