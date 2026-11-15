---
tags:
  - astronomy
  - lines
  - audio-reactive
  - graphic
---

# Pulsar Ridgeline Waveforms

> Dozens of thin lines, each one a beat from the past, piled into a wire mountain where every peak hides what stands behind it.

## The idea

A ridgeline display stacks many one-dimensional curves. Each row is a trace: time on x, intensity as height. Rows are offset in y and overlap. Every row is filled with the background colour, so a peak in front cuts off the lines behind it. This is the hidden-line effect. The result reads as a solid range of mountains made of wire.

The technique comes from radio astronomy. A pulsar is a dense, fast-spinning star that sweeps a beam of radio waves across the sky like a lighthouse, and each rotation gives one pulse ([University of Cambridge](https://www.cam.ac.uk/stories/journeysofdiscovery-pulsars)). A single pulse is noisy and varies in shape. Plotting many consecutive pulses one under the other shows two things at once: the steady period, as a stable column of peaks, and the pulse-to-pulse variation, as noise on every line ([Scientific American](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/)).

On a club screen the plot becomes a clock you can see. Make one row per beat and the tempo draws itself as a column. A kick pushes a tall peak in at the front and it travels back into the distance as a ridge. The mood is cold, scientific and tense. It is also about 90 percent dark, so it sits well on LED walls and over other material.

This chapter covers the general technique. It does not reproduce any specific published artwork.

## Origins and references

- **Jocelyn Bell (Bell Burnell)** found the first pulsar as a 24-year-old PhD student at Cambridge. Pulses arrive about 1.337 s apart. The object was nicknamed LGM, for Little Green Men, and is now called PSR B1919+21 ([University of Cambridge](https://www.cam.ac.uk/stories/journeysofdiscovery-pulsars), [Big Think](https://bigthink.com/strange-maps/known-pleasures-from-pulsar-signal-to-post-punk-icon/)).
- **R. Wielebinski** observed the same source at Parkes in 1968 and analysed its pulse statistics ([Nature, 1968](https://www.nature.com/articles/2191135a0)).
- **Harold D. Craft Jr.** made the stacked plot for his 1970 Cornell PhD thesis, figure 5-37. It stacks 80 successive pulses recorded at 318 MHz ([Timmes, cococubed](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml), [Scientific American](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/)). He wrote the program that stacked the pulses and tilted the stack slightly, like looking up a hillside ([Big Think](https://bigthink.com/strange-maps/known-pleasures-from-pulsar-signal-to-post-punk-icon/)).
- **Reprints** appeared in Scientific American (1971), Graphis Diagrams (1974) and the Cambridge Encyclopaedia of Astronomy (1977), in different colour schemes ([Big Think](https://bigthink.com/strange-maps/known-pleasures-from-pulsar-signal-to-post-punk-icon/), [cococubed](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml)).
- **Joy Division, *Unknown Pleasures* (1979)**: the album cover uses an inverted version of this plot. The cover is copyrighted and is not shown here. Read its [context at Scientific American](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/).
- **Claus Wilke** (ggridges, ggjoy) made the technique a standard chart type. In 2017 he renamed the "joy plot" to "ridgeline plot" ([Wilke](https://clauswilke.com/blog/2017/09/15/goodbye-joyplots/)). This book uses "ridgeline".
- **Jen Christiansen** (Scientific American) traced the plot back to Craft ([cococubed](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml)). **Frank Timmes** made vector recreations of Craft's figure (same page).

## Visual language

- **Shape.** 48 to 96 thin horizontal traces, stacked from the bottom of the frame to about two thirds up, slightly in perspective. Back rows are narrower, so the stack reads like a hillside seen from below.
- **Silhouette.** The edges of each trace are almost flat and only twitch with noise. The centre band rises into a cluster of sharp peaks.
- **Occlusion.** Each row hides the rows behind it. The fill equals the background, so the occlusion is pure shape.
- **Motion.** A new row arrives at the front on each beat and pushes older rows back and up. Rhythm travels visibly through the mountain. Between steps the rows glide instead of jumping.
- **Rhythm and tension.** The stable column of peaks gives the rhythm. The noise on every line gives the tension.
- **Colour.** Default is warm off-white lines on near-black. Alternatives with historical precedent are black on paper white and white on a cyan fill ([cococubed](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml)). A fourth scheme colours rows by age along a ramp.
- **What gets boring.** A static, symmetrical stack with no variation, a fixed tilt, or white-on-black with no change for a whole set.

## See it

- [Timmes: Unknown Pleasures / CP 1919 page](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml) — history of the plot with vector recreations of Craft's figure 5-37 and its reprints in several colour schemes.
- [Scientific American: the pulsar chart turns 50](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/) — the plot's history, including the white-on-cyan Scientific American version.
- [R-bloggers: CP 1919 dataset in ggridges](https://www.r-bloggers.com/2024/06/cp-1919-psr-b191921-dataset-2/) — the CP 1919 stack redrawn as a ridgeline plot in three colour schemes.
- [R Graph Gallery: ridgeline chart](https://r-graph-gallery.com/ridgeline-plot) — examples of the general ridgeline technique with colour gradients.

## Code design

All data is generated live from audio or noise. The core is a ring buffer of rows plus one render pass that draws them with occlusion.

**Row signal.** Each trace is an array of 512 samples. Three sources: a procedural pulse (2 to 4 Gaussian bumps near the centre plus 1D noise, shaped by a window that keeps the edges flat), a triggered snapshot of the audio waveform, or an FFT snapshot. A scope source needs an oscilloscope-style trigger at a rising zero crossing, or the peaks jump around and no column forms.

**History.** The last N rows (default 64) live in a W × N float texture. A `head` index advances on each step and only one row is written. Nothing is copied. The step is locked to a beat division, so the peak column follows the tempo, as the 1.337 s period does in the pulsar data.

**Render.** Per pixel, walk the rows from front to back. If the pixel is within half a line width of a row's curve, draw the line. If it is below the curve, draw the fill. Otherwise continue. Line width is measured as distance to the curve, so it stays constant on steep flanks. This is the painter's algorithm that ridgeline plot tools use, run front to back so it can exit early ([R-bloggers](https://www.r-bloggers.com/2024/06/cp-1919-psr-b191921-dataset-2/), [R Graph Gallery](https://r-graph-gallery.com/ridgeline-plot)).

**Post.** Bloom (threshold 0.7, intensity 0.6), light monochrome grain, and a vignette that darkens the top 15 percent so the back rows fade into the distance. No full-frame feedback: it smears the lines and kills the occlusion.

**Modes.** Solid (opaque fill), matte (the ridgeline as a mask over video) and see-through (fill alpha 0.2 to 0.5, a wireframe hologram).

=== "Resolume Wire"
    - Arena has no built-in ridgeline generator. Build a custom ISF source with two passes, using persistent float buffers ([ISF JSON reference](https://docs.isf.video/ref_json)). Pass 1 writes the head row into a 512 × 64 `history` buffer. Pass 2 draws the stack. ISF defines `audio` and `audioFFT` inputs ([ISF primer, chapter 8](https://docs.isf.video/primer_chapter_8)). If your Arena version does not feed them, use the pulse source and drive parameters from Arena's audio analysis.
    - In Wire, feed the audio input into an FFT or band analysis (kick 40-120 Hz, bass 60-250 Hz, hats 6-12 kHz). Kick band to an envelope follower (attack 5 ms, release 250 ms), threshold 0.6, triggers `kickPulse`. Bass envelope (attack 40 ms, release 400 ms) adds to height. Hats envelope (attack 5 ms, release 80 ms) drives jitter.
    - Load the ISF in Wire's shader node and connect the parameters plus beat phase. Exact node names depend on your Wire version.
    - Expose height, density, rate, spread, tilt, jitter and scheme as patch parameters so they appear as normal effect parameters in Arena.
    - On the layer: bloom at the low settings above, plus grain if available.

=== "Unreal Engine 5.7"
    - History: `RT_History`, a 512 × 64 render target in R16f, no mips, bilinear in x, never cleared.
    - Row write: on each step event (beat clock or Quartz at 1/8), draw the new row into the rectangle (0, head, 512, 1) with Begin Draw Canvas to Render Target. Only one row is touched.
    - Audio: Audio Synesthesia or a Submix spectrum analysis for the bands. Pulse mode needs only up to 16 band values as material parameters.
    - Route A (2D): a Post Process material with a Custom node doing the front-to-back row walk.
    - Route B (3D fins): a mesh of N vertical strips at z = row index × spacing. World Position Offset lifts the top vertices by height sampled from `RT_History`. The depth buffer does the occlusion, and the camera can fly along the ridges. Niagara ribbons reading the same render target give thicker lines.
    - Post Process Volume: bloom 0.6 / threshold 0.7, film grain, vignette, and fixed manual exposure so the dark field does not auto-brighten.
    - Remote Control: expose `Height`, `Density`, `Spread`, `Tilt`, `Jitter`, `Flash`, `Scheme` and the step rate in a Preset bound to OSC and MIDI. Send out via Spout (RGBA, so matte mode works) or NDI.

=== "GODOT"
    - Godot 4. Band energies from an AudioEffectSpectrumAnalyzer on the music bus. For scope mode, an AudioEffectCapture on the same bus, read each step, trigger, resample.
    - History: an `Image` W × N in FORMAT_RF wrapped in an `ImageTexture`. Each step the script writes one row and calls `update()`. At 512 × 64 this is cheap.
    - Render: a full-screen ColorRect with a `canvas_item` shader doing the row walk. Uniforms include `history`, `head`, `rows`, `height`, `tilt`, `persp`, `shrink`, `line_width`, `flash`, `fill_alpha`, `bg`, `line_col` and a `ramp` GradientTexture1D.
    - Post: WorldEnvironment glow (intensity 0.6, threshold 0.7, 2D HDR on). Grain as a second ColorRect.
    - 3D variant: MeshInstance3D fins with a spatial shader doing the same vertex displacement as the Unreal route B.
    - Beat clock from a script BPM value. MIDI through `OS.open_midi_inputs()` and InputEventMIDI, or an OSC add-on.

## On stage

**Audio reactivity**

| band | drives |
|------|--------|
| kick, 40-120 Hz | one tall peak (1.2 × height) in the newest row, accent colour on that row for 2 steps |
| bass, 60-250 Hz | height, +0 to 2 rows |
| low mids, 250-800 Hz | spread, +0 to 0.1 |
| hats, 6-12 kHz | jitter 0.02-0.15, flash on the front 2 rows |
| RMS loudness | bloom 0.4 to 0.9 |

In scope or spectrum mode the audio is the row itself. Halve the kick and bass mappings to avoid double counting.

**Controls (KeyStep 37 / BeatStep Pro)**

| parameter | range | suggested control |
|-----------|-------|-------------------|
| height | 0-10 rows | KeyStep mod strip or a fader |
| density | 16-128 rows | BeatStep Pro encoder, stepped |
| rate | 1/4, 1/8, 1/16, free | BeatStep Pro pad to cycle |
| spread | 0.01-0.4 | encoder, automate on build-ups |
| tilt | -10° to +10° | encoder, eased over 1 bar |
| jitter | 0-0.15 | hi-hat envelope, with an encoder for offset |

**Song structure**

- **Build-up.** Over the 8 bars before the drop, ramp height from 1 to 1.6 and narrow spread from 0.15 to 0.03. The peaks collapse into one tall column.
- **Drop.** Height doubles over one beat, tilt snaps to +6°, the front 4 rows flash, and the step switches to 1/16 for 4 bars.
- **Break.** Step to 1/4, height at 0.4, pure noise source, bloom down, slow tilt drift.
- **Grid breakers.** Tilt, polar mode, and glitch rows (on a random 1 in 16 steps, duplicate a row 3 to 6 times for a frozen block).

**Layering and transitions**

- Use matte mode over footage: put a video layer above and use the ridgeline as a mask or luma key, so footage shows only inside the mountain, or only in the sky when inverted.
- See-through mode with Add blend works over dark material.
- Swap colour schemes on phrase changes (every 8 or 16 bars) with a 1-beat crossfade.

**Tips**

- Set line width in output pixels, not UV, or lines vanish on 4K walls. On coarse LED walls (pitch above 3 mm) use 3 to 4 px lines and 32 to 48 rows.
- Keep the black level at `#07080A`, not pure black, and line luminance at or below 90 percent. Let bloom provide the rest.
- On a 3:1 strip, use the centre third for the stack, or tile three stacks with offset histories.
- Do not use mipmaps on the history texture. Peaks bleed into neighbouring rows.
- Walk cost grows with rows × pixels. Early exit and a bounded loop keep 64 rows at 1080p cheap. 128 rows at 4K can cost several milliseconds (an estimate, not measured).

## Variations

1. **Spectrum ridgeline.** Each row is an FFT snapshot, so frequency becomes x.
2. **Polar ridgeline.** Rows as concentric rings, peaks pushing outward.
3. **Colour by age.** Rows fade through a palette as they age, driven by BPM.
4. **3D rotation.** The stack as a mesh or Niagara ribbons in Unreal, with the camera flying along the ridges.
5. **Row feedback.** Each row feeds from the one before through a delay, so a kick echoes down the stack.

## Did you know?

- The first pulsar was nicknamed LGM, for Little Green Men ([University of Cambridge](https://www.cam.ac.uk/stories/journeysofdiscovery-pulsars)).
- The 1974 Nobel Prize in Physics went to Antony Hewish and Martin Ryle, not to Bell ([University of Cambridge](https://www.cam.ac.uk/stories/journeysofdiscovery-pulsars)).
- Craft learned of the album cover only when a Cornell professor pointed it out ([Big Think](https://bigthink.com/strange-maps/known-pleasures-from-pulsar-signal-to-post-punk-icon/)).
- Each pulse lasts about 0.04 s, so it fills only about 3 percent of the 1.337 s period ([Scientific American](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/); the percentage is a simple calculation).

## Sources

- [University of Cambridge: journeys of discovery, pulsars](https://www.cam.ac.uk/stories/journeysofdiscovery-pulsars)
- [Scientific American: the pulsar chart turns 50](https://www.scientificamerican.com/article/the-pulsar-chart-that-became-a-pop-icon-turns-50-joy-division-rsquo-s-unknown-pleasures/)
- [Big Think: from pulsar signal to post-punk icon](https://bigthink.com/strange-maps/known-pleasures-from-pulsar-signal-to-post-punk-icon/)
- [Timmes, cococubed](https://cococubed.com/pix_pages/joy_division_unknown_pleasures.shtml)
- [Wilke: Goodbye joyplots](https://clauswilke.com/blog/2017/09/15/goodbye-joyplots/)
- [Wielebinski, Nature 1968](https://www.nature.com/articles/2191135a0)
- [R-bloggers: CP 1919 dataset](https://www.r-bloggers.com/2024/06/cp-1919-psr-b191921-dataset-2/)
- [R Graph Gallery: ridgeline](https://r-graph-gallery.com/ridgeline-plot)
- [ISF JSON reference](https://docs.isf.video/ref_json)
- [ISF primer, chapter 8: audio visualizers](https://docs.isf.video/primer_chapter_8)
