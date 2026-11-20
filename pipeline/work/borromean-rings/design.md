# Borromean Rings — design

## Look

Three thick loops float on true black, slowly turning around their 3-fold axis. Each loop sits in its own plane, and the three planes are at right angles to each other. No two loops are linked, yet the three hold together. The set plays as a story in four scenes, built from the vocabulary of the Gunn and Sullivan IMU logo video ([Bridges 2008 PDF](https://archive.bridgesmathart.org/2008/bridges2008-63.pdf)) and the sculpture families of Tom and Koos Verhoeff ([Bridges 2015](https://archive.bridgesmathart.org/2015/bridges2015-53.pdf)).

1. **Jitterbug.** A thin white polyhedron wireframe breathes from cuboctahedron to icosahedron. Its 12 corners mark out three rectangles, and these thicken into the rings.
2. **Families.** The rings change shape: smooth ellipses ("tight" fat rope), hexagonal mitered beams, square-beam rectangles that sit on a cube grid like a weaving, and a closed three-strand braid.
3. **Glass.** The rings turn into clear glass tubes, each with a glowing thin core and woven rope stripes.
4. **Cut.** On the drop, one ring burns away from a cut point with ember edges. The other two then slide apart, unharmed, which shows the whole idea of the link. On the next phrase they slide back and the ring grows back.

Seen straight down the (1,1,1) axis (the "logo view"), the figure flattens into the familiar three-circle emblem with six over/under crossings. Each crossing gets a brief white spark on the hi-hats. The mood is calm, precise and sculptural, like a lit museum object, with one moment of violence on the drop.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #000000, optional faint radial #06080D at centre (radius 0.7 of the short side) | Normal | 100 % | LED black stays at 0 |
| 2 | Control (hidden) | CPU / Wire / Blueprint: rotation, jitter, family, cut state machine, audio envelopes | — | — | pass P0, data only |
| 3 | Backdrop (optional) | footage or a feedback layer, used only as the refraction source for the glass | Normal | 100 % | the ISF reads it as `inputImage` |
| 4 | Rings | SDF raymarch of 3 tubes (or the braid), solid / beam / glass shading, dissolve, embers, crossing sparks | Normal with alpha (premultiplied) | 100 % | pass P1, RGBA16F |
| 5 | Jitterbug wireframe | 30 projected polyhedron edges + 12 corner dots, screen-space lines | Add | 0–60 % | pass P2, fades out once the rings are fat |
| 6 | Afterimage (optional) | feedback, decay 0.80–0.90, no zoom | Add of the loop | 30–50 % | pass P3, only in slow breaks |
| 7 | Post | Bloom, slight chromatic aberration, grain | — | — | pass P4 / built-in effects |

## Techniques

**1. Geometry from the jitterbug (one shared formula for rings and polyhedron)**
1. Two half-lengths, a (short) and b (long, fixed at 1). Ring 1 is the loop through the corners `(0, ±a, ±b)` in the plane x = 0. Ring 2 goes through `(±b, 0, ±a)` in y = 0, and ring 3 through `(±a, ±b, 0)` in z = 0. These are the cyclic permutations of (0, a, b).
2. The same 12 points are the vertices of Fuller's jitterbug. At a = b they form a cuboctahedron, at a/b = 1/φ ≈ 0.618 a regular icosahedron (the three golden rectangles of the textbook picture, [Wikipedia](https://en.wikipedia.org/wiki/Borromean_rings)), and at a = 0 the points pair up into an octahedron. This is the motion that the Gunn/Sullivan video uses to trace the golden rectangles ([Bridges 2008 PDF](https://archive.bridgesmathart.org/2008/bridges2008-63.pdf)).
3. Stage parameter `jitter` j ∈ [0, 1] sets the shape: `a/b = 1 − 0.764·j` for j ≤ 0.5 (1 → 0.618), then `a/b = 0.618 · (1 − (j − 0.5)/0.5)` for j > 0.5 (0.618 → 0). Rescale all positions by `s = 1 / sqrt(a² + b² + (b − a)²)`. This keeps the jitterbug's triangle edges at constant length, so the polyhedron "breathes" instead of shrinking.
4. The jitterbug's 8 rigid triangles are, per octant (σx, σy, σz): `(0, σy·a, σz·b)`, `(σx·b, 0, σz·a)`, `(σx·a, σy·b, 0)`. The 6 rectangle short sides, of length 2a, complete the 30 icosahedron edges. P2 draws these 30 edges.
5. Clearance between two rings, checked numerically for this design: for rectangles it is `min(a, b − a)`, largest at a/b = 0.5. At a/b = 1 the rings touch and at a = 0 they collapse, so the tube radius goes to 0 at both ends of the jitter range. The rings are "born" from the polyhedron and "die" back into it.

**2. Ring families as one swept profile**
1. Move p into each ring's local frame by the same cyclic permutation. Ring 1: (u, v, n) = (y, z, x). Ring 2: (z, x, y). Ring 3: (x, y, z). Here u runs along the short axis a, v along the long axis b, and n is the plane normal.
2. 2D outline distance, signed: `e = sdProfile(u, v)`. The profile is a weighted mix of three shapes of the same extent: ellipse with semi-axes (a, b), using the cheap estimate f/|∇f| with `f = length((u, v)/(a, b)) − 1` ([iq, ellipse distance](https://iquilezles.org/articles/ellipsedist)); elongated hexagon with vertices (0, ±b) and (±a, ±b/2), after the Verhoeff hexagonal rings; and rectangle ±a × ±b.
3. Tube cross-section: `q = (e, n)`. Distance `d = N(q) − r`, where N is the cross-section norm. Round: `length(q)`. Square beam: `max(|q.x|, |q.y|)`. Hexagonal beam: 2D hexagon SDF on q. Square and hex sections give the mitered corners of the Verhoeff beams for free, because the miter is the bisector of the 2D outline distance. This is the extrusion and revolution idea from [iq's distance functions](https://iquilezles.org/articles/distfunctions), applied along a closed 2D outline instead of a straight line.
4. `family` f ∈ [0, 3]: 0 = ellipse with round section (smooth, "tight" look), 1 = hexagon with hex beam (Verhoeff Trinity II style), 2 = rectangle with square beam at a/b → 0.5 (corners on a half-unit cube grid, our live stand-in for the cube weavings), 3 = braid (technique 3). Between 0 and 2, mix the profile distances and the cross-section norms linearly. This is a cheap SDF morph; it stays collision-free because the radius limit below uses the worst case of the mix.
5. Safe tube radius: `clear = min(ka·a, kd·(b − a))` with (ka, kd) = rectangle (1.0, 1.0), ellipse (0.80, 1.0), hexagon (0.70, 0.78), mixed by the family weights. Then `r = thick · 0.5 · clear`, with thick ∈ [0.05, 0.98]. The constants come from a numeric minimum-distance check of the three families at a/b = 0.5–0.8. At thick ≈ 0.95, ellipse profile and a/b ≈ 0.55 you get the fat "tight rope" look. It is not the true minimal-length tight shape, which has 42 smooth pieces ([Bridges 2008 PDF](https://archive.bridgesmathart.org/2008/bridges2008-63.pdf)) and is not computed live.
6. Honest limit: Verhoeff's Trinity III is a single band of non-convex octagons ([Bridges 2015](https://archive.bridgesmathart.org/2015/bridges2015-53.pdf)). The rectangle mode only suggests it. For the exact sculptures, model them in a DCC from the paper and import them as meshes (UE / Godot only).

**3. Closed three-strand braid (family 3)**
1. Main circle of radius R = 1 in the z = 0 plane. Polar angle `t = atan2(p.y, p.x)`, local cross-section coordinates `(ρr, ρz) = (length(p.xy) − R, p.z)`.
2. Strand k (k = 0, 1, 2) sits at `(ρ·sin(t + 2πk/3), c·ρ·sin(2(t + 2πk/3)))`, with ρ = 0.35 and c = 0.7. This is the classic hair plait (Lissajous 1:2). One period around the circle makes 6 crossings, the braid word (σ1 σ2⁻¹)³, and each strand closes on itself. The result is Borromean rings made of three identical loops with 3-fold symmetry, like the Verhoeff braid sculpture.
3. Distance to strand k ≈ the 2D distance in the (ρr, ρz) plane minus r. This ignores the tilt of the strand, so multiply the march step by 0.6. Numeric minimum distance between strands is 0.37 at these values, so r ≤ 0.17.
4. Transition 2 → 3 is a dithered crossfade (blue-noise threshold over 1 beat), not an SDF morph. Mixing rectangle and braid distances would show tubes passing through each other, and that would break the topology the piece is about. Ring k maps to strand k, so the colours carry over.

**4. Glass tube around a thin core (Gunn/Sullivan vocabulary)**
1. Shell = the tube SDF with radius r. Core = the same profile with radius `r_core = 0.15·r` (range 0.08–0.3).
2. On a shell hit: normal from a 4-tap tetrahedral gradient. Fresnel `F = 0.04 + 0.96·(1 − cosθ)^5`, capped at 0.8 so that the rim does not blow out on LEDs.
3. Reflection: a procedural studio environment of two soft boxes, #FFFFFF at 1.5 above-left and #CFE8FF at 0.6 behind-right, plus a black floor. Refraction: bend the ray with IOR 1.45 and march the core SDFs only (max 32 steps, inside the shell). Core colour is the ring colour × 3 (HDR) for the bloom.
4. Transmission tint `exp(−σ·L)`, where L is the path length inside the shell, σ = 1.5/r, tint #CFE8FF. Behind the tube, sample the backdrop (layer 3) at `uv + refracted.xy · 0.03`.
5. `glass` g ∈ [0, 1] mixes the solid shading (diffuse + spec, rope bump) into the glass shading. At g = 1 only the cores keep the ring colour.

**5. Woven rope surface**
1. Tube angle `ψ = atan2(n, e)`. Arc position along the ring `s = θ · P/(2π)`, with `θ = atan2(v/b, u/a)` and P the ring perimeter (ellipse: Ramanujan approximation).
2. Rope `w = 0.5 + 0.5·cos(3ψ + 2π·s/pitch − scroll)`: 3 strands, pitch = 4·r. `scroll` advances with the mids (Audio reactivity).
3. Displace the SDF by `d −= 0.08·r·w` (step factor 0.8) and darken the grooves by `0.6 + 0.4·w`. Fade the rope out above facet 0.5, because beams stay clean and mitered.

**6. Cut, dissolve and slide (the drop)**
1. The cut ring k gets a cut point θc (random per cut or fixed at the top). Angular distance `δ = |wrap(θ − θc)|` ∈ [0, π].
2. Gate `G = (cut·π + 0.25·noise(θ·6, ψ·2, time) − δ) · 0.5·b`. Carve the ring: `d_k = max(d_k, G)`. With cut 0 → 1 the gap opens from the cut point in both directions until the ring is gone.
3. Ember edge: emissive `#FF6A1A · 4 · exp(−|G|/(0.15·r))` on the remaining surface near the front, plus sparks: hashed points scattered along the front, drifting outward and up over 0.6 s. In UE / Godot, use real particles instead (Build recipes).
4. Slide: once cut ≥ 0.7, ring k+1 (mod 3) moves along its own plane normal by `slide · 2.5·b`, eased out. The third ring moves the opposite way by half of that. A numeric check of this design shows the clearance between the two remaining rings stays at the full static value along the whole path (cut ring 3 → ring 1 slides along x; cut ring 1 → ring 2 along y; cut ring 2 → ring 3 along z). So the separation is physically honest: the two rings were never linked ([Bridges 2008 PDF](https://archive.bridgesmathart.org/2008/bridges2008-63.pdf)).
5. Rejoin = play the slide back first, then cut 1 → 0 so the ring grows back from both fronts and closes at θc. The embers flash white #FFF4E0 for 2 frames at the moment of closure.
6. Solo variant (the end of the video dissolves two components and leaves one): cut two rings in sequence, one beat apart, and the last ring stays alone in the logo view.

**7. Crossing sparks**
1. During the march, track `g += exp(−24 · max(d_i, d_j) / r)` for each ring pair (i, j). This is large only where two tubes come close, which happens at the six crossings.
2. Multiply by the hi-hat envelope and add as #FFFFFF × 2 (HDR). In the logo view this outlines the six over/under points one by one with very little cost.

## Shaders

**P0 — Control** (CPU / Wire logic / Blueprint, not a shader)
- Inputs: BPM, beat position, MIDI faders, audio bands, cut / rejoin triggers.
- Per frame: rotation quaternion (Motion), the a/b chain from jitter, s, clear, r (techniques 1–2), family weights, cut state machine (idle → cutting 1 beat → sliding 2 beats → hold → rejoin), audio envelopes.
- Output uniforms: `rot` (mat3), `a`, `b`, `s`, `r`, `rCore`, `wEllipse`, `wHex`, `wRect`, `braidMix`, `facet`, `glass`, `cut`, `cutRing`, `thetaC`, `slide`, `fov`, `kick`, `hat`, `scroll`, ring colours ×3.

**P1 — Rings raymarch** (fragment, full-screen, RGBA16F, 0.75× of output, upscaled bilinear; 1.0× for 1080p walls on a good GPU)
1. `p = (fragCoord − 0.5·res) / res.y`. Camera on the rotated (1,1,1) axis at distance `D = 1.9 / tan(fov/2)`, field of view `fov`. Ray origin and direction, then multiply both by `rot`ᵀ (rotate the world, not the camera).
2. Bounding sphere of radius `s·sqrt(a² + b²) + r + slide·2.5·b`: if it misses, write alpha 0 and return.
3. March: max 96 steps, hit when `d < 0.0008·t`, step = `d · 0.8` (× 0.6 in braid mode), give up at t > D + 4. Scene `d = min(ring 1, ring 2, ring 3)` (technique 2, with the dissolve gate on ring cutRing) or the braid (technique 3), blended by the dither for braidMix. Keep the index of the nearest ring for colour. Accumulate the crossing spark g (technique 7).
4. Shade: normal; solid = Lambert from key light (−0.4, 0.8, 0.5) + rim + Blinn spec (power 64 for round, 200 for beam); rope (technique 5); glass (technique 4); mix by glass. Ember emission (technique 6).
5. Output premultiplied RGBA. Alpha = 1 on solid hits, `F + 0.6·coreCoverage` for glass, so that the layer works as a Spout/NDI source over other content.

**P2 — Jitterbug wireframe** (fragment, same resolution as P1, or the same pass)
- Inputs: the 12 vertices (P0 computes them on the CPU, rotates them by `rot` and projects them to screen), visible flag per edge.
- Per pixel: distance to 30 screen-space segments, line width 1.5 px at 1080p (scale with res.y), `exp(−d²/w²)`, brightness × depth fade (far edges 40 %). Corner dots: 12 disks of 4 px. Colour #F2F2F2. Opacity `wire = 1 − smoothstep(0.2, 0.6, thick)`, so the wireframe gives way when the rings fatten.

**P3 — Afterimage** (optional, ping-pong pair at 0.5×)
- `out = max(prev · decay, current)`, decay 0.80–0.90, no zoom or rotation. Use it only while the spin is slow, otherwise it smears into a ghost of the 3-fold figure. Off by default.

**P4 — Post** (full output resolution)
- Order: composite P1 + P2 (Add) + P3 → soft clip `x / (1 + x)` on HDR highlights only above 1.0 → Bloom → chromatic aberration 0–1.5 px radial → grain 2 % mono.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base / black | #000000 | background, LED black |
| background lift | #06080D | optional centre glow |
| ring 1 | #E63B2E | vermilion |
| ring 2 | #F5B700 | gold |
| ring 3 | #1E6FD9 | cobalt |
| highlight | #FFF4E0 | core hot spot, spec, rejoin flash |
| glass tint | #CFE8FF | transmission and soft box |
| accent (cut) | #FF6A1A | ember edge and sparks |
| beam metal (mono mode) | #B8BCC4 steel / #A8743A bronze | Verhoeff-style sculpture look |
| wireframe | #F2F2F2 | jitterbug lines |

- Mapping: flat per-ring colours, not a gradient. The topology reads best when each loop has one colour. Mono mode (all rings in steel or bronze with only spec and rim light) is the "sculpture" scene. Glass mode keeps colour only in the cores.
- Over time: ring colours stay fixed within a phrase. Every 8 bars you may rotate the three colours by one slot, in sync with a 120° rotation, so the figure looks like it continues turning while the loop is seamless.
- On the drop the cut ring brightens to its colour × 2 just before it burns. After the slide the two remaining rings drop to 70 % for the hold, which makes the space where the third ring was feel empty.
- LED wall: black level 0, no fog or ambient term (ambient ≤ 0.02). Contrast comes from spec and rim. Keep the HDR peak (cores, embers) ≤ 4 before the soft clip, so that bloom does not wash out the dark gaps between the rings, which carry the over/under reading.

## Blending and compositing

- Rings (layer 4) are Normal with premultiplied alpha. They are a solid object; Add would make the crossings light up wherever tubes overlap on screen, and the over/under reading would be lost.
- Wireframe (layer 5) is Add, because it is light, not matter. Depth-fade it so it reads as behind or through the glass.
- Glass refraction: in the ISF effect version, the layer below comes in as `inputImage` and is bent through the tubes. In the Spout version (UE / Godot), only the alpha edge composites, so use the in-engine backdrop or accept no refraction of Arena content.
- Masks: the ring alpha makes a good matte. Put footage above in Arena with "Mask" or the ring layer as luma key source, so the footage shows only inside the rings. This works best with solid mode.
- Feedback mix (P3): 30–50 %, decay ≤ 0.9. Never during the cut, because the embers turn into streaks.
- Post order: soft clip → Bloom (threshold 0.85, intensity 1.2, radius medium) → chromatic aberration 0.5 px default, 1.5 px for 1 beat on the cut → grain 2 %. No sharpen, because it rings on the thin wireframe.

## Motion and timing

- **Spin**: rotation about the (1,1,1) axis. A 120° turn maps ring 1 → 2 → 3 → 1, so the figure looks the same after each third of a turn ([Bridges 2008 PDF](https://archive.bridgesmathart.org/2008/bridges2008-63.pdf) describes the 3-fold axis). Default: 120° per 2 bars (at 124 BPM, 1.6°/frame at 60 fps). Range: 120° per 1–8 bars. Motion is linear inside the phrase, with no easing, so the loop point is invisible.
- **Tumble**: an extra ±0–20° oscillation about the axis (1, −1, 0)/√2 with a period of 8 bars, sine-eased. At 0 with fov ≤ 10° you get the flat logo view.
- **Jitter**: eases over 1–8 bars with a cubic in-out. Build-up: 0 → 0.5 over 8 bars (cuboctahedron → icosahedron, rings grow out of the wireframe). The outro goes 0.5 → 1 and the rings collapse into the octahedron.
- **Family**: change only on phrase starts, easing over 1 bar (0 ↔ 1 ↔ 2). The braid (3) crossfades over 1 beat on a downbeat.
- **Kick**: tube radius +12 % (still capped at the safe radius), attack 5 ms, release 180 ms. Spin speed does not react, so the 3-fold loop stays locked.
- **Drop**: the cut trigger fires on beat 1. Dissolve 0 → 1 over 1 beat (ease-in), slide over 2 beats (ease-out cubic), chromatic aberration pulse, embers. Hold 4–8 bars with the two rings drifting slowly (they can keep spinning).
- **Rejoin**: on the next phrase start, slide back over 1 bar, regrow over 1 beat, closing flash.
- **Break**: glass 1, logo view (tumble 0, fov 8°), spin 120° per 8 bars, afterimage on at 40 %, cores breathing at 1 cycle per bar.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|------------------------|----------------------------|
| jitter | 0–1 | 0.55 | cuboctahedron → golden icosahedron rings → octahedron collapse; rings are born and die | BeatStep Pro encoder 1, or 8-bar ramp clip |
| family | 0–3 | 0 | ellipse → hex beam → square-beam cube weave → braid | BSP encoder 2, snapped to whole steps on pads 1–4 |
| thick | 0.05–0.98 | 0.8 | tube radius as fraction of the safe radius; thin wire to fat tight rope | KeyStep 37 mod strip, kick adds +12 % |
| glass | 0–1 | 0 | solid colour / metal → clear glass with glowing core | BSP encoder 3 |
| cut | trigger (+ progress 0–1) | 0 | burns one ring away, the other two slide apart; second press rejoins | BSP pad 16 (cut), pad 15 (rejoin), KeyStep note C3/D3/E3 = cut ring 1/2/3 |
| depth | 8°–60° fov | 28° | narrow = flat logo graphic, wide = deep sculptural 3D | BSP encoder 4 |

Secondary (expose, but not on faders): spin bars per 120° (1/2/4/8), tumble, facet (0–1, overrides the family section), rCore, wire opacity, palette slot.

## Build recipes

### Resolume Arena
- Layer 1: Solid Color #000000. Layer 2 (optional): footage or feedback as the backdrop.
- Layer 3: a custom ISF effect "BorromeanRings" containing P1 + P2. As an effect on layer 3 it uses the layer below as `inputImage` for glass refraction. As a source it renders over transparent. Resolume documents ISF loading in connection with Wire ([Resolume ISF](https://resolume.com/support/en/isf)); test it in your Arena version before the gig. Inputs to expose: jitter, family, thick, glass, facet, cut, cutRing (long, 0–2), slide, fov, spinPhase, tumble, kick, hat. Compute the rotation inside the ISF from `spinPhase` (0–1 = 120°) so Arena only needs to send scalars.
- Drive `spinPhase` with the parameter's BPM-sync animation (Loop, duration 2 bars). Drive `cut` and `slide` with Arena's parameter animation in "Play once" on a pad, or from P0 in Wire. Link `kick` to the Audio analysis Low band, `hat` to High.
- Post on layer 3: Bloom ([in Resolume since 7.2](https://resolume.com/blog/19880)) threshold 0.85, then a light grain effect. Afterimage: FeedbackPro (Arena 7.18.2+, [Resolume blog](https://www.resolume.com/blog/23113)) at 30–40 % with scale 100 %, only in breaks.
- External route: the UE or Godot build as a Spout source on layer 3 (Normal, alpha on), and the Arena clip only for post.

### Resolume Wire
- **Control (P0)**: `BPM` / beat-position node → divide by (4 · barsPer120) → fract → spinPhase. Tumble: beat position / 32 → sin → × tumble. Jitter, family, thick, glass, fov from exposed float inputs, each through a `Smooth` (time constant 0.25 s; 0 for family steps).
- **Cut state machine**: trigger in → counter state 0 → 1 → 0. While in state 1: ramp cut 0 → 1 over 1 beat (beat-length ramp), then slide 0 → 1 over 2 beats. On the rejoin trigger, slide 1 → 0, then cut 1 → 0. cutRing from a 3-way selector (exposed int) or from MIDI note number mod 3.
- **Audio**: audio analysis → low band → envelope (attack 5 ms, release 180 ms) → kick; high band → envelope (1 ms / 90 ms) → hat; mid band → integrate → scroll.
- **Render**: ISF node "BorromeanRings" (P1 + P2) → optional feedback node for P3 → Bloom → output. Keep it to one ISF pass; do not rely on multipass ISF across Wire versions.
- **Exposed parameters**: jitter, family, thick, glass, depth, cut (trigger), rejoin (trigger), cutRing, spin, tumble, palette.

### Unreal Engine 5.7
- **Geometry**: one tube mesh per ring, a closed cylinder grid of 384 × 24 segments with UV.x = θ/2π along the ring and UV.y = ψ/2π around the tube. The material does all shaping in World Position Offset. In a Custom node (or material functions), compute the centre curve C(θ) from a, b and the family weights (ellipse/hexagon/rectangle as a polar blend), the frame (tangent T, in-plane normal N, plane normal B), and the offset `r·(cos ψ·N + sin ψ·B)`. Pass the analytic world normal through a Vertex Interpolator into Normal, with "Tangent Space Normal" off. The braid uses a 4th mesh set (3 strands) with its own WPO (technique 3), crossfaded through Opacity Mask dither (DitherTemporalAA).
- **Materials**: `M_RingSolid` (Opaque/Masked, Default Lit, metallic 0–1 for beams, rope as a normal from the analytic stripe), `M_RingGlass` (Translucent, Lighting Mode Surface ForwardShading, Refraction IOR 1.45, opacity from Fresnel 0.04–0.8), `M_RingCore` (Unlit, Additive, emissive ring colour × 3, smaller radius on the same WPO). Dissolve: Opacity Mask = gate G from technique 6 (UV.x drives θ, plus a Noise node), and an ember emissive band where |G| is small.
- **Niagara `NE_CutEmbers`**: GPU sim, spawn rate = 4000 × d(cut)/dt. A Scratch Pad module places particles at θ = θc ± cut·π on the cut ring (same C(θ) formula, read from User parameters), random ψ on the tube surface. Then Curl Noise Force (strength 50, frequency 0.5), Drag 2, Gravity −100, Lifetime 0.6–1.2 s, Sprite Renderer additive #FF6A1A → #FFF4E0 over age.
- **Jitterbug**: a Blueprint computes the 12 vertices each tick and drives an Instanced Static Mesh of 30 thin cylinders (unlit, additive #F2F2F2) plus 12 spheres.
- **Control**: Blueprint actor `BP_Borromean` holds P0. It writes a Material Parameter Collection `MPC_Borromean` (A, B, S, R, RCore, WEllipse, WHex, WRect, Facet, Glass, Cut, CutRing, ThetaC, Slide, Kick, Hat, Scroll) and rotates the parent actor about (1,1,1).
- **Camera**: CineCameraActor on the (1,1,1) axis; depth maps to focal length (about 18–150 mm on a 36 mm sensor, roughly 60°–14°; go orthographic for the pure logo view). Post-process volume: Bloom 1.0 (threshold 1.0), Exposure Manual, Chromatic Aberration 0–0.3, Film Grain 0.1.
- **Remote Control**: a Remote Control Preset with jitter, family, thick, glass, depth, spin, tumble, plus functions TriggerCut(ring) and Rejoin. Bind faders to MIDI (BeatStep Pro encoders as CC) and to OSC (from Arena or Wire, e.g. `/borromean/jitter`).
- **Output**: Spout or NDI into Arena. For alpha over other layers, enable alpha propagation in post-processing in the project settings and test the alpha in your Spout/NDI plugin; otherwise render on black and use Arena's Add or luma key.

### Godot
- **Option A (2D, matches the ISF)**: full-screen `ColorRect` with a `canvas_item` shader that holds P1 + P2. Uniforms: `jitter`, `family`, `thick`, `glass`, `facet`, `cut`, `cut_ring`, `slide`, `fov`, `spin_phase`, `tumble`, `kick`, `hat`, `scroll`, `ring_colors[3]`, and `backdrop` (sampler2D) for refraction. A GDScript on the root computes P0 and sets the shader parameters.
- **Option B (3D, mesh like UE)**: three `MeshInstance3D` with a 384 × 24 tube `ArrayMesh` generated once in GDScript. A spatial shader does the shaping in `vertex()` (VERTEX and NORMAL from the analytic frame). Glass: second material with `hint_screen_texture` for the refraction offset and Fresnel alpha. Cores: a third material with `render_mode unshaded, blend_add`. Embers: `GPUParticles3D` whose emission position the script moves along the dissolve front, with a turbulence-enabled `ParticleProcessMaterial`, lifetime 0.8 s.
- **Feedback (P3)**: a `SubViewport` with "Clear Mode: Never" plus a ping-pong pair of TextureRects. Decay in the shader.
- **Output**: Godot has no built-in Spout or NDI. Use a community extension (check it against your Godot version) or capture the window.

## Audio reactivity

| band | drives | smoothing | threshold / range |
|------|--------|-----------|-------------------|
| sub / kick (40–120 Hz) | tube radius +12 % | attack 5 ms, release 180 ms | onset above −18 dBFS; cap at safe radius |
| bass (60–250 Hz, RMS) | core glow 1.0–2.5× (glass mode) | attack 80 ms, release 600 ms | normalised over the last 8 bars |
| mids (250 Hz–2 kHz) | rope scroll speed 0–2 strand pitches per second | attack 150 ms, release 1 s | continuous |
| highs / hats (6–12 kHz) | crossing sparks | attack 1 ms, release 90 ms | onset gate, minimum 60 ms between hits |
| energy (full-band, 4-bar average) | auto-suggest family step (display only, the VJ confirms) | 4 bars | — |

The cut is never audio-triggered, only by hand or on a cued drop. A false cut in the middle of a phrase breaks the story.

## Performance tips

- P1 cost: 96 steps × 3 rings × (profile mix ≈ 3 SDFs) is about 900 SDF calls per pixel at worst. With the bounding sphere, most of the frame stays empty. Typical cost: 2–4 ms at 1080p on a mid-range GPU at 0.75× scale. Glass adds the 32-step inner core march (+40 %). Use the braid alone (no profile mix).
- Do not evaluate all three profile shapes when a weight is 0. Branch on the weights; they are uniforms, so the branch is coherent.
- The ellipse estimate f/|∇f| is cheap but overestimates at strong eccentricity. Keep a/b ≥ 0.45 when the ellipse weight is high, or reduce the step factor to 0.7.
- 4K or large LED walls: render P1 at 0.5–0.66× and upscale. The rings are smooth, so this hardly shows. Keep P2 lines at full resolution so they stay crisp.
- Odd aspect ratios: frame by the short side (`p / res.y`). On very wide strips (above 4:1), render a square at the strip height and repeat it in Arena with slices or as three instances, each with its spin phase offset by 1/3 (each shows the figure turned by another 40°).
- Pitfalls: (1) a hard 120° colour pop if you rotate colours without also rotating geometry; (2) thick ≥ 0.98 with mixed profiles can make tubes touch for a frame, so keep the cap; (3) glass on pure black has nothing to refract, so give it the soft box environment or a dim backdrop; (4) LED refresh against the fine rope stripes produces moiré, so drop the rope above about 4 px stripe period at the wall's pixel pitch.

## Further references

- [Gunn and Sullivan, "The Borromean Rings: A Video about the New IMU Logo" (Bridges 2008, PDF)](https://archive.bridgesmathart.org/2008/bridges2008-63.pdf): jitterbug, tight rings, glass tubes and the dissolve scene.
- [Tom and Koos Verhoeff, Borromean sculptures (Bridges 2015, PDF)](https://archive.bridgesmathart.org/2015/bridges2015-53.pdf): hexagonal mitered rings, cube weavings, triangular braid.
- [Inigo Quilez, distance functions](https://iquilezles.org/articles/distfunctions) and [ellipse distance](https://iquilezles.org/articles/ellipsedist): SDF primitives, extrusion and revolution, ellipse distance.
- [Wikipedia, Borromean rings](https://en.wikipedia.org/wiki/Borromean_rings): golden-rectangle construction and topology facts.
