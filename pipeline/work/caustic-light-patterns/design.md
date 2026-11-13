# Caustic Light Patterns — design

## Look

Picture the floor of a sunlit pool, filmed from straight above and turned into pure light. A deep blue-black field is covered by a net of thin, near-white filaments. They bunch into bright cusps and knots, split apart and cross again. Between them lie soft teal pools of lower light. Two nets move together: a large, slow one in front and a finer, faster one behind it, so the image has depth without any 3D. On a kick a ring-shaped ripple spreads from a point and bends the net as it passes. On the drop the light "focuses": soft blobs snap into razor lines, the edges split into thin red and blue fringes, and the tint shifts from cold teal to warm sunlight. For the reference, think of the shallow-water photo on [Atmospheric Optics](https://atoptics.co.uk/fz776.htm), where the bright lines are a 2D slice through moving 3D caustic sheets. The calm, architectural feel comes from [Loop.pH's Caustic Rays](https://www.kristallwelten.swarovski.com/en/artworks/caustic-rays). As an optional finale, the filaments gather into a word or logo, like Jussi Ängeslevä's [River is...](https://angesleva.iki.fi/art/river-is/). The mood is hypnotic, liquid and clean. The motion never repeats, but the pattern always looks organised.

## Layer stack

| # | layer | content / technique | blend mode | opacity | notes |
|---|-------|---------------------|-----------|---------|-------|
| 1 | Background | flat #02080C with a very slow vertical gradient to #041820 at the top (sunward side) | Normal | 100 % | not pure black: keeps LED modules alive and gives "water" volume |
| 2 | Deep pool tint | caustic layer B (see 4), heavily blurred (radius about 3 % of the frame height), coloured #0B5A6B | Add | 40 % | soft pools of light under the lines; keeps the frame from looking like wireframe |
| 3 | Caustic B (far) | small scale (scale 2.2×), speed 1.4×, rotated 30°, focus 0.7× of layer 4 | Add | 45 % | parallax layer; the rotation hides any tiling of layer 4 |
| 4 | Caustic A (near) | main caustic net, scale 1×, full focus, optional RGB prism split | Add | 100 % | hero layer |
| 5 | Ripple field | not a visible layer: ring impulses added to the height field used by 3 and 4 | (input) | — | kicks become visible as rings bending the net |
| 6 | Temporal smooth | feedback of layers 3+4: `out = mix(current, previous, 0.25)` | inside feedback buffer | — | removes flicker on thin lines; 0 on drops for crisp snaps |
| 7 | Reveal mask | logo/word, white on black, blurred 2–6 px | Multiply (by opacity `reveal`) plus a blurred copy in Add at `reveal · 0.3` | 0–100 % | River is... style payoff; off most of the set |
| 8 | Post | bloom, then light grain, then vignette | see compositing | — | order matters |

## Techniques

**1. Iterated sine-warp caustic (cheap fake, works everywhere)**
This is the well-known Shadertoy effect (MdlXz8), ported to Godot on [Godot Shaders](https://godotshaders.com/shader/water-caustic/). Here is what it computes:
1. Tile space: `p = mod(uv · TAU, TAU) − 250`, and set `i = p`. Fix the aspect first (`uv.x *= width/height`) so the cells stay round. Multiply uv by `scale` to set how many tiles show.
2. Run the loop 5 times (3–7 is usable): warp `i = p + (cos(t − i.x) + sin(t + i.y), sin(t − i.y) + cos(t + i.x))`. Then add `1 / length(p.x / (sin(i.x + t)/k), p.y / (cos(i.y + t)/k))` to an accumulator c, with k = 0.005 and c starting at 1.
3. Average: `c /= iterations`, then `c = 1.17 − c^1.4`.
4. Sharpen with `v = |c|^8`. The exponent is the line-sharpness control: 4 gives soft blobs, 8 the default net, 14 hairlines.
5. Why it works: every iteration adds a field of thin ridges where the warped coordinates pass zero. Where several ridges overlap you get the bright crossings that real caustics show.
6. Licence: Shadertoy code is CC BY-NC-SA 3.0 unless the author says otherwise (noted on the Godot Shaders page). For paid gigs, use technique 2, which is built from first principles here, or get permission.

**2. Height-field refraction caustic (physically motivated, our main method)**
A wavy surface acts like many small lenses. Rays bunch up where neighbouring rays cross, and the floor shows a slice through those caustic sheets ([Atmospheric Optics](https://atoptics.co.uk/fz776.htm)). We compute how strongly the surface squeezes the light:
1. Height field: `h(p,t) = Σ aᵢ · sin(kᵢ·p − ωᵢ·t + φᵢ)` with 6 waves. Pick directions spread over 180°, with wavelengths from 0.12 to 0.6 frame heights. Set amplitude in proportion to wavelength (aᵢ = 0.04 · λᵢ) and use `ωᵢ ∝ sqrt(|kᵢ|)`, so long waves move slower, as in deep water. Then add the ripple impulses (technique 3).
2. Gradient and Hessian, analytically, for each wave: `∇h += a·k·cos(θ)` and `H += −a·(k kᵀ)·sin(θ)`, where θ is the wave phase. No texture lookups and no finite-difference noise.
3. Refraction (small-angle): a vertical ray that hits the surface at p reaches the floor at `q = p − D · ∇h`. D is "focus": the floor depth times about (1 − 1/1.33). GPU Gems also uses IOR 1.33 for water ([GPU Gems ch. 2](https://developer.nvidia.com/gpugems/gpugems/part-i-natural-effects/chapter-2-rendering-water-caustics)).
4. Light concentration is the inverse area change of the mapping: `J = I − D·H` and `det J = (1 − D·hxx)(1 − D·hyy) − (D·hxy)²`. Intensity `E = 1 / max(|det J|, ε)`, with ε = 0.05 (range 0.01–0.2; lower = sharper, harsher lines).
5. Gather version (single fragment pass, works in ISF/Wire): show E at p. You are drawing the curves on the surface where det J = 0. They look almost identical to the floor caustics for small D, and you need no scatter. Tone with `v = clamp(E / Emax, 0, 1)^γ`, where Emax = 1/ε and γ = 1.5.
6. Scatter version (true caustic, UE/Godot only): take a dense grid mesh (512 × 512 vertices), move each vertex to q, and render with additive blending under an orthographic camera. In the pixel shader, intensity = `|ddx(p₀) × ddy(p₀)| / pixelArea`, where p₀ is the undisplaced position passed as an interpolator. Each screen pixel covers a fixed floor area, so the source area that lands on it is the brightness. Evan Wallace's [WebGL Water](https://madebyevan.com/webgl-water/) renders caustics with the help of screen-space derivatives (the page notes they need OES_standard_derivatives).

Stage meaning of D: sweeping D from 0 to its max moves the floor through the caustic sheets. You get soft patches, then sharp lines, then crossed nets. This is the "focus" fader.

**3. Beat ripples (impulse height field)**
1. Keep 4 ripple slots, each with centre c, start time t₀ and amplitude A. On every kick, a counter (wrap 4) overwrites the oldest slot.
2. Ripple height: `r = |p − c|` (aspect-corrected), `τ = t − t₀`, `h_ripple = A · sin(κ·(r − v·τ)) · exp(−τ/T) · smoothstep(v·τ + w, v·τ, r)`. The last factor keeps the ring only up to its wavefront. Use κ = 40 rad per frame height, v = 0.6 frame heights/s, T = 0.9 s, w = 0.05.
3. Add its gradient and Hessian, derived from the same expression, to technique 2. In technique 1, add `D_r · ∇h_ripple` to uv before the loop instead, so the net bends.
4. Centre choice: random inside the middle 60 % of the frame, or fixed at the centre on the drop.

**4. Two-scale parallax**
1. Evaluate the caustic twice, once at `p` and once at `R(30°) · p · 2.2 + offset`.
2. Run the far layer at speed 1.4× and focus 0.7×. Smaller, faster and softer reads as deeper.
3. The far layer also feeds the blurred pool tint (layer 2), so the bright lines and the soft pools stay in sync.

**5. Prismatic split (dispersion)**
1. Evaluate layer A three times, with D scaled per channel: R = D · (1 − s), G = D, B = D · (1 + s), where s = prism (0–0.08).
2. Write each result into its own channel. Where the lines cross you get white cores with red and blue fringes, like sun through a glass of water.
3. Cost is 3× layer A, so keep it off when the GPU is tight. At s above 0.08 the fringes separate into three nets and the look turns to glitch.

**6. Hidden word reveal (after River is...)**
1. Input a logo/word mask M (white on black), blurred by 2–6 px into M_b.
2. Inside the mask, lower the wave amplitude, `a_eff = a · (1 − 0.6·reveal·M_b)`, so the letters become calm bright pools. Then multiply the final light by `mix(1, 0.15 + M_b, reveal)`.
3. Raise reveal over 1–2 bars before the drop. The net seems to collect into the letters. At reveal 1 only the word glows, still filled with moving caustics.
4. This is a stage fake, not true inverse caustic design (Schwartzburg et al. solve that offline with optimal transport, as noted on [Wikipedia](https://en.wikipedia.org/wiki/Caustic_(optics))).

## Shaders

Render order per frame: P0 height field (optional), P1 caustic (layers A + B), P2 temporal smooth, P3 composite + post. In the Wire/ISF build, P0 and P1 are merged into one single-pass shader.

**P0 — Height field (optional, only for the scatter build)**
- Purpose: compute h, ∇h, H once and share them between the scatter mesh and other effects (displacing other clips).
- Inputs: uniforms for time (accumulated, see Motion), the six wave tables (dir, λ, a, φ), amplitude, the four ripple slots (cx, cy, t₀, A), reveal, and the mask texture.
- Per pixel: sum the waves and ripples, then write R = h, G = hx, B = hy, A = hxx + hyy (the Laplacian, also useful as a cheap sparkle mask).
- Output: RGBA16F at 1024 × 1024 (tileable domain) or half output resolution.

**P1 — Caustic**
- Purpose: produce the light pattern for the near and far layers.
- Inputs: P0 (or inline waves), uniforms `scale`, `focus` (D), `eps`, `gamma`, `prism`, `farScale` 2.2, `farAngle` 30°, `farSpeed` 1.4, `mode` (0 = sine-warp, 1 = height-field), `iterations`, `sharp` (exponent), `ripple` slots.
- Per pixel: (1) aspect-correct uv, add the ripple offset; (2) near layer, via technique 2 (or 1), three times when prism > 0; (3) far layer, via the same method at its transformed coordinates, single channel; (4) write RGB = near (per channel), A = far.
- Output: RGBA16F at full output resolution. Thin lines need full resolution; at half resolution they turn to mush.

**P2 — Temporal smooth (ping-pong)**
- Inputs: P1, previous P2, `smooth` (0–0.5).
- Per pixel: `out = mix(P1, prev, smooth)`. Default 0.25; set it to 0 during drops so the snaps stay crisp.
- Output: RGBA16F, two buffers swapped each frame.

**P3 — Composite and post**
- Inputs: P2, mask M, palette uniforms, `intensity`, `tintAmount`, `reveal`.
- Per pixel: background gradient; add the far layer (A channel) × #0B5A6B × 0.45; add the blurred far layer (pool tint, a 9-tap separable blur, or a mip sample of P2 at level 4) × 0.4. Then add the near layer: for luminance v use `mix(lineColour, highlight, v²)`, and with prism use the per-channel values × lineColour. Multiply by the reveal term, then by intensity.
- Post: bloom (threshold 0.8, intensity 1.3, medium radius), grain 2 % monochrome, vignette 0.2, then clamp to 8-bit for Spout/NDI.

## Color scheme

| role | hex | use |
|------|-----|-----|
| black / base | #02080C | background, LED black level |
| water top | #041820 | top of background gradient |
| pool tint | #0B5A6B | far layer and blurred pools (close to the Godot default tint rgb 0, 0.35, 0.5 = #005980) |
| line colour | #9FEFFF | filament body, cold |
| highlight | #FFFFFF | filament crossings and cusps (v² term) |
| sun accent | #FFC86B | drop palette: line colour moves here, pools to #5A3A12 |
| prism fringes | from the RGB split | red and blue edges, no extra colour needed |

- Mapping: a two-step ramp per pixel, `base → line colour → highlight`, driven by caustic luminance. The highlight only appears where v > 0.7, so white stays rare and precious.
- Over time: rotate the hue of line colour and pool tint by ±10° per 16-bar phrase (teal ↔ cyan ↔ blue-green). Stay inside the water family until the drop.
- Music: on the drop, crossfade over 1 beat to the sun palette (line #FFC86B, pool #5A3A12, highlight stays #FFFFFF). Back to water on the break.
- LED walls: hold the black at #02080C. Cap the near layer at about 90 % before bloom, so dense crossings do not clip into flat white blobs. Contrast is high by design: lines cover less than 15 % of the frame, so average power draw stays low, even at full intensity.

## Blending and compositing

- **Everything above the background is Add.** Caustics are light on a dark floor, and additive blending makes crossings brighter, as in reality. Screen is an acceptable alternative if a layer clips too fast.
- **Near over far:** both Add. The far layer stays at 45 % so it never competes with the hero net.
- **Reveal:** a Multiply with the mask at opacity = reveal gives exactly `caustic · mix(1, mask, reveal)`. A blurred copy of the mask in Add at reveal · 0.3 gives the glowing pool behind the letters.
- **Feedback mix:** 0.25 by default (range 0–0.5). Above 0.5 the motion goes syrupy and the ripples smear.
- **Post chain in order:** bloom (threshold 0.8, intensity 1.3) → grain 2 % → vignette 0.2 → optional sharpen 0.15 on coarse walls (pitch 4 mm and up). Skip a separate chromatic aberration effect: the prism split already does that, and at the right place (in the caustic, not on the frame edge).
- **As a source for other clips (mapped use):** output the near layer alone, in white, as a luma mask or light pass over projection-mapped architecture. Or use ∇h (P0 G/B) as a displacement map on video, which makes the video look underwater.

## Motion and timing

- **Time accumulation:** never write `t = TIME · speed`, because moving the speed fader then makes the pattern jump. Accumulate `t += speed · dt` each frame (in the host, then pass t as a uniform). Reversing and freezing then stay smooth.
- **Tempo lock:** speed in units per beat; default 0.25 (one time unit per bar at 4/4). Range 0 to 1 per beat. Far layer at 1.4×.
- **Easing:** speed and focus changes glide over 1/2 beat, with exponential smoothing at a time constant of about 120 ms.
- **Kick:** spawn a ripple (technique 3) and pulse intensity +25 %, decaying over 200 ms.
- **Every bar:** drift the focus a little (±5 %) on a slow sine with a 4-bar period, so the net breathes even without audio.
- **Build:** over the last 8 bars, raise focus from 0.5 to 1.2× default (lines tighten and multiply), raise the ripple rate to every 8th note, and ramp reveal from 0 to 1 if the logo is used.
- **Drop:** hard cut. Temporal smooth 0, a big ripple from the screen centre (A = 2×), prism to 0.05, sun palette crossfade over 1 beat, speed doubled for 2 bars, then back to normal.
- **Break:** speed to 0.3× and focus to 0.5× (soft patches), far layer off, prism 0, reveal 0. Optional freeze: speed to 0 over 1 beat. Reversal (speed −0.25) on a 2-bar breakdown looks like water flowing back.

## Parameters

| name | range | default | what it does on stage | suggested MIDI / BPM mapping |
|------|-------|---------|----------------------|------------------------------|
| speed | −1 to 1 units/beat | 0.25 | how fast the water moves; negative = reverse | KeyStep knob; BPM-locked, Shift = freeze |
| focus | 0–2 (× default D) | 1.0 | soft light patches (low) → sharp lines → dense crossing nets (high) | fader; automated in builds |
| scale | 0.5–4 | 1.0 | size of the cells; low = big lazy swells, high = fine shimmer | knob |
| ripple | 0–2 (ring amplitude) | 1.0 | how strongly kicks bend the net | kick trigger from BeatStep Pro pad or audio; knob for amount |
| prism | 0–0.08 | 0 | red and blue fringes on the filaments | knob; jump to 0.05 on drops |
| reveal | 0–1 | 0 | the net gathers into a word or logo | fader, ramp over 1–2 bars before a drop |

## Build recipes

### Resolume Arena

- **Layer 1:** Solid Color source, #02080C. For the gradient, add a second solid layer at #041820 with a Gradient-style mask, or simply keep it flat.
- **Layer 2 (pool tint):** a duplicate of the caustic source with the built-in Blur effect (large radius) and Colorize to #0B5A6B, Add at 40 %.
- **Layer 3 (far):** the caustic source with Transform scale 220 %, rotation 30°, and speed parameter 1.4×. Add at 45 %.
- **Layer 4 (near):** the caustic source itself, Add 100 %.
- **The caustic source:** a single-pass ISF shader with P0 + P1 merged (techniques 1 and 2, mode switch, prism, 4 ripple slots). Resolume documents ISF only inside Wire, via the ISF node ([Resolume ISF support](https://resolume.com/support/en/isf)), so build it as a Wire patch and load the patch in Arena as a source. Expose speed, focus, scale, ripple, prism and mode as parameters. Feed accumulated time from Wire, not from a raw clock.
- **Layer 5 (reveal):** the logo clip in Multiply blend mode, with the layer opacity mapped to reveal. Layer 6: the same logo with Blur, Add, opacity = reveal · 0.3.
- **Composition effects:** Bloom (threshold about 0.8, intensity about 1.3), then Vignette.
- **No-shader fallback:** a pre-rendered caustic loop (rendered from Godot or UE at the wall resolution) on layers 3 and 4 at different playback speeds and scales. The two loops never line up, which hides the loop point.
- **Audio:** Arena's audio analysis, low band → the ripple trigger and intensity; BPM sync on speed.

### Resolume Wire

Node outline (single-pass, gather version; Wire has no vertex shaders, so the scatter version is not possible here):
- `Time` (delta) × speed → an accumulator (an add-to-self or feedback loop on a number) → uniform `t`. Check what your Wire version calls these nodes.
- `BPM` / beat trigger → `Counter` (wrap 4) → selects which ripple slot gets `t₀ = t` and a random centre (`Random` node sampled on the trigger). Four sample-and-hold values per slot → the ISF inputs `r0..r3` (x, y, t₀, A).
- `Audio Analysis` → low band → `Envelope` (attack 5 ms, release 200 ms) → a threshold → the ripple trigger; the same envelope → intensity +25 %.
- `ISF` node (the caustic shader, near layer RGB + far layer A) → `Feedback`/delay node mixed at 0.25 (temporal smooth) → split: one path to a blur + colour (pool tint), one to the composite → `Blend` (Add) stack → bloom chain (threshold, blur, add) → output.
- `Texture` input for the logo mask → into a second ISF input, or a `Blend` (Multiply) at reveal.
- Exposed parameters: speed, focus, scale, ripple, prism, reveal, mode.
- Multipass ISF support in Wire depends on the version, so keep the shader single-pass and do the smoothing with Wire's own feedback.

### Unreal Engine 5.7

- **Main material** `M_Caustic` (Unlit, Surface, Additive). For a screen-filling VJ output use a Post Process material instead. A Custom node (HLSL) evaluates technique 2: six waves, ripple slots and the det J intensity. Read the parameters from a **Material Parameter Collection** `MPC_Caustic` (scalars: T, Speed, Focus, Eps, Scale, Prism, Reveal, RippleN_X/Y/T0/A). Emissive = the ramp from the colour scheme.
- **Mapped and 3D use:** make a second material in the **Light Function** material domain, using the same Custom node, monochrome. Assign it to a Spot Light or Rect Light above the scene: it projects animated caustics onto any geometry (stage models, mapped buildings). Light functions are greyscale, so prism only works in the post or surface version.
- **True scatter (optional, sharpest):** a 512 × 512 subdivided plane with `M_CausticScatter`. World Position Offset moves the vertices by −D·∇h. A Vertex Interpolator carries the undisplaced position p₀. Emissive = `|cross(DDX(p₀), DDY(p₀))| / pixelArea`, Additive. Capture it with a top-down orthographic **SceneCapture2D** into a render target `RT_Caustic` (RGBA16F), then sample RT_Caustic in the light function or post material.
- **Niagara (optional sparkle):** a GPU sprite emitter samples RT_Caustic and spawns short-lived (0.2 s) tiny sprites where luminance > 0.85: hi-hat-driven glints on crossings.
- **Remote Control:** a preset exposing the MPC scalars. Remote Control supports OSC, MIDI and DMX through protocol plugins ([Epic docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-protocols-in-unreal-engine)). Map BeatStep Pro knobs via MIDI, and kicks and the envelope via OSC from Resolume or an audio analyser. Accumulate T in Blueprint (Tick: T += Speed · DeltaSeconds) and write it to the MPC.
- **Post-process volume:** Bloom intensity 1.3, threshold 0.8; manual exposure (no eye adaptation, or kicks pump the image); Film Grain 0.02; Vignette 0.2.
- **Output:** Spout via a third-party plugin, or NDI via the NDI SDK for Unreal, at the LED processor's native resolution.

### Godot

- **2D VJ output:** a `ColorRect` filling the viewport with a `canvas_item` shader, techniques 1–5 in the fragment function. Uniforms: `t, focus, eps, gamma, scale, prism, far_scale, far_angle, far_speed, mode, iterations, sharp, ripples (vec4 array[4]), reveal, mask (sampler2D), line_color, pool_color, highlight`. Accumulate `t` in GDScript `_process(delta)`.
- **Temporal smooth:** render the ColorRect inside a `SubViewport` (HDR 2D on, `render_target_clear_mode` Never). On top, draw a second `ColorRect` whose shader mixes a `ViewportTexture` copy of the previous frame at 0.25. Or skip smoothing and rely on lower `sharp` values on LED.
- **Scatter version (3D):** a `SubViewport` with an orthographic `Camera3D` looking down at a `MeshInstance3D` (PlaneMesh, subdivide 511 × 511). Use a spatial shader with `render_mode unshaded, blend_add, cull_disabled`. `vertex()` offsets VERTEX.xz by −D·∇h and stores p₀ in a varying. `fragment()` sets ALBEDO = intensity from `length(cross(dFdx(p₀), dFdy(p₀)))`. Use the `ViewportTexture` as the caustic map.
- **Mapped and 3D use:** assign that ViewportTexture as the `light_projector` of a `SpotLight3D` above the scene, or as the emission texture of a `Decal`.
- **Post:** `WorldEnvironment` glow (for 2D, set the background mode to Canvas), intensity 1.3, HDR threshold 0.8.
- **Control:** `InputEventMIDI` → uniforms. For Spout, use a community Spout addon if one is available for your Godot version; otherwise use NDI via screen capture.

## Audio reactivity

| band | drives | smoothing | threshold |
|------|--------|-----------|-----------|
| kick, 40–100 Hz onset | spawns a ripple; intensity +25 % | attack 2 ms, release 200 ms | onset at +6 dB over the 1 s average; at most one ripple per 1/8 note |
| bass, 60–250 Hz level | wave amplitude 0.8–1.3× (more crossings, tighter filaments) | attack 60 ms, release 500 ms | none, continuous |
| mids, 300 Hz–2 kHz | focus ±10 % around the fader value | attack 150 ms, release 1 s | none |
| hi-hats, 6–12 kHz | a small extra wave (λ = 0.04 frame heights, amplitude 0–0.3×) for shimmer, plus Niagara glints | attack 1 ms, release 80 ms | gate at −30 dBFS |

Keep only the kick fast. The water should breathe with the bass, not twitch. If more than two parameters move per beat, the pattern loses its calm, organised quality.

## Performance tips

- **Sine-warp (technique 1):** 5 iterations is about 30 trig calls per pixel, trivial at 4K. Each extra iteration adds detail, but past 7 it only makes noise.
- **Height-field gather (technique 2):** 6 waves plus 4 ripples, with gradient and Hessian, costs about 40 trig calls per layer. Prism triples the near layer. Total stays under 1 ms at 4K on a mid-range GPU.
- **Scatter mesh:** 512² vertices = 262k vertices, fine everywhere. Use 1024² for walls above 4K, or you will see faceting on the sharpest cusps.
- **Aliasing:** the lines are thinner than a pixel at high focus. Temporal smooth (0.25) or 2× supersampling of P1 stops the crawling. On coarse LED walls (pitch 4 mm and up), lower `sharp` to 5–6 or raise eps to 0.1 so the lines are at least 2 px wide.
- **Tiling:** the sine-warp fake repeats every TAU, so at high scale the grid shows. The rotated far layer hides it. The height-field method does not tile if the wave directions are irrational multiples of each other.
- **Odd aspect ratios:** always aspect-correct uv and ripple distances, or the cells stretch on 32:9 strips. For very wide strips, increase scale on x only by up to 1.3× before it looks stretched.
- **Pitfalls:** `TIME · speed` jumps when the speed fader moves; eps too small gives single-pixel fireflies; auto-exposure in UE pumping on kicks; bloom threshold below 0.7 floods the whole frame cyan.

## Further references

- [Water Caustic — Godot Shaders](https://godotshaders.com/shader/water-caustic/): the full sine-warp caustic shader with speed, tint, intensity and iteration count (port of Shadertoy MdlXz8, licence note on the page).
- [GPU Gems, Chapter 2: Rendering Water Caustics](https://developer.nvidia.com/gpugems/gpugems/part-i-natural-effects/chapter-2-rendering-water-caustics): wave normals, Snell refraction with IOR 1.33, and a per-pixel GPU version with additive blending.
- [Evan Wallace — WebGL Water](https://madebyevan.com/webgl-water/): an interactive real-time water demo with caustics, ripples you can draw, and a movable light.
- [Cyanilux — Water Shader Breakdown](https://www.cyanilux.com/tutorials/water-shader-breakdown/): the Voronoi-based caustic fake (animated angle offset, power 5, world-space xz projection) for Shader Graph users.
