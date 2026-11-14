# Analog Video Feedback Loops — research

## Origin
- Video feedback is what you get when a camera looks at a monitor that shows the camera's own live output. Each captured frame contains the previous one, so the image recurses. [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- Video artists started using it as an image source (not just a glitch) in the late 1960s and early 1970s. Steina and Woody Vasulka worked with feedback from about 1970-71; their 1971 piece *Elements* ran feedback through a keyer and colorizer. [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- Skip Sweeney, who worked with feedback from the late 1960s, co-founded Electric Eye (1969) and Video Free America (1970, with Arthur Ginsberg) in San Francisco. A catalogue entry calls him the acknowledged master of feedback. His tools were a studio camera and a monitor with contrast, brightness and zoom controls. [Vasulka archive, Eigenwelt p. 148-149](https://www.vasulka.org/archive/eigenwelt/pdf/148-149.pdf)
- British TV used the same effect as "howlround". Engineer Norman Taylor found it, and Bernard Lodge turned it into the Doctor Who title sequence (1963-1973), replaced in 1973 by slit-scan. [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- Science caught up in 1984: physicist James P. Crutchfield published "Space-Time Dynamics in Video Feedback" (Physica D 10, 229-245), treating the loop as a space-time analog computer. [UC Davis Chaos group](https://csc.ucdavis.edu/~chaos/chaos/pubs/stdvf-title.html)

## Key works and artists
- **Steina and Woody Vasulka, *Elements* (1971)**: early feedback through keyer and colorizer. [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- **Skip Sweeney, *Illuminatin' Sweeney* (1975)**: feedback combined with synthesizers (Moog, Vidium). [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- **Doctor Who title sequence (1963-1973), Bernard Lodge**: monochrome howlround; a 1967 version added Patrick Troughton's face. [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- **Queen, "Bohemian Rhapsody" video (1975), dir. Bruce Gowers**: the cascading Freddie Mercury images were made in-camera by feeding a monitor back into a camera (per Wikipedia, citing the BBC). [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- **VJ Mandala (Adem Jaffers)**: used feedback in Melbourne's rave scene and the *Cyberthon* broadcasts (1991-1995). [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- **Dave Blair, "The Light Herder" / the Device**: a modern kinetic sculpture making fractals with no computer: HD/4K monitors with analog hue, saturation, brightness and contrast knobs, up to three cameras, 50/50 beam-splitter glass between monitors, a Roland V-02HD switcher/keyer, and delay converters; the latest version uses five 4K screens. [thelightherder.com](https://www.thelightherder.com/), [Hackaday, 2026-08-27](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/)

## How it works
- Camera position and angle, zoom, focus, and the monitor's brightness and contrast all change the result; tiny changes can give very different patterns (concentric, rotating or irregular). [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- Crutchfield modelled it two ways: a discrete-time iterated functional equation (each frame = transformed previous frame) and a reaction-diffusion PDE. Digital simulations reproduced the main behaviour of the real rig. [UC Davis Chaos group](https://csc.ucdavis.edu/~chaos/chaos/pubs/stdvf-title.html)
- Plain version: output = blend(new input, transform(previous output)), where the transform is zoom, rotation, shift, colour and gain. Camera lag, lens blur and the screen's finite brightness act as the free "damping" that stops it blowing out.
- Rotation and magnification set the symmetry of the spot patterns (Essevaz-Roulet et al., 2000, via [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)). Optics papers also show feedback with mirrors/lenses or pixel-based monitors can give stationary fractals such as Koch-snowflake and Sierpinski-like forms (Courtial, Leach, Padgett, *Nature* 2001, via [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)).
- Emulation: TouchDesigner's Feedback TOP supplies the previous frame so a loop can be built without cook dependencies or infinite recursion; a Composite TOP is part of the loop. [Derivative learn](https://learn.derivative.ca/courses/100-fundamentals/lessons/102-tops-working-with-images/topic/creating-a-feedback/) The same idea works as a ping-pong render target in Resolume, Wire, Unreal or a shader.

## Visual language
- Tunnels and spirals receding to a centre, nested copies of the frame, rotating arms, and lattices of bright spots with n-fold symmetry.
- Smeared, glowing trails; colour drift as hue shifts each pass; hard-edged inputs (a white shape on black) turn into organic filaments.
- Motion is self-propelled: it grows, locks into a pattern, then collapses or mutates on a small knob move. Rhythm comes from the loop delay (frame lag).
- Look splits between analog (soft, CRT-like, chromatic, noisy) and digital (crisp, can be pushed to fractal sharpness).

## Live potential
- Kick: pulse the zoom or gain for one beat (zoom ~1.02-1.1 then decay), or inject a flash of new input into the loop. Bass envelope: rotation speed or hue drift. Hi-hats: tiny jitter of offset or a thin outline injection, which seeds new detail.
- BPM: lock the rotation per bar (for example a full turn every 16 beats) and tempo-sync hue cycles; step the loop's transform on beat divisions.
- MIDI faders (KeyStep/BeatStep Pro knobs): zoom, rotation, hue shift, feedback gain, blur, delay frames, input mix. Knobs are the original interface (Blair's rig is operated by hue/saturation/brightness/contrast knobs), so mapping is natural.
- Layering: use it as a texture over other material: feed a clip, text or Niagara render in as the seed; or run it as a mask or displacement map for another layer; or key it with a colour so only the glow shows. Spout/NDI output of the loop can be re-fed into Resolume as a source.
- Boring risks: sitting at one fixed point (it dies to black, white or a static pattern), constant maximum gain (white-out), only a centred zoom (looks like a screensaver). Always keep a seed injection and a decay, and change parameters in slow sweeps plus rare hits.

## Variations
- **Two-loop cross-feed**: two feedback chains mix a fraction of each other, as in Blair's two monitor structures that influence each other. [Hackaday](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/)
- **Mirror/beam-splitter fold**: add a mirrored copy of the previous frame into the loop to emulate the glass trick and get kaleidoscopic fractal forms. [thelightherder.com](https://www.thelightherder.com/)
- **Frame-delay stack**: feed back from N frames ago (Blair's rig uses delay units of up to 30 frames, per Hackaday comments) for echo rings. [Hackaday](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/)
- **Seeded feedback**: inject text, a logo or a live camera of the booth; Doctor Who's loop was fed a logo. [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- **Reaction-diffusion cousin**: add blur and threshold in the loop, following Crutchfield's reaction-diffusion model, for Turing-like spots. [UC Davis](https://csc.ucdavis.edu/~chaos/chaos/pubs/stdvf-title.html)
- **Sync to slit-scan**: the book already has time-displacement; use its output as the loop's seed rather than repeating it.

## Facts
- The 1973 Doctor Who titles dropped feedback for slit-scan, inspired by the 2001 star gate. [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- Hofstadter's *I Am a Strange Loop* (2007) has a chapter "On Video Feedback" using the camera-monitor loop to illustrate recursion. [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- Sweeney called feedback "a religion - a wave to ride." [Vasulka archive](https://www.vasulka.org/archive/eigenwelt/pdf/148-149.pdf)
- Feedback with pixel-based monitors can produce stationary fractals (Nature, 2001). [Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- Blair's rig drives LCD display settings from knobs via a Teensy over RS-232. [Hackaday](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/)
- The Light Herder page lists praise from Douglas Hofstadter and Heinz-Otto Peitgen. [thelightherder.com](https://www.thelightherder.com/)

## Visual references
- [The New 4K Video Feedback Fractal Device (YouTube, channel The Light Herder)](https://www.youtube.com/watch?v=8XjQ10ph4yo) — Dave Blair's analog camera-at-screen rig making fractals in motion.
- [4K Analog Video Feedback Fractal Device Finally Complete (YouTube, channel The Light Herder)](https://www.youtube.com/watch?v=koDCabeh5kQ) — the build and first results of the same device.
- [The Light Herder](https://www.thelightherder.com/) — artist project page with hardware description and video links.
- [Hackaday: Analog Optical Feedback Generates 4K Fractals](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/) — write-up of the rig and its knobs.
- [Wikimedia Commons: Video feedback octagon pattern](https://commons.wikimedia.org/wiki/File:Video_feedback_octagon_pattern.jpg) — a still of a symmetric camera-at-monitor feedback pattern.

## Lead image
- file_page: https://commons.wikimedia.org/wiki/File:Video_feedback_octagon_pattern.jpg
- image_url: https://upload.wikimedia.org/wikipedia/commons/thumb/6/69/Video_feedback_octagon_pattern.jpg/1280px-Video_feedback_octagon_pattern.jpg
- shows: a camera-at-monitor feedback pattern with octagonal symmetry (Commons description: "The mirror effect")
- author: Deathgleaner
- license: CC BY-SA 3.0
- license_url: https://creativecommons.org/licenses/by-sa/3.0

## Sources
- [Video feedback, Wikipedia](https://en.wikipedia.org/wiki/Video_feedback)
- [Skip Sweeney entry, Vasulka archive](https://www.vasulka.org/archive/eigenwelt/pdf/148-149.pdf)
- [Crutchfield, Space-Time Dynamics in Video Feedback](https://csc.ucdavis.edu/~chaos/chaos/pubs/stdvf-title.html)
- [The Light Herder](https://www.thelightherder.com/)
- [Hackaday, 2026-08-27](https://hackaday.com/2026/08/27/analog-optical-feedback-generates-4k-fractals-with-no-computer/)
- [Derivative Learn: Creating a Feedback](https://learn.derivative.ca/courses/100-fundamentals/lessons/102-tops-working-with-images/topic/creating-a-feedback/)
- [Commons API: Video feedback octagon pattern.jpg](https://commons.wikimedia.org/wiki/File:Video_feedback_octagon_pattern.jpg)
