# Kuramoto Firefly Synchronization — design

## Look

A dark summer meadow at night, almost black, filled with a few thousand small yellow-green points of light. Each one is a firefly with its own clock. At low coupling the field twinkles at random, like static made of light. As the performer raises the coupling, nearby lights start to agree: small clusters blink together, and then waves roll across the screen as neighbours pull each other along. At full coupling the whole field breathes as one, and a single unison flash lights the wall on the downbeat. Colour tells the audience how "together" the swarm is. Incoherent fireflies are cool green-cyan; a locked swarm turns warm amber and the bloom swells. A kick can snap all clocks to the beat, and a "scatter" gesture throws them back into chaos, so the swarm re-syncs live in front of the crowd. That scatter-and-resync arc is the core of Nicky Case's explorable, where dots in the dark start random, form clusters, then waves, then near unison, and dragging scatters them again ([Nicky Case: Fireflies](https://ncase.me/fireflies/)). An optional overlay shows the maths: a thin ring where every firefly is a point at its phase angle, and the cloud collapses into one clump as the swarm locks, as in the hdietert animation ([hdietert](https://hdietert.github.io/static/kuramoto-animation/kuramoto.html)). The mood is quiet, organic and suspenseful. It feels like waiting for a crowd to start clapping in time.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | flat #020508 with a very faint low-frequency noise (±2/255) | Normal | 100 % | not pure 0. Keeps LED modules alive and hides 8-bit banding in the glow falloff |
| 2 | Oscillator state | P0: 64 × 64 float texture, one texel = one firefly (phase, natural frequency, flash flag, seed), ping-pong | internal | — | the simulation; never shown |
| 3 | Order parameter | P1a/P1b: reduction 64×64 → 8×8 → 1×1, gives r and ψ (mean phase) | internal | — | drives colour, bloom and the global coupling term |
| 4 | Firefly field | P2: each pixel sums the glow of the fireflies in the 3 × 3 nearest cells | Add | 100 % | the hero layer |
| 5 | Flash trails | P3: persistent buffer, `trail = max(trail·decay, field)` | Add | 35–60 % | short afterglow of each flash; makes the waves readable as fronts |
| 6 | Phase ring (optional) | P4: 128-bin phase histogram drawn as a polar ring, plus the mean vector of length r | Add | 0–70 % | the "maths view" for breaks or explainer moments |
| 7 | Raster strip (optional) | P5: one grid row per frame scrolled down, greyscale phase | Add or Screen | 0–40 % | homage to the time-on-one-axis phase raster in the Kuramoto Web Installation ([Research Catalogue](https://www.researchcatalogue.net/view/552571/621219)) |
| 8 | Post | bloom (gain driven by r), then 1-LSB dither | see compositing | — | bloom carries the unison flash |

## Techniques

**1. Kuramoto mean-field coupling (the main engine)**
The model: N oscillators, each with its own natural frequency, all coupled to all, with a pull that depends on the sine of the phase difference. Below a critical coupling they stay incoherent; above it they synchronise. The order parameter r is 0 for uniformly spread phases and near 1 for unison, and ψ is the mean phase ([Wikipedia: Kuramoto model](https://en.wikipedia.org/wiki/Kuramoto_model)).
1. Each firefly i holds a phase φᵢ in [0, 2π) and a natural angular frequency ωᵢ (rad/s).
2. Mean field: `z = r·e^{iψ} = mean(cos φⱼ) + i·mean(sin φⱼ)`, computed by the reduction pass (technique 4).
3. Update per frame: `φᵢ += Δt · (ωᵢ + K·r·sin(ψ − φᵢ) + σ·ηᵢ)`, where η is fresh per-frame noise in [−1, 1] and σ the noise strength. Wrap φ into [0, 2π).
4. Frequency spread: draw a fixed uniform u per firefly from its seed and set `ωᵢ = ω₀ + γ·tan(π(u − 0.5))`, clamped to ±4γ. This is a (truncated) Lorentzian with half-width γ. For a Lorentzian, the critical coupling is Kc = 2γ ([Wikipedia: Kuramoto model](https://en.wikipedia.org/wiki/Kuramoto_model)); the clamp moves it slightly, so treat that as approximate.
5. Stage normalisation: expose `couple` 0–1 and set `K = couple · 4γ`. Then couple = 0.5 sits at about the threshold, which is the most interesting place to hover. When γ = 0, use `K = couple · 4 · 0.3 rad/s` so the fader still does something.
6. ω₀ comes from the BPM: `ω₀ = 2π · (BPM / 60) / beatsPerFlash`. Default beatsPerFlash = 2 (one flash every 2 beats, 1.067 Hz at 128 BPM).

**2. Local (spatial) coupling for travelling waves**
The research notes that coupling only to neighbours makes waves sweep across a grid. Mix it with the global term.
1. For each grid cell, average `e^{iφ}` over its 8 neighbours (wrap at the edges or clamp, see Performance): `z_loc = r_loc·e^{iψ_loc}`.
2. Coupling term: `C = K · ( reach·r·sin(ψ − φ) + (1 − reach)·r_loc·sin(ψ_loc − φ) )`.
3. reach = 0: neighbours only. This gives spirals, wavefronts and patches that never fully agree. reach = 1: all-to-all. Unison arrives fast and clean. Around 0.2–0.4 you get clusters that merge into one wave, then lock.
4. Local coupling needs roughly 2× the global K to look locked. Multiply the local half by 2 internally so the `couple` fader feels the same in both modes.

**3. Pulse-coupled firefly mode (the alternative rule)**
Strictly, firefly coupling is a pulse rule, not a smooth sine. In Nicky Case's explorable every firefly has a clock, flashes when it "strikes twelve", and nudges its clock forward when it sees a neighbour flash ([Nicky Case: Fireflies](https://ncase.me/fireflies/), [Bram.us](https://bram.us/2021/04/03/synchronizing-fireflies-visualizer)).
1. The state texture stores a flag B = 1 for every firefly that wrapped (flashed) this frame.
2. Each firefly counts flashing neighbours in its 3 × 3 (or 5 × 5) block from the previous frame: n.
3. `φᵢ += Δt·ωᵢ + ε·n·2π`, with ε = nudge per seen flash, 0.01–0.08 of a cycle (default 0.03). If this pushes φ past 2π, it flashes now (it gets absorbed into the group).
4. Refractory window: ignore nudges while φ < 0.15·2π, so a firefly that just flashed is not dragged into a double flash.
5. Use a `rule` switch: 0 = Kuramoto (smooth, the swarm "breathes"), 1 = pulse (snappier, more like real fireflies, waves have harder fronts). Both share every other pass.

**4. Order parameter by reduction**
1. P1a: an 8 × 8 target. Each pixel averages `(cos φ, sin φ)` over its 8 × 8 block of the state texture (64 fetches).
2. P1b: a 1 × 1 target averages the 8 × 8 result (64 fetches). Store `(X, Y, r = length(X, Y), ψ = atan2(Y, X))`.
3. Also store a smoothed copy: `r_s = mix(r_s, r, 1 − exp(−Δt/0.15 s))`. Use r_s for anything visual (colour, bloom), so it doesn't flicker.
4. P0 of the next frame reads P1b. The one-frame lag is invisible.
5. r is the most useful number in the whole patch: it is "how together the crowd is", and can be sent to other layers (see Audio reactivity).

**5. Beat pin, kick and scatter (performance gestures)**
1. Beat pin: a pacemaker term `+ P·sin(φ_beat − φᵢ)`, where `φ_beat = 2π·(beatClock / beatsPerFlash)` from the host BPM, and P = pin·2 rad/s. Without it a locked swarm flashes at its own mean phase, which drifts against the music. With pin ≈ 0.3 the unison flash lands on the beat.
2. Kick (a phase reset toward a target angle; the hdietert demo has a similar perturbation control with a target angle and strength ([hdietert](https://hdietert.github.io/static/kuramoto-animation/kuramoto.html))): on a trigger, `φᵢ += kick·wrap(φ_target − φᵢ)`, where wrap maps to (−π, π]. kick = 1 aligns everyone at once; 0.3 just tightens the swarm.
3. Scatter: on a trigger, `φᵢ += scatter·2π·hash(seed, frame)`. Scatter = 1 destroys all sync in one frame; the swarm then re-syncs at a speed set by `couple`.
4. Local scatter: a MIDI-key gesture that scatters only cells inside a circle (centre from the note number, radius 0.15 of the frame). Like dragging in the explorable, it punches a hole of chaos that heals from the edges.

**6. Firefly rendering (glow sprites without geometry)**
1. Each firefly lives at a grid cell (gx, gy), drawn at `pos = (cell + 0.5 + J(seed) + W(t, seed)) / grid`, with jitter J in ±0.3 cells and a slow wander `W = 0.25·(sin(0.3t + 6.28·s₁), sin(0.23t + 6.28·s₂))` cells. Map grid to screen with the aspect ratio (see Performance).
2. Flash envelope from phase: with `c = φ / 2π` (0 right after the flash), `b = exp(−c / flash) + idle`, where flash = 0.04–0.3 (fraction of a cycle) and idle = 0.03 (a dim ember so the swarm is never invisible).
3. Per pixel, find its cell and loop over the 3 × 3 neighbour cells (9 fireflies). For each: `d = distance(pixel, pos)` in cell units. `glow = b·( exp(−d²/(2·0.06²)) + 0.25 / (1 + (d/0.18)²) )`: a sharp core plus a soft halo.
4. Size breathes with the flash: core σ × (1 + 0.5·b), so flashing fireflies swell a little.
5. Sum the 9 contributions. Colour comes from the palette (Color scheme), not here.

## Shaders

All passes run in one multipass shader (ISF PASSES with PERSISTENT FLOAT targets, or render targets in UE/Godot). Render order per frame: P0 → P1a → P1b → P2 → P3 → P4 → P5 → P6.

**P0 — State update (ping-pong)**
- Purpose: advance every oscillator one step.
- Inputs: previous state (64 × 64, RGBA float, persistent), P1b (r, ψ), uniforms couple, spread (γ), reach, rule, nudge, noise (σ), pin, beatClock, BPM, beatsPerFlash, kick, kickTarget, scatter, localScatter (x, y, amount), TIMEDELTA, FRAMEINDEX.
- Per texel: (1) on frame 0 or on `reset`, init φ = 2π·hash, ω from seed (technique 1.4), B = 0, A = seed. (2) Read the 8 neighbours, build z_loc and the flash count n. (3) Apply technique 1 or 3 depending on rule, plus pin, kick and scatter. (4) Detect a wrap: B = 1 if the new φ < old φ (after adding, before wrap) or if it crossed 2π. (5) Wrap φ.
- Output: RGBA = (φ, ω, B, seed). Resolution: grid × grid, default 64 (4096 fireflies), range 32–128.

**P1a / P1b — Reduction**
- Purpose: compute r and ψ.
- Inputs: P0 output; P1b previous value (for r_s smoothing).
- Per pixel: average cos and sin over a block (technique 4).
- Output: P1a 8 × 8 RG float; P1b 1 × 1 RGBA float = (r, ψ, r_s, unused), persistent.

**P2 — Firefly field**
- Purpose: draw the glow.
- Inputs: P0, P1b, uniforms flash, idle, sizeScale, palette colours, time.
- Per pixel: technique 6 steps 1–5, giving intensity I. Colour: `col = I · mix(coolColour, warmColour, smoothstep(0.3, 0.9, r_s))`, and the cores are pushed toward white with `col += max(I − 1, 0)·#FFF6D0`.
- Output: RGBA16F at full output resolution (half resolution is acceptable on huge walls; the cores then get softer).

**P3 — Trails (persistent)**
- Purpose: afterglow of each flash so waves read as moving fronts.
- Inputs: P2, previous P3.
- Per pixel: `trail = max(prev·k, P2)`, with `k = exp(−Δt / τ)` and τ = trail length 0.05–0.6 s (default 0.2 s).
- Output: RGBA16F, half resolution is fine (the trail is blurry by nature).

**P4 — Phase histogram and ring (optional)**
- Purpose: the unit-circle "maths view".
- Inputs: P0, P1b.
- Pass A (128 × 1): each bin loops over all N texels and counts phases falling in its 2π/128 slice. That is 128 × 4096 ≈ 0.5 M fetches, so only run it if the ring is on, or use grid 32 for this pass (sample every second texel).
- Pass B (full res, polar): for each pixel, compute angle a and radius ρ around the centre. Bin = a / 2π · 128. Ring line at radius R = 0.32 of the frame height, thickness 1.5 px, brightness `0.15 + 2·count/N`. Plus a mean vector: a line from the centre with length r·R at angle ψ, 2 px, highlight colour.
- Output: RGBA8, composited Add.

**P5 — Phase raster (optional, persistent)**
- Purpose: time-on-one-axis stripe pattern.
- Inputs: P0, previous P5.
- Per pixel: if the row is the top row, write grey = `0.5 + 0.5·cos φ` of grid row 32 sampled along x; else copy the pixel one row above (scroll by 1 to 4 px per frame, BPM-tied: one screen height per 8 bars).
- Output: RGBA8, persistent, full width × 0.5 height, placed on the lower half or rotated as a side band.

**P6 — Composite and post**
- Inputs: background, P2, P3, P4, P5, r_s.
- Order: background + P2 + trailMix·P3 + ringMix·P4 + rasterMix·P5 → bloom (if done in-shader: threshold 0.7, 5-tap downsample chain) with gain `0.6 + 1.4·r_s` → soft clip `x / (1 + x)` per channel above 0.8 → 1-LSB triangular dither.
- Output: final frame.

## Color scheme

| role | hex | use |
|------|-----|-----|
| base / night | #020508 | background, never darker |
| shadow / ember | #14200C | the idle glow of non-flashing fireflies |
| cool (incoherent) | #3AFFB4 | flash colour when r_s < 0.3 |
| firefly (mid) | #C8FF4D | yellow-green flash colour, around r_s = 0.6 |
| warm (locked) | #FFB43C | amber flash colour when r_s > 0.9 |
| highlight | #FFF6D0 | over-exposed cores, mean vector on the ring |
| ring / raster | #5A7A6A | the maths overlay, kept dim |

- Mapping: a three-stop gradient ramp cool → firefly → warm, indexed by r_s. Each firefly's colour is the same at a given moment; only its brightness differs. This is on purpose: the colour is a readout of the whole swarm.
- Hue-wheel alternative (`colorMode = phase`): hue = φ / 2π, saturation 0.7, value = b. Incoherence becomes confetti; sync turns the whole field one hue that rotates. Good for a break, too busy for long stretches.
- Over time: nothing drifts on its own. Colour only follows r, so the audience learns the code after one build-up.
- LED walls: black level 2/255 (#020508), maximum flash about 0.9 before bloom so the unison flash does not clip into a flat white panel. A unison flash at full couple can raise wall brightness by 5–10×, so cap the total with the soft clip in P6 and check the venue's brightness limit.

## Blending and compositing

- Fireflies, trails, ring and raster: Add. They are light on dark, and Add keeps overlapping halos glowing rather than covering each other.
- Raster strip: Screen if it sits under the field, so it never pushes the field to white.
- Trail mix 35–60 %: lower when reach is high (unison flashes leave a big flat trail that looks like a flicker), higher when reach is low (waves need trails to show their direction).
- Masks: optional silhouette (grass, tree line, crowd key) in Normal on top, black. It sells the "meadow" and hides the bottom edge of the grid.
- Post chain in order: bloom (threshold 0.7, intensity 0.6–2.0 driven by r_s, radius about 1.5 % of frame height) → soft clip → light grain/dither 1 LSB. No chromatic aberration: it smears the tiny cores into coloured noise. No sharpen.

## Motion and timing

- Phases: the flash rate is ω₀ = one flash every `beatsPerFlash` beats (1, 2 or 4). The spread γ in Hz terms is 0–0.5 Hz around that.
- Fireflies wander slowly (technique 6.1): about 0.04 cells/s, never synced to the beat. The positions are calm; all rhythm is in the light.
- Sync speed: above the threshold, the swarm locks within roughly 2–8 flash cycles, depending on couple. So a fader move from 0.3 to 0.8 takes about 2 to 4 bars to fully show. Plan ahead.
- Typical phrase structure:
  - Intro / break: couple 0.2–0.35, reach 0.1, flash 0.08. Random twinkle with an occasional local cluster.
  - Build (8–16 bars): ramp couple 0.35 → 0.65 linearly, reach 0.1 → 0.4. Clusters merge, waves roll across the wall, colour moves from cool green to yellow-green.
  - Drop: on the downbeat, kick = 1 with kickTarget = beat phase, couple → 0.9, pin 0.4, flash 0.15. The whole wall flashes as one on every 2nd beat, amber, with full bloom.
  - Mid-drop variation (every 8 bars): a scatter of 0.5 on bar 1 of the phrase. The swarm breaks and re-syncs within 1–2 bars, a visible "tension and release".
  - Breakdown: scatter = 1, couple 0.3, ring overlay fades in over 4 bars at 50 %. The audience sees the clump on the ring dissolve into a full circle.
- Easing: couple and reach follow linear or ease-in ramps (the system already supplies its own nonlinearity around the threshold). Triggers (kick, scatter) are one-frame impulses.
- Boredom guard: don't leave couple above 0.85 for more than 16 bars, or below 0.3 for more than 16 bars. The interesting zone is 0.4–0.7.

## Parameters

| name | range | default | what it does on stage | suggested mapping |
|------|-------|---------|-----------------------|-------------------|
| couple | 0–1 | 0.45 | coupling strength K, normalised so 0.5 ≈ threshold: chaos → clusters → unison | KeyStep mod strip or a fader; build ramps via Arena dashboard |
| spread | 0–0.5 Hz | 0.12 | natural frequency spread γ: wider resists sync, gives more "individual" fireflies | BeatStep Pro knob |
| reach | 0–1 | 0.3 | local (waves, spirals) vs global (instant unison) coupling | BeatStep Pro knob |
| flash | 0.04–0.3 of a cycle | 0.10 | length of each flash: short = sparkle, long = soft breathing | knob; tie to track tempo feel |
| pin | 0–1 | 0.3 | how strongly the unison flash locks to the BPM grid | knob; push to 0.5 on the drop |
| kick / scatter | trigger, amount 0–1 | 0 | kick snaps all clocks to the beat; scatter throws them into chaos | BeatStep Pro drum pads (kick = pad 1, scatter = pad 2); KeyStep keys = local scatter at a position |

Secondary (set and forget): grid 32–128, idle 0–0.08, trail τ 0.05–0.6 s, beatsPerFlash 1/2/4, rule 0/1, noise σ 0–1 rad/s, colorMode.

## Build recipes

### Resolume Arena

- **Layer 1, background:** Solid Color #020508, Normal, 100 %.
- **Layer 2, swarm:** the Wire patch (below) as a source, blend mode Add. Resolume documents ISF inside Wire through its ISF node ([Resolume ISF](https://resolume.com/support/en/isf)), so the custom shader goes through Wire. Publish couple, spread, reach, flash, pin, trail, ringMix and the kick/scatter triggers.
- **Layer 3, silhouette (optional):** a PNG with alpha (grass or tree line), Normal, 100 %.
- **Layer 4, foreground (optional):** type or line work in Add. The field is mostly dark, so text always reads.
- **Composition effects:** Bloom, threshold about 0.65 and intensity about 1.0. If r is not exported from Wire, link the bloom intensity to the same dashboard fader as couple (range 0.8–1.6), so it rises with the build.
- **BPM:** set Arena's BPM from your tap or Ableton Link; Wire reads it for ω₀ and the pin clock.
- **MIDI:** map the BeatStep Pro pads to kick and scatter (trigger parameters, "note on" only). Map couple to a KeyStep 37 mk2 control (mod strip or a CC knob) so a build can be ridden by hand.
- **No-shader fallback:** there is none that keeps the sync behaviour. For an emergency, use a particle/dot clip with Strobe at 1/2 beat in Add plus a Threshold. That gives "flashing dots" but no emergence, so keep the real patch for the show.

### Resolume Wire

- **Clock:** Wire's time and BPM/beat-position outputs → ISF inputs `beatClock` (float, beats) and `bpm`. Check what your Wire version calls these nodes. Use ISF's own TIMEDELTA for Δt.
- **Main ISF node:** one multipass ISF with P0 `state` (PERSISTENT, FLOAT, width/height = grid), P1a `red8` (FLOAT, 8 × 8), P1b `order` (PERSISTENT, FLOAT, 1 × 1), P2 `field`, P3 `trail` (PERSISTENT, FLOAT, `$WIDTH/2` × `$HEIGHT/2`), optional P4a `hist` (128 × 1), P5 `raster` (PERSISTENT), and the final pass to output. Multipass, PERSISTENT and FLOAT targets and size expressions are part of the ISF spec ([ISF multipass reference](https://docs.isf.video/ref_multipass.html)). INPUTS: float sliders with MIN/MAX for the parameters table, `kick` and `scatter` as event inputs, `kickTarget` float 0–1 (fraction of a cycle), `rule` and `colorMode` as long/menu inputs, palette colours as color inputs.
- **Version caveat:** the Resolume ISF page does not cover multipass or persistent buffers, and support can differ between Wire versions. Test first that `state` keeps its values across frames (a quick test: set couple 0, kick 1 once, and see whether all fireflies flash together afterwards).
- **Fallback with Wire feedback:** split P0 into its own single-pass ISF node and close the loop with a Wire feedback/delay texture node (mix 1.0, i.e. full replace, float format if available). A second ISF node does P1 + P2 + P6 (doing the reduction as a 64-tap loop inside each pixel is too costly; instead read r from a 1 × 1 resize of a cos/sin texture if your Wire version offers a resize node). Phases in 8-bit lose precision: φ quantised to 1/256 of a cycle still works, but ω must then come from the seed inside the shader rather than from the texture.
- **Exposing r:** if Wire lets you read a texture value back as a number in your version, route r_s out as a published parameter or OSC message so it can drive other layers or UE. If not, keep r's effects (colour, bloom) inside the shader.
- **Audio:** `Audio Analysis` → low band → envelope (attack 2 ms, release 100 ms) → threshold 0.6 → trigger `kick` with amount 0.35 (gentle tightening on every kick, not a full reset). High band → envelope (attack 1 ms, release 60 ms) → `noise` σ × 0.6. Full-band RMS → slow envelope (attack 1 s, release 3 s) → adds up to +0.15 to couple.

### Unreal Engine 5.7

Two routes.

**A. Render-target route (same as ISF, cheapest, best for big N).**
1. Render targets: `RT_StateA`, `RT_StateB` (64 × 64, RGBA32f), `RT_Red8` (8 × 8, RG32f), `RT_Order` (1 × 1, RGBA32f), `RT_Trail` (half output res, RGBA16f).
2. In a Blueprint actor's Tick: `Draw Material to Render Target` in order: M_State (reads StateA + Order, writes StateB) → M_Reduce8 → M_Reduce1 → swap A/B. The materials are Unlit with a Custom node for the 8-neighbour and 64-tap loops.
3. Display: a full-screen post-process material or an Unlit plane that runs P2 + trails + composite. Bloom via the post-process volume (Bloom intensity driven from r via a Material Parameter Collection).
4. Reading r into Blueprint: `Read Render Target Raw Pixel` on the 1 × 1 RT_Order once per frame (a GPU readback; fine at 1 pixel, but it adds a frame of latency) → store r_s → set MPC_Firefly.r, and send it out as OSC with the OSC plugin if other tools need it.

**B. Niagara route (true 3D fireflies, camera moves).**
1. One emitter, GPU sim, Spawn Burst Instantaneous of 2,000–8,000 particles in a box (e.g. 40 × 40 × 6 m meadow volume).
2. Particle attributes: Phase (float), NatFreq (float, from a random Lorentzian in a scratch pad), Flash (float).
3. Global coupling: a Simulation Stage or a scratch pad that reads the mean field. The simplest form is the render-target route above running alongside, with the particle's ExecIndex mapping to a texel of RT_State (Sample Texture with UV = (index mod 64, index / 64)). Then particles only read their phase; the sim stays in materials.
4. Local coupling in true 3D: a Neighbor Grid 3D with a radius of about 1.5 m, counting flashing neighbours for the pulse rule (technique 3). This costs more; keep N ≤ 4,000.
5. Renderer: Sprite Renderer, material Unlit Additive, a radial falloff texture, Emissive = palette colour × flash × 20 (to feed bloom). Camera: a slow dolly through the meadow at 0.2–0.5 m/s, depth of field on for the far fireflies.
6. Movement: Curl Noise Force, strength 20–50 cm/s², frequency low, plus Drag 1.0 so they drift, not fly.

**Control and output (both routes).**
- Remote Control Preset exposing MPC_Firefly scalars (couple, spread, reach, flash, pin, trail, beatsPerFlash) and Niagara user parameters. Kick and scatter are exposed as Blueprint functions (or as float params that the material treats as impulses for one frame and then resets from Blueprint).
- Bind the scalars via OSC (from Wire/Arena audio envelopes) and MIDI (BeatStep Pro pads, KeyStep CCs) through Remote Control's protocol bindings.
- BPM: send it from Arena by OSC, or tap in UE; feed beatClock from a Blueprint accumulator `beat += Δt·BPM/60`.
- Output to Arena via Spout or NDI with the plugin you already use. Arena then adds the silhouette and the final bloom.

### Godot

- **Simulation (Godot 4):**
  1. `SubViewport StateA` and `StateB` (64 × 64, HDR 2D on so the texture is float, update mode Always, no filtering on the reading sampler: use `filter_nearest`). Each holds a `ColorRect` with a canvas_item shader running P0, reading the other viewport's texture and the Order viewport.
  2. A GDScript toggles which viewport writes each frame (ping-pong), or use one viewport whose shader reads a texture copied from the previous frame.
  3. `SubViewport Red8` (8 × 8) and `SubViewport Order` (1 × 1) with the reduction shaders.
  4. Alternative for big N: a compute shader through RenderingDevice holding phases in a storage buffer, with a parallel reduction for r. It is more work but removes the texture precision questions.
- **Display:** a full-screen `ColorRect` with the P2/P6 shader, a `SubViewport Trail` for P3, and a `WorldEnvironment` with Glow on (intensity 0.8–1.6 from r, HDR threshold 0.7).
- **Uniforms:** couple, spread, reach, flash, pin, idle, trail_tau, beats_per_flash, bpm, beat_clock, kick (impulse), kick_target, scatter (impulse), local_scatter (vec3: x, y, amount), rule (int), color_mode (int), cool/firefly/warm colours (`source_color`), delta (from `_process(delta)`).
- **r in script:** `Order` viewport's texture → `get_image()` once per frame (GPU readback, small but synchronous) to drive the Glow intensity and other nodes. Or keep it all in the shader.
- **Audio:** `AudioEffectSpectrumAnalyzer` on the music bus; read bands in `_process` and apply the envelopes from the Audio reactivity section in script.
- **Output:** Spout or NDI through the add-on you use, or window capture in Arena.

## Audio reactivity

| source | band | smoothing | threshold | drives |
|--------|------|-----------|-----------|--------|
| kick | 40–120 Hz | attack 2 ms, release 100 ms | 0.6 of the running peak, 150 ms hold-off | `kick` trigger, amount 0.35 (on the drop: 1.0) |
| snare / clap | 1–4 kHz transient | attack 2 ms, release 150 ms | 0.7 | optional local scatter at a random position, amount 0.6 (holes that heal) |
| hi-hats | 6–12 kHz | attack 1 ms, release 60 ms | none, continuous | noise σ, 0 → 0.6 rad/s: busy hats keep the swarm slightly restless |
| energy | full-band RMS | attack 1 s, release 3 s | none | +0 to +0.15 on couple, so loud sections pull together on their own |
| out: r_s | — | 150 ms (already in P1b) | — | bloom gain, and outbound OSC to other layers or UE (camera shake, a bass swell, a light cue) |

- Keep the kick amount low (0.2–0.4) through most of a track. A full reset on every kick kills the emergence, which is the whole point.
- The research suggests the inverse direction too: using r as a control signal. A good use is a sidechain on another visual layer's opacity, so the rest of the show "comes together" when the fireflies do.

## Performance tips

- P0 at 64 × 64 is trivial (4,096 texels × 9 fetches). Even 128 × 128 (16k fireflies) is cheap. The cost is in P2 (9 fireflies per output pixel, about 2 exp() each) at full resolution: about the cost of a medium blur. On a 4K or a very wide wall, run P2 at half resolution and upsample; the halos hide it.
- P4 (histogram) is the only expensive pass (~0.5 M fetches). Skip it when ringMix = 0, or run it at grid 32.
- Precision: phases and ω must be in float targets (RGBA16F at least; 32F preferred for φ, since 16F loses fine phase steps near 2π and small spreads get rounded away). 8-bit state works only for the fallback in Wire, with ω rebuilt from the seed.
- Frame rate independence: everything uses Δt. Cap Δt at 1/30 s so a dropped frame doesn't jump phases and create fake sync.
- Aspect ratio: make the grid match the output. For a 32:9 wall use, for example, 128 × 36 cells, not 64 × 64 stretched, otherwise the fireflies become horizontal ellipses and the waves look squashed. Keep cells square in pixels.
- Edges: use wrap-around neighbours in P0 for a seamless field (good for panorama rings and 360 domes), or clamp for a field that feels bounded. Wrap is the default.
- LED walls: the unison flash is a near full-wall brightness jump. Many LED processors and audiences dislike sudden full-field flashes. Keep `flash` ≥ 0.08 (softer attack), cap the composite brightness in P6, and avoid unison at more than about 3 flashes per second (the general photosensitivity guideline area; check your venue's rules).
- 1-pixel fireflies alias on LED pitches: keep the core σ at least 1.5 LED pixels. Calculate from the wall's actual pixel count, not the render resolution.

## Further references

- [Wikipedia: Kuramoto model](https://en.wikipedia.org/wiki/Kuramoto_model): the equations, the order parameter r, the mean phase ψ and the critical coupling for a Lorentzian frequency distribution used in technique 1.
- [Nicky Case: Fireflies](https://ncase.me/fireflies/): the pulse-coupled clock-nudge rule and the scatter-and-resync interaction. The page links its public-domain source code on GitHub.
- [Interactive Kuramoto animation (hdietert)](https://hdietert.github.io/static/kuramoto-animation/kuramoto.html): unit-circle phase view, order-parameter plot, noise and perturbation controls, a model for the P4 ring overlay and the kick gesture.
- [ISF Multi-Pass and Persistent Buffer Reference](https://docs.isf.video/ref_multipass.html): PASSES, PERSISTENT, FLOAT and size expressions behind P0–P6.
