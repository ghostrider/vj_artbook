---
tags:
  - feedback
  - analog-video
  - recursion
  - video
---

# Analog Video Feedback Loops

> Point a camera at its own monitor and the picture falls into itself: a tunnel that grows, locks and mutates under your hands.

![A camera-at-monitor feedback pattern with octagonal symmetry](https://upload.wikimedia.org/wikipedia/commons/thumb/6/69/Video_feedback_octagon_pattern.jpg/1280px-Video_feedback_octagon_pattern.jpg)
*A camera-at-monitor feedback pattern with octagonal symmetry. Image: [Deathgleaner](https://commons.wikimedia.org/wiki/File:Video_feedback_octagon_pattern.jpg), [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0).*

## The idea

Video feedback is what you get when a camera looks at a monitor that shows the camera's own live output. Each captured frame contains the previous one, so the image recurses ([Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)). Camera angle, zoom, focus and the monitor's brightness and contrast all shape the result. Tiny changes give very different patterns: concentric, rotating or irregular.

Video artists began to use it as an image source, not a glitch, in the late 1960s and early 1970s. British TV called it "howlround" and put it in the Doctor Who titles. Physics followed in 1984, when James P. Crutchfield treated the loop as a space-time analog computer ([UC Davis Chaos group](https://csc.ucdavis.edu/~chaos/chaos/pubs/stdvf-title.html)).

On a club screen it is a self-propelled image. A small bright seed (a logo, a line of text, a dancer) becomes a spiralling tunnel of nested copies. It grows, locks into a pattern, then collapses or mutates when you nudge a knob. You do not animate it. You steer it. Knobs were the original interface, so a KeyStep or BeatStep Pro fits naturally.

## Origins and references

- **Steina and Woody Vasulka, *Elements* (1971)**: early feedback through a keyer and colorizer. [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- **Skip Sweeney**: worked with feedback from the late 1960s, co-founded Electric Eye (1969) and Video Free America (1970). His tools were a studio camera and a monitor with contrast, brightness and zoom controls. He called feedback "a religion - a wave to ride." [Vasulka archive](https://www.vasulka.org/archive/eigenwelt/pdf/148-149.pdf)
- **Skip Sweeney, *Illuminatin' Sweeney* (1975)**: feedback combined with synthesizers (Moog, Vidium). [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- **Doctor Who title sequence (1963-1973), Bernard Lodge**: monochrome howlround, found by engineer Norman Taylor. Replaced in 1973 by slit-scan. [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- **Queen, "Bohemian Rhapsody" video (1975), dir. Bruce Gowers**: the cascading Freddie Mercury images were made in-camera with a monitor fed back into a camera (per Wikipedia, citing the BBC). [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- **VJ Mandala (Adem Jaffers)**: used feedback in Melbourne's rave scene and the *Cyberthon* broadcasts (1991-1995). [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- **James P. Crutchfield, "Space-Time Dynamics in Video Feedback" (1984)**: models the loop as an iterated functional equation and as a reaction-diffusion PDE. [UC Davis](https://csc.ucdavis.edu/~chaos/chaos/pubs/stdvf-title.html)
- **Dave Blair, The Light Herder**: a modern kinetic sculpture that makes fractals with no computer. HD/4K monitors with analog hue, saturation, brightness and contrast knobs, up to three cameras, 50/50 beam-splitter glass, a Roland V-02HD switcher/keyer and delay converters. [thelightherder.com](https://www.thelightherder.com/), [Hackaday](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/)

## Visual language

- **Shapes:** tunnels and spirals receding to a centre, nested copies of the frame, rotating arms, lattices of bright spots with n-fold symmetry. Rotation and magnification set the symmetry ([Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)).
- **Edges:** hard-edged input, such as a white shape on black, turns into glowing, organic filaments.
- **Colour:** hue shifts a little on each pass, so the rings run through the colour wheel. Or keep it monochrome, amber on black, for the howlround look.
- **Motion:** self-propelled. It grows, locks, then mutates. Rhythm comes from the loop delay, the frame lag.
- **Two flavours:** analog is soft, chromatic, noisy and slow. Digital is crisp and can be pushed to fractal sharpness.

## See it

- [The New 4K Video Feedback Fractal Device (YouTube, channel The Light Herder)](https://www.youtube.com/watch?v=8XjQ10ph4yo) — Dave Blair's analog camera-at-screen rig making fractals in motion.
- [4K Analog Video Feedback Fractal Device Finally Complete (YouTube, channel The Light Herder)](https://www.youtube.com/watch?v=koDCabeh5kQ) — the build and first results of the same device.
- [The Light Herder](https://www.thelightherder.com/) — artist project page with hardware description and video links.
- [Hackaday: Analog Optical Feedback Generates 4K Fractals](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/) — write-up of the rig and its knobs.
- [Wikimedia Commons: Video feedback octagon pattern](https://commons.wikimedia.org/wiki/File:Video_feedback_octagon_pattern.jpg) — a still of a symmetric camera-at-monitor feedback pattern.

## Code design

The core is one equation: `out = softclip(floor_cut(gain * colour(optics(T(prev)))) + seed * S)`. `prev` is last frame's output, `T` is a zoom, spin and offset transform, and `S` is a keyed seed. This is the "iterated functional equation" view of video feedback ([UC Davis](https://csc.ucdavis.edu/~chaos/chaos/pubs/stdvf-title.html)). Two buffers ping-pong, so a pass never reads and writes the same texture. All per-frame amounts are scaled by `dt`, so the look survives frame drops.

Inside the loop, small blur and contrast play the diffusion and reaction terms. A chroma split gives rainbow fringes. A hue step per frame colours each ring. Sharpening and palette work stay outside the loop, because inside they compound every generation.

Stability is the craft. Gain above 1 runs to white, below 1 to black. The design rebuilds what a real monitor and camera do for free: a soft ceiling at about 0.95, decay before gain, an auto-gain meter that watches mean luma, a floor cut of 1/255 per frame, a 16-bit float buffer, and a small seed floor so a decayed loop restarts. A panic pad clears the buffer.

The same chain can also run as a real camera aimed at a hooded monitor, with Resolume as the knob board.

=== "Resolume Wire"
    - **Nodes:** seed input, ISF `Seed` (keys any source into thin bright shapes), ISF `Loop` (one generation), ISF `Post` (colourise, black crush, bloom, grain).
    - **State:** declare a PERSISTENT FLOAT pass target in the `Loop` ISF and read the previous state by its name ([ISF multipass reference](https://docs.isf.video/ref_multipass.html)). The auto-gain can be a second 1x1 persistent pass averaging a coarse grid of taps.
    - **Caveat:** Resolume's ISF page does not cover multipass or persistent buffers, and support depends on your Wire version. Test the persistent buffer first. If it does not keep its state, fall back to a Spout round trip with the ISF node doing only the per-pass maths. Wire has no vertex shaders, which is fine here ([Resolume ISF](https://resolume.com/support/en/isf)).
    - **Parameters:** float inputs with MIN/MAX for zoom (0.94-1.05), spin, hue, feedback (0.9-1.1), seed and fold, plus an event input for the panic clear. Publish them to the Arena effect panel for MIDI mapping.

=== "Unreal Engine 5.7"
    - **Targets:** RT_A and RT_B (RGBA16f, 1920x1080, border addressing), RT_Seed and a 1x1 RT_AGC.
    - **Actor:** `BP_FeedbackLoop` draws M_Seed, then M_Loop (with `Prev` set to the current buffer), then M_Meter on Tick with `Draw Material to Render Target`, then swaps A and B.
    - **Seed:** a SceneCapture2D looking at a Niagara system (sprite burst on the kick, ribbon spirals) writes RT_Seed. Niagara becomes the seed generator.
    - **Control:** put zoom, spin, hue, feedback, seed and fold in a Material Parameter Collection, expose them in a Remote Control preset, and bind them through OSC and MIDI. A `PanicClear` function clears both targets.
    - **Output:** show RT_B on a full-screen widget or a plane, and send it over Spout or NDI so Resolume can take it as a layer.

=== "GODOT"
    - **Ping-pong:** `SubViewport LoopA` runs the loop shader and reads `LoopB`'s ViewportTexture as `prev`. `LoopB` shows LoopA through a TextureRect. Both use `UPDATE_ALWAYS` and `CLEAR_MODE_NEVER` ([Godot SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)), with HDR 2D on for float precision. Expect one or two frames of loop delay.
    - **Seed:** a third SubViewport holding a Label, a logo Sprite2D, a GPUParticles2D burst or a video clip.
    - **Auto-gain:** a small SubViewport averaging taps of LoopA. Avoid `get_image()` readback every frame, as it stalls the CPU.
    - **Control:** expose `zoom`, `spin`, `hue_step`, `feedback`, `seed_amt` and `fold` as uniforms, driven from MIDI input events and an audio bus spectrum analyzer. A final TextureRect carries the post shader.

!!! note "Arena route, untested"
    Resolume's FeedbackPro effect (Arena 7.18.2 and later) luma-keys the footage and lets you move, scale and hue-shift the feedback ([Resolume blog](https://www.resolume.com/blog/23113)). It is the no-code route. The other Arena route is a Spout round trip: Arena receives its own Spout output on a bottom layer, with Transform, Hue Rotate and a gain effect on top of it. This round trip is untested. Depending on routing it adds one or more frames of delay, and the Spout path is 8 bit, so add a small black floor against ghosting. Try it on your version before a show.

## On stage

!!! warning "No white-out, no fade to black"
    A loop with total gain above 1 runs to white. Below 1 it runs to black. Both kill the set.

    - Keep the soft ceiling at about 0.95 and never use hard clipping. Warm white, never pure #FFFFFF.
    - Keep the auto-gain on (strength about 0.7, target mean luma 0.18-0.30).
    - Keep a seed floor so a decayed loop always restarts.
    - Keep the floor cut on so blacks reach true 0. On LED walls, grey 3/255 is visibly on.
    - Never drive the feedback (gain times decay) from audio. It fights the auto-gain and causes white flashes.
    - Keep a MIDI pad for the panic clear.

**Audio reactivity**

| band | drives |
|------|--------|
| Kick (40-120 Hz) onset | zoom pulse of +0.02 to +0.04, released over about 120 ms |
| Bass envelope (60-250 Hz) | spin speed, up to +50 % |
| Snare / clap (1-4 kHz) onset | seed flash to 1.0 for 2 frames, with a 150 ms hold-off |
| Hats (8-14 kHz) | offset jitter of +/-0.002 uv, seeds fine detail |
| Full-band RMS | hue step x0.5-1.5, slow release |

**MIDI mapping**

| control | parameter |
|---------|-----------|
| BeatStep Pro encoder 1 | zoom (0.94-1.05 per frame, default 0.985) |
| Encoder 2 | spin (-3 to +3 degrees per frame, default 0.4) |
| Encoder 3 | hue drift (-6 to +6 degrees per frame, default 1.5) |
| Encoder 4 | fold (0-1), ramp it over a build |
| KeyStep 37 mod strip or fader | feedback (0.9-1.1), ridden by hand only |
| Pad velocity | snare seed flash |
| Pad | drop: flip the sign of spin and hue |
| Pad | panic clear |

**Timing and structure**

- Base state: zoom 0.985, spin 0.4 degrees, hue step 1.5 degrees, auto-gain 0.7.
- Sweep zoom and spin over 8-16 bars with a sine ease. Rare hits beat constant motion, because a fixed setting locks into one static pattern.
- BPM lock: one full spiral turn per 16 beats. At 128 BPM that is 7.5 s, or 0.8 degrees per frame at 60 fps.
- Break: lower the auto-gain target to 0.1, seed off, spin 0. The tunnel fades to embers but never dies.
- Build: contrast 1.0 to 1.25 and fold 0 to 0.6 over the last 8 bars.
- Drop: flip spin and hue, jump zoom to 1.01 for a bar, then return to receding.

**Layering**

- Use the loop as a texture over other material, as a luma matte, or as a displacement source for another layer.
- Add the dry seed on top at 20-50 % so the "now" shape stays readable in the tunnel.
- Output the loop over Spout or NDI and feed it back into Resolume as a source.
- The book already has [time displacement](time-displacement-slit-scan.md). Use its output as the loop's seed.

**Real camera rig**

- Path: camera, capture card or NDI, a Resolume layer, then a dedicated loop monitor that the camera films. The audience sees the main composition, not the loop monitor.
- Hood the camera and monitor in a black box or under black molton. Stage light adds an offset every pass and pushes the loop to white.
- Lock auto exposure, white balance and focus. Defocus slightly to avoid moiré from the pixel grid.
- Latency is several frames. Keep the capture chain identical during a show, or the loop speed changes.
- Keep the hue step under about 4 degrees per frame at full brightness, and follow venue flash rules.

## Variations

- **Two-loop cross-feed:** two chains each mix in a fraction of the other, like Blair's two monitor structures that influence each other. [Hackaday](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/)
- **Mirror / beam-splitter fold:** sum a mirrored second transform of the previous frame into the loop to emulate half-mirrored glass and get kaleidoscopic fractal forms. [thelightherder.com](https://www.thelightherder.com/)
- **Frame-delay stack:** feed back from N frames ago for echo rings. Blair's rig has delay units of up to 30 frames (per Hackaday comments). [Hackaday](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/)
- **Seeded feedback:** inject text, a logo or a live camera of the booth. Doctor Who's loop was fed a logo. [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- **Reaction-diffusion cousin:** add blur and threshold in the loop, following Crutchfield's second model, for Turing-like spots. [UC Davis](https://csc.ucdavis.edu/~chaos/chaos/pubs/stdvf-title.html)
- **Mono howlround:** zero hue step with an amber seed for the Doctor Who look.

## Did you know?

- The 1973 Doctor Who titles dropped feedback for slit-scan, inspired by the 2001 star gate. [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- Douglas Hofstadter's *I Am a Strange Loop* (2007) has a chapter, "On Video Feedback", that uses the camera-monitor loop to illustrate recursion. [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- Feedback with pixel-based monitors can produce stationary fractals (*Nature*, 2001). [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- Blair's rig drives LCD display settings from knobs via a Teensy over RS-232. [Hackaday](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/)

## Sources

- [Video feedback, Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- [Skip Sweeney entry, Vasulka archive](https://www.vasulka.org/archive/eigenwelt/pdf/148-149.pdf)
- [Crutchfield, Space-Time Dynamics in Video Feedback](https://csc.ucdavis.edu/~chaos/chaos/pubs/stdvf-title.html)
- [The Light Herder](https://www.thelightherder.com/)
- [Hackaday, 2026-08-27](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/)
- [Derivative Learn: Creating a Feedback](https://learn.derivative.ca/courses/100-fundamentals/lessons/102-tops-working-with-images/topic/creating-a-feedback/)
- [Wikimedia Commons: Video feedback octagon pattern](https://commons.wikimedia.org/wiki/File:Video_feedback_octagon_pattern.jpg)
- [Resolume blog: FeedbackPro](https://www.resolume.com/blog/23113)
- [Resolume: ISF in Wire](https://resolume.com/support/en/isf)
- [ISF Multi-Pass and Persistent Buffer Reference](https://docs.isf.video/ref_multipass.html)
- [Godot SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)
