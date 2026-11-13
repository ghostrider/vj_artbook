---
tags:
  - analog-video
  - 3d
  - lines
  - video
---

# Rutt-Etra Scan-Line Displacement

> A picture is torn into glowing rows, and brightness lifts each row off the screen until a face becomes a mountain range of light.

![Still from a video of Rutt-Etra-style scan processing (Pure Data) on a Vectrex CRT: a photo becomes displaced blue scan lines. Not an original Rutt/Etra unit](https://upload.wikimedia.org/wikipedia/commons/thumb/4/49/Vector_Synthesis-_Ocular_Pathogen_Transmission_%281080p%29.webm/1280px--Vector_Synthesis-_Ocular_Pathogen_Transmission_%281080p%29.webm.jpg)
*Still from a video of Rutt-Etra-style scan processing (Pure Data) on a Vectrex CRT: a photo becomes displaced blue scan lines. Not an original Rutt/Etra unit. Image: [macumbista](https://commons.wikimedia.org/wiki/File:Vector_Synthesis-_Ocular_Pathogen_Transmission_(1080p).webm), [CC BY 3.0](https://creativecommons.org/licenses/by/3.0).*

## The idea

The Rutt/Etra Video Synthesizer, also called the Scan Processor, was an analog raster manipulator. It changed a video image by bending the electron beam of a CRT with electromagnetic deflection. Its co-inventors were Steve Rutt and Bill Etra ([Wikipedia](https://en.wikipedia.org/wiki/Rutt/Etra_Video_Synthesizer)). Image brightness pushed the scan lines in three axes, and a camera re-scanned the result ([Video History Project](https://www.videohistoryproject.org/node/5541)).

Software emulations do the same thing in a simpler way. They scan the source top to bottom, draw one polyline per row, and lift each vertex by pixel brightness. Bright areas rise, dark areas recede ([Chrome Experiments](https://experiments.withgoogle.com/rutt-etra-izer), [Airtight](https://www.airtightinteractive.com/2011/06/rutt-etra-izer/)).

On a club screen it works because the picture is mostly black. Thin lines of light on a dark field read from far away, they add cleanly over other layers, and a moving source plus a moving camera keeps the relief alive. Faces and silhouettes read as contour maps. The risk is a static photo seen from the front. That looks dead.

## Origins and references

- [Rutt/Etra Video Synthesizer (Wikipedia)](https://en.wikipedia.org/wiki/Rutt/Etra_Video_Synthesizer) — the device and its beam-deflection principle.
- [Video History Project](https://www.videohistoryproject.org/node/5541) — Etra apprenticed at the WNET TV Lab in 1972. The prototype was finished by the end of 1973. About 20 units were sold (Rutt says 17, Etra says 22). It was a more controllable successor to Nam June Paik's Wobbulator. The original was black and white and had no recordable output.
- [Bill Etra (Wikipedia)](https://en.wikipedia.org/wiki/Bill_Etra) — Etra called *Narcissikon*, made with Louise Etra, his first Rutt-Etra piece. In 1971 he started a performance space at The Kitchen with Steina and Woody Vasulka.
- [Steina and Woody Vasulka (Wikipedia)](https://en.wikipedia.org/wiki/Steina_and_Woody_Vasulka) — after 1974 Woody Vasulka's work moved toward tools like the Rutt/Etra. The page does not say which work used which tool.
- [CDM: Steve Nalepa "Daytime"](https://cdm.link/?p=43365) — a 2012 video made on Etra's own unit by Benton-C Bainbridge, V Owen Bush and Nalepa at dvlabs. Clouds and planets drift out of lace-like line tendrils.
- [Rutt-Etra-Izer by Felix Turner (CAN)](https://www.creativeapplications.net/project/rutt-etra-izer-scripts-webapp/) — the 2011 WebGL emulation. Turner also made an [audio-reactive Processing version](https://www.airtightinteractive.com/2011/06/rutt-etra-izer/).
- [VDMX: Advanced v002 Rutt Etra techniques](https://vdmx.vidvox.net/tutorials/advanced-v002-rutt-etra-techniques) — an RGB-to-XYZ displacement map instead of luma, driven by audio waveform, FFT or camera.

## Visual language

Dense parallel luminous lines on black. Where brightness is high they turn into ridges, terrain or waves. Tilt the camera and the relief appears. The lines bunch together toward the back, like a CRT raster seen from slightly above.

Two looks share one idea. In hidden-line mode each line hides the ones behind it, like a stacked ridgeline plot. In wire mode all lines add up into glowing lace, close to the look of the Nalepa piece above.

The palette is cold and analog. A phosphor cyan line sits on deep black, and peaks glow warmer as they rise, ending in warm white. The original was black and white, so a single phosphor colour is a valid "authentic" mode. Motion comes from the source video, a slow camera orbit and a scroll that lets the image flow through the lines.

## See it

- [Rutt-Etra-Izer demo page](https://experiments.withgoogle.com/rutt-etra-izer) — Chrome Experiments listing of the WebGL emulation. Drag in an image and see the line relief.
- [Rutt-etra Video Synthesizer (YouTube, Sebastian Macchia)](https://www.youtube.com/watch?v=EZQiHuTbnes) — overview video of the original device.
- [Rutt Etra by SYNTHI - A (YouTube, FSOLdigital)](https://www.youtube.com/watch?v=JdDz2lcpPj0) — a Rutt Etra video; title and channel confirmed, content not viewed.
- [CDM: Steve Nalepa "Daytime"](https://cdm.link/?p=43365) — page embedding Rutt/Etra music videos.

## Code design

The designer's approach has three parts. First, clean the source: convert it to luma, blur it by 3 to 6 px, downscale it to 480x270, and apply a contrast curve and gamma. Without the blur, sensor noise and compression blocks turn into jitter. Second, draw the relief. Height is `pow(luma, gamma) * depth`, applied to about 96 rows. Third, post: a black clamp, bloom, then grain.

Where the tool has vertex shaders, the lines are real 3D strips displaced by the source texture, so the camera can orbit. Where it does not, a single-pass fragment shader searches per pixel for the displaced rows that cross it. That gives a fixed oblique view with a roll. The recipes below are a build plan from the designer's knowledge, not tested results.

=== "Resolume Wire"
    Wire has no vertex shaders ([Resolume ISF docs](https://resolume.com/support/en/isf)) and no 3D mesh rendering. The whole relief is therefore the fragment fake inside an ISF node.

    - Chain: effect texture input, Resize to 480x270, Gaussian blur (radius 2 at that size), Levels (black 0.08, white 0.9), ISF node with the relief shader, Bloom, output.
    - The shader loops front to back over only the rows that can reach the pixel (capped at 64). Wire mode adds up the coverage of all rows. Hidden-line mode stops at the first row that covers or occludes the pixel.
    - It is single pass and needs no persistent buffer. Temporal smoothing and the audio-history variation need a buffer, so test your Wire version before building them.
    - Control: low-band audio analysis into an envelope (attack 5 ms, release 250 ms) added to `depth`. A beat counter sets `lines` to 24 times 2 to the power of (bar count mod 4). A slow LFO sets `roll` within 6 degrees.
    - Published parameters: depth, lines, tilt, width, scroll, occlude, palette.

    In Resolume Arena, the designer's search found no built-in line displacement. Load the Wire effect on the source layer: Blur, then the effect, then Bloom, with the layer in Add mode. To colour the lines, group a colour clip in Multiply over it. Whether your Arena build loads ISF effects directly is unverified. Test it before the gig.

=== "Unreal Engine 5.7"
    The main route is a static line mesh displaced in the material through World Position Offset. Niagara ribbons are an alternative, but ribbon ordering at this count needs testing.

    - Mesh `SM_ScanLines`, built in a DCC tool: 160 rows by 256 columns of thin strips, with the same UV on both vertices of a column. Vertex colour flags mark line and displace. Optional skirt strips, drawn black, give hidden-line mode. About 164k vertices.
    - Source: Media Player or NDI texture, drawn each tick into a 480x270 render target by a small prep material (blur, levels, gamma).
    - Material `M_RuttEtra` is Unlit. WPO samples the render target at mip level 0, applies gamma, multiplies by `Depth`, and moves the vertex up. Emissive comes from a height ramp. An opacity mask handles the line-count step.
    - Set Bounds Scale to about 2, or the displaced vertices get culled.
    - Post process: bloom 1.5 with threshold 0.6, manual exposure (Min = Max = 1) so the frame does not dim as lines rise. TSR smears thin lines in motion. Use the forward renderer with MSAA 4x, or keep lines at 1.5 px or thicker.
    - Remote Control: a Material Parameter Collection with Depth, Step, Gamma, Intensity, Scroll and HueShift, bound to OSC and MIDI. Camera yaw and pitch are actor properties.
    - Send the camera view to Arena over Spout or NDI and layer it in Add mode.

=== "GODOT"
    Godot 4 can do the 3D route and the 2D route.

    - 3D: a `@tool` script builds an ArrayMesh with SurfaceTool, with the same layout as the UE mesh. A quick test can use PRIMITIVE_LINES, which are always 1 px wide.
    - Spatial shader with `unshaded`, `cull_disabled` and `blend_add` for wire mode ([Godot spatial shader docs](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/spatial_shader.html)). The vertex stage reads the source texture at LOD 0 and adds the height to `VERTEX.y`. Hidden-line mode uses opaque skirts. Rows are `discard`ed for line-count steps.
    - Source: a 480x270 SubViewport with the video or camera and a small blur and levels shader.
    - Set `extra_cull_margin` to about the maximum displacement. Camera at pitch -30 degrees, FOV 40. WorldEnvironment with a black background and Glow on.
    - 2D: a full-screen ColorRect with a `canvas_item` shader that does the same fragment fake as Wire.
    - Drive the uniforms from an AudioEffectSpectrumAnalyzer on the music bus.

## On stage

**Audio.** The kick adds a depth punch of +0.12 (attack 5 ms, release 250 ms), so the relief jumps and settles. Bass lowers gamma from 2.0 to 1.2, so more of the image rises. Hi-hats add about 0.5 px of line width, but switch this off in hidden-line mode with more than 150 lines, where it looks like noise. Loudness lifts bloom from 0.8 to 1.6 in drops only.

**MIDI.** The BeatStep Pro knobs take the main controls. The KeyStep 37 mod strip is good for live sweeps of the tilt.

| control | parameter |
|---|---|
| Knob 1 | depth (kick envelope adds on top) |
| Knob 2 | lines |
| Knob 3 | tilt |
| Knob 4 | width |
| Knob 5 | scroll |
| Pad 1 | occlude (wire / hidden line) |
| Pad (other) | toggles bar-stepped halving and doubling of lines |
| KeyStep mod strip | tilt sweep |

**Layering.** The lines layer is pure black outside the lines. Add it over anything in wire mode. In hidden-line mode, use Normal if the black skirts must block what is below. A colour clip in Multiply, grouped with the lines only, paints the relief. The line layer's luma can also serve as a mask, so another clip shows only through the lines.

**Arc of a set.**

- Build: line count steps 24, 48, 96, 192, one step per bar. Gamma falls from 2.2 to 1.0 over 8 bars.
- Drop: lines halve to 48, wire mode, depth 0.3, and the camera snaps to a steep tilt in one beat. The accent swaps from cyan to magenta (#FF2E88) for 8 bars.
- Break: depth eases to 0.05 over 2 bars, so the lines go almost flat, a clean CRT raster. Bring depth back on the first kick.
- Change source, mode or camera at least every 16 bars.

**Tips.**

- Use moving relief: faces, dancers, smoke, or slow fractal noise when no video is available.
- Blur the source. Unblurred video jitters.
- Above about 150 lines in wire mode the lines wash to grey. Lower the line brightness or use hidden-line mode.
- On LED walls the background must be true black. Make lines at least 1 LED pixel wide, better 2. On a low-resolution wall keep lines to 80 or fewer.
- Skip scanline darkening on LED walls, since it moires with the pixel grid.
- For tall portrait screens, use vertical lines so they run the long way.

## Variations

1. Audio as image: feed a scrolling spectrum history as the source, so the relief is the sound itself. Front row is now, back row is about 2 s ago.
2. Vertical lines, or radial rings and spokes instead of horizontal rows.
3. Recursive: mix 20 to 40 % of the previous output into the displacement source. The original was re-scanned by a camera.
4. Colour from a second layer, height from the first, in the spirit of the RGB-to-XYZ approach in the [VDMX tutorial](https://vdmx.vidvox.net/tutorials/advanced-v002-rutt-etra-techniques).
5. Beat-locked rotation steps, with line count halving or doubling per bar.

## Did you know?

- About 20 units were built, and the count depends on who you ask: Rutt says 17, Etra says 22 ([Video History Project](https://www.videohistoryproject.org/node/5541)).
- The prototype cost about $13,000 to build. Units sold for about $7,000 to 8,000 ([Video History Project](https://www.videohistoryproject.org/node/5541)).
- In 2012 Etra's own unit was said to be one of fewer than a dozen believed to remain ([CDM](https://cdm.link/?p=43365)).
- Felix Turner's browser emulation dates from June 2011 ([CAN](https://www.creativeapplications.net/project/rutt-etra-izer-scripts-webapp/)).
- The VDMX version can read audio waveforms or FFTs as its displacement lookup ([VDMX](https://vdmx.vidvox.net/tutorials/advanced-v002-rutt-etra-techniques)).

## Sources

- [Wikipedia: Rutt/Etra Video Synthesizer](https://en.wikipedia.org/wiki/Rutt/Etra_Video_Synthesizer)
- [Wikipedia: Bill Etra](https://en.wikipedia.org/wiki/Bill_Etra)
- [Wikipedia: Steina and Woody Vasulka](https://en.wikipedia.org/wiki/Steina_and_Woody_Vasulka)
- [Video History Project: Notes Toward a History of Image-Processed Video](https://www.videohistoryproject.org/node/5541)
- [Chrome Experiments: Rutt-Etra-Izer](https://experiments.withgoogle.com/rutt-etra-izer)
- [Airtight Interactive: Rutt-Etra-Izer](https://www.airtightinteractive.com/2011/06/rutt-etra-izer/)
- [Creative Applications: Rutt-Etra-Izer](https://www.creativeapplications.net/project/rutt-etra-izer-scripts-webapp/)
- [CDM: Nalepa "Daytime"](https://cdm.link/?p=43365)
- [VDMX: Advanced v002 Rutt Etra techniques](https://vdmx.vidvox.net/tutorials/advanced-v002-rutt-etra-techniques)
- [EAI: Noisefields](https://www.eai.org/titles/noisefields)
- [Rutt-etra Video Synthesizer (YouTube, Sebastian Macchia)](https://www.youtube.com/watch?v=EZQiHuTbnes)
- [Rutt Etra by SYNTHI - A (YouTube, FSOLdigital)](https://www.youtube.com/watch?v=JdDz2lcpPj0)
- [Resolume: ISF in Wire](https://resolume.com/support/en/isf)
- [Godot: Spatial shaders](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/spatial_shader.html)
