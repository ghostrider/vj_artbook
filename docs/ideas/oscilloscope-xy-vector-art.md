---
tags:
  - vector
  - laser
  - audio-reactive
  - lines
---

# Oscilloscope XY Vector Art

> One glowing line, drawn in a single stroke: the sound itself is the pen.

![The screen of an analogue Goldstar oscilloscope in XY mode](https://upload.wikimedia.org/wikipedia/commons/0/0f/Analog_oscilloscope.jpg)
*The screen of an analogue Goldstar oscilloscope in XY mode. Image: [premek.v](https://commons.wikimedia.org/wiki/File:Analog_oscilloscope.jpg), [Public domain](https://commons.wikimedia.org/wiki/File:Analog_oscilloscope.jpg).*

## The idea

In XY mode an oscilloscope ignores its time base. The left audio channel moves the beam horizontally and the right channel moves it vertically ([IEEE Spectrum](https://spectrum.ieee.org/jerobeam-fendersons-trippy-oscilloscope-music)). One sample pair (L, R) is a point. Played fast, the points become a continuous line.

Two sine waves at different frequencies draw a Lissajous figure. A ratio of 1 gives an ellipse, a rational ratio gives a closed pattern, and an irrational ratio makes the figure look like it rotates ([Wikipedia](https://en.wikipedia.org/wiki/Lissajous_curve)). Any drawing, such as an SVG path or a 3D model, can also be turned into a sample stream.

On a club screen this gives a laser-like image that is never broken. The shape is drawn in one stroke. Brightness builds where the beam lingers or retraces, and motion is smooth because it is the signal itself.

## Origins and references

- Math: Nathaniel Bowditch studied the curves in 1815, and Jules Antoine Lissajous studied them in more detail in 1857 ([Wikipedia: Lissajous curve](https://en.wikipedia.org/wiki/Lissajous_curve)).
- Ben F. Laposky began with pendulum tracings in 1946 and photographed oscilloscope patterns he called "Oscillons". *Scripta Mathematica* published the first ones in 1952. His "Electronic Abstractions" show toured the US and France from 1952 to 1961. Colour came from motorized rotating filters ([Wikipedia: Ben F. Laposky](https://en.wikipedia.org/wiki/Ben_F._Laposky)).
- Jerobeam Fenderson and Hansi Raber: the Kickstarter-funded album *Oscilloscope Music*. The stereo audio itself draws the shapes on an analogue scope ([Creative Applications](https://www.creativeapplications.net/project/oscilloscope-music-jerobeam-fenderson-and-hansi-raber/)).
- Fenderson's videos use a Tektronix D11 5103N scope. The "dancing mushrooms" video used an RME Fireface UC at 192 kHz ([IEEE Spectrum](https://spectrum.ieee.org/jerobeam-fendersons-trippy-oscilloscope-music)).
- *Primer* by DJ_Level_3 and Marv1994 (BUS ERROR Collective) builds from simple waveforms to spinning, morphing shapes. It won the Wild category and the Crowd Favorite award at Revision 2025 ([Hackaday](https://hackaday.com/2025/04/26/amazing-oscilloscope-demo-scores-the-win-at-revision-2025)).
- *N-Spheres* EP by Jerobeam Fenderson and Hansi3D: five tracks (Function, Intersect, Attractor, Flux, Core) ([oscilloscopemusic.com](https://oscilloscopemusic.com/)).
- Hobby route: a Teensy 3.2 with an Audio Shield plays stereo WAV files into an antique scope ([PJRC](https://www.pjrc.com/oscilloscope-music-visualizer/)).
- Tools: [osci-render](https://github.com/jameshball/osci-render) (SVG, .obj, text, Lua, Blender, MIDI control) and OsciStudio (Blender link).

## Visual language

A single bright line, usually green phosphor, on pure black. The core burns almost white and the edges fall off into saturated green, with a soft halo around it. Where the beam slows down, turns a corner or retraces, the line gets brighter. Where it moves fast it thins to a faint thread. This is the scope's own brightness logic ([woscope](https://m1el.github.io/woscope-how/)).

Shapes run from Lissajous knots (3:2, slowly precessing) to spinning wireframe cubes and one-stroke logos. The retrace lines between letters stay visible, and that is part of the look. Every shape leaves a short afterglow tail, so motion reads as smeared light, not frames. The mood is lab equipment turned instrument: cold, precise, hypnotic.

Palette: green phosphor `#33FF66` on true black, core `#E8FFF0`. Amber `#FFB000` for breaks, cyan `#29E6FF` for drops, and magenta `#FF2E88` in a Laposky mode where the hue turns slowly like a filter wheel.

## See it

- [How To Draw Mushrooms On An Oscilloscope With Sound](https://www.youtube.com/watch?v=rtR63-ecUNo) — mushrooms drawn on an oscilloscope by sound (YouTube, Jellybean Fenderson)
- [Khrậng](https://www.youtube.com/watch?v=vAyCl4IHIz8) — oscilloscope music piece (YouTube, Jellybean Fenderson)
- [IEEE Spectrum feature](https://spectrum.ieee.org/jerobeam-fendersons-trippy-oscilloscope-music) — write-up with the stills and setup photos
- [Oscilloscope Music site](https://oscilloscopemusic.com/) — N-Spheres EP videos and OsciStudio
- [osci-render](https://github.com/jameshball/osci-render) — tool with a built-in software oscilloscope

## Code design

The look is a software scope. A stereo sample stream (real audio or generated oscillators) becomes a chain of line segments. Each segment is drawn as a Gaussian beam, dimmer where it is long and fast, and brighter where it is short or retraced. Segments add up (additive blend). Each is faded by its age in signal time, which gives the phosphor afterglow independent of frame rate. A tone map turns intensity into green that burns to white, and bloom carries the glow.

Key rules from the design:

- Draw the whole block of samples every frame, plus a history window. One point per frame gives noise, not a shape. About 800 new sample pairs arrive per frame at 48 kHz and 60 fps.
- Use a ring buffer of 8192 stereo frames and always draw a fixed window ending at the write head.
- Fade by `exp(-age / tau)` in seconds, never by a per-frame factor without `dt`.
- Long smears (over 300 ms) need a 16-bit float feedback buffer. 8 bit leaves a grey ghost.
- Ordinary stereo music is mostly mono, so raw XY gives a diagonal smear. Rotate 45 degrees (`x = (L - R) / sqrt(2)`, `y = (L + R) / sqrt(2)`) for a goniometer view. Use raw XY only for tracks composed for the scope.
- For a rotating figure, keep a rational ratio and add a small detune (0 to 2 Hz) on the x oscillator. An irrational ratio smears.

=== "Resolume Wire"
    Wire has no vertex shaders ([Resolume ISF docs](https://resolume.com/support/en/isf)), so the beam is a per-pixel loop inside one ISF node.

    - Chain: parameters, ISF node "XYBeam" (beam plus tone map in one pass, rendered at 960x540), Resize to output, Bloom, output.
    - Each pixel loops over 256 to 512 segments, skips those outside the segment's bounding box grown by 3 sigma, and adds the Gaussian beam integral times the age weight.
    - Generated signals: ratio, detune, phase, window, tau, width, intensity and scale drive the figure.
    - Live audio: the ISF spec defines an `audio` input with one row per channel ([ISF spec](https://docs.isf.video/ref_json.html)). Resolume's ISF page does not say whether Wire fills it. **Test it:** check whether Wire fills the ISF audio input. If it does, set `MAX` to 512 to decimate and use the goniometer rotation. If not, use generated signals driven by Wire's audio analysis.
    - A feedback tail needs a second pass with `PERSISTENT` and `FLOAT`. Only use it if your Wire version supports multipass ISF. Otherwise the history fade alone covers tau up to about 300 ms.
    - Control: low band to envelope (attack 5 ms, release 180 ms) to `punch`. Beat counter per phrase to `ratio`. BPM to `spin`. A 2-bar LFO to `phase`.
    - In Arena, Bloom is available since 7.2 ([Resolume blog](https://resolume.com/blog/19880)). Load the Wire effect on layer 2 in Add mode. Test ISF loading in your Arena version before the gig.

=== "Unreal Engine 5.7"
    Main route: a Niagara GPU sprite emitter with one sprite per beam segment.

    - Emitter `NE_XYBeam`: GPU Compute Sim, 2048 particles spawned at t = 0, infinite lifetime. Each particle is a segment with index `i = ExecutionIndex`.
    - A Scratch Pad module computes both end points. Generated: `t_i = Engine.Time - (N - 1 - i) * Window / N` with user parameters Ratio, Detune, Phase, F0, Window, Scale, Yaw and Pitch.
    - Sprite position is the segment midpoint, size is (l + 6 sigma, 6 sigma), aligned along the segment. Pass `l` and `age` to the material through a Dynamic Parameter.
    - Material `M_Beam`: Unlit, Additive. It rebuilds segment-space coordinates from the sprite UV and applies the beam integral times the age weight times Intensity (default 6).
    - Live audio: Niagara has an Audio Oscilloscope data interface that reads a submix ([UE forum](https://forums.unrealengine.com/t/niagara-oscilloscope-attribute-wont-package/472063)). **Test it:** check in 5.7 how the Audio Oscilloscope exposes the left and right channels separately before building on it. The same thread reports packaged builds hanging on launch, with `[Audio] UseAudioThread=false` in DefaultEngine.ini as a user workaround. Test a packaged build, not just the editor. Fallback: write the stereo ring buffer to a texture from C++.
    - Camera: orthographic CineCamera. Post Process Volume: Bloom 1.4, threshold 0.35, manual exposure, no vignette. Turn AA off or use FXAA, because TSR smears thin fast lines into ghosts.
    - Long tails: SceneCapture2D into a 16-bit float render target, then Draw Material to Render Target with two targets swapped.
    - Remote Control: a preset exposing Ratio, Detune, Tau, Width, Scale, Source and Palette, bound to OSC and MIDI. Output by Spout or NDI to Arena, in Add mode.

=== "GODOT"
    - Audio in: an AudioEffectCapture on the music bus (or a bus fed by AudioStreamMicrophone for line-in). `get_buffer` returns stereo frames as a PackedVector2Array. Set `buffer_length` to about 0.05 s for low latency ([Godot AudioEffectCapture](https://docs.godotengine.org/en/stable/classes/class_audioeffectcapture.html)).
    - Each frame, copy the available samples into an 8192-frame ring, write it into an RGF image with `set_data`, and update an ImageTexture. Pass the head index and sample rate as uniforms. Do not call `set_pixel` per sample.
    - Beam: a MultiMeshInstance2D with a 1x1 quad and 2048 instances inside a SubViewport with HDR 2D. The canvas_item shader uses `blend_add` and `unshaded`. `vertex()` uses `INSTANCE_ID` and `VERTEX_ID` to fetch the two points and place the quad corners ([Godot canvas_item shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/canvas_item_shader.html)). `fragment()` computes the beam integral times the age weight.
    - If segments vanish at screen edges, enlarge the mesh bounds or skip culling.
    - Feedback: a second SubViewport with clear mode Never, using HDR 2D so the trail fades to black. A ColorRect shader does the tone map, with glow or a two-tap bloom.
    - Quick test: Line2D with updated points. It works but has uniform brightness and no beam physics.
    - Map MIDI through InputEventMIDI, or OSC through an addon.

!!! note "osci-render and GPLv3"
    osci-render's source is GPLv3, and its name, logos and branding are not covered by that licence ([GitHub](https://github.com/jameshball/osci-render)). Using the app or plugin live to generate the XY signal, routed by a virtual audio cable into Godot, UE or a real scope, carries no obligation for your visuals. If you copy or port its code into a shader or tool you distribute, that work must be released under GPLv3 too. This design is built from the published math and the woscope explanation, not from osci-render code.

## On stage

**Audio reactivity**

| band | drives | smoothing |
|------|--------|-----------|
| Kick, 40-120 Hz | scale +15 % (punch) | attack 5 ms, release 180 ms |
| Bass, 60-250 Hz | amplitude, or ratio step on strong hits in build-ups | attack 60 ms, release 500 ms |
| Mids, 500 Hz-2 kHz | phase nudge of +/-0.3 rad | attack 50 ms, release 400 ms |
| Hats, 6-12 kHz | beam width +0.5 px, small XY jitter | attack 3 ms, release 60 ms |
| RMS loudness | intensity and bloom, drops only | attack 300 ms, release 2 s |
| Raw stereo waveform | the picture itself (live XY or goniometer) | none, normalise with a slow peak follower |

Turn jitter off in logo mode, or the text becomes unreadable.

**MIDI mapping (KeyStep 37 / BeatStep Pro)**

| control | parameter |
|---------|-----------|
| BeatStep Pro pads 1-6 | ratio: 1:1, 1:2, 2:3, 3:4, 3:5, 4:5 |
| Pads 7-8 | source: generated, wireframe, live XY, goniometer |
| Knob 1 | spin (detune 0-2 Hz), plus a BPM-lock switch for one turn per bar |
| Knob 2 | afterglow (tau 5-300 ms) |
| Knob 3 | beam width, 1-6 px |
| Knob 4 | punch (scale 0.5-1.3) |
| KeyStep mod strip | live sweeps of afterglow |

**Set arc**

- Build-up: tau from 20 to 120 ms over 8 bars, detune 0 to 1 Hz, bloom 1.0 to 1.8.
- Drop: hard ratio jump, cyan palette, feedback on, beam +1 px for one bar.
- Break: amber palette, a single ellipse breathing slowly, or the goniometer view of the live music. Very sparse, so the next drop hits harder.
- Change the shape family (Lissajous, wireframe, logo, live audio) at least every 16 bars. A static ellipse is the main boredom risk.

**Layering**

- Add the beam layer over footage. Use Screen over bright footage to avoid clipping.
- Use the beam's luma as a mask, so footage shows only through the line.
- Never feed the bloomed image back into the feedback buffer. The halo would grow until the screen washes out.

**Tips**

- LED walls: the background must be exactly black, or bloom haze lifts the whole wall. A 1-px line is invisible from 20 m, so use 2 to 4 px and let bloom spread it.
- Keep audio capture at 50 ms or less so figures stay in sync with the PA.
- Per-pixel Wire route: about 265 million segment tests per frame at 960x540 with 512 segments. Measure first, and fall back to 256 segments or 640x360 on laptops.
- On ultra-wide strips, place three figures side by side with phase offsets of 2 pi / 3 instead of stretching one.
- Common faults: a mono track in raw XY (diagonal line only), DC offset (figure drifts off-centre), and TSR ghosting in UE.

## Variations

1. Phosphor simulator with variable persistence, gamma and a curved CRT mask.
2. Lissajous morphing: step the ratio on the beat between rational values, then drift towards irrational ones.
3. Logo or typography as one stroke, with a wobble from the hi-hats.
4. 3D wireframe objects rotating on BPM, with depth-based brightness.
5. Laser output: OsciStudio 7 supports WickedLasers projectors ([oscilloscopemusic.com](https://oscilloscopemusic.com/)). Or add Laposky-style rotating colour filters.
6. Real scope: film an analogue scope fed by an audio interface and bring it into Arena through a capture card.

## Did you know?

- Laposky's work was published over 160 times and shown at over 200 exhibitions ([Wikipedia](https://en.wikipedia.org/wiki/Ben_F._Laposky)).
- The Lissajous XY display is also used in professional audio to check stereo phase ([Wikipedia](https://en.wikipedia.org/wiki/Lissajous_curve)).
- *Primer* was recorded on a BK Precision 2120 scope with a green display ([Hackaday](https://hackaday.com/2025/04/26/amazing-oscilloscope-demo-scores-the-win-at-revision-2025)).
- Max/MSP and Pure Data patches were released along with the *Oscilloscope Music* album ([Creative Applications](https://www.creativeapplications.net/project/oscilloscope-music-jerobeam-fenderson-and-hansi-raber/)).

## Sources

- [Creative Applications: Oscilloscope Music](https://www.creativeapplications.net/project/oscilloscope-music-jerobeam-fenderson-and-hansi-raber/)
- [Wikipedia: Ben F. Laposky](https://en.wikipedia.org/wiki/Ben_F._Laposky)
- [Wikipedia: Lissajous curve](https://en.wikipedia.org/wiki/Lissajous_curve)
- [Hackaday: Primer at Revision 2025](https://hackaday.com/2025/04/26/amazing-oscilloscope-demo-scores-the-win-at-revision-2025)
- [IEEE Spectrum](https://spectrum.ieee.org/jerobeam-fendersons-trippy-oscilloscope-music)
- [oscilloscopemusic.com](https://oscilloscopemusic.com/)
- [osci-render on GitHub](https://github.com/jameshball/osci-render)
- [PJRC oscilloscope music player](https://www.pjrc.com/oscilloscope-music-visualizer/)
- [woscope: how it works](https://m1el.github.io/woscope-how/)
- [ISF JSON reference](https://docs.isf.video/ref_json.html)
- [Resolume ISF docs](https://resolume.com/support/en/isf)
- [Resolume blog: Bloom and Pixel Blur](https://resolume.com/blog/19880)
- [UE forum: Niagara oscilloscope](https://forums.unrealengine.com/t/niagara-oscilloscope-attribute-wont-package/472063)
- [Godot AudioEffectCapture](https://docs.godotengine.org/en/stable/classes/class_audioeffectcapture.html)
- [Godot canvas_item shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/canvas_item_shader.html)
