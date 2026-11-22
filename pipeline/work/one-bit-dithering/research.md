# One-Bit Dithering — research

## Origin
- **Word and idea.** "Dither" appeared in books on analog computation shortly after WWII. Lawrence G. Roberts applied dither to hide quantization patterns in his 1961 MIT thesis; by 1964 it was used in the modern sense ([Wikipedia: Dither](https://en.wikipedia.org/wiki/Dither)).
- **Ordered dithering.** Bryce Bayer presented an optimal threshold method for two-level rendering of continuous-tone pictures at the IEEE ICC in June 1973; hand-built threshold maps by Lippel and Kurland date to 1971 ([Wikipedia: Ordered dithering](https://en.wikipedia.org/wiki/Ordered_dithering)).
- **Error diffusion.** Robert Floyd and Louis Steinberg published their four-neighbour method in 1976 ([Wikipedia: Floyd–Steinberg](https://en.wikipedia.org/wiki/Floyd%E2%80%93Steinberg_dithering)). Variants followed: Jarvis-Judice-Ninke (12 neighbours), Stucki, Burkes, Sierra ([Wikipedia: Dither](https://en.wikipedia.org/wiki/Dither)).
- **Early Mac.** Bill Atkinson, one of the first thirty members of the original Macintosh team, wrote MacPaint and QuickDraw at Apple (1978-1990) and designed the Atkinson variant of Floyd–Steinberg ([Wikipedia: Bill Atkinson](https://en.wikipedia.org/wiki/Bill_Atkinson), [Wikipedia: Atkinson dithering](https://en.wikipedia.org/wiki/Atkinson_dithering)). The 1984 Macintosh had a 512x342 black-and-white screen, and dithered images became a core part of its look ([Beyond Loom](https://beyondloom.com/blog/dither.html)). Atkinson died 5 June 2025.
- **Blue noise.** Robert Ulichney defined blue noise and published the void-and-cluster generation method in 1993 ([Moments in Graphics](https://momentsingraphics.de/BlueNoise.html)).

## Key works and artists
- **Return of the Obra Dinn** (Lucas Pope, released 2018). A first-person 3D game rendered fully in 1-bit, inspired by black-and-white games Pope played on a Macintosh Plus. Pope compared Bayer and blue noise and, in the end, used dithering "as little as possible" ([PlayStation Blog](https://blog.playstation.com/archive/2019/10/17/lucas-pope-on-return-of-the-obra-dinns-art-style), [PC Gamer](https://www.pcgamer.com/lucas-pope-on-the-challenge-of-creating-obra-dinns-1-bit-aesthetic)).
- **Daniel Temkin, Dither Studies** (prints from 2011/2012). Art series that makes the dither pattern itself the subject: web-app generator with custom kernels, installations at the Museum of the Moving Image, projections at Thoma Art House, works in the Buffalo AKG Museum and Rhizome ArtBase ([Temkin](https://danieltemkin.com/DitherStudies/About/)).
- **Windell Oskay (Evil Mad Scientist), live Atkinson webcam sketch** (2012). A Processing sketch that dithers webcam video in real time, with a 512x342 original-Mac-sized window ([Evil Mad Scientist](https://www.evilmadscientist.com/2012/dithering/)).
- **Mars After Midnight devlog "Working in One Bit"** (Pope's itch.io page, Playdate-style Sharp screen). Describes pixel strobing when a dithered image scrolls; moving content needs a static dither applied after movement or art that keeps pixels persistent along the motion ([itch.io devlog](https://dukope.itch.io/mars-after-midnight/devlog/285964/working-in-one-bit)).
- Other 1-bit game and Mac references named by Game Developer: Papers, Please, Downwell, Dark Castle, Shadowgate, Deja Vu ([Game Developer](https://www.gamedeveloper.com/design/lucas-pope-and-the-rise-of-the-1-bit-dither-punk-aesthetic), 30 Nov 2015).

## How it works
Every pixel is compared to a threshold and becomes black or white. Plain thresholding loses all midtones, so dithering distributes the error so that the average over a small area matches the original grey. The eye does the blending at a distance.
- **Ordered (Bayer):** each pixel is compared to a tiled threshold matrix (2x2, 4x4, 8x8...). Pixels do not depend on neighbours, so it is parallel, cheap and stable in animation; look is a cross-hatch grid ([Wikipedia: Ordered dithering](https://en.wikipedia.org/wiki/Ordered_dithering)).
- **Error diffusion (Floyd–Steinberg):** the quantization error is pushed to the right (7/16) and three pixels below (3/16, 5/16, 1/16) ([Wikipedia](https://en.wikipedia.org/wiki/Floyd%E2%80%93Steinberg_dithering)). Sequential and feedback-based, so a small change can shimmer across the frame, which is why it is awkward on the GPU and in video.
- **Atkinson:** like Floyd–Steinberg but spreads only 6/8 of the error, 1/8 to each of six neighbours. Result: more contrast, crisper look, but near-white and near-black areas blow out ([Beyond Loom](https://beyondloom.com/blog/dither.html), [Wikipedia](https://en.wikipedia.org/wiki/Atkinson_dithering)).
- **Blue noise:** compare against a precomputed blue-noise texture instead of a Bayer matrix. It has little low-frequency content, so no visible grid or clumps. Quality is close to error diffusion but it is per-pixel and shader-friendly. Free CC0 tileable textures from 16² to 1024² exist ([Moments in Graphics](https://momentsingraphics.de/BlueNoise.html)). Bart Wronski calls Bayer's patterns unpleasant and blue noise "the way to go" when you can afford the texture fetch ([Wronski](https://bartwronski.com/2016/10/30/dithering-part-three-real-world-2d-quantization-dithering/)).

## Visual language
- Pure black and white, no antialiasing; pixel size is a design choice. Chunky 2-4 px cells read as "Mac"; 1 px on a big screen reads as fine grain.
- Bayer gives geometric hatch grids and visible banding in gradients (graphic, print-like). Blue noise gives organic grain like film or stipple. Floyd–Steinberg gives worm-like maze textures; Atkinson gives high-contrast clean whites.
- Gradients become the star: a vignette or a radial glow turns into a moving moiré of dots.
- Pope found that at large size the pixels no longer blend for the eye, and the pattern gets messy; he used outlines and mostly solid black or white areas for legibility ([PlayStation Blog](https://blog.playstation.com/archive/2019/10/17/lucas-pope-on-return-of-the-obra-dinns-art-style), [Game Developer](https://www.gamedeveloper.com/design/lucas-pope-and-the-rise-of-the-1-bit-dither-punk-aesthetic)).

## Live potential
These are design suggestions drawn from the technique, not claims from sources.
- **Kick:** threshold pump (image flashes toward white then decays); or swap Bayer size 8x8 to 2x2 on the hit.
- **Bass:** drive pattern scale (pixel size) or the brightness offset, so the dither "breathes".
- **Hi-hats:** jitter the matrix offset or step through blue-noise frames for sparkle.
- **BPM:** tempo-synced scroll of the threshold matrix, or blue-noise frame advance on beat divisions.
- **MIDI faders:** algorithm select (Bayer / blue noise / Atkinson), pixel size, threshold, contrast, invert, ink and paper colour.
- **Layering:** works as a final effect on any clip, 3D render (Unreal/Spout) or feedback layer; use as a mask or luma matte to dither only part of a stack. Tint the two colours for 1-bit duotone.
- **Stability:** ordered and blue-noise thresholds are per-pixel and cheap, so they hold in motion. Error diffusion shimmers on moving content. Pope's fix was a "smooth" mode where the dither pattern warps with the camera ([PC Gamer](https://www.pcgamer.com/lucas-pope-on-the-challenge-of-creating-obra-dinns-1-bit-aesthetic)). In a VJ chain, a screen-locked pattern on a moving image gives the "dirty window" effect.
- **Boring when:** the pixel size is tiny on a large LED wall (reads as noise), the image is mid-grey, or it is a static filter with no parameter motion.

## Variations
1. **Beat-matrix swap:** cycle Bayer 2x2, 4x4, 8x8 and blue noise on bar boundaries.
2. **Dither to coloured ink:** map black and white to a two-colour palette that changes per section; ink from a second video layer.
3. **Dither as mask:** use the 1-bit result as an alpha for a different clip underneath (stencil effect).
4. **Custom kernels:** hand-made kernels or odd pixel shapes (hexagons, triangles) in the spirit of Temkin's generator ([Temkin](https://danieltemkin.com/DitherStudies/About/)).
5. **Camera-locked or world-locked pattern:** pattern sticks to 3D objects in UE, or slides with the camera, following Pope's approach.
6. **Audio-reactive error:** feed the spectrum into the diffusion weights so the worm texture reshapes on bass.

## Facts
- Dithering predates computer graphics: random dithering was tried at least as early as 1951 and similar techniques around 1915 ([Wikipedia: Dither](https://en.wikipedia.org/wiki/Dither)).
- Atkinson passes on only three quarters of the error, which is the reason for its signature punchy contrast ([Wikipedia: Atkinson dithering](https://en.wikipedia.org/wiki/Atkinson_dithering)).
- Pope's first fullscreen Obra Dinn dither made some playtesters ill, leading to the "smooth" camera-warped mode ([PC Gamer](https://www.pcgamer.com/lucas-pope-on-the-challenge-of-creating-obra-dinn-1-bit-aesthetic)).
- By 2018 Pope raised the resolution from 640x360 to 800x450 and tied the 3D camera to the 2D pattern to reduce flicker ([PlayStation Blog](https://blog.playstation.com/archive/2019/10/17/lucas-pope-on-return-of-the-obra-dinns-art-style)).
- Christoph Peters released a free database of blue-noise textures under CC0 ([Moments in Graphics](https://momentsingraphics.de/BlueNoise.html)).
- Ordered dithering uses no neighbour information, so every pixel can be computed independently, making it fit for animation ([Wikipedia: Dither](https://en.wikipedia.org/wiki/Dither)).
- Dithered video is also hard to compress: Pope chose a random pattern for environments because it compresses better in video than a regular one ([Game Developer](https://www.gamedeveloper.com/design/lucas-pope-and-the-rise-of-the-1-bit-dither-punk-aesthetic)).

## Visual references
- [Daniel Temkin, Dither Studies](https://danieltemkin.com/DitherStudies/About/) — art series and web app built on dither kernels, shown in installations and projections.
- [Evil Mad Scientist: Atkinson dithering on webcam](https://www.evilmadscientist.com/2012/dithering/) — live webcam turned 1-bit Mac look, with 512x342 window.
- [PlayStation Blog: Pope on Obra Dinn's art style](https://blog.playstation.com/archive/2019/10/17/lucas-pope-on-return-of-the-obra-dinns-art-style) — the 1-bit game look in screenshots and the dithering decisions.
- [PC Gamer: Pope on the 1-bit aesthetic](https://www.pcgamer.com/lucas-pope-on-the-challenge-of-creating-obra-dinns-1-bit-aesthetic) — interview with game stills, smooth versus sharp mode.
- [Game Developer: the 1-bit dither-punk aesthetic](https://www.gamedeveloper.com/design/lucas-pope-and-the-rise-of-the-1-bit-dither-punk-aesthetic) — survey of 1-bit games and Mac art.
- [Moments in Graphics: blue noise textures](https://momentsingraphics.de/BlueNoise.html) — blue-noise textures and dither comparisons.

## Lead image
- kind: free
- image_url: https://live.staticflickr.com/8057/8183886279_b5b4f28938.jpg
- page_url: https://www.flickr.com/photos/oskay/8183886279/
- author: Windell Oskay (Evil Mad Scientist Laboratories), "Goggles&Calipers", 2012
- license: CC BY 2.0, https://creativecommons.org/licenses/by/2.0/ (shown on the Flickr photo page)
- shows: Atkinson dithering made with the Mirror Dither app (webcam dithering sketch, see https://www.evilmadscientist.com/2012/dithering/)

## Quotation images
- kind: quotation
- page_url: https://obradinn.com/
- image_url: https://obradinn.com/img/shots/Ship-01.png
- shows: in-game screenshot of Return of the Obra Dinn (ship view), fully 1-bit rendered; from the screenshot gallery on the game's official site
- work: Return of the Obra Dinn, Lucas Pope, 2018
- rights_holder: 3909 LLC (site footer: "Copyright © 2018-2026, 3909 LLC. All rights reserved.")
- note: Commons was not reachable; the chapter discusses the game in "Origins and references".

## Sources
- [Wikipedia: Dither](https://en.wikipedia.org/wiki/Dither)
- [Wikipedia: Ordered dithering](https://en.wikipedia.org/wiki/Ordered_dithering)
- [Wikipedia: Floyd–Steinberg dithering](https://en.wikipedia.org/wiki/Floyd%E2%80%93Steinberg_dithering)
- [Wikipedia: Atkinson dithering](https://en.wikipedia.org/wiki/Atkinson_dithering)
- [Wikipedia: Bill Atkinson](https://en.wikipedia.org/wiki/Bill_Atkinson)
- [Beyond Loom: Atkinson Dithering](https://beyondloom.com/blog/dither.html)
- [Evil Mad Scientist: dithering](https://www.evilmadscientist.com/2012/dithering/)
- [Moments in Graphics: blue noise](https://momentsingraphics.de/BlueNoise.html)
- [Bart Wronski: Dithering part three](https://bartwronski.com/2016/10/30/dithering-part-three-real-world-2d-quantization-dithering/)
- [PlayStation Blog](https://blog.playstation.com/archive/2019/10/17/lucas-pope-on-return-of-the-obra-dinns-art-style)
- [PC Gamer](https://www.pcgamer.com/lucas-pope-on-the-challenge-of-creating-obra-dinns-1-bit-aesthetic)
- [Game Developer](https://www.gamedeveloper.com/design/lucas-pope-and-the-rise-of-the-1-bit-dither-punk-aesthetic)
- [Pope: Working in One Bit](https://dukope.itch.io/mars-after-midnight/devlog/285964/working-in-one-bit)
- [Daniel Temkin: Dither Studies](https://danieltemkin.com/DitherStudies/About/)
