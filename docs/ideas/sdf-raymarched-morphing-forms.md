---
tags:
  - 3d
  - raymarching
  - shader
  - generative
---

# SDF Raymarched Morphing Forms

> Soft forms of liquid graphite swell, merge and burst apart on the kick, all drawn by one shader and no mesh.

## The idea

A signed distance function (SDF) answers one question: how far is this point from the nearest surface? The sign says whether the point is inside or outside ([Wikipedia: Signed distance function](https://en.wikipedia.org/wiki/Signed_distance_function)). Describe a scene that way and you can render it by ray marching. Each pixel casts a ray. At every step the scene tells the ray how far it may safely jump. The ray stops when the distance falls below a small threshold ([Wikipedia: Ray marching](https://en.wikipedia.org/wiki/Ray_marching)).

The payoff is what meshes find hard. Shapes combine with boolean operators. A modulus repeats one cell through endless space. Morphing between two shapes is a blend of their distance values, and a smooth union rounds the joint instead of leaving a crease. Ray marching suits morphing, soft shadows, repetition and algorithmic scenes ([Wikipedia: Ray marching](https://en.wikipedia.org/wiki/Ray_marching)).

On a club screen this reads as matter with a pulse. Spheres, boxes and tori melt into one another like clay or metal. A camera flies down a corridor of cells that breathe in a wave. The kick makes the necks between forms fatter. The whole world is a few dozen lines of shader, and every number in it can sit on a knob.

## Origins and references

- **Sphere tracing** is credited to John C. Hart, in a June 1995 paper on antialiased ray tracing of implicit surfaces. The demoscene and Inigo Quilez, co-creator of Shadertoy, later made it popular ([Wikipedia: Ray marching](https://en.wikipedia.org/wiki/Ray_marching)).
- **Inigo Quilez** builds SDF scenes from scratch in live-coding sessions such as "Sphere Gears" ([Part 1](https://www.youtube.com/watch?v=sl9x19EnKng), [Part 2](https://www.youtube.com/watch?v=bdICU2uvOdU)).
- **evvvvil + OddJohn, Generative Show (2017)**: a live AV set built on raymarched distance fields, crediting Quilez's distance-field work. Resolume is among the listed tools ([vvvv blog](https://legacy.vvvv.org/blog/evvvvil-oddjohn-generative-show)).
- **Marching.js** by Charlie Roberts: a JavaScript library that compiles GLSL raymarchers, with an API terse enough for live coding ([GitHub](https://github.com/charlieroberts/marching)).
- **Sebastian Lague**, "Coding Adventure: Ray Marching": a tutorial for building the technique ([YouTube](https://www.youtube.com/watch?v=Cp5WWtMoeKg)).
- **Trippasch**, MandelBox Audio Visualization: a Unity raymarcher driven by audio ([YouTube](https://www.youtube.com/watch?v=A9TlDTLxBmk)).

## Visual language

Smooth, liquid joins between spheres, boxes and tori. Depending on the shading, the surfaces look like clay, metal or glass. The design here goes for satin graphite: grey-blue bodies, crisp white speculars, deep ambient occlusion in the creases, and one hot orange rim light on the edges facing away from the camera. Far parts fade into blue-black fog.

Three scenes share one renderer:

- **Blob chain.** Three to six forms orbit each other and melt together. On every kick the necks get fatter and the cluster gulps together, then pulls apart.
- **Corridor.** An endless run of repeated cells, with the camera on a tempo-locked rail. Each cell morphs between box and sphere on its own phase, so a wave travels through the tunnel.
- **Twist.** A stack of primitives wrung around the vertical axis.

The mood is cool and sculptural. Shading is mostly value, not hue: near-monochrome with a single accent. Rhythm comes from the blend radius pumping, forms swelling and splitting, and camera cuts. Fractal forms in the Mandelbox family fit the same renderer.

## See it

- [Quilez, Live Coding "Sphere Gears" Part 1](https://www.youtube.com/watch?v=sl9x19EnKng) — an SDF scene built live from scratch (YouTube, Inigo Quilez)
- [Quilez, Live Coding "Sphere Gears" Part 2](https://www.youtube.com/watch?v=bdICU2uvOdU) — continuation of the same session (YouTube, Inigo Quilez)
- [Sebastian Lague, Coding Adventure: Ray Marching](https://www.youtube.com/watch?v=Cp5WWtMoeKg) — tutorial with morphing and repeated SDF forms (YouTube, Sebastian Lague)
- [MandelBox Audio Visualization, Unity Demo](https://www.youtube.com/watch?v=A9TlDTLxBmk) — audio-driven raymarched fractal (YouTube, Trippasch)
- [evvvvil + OddJohn Generative Show](https://legacy.vvvv.org/blog/evvvvil-oddjohn-generative-show) — project page for a live raymarching AV set (no video on the page)

## Code design

Everything runs from the host's beat clock (bar phase and beats elapsed), never from wall-clock time, so the orbit stays locked after tempo changes. The scene function `map(p)` holds the primitives. A smooth union `h = clamp(0.5 + 0.5 * (b - a) / k, 0, 1)`, `d = mix(b, a, h) - k * h * (1 - h)` makes the liquid joins. Here `k` is the blend radius: 0 is a hard union, 0.3 a soft neck, 1.0 one blob. A morph is `mix(dA, dB, morph)` between two primitives. Normals, a five-sample ambient occlusion, a rim term and fog finish the shading.

Smooth union, twist, bend and ripples all make `map` return a distance that is no longer exact. A true SDF has gradient length 1 almost everywhere ([Wikipedia: Signed distance function](https://en.wikipedia.org/wiki/Signed_distance_function)). The marcher then overshoots and shows holes at silhouettes. The fix is a lower step scale, `0.9 / (1 + 0.35 * twist + 0.5 * bend + 6 * amp)` clamped to 0.45–0.9. That makes the twist fader a cost fader too.

Render order per frame: raymarch, upscale and sharpen, optional feedback, glow, composite. The raymarch pass runs at half the output resolution by default. Step-by-step background: [Jamie Wong's tutorial](https://jamie-wong.com/2016/07/15/ray-marching-signed-distance-functions/).

=== "Resolume Wire"
    Arena runs the raymarcher as a Wire patch loaded as a source. Wire has its own ISF node ([Resolume ISF support](https://resolume.com/support/en/isf)).

    - The `BPM` transport gives bar phase and a beat counter. Wrap the counter to protect float precision.
    - `Audio Analysis` low band goes through an envelope (attack 5 ms, release 180 ms), times `pulse`, into the ISF `pulse` input. The bass band gets a slower envelope (60 ms / 500 ms) and offsets `twist`.
    - One `ISF` node holds the whole raymarch (march, shade, fog, alpha) as a single pass at 1920 x 1080. Let Arena scale the clip up to a 4K composition. This is the simplest way to get half resolution.
    - Build feedback and glow from Wire nodes: previous frame times decay (0.75) plus the current frame, then bright-pass, downscale, blur, Add. Wire's multipass ISF support has varied by version, so keep the raymarch single-pass. The [ISF JSON reference](https://docs.isf.video/ref_json.html) describes passes and persistent buffers for hosts that support them.
    - Expose blend, morph, twist, density, speed, pulse, scene, quality, trail amount and accent colour. In Arena, layer 1 is a #07080B to #0B0E14 gradient, layer 2 is the patch with Normal blend, then Bloom (threshold about 0.7) and a light Sharpen.
    - Mask mode: a second instance outputting alpha only (white forms on black), placed above a video layer as its luma mask.

=== "Unreal Engine 5.7"
    - One material with a **Custom** node holds the march loop, `map`, normal, AO and shading in HLSL. Epic notes that a Custom node blocks constant folding and can cost more instructions than built-in nodes ([UE docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/custom-material-expressions-in-unreal-engine)). For a loop it is still the practical route.
    - For resolution control, use *Draw Material to Render Target* each tick (RGBA16F, 1920 x 1080 for a 4K wall) from a Blueprint. Show the target full screen with bilinear sampling. The expensive pass then stays at a fixed size.
    - A Material Parameter Collection `MPC_SDF` carries Blend, Morph, Twist, Density, Speed, Pulse, Scene, Quality, BarPhase, BeatsElapsed and the colours. Expose it in a Remote Control Preset bound to OSC (BarPhase and Pulse from Resolume or an analyser) and MIDI.
    - Optional: a GPU Niagara emitter with 20–50k dim additive sprites on the same Lissajous path as glowing dust. They are not occluded by the forms unless you compare against a depth target, so keep them sparse.
    - Post-process volume: Bloom intensity about 1 with threshold 0.7, manual exposure (eye adaptation would pump on every kick), low Film Grain, Vignette 0.2.
    - Send the final frame to Resolume via Spout or NDI at the LED processor's native resolution.

=== "GODOT"
    - A `SubViewport` at render scale (for example 1920 x 1080, update mode Always, HDR 2D on) holds a full-rect `ColorRect` with a `canvas_item` shader. Build uv from `FRAGCOORD.xy` and `1.0 / SCREEN_PIXEL_SIZE` ([canvas_item reference](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/canvas_item_shader.html)) and march in `fragment()`.
    - Show it through a `TextureRect` with a `ViewportTexture`, linear filter. A second shader on the TextureRect does the sharpen.
    - Feedback: a second `SubViewport` reads its own previous frame through a `ViewportTexture` (one frame of delay), times decay, plus the raymarch texture.
    - Uniforms with `hint_range`: blend_k, morph, twist, bend, ripple, density, speed, pulse, scene, quality, bar_phase, beats_elapsed, accent_color, body_color. Drive `bar_phase` from a script that integrates BPM, not from `TIME`.
    - Glow through `WorldEnvironment` (threshold 0.7). Control by `InputEventMIDI` in `_input()`, mapping CC numbers to uniforms. OSC needs a community addon.

## On stage

**Audio reactivity.** Use envelopes, not raw levels. Only the kick moves geometry fast. Everything else breathes.

| band | drives | smoothing | threshold |
|------|--------|-----------|-----------|
| kick, 40–100 Hz onset | pulse envelope: blend k (+0.35), radii (+0.08), rim x 2 | attack 5 ms, release 180 ms | onset at +6 dB over the 1 s average |
| bass, 60–250 Hz level | twist offset (0–0.4), camera bob | attack 60 ms, release 500 ms | none, continuous |
| mids, 300 Hz–2 kHz | ripple amp 0–0.03 (surface shimmer) | attack 80 ms, release 600 ms | gate below -36 dBFS |
| hi-hats, 6–12 kHz | specular flash (x 1.5, 40 ms), corridor cell-offset step | attack 1 ms, release 60 ms | gate at -30 dBFS |
| full-band loudness, 4 s average | fog density: louder is clearer, quieter is foggier | attack 1 s, release 3 s | none |

**MIDI mapping.**

| parameter | range | default | KeyStep 37 / BeatStep Pro |
|-----------|-------|---------|---------------------------|
| blend | 0–1.2 | 0.25 | KeyStep knob; the kick envelope adds +0.35 |
| morph | 0–2 | 0 | BeatStep Pro pads 1–8 pick target pairs, eased over 1 bar |
| twist | 0–3 rad/unit | 0 | fader; bass envelope adds up to +0.4 |
| density | 1.5–4 cell size, or 3–6 blobs | 2.5 / 5 | stepped knob; change on bar lines only |
| speed | 0–4 | 1 | knob quantised to 0.25, 0.5, 1, 2, 4 |
| pulse | 0–1 | 0.6 | kick envelope x this knob |

Pads also select the scene (0 blob, 1 corridor, 2 twist). A KeyStep note can trigger the next morph pair. A quality toggle (low/high) switches steps 56/96, AO 3/5 samples and the soft shadow.

**Song structure.**

- **Build** (last 8 bars): k ramps from 0.25 to 0.9 so everything merges into one blob. The camera closes from 5 to 3.2 units, ripple climbs to 0.05, fog thickens.
- **Drop**: hard cut on the downbeat. Switch scene, snap k to 0.15 so the forms burst apart, swap the accent, allow a chromatic offset for one beat, trails off.
- **Break**: camera speed halves, ghost trails go to about 30 %, rim drops to 0.5, the forms drift with k near 0.6.

**Layering.** Output with alpha or depth for Resolume, or share via Spout/NDI. A dark background stacks well with other layers. In mask mode the forms cut other video. Depth as a matte gives a soft falloff for fog-like blends.

**Colour and blending.** Body #5A6170, shadow #1C1F26, highlight #F2F4F7, rim #FF5A1F. Swap the rim to ice cyan #3DDCFF per 16-bar phrase, and use acid green #C6FF3D only on the peak drop. Cut accent swaps on the downbeat. A fade passes through muddy mixed tones. Draw the forms with premultiplied Normal blend, since they are solid, and add glow and trails with Add. Keep the trail decay at or below 0.75: above 0.9 the dark bodies smear into grey soup. On LED walls keep black at #07080B rather than 0, and keep body albedo under 40 % brightness so speculars and rim have headroom.

**Cost.** Raymarching cost scales with pixels x steps x the cost of `map`. Estimated budget, assuming one `map` for six smooth-unioned spheres is roughly 60–80 GPU instructions. These ms figures are order-of-magnitude estimates from that count, not benchmarks. Measure on your machine.

| output | render scale | pixels marched | scene A (blobs, ~35 % of pixels hit) | scene B (corridor, ~100 % hit) |
|--------|-------------|----------------|-------------------------------------|-------------------------------|
| 1920 x 1080 | 1.0 | 2.07 M | ~2–4 ms mid-range GPU | ~6–10 ms |
| 3840 x 2160 | 0.5 | 2.07 M | ~2–4 ms | ~6–10 ms |
| 3840 x 2160 | 1.0 | 8.3 M | ~8–15 ms | ~25–40 ms (misses 60 fps) |
| 7680 x 1080 (wide strip) | 0.5 | 2.07 M | ~2–4 ms | ~6–10 ms |
| 7680 x 2160 | 0.5 | 4.15 M | ~4–8 ms | ~12–20 ms |

**Performance tips.**

- Render scale is the main lever: 0.5 on any output above 2.5 M pixels, 0.67 for fine-pitch 1080p walls. LED walls with pitch 3.9 mm and up hide 0.5 scale completely.
- Cap steps at about 80. Below 48 the twist scene shows holes. Above 128 the extra steps go into grazing rays that hit nothing.
- Use a relative epsilon (`eps * t`). It saves steps at distance and costs nothing visible.
- A bounding sphere skips 50–70 % of pixels in wide shots of the blob and twist scenes. The corridor cannot use it, so it is the most expensive scene. Give it fog 0.035+ and MAX_DIST 40.
- Under load, drop the soft shadow first (about 15 %), then AO to 3 samples, then steps to 56.
- Unroll fixed blob counts instead of looping over a uniform. Dynamic loops inside `map` multiply the cost of every step.
- Wrap the camera position and beat counter. After an hour in the corridor, 32-bit floats start to shimmer.
- Holes at silhouettes during twist mean overstepping, not too few steps. Lower the step scale.
- On 32:9 strips, raise the focal length to 2.2–2.5 or run two cameras. Never stretch the render, or the spheres turn into ovals.
- Mask mode needs only alpha: skip AO, lighting and shadows and stop at the first hit, about 3x cheaper.
- Boring risks: a single blob that only grows and shrinks, a static camera, uniform grey shading, no contrast change over minutes.

## Variations

1. **Kick-blob chain**: three to six spheres on a BPM-locked orbit, smooth-merged, k driven by the kick.
2. **Repeated corridor**: domain repetition with a per-cell shape morph, camera on a tempo-synced rail.
3. **Twist and melt**: a stack of primitives with bend and twist on faders, hi-hats adding noise displacement.
4. **Mask mode**: render the SDF as a silhouette or depth mask to cut other video layers in Resolume.
5. **Sequenced shape bank**: MIDI notes select the primitive pairs, so a keyboard plays the morph targets.
6. **Live-coded forms**: write the scene in the browser with [Marching.js](https://github.com/charlieroberts/marching) and feed it in by capture.

## Did you know?

- Sphere tracing dates from 1995 ([Wikipedia: Ray marching](https://en.wikipedia.org/wiki/Ray_marching)).
- A true SDF has gradient length 1 almost everywhere. Smooth operations break that property, so the marcher must take more cautious steps ([Wikipedia: Signed distance function](https://en.wikipedia.org/wiki/Signed_distance_function)).
- Raymarched live sets are not new: evvvvil and OddJohn played one in 2017 ([vvvv blog](https://legacy.vvvv.org/blog/evvvvil-oddjohn-generative-show)).
- Raymarchers can be live-coded in the browser through JavaScript that compiles to GLSL ([Marching.js](https://github.com/charlieroberts/marching)).

## Sources

- [Wikipedia: Ray marching](https://en.wikipedia.org/wiki/Ray_marching)
- [Wikipedia: Signed distance function](https://en.wikipedia.org/wiki/Signed_distance_function)
- [vvvv blog: evvvvil + OddJohn Generative Show](https://legacy.vvvv.org/blog/evvvvil-oddjohn-generative-show)
- [Marching.js on GitHub](https://github.com/charlieroberts/marching)
- [Quilez, "Sphere Gears" Part 1](https://www.youtube.com/watch?v=sl9x19EnKng)
- [Quilez, "Sphere Gears" Part 2](https://www.youtube.com/watch?v=bdICU2uvOdU)
- [Sebastian Lague, Coding Adventure: Ray Marching](https://www.youtube.com/watch?v=Cp5WWtMoeKg)
- [Trippasch, MandelBox Audio Visualization](https://www.youtube.com/watch?v=A9TlDTLxBmk)
- [Jamie Wong: Ray Marching and Signed Distance Functions](https://jamie-wong.com/2016/07/15/ray-marching-signed-distance-functions/)
- [Resolume ISF support](https://resolume.com/support/en/isf)
- [ISF JSON reference](https://docs.isf.video/ref_json.html)
- [UE docs: Custom material expressions](https://dev.epicgames.com/documentation/en-us/unreal-engine/custom-material-expressions-in-unreal-engine)
- [Godot canvas_item shader reference](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/canvas_item_shader.html)
