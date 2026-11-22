---
tags:
  - look
  - print
  - risograph
  - halftone
  - color
---

# Risograph Overprint and Misregistration

> Turn any footage into a cheap, gorgeous spot-ink print: pink, blue and yellow plates that never quite line up.

## The idea

The look splits the picture into two or three flat ink layers. Each one is built from coarse halftone dots or sandy grain and tinted with a spot colour such as fluorescent pink, blue or yellow. The layers multiply onto an off-white paper colour. Where they overlap, new colours appear: purple, green, orange. Each plate sits a few pixels off, so edges pick up coloured fringes and thin slivers of bare paper show.

It comes from the Risograph, a digital stencil duplicator made by Riso Kagaku and introduced in Japan in 1980 ([Wikipedia: Risograph](https://en.wikipedia.org/wiki/Risograph)). One drum carries one ink colour, so a multicolour print needs one pass per colour. The passes never line up perfectly, and designers use trapping, slight overlaps, to hide the gaps ([College for Creative Studies](https://campus.collegeforcreativestudies.edu/imaging-center/?p=3493)). Misregistration is part of the medium. On screen you do not add a defect; you imitate a process.

On a club screen it works because it is loud, warm and tactile in a field of glossy RGB. A kick can throw the plates apart and let them snap back. A phrase change can swap the whole ink set. Stepped 8 to 12 fps timing makes the picture breathe like hand-printed animation.

**Suits:** high-contrast footage with big shapes, such as faces, silhouettes, bold 3D renders and graphic motion. Busy detail turns to mush at coarse dot sizes.

## Origins and references

- **The machine.** Riso Kagaku goes back to Noboru Hayama, who mixed inks at his kitchen sink after WWII and founded a mimeograph printing company in 1946 ([Wikipedia: Riso Kagaku](https://en.wikipedia.org/wiki/Riso_Kagaku_Corporation)). The Risograph burns holes in a master with a thermal head and pushes soy-based ink through them onto uncoated paper ([Wikipedia: Risograph](https://en.wikipedia.org/wiki/Risograph)). In Japan it is mostly an office and school printer; the artist scene is a later resurgence ([It's Nice That](https://www.itsnicethat.com/articles/the-view-from-tokyo-risograph-in-japan-print-graphic-design-261124)).
- **Hiromu Oka, "Hodo Station" opening titles (TV Asahi, 2021).** Motion was designed in After Effects, one file per colour, then printed in layers, scanned and animated. Over 1,000 A3 sheets went into about 15 seconds. Oka also wrote custom shaders to imitate the Riso texture on screen ([It's Nice That, process](https://www.itsnicethat.com/articles/hiromu-oka-hodo-station-process-animation-051222), [It's Nice That, intro](https://www.itsnicethat.com/articles/hiromu-oka-animation-130422), [Vimeo](https://vimeo.com/639447319)). He calls the unevenness of paper and ink a form of "information unevenness" ([Eye Magazine](https://eyemagazine.com/feature/article/jump-cuts-hiromu-oka)).
- **Julia Schimautz, "Every Thought You've Ever Had" (Pan Well, 2021).** 1,102 frames on 12 A3 sheets in yellow, orange, bright red, teal and black, scanned and composed in After Effects ([People of Print](https://peopleofprint.com/animation/julia-schimautz-every-thought-youve-ever-had-music-video)).
- **Julia Schimautz and Francis Broek (DTAN, Berlin).** Animated Riso pieces and "In Bloom". Colour and movement are planned in After Effects, then printed; mis-registrations make the result partly unpredictable ([It's Nice That](https://www.itsnicethat.com/articles/julia-schimautz-project-animation-021122)).
- **A 70-frame loop** by Schimautz in yellow, fluorescent pink and blue ([Boing Boing](https://boingboing.net/2023/07/09/animation-plays-with-the-explosively-bright-colors-of-a-risograph-machine.html)).
- **More riso animators:** Rapapawn, Suejee Lee and Magdalene Kennedy, who describes her own method ([Kennedy](https://magdalenekennedy.substack.com/p/risograph-animation)).
- **On-screen recipe.** A generative web example uses offset blue and pink stripes, pink halftone dots and a yellow disc that turns green or orange on overlap, all multiplied ([UX Pilot Riso Club](https://uxpilot.ai/labs/creative-direction-playground/riso-club)).

## Visual language

- **Colour.** Fluorescent pink, blue, yellow, teal and orange on off-white paper. Overlaps give purple, green and orange for free. Inks only subtract, so nothing is ever brighter than the paper.
- **Texture.** Halftone dots or grainy gradients instead of smooth tone. Speckle, small white pinholes and uneven ink density.
- **Edges.** Plates are slightly offset, so you get coloured fringes and slivers of paper.
- **Motion.** Each hand-printed frame sits a little differently, so the image jitters and breathes ([It's Nice That](https://www.itsnicethat.com/articles/julia-schimautz-project-animation-021122)). The design runs it at 8 to 12 printed frames per second.
- **Mood.** Zine, gig poster, children's book: warm, handmade, a bit loud.

The key rule is that misregistration must vary, between plates, between frames and across the frame. A static offset reads as a cheap chromatic aberration.

## See it

- [Pan Well, Every Thought You've Ever Had (official video)](https://www.youtube.com/watch?v=VdAVN8zzV38) — music video by Julia Schimautz built from Riso-printed frames; channel Pan Well.
- [Hodo Station 2021 opening, Hiromu Oka](https://vimeo.com/639447319) — news-show titles, every cut printed and scanned on a Riso; uploaded by Hiromu Oka.
- [It's Nice That: Schimautz animations](https://www.itsnicethat.com/articles/julia-schimautz-project-animation-021122) — stills and GIFs of grainy, offset, saturated Riso animation.
- [It's Nice That: Oka, Hodo Station](https://www.itsnicethat.com/articles/hiromu-oka-hodo-station-process-animation-051222) — stills and process of colour-separated layers.
- [Boing Boing: 70-frame riso animation](https://boingboing.net/2023/07/09/animation-plays-with-the-explosively-bright-colors-of-a-risograph-machine.html) — pink, blue, yellow animation.
- [UX Pilot Riso Club](https://uxpilot.ai/labs/creative-direction-playground/riso-club) — a generative web version with offset stripes, halftone dots and multiply.

## Code design

One effect, three passes. A print clock holds the source at `print_fps` (8 to 12). A separation pass turns the picture into one to three ink densities, either tonal (posterised by luma thresholds, deep shadows get all inks) or by channel (CMY-like, keeping source colours). A plate pass then gives each plate its own screen (halftone dots at 15, 75 and 45 degrees, or stochastic grain), its own offset, small rotation, per-print jitter and optional regional wobble. Ink mottle, speckle and drum run-out add texture. Plates multiply onto paper. This is the last pass, with no bloom, sharpen or extra grain after it.

Ink hex values follow the standard Riso list as collected in the [risoinks colour list](https://rdrr.io/github/anfederico/risoinks/src/R/color.R). Screen angles and dot-area rules follow [Stefan Gustavson's halftone tutorial](https://itn-web.it.liu.se/~stegu76/webglshadertutorial/shadertutorial.html) and [Maxime Heckel's Shades of Halftone](https://blog.maximeheckel.com/posts/shades-of-halftone/).

Default inks: Blue `#0078BF`, Fluorescent Pink `#FF48B0`, Yellow `#FFE800` on paper `#F4EFE4`. Alternative sets: Teal / Orange / Yellow, Black / Bright Red / Yellow on newsprint `#E8E2D0`, and a two-colour Pink / Medium Blue on kraft `#C9A982`.

=== "Resolume Wire"
    Patch a "RisoPrint" effect with a texture input and float inputs `offset`, `jitter`, `dot_size`, `grain`, `runout`, `kick`, `print_fps`, plus integers `ink_set` and `plates`.

    - **Print clock:** Time times `print_fps`, then Floor, then a change detector against a one-frame delay. The trigger gates a sample-and-hold of the texture.
    - **Per-print randomness:** `frame_id` seeds a Random/Noise node per plate, scaled by `jitter`.
    - **Separation:** a GLSL node, or Luma plus three Smoothstep nodes, giving three densities.
    - **Plates:** three branches, each a 2D Transform (static offset, jitter, kick times direction, rotation), a GLSL halftone/grain node with its own angle, and Colorize to the ink hex.
    - **Overprint:** Blend nodes in Multiply: paper, then plate 1, 2, 3, then an optional scan texture.
    - **Ink sets:** a Switch node indexed by `ink_set` picks three colours and the paper.
    - **Audio:** the FFT node on the low band into an envelope follower (attack 5 ms, release 200 ms) drives `kick` ([Resolume Wire FFT](https://resolume.com/support/en/wire-fft)). Expose all inputs so Arena can MIDI-map them.

    In Arena there are two other routes. A native path builds each plate as a Multiply group: solid ink colour, with the source below a threshold and a Transform offset, over a solid paper layer. It needs no code but looks more like posterised overprint than real Riso ([Resolume effects reference 7.7](https://resolume.com/support/en/7.7/effects)). The recommended path is one ISF filter with P0 to P2 as passes and a persistent hold buffer ([ISF multipass reference](https://docs.isf.video/ref_multipass.html), [Resolume ISF support](https://resolume.com/support/en/isf)).
=== "Unreal Engine 5.7"
    Use a post-process material (domain Post Process, Blendable Location After Tonemapping) on an unbound Post Process Volume, so the whole render is printed.

    - **Print clock:** a Blueprint timer at `1 / print_fps` draws a copy of the scene into `RT_Hold` and increments `FrameID` in a Material Parameter Collection `MPC_Riso`.
    - **Material graph:** sample `RT_Hold`, separate with a luma `Dot` and three `SmoothStep` nodes (or a 3x3 matrix in channel mode). Per plate, rotate the screen UV, add the offset from the MPC, and hash `FrameID + i` for jitter.
    - **Screen:** halftone via `Frac`, `Distance` to the cell centre and a `SmoothStep` against `sqrt(d)`, with width from `DDX/DDY`. Grain via `Noise` and `Step`. `Lerp` between them with `screen_mix`.
    - **Overprint:** three `Lerp(1, ink_i, cov_i)` nodes multiplied onto paper, then an optional scan texture, into Emissive Color.
    - **Remote Control:** expose `Offset`, `Jitter`, `DotSize`, `Grain`, `Runout`, `Kick`, `PrintFPS`, `Plates` and `InkSet` in a Remote Control Preset, bound to OSC and MIDI.
    - **Output:** over Spout to Resolume on the same machine. NDI's chroma subsampling smears fine dots and fringes, so over NDI send the clean render and print on the receiving side.
=== "GODOT"
    Route the source (a `VideoStreamPlayer`, a camera or Spout texture, or a 3D scene in a `SubViewport`) into a "Hold" `SubViewport` with `render_target_update_mode = UPDATE_ONCE`, re-armed by a script timer at `1 / print_fps` ([Godot SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)).

    A full-screen `ColorRect` on a top `CanvasLayer` runs a `canvas_item` shader that does separation and plates in one pass ([Godot canvas_item shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/canvas_item_shader.html)). Uniforms include `held`, `offset_px`, `jitter_px`, `cell_px`, `screen_mix`, `runout`, `kick`, `frame_id`, `plates`, `ink[3]`, `paper`, `thresholds`, `angles_deg` and an optional `scan` texture.

    The script increments `frame_id` per print, switches ink sets from an array of Color presets, and feeds `kick` from an `AudioEffectSpectrumAnalyzer` low band.

## On stage

**Place in the chain:** last. Put it on the composition or a group so unrelated clips share one print, or on a single layer so only some sources are printed and others stay clean. Any feedback goes before the effect; feeding the print back through itself turns the dots to mud. After the print, add nothing, or at most a 0.3 to 0.5 px blur for ink spread. No bloom, no chromatic aberration, no extra grain, no sharpen.

**Audio reactivity**

- **Kick (40 to 120 Hz):** drives `kick`, which throws the plates apart by 12 to 24 px. Attack 0 to 10 ms, release 150 to 250 ms. Ignore new onsets for 100 ms.
- **Bass:** dot size plus or minus 4 px, or ink opacity. Slow smoothing (attack 80 ms, release 400 ms) so dots swell without flicker.
- **Hi-hats (6 to 12 kHz):** `jitter` plus 0 to 2 px and a grain re-seed. This only lands at print-frame boundaries, so it quantises to `print_fps` by itself.
- **Loudness over 8 bars:** inverse `runout`. Quiet sections run dry and the drop refills the drum.

**MIDI**

| control | parameter | note |
|---|---|---|
| KeyStep 37 knob 1 | `offset` | kick envelope adds on top |
| KeyStep 37 knob 2 | `jitter` | hi-hats add 1 to 2 px |
| KeyStep 37 knob 3 | `dot_size` | 4 to 32 px, default 12 |
| KeyStep 37 knob 4 or fader | `grain` | 0 halftone to 1 grain |
| BeatStep Pro pads 1 to 4 | `ink_set` A to D | quantised to the bar |
| a pad toggle | `print_fps` off/on | 0 means smooth video; or BPM / 15 |

**Timing.** At 120 BPM, `print_fps = BPM / 15` gives 8 fps, four prints per beat. During builds, double it. Rotate the halftone angles by 7.5 degrees per bar so fringe colours move around the shapes. Swap ink sets on phrase lines (8 or 16 bars), never fade between them.

**Build and drop.** Over 8 to 16 bars ramp `runout` to 0.6, `jitter` from 2 to 6 px and `cell` from 12 to 24 px. On the drop, `runout` goes to 0, `offset` jumps to 20 px for one beat and eases back to 4 px over a bar. In a break, drop to two plates, ink set D on kraft, `print_fps` 6, grain mode.

**Tips**

- Cut between full-print and clean frames. A constant three-colour palette becomes a preset.
- Set `offset`, `jitter` and `cell` in pixels relative to 1080p and scale by output height, so a 4K wall looks like the preview.
- Keep `cell` at 3 LED pixels or more, or the dots alias against the LED grid. Use grain mode on very low-resolution walls.
- Compute cells in pixels, not normalised UV, or dots stretch on 32:9 and tall columns.
- Keep screen angles 30 degrees apart (15, 45, 75). Render at the wall's native resolution; scaling afterwards blurs dots and causes moire.
- Print after transport. Halftone and 1 px fringes are worst-case for codecs and NDI.
- If the wall clips fluorescent pink, reduce `ink_2` to 85 per cent. If paper is too bright, darken it to `#E0DACD` rather than lowering ink opacity.
- Flicker safety: at 12 fps or more, full-frame ink swaps on every beat can strobe hard. Keep swaps on bar lines.
- Cost is under 0.5 ms at 1080p for the one-pass version on a mid-range GPU.

## Variations

- **Plates by audio band:** kick drives plate 1, mids plate 2, hi-hats plate 3, so each ink dances to its own instrument.
- **Stepped print animation:** freeze at 6 to 12 fps with random registration per frame, imitating the scanned-frame technique.
- **Drum run-out:** ink density fades like a drying drum, then refills on a drop.
- **Paper stock switch:** newsprint, kraft and white on a fader, which changes how multiply behaves.
- **Real scan loop:** print a few real frames, scan them and use the scans as an overlay texture for grain, fibre and ink pooling, as Oka does with the physical process.

## Did you know?

- The name is not from "lithograph"; it plays on a Japanese word for "ideal" ([It's Nice That](https://www.itsnicethat.com/articles/the-view-from-tokyo-risograph-in-japan-print-graphic-design-261124)).
- A good Riso lasts about 100,000 masters and 5,000,000 copies, and is cheaper per page than copiers above about 100 copies ([Wikipedia](https://en.wikipedia.org/wiki/Risograph)).
- Oka first used a Risograph in 2019 and was reminded of the Print Gocco his parents used ([It's Nice That](https://www.itsnicethat.com/articles/hiromu-oka-animation-130422)).
- Schimautz's music video needed 1,102 frames but only 12 A3 sheets ([People of Print](https://peopleofprint.com/animation/julia-schimautz-every-thought-youve-ever-had-music-video)).

## Sources

- [Wikipedia: Risograph](https://en.wikipedia.org/wiki/Risograph)
- [Wikipedia: Riso Kagaku](https://en.wikipedia.org/wiki/Riso_Kagaku_Corporation)
- [It's Nice That: Risograph in Japan](https://www.itsnicethat.com/articles/the-view-from-tokyo-risograph-in-japan-print-graphic-design-261124)
- [It's Nice That: Oka, Hodo Station process](https://www.itsnicethat.com/articles/hiromu-oka-hodo-station-process-animation-051222)
- [It's Nice That: Oka intro](https://www.itsnicethat.com/articles/hiromu-oka-animation-130422)
- [It's Nice That: Schimautz](https://www.itsnicethat.com/articles/julia-schimautz-project-animation-021122)
- [People of Print: Schimautz](https://peopleofprint.com/animation/julia-schimautz-every-thought-youve-ever-had-music-video)
- [Boing Boing](https://boingboing.net/2023/07/09/animation-plays-with-the-explosively-bright-colors-of-a-risograph-machine.html)
- [College for Creative Studies](https://campus.collegeforcreativestudies.edu/imaging-center/?p=3493)
- [Magdalene Kennedy](https://magdalenekennedy.substack.com/p/risograph-animation)
- [Eye Magazine](https://eyemagazine.com/feature/article/jump-cuts-hiromu-oka)
- [UX Pilot Riso Club](https://uxpilot.ai/labs/creative-direction-playground/riso-club)
- [Vimeo: Hodo Station](https://vimeo.com/639447319)
- [YouTube: Pan Well video](https://www.youtube.com/watch?v=VdAVN8zzV38)
- [risoinks colour list](https://rdrr.io/github/anfederico/risoinks/src/R/color.R)
- [Stefan Gustavson, WebGL halftone tutorial](https://itn-web.it.liu.se/~stegu76/webglshadertutorial/shadertutorial.html)
- [Maxime Heckel, Shades of Halftone](https://blog.maximeheckel.com/posts/shades-of-halftone/)
- [ISF multipass reference](https://docs.isf.video/ref_multipass.html)
- [Resolume ISF support](https://resolume.com/support/en/isf)
- [Resolume effects reference 7.7](https://resolume.com/support/en/7.7/effects)
- [Resolume Wire FFT](https://resolume.com/support/en/wire-fft)
- [Godot SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)
- [Godot canvas_item shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/canvas_item_shader.html)
- [Wepresent: Tools of the trade, risograph](https://wepresent.wetransfer.com/stories/tools-of-the-trade-risograph)
