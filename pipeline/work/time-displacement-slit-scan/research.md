# Time-Displacement Slit-Scan — research

## Origin
- Slit-scan began as a photographic trick: a slit is placed in front of the film, and either the film or the scene moves during the exposure. One early accidental example is Lartigue's *Car Trip, Papa at 80 Km/h* (1913), caused by a slow vertical shutter ([Levin's catalogue](https://flong.com/archive/texts/lists/slit_scan/index.html)).
- The film-effects version was developed by John Whitney, who used it for the *Vertigo* titles. He sent test footage to Kubrick, who passed the technique to Douglas Trumbull. Trumbull adapted it for the Stargate sequence in *2001: A Space Odyssey* (1968) with a custom machine ([Wikipedia](https://en.wikipedia.org/wiki/Slit-scan_photography)).
- Bernard Lodge used it for the Doctor Who title sequences from Dec 1973 to Jan 1980. It was also used in *Superman* (1978) and for the Enterprise-D warp stretch in *Star Trek: The Next Generation* ([Wikipedia](https://en.wikipedia.org/wiki/Slit-scan_photography)).
- The digital version cuts thin slices from successive video frames and joins them into a new image, so time becomes a spatial axis. Eddie Elliott's *Video Streamer* (1992) is an early digital example ([Levin's catalogue](https://flong.com/archive/texts/lists/slit_scan/index.html)).
- The VJ-relevant generalization is per-pixel time displacement: Glen Murphy's *Robot Mirror* (2005) is listed in the catalogue as a per-pixel generalization beyond straight slits.

## Key works and artists
- **2001: A Space Odyssey, Stargate** (Douglas Trumbull, 1968): the classic analog slit-scan, with long streaks of light rushing toward the viewer ([Wikipedia](https://en.wikipedia.org/wiki/Slit-scan_photography), [catalogue](https://flong.com/archive/texts/lists/slit_scan/index.html)).
- **Time Scan Mirror** (Daniel Rozin, 2004): scans one vertical line at the center of the camera frame and copies it sideways, giving a roughly 30-second "log" of whatever passed by. Faces turn into multi-angled smears ([Rozin](http://www.smoothware.com/danny/timescan.html)).
- **Khronos Projector** (Alvaro Cassinelli and Masatoshi Ishikawa, 2005): the viewer pushes a deformable screen, and the pushed parts of pre-recorded video move forward or backward in time ([Make](https://makezine.com/article/craft/the-khronos-projector), [SIGGRAPH archive](https://history.siggraph.org/experience/khronos-projector-by-cassinelli-and-ishikawa)).
- **Slithering** (Forrest Oliphant, 2011): dancers stretch, merge and fold into each other in slow motion. It uses a custom tool, Redimensionator, that takes one pixel column from each frame ([VICE](https://www.vice.com/en/article/post-processing-technique-really-makes-these-dancers-slither-video/)).
- **Liquid Time** (Camille Utterback, 2000) and **Don't Look Now** (Brendan Dawes, 2004, every film frame as a one-pixel strip) are further entries in [Levin's catalogue](https://flong.com/archive/texts/lists/slit_scan/index.html), which lists more than 90 projects.

## How it works
- Think of video as a stack of frames: x, y and time. A normal player shows one frame (one slice at constant time). Slit-scan shows a different slice through the stack, so each part of the image comes from a different moment.
- Classic digital slit: take one pixel column per frame and place columns side by side. A moving subject is drawn out into a time-trace ([Rozin](http://www.smoothware.com/danny/timescan.html)).
- Time displacement generalizes this. A grayscale "offset map" says, for each pixel, how many frames back to read. A horizontal ramp gives a classic smear, a radial ramp gives a tunnel, and a moving or noisy map makes the picture ripple. This is my own summary of the standard technique; the same idea is behind Murphy's per-pixel approach in the catalogue.
- Implementation needs a ring buffer of the last N frames (for example 64 to 256). In TouchDesigner the official course pairs a Texture 3D TOP with a Time Machine TOP to "warp images in time" ([Derivative](https://learn.derivative.ca/courses/200-intermediate/lessons/202-tops-intermediate/topic/slit-scan-using-the-time-machine-top/)). The page only names the operators and links an example .tox, so check that file for the network.
- Cost is mostly memory: N frames at 1080p, held on the GPU.

## Visual language
- Smooth flowing streaks, bent lines, and subjects that stretch or slide along the scan direction. Static background stays sharp while moving things become ribbons.
- The look ranges from psychedelic color rivers (Stargate, Doctor Who titles) to uncanny, melted portraits (Rozin, Oliphant).
- Rhythm comes from the map: gradient direction, rotation and softness set whether it reads as a tunnel, a wave or a wipe.
- Needs motion in the source. A still frame gives nothing; a hand, a dancer or a light moving across the frame gives the most.

## Live potential
- **Kick:** punch the delay depth (offset map gain) up and let it decay, so the image smears on the hit and relaxes between hits.
- **Bass:** slowly modulate the map, for example a sine wobble or noise, to ripple the time offsets.
- **Hi-hats:** small jitter on slit position or a ticking step in map angle.
- **BPM:** lock the scan-line sweep or map rotation to one bar, with MIDI faders for depth, angle, softness and feedback amount.
- **Layering:** works as a processing layer on top of any clip, camera or feedback source. Blend the processed and unprocessed image with an opacity fader. Stacks well with kaleidoscope symmetry and feedback.
- **Boring on a big screen:** a source with little motion, a fixed map that never changes, or too much depth, which produces mush. Keep a clear moving subject and vary the map.

## Variations
1. **Radial tunnel map:** a map from distance to center makes a Stargate-like warp tunnel, with the kick pushing the tunnel depth.
2. **Beat-stepped delay:** quantize the offset to whole beats or bars, so the image shows what happened one bar ago in the lower half and now in the top half.
3. **Self-feeding scan:** feed the output back into the buffer as a slow feedback loop, for streaks that persist and drift.
4. **Audio-shaped map:** build the offset map from a spectrum or waveform texture instead of a gradient, so the sound shapes the time warp.
5. **Depth-from-brightness map:** use the clip's own luminance as the offset, so bright parts lag behind dark parts.

## Facts
- The 1913 Lartigue photograph is a slit-scan effect that happened by accident, from a slow shutter ([catalogue](https://flong.com/archive/texts/lists/slit_scan/index.html)).
- Making analog slit-scan was slow: a 10-second sequence at 24 fps needed at least 240 adjustments of the mask ([Wikipedia](https://en.wikipedia.org/wiki/Slit-scan_photography)).
- The same three warp-entry shots made by ILM were reused throughout *Star Trek: The Next Generation* ([Wikipedia](https://en.wikipedia.org/wiki/Slit-scan_photography)).
- Greg Ercolano's *2001 Unwrapping the Slit Scan Sequences* (2001) reversed the Trumbull footage back into source images ([catalogue](https://flong.com/archive/texts/lists/slit_scan/index.html)).
- In *Slithering*, the dancers were filmed before the software was finished, and on set they only saw a still preview ([VICE](https://www.vice.com/en/article/post-processing-technique-really-makes-these-dancers-slither-video/)).
- Rozin's Time Scan Mirror was the first of his Software Mirrors to explore time ([Rozin](http://www.smoothware.com/danny/timescan.html)).

## Visual references
- [Levin's slit-scan catalogue](https://flong.com/archive/texts/lists/slit_scan/index.html) — over 90 projects, with links to videos of Trumbull, Rist, Jarre and others (links on that page not individually opened).
- [Time Scan Mirror, Daniel Rozin](http://www.smoothware.com/danny/timescan.html) — the artist's page for the 2004 mirror, with a screenshot of the piece and a link to a video.
- [Easy Slitscan in TouchDesigner - TouchDesigner Tutorial 067](https://www.youtube.com/watch?v=1ACBmGYB8uw) — a short tutorial building a slit-scan from a few operators (YouTube, The Interactive & Immersive HQ).
- [Slithering, Forrest Oliphant (VICE)](https://www.vice.com/en/article/post-processing-technique-really-makes-these-dancers-slither-video/) — article on the dancer slit-scan piece with embedded video.
- [Khronos Projector (Make)](https://makezine.com/article/craft/the-khronos-projector) — page with photos of the installation; no embedded video.
- [Slit Scan using the Time Machine TOP (Derivative)](https://learn.derivative.ca/courses/200-intermediate/lessons/202-tops-intermediate/topic/slit-scan-using-the-time-machine-top/) — official TouchDesigner lesson with example file.

## Sources
- [Levin, An Informal Catalogue of Slit-Scan Video Artworks and Research](https://flong.com/archive/texts/lists/slit_scan/index.html)
- [Wikipedia, Slit-scan photography](https://en.wikipedia.org/wiki/Slit-scan_photography)
- [Rozin, Time Scan Mirror](http://www.smoothware.com/danny/timescan.html)
- [Make, The Khronos Projector](https://makezine.com/article/craft/the-khronos-projector)
- [SIGGRAPH History Archives, Khronos Projector](https://history.siggraph.org/experience/khronos-projector-by-cassinelli-and-ishikawa)
- [VICE, Slithering](https://www.vice.com/en/article/post-processing-technique-really-makes-these-dancers-slither-video/)
- [Derivative, Slit Scan using the Time Machine TOP](https://learn.derivative.ca/courses/200-intermediate/lessons/202-tops-intermediate/topic/slit-scan-using-the-time-machine-top/)
