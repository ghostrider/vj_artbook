---
tags:
  - optics
  - light
  - water
  - shader
---

# Caustic Light Patterns

> The bright, drifting net of light at the bottom of a pool, turned into a calm, liquid layer for the wall.

![Bright, wobbling caustic light lines on the floor of a turquoise pool, seen through moving water](https://upload.wikimedia.org/wikipedia/commons/thumb/7/7a/Caustic_light_-_Flickr_-_fdecomite.jpg/1280px-Caustic_light_-_Flickr_-_fdecomite.jpg)
*Bright, wobbling caustic light lines on the floor of a turquoise pool, seen through moving water. Image: [fdecomite](https://commons.wikimedia.org/wiki/File:Caustic_light_-_Flickr_-_fdecomite.jpg), [CC BY 2.0](https://creativecommons.org/licenses/by/2.0).*

## The idea

A caustic is the envelope of light rays reflected or refracted by a curved surface, or its projection onto another surface. The word comes from the Greek for "burnt". Rainbows are caustics too ([Wikipedia](https://en.wikipedia.org/wiki/Caustic_(optics))).

The pool-floor version is the one we know. A wavy surface acts like many small curved lenses. Refracted rays bunch up at the edges of regions where they cross, so those edges are very bright and sharp. The floor pattern is a 2D slice through moving 3D caustic sheets ([Atmospheric Optics](https://atoptics.co.uk/fz776.htm)).

On a club screen it works as a quiet texture. Thin filaments flicker and drift. They never repeat, yet they always look organised: cells, cusps, crossing nets. It reads as water and as architecture, so it suits walls, floors and mapped surfaces. It needs dynamics to stay alive: a kick that bends the net, a second scale for depth, a drop that snaps the light into focus.

## Origins and references

- [Caustic (optics), Wikipedia](https://en.wikipedia.org/wiki/Caustic_(optics)): definition, and the note that games often use pre-computed textures instead of simulation.
- [Atmospheric Optics, caustics](https://atoptics.co.uk/fz776.htm): why the lines are sharp, and why only seven stable elementary forms exist (catastrophe theory).
- [Mark Watt, "Water Caustics"](https://history.siggraph.org/?p=65616): a 30-second animation shown at the SIGGRAPH 1991 Electronic Theatre.
- [GPU Gems, ch. 2: Rendering Water Caustics](https://developer.nvidia.com/gpugems/gpugems/part-i-natural-effects/chapter-2-rendering-water-caustics): real-time water caustics (Guardado, 2004).
- [River is..., Jussi Ängeslevä (2012)](https://angesleva.iki.fi/art/river-is/): a 23 m² chrome sculpture, computationally designed and CNC-milled. Two moving lights make its reflections spell words on the wall.
- [Caustic Rays, Loop.pH](https://www.kristallwelten.swarovski.com/en/artworks/caustic-rays): curved reflective filters and programmable LEDs recreate shifting water light over an escalator at Swarovski Kristallwelten Store Wien.
- [Naturally Focused, Alison Stott](https://physicsworld.com/a/caustic-light-patterns-inspire-new-glass-artwork/): a hand-blown glass lens in gyroscopic rings that projects caustics (reported Nov 2025).
- Inverse "caustic engineering", designing a surface that casts a chosen image, was formalised with optimal transport, for example by Schwartzburg et al. in 2014 ([Wikipedia](https://en.wikipedia.org/wiki/Caustic_(optics))).

## Visual language

Picture the floor of a sunlit pool, seen from straight above and turned into pure light. A blue-black field (#02080C) carries a net of thin, near-white filaments. They bunch into bright cusps and knots, split and cross again. Between them lie soft teal pools of lower light.

Two nets move together. A large, slow one sits in front. A finer, faster, softer one sits behind it and is rotated 30 degrees. Smaller, faster and softer reads as deeper, so the image has depth without any 3D.

On a kick, a ring ripple spreads from a point and bends the net as it passes. On the drop the light focuses: soft blobs snap into razor lines, the edges split into thin red and blue fringes, and the tint moves from cold teal (#9FEFFF lines) to warm sun (#FFC86B).

White stays rare. Highlights only appear at the brightest crossings. Lines should cover less than 15 % of the frame. The mood is hypnotic, liquid and clean.

## See it

- [River is... — Jussi Ängeslevä](https://angesleva.iki.fi/art/river-is/) — chrome sculpture projecting caustic words onto a wall, documentation images
- [Caustic Rays — Loop.pH / Swarovski](https://www.kristallwelten.swarovski.com/en/artworks/caustic-rays) — ceiling caustic light installation
- [Naturally Focused — Physics World](https://physicsworld.com/a/caustic-light-patterns-inspire-new-glass-artwork/) — glass lens art projecting caustics
- [Water Caustic — Godot Shaders](https://godotshaders.com/shader/water-caustic/) — code for a cheap animated caustic with speed and tint
- [High-contrast Computational Caustic Design (YouTube, GCM EPFL)](https://www.youtube.com/watch?v=R00IvqcI9jU) — SIGGRAPH 2014 video of surfaces engineered to cast images as caustics

## Code design

Two ways to make the net, one shared set of controls.

The cheap fake is an iterated sine warp: warp the coordinates in a loop with time-driven sin and cos, accumulate `1/length(...)`, then raise the result to a power to sharpen thin lines. The exponent is the sharpness control: 4 gives soft blobs, 8 the default net, 14 hairlines. It is the well-known Shadertoy effect ported on [Godot Shaders](https://godotshaders.com/shader/water-caustic/).

!!! warning "Licence"
    Shadertoy code is CC BY-NC-SA 3.0 unless the author says otherwise ([Godot Shaders](https://godotshaders.com/shader/water-caustic/)). The sine-warp method is non-commercial by default. For paid gigs, use the height-field method below, or get the author's permission.

The recommended method, and the one to use on commercial gigs, is the height-field refraction caustic. Sum about six sine waves into a height field `h`. Long waves move slower, as in deep water. A vertical ray hitting the surface at `p` lands on the floor at `q = p − D·∇h`, where `D` is the "focus". Light concentration is the inverse area change of that mapping, `E = 1 / max(|det J|, ε)` with `J = I − D·H`. Gradient and Hessian are computed analytically, so there are no texture lookups. Sweeping `D` from 0 up moves the floor through the caustic sheets: soft patches, then sharp lines, then crossed nets ([Atmospheric Optics](https://atoptics.co.uk/fz776.htm)). GPU Gems uses an index of refraction of 1.33 for water.

Around that core:

- **Beat ripples:** four ripple slots. Each kick overwrites the oldest slot with a new centre, start time and amplitude. The ring is added to the height field, so the net bends.
- **Two-scale parallax:** evaluate the caustic twice, the second time at 2.2× scale, rotated 30°, at 1.4× speed and 0.7× focus.
- **Prismatic split:** evaluate the near layer with `D` scaled per channel (R `D·(1−s)`, B `D·(1+s)`, `s` up to 0.08). Triple cost.
- **Hidden word reveal:** lower wave amplitude inside a blurred logo mask, so the letters become calm bright pools. This is a stage fake after *River is...*, not true inverse caustic design.
- **Time:** accumulate `t += speed · dt`. Never use `TIME · speed`, or the pattern jumps when the fader moves.

Layer stack, bottom to top: flat #02080C background, blurred far layer as pool tint (Add 40 %), far caustic (Add 45 %), near caustic (Add 100 %), optional reveal mask, then bloom, grain and vignette. A feedback mix of 0.25 smooths flicker on thin lines. Set it to 0 on drops.

=== "Resolume Wire"
    Use the gather version: a single-pass ISF shader that merges height field and caustic, with a mode switch between both methods, prism and four ripple slots. Wire has no vertex shaders, so the scatter version is not possible here. Resolume documents ISF inside Wire via the ISF node ([Resolume ISF support](https://resolume.com/support/en/isf)). Build it as a Wire patch and load the patch in Arena as a source.

    - Accumulate time in Wire (delta times speed, fed back into itself) and pass it as a uniform.
    - A beat trigger feeds a counter (wrap 4). It picks the ripple slot, and sample-and-hold values give its random centre and start time.
    - Audio analysis low band, then an envelope (attack 5 ms, release 200 ms), then a threshold gives the ripple trigger and the +25 % intensity pulse.
    - Do the temporal smoothing with Wire's own feedback at 0.25. Multipass ISF support depends on the version, so keep the shader single-pass.
    - Expose speed, focus, scale, ripple, prism, reveal and mode.

    In Arena, stack the source on several layers: a blurred, colourised copy (Add 40 %), a far copy at 220 % scale, 30° rotation and 1.4× speed (Add 45 %), and the near layer (Add 100 %). The logo clip goes in Multiply with opacity mapped to reveal. Add Bloom (threshold about 0.8, intensity about 1.3) and Vignette. No-shader fallback: two pre-rendered caustic loops at different speeds and scales. They never line up, which hides the loop point.

=== "Unreal Engine 5.7"
    Use a Post Process material (Unlit, Additive for surface use) with a Custom HLSL node that evaluates the height-field method. Read all parameters from a Material Parameter Collection `MPC_Caustic` (time, speed, focus, eps, scale, prism, reveal, ripple slots). Accumulate time in Blueprint Tick (`T += Speed · DeltaSeconds`).

    - **Mapped and 3D use:** a second material in the Light Function domain, monochrome, on a Spot or Rect Light. It projects animated caustics onto any geometry. Light functions are greyscale, so prism only works in the post or surface version.
    - **True scatter (sharpest):** a 512 × 512 subdivided plane. World Position Offset moves vertices by `−D·∇h`, a Vertex Interpolator carries the undisplaced position, and emissive is `|cross(DDX(p₀), DDY(p₀))| / pixelArea`, Additive. Capture with a top-down orthographic SceneCapture2D into an RGBA16F render target.
    - **Niagara glints:** a GPU sprite emitter samples the render target and spawns 0.2 s sprites where luminance is above 0.85, driven by hi-hats.
    - **Remote Control:** expose the collection scalars in a preset. Remote Control supports OSC, MIDI and DMX through protocol plugins ([Epic docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-protocols-in-unreal-engine)). Map BeatStep Pro knobs via MIDI, and kicks and envelopes via OSC from Resolume.
    - **Post:** Bloom 1.3 at threshold 0.8, manual exposure (eye adaptation pumps on kicks), Film Grain 0.02, Vignette 0.2.
    - **Output:** Spout via a third-party plugin, or NDI, at the LED processor's native resolution.

=== "GODOT"
    - **2D output:** a `ColorRect` filling the viewport with a `canvas_item` shader that holds both methods, parallax, prism and reveal. Accumulate `t` in GDScript `_process(delta)`.
    - **Temporal smooth:** render inside a `SubViewport` (HDR 2D on, clear mode Never) and mix the previous frame at 0.25 in a second `ColorRect`. Or skip it and lower `sharp` on LED.
    - **Scatter version (3D):** a `SubViewport` with an orthographic `Camera3D` over a PlaneMesh subdivided 511 × 511. A spatial shader with `render_mode unshaded, blend_add, cull_disabled` offsets `VERTEX.xz` by `−D·∇h` and stores the original position in a varying. `fragment()` sets `ALBEDO` from `length(cross(dFdx(p₀), dFdy(p₀)))`. Evan Wallace's [WebGL Water](https://madebyevan.com/webgl-water/) renders caustics with screen-space derivatives in a similar way.
    - **Mapped and 3D use:** use the ViewportTexture as the `light_projector` of a `SpotLight3D`, or as the emission texture of a `Decal`.
    - **Post:** `WorldEnvironment` glow, intensity 1.3, HDR threshold 0.8. For 2D, set the background mode to Canvas.
    - **Control:** `InputEventMIDI` drives the uniforms. For Spout, use a community addon if one exists for your Godot version, otherwise NDI via screen capture.

## On stage

**Audio reactivity**

| band | drives | feel |
|------|--------|------|
| kick, 40–100 Hz onset | spawns a ripple, intensity +25 % | attack 2 ms, release 200 ms, at most one ripple per 1/8 note |
| bass, 60–250 Hz | wave amplitude 0.8–1.3× | slow: attack 60 ms, release 500 ms |
| mids, 300 Hz–2 kHz | focus ±10 % | attack 150 ms, release 1 s |
| hi-hats, 6–12 kHz | small extra wave for shimmer, optional Niagara glints | attack 1 ms, release 80 ms, gate at −30 dBFS |

Keep only the kick fast. The water should breathe with the bass, not twitch. If more than two parameters move per beat, the calm is gone.

**MIDI (KeyStep 37 / BeatStep Pro)**

| parameter | range | default | suggested control |
|-----------|-------|---------|-------------------|
| speed | −1 to 1 units/beat | 0.25 | knob, BPM-locked, a key or pad for freeze |
| focus | 0–2 | 1.0 | fader, automated in builds |
| scale | 0.5–4 | 1.0 | knob |
| ripple | 0–2 | 1.0 | BeatStep Pro pad as kick trigger, knob for amount |
| prism | 0–0.08 | 0 | knob, jump to 0.05 on drops |
| reveal | 0–1 | 0 | fader, ramp 1–2 bars before a drop |

**Set shape**

- **Speed:** lock to tempo, default one time unit per bar at 4/4. Glide speed and focus changes over half a beat.
- **Breathing:** drift focus ±5 % on a 4-bar sine, so the net moves even without audio.
- **Build (last 8 bars):** focus from 0.5 to 1.2× default, ripples every 8th note, reveal 0 to 1 if you use a logo.
- **Drop:** hard cut. Smoothing 0, a big ripple from screen centre (2× amplitude), prism 0.05, crossfade to the sun palette over 1 beat, speed doubled for 2 bars.
- **Break:** speed 0.3×, focus 0.5×, far layer off, prism 0, reveal 0. Freeze over 1 beat, or reverse at −0.25 for a 2-bar breakdown. It looks like water flowing back.
- **Palette:** rotate hue ±10° per 16-bar phrase and stay in the water family until the drop.

**Layering and tips**

- Everything above the background is Add. Crossings get brighter, as in reality.
- As a lighting pass, output the near layer alone in white as a luma mask over mapped architecture. Or use the gradient of the height field as a displacement map on video, which makes it look underwater.
- LED walls: hold the black at #02080C and cap the near layer at about 90 % before bloom. On coarse pitches (4 mm and up) lower `sharp` to 5–6 or raise eps to 0.1, so lines are at least 2 px wide.
- Always aspect-correct coordinates and ripple distances, or cells stretch on 32:9 strips.
- Do not use auto-exposure in UE. Do not set the bloom threshold below 0.7, or the frame floods cyan. Too small an eps gives single-pixel fireflies.
- Thin lines need full resolution. Half resolution turns them to mush.
- The sine-warp fake repeats every TAU; the rotated far layer hides the grid. The height-field method does not tile if the wave directions are irrational multiples of each other.

## Variations

1. Two caustic layers at different scales and speeds with parallax, for depth.
2. Beat-driven impulse ripples feeding the height field, so every hit is visible as a ring.
3. Chrome-sculpture style: the caustic resolves into a hidden word or logo on a bar drop, after *River is...*.
4. Prismatic split: offset R, G, B caustics for coloured fringes.
5. Mapped use: caustic as light on projection-mapped architecture, sent via Spout from Resolume or UE.
6. Caustic as a texture source: use it as a luma mask or displacement map for other clips.

## Did you know?

- A rainbow is a caustic ([Wikipedia](https://en.wikipedia.org/wiki/Caustic_(optics))).
- Catastrophe theory limits stable caustics to seven elementary forms ([Atmospheric Optics](https://atoptics.co.uk/fz776.htm)).
- The Atmospheric Optics example was photographed in only 30–50 cm of sea water ([Atmospheric Optics](https://atoptics.co.uk/fz776.htm)).
- A 1991 caustics animation ran just 30 seconds ([SIGGRAPH History Archive](https://history.siggraph.org/?p=65616)).

## Sources

- <https://en.wikipedia.org/wiki/Caustic_(optics)>
- <https://atoptics.co.uk/fz776.htm>
- <https://angesleva.iki.fi/art/river-is/>
- <https://www.kristallwelten.swarovski.com/en/artworks/caustic-rays>
- <https://physicsworld.com/a/caustic-light-patterns-inspire-new-glass-artwork/>
- <https://history.siggraph.org/?p=65616>
- <https://godotshaders.com/shader/water-caustic/>
- <https://wallisc.github.io/rendering/2020/12/08/Making-Of-Moana-the-shadertoy.html>
- <https://www.youtube.com/watch?v=R00IvqcI9jU>
- <https://developer.nvidia.com/gpugems/gpugems/part-i-natural-effects/chapter-2-rendering-water-caustics>
- <https://madebyevan.com/webgl-water/>
- <https://www.cyanilux.com/tutorials/water-shader-breakdown/>
- <https://resolume.com/support/en/isf>
- <https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-protocols-in-unreal-engine>
