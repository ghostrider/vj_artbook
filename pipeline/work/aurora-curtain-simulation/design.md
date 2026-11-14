# Aurora Curtain Simulation — design

## Look

A near-black night sky with fixed stars and a black horizon of hills or trees. One to three curtains of light hang above it. Each has a razor-sharp, bright lower hem that waves slowly from side to side. Above the hem, a green body breaks into thin vertical rays. They fade upward into a soft, diffuse red crown. When the music is energetic, a violet-blue line sits right at the hem. On every kick that hem flashes violet at once, the green body brightens with it, and then the green dies away over about 0.7 s. Red only builds when the energy holds over several bars, and it lingers long after the music has dropped. All of this follows the real emission physics: blue nitrogen is fast, green oxygen takes about 0.7 s, and red oxygen takes up to about two minutes ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)). Where a curtain folds and is seen edge-on, it brightens into a hot seam. On the drop a "substorm" hits: the curtains multiply, fold over, the rays sharpen and race along the arc, and the hem drops lower and turns violet. In a break the sky calms to dim, diffuse patches pulsing every 8 to 16 beats. The reference is the green, ray-filled curtain photo over Nordmo, Norway on [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Northern_lights_curtain_over_Nordmo_in_Bjerkvik,_Narvik,_Nordland,_Norway,_2023_September.jpg). The mood is vast, quiet and cold, with sudden bursts of life.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Sky | vertical gradient from #01030A (zenith) to #071426 (horizon) | Normal | 100 % | never pure 0: about 1–3 % keeps LED modules alive and gives depth |
| 2 | Stars | hashed point stars, 1–2 px, #CFDBFF, no twinkle | Add | 60–80 % | the stars stay still while the aurora moves, which is what sells real-time footage |
| 3 | Far curtain | curtain instance C, sits low (near the horizon), height ×0.5, rays ×2 finer, speed ×0.6 | Add (inside shader) | 45 % | parallax depth |
| 4 | Mid curtain | curtain instance B, height ×0.75, rays ×1.4 | Add (inside shader) | 70 % | off at storm < 0.35 |
| 5 | Near curtain | curtain instance A, hero, full height | Add (inside shader) | 100 % | always on |
| 6 | Afterglow | persistent float buffer in "species space" (N2, O-green, O-red), per-channel decay | inside P2 | — | gives the emission delays and the smear of motion |
| 7 | Colourise | species → RGB via palette, white cores, hue-preserving tonemap | inside P3 | — | swapping the palette never changes the physics |
| 8 | Horizon silhouette | black PNG/alpha clip (mountains, trees, city, crowd cam key) | Normal (alpha), black on top | 100 % | covers the aurora; it is a mask, not a colour |
| 9 | Foreground (optional) | type, line work, a logo | Add or Normal | 0–100 % | the aurora is mostly black, so this always reads |
| 10 | Post | bloom, then grain/dither | see compositing | — | bloom mostly lifts the hem |

## Techniques

**1. 2D curtain from a footprint curve (main method)**
Lawlor and Genetti treat the aurora as purely emissive. They factor it into a 2D electron-flux footprint times a height-dependent energy deposition function ([Lawlor & Genetti 2011](https://www.cs.uaf.edu/~olawlor/papers/2010/aurora/lawlor_aurora_2010.pdf)). We use the same split in screen space. Screen coords: `uv` in 0–1, `y` up, `x` scaled by the aspect ratio.
1. Hem line: `y_e(x,t) = base + Σ₂ aᵢ·sin(kᵢ·x + ωᵢ·t + φᵢ)`, with base 0.30 (near), 0.22 (mid), 0.15 (far). Use amplitudes 0.02–0.06, kᵢ 2–6 rad per frame width and ωᵢ 0.05–0.3 rad/s.
2. Altitude coordinate: `a = (y − y_e) / H`, where H is the curtain height (0.3–0.75 of the frame height). a = 0 is the hem. For physics thinking, read a = 0 as 100 km and a = 1 as about 300 km.
3. Coordinate along the curtain (the folded sheet): `s = x + Σ₃ Aᵢ·sin(κᵢ·x + Ωᵢ·t + ψᵢ)`. The fold strength is `F = max Σ Aᵢ·κᵢ`. When F < 1 the sheet only bends. When F > 1, `ds/dx` changes sign and the curtain folds back over itself.
4. Rays: `r = fbm(s·R, a·0.8 − t·v_ray)`, with 3 octaves of value or simplex noise, R = 20–120 rays per frame height, and an anisotropy of about 50:1 so the rays come out vertical. Sharpen with `r = r^p` (p = 1.5 soft, 4 storm). Each ray gets its own top: `top(s) = 0.6 + 0.4·noise1(s·R·0.3)`, and `r *= smoothstep(top, top − 0.25, a)`.
5. Edge-on brightening: looking through a sheet at a shallow angle crosses more of its thickness. Fake this with `B = 1 / max(|ds/dx|, 0.2)`, clamped to 4. Folds then light up as seams, the same Jacobian trick as the caustic chapter.
6. Flux footprint per pixel: `Φ = r · B · excitation(t)`.

**2. Vertical deposition and the sharp lower edge**
Lawlor and Genetti stretch the footprint into 3D with the Lazarev deposition model. It peaks just above the lowest penetration height and falls off upward. They pre-store it as a lookup table ([Lawlor & Genetti 2011](https://www.cs.uaf.edu/~olawlor/papers/2010/aurora/lawlor_aurora_2010.pdf)). The sharp hem comes from the rapid drop of atomic oxygen below about 100 km ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)).
1. Base profile: `D(a) = exp(−(a/e)²)` for a < 0 (below the hem), and `D(a) = (1 − exp(−a/e)) · exp(−a/λ)` for a ≥ 0. Here e = hem softness (0.004–0.02 of H, default 0.008) and λ = the upward fade (0.25–0.6, default 0.35).
2. Energy shifts the hem. In the Lazarev model, higher-energy particles get a larger characteristic shielding mass (it grows with E^1.65), so they reach deeper ([Lawlor & Genetti 2011](https://www.cs.uaf.edu/~olawlor/papers/2010/aurora/lawlor_aurora_2010.pdf)). On stage, the `energy` parameter lowers `base` by up to 0.08 and widens the N2 band.
3. Bake D and the species weights (technique 3) into a 256 × 1 RGBA lookup over a ∈ [−0.1, 1.5]. The paper does the same with a table for speed. In ISF you can also evaluate it inline, because it is cheap.

**3. Species bands by altitude (the colour backbone)**
The stack runs red on top, green in the middle and blue/purple at the bottom. Blue and purple show at the lower edge in high activity ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)). We compute three emission sources, not colours:
1. N2 (428 nm, hem): `w_N = exp(−max(a,0)/0.04) · D(a) · energy`.
2. O green (557.7 nm, body): `w_G = D(a)`.
3. O red (630 nm, crown): `w_R = smoothstep(0.35, 0.75, a) · exp(−max(a − 0.75, 0)/0.6) · crown`. The red uses its own ray field at ¼ of R and p = 1, so the crown stays diffuse, as in real photos.
4. Write `P = (w_N·Φ, w_G·Φ, w_R·Φ_red)` as the excitation rate per species. Colour comes later (P3), so palette swaps and band reversal are a single remap.
5. Sources disagree on the green altitude range (the research file lists two). These band edges are a stage choice, not a measurement.

**4. Emission delays (per-species afterglow)**
The real lifetimes are about 0.7 s for green and up to about two minutes for red. Blue is much faster ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)). Model each species as an excited population N that decays with its own τ.
1. Decay per frame: `k = exp(−Δt/τ)`, with Δt = TIMEDELTA.
2. N2: τ ≈ 0, so the emission is just P. The hem flashes and stops in the same frame.
3. Green, in "stage" mode (default): `E_G = max(E_G·k_G, P_G)`, an instant attack with an exponential release, τ_G = 0.7 s × lifetime. "Physical" mode is `N = N·k + P·Δt` with `E = N/τ`. It is softer and correct, but kicks look weaker.
4. Red, always physical: `N_R = N_R·k_R + P_R·Δt`, `E_R = gain_R · N_R / τ_R`, with τ_R = 4–30 s (default 12 s, about 6 bars at 128 BPM). The real ~2 min is too slow for a set, so `lifetime` scales it. Because red integrates, one kick barely shows. A long build turns the crown red, and it lingers through the break.
5. This is done per pixel in a persistent buffer, not with a global envelope. So when a curtain moves, its green leaves a 0.7 s smear and its red leaves a slow ghost. Real red aurora looks diffuse; this is a stage version of that look.
6. At 60 fps: k_G = 0.976, and k_R = 0.9986 for τ = 12 s. These factors only work in a float buffer (see Performance).

**5. Substorm (the drop) and pulsating recovery (the break)**
A quiet arc can turn into an active display within a few minutes, and pulsating aurora has periods of about 2–20 s ([Wikipedia: Aurora](https://en.wikipedia.org/wiki/Aurora)). Lawlor and Genetti describe a substorm as smooth curtains that grow, fold over, fragment and recover into dim pulsations ([Lawlor & Genetti 2011](https://www.cs.uaf.edu/~olawlor/papers/2010/aurora/lawlor_aurora_2010.pdf)).
1. `storm` 0 → 1 drives: fold strength F 0.3 → 1.6, ray sharpness p 1.5 → 4, ray scroll v_ray 0.03 → 0.4, curtain count 1 → 3, energy +0.3.
2. Onset: ramp storm over 1 bar with ease-out (cubic), so it snaps in and settles.
3. Recovery (break): storm falls to 0.1, and the rays are replaced by patches. Lower R to 6 and p to 1. Multiply Φ by `0.5 + 0.5·sin(2π·beat/period + 3·noise(s·4))`, with period 8 or 16 beats, so the patches pulse out of phase.

**6. Volumetric alternative (UE / Godot only)**
1. The footprint lives in a top-view texture (2048² RGBA16F). It is drawn each frame from the same s-curve and noise as technique 1, now in world xz.
2. Ray-march through a slab from 80 to 300 scene units (1 unit = 1 km). At each sample, add `footprint(xz) × LUT(altitude)` per species. There is no absorption, because the aurora is emissive.
3. Step count: Lawlor and Genetti step 2 km inside curtains and up to 1000 km in empty space, using a 2D distance field built with jump flooding. Without it, they measured +350 % cost. 1 km steps cost +60 % and 4 km steps −33 % ([Lawlor & Genetti 2011](https://www.cs.uaf.edu/~olawlor/papers/2010/aurora/lawlor_aurora_2010.pdf)). A simpler route is a fixed 30–64 steps with per-pixel jitter. The CC0 Godot aurora sky shader uses 30 steps with jitter ([Godot Shaders](https://godotshaders.com/shader/aurora-sky-shader/)).
4. Gain: true parallax and an overhead view (rays converging toward the zenith), and edge-on brightening for free. Cost: see Performance. Gaia Sky plans a lower-resolution off-screen buffer with blur and glow against jitter ([Toni Sagristà](https://tonisagrista.com/blog/2024/rendering-aurorae-nebulae/)). Do the same: render at half resolution, then blur and upscale.

## Shaders

Render order: P0 → P1 → P2 (persistent) → P3. In ISF all four are one file with PASSES. The fallback is described under Wire.

**P0 — Columns (1D curtain data)**
- Purpose: everything that depends only on x, computed once per column instead of once per pixel.
- Inputs: time t, storm, rays, energy, speed. Uniforms only.
- Per pixel (row j = curtain A/B/C): compute y_e(x), s(x), B = edge-on factor and Φ_col = the ray fbm at a = 0 (seed). Write RGBA = (y_e, s, B, ray top).
- Output: buffer `cols`, width = $WIDTH, height = 3, FLOAT. Cost is negligible.

**P1 — Excitation**
- Purpose: species excitation rate P per pixel.
- Inputs: `cols`, t, H, hem softness e, crown, energy, storm.
- Per pixel: for each active curtain, fetch its column row, compute a, D(a), the vertical ray modulation fbm(s·R, a·0.8 − t·v_ray), the ray top fade, and the three species weights (technique 3). Sum the curtains (far ×0.45, mid ×0.7) into RGB = (N2, O-green, O-red). A = hem mask `smoothstep(y_e − e, y_e, y)` of the near curtain.
- Output: buffer `excite`, $WIDTH/2 × $HEIGHT/2, FLOAT. The rays are vertical and soft, so half resolution loses nothing visible. The hem is restored at full resolution in P3.

**P2 — Afterglow (ping-pong / persistent)**
- Purpose: emission delays (technique 4).
- Inputs: `excite`, own previous frame `glow`, TIMEDELTA, lifetime, red gain, a `clear` event.
- Per pixel: R (N2) = excite.R. G = max(prev.G·k_G, excite.G). B = prev.B·k_R + excite.B·Δt. Clamp to 0–64 against runaway values. On `clear`, write 0.
- Output: buffer `glow`, $WIDTH/2 × $HEIGHT/2, PERSISTENT, FLOAT ([ISF multipass reference](https://docs.isf.video/ref_multipass.html)). TIMEDELTA is a built-in ISF uniform ([ISF variables](https://docs.isf.video/ref_variables.html)).

**P3 — Colourise and composite (full resolution)**
- Purpose: turn species into colour; add sky and stars; sharpen the hem.
- Inputs: `glow` (bilinear upsample), `cols`, palette colours, exposure, an optional silhouette texture.
- Per pixel:
  1. Recompute the near-curtain hem at full resolution from `cols`. Multiply the upsampled glow by `mix(1, hemMask, edgeLock)` (default 0.8), so the edge stays razor sharp while some lag shows.
  2. `c = E_N·C_N2 + E_G·C_green + (E_R/τ_R·gain)·C_red`.
  3. White cores: `c += C_core · smoothstep(1.2, 3.0, E_G)`.
  4. Hue-preserving tonemap: `L = length(c)`, `c *= (L/(1+L))/L`. Lawlor and Genetti rescale colour by its vector magnitude in the same way, for gamma, so bright cores keep their hue.
  5. Add the sky gradient and the stars. Multiply by (1 − silhouette alpha).
  6. Add triangular dither of ±1/255.
- Output: the final frame. Bloom is applied after this, in the host.

## Color scheme

| role | hex | notes |
|------|-----|-------|
| sky zenith (base) | #01030A | black level about 1 % |
| sky horizon (base) | #071426 | faint light-pollution glow behind the silhouette |
| silhouette (shadow) | #000000 | the only true black |
| N2 hem (accent) | #5B3BFF | violet-blue, standing in for 428 nm |
| N2 hem, pink variant | #FF5FB0 | for the "pink lower edge" look in photos |
| O green body (base glow) | #3DFF8F | photo-style green; #7CFF4F is closer to the spectral yellow-green |
| O red crown (accent) | #E8263A | 630 nm, kept darker than the green |
| core highlight | #D9FFE6 | where the green saturates |
| stars | #CFDBFF | cool white |

- **Mapping:** three fixed colour vectors weighted by species emission (technique 3), not a gradient ramp. This keeps the altitude order physical even when the curtain bends.
- **Non-physical palettes** (same order): "solar" = hem #29E0FF, body #FF2BD6, crown #FFB52E. "Ice" = hem #FFFFFF, body #59C8FF, crown #3B4BFF. Crossfade palettes over 4–8 bars, never on the beat.
- **Over time:** red only appears through the slow integrator, so colour follows the energy of the set by itself. Calm sections are green-only, long builds turn the crown red, and drops add violet hems.
- **LED walls:** green LEDs are the most efficient, so a full #3DFF8F field is harsh and clips. Cap the green emission at about 80 % (`exposure` 0.8) and let the bloom give the extra glow. Keep about 80 % of the frame below 5 % luminance, because the darkness carries the image. Contrast is high: sky about 1 %, body 40–70 %, hem and seams 100 %.

## Blending and compositing

- The curtains sum additively inside P1. Aurora light is emissive and does not absorb, so additive is physically right, and it is why overlapping folds brighten.
- Species are mixed in linear float space and colourised once in P3. Never blend already-coloured curtain layers with Screen; it washes out the hem.
- Feedback amount: green ≈ 0.976/frame and red ≈ 0.9986/frame at 60 fps. These are derived from τ, not set by hand. Always scale by TIMEDELTA, so a frame-rate drop on stage does not change the timing.
- Silhouette: a Normal-blended alpha layer on top, or a matte multiplied in P3. Feather it by 1 px only; real horizons are hard.
- Post chain, in order:
  1. Bloom, threshold 0.6, intensity 1.0–1.4, radius about 2 % of the frame height. It mainly lifts the hem and the fold seams.
  2. Optional extra soft blur on the red only, inside P3.
  3. Grain or dither, 2–3 %, to stop banding in the slow green fades.
- No chromatic aberration or sharpening. Both fight the soft, natural look.

## Motion and timing

- **Hem waves:** ω 0.05–0.3 rad/s, about 20–120 s per cycle at rest. Never sync to the beat; the slowness is the scale reference.
- **Fold drift** (Ωᵢ): 0.1–0.6 rad/s. At storm 1, multiply by 3.
- **Ray scroll along the arc** (v_ray): 0.03 frame widths/s calm, 0.4 in a storm. Flip direction every 32 beats (a phrase), easing over 4 beats.
- **Pulsation (break):** period 8 or 16 beats (3.75 s or 7.5 s at 128 BPM), inside the real 2–20 s range. Use 4 beats only at ≤ 120 BPM (≥ 2 s).
- **Kick:** an excitation pulse of about +0.8, attack 2 ms and release 120 ms, on the near curtain only. The hem flashes violet, and the green follows with a 0.7 s tail.
- **Drop:** storm 0 → 1 over 1 bar (ease-out cubic), plus a one-shot `energy` +0.3. Mid and far curtains fade in over 2 beats.
- **Break:** storm → 0.1 over 2 bars (ease-in-out). Switch to pulsating patches. The red crown from the build lingers for τ_R, which is the payoff of the emission physics.
- **Real-time versus time-lapse:** calm settings feel like real-time shimmer. Speed × 4 gives the familiar time-lapse sweep. Use it as a transition only.

## Parameters

| name | range | default | what it does on stage | suggested MIDI/BPM mapping |
|------|-------|---------|----------------------|---------------------------|
| storm | 0–1 | 0.2 | quiet arc to full substorm: folds, curtain count, ray sharpness, speed | BeatStep Pro knob 1; drop pad = ramp to 1 over 1 bar |
| energy | 0–1 | 0.3 | lowers the hem and widens the violet N2 band ("harder electrons") | bass RMS, smoothed; knob 2 as offset |
| rays | 6–120 (rays per frame height) | 40 | ray density, from soft patches to fine combs | knob 3; clamp to wall resolution (see Performance) |
| height | 0.3–0.75 (frame heights) | 0.5 | curtain height H, from short bands to towering sheets | bass level ±0.1; knob 4 |
| lifetime | 0.5–2 (×τ) | 1 | scales green τ (0.7 s) and red τ (12 s) together: snappy to dreamy | knob 5; double it in breaks |
| crown | 0–2 | 1 | red emission gain | KeyStep mod strip; slow fade-ins only |

## Build recipes

### Resolume Arena

- **Layer 1, sky:** Solid Color #01030A, plus a second solid #071426 with a soft gradient mask at the bottom. Or simply keep the gradient in P3.
- **Layer 2, stars:** a still image or a slow star clip in Add at 70 %. Use it if you build the shader without stars.
- **Layer 3, aurora:** the Wire patch (below) loaded as a source, blend mode Add. Resolume documents ISF only inside Wire, through the ISF node ([Resolume ISF](https://resolume.com/support/en/isf)), so the custom shader always goes through Wire. Publish storm, energy, rays, height, lifetime and crown to the clip panel.
- **Layer 4, silhouette:** a PNG with alpha (mountain or city line), Normal, 100 %. A live keyed crowd or DJ camera also works.
- **Layer 5, foreground:** type or line work in Add.
- **Composition effects:** Bloom, threshold about 0.6 and intensity about 1.2, then a light grain/noise effect from your set.
- **No-shader fallback:** real-time aurora footage on layer 3, with Hue Rotate kept at 0 and a Levels or Brightness/Contrast effect pushing the black point up to crush the noise. Use a duplicate layer with Blur in Add at 30 % for the glow. You lose the species timing, so keep the drop for the shader version.
- **Audio:** use Arena's audio analysis on the published parameters only as a backup. The envelopes belong in Wire (attack and release control).

### Resolume Wire

- **Clock:** `Time` (delta) × speed → an accumulator → ISF input `t`. Check what your Wire version calls these nodes. Pass TIMEDELTA through ISF's own uniform.
- **Main ISF node:** one multipass ISF with P0 `cols` (FLOAT, height 3), P1 `excite` (half size via `$WIDTH/2`, `$HEIGHT/2`), P2 `glow` (PERSISTENT, FLOAT) and P3 to the output ([ISF multipass reference](https://docs.isf.video/ref_multipass.html)). Float INPUTS with MIN/MAX for the six parameters, plus `kick` (float 0–1), `hat` (float), `palette` (float 0–2), `edgeLock` and a `clear` event ([Resolume ISF](https://resolume.com/support/en/isf)).
- **Version caveat:** the Resolume ISF page does not cover multipass or persistent buffers, and support depends on your Wire version. Test first that `glow` keeps its state across frames.
- **Fallback, two single-pass ISF nodes:**
  1. Node "Green" outputs only the green emission (P0 + P1 merged).
  2. Node "Red" outputs only the red emission.
  3. Each goes into its own Wire feedback/delay loop with mix = k (0.976 green, 0.998 red).
  4. A third ISF node (P3) takes Green, Red and an N2 output with no feedback, and colourises.
  5. Add a black floor of −1/255 per frame on the red loop if it is 8 bit (see Performance).
- **Audio:** `Audio Analysis` → low band → envelope (attack 2 ms, release 120 ms) → `kick`. The low band → envelope (attack 80 ms, release 400 ms) → `energy` and `height`. The high band → envelope (attack 1 ms, release 50 ms) → `hat`. In P1, hat adds `±0.15·hat` noise to the per-ray brightness (shimmer).
- **Drop:** a MIDI note from the BeatStep Pro → a ramp/envelope of 1 bar (from BPM) → storm.
- Wire has no vertex shaders ([Resolume ISF](https://resolume.com/support/en/isf)). That is no problem here, because everything is fragment work. The volumetric route is not worth it in Wire.

### Unreal Engine 5.7

Three routes, from cheap to expensive.

**A. 2D route (same as ISF).**
1. Two render targets `RT_GlowA` and `RT_GlowB`, RGBA16f at half resolution, plus `RT_Excite`.
2. In the Level Blueprint tick, run `Draw Material to Render Target`: excite material → RT_Excite, then afterglow material (reads RT_Excite + the previous Glow RT) → the other Glow RT. Swap the two each frame.
3. A final Unlit material on a screen-filling plane, or a post-process material, colourises.
4. Parameters live in a Material Parameter Collection `MPC_Aurora` (storm, energy, rays, height, lifetime, crown, kick, hat).

**B. Ribbon route (3D sheets, real parallax, cheap).**
1. One Niagara emitter per curtain. Spawn Burst Instantaneous of 300–600 particles along x.
2. A scratch pad module sets the position to (s-curve x, depth z from the fold function, hem height).
3. Ribbon Renderer, with Ribbon Width = curtain height and the facing set so the ribbon stands vertical (custom side vector = world up; check the mode name in 5.7).
4. Material: Unlit, Additive, two-sided. UV.v gives altitude a, which reads the species weights from the 256 × 1 LUT. U × R scrolled gives the rays. The edge-on gain is `1/max(|dot(N, V)|, 0.25)`.
5. Emission delays: keep it simple and drive the per-species brightness from Blueprint envelopes (green release 0.7 s, red integrator), rather than per-pixel.

**C. Volumetric route.**
1. An inverted box covering the "sky slab". Material: Unlit, Additive, with a Custom node ray-marching 48 steps with per-pixel jitter (technique 6).
2. The footprint render target is drawn from the s-curve in world xz. The species LUT is a 256 × 1 texture.
3. Optional: a jump-flood distance-field pass into a second render target for large empty-space steps, as in Lawlor and Genetti.
4. Render at reduced screen percentage and let TSR clean up the jitter.

**Control and output (all routes).**
- Create a Remote Control Preset that exposes the MPC scalars and the Niagara user parameters. Bind them to OSC from the Wire/Arena audio envelopes and to MIDI from the BeatStep Pro, through Remote Control's protocol bindings.
- Send the final frame to Arena by Spout or NDI with the plugin you already use, with the silhouette done in Arena or in-engine.

### Godot

- **2D route (Godot 4):**
  1. `SubViewport Excite` (half size, HDR 2D on) contains a `ColorRect` with a canvas_item shader (P0 + P1 merged; Godot handles per-pixel columns fine).
  2. `SubViewport GlowA` and `SubViewport GlowB` (half size, update mode Always). Each one's ColorRect shader reads the Excite ViewportTexture and the other Glow's texture, and applies P2.
  3. Show the texture of whichever buffer was written last. Toggle which one writes each frame from a script.
  4. The main scene is a full-screen `ColorRect` with the P3 shader, followed by a `WorldEnvironment` glow for the bloom.
- **Uniforms:** storm, energy, rays, height, lifetime, crown, kick, hat, palette_n2/green/red (`source_color`), edge_lock, a hem_softness float, and `TIME` from the engine. Pass delta from `_process(delta)`.
- **Volumetric route:** a `WorldEnvironment` Sky with a sky shader. Start from the CC0 aurora sky shader (30 steps, jitter, a colour-gradient sampler, brightness, speed, height and scale uniforms; a port of a Shadertoy original) ([Godot Shaders](https://godotshaders.com/shader/aurora-sky-shader/)). Replace its gradient with the three-species LUT, and modulate its brightness with the kick and red envelopes from GDScript.
- **Audio:** `AudioEffectSpectrumAnalyzer` on the music bus. Read the bands in `_process` and apply the attack/release envelopes in script.
- **Output:** Spout or NDI through the add-on you use, or capture the window in Arena.

## Audio reactivity

| band | signal | drives | smoothing | threshold |
|------|--------|--------|-----------|-----------|
| 40–120 Hz | onset (kick) | excitation pulse on the near curtain → instant N2 hem flash + green with 0.7 s × lifetime tail | attack 2 ms, release 120 ms | trigger when the envelope is > 1.4 × its 1 s running mean and above −40 dBFS |
| 40–160 Hz | RMS | energy (hem depth, violet width) and height ±0.1 | attack 80 ms, release 400 ms | gate below −45 dBFS |
| 300 Hz–2 kHz | RMS | sustained excitation into the red integrator (crown) | attack 1 s, release 4 s, then τ_R in P2 | none; the integrator already ignores single hits |
| 6–12 kHz | onset (hats) | per-ray shimmer, ±15 % brightness noise | attack 1 ms, release 50 ms | > 1.3 × running mean |
| BPM | clock | pulsation period (8/16 beats), ray direction flips (32 beats), drop ramp (1 bar) | — | — |

- The emission delays are the smoothing for the light itself. Keep the audio envelopes fast, and let P2 give green its 0.7 s and red its slow build. Do not double-smooth.
- Trigger the drop/substorm by hand (pad), not automatically. False triggers on a loud breakdown ruin the arc.

## Performance tips

- **2D route cost** (estimates; measure on your machine): P0 is negligible. P1 at half resolution (3 curtains × 3-octave fbm) is the main cost, still small at 1080p. P2 is one texture read and write. P3 is full resolution but cheap. Turning off the mid and far curtains at low storm saves about 60 % of P1.
- **Volumetric cost:** Lawlor and Genetti measured 60 fps at 720p on a GTX 280 with distance-field stepping. Naive stepping cost +350 % ([Lawlor & Genetti 2011](https://www.cs.uaf.edu/~olawlor/papers/2010/aurora/lawlor_aurora_2010.pdf)). At 4K the per-pixel step count dominates, so march at half or quarter resolution and upscale. Only pay this cost if the camera moves through or under the aurora. For a fixed frontal LED view, the 2D route looks the same.
- **Float buffers are required for red:** with k = 0.9986, an 8-bit value of 10/255 × 0.9986 rounds back to 10/255, so the ghost never fades. Use FLOAT/16f buffers. In 8-bit fallbacks, subtract a 1/255 floor per frame or update the red only every 8th frame with k⁸.
- **Ray aliasing on coarse LED pitch:** keep each ray at least 3 LED pixels wide, so `rays ≤ wall_height_px / 6`. Otherwise rays crawl and shimmer. Project Aurora used only 34,560 LEDs ([Nordic Museum](https://nordicmuseum.org/exhibitions/project-aurora)). On low-resolution walls, favour fewer, wider rays and stronger folds.
- **Odd aspect ratios:** define the ray density per frame height, not width, so wide strips (e.g. 6:1) get more rays instead of stretched ones. Wide walls suit 2–3 curtains side by side with different base heights. Tall portrait walls want H up to 0.75 and the hem low (base 0.15).
- **Banding:** slow green gradients on dark backgrounds band badly on 8-bit LED processors. Keep the P3 dither on, and test the black level on the actual processor, because some crush the first 2–3 % to black.

## Further references

- [Lawlor & Genetti, Interactive Volume Rendering Aurora on the GPU (2011, PDF)](https://www.cs.uaf.edu/~olawlor/papers/2010/aurora/lawlor_aurora_2010.pdf): the footprint × deposition factorisation, the Lazarev deposition formula, distance-field stepping with jump flooding, and the performance tables used above.
- [Toni Sagristà: Rendering aurorae and nebulae](https://tonisagrista.com/blog/2024/rendering-aurorae-nebulae/): why a flat mesh and a noise box were dropped for a volume ray-caster in Gaia Sky, footprints as cubemaps with SDFs, and the low-resolution, blur and glow plan.
- [Godot Shaders: Aurora sky shader](https://godotshaders.com/shader/aurora-sky-shader/): a CC0 Godot 4 sky shader with 30 jittered march steps and a gradient sampler. A quick start for the volumetric route.
- [ISF Multi-Pass and Persistent Buffer Reference](https://docs.isf.video/ref_multipass.html): PASSES, TARGET, PERSISTENT, FLOAT and size expressions, the basis of P0–P3 in Wire.
