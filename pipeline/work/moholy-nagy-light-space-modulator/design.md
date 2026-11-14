# Moholy-Nagy Light-Space Modulator — design

## Look

The audience never sees the machine. They see a big pale wall in a dark room, and over it a moving field of hard shadows and light: grids of round holes that orbit and stretch, comb-like bars that sweep sideways and squeeze shut as the blade turns edge-on, the thin line of a glass spiral that seems to pour outwards, and a ring that swells into an ellipse and folds flat. Because the shadow caster is lit by sixteen small bulbs at once, every part throws sixteen shadows. Where they overlap you get a whole scale of greys, from paper white to deep black. Bright splashes cross the wall whenever a polished blade faces a bulb. Most of the time the picture is black, white and grey, with colour only where an amber, red or blue bulb lights a gap the others cannot reach. This follows the real object: rotating metal and glass parts, more than seventy bulbs, some coloured, throwing light and shadow into the room ([Bauhaus-Archiv](https://www.bauhaus.de/en/discover/objects/lichtrequisit-einer-elektrischen-buehne-reconstruction-of-the-original)). The edit borrows from Moholy-Nagy's own film of the machine, which adds superimposition, lens fragmentation and a short negative sequence ([Coulthart](https://www.johncoulthart.com/feuilleton/2015/06/27/lichtspiel-schwarz-weiss-grau-a-film-by-laszlo-moholy-nagy/)). Our sculpture, the **Lumen Rotor**, is an original form and not a replica: a perforated disc, a slotted blade, a glass spiral, a precessing ring and a cage of rods, each turning at its own rate. The mood is a 1930s laboratory at night: precise, mechanical, hypnotic. It must be big and dark, because the real replicas can feel "precious and underwhelming" when seen as objects ([greg.org](https://greg.org/archive/2010/08/09/so-many-light-space-modulators.html)).

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Room black | flat #050505 | Normal | 100 % | LED black level; the wall only exists where light hits it |
| 2 | Wall light field (Rotor A) | analytic shadow casting: 16 point lights × 5 caster parts onto a wall (or a wall-plus-side-wall corner), soft penumbra from the SDF | Normal over layer 1 (it is the image) | 100 % | hero layer, see Techniques 1–4 |
| 3 | Glints | image-source reflections of the blade and the disc (Technique 5) | Add | 70 % | bright moving patches; rendered in the same pass, kept as a separate output so it can be gated |
| 4 | Glass caustic | focusing line inside the glass spiral's shadow (Technique 6) | Add | 60 % | in the same pass, separate output |
| 5 | Rotor B (superimposition) | a second rotor, counter-rotating, different light ring | Multiply (shadows add up) or Lighten (light adds up) | 0–100 % | film-style double exposure; off in verses |
| 6 | Photogram memory | ping-pong buffer: `print = max(lit, prev · decay)`, decay 0.90–0.995 per frame | Screen over 2–5, tinted | 0–80 % | trails of the light like a photogram print |
| 7 | Lens fragment | 2–4 offset strips of the composite (Technique 8) | Normal (replaces) | 0 / 100 % | cut in on phrase changes only |
| 8 | Negative | `out = 1 − out`, then back to the black level | replaces | flash | snare or a pad, 1/16 note long |
| 9 | Post | film grain, gate weave, vignette, soft bloom on glints only | see compositing | — | order matters |

## Techniques

**1. Wall-space gather shadow casting (the core)**
Each pixel is a point on the wall. For each light we ask: does the ray from this wall point to the light pass through a part, and how much light gets through?
1. Units: frame height = 2, wall at z = 0, x from −aspect to +aspect. The rotor axis is vertical at x = 0, z = zS = 1.2 (in front of the wall). Lights sit at z = zL, between 2.0 and 4.0 (the parameter `throw`).
2. Wall point W = (x, y, 0). In corner mode, pixels left of the fold x < xc lie on a side wall: W = (xc, y, xc − x). This makes the shadows bend across the corner, as they do on the real walls.
3. For light Lᵢ and part k: intersect the ray `W + t·(Lᵢ − W)` with the part's plane (centre Cₖ, normal nₖ): `t = dot(Cₖ − W, nₖ) / dot(Lᵢ − W, nₖ)`. If t is between 0 and 1, the part lies between wall and light. Then P = point on the ray, and the local coordinates are `u = dot(P − Cₖ, tangentₖ)` and `v = dot(P − Cₖ, bitangentₖ)`.
4. Evaluate the part's 2D signed distance d(u, v) (Technique 2). Coverage = `1 − smoothstep(−w, +w, d)`, with the penumbra width w from Technique 3.
5. Transmission for this light: `Tᵢ = Π over parts (1 − opacityₖ · coverageₖ)`. Metal opacity = 1, glass 0.45, plexiglass 0.25. Wall light: `E = Σ Lᵢ.intensity · colourᵢ · stateᵢ · Tᵢ · cosᵢ / (1 + 0.15·|Lᵢ − W|²)`, with cosᵢ = the angle term against the wall normal.
6. Why gather and not real 3D: it is one fragment pass with no shadow maps and no vertex stage. It runs in ISF inside Wire, in a Godot canvas shader and in a UE post material. It also gives exact, soft-edged shadows at any resolution.

**2. The Lumen Rotor parts (2D SDF masks in each part's plane)**
All sizes are in frame-height units (2 = full height).
- **Sieve** (perforated steel disc): radius 0.55. Hex grid of holes with pitch 0.06 and hole radius 0.022, with the outer ring of holes left solid. `d = max(disc, −holes)`. It spins in its own plane (the angle goes into u, v), and the plane is tilted 20° about x so the shadow is an ellipse.
- **Comb** (slotted aluminium blade): 0.35 × 1.10 rectangle with 9 slots of width 0.025, open at one end. It turns around the vertical axis, so its shadow stretches, narrows to a line when edge-on, then widens again. This is the most Moholy-like motion.
- **Helix** (glass spiral): an Archimedean strip `r = 0.08 + 0.05·θ/(2π)` over 4.5 turns, strip width 0.02, distance taken in polar form (the radial distance to the nearest turn). It spins in-plane, so its shadow seems to flow outwards. Glass is half-transparent (opacity 0.45), plus the caustic line from Technique 6.
- **Halo** (ring): annulus r = 0.70–0.74 with 3 spokes. Its tilt axis precesses (tilt 0–60°, cycle of 8 bars), so the shadow breathes between circle, ellipse and line.
- **Cage** (6 vertical rods): radius 0.012, on a circle of radius 0.45 around the axis, orbiting. This is not a plane. Do a 2D ray–circle test in the xz plane per rod; it is exact and cheap. The shadows are vertical bars that sweep and cross.
Bounding test first: if the ray misses a part's bounding circle by more than w, skip its SDF. This roughly halves the cost.

**3. Penumbra from light size (soft shadows without sampling)**
1. Similar triangles: a light of radius ρ gives a penumbra on the wall of width `ρ · (distance part→wall) / (distance light→part)`. Along the ray these distances are t·|L − W| and (1 − t)·|L − W|.
2. Converted into the part's plane, the smoothstep half-width is `w = ρ · t / (1 − t)`, clamped at least 1.5 output pixels (in plane units) against aliasing.
3. The effect: parts near the wall are sharp and parts near the bulbs are blurry. That is exactly the depth cue the real machine has. The `soft` fader scales ρ from 0.002 to 0.06.
4. This is the same idea as the percentage-closer soft shadows that Godot uses for `light_size` ([Godot Light3D](https://docs.godotengine.org/en/stable/classes/class_light3d.html)), but solved analytically: no blocker search, because we know the blocker distance (t) directly.

**4. Bulb field and flicker**
1. 16 lights by default (8–24 usable). Twelve white bulbs sit on a ring of radius 0.9 around the line from the rotor to the viewer, at z = zL. The ring turns slowly (−1 turn per 16 bars). Four coloured bulbs sit outside the ring at ±1.3 x, ±0.6 y.
2. Each bulb has a state from a 16-step pattern (one bit per bulb), sent as one integer bitmask uniform. Decode bit i in the shader with `floor(mod(mask / 2^i, 2))`, because ISF has no array inputs.
3. Each state is an envelope, not a switch: on = attack 10 ms, off = release 80–300 ms (tungsten-like glow-out). The `bulbs` fader sets how many bits may be on.
4. Normalise the intensity: `Lᵢ.intensity = 1.6 / activeCount`. One bulb gives a single hard, deep shadow; sixteen give a soft, layered grey field at the same average brightness.

**5. Glints: reflections by the image-source method**
1. For a mirror part (Comb face, Sieve face), mirror each light across the part's plane: `L' = L − 2·dot(L − C, n)·n`.
2. Intersect the ray from W to L' with the part plane. If the hit lies inside the part (d < 0, holes included), W receives a reflected patch shaped like the part.
3. Add `reflectivity (0.6) · intensityᵢ · colourᵢ · mask · facing`, where facing = `smoothstep(0, 0.3, dot(n, normalize(L − C)))`, so only the side facing the light shines.
4. Cost control: only the 6 brightest active bulbs and 2 mirror parts. The result is bright, sieve-shaped flashes that sweep across the wall once per blade turn, like the reflected light patches thrown by the real machine's polished parts.

**6. Glass caustic line (fake focusing)**
1. The Helix strip is treated as a thin cylindrical lens. Inside its coverage, the transmitted light is redistributed across the strip's width instead of only dimmed.
2. Let s = signed distance from the strip's centre line in units of half its width (−1…1). Focus line width `f = clamp(|1 − D/F|, 0.08, 1)`, where D = the part→wall distance and F = 1.0 (`focus`).
3. Gain inside the strip: `g = 0.35 + 1.4 · exp(−(s/f)²)`. The edges get darker and the centre turns into a bright line, sharpest where D ≈ F.
4. Output it as a separate channel (layer 4), so it can be coloured or gated by the hats.

**7. Photogram memory**
1. Ping-pong buffer: `print = max(wallLight, prev · decay)`.
2. decay 0.90 = short smear; 0.995 = shadows "burn in" for several bars.
3. Show it inverted and tinted (cyanotype or white-on-black) in Screen. Clear it on a pad hit with a 1-frame reset. This is research Variation 5 turned into a stage control.

**8. Lens fragmentation and superimposition**
1. Fragment: split the frame into N = 2–4 vertical (or diagonal at 12°) strips. Each strip samples the composite with its own offset (±4–12 % of width) and scale (0.9–1.15). This works like looking through a cut lens.
2. Superimpose: run Rotor B with the opposite spin, its light ring rotated 37°, and zL 20 % larger, then combine (see compositing). These two moves come from the film's techniques listed by [Coulthart](https://www.johncoulthart.com/feuilleton/2015/06/27/lichtspiel-schwarz-weiss-grau-a-film-by-laszlo-moholy-nagy/). The concrete numbers are ours.

## Shaders

Render order per frame: P1 wall light field (Rotor A, and Rotor B if used) → P2 photogram ping-pong → P3 composite, fragment, negative, post.

**P1 — Wall light field**
- Purpose: everything physical. Shadows, glints and the caustic for one rotor.
- Inputs (uniforms): `t` (accumulated rotor phase, in turns), per-part speed ratios, `throw` (zL), `soft` (ρ), `focus` (F), `tilt` (Halo), `lightMask` (int/float bitmask), `lightEnv` (envelope level, either per-bulb as a 4×4 texture or global), `ringAngle`, `colourAmt`, `corner` (on/off and xc), `aspect`, bulb colour uniforms. For Rotor B, set `mirror = −1` and the phase offsets.
- Per pixel: (1) build W, including the corner fold; (2) loop over the active bulbs: compute the per-part intersection, bounding test, SDF, penumbra and coverage, then multiply Tᵢ and accumulate E; (3) for the 6 strongest bulbs, add glints; (4) add the Helix gain into a separate channel.
- Output: RGBA16F. RGB = wall light (with colour), A = glint + caustic luminance (recoloured in P3). Run it at full output resolution. The penumbra clamp handles aliasing, so no supersampling is needed.
- If both rotors are used: two P1 instances (or one pass that loops twice, at double the cost).

**P2 — Photogram (ping-pong)**
- Inputs: P1 luminance, the previous P2, `decay`, `reset`.
- Per pixel: `out = reset ? 0 : max(lum(P1), prev · decay)`.
- Output: R16F, two buffers swapped each frame. Half resolution is fine, because the trails are soft by nature.

**P3 — Composite and post**
- Inputs: P1 (A and B), P2, `superMode`, `superAmt`, `printAmt`, `fragN`, `fragAmt`, `negative`, palette uniforms, `grain`, `weave`.
- Per pixel: (1) combine A with B: Multiply = `A·mix(1, B, superAmt)`, Lighten = `max(A, B·superAmt)`; (2) map luminance through the grey ramp, but keep the bulbs' own colour where colourAmt > 0 (Colour scheme); (3) add the glints and caustic from the A channel × highlight colour; (4) Screen in the print at printAmt, tinted; (5) fragment: strip index = `floor(uv.x' · fragN)`, then resample with that strip's offset (this needs the result of 1–4 as a texture, so in practice P3a renders the composite and P3b fragments and adds post); (6) negative: `mix(c, 1 − c, negative)`, then remap so black stays at #050505; (7) post.
- Post: grain 3 % monochrome at 24 fps (refreshed every 2nd frame at 50/60 fps), gate weave ±0.6 px random walk, vignette 0.25, bloom only on the glint channel (threshold 0.85, intensity 1.2).
- Output: 8-bit RGBA to Spout/NDI or the LED processor.

## Color scheme

| role | hex | use |
|------|-----|-----|
| room black | #050505 | background, LED black level, negative floor |
| deep shadow | #1C1B19 | umbra where most bulbs are blocked |
| mid grey | #6E6A63 | half-covered zones (the most common tone) |
| wall white | #E9E1D0 | fully lit wall, warm like tungsten on paper |
| glint highlight | #FFF7E6 | reflections and the caustic line |
| amber bulb | #FF9A3C | coloured bulb 1 |
| red bulb | #D8352A | coloured bulb 2 |
| cobalt bulb | #2F5BFF | coloured bulbs 3 and 4 |
| cyanotype print | #0D3A66 → #DCE9F2 | photogram mode ramp |

- **Mapping:** compute the light in linear RGB from the bulb colours (the 12 ring bulbs at #FFE9C8, the four colour bulbs at their own hex). Then use a 3-stop tone curve: 0 → #050505, 0.5 → #6E6A63, 1 → #E9E1D0, and above 1 roll off softly into #FFF7E6. `colourAmt` (0–1) scales the saturation of the four colour bulbs. At 0 they are white like the others, so the picture is pure black-white-grey.
- **Coloured shadows:** when one colour bulb is blocked and the others are not, the shadow takes on the complementary mix of the remaining bulbs. Red and blue bulbs on opposite sides give red shadows on a blue field and the other way round. This is research Variation 3. It needs no extra code: it falls out of Technique 1.
- **Over time:** monochrome for the intro and verses. `colourAmt` rises to 0.6 in builds and 1.0 on the drop, then falls back to 0.2 over 8 bars. The cyanotype print appears only in breaks.
- **LED walls:** hold black at #050505 and cap the wall white at about 88 % before the glints, so the white wall does not pull full power across a huge area. If the house is very dark, lower the overall level to 70 %: a 2 m paper wall is not meant to look like daylight. Contrast between umbra and lit wall should be at least 1:12, so the shadows still read from the back of the room.

## Blending and compositing

- **The wall field is not blended over anything: it is the image.** Room black under it only matters if you scale or fragment the wall.
- **Glints and caustic: Add.** They are extra light on top of lit wall.
- **Superimposition:** use Multiply when you want shadows to pile up (darker, more graphic, "two machines in one room"). Use Lighten when you want the film's double-exposure feel (bright areas win, shadows only stay where both rotors are dark). Do not use Add for two full rotors: the wall white clips.
- **Photogram:** Screen at 0–80 %, tinted cyanotype or plain white. Screen cannot darken, so the live shadows stay readable on top.
- **As a matte for other content:** output P1 luminance alone. In Resolume, put footage or type above it and use the shadow field as its luma mask. Then the footage only appears where light falls, as if it were projected through the rotor. Inverted, it only appears in the shadows.
- **Negative:** a hard 1/16 note flash, never a fade.
- **Post chain, in order:** composite → lens fragment → negative → bloom (glint channel only, threshold 0.85, intensity 1.2) → grain 3 % → gate weave → vignette 0.25. Do not add chromatic aberration: the colour must only come from the bulbs.

## Motion and timing

- **Accumulate the phase:** `t += (speed / barLength) · dt` in the host, never `TIME · speed`, so speed changes and reversals never jump.
- **Base tempo:** the Comb blade does 1 turn per bar (default). Ratios to the Comb: Sieve 0.5 (2 bars per turn), Helix −0.25 (counter-rotating, 4 bars), Halo precession 1/8, Cage 0.75. The ratios are not whole multiples, so the full figure only repeats every 8 bars. That keeps it from feeling looped while the beat-level motion stays locked.
- **Easing:** the Comb's angle can use a beat-wise ease: `θ = bar + 0.25·smoothstep(0, 1, frac(beat))` per beat quarter. It gives a mechanical stepping feel like a ratchet. A `ratchet` amount from 0 to 1 blends from smooth rotation to this.
- **Light pattern:** the 16-step bulb mask advances one step per 1/16 note. Patterns: chase (one bulb around the ring), all-on, alternating halves, random 50 % per step.
- **Kick:** all bulbs on for 1 frame at 1.5× intensity, decaying over 120 ms. The shadows snap into one hard set and then dissolve back into grey layers.
- **Snare:** negative flash, 1/16 note.
- **Build (last 8 bars):** throw shrinks from 3.0 to 2.0, so the shadows grow and get sharper. The bulb chase rate doubles, the colour comes in to 0.6, and the ratchet goes to 1.
- **Drop:** hard cut. Superimposition of Rotor B in Multiply at 100 %, corner fold on, spin doubled for 4 bars, colourAmt 1, and a lens fragment of 3 strips for the first bar only.
- **Break:** a single bulb (bitmask with one bit), soft 0.002, the Sieve alone at slow speed (1 turn per 8 bars), and the photogram on at decay 0.995 so the holes draw slow arcs. Optional total freeze: spin to 0 over 2 beats, and the bulbs keep flickering.
- **Camera:** even in 2D, move it. Do a slow dolly into the wall (scale 1.0 → 1.35 over 16 bars) and drift the light ring position ±0.2 over 32 bars, so the room never feels static.

## Parameters

| name | range | default | what it does on stage | suggested MIDI / BPM mapping |
|------|-------|---------|----------------------|------------------------------|
| spin | −2 to 2 turns/bar (Comb) | 1 | how fast the sculpture turns; negative = reverse | KeyStep knob, BPM-locked; Shift = freeze |
| bulbs | 1–16 active | 12 | one hard black shadow ↔ a soft field of layered greys | fader; BeatStep Pro sequencer sends the step mask |
| throw | 2.0–4.0 (light distance) | 3.0 | near = huge, warped shadows; far = small, crisp, graphic ones | fader, automated in builds |
| soft | 0.002–0.06 (bulb radius) | 0.012 | razor shadows ↔ smoky penumbra | knob |
| colour | 0–1 | 0 | black-white-grey ↔ coloured bulbs and coloured shadows | fader; 1 on the drop |
| print | 0–1 (sets decay 0.90–0.995 and printAmt) | 0 | photogram trails of the light | knob; pad = clear print |

Triggers (not faders): negative (snare pad), fragment (pad, holds for 1 bar), superimpose (toggle), corner (toggle).

## Build recipes

### Resolume Arena

- **Virtual rotor:** P1 to P3 built as a Wire patch (below) and loaded as a source in a layer. Resolume documents ISF only inside Wire, through the ISF node ([Resolume ISF](https://resolume.com/support/en/isf)), so the custom shader belongs there and not in an Arena effect slot.
- **Layer 1:** Solid Color #050505.
- **Layer 2:** the Wire rotor source (Rotor A). Parameters exposed: spin, bulbs, throw, soft, colour, print.
- **Layer 3:** a second instance of the same Wire source with spin inverted and the ring offset (Rotor B), blend mode Multiply or Lighten, opacity = superAmt.
- **Group effects:** Invert RGB driven by a snare trigger (1/16 note) for the negative; Mirror/Transform or a small fragment ISF for the lens split; Colorize for the cyanotype print look on a duplicated layer with Feedback, if you do the photogram in Arena instead of Wire.
- **Shadow as a matte:** put footage or type on a layer above and mask it with the rotor's luminance (a luma-key or mask effect on the footage layer, depending on your version). The footage is only visible where the virtual bulbs reach.
- **No-shader fallback:** render 8- or 16-bar loops of the rotor from Godot or UE at the wall's native resolution (black-white-grey and colour versions). Play two of them on separate layers at different speeds in Multiply: that is superimposition for free.
- **Real-world variant input:** see the dedicated section below. The camera appears in the Sources tab as a live input ([Resolume Live Inputs](https://resolume.com/article/49)).

### Resolume Wire

Node outline. Keep the shader single-pass per rotor: Wire has no vertex shaders and multipass ISF support depends on the Wire version, so do the ping-pong with Wire's own feedback.
- **Clock:** BPM/transport → bar phase. `Time` delta × spin ÷ bar length → an accumulator (an add-to-self loop on a number) → uniform `t`. Check what your Wire version calls these nodes.
- **Bulb sequencer:** beat clock ÷ 4 (1/16 note) → `Counter` (wrap 16) → selects one of 4 stored 16-bit patterns (constants or a lookup) → AND with "first N bits" from the `bulbs` fader → uniform `lightMask` (float holding the integer).
- **Envelopes:** kick trigger → `Envelope` (attack 1 ms, release 120 ms) → `flash`. Snare → a 1/16 gate → `negative`.
- **ISF node "lumen_rotor"** (P1, inputs as in Shaders) → texture A. A second ISF node with `mirror = −1` → texture B. Both share an `ISF Resource` node if they need the same lookup texture.
- **Composite:** A, B → `Blend` (Multiply or Lighten, mix = superAmt) → split: one path into a `Feedback`/delay loop with a `Max` blend against the faded previous frame (decay) for the photogram, the other into the final `Blend` (Screen at printAmt) → fragment ISF (single pass, reads the composite texture) → invert by `negative` → grain/vignette ISF → output.
- **Exposed parameters:** spin, bulbs, throw, soft, colour, print, superAmt, plus toggles corner and superimpose, and the negative and fragment triggers.

### Unreal Engine 5.7

Two routes. Route A gives the most authentic light; Route B is a 1:1 port of the shader.
- **Route A, real geometry and real lights:**
  - Build the Lumen Rotor as Static Meshes on a Blueprint `BP_LumenRotor` with `Rotating Movement` components (or Timeline-driven rotations from the bar phase). The Sieve and Comb use **Masked** materials with an opacity-mask texture for the holes and slots, so shadows are exact. Metal: Metallic 1, Roughness 0.15. For the glass Helix, use a Masked material with dithered opacity (DitherTemporalAA into Opacity Mask at about 0.55), so it casts a partial shadow.
  - Lights: 12–16 **Spot Lights** (not Point Lights) aimed at the rotor, Movable, cast shadows on, Source Radius 0.5–3 cm (the `soft` fader), with an outer cone just big enough to cover the rotor. Spot lights only shadow one cone, which is far cheaper than cube-map point shadows.
  - Room: a white wall plane (and a side wall for the corner mode) with a matte grey-white material (#E9E1D0, roughness 0.9). A **CineCamera** faces the wall, with the rotor out of frame or just a sliver of it at the edge.
  - Glints: do not rely on global illumination to throw mirror reflections onto the wall. Fake them with 2–4 extra Spot Lights placed in Blueprint at the mirrored light positions (L' from Technique 5), aimed through the part, with a **Light Function** material that is the part's mask. Light functions are materials in the Light Function domain that mask a light, and they work on Movable or Stationary lights ([Epic docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/using-light-functions-in-unreal-engine)). The same trick gives the glass caustic line: a Spot Light behind the Helix with a light function drawing the spiral line.
  - Bulb flicker: drive each light's Intensity from the bitmask and envelope in Blueprint.
- **Route B, post-process port:** a Post Process material (Blendable Location: before tonemapping) with a Custom node that runs P1 as written. A second material does P3. The photogram uses a render target written by `Draw Material to Render Target` each frame (two targets swapped).
- **Remote Control:** a preset exposing spin, bulbs, throw, soft, colour, print, superAmt, negative and lightMask (on the Blueprint or a Material Parameter Collection). Remote Control supports OSC, MIDI and DMX through the Remote Control Protocol OSC, MIDI and DMX plugins ([Epic docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-protocols-in-unreal-engine)). Map BeatStep Pro knobs via MIDI, and send beat, kick and snare as OSC from Resolume.
- **Post-process volume:** manual exposure (eye adaptation off, or the kick flash pumps the image), Bloom intensity 0.6, threshold 1.0 (glints only), Film Grain 0.03, Vignette 0.25. Tonemapper: keep it neutral, and grade toward #E9E1D0 whites.
- **Output:** Spout through a third-party plugin, or NDI through the NDI SDK for Unreal, at the LED processor's native resolution.

### Godot

- **2D build (matches the ISF):** a `ColorRect` filling a `SubViewport`, with a `canvas_item` shader for P1 (uniforms: `t, part_ratios (vec4 + float), throw, soft, focus, tilt, light_mask (int), light_env, ring_angle, colour_amt, corner (vec2), aspect, bulb_colors (vec3 array[4])`). Godot shaders support int uniforms and arrays, so you can skip the bitmask trick if you like. A second `SubViewport` with `render_target_clear_mode` set to Never does the photogram max-blend from a `ViewportTexture` of the previous frame. A final ColorRect does P3. Accumulate `t` in GDScript `_process(delta)`.
- **3D build (real shadows):** rotor meshes as `MeshInstance3D` children of `Node3D` pivots rotated in `_process`. Hole patterns use alpha-scissor materials so shadows follow the holes. Use `SpotLight3D` × 12–16 with `shadow_enabled` and `light_size` for soft penumbras. Godot handles these with PCSS, which costs noticeably more above 0 ([Godot Light3D](https://docs.godotengine.org/en/stable/classes/class_light3d.html)), so keep `light_size` at 0 on most bulbs and soft only on 2–4 of them. Raise the positional shadow atlas size in Project Settings for 16 shadowed lights.
- **Glass caustic and glints in 3D:** a `SpotLight3D` with a `light_projector` texture (the spiral line, or a ViewportTexture of the 2D caustic). Projectors need `shadow_enabled` and only work in Forward+ and Mobile ([Godot Light3D](https://docs.godotengine.org/en/stable/classes/class_light3d.html)).
- **Post:** `WorldEnvironment` glow with HDR threshold 1.0 (glints only), plus the grain, weave and negative in a final canvas shader.
- **Control:** `InputEventMIDI` → uniforms or light energies. Spout via a community addon if one exists for your version, otherwise NDI.

### Real-world variant: physical rotor, lights, camera

The research's key point taken literally: build a small, cheap object, light it hard, film the wall, not the object.
- **Object (30–60 cm tall):** a vertical rod on a slow motor (a mirror-ball motor or a geared turntable around 1–5 rpm). Hang on it: a disc of perforated steel sheet (holes 3–6 mm), a blade of aluminium sheet with sawn slots, a laser-cut clear acrylic spiral (3–5 mm thick, it gives the glass line and caustic), a wire ring on a pivot, and 4–6 thin rods or knitting needles as the cage. A polished face on the blade gives the glints. Mount the parts at different angles, so a turn changes their shadow shape and not just its position.
- **Lights:** 6–16 small, point-like LED sources (bare LEDs or small torches with the reflector removed). Size matters more than power: penumbra on the wall ≈ source size × (object→wall) / (light→object). A 5 mm source 0.5 m from the object, with the wall 2 m behind it, gives about 20 mm of penumbra: sharp enough. Spread the sources over a 1 m wide arc, so each throws its own copy of the shadow. Add 2–4 coloured LEDs (amber, red, blue) to get coloured shadows. Switch them in steps from a DMX LED controller or a microcontroller clocked by the BeatStep Pro, to get the bulb sequencer.
- **Screen:** a matte white wall, or a large paper or fabric sheet 1.5–3 m wide. With rear-projection fabric, put the rig behind it and the camera in front. Cover the rest of the room in black.
- **Camera:** a camera or webcam aimed at the screen only, locked off. Use manual exposure, manual focus and fixed white balance. Set the shutter to a multiple of the mains period (1/50 or 1/100 s on 50 Hz mains), so dimmed or PWM LEDs do not roll bars through the image. Expose so that the fully lit wall sits just below clipping.
- **Into Resolume:** a USB webcam appears automatically in the Sources tab. HDMI or SDI cameras need a capture device; Resolume supports Blackmagic, Datapath and AJA natively, and other devices through DirectShow (Windows) or AV Foundation (Mac). Expect about 60–120 ms of latency ([Resolume Live Inputs](https://resolume.com/article/49)). Delay the beat triggers you send to the lights, or accept that the physical flicker trails the audio slightly.
- **Processing in Arena:** live input layer → Levels/contrast to crush it into black-white-grey (lift the black to #050505) → optional Invert on snare → a duplicate with Feedback for the photogram → used as a luma mask for other layers, or Multiplied with the virtual Rotor A. Physical shadows times virtual shadows looks best of all: real dust and wobble, virtual precision.
- **Safety:** LEDs only, no hot bulbs near paper. Secure the rotating parts and keep the motor speed low, because thin metal blades at the edge of a turning rod cut.

## Audio reactivity

| band | drives | smoothing | threshold |
|------|--------|-----------|-----------|
| kick, 40–100 Hz onset | all bulbs flash on (1.5×), shadows snap hard | attack 1 ms, release 120 ms | onset +6 dB over the 1 s average; at most one per 1/8 note |
| bass, 60–250 Hz level | intensity of the 4 coloured bulbs (0.3–1.2×) | attack 50 ms, release 400 ms | none, continuous; only when colour > 0 |
| snare, 150–250 Hz + 2–5 kHz onset | negative flash, 1/16 note | none (gate) | onset +8 dB; can be disabled in verses |
| mids, 300 Hz–2 kHz | Halo tilt ±10° around its programmed value | attack 200 ms, release 1.2 s | none |
| hi-hats, 6–12 kHz | random 25 % of the bulbs off for 1 frame (shutter flicker), caustic line gain 0.8–1.4× | attack 1 ms, release 60 ms | gate at −30 dBFS |

Never let audio drive spin directly: rotation stays tempo-locked, and the audio only changes the light. A machine that jitters with the music stops reading as a machine.

## Performance tips

- **Cost scales with pixels × bulbs × parts.** 16 bulbs × 5 parts, plus 6 × 2 glint tests, is about 90 SDF evaluations per pixel. The bounding test cuts that by roughly half. At 1080p this is light; at 4K, or with two rotors, drop to 8–10 bulbs or render P1 at 0.75 scale. The penumbra keeps edges smooth when upscaled.
- **The Sieve is the expensive part:** the hex hole grid is cheap per evaluation, but it is evaluated per bulb. If you need to save, sample it only for the 8 strongest bulbs and treat it as solid for the rest. On a 16-bulb field nobody sees the difference.
- **Rotation aliasing:** a fine hole grid rotating against the LED pixel grid crawls and moirés. Keep projected holes at least 4 LED pixels wide (raise the hole pitch, or lower throw), and keep the penumbra clamp at least 1.5 px.
- **Big walls:** large white areas pull a lot of power and wash out the room. Keep the average frame level under 45 %: more bulbs blocked, a smaller lit area, darker vignette.
- **Odd aspect ratios:** always build W with the aspect. On very wide strips (32:9 and wider), use two or three rotors side by side with different phase offsets rather than one stretched rotor, and spread the bulb ring horizontally (radius_x = 0.9 · aspect/1.78).
- **UE Route A:** each shadowed Movable spot light costs shadow-map time. 16 is a lot: use Virtual Shadow Maps, keep light cones tight, and turn shadows off on the colour bulbs if needed (they are fill). Keep the rotor small in world space so the shadow resolution concentrates on it.
- **Pitfalls:** `TIME · speed` jumps when the fader moves; auto-exposure pumping on the kick flash; intensity not normalised by active bulb count (the wall doubles in brightness when the chase opens up); the photogram never cleared (the frame turns white after a few minutes at decay 0.995).

## Further references

- [Bauhaus-Archiv: Lichtrequisit einer elektrischen Bühne (reconstruction)](https://www.bauhaus.de/en/discover/objects/lichtrequisit-einer-elektrischen-buehne-reconstruction-of-the-original): parts, materials, more than seventy bulbs, and a photo of the shadow on the wall. This is the spec our original form departs from.
- [Coulthart: Lichtspiel Schwarz-Weiss-Grau](https://www.johncoulthart.com/feuilleton/2015/06/27/lichtspiel-schwarz-weiss-grau-a-film-by-laszlo-moholy-nagy/): the 1930 film and its superimposition, lens fragmentation and negative, the source for our edit language.
- [Godot Light3D class reference](https://docs.godotengine.org/en/stable/classes/class_light3d.html): `light_size` (PCSS soft shadows and their cost) and `light_projector` (projected textures), the two properties the 3D build relies on.
- [Unreal Engine: Using Light Functions](https://dev.epicgames.com/documentation/en-us/unreal-engine/using-light-functions-in-unreal-engine): the Light Function domain used for the fake glints and the glass caustic line.
