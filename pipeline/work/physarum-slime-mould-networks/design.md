# Physarum Slime Mould Networks — design

## Look

A black wall slowly fills with glowing veins. Thin filaments branch, merge and close into loops, so the image moves between fine fibre fields, honeycomb cells, fingerprint swirls and dotted colonies. Nothing cuts: the network grows, thickens where many agents agree and fades where they leave, like a time-lapse of the living mould growing toward food ([Wikipedia](https://en.wikipedia.org/wiki/Physarum_polycephalum)). Up close, a cloud of agents seems to crawl along the veins. On every kick a small share of agents loses its way, the network tears and then heals within the beat. On the drop a burst from the centre throws the whole structure outward, and it re-forms as a new topology. The default colour is bioluminescent: black floor, deep teal for weak trails, green for veins and warm cream for the densest junctions. An optional magenta "contagion" spreads through the network on the beat. The mood is organic, calm and alive, with the music audible in how the structure behaves rather than as flashing.

## Layer stack

The agent simulation runs in a sim host (TouchDesigner, Unreal Engine 5.7 or Godot) and comes into Resolume over Spout or NDI. Resolume handles colour, compositing and post. Wire cannot run the full sim (no vertex shaders, see Build recipes), so it only gets an optional fallback.

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | solid #000000 | Normal | 100 % | LED black |
| 2 | Video bed (optional) | clip or UE scene shown only inside the veins | Normal | 100 % | used with layer 4 in Multiply mode as a matte |
| 3 | Agent state | per-agent position, heading, flags; lives in the sim host (texture, particle buffer or storage buffer) | not visible | — | 0.25–4 M agents, see Techniques 1 |
| 4 | Trail map (sim output) | float trail field, tonemapped and packed: R = trail, G = contagion trail, B = agent density | Normal (or Multiply over 2 for matte mode) | 100 % | arrives as a Spout/NDI source, 1280×720 |
| 5 | Colourize | 5-stop gradient ramp on R, contagion colour on G, sparkle on B | inside layer 4 (Wire/ISF effect) | — | see Color scheme |
| 6 | Inject / seed (optional) | 1000a logo, spectrum spots or a hand-drawn mask sent back to the sim host | not visible here (Resolume Spout output → sim) | — | agents grow toward or around it; see Techniques 4 |
| 7 | Post | black floor, bloom, grain | effects on layer 4 | — | bloom threshold 0.7, grain 1.5 % |

## Techniques

**1. Agents (sense, rotate, move), after Jones 2010 as described by Jenson ([Sage Jenson](https://cargocollective.com/sagejenson/physarum))**
1. Each agent stores position `p` (float, trail-pixel units) and heading `a` (radians). Optional: species index, infected flag, infection timer.
2. Sense: sample the trail at three points, `sF = trail(p + sd·dir(a))`, `sL = trail(p + sd·dir(a + sa))`, `sR = trail(p + sd·dir(a − sa))`, with `dir(a) = (cos a, sin a)`, `sd` = sensor distance, `sa` = sensor angle. Bilinear sampling is enough; Jenson's "sensor size" (a larger sensor footprint) can be added by sampling a blurred mip, but is off by default.
3. Rotate: if `sF ≥ sL` and `sF ≥ sR` keep heading; else if `sL > sR` then `a += ra`; else if `sR > sL` then `a −= ra`; on a tie (`sL == sR` above `sF`) turn by `±ra` chosen by a coin flip. `ra` = rotation angle. Ghassaei's demo also flips the turn direction randomly per step ([gpu-io source](https://raw.githubusercontent.com/amandaghassaei/gpu-io/main/examples/physarum/index.js)).
4. Move: `p += ss·dir(a)`, `ss` = step size. Wrap `p` around the edges (torus), as Ghassaei's trail texture does with REPEAT wrapping. Wrapping keeps the density even and makes wide panoramas seamless.
5. No collision step: Jenson usually skips Jones's one-agent-per-cell rule because it looks better and lets every agent run in parallel ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)). Only the Wire fallback (Technique 5) uses it.
6. Randomness: use an integer hash (PCG or similar) of `(agentId, frame)`, not `fract(sin(...))`, which repeats visibly at millions of agents.

**2. Trail map (deposit, diffuse, decay)**
1. Each tick every agent adds `dep` at its cell. Two deposit styles:
   - Count deposit: additive (point sprites with Add blending, or atomic add). Dense veins get brighter. Best look.
   - Presence deposit: the cell is set to `dep` (plain write, no add). Racing writes all write the same value, so no atomics are needed. Use it where only plain writes exist (Niagara Grid2D, compute without atomics).
2. Diffuse: `m = mean of 3×3 neighbourhood` ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)); `t = mix(t, m, diffuse)`.
3. Decay: `t = t · decay`, default 0.90 (Ghassaei's default).
4. Inject: `t += inject(uv) · injectGain` (attractant from logos, audio spots, touch). Negative gain makes a repellent.
5. Store as float (R16F is enough for display; R32F if the same texture is also an atomic target). The values are not normalised: with deposit 4 and decay 0.9, a busy vein sits in the tens, which is why Ghassaei draws it with an amplitude of 0.03.

**3. Tonemap and colour**
1. `v = 1 − exp(−t · exposure)`, exposure default 0.03 (matches Ghassaei's render amplitude), range 0.005–0.2. This compresses dense junctions instead of clipping them.
2. Black floor: `v = max(v − 0.02, 0) / 0.98`, so the diffuse haze does not light the LED wall.
3. Ramp lookup on `v` (Color scheme). The sim host sends `v`, not raw `t`, because Spout and NDI links are usually 8-bit.

**4. Steering the network for the show (design proposals built on the parameter list)**
1. Presets: the five presets in Ghassaei's demo give five clearly different topologies (values below). Jump between them on pads, but unlike the demo do not reset the agents. The live network then morphs from one structure into the next over 1–3 s.

   | preset | sensor distance (px) | sensor angle (°) | step size (px) | rotation angle (°) |
   |--------|---------------------|------------------|----------------|--------------------|
   | Fibers | 18 | 5.5 | 2 | 45 |
   | Fingerprint | 14 | 70 | 1.5 | −25 |
   | Honeycomb | 7.5 | 90 | 2 | −45 |
   | Net | 18 | 90 | 2 | −16 |
   | Dots | 26 | 5.5 | 1.5 | −70 |

   All share decay 0.9, deposit 4, density 0.35 agents per trail pixel ([gpu-io source](https://raw.githubusercontent.com/amandaghassaei/gpu-io/main/examples/physarum/index.js)). Distances are in trail pixels; Ghassaei's trail is screen-sized. Scale them by `k = trailHeight / 720` if the trail is not 720 px high.
2. Scatter (kick): agents with `hash(agentId, beatIndex) < scatter` get a new random heading for that tick. 4 % tears the network visibly; 30 % shatters it.
3. Explosion (drop): every agent within radius `R` (default 0.35 of trail height) of a centre point gets `a = atan2(p − c)` (pointing outward) and a deposit ring is injected at `R`. Inspired by the "explosion" in Jenson's write-up, whose exact rule is not described there.
4. Contagion: infected agents deposit into channel G. A healthy agent whose front sensor reads `G > 0.5 · its R value` becomes infected with probability 0.02 per tick; infection ends after 4 beats. On a beat, infect the ~500 agents nearest a random point. Jenson describes infected particles laying a trail that spreads ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)); the rates here are starting points.
5. Multi-species (variation): up to four populations, each depositing into its own RGBA channel. Each agent senses `dot(trail, w)` with `w = +1` on its own channel and `−0.5` on the others, so the species avoid each other.

**5. Fragment-only fallback (grid agents, for ISF in Wire)**
Fragment shaders cannot scatter, so agents must live in grid cells, one per cell: exactly Jones's collision rule. This is a design proposal, untested, and needs ISF persistent float buffers.
1. Buffer AGENTS (persistent, float, 640×360): `r = occupied (0/1)`, `g = heading`, `b = species/flags`.
2. Pass INTENT: each occupied cell senses TRAIL, turns as in Technique 1, and stores the target neighbour `d = round(a / 45°) mod 8` (one of 8 directions).
3. Pass MOVE: an empty cell scans its 8 neighbours in a fixed order rotated by `FRAMEINDEX mod 8` and accepts the first one whose `d` points at it. An occupied cell checks its own target the same way: if the target is empty and accepts this cell, the cell becomes empty; otherwise the agent stays and picks a random new heading (Jones's blocked-agent rule). Both sides run the same deterministic rule, so no agent is lost or duplicated.
4. Pass TRAIL: 3×3 mean × decay, plus `dep` where AGENTS is occupied.
5. Limits: one cell per tick (slow; run it at 640×360 so a cell is 3 output pixels), about 25 % occupancy = ~58 k agents. It shows coarser, blockier networks than the float-position version.

## Shaders

Generic pass list. Tool-specific storage is in Build recipes. Per frame: run P1–P3 `substeps` times (1–3), then P4 once.

**P0 — Inject (optional, once per frame)**
- Purpose: add attractant or repellent.
- Inputs: trail (current), inject texture (from Resolume over Spout, 1280×720 RGBA8), uniforms `injectGain` (−2..+2), audio spot positions.
- Per pixel: `t += luma(inject) · injectGain · 4`, clamp at 0.
- Output: in place on the trail ping-pong (can be merged into P3).

**P1 — Agent update (per agent, not per pixel)**
- Purpose: sense, rotate, move, scatter, explosion, contagion.
- Inputs: agent state (previous), trail (previous tick), uniforms `sensorDist`, `sensorAngle`, `rotAngle`, `stepSize`, `scatter`, `beatIndex`, `explode` (0/1 trigger), `explodeCentre`, `explodeRadius`, `frame`, `trailSize`.
- Per agent: (1) if `explode` and within radius, set heading outward; (2) if `hash < scatter` on a beat tick, random heading; (3) three sensor reads; (4) turn rule; (5) move and wrap; (6) contagion check against G.
- Output: new agent state (ping-pong or in place in a storage buffer). Resolution: one element per agent.

**P2 — Deposit (per agent, scatter)**
- Purpose: write the agents into a deposit field.
- Inputs: agent state after P1, `dep` (default 4), `depContagion`.
- Per agent: add `dep` (count) or write `dep` (presence) at `floor(p)`; infected agents also into G; every agent adds 1 into B (density, for the sparkle layer).
- Output: deposit texture, trail resolution, cleared every tick.

**P3 — Diffuse and decay (per trail pixel)**
- Inputs: trail (previous), deposit (P2), `decay` (0.80–0.99), `diffuse` (0–1).
- Per pixel: `t = mix(t, mean3x3(t), diffuse) · decay + deposit`; same for G with its own decay (0.95, so contagion lingers longer). B is not diffused; it is copied for the frame.
- Output: trail ping-pong A/B, 1280×720, R16F or RGBA16F (with contagion and species).

**P4 — Grade for transmission (per output pixel)**
- Inputs: trail, `exposure`, `flash` (kick envelope).
- Per pixel: `v = 1 − exp(−t · exposure · (1 + flash))`, black floor, pack `(vR, vG, min(B / 4, 1))`.
- Output: RGBA8 1280×720 → Spout / NDI.

**P5 — Colourize (in Resolume, Wire/ISF)**
- Inputs: the Spout texture, five ramp colours, contagion colour, `sparkle` (0–1).
- Per pixel: `c = ramp(R)`; `c = mix(c, contagionColour, G · 0.8)`; `c += sparkle · B · highlight`.
- Output: composition resolution. Bilinear upscale from 1280×720 is fine; the veins are soft anyway.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base / black | #000000 | ramp at v = 0, background |
| shadow | #06131A | ramp at v = 0.15, faint old trails |
| mid | #0E6E6A | ramp at v = 0.40, settled veins |
| accent | #3FE0A0 | ramp at v = 0.70, active veins |
| highlight | #F5E6A8 | ramp at v = 1.0, dense junctions and agent sparkle |
| contagion | #FF2E88 | G channel overlay |
| species 2 / 3 | #FFB000 / #3A7BFF | multi-species variation, one hue per population |

- Mapping: piecewise-linear 5-stop ramp on the tonemapped trail `v`. Alternative "gold" ramp for warm sets: #000000 → #3A1C00 → #C77700 → #FFD23F → #FFF8E1.
- Over time: rotate the three middle stops 20° in hue per phrase (8 bars). Hold base and highlight fixed so blacks and peaks stay stable.
- Music: the kick `flash` raises exposure for 250 ms, so the veins brighten along the ramp toward cream instead of turning white. Contagion magenta appears only on drops and builds.
- LED walls: everything below v = 0.02 is true black (floor in P4). Cap highlight at about 90 % output; a full-screen Honeycomb preset at full white draws a lot of power and glares. Contrast comes from the black gaps between veins, so do not lift the floor for "more visibility".

## Blending and compositing

- Trail layer over black in Normal: it is already a complete image with a black background.
- Over video or a UE scene, use Add (veins as light on top) or Screen for softer highlights. Never Normal over video, the black gaps would hide it.
- Matte mode: video on layer 2, trail on layer 4 in Multiply. The video then shows only inside the veins, which suits portraits or the 1000a logo footage.
- Displacement (variation): use the trail as a displacement map on video in a Wire patch (offset UV by the trail gradient, 0–20 px), so the image ripples along the veins.
- Sim feedback: the trail itself is the feedback loop (decay 0.80–0.99 in P3). Do not add Resolume feedback or trails effects on top; they smear the crisp filaments into fog.
- Post chain in order: black floor (in P4) → colourize (P5) → Bloom (threshold 0.7, intensity 0.8, small radius) → grain 1.5 % monochrome. Grain last; it hides 8-bit banding in the dark teal gradients after Spout.

## Motion and timing

- Base speed: step size 1–2 trail px per tick, 1 substep per frame at 60 fps, so agents move 60–120 px/s at 720p. Topology changes settle in roughly 1–3 s; persistence (decay) sets how fast old structure fades.
- Slow drift so the network never freezes: sensor angle ±10° around the preset value with a sine LFO, one cycle per 8 bars, linear. The research names a settled network as the main reason this effect gets boring.
- Kick (every beat): scatter 4 %, flash +30 % exposure, attack 5 ms, release 250 ms.
- Snare / clap: optional small explosion, radius 0.1 at a random point.
- Hats: decay dip −0.03 for 80 ms, a shimmer without changing structure (from the research).
- Bar: preset glide target can change on the downbeat; parameters glide over one bar with ease-in-out.
- Build-up (last 8 bars): decay 0.90 → 0.97 (denser, longer trails), step size 2 → 3, substeps 1 → 2, scatter moves from every beat to every 1/8 note.
- Drop: explosion from the centre (radius 0.35), switch to the Net preset, infect 500 agents. Contagion magenta spreads through the new network over the first 4 bars.
- Break: step size 0.3, decay 0.98, scatter off, ramp desaturates 50 %. For a full hold, set step size 0, decay 1.0 and diffuse 0: the network freezes as a still image and comes back alive when the knobs return.
- Phrase (8 or 16 bars): optionally reseed (pad), with agents spawned in a small disc in the centre so the network grows out again (like Jenson's "contagion spread from the centre" caption).

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|-----------------------|----------------------------|
| topology (sensor angle) | 5–120° | 90° (Net) | fine fibres (small) to cells and honeycomb (large) | BeatStep Pro knob 1; LFO ±10° per 8 bars on top |
| scale (sensor distance) | 4–40 px at 720p | 18 | size of the cells and spacing of the veins | knob 2 |
| turn (rotation angle) | −90..+90° | −16° | positive: agents follow trails into veins; negative: swirls, fingerprints, dots | knob 3; bass adds ±15° |
| speed (step size) | 0–3 px/tick | 2 | growth rate; 0 freezes the agents | knob 4; build-up ramp |
| persistence (decay) | 0.80–0.99 | 0.90 | short-lived wisps (low) to dense, sticky networks (high) | knob 5; hats dip |
| scatter | 0–0.3 of agents per beat | 0.04 | how hard each kick tears the network | knob 6; fired by kick or beat clock |

Pads: BeatStep Pro pads 1–5 = presets Fibers, Fingerprint, Honeycomb, Net, Dots (glide, no reset); pad 6 = explosion; pad 7 = infect; pad 8 = reseed from centre. KeyStep 37 mod strip = injectGain (−2..+2) for the logo or audio attractors.

## Build recipes

### Resolume Arena

- Sim host sends the packed trail (P4 output) as a Spout sender named `Physarum`, 1280×720. In Arena, add it from the Sources browser (Spout) to a clip on layer 4.
- Colourize: Arena has no effect that reads the packed R/G/B channels into a custom ramp, so P5 goes into a Wire effect (next section) applied to that clip. Resolume documents ISF inside Wire, with float, bool, color and event inputs ([Resolume ISF](https://resolume.com/support/en/isf)); five `color` inputs give an editable palette right in Arena's effect panel.
- After the Wire effect: Arena's Bloom (threshold 0.7, intensity 0.8). Layer opacity = master fader for the effect.
- Matte mode: video clip on layer 2, layer 4 in Multiply. Light mode: layer 4 in Add over any content.
- Inject back to the sim: send a dedicated layer (white logo on black, or spectrum spots) via Arena's Advanced Output to a Spout output named `PhysarumInject`; the sim host reads it as the inject texture. Keep this layer out of the main output (Advanced Output slice from that layer only).
- Control: map BeatStep Pro and audio directly in the sim host, because the parameters live there. Use Arena only for colour, layer opacity and bloom (MIDI shortcuts, audio FFT on exposure if wanted). Share tempo between Arena and the sim host with Ableton Link, so beat-based scatter lines up with Arena's BPM.

### Resolume Wire

- Main patch (colourize effect): effect texture input → ISF node with the P5 shader (inputs: 5 ramp colours, contagion colour, sparkle, hue shift) → effect output. Load the shader in the node panel under Attributes → Fragment Shader ([Resolume ISF](https://resolume.com/support/en/isf)). Expose the inputs as patch parameters so they appear in Arena.
- Optional displacement patch: two inputs (video and trail) → ISF node that offsets the video UV by the trail gradient × amount (0–20 px) → output.
- Full simulation in Wire: not with real agents. Resolume's ISF page states that Wire (7.22) has no vertex shaders, so agents cannot be drawn as points, and fragment passes cannot scatter. The only in-Wire option is the grid-agent fallback (Technique 5) as one multipass ISF with three persistent float buffers (`PERSISTENT: true`, `FLOAT: true`, size `$WIDTH/3`, `$HEIGHT/3` from the [ISF multipass spec](https://docs.isf.video/ref_multipass.html)). Resolume's ISF page does not mention multipass or persistent buffers, so test that the buffers keep their content across frames in your Wire version before relying on it. Expect ~58 k agents and a blockier look.

### TouchDesigner (recommended sim host on the Windows VJ machine)

- Agent state persists in a texture loop: GLSL TOP `agents_update`, 1024×1024, 32-bit float RGBA (one texel = one agent: `r, g` = position in trail px, `b` = heading, `a` = flags). Its input 0 is a Feedback TOP whose Target TOP is `agents_update`, so each frame reads last frame's agents. The Feedback TOP's own input is a GLSL TOP `agents_init` (random positions, or a centre disc); pulsing the Feedback TOP's Reset loads it (= reseed). Do not use 16-bit float for positions: at 2048 px, half floats step in 2 px increments.
- Deposit: Render TOP `deposit`, 1280×720, 32-bit float mono, orthographic Camera COMP covering the trail in pixels, black background. Geometry COMP with one point, instanced 1,048,576 times with `agents_update` as instance source (translate from r, g). Constant MAT, colour `(dep, infected·dep, 1)`, blending on with Add. This gives count deposit.
- Trail persists in a second loop: GLSL TOP `trail_step` 1280×720, 16-bit float RGBA, inputs: Feedback TOP (target `trail_step`), `deposit`, inject (Syphon Spout In TOP `PhysarumInject`). Shader = P0 + P3.
- Wire `agents_update` input 1 to `trail_step` so agents sense the previous frame's trail (one frame of lag is invisible).
- Grade: GLSL TOP `grade` = P4 → Syphon Spout Out TOP `Physarum` (and/or NDI Out TOP for another machine).
- Controls: MIDI In CHOP (BeatStep Pro) → Lag CHOP → GLSL uniforms. Audio Device In CHOP → Audio Spectrum CHOP → band selects → Lag CHOP (attack/release from Audio reactivity). Ableton Link CHOP for beat index and bar phase.
- Agent counts: 512×512 = 262 k (laptop), 1024×1024 = 1.05 M (desktop default), 2048×2048 = 4.2 M (high-end; instanced draw becomes the main cost). Jenson ran 5–10 M with hand-written GLSL on a GTX 1070 ([Sage Jenson](https://cargocollective.com/sagejenson/physarum)), so these are conservative. Measure with the Performance Monitor.

### Unreal Engine 5.7

- Niagara system `NS_Physarum`, one emitter, Sim Target GPUCompute, fixed bounds (a 1280×720 world-unit plane), Determinism off.
- Agent state persists as particle attributes: Emitter Spawn → Spawn Burst Instantaneous, Count = user parameter `AgentCount` (default 1,000,000). Particle Spawn: set `Particles.Position` to a random point on the plane and a custom float attribute `Particles.Heading`. No Lifetime kill, no Solve Forces and Velocity: positions are written directly by your own module, so agents live until the system resets.
- Trail persists in emitter data: an emitter attribute `Trail` of type Grid2D Collection, Num Cells X/Y 1280×720, one attribute (four for contagion and species), Override Buffer Format = half float ([Grid2D Collection API](https://dev.epicgames.com/documentation/en-us/unreal-engine/python-api/class/NiagaraDataInterfaceGrid2DCollection)). Grid 2D Collections store named attributes on 2D grids, and simulation stages run in order with an optional Num Iterations ([Epic: fluid simulation overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/fluid-simulation-in-unreal-engine---overview)).
- A second Grid2D Collection `Deposit` with Clear Before Non Iteration Stage = on, so it starts empty each tick.
- Stage order:
  1. Particle Update: custom module = P1. Sense by sampling the previous `Trail` values at the three sensor UVs, turn, move, wrap.
  2. Simulation stage `Deposit`, iteration source Particles: write `dep` into `Deposit` at the agent's cell (presence deposit; plain writes race but write the same value).
  3. Simulation stage `DiffuseDecay`, iteration source Data Interface = `Trail`: P0 + P3 reading previous `Trail`, `Deposit` and an inject texture sample.
  4. Simulation stage `Export`, iteration source Data Interface = Render Target 2D bound to user parameter `RT_Physarum` (1280×720, RGBA8): write P4. This is the render-target setup described by [80.lv](https://80.lv/articles/niagara-system-integration-with-render-targets-in-unreal-engine-5) (User-level Texture Render Target parameter, emitter Render Target 2D, Generic Simulation Stage with Iteration Source = Data Interface).
  Grid function names (previous-value sampling, set value) vary between engine versions; search the Grid2D node list in 5.7.
- Output: send `RT_Physarum` with your Spout or NDI plugin, or show it in-scene on a plane material. For "network as geometry", use `RT_Physarum` as World Position Offset height on a dense plane (0–40 cm).
- Remote Control: a Blueprint actor `BP_PhysarumControl` with float variables `Topology`, `Scale`, `Turn`, `Speed`, `Persistence`, `Scatter`, `InjectGain` and trigger functions `Explode`, `Infect`, `Reseed`. Expose them in a Remote Control Preset, bind to OSC and MIDI (BeatStep Pro). On change, call Set Niagara Variable (Float) on the Niagara component. `Reseed` calls Reset System.
- Agent count: 0.5–2 M on a current desktop GPU as a starting point (Niagara particles carry more attributes than a bare 16-byte agent). Check `stat GPU`.

### Godot

- Godot 4 with the Forward+ or Mobile renderer; compute shaders only work on RenderingDevice-based renderers ([Godot compute shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/compute_shaders.html)).
- Use the main RenderingDevice (`RenderingServer.get_rendering_device()`), not a local one, so the trail texture can be displayed directly through a Texture2DRD ([Godot Texture2DRD](https://docs.godotengine.org/en/stable/classes/class_texture2drd.html)). Record dispatches inside `RenderingServer.call_on_render_thread`.
- Agent state persists in a storage buffer: N × vec4 (`x, y, heading, flags`), 16 bytes per agent, created once in `_ready` and freed with `free_rid()` on exit (RIDs are not freed automatically, per the compute docs).
- Trail persists in two storage images, R16F or R32F, 1280×720 (A/B ping-pong), plus a deposit image `R32_UINT` for count deposit with `imageAtomicAdd` (store deposit × 256 as fixed point; clear each tick with a small clear dispatch).
- Per frame: dispatch P1+P2 merged (N / 256 workgroups, local size 256), barrier, dispatch P0+P3 (local size 8×8), swap A/B. No `sync()`; the compute docs warn that syncing blocks the CPU.
- Display: TextureRect with the Texture2DRD of the current trail image, `canvas_item` shader for P4 + P5 (if showing in Godot) or P4 only (if sending to Resolume). Godot has no built-in Spout or NDI output; run it fullscreen on its own display output or use a community addon.
- Uniforms (push constant): `sensor_dist`, `sensor_angle`, `rot_angle`, `step_size`, `decay`, `diffuse`, `dep`, `scatter`, `beat_index`, `explode`, `frame`. Drive them from AudioEffectSpectrumAnalyzer on the music bus and from MIDI input events.
- Agent count: 1–4 M; a plain compute path is closest to Jenson's setup.

## Audio reactivity

| band | drives | smoothing | threshold / scaling |
|------|--------|-----------|---------------------|
| kick, 40–120 Hz | scatter 4 % (one tick), flash +30 % exposure | attack 5 ms, release 250 ms | trigger at 0.6 of the band's running peak (2 s window), retrigger blocked 120 ms |
| bass, 60–250 Hz | turn ±15° around the knob value | attack 60 ms, release 500 ms | linear above a 0.2 floor; slow, so topology bends instead of flickering |
| snare / clap, 1–3 kHz transient | small explosion at a random point, radius 0.1 | trigger only | at 0.5; max one per beat |
| hats, 6–12 kHz | decay −0.03 | attack 5 ms, release 80 ms | at 0.5 |
| spectrum, 8 log bands | inject spots: band k = a disc at a fixed position on a circle, gain = band level | per band `s = max(new, s · 0.9)` | agents grow toward the active frequencies (variation from the research) |
| RMS loudness | substeps 1 → 2 | attack 500 ms, release 3 s | only in drop sections |

Scatter and explosion are counted in beats (Link / beat clock), not frames, so they stay on the grid if the frame rate drops.

## Performance tips

- Memory is small: 1 M agents × 16 B = 16 MB; trail 1280×720 RGBA16F = 7.4 MB per buffer. The cost is the scattered reads and writes: three sensor reads per agent plus one deposit write.
- Count deposit with Add blending or atomics slows down when many agents hit the same pixel (dense Honeycomb or Dots). If the frame time spikes there, switch to presence deposit or lower the agent count.
- Density rule: keep about 0.35 agents per trail pixel (Ghassaei's default). Agents = 0.35 × trail pixels: 1280×720 → 323 k; 1920×1080 → 726 k. More agents per pixel give brighter, thicker veins rather than more detail, so raise trail resolution and agent count together.
- Trail resolution: 1280×720 is the sweet spot for a 1080p output. Filaments are 1–3 trail px wide; at native 4K they become hair-thin and alias on LED pitch. On a low-resolution LED wall, set the trail to the wall's native pixel count and scale sensor distance and step size by `k`.
- Odd aspect ratios: give the trail the wall's aspect with square pixels, e.g. 5760×1080 strip → trail 2880×540, 0.35 × 1.56 M ≈ 545 k agents. Keep torus wrap on X for seamless 360° panoramas.
- Spout/NDI: send the tonemapped 8-bit image (P4), never raw trail values, and add grain in Resolume against banding. 1280×720 RGBA8 over Spout is cheap; NDI to another machine adds a frame or two of latency, fine for this slow effect.
- Frame rate: one sim tick per frame, so a drop from 60 to 30 fps halves growth speed. Beat-triggered events stay on time because they are counted in beats.
- Pitfalls: positions in half float (jitter, banding of the network along a grid); sensor distances not scaled after changing trail resolution (wrong cell size); resetting agents on every preset change (loses the morph); bloom with a low threshold (turns veins into fog).

## Further references

- [Sage Jenson, Physarum](https://cargocollective.com/sagejenson/physarum) — the real-time adaptation of Jones's model: parameter list, 3×3 diffuse plus decay, no collision step, contagion, 5–10 M particles on a GTX 1070.
- [Amanda Ghassaei, gpu-io Physarum source](https://raw.githubusercontent.com/amandaghassaei/gpu-io/main/examples/physarum/index.js) — readable WebGL fragment-shader implementation with the five presets used above; [live demo](https://apps.amandaghassaei.com/gpu-io/examples/physarum/).
- [Sebastian Lague, Slime-Simulation (GitHub, Unity, GPL-3.0)](https://github.com/SebLague/Slime-Simulation) — the Unity project behind his [Coding Adventure video](https://www.youtube.com/watch?v=X-iSQQgOd1A), which shows the slime agents forming veins on the GPU; a good companion for the Godot compute route.
- [80.lv, Niagara system integration with render targets in UE5](https://80.lv/articles/niagara-system-integration-with-render-targets-in-unreal-engine-5) — step-by-step for the Export stage in the Unreal recipe.
