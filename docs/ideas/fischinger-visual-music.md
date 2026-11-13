---
tags:
  - art-history
  - visual-music
  - shapes
  - sync
---

# Fischinger Visual Music

> Circles, bars and dots land on the beat, then drift away into black: a shape sequencer in the language of a film pioneer.

![Printed programme of a 1953 San Francisco Museum of Art evening with Fischinger's films and a Lumigraph demonstration. A text document, not a film still](https://upload.wikimedia.org/wikipedia/commons/7/77/An_Evening_with_Oskar_Fischinger_-_San_Francisco_Museum_of_Art_-_23_February_1953.jpg)
*Printed programme of a 1953 San Francisco Museum of Art evening with Fischinger's films and a Lumigraph demonstration. A text document, not a film still. Image: [San Francisco Museum of Art](https://commons.wikimedia.org/wiki/File:An_Evening_with_Oskar_Fischinger_-_San_Francisco_Museum_of_Art_-_23_February_1953.jpg), Public domain (PD-US-no-notice; public domain in the US only).*

## The idea

Oskar Fischinger placed every shape change on a musical event, frame by frame. The shapes are simple, so the eye reads the sync before it reads the content. A VJ does the same job with a MIDI note map: a kick fires a disc, a snare fires a bar, the hi-hats fire dots.

This chapter is an original homage in Fischinger's visual language. It is not a recreation of any of his films. It takes his rules (pure geometry, exact sync, layers as voices, real rests) and builds a live shape sequencer from them.

On a club screen the look works because it is mostly black. Flat shapes sit well over or under other material. The hit is the picture: a shape is fully there on the first frame after the onset, and everything soft happens afterwards.

## Origins and references

- **Oskar Fischinger (1900–1967)** turned to abstract film in 1921 after seeing Walther Ruttmann's *Lichtspiel Opus 1*. He moved to Munich as a full-time filmmaker ([CVM biography](https://www.centerforvisualmusic.org/Fischinger/OFBio.htm)).
- **Studies (from 1929)**: black-and-white films synchronized to music. The early ones were synced to Electrola records, which makes them arguably the first music videos ([Wikipedia](https://en.wikipedia.org/wiki/Oskar_Fischinger)). *Studie Nr. 7* (1931) has an excerpt on CVM's Vimeo channel ([CVM filmography](https://www.centerforvisualmusic.org/Fischinger/Filmography.htm)).
- **Kreise (1933–34) and Komposition in Blau (1935)**: color work made with the Gaspar process ([CVM biography](https://www.centerforvisualmusic.org/Fischinger/OFBio.htm)).
- **An Optical Poem (1937, MGM)**: paper circles, triangles and rectangles hung on fine fishing line, shot one frame at a time, set to Liszt's Hungarian Rhapsody No. 2 ([TCM](https://www.tcm.com/articles/246892/an-optical-poem)). The year varies by source; TCM notes production around 1936.
- **Allegretto (1936–43)**: it began as *Radio Dynamics*, synced to a Ralph Rainger tune. Fischinger bought it back and remade it in color ([Wikipedia](https://en.wikipedia.org/wiki/Oskar_Fischinger)).
- **Radio Dynamics (1942)**: a silent film made in secret, so the rhythm comes purely from the image ([Wikipedia](https://en.wikipedia.org/wiki/Oskar_Fischinger)).
- **Motion Painting No. 1 (1947)**: oil on acrylic glass, filmed one brushstroke at a time over about nine months, about 11 minutes, set to Bach's Brandenburg Concerto No. 3 ([Visual Music Archive](https://visualmusicarchive.org/works/w/motion-painting-no-1.html)). It won the Grand Prix at the Brussels experimental film competition in 1949 ([CVM biography](https://www.centerforvisualmusic.org/Fischinger/OFBio.htm)).
- **Fantasia (1940)**: he designed the Bach Toccata and Fugue sequence, then left when Disney made his designs more representational ([CVM biography](https://www.centerforvisualmusic.org/Fischinger/OFBio.htm)).
- **Lumigraph**: a live light instrument, patented in 1955, with shows from 1951 ([CVM Lumigraph](https://centerforvisualmusic.org/Fischinger/Lumigraph.htm)).

## Visual language

- **Pure geometry**: circles, squares, bars, spirals and staffs. Black and white in the Studies, flat saturated color in *Kreise*, *An Optical Poem* and *Allegretto* ([CVM filmography](https://www.centerforvisualmusic.org/Fischinger/Filmography.htm)).
- **Shapes pop out of black on the beat**, then drift or recede in depth. In the homage they spring into place with a small overshoot, sway as if hung on thread, cast a soft shadow, then fade back.
- **Layers are voices.** Each has its own size, speed and depth: a large disc for the kick, a long bar sliding in for the snare, pin-point dots for the hi-hats, a thin arc or spiral that climbs with the melody.
- **A hand-made edge.** Edges wobble slightly and boil at 12 frames per second, like redrawn frames. This is the softness a digital version has to add on purpose.
- **Rests.** Between phrases the screen empties to near black. Silence is part of the language.
- **Three looks**: *Studie* (silver-white light on black), *Poem* (flat paper colors with shadows), *Lumigraph* (colored light drawn by a second performer).

## See it

- [CVM Filmography](https://www.centerforvisualmusic.org/Fischinger/Filmography.htm) — index of the films, with links to official excerpts on CVM's Vimeo channel.
- [Wax Experiments (excerpt), CVM on Vimeo](https://vimeo.com/54587174) — the earliest work, an official excerpt listed by CVM.
- [CVM: The Lumigraph](https://centerforvisualmusic.org/Fischinger/Lumigraph.htm) — photos of the instrument and a note on a silent c. 1969 excerpt of Elfriede Fischinger performing.
- [Motion Painting No. 1, Visual Music Archive](https://visualmusicarchive.org/works/w/motion-painting-no-1.html) — description and context for the film.

CVM warns that YouTube copies of *An Optical Poem* are faded and many are stretched ([CVM filmography](https://www.centerforvisualmusic.org/Fischinger/Filmography.htm)). Use the official sources above.

## Code design

The core is one pass that draws up to eight live shapes as 2D signed distance fields: four voices (bass discs, snare bars, hat dots, melody arcs), two slots each. Each shape has an age since its trigger. A damped spring drives its scale, so it overshoots by about 8 % like paper jerked on a thread. Each voice has its own spring frequency (bass 2 Hz, melody 3 Hz, snare 4 Hz, hats 8 Hz), which makes them read as different instruments. There is no fade-in longer than 30 ms, or the sync goes soft.

Triggering is score-driven. A 16-step pattern per voice acts as the graphic score, and a `density` fader gates it by probability. The coin flip is hashed from bar and step, so a repeated bar repeats its pattern. If the input stays silent for a bar, all shapes release and the screen goes black. Depth pulls shapes toward a vanishing point and grows their shadows. Time-stepped noise at 12 fps gives the boiling edge. An optional Lumigraph layer lights only what crosses a thin virtual sheet, with persistence for after-glow. Optional paint-build stamps past shapes into a buffer that resets every N bars.

The design draws on [CVM Process](https://www.centerforvisualmusic.org/Fischinger/OFProcess.htm) for the score idea and [The Book of Shaders: Shapes](https://thebookofshaders.com/07/) for the distance-field shapes.

=== "Resolume Wire"
    Arena plays a Wire patch as a source ([Resolume ISF support](https://resolume.com/support/en/isf)), so the custom parts live in Wire.

    - **Audio in**: `Spectrum In` delivers 1024 bands, about 23 Hz each at 48 kHz ([Resolume Wire FFT](https://resolume.com/support/en/wire-fft)). Sum bins 2–4 (about 45–115 Hz) for kick, bins 7–11 plus 90–200 for snare, bins 300–520 for hats. Compare each band to its 1 s average (+6 dB) for an onset, with a 90 ms hold-off.
    - **Score gate**: beat clock, step counter (wrap 16), pattern lookup, AND with the onset, then a probability compare against `density` using a hash of bar and step.
    - **Slot logic**: on a gated onset, copy the current age and seed to the previous slot, reset the current age, increment the seed.
    - **MIDI in**: KeyStep 37 note gives melody pitch, velocity and an onset event.
    - **ISF nodes**: "Voices" is a single pass (shapes, shadows, staff, post). "Lumigraph" is a stamp plus persistence pass. Multipass and persistent buffers are in the [ISF JSON reference](https://docs.isf.video/ref_json.html), but Wire support varies by version. If your version keeps no persistent buffer, use Wire's feedback node for the decay.
    - **Arena side**: layer 1 solid #0A0908, layer 2 the Voices patch (Normal), layer 3 Lumigraph (Add, opacity on a fader), layer 4 optional video masked by the Voices alpha ([Resolume effects](https://resolume.com/support/en/effects)).

=== "Unreal Engine 5.7"
    - **Voices as Niagara**: one emitter per voice, sprite renderer, one particle per onset from a User parameter such as `User.KickTrigger`, incremented by OSC. Lifetime is `beats · 60 / User.BPM`.
    - **Modules**: a Scratch Pad module for the spring scale and sway rotation, size and color with depth darkening. Pools of 16 per voice are fine.
    - **Material**: unlit sprite material with an SDF per shape type (disc, box, ring, spiral), a one-pixel edge from DDX/DDY, and wobble from noise fed with `floor(Time·12)/12`.
    - **Shadows**: a second sprite renderer, offset toward the back plane, black, translucent at 45 %, with edge softness growing with depth.
    - **Camera**: orthographic for Studie, perspective 30° for Poem so depth is real.
    - **Lumigraph**: depth camera via its plugin or Spout/NDI, a material with the sheet band, and two Canvas Render Targets ping-ponged by a Blueprint each tick.
    - **Post**: a post-process material for gate weave, grain and flicker ([UE post-process materials](https://dev.epicgames.com/documentation/en-us/unreal-engine/post-process-materials-in-unreal-engine)). Use manual exposure, or big discs will pump the image.
    - **Control**: a Remote Control Preset with the parameters and four onset triggers, bound through OSC and MIDI ([UE Remote Control protocols](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-protocols-in-unreal-engine)). Output by Spout or NDI to Resolume.

=== "GODOT"
    - **Voices**: one full-screen `ColorRect` with a `canvas_item` shader for the voices pass. Arrays of floats per voice allow 8 slots per voice instead of 2. A GDScript keeps the slot ring buffers and sets the uniforms every frame.
    - **Light mode**: a second `ColorRect` with `render_mode blend_add`, crossfaded with `modulate.a` by `paper` ([Godot CanvasItem shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/canvas_item_shader.html)).
    - **Lumigraph persistence**: a `SubViewport` with `render_target_clear_mode` set to Never and update mode Always ([Godot SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)). For an exact decay, ping-pong two SubViewports.
    - **Paint-build**: another SubViewport with clear mode Never. Reset it by setting clear mode to Once.
    - **Input**: `InputEventMIDI` for KeyStep notes and BeatStep knobs, `AudioEffectSpectrumAnalyzer` for band energies, an OSC addon if driven from Resolume.

## On stage

**Audio reactivity.** Only onsets are fast. Continuous modulation of size or color by levels destroys the score-like clarity.

| Band | Drives |
|------|--------|
| Kick, 45–115 Hz | bass disc spawn, gated by score and density; the drop disc |
| Snare / clap, 150–250 Hz and 2–5 kHz | snare bar spawn (both bands must fire within 20 ms) |
| Hi-hat, 7–12 kHz | hat dot spawn, half-beat life, 60 % opacity, max 8 per beat |
| Mids, 300 Hz–2 kHz | sway and wobble, slow, +0 to +0.2 |
| Full RMS | rest detection: below -40 dBFS for a bar, everything releases |
| KeyStep 37 notes | melody arcs: pitch gives height, velocity gives size |

**Controls.**

| Parameter | Mapping |
|-----------|---------|
| density | BeatStep Pro knob 1; automate up through builds |
| depth | knob 2; to 0 on builds, to 1 after the drop |
| sway | knob 3; mids add up to +0.2 |
| paper | fader; snap to 0 or 1 with a pad |
| chapter (1–7) | BeatStep Pro pads 1–7, or auto every 32 bars |
| sheet | second player: a fader for the sheet, four knobs for the color slots |

**Chapters.** The vocabulary grows over a set: 1 bass discs only (Studie); 2 add snare bars; 3 add hats and the staff; 4 Poem palette with depth and shadow; 5 melody arcs and spirals; 6 Lumigraph opens; 7 paint-build on. Going back a chapter is a strong break gesture.

**Build, drop, break.**

- **Build**: density climbs from 0.3 to 1.0 over 8 bars, depth and sway fall to 0. The picture gets tense and front-facing.
- **Drop**: one huge bass disc on the downbeat with a bouncy spring, one frame of flash in the disc color, then density 0.6 with full depth and shadow.
- **Break**: hats and snare off, one melody voice draws slow spirals, the Lumigraph performer takes the lead. A full bar of black is allowed.

**Layering.** The output is mostly black, so it sits on top of other layers in Add or Screen without a key. The voices alpha is a clean matte for showing other video only inside the shapes. Keep feedback off the voice layer so the sync stays crisp.

**Color and LED walls.** Keep one flat color per voice, and never color-cycle a living shape. Use #0A0908 or #0B0C14 as black so modules stay just above off. Peak cream at #ECE7DC and cap any single shape at 35 % of screen area on power-limited processors. On low-res LED, widen the edge to 2.5 px and turn wobble and grain down or off.

**Pitfalls.** Fading shapes in (kills the sync), every voice firing on every hit, the same spring on all voices, and auto-exposure in UE.

## Variations

- **Score view**: a scrolling graphic score, like Fischinger's notated plans, that is also the sequencer.
- **Hanging paper mode**: shapes with a slight sway and soft shadows, after the suspended cut-outs of *An Optical Poem*.
- **Radio Dynamics mode**: silent rhythm only. An internal clock drives the step patterns, the visuals lead and the musician follows a click or bar counter.
- **Lumigraph light-sheet mode**: draw with a hand or controller in a thin slit of light, with color set by a second player or a fader ([CVM Lumigraph](https://centerforvisualmusic.org/Fischinger/Lumigraph.htm)). It suits a two-person performance.
- **Paint-build mode**: accumulating strokes that cover and overlay, after *Motion Painting*, reset every N bars.

## Did you know?

- Fischinger met Ruttmann in 1921 and licensed him his wax-slicing machine, which Ruttmann used for backgrounds in *The Adventures of Prince Achmed* ([Wikipedia](https://en.wikipedia.org/wiki/Oskar_Fischinger)).
- The Lumigraph has no keyboard. Lights sit in a frame that emits a thin slit, so only objects moving into the sheet are seen. A second player changes four color wheels by pulling "color control ropes" ([CVM Lumigraph](https://centerforvisualmusic.org/Fischinger/Lumigraph.htm)).
- *Radio Dynamics* is silent. Fischinger made it in secret after being pressed to make a film with sound ([Wikipedia](https://en.wikipedia.org/wiki/Oskar_Fischinger)).
- For *An Optical Poem*, the paper shapes were moved about a millimeter per frame ([TCM](https://www.tcm.com/articles/246892/an-optical-poem)).

## Sources

- [CVM Fischinger biography](https://www.centerforvisualmusic.org/Fischinger/OFBio.htm)
- [CVM Filmography](https://www.centerforvisualmusic.org/Fischinger/Filmography.htm)
- [CVM Process](https://www.centerforvisualmusic.org/Fischinger/OFProcess.htm)
- [CVM Lumigraph](https://centerforvisualmusic.org/Fischinger/Lumigraph.htm)
- [Wax Experiments (excerpt), CVM on Vimeo](https://vimeo.com/54587174)
- [Visual Music Archive: Motion Painting No. 1](https://visualmusicarchive.org/works/w/motion-painting-no-1.html)
- [TCM: An Optical Poem](https://www.tcm.com/articles/246892/an-optical-poem)
- [Wikipedia: Oskar Fischinger](https://en.wikipedia.org/wiki/Oskar_Fischinger)
- [The Book of Shaders: Shapes](https://thebookofshaders.com/07/)
- [ISF JSON reference](https://docs.isf.video/ref_json.html)
- [Resolume ISF support](https://resolume.com/support/en/isf)
- [Resolume effects](https://resolume.com/support/en/effects)
- [Resolume Wire FFT](https://resolume.com/support/en/wire-fft)
- [UE post-process materials](https://dev.epicgames.com/documentation/en-us/unreal-engine/post-process-materials-in-unreal-engine)
- [UE Remote Control protocols](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-control-protocols-in-unreal-engine)
- [Godot CanvasItem shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/canvas_item_shader.html)
- [Godot SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)
