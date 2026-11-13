---
tags:
  - video
  - time
  - analog-video
  - feedback
  - camera
---

# Time-Displacement Slit-Scan

> Time stops running evenly across the frame, so everything that moves turns into a ribbon and everything that stands still stays sharp.

![Digital slit-scan of passers-by in a spiral projection; people become stretched time-streaks around a tunnel](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ef/Spiral_Slitscan%3B_March_2010.jpg/1280px-Spiral_Slitscan%3B_March_2010.jpg)
*Digital slit-scan of passers-by in a spiral projection; people become stretched time-streaks around a tunnel. Image: [Masakazu Matsumoto (vitroid)](https://commons.wikimedia.org/wiki/File:Spiral_Slitscan;_March_2010.jpg), [CC BY 2.0](https://creativecommons.org/licenses/by/2.0).*

## The idea

Think of video as a stack of frames: x, y and time. A normal player shows one slice at constant time. Slit-scan shows a different slice through the stack, so each part of the image comes from a different moment. Time becomes a spatial axis ([Rozin](http://www.smoothware.com/danny/timescan.html)).

Time displacement is the general form. A grayscale offset map says, for each pixel, how many frames back to read. A horizontal ramp gives a classic smear. A radial ramp gives a tunnel. A moving or noisy map makes the picture ripple. This is the standard technique, and Glen Murphy's *Robot Mirror* (2005) is listed as a per-pixel generalization in [Levin's catalogue](https://flong.com/archive/texts/lists/slit_scan/index.html).

On a club screen it works because it needs no content of its own. Put it over a camera, a clip or a feedback source. A hand becomes a fan of fingers. A dancer stretches and folds. The kick can push the delay depth up and let it relax, so the image smears on the hit and snaps back between hits.

## Origins and references

- **Lartigue, *Car Trip, Papa at 80 Km/h*, 1913**: an accidental slit-scan effect caused by a slow vertical shutter ([Levin's catalogue](https://flong.com/archive/texts/lists/slit_scan/index.html)).
- **John Whitney, *Vertigo* titles**: he sent test footage to Kubrick, who passed the technique to Douglas Trumbull ([Wikipedia](https://en.wikipedia.org/wiki/Slit-scan_photography)).
- **Douglas Trumbull, *2001: A Space Odyssey*, Stargate, 1968**: a custom machine and long streaks of light rushing toward the viewer ([Wikipedia](https://en.wikipedia.org/wiki/Slit-scan_photography), [catalogue](https://flong.com/archive/texts/lists/slit_scan/index.html)).
- **Bernard Lodge, Doctor Who titles, Dec 1973 to Jan 1980**. The technique also appears in *Superman* (1978) and in the Enterprise-D warp stretch in *Star Trek: The Next Generation* ([Wikipedia](https://en.wikipedia.org/wiki/Slit-scan_photography)).
- **Eddie Elliott, *Video Streamer*, 1992**: an early digital example that cuts thin slices from successive frames ([Levin's catalogue](https://flong.com/archive/texts/lists/slit_scan/index.html)).
- **Daniel Rozin, *Time Scan Mirror*, 2004**: scans one vertical line at the centre of the camera frame and copies it sideways, giving a roughly 30-second log of what passed by ([Rozin](http://www.smoothware.com/danny/timescan.html)).
- **Alvaro Cassinelli and Masatoshi Ishikawa, *Khronos Projector*, 2005**: the viewer pushes a deformable screen, and the pushed parts of pre-recorded video move forward or backward in time ([Make](https://makezine.com/article/craft/the-khronos-projector), [SIGGRAPH archive](https://history.siggraph.org/experience/khronos-projector-by-cassinelli-and-ishikawa)).
- **Forrest Oliphant, *Slithering*, 2011**: dancers stretch, merge and fold into each other in slow motion, made with a custom tool that takes one pixel column from each frame ([VICE](https://www.vice.com/en/article/post-processing-technique-really-makes-these-dancers-slither-video/)).
- **Camille Utterback, *Liquid Time*, 2000** and **Brendan Dawes, *Don't Look Now*, 2004**: further entries in [Levin's catalogue](https://flong.com/archive/texts/lists/slit_scan/index.html), which lists more than 90 projects.

## Visual language

Smooth flowing streaks, bent lines, and subjects that slide along the scan direction. Static background stays sharp while moving things become ribbons.

The look ranges from psychedelic colour rivers (Stargate, Doctor Who titles) to uncanny melted portraits (Rozin, Oliphant). Rhythm comes from the map. Gradient direction, rotation and softness decide whether it reads as a tunnel, a wave or a wipe.

In the design, the edge where the scan starts shows "now". Every step across the image reaches further into the past, up to about two seconds. On the drop the map switches to a radial tunnel: the centre is now, the edges are the past, and moving light pours outward.

Colour comes from the source. An optional age tint multiplies a three-stop ramp into the image: cyan (#2DE2E6) for now, magenta (#FF3D7F) for the past, violet (#6A2C91) for the oldest part, on a #000000 base. With tint at 0 the effect is natural and uncanny. At 0.4 to 0.6 it becomes the colour river.

The effect needs motion in the source. A still frame gives nothing. A hand, a dancer or a light moving across the frame gives the most.

## See it

- [Levin's slit-scan catalogue](https://flong.com/archive/texts/lists/slit_scan/index.html) — over 90 projects, with links to videos of Trumbull, Rist, Jarre and others
- [Time Scan Mirror, Daniel Rozin](http://www.smoothware.com/danny/timescan.html) — the artist's page for the 2004 mirror, with a screenshot and a link to a video
- [Easy Slitscan in TouchDesigner - TouchDesigner Tutorial 067](https://www.youtube.com/watch?v=1ACBmGYB8uw) — a short tutorial building a slit-scan from a few operators (YouTube, The Interactive & Immersive HQ)
- [Slithering, Forrest Oliphant (VICE)](https://www.vice.com/en/article/post-processing-technique-really-makes-these-dancers-slither-video/) — article on the dancer slit-scan piece with embedded video
- [Khronos Projector (Make)](https://makezine.com/article/craft/the-khronos-projector) — photos of the installation
- [Slit Scan using the Time Machine TOP (Derivative)](https://learn.derivative.ca/courses/200-intermediate/lessons/202-tops-intermediate/topic/slit-scan-using-the-time-machine-top/) — official TouchDesigner lesson with example file

## Code design

The core is a ring buffer of the last N frames. A write pointer `head` advances by one each output frame, and nothing is copied. The default is N = 128 frames at 960×540, which is 2.13 s at 60 fps and covers one bar at 128 BPM. Each output pixel reads frame `head - d`, where `d = m · depth · (N - 1)` and `m` is the offset map (0 = now, 1 = oldest). A second read from the neighbouring frame, mixed by the fraction, removes visible steps in smooth ramps.

Not every tool can write into a slice of a texture array, but all can draw into a region of one large 2D render target. So the default storage is an atlas: a 16×8 grid of 960×540 tiles, 15360×4320 in total, which fits the usual 16384 px texture limit. Clamp the read uv inside each tile and turn mipmaps off, or thin lines of a neighbouring frame show at band edges.

Map modes are linear (angle), radial, luma, noise and spectrum. Wobble and hard bands shape any of them. A beat-stepped mode rounds the delay to whole beats, so strips show now, one, two and three beats ago. An optional self-feed mixes the previous output into what is written to the history. Keep it at 0.5 or below, or the image turns to mush. Render order: history write, read with map and tint, dry/wet composite, post (black crush, bloom, grain).

| Parameter | Range | Default | Use |
|-----------|-------|---------|-----|
| depth | 0–1 | 0.35 | how far back the far edge reaches |
| angle | 0–360° | 90° | direction of the sweep (linear mode) |
| shape | 0–4 | 0 | linear, radial, luma, noise, spectrum |
| bands | 0–32 | 0 | hard stepped slits; with beat-step, one strip per beat |
| wobble | 0–0.3 | 0.05 | ripples the time offsets |
| feed | 0–0.5 | 0 | streaks that persist and drift |

Dry/wet is the layer opacity. Use Normal blend, not Add, because both layers are full images. For an echo look, put the wet layer in Lighten over the dry.

=== "Resolume Wire"
    Arena has no built-in effect that stores a multi-frame history and reads it per pixel. Its Feedback and Trails effects blend into one buffer, so they cannot do real slit-scan. Build the patch in Wire and load it in Arena as an effect on the source layer.

    - **Option A, Ring Buffer node.** It keeps the last X frames as a collection, and the Split node turns the collection into separate outputs. Check the maximum buffer size in your Wire version first. A later changelog limits Ring Buffer sizes to prevent freezing, so do not plan for 128 full-HD frames. Banded build: Ring Buffer size 16 at 960×540, Split into 16 outputs, mask each output to one band of the offset map, composite the 16 layers. This gives the stepped "bands = 16" look. Smooth ramps are not possible this way.
    - **Option B, ISF atlas.** Use a two-pass ISF in Wire's ISF shader node. Pass 1 is a persistent buffer: if the pixel lies in tile `FRAMEINDEX mod N`, write the input, else copy the previous value. Pass 2 is the read shader. The whole atlas is read and written every frame, so stay at tile 480×270 and a 8×8 grid (64 frames, atlas 3840×2160, 33 MB). Multipass and persistent-buffer support depends on the version. Test it before the gig, and fall back to option A if the buffer does not keep its content.
    - **Outline for B.** Texture input, Resize to 960×540, ISF slit-scan, Mix with the original by dry/wet, Output.
    - **Control.** A beat node feeds an envelope (attack 10 ms, release 300 ms) that is added to depth. An LFO at 1/4 bar rate drives angle. The low audio band drives wobble. Exact node names differ between Wire versions.
    - Third-party FFGL plugins such as [BeatScanner](https://tombass.gumroad.com/l/beatscanner) and [Slice Displace](https://rexaffects.gumroad.com/l/gmtca) exist but were not tested for this book.

=== "Unreal Engine 5.7"
    - **Source.** A Media Player and Media Texture (camera or file), or an NDI receiver texture.
    - **History.** `RT_History`, a Texture Render Target 2D, 15360×4320, RGBA8, mips off, clear colour black, bilinear. 128 tiles of 960×540, about 265 MB.
    - **Write.** In a Blueprint on Tick, compute `head = frameCount mod 128` and the tile origin `(head mod 16 · 960, floor(head / 16) · 540)`. Call Begin Draw Canvas to Render Target on `RT_History`, Draw Texture at the tile origin with size 960×540 and Opaque blend, then End Draw Canvas ([UE docs](https://dev.epicgames.com/documentation/unreal-engine/BlueprintAPI/Rendering/BeginDrawCanvastoRenderTarget)). Never call Clear Render Target 2D, or the other tiles are lost. Only one tile is touched per frame, so the write is cheap. For feed above 0, draw a material that mixes the Media Texture and the previous output.
    - **Read.** Material `M_SlitScan` (User Interface or Post Process domain). The map logic uses standard nodes (Dot, Length, Noise, Floor), no Custom node. Compute the delay, build two atlas UVs with Fmod, Floor and Frac plus a clamp inset, take two Texture Samples on `RT_History`, Lerp by the fraction, then apply the age tint through a three-colour Lerp.
    - **Output.** Draw the material into `RT_Out` (1920×1080) and send it with Spout or NDI, or use it as a Post Process Material on a fullscreen camera view.
    - **Remote Control.** Put the scalars `Depth`, `Angle`, `Shape`, `Bands`, `Wobble`, `Feed`, `DryWet`, `Tint` in a Material Parameter Collection `MPC_SlitScan`. Expose them in a Remote Control Preset and bind to OSC and MIDI. `Head` and `Filled` are set by the Blueprint.
    - Niagara is not needed.

=== "GODOT"
    - **Source.** Godot 4. A VideoStreamPlayer (Theora .ogv) or a CameraTexture, rendered at 960×540.
    - **History.** A SubViewport `History` at 15360×4320 with `render_target_clear_mode = CLEAR_MODE_NEVER` and `render_target_update_mode = UPDATE_ALWAYS` ([Godot docs](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)). Transparent background off, 2D only. Inside it, one TextureRect shows the source at 960×540, and a script moves it to the tile origin of `head` each frame. Because the viewport never clears, older tiles stay.
    - If a 15360 px wide viewport fails on the target GPU, use 64 tiles (8×8, atlas 7680×4320).
    - **Read.** A full-screen ColorRect with a `canvas_item` shader samples the History ViewportTexture (linear filter, no mipmaps) and does the read, tint and dry/wet.
    - **Feed.** A second SubViewport at 1920×1080 holds the previous output. The TextureRect in `History` uses a small shader that mixes source and that texture by `feed`.
    - **Uniforms.** `depth`, `angle`, `shape`, `bands`, `wobble`, `feed`, `tint`, `dry_wet`. The script sets `head`, `filled`, `cols`, `rows`. Drive them from an AudioEffectSpectrumAnalyzer on the music bus: low band for the kick, 60–250 Hz for wobble.

## On stage

**VRAM is the cost, not shader math.** One RGBA8 frame is 8.3 MB at 1920×1080, 2.1 MB at 960×540 and 0.5 MB at 480×270. Do not use float for the history, since RGBA16F doubles every figure.

| History | 64 frames | 128 frames | 256 frames |
|---------|-----------|------------|------------|
| 1920×1080 RGBA8 | 531 MB | 1.06 GB | 2.12 GB |
| 960×540 RGBA8 | 133 MB | 265 MB | 531 MB |
| 480×270 RGBA8 | 33 MB | 66 MB | 133 MB |

The recommended default is 960×540 with 128 frames (265 MB). The smear hides the lower resolution, and the dry layer keeps the sharp full-res image where nothing moves.

**Audio reactivity**

| Band | Drives | Smoothing | Threshold |
|------|--------|-----------|-----------|
| kick, 40–120 Hz | depth punch +0.35 | attack 10 ms, release 300 ms | 0.6 of the running peak (2 s window), retrigger blocked for 120 ms |
| bass, 60–250 Hz | wobble 0–0.2 | attack 50 ms, release 400 ms | linear above a 0.2 floor |
| hats, 6–12 kHz | angle tick 7.5° or band jitter of one band | attack 5 ms, release 80 ms | trigger at 0.5 |
| spectrum, 128 bins | map in spectrum mode | per bin `s = max(new, s · 0.85)` | log-scaled frequency axis |
| RMS loudness | tint 0 to 0.5 | attack 200 ms, release 2 s | drop sections only |

In beat-stepped mode, turn the kick punch off. The beat grid already pulses the image.

**MIDI (BeatStep Pro and KeyStep 37)**

- Knob 1: depth. The kick envelope is added on top.
- Knob 2: angle. Or an LFO with one turn per 4 bars.
- Knob 3: bands. Pad 6 toggles beat-step.
- Knob 4: wobble.
- Pads 1 to 5: shape (linear, radial, luma, noise, spectrum).
- KeyStep mod strip: feed.

**Timing and structure**

- Angle: one full turn per 4 bars, linear. On hi-hats, an optional 7.5° tick instead.
- Depth: base 0.35, kick punch +0.35.
- Beat-stepped mode: switch it on for a phrase. Bands = 4 gives four strips showing now, 1, 2 and 3 beats ago, which reads clearly on a big wall.
- Build-up: depth ramps from 0.2 to 0.7 over the last 4 bars, and bands rise from 2 to 16.
- Drop: cut to radial tunnel, depth 0.8, tint 0.5, angle rotation stops, kick punch stays on.
- Break: depth eases to 0.1 over 2 bars, the map freezes, dry/wet goes to 50 %, feed 0.3 so leftover streaks drift.
- Never hold one fixed map for more than about 16 bars. A static map is the main reason this effect gets boring.

**Layering.** Run it as a processing layer on top of any clip, camera or feedback source, and fade between processed and unprocessed with an opacity fader. It stacks with kaleidoscope symmetry and feedback. An optional soft luma key of the source (threshold 0.15, softness 0.1) as a matte on the wet layer keeps smears on the subject and the dark background clean.

**Performance and LED walls**

- Most D3D11/D3D12 GPUs cap 2D textures at 16384 px per side. With 1080p tiles you get at most 120 frames and 1 GB, so avoid that.
- The ISF route copies the whole atlas each frame. At 7680×4320 RGBA8 that is about 16 GB/s at 60 fps, fine on a desktop GPU and too much on laptops.
- Frame rate: delay is counted in frames. If output drops from 60 to 30 fps, delays double in seconds and beat-stepped bands drift off the beat. Lock the output fps.
- Wide walls: give tiles the output aspect. A 5760×1080 strip uses tiles of 1440×270. Compute the linear ramp with the real aspect, or it only covers part of the wall.
- Turn blacks fully off after the crush. Static source backgrounds often sit at 5–10 % grey, which reads as haze on LEDs. Keep peak white near 90 %, since the tunnel mode spreads bright objects over many pixels.
- Too much depth, a fixed map or a source with little motion all lead to mush or nothing. Keep a clear moving subject and vary the map.

## Variations

1. **Radial tunnel map.** Distance from the centre as the offset gives a Stargate-like warp tunnel, and the kick pushes the depth.
2. **Beat-stepped delay.** Quantize the offset to whole beats or bars, so the lower half shows a bar ago and the top half shows now.
3. **Self-feeding scan.** Feed the output back into the buffer for streaks that persist and drift.
4. **Audio-shaped map.** Build the offset map from a spectrum or waveform texture, so the sound shapes the time warp.
5. **Depth-from-brightness map.** Use the clip's own luminance as the offset, so bright parts lag behind dark parts.

## Did you know?

- Making analog slit-scan was slow. A 10-second sequence at 24 fps needed at least 240 adjustments of the mask ([Wikipedia](https://en.wikipedia.org/wiki/Slit-scan_photography)).
- The same three warp-entry shots made by ILM were reused throughout *Star Trek: The Next Generation* ([Wikipedia](https://en.wikipedia.org/wiki/Slit-scan_photography)).
- Greg Ercolano's *2001 Unwrapping the Slit Scan Sequences* (2001) reversed the Trumbull footage back into source images ([catalogue](https://flong.com/archive/texts/lists/slit_scan/index.html)).
- In *Slithering*, the dancers were filmed before the software was finished, and on set they only saw a still preview ([VICE](https://www.vice.com/en/article/post-processing-technique-really-makes-these-dancers-slither-video/)).

## Sources

- [Levin, An Informal Catalogue of Slit-Scan Video Artworks and Research](https://flong.com/archive/texts/lists/slit_scan/index.html)
- [Wikipedia, Slit-scan photography](https://en.wikipedia.org/wiki/Slit-scan_photography)
- [Rozin, Time Scan Mirror](http://www.smoothware.com/danny/timescan.html)
- [Make, The Khronos Projector](https://makezine.com/article/craft/the-khronos-projector)
- [SIGGRAPH History Archives, Khronos Projector](https://history.siggraph.org/experience/khronos-projector-by-cassinelli-and-ishikawa)
- [VICE, Slithering](https://www.vice.com/en/article/post-processing-technique-really-makes-these-dancers-slither-video/)
- [Derivative, Slit Scan using the Time Machine TOP](https://learn.derivative.ca/courses/200-intermediate/lessons/202-tops-intermediate/topic/slit-scan-using-the-time-machine-top/)
- [The Interactive & Immersive HQ, Easy Slitscan in TouchDesigner (YouTube)](https://www.youtube.com/watch?v=1ACBmGYB8uw)
- [Epic Games, Begin Draw Canvas to Render Target](https://dev.epicgames.com/documentation/unreal-engine/BlueprintAPI/Rendering/BeginDrawCanvastoRenderTarget)
- [Godot docs, SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)
- [BeatScanner (Gumroad, untested)](https://tombass.gumroad.com/l/beatscanner)
- [Slice Displace (Gumroad, untested)](https://rexaffects.gumroad.com/l/gmtca)
