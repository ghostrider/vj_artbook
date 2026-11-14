# Oscilloscope XY Vector Art — research

## Origin
- Math: Nathaniel Bowditch studied the curves in 1815 and Jules Antoine Lissajous studied them in more detail in 1857. Two perpendicular sine oscillations at different frequencies draw the figure ([Wikipedia: Lissajous curve](https://en.wikipedia.org/wiki/Lissajous_curve)).
- Art: Ben F. Laposky (Iowa, 1914–2000) began with pendulum tracings in 1946. He photographed oscilloscope patterns he called "Oscillons", and *Scripta Mathematica* published the first ones in 1952. He toured the "Electronic Abstractions" show in the US and France from 1952 to 1961 ([Wikipedia: Ben F. Laposky](https://en.wikipedia.org/wiki/Ben_F._Laposky)).
- Today's form: Jerobeam Fenderson and Hansi Raber released the Kickstarter-funded album *Oscilloscope Music* (page dated 25 Oct 2016). The tracks are synthesized so that the stereo audio itself draws the shapes on an analogue scope ([Creative Applications](https://www.creativeapplications.net/project/oscilloscope-music-jerobeam-fenderson-and-hansi-raber/)).

## Key works and artists
- **Jerobeam Fenderson**: his videos show hand-crafted audio drawing on a Tektronix D11 5103N scope. The "dancing mushrooms" video used an RME Fireface UC at 192 kHz ([IEEE Spectrum](https://spectrum.ieee.org/jerobeam-fendersons-trippy-oscilloscope-music)).
- **Primer** by DJ_Level_3 and Marv1994 (BUS ERROR Collective): it starts with simple waveforms and builds to spinning, morphing shapes. It won the Wild category and the Crowd Favorite award at Revision 2025 ([Hackaday](https://hackaday.com/2025/04/26/amazing-oscilloscope-demo-scores-the-win-at-revision-2025)).
- **Ben Laposky's Oscillons**: photographed analogue vector abstractions, with colour added by motorized rotating filters ([Wikipedia](https://en.wikipedia.org/wiki/Ben_F._Laposky)).
- **N-Spheres EP** by Jerobeam Fenderson and Hansi3D: a five-track EP (Function, Intersect, Attractor, Flux, Core) ([oscilloscopemusic.com](https://oscilloscopemusic.com/)).
- **Hobby players**: a Teensy 3.2 with an Audio Shield plays stereo WAV files from SD into an antique scope's X and Y inputs ([PJRC](https://www.pjrc.com/oscilloscope-music-visualizer/)).

## How it works
In XY mode the scope ignores its time base. The left channel moves the beam horizontally and the right channel moves it vertically ([IEEE Spectrum](https://spectrum.ieee.org/jerobeam-fendersons-trippy-oscilloscope-music)). A sample pair (L, R) is therefore a point, and fast playback traces a continuous line. Two sines with frequency ratio a/b give a Lissajous figure: 1 gives an ellipse, a rational ratio gives a closed pattern, and an irrational ratio makes it look like it rotates ([Wikipedia](https://en.wikipedia.org/wiki/Lissajous_curve)). Any drawing, such as an SVG path or a 3D model, can be turned into a sample stream. The tools osci-render (SVG, .obj, text and Blender scenes, with Lua scripting and MIDI control) and OsciStudio (Blender link) do this ([osci-render](https://github.com/jameshball/osci-render), [Creative Applications](https://www.creativeapplications.net/project/oscilloscope-music-jerobeam-fenderson-and-hansi-raber/)).

## Visual language
Single bright glowing line, usually green phosphor, on black. The look is wireframe geometry, spirals, spinning 3D wireframe objects, Lissajous knots and line portraits (see the Fenderson videos below). The line is never broken, so shapes are drawn "in one stroke" and brightness builds where the beam lingers or retraces. Motion is smooth and continuous because it is the sound itself.

## Live potential
- Software scope (Unreal, TouchDesigner, shader): accumulate the last N samples as a line strip, then add decay (phosphor persistence) and bloom.
- Audio mapping: kick scales the radius or a Lissajous ratio jump. Bass sets a/b. Hi-hats add jitter or phase offset. BPM sets the rotation speed, and MIDI faders set the ratio, phase, point count, decay and beam thickness.
- Layering: additive blend over footage, or use as a mask and displacement source in Resolume. Colour it with a gradient or keep it green.
- Boring risks: a static ellipse with no change, and constant thin lines on a big screen. Fix with persistence trails, intensity bloom, depth and shape morphs on the beat.
- Sound routing: the audio can be both music and drawing only if the track is composed for it. An alternative for a VJ rig is to drive the picture with generated oscillators, or with the incoming audio only as modulation.

## Variations
1. Phosphor simulator in Unreal or a shader, with variable persistence, gamma and a curved CRT mask.
2. Lissajous morphing: step the frequency ratio on the beat between rational values, then drift to irrational ones.
3. SVG and logo to path: draw a logo or typography as one stroke, with a wobble from the hi-hats.
4. 3D wireframe objects (osci-render style) rotating on BPM, with depth-based brightness.
5. Output to a real or simulated laser (OsciStudio 7 supports WickedLasers projectors) ([oscilloscopemusic.com](https://oscilloscopemusic.com/)). Colour-rotating filters as in Laposky's Oscillons.

## Facts
- Fenderson's setup: a computer audio interface feeds the scope, left channel horizontal and right channel vertical ([IEEE Spectrum](https://spectrum.ieee.org/jerobeam-fendersons-trippy-oscilloscope-music)).
- Laposky's work was published over 160 times and shown at over 200 exhibitions ([Wikipedia](https://en.wikipedia.org/wiki/Ben_F._Laposky)).
- Lissajous XY display is also used in professional audio to check stereo phase ([Wikipedia](https://en.wikipedia.org/wiki/Lissajous_curve)).
- Primer was recorded on a BK Precision 2120 scope with a green display ([Hackaday](https://hackaday.com/2025/04/26/amazing-oscilloscope-demo-scores-the-win-at-revision-2025)).
- osci-render runs as a standalone app or a VST3/AU plugin and is GPLv3 ([GitHub](https://github.com/jameshball/osci-render)).
- Max/MSP and Pure Data patches were released along with the album ([Creative Applications](https://www.creativeapplications.net/project/oscilloscope-music-jerobeam-fenderson-and-hansi-raber/)).

## Visual references
- [How To Draw Mushrooms On An Oscilloscope With Sound](https://www.youtube.com/watch?v=rtR63-ecUNo) — mushrooms drawn on an oscilloscope by sound (YouTube, Jellybean Fenderson)
- [Khrậng](https://www.youtube.com/watch?v=vAyCl4IHIz8) — oscilloscope music piece (YouTube, Jellybean Fenderson)
- [IEEE Spectrum feature](https://spectrum.ieee.org/jerobeam-fendersons-trippy-oscilloscope-music) — write-up with the stills and setup photos.
- [Oscilloscope Music site](https://oscilloscopemusic.com/) — N-Spheres EP videos and OsciStudio.
- [osci-render](https://github.com/jameshball/osci-render) — tool with a built-in software oscilloscope.

## Lead image
- file_page: https://commons.wikimedia.org/wiki/File:Analog_oscilloscope.jpg
- image_url: https://upload.wikimedia.org/wikipedia/commons/0/0f/Analog_oscilloscope.jpg
- shows: the screen of an analogue Goldstar oscilloscope in XY mode
- author: premek.v
- license: Public domain
- license_url: https://commons.wikimedia.org/wiki/File:Analog_oscilloscope.jpg

## Sources
- [Creative Applications: Oscilloscope Music](https://www.creativeapplications.net/project/oscilloscope-music-jerobeam-fenderson-and-hansi-raber/)
- [Wikipedia: Ben F. Laposky](https://en.wikipedia.org/wiki/Ben_F._Laposky)
- [Wikipedia: Lissajous curve](https://en.wikipedia.org/wiki/Lissajous_curve)
- [Hackaday: Primer at Revision 2025](https://hackaday.com/2025/04/26/amazing-oscilloscope-demo-scores-the-win-at-revision-2025)
- [IEEE Spectrum](https://spectrum.ieee.org/jerobeam-fendersons-trippy-oscilloscope-music)
- [oscilloscopemusic.com](https://oscilloscopemusic.com/)
- [osci-render on GitHub](https://github.com/jameshball/osci-render)
- [PJRC oscilloscope music player](https://www.pjrc.com/oscilloscope-music-visualizer/)
