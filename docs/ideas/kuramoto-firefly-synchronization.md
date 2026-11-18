---
tags:
  - physics
  - biology
  - oscillators
  - sync
---

# Kuramoto Firefly Synchronization

> A field of lights flickers at random, then learns to agree, and one fader decides how fast.

## The idea

Many small clocks, each with its own tempo, pull on each other. Below a critical coupling the crowd stays incoherent. Above it, synchrony emerges, and with strong coupling all oscillators share one frequency. That is the Kuramoto model ([Wikipedia](https://en.wikipedia.org/wiki/Kuramoto_model)).

Fireflies are the natural showcase. About 1 percent of roughly 2,000 firefly species synchronise their flashes over large areas, for example *Photinus carolinus* in the Great Smoky Mountains ([UConn Today](https://today.uconn.edu/?p=39950)). Strictly, real firefly coupling is a pulse rule: each firefly flashes when its clock strikes twelve and nudges its clock forward when it sees a neighbour flash ([Nicky Case](https://ncase.me/fireflies/)). Kuramoto is the smooth-phase cousin. This chapter offers both.

On a club screen the story plays out in real time. Random twinkle, then clusters, then travelling waves, then one unison flash on the downbeat. The performer holds the swarm near the threshold and breaks it on cue. It feels like waiting for a crowd to start clapping in time.

## Origins and references

- [Kuramoto model](https://en.wikipedia.org/wiki/Kuramoto_model): Yoshiki Kuramoto's 1975 paper, motivated by chemical and biological oscillators.
- [Veritasium: The Secret of Synchronization](https://veritasium.com/videos/2021/3/31/the-secret-of-synchronization): popular account, drawing on Steven Strogatz's book *Sync*.
- [Nicky Case, Fireflies](https://ncase.me/fireflies/): explorable. Dots in darkness start random, form clusters, then waves, then near-total unison. Dragging scatters the swarm and it re-syncs. See also [Bram.us](https://bram.us/2021/04/03/synchronizing-fireflies-visualizer).
- [Kuramoto Web Installation](https://www.researchcatalogue.net/view/552571/621219) (2021): David Pirrò, Hanns Holger Rutz, Luc Döbereiner and Daniele Pozzi, for the ALMAT project. Coupled 1D oscillators shown as grey-scale phase, time on the vertical axis.
- [Contingency and Synchronization](https://pirro.mur.at/contsync.html): Luc Döbereiner and David Pirrò, Orpheus Institute, Ghent, March 2019. Six loudspeaker-microphone pairs, each oscillator trying to lock onto what its microphone hears.
- [Interactive Kuramoto animation](https://hdietert.github.io/static/kuramoto-animation/kuramoto.html) by hdietert: phases on a unit circle, order-parameter plot, controls for coupling, noise and perturbation.

## Visual language

- A dark meadow, almost black, with a few thousand small yellow-green points. Each point is a firefly with its own clock.
- Incoherent state: twinkling noise. Near the threshold: clusters and travelling waves. At high coupling: the field pulses as one.
- Colour is a readout of the whole swarm. Incoherent flashes are cool green-cyan, a locked swarm turns warm amber, and the bloom swells with it.
- Each flash is a sharp core with a soft halo. Flashing fireflies swell slightly. A dim ember keeps the field from ever going fully empty.
- Short trails make waves read as moving fronts.
- Optional maths view: a thin ring where every firefly is a dot at its phase angle. The cloud collapses into one clump as the swarm locks ([hdietert](https://hdietert.github.io/static/kuramoto-animation/kuramoto.html)).
- Optional raster strip: greyscale phase scrolling down, after the Kuramoto Web Installation.
- Palette: night #020508, ember #14200C, cool #3AFFB4, firefly #C8FF4D, warm #FFB43C, highlight #FFF6D0.

## See it

- [Nicky Case: Fireflies](https://ncase.me/fireflies/) — glowing dots flashing, clustering into waves, then unison; drag to scatter.
- [Kuramoto animation (hdietert)](https://hdietert.github.io/static/kuramoto-animation/kuramoto.html) — phases on a unit circle, order-parameter plot, perturbation button.
- [Kuramoto Web Installation](https://www.researchcatalogue.net/view/552571/621219) — grey-scale phase raster of coupled oscillators over time.
- [Contingency and Synchronization](https://pirro.mur.at/contsync.html) — sound installation documentation; page links audio and video recordings.
- [Veritasium: The Secret of Synchronization](https://veritasium.com/videos/2021/3/31/the-secret-of-synchronization) — page for the video on sync, with the Kuramoto and firefly interactives in its references.

## Code design

One simulation, one shader. A 64 x 64 float texture holds the swarm, one texel per firefly: phase, natural frequency, flash flag, seed. It is ping-ponged each frame. A two-step reduction (64 to 8 to 1) yields the order parameter r (0 is scattered, near 1 is unison) and the mean phase ψ. The smoothed r drives colour, bloom and the global coupling term. A glow pass draws each pixel from its nine nearest fireflies, a trail buffer adds afterglow, and an optional ring and raster join in Add.

The main engine is mean-field coupling: each phase advances by its own frequency plus K·r·sin(ψ − φ) plus a little noise. Frequencies come from a truncated Lorentzian, whose critical coupling is Kc = 2γ ([Wikipedia](https://en.wikipedia.org/wiki/Kuramoto_model)). The `couple` fader is normalised so that 0.5 sits near the threshold.

A `reach` knob mixes in neighbour-only coupling. At 0 you get spirals and wavefronts that never fully agree. At 1 unison arrives fast. Around 0.2 to 0.4 clusters merge into one wave, then lock. A `rule` switch swaps the smooth sine for the pulse-coupled firefly rule: seeing a neighbour flash nudges the clock forward, with a short refractory window.

Performance gestures: a beat pin pulls the unison flash onto the BPM grid, a kick snaps clocks toward a target phase, and a scatter throws them into chaos. A local scatter punches a hole of chaos that heals from the edges.

=== "Resolume Wire"
    - Build one multipass ISF node: state (persistent, float, grid size), 8 x 8 reduction, 1 x 1 order buffer (persistent), field, trail (persistent, half resolution), optional ring and raster, final composite. Multipass, persistent and float targets are covered by the [ISF multipass reference](https://docs.isf.video/ref_multipass.html). The [Resolume ISF page](https://resolume.com/support/en/isf) does not cover them, so test that `state` keeps its values across frames. A quick test is couple 0, one kick, and all fireflies should flash together afterwards.
    - Feed `beatClock` and `bpm` from Wire's time and beat outputs. Use ISF `TIMEDELTA` for the time step.
    - Fallback: split the state update into its own ISF node and close the loop with a Wire feedback texture node. In 8-bit, rebuild the natural frequency from the seed inside the shader.
    - In Arena, put the patch on a layer in Add over a solid #020508 background. Add Bloom, threshold about 0.65. Publish couple, spread, reach, flash, pin, trail, ring mix, kick and scatter.
=== "Unreal Engine 5.7"
    - Route A, render targets: ping-pong state targets, an 8 x 8 and a 1 x 1 reduction target, a half-resolution trail target. A Blueprint Tick calls `Draw Material to Render Target` in order. Read r once per frame from the 1 x 1 target into a Material Parameter Collection to drive bloom.
    - Route B, Niagara: 2,000 to 8,000 GPU particles in a meadow volume, each reading its phase from the state target by index. Sprite Renderer, Unlit Additive, emissive scaled to feed bloom. Curl noise and drag for gentle drift, slow camera dolly. A Neighbor Grid 3D allows true 3D local coupling; keep N at or below 4,000.
    - Expose the scalars in a Remote Control Preset and bind them to OSC and MIDI. Send BPM from Arena. Return the image via Spout or NDI.
=== "GODOT"
    - Two 64 x 64 SubViewports in HDR 2D with nearest filtering, toggled each frame as ping-pong. An 8 x 8 and a 1 x 1 viewport for the reduction. A full-screen ColorRect for glow and composite, a SubViewport for trails, WorldEnvironment Glow driven by r.
    - For large N, a compute shader through RenderingDevice with a storage buffer and a parallel reduction avoids texture precision questions.
    - Read the audio bands with `AudioEffectSpectrumAnalyzer` and apply the envelopes in `_process`. Send out via Spout or NDI, or window capture into Arena.

## On stage

**Parameters**

| name | range | default | on stage |
|------|-------|---------|----------|
| couple | 0 to 1 | 0.45 | chaos, clusters, unison; 0.5 is near the threshold |
| spread | 0 to 0.5 Hz | 0.12 | wider spread resists sync |
| reach | 0 to 1 | 0.3 | local waves versus instant unison |
| flash | 0.04 to 0.3 cycle | 0.10 | sparkle versus soft breathing |
| pin | 0 to 1 | 0.3 | locks the unison flash to the BPM grid |
| kick / scatter | trigger | 0 | snap to the beat, or throw into chaos |

**MIDI.** Map couple to the KeyStep 37 mod strip or a CC knob, so a build can be ridden by hand. Put spread and reach on BeatStep Pro knobs. Kick on pad 1 and scatter on pad 2 as note-on triggers. KeyStep keys can trigger a local scatter, with the note number setting the position.

**Audio.**

- Kick (40 to 120 Hz): trigger `kick` at amount 0.35. A full reset on every kick kills the emergence. Use 1.0 only on the drop.
- Snare or clap: optional local scatter at a random spot, amount 0.6.
- Hi-hats (6 to 12 kHz): noise, 0 to 0.6 rad/s, keeps the swarm slightly restless.
- Full-band energy: slow envelope adds up to 0.15 to couple, so loud sections pull together.
- Output: smoothed r can drive bloom, camera shake, a bass swell, or the opacity of another layer, so the rest of the show "comes together" when the fireflies do.

**Phrase structure.**

- Intro or break: couple 0.2 to 0.35, reach 0.1. Random twinkle with an occasional cluster.
- Build (8 to 16 bars): ramp couple 0.35 to 0.65 and reach 0.1 to 0.4. Waves roll across the wall and the colour moves from cool to yellow-green.
- Drop: on the downbeat, kick 1 toward the beat phase, couple 0.9, pin 0.4. The wall flashes as one, amber, with full bloom.
- Mid-drop: a scatter of 0.5 every 8 bars. The swarm breaks and re-syncs in 1 to 2 bars.
- Breakdown: scatter 1, couple 0.3, ring overlay fades in. The audience sees the clump dissolve into a full circle.

**Tips.** Sync takes roughly 2 to 8 flash cycles, so a fader move from 0.3 to 0.8 needs 2 to 4 bars to show. Plan ahead. Stay in the 0.4 to 0.7 zone. Do not leave couple above 0.85 or below 0.3 for more than 16 bars. On LED walls, a unison flash is a large brightness jump. Keep flash at 0.08 or more, cap total brightness with a soft clip, avoid unison faster than about 3 flashes per second, and check the venue's rules. Match the grid to the wall's aspect ratio so fireflies stay round.

## Variations

- Spatial coupling only with neighbours, so waves sweep across a grid.
- Rings of oscillators, each ring a different frequency band, with phase as ring rotation.
- Two populations with different frequency distributions. The hdietert demo offers a two-population option.
- Audio feedback: phases modulate shader parameters, with onset pulses as the coupling input, like the sound installation above.
- Hue-wheel mode: hue equals phase. Incoherence becomes confetti, sync turns the field one rotating hue. Good for a break, too busy for long.
- Wrap-around edges for panorama rings and domes.

## Did you know?

- Strogatz's Millennium Bridge paper (Nature, 3 Nov 2005) attributes the 2000 opening-day wobble to pedestrians unintentionally falling into step. The critical crowd was as low as 160, with up to 2,000 on the bridge ([ScienceDaily](https://www.sciencedaily.com/releases/2005/11/051103080801.htm)).
- Female *P. carolinus* in a lab test responded over 80 percent of the time to synchronised flashes and 10 percent or less to out-of-sync ones ([UConn Today](https://today.uconn.edu/?p=39950)).
- Kuramoto was surprised when arrays of Josephson junctions followed his model ([Wikipedia](https://en.wikipedia.org/wiki/Kuramoto_model)).
- The model is applied in neuroscience, power grids and flocking ([Wikipedia](https://en.wikipedia.org/wiki/Kuramoto_model)).

## Sources

- [Wikipedia: Kuramoto model](https://en.wikipedia.org/wiki/Kuramoto_model)
- [Veritasium page](https://veritasium.com/videos/2021/3/31/the-secret-of-synchronization)
- [UConn Today](https://today.uconn.edu/?p=39950)
- [Nicky Case, Fireflies](https://ncase.me/fireflies/)
- [Bram.us](https://bram.us/2021/04/03/synchronizing-fireflies-visualizer)
- [Research Catalogue: Kuramoto Web Installation](https://www.researchcatalogue.net/view/552571/621219)
- [pirro.mur.at: Contingency and Synchronization](https://pirro.mur.at/contsync.html)
- [hdietert animation](https://hdietert.github.io/static/kuramoto-animation/kuramoto.html)
- [ScienceDaily](https://www.sciencedaily.com/releases/2005/11/051103080801.htm)
- [Resolume ISF](https://resolume.com/support/en/isf)
- [ISF multipass reference](https://docs.isf.video/ref_multipass.html)
